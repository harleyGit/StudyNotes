- [`kafka.yaml` consumers 配置](#kafka.yaml-consumers配置)


***
<br/><br/><br/>

> <h1 id="kafka.yaml-consumers配置"><code>kafka.yaml</code> consumers 配置</h1>

```yaml
consumers:
  feed:
    enabled: true
    group_id: mlc-go-debug-feed
    client_id: mlc-go-debug-feed
  search:
    enabled: false
    group_id: mlc-go-debug-search
    client_id: mlc-go-debug-search
  statistic:
    enabled: false
    group_id: mlc-go-debug-statistic
    client_id: mlc-go-debug-statistic
  audit:
    enabled: false
    group_id: mlc-go-debug-audit
    client_id: mlc-go-debug-audit
```

该配置定义 `feed`、`search`、`statistic`、`audit` 四组消费者参数。是否会创建四个独立客户端、如何绑定 topic 和 handler，仍取决于配置加载与启动代码；从字段设计看，每项都可拥有独立开关、消费组和客户端标识。

***
<br/>

## 字段含义

- **`enabled`**：应用级启停开关。当前配置只有 `feed` 为 `true`。`false` 是否完全跳过客户端和 goroutine 创建，需要以启动代码实现为准。
- **`group_id`**：Kafka consumer group ID。不同 group 消费同一 topic 时，各自维护消费位点，通常每组都能消费一份消息；同一 group 内的实例则共同分配分区，而不是每个实例都收到全部消息。
- **`client_id`**：客户端标识，会出现在 broker 请求、日志或指标中，便于识别客户端；它可以与 `group_id` 同名，但两者语义不同。

不同业务是否必须使用不同 `group_id`，取决于预期语义：需要各自完整消费同一事件流时应使用不同 group；需要共同分担同一消费任务时应使用同一 group。

***
<br/>

## 各消费者的预期职责

以下职责来自原文的工程背景判断，最终应以项目注释、handler 注册和依赖实现为准：

- **`feed`，已启用**：主信息流消费，可能写入 Redis ZSET 分片，并使用 Kafka partition offset 维护水位。原文提到 64 分片用于降低全局热 key 风险。
- **`search`，未启用**：预期写入 ES/OpenSearch 构建检索索引；启用前需确认 mapping、批量客户端和失败补偿存储已就绪。
- **`statistic`，未启用**：预期执行分片统计并写入 Redis/ClickHouse；启用前需确认对账任务、验收和告警。
- **`audit`，未启用**：预期调用审核服务；启用前需确认超时、重试、幂等键和限流契约。

这些未启用项不宜仅凭 YAML 断言为“代码已完成”或“绝对不能运行”，但原文中的依赖与契约风险值得在启用前核实。

***
<br/>

## 启动流程与隔离性

典型加载逻辑为：

1. 遍历 `consumers` 下的 `feed`、`search`、`statistic`、`audit`。
2. 对 `enabled: true` 的配置初始化 Kafka client，绑定 handler 并启动消费循环。
3. 对 `enabled: false` 的配置跳过启动。

如果每项确实创建独立 client 且使用不同 `group_id`，则它们具有独立的分区分配和已提交位点，一个业务的 lag 或失败通常不会直接阻塞其他 group，但会增加 broker 拉取流量和客户端资源开销。

与“单 consumer 拉取后在进程内多路分发”相比：

- **多 group**：位点、启停和故障域更独立，但同一消息会被多组分别拉取和处理。
- **单 group 内部分发**：可减少重复拉取，但各下游的成功确认、背压和失败补偿更耦合；具体风险取决于并发和提交设计。

***
<br/>

## 生产注意事项

1. 启用新消费者前，确认 topic、handler、依赖服务、重试/DLQ 策略和监控均已就绪。
2. 修改 `group_id` 会让 Kafka 将其视为另一个消费组。新 group 没有旧 group 的已提交位点时，从哪里开始消费取决于 `auto.offset.reset`、显式位点初始化及 broker 状态，可能从 earliest、latest 或直接报错，不能简单等同于“重置 offset”。
3. `mlc-go-debug-*` 看起来是调试环境命名；生产环境的实际命名应以部署配置为准。
4. 启用 `search`、`statistic` 或 `audit` 后，应关注 consumer lag、处理吞吐、错误率和 DLQ；是否消费历史消息取决于该 group 是否已有提交位点及 offset reset 策略。
5. 如果每个配置项创建独立 client，关闭服务时应逐一取消上下文、等待消费循环退出并关闭客户端及相关资源。

可能的整体链路：

```text
kafka.yaml
    ↓ 读取 consumers 配置
按 enabled 创建 HGBaseConsumer / handler
    ↓
feed group       → feed handler
search group     → search handler
statistic group  → statistic handler
audit group      → audit handler
    ↓
shutdown 时 cancel context，并等待各消费循环退出
```

**结论：**`consumers` 配置用于分别控制多类 Kafka 消费任务。不同 `group_id` 提供独立消费进度和故障隔离，但启动行为、业务职责及历史消息起点必须结合代码和 Kafka offset 策略确认。
