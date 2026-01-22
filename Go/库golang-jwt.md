> # 库golnag-jwt
- [JWT作用介绍](#JWT作用介绍)
- [Token的用途](#Token的用途)
- [AccessToken和RefreshToken简单实现](#AccessToken和RefreshToken简单实现)


<br/><br/><br/>

***
<br/>

> <h1 id="JWT作用介绍">JWT作用介绍</h1>

| 问题 | 回答 |
|------|------|
| JWT 是干嘛的？ | 用于安全传递用户身份/声明，常用于认证和授权 |
| Token 是签名吗？ | 不是，是“带签名的数据”，签名用于防伪和防篡改 |
| 只要唯一就行？ | ❌ 不行！必须可验证、有时效、防伪造，唯一性只是辅助 |

<br/><br/>

&emsp; **JWT（JSON Web Token）** 是一种开放标准（[RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)），用于在各方之间**安全地传输信息**作为 JSON 对象。它通常用于：

- **身份认证（Authentication）**：用户登录后，服务器返回一个 JWT，客户端后续请求带上这个 token，服务器验证后就知道是谁在请求。
- **信息交换（Information Exchange）**：因为 JWT 可以被签名（比如用 HMAC 或 RSA），所以可以验证内容是否被篡改。

<br/>

```go
at, _ := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString(Secret)

rt, _ := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString(Secret)
```

- **在你的代码中：**
	- `at` 是 **Access Token（访问令牌）**，短期有效（比如 15 分钟），用于常规 API 请求的身份验证。
	- `rt` 是 **Refresh Token（刷新令牌）**，长期有效（比如 7 天），用于在 Access Token 过期后获取新的 Access Token，而无需用户重新登录。

<br/><br/><br/>

***
<br/>

> <h1 id="Token的用途">Token的用途</h1>
JWT **包含签名，但本身不只是签名**。一个典型的 JWT 由三部分组成（用 `.` 分隔）：

```
header.payload.signature
```

- **Header**：说明签名算法（如 HS256）和 token 类型。
- **Payload**：实际数据（称为 Claims），比如用户 ID、过期时间等。
- **Signature**：用密钥（你的 `Secret`）对前两部分进行签名，确保内容未被篡改。

> 所以：**Token ≠ 签名**，而是“带签名的数据包”。

**签名的作用**：
- 防止伪造：只有知道 `Secret` 的服务端才能生成合法 token。
- 防篡改：如果有人修改了 payload（比如把 `userID=123` 改成 `userID=456`），签名就会不匹配，验证失败。

---
<br/>

 **只要保留唯一是不是就可以了？**

**不是的！** 仅仅“唯一”远远不够。JWT 的核心价值在于：

| 要求 | 说明 |
|------|------|
| ✅ **可验证性** | 必须能通过签名验证 token 是否由可信方签发 |
| ✅ **时效性** | 通过 `ExpiresAt` 控制有效期，防止长期滥用 |
| ✅ **防重放** | 通过 `JTI`（JWT ID）可以防止 token 被重复使用（需配合服务端记录） |
| ❌ **仅唯一 ≠ 安全** | 如果没有签名，攻击者可以随便伪造一个“唯一”的 token |

<br/>

**举个反例：**
> 假设你只用 UUID 当 token（无签名、无过期），一旦泄露，攻击者永远可以用它冒充用户。而 JWT 有过期时间 + 签名，即使泄露，也只在短时间内有效，且无法伪造新 token。

---
<br/>

- **建议**
	- 1.**不要把敏感信息放 JWT payload**  
		- JWT 默认只是 Base64 编码（**不是加密**），任何人都能解码看到内容。所以不要放密码、身份证号等。
	- 2.**Refresh Token 要更严格保护**  
		- 因为它生命周期长，建议：
	   - 存储在 HttpOnly Cookie 中（防 XSS）
	   - 绑定 IP/User-Agent（可选）
	   - 支持主动吊销（需服务端存储黑名单或白名单）
	- 3.**Secret 要足够强且保密**  
		- 使用至少 32 字节的随机密钥（如 `crypto/rand` 生成），不要硬编码在代码里。


<br/>

**❓常见疑问**

- **Q：为什么要有两个 token？**
	- A：安全！Access Token 短期有效，即使泄露危害小；Refresh Token 长期有效，但只用于换 token，不用于普通请求，且可配合 HttpOnly Cookie 存储更安全。

- **Q：Secret 是什么？**
	- A：一个只有你服务器知道的密钥（比如 32 位随机字符串），用于签名和验证。**绝对不能泄露！**

- Q：JTI 有什么用？
	- A：每个 token 有唯一 ID。你可以把已使用的 JTI 存起来，防止别人重复使用同一个 token（重放攻击）。

<br/><br/><br/>

***
<br/>

> <h1 id="AccessToken和RefreshToken简单实现">AccessToken和RefreshToken简单实现</h1>

&emsp; JWT 是一种“自包含”的令牌（token），它把用户信息（比如用户ID）打包成一段字符串，并用密钥签名，防止别人伪造。服务端发给客户端后，客户端每次请求带上它，服务端验证签名合法 + 没过期 → 就信任这个用户。

<br/>

```go
type Claims struct {
    UserID string `json:"user_id"`
    JTI    string `json:"jti"` // JWT ID
    jwt.RegisteredClaims         // 嵌入标准字段（exp, iat, iss 等）
}
```

可以用 [https://jwt.io](https://jwt.io) 粘贴生成的 token，看到里面的内容（注意：**这只是编码，不是加密！**）

***
<br/>

**定义 Claims（要放进 token 的数据）**

```go
claims := Claims{
    UserID: userID,
    JTI:    jti,
    RegisteredClaims: jwt.RegisteredClaims{
        IssuedAt:  jwt.NewNumericDate(now),
        ExpiresAt: jwt.NewNumericDate(now.Add(AccessTTL)),
    },
}
```

- ✅ 解读：
	- `Claims` 是你自己定义的结构体（后面会说），用来存放你想塞进 JWT 的自定义数据。
		- `UserID`: 比如 `"user123"`，表示这是哪个用户。
		- `JTI`: JWT 的唯一 ID（类似 UUID），用于防重放攻击（比如防止别人重复使用同一个 token）。
	- `RegisteredClaims` 是 JWT 标准里规定的字段（来自 RFC 7519）：
		- `IssuedAt`: token 签发时间（这里是 `now`）。
		- `ExpiresAt`: token 过期时间（这里是 `now + AccessTTL`，比如 15 分钟后过期）。

> 💡 `jwt.NewNumericDate()` 是把 `time.Time` 转成 JWT 要求的 Unix 时间戳格式。

---
<br/>

**生成两个 Token（Access Token + Refresh Token）**

```go
at, _ := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString(Secret)
```

- **✅ 解释：**
	- `jwt.NewWithClaims(...)`: 创建一个 JWT 对象。
		- 使用签名算法 `HS256`（HMAC-SHA256，对称加密，用同一个密钥签名和验证）。
		- 把上面定义的 `claims` 塞进去。
	- `.SignedString(Secret)`: 用你的密钥 `Secret`（比如 `"my-secret-key"`）对这个 JWT 签名，生成一串字符串。
	- `at` 就是 **Access Token**（短期有效，比如 15 分钟）。

---
<br/>

```go
claims.ExpiresAt = jwt.NewNumericDate(now.Add(RefreshTTL))
rt, _ := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString(Secret)
```

- **✅ 解释：**
	- 把刚才的 `claims` 的过期时间改成更长的（比如 7 天）→ 这是为了生成 **Refresh Token**。
	- 再次签名，得到 `rt`（Refresh Token）。
	- 注意：这里复用了同一个 `claims` 结构，只是改了 `ExpiresAt`。

> ⚠️ 小问题：严格来说，Refresh Token 应该有**不同的 claims 结构**（比如不包含权限信息），或者至少 `JTI` 不同。但你这段代码为了简单复用了，实际项目中建议分开定义或重新赋值 `JTI`。

---
<br/>

**解析（验证）一个 Refresh Token**

```go
claims := &Claims{}
t, err := jwt.ParseWithClaims(refreshToken, claims, func(t *jwt.Token) (any, error) {
    return Secret, nil
})
```

- ✅ 解释：
	- `claims := &Claims{}`: 创建一个空的 `Claims` 实例，用于接收解析出来的数据。
	- `jwt.ParseWithClaims(...)`:
		- 输入：一个 token 字符串（`refreshToken`）
		- 输出：解析后的 token 对象 `t` 和错误 `err`
	- 第三个参数是一个 **key 函数**：
		- 当 JWT 库需要验证签名时，会调用这个函数。
		- 它返回用于验证的密钥（就是你签发时用的 `Secret`）。
		- 这里直接返回 `Secret`，因为用的是对称加密（HS256）。

> ✅ 如果 `err == nil` 且 `t.Valid == true`，说明：
> - 签名正确（没被篡改）
> - 没过期（`ExpiresAt` 有效）
> - `claims` 里已经填好了 `UserID`, `JTI` 等数据！

---
<br/>

**整个流程干了啥？**

1. **登录成功时**：
   - 生成一个短期有效的 **Access Token**（用于 API 请求）
   - 生成一个长期有效的 **Refresh Token**（用于换新 Access Token）
   - 返回给前端（比如存到 localStorage / cookie）

2. **前端每次请求 API**：
   - 带上 `Authorization: Bearer <access_token>`
   - 后端用 `ParseWithClaims` 验证，取出 `UserID`，知道是谁在操作

3. **Access Token 过期后**：
   - 前端用 `refresh_token` 调用 `/refresh` 接口
   - 后端验证 refresh token 合法 → 发一个新的 access token

