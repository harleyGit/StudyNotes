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


***
<br/><br/><br/>
> <h2 id="错误格式选择">错误格式选择</h2>

* `errors.New("...")`：生成一个**固定文本**的错误（常用于“哨兵错误 / sentinel error”）。
* `fmt.Errorf("...")`：像 `Sprintf` 一样**格式化**错误信息；若用 **`%w`** 包含另一个错误，就变成**包装（wrapping）**，便于后续用 `errors.Is/As` 判断原因。

<br/>

| 场景                  | 用法                                                            | 说明                                                 |
| ------------------- | ------------------------------------------------------------- | -------------------------------------------------- |
| 固定语义的可判定错误（包级常量/变量） | `var ErrNodeIDRange = errors.New("node-id must be [0,1024)")` | 作为“哨兵错误”在包外可被判定（`errors.Is(err, ErrNodeIDRange)`）。 |
| 给错误增加上下文（不关心底层类型）   | `fmt.Errorf("failed to lock %q", path)`                       | 只生成一条描述信息，不保留“因果链”。                                |
| 给错误增加上下文且**保留因果链**  | `fmt.Errorf("failed to lock %q: %w", path, err)`              | 用 **`%w`** 包装底层错误，之后可用 `errors.Is/As/Unwrap` 追溯。   |
| 动态信息但仍想可判定          | `fmt.Errorf("%w: got %d", ErrNodeIDRange, id)`                | 在哨兵错误外再附加细节。                                       |

> `fmt.Errorf("failed to lock data-path: %v", err)` **只拼文案**，不会建立可判定的“因果链”。若要后续判断底层错误，请改为 **`%w`**：
> `fmt.Errorf("failed to lock data-path: %w", err)`。

<br/>

**示例**

**1)校验 ID：哨兵错误 + 可判定**

```go
package node

import (
	"errors"
	"fmt"
)

var ErrNodeIDRange = errors.New("node-id must be [0,1024)")

func ValidateNodeID(id int) error {
	if id < 0 || id >= 1024 {
		// 保留可判定的语义（ErrNodeIDRange），同时携带细节
		return fmt.Errorf("%w: got %d", ErrNodeIDRange, id)
	}
	return nil
}
```

<br/>

调用方：

```go
if err := node.ValidateNodeID(2048); err != nil {
	if errors.Is(err, node.ErrNodeIDRange) {
		// 明确知道是“范围错误”
		// 可以返回 400、提示用户、或走特定分支
	}
	// 记录日志时打印完整文本
	// log.Printf("validate failed: %v", err) // node-id must be [0,1024): got 2048
}
```

<br/> 

**2) 加锁失败：保留根因（用 `%w` 包装）**

```go
package locker

import (
	"fmt"
	"os"
)

func Lock(path string) error {
	// 假设底层返回的是某个具体错误（例如 os.ErrExist）
	if err := doLock(path); err != nil {
		return fmt.Errorf("lock %q: %w", path, err) // 用 %w 才能被 Is/As 判断
	}
	return nil
}

func doLock(path string) error {
	// 仅示意：返回一个具体根因
	return os.ErrExist
}
```

<br/>

调用方可精确分支：

```go
err := locker.Lock("/data/nsqd")
if err != nil {
	switch {
	case errors.Is(err, os.ErrExist):
		// 目录/锁已存在（比如已有进程占用）
	case errors.Is(err, os.ErrPermission):
		// 权限问题
	default:
		// 其他未知问题
	}
}
```

> 如果这里用了 `%v`：`fmt.Errorf("lock %q: %v", path, err)`，**`errors.Is(err, os.ErrExist)` 将会失败**，因为没有建立“包装链”。

<br/> 

**3) `errors.As`：提取具体错误类型**

有时你关心**类型**而不是等值：

```go
type ErrRemote struct {
	Code int
	Msg  string
}
func (e *ErrRemote) Error() string { return fmt.Sprintf("remote: %d %s", e.Code, e.Msg) }

func call() error {
	return fmt.Errorf("rpc failed: %w", &ErrRemote{Code: 502, Msg: "bad gateway"})
}

if err := call(); err != nil {
	var r *ErrRemote
	if errors.As(err, &r) {
		// 拿到结构化信息 r.Code / r.Msg
	}
}
```

<br/> 

**4) 多个根因（Go 1.20+）：`errors.Join`**

```go
if e1 != nil && e2 != nil {
	return errors.Join(e1, e2) // 两个都是真根因
}
```

`errors.Is/As` 会对 join 后的错误逐个匹配。
<br/>

**规则/细节你可能会用到**

* **只在需要“可判定根因”时用 `%w`**；否则 `%v` 就够（例如仅日志用）。
* **一个 `fmt.Errorf` 格式串里只能有一个 `%w`**。
* `errors.New` 适合做**包级**哨兵错误（`var ErrXxx = errors.New("...")`）。不要在每次返回时都新建一个 `errors.New("...")` 再拿来比较，**那样比较会失败**；要和同一个包级变量比或用 `errors.Is`。
* 错误信息遵循 Go 惯例：**不用首字母大写**、**不以句号结尾**（日志行会拼接更多上下文）。

<br/>

**你给的两行对比（推荐写法）**

```go
// ✅ 推荐：保留根因
return fmt.Errorf("failed to lock data-path %q: %w", dataPath, err)

// ✅ 作为哨兵（包级变量）
var ErrNodeIDRange = errors.New("node-id must be [0,1024)")
// 返回时带细节但保持可判定性
return fmt.Errorf("%w: got %d", ErrNodeIDRange, id)
```




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


