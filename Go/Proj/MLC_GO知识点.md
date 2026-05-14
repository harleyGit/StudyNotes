- [Go 知识点](#Go知识点)
	- [中间件链式调用](#中间件链式调用)
	- [完整请求流程](#完整请求流程)
	- [ChainInterceptors 实现原理](#ChainInterceptors实现原理)
	- [洋葱模型图解](#洋葱模型图解)
	- [运行示例](#运行示例)


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
