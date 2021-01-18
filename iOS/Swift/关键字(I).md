

- **Unmanaged**
- **where**
- **intout**
- [**typealias、associatedtype**](https://swifter.tips/typealias/)

- **Mutating**

- **guard**

- **Lazy**

- **internal**

- **associatedtype** 

- **Any和AnyObject**

- **AnyClass**

- **Self 和 discardableResult**

- **derf**

- **@inline(never) 和 @inline(__always)**



<br/>

***
<br/>

># Unmanaged

**`不安全的代码`**

&emsp;  Swift 语言的类都是采用引用计数进行内存管理的。Swift 编译器会在每次对象被访问的时候插入增加引用计数的代码。例如，考虑一个遍历使用类实现的一个链表的例子。遍历链表是通过移动引用到链表的下一个节点来完成的：elem = elem.next，每次移动这个引用，Swift 都要增加 next 对象的引用计数并减少前一个对象的引用计数，这种引用计数代价昂贵但是只要使用 swift 类就无法避免。

```
final class Node {
 var next: Node?
 var data: Int
 ...
}
```
建议：使用未托管的引用避免引用计数的负荷

在效率至上的代码中你可以选择使用未托管的引用。Unmanaged结构体允许开发者对特别的引用关闭引用计数 

```
var Ref : Unmanaged = Unmanaged.passUnretained(Head)
  
while let Next = Ref.takeUnretainedValue().next {
  ...
  Ref = Unmanaged.passUnretained(Next)
}
```


参考资料
[^fn1]
[^fn1]: [Unmanaged](https://nshipster.cn/unmanaged/)




<br/>

***
<br/>


># where

-  **协议使用where**

只有基类实现了当前协议才能添加扩展,也就是多个类实现了同一个协议，根据类名分别为这些类添加扩展。

```

protocol SomeProtocol {
    func someMethod()
}
 
class A: SomeProtocol {
    let a = 1
    
    func someMethod() {
       print("call someMethod")
    }
}
 
class B {
    let a = 2
}
 
//基类A继承了SomeProtocol协议才能添加扩展
extension SomeProtocol where Self: A {
    func showParamA() {
        print(self.a)
    }
}
//反例，不符合where条件
extension SomeProtocol where Self: B {
    func showParamA() {
        print(self.a)
    }
}


let objA = A()
let objB = B()  //类B没实现SomeProtocol， 所有没有协议方法
objA.showParamA()  //输出1

```


<br/>








<br/>

***
<br/>

># intout

**`使用intout注意事项：`**
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


<br/>

**`inout 参数传递过程`**

-  当函数被调用时，参数值被拷贝
-  在函数体内，被拷贝的参数修改
-  函数返回时，被拷贝的参数值被赋值给原有的变量

&emsp;  官方称这个行为为：copy-in copy-out 或 call by value result。我们可以使用 KVO 或计算属性来跟踪这一过程，这里以计算属性为例。排除在调用函数之前与之后的 center GETTER call，从中可以发现：参数值先被获取到（setter 被调用），接着被设值（setter 被调用）。

&emsp;  根据 inout 参数的传递过程，可以得知：inout 参数的本质与引用类型的传参并不是同一回事。inout 参数打破了其生命周期，是一个可变浅拷贝。在 Swift 3.0 中，也彻底摒除了在逃逸闭包（Escape Closure）中被捕获。




<br/>

***
<br/>

># Mutating

- **`使用场景:`**
	-  结构体,枚举类型中声明修饰方法 mutating func funcName()
	-  extension, protocol 修饰 方法

<br/>

- **`注意:`**
-  Swift 中struct,enum 均可以包含类方法和实例方法,Swift官方是不建议在struct,enum 的普通方法内修改属性变量,但是在func 前面添加 mutating 关键字之后就可以在方法内修改；     
-  对于protocol 方法也是适用的,mutating 可以修饰的代理方法,如果struct,enum,class 实现协议之后也可以在其对应的 mutating 代理方法内修改本身的属性变量。(class 不影响,因为属性变量对于类的类方法,实例方法 是透明的,即随时都可以改变)

**`Test 工程中的 CustomController.swift 文件`**

```
import UIKit

@objc
class CustomController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        var ms = MyStruct()
        ms.testMutatingKeyValue(index: 20)
        print("index \(ms.index)")

    }
    

}

extension String {
    mutating func appendString(astring: String){
        self.append("test" + astring)
    }
}

protocol MyProtocol {
    mutating func testMutatingKeyValue(index: Int)
}

struct MyStruct: MyProtocol {
    var  index = 0
    
    mutating func testMutatingKeyValue(index: Int) {
        self.index = index
    }
    
}

```

**`Test 工程中的 ViewController.m 文件`**

```
#import "Test-Swift.h"

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    
    CustomController *cc = [CustomController new];
    [self presentViewController:cc animated:YES completion:nil];
}
```
输出：
`index 20`





<br/>

***
<br/>

># guard

 [guard关键字](https://www.cnblogs.com/edensyd/p/9566979.html)



<br/>

***
<br/>

># Lazy 的使用

[lazy 简单的使用](https://www.jianshu.com/p/aa6707791ab6)




<br/>

***
<br/>

># 访问权限关键字 使用

![关键字权限图](https://upload-images.jianshu.io/upload_images/2959789-c62481c8db74300b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[访问控制 private和fileprivate的区别](https://www.jianshu.com/p/2a9a94d4fe34)
[private、filePrivate、open、public](https://blog.csdn.net/Mazy_ma/article/details/70135990)




<br/>

***
<br/>

># internal(set) 的使用
[访问控制 US](https://www.cnblogs.com/xjf125/p/10088307.html)










<br/>

***
<br/>

># associatedtype协议关联类型

&emsp;  定义一个协议时，有的时候声明一个或多个关联类型作为协议定义的一部分将会非常有用。关联类型为协议中的某个类型（任意类型）提供了一个占位名（或者说别名），其代表的实际类型在协议被采纳时才会被指定。你可以通过 associatedtype 关键字来指定关联类型，当然你也可以用来设计api用来构建统一的处理结构。比如使用协议声明更新cell的方法。


```
class  detail {
          //some properties
}
class  briefdetail {
          //some properties
}
class  skudetail {
          //some properties
}

protocol DTCellItemPro {
    associatedtype  T
   func updateCell(_ data: T)
}

class MyDTTableViewCell: UITableViewCell, DTCellItemPro{
    ///在类型不确定的情况下，需要声明一个或多个关联类型。
   ///相当于一个占位名。作为关联类型在协议被实现之前是不需要指定的。关键字：typealias
    typealias T = detail
    func updateCell(_ data: detail) {
             
    }
}
class MySKUTableViewCell: UITableViewCell, DTCellItemPro{
    typealias T = skudetail
    func updateCell(_ data: skudetail) {
             
    }
}
```


<br/>

***
<br/>

>#  Any 和 AnyObject

* AnyObject 可以代表任何 class 类型的实例*
* Any 可以表示任意类型，甚至包括方法（func）类型*

概念不多做介绍了，可以参考：[王巍 Any和AnyObject](http://swifter.tips/any-anyobject/)

<br/>

* 类型判断和应用

```
class Boy {
}

class Girl {
}

func enter(_ child: AnyObject) {
    // 方法1
    switch child {
    case let boy as Boy:
        print("boy")
    case let girl as Girl:
        print("girl")
    default:
        break
    }
    
    // 方法2
    if child is Boy {
        print("boy")
    }
    if child is Girl {
        print("girl")
    }
}

enter(Boy())
```



<br/>

***
<br/>

>#  AnyClass
 
-  AnyObject.Type
&emsp;  `AnyObject.Type` 这种方式得到的就是一个元类型`（Meta）`，也就是 `AnyClass`。在 Swift 中被一个 typealias 定义：

```
typealias AnyClass = AnyObject.Type
```

<br/>

`.self `用在类型后面取得类型本身，用在实例后面取得实例本身:
`Meta.Type `代表 `Meta` 这个类型的类型，也就是一个用来存储` Meta `类型的元类型。然后用  `.self `从 `Meta` 中取出其类型

```
class Meta {
    var title = ""
    required init(name: String) {
        self.title = name
    }
    
    func method1() {
        print(title)
    }
    
    class func method2() {
        print("class method")
    }
}

let typeMeta : Meta.Type = Meta.self
```

<br/>

- 类方法调用
- 
&emsp;  可以通过元类型直接调用 类方法，上面代码中 Meta 中声明了一个实例方法(method1)和一个类方法(method2)

```
let typeMeta1 : Meta.Type = Meta.self
typeMeta1.method2()

let typeMeta2: AnyClass = Meta.self
(typeMeta2 as! Meta.Type).method2()
```

<br/>

- 实例方法调用

&emsp;  实例方法的调用要先声明实例，直接通过 init 方法获取实例固然可以，但是我们这里介绍如果使用元类型获取实例，甚至调用实例方法

```
let typeMeta : Meta.Type = Meta.self
typeMeta.method1(Meta(name: "new meta"))()
// print: new meta
```

&emsp;  上面代码中` typeMeta` 是 `Meta.Type `类型，通过它调用实例方法 method1需要传入一个实例变量，然后系统会返回这个实例方法本身。如果上面第二行代码不好理解，看这里 :

```
let f = typeMeta.method1(Meta(name: "new meta"))
f()
```
或者
```
let meta = typeMeta.init(name: "type init")
meta.method1()
```

&emsp;  调用实例方法为什么不能直接创建实例再调用呢？例如：Meta().method1()。确实，对于一个独立的类型来说我们完全没有必要关心它的元类型，但是元类型或者元编程的概念的理解可以让你在设计框架时避免不断改动代码。下面这段代码便可看出元类型的优势：

```
class vc1: UIViewController {
}
class vc2: UIViewController {
}

func setupViewControllers(_ vcTypes: [AnyClass]) {
    for vctype in vcTypes {
        if vctype is UIViewController.Type {
            let vc = vctype.init()
            print(vc)
        }
    }
}

let usingvcTypes: [AnyClass] = [vc1.self, vc2.self]
setupViewControllers(usingvcTypes)
```

&emsp; 类似这样的用法，就可以避免你在开发中不断修改代码，也迫使你在封装代码时不断严格要求自己。



<br/>

***
<br/>

># Self 和 discardableResult

```
class CustomClass {
    var a: Int
    var b: Int
    
    init(a: Int, b: Int) {
        self.a = a
        self.b = b
    }
    
    @discardableResult
    func setValueA(parameter: Int) -> Self {
        self.a = parameter
        
        return self
    }
    
    deinit {
        print("CustomClass 释放")    
    }
    
}


//调用
let obj = CustomClass(a: 1, b: 2)
//有了这个discardableResult关键字，可以没有警告⚠️了
obj.setValueA(parameter: 3)
print("objA a: \(obj.a)")
```




<br/>

***
<br/>

># derf

[derf 的使用场景](https://www.jianshu.com/p/478cbb5cf8a1)



<br/>

***
<br/>

># @inline(never) 和 @inline(__always)

- @inline(never) // 声明这个函数never编译成inline的形式

- @inline(__always) // 声明这个函数总是编译成inline的形式

inline函数可以用 闭包 的形式实现




<br/>

***
<br/>

参考资料：[^fn2]
[^fn2]:[Swift 关键字集锦](https://www.cnblogs.com/liYongJun0526/p/7522130.html)
