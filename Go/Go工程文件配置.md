
> # 知识点
- [企业工程中环境变量安全度方案](#企业工程中环境变量安全度方案)
	- [工程目录结构](#工程目录结构) 
	- [使用环境变量](#使用环境变量) 
	- [脚本db.sh](#脚本db.sh) 
	- [执行SQL](#执行SQL) 
	- [工程启动校验数据库](#工程启动校验数据库)   
	- [数据库引入迁移](#数据库引入迁移)      
	- [Makefile文件统一入口](#Makefile文件统一入口)
- [CI/运维系统注入环境变量流程](#CI/运维系统注入环境变量流程)
	- [Go代码读取环境变量](#Go代码读取环境变量)
		- [环境变量来源讲解](#环境变量来源讲解)
	- [生产环境“注入”环境变量](#生产环境“注入”环境变量)
		- [方案一：Docker注入](#方案一：Docker注入)	
		- [方案二：Kubernetes（K8s）](#方案二：Kubernetes（K8s）) 
		- [方案三：CI/CD](#方案三：CI/CD)
- [数据库迁移进级](#数据库迁移进级)
	- [Go中自动检测migrate状态-只读，绝不改表](#Go中自动检测migrate状态-只读，绝不改表)	
	- [2.多环境配置拆分【dev/test/prod】](#2.多环境配置拆分【dev/test/prod】)
	- [3.Docker+MySQL+migrate一体化](#3.Docker+MySQL+migrate一体化)


<br/><br/><br/>

***
<br/>

> <h1 id="企业工程中环境变量安全度方案">企业工程中环境变量安全度方案</h1>
- **完成 4 件事：**
	- 1.**MySQL 凭证使用环境变量（安全化）**
	- 2.**多个 SQL 按顺序执行（迁移机制）**
	- 3.**Go 程序启动时做数据库就绪校验**
	- 4.**升级为 golang-migrate（企业标准）**

***
<br/><br/><br/>
> <h2 id="工程目录结构">工程目录结构</h2>

```text
project/
├── cmd/
│   └── server/
│       └── main.go
│
├── internal/
│   └── db/
│       └── mysql.go
│
├── migrations/
│   ├── 001_init.up.sql
│   ├── 001_init.down.sql
│   ├── 002_add_user_index.up.sql
│   ├── 002_add_user_index.down.sql
│
├── scripts/
│   └── db.sh
│
├── .env
├── Makefile
└── go.mod
```

***
<br/><br/><br/>
> <h2 id="使用环境变量">使用环境变量</h2>

- **`.env` 文件（本地开发用）**

```env
MYSQL_HOST=127.0.0.1
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=123456 # 密码不要在这里写死，不安全
MYSQL_DB=app_db
```

> 生产环境：
>
> * 不提交 `.env`
> * [由 CI / 运维系统注入](#)

***
<br/><br/><br/>
> <h2 id="脚本db.sh">脚本db.sh</h2>

- **scripts/db.sh（使用环境变量）**

```bash
#!/bin/bash
set -e

### ===== 读取环境变量 =====
export $(grep -v '^#' .env | xargs)

MYSQL_CMD="mysql -h$MYSQL_HOST -P$MYSQL_PORT -u$MYSQL_USER -p$MYSQL_PASSWORD"

check_mysql() {
    if ! mysqladmin ping -h"$MYSQL_HOST" -P"$MYSQL_PORT" \
        -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" --silent; then
        echo "[INFO] MySQL 未启动，启动中..."
        brew services start mysql
        sleep 5
    fi
}

case "$1" in
    shell)
        check_mysql
        $MYSQL_CMD "$MYSQL_DB"
        ;;
    *)
        echo "用法: ./scripts/db.sh shell"
        ;;
esac
```

***
<br/><br/><br/>
> <h2 id="执行SQL">执行SQL</h2>
- **顺序执行 SQL（迁移的“本质”）**

**`migrations/001_init.up.sql`**

```sql
CREATE DATABASE IF NOT EXISTS app_db
  DEFAULT CHARSET utf8mb4;

USE app_db;

CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(128) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_email (email)
);
```

<br/>

**`migrations/002_add_user_index.up.sql`**

```sql
ALTER TABLE users ADD INDEX idx_created_at (created_at);
```

> 规则（企业级铁律）：
>
> * **只写 up.sql / down.sql**
> * **文件名即版本号**
> * **永不修改已执行的 SQL**

***
<br/><br/><br/>
> <h2 id="工程启动校验数据库">工程启动校验数据库</h2>


- **Go 程序启动时校验数据库（不建表）**

	- **`internal/db/mysql.go`**

```go
package db

import (
	"database/sql"
	"fmt"
	"os"

	_ "github.com/go-sql-driver/mysql"
)

func NewMySQL() (*sql.DB, error) {
	dsn := fmt.Sprintf(
		"%s:%s@tcp(%s:%s)/%s?parseTime=true",
		os.Getenv("MYSQL_USER"),
		os.Getenv("MYSQL_PASSWORD"),
		os.Getenv("MYSQL_HOST"),
		os.Getenv("MYSQL_PORT"),
		os.Getenv("MYSQL_DB"),
	)

	db, err := sql.Open("mysql", dsn)
	if err != nil {
		return nil, err
	}

	// 启动即校验 schema
	if _, err := db.Exec("SELECT 1 FROM users LIMIT 1"); err != nil {
		return nil, fmt.Errorf("database schema not ready: %w", err)
	}

	return db, nil
}
```

> **注意：**
>
> * 表不存在 → 程序直接失败
> * 这是“部署错误”，不是“运行时错误”


***
<br/><br/><br/>
> <h2 id="数据库引入迁移">数据库引入迁移</h2>

**正式引入 golang-migrate（企业标准）**

- **1️⃣ 安装 migrate**

```bash
brew install golang-migrate
```

<br/>

**2️⃣ 执行迁移（一次命令）**

```bash
migrate \
  -path migrations \
  -database "mysql://root:123456@tcp(127.0.0.1:3306)/app_db" \
  up
```

- **migrate 做了什么？**
	* 自动创建 `schema_migrations` 表
	* 记录已执行版本
	* 防止重复执行
	* 支持回滚

<br/><br/>

&emsp;&emsp; 上述**终端命令**是在使用 **数据库迁移工具**（很可能是 [Golang 的 `golang-migrate`](https://github.com/golang-migrate/migrate)）来 **将数据库结构“升级”到最新版本**。下面我来逐部分解释：


- **1.`migrate`**
	- 这是调用 `migrate` 工具的命令行程序（通常由 `go install github.com/golang-migrate/migrate/v4/cmd/migrate@latest` 安装）。

<br/>

- **2.`-path migrations`**
	- 指定 **迁移文件所在的目录**。  
	- 这个目录里通常包含一系列 `.sql` 文件（也可能有 `.up.sql` 和 `.down.sql` 配对），例如：
  
```sh
migrations/
├── 1_init.up.sql
├── 1_init.down.sql
├── 2_add_users.up.sql
├── 2_add_users.down.sql
└── ...
```
- 每个 `.up.sql` 文件定义了“如何升级”数据库（比如创建表、加字段等）。
- `.down.sql` 则用于回滚（降级）。

<br/>

- **3. `-database "mysql://..."`**  
	- 指定目标数据库连接信息：
		- 数据库类型：`mysql`
		- 用户名：`root`
		- 密码：`123456`
		- 主机地址：`127.0.0.1:3306`
		- 数据库名：`app_db`

> ⚠️ 注意：密码明文写在命令行中存在安全风险，生产环境建议使用配置文件或环境变量。

<br/>

- **4.`up`**
	- 表示 **执行“向上迁移”**，即：
		- 检查当前数据库已应用到哪个版本（`schema_migrations` 表会记录）
		- 找出所有 **尚未执行的 `.up.sql` 文件**
		- 按顺序依次执行它们

---
<br/>

**举个例子 🌰**

假设你的 `migrations/` 目录中有：

```sh
1_create_users.up.sql     → CREATE TABLE users (...);
2_add_email.up.sql        → ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

而数据库 `app_db` 是空的（或只执行过第1个迁移），那么运行这个 `up` 命令后：
- 如果还没执行过任何迁移 → 会先执行 `1_create_users.up.sql`，再执行 `2_add_email.up.sql`
- 如果已经执行到版本1 → 只执行 `2_add_email.up.sql`
- 最终数据库结构就和代码期望的一致了

<br/>

**总结 ✅**

这行命令的作用是：
> **将本地 MySQL 数据库 `app_db` 按照 `migrations/` 目录中的 SQL 脚本，自动升级到最新结构（比如建表、改字段等）**

***
<br/> 

**3️⃣ 回滚（down）**

```bash
migrate -path migrations -database "xxx" down 1
```

***
<br/><br/><br/>
> <h2 id="Makefile文件统一入口">Makefile文件统一入口</h2>

**Makefile（统一入口，企业必备）**

**Makefile（完整版）**

```makefile
.PHONY: db-shell db-up db-down

db-shell:
	@./scripts/db.sh shell

db-up:
	@migrate -path migrations \
	-database "mysql://$(MYSQL_USER):$(MYSQL_PASSWORD)@tcp($(MYSQL_HOST):$(MYSQL_PORT))/$(MYSQL_DB)" \
	up

db-down:
	@migrate -path migrations \
	-database "mysql://$(MYSQL_USER):$(MYSQL_PASSWORD)@tcp($(MYSQL_HOST):$(MYSQL_PORT))/$(MYSQL_DB)" \
	down 1
```

<br/>

**使用方式**

```bash
make db-up
make db-down
make db-shell
```

---
- **企业能力:现在已经完整掌握：**
	* 数据库 **初始化 / 迁移 / 回滚**
	* SQL **版本化管理**
	* Go 程序 **与 Schema 解耦**
	* Makefile **工程化驱动**
	* 本地 ≠ 生产 的正确认知

**这已经是中大型 Go 后端团队的标准工作流。**


<br/><br/><br/>

***
<br/>

> <h1 id="CI/运维系统注入环境变量流程">CI/运维系统注入环境变量流程</h1>

在 Go（Golang）项目中，`.env` 文件通常用于**本地开发环境**加载配置，而在**生产环境**中，出于安全和可维护性考虑，我们**不会提交 `.env` 文件到代码仓库**，而是通过 **环境变量（Environment Variables）** 的方式注入敏感配置（如数据库密码、API 密钥等）。

---
<br/> 

**✅ 核心思想**

> **无论本地还是生产，Go 程序都从「环境变量」读取配置。**
>
> - 本地：用 `.env` 文件把变量“临时加载”进当前 shell。
> - 生产：由部署平台（如 Docker、Kubernetes、CI/CD 工具）直接设置环境变量。

所以你 Go 代码 **不需要区分环境**，只需要统一读取环境变量即可！

***
<br/><br/><br/>
> <h2 id="Go代码读取环境变量">Go代码读取环境变量</h2>

推荐使用 [github.com/joho/godotenv](https://github.com/joho/godotenv)（仅用于本地开发），配合 `os.Getenv`。

**示例**

```go
package main

import (
	"log"
	"os"

	"github.com/joho/godotenv"
)

func init() {
	// 仅在本地开发时加载 .env 文件
	// 如果是生产环境，.env 文件不存在，这步会失败，但没关系
	if os.Getenv("APP_ENV") != "production" {
		err := godotenv.Load()
		if err != nil {
			log.Println("Warning: No .env file found (assuming production)")
		}
	}
}

func getEnv(key, fallback string) string {
	if value := os.Getenv(key); value != "" {
		return value
	}
	return fallback
}

func main() {
	host := getEnv("MYSQL_HOST", "localhost")
	port := getEnv("MYSQL_PORT", "3306")
	user := getEnv("MYSQL_USER", "root")
	password := getEnv("MYSQL_PASSWORD", "")
	dbName := getEnv("MYSQL_DB", "app_db")

	log.Printf("Connecting to MySQL at %s:%s as %s", host, port, user)
	// 这里连接数据库...
}
```

> 💡 注意：`godotenv.Load()` 只在本地运行时加载 `.env`；在生产环境，因为没有 `.env` 文件，它会报错，但我们忽略错误（或通过 `APP_ENV=production` 跳过加载）。

***
<br/>

**`os.Getenv("APP_ENV")`**

- **含义：**
	- 这是 Go 标准库 `os` 提供的方法，用于 **从当前进程的环境变量中获取名为 `"APP_ENV"` 的值**。
	- `godotenv.Load() `的作用是 把 .env 文件里的内容「注入」到当前进程的环境变量中。
		- 但如果终端（操作系统）已经设置了同名变量，它会覆盖 .env 中的值（取决于 godotenv 的行为）。
	- 如果环境变量存在，返回其值（字符串）。
	- 如果不存在，返回空字符串 `""`。

<br/>

运行前如果你在终端设置了环境变量：

```bash
export APP_ENV=production
go run main.go
```

输出会是：

```
APP_ENV = production
```

---
<br/>

**`err := godotenv.Load()`**

- **含义：**
	- 这是第三方库 [`github.com/joho/godotenv`](https://github.com/joho/godotenv) 提供的功能，用于 **从项目根目录下的 `.env` 文件中加载环境变量到程序的运行环境中**，这样后续就可以用 `os.Getenv()` 读取了。

<br/>

- **安装：**
先通过 go get 安装该包：

```bash
go get github.com/joho/godotenv
```

- **使用方式：**
假设你的项目根目录有一个 `.env` 文件：

```env
# .env
APP_ENV=development
DB_HOST=localhost
DB_PORT=5432
```

然后在 Go 代码中：

```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/joho/godotenv"
)

func main() {
    // 加载 .env 文件
    err := godotenv.Load()
    if err != nil {
        log.Fatal("Error loading .env file")
    }

    // 现在可以读取 .env 中定义的变量
    appEnv := os.Getenv("APP_ENV")
    dbHost := os.Getenv("DB_HOST")

    fmt.Println("APP_ENV =", appEnv)
    fmt.Println("DB_HOST =", dbHost)
}
```

运行结果：

```
APP_ENV = development
DB_HOST = localhost
```

> 注意：`.env` 文件通常放在项目根目录（和 `main.go` 同级），`godotenv.Load()` 默认会读取这个位置的 `.env`。也可以指定路径，如 `godotenv.Load(".env.local")`。

---
<br/>

**能不能在终端直接输入这些命令？**

**不能。**

- `os.Getenv("APP_ENV")` 是 Go 语言代码，不是 shell 命令。
- `godotenv.Load()` 也是 Go 函数调用，只能在 Go 程序中执行。

但你可以在终端做类似的事情：

**✅ 在终端查看环境变量：**

```bash
echo $APP_ENV
```

<br/>

**✅ 手动加载 .env（在 bash/zsh 中）：**

```bash
set -a
source .env
set +a
echo $APP_ENV  # 会输出 development
```

但这只是临时将 `.env` 中的变量导入当前 shell，并不会影响 Go 程序——除非你在运行 Go 程序前已经这样做了。


<br/><br/>
> <h3 id="环境变量来源讲解">环境变量来源讲解</h3>


- **环境变量的来源有两个：**
	- **系统/终端已设置的环境变量**（比如你运行 `export APP_ENV=production`）
	- **`.env` 文件中定义的变量**（通过 `godotenv.Load()` 加载）

<br/>

**`godotenv.Load()` 的默认行为：**
> **如果某个环境变量已经存在（比如从终端继承来的），`godotenv.Load()` 默认不会覆盖它！**

也就是说：
- 如果你在终端执行了 `export APP_ENV=production`
- 然后运行 Go 程序，程序里调用 `godotenv.Load()`
- 而 `.env` 文件里写的是 `APP_ENV=development`

👉 **最终 `os.Getenv("APP_ENV")` 会返回 `production`（终端的值），而不是 `.env` 的 `development`！**

这是 **有意设计的安全行为**：避免 `.env` 意外覆盖生产环境的重要配置。

---
<br/> 

**🧪 验证示例**

假设你的 `.env` 文件内容是：

```env
APP_ENV=development
```

<br/>

**Go 代码：**

```go
package main

import (
    "fmt"
    "os"
    "github.com/joho/godotenv"
)

func main() {
    _ = godotenv.Load() // 加载 .env
    fmt.Println("APP_ENV =", os.Getenv("APP_ENV"))
}
```

<br/>

- **情况一：没在终端设置变量**

```bash
go run main.go
```

输出：

```sh
APP_ENV = development   ✅ 来自 .env
```

<br/>

- **情况二：在终端设置了同名变量**

```bash
export APP_ENV=production
go run main.go
```

输出：

```sh
APP_ENV = production    ✅ 来自终端，.env 被忽略！
```

---
<br/> 

**❓那怎么强制用 `.env` 的值？**

如果你**确实想让 `.env` 覆盖已有环境变量**，可以使用：

```go
err := godotenv.Overload()
```

`Overload()` 会 **强制覆盖** 已存在的环境变量。

```go
_ = godotenv.Overload() // 即使终端有 APP_ENV，也会被 .env 覆盖
```

> ⚠️ 注意：生产环境中慎用 `Overload()`，可能会覆盖重要的部署配置！


***
<br/><br/><br/>
> <h2 id="生产环境“注入”环境变量">生产环境“注入”环境变量</h2>

<br/><br/>
> <h3 id="方案一：Docker注入"> 方案一： Docker注入</h3>

**`这种最常见`** ，构建镜像时 **不包含 `.env`**

**Dockerfile：**

```dockerfile
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
# 不复制 .env！
CMD ["./main"]
```

这个文件的详细介绍，看这里：[语法介绍](./Docker.md#Dockerfile文件语法介绍)

<br/>

 **启动容器时传入环境变量**

```bash
docker run -d \
  -e MYSQL_HOST=mysql.prod.example.com \
  -e MYSQL_PORT=3306 \
  -e MYSQL_USER=prod_user \
  -e MYSQL_PASSWORD=super_secret_password \
  -e MYSQL_DB=prod_app_db \
  -e APP_ENV=production \
  your-go-app-image
```

> ✅ 所有敏感信息通过 `-e` 注入，**不会出现在代码或镜像中**。

***
<br/><br/><br/>
> <h2 id="方案二：Kubernetes（K8s）">方案二：Kubernetes（K8s）</h2>

使用 `Secret` + `Deployment` 注入：

- **1.创建 Secret（加密存储敏感数据）**

```yaml
# mysql-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  # 值必须是 base64 编码
  MYSQL_PASSWORD: c3VwZXJfc2VjcmV0X3Bhc3N3b3Jk  # echo -n "super_secret_password" | base64
```

应用：

```bash
kubectl apply -f mysql-secret.yaml
```

<br/>

- **2.在 Deployment 中引用**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
        - name: app
          image: your-go-app
          env:
            - name: MYSQL_HOST
              value: "mysql.prod.svc.cluster.local"
            - name: MYSQL_PORT
              value: "3306"
            - name: MYSQL_USER
              value: "prod_user"
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD
            - name: MYSQL_DB
              value: "prod_app_db"
            - name: APP_ENV
              value: "production"
```

这些文件的详细解读和如何使用，看这里：[文件详解](./K8s部署.md#安全的将数据库密码传给Go应用容器)

***
<br/><br/><br/>
> <h2 id="方案三：CI/CD">方案三：CI/CD</h2>
**CI/CD（如 GitHub Actions、GitLab CI）**

在 CI 流水线中设置环境变量（通常通过“Secrets”功能）：

**GitHub Actions 示例：**

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        env:
          MYSQL_HOST: ${{ secrets.MYSQL_HOST }}
          MYSQL_PASSWORD: ${{ secrets.MYSQL_PASSWORD }}
          # ...其他变量
        run: |
          ./your-go-app-binary
```

> 在 GitHub 仓库 Settings → Secrets 中设置 `MYSQL_PASSWORD` 等值。

---
<br/>

**总结：最佳实践**

| 环境     | 配置来源         | 是否提交 `.env` | 安全性 |
|----------|------------------|------------------|--------|
| 本地开发 | `.env` 文件       | ✅（但加到 `.gitignore`） | 低（仅本地）|
| 生产环境 | 环境变量（由平台注入） | ❌ 不提交           | 高 ✅ |

> 📌 **永远不要把 `.env` 提交到 Git！**  
> 在 `.gitignore` 中加入：
> ```
> .env
> *.env
> ```


<br/><br/><br/>

***
<br/>

> <h1 id="数据库迁移进级">数据库迁移进级</h1>

>- **完成的功能：**
	- 1️⃣ Go 启动时：只读检测数据库 migrate 状态
	- 2️⃣ 同一套代码：支持 dev / test / prod
	- 3️⃣ 一条命令：Docker 启动 MySQL + migrate + Go
	- 4️⃣ CI：自动校验数据库变更是否合法

这是 **中大型公司后端服务的基本操作，才刚刚开始**

很好，这一步你已经进入**“真正的工程化后端”**阶段了。
下面我会**一步不跳、一步不省**，把你点名的 **4 件事** 按**真实企业顺序**全部给你落到：

* ✅ 流程（为什么这么做）
* ✅ 文件结构（长什么样）
* ✅ 代码（可直接用）
* ✅ 命令（你在终端敲什么）

**默认前提**：
你是 **Mac + Go + MySQL + 小白**，我会按 **0 → 1** 来讲，不假设你已经会。

---

# 总览：你最终会得到什么

当你做完下面全部内容，你的项目将具备：

```
1️⃣ Go 启动时：只读检测数据库 migrate 状态
2️⃣ 同一套代码：支持 dev / test / prod
3️⃣ 一条命令：Docker 启动 MySQL + migrate + Go
4️⃣ CI：自动校验数据库变更是否合法
```

这是 **中大型公司后端服务的完整起跑线**。

***
<br/><br/><br/>
> <h2 id="1.Go中自动检测migrate状态-只读，绝不改表">1.Go中自动检测migrate状态-只读，绝不改表</h2>


**🎯 目的**

> Go 程序 **不能建表、不能迁移**
> 但必须 **知道数据库是不是“对的版本”**

否则：

* 程序启动成功
* 运行到一半 SQL 报错
* 线上事故

<br/>

**1️⃣ migrate 工具默认会建一张表**

```sql
schema_migrations
```

内容大致是：

| version | dirty |
| ------- | ----- |
| 2       | false |

<br/>

 **2️⃣ Go 中只读检测当前 migrate 版本**

**`internal/db/migrate_check.go`**

```go
package db

import (
	"database/sql"
	"fmt"
)

func CheckMigrateVersion(db *sql.DB, expectVersion int) error {
	var version int
	var dirty bool

	err := db.QueryRow(`
		SELECT version, dirty
		FROM schema_migrations
		LIMIT 1
	`).Scan(&version, &dirty)

	if err != nil {
		return fmt.Errorf("cannot read migrate version: %w", err)
	}

	if dirty {
		return fmt.Errorf("database is in dirty migrate state")
	}

	if version < expectVersion {
		return fmt.Errorf(
			"database version too old: current=%d expect>=%d",
			version, expectVersion,
		)
	}

	return nil
}
```


<br/>

**3️⃣ 在 Go 启动时调用（非常关键）**

**`cmd/server/main.go`**

```go
db, err := db.NewMySQL()
if err != nil {
	log.Fatal(err)
}

if err := db.CheckMigrateVersion(db, 2); err != nil {
	log.Fatal(err)
}
```

> 说明
>
> * `2` = 当前代码**最低依赖的数据库版本**
> * 不满足 → **直接拒绝启动**

✅ 这是 **企业级“防错机制”**


***
<br/><br/><br/>
> <h2 id="2.多环境配置拆分【dev/test/prod】">2.多环境配置拆分（dev / test / prod）</h2>

<br/> 

**🎯 目的**

- **同一套代码：**
	* 本地开发 → dev
	* CI 测试 → test
	* 线上 → prod

**绝不 hardcode**

<br/>

**1️⃣ 目录结构（企业标准）**

```text
configs/
├── dev.env
├── test.env
└── prod.env
```

<br/> 

**2️⃣ 配置文件内容**

**configs/dev.env**

```env
APP_ENV=dev
MYSQL_HOST=127.0.0.1
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=123456
MYSQL_DB=app_db
MIGRATE_EXPECT_VERSION=2
```

<br/> 

**configs/test.env**

```env
APP_ENV=test
MYSQL_HOST=mysql
MYSQL_PORT=3306
MYSQL_USER=test
MYSQL_PASSWORD=test
MYSQL_DB=test_db
MIGRATE_EXPECT_VERSION=2
```

<br/>

**configs/prod.env（示例）**

```env
APP_ENV=prod
MYSQL_HOST=prod-mysql.internal
MYSQL_PORT=3306
MYSQL_USER=app
MYSQL_PASSWORD=********
MYSQL_DB=app_db
MIGRATE_EXPECT_VERSION=2
```

<br/> 

**3️⃣ Go 加载配置（标准写法）**

**`internal/config/config.go`**

```go
package config

import (
	"os"
	"strconv"
)

type Config struct {
	MySQLHost string
	MySQLPort string
	MySQLUser string
	MySQLPass string
	MySQLDB   string
	MigrateVersion int
}

func Load() *Config {
	v, _ := strconv.Atoi(os.Getenv("MIGRATE_EXPECT_VERSION"))

	return &Config{
		MySQLHost: os.Getenv("MYSQL_HOST"),
		MySQLPort: os.Getenv("MYSQL_PORT"),
		MySQLUser: os.Getenv("MYSQL_USER"),
		MySQLPass: os.Getenv("MYSQL_PASSWORD"),
		MySQLDB:   os.Getenv("MYSQL_DB"),
		MigrateVersion: v,
	}
}
```

<br/>

**使用步骤：**

- **步骤 1：在程序启动时加载 `.env` 文件（如果使用）**

你需要在 `main()` 或初始化阶段**手动加载 `.env` 文件**，例如使用 [github.com/joho/godotenv](https://github.com/joho/godotenv)：

```go
import "github.com/joho/godotenv"

func main() {
	// 加载 dev.env 到环境变量中
	err := godotenv.Load("dev.env")
	if err != nil {
		log.Fatal("Error loading dev.env file")
	}

	cfg := Load()
	fmt.Printf("MySQL Host: %s\n", cfg.MySQLHost)
	// 后续使用 cfg 建立数据库连接等
}
```

> 如果你**不使用 `.env` 文件**，而是通过 shell 直接设置环境变量（如 `export MYSQL_HOST=...`），那就不需要 `godotenv`。

<br/>

- **步骤 2：使用 Config 对象**

```go
cfg := Load()

// 构造 DSN（Data Source Name）用于连接 MySQL
dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
	cfg.MySQLUser,
	cfg.MySQLPass,
	cfg.MySQLHost,
	cfg.MySQLPort,
	cfg.MySQLDB)

db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
if err != nil {
	panic("failed to connect database")
}
```



***
<br/><br/><br/>
> <h2 id="3.Docker+MySQL+migrate一体化">3.Docker+MySQL+migrate 一体化</h2>

- **🎯 下面的这套配置的目标是：**
	- 启动一个 MySQL 数据库。
	- 在数据库启动并健康后，自动运行数据库迁移脚本（比如建表、加字段等）。
	- 迁移成功后，再启动你的 Go 写的 Web 应用程序。
	- 所有这些都通过 docker-compose up 一键启动！

---

**`hg_docker_compose.dev.yml`**

```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: hh109
      MYSQL_DATABASE: HG_MLC_DB
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-phh109"]
      interval: 5s
      # timeout: 30s
      retries: 5
      # start_period: 30s

  migrate:
    image: migrate/migrate
    depends_on:
      mysql:
        condition: service_healthy
    volumes:
      - ./migrations:/migrations # 注意： ./migrations 是相对于 本文件，否则不对
    command:
      [
        "-path",
        "/migrations",
        "-database",
        "mysql://root:hh109@tcp(mysql:3306)/HG_MLC_DB",
        "up",
      ]
  app:
    build:
      context: ../../
      dockerfile: Dockerfile
    depends_on:
      migrate:
        condition: service_completed_successfully
    env_file:
      - ./../env_configs/hg_debug.env
    ports:
      - "8080:8080"
```


<br/>

**2️⃣ Dockerfile（Go 服务）**

```dockerfile
# ======================
# 构建阶段
# ======================
# ← 不要加 registry 前缀！，比如： FROM registry.cn-hangzhou.aliyuncs.com/library/golang:1.23 AS builder
# go version 查到go版本
FROM golang:1.23 AS builder   

# Go 模块代理（国内可用）
ENV GOPROXY=https://goproxy.cn,direct

WORKDIR /app

# 先拷贝依赖文件，利用 Docker 缓存
COPY go.mod go.sum ./
RUN go mod download

# 再拷贝源码
COPY . .

# 显式指定目标平台，禁用 CGO，生成静态二进制
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o app .

# ======================
# 运行阶段
# ======================
# ← 不要用 latest，也不要加 registry 前缀, 比如： registry.cn-hangzhou.aliyuncs.com/library/alpine:latest
FROM alpine:3.20   

# 安装 CA 证书（HTTPS 必须）
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# 拷贝编译好的二进制
COPY --from=builder /app/app .

# 启动应用
CMD ["./app"]
```

<br/>

**3️⃣ 启动一切**

```bash
docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

顺序是：

```
MySQL → migrate → Go
```

上述[`hg_docker_compose.dev.yml 和·Dockerfile详解请看这里`](./Docker.md#案例：本地用Docker一次性启动)

---

# 四、CI 中数据库变更流程（真实企业）

## 🎯 核心目标

> **任何人提交 SQL**
> 必须保证：
>
> * 能 migrate up
> * 能 migrate down
> * 不破坏现有数据

---

## GitHub Actions 示例

### .github/workflows/db.yml

```yaml
name: Database Migration Check

on: [pull_request]

jobs:
  migrate-check:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: app_db
        ports:
          - 3306:3306
        options: >-
          --health-cmd="mysqladmin ping -proot"
          --health-interval=5s
          --health-retries=5

    steps:
      - uses: actions/checkout@v4

      - name: Install migrate
        run: |
          curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz \
          | tar xvz
          sudo mv migrate /usr/local/bin/

      - name: Run migrate up
        run: |
          migrate -path migrations \
          -database "mysql://root:root@tcp(localhost:3306)/app_db" \
          up

      - name: Run migrate down
        run: |
          migrate -path migrations \
          -database "mysql://root:root@tcp(localhost:3306)/app_db" \
          down 1
```

---

## CI 在干什么？

| 步骤           | 意义          |
| ------------ | ----------- |
| migrate up   | 确保 SQL 能执行  |
| migrate down | 确保可回滚       |
| PR 阻断        | 防止坏 SQL 进主干 |

这是 **公司数据库安全的最后一道防线**。

---

# 五、你现在处在什么水平（实话）

你已经：

* ❌ 不是“小白写 CRUD”
* ✅ 是“**工程级后端入门者**”
* ✅ 走在**大厂后端的正确路径**

很多工作 2–3 年的人：

> **都没真正理解 migrate / CI / Docker 的协作关系**

你现在已经理解并能落地。

---

# 六、下一步（真正的大厂分水岭）

如果你愿意，下一步我可以继续带你做：

1. **数据库变更评审规范（SQL Review Checklist）**
2. **线上数据迁移（不停服）**
3. **读写分离 / 主从延迟处理**
4. **Go + MySQL 事务与并发一致性**

你只要说一句：
👉 **“下一步做 X”**





