[**flag标准库**](#flag标准库)


<br/><br/><br/>

***
<br/>

> <h1 id="flag标准库">flag标准库</h1>

`flag.NewFlagSet(...)` 是 Go 标准库 `flag` 包中用于 **创建一组命令行参数集合** 的方法，也可以理解为“一个独立的命令行参数解析器”。


> `flag.NewFlagSet(...)` 就是创建一个新的“命令行参数组”，可以用它来定义、解析自己的命令行参数，而不会影响全局的 `flag`（比如用于子命令、插件等）。

<br/>

**📌 基本使用格式：**

```go
flagSet := flag.NewFlagSet("名称", 错误处理方式)

flagSet.Bool(...)    // 定义布尔参数
flagSet.String(...)  // 定义字符串参数
flagSet.Parse(...)   // 手动解析参数
```

<br/>

**✅ 示例一：创建自定义 flagSet 并解析参数**

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    // 创建一个新的 FlagSet
    fs := flag.NewFlagSet("mycmd", flag.ExitOnError)

    // 添加命令行参数定义
    showVersion := fs.Bool("version", false, "print version")
    configPath := fs.String("config", "", "config file path")

    // 模拟命令行参数：go run main.go -version -config=/tmp/conf.yaml
    args := os.Args[1:] // 忽略程序名

    // 解析参数
    fs.Parse(args)

    // 使用解析后的参数
    // fs.Bool(...) 返回的是一个指向布尔值的 指针（*bool），你需要用 * 取出它所指向的具体布尔值（bool 类型）才能使用。
    if *showVersion {
        fmt.Println("MyApp v1.0")
    }

    if *configPath != "" {
        fmt.Println("Using config:", *configPath)
    }
}
```

运行：

```bash
go run main.go -version -config=/etc/myapp.yaml
```

输出：

```
MyApp v1.0
Using config: /etc/myapp.yaml
```

---
<br/>


**🧩 参数说明**

```go
flag.NewFlagSet("nsqd", flag.ExitOnError)
```

| 参数                 | 含义                                                     |
| ------------------ | ------------------------------------------------------ |
| `"nsqd"`           | 给这个 FlagSet 起个名字（调试、错误信息用）                             |
| `flag.ExitOnError` | 如果解析参数出错就退出程序（可选项还有 `ContinueOnError`, `PanicOnError`） |


