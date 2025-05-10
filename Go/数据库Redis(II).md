> <h1 id=""></h1>
- [<font color='red'>**过滤器** </font>](#过滤器)
	- [布隆过滤器](#布隆过滤器)
	- [布谷鸟过滤器](#布谷鸟过滤器)
- [<font color='red'>**Lua脚本** </font>](#Lua脚本)
	- [Lua脚本简单示例](#Lua脚本简单示例)

<br/><br/><br/>

***
<br/>

> <h1 id="过滤器">过滤器</h1>


***
<br/><br/><br/>

> <h2 id="布隆过滤器">布隆过滤器</h2>

布隆过滤器使用Redis的bitmap对查询的数据查看是否存在,其主要通过hashmap进行存储布尔值.解决了过高对redis、mysql具体数据查询的压力,可以解决**缓存击穿**的问题.



***
<br/><br/><br/>

> <h2 id="布谷鸟过滤器">布谷鸟过滤器</h2>
**布谷鸟过滤器**解决了布隆过滤器中的因为`hash`冲突导致的一个坑位存储了多个值不能 删除的问题.


<br/><br/><br/>

***
<br/>

> <h1 id="Lua脚本">Lua脚本</h1>
Lua脚本可以使多个**Redis**命令合成一个命令,具有原子性!避免了非原子操作.

***
<br/><br/><br/>
> <h2 id="Lua脚本简单示例">Lua脚本简单示例</h2>
在 Go 中嵌入 Lua 脚本（用于操作 Redis），可以通过以下方式实现：

1. 使用 [gopher-lua](https://github.com/yuin/gopher-lua) 来运行 Lua 脚本。
2. 在 Lua 中通过 Redis 的客户端访问 Redis（例如使用 `redis.call` 形式的模拟）。
3. 用 Go 封装 Redis 操作，在 Lua 脚本中调用这些函数。

---

### 示例场景：

你想在 Lua 脚本中写逻辑如：

```lua
redis_set("mykey", "myvalue")
value = redis_get("mykey")
print(value)
```

---

### 步骤一：安装依赖

```bash
go get github.com/yuin/gopher-lua
go get github.com/go-redis/redis/v8
```

---

### 步骤二：Go 代码实现

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-redis/redis/v8"
	lua "github.com/yuin/gopher-lua"
)

var ctx = context.Background()

// 初始化 Redis 客户端
var rdb = redis.NewClient(&redis.Options{
	Addr: "localhost:6379", // 修改为你的 Redis 地址
})

// redis_set(key, value)
func luaRedisSet(L *lua.LState) int {
	key := L.ToString(1)
	value := L.ToString(2)

	err := rdb.Set(ctx, key, value, 0).Err()
	if err != nil {
		L.RaiseError("Redis SET error: %v", err)
	}
	return 0 // no return value
}

// redis_get(key)
func luaRedisGet(L *lua.LState) int {
	key := L.ToString(1)
	val, err := rdb.Get(ctx, key).Result()
	if err == redis.Nil {
		L.Push(lua.LNil)
	} else if err != nil {
		L.RaiseError("Redis GET error: %v", err)
	} else {
		L.Push(lua.LString(val))
	}
	return 1 // one return value
}

func main() {
	L := lua.NewState()
	defer L.Close()

	// 注册 Go 函数到 Lua
	L.SetGlobal("redis_set", L.NewFunction(luaRedisSet))
	L.SetGlobal("redis_get", L.NewFunction(luaRedisGet))

	// 示例 Lua 脚本
	script := `
        redis_set("foo", "bar")
        val = redis_get("foo")
        print("Lua got from Redis:", val)
    `

	if err := L.DoString(script); err != nil {
		fmt.Println("Lua error:", err)
	}
}
```

---

### 输出示例：

```bash
Lua got from Redis: bar
```

---

这个实现允许你用 Lua 控制逻辑，用 Go 控制 Redis 的访问。

你是希望 Lua 脚本完全使用 Redis 的原生语法（如 `redis.call()`），还是更偏向用类似 `redis_get()` 这种形式？


