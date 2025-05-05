> <h4/>
- [**简单介绍‌**](#简单介绍)
	- [**‌安装**](#安装)
	- [‌基本命令](#基本命令)
		- [服务端启动](#服务端启动) 
		- [启动redis客户端](#启动redis客户端)
- [**协议**](#协议)
- [**操作Redis**](#操作Redis)
- [**十大数据类型**](#十大数据类型)
	- [Redis流(Stream)](#Redis流(Stream))
- [**Redis的MQ中间件**](#Redis的MQ中间件)  
	- [kafka消息队列](#kafka消息队列)
- [**专业概念详介**](#专业概念详介)
	- [Raft算法白话解释](#Raft算法白话解释)
	- [一致性hash和hash槽](#一致性hash和hash槽)
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
brew install redis
```

<br/>

**验证是否安装成功:**

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

















