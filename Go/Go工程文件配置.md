
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





