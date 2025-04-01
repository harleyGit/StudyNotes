
> <h1></h1>
> [gin的结构化标签](#gin的结构化标签)
- [Gin构建 `RESTful API`](#Gin构建RESTfulAPI)
- [中间件解决跨域](#中间件解决跨域)
- [把爬虫程序设置成Web服务](#把爬虫程序设置成Web服务)
- **资料**
	- [手把手，带你从零封装Gin框架（一）：开篇 & 项目初始化](https://juejin.cn/post/7016742808560074783)


<br/><br/><br/>
> <h2 id="gin的结构化标签">gin的结构化标签</h2>

```go
type User struct {
	Name  string `json:"name" binding:"required"`  // 必填
	Email string `json:"email" binding:"required,email"` // 必填，并且必须是有效的 email 格式
	Age   int    `json:"age" binding:"required"`  // 必填
}
```

`binding:"required"` 是 Gin 框架中用于 **数据验证（Validation）** 的标签（tag），它属于 `github.com/go-playground/validator/v10` 库，Gin 内部集成了这个库来自动验证请求参数。

---

### **作用**
当 `binding:"required"` 作用于结构体字段时：
- **如果字段值为空（未提供），Gin 会自动返回 400 状态码，并提示该字段是必填项**。

---

```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "age": 25
}
```
那么 `c.ShouldBindJSON(&user)` 会成功解析 JSON，并且不会报错。

---

#### **2. 缺少必填字段**
如果请求数据如下（缺少 `email` 字段）：

```json
{
  "name": "Alice",
  "age": 25
}
```
Gin 会自动返回：

```json
{
  "error": "Key: 'User.Email' Error:Field validation for 'Email' failed on the 'required' tag"
}
```
并且 HTTP 状态码是 `400 Bad Request`。

---

### **其他常见的验证标签**
| 标签 | 作用 |
|------|------|
| `binding:"required"` | 字段必须存在，且不能为空 |
| `binding:"email"` | 该字段必须是一个有效的 email 格式 |
| `binding:"min=6"` | 该字段最小长度为 6 |
| `binding:"max=20"` | 该字段最大长度为 20 |
| `binding:"gte=18"` | 该字段值必须 `>=18`（常用于年龄限制） |
| `binding:"lte=100"` | 该字段值必须 `<=100` |
| `binding:"omitempty"` | 该字段可选，但如果提供了值，则必须满足其他条件 |
| ` binding:" alphanumunicode "` | 允许 Unicode 字母（包括中文、英文、韩文等）和数字，但 不能包含特殊字符（如 @!#） |
| ` binding:" exists "` | 必须包含 指明 字段，但可为空 |
| ` binding:"regex=^[\p{L}\d_]+$"` | 允许 Unicode 语言的字母（中文）、数字、下划线，但禁止特殊字符。 |
|  |  |
|  |  |

---

### **总结**
- `binding:"required"` **确保字段不能为空**，否则返回 400 错误。
- **可以结合多个验证规则**（如 `email`, `min`, `max`）提高数据可靠性。
- **Gin 自动处理验证**，无须手写 `if 判断` 逐一检查字段，简化代码。


<br/><br/><br/>
> <h2 id="Gin构建 RESTfulAPI">Gin构建 RESTful API</h2>

- **使用 `Gin` 构建 RESTful API**
相比 `net/http`，`Gin` 更简洁高效：

```go
package main

import (
	"net/http"
	"strconv"

	"github.com/gin-gonic/gin"
)

type User struct {
	ID    int    `json:"id"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

var users = []User{
	{ID: 1, Name: "Alice", Email: "alice@example.com"},
	{ID: 2, Name: "Bob", Email: "bob@example.com"},
}

func main() {
	r := gin.Default()

	// 获取所有用户
	r.GET("/users", func(c *gin.Context) {
		c.JSON(http.StatusOK, users)
	})

	// 获取单个用户
	r.GET("/users/:id", func(c *gin.Context) {
		id, err := strconv.Atoi(c.Param("id"))
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid user ID"})
			return
		}

		for _, user := range users {
			if user.ID == id {
				c.JSON(http.StatusOK, user)
				return
			}
		}
		c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
	})

	r.Run(":8080")
}
```
- 运行后，访问：
  - `GET http://localhost:8080/users`
  - `GET http://localhost:8080/users/1`


<br/><br/><br/>
> <h2 id="中间件解决跨域">中间件解决跨域</h2>

**范例代码:**

```go
// 中间件解决跨域
func (ginPracticeV1 *GinPracticeV1) GormPracticeV1_v9() {
	r := gin.Default()

	// 使用全局中间件处理跨域问题(局部中间件是加在路由组里面)
	r.Use(corsMiddleware())

	// 其他路由注册
	r.GET("/hello", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Hello, CORS is enabled!"})
	})

	// 启动 Gin 服务器
	r.Run(":8080")
}
// corsMiddleware 中间件处理跨域问题
func corsMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
		c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
		c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE")
		c.Writer.Header().Set("Access-Control-Allow-Headers", "Origin, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization")

		if c.Request.Method == "OPTIONS" {
			c.AbortWithStatus(204)
			return
		}

		c.Next()
	}
}
```

**如下测试:**

- **1.测试 GET 请求**

```sh
curl -X GET http://localhost:8080/hello -i
```
**预期输出（响应头中应包含 CORS 相关信息）：**

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
...
{"message":"Hello, CORS is enabled!"}
```

---

- **2.测试 OPTIONS 预检请求**

```sh
curl -X OPTIONS http://localhost:8080/hello -i \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: Content-Type"
```
**预期输出（状态码 204，且包含 CORS 头）：**

```sh
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE
Access-Control-Allow-Headers: Origin, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization
```

---

- **3.测试携带自定义 Header 的请求**

```sh
curl -X GET http://localhost:8080/hello -i \
  -H "Origin: http://example.com" \
  -H "X-CSRF-Token: testtoken"
```
**预期输出（服务器应返回 `Access-Control-Allow-Origin: *`）：**

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
...
{"message":"Hello, CORS is enabled!"}
```

---

- **4. 测试携带认证信息的请求**

```sh
curl -X GET http://localhost:8080/hello -i \
  -H "Origin: http://example.com" \
  -H "Authorization: Bearer testtoken"
```
**预期输出（服务器应返回 `Access-Control-Allow-Origin: *`，并包含 `Authorization` 头）：**

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
...
{"message":"Hello, CORS is enabled!"}
```

<br/><br/>

**提问:** 对于跨域中的`‌https://www.youdao.com`携带这个有啥用?跨域这个指的是请求别的域名数据吗

是的，`Origin: http://example.com` 在 `curl` 请求中代表请求的来源域（即浏览器中访问的站点）。  

- **CORS（跨域资源共享）的作用**
跨域的意思是 **在一个域名下运行的网页，尝试请求另一个域名的数据**。  
例如：
	- 你的前端网页运行在 `https://www.youdao.com`
	- 你的后端 API 运行在 `http://api.example.com`

由于 **浏览器的同源策略**，默认情况下，`https://www.youdao.com` 不能直接请求 `http://api.example.com` 的数据。  
CORS 机制允许服务器 **显式地告诉浏览器**：哪些来源的请求是允许的，哪些是不允许的。

---

- **你 `curl` 里 `-H "Origin: https://www.youdao.com"` 的作用**
当你加上 `Origin: https://www.youdao.com`，是在模拟从 `https://www.youdao.com` 这个域发起的请求。  
服务器会根据 `Access-Control-Allow-Origin` 头来决定是否允许这个请求：

- **情况 1：你的代码 `Access-Control-Allow-Origin: *`**
	- `*` 代表允许任何来源（所有网站都可以请求这个 API）
	- 所以 `curl -H "Origin: https://www.youdao.com"` 依然能成功返回数据

- **情况 2：如果你改成 `Access-Control-Allow-Origin: https://www.youdao.com`**
	- 服务器只允许 `https://www.youdao.com` 访问
	- 其他来源（比如 `http://notallowed.com`）的请求会被 **浏览器拦截**（但 `curl` 仍然可以直接请求）

---

- **浏览器行为 vs. `curl`**
	- **浏览器会检查 CORS 响应头**，如果不允许跨域，它会拦截请求，前端 JavaScript 无法获取数据。
	- **`curl` 不受浏览器同源策略限制**，它直接发送请求并获取响应，所以你可以用 `curl` 看到完整返回数据。

---

- **总结**
	- **CORS 主要是浏览器的安全机制**，防止网页随意访问别的域名的数据。
	- 你用 `curl -H "Origin: https://www.youdao.com"` 是在模拟跨域请求，查看服务器是否正确返回 CORS 头。
	- 但如果你的前端网页访问 **没有 CORS 允许的 API**，就会被浏览器拦截。


<br/><br/><br/>
> <h2 id="把爬虫程序设置成Web服务">把爬虫程序设置成Web服务</h2>

把爬虫程序设置成Web服务。具体修改如下：运行修改后的爬虫程序，打开浏览器，访问“本机IP：端口”​；当在网页上看到提示信息（​“已经把http://news.baidu.com的网页内容全部抓取下来，请查看当前项目目录下的news.txt文件！”​）时，说明已经把http://news.baidu.com的网页内容全部抓取下来，并存储在当前项目目录下的news.txt文件中。

为了满足上述的修改要求，需要下载第三方包gin。如图19.8所示，下载gin包的步骤如下。

```go
// 把爬虫程序设置成Web服务
func testCrawlerAppointWebpageAndSaveTxtV2() {
	// 初始化引擎
	engine := gin.Default()
	// 注册一个路由和处理函数
	engine.Any("/", WebRoot)
	// 绑定端口
	engine.Run(":9200")
}
// 处理路由函数
func WebRoot(context *gin.Context){
	// 调用 testCrawlerAppointWebpageAndSaveTxt()函数
	testCrawlerAppointWebpageAndSaveTxt("https://news.baidu.com")
	// 设置网页上的文本内容
	context.String(http.StatusOK, "已经把http://news.baidu.com 的网页内容全部抓取下来\n 请查看当前项目目录下的news.txt文本文件！")
}

// 存储爬虫文件
func testCrawlerAppointWebpageAndSaveTxt(urls string) {
	// 创建 Collector 对象
	c := colly.NewCollector()
	/* 
	* 是否抓取指定链接的网页内容
	* 初始设置为不抓取指定链接的网页内容
	 */
	visited := false
	// 使用 Collector 对象抓取 URL
	c.OnResponse(func (r *colly.Response)  {
		if !visited {
			visited = true
			r.Request.Visit("/get?q=2")
		}
	})

	// 文件的创建和持续写入
	filename := "news.txt"	// 文件名
	var f *os.File
	var fileErr error 
	if testCheckFileExist(filename) {	// 如果文件存在
		f, fileErr = os.OpenFile(filename, os.O_APPEND | os.O_WRONLY, 0666)	// 打开文件
		if fileErr != nil {
			fmt.Printf("❌ 打开文件 news.txt 失败：%v\n", fileErr)
		}
	} else {
		f,_ = os.Create(filename)	// 创建文件
		fmt.Println("🍎 新建文件 news.txt")

	}
	defer f.Close()	// 关闭文件
	write := bufio.NewWriter(f)
	defer write.Flush()

	// 对指定链接网页内容进行处理
	c.OnHTML("a[href]", func (e *colly.HTMLElement)  {
		href := e.Text	// 获取指定链接的网页内容
		write.WriteString("🍒 "+href+"\n")
		fmt.Println("🍎 网页内容：",href)	// 打印指定链接的网页内容
	})


	c.Visit(urls)	//访问指定链接
}
/* 检测文件是否存在
* fileName 文件名
 */
func testCheckFileExist(filename string) bool {
	var exist = true
	if _, err := os.Stat(filename); os.IsNotExist(err) {
		exist = false
	}
	return exist
}
```

**Log:**

```
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.WebRoot (3 handlers)
[GIN-debug] POST   /                         --> main.WebRoot (3 handlers)
[GIN-debug] PUT    /                         --> main.WebRoot (3 handlers)
[GIN-debug] PATCH  /                         --> main.WebRoot (3 handlers)
[GIN-debug] HEAD   /                         --> main.WebRoot (3 handlers)
[GIN-debug] OPTIONS /                         --> main.WebRoot (3 handlers)
[GIN-debug] DELETE /                         --> main.WebRoot (3 handlers)
[GIN-debug] CONNECT /                         --> main.WebRoot (3 handlers)
[GIN-debug] TRACE  /                         --> main.WebRoot (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on :9200
```

<br/>

此时，Mac系统弹出如图19.9所示的Mac安全警报。选中“公用网络”复选框，单击“允许访问”按钮。

打开默认浏览器（比如：Safari浏览器），访问“本机IP：端口”​（即127.0.0.1:9200）​，即可看到如图19.10所示的提示信息。这段程序在当前项目目录下生成news.txt文件。单击news.txt，即可看到抓取的网页内容，如图所示。

![go.0.0.57.png](./../Pictures/go.0.0.57.png)

