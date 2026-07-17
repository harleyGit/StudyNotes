> # go-redis
- [go-redis安装](#go-redis安装)
- [go-redis 渐进式遍历 Scan](#go-redis渐进式遍历Scan)
- [Redis HIncrBy 状态计数](#Redis-HIncrBy-状态计数)
	- [HIncrBy 是什么](#HIncrBy-是什么)
	- [为什么不用 GET + SET](#为什么不用-GET--SET)
	- [视频上传系统中的用途](#视频上传系统中的用途)
- [在线高 QPS 业务不要依赖 Redis Scan](#在线高QPS业务不要依赖RedisScan)
- [Scan 通过 .Result 获取结果](#Scan通过-Result获取结果)
- [核心功能](#核心功能)
	- [单机模式](#单机模式)	
	- [集群模式](#集群模式)
- [基本数据类型操作](#基本数据类型操作)
	- [String类型](#String类型)	
		- [登录验证码字符串次数限制](#登录验证码字符串次数限制)
			- [Incr方法](#Incr方法)  
			- [Expire方法](#Expire方法) 
			- [Val方法](#Val方法) 
			- [优化建议](#优化建议)
	- [Hash类型](#Hash类型) 
	- [List类型](#List类型) 
	- [Set类型](#Set类型) 
		- [Redis ZSET 与 ZREM](#RedisZSET与ZREM)
	- [SortedSet类型](#SortedSet类型)
- [管道-Pipeline](#管道-Pipeline)
- [事务-Transaction](#事务-Transaction)
- [原子操作-Lua脚本](#原子操作-Lua脚本)
- [发布订阅](#发布订阅)
- [监控和指标](#监控和指标)
- [连接管理](#连接管理)
- [经典简单案例](#经典简单案例)
	- [分布式锁实现](#分布式锁实现)  
	- [缓存封装](#缓存封装)




<br/><br/><br/>

***
<br/>

> <h1 id="go-redis安装">go-redis安装</h1>

 **创建 Go 项目并安装 go-redis**

```bash
# 创建项目目录
mkdir my-redis-project
cd my-redis-project

# 初始化 Go module
go mod init github.com/yourusername/my-redis-project

# 安装 go-redis v9（最新稳定版）
go get github.com/redis/go-redis/v9

# 或者安装特定版本
go get github.com/redis/go-redis/v9@v9.0.5
```


***
<br/><br/><br/>

> <h2 id="Redis-HIncrBy-状态计数">Redis HIncrBy 状态计数</h2>

核心代码：

```go
err := redisClient.HIncrBy(
    ctx,
    videoStatusCounterKey(),
    status,
    delta,
).Err()
```

这句是 `go-redis` 中对 Redis **Hash 字段做原子自增/自减**的操作。

```go
HIncrBy(ctx, key, field, increment)
```

对应 Redis 原生命令：

```redis
HINCRBY key field increment
```

---
<br/>

> <h3 id="HIncrBy-是什么">HIncrBy 是什么</h3>

Redis 常用数据结构有 `String`、`List`、`Set`、`Hash`、`ZSet`。这里使用的是 `Hash`，类似 Go 中的：

```go
map[string]map[string]int64
```

例如：

```text
video_status_counter
{
    uploading: 100,
    completed: 200,
    failed: 5
}
```

Redis 中可以理解为：

```text
key
 |
 +---- field:value
 +---- field:value
```

假设：

```go
videoStatusCounterKey() // "video:status:counter"
status                  // "completed"
delta                   // 1
```

执行：

```go
HIncrBy(ctx, "video:status:counter", "completed", 1)
```

等价 Redis：

```redis
HINCRBY video:status:counter completed 1
```

执行前后：

```text
video:status:counter
completed = 100 -> 101
```

---
<br/>

> <h3 id="为什么不用-GET--SET">为什么不用 GET + SET</h3>

很多新人会写：

```go
count := redis.Get("completed")
count++
redis.Set("completed", count)
```

看似一样，但高并发下会丢数据：

```text
请求A: GET = 100
请求B: GET = 100

A: 100 + 1 = 101
B: 100 + 1 = 101

最终结果: 101
实际应该: 102
```

`HIncrBy` 是 Redis 原子操作，Redis 单线程顺序执行，不会丢失递增：

```text
请求A -> Redis -> +1
请求B -> Redis -> +1

结果: 102
```

---
<br/>

## `.Err()` 是什么

完整代码可以拆成：

```go
result := redisClient.HIncrBy(ctx, key, field, delta)
err := result.Err()
```

`HIncrBy` 返回 `*redis.IntCmd`，类似：

```go
type IntCmd struct {
    val int64
    err error
}
```

其中 `val` 保存 Redis 返回值。例如：

```redis
HINCRBY video:status:counter completed 1
```

返回：

```text
101
```

Go 中通过 `result.Val()` 获取：

```go
result.Val() // 101
```

`.Err()` 用来获取执行错误：

```go
err == nil           // 成功
redis.Nil            // 空值场景
connection timeout   // 网络或连接错误
```

所以：

```go
err := redisClient.HIncrBy(...).Err()
```

表示：**只关心 Redis 操作有没有失败，不关心增加后的数字。**

---
<br/>

## delta 是什么

`delta` 是变化量：

```go
delta := int64(1)  // +1
delta := int64(-1) // -1
```

自减时对应：

```redis
HINCRBY video:status:counter completed -1
```

---
<br/>

> <h3 id="视频上传系统中的用途">视频上传系统中的用途</h3>

结合 `video_files`、`video_submission`、`submission_id`，这里可能用于统计视频状态数量。

Redis 保存：

```text
video:status:counter
{
    uploading: 50000,
    completed: 900000,
    failed: 100
}
```

用户完成上传时，数据库更新：

```sql
UPDATE video_submission
SET status = 'completed'
WHERE id = 10001;
```

同时更新 Redis：

```go
redis.HIncrBy(ctx, "video:status:counter", "completed", 1)
```

结果：

```text
completed = 900000 -> 900001
```

状态迁移时，例如 `uploading -> completed`，不能只增加 `completed`，还要减少 `uploading`：

```text
uploading
    |
    v
completed
```

正确做法是同时调整两个状态：

```go
redis.TxPipeline(ctx, func(pipe redis.Pipeliner) error {
    pipe.HIncrBy(ctx, "video:status:counter", "uploading", -1)
    pipe.HIncrBy(ctx, "video:status:counter", "completed", 1)
    return nil
})
```

结果：

```text
uploading -1
completed +1
```

---
<br/>

## HIncrBy 和 HIncrByFloat 区别

整数计数用：

```go
HIncrBy()
```

对应 Redis：

```redis
HINCRBY
```

例如 `100 -> 101`。浮点计数用：

```go
HIncrByFloat()
```

例如 `1.5 -> 2.7`。

---
<br/>

## 大厂场景

字节、阿里、腾讯类似系统中，`Redis Hash + HIncrBy` 常用于：

### 实时计数

```text
video:view_count
{
    today: 10000000
}
```

### 状态统计

```text
order:status
{
    paid: 100000,
    unpaid: 200000
}
```

### 限流计数

```text
api:limit
{
    user_10001: 50
}
```

### 分片上传统计

```text
upload:counter:{submission_id}
{
    uploaded_parts: 100,
    uploaded_bytes: 52428800
}
```

所以：

```go
HIncrBy(ctx, videoStatusCounterKey(), status, delta).Err()
```

本质就是：**在 Redis Hash 中，对某个视频状态字段进行原子加减操作，并只检查是否执行成功。**在亿级数据、高并发上传系统里，这是非常常见的实时统计计数方案。



***
<br/><br/><br/>
> <h3 id="go-redis渐进式遍历Scan">go-redis 渐进式遍历 Scan</h3>

`go-redis` 的 `client.Scan(...)` 是 Redis `SCAN` 命令的 Go 封装，**通过游标 + 渐进扫描遍历 Key，避免 `KEYS *` 阻塞 Redis 主线程**，是亿级 Key 数据下唯一推荐的遍历方式。

```go
func (c cmdable) Scan(
    ctx context.Context,
    cursor uint64,
    match string,
    count int64,
) *ScanCmd
```

> 常见误解：`Scan = 遍历 Redis 所有 Key`。其实它对应的是 `SCAN`，设计目标是**不阻塞 Redis 的情况下渐进式遍历**。

---

## 为什么需要 Scan

`KEYS *` 会扫描全部 Key 并一次性返回，Redis 单线程一直工作，其它命令全部等待——生产环境几乎禁止。`SCAN` 不阻塞、每次扫描一点、可暂停可继续，是大厂标准方案。

---

## cursor 是什么

`cursor` **不是"第几页"**，而是 Redis 内部遍历 Hash Table 的游标位置。每次 `SCAN <cursor>` 由 Redis 自身算出下一次游标，`cursor == 0` 表示扫描结束。

```text
SCAN 0   → cursor: 18, keys: A C
SCAN 18  → cursor: 95, keys: D E
SCAN 95  → cursor: 0   → 结束
```

---

## match / count 参数

- `match`：模式过滤（`user:*` 只返回匹配 Key），但 Redis 仍需遍历整个哈希表，仅在返回时过滤，不能理解为索引查询。
- `count`：**Hint（建议值）**，不保证返回固定数量。`count=100` 可能返回 `78`、`132` 甚至 `3`，**千万不要用 `len(keys) == count` 判断结束**。

---

## 返回值与标准写法

`Scan()` 返回的是 `*ScanCmd` 命令对象，真正的数据通过 `.Result()` 取出。结束条件：**`nextCursor == 0`**。

```go
var cursor uint64
for {
    keys, nextCursor, err := client.Scan(
        ctx, cursor, "user:*", 100,
    ).Result()
    if err != nil {
        return err
    }
    for _, key := range keys {
        // 处理
    }
    cursor = nextCursor
    if cursor == 0 {
        break
    }
}
```

---

## 为什么不会阻塞 Redis

每次只扫描一小部分 Hash Bucket，执行时间通常几十微秒到几毫秒，其它客户端的 `GET/SET/DEL/INCR` 几乎不受影响：

```text
Bucket1 → 返回
Bucket2 → 返回
Bucket3 → 返回
```

---

## Scan 的关键特性

1. **可能返回重复 Key**：扫描期间可能发生 rehash 或 Key 被修改，业务要求每个 Key 只处理一次时需客户端去重。
2. **不保证遍历期间数据一致性**：新增 Key 可能扫到也可能扫不到，已删除 Key 同理。`SCAN` 的目标是**最终遍历**，不是一致性快照。
3. **大厂用法**：`SCAN` 用于**后台运维、缓存清理、数据迁移、灰度删除**，**不会用于在线业务查询**。

去重示例：

```go
seen := make(map[string]struct{})
for _, key := range keys {
    if _, ok := seen[key]; ok {
        continue
    }
    seen[key] = struct{}{}
    // process
}
```

---

## 一句话总结

> `Scan()` 是 Redis `SCAN` 命令的 Go 封装；`cursor` 是游标不是页码；`count` 是 Hint 不保证数量；结束标志是 `cursor == 0`；结果可能重复，不是一致性快照；只能用于后台任务，不能用于在线高 QPS 业务。


***
<br/><br/><br/>
> <h2 id="在线高QPS业务不要依赖RedisScan">在线高 QPS 业务不要依赖 Redis Scan</h2>

**核心原则**：Redis 不是 MySQL，**不应该靠"搜索"找数据**，而应通过 Key 设计 + 数据结构设计让数据可以 `O(1)` 或 `O(logN)` 获取。

```text
错误：我有什么数据？→ SCAN 找
正确：我需要什么数据？→ 提前维护索引 Key → 直接 GET/ZSET/HGET
```

---

## 错误方案：线上请求 Scan

需求是"获取某用户最近上传的视频"，但 Redis 只存 `video:10001`、`video:10002`...，然后：

```go
keys, _, _ := redis.Scan(ctx, 0, "video:*", 100)
for _, k := range keys {
    if video.UserID == uid { /* ... */ }
}
```

假设 Redis 有 10 亿 video Key，目标用户 `uid=888` 只有 20 个视频，但需要扫描 10 亿，复杂度 `O(N)`，并发一高 Redis CPU 直接爆炸。

---

## 正确方案：业务索引

设计思路：**数据实体 + 索引结构**，类似 MySQL `video` 表 + `index(user_id)`，Redis 自己维护索引。

### 案例1：用户视频列表

```text
视频详情：
  Key：   video:{video_id}
  Value：Hash { id, user_id, title, status, created_at }

用户视频索引（Sorted Set）：
  Key：   user:{user_id}:videos
  Score： 发布时间
  Member：video_id
```

查询流程 `GET /users/888/videos`：

```redis
ZREVRANGE user:888:videos 0 19    # 取最新 20 个 video_id
MGET video:10001 video:10002 ...  # 批量取详情
```

复杂度 `O(logN + M)`，没有搜索、没有遍历。

### 案例2：预约发布任务

```text
Key：   video:scheduled:queue
Score： 发布时间
Member：submission:10001
```

Worker 每秒执行：

```redis
ZRANGEBYSCORE video:scheduled:queue 0 <当前时间> LIMIT 0 100
```

发布成功后 `ZREM video:scheduled:queue submission:10001`，避免重复消费。

### 案例3：用户在线状态

```text
Key： online:users（Set）
上线：SADD online:users 10001
下线：SREM online:users 10001
查询：SCARD online:users
```

### 案例4：点赞数量

```text
Key：   video:{id}:likes（String）
增加：  INCR video:10001:likes
读取：  GET video:10001:likes
```

复杂度 `O(1)`。

### 案例5：排行榜

```text
Key：   video:hot（ZSET，score=热度）
查询：  ZREVRANGE video:hot 0 99   # Top100
```

---

## Go 代码示例

添加视频（Pipeline 一次写实体 + 索引）：

```go
func AddUserVideo(
    ctx context.Context,
    uid int64, videoID int64, publishTime int64,
) error {
    pipe := redis.TxPipeline(ctx)
    pipe.HSet(ctx, fmt.Sprintf("video:%d", videoID), map[string]interface{}{
        "user_id": uid, "status": "published",
    })
    pipe.ZAdd(ctx, fmt.Sprintf("user:%d:videos", uid), redis.Z{
        Score:  float64(publishTime),
        Member: videoID,
    })
    _, err := pipe.Exec(ctx)
    return err
}
```

查询：

```go
func GetUserVideos(ctx context.Context, uid int64) {
    ids, _ := redis.ZRevRange(
        ctx, fmt.Sprintf("user:%d:videos", uid), 0, 19,
    )
    // pipeline MGET
}
```

---

## Scan 使用场景速查

| 场景 | Scan |
|------|------|
| 线上接口查询 | ❌ |
| 用户列表/视频列表查询 | ❌ |
| 排行榜 | ❌ |
| 定时清理缓存 | ✅ |
| 迁移 Redis 数据 | ✅ |
| 统计 Key | ✅ |
| 后台运维 | ✅ |

推荐数据建模：

```text
MySQL  → 数据真相
Redis  → Entity Cache
         + List Index
         + Rank Index
         + Delay Queue
```


***
<br/><br/><br/>
> <h2 id="Scan通过-Result获取结果">Scan 通过 .Result 获取结果</h2>

`go-redis` API 风格高度统一：**`Scan()` 返回 `*ScanCmd` 命令对象，真正数据通过 `.Result()` 取出**。`Result()` 只是把 `cmd` 内部的 `page/cursor/err` 返回出来，**不会再访问 Redis**。

```go
type ScanCmd struct {
    baseCmd
    page   []string
    cursor uint64
}
```

```go
func (cmd *ScanCmd) Result() ([]string, uint64, error) {
    return cmd.page, cmd.cursor, cmd.err
}
```

---

## 调用流程

```text
client.Scan()
    → 创建 ScanCmd
    → 发送 SCAN 0 MATCH user:* COUNT 100
    → Redis 返回 cursor + keys
    → 解析 RESP 写入 ScanCmd
    → 返回 ScanCmd
Result()
    → 返回 keys, cursor, err（不再访问 Redis）
```

---

## 为什么不直接返回三值

为了和 go-redis 整体 API 保持一致——`Get → *StringCmd`、`Set → *StatusCmd`、`Incr → *IntCmd`、`HGetAll → *MapStringStringCmd`，全部都是"先取命令对象，再 `.Result()`"的模式。

**`ScanCmd` 没有 `Val()`** 是因为它有两个主要返回值（`keys` + `cursor`），无法用单个 `Val()` 表示。

---

## 其它取值方法

```go
cmd := client.Get(ctx, "name")
if err := cmd.Err(); err != nil { /* 只关心错误 */ }
name, err := cmd.Result()

cmd := client.Incr(ctx, "count")
n := cmd.Val()  // IntCmd 提供 Val()，直接拿值
```

---

## 工程意义

- **统一 API**：所有命令都返回对象 + `.Result()`，调用方式一致。
- **可扩展**：以后增加耗时、原始响应、重试次数等字段，无需修改函数签名。
- **便于 Pipeline / 事务**：先收集命令对象，统一 `Exec` 后再分别 `.Result()`。

Pipeline 延迟执行示例：

```go
pipe := rdb.Pipeline()
getCmd  := pipe.Get(ctx, "user:1")
scanCmd := pipe.Scan(ctx, 0, "user:*", 100)
_, err := pipe.Exec(ctx)        // 统一发送
name, err  := getCmd.Result()   // 分别取值
keys, cur, err := scanCmd.Result()
```

如果 `Scan()` 一开始就返回 `([]string, uint64, error)`，Pipeline 的延迟执行模式无法实现。



<br/><br/><br/>

***
<br/>

> <h1 id="核心功能">核心功能</h1>

***
<br/><br/><br/>
> <h2 id="单机模式">单机模式</h2>

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
    "time"
)

func main() {
    // 创建 Redis 客户端
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",      // Redis 地址
        Password: "",                    // 密码，没有则留空
        DB:       0,                     // 使用默认 DB
        PoolSize: 20,                    // 连接池大小
        MinIdleConns: 10,                // 最小空闲连接数
        MaxRetries:      3,              // 最大重试次数
        MinRetryBackoff: 8 * time.Millisecond,  // 重试最小间隔
        MaxRetryBackoff: 512 * time.Millisecond, // 重试最大间隔
        DialTimeout:  5 * time.Second,   // 连接超时
        ReadTimeout:  3 * time.Second,   // 读超时
        WriteTimeout: 3 * time.Second,   // 写超时
        PoolTimeout:  4 * time.Second,   // 连接池超时
        IdleTimeout:  5 * time.Minute,   // 空闲连接超时
    })

    ctx := context.Background()
    
    // 测试连接
    pong, err := rdb.Ping(ctx).Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("连接成功:", pong)
}
```

***
<br/><br/><br/>
> <h2 id="集群模式">集群模式</h2>

```go
// 连接 Redis 集群
clusterRdb := redis.NewClusterClient(&redis.ClusterOptions{
    Addrs: []string{
        "localhost:7000",
        "localhost:7001",
        "localhost:7002",
    },
    Password: "",
    PoolSize: 20,
})

// 哨兵模式
sentinelRdb := redis.NewFailoverClient(&redis.FailoverOptions{
    MasterName:    "mymaster",
    SentinelAddrs: []string{
        "localhost:26379",
        "localhost:26380",
    },
    Password: "",
    DB: 0,
})
```

<br/><br/><br/>

***
<br/>

> <h1 id="基本数据类型操作">基本数据类型操作</h1>

***
<br/><br/><br/>
> <h2 id="String类型">String类型</h2>

```go
// SET/GET
err := rdb.Set(ctx, "key", "value", 0).Err()
if err != nil {
    panic(err)
}

val, err := rdb.Get(ctx, "key").Result()
if err == redis.Nil {
    fmt.Println("key 不存在")
} else if err != nil {
    panic(err)
} else {
    fmt.Println("key", val)
}

// 带过期时间的 SETEX
rdb.SetEx(ctx, "key2", "value2", time.Hour)

// 设置多个值 MSET
rdb.MSet(ctx, "key1", "value1", "key2", "value2")

// 获取多个值 MGET
vals := rdb.MGet(ctx, "key1", "key2").Val()
for i, val := range vals {
    fmt.Printf("key%d: %v\n", i+1, val)
}

// INCR/DECR
rdb.Set(ctx, "counter", 0, 0)
rdb.Incr(ctx, "counter")      // 1
rdb.IncrBy(ctx, "counter", 5) // 6
rdb.Decr(ctx, "counter")      // 5
```


<br/><br/>
> <h3 id="登录验证码字符串次数限制">登录验证码字符串次数限制</h3>


<br/><br/>
> <h3 id="Incr方法">Incr方法</h3>

- **作用：**
	- 原子性地将 Redis 中指定 key 的值增加 1。如果 key 不存在，则会先初始化为 0，然后执行增加操作。
- 返回值类型：
	- `*redis.IntCmd` - 这是 go-redis 库中的一个类型，包含了命令执行的结果

```go
// 代码中的使用
p := r.rdb.Incr(ctx, phoneKey)  // phoneKey 的值 +1
i := r.rdb.Incr(ctx, ipKey)      // ipKey 的值 +1

// 等效于 Redis 命令：
// INCR phoneKey
// INCR ipKey
```

<br/><br/>
> <h3 id=" Expire方法">Expire方法</h3>
- **作用：**
	- 为指定的 key 设置过期时间，时间到达后 key 会被自动删除。
- **参数：**
	- `ctx context.Context`：上下文
	- `key string`：要设置过期时间的 key
	- `expiration time.Duration`：过期时间（这里是 1 分钟）

```go
// 代码中的使用
r.rdb.Expire(ctx, phoneKey, time.Minute)  // 设置 phoneKey 1 分钟后过期
r.rdb.Expire(ctx, ipKey, time.Minute)      // 设置 ipKey 1 分钟后过期

// 等效于 Redis 命令：
// EXPIRE phoneKey 60
// EXPIRE ipKey 60
```

<br/><br/>
> <h3 id="Val方法">Val方法</h3>
- **作用：**
	- 从 `*redis.IntCmd` 中获取命令执行的整数值。

```go
// 代码中的使用
if p.Val() > 5 {  // 获取 phoneKey 的当前值，判断是否大于 5
    return errors.New("手机号发送过于频繁")
}

// 完整流程：
// 1. 第一次调用：phoneKey 不存在，Incr 后值变为 1，Val() = 1
// 2. 第二次调用：phoneKey = 1，Incr 后值变为 2，Val() = 2
// ...
// 6. 第六次调用：phoneKey = 5，Incr 后值变为 6，Val() = 6，触发限制
```

<br/>

**完整示例代码：**

```go
package main

import (
    "context"
    "fmt"
    "time"
    "github.com/go-redis/redis/v8"
)

type RateLimiter struct {
    rdb *redis.Client
}

func (r *RateLimiter) CheckLimit(ctx context.Context, phone, ip string) error {
    phoneKey := fmt.Sprintf("sms:phone:%s", phone)
    ipKey := fmt.Sprintf("sms:ip:%s", ip)
    
    // 执行 Incr 操作
    p := r.rdb.Incr(ctx, phoneKey)
    i := r.rdb.Incr(ctx, ipKey)
    
    // 设置过期时间（只有第一次设置时需要）
    r.rdb.Expire(ctx, phoneKey, time.Minute)
    r.rdb.Expire(ctx, ipKey, time.Minute)
    
    // 检查限制
    if p.Val() > 5 {
        return fmt.Errorf("手机号 %s 发送过于频繁", phone)
    }
    
    if i.Val() > 10 {
        return fmt.Errorf("IP %s 发送过于频繁", ip)
    }
    
    return nil
}

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    
    limiter := &RateLimiter{rdb: rdb}
    ctx := context.Background()
    
    // 模拟连续调用
    for i := 1; i <= 7; i++ {
        err := limiter.CheckLimit(ctx, "13800138000", "192.168.1.1")
        if err != nil {
            fmt.Printf("第 %d 次调用: %v\n", i, err)
            break
        }
        fmt.Printf("第 %d 次调用: 成功\n", i)
    }
}
```

<br/>

**输出结果：**

```
第 1 次调用: 成功
第 2 次调用: 成功
第 3 次调用: 成功
第 4 次调用: 成功
第 5 次调用: 成功
第 6 次调用: 手机号 13800138000 发送过于频繁
```

<br/><br/>
> <h3 id="优化建议">优化建议</h3>
- **问题：当前代码存在竞态条件**
	- 在 Incr 和 Expire 之间如果程序崩溃，可能导致 key 永不过期。

<br/>

**改进方案 1：使用管道（Pipeline）保证原子性**

```go
func (r *RateLimiter) CheckLimitOptimized(ctx context.Context, phone, ip string) error {
    phoneKey := fmt.Sprintf("sms:phone:%s", phone)
    ipKey := fmt.Sprintf("sms:ip:%s", ip)
    
    // 使用管道批量执行命令
    pipe := r.rdb.Pipeline()
    p := pipe.Incr(ctx, phoneKey)
    i := pipe.Incr(ctx, ipKey)
    pipe.Expire(ctx, phoneKey, time.Minute)
    pipe.Expire(ctx, ipKey, time.Minute)
    
    // 执行所有命令
    _, err := pipe.Exec(ctx)
    if err != nil {
        return err
    }
    
    // 检查结果
    if p.Val() > 5 {
        return fmt.Errorf("手机号 %s 发送过于频繁", phone)
    }
    
    if i.Val() > 10 {
        return fmt.Errorf("IP %s 发送过于频繁", ip)
    }
    
    return nil
}
```

<br/>

- **改进方案 2：使用 Lua 脚本（最优解）**

```go
var checkLimitScript = redis.NewScript(`
    local phoneKey = KEYS[1]
    local ipKey = KEYS[2]
    local phoneLimit = tonumber(ARGV[1])
    local ipLimit = tonumber(ARGV[2])
    local expireTime = tonumber(ARGV[3])
    
    local phoneCount = redis.call("INCR", phoneKey)
    local ipCount = redis.call("INCR", ipKey)
    
    if phoneCount == 1 then
        redis.call("EXPIRE", phoneKey, expireTime)
    end
    
    if ipCount == 1 then
        redis.call("EXPIRE", ipKey, expireTime)
    end
    
    if phoneCount > phoneLimit then
        return {"phone_limit", phoneCount}
    end
    
    if ipCount > ipLimit then
        return {"ip_limit", ipCount}
    end
    
    return {"ok", phoneCount, ipCount}
`)

func (r *RateLimiter) CheckLimitLua(ctx context.Context, phone, ip string) error {
    phoneKey := fmt.Sprintf("sms:phone:%s", phone)
    ipKey := fmt.Sprintf("sms:ip:%s", ip)
    
    result, err := checkLimitScript.Run(ctx, r.rdb, []string{phoneKey, ipKey}, 
        5, 10, 60).Result()
    if err != nil {
        return err
    }
    
    resp := result.([]interface{})
    if resp[0] == "phone_limit" {
        return fmt.Errorf("手机号发送过于频繁，当前次数: %v", resp[1])
    }
    if resp[0] == "ip_limit" {
        return fmt.Errorf("IP发送过于频繁，当前次数: %v", resp[1])
    }
    
    return nil
}
```


***
<br/><br/><br/>
> <h2 id="Hash类型">Hash类型</h2>

```go
// HSET/HGET
rdb.HSet(ctx, "user:1000", "name", "张三", "age", 30)

name := rdb.HGet(ctx, "user:1000", "name").Val()
age := rdb.HGet(ctx, "user:1000", "age").Int()

// HMSET/HMGET
rdb.HMSet(ctx, "user:1001", 
    "name", "李四",
    "email", "lisi@example.com",
    "age", 25,
)

fields := rdb.HMGet(ctx, "user:1001", "name", "email").Val()

// 获取所有字段 HGETALL
userData := rdb.HGetAll(ctx, "user:1001").Val()
for field, value := range userData {
    fmt.Printf("%s: %s\n", field, value)
}

// HINCRBY
rdb.HIncrBy(ctx, "user:1000", "age", 1)
```

***
<br/><br/><br/>
> <h2 id="List类型">List类型</h2>

```go
// LPUSH/RPUSH
rdb.LPush(ctx, "mylist", "world")
rdb.RPush(ctx, "mylist", "hello")

// LRANGE
items := rdb.LRange(ctx, "mylist", 0, -1).Val()
for _, item := range items {
    fmt.Println(item)
}

// LINDEX
first := rdb.LIndex(ctx, "mylist", 0).Val()

// LLEN
length := rdb.LLen(ctx, "mylist").Val()
```

***
<br/><br/><br/>
> <h2 id="Set类型">Set类型</h2>


```go
// SADD/SMEMBERS
rdb.SAdd(ctx, "myset", "apple", "banana", "orange")
members := rdb.SMembers(ctx, "myset").Val()

// SISMEMBER
isMember := rdb.SIsMember(ctx, "myset", "apple").Val()

// 集合运算
rdb.SAdd(ctx, "set1", "a", "b", "c")
rdb.SAdd(ctx, "set2", "b", "c", "d")

// 交集 SINTER
intersection := rdb.SInter(ctx, "set1", "set2").Val()

// 并集 SUNION
union := rdb.SUnion(ctx, "set1", "set2").Val()

// 差集 SDIFF
diff := rdb.SDiff(ctx, "set1", "set2").Val()
```


***
<br/><br/><br/>
> <h2 id="RedisZSET与ZREM">Redis ZSET 与 ZREM</h2>

`ZSET`（Sorted Set）= 有序集合，每个 member 带一个 score 用于排序；`ZREM` = 删除 ZSET 中的指定 member。是排行榜、延迟队列、Feed 流的核心结构。

```redis
ZADD key score member
ZREM key member
```

---

## 与 Set 的区别

| 类型 | 特点 | 典型用途 |
|------|------|----------|
| Set | 无序，去重 | 标签、去重、在线状态 |
| ZSET | 按 score 排序，去重 | 排行榜、延迟队列、时间排序 |

```redis
SADD users 1001 1002 1003            # Set，无顺序
ZADD users 100 1001 90 1002 80 1003  # ZSET，按 score 排序
```

score 可以是任意 double，member 是去重的。

---

## 核心概念

- **member**：排序的对象，如 `video:10001`、`submission:10001`。
- **score**：排序依据，如发布时间 `1782907200`、热度 `1000`。

```redis
ZADD video:hot 1000 video:10001
```

---

## 视频系统典型应用

**热门视频排行**：

```redis
ZADD video:hot 1200 video:3 999 video:1 800 video:2
ZREVRANGE video:hot 0 99   # Top100
```

**预约发布**（按时间排序的任务队列）：

```redis
ZADD video:scheduled 1783684800 submission:10001
```

Worker 每秒取到期任务：

```redis
ZRANGEBYSCORE video:scheduled 0 <now> LIMIT 0 100
```

发布成功后**必须 `ZREM`**，否则下一秒会再次被消费，导致重复发布 / 重复发通知 / 重复写库。

完整流程：

```text
Redis ZSET → 到期任务 → 发布服务 → MySQL 更新 status=published → ZREM 删除任务
```

---

## Go 用法

```go
// 添加任务
err := rdb.ZAdd(ctx, "video:scheduled", redis.Z{
    Score:  float64(publishTime.Unix()),
    Member: submissionID,
}).Err()

// 取到期任务
tasks, err := rdb.ZRangeByScore(ctx, "video:scheduled", &redis.ZRangeBy{
    Min:   "0",
    Max:   strconv.FormatInt(time.Now().Unix(), 10),
    Count: 100,
}).Result()

// 删除任务
rdb.ZRem(ctx, "video:scheduled", submissionID)
```

---

## 底层为什么快

```text
Hash Table   → member -> score，O(1) 查找
SkipList     → 按 score 排序，O(logN) 范围查询
```

| 操作 | 复杂度 |
|------|--------|
| `ZADD` | `O(logN)` |
| `ZREM` | `O(logN)` |
| 范围查询 | `O(logN + M)` |

---

## 视频系统 Redis 建模推荐

```text
video:{id}            Hash     视频详情
user:{uid}:videos     ZSET     用户视频列表（score=create_time）
video:hot             ZSET     热门视频（score=hot_score）
video:scheduled       ZSET     预约发布（score=publish_timestamp）
video:{id}:likes      String   点赞数
```

**ZSET 是 Redis 的"排序索引"**，ZREM 是删除排序索引中的元素；大厂延迟任务系统最常见的 Redis 建模方式。


***
<br/><br/><br/>
> <h2 id="SortedSet类型">SortedSet类型</h2>

```go
// ZADD
rdb.ZAdd(ctx, "leaderboard", redis.Z{
    Score:  100,
    Member: "player1",
})

// 批量添加
members := []redis.Z{
    {Score: 200, Member: "player2"},
    {Score: 150, Member: "player3"},
}
rdb.ZAdd(ctx, "leaderboard", members...)

// ZRANGE 按分数升序
top3 := rdb.ZRange(ctx, "leaderboard", 0, 2).Val()

// ZREVRANGE 按分数降序
topPlayers := rdb.ZRevRangeWithScores(ctx, "leaderboard", 0, 2).Val()
for _, z := range topPlayers {
    fmt.Printf("%s: %.0f\n", z.Member, z.Score)
}

// ZSCORE 获取成员分数
score := rdb.ZScore(ctx, "leaderboard", "player1").Val()
```


<br/><br/><br/>

***
<br/>

> <h1 id="管道-Pipeline">管道-Pipeline</h1>

```go
// 使用管道批量执行命令
pipe := rdb.Pipeline()

incr := pipe.Incr(ctx, "counter")
pipe.Expire(ctx, "counter", time.Minute)
pipe.Set(ctx, "key", "value", 0)

// 执行所有命令
cmds, err := pipe.Exec(ctx)
if err != nil {
    panic(err)
}

// 获取结果
fmt.Println("counter:", incr.Val())
```

<br/><br/><br/>

***
<br/>

> <h1 id="事务-Transaction">事务-Transaction</h1>

```go
// 使用 MULTI/EXEC 事务
tx := rdb.TxPipeline()

tx.Set(ctx, "key1", "value1", 0)
tx.Set(ctx, "key2", "value2", 0)
tx.Incr(ctx, "counter")

_, err := tx.Exec(ctx)
if err != nil {
    panic(err)
}

// Watch 实现乐观锁
err = rdb.Watch(ctx, func(tx *redis.Tx) error {
    // 获取当前值
    n, err := tx.Get(ctx, "counter").Int()
    if err != nil && err != redis.Nil {
        return err
    }

    // 在事务中执行操作
    _, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
        pipe.Set(ctx, "counter", n+1, 0)
        return nil
    })
    return err
}, "counter")
```

<br/><br/><br/>

***
<br/>

> <h1 id="原子操作-Lua脚本">原子操作-Lua脚本</h1>

```go
// 执行 Lua 脚本
script := `
    local key = KEYS[1]
    local limit = tonumber(ARGV[1])
    local current = redis.call('GET', key)
    
    if current == false then
        redis.call('SET', key, 1, 'EX', 60)
        return 1
    end
    
    if tonumber(current) >= limit then
        return 0
    end
    
    redis.call('INCR', key)
    return 1
`

// 预加载脚本
sha, err := rdb.ScriptLoad(ctx, script).Result()
if err != nil {
    panic(err)
}

// 使用 EVALSHA 执行
result, err := rdb.EvalSha(ctx, sha, []string{"rate:limit"}, 10).Result()
if err != nil {
    panic(err)
}

fmt.Println("Result:", result)
```

<br/><br/><br/>

***
<br/>

> <h1 id="发布订阅">发布订阅</h1>

```go
// 订阅者
pubsub := rdb.Subscribe(ctx, "mychannel")
defer pubsub.Close()

// 接收消息
ch := pubsub.Channel()
for msg := range ch {
    fmt.Printf("Channel: %s, Payload: %s\n", msg.Channel, msg.Payload)
}

// 发布者
rdb.Publish(ctx, "mychannel", "hello world")
```

<br/><br/><br/>

***
<br/>

> <h1 id="监控和指标">监控和指标</h1>

```go
// 获取 Redis 信息
info := rdb.Info(ctx).Val()
fmt.Println(info)

// 获取内存信息
memoryInfo := rdb.Info(ctx, "memory").Val()

// 获取慢查询
slowLogs := rdb.SlowLogGet(ctx, 10).Val()
for _, log := range slowLogs {
    fmt.Printf("慢查询: %s, 耗时: %v\n", log.Args, log.Duration)
}

// 监控指标
stats := rdb.PoolStats()
fmt.Printf("连接池统计: TotalConns=%d, IdleConns=%d, StaleConns=%d\n",
    stats.TotalConns, stats.IdleConns, stats.StaleConns)
```

<br/><br/><br/>

***
<br/>

> <h1 id="连接管理">连接管理</h1>


```go
// 关闭连接
rdb.Close()

// 连接状态检查
if rdb.Ping(ctx).Err() != nil {
    fmt.Println("Redis 连接断开")
}

// 优雅关闭
func gracefulShutdown(rdb *redis.Client) {
    // 停止接受新连接
    // 等待正在处理的请求完成
    
    // 关闭 Redis 连接
    if err := rdb.Close(); err != nil {
        fmt.Printf("关闭 Redis 连接失败: %v\n", err)
    }
}
```

<br/><br/><br/>

***
<br/>

> <h1 id="经典简单案例">经典简单案例</h1>

***
<br/><br/><br/>
> <h2 id="分布式锁实现">分布式锁实现</h2>


```go
package redislock

import (
    "context"
    "errors"
    "github.com/redis/go-redis/v9"
    "time"
)

type RedisLock struct {
    rdb     *redis.Client
    key     string
    token   string
    timeout time.Duration
}

func NewRedisLock(rdb *redis.Client, key string, timeout time.Duration) *RedisLock {
    return &RedisLock{
        rdb:     rdb,
        key:     key,
        token:   generateToken(),
        timeout: timeout,
    }
}

func (l *RedisLock) Lock(ctx context.Context) (bool, error) {
    // 使用 SET NX EX 获取锁
    ok, err := l.rdb.SetNX(ctx, l.key, l.token, l.timeout).Result()
    if err != nil {
        return false, err
    }
    return ok, nil
}

func (l *RedisLock) Unlock(ctx context.Context) error {
    // 使用 Lua 脚本确保原子性删除
    script := `
        if redis.call("GET", KEYS[1]) == ARGV[1] then
            return redis.call("DEL", KEYS[1])
        else
            return 0
        end
    `
    
    result, err := l.rdb.Eval(ctx, script, []string{l.key}, l.token).Result()
    if err != nil {
        return err
    }
    
    if result.(int64) == 0 {
        return errors.New("解锁失败: 锁不存在或token不匹配")
    }
    
    return nil
}

func generateToken() string {
    // 生成唯一 token
    return time.Now().String() // 实际应用中应该使用 UUID
}
```

***
<br/><br/><br/>
> <h2 id="缓存封装">缓存封装</h2>

```go
package cache

import (
    "context"
    "encoding/json"
    "time"
    "github.com/redis/go-redis/v9"
)

type RedisCache struct {
    rdb    *redis.Client
    prefix string
}

func NewRedisCache(rdb *redis.Client, prefix string) *RedisCache {
    return &RedisCache{rdb: rdb, prefix: prefix}
}

func (c *RedisCache) Set(ctx context.Context, key string, value interface{}, expiration time.Duration) error {
    data, err := json.Marshal(value)
    if err != nil {
        return err
    }
    
    fullKey := c.prefix + ":" + key
    return c.rdb.Set(ctx, fullKey, data, expiration).Err()
}

func (c *RedisCache) Get(ctx context.Context, key string, dest interface{}) (bool, error) {
    fullKey := c.prefix + ":" + key
    data, err := c.rdb.Get(ctx, fullKey).Bytes()
    if err == redis.Nil {
        return false, nil
    }
    if err != nil {
        return false, err
    }
    
    return true, json.Unmarshal(data, dest)
}

func (c *RedisCache) Delete(ctx context.Context, key string) error {
    fullKey := c.prefix + ":" + key
    return c.rdb.Del(ctx, fullKey).Err()
}

// 批量删除
func (c *RedisCache) DeletePattern(ctx context.Context, pattern string) error {
    iter := c.rdb.Scan(ctx, 0, c.prefix+":"+pattern, 0).Iterator()
    var keys []string
    
    for iter.Next(ctx) {
        keys = append(keys, iter.Val())
    }
    
    if err := iter.Err(); err != nil {
        return err
    }
    
    if len(keys) > 0 {
        return c.rdb.Del(ctx, keys...).Err()
    }
    
    return nil
}
```

<br/><br/><br/>

***
<br/>

> <h1 id="性能优化建议">性能优化建议</h1>


1. **连接池配置**：根据并发量调整 PoolSize
2. **使用 Pipeline**：批量操作减少网络往返
3. **合理使用上下文**：设置超时避免阻塞
4. **监控慢查询**：定期检查慢查询日志
5. **数据分片**：大数据量时考虑分片

<br/><br/><br/>

***
<br/>

> <h1 id="常见问题解决">常见问题解决</h1>

1. **连接超时**：检查网络和防火墙设置
2. **内存不足**：合理设置 maxmemory 策略
3. **键冲突**：使用命名空间前缀
4. **序列化问题**：使用 JSON 或 Protobuf
