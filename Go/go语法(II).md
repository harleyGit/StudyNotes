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
