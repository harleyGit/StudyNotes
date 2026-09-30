- 🍎
- [文本](#文本)
	- [strings.Builder 字符串清洗](#stringsBuilder字符串清洗)
- [集合](#集合)
	- [seen 去重与 map 预分配](#seen去重与map预分配)
- [新类型和别名](#新类型和别名)
	- [新类型不可以使用原类型的方法](#新类型不可以使用原类型的方法)
- [范型](#范型)
	- [范型推断](#范型推断)
- [**日志**](#日志)
	- [日志格式](#日志格式)
- [文件](#文件)
	- [文件锁](#文件锁)
	- [osMkdirAll递归创建目录](#osMkdirAll递归创建目录)
- [数据解析](#数据解析)
	- [json.RawMessage 延迟解析](#jsonRawMessage延迟解析)
	- [`json.RawMessage` 详解](#json.RawMessage详解)
		- [核心原理](#核心原理)
		- [延迟解析](#延迟解析)
		- [与其他类型的区别](#与其他类型的区别)
		- [Kafka 事件消息](#Kafka事件消息)
		- [HTTP API 与原样透传](#HTTPAPI与原样透传)
		- [按字段延迟解析](#按字段延迟解析)
		- [注意事项](#注意事项)
- [终止型错误](#终止型错误)
	- [`errors.As` 的匹配条件](#errors.As的匹配条件)
	- [使用与风险](#终止型错误的使用与风险)



<br/><br/><br/>

***
<br/>

> <h1 id="文本">文本</h1>


***
<br/><br/><br/>
> <h3 id="stringsBuilder字符串清洗">strings.Builder 字符串清洗</h3>

`strings.Builder` 是 Go 官方推荐的高性能字符串拼接工具，适合在循环中逐字符构造新字符串。相比 `s += string(r)`，它复用底层缓冲区，能减少反复分配和拷贝。

```go
func filterLowerOnly(value string) string {
    var builder strings.Builder
    builder.Grow(len(value))

    for _, r := range value {
        switch {
        case r >= 'a' && r <= 'z':
            builder.WriteRune(r)
        // 其他字符全部忽略，不写入
        default:
            continue
        }
    }

    return builder.String()
}
```

逐段含义：

| 代码 | 作用 |
|------|------|
| `var builder strings.Builder` | 创建字符串构造器 |
| `builder.Grow(len(value))` | 按原始字符串字节长度预分配容量，减少扩容 |
| `for _, r := range value` | 按 Unicode 字符 `rune` 遍历字符串 |
| `switch {}` | 无表达式 `switch`，等价于多段 `if/else if` 条件判断 |
| `builder.WriteRune(r)` | 把当前字符写入 Builder |

示例：输入 `User_Image/123.png`，如果只保留小写英文字母，结果是 `sermagepng`；大写字母、下划线、斜杠、数字、点号都会被过滤。

这个模式常用于 `sanitizePathPart` 之类的路径清洗函数：只保留允许字符，过滤中文、符号、斜杠、空格等非法路径片段，降低路径穿越和非法文件名风险。

注意：`len(value)` 是字节长度，不是字符数量；作为 `Grow` 的容量预估通常没问题，因为最终字符串不会比原始字节更长。



<br/><br/><br/>

***
<br/>

> <h1 id="集合">集合</h1>

***
<br/><br/><br/>
> <h2 id="seen去重与map预分配">seen 去重与 map 预分配</h2>


`seen` 常用于列表组装时按 `id` 去重：第一次遇到某个 `id` 就记录到 map，后面再次遇到同一个 `id` 时直接 `continue` 跳过。

```go
seen := make(map[string]struct{}, limit)

for rows.Next() {
    item, err := scanAdminUserRow(rows, hasEmail)
    if err != nil {
        return err
    }

    id := item["id"].(string)
    if _, ok := seen[id]; ok {
        continue
    }

    seen[id] = struct{}{}
    list = append(list, item)

    if len(list) >= limit {
        break
    }
}

return rows.Err()
```

执行过程：

```text
第一次 id=100：
seen = {}
_, ok := seen["100"] → ok=false
seen["100"] = struct{}{}

第二次 id=100：
_, ok := seen["100"] → ok=true
continue，跳过重复数据
```

---
<br/>

## make(map[string]struct{}, limit)

下面两种写法的类型完全一样，都是 `map[string]struct{}`：

```go
seen := make(map[string]struct{}, limit)
seen := map[string]struct{}{}
```

区别只在初始化方式。`make(map[string]struct{}, limit)` 的第二个参数对 map 来说不是长度，也不能通过 `cap(m)` 读取，而是**初始容量提示（capacity hint）**：告诉 Go 预计后面大约会放 `limit` 个元素，提前准备哈希桶，减少扩容和 rehash。

```go
seen := make(map[string]struct{}, 100)
fmt.Println(len(seen)) // 0
```

`len(seen)` 仍然是 `0`，因为 map 里还没有元素；第二个参数只是预估容量，不是已有数据数量。

---
<br/>

## map 扩容成本

没有容量提示时，持续插入大量 key 可能触发多次扩容：

```text
开始
↓
很小的 Hash Bucket
↓
放满
↓
扩容
↓
重新计算 Hash
↓
搬迁数据
↓
继续插入
↓
再次扩容
```

扩容涉及新内存分配、Rehash、Bucket 搬迁和数据复制。项目中最多只会收集 `limit` 个管理员 ID，因此 `make(map[string]struct{}, limit)` 是合理的性能优化。

对比：

| 写法 | 类型 | 是否预分配 | 推荐场景 |
|------|------|------------|----------|
| `map[string]struct{}{}` | `map[string]struct{}` | 否 | 数据量未知、小型程序、示例代码 |
| `make(map[string]struct{})` | `map[string]struct{}` | 否 | 与字面量等价，初始化空 map |
| `make(map[string]struct{}, limit)` | `map[string]struct{}` | 是（容量提示） | 数据量已知或可预估，项目代码推荐 |

---
<br/>

## 为什么 value 是 struct{}

`map[string]struct{}` 表示只关心 key 是否存在，不关心 value。`struct{}{}` 是空结构体，占用 `0` 字节：

```go
unsafe.Sizeof(struct{}{}) // 0
```

相比 `map[string]bool`，`map[string]struct{}` 更适合表达集合（Set）语义：

```go
seen[id] = struct{}{}
if _, ok := seen[id]; ok {
    continue
}
```

结论：**`seen := make(map[string]struct{}, limit)` 同时表达了去重集合和预估容量，是 Go 项目中常见且推荐的写法**。


<br/><br/><br/>

***
<br/>

> <h1 id="新类型和别名">新类型和别名</h1>

```go
type Experiment string

var AllExperiments = []Experiment{
	TopologyAwareConsumption,
}
```

***
<br/><br/>

**新类型和别名的区别：**

在 Go 里 `type` 关键字既可以 **定义新类型**，也可以 **创建类型别名**，它们长得像，但语义完全不一样。

<br/>

**1️⃣ 定义“新类型”**

```go
type Experiment string
```

* 这是一个**全新的类型**，名字是 `Experiment`，底层是 `string`。
* 它和 `string` 是不同的类型，编译器会做严格区分。
* 只能显式转换：

```go
var s string = "abc"
var e Experiment = Experiment(s) // 需要显式转换
```

* 你可以给 `Experiment` 定义自己的方法：

```go
func (e Experiment) Enabled() bool { return e != "" }
```

***
<br/><br/>

**2️⃣ 创建“别名”**

```go
type Experiment = string
```

* 这是一个 **别名**，等价于 `string`，只是多了一个名字。
* 编译器不会当作新类型，`Experiment` 与 `string` 可以直接互换：

```go
var s string = "abc"
var e Experiment = s     // 直接赋值
fmt.Println(reflect.TypeOf(e)) // 输出 string
```

* 不能给别名单独定义方法（因为底层还是 `string`）。

<br/> 

**3️⃣ 区别总结**

| 特性     | `type Experiment string`（新类型） | `type Experiment = string`（别名） |
| ------ | ----------------------------- | ------------------------------ |
| 与原类型关系 | **不同类型**                      | **同一个类型**                      |
| 赋值/传参  | 需要显式转换                        | 可直接赋值                          |
| 方法定义   | ✅ 可以给 `Experiment` 定义方法       | ❌ 不能定义                         |
| 语义     | 表示一个概念不同的值域，编译器做类型检查          | 只是为了写代码时简化名称                   |

<br/> 

**4️⃣ 对你提到的例子**

```go
type Experiment string

const TopologyAwareConsumption Experiment = "topology-aware-consumption"
```

这里选 **新类型** 而不是别名，是为了让 `TopologyAwareConsumption` 这种常量：

* 不能随便和普通 `string` 混用。
* 能当成独立的“实验枚举”来约束函数参数，例如：

```go
func HasExperiment(e Experiment) bool { ... }
```

这样 `HasExperiment("abc")` 会直接编译报错，避免错误调用。


***
<br/><br/><br/>
> <h2 id="新类型不可以使用原类型的方法">新类型不可以使用原类型的方法</h2>

这是 Go 里一个很容易混淆的点 😊

> **简短答案**：
> ➡️ **不会自动继承**底层类型的方法，但可以用显式转换把值转成底层类型，再调用它的方法。

<br/>

**1️⃣ 规则**

* 当你写：

```go
type Experiment string
```

`Experiment` 是一个“基于 `string` 的新类型”。
它**不自动拥有** `string` 的任何方法（即使底层是一样的）。

Go 的方法集只会包含：

* 该类型自己定义的方法；
* 它所嵌入（embedding）的字段的方法。

<br/>

**比如：**

```go
type MyString string

func (m MyString) Upper() string {
    return strings.ToUpper(string(m)) // 这里显式转回 string
}

func main() {
    var s MyString = "hello"

    fmt.Println(s.Upper())      // ✅ 调用我们自己定义的方法
    fmt.Println(len(s))         // ✅ 内建函数可以直接用
    fmt.Println(s.HasPrefix("h")) // ❌ 编译错误：MyString 没有 HasPrefix
}
```

`strings.HasPrefix` 是 `strings` 包里的普通函数，你可以写：

```go
strings.HasPrefix(string(s), "h") // ✅ 显式转换成 string 后再用
```

<br/>

**如果想“继承”方法怎么办？**

Go 没有继承，但有 **类型嵌入（embedding）**：

```go
type MyString struct {
    string
}

func main() {
    var s MyString
    s.string = "hello"
    fmt.Println(strings.HasPrefix(s.string, "h")) // 访问内嵌字段
}
```

不过这会把 `MyString` 变成结构体，不再是单纯的“新字符串类型”。


<br/><br/><br/>

***
<br/>

> <h1 id= "范型">范型</h1>

***
<br/><br/><br/>
> <h2 id="范型推断">范型推断</h2>

```go
func SuccessResult[T any](w http.ResponseWriter, r *http.Request, data T) {

	result := HGResultModel[T]{
		Code:      OKCode,
		Message:   "success💯",
		Result:    data,
		TID:       UtilsPackage.GetTID(r.Context()),
		Timestamp: time.Now().UnixMilli(),
	}

	writeResult(result, w)
}
```

**提问：** 那为什么在下面的函数中不需要在调用的时候指明范型的类型：

```go
// 调用
HGResponsePakcage.SuccessResult(w, r, resp)
```
***
<br/>


这个涉及 **Go 泛型的类型推导（type inference）机制**。

不需要写成：

```go
HGResponsePakcage.SuccessResult[HGPageResultModel[*UserDtoPackage.HGCreateUserDTO]](w, r, resp)
```

就能正常工作。

---
<br/>

因为**Go 编译器能从实参 `resp` 的类型自动推导出泛型参数 `T`，所以无需显式指定。**

<br/>

**函数定义是：**

```go
func SuccessResult[T any](w http.ResponseWriter, r *http.Request, data T)
```
<br/>

**调用：**

```go
SuccessResult(w, r, resp)
```

其中 `resp` 的类型是：

```go
HGResponsePakcage.HGPageResultModel[*UserDtoPackage.HGCreateUserDTO]
```

Go 编译器会做以下事情：

1. 看到第三个参数 `data T` 被传入了 `resp`；
2. 推断出：为了让 `data T` 能接收 `resp`，**`T` 必须等于 `resp` 的类型**；
3. 因此自动将 `T` 推导为 `HGPageResultModel[*UserDtoPackage.HGCreateUserDTO]`；
4. 整个函数实例化为：
   ```go
   func SuccessResult[HGPageResultModel[*UserDtoPackage.HGCreateUserDTO]](...)
   ```

这个过程叫做 **类型推导（Type Inference）**，是 Go 泛型的核心便利特性之一。

---
<br/>

**🆚 对比：什么时候必须显式指定泛型参数？**

只有当 **编译器无法从参数中推断出 `T`** 时，才需要手动指定。

**❌ 例子：无参数或参数不包含 `T`**

```go
func NewList[T any]() []T {
    return make([]T, 0)
}

// 错误：无法推断 T 是什么
list := NewList() // ❌ 编译错误！

// 正确：必须显式指定
list := NewList[string]() // ✅
```
<br/>

**✅ 你的场景：参数包含 `T`**

```go
func SuccessResult[T any](..., data T)
//                    ↗
//             data 的类型 = T
```
→ 所以 **有足够信息推导 `T`**，无需手动写。

<br/>

**🧪 验证：尝试传不同类型的 `resp`**

如果你传一个完全无关的类型，比如 `int`：

```go
HGResponsePakcage.SuccessResult(w, r, 42)
```

它依然能编译！因为 `T` 被推导为 `int`，生成的是：

```go
HGResultModel[int]{ Result: 42, ... }
```

这也说明了：**只要参数能匹配，Go 就能推导泛型类型**。


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


***
<br/><br/><br/>
> <h3 id="osMkdirAll递归创建目录">os.MkdirAll 递归创建目录</h3>

`os.MkdirAll(dir, 0755)` 用于递归创建多级目录：不存在的父目录会一并创建，目录已存在时不会报错，适合上传文件按模块、日期分目录存储的场景。

```go
func MkdirAll(path string, perm fs.FileMode) error
```

示例：

```go
saveDir := fmt.Sprintf("./upload/%s/%s", moduleName, timeStr)
if err := os.MkdirAll(saveDir, 0755); err != nil {
    return nil, fmt.Errorf("创建存储目录失败: %w", err)
}
```

如果目标路径是 `./upload/user/20260704`，即使 `upload`、`user` 都不存在，`MkdirAll` 也会一次性创建完整目录链。`os.Mkdir` 只能创建最后一级目录，父目录不存在会直接失败。

---
<br/>

## 0755 目录权限

`0755` 是八进制权限，拆成三段：所有者、同组用户、其他用户。

| 数字 | 权限 | 含义 |
|------|------|------|
| `4` | `r` | 读 |
| `2` | `w` | 写 |
| `1` | `x` | 执行；对目录表示可进入 |

`0755` 表示：

| 对象 | 权限 | 说明 |
|------|------|------|
| 所有者 | `7 = 4+2+1` | 可读、可写、可进入 |
| 同组用户 | `5 = 4+1` | 可读、可进入，不可修改 |
| 其他用户 | `5 = 4+1` | 可读、可进入，不可修改 |

目录的 `x` 权限表示能否进入目录；只有 `r` 没有 `x`，即使能看到目录名，也无法正常访问目录内容。

---
<br/>

## 和 os.Mkdir 的区别

```go
// 递归创建多级目录，推荐上传存储场景使用
err := os.MkdirAll("./upload/avatar", 0755)

// 只能创建最后一级，父目录不存在会报错
err := os.Mkdir("./upload/avatar", 0755)
```

注意点：Go 中 `0755` 前面的 `0` 不能省略，`0755` 表示八进制；直接写 `755` 会被当成十进制，权限含义完全不同。Windows 基本忽略 `perm` 参数；如果目录只允许程序自己读写，可用 `0700`。





<br/><br/><br/>

***
<br/>

> <h1 id="数据解析">数据解析</h1>


***
<br/><br/><br/>
> <h3 id="jsonRawMessage延迟解析">json.RawMessage 延迟解析</h3>

`json.RawMessage` 是 `[]byte` 的别名，用来**暂存未解析的原始 JSON 片段**，延迟到具体使用时再二次解析。`map[string]json.RawMessage` 只会拆顶层 key，子内容原样保留字节，适合**只关心部分字段、嵌套结构复杂**的场景。

```go
var raw map[string]json.RawMessage
if err := json.Unmarshal(data, &raw); err != nil {
    return err
}
```

直观示例，原始 JSON：

```json
{
  "name": "张三",
  "info": {"age":18, "sex":"男"},
  "images": ["a.png","b.jpg"]
}
```

执行 `Unmarshal` 后：

```text
raw["name"]   = []byte(`"张三"`)
raw["info"]   = []byte(`{"age":18, "sex":"男"}`)
raw["images"] = []byte(`["a.png","b.jpg"]`)
```

所有 value 都是原始未拆解的 JSON 字节，不会自动转成结构体/切片。

---

## 按需二次解析

取出 `info` 字段后单独反序列化为结构体：

```go
infoByte := raw["info"]
var info struct {
    Age int    `json:"age"`
    Sex string `json:"sex"`
}
if err := json.Unmarshal(infoByte, &info); err != nil {
    return err
}
```

优势是**只对真正用到的字段做二次解析**，避免一次性递归拆解整个 JSON，对大体积或嵌套复杂的请求体能提升性能。

---

## 报错场景

`json.Unmarshal(data, &raw)` 返回 `err != nil` 的常见原因：

1. `data` 不是合法 JSON（如 `[]` 数组、字符串、数字）；
2. JSON 格式非法（缺逗号、括号不匹配、转义错误）；
3. 顶层是 JSON 数组而非对象，无法映射到 `map[string]json.RawMessage`——顶层必须是 `{}` 对象。

---

## 解析方案对比

| 方案 | 写法 | 特点 |
|------|------|------|
| 普通 map | `map[string]string` | 字段是对象/数组时直接失败，无法承载复杂子结构 |
| 延迟解析 | `map[string]json.RawMessage` | 不管子字段是字符串、数字、对象还是数组，全部原样存储，按需二次解析 |
| 完整结构体 | `struct{...}` | 一次性解析所有字段；多余字段直接丢弃，扩展不灵活 |

---

## 业务典型用法

接收前端复杂参数，只想先取顶层部分字段、其它字段按需解析：

```go
var raw map[string]json.RawMessage
if err := json.Unmarshal(bodyBytes, &raw); err != nil {
    return err
}

var imageStr string
if err := json.Unmarshal(raw["image"], &imageStr); err != nil {
    return err
}
```

---

## 关键总结

1. `json.RawMessage` = 原始 JSON 字节占位容器，延迟解析。
2. `map[string]json.RawMessage` 适合**只关心部分字段、嵌套复杂 JSON**的场景，性能更好。
3. 只会解析顶层键值对，子 JSON 内容保留原始字节。
4. 顶层必须是 `{}` 对象，顶层为 `[]` 数组会解析报错。


<br/>

***
<br/><br/><br/>
> <h1 id="json.RawMessage详解"><code>json.RawMessage</code> 详解</h1>

`json.RawMessage` 用于保存一段合法 JSON 的原始字节，并把这部分内容留到确定具体类型后再解析。它常用于动态 JSON、Kafka 事件消息、HTTP 多态请求和网关透传。

核心写法：

```go
type Event struct {
	EventType string          `json:"event_type"`
	Data      json.RawMessage `json:"data"`
}

var event Event
if err := json.Unmarshal(body, &event); err != nil {
	return err
}

switch event.EventType {
case "video_created":
	var data VideoCreated
	if err := json.Unmarshal(event.Data, &data); err != nil {
		return err
	}
case "user_created":
	var data UserCreated
	if err := json.Unmarshal(event.Data, &data); err != nil {
		return err
	}
}
```

```text
第一次 Unmarshal
        ↓
解析公共字段
        ↓
Data 保留为 json.RawMessage
        ↓
根据 event_type 判断具体类型
        ↓
第二次 Unmarshal
        ↓
得到具体结构体
```

这种方式称为**延迟解析（deferred decoding）**，更准确地说是**局部延迟解析**：外层 JSON 已经解析，只是不继续解析 `Data` 字段。

***
<br/><br/><br/>

> <h2 id="核心原理">核心原理</h2>

源码定义：

```go
type RawMessage []byte
```

`json.RawMessage` 的底层类型是 `[]byte`，但它实现了 `json.Marshaler` 和 `json.Unmarshaler`。因此，`encoding/json` 会把它作为 JSON 原文处理，而不是普通字节切片。

```go
raw := json.RawMessage(`{"name":"Harley","age":30}`)
fmt.Println(string(raw))
```

输出：

```json
{"name":"Harley","age":30}
```

反序列化到 `RawMessage` 时，其 `UnmarshalJSON` 会复制输入数据，因此结果不会直接引用解码器传入的临时字节。

### 为什么不直接使用 `[]byte`

```go
type Event struct {
	Type string `json:"type"`
	Data []byte `json:"data"`
}
```

普通 `[]byte` 在 `encoding/json` 中按 Base64 字符串处理，不能直接接收 JSON 对象作为原始内容；`json.RawMessage` 则明确表示“这里保存的是 JSON 本身”。

例如，普通 `[]byte` 的 JSON 形式通常是：

```json
{"data":"aGVsbG8="}
```

而 `RawMessage` 可以直接保存对象：

```json
{"data":{"id":1}}
```

***
<br/><br/><br/>

> <h2 id="延迟解析">延迟解析</h2>

假设同一消息入口可能接收两种事件：

```json
{
    "event_type": "video_created",
    "data": {
        "video_id": 10001,
        "title": "Go Kafka"
    }
}
```

```json
{
    "event_type": "user_created",
    "data": {
        "user_id": 20001,
        "name": "Harley"
    }
}
```

先定义公共信封和具体载荷：

```go
type Event struct {
	EventType string          `json:"event_type"`
	Data      json.RawMessage `json:"data"`
}

type VideoCreated struct {
	VideoID int64  `json:"video_id"`
	Title   string `json:"title"`
}

type UserCreated struct {
	UserID int64  `json:"user_id"`
	Name   string `json:"name"`
}
```

第一次反序列化只解析公共字段：

```go
var event Event
if err := json.Unmarshal(body, &event); err != nil {
	return err
}
```

此时 `event.EventType` 是 `video_created`，`event.Data` 仍保存以下 JSON：

```json
{
    "video_id": 10001,
    "title": "Go Kafka"
}
```

再根据事件类型反序列化载荷：

```go
switch event.EventType {
case "video_created":
	var data VideoCreated
	if err := json.Unmarshal(event.Data, &data); err != nil {
		return err
	}

case "user_created":
	var data UserCreated
	if err := json.Unmarshal(event.Data, &data); err != nil {
		return err
	}
}
```

```text
                    JSON
                     |
                     v
              ┌──────────────┐
              │ Event        │
              │              │
              │ event_type   │
              │ data RawMsg  │
              └──────┬───────┘
                     |
            根据 event_type
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
   video_created          user_created
          ↓                     ↓
  VideoCreated             UserCreated
```

### 完整示例

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Event struct {
	EventType string          `json:"event_type"`
	Data      json.RawMessage `json:"data"`
}

type VideoCreated struct {
	VideoID int64  `json:"video_id"`
	Title   string `json:"title"`
}

type UserCreated struct {
	UserID int64  `json:"user_id"`
	Name   string `json:"name"`
}

func main() {
	body := []byte(`{
        "event_type": "video_created",
        "data": {
            "video_id": 10001,
            "title": "Go Kafka"
        }
    }`)

	var event Event
	if err := json.Unmarshal(body, &event); err != nil {
		panic(err)
	}

	fmt.Println(event.EventType)
	fmt.Println(string(event.Data))

	switch event.EventType {
	case "video_created":
		var data VideoCreated
		if err := json.Unmarshal(event.Data, &data); err != nil {
			panic(err)
		}

		fmt.Println(data.VideoID)
		fmt.Println(data.Title)

	case "user_created":
		var data UserCreated
		if err := json.Unmarshal(event.Data, &data); err != nil {
			panic(err)
		}

		fmt.Println(data.UserID)
		fmt.Println(data.Name)
	}
}
```

输出：

```text
video_created
{"video_id":10001,"title":"Go Kafka"}
10001
Go Kafka
```

***
<br/><br/><br/>

> <h2 id="与其他类型的区别">与其他类型的区别</h2>

| 字段类型 | 解码结果 | 适用场景 |
| --- | --- | --- |
| `any` / `interface{}` | 通常为 `map[string]any`、`[]any` 等通用类型 | 需要立即操作未知结构 |
| `json.RawMessage` | 保存 JSON 原始字节 | 需要延迟解析或原样透传 |
| 具体结构体 | 直接得到强类型值 | JSON 结构固定且已知 |
| `[]byte` | 按 Base64 JSON 字符串处理 | JSON 字段本身表示二进制数据 |

使用 `any`：

```go
type Event struct {
	Type string `json:"type"`
	Data any    `json:"data"`
}
```

对象类型的 `Data` 通常会解码为 `map[string]any`，其中 JSON 数字默认成为 `float64`。如果只想先判断事件类型，再按具体结构解析，`RawMessage` 能避免先解码成通用结构后再转换。

```text
any
  ↓
立即解析为通用 Go 值

json.RawMessage
  ↓
保留 JSON 字节，稍后按目标类型解析
```

***
<br/><br/><br/>

> <h2 id="Kafka事件消息">Kafka 事件消息</h2>

同一 Kafka Topic 中可能包含 `video_created`、`video_deleted`、`user_created`、`comment_created` 等不同事件。将所有业务字段平铺到一个结构体会产生大量无关字段，更适合使用“公共信封 + RawMessage 载荷”：

```go
type Event struct {
	ID        string          `json:"id"`
	EventType string          `json:"event_type"`
	Timestamp int64           `json:"timestamp"`
	Data      json.RawMessage `json:"data"`
}
```

消息示例：

```json
{
    "id": "evt_10001",
    "event_type": "video_created",
    "timestamp": 1750000000,
    "data": {
        "video_id": 10001,
        "title": "Kafka"
    }
}
```

Consumer 先解析信封，再分发给对应处理器：

```go
func handleMessage(data []byte) error {
	var event Event
	if err := json.Unmarshal(data, &event); err != nil {
		return err
	}

	switch event.EventType {
	case "video_created":
		return handleVideoCreated(event.Data)
	case "user_created":
		return handleUserCreated(event.Data)
	default:
		return fmt.Errorf("unknown event type: %s", event.EventType)
	}
}

func handleVideoCreated(data json.RawMessage) error {
	var event VideoCreated
	if err := json.Unmarshal(data, &event); err != nil {
		return err
	}

	// 业务处理
	return nil
}
```

该模式将公共路由信息与业务载荷解耦，适合事件驱动系统；生产环境还应校验事件版本、未知类型策略和载荷字段。

***
<br/><br/><br/>

> <h2 id="HTTPAPI与原样透传">HTTP API 与原样透传</h2>

同一个 HTTP 接口接收多种请求类型时，也可以先解析公共字段：

```json
{
    "type": "purchase",
    "data": {
        "product_id": 10001,
        "quantity": 2
    }
}
```

```go
type Request struct {
	Type string          `json:"type"`
	Data json.RawMessage `json:"data"`
}

func handler(w http.ResponseWriter, r *http.Request) {
	var req Request
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "invalid json", http.StatusBadRequest)
		return
	}

	switch req.Type {
	case "login":
		var data LoginRequest
		if err := json.Unmarshal(req.Data, &data); err != nil {
			http.Error(w, "invalid login data", http.StatusBadRequest)
			return
		}

		// login

	case "purchase":
		var data PurchaseRequest
		if err := json.Unmarshal(req.Data, &data); err != nil {
			http.Error(w, "invalid purchase data", http.StatusBadRequest)
			return
		}

		// purchase
	}
}
```

如果中间层不需要理解 `Data`，可以直接重新编码整个结构：

```go
data := json.RawMessage(`{
    "id": 10001,
    "title": "hello"
}`)

event := Event{
	EventType: "video",
	Data:      data,
}

result, err := json.Marshal(event)
if err != nil {
	return err
}

fmt.Println(string(result))
```

输出：

```json
{"event_type":"video","data":{"id":10001,"title":"hello"}}
```

```text
Gateway
   ↓
接收 JSON
   ↓
解析公共字段
   ↓
RawMessage 保存业务 payload
   ↓
Kafka
   ↓
Service
   ↓
业务 Service 再解析
```

这里的“原样透传”是指保持 JSON 的结构和值，不保证重新 `Marshal` 后仍保留原始空白、缩进等文本格式。

***
<br/><br/><br/>

> <h2 id="按字段延迟解析">按字段延迟解析</h2>

`map[string]json.RawMessage` 适合只关心部分字段、但又不想把所有内容立即解码为 `any` 的场景：

```go
var fields map[string]json.RawMessage
if err := json.Unmarshal(data, &fields); err != nil {
	return err
}
```

输入：

```json
{
    "name": "Harley",
    "age": 30,
    "profile": {
        "city": "Shanghai"
    }
}
```

解析结果可理解为：

```text
name    → "Harley"
age     → 30
profile → {"city":"Shanghai"}
```

只解析需要的字段：

```go
var name string
if err := json.Unmarshal(fields["name"], &name); err != nil {
	return err
}

var profile Profile
if err := json.Unmarshal(fields["profile"], &profile); err != nil {
	return err
}
```

读取前应判断 key 是否存在，否则缺失字段得到的 `nil` RawMessage 会导致 `json.Unmarshal` 返回 `unexpected end of JSON input`。

***
<br/><br/><br/>

> <h2 id="注意事项">注意事项</h2>

### 内容必须是合法 JSON

`RawMessage` 可以保存 JSON 对象、数组、数字、布尔值、`null` 和 JSON 字符串，但不能把普通文本直接当作 JSON。

```go
json.RawMessage(`hello`)     // 非法：字符串缺少双引号
json.RawMessage(`"hello"`) // 合法 JSON 字符串
json.RawMessage(`123`)       // 合法 JSON 数字
json.RawMessage(`true`)      // 合法 JSON 布尔值
json.RawMessage(`null`)      // 合法 JSON null
json.RawMessage(`[1,2,3]`)   // 合法 JSON 数组
json.RawMessage(`{"id":1}`) // 合法 JSON 对象
```

手动构造 `RawMessage` 时不会立即校验；在 `json.Marshal`、显式校验或后续解析时才会暴露非法 JSON。需要提前判断时可使用：

```go
if !json.Valid(raw) {
	return errors.New("invalid json payload")
}
```

### 仍然需要校验具体载荷

延迟解析不等于跳过验证。完成第二次 `Unmarshal` 后，仍需检查必填字段、取值范围、事件版本和业务约束。

### 适用边界

- JSON 结构固定且明确时，优先直接解析为具体结构体。
- 需要根据判别字段选择具体类型时，使用 `json.RawMessage`。
- 需要直接操作完全未知的 JSON 树时，可使用 `any`、`map[string]any` 或专门的动态 JSON 工具。
- 需要保存二进制数据时，使用 `[]byte`，由 `encoding/json` 按 Base64 字符串编码。

**结论：**`json.RawMessage` 的核心价值是保留某个 JSON 子树，在确定目标类型后再解析，或在不理解载荷的中间层中继续传递。


<br/>

***
<br/><br/><br/>
> <h1 id="终止型错误">终止型错误</h1>

核心判断函数：

```go
type hgTerminalError struct{ cause error }

func hgIsTerminalError(err error) bool {
	var terminal hgTerminalError
	return errors.As(err, &terminal)
}
```

`hgTerminalError` 用于标记**不应重试、应终止当前处理流程**的错误；`hgIsTerminalError` 检查错误链中是否存在该类型。

***
<br/><br/>
> <h2 id="errors.As的匹配条件"><code>errors.As</code> 的匹配条件</h2>

```go
var terminal hgTerminalError
return errors.As(err, &terminal)
```

`errors.As` 会沿错误链逐层检查，并尝试把匹配层赋值给目标变量。目标必须是非 `nil` 指针，且其指向的类型需要实现 `error`，或目标指向接口类型。

仅从当前片段看，`hgTerminalError` 没有展示 `Error() string` 方法。如果项目其他位置也没有为它实现 `error`，则它不能作为 `error` 返回；对非 `nil` 错误调用上述 `errors.As` 也会因目标类型不合法而 panic。完整实现可写为：

```go
// 必须实现 error
func (e hgTerminalError) Error() string {
    return e.cause.Error()
}
func (e hgTerminalError) Unwrap() error {
    return e.cause
}

// 包装错误向外返回
func wrapTerminal(err error) error {
    return hgTerminalError{cause: err}
}
```

- `Error()` 使 `hgTerminalError` 实现 `error`。
- `Unwrap()` 把 `cause` 接入错误链，便于继续使用 `errors.Is`、`errors.As` 检查根因。
- 使用值接收者时，错误链中的动态类型为 `hgTerminalError`，与 `var terminal hgTerminalError` 对应。

`errors.Is` 与 `errors.As` 的侧重点不同：

- `errors.Is(err, target)`：判断错误链是否匹配目标错误值或目标定义的 `Is` 规则，常用于哨兵错误。
- `errors.As(err, &target)`：提取错误链中可赋值给目标类型的错误。

***
<br/><br/>
> <h2 id="终止型错误的使用与风险">使用与风险</h2>

```go
err := doKafkaOp()
if hgIsTerminalError(err) {
    // 终止错误：不再重试，直接退出consumer/放弃当前任务
    return err
}
// 普通错误，可以sleep后重试
```

通常可按业务语义区分：

- **终止型错误**：当前处理路径无法通过重试恢复，例如确定的配置非法、认证失败或不支持的消息协议版本。
- **可重试错误**：可能随时间恢复，例如短暂网络抖动或 broker 暂时繁忙。

是否终止必须依据具体客户端和业务契约判断。例如 topic 不存在有时是永久配置错误，有时也可能由自动创建或稍后部署恢复，不宜只按错误名称固定分类。

**结论：**`hgTerminalError` 是不可重试标记，`hgIsTerminalError` 通过错误链类型识别该标记；前提是该类型已正确实现 `error`。
