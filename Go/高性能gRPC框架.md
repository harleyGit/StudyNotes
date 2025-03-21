> <h3/>
- [RPC](#RPC)
- [gRPC框架](#gRPC框架)
	- [gRPC在Golang内部API通信示例](#gRPC在Golang内部API通信示例)
	- [grpc环境配置](#grpc环境配置)
	- [protoc使用](#protoc使用)
	- [简单Demo流程](#简单Demo流程)
	- [proto编译成go代码](#proto编译成go代码)	
	- [命令行模块cmd](#命令行模块cmd)
- [流式gRPC](#流式gRPC)
- [基于 CA 的 TLS 证书认证](#基于CA的TLS证书认证)
	- [根证书](#根证书) 
	- [Server](#Server) 
	- [Client](#Client)
- [拦截器](#拦截器)
- [gRPC提供HTTP接口](#gRPC提供HTTP接口)
- [RPC自定义认证](#RPC自定义认证)
	- [截止时间Deadlines](#截止时间Deadlines)




<br/><br/><br/>

***
<br/>
> <h1 id="RPC">RPC</h1>
**什么是 RPC?**
RPC 代指远程过程调用（Remote Procedure Call），它的调用包含了传输协议和编码（对象序列号）协议等等。允许运行于一台计算机的程序调用另一台计算机的子程序，而开发人员无需额外地为这个交互作用编程

<br/>

**实际场景：**

有两台服务器，分别是 A、B。在 A 上的应用 C 想要调用 B 服务器上的应用 D，它们可以直接本地调用吗？
答案是不能的，但走 RPC 的话，十分方便。因此常有人称使用 RPC，就跟本地调用一个函数一样简单

<br/>

**RPC 框架**

我认为，一个完整的 RPC 框架，应包含负载均衡、服务注册和发现、服务治理等功能，并具有可拓展性便于流量监控系统等接入
那么它才算完整的，当然了。有些较单一的 RPC 框架，通过组合多组件也能达到这个标准

<br/>

**常见RPC框架**
- gPRC
- Thrift
- Rpcx
- Dubbo

**比较下:**

| \ | 跨语言 | 多IDL | 服务治理 | 注册中心 | 服务管理 |
|:--|:--|:--|:--|:--|:--|
| gRPC | 💯 | ❌ | ❌ | ❌ | ❌ |
| Thrift | 💯 | ❌ | ❌ | ❌ | ❌ |
| Rpcx | ❌ | 💯 | 💯 | 💯 | 💯 |
| Dubbo | ❌ | 💯 | 💯 | 💯 | 💯 |

<br/>

**为什么要 RPC**
简单、通用、安全、效率

**RPC 可以基于 HTTP 吗**
- RPC 是代指远程过程调用，是可以基于 HTTP 协议的

- 肯定会有人说效率优势，我可以告诉你，那是基于 HTTP/1.1 来讲的，HTTP/2 优化了许多问题（当然也存在新的问题），所以你看到了本文的主题 gRPC

<br/>

**相较 Protobuf，为什么不使用 XML？**
- 更简单
- 数据描述文件只需原来的 1/10 至 1/3
- 解析速度是原来的 20 倍至 100 倍
- 减少了二义性
- 生成了更易使用的数据访问类


<br/><br/><br/>

***
<br/>
># <h1 id="gRPC框架">[gRPC框架](https://grpc.io/docs/)</h1>
[Protocol Buffers](https://protobuf.dev/programming-guides/proto3/)
**介绍**
&emsp; gRPC 是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计

<br/>

**多语言**
- C++
- C#
- Dart
- Go
- Java
- Node.js
- Objective-C
- PHP
- Python
- Ruby

<br/>

**特点**
1、HTTP/2
2、Protobuf
3、客户端、服务端基于同一份 IDL
4、移动网络的良好支持
5、支持多语言

<br/>

**‌ 讲解**
- 1、客户端（gRPC Sub）调用 A 方法，发起 RPC 调用
- 2、对请求信息使用 Protobuf 进行对象序列化压缩（IDL）
- 3、服务端（gRPC Server）接收到请求后，解码请求体，进行业务逻辑处理并返回
- 4、对响应结果使用 Protobuf 进行对象序列化压缩（IDL）
- 5、客户端接受到服务端响应，解码请求体。回调被调用的 A 方法，唤醒正在等待响应（阻塞）的客户端调用并返回响应结果



<br/><br/><br/>

***
<br/>

> <h1 id="gRPC在Golang内部API通信示例">gRPC在Golang内部API通信示例</h1>
## **1. gRPC 在 Golang 内部 API 通信示例**
### **定义 gRPC 服务**
创建 `.proto` 文件：

```proto
syntax = "proto3";

package userpb;

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  int32 id = 1;
}

message UserResponse {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

### **生成 Golang 代码**

```sh
protoc --go_out=. --go-grpc_out=. user.proto
```

### **实现 gRPC 服务器**

```go
package main

import (
	"context"
	"log"
	"net"

	pb "example.com/userpb"
	"google.golang.org/grpc"
)

type server struct {
	pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.UserResponse, error) {
	return &pb.UserResponse{Id: req.Id, Name: "Alice", Email: "alice@example.com"}, nil
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("Failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterUserServiceServer(s, &server{})

	log.Println("gRPC server listening on port 50051")
	s.Serve(lis)
}
```

### **gRPC 客户端**

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	pb "example.com/userpb"
	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("Failed to connect: %v", err)
	}
	defer conn.Close()

	client := pb.NewUserServiceClient(conn)

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	res, err := client.GetUser(ctx, &pb.UserRequest{Id: 1})
	if err != nil {
		log.Fatalf("Could not get user: %v", err)
	}
	fmt.Printf("User: %v\n", res)
}
```

---

## **1. 总结**
| **选择** | **适用场景** |
|---------|------------|
| **RESTful API（HTTP+JSON）** | 前后端交互，兼容性强，浏览器友好 |
| **传统 RPC（JSON-RPC、Thrift）** | 轻量级 RPC，适合小规模内部服务 |
| **gRPC** | 高性能微服务，低延迟、流式通信，云原生架构 |

如果你的系统是**高吞吐量、跨语言、多微服务架构**，那么 **gRPC 是更好的选择**。如果只是简单的**内部 API 通信**，可以用 **JSON-RPC/Thrift** 代替。

<br/><br/><br/>
> <h2 id="grpc环境配置">grpc环境配置</h2>
**grpc安装**

```sh
go get -u google.golang.org/grpc
```

检查是否安装成功

```sh
protoc --version
libprotoc 29.3
```

<br/>

**Protoc Plugin(protobuf 插件)安装**

为了在 Golang 中使用 protobuf，你需要安装 Go 的 protobuf 插件。运行以下命令来安装

```sh
//或者 go get -u github.com/golang/protobuf/protoc-gen-go
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

<br/>

**编译和安装 Protocol Buffers (protobuf)** 
protobuf 的安装过程。运行以下命令来安装 protobuf：

```sh
brew install protobuf
```

安装完成后，你可以通过以下命令验证 protobuf 是否安装成功：

```bash
protoc --version
```
如果安装成功，这将输出 protobuf 的版本号。
<br/>

**更新你的 PATH**

确保你的 $GOPATH/bin 或 $HOME/go/bin 在你的 PATH 中，这样你就可以直接使用 protoc-gen-go。你可以通过以下命令将其添加到你的 .bashrc 或 .zshrc 文件中：

```
bash
export PATH=$PATH:$(go env GOPATH)/bin
```
然后运行以下命令使更改生效：

```bash
source ~/.bashrc  # 或者 source ~/.zshrc
```

<br/>

**Grpc-gateway**
grpc-gateway是protoc的一个插件。它读取gRPC服务定义，并生成一个反向代理服务器，将RESTful JSON API转换为gRPC。此服务器是根据gRPC定义中的自定义选项生成的。

**安装：**

```sh
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
```


<br/><br/><br/>
> <h2 id="protoc使用">protoc使用</h2>
- 我们按照惯例执行protoc --help（查看帮助文档），我们抽出几个常用的命令进行讲解：
	- 1、-IPATH, --proto_path=PATH：指定import搜索的目录，可指定多个，如果不指定则默认当前工作目录
	- 2、--go_out：生成golang源文件

<br/>

**参数**
若要将额外的参数传递给插件，可使用从输出目录中分离出来的逗号分隔的参数列表:

使用 Protocol Buffers 编译器（`protoc`）生成 Go 代码的命令，且支持 gRPC 服务。如下面的命令：

```sh
protoc --go_out=plugins=grpc,import_path=mypackage:. *.proto
```
- import_prefix=xxx：将指定前缀添加到所有import路径的开头
- import_path=foo/bar：如果文件没有声明go_package，则用作包。如果它包含斜杠，那么最右边的斜杠将被忽略。
- plugins=plugin1+plugin2：指定要加载的子插件列表（我们所下载的repo中唯一的插件是grpc）
- Mfoo/bar.proto=quux/shme： M参数，指定.proto文件编译后的包名（foo/bar.proto编译后为包名为quux/shme）

<br/>

***
### **各部分详解：**

1. **`protoc`**
   - 这是 Protocol Buffers 编译器的命令行工具，用来将 `.proto` 文件编译为指定语言（如 Go、Java、Python）的代码。

2. **`--go_out`**
   - `--go_out` 是指示 `protoc` 编译器生成 Go 语言代码的标志。
   - 通过这个选项，`protoc` 会将 `.proto` 文件编译为 Go 代码。

3. **`plugins=grpc`**
   - `plugins=grpc` 选项启用 gRPC 插件。gRPC 是一种高效的 RPC 框架，`protoc` 会生成用于 gRPC 服务的 Go 代码。
   - 使用此选项时，生成的 Go 代码会包括 gRPC 所需的客户端和服务器代码。
   
   **解释：**
   - gRPC 服务和客户端代码包含了与 Protocol Buffers 相关的 `Client` 和 `Server` 接口方法，这些方法用于服务的通信。
   
4. **`import_path=mypackage`**
   - `import_path` 选项指定 Go 代码中 `import` 的包路径。
   - 这表示生成的 Go 文件将在 `mypackage` 包下。例如，生成的文件将通过 `import "mypackage/yourfile"` 来引用。
   
   **注意：**
   - `mypackage` 应该是你希望在 Go 项目中使用的包名称。它通常是你项目的 Go 模块路径或目录路径。

5. **`:.`**
   - `.` 表示当前目录。这指定了生成的 Go 代码应该放置在当前目录下。
   - 所以，生成的 Go 文件将被输出到执行命令时所在的目录。

6. **`*.proto`**
   - `*.proto` 是你要编译的 `.proto` 文件的通配符。这个命令会匹配当前目录下的所有 `.proto` 文件，并将它们作为输入传递给 `protoc` 编译器。
   
### **总结**

- 这个命令会将当前目录下所有的 `.proto` 文件编译成 Go 代码，并且会生成包含 gRPC 客户端和服务器实现的代码。
- 生成的 Go 文件将被放置在当前目录中，并且这些文件会被包含在 `mypackage` 包中。

### **示例：**

假设你有一个 `hello.proto` 文件，如下所示：

```proto
syntax = "proto3";

package hello;

service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

运行命令：

```bash
protoc --go_out=plugins=grpc,import_path=mypackage:. hello.proto
```

这会生成一个包含 gRPC 客户端和服务器代码的 Go 文件，类似于：

- `hello.pb.go`：包括与消息类型和 gRPC 服务相关的代码。
- `hello_grpc.pb.go`：包括 gRPC 相关的客户端和服务器代码。

这样，你就可以使用生成的 Go 代码来实现和调用 `Greeter` 服务。

### **常见问题：**
- **Go 插件未安装：**
  如果你遇到 `protoc-gen-go: program not found or is not executable` 错误，意味着你没有安装 Go 的插件。
  安装命令：
  ```bash
  go get google.golang.org/protobuf/cmd/protoc-gen-go
  ```

- **gRPC 插件未安装：**
  如果没有安装 gRPC 插件，可以通过以下命令安装：
  ```bash
  go get google.golang.org/grpc/cmd/protoc-gen-go-grpc
  ```
  
<br/>
**Grpc支持**
如果proto文件指定了RPC服务，protoc-gen-go可以生成与grpc相兼容的代码，我们仅需要将plugins=grpc参数传递给--go_out，就可以达到这个目的

```
protoc --go_out=plugins=grpc:. *.proto
```

<br/><br/><br/>
> <h2 id="简单Demo流程">简单Demo流程</h2>

**初始化目录**

```sh
grpc-hello-world/
├── certs
├── client
├── cmd
├── pkg
├── proto
│   ├── google
│   │   └── api
└── server
```
- certs：证书凭证
- client：客户端
- cmd：命令行
- pkg：第三方公共模块
- proto：protobuf的一些相关文件（含.proto、pb.go、.pb.gw.go)，google/api中用于存放annotations.proto、http.proto
- server：服务端

<br/><br/>

**制作证书**
在服务端支持Rpc和Restful Api，需要用到TLS，因此我们要先制作证书

进入certs目录，生成TLS所需的公钥密钥文件

**私钥**

```sh
openssl genrsa -out server.key 2048

openssl ecparam -genkey -name secp384r1 -out server.key
```

- openssl genrsa：生成RSA私钥，命令的最后一个参数，将指定生成密钥的位数，如果没有指定，默认512
- openssl ecparam：生成ECC私钥，命令为椭圆曲线密钥参数生成及操作，本文中ECC曲线选择的是secp384r1

**自签名公钥**

```
openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650
```

openssl req：生成自签名证书，-new指生成证书请求、-sha256指使用sha256加密、-key指定私钥文件、-x509指输出证书、-days 3650为有效期，此后则输入证书拥有者信息

**填写信息**

```sh
Country Name (2 letter code) [AU]:ShangHai
String too long, must be at most 2 bytes long
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:ShangHai
Locality Name (eg, city) []:ShangaHai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:HuangGang
Organizational Unit Name (eg, section) []:HuangGang.dev.use 
Common Name (e.g. server FQDN or YOUR name) []:HuangGang_personalCertificate       
Email Address []:harleysor@qq.com 
```

<br/>

**或者你也可以这样做：**

是的，你可以 **不创建 `openssl.cnf` 文件**，直接在命令行执行 `openssl` 命令来生成 **带 `SAN`（Subject Alternative Name）** 的证书。

 **🚀 直接执行 OpenSSL 命令**

```sh
openssl req -x509 -nodes -newkey rsa:4096 -keyout server.key -out server.pem -days 365 \
-subj "/C=CN/ST=ShangHai/L=ShangHai/O=HuangGang/OU=HuangGang.dev.use/CN=dev" \
-addext "subjectAltName = DNS:dev, DNS:localhost, IP:127.0.0.1"
```
---

### **🛠 参数解析**
1. **`-x509`**  → 生成自签名证书  
2. **`-nodes`**  → 不加密私钥（无需密码）  
3. **`-newkey rsa:4096`**  → 生成 4096-bit RSA 私钥  
4. **`-keyout server.key`**  → 生成私钥 `server.key`  
5. **`-out server.pem`**  → 生成证书 `server.pem`  
6. **`-days 365`**  → 证书有效期 365 天  
7. **`-subj`**  → 直接在命令行设置证书信息（避免交互输入）  
8. **`-addext "subjectAltName = DNS:dev, DNS:localhost, IP:127.0.0.1"`**  
   - 添加 `SAN`，支持：
     - `DNS:dev`
     - `DNS:localhost`
     - `IP:127.0.0.1`

---

### **✅ 这样你不需要 `openssl.cnf`，就能直接生成带 `SAN` 的证书！**  
然后，你可以在 **Go gRPC 客户端** 中这样使用：

```go
creds, err := credentials.NewClientTLSFromFile("server.pem", "dev")
```
确保 `"dev"` 这个名称 **和 `SAN` 里的 `DNS:dev` 匹配**，这样 TLS 验证就不会失败了！ 🚀

这里的 "dev" 需要匹配服务器证书的 CN 或 SAN，否则会导致 tls: failed to verify certificate 错误。

✅ 检查服务器证书的 CN 和 SAN：

```sh
openssl x509 -in ../certs/server.pem -text -noout | grep -E 'Subject:|DNS:'
```
如果输出类似：

```pgsql
Subject: CN = myserver.local
X509v3 Subject Alternative Name:
    DNS:myserver.local, IP Address:127.0.0.1
```

<br/>

**server.pem证书信息：**

```
openssl x509 -in ../certs/server.pem -text -noout | grep -E 'Subject:|DNS:'
  
// 我自己证书信息
Subject: C=CN, ST=ShangHai, L=ShangaHai, O=HuangGang, OU=HuangGang.dev.use, CN=HuangGang_personalCertificate, emailAddress=harleysor@qq.com
```

<br/>

**client_server.pem证书信息：**

```
openssl x509 -in ../certs/client_server.pem -text -noout | grep -E 'Subject:|DNS:'

Subject: C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use, CN=dev
        DNS:dev, DNS:localhost, IP Address:127.0.0.1
```



说明 证书的 CN 或 SAN 不是 dev，你应该修改 client.go：

```go
creds, err := credentials.NewClientTLSFromFile("../certs/server.pem", "myserver.local")
```
或者：

```go
creds, err := credentials.NewClientTLSFromFile("../certs/server.pem", "")
（"" 表示不检查证书的 CN/SAN，仅校验证书是否可信）
```

<br/><br/>
> <h3 id="proto">proto</h3>
**annotations.proto文件：**

```proto
annotations.proto文件如下代码

syntax = "proto3";

package google.api;
option go_package = "github.com/golang/protobuf/protoc-gen-go/descriptor;descriptor";


import "google/api/http.proto";
import "google/protobuf/descriptor.proto";

option java_multiple_files = true;
option java_outer_classname = "AnnotationsProto";
option java_package = "com.google.api";

extend google.protobuf.MethodOptions {
  // See `HttpRule`.
  HttpRule http = 72295728;
}
```

这段 Protobuf (`annotations.proto`) 代码定义了一个 **protobuf 扩展 (extension)**，用于给 gRPC 方法添加 HTTP 绑定，使得 gRPC 方法可以通过 RESTful API 进行访问。下面是详细解析：

---

### **1. Protobuf 版本**

```proto
syntax = "proto3";
```
这表示该文件使用的是 **Protocol Buffers v3 (proto3) 语法**，是目前 Google 推荐的版本。

---

### **2. `package google.api;`**

```proto
package google.api;
```
- 这表示该 `proto` 文件属于 `google.api` 这个命名空间。
- 避免不同 `proto` 文件的消息类型（message）和服务（service）冲突。

---

### **3. 依赖的 `proto` 文件**

```proto
import "google/api/http.proto";
import "google/protobuf/descriptor.proto";
```
- `google/api/http.proto`
  - 该文件定义了 **`HttpRule`**，用于描述 gRPC 方法如何映射到 HTTP 路由（REST API）。
  - `HttpRule` 允许将 gRPC 方法绑定到 `GET`、`POST` 等 HTTP 方法，使得 gRPC 服务器可以作为 RESTful API 访问。

- `google/protobuf/descriptor.proto`
  - 该文件定义了 **protobuf 反射 (reflection) 和扩展机制**，允许我们扩展 protobuf 自带的功能，例如：
    - 扩展 `MethodOptions`，给 RPC 方法增加额外的元信息（metadata）。

---

### **4. Java 相关的 `option` 选项**

```proto
option java_multiple_files = true;
option java_outer_classname = "AnnotationsProto";
option java_package = "com.google.api";
```
- `java_multiple_files = true;`
  - 生成 Java 代码时，**每个 `message` 和 `service` 都会生成独立的 Java 类**，而不是全部嵌套在一个类中。
  - 这样可以提高 Java 代码的可读性和管理性。

- `java_outer_classname = "AnnotationsProto";`
  - 如果 `java_multiple_files = false`，则所有定义的 `message` 和 `service` 会嵌套在 `AnnotationsProto` 这个外部类中。
  - 但这里 `java_multiple_files = true`，所以这个 `option` 影响不大。

- `java_package = "com.google.api";`
  - 生成的 Java 代码会放在 `com.google.api` 这个 Java 包下。

---

### **5. `extend google.protobuf.MethodOptions` 扩展 `MethodOptions`**

```proto
extend google.protobuf.MethodOptions {
  // See `HttpRule`.
  HttpRule http = 72295728;
}
```
这一部分是 **protobuf 的扩展 (extension) 机制**，用于给 gRPC 方法的 `MethodOptions` 添加 `http` 选项，使得 gRPC 方法可以通过 HTTP 访问：

- `extend google.protobuf.MethodOptions`：
  - `MethodOptions` 是 **protobuf 自带的 RPC 方法元数据**，用于存储 gRPC 方法的额外信息。
  - `extend` 关键字允许我们**扩展** `MethodOptions`，增加新的字段。

- `HttpRule http = 72295728;`
  - 这里定义了一个 `HttpRule` 类型的 `http` 字段，并且它的**唯一标识符**为 `72295728`。
  - 这个字段的作用是：**允许 gRPC 方法映射到 HTTP 端点（REST API）**。

---

### **6. 这个扩展的作用**
这个 `extend` 语句的作用是**允许在 gRPC 方法上定义 HTTP 规则**，以支持 HTTP/REST 访问。

#### **使用示例**
在其他 `.proto` 文件中，我们可以这样使用：

```proto
import "google/api/annotations.proto";
import "google/api/http.proto";

service MyService {
  rpc GetUserInfo (UserRequest) returns (UserResponse) {
    option (google.api.http) = {
      get: "/v1/user/{id}"
    };
  }
}
```
- `option (google.api.http) = { get: "/v1/user/{id}" }`
  - 这个 `option` 使用了 `annotations.proto` 定义的 `http` 扩展。
  - 使得 `GetUserInfo` 这个 gRPC 方法可以通过 **HTTP `GET /v1/user/{id}`** 访问。

**等效于 REST API：**

```http
GET /v1/user/123
```
会被转换为 gRPC 调用：

```proto
rpc GetUserInfo(UserRequest) returns (UserResponse);
```

---

### **7. `72295728` 这个数字的意义**
在 `protobuf` 里，每个字段都需要一个唯一的 **标识符 (Tag Number)**，如：

```proto
string name = 1;
int32 age = 2;
```
- 这里 `name` 的 Tag 是 `1`，`age` 的 Tag 是 `2`。

在 `extend google.protobuf.MethodOptions` 里：

```proto
HttpRule http = 72295728;
```
- `72295728` 是 **扩展字段的 Tag Number**，用于唯一标识 `http` 这个扩展字段。
- Google 选择了一个**高位数字**，避免与 Protobuf 内部字段冲突。
- 该编号在 Google API 设计指南里是固定的，不要随意更改。

---

### **8. `HttpRule` 作用**
`HttpRule`（在 `google/api/http.proto` 中定义）用于指定 gRPC 方法和 HTTP 请求之间的映射关系：

```proto
message HttpRule {
  string get = 1;
  string post = 2;
  string put = 3;
  string delete = 4;
  ...
}
```
因此，在 `.proto` 里，我们可以这样使用：

```proto
rpc CreateUser (UserRequest) returns (UserResponse) {
  option (google.api.http) = {
    post: "/v1/users"
    body: "*"
  };
}
```
这意味着：
- `CreateUser` 方法可以通过 **HTTP `POST /v1/users`** 访问。
- `body: "*"` 表示请求的 body 作为 gRPC 方法的参数。

---

### **总结**
1. **`extend google.protobuf.MethodOptions`** 是 Protobuf 的扩展机制，允许为 gRPC 方法添加自定义的 **HTTP 绑定选项**。
2. `HttpRule` 允许 gRPC 方法支持 RESTful API 调用，使得 gRPC 方法可以通过 HTTP（`GET`、`POST`、`PUT`、`DELETE`）访问。
3. `option (google.api.http)` 语法用于在 `.proto` 文件中启用 HTTP 绑定。
4. `72295728` 是 Google 预留的 **唯一标识符 (Tag Number)**，表示 `http` 这个扩展字段。
5. 在 **Google API 设计规范** 里，`annotations.proto` 主要用于 `gRPC-Gateway`，使 gRPC 方法可以通过 REST 访问。

---

### **实战意义**
- **如果你使用 gRPC**，但想让客户端用 REST API 访问，你可以使用 `google.api.http` 选项。
- **如果你在写 Protobuf API**，但希望支持 `gRPC-Gateway`，`annotations.proto` 是必不可少的。

---

这个 `annotations.proto` 是 Google API 设计中的关键部分，使得 **gRPC 和 REST API 可以无缝结合**，在 Google Cloud 和其他微服务架构中广泛使用。

<br/><br/>

**http.proto文件：**

```proto
syntax = "proto3";

package google.api;
option go_package = "github.com/golang/protobuf/protoc-gen-go/descriptor;descriptor";

option cc_enable_arenas = true;
option java_multiple_files = true;
option java_outer_classname = "HttpProto";
option java_package = "com.google.api";

message HttpRule {
  // Selects methods to which this rule applies.
  //
  // Refer to [selector][google.api.DocumentationRule.selector] for syntax details.
  string selector = 1;

  // Determines the URL pattern is matched by this rules. This pattern can be
  // used with any of the {get|put|post|delete|patch} methods. A custom method
  // can be defined using the 'custom' field.
  oneof pattern {
    // Used for listing and getting information about resources.
    string get = 2;

    // Used for updating a resource.
    string put = 3;

    // Used for creating a resource.
    string post = 4;

    // Used for deleting a resource.
    string delete = 5;

    // Used for updating a resource.
    string patch = 6;

    // Custom pattern is used for defining custom verbs.
    CustomHttpPattern custom = 8;
  }

  // The name of the request field whose value is mapped to the HTTP body, or
  // `*` for mapping all fields not captured by the path pattern to the HTTP
  // body. NOTE: the referred field must not be a repeated field.
  string body = 7;

  // Additional HTTP bindings for the selector. Nested bindings must
  // not contain an `additional_bindings` field themselves (that is,
  // the nesting may only be one level deep).
  repeated HttpRule additional_bindings = 11;
}

// A custom pattern is used for defining custom HTTP verb.
message CustomHttpPattern {
  // The name of this custom HTTP verb.
  string kind = 1;

  // The path matched by this custom verb.
  string path = 2;
}
```

`http.proto` 主要定义了 **gRPC 方法与 HTTP API 之间的映射规则**，核心是 `HttpRule` 这个消息（message），它允许我们把 gRPC 方法映射到 RESTful API，如 `GET`、`POST`、`PUT` 等 HTTP 方法。  

---

## **1. 头部选项 (`option`)**

```proto
option cc_enable_arenas = true;
option java_multiple_files = true;
option java_outer_classname = "HttpProto";
option java_package = "com.google.api";
```
这些选项用于控制代码生成：

- `cc_enable_arenas = true;`
  - **C++ 相关优化**，启用 **Arena Allocation** 机制，提高对象的内存管理效率，减少内存分配和回收的开销。

- `java_multiple_files = true;`
  - 生成的 Java 代码会为 `HttpRule`、`CustomHttpPattern` 等每个 message 单独创建一个 `.java` 文件，而不是全部嵌套在 `HttpProto` 这个类里。

- `java_outer_classname = "HttpProto";`
  - 设定 Java 代码的外部类名为 `HttpProto`（如果 `java_multiple_files = false`）。

- `java_package = "com.google.api";`
  - 生成的 Java 代码放在 `com.google.api` 这个 Java 包下。

---

## **2. `HttpRule` 结构**

```proto
message HttpRule {
  string selector = 1;

  oneof pattern {
    string get = 2;
    string put = 3;
    string post = 4;
    string delete = 5;
    string patch = 6;
    CustomHttpPattern custom = 8;
  }

  string body = 7;

  repeated HttpRule additional_bindings = 11;
}
```
`HttpRule` 主要用于**将 gRPC 方法映射到 HTTP 端点**，它包含以下字段：

### **(1) `selector`**

```proto
string selector = 1;
```
- `selector` 指定要应用 HTTP 规则的 **gRPC 方法**，其值是 `package.service/method` 形式。
- 例如：

```proto
selector = "my.package.MyService.GetUserInfo";
```

### **(2) `pattern` (HTTP 方法绑定)**

```proto
oneof pattern {
  string get = 2;
  string put = 3;
  string post = 4;
  string delete = 5;
  string patch = 6;
  CustomHttpPattern custom = 8;
}
```
- 这是 **`oneof` 字段**，即**同一时间只能选其中一个**。
- 允许 gRPC 方法映射到标准 HTTP 方法：
  - `get = 2;`  → HTTP `GET` 请求
  - `put = 3;`  → HTTP `PUT` 请求
  - `post = 4;` → HTTP `POST` 请求
  - `delete = 5;` → HTTP `DELETE` 请求
  - `patch = 6;` → HTTP `PATCH` 请求
  - `custom = 8;` → **自定义 HTTP 方法**（如 `OPTIONS`）

#### **示例**

```proto
rpc GetUser (GetUserRequest) returns (User) {
  option (google.api.http) = {
    get: "/v1/user/{id}"
  };
}
```
这里的 `get: "/v1/user/{id}"` 表示：
- gRPC 方法 `GetUser` 可以通过 HTTP `GET /v1/user/123` 访问，并将 `{id}` 绑定到 `GetUserRequest` 参数。

---

### **(3) `body` (HTTP 请求体绑定)**

```proto
string body = 7;
```
- 指定哪个请求字段映射到 HTTP `body` 中。
- 特殊值：
  - `"*"` → **全部字段** 作为 `body`
  - `""`（空字符串）→ **不映射 `body`**，参数需通过 URL 查询参数传递。

#### **示例**

```proto
rpc CreateUser (CreateUserRequest) returns (User) {
  option (google.api.http) = {
    post: "/v1/users"
    body: "*"
  };
}
```
等效于 REST API：

```http
POST /v1/users
Content-Type: application/json

{
  "name": "Alice",
  "age": 25
}
```
- `body: "*"` 让整个 `CreateUserRequest` 作为 HTTP `body` 传递。

如果 `body: "user"`：

```proto
message CreateUserRequest {
  User user = 1;
}
```
则 HTTP `body` 变成：

```json
{
  "user": {
    "name": "Alice",
    "age": 25
  }
}
```

---

### **(4) `additional_bindings` (额外的 HTTP 规则)**

```proto
repeated HttpRule additional_bindings = 11;
```
- 允许一个 gRPC 方法有 **多个 HTTP 绑定**（比如既支持 `GET` 也支持 `POST`）。
- 但**不能嵌套多层**（即 `additional_bindings` 不能再包含 `additional_bindings`）。

#### **示例**

```proto
rpc GetUser (GetUserRequest) returns (User) {
  option (google.api.http) = {
    get: "/v1/user/{id}"
    additional_bindings {
      post: "/v1/user"
      body: "*"
    }
  };
}
```
等效于：
- `GET /v1/user/123` → 通过 URL 查询参数传递 `id`
- `POST /v1/user` + `body: "*"` → 通过 `body` 传递 `id`

---

## **3. `CustomHttpPattern` 结构**

```proto
message CustomHttpPattern {
  string kind = 1;
  string path = 2;
}
```
- 允许使用自定义的 HTTP 方法（如 `OPTIONS`、`HEAD`）。
- `kind`：自定义 HTTP 方法的名称，如 `OPTIONS`。
- `path`：HTTP 端点。

#### **示例**

```proto
rpc CustomOptions (Empty) returns (Empty) {
  option (google.api.http) = {
    custom {
      kind: "OPTIONS"
      path: "/v1/custom"
    }
  };
}
```
等效于：

```http
OPTIONS /v1/custom
```

---

## **总结**
1. **`HttpRule` 作用**
   - 允许 gRPC 方法映射到 HTTP API，使 gRPC 可通过 RESTful API 访问。
   - `get`、`post`、`put`、`delete`、`patch` 定义 HTTP 方法绑定。
   - `body` 决定请求参数是否放入 `body`（默认放 URL 查询参数）。
   - `additional_bindings` 允许一个 gRPC 方法支持多种 HTTP 映射。
   - `custom` 允许定义自定义 HTTP 方法（如 `OPTIONS`）。

2. **示例**

```proto
rpc CreateUser (CreateUserRequest) returns (User) {
 option (google.api.http) = {
   post: "/v1/users"
   body: "*"
 };
}
```
   - gRPC 方法 `CreateUser` → `POST /v1/users`
   - `body: "*"` 表示整个请求体作为 HTTP `body`。

3. **与 `annotations.proto` 结合**
   - `annotations.proto` 通过 `extend MethodOptions` **把 `HttpRule` 绑定到 gRPC 方法**。
   - 让 `gRPC-Gateway` **自动转发** RESTful API 调用到 gRPC 服务。

---

这就是 `http.proto` 的核心，它是 `gRPC-Gateway` 和 Google Cloud API 的关键组件，使 gRPC 既能支持高效的 RPC，又能兼容 RESTful API 🚀

<br/><br/>
> <h3 id="proto编译成go代码">proto编译成go代码</h3>

```sh
# 编译google.api
# 首先cd到proto目录
ganghuang@GangHuangs-MacBook-Pro proto% cd MLC_GO/TestNotes/PracticeGRPCExample/proto 
protoc -I . \
--go_out=Mgoogle/api/http.proto=github.com/golang/protobuf/protoc-gen-go/descriptor:. \
--go-grpc_out=Mgoogle/api/http.proto=github.com/golang/protobuf/protoc-gen-go/descriptor:. \
google/api/*.proto

# 编译hello_http.proto为hello_http.pb.proto
ganghuang@GangHuangs-MacBook-Pro proto % protoc -I . \
--go_out=Mgoogle/api/annotations.proto=grpc-hello-world/proto/google/api,\
Mhello.proto=github.com/your-username/your-repo/grpc-hello-world/proto:. \
--go-grpc_out=Mgoogle/api/annotations.proto=grpc-hello-world/proto/google/api,\
Mhello.proto=github.com/your-username/your-repo/grpc-hello-world/proto:. \
./hello.proto

# 编译hello_http.proto为hello_http.pb.gw.proto
protoc -I . -I ./third_party/googleapis \
--grpc-gateway_out=logtostderr=true,\
Mhello.proto=github.com/your-username/your-repo/grpc-hello-world/proto,\
Mgoogle/api/annotations.proto=github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis/google/api:. \
./hello.proto
```
执行完毕后将生成hello.pb.go和hello.gw.pb.go，分别针对grpc和grpc-gateway的功能支持

![go.0.0.84.png](./../Pictures/go.0.0.84.png)

<br/><br/>
> <h3 id="命令行模块cmd">命令行模块cmd</h3>

这一小节我们编写命令行模块，为什么要独立出来呢，是为了将cmd和server两者解耦，避免混淆在一起。

- 我们采用 [Cobra](#CLI（命令行界面）Cobra库) 来完成这项功能，Cobra既是创建强大的现代CLI应用程序的库，也是生成应用程序和命令文件的程序。提供了以下功能：
	- 简易的子命令行模式
	- 完全兼容posix的命令行模式(包括短和长版本)
	- 嵌套的子命令
	- 全局、本地和级联flags
	- 使用Cobra很容易的生成应用程序和命令，使用cobra create appname和cobra add cmdname
	- 智能提示
	- 自动生成commands和flags的帮助信息
	- 自动生成详细的help信息-h，--help等等
	- 自动生成的bash自动完成功能
	- 为应用程序自动生成手册
	- 命令别名
	- 定义您自己的帮助、用法等的灵活性。
	- 可选与viper紧密集成的apps


<br/><br/><br/>

***
<br/>

> <h1 id="流式gRPC">流式gRPC</h1>
- **gRPC 的流式，分为三种类型：**
	- Server-side streaming RPC：服务器端流式 RPC
	- Client-side streaming RPC：客户端流式 RPC
	- Bidirectional streaming RPC：双向流式 RPC

![go.0.0.96.png](./../Pictures/go.0.0.96.png)

<br/>

**为什么不用 Simple RPC?**
&emsp; 流式为什么要存在呢，是 Simple RPC 有什么问题吗？通过模拟业务场景，可得知在使用 Simple RPC 时，有如下问题：
- 数据包过大造成的瞬时压力
- 接收数据包时，需要所有数据包都接受成功且正确后，才能够回调响应，进行业务处理（无法客户端边发送，服务端边处理）

<br/>

**为什么用 Streaming RPC?**
- 大规模数据包
- 实时场景

<br/>

**模拟场景**
每天早上 6 点，都有一批百万级别的数据集要同从 A 同步到 B，在同步的时候，会做一系列操作（归档、数据分析、画像、日志等）。这一次性涉及的数据量确实大

在同步完成后，也有人马上会去查阅数据，为了新的一天筹备。也符合实时性。

两者相较下，这个场景下更适合使用 Streaming RPC



<br/><br/><br/>

***
<br/>

> <h1 id="基于CA的TLS证书认证">基于 CA 的 TLS 证书认证</h1>
**CA**
为了保证证书的可靠性和有效性，在这里可引入 CA 颁发的根证书的概念。其遵守 X.509 标准


<br/>

**总结**
1. 生成 CA 私钥：ca.key
2. 生成自签名 CA 证书：ca.pem
3. 生成服务器 CSR：server.csr（使用 server.key）
4. 用 CA 签发服务器证书：server.pem
5. 生成客户端 EC 私钥：client.key
6. 生成客户端 CSR：client.csr
7. 用 CA 签发客户端证书：client.pem

这样一来，利用 CA 签发的证书就可以在 TLS/SSL 通信中使用，用于验证服务器和客户端的身份，实现双向认证和加密传输。每个步骤都是建立信任链的重要环节。

<br/><br/><br/>
> <h2 id="">根证书</h2>
**根证书**
根证书（root certificate）是属于根证书颁发机构（CA）的公钥证书。我们可以通过验证 CA 的签名从而信任 CA ，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书（客户端、服务端）

它包含的文件如下：
- 公钥
- 密钥

<br/>

**生成 Key(生成 CA 私钥)**

```sh
openssl genrsa -out ca.key 2048
```
- **作用：** 生成一个 2048 位的 RSA 私钥，用于后续创建 CA 证书。
- **参数说明**：
	- genrsa：生成 RSA 私钥。
	- -out ca.key：将生成的私钥保存到文件 ca.key 中。
	- 2048：指定密钥的位数（2048 位），数字越大安全性越高，但计算速度越慢。

<br/>

**生成密钥(基于 CA 私钥生成自签名的 CA 证书)**

```sh
openssl req -new -x509 -days 7200 -key ca.key -out ca.pem

# 填写信息
Country Name (2 letter code) [AU]:ShangHai
String too long, must be at most 2 bytes long
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:ShangHai
Locality Name (eg, city) []:ShangHai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:HuangGang
Organizational Unit Name (eg, section) []:HuangGang.dev.use 
Common Name (e.g. server FQDN or YOUR name) []:HuangGang.dev.use 
Email Address []:harleysor@qq.com
```

- **作用：** 使用上一步生成的 CA 私钥创建一个自签名证书（CA 证书），用于签发其他证书。
- **参数说明：**
	- req：生成证书签名请求（CSR）或自签名证书。
	- -new：表示生成一个新的证书请求或自签名证书。
	- -x509：指定生成的是一个 X.509 格式的自签名证书，而不是 CSR。
	- -days 7200：证书的有效期为 7200 天（约 20 年）。
	- -key ca.key：使用之前生成的 CA 私钥。
	- -out ca.pem：将生成的 CA 证书保存到文件 ca.pem 中。

<br/><br/><br/>
> <h2 id="Server"> Server </h2>
<br/>

**服务器证书生成**
**私钥(生成服务器的椭圆曲线私钥)**

```sh
openssl ecparam -genkey -name secp384r1 -out server.key
```

- openssl ecparam
	- 该子命令用于操作椭圆曲线（Elliptic Curve，简称 EC）的参数。可以生成 EC 参数、打印参数信息，或直接生成 EC 密钥。
- -genkey
	- 指示生成一个新的 EC 私钥，而不是仅仅显示参数信息。
- -name secp384r1
	- 指定生成私钥所使用的曲线。
		- secp384r1 是一种常用的椭圆曲线，安全性较高，生成的密钥长度和强度适中。
- -out server.key
	- 指定将生成的私钥保存到文件 server.key 中

**作用:** 这条命令生成一把基于 secp384r1 曲线的椭圆曲线私钥，用于服务器的 TLS 证书签名和加密。生成的私钥将保存在 server.key 文件中。

<br/>

**‌ 基于私钥生成自签名的服务器证书**
**自签公钥**

```sh
openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650

# 填写信息
Country Name (2 letter code) []:
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:go-grpc-example
Email Address []:
```

- **openssl req**  
		- 该子命令用于创建证书签名请求（CSR）或直接生成自签名证书。

- **-new**  
	- 表示生成一个新的证书请求或证书。

- **-x509**  
	- 表示生成一个自签名的 X.509 证书，而不是生成 CSR。X.509 是数字证书的标准格式。自签名意味着该证书由自己签发，而不是通过第三方 CA。

- **-sha256**  
	- 指定在签名过程中使用 SHA-256 哈希算法，确保数字签名的安全性。

- **-key server.key**  
	- 指定使用前面生成的私钥（即 `server.key`）来签名证书。证书将与该私钥关联。

- **-out server.pem**  
	- 指定输出证书文件为 `server.pem`。通常 PEM 格式的证书文件以 `.pem` 后缀保存，内容是 Base64 编码的文本。

- **-days 3650**  
	- 设置证书的有效期为 3650 天（大约 10 年）。这表示从生成之日起，证书将保持有效 3650 天。

**整体作用：**
这条命令利用生成的私钥创建一个自签名的 X.509 证书，用于服务器身份认证和加密通信。生成的证书文件为 `server.pem`，使用 SHA-256 签名算法，有效期为 10 年。


<br/><br/>

**生成 CSR(为服务器生成证书签名请求 (CSR))**
CSR 是 Cerificate Signing Request 的英文缩写，为证书请求文件。主要作用是 CA 会利用 CSR 文件进行签名使得攻击者无法伪装或篡改原有证书

```sh
openssl req -new -key server.key -out server.csr

# 填写信息
Country Name (2 letter code) [AU]:ShangHai
String too long, must be at most 2 bytes long
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:ShangHai
Locality Name (eg, city) []:ShangHai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:HuangGang
Organizational Unit Name (eg, section) []:HuangGang.dev.use 
Common Name (e.g. server FQDN or YOUR name) []:HuangGang.dev.use 
Email Address []:harleysor@qq.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:109
String too short, must be at least 4 bytes long
A challenge password []:h109
An optional company name []:HuangGang
```

- **作用：** 基于服务器的私钥 server.key 生成一个证书签名请求（CSR），用于向 CA 申请签发服务器证书。
- **参数说明：**
- req -new：创建新的证书签名请求。
- -key server.key：使用已有的服务器私钥（文件 server.key 必须事先生成）。
- -out server.csr：将生成的 CSR 输出到文件 server.csr。
- 注意：生成服务器私钥 server.key 的步骤与生成 CA 私钥类似，也可以使用 openssl genrsa -out server.key 2048（如果你还未生成服务器私钥）

> 注意：生成服务器私钥 server.key 的步骤与生成 CA 私钥类似，也可以使用 openssl genrsa -out server.key 2048（如果你还未生成服务器私钥）。

<br/><br/>

**基于 CA 签发( 使用 CA 签发服务器证书)**

```sh
openssl x509 -req -sha256 -CA ca.pem -CAkey ca.key -CAcreateserial -days 3650 -in server.csr -out server.pem

Certificate request self-signature ok
subject=C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use , CN=HuangGang.dev.use , emailAddress=harleysor@qq.com
```

- **作用：** 利用 CA 的私钥和证书，对服务器的 CSR 进行签名，生成服务器的正式证书。
- **参数说明：**
	- x509：用于证书相关操作，比如签发证书。
	- -req：表示输入文件是一个证书签名请求（CSR）。
	- -sha256：使用 SHA-256 哈希算法进行签名。
	- -CA ca.pem：指定 CA 的证书（之前生成的 ca.pem）。
	- -CAkey ca.key：指定 CA 的私钥（之前生成的 ca.key）。
	- -CAcreateserial：如果没有现成的序列号文件，则自动创建一个（生成文件 ca.srl），用于记录签发证书的序列号。
	- -days 3650：签发的服务器证书有效期为 3650 天（约 10 年）。
	- -in server.csr：使用之前生成的服务器证书签名请求。
	- -out server.pem：生成的服务器证书保存到文件 server.pem。

<br/><br/>
**‼️注意:使用上述命令在运行时后会出现下面的错误:**

```sh
❌ [客户端tls- client.Search err:  rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate is not valid for any names, but wanted to match HuangGang.dev.use"]
```

这是因为缺少SAN导致的,下面有解释,这里不过多赘述了!这里有2种方案解决:
- 一是创建san.cnf文件(下面有详细步骤);
- 2️⃣是通过命令,一行即解决.

**下面通过命令解决,不用san.cnf文件:**

是的，你可以 **不使用 `san.cnf` 文件**，直接在命令行中指定 `SAN` 选项。  

- **使用 `-addext` 选项重新生成 CSR 并包含 SAN**

```sh
openssl req -new -key server.key -out server.csr -subj "/C=CN/ST=ShangHai/L=ShangHai/O=HuangGang/OU=HuangGang.dev.use/CN=HuangGang.dev.use/emailAddress=harleysor@qq.com" -addext "subjectAltName=DNS:HuangGang.dev.use,DNS:grpc.example.com,DNS:localhost,IP:127.0.0.1"
```
**说明**：
- `-subj` 直接指定证书的 `CN`（Common Name）和其他字段。
- `-addext "subjectAltName=..."` 直接在命令行添加 **SAN 信息**。

---

- **使用 `-extfile` 选项（无 `san.cnf`）签发证书**
如果要 **签发证书时** 添加 SAN，可以这样：

```sh
openssl x509 -req -in server.csr -CA ../ca.pem -CAkey ../ca.key -CAcreateserial -out server.pem -days 3650 -sha256 -extfile <(printf "subjectAltName=DNS:HuangGang.dev.use,DNS:grpc.example.com,IP:127.0.0.1")
```
**`<(printf "...")` 作用**：  
- 它相当于一个**临时文件**，直接传递 OpenSSL 需要的 `subjectAltName` 信息，**不需要创建 `san.cnf` 文件**。

---

- **验证 SAN 是否生效**

```sh
openssl x509 -in server.pem -text -noout | grep -A1 "Subject Alternative Name"

#比如:
openssl x509 -in server.pem -text -noout | grep -A1 "HuangGang.dev.use"
```
如果正确，应该输出：

```plaintext
X509v3 Subject Alternative Name:
    DNS:HuangGang.dev.use, DNS:grpc.example.com, IP Address:127.0.0.1
```

<br/>

**然后，使用 CA 签发证书：**

```
 openssl x509 -req -in server.csr -CA ../ca.pem -CAkey ../ca.key -CAcreateserial -out server.pem -days 3650 -sha256 -extfile <(printf "subjectAltName=DNS:HuangGang.dev.use,DNS:grpc.example.com,IP:127.0.0.1")

Certificate request self-signature ok
subject=C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use, CN=HuangGang.dev.use, emailAddress=harleysor@qq.com
```





<br/><br/><br/>
> <h2 id="Client">Client</h2>
**生成 Key(生成客户端的 EC 私钥)**

```sh
openssl ecparam -genkey -name secp384r1 -out client.key
```

- **作用：** 生成一个椭圆曲线（EC）私钥，用于客户端证书。这里使用的曲线是 secp384r1。
- **参数说明：**
	- ecparam：操作椭圆曲线参数和密钥。
	- -genkey：生成一个新的密钥。
	- -name secp384r1：指定使用的椭圆曲线为 secp384r1，这种曲线安全性较高。
	- -out client.key：将生成的客户端私钥保存到文件 client.key。


<br/>

**生成 CSR(为客户端生成证书签名请求 (CSR))**

```sh
openssl req -new -key client.key -out client.csr

Country Name (2 letter code) [AU]:ShangHai
String too long, must be at most 2 bytes long
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:ShangHai    
Locality Name (eg, city) []:ShangHai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:HuangGang
Organizational Unit Name (eg, section) []:HuangGang.dev.use 
Common Name (e.g. server FQDN or YOUR name) []:HuangGang.dev.use 
Email Address []:harleysor@qq.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:h109
An optional company name []:h109
```

- **作用：** 使用客户端私钥 client.key 生成一个 CSR，用于向 CA 申请签发客户端证书。
- **参数说明：**
	- 与生成服务器 CSR 类似，-new 表示新建请求，-key client.key 指定客户端私钥，-out client.csr 指定输出文件为 client.csr。


<br/>

**基于 CA 签发(使用 CA 签发客户端证书)**

```sh
openssl x509 -req -sha256 -CA ca.pem -CAkey ca.key -CAcreateserial -days 3650 -in client.csr -out client.pem

Certificate request self-signature ok
subject=C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use, CN=HuangGang.dev.use, emailAddress=harleysor@qq.com
```

- **作用：** 利用 CA 的私钥和证书，对客户端 CSR 进行签名，生成客户端的正式证书。
- **参数说明：**
	- 参数与签发服务器证书类似：
		- -req 表示输入为 CSR。
		- -sha256 使用 SHA-256 签名。
		- -CA ca.pem 和 -CAkey ca.key 指定 CA 的证书和私钥。
		- -CAcreateserial 自动创建序列号文件（如果尚不存在）。
		- -days 3650 设置证书有效期为 3650 天（约 10 年）。
		- -in client.csr 指定客户端 CSR。
		- -out client.pem 指定输出客户端证书文件为 client.pem。

<br/><br/>

**查看xxx.pem证书信息,这里查看server.pem证书:**

```sh
openssl x509 -in server.pem -text -noout

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            20:f9:6b:1f:e5:50:b7:c4:3c:a4:25:5e:f3:01:5f:49:5b:50:a5:26
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use, CN=HuangGang.dev.use, emailAddress=harleysor@qq.com
        Validity
            Not Before: Mar 16 12:19:31 2025 GMT
            Not After : Mar 14 12:19:31 2035 GMT
        Subject: C=CN, ST=ShangHai, L=ShangHai, O=HuangGang, OU=HuangGang.dev.use , CN=HuangGang.dev.use , emailAddress=harleysor@qq.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:9b:dc:54:9b:38:ed:39:45:09:42:e7:91:19:64:
                    55:68:2e:60:69:86:20:fe:c5:ef:2a:14:7c:27:65:
                    61:52:74:9c:c2:af:81:c6:c3:86:b5:f1:b7:01:09:
                    e9:0a:71:cd:4a:60:fc:97:f8:03:1e:30:e3:51:a7:
                    8e:b3:10:b5:8f:b6:7a:87:74:9e:6d:f3:6b:98:2d:
                    88:e8:28:2c:1d:7c:6d:cd:66:a5:b4:99:f4:27:0d:
                    9b:73:6e:2a:78:56:aa
                ASN1 OID: secp384r1
                NIST CURVE: P-384
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                32:55:DE:90:2A:E1:C1:70:90:6E:C5:B6:18:8C:19:55:66:8A:72:D3
            X509v3 Authority Key Identifier: 
                C0:65:8D:CE:89:F7:65:0B:C9:74:1F:36:F7:C9:D1:10:36:E4:C8:61
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        79:d6:af:11:62:64:4b:72:12:ad:6c:fd:c6:97:4f:49:22:58:
        b9:34:52:23:42:5b:79:96:ff:33:79:97:5a:1f:4c:33:34:69:
        1e:e7:ff:f6:27:2c:82:22:29:74:6f:60:28:ee:46:ea:07:3a:
        2b:f3:84:7c:e9:4a:fa:f0:31:3b:72:84:3e:e0:fe:60:77:04:
        ce:5e:bc:ff:31:ae:2b:de:5e:36:bb:b3:45:3b:9c:da:73:64:
        49:c6:3a:b6:3c:33:b5:fa:59:15:05:0c:26:95:da:b3:7c:68:
        61:4d:99:7b:bf:01:67:de:c8:09:b2:97:67:de:5b:63:27:b1:
        70:47:8a:dc:10:8c:e0:98:ab:57:9f:d9:0f:47:5a:43:37:2d:
        9b:19:6c:5a:d5:46:c9:e4:61:e9:ec:0c:ef:cd:9d:5b:46:a9:
        76:ef:f4:64:ee:b8:ef:88:5f:0a:d5:e3:2e:9e:10:84:c4:72:
        3d:87:2a:eb:01:22:52:a1:2b:98:45:03:7f:e1:bd:d6:af:4d:
        c8:4e:3b:c2:f0:9f:ba:72:6e:70:62:e1:d1:84:e6:9b:c0:9a:
        5c:d8:15:55:4b:ca:89:34:2e:7e:ec:83:44:03:b4:c4:db:f6:
        36:ec:eb:82:07:10:fb:91:b4:2f:26:31:0c:29:c3:a2:b8:6e:
        48:94:42:94
```

---

<br/><br/>

**运行后会出现错误:**

```sh
❌ [客户端tls- client.Search err:  rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate is not valid for any names, but wanted to match HuangGang.dev.use"]
```

从你提供的证书信息来看，没有看到 S**AN（Subject Alternative Name） 字段**，这可能是导致 TLS 认证失败的原因。SAN 主要用于存放多个域名或 IP 地址，使得客户端可以正确验证服务器证书。

在 OpenSSL 输出中，如果证书包含 SAN，应该能看到类似于以下的部分：

```sh
X509v3 extensions:
    X509v3 Subject Alternative Name:
        DNS:example.com, DNS:*.example.com, IP Address:192.168.1.1
```
但你的证书输出没有这个字段，说明 SAN 可能未被设置。

<br/>

**为什么需要 SAN？**

在 gRPC 认证时，客户端会检查服务器证书是否匹配预期的域名（ServerName）。如果你的证书没有 Subject Alternative Name (SAN)，但 gRPC 需要验证 ServerName，就会导致 TLS 认证失败，如你的错误信息所示：

```sh
tls: failed to verify certificate: x509: certificate is not valid for any names, but wanted to match gRPC_practice-gRPC_practice_v2
```
添加 SAN 后，就不会再遇到这个问题了。

**不可以直接修改已签发的证书的 SAN 信息。**  

证书一旦被 CA 签名，就无法直接修改其内容（包括 SAN 字段）。如果你的证书没有 SAN，你需要 **重新生成 CSR 并使用原有 CA 重新签发新的证书**。但你 **可以复用原有的私钥**，这样不会影响之前的密钥对。

---

<br/><br/>

**如何在现有证书的基础上添加 SAN？**
虽然你不能修改已有的证书，但你可以使用原来的 `server.key` 重新生成 CSR（证书签名请求），然后使用 CA 重新签名，加入 SAN 信息。

**1.确保你有原来的私钥**
如果你之前生成的私钥是 `server.key`，可以复用它：

```sh
ls -l server.key
```
如果私钥存在，就可以继续下面的步骤。

---

**2.生成新的 CSR（证书签名请求）**
创建一个新的 OpenSSL 配置文件 `san.cnf`（如果没有的话），并加入 SAN 字段：

```ini
[ req ]
default_bits       = 2048
prompt            = no
default_md        = sha256
distinguished_name = dn
req_extensions    = req_ext

[ dn ]
C  = CN
ST = ShangHai
L  = ShangHai
O  = HuangGang
OU = HuangGang.dev.use
CN = HuangGang.dev.use
emailAddress = harleysor@qq.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = HuangGang.dev.use
DNS.2 = grpc.example.com
IP.1  = 127.0.0.1
```

**请修改 `DNS.x` 和 `IP.x`，确保它们符合你的需求。**
<br/>

然后使用原来的 `server.key` 生成新的 `server.csr`：

```sh
openssl req -new -key server.key -out server.csr -config san.cnf
```
**这不会影响原来的私钥，只是生成了新的 CSR。**

---

- **3.使用 CA 重新签名新的证书**
用原来的 CA 重新签发证书：

```sh
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out server.pem -days 3650 -sha256 -extfile san.cnf -extensions req_ext
```
**这会生成一个新的 `server.pem` 证书，它包含 SAN 字段。**

---

- **4.验证 SAN 是否生效**
运行以下命令，检查新证书是否包含 SAN：

```sh
openssl x509 -in server.pem -text -noout | grep -A1 "Subject Alternative Name"
```
如果正确，应该能看到：

```plaintext
X509v3 Subject Alternative Name:
    DNS:HuangGang.dev.use, DNS:grpc.example.com, IP Address:127.0.0.1
```

---

- **5.替换旧证书并重启服务**
将新的 `server.pem` 复制到 gRPC 服务器，并重启它：

```sh
systemctl restart my_grpc_service  # 如果你的服务是 systemd 管理的
# 或者直接手动运行 gRPC 服务器
./my_grpc_server
```


<br/><br/><br/>

***
<br/>

> <h1 id="拦截器">拦截器</h1>
**在 gRPC 中，大类可分为两种 RPC 方法，与拦截器的对应关系是：**
- 普通方法：一元拦截器（grpc.UnaryInterceptor）
- 流方法：流拦截器（grpc.StreamInterceptor）

<br/>

**如何实现多个拦截器**

另外，可以发现 gRPC 本身居然只能设置一个拦截器，难道所有的逻辑都只能写在一起？

关于这一点，你可以放心。采用开源项目 **go-grpc-middleware** 就可以解决这个问题，本章也会使用它。


<br/><br/><br/>

***
<br/>

> <h1 id="gRPC提供HTTP接口">gRPC提供HTTP接口</h1>
[**同时提供 RPC 和 RESTful JSON API 两种接口**](https://segmentfault.com/a/1190000013339403)
**前言**
- 接口需要提供给其他业务组访问，但是 RPC 协议不同,无法内调.对方问能否走 HTTP 接口，怎么办？
- 微信（公众号、小程序）等第三方回调接口只支持 HTTP 接口，怎么办

在实际工作中都会都遇到如上问题，在 gRPC 中都是有解决方案的🤔

<br/>

**为什么可以同时提供 HTTP 接口**
关键一点，gRPC 的协议是基于 HTTP/2 的，因此应用程序能够在单个 TCP 端口上提供 HTTP/1.1 和 gRPC 接口服务（两种不同的流量）

**怎么同时提供 HTTP 接口?**
**检测协议**

```go
if r.ProtoMajor == 2 && strings.Contains(r.Header.Get("Content-Type"), "application/grpc") {
    server.ServeHTTP(w, r)
} else {
    mux.ServeHTTP(w, r)
}
```

**流程**
- 检测请求协议是否为 HTTP/2
- 判断 Content-Type 是否为 application/grpc（gRPC 的默认标识位）
- 根据协议的不同转发到不同的服务处理


<br/><br/><br/>

***
<br/>

> <h1 id="RPC自定义认证">RPC自定义认证</h1>


而在实际需求中，常常会对某些模块的 RPC 方法做特殊认证或校验。

```go
type PerRPCCredentials interface {
    GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)
    RequireTransportSecurity() bool
}
```
在 gRPC 中默认定义了 PerRPCCredentials，它就是本章节的主角，是 gRPC 默认提供用于自定义认证的接口，它的作用是将所需的安全认证信息添加到每个 RPC 方法的上下文中。其包含 2 个方法：
- GetRequestMetadata：获取当前请求认证所需的元数据（metadata）
- RequireTransportSecurity：是否需要基于 TLS 认证进行安全传输


<br/><br/><br/>
> <h2 id="截止时间Deadlines">截止时间Deadlines</h2>
**Deadlines**
Deadlines 意指截止时间，在 gRPC 中强调 TL;DR（Too long, Don’t read）并建议始终设定截止日期，为什么呢？

<br/>

**为什么要设置?**
当未设置 Deadlines 时，将采用默认的 DEADLINE_EXCEEDED（这个时间非常大）

如果产生了阻塞等待，就会造成大量正在进行的请求都会被保留，并且所有请求都有可能达到最大超时

这会使服务面临资源耗尽的风险，例如内存，这会增加服务的延迟，或者在最坏的情况下可能导致整个进程崩溃



