- [**MySQL数据库编程**](#MySQL数据库编程)
- [mysql使用前命令和配置](#mysql使用前命令和配置)
- [下载go-mysql驱动程序](#下载go-mysql驱动程序) 
- [操作mysql数据库](#操作mysql数据库)  	
	- [查询mysql版本](#查询mysql版本)
- [database/sql抽象层接口](#database/sql抽象层接口)
- [SQL架构设计](#SQL架构设计)
	- [大厂底层SQL设计](#大厂底层SQL设计)
- [新建数据表user](#新建数据表user)
- [插入数据](#插入数据)
	- [管理员角色批量绑定](#管理员角色批量绑定)
	- [INSERT ... ON DUPLICATE KEY UPDATE 【插入数据优化】](#INSERTONDUPLICATEKEYUPDATE)
- [查询数据](#查询数据)
	- [查寻一条数据](#查寻一条数据)
	- [database/sql Rows 与 Cursor](#databaseSQLRows与Cursor)
	- [scanAdminUserRow 行扫描](#scanAdminUserRow行扫描)
	- [角色列表游标分页 SQL](#角色列表游标分页SQL)
- [增加一条数据](#增加一条数据)
- [修改数据](#修改数据)
	- [Tx ExecContext事务执行](#TxExecContext事务执行)
	- [动态 UPDATE user_security](#动态UPDATEuser_security)
- [删除数据](#删除数据)
- [结构体中sql.NullString使用](#结构体中sql.NullString使用)
- [数据库&表执行方式](#数据库&表执行方式)
	- [终端手动执行](#终端手动执行)
	- [脚本执行](#脚本执行) 
	- [数据库迁移工具](#数据库迁移工具) 
	- [Go内执行sql文件](#Go内执行sql文件)
- [mysql指令](#mysql指令)
- [xxx.sql文件加注释方式](#xxx.sql文件加注释方式)




<br/>

***
<br/><br/><br/>
> <h1 id="MySQL数据库编程">MySQL数据库编程</h1>

<br/><br/><br/>
> <h2 id="mysql使用前命令和配置"> mysql使用前命令和配置</h2>


- **MySQL启动**

```
# /usr/local/mysql/support-files/mysql.server 是mysql的启动脚本
sudo /usr/local/mysql/support-files/mysql.server start 
```

- **MySQL暂停**

```
# /usr/local/mysql/support-files/mysql.server 是mysql的启动脚本
sudo /usr/local/mysql/support-files/mysql.server stop 
```

为了把路径`/usr/local/mysql/support-files`省略，可以在 `~/.bash_profile`中配置：

```
# 是为了简写mysql.server脚本的路径，方便启动
export PATH=$PATH:/usr/local/mysql/support-files
```

这样在终端输入：

```
sudo mysql.server start
Password:

Starting MySQL
.. SUCCESS! 
ganghuang@GangHuangs-MacBook-Pro etc % 
```

就可以把mysql进行启动了！！

<br/>

上述**通过命令启动mysql会有系统的守护作用**，当然通过`系统->偏好设置->mysql`进行启动也是可以的，但是没有了守护buff了，一般不建议。

<br/><br/>

在安装mysql时，其实也自动安装了一个工具（客户端），让我们快速实现连接MySQL并发送指令：

![go.0.0.35.png](./../Pictures/go.0.0.35.png)

![go.0.0.36.png](./../Pictures/go.0.0.36.png)


<br/><br/>

因为也配置了启动mysql的环境变量，若以也可以这样启动：

```
mysql -h 127.0.0.1  -P 3306 -u root -p

// 或者（在使用下面的命令执行时要确定是否mysql启动起来了）
// 查看服务状态： sudo /usr/local/mysql/support-files/mysql.server status
// 若是mysql没有启动，启动： sudo mysql.server start
```

![go.0.0.39.png](./../Pictures/go.0.0.39.png)

<br/>

- **进入mysql指令环境**

```
sudo mysql -u root -p
```

<br/><br/><br/>

> <h2 id="下载go-mysql驱动程序">下载go-mysql驱动程序</h2>

- **在终端需要定位到项目文件夹下**

```
cd /Users/ganghuang/HGFiles/GitHub/GoProject/MLC_GO
```

<br/>

- **初始化go.mod文件**

```
go mod init github.com
```

<br/>

- **‌ 下载go-mysql驱动程序**

```
go get -u github.com/go-sql-driver/mysql


go: downloading github.com/go-sql-driver/mysql v1.8.1
go: downloading filippo.io/edwards25519 v1.1.0
go: added filippo.io/edwards25519 v1.1.0
go: added github.com/go-sql-driver/mysql v1.8.1
```

go-mysql驱动程序下载完成，可以看到的go-mysql驱动程序的名称及其版本号，即mysql v 1.8.1。

<br/><br/><br/>

> <h2 id="操作mysql数据库">操作mysql数据库</h2>


- **导入go-mysql驱动程序**

```
import (
	// 当导入带有空白标识符前缀“_”的包时，将调用包的init()函数，以注册go-mysql驱动程序。
	_ "github.com/go-sql-driver/mysql"
	"database/sql"
)
```

<br/>

- **创建一个数据库对象**

在与MySQL数据库建立连接后，需调用sql包中的Open()函数创建一个数据库对象。

```
db, err := sql.Open("mysql", "<user>:<password>@tcp(127.0.0.1:3306)/<database-name>")
```
- **参数说明如下。**
	- user: MySQL数据库的用户名。
	- password: MySQL数据库的密码。
	- database-name：自定义数据库的名称。

- **说明:** 
	- 调用sql包中的Open()函数，打开由其数据库驱动程序名称和驱动程序特定数据源名称指定的数据库，通常至少由数据库名称和连接信息组成。

这行代码既不与MySQL数据库建立任何连接，也不验证go-mysql驱动程序的连接参数，而是创建一个数据库对象。


<br/><br/>

> <h3 id="查询mysql版本">查询mysql版本</h3>

**查询mysql服务是否在启动**

```
sudo /usr/local/mysql/support-files/mysql.server status

```

<br/>

- **若是没有启动，启动服务**

```
sudo /usr/local/mysql/support-files/mysql.server start
```

<br/>


- **进入执行mysql环境**

```
sudo mysql -u root -p
```
<br/><br/>

**显示数据库版本Demo**

```
package main

import (
	"fmt"
	"log"

	// 当导入带有空白标识符前缀“_”的包时，将调用包的init()函数，以注册go-mysql驱动程序。
	"database/sql"

	_ "github.com/go-sql-driver/mysql"
)

func testMysqlV1() {
	// 创建数据对象，xx109 是进入mysql密码
	db, err := sql.Open("mysql", "root:xx109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	var version string                                     // 声明 MySQL 数据库版本
	err2 := db.QueryRow("SELECT VERSION()").Scan(&version) // 单行查询

	if err2 != nil {
		log.Fatal(err2)
	}

	fmt.Println(version) // 打印 MySQL 数据库版本
}

func main() {
	testMysqlV1()
}
```

打印：

```
ganghuang@GangHuangs-MacBook-Pro TestMySQLV1 % go run test_mysql_v1.go                                
8.4.0
```


<br/>

***
<br/><br/><br/>
> <h1 id="database/sql抽象层接口"> database/sql抽象层接口</h1>

`database/sql`、gRPC、AWS SDK for Go、go-redis 虽然用途完全不同，但都有一个共同设计思想：**Command Object（命令对象）+ Result Object（结果对象）+ 延迟处理（Deferred Execution）**。这是 Go 大型框架最经典的设计模式之一。

***
<br/>

## 先理解什么叫 Command Object（命令对象）

执行一条 SQL 时，很多人会设计成直接返回结果：

```go
name, err := db.GetUserName(1001)
```

而大厂 SDK 更喜欢先返回命令对象：

```go
cmd := db.GetUserName(1001)
```

此时得到的是 `GetUserNameCommand`，它可能保存 SQL、参数、`Context`、`Timeout`、`Retry` 等信息：

```text
GetUserNameCommand
    ├── SQL
    │   SELECT name
    │   FROM user
    │   WHERE id = 1001
    ├── 参数: 1001
    ├── Context: ctx
    ├── Timeout
    └── Retry
```

这时候它不是结果，只是在描述：**我要执行什么**。这就是 Command Object。

***
<br/>

## 什么叫 Result Object（结果对象）

执行以后得到的值和错误，就是 Result Object：

```text
Result
    ├── Name: Tom
    └── Error: nil
```

也可以理解成：

```text
Result
    ↓
Value
    ↓
Err
```

***
<br/>

## go-redis 就是 Command Object

例如：

```go
cmd := client.Get(ctx, "user:1001")
```

返回的是 `*StringCmd`，里面包含 key、value、err：

```text
StringCmd
    ↓
key = user:1001
    ↓
value
    ↓
err
```

真正取结果时才调用：

```go
name, err := cmd.Result()
```

源码大概是：

```go
type StringCmd struct {
    baseCmd
    val string
}

func (cmd *StringCmd) Result() (string, error) {
    return cmd.val, cmd.err
}
```

***
<br/>

## database/sql 为什么也是这种思想？

例如：

```go
row := db.QueryRowContext(
    ctx,
    "select name from user where id=?",
    1001,
)
```

这里返回的是 `*sql.Row`，不是 `string`。因为数据库此时还没有解析出 `name`，真正读取结果是在 `Scan()`：

```go
var name string
err := row.Scan(&name)
```

流程是：

```text
QueryRow
    ↓
Row Object
    ↓
Scan()
    ↓
Result
```

这也是 Command / Result 思想。

***
<br/>

## Rows 更明显

例如：

```go
rows, err := db.QueryContext(...)
```

返回的是 `*sql.Rows`。`Rows` 不是最终结果，只是一个带游标的 `Result Set`：

```text
Result Set
    ↓
Cursor
    ↓
Row1
    ↓
Row2
    ↓
Row3
```

真正读取时：

```go
for rows.Next() {
    rows.Scan(...)
}
```

这和 Redis Scan 很像。

***
<br/>

## gRPC 是什么？

很多人容易误解为 `gRPC = RPC`，其实这不是重点。gRPC 真正解决的是：**微服务之间调用 HTTP 的各种痛点**。

以前 Service A 调用 Service B，通常是 HTTP：

```text
POST /user/info

{
  "id": 1001
}
```

问题包括：JSON 解析慢；字段容易写错，例如 `userid`、`userId`、`user_id` 都可能出现；API 只能看文档；前后端容易不一致；HTTP/1 连接利用率低。

于是 Google 推出 gRPC。它基于 HTTP/2，采用 Protocol Buffer，例如：

```protobuf
service UserService {
    rpc GetUser(UserRequest) returns (UserReply);
}
```

然后自动生成各语言客户端：

```go
client.GetUser(...)
```

```java
client.getUser(...)
```

```python
client.get_user(...)
```

这些代码全部自动生成，不需要自己写 HTTP。

<br/><br/>
### gRPC 解决什么痛点？

以前：

```text
JSON
    ↓
Marshal
    ↓
HTTP
    ↓
Unmarshal
```

现在：

```text
protobuf
    ↓
Binary
    ↓
HTTP2
    ↓
protobuf
```

所以 gRPC 速度更快、类型安全、接口自动生成，并支持 Streaming / 双向流。字节、阿里、腾讯内部微服务几乎大量使用 gRPC。

***
<br/>

## AWS SDK 是什么？

AWS 提供几百个云服务，例如：

```text
S3
EC2
Lambda
SNS
SQS
DynamoDB
CloudWatch
```

如果没有 SDK，上传 S3 时要自己处理 HTTP、`Authorization`、`Signature`、`Header`、`Body`，非常复杂。AWS SDK 会自动完成：

```text
签名
    ↓
HTTP
    ↓
Retry
    ↓
Error
    ↓
Response
```

例如上传 S3：

```go
resp, err := client.PutObject(
    ctx,
    &s3.PutObjectInput{
        Bucket: aws.String("video"),
        Key:    aws.String("1.mp4"),
        Body:   file,
    },
)
```

这里传入的是 `PutObjectInput`，不是十几个参数。因为以后新增 `ACL`、`Metadata`、`Tag`、`Encryption` 等字段时，不用修改函数签名。

返回的是 `PutObjectOutput`，里面可能包含：

```text
ETag
VersionID
Location
```

这就是 Result Object。

***
<br/>

## AWS SDK 最经典的 Command + Result

上传对象的链路是：

```text
PutObject
    ↓
PutObjectInput
    ↓
SDK
    ↓
PutObjectOutput
```

代码：

```go
input := &s3.PutObjectInput{
    Bucket: aws.String("video"),
    Key:    aws.String("1.mp4"),
    Body:   file,
}

output, err := s3Client.PutObject(ctx, input)
```

这里：

```text
PutObjectInput
    ↓
Command Object

PutObjectOutput
    ↓
Result Object
```

非常典型。

***
<br/>

## 大厂为什么喜欢这种设计？

假设今天函数是：

```go
func Upload(
    bucket string,
    key string,
    body io.Reader,
)
```

一年以后增加 `Metadata`、`StorageClass`、`ACL`、`Tag`、`CacheControl`、`ContentType`、`Encryption` 等字段，函数可能变成：

```go
Upload(
    bucket,
    key,
    body,
    metadata,
    acl,
    tag,
    cache,
    ...
)
```

二十多个参数会造成维护灾难。所以更推荐统一成：

```go
Upload(
    ctx,
    &UploadInput{
        Bucket:   ...,
        Key:      ...,
        ACL:      ...,
        Metadata: ...,
    },
)
```

以后增加字段，不用修改 API。

***
<br/>

## 四个库设计思想对比

| 库              | 命令对象（Command）                      | 结果对象（Result）                        | 解决的核心问题                     |
| -------------- | ---------------------------------- | ----------------------------------- | --------------------------- |
| go-redis       | `StringCmd`、`ScanCmd`、`IntCmd`     | `Result()` 返回值                      | 统一 Redis 命令、支持 Pipeline 和事务 |
| `database/sql` | `*sql.Row`、`*sql.Rows`、`*sql.Stmt` | `Scan()` 填充变量                       | 统一数据库访问接口、流式读取结果集           |
| gRPC           | `XXXRequest`（如 `GetUserRequest`）   | `XXXResponse`（如 `GetUserResponse`）  | 解决微服务之间高性能、类型安全的通信问题        |
| AWS SDK for Go | `PutObjectInput`、`GetObjectInput`  | `PutObjectOutput`、`GetObjectOutput` | 屏蔽云服务 API 细节，统一认证、重试、序列化等能力 |

***
<br/>

## 你的视频系统以后也建议采用这种模式

例如，不建议：

```go
PublishVideo(
    ctx,
    submissionID,
    userID,
    publishTime,
    notify,
    operator,
)
```

更推荐：

```go
type PublishVideoCommand struct {
    SubmissionID int64
    UserID       int64
    PublishTime  time.Time
    Notify       bool
    Operator     string
}

type PublishVideoResult struct {
    VideoID     int64
    PublishedAt time.Time
    Status      string
}

func (s *VideoService) Publish(
    ctx context.Context,
    cmd *PublishVideoCommand,
) (*PublishVideoResult, error)
```

这种设计与 `gRPC` 的 `Request/Response`、AWS SDK 的 `Input/Output`、go-redis 的命令对象设计风格一致，扩展性更好，也更符合大型 Go 项目的工程实践。


<br/>

***
<br/><br/><br/>
> <h1 id="SQL架构设计">SQL架构设计</h1>

***
<br/><br/><br/>
> <h2 id="大厂底层SQL设计">大厂底层SQL设计</h2>

这是一个非常好的方向，但要先澄清一个容易误解的地方：**阿里、字节、腾讯并不会把 MySQL 查询全部设计成 `GetUserNameCommand` 这种 Command Object。**这是很多文章误导的地方。

真正的大厂核心服务，尤其 Go 服务，一般采用的是：

```text
HTTP
    │
Handler
    │
Request DTO
    │
Service
    │
Repository
    │
database/sql
    │
MySQL
```

而 **Command Object 更多用于**：

* CQRS（Command Query Responsibility Segregation）
* Event Sourcing
* Saga
* 工作流
* DDD Application Service
* SDK（AWS、gRPC）
* Pipeline
* 异步任务

**Repository 里的 SQL 一般不会包装成几十个 Command 类。**

***
<br/>

## 那大厂 SQL 层到底是什么样？

如果对标字节（抖音）、阿里（淘宝）、腾讯（微信视频号）这种高并发 Go 服务，SQL 层一般长这样：

```text
internal/
    repository/
        mysql/
            video/
                command/
                    create_submission.go
                    update_publish.go
                    delete_video.go
                query/
                    get_submission.go
                    list_submission.go
                    search_video.go
                sql/
                    submission.sql.go
                repository.go
```

注意：这里的 **Command 不是 SQL 命令对象**，而是**业务 Command**。例如 `CreateSubmissionCommand` 表示“我要创建一个投稿”，而不是 `INSERT Command`。

***
<br/>

## 真正的大厂 Command Object

例如投稿，不是直接写：

```go
repo.Insert(...)
```

而是：

```go
cmd := &CreateSubmissionCommand{
    SubmissionID: submissionID,
    UserID:       uid,
    Title:        title,
    Description:  desc,
    PublishType:  Scheduled,
    PublishTime:  publishTime,
}
```

Command 定义：

```go
package command

import "time"

type CreateSubmissionCommand struct {
    SubmissionID string
    UserID       int64

    Title        string
    Description  string
    PublishType  int8
    PublishTime  time.Time
    ClientIP     string
    TraceID      string
    RequestID    string
}
```

这里没有 SQL、没有 `db`、没有 `Exec()`，因为 Command 只是**描述业务**。

***
<br/>

## Repository

Repository 接收业务 Command：

```go
type SubmissionRepository interface {
    Create(
        ctx context.Context,
        cmd *command.CreateSubmissionCommand,
    ) error
}
```

实现：

```go
type submissionRepository struct {
    db *sql.DB
}

func (r *submissionRepository) Create(
    ctx context.Context,
    cmd *command.CreateSubmissionCommand,
) error {
    _, err := r.db.ExecContext(
        ctx,
        insertSubmissionSQL,
        cmd.SubmissionID,
        cmd.UserID,
        cmd.Title,
        cmd.Description,
        cmd.PublishType,
        cmd.PublishTime,
    )

    return err
}
```

SQL 仍然非常简单。

***
<br/>

## SQL 独立

SQL 不会写进 Repository，而是独立放在：

```text
sql/
    submission.sql.go
```

例如：

```go
package sql

const InsertSubmissionSQL = `
INSERT INTO video_submission
(
    submission_id,
    user_id,
    title,
    description,
    publish_type,
    publish_time
)
VALUES
(
    ?,?,?,?,?,?
)
`
```

Repository 调用：

```go
db.ExecContext(
    ctx,
    sql.InsertSubmissionSQL,
    ...
)
```

***
<br/>

## Query Object

查询例如 `GET /user/videos`，不是十几个参数，而是：

```go
type ListSubmissionQuery struct {
    UserID int64
    Cursor int64
    Limit  int
    Status int8
}
```

Repository：

```go
func (r *Repository) List(
    ctx context.Context,
    query *ListSubmissionQuery,
) ([]*Submission, error)
```

SQL：

```sql
SELECT
    ...
```

***
<br/>

## Result Object

查询返回：

```go
type SubmissionResult struct {
    SubmissionID string
    UserID       int64
    Status       int8
    PublishTime  time.Time
}
```

Repository 返回 `[]SubmissionResult`，而不是几十个参数。

***
<br/>

## 真正面对千万并发会继续拆

Repository 会继续拆成写库和读库：

```text
Repository
      │
  ┌───┴──────────┐
  │              │
WriteRepo     ReadRepo
  │              │
Master       Replica
```

创建投稿走主库：

```text
CreateSubmission()
    ↓
Master
```

查询走读库：

```text
ListSubmission()
    ↓
Read Replica
```

***
<br/>

## 真正的大厂 SQL Executor

Repository 不会总是直接依赖 `db.ExecContext()`，而是封装 `DBExecutor`：

```go
type Executor interface {
    ExecContext(
        ctx context.Context,
        sql string,
        args ...any,
    ) (sql.Result, error)

    QueryContext(...)
}
```

Repository 依赖 `Executor`，不是 `sql.DB`，这样方便事务：

```text
sql.DB
    ↓
Executor
    ↓
Tx
    ↓
Mock
```

全部统一。

***
<br/>

## 真正的大厂结构

```text
cmd/

internal/
    api/
    service/
    application/
        command/
            create_submission.go
            publish_video.go
        query/
            list_video.go
    repository/
        mysql/
            submission/
                repository.go
                sql.go
        redis/
        kafka/
    domain/
        entity/
        aggregate/
        repository/
    infrastructure/
        mysql/
        redis/
        kafka/

pkg/
```

这是目前国内一线互联网公司比较常见的分层思路。

***
<br/>

## 如果要求完整 SQL Command Framework

如果要求的是“完全按照字节/阿里 P6~P8 核心 Go 服务规范，设计一套可以直接运行、支持亿级数据、千万 QPS 的 SQL Command Framework（包含连接池、读写分离、Sharding、Repository、Command Bus、Query Bus、事务、Pipeline、Metrics、Trace、Retry、熔断等）”，那就不是一两个回答能够讲完的内容，大约需要 **5000~10000 行 Go 代码**，会包含二三十个包，基本相当于一个小型数据库访问框架。

它通常会包括：

* DB Core（连接池封装）
* Executor（统一执行器）
* Command Bus
* Query Bus
* Repository Framework
* Transaction Manager
* Unit of Work
* Sharding Router
* Read/Write Router
* SQL Builder
* Metrics（Prometheus）
* Trace（OpenTelemetry）
* Retry
* Circuit Breaker
* Cache Aside（Redis）
* Kafka Outbox
* Cursor Pagination
* Batch Executor
* Prepared Statement Cache
* SQL 模板管理
* Mock/Test Framework

这已经是一个完整的基础设施项目，而不是几个示例文件的规模。对于现在的视频平台项目来说，这样的框架可以设计，但需要按模块逐步搭建，而不是在一次回复中给出全部代码。






<br/><br/><br/>

***
<br/><br/>

> <h1 id="新建数据表user">新建数据表user</h1>

在数据库db_test中新建数据表user。在数据表user中，包含主键id和用户的名字name。为了实现上述操作，可以使用如下的SQL语句。

```
CREATE TABLE user(id INT NOT NULL, name VARCHAR(20), PRIMARY KEY(ID));
```

<br/>

`sql.Open(driverName, dataSourceName string)` 中的数据源dataSourceName范例解析：

```sh
root:password@tcp(127.0.0.1:3306)/hg_mlc_db?charset=utf8mb4&parseTime=True&&loc=UTC
│     │        │        │            │              │             │           └─ 时区设为本地（如 Asia/Shanghai）
│     │        │        │            │              │             └─ 自动将 TIME/DATE 转为 Go 的 time.Time
│     │        │        │            │              └─ 指定连接字符集为 utf8mb4（支持 emoji 等）
│     │        │        │            └─ 要连接的数据库名（必须已存在！）
│     │        │        └─ MySQL 服务器地址和端口
│     │        └─ 使用 TCP 协议连接（也可用 unix socket）
│     └─ 密码（明文，生产环境建议用配置或环境变量）
└─ 用户名
```

- **❌ 问题：loc=Local 使用服务器本地时区，在面向国际化的时候会有问题：**

```sh
loc=Local
```
这是最值得调整的部分。

- `loc=Local` 表示 Go 程序使用 运行该程序的机器的本地时区（比如你服务器在上海，就是 Asia/Shanghai）。
- 当你把时间存入数据库（尤其是 DATETIME 类型），MySQL 不会存储时区信息。
- 如果你的用户来自不同时区（如纽约、伦敦、东京），而你又用 loc=Local 去解析或写入时间，会导致：
- 同一个 UTC 时间，在不同部署环境下显示不同；
- 数据混乱，难以做跨时区业务逻辑（如“用户看到的是自己当地时间”）。

<br/>

**DSN 中指定 loc=UTC**

```go
dsn := "...&loc=UTC"
```
- 所有时间在 Go 中以 time.Time（UTC）处理；
- 存入数据库的是 UTC 时间；
- 前端或业务层根据用户时区转换显示（如 t.In(time.FixedZone("Tokyo", 9*3600))）


<br/>

```go
//  新建数据表user
func TestMySQLV2_createTable() {
	// 创建数据对象
	db, err := sql.Open("mysql", "root:hh109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	// 执行SQL语句
	_,err2 := db.Exec("CREATE TABLE user(id INT NOT NULL, name VARCHAR(20), PRIMARY KEY(ID));")
	if err2 != nil {
		log.Fatal(err2)
	}
	
	fmt.Print("已成功新建数据表 user! \n") // 打印 MySQL 数据库版本
}
```

打印：

```
已成功新建数据表 user! 
```

<br/>

在终端进行查看创建的表：

**查看数据库**

```

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db_test            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

```

<br/>
**使用db_test;**

```
mysql> use db_test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_db_test |
+-------------------+
| user              |
+-------------------+
1 row in set (0.01 sec)

```

<br/><br/><br/>

> <h2 id="插入数据">插入数据</h2>

向数据表user插入一条数据。其中，id的值为1, name的值为David。为了实现上述操作，可以使用如下的SQL语句。

```
"INSERT INTO user VALUES(1, 'David')"
```

<br/>

```
// 插入数据
func testMySQLV2_insert() {
	// 创建数据对象
	db, err := sql.Open("mysql", "root:hh109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	_,err2 := db.Query("INSERT INTO user VALUES(1, 'David')")
	if err2 != nil {
		log.Fatal(err2)
	}
	fmt.Print("已成功向数据表 user 插入数据！\n")
}
```

打印：

```
已成功向数据表 user 插入数据！
```

<br/>

**命令查看：**

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db_test            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use db_test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_db_test |
+-------------------+
| user              |
+-------------------+
1 row in set (0.01 sec)


mysql> select * from user;
+----+-------+
| id | name  |
+----+-------+
|  1 | David |
+----+-------+
1 row in set (0.00 sec)
```


***
<br/><br/>
># <h3 id="管理员角色批量绑定">[管理员角色批量绑定](管理员角色批量绑定)</h3>

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


<br/><br/><br/>
> <h2 id="查询数据">查询数据</h2>

使用SQL语句`(select * from user；)`可以查询数据表user中的所有数据。如果使用Go语言实现，那么除了要使用上述SQL语句，还要通过数据库对象调用Query()函数以执行SQL语句。代码如下。

```
result,err3 := db.Query("SELECT * FROM user")
```

<br/>

```
// 查询数据
func testMySQLV2_query() {
	// 创建数据对象
	db, err := sql.Open("mysql", "root:hh109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	// 插入一条数据
	_,err2 := db.Query("INSERT INTO user VALUES(2, '司马懿🍎')")
	if err2 != nil {
		log.Fatal(err2)
	}

	// 查询数据表 user 中的所有数据
	result,err3 := db.Query("SELECT * FROM user")
	if err3 != nil {
		log.Fatal(err3)
	}

	//遍历查询结果
	for result.Next() {
		var id int	// 主键id
		var name string	// 用户的名字

		err = result.Scan(&id, &name)
		if err != nil {
			panic(err)
		}
		fmt.Printf("id: %d, name: %s\n", id,name)
	}

}
```

打印：

```
id: 1, name: David
id: 2, name: 司马懿🍎
```

<br/><br/><br/>
> <h2 id="查寻一条数据">查寻一条数据</h2>

```go
err := r.db.QueryRowContext(
    ctx,
    `SELECT id, email, phone, password_hash, salt
     FROM users WHERE id = ?`,
    id,
).Scan(
    &u.ID,
    &u.Email,
    &u.Phone,
    &u.PasswordHash,
    &u.Salt,
)
```

这是 Go 中使用标准库 `database/sql` **查询单行数据** 的经典写法。下面我们详细拆解 `QueryRowContext` 和 `.Scan()` 是如何工作的，以及它们背后的机制。

---
<br/>

**整体流程概览**

1. **`r.db.QueryRowContext(...)`**  
   → 向数据库发送一条 **只期望返回一行** 的 SQL 查询。
2. **返回一个 `*sql.Row` 对象**（代表“可能有一行结果”）。
3. **调用 `.Scan(...)`**  
   → 将这一行的各列值 **按顺序读取并赋值给传入的变量指针**。
4. 如果查询出错（如连接失败、SQL 语法错误）或查不到数据（`id` 不存在），`.Scan()` 会返回对应错误。

---
<br/>


```go
func (db *DB) QueryRowContext(ctx context.Context, query string, args ...interface{}) *sql.Row
```

- **🔹 作用**
	- 执行一条 **最多返回一行** 的 SQL 查询（通常是带主键或唯一条件的 `SELECT`）。
	- **不会立即执行查询**，而是返回一个 `*sql.Row` 对象，真正的数据库交互发生在 `.Scan()` 调用时。
	- 支持 `context.Context`：可用于超时控制、请求取消等。

<br/>

- **🔹 为什么叫 “Row”？**
	- 它假设你只关心 **一行结果**。即使 SQL 返回多行，它也**只取第一行**，其余忽略（但不报错）。
	- 如果你确实需要多行，请用 `QueryContext` + 循环 `rows.Next()`。

<br/>

- **🔹 特殊行为：查不到数据怎么办？**
	- 如果 SQL 返回 **0 行**，`QueryRowContext` **不会报错**！
	- 错误会在后续调用 `.Scan()` 时返回：`sql.ErrNoRows`

> ✅ 这是 Go 的惯用设计：把“无结果”当作一种可预期的状态，而非异常。

---
<br/>

```go
func (r *Row) Scan(dest ...interface{}) error
```

- **🔹 作用**
	- 将当前行的 **每一列的值**，按顺序 **赋值给 `dest` 中对应的变量指针**。
	- 自动进行 **类型转换**（如数据库 `INT` → Go `int64`，`VARCHAR` → `string` 等）。
	- 如果列数 ≠ `dest` 参数个数，会报错：`sql: expected X destination arguments, got Y`

<br/>

- **🔹 关键点：必须传指针！**

```go
// ❌ 错误：传的是值，Scan 无法修改原变量
Scan(u.ID, u.Email)

// ✅ 正确：传地址，Scan 才能写入
Scan(&u.ID, &u.Email)
```

<br/>

- **🔹 类型匹配规则（简化版）**

| 数据库类型（如 MySQL） | Go 目标类型（推荐） |
|------------------|------------------|
| `INT`, `BIGINT`     | `int64` / `*int64` |
| `VARCHAR`, `TEXT`   | `string`          |
| `BOOLEAN`           | `bool`            |
| `DATETIME`          | `time.Time`       |
| `NULL`              | 使用指针类型（如 `*string`）或 `sql.NullString` |

> 💡 如果字段可能为 `NULL`，建议用 `sql.NullXXX` 类型或指针，否则 `.Scan()` 会报错。

---
<br/>

**完整执行流程**

1. 调用 `QueryRowContext(ctx, sql, args...)`
   - 构造带参数的 SQL（防注入）
   - 返回一个 `*sql.Row` 对象（此时还没连数据库）

2. 调用 `.Scan(&a, &b, ...)`
   - 触发实际数据库查询
   - 驱动（如 `go-sql-driver/mysql`）执行 SQL
   - 获取结果集的第一行（如果有）
   - 按列顺序，将每列的值转换为目标 Go 类型
   - 写入到你传入的指针变量中
   - 如果出错（连接失败、类型不匹配、无结果等），返回 `error`

---
<br/>

**错误处理示例**

```go
var u model.User
err := r.db.QueryRowContext(ctx, "SELECT ...", id).Scan(&u.ID, &u.Email, ...)

if err != nil {
    if errors.Is(err, sql.ErrNoRows) {
        // 用户不存在
        return nil, ErrUserNotFound
    }
    // 其他数据库错误
    return nil, fmt.Errorf("failed to query user: %w", err)
}
// 成功，u 已被填充
return &u, nil
```

> ✅ **务必检查 `sql.ErrNoRows`**！这是“查不到”的标准错误。

---
<br/>

**对比其他方法**

| 方法 | 用途 | 返回 |
|------|------|------|
| `ExecContext` | 执行 `INSERT/UPDATE/DELETE` | `sql.Result`, `error` |
| `QueryContext` | 查询多行 | `*sql.Rows`, `error` |
| `QueryRowContext` | 查询单行 | `*sql.Row`（延迟执行）|

---
<br/>

**总结**

| 组件 | 作用 |
|------|------|
| **`QueryRowContext`** | 发起一个“最多返回一行”的查询，返回 `*sql.Row`（惰性执行） |
| **`.Scan(...)`** | 触发实际查询，并将结果列按顺序写入传入的指针变量 |
| **关键要求** | 列数 = 参数数；必须传指针；处理 `sql.ErrNoRows` |
| **适用场景** | 通过主键/唯一索引查单条记录（如用户登录、详情页） |

这种模式是 Go 操作数据库的 **标准且高效** 的方式，既安全（防注入），又简洁（一行搞定查询+赋值）。

***
<br/><br/><br/>
> <h2 id="databaseSQLRows与Cursor">database/sql Rows 与 Cursor</h2>


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
> <h2 id="scanAdminUserRow行扫描">scanAdminUserRow 行扫描</h2>


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



<br/><br/><br/>

***
<br/><br/>
> <h1 id="增加一条数据">增加一条数据</h1>

```go
func (r *UserRepo) Insert(ctx context.Context, u *model.User) error {
    res, err := r.db.ExecContext(
        ctx,
        `INSERT INTO users (email, phone, password_hash, salt)
         VALUES (?, ?, ?, ?)`,
        u.Email,
        u.Phone,
        u.PasswordHash,
        u.Salt,
    )
    if err != nil {
        return err
    }
    u.ID, _ = res.LastInsertId()
    return nil
}
```
`ExecContext`方法干嘛的？`LastInsertId()`有啥用？

***
<br/>

**SQL 插入语句**

```sql
INSERT INTO users (email, phone, password_hash, salt)
VALUES (?, ?, ?, ?)
```

- 向 `users` 表插入一条新记录。
- 使用了 **参数化查询（? 占位符）**，防止 SQL 注入。
- 插入字段不包括 `id`，说明 `id` 很可能是数据库自增主键（如 MySQL 的 `AUTO_INCREMENT`）。

<br/>

**🆔 `.LastInsertId()` 是什么？**

```go
u.ID, _ = res.LastInsertId()
```

- `res` 是 `sql.Result` 类型，由 `ExecContext` 返回。
- `.LastInsertId()` 是 Go 标准库 `database/sql` 提供的方法，用于**获取刚刚插入行的自增主键 ID**。
- 它只在**支持自增主键的数据库**（如 MySQL、SQLite）中有意义；在 PostgreSQL 中通常用 `RETURNING id` 配合 `QueryRowContext` 来实现类似功能。

> ⚠️ 注意：这里忽略了第二个返回值（错误），实际生产中建议检查：
```go
id, err := res.LastInsertId()
if err != nil {
     return fmt.Errorf("failed to get last insert id: %w", err)
 }
 u.ID = id
```

<br/>

**💡 为什么要把 ID 回填到 `u`？**

- 调用方可能需要知道新创建用户的 ID（比如后续关联其他表、返回 API 响应等）。
- 通过修改传入的指针 `u`，调用方可以直接使用 `u.ID`，无需额外查询。

例如：

```go
user := &model.User{
    Email: "alice@example.com",
    Phone: "13800138000",
    PasswordHash: "...",
    Salt: "...",
}
err := repo.Insert(ctx, user)
if err != nil { /* handle */ }

// 此时 user.ID 已被填充为数据库分配的 ID
fmt.Println("New user ID:", user.ID)
```

***
<br/>

```go
func (db *DB) ExecContext(ctx context.Context, query string, args ...interface{}) (sql.Result, error)
```
&emsp; 这个方法是 Go 语言标准库 `database/sql` 中的一个方法，**用于执行不返回行数据的 SQL 语句（比如 `INSERT`、`UPDATE`、`DELETE`）**，并支持上下文（`context.Context`）控制。

- **作用**：执行一条“写操作”SQL（不会返回结果集，只返回影响行数或自增 ID）。
- **典型用途**：
  - 插入新记录（`INSERT`）
  - 更新已有数据（`UPDATE`）
  - 删除数据（`DELETE`）

> ✅ 和它对应的“读操作”方法是 `QueryContext`（用于 `SELECT`，会返回多行数据）和 `QueryRowContext`（用于单行查询）。

<br/>

**参数说明**

| 参数 | 说明 |
|------|------|
| `ctx context.Context` | 上下文，用于控制超时、取消等。比如 HTTP 请求取消时，数据库操作也能及时停止。 |
| `query string` | 要执行的 SQL 语句，通常用 `?` 作为占位符（如 `"INSERT INTO users (name) VALUES (?)"`）。 |
| `args ...interface{}` | 替换 `?` 的实际参数值，自动防 SQL 注入。 |

<br/>

**返回值**

```go
res, err := db.ExecContext(ctx, "INSERT ...", ...)
```

- **`err`**：如果 SQL 执行出错（如连接失败、语法错误、唯一键冲突等），这里会返回错误。
- **`res sql.Result`**：包含执行结果的元信息，主要有两个方法：
  - `res.RowsAffected()` → 返回受影响的行数（比如更新了 3 行）。
  - `res.LastInsertId()` → 返回刚插入行的自增主键 ID（仅在 MySQL、SQLite 等支持的数据库中有效）。

---
<br/>

**为什么用 `ExecContext` 而不是 `Exec`？**

- `Exec` 是旧版方法，**不支持 `context.Context`**。
- `ExecContext` 允许你：
  - 设置超时（防止数据库慢查询拖垮服务）：
    ```go
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    db.ExecContext(ctx, "INSERT ...")
    ```
  - 在 HTTP 请求取消时自动中断数据库操作（提升系统健壮性）。

> ✅ **最佳实践：始终使用 `*Context` 版本的方法（如 `ExecContext`, `QueryContext`）**。

---
<br/>

**举个完整例子**

```go
_, err := db.ExecContext(ctx,
    "UPDATE users SET last_login = ? WHERE id = ?",
    time.Now(),
    userID,
)
if err != nil {
    return fmt.Errorf("failed to update last login: %w", err)
}
```

或者你的原始代码：

```go
res, err := r.db.ExecContext(ctx,
    "INSERT INTO users (email, phone, password_hash, salt) VALUES (?, ?, ?, ?)",
    u.Email, u.Phone, u.PasswordHash, u.Salt,
)
if err != nil {
    return err // 比如邮箱已存在（违反唯一索引）
}
u.ID, _ = res.LastInsertId() // 获取新用户的 ID
```



<br/><br/><br/>

> <h2 id="修改数据">修改数据</h2>


向数据表user插入第二条数据。其中，id的值为2, name的值为Leon。如果想把Leon修改为“张三”​，除了要使用修改name的值的SQL语句，还要通过数据库对象调用Exec()函数以执行SQL语句。代码如下。

```
// 修改用户名字的SQL语句
sql := "update user set name = ? WHERE id = ?"
_,err2 := db.Exec(sql, "张三🍔", 2)
```

<br/>

```
// 查询数据
func testMySQLV2_update() {
	// 创建数据对象
	db, err := sql.Open("mysql", "root:hh109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	// 修改用户名字的SQL语句
	sql := "update user set name = ? WHERE id = ?"
	_,err2 := db.Exec(sql, "张三🍔", 2)
	if err2 != nil {
		log.Fatal(err2)
	}
	fmt.Print("已成功修改数据表 user 中的数据！\n") //修改数据后，打印提示信息


	// 查询数据表 user 中的所有数据
	result,err3 := db.Query("SELECT * FROM user")
	if err3 != nil {
		log.Fatal(err3)
	}

	//遍历查询结果
	for result.Next() {
		var id int	// 主键id
		var name string	// 用户的名字

		err = result.Scan(&id, &name)
		if err != nil {
			panic(err)
		}
		fmt.Printf("id: %d, name: %s\n", id,name)
	}

}
```

打印：

```
ganghuang@GangHuangs-MacBook-Pro TestMySQLV1 % go run test_mysql_v1.go
已成功修改数据表 user 中的数据！
id: 1, name: David
id: 2, name: 张三🍔
```

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


<br/><br/><br/>
> <h2 id="删除数据">删除数据</h2>

向数据表user插入第一条数据。其中，id的值为1, name的值为David。如果想删除这条数据，除了要使用根据id删除用户的SQL语句，还要通过数据库对象调用Exec()函数以执行SQL语句。代码如下。

```
// 删除用户名字的SQL语句
	sql := "DELETE FROM user WHERE id = 1"
	_,err2 := db.Exec(sql)
```

<br/>

```
// 删除数据
func testMySQLV2_delete() {
	// 创建数据对象
	db, err := sql.Open("mysql", "root:hh109@tcp(127.0.0.1:3306)/DB_TEST")
	db.Ping()        // 与数据库建立连接
	defer db.Close() // 延迟关闭数据库

	if err != nil {
		fmt.Println("数据库连接失败！")
		log.Fatal(err)
	}

	// 删除用户名字的SQL语句
	sql := "DELETE FROM user WHERE id = 1"
	_,err2 := db.Exec(sql)
	if err2 != nil {
		log.Fatal(err2)
	}
	fmt.Print("已成功删除数据表 user 中的数据！\n") //修改数据后，打印提示信息


	// 查询数据表 user 中的所有数据
	result,err3 := db.Query("SELECT * FROM user")
	if err3 != nil {
		log.Fatal(err3)
	}

	//遍历查询结果
	for result.Next() {
		var id int	// 主键id
		var name string	// 用户的名字

		err = result.Scan(&id, &name)
		if err != nil {
			panic(err)
		}
		fmt.Printf("id: %d, name: %s\n", id,name)
	}

}
```

打印：

```
ganghuang@GangHuangs-MacBook-Pro TestMySQLV1 % go run test_mysql_v1.go
已成功删除数据表 user 中的数据！
id: 2, name: 张三🍔
```


***
<br/><br/><br/>
> <h2 id="结构体中sql.NullString使用">结构体中sql.NullString使用</h2>


```go
type User struct {
	ID           int64
	Email        sql.NullString
	Phone        sql.NullString
	PasswordHash string
	Salt         string
}
```

**⁉️提问： `sql.NullString 这个是干嘛的？用string不行吗`**

***
<br/>

- **结论先行：**
> **`sql.NullString` 是用来正确表示「数据库字段允许为 NULL」的。**
> 如果字段在数据库中可能为 `NULL`，**直接用 `string` 是不严谨、甚至是错误的**。

<br/>

**问题的本质：数据库有 `NULL`，Go 没有**

**数据库中的三种状态**

在数据库里，一个字段可以是：

| 状态      | 含义   |
| ------- | ---- |
| `NULL`  | 没有值  |
| `''`    | 空字符串 |
| `'abc'` | 有值   |

而在 Go 中：

```go
string
```

**只有两种状态：**

* 空字符串 `""`
* 非空字符串 `"abc"`

👉 **Go 的 `string` 无法表达 “NULL”**

<br/>

**`sql.NullString` 是什么？**

源码（简化）：

```go
type NullString struct {
    String string
    Valid  bool
}
```

**含义：**

| 字段       | 说明        |
| -------- | --------- |
| `String` | 实际字符串值    |
| `Valid`  | 是否为非 NULL |

**行为示例**

| 数据库字段值  | Go 结构体                                     |
| ------- | ------------------------------------------ |
| `NULL`  | `sql.NullString{String:"", Valid:false}`   |
| `''`    | `sql.NullString{String:"", Valid:true}`    |
| `'abc'` | `sql.NullString{String:"abc", Valid:true}` |

👉 **NULL 和空字符串被严格区分开了**

<br/>

**如果用 `string` 会发生什么？**

**1️⃣ 查询时的隐性错误**

假设数据库：

```sql
email VARCHAR(255) NULL
```

<br/>

你用：

```go
Email string
```

查询 `NULL` 时：

```go
Email == ""
```

问题来了：

* 你 **不知道**：
	* 是用户真的填了 `""`
	* 还是数据库是 `NULL`

逻辑判断会出错。

<br/>

**2️⃣ 扫描时可能直接报错（视驱动而定）**

某些 driver 在扫描 `NULL` 到 `string` 时会报：

```text
sql: Scan error on column index X: converting NULL to string is unsupported
```

<br/>

**3️⃣ 写入时语义丢失**

如果你想：

* “不更新 email”
* 或 “明确设置为 NULL”

用 `string` **无法表达**

<br/>

**什么时候必须用 `sql.NullString`？**

**✅ 必须使用的场景**

| 场景                    | 原因          |
| --------------------- | ----------- |
| 数据库字段允许 `NULL`        | Go 需要能表达    |
| 业务上区分「未填写」与「空值」       | `NULL ≠ ""` |
| 通用 DAO / Repository 层 | 数据语义必须准确    |

<br/>

**例子：**

```go
Email sql.NullString
Phone sql.NullString
```

这是**非常标准、正确的建模方式**。

<br/>

**什么时候可以用 `string`？**

**✅ 可以直接用 `string` 的前提**

**数据库层必须保证：**

```sql
email VARCHAR(255) NOT NULL DEFAULT ''
```

并且：

* 业务允许用 `""` 表示“无”
* 永远不关心 `NULL`

否则，不建议。

<br/>

**业务层怎么用 `sql.NullString` 才不别扭？**

**1️⃣ 判断是否存在值**

```go
if user.Email.Valid {
    fmt.Println(user.Email.String)
}
```

<br/>

**2️⃣ 设置值**

```go
user.Email = sql.NullString{
    String: "a@b.com",
    Valid:  true,
}
```

**3️⃣ 设置为 NULL**

```go
user.Email = sql.NullString{
    Valid: false,
}
```

---
<br/>

**进阶：为什么不用 `*string`？**

你可能会问：

> 那我用 `*string` 不也能表示 NULL 吗？

答案：**可以，但不推荐在 DB 层**

原因：

| 对比                  | `*string` | `sql.NullString` |
| ------------------- | --------- | ---------------- |
| 是否语义清晰              | 一般        | ✅ 非常清晰           |
| 是否 DB 标准            | ❌         | ✅                |
| 与 `database/sql` 集成 | ❌         | ✅                |
| 批量扫描安全性             | 差         | 好                |


<br/><br/><br/>

***
<br/>

> <h1 id="数据库&表执行方式">数据库&表执行方式</h1>


***
<br/><br/><br/>
> <h2 id="终端手动执行">终端手动执行</h2>
**终端手动执行 ⭐⭐⭐⭐**

**1️⃣ 项目结构**

```text
go-user-service/
├── migrations/
│   └── 001_init.sql
```

<br/>

**2️⃣ 执行命令（创建库 / 表）**

```bash
mysql -u root -p < migrations/001_init.sql
```

或者指定数据库：

```bash
mysql -u root -p app_db < migrations/001_init.sql
```

<br/>

执行流程是：

```
你 → mysql 客户端 → MySQL Server
```

**Go 程序完全不参与**

<br/>

**3️⃣ 企业真实情况**

* 本地：开发者自己执行
* 测试环境：开发或运维执行
* 生产环境：DBA 执行

这是**完全合规的企业做法**。

***
<br/><br/><br/>
> <h2 id="脚本执行">脚本执行</h2>
**Makefile / Shell 脚本（推荐进阶）⭐⭐⭐⭐⭐**

中大型项目**几乎一定会有这一层**。

<br/> 

**1️⃣ scripts/migrate.sh**

```bash
#!/bin/bash

MYSQL_USER=root
MYSQL_PASSWORD=123456
MYSQL_DB=app_db

mysql -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DB < migrations/001_init.sql
```

<br/>

**2️⃣ 执行**

```bash
chmod +x scripts/migrate.sh
./scripts/migrate.sh
```

- **好处**
	* 统一入口
	* 不怕忘命令
	* CI / 运维可复用

***
<br/><br/><br/>
> <h2 id="数据库迁移工具">数据库迁移工具</h2>
**数据库迁移工具（企业级标准）⭐⭐⭐⭐⭐**

这是**中大型公司最主流方案**。

- **常用工具**
	* `golang-migrate`（最常见）
	* `goose`
	* `flyway`（跨语言）

<br/>

**golang-migrate**

**1️⃣ 安装**

```bash
brew install golang-migrate
```

<br/>

**2️⃣ 目录结构（固定规范）**

```text
migrations/
├── 001_init.up.sql
├── 001_init.down.sql
├── 002_add_index.up.sql
├── 002_add_index.down.sql
```

<br/>

**3️⃣ 执行迁移**

```bash
migrate \
  -path migrations \
  -database "mysql://user:pass@tcp(127.0.0.1:3306)/app_db" \
  up
```

**特点：**
* 自动记录执行版本
* 不会重复执行
* 支持回滚

<br/>

**企业为什么一定用迁移工具？**

| 问题     | 手写 SQL | 迁移工具 |
| ------ | ------ | ---- |
| 防止重复执行 | ❌      | ✅    |
| 版本管理   | ❌      | ✅    |
| 回滚     | ❌      | ✅    |
| CI 自动化 | ❌      | ✅    |


***
<br/><br/><br/>
> <h2 id="Go内执行sql文件">Go内执行sql文件</h2>

**Go 程序“执行 SQL 文件”（不推荐，仅说明）**

你**可能会想到**这样做：

```go
sqlBytes, _ := os.ReadFile("001_init.sql")
db.Exec(string(sqlBytes))
```

**⚠️ 企业明确反对这样做**

原因：

1. 业务程序拥有 **DDL 权限（极其危险）**
2. 一次部署，多实例并发执行 → **直接事故**
3. 不可控、不可审计

**真实企业态度**

> **“宁愿部署失败，也不允许程序自动改表结构。”**

---
<br/>

 **Go 工程“什么时候”依赖这些 SQL？**

**正确时间线（非常重要）**

```
1️⃣ 执行 SQL（建库 / 建表）
2️⃣ 启动 Go 程序
3️⃣ Go 程序只做 CRUD
```

**顺序不能反**

---
<br/>

**Go 工程中如何“确认 SQL 已执行”？（正确方式）**

不是检查表是否存在，而是：

**启动时健康检查**

```go
func CheckDB(db *sql.DB) {
    if _, err := db.Exec("SELECT 1 FROM users LIMIT 1"); err != nil {
        panic("database schema not ready")
    }
}
```

* 表不存在 → 程序直接退出
* 由部署系统处理

---
<br/>


**真实企业完整流程图**

```
开发写 SQL
   ↓
SQL 提交 Git
   ↓
评审 / 审核
   ↓
执行迁移（人工 / CI / 工具）
   ↓
数据库 OK
   ↓
启动 Go 服务
```

---
<br/>

**当前最推荐采用的方案（明确建议）**

✅ **方式一 + 方式二**

* 手动执行 SQL
* 配合 shell / Makefile

<br/>

**下一阶段**

✅ **方式三：迁移工具**

* 尤其是 golang-migrate

---
<br/>
**下面可以针对如下做扩展：**

1️⃣ 手把手 **把 golang-migrate 集成进 Go 项目**
2️⃣ 讲清楚 **up / down SQL 怎么写才专业**
3️⃣ 演示 **生产环境数据库变更真实案例**

**比如： migrate”** 或 **“讲 CI / 生产流程”** 




<br/><br/><br/>

***
<br/>

> <h1 id="mysql指令">mysql指令</h1>

- **展示所有数据库**

```
mysql> show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```

<br/>

- **通过CREATE DATABASE语句创建一个名为db_test的数据库**

```
create database db_test;

Query OK, 1 row affected (0.00 sec)
```




<br/><br/><br/>

***
<br/><br/>
<h1 id='xxx.sql文件加注释方式'>xxx.sql文件加注释方式</h1>
**✅ SQL 中两种标准注释方式**

**1. **单行注释**：使用 `--`（注意后面要跟一个空格）**

```sql
-- 这是一个单行注释
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

> ⚠️ 注意：`--` 后**必须有一个空格**（或换行），否则某些数据库（如 MySQL）可能不识别。

<br/>

- **2. **多行注释**：使用 `/* ... */`**

```sql
/*
这是多行注释
可以写很多行
用于说明复杂逻辑
*/
INSERT INTO users (name) VALUES ('Alice');
```

也可以用于**行内注释**：
```sql
SELECT id, name /* 用户基本信息 */, created_at FROM users;
```

---
<br/>


**⚠️ 注意事项**

| 数据库 | 是否支持 | 特别说明 |
|--------|--------|--------|
| **MySQL** | ✅ | 支持 `-- ` 和 `/* */`；注意 `--` 后必须有空格 |
| **PostgreSQL** | ✅ | 完全支持 |
| **SQLite** | ✅ | 完全支持 |
| **SQL Server** | ✅ | 支持 `--`（不要求空格）和 `/* */` |
| **Oracle** | ✅ | 支持 |

> 💡 几乎所有主流数据库都遵循 SQL 标准的注释语法，所以你的 `.sql` 文件在不同数据库间迁移时，注释通常不会出问题。

---
<br/>

**完整的 `init_db.sql` 文件**

```sql
-- =============================================
-- 数据库初始化脚本
-- 作用：创建用户表和订单表
-- 作者：dev@example.com
-- 时间：2026-01-21
-- =============================================

/* 
 * 用户表：存储系统用户基本信息
 * 注意：email 必须唯一
 */
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    password_hash CHAR(64) NOT NULL,
    salt CHAR(32) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 订单表（后续再加外键约束）
CREATE TABLE orders (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status TINYINT DEFAULT 0,  -- 0:待支付, 1:已支付, 2:已取消
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

