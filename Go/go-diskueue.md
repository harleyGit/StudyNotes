- [**‌介绍**](#介绍)
	-[多个生产者和消费者读写数据Demo](#多个生产者和消费者读写数据Demo) 





<br/><br/><br/>

***
<br/>

> <h1 id="介绍">介绍</h1>

`go-diskqueue` 是一个用 Go 写的 **磁盘消息队列**（disk-backed queue）库。

**安装依赖：**

```sh
go get github.com/nsqio/go-diskqueue
```

<br/>

**主要作用**

> 提供一个“像 channel 一样”的消息队列，但数据会写到 **磁盘**，即使程序崩溃/重启也不会丢。

适合用在：

* 内存队列太小，消息量大时需要落盘
* 程序意外退出后希望能恢复数据
* 做 **日志缓冲** / **异步任务队列**

<br/>

**工作原理**

- 1.生产者 `Put()` 消息，`go-diskqueue` 把消息写进磁盘文件（顺序写，追加模式）。
- 2.消费者 `ReadChan()` 读取消息，库内部会顺序读取文件内容。
- 3.读取过的数据达到阈值后，旧文件会被删除/回收。

<br/>

- **内部会维护：**
	* 当前写入文件和偏移量
	* 当前读取文件和偏移量
	* 文件滚动、GC、错误修复等逻辑

<br/>

**🧩 与其他队列的对比**

| 特性  | `go-diskqueue` | `chan` | Kafka/RabbitMQ |
| --- | -------------- | ------ | -------------- |
| 存储  | 磁盘             | 内存     | 分布式磁盘          |
| 可靠性 | 程序重启后仍可恢复      | 程序结束即丢 | 强一致性           |
| 性能  | 顺序写/读，速度中等     | 超快     | 较复杂            |
| 部署  | 嵌入到 Go 程序      | Go 内建  | 需要单独服务         |

<br/>

> `go-diskqueue` 介于 “内存 channel” 和 “Kafka/RabbitMQ” 之间，适合**单机**、轻量、对持久化有要求但不想引入外部服务的场景。

<br/>

**📝 使用示例**

```go
dq := diskqueue.New(
    "my-queue",
    "/tmp/diskqueue",
    1024,    // 最大消息大小 (bytes)
    1<<20,   // 最大队列大小 (bytes)
    2500,    // 同步到磁盘的阈值
    2*time.Second, // 同步时间间隔
    nil,
)

dq.Put([]byte("hello"))
msg := <-dq.ReadChan()
fmt.Println(string(msg)) // 输出 hello
```

<br/>

💡 **总结：**
`go-diskqueue` = **“磁盘版 channel”**

> 当消息太多放不下内存，又不想搭 Kafka/RabbitMQ 时，可以把 `go-diskqueue` 嵌到程序里，保证消息在磁盘里持久化，程序挂了也能继续消费。



***
<br/><br/><br/>
> <h2 id="多个生产者和消费者读写数据Demo">多个生产者和消费者读写数据Demo</h2>


下面演示一个示例，将消息写入磁盘并再读取出来。

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/nsqio/go-diskqueue"
)

func main() {
	// 创建一个磁盘队列
	// 参数分别是：
	//   name, dataPath, maxBytesPerFile, minMsgSize, maxMsgSize,
	//   syncEvery, syncTimeout, logger
	dq := diskqueue.New(
		"example",            // 队列名称
		"./data",             // 存储目录
		1024*1024,            // 每个数据文件最大 1MB
		1,                    // 最小消息大小
		1024,                 // 最大消息大小
		5,                    // 每写 5 条就刷盘一次
		2*time.Second,        // 或每 2s 刷盘
		log.New(log.Writer(), "[diskqueue] ", log.LstdFlags),
	)
	// 关闭队列（会把元信息刷盘，确保下次可恢复）
	defer dq.Close()

	// 启动 3 个生产者
	var wg sync.WaitGroup
	producerCount := 3
	consumerCount := 2
	msgPerProducer := 5

	for p := 0; p < producerCount; p++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			for i := 1; i <= msgPerProducer; i++ {
				body := fmt.Sprintf("producer-%d message-%d", id, i)
				if err := dq.Put([]byte(body)); err != nil {
					log.Println("put error:", err)
				} else {
					fmt.Println("produced:", body)
				}
				time.Sleep(100 * time.Millisecond)
			}
		}(p)
	}

	// 启动 2 个消费者
	// 启动 2 个消费者，支持超时退出
	// 实现「消费者在一段时间内没有新消息，就自动退出」的逻辑
	for c := 0; c < consumerCount; c++ {
	    wg.Add(1)
	    go func(id int) {
	        defer wg.Done()
	        for {
	            select {
	            case msg := <-dq.ReadChan():
	                fmt.Printf("[consumer-%d] got: %s\n", id, string(msg))
	
	            case <-time.After(2 * time.Second): // 2 秒没有消息就退出
	                fmt.Printf("[consumer-%d] no new message, exit\n", id)
	                // 若是想超出后持续等待，可以将return换成 continue
	                return
	            }
	        }
	    }(c)
	}


	wg.Wait()
	fmt.Println("all done!")
}
```

<br/>

- **💡 提示：**
	- 生产者每人发 5 条消息，总共 3*5 = 15 条。
	- 消费者一共 2 个，每个读一半。
	* `./data` 目录里会生成 `example.diskqueue.*` 文件，保存消息内容和元信息。
	* 如果程序中途崩溃，重启后用 `ReadChan()` 还能接着读未消费的消息。
	* `Put()` 和 `ReadChan()` 都是线程安全的，**不需要自己加锁**,可在多个 goroutine 中并发使用。
	* 程序结束后，可以去 ./data 看磁盘里的文件，里面保存了消息数据和队列状态。


<br/>

**说明**
- dq.ReadChan() 是一个 chan []byte，会阻塞直到有消息。
- time.After(d) 会返回一个 一次性的通道，d 时间后收到事件。
- select 会同时监听两个通道：
	- 有消息就立即处理；
	- 如果 d 时间内没有消息，time.After 会触发，消费者 return 退出。

***
<br/><br/>

💡 **其他技巧**

- 1.**持续等待但能超时重试**
   如果你不想退出，而是定期打印「等待中…」，可以在超时后继续循环，而不是 `return`：

   ```go
   case <-time.After(2 * time.Second):
       fmt.Printf("[consumer-%d] still waiting...\n", id)
       continue
   ```

<br/>

- 2.**手动停止整个队列**
   生产端完成后，可以 `dq.Close()` 或 `dq.Empty()`，然后消费者收到关闭信号就能退出。

<br/>

- 3.**控制内存占用**
   `go-diskqueue` 会自动把超出的消息落盘，所以不用担心消息太多占用内存；你可以通过 `maxBytesPerFile`、`syncEvery`、`syncTimeout` 等参数控制磁盘文件大小和写盘频率。

<br/>

这样，当队列中长时间没有新数据时，消费者就会优雅退出，不会一直阻塞在 `<-dq.ReadChan()` 上了 🎉




