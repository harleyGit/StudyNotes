- [**‌Router结构体**](#Router结构体)


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




