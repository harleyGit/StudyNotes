- [视频状态统计](#视频状态统计)
	- [WHERE 与 GROUP BY](#WHERE-与-GROUP-BY)
	- [Redis 与 MySQL 统计的区别](#Redis-与-MySQL-统计的区别)
	- [亿级数据优化](#亿级数据优化)





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
