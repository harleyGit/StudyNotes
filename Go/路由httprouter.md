- [**‌Router结构体**](#Router结构体)
- [‌高阶语法-装饰器](#‌高阶语法-装饰器)


<br/><br/><br/>

***
<br/>

> <h1 id="Router机构体">Router机构体</h1>

这个 `Router` 是 [**julienschmidt/httprouter**](https://github.com/julienschmidt/httprouter) 里的核心类型 —— `httprouter.Router`。


`httprouter` 是 Go 社区非常流行的高性能路由库，很多框架（包括 **NSQ 的 HTTP API**）都会用它来处理路由。

<br/>

源码里确实有类似定义（部分字段）：

```go
// httprouter/router.go
type Router struct {
    RedirectTrailingSlash  bool
    RedirectFixedPath      bool
    HandleMethodNotAllowed bool
    HandleOPTIONS          bool
    // ...
}
```

<br/>

**字段含义**

* **RedirectTrailingSlash**
  如果 URL 少了或多了 `/`，是否自动重定向。

	* true: `/foo/` → `/foo`
	* true: `/foo` → `/foo/`

* **RedirectFixedPath**
  是否自动修正 URL 大小写或多余斜杠。

	* 比如 `/Foo//bar` → `/foo/bar`

* **HandleMethodNotAllowed**
  如果路由存在但 HTTP 方法不匹配，是否返回 `405 Method Not Allowed`。

* **HandleOPTIONS**
  是否自动处理 `OPTIONS` 请求（主要用于 CORS）。

<br/>


**一个最小可运行的例子：**

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/julienschmidt/httprouter"
)

func main() {
	// 创建 router（你贴的 New() 就是类似的工厂函数）
	router := &httprouter.Router{
		RedirectTrailingSlash:  true,
		RedirectFixedPath:      true,
		HandleMethodNotAllowed: true,
		HandleOPTIONS:          true,
	}

	// 注册一个路由
	router.GET("/hello/:name", func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
		fmt.Fprintf(w, "Hello, %s!\n", ps.ByName("name"))
	})

	// 启动 HTTP 服务
	http.ListenAndServe(":8080", router)
}
```

运行后访问：

* `http://localhost:8080/hello/Harley`
  → 输出：`Hello, Harley!`

* `http://localhost:8080/hello/Harley/`
  因为 `RedirectTrailingSlash: true`，会自动跳转到 `/hello/Harley`。



<br/><br/><br/>

***
<br/>

> <h1 id="‌高阶语法-装饰器">‌高阶语法-装饰器</h1>

**1.先看类型定义**

```go
type APIHandler func(http.ResponseWriter, *http.Request, httprouter.Params) (interface{}, error)
```

也就是说，一个 **业务处理函数** 长这样：

* 参数：
	
	* `w http.ResponseWriter`（HTTP 响应对象）
	* `req *http.Request`（请求对象）
	* `ps httprouter.Params`（路由参数）
* 返回值：

	* `interface{}`（任意数据，要写回客户端的内容）
	* `error`（错误，通常是 `Err{Code, Message}`）

<br/>

**2.PlainText 的签名**

```go
func PlainText(f APIHandler) APIHandler
```

* 输入：一个 `APIHandler`（业务函数）
* 输出：一个新的 `APIHandler`（被“包装”的业务函数）

所以 PlainText 是一个 **装饰器**。
它不会直接被 `httprouter` 调用，而是作为参数传入 `Decorate`，和业务函数拼接在一起。

<br/>

**3.PlainText 内部逻辑**

```go
return func(w http.ResponseWriter, req *http.Request, ps httprouter.Params) (interface{}, error) {
    code := 200
    data, err := f(w, req, ps)  // 调用传入的 f（业务函数）
    if err != nil {
        code = err.(Err).Code
        data = err.Error()
    }

    // 根据 data 的类型写回响应
    switch d := data.(type) {
    case string:
        w.WriteHeader(code)
        io.WriteString(w, d)
    case []byte:
        w.WriteHeader(code)
        w.Write(d)
    default:
        panic(fmt.Sprintf("unknown response type %T", data))
    }
    return nil, nil
}
```

**关键点：**

1. **调用 f** → `f(w, req, ps)`，得到原始返回值 `data, err`
2. **处理错误** → 如果有错误，把 `error` 转成字符串作为响应内容
3. **根据类型写回响应** → 如果是 `string` / `[]byte`，直接写到 `http.ResponseWriter`
4. **返回值始终是 `nil, nil`**，因为数据已经写入 `w`，不需要再向后传递

<br/>

**4.结合调用例子**

```go
router := httprouter.New()
router.Handle("GET", "/ping", http_api.Decorate(s.pingHandler, log, http_api.PlainText))
```

* `s.pingHandler` 是你的业务逻辑函数（返回数据）。
* `http_api.PlainText` 是装饰器，把返回的数据写成 `text/plain`。
* `Decorate` 的作用就是：把一组 **装饰器（PlainText、Log...）** 按顺序作用到业务函数上，返回一个最终的 `APIHandler`。
* 最终这个 `APIHandler` 被适配成 `httprouter.Handle` 兼容的形式，供路由调用。

<br/>

**5.举个完整例子**

比如定义一个业务函数：

```go
func pingHandler(w http.ResponseWriter, r *http.Request, ps httprouter.Params) (interface{}, error) {
	return "pong", nil
}
```

装饰后：

```go
router.Handle("GET", "/ping", http_api.Decorate(pingHandler, http_api.PlainText))
```

访问 `/ping` 时：

1. `pingHandler` 返回 `"pong"`
2. `PlainText` 装饰器检测到返回的是 `string`
3. 写回响应：

   ```
   HTTP/1.1 200 OK
   Content-Type: text/plain
   pong
   ```
4. `PlainText` 返回 `(nil, nil)`，因为数据已经写到响应里了。

<br/>

✅ 总结：

* **参数的传递**：`PlainText` 接收一个 `APIHandler`，返回一个新的 `APIHandler`。调用的时候，它会先执行原始 `f`，拿到结果，再写到响应。
* **返回数据**：并不是通过 `return` 返回的，而是直接写入 `http.ResponseWriter`。`return nil, nil` 只是为了满足函数签名。

***
<br/>

 **调用顺序图**，展示 `httprouter → Decorate → PlainText → pingHandler` :
 
 

```text
客户端请求: GET /ping
        │
        ▼
 ┌─────────────────────────────┐
 │ httprouter.Router.Handle    │
 │  匹配路由 "/ping"           │
 └─────────────────────────────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ http_api.Decorate(...)      │
 │  组合:                      │
 │   - s.pingHandler (业务逻辑) │
 │   - PlainText (装饰器)       │
 │   - Log(...) (装饰器，可选) │
 └─────────────────────────────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ PlainText 装饰器 (APIHandler)│
 │ 调用: f(w, req, ps)         │
 └─────────────────────────────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ s.pingHandler                │
 │ return "pong", nil           │
 └─────────────────────────────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ PlainText 收到结果           │
 │ data = "pong"                │
 │ err = nil                    │
 │ → 类型是 string               │
 │ → w.WriteHeader(200)         │
 │ → io.WriteString(w, "pong")  │
 └─────────────────────────────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ HTTP Response                │
 │ 200 OK                       │
 │ pong                         │
 └─────────────────────────────┘
```

<br/>

**总结**

* **httprouter**：只负责路由匹配，把请求交给最终的 `APIHandler`。
* **Decorate**：把多个装饰器（`PlainText`, `Log`, `V1`...）组合起来形成最终的处理函数。
* **PlainText**：拦截 `APIHandler` 的返回值，把它写到 `http.ResponseWriter`。
* **业务函数**：只需返回 `string` / `[]byte` / 错误，不用关心怎么写 HTTP 响应。


具体代码看**`工程中的 nsq_decorate_pt.go`**，看流程需要在浏览器中执行：

- **览器访问：**

	- `http://localhost:8080/ping`
 → 返回 pong

	- `http://localhost:8080/hello/Alice`
 → 返回 Hello, Alice!
 
 
 ***
 <br/><br/>
 
 **在 工程中的 `nsq_decorate_pt.go`**有不少代码看不太懂，比如：

 <br/>
  
 **1.装饰器Decorate点参数**

```go
// Decorate 把多个装饰器包裹起来
func decorate(f APIHandler, decorators ...func(APIHandler) APIHandler) APIHandler {
	// 反向应用：最后一个 decorator 最先执行
	for i := len(decorators) - 1; i >= 0; i-- {
		f = decorators[i](f)
	}
	return f
}
```
中的 `decorators ...func(APIHandler) APIHandler` 啥意思，特别是`...func(APIHandler)` **？？**

这是 Go 里一个**变长参数**（variadic parameter）+ 函数类型的组合，看起来有点陌生，我们对其进行分解看看：

 <br/> 
 
 **1️⃣ 先看 `func(APIHandler) APIHandler`**

* 这表示一种**函数类型**：

  > “接受一个 `APIHandler`，返回一个 `APIHandler`”

换句话说，它是「装饰器」函数的类型签名：输入一个 handler，输出一个“包了一层”的 handler。

例如：

```go
func PlainText(next APIHandler) APIHandler {
    return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) (interface{}, error) {
        // ...处理逻辑...
        return next(w, r, ps)
    }
}
```

这里 `PlainText` 的类型就是 `func(APIHandler) APIHandler`。

 <br/>

 **2️⃣ 再看 `decorators ...func(APIHandler) APIHandler`**

* 这是一个 **变长参数**（`...`）。
* 它的意思是：`Decorate` 函数可以接收任意多个「函数装饰器」。

等价于：

```go
decorators []func(APIHandler) APIHandler
```

只是写成 `...` 后，调用时可以直接写多个参数，而不用手动创建切片。

 <br/>

**3️⃣ `Decorate` 函数整体**

把它组合在一起：

```go
func Decorate(
    f APIHandler,                               // 原始 handler
    decorators ...func(APIHandler) APIHandler, // 任意数量的装饰器
) APIHandler
```

* `f` 是你要被装饰的核心 handler。
* `decorators` 是一串函数，每个都能接收 `APIHandler` 并返回一个“包裹后”的 `APIHandler`。

在 `Decorate` 里，一般会把这些装饰器按顺序套在 `f` 外面：

```go
func Decorate(f APIHandler, decorators ...func(APIHandler) APIHandler) APIHandler {
    for i := len(decorators) - 1; i >= 0; i-- {
        f = decorators[i](f)
    }
    return f
}
```

> 从后往前套，保证调用顺序和书写顺序一致。

 <br/> 
 
 **4️⃣ 调用示例**

```go
handler := Decorate(pingHandler, PlainText, Log, Auth)
```

* `pingHandler` 是最原始的处理函数。
* `PlainText`、`Log`、`Auth` 都是 `func(APIHandler) APIHandler`。

等价于（展开后）：

```go
handler := Auth(Log(PlainText(pingHandler)))
```

 <br/>

✅ **总结**

* `func(APIHandler) APIHandler` → “接受一个 `APIHandler`，返回一个新的 `APIHandler`”。
* `decorators ...func(APIHandler) APIHandler` → “任意多个这种装饰器函数”。
* `Decorate` 用来把 `f` 依次传入每个装饰器，生成最终 handler。

<br/><br/>

**2.有点看象回调函数，不知道参数如何传递**
  
```go
router := httprouter.New()

// 使用 PlainText 装饰器
router.Handle("GET", "/ping", func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	decorate(pingHandler, plainText)(w, r, ps)
})
```

有点不明白没有看到其他函数传值给`func(w http.ResponseWriter, r *http.Request, ps httprouter.Params)`啊？

还有` (w, r, ps)` 来自于 `func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) `吗？

<br/>

我们把这个「调用链」一步步拆开，就能看到 `(w, r, ps)` 到底是怎么“穿”进那堆函数里的了👇

<br/>

**1️⃣ 入口：`router.Handle`**

```go
router.Handle("GET", "/ping",
    func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
        Decorate(pingHandler, PlainText)(w, r, ps)
    })
```

<br/>

* `httprouter` 规定：所有注册到 `Handle` 的 handler，必须长这样：

  ```go
  func(http.ResponseWriter, *http.Request, httprouter.Params)
  ```

  所以 `httprouter` 在收到请求时，会自动把 `w`（响应写入器）、`r`（请求对象）、`ps`（路由参数）传给你注册的这个匿名函数。

<br/>

**2️⃣`Decorate(...)` 的返回值**

```go
Decorate(pingHandler, PlainText)
```

<br/>

* `Decorate` 返回的是一个 **新的函数**，它的签名也是：

  ```go
  func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) (interface{}, error)
  ```

  也就是 `APIHandler`。

* 所以写成 `Decorate(...)(w, r, ps)`，意思是：

  > “把 `(w, r, ps)` 这三个参数传进去，执行那个被装饰后的 handler”。

<br/>

**3️⃣ 装饰器内部的调用**

`PlainText` 的定义：

```go
func PlainText(f APIHandler) APIHandler {
    return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) (interface{}, error) {
        data, err := f(w, r, ps) // 👈 这里又把 w/r/ps 传给下一层
        ...
    }
}
```

* 你可以看到，`PlainText` 返回的匿名函数也接受 `(w, r, ps)`，然后把它们继续传给 `f`（也就是 `pingHandler`）。

`pingHandler` 自身签名也是 `func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) (interface{}, error)`，所以参数完美对上。

<br/>

**4️⃣ 整个调用栈**

收到一个 `GET /ping` 请求时，执行顺序是：

```
httprouter → 调用匿名函数(w,r,ps)
    ↓
Decorate(pingHandler, PlainText) 返回的 handler
    ↓
PlainText(...) 返回的函数(w,r,ps)
    ↓
pingHandler(w,r,ps)
```

所有的 `w / r / ps` 都是从最外层一路“传递”进去的，没有任何魔法，Go 只是把这些函数值嵌套了起来。

<br/>

✅ **关键点**

* `httprouter` 负责把 `w r ps` 提供给你注册的最外层函数。
* `Decorate` 只是返回一个“包好”的 `APIHandler`，仍然要你显式调用 `(w, r, ps)` 才会执行。
* 每个装饰器都接收 `(w,r,ps)`，并把它传下去，所以参数能一直传到最里面的业务 handler。


***
<br/><br/>

下面是一个 **ASCII 调用流程图**


**请求进入**

```
浏览器/客户端
      │
      ▼
HTTP Server (ListenAndServe)
      │
      ▼
httprouter → 根据 URL 匹配到 "/ping"
      │
      ▼
你注册的匿名函数:
    func(w, r, ps) {
        Decorate(pingHandler, PlainText)(w, r, ps)
    }
```

<br/> 

**Decorate 里的逻辑**

```
Decorate(pingHandler, PlainText)
    │
    ▼
调用顺序反转 → 得到:
    PlainText(pingHandler)
      (返回一个新的函数: func(w,r,ps) {...})
```

<br/>

**处理请求时的调用链**

```
httprouter 把 (w,r,ps) 传进匿名函数
        │
        ▼
匿名函数调用 Decorate(...) 返回的 handler:
        handler(w, r, ps)
        │
        ▼
PlainText(...) 返回的函数被执行:
        ┌─────────────────────────────────┐
        │ data, err := f(w, r, ps)       │ ← 这里的 f = pingHandler
        └─────────────────────────────────┘
        │
        ▼
pingHandler(w, r, ps)  ← 真正的业务逻辑
        │
        ▼
返回 "pong"
        │
        ▼
PlainText 负责把结果写入 w (ResponseWriter)
```

<br/>

**总结**

* `w`, `r`, `ps` 是 **httprouter 提供** 的，请求到来时它会传入你注册的 handler。
* `Decorate`/`PlainText` 只是生成了一个“函数工厂”，返回值本质还是 `func(w,r,ps)`。
* 当你写 `Decorate(...)(w,r,ps)`，就是在调用那个返回的函数，所以参数顺理成章地一路传到底。

<br/>

这样你就可以把每层理解为：

```
[httprouter Handler] -> [Decorate生成的新Handler] -> [PlainText包装] -> [业务Handler]
```

每个环节都显式接收 `w / r / ps`，再传给下一层，没有隐藏传参的魔法。




