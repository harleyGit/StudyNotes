> # 
- [Makefile基础简述](#Makefile基础简述)
- [工程Makefile详解](#工程Makefile详解)
- [.PHONY的换行注意](#.PHONY的换行注意)




<br/><br/><br/>

***
<br/>

> <h1 id="Makefile基础简述">Makefile基础简述</h1>
- **Makefile** 是一个用于自动化构建项目的脚本文件，通常用在 C/C++、Go 等编译型语言中。
- 它由 **目标（target）**、**依赖（prerequisites）** 和 **命令（recipe）** 组成。
- 每个命令前必须用 **Tab 键缩进**（不是空格！这是常见错误）。
- `.PHONY` 是一个特殊目标，用来声明某些目标不是真实文件名，避免和同名文件冲突。

<br/><br/><br/>

***
<br/>

> <h1 id="工程Makefile详解">工程Makefile详解</h1>

**工程文件结构：**

```sh
|
|————scripts
|   ｜————db.sh
|
|————Makefile
|
```

<br/>

```makefile
.PHONY: build clean tool lint help test db-init db-run db-shell db-reset

all: build

build:
	go build -v .

tool:
	go tool vet . |& grep -v vendor; true
	gofmt -w .

lint:
	golint ./...

clean:
	rm -rf go-gin-example
	go clean -i .

test:
	go test ./...

db-init:
	@./scripts/db.sh

db-run:
	@if [ -z "$(SQL)" ]; then \
		echo "[Info]: 用法： make db-run SQL=path/to/file.sql"; \
		exit 1; \
	fi
	@./scripts/db.sh run

db-shell:
	@./scripts/db.sh shell

db-reset:
	@mysql -uroot -phh109 -e "DROP DaTABASE IF EXISTS HG_MLC_DB;"
	@./scripts/db.sh init

help:
	@echo "make: compile packages and dependencies"
	@echo "make tool: run specified go tool"
	@echo "make lint: golint ./..."
	@echo "make clean: remove object files and cached files"
	@echo "make test: run go tests"
```

***
<br/> 

**逐行讲解Makefile文件：**

```makefile
.PHONY: build clean tool lint help test db-init db-run db-shell db-reset
```
> **说明**：  
> 声明 `build`, `clean`, ..., `db-reset` 这些目标都是“伪目标”（phony targets），即它们不代表实际的文件。  
> 例如，即使当前目录下有个叫 `clean` 的文件，执行 `make clean` 时仍然会运行 clean 的命令，而不是认为目标已是最新的。

<br/> 

```makefile
all: build
```
> **说明**：  
> 默认目标是 `all`（当你只输入 `make` 时，会执行这个）。  
> 它依赖于 `build`，所以会先执行 `build` 目标。

<br/> 

```makefile
build:
	go build -v .
```
> **说明**：  
> 定义 `build` 目标。  
> 执行命令：用 Go 编译当前目录下的代码（`-v` 表示显示详细过程）。

---
<br/> 

```makefile
tool:
	go tool vet . |& grep -v vendor; true
	gofmt -w .
```
> **说明**：  
> `tool` 目标做两件事：
> 1. 用 `go vet` 检查代码（`|&` 是 bash 中同时重定向 stdout 和 stderr 的写法），过滤掉 `vendor` 目录的输出；后面的 `; true` 是为了确保即使 `grep` 没匹配到内容（返回非0），整个命令也不会失败（因为 Make 遇到非0退出码会中断）。
> 2. 用 `gofmt -w .` 自动格式化所有 Go 文件并保存。

---
<br/> 

```makefile
lint:
	golint ./...
```
> **说明**：  
> 运行 `golint` 工具检查整个项目（`./...` 表示递归所有子包）的代码风格。

> ⚠️ 注意：`golint` 已被官方弃用，建议改用 `golangci-lint` 等现代工具。

---
<br/> 

```makefile
clean:
	rm -rf go-gin-example
	go clean -i .
```
> **说明**：  
> 清理构建产物：
> - 删除名为 `go-gin-example` 的可执行文件（或目录）。
> - `go clean -i .` 会删除通过 `go install` 安装到 `$GOBIN` 或 `$GOPATH/bin` 的可执行文件。

---
<br/> 

```makefile
test:
	go test ./...
```
> **说明**：  
> 运行项目中所有包的测试。

---

```makefile
db-init:
	@./scripts/db.sh
```
> **说明**：  
> 执行数据库初始化脚本。  
> `@` 表示不打印该命令本身（只显示输出结果）。

---
<br/> 

```makefile
db-run:
	@if [ -z "$(SQL)" ]; then \
		echo "[Info]: 用法： make db-run SQL=path/to/file.sql"; \
		exit 1; \
	fi
	@./scripts/db.sh run
```
> **说明**：  
> 这是一个带参数检查的 shell 脚本调用：
> - `$(SQL)` 是 Make 变量，用户需通过 `make db-run SQL=xxx.sql` 传入。
> - 如果 `SQL` 为空（`-z` 判断字符串长度为0），就打印用法并退出（`exit 1` 表示错误）。
> - 注意：这里用了 `\` 来**续行**（把多行 shell 命令拼成一行）。
> - 最后调用 `./scripts/db.sh run` 执行 SQL。

<br/> 

```makefile
db-shell:
	@./scripts/db.sh shell
```
> **说明**：  
> 进入数据库交互式 shell（比如 MySQL 客户端）。

---
<br/> 

```makefile
db-reset:
	@mysql -uroot -phh109 -e "DROP DATABASE IF EXISTS HG_MLC_DB;"
	@./scripts/db.sh init
```
> **说明**：  
> 重置数据库：
> 1. 用 `mysql` 命令行客户端删除名为 `HG_MLC_DB` 的数据库（注意大小写敏感？MySQL 在 Linux 下默认区分大小写）。
> 2. 重新初始化数据库（调用脚本）。

> ⚠️ 安全提示：密码 `hh109` 明文写在 Makefile 中有风险，建议改用配置文件或环境变量。

---
<br/> 

```makefile
help:
	@echo "make: compile packages and dependencies"
	@echo "make tool: run specified go tool"
	@echo "make lint: golint ./..."
	@echo "make clean: remove object files and cached files"
	@echo "make test: run go tests"
```
> **说明**：  
> 打印帮助信息。用户运行 `make help` 就能看到常用命令说明。

<br/><br/><br/>

***
<br/>

> <h1 id=".PHONY的换行注意">.PHONY的换行注意</h1>


**❓Q1：`.PHONY` 行太长了，在行尾加 **反斜杠 `\`**，如下：**

```makefile
.PHONY: build clean tool lint help test \
        db-init db-run db-shell db-reset
```

> ✅ 这是标准做法，Make 会把 `\` 后的下一行视为当前行的延续。

<br/>

**Q2：`\` 后的下一行可以跟注释吗？会影响吗？**

❌ **不可以！**

在 Makefile 中，**续行符 `\` 后面不能有注释**，否则会导致语法错误或行为异常。

**错误示例（不要这样写）：**

```makefile
.PHONY: build clean \
        # 这是注释！ ← ❌ 错误！
        tool lint
```

> 上面的 `# 这是注释` 会被当作目标名的一部分，导致 `.PHONY` 包含了一个叫 `# 这是注释！` 的无效目标，可能报错或静默失败。

<br/>

- **正确做法：**
	- 注释只能写在独立行，或者写在 `\` **之前**的行末（但不推荐，易混淆）。
	- 推荐：把注释放在 `.PHONY` 上方或下方单独一行。

✅ 正确示例：

```makefile
# 数据库相关伪目标
.PHONY: db-init db-run db-shell db-reset
```

或者换行时不要加注释：
```makefile
.PHONY: build clean tool lint help test \
        db-init db-run db-shell db-reset
```
