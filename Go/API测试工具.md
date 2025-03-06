> <h4/>
- [Curl进行API测试](#Curl进行API测试) 
	- [发送GET请求](#发送GET请求) 
	- [发送POST请求](#发送POST请求) 
	- [添加请求头](#添加请求头) 
	- [发送带认证的请求](#发送带认证的请求) 
	- [下载文件](#下载文件) 
	- [跟随重定向](#跟随重定向) 
	- [显示响应头](#显示响应头) 
	- [保存响应到文件](#保存响应到文件) 
	- [显示请求&响应详情（调试API）](#显示请求&响应详情（调试API）)
	- [🌟`curl`高级用法](#🌟`curl`高级用法)
- [HTTPie进行API测试](#HTTPie进行API测试) 
- [Postman进行API测试](#Postman进行API测试) 
- [VSCode插件API测试](#VSCode插件API测试)


<br/><br/><br/>

***
<br/>

> <h1 id="Curl进行API测试">Curl进行API测试</h1>

**📌 `curl` 是什么？**
`curl`（Client URL）是一个**命令行工具**，用于发送 HTTP 请求、下载文件、调试 API 等。支持多种协议，如 HTTP、HTTPS、FTP、SFTP 等。

<br/><br/>
> <h3 id="发送GET请求">发送GET请求</h3>

```bash
curl http://example.com
```
🔹 **默认使用 GET 方法**，相当于在浏览器里访问 `http://example.com`。

<br/>

**发送带 Token 的请求**

```sh
curl -X GET http://localhost:8080/api/protected \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

<br/><br/>
> <h3 id="发送POST请求">发送POST请求</h3>

```bash
curl -X POST http://example.com -d "username=admin&password=123456"
```
🔹 `-X POST` 指定 `POST` 方法，`-d` 发送表单数据。

<br/>

**发送 POST 请求（JSON 数据）**

```sh
curl -X POST http://localhost:8080/api/users \
     -H "Content-Type: application/json" \
     -d '{"name": "Alice", "email": "alice@example.com"}'
```
🔹 `-H` 设置请求头，`-d` 发送 JSON 数据。

<br/><br/>
> <h3 id="添加请求头">添加请求头</h3>

```bash
curl -H "Authorization: Bearer TOKEN" \
     -H "Accept: application/json" \
     http://example.com/api
```
🔹 `-H` 设置多个请求头，适用于 API 调试。


<br/><br/>
> <h3 id="发送带认证的请求">发送带认证的请求</h3>

```bash
curl -u username:password http://example.com/protected
```
🔹 `-u` 发送 **Basic Auth** 认证（用户名/密码）。

<br/><br/>
> <h3 id="下载文件">下载文件</h3>

```bash
curl -O http://example.com/file.zip
```
🔹 `-O` **保存文件**，文件名与远程一致。

```bash
curl -o myfile.zip http://example.com/file.zip
```
🔹 `-o` **自定义文件名**。

<br/><br/>
> <h3 id="跟随重定向">跟随重定向</h3>

```bash
curl -L http://example.com
```
🔹 `-L` **自动跟随 301/302 重定向**。

<br/><br/>
> <h3 id="显示响应头">显示响应头</h3>

```bash
curl -I http://example.com
```
🔹 `-I` **只显示响应头**（HEAD 请求）。

<br/><br/>
> <h3 id="保存响应到文件">保存响应到文件</h3>

```bash
curl http://example.com -o output.html
```
🔹 `-o` **将响应内容保存为 `output.html`**。


<br/>

***
<br/>

> <h3 id="显示请求&响应详情（调试API）">显示请求 & 响应详情（调试 API）</h3>


```bash
curl -v http://example.com
```
🔹 `-v` **显示详细的请求/响应信息**，用于调试。

```bash
curl -vvv http://example.com
```
🔹 `-vvv` **更详细的调试信息**。

---
<br/><br/>
> <h3 id="🌟`curl`高级用法">🌟 `curl` 高级用法</h3>

| 功能 | 命令 |
|------|------|
| **GET 请求** | `curl http://example.com` |
| **POST 请求** | `curl -X POST -d "name=John" http://example.com` |
| **发送 JSON** | `curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' http://example.com` |
| **下载文件** | `curl -O http://example.com/file.zip` |
| **认证请求** | `curl -u user:pass http://example.com` |
| **设置请求头** | `curl -H "Authorization: Bearer TOKEN" http://example.com` |
| **自动重定向** | `curl -L http://example.com` |
| **查看响应头** | `curl -I http://example.com` |
| **调试模式** | `curl -v http://example.com` |



<br/><br/><br/>
> <h2 id="HTTPie进行API测试">HTTPie进行API测试</h2>

`HTTPie` 是 `Curl` 的替代品，语法更直观，适用于 API 调试。

- **安装 HTTPie**

```sh
pip install httpie  # 需要 Python
```

<br/>

**示例 1：发送 GET 请求**

```sh
http GET http://localhost:8080/api/users
```

<br/>

**示例 2：发送 POST 请求（JSON 数据）**

```sh
http POST http://localhost:8080/api/users name="Alice" email="alice@example.com"
```

<br/>

**示例 3：发送带 Token 的请求**

```sh
http GET http://localhost:8080/api/protected "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

<br/><br/><br/>
> <h2 id="Postman进行API测试">Postman进行API测试</h2>

Postman 是一个图形化 API 测试工具，适用于复杂的 API 交互、自动化测试、Mock API 等。

- **使用步骤**
	1. **安装 Postman**：从 [Postman 官网](https://www.postman.com/) 下载并安装。
	2. **创建请求**：
	   - 选择 **GET、POST、PUT、DELETE** 等方法。
	   - 填写 **URL**（如 `http://localhost:8080/api/users`）。
	   - 添加 **Headers**（如 `Content-Type: application/json`）。
	   - 若为 `POST` 或 `PUT`，在 **Body** 选项卡填入 JSON 数据。
	3. **点击“Send”** 发送请求并查看响应。

<br/><br/><br/>
> <h2 id="VSCode插件API测试">VSCode插件API测试</h2>

VSCode 提供多种 API 测试插件，常用的是 `REST Client` 插件。

- **安装 REST Client 插件**
	- 打开 VSCode，按 `Cmd + Shift + X`（Mac）或 `Ctrl + Shift + X`（Windows）。
	- 搜索 `REST Client` 并安装。

<br/>

- **创建 API 请求**

在 `.http` 或 `.rest` 文件中写入请求，例如：

```http
### 发送 GET 请求
GET http://localhost:8080/api/users

### 发送 POST 请求
POST http://localhost:8080/api/users
Content-Type: application/json

{
    "name": "Alice",
    "email": "alice@example.com"
}
```

<br/>

- **执行请求**
	1. 在 VSCode 打开 `.http` 文件。
	2. 在请求上方会出现 `Send Request` 按钮，点击即可运行并查看响应。

