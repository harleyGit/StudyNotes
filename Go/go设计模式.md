> <h1 id=""></h1>
- [**工厂模式**](#工厂模式)
- [依赖注入+组合接口模式](#依赖注入+组合接口模式)




<br/><br/><br/>

***
<br/>

> <h1 id="工厂模式">工厂模式</h1>

**说明：**

Golang中的结构体没有**构造函数**，通常可以使用工厂模式来解决问题。

```go
pacage model

type Student struct {
	Name string...
}
```

因为Student的S是大写可以暴露给其他包使用，但是若是`type student struct{}`,该如何做呢？这个时候可以使用**工厂模式了**。

**如下：**

**model文件下有：**

```go
pacage model

type student struct {
	Name string
	score float64
}


// 因为student结构体首字母是小写，因此是只能在model使用
// 可以通过工厂模式来解决

func NewStudent(n string, s float64) *student {
	return &student {
		Name: n,
		score: s,
	}
}

// 如果score也是小写，我们可以提供一个get方法
func (s *student) getScore() {
	return s.score //ok
}
```

<br/>

**main.go文件：**

```go
import "go_code/factory/model"

func main() {

	// 定义studnet结构体是小写字母，我们可以通过工厂模式来解决
	var stu  = model.NewStudent("tom~", 88.8)
	
	fmt.Println(*stu) //&{...}
	fmt.Println("name=", stu.Name, "score=", &stu.getScore() cxxdcxZ)
}
```

<br/><br/><br/>

***
<br/>

> <h1 id="依赖注入+组合接口模式">依赖注入+组合接口模式</h1>

这段 Go 代码：

```go
type Channel struct {
    backend BackendQueue // 这是一个接口
}
```

***
<br/>

- **1️⃣ `type Channel struct { ... }`**
	* 定义了一个 **结构体类型**，名字叫 `Channel`。
	* 结构体是 Go 中用来组合一组字段的类型。

<br/>

**2️⃣ `backend BackendQueue`**

* `backend` 是 `Channel` 结构体里的一个字段。
* 字段的类型是 `BackendQueue`，而 `BackendQueue` 是一个 **接口类型**（interface）。

接口在 Go 中是“一组方法签名”的集合，任何实现了接口中所有方法的类型都可以赋值给这个接口变量。

<br/>

**3️⃣ 作用**

把接口作为字段放进结构体的主要目的，是为了 **解耦** 和 **可扩展**：

* `Channel` 只依赖 `BackendQueue` 接口，而不依赖它的具体实现。
* 这样 `Channel` 可以在不同场景下组合不同的 `BackendQueue` 实现（比如：内存队列、Redis 队列、Kafka 队列……）。

<br/>

**4️⃣ 举例**

假设接口 `BackendQueue` 是这样定义的：

```go
type BackendQueue interface {
    Push(item any) error
    Pop() (any, error)
}
```

<br/>

然后可以写多个实现：

```go
type MemoryQueue struct {
    data []any
}

func (m *MemoryQueue) Push(item any) error {
    m.data = append(m.data, item)
    return nil
}
func (m *MemoryQueue) Pop() (any, error) {
    if len(m.data) == 0 {
        return nil, errors.New("empty")
    }
    item := m.data[0]
    m.data = m.data[1:]
    return item, nil
}
```

<br/>

再把它注入到 `Channel`：

```go
q := &MemoryQueue{}
ch := Channel{backend: q}

ch.backend.Push("hello")
val, _ := ch.backend.Pop()
fmt.Println(val) // 输出 hello
```

<br/>

**5️⃣ 总结**

> `Channel` 里的 `backend BackendQueue`
> 👉 是用来把一个“队列功能”的接口嵌进结构体中，使 `Channel` 能通过接口来访问不同的队列实现，而不用关心细节。

这就是 Go 常用的 **依赖注入 / 组合接口** 模式。












