


```
class SwiftClass {
    var name: String?
    var height = 0.0
    var width = 0.0
    
    var description: String {
        return "ResolutionClass(height: \(height), width: \(width))"
    }
    
    func printString(alert: String) -> Void {
        print("\(alert)")
    }
}

struct SwiftStruct {
    var height = 0.0
    var width = 0.0
}
```

<br/>
#`引用类型使用intout参数，意义不大`
```
func swap(clss: inout SwiftClass) {
    //打印引用类型变量指向的内存地址
    print("During calling: \(Unmanaged.passUnretained(clss).toOpaque())")
    let temp = clss.height
    clss.height = clss.width
    clss.width = temp
}


sc.height = 1080
sc.width = 1920
print(sc)
print("Before calling: \(Unmanaged.passUnretained(sc).toOpaque())")
swap(clss: &sc)
print(sc)
print("After calling: \(Unmanaged.passUnretained(sc).toOpaque())")
```
打印：
`SwiftTest.SwiftClass`
`Before calling: 0x000000010284b5d0`
`During calling: 0x000000010284b5d0`
`SwiftTest.SwiftClass`
`After calling: 0x000000010284b5d0`


#`使用intout注意事项：`
-  使用 inout 关键字的函数，在调用时需要在该参数前加上 & 符号;
-  inout 参数在传入时必须为变量，不能为常量或字面量（literal）;
```
//常量使用关键字 let 来声明
格式：let constantName = <initial value>
如：let constA = 42

//字面量：就是指能够直接了当地指出自己的类型并为变量进行赋值的值，与常量无异。
//字符串型字面常量
let name = "DevZhang"

```
-  inout 参数不能有默认值，不能为可变参数
```
//可变参数，有多个参数用省略号表示
func add(a:Int, b:Int ,others:Int ...) -> Int {
var result = a + b
for num in others {
    result += num
}
    return result
}

let number = add(2, b: 5, others: 2, 50, 4)
print(number)  //63
```
-  inout 参数不等同于函数返回值，是一种使参数的作用域超出函数体的方式
-  多个 inout 参数不能同时传入同一个变量，因为拷入拷出的顺序不定，那么最终值也不能确定



<br/>
```
struct Point {
    var x = 0.0
    var y = 0.0
}

struct Rectangle {
    var width = 0.0
    var height = 0.0
    var origin = Point()
    
    var center: Point {
        get {
            print("center GETTER call")
            return Point(x: origin.x + width / 2,
                         y: origin.y + height / 2)
        }
        
        set {
            print("center SETTER call")
            origin.x = newValue.x - width / 2
            origin.y = newValue.y - height / 2
        }
    }
    
    func reset(center: inout Point) {
        center.x = 0.0
        center.y = 0.0
    }
    
}

var rect = Rectangle(width: 100, height: 100, origin: Point(x: -100, y: -100))
print("rect.center 值：\(rect.center)\n")
rect.reset(center: &rect.center)
print("rect.center 重置后的值：\(rect.center)")

```
打印：
`center GETTER call`
`rect.center 值：Point(x: -50.0, y: -50.0)`

`center GETTER call`
`center SETTER call`
`center GETTER call`
`rect.center 重置后的值：Point(x: 0.0, y: 0.0)`

#`inout 参数传递过程`
-  当函数被调用时，参数值被拷贝
-  在函数体内，被拷贝的参数修改
-  函数返回时，被拷贝的参数值被赋值给原有的变量

&emsp;  官方称这个行为为：copy-in copy-out 或 call by value result。我们可以使用 KVO 或计算属性来跟踪这一过程，这里以计算属性为例。排除在调用函数之前与之后的 center GETTER call，从中可以发现：参数值先被获取到（setter 被调用），接着被设值（setter 被调用）。

&emsp;  根据 inout 参数的传递过程，可以得知：inout 参数的本质与引用类型的传参并不是同一回事。inout 参数打破了其生命周期，是一个可变浅拷贝。在 Swift 3.0 中，也彻底摒除了在逃逸闭包（Escape Closure）中被捕获。


<br/>
***
<br/>



># 嵌套类型
#`值类型嵌套值类型`


<br/>

#`值类型嵌套引用类型`


<br/>

#`引用类型嵌套值类型`


<br/>

#``


<br/>
***
<br/>








<br/>
***
<br/>







<br/>
***
<br/>







<br/>
***
<br/>







<br/>
***
<br/>








<br/>
参考资料：[^fn1]
[^fn1]:[swift的值类型和引用类型](https://www.cnblogs.com/luoxiaofu/p/8528383.html)
