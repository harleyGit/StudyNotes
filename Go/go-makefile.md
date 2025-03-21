> <h3/>
- [**Makefile构建项目**](#Makefile构建项目)
- [语法规范](#语法规范)
- [Go项目的命令](#Go项目的命令)
	- [Makefile文件执行](#Makefile文件执行)
- [makefile文件](#makefile文件)
	- [变量定义部分](#变量定义部分)
	- [链接参数（LDFLAGS）](#链接参数（LDFLAGS）) 
	- [构建目标](#构建目标) 
	- [运行](#运行)



<br/><br/><br/>

***
<br/>

> <h1 id="Makefile构建项目">Makefile构建项目</h1>
**引入:** 含一定复杂度的软件工程，基本上都是先编译 A，再依赖 B，再编译 C…，最后才执行构建。如果每次都人为编排，又或是每新来一个同事就问你项目 D 怎么构建、重新构建需要注意什么…等等情况，岂不是要崩溃？

我们常常会在开源项目中发现 Makefile，你是否有过疑问？

**怎么解决**

对于构建编排，Docker 有 Dockerfile ，在 Unix 中有神器 Make ….

**Make是什么?**

Make 是一个构建自动化工具，会在当前目录下寻找 Makefile 或 makefile 文件。如果存在，会依据 Makefile 的构建规则去完成构建

当然了，实际上 Makefile 内都是你根据 make 语法规则，自己编写的特定 Shell 命令等

它是一个工具，规则也很简单。在支持的范围内，编译 A， 依赖 B，再编译 C，完全没问题

<br/>

Make是常用的构建工具，Makefile是一个文本文件，遵循一套语法规范，可用来对复杂项目的构建、编译等流程定义一系列规则和指定执行的命令，类似于Shell脚本。通过Makefile文件的定义，最后执行make command便可执行相应的命令。

<br/><br/><br/>
> <h2 id="语法规范">语法规范</h2>

Makefile 由多条规则组成，每条规则都以一个 target（目标）开头，后跟一个 : 冒号，冒号后是这一个目标的 prerequisites（前置条件）

紧接着新的一行，必须以一个 tab 作为开头，后面跟随 command（命令），也就是你希望这一个 target 所执行的构建命令

<br/>

Make的语法规范如下：

```makefile
<target> : <prerequisites>
[tab]  <commands>
```

-  target：一个目标代表一条规则，可以是一个或多个文件名。也可以是某个操作的名字（标签），称为伪目标（phony）。

-  prerequisites：前置条件，这一项是可选参数。通常是多个文件名、伪目标。它的作用是 target 是否需要重新构建的标准，如果前置条件不存在或有过更新（文件的最后一次修改时间）则认为 target 需要重新构建

-  [tab]​：每个命令之前必须有一个Tab键，当然可以自定义其他的符号。

-  commands：构建这一个 target 的具体命令集(具体的执行命令)。

Makefile文件就是由这样一套语法规则构成的，使用的命令是：`make <target>`。还支持一些其他的语法规范，整体使用起来和Shell脚本非常像。


(1)注释：

```makefile
# 注释
```

(2)变量

```makefile
BINARY=go
```

(3)命令

```makefile
<target>: <prerequisites>
[tab]  @<commands>
```

<br/><br/>

**示例**

```makefile
PROJECT=go
default:
     go env
version:
     go version
noecho:
     @go version
.PHONY: default version noecho
```

<br/>

在Makefile所在目录下执行命令：make不带参数时，默认执行第一个命令。

```sh
make
>>
go env
GOARCH="amd64"
GOBIN="/Users/xiewei/go/bin"
GOCACHE="/Users/xiewei/Library/Caches/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
...
```

<br/>

make version实质是执行go version。
 
```sh
// 命令
make version
// 显示结果
go version  // 回显具体执行的命令语句
go version go1.12.7 darwin/amd64 // 执行命令的结果
```

<br/>

make noecho实质是执行go version，不显示具体的命令。

```sh
// 命令
make noecho
// 显示结果
go version go1.12.7 darwin/amd64
```

- .PHONY表示伪目标，内置的字段一般前置命令是所有的命令。

<br/><br/><br/>
> <h2 id="Go项目的命令">Go项目的命令</h2>

在GitHub上的许多开源项目中都有Makefile文件的身影，用于项目编译、构建，配合自动化流水线，可以完成一整套的持续集成、持续部署操作。针对Go项目，开发者应该提供哪些命令来操作项目呢？在终端下输入Go命令，可以查看Go支持的命令。

- 比较重要且常用的是：
	- 测试。
	- 编译。
	- 静态检查。
	- 运行。
	- 安装下载库。
	- 格式化代码。

<br/>

所以，一个适用于Go项目的Makefile文件也应该支持这些操作，具体可根据项目的实际需求而定，当然也可以自定义命令来完成相应的操作。

- make default：编译。
- make install：下载安装。
- make vet：静态检查。
- make fmt：格式化代码。
- make clean：移除编译的文件。

<br/>

比如下面这个示例：

```makefile
BINARY="votes"
VERSION=1.0.0
BUILD='date +%FT%T%z'
PACKAGES=`go list ./... | grep -v /vendor/`
VETPACKAGES=`go list ./... | grep -v /vendor/ | grep -v /examples/`
GOFILES=`find . -name "*.go" -type f -not -path "./vendor/*"`
default:
 @go build -o ${BINARY} -tags=jsoniter
list:
 @echo ${PACKAGES}
 @echo ${VETPACKAGES}
 @echo ${GOFILES}
fmt:
 @gofmt -s -w ${GOFILES}
fmt-check:
 @diff=$$(gofmt -s -d $(GOFILES)); \
 if [ -n "$$diff" ]; then \
     echo "Please run 'make fmt' and commit the result:"; \
     echo "$${diff}"; \
     exit 1; \
 fi;
install:
 @govendor sync -v
test:
 @go test -cpu=1,2,4 -v -tags integration ./...
vet:
 @go vet $(VETPACKAGES)
docker:
 @docker build . -t wuxiaoxiaoshen/votes:latest -f chapter11/deployments/Dockerfile
clean:
 @if [[ -f ${BINARY} ]] ; then rm ${BINARY} ; fi
.PHONY: default fmt fmt-check install test vet docker clean
```

Makefile文件不但方便开发人员在本地编译、构建项目，而且可以配合自动化流水线，从而完成更多的前置任务。


<br/><br/>
> <h3 id="Makefile文件执行">Makefile文件执行</h3>

 **在 Go 项目中使用 `Makefile` 的具体步骤**  
使用 `Makefile` 可以 **自动化管理** Go 项目的构建、代码检查、格式化和清理等任务。  
以下是具体的使用方法：

---

**1.在 Go 项目中创建 `Makefile`**

在你的 Go 项目根目录下创建一个 `Makefile` 文件：

```sh
touch Makefile
```

然后，将以下 `Makefile` 代码写入文件：

```makefile
.PHONY: build clean tool lint help test

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

help:
	@echo "make: compile packages and dependencies"
	@echo "make tool: run specified go tool"
	@echo "make lint: golint ./..."
	@echo "make clean: remove object files and cached files"
	@echo "make test: run go tests"
```

<br/><br/>
> <h3 id="命令解读">命令解读</h3>

**1.1`.PHONY: build clean tool lint help`**
**作用**：声明 **伪目标（phony targets）**，避免与实际文件名冲突。  
在 `Makefile` 中，如果有同名文件，`make` 默认会认为它是目标文件，而不会执行目标规则。  
`PHONY` 让 `make` 知道这些不是文件，而是 **执行命令的目标**。

---

**1.2.`all: build`**
- `all` 目标：当你运行 `make` 时，默认执行 `build` 目标。
- `build` 依赖于 `build` 目标，因此它会执行 `build` 规则。

📌 **等效于：**
```sh
make   # 等于 make build
```

---

**1.3.`build:`**

```makefile
build:
	go build -v .
```

**作用**：
- `go build -v .` 编译当前目录下的 Go 项目。
- `-v` 参数：显示被编译的包名，便于调试。

📌 **运行命令**

```sh
make build  # 编译项目
```

---

**1.4. `tool:`**

```makefile
tool:
	go tool vet . |& grep -v vendor; true
	gofmt -w .
```
**作用**：
- `go tool vet .` 静态分析 Go 代码，查找潜在错误（如 `printf` 格式错误）。
- `|& grep -v vendor; true` 过滤掉 `vendor/` 目录的输出，并且即使 `vet` 失败也不会影响 `make`。
- `gofmt -w .` 格式化当前目录下的所有 Go 源代码文件。

📌 **运行命令**

```sh
make tool  # 执行代码检查和格式化
```

---

**1.5. `lint:`**

```makefile
lint:
	golint ./...
```
**作用**：
- `golint ./...` 运行 `golint` 静态分析，检查代码风格问题。

📌 **运行命令**

```sh
make lint  # 运行 golint
```

💡 **如果 `golint` 未安装，可以这样安装：**

```sh
go install golang.org/x/lint/golint@latest
```

---

**1.6.`clean:`**

```makefile
clean:
	rm -rf go-gin-example
	go clean -i .
```
**作用**：
- `rm -rf go-gin-example` 删除可执行文件 `go-gin-example`（如果有的话）。
- `go clean -i .` 清理编译缓存。

📌 **运行命令**

```sh
make clean  # 清理项目
```

---

**1.7. `help:`**

```makefile
help:
	@echo "make: compile packages and dependencies"
	@echo "make tool: run specified go tool"
	@echo "make lint: golint ./..."
	@echo "make clean: remove object files and cached files"
```
**作用**：
- `@echo` 直接输出帮助信息，`@` 符号让 `make` **不打印命令本身**。

📌 **运行命令**

```sh
make help  # 显示帮助信息
```
🔹 **示例输出**

```
make: compile packages and dependencies
make tool: run specified go tool
make lint: golint ./...
make clean: remove object files and cached files
```

---

## **总结**
| 目标 | 作用 | 运行命令 |
|------|------|---------|
| `all` | 默认执行 `build` | `make` |
| `build` | 编译 Go 项目 | `make build` |
| `tool` | 代码静态分析 (`vet`) 和格式化 (`gofmt`) | `make tool` |
| `lint` | 代码风格检查 (`golint`) | `make lint` |
| `clean` | 清理编译产物 | `make clean` |
| `help` | 显示可用命令 | `make help` |

---

### **常见问题**
#### **1.8. `make` 运行报错：`command not found`**
🔹 **原因**：你的系统可能没有安装 `make`。  
🔹 **解决方案**：
- **Mac**:  
  
  ```sh
  brew install make
  ```
- **Ubuntu/Debian**:  
  
  ```sh
  sudo apt install make
  ```
- **Windows**:  
  需要安装 [Make for Windows](http://gnuwin32.sourceforge.net/packages/make.htm) 或者用 Git Bash。

---

### **结论**
这个 `Makefile` 主要是为了：
✅ **自动化编译 (`build`)**  
✅ **代码检查 (`vet`、`golint`)**  
✅ **代码格式化 (`gofmt`)**  
✅ **清理无用文件 (`clean`)**  

💡 **你可以扩展它，比如加上 `test` 目标运行 `go test ./...`**：

```makefile
test:
	go test ./...
```
📌 运行：

```sh
make test
```

<br/><br/>
---
**2.安装必要的工具**

确保你安装了 `golint`，否则 `make lint` 会报错：

```sh
go install golang.org/x/lint/golint@latest
```

---

**3.在 Go 项目中使用 `Makefile`**

在项目根目录中运行 `make` 命令，执行不同的任务。

**(1)编译项目**

```sh
make build
```
🔹 **作用**：编译当前 Go 项目，生成可执行文件。  
🔹 **等价于**：`go build -v .`

---

**(2) 代码静态检查 & 格式化**

```sh
make tool
```
🔹 **作用**：
- 运行 `go vet` 检查代码中可能的错误
- 运行 `gofmt` 格式化代码

---

**(3) 代码风格检查**

```sh
make lint
```
🔹 **作用**：运行 `golint ./...` 检查代码风格问题。

---

**(4) 运行测试**

```sh
make test
```
🔹 **作用**：执行 `go test ./...` 运行项目的所有单元测试。

---

**(5) 清理项目**

```sh
make clean
```
🔹 **作用**：
- 删除 `go-gin-example` 可执行文件
- 运行 `go clean -i .` 清理编译缓存

---

**(6) 查看帮助**

```sh
make help
```
🔹 **作用**：列出所有可用的 `make` 任务。

🔹 **示例输出**
```
make: compile packages and dependencies
make tool: run specified go tool
make lint: golint ./...
make clean: remove object files and cached files
make test: run go tests
```

---

**4. 在 `Makefile` 中添加自定义任务**
你可以自定义 `Makefile`，比如添加 **API 文档生成** 或 **Docker 构建**：

**(1) 生成 API 文档**

```makefile
docs:
	swagger generate spec -o ./swagger.json --scan-models
```
运行：

```sh
make docs
```

---

**(2) 使用 Docker 构建 Go 项目**

```makefile
docker:
	docker build -t my-go-app .
```
运行：

```sh
make docker
```

---

**总结**
📌 **完整的 `Makefile` 用法**

```sh
make build    # 编译项目
make tool     # 代码静态检查 & 格式化
make lint     # 代码风格检查
make test     # 运行测试
make clean    # 清理项目
make help     # 显示帮助信息
```





<br/><br/><br/>
> <h2 id="">makefile文件</h2>
**Makefile文件**

```makefile
BINARY=BeeQuick

VERSION=1.0.0

BUILD=`date +%FT%T%z`

LDFLAGS=-ldflags "-X main.Env=production -s -w"

DEV_LDFLAGS=-ldflags "-X main.Env=dev"

TEST_LDFLAGS=-ldflags "-X main.Env=test"

default:
	go build -o ${BINARY} -v ${DEV_LDFLAGS} -tags=jsoniter

production:
	go build -o ${BINARY} -v ${LDFLAGS} -tags=jsoniter

dev:
	go build -o ${BINARY} -v ${DEV_LDFLAGS} -tags=jsoniter

test:
	go build -o ${BINARY} -v ${TEST_LDFLAGS} -tags=jsoniter

run:
	go run -v ${DEV_LDFLAGS} -tags=jsoniter main.go

.PHONY: default production dev test run
```


这段 `Makefile` 用于编译和运行 Go 代码，并根据不同的环境（开发、测试、生产）使用不同的编译参数。下面是对代码的详细解析：

<br/><br/>
> <h2 id="变量定义部分">变量定义部分</h2>

指定编译后生成的二进制文件名称为 `BeeQuick`。

```makefile
BINARY=BeeQuick
```

<br/>

设定应用程序的版本号为 `1.0.0`。

```makefile
VERSION=1.0.0
```

<br/>

- 获取当前时间作为 `BUILD` 变量的值，格式为 `YYYY-MM-DDTHH:MM:SS+时区`（符合 ISO 8601 格式）。
- 但这里 `BUILD` 变量没有被使用。

```makefile
BUILD=`date +%FT%T%z`
```


<br/><br/>
> <h2 id="链接参数（LDFLAGS）">链接参数（LDFLAGS）</h2>

- `-ldflags` 传递链接参数：
  - `-X main.Env=production`：将 `main.Env` 变量的值设置为 `"production"`。
  - `-s -w`：
    - `-s`：去掉符号表，减少二进制文件体积。
    - `-w`：去掉调试信息，进一步压缩体积。

```makefile
LDFLAGS=-ldflags "-X main.Env=production -s -w"
```

<br/>

- `-X main.Env=dev`：在开发环境下，`main.Env` 变量的值为 `"dev"`。

```makefile
DEV_LDFLAGS=-ldflags "-X main.Env=dev"
```

<br/>

- `-X main.Env=test`：在测试环境下，`main.Env` 变量的值为 `"test"`。

```makefile
TEST_LDFLAGS=-ldflags "-X main.Env=test"
```

<br/><br/>
> <h2 id="构建目标">构建目标</h2>

1.**默认构建 (`default`)**

```makefile
default:
	go build -o ${BINARY} -v ${DEV_LDFLAGS} -tags=jsoniter
```

- `go build`：编译 Go 代码。
- `-o ${BINARY}`：输出二进制文件到 `BeeQuick`。
- `-v`：显示编译过程中导入的包。
- `${DEV_LDFLAGS}`：使用 `dev` 环境的 `ldflags`，即 `-X main.Env=dev`。
- `-tags=jsoniter`：使用 `jsoniter` 作为 JSON 解析库（需要在代码中 `// +build jsoniter` 或 `//go:build jsoniter`）。

<br/>

- 2.**生产环境构建 (`production`)**

```makefile
production:
	go build -o ${BINARY} -v ${LDFLAGS} -tags=jsoniter
```
- 和 `default` 类似，但使用 `LDFLAGS`（即 `main.Env=production`）。

<br/>

- 3.**开发环境构建 (`dev`)**

```makefile
dev:
	go build -o ${BINARY} -v ${DEV_LDFLAGS} -tags=jsoniter
```

- 与 `default` 目标一致，也是开发环境构建。

<br/>

- 4.**测试环境构建 (`test`)**

```makefile
test:
	go build -o ${BINARY} -v ${TEST_LDFLAGS} -tags=jsoniter
```
- 使用 `TEST_LDFLAGS`，即 `main.Env=test`。

<br/><br/>
> <h2 id="">运行</h2>

- **运行目标**

```makefile
run:
	go run -v ${DEV_LDFLAGS} -tags=jsoniter main.go
```
- `go run`：编译并运行 `main.go`。
- `-v`：显示编译过程。
- `${DEV_LDFLAGS}`：使用开发环境（`main.Env=dev`）。
- `-tags=jsoniter`：使用 `jsoniter` 作为 JSON 解析库。

<br/>

- **伪目标 (`.PHONY`)**

```makefile
.PHONY: default production dev test run
```

- `.PHONY` 声明这些目标是伪目标，即使目录中存在同名文件，也不会把它们当作文件来执行，而是始终执行 Makefile 中的规则。

<br/><br/>

**总结**
1. **支持不同环境的编译**：
	- `production`：生产环境（优化二进制大小）。
	- `dev`：开发环境。
	- `test`：测试环境。

2. **支持 JSON 库切换**
	- 通过 `-tags=jsoniter` 让代码可以选择 `jsoniter` 作为 JSON 解析库。

3. **提供 `run` 直接运行 Go 代码**
	- `run` 目标适用于开发阶段，避免每次手动执行 `go build` + `./BeeQuick`。
