> # go-redis
- [go-redis安装](#go-redis安装)
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

## 四、go.mod 完整示例

```go
// go.mod 文件示例
module github.com/yourusername/my-redis-project

go 1.19

require (
    github.com/redis/go-redis/v9 v9.0.5
)

require (
    github.com/cespare/xxhash/v2 v2.2.0 // indirect
    github.com/dgryski/go-rendezvous v0.0.0-20200823014737-9f7001d12a5f // indirect
)
```

## 五、性能优化建议

1. **连接池配置**：根据并发量调整 PoolSize
2. **使用 Pipeline**：批量操作减少网络往返
3. **合理使用上下文**：设置超时避免阻塞
4. **监控慢查询**：定期检查慢查询日志
5. **数据分片**：大数据量时考虑分片

## 六、常见问题解决

1. **连接超时**：检查网络和防火墙设置
2. **内存不足**：合理设置 maxmemory 策略
3. **键冲突**：使用命名空间前缀
4. **序列化问题**：使用 JSON 或 Protobuf

这个完整的指南应该能帮助你在 Mac 上顺利使用 go-redis 进行开发。