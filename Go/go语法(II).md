- [**日志**](#日志)
	- [日志格式](#日志格式)
- [文件](#文件)
	- [文件锁](#文件锁)







<br/><br/><br/>

***
<br/>

> <h1 id="日志">日志</h1>


***
<br/><br/><br/>
> <h2 id="日志格式">日志格式</h2>

```go
// 创建一个带前缀和微秒时间戳的 Logger
logger := log.New(os.Stderr, "[MyApp] ", log.Ldate|log.Ltime|log.Lmicroseconds)

// 打印几条日志看看效果
logger.Println("启动服务中...")
logger.Println("连接数据库成功")
logger.Println("监听端口 8080")
```

**log：**

```sh
[MyApp] 2025/07/28 20:43:14.219516 启动服务中...
[MyApp] 2025/07/28 20:43:14.220162 连接数据库成功
[MyApp] 2025/07/28 20:43:14.220168 监听端口 8080
```

<br/>

```go
log.New(output io.Writer, prefix string, flag int)
```

你的代码中传的是：

* `os.Stderr`：输出位置为标准错误（也可以是文件，比如 `os.Stdout` 或日志文件）
* `opts.LogPrefix`：日志前缀，比如可以设置为 `"[NSQ] "`，每条日志前面会自动加上它
* `log.Ldate | log.Ltime | log.Lmicroseconds`：日志格式标志，表示：

  * `log.Ldate`：日志中加日期，如 `2025/07/28`
  * `log.Ltime`：日志中加时间，如 `10:15:45`
  * `log.Lmicroseconds`：加微秒，比如 `10:15:45.123456`


<br/>

**💡 这个 Logger 有啥用？**

* 统一格式打印日志（带日期、时间、微秒）
* 设置前缀区分不同模块（如 `[NSQD]`, `[NSQLookupd]`）
* 控制输出位置（终端或文件）
* 支持多种日志级别（通过自己封装）

<br/>

**拓展：写入到文件示例**

```go
f, _ := os.Create("/Users/ganghuang/HGFiles/GitHub/GoProject/src/MLC_GO/app.log")
logger01 := log.New(f, "[MyApp] ", log.Ldate|log.Ltime|log.Lmicroseconds)
logger01.Println("写入日志文件---------实打实的发送哈")
```


若是日志文件不存在，则会根据文件路径，自动创建。

**app.log**

```txt
[MyApp] 2025/07/28 20:43:14.220676 写入日志文件---------实打实的发送哈
```

不过这个文件有点问题，它会覆盖上一个打印的日志。


<br/><br/><br/>

***
<br/>

> <h1 id="文件">文件</h1>


***
<br/><br/><br/>
> <h2 id="文件锁">文件锁</h2>


```go
dl:= dirlock.New(dataPath),
```

是在创建一个 **目录锁（dir lock）** 的实例，防止**多个进程同时访问或写入同一个目录**，是 NSQ 或类似系统中常见的 **并发安全机制**。

br/>

 **`dirlock.New(dataPath)` 是干嘛的？**

它调用 `dirlock` 包中的 `New()` 方法，用来创建一个 **针对指定目录的锁**。

* `dataPath` 是一个字符串，表示某个目录路径（比如 `./data`）。
* `dirlock.New(dataPath)` 返回一个对象（或结构体），表示“我现在想要锁定这个目录”。

锁定通常是通过在该目录下创建一个 `.lock` 文件实现的。

br/> 

**✅ `dl: dirlock.New(dataPath),` 是干嘛的？**

这是 Go 中结构体赋值的语法（以 map/struct 风格写法），比如你定义一个结构体：

```go
type NSQD struct {
	dl *dirlock.DirLock
}
```

然后你创建这个结构体实例时这样写：

```go
nsqd := &NSQD{
	dl: dirlock.New(dataPath),
}
```

也就是说，这一行是在初始化某个组件（如 NSQD 或 NSQLookupd），并将目录锁绑定进去，保存在 `dl` 字段中。

<br/>

**假设你有一个 `dirlock` 包，我们自己模拟它：**

```go
package dirlock

import (
	"fmt"
)

type DirLock struct {
	Path string
}

func New(path string) *DirLock {
	fmt.Println("🔐 初始化目录锁:", path)
	return &DirLock{Path: path}
}
```

然后主程序：

```go
package main

import (
	"./dirlock"
)

type App struct {
	dl *dirlock.DirLock
}

func main() {
	app := &App{
		dl: dirlock.New("./data"),
	}

	fmt.Println("目录锁路径为：", app.dl.Path)
}
```

<br/> 

**用途总结：**

| 场景                  | 目的                               |
| ------------------- | -------------------------------- |
| 多个 NSQD 实例指向同一个数据目录 | 防止数据竞争或文件冲突（两个进程写一个文件会导致崩溃或数据丢失） |
| 检查目录是否已被其他进程占用      | `.lock` 文件 + 文件锁（fcntl/flock）机制  |
| 启动时先加锁，退出时释放        | 确保目录生命周期是独占的                     |


