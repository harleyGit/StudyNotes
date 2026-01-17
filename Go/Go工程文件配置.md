
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
	- [生产环境“注入”环境变量](#生产环境“注入”环境变量)
		- [方案一：Docker注入](#方案一：Docker注入)	
		- [方案二：Kubernetes（K8s）](#方案二：Kubernetes（K8s）) 
		- [方案三：CI/CD](#方案三：CI/CD)


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
	- 如果环境变量存在，返回其值（字符串）。
	- 如果不存在，返回空字符串 `""`。


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


