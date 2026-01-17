> # 
- [Kubernetes 是什么？](#Kubernetes是什么？)
- [安全的将数据库密码传给Go应用容器](#安全的将数据库密码传给Go应用容器)
	- [mysql-secret.yaml细解](#mysql-secret.yaml细解)		
	- [deployment.yaml详解](#deployment.yaml详解)		
	- [如何使用这些文件？](#如何使用这些文件？)


<br/><br/><br/>

***
<br/>

> <h1 id="Kubernetes 是什么？">Kubernetes 是什么？</h1>

&emsp; **`Kubernetes（常缩写为 K8s）`**是一个开源的容器编排平台，用来自动化部署、扩展和管理容器化应用（比如用 Docker 打包的 Go 程序）。  
在 K8s 中，所有资源（如 **`Pod、Deployment、Service、Secret 等`**）都通过 YAML 或 JSON 文件来定义。

<br/><br/><br/>

***
<br/>

> <h1 id="安全的将数据库密码传给Go应用容器">安全的将数据库密码传给Go应用容器</h1>

**有这么2段文件代码，如下：**

**`mysql-secret.yaml`文件**

```
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

<br/>

然后在 在 Deployment 中引用：

**`deployment.yaml `文件**

```
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

2段 `YAML` 文件是 `Kubernetes（简称 K8s）`中用于部署应用的配置文件，属于 **Kubernetes 资源清单（manifest）**。接下来用通俗易懂的方式解读这些代码的含义、作用以及如何使用。

<br/>

- **两个文件：**
	- `mysql-secret.yaml`：定义一个 **Secret**（机密信息）
	- `deployment.yaml`：定义一个 **Deployment**（部署配置）

共同作用：**安全地把数据库密码传给你的 Go 应用容器**。

***
<br/><br/><br/>
> <h2 id="mysql-secret.yaml细解">mysql-secret.yaml细解</h2>

**`mysql-secret.yaml` 详解**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_PASSWORD: c3VwZXJfc2VjcmV0X3Bhc3N3b3Jk
```

- **`apiVersion: v1`**
	- 表示使用的是 Kubernetes 的核心 API 版本 v1。`Secret` 是核心资源，所以用 `v1`。

<br/>

- **`kind: Secret`**
	- 告诉 Kubernetes：这是一个 **Secret 资源**。  
	- Secret 用于存储敏感数据，比如密码、token、密钥等。它会被加密存储（虽然默认只是 base64 编码，不是强加密，但比明文好），并且可以被 Pod 安全引用。

> ⚠️ 注意：K8s 的 Secret 默认只是 base64 编码（可逆），不是加密！生产环境建议启用 [Encryption at Rest](https) 或使用外部密钥管理（如 Vault）。

<br/>

- **`metadata.name: mysql-secret`**
	- 给这个 Secret 起个名字，叫 `mysql-secret`。后续其他资源（如 Deployment）可以通过这个名字引用它。

<br/>

- **`type: Opaque`**
	- 表示这是一个通用类型的 Secret（最常用）。还有其他类型如 `kubernetes.io/tls`（用于 TLS 证书）、`kubernetes.io/dockerconfigjson`（用于镜像拉取凭证）等。

<br/>

- **`data.MYSQL_PASSWORD: c3VwZXJfc2VjcmV0X3Bhc3N3b3Jk`**

- 这是重点！
	- `data` 字段下的所有值 **必须是 base64 编码的字符串**。
	- `MYSQL_PASSWORD` 是 key（键），`c3VwZXJfc2VjcmV0X3Bhc3N3b3Jk` 是 value（值）。

**那这个 base64 值是怎么来的？**

```bash
echo -n "super_secret_password" | base64
```

<br/>

执行这条命令：

```bash
$ echo -n "super_secret_password" | base64
c3VwZXJfc2VjcmV0X3Bhc3N3b3Jk
```

- `echo -n` 表示不加换行符（非常重要！否则解码会多出 `\n`）
- `base64` 把明文转成 base64 编码

> ✅ 所以，当你在 Pod 中读取这个 Secret 时，K8s 会自动帮你 **解码成原始字符串 `"super_secret_password"`**。

***
<br/><br/><br/>
> <h2 id="deployment.yaml详解">`deployment.yaml`详解</h2>


```yaml
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

<br/>

- **`apiVersion: apps/v1` + `kind: Deployment`**
	- 表示这是一个 **Deployment 控制器**。  
	- Deployment 是 K8s 中最常用的资源之一，用于管理 **Pod 的副本数、滚动更新、回滚**等。

<br/>

- **`spec.template.spec.containers[0]`**
	- Deployment 本身不直接运行容器，而是通过 **Pod 模板（template）** 来定义要运行的容器。

这里定义了一个容器：
- 名字：`app`
- 镜像：`your-go-app`（你自己的 Go 应用 Docker 镜像）

<br/>

- **`env`：设置环境变量**
	- Go 应用通常通过环境变量读取配置，比如：

```go
host := os.Getenv("MYSQL_HOST")
password := os.Getenv("MYSQL_PASSWORD")
```

K8s 允许你在容器启动时注入环境变量。

**前几个是普通变量（明文）：**

```yaml
- name: MYSQL_HOST
  value: "mysql.prod.svc.cluster.local"
```
- 这是 MySQL 服务在集群内的 DNS 地址（假设你有一个名为 `mysql` 的 Service，位于 `prod` 命名空间）。

**关键部分：从 Secret 读取密码**

```yaml
- name: MYSQL_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: MYSQL_PASSWORD
```

- **这表示：**
	- 环境变量 `MYSQL_PASSWORD` 的值 **不是直接写死**，而是从名为 `mysql-secret` 的 Secret 中获取。
	- 具体取 Secret 里的哪个字段？`key: MYSQL_PASSWORD`（就是你在 `mysql-secret.yaml` 里定义的那个 key）。

<br/>

✅ 当 Pod 启动时，K8s 会：
1. 找到 `mysql-secret` Secret
2. 读取 `MYSQL_PASSWORD` 对应的 base64 值
3. 自动解码成 `"super_secret_password"`
4. 设置为容器的环境变量 `MYSQL_PASSWORD`

这样，你的 Go 程序就能通过 `os.Getenv("MYSQL_PASSWORD")` 拿到密码，而 **YAML 文件里没有明文密码**！

***
<br/><br/><br/>
> <h2 id="如何使用这些文件？">如何使用这些文件？</h2>


- **1：准备你的 Go 应用**

确保你的 Go 程序能从环境变量读取数据库配置，例如：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	password := os.Getenv("MYSQL_PASSWORD")
	fmt.Println("DB Password:", password)
	// 实际连接数据库...
}
```

然后打包成 Docker 镜像，推送到镜像仓库（如 Docker Hub、Harbor 等），比如叫 `your-go-app:v1`。

> 注意：`deployment.yaml` 中的 `image: your-go-app` 要替换成你实际的镜像地址，比如 `docker.io/yourname/your-go-app:v1`

---
<br/>

- **步骤 2：应用 YAML 文件到 Kubernetes 集群**

假设你已经配置好 `kubectl`（K8s 命令行工具），并连接到集群。

```bash
# 1. 创建 Secret
kubectl apply -f mysql-secret.yaml

# 2. 创建 Deployment
kubectl apply -f deployment.yaml
```
<br/>

- **步骤 3：验证是否成功**

```bash
# 查看 Secret（注意：显示的是 base64，不是明文）
kubectl get secret mysql-secret -o yaml

# 查看 Pod 是否运行
kubectl get pods

# 进入 Pod 查看环境变量（密码会显示明文！谨慎操作）
kubectl exec -it <pod-name> -- env | grep MYSQL_PASSWORD
```

> ⚠️ 警告：`env` 命令会打印明文密码！仅用于调试，不要在生产环境随意执行。

---
<br/>

 **安全建议（重要！）**

1. **不要把 Secret 提交到 Git**  
   虽然用了 base64，但本质还是明文。建议：
	- 使用 `.gitignore` 忽略 `*-secret.yaml`
	- 用 CI/CD 工具动态生成 Secret（如 Helm + Vault / External Secrets Operator）

2. **限制 Secret 的访问权限**  
   通过 RBAC 控制哪些 ServiceAccount 能读取 Secret。

3. **考虑使用更安全的方案**  
	- [HashiCorp Vault](https://www.vaultproject.io/)
	- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
	- [Kubernetes External Secrets](https://external-secrets.io/)
	
---
<br/>

**✅ 总结**

| 文件 | 作用 |
|------|------|
| `mysql-secret.yaml` | 安全存储数据库密码（base64 编码） |
| `deployment.yaml` | 部署 Go 应用，并从 Secret 中注入密码作为环境变量 |

你的 Go 应用通过 `os.Getenv("MYSQL_PASSWORD")` 就能拿到密码，整个过程 **避免了在代码或配置文件中硬编码敏感信息**，符合安全最佳实践。

