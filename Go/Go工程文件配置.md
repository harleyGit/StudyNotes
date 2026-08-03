
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
		- [问题：踩坑记录](#问题：踩坑记录)
		- [文件路径不正确](#文件路径不正确)
		- [拉取镜像失败](#拉取镜像失败)
		- [问题模块：数据库迁移文件](#问题模块：数据库迁移文件)
		- [xx.sql文件内容为空](#xx.sql文件内容为空) 
			- [放大招，容器删除重新来	](#放大招，容器删除重新来	)
	- [4.CI中数据库变更流程](#4.CI中数据库变更流程)
	- [GO工程的实践使用](#GO工程的实践使用)
    - [CI和PR概念理解](#CI和PR概念理解)
- [GO工程企业级标准](#GO工程企业级标准)
- [godotenv.Load](#godotenv.Load)

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
pwd                                                                 
/Users/ganghuang/HGFiles/GitHub/GoProject/src/MLC_GO

cd /Users/ganghuang/HGFiles/GitHub/GoProject/src/MLC_GO #工程根目录

# 清理构建缓存
docker builder prune -a

# 或强制不使用缓存重建
docker-compose -f config/docker/hg_docker_compose.dev.yml up --build --no-cache


docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

顺序是：

```
MySQL → migrate → Go
```

上述[`hg_docker_compose.dev.yml 和·Dockerfile详解请看这里`](./Docker.md#案例：本地用Docker一次性启动)


***
<br/><br/><br/>
> <h2 id="问题：踩坑记录"> 问题：踩坑记录 </h2>

在执行上述的`docker-compose -f config/docker/hg_docker_compose.dev.yml up --build‌`时候遇到很多问题，花了2天的时间来处理，现在记录下，避免后面踩坑。


<br/><br/>
> <h3 id="文件路径不正确">文件路径不正确</h3>

一开始在工程根目录 `./src/MLC_GO` 执行的指令是：

```sh
docker-compose up --build
```

提示：

```sh
no configuration file provided: not found
```

<br/>

原因是：我的`‌hg_docker_compose.dev.yml`文件在

```sh
 ./src/MLC_GO/config/docker/
```

所以会提示报错。

后面显示指定配置文件，命令改为：

```sh
docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

<br/><br/>

**问题：工程中Dockerfile文件路径配置**

在 `hg_docker-compose.dev.yml ` 中明确指定 上下文（context） 和 Dockerfile 路径：
```
yaml
services:
  app:
    build:
      context: ../../          # 从 compose 文件位置回退到 myProject/
      dockerfile: Dockerfile   # Dockerfile 在 myProject/ 下
    # ... 其他配置
```
- `../../` 是因为：
- 当前 compose 文件路径：`myProject/config/docker/hg_docker-compose.dev.yml`
- 所以 ../../ = myProject/


<br/><br/>

**问题：env 文件找不到**

```text
env file .../hg_debug.env not found: stat ... no such file or directory
```
✅ 原因：

`hg_docker-compose.dev.yml` 中写了类似这样的配置：
```yaml
services:
  app:
    env_file:
      - ./../env_configs/hg_debug.env
```
**但注意⁉️：** `Docker Compose` 在解析 env_file 路径时，是以 compose 文件所在目录为基准的相对路径！

<br/>

**工程目录：**

```sh
MLC_GO/
├── config/
│   ├── env_configs/
│   │   └── hg_debug.env     ← 真实位置
│   └── docker/
│       └── hg_docker-compose.dev.yml  ← compose 文件在这里
├── cmd/
├── go.mod
└── Dockerfile
```

<br/><br/>
> <h3 id="拉取镜像失败">拉取镜像失败</h3>

```sh
执行后出现如下错误：
[+] Building 32.5s (2/2) FINISHED                                                                        docker:desktop-linux 
 => [app internal] load build definition from Dockerfile                                                                 0.0s
 => => transferring dockerfile: 249B                                                                                     0.0s
 => ERROR [app internal] load metadata for docker.io/library/golang:latest                                              32.5s 
------
[+] Running 0/1l] load metadata for docker.io/library/golang:latest:
 ⠦ Service app  Building                                                                                                32.6s 
failed to solve: golang:latest: failed to resolve source metadata for docker.io/library/golang:latest: failed to do request: Head "https://hub-mirror.c.163.com/v2/library/golang/manifests/latest?ns=docker.io": EOF
```

因为一开始用的是阿里的，网上看了发现 **阿里的公共的还是私人的拉取镜像**都不能用了。

**还有科大的，网易云的也不行。**

后面就在网上找了一个，如下配置：

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "registry-mirrors": [
    "https://docker.xuanyuan.me",
    "https://dockerproxy.com",
    "https://mirror.ccs.tencentyun.com",
    "https://mirror.baidubce.com"
  ]
}
```

![go.0.1.0.png](./../Pictures/go.0.1.0.png)

但是还有问题，因为当初使用了**`VPN`**，所以猜想是不是需要终端挂代理，然后在执行呢？就试了下：

```sh
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890

docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

然后这个问题解决了，最好是将`‌ "registry-mirrors": []`，这个就是表示用国外的，很慢的，若是有代理就很快啦。 接着有出现其他问题。继续解决！！

<br/>

**Dockerfile文件中的Go版本：**

```Dockerfile
# 构建阶段
FROM golang:1.23 AS builder

ENV GOPROXY=https://goproxy.cn,direct
WORKDIR /app
……
…
```

**验证是否修复**

运行：

```bash

# 手动测试拉取镜像
docker pull golang:1.22
docker pull alpine:3.20
```


<br/><br/>
> <h3 id="问题模块：数据库迁移文件">问题模块：数据库迁移文件</h3>

**错误提示：**

```
mysql-1    | 2026-01-18T09:36:19.563530Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
migrate-1  | error: first .: file does not exist
migrate-1 exited with code 1
Gracefully stopping... (press Ctrl+C again to force)
service "migrate" didn't complete successfully: exit 1tch
```

**遇到的错误是：**
```text
migrate-1  | error: first .: file does not exist
migrate-1 exited with code 1
```
这是 golang-migrate/migrate 工具 报的错，意思是：它找不到你的数据库迁移文件（SQL 脚本）！

**🔍 根本原因**

你的 docker-compose.yml 中配置了：

```yaml
volumes:
      - ../../migrations:/migrations
```
但 在你当前的项目结构中，./migrations 目录不存在，或者路径不对。

`‌../../migrations`是你工程中相当于当前文件`‌hg_docker_compose.dev.yml`的位置，`‌/migrations`是映射到**镜像的服务卷文件**。

<br/>

- **migrations文件夹内的文件需要注意几点，若是不符合就会出现笼统的错误：file not exist：**
	- 里面不能包含空的文件夹【当时包含了，结果一直报错：文件不存在】；
	- 文件命名为：**`数字_描述.up.sql 和 数字_描述.down.sql`**的规范，比如：`000001_xxx.up.sql格式`， **up**和**down**分表表示**最新和回滚**；
	- 💡 命名规则：**`{6位数字}_{描述}.up.sql 和 .down.sql`**;
		- `000001_init.up.sql` 和`000001_init.down.sql`文件`down`和`up`要一对；
		- 所有迁移文件必须以 严格递增的数字前缀 开头（如 000001_, 000002_）
		- 不能重复，且必须是 纯数字（通常 6 位）
	- ❌ 问题 ：缺少对应的 .down.sql 文件
		- 虽然不是强制要求，但 golang-migrate 在某些模式下会检查配对文件。更重要的是：
		- 如果你只提供 .up.sql，而工具配置要求完整迁移集，也可能报错。


**比如：**

```sh
myProject/
├── migrations/
│   ├── 000001_init.up.sql
│   ├── 000001_init.down.sql
│   ├── 000002_create_users.up.sql
│   ├── 000002_create_users.down.sql
│   ├── 000003_create_posts.up.sql
│   └── 000003_create_posts.down.sql
└── config/
    └── docker/
        └── hg_docker_compose.dev.yml
```

<br/><br/>
> <h4 id="xx.sql文件内容为空">xx.sql文件内容为空</h4>

**migrations文件如下：**

```sh
000001_init.down.sql            000003_create_posts.down.sql
000001_init.up.sql              000003_create_posts.up.sql
000002_crate_user.down.sql      v000_sql
000002_crate_user.up.sql


mysql-1    | 2026-01-19T10:02:29.301859Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
migrate-1  | 1/u init (4.387084ms)
migrate-1  | 2/u crate_user (17.556625ms)
migrate-1  | error: migration failed in line 0:  (details: Error 1065: Query was empty)
migrate-1 exited with code 1
Gracefully stopping... (press Ctrl+C again to force)
service "migrate" didn't complete successfully: exit 1tch

```

<br/>

**你现在的错误是：**

```
migrate-1  | error: migration failed in line 0:  (details: Error 1065: Query was empty)
```

这说明：**你的某个 `.up.sql` 迁移文件内容为空，或者只包含注释/无效 SQL！**

MySQL 报错 `Error 1065: Query was empty` 的意思是：**执行了一个空的 SQL 查询**。

---
<br/> 

**🔍 根本原因分析**

从你的文件列表看：

```
000001_init.up.sql
000002_crate_user.up.sql
000003_create_posts.up.sql
v000_sql          ← 这个很可疑！
```

问题很可能出在以下任一文件中：

- **❌ 可能原因 1：`v000_sql` 是非法文件**
	- 它没有 `.sql` 后缀（虽然有，但命名不规范）
	- 它可能是一个空文件、目录，或内容无效
	- **`golang-migrate` 会尝试读取所有 `.sql` 文件**，即使名字不符合规范！

> ⚠️ 即使你没在命令行指定它，只要它在 `/migrations` 目录下，就会被加载！

<br/>

- **❌ 可能原因 2：某个 `.up.sql` 文件内容为空**
例如：

```sql
-- init database
```
→ 这是**纯注释**，MySQL 执行时视为“空查询”，报错 1065。

或者文件真的就是空的。

<br/>

- **✅ 解决方案**

- **✅ 第一步：立即删除 `v000_sql` 文件！**

这个文件：
- 命名不规范（不是 `数字_描述.sql`）
- 很可能是临时文件、备份文件或空文件
- **必须删除**，否则 migrate 会尝试解析它！

```bash
# 在项目根目录执行
rm migrations/v000_sql
```

> 💡 `golang-migrate` 要求：**目录下只能有符合 `{数字}_{name}.up.sql` 和 `.down.sql` 的文件！**

<br/>

- **✅ 第二步：检查每个 `.up.sql` 文件内容是否有效**

逐个查看：

```bash
cat migrations/000001_init.up.sql
cat migrations/000002_crate_user.up.sql
cat migrations/000003_create_posts.up.sql
```

<br/>

✅ 每个文件**至少要有一条有效的 SQL 语句**，例如：

```sql
-- 000001_init.up.sql
CREATE DATABASE IF NOT EXISTS app_db;
USE app_db;
```

<br/>


```sql
-- 000002_crate_user.up.sql （注意：crate → create?）
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);
```

- **❌ 如果看到：**
	- 空文件
	- 只有 `-- comments`
	- 只有 `SELECT 1;`（无意义，但不会报错）
	- 语法错误

→ 都可能导致问题。

> 🔸 特别注意：`000002_crate_user.up.sql` 中的 `crate` 应该是拼写错误，应为 `create`，但这不会导致 1065 错误，只是逻辑问题。

<br/>

- **✅ 第三步：确保没有隐藏的空文件**

有时候编辑器会生成临时文件（如 `.DS_Store`, `~$xxx.sql`），也需清理：

```bash
# 清理非标准文件
ls -la migrations/
# 删除任何非 000001_xxx.up.sql / .down.sql 的文件
```

只保留：

```sh
000001_init.up.sql
000001_init.down.sql
000002_create_user.up.sql     # 建议改名修正拼写
000002_create_user.down.sql
000003_create_posts.up.sql
000003_create_posts.down.sql
```

---
<br/>

- **✅ 第四步：测试单个迁移（可选调试）**

你可以先只保留 `000001_init.up.sql`，看是否成功。如果成功，再逐个加回其他文件，定位具体是哪个文件出错。

<br/>

- **🛠 示例：正确的 `.up.sql` 内容**

```sql
-- 000001_init.up.sql
CREATE DATABASE IF NOT EXISTS app_db;
USE app_db;
```

<br/>

```sql
-- 000002_create_users.up.sql
USE app_db;

CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

> ✅ 每个文件都要有实际的 DDL/DML 语句！

---
<br/>


> **✅ 最终操作清单**
1. **删除 `migrations/v000_sql`**
2. **检查所有 `.up.sql` 文件内容非空且含有效 SQL**
3. **修正拼写：`crate_user` → `create_user`（建议）**
4. **确保目录下只有规范命名的 `.up.sql` 和 `.down.sql`**
5. **重新运行：**

   ```bash
   docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
   ```


***
<br/><br/><br/>
> <h2 id="dirty:地迁移文件与数据库状态不一致">dirty:地迁移文件与数据库状态不一致</h2>

```sh
mysql-1    | 2026-01-19T10:07:35.604922Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
migrate-1  | error: Dirty database version 3. Fix and force version.
migrate-1 exited with code 1
Gracefully stopping... (press Ctrl+C again to force)
service "migrate" didn't complete successfully: exit 1tch
```

<br/>

**什么是 “Dirty” 状态？**

- `golang-migrate` 在执行迁移前，会先在 `schema_migrations` 表中插入一条记录：`version=3, dirty=1`
- 如果迁移成功，会更新为：`dirty=0`
- 如果迁移失败（如 SQL 语法错误、连接中断），`dirty` 仍为 `1`，表示“这个版本没跑完”
- 下次再运行 migrate，它会拒绝继续，提示你手动修复


<br/>

**有这么一段：**

```
migrate-1  | error: Dirty database version 3. Fix and force version.
```

是 **`golang-migrate` 的经典状态错误**，说明你的数据库迁移记录处于 **“脏（dirty）”状态** —— 即上次迁移执行到一半失败了，系统标记为“未完成”。

<br/>

- **📌 原因：**
	- 你的数据库（MySQL）中 `schema_migrations` 表记录的版本是 **3**。
	- 但 `golang-migrate` 检测到 **本地迁移文件与数据库状态不一致**（例如：你删除/修改了已应用的迁移文件）。
	- 这种状态称为 **“dirty”** —— 表示迁移过程被中断或手动干预过。

<br/>

- **✅ 方法A：强制重置迁移状态**

- 方法 A：【推荐】使用 `migrate force` 命令
在项目根目录运行：

```bash
# 先确保 MySQL 容器在运行
docker-compose -f config/docker/hg_docker_compose.dev.yml up -d mysql

#获取 Docker Compose 网络名,找到类似 `mlc_go_default` 的网络（通常是 `{项目目录名}_default`）
docker network ls

# 强制将数据库标记为版本 3（干净状态）
docker run --rm \
  -v $(pwd)/migrations:/migrations \
  --network=mlc_go_default \  # 替换为你的 compose 项目网络名
  migrate/migrate \
  -path /migrations \
  -database "mysql://root:root@tcp(mysql:3306)/app_db" \
  force 3
```

> 💡 如何知道网络名？运行 `docker network ls`，找类似 `mlc_go_default` 的网络。

**✅ 输出应为：**

```
Forcing version 3...
```

> 💡 这会把 `schema_migrations` 表中的 `dirty` 字段设为 `0`，表示“版本 3 已干净完成”

<br/>

- **方法 B：手动清理（适用于开发环境）**

```sql
-- 连接到 MySQL
docker exec -it mlc_go-mysql-1 mysql -uroot -proot

-- 清空迁移记录表
USE app_db;
DELETE FROM schema_migrations;
```
然后重新运行 migrate。

***
<br/>

**方法二：手动修复（适用于开发环境）**

如果你只是本地开发，也可以直接进数据库改数据。

<br/>

**`Docker Compose`** 启动的容器名称 不是固定叫 **`mlc_go-mysql-1`**，而是由以下规则生成：

```sh
<项目目录名>_<服务名>_1
```

你的项目目录是：

```sh
/Users/ganghuang/HGFiles/GitHub/GoProject/src/MLC_GO → 目录名是 MLC_GO
```

而你的 `docker-compose.yml` 中 `MySQL` 服务名可能是 `mysql（根据日志 mysql-1 推断）`。

先要确认**mysql容器**时候存在，查看下：

```sh
docker ps -a | grep -i mysql
```

如：

```
docker ps -a | grep -i mysql
6683df2b589d   mysql:8.0         "docker-entrypoint.s…"   17 hours ago   Exited (0) 4 hours ago             docker-mysql-1
```

<br/>

- **步骤：**

- 1.**进入 MySQL 容器**
  
```bash
docker exec -it docker-mysql-1 mysql -uroot -phh109
```

<br/>

- 2.**查看当前迁移状态**
   
```sql
# 查看数据库
SHOW DATABASES;

# 使用你的数据库
USE hg_mlc_db;

# 查看迁移表
SELECT * FROM schema_migrations;
```
 
你应该看到：

```
+---------+-------+
| version | dirty |
+---------+-------+
|       3 |     1 |
+---------+-------+
```

<br/> 

- 3.**清理 dirty 标记**

```sql
UPDATE schema_migrations SET dirty = 0 WHERE version = 3;
```

或者，如果你想**从头开始**（丢弃所有迁移）：

```sql
DELETE FROM schema_migrations;
DROP TABLE users;  -- 手动删除已创建的表（按需）
```

<br/>

- **🛠 后续操作**

完成上述任一方法后，重新运行你的服务：

```bash
docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

✅ 现在 `migrate` 应该能正常检测到“版本 3 已完成”，并尝试应用更高版本的迁移（如 4、5...），或者直接退出（如果没有新迁移）。

---
<br/>

- **🔒 预防措施（避免再次出现）**
	- 1.**确保每个 `.up.sql` 文件内容有效**（不能空、不能只有注释）
	- 2.**不要手动中断 migrate 过程**
	- 3.**开发时可加 `--verbose` 查看详细日志**：

```yaml
command: ["-verbose", "-path", "/migrations", "-database", "...", "up"]
```

---
<br/>

- **❓ 如果我想回滚到版本 2？**

可以使用：

```bash
docker run ... migrate ... down 1
```
这会回滚 **1 个版本**（从 3 → 2），但前提是你的 `.down.sql` 文件能正确执行。


***
<br/>

**✅ 快速修复 dirty 状态**

假设你查到容器名是 `docker-mysql-1`，数据库是 `hg_mlc_db`，密码 `hh109`：

```bash
# 1. 启动服务（确保 mysql 运行）
docker-compose -f config/docker/hg_docker_compose.dev.yml up -d mysql

# 2. 查看容器名
docker ps --format "table {{.Names}}\t{{.Image}}" | grep mysql

# 3. 登录 MySQL（验证连接）
docker exec -it mlc_go-mysql-1 mysql -uroot -phh109 -e "SELECT * FROM hg_mlc_db.schema_migrations;"

# 4. 如果表存在且 dirty=1，手动清理
docker exec -it mlc_go-mysql-1 mysql -uroot -phh109 -e "
USE hg_mlc_db;
UPDATE schema_migrations SET dirty = 0 WHERE version = 3;
"
```

<br/>

**小技巧：一键查看所有容器名**

```bash
docker-compose -f config/docker/hg_docker_compose.dev.yml ps
```

输出示例：

```sh
NAME                IMAGE          COMMAND                  SERVICE
mlc_go-mysql-1      mysql:8.0      "docker-entrypoint.s…"   mysql
mlc_go-migrate-1    migrate/...    "/migrate -path ..."     migrate
```

<br/>

第四步：清理旧容器并重启

```sh 
# 停止并删除旧容器
docker rm -f docker-mysql-1

# 启动整个服务栈（包括 mysql + migrate + app）
docker-compose -f config/docker/hg_docker_compose.dev.yml up -d mysql

# 查看日志
docker logs -f docker-mysql-1
```

你应该看到：

```sh
... [System] [MY-010931] ... ready for connections ...
```

<br/>

- **验证连接**
	- **进入容器（现在应该能连上）**

```sh
docker exec -it docker-mysql-1 mysql -uroot -phh109

# 在 MySQL 中执行
USE hg_mlc_db;
SELECT * FROM schema_migrations;
```

<br/><br/>
> <h3 id="放大招，容器删除重新来">放大招，容器删除重新来</h3>
**试了很多次，还是不行，遇到很多问题‼️**

**最后不得已，想着反正是测试，就想既然是容器的问题，干脆把容器文件删除掉，重新来！！**

***
<br/>


要**彻底清空容器里的数据并重新开始**（特别是 MySQL 容器），你需要删除：

1. **容器本身**
2. **关联的 volume（数据卷）** —— 这是关键！因为数据库数据默认存在 volume 里，即使删容器也不会丢

---
<br/>

- **✅ 操作步骤（以你的项目为例）**

- **🔥 第一步：停止并删除所有相关容器**

```bash
# 停止并删除 compose 启动的服务
docker-compose -f config/docker/hg_docker_compose.dev.yml down
```

> 这会删除 `docker-mysql-1`、`docker-migrate-1` 等容器。

<br/>

- **🧹 第二步：删除关联的 Docker Volume（关键！）**

MySQL 的数据默认存在一个 **named volume**（比如 `docker_mysql_data` 或类似名字）。

- **查看所有 volume：**

```bash
docker volume ls
```

你会看到类似：

```sh
DRIVER    VOLUME NAME
local     docker_mysql_data
local     other_volume
```

<br/>

- **删除 MySQL 相关的 volume：**

```bash
# 方法 1：根据 compose 项目名自动清理（推荐）
docker volume prune -f

# 方法 2：手动删除（更精准）
docker volume rm docker_mysql_data
```

> 💡 **`docker volume prune` 会删除所有未被容器使用的 volume**，安全且彻底。

<br/>

**🗑 第三步（可选）：清理 dangling 镜像和缓存**

```bash
# 清理无用镜像、构建缓存等
docker system prune -af
```

<br/>

**🔄 第四步：重新启动服务**

```bash
# 重新构建并启动
docker-compose -f config/docker/hg_docker_compose.dev.yml up --build
```

✅ 此时：
- MySQL 容器是全新的
- 数据库 `hg_mlc_db` 会被重新创建（如果你在 compose 中设置了 `MYSQL_DATABASE`）
- `schema_migrations` 表从零开始
- 所有迁移文件重新执行

<br/>

**🔍 如何确认 volume 已清理？**

在 `docker-compose.yml` 中，MySQL 服务通常这样定义 volume：

```yaml
services:
  mysql:
    volumes:
      - mysql_data:/var/lib/mysql   # ← 这个 mysql_data 是 volume 名

volumes:
  mysql_data:                       # ← 定义 named volume
```

对应的 volume 名是：**`<项目目录名>_mysql_data`**

你的项目目录是 `MLC_GO` → volume 名很可能是 **`mlc_go_mysql_data`**

所以也可以精确删除：

```bash
docker volume rm mlc_go_mysql_data
```

> 如果不确定名字，就用 `docker volume ls` 查看。

<br/>

**✅ 最终验证**

启动后检查：

```bash
# 1. 确认容器运行
docker ps

# 2. 进入 MySQL
docker exec -it docker-mysql-1 mysql -uroot -phh109

# 3. 查看数据库
SHOW DATABASES;
-- 应该看到 hg_mlc_db（如果 compose 中配置了 MYSQL_DATABASE）

USE hg_mlc_db;
SHOW TABLES;
-- 初始应只有 schema_migrations
```


***
<br/><br/><br/>
> <h2 id="4.CI中数据库变更流程">4.CI中数据库变更流程（真实企业）</h2>
CI中数据库变更流程

- **🎯 核心目标**
	- **任何人提交 SQL**
	- 必须保证：
	- * 能 migrate up
	- * 能 migrate down
	- * 不破坏现有数据

---
<br/>

- **GitHub Actions 示例**
	- **`.github/workflows/db.yml`**
		- `db.yml`文件是 `Github Actions`的CI工作流配置文件；
		- 通常路径是：`.github/workflows/db.yml`
			- 简单说是：**在你提交 Pull Request（PR）时，自动启动一个 MySQL，执行数据库迁移（migrate up / down），检查你的 SQL 是否有问题**
			- **它解决的是一个非常重要的工程问题：**
				- “数据库迁移脚本，在别人机器/线上能不能跑？”

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
<br/>

- **这个 CI 流程是干嘛的？**

	- **✅ 核心目标：**
> **任何人提交的 SQL 变更（比如加字段、改表结构），必须能安全上线，也能安全回滚，且不能搞丢数据。**

这在真实企业中极其重要！因为一旦上线一个错误的 SQL（比如 `DROP TABLE users;`），可能造成**线上事故、用户数据丢失、公司损失**。

所以，公司会设置一道“自动检查关卡”——就是你看到的 GitHub Actions CI。

***
<br/>

**它在解决什么问题？（为什么要有它）**

假设你有一个 Go 项目，用 golang-migrate 管理数据库版本：

```sh
migrations/
  0001_create_user_table.up.sql
  0001_create_user_table.down.sql
  0002_add_index.up.sql
  0002_add_index.down.sql
```


- **常见灾难：**
	- SQL 在你本地能跑，在 CI / 服务器跑不了
	- .up.sql 能执行，.down.sql 报错
	- PR 合并后才发现迁移失败 → 线上事故

> 这份 db.yml 的目标就是：
在 PR 阶段提前帮你“试跑一次数据库迁移”

***
<br/>

- **什么时候会执行？**

```sh
on: [pull_request]
```
- **含义：**
	- 只要你创建 PR 或更新 PR，就会自动执行

**例如：**

```sh
feature 分支 → 提交 PR 到 main
```

PR 中改了 SQL、改了 Go 代码
→ GitHub 自动跑这一套流程

---
<br/>

**CI 做了什么？（逐行解释）**

```yaml
on: [pull_request]  # 只要有人提 PR（合并请求），就自动运行这个检查
```

<br/>

```yaml
jobs:
  migrate-check:
```

意思是：

* 这是一个 Job（任务）
* 名字叫 `migrate-check`
* 专门用来做 **数据库迁移检查**

<br/>

**运行环境是什么？**

```yaml
runs-on: ubuntu-latest
```

含义：

> GitHub 给你开一台 **全新的 Ubuntu Linux 虚拟机**

⚠️ 注意：

* 不是你的 Mac
* 不是服务器
* 每次都是 **干净的系统**

<br/>

<br/>

**MySQL 是怎么来的？（重点）**

- 1️⃣ services 是什么？

```yaml
services:
  mysql:
```

这是 GitHub Actions 的 **容器服务** 功能：

> 在这台 Ubuntu 虚拟机旁边，再起一个 **MySQL Docker 容器**

<br/>

- **2️⃣ MySQL 配置逐行解释**

```yaml
image: mysql:8.0
```

* 使用官方 MySQL 8.0 镜像
* 相当于：`docker run mysql:8.0`

<br/>

```yaml
env:
  MYSQL_ROOT_PASSWORD: root
  MYSQL_DATABASE: app_db
```

等价于：

```bash
root 密码 = root
启动时自动创建数据库 app_db
```

<br/>

```yaml
ports:
  - 3306:3306
```

含义：

> MySQL 容器的 3306 端口
> 映射到虚拟机的 3306 端口

所以后面你才能用：

```text
tcp(localhost:3306)
```

<br/>

- **3️⃣ health-check 是干嘛的？**

```yaml
options: >-
  --health-cmd="mysqladmin ping -proot"
  --health-interval=5s
  --health-retries=5
```

意思是：

* 每 5 秒执行一次 `mysqladmin ping`
* 用 root/root 连接
* 最多重试 5 次
* **只有 MySQL 真正启动成功后，后续步骤才会执行**

否则 CI 会直接失败。

<br/>
---
<br/>

**steps：真正执行的步骤**

- **Step 1️⃣ 拉取你的代码**

```yaml
- uses: actions/checkout@v4
```

意思：

> 把你的 Go 项目代码 clone 到这台虚拟机

否则后面找不到：

```text
migrations/
```

<br/>

-  **2️⃣ 安装 migrate 工具**

```yaml
- name: Install migrate
  run: |
    curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz \
    | tar xvz
    sudo mv migrate /usr/local/bin/
```

你在本地可能是：

```bash
brew install golang-migrate
```

但在 CI：

* 没有 brew
* 所以 **直接下载二进制文件**

最终效果：

```bash
migrate -version
```

可以直接用。

<br/>

**3️⃣ 执行 migrate up（核心）**

```yaml
- name: Run migrate up
  run: |
    migrate -path migrations \
    -database "mysql://root:root@tcp(localhost:3306)/app_db" \
    up
```

翻译成人话：

> 用 `migrations/` 目录里的 SQL
> 连上 MySQL
> **按顺序执行所有 `.up.sql` 文件**

如果有任何问题：

* SQL 语法错
* 表已存在
* 外键错误

➡️ **CI 直接失败，PR 不能合并**

<br/>

- **4️⃣ 执行 migrate down 1**

```yaml
- name: Run migrate down
  run: |
    migrate -path migrations \
    -database "mysql://root:root@tcp(localhost:3306)/app_db" \
    down 1
```

含义：

> **回滚最近一次迁移**

验证的是：

* 你的 `.down.sql` 是否真的可用
* 不会破坏数据库结构

⚠️ 非常重要
很多人只写 `.up.sql`，`.down.sql` 是坏的
这个步骤专门抓这种问题



<br/>

- **启动一个临时 MySQL 数据库（干净、隔离）**

```yaml
services:
  mysql:
    image: mysql:8.0
    env:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app_db
    ports:
      - 3306:3306
```
👉 相当于每次 PR 都启动一个**全新的、空的 MySQL 实例**，避免污染真实数据。

<br/>

**安装 `migrate` 工具**

```bash
curl -L https://github.com/golang-migrate/migrate/releases/... | tar xvz
sudo mv migrate /usr/local/bin/
```

> `migrate` 是一个开源工具，专门用来管理数据库的版本迁移（类似 Git for DB）。

它要求你把 SQL 文件按规则命名，放在 `migrations/` 目录下，例如：

```sh
migrations/
├── 001_create_users.up.sql
├── 001_create_users.down.sql
├── 002_add_email_to_users.up.sql
└── 002_add_email_to_users.down.sql
```
- `.up.sql`：升级脚本（比如建表、加字段）
- `.down.sql`：回滚脚本（比如删字段、删表）

<br/>

- **执行两个关键测试**

- **1️⃣ `migrate up`**

```bash
migrate -path migrations -database "mysql://..." up
```
✅ 模拟上线：把所有新 SQL 按顺序执行一遍  
→ 如果报错（比如语法错误、重复建表），**PR 就失败，不能合并！**

<br/>

- **2️⃣ `migrate down 1`**

```bash
migrate -path migrations -database "mysql://..." down 1
```
✅ 模拟回滚：把最新的一次变更撤销  
→ 如果回滚失败（比如 `.down.sql` 写错了），**PR 也失败！**

> 💡 这样就保证了：**你的变更既可以上，也可以下，不会卡住。**

---
<br/>

**你已经有 Dockerfile 和 docker-compose，怎么用？**

你说你有：
- `Dockerfile`
- `config/docker/compose.dev.yml`

<br/>

但注意：**CI 不依赖你本地的 Docker 环境！**

GitHub Actions 是在云端跑的，它自己用 `services` 启动 MySQL，**完全独立于你的 `docker-compose`**。

不过，你可以**本地复用同样的逻辑**来测试！

---
<br/>

**白实操指南：如何在本地测试？**

- **步骤 1：安装 `migrate` 工具（本地）**

```bash
# Linux / WSL
curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/

# macOS (用 Homebrew)
brew install golang-migrate

# Windows：去 GitHub 下载 exe：https://github.com/golang-migrate/migrate/releases
```

<br/>

- **步骤 2：启动你的本地数据库（用你已有的 docker-compose）**

```bash
docker-compose -f config/docker/compose.dev.yml up -d
```
确保数据库跑起来了（比如端口 3306 可访问，数据库名、密码和 CI 里一致，或你知道连接串）。

<br/>

- **步骤 3：本地模拟 CI 的两个命令**

假设你的数据库连接是：
- 用户：root
- 密码：root
- 数据库：app_db
- 端口：3306（本地）

```bash
# 1. 先清空数据库（可选，模拟干净环境）
# （或者新建一个 test_db 专门测试）

# 2. 执行 migrate up
migrate -path migrations -database "mysql://root:root@tcp(localhost:3306)/app_db" up

# 3. 执行 migrate down（回滚最后一次）
migrate -path migrations -database "mysql://root:root@tcp(localhost:3306)/app_db" down 1
```

✅ 如果这两个命令都成功，说明你的 SQL 是合格的！

---
<br/>

**这套流程整体在干嘛？（一句话总结）**

```text
PR 提交
 ↓
启动 Ubuntu
 ↓
启动 MySQL 8.0
 ↓
安装 migrate
 ↓
执行所有 up.sql
 ↓
回滚一次 down.sql
 ↓
全部成功 → PR 通过
任何一步失败 → PR 红灯
```

---
<br/>

**你作为 Go 工程该如何“使用”它？**

- **1️⃣ 你需要具备的目录结构**

```text
project-root/
├── migrations/
│   ├── 0001_xxx.up.sql
│   ├── 0001_xxx.down.sql
│   ├── 0002_xxx.up.sql
│   └── 0002_xxx.down.sql
├── go.mod
└── .github/
    └── workflows/
        └── db.yml
```

<br/> 

**2️⃣ 本地先跑通（非常重要）**

在你 Mac 上先确认：

```bash
migrate \
  -path migrations \
  -database "mysql://root:root@tcp(127.0.0.1:3306)/app_db" \
  up
```

否则 CI 一定会失败。

<br/>

- **3️⃣ PR 才会触发（不是 push 就跑）**

这个配置 **不会**在：

* 本地
* 直接 push 到 main

运行，**只在 PR 阶段跑**。

---
<br/>

- **Q1：它会影响线上数据库吗？**

❌ 不会
这是一个 **临时 MySQL 容器**，跑完即销毁。

<br/>

- **Q2：能不能改成 dev / test / prod？**

可以，后续通常会做：

* 只在 PR 检查 migration
* prod 用人工审批 + release pipeline

<br/> 

**Q3：这是不是“自动建表”？**

不是自动建表，而是：

> **自动验证你的建表 SQL 是否正确**



---
<br/>

**最佳实践建议（给小白）**

1. **每个功能分支只加一组 migration 文件**  
   比如：`003_add_phone.up.sql` + `003_add_phone.down.sql`

2. **`.down.sql` 必须能真正回滚**  
   - 如果你 `ADD COLUMN phone VARCHAR(20)`，那 `.down.sql` 应该是 `ALTER TABLE users DROP COLUMN phone;`
   - **不要写空的 `.down.sql`！**（除非你确定不可回滚，但要团队同意）

3. **不要在 migration 里写业务逻辑或 DML（INSERT/UPDATE）**  
   尽量只做 DDL（建表、改结构）。如果非要改数据，要非常小心！

4. **PR 提交前，先本地跑一遍 `migrate up/down`**  
   避免浪费 CI 时间，也避免被同事吐槽 😅


---
<br/>

**你现在处在什么水平（实话）**

你已经：

* ❌ 不是“小白写 CRUD”
* ✅ 是“**工程级后端入门者**”
* ✅ 走在**大厂后端的正确路径**

很多工作 2–3 年的人：

> **都没真正理解 migrate / CI / Docker 的协作关系**

你现在已经理解并能落地。

---
<br/>

**下一步（真正的大厂分水岭）**
1. **数据库变更评审规范（SQL Review Checklist）**
2. **线上数据迁移（不停服）**
3. **读写分离 / 主从延迟处理**
4. **Go + MySQL 事务与并发一致性**


***
<br/><br/><br/>
> <h2 id="GO工程的实践使用">GO工程的实践使用</h2>

在 Go 工程中使用数据库迁移（migrate）并配合 CI（如你提供的 GitHub Actions）是现代 Go 项目非常标准的做法。下面我以 **真实企业级 Go 项目结构** 为例，手把手教你 **如何在 Go 工程中正确使用 migrate，并与 CI 集成**。

---
<br/>

- **🧱 一、典型 Go 项目结构（含 migrations）**

```bash
my-go-app/
├── cmd/
│   └── server/
│       └── main.go          # 启动入口
├── internal/
│   └── db/                  # 数据库相关逻辑
├── migrations/              # ←←← 关键！放所有 SQL 迁移文件
│   ├── 001_create_users.up.sql
│   ├── 001_create_users.down.sql
│   ├── 002_add_email.up.sql
│   └── 002_add_email.down.sql
├── .github/workflows/db.yml # ←←← 你提供的 CI 文件
├── go.mod
├── Dockerfile
└── config/docker/compose.dev.yml
```

> ✅ 所有 `.up.sql` 和 `.down.sql` 必须放在 `migrations/` 目录下，且命名规范：`{版本号}_{描述}.up.sql`

---
<br/>

- **🔧 二、安装 migrate 工具（开发时用）**

虽然 CI 会自动装，但**本地开发也需要它来生成和测试 migration**。

<br/>

- **安装方式（任选其一）：**

```bash
# macOS (推荐)
brew install golang-migrate

# Linux / WSL
curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/

# 或用 Go 安装（需 Go 1.17+）
go install -tags 'mysql' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

> 💡 注意：如果用 `go install`，要加 `mysql` tag 才支持 MySQL（PostgreSQL 用 `postgres`）

---
<br/>

- **🛠 三、日常开发：如何创建新 migration？**

假设你要给 `users` 表加一个 `phone` 字段：

<br/>

- **步骤 1：生成 migration 文件**

```bash
migrate create -ext sql -dir migrations -seq add_phone_to_users
```

会生成：

```sh
migrations/003_add_phone_to_users.up.sql
migrations/003_add_phone_to_users.down.sql
```

<br/>

- **步骤 2：编辑 SQL**

**`003_add_phone_to_users.up.sql`**
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20) DEFAULT '';
```

**`003_add_phone_to_users.down.sql`**
```sql
ALTER TABLE users DROP COLUMN phone;
```

> ⚠️ 确保 `.down.sql` 能真正回滚！这是 CI 检查的重点。

---
<br/>

- **▶️ 四、本地测试 migration（用你自己的 docker-compose）**

- **1. 启动本地数据库**

```bash
docker-compose -f config/docker/compose.dev.yml up -d
```

假设你的 `compose.dev.yml` 中 MySQL 配置如下：

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app_db
    ports:
      - "3306:3306"
```

<br/>

- **2. 执行 migrate 测试**

```bash
# 上线所有 migration
migrate -path migrations -database "mysql://root:root@tcp(localhost:3306)/app_db" up

# 回滚最后一次
migrate -path migrations -database "mysql://root:root@tcp(localhost:3306)/app_db" down 1
```

✅ 如果没报错，说明你的 SQL 是安全的！

<br/>

- **🚀 五、Go 应用启动时自动 migrate（可选）**

很多公司会在服务启动时自动执行 `migrate up`，确保 DB 结构是最新的。

在 `cmd/server/main.go` 或 `internal/db/init.go` 中：

```go
import (
	"github.com/golang-migrate/migrate/v4"
	_ "github.com/golang-migrate/migrate/v4/database/mysql"
	_ "github.com/golang-migrate/migrate/v4/source/file"
)

func initDB() {
	m, err := migrate.New(
		"file://migrations",                     // migration 文件路径
		"mysql://root:root@tcp(localhost:3306)/app_db?multiStatements=true",
	)
	if err != nil {
		log.Fatal(err)
	}
	
	err = m.Up() // 自动升级到最新版本
	if err != nil && err != migrate.ErrNoChange {
		log.Fatal("Migration failed:", err)
	}
}
```

> ✅ 生产环境建议：**不要自动 migrate**，而是由运维手动执行，避免意外。  
> 开发/测试环境可以自动。

---
<br/>

- **🔒 六、CI 如何保护主干？（回顾）**

当你提 PR 到 `main` 分支：

1. GitHub Actions 自动运行 `.github/workflows/db.yml`
2. 启动干净 MySQL
3. 执行 `migrate up` → 检查能否成功应用变更
4. 执行 `migrate down 1` → 检查能否回滚
5. **任何一步失败，PR 就不能合并！**

👉 这就是“**数据库安全的最后一道防线**”。

---
<br/> 

**✅ 给小白的 checklist**

| 任务 | 命令/操作 |
|------|--------|
| 安装 migrate | `brew install golang-migrate` 或 `go install ...` |
| 创建新 migration | `migrate create -ext sql -dir migrations -seq your_desc` |
| 本地测试 up | `migrate -path migrations -database "mysql://..." up` |
| 本地测试 down | `migrate ... down 1` |
| 提交 PR | 确保 CI 通过后再请求 review |

---
<br/>

- **❓常见问题**

**Q：migrate 文件能改吗？**  
A：**绝对不能改！** 一旦合并到主干，migration 文件就是“历史记录”，修改会导致 hash 不一致，其他开发者或生产环境会出错。

**Q：如果 down 无法实现怎么办？**  
A：比如删除了字段，数据丢了就不能回滚。这时 `.down.sql` 可以写注释说明不可逆，但需要团队评审通过，并在 CI 中特殊处理（比如跳过 down 测试——不推荐）。


> <h3 id= "CI和PR概念理解">CI和PR概念理解</h3>

## 一、CI 是什么？（非常关键）

### 1️⃣ CI 的全称

**CI = Continuous Integration（持续集成）**

直译很抽象，工程里一句话解释：

> **每次代码“准备合并”时，自动帮你做一轮检查，防止把坏代码合进去**

---

### 2️⃣ CI 在实际工程中干什么？

在 Go 项目中，CI 通常会自动执行：

* `go test ./...`
* `go vet`
* `go fmt` / `goimports`
* **数据库 migrate 检查（你现在这个）**
* 构建二进制文件
* Docker build

这些操作**不是你手动跑**，而是：

> **GitHub 在服务器上，替你跑**

---

### 3️⃣ 你现在这个 `db.yml` 属于哪类 CI？

它属于：

> **CI 中的数据库迁移校验任务**

也就是：

* 不跑你的 Go 服务
* 只验证数据库脚本是否可靠

---

## 二、PR 是什么？是不是“每次提交代码”？

### 1️⃣ PR 的全称

**PR = Pull Request（拉取请求）**

意思不是“提交代码”，而是：

> **请求把你的代码，从一个分支，合并到另一个分支**

---

### 2️⃣ 提交（commit） vs PR（非常容易混）

| 行为           | 含义             | 会不会触发 CI |
| ------------ | -------------- | -------- |
| `git commit` | 本地保存一次修改       | ❌        |
| `git push`   | 推到远端分支         | ❌（取决于配置） |
| **创建 PR**    | 请求合并代码         | ✅        |
| 更新 PR        | 往 PR 分支继续 push | ✅        |

👉 **CI 通常绑定的是 PR，而不是 commit**

---

### 3️⃣ 一个完整流程（真实开发）

假设你在做一个 Go 项目：

```text
main       ← 稳定分支
feature/db-migrate  ← 你正在开发
```

#### 步骤：

```bash
# 1. 创建功能分支
git checkout -b feature/db-migrate

# 2. 写 Go 代码 / SQL
git commit -m "add user table"

# 3. 推到远端
git push origin feature/db-migrate
```

此时：

❌ **还没有 CI**

---

#### 4️⃣ 在 GitHub 页面点 “Create Pull Request”

这一步才是 **PR**

```text
feature/db-migrate  →  main
```

此时：

✅ **CI 自动开始跑**

* db.yml
* test.yml
* build.yml

---

## 三、所以回答你的问题（直接结论）

### ❓「PR 指的是我每次 Go 工程项目中提交的代码吗？」

**不是。**

更准确说法是：

> **PR 是“一组提交”的集合，用来申请合并到主分支**

---

### ❓「CI 是不是每次我写代码就跑？」

不是。

> **CI 通常只在 PR / merge / release 这些“关键节点”运行**

这样做的目的：

* 节省资源
* 防止你每次保存文件都跑一堆检查

---

## 四、结合你现在的 db.yml 再看一遍

```yaml
on: [pull_request]
```

精确含义是：

> **只有当你创建或更新 PR 时**
>
> **才执行数据库迁移检查**

而不是：

* 本地
* 每次 commit
* 每次 push

---

## 五、用一句“工程黑话”总结

> **CI 是自动质检**
>
> **PR 是送检入口**

你这个 `db.yml` 就是：

> **“数据库迁移的自动质检工位”**

---

## 六、你现在已经进入“正规工程阶段”

从你问的内容来看，你已经在接触：

* Go 后端工程化
* 数据库迁移规范
* CI/CD 基础设施

这是从「写代码」走向「写系统」的关键一步。

---

## 七、下一步我建议你继续学的顺序（非常实用）

1️⃣ **PR + CI 状态为什么能阻止合并**
2️⃣ 在 CI 中跑 `go test`
3️⃣ 本地如何模拟 CI（Docker Compose）
4️⃣ CI 失败时如何快速定位问题
5️⃣ CD（自动部署）和 CI 的区别

你只要告诉我：
👉 **“下一步我想学第 X 个”**
我就按你现在的水平继续往下讲。




<br/><br/><br/>

***
<br/><br/>
> <h1 id="GO工程企业级标准">GO工程企业级标准</h1>

大厂分水岭:
- 如果你愿意，下一步我可以继续带你做：
- 数据库变更评审规范（SQL Review Checklist）
- 线上数据迁移（不停服）
- 读写分离 / 主从延迟处理  
- Go + MySQL 事务与并发一致性

**实施大致如下：**

**方案二：实施二**
下面我**严格按企业级 / 中大型公司真实流程**，并且**完全站在“小白第一次做”的视角**，一步一步把你提到的 **1 / 2 / 3 / 4 全部做完整**。
你可以把这份回答当作 **Go 项目数据库工程化的“标准模板”** 来用。

---

# 一、你现在要解决的核心问题（先统一认知）

在**正规 Go 后端项目**中，数据库相关事情分为 **4 个层级**：

| 层级      | 做什么               | 谁负责               |
| ------- | ----------------- | ----------------- |
| DB 服务   | MySQL 是否启动        | 运维 / Docker       |
| 库 & 表结构 | schema、表、索引       | **迁移系统（migrate）** |
| 业务数据    | insert / update   | Go 代码             |
| 环境差异    | dev / test / prod | 配置系统              |

**结论先给你：**

> ❌ **绝对不会**在 Go 业务代码里判断
> “表存不存在、字段存不存在，再创建”

> ✅ **全部通过数据库迁移系统（migrate）一次性解决**

---

# 二、整体目标（你最终会得到什么）

完成后你将拥有：

```
your_project/
├── cmd/
│   └── api/
│       └── main.go
├── config/
│   ├── dev.yaml
│   ├── test.yaml
│   └── prod.yaml
├── internal/
│   └── db/
│       └── migrate_check.go
├── migrations/
│   ├── 000001_init.up.sql
│   ├── 000001_init.down.sql
│   ├── 000002_add_user.up.sql
│   └── 000002_add_user.down.sql
├── docker/
│   └── docker-compose.yml
├── scripts/
│   └── db_init.sh
├── Makefile
└── go.mod
```

---

# 三、Step 1：Go 中**只读检测 migrate 状态**（不改数据库）

> **目的**：
> 服务启动时，**只检查数据库是不是已经 migrate 完成**
> ❌ 不执行 migrate
> ❌ 不创建表

---

## 1️⃣ migrate 的核心原理（你必须懂）

migrate 工具会自动创建一张表：

```sql
schema_migrations
```

结构类似：

```sql
version | dirty
```

* version：当前迁移版本
* dirty：是否迁移中断

---

## 2️⃣ Go 中检测 migrate 状态（只读）

`internal/db/migrate_check.go`

```go
package db

import (
	"database/sql"
	"errors"
	"fmt"
)

func CheckMigrateStatus(db *sql.DB) error {
	var version sql.NullInt64
	var dirty bool

	err := db.QueryRow(`
		SELECT version, dirty
		FROM schema_migrations
		LIMIT 1
	`).Scan(&version, &dirty)

	if err != nil {
		return errors.New("database not migrated: schema_migrations table not found")
	}

	if dirty {
		return fmt.Errorf("database migration dirty at version %d", version.Int64)
	}

	return nil
}
```

---

## 3️⃣ 在 main.go 中使用

```go
err = db.CheckMigrateStatus(mysqlDB)
if err != nil {
	log.Fatalf("DB migrate check failed: %v", err)
}
```

### 企业级规则

> 🚨 **migrate 不通过 → 服务直接拒绝启动**

---

# 四、Step 2：多环境（dev / test / prod）配置拆分

---

## 1️⃣ 为什么必须拆环境？

| 环境   | 用途      |
| ---- | ------- |
| dev  | 本地开发    |
| test | CI / 测试 |
| prod | 线上      |

> ❌ 禁止：
> dev 代码直连 prod 数据库

---

## 2️⃣ 配置文件示例

### `config/dev.yaml`

```yaml
mysql:
  host: 127.0.0.1
  port: 3306
  user: root
  password: root
  database: go_demo_dev
```

### `config/prod.yaml`

```yaml
mysql:
  host: mysql-prod
  port: 3306
  user: go_app
  password: ${MYSQL_PASSWORD}
  database: go_demo
```

---

## 3️⃣ Go 加载配置（最简版）

```go
type Config struct {
	MySQL struct {
		Host     string
		Port     int
		User     string
		Password string
		Database string
	}
}
```

通过环境变量控制：

```bash
export APP_ENV=dev
```

```go
env := os.Getenv("APP_ENV")
cfgFile := fmt.Sprintf("config/%s.yaml", env)
```

---

# 五、Step 3：Docker + MySQL + migrate 一体化（重点）

---

## 1️⃣ docker-compose.yml

`docker/docker-compose.yml`

```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-dev
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: go_demo_dev
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

---

## 2️⃣ 安装 migrate 工具（本地）

```bash
brew install golang-migrate
```

---

## 3️⃣ migrations 目录规范（企业级）

```
migrations/
├── 000001_init.up.sql
├── 000001_init.down.sql
```

### `000001_init.up.sql`

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### `000001_init.down.sql`

```sql
DROP TABLE users;
```

---

## 4️⃣ 执行 migrate

```bash
migrate \
  -database "mysql://root:root@tcp(127.0.0.1:3306)/go_demo_dev" \
  -path migrations \
  up
```

---

# 六、Step 4：CI 中数据库变更流程（真实企业流程）

---

## 1️⃣ 提交数据库变更的**唯一方式**

> ❌ 禁止手写 SQL 到线上
> ❌ 禁止改表后不留记录

> ✅ **只能提交 migration 文件**

---

## 2️⃣ CI 执行流程（伪代码）

```yaml
steps:
  - docker-compose up -d mysql
  - migrate up
  - go test ./...
```

---

## 3️⃣ CI 失败场景

| 情况           | 结果      |
| ------------ | ------- |
| migrate 失败   | ❌ CI 阻断 |
| dirty = true | ❌ 禁止合并  |
| Go 测试失败      | ❌ 禁止发布  |

---

# 七、你现在“绝对不应该做”的事情

❌ Go 代码中判断表是否存在
❌ Go 中 `CREATE TABLE IF NOT EXISTS`
❌ 线上直接 `ALTER TABLE`
❌ 手工改库不提交 migration

---

# 八、你下一步应该做什么（学习路线）

### ✅ 第一步（你现在）

* 理解 migrate + 环境隔离

### ✅ 第二步

* 为 **每一个表结构变更** 写 migration

### ✅ 第三步

* 在 CI 中强制 migrate 校验

---

## 如果你愿意，下一步我可以直接帮你：

1️⃣ 给你 **一个完整可运行的 Go + Docker + migrate 示例仓库结构**
2️⃣ 给你 **Makefile 一键启动 / migrate / rollback**
3️⃣ 给你 **真实公司级 DB 变更 SOP 文档**

你只需要告诉我：
👉 **你现在是本地开发优先，还是准备直接做上线项目？**


***
<br/><br/><br/>

# 方案一：

很好，这一步开始你已经进入**真正的“企业级后端数据库工程”阶段**了。
下面我**不假设你有任何经验**，会用**“为什么 → 怎么做 → 示例 → 企业规范”**的方式，把你点名的 **1 / 2 / 3 / 4** 全部完整讲清楚。

> 你可以把这份内容当作
> **中大型互联网公司新人入职后 1～2 个月必须掌握的数据库基础内训文档**

---

# 总览：这 4 件事在企业里分别解决什么问题？

| 编号 | 主题                | 解决的核心问题          |
| -- | ----------------- | ---------------- |
| 1  | 数据库变更评审规范         | 防止「一个 SQL 把线上干崩」 |
| 2  | 线上数据迁移（不停服）       | 表结构升级时不影响用户      |
| 3  | 读写分离 / 主从延迟       | 扛高并发、避免读脏数据      |
| 4  | Go + MySQL 事务与一致性 | 并发下数据不乱、不丢       |

---

# 一、数据库变更评审规范（SQL Review Checklist）

## 1️⃣ 为什么需要 SQL Review？

真实事故（非常常见）：

* `ALTER TABLE users ADD COLUMN ...`
  → **锁表 30 分钟**
* `DELETE FROM orders;`
  → **误删全表**
* 少了 `WHERE`
  → **线上数据全毁**

> 所以在企业里：
> **任何 SQL 变更 ≈ 代码上线级别的严肃程度**

---

## 2️⃣ 企业标准流程（你以后一定会遇到）

```
开发提出需求
   ↓
写 migration SQL
   ↓
SQL Review（至少 1 人）
   ↓
测试环境执行
   ↓
预发 / 灰度
   ↓
生产执行
```

---

## 3️⃣ SQL Review Checklist（逐条给你）

### ✅ 1. 是否走 migration（而不是手工 SQL）

✔️ 正确：

```text
migrations/000003_add_user_status.up.sql
```

❌ 错误：

```sql
直接在生产库执行 ALTER TABLE
```

---

### ✅ 2. 是否有 down.sql（回滚方案）

**企业强制要求：**

```sql
-- up
ALTER TABLE users ADD COLUMN status TINYINT NOT NULL DEFAULT 1;

-- down
ALTER TABLE users DROP COLUMN status;
```

> 没有 down.sql = 不允许合并代码

---

### ✅ 3. 是否会锁表（重点）

**高风险 SQL：**

```sql
ALTER TABLE users MODIFY email VARCHAR(512);
```

* 行数大？
* 是否 InnoDB？
* 是否支持 online DDL？

👉 需要 DBA / 资深工程师确认

---

### ✅ 4. 是否全表扫描

```sql
UPDATE users SET status = 0;
```

❌ 直接禁止

必须：

```sql
UPDATE users SET status = 0 WHERE last_login < '2024-01-01';
```

---

## 4️⃣ 企业里 SQL Review 的“潜规则”

* **宁可慢，也不能炸**
* 改表 ≠ 普通功能上线
* 新人改表，必须有人兜底

---

# 二、线上数据迁移（不停服）

> 这是 **90% 小白第一次听就会懵的地方**

---

## 1️⃣ 什么叫“停服迁移”？

```text
停服务
改表
启动服务
```

❌ **中大型公司基本不允许**

---

## 2️⃣ 不停服迁移的核心思想（记住这 8 个字）

> **“向前兼容，分阶段上线”**

---

## 3️⃣ 最经典场景：给 users 表加字段

### 目标

新增字段：

```sql
user_level INT
```

---

### 阶段 1：只加字段（不使用）

```sql
ALTER TABLE users ADD COLUMN user_level INT DEFAULT 0;
```

Go 代码：

* ❌ 不读
* ❌ 不写

---

### 阶段 2：代码开始写新字段（兼容旧数据）

```go
INSERT INTO users (...) VALUES (..., userLevel)
```

读取时：

```go
if userLevel == 0 {
    // 当作普通用户
}
```

---

### 阶段 3：数据回填（后台慢慢跑）

```sql
UPDATE users SET user_level = 1 WHERE created_at < '2024-01-01' LIMIT 1000;
```

> 分批跑，避免锁表

---

### 阶段 4：强制约束（可选）

```sql
ALTER TABLE users MODIFY user_level INT NOT NULL;
```

---

## 4️⃣ 企业总结口诀

> **加字段先不读 → 再写 → 再补数据 → 最后加约束**

---

# 三、读写分离 / 主从延迟处理（入门版）

---

## 1️⃣ 为什么需要读写分离？

```
写请求 → 主库
读请求 → 从库
```

好处：

* 主库压力小
* 从库可水平扩展

---

## 2️⃣ 主从延迟是怎么来的？

```text
主库写入 → binlog → 从库同步（有延迟）
```

延迟可能：

* 10ms
* 100ms
* 1s+

---

## 3️⃣ 最经典的坑（新手 100% 会踩）

```text
注册成功
↓
立刻查询用户信息
↓
从库查
↓
查不到
```

---

## 4️⃣ 企业级解决方案（你先掌握前 2 个）

### ✅ 方案 1：关键读走主库（最常用）

```go
// 注册后，强制主库读取
user, _ := repo.GetUserByID(ctx, id, UseMaster())
```

---

### ✅ 方案 2：延迟读

```go
time.Sleep(100 * time.Millisecond)
```

（简单粗暴，但常见）

---

## 5️⃣ 架构层设计（给你概念）

```text
DBRouter
 ├── Master
 └── Slave
```

---

# 四、Go + MySQL 事务与并发一致性（非常重要）

---

## 1️⃣ 为什么要事务？

没有事务：

```text
扣钱成功
订单创建失败
```

→ 数据错乱

---

## 2️⃣ Go 中正确使用事务（标准模板）

```go
tx, err := db.BeginTx(ctx, nil)
if err != nil {
    return err
}

defer tx.Rollback()

_, err = tx.Exec(`UPDATE accounts SET balance = balance - ? WHERE id = ?`, amount, uid)
if err != nil {
    return err
}

_, err = tx.Exec(`INSERT INTO orders (...) VALUES (...)`)
if err != nil {
    return err
}

return tx.Commit()
```

---

## 3️⃣ 并发一致性的根本原因

```text
多个 goroutine 同时改同一行
```

---

## 4️⃣ 必须掌握的 2 个锁（入门）

### ✅ 行锁（最重要）

```sql
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
```

作用：

* 锁住这一行
* 其他事务必须等

---

### ✅ 乐观锁（进阶）

```sql
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 3;
```

---

## 5️⃣ 企业选型原则（记住）

| 场景 | 用什么  |
| -- | ---- |
| 金额 | 行锁   |
| 库存 | 乐观锁  |
| 配置 | 最终一致 |

---

# 五、你现在的学习路线（非常重要）

### 第 1 周

* 理解 migration + SQL review

### 第 2 周

* 会设计不停服迁移

### 第 3 周

* 理解主从延迟

### 第 4 周

* 事务 + 锁

---

## 下一步我可以继续带你做（直接实战）：

* ✅ 一个 **并发扣库存的 Go 示例（含事务 + 锁）**
* ✅ 一个 **不停服迁移完整演示（从 0 到上线）**
* ✅ 一个 **主从读写分离的代码结构示例**

你只要告诉我下一步选哪个，我就直接写代码给你。



***
<br/>

> <h3 id="godotenv.Load">godotenv.Load</h3>

```go
err := godotenv.Load(envFile)
```

`github.com/joho/godotenv` 的 `Load` 会读取 `.env` 文件，将其中的 `KEY=VALUE` 加载到**当前进程**的环境变量，之后可通过 `os.Getenv()` 获取。

```env
MYSQL_USER=root
MYSQL_PASSWORD=123456
MYSQL_HOST=127.0.0.1
MYSQL_PORT=3306
REDIS_ADDR=127.0.0.1:6379
```

```go
func main() {
    if err := godotenv.Load(".env"); err != nil {
        log.Printf("load .env: %v", err)
    }

    user := os.Getenv("MYSQL_USER")
    fmt.Println(user)
}
```

执行流程：

```text
.env 文件
    |
    v
解析 KEY=VALUE
    |
    v
写入当前进程环境
    |
    v
os.Getenv()
    |
    v
业务配置初始化
```

这些变量只影响当前进程及其后续创建的子进程，不会永久修改操作系统的全局环境。

### Load 与 Overload

| 方法 | 已存在同名环境变量时 |
| --- | --- |
| `godotenv.Load()` | 保留原环境变量，不覆盖 |
| `godotenv.Overload()` | 使用 `.env` 中的值强制覆盖 |

环境变量通常应拥有比本地 `.env` 更高的优先级，因此普通项目更常使用 `Load()`。

### 多环境文件

```go
envFile := ".env.dev"
if os.Getenv("APP_ENV") == "prod" {
    envFile = ".env.prod"
}

if err := godotenv.Load(envFile); err != nil {
    return fmt.Errorf("load %s: %w", envFile, err)
}
```

注意相对路径基于程序启动时的当前工作目录，而不是 `main.go` 所在目录：

```text
project/
├── configs/.env
└── cmd/server/main.go
```

从 `project/` 启动时，应加载 `configs/.env`。

### 生产安全

开发环境可使用 `.env`，但不应将真实密码提交到 Git。生产环境通常使用 Kubernetes Secret、AWS Secrets Manager、HashiCorp Vault 或云厂商 KMS，并通过容器环境变量注入应用。

`godotenv` 负责加载简单的 `KEY=VALUE`；Viper 更适合 YAML/JSON/TOML、多来源合并和类型化配置。







