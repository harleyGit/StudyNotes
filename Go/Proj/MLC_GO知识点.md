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
	- [Tx ExecContext 事务执行](#TxExecContext事务执行)
	- [管理员角色批量绑定](#管理员角色批量绑定)
	- [角色列表游标分页 SQL](#角色列表游标分页SQL)
	- [动态 UPDATE user_security](#动态UPDATEuser_security)
	- [图片 DataURL 解析](#图片DataURL解析)
	- [文件真实 MIME 检测](#文件真实MIME检测)
	- [strings.Builder 字符串清洗](#stringsBuilder字符串清洗)
	- [os.MkdirAll 递归创建目录](#osMkdirAll递归创建目录)
	- [json.RawMessage 延迟解析](#jsonRawMessage延迟解析)
	- [Mac Go 环境升级与重新配置](#MacGo环境升级与重新配置)
	- [任务生命周期控制](#任务生命周期控制)
	- [go-redis 渐进式遍历 Scan](#go-redis渐进式遍历Scan)
	- [在线高 QPS 业务不要依赖 Redis Scan](#在线高QPS业务不要依赖RedisScan)
	- [Scan 通过 .Result 获取结果](#Scan通过-Result获取结果)
	- [Redis ZSET 与 ZREM](#RedisZSET与ZREM)
	- [视频流式读取 io.LimitReader + ReadAll](#视频流式读取ioLimitReaderReadAll)
	- [千万级视频业务标准架构](#千万级视频业务标准架构)
	- [INSERT ... ON DUPLICATE KEY UPDATE](#INSERTONDUPLICATEKEYUPDATE)
	- [客户端时间统一解析为 UTC](#客户端时间统一解析为UTC)
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


***
<br/><br/><br/>
> <h3 id="TxExecContext事务执行">Tx ExecContext 事务执行</h3>

`tx.ExecContext(ctx, query, args...)` 是 Go 标准库 `database/sql` 中 `*sql.Tx` 的事务执行方法：**在事务绑定的连接里执行一条不返回结果集的 SQL，并通过 `context.Context` 控制超时、取消和链路生命周期**。

```go
func (tx *Tx) ExecContext(ctx context.Context, query string, args ...any) (Result, error)
```

适合执行 `INSERT`、`UPDATE`、`DELETE`、`CREATE / ALTER / DROP` 等非查询 SQL；查询多行数据应使用 `QueryContext`。

<br/>

## 基本用法

```go
tx, err := db.BeginTx(ctx, nil)
if err != nil {
    return err
}
defer tx.Rollback()

res, err := tx.ExecContext(ctx, "UPDATE user SET name=? WHERE id=?", "Tom", 1)
if err != nil {
    return err
}

n, err := res.RowsAffected()
if err != nil {
    return err
}
fmt.Println(n)

return tx.Commit()
```

执行流程：

```text
Tx.ExecContext
   ↓
检查事务是否已提交/回滚
   ↓
使用 Tx 绑定的 Conn
   ↓
调用 driver.ExecContext
   ↓
数据库执行 SQL
   ↓
返回 sql.Result / error
```

`ctx` 用于控制 SQL 生命周期，例如 `context.WithTimeout(context.Background(), 2*time.Second)` 超时后会中断数据库请求；`args...` 对应 SQL 中的 `?` 占位符，由驱动做参数绑定，避免手动拼接造成 SQL 注入。

---
<br/>

## Result 返回值

```go
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```

| 方法 | 作用 | 注意点 |
|------|------|--------|
| `RowsAffected()` | 返回受影响行数，最常用于 `UPDATE / DELETE` | 可用于判断是否真的更新到数据 |
| `LastInsertId()` | 返回插入后的自增 ID | MySQL 常用，PostgreSQL 通常使用 `RETURNING` |

如果业务不需要返回值，可以用 `_` 丢弃：

```go
if _, err := tx.ExecContext(ctx, query, args...); err != nil {
    return err
}
```

---
<br/>

## 和 db.ExecContext 的区别

| 方法 | 是否事务 | 连接行为 | 提交方式 |
|------|----------|----------|----------|
| `db.ExecContext` | 否 | 从连接池拿连接，执行完释放 | 自动提交 |
| `tx.ExecContext` | 是 | 复用事务独占连接 | 必须 `Commit()`，失败 `Rollback()` |

事务中的多条 `ExecContext` 要么全部成功提交，要么回滚：

```go
tx, _ := db.BeginTx(ctx, nil)

tx.ExecContext(ctx, "UPDATE account SET balance=balance-100 WHERE id=?", 1)
tx.ExecContext(ctx, "UPDATE account SET balance=balance+100 WHERE id=?", 2)

tx.Commit()
```

常见坑：忘记 `Commit()` 数据不会落库；错误路径没有 `Rollback()` 可能导致事务和连接占用；`ctx` 超时后当前 SQL 会被中断，事务对象也可能变为不可继续使用。


***
<br/><br/><br/>
> <h3 id="管理员角色批量绑定">管理员角色批量绑定</h3>

管理员角色绑定本质是向 `admin_user_role` 中间表写入 `admin_user_id + role_id` 多对多关系。核心要求是：**同一事务内批量写入、校验 roleID、去重、失败统一回滚**。

```sql
admin_user_role
-------------------------
admin_user_id | role_id
```

<br/>

## 逐条插入写法

```go
stmt, err := tx.PrepareContext(ctx, SQLQueriesPackage.InsertOpsAdminUserRoleSQL)
if err != nil {
    return err
}
defer stmt.Close()

seen := make(map[int64]struct{}, len(roleIDs))
for _, roleIDText := range roleIDs {
    roleID, err := strconv.ParseInt(roleIDText, 10, 64)
    if err != nil || roleID <= 0 {
        return fmt.Errorf("invalid roleID")
    }

    if _, ok := seen[roleID]; ok {
        continue
    }
    seen[roleID] = struct{}{}

    if _, err := stmt.ExecContext(ctx, adminUserID, roleID); err != nil {
        return err
    }
}
```

对应 SQL 通常是：

```sql
INSERT INTO admin_user_role(admin_user_id, role_id)
VALUES(?, ?)
```

执行流程：

```text
Prepare SQL
   ↓
创建 seen set
   ↓
遍历 roleIDs
   ↓
ParseInt 校验
   ↓
去重判断
   ↓
Exec 插入一行
   ↓
成功继续 / 失败返回
```

`map[int64]struct{}` 是 Go 常见 set 写法，`struct{}{}` 不保存额外值，只表达 key 是否存在，查重平均复杂度为 `O(1)`。

---
<br/>

## 批量 INSERT 写法

逐条 `ExecContext` 简单、容易控制错误，但 roleIDs 很多时会产生 N 次数据库交互。更高性能的 MySQL 写法是拼接批量 `VALUES`，并用唯一键冲突分支忽略重复绑定：

```go
querySQL := "INSERT INTO `admin_user_role` (`admin_user_id`, `role_id`, `update_at`, `update_by`) VALUES " + strings.Join(valueParts, ",")
querySQL += " ON DUPLICATE KEY UPDATE `role_id` = `role_id`"

_, err := tx.ExecContext(ctx, querySQL, args...)
```

假设插入 2 条数据，最终 SQL 形态是：

```sql
INSERT INTO `admin_user_role` (`admin_user_id`, `role_id`, `update_at`, `update_by`)
VALUES (?,?,?,?),(?,?,?,?)
ON DUPLICATE KEY UPDATE `role_id` = `role_id`
```

`valueParts` 中每个元素都是 `(?,?,?,?)`，`args` 长度必须等于 `4 * 记录数`，顺序依次对应所有占位符。

---
<br/>

## ON DUPLICATE KEY UPDATE

必须先有联合唯一索引，否则冲突分支不会按预期生效：

```sql
UNIQUE KEY uidx_aduid_role_id (admin_user_id, role_id)
```

```sql
ON DUPLICATE KEY UPDATE `role_id` = `role_id`
```

含义是：插入新关系时正常新增；如果同一管理员和角色已存在，则把 `role_id` 更新为自身，等价于不修改原行但不报重复键错误。相比 `INSERT IGNORE`，`ON DUPLICATE KEY` 只处理唯一键冲突，字段长度、参数数量、连接异常等其他错误仍会正常抛出，更适合工程代码。

注意点：批量数据过大要分批，避免 SQL 过长；如果需要统计新增数量，不要丢弃 `sql.Result`，可读取 `RowsAffected()`。


***
<br/><br/><br/>
> <h3 id="角色列表游标分页SQL">角色列表游标分页 SQL</h3>

这条 SQL 是典型的 **Cursor Pagination / 游标分页**：从 `role` 表中取出启用状态的数据，按 `id` 倒序，加载 `id < cursor` 的下一页。

```sql
SELECT `role_id`, `id`, `name`, `description`, `create_at`
FROM `role`
WHERE `status` = 1
  AND `id` < ?
ORDER BY `id` DESC
LIMIT ?
```

执行逻辑：

```text
status = 1
AND id < cursor
   ↓
按 id DESC 排序
   ↓
LIMIT pageSize
   ↓
返回下一页数据
```

例如参数是 `id < 1000`、`LIMIT 10`，返回结果通常是 `999 ~ 990` 这一段。

---
<br/>

## 为什么不用 OFFSET

OFFSET 分页越往后越慢：

```sql
SELECT * FROM role
LIMIT 10 OFFSET 100000
```

数据库需要扫描并丢弃前 `100000` 行，再返回后面的 10 行。游标分页使用 `WHERE id < ? ORDER BY id DESC LIMIT ?`，能直接从上一次位置继续读，性能更稳定。

| 方式 | 性能特点 |
|------|----------|
| `LIMIT ... OFFSET ...` | 页码越大越慢 |
| `id < ? ORDER BY id DESC LIMIT ?` | 基于索引定位，稳定加载下一页 |

典型请求链路：第一次请求不带 cursor，只按 `status=1 ORDER BY id DESC LIMIT 10` 取最新 10 条；下一页传上一页最后一条的 `id`，例如 `WHERE id < 990 LIMIT 10`。

---
<br/>

## 索引要求

推荐索引：

```sql
CREATE INDEX idx_role_status_id ON role(status, id);
```

没有合适索引时，可能出现全表扫描和 filesort，分页接口在大数据量下会变成性能瓶颈。


***
<br/><br/><br/>
> <h3 id="动态UPDATEuser_security">动态 UPDATE user_security</h3>

这段代码根据 `setClauses` 动态拼接 `UPDATE user_security` 的 SET 部分，再把 `userID` 追加到参数列表末尾，最后在事务中执行更新。

```go
query := fmt.Sprintf("UPDATE user_security SET %s WHERE user_id = ?", strings.Join(setClauses, ", "))
args = append(args, userID)
if _, err := tx.ExecContext(ctx, query, args...); err != nil {
    return wrapUserSecurityWriteErr("update user security", err)
}
```

假设：

```go
setClauses := []string{"password = ?", "salt = ?"}
args := []any{passwordHash, salt}
```

拼接后得到：

```sql
UPDATE user_security SET password = ?, salt = ? WHERE user_id = ?
```

参数顺序是：

```text
passwordHash → salt → userID
```

关键点：`strings.Join(setClauses, ", ")` 只负责拼接字段赋值片段，真实值仍通过 `args...` 绑定到占位符，避免把用户输入直接拼进 SQL。`_` 表示忽略 `sql.Result`；如果业务需要判断是否更新到用户，可接收 `res` 并读取 `RowsAffected()`。

`wrapUserSecurityWriteErr("update user security", err)` 用于给底层数据库错误补充业务上下文，上层仍可继续识别唯一键冲突、连接错误或字段约束错误。


***
<br/><br/><br/>
> <h3 id="图片DataURL解析">图片 DataURL 解析</h3>

这段代码用于解析前端传来的 Base64 图片 DataURL，格式通常是：

```text
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...
```

核心逻辑：**先用逗号把元信息和 Base64 原文分开，再根据 MIME 类型推断文件后缀**。

```go
commaIdx := strings.Index(dataURL, ",")
if commaIdx < 0 || !strings.HasPrefix(dataURL, "data:") {
    return "", errors.New("无效的图片 data URL")
}

meta := dataURL[:commaIdx]
raw := dataURL[commaIdx+1:]

ext := "jpg"
if strings.Contains(meta, "image/png") {
    ext = "png"
} else if strings.Contains(meta, "image/webp") {
    ext = "webp"
}
```

拆分示例：

| 原始部分 | 解析结果 |
|----------|----------|
| `data:image/png;base64,xxxxxx` | `meta = data:image/png;base64` |
| 逗号后内容 | `raw = xxxxxx` |
| `meta` 包含 `image/png` | `ext = "png"` |

`strings.Index(dataURL, ",")` 找不到逗号时返回 `-1`；不以 `data:` 开头说明不是标准 DataURL，直接返回错误。默认后缀是 `jpg`，因此 `image/jpeg` 不需要单独判断；`image/png` 和 `image/webp` 会覆盖默认值。

---
<br/>

## 后续保存文件

通常解析出 `raw` 和 `ext` 后，会继续 Base64 解码并写入本地文件：

```go
decoded, err := base64.StdEncoding.DecodeString(raw)
if err != nil {
    return "", err
}

filename := fmt.Sprintf("%s.%s", uuid.NewString(), ext)
os.WriteFile("./upload/"+filename, decoded, 0644)
```

常见错误：前端传普通 `https://xxx.jpg` 链接会因为没有 `data:` 前缀报错；残缺 Base64 缺少逗号会导致 `commaIdx = -1`；如果传 `data:text/plain`，当前逻辑会默认当成 `jpg`，业务严格时应额外校验 `meta` 必须包含图片 MIME 类型。


***
<br/><br/><br/>
> <h3 id="文件真实MIME检测">文件真实 MIME 检测</h3>

文件上传不能只相信后缀名，推荐同时校验**后缀对应的预期 MIME**和**二进制内容识别出的真实 MIME**，防止用户把非法文件改后缀伪装成图片上传。

```go
contentType := http.DetectContentType(data)
expected := getContentType(ext)

// 截取真实 MIME 前缀，去掉 ; charset=xxx 部分
realMime, _, _ := strings.Cut(contentType, ";")
if realMime != expected {
    return errors.New("文件后缀与实际文件类型不匹配，禁止上传")
}
```

`http.DetectContentType(data)` 是 `net/http` 标准库方法，会读取前 `512` 字节，根据文件魔数判断真实类型，而不是根据后缀猜测。

常见文件头：

| 文件类型 | 文件头特征 | MIME |
|----------|------------|------|
| JPG | `FFD8FF` | `image/jpeg` |
| PNG | `89504E47` | `image/png` |
| WebP | `RIFF` | `image/webp` |

返回值示例：

```text
image/jpeg; charset=utf-8
image/png; charset=utf-8
application/octet-stream
```

对比前要先用 `strings.Cut(contentType, ";")` 去掉 `; charset=xxx`，否则 `image/png; charset=utf-8` 和 `image/png` 会被误判为不相等。

---
<br/>

## 后缀映射 MIME

`getContentType(ext)` 是项目自定义工具函数，根据清洗后的小写无点后缀返回预期 MIME：

```go
func getContentType(ext string) string {
    switch ext {
    case "jpg", "jpeg":
        return "image/jpeg"
    case "png":
        return "image/png"
    case "webp":
        return "image/webp"
    default:
        return "application/octet-stream"
    }
}
```

`jpg` 和 `jpeg` 是同一种图片格式，对应同一个 MIME：

```go
const (
    ImageTypeJPG  = "jpg"
    ImageTypeJPEG = "jpeg"
    ImageTypePNG  = "png"
    ImageTypeWebP = "webp"
)

var mime string
if ext == ImageTypeJPG || ext == ImageTypeJPEG {
    mime = "image/jpeg"
} else if ext == ImageTypePNG {
    mime = "image/png"
}
```

完整业务流程：清洗传入后缀 `ext`，拿到预期 MIME；读取文件二进制，用 `http.DetectContentType` 识别真实 MIME；真实 MIME 和预期 MIME 不一致时拦截上传；最后对 `jpg/jpeg` 这类等价后缀做统一处理。


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

***
<br/><br/><br/>
> <h3 id="MacGo环境升级与重新配置">Mac Go 环境升级与重新配置</h3>

**核心问题**：Homebrew 升级 Go 后，旧配置中写死的版本化路径 `/opt/homebrew/Cellar/go/<版本>/libexec` 会失效，终端执行 `go version` 报：

```text
go: cannot find GOROOT directory: /opt/homebrew/Cellar/go/1.23.5/libexec
```

**解决思路**：不再手动写 `GOROOT` 与版本化 Cellar 路径，统一使用 Homebrew 稳定入口 `/opt/homebrew/bin/go`，升级后由 Homebrew 自动更新软链接。

**当前推荐配置**（写入 `~/.bash_profile`）：

```bash
# Homebrew manages GOROOT via /opt/homebrew/bin/go; do not pin it to a versioned Cellar path.
unset GOROOT

export GOPATH=$HOME/HGFiles/GitHub/GoProject
export GOBIN=$GOPATH/bin

case ":$PATH:" in
  *":/opt/homebrew/bin:"*) ;;
  *) export PATH="/opt/homebrew/bin:$PATH" ;;
esac

case ":$PATH:" in
  *":$GOBIN:"*) ;;
  *) export PATH="$GOBIN:$PATH" ;;
esac

export GO111MODULE=auto
export GOPROXY=https://proxy.golang.org,direct
```

`~/.zshrc` 中加载 `~/.bash_profile`：

```bash
if [ -f ~/.bash_profile ]; then
  source ~/.bash_profile
fi
```

**验证命令与结果**：

```bash
which go
go version
go env GOVERSION GOROOT GOPROXY
```

```text
/opt/homebrew/bin/go
go version go1.26.4 darwin/arm64
go1.26.4
/opt/homebrew/Cellar/go/1.26.4/libexec
https://proxy.golang.org,direct
```

> 这里 `go env GOROOT` 显示版本化路径是正常的——这是 Go 自己根据当前安装位置算出来的，不是 shell 里写死的，升级后无需改动配置。

---

## 为什么不要写死 GOROOT

错误做法：

```bash
export GOROOT=/opt/homebrew/Cellar/go/1.23.5/libexec
```

路径中带具体版本号，升级后旧目录会被清理，配置立即失效。正确做法是在 shell 里 `unset GOROOT`，让 `go` 命令通过 `/opt/homebrew/bin/go` 自动识别。Homebrew 会自动维护软链接：

```text
/opt/homebrew/bin/go -> ../Cellar/go/1.26.4/bin/go
```

升级到新版本后，Homebrew 会自动把软链接指向新版本目录，shell 配置不需要再改。

---

## 从零配置 Go 环境步骤

**第一步：确认架构**

```bash
uname -m
```

输出 `arm64` 表示 Apple Silicon（M1/M2/M3/M4）；输出 `x86_64` 表示 Intel Mac。当前设备为 `darwin/arm64`。

**第二步：确认 Homebrew**

```bash
which brew
brew --version
```

正常输出类似 `/opt/homebrew/bin/brew`（Apple Silicon）或 `/usr/local/bin/brew`（Intel）。未安装则到 https://brew.sh/ 安装。

**第三步：让 Homebrew 进入 PATH**

在 `~/.zprofile` 中保留：

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

没有则追加：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
source ~/.zprofile
```

**第四步：安装 / 升级 Go**

```bash
brew install go
brew update && brew upgrade go
```

代理链路不稳定时降低并发：

```bash
HOMEBREW_DOWNLOAD_CONCURRENCY=1 brew upgrade go
```

**第五步：不要写死 GOROOT**

shell 配置中显式 `unset GOROOT`，让 Homebrew 软链接生效。

**第六步：配置 GOPATH / GOBIN**

```bash
export GOPATH=$HOME/HGFiles/GitHub/GoProject
export GOBIN=$GOPATH/bin
```

`go install` 输出的可执行文件会落在 `$GOBIN`。

**第七步：配置 PATH**

保证 `/opt/homebrew/bin` 和 `$GOBIN` 在 `PATH` 中，且 `/opt/homebrew/bin` 排在旧 Go 路径之前。

**第八步：配置 Go 模块代理（官方源）**

```bash
unset GOPROXY
go env -w GOPROXY=https://proxy.golang.org,direct
export GOPROXY=https://proxy.golang.org,direct
```

`go env -w` 写入持久配置；`export` 写入当前 shell；二者都要做。

**第九步：刷新终端**

```bash
source ~/.zshrc
hash -r
```

或重新打开终端窗口。

**第十步：验证**

```bash
which go
go version
go env GOVERSION GOROOT GOPATH GOBIN GOPROXY
```

**第十一步：确认 Homebrew 软链接自动升级机制**

```bash
ls -l /opt/homebrew/bin/go
ls -l /opt/homebrew/opt/go
```

输出软链接指向当前 Cellar 版本。后续 `brew upgrade go` 后，软链接会自动迁移到新版本。

---

## 常见问题

**问题 1：cannot find GOROOT directory**

旧 `GOROOT` 仍在环境变量里：

```bash
unset GOROOT
source ~/.zshrc
hash -r
go version
```

仍报错则搜索所有 shell 配置：

```bash
grep -n "GOROOT\|Cellar/go" ~/.bash_profile ~/.zshrc ~/.zprofile ~/.profile
```

删除写死的 `export GOROOT=/opt/homebrew/Cellar/go/<版本>/libexec`。

**问题 2：which go 不是 /opt/homebrew/bin/go**

PATH 顺序不对：

```bash
source ~/.zprofile
source ~/.zshrc
hash -r
echo $PATH
```

确保 `/opt/homebrew/bin` 出现在旧 Go 路径之前。

**问题 3：brew upgrade go 下载失败**

常见为 `curl: (35) LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to ghcr.io:443`，多因代理未继承、节点到 GitHub Container Registry 不稳、并发过高。处理：

```bash
env | grep -i proxy
curl -Iv https://ghcr.io/v2/
HOMEBREW_DOWNLOAD_CONCURRENCY=1 brew upgrade go
```

**问题 4：还在用国内镜像**

```bash
go env GOPROXY
# 输出 https://goproxy.cn,direct 表示还在用国内镜像

unset GOPROXY
go env -w GOPROXY=https://proxy.golang.org,direct
export GOPROXY=https://proxy.golang.org,direct
source ~/.zshrc
```

---

## 日常维护命令速查

```bash
go version
go env
go env GOVERSION GOROOT GOPATH GOBIN GOPROXY GOTOOLCHAIN

brew update
brew upgrade go
HOMEBREW_DOWNLOAD_CONCURRENCY=1 brew upgrade go

ls -l /opt/homebrew/bin/go
ls -l /opt/homebrew/opt/go

grep -n "GOROOT\|Cellar/go" ~/.bash_profile ~/.zshrc ~/.zprofile ~/.profile
```

---

## 最终判断标准

- `which go` 输出 `/opt/homebrew/bin/go`。
- `go version` 输出当前 Homebrew 安装的 Go 版本。
- shell 配置中没有 `export GOROOT=/opt/homebrew/Cellar/go/<版本>/libexec`。
- `go env GOPROXY` 输出 `https://proxy.golang.org,direct`。
- 后续 `brew upgrade go` 后，shell 配置无需再手动改版本号。

***
<br/><br/><br/>
> <h1 id="任务生命周期控制">任务生命周期控制</h1>

```go
if s.syncer != nil {
    s.syncer.Start(context.WithoutCancel(ctx))
}
```

**整体语义**：如果 syncer 存在就启动它，并传入一个**不会继承父 ctx 取消信号**的 context——保留 trace / value 信息，但切断生命周期继承。

---

## 逐行解释

**`if s.syncer != nil`**：防御式写法，避免 nil panic；syncer 是可选组件。

**`s.syncer.Start(...)`**：启动后台任务组件，常见场景：

- 数据同步（DB → cache）
- 消息消费（Kafka consumer）
- 定时 flush
- 状态同步（IoT / WebRTC / 视频系统）

**`context.WithoutCancel(ctx)`**：Go 1.21+ 提供的 context 包装，**创建一个不继承父 ctx cancel 信号的新 ctx**：

```text
原 ctx：可能被 cancel（HTTP 请求结束、服务 shutdown）
新 ctx：父 ctx cancel 时不会自动 cancel
```

---

## 为什么需要 WithoutCancel

假设直接传父 ctx：

```go
ctx := request.Context()
s.syncer.Start(ctx)
```

当 HTTP 请求结束、用户断开连接或上游 `cancel()` 时，syncer 会被强制停止。但 syncer 通常是后台长期任务，**不应该随请求结束而终止**：

| 组件 | 是否应随 request ctx 结束 |
|------|--------------------------|
| HTTP handler | 是 |
| syncer 同步器 | 否 |
| Kafka consumer | 否 |
| heartbeat | 否 |

`context.WithoutCancel` 的作用就是**切断生命周期继承**，但**保留 trace / value 信息**——既能继续打链路日志，又不被父 ctx 误杀。

---

## 与 context.Background 的区别

| 方式 | cancel 是否继承 | trace / value 是否保留 |
|------|----------------|------------------------|
| `ctx` | 是 | 是 |
| `context.Background()` | 否 | 否 |
| `context.WithoutCancel(ctx)` | 否 | 是 |

定位：**保留上下文信息，断开生命周期控制**。

---

## 工程意义

**1. request → background worker 解耦**

```text
HTTP request ctx
   ↓
Start syncer（后台任务）
   ↓
request 结束，syncer 继续跑
```

**2. 避免误杀后台任务**：不做隔离时，请求关闭 = syncer 停止 → 数据同步中断 / 状态不一致。

**3. 保留 tracing**：context 常带 `trace_id` / `span_id` / `user_id`，WithoutCancel 后这些信息仍可在 goroutine 链路日志中传递。

---

## 潜在风险

**风险 1：goroutine 泄漏**。syncer 不主动退出 / 无 stop signal 时，会变成永远运行的 goroutine。

**风险 2：无法响应 shutdown**。滥用 WithoutCancel 会导致 `app shutdown -> syncer 不停`。

**风险 3：生命周期不清晰**。容易出现"谁创建谁不负责释放"。

---

## 推荐工程模式

大厂一般不会只靠 ctx，而是采用**双控制模型**：

```go
Start(ctx context.Context)
Stop()
```

或显式 stop channel：

```go
Start(ctx context.Context, stopCh <-chan struct{})
```

或显式 cancel，由 service 统一管理：

```go
ctx, cancel := context.WithCancel(parent)
```

---

## 一句话总结

> 在 Go 中启动一个后台 syncer，通过 `context.WithoutCancel` 将其从请求生命周期中解耦，但仍保留 trace / value 信息——是 Go 1.21+ 在"解耦后台任务 + 保留链路信息"场景下的推荐写法。

<br/><br/><br/>

***
<br/><br/><br/>
> <h3 id="go-redis渐进式遍历Scan">go-redis 渐进式遍历 Scan</h3>

`go-redis` 的 `client.Scan(...)` 是 Redis `SCAN` 命令的 Go 封装，**通过游标 + 渐进扫描遍历 Key，避免 `KEYS *` 阻塞 Redis 主线程**，是亿级 Key 数据下唯一推荐的遍历方式。

```go
func (c cmdable) Scan(
    ctx context.Context,
    cursor uint64,
    match string,
    count int64,
) *ScanCmd
```

> 常见误解：`Scan = 遍历 Redis 所有 Key`。其实它对应的是 `SCAN`，设计目标是**不阻塞 Redis 的情况下渐进式遍历**。

---

## 为什么需要 Scan

`KEYS *` 会扫描全部 Key 并一次性返回，Redis 单线程一直工作，其它命令全部等待——生产环境几乎禁止。`SCAN` 不阻塞、每次扫描一点、可暂停可继续，是大厂标准方案。

---

## cursor 是什么

`cursor` **不是"第几页"**，而是 Redis 内部遍历 Hash Table 的游标位置。每次 `SCAN <cursor>` 由 Redis 自身算出下一次游标，`cursor == 0` 表示扫描结束。

```text
SCAN 0   → cursor: 18, keys: A C
SCAN 18  → cursor: 95, keys: D E
SCAN 95  → cursor: 0   → 结束
```

---

## match / count 参数

- `match`：模式过滤（`user:*` 只返回匹配 Key），但 Redis 仍需遍历整个哈希表，仅在返回时过滤，不能理解为索引查询。
- `count`：**Hint（建议值）**，不保证返回固定数量。`count=100` 可能返回 `78`、`132` 甚至 `3`，**千万不要用 `len(keys) == count` 判断结束**。

---

## 返回值与标准写法

`Scan()` 返回的是 `*ScanCmd` 命令对象，真正的数据通过 `.Result()` 取出。结束条件：**`nextCursor == 0`**。

```go
var cursor uint64
for {
    keys, nextCursor, err := client.Scan(
        ctx, cursor, "user:*", 100,
    ).Result()
    if err != nil {
        return err
    }
    for _, key := range keys {
        // 处理
    }
    cursor = nextCursor
    if cursor == 0 {
        break
    }
}
```

---

## 为什么不会阻塞 Redis

每次只扫描一小部分 Hash Bucket，执行时间通常几十微秒到几毫秒，其它客户端的 `GET/SET/DEL/INCR` 几乎不受影响：

```text
Bucket1 → 返回
Bucket2 → 返回
Bucket3 → 返回
```

---

## Scan 的关键特性

1. **可能返回重复 Key**：扫描期间可能发生 rehash 或 Key 被修改，业务要求每个 Key 只处理一次时需客户端去重。
2. **不保证遍历期间数据一致性**：新增 Key 可能扫到也可能扫不到，已删除 Key 同理。`SCAN` 的目标是**最终遍历**，不是一致性快照。
3. **大厂用法**：`SCAN` 用于**后台运维、缓存清理、数据迁移、灰度删除**，**不会用于在线业务查询**。

去重示例：

```go
seen := make(map[string]struct{})
for _, key := range keys {
    if _, ok := seen[key]; ok {
        continue
    }
    seen[key] = struct{}{}
    // process
}
```

---

## 一句话总结

> `Scan()` 是 Redis `SCAN` 命令的 Go 封装；`cursor` 是游标不是页码；`count` 是 Hint 不保证数量；结束标志是 `cursor == 0`；结果可能重复，不是一致性快照；只能用于后台任务，不能用于在线高 QPS 业务。

***
<br/><br/><br/>
> <h2 id="在线高QPS业务不要依赖RedisScan">在线高 QPS 业务不要依赖 Redis Scan</h2>

**核心原则**：Redis 不是 MySQL，**不应该靠"搜索"找数据**，而应通过 Key 设计 + 数据结构设计让数据可以 `O(1)` 或 `O(logN)` 获取。

```text
错误：我有什么数据？→ SCAN 找
正确：我需要什么数据？→ 提前维护索引 Key → 直接 GET/ZSET/HGET
```

---

## 错误方案：线上请求 Scan

需求是"获取某用户最近上传的视频"，但 Redis 只存 `video:10001`、`video:10002`...，然后：

```go
keys, _, _ := redis.Scan(ctx, 0, "video:*", 100)
for _, k := range keys {
    if video.UserID == uid { /* ... */ }
}
```

假设 Redis 有 10 亿 video Key，目标用户 `uid=888` 只有 20 个视频，但需要扫描 10 亿，复杂度 `O(N)`，并发一高 Redis CPU 直接爆炸。

---

## 正确方案：业务索引

设计思路：**数据实体 + 索引结构**，类似 MySQL `video` 表 + `index(user_id)`，Redis 自己维护索引。

### 案例1：用户视频列表

```text
视频详情：
  Key：   video:{video_id}
  Value：Hash { id, user_id, title, status, created_at }

用户视频索引（Sorted Set）：
  Key：   user:{user_id}:videos
  Score： 发布时间
  Member：video_id
```

查询流程 `GET /users/888/videos`：

```redis
ZREVRANGE user:888:videos 0 19    # 取最新 20 个 video_id
MGET video:10001 video:10002 ...  # 批量取详情
```

复杂度 `O(logN + M)`，没有搜索、没有遍历。

### 案例2：预约发布任务

```text
Key：   video:scheduled:queue
Score： 发布时间
Member：submission:10001
```

Worker 每秒执行：

```redis
ZRANGEBYSCORE video:scheduled:queue 0 <当前时间> LIMIT 0 100
```

发布成功后 `ZREM video:scheduled:queue submission:10001`，避免重复消费。

### 案例3：用户在线状态

```text
Key： online:users（Set）
上线：SADD online:users 10001
下线：SREM online:users 10001
查询：SCARD online:users
```

### 案例4：点赞数量

```text
Key：   video:{id}:likes（String）
增加：  INCR video:10001:likes
读取：  GET video:10001:likes
```

复杂度 `O(1)`。

### 案例5：排行榜

```text
Key：   video:hot（ZSET，score=热度）
查询：  ZREVRANGE video:hot 0 99   # Top100
```

---

## Go 代码示例

添加视频（Pipeline 一次写实体 + 索引）：

```go
func AddUserVideo(
    ctx context.Context,
    uid int64, videoID int64, publishTime int64,
) error {
    pipe := redis.TxPipeline(ctx)
    pipe.HSet(ctx, fmt.Sprintf("video:%d", videoID), map[string]interface{}{
        "user_id": uid, "status": "published",
    })
    pipe.ZAdd(ctx, fmt.Sprintf("user:%d:videos", uid), redis.Z{
        Score:  float64(publishTime),
        Member: videoID,
    })
    _, err := pipe.Exec(ctx)
    return err
}
```

查询：

```go
func GetUserVideos(ctx context.Context, uid int64) {
    ids, _ := redis.ZRevRange(
        ctx, fmt.Sprintf("user:%d:videos", uid), 0, 19,
    )
    // pipeline MGET
}
```

---

## Scan 使用场景速查

| 场景 | Scan |
|------|------|
| 线上接口查询 | ❌ |
| 用户列表/视频列表查询 | ❌ |
| 排行榜 | ❌ |
| 定时清理缓存 | ✅ |
| 迁移 Redis 数据 | ✅ |
| 统计 Key | ✅ |
| 后台运维 | ✅ |

推荐数据建模：

```text
MySQL  → 数据真相
Redis  → Entity Cache
         + List Index
         + Rank Index
         + Delay Queue
```

***
<br/><br/><br/>
> <h2 id="Scan通过-Result获取结果">Scan 通过 .Result 获取结果</h2>

`go-redis` API 风格高度统一：**`Scan()` 返回 `*ScanCmd` 命令对象，真正数据通过 `.Result()` 取出**。`Result()` 只是把 `cmd` 内部的 `page/cursor/err` 返回出来，**不会再访问 Redis**。

```go
type ScanCmd struct {
    baseCmd
    page   []string
    cursor uint64
}
```

```go
func (cmd *ScanCmd) Result() ([]string, uint64, error) {
    return cmd.page, cmd.cursor, cmd.err
}
```

---

## 调用流程

```text
client.Scan()
    → 创建 ScanCmd
    → 发送 SCAN 0 MATCH user:* COUNT 100
    → Redis 返回 cursor + keys
    → 解析 RESP 写入 ScanCmd
    → 返回 ScanCmd
Result()
    → 返回 keys, cursor, err（不再访问 Redis）
```

---

## 为什么不直接返回三值

为了和 go-redis 整体 API 保持一致——`Get → *StringCmd`、`Set → *StatusCmd`、`Incr → *IntCmd`、`HGetAll → *MapStringStringCmd`，全部都是"先取命令对象，再 `.Result()`"的模式。

**`ScanCmd` 没有 `Val()`** 是因为它有两个主要返回值（`keys` + `cursor`），无法用单个 `Val()` 表示。

---

## 其它取值方法

```go
cmd := client.Get(ctx, "name")
if err := cmd.Err(); err != nil { /* 只关心错误 */ }
name, err := cmd.Result()

cmd := client.Incr(ctx, "count")
n := cmd.Val()  // IntCmd 提供 Val()，直接拿值
```

---

## 工程意义

- **统一 API**：所有命令都返回对象 + `.Result()`，调用方式一致。
- **可扩展**：以后增加耗时、原始响应、重试次数等字段，无需修改函数签名。
- **便于 Pipeline / 事务**：先收集命令对象，统一 `Exec` 后再分别 `.Result()`。

Pipeline 延迟执行示例：

```go
pipe := rdb.Pipeline()
getCmd  := pipe.Get(ctx, "user:1")
scanCmd := pipe.Scan(ctx, 0, "user:*", 100)
_, err := pipe.Exec(ctx)        // 统一发送
name, err  := getCmd.Result()   // 分别取值
keys, cur, err := scanCmd.Result()
```

如果 `Scan()` 一开始就返回 `([]string, uint64, error)`，Pipeline 的延迟执行模式无法实现。

***
<br/><br/><br/>
> <h2 id="RedisZSET与ZREM">Redis ZSET 与 ZREM</h2>

`ZSET`（Sorted Set）= 有序集合，每个 member 带一个 score 用于排序；`ZREM` = 删除 ZSET 中的指定 member。是排行榜、延迟队列、Feed 流的核心结构。

```redis
ZADD key score member
ZREM key member
```

---

## 与 Set 的区别

| 类型 | 特点 | 典型用途 |
|------|------|----------|
| Set | 无序，去重 | 标签、去重、在线状态 |
| ZSET | 按 score 排序，去重 | 排行榜、延迟队列、时间排序 |

```redis
SADD users 1001 1002 1003            # Set，无顺序
ZADD users 100 1001 90 1002 80 1003  # ZSET，按 score 排序
```

score 可以是任意 double，member 是去重的。

---

## 核心概念

- **member**：排序的对象，如 `video:10001`、`submission:10001`。
- **score**：排序依据，如发布时间 `1782907200`、热度 `1000`。

```redis
ZADD video:hot 1000 video:10001
```

---

## 视频系统典型应用

**热门视频排行**：

```redis
ZADD video:hot 1200 video:3 999 video:1 800 video:2
ZREVRANGE video:hot 0 99   # Top100
```

**预约发布**（按时间排序的任务队列）：

```redis
ZADD video:scheduled 1783684800 submission:10001
```

Worker 每秒取到期任务：

```redis
ZRANGEBYSCORE video:scheduled 0 <now> LIMIT 0 100
```

发布成功后**必须 `ZREM`**，否则下一秒会再次被消费，导致重复发布 / 重复发通知 / 重复写库。

完整流程：

```text
Redis ZSET → 到期任务 → 发布服务 → MySQL 更新 status=published → ZREM 删除任务
```

---

## Go 用法

```go
// 添加任务
err := rdb.ZAdd(ctx, "video:scheduled", redis.Z{
    Score:  float64(publishTime.Unix()),
    Member: submissionID,
}).Err()

// 取到期任务
tasks, err := rdb.ZRangeByScore(ctx, "video:scheduled", &redis.ZRangeBy{
    Min:   "0",
    Max:   strconv.FormatInt(time.Now().Unix(), 10),
    Count: 100,
}).Result()

// 删除任务
rdb.ZRem(ctx, "video:scheduled", submissionID)
```

---

## 底层为什么快

```text
Hash Table   → member -> score，O(1) 查找
SkipList     → 按 score 排序，O(logN) 范围查询
```

| 操作 | 复杂度 |
|------|--------|
| `ZADD` | `O(logN)` |
| `ZREM` | `O(logN)` |
| 范围查询 | `O(logN + M)` |

---

## 视频系统 Redis 建模推荐

```text
video:{id}            Hash     视频详情
user:{uid}:videos     ZSET     用户视频列表（score=create_time）
video:hot             ZSET     热门视频（score=hot_score）
video:scheduled       ZSET     预约发布（score=publish_timestamp）
video:{id}:likes      String   点赞数
```

**ZSET 是 Redis 的"排序索引"**，ZREM 是删除排序索引中的元素；大厂延迟任务系统最常见的 Redis 建模方式。

***
<br/><br/><br/>
> <h1 id="视频流式读取ioLimitReaderReadAll">视频流式读取 io.LimitReader + ReadAll</h1>

```go
limited := io.LimitReader(part, maxMultipartHeaderBytes+1)
data, err := io.ReadAll(limited)
```

这是 Go 标准库中**受限流读取（Limited Stream Read）**的经典模式，目的不是提速，而是**限制内存占用、防止恶意输入导致 OOM，并利用 `+1` 字节检测数据是否超过允许上限**。

---

## 整体执行流程

`part`（`multipart.Part`）本质是 `io.Reader`——不是整个文件，而是**一个可以不断 `Read()` 的数据流**：

```text
网络 → TCP → HTTP → multipart → part → Read()
```

`io.LimitReader(part, n)` 返回的 `limited` 本身**不读任何数据**，只是包装了一层 `*LimitedReader{R: part, N: n}`，在每次 `Read()` 时扣减 `N`，达到上限返回 `EOF`。

```go
type LimitedReader struct {
    R Reader
    N int64
}
```

`Read()` 行为（简化）：

```go
func (l *LimitedReader) Read(p []byte) (int, error) {
    if l.N <= 0 {
        return 0, EOF
    }
    if len(p) > l.N {
        p = p[:l.N]
    }
    n, err := l.R.Read(p)
    l.N -= int64(n)
    return n, err
}
```

---

## 为什么是 `maxMultipartHeaderBytes + 1`

这是 Go 标准库非常经典的技巧。常见后续判断是：

```go
if len(data) > maxMultipartHeaderBytes {
    return ErrTooLarge
}
```

- 用户上传 `8191` 字节 → `ReadAll` 读 `8191`，不超限；
- 用户上传 `8192` 字节 → 读 `8192`，不超限；
- 用户上传 `9000` 字节 → 读到 `8193`（`LimitReader` 多读 1 字节），`len(data) > 8192` 成立，立即知道"原始数据至少超过限制"。

**`+1` 不是为了多读一个字节，而是为了准确判断是否超限**。

---

## 为什么不能直接 `io.ReadAll(part)`

攻击者上传 5GB Header 时 `ReadAll` 会一直 `malloc`，最终 OOM，服务器挂。生产代码几乎都是 `Reader → LimitReader → ReadAll` 模式。

---

## 执行示例

`part` 内容 `ABCDE12345`，`Limit=6`：

```text
ReadAll 第1次：ABC     → N 剩 3
ReadAll 第2次：DE1     → N 剩 0
ReadAll 第3次：EOF     → 结束
结果：ABCDE1
```

---

## 总结

这两行代码采用**受限流读取**模式，限制内存占用、防止恶意输入 OOM，并通过 `+1` 字节检测是否超限，是生产级 Go 服务处理上传数据时最常见、最推荐的写法。

***
<br/><br/><br/>
> <h2 id="千万级视频业务标准架构">千万级视频业务标准架构</h2>

视频、图片、GB 级文件**不能 `io.ReadAll`**——100 个用户同时上传 2GB 文件就要求 200GB 内存，服务器直接挂。**核心思想：文件不要进入内存，让它像水流一样从输入流直接写入目标**。

```text
HTTP Request
   ↓
Go Memory   ← 错误：[2GB byte slice] 内存暴涨 / GC 压力 / OOM
```

**正确方案**：

```text
客户端
  | multipart/form-data
  v
Go HTTP Server
  | io.Reader
  v
对象存储（S3/OSS/COS）

内存只保存几十 KB buffer，不保存整个文件
```

---

## 方案1：io.Copy 流式上传（基础版）

```go
func UploadVideo(w http.ResponseWriter, r *http.Request) {
    file, header, err := r.FormFile("file")
    if err != nil { return }
    defer file.Close()

    dst, err := os.Create("/data/video.mp4")
    if err != nil { return }
    defer dst.Close()

    written, err := io.Copy(dst, file)
    fmt.Println("uploaded bytes:", written)
}
```

执行过程：

```text
file.Reader
  → 读取 32KB → buffer → disk.Write
  → 读取 32KB → buffer → disk.Write
  → ...
```

必须加大小限制，防止 100GB 攻击：

```go
const MaxVideoSize = 5 << 30  // 5GB
reader := io.LimitReader(file, MaxVideoSize+1)
written, err := io.Copy(dst, reader)
if written > MaxVideoSize {
    return errors.New("file too large")
}
```

---

## 方案2：预签名上传（大厂标准）

Go 服务**不中转文件**，只负责生成上传凭证，客户端直传对象存储：

```go
func CreateUploadURL(ctx context.Context, userID int64) (string, error) {
    key := fmt.Sprintf("videos/%d/%s.mp4", userID, uuid.New())
    url, err := s3Client.PresignPutObject(ctx, "video-bucket", key, time.Hour)
    return url, err
}
```

完整链路：

```text
1. App ──请求上传──> Go API
2. Go  生成 upload token
3. App <──返回上传地址──
4. App ──直接上传──> S3
5. S3 ──回调──> Go
```

---

## 方案3：分片上传（GB 级视频）

10GB 视频拆 100 个 100MB chunk 分别上传：

```text
10GB → 100MB × 100 chunk
chunk1, chunk2, ..., chunk100
```

分片表设计：

| upload_id | part | status |
|-----------|------|--------|
| abc | 1 | done |
| abc | 2 | done |
| abc | 3 | uploading |

`part` 本身是 `io.Reader`，直接 `io.Copy()` 即可。

接口设计：

```text
POST /video/upload/init     → 返回 upload_id
PUT  /video/upload/part     → 上传单个分片
POST /video/upload/complete → CompleteMultipartUpload
```

---

## 表拆分设计

```sql
video_upload_tasks     -- 上传任务
  id, user_id, upload_id, file_size, status, created_at

video_upload_parts     -- 分片
  upload_id, part_number, size, etag, status

videos                 -- 视频
  video_id, storage_key, duration, size, status
```

---

## 完整生产链路

```text
用户
  ↓
Go API（控制面：鉴权、生成凭证、状态管理）
  ↓
客户端（数据面：分片上传）
  ↓
OSS/S3/COS
  ↓
Kafka 异步
  ↓
视频处理服务（转码 → 审核 → 发布）
```

---

## 方案选型速查

| 场景 | 方案 |
|------|------|
| 头像、小图片 | `io.Copy` |
| 10MB 以内文件 | Go 代理上传 |
| 100MB ~ 5GB 视频 | 对象存储直传 |
| GB 级视频 | Multipart Upload |
| 千万用户视频平台 | 预签名 URL + 分片上传 + Kafka |

`io.Copy(dst, io.LimitReader(part, maxFileSize))` 属于**服务端接收流式上传的基础方案**；对标字节、阿里视频系统应升级为：**Go 只负责上传控制面，数据面由客户端直接进入 OSS/S3/COS，采用分片上传 + Kafka 异步处理**。

***
<br/><br/><br/>
> <h1 id="INSERTONDUPLICATEKEYUPDATE">INSERT ... ON DUPLICATE KEY UPDATE</h1>

MySQL **UPSERT 语法**：数据不存在则插入，存在则更新（依赖唯一键冲突）。在预约发布、用户配置、点赞、收藏、任务状态等场景中大量使用。

```sql
INSERT INTO video_scheduled_publish (
    submission_id,
    user_id,
    scheduled_time,
    status
)
VALUES (?, ?, ?, 'pending')
ON DUPLICATE KEY UPDATE
    scheduled_time = VALUES(scheduled_time),
    status = 'pending',
    updated_at = CURRENT_TIMESTAMP;
```

**必须依赖唯一键或主键**：

```sql
UNIQUE KEY uk_submission(submission_id)
```

---

## INSERT 部分

无冲突时正常插入：

```text
submission_id = 1001
user_id       = 88
scheduled_time = 2026-07-05 20:00:00
status        = pending
```

---

## ON DUPLICATE KEY UPDATE

INSERT 触发唯一键冲突时**不报错**，转而执行 UPDATE：

```text
原来：1001 / 20:00
新值：1001 / 21:00
结果：1001 / 21:00（更新成功）
```

### `VALUES(column)` 含义

代表"INSERT 这一行准备插入的值"，**不是数据库里的值**。例如：

```sql
VALUES(scheduled_time) → '2026-07-05 21:00'
```

> ⚠️ MySQL 8.0.20 之后 `VALUES(column)` 被标记为 deprecated，新项目推荐别名写法：
>
> ```sql
> INSERT INTO table (...) VALUES (...) AS new
> ON DUPLICATE KEY UPDATE scheduled_time = new.scheduled_time;
> ```
>
> 目前很多项目仍在用 `VALUES()`，新项目建议关注目标 MySQL 版本。

### `status = 'pending'` 与 `updated_at`

- `status = 'pending'`：无论之前是什么状态，都强制进入等待发布状态。
- `updated_at = CURRENT_TIMESTAMP`：更新最后修改时间，便于审计、排查、CDC、缓存刷新。

---

## 整体流程

```text
第一次：submission_id=1001 → 数据库没有 → INSERT 成功
       → 1001 / pending / 20:00

第二次：submission_id=1001 → 已存在
       → INSERT 触发唯一键冲突
       → ON DUPLICATE KEY UPDATE
       → 1001 / pending / 21:00
```

---

## 为什么不用先 SELECT 再 UPDATE

`SELECT + INSERT/UPDATE` 两次 SQL，且存在**并发竞争（Race Condition）**——A、B 同一时刻都 `SELECT` 到不存在，都 `INSERT`，B 触发 `Duplicate Key`。

`INSERT ... ON DUPLICATE KEY UPDATE` 由数据库**原子执行**，一次 SQL 完成，无竞争问题。

---

## 大厂为什么喜欢

- **原子性**：插入或更新由数据库一次完成，避免并发数据不一致。
- **减少数据库往返**：无需 `SELECT` 决定 `INSERT/UPDATE`。
- **代码简洁**：业务层无需处理重复键异常和重试逻辑。
- **适合高并发**：数据库唯一索引保证一致性，比业务层判断更可靠。

亿级 / 千万级并发场景下通常会结合**分库分表、消息队列（Kafka）、批量写入**和合理唯一键设计，降低热点竞争和索引维护成本。

***
<br/><br/><br/>
> <h1 id="客户端时间统一解析为UTC">客户端时间统一解析为 UTC</h1>

核心作用：**把客户端不同格式的时间统一解析成 `time.Time`，并按规范处理时区**。本质是一个**多协议时间解析器（time parser / normalizer）**。

```go
type ClientTime struct {
    Format   string
    Value    string
    Timezone string
}

func ParseClientTime(ct ClientTime) (time.Time, error)
```

支持 3 种格式：

| Format | 含义 | 示例 |
|--------|------|------|
| `rfc3339` | 标准时间字符串 | `2026-01-01T10:00:00Z` |
| `datetime-local` | 本地时间（无时区） | `2026-01-01T10:00` |
| `unix` | 时间戳 | `1700000000` |

---

## RFC3339 格式

```go
case "rfc3339":
    return time.Parse(time.RFC3339, ct.Value)
```

输入 `2026-01-01T10:00:00Z` / `2026-01-01T10:00:00+08:00` 都自带时区，Go 标准库直接解析。

---

## datetime-local（无时区）

HTML `<input type="datetime-local">` 常见格式 `2026-01-01T10:00`，**没有时区信息**——无法知道是北京时间、UTC 还是东京时间。

**强制时区**：

```go
case "datetime-local":
    if ct.Timezone == "" {
        return time.Time{}, errors.New("timezone required for datetime-local format")
    }
    loc, err := time.LoadLocation(ct.Timezone)
    if err != nil {
        return time.Time{}, fmt.Errorf("invalid timezone %q", ct.Timezone)
    }
    return time.ParseInLocation("2006-01-02T15:04", ct.Value, loc)
```

**`ParseInLocation` 意义**：用指定时区解析一个无时区的时间字符串，例如：

```go
ct.Value = "2026-01-01T10:00"
ct.Timezone = "Asia/Shanghai"
// → 2026-01-01 10:00:00 +0800 CST
// → 2026-01-01 02:00:00 UTC
```

> ⚠️ 不能用 `time.Parse()`，它默认按 UTC 或系统规则处理，跨时区场景会出错。

---

## Unix 时间戳

```go
case "unix":
    ts, err := strconv.ParseInt(ct.Value, 10, 64)
    if err != nil {
        return time.Time{}, err
    }
    return time.Unix(ts, 0).UTC(), nil
```

`1700000000 → 2023-11-14 02:13:20 UTC`。**只支持秒级时间戳**，毫秒（13 位）会被解析成错误时间。

---

## default 兜底

```go
default:
    return time.Time{}, fmt.Errorf("unsupported time format: %q", ct.Format)
```

防止未知格式、拼写错误、新协议未适配。

---

## 整体流程

```text
ClientTime
   ↓
Format 判断
   ├─ rfc3339        → time.Parse
   ├─ datetime-local → ParseInLocation + timezone
   └─ unix           → time.Unix + UTC
   ↓
time.Time（标准化）
```

---

## 工程价值

1. **统一时间入口**：所有时间最终变成 `time.Time`，避免 string 混乱、多格式共存、前后端不一致。
2. **强制时区意识**：`datetime-local + timezone` 是防止"时间错位 bug"的关键防线。
3. **支持多协议输入**：Web（datetime-local）、API（RFC3339）、SDK/DB（unix）。
4. **防御式编程**：timezone 必填、unknown format 报错、unix parse 校验。

---

## 潜在优化点

- **支持毫秒时间戳**：`if len(ct.Value) > 10 { /* ms */ }`。
- **timezone cache**：`time.LoadLocation()` 有 IO + 文件读取，可用 `sync.Map` 缓存常用时区。
- **format enum 化**：用 `type TimeFormat int` 替代 string，避免拼写错误。

---

## 一句话总结

> 将客户端传入的多种时间表达方式（RFC3339 / 本地时间 / Unix 时间戳）统一解析为 `time.Time`，并通过强制时区处理避免跨区域时间错误——是 API / 跨端系统里非常常见的时间处理模型。

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
