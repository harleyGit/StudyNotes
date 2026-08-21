- [每轮批量消费结束后的收尾动作](#每轮批量消费结束后的收尾动作)
	- [提交成功记录](#提交成功记录)
	- [恢复失败分区位点](#恢复失败分区位点)
	- [放行重平衡](#放行重平衡)
- [解耦业务与 Kafka 底层](#解耦业务与Kafka底层)
	- [适配流程](#Kafka适配流程)
	- [错误分类](#Kafka适配错误分类)

<br/>

***
<br/><br/><br/>
> <h1 id="每轮批量消费结束后的收尾动作">每轮批量消费结束后的收尾动作</h1>

```go
func (b *HGBaseConsumer) hgCommitAndRestore(ctx context.Context,
    commitRecords []*kgo.Record,
    failedOffsets map[string]map[int32]kgo.EpochOffset) {
    if len(commitRecords) > 0 {
        startedAt := time.Now()
        err := b.cli.CommitRecords(ctx, commitRecords...)
        hgObserveCommit(len(commitRecords), time.Since(startedAt), err)
    }
    if len(failedOffsets) > 0 {
        b.cli.SetOffsets(failedOffsets)
    }
    b.cli.AllowRebalance()
}
```

该函数承接一轮批量消费的处理结果，依次提交成功记录、设置失败分区下一次拉取的起点，并放行可能被阻塞的消费者组重平衡。

参数含义：

- `commitRecords []*kgo.Record`：用于提交消费位点的记录。`CommitRecords` 按分区提交所给记录之后的位点，即通常提交 `record.Offset + 1`。
- `failedOffsets map[string]map[int32]kgo.EpochOffset`：`topic -> partition -> EpochOffset`，指定失败分区后续应从哪个 leader epoch 和 offset 开始消费。

***
<br/>

> <h2 id="提交成功记录">提交成功记录</h2>

```go
if len(commitRecords) > 0 {
    startedAt := time.Now()
    err := b.cli.CommitRecords(ctx, commitRecords...)
    hgObserveCommit(len(commitRecords), time.Since(startedAt), err)
}
```

`CommitRecords` 向 broker 提交位点，并将提交条数、耗时和错误交给 `hgObserveCommit` 记录。函数没有根据 `err` 中断、重试或返回错误；是否已有日志或告警，取决于 `hgObserveCommit` 的具体实现。

同一分区通常只需提交**连续处理成功前缀中的最后一条记录**。如果成功记录之间存在尚未处理或处理失败的空洞，直接提交更高 offset 可能跳过空洞，因此不能简单理解为“任意最后一条成功记录”。

提交失败时，broker 位点可能保持不变；若进程随后退出，已处理消息可能被再次消费，这符合常见的 at-least-once 行为。后续一轮是否自然补交，取决于后续拉取、处理和提交是否继续成功。

***
<br/>

> <h2 id="恢复失败分区位点">恢复失败分区位点</h2>

```go
if len(failedOffsets) > 0 {
    b.cli.SetOffsets(failedOffsets)
}
```

`SetOffsets` 设置客户端后续消费指定分区时使用的本地位点，不会把这些位点提交到 broker。常见用途包括：

- **可重试错误**：定位到失败记录，后续重新处理。
- **终止错误且 DLQ 投递成功**：定位到失败记录之后，跳过已进入死信队列的记录。

需要注意，`SetOffsets` 不会撤回应用已经取得的旧 fetch 结果。如果程序并发处理已拉取记录，或调用后仍处理旧批次，旧记录仍可能被处理；应结合实际 poll/处理模型、分区串行约束及 franz-go 文档确认安全边界。

当前顺序是先 `CommitRecords`、再 `SetOffsets`。二者分别使用显式记录和显式位点，不能笼统认为调换顺序就一定会把 seek 位点提交出去；但在本函数的批次收尾语义中，保持现有顺序更清晰，也避免在提交完成前提前改变下一轮本地起点。

***
<br/>

> <h2 id="放行重平衡">放行重平衡</h2>

```go
b.cli.AllowRebalance()
```

`AllowRebalance` 主要与 franz-go 的 `BlockRebalanceOnPoll` 配置配合：应用处理完本次 poll 返回的记录后，显式允许客户端执行被阻塞的重平衡。调用后，分区可能很快被撤销，因此不应继续使用本批次记录做未完成处理。

如果没有启用 `BlockRebalanceOnPoll`，该调用通常没有对应的“解锁本批次”作用。若启用了该选项却长期不调用，重平衡可能持续被阻塞，并增加超过 `max.poll.interval.ms` 等风险；具体表现取决于客户端配置和处理耗时。

整体调用链：

```text
业务处理完一批 fetches
        ↓
commitRecords：各个分区可安全提交的最后一条记录
failedOffsets：失败分区需要恢复的位点
        ↓
hgCommitAndRestore
    CommitRecords 提交成功位点
    SetOffsets 设置失败分区的本地位点
    AllowRebalance 放行重平衡
        ↓
回到循环，下一轮 PollRecords 拉取消息
```

**结论：**`hgCommitAndRestore` 实现“提交可确认位点 + 恢复失败分区 + 放行重平衡”；正确性依赖 `commitRecords` 没有跨过处理空洞，并且调用时已停止处理本批次记录。

***
<br/><br/><br/>

> <h1 id="解耦业务与Kafka底层">解耦业务与 Kafka 底层</h1>

```go
func hgDomainEventRecordHandler(handler consumer.Handler) HGKafkaPackage.HGRecordHandler {
	return func(ctx context.Context, record *kgo.Record) error {
		ctx = consumer.WithDelivery(ctx, consumer.Delivery{
			Topic: record.Topic,
			Partition: record.Partition,
			Offset: record.Offset,
		})
		// 业务 Handler 只接收稳定 EventEnvelope，不直接依赖 kgo.Record。
		envelope, err := consumer.DecodeEnvelope(record.Value)
		if err != nil {
			if errors.Is(err, consumer.ErrUnsupportedEnvelopeVersion) {
				return HGKafkaPackage.HGNewTerminalError(err)
			}
			return err
		}
		return handler.Handle(ctx, envelope)
	}
}
```

这是 Kafka adapter：把框架层的 `*kgo.Record` 转成业务层使用的事件信封，并通过 `context.Context` 透传投递元数据。业务 handler 不直接依赖 franz-go 的记录类型。

***
<br/>

> <h2 id="Kafka适配流程">适配流程</h2>

1. 接收业务 `consumer.Handler`，返回框架要求的 `HGRecordHandler`。
2. 将 `Topic`、`Partition`、`Offset` 封装为 `consumer.Delivery` 并写入新的 `ctx`。
3. 使用 `consumer.DecodeEnvelope(record.Value)` 将 Kafka payload 解码为统一事件信封。
4. 解码成功后调用 `handler.Handle(ctx, envelope)`，并原样返回业务错误。

数据流如下：

```text
kgo.Record（Kafka 原始消息）
        ↓ hgDomainEventRecordHandler
往 ctx 写入 Delivery 元数据
record.Value 二进制 → DecodeEnvelope → EventEnvelope
        ↓
handler.Handle(ctx, envelope) 执行业务逻辑
        ↓ 返回 error
上层 consumeBatchLoop 根据 error 类型：
    terminal 错误 → DLQ，跳过消息
    其他错误 → 按上层策略恢复 offset / 重试
```

该适配层提供：

- **依赖隔离**：业务层依赖稳定的 `Envelope` 和 `Delivery`，不直接接触 `kgo.Record`。
- **统一解码入口**：领域事件集中执行 Envelope 解码。
- **元数据透传**：Kafka 位点信息通过 `ctx` 传递给后续 handler。
- **错误分类**：把确定不兼容的协议版本标记为终止型错误。

***
<br/>

> <h2 id="Kafka适配错误分类">错误分类</h2>

```go
if err != nil {
    if errors.Is(err, consumer.ErrUnsupportedEnvelopeVersion) {
        return HGKafkaPackage.HGNewTerminalError(err)
    }
    return err
}
```

- `consumer.ErrUnsupportedEnvelopeVersion`：包装为 terminal error，供上层识别并按 DLQ 等终止策略处理。
- 其他解码错误：直接透传。它们是否持续重试、限次重试或进入 DLQ，最终取决于上层消费循环的错误策略。
- `handler.Handle` 返回的错误不在 adapter 内重新分类，直接交给上层。

损坏 payload 如果始终以普通错误返回，而上层又对普通错误无限重试，可能阻塞对应分区。是否也应将其视为 terminal error，需要依据消息契约和 DLQ 策略决定，不能仅凭“解码失败”统一处理。

该函数处理单条记录；如果上层按 partition 或 batch 工作，通常还需要一层循环适配，逐条调用该 handler。

**结论：**这一层负责 Kafka 记录到领域事件的转换、Delivery 元数据注入和协议错误分类，使业务 handler 与 Kafka 客户端实现解耦。
