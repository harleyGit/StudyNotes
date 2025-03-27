> <h1 id=""></h1>
- [**mysql管理 ‌**](#mysql管理)
	- [基本命令行](#基本命令行)
	- [终端单行修改sql语句](#终端单行修改sql语句)
	- [终端多行修改sql语句](#终端多行修改sql语句)
- [**基本信息**](#基本信息)
	- [查看创建表信息](#查看创建表信息)
	- [查看表编码](#查看表编码)
- [**关系模型**](#关系模型)
	- [主键](#主键)
	- [外键](#外键)
	- [索引](#索引)
- [创建表](#创建表)
- [连接join](连接join)
	- [连接inner join](#连接innerJoin)
		- [建表、插入表](#建表、插入表)
		- [数据类型约束讲解](#数据类型约束讲解)
- **资料**
	- [廖雪峰SQL教程-(可以执行sql语句查看结果)](https://liaoxuefeng.com/books/sql/relational/foreign-key/index.html)
	- [SQL-菜鸟教程](https://www.runoob.com/sql/sql-tutorial.html)


<br/><br/><br/>

***
<br/>

> <h1 id="mysql管理">mysql管理</h1>
<br/>

> <h2 id="基本命令行">基本命令行</h2>

| **命令行** | 作用 | 效果 |
|:--|:--| :--|
| sudo /usr/local/mysql/support-files/mysql.server start <br/> <br/>// 若是配置了环境变量后可以: <br/> sudo mysql.server start | **‌ 启动 MySQL 服务** | Starting MySQL <br/>SUCCESS!  |
| sudo mysql.server stop | **关闭 MySQL 服务** |  |
| sudo restart | **重启 MySQL 服务** |  |
| sudo mysql.server status | 检查 MySQL 服务状态 | SUCCESS! MySQL running (99253) |
| sudo mysql -u root -p | 进入mysql指令环境(否则无法通过sql语句在终端操作数据库) |  |
| show databases | 列出 MySQL 数据库管理系统的数据库列表 |  |
| use db_test(数据库名); | USE 数据库名 |  |
| show tables; | 显示指定数据库的所有表，使用该命令前需要使用 use 命令来选择要操作的数据库 |  |
| exit; <br/><br/> quit; | 退出 mysql> 命令提示窗口可以使用 exit 命令 |  |
| ALTER TABLE old_table_name RENAME TO new_table_name; | 修改表名old_table_name为new_table_name | |
|  |  |  |
|  |  |  |

<br/><br/>

**要想在终端通过Shell命令操作数据库,你需要进入sql的指令环境如下:**

```sh
mysql -u root -p;                      
Enter password: 

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26
Server version: 8.4.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

<br/>

**查看数据库版本:**

```sh
SELECT VERSION();
+-----------+
| VERSION() |
+-----------+
| 8.4.0     |
+-----------+
1 row in set (0.00 sec)
```

<br/>

**列出 MySQL 数据库管理系统的数据库列表**

```sh
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

**使用db_test数据库**

```sh
use db_test;

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

<br/>

**展示db_test数据库下的所有的数据表:**

```sh
mysql> show tables;

+-------------------+
| Tables_in_db_test |
+-------------------+
| blog_article      |
| blog_auth         |
| blog_tag          |
| sp_douban_movie   |
| user              |
+-------------------+
5 rows in set (0.01 sec)
```

<br/>

**显示user数据表的属性，属性类型，主键信息 ，是否为 NULL，默认值等其他信息。**

```sh
show columns from user;

+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id    | int          | NO   | PRI | NULL    |       |
| name  | varchar(191) | YES  |     | NULL    |       |
| age   | int          | YES  |     | NULL    |       |
| sex   | tinyint      | YES  |     | NULL    |       |
| phone | varchar(191) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
5 rows in set (0.01 sec)
```

<br/>

**输出db_test数据库管理系统的性能及统计信息**

```sh
show table status from db_test;

mysql> show table status from db_test;
+-----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+--------------------+
| Name            | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation          | Checksum | Create_options | Comment            |
+-----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+--------------------+
| blog_article    | InnoDB |      10 | Dynamic    |    0 |              0 |       16384 |               0 |            0 |         0 |              1 | 2025-03-19 18:17:07 | NULL                | NULL       | utf8mb3_general_ci |     NULL |                | 文章管理           |
| blog_auth       | InnoDB |      10 | Dynamic    |    1 |          16384 |       16384 |               0 |            0 |         0 |              2 | 2025-03-19 18:17:07 | 2025-03-19 18:17:07 | NULL       | utf8mb3_general_ci |     NULL |                |                    |
| blog_tag        | InnoDB |      10 | Dynamic    |    0 |              0 |       16384 |               0 |            0 |         0 |              1 | 2025-03-19 18:17:07 | NULL                | NULL       | utf8mb3_general_ci |     NULL |                | 文章标签管理       |
| gorm_user       | InnoDB |      10 | Dynamic    |    4 |           4096 |       16384 |               0 |            0 |         0 |              5 | 2025-03-19 19:00:18 | 2025-03-20 20:06:38 | NULL       | utf8mb4_0900_ai_ci |     NULL |                |                    |
| sp_douban_movie | InnoDB |      10 | Dynamic    |    0 |              0 |       16384 |               0 |            0 |         0 |              1 | 2025-03-05 14:21:56 | NULL                | NULL       | utf8mb4_0900_ai_ci |     NULL |                |                    |
| user            | InnoDB |      10 | Dynamic    |    1 |          16384 |       16384 |               0 |            0 |         0 |           NULL | 2025-03-19 16:25:25 | NULL                | NULL       | utf8mb4_0900_ai_ci |     NULL |                |                    |
+-----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+--------------------+
6 rows in set (0.01 sec)
```

<br/><br/><br/>
> <h2 id="终端单行修改sql语句">终端单行修改sql语句</h2>

| 功能 | Windows/Linux 快捷键	| Mac 等效操作| 
| :-- | :-- | :--|  
| 删除前字符| 	Backspace | 	Delete 键| 
| 删除后字符	| Delete 或 Ctrl+D	| Fn + Delete | 
| 跳行首/行尾	| Ctrl+A/Ctrl+E	| 同 Ctrl+A/Ctrl+E ✅ | 
| 删除到单词开头	| Ctrl+W	| Ctrl+W ✅ | 
| 删除到行尾	| Ctrl+K	| Ctrl+K ✅ | 


<br/><br/><br/>

***
<br/>

> <h1 id="基本信息">基本信息</h1>

**创建数据库：**

```sql
create database HGDatabase;
Query OK, 1 row affected (0.02 sec)

 show databases;
+--------------------+
| Database           |
+--------------------+
| HGDatabase         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

<br/>

**创建 employees 表**

```sql
 create table employees(id int, name varchar(15));
Query OK, 0 rows affected (0.10 sec)

mysql> show tables;
+----------------------+
| Tables_in_hgdatabase |
+----------------------+
| employees            |
+----------------------+
1 row in set (0.00 sec)
```

<br/>

**插入数据：**

```sql
insert into employees values(1001, 'Tom');

insert into employees values(1002, 'shtart');

insert into employees values(1003, '李白');

```

***
<br/><br/>
> <h3 id="查看创建表信息">查看创建表信息</h3>


**展示创建表employees信息：**

```sql
show create table employees;
+-----------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table     | Create Table                                                                                                                                             |
+-----------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| employees | CREATE TABLE `employees` (
  `id` int DEFAULT NULL,
  `name` varchar(15) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-----------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

`CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci：` 表示表的编码，若不是utf8可能无法插入中文数据，要注意。

***<br/><br/>
> <h3 id="查看表编码">查看表编码</h3>


**查看编码指令：**

```sql
mysql> show variables like 'character_%'
    -> ;
+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_set_client     | utf8mb4                                                |
| character_set_connection | utf8mb4                                                |
| character_set_database   | utf8mb4                                                |
| character_set_filesystem | binary                                                 |
| character_set_results    | utf8mb4                                                |
| character_set_server     | utf8mb4                                                |
| character_set_system     | utf8                                                   |
| character_sets_dir       | /usr/local/Cellar/mysql/8.0.23_1/share/mysql/charsets/ |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.05 sec)
```

<br/>


```sql

```

<br/>



```sql

```

<br/>


```sql

```

<br/>


```sql

```

<br/>



```sql

```

<br/>



```sql

```

<br/>


```sql

```

<br/>






<br/><br/><br/>
> <h2 id="终端多行修改sql语句">终端多行修改sql语句</h2>


```sh
mysql> CREATE TABLE test_teacher00 (
    -> id_num INT PRIMARY KEY AUTO_INCREMENT,
    -> age INT NOT NULL,
    -> 
```

在 macOS 终端的 `mysql` 命令行中，无法直接用鼠标或快捷键在 SQL 语句内部自由移动光标并修改已输入的内容。但可以采取以下方法解决：  

- **方法 1：使用 Ctrl + C 取消当前输入**  
因为 `mysql>` 提示符显示 `->` 说明当前 SQL 语句没有结束，仍在等待输入。这时，你可以按 **`Ctrl + C`** 终止当前输入，重新开始输入正确的 SQL 语句。

<br/>

- **方法 2：使用 `\c` 取消当前输入**  
如果已经输入了一部分 SQL 但未执行，可以输入 `\c` 然后按回车，取消当前 SQL 语句，之后重新输入：  

```sql
mysql> \c
```
然后重新正确输入 `CREATE TABLE` 语句。

<br/>

**方法 3：使用 `edit` 命令（推荐）**
如果你的 MySQL 支持 `edit` 命令，你可以在输入 SQL 语句时输入：

```sql
mysql> \e
```
这会打开默认的文本编辑器（通常是 `vi` 或 `nano`），你可以在其中编辑 SQL 语句，编辑完成后保存退出，MySQL 会自动执行修改后的 SQL 语句。
<br/>

**vim退出方法:**
- 按 I 键是进入编辑模式
- 按 Esc 退出编辑模式;
	- 输入 :wq 然后按 Enter 进行保存并退出;
	- 强制退出不保存，可以使用 :q!

<br/>

- **方法 4：使用 `source` 方式执行 SQL 文件**  
如果 SQL 语句较长，可以使用文本编辑器（如 `vim` 或 `nano`）编辑 SQL 文件，然后在 MySQL 终端执行：

```sql
mysql> source /path/to/sqlfile.sql;
```
这样可以更方便地编辑和修改 SQL 语句。



<br/><br/><br/>

***
<br/>

> <h1 id="关系模型">关系模型</h1>
<br/>

> <h2 id="主键">主键</h2>
在关系数据库中，一张表中的每一行数据被称为一条记录。一条记录就是由多个字段组成的。例如，students表的两行记录：

| id	| class_id	| name	| gender | 	score | 
|:--|:--|:--|:--|:--|
| 1	| 1	| 小明	| M	| 90 | 
| 2	| 1	| 小红	| F	| 95 | 

每一条记录都包含若干定义好的字段。同一个表的所有记录都有相同的字段定义。

对于关系表，有个很重要的约束，就是任意两条记录不能重复。不能重复不是指两条记录不完全相同，而是指能够通过某个字段唯一区分出不同的记录，这个字段被称为主键。

<br/><br/><br/>
> <h2 id="外键">外键</h2>


<br/><br/><br/>
> <h2 id="索引">索引</h2>


<br/><br/><br/>

***
<br/>

> <h1 id="创建表">创建表</h1>





<br/><br/><br/>

***
<br/>

> <h1 id="连接">连接</h1>



<br/><br/><br/>

># <h2 id="连接innerJoin">[连接inner join](https://www.runoob.com/sql/sql-join-inner.html)</h2>

```sql
SELECT a.* 
FROM test_articles a 
JOIN test_article_tags at ON a.id = at.article_model_id 
JOIN test_tags t ON at.tag_model_id = t.id 
WHERE t.tag = 'Golang' 
LIMIT 20 OFFSET 0;
```
让我们逐步解析这个包含多表关联的SQL查询，结合示例数据表说明每个环节的作用：

<br/><br/>
> <h3 id="建表、插入表">建表、插入表</h3>

- **1.数据表结构说明**
假设我们有3个数据表：test_articles（文章表）、tags（标签表）、article_tags（文章-标签关系表）.
<br/>

**创建 articles（文章表）:**

```sql
CREATE TABLE test_articles (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- 自增 ID
    title VARCHAR(255) NOT NULL,        -- 文章标题
    content TEXT NOT NULL,              -- 文章内容
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 创建时间
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP -- 更新时间
);
```

<br/><br/>
> <h3 id="数据类型约束讲解">数据类型约束讲解</h3>

**1️⃣数据类型约束详解:**

```sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

**作用：**
- 自动记录数据创建时间
当向表中插入新记录时，如果未显式指定 created_at 的值，数据库会自动将当前时间戳（服务器时间）存入该字段

- **关键特性：**
	- 仅触发一次：只在数据插入时自动填充
	- 不可修改：除非手动更新该字段值
	- 典型用途：数据创建时间的审计追踪

<br/>

**2️⃣数据类型约束详解:**

```sql
updated_at TIMESTAMP DEFAULT 0 ON UPDATE CURRENT_TIMESTAMP
```


- **作用：**
	- 初始默认值：DEFAULT 0
		- 插入数据时若未指定值，会存入 0000-00-00 00:00:00（零日期值）
	- 自动更新时间：ON UPDATE CURRENT_TIMESTAMP
		- 当记录发生任何更新时，自动将当前时间戳写入该字段
<br/>

**插入数据:**

```sql
mysql> INSERT INTO test_articles (title, content) VALUES 
    -> ('Golang 入门指南', '学习 Go 语言基础'),
    -> ('深入理解数据库索引', '讲解数据库索引原理'),
    -> ('分布式系统设计', '分布式架构核心概念');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

**查看数据:**

```sql
mysql> select * from test_articles;

+----+-----------------------------+-----------------------------+---------------------+---------------------+
| id | title                       | content                     | created_at          | updated_at          |
+----+-----------------------------+-----------------------------+---------------------+---------------------+
|  1 | Golang 入门指南             | 学习 Go 语言基础            | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
|  2 | 深入理解数据库索引          | 讲解数据库索引原理          | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
|  3 | 分布式系统设计              | 分布式架构核心概念          | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
+----+-----------------------------+-----------------------------+---------------------+---------------------+
3 rows in set (0.00 sec)
```

<br/><br/>

**创建 tags（标签表）**

```sql
CREATE TABLE test_tags (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- 自增 ID
    tag VARCHAR(100) UNIQUE NOT NULL    -- 标签名称（唯一）
);
```

<br/>

**插入数据:**

```
INSERT INTO test_tags (tag) VALUES 
('Golang'), 
('数据库'), 
('分布式系统');
```

<br/>

**查看:**

```
mysql> select * from test_tags;
+----+-----------------+
| id | tag             |
+----+-----------------+
|  1 | Golang          |
|  3 | 分布式系统      |
|  2 | 数据库          |
+----+-----------------+
3 rows in set (0.00 sec)
```

<br/><br/>

**‌ 创建 article_tags（文章-标签关系表）**

```sql
CREATE TABLE test_article_tags (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- 关系 ID
    article_model_id INT NOT NULL,      -- 文章 ID（外键）
    tag_model_id INT NOT NULL,          -- 标签 ID（外键）
    FOREIGN KEY (article_model_id) REFERENCES test_articles(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_model_id) REFERENCES test_tags(id) ON DELETE CASCADE,
    UNIQUE (article_model_id, tag_model_id) -- 确保文章和标签的关系不重复
);
```

<br/>

**插入数据:**

```sql
INSERT INTO test_article_tags (article_model_id, tag_model_id) VALUES 
(1, 1), -- 文章 1 关联 "Golang"
(2, 2), -- 文章 2 关联 "数据库"
(3, 3), -- 文章 3 关联 "分布式系统"
(1, 2); -- 文章 1 也关联 "数据库"
```

**查看:**

```
select * from test_article_tags;

+----+------------------+--------------+
| id | article_model_id | tag_model_id |
+----+------------------+--------------+
|  1 |                1 |            1 |
|  4 |                1 |            2 |
|  2 |                2 |            2 |
|  3 |                3 |            3 |
+----+------------------+--------------+
4 rows in set (0.00 sec)
```

***
<br/><br/>

**🔹第一步：先执行 JOIN article_tags**

```sql
SELECT a.*
FROM test_articles a
JOIN test_article_tags at ON a.id = at.article_model_id;
```
<br/>

**查询结果**

```
+----+-----------------------------+-----------------------------+---------------------+---------------------+
| id | title                       | content                     | created_at          | updated_at          |
+----+-----------------------------+-----------------------------+---------------------+---------------------+
|  1 | Golang 入门指南             | 学习 Go 语言基础            | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
|  1 | Golang 入门指南             | 学习 Go 语言基础            | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
|  2 | 深入理解数据库索引          | 讲解数据库索引原理          | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
|  3 | 分布式系统设计              | 分布式架构核心概念          | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
+----+-----------------------------+-----------------------------+---------------------+---------------------+
4 rows in set (0.01 sec)
```

📌 **解析**
- `JOIN test_article_tags` 让 **文章表 (`test_articles`) 和 `test_article_tags`（文章-标签关系表）** 连接起来。
- 每篇文章可能匹配多个标签，所以 `文章 1` 出现了两次（因为它有两个标签）。


<br/>

**SQL语句逐句解析:**

```sql
SELECT a.*
```
- **作用**：选取文章表所有字段
- **示例结果**：将返回id、title、content三个字段
- **注意**：使用`a.*`而非`*`可避免字段歧义

<br/>

```sql
FROM test_articles a
```
- **作用**：指定主查询表并设置别名
- **别名机制**：`test_articles a`等价于`test_articles AS a`
- **后续引用**：所有test_articles字段通过`a.column`访问

<br/>

```sql
JOIN test_article_tags at ON a.id = at.article_model_id
```
- **连接类型**：INNER JOIN（默认）
- **连接逻辑**：通过桥梁表建立文章与标签的关联

***
<br/><br/>

**🔹第一步：再 JOIN tags 过滤指定标签**

```
SELECT a.*
FROM test_articles a
JOIN test_article_tags at ON a.id = at.article_model_id
JOIN test_tags t ON at.tag_model_id = t.id
WHERE t.tag = 'Golang';

+----+---------------------+------------------------+---------------------+---------------------+
| id | title               | content                | created_at          | updated_at          |
+----+---------------------+------------------------+---------------------+---------------------+
|  1 | Golang 入门指南     | 学习 Go 语言基础       | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
+----+---------------------+------------------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

📌 **解析**
- 通过 **`JOIN tags`**，我们把 `test_article_tags` 表中的 `tag_model_id` 和 `test_tags` 表的 `id` 进行匹配，获取标签名称。
- 通过 **`WHERE t.tag = 'Golang'`**，我们筛选出了 **只带有 `Golang` 标签的文章**。

---
<br/><br/>

**🔹第三步：加上 `LIMIT` 进行分页**

```sql
SELECT a.*
FROM test_articles a
JOIN test_article_tags at ON a.id = at.article_model_id
JOIN test_tags t ON at.tag_model_id = t.id
WHERE t.tag = 'Golang'
LIMIT 20 OFFSET 0;

+----+---------------------+------------------------+---------------------+---------------------+
| id | title               | content                | created_at          | updated_at          |
+----+---------------------+------------------------+---------------------+---------------------+
|  1 | Golang 入门指南     | 学习 Go 语言基础       | 2025-03-24 12:09:49 | 2025-03-24 12:09:49 |
+----+---------------------+------------------------+---------------------+---------------------+
1 row in set (0.01 sec)
```

📌 **解析**
- `LIMIT 20 OFFSET 0` 让我们 **只获取前 20 条数据**，而不会返回所有匹配结果。
- **分页逻辑**
  - `OFFSET 0`：从第 0 条开始取数据（默认）。
  - `LIMIT 20`：最多取 20 条数据。