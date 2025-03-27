> <h1></h1>
- [**GORM框架**](#GORM框架)
- [Gorm配置](#Gorm配置)
	- [gorm提供初始化配置](#gorm提供初始化配置)
	- [Gorm约定配置](#Gorm约定配置)
		- [Gorm定义模型](#Gorm定义模型)
		- [Gorm标签](#Gorm标签) 
			- [3种gorm结构体标签区别](#3种gorm结构体标签区别)
		- [GORM初始化数据库](#GORM初始化数据库) 
			- [自动迁移AutoMigrate()详解](#自动迁移AutoMigrate()详解)
			- [自动迁移替代方案](#自动迁移替代方案)
				- [gormigrate迁移](#gormigrate迁移)
				- [使用外部数据库迁移工具golang-migrate回滚](#使用外部数据库迁移工具golang-migrate回滚)
- [创建表](#创建表)
- [GORM插入数据](#GORM插入数据) 
	- [插入时出现主键冲突解决](#插入时出现主键冲突解决)
- [GORM查询数据](#GORM查询数据) 
	- [db.Begin()开启事务](#db.Begin()开启事务)
- [GORM更新数据](#GORM更新数据) 
- [GORM删除数据](#GORM删除数据)
	- [无法批量删除解决](#无法批量删除解决)
- **资料**
	- [Gorm文档](https://gorm.io/zh_CN/docs/connecting_to_the_database.html)
	- [golang最简单的gorm教程(枫枫知道-B站)-Video](https://www.bilibili.com/video/BV1xg411t7RZ?spm_id_from=333.788.player.switch&vd_source=a7fe275f0ee54c4d2f691a823f8876b8&p=6)
		- [](https://www.fengfengzhidao.com/article/4NlbH4sBEG4v2tWkFmpL)




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
  NamingStrategy           schema.Namer // 命名前缀,对表名的设置 
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
跳过默认开启事务模式。为了确保数据一致性，GORM 会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，可以在初始化时禁用它。关掉它可以提高60%的性能,但是一般是开着的.

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
  NamingStrategy: schema.NamingStraztegy{
	  TablePrefix: "f_", //表名前缀,
	  SingularTable: true, //是否单数表名,
	  NoLowerCase:true,  // 不要小写转换
  }
})
```

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
> <h3 id="3种gorm结构体标签区别">3种gorm结构体标签区别</h3>

```go
type GormUser struct {
	Id int64 `json:"id" gorm:"primary_key"`
	
	Username1 string `gorm:"column:username1" json:"username1"`
	
	Username2 string `gorm:"column:username2"`
	
	Username3 string `gorm:"username3"`
	
	Username4 string `json:"username4"`
}
```

**提问: Username1、Username2、Username3、Username4这4种结构体有什么区别嘛?**

**Username1**

- **同时使用 GORM 和 JSON 标签，这样：**
	- GORM 负责数据库字段映射
	- JSON 负责 API 数据格式

```go
type GormUser struct {
    Username1 string `gorm:"column:username1" json:"username1"
}
```

**数据库查询**

```sql
SELECT username FROM users;
```


<br/>

- **`Username2`**

当 GORM 读取或写入数据库时，会将 Username2 字段映射到数据库中的 username2 列，而不是默认的 user_name2（GORM 默认会将字段名转换为 snake_case）。
例如：

```sql
SELECT username FROM users;
```

<br/>

**Username3**

- `gorm:"username3"` 不会生效，因为 GORM 需要明确的 column: 指定字段名。
- GORM 只会按照默认规则推测列名，例如：
	- Username → username（自动转为 snake_case）

<br/>

**Username4**

gorm在读取和写入时,会把Username4映射到数据库为: `user_name4`


<br/><br/>

什么时候可以省略 **gorm:"column:username"？**
如果你的字段 本身已经符合 GORM 的默认映射规则，可以省略 column:，GORM 会自动处理：

```go
type User struct {
    ID       uint   `gorm:"primaryKey"`
    Username string // GORM 自动映射为 `username`
}
```

但如果数据库中的列名 不符合 GORM 的默认规则，比如：

```sql
CREATE TABLE users (
    user_name TEXT NOT NULL
);
```

那么你必须手动指定：

```go
Username string `gorm:"column:user_name"`
```
否则 GORM 会找 username，导致字段不匹配。

<br/><br/>

**提问:** 为什么CreatedAt字段被gorm映射为`crated_at`?

在 GORM 中，结构体字段的默认映射规则是 **驼峰转蛇形（CamelCase → snake_case）**，因此 `CreatedAt` 会被映射为 `created_at`，而 `Username` 会被映射为 `username`。

---

**1.GORM 的默认命名规则**
GORM 默认会将 Go 结构体字段名转换为 **小写的蛇形命名（snake_case）**：

| 结构体字段 | 默认映射的数据库列名 |
|------------|----------------------|
| `CreatedAt` | `created_at` |
| `UpdatedAt` | `updated_at` |
| `DeletedAt` | `deleted_at` |
| `UserID` | `user_id` |
| `Username` | `username` |
| `FirstName` | `first_name` |

GORM 这样做是因为 **snake_case 在 SQL 领域更常见**，比如：

```sql
SELECT created_at FROM users;
```
而不是：

```sql
SELECT CreatedAt FROM users; -- 不符合 SQL 规范
```

---

- **2.为什么 `CreatedAt` 变成 `created_at`？**
	- `CreatedAt` 是 **大写驼峰（PascalCase）**，GORM **自动转换** 为小写蛇形 `created_at`。
	- 这种转换是 GORM **默认行为**，你可以通过 `gorm:"column:CreatedAt"` 来阻止转换：

```go
CreatedAt time.Time `gorm:"column:CreatedAt"`
```
  这样 `CreatedAt` 就不会变成 `created_at` 了。

---

- **3. 为什么 `Username` 变成 `username`？**
	- `Username` 只有一个单词，GORM 直接 **变小写**，所以 `Username` → `username`。
	- 但如果字段是 `UserName`（大写 `N`），GORM 仍然会把它转换成 `user_name`。

---

- **4.如何自定义数据库字段名？**
如果你不希望 GORM 进行默认转换，可以手动指定列名：

```go
type User struct {
    Username  string    `gorm:"column:UserName"`  // 变成 UserName（区分大小写）
    CreatedAt time.Time `gorm:"column:Created_At"` // 变成 Created_At
}
```

---

- **5.关闭 GORM 的自动命名转换**
如果你希望 GORM **完全按照结构体字段的大小写映射数据库**，可以在初始化 GORM 时关闭默认转换：

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
    NamingStrategy: schema.NamingStrategy{
        SingularTable: true,  // 关闭表名复数形式
        NoLowerCase:   true,  // 关闭小写转换
    },
})
```
这样：
- `CreatedAt` → `CreatedAt`
- `Username` → `Username`

但这通常不推荐，因为 **SQL 语句区分大小写可能导致兼容性问题**。


<br/><br/>
> <h3 id="GORM初始化数据库"> GORM 初始化数据库 </h3>
定义完这些标签之后，你可以使用 AutoMigrate 在 MySQL 建立连接之后创建表：

```go
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

<br/><br/>
> <h3 id="自动迁移AutoMigrate()详解">自动迁移AutoMigrate()详解</h3>

**`db.AutoMigrate(&UserModel{})` 在 GORM 中的作用**
在 GORM 中：

```go
db.AutoMigrate(&UserModel{})
db.AutoMigrate(&FollowModel{})
```
用于**自动创建或更新数据库表结构**，确保表与 Go 结构体模型匹配。

---

- **1.`AutoMigrate` 具体做了什么？**
- `AutoMigrate` 的作用：
	- **如果数据库中不存在对应的表，则创建该表**。
	- **如果表已存在**，则：
		- **检查是否有新的字段**，如果有，则**自动添加**。
		- **不会删除已有字段或索引**（不会影响数据）。
		- **不会修改已有字段的类型**（如果类型不匹配，需要手动迁移）。
   
**示例：**

```go
type UserModel struct {
    ID       uint   `gorm:"primaryKey"`
    Username string `gorm:"unique"`
    Email    string `gorm:"index"`
}
```
执行：

```go
db.AutoMigrate(&UserModel{})
```
GORM 可能会自动生成以下 SQL：

```sql
CREATE TABLE IF NOT EXISTS user_models (
    id SERIAL PRIMARY KEY,
    username TEXT UNIQUE,
    email TEXT,
    INDEX (email)
);
```

如果你 **之后** 修改 `UserModel` 结构，比如新增 `Age` 字段：

```go
type UserModel struct {
    ID       uint   `gorm:"primaryKey"`
    Username string `gorm:"unique"`
    Email    string `gorm:"index"`
    Age      int
}
```
再次运行：

```go
db.AutoMigrate(&UserModel{})
```
GORM 可能会执行：

```sql
ALTER TABLE user_models ADD COLUMN age INT;
```
**不会删除旧字段，但会新增 `age` 字段。**

---

- **2.`AutoMigrate` 是必须的吗？**
**不一定**，但建议在开发阶段使用，生产环境需要谨慎：

- **✅ 推荐使用（开发阶段）**
	- **自动创建表结构，减少手写 SQL**
	- **表字段变化时自动更新，省去手动 `ALTER TABLE` 的麻烦**
	- **适合快速迭代的项目**

<br/>

- **⚠️ 生产环境需要注意**
- 在生产环境：
	- **建议手动管理数据库迁移**（如 `gormigrate` 或 `golang-migrate`）。
	- `AutoMigrate` **不会删除旧字段**，但如果某个字段在结构体中被删除，表里仍然保留该字段。
	- **字段类型变更不会生效**，例如：

```go
type UserModel struct {
  Age int `gorm:"size:3"`
}
```
如果 `Age` 已经存在，GORM **不会** 自动更新其大小 `size:3`，可能导致数据库字段类型不匹配。

<br/><br/><br/>
> <h2 id="自动迁移替代方案">自动迁移替代方案</h2>
**`AutoMigrate`总结**

| 方式 | 适用场景 | 优势 | 劣势 |
|------|----------|------|------|
| `AutoMigrate` | **开发环境** | 自动更新表结构，无需手写 SQL | **不能删除字段，不能修改字段类型** |
| `gormigrate` | **生产环境** | **支持版本控制和回滚**，可手动管理变更 | 需要写代码 |
| `golang-migrate` | **生产环境** | **纯 SQL 迁移，可与不同数据库兼容** | 需要维护 `.sql` 文件 |

<br/><br/>
> <h3 id="gormigrate迁移">gormigrate迁移</h3>
- **3.`AutoMigrate` 的替代方案**
如果不想使用 `AutoMigrate`，可以使用 **数据库迁移工具**：
	- 1.**`gormigrate`**（GORM 官方推荐）
		- 需要手动编写迁移代码：

```go
package main

import (
    "github.com/go-gormigrate/gormigrate/v2"
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
    "log"
)

// 定义 UserModel
type UserModel struct {
    ID   uint
    Name string
    Age  int
}

func main() {
    // 连接数据库
    db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }

    // 定义迁移
    m := gormigrate.New(db, gormigrate.DefaultOptions, []*gormigrate.Migration{
        {
            ID: "20240321_add_age_column",
            Migrate: func(tx *gorm.DB) error {
                return tx.Migrator().AddColumn(&UserModel{}, "Age")
            },
            // 回滚（Rollback） 就是 撤销数据库的某次变更，恢复到迁移前的状态。
            Rollback: func(tx *gorm.DB) error {
                return tx.Migrator().DropColumn(&UserModel{}, "Age")
            },
        },
    })

    // 执行迁移, 会执行 所有未执行过的 Migrate 迁移步骤，并按顺序运行 Migrate 函数
    // GORM 执行 ALTER TABLE 语句，数据库会新增 age 字段。
    if err := m.Migrate(); err != nil {
        log.Fatal("迁移失败:", err)
    } else {
        log.Println("迁移成功")
    }
    
	// 如果想回滚：GORM 执行 ALTER TABLE DROP COLUMN 语句，数据库会删除 age 字段，恢复到迁移前的状态
	// if err := m.RollbackLast(); err != nil {
	//     panic(err)
	// }
}
```

**代码解读:**

这段代码是 GORM 迁移（Migration）和回滚（Rollback） 的一个迁移步骤：

- Migrate（迁移）: `tx.Migrator().AddColumn(&UserModel{}, "Age")`
	- 这会在数据库表 user_models 中添加 Age 字段。
	- 也就是执行 SQL：

```sql
ALTER TABLE user_models ADD COLUMN age INT;
```

- Rollback（回滚）: `tx.Migrator().DropColumn(&UserModel{}, "Age")`
	- 这会删除 user_models 表的 Age 字段。
	- 也就是执行 SQL：

```sql
ALTER TABLE user_models DROP COLUMN age;
```

- ID: "20240321_add_age_column"
	- 这是给这个迁移操作起的一个唯一标识符（通常是时间戳+描述），防止重复执行。

<br/><br/>

**支持回滚（Rollback），更安全。**

- **Migrate() 方法的执行逻辑**

- 1.检查数据库中是否已经执行过 ID: "20240321_add_age_column"
	- GORMigrate 维护一个迁移历史表（默认 gormigrate_migrations），记录哪些迁移已执行。
	- 如果 20240321_add_age_column 还没有执行，它就会运行 Migrate 方法。
	- 如果已经执行过，则跳过，不会重复执行。

- 执行 Migrate 函数
	- 运行 tx.Migrator().AddColumn(&UserModel{}, "Age")，向 users 表添加 age 列。

- 记录迁移已完成
	- 在 gormigrate_migrations 表中记录该 ID，防止重复执行。


<br/><br/>
> <h3 id="使用外部数据库迁移工具golang-migrate回滚"> 使用外部数据库迁移工具golang-migrate回滚 </h3>
**golang-migrate 是什么？**
golang-migrate 是一个命令行工具，不是 GUI（图形界面工具）。 它专门用来管理数据库迁移，支持：
- MySQL
- PostgreSQL
- SQLite
- SQL Server
- MongoDB 等

<br/>

**`golang-migrate` 使用示例**
**📌 1️⃣ 安装**

```sh
go install -tags 'mysql postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```
或者：

```sh
brew install golang-migrate
```

<br/>

 **📌 2️⃣ 创建迁移文件**

```sh
migrate create -ext sql -dir migrations -seq add_age_to_users
```
这会生成：

```
migrations/
  000001_add_age_to_users.up.sql
  000001_add_age_to_users.down.sql
```
<br/>

**📌 3️⃣ 编辑 SQL**
**添加 `age` 字段（`up.sql`）**

```sql
ALTER TABLE users ADD COLUMN age INT;
```

**删除 `age` 字段（`down.sql`）**

```sql
ALTER TABLE users DROP COLUMN age;
```

<br/>

**📌 4️⃣ 执行迁移**

```sh
migrate -database "mysql://root:password@tcp(localhost:3306)/testdb" -path ./migrations up
```
这会**执行 `.up.sql` 迁移**，添加 `age` 字段。

<br/>

**📌 5️⃣ 回滚**

```sh
migrate -database "mysql://root:password@tcp(localhost:3306)/testdb" -path ./migrations down
```
这会**执行 `.down.sql` 迁移**，删除 `age` 字段。

---

<br/>

**`gormigrate` vs `golang-migrate`**

| 工具 | 适用场景 | 方式 | 是否支持回滚 |
|------|--------|--------|-------------|
| `gormigrate` | **GORM 内部迁移**，写 Go 代码 | 代码方式（GORM API） | ✅ 支持 |
| `golang-migrate` | **数据库迁移工具**，通用 SQL | SQL 方式（命令行） | ✅ 支持 |

<br/>

- **🚀 选择建议**
	- **如果是 GORM 项目，想用 Go 代码管理数据库** → ✅ `gormigrate`
	- **如果想独立管理数据库迁移，不依赖 GORM** → ✅ `golang-migrate`


<br/><br/><br/>

***
<br/>

> <h1 id="创建表">创建表</h1>

```go
package main

import (
	"fmt"
	"log"

	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

// UserModel 定义用户模型
type UserModel struct {
	ID       uint   `gorm:"primaryKey"`
	Username string `gorm:"unique"`
	Email    string `gorm:"unique"`
	Password string `gorm:"not null"`
}

// 创建数据库连接并返回 gorm.DB 实例
func setupDatabase() (*gorm.DB, error) {
	// 使用 SQLite 作为数据库，当然你也可以更换为其他数据库如 MySQL、PostgreSQL 等
	// 执行下面代码时，数据库文件 gorm.db 会生成在项目的根目录。
	db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{})
	if err != nil {
		return nil, err
	}

	// 可选：确保数据库连接池配置
	sqlDB, err := db.DB()
	if err != nil {
		return nil, err
	}
	sqlDB.SetMaxIdleConns(10) // 设置最大空闲连接数

	return db, nil
}

func main() {
	// 设置数据库连接
	db, err := setupDatabase()
	if err != nil {
		log.Fatal("数据库连接失败:", err)
	}

	// 使用 AutoMigrate 自动创建表
	err = db.AutoMigrate(&UserModel{})
	if err != nil {
		log.Fatal("表创建失败:", err)
	} else {
		fmt.Println("用户表创建成功")
	}
}
```

- **方法一:**
	- 首先，通过 setupDatabase() 获取数据库连接。
	- 然后，调用 `db.AutoMigrate(&UserModel{}) `来自动迁移，创建 UserModel 对应的表。

<br/><br/>

**方法2️⃣**

```go
func main() {
	// 设置数据库连接
	db, err := setupDatabase()
	if err != nil {
		log.Fatal("数据库连接失败:", err)
	}

	// 使用 CreateTable 方法创建表
	if !db.Migrator().HasTable(&UserModel{}) {
		err := db.Migrator().CreateTable(&UserModel{})
		if err != nil {
			log.Fatal("创建表失败:", err)
		} else {
			fmt.Println("用户表创建成功")
		}
	} else {
		fmt.Println("表已存在")
	}
}
```

- **说明：**
	- db.Migrator().HasTable(&UserModel{})：检查表是否已存在。
	- db.Migrator().CreateTable(&UserModel{})：如果表不存在，使用 CreateTable 方法显式地创建表。

<br/><br/>

**方法3️⃣:** 手动执行 SQL 创建表

你也可以使用 Raw 或 Exec 方法直接执行 SQL 语句来创建表。这种方式给你更多的灵活性，可以完全自定义 SQL 语句

```go
func main() {
	// 设置数据库连接
	db, err := setupDatabase()
	if err != nil {
		log.Fatal("数据库连接失败:", err)
	}

	// 手动执行 SQL 语句创建表
	sql := `CREATE TABLE IF NOT EXISTS users (
		id INTEGER PRIMARY KEY AUTOINCREMENT,
		username TEXT NOT NULL UNIQUE,
		email TEXT NOT NULL UNIQUE,
		password TEXT NOT NULL
	);`

	// 使用 db.Exec 执行 SQL 语句
	if err := db.Exec(sql).Error; err != nil {
		log.Fatal("创建表失败:", err)
	} else {
		fmt.Println("用户表创建成功")
	}
}
```

- **说明：**
	- 使用 db.Exec(sql) 方法执行原始 SQL 语句，直接在数据库中创建表。


<br/><br/><br/>

***
<br/>

> <h1 id="GORM插入数据"> GORM插入数据 </h1>

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
**代码:**

```go

db *gorm.DB = ...

type UserModel struct {
    ID   uint   `gorm:"primaryKey"`
    Name string
    Age  int
}

var model UserModel
err := db.Where(condition).First(&model).Error
```
上述的 **`condition`** 条件可能是：

**字符串查询：**

```go
db.Where("name = ?", "Alice").First(&model)
```

<br/>

**结构体查询：**

```go
db.Where(UserModel{Name: "Alice", Age: 0}).First(&user)
```
**问题：**
- Age 的值是 0（int 的零值），GORM 不会 把它加入 WHERE 条件中。
- 结果：可能查出 name = "Alice" 但 age ≠ 0 的记录。

如果改成 &UserModel{}，则不会有这个问题：

```go
db.Where(&UserModel{Name: "Alice"}).First(&model)
```
**结构体值传递 vs. 指针传递**
- UserModel{Name: "Alice"}（值）
	- GORM 可能不会正确识别 struct 的 gorm:"default" 默认值字段。

- &UserModel{Name: "Alice"}（指针）
	- 结构体指针能确保 GORM 处理所有字段，包括默认值和零值字段。

- 避免 GORM 复制结构体
	- 传指针 更高效，不会复制整个 UserModel 结构体。

<br/>

**可以不加 & 吗？**
如果 UserModel 结构体没有默认值或零值字段，通常可以不加 &。

但是 推荐 使用 &UserModel{}，以避免潜在问题。



<br/>

**Map 查询：**

```go
db.Where(map[string]interface{}{"name": "Alice", "age": 25}).First(&model)
```

<br/>

**多条查询**

```go
var users []User
db.Where("age > ?", 20).Find(&users)
```

<br/><br/>
> <h2 id="db.Begin()开启事务">db.Begin()开启事务</h2>

```go
// 用于查找文章的 Go 语言函数，支持 按标签（tag）、作者（author）、收藏（favorited）等条件查询文章列表，并支持 分页
func FindManyArticle(tag, author, limit, offset, favorited string) ([]ArticleModel, int, error) {
	db := common.GetDB()
	var models []ArticleModel
	var count int

	offset_int, err := strconv.Atoi(offset) //转换字符串为整数，并做错误处理
	if err != nil {
		offset_int = 0
	}

	limit_int, err := strconv.Atoi(limit) //转换字符串为整数，并做错误处理
	if err != nil {
		limit_int = 20
	}

	tx := db.Begin()// 开启一个 数据库事务（避免部分查询失败时数据不一致）
	if tag != "" {// 按 tag（标签）查询, 如果 标签存在，查询 与该标签关联的文章（ArticleModels）
		var tagModel TagModel
		tx.Where(TagModel{Tag: tag}).First(&tagModel)
		if tagModel.ID != 0 {
			tx.Model(&tagModel).Offset(offset_int).Limit(limit_int).Related(&models, "ArticleModels") // 查询当前 tagModel 关联的 ArticleModel 数据
			count = tx.Model(&tagModel).Association("ArticleModels").Count() // 统计符合条件的文章总数。
		}
	} else if author != "" {//按 author（作者）查询, 先查询 UserModel（用户），然后找到 该用户关联的 ArticleModel
		var userModel users.UserModel
		tx.Where(users.UserModel{Username: author}).First(&userModel)
		articleUserModel := GetArticleUserModel(userModel)

		if articleUserModel.ID != 0 {
			count = tx.Model(&articleUserModel).Association("ArticleModels").Count() // 计算文章总数
			tx.Model(&articleUserModel).Offset(offset_int).Limit(limit_int).Related(&models, "ArticleModels") // 查询该作者的文章
		}
	} else if favorited != "" { // 按 favorited（收藏）查询, favorited 参数是一个用户名，表示 查找某个用户收藏的文章
		var userModel users.UserModel
		tx.Where(users.UserModel{Username: favorited}).First(&userModel)
		articleUserModel := GetArticleUserModel(userModel)
		if articleUserModel.ID != 0 {
			var favoriteModels []FavoriteModel
			// 在 FavoriteModel 表中查询 该用户收藏的文章 ID。
			tx.Where(FavoriteModel{
				FavoriteByID: articleUserModel.ID,
			}).Offset(offset_int).Limit(limit_int).Find(&favoriteModels)

			// 统计符合条件的文章数量
			count = tx.Model(&articleUserModel).Association("FavoriteModels").Count()
			// 遍历收藏记录 favoriteModels，找到每篇被收藏的 ArticleModel
			for _, favorite := range favoriteModels {
				var model ArticleModel
				tx.Model(&favorite).Related(&model, "Favorite")
				models = append(models, model)
			}
		}
	} else {
		// 如果 tag、author、favorited 都为空，则查询所有文章
		db.Model(&models).Count(&count) // 统计所有文章
		db.Offset(offset_int).Limit(limit_int).Find(&models) // 查询所有文章
	}

	for i, _ := range models {
		tx.Model(&models[i]).Related(&models[i].Author, "Author") // 关联作者
		tx.Model(&models[i].Author).Related(&models[i].Author.UserModel) // 关联用户信息
		tx.Model(&models[i]).Related(&models[i].Tags, "Tags") // 关联文章标签
	}
	// tx.Commit() 提交事务，如果过程中出现错误，tx.Commit().Error 返回错误信息。
	err = tx.Commit().Error
	return models, count, err
}
```

**📌 `tx := db.Begin()` 开启事务的作用**
在 `FindManyArticle` 方法中，使用了 `tx := db.Begin()` **开启事务**，事务的作用主要有以下几点：

---

**🔹 1️⃣ 事务的基本概念**
- **事务（Transaction）** 是数据库中的**最小执行单元**，包含一组 SQL 语句，要么全部执行成功，要么全部回滚（失败时撤销已执行的操作）。
- **ACID 四大特性**：
  - **原子性（Atomicity）**：所有操作要么全部成功，要么全部失败。
  - **一致性（Consistency）**：事务完成后，数据库状态必须保持一致。
  - **隔离性（Isolation）**：不同事务之间相互独立，不影响彼此。
  - **持久性（Durability）**：事务提交后，数据永久存储在数据库中。

---

**🔹 2️⃣ 代码中的事务**
在 `FindManyArticle` 方法中：

```go
tx := db.Begin() // 开启事务
...
err = tx.Commit().Error // 提交事务
```
**如果在事务内发生错误，应该调用 `tx.Rollback()`，防止数据不一致**：

```go
if err != nil {
    tx.Rollback() // 事务回滚，撤销所有更改
    return nil, 0, err
}
```

---

**🔹 3️⃣ 为什么这里要使用事务？**
**✅ 防止数据读取时发生不一致**
- 代码中有多个 `tx.Model(...).Related(...)` 操作：

```go
tx.Model(&tagModel).Offset(offset_int).Limit(limit_int).Related(&models, "ArticleModels")
```
  由于数据库是**并发访问**的，在一个查询尚未完成时，其他事务可能**更新**或**删除**数据，导致查询结果不一致。使用事务可以**锁定数据**，确保一致性。

 **✅ 处理多个关联查询，保证一致性**
- 代码涉及 **多张表**（`ArticleModel`、`TagModel`、`FavoriteModel`）。
- 在事务内进行所有查询，确保这些数据**不会在查询过程中被修改**，避免数据不一致的情况。

**✅ 避免脏读 & 幻读**
- **脏读**：事务 A 读取了事务 B 未提交的数据，B 回滚后，A 读到的数据就**变成了无效数据**。
- **幻读**：事务 A 读取数据时，事务 B **插入了新数据**，导致 A 看到的数据突然变化。

---

**🔹 4️⃣ 如果不使用事务会有什么问题？**
**假设场景**：
1. 你在 **查询** `TagModel` 相关文章的同时，另一用户在 **修改** `TagModel` 或 `ArticleModel` 数据。
2. 如果不使用事务，你可能会获取到**部分更新**的数据，导致：
   - **部分数据丢失**。
   - **查询到的数据和最终数据库状态不一致**。

---

**🔹 5️⃣ 事务的完整写法**
如果你需要在事务失败时回滚：

```go
tx := db.Begin() // 开启事务
if tx.Error != nil {
    return nil, 0, tx.Error
}

defer func() {
    if r := recover(); r != nil {
        tx.Rollback() // 发生 panic 时回滚
    }
}()

// 业务逻辑
if err := tx.Model(&tagModel).Offset(offset_int).Limit(limit_int).Related(&models, "ArticleModels").Error; err != nil {
    tx.Rollback()
    return nil, 0, err
}

err = tx.Commit().Error
if err != nil {
    return nil, 0, err
}

return models, count, nil
```

---

 **📌 总结**

| **为什么要用事务？** | **解释** |
|-----------------|----------------------------|
| **保证数据一致性** | 确保多个查询结果一致，不被其他操作影响 |
| **避免脏读/幻读** | 事务控制了数据变更，防止读取到未提交的数据 |
| **防止部分更新成功** | 如果事务中某一部分失败，可以回滚所有操作 |
| **保护并发操作** | 避免多个查询同时修改数据，导致数据错乱 |


<br/><br/>
> <h3 id="">SQL数据库Inner join实践</h3>

上述中一段代码如下:

```go
type TagModel struct {
	gorm.Model
	// 在数据库层面创建 唯一索引，保证 Tag 不重复
	Tag           string         `gorm:"unique_index"` 
	//  定义 多对多关系（many2many）。 中间表 article_tags 负责 连接 TagModel 和 ArticleModel
	ArticleModels []ArticleModel `gorm:"many2many:article_tags;"` 
	/*
	article_tags结构如下:
	| tag_model_id | article_model_id |
	|:--|:--|
	| 1 | 101 |
	| 1 | 102 |
	| 2 | 103 |
	*/
}


if tag != "" {// 按 tag（标签）查询, 如果 标签存在，查询 与该标签关联的文章（ArticleModels）
	var tagModel TagModel
	tx.Where(TagModel{Tag: tag}).First(&tagModel)
	if tagModel.ID != 0 {
		tx.Model(&tagModel).Offset(offset_int).Limit(limit_int).Related(&models, "ArticleModels") // 查询当前 tagModel 关联的 ArticleModel 数据
		count = tx.Model(&tagModel).Association("ArticleModels").Count() // 统计符合条件的文章总数。
	}
}
```

这段涉及3个表的操作:
- **tags（标签表 TagModel**）
- **articles（文章表 ArticleModel）**
- **article_tags（文章与标签的多对多关系表）**

对数据库的操作等效于:

```sql
SELECT a.* 
FROM articles a 
JOIN article_tags at ON a.id = at.article_model_id 
JOIN tags t ON at.tag_model_id = t.id 
WHERE t.tag = 'Golang' 
LIMIT 20 OFFSET 0;
```

[具体解释请看这里](./数据库SQL.md#连接innerJoin)


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

