- [Go 知识点](#Go知识点)
	- [中间件链式调用](#中间件链式调用)
	- [完整请求流程](#完整请求流程)
	- [ChainInterceptors 实现原理](#ChainInterceptors实现原理)
	- [洋葱模型图解](#洋葱模型图解)
	- [运行示例](#运行示例)
	- [HTTP Query 参数解析](#HTTPQuery参数解析)
	- [strconv.ParseInt 字符串转 int64](#strconvParseInt字符串转int64)
	- [database/sql Rows 与 Cursor](#databaseSQLRows与Cursor)
	- [scanAdminUserRow 行扫描](#scanAdminUserRow行扫描)
	- [seen 去重与 map 预分配](#seen去重与map预分配)
- [工程表](#工程表)
- [后台管理接口设计](#后台管理接口设计)
- [分布式限流-Lua脚本](#分布式限流-Lua脚本)


***
<br/><br/><br/>
> <h2 id="Go知识点">Go 知识点</h2>

本文整理 Go HTTP 中间件链式调用 `ChainInterceptors`：**启动阶段从后往前包装 handler，请求阶段从外到内进入，响应阶段从内到外退出**。


***
<br/><br/><br/>
> <h3 id="中间件链式调用">中间件链式调用</h3>

## 核心概念：洋葱模型

```text
ChainInterceptors(guarded, A, B, C)

请求流向（洋葱模型）：
┌─────────────────────────────────────────┐
│  C 进入 → B 进入 → A 进入 → guarded     │
│                 ↓ 业务处理              │
│  C 退出 ← B 退出 ← A 退出 ← guarded     │
└─────────────────────────────────────────┘
```

`ChainInterceptors(guarded, A, B, C)` 最终表现为：**C 最先进入，A 最靠近业务，响应返回时反向退出**。

<br/>

## 4 层中间件洋葱模型

```text
请求 ──→
        ┌────────────────────────────────────────┐
        │ 【1】JSONHeaderInterceptor 进入         │
        │   ┌──────────────────────────────────┐ │
        │   │ 【2】RecoverInterceptor 进入      │ │
        │   │   ┌────────────────────────────┐ │ │
        │   │   │ 【3】AccessLogInterceptor  │ │ │
        │   │   │ 进入                       │ │ │
        │   │   │   ┌──────────────────────┐ │ │ │
        │   │   │   │ 【4】TIDInterceptor  │ │ │ │
        │   │   │   │ 进入                  │ │ │ │
        │   │   │   │   ┌────────────────┐ │ │ │ │
        │   │   │   │   │   业务处理      │ │ │ │ │
        │   │   │   │   └────────────────┘ │ │ │ │
        │   │   │   │ 退出                  │ │ │ │
        │   │   │   └──────────────────────┘ │ │ │
        │   │   │ 退出                       │ │ │
        │   │   └────────────────────────────┘ │ │
        │   │ 退出                             │ │
        │   └──────────────────────────────────┘ │
        │ 退出                                   │
        └────────────────────────────────────────┘
                                     ←── 响应
```

**执行顺序：**

| 阶段 | 执行顺序 | 说明 |
|------|---------|------|
| 请求进入 | 1 → 2 → 3 → 4 → 业务 | 从外层到内层，层层深入 |
| 响应返回 | 业务 → 4 → 3 → 2 → 1 | 从内层到外层，层层退出 |

**每一层职责：**

| 层级 | 中间件 | 职责 |
|------|--------|------|
| 1 | `JSONHeaderInterceptor` | 设置响应头 `Content-Type: application/json` |
| 2 | `RecoverInterceptor` | 捕获 `panic`，防止服务崩溃 |
| 3 | `AccessLogInterceptor` | 记录请求方法、路径、耗时 |
| 4 | `RequestTIDInterceptor` | 生成追踪 ID，便于日志追踪 |

<br/>

## 简化代码示例

```go
// 假设 3 个简单拦截器
func A(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("A: 进入")
        next.ServeHTTP(w, r) // 调用下一层
        fmt.Println("A: 退出")
    })
}

func B(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("B: 进入")
        next.ServeHTTP(w, r)
        fmt.Println("B: 退出")
    })
}

func C(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("C: 进入")
        next.ServeHTTP(w, r) // 调用下一层
        fmt.Println("C: 退出")
    })
}

// 调用 ChainInterceptors
handler := ChainInterceptors(
    guarded, // 业务处理
    A, B, C, // 中间件
)
```

请求到达时输出：

```text
C: 进入
B: 进入
A: 进入
[guarded 业务处理]
A: 退出
B: 退出
C: 退出
```


***
<br/><br/><br/>
> <h3 id="完整请求流程">完整请求流程</h3>

假设用户调用登录接口：

```text
POST /api/v1/auth/login
Body: {"username": "test", "password": "123456"}
```

代码结构：

```go
return ChainInterceptors(
    guarded,                       // 内层：实际业务处理
    RequestTIDInterceptor,         // 4
    AccessLogInterceptor,          // 3
    RecoverInterceptor,            // 2
    JSONHeaderInterceptor,         // 1
)
```

请求完整旅程：

```text
┌─────────────────────────────────────────────────────────────────────┐
│  客户端发起请求                                                       │
│  POST /api/v1/auth/login                                            │
│  Body: {"username": "test", "password": "123456"}                    │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 1 层：JSONHeaderInterceptor                                       │
│   · 设置响应头：Content-Type: application/json                       │
│   · 调用 next.ServeHTTP()，进入下一层                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 2 层：RecoverInterceptor                                          │
│   · defer recover() 捕获可能的 panic                                 │
│   · 调用 next.ServeHTTP()，进入下一层                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 3 层：AccessLogInterceptor                                        │
│   · 记录请求开始时间                                                  │
│   · 调用 next.ServeHTTP()，进入下一层                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 4 层：RequestTIDInterceptor                                       │
│   · 生成/提取追踪ID (例如: TID-20260430-abc123)                       │
│   · 设置到 context 中                                                │
│   · 调用 next.ServeHTTP()，进入下一层                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 5 层：guarded (业务路由)                                          │
│   · APIGuardInterceptor 鉴权检查（公开接口，直接通过）                 │
│   · 路由匹配：找到 /api/v1/auth/login 对应的 Login 处理函数           │
│   · 执行 userHandler.Login(w, r)                                     │
│   · 业务逻辑：                                                        │
│     - 解析请求体                                                     │
│     - 验证用户名密码                                                  │
│     - 生成 Token                                                     │
│     - 返回 JSON 响应                                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
                        【响应原路返回】
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 4 层：RequestTIDInterceptor 返回                                  │
│   · 无额外处理，返回                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 3 层：AccessLogInterceptor 返回                                   │
│   · 计算耗时：200ms                                                   │
│   · 记录日志：[TID-20260430-abc123] POST /login 200 200ms            │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 2 层：RecoverInterceptor 返回                                     │
│   · 无 panic，直接返回                                               │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│ 第 1 层：JSONHeaderInterceptor 返回                                  │
│   · 响应头已设置，直接返回                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│  客户端收到响应                                                       │
│  HTTP/1.1 200 OK                                                     │
│  Content-Type: application/json                                      │
│  Body: {"code":0,"data":{"token":"xxx"},"message":"登录成功"}        │
└─────────────────────────────────────────────────────────────────────┘
```

<br/>

## 快递包裹类比

```text
你寄快递（发送请求）：

1. 📦 打包盒           → JSONHeaderInterceptor（标记内容类型）
2. 📋 贴保价单         → RecoverInterceptor（如果出问题有保障）
3. 🏷️ 贴运单号         → AccessLogInterceptor（记录追踪信息）
4. 🚪 放入快递柜       → RequestTIDInterceptor（生成取件码）
5. 🏠 快递员取走配送   → guarded（实际送达目的地）

签收后，快递员返回确认，一层层上报回来...
```

<br/>

## 如果中间发生 panic

```text
假设 Login 内部 panic("数据库连接失败")

第 5 层：guarded → panic
         ↓
第 4 层：RequestTIDInterceptor → 正常返回
         ↓
第 3 层：AccessLogInterceptor → 正常返回
         ↓
第 2 层：RecoverInterceptor
         · 捕获到 panic
         · 记录错误日志
         · 返回 500 错误响应
         · 阻止程序崩溃
         ↓
客户端收到：{"code":500,"message":"服务器内部错误"}
```

**请求进入**：外层 → 内层（像剥洋葱皮）  
**响应返回**：内层 → 外层（像穿洋葱皮）

每一层都可以在请求前做预处理，也可以在响应后做日志、异常、清理等后处理。


***
<br/><br/><br/>
> <h3 id="ChainInterceptors实现原理">ChainInterceptors 实现原理</h3>

**核心代码：**

```go
// HGHTTPInterceptor 表示一个可组合的 HTTP 拦截器。
type HGHTTPInterceptor func(http.Handler) http.Handler

// ChainInterceptors 在启动阶段把拦截器链装配为最终 handler，避免请求期重复构建。
func ChainInterceptors(base http.Handler, interceptors ...HGHTTPInterceptor) http.Handler {
    if base == nil {
        return nil
    }

    wrapped := base
    for i := len(interceptors) - 1; i >= 0; i-- {
        if interceptors[i] == nil {
            continue
        }
        wrapped = interceptors[i](wrapped)
    }

    return wrapped
}
```

从后往前包装是为了先构建内层，再把内层作为 `next` 交给外层：

```go
for i := len(interceptors) - 1; i >= 0; i-- {
    wrapped = interceptors[i](wrapped)
}
```

这样保证：**第一个外层参数最先执行进入逻辑，最后一个内层参数最靠近业务逻辑**。


***
<br/><br/><br/>
> <h3 id="洋葱模型图解">洋葱模型图解</h3>

## 函数嵌套调用

中间件的洋葱模型源于函数嵌套调用：外层在 `next.ServeHTTP` 前执行进入逻辑，在 `next.ServeHTTP` 返回后执行退出逻辑。

```go
func MiddlewareA(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("A: 进入")           // 1. 请求前
        next.ServeHTTP(w, r)             // 2. 调用下一层（进入洋葱内部）
        fmt.Println("A: 退出")           // 5. 响应后
    })
}

func MiddlewareB(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("B: 进入")           // 2. 请求前
        next.ServeHTTP(w, r)             // 3. 调用下一层（进入洋葱内部）
        fmt.Println("B: 退出")           // 4. 响应后
    })
}
```

## 组装和执行流程

```text
ChainInterceptors(business, A, B)

组装过程（从后往前）：
┌─────────────────────────────────────────────────────────┐
│  Step 1: wrapped = business                            │
│  Step 2: wrapped = B(business)     // B 包裹 business  │
│  Step 3: wrapped = A(B(business))  // A 包裹 B         │
│                                                         │
│  最终结构：A(B(business))                               │
│  就像：A { B { business } }                             │
└─────────────────────────────────────────────────────────┘

请求到达时的执行顺序：
┌─────────────────────────────────────────────────────────┐
│  请求 ──→ A.ServeHTTP()                                 │
│            │                                            │
│            ├─→ 打印 "A: 进入"                            │
│            │                                            │
│            ├─→ next.ServeHTTP()  (调用 B)               │
│            │     │                                      │
│            │     ├─→ 打印 "B: 进入"                     │
│            │     │                                      │
│            │     ├─→ next.ServeHTTP()  (调用 business)  │
│            │     │     │                                │
│            │     │     └─→ 业务处理                      │
│            │     │                                      │
│            │     ├─→ 打印 "B: 退出"                      │
│            │     │                                      │
│            │     └─→ 返回                               │
│            │                                            │
│            ├─→ 打印 "A: 退出"                            │
│            │                                            │
│            └─→ 返回 ──→ 响应                             │
└─────────────────────────────────────────────────────────┘
```

## 洋葱比喻

```text
        请求 ──→
                    ┌──────────────┐
                    │   中间件 A   │
                    │  ┌────────┐  │
                    │  │中间件 B│  │
                    │  │ ┌────┐ │  │
                    │  │ │业务│ │  │
                    │  │ │处理│ │  │
                    │  │ └────┘ │  │
                    │  └────────┘  │
                    └──────────────┘
                            ↓
        ←── 响应
```

## 递归调用比喻

```go
func A() {
    print("进入 A")
    B()           // A 等待 B 返回
    print("退出 A")
}

func B() {
    print("进入 B")
    business()    // B 等待 business 返回
    print("退出 B")
}

// 输出顺序：
// 进入 A → 进入 B → 业务处理 → 退出 B → 退出 A
```

## 电话流程比喻

```text
打电话找人：

你（客户端）
  ↓ 拨号
总机（中间件 A）─→ "请稍等" ─→ 转接
  ↓
分机（中间件 B）─→ "正在接通" ─→ 转接
  ↓
目标人员（业务处理）─→ "喂，你好"
  ↓ 挂断
分机（中间件 B）←─ "再见"
  ↓
总机（中间件 A）←─ "再见"
  ↓
你（客户端）收到响应
```

## 洋葱模型的好处

```text
┌─────────────────────────────────────────────────────────────┐
│ 1. 统一的请求/响应处理                                        │
│    - 每个中间件都能在请求前后做处理                           │
│    - 不需要在每个 handler 里重复写日志、错误处理等            │
├─────────────────────────────────────────────────────────────┤
│ 2. 责任分离                                                  │
│    - 日志中间件只管日志                                       │
│    - 恢复中间件只管 panic 捕获                                │
│    - 业务 handler 只管业务逻辑                               │
├─────────────────────────────────────────────────────────────┤
│ 3. 可插拔、可组合                                            │
│    - 随时添加/移除中间件                                      │
│    - 中间件顺序可调整                                         │
│    - 不同路由可使用不同中间件组合                             │
├─────────────────────────────────────────────────────────────┤
│ 4. 异常统一处理                                              │
│    - RecoverInterceptor 在外层捕获所有 panic                 │
│    - 避免整个服务崩溃                                         │
└─────────────────────────────────────────────────────────────┘
```


***
<br/><br/><br/>
> <h3 id="运行示例">运行示例</h3>

## 中间件组合

```go
handler := ChainInterceptors(
    businessHandler,
    RequestTIDInterceptor,   // 4 (最内层，最后进入，最先退出)
    AccessLogInterceptor,    // 3
    RecoverInterceptor,      // 2
    JSONHeaderInterceptor,   // 1 (最外层，最先进入，最后退出)
)
```

## 正常请求

```bash
curl http://localhost:8080/hello
```

服务器输出：

```text
========================================
【1-进入】JSONHeaderInterceptor - 设置响应头
【2-进入】RecoverInterceptor - 设置 panic 捕获
【3-进入】AccessLogInterceptor - GET /hello
【4-进入】RequestTIDInterceptor - TID: TID-20260430-ABC123
>>> 【业务处理】开始执行 <<<
>>> 【业务处理】执行完毕 <<<
【4-退出】RequestTIDInterceptor - TID: TID-20260430-ABC123
【3-退出】AccessLogInterceptor - 请求完成
【2-退出】RecoverInterceptor - 正常完成
【1-退出】JSONHeaderInterceptor - 响应已完成
========================================
```

客户端收到：

```json
{"code":0,"message":"Hello, World!"}
```

<br/>

**原始运行输出：**

```bash
$ curl http://localhost:8080/hello

# 服务器输出：
========================================
【1. JSONHeaderInterceptor】进入 - 设置响应头
【2. RecoverInterceptor】进入 - 设置 panic 恢复
【3. AccessLogInterceptor】进入 - GET /hello
【4. RequestTIDInterceptor】进入 - 生成追踪ID: TID-20260430-ABC123
>>> 【业务处理】helloHandler 开始执行 <<<
>>> 【业务处理】处理用户请求 <<<
>>> 【业务处理】返回响应内容 <<<
【4. RequestTIDInterceptor】退出 - 追踪ID: TID-20260430-ABC123
【3. AccessLogInterceptor】退出 - 请求处理完成
【2. RecoverInterceptor】退出 - 正常结束
【1. JSONHeaderInterceptor】退出 - 响应已返回
========================================

# 客户端收到：
{"code":0,"message":"Hello, World!"}
```

---
<br/>

## panic 捕获

```bash
curl http://localhost:8080/panic
```

服务器输出：

```text
========================================
【1-进入】JSONHeaderInterceptor - 设置响应头
【2-进入】RecoverInterceptor - 设置 panic 捕获
【3-进入】AccessLogInterceptor - GET /panic
【4-进入】RequestTIDInterceptor - TID: TID-20260430-ABC123
>>> 【业务处理】即将 panic <<<
【2-捕获】RecoverInterceptor - panic: 模拟数据库连接失败
【1-退出】JSONHeaderInterceptor - 响应已完成
========================================
```

客户端收到：

```json
{"code":500,"message":"服务器内部错误"}
```

<br/>

**原始 panic 输出：**

```bash
$ curl http://localhost:8080/panic

# 服务器输出：
========================================
【1. JSONHeaderInterceptor】进入 - 设置响应头
【2. RecoverInterceptor】进入 - 设置 panic 恢复
【3. AccessLogInterceptor】进入 - GET /panic
【4. RequestTIDInterceptor】进入 - 生成追踪ID: TID-20260430-ABC123
>>> 【业务处理】即将 panic <<<
【2. RecoverInterceptor】捕获 panic: 数据库连接失败！
【1. JSONHeaderInterceptor】退出 - 响应已返回
========================================

# 客户端收到：
{"code":500,"message":"服务器内部错误"}
```

关键点：`panic` 发生时，内层中间件的退出日志可能不会打印；`RecoverInterceptor` 捕获后返回外层继续执行，服务器不会崩溃。

---
<br/>

## 鉴权失败提前终止

```go
func AuthInterceptor(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !isAuthenticated(r) {
            w.WriteHeader(401)
            w.Write([]byte(`{"code":401,"message":"未授权"}`))
            return // 不调用 next，直接返回
        }
        next.ServeHTTP(w, r)
    })
}
```

请求流程：

```text
【1-进入】JSONHeaderInterceptor
【2-进入】RecoverInterceptor
【3-进入】AuthInterceptor
【3-拦截】返回 401，不调用 next
【2-退出】RecoverInterceptor
【1-退出】JSONHeaderInterceptor
```

客户端收到：

```json
{"code":401,"message":"未授权"}
```

<br/>

**执行顺序总结表：**

| 场景 | 进入顺序 | 退出顺序 | 特点 |
|------|---------|---------|------|
| 正常请求 | 1→2→3→4→业务 | 业务→4→3→2→1 | 完整洋葱 |
| panic | 1→2→3→4→业务→panic | 2(捕获)→1 | 内层退出跳过 |
| 鉴权失败 | 1→2→3 | 3→2→1 | 提前终止 |

<br/>

## 运行测试方法

```bash
cd /Users/ganghuang/HGFiles/GitHub/GoProject/src/MLC_GO
go run main.go

# 输入 16 进入中间件演示
# 测试1: curl http://localhost:8080/hello
# 测试2: curl http://localhost:8080/panic
```

代码位置：

```text
TestNotes/ungrammar_pt/middleware_pt/middleware_demo.go
```

<br/>

## 总结

- **链式装配：** `ChainInterceptors` 在启动阶段从后往前包装中间件。
- **请求进入：** 外层 → 内层 → 业务。
- **响应返回：** 业务 → 内层 → 外层。
- **异常处理：** `RecoverInterceptor` 放在较外层，用于统一捕获内层 `panic`。
- **提前终止：** 中间件不调用 `next.ServeHTTP` 时，请求不会继续进入内层或业务处理。


***
<br/><br/><br/>
> <h3 id="HTTPQuery参数解析">HTTP Query 参数解析</h3>

下面这行代码常用于从 URL 查询参数中读取 `limit` 并转换成 `int`：

```go
limit, _ := strconv.Atoi(r.URL.Query().Get("limit"))
```

例如请求：

```text
GET /list?limit=20
```

最终得到：

```go
limit = 20
```

<br/>

## 执行流程

```go
r.URL.Query().Get("limit") // "20"
strconv.Atoi("20")         // (20, nil)
limit = 20
```

整体链路：

```text
HTTP Query String → string → int → limit
```

<br/>

## 逐段解析

```go
r.URL.Query()
```

获取所有 URL 查询参数。例如：

```text
/list?limit=20&page=2
```

会解析成：

```go
map[string][]string{
    "limit": {"20"},
    "page":  {"2"},
}
```

<br/>

```go
r.URL.Query().Get("limit")
```

取出 `limit` 参数的第一个值，返回类型是 `string`；如果参数不存在，返回空字符串 `""`。

<br/>

```go
strconv.Atoi("20")
```

把字符串转成 `int`，返回值是 `(int, error)`：

```go
n, err := strconv.Atoi("20")
// n = 20
// err = nil
```

如果传入非法数字：

```go
n, err := strconv.Atoi("abc")
// n = 0
// err != nil
```

<br/>

```go
limit, _ := strconv.Atoi(...)
```

这是 Go 多返回值的忽略写法：`limit` 接收转换后的 `int`，`_` 丢弃 `error`。

<br/>

## 风险点

直接忽略 `error` 会把缺失参数和非法参数都转换成 `0`，容易让业务误判：

| 请求 | 解析结果 | 问题 |
|------|----------|------|
| `/list` | `Atoi("")` → `limit = 0` | 未传参数被当成 0 |
| `/list?limit=abc` | `Atoi("abc")` → `limit = 0` | 非法参数被当成 0 |

<br/>

## 推荐写法

如果 `limit` 有默认值，优先显式处理错误和非法范围：

```go
limitStr := r.URL.Query().Get("limit")

limit := 10 // 默认值
if v, err := strconv.Atoi(limitStr); err == nil && v > 0 {
    limit = v
}
```

更直接的写法：

```go
limitStr := r.URL.Query().Get("limit")

limit, err := strconv.Atoi(limitStr)
if err != nil || limit <= 0 {
    limit = 10
}
```

核心原则：**外部输入都不可信，Query 参数要同时处理缺失、格式错误和范围非法三类情况**。


***
<br/><br/><br/>
> <h3 id="strconvParseInt字符串转int64">strconv.ParseInt 字符串转 int64</h3>

`strconv.ParseInt(keyword, 10, 64)` 的作用是：**把字符串 `keyword` 按十进制解析成 `int64`**。

```go
n, err := strconv.ParseInt(keyword, 10, 64)
```

函数原型：

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```

参数含义：

| 参数 | 示例 | 说明 |
|------|------|------|
| `s` | `keyword`、`"123"` | 要转换的字符串 |
| `base` | `10` | 按什么进制解析，常用 `10` 表示十进制 |
| `bitSize` | `64` | 限制结果整数位数，`64` 表示按 `int64` 范围解析 |

常见 `base`：

| base | 含义 |
|------|------|
| `10` | 十进制 |
| `2` | 二进制 |
| `8` | 八进制 |
| `16` | 十六进制 |

常见 `bitSize`：

| bitSize | 对应范围 |
|---------|----------|
| `8` | `int8` |
| `16` | `int16` |
| `32` | `int32` |
| `64` | `int64` |

---
<br/>

## 示例

正常数字：

```go
keyword := "123"

n, err := strconv.ParseInt(keyword, 10, 64)
// n = 123
// err = nil
```

非法数字：

```go
keyword := "abc"

n, err := strconv.ParseInt(keyword, 10, 64)
// n = 0
// err != nil
```

负数：

```go
keyword := "-100"

n, err := strconv.ParseInt(keyword, 10, 64)
// n = -100
```

二进制解析：

```go
n, err := strconv.ParseInt("1010", 2, 64)
// n = 10
// err = nil
```

---
<br/>

## 和 Atoi 的区别

`strconv.Atoi(s)` 等价于 `strconv.ParseInt(s, 10, 0)` 后再转成 `int`，返回类型依赖当前平台的 `int` 大小；`ParseInt` 可以显式指定进制和位数，更适合数据库 ID、Redis key、订单号、用户 ID 等需要明确 `int64` 的场景。

| 方法 | 示例 | 返回类型 | 是否可控 bitSize |
|------|------|----------|------------------|
| `Atoi` | `strconv.Atoi("123")` | `int` | 否 |
| `ParseInt` | `strconv.ParseInt("123", 10, 64)` | `int64` | 是 |

典型用途：

```text
?keyword=123
?id=10001
?page=2
```

推荐写法是保留 `err` 并显式处理非法输入，不要把错误静默吞掉：

```go
id, err := strconv.ParseInt(r.URL.Query().Get("id"), 10, 64)
if err != nil || id <= 0 {
    // 返回参数错误或使用默认值
}
```

结论：**`strconv.ParseInt(keyword, 10, 64)` 就是把字符串 `keyword` 按十进制转换成 `int64`，如果不是合法数字就返回错误**。


***
<br/><br/><br/>
> <h3 id="databaseSQLRows与Cursor">database/sql Rows 与 Cursor</h3>

`*sql.Rows` 不是已经全部加载到内存里的数组，而是 Go 对数据库 `Cursor`（游标）的封装：`rows.Next()` 推进游标，`rows.Scan()` 读取当前行，`rows.Err()` 检查遍历过程中的延迟错误，`rows.Close()` 释放游标占用的连接、网络、内存和数据库资源。

```go
rows, err := db.Query(query)
if err != nil {
    return err
}
defer rows.Close()

for rows.Next() {
    if err := rows.Scan(&id, &name); err != nil {
        return err
    }
}

return rows.Err()
```

核心链路：

```text
SQL
  │
  ▼
数据库执行查询
  │
  ▼
数据库创建 Cursor（游标）
  │
  ▼
Go 的 *sql.Rows 持有这个 Cursor
  │
  ▼
rows.Next() → Cursor 向下一行移动
  │
  ▼
rows.Scan() → 读取 Cursor 当前指向的这一行
  │
  ▼
rows.Close() → 关闭 Cursor，释放数据库连接和相关资源
```

---
<br/>

## rows.Next()

`rows.Next()` 表示游标向下一行移动，返回 `true` 时才有当前行可供 `rows.Scan()` 读取；返回 `false` 可能是正常读完，也可能是遍历过程中发生了错误，最终要通过 `rows.Err()` 区分。

例如数据库结果：

| id | name |
| -- | ---- |
| 1  | Tom  |
| 2  | Jack |
| 3  | Lucy |

游标推进过程：

```text
刚开始：
      ↓
未开始

第一次 rows.Next()：
      ↓
第一行 Tom

第二次 rows.Next()：
Tom
      ↓
Jack

第三次 rows.Next()：
Tom
Jack
      ↓
Lucy

第四次 rows.Next()：
Tom
Jack
Lucy

↓
结束，返回 false
```

所以 `rows.Scan(...)` 不需要指定第几行，因为 Cursor 已经记录了当前位置，`Scan` 读取的就是当前行。

---
<br/>

## Cursor 不是错误

`Cursor` 不是异常，而是数据库读取结果集时维护当前位置的对象。所谓 `Cursor 出错`，指的是读取过程中连接、网络或数据库状态异常，错误会被 `database/sql` 记录下来，遍历结束后从 `rows.Err()` 取出。

常见场景：

| 场景 | 表现 | rows.Err() |
|------|------|------------|
| 数据库连接断开 | `rows.Next()` 提前结束 | `connection reset` |
| 网络断开 | Cursor 没读完 | `read tcp ...` |
| 数据库重启 | Cursor 失效 | 返回对应数据库错误 |
| 服务器超时 | 连接被关闭 | 返回超时或连接错误 |

Go 官方 API 没有把 `Next()` 设计成 `(bool, error)`，而是使用固定模式：

```go
for rows.Next() {
    rows.Scan(...)
}

if err := rows.Err(); err != nil {
    return err
}
```

项目中直接 `return rows.Err()`，就是遍历结束后统一返回 Cursor 读取过程中的错误。


***
<br/><br/><br/>
> <h3 id="scanAdminUserRow行扫描">scanAdminUserRow 行扫描</h3>

`scanAdminUserRow(rows, hasEmail)` 的作用是：**把 `rows` 当前指向的一行数据库记录扫描成 `map[string]interface{}`，供后续接口返回或列表组装使用**。

```go
func scanAdminUserRow(rows *sql.Rows, hasEmail bool) (map[string]interface{}, error) {
    var id sql.NullString
    var name string
    var nickName string
    var email sql.NullString
    var mobile string
    var status int

    var err error
    if hasEmail {
        err = rows.Scan(&id, &name, &nickName, &email, &mobile, &status)
    } else {
        err = rows.Scan(&id, &name, &nickName, &mobile, &status)
    }
    if err != nil {
        return nil, err
    }

    item := map[string]interface{}{
        "id":       id.String,
        "name":     name,
        "nickName": nickName,
        "mobile":   mobile,
        "status":   status,
    }
    if hasEmail {
        item["email"] = email.String
    }

    return item, nil
}
```

整体流程：

```text
数据库
  │
  │ Query()
  ▼
rows (*sql.Rows)
  │
  │ rows.Next()
  ▼
当前一行
  │
  │ rows.Scan(...)
  ▼
Go变量
  │
  │ 组装
  ▼
map[string]interface{}
  │
  ▼
返回
```

---
<br/>

## 为什么传入 *sql.Rows

调用方通常是：

```go
for rows.Next() {
    item, err := scanAdminUserRow(rows, hasEmail)
}
```

`rows.Next()` 已经把游标移动到当前行，`scanAdminUserRow(rows, ...)` 内部执行 `rows.Scan(...)` 时，读取的就是当前这一行。

```text
rows
│
├── 第一行
├── 第二行
├── 第三行
└── ...

rows.Next() 后：

rows
      ↓
┌───────────────┐
│ 第一行        │
├───────────────┤
│ 第二行        │
├───────────────┤
│ 第三行        │
└───────────────┘
```

---
<br/>

## sql.NullString

数据库字段可能是 `NULL` 时，不能直接扫描到普通 `string`，否则可能报错：

```text
converting NULL to string is unsupported
```

`sql.NullString` 用来同时保存字符串值和是否有效：

```go
type NullString struct {
    String string
    Valid  bool
}
```

扫描结果示例：

| 数据库值 | String | Valid |
|----------|--------|-------|
| `NULL` | `""` | `false` |
| `10001` | `"10001"` | `true` |

因此 `id`、`email` 使用 `sql.NullString`，是为了兼容数据库 `NULL`；`name`、`nickName`、`mobile` 等字段如果数据库约束为 `NOT NULL`，就可以直接使用普通 `string`。

---
<br/>

## 为什么根据 hasEmail 分两种 Scan

`rows.Scan()` 的参数数量必须和 `SELECT` 字段数量完全一致。

有邮箱字段时：

```sql
SELECT
user_id,
name,
nickname,
email,
mobile,
status
```

对应：

```go
rows.Scan(&id, &name, &nickName, &email, &mobile, &status)
```

没有邮箱字段时：

```sql
SELECT
user_id,
name,
nickname,
mobile,
status
```

对应：

```go
rows.Scan(&id, &name, &nickName, &mobile, &status)
```

如果 SQL 返回 2 个字段，却传入 3 个 Scan 目标变量，会报错：

```text
expected 2 destination arguments in Scan, not 3
```

---
<br/>

## 不要扫描 admin_user.id

注释中的提醒：

```go
// SELECT 的第一个字段固定是 admin_user.user_id
// 不要扫描 admin_user.id
```

意思是后台管理员表可能同时存在两个 ID：

| id | user_id |
| -- | ------- |
| 1  | 10001   |

`id` 是 `admin_user` 表自身的自增主键，`user_id` 才是业务身份字段。前端管理员选择、角色绑定、权限分配等场景应该使用 `user_id`，不能误用 `admin_user.id`。

---
<br/>

## 返回 map 与 NULL 风险

返回 `map[string]interface{}` 的好处是字段灵活，适合直接 JSON 序列化，也不需要额外定义结构体：

```go
json.NewEncoder(w).Encode(item)
```

但直接返回 `id.String`、`email.String` 会把数据库 `NULL` 和空字符串都变成 `""`，调用方无法区分：

```json
{
    "id": ""
}
```

如果业务需要区分 `NULL` 和空字符串，建议显式判断 `Valid`：

```go
result := map[string]interface{}{
    "name":     name,
    "nickName": nickName,
    "mobile":   mobile,
    "status":   status,
}

if id.Valid {
    result["id"] = id.String
} else {
    result["id"] = nil
}

if hasEmail {
    if email.Valid {
        result["email"] = email.String
    } else {
        result["email"] = nil
    }
}

return result, nil
```

长期维护的项目更推荐定义 `AdminUser` 结构体，能获得类型安全、IDE 自动补全和编译期检查；简单动态接口使用 `map[string]interface{}` 更快，但要明确字段含义和 `NULL` 处理策略。

---
<br/>

## 函数执行流程

```text
rows.Next()
      │
      ▼
当前游标指向一行
      │
      ▼
rows.Scan(...)
      │
      ▼
数据库字段
      │
      ├──── user_id ─────► sql.NullString(id)
      ├──── name ────────► string(name)
      ├──── nickname ────► string(nickName)
      ├──── email ───────► sql.NullString(email)
      ├──── mobile ──────► string(mobile)
      └──── status ──────► int(status)
      │
      ▼
读取变量中的值
      │
      ▼
组装 map[string]interface{}
      │
      ▼
返回给调用者
      │
      ▼
appendAdmins()
      │
      ▼
去重 → append 到 list → 返回接口响应
```


***
<br/><br/><br/>
> <h3 id="seen去重与map预分配">seen 去重与 map 预分配</h3>

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

> <h1 id="工程表">工程表</h1>

## **‌菜单权限表**

```sql
CREATE TABLE `permission` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `code` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '权限编码',
  `type` tinyint NOT NULL COMMENT '1:菜单  2:操作',
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '权限名称',
  `page_path` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '菜单路径',
  `parent_id` bigint NOT NULL DEFAULT '-1' COMMENT '父级权限ID',
  `status` tinyint NOT NULL COMMENT '1:正常 -1:禁用',
  `sort` int NOT NULL DEFAULT '1',
  `desc` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '权限描述',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `update_by` bigint NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `idx_code` (`code`,`status`) USING BTREE,
  KEY `idx_name` (`name`) USING BTREE,
  KEY `idx_parent_id` (`parent_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
ROW_FORMAT=DYNAMIC COMMENT='权限清单表';
```

<br/>

## 管理员表

```sql
CREATE TABLE `admin_user` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键-管理员ID表',
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '名字',
  `nick_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '',
  `mobile` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '手机',
  `lark_open_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '1:正常 -1:禁用',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `create_by` bigint NOT NULL DEFAULT '0',
  `update_by` bigint NOT NULL DEFAULT '1',
  `sex` tinyint NOT NULL DEFAULT '3' COMMENT '3:其他 1: 男 2: 女',
  `is_delete` tinyint NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `idx_mobile` (`mobile`) USING BTREE,
  KEY `idx_name` (`name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=57 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC;
```

---
### 字段说明💡
1. **核心身份字段**：`name`(姓名)、`nick_name`(昵称)、`mobile`(手机号，唯一索引)、`password`(密码)、`lark_open_id`(飞书OpenID，适配企业办公登录)
2. **状态控制**：`status`(账号启用/禁用)、`is_delete`(逻辑删除，不做物理删除)
3. **基础属性**：`sex`(性别)
4. **审计字段**：`create_at`/`update_at`(自动维护创建/更新时间)、`create_by`/`update_by`(记录操作人ID)
5. **索引设计**：手机号唯一索引防重复，姓名索引优化模糊查询


<br/>

## 角色权限表

```sql
CREATE TABLE `role_permission` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `role_id` bigint NOT NULL COMMENT '角色ID',
  `permission_id` bigint NOT NULL COMMENT '权限ID',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `create_by` bigint NOT NULL DEFAULT '0',
  `update_by` bigint NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_role_id` (`role_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
ROW_FORMAT=DYNAMIC COMMENT='角色权限表';
```

---
### 表结构说明💡
1. **表用途**：**多对多中间表**，用于绑定角色与权限的关联关系，实现一个角色分配多个权限、一个权限可被多个角色复用。
2. **核心关联字段**
   - `role_id`：关联角色表主键
   - `permission_id`：关联菜单权限表主键
3. **审计字段**：`create_at`/`update_at`/`create_by`/`update_by`，记录关联关系的创建、更新时间与操作人。
4. **索引设计**：`idx_role_id` 索引，优化按角色批量查询权限的性能。
5. **优化建议**：可增加**联合唯一索引** `UNIQUE KEY uk_role_permission(role_id,permission_id)`，防止同一角色重复绑定相同权限。

<br/>


## 管理员角色表

```sql
CREATE TABLE `admin_user_role` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `admin_user_id` bigint NOT NULL COMMENT '管理员ID',
  `role_id` bigint NOT NULL COMMENT '角色ID',
  `update_at` datetime NOT NULL,
  `update_by` bigint NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uidx_aduid_role_id` (`admin_user_id`,`role_id`),
  KEY `idx_role_id` (`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

---
### 表结构说明💡
1. **表用途**：RBAC权限体系中**管理员-角色**的多对多中间表，实现一个管理员可绑定多个角色、一个角色可分配给多个管理员。
2. **核心约束**
   - `uidx_aduid_role_id`：**联合唯一索引**，防止同一管理员重复绑定相同角色
   - `idx_role_id`：角色ID普通索引，优化按角色批量查询管理员的性能
3. **审计字段**：`update_at`记录绑定关系更新时间，`update_by`记录操作人ID
4. **补充优化**：可补全`create_at`、`create_by`字段，完善全链路操作日志，与前面几张权限表设计保持统一


<br/>

## 用户主表

```sql
CREATE TABLE `user` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '全局user_id',
  `nick_name` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `sex` tinyint NOT NULL COMMENT '默认0 其他，1: 男 2: 女',
  `password` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '默认1: 正常  -1: 禁用',
  `icon_key` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `create_at` datetime NOT NULL,
  `last_login_at` datetime DEFAULT NULL,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_name` (`nick_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='用户主表';
```

---
### 表结构说明💡
1. **基础信息字段**
   - `nick_name`：用户昵称，长度64，建立索引优化昵称检索
   - `sex`：性别枚举，0-其他、1-男、2-女
   - `icon_key`：头像资源标识，用于存储对象存储地址/键值
2. **账号安全字段**
   - `password`：用户密码，存储加密后密文
   - `status`：账号状态，1正常、-1禁用
3. **时间审计字段**
   - `create_at`：账号创建时间
   - `last_login_at`：最后登录时间，用于风控、活跃度统计
   - `update_at`：信息更新时间，自动更新
4. **设计特点**
   - 使用 `bigint unsigned` 无符号主键，支持更大用户量级
   - 字符集统一使用 `utf8mb4`，兼容emoji表情存储
   - 索引精简，仅对高频查询的昵称建立普通索引

<br/>

## a. 微信用户表

```sql
CREATE TABLE `wechat_user` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID,自增(无实际用途)',
  `user_id` bigint NOT NULL COMMENT '全局用户id,user表主键',
  `union_id` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '微信union_id',
  `nick_name` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '微信昵称',
  `icon_url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '微信头像地址',
  `create_at` datetime NOT NULL,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uidx_user` (`user_id`) USING BTREE,
  UNIQUE KEY `uidx_union` (`union_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='微信用户表';
```

---
### 表结构说明💡
1. **表用途**：第三方微信登录关联表，与**用户主表(user)** 一对一绑定，用于存储微信授权信息，实现微信快捷登录。
2. **核心关联&唯一约束**
   - `user_id`：关联用户主表主键，**唯一索引**保证一个用户仅绑定一个微信账号
   - `union_id`：微信全局唯一标识，**唯一索引**防止重复授权注册
3. **微信信息字段**：存储微信昵称、头像地址，同步用户微信端资料
4. **时间字段**：`create_at`记录绑定时间，`update_at`自动维护信息更新时间
5. **设计特点**：采用冗余自增主键`id`，业务实际以`user_id`、`union_id`做唯一校验，适配微信开放平台多端登录体系

<br/>

## c. 应用用户表

```sql
CREATE TABLE `app_user` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '自增(无实际用途)',
  `user_id` bigint NOT NULL COMMENT '全局用户id,user表主键',
  `app_code` int NOT NULL COMMENT '1000=公众号，1001=小程序，其他进行扩展，固定值不能变更',
  `open_id` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '微信对应应用openid',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '默认1: 正常 -1: 禁用',
  `create_at` datetime NOT NULL,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uidx_user_appcode` (`user_id`,`app_code`),
  UNIQUE KEY `uidx_openId` (`open_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='应用用户表';
```

---
### 表结构说明💡
1. **表用途**：区分**公众号、小程序**等不同微信应用来源的用户绑定关系，实现**一个全局用户可绑定多个微信应用**，支持多端登录。
2. **核心唯一约束**
   - `uidx_user_appcode`：联合唯一索引，保证**同一个用户在同一个微信应用下只能绑定一次**
   - `uidx_openId`：open_id全局唯一，避免重复注册
3. **应用区分**：`app_code` 固定编码区分渠道（1000公众号、1001小程序），便于后续扩展其他第三方应用
4. **状态与审计**：`status`控制应用渠道账号启用/禁用；`create_at`/`update_at`记录绑定与更新时间
5. **与微信用户表区别**：`wechat_user`存**union_id（微信全局唯一）**，`app_user`存**open_id（单应用唯一）**，二者配合实现微信多端账号打通

<br/>

## 用户权益表（用户购买的课程商品权益）

```sql
CREATE TABLE `user_course_goods` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `user_id` bigint NOT NULL COMMENT '用户ID',
  `order_id` bigint NOT NULL COMMENT '订单ID',
  `goods_id` bigint NOT NULL COMMENT '商品ID',
  `goods_type` tinyint NOT NULL COMMENT '1.课程商品',
  `buy_time` bigint NOT NULL COMMENT '购买时间',
  `service_expire_time` bigint NOT NULL COMMENT '服务到期时间',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`user_id`),
  KEY `idx_cid` (`goods_id`),
  KEY `idx_se_ex_t` (`service_expire_time`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户购买的课程列表';
```

---
### 表结构说明💡
1. **表用途**：记录用户购买课程后的**权益凭证**，用于校验用户是否拥有对应课程的观看、学习权限。
2. **核心关联字段**
   - `user_id`：关联全局用户ID
   - `order_id`：关联订单ID，溯源购买来源
   - `goods_id`：关联课程商品ID
3. **时间字段**：使用**时间戳(bigint)** 存储购买时间、服务到期时间，便于跨时区处理与计算
4. **索引设计**
   - `idx_uid`：按用户快速查询已购课程
   - `idx_cid`：按课程查询购买用户
   - `idx_se_ex_t`：按到期时间索引，用于批量处理即将过期/已过期的权益
5. **业务扩展**：`goods_type`预留类型字段，可后续拓展其他类型付费商品


<br/>

## 课程商品主表

```sql
CREATE TABLE `course_goods` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `cover_key` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `intro_key` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '介绍图资源标识',
  `desc` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `goods_price` bigint NOT NULL DEFAULT '0' COMMENT '商品价格',
  `service_time` tinyint NOT NULL COMMENT '辅导服务时长\r\n1: 一个月 2: 三个月 3: 半年 4: 一年',
  `sale_type` tinyint NOT NULL COMMENT '1: 免费 2: 收费',
  `status` tinyint NOT NULL DEFAULT '-1' COMMENT '-1: 下架 1: 上架',
  `create_at` datetime NOT NULL,
  `create_by` bigint NOT NULL DEFAULT '0',
  `update_at` datetime NOT NULL,
  `update_by` bigint NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uidx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='课程商品表';
```

---
### 表结构说明💡
1. **表用途**：存储课程商品基础信息，是**用户权益表**的主表，定义课程名称、价格、服务周期、上下架状态。
2. **核心业务字段**
   - `name`：课程名称，**唯一索引**防止重复创建同名课程
   - `cover_key`/`intro_key`：封面图、介绍图的资源存储标识
   - `goods_price`：商品价格，使用`bigint`存储**分**，避免浮点数精度问题
   - `service_time`：服务周期枚举，定义辅导有效时长
   - `sale_type`：售卖类型，区分免费/付费课程
   - `status`：上架状态，控制前端是否展示
3. **审计字段**：记录创建/更新时间与操作人ID，用于后台管理溯源
4. **关联关系**：与`user_course_goods`一对多关联，一个课程可被多个用户购买


<br/>

##  课程目录表

```sql
CREATE TABLE `course_catalog` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `parent_id` bigint NOT NULL DEFAULT '-1',
  `level` int NOT NULL DEFAULT '1',
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `good_id` bigint NOT NULL,
  `sort` bigint NOT NULL,
  `update_at` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `update_by` bigint NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_course` (`course_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='课程目录表';
```

---
### 表结构说明💡
1. **表用途**：存储课程的**树形层级目录**，支持章节、小节多级结构，用于课程内容展示。
2. **树形结构字段**
   - `parent_id`：父级目录ID，`-1`代表一级根目录，实现无限级树形结构
   - `level`：层级深度，1=一级目录、2=二级目录，用于区分章节/小节
   - `sort`：排序字段，控制目录展示顺序
3. **关联字段**
   - `good_id`：关联课程商品主表ID，绑定归属课程
   - `idx_course`：课程ID索引，快速查询某课程下所有目录
4. **审计字段**：`update_at`自动更新、`update_by`记录操作人，适配后台编辑维护
5. **小瑕疵**：索引字段为`course_id`，实际表中字段为`good_id`，建议修正索引为 `KEY idx_good_id (good_id)` 保证匹配

<br/>

## b. 课程课时表

```sql
CREATE TABLE `course_lessons` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `goods_id` bigint NOT NULL,
  `catalog_id` bigint NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '课时名称',
  `enable_trial` tinyint NOT NULL DEFAULT '0' COMMENT '1:试听 其他值表示非试听',
  `status` tinyint NOT NULL COMMENT '1:启用 -1: 禁用',
  `video_key` varchar(255) COLLATE utf8mb4_general_ci NOT NULL COMMENT '文件key',
  `detail` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '课时详情',
  `homework` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '课后练习',
  `sort` tinyint NOT NULL DEFAULT '0',
  `update_at` datetime NOT NULL,
  `update_by` bigint NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_gid` (`goods_id`),
  KEY `idx_ct_id` (`catalog_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='课程课时表';
```

---
### 表结构说明💡
1. **表用途**：存储课程**最小播放单元（课时）**，关联课程目录，承载视频、课时详情、课后作业等核心教学内容。
2. **核心关联**
   - `goods_id`：关联课程商品主表
   - `catalog_id`：关联课程目录表，归属某一章节/小节
3. **业务控制字段**
   - `enable_trial`：试听开关，免费试看核心字段
   - `status`：控制课时是否启用展示
   - `video_key`：视频文件资源标识，对接对象存储
   - `sort`：课时排序，控制同目录下课时播放顺序
4. **富文本内容**：`detail`/`homework` 使用 `text` 类型，支持存储长文本、富文本格式的课时详情与作业
5. **索引设计**：分别对课程ID、目录ID建立索引，快速查询对应目录下的所有课时

<br/>

##  订单主表（完整补全版SQL）

```sql
CREATE TABLE `orders` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `order_no` char(18) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '订单编号',
  `user_id` bigint NOT NULL COMMENT '用户ID',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '-1:已取消, 1:待支付 2:已支付(待发货) 3:已完成',
  `order_source` tinyint NOT NULL COMMENT '1: 用户下单 2: 管理后台 3: 系统赠送',
  `order_amount` bigint NOT NULL COMMENT '订单金额=支付金额，单位分',
  `order_origin_amount` bigint NOT NULL COMMENT '订单金额-商品原价，单位分',
  `payment_amount` bigint NOT NULL COMMENT '支付金额-实际支付金额，单位分',
  `trade_no` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '第三方支付单号',
  `inner_trade_no` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '内部交易单号',
  `order_desc` mediumtext CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '订单描述',
  `payment_at` bigint NOT NULL DEFAULT '0' COMMENT '订单支付时间，毫秒时间戳',
  `user_remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '用户备注',
  `receiver_confirm_at` bigint DEFAULT NULL COMMENT '确认收货时间',
  `receiver_confirm_type` tinyint DEFAULT NULL COMMENT '确认收货方式 1: 用户确认收货 99:发货自动确认',
  `refund_amount` bigint NOT NULL DEFAULT '0' COMMENT '订单退款金额',
  `refund_at` bigint DEFAULT NULL COMMENT '退款时间，毫秒时间戳',
  `cancel_at` bigint DEFAULT NULL COMMENT '取消时间，毫秒时间戳',
  `cancel_type` tinyint DEFAULT NULL COMMENT '1: 用户取消 2: 客服取消 3: 超时取消',
  `cancel_by` bigint DEFAULT NULL COMMENT '取消人ID，根据取消类型判断是用户还是客服，-1为系统',
  `cancel_reason` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '取消原因',
  `create_at` bigint NOT NULL COMMENT '订单创建时间，毫秒时间戳',
  `create_by` bigint NOT NULL COMMENT '订单创建人ID，根据order_source判断是用户，还是客服ID，系统为0',
  `transfer_at` bigint NOT NULL DEFAULT '0' COMMENT '转交时间',
  `transfer_by` bigint NOT NULL DEFAULT '0' COMMENT '转交人',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_odno` (`order_no`) USING BTREE,
  KEY `idx_uid` (`user_id`) USING BTREE,
  KEY `idx_create` (`create_at`) USING BTREE,
  KEY `idx_trade_no` (`trade_no`) USING BTREE,
  KEY `idx_pay_at` (`payment_at`) USING BTREE,
  KEY `idx_osrc` (`order_source`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='订单主表';
```

---
### 补充&修正说明💡
1. **缺失字段补全**
   - 补全`trade_no`、`inner_trade_no`、`order_desc`、`user_remark`、`cancel_reason`等字段的**默认值、注释、字段闭合**
   - 完善枚举注释：订单状态、下单来源、取消类型、确认收货方式
2. **索引格式统一**
   原SQL索引格式错乱，统一规范为标准MySQL索引语法，去除重复冗余内容
3. **时间类型规范**
   全表使用**bigint毫秒时间戳**，和你之前用户表、权益表时间存储风格保持一致
4. **业务逻辑完善**
   补充创建人`create_by`注释，区分用户/客服/系统下单场景，适配后台订单管理


<br/>

## a. 订单对象表（完整补全版）

```sql
CREATE TABLE `order_items` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `order_id` bigint NOT NULL COMMENT '订单ID',
  `user_id` bigint NOT NULL COMMENT '用户ID',
  `goods_id` bigint NOT NULL COMMENT '商品ID',
  `goods_type` tinyint NOT NULL DEFAULT '1' COMMENT '1:课程商品',
  `quantity` int NOT NULL DEFAULT '1' COMMENT '商品数量',
  `goods_snap` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '商品快照',
  PRIMARY KEY (`id`),
  KEY `idx_oid` (`order_id`),
  KEY `idx_gid` (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='订单商品对象表';
```

---
### 表结构说明💡
1. **表用途**：订单主表的**子表（订单明细）**，存储一个订单内购买的课程商品明细，支持一订单多商品。
2. **核心关联字段**
   - `order_id`：关联订单主表ID，建立索引快速查询订单下所有商品
   - `user_id`：冗余存储用户ID，方便直接查询用户订单商品
   - `goods_id`：关联课程商品表ID
3. **关键字段**
   - `goods_snap`：**商品快照**，存储下单时商品信息（JSON格式），防止商品后续修改导致订单历史信息丢失
   - `quantity`：购买数量，课程类商品一般默认为1
4. **索引设计**：对订单ID、商品ID建立索引，适配订单明细查询、商品销量统计场景
5. **设计规范**：和前面订单主表、课程商品表字段命名、字符集完全统一


<br/>

## 短信模板表（完整补全版SQL）

```sql
CREATE TABLE `sms_template` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `scene_code` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '场景编码，唯一标识业务场景',
  `sign_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '短信签名',
  `platform_tmpl_id` int NOT NULL COMMENT '第三方短信平台的模版ID',
  `tmpl_str` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '短信模板内容',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '默认1: 正常 -1: 禁用',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `admin_user_id` bigint NOT NULL DEFAULT '0' COMMENT '管理后台admin_user_id',
  `platform` char(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '短信平台，如tencent(腾讯云)',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uidx_scene` (`scene_code`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='短信模板表';
```

---
### 表结构说明💡
1. **表用途**：统一管理系统内各类业务短信模板（验证码、通知、营销短信），对接第三方短信平台，实现模板配置化。
2. **核心业务字段**
   - `scene_code`：**唯一场景编码**，用于代码中指定发送对应模板，全局唯一索引约束
   - `sign_name`：短信签名，合规发送必备
   - `platform_tmpl_id`：第三方平台（腾讯云/阿里云等）审核通过的模板ID
   - `tmpl_str`：模板文本，支持变量占位符
   - `platform`：区分短信服务商，便于多平台兼容切换
3. **状态与审计**
   - `status`：控制模板启用/禁用
   - `create_at`/`update_at`：自动维护时间
   - `admin_user_id`：记录后台配置人ID
4. **设计规范**：字符集、存储格式与项目内其他表完全统一，适配后台配置管理

<br/>

## 云存储的资源文件表（完整补全版SQL）

```sql
CREATE TABLE `resource_upload_files` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `scene` char(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '业务场景编码',
  `file_key` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '文件云存储唯一标识key',
  `user_id` bigint NOT NULL COMMENT '用户id',
  `user_type` bigint NOT NULL COMMENT '用户类型 1:普通用户 2:后台管理员',
  `file_type` char(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '文件类型 如img、video、doc',
  `file_size` bigint NOT NULL COMMENT '文件大小对应字节数',
  `file_name` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '文件原始名称',
  `upload_client_ip` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '上传客户端IP',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '文件创建时间=上传时间',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uidx_key` (`file_key`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='云存储资源文件表';
```

---
### 表结构说明💡
1. **表用途**：统一记录**所有上传至对象存储（OSS/COS）**的文件信息，实现文件溯源、权限管控、业务场景绑定。
2. **核心字段**
   - `file_key`：云存储文件唯一标识，**唯一索引**防止重复上传
   - `scene`：业务场景编码，区分头像、课程封面、课时视频、作业文件等场景
   - `user_id`+`user_type`：区分普通用户/管理员上传，用于权限校验
   - `file_type`：区分图片、视频、文档，便于业务筛选
   - `file_size`：字节存储，精准记录文件大小
3. **审计字段**：记录上传IP、上传时间，用于风控与日志追溯
4. **设计规范**：与项目其他表字符集、索引风格保持一致，可与课程、用户、订单表关联使用


<br/><br/><br/>

***
<br/>

> <h1 id="后台管理接口设计">后台管理接口设计</h1>

# 五、后台管理接口设计（Go课程商城）
## 1. 菜单权限模块

```sh
1. 添加权限菜单
POST /api/admin/v1/perm/create
2. 更改权限菜单
POST /api/admin/v1/perm/update
3. 更新权限菜单状态
POST /api/admin/v1/perm/update_status
4. 获取权限菜单列表
GET /api/admin/v1/perm/list
5. 删除权限菜单
POST /api/admin/v1/perm/delete
```

## 2. 管理员模块

```sh
1. 添加管理员(含角色)
POST /api/admin/v1/user/create
2. 更新管理员(含角色)
POST /api/admin/v1/user/update
3. 更新管理员状态
POST /api/admin/v1/user/update_status
4. 获取管理员列表
GET /api/admin/v1/user/list
5. 获取管理员信息(返回角色)
GET /api/admin/v1/user/info
6. 更换管理员手机号
POST /api/admin/v1/user/change_mobile
7. 管理员绑定飞书账号
POST /api/admin/v1/user/lark/bind
8. 管理员解绑飞书账号
POST /api/admin/v1/user/lark/unbind
```


<br/><br/><br/>

***
<br/>

> <h1 id="分布式限流-Lua脚本">[分布式限流-Lua脚本](../Proj/MLC_GO知识点.md#分布式限流-Lua脚本)</h1>
