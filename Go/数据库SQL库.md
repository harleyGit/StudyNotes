- [**MySQL数据库编程**](#MySQL数据库编程)
- [mysql使用前命令和配置](#mysql使用前命令和配置)
- [下载go-mysql驱动程序](#下载go-mysql驱动程序) 
- [操作mysql数据库](#操作mysql数据库)  	
	- [查询mysql版本](#查询mysql版本) 
	- [新建数据表user](#新建数据表user)
	- [增加一条数据](#增加一条数据)
	- [修改数据](#修改数据)
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

<br/><br/><br/>

> <h2 id="新建数据表user">新建数据表user</h2>

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

***
<br/><br/>
> <h2 id="增加一条数据">增加一条数据</h2>

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

<br/><br/><br/>

> <h2 id="">删除数据</h2>

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

