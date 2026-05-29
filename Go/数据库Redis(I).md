> <h4/>
- [**简单介绍‌**](#简单介绍)
	- [**‌安装**](#安装)
	- [‌基本命令](#基本命令)
		- [服务端启动](#服务端启动) 
		- [启动redis客户端](#启动redis客户端)
		- [启动redis服务后一定启动它的客户端吗](#启动redis服务后一定启动它的客户端吗)
- [**协议**](#协议)
- [**操作Redis**](#操作Redis)
- [**十大数据类型**](#十大数据类型)
	- [Redis流(Stream)](#Redis流(Stream))
- [**Redis的MQ中间件**](#Redis的MQ中间件)  
	- [kafka消息队列](#kafka消息队列)
- [**专业概念详介**](#专业概念详介)
	- [Raft算法白话解释](#Raft算法白话解释)
	- [一致性hash和hash槽](#一致性hash和hash槽)
- [分布式限流-Lua脚本](#分布式限流-Lua脚本)
	- [Redis Lua脚本解读](#Redis_Lua脚本解读)
	- [临界突刺问题解决](#临界突刺问题解决)
- **资料**
	- [Redis资料-尚硅谷](https://github.com/harleyGit/Learning-in-practice/tree/master/Redis)
	- [Redis在线测试](https://try.redis.io/)
	- [go-redis使用入门](https://juejin.cn/post/7202521955366879288)


<br/><br/><br/>

***
<br/>

> <h1 id="简单介绍"> <font color='red'>**‌简单介绍**</font></h1>

Redis 的客户端和服务端是通过 TCP 连接来进行数据交互， 服务器默认的端口号为<font color='red'>**6379**</font>


客户端和服务器发送的命令或数据一律以 \r\n （CRLF）结尾（这是一条约定）


<br/><br/><br/>
> <h2 id="安装">安装</h2>
**安装:**

```sh
# 1. 安装 Go (1.13+ 版本)
brew install go

# 2. 验证安装
go version

# 3. 安装 Redis（用于本地测试）
brew install redis

# 4. 启动 Redis
brew services start redis
# 或手动启动
redis-server /usr/local/etc/redis.conf
```

<br/>

**客户端验证是否安装成功:**

```sh
% redis-cli -v
redis-cli 7.2.7

% redis-server -v
Redis server v=7.2.7 sha=00000000:0 malloc=libc bits=64 build=7f24f11dd7e42c58
```

<br/>

- Redis安装好后，默认有16个数据库，初始默认使用0号库，编号是0...15。
	- 添加key-val[set]
	- 查看当前redis的所有key[keys *]
	- 获取key对应的值[get key];
	- 切换redis数据库[select index];
		- 如何查看当前数据库的key-val数量[dbsize];
	- 清空当前数据库的key-val和清空所有数据库的key-val [flushdb flushall]

<br/><br/>

因为通过brew安装的redis,所以可以查看redis的信息:

```sh
brew info redis

==> redis: stable 7.2.7 (bottled), HEAD
Persistent key-value database, with built-in net interface
https://redis.io/
Conflicts with:
  valkey (because both install `redis-*` binaries)
Installed
/opt/homebrew/Cellar/redis/7.2.7 (15 files, 2.4MB) *
  Poured from bottle using the formulae.brew.sh API on 2025-03-13 at 15:11:58
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/r/redis.rb
License: BSD-3-Clause AND BSD-2-Clause AND BSL-1.0 AND MIT AND (CC0-1.0 OR BSD-2-Clause)
==> Dependencies
Required: openssl@3 ✔
==> Options
--HEAD
	Install HEAD version
==> Caveats
To restart redis after an upgrade:
  brew services restart redis
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/redis/bin/redis-server /opt/homebrew/etc/redis.conf
==> Analytics
install: 20,112 (30 days), 67,377 (90 days), 278,498 (365 days)
install-on-request: 20,029 (30 days), 67,145 (90 days), 275,783 (365 days)
build-error: 40 (30 days)
```

上述redis并不是在下载 Redis 本身，而是执行了一些联网行为**来获取最新信息.**

**查看redis的配置信息:**

```sh
open /opt/homebrew/etc/redis.conf
```

为了避免自己平时为了方便和自定义一些配置,将`redis.conf`重新拷贝出来重命名为`myRedis.conf`,然后在`.bash_profile文件`配置如下:

```sh
export REDIS_CONF_PATH=/Users/ganghuang/SDConfig/redis/myRedis.conf

source ~/.bash_profile
```


***
<br/><br/><br/>

># <h2 id="基本命令"> [基本命令](https://www.redis.net.cn/order/) </h2>
[**Redis命令参考**](http://doc.redisfans.com/index.html)

<br/><br/>
> <h3 id="服务端启动">服务端启动</h3>

**使用系统的配置文件`redis.conf`启动服务器命令:**

```sh
% redis-server

893:C 13 Mar 2025 15:31:21.307 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
893:C 13 Mar 2025 15:31:21.307 * Redis version=7.2.7, bits=64, commit=00000000, modified=0, pid=893, just started
893:C 13 Mar 2025 15:31:21.307 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
893:M 13 Mar 2025 15:31:21.307 * Increased maximum number of open files to 10032 (it was originally set to 2560).
893:M 13 Mar 2025 15:31:21.307 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 7.2.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 893
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

893:M 13 Mar 2025 15:31:21.308 # WARNING: The TCP backlog setting of 511 cannot be enforced because kern.ipc.somaxconn is set to the lower value of 128.
893:M 13 Mar 2025 15:31:21.308 * Server initialized
893:M 13 Mar 2025 15:31:21.308 * Ready to accept connections tcp



#关闭
快捷键: Ctrl+C即可
```

<br/>

使用自己的配置的`myRedis.conf`启动如下:

```sh
# 环境变量启动 redis-server
redis-server $REDIS_CONF_PATH

#关闭
快捷键: Ctrl+C即可
```

当初在使用这个的时候出错了,因为我是用brew启动了,导致6379端口占用了,无法启动! 解决如下:

```sh

```

<br/><br/>

**确认Redis是否在运行:**

```sh
// 检查 Redis 是否监听 6379 端口：
% lsof -i :6379;

COMMAND   PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
redis-ser 893 ganghuang    7u  IPv4 0xf4e154464036c58d      0t0  TCP *:6379 (LISTEN)
redis-ser 893 ganghuang    8u  IPv6 0xf4e154417ddefb3d      0t0  TCP *:6379 (LISTEN)

// 直接使用 Redis CLI 测试连接, 如果返回 PONG，说明 Redis 正常运行
% redis-cli ping;
PONG

# 若是出现端口占用,可以使用kill命令将无关占用的端口给杀掉
kill -9 端口号占用的PID
```

<br/>

**查看redis-server的PID(确认 Redis 启动状态)**

```sh
ps aux | grep redis
```

<br/><br/>
> <h3 id="启动redis客户端">启动redis客户端</h3> 

**本地启动客户端:**

```sh
# 启动客户端的时候要先启动服务端,否则无法连接
% redis-cli

# 出现如下: ip地址: 端口, 表示已经连上了
127.0.0.1:6379> 


# 退出
shutdown // 停止redis服务 
quit // 退出redis-cli 的交互模式
```

<br/>

**连接远程 Redis（如果有）**

```sh
# 连接本地: redis-cli -h 127.0.0.1 -p 6379
# 连接远端
redis-cli -h 192.168.1.100 -p 6379

// 单实例关闭: redis-cli -a 密码 shutdown
redis-cli -a 123456 shutdown，如果在Redis服务器里面可以直接使用shutdown命令
shutdown // 停止redis服务 
quit // 退出redis-cli 的交互模式


// 多实例关闭,指定端口关闭: redis-cli -p 端口 shutdown
redis-cli -p 6379 shutdown
```

<br/>

**常用语法**

```redis
keys * // 查询所有key

get [key] // 获取一条

# 慎用,强烈建议不要
flushall // 删除所有

set [key] [value] // 设置一条
```
***
<br/><br/><br/>
> <h2 id="启动redis服务后一定启动它的客户端吗">启动redis服务后一定启动它的客户端吗</h2>
**回答：不需要。**
在你已经启动 **`redis-server`** 的前提下，**程序中使用 `GET` / `SET` 完全不需要再启动 `redis-cli` 客户端**。

下面把**服务端 / 客户端 / 程序访问**三者的关系讲清楚，避免概念混淆。

---
<br/>

**1️⃣ `redis-server` —— **真正的服务端（必须启动）**

这是 Redis 的核心进程：

* 负责：
  * 内存数据存储
  * Key 管理
  * 网络监听（默认 6379）
* **你的 Go 程序是直接连它的**

<br/>

启动示例：

```bash
redis-server
# 或
redis-server /usr/local/etc/redis.conf
```

只要它在跑，你的 Go 程序就能 `Get / Set`。

<br/>

**2️⃣ `redis-cli` —— **命令行调试工具（可选）**

`redis-cli` 只是一个 **人用的客户端工具**：

* 用途：
  * 手动查看数据
  * 调试 key 是否存在
  * 测试 Redis 是否正常
* **和你的 Go 程序没有任何依赖关系**

<br/>

示例：

```bash
redis-cli
> GET user:code:xxx
```

👉 不启动 `redis-cli`，**Go 程序照样正常访问 Redis**。

<br/>

**3️⃣ 你的 Go 程序 —— **业务客户端**

```text
Go 程序
   |
   | TCP 6379
   v
redis-server
```

* Go 程序内部通过：

  * `go-redis`
  * 连接池
* 直接和 `redis-server` 通信
* 不需要 `redis-cli` 参与

---
<br/>

**为什么很多人会误以为“要启动 redis-cli”？**

**❌ 误解 1：redis-cli 像 MySQL client 一样“必须存在”**

实际上：

* MySQL CLI：也是调试工具
* Redis CLI：也是调试工具

**它们都不是服务的一部分**

<br/>

**❌ 误解 2：看到 redis-server 启动在前台**

```bash
redis-server
```

此时终端被占用，另一个终端里用：

```bash
redis-cli
```

让人误以为「cli 是必须配套的」

实际上：

* redis-server：必须
* redis-cli：只是你开的一个“观察窗口”

<br/>

**❌ 误解 3：程序 Get 失败，于是去启动 redis-cli**

如果你遇到：

```text
redis: connection pool: failed to dial
```

**根因一定不是没启动 redis-cli**，而是：

* redis-server 没启动
* 地址/端口错误
* client 初始化错误

---
<br/>

**什么时候“需要”用 redis-cli？**

仅在以下场景（开发 / 运维）：

**1️⃣ 验证 Redis 是否启动成功**

```bash
redis-cli ping
# PONG
```

<br/>

**2️⃣ 手动查看 / 清理数据**

```bash
redis-cli
> KEYS *
> GET xxx
> DEL xxx
```

<br/>

**3️⃣ 排查程序问题**

对比：

* 程序写入成功了吗？
* TTL 是否正确？





<br/><br/><br/>

***
<br/>

> <h1 id="协议">协议</h1>

**请求协议**
格式示例

```sh
*<参数数量> CR LF
$<参数 1 的字节数量> CR LF
<参数 1 的数据> CR LF
...
$<参数 N 的字节数量> CR LF
<参数 N 的数据> CR LF
```
在该协议下所有发送至 Redis 服务器的参数都是二进制安全（binary safe）的

<br/>

**打印示例**

```json
*3
$3
SET
$5
mykey
$7
myvalue
```

<br/>

**实际协议值**

```sh
"*3\r\n$3\r\nSET\r\n$5\r\nmykey\r\n$7\r\nmyvalue\r\n"
```
这就是 Redis 的请求协议规范，按照范例1编写客户端逻辑，最终发送的是范例3，相信你已经有大致的概念了，Redis 的协议非常的简洁易懂，这也是好上手的原因之一，你可以想想协议这么定义的好处在哪？

<br/>

**回复**

Redis 会根据你请求协议的不同（执行的命令结果也不同），返回多种不同类型的回复。在这个回复“协议”中，可以通过检查第一个字节，确定这个回复是什么类型，如下：

- 状态回复（status reply）的第一个字节是 “+”
- 错误回复（error reply）的第一个字节是 “-”
- 整数回复（integer reply）的第一个字节是 “:”
- 批量回复（bulk reply）的第一个字节是 “$”
- 多条批量回复（multi bulk reply）的第一个字节是 “*”
有了回复的头部标识，结尾的 CRLF，你可以大致猜想出回复“协议”是怎么样的，但是实践才能得出真理


<br/><br/><br/>

***
<br/>

> <h1 id="操作Redis">操作Redis</h1>
[go语言-选择redigo-查看 API](https://pkg.go.dev/github.com/gomodule/redigo/redis#pkg-examples)
[Redis API文档](https://pkg.go.dev/github.com/gomodule/redigo/redis)


**安装第三方包:**

```sh
go get github.com/gomodule/redigo@latest
```


<br/><br/>

- **主要分为 3 类：**
	- 连接数据库
		- API文档中，所有以 Dial 开头

	- 操作数据库
		- Do() 函数【推荐】
		- Send()函数, 需要配合Flush()、Receive() 3 个函数使用

	- 回复助手
		- 相当于 “类型断言”。根据使用的具体数据类型，选择调用
		- 将DO函数返回的interface{}确定为具体类型


**测试案例:**

```go

package redisexample

import (
	"fmt"

	"github.com/gomodule/redigo/redis"
)

// .int类型数据插入与查询, 控制台出现：9990，就代表成功了
func RedisExamplePractice() {
	c, err := redis.Dial("tcp", "127.0.0.1:6379")
	if err != nil {
		fmt.Println("连接 redis 失败 err:", err)
		return
	}

	defer c.Close()
	_, err = c.Do("Set", "abc", 9990)
	if err != nil {
		fmt.Println("redis 操作失败")
		return
	}

	r, err := redis.Int(c.Do("Get", "abc"))
	if err != nil {
		fmt.Println("redis get abc 操作失败")
		return
	}

	fmt.Println(r)
}

```

<br/><br/><br/>

***
<br/>

> <h1 id="十大数据类型">十大数据类型</h1>


***
<br/><br/><br/>

> <h2 id="Redis流(Stream)">Redis流(Stream)</h2>

<br/><br/><br/>

***
<br/>

> <h1 id="Redis的MQ中间件">Redis的MQ中间件</h1>

![go.0.0.181.png](./../Pictures/go.0.0.181.png)

***
<br/><br/><br/>

> <h2 id="kafka消息队列">kafka消息队列</h2>



<br/><br/><br/>

***
<br/>

> <h1 id="专业概念详介">专业概念详介</h1>


***
<br/><br/><br/>

> <h2 id="Raft算法白话解释">Raft算法白话解释</h2>

好呀！我来用**超白话**的方式给你解释一下 **Raft** 算法（保证你能秒懂，不硬啃理论 😂）。

---

# 🎯 Raft 是干嘛的？
**Raft** 是一种 "**让一堆服务器达成一致**" 的算法。  
也就是说，**即使网络很烂、机器坏掉，大家也能保持一条心，一起记账、不乱套。**

这种东西在分布式系统里超级重要，比如分布式数据库、分布式存储系统。

# 🎯 为什么需要 Raft？
想象一下：
- 你有5台服务器。
- 客户端来请求："给我存一条数据 A！"
- 结果其中一台机器断电了，或者两台机器收到的消息不一样。
- 怎么保证**大家最终都存的是同一条数据？**  
**不能各自乱来！**

这就是 Raft 要解决的问题。**一致性问题。**

---

# 🚀 Raft 的基本流程（用超简单的比喻）

想象一下，5个人开会，要做决定（比如吃什么外卖）。Raft的步骤就像是：

### 1. 选一个老大（Leader选举）
- 开会前，大家投票，先选一个**Leader（老大）**。
- 谁最快拉到多数人的支持，谁就当老大。
- 老大是**唯一能拍板的人**。

👉 技术上叫做 "**Leader Election**"

### 2. 有事找老大（日志复制）
- 客户（比如一个应用）来找，"我要存一条数据"。
- 只有老大有权利接收请求。
- 老大收到后，告诉其他小弟（Followers）："记下来这条！"
- 小弟们说 "收到！记好了！"  
- 等**大多数人**确认了，老大就告诉客户端："搞定了！"

👉 技术上叫 "**Log Replication**"

### 3. 出事了换老大（容错）
- 如果老大突然死了（比如掉线、宕机了），那其他人会发现：
  - "咦？老大一段时间没说话了。"
- 大家就重新开始投票，选一个新的老大。

👉 技术上叫 "**Leader Re-Election**"

---

# 🧠 小细节（高级一点点）
- 每一次选举、每一批数据，都会有一个 **任期（term）**。类似于“第几届老大”。
- 日志有顺序，每条数据都编号，保证先后顺序一致。
- 如果不同人存的数据有冲突，新的 Leader 会强制大家统一成正确的版本。

---

# 🔥 总结一把梳
| 概念            | 白话解释                                   |
|-----------------|-------------------------------------------|
| Leader          | 老大，统一指挥，拍板做决定                  |
| Followers       | 小弟，听从老大指挥                         |
| Term            | 任期，比如“第5届领导班子”                   |
| Log Replication | 老大发号施令，小弟们同步记录                 |
| Election        | 老大死了要重新投票                         |
| Commit          | 大多数同意后，才算正式记账                  |

---

# 🎯 Raft 的设计目标
> **“让一致性算法容易理解。”**

Raft就是为了比以前的Paxos算法更简单、更好懂而发明的。  
**所以你能懂，很正常，Raft本来就是为让人类看得懂设计的！**

---

要不要我顺便再用一个小动画或者流程图给你画一下，"选老大"和"同步数据"是怎么跑的？一张图就秒懂那种～要的话告诉我 😁



***
<br/><br/><br/>
> <h2 id="一致性hash和hash槽">一致性hash和hash槽</h2>
OK，来啦！咱们继续用**超白话**、**一步步讲懂**的方式，把「一致性哈希」和「哈希槽」都解释清楚～🚀

---

# 🎯 首先，什么是哈希（Hash）？
简单理解：
- 给一个东西（比如名字、数据）跑一个“压缩”算法，得到一个短的数字/值。
- 比如：`"apple"` → 哈希后变成 `2384723`
- 哈希的特点是：
  - 输入一样，输出一定一样。
  - 改一点点，结果差很多。

就像给每个东西打上一个**独特编号**。

---

# 🎯 为什么要用一致性哈希？
想象一个场景：
- 你有3台服务器存数据（A、B、C）。
- 用户数据太多了，分开存才扛得住。
- 你想用 Hash 把数据分到不同服务器，比如：
  - `hash(小明)` % 3 = 0 ➔ 存 A
  - `hash(小红)` % 3 = 1 ➔ 存 B
  - `hash(小张)` % 3 = 2 ➔ 存 C

一开始挺好。但是——
- 如果加一台新服务器 D（变成4台了）？
- 全部数据的 `% 4` 结果就变了。
- 大量老数据要重新搬家（超级痛苦）。

所以：**服务器一变化，所有数据几乎都要重新分配，这很坑。**

这时候就需要——  
✅ **一致性哈希**来解决问题！

---

# 🎯 一致性哈希是怎么解决的？（超简单比喻）

可以想象成：

1. 把整个 Hash 空间，画成一个 **圆圈**（像时钟盘，0点到24点）。
2. 每个服务器也算一遍 Hash，放到圆圈的某个位置。
   - 比如 A 落在 3点位置，B 落在 7点，C 落在 11点。
3. 存数据时，算数据的 Hash，看它落在圆圈哪个位置。
4. 按顺时针方向，找到第一个服务器，就是存它的地方。

举个例子：
- 数据 `小明` Hash 落在 5点 ➔ 顺时针第一个服务器是 B ➔ 存到 B。
- 数据 `小张` Hash 落在 12点 ➔ 顺时针回到最早的 A ➔ 存到 A。

✅ 这样，即使增加或减少服务器，只影响一小部分数据，其他数据不动！

---

# 🔥 一句话总结一致性哈希：
> **"用一个圆圈+顺时针找最近服务器，让新增/下线服务器只影响最小的数据量。"**

（解决大规模数据重新分配的问题！）

---

# 🎯 再说说哈希槽（Hash Slot）

哈希槽是另外一种更**细分**的策略，常见在像 Redis 这种场景。

理解起来非常简单：

可以把哈希槽想成——  
✅ **小格子、小房间，每个房间装一部分数据。**

比如：
- 总共有 16384 个哈希槽（Redis Cluster 规范就是 0 ～ 16383）。
- 每条数据根据 Hash 算出来，落到一个具体的槽号。
- 每台服务器（节点）负责一部分哈希槽。

比如分配是：
- 服务器 A：负责 0 ~ 5460
- 服务器 B：负责 5461 ~ 10922
- 服务器 C：负责 10923 ~ 16383

存数据时流程就是：
- 把 key（比如 "xiaoming"）算一下 CRC16 哈希，结果比如是 8193。
- 8193 落在 B 服务器负责的范围，所以把 "xiaoming" 存到 B。

---

# 🏆 那哈希槽到底是干嘛的呢？
- **预先固定了分区数（16384个）**，不会变来变去。
- 新加/移除服务器，只是重新分配"槽"的归属，不是重算所有数据。
- 管理更简单，迁移数据更快，集群伸缩性好。

---

# 🧠 小对比总结

| 名字          | 白话理解                         | 典型应用                   |
|--------------|------------------------------|---------------------------|
| 一致性哈希     | 把服务器放在圆圈上，顺时针找最近的服务器 | 大型存储系统、缓存系统，比如一致性哈希负载均衡 |
| 哈希槽         | 先划好一堆小格子（固定数量），每台服务器负责一些格子 | Redis Cluster |

---

# 🎨 最形象的比喻

| 一致性哈希            | 哈希槽             |
|-----------------------|-------------------|
| 一群人在钟表上排成圈，顺时针找人 | 一栋大楼分成16384个小房间，每个人守一些房间 |

---

# ⚡总结一句话大白话：
✅ 一致性哈希 = 圆圈找最近！  
✅ 哈希槽 = 小格子分配！




<br/><br/><br/>

***
<br/>

> <h1 id= "分布式限流-Lua脚本">分布式限流-Lua脚本</h1>

```go
client *redis.Client 

rateLimitLuaScript = `
	local current = redis.call('INCR', KEYS[1]) 
	if current == 1 then 
			redis.call('EXPIRE', KEYS[1], ARGV[2]) 
	end 
	if current > tonumber(ARGV[1]) then
		  return 0
	end
	
	return 1
	`
	
	
// 调用脚本
client.Eval(
    ctx,
    rateLimitLuaScript,
    []string{key},
    120,
    int(time.Minute.Seconds()),
).Int64()
```
这段代码本质上是在做：**‌ Redis 分布式限流（Rate Limit）**

这是 **`Redis Lua`,Redis** 内部执行。

**一般用于：**

* 接口防刷
* 登录频率限制
* 短信验证码限制
* API QPS 限制
* 用户请求次数限制

而且： **Lua脚本保证了 Redis 操作原子性‌**， 这是重点。

***
<br/><br/><br/>
> <h2 id="Redis_Lua脚本解读">Redis Lua脚本解读</h2>

## Redis Lua 是什么？

**Redis** 支持在 **Redis** 服务端执行 **Lua** 脚本,即：**多条Redis命令一次性原子执行, 避免并发问题.**

<br/>

```lua
// 对某个 key 执行 INCR 自增
local current = redis.call('INCR', KEYS[1])
```
等价于 Redis 命令：

```redis
INCR user:1001:limit
```

<br/>

### INCR什么意思？

- **Redis中**： 
	- `INCR key`会 `‌key 数字 +1`
		- 例如：
			- 第一次：`‌INCR test`, 结果：`1`
			- 第二次：`‌2`
			- 第三次：`‌3`
	- 所以：`‌current` 表示：**`‌当前请求次数`**

<br/>

### KEYS[1] 是什么？

- **Redis Lua：**
	- `KEYS`是`‌传进来的 Redis Key 数组`
	- 这里`‌[]string{key}`, 所以`‌KEYS[1]`就是`key`
		- 例如：**`"user:1001:api_limit"`**

<br/>

```sh
‌if current == 1 then
```

意思：**‌ 如果这是第一次访问**, 因为**‌ INCR 第一次一定返回1**

<br/>

```lua
redis.call('EXPIRE', KEYS[1], ARGV[2])
```
意思：**‌ 给 key 设置过期时间**
<br/>

**等价：**

```redis
EXPIRE user:1001:limit 60
```

---
<br/>

### ARGV 是什么？

Lua 中：`‌ARGV` 是`‌参数数组`，来自：

```go
120,
int(time.Minute.Seconds())
```
所以：

```text
ARGV[1] = 120
ARGV[2] = 60
```

注意：

Lua 数组从：**`1开始`**，**不是0。** 所以**`ARGV[2]`** 就是：**`60 秒`**

---
<br/>

### 为什么只在第一次设置 EXPIRE？

因为**后续请求不能刷新过期时间**，否则**限流永远不会失效。**

比较**‌正确和错误逻辑**

#### 1.正确逻辑（你的代码）

第一次：

```text
key=1
设置60秒过期
```

后面：

```text
2
3
4
```

但：

```text
TTL 不变
```

60秒后自动清空。

<br/>

#### 错误逻辑（每次都 EXPIRE）

如果每次请求：‌`都刷新60秒`,那`‌key 永远不会过期`,因为用户一直在请求。
<br/>

```go
client.Eval(
    ctx,
    rateLimitLuaScript,
    []string{key},
    120,
    int(time.Minute.Seconds()),
).Int64()
```

对应**Eval 参数**,Redis Go Client：

```go
Eval(ctx, script, keys, args...)
```

- **参数3**`‌[]string{key}`,传递给`‌KEYS`。所以`KEYS[1]`就是这个 key。

- **参数4、5**

```go
120,
int(time.Minute.Seconds())
```
传给：

```lua
ARGV
```

即：

| Lua     | 值   |
| ------- | --- |
| ARGV[1] | 120 |
| ARGV[2] | 60  |

意思：`‌1分钟最多120次`


---

### 整体执行流程

假设：

```go
key = "user:1001:api"
limit = 120
expire = 60秒
```

---

#### 第1次请求

```text
INCR -> 1
设置 EXPIRE 60秒
返回1
```

---

#### 第2次请求

```text
INCR -> 2
返回2
```

---

####  第121次请求

```text
INCR -> 121
超过120
拒绝
```

---

### 为什么用 Lua？

因为：

---

### 错误写法（不用 Lua）

很多人这样：

```go
count := GET key
if count > limit {
   reject
}
INCR key
```

问题：

---

### 并发会炸

多个请求同时：

```text
都读到119
```

然后都通过,结果`‌实际变成130+`

---

## 这其实是经典固定窗口限流

你这个属于`‌Fixed Window Rate Limiter`,即`‌固定时间窗口`,例如：`‌每分钟120次`

## 但它有缺点,会有**`‌临界突刺问题`**,例如：

```text
00:00:59 发120次
00:01:00 再发120次
```

瞬间：`‌240次`,所以高级系统会用：

* 滑动窗口
* 漏桶
* 令牌桶

**例如：**

* Sentinel
* Envoy
* Nginx
* Kong
* Istio

***
<br/><br/><br/>
> <h2 id="临界突刺问题解决">临界突刺问题解决</h2>

```go
// TokenBucketRateLimitLuaScript 是令牌桶限流脚本，用于替代固定窗口 INCR + EXPIRE。
// 固定窗口的问题：例如每分钟限制 120 次，用户可能在 00:59 打 120 次、01:00 再打 120 次，瞬间形成 240 次临界突刺。
// 令牌桶按时间连续补充 token，不依赖自然分钟边界，因此可以削平固定窗口的边界突刺。
// KEYS[1]：令牌桶 key，例如 video_upload:rate:user:{user_id} 或 video_upload:rate:ip:{ip}。
// ARGV[1]：桶容量 capacity，最多能积攒多少 token；容量越大，允许的短时 burst 越大。
// ARGV[2]：补充速率 refill_rate，单位 token/秒，例如 120 次/分钟 = 2 token/秒。
// ARGV[3]：当前时间 now_ms，单位毫秒，由 Go 传入，避免 Redis 版本差异影响 TIME 解析。
// ARGV[4]：本次请求消耗 token 数，普通上传请求消耗 1。
// ARGV[5]：Redis key 过期秒数，长时间无请求时自动清理桶状态。
// 执行逻辑：
// 1. 读取桶内剩余 tokens 和上次更新时间 updated_at。
// 2. 按 elapsed_seconds * refill_rate 计算这段时间应补充多少 token，但不超过 capacity。
// 3. 如果 tokens 足够支付本次请求，则扣减 token 并返回 1；否则不扣减并返回 0。
// 4. 更新 tokens/updated_at，并设置过期时间。
// 为什么用 Lua：读取、补充、扣减、写回必须原子执行；拆成多条 Redis 命令会被并发请求穿插，导致超发。
	TokenBucketRateLimitLuaScript = `
local capacity = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local now_ms = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])
local ttl_seconds = tonumber(ARGV[5])

local bucket = redis.call('HMGET', KEYS[1], 'tokens', 'updated_at')
local tokens = tonumber(bucket[1])
local updated_at = tonumber(bucket[2])

if tokens == nil then
  tokens = capacity
end
if updated_at == nil then
  updated_at = now_ms
end

local elapsed_seconds = math.max(0, now_ms - updated_at) / 1000
tokens = math.min(capacity, tokens + elapsed_seconds * refill_rate)

local allowed = 0
if tokens >= requested then
  tokens = tokens - requested
  allowed = 1
end

redis.call('HMSET', KEYS[1], 'tokens', tokens, 'updated_at', now_ms)
redis.call('EXPIRE', KEYS[1], ttl_seconds)

return allowed
`
这单lua脚本还是不能理解，可以加代码详细解读下吗？
```

这个 Lua 已经是：`‌生产级 Redis 令牌桶限流`了。它比：`INCR + EXPIRE 固定窗口`复杂很多。因为它不再是 **简单计数**，而是**`动态计算 token`**，这个最容易绕。

“令牌桶”核心思想：**‌ 请求必须拿到 token 才能执行**,类似：**`‌地铁闸机`**

- **桶里有 token**,例如：`‌桶容量 = 10`,初始：`‌tokens = 10`

- **每次请求**,消耗：`‌1 token`.例如：
	- 第1次请求：`‌10 -> 9`
	- 第2次：`‌9 -> 8`
	- 直到：`‌0`

- **没 token 了？**
	- 直接拒绝：`‌429 Too Many Requests`

- **token 会自动恢复**
	- 例如：`每秒恢复 2 个 token`,这就是：`‌refill_rate`

---

## 这个脚本的配置含义

假设：

```text
capacity = 120
refill_rate = 2
```

意思：**最大桶容量,** `‌最多存120个token`,即：**‌ 最多允许瞬时120次 burst**


**补充速度**：**每秒恢复2个token**,即：`‌每分钟恢复120个token`

---
<br/>

## Redis 中实际存的是什么？

Redis 里：`‌video_upload:rate:user:1001` **对应** `‌ HASH`内容：

```text
tokens      => 57.3
updated_at  => 1747640000000
```

- **字段1：tokens**
	- 当前剩余 token 数量。
	- 注意：**‌ 允许小数**,因为：**token 是连续恢复的**
		- 例如：`‌0.5秒恢复1个`

- **字段2：updated_at**
	- 上次更新时间。
		- 单位：毫秒

---
<br/>

## 脚本整体流程（核心）

- **这个 Lua 干的事：**
	- **第一步**,读取：`当前剩余 token`
	- **第二步**,计算：`‌距离上次过了多久`
	- **第三步**,按时间恢复 token。
	- **第四步**,判断：`‌token够不够`
	- **第五步**
		- 够：`‌扣token,允许请求`
		- 不够：拒绝
	- **第六步**,写回 Redis。

---
<br/>

## 详细拆解

- **1.读取参数**

```lua
local capacity = tonumber(ARGV[1])
```

例如：`120`

<br/>

```lua
local refill_rate = tonumber(ARGV[2])
```

例如：`‌2`, 即：`‌每秒恢复2个token`

<br/>


```lua
local now_ms = tonumber(ARGV[3])
```

当前时间：`‌1747640000000`,毫秒时间戳。

<br/>

```lua
local requested = tonumber(ARGV[4])
```
本次请求需要多少 token。一般：`‌1`,但上传视频可能：`‌5或10`

<br/>

```lua
local ttl_seconds = tonumber(ARGV[5])
```

**Redis key 自动过期时间。** 例如：`‌3600`，**1小时没请求自动删除桶。**

---
<br/>

## 读取 Redis 中的桶

```lua
local bucket = redis.call(
  'HMGET',
  KEYS[1],
  'tokens',
  'updated_at'
)
```

**等价：** `‌HMGET key tokens updated_at`
<br/>

假设 Redis 里：

```text
tokens = 80
updated_at = 100000
```

则：

```lua
bucket[1] = 80
bucket[2] = 100000
```

---
<br/>

## 转数字

```lua
local tokens = tonumber(bucket[1])
local updated_at = tonumber(bucket[2])
```

得到：

```text
tokens = 80
updated_at = 100000
```

---
<br/>

## 第一次请求处理

```lua
if tokens == nil then
  tokens = capacity
end
```

如果 Redis 没数据,说明`‌第一次访问`则`‌tokens = capacity`,即**`‌桶默认是满的`**,例如:`‌120`

<br/>

```lua
if updated_at == nil then
  updated_at = now_ms
end
```

第一次：**更新时间设当前时间。**

---
<br/>

## 最核心部分：恢复 token,这段是灵魂。

```lua
local elapsed_seconds =
  math.max(0, now_ms - updated_at) / 1000
```

意思：`‌距离上次过去了多少秒`.例如：上次 `‌100000ms`,现在`‌105000ms`,则 `‌elapsed_seconds = 5`

---
<br/>

## 计算恢复 token

```lua
tokens = math.min(
  capacity,
  tokens + elapsed_seconds * refill_rate
)
```

这是最关键公式。假设当前：`‌tokens = 80`,过去 `‌5秒`,恢复速度 `‌2 token/秒`,恢复`‌5 * 2 = 10`,新 token`‌80 + 10 = 90`

---
<br/>

## 为什么 math.min？

**避免超过桶容量。**

例如:`‌最多120`,不能`‌150`， 所以：`‌math.min(capacity, xxx)`

---
<br/>

## 判断能否请求

```lua
local allowed = 0
```

默认：**‌拒绝**
<br/>

```lua
if tokens >= requested then
```

如果 token 足够,例如：

```text
tokens = 90
requested = 1
```

则允许。

---
<br/>

## 扣减 token

```lua
tokens = tokens - requested
```

变：

```text
89
```

---
<br/>

## 标记允许

```lua
allowed = 1
```

---
<br/>

## 写回 Redis

```lua
redis.call(
  'HMSET',
  KEYS[1],
  'tokens',
  tokens,
  'updated_at',
  now_ms
)
```

写回：

```text
tokens = 89
updated_at = 当前时间
```

---
<br/>

## 设置过期

```lua
redis.call('EXPIRE', KEYS[1], ttl_seconds)
```

避免：**‌永久垃圾key**,长时间没人访问自动删除。

---
<br/>

## 返回结果

```lua
return allowed
```

返回,允许`‌1`,拒绝`‌0`

---
<br/>

## 完整时间线模拟（最重要）


**配置**

```text
capacity = 10
refill_rate = 2/s
```
<br/>

**初始**

```text
tokens = 10
```

---

- **用户连续请求5次**

**请求1：**

```text
10 -> 9
```
<br/>

**请求2：**

```text
9 -> 8
```
<br/>

**请求5：**

```text
6 -> 5
```
<br/>

**停止3秒**,恢复：

```text
3 * 2 = 6
```

则：

```text
5 -> 11
```

但最大：

```text
capacity=10
```

最终：

```text
10
```

---
<br/>

## 为什么允许 burst？

因为**桶能积攒 token。**,例如：用户10秒没请求,恢复**`‌20 token`**,则**`‌允许瞬间20次请求`**,这叫**‌ Burst Traffic**,非常适合：

* 上传
* API
* 视频
* 秒杀
* AI请求

---
<br/>

## Lua 为什么必须？

因为下面步骤必须原子：

```text
读 token
↓
算恢复
↓
判断
↓
扣减
↓
写回
```

如果拆开,并发会`‌超发token`。Lua 可以：`‌Redis单线程原子执行`

<br/>

这个脚本本质：

```text
根据时间自动恢复 token，
请求来了尝试扣减 token，
扣成功则放行，
否则限流。
```

它是`‌生产环境最经典限流方案之一`,大量用于：

* Nginx
* Gateway
* API网关
* 秒杀
* OpenAI API
* Stripe
* AWS API Gateway
* Kubernetes Ingress
* Envoy

等系统。



















