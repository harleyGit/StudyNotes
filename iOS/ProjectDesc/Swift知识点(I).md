1. swift 语法糖 ？ ！的本质（实现原理）

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd3.png)

这是底层可选项的代码，可以看出本质是enum

`var age: Int? = 10 `

等价于以下四种：

```
var age0: Optional<Int> = Optional<Int>.some(10)
var age1: Optional = .some(10)
var age2 = Optional.some(10)
var age3 = Optional(10)

```

```
？为optional的语法糖
optional 是一个包含了nil 和普通类型的枚举，确保使用者在变量为nil的情况下处理

！为optional 强制解包的语法糖

```