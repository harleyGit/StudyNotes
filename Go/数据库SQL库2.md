- [视频状态统计](#视频状态统计)
	- [WHERE 与 GROUP BY](#WHERE-与-GROUP-BY)
	- [Redis 与 MySQL 统计的区别](#Redis-与-MySQL-统计的区别)
	- [亿级数据优化](#亿级数据优化)
- [Go + MySQL 数据库迁移与元数据检查](#Go+MySQL数据库迁移与元数据检查)
- [启动时执行 MySQL Migration](#启动时执行MySQLMigration)
- [检查指定索引是否存在](#检查指定索引是否存在)
- [MySQL information_schema](#MySQLinformation_schema)
- [Go 后端常用代码与数据库基础](#Go后端常用代码与数据库基础)
- [schema_migrations 迁移版本表](#schema_migrations迁移版本表)
- [视频标签游标分页 SQL](#视频标签游标分页SQL)
- [strings.SplitN](#strings.SplitN)
- [ORDER BY 与 LIMIT](#ORDER-BY与LIMIT)
- [识别 MySQL Duplicate Key](#识别MySQL-Duplicate-Key)
- [mysql.Config.FormatDSN](mysql.Config.FormatDSN)
- [ClickHouse 弹幕异步写入](#ClickHouse-弹幕异步写入)
	- [异步插入与等待结果](#异步插入与等待结果)




<br/>

***
<br/><br/><br/>
> <h1 id="视频状态统计"> 视频状态统计 </h1>

核心 SQL：

```go
id="GetVideoStatusCountsSQL"
GetVideoStatusCountsSQL = `
SELECT status, COUNT(*)
FROM video_submissions
WHERE status IN ('reviewing', 'published')
GROUP BY status`
```

作用：**统计 `video_submissions` 表中不同视频状态 `status` 的数量**，例如有多少视频正在审核 `reviewing`，有多少视频已经发布 `published`。

---
<br/>

## 假设表结构

`video_submissions` 可能类似：

| id | user_id | video_id | status    |
| -- | ------- | -------- | --------- |
| 1  | 100     | v001     | reviewing |
| 2  | 101     | v002     | published |
| 3  | 102     | v003     | published |
| 4  | 103     | v004     | failed    |
| 5  | 104     | v005     | reviewing |
| 6  | 105     | v006     | draft     |

---
<br/>

> <h3 id="WHERE-与-GROUP-BY">WHERE 与 GROUP BY</h3>

```sql
FROM video_submissions
```

表示从视频提交表查询，类似：

```go
db.QueryContext(ctx, GetVideoStatusCountsSQL)
```

数据库扫描：

```text
video_submissions
        |
        v
    查询状态
```

`WHERE status IN (...)` 只统计指定状态：

```sql
WHERE status IN ('reviewing', 'published')
```

等价：

```sql
WHERE status = 'reviewing'
   OR status = 'published'
```

原始数据：

| id | status    |
| -- | --------- |
| 1  | reviewing |
| 2  | published |
| 3  | published |
| 4  | failed    |
| 5  | reviewing |
| 6  | draft     |

过滤后：

| id | status    |
| -- | --------- |
| 1  | reviewing |
| 2  | published |
| 3  | published |
| 5  | reviewing |

`failed` 和 `draft` 被排除。

`COUNT(*)` 用于统计数量：

```text
reviewing: id=1, id=5 -> 2
published: id=2, id=3 -> 2
```

`GROUP BY status` 表示按照 `status` 分组统计：

```sql
GROUP BY status
```

执行过程：

```text
reviewing
------------
id=1
id=5

published
------------
id=2
id=3
```

再通过 `COUNT(*)` 计算：

```text
reviewing -> 2
published -> 2
```

最终返回：

| status    | COUNT(*) |
| --------- | -------- |
| reviewing | 2        |
| published | 2        |

Go 中通常这样读取：

```go
rows, err := db.QueryContext(ctx, GetVideoStatusCountsSQL)
for rows.Next() {
    var status string
    var count int64
    err := rows.Scan(&status, &count)
}
```

得到：

```text
reviewing = 2
published = 2
```

---
<br/>

> <h3 id="Redis-与-MySQL-统计的区别">Redis 与 MySQL 统计的区别</h3>

前面提到的：

```go
HGetAll(ctx, videoStatusCounterKey())
```

读取的是 Redis 实时统计：

```text
video:status:counter
{
    reviewing: 1000,
    published: 900000
}
```

而这个 SQL：

```sql
SELECT status, COUNT(*)
FROM video_submissions
GROUP BY status
```

是 MySQL 真实数据统计。两者用途不同：

|     | Redis HIncrBy | MySQL COUNT |
| --- | ------------- | ----------- |
| 用途 | 实时统计      | 真实统计    |
| 速度 | 微秒~毫秒     | 毫秒~秒     |
| 一致性 | 最终一致    | 强一致      |
| 适合 | 首页数据      | 后台校验    |

---
<br/>

> <h3 id="亿级数据优化">亿级数据优化</h3>

如果 `video_submissions` 有 10 亿行，执行：

```sql
SELECT status, COUNT(*)
FROM video_submissions
GROUP BY status
```

可能很慢，因为 MySQL 需要扫描大量数据、按 `status` 分组、再统计。

### 优化1：建立索引

查询条件是：

```sql
WHERE status IN (...)
GROUP BY status
```

适合建立索引：

```sql
CREATE INDEX idx_video_submission_status
ON video_submissions(status);
```

通过 `status` 索引树快速定位 `reviewing`、`published`。

### 优化2：覆盖索引

更进一步：

```sql
CREATE INDEX idx_status_cover
ON video_submissions(status, id);
```

因为查询只需要 `status` 和数量，可以减少回表。

---
<br/>

## 大厂通常怎么做

字节视频系统这类场景，不会每天用 `COUNT(*)` 扫描几十亿视频。常见架构：

```text
视频状态变化
    |
    +-------- MySQL
    |
    +-------- Kafka
                |
                v
             聚合服务
                |
                v
              Redis
```

状态变化：

```text
reviewing -> published
```

发送事件：

```json
{
    "video_id": "xxx",
    "old_status": "reviewing",
    "new_status": "published"
}
```

消费者更新计数：

```go
HIncrBy("video:status:counter", "published", 1)
HIncrBy("video:status:counter", "reviewing", -1)
```

查询时：

```go
HGetAll("video:status:counter")
```

毫秒返回。MySQL SQL 更多用于数据校验、离线统计、管理后台、定时任务修正 Redis。

---
<br/>

## 这个 SQL 在上传系统里的意义

结合 `video_files`、`video_submissions`、`submission_id`、`status`，状态流可能是：

```text
上传中
  |
  v
reviewing
  |
  v
published
```

这个 SQL 查询当前有多少视频审核中、已发布。例如后台 Dashboard：

```text
视频管理 Dashboard
审核中: 12000
已发布: 9800000
```

总结：

```sql
SELECT status, COUNT(*)
FROM video_submissions
WHERE status IN ('reviewing','published')
GROUP BY status
```

就是从视频提交表里只筛选「审核中」和「已发布」的视频，然后按状态分组，统计每种状态有多少条记录。这是典型的 **SQL 聚合统计查询（GROUP BY + COUNT）**。小中型系统可以直接使用，亿级系统通常会改为 **MySQL + MQ + Redis 实时聚合计数架构**。



***
<br/><br/><br/>

> <h2 id="Go+MySQL数据库迁移与元数据检查">Go + MySQL 数据库迁移与元数据检查</h2>

本文整理 Go 后端项目中的 4 个相关知识点：

- 根据配置生成 `golang-migrate` 的 MySQL URL，并调用 CLI 执行迁移。
- 通过 `information_schema.statistics` 判断索引是否存在。
- 理解 MySQL 元数据与常用系统库。
- 使用 `net.JoinHostPort` 和 `strings.TrimPrefix` 生成标准网络地址。

***
<br/>

> <h2 id="启动时执行MySQLMigration">启动时执行 MySQL Migration</h2>

程序启动时执行 `migrate up`，将数据库结构升级到迁移目录中的最新版本。核心实现如下：

```go
func buildMySQLMigrationURL(cfg ConfigPackage.HGMySQLConfig) string {
	credentials := url.UserPassword(cfg.User, cfg.Password).String()
	address := net.JoinHostPort(cfg.Host, cfg.Port)
	return fmt.Sprintf("mysql://%s@tcp(%s)/%s", credentials, address, url.PathEscape(cfg.Database))
}

func runMySQLMigrations(
	cfg ConfigPackage.HGMySQLConfig,
	migrationsDir string,
) error {
	migratePath, err := exec.LookPath("migrate")
	if err != nil {
		return fmt.Errorf("未找到 migrate 命令，请先安装 golang-migrate: %w", err)
	}

	cmd := exec.Command(
		migratePath,
		"-path",
		migrationsDir,
		"-database",
		buildMySQLMigrationURL(cfg),
		"up",
	)
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		return fmt.Errorf("执行数据库迁移失败: %w", err)
	}

	return nil
}
```

两个函数的职责：

- `buildMySQLMigrationURL`：将 MySQL 配置转换为 `golang-migrate` 使用的连接 URL。
- `runMySQLMigrations`：查找并调用系统中的 `migrate` 命令执行数据库升级。

---
<br/>

## 构建 MySQL Migration URL

假设配置为：

```go
cfg := HGMySQLConfig{
	User:     "root",
	Password: "123456",
	Host:     "127.0.0.1",
	Port:     "3306",
	Database: "video_db",
}
```

最终生成：

```text
mysql://root:123456@tcp(127.0.0.1:3306)/video_db
```

### `url.UserPassword`

```go
credentials := url.UserPassword(cfg.User, cfg.Password).String()
```

该方法生成并转义 URL 中的用户名、密码。例如密码为 `abc@123` 时，直接拼接会得到 `root:abc@123`，其中 `@` 会被当作 URL 分隔符；`url.UserPassword` 会生成：

```text
root:abc%40123
```

因此不要用下面的方式手动拼接凭据：

```go
credentials := cfg.User + ":" + cfg.Password
```

### `net.JoinHostPort`

```go
address := net.JoinHostPort(cfg.Host, cfg.Port)
```

对于 IPv4，结果为 `127.0.0.1:3306`。它比 `cfg.Host + ":" + cfg.Port` 更安全，因为可以正确处理 IPv6：

```go
net.JoinHostPort("2001:db8::1", "3306")
```

结果：

```text
[2001:db8::1]:3306
```

直接拼接会得到含义不明确的 `2001:db8::1:3306`。

### `fmt.Sprintf` 与 `url.PathEscape`

```go
return fmt.Sprintf(
	"mysql://%s@tcp(%s)/%s",
	credentials,
	address,
	url.PathEscape(cfg.Database),
)
```

三个 `%s` 依次对应凭据、网络地址和数据库名。`url.PathEscape` 会转义数据库名中的路径特殊字符，例如 `video db` 会变成 `video%20db`。

| 位置 | 内容 | 示例 |
| --- | --- | --- |
| 第一个 `%s` | 用户名和密码 | `root:123456` |
| 第二个 `%s` | 主机和端口 | `127.0.0.1:3306` |
| 第三个 `%s` | 数据库名 | `video_db` |

---
<br/>

## 执行 Migration

### 迁移目录

`migrationsDir` 指向 SQL 迁移文件目录，例如：

```text
project
 |
 |-- migrations
 |      |
 |      |-- 001_create_user.up.sql
 |      |-- 001_create_user.down.sql
 |      |-- 002_add_video.up.sql
 |      |-- 002_add_video.down.sql
 |
 |-- main.go
```

对应配置：

```go
migrationsDir := "./migrations"
```

### 查找命令

```go
migratePath, err := exec.LookPath("migrate")
```

`exec.LookPath` 类似 Shell 中的 `which migrate`，用于确认系统是否安装了 `migrate` CLI。

```bash
brew install golang-migrate
which migrate
```

可能得到：

```text
/opt/homebrew/bin/migrate
```

未找到时返回：

```text
未找到 migrate 命令，请先安装 golang-migrate:
exec: "migrate": executable file not found
```

### 组装并执行命令

```go
cmd := exec.Command(
	migratePath,
	"-path",
	migrationsDir,
	"-database",
	buildMySQLMigrationURL(cfg),
	"up",
)
```

等价于执行：

```bash
migrate \
  -path ./migrations \
  -database "mysql://root:123456@tcp(127.0.0.1:3306)/video_db" \
  up
```

| 参数 | 作用 |
| --- | --- |
| `migratePath` | `migrate` 可执行文件路径 |
| `-path ./migrations` | 指定迁移文件目录 |
| `-database mysql://...` | 指定目标数据库 |
| `up` | 执行尚未应用的升级迁移 |

常用命令：

| 命令 | 作用 |
| --- | --- |
| `up` | 升级数据库 |
| `down` | 回滚迁移 |
| `drop` | 删除全部表，需谨慎使用 |
| `version` | 查看迁移版本 |
| `force` | 强制设置版本，通常用于处理 dirty 状态 |

### 转发输出并处理错误

```go
cmd.Stdout = os.Stdout
cmd.Stderr = os.Stderr
```

这会将子进程的标准输出和错误输出直接显示在当前终端。例如：

```text
1/u create_users
2/u create_video
```

连接失败时可能输出：

```text
Error: dial tcp 127.0.0.1:3306 connection refused
```

`cmd.Run()` 真正启动命令并等待执行完成。成功时返回 `nil`；失败时包装原始错误：

```go
if err := cmd.Run(); err != nil {
	return fmt.Errorf("执行数据库迁移失败: %w", err)
}
```

---
<br/>

## 完整执行流程

```text
main.go
  |
读取 HGMySQLConfig
  |
buildMySQLMigrationURL()
  |
生成 mysql://root:123456@tcp(127.0.0.1:3306)/video_db
  |
exec.LookPath("migrate")
  |
找到 /opt/homebrew/bin/migrate
  |
exec.Command()
  |
migrate -path ./migrations -database xxx up
  |
golang-migrate 读取并执行迁移文件
  |
修改数据库结构
  |
启动服务
```

`golang-migrate` 会在业务数据库中维护 `schema_migrations` 表。下面的状态表示数据库已迁移到版本 `3`，且没有处于失败未清理的 dirty 状态：

| version | dirty |
| --- | --- |
| 3 | false |

自动迁移的优点是减少手工执行 SQL，并确保开发、测试、生产环境使用同一套版本化迁移文件。

### 生产环境注意点

生产环境通常由 CI/CD 中的独立 migration job 先完成迁移，再部署业务服务：

```text
CI/CD Pipeline
     |
 migrate job
     |
 deploy service
```

不建议每个业务实例启动时同时修改数据库：

```text
server1
server2  ---> 同时 migrate，可能发生冲突
server3
```

当前方案还依赖机器预装 `golang-migrate` CLI。若要消除外部命令依赖，可评估直接使用 Go SDK：

```go
import "github.com/golang-migrate/migrate/v4"
```

**核心链路：**

```text
HGMySQLConfig
      |
      v
buildMySQLMigrationURL()
      |
      v
mysql://user:password@tcp(host:port)/database
      |
      v
exec.Command("migrate", "...", "up")
      |
      v
执行 SQL migration
      |
      v
数据库升级
```

***
<br/>

> <h2 id="检查指定索引是否存在">检查指定索引是否存在</h2>

下面的 SQL 用于检查当前数据库中，`video_submissions` 表是否存在指定名称的索引：

```sql
CheckVideoSubmissionStatusTimeIndexSQL = `
SELECT COUNT(*)
FROM information_schema.statistics
WHERE table_schema = DATABASE()
  AND table_name = 'video_submissions'
  AND index_name = ?`
```

典型用途是在创建索引前先检查，避免重复执行 `CREATE INDEX`。

---
<br/>

## 查询条件

### `information_schema.statistics`

`information_schema.statistics` 是 MySQL 提供的索引元数据表，记录索引名称、字段、顺序、唯一性和类型等信息。它提供的信息与下面的命令类似：

```sql
SHOW INDEX FROM video_submissions;
```

### `table_schema = DATABASE()`

`DATABASE()` 返回当前连接使用的数据库：

```sql
SELECT DATABASE();
```

例如当前连接 `video_db`，该条件等价于：

```sql
table_schema = 'video_db'
```

必须限定数据库，因为同一个 MySQL 实例中的多个数据库可能都有 `video_submissions` 表。只按表名查询可能同时匹配：

```text
test_db.video_submissions
video_db.video_submissions
```

### `table_name` 与 `index_name`

```sql
AND table_name = 'video_submissions'
AND index_name = ?
```

`table_name` 限定目标表，`?` 是参数占位符，应由数据库驱动安全绑定：

```go
indexName := "idx_status_submit_time"

db.QueryRow(
	CheckVideoSubmissionStatusTimeIndexSQL,
	indexName,
)
```

---
<br/>

## Go 中的典型用法

```go
func ensureIndex(db *sql.DB) error {
	var count int

	err := db.QueryRow(
		CheckVideoSubmissionStatusTimeIndexSQL,
		"idx_status_submit_time",
	).Scan(&count)
	if err != nil {
		return err
	}

	if count == 0 {
		_, err := db.Exec(`
            CREATE INDEX idx_status_submit_time
            ON video_submissions(status, submit_time)
        `)
		return err
	}

	return nil
}
```

执行流程：

```text
启动服务
    |
检查索引
    |
information_schema.statistics
    |
  存在?
 /     \
是      否
 |       |
继续    CREATE INDEX
```

### 联合索引的计数

`statistics` 通常按**索引字段**保存记录。联合索引：

```sql
CREATE INDEX idx_status_time
ON video_submissions(status, submit_time);
```

可能对应两行：

| index_name | column_name | seq_in_index |
| --- | --- | --- |
| `idx_status_time` | `status` | 1 |
| `idx_status_time` | `submit_time` | 2 |

因此 `COUNT(*)` 可能返回 `2`，不能假设索引存在时一定返回 `1`。判断逻辑应是：

```go
if count > 0 {
	// 索引已存在
}
```

### 为什么程序查询元数据而不是 `SHOW INDEX`

`SHOW INDEX FROM video_submissions` 也能查看索引，但会返回该表的全部索引，需要应用程序继续遍历。查询 `information_schema.statistics` 可以直接在数据库层按数据库、表名和索引名过滤，更适合程序化检查。

**判断关系：**

```text
information_schema.statistics
            |
      MySQL 索引信息
            |
WHERE 当前数据库 + video_submissions + 指定 index_name
            |
         COUNT(*)

0  -> 没有索引
>0 -> 已存在索引
```

***
<br/>

> <h2 id="MySQLinformation_schema">MySQL information_schema</h2>

数据库不仅保存业务数据，还需要保存描述数据库自身结构的数据，这类数据称为**元数据（metadata）**。

例如，`video_submissions` 中的用户 ID、状态是业务数据；表有哪些字段、主键和索引，则属于元数据。MySQL 主要通过 `information_schema` 提供这类信息。

---
<br/>

## MySQL 系统库

执行：

```sql
SHOW DATABASES;
```

通常可以看到：

```text
information_schema
mysql
performance_schema
sys
video_db
```

| 数据库 | 用途 |
| --- | --- |
| `information_schema` | 提供数据库结构元数据 |
| `mysql` | 保存用户、权限和部分系统配置 |
| `performance_schema` | 收集性能监控数据 |
| `sys` | 提供更易读的性能分析视图 |
| `video_db` | 保存业务数据和 migration 版本表 |

它们之间的关系：

```text
                 MySQL
                   |
       -------------------------
       |                       |
 information_schema       video_db
       |                       |
  描述数据库结构              业务数据
       |
 tables
 columns
 statistics
```

---
<br/>

## `information_schema.statistics`

该表可以理解为 MySQL 维护的“索引登记表”，记录当前实例中各数据库、表和字段的索引信息。

```sql
SELECT *
FROM information_schema.statistics
LIMIT 5;
```

常用字段：

| 字段 | 含义 |
| --- | --- |
| `TABLE_SCHEMA` | 数据库名 |
| `TABLE_NAME` | 表名 |
| `NON_UNIQUE` | 是否为非唯一索引 |
| `INDEX_NAME` | 索引名 |
| `COLUMN_NAME` | 索引字段名 |
| `SEQ_IN_INDEX` | 字段在联合索引中的顺序 |
| `INDEX_TYPE` | 索引类型 |

例如：

```sql
CREATE INDEX idx_status_time
ON video_submissions(status, submit_time);
```

会形成类似记录：

| TABLE_NAME | INDEX_NAME | COLUMN_NAME | SEQ_IN_INDEX |
| --- | --- | --- | --- |
| `video_submissions` | `idx_status_time` | `status` | 1 |
| `video_submissions` | `idx_status_time` | `submit_time` | 2 |

常见用途包括索引存在性检查、数据库巡检、迁移工具读取现有结构，以及辅助排查缺失索引。自动创建索引前仍需评估建索引带来的锁、资源和写入开销。

---
<br/>

## 常用元数据表

| 表 | 主要内容 | 常见场景 |
| --- | --- | --- |
| `information_schema.tables` | 表信息 | 判断表是否存在、统计容量 |
| `information_schema.columns` | 字段信息 | 判断字段是否存在 |
| `information_schema.statistics` | 索引信息 | 判断索引是否存在 |
| `information_schema.key_column_usage` | 键与字段关系 | 查询主键、外键、唯一键字段 |
| `information_schema.table_constraints` | 表约束 | 查询主键、外键、唯一约束 |
| `information_schema.views` | 视图信息 | 查看视图定义 |
| `information_schema.routines` | 存储过程和函数 | 查询 routine 元数据 |

### 判断表是否存在

```sql
SELECT COUNT(*)
FROM information_schema.tables
WHERE table_schema = DATABASE()
  AND table_name = 'video_submissions';
```

### 查看表容量

```sql
SELECT
    table_name,
    data_length / 1024 / 1024 AS data_mb
FROM information_schema.tables
WHERE table_schema = DATABASE();
```

### 判断字段是否存在

```sql
SELECT COUNT(*)
FROM information_schema.columns
WHERE table_schema = DATABASE()
  AND table_name = 'video_submissions'
  AND column_name = 'cover_url';
```

确认字段不存在后，才执行：

```sql
ALTER TABLE video_submissions
ADD COLUMN cover_url VARCHAR(255);
```

### 查询键和约束

```sql
SELECT *
FROM information_schema.key_column_usage
WHERE table_schema = DATABASE();
```

```sql
SELECT *
FROM information_schema.table_constraints
WHERE table_schema = DATABASE();
```

### 查询视图与存储过程

```sql
SELECT *
FROM information_schema.views
WHERE table_schema = DATABASE();
```

```sql
SELECT *
FROM information_schema.routines
WHERE routine_schema = DATABASE();
```

---
<br/>

## 其他系统库

### `mysql`

`mysql` 库保存用户和权限等系统数据。例如：

```sql
SELECT user, host
FROM mysql.user;
```

可能得到：

```text
root localhost
app  %
```

`mysql.db`、`mysql.tables_priv` 等表记录数据库和表级权限。

### `performance_schema`

`performance_schema` 用于采集 SQL 执行、锁等待、I/O 等性能数据。例如：

```sql
SELECT *
FROM performance_schema.events_statements_history;
```

### `sys`

`sys` 基于 `performance_schema` 提供更易读的诊断视图。例如：

```sql
SELECT *
FROM sys.statement_analysis;
```

### 与 migration 的关系

`golang-migrate` 在业务数据库中维护 `schema_migrations`；MySQL 则通过 `information_schema` 描述该业务数据库中的表、字段和索引：

```text
video_db
 |
 +-- video_submissions
 +-- video_files
 +-- schema_migrations

information_schema
 |
 +-- tables
 +-- columns
 +-- statistics
```


***
<br/><br/><br/>

> <h2 id="Go后端常用代码与数据库基础">Go 后端常用代码与数据库基础</h2>

本文整理 Go 后端开发中常见的数据库迁移、SQL 排序与分页、错误判断、Redis 缓存、配置加载和 MySQL 连接配置。

***
<br/>

> <h3 id="schema_migrations迁移版本表">schema_migrations 迁移版本表</h3>

```go
var version uint
var dirty bool

err := db.QueryRow(`
    SELECT version, dirty
    FROM schema_migrations
    LIMIT 1
`).Scan(&version, &dirty)
```

`schema_migrations` **不是 MySQL 或 PostgreSQL 的系统表**，而是 `golang-migrate` 等数据库迁移工具创建的版本管理表。

| 字段 | 含义 |
| --- | --- |
| `version` | 当前数据库已执行到的 migration 版本 |
| `dirty` | 当前版本是否处于迁移失败或未完成状态 |

例如：

```text
schema_migrations

version | dirty
----------------
3       | false
```

表示数据库结构已升级到第 3 版，并且迁移状态正常。

### version 的作用

项目中的迁移文件按版本演进：

```text
v1
 |
 + 创建 users 表

v2
 |
 + 增加 email 字段

v3
 |
 + 创建 video 表
```

迁移工具通过 `version` 判断数据库当前执行到哪个版本，以及下一步需要执行哪些迁移。

### dirty 的含义

假设 migration 13 包含多项操作，其中一部分成功、一部分失败：

```sql
ALTER TABLE videos
ADD COLUMN duration INT;
```

迁移表可能记录为：

```text
version = 13
dirty   = true
```

`dirty = true` 表示数据库可能处于不完整的结构状态，需要先人工检查和修复，不能直接继续升级。

### 常见迁移工具

| 工具 | 默认版本表 |
| --- | --- |
| `golang-migrate` | `schema_migrations` |
| `goose` | `goose_db_version` |
| Flyway | `flyway_schema_history` |
| Liquibase | `databasechangelog` |

`golang-migrate` 创建的表结构通常类似：

```sql
CREATE TABLE schema_migrations (
    version BIGINT NOT NULL,
    dirty BOOLEAN NOT NULL
);
```

### 初始化迁移

迁移目录示例：

```text
migrations/
├── 000001_create_user.up.sql
├── 000001_create_user.down.sql
├── 000002_create_video.up.sql
└── 000002_create_video.down.sql
```

执行：

```bash
migrate \
  -path ./migrations \
  -database "mysql://root:xxx@tcp(localhost:3306)/video" \
  up
```

如果数据库中没有 `schema_migrations`，通常说明迁移工具尚未初始化、迁移命令未执行，或项目的数据库初始化流程缺失。**不建议手动创建并随意插入版本号**，否则迁移工具无法知道哪些 SQL 已真实执行。

### 错误处理

`sql.ErrNoRows` 只表示表存在但查询没有返回记录；表不存在时，MySQL 会返回类似错误：

```text
Error 1146: Table 'xxx.schema_migrations' doesn't exist
```

```go
err := db.QueryRow(`
    SELECT version, dirty
    FROM schema_migrations
    LIMIT 1
`).Scan(&version, &dirty)

switch {
case errors.Is(err, sql.ErrNoRows):
    // 迁移表存在，但没有版本记录
case err != nil:
    // 包括表不存在、连接失败、权限不足等错误
    return err
}
```

生产发布流程通常为：

```text
代码提交
   |
   v
migration 文件
   |
   v
CI/CD 执行 migrate up
   |
   v
数据库升级
   |
   v
schema_migrations 记录版本
   |
   v
启动应用服务
```

**核心结论：`schema_migrations` 是数据库发布流程中的结构版本表，不是业务表。**

***
<br/>

> <h3 id="视频标签游标分页SQL">视频标签游标分页 SQL</h3>

典型场景：用户点击某个标签，查询该标签下审核中或已发布的视频，并使用 Cursor Pagination 分页。

```sql
SELECT
    vs.submission_id,
    vs.user_id,
    vs.title,
    vs.cover_url,
    vs.category,
    vs.video_type,
    vf.video_id,
    vf.file_path,
    vf.file_name,
    vf.file_size,
    vf.mime_type,
    vf.part_number
FROM video_tags vt
INNER JOIN video_files vf
    ON vf.video_id = vt.video_id
INNER JOIN video_submissions vs
    ON vs.submission_id = vf.submission_id
WHERE vt.tag_name = ?
  AND vs.status IN ('reviewing', 'published')
  AND vf.part_number = 1
  AND (
      vs.submit_time < ?
      OR (
          vs.submit_time = ?
          AND vs.submission_id < ?
      )
  )
ORDER BY vs.submit_time DESC, vs.submission_id DESC
LIMIT ?;
```

数据关系：

```text
video_submissions
        |
        | submission_id
        v
video_files
        |
        | video_id
        v
video_tags
```

- `video_tags`：根据 `tag_name` 找到视频 ID。
- `video_files`：根据 `video_id` 获取文件，并通过 `part_number = 1` 避免一个多分片视频返回多条。
- `video_submissions`：获取投稿主体信息并过滤业务状态。

### Cursor 分页

核心条件：

```sql
AND (
    vs.submit_time < ?
    OR (
        vs.submit_time = ?
        AND vs.submission_id < ?
    )
)
```

排序规则：

```sql
ORDER BY vs.submit_time DESC, vs.submission_id DESC
```

假设第一页数据为：

| `submit_time` | `submission_id` |
| --- | ---: |
| 10:10 | 100 |
| 10:09 | 99 |

下一页游标为 `10:09|99`。查询下一页时，获取提交时间更早的数据；如果提交时间相同，则获取 ID 更小的数据。第二排序字段保证排序结果唯一、稳定，避免重复或漏数。

相比深分页：

```sql
LIMIT 20 OFFSET 10000000
```

`OFFSET` 需要扫描并丢弃前面的记录，页数越深越慢；Keyset/Seek Pagination 可利用有序索引从游标位置继续读取。

### 建议索引

```sql
CREATE INDEX idx_tag_video
ON video_tags(tag_name, video_id);

CREATE INDEX idx_video_files_query
ON video_files(video_id, part_number, submission_id);

CREATE INDEX idx_video_feed
ON video_submissions(status, submit_time DESC, submission_id DESC);
```

索引是否真正生效仍取决于数据分布、连接顺序和 MySQL 执行计划，应使用 `EXPLAIN` 或 `EXPLAIN ANALYZE` 验证，不能只根据字段顺序下结论。

### 亿级数据架构

大规模流量下，即使索引正确，也不应让 MySQL 持续承担热门标签的多表 Feed 查询。常见演进方案：

1. 建立标签 Feed 索引表，只保存排序和定位所需字段。
2. Redis 缓存热门标签的最新一批 `submission_id`。
3. 根据 ID 批量查询视频详情。
4. 数据规模继续增长时，对详情表分库分表。
5. 将 Feed 查询独立为服务。

```sql
CREATE TABLE video_tag_feed (
    id BIGINT,
    tag_id BIGINT,
    submission_id BIGINT,
    submit_time DATETIME,
    PRIMARY KEY (tag_id, submit_time, submission_id)
);
```

```sql
SELECT submission_id
FROM video_tag_feed
WHERE tag_id = ?
  AND submit_time < ?
ORDER BY submit_time DESC, submission_id DESC
LIMIT 20;
```

整体架构：

```text
                 用户
                  |
             API Gateway
                  |
             Feed Service
                  |
        +---------+---------+
        |                   |
      Redis           Feed Index
        |                   |
        +---------+---------+
                  |
            MySQL Sharding
                  |
       video_submissions / video_files
```

**核心结论：当前 SQL 属于业务查询层；高流量系统通常演进为“Feed 索引层 + 缓存层 + 详情存储层”。**

***
<br/>

> <h3 id="strings.SplitN">strings.SplitN</h3>

```go
parts := strings.SplitN(cursor, "|", 2)
```

`SplitN(s, sep, n)` 按分隔符切割字符串，最多返回 `n` 个元素。`n = 2` 表示只在第一个 `|` 处分割，右侧剩余内容全部保留。

```go
cursor := "abc|123|456"

fmt.Println(strings.Split(cursor, "|"))
// [abc 123 456]

fmt.Println(strings.SplitN(cursor, "|", 2))
// [abc 123|456]
```

常用于 Cursor、Token 或协议字段等“左侧格式固定，右侧内容可能继续包含分隔符”的场景。

### n 参数

| `n` | 结果 |
| ---: | --- |
| `0` | 返回 `nil` |
| `1` | 不切割，返回原字符串 |
| `2` | 最多返回两部分 |
| `< 0` | 切割所有匹配项，效果与 `strings.Split` 相同 |

```go
strings.SplitN("a|b|c", "|", 0)  // nil
strings.SplitN("a|b|c", "|", 1)  // [a|b|c]
strings.SplitN("a|b|c", "|", 2)  // [a b|c]
strings.SplitN("a|b|c", "|", -1) // [a b c]
```

### Cursor 解析示例

```go
func parseCursor(cursor string) (int64, string, error) {
    parts := strings.SplitN(cursor, "|", 2)
    if len(parts) != 2 {
        return 0, "", fmt.Errorf("invalid cursor")
    }

    offset, err := strconv.ParseInt(parts[0], 10, 64)
    if err != nil {
        return 0, "", fmt.Errorf("parse cursor offset: %w", err)
    }

    return offset, parts[1], nil
}
```

输入 `12345|abc|def` 后，得到 `offset = 12345`、`value = "abc|def"`。

**核心结论：`strings.SplitN(cursor, "|", 2)` 只拆分第一个分隔符，适合解析两段式结构。**

***
<br/>

> <h3 id="ORDER-BY与LIMIT">ORDER BY 与 LIMIT</h3>

```sql
SELECT
    tag_id,
    id,
    name,
    sort_order,
    status,
    created_at,
    updated_at
FROM bilibili_douga_tags
WHERE is_deleted = 0
  AND status = 1
ORDER BY sort_order ASC, id ASC
LIMIT 100;
```

业务含义：查询未删除且已启用的标签，先按运营排序权重升序排列；权重相同时按 ID 升序排列，最后只返回前 100 条。

### 多字段排序

```sql
ORDER BY sort_order ASC, id ASC
```

- `ASC`：升序，从小到大。
- `DESC`：降序，从大到小。
- 先按 `sort_order` 排序；值相同时再按 `id` 排序。
- `id` 作为唯一的第二排序字段，可使结果顺序稳定、确定。

| `id` | `name` | `sort_order` |
| ---: | --- | ---: |
| 1 | 游戏 | 10 |
| 2 | 动漫 | 20 |
| 3 | 科技 | 10 |
| 4 | 电影 | 20 |
| 5 | 音乐 | 10 |

排序结果：`1、3、5、2、4`。

### LIMIT

```sql
LIMIT 100
```

只返回排序后的前 100 条，避免一次返回过多数据，降低数据库、网络和前端渲染压力。

```sql
LIMIT 100, 100
```

表示跳过前 100 条，再取 100 条。数据量较大时应避免深 `OFFSET`，改用与排序字段一致的游标条件。例如按 `(sort_order, id)` 分页时，不能只使用 `id > ?`，而应同时携带两个排序值：

```sql
WHERE is_deleted = 0
  AND status = 1
  AND (
      sort_order > ?
      OR (sort_order = ? AND id > ?)
  )
ORDER BY sort_order ASC, id ASC
LIMIT 100;
```

### 联合索引

```sql
CREATE INDEX idx_tag_list
ON bilibili_douga_tags(is_deleted, status, sort_order, id);
```

该索引与等值过滤字段和排序字段的顺序对应，但仍需结合字段基数和执行计划验证。

***
<br/>

> <h3 id="识别MySQL-Duplicate-Key">识别 MySQL Duplicate Key</h3>

```go
func isDuplicateKeyError(err error) bool {
    if err == nil {
        return false
    }

    var mysqlErr *mysql.MySQLError
    if errors.As(err, &mysqlErr) {
        return mysqlErr.Number == 1062
    }

    return strings.Contains(err.Error(), "Duplicate entry")
}
```

该函数判断 MySQL 操作是否因唯一键冲突失败。MySQL 错误码 `1062` 对应 `ER_DUP_ENTRY`：

```text
Error 1062: Duplicate entry 'test@gmail.com' for key 'users.email'
```

### errors.As

`errors.As(err, &mysqlErr)` 会沿错误链查找 `*mysql.MySQLError`，即使底层错误被 `%w` 包装也能提取：

```go
return fmt.Errorf("insert user failed: %w", err)
```

```text
insert user failed
        |
        v
*mysql.MySQLError
        |
        v
Number = 1062
```

相比直接类型断言，`errors.As` 更适合现代 Go 的错误包装机制。

### 字符串兜底

```go
strings.Contains(err.Error(), "Duplicate entry")
```

当某些库丢失底层错误类型但保留错误文本时，可以兜底识别；不过字符串判断容易受驱动、语言和错误格式变化影响，**应优先使用错误类型和错误码**。

### 业务用途

重复键通常属于可预期的业务冲突，而不一定是服务器故障：

```go
err := createOrder()
if isDuplicateKeyError(err) {
    // 幂等请求：返回已存在的订单
    return existingOrder
}
if err != nil {
    return fmt.Errorf("create order: %w", err)
}
```

典型用途：幂等接口、防重复提交、唯一业务号和并发注册。


***
<br/>

> <h3 id="mysql.Config.FormatDSN">mysql.Config.FormatDSN</h3>

```go
cfg := mysql.Config{
    User:      "root",
    Passwd:    "123456",
    Net:       "tcp",
    Addr:      "127.0.0.1:3306",
    DBName:    "test",
    ParseTime: true,
    Loc:       time.Local,
    Params: map[string]string{
        "charset": "utf8mb4",
    },
}

dsn := cfg.FormatDSN()
```

`FormatDSN()` 将结构化的 MySQL 连接参数转换成驱动可识别的 DSN（Data Source Name）：

```text
root:123456@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=true
```

| 配置 | 含义 |
| --- | --- |
| `User` | 数据库用户名 |
| `Passwd` | 数据库密码 |
| `Net` | 网络类型，如 `tcp` 或 `unix` |
| `Addr` | MySQL 地址 |
| `DBName` | 数据库名 |
| `ParseTime` | 将 `DATE`/`DATETIME` 扫描为 `time.Time` |
| `Loc` | 解析时间时使用的时区 |
| `Params` | 额外 DSN 参数，例如 `charset=utf8mb4` |

使用 `mysql.Config` 比手动拼接 DSN 更清晰，也能正确处理驱动要求的转义规则。

### 完整连接流程

```go
func openMySQL(ctx context.Context, cfg MySQLConfig) (*sql.DB, error) {
    mysqlCfg := mysql.Config{
        User:      cfg.User,
        Passwd:    cfg.Password,
        Net:       "tcp",
        Addr:      net.JoinHostPort(cfg.Host, strconv.Itoa(cfg.Port)),
        DBName:    cfg.Database,
        ParseTime: true,
        Loc:       time.Local,
        Params: map[string]string{
            "charset": "utf8mb4",
        },
    }

    db, err := sql.Open("mysql", mysqlCfg.FormatDSN())
    if err != nil {
        return nil, fmt.Errorf("open mysql: %w", err)
    }

    if err := db.PingContext(ctx); err != nil {
        db.Close()
        return nil, fmt.Errorf("ping mysql: %w", err)
    }

    return db, nil
}
```

`sql.Open()` 通常只验证参数并创建连接池句柄，不保证数据库当前可连接；启动阶段应使用 `PingContext()` 验证连接。

```text
配置文件 / 环境变量
        |
        v
mysql.Config
        |
        v
FormatDSN()
        |
        v
DSN 字符串
        |
        v
sql.Open()
        |
        v
连接池句柄
        |
        v
PingContext()
        |
        v
MySQL 可用
```

**核心结论：`FormatDSN()` 负责规范生成连接字符串，`sql.Open()` 创建连接池句柄，`PingContext()` 才用于验证实际连接。**



***
<br/><br/><br/>

> <h1 id="ClickHouse-弹幕异步写入">ClickHouse 弹幕异步写入</h1>

## 核心代码

```go
query := fmt.Sprintf(
    "INSERT INTO %s.%s SETTINGS async_insert=1, wait_for_async_insert=1 FORMAT JSONEachRow",
    c.config.Database,
    c.config.DanmakuHistoryTable,
)

return c.execute(
    ctx,
    c.config.WriteTimeout,
    query,
    body.Bytes(),
    nil,
)
```

这段代码构造 ClickHouse 的 `INSERT` 查询，将 `body.Bytes()` 中按 `JSONEachRow` 组织的弹幕数据发送到指定表，并启用服务端异步插入。`wait_for_async_insert=1` 要求等待异步插入的 flush/处理结果后再返回，但不能泛化为所有副本都已完成耐久化。

假设配置为：

```yaml
clickhouse:
  database: mlc
  danmaku_history_table: video_danmaku_history
```

最终查询：

```sql
INSERT INTO mlc.video_danmaku_history
SETTINGS async_insert=1, wait_for_async_insert=1
FORMAT JSONEachRow
```

---
<br/>

## `INSERT INTO` 与 `JSONEachRow`

`INSERT INTO mlc.video_danmaku_history` 指定目标表。对比普通 `VALUES` 插入：

```sql
INSERT INTO user
    (id, name)
VALUES
    (1, '张三');
```

本文不使用 `VALUES`，而由 `FORMAT JSONEachRow` 指定输入格式。

典型数据：

```json
{"video_id":"1001","user_id":"2001","content":"哈哈哈","timestamp":1720000000}
{"video_id":"1001","user_id":"2002","content":"666","timestamp":1720000001}
{"video_id":"1001","user_id":"2003","content":"来了来了","timestamp":1720000002}
```

**一行一个 JSON 对象，每行对应一条记录。**

```text
一行 JSON
↓
一条数据库记录
```

普通 JSON 可能是数组：

```json
[
    {
        "video_id": "1001",
        "user_id": "2001",
        "content": "哈哈哈"
    },
    {
        "video_id": "1001",
        "user_id": "2002",
        "content": "666"
    }
]
```

而 `JSONEachRow` 是多行 JSON：

```json
{"video_id":"1001","user_id":"2001","content":"哈哈哈"}
{"video_id":"1001","user_id":"2002","content":"666"}
```

需要注意：`JSONEachRow` 只规定输入数据格式；本文封装中，`query` 与 HTTP body 分离，SQL/query 通过 `query` 传递，实际行数据通过 `body.Bytes()` 传递。不能把“格式要求”概括成 ClickHouse 在所有客户端中都强制某个 HTTP body 位置。

---
<br/>

> <h2 id="异步插入与等待结果">异步插入与等待结果</h2>

### `async_insert=1`

它开启 ClickHouse 的服务端异步插入机制：服务端可以先把数据放入异步缓冲区，之后再批量写入目标存储。

普通插入：

```text
Go服务
   ↓
INSERT
   ↓
ClickHouse
   ↓
处理数据
   ↓
返回
```

开启服务端缓冲后：

```text
Go服务
   ↓
INSERT
   ↓
ClickHouse
   ↓
异步 INSERT Buffer
   ↓
后面批量写入 MergeTree
```

目的主要是减少大量小批量 `INSERT` 对 ClickHouse 的压力。

如果弹幕服务每秒产生 `1000条弹幕`，每条都单独写入会产生大量请求：

```text
INSERT
INSERT
INSERT
INSERT
INSERT
INSERT
...
```

更合理的路径是：

```text
1000条
 ↓
Go程序攒一批
 ↓
一次 INSERT
 ↓
ClickHouse
 ↓
批量处理
```

### `wait_for_async_insert=0`

大致流程：

```text
Go
 ↓
发送数据
 ↓
ClickHouse 收到
 ↓
进入异步缓冲
 ↓
马上告诉 Go：OK
 ↓
Go继续执行
```

此时 **ClickHouse 收到不等于数据已经完成后续 flush 或最终存储落盘**。

### `wait_for_async_insert=1`

大致流程：

```text
Go
 ↓
发送数据
 ↓
ClickHouse
 ↓
异步 INSERT Buffer
 ↓
ClickHouse处理
 ↓
成功
 ↓
返回 Go：OK
```

`async_insert=1, wait_for_async_insert=1` 使用服务端缓冲并等待 flush 结果；不是 Go 启动后台任务后不等待。上图表示成功分支，flush 失败也应返回错误，成功不等于所有副本或下游都已持久化。

### 批量写入效果

ClickHouse 擅长批量写入。例如 10000 条弹幕：

```text
INSERT
 ├── 弹幕1
 ├── 弹幕2
 ├── 弹幕3
 ├── ...
 └── 弹幕10000
```

通常优于：

```text
INSERT 1
INSERT 2
INSERT 3
...
INSERT 10000
```

整体写入链路：

```text
弹幕服务
   │
   │ 批量JSON
   ▼
ClickHouse
   │
   │ async_insert
   ▼
异步缓冲区
   │
   ▼
批量写入
   │
   ▼
video_danmaku_history
```

---
<br/>

## `query`、`body.Bytes()` 与 `execute`

`fmt.Sprintf` 用配置替换两个 `%s`。当 `database := "mlc"`、`table := "video_danmaku_history"` 时，得到：

```sql
INSERT INTO mlc.video_danmaku_history SETTINGS async_insert=1, wait_for_async_insert=1 FORMAT JSONEachRow
```

### `body.Bytes()`

`body` 可以理解为 `bytes.Buffer`：

```go
var body bytes.Buffer

body.WriteString(`{"video_id":"1001","content":"666"}`)
body.WriteByte('\n')

body.WriteString(`{"video_id":"1002","content":"哈哈"}`)
body.WriteByte('\n')
```

`body.Bytes()` 取出其中的 `[]byte`，即要发送给 ClickHouse 的实际数据：

```text
{"video_id":"1001","content":"666"}
{"video_id":"1002","content":"哈哈"}
```

所以请求可视化为：

```text
┌──────────────────────────────────────────┐
│ ClickHouse INSERT 请求                   │
│                                          │
│ SQL：                                    │
│ INSERT INTO mlc.video_danmaku_history    │
│ SETTINGS async_insert=1                  │
│          wait_for_async_insert=1          │
│ FORMAT JSONEachRow                       │
│                                          │
│ Body：                                   │
│ {"video_id":"1001","content":"哈哈哈"}   │
│ {"video_id":"1001","content":"666"}      │
│ {"video_id":"1001","content":"666666"}   │
└──────────────────────────────────────────┘
```

### `c.execute(...)` 的已知与未知

从调用点只能推断参数大致代表：

```text
execute(
    ctx,                    // 上下文
    WriteTimeout,           // 写入超时时间
    query,                  // SQL
    body.Bytes(),           // SQL对应的数据
    nil                     // 其他可选参数
)
```

但 `execute` 的实现没有提供，因而不能确定：

- `nil` 的具体含义；它可能是请求选项、额外参数或其他可选值；
- `WriteTimeout` 是如何实现的，是否通过 `context.WithTimeout`、HTTP client timeout 或其他机制；
- query/body 是通过 URL 参数、请求体拼装还是客户端库的其他接口发送。

常见职责可能包括：

```text
创建 HTTP 请求
       ↓
连接 ClickHouse
       ↓
发送 query
       ↓
发送 body
       ↓
等待 ClickHouse 返回
       ↓
判断成功/失败
```

### `ctx` 与取消边界

`ctx` 是 `context.Context`。下图保留原文，表示取消意图，并非服务端未落库的保证；实际效果还取决于 `execute` 是否传递上下文。

```text
用户请求
   ↓
HTTP Handler
   ↓
写 ClickHouse
   ↓
用户断开连接
   ↓
ctx取消
   ↓
ClickHouse写入操作可以被取消
```

但 `ctx` 取消**不保证 ClickHouse 服务端一定没有接收或落库**：取消可能发生在请求已发送、服务端已入缓冲或已处理之后。因此重试策略仍需考虑重复写入、业务幂等和 ClickHouse 的实际确认边界。

`c.config.WriteTimeout` 例如可能配置为：

```yaml
write_timeout: 5s
```

至于它是否真的限制为 `5秒`，必须查看 `execute` 实现和 HTTP 客户端配置，不能仅由字段名断言。

---
<br/>

## 完整数据流

下面的 `Statistic Consumer` 架构是原文根据上下文做出的假设；仅凭当前代码片段无法确认真实 consumer 名称、消息链路或完整架构。

```text
                 弹幕
                  │
                  ▼
             Go 弹幕服务
                  │
                  ▼
               Kafka
                  │
                  ▼
          Statistic Consumer
                  │
                  ▼
       组装/批量构造弹幕数据
                  │
                  ▼
             body.Buffer
                  │
        ┌─────────┴─────────┐
        │                   │
        │ body.Bytes()      │
        ▼                   │
  {"video_id":"1001"...}    │
  {"video_id":"1002"...}    │
  {"video_id":"1003"...}    │
        │                   │
        └─────────┬─────────┘
                  ▼
             c.execute()
                  │
                  ▼
        ┌───────────────────┐
        │    ClickHouse     │
        │                   │
        │ INSERT INTO       │
        │ mlc.video_        │
        │ danmaku_history   │
        │                   │
        │ async_insert=1    │
        │ wait_for_async... │
        │ JSONEachRow       │
        └─────────┬─────────┘
                  │
                  ▼
             异步写入
                  │
                  ▼
      video_danmaku_history
```

| 代码 | 含义 |
| --- | --- |
| `INSERT INTO mlc.video_danmaku_history` | 写入哪个表 |
| `FORMAT JSONEachRow` | body 中数据的格式 |
| `async_insert=1` | ClickHouse 使用异步插入缓冲 |
| `wait_for_async_insert=1` | 等待异步插入处理结果后返回 |
