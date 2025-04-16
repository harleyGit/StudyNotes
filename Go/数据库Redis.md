> <h4/>
- [**简单介绍‌**](#简单介绍)
	- [**‌安装**](#安装)
	- [‌基本命令](#基本命令)
- [**协议**](#协议)
- [**操作Redis**](#操作Redis)
- [**十大数据类型**](#十大数据类型)
	- [Redis流(Stream)](#Redis流(Stream))
- [**Redis的MQ中间件**](#Redis的MQ中间件)  
	- [kafka消息队列](#kafka消息队列)
- **资料**
	- [Redis资料-尚硅谷](https://github.com/harleyGit/Learning-in-practice/tree/master/Redis)
	- [Redis在线测试](https://try.redis.io/)
	- [go-redis使用入门](https://juejin.cn/post/7202521955366879288)


<br/><br/><br/>

***
<br/>

> <h1 id="简单介绍">简单介绍</h1>

Redis 的客户端和服务端是通过 TCP 连接来进行数据交互， 服务器默认的端口号为 6379

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


***
<br/><br/><br/>

># <h2 id="基本命令"> [基本命令](https://www.redis.net.cn/order/) </h2>
[**Redis命令参考**](http://doc.redisfans.com/index.html)

<br/>

**启动服务器命令:**

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
```

<br/>

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
```

<br/>

**查看redis-server的PID**

```sh
ps xua | grep redis
```

<br/>

**启动redis客户端**

```sh
redis-cli -h 127.0.0.1 -p 6379

// 单实例关闭: redis-cli -a 密码 shutdown
redis-cli -a 123456 shutdown，如果在Redis服务器里面可以直接使用shutdown命令
shutdown // 停止redis服务 
quit // 退出redis-cli 的交互模式


// 多实例关闭,指定端口关闭: redis-cli -p 端口 shutdown
redis-cli -p 6379 shutdown
```

<br/>

**常用语法**

```
keys * // 查询所有key
get [key] // 获取一条
flushall // 删除所有
set [key] [value] // 设置一条
```


<br/><br/><br/>

***
<br/>

> <h1 id="协议">协议</h1>


**请求协议**
格式示例

```
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

```
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

```
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














