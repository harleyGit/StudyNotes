- [**介绍**](#介绍)
- [**go-svc服务demo**](#go-svc服务demo)
	- [3个协议方法](#3个协议方法)


<br/><br/><br/>

***
<br/>

> <h1 id="介绍">介绍</h1>


`go-svc` 是 Go 语言中一个用于 **服务生命周期管理（service lifecycle management）** 的轻量库，适合用于构建长运行（long-running）服务应用，比如守护进程或后台服务。

它提供了标准的服务生命周期钩子：

* `Init`：初始化资源
* `Start`：启动服务
* `Stop`：优雅关闭

而且它还支持 **Unix/Linux 的信号处理（如 SIGTERM）**，可以干净地关闭服务。

<br/>

**📦 安装 go-svc**

```bash
go get github.com/judwhite/go-svc
```


<br/><br/><br/>

***
<br/>

> <h1 id="go-svc服务demo">go-svc服务demo</h1>


**一个典型的 go-svc 服务**

下面是一个完整示例，包括服务初始化、启动、停止，并在停止时优雅释放资源。

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/judwhite/go-svc/svc"
)

// MyService 实现 svc.Service 接口
type MyService struct {
    quit chan struct{}
}

func (s *MyService) Init(env svc.Environment) error {
    fmt.Println("Init: 初始化服务资源")
    s.quit = make(chan struct{})
    return nil
}

func (s *MyService) Start() error {
    fmt.Println("Start: 启动服务")

    // 启动一个 goroutine 执行任务
    go func() {
        for {
            select {
            case <-s.quit:
                fmt.Println("收到 quit 信号，停止服务 goroutine")
                return
            default:
                fmt.Println("服务运行中...")
                time.Sleep(2 * time.Second)
            }
        }
    }()
    return nil
}

func (s *MyService) Stop() error {
    fmt.Println("Stop: 停止服务")
    close(s.quit) // 通知 goroutine 停止
    return nil
}

func main() {
    service := &MyService{}

    // run 会阻塞，直到收到停止信号（如 Ctrl+C 或 SIGTERM）
    if err := svc.Run(service); err != nil {
        log.Fatal(err)
    }
    fmt.Println("服务已退出")
}
```


**🚦说明**

* `svc.Run(service)`：

  * 会调用 `Init()`
  * 然后调用 `Start()`
  * 监听信号（如 `SIGINT`, `SIGTERM`），当捕获到时，自动调用 `Stop()`

* `MyService` 中的 `quit` 是一个通道，作用是优雅通知 goroutine 停止。

<br/>

**📌 高级用法（带 context 和自定义信号）**

如果你想自定义服务的启动流程，例如配合 `context` 使用：

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())

    go func() {
        // 捕捉中断信号（比如 Ctrl+C）
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt, syscall.SIGTERM)
        <-c
        fmt.Println("收到终止信号")
        cancel()
    }()

    // 自定义服务控制逻辑
    service := &MyService{}
    if err := svc.RunContext(ctx, service); err != nil {
        log.Fatal(err)
    }
}
```

<br/>

**✅ 适用场景**

* 编写系统服务（如 Linux daemon）
* 编写微服务或中间件后台进程
* 实现长时间运行、需要优雅退出的服务

<br/>

**📚 总结**

| 生命周期方法  | 说明          |
| ------- | ----------- |
| `Init`  | 初始化资源，仅执行一次 |
| `Start` | 启动服务，可以异步任务 |
| `Stop`  | 优雅停止，释放资源   |


***
<br/><br/><br/>
> <h2 id="3个协议方法">3个协议方法</h2>


`go-svc` 框架的设计核心所在，是通过调用 `svc.Run(service)` 实现了对 `Init()`, `Start()`, `Stop()` 这三个生命周期方法的**自动调用**。下面我来详细解释**为什么你没写调用代码，它们却被自动调用了**。

<br/>

 **一切的关键在于：`svc.Run(service)` 做了什么？**

```go
if err := svc.Run(service); err != nil {
    log.Fatal(err)
}
```

你调用的是 `svc.Run(service)`，而这个 `Run` 方法接收的是一个实现了 `svc.Service` 接口的对象。

<br/>

**🎯 重点：`service` 实现了接口 `svc.Service`**

```go
type Service interface {
    Init(env Environment) error
    Start() error
    Stop() error
}
```

<br/>

你定义的：

```go
type MyService struct {
    quit chan struct{}
}

// 自动实现了上面接口：
func (s *MyService) Init(env svc.Environment) error { ... }
func (s *MyService) Start() error { ... }
func (s *MyService) Stop() error { ... }
```

<br/>

**🔍 Run(service) 的内部逻辑（源码简化版）**（伪代码简化）：

```go
func Run(service Service) error {
    // 捕捉系统中断信号（比如 Ctrl+C）
    signalChan := make(chan os.Signal, 1)
    signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)

    // 1. 调用 Init()
    err := service.Init(...)
    if err != nil {
        return err
    }

    // 2. 调用 Start()
    err = service.Start()
    if err != nil {
        return err
    }

    // 3. 等待中断信号
    <-signalChan

    // 4. 调用 Stop()
    return service.Stop()
}
```

> 所以你没直接调用 `Init`, `Start`, `Stop`，**是 `svc.Run()` 内部帮你自动调用的**。

<br/>

**🧩 总结流程图**

```
main() 调用 svc.Run(service)
         |
         ↓
  svc.Run(service)
    ├──> 调用 service.Init()
    ├──> 调用 service.Start()
    ├──> 等待 OS 信号（如 Ctrl+C）
    └──> 接收到信号后调用 service.Stop()
```

<br/>

**📦 为什么这么设计？**

这是典型的“**控制反转（IoC）**”模式，框架控制你的服务流程，而不是你手动控制：

* 你只要定义好“服务生命周期怎么处理”（通过实现接口）
* 框架来控制“什么时候执行这些处理逻辑”

这样你写的代码就非常清晰，也不容易出错。

<br/>

**✅ 如果你想手动调用这些方法**

你当然也可以不用 `svc.Run()`，自己手动调用这些方法，例如：

```go
func main() {
    service := &MyService{}

    _ = service.Init(nil) // 手动调用
    _ = service.Start()

    // 这里可以执行任务或阻塞

    _ = service.Stop()
}
```

但这样你就需要自己捕捉信号、处理退出逻辑、资源清理等，变得不如使用 `svc.Run()` 自动管理优雅。


