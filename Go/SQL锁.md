- **[SQL锁](#SQL锁)**
	- [FOR UPDATE](#FOR_UPDATE)   
	- [共享锁](#共享锁)   
	- [NOWAIT](#NOWAIT)     
	- [SKIP LOCKED跳过已经被锁住的数据](#SKIP_LOCKED跳过已经被锁住的数据)    
	- [表锁(LOCK TABLES)](#表锁_LOCK_TABLES)     
	- [意向锁 InnoDB自动加](#意向锁_InnoDB自动加)    
	- [(Gap Lock间隙锁)](#Gap_Lock间隙锁)   
	- [(Next Key Lock)](#Next-Key_Lock)    
	- [插入意向锁(Insert Intention Lock)](#插入意向锁_Insert_Intention_Lock)      
	- [(乐观锁)不是数据库锁](#乐观锁_不是数据库锁)     
	- [实际项目最常用的是哪些？](#实际项目最常用的是哪些？)

<br/><br/><br/>

***
<br/>

> <h1 id= "SQL锁">SQL锁</h1>

### 在 MySQL / InnoDB 里，`FOR UPDATE` 属于 `行级锁（Row Lock）`中的一种

类似的锁还有很多，可以从：
* SQL语法层
* 锁类型层
* InnoDB实现层

三个角度理解

---
***
<br/><br/><br/>
> <h2 id="FOR_UPDATE">FOR UPDATE</h2> 

```sql
SELECT * FROM users
WHERE user_id = 1
FOR UPDATE;
```

特点：

| 特性     | 说明               |
| ------ | ---------------- |
| 排他锁    | 别人不能修改           |
| 会阻塞    | 其他 FOR UPDATE 要等 |
| 用于写前读取 | 最常见              |

***
<br/><br/><br/>
> <h2 id="共享锁">共享锁</h2>
### `LOCK IN SHARE MODE（MySQL 5.x）`

**MySQL 5.x：**

```sql
SELECT * FROM users
WHERE user_id = 1
LOCK IN SHARE MODE;
```

<br/>

**MySQL 8：**

```sql
SELECT * FROM users
WHERE user_id = 1
FOR SHARE;
```

这是：**共享锁（S锁）**

意思：**`我只读，但别人不能改。`**
- **特点**
	- 允许：**多个事务同时读**
	- **不能 UPDATE**
	- **不能 DELETE**
---
<br/>

- **举例**

**事务A：**

```sql
SELECT * FROM users
WHERE id = 1
FOR SHARE;
```
<br/>

**事务B：**

```sql
SELECT * FROM users
WHERE id = 1
FOR SHARE;
```

可以同时成功,但：

```sql
UPDATE users ...
```

会被阻塞。


***
<br/><br/><br/>
> <h2 id="NOWAIT">NOWAIT</h2>

# 二、NOWAIT

- 有时候：**你不想等待锁。**
- 希望：**如果被锁住, 直接失败**

**可以：**

```sql
SELECT * FROM users
WHERE id = 1
FOR UPDATE NOWAIT;
```

<br/>

- **行为:**

**如果别人已经锁住, 立即报错：**

```text
Lock wait timeout / NOWAIT
```

不会卡住。

---

## 适合

* 秒杀
* 抢单
* 高并发任务

***
<br/><br/><br/>
> <h2 id="SKIP_LOCKED跳过已经被锁住的数据"> SKIP LOCKED跳过已经被锁住的数据</h2>


### **SKIP LOCKED**这是高并发里非常经典的。

```sql
SELECT * FROM jobs
WHERE status = 'waiting'
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

意思：**跳过已经被锁住的数据**

---

## 场景

多个 worker 抢任务。

---

- **worker1：**

锁住：**job1**

<br/>

- **worker2：**

自动跳过： **‌job1**

去拿：**`job2`**

---

## 非常适合

| 场景    | 是否常用 |
| ----- | ---- |
| 消息队列  | ✓    |
| 任务调度  | ✓    |
| 秒杀订单  | ✓    |
| 分布式消费 | ✓    |

***
<br/><br/><br/>
> <h2 id="表锁_LOCK_TABLES">表锁（LOCK TABLES） </h2>

表锁（LOCK TABLES）,不是行锁。而是：**整张表锁住**

---

## 写锁

```sql
LOCK TABLES users WRITE;
```

效果：

```text
别人不能读
别人不能写
```

---

## 读锁

```sql
LOCK TABLES users READ;
```

效果：

```text
别人只能读
不能写
```

---

- **缺点**
	- 并发性能很差。

现在：**大部分业务很少用,因为 InnoDB 行锁更好**。

---

***
<br/><br/><br/>
> <h2 id="意向锁_InnoDB自动加">意向锁（InnoDB自动加）</h2>

**意向锁（InnoDB自动加）,这个开发者通常看不到。**

比如：

```sql
FOR UPDATE
```

底层会自动加：
<br/>

- **IX锁（意向排他锁）**

### 意思：**我要锁某些行**,这是：**表级“声明”**,方便数据库快速判断锁冲突。

***
<br/><br/><br/>
> <h2 id="Gap_Lock间隙锁">Gap Lock（间隙锁）</h2>
这是：**MySQL 最容易让人迷惑的锁**

---

## 举例

表中：

```text
1
3
5
10
```

事务A：

```sql
SELECT * FROM users
WHERE id BETWEEN 3 AND 10
FOR UPDATE;
```

不仅锁：

```text
3
5
10
```

还锁：

```text
3~10之间的“空隙”
```

于是：

别人不能插入：

```text
4
6
7
8
```

---

### 为什么？

防止： **幻读（Phantom Read）**

***
<br/><br/><br/>
> <h2 id="Next-Key_Lock">Next-Key Lock</h2> 

这是：**行锁 + Gap Lock**,InnoDB 默认 RR 隔离级别下大量使用。

本质：

```text
锁记录
+
锁间隙
```

这是 MySQL 防幻读的核心。

***
<br/><br/><br/>
> <h2 id="插入意向锁_Insert_Intention_Lock">插入意向锁（Insert Intention Lock）</h2>

**插入数据时自动产生,** 比如：

```sql
INSERT INTO users(id) VALUES(6);
```

数据库会：

```text
声明：
我要插入这个位置
```

避免多个插入完全互斥。

***
<br/><br/><br/>
> <h2 id="乐观锁_不是数据库锁">乐观锁（不是数据库锁）</h2>
这和 `FOR UPDATE` 对应。

---

### 悲观锁,认为：`别人会抢, 先锁`

**典型：**

```sql
FOR UPDATE
```

---

### 乐观锁,认为：`冲突很少,提交时检查`

**常见实现**

version 字段：

```sql
UPDATE users
SET balance = 90,
    version = version + 1
WHERE id = 1
AND version = 5;
```

如果：

```text
version 不匹配
```

说明别人改过,更新失败。

***
<br/><br/><br/>
> <h2 id="实际项目最常用的是哪些？">实际项目最常用的是哪些？</h2>
**真正 Go / Java / PHP 项目里：**

最常用的是：

| 锁           | 常用度       |
| ----------- | --------- |
| FOR UPDATE  | ★★★★★     |
| 乐观锁 version | ★★★★★     |
| SKIP LOCKED | ★★★★      |
| FOR SHARE   | ★★★       |
| Gap Lock    | ★★★（经常被坑） |
| 表锁          | ★         |
| LOCK TABLES | ★         |

***
<br/>

## **Go 项目里推荐策略**
- 普通业务推荐：**乐观锁,性能高。**

***
- 高一致性业务推荐：**`FOR UPDATE`**,比如：
	* 钱
	* 库存
	* 订单

---

- 高并发任务队列,推荐：**`SKIP LOCKED`**,非常强。

---

#### 最核心的锁兼容关系

| 锁类型        | 是否允许别人读 | 是否允许别人写 |
| ---------- | ------- | ------- |
| FOR SHARE  | ✓       | ✗       |
| FOR UPDATE | 部分允许    | ✗       |
| 表READ锁     | ✓       | ✗       |
| 表WRITE锁    | ✗       | ✗       |

---

#### 你可以这样理解,数据库锁本质是“控制并发访问资源”,从轻到重：

```text
乐观锁
→ 行锁
→ Gap Lock
→ Next-Key Lock
→ 表锁
```

**性能越来越低，保护越来越强。**



