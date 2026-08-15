- [Kafka 与 franz-go工程实践](#Kafka与franz-go工程实践)
- [Kafka基础](#Kafka基础) 
- [生产级Go工程设计](#生产级Go工程设计) 
- [CommitRecords与消费位点](#CommitRecords与消费位点) 
- [Producer原子指标](#Producer原子指标)
- [Kafka Broker 介绍](#kafka-broker介绍)
	- [Broker 的职责](#broker的职责)
	- [Broker 核心配置](#broker核心配置)
	- [多 Broker 的价值](#多broker的价值)
	- [Go franz-go 配置](#go-franz-go配置)
- [Kafka Cluster 介绍](#kafka-cluster介绍)
	- [Cluster 的作用](#cluster的作用)
	- [Cluster、Topic 与 Broker 的关系](#cluster-topic与broker的关系)
	- [直播弹幕场景](#直播弹幕场景)
- [Go 微服务中的配置、Kafka 与数据库操作](#Go微服务中的配置Kafka与数据库操作)
	- [服务启动配置初始化](#服务启动配置初始化)
	- [Kafka Client 的 Functional Options](#KafkaClient的FunctionalOptions)
	- [Kafka 健康检查](#Kafka健康检查)
	- [Kafka 同步生产消息](#Kafka同步生产消息)
	- [Kafka 异步生产与 DLQ](#Kafka异步生产与DLQ)
	- [Kafka Producer 优雅关闭](#KafkaProducer优雅关闭)
	- [Kafka acks 确认策略](#Kafkaacks确认策略)
	- [Kafka Consumer 生命周期](#KafkaConsumer生命周期)
	- [PollFetches 拉取与提交消息](#PollFetches拉取与提交消息)
	- [append 构造批量 INSERT](#append构造批量INSERT)
	- [RowsAffected 获取影响行数](#RowsAffected获取影响行数)
- [大厂kafka方案](#大厂kafka方案)
	- [项目目录布局](#项目目录布局)
	- [依赖引入](#依赖引入) 
	- [配置层](#配置层) 
	- [kafka配置工厂](#kafka配置工厂)  
	- [全局生产者单例](#全局生产者单例)  
	- [统一消费基类](#统一消费基类)  
	- [业务消费任务示例](#业务消费任务示例)  
	- [HTTP接口调用示例](#HTTP接口调用示例)  
	- [main入口初始化](#main入口初始化) 
	- [MLC_GO工程改动](#MLC_GO工程改动)



***
<br/><br/><br/>
> <h2 id="Kafka与franz-go工程实践">Kafka 与 franz-go 工程实践</h2>

Kafka 是用于处理**海量消息、异步任务和实时数据流**的分布式事件流平台。核心思想是：**生产者只发布事件，由各下游服务独立消费，以异步方式实现削峰、解耦和水平扩展。**

本文以短视频上传系统为例，介绍 Kafka 基础概念、`franz-go` 生产级工程设计、消费者位点提交，以及 Producer 指标采集。

***
<br/><br/>
> <h3 id="Kafka基础">Kafka 基础</h3>

## Kafka 解决的问题

短视频上传接口可能同时承担数据库写入、缩略图生成、审核、识别、通知、推荐和统计：

```text
用户
 |
上传视频接口
 |
Go 服务
 |
 +---- 保存视频信息 MySQL
 |
 +---- 生成缩略图
 |
 +---- 视频审核
 |
 +---- AI 识别
 |
 +---- 推送通知
 |
 +---- 推荐系统
 |
 +---- 数据统计
```

如果直接同步调用：

```go
func UploadVideo() {
    saveMySQL()
    generateThumbnail()
    aiCheck()
    sendNotification()
    updateRecommend()
}
```

会产生三个主要问题：

- **接口慢：** `MySQL 50ms + 缩略图 3s + AI 审核 5s + 推荐更新 2s`，总耗时超过 `10s`。
- **故障传递：** 某个下游失败，可能导致后续流程全部失败。
- **强耦合：** 每增加搜索、广告或画像服务，都要修改上传服务。

```text
上传接口
   |
   +---- MySQL 成功
   |
   +---- AI 失败
   |
   +---- 后面全部失败
```

引入 Kafka 后，上传服务只保存核心数据并发布 `VideoUploaded` 事件，然后立即返回：

```text
              Kafka
                |
        video_uploaded 事件
                |
 ------------------------------------------------
 |              |             |                 |
审核服务      推荐服务       通知服务          数据分析
```

**设计原则：不直接调用所有下游，而是把已发生的事情发布为事件，让下游自行消费。**

---
<br/>

## Kafka 是什么

Kafka 是一个**高性能、可持久化、可水平扩展的消息日志系统**，可用于：

1. 发布事件。
2. 持久化保存事件。
3. 实时或按需消费事件。

Kafka 不会因某个消费者读取消息就立即删除数据，因此多个独立消费者组可以读取同一事件流。

---
<br/>

## 核心概念

```text
Producer
    |
  Topic
    |
 Partition
    |
 Consumer
```

### Producer

**Producer / 生产者**负责向 Kafka 发送事件。例如上传服务向 `video_events` 发送：

```json
{
  "event": "video_uploaded",
  "video_id": 10001,
  "user_id": 888
}
```

### Topic

**Topic / 主题**是事件的逻辑分类，类似数据库中的表。例如 `video_events` 保存视频相关事件：

```text
topic: video_events

事件1
事件2
事件3
```

### Partition

一个 Topic 可拆成多个 **Partition / 分区**：

```text
video_events

partition-0
partition-1
partition-2
partition-3
```

不同分区可并行读写，从而提高吞吐：

```text
机器1 -> 消费 partition-0
机器2 -> 消费 partition-1
机器3 -> 消费 partition-2
```

### Consumer

**Consumer / 消费者**读取并处理消息。例如 `video-review-service` 消费 `video_uploaded` 后执行审核。

### Consumer Group

同一业务的多个 Consumer 可加入一个 **Consumer Group / 消费者组**：

```text
Group: video-review-group

partition-0 ---> review-worker-1
partition-1 ---> review-worker-2
partition-2 ---> review-worker-3
```

在同一消费者组内，一个分区同一时刻只分配给一个消费者实例；增加实例可水平扩展，但实例数超过分区数后，多余实例不会分到分区。

---
<br/>

## Kafka 与普通 MQ

传统队列常强调消息投递和消费确认：

```text
消息到达 -> 消费 -> 确认
```

Kafka 更接近可持久化的追加日志：

```text
消息到达 -> 顺序写入日志 -> 按保留策略保存 -> 多个消费者组读取
```

同一个 `order_created` 事件可被不同消费者组独立处理：

```text
order_created

消费者 A：库存
消费者 B：支付
消费者 C：推荐
消费者 D：数据分析
```

两者并非简单的优劣关系，应根据路由模型、吞吐、延迟、数据保留和重放需求选择。

---
<br/>

## Kafka 的主要价值

### 高吞吐与持久化

Kafka 通过顺序写磁盘、Page Cache、批处理和分区并行获得高吞吐。消息按保留策略持久化，不依赖消费者在线才能保存。

### 削峰

秒杀等突发流量可先进入 Kafka，再由下游按自身能力消费：

```text
请求
 |
Kafka
 |
消费者按处理能力拉取
 |
数据库
```

### 解耦

同步调用链：

```text
订单服务 -> 支付服务 -> 库存服务 -> 物流服务
```

事件驱动：

```text
订单服务
   |
发送 OrderCreated
   |
 Kafka
   |
支付 / 库存 / 物流
```

---
<br/>

## franz-go 快速入门

Go 常见 Kafka 客户端：

| 库 | 特点 |
| --- | --- |
| `sarama` | 老牌客户端 |
| `segmentio/kafka-go` | API 简单 |
| `confluent-kafka-go` | Confluent 生态，依赖 `librdkafka` |
| `franz-go` | 纯 Go、高性能、功能完整 |

安装：

```bash
go get github.com/twmb/franz-go
```

### 事件与 Producer

```go
package event

type VideoUploaded struct {
    VideoID int64 `json:"video_id"`
    UserID  int64 `json:"user_id"`
}
```

```go
package kafka

import (
    "context"
    "encoding/json"

    "github.com/twmb/franz-go/pkg/kgo"
)

type Producer struct {
    client *kgo.Client
}

func NewProducer() (*Producer, error) {
    client, err := kgo.NewClient(
        kgo.SeedBrokers("localhost:9092"),
    )
    if err != nil {
        return nil, err
    }

    return &Producer{client: client}, nil
}

func (p *Producer) Send(ctx context.Context, topic string, data any) error {
    body, err := json.Marshal(data)
    if err != nil {
        return err
    }

    record := &kgo.Record{
        Topic: topic,
        Value: body,
    }

    return p.client.ProduceSync(ctx, record).FirstErr()
}
```

调用：

```go
event := VideoUploaded{
    VideoID: 10001,
    UserID:  888,
}

if err := producer.Send(ctx, "video_events", event); err != nil {
    return err
}
```

产生的消息：

```json
{
  "video_id": 10001,
  "user_id": 888
}
```

### Consumer

```go
client, err := kgo.NewClient(
    kgo.SeedBrokers("localhost:9092"),
    kgo.ConsumerGroup("video-review-group"),
    kgo.ConsumeTopics("video_events"),
)
```

```go
for {
    fetches := client.PollFetches(ctx)
    if errs := fetches.Errors(); len(errs) > 0 {
        // 记录并按业务策略处理拉取错误
    }

    fetches.EachRecord(func(record *kgo.Record) {
        fmt.Println(string(record.Value))
    })
}
```

输出：

```json
{
  "video_id": 10001,
  "user_id": 888
}
```

---
<br/>

## 适用场景

| 场景 | 是否适合 Kafka |
| --- | --- |
| 日志采集 | 是 |
| 订单事件 | 是 |
| 支付流水 | 是 |
| 视频处理 | 是 |
| IoT 设备数据 | 是 |
| 埋点与实时数据流 | 是 |
| 用户登录、查询信息、修改密码等简单 CRUD | 通常不需要 |

典型位置：

```text
Upload Service
      |
    Kafka
      |
 --------------------
 |        |          |
审核    转码       推荐
Worker  Worker     Worker
```

Kafka 负责系统间的高速事件传输和解耦，不应替代所有同步请求，也不能自动解决业务幂等、数据库一致性和错误恢复问题。

***
<br/><br/>
> <h3 id="生产级Go工程设计">生产级 Go 工程设计</h3>

生产级 Kafka 服务通常需要处理：水平扩容、可靠生产、消费重试、死信队列、幂等、优雅关闭、监控和链路追踪。

## 整体架构

```text
                  API Gateway
                       |
                video-service
                       |
                   Producer
                       |
                 Kafka Cluster
             video.events.uploaded
                       |
     --------------------------------
     |              |               |
 review service  transcode service  recommend service
     |
 MySQL / Redis / ES
```

完整技术栈可包含：

```text
Go
 |
 |-- franz-go
 |-- Kafka Cluster
 |-- MySQL / Redis / ES
 |-- Prometheus / Grafana
 |-- OpenTelemetry
```

---
<br/>

## 工程目录

```text
video-service
├── cmd
│   └── server
│       └── main.go
├── internal
│   ├── kafka
│   │   ├── client.go
│   │   ├── config.go
│   │   ├── producer.go
│   │   ├── consumer.go
│   │   ├── handler.go
│   │   ├── retry.go
│   │   ├── dlq.go
│   │   └── middleware.go
│   ├── event
│   │   ├── event.go
│   │   ├── video_uploaded.go
│   │   └── order_created.go
│   ├── service
│   │   └── video_service.go
│   ├── repository
│   │   └── mysql.go
│   └── config
│       └── config.go
├── pkg
│   ├── logger
│   ├── trace
│   └── metrics
└── go.mod
```

Kafka 基础设施、事件定义和业务处理应分离，避免 Handler 与具体客户端初始化逻辑强耦合。

---
<br/>

## 配置与 Client

`internal/kafka/config.go`：

```go
package kafka

type Config struct {
    Brokers  []string
    ClientID string
    GroupID  string
}
```

生产配置：

```yaml
kafka:
  brokers:
    - kafka1:9092
    - kafka2:9092
    - kafka3:9092
  client_id: video-service
  group_id: video-review-group
```

`client.go`：

```go
package kafka

import "github.com/twmb/franz-go/pkg/kgo"

func NewProducerClient(cfg Config) (*kgo.Client, error) {
    return kgo.NewClient(
        kgo.SeedBrokers(cfg.Brokers...),
        kgo.ClientID(cfg.ClientID),
    )
}
```

Producer 和 Consumer 通常使用不同配置创建独立 Client，Consumer 还需配置 `ConsumerGroup`、`ConsumeTopics` 和提交策略。

---
<br/>

## 事件模型

生产系统不应只发送缺少上下文的业务对象：

```json
{
  "id": 10001
}
```

可使用 Event Envelope 统一事件元数据：

```go
package event

type Envelope struct {
    EventID   string `json:"event_id"`
    EventType string `json:"event_type"`
    Version   int    `json:"version"`
    Timestamp int64  `json:"timestamp"`
    Data      any    `json:"data"`
}
```

```go
package event

type VideoUploaded struct {
    VideoID int64  `json:"video_id"`
    UserID  int64  `json:"user_id"`
    URL     string `json:"url"`
}
```

最终消息：

```json
{
  "event_id": "a8d9f",
  "event_type": "VIDEO_UPLOADED",
  "version": 1,
  "timestamp": 178000000,
  "data": {
    "video_id": 10001,
    "user_id": 888
  }
}
```

`event_id` 用于幂等与追踪，`event_type` 用于路由，`version` 用于 Schema 演进，`timestamp` 表示事件发生时间。

---
<br/>

## Producer 封装

```go
package kafka

import (
    "context"
    "encoding/json"

    "github.com/twmb/franz-go/pkg/kgo"
)

type Producer struct {
    client *kgo.Client
}

func NewProducer(client *kgo.Client) *Producer {
    return &Producer{client: client}
}

func (p *Producer) Publish(ctx context.Context, topic string, msg any) error {
    data, err := json.Marshal(msg)
    if err != nil {
        return err
    }

    record := &kgo.Record{
        Topic: topic,
        Value: data,
    }

    return p.client.ProduceSync(ctx, record).FirstErr()
}
```

```go
err := producer.Publish(
    ctx,
    "video.events",
    event.Envelope{
        EventID:   "a8d9f",
        EventType: "VIDEO_UPLOADED",
        Version:   1,
        Timestamp: time.Now().Unix(),
        Data:      video,
    },
)
```

同步生产便于理解和确认结果；高吞吐场景常使用异步 `Produce`，并在回调中处理投递结果。

---
<br/>

## Consumer 与 Worker Pool

典型分层：

```text
Kafka Consumer
      |
    Fetch
      |
 Worker Pool
      |
   Handler
```

Consumer 和 Handler 接口：

```go
package kafka

import (
    "context"

    "github.com/twmb/franz-go/pkg/kgo"
)

type Handler interface {
    Handle(context.Context, *kgo.Record) error
}

type Consumer struct {
    client  *kgo.Client
    workers int
    handler Handler
}
```

原始示意代码：

```go
func (c *Consumer) Start(ctx context.Context) {
    for {
        fetches := c.client.PollFetches(ctx)

        fetches.EachRecord(func(r *kgo.Record) {
            go c.process(ctx, r)
        })
    }
}

func (c *Consumer) process(ctx context.Context, r *kgo.Record) {
    if err := c.handler.Handle(ctx, r); err != nil {
        // retry
        return
    }
}
```

**注意：不能简单地为每条消息无限创建 goroutine。**生产实现需要有界 Worker Pool、背压、分区内顺序控制、Rebalance 协调和明确的 Offset 提交策略。

---
<br/>

## 消费幂等

Kafka 常用 **At Least Once / 至少一次**语义：业务处理成功后，如果 Offset 提交失败，消息可能再次投递。

```text
消息到达
  |
业务处理成功
  |
Offset 提交失败
  |
Kafka 再次投递
  |
重复消费
```

可用 `event_id` 建立唯一约束：

```sql
CREATE TABLE video_event_log
(
    event_id  varchar(64),
    created_at datetime,
    PRIMARY KEY (event_id)
);
```

概念代码：

```go
func Handle(event Event) error {
    if db.Exists(event.EventID) {
        return nil
    }

    if err := process(); err != nil {
        return err
    }

    return insertEventID(event.EventID)
}
```

业务变更与幂等记录最好放在同一数据库事务内，否则 `process()` 成功而 `insertEventID()` 失败时仍可能重复执行。

---
<br/>

## Retry Topic 与 DLQ

不应在消费线程中无限 `sleep` 重试。可使用分级 Retry Topic：

```text
video.events
     |
   失败
     |
video.events.retry.5s
     |
video.events.retry.1m
     |
video.events.dlq
```

Topic 示例：

```text
video.uploaded
video.uploaded.retry.30s
video.uploaded.retry.5m
video.uploaded.dlq
```

无法继续处理的 JSON 错误、非法数据或不可恢复业务异常进入 DLQ：

```go
func SendDLQ(ctx context.Context, producer *kgo.Client, record *kgo.Record, err error) error {
    newRecord := &kgo.Record{
        Topic: "video.events.dlq",
        Value: record.Value,
        Headers: []kgo.RecordHeader{
            {
                Key:   "error",
                Value: []byte(err.Error()),
            },
        },
    }

    return producer.ProduceSync(ctx, newRecord).FirstErr()
}
```

DLQ 还应保留原 Topic、Partition、Offset、事件 ID、重试次数和时间，便于定位与回放。

---
<br/>

## Topic 与 Partition 设计

Topic 可按 `业务.事件.版本` 命名：

```text
video.lifecycle.v1
video.upload.v1
video.review.v1
video.transcode.v1
video.dlq.v1
```

假设每天有 `10 亿`条视频事件：

```text
一天 86400 秒
10 亿 / 86400 ≈ 11500 条/秒
```

若配置 `64` 个 Partition，平均约为：

```text
11500 / 64 ≈ 180 条/秒/Partition
```

这只是平均吞吐估算。实际分区数还要考虑峰值流量、消息大小、处理耗时、消费者并行度、Broker 数量、Key 分布、顺序要求和未来扩容成本。

---
<br/>

## Producer 生产配置

原笔记中的生产参数示意：

```go
kgo.RecordDeliveryTimeout(30 * time.Second)
kgo.RequiredAcks(kgo.AllISRAcks())
kgo.RecordRetries(5)
kgo.Compression(kgo.SCompressionCodec)
```

目标包括可靠确认、失败重试和压缩。具体选项名称应以当前 `franz-go` 版本 API 为准，并结合吞吐、延迟、幂等生产和 Broker 配置测试。

---
<br/>

## 启动与完整数据流

```go
func main() {
    cfg := kafka.Config{
        Brokers: []string{
            "kafka1:9092",
            "kafka2:9092",
        },
        ClientID: "video-service",
    }

    client, err := kafka.NewProducerClient(cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer client.Close()

    producer := kafka.NewProducer(client)
    service := NewVideoService(producer)
    service.Start()
}
```

```text
POST /video/upload
        |
  video-service
        |
MySQL: video status=uploaded
        |
 Kafka Producer
        |
Topic: video.upload.v1
        |
---------------------
|          |        |
审核       转码     推荐
        |
     处理成功
        |
发送 video.review.completed
```

---
<br/>

## 生产系统补充能力

- **Schema Registry：** 使用 Protobuf、Avro 或 JSON Schema 管理消息契约与兼容性。
- **数据库与消息一致性：** 常用 Transactional Outbox，而不是把普通数据库事务和 Kafka 发送直接视为同一事务。
- **链路追踪：** 在 Header 或 Envelope 中传递 `trace_id`、`request_id`、`user_id`、`event_id`。
- **监控：** 关注 `consumer lag`、生产延迟、错误率、吞吐、分区热点和 Rebalance。
- **优雅关闭：** 停止接收新任务、等待在途任务结束、提交安全位点，再关闭 Client。

推荐组合示例：

```text
Kafka:       3.x
Go Client:   franz-go
消息格式:    Protobuf
序列化:      google.golang.org/protobuf
配置:        viper
日志:        zap
指标:        Prometheus
链路:        OpenTelemetry
数据库:      MySQL + Redis
```

最终架构：

```text
                 API
                  |
             Go Service
                  |
               Kafka
                  |
 ------------------------------------------------
 |              |              |                |
审核          转码           推荐             数据
Worker        Worker         Worker           Pipeline
                  |
          MySQL / Redis / ES
```

***
<br/>

> <h3 id="CommitRecords与消费位点">CommitRecords 与消费位点</h3>

核心代码：

```go
if err := handler.Handle(ctx, record); err != nil {
    // 业务失败，不推进消费位点。
    return
}

if err := b.cli.CommitRecords(ctx, record); err != nil {
    log.Error(err)
}
```

`CommitRecords` 用于在业务处理成功后，为当前 Consumer Group 提交消费位点。它本质是**消费成功确认机制**，直接影响消息丢失、重复消费和系统可靠性。

## Offset 是什么

Kafka 为每个 Topic Partition 中的消息分配递增 Offset：

```text
Topic: video_events

Partition 0:
offset=0  消息 A
offset=1  消息 B
offset=2  消息 C
offset=3  消息 D
```

Consumer Group 提交位点后，Kafka 能知道该组下次应从哪里继续消费。提交 `offset=100` 对应的 Record，通常意味着下一次从 `101` 继续。

---
<br/>

## 完整消费流程

### 拉取消息

```go
fetches := b.cli.PollFetches(ctx)
```

Record 包含：

```text
topic:     order.created
partition: 3
offset:    200
value:     {"order_id":10001}
```

### 业务处理并提交

```go
fetches.EachRecord(func(record *kgo.Record) {
    if err := handler.Handle(ctx, record); err != nil {
        // 不提交，交给重试策略处理。
        return
    }

    // 成功后再提交。
    if err := b.cli.CommitRecords(ctx, record); err != nil {
        log.Error(err)
    }
})
```

```text
Kafka
  |
PollFetches
  |
Record
  |
Handler 业务处理
  |
成功
  |
CommitRecords
  |
Kafka 保存 Consumer Group 位点
```

---
<br/>

## 为什么不能先提交

错误顺序：

```go
for {
    record := Poll()
    CommitRecords(record)
    handle(record)
}
```

```text
收到消息
  |
提交 Offset
  |
处理业务
  |
业务失败
```

Kafka 已认为位点推进成功，但业务实际未完成，重启后可能跳过该消息，形成消息丢失。

正确顺序：

```text
Kafka -> PollFetches -> 业务处理 -> 成功 -> CommitRecords
```

---
<br/>

## Commit 失败与幂等

如果业务成功而 Commit 失败，Kafka 可能再次投递同一消息：

```text
订单创建成功
  |
CommitRecords 失败
  |
Kafka 认为尚未处理
  |
再次投递
  |
订单可能重复创建
```

因此消费者必须幂等：

```sql
CREATE TABLE consumer_message_log
(
    message_id varchar(64),
    PRIMARY KEY (message_id)
);
```

```go
func Handle(record *kgo.Record) error {
    id := messageID(record)
    if exists(id) {
        return nil
    }

    if err := createOrder(); err != nil {
        return err
    }

    return saveMessageID(id)
}
```

订单写入和 `message_id` 记录应尽量处于同一数据库事务中。

---
<br/>

## 自动提交与手动提交

| 方式 | 特点 | 风险与成本 |
| --- | --- | --- |
| 自动提交 | 配置简单，客户端周期性提交 | 提交时机可能早于业务完成 |
| 手动提交 | 业务成功后显式提交 | 控制更精确，但需处理失败、顺序和 Rebalance |

如果使用显式 `CommitRecords`，通常应同时配置关闭自动提交，避免两套策略混用。具体配置以当前 `franz-go` 版本为准。

---
<br/>

## 批量与并发提交

逐条同步提交：

```go
CommitRecords(ctx, record)
```

实现简单，但高吞吐场景下开销较大。可在保证处理成功和分区位点连续性的前提下批量提交：

```go
records := []*kgo.Record{
    record1,
    record2,
    record3,
}

if err := client.CommitRecords(ctx, records...); err != nil {
    return err
}
```

```text
Kafka Consumer
      |
    Batch
      |
 Worker Pool
      |
Business Handler
      |
   Success
      |
Offset Manager
      |
    Commit
```

需要特别注意：并发处理同一分区的消息时，后面的 Offset 先完成并提交，可能跨过前面尚未成功的消息。生产实现应按 Partition 管理完成位点，或采用保持分区顺序的处理模型。

---
<br/>

## `CommitRecords` 的本质

方法签名：

```go
func (c *Client) CommitRecords(
    ctx context.Context,
    records ...*Record,
) error
```

Record 提供：

```go
record.Topic
record.Partition
record.Offset
```

提交内容可理解为：

```text
Group:     video-review-group
Topic:     video.events
Partition: 2
Offset:    500
```

**`CommitRecords` 告诉 Kafka：当前 Consumer Group 已完成这些 Record，可以推进对应分区的消费位点。**提交成功不代表业务天然“Exactly Once”，消费者仍需幂等和一致性设计。

***
<br/>

> <h3 id="Producer原子指标">Producer 原子指标</h3>

核心代码：

```go
var (
    hgKafkaBufferedRecords atomic.Uint64
    hgKafkaWrittenBatches  atomic.Uint64
)

func HGMetricsSnapshot() (
    bufferedRecords uint64,
    writtenBatches uint64,
) {
    return hgKafkaBufferedRecords.Load(),
        hgKafkaWrittenBatches.Load()
}
```

这两个原子计数器用于采集 Producer 外层业务指标，并向 Prometheus、Grafana 或诊断接口提供线程安全的快照。

## 指标含义

| 变量 | 含义 |
| --- | --- |
| `hgKafkaBufferedRecords` | 当前已进入 Producer、尚未完成投递的 Record 数量 |
| `hgKafkaWrittenBatches` | 已成功写入 Kafka 的 Batch 数量 |

Producer 通常先在内存中聚合消息，再按 Batch 发送：

```text
业务代码
   |
Produce()
   |
Kafka Client 内存 Buffer
   |
Batch
   |
Kafka Broker
```

```text
Batch:
record1
record2
record3
record4
   |
一次发送 Kafka
```

`writtenBatches=100` 表示已成功写入 `100` 个批次，不等于写入了 `100` 条消息。该指标必须在确实能观察 Batch 写入结果的 Hook 或统计位置更新，不能用每条 Record 的成功回调冒充 Batch 数。

---
<br/>

## 为什么使用 `atomic.Uint64`

多个 goroutine 可能同时调用 Producer：

```text
goroutine 1 -> Produce()
goroutine 2 -> Produce()
goroutine 3 -> Produce()
```

普通 `count++` 实际包含“读取、加一、写回”三个步骤，可能丢失更新：

```text
初始 count = 10

线程 A 读取 10
线程 B 读取 10
线程 A 写入 11
线程 B 写入 11

期望结果：12
实际结果：11
```

这就是 Race Condition / 竞态。`atomic.Uint64` 使用原子操作提供并发安全的 `Add` 和 `Load`。

---
<br/>

## `Add` 与 `Load`

增加计数：

```go
hgKafkaBufferedRecords.Add(1)
```

读取快照：

```go
buffered := hgKafkaBufferedRecords.Load()
```

对 `atomic.Uint64` 减一可使用补码方式：

```go
hgKafkaBufferedRecords.Add(^uint64(0))
```

由于无符号整数下溢会变成极大值，必须保证只在对应增量已发生时递减。若业务需要频繁增减并表达负值，使用 `atomic.Int64` 通常更直观。

---
<br/>

## Producer 中的更新流程

概念流程：

```text
业务线程
   |
Produce(record)
   |
bufferedRecords.Add(1)
   |
内存 Buffer
   |
Batch 发送
   |
Kafka Broker
   |
投递完成
   |
bufferedRecords 减少
writtenBatches.Add(1)
```

异步生产示例：

```go
func Produce(ctx context.Context, client *kgo.Client, record *kgo.Record) {
    hgKafkaBufferedRecords.Add(1)

    client.Produce(ctx, record, func(_ *kgo.Record, err error) {
        hgKafkaBufferedRecords.Add(^uint64(0))

        if err != nil {
            // 记录 produceErrors，不增加成功指标。
            return
        }

        // 这里能准确统计成功 Record；Batch 数应通过 franz-go Hook 采集。
    })
}
```

如果 `hgKafkaWrittenBatches` 的定义确实是 Batch 数，应在 franz-go 请求或 Batch 完成 Hook 中更新：

```go
func onBatchWritten() {
    hgKafkaWrittenBatches.Add(1)
}
```

---
<br/>

## 指标快照

```go
buffered, written := HGMetricsSnapshot()
```

返回示例：

```json
{
  "bufferedRecords": 500,
  "writtenBatches": 10000
}
```

`Load()` 能获得某一时刻的原子值，但两个连续 `Load()` 并不是跨变量事务快照；用于监控通常足够，不能据此建立要求严格一致性的业务判断。

---
<br/>

## 监控用途

### Producer 堵塞

正常波动：

```text
bufferedRecords: 0 -> 10 -> 20 -> 30 -> 0
```

持续增长：

```text
bufferedRecords: 1000 -> 5000 -> 10000 -> 50000
```

可能原因包括 Broker 压力、网络异常、生产速度过快、分区热点、重试积压或配置不合理。它表示 Producer 待投递消息增长，不是“Kafka 消费速度慢”的直接证据。

### Grafana 与告警

```text
hg_kafka_buffered_records
      |
      0 ---- 100 ---- 5000
```

告警示例：

```text
hg_kafka_buffered_records > 10000 持续 5 分钟
```

Batch 吞吐：

```text
rate(hg_kafka_written_batches_total[5m])

示例：500 batch/s
```

---
<br/>

## 建议补充的指标

- `produced_messages_total`：生产消息总数。
- `produce_errors_total`：生产失败数。
- `produce_latency_seconds`：生产延迟与 P99。
- `produced_bytes_total`：发送字节数。
- `retries_total`：重试次数。
- `dlq_messages_total`：进入 DLQ 的消息数。
- `consumer_lag`：消费者滞后量，属于 Consumer 侧指标。

这些指标通常放在：

```text
internal/kafka/metrics.go
```

```text
业务代码
   |
Metrics Wrapper / franz-go Hooks
   |
Producer Client
   |
Kafka Broker
```

**原子计数器解决并发安全；准确的指标定义、采集位置和告警阈值，决定这些数据是否真正可用于性能监控、容量评估和故障定位。**
	
	
***
<br/><br/><br/>
> <h2 id="kafka-broker介绍">Kafka Broker 介绍</h2>

**Broker = 一台运行 Kafka 服务的服务器节点，也可以理解为一个 Kafka Server 进程。**

Kafka 集群通常由多个 Broker 组成。每个 Broker 负责存储消息、接收 Producer 写入、向 Consumer 提供数据，并通过唯一的 `broker.id` 区分节点。

```text
                Producer
                    |
                    |
              Kafka Cluster
                    |
     --------------------------------
     |              |               |
  Broker-1       Broker-2        Broker-3
  id=1           id=2            id=3
     |              |               |
 Topic A        Topic A         Topic A
 Partitions     Partitions      Partitions

                    |
                 Consumer
```

例如，一个 Kafka 集群可以包含以下三个 Broker：

```text
Kafka Cluster

broker-1  192.168.1.10:9092
broker-2  192.168.1.11:9092
broker-3  192.168.1.12:9092
```

- **Kafka Cluster**：对外提供服务的集群整体。
- **Broker**：集群中的一个 Kafka 节点。
- **Topic Partition**：实际分布并存储在不同 Broker 上。

***
<br/>

> <h3 id="broker的职责">Broker 的职责</h3>

### 存储消息

Kafka 消息最终以 Partition 日志的形式存储在 Broker 磁盘中。假设创建以下 Topic：

```text
topic = user_action
partition = 6
replication-factor = 3
```

6 个 Partition 可能分布如下：

```text
Broker-1
 └── user_action-0
 └── user_action-3

Broker-2
 └── user_action-1
 └── user_action-4

Broker-3
 └── user_action-2
 └── user_action-5
```

每个 Partition 本质上都是一组追加写入的日志文件。

---
<br/>

### 接收 Producer 写入

例如 Go 服务向 `room_message` 发送消息：

```go
producer.SendMessage(
    topic="room_message",
    value="hello"
)
```

```text
Go Gateway Service
        |
        v
Kafka Producer
        |
        v
Broker-2
        |
        v
写入 partition log
```

Broker 接收消息后，将其追加到对应的 Partition 日志。

---
<br/>

### 向 Consumer 提供数据

以直播弹幕为例：

```text
用户发送弹幕
        |
        v
gateway-service
        |
        v
Kafka topic:
live_room_message
        |
        v
room-service consumer
        |
        v
广播给 WebSocket 用户
```

Consumer 连接对应 Broker，并从 Partition 中读取消息：

```text
Consumer
   |
   v
Broker
   |
   v
读取 partition
```

***
<br/>

> <h3 id="broker核心配置">Broker 核心配置</h3>

典型的 `server.properties` 配置如下：

```properties
broker.id=1
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://192.168.1.10:9092
log.dirs=/data/kafka/logs
```

### `broker.id`

`broker.id` 是 Broker 的唯一标识，集群通过它区分不同节点，配置值不能重复：

```properties
# broker1
broker.id=1

# broker2
broker.id=2

# broker3
broker.id=3
```

> 新版 KRaft 模式通常使用 `node.id` 标识节点；`broker.id` 常见于 ZooKeeper 模式及旧版配置。

---
<br/>

### `listeners`

`listeners` 指定 Kafka 实际绑定并监听的网络地址：

```properties
listeners=PLAINTEXT://0.0.0.0:9092
```

```text
所有网卡
   |
   v
9092 端口
   |
   v
Kafka Broker
```

此时客户端可以通过 Broker 的可达 IP 和 `9092` 端口建立初始连接，例如：

```text
192.168.1.10:9092
```

---
<br/>

### `advertised.listeners`

`advertised.listeners` 是 Broker 向客户端公布的访问地址。在云环境、容器或 NAT 环境中，该地址可能与实际监听地址不同。

```properties
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://kafka01.xxx.com:9092
```

- `listeners`：Kafka 服务实际绑定哪个 IP 和端口。
- `advertised.listeners`：客户端后续应该通过哪个地址连接该 Broker。

客户端首先通过入口地址连接：

```properties
bootstrap.servers=kafka01.xxx.com:9092
```

Broker 返回集群元数据，其中可能包含：

```text
partition leader:

broker2:
10.0.0.5:9092
```

客户端随后会直接连接对应的 Leader Broker。因此，`advertised.listeners` 必须是客户端实际可访问的地址，否则初始连接可能成功，但后续生产或消费仍会失败。

***
<br/>

> <h3 id="多broker的价值">多 Broker 的价值</h3>

### 扩展吞吐量

假设一个 Broker 的写入能力为：

```text
10 万 msg/s
```

三个 Broker 理论上可并行处理：

```text
30 万 msg/s
```

原因是 Topic 的 Partition 可以分散到不同 Broker：

```text
Topic:

partition0 --> broker1
partition1 --> broker2
partition2 --> broker3
```

实际吞吐量还会受磁盘、网络、副本同步、消息大小和客户端配置等因素影响，并不一定严格按 Broker 数量线性增长。

---
<br/>

### 提供高可用

Kafka 通过 Leader 和 Replica 保存 Partition 副本：

```text
Topic:
payment

partition 0:

Leader:
Broker-1

Replica:
Broker-2
Broker-3
```

正常情况下，Producer 向 Leader 写入：

```text
Producer
   |
   v
Broker-1
```

如果 Broker-1 宕机，符合条件的 Replica 可被选为新 Leader：

```text
Broker-1 ❌

Broker-2
成为 Leader

Producer 继续写
```

故障切换是否无数据丢失，还取决于 `acks`、ISR、副本数、`min.insync.replicas` 等配置。

***
<br/>

## Broker 在直播弹幕系统中的应用

整体链路：

```text
用户
 |
WebSocket
 |
gateway-service
 |
Kafka
 |
room-service
 |
广播
 |
百万用户
```

`live_message` 的 Partition 可以分布到多个 Broker：

```text
Kafka Cluster

Broker-1
 |
 |-- live_message partition0
 |-- live_message partition3

Broker-2
 |
 |-- live_message partition1
 |-- live_message partition4

Broker-3
 |
 |-- live_message partition2
 |-- live_message partition5
```

例如，面向大量直播间的 Topic 可以按容量规划设置：

```text
Topic: live_room_message
partition=1000
replication=3
```

Leader 可能分布如下：

```text
partition0
    leader broker1

partition1
    leader broker2

partition2
    leader broker3
```

这样可以分散写入压力、并行消费，并在 Broker 故障后重新选举 Leader。Partition 数量不能只按直播间数量机械设置，还需要结合吞吐量、Consumer 并行度、Broker 数量和运维成本评估。

***
<br/>

> <h3 id="go-franz-go配置">Go franz-go 配置</h3>

使用 `franz-go` 创建客户端时，通过 `kgo.SeedBrokers` 配置一个或多个集群入口：

```go
kgo.NewClient(
    kgo.SeedBrokers(
        "localhost:9092",
    ),
)
```

生产环境通常配置多个 Seed Broker，避免单个入口不可用：

```go
kgo.SeedBrokers(
    "kafka01:9092",
    "kafka02:9092",
    "kafka03:9092",
)
```

`SeedBrokers` 只是客户端发现集群的初始入口，并不表示客户端只连接这些 Broker：

```text
启动:
client
 |
连接任意 broker
 |
获取整个 cluster metadata

得到:
broker1
broker2
broker3
partition leader
```

获取元数据后，客户端会根据 Partition Leader 自动连接正确的 Broker。

---
<br/>

### 生产级配置示例

假设一个三节点 Kafka 集群：

```text
Kafka Cluster

broker-1
CPU 32 核
Memory 128G
Disk NVMe 4T
IP: 10.0.0.1

broker-2
IP: 10.0.0.2

broker-3
IP: 10.0.0.3
```

各 Broker 分别配置自己的唯一 ID 和对外地址：

```properties
# broker1
broker.id=1
advertised.listeners=PLAINTEXT://10.0.0.1:9092

# broker2
broker.id=2
advertised.listeners=PLAINTEXT://10.0.0.2:9092

# broker3
broker.id=3
advertised.listeners=PLAINTEXT://10.0.0.3:9092
```

Go 客户端配置多个入口：

```go
kgo.SeedBrokers(
    "10.0.0.1:9092",
    "10.0.0.2:9092",
    "10.0.0.3:9092",
)
```

---
<br/>

### Broker 总结

**Kafka Broker 就是一个 Kafka 节点，是 Kafka 水平扩展和高可用的基础。**

| 功能 | 说明 |
| --- | --- |
| 存储消息 | 保存 Partition 日志 |
| 接收生产 | 接收 Producer 写入 |
| 提供消费 | 向 Consumer 提供消息 |
| 参与选主 | 承担 Leader 或 Replica 角色 |
| 水平扩展 | 多 Broker 分散存储和吞吐压力 |
| 故障恢复 | Broker 宕机后重新选举 Leader |

```text
gateway-service
        |
      Kafka
        |
 -----------------
 |       |       |
broker1 broker2 broker3
        |
room-service
        |
WebSocket 广播
```

***
<br/><br/><br/>

> <h2 id="kafka-cluster介绍">Kafka Cluster 介绍</h2>

**Kafka Cluster = 多个 Kafka Broker 组成的整体，对外提供统一的消息存储、生产和消费服务。**

假设三台服务器分别运行一个 Kafka 进程：

```text
服务器 A
运行 Kafka
broker.id=1

服务器 B
运行 Kafka
broker.id=2

服务器 C
运行 Kafka
broker.id=3
```

它们共同组成 Kafka Cluster：

```text
             Kafka Cluster

        +----------------+
        |                |
        |   Kafka 集群   |
        |                |
        +----------------+
          |      |      |
          |      |      |
       Broker1 Broker2 Broker3
```

其中，整体叫 **Kafka Cluster**，每个节点叫 **Kafka Broker**。

***
<br/>

> <h3 id="cluster的作用">Cluster 的作用</h3>

### 扩展消息存储能力

单台 Broker 的磁盘容量有限：

```text
一个 Broker:

磁盘: 4TB
每天消息: 2TB
最多存: 2 天
```

三台 Broker 可以提供更大的总存储空间：

```text
Kafka Cluster

Broker1
4TB

Broker2
4TB

Broker3
4TB

总容量:
12TB
```

例如，创建一个包含 6 个 Partition 的 Topic：

```text
topic: video_comment
partition=6
```

Kafka 可以将 Partition 分散存储：

```text
Kafka Cluster

Broker1
---------
partition-0
partition-3

Broker2
---------
partition-1
partition-4

Broker3
---------
partition-2
partition-5
```

注意：启用副本后，同一条数据会占用多个 Broker 的磁盘，因此可用业务容量不是所有磁盘容量的简单相加。

---
<br/>

### 提升吞吐量

假设单个 Broker 的写入能力约为：

```text
10 万消息/秒
```

三个 Broker 可以并行处理不同 Partition：

```text
Broker1
10 万/s

Broker2
10 万/s

Broker3
10 万/s

理论总吞吐:
30 万消息/s
```

例如直播弹幕 Topic：

```text
100 万个用户发送弹幕
        |
        v
Kafka Topic
live_message

partition0 ---> Broker1
partition1 ---> Broker2
partition2 ---> Broker3
partition3 ---> Broker1
...
```

多个 Broker 同时处理不同 Partition，从而提高并发能力。

---
<br/>

### 提供高可用

单节点宕机会导致整个 Kafka 服务不可用：

```text
Kafka Server
      |
     挂了 ❌
      |
所有服务停止
```

集群通过副本机制降低单点故障风险：

```text
        Kafka Cluster

       Topic:
     order_event

Partition 0

Leader
Broker1

Replica
Broker2
Broker3
```

正常情况下由 Broker1 提供读写：

```text
Producer
   |
Broker1
```

Broker1 宕机后，可由 Broker2 成为新 Leader：

```text
Broker1 ❌

Broker2
成为 Leader

继续提供服务
```

---
<br/>

### 隐藏底层复杂性

业务服务通常不需要直接管理以下信息：

```text
消息在哪台机器
Partition 在哪里
Leader 是谁
```

业务代码只需要指定 Topic 和消息：

```go
producer.SendMessage(
    "live_room_message",
    "hello"
)
```

Kafka Cluster 根据元数据完成路由和副本同步：

```text
找到 Partition
        ↓
找到 Leader Broker
        ↓
写入
        ↓
同步副本
```

***
<br/>

> <h3 id="cluster-topic与broker的关系">Cluster、Topic 与 Broker 的关系</h3>

```text
Kafka Cluster
       |
     Topic
       |
   Partitions
       |
   Replicas
       |
    Brokers
```

- **Cluster**：管理多个 Broker，对外提供统一服务。
- **Topic**：消息的逻辑分类。
- **Partition**：Topic 的分片，是并行读写和数据分布的基本单位。
- **Replica**：Partition 的副本，用于故障恢复。
- **Broker**：实际存储 Partition 数据并处理请求的节点。

例如 `user_behavior` Topic 包含 4 个 Partition：

```text
Kafka Cluster

Topic:
user_behavior

Partition:
0
1
2
3

分布：
Broker1:
 partition0
 partition2

Broker2:
 partition1
 partition3

Broker3:
 replica 备份
```

这里的示意图仅用于说明分布关系。生产环境中，每个 Partition 的 Replica 应分散到不同 Broker，Leader 也应尽量均衡分布。

***
<br/>

> <h3 id="直播弹幕场景">直播弹幕场景</h3>

在直播弹幕架构中，Kafka Cluster 位于 WebSocket Gateway 和下游 Room Service 之间：

```text
千万用户观看直播
        |
WebSocket Gateway
        |
      Kafka
        |
 Kafka Cluster
 ----------------------
 |          |          |
Broker1  Broker2  Broker3
        |
 room-service
        |
 用户弹幕广播
```

如果系统每天产生大量弹幕，例如：

```text
10 亿条弹幕/天
```

单 Broker 可能无法承担全部存储和吞吐压力，可以逐步扩展集群：

```text
Kafka Cluster:

10 个 Broker
100 个 Broker
```

业务代码通常不需要因增加 Broker 而修改，但需要重新评估 Partition 数量、副本分配、客户端连接配置和集群再均衡成本。

---
<br/>

### 为什么强调 Cluster

Kafka 本身是分布式系统：单个 Broker 只是一个节点，多个 Broker 才能共同提供可扩展、高可用的服务。

| 系统 | 单机节点 | 集群 |
| --- | --- | --- |
| MySQL | MySQL Server | MySQL Cluster |
| Redis | Redis Server | Redis Cluster |
| Kafka | Kafka Broker | Kafka Cluster |

可以用以下类比辅助记忆：

```text
Kafka Cluster
=
一个公司

Broker
=
公司里的员工

Topic
=
业务部门

Partition
=
部门里的任务分组

Message
=
具体工作内容
```

- **Cluster** 管理整体。
- **Broker** 承担具体存储和请求处理。
- **Topic** 表示业务分类。
- **Partition** 决定数据分片、并行度和分布方式。

**在弹幕、评论、点赞流等大规模事件系统中，Kafka Cluster 通过 Partition 和 Replica 实现消息的分片、存储、复制与横向扩展。**

	
<br/>

***
<br/><br/><br/>
># <h1 id="Go微服务中的配置Kafka与数据库操作">Go 微服务中的配置、Kafka 与数据库操作</h1>

本文整理 Go 微服务启动过程中常见的配置加载、Kafka Producer / Consumer 生命周期，以及数据库批量写入与结果校验方式。

整体链路：

```text
启动参数
   |
   v
加载并校验配置
   |
   v
初始化 Kafka Client
   |
   +------ Producer：发送消息、失败补偿、优雅关闭
   |
   +------ Consumer：拉取消息、业务处理、提交 offset
   |
   v
执行数据库批量写入并校验影响行数
```

***
<br/>

> <h3 id="服务启动配置初始化">服务启动配置初始化</h3>

核心代码：

```go
if err := os.Setenv("MLC_CONFIG_DIR", *configDir); err != nil {
	exitWithError(err)
}

// 先完成 base + 当前环境配置合并，再读取经过类型校验的 MySQL/Redis 配置。
if err := ConfigPackage.LoadConfig(*env); err != nil {
	exitWithError(err)
}

mysqlConfig, err := ConfigPackage.GetMySQLConfig()
if err != nil {
	exitWithError(err)
}
```

这段代码完成三个步骤：设置配置目录、合并基础配置与环境配置、读取并校验强类型 MySQL 配置。任一步骤失败都会立即退出，体现了 **Fail Fast（快速失败）**。

---
<br/>

## 设置配置目录环境变量

`os.Setenv(key, value)` 设置当前进程的环境变量：

```go
func Setenv(key, value string) error
```

例如：

```go
configDir := flag.String(
	"config-dir",
	"./configs",
	"config directory",
)
```

使用以下参数启动服务：

```bash
./server -config-dir=/etc/mlc/config
```

执行：

```go
os.Setenv("MLC_CONFIG_DIR", *configDir)
```

当前进程最终得到：

```text
MLC_CONFIG_DIR=/etc/mlc/config
```

它与 Shell 中的以下命令作用相似，但只影响当前进程及其子进程：

```bash
export MLC_CONFIG_DIR=/etc/mlc/config
```

配置包内部可以统一读取该变量：

```go
func LoadConfig(env string) error {
	dir := os.Getenv("MLC_CONFIG_DIR")
	load(dir)
	return nil
}
```

调用关系：

```text
main
 |
 | 设置 MLC_CONFIG_DIR
 v
ConfigPackage
 |
 | 读取环境变量
 v
Viper / 配置加载器
```

这样 `main` 只负责传递启动环境，不需要了解配置文件的内部组织方式。

---
<br/>

## `if err := ...; err != nil` 语法

```go
if err := os.Setenv("MLC_CONFIG_DIR", *configDir); err != nil {
	exitWithError(err)
}
```

等价于：

```go
err := os.Setenv("MLC_CONFIG_DIR", *configDir)
if err != nil {
	exitWithError(err)
}
```

区别是第一种写法中的 `err` 只在 `if` 语句及其分支内有效，适合只需就地检查一次的错误。

```text
进入 if
  |
创建 err
  |
判断 err != nil
  |
离开 if，err 作用域结束
```

---
<br/>

## 合并 base 与环境配置

常见目录结构：

```text
configs
├── base
│   ├── mysql.yaml
│   └── redis.yaml
├── dev
│   └── mysql.yaml
└── prod
    └── mysql.yaml
```

基础配置：

```yaml
mysql:
  host: localhost
  port: 3306
  user: root
```

生产环境配置：

```yaml
mysql:
  host: mysql.prod.com
```

调用 `LoadConfig("prod")` 后，环境配置覆盖同名字段，其余字段继承基础配置：

```yaml
mysql:
  host: mysql.prod.com
  port: 3306
  user: root
```

---
<br/>

## 获取并校验强类型配置

```go
mysqlConfig, err := ConfigPackage.GetMySQLConfig()
```

配置可映射到 Go 结构体：

```go
type MySQLConfig struct {
	Host     string
	Port     int
	Username string
	Password string
	Database string
}
```

如果配置为：

```yaml
mysql:
  port: abc
```

由于 `Port` 要求为 `int`，解析或校验会失败。**配置文件成功加载，不代表配置值满足程序所需类型与约束。**

完整启动流程：

```text
程序启动
   |
读取命令行参数并 flag.Parse()
   |
得到 env=prod、configDir=/etc/config
   |
os.Setenv("MLC_CONFIG_DIR", "/etc/config")
   |
LoadConfig("prod")
   |
base 配置 + prod 配置
   |
GetMySQLConfig()
   |
类型转换与校验
   |
创建 MySQL 连接池
   |
启动 HTTP / Kafka 服务
```

**设计要点：** 配置初始化前置、环境隔离、模块解耦、错误快速暴露。

***
<br/>

> <h3 id="KafkaClient的FunctionalOptions">Kafka Client 的 Functional Options</h3>

核心代码：

```go
opts := []kgo.Opt{
	kgo.SeedBrokers(cfg.Brokers...),
}

if cfg.GroupID != "" {
	opts = append(opts, kgo.ConsumerGroup(cfg.GroupID))
}

client, err := kgo.NewClient(opts...)
```

`opts` 是元素类型为 `kgo.Opt` 的切片，用来收集 Kafka Client 配置项；最后通过 `opts...` 展开为可变参数传给 `kgo.NewClient`。

---
<br/>

## `[]kgo.Opt` 与 Option Pattern

```go
opts := []kgo.Opt{
	kgo.SeedBrokers(cfg.Brokers...),
}
```

其类型为：

```go
[]kgo.Opt
```

结构可理解为：

```text
[]kgo.Opt
    |
    +-- kgo.Opt
    +-- kgo.Opt
    +-- kgo.Opt
```

franz-go 使用函数式选项模式。概念上，配置项会修改客户端内部配置：

```go
type Opt interface {
	apply(*cfg)
}
```

例如 `kgo.SeedBrokers(...)` 返回一个配置项，客户端创建时依次应用所有配置项：

```go
func NewClient(opts ...Opt) (*Client, error) {
	cfg := defaultConfig()
	for _, opt := range opts {
		opt.apply(&cfg)
	}
	return &Client{cfg: cfg}, nil
}
```

以上代码是原理化示意，实际库内部实现以 franz-go 源码为准。

---
<br/>

## 两处 `...` 的含义

### 展开 Broker 地址

```go
kgo.SeedBrokers(cfg.Brokers...)
```

假设：

```go
cfg.Brokers = []string{
	"10.0.0.1:9092",
	"10.0.0.2:9092",
}
```

则调用等价于：

```go
kgo.SeedBrokers(
	"10.0.0.1:9092",
	"10.0.0.2:9092",
)
```

### 展开配置切片

```go
kgo.NewClient(opts...)
```

`NewClient` 接收可变参数：

```go
func NewClient(opts ...Opt) (*Client, error)
```

如果：

```go
opts := []kgo.Opt{opt1, opt2}
```

则 `NewClient(opts...)` 等价于：

```go
NewClient(opt1, opt2)
```

直接传 `NewClient(opts)` 会发生类型不匹配：函数需要多个 `Opt`，而不是一个 `[]Opt`。

---
<br/>

## 动态组合配置

```go
func NewKafkaClient(cfg Config) (*kgo.Client, error) {
	opts := []kgo.Opt{
		kgo.SeedBrokers(cfg.Brokers...),
	}

	if cfg.ClientID != "" {
		opts = append(opts, kgo.ClientID(cfg.ClientID))
	}

	if cfg.GroupID != "" {
		opts = append(opts, kgo.ConsumerGroup(cfg.GroupID))
	}

	return kgo.NewClient(opts...)
}
```

函数式选项适合配置项很多、部分配置需要按条件启用的场景，可避免构造函数参数持续膨胀。

| 写法 | 含义 |
| --- | --- |
| `[]kgo.Opt` | `kgo.Opt` 类型的切片 |
| `append(opts, opt)` | 动态增加配置项 |
| `cfg.Brokers...` | 将 `[]string` 展开为多个字符串参数 |
| `opts...` | 将 `[]kgo.Opt` 展开为多个配置参数 |
| `NewClient(opts ...Opt)` | 接收任意数量的配置项 |

***
<br/>

> <h3 id="Kafka健康检查">Kafka 健康检查</h3>

核心代码：

```go
func hgPingKafkaClient(client *kgo.Client, timeout time.Duration) error {
	// 启动期没有上游请求 ctx，因此使用固定超时的 Background context；cancel 必须释放计时器资源。
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()

	return client.Ping(ctx)
}
```

该函数用于在服务启动阶段主动验证 Kafka Client 是否能连接 Broker。

---
<br/>

## 为什么创建 Client 后还要 Ping

```go
client, err := kgo.NewClient(opts...)
```

客户端创建成功主要表示配置可用于构造对象，**不一定代表 Kafka 集群当前可访问**。Broker 宕机、DNS 失败、网络不通、SASL 认证失败或 TLS 握手失败，都可能在首次网络请求时才暴露。

```text
创建 Client
    |
    v
保存并应用配置
    |
    v
client.Ping(ctx)
    |
    +---- 成功：继续启动服务
    |
    +---- 失败：快速退出或标记 readiness=false
```

`client.Ping(ctx)` 会发起 Kafka 协议请求，用于验证网络、Broker、协议与认证链路。

| 组件 | 常见探活方式 |
| --- | --- |
| MySQL | `SELECT 1` |
| Redis | `PING` |
| HTTP 服务 | `/health` |
| Kafka | `client.Ping(ctx)` |

---
<br/>

## 超时与资源释放

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()
```

超时用于避免网络异常时启动流程无限等待：

```text
0s：开始 Ping
 |
 | 等待 Broker
 |
3s：context deadline exceeded
```

`defer cancel()` 会在函数结束时及时释放 `WithTimeout` 创建的计时器资源，即使请求在超时前已经完成也应调用。

启动检查示例：

```go
func InitKafka(cfg Config) (*kgo.Client, error) {
	client, err := kgo.NewClient(kgo.SeedBrokers(cfg.Brokers...))
	if err != nil {
		return nil, err
	}

	if err := hgPingKafkaClient(client, 5*time.Second); err != nil {
		client.Close()
		return nil, fmt.Errorf("kafka unavailable: %w", err)
	}

	return client, nil
}
```

在 Kubernetes 中，可将探活结果用于 readiness：失败时返回 `503`，避免 Pod 接收业务流量。

***
<br/>

> <h3 id="Kafka同步生产消息">Kafka 同步生产消息</h3>

核心调用：

```go
err := client.ProduceSync(ctx, record).FirstErr()
```

含义：同步发送一条或多条消息，等待 Broker 返回结果，再获取第一个发送错误。

---
<br/>

## `Record` 消息结构

```go
record := &kgo.Record{
	Topic: "video-events",
	Key:   []byte("user_10001"),
	Value: []byte(`{
		"event": "video_uploaded",
		"video_id": "abc123"
	}`),
}
```

常用字段：

| 字段 | 作用 |
| --- | --- |
| `Topic` | 目标 Topic |
| `Key` | 分区键或业务键 |
| `Value` | 消息内容 |

---
<br/>

## 同步发送与 `FirstErr`

```text
业务 goroutine
    |
ProduceSync(ctx, records...)
    |
等待 Kafka Broker ACK
    |
返回每条消息的 ProduceResult
    |
FirstErr() 获取第一个错误
```

`ProduceSync` 可以一次发送多条消息，因此返回结果集合而不是单个 `error`：

```go
results := client.ProduceSync(ctx, record1, record2, record3)
err := results.FirstErr()
```

- 全部成功：`FirstErr()` 返回 `nil`。
- 任意一条失败：返回结果中的第一个错误。

完整封装：

```go
func SendVideoEvent(
	ctx context.Context,
	client *kgo.Client,
	record *kgo.Record,
) error {
	if err := client.ProduceSync(ctx, record).FirstErr(); err != nil {
		return fmt.Errorf("produce kafka failed: %w", err)
	}
	return nil
}
```

**发送成功仅表示 Producer 按当前 `acks` 策略得到了 Kafka 的确认，不表示 Consumer 已完成业务处理。**

```text
Producer -> Kafka 写入并 ACK -> Consumer 拉取 -> 业务处理
             ^
             |
       ProduceSync 成功点
```

同步发送适合订单、支付结果等需要立即获知投递结果的关键事件；高吞吐日志、埋点通常更适合异步 `Produce`。

***
<br/>

> <h3 id="Kafka异步生产与DLQ">Kafka 异步生产与 DLQ</h3>

核心代码：

```go
client := HGClient()

client.Produce(ctx, record, func(r *kgo.Record, err error) {
	if err == nil {
		return
	}

	logHG.ErrFInfo(
		"produce kafka log event failed topic=%s err=%v",
		topic,
		err,
	)

	if dlqErr := HGSendDLQ(ctx, r, "log", err.Error()); dlqErr != nil {
		logHG.ErrFInfo(
			"send kafka log dlq failed topic=%s err=%v",
			topic,
			dlqErr,
		)
	}
})
```

`Produce` 异步提交消息，不阻塞当前业务 goroutine；Kafka 返回结果后执行 callback。

```text
业务 goroutine
   |
client.Produce()
   |
立即返回并继续业务

franz-go 后台 Producer
   |
发送到 Kafka Broker
   |
执行 callback(record, err)
```

---
<br/>

## callback 与 DLQ

callback 参数：

```go
func(r *kgo.Record, err error)
```

- `err == nil`：消息发送成功，直接返回。
- `err != nil`：记录错误，并将失败消息转移到 DLQ。

DLQ（Dead Letter Queue，死信队列）用于保存无法正常投递或处理的消息，便于后续重试、审计和人工排查：

```text
业务服务
   |
发送正常 Topic
   |
   +---- 成功
   |
   +---- 失败 ----> DLQ Topic ----> 重试 / 告警 / 人工处理
```

`Produce + callback + DLQ` 适合日志、埋点等高吞吐场景，在不阻塞主流程的同时提供失败补偿。

---
<br/>

## `Produce` 与 `ProduceSync` 对比

| 对比项 | `Produce` | `ProduceSync` |
| --- | --- | --- |
| 模式 | 异步 | 同步 |
| 是否等待 ACK | 当前调用不等待 | 等待 |
| 结果处理 | callback | 返回结果集合并调用 `FirstErr()` |
| 吞吐 | 较高 | 较低 |
| 适用场景 | 日志、埋点、事件流 | 订单、支付、关键任务 |

---
<br/>

## 需要注意的边界

**Client 判空：** 如果 `HGClient()` 可能返回 `nil`，调用 `Produce` 前必须处理，否则会 panic。

```go
client := HGClient()
if client == nil {
	return errors.New("kafka client not initialized")
}
```

**Context 生命周期：** 如果异步日志使用 HTTP 请求的 `ctx`，请求结束后 context 可能被取消，从而影响尚未完成的发送。是否改用独立 context，应根据业务是否允许发送脱离请求生命周期决定，不能一律替换为 `context.Background()`。

***
<br/>

> <h3 id="KafkaProducer优雅关闭">Kafka Producer 优雅关闭</h3>

核心代码：

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	10*time.Second,
)
defer cancel()

if err := client.Flush(ctx); err != nil {
	logHG.ErrFInfo(
		"flush kafka client failed err=%v",
		err,
	)
}

client.Close()
```

异步 `Produce` 返回时，消息可能仍在客户端缓冲区中。服务退出前应先等待缓冲消息完成，再释放客户端资源。

```text
收到 SIGTERM
   |
停止接收新请求
   |
Flush(ctx)
   |
等待缓冲消息发送与 callback 完成
   |
Close()
   |
释放连接和后台资源
   |
进程退出
```

---
<br/>

## `Flush` 与 `Close` 的职责

| 方法 | 职责 |
| --- | --- |
| `Flush(ctx)` | 等待当前缓冲区内未完成的 Produce 请求结束 |
| `Close()` | 关闭 Kafka Client，释放 TCP 连接、goroutine、缓存和计时器等资源 |

`Flush` 需要超时限制。如果 Kafka 不可用，无超时等待可能导致服务无法在 Kubernetes 的 `terminationGracePeriod` 内退出，最终被 `SIGKILL` 强制终止。

```text
0s：开始 Flush
 |
 | Kafka 正常：消息发送完成，提前返回 nil
 |
10s：仍未完成，context deadline exceeded
```

生产服务的关闭顺序通常应先停止新流量，再关闭依赖组件：

```go
func shutdown() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	_ = httpServer.Shutdown(ctx)
	_ = kafkaClient.Flush(ctx)
	kafkaClient.Close()
}
```

具体顺序需结合业务：若 HTTP 请求还会继续产生 Kafka 消息，应先停止接收请求并等待处理结束，再 Flush Kafka。

***
<br/>

> <h3 id="Kafkaacks确认策略">Kafka acks 确认策略</h3>

核心代码：

```go
switch acks {
case HGAcksNone:
	return kgo.NoAck()
case HGAcksLeader:
	return kgo.LeaderAck()
default:
	return kgo.AllISRAcks()
}
```

`acks` 决定 Producer 在什么条件下认为消息发送成功，是可靠性与性能之间的核心权衡。

| `acks` | franz-go 值 | 成功条件 | 可靠性 | 性能 |
| --- | --- | --- | --- | --- |
| `0` | `kgo.NoAck()` | 不等待 Broker 确认 | 最低 | 最高 |
| `1` | `kgo.LeaderAck()` | Leader 写入后确认 | 中等 | 较高 |
| `all/-1` | `kgo.AllISRAcks()` | 满足 ISR 确认条件 | 最高 | 较低 |

---
<br/>

## `acks=0`：不等待确认

```text
Producer
   |
发送消息
   |
立即认为完成

Broker 可能尚未收到消息
```

适合允许少量丢失、吞吐优先的数据，例如部分 debug 日志、trace 或监控指标。Producer 无法通过 ACK 判断 Broker 是否真正写入消息。

---
<br/>

## `acks=1`：等待 Leader 确认

```text
Producer
   |
Leader Broker 写入
   |
返回 ACK
   |
Follower 可能仍在同步
```

它兼顾性能与可靠性，但 Leader 在副本同步前故障时仍可能丢失消息。

---
<br/>

## `acks=all`：等待 ISR 条件

ISR 是 `In-Sync Replica`，即保持同步的副本集合：

```text
Producer
   |
Leader 写入
   |
ISR 副本同步
   |
满足确认条件
   |
返回 ACK
```

该策略可靠性更高，但会增加等待时间。核心交易场景通常还需结合幂等生产、合理的副本数与 `min.insync.replicas` 配置，不能只依赖 `acks=all`。

使用 `default -> AllISRAcks()` 表示未知配置值优先采用可靠策略。不过更严格的系统也可以在配置校验阶段直接拒绝未知值，避免静默接受拼写错误。

***
<br/>

> <h3 id="KafkaConsumer生命周期">Kafka Consumer 生命周期</h3>

核心模式：

```go
type HGBaseConsumer struct {
	once sync.Once
	cli  *kgo.Client
}

func (b *HGBaseConsumer) Start(
	ctx context.Context,
	handle func(*kgo.Record),
) {
	b.once.Do(func() {
		go b.consumeLoop(ctx, handle)
	})
}
```

它组合了三种机制：

- `sync.Once`：确保消费循环只启动一次。
- `go b.consumeLoop(...)`：让长期运行的消费循环在独立 goroutine 中执行。
- `context`：控制消费循环停止。

---
<br/>

## `sync.Once` 保证唯一启动

如果 `Start()` 被重复调用，而每次都执行 `go consumeLoop()`，就会启动多个消费循环：

```text
Start() ----> consumer goroutine 1
Start() ----> consumer goroutine 2
Start() ----> consumer goroutine 3
```

这可能引发重复逻辑、并发提交和额外资源消耗。`once.Do` 保证传入函数在该 `sync.Once` 生命周期内最多执行一次：

```go
b.once.Do(func() {
	go b.consumeLoop(ctx, handle)
})
```

注意：`sync.Once` 执行后不能重置，因此这种设计通常表示 Consumer 实例只能启动一次，停止后不能通过再次调用 `Start()` 重启。

---
<br/>

## goroutine 避免阻塞启动流程

消费循环通常长期运行：

```go
func (b *HGBaseConsumer) consumeLoop(ctx context.Context) {
	for {
		fetches := b.cli.PollFetches(ctx)
		_ = fetches
	}
}
```

直接调用会阻塞当前 goroutine；使用 `go` 后，主流程可以继续启动 HTTP 服务等组件：

```text
main goroutine
   |
启动 consumer goroutine
   |
继续启动其他组件

consumer goroutine
   |
持续 Poll Kafka
```

goroutine 由 Go Runtime 调度，不等同于一个操作系统线程。

---
<br/>

## 使用 context 停止循环

非阻塞检查写法：

```go
select {
case <-ctx.Done():
	return
default:
}
```

- context 已取消：`ctx.Done()` 可读，执行 `return`。
- context 未取消：执行 `default`，继续消费。

如果只有 `<-ctx.Done()` 而没有 `default`，代码会一直等待取消，后续 Poll 无法执行。

由于 `PollFetches(ctx)` 本身支持 context，通常也可以在 Poll 返回后检查：

```go
for {
	fetches := b.cli.PollFetches(ctx)
	if ctx.Err() != nil {
		return
	}

	// 处理 fetches
	_ = fetches
}
```

完整生命周期：

```text
Start()
  |
once.Do()
  |
go consumeLoop()
  |
PollFetches(ctx)
  |
处理消息
  |
服务关闭时 cancel()
  |
Poll 返回，consumeLoop 退出
  |
goroutine 结束
```

***
<br/>

> <h3 id="PollFetches拉取与提交消息">PollFetches 拉取与提交消息</h3>

核心代码：

```go
for {
	fetches := b.cli.PollFetches(ctx)
	if ctx.Err() != nil {
		return
	}

	if errs := fetches.Errors(); len(errs) > 0 {
		// 记录或按错误类型处理拉取错误
	}

	fetches.EachRecord(func(record *kgo.Record) {
		handle(record)
	})
}
```

`PollFetches` 主动向 Kafka 拉取一批消息，并阻塞到消息可用、发生错误或 context 被取消。Kafka Consumer 属于 **Pull 模型**。

```text
Consumer
   |
   | Fetch Request
   v
Kafka Broker
   |
   | Fetch Response
   v
kgo.Fetches
```

---
<br/>

## 批量返回 `kgo.Fetches`

Kafka 为提高吞吐，一次 Fetch 可以返回多个分区中的多条消息：

```text
Fetches
├── Partition 0
│   ├── record offset=100
│   ├── record offset=101
│   └── record offset=102
└── Partition 1
    ├── record offset=40
    └── record offset=41
```

随后使用：

```go
fetches.EachRecord(func(record *kgo.Record) {
	handle(record)
})
```

批量 Fetch 能减少网络往返，相比每条消息一次请求更适合高吞吐场景。

---
<br/>

## Poll、处理与 offset 提交

典型流程：

```text
Kafka Broker
   |
PollFetches(ctx)
   |
检查 fetches.Errors()
   |
EachRecord()
   |
业务 Handler
   |
成功后提交 offset
```

手动提交示意：

```go
fetches.EachRecord(func(record *kgo.Record) {
	if err := handle(ctx, record); err != nil {
		return
	}

	if err := b.cli.CommitRecords(ctx, record); err != nil {
		// 记录提交失败，结合业务设计重试或终止策略
	}
})
```

处理成功后再提交 offset，可形成至少一次处理语义：

```text
拉取消息
   |
业务处理成功
   |
提交 offset

业务处理失败
   |
不提交 offset
   |
后续可能再次消费
```

需要注意：逐条 `CommitRecords` 会产生较多提交请求；并发处理同一分区消息时，还要避免提前提交更高 offset 导致低 offset 失败后无法重放。具体提交策略应根据顺序、吞吐和重复消费容忍度设计。

***
<br/>

> <h3 id="append构造批量INSERT">append 构造批量 INSERT</h3>

核心代码：

```go
valueParts = append(valueParts, "(?, ?, ?, ?, NOW())")
```

该语句向 `[]string` 切片追加一组 SQL `VALUES` 占位模板，后续通过 `strings.Join` 拼接批量插入语句。

---
<br/>

## `append` 的语义

可将内置函数理解为：

```go
func append(slice []T, elems ...T) []T
```

例如：

```go
nums := []int{1, 2, 3}
nums = append(nums, 4)
```

结果：

```text
[1, 2, 3, 4]
```

必须接收 `append` 的返回值，因为追加元素时底层数组可能扩容，返回的 slice header 可能指向新的数组：

```text
原 slice -> 原底层数组（容量已满）
                 |
               append
                 |
新 slice -> 更大的底层数组
```

因此应写：

```go
valueParts = append(valueParts, "...")
```

---
<br/>

## 构造批量 INSERT

```go
valueParts := make([]string, 0, len(users))
args := make([]any, 0, len(users)*4)

for _, user := range users {
	valueParts = append(valueParts, "(?, ?, ?, ?, NOW())")
	args = append(
		args,
		user.ID,
		user.Name,
		user.Age,
		user.Email,
	)
}

query := fmt.Sprintf(`
INSERT INTO users
    (id, name, age, email, created_at)
VALUES %s
`, strings.Join(valueParts, ","))

res, err := db.ExecContext(ctx, query, args...)
```

生成的 SQL：

```sql
INSERT INTO users
    (id, name, age, email, created_at)
VALUES
    (?, ?, ?, ?, NOW()),
    (?, ?, ?, ?, NOW()),
    (?, ?, ?, ?, NOW())
```

`args` 则按占位符顺序保存每条记录的参数，最后通过 `args...` 展开传给 `ExecContext`。

使用参数占位符而不是直接拼接用户数据，可以降低 SQL 注入风险，并让驱动正确处理转义与类型。动态拼接的内容只应是受程序控制的 SQL 结构片段。

---
<br/>

## 预分配容量

已知批次大小时，建议预分配切片容量：

```go
valueParts := make([]string, 0, batchSize)
args := make([]any, 0, batchSize*4)
```

这样可以减少底层数组扩容、数据复制和 GC 压力。实际批次还需受数据库最大参数数量、SQL 包大小和事务耗时限制，不能无限增大。

***
<br/>

> <h3 id="RowsAffected获取影响行数">RowsAffected 获取影响行数</h3>

核心代码：

```go
res, err := db.ExecContext(ctx, query, args...)
if err != nil {
	return err
}

rowsAffected, err := res.RowsAffected()
if err != nil {
	return err
}
```

`RowsAffected()` 从 `sql.Result` 中获取 SQL 实际影响的行数，常用于校验 `INSERT`、`UPDATE` 和 `DELETE` 的业务结果。

```go
type Result interface {
	LastInsertId() (int64, error)
	RowsAffected() (int64, error)
}
```

---
<br/>

## 常见返回结果

### UPDATE

```go
res, err := db.ExecContext(
	ctx,
	"UPDATE users SET name=? WHERE id=?",
	"Mike",
	2,
)
```

若成功修改一行：

```go
rowsAffected == 1
```

如果 `WHERE` 没有匹配记录，SQL 语法仍然合法，因此可能出现：

```text
err = nil
rowsAffected = 0
```

所以 `err == nil` 只表示 SQL 成功执行，不一定表示目标业务数据存在并被修改。

### DELETE

```go
res, err := db.ExecContext(ctx, "DELETE FROM users WHERE id=?", 100)
```

- 记录不存在：`rowsAffected == 0`。
- 成功删除一行：`rowsAffected == 1`。

### 批量 INSERT

插入 1000 条记录时，正常情况下：

```go
rowsAffected == 1000
```

可用于批次完整性校验：

```go
rows, err := res.RowsAffected()
if err != nil {
	return err
}

if rows != int64(len(items)) {
	log.Warn(
		"batch insert incomplete",
		"expected", len(items),
		"actual", rows,
	)
}
```

---
<br/>

## MySQL UPDATE 的特殊语义

默认情况下，MySQL 通常返回实际发生变化的行数。若数据原本已经是目标值：

```sql
UPDATE users
SET name = 'Tom'
WHERE id = 1;
```

即使 `id=1` 存在，也可能得到：

```text
RowsAffected = 0
```

因此不能在所有业务中简单地把 `rowsAffected == 0` 等同于“记录不存在”。如果需要返回匹配行数，可根据驱动使用类似 `clientFoundRows=true` 的 DSN 配置，但这会改变语义，应在项目中统一约定。

---
<br/>

## `LastInsertId` 与 `RowsAffected`

| 方法 | 作用 |
| --- | --- |
| `LastInsertId()` | 获取数据库返回的最后插入 ID，是否支持取决于驱动和数据库 |
| `RowsAffected()` | 获取 INSERT、UPDATE、DELETE 影响的行数 |

最终校验链路：

```text
执行 SQL
   |
检查 ExecContext error
   |
获取 RowsAffected
   |
结合 SQL 类型和数据库语义判断业务结果
   |
记录指标、告警或执行重试
```

**核心原则：先检查 SQL 执行错误，再结合影响行数和具体数据库语义判断业务操作是否真正达到预期。**


<br/>

***
<br/><br/><br/>
># <h1 id="大厂kafka方案">大厂kafka方案</h1>

 franz-go 大厂级完整工程方案（适配亿万数据、千万并发）
 
 ***
<br/><br/><br/>
> <h2 id="项目目录布局">项目目录布局</h2>
 
## 项目目录布局（对标之前sarama规范，统一分层）

```
service-log-stream/
├── cmd/server/main.go          # 初始化全局kgo单例、注册消费任务
├── internal
│   ├── conf/config.go          # Nacos配置，多集群隔离（埋点/业务）
│   ├── kafka                   # franz-go统一封装层（核心）
│   │   ├── client.go           # 全局生产者单例、发送通用方法
│   │   ├── consumer.go         # 统一消费基类、自动offset管理、DLQ
│   │   ├── config_builder.go   # 两套配置：高吞吐埋点 / 高可靠交易
│   │   ├── metric_hook.go      # prometheus埋点钩子（发送成功/lag/耗时）
│   │   ├── trace_hook.go       # header注入traceId，全链路追踪
│   │   └── dlq.go              # 统一死信投递、失败消息缓存补偿
│   ├── domain/event            # 事件结构体、topic/group常量
│   ├── service                 # 业务层，调用kafka发送事件
│   ├── handler                 # HTTP/RPC接口层，透传ctx到kafka
│   └── consumer_task           # 所有消费任务注册入口
├── pkg/logger
└── go.mod
```

***
<br/><br/><br/>
> <h2 id="依赖引入">依赖引入</h2>

```bash
go get github.com/twmb/franz-go/pkg/kgo
go get github.com/twmb/franz-go/pkg/kadm # 集群管理API（创建topic/查询offset）
```

***
<br/><br/><br/>
> <h2 id="配置层">配置层</h2>

### 配置层 internal/conf/config.go

```go
package conf

type AppConf struct {
	Kafka KafkaClusterConf `yaml:"kafka"`
}

type KafkaClusterConf struct {
	Business Cluster `yaml:"business"` // 业务消息集群
	Log      Cluster `yaml:"log"`      // 埋点日志集群
}

type Cluster struct {
	Brokers []string `yaml:"brokers"`
	Acks    string   `yaml:"acks"` // 1 / all
	Retry   int      `yaml:"retry"`
}
```

***
<br/><br/><br/>
> <h2 id="kafka配置工厂">kafka配置工厂</h2>

### kafka配置工厂 internal/kafka/config_builder.go
两套生产配置，大厂分级策略（franz-go option模式，无臃肿结构体）
```go
package kafka

import (
	"github.com/twmb/franz-go/pkg/kgo"
	"service-log-stream/internal/conf"
	"github.com/twmb/franz-go/pkg/kcompress"
)

// NewLogProducerOpts 埋点集群：极致吞吐，acks=1，高批量
func NewLogProducerOpts(cfg conf.Cluster) []kgo.Opt {
	opts := []kgo.Opt{
		kgo.SeedBrokers(cfg.Brokers...),
		kgo.RequiredAcks(kgo.AcksOne),
		kgo.ProducerBatchCompression(kcompress.Lz4),
		kgo.ProducerBatchMaxBytes(16 * 1024 * 1024),
		kgo.ProducerBatchMaxDuration(50), // 50ms强制刷批
		kgo.ProducerMaxRetries(cfg.Retry),
		// 开启幂等生产者，防止重试重复
		kgo.IdempotentProducer(),
	}
	// 注入监控、链路钩子
	opts = append(opts, MetricHookOpt(), TraceHookOpt())
	return opts
}

// NewBusinessProducerOpts 业务交易集群：acks=all，可靠不丢消息
func NewBusinessProducerOpts(cfg conf.Cluster) []kgo.Opt {
	opts := []kgo.Opt{
		kgo.SeedBrokers(cfg.Brokers...),
		kgo.RequiredAcks(kgo.AcksAll),
		kgo.ProducerBatchCompression(kcompress.Lz4),
		kgo.ProducerBatchMaxDuration(100),
		kgo.ProducerMaxRetries(cfg.Retry),
		kgo.IdempotentProducer(),
		// 如需事务，开启：kgo.TransactionalID("service-order-v1"),
	}
	opts = append(opts, MetricHookOpt(), TraceHookOpt())
	return opts
}
```

***
<br/><br/><br/>
> <h2 id="全局生产者单例">全局生产者单例</h2>

### 全局生产者单例 internal/kafka/client.go（千万并发核心封装）
franz-go单Client同时支持生产+消费，无需区分producer/consumer两套连接，大幅减少Broker连接数（大厂集群连接管控核心优化）
```go
package kafka

import (
	"context"
	"encoding/json"
	"github.com/twmb/franz-go/pkg/kgo"
	"service-log-stream/internal/conf"
	"service-log-stream/pkg/logger"
)

var GlobalKgoClient *kgo.Client

// InitKafka 程序启动初始化全局客户端
func InitKafka(cfg conf.KafkaClusterConf) error {
	// 业务客户端（生产+消费共用）
	bizOpts := NewBusinessProducerOpts(cfg.Business)
	bizCli, err := kgo.NewClient(bizOpts...)
	if err != nil {
		return err
	}
	GlobalKgoClient = bizCli

	// 埋点客户端可独立创建，或复用同一client（看流量隔离需求）
	return nil
}

// SendBusinessEvent 接口/service层调用发送业务事件
func SendBusinessEvent(ctx context.Context, topic string, key string, data any) error {
	val, err := json.Marshal(data)
	if err != nil {
		return err
	}
	record := &kgo.Record{
		Topic: topic,
		Key:   []byte(key),
		Value: val,
	}
	// 注入trace到header
	InjectTraceToRecord(ctx, record)
	// 同步发送；超高吞吐用ProduceAsync非阻塞
	return GlobalKgoClient.ProduceSync(ctx, record).FirstErr()
}

// SendLogEvent 埋点日志异步发送，不阻塞业务接口
func SendLogEvent(ctx context.Context, topic string, data any) {
	val, _ := json.Marshal(data)
	record := &kgo.Record{Topic: topic, Value: val}
	InjectTraceToRecord(ctx, record)
	GlobalKgoClient.ProduceAsync(ctx, record, func(r *kgo.Record, err error) {
		if err != nil {
			logger.Error("send log failed", logger.Err(err), logger.String("topic", topic))
			_ = SendDLQ(ctx, r, "log", err.Error())
		}
	})
}

// Close 优雅关闭，等待缓存消息全部发送完成
func Close() {
	if GlobalKgoClient != nil {
		_ = GlobalKgoClient.Flush(context.Background())
		GlobalKgoClient.Close()
	}
}
```

***
<br/><br/><br/>
> <h2 id="统一消费基类">统一消费基类</h2>

### 统一消费基类 internal/kafka/consumer.go（大厂标准消费模型）
franz-go消费者天然支持手动提交offset，自动重平衡、动态感知新增分区，无需像sarama维护复杂ConsumerGroupHandler

```go
package kafka

import (
	"context"
	"github.com/twmb/franz-go/pkg/kgo"
	"service-log-stream/pkg/logger"
)

type BaseConsumer struct {
	dlqTopic string
	cli      *kgo.Client
}

func NewBaseConsumer(cli *kgo.Client, dlqTopic string) *BaseConsumer {
	return &BaseConsumer{cli: cli, dlqTopic: dlqTopic}
}

// StartConsume 启动消费循环，支持多topic、自动负载均衡
func (b *BaseConsumer) StartConsume(ctx context.Context, topics []string, handle func(ctx context.Context, r *kgo.Record) error) {
	// 订阅topic，自动监听分区扩容
	b.cli.Subscribe(topics...)
	go func() {
		for {
			fetches := b.cli.PollFetches(ctx)
			if errs := fetches.Errors(); len(errs) > 0 {
				logger.Error("kafka fetch error", logger.Any("errs", errs))
				continue
			}
			// 遍历所有拉取到的消息
			fetches.EachRecord(func(r *kgo.Record) {
				traceCtx := ExtractTraceFromRecord(r)
				err := handle(traceCtx, r)
				if err != nil {
					logger.Error("consume handle fail, send dlq", logger.Err(err), logger.String("topic", r.Topic))
					_ = SendDLQ(traceCtx, r, "business", err.Error())
					return // 失败不提交offset，下次重新消费
				}
				// 业务处理成功，手动提交offset
				b.cli.CommitRecords(ctx, r)
			})
		}
	}()
}
```

***
<br/><br/><br/>
> <h2 id="业务消费任务示例">业务消费任务示例</h2>

### 业务消费任务示例 internal/consumer_task/order_consumer.go

```go
package consumer_task

import (
	"context"
	"encoding/json"
	"service-log-stream/internal/kafka"
	"service-log-stream/internal/domain/event"
	"service-log-stream/internal/service"
)

type OrderConsumer struct {
	base *kafka.BaseConsumer
	svc  *service.OrderSvc
}

func NewOrderConsumer(svc *service.OrderSvc) *OrderConsumer {
	base := kafka.NewBaseConsumer(kafka.GlobalKgoClient, event.TopicDLQBusiness)
	return &OrderConsumer{base: base, svc: svc}
}

func (o *OrderConsumer) Register() {
	topics := []string{event.TopicOrderCreate}
	o.base.StartConsume(context.Background(), topics, o.handleMsg)
}

func (o *OrderConsumer) handleMsg(ctx context.Context, r *kgo.Record) error {
	var evt event.OrderCreateEvent
	if err := json.Unmarshal(r.Value, &evt); err != nil {
		return err
	}
	return o.svc.ConsumeOrderEvent(ctx, &evt)
}
```

***
<br/><br/><br/>
> <h2 id="HTTP接口调用示例">HTTP接口调用示例</h2>

### HTTP接口调用示例 internal/handler/order_handler.go
和sarama调用逻辑完全对齐，业务层无感知底层库切换

```go
func (h *OrderHandler) CreateOrder(c *gin.Context) {
	orderId := "ORD99999"
	evt := event.OrderCreateEvent{OrderID: orderId, UserID: 10001}
	// 透传ctx，自动注入traceId到kafka header
	err := kafka.SendBusinessEvent(c.Request.Context(), event.TopicOrderCreate, orderId, evt)
	if err != nil {
		c.JSON(500, gin.H{"code": -1, "msg": "发送事件失败"})
		return
	}
	c.JSON(200, gin.H{"msg": "ok"})
}
```

***
<br/><br/><br/>
> <h2 id="main入口初始化">main入口初始化</h2>

```go
func main() {
	conf := conf.LoadNacosConfig()
	// 全局一次性初始化franz-go客户端
	if err := kafka.InitKafka(conf.Kafka); err != nil {
		log.Fatal(err)
	}
	defer kafka.Close()

	// 注册所有消费任务
	orderConsumer := consumer_task.NewOrderConsumer(service.NewOrderSvc())
	orderConsumer.Register()

	// 启动http服务
	r := gin.Default()
	r.POST("/api/order/create", handler.NewOrderHandler().CreateOrder)
	r.Run(":8080")
}
```

<br/>

## 千万并发、亿万流量franz-go专属优化（sarama做不到）
1. **单Client复用生产+消费**
   sarama必须分开创建Producer、ConsumerGroup两套连接，Broker连接数翻倍；franz-go一个Client同时生产消费，大幅降低集群连接压力，适合千实例微服务集群。
2. **自动动态分区感知**
   Topic扩容分区后，客户端立刻感知，无需重启服务，大数据平台频繁扩分区场景刚需。
3. **精细化批量与in-flight控制**
   franz-go自动合并多分区请求，充分打满网卡，sarama多分区发送时吞吐衰减严重。
4. **原生事务/幂等**
   订单+数据库本地事务原子提交，实现EOS Exactly Once，sarama需要大量自研封装，极易丢消息/重复消息。
5. **钩子扩展无侵入**
   监控、日志、trace全部通过hook注入，不用修改发送/消费主循环代码，架构更干净。
6. **内存池化减少GC**
   高并发7×24小时运行无内存毛刺，日志/埋点服务不会出现半夜OOM。

***
<br/><br/><br/>
> <h2 id="MLC_GO工程改动">MLC_GO工程改动</h2>

MLC_GO 工程落地记录（2026-07-04）

### 1. 修改了哪些文件

- `go.mod`
- `config/config.debug.yaml`
- `config/config.pre.yaml`
- `config/config.prod.yaml`
- `main_mlc_project.go`
- `hg_kafka_application.go`
- `hg_kafka_application_test.go`
- `internal/pkg/config/hg_kafka_config.go`
- `internal/pkg/config/hg_kafka_config_test.go`
- `internal/pkg/kafka/hg_config.go`
- `internal/pkg/kafka/hg_config_builder.go`
- `internal/pkg/kafka/hg_client.go`
- `internal/pkg/kafka/hg_consumer.go`
- `internal/pkg/kafka/hg_dlq.go`
- `internal/pkg/kafka/hg_metric_hook.go`
- `internal/pkg/kafka/hg_trace.go`
- `internal/pkg/kafka/hg_client_test.go`
- `internal/pkg/kafka/hg_config_builder_test.go`
- `internal/pkg/kafka/hg_trace_test.go`

### 2. 做了什么改动

- 引入 `franz-go` 核心 Kafka 客户端依赖：
	- `github.com/twmb/franz-go v1.16.0`
	- `github.com/twmb/franz-go/pkg/kmsg v1.7.0`
	- `github.com/pierrec/lz4/v4 v4.1.19`
- 新增 `internal/pkg/kafka` 统一封装层，对齐本文前面的工程方法：
	- `HGKafkaClusterConfig` / `HGClusterConfig`：业务集群、日志集群分层配置。
	- `HGNewBusinessProducerOpts`：业务消息高可靠配置，默认 `acks=all`，使用 franz-go 默认幂等写入。
	- `HGNewLogProducerOpts`：日志/埋点高吞吐配置，强制 `acks=1`，使用 LZ4 优先压缩。
	- `HGGlobalKgoClient` / `HGInitKafka` / `HGCloseKafka`：全局长生命周期 `kgo.Client`，支持生产/消费复用。
	- `HGSendBusinessEvent`：同步发送业务事件，适合必须确认写入 Kafka 的核心链路。
	- `HGSendLogEvent`：异步发送日志/埋点事件，失败后尝试进入 DLQ。
	- `HGBaseConsumer`：消费基类，封装 `PollFetches`、手动提交 offset、panic 保护、DLQ 投递。
	- `HGSendDLQ`：统一死信投递，默认提供 `hg.dlq.business` / `hg.dlq.log`。
	- `HGInjectTraceToRecord` / `HGExtractTraceFromRecord`：通过 Kafka header 透传项目现有 TID。
	- `HGMetricHook` / `HGTraceHook`：franz-go hook 扩展点，当前提供轻量计数与 trace 扩展占位。
- 新增 `internal/pkg/config/hg_kafka_config.go`：从当前 viper 配置读取 Kafka 配置。
- 在 `config/config.debug.yaml`、`config/config.pre.yaml`、`config/config.prod.yaml` 中新增 Kafka 配置模板。
- 新增 `hg_kafka_application.go`，并在 `main_mlc_project.go` 中接入 Kafka 生命周期：
	- 启动时调用 `initKafkaIfConfigured()`。
	- 未配置 `kafka.business.brokers` 时跳过 Kafka 初始化。
	- 已配置 broker 但初始化失败时启动失败，避免请求期才暴露 MQ 不可用。
	- 应用关闭时调用 `HGCloseKafka()`，先 flush 再关闭 client。
- 补充较详细注释，说明：
	- `acks=1` 与 `acks=all` 的适用边界。
	- 高并发下为什么限制 `MaxBufferedRecords` / `MaxBufferedBytes`。
	- 为什么复用单 `kgo.Client`。
	- DLQ 在 Kafka 整体不可用时仍可能失败，需要生产兜底。
	- “千万并发/亿万数据”属于设计适配，不等于已完成真实压测验证。

<br/>

### 3. 为什么这样改

- 文档示例中的 `github.com/twmb/franz-go/pkg/kcompress` 在当前 franz-go 版本不存在；实际压缩 API 在 `kgo` 包内，所以使用 `kgo.Lz4Compression()` / `kgo.SnappyCompression()` / `kgo.NoCompression()`。
- 文档要求引入 `kadm`，但当前项目 `go.mod` 是 `go 1.23.5`，可用的 `kadm` 版本会要求更高 Go 版本或拉升 `kgo` 到不兼容组合。为避免擅自升级项目 Go 版本，本次先落地核心生产/消费能力，暂未接入 `kadm` 集群管理 API。
- franz-go `Client` 本身支持生产和消费复用，符合“单 Client 降低 broker 连接数”的目标。
- 高并发场景不能使用无界缓冲，所以生产者显式配置：
  - `kgo.MaxBufferedRecords(100000)`
  - `kgo.MaxBufferedBytes(512 * 1024 * 1024)`
- 业务消息和日志消息配置分级：
  - 业务消息优先可靠性。
  - 日志/埋点优先吞吐和削峰。
- 消费端采用手动提交 offset，只有 handler 成功后才 `CommitRecords`，避免处理失败后误提交。
- 工程启动接入采用“可选 Kafka”：本地、单测和暂未接入 MQ 的环境可以保持 `brokers: []`，生产配置 broker 后才真正初始化。

<br/>

### 4. MLC_GO 工程使用说明

#### 4.1 配置 Kafka

在 `config/config.debug.yaml`、`config/config.pre.yaml` 或 `config/config.prod.yaml` 中配置：

```yaml
kafka:
  business:
    brokers:
      - 127.0.0.1:9092
    acks: all
    retry: 3
    client_id: mlc-go-debug-business
  log:
    brokers:
      - 127.0.0.1:9092
    acks: "1"
    retry: 1
    client_id: mlc-go-debug-log
```

说明：

- `business.brokers` 为空时，应用启动会跳过 Kafka 初始化。
- `business.brokers` 非空时，启动阶段会初始化 `HGGlobalKgoClient`；初始化失败会阻止服务启动。
- `business.acks` 建议生产环境使用 `all`。
- `log.acks` 建议日志/埋点使用 `"1"`，吞吐更高，但极端故障窗口内存在少量丢失风险。
- `client_id` 建议包含服务名、环境和用途，方便 broker 侧排查连接来源。

<br/>

#### 4.2 发送业务事件

业务代码中使用上游请求 `ctx`，不要使用 `context.Background()` 替代请求上下文：

```go
err := HGKafkaPackage.HGSendBusinessEvent(
    ctx,
    "mlc.business.user.created",
    userID,
    payload,
)
if err != nil {
    return fmt.Errorf("发送用户创建事件失败: %w", err)
}
```

建议：

- `topic` 使用领域命名，例如 `mlc.business.user.created`。
- `key` 使用稳定业务 ID，例如 `user_id` / `order_id`，保证同一实体事件进入同一分区后有序。
- 上游 `ctx` 应带超时，避免 Kafka 异常时请求无限等待。

<br/>

#### 4.3 发送日志/埋点事件

```go
HGKafkaPackage.HGSendLogEvent(ctx, "mlc.log.access", payload)
```

说明：

- 该方法异步发送，不阻塞主业务链路。
- 发送失败会尝试进入 DLQ。
- 进程异常退出时，尚未 flush 的异步消息可能丢失，因此服务退出必须走 `MLCApplication.Close()`。

<br/>

#### 4.4 注册消费者

创建消费者时复用全局 Kafka Client，并在 handler 成功后由基类提交 offset：

```go
consumer := HGKafkaPackage.HGNewBaseConsumer(
    HGKafkaPackage.HGClient(),
    HGKafkaPackage.HGTopicDLQBusiness,
)

err := consumer.HGStartConsume(ctx, func(ctx context.Context, record *kgo.Record) error {
    var evt UserCreatedEvent
    if err := json.Unmarshal(record.Value, &evt); err != nil {
        return err
    }
    return userService.ConsumeUserCreated(ctx, evt)
})
```

注意：

- handler 返回 `nil` 后才提交 offset。
- handler 返回 error 时会尝试投递 DLQ，当前 offset 不提交，后续可能被重新消费。
- 消费者 client 需要在创建时配置 `kgo.ConsumeTopics(...)` 和 `kgo.ConsumerGroup(...)`；当前工程已提供消费基类，具体业务消费者接入时再按 topic/group 创建。

<br/>

#### 4.5 启动与关闭生命周期

- `main_mlc_project.go` 的 `buildMLCApplication()` 会调用 `initKafkaIfConfigured()`。
- `MLCApplication.Close()` 会调用 `HGCloseKafka()`。
- `HGCloseKafka()` 会先 `Flush` 再 `Close`，尽量发送完客户端缓冲中的异步消息。

### 5. 准确性检查结果

- TDD RED 阶段已执行：
	- `go test ./internal/pkg/kafka`
	- 初次失败原因为缺少 `github.com/twmb/franz-go/pkg/kgo`，符合新增依赖前的预期失败。
	- `go test ./internal/pkg/config`
	- 初次失败原因为缺少 `GetKafkaConfig`，符合先写测试再实现。
	- `go test . -run TestInitKafkaIfConfiguredSkipsEmptyConfig`
	- 初次失败原因为缺少 `initKafkaIfConfigured`，符合先写测试再实现。
- 修正 franz-go v1.16.0 API 差异：
	- `ProduceAsync` 在当前版本使用 `Produce(ctx, record, callback)`。
	- `ProducerMaxRetries` 在当前版本使用 `RecordRetries`。
	- `ProducerBatchMaxDuration` 在当前版本使用 `ProducerLinger`。
	- `AcksOne` / `AcksAll` 在当前版本使用 `LeaderAck()` / `AllISRAcks()`。
	- hook 签名按 v1.16.0 的 `HookProduceBatchWritten` 接口实现。
- Kafka 封装测试已通过：
	- `go test ./internal/pkg/kafka`
- 配置读取测试已通过：
	- `go test ./internal/pkg/config`
- 应用 Kafka 接入测试已通过：
	- `go test . -run TestInitKafkaIfConfiguredSkipsEmptyConfig`
- 业务入口编译通过：
  - `go build -o /tmp/mlc_go_kafka_verify .`

<br/>

### 6. 潜在影响

- 新增 `franz-go` 依赖，`go.mod` 已更新。
- 新增 Kafka 配置模板，但默认 `brokers: []`，因此不会改变本地和现有环境启动行为。
- 未修改现有 API、鉴权、数据库、Redis 业务语义。
- 当前未接入真实 Kafka broker 做集成测试，因此只能确认：
  - 代码可编译。
  - 封装逻辑和无 broker 单测通过。
  - franz-go Client 可按配置创建。
  - 工程启动逻辑可在未配置 Kafka 时跳过初始化。
- `kadm` 未接入，原因是当前 Go 版本约束。后续如要做 topic 创建、offset 查询、lag 管理，需要先确认是否允许升级 Go 版本或锁定兼容版本组合。
- DLQ 仍依赖 Kafka 可用。如果整个 Kafka 集群不可用，DLQ 也会失败，生产环境建议增加本地磁盘 WAL 或补偿任务。

<br/>

### 7. 格式化/编译/测试说明

- 已格式化：
	- `gofmt` 覆盖本次新增/修改 Go 文件。
	- 已执行并通过：
	- `go test ./internal/pkg/kafka`
	- `go test ./internal/pkg/config`
	- `go test . -run TestInitKafkaIfConfiguredSkipsEmptyConfig`
	- `go build -o /tmp/mlc_go_kafka_verify .`
- 已执行但存在既有失败：
	- `go test ./internal/...`
- `go test ./internal/...` 中 Kafka 包通过，但以下既有鉴权测试失败，和 Kafka 改动无关：
	- `MLC_GO/internal/modules/user/middleware`
	- `MLC_GO/internal/pkg/middleware`
- 失败原因是测试期望错误码 `101001`，实际返回 `300001`：
	- `TestAuthMiddleware_MissingTokenReturnStandardError`
	- `TestAuthMiddleware_DeviceMismatchHideInternalReason`
	- `TestTokenAuthMiddleware_MissingTokenReturnStandardError`

<br/>

### 8. 后续生产级优化建议

1. 接入真实 Kafka broker 后，增加集成测试，验证 produce、consume、commit、DLQ。
2. 为业务领域新增 topic/group 常量包，避免字符串散落。
3. 为消费者创建独立 client 构造函数，显式配置 `kgo.ConsumeTopics(...)` 和 `kgo.ConsumerGroup(...)`。
4. 增加压测脚本，验证 TPS、P99、失败率、broker throttle、客户端 buffer、GC、网络吞吐。
5. 接入 Prometheus/OpenTelemetry，把 `HGMetricHook` / `HGTraceHook` 对接真实监控系统。
6. 评估 Go 版本升级；如果要接入 `kadm` 集群管理 API，建议先确认是否升级项目 Go 版本到满足 `kadm` 要求的版本。


