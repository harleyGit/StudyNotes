- [franz-go 库](#franz-go-库)
	- [核心模型与安装](#核心模型与安装)
	- [Producer](#Producer)
	- [Consumer](#Consumer)
	- [Offset 与 Rebalance](#Offset与Rebalance)
	- [工程封装与消息设计](#工程封装与消息设计)
	- [生产环境实践](#生产环境实践)
	- [学习路线](#学习路线)

***
<br/><br/><br/>

> <h2 id="franz-go-库">franz-go 库</h2>

`franz-go` 是功能完整的 Kafka 原生 Go 客户端，核心包为 `github.com/twmb/franz-go/pkg/kgo`。它使用纯 Go 实现，不依赖 `librdkafka`，支持 Producer、Consumer Group、事务、幂等生产、SASL/TLS 和压缩等功能。

最核心的对象是：

```go
*kgo.Client
```

一个 `kgo.Client` 既可以作为 Producer，也可以作为 Consumer，并且应该作为 Service 内部的**长生命周期对象**复用。

***
<br/>

> <h3 id="核心模型与安装">核心模型与安装</h3>

## 核心模型

```text
Kafka
  │
  ├── Topic
  │     └── Partition
  │
  └── franz-go
          │
          └── kgo.Client
                │
                ├── Produce       → 异步生产消息
                ├── ProduceSync   → 同步生产消息
                ├── PollRecords   → 拉取消息
                ├── CommitRecords → 提交 offset
                └── Close         → 关闭客户端
```

`kgo.Record` 表示一条 Kafka 消息：

```text
Record
 ├── Topic
 ├── Key
 ├── Value
 ├── Headers
 ├── Partition
 └── Timestamp
```

## 安装

```bash
go get github.com/twmb/franz-go
```

```go
import "github.com/twmb/franz-go/pkg/kgo"
```

***
<br/>

> <h3 id="Producer">Producer</h3>

## 同步生产：ProduceSync

```go
package main

import (
	"context"
	"fmt"

	"github.com/twmb/franz-go/pkg/kgo"
)

func main() {
	client, err := kgo.NewClient(
		kgo.SeedBrokers("localhost:9092"),
	)
	if err != nil {
		panic(err)
	}
	defer client.Close()

	record := &kgo.Record{
		Topic: "video-events",
		Key:   []byte("video-10001"),
		Value: []byte(`{"video_id":10001,"action":"play"}`),
	}

	err = client.ProduceSync(
		context.Background(),
		record,
	).FirstErr()
	if err != nil {
		panic(err)
	}

	fmt.Println("produce success")
}
```

`ProduceSync` 等待 Kafka 返回发送结果后再继续执行：

```text
发送消息
   ↓
等待 Kafka 返回结果
   ↓
成功 / 失败
   ↓
继续执行
```

适合测试、后台任务、管理操作，以及必须立即确认发送结果的场景：

```go
result := client.ProduceSync(ctx, record)
if err := result.FirstErr(); err != nil {
	return err
}
```

## 异步生产：Produce

高吞吐系统通常使用异步 `Produce`：

```go
client.Produce(ctx, record, func(record *kgo.Record, err error) {
	if err != nil {
		fmt.Println("produce failed:", err)
		return
	}

	fmt.Println("produce success")
})
```

```text
业务代码
   │
   │ Produce()
   ↓
franz-go
   │
   ↓
Kafka
   │
   ↓
callback
   │
   ├── err == nil
   └── err != nil
```

`Produce` 将消息交给客户端缓冲并立即返回，最终结果通过回调通知。应用关闭前调用 `client.Close()`，或者按需要使用 `Flush`，确保缓冲消息处理完成。

## Producer 配置

```go
client, err := kgo.NewClient(
	kgo.SeedBrokers(
		"10.0.0.1:9092",
		"10.0.0.2:9092",
		"10.0.0.3:9092",
	),
	kgo.ClientID("video-service"),
	kgo.DefaultProduceTopic("video-events"),
	kgo.ProducerLinger(10*time.Millisecond),
)
```

配置 `DefaultProduceTopic` 后，Record 可以省略 `Topic`：

```go
record := &kgo.Record{
	Key:   []byte("10001"),
	Value: []byte("hello"),
}
```

高并发场景不要默认每条消息都调用 `ProduceSync`：

```text
用户
 ↓
API Gateway
 ↓
Video Service
 ↓
Kafka
```

```text
HTTP Request
      │
      ↓
业务逻辑
      │
      ↓
Produce()
      │
      ↓
Kafka
```

异步生产可以利用 franz-go 内部的 batching、buffering 和 compression。常用调优项包括 `ProducerLinger`、`MaxBufferedRecords`、`MaxBufferedBytes`、压缩方式和批次大小；具体值应根据吞吐、延迟、消息大小和内存限制压测决定。

***
<br/>

> <h3 id="Consumer">Consumer</h3>

## Consumer Group 示例

以下 Consumer 订阅 `video-events`，加入 `video-service` Consumer Group，每次最多返回 100 条消息：

```go
package main

import (
	"context"
	"fmt"

	"github.com/twmb/franz-go/pkg/kgo"
)

func main() {
	client, err := kgo.NewClient(
		kgo.SeedBrokers("localhost:9092"),
		kgo.ConsumerGroup("video-service"),
		kgo.ConsumeTopics("video-events"),
	)
	if err != nil {
		panic(err)
	}
	defer client.Close()

	ctx := context.Background()
	for {
		fetches := client.PollRecords(ctx, 100)

		if ctx.Err() != nil {
			return
		}

		if errs := fetches.Errors(); len(errs) > 0 {
			for _, err := range errs {
				fmt.Println("kafka error:", err)
			}
			continue
		}

		fetches.EachRecord(func(record *kgo.Record) {
			fmt.Printf(
				"topic=%s partition=%d offset=%d key=%s value=%s\n",
				record.Topic,
				record.Partition,
				record.Offset,
				string(record.Key),
				string(record.Value),
			)
		})
	}
}
```

## PollRecords 与 Fetches

```go
fetches := client.PollRecords(ctx, 100)
```

`PollRecords` 等待并拉取消息，参数 `100` 限制本次最多返回 100 条 Record。返回值不是 `[]*kgo.Record`，而是 `kgo.Fetches`：

```text
Fetches
 │
 ├── Fetch
 │     ├── Topic
 │     ├── Partition
 │     └── Records
 │
 ├── Fetch
 │     ├── Topic
 │     ├── Partition
 │     └── Records
 │
 └── Errors
```

一次 Fetch 可能同时包含多个 Topic、多个 Partition 的数据：

```text
topic A
 ├── partition 0
 ├── partition 1
 └── partition 2

topic B
 ├── partition 0
 └── partition 1
```

常用遍历方式：

```go
fetches.EachRecord(func(record *kgo.Record) {
	// 逐条处理
})
```

或：

```go
for _, record := range fetches.Records() {
	// 逐条处理
}
```

## Consumer Group 分区分配

假设 `video-events` 有 4 个 Partition，组内有两个 Consumer：

```text
video-service

Consumer A
 ├── P0
 └── P1

Consumer B
 ├── P2
 └── P3
```

如果 Consumer A 退出，Kafka 会触发 Rebalance，将分区重新分配：

```text
Consumer B
 ├── P0
 ├── P1
 ├── P2
 └── P3
```

同一个 Consumer Group 内，一个 Partition 同一时刻只会分配给一个组成员。Consumer 数量超过 Partition 数量时，多出的 Consumer 会处于空闲状态。

***
<br/>

> <h3 id="Offset与Rebalance">Offset 与 Rebalance</h3>

## Offset 提交

`PollRecords` 只表示消息已经被客户端取回，**不表示业务处理完成**：

```text
PollRecords
    ↓
拿到消息
    ↓
业务处理
    ↓
Commit
    ↓
消费进度被持久化
```

假设 Partition 中存在以下 Offset：

```text
Partition 0

Offset
100
101
102
103
104
```

处理完 `100`、`101`、`102` 后，提交的语义是记录该 Partition 的下一消费位置。成功提交后，Consumer 重启通常从 `103` 继续。

手动提交示例：

```go
fetches := client.PollRecords(ctx, 100)

fetches.EachRecord(func(record *kgo.Record) {
	// 业务处理
})

if err := client.CommitRecords(ctx, fetches.Records()...); err != nil {
	fmt.Println("commit failed:", err)
}
```

`CommitRecords` 会提交各 Partition 中给定 Record 对应的消费进度。不要在较早消息尚未处理完成时提交更高 Offset，否则崩溃后可能跳过未完成的消息。并发处理时尤其要维护每个 Partition 的连续完成边界。

## Rebalance 风险

Consumer 从 Poll 到处理、提交之间可能发生 Rebalance：

```text
Consumer A
    ↓
拿到 P0
    ↓
正在处理
    ↓
发生 Rebalance
    ↓
P0 被分配给 Consumer B
    ↓
Consumer A 仍在处理 P0
```

这可能造成重复处理，或尝试提交已经不再属于当前 Consumer 的 Partition。

启用 `kgo.BlockRebalanceOnPoll()` 后，客户端会在一次 Poll 返回后暂缓 Rebalance，直到下一次 Poll 或显式调用 `AllowRebalance()`。典型结构如下：

```go
fetches := client.PollRecords(ctx, hgConsumerMaxPollRecords)

if ctx.Err() != nil {
	client.AllowRebalance()
	return
}

// 处理并提交本批消息

client.AllowRebalance()
```

`AllowRebalance` 只有配合 `BlockRebalanceOnPoll` 才有意义。阻塞 Rebalance 太久会延迟组内重新分配；如果两次 Poll 的间隔超过 `max.poll.interval.ms`，Consumer 还可能离开消费组。因此业务处理时间、`max.poll.interval.ms` 与批量大小必须协调配置。

***
<br/>

> <h3 id="工程封装与消息设计">工程封装与消息设计</h3>

## 目录结构

不要让 Kafka 调用散落在业务代码中，可以集中放在基础设施层：

```text
internal/
│
├── kafka/
│   ├── client.go
│   ├── producer.go
│   ├── consumer.go
│   ├── config.go
│   └── errors.go
│
├── video/
│   ├── service.go
│   └── handler.go
│
└── ...
```

## Client 与 Producer 封装

```go
type KafkaClient struct {
	client *kgo.Client
}

func NewKafkaClient(cfg Config) (*KafkaClient, error) {
	client, err := kgo.NewClient(
		kgo.SeedBrokers(cfg.Brokers...),
		kgo.ClientID(cfg.ClientID),
	)
	if err != nil {
		return nil, err
	}

	return &KafkaClient{client: client}, nil
}
```

```go
type Producer struct {
	client *kgo.Client
	topic  string
}

func (p *Producer) Produce(
	ctx context.Context,
	key string,
	value []byte,
) error {
	record := &kgo.Record{
		Topic: p.topic,
		Key:   []byte(key),
		Value: value,
	}

	return p.client.ProduceSync(ctx, record).FirstErr()
}
```

业务层只依赖封装后的 Producer：

```go
err := videoProducer.Produce(ctx, videoID, data)
```

实际项目可以定义接口隐藏 `kgo` 类型，便于测试和替换实现。同步还是异步发送，应由业务可靠性和延迟要求决定，而不是固定写死。

## Consumer Service

```go
type Consumer struct {
	client *kgo.Client
}

func (c *Consumer) Run(ctx context.Context) error {
	for {
		fetches := c.client.PollRecords(ctx, 100)

		if ctx.Err() != nil {
			return ctx.Err()
		}

		if errs := fetches.Errors(); len(errs) > 0 {
			for _, err := range errs {
				log.Printf("kafka fetch error: %v", err)
			}
			continue
		}

		for _, record := range fetches.Records() {
			if err := c.handle(ctx, record); err != nil {
				log.Printf(
					"handle failed topic=%s partition=%d offset=%d err=%v",
					record.Topic,
					record.Partition,
					record.Offset,
					err,
				)
				continue
			}
		}
	}
}
```

生产实现还需要明确失败重试、DLQ、Offset 提交、优雅退出和 Rebalance 策略。仅记录错误后继续循环，可能导致自动提交尚未成功处理的消息。

## Kafka Key 与顺序

```go
record := &kgo.Record{
	Topic: "video-events",
	Key:   []byte("video-10001"),
	Value: []byte("play"),
}
```

在默认按 Key 分区的策略下，同一个 Key 通常会进入同一个 Partition：

```text
Partition 3

offset 100 play
offset 101 like
offset 102 coin
offset 103 share
```

Kafka 只保证**单个 Partition 内的顺序**。因此用户、视频、订单或库存等需要局部有序的事件，应选择稳定的业务标识作为 Key。Partition 数量变化时，Key 到 Partition 的映射可能改变，不能将其理解为永久固定在某个具体 Partition。

## Message 与 Headers

大型项目通常使用结构化事件，而不是裸字符串：

```go
type VideoPlayEvent struct {
	EventID string `json:"event_id"`
	UserID  int64  `json:"user_id"`
	VideoID int64  `json:"video_id"`
	Time    int64  `json:"time"`
}

data, err := json.Marshal(event)
if err != nil {
	return err
}

record := &kgo.Record{
	Topic: "video-events",
	Key:   []byte(strconv.FormatInt(event.VideoID, 10)),
	Value: data,
}
```

```json
{
  "event_id": "xxx",
  "user_id": 10001,
  "video_id": 20001,
  "time": 1750000000
}
```

Headers 可以携带事件类型、版本、Trace ID 等元数据：

```go
Headers: []kgo.RecordHeader{
	{
		Key:   "event-type",
		Value: []byte("video.play"),
	},
	{
		Key:   "version",
		Value: []byte("v1"),
	},
}
```

```text
Topic
video-events

Key
video-10001

Value
JSON

Headers
 ├── event-type = video.play
 └── version = v1
```

建议消息包含唯一 `event_id` 和 Schema 版本；Consumer 使用 `event_id` 实现幂等处理，以应对 Kafka 的至少一次投递和业务重试。

***
<br/>

> <h3 id="生产环境实践">生产环境实践</h3>

## Client 生命周期

`kgo.Client` 内部维护连接、元数据、缓冲、批处理和后台协程，应在整个 Service 生命周期内复用：

```go
client, err := kgo.NewClient(...)
if err != nil {
	return err
}
defer client.Close()
```

```text
Service Start
     │
     ↓
NewClient
     │
     ↓
运行几个小时 / 几天
     │
     ↓
Produce / PollRecords
     │
     ↓
Service Shutdown
     │
     ↓
Close
```

不要在每次请求中创建 Client：

```go
func SendMessage() {
	client, _ := kgo.NewClient(...)
	defer client.Close()

	client.Produce(...)
}
```

每个 Service 或具有独立配置、职责和生命周期的组件维护自己的 Client；不要让所有业务无边界地共享一个全局 Client。

## 架构示例

```text
                       ┌──────────────┐
                       │ API Gateway  │
                       └──────┬───────┘
                              │
                              ↓
                    ┌──────────────────┐
                    │ Video Service    │
                    └────────┬─────────┘
                             │
                       franz-go
                             │
                             ↓
                    ┌──────────────────┐
                    │      Kafka       │
                    │                  │
                    │ video-events     │
                    │ comment-events   │
                    │ like-events      │
                    │ follow-events    │
                    └───────┬──────────┘
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
       Video Consumer  Comment Consumer  Statistic
             │              │              │
             ↓              ↓              ↓
           MySQL          MySQL         ClickHouse
             │
             ↓
           Redis
```

```text
video-service
 └── kafka client

comment-service
 └── kafka client

statistic-service
 └── kafka client
```

## 生产级检查项

**Producer：**

- `Produce`、`ProduceSync`、`TryProduce`
- Batch、Linger、Buffer、Compression
- Retry、Timeout、Delivery Callback
- Idempotent Producer、Transactional Producer

**Consumer：**

- Consumer Group、`PollRecords`、`Fetches`
- Auto Commit、Manual Commit、`CommitRecords`
- Rebalance、`BlockRebalanceOnPoll`、`AllowRebalance`
- 并发处理、顺序保证、优雅退出

**容错：**

- Broker 或 Partition Leader 故障
- Consumer Crash、重复消息、消息丢失风险
- 重试、幂等、DLQ、毒消息隔离
- Schema 演进与兼容性

**性能：**

- Batch、Compression、Buffer、Partition
- `FetchMaxBytes`、每次 Poll 的最大 Record 数
- 吞吐、端到端延迟、内存之间的平衡

**可观测性：**

- Produce / Consume latency
- Consumer lag
- Produce / Fetch / Commit error
- Rebalance count
- Buffer 使用量与请求重试次数

***
<br/>

> <h3 id="学习路线">学习路线</h3>

franz-go 的主要调用关系：

```text
                 franz-go

                    Client
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Producer       Consumer       Admin
        │             │
        ↓             ↓
    Produce       PollRecords
    ProduceSync       │
    TryProduce        ↓
                   Fetches
                      │
                      ↓
                   Records
                      │
                      ↓
                  CommitRecords
```

推荐学习顺序：

```text
1. NewClient
      ↓
2. Producer
      ↓
3. Consumer
      ↓
4. Consumer Group
      ↓
5. Offset Commit
      ↓
6. Rebalance
      ↓
7. Retry / DLQ
      ↓
8. Idempotent Producer
      ↓
9. Transaction / EOS
      ↓
10. Consumer Lag + Metrics
      ↓
11. 高并发参数调优
```

重点 API：`kgo.NewClient`、`kgo.Record`、`Produce`、`ProduceSync`、`PollRecords`、`Fetches`、`CommitRecords`、`ConsumerGroup`、Rebalance、Idempotent Producer 和 Transaction。

**核心原则：Producer 要明确发送结果与可靠性策略，Consumer 要明确处理成功、Offset 提交和 Rebalance 之间的边界。**

## 官方资料

- [franz-go GitHub 官方仓库](https://github.com/twmb/franz-go)
- [Producer / Consumer 官方文档](https://github.com/twmb/franz-go/blob/master/docs/producing-and-consuming.md)
- [kgo API 文档](https://pkg.go.dev/github.com/twmb/franz-go/pkg/kgo)
