> <h1></h1>
- [**GORM框架**](#GORM框架)
- [Gorm配置](#Gorm配置)
	- [gorm提供初始化配置](#gorm提供初始化配置)
	- [Gorm约定配置](#Gorm约定配置)
		- [Gorm定义模型](#Gorm定义模型)
		- [Gorm标签](#Gorm标签) 
		- [GORM初始化数据库](#GORM初始化数据库) 
	- [GORM插入数据](#GORM插入数据) 
		- [插入时出现主键冲突解决](#插入时出现主键冲突解决)
	- [GORM查询数据](#GORM查询数据) 
	- [GORM更新数据](#GORM更新数据) 
	- [GORM删除数据](#GORM删除数据)
		- [无法批量删除解决](#无法批量删除解决)
- **资料**
	- [Gorm文档](https://gorm.io/zh_CN/docs/connecting_to_the_database.html)




<br/><br/><br/>

***
<br/>

># <h1 id="GORM框架">[GORM框架](https://learnku.com/docs/gorm/v2/write_plugins/9750)</h1>
[](https://www.cnblogs.com/rickiyang/p/14517120.html)
[编写插件和钩子](https://learnku.com/docs/gorm/v2/write_plugins/9750)

GORM 官方支持的数据库类型有： MySQL, PostgreSQL, SQlite, SQL Server。


<br/><br/><br/>

***
<br/>

> <h1 id="Gorm配置">Gorm配置</h1>

**引入:**

**连接MySQL示例:**

```go
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  // 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

- **`<user>`**：数据库用户名。
- **`<password>`**：数据库密码。
- **`@tcp(<host>)`**：使用 TCP 协议连接到指定的主机（包括端口，通常默认端口是 3306）。
- **`/<databaseName>`**：要连接的数据库名称。
- 查询参数部分：
- **charset=utf8mb4**：设置字符集为 utf8mb4，支持更多 Unicode 字符（比如 emoji）。
- **parseTime=True**：启用将数据库中的日期时间解析为 Go 的 time.Time 类型。
- **loc=Local**：使用本地时区。

<br/>

MySQl 驱动程序提供了一些高级配置可以在初始化过程中使用，例如：

```go
db, err := gorm.Open(mysql.New(mysql.Config{
  DSN: "gorm:gorm@tcp(127.0.0.1:3306)/gorm?charset=utf8&parseTime=True&loc=Local", // DSN data source name
  DefaultStringSize: 256, // string 类型字段的默认长度
  DisableDatetimePrecision: true, // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  DontSupportRenameIndex: true, // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
  DontSupportRenameColumn: true, // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
  SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
}), &gorm.Config{})
```

- **参数:**
	- DSN: dsn
		- 将上面生成的 DSN 字符串传入，告知 MySQL 驱动如何连接到数据库。
	- DefaultStringSize: 256
		- 当使用 GORM 进行迁移（auto migration）时，如果数据表中有 string 类型的字段且没有指定长度，默认使用 256 个字符的长度。
		- DisableDatetimePrecision: true
			- 禁用 datetime 的精度支持，因为 MySQL 5.6 及更早版本不支持 datetime 字段的精度设置。如果连接的 MySQL 版本较低，设置此选项可避免错误。
	- DontSupportRenameIndex: true
		- 表示当前数据库不支持直接重命名索引，因此在迁移过程中，如果需要修改索引名称，GORM 会采用先删除后重新创建的方式。
		- 这对于 MySQL 5.7 之前的版本或某些 MariaDB 版本是必要的。
	- DontSupportRenameColumn: true
		- 表示数据库不支持直接重命名列，GORM 会采用 CHANGE 语句来修改列名。
		- MySQL 8.0 之前的版本和某些 MariaDB 版本不支持直接的 RENAME COLUMN 操作。
	- SkipInitializeWithVersion: false
		- 设置为 false 表示 GORM 在初始化时会根据当前 MySQL 的版本来进行一些自动配置，例如调整默认选项，以便更好地兼容不同版本的 MySQL。
		- 如果你知道目标数据库版本，通常建议保留这个选项，让 GORM 自动配置。
	- &gorm.Config{}
		- 第二个参数为 GORM 的全局配置，使用默认配置即可

**注意:** `gorm.Open(dialector Dialector, opts ...Option) `函数的第二个参数是接收一个 `gorm.Config{}` 类型的参数，这里就是 gorm 在数据库建立连接后框架本身做的一些默认配置，**请注意这里如果没有配置好，后面你的数据库操作将会很痛苦！**

<br/><br/><br/>
> <h2 id="gorm提供初始化配置">gorm提供初始化配置</h2>

```go
type Config struct {
  SkipDefaultTransaction   bool
  NamingStrategy           schema.Namer
  Logger                   logger.Interface
  NowFunc                  func() time.Time
  DryRun                   bool
  PrepareStmt              bool
  DisableNestedTransaction bool
  AllowGlobalUpdate        bool
  DisableAutomaticPing     bool
  DisableForeignKeyConstraintWhenMigrating bool
}
```

**SkipDefaultTransaction**
跳过默认开启事务模式。为了确保数据一致性，GORM 会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，可以在初始化时禁用它。

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
})
```

<br/>

**NamingStrategy**
表名称的命名策略，下面会说。GORM 允许用户通过覆盖默认的NamingStrategy来更改命名约定，这需要实现接口 Namer：

```go
type Namer interface {
    TableName(table string) string
    ColumnName(table, column string) string
    JoinTableName(table string) string
    RelationshipFKName(Relationship) string
    CheckerName(table, column string) string
    IndexName(table, column string) string
}
```
默认 NamingStrategy 也提供了几个选项，如：

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  NamingStrategy: schema.NamingStrategy{
    TablePrefix: "t_",   // 表名前缀，`User`表为`t_users`
    SingularTable: true, // 使用单数表名，启用该选项后，`User` 表将是`user`
    NameReplacer: strings.NewReplacer("CID", "Cid"), // 在转为数据库名称之前，使用NameReplacer更改结构/字段名称。
  },
})
```
**一般来说这里是一定要配置 SingularTable: true 这一项的。**

<br/>

**Logger**

允许通过覆盖此选项更改 GORM 的默认 logger。

<br/>

**NowFunc**
更改创建时间使用的函数:

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  NowFunc: func() time.Time {
    return time.Now().Local()
  },
})
```

<br/>

**DryRun**
生成 SQL 但不执行，可以用于准备或测试生成的 SQL：

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  DryRun: false,
})
```

<br/>

**PrepareStmt**
PreparedStmt 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率：

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  PrepareStmt: false,
})
```

<br/><br/><br/>
> <h2 id="Gorm约定配置"> Gorm约定配置</h2>
使用别人的框架就要受制别人的约束，在 GORM 中有很多的约定，如果你没有遵循这些约定可能你认为正常的代码跑起来会发生意想不到的问题。

<br/><br/>
> <h3 id="Gorm定义模型">Gorm定义模型</h3>
默认情况下，GORM 会使用 ID 作为表的主键。

```go
type User struct {
    gorm.Model  // 提供 ID、CreatedAt、UpdatedAt
    ID   string // 默认情况下，名为 `ID` 的字段会作为表的主键
    DeletedAt
    Name  string `gorm:"size:100"`
    Age   int
}
```

如果你当前的表主键不是 id 字段，那么你可以通过 primaryKey标签将其它字段设为主键：

```go
// 将 `UUID` 设为主键
type Animal struct {
  ID     int64
  UUID   string `gorm:"primaryKey"`
  Name   string
  Age    int64
}
```

如果你的表采用了复合主键，那也没关系：

```go
type Product struct {
  ID           string `gorm:"primaryKey"`
  LanguageCode string `gorm:"primaryKey"`
  Code         string
  Name         string
}
```

**注意：**默认情况下，整型 `PrioritizedPrimaryField` 启用了 `AutoIncrement`，要禁用它，您需要为整型字段关闭 autoIncrement：

```go
type Product struct {
  CategoryID uint64 `gorm:"primaryKey;autoIncrement:false"`
  TypeID     uint64 `gorm:"primaryKey;autoIncrement:false"`
}
```

<br/><br/>
> <h3 id="Gorm标签">Gorm标签</h3>
GORM 通过在 struct 上定义自定义的 gorm 标签来实现自动化创建表的功能：

```go
type User struct {
	Name string  `gorm:"size:255"` //string默认长度255,size重设长度
	Age int `gorm:"column:my_age"` //设置列名为my_age
	Num int  `gorm:"AUTO_INCREMENT"` //自增
	IgnoreMe int `gorm:"-"` // 忽略字段
	Email string `gorm:"type:varchar(100);unique_index"` //type设置sql类型，unique_index为该列设置唯一索引
	Address string `gorm:"not null;unique"` //非空
	No string `gorm:"index:idx_no"` // 创建索引并命名，如果有其他同名索引，则创建组合索引
	Remark string `gorm:"default:''"` //默认值
}
```

<br/><br/>
> <h3 id="GORM初始化数据库"> GORM 初始化数据库 </h3>

定义完这些标签之后，你可以使用 AutoMigrate 在 MySQL 建立连接之后创建表：

```
func main() {
	db, err := gorm.Open("mysql", "root:123456789@/test_db?charset=utf8&parseTime=True&loc=Local")
	if err != nil {
		fmt.Println("connect db error: ", err)
	}
	db.AutoMigrate(&model.User{})
}
```
**`AutoMigrate`** 用于自动迁移你的 schema，保持 schema 是最新的。 该 API 会创建表、缺失的外键、约束、列和索引。 如果大小、精度、是否为空可以更改，则 AutoMigrate 会改变列的类型。

出于保护数据的目的，它 不会 删除未使用的列。



<br/>

```go

```



<br/><br/>
> <h3 id="GORM插入数据"> GORM插入数据 </h3>

```go
user := User{Name: "Alice", Age: 25}
db.Create(&user)
```


<br/><br/>
> <h3 id="插入时出现主键冲突解决">插入时出现主键冲突解决</h3>

- **1.什么是主键冲突？**
在 **数据库** 中，**主键冲突** 是指 **向表中插入一条数据时，如果主键（Primary Key）值已经存在，则数据库报错**。

- **示例：**
假设 `gorm_user` 表的 `id` 是 **主键**，表中已有一条数据：

```sql
+----+-------+-----+-----+------------+
| id | name  | age | sex | phone      |
+----+-------+-----+-----+------------+
| 1  | Alice | 18  | 2   | 1234567890 |
+----+-------+-----+-----+------------+
```
如果你执行：

```sql
INSERT INTO gorm_user (id, name, age, sex, phone) VALUES (1, 'Bob', 20, 1, '0987654321');
```
就会 **报错**：

```
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```
**因为 `id=1` 已经存在，不能重复插入相同主键的值。**

---

- **2.如何通过接口调用制造主键冲突？**
你可以**连续发送两次相同 `id` 的请求**，模拟主键冲突。

**示例 `curl` 请求**

```sh
curl -X POST http://localhost:8080/api/user/addUser \
     -H "Content-Type: application/json" \
     -d '{"id": 1, "name": "Alice", "age": 18, "sex": 2, "phone": "1234567890"}'

curl -X POST http://localhost:8080/api/user/addUser \
     -H "Content-Type: application/json" \
     -d '{"id": 1, "name": "Bob", "age": 20, "sex": 1, "phone": "0987654321"}'
```
**第一次插入成功，第二次由于 `id=1` 已经存在，会发生主键冲突。**

---

- **3.解决主键冲突的方法（Upsert 操作）**
GORM 提供 `clause.OnConflict` 机制，可以在主键冲突时执行不同的策略：

**(1) `DoNothing` —— 如果冲突，则忽略**

```go
gorm_practice_config.GormDB.Clauses(clause.OnConflict{DoNothing: true}).Create(&user)
```
📌 **效果**：如果 `id` 已存在，则不插入数据，不报错。

---

**(2) `DoUpdates` —— 如果冲突，则更新指定字段**

```go
gorm_practice_config.GormDB.Clauses(clause.OnConflict{
    Columns:   []clause.Column{{Name: "id"}}, // 发生冲突的列
    DoUpdates: clause.AssignmentColumns([]string{"name", "age", "sex", "phone"}),
}).Create(&user)
```
📌 **效果**：
- 如果 `id` **不存在**，则正常插入。
- 如果 `id` **已存在**，则 **更新 `name`, `age`, `sex`, `phone` 的值**。

**示例：**
1. `curl` 第一次请求（创建新用户）：

```json
{"id": 1, "name": "Alice", "age": 18, "sex": 2, "phone": "1234567890"}
```
2. `curl` 第二次请求（冲突，但更新数据）：

```json
{"id": 1, "name": "Bob", "age": 20, "sex": 1, "phone": "0987654321"}
```
**最终数据库的值会变成：**

```sql
+----+------+-----+-----+------------+
| id | name | age | sex | phone      |
+----+------+-----+-----+------------+
| 1  | Bob  | 20  | 1   | 0987654321 |
+----+------+-----+-----+------------+
```

---

**(3) `UpdateAll` —— 如果冲突，则更新所有非主键字段**

```go
gorm_practice_config.GormDB.Clauses(clause.OnConflict{UpdateAll: true}).Create(&user)
```
📌 **效果**：
- 如果 `id` **不存在**，则正常插入。
- 如果 `id` **已存在**，则 **更新除 `id` 以外的所有字段**。

---

**4. 在 API 里实现 Upsert（插入或更新）**
如果你希望 **在 API 里自动处理主键冲突**，你可以这样实现：

```go
type GormUser struct {
	Id int64 `json:"id" gorm:"primary_key"`
	Name string `json:"name"`
	Age int32 `json:"age"`
	Sex int8 `json:"sex"`
	Phone string `json:"phone`
	Birthday time.Time `gorm:"column:day_of_the_beast"` // 将列名设为 `day_of_the_beast`
	CreatedAt time.Time // 在创建时，如果该字段值为零值，则使用当前时间填充
	UpdatedAt int       // 在创建时该字段值为零值或者在更新时，使用当前时间戳秒数填充
}


func AddUser(c *gin.Context) {
    var user GormUser

    // 解析 JSON
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request format"})
        return
    }

    // Upsert 操作
    err := gorm_practice_config.GormDB.Clauses(clause.OnConflict{
        Columns:   []clause.Column{{Name: "id"}},  // 以 `id` 为冲突判断条件
        DoUpdates: clause.AssignmentColumns([]string{"name", "age", "sex", "phone", "birthday"}),
    }).Create(&user).Error

    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{"message": "User added or updated successfully!"})
}
```

**其他3种解决方案:**

```go
func UpsertOp(user GormUser) {
	gorm_practice_config.GormDB.Clauses(clause.OnConflict{DoNothing: true}).Create(&user)
	
	// 在`id`冲突时，将列更新为默认值
	gorm_practice_config.GormDB.Clauses(clause.OnConflict{ 
		Columns: []clause.Column{{Name: "id"}},
		DoUpdates: clause.Assignments(map[string] interface{}{"name": "", "age":0, "sex": 1}),
	}).Create(&user)

	// 在`id`冲突时，将列更新为新值
	gorm_practice_config.GormDB.Clauses(clause.OnConflict{
		Columns:   []clause.Column{{Name: "id"}},
		DoUpdates: clause.AssignmentColumns([]string{"name", "age", "sex", "phone"}),
	}).Create(&user)

	// 在冲突时，更新除主键以外的所有列到新值
	gorm_practice_config.GormDB.Clauses(clause.OnConflict{UpdateAll: true}).Create(&user)	
}
```


---

- **5. 总结**

| 方式 | 适用场景 | 示例 |
|------|---------|------|
| **`DoNothing`** | 如果主键冲突，忽略插入 | `clause.OnConflict{DoNothing: true}` |
| **`DoUpdates`** | 如果主键冲突，更新指定字段 | `clause.OnConflict{DoUpdates: clause.AssignmentColumns([]string{"name", "age"})}` |
| **`UpdateAll`** | 如果主键冲突，更新所有字段（除主键） | `clause.OnConflict{UpdateAll: true}` |




<br/><br/>
> <h3 id="GORM查询数据"> GORM查询数据 </h3>

```go
var user User
db.First(&user, "name = ?", "Alice")
```

<br/>

**多条查询**

```go
var users []User
db.Where("age > ?", 20).Find(&users)
```

<br/><br/>
> <h3 id="GORM更新数据">GORM更新数据</h3>

```go
db.Model(&user).Update("age", 30)
```

<br/><br/>
> <h3 id="GORM删除数据">GORM删除数据</h3>

```go
db.Delete(&user)
```

<br/><br/>
> <h3 id="无法批量删除解决">无法批量删除解决</h3>

```go
//根据 id 批量删除数据
func BatchDeleteUserByIds(ids []int64) (err error) {
	if ids == nil || len(ids) == 0 {
		return
	}
	//删除方式1
	err = gorm_practice_config.GormDB.Where("id in ?", ids).Delete(GormUser{}).Error
	if err != nil {
		hglog.ErrInfo("gorm 批量删除数据 DeleteUserById err: ", err)
		return err
	}
	//删除方式 2
	//constants.GVA_DB.Delete(model.User{}, "id in ?", ids)

	return nil
}
```

对于全局删除的阻止设定

如果在没有任何条件的情况下执行批量删除，GORM 不会执行该操作，并返回 ErrMissingWhereClause 错误。对此，你必须加一些条件，或者使用原生 SQL，或者启用 AllowGlobalUpdate 模式，例如：

```go
// DELETE FROM `user` WHERE 1=1
constants.GVA_DB.Where("1 = 1").Delete(&model.User{})

//原生sql删除
constants.GVA_DB.Exec("DELETE FROM user")

//跳过设定
constants.GVA_DB.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&model.User{})
```

