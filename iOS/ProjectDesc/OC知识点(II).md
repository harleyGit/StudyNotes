- [**OC知识点(I)**](./OC知识点(I).md)


<br/>

***
<br/>


> <h2 id=""></h2>
- [**Swfit开源代码**](https://github.com/apple)
- [**Swift基础**](#Swift基础)
	- [道长基础面试题](https://www.jianshu.com/p/07c9c6464f83) 
	- [Swift的缺省基类](#Swift的缺省基类)
	- [Swift和OC区别](#Swift和OC区别)
	- [Swift和OC混编注意事项](#Swift和OC混编注意事项)
	- [面向协议编程](#面向协议编程)
	- [协议中实用泛型](#协议中实用泛型)
	- [路由导航](#路由导航)
	- [Any、AnyObject、AnyClass的区别](#AnyAnyObjectAnyClass的区别)
	- [逃逸闭包怎么使用，它的关键字@escaping什么时候使用](#逃逸闭包使用)
	- [什么是写时复制(copy-on-write)](#什么是写时复制(copy-on-write))
	- [值类型和引用类型区别](#值类型和引用类型区别)
	- [关键字](#关键字)
		- [final修饰符](#final修饰符)
		- [@Objc和Dynamic的使用(Storehub)](#@Objc和Dynamic的使用)
		- [Swfit中的@Objc和dynamic的原理(Versh面试)](#Swfit中的@Objc和dynamic的原理)
	- [语法基础](#语法基础)
		- [只能被类遵守的protocol](#只能被类遵守的protocol)
		- [defer 使用场景](#defer使用场景)
		- [数组索引越界会Crash,字典取不到为nil](#数组索引越界会Crash,字典取不到为nil)
		- [Self的使用场景](#Self的使用场景)
		- [throws和rethrows的用法与作用](#throws和rethrows的用法与作用)
		- [值类型和引用类型在什么时候使用](#值类型和引用类型在什么时候使用)
		- [协议的动态](#协议的动态)
		- [Swift和OC常量区别](#Swift和OC常量区别)
		- [autoclosure的作用](#autoclosure的作用)
		- [编译选择whole module optmization优化了什么](#编译选择wholemoduleoptmization优化了什么)
		- [mutaing 的作用](#mutaing的作用)
		- [怎么表示函数的参数类型只要是数字](#怎么表示函数的参数类型只要是数字)
		- [dynamic的作用.](#dynamic的作用)
		- [Swift的动态性](#Swift的动态性)
- [**常用类库**](#常用类库)
	- [ObjectMapper](#ObjectMapper)
	- [SwiftJSON](#SwiftJSON)
	- [RxSwift](#RxSwift)
	- [Snapkit](#Snapkit)
- [**数据存储**](#数据存储)
	- [CoreData](#CoreData) 
- [**组件化**](#组件化)
	- [路由导航](#路由导航)
- [**安全**](#安全)
	- [MD5](#MD5)
	- [防止反编译](#防止反编译)
- [**多线程**](#多线程)
	**‌**- [NSOperation](#NSOperation)
	- [GCD控制线程数量](#GCD控制线程数量) 
	- [线程锁](#线程锁)
	- [文件读写锁](#文件读写锁)
	- [文件分片下载](https://benarvintec.com/2018/10/10/iOS分片下载器)
- [**底层**](#底层)
	-  [Swift动态性 ](#Swift动态性)
- [**前端**](#前端)



<br/>

***
<br/>





> <h1 id = "Swift基础">Swift基础</h1>



<br/>

>## <h2 id = "Swift的缺省基类">[Swift的缺省基类](https://struggleblog.com/2020/06/22/swift的缺省基类/)</h2>

<br/>

> <h2 id = "Swift和OC区别">Swift和OC区别</h2>

- swift是静态类型语言，OC是动态类型语言
	- 静态概念：类型判断是在运行前做的（如编译阶段） 
	- 静态表现：使用变量前需要声明变量。
	- 动态概念：意思就是类型的检查是在运行时做的；
	- 动态表现：使用变量前不需要声明变量。
- swift是类型安全的语言、注重安全（解释为：是因为它清楚的知道代码在编译阶段处理值的类型**‌**），OC注重灵活；
- swift注重面向协议编程、函数式编程、面向对象编程，OC注重面向对象编程；


<br/>
<br/>

> <h2 id = "Swift和OC混编注意事项">Swift和OC混编注意事项</h2>

- **导入文件注意事项：**
	- 在OC中创建swift文件，会弹出是否需要创建桥接文件项目名称-Bridging-Header.h，点击创建，在swift中调用OC类，只需要把OC类的头文件import到桥接文件中即可
	
	- 在OC中调用swift文件，需要导入隐式头文件：工程项目名-swift.h，xxx-Swift.h在项目中是看不到的，但是确实是可以import的
	
	- 在Swift代码中将需要暴露给OC调用的属性和方法前加上 @objc修饰符。

<br/>

- **使用第三方Framework相关配置：**
	- 设置： target-->build setting -->Packaging -->Defines Module为 “Yes”；

	- 然后，配置文件Target -> Build Phases  -> Link Binary，添加要导入的Framework；
	
	- 最后，还是要配置桥接文件，比如要使用 abc-lib.framework库中的 abc.h 就要这样配置：#import"abc-lib/abc.h";


<br/>

- **Subclass子类问题**
	- 对于自定义的类而言，Objective-C的类，不能继承自Swift的类，即要混编的OC类不能是Swift类的子类。反过来，需要混编的Swift类可以继承自OC的类。 
	- 提问：为什呢OC不能继承字Swift的子类？
		- 答：这是因为派发机制不同造成的,在原生声明（非 extension）中定义的普通方法和标记为 @objc 的方法都使用 V-Table 机制派发。用 Swift 编写的类是不能被 Objective-C 继承的，@objc 只是把方法暴露给 Objective-C，并没有改变方法派发的本质。


<br/>

- **宏文件**
	- 如Swift文件要使用OC中定义的宏，只能使用常量简单宏文件。

<br/>

- **Swift独有特性**
	- Swift中有许多OC没有的特性，比如，Swift有元组、为一等公民的函数、还有特有的枚举类型。所以，要使用的混编文件要注意Swift独有属性问题。



<br/>

- **Swift中使用OC的block**

Swift中使用Closure不能使用Block作为属性进行传值，必须是初始化方法或函数。

**Objective-C文件中：**

```
#import typedef void (^Myblock)(NSString *arg);

@interface FirViewController : UIViewController

//@property (copy, nonatomic) Myblock myBlock;

//这种作为公共参数的形式，如果在Swift类中去回调的话，是有问题的。提示没有初始化方法，所以使用下面的以Block为参数的方法

- (void)transValue:(Myblock) block;

@end
```

下面是.m文件

```
#import "FirViewController.h"

@implementation FirViewController

- (void)viewDidLoad{

	[super viewDidLoad];
	
	self.view.backgroundColor = [UIColor whiteColor];

}

- (void)transValue:(Myblock)block {
	
	if (block) {
		block(@"firBack");
	}
}

@end
```

在Swift文件回调：

在Swift使用OC的类时，首先在桥接文件中声明oc的头文件:**工程名-Bridging-Header.h**这是创建Swift工程的情况下

```
import UIKit

class ViewController: UIViewController {

override func viewDidLoad() {

	super.viewDidLoad()
	
	self.view.backgroundColor = UIColor.whiteColor()

}

@IBOutlet weak var goFirst: UIButton!

@IBAction func goFirstAction(sender: AnyObject) {

	let firVC:FirViewController = FirViewController()
	
	firVC. transValue { ( arg:String !) -> Void in
	
	self.aBtn?.setTitle(arg, forState: UIControlState.Normal)

}

self.navigationController?.pushViewController(firVC, animated: true)

}

```


<br/>
<br/>

> <h2 id = "面向协议编程">面向协议编程</h2>

- **OOP**：面向对象编程（英文Object Oriented Programming）；
- **POP**：面向协议编程（Protocol Oriented Programming），是Swift的一种编程范式；


&emsp; 在Swift的协议中定义属性永远不要用 `let` 关键字。只读属性规定使用 `var` 关键字，并在后面单独跟上 `{ get }`。如果有一个方法改变了一个或多个属性，你需要标记它为 `mutating`。


<br/>


在[面向协议编程与 Cocoa 的邂逅 (上)](https://onevcat.com/2016/11/pop-cocoa-1/)的文章里POP解决了面向对象编程的**OC动态派发安全性、横切关注点、菱形缺陷**问题。

协议扩展：
对于 P，可以在 extension P 中为 myMethod 添加一个实现：


```
protocol P {
    func myMethod()
}

extension P {
    func myMethod() {
        doWork()
    }
}
```


有了这个协议扩展后，我们只需要简单地声明 ViewController 和 AnotherViewController 遵守 P，就可以直接使用 myMethod 的实现了：

```
extension ViewController: P { }
extension AnotherViewController: P { }

viewController.myMethod()
anotherViewController.myMethod()
```
不仅如此，除了已经定义过的方法，我们甚至可以在扩展中添加协议里没有定义过的方法。在这些额外的方法中，我们可以依赖协议定义过的方法进行操作。我们之后会看到更多的例子。总结下来：

- **协议定义**
	- 提供实现的入口
	- 遵循协议的类型需要对其进行实现
- **协议扩展**
	- 为入口提供默认实现
	- 根据入口提供额外实现


这样一来，横切点关注的问题也简单安全地得到了解决。
- ✅ 动态派发安全性
- ✅ 横切关注点
- 菱形缺陷


<br/>

在[**面向协议编程与 Cocoa 的邂逅 (下)**](https://onevcat.com/2016/12/pop-cocoa-2/)中我们把POP运用到实际的项目中去，比如：网络的封装。很赞的，可以把其运用到项目中去，重构项目中的网络层。



<br/>
<br/>

> <h2 id = "协议中实用泛型">协议中实用泛型</h2>
[protocol使用associatedType和泛型](https://blog.csdn.net/boildoctor/article/details/113116245)

- 协议中使用关联类型代替泛型
	- 协议不允许使用泛型参数,想要协议使用泛型,请使用关联类型代替；
	- 只是把protocol<泛型类型> 改成了在{添加 associatedtype 泛型类型}

```
protocol Stackble {   //定义一个栈的协议
    associatedtype Element// 在协议中用来占位的类型是关联类型,声明一个关联类型Element
    mutating func push(_ element:Element)
    mutating func pop()->Element
    func top() ->Element
    func size() ->Int
}

```

- 带泛型的class中,泛型类型填充关联类型

```
class Stack <E>: Stackble {
   
    var elements = [E]() //用关联类型
     func push(_ element:E){  //这里因为实现push 的时候给element输入了E,所以就相当于给Element 起了别名E
        //等于typealias E = Element
        
        elements.append(element)
    }
     func pop()->E{
        elements.removeLast()
    }
    func top() ->E{
        elements.last!
    }
    func size() ->Int{
        elements.count
    }
}


///代码调用
var s1 = Stack<Int>()
s1.push(1)
s1.push(2)
s1.push(3)
print("s1.top()" ,s1.top())

```

打印结果：`s1.top() 3`


<br/>

- **解决让class遵循带关联类型的协议,并且能当做形参和返回值的方法**

**方法一：让泛型遵循协议,然后让泛型当做形参或返回值,代码如下**

```
protocol Runnable {
    associatedtype Speed
    var speed : Speed {get}
    
} //协议中有关联类型associatedtype,类型不确定

class Person:Runnable{
    var speed: Double = 0.0
}
class Car:Runnable{
    var speed: Double = 0.0
}


func get<T:Runnable>(_ type:Int)->T{ //让泛型类型T遵守协议,然后返回T
    if  0 == type{
        
        //        let result = Person() as T //编译错误,系统认为 Person() 创建的结果 强转成T 可能失败,所以要用as!强转,因为可能失败,可能返回nil,所以是可选类型,要用as !
        let result = Person() as! T
        return result
    }
    return Car() as! T
}
```


<br/>

**方法二：不明确类型**

some让协议的关联类型变成透明的, 在协议前面标记上 some 后，返回值的类型对编译器就变成透明的了。在这个值使用的时候编译器可以根据返回值进行类型推断得到具体类型。如果不加some 编译报错,会认为返回的是个关联类型,是不确定的类型

```
@available(OSX 10.15.0, *)//要求系统超过10.15,编译提示自动添加
func get2(_ type:Int )-> some Runnable{ //some让协议的关联类型变成透明的, 在协议前面标记上 some 后，返回值的类型对编译器就变成透明的了。在这个值使用的时候编译器可以根据返回值进行类型推断得到具体类型。如果不加some 编译报错,会认为返回的是个关联类型,是不确定的类型
    return Car() //some只能返回一种类型
```

下面代码是错误的,因为some不能返回2种类型:

```
 func get3(_ type:Int )-> some Runnable{ //编译错误,some限制的类型不能返回2种类型
 if  0 == type{
 return Person()
 }
 return Car() //some只能返回一种类型
 }
```




<br/>
<br/>


> <h2 id="AnyAnyObjectAnyClass的区别">Any、AnyObject、AnyClass的区别</h2>

- Any: 可以表示任意类型，甚至方法类型（func）
- AnyObject: 表示任何class类型的实例对象（类似OC中的id类型）
- AnyClass：表示任意类的元类型.任意类的类型都隐式遵守这个协议.  AnyObject.Type中的.Type就是获取元类型, 辟如你有一个Student类, Student.Type就是获取Student的元类型.

```
// AnyObject的定义：
protocol AnyObject {

}
```

&emsp; 特别之处在于，所有的 class 都隐式地实现了这个接口，这也是 AnyObject 只适用于 class 类型的原因。而在 Swift 中所有的基本类型，包括 Array 和 Dictionary 这些传统意义上会是 class 的东西，统统都是 struct 类型，并不能由 AnyObject 来表示，于是 Apple 提出了一个更为特殊的 Any，除了 class 以外，它还可以表示包括 struct 和 enum 在内的所有类型。
为了深入理解，举个很有意思的例子。为了实验 Any 和 AnyObject 的特性，在 Playground 里写如下代码：

```
import UIKit

let swiftInt: Int = 1
let swiftString: String = "miao"

var array: [AnyObject] = []
array.append(swiftInt)
array.append(swiftString)
```

&emsp; 我们在这里声明了一个 Int 和一个 String，按理说它们都应该只能被 Any 代表，而不能被 AnyObject 代表的。但是你会发现这段代码是可以编译运行通过的。那是不是说其实 Apple 的编程指南出错了呢？不是这样的，你可以打印一下 array，就会发现里面的元素其实已经变成了 NSNumber 和 NSString 了，这里发生了一个自动的转换。因为我们 import 了 UIKit (其实这里我们需要的只是 Foundation，而在导入 UIKit 的时候也会同时将 Foundation 导入)，在 Swift 和 Cocoa 中的这几个对应的类型是可以进行自动转换的。因为我们显式地声明了需要 AnyObject，编译器认为我们需要的的是 Cocoa 类型而非原生类型，而帮我们进行了自动的转换。

&emsp; 在上面的代码中如果我们把 import UIKit 去掉的话，就会得到无法适配 AnyObject 的编译错误了。我们需要做的是将声明 array 时的 [AnyObject] 换成 [Any]，就一切正确了。

```
let swiftInt: Int = 1
let swiftString: String = "miao"

var array: [Any] = []
array.append(swiftInt)
array.append(swiftString)
array

```

&emsp; 顺便值得一提的是，只使用 Swift 类型而不转为 Cocoa 类型，对性能的提升是有所帮助的，所以我们应该尽可能地使用原生的类型。


<br/>
<br/>

> <h2 id="逃逸闭包使用">逃逸闭包怎么使用，它的关键字@escaping什么时候使用</h2>

&emsp; **逃逸闭包概念：**一个接受闭包作为参数的函数，该闭包可能在函数返回后才被调用，也就是说这个闭包逃离了函数的作用域，这种闭包称为逃逸闭包。当你声明一个接受闭包作为形式参数的函数时，你可以在形式参数前写@escaping来明确闭包是允许逃逸。

&emsp; 例如：当网络请求结束后调用的闭包。发起请求后过了一段时间后这个闭包才执行，并不一定是在函数作用域内执行的。


```
class ViewController: UIViewController {
   
    override func viewDidLoad() {
        super.viewDidLoad()
        getData { (data) in
            print("闭包结果返回--\(data)--\(Thread.current)")
        }
    }

    func getData(closure:@escaping (Any) -> Void) {
        print("函数开始执行--\(Thread.current)")
        DispatchQueue.global().async {
            DispatchQueue.main.asyncAfter(deadline: DispatchTime.now()+2, execute: {
                print("执行了闭包---\(Thread.current)")
                closure("345")
            })
        }
        print("函数执行结束---\(Thread.current)")
    }
}


```

打印结果：

```

函数开始执行--<NSThread: 0x600000072f40>{number = 1, name = main}
函数执行结束---<NSThread: 0x600000072f40>{number = 1, name = main}
执行了闭包---<NSThread: 0x600000072f40>{number = 1, name = main}
闭包结果返回--345--<NSThread: 0x600000072f40>{number = 1, name = main}

```

从结果可以看出，逃逸闭包的生命周期是长于函数的。

- 逃逸闭包的生命周期：

	- 闭包作为参数传递给函数；
	
	- 退出函数；
	
	- 闭包被调用，闭包生命周期结束。

即逃逸闭包的生命周期长于函数，函数退出的时候，逃逸闭包的引用仍被其他对象持有，不会在函数结束时释放。


<br/>


**非逃逸闭包**

&emsp; **非逃逸闭包概念：**一个接受闭包作为参数的函数， 闭包是在这个函数结束前内被调用。
例如：

```
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        handleData { (data) in
            print("闭包结果返回--\(data)--\(Thread.current)")
        }
    }

    func handleData(closure:(Any) -> Void) {
        print("函数开始执行--\(Thread.current)")
        print("执行了闭包---\(Thread.current)")
        closure("4456")
        print("函数执行结束---\(Thread.current)")
    }
}
```

复制代码以上代码最后的执行结果为：

```
函数开始执行--<NSThread: 0x6000000fe8c0>{number = 1, name = main}
执行了闭包---<NSThread: 0x6000000fe8c0>{number = 1, name = main}
闭包结果返回--4456--<NSThread: 0x6000000fe8c0>{number = 1, name = main}
函数执行结束---<NSThread: 0x6000000fe8c0>{number = 1, name = main}
```

&emsp; 从结果可以看出，非逃逸闭包被限制在函数内。

非逃逸闭包的生命周期：
1、闭包作为参数传给函数；
2、函数中运行改闭包；
3、退出函数。

**为什么要分逃逸闭包和非逃逸闭包?**

&emsp; 为了管理内存，闭包会强引用它捕获的所有对象，比如你在闭包中访问了当前控制器的属性、函数，编译器会要求你在闭包中显示 self 的引用，这样闭包会持有当前对象，容易导致循环引用。
而对于非逃逸闭包：

&emsp; 非逃逸闭包不会产生循环引用，它会在函数作用域内释放，编译器可以保证在函数结束时闭包会释放它捕获的所有对象。

&emsp; 使用非逃逸闭包可以使编译器应用更多强有力的性能优化，例如，当明确了一个闭包的生命周期的话，就可以省去一些保留（retain）和释放（release）的调用。

&emsp; 非逃逸闭包它的上下文的内存可以保存在栈上而不是堆上。



<br/>
<br/>

> <h2 id="什么是写时复制(copy-on-write)">什么是写时复制(copy-on-write)</h2>

&emsp; **有两种传值方式：** 引用类型(Class)和值类型(Struct/Enum)。而值类型有一个copy的操作，它的意思是当你传递一个值类型的变量的时候(给一个变量赋值，或者函数中的参数传值)，它会拷贝一份新的值让你进行传递。你会得到拥有相同内容的两个变量，分别指向两块内存。

&emsp; **问题：** 这样的话，在你频繁操作占用内存比较大的变量的时候就会带来严重的性能问题，Swift 也意识到了这个问题，所以推出了 Copy-on-Write 机制，用来提升性能。


**什么是 Copy-on-Write？**

&emsp; 当你有一个占用内存很大的一个值类型，并且不得不将它赋值给另一个变量或者当做函数的参数传递的时候，拷贝它的值是一个非常消耗内存的操作，因为你不得不拷贝它所有的东西放置在另一块内存中。

&emsp; 为了优化这个问题，Swift 对于一些特定的值类型(集合类型：Array、Dictionary、Set)做了一些优化，在`对于 Array 进行拷贝的时候，当传递的值进行改变的时候才会发生真正的拷贝`。而对于`String、Int 等值类型，在赋值的时候就会发生拷贝`。下面来看代码验证一下：

**先看一下基本类型(Int、String等)**

```

var num1 = 10
var num2 = num1
print(address(of: &num1)) //0x7ffee0c29828
print(address(of: &num2)) //0x7ffee0c29820

var str1 = "abc"          
var str2 = str1
print(address(of: &str1)) //0x7ffee0c29810
print(address(of: &str2)) //0x7ffee0c29800

//打印内存地址
func address(of object: UnsafeRawPointer) -> String {
    let addr = Int(bitPattern: object)
    return String(format: "%p", addr)
}


```
**`从上面的打印我们可以看出基本类型在进行赋值的时候就发生了拷贝操作。`**




<br/>

**在看一下集合类型：**

```
var arr1 = [1,2,3,4,5]
var arr2 = arr1
print(address(of: &arr1)) //0x6000023b06b0
print(address(of: &arr2)) //0x6000023b06b0

arr2[2] = 4
print(address(of: &arr1)) //0x6000023b06b0
print(address(of: &arr2)) //0x6000023b11f0



```

**`从上面代码我们可以看出，当arr1赋值给arr2时并没有发生拷贝操作，当arr2的值改变的时候才发生了拷贝操作`**


<br/>

**原理：**
&emsp; 使用class，这是一个引用类型，因为当我们将引用类型分配给另一个时，两个变量将共享同一个实例，而不是像值类型一样复制它。

```
final class Ref<T> {
    var value: T
    init(value: T) {
        self.value = value
    }
}
```

创建一个struct包装Ref：

```
struct Box<T> {
    private var ref: Ref<T>
    init(value: T) {
        ref = Ref(value: value)
    }
 
    var value: T {
        get { return ref.value }
        set {
            guard isKnownUniquelyReferenced(&ref) else {
                ref = Ref(value: newValue)
                return
            }
            ref.value = newValue
        }
    }
}


```


由于struct是一个值类型，当我们将它分配给另一个变量时，它的值被复制，而属性ref的实例仍由两个副本共享，因为它是一个引用类型。

然后，我们第一次更改两个Box变量的值时，我们创建了一个新的ref实例，这要归功于

```
guard isKnownUniquelyReferenced(&ref) else {
    ref = Ref(value: newValue)
    return
}
```

两个Box变量不再共享相同的ref实例。

>为了提供高效的写时复制特性，我们需要知道一个对象是否是唯一的。如果它是唯一引用，那么我们就可以直接原地修改对象。否则，我们需要在修改前创 建对象的复制。在 Swift 中，我们可以使用 isKnownUniquelyReferenced 函数来检查某个引 用只有一个持有者。如果你将一个 Swift 类的实例传递给这个函数，并且没有其他变量强引用 这个对象的话，函数将返回 true。如果还有其他的强引用，则返回 false。不过，对于 Objective-C 的类，它会直接返回 false。



<br/>


- **总结：**
	- Copy-on-Write 是一种用来优化占用内存大的值类型的拷贝操作的机制。
	- 对于 Int、String 等基本类型的值类型，它们在赋值的时候就会发生拷贝，它们没有 Copy-on-Write 这一特性（因为Copy-on-Write带来的开销往往比直接复制的开销要大）。
	- 对于 Array、Dictionary、Se t类型，当它们赋值的时候不会发生拷贝，只有在修改的之后才会发生拷贝，即 Copy-on-Write。
	- 对于自定义的结构体，不支持 Copy-on-Write。


提问：为什么自定义struct不支持 Copy-on-Write？

但是我们可以这样做：

```
struct User {
    var identifier = 1
}


//使用
let user = User()
 
let box = Box(value: user)
// box2 shares instance of box.ref
var box2 = box                   
box2.value.identifier = 2 
```





<br/>
<br/>

>## <h2 id="值类型和引用类型区别">[值类型和引用类型区别](https://juejin.cn/post/6844904000794394637)</h2>

[值类型和引用类型](https://www.cnblogs.com/luoxiaofu/p/8528383.html)



- **存储区别：**
	- 值类型存储在栈区。 每个值类型变量都有其自己的数据副本，并且对一个变量的操作不会影响另一个变量。
	
	- 引用类型存储在其他位置（堆区），我们在内存中有一个指向该位置的引用。 引用类型的变量可以指向相同类型的数据。 因此，对一个变量进行的操作会影响另一变量所指向的数据。

&emsp; Swift有三种声明类型的方式：**class，struct和enum**。 它们可以分为值类型**（struct和enum）和引用类型（class）**。 

**一般Swift值类型在栈上分配。 引用类型在堆上分配。**



<br/>
<br/>


> <h2 id="关键字">关键字</h2>

<br/>

> <h3 id="final修饰符">final修饰符</h3>

- **使用final规则：**
	- final修饰符只能修饰类，表明该类不能被其他类继承，也就是它没资格当父类；
	- final修饰符也可以修饰类中的属性、方法和下标，但前提是该类并没有被final修饰过；
	- final不能修饰结构体和枚举；
	- 结构体和枚举只能遵循协议（protocol）。虽然协议也可以遵循其他协议，但是它并不能重写遵循的协议的任何成员，这就是结构体和枚举不需要final修饰的原因。




<br/>
<br/>


> <h2 id="@Objc和Dynamic的使用">@Objc和Dynamic的使用</h2>

[Swift动态性](https://juejin.cn/post/6888989886280368141)


&emsp; Swift Runtime system主要包括动态类型转换，泛型实例化和协议一致性注册，它仅用于描述编译器生成的代码应遵循的运行时接口，而不是有关如何实现的细节。

&emsp; Swift中的方法派发分为静态派发和动态派发，与Objective-C的消息派发机制不同，静态派发会在编译时确定方法的实现，并且以内联的方式对方法进行优化，指定函数被调用的指针；其中Swift结构体被分配在堆区，其函数默认是静态派发模式，以及使用final、private、static关键字修饰的类也是静态派发模式。动态派发是在程序运行时才确定方法的实现，Swift中使用dynamic修饰的方法是使用动态派发模式（ps：@objc修饰的方法不一定是动态派发，只是标明该方法对Objective-C可见）。

&emsp; 由于Swift为了性能，牺牲了它的动态性，使得我们在Swift层面上能做的事情很少。不过，由于Swift的类分为两种: 继承自NSObject的类以及默认继承自SwiftObject的类，既然Swift中有继承自NSObject的派生类，那么也就意味着OC的动态性也能在Swift里面应用





&emsp; **Objective-C** 和 **Swift** 在底层使用的是两套完全不同的机制:

- Cocoa 中的 Objective-C 对象是基于运行时的，它从骨子里遵循了 KVC (Key-Value Coding，通过类似字典的方式存储对象信息) 以及动态派发 (Dynamic Dispatch，在运行调用时再决定实际调用的具体实现)。
- Swift 为了追求性能，如果没有特殊需要的话，是不会在运行时再来决定这些的。也就是说，Swift 类型的成员或者方法在编译时就已经决定，而运行时便不再需要经过一次查找，而可以直接使用。

**问题是:** 如果我们要使用 Objective-C 的代码或者特性来调用纯 Swift 的类型时候，我们会因为找不到所需要的这些运行时信息而导致失败,怎么办？

1）.在 Swift 类型文件中，我们可以将需要暴露给 Objective-C 使用的任何地方 (包括类，属性和方法等) 的声明前面加上@objc修饰符。(注意这个步骤只需要对那些不是继承自NSObject的类型进行，如果你用 Swift 写的 class 是继承自NSObject的话，Swift 会默认自动为所有的非 private 的类和成员加上@objc。)

2）.@objc修饰符的另一个作用是为 Objective-C 侧重新声明方法或者变量的名字。虽然绝大部分时候自动转换的方法名已经足够好用 (比如会将 Swift 中类似init(name: String)的方法转换成-initWithName:(NSString *)name这样)，但是有时候我们还是期望 Objective-C 里使用和 Swift 中不一样的方法名或者类的名字;

3）.在[**Selector**](https://swifter.tips/selector/)一节中所提到的，即使是NSObject的子类，Swift 也不会在被标记为private的方法或成员上自动加@objc，以保证尽量不使用动态派发来提高代码执行效率。

4）.如果我们确定使用这些内容的动态特性的话，我们需要手动给它们加上@objc修饰.

但是需要注意的是，添加@objc修饰符并不意味着这个方法或者属性会变成动态派发，Swift 依然可能会将其优化为静态调用。

**一定要用dynamic情况：** 如果你需要和 Objective-C 里动态调用时相同的运行时特性的话，你需要使用的修饰符是dynamic。

5）.一般情况下在做 app 开发时应该用不上，但是在施展一些像动态替换方法或者运行时再决定实现这样的 "黑魔法" 的时候，我们就需要用到dynamic修饰符了。



<br/>

- **Dynamic**

	- Swift中也有dynamic关键字，它可以用于修饰变量或函数，它的意思也与OC完全不同。它告诉编译器使用动态分发而不是静态分发。OC区别于其他语言的一个特点在于它的动态性，任何方法调用实际上都是消息分发，而Swift则尽可能做到静态分发。

	- 因此，标记为dynamic的变量/函数会隐式的加上@objc关键字，它会使用OC的runtime机制。

	- 虽然静态分发在效率上可能更好，不过一些app分析统计的库需要依赖动态分发的特性，动态的添加一些统计代码，这一点在Swift的静态分发机制下很难完成。这种情况下，虽然使用dynamic关键字会牺牲因为使用静态分发而获得的一些性能优化，但也依然是值得的。

	- 使用动态分发，您可以更好的与OC中runtime的一些特性（如CoreData，KVC/KVO）进行交互，不过如果您不能确定变量或函数会被动态的修改、添加或使用了Method-Swizzle，那么就不应该使用dynamic关键字，否则有可能程序崩溃。











<br/>
<br/>


> <h3 id="Swfit中的@Objc和dynamic的原理">Swfit中的@Objc和dynamic的原理</h3>

[<br/>](https://gpake.github.io/2019/02/11/swiftMethodDispatchBrief/)

&emsp; 这个问题涉及到**Swift派发原理**

- **类类型(Class Type)**

	- 对于一个纯 Swift class 来说，默认使用 Table 派发，影响它方法调用的关键字有 final、 dynamic 和 extension。
	
	- 函数如果被标记成 final (可以在类和其类的extension(扩展)中使用)，编译器就会知道这个方法不会被 override，并把它的调用方式标记成直接调用。而对于未标记成 final 并在 class 内部（非 extension）中定义的方法，Swift 会用一种叫作 **Virtual Table 的机制**来在运行时查找到这个方法并进行调用。

	- 	当一个方法被标记为 dymanic，你必须同时把它标记上 @objc，此时这个方法会使用 **Message 调用**，依赖 Objc runtime。

	- 	因为定义在 extension 中的方法目前还不支持 override，所以定义在其中的方法都是直接派发的。


<br/>

- **值类型(Value Type)**

	- struct, enum 这样的值类型不支持继承，所以无需动态派发，它所有的方法调用（包括遵守的协议方法），都是直接调用。
	
	- 虽然不支持继承，但值类型还是可以通过 extension 和 Protocol 可以实现扩展。

<br/>

- **Protocol**

	- Protocol 是一个比较特殊的情况，不同于 Objective-C，Swift 在对待 Protocol 方法调用时更重视实例的类型，而不是实例的内在（比如 Objc 中的 isa）。

<br/>

**Dispatch**,这个Dispatch 是什么？

>Dispatch 派发，指的是**语言底层**找到用户想要调用的方法，并执行调用过程的动作。
Call 调用，指的是**语言在高级层面**，指示一个函数进行相关命令的行为。

- 对于一个编译型语言来说，一般有三种方式可以派发到方法：**静态派发，基于 Table 的派发，消息派发**(**前者也被称作 Static Dispatch，后两个为 Dynamic Dispatch。**)。举例：

	- Java 默认是使用 Table 方式派发的，你可以使用 final 关键字来强制动态派发。
	
	- C++ 默认是静态派发的，你可以使用 virtual 关键字来启用 Table 派发(一句话总结，编译器在编译期间就已经确定了的推断路径，运行期间按照既定路线走就行。)。
	
	- Objective-C 全都基于消息派发，不过也允许你使用 C level 的函数直接派发。
	
	- Swift 则巧妙的使用了这三种方法，分别应对不同的情况.


<br/>


**Dispatch种类：**

- Direct Dispatch 直接派发
	- 直接派发（静态派发）最快。不只是因为他的汇编命令少，还因为他可以应用很多编译器黑魔法，比如 inline 优化等。

	- 不过这种方式局限性最大，因为不够动态而无法支持子类。

<br/>

- **Table 派发**

	- 基于 Table 的派发机制是编译语言最常用的方式，Table 一般是用函数地址的数组来存储每个类的声明。大多数语言包括 Swift 都把这个称作 VTable，不过 Swift 中还有一个术语叫做 Witness Table 是服务于 Protocol 的，下面会提到。

	- 每个子类都会有自己的 VTable 副本，子类中 override 的方法指针也会被替换成新的，子类新添加的方法则会被添加在 Table 的尾部。程序会在运行时确定每个函数具体的地址。

	- 表的查找就实现而言非常简单，而且性能可以预测，不过还是比直接派发更慢一些。从字节码的角度来说，这种方法多了两步查找和一部跳转，这些都是开销。而且，这种方法没法使用一些黑魔法来优化。

	- 另一个不好的点在于，这种基于数组的派发让 extension 没法扩展这个 table。因为在子类添加方法列表到尾部后，extension 就没有一个安全的 index 可以添加他的方法到 table 中。



<br/>


- **Message 派发**

	- Message 派发是最灵活的派发方式。他是 Cocoa 的基石，也是 KVO，UIAppearance，Core Data 的核心。

	- 他的关键功能是可以让开发者在运行时改变消息发送的行为。不仅可以通过 swizzling 修改 method，还可以通过 isa-swizzling 来修改对象。

	- 一旦有消息发出，runtime 就会基于继承关系开始查找，虽然听起来很慢，但是有 cache 做保障，一旦 cache 经过预热，就和 Table 方式差不多快了。

<br/>


[**队列派发**](https://hechen.xyz/post/messagedispatchinswift/#pwt)

&emsp; 在 Swift 中，针对拥有继承的 Class 类型来说，依然采用了 V-Table 这种实现形式达到对多态的支持，而针对值类型由于 Protocol 产生的多态而采用了另外一种函数表派发形式，这个表格被称为协议目击表 （Protocol Witness Table，简称 PWT），这个暂时略去不表。


**V-Table**

- 对于 V-Table 的应用场景下，每一个类都会维护一个函数表，里面记录着该类所有的函数指针，主要包含：

	- 由父类继承而来的方法执行地址；
	- 如果子类覆写了父类方法的话，表格里面就会保存被重载之后的函数。
	- 子类新添加的函数会插入到这个表格的末尾
- 在程序运行期间会根据这个表去查询真正要调用的函数是哪一个。这种特性就极大的提升了代码的灵活性，也是 Java，C++ 等语言得以支持多态这种语言特性的基石。当然，相对于静态派发而言，表格派发则多了很多的隐式花销，因为函数调用不再是直接调用了，而是需要通过先行查表的形式找到真实的函数指针来执行，编译器也无法再进行诸如 inline 等编译期优化了。


<br/>

- **PWT**

	- 对于 Swift 来说，还有更为重要的 Protocol，对于符合同一协议的对象本身是不一定拥有继承关系的，因此 V-Table 就没法使用了。这里，Swift 使用了 Protocol Witness Table 的数据结构达到动态查询协议方法的目的。如果将上面的例子中的 Drawable 抽象成协议。




<br/>


- **关键字的影响:**
	
	- **final**
		- 使用了 final 的，都用静态派发，因为 final 意味着完全没有动态性。final 用于类型，或是 function，都会造成这样的情况。
		
		- 而且 final 作用范围的方法，都不会被 export 到 OC runtime，也不会生成 selector



	- **dynamic**
		- 使用了 dynamic 的 class （只有 class 可以 dynamic），会开启 message 模式，让 OC runtime 可以调用。

		- 必须 @import Foundation，必须有 @objc，如果是 class，还必须是 NSObject 的子类

		- 延展阅读： dynamic vs @objc: dynamic 是强制使用 message 派发，KVO 需要。@objc 只是 export 给 objc，swift 还是静态派发


	- **@objc / @nonobjc**
		- @objc / @nonobjc 控制方法对于 objc 的可见性。但是不会改变 swift 中的函数如何被派发。

		- @nonobjc 可以禁止函数使用 message 派发 （和 final 在汇编上看起来类似，偏向于使用 final)

		- // 并不会这样，@nonobjc 依然是使用 Table 调用，但是 @nonobjc 之后无法使用 dynamic，会提示 error: a declaration cannot be both '@nonobjc' and 'dynamic'

		- @objc 的原理是生成两个函数引用，一个给 swift 调用，一个给 objc 调用

		- @objc final func aFunc() {} 会让消息使用直接派发，不过依然会 export 给 objc


	- **@inline**
		- @inline 可以告诉编译器去优化直接派发的性能。
	
		- see: https://github.com/vandadnp/swift-weekly/blob/master/issue11/README.md#inline
	
		- @inline 可以给生成的函数加上标记，后期编译器进行指定的优化
	
		- inline 可以选择的参数有两个 never 和 __always
	
		- 对于 inline 的使用建议：
			- 0，默认行为是编译器自己决定要不要使用 inline 进行优化，你也应该保持默认行为。
			- 1，如果你的函数特别长，你不想让你的打包体积变得特别大，可以使用 @inline(never)
			- 2，如果你的函数很小，你希望他可以快一点，就可以使用 @inline(__always)，不过其实编译器也会帮你做这样的事情，所以你这么做也基本上不会让他变得更快
			- 有趣的是，如果你用 dynamic @inline(__always) func dynamicOrDirect() {} 依然会得到一个 message 派发的函数。



<br/>

**方法可见性的影响**

&ems; Swift 编译器会尽可能的帮你优化派发，比如：你的方法没有被继承，那他就会注意到这个，并用尝试使用直接派发来优化性能。

<br/>

**KVO**

&ems;值得注意的是 KVO，被观察的属性也必须被声明为 dynamic，否则 setter 会走直接派发，无法触发变化。



**派发效率从高到低为： 直接派发 > Table 派发 > Message 派发**



<br/>
<br/>


> <h2 id="语法基础">语法基础</h2>

<br/>

> <h3 id="只能被类遵守的protocol">只能被类遵守的protocol</h3>

```
protocol OnlyClassProtocol : class {
}
```


<br/>


> <h3 id="defer使用场景">defer使用场景</h3>

defer 语句块中的代码, 会在当前作用域结束前调用, 常用场景如异常退出后, 关闭数据库连接



```
func someQuery() -> ([Result], [Result]){
    let db = DBOpen("xxx")
    defer {
        db.close()
    }
    guard results1 = db.query("query1") else {
        return nil
    }
    guard results2 = db.query("query2") else {
        return nil
    }
    return (results1, results2)
}

```

需要注意的是, 如果有多个 defer, 那么后加入的先执行


```

func someDeferFunction() {
    defer {
        print("\(#function)-end-1-1")
        print("\(#function)-end-1-2")
    }
    defer {
        print("\(#function)-end-2-1")
        print("\(#function)-end-2-2")
    }
    if true {
        defer {
            print("if defer")
        }
        print("if end")
    }
    print("function end")
}
someDeferFunction()
// 输出
// if end
// if defer
// function end
// someDeferFunction()-end-2-1
// someDeferFunction()-end-2-2
// someDeferFunction()-end-1-1
// someDeferFunction()-end-1-2
```

<br/>


> <h3 id="数组索引越界会Crash,字典取不到为nil">数组索引越界会Crash,字典取不到为nil</h3>

- 数组索引本来就是访问一段连续地址,越界访问也能访问到内存，但这段内存不一定可用，所以会引起Crash.
- 字典的key并没有对应确定的内存地址,所以是安全的.




<br/>


> <h3 id="Self的使用场景">Self的使用场景</h3>

Self 通常在协议中使用, 用来表示实现者或者实现者的子类类型.定义一个复制的协议:

```
protocol CopyProtocol {
    func copy() -> Self
}
```


如果是结构体去实现, 要将Self 换为具体的类型:

```
struct SomeStruct: CopyProtocol {
    let value: Int
    func copySelf() -> SomeStruct {
        return SomeStruct(value: self.value)
    }
}
```

如果是类去实现, 则有点复杂, 需要有一个 required 初始化方法, 具体可以看这里:


```
class SomeCopyableClass: CopyProtocol {
    func copySelf() -> Self {
        return type(of: self).init()
    }
    required init(){}
}
```


<br/>


> <h3 id="throws和rethrows的用法与作用">throws和rethrows的用法与作用</h3>

throws 用在函数上, 表示这个函数会抛出错误.
有两种情况会抛出错误, 一种是直接使用 throw 抛出, 另一种是调用其他抛出异常的函数时, 直接使用 try xx 没有处理异常.

```
enum DivideError: Error {
    case EqualZeroError;
}
func divide(_ a: Double, _ b: Double) throws -> Double {
    guard b != Double(0) else {
        throw DivideError.EqualZeroError
    }
    return a / b
}
func split(pieces: Int) throws -> Double {
    return try divide(1, Double(pieces))
}
```


rethrows 与 throws 类似, 不过只适用于参数中有函数, 且函数会抛出异常的情况, rethrows 可以用 throws 替换, 反过来不行

```
func processNumber(a: Double, b: Double, function: (Double, Double) throws -> Double) rethrows -> Double {
    return try function(a, b)
}
```

<br/>


> <h3 id="值类型和引用类型在什么时候使用">值类型和引用类型在什么时候使用</h3>

- **什么时候该用值类型：**
	- 要用==运算符来比较实例的数据时.
	- 你希望那个实例的拷贝能保持独立的状态时.
	- 数据会被多个线程使用时.

- **什么时候该用引用类型（class）：**
	- 	要用==运算符来比较实例身份的时候.
	- 	你希望有创建一个共享的、可变对象的时候.


```
let test1 = Test1()
let test2 = Test1()
//test1地址：SwiftTest.Test1	0x0000600003a1c300
//test2地址：SwiftTest.Test1	0x0000600003a1c320
if test1 === test2 {
    print("test1 == test2 ")
}else {
    print("test1 != test2 ")
    
}

let a1 = "123"
let a2 = "123"


if a1 == a2 {
    print("a1 == a2 ")
}else {
    print("a1 != a2 ")
    
}
```

打印：

```
test1 != test2 
a1 == a2 
```





<br/>


> <h3 id="">协议的动态</h3>

```
protocol Pizzeria { 
  func makePizza(_ ingredients: [String])
  func makeMargherita()
} 

extension Pizzeria { 
  func makeMargherita() { 
    return makePizza(["tomato", "mozzarella"]) 
  }
}

struct Lombardis: Pizzeria { 
  func makePizza(_ ingredients: [String]) { 
    print(ingredients)
  } 

  func makeMargherita() {
    return makePizza(["tomato", "basil", "mozzarella"]) 
  }
}

let lombardis1: Pizzeria  = Lombardis()
let lombardis2: Lombardis = Lombardis() 
lombardis1.makeMargherita()
lombardis2.makeMargherita()


//打印
["tomato", "basil", "mozzarella"]
["tomato", "basil", "mozzarella"]
```

分析: 在Lombardis的代码中，重写了makeMargherita的代码，所以永远调用的是Lombardis 中的 makeMargherita.

再进一步，我们把 protocol Pizzeria 中的 func makeMargherita() 删掉，代码变为:



```
protocol Pizzeria {
  func makePizza(_ ingredients: [String])
}

extension Pizzeria  {
  func makeMargherita()  {
    return makePizza(["tomato", "mozzarella"])
  }
}

struct Lombardis: Pizzeria  {
  func makePizza(_ ingredients: [String])  {
    print(ingredients)
  }
  func makeMargherita() {
    return makePizza(["tomato", "basil", "mozzarella"])
  }
}
let lombardis1: Pizzeria = Lombardis()
let lombardis2: Lombardis = Lombardis()
lombardis1.makeMargherita()
lombardis2.makeMargherita()

* 打印结果:
["tomato", "mozzarella"]
["tomato", "basil", "mozzarella"]
```

因为lombardis1 是 Pizzeria，而 makeMargherita() 有默认实现，这时候我们调用默认实现。



<br/>


> <h3 id="Swift和OC常量区别">Swift和OC常量区别</h3>

OC中定义的常量:

```
const int number = 0;
```


Swift 是这样定义常量的：
```
let number: Int = 0
```


- **区别:**
	- OC中用 const 来表示常量，而 Swift 中用 let 来判断是不是常量.
	- OC中 const 表明的常量类型和数值是在 compilation time 时确定的；
	- Swift 中 let 只是表明常量（只能赋值一次），其类型和值既可以是静态的，也可以是一个动态的计算方法，它们在 runtime 时确定的。





<br/>


> <h3 id="autoclosure的作用">autoclosure的作用</h3>

自动闭包, 会自动将某一个表达式封装为闭包

```
func autoClosureFunction(_ closure: @autoclosure () -> Int) {
   closure()
}
autoClosureFunction(1)

```


<br/>


> <h3 id="编译选择wholemoduleoptmization优化了什么">编译选择 whole module optmization 优化了什么</h3>

编译器可以跨文件优化编译代码, 不局限于一个文件;
[这里](https://www.jianshu.com/p/8dbf2bb05a1c)


<br/>


> <h3 id="mutaing的作用">mutaing的作用</h3>

- **作用1：**

```
struct Person  {
   var name: String  {
       mutating get  {
        return store
        }
     }
} 
```
让不可变对象无法访问name 属性;


<br/>


> <h3 id="怎么表示函数的参数类型只要是数字">函数的参数类型只要是数字（Int、Float）都可以，要怎么表示</h3>

使用泛型

```
func isNumber<T : SignedNumber>(number : T){
print(" it is a number")
}
```



<br/>


> <h3 id="dynamic的作用">dynamic的作用.</h3>

- **dynamic的作用.**

	- 由于swift是一门静态语言，所以没有Objective-C中的消息发送这些动态机制，dynamic的作用就是让swift代码也能有oc中的动态机制，常用的就是KVO。
	- 使用dynamic关键字标记属性，使属性启用Objc的动态转发功能；
	- dynamic只用于类，不能用于结构体和枚举，因为它们没有继承机制，而Objc的动态转发就是根据继承关系来实现转发。





<br/>


>### <h3 id="Swift的动态性">[Swift的动态性](https://juejin.cn/post/6888989886280368141)</h3>



<br/>


> <h3 id=""></h3>

<br/>

***
<br/>


> <h1 id="常用类库">常用类库</h1>
<br/>

> <h2 id="ObjectMapper">ObjectMapper</h2>

使用ObjectMapper的时候，我们一定要实现Mappable协议。这个协议里又有两个要实现的方法：

```
init?(map: Map)
mutating func mapping(map: Map)
```

流程解析如下：

```
//其对应的JSON如下：
let json = """
{
"mathematics": "excellent",
"physics": "bad",
"chemistry": "fine"
}



//使用的时候，只用如下:
struct Ability: Mappable {
	var mathematics: String?
	var physics: String?
	var chemistry: String?

	init?(map: Map) {

	}

	mutating func mapping(map: Map) {
		mathematics <- map["mathematics"]
		physics <- map["physics"]
		chemistry <- map["chemistry"]
	}
}



//然后将这段json解析为Ability的Model, 即:
let ability = Mapper<Ability>().map(JSONString: json)
```

那么 **'<-'** 这么奇怪的符号，它又是啥意思呢？

去看源码，发现这个符号是这个库自定义的一个操作符，在Operators.swift里如下：

```
infix operator <-

/// Object of Basic type
public func <- <T>(left: inout T, right: Map) {
	switch right.mappingType {
	case .fromJSON where right.isKeyPresent:
		FromJSON.basicType(&left, object: right.value())
	case .toJSON:
		left >>> right
	default: ()
	}
}

```

然后根据不同的泛型类型，这个操作符会进行不同的处理。

接着，我们再看一下map方法。

map方法存在于Mapper类中, 定义如下:

```

 func map(JSONString: String) -> M? {
	if let JSON = Mapper.parseJSONString(JSONString: JSONString) as? [String: Any] {
  		return map(JSON: JSON)
	}
	return nil
}

func map(JSON: [String: Any]) -> M? {
	let map = Map(JSON: JSON)
	if let klass = M.self as? Mappable.Type {
  	if var obj = klass.init(map: map) as? M {
    	obj.mapping(map: map)
    	return obj
  		}
	}
	return nil
}

```

可以看到，在map的方法中，我们最后会调用Mappable协议中定义的mapping方法，来对json数据做出转化。

最后再看一下Map这个类，这个类主要用来处理找到key所对应的value。处理方式如下:


```
private func valueFor(_ keyPathComponents: ArraySlice<String>, dict: [String: Any]) -> (Bool, Any?) {
	guard !keyPathComponents.isEmpty else { return (false, nil) }

	if let keyPath = keyPathComponents.first {
  		let obj = dict[keyPath]
  		if obj is NSNull {
    		return (true, nil)
  		} else if keyPathComponents.count > 1, let d = obj as? [String: Any] {
    		let tail = keyPathComponents.dropFirst()
    		return valueFor(tail, dict: d)
  		} else if keyPathComponents.count > 1, let arr = obj as? [Any] {
    		let tail = keyPathComponents.dropFirst()
    		return valueFor(tail, array: arr)
  		} else {
    		return (obj != nil, obj)
  		}
	}

	return (false, nil)
}

private func valueFor(_ keyPathComponents: ArraySlice<String>, array: [Any]) -> (Bool, Any?) {
	guard !keyPathComponents.isEmpty else { return (false, nil) }

	if let keyPath = keyPathComponents.first, let index = Int(keyPath), index >= 0 && index < array.count {
  		let obj = array[index]
  		if obj is NSNull {
    		return (true, nil)
  		} else if keyPathComponents.count > 1, let dict = obj as? [String: Any] {
    		let tail = keyPathComponents.dropFirst()
    		return valueFor(tail, dict: dict)
  		} else if keyPathComponents.count > 1, let arr = obj as? [Any] {
    		let tail = keyPathComponents.dropFirst()
    		return valueFor(tail, array: arr)
  		} else {
    		return (true, obj)
  		}
	}

	return (false, nil)
}
```


其中在处理分隔符上，采用的是递归调用的方式，不过就我们目前项目中，还没有用到过。

上述这几个步骤，就是ObjectMapper的核心方法。我也根据这些步骤，自己实现了一个解析的库。

但是这个只能解析一些最简单的类型，其他的像enum之类的，还需要做一些自定义的转化。主要的数据转化都在Operators文件夹中。





<br/>
<br/>



> <h2 id="SwiftJSON">SwiftJSON</h2>


构造器

SwiftyJSON对外暴露的主要的构造器:


```
public init(data: Data, options opt: JSONSerialization.ReadingOptions = []) throws
public init(_ object: Any)
public init(parseJSON jsonString: String)
```




<br/>
<br/>

>## <h2 id="RxSwift">[RxSwift](https://juejin.cn/post/6844903862571106317#heading-2)</h2>





<br/>
<br/>

>## <h2 id="Snapkit">[Snapkit](https://www.jianshu.com/p/44f3d812839f)</h2>






<br/>

***
<br/>


> <h1 id="数据存储">数据存储</h1>
	
<br/>
	
<h2 id="CoreData">CoreData</h2>

&emsp; 一个基本的 [**Core Data**](https://objccn.io/products/core-data/preview/) 栈由四个主要部分组成：托管对象 (managed objects) (**NSManagedObject**)，托管对象上下文 (managed object context) (**NSManagedObjectContext**)，持久化存储协调器 (persistent store coordinator) (**NSPersistentStoreCoordinator**)，以及持久化存储 (persistent store) (**NSPersistentStore**):

![<br/>](./../../Pictures/ios_pd13.png)

- **托管对象:** 位于这张图的最上层，它是架构里最有趣的部分，同时也是我们的数据模型 - 在这个例子里，它是 Mood 类的实例们。Mood 需要是 NSManagedObject 类的子类，这样它才能与 Core Data 其他的部分进行集成。每个 Mood 实例表示了一个 **mood**，也就是用户用相机拍摄的照片。

- **托管对象上下文：** **mood** 对象是被 Core Data 托管的对象。也就是说，它们存在于一个特定的上下文 (context) 里：那就是托管对象上下文。托管对象上下文记录了它管理的对象，以及你对这些对象的所有操作，比如插入，删除和修改等。每个被托管的对象都知道自己属于哪个上下文。

- **持久化存储协调器：**上下文与持久化存储协调器相连，协调器位于持久化存储和托管对象上下文之间。对于本章中的这个简单例子，我们不用太关心持久化存储协调器或者持久化存储，因为 NSPersistentContainer 这个辅助类会帮助我们把它们都设置好。可以这么说，默认情况下 Core Data 会使用一个 SQLite 类型的持久化存储，也就是说你的数据在底层实际上会被存储在一个 SQLite 数据库里。Core Data 也提供其他的存储类型 (比如 XML，二进制数据，内存)，但是现在我们不需要考虑其他的存储类型。



<br/>

***
<br/>



> <h1 id="组件化">组件化</h1>

[iOS组件化实践](https://www.jianshu.com/p/510ee1290ab4)


<br/>

***
<br/>



> <h1 id="路由导航">路由导航</h1>

[路由设计思路分析](https://github.com/harleyGit/StudyNotes/blob/master/Sources/iOS组件化路由设计思路分析.pdf)
[](https://www.cnblogs.com/oc-bowen/p/6489070.html)

[路由导航](http://www.cocoachina.com/cms/wap.php?action=article&id=27025)
[ALRouter路由导航](https://www.jianshu.com/p/61f20e23afc0)





<br/>

***
<br/>

> <h1 id="安全">安全</h1>

<br/>

> <h2 id="MD5">MD5</h2>

<br/>

- 使用场景：

&emsp; MD5常用在密码加密中，一般为了保证用户密码的安全，在数据库中存储的都是用户的密码经过MD5加密后的值，在客户端用户输入密码后，也会使用MD5进行加密，这样即使用户的网络被窃听，窃听者依然无法拿到用户的原始密码，并且即使用户数据库被盗，没有存储明文的密码对用户来说也多了一层安全保障。

&emsp;MD5签名技术还常用于防止信息的篡改。使用MD5可以对进行进行签名，接收者拿到信息后只要重新计算签名和原始签名进行对比，即可知道数据信息是否中途被篡改了。

<br/>

- MD5 加密参数可能会发生丢失？解释下原理？

[MD5原理](https://cloud.tencent.com/developer/article/1402024)


OC实现MD5加密：

获得MD5加密

```
- (NSString *)md5:(NSString *)str {

	NSString *resultStr = nil;
	
	const char *cStr = [str UTF8String];//指针不能变，cStr指针变量本身可以变化
	
	unsigned char result[16];//这里可以CC_MD5_DIGEST_LENGTH宏代替16，这里不明白为什么写16后面我有解释的！
	
	CC_MD5(cStr, (unsigned int)strlen(cStr), result);
	
	resultStr = [NSString stringWithFormat:
	
	@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
	
	result[0], result[1], result[2], result[3],
	
	result[4], result[5], result[6], result[7],
	
	result[8], result[9], result[10], result[11],
	
	result[12], result[13], result[14], result[15]
	
	];
	
	return [resultStr uppercaseString];
	
}
```
值得注意的是：

&emsp; 其中%02x是格式控制符：‘x’表示以16进制输出，‘02’表示不足两位，前面补0；如‘f’输出为0f，‘1f3’则输出1f3;还有就是为什么是result[16]呢，这是因为MD5算法最后生成的是128位，而在计算机的最小存储单位为字节，1个字节是8位，对应一个char类型，计算可得需要16个char。所以result是[16]。那么为什么输出的格式一定是%02x呢，而不是其它呢。这也是有原因的：因为约定MD5一般是以16进制的格式输出，那么其实这个问题就转换为把128个0和1以16进制来表示，每4位二进制对应一个16进制的元素，则需要32个16进制的元素，如果元素全部为0，放到char的数组中，正常是不会输出，如00001111，以%x输出，则是f,那么就会丢失0；但如果以%02x表示则输出结果是0f，正好是转换的正确结果。

所以以上就是char[16]和%02x的来历。




<br/>
<br/>


> <h2 id="防止反编译">如何防止反编译？</h2>

[防止反编译](https://icocos.github.io/2017/10/26/iOS——防止反编译总结/)

<br/>

- **本地数据加密**

对NSUserDefaults，sqlite存储文件数据加密，保护帐号和关键信息,将文件进行加密

注意： **Base64并不是一种加密方式**，明文使用Base64编码后的字符串通过索引表可以直接还原为明文。因此，Base64只能作为一种数据的存储格式。

```
// 获取需要加密文件的二进制数据
NSData *data = [NSData dataWithContentsOfFile:@"/Users/wangpengfei/Desktop/photo/IMG_5551.jpg"];

// 或 base64EncodedStringWithOptions
NSData *base64Data = [data base64EncodedDataWithOptions:0];

// 将加密后的文件存储到桌面
[base64Data writeToFile:@"/Users/wangpengfei/Desktop/123" atomically:YES];
    
```

将文件进行解密,**不过我们可以对其中编码后的文本进行替换就好了。😂**

```
// 获得加密后的二进制数据
NSData *base64Data = [NSData dataWithContentsOfFile:@"/Users/wangpengfei/Desktop/123"];

// 解密 base64 数据
NSData *baseData = [[NSData alloc] initWithBase64EncodedData:base64Data options:0];

// 写入桌面
[baseData writeToFile:@"/Users/wangpengfei/Desktop/IMG_5551.jpg" atomically:YES];
```    



<br/>

- **URL编码加密**
对程序中出现的URL进行编码加密，防止URL被静态分析
ARC模式下,编码

```
+ (NSString *)encodeToPercentEscapeString: (NSString *) input

{
    
    NSString *outputStr =
    
    (__bridge NSString *)CFURLCreateStringByAddingPercentEscapes(
                                                                 
                                                                 NULL, /* allocator */
                                                                 
                                                                 (__bridge CFStringRef)input,
                                                                 
                                                                 NULL, /* charactersToLeaveUnescaped */
                                                                 
                                                                 (CFStringRef)@"!*'();:@&=+$,/?%#[]",
                                                                 
                                                                 kCFStringEncodingUTF8);
    
    return outputStr;
    
}
```

解码

```
+ (NSString *)decodeFromPercentEscapeString: (NSString *) input

{
    
    NSMutableString *outputStr = [NSMutableString stringWithString:input];
    
    [outputStr replaceOccurrencesOfString:@"+"
     
                               withString:@""
     
                                  options:NSLiteralSearch
     
                                    range:NSMakeRange(0,[outputStr length])];
    
    return
    
    [outputStr stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    
}

```


<br/>


- 网络传输数据加密

使用MD5对一些参数进行hash防篡改：
需要导入第三方框架: NSString+Hash

```
NSString *password = @"WangPengfei";
password = [password md5String];
NSLog(@"password1:%@", password);


//为了保险，可以进行加盐
NSString *salt = @"234567890-!@#$%^&*()_+QWERTYUIOP{ASDFGHJKL:XCVBNM<>";
[password stringByAppendingString:salt];
password = [password md5String];
NSLog(@"password2:%@", password);
```


<br/>


- **方法体，方法名高级混淆**

对应用程序的方法名和方法体进行混淆，保证源码被逆向后无法解析代码。 使用hopper disassembler 反编译iPA之后不能得到相应的方法调用信息。

![<br/>](https://user-gold-cdn.xitu.io/2018/1/11/160e332682495fe8?imageView2/0/w/1280/h/960/ignore-error/1)

创建shell脚本：

```
TABLENAME=symbols
SYMBOL_DB_FILE="symbols"
STRING_SYMBOL_FILE="fun.list"
HEAD_FILE="$PROJECT_DIR/$PROJECT_NAME/codeObfuscation.h"
export LC_CTYPE=C

createTable(){
    echo "create table $TABLENAME(src text, des text);" | sqlite3 $SYMBOL_DB_FILE
}

insertValue(){
    echo "insert into $TABLENAME values('$1' ,'$2');" | sqlite3  $SYMBOL_DB_FILE
}

query(){
    echo "select * from $TABLENAME where src='$1';" | sqlite3 $SYMBOL_DB_FILE
}

ramdomString(){
    openssl rand -base64 64 | tr -cd 'a-zA-Z' |head -c 16
}

rm -f $SYMBOL_DB_FILE
rm -f $HEAD_FILE
createTable

touch $HEAD_FILE
echo '#ifndef Demo_codeObfuscation_h
#define Demo_codeObfuscation_h' >> $HEAD_FILE
echo "//confuse string at `date`" >> $HEAD_FILE
cat "$STRING_SYMBOL_FILE" | while read -ra line; do
if [[ ! -z "$line" ]]; then
ramdom=`ramdomString`
echo $line $ramdom
insertValue $line $ramdom
echo "#define $line $ramdom" >> $HEAD_FILE
fi
done
echo "#endif" >> $HEAD_FILE
sqlite3 $SYMBOL_DB_FILE .dump


```


声明要替换的方法名列表，[运行Shell脚本](https://www.jianshu.com/p/5fb895ca503d)

```
//在上边脚本中提到了 STRING_SYMBOL_FILE="fun.list"，意思就是运行脚本的时候会到这个文件去读取需要替换的方法名，重新写入符号表中。
nameAction
refreshAction

```


生成对应的转义之后的无序字符串


![<br/>](https://user-gold-cdn.xitu.io/2018/1/11/160e332699f100d4?imageView2/0/w/1280/h/960/ignore-error/1)



<br/>

- **程序结构混排加密**


<br/>

- **借助第三方APP加固**

如：[网易易盾](https://dun.163.com/?from=baiduP_PP_PPWYY1)




<br/>

***
<br/>

> <h1 id="多线程">多线程</h1>


<br/>

>## <h2 id="NSOperation">[NSOperation](https://www.jianshu.com/p/d8caf596d5d0)</h2>

<br/>

> <h2 id="GCD控制线程数量">GCD控制线程数量</h2>


GCD 不像 NSOperation 那样有直接提供线程数量控制方法，但是通过 GCD 的 semaphore 功能一样可以达到控制线程数量的效果。

- dispatch_semaphore_create(long value); 利用给定的输出时创建一个新的可计数的信号量

- dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout); 如果信号量大于 0 ，信号量减 1 ，执行程序。否则等待信号量

- dispatch_semaphore_signal(dispatch_semaphore_t dsema); 增加信号量

```
// 控制线程数量
- (void)runMaxThreadCountWithGCD
{
    dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentRunMaxThreadCountWithGCD", DISPATCH_QUEUE_CONCURRENT);
    dispatch_queue_t serialQueue = dispatch_queue_create("serialRunMaxThreadCountWithGCD", DISPATCH_QUEUE_SERIAL);
    // 创建一个semaphore,并设置最大信号量，最大信号量表示最大线程数量
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(2);
    // 使用循环往串行队列 serialQueue 增加 10 个任务
    for (int i = 0; i < 10 ; i++) {
        dispatch_async(serialQueue, ^{
            // 只有当信号量大于 0 的时候，线程将信号量减 1，程序向下执行
            // 否则线程会阻塞并且一直等待，直到信号量大于 0
            dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
            dispatch_async(concurrentQueue, ^{
                NSLog(@"%@ 执行任务一次  i = %d",[NSThread currentThread],i);
                // 当线程任务执行完成之后，发送一个信号，增加信号量。
                dispatch_semaphore_signal(semaphore);
            });
        });
    }
    NSLog(@"%@ 执行任务结束",[NSThread currentThread]);
}
```


<br/>
<br/>

>## <h2 id="线程锁">[线程锁](https://www.jianshu.com/p/256f555cf7b5)</h2>

线程锁的种类：

```
 iOS中的常用的锁有如下几种：
 1、自旋锁：
    使用与多线程同步的一种锁，线程反复检查锁变量是否可用。由于线程在这一过程中保持执行，因此是一种忙等待。一旦获取了自旋锁，线程会一直保持该锁，直到显示释放自旋锁。自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的。NSSpinLock ,它现在被废弃了，不能使用了，它是有缺陷的，会造成死锁
 2、互斥锁
    是一种用于多线程编程中，防止两条线程同时对同一公共资源（例如：同一个全局变量）进行读写的机制。互相排斥。例如线程A获取到锁，在释放锁之前，其他线程都获取不到锁。互斥锁也分为两种：递归锁和非递归锁。互斥锁是通过将代码切片成一个一个的临时区来实现。p_thread_mutex,NSLock,@synchronized这个顺序是按照性能排序的，也是我们常用的几个互斥锁。
 3、读写锁：
    计算机程序的并发控制的一种同步机制，也称“共享 - 互斥锁”、多个读者，单个作者（写入）的锁机制。用于解决多线程对公共资源读写问题，读操作可并发重入，写操作时互斥的。读写锁通常用互斥锁、条件变量、信号量实现。
 4、信号量：
    是一种更高级的同步机制，有更多的取值空间。用来实现更加复杂的同步，而不单单是线程间互斥。semphone在一定程度也可以当互斥锁用，它适用于编程逻辑更复杂的场景，同时它也是除了自旋锁以外性能最高的锁
 5、条件锁：
    就是条件变量，当进程的某些资源要求不满足时就锁住进入休眠。当资源被分配到了，条件锁打开继续运行。NSCondition,条件锁我们调用wait方法就把当前线程进入等待状态，当调用了signal方法就可以让该线程继续执行，也可以调用broadcast广播方法。

 临时区：
    指的是一块对公共资源进行访问的代码，并非一种机制或是算法。
```

- **iOS开发中常用的锁有如下几种：**
	- @synchronized
	- NSLock 非递归互斥锁
	- NSRecursiveLock 递归锁
	- NSConditionLock 条件锁
	- pthread_mutex 互斥锁（C语言）
	- dispatch_semaphore 信号量实现加锁（GCD）
	- OSSpinLock （暂不建议使用，原因参见这里）


<br/>

- **自旋锁和互斥锁的一些问题**
	- 互斥锁
		- 互斥锁又分 递归锁(NSRecursiveLock 等) 和 非递归锁(NSLock 等)。
		- 递归锁：可重入锁，统一线程在锁释放前可再次获取锁，即可以递归调用
		- 非递归锁：不可重入，必须等锁释放后才能再次获取锁。

	- 自旋锁和互斥锁的区别？
		- 互斥锁：当线程获取锁但没有获取到时，线程进入休眠状态。等到锁被释放，线程会被唤醒同时获取到锁。继续执行任务改变线程状态。
		- 自旋锁：当线程获取锁没有获取到时，不会进入休眠，而是一直循环看是否可用。线程一直处于活跃状态，不会改变线程状态。

	- 自旋锁和互斥锁的使用场景分别是？
		- 自旋锁：由于自旋锁一直等待会消耗较多CPU 资源，但是效率较高一旦锁释放立刻就能执行无序唤醒。所以适用于短时间内的轻量级锁定。
		- 互斥锁：需要修改线程状态，唤醒或休眠线程。所以适用于时间长相对自旋锁效率低的场景。



<br/>
<br/>

>## <h2 id="文件读写锁">[文件读写锁](https://github.com/harleyGit/StudyNotes/blob/master/多线程/GCD(II).md)</h2>



<br/>

***
<br/>


> <h1 id="底层">底层</h2>

<br/>

> <h2 id="Swift动态性">Swift动态性</h2>

- **Swift Runtime**
	- **Swift Runtime system**主要包括动态类型转换，泛型实例化和协议一致性注册，它仅用于描述编译器生成的代码应遵循的运行时接口，而不是有关如何实现的细节。
	- Swift中的方法派发分为[静态派发和动态派发](https://www.jianshu.com/p/e0659093eaac)，与Objective-C的消息派发机制不同，静态派发会在编译时确定方法的实现，并且以内联的方式对方法进行优化，指定函数被调用的指针；其中Swift结构体被分配在堆区，其函数默认是静态派发模式，以及使用final、private、static关键字修饰的类也是静态派发模式。动态派发是在程序运行时才确定方法的实现，Swift中使用dynamic修饰的方法是使用动态派发模式（ps：@objc修饰的方法不一定是动态派发，只是标明该方法对Objective-C可见）。
	- 由于Swift为了性能，牺牲了它的动态性，使得我们在Swift层面上能做的事情很少。不过，由于Swift的类分为两种: 继承自NSObject的类以及默认继承自SwiftObject的类，既然Swift中有继承自NSObject的派生类，那么也就意味着OC的动态性也能在Swift里面应用



<br/>


- **OC Runtime**
	- 在Swift中，继承自NSObject的类都保留了其动态性，所以我们可以通过OC runtime获取到其方法，所以，也可以通过这个方式对Swift代码进行hook。
	- 从OC的运行时特性可知，所有的运行时方法都依赖TypeEncoding，也就是method_getTypeEncoding返回的结果，他指定了方法的参数类型以及在函数调用时参数入栈所要的内存空间，没有这个标识就无法动态的压入参数，而一些Swift特有的类型无法映射到OC的类型,也无法用OC的typeEncoding表示，就没法通过runtime获取;
	- 除了继承自NSObject的类之外，继承自SwiftObject类也能开启其动态性，其开启方式是**在属性或方法前加上@objc和dynamic。@objc是用来将Swift的API导出给Objective-C和Objective-C runtime使用的，如果你的类继承自Objective-c的类（如NSObject）将会自动被编译器插入@objc标识。加了@objc标识的方法、属性无法保证都会被运行时调用，因为Swift会做静态优化。要想完全被动态调用，必须使用dynamic修饰。使用dynamic修饰将会隐式的加上@objc标识。**



<br/>

***
<br/>


> <h1 id='前端'>前端</h1>

>### <h3 id='输入URL到加载过程发生了什么'>[输入URL到加载过程发生了什么](https://segmentfault.com/a/1190000006879700)</h3>

[从输入 URL 到页面展示到底发生了什么？看完吊打面试官！](https://zhuanlan.zhihu.com/p/133906695)

- 302和301的区别?以及原理

> - 301 和 302 的区别:

&emsp; 301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）——这是它们的共同点。

&emsp; 他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；

&emsp; 302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。SEO302好于301

<br/>

> - 重定向原因：

- 网站调整（如改变网页目录结构）；
- 网页被移到一个新地址；
- 网页扩展名改变(如应用需要把.php改成.Html或.shtml)。
- 这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个404页面错误信息，访问流量白白丧失；再者某些注册了多个域名的网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。


<br/>

>- 什么时候进行301或者302跳转呢？

&emsp; 当一个网站或者网页24—48小时内临时移动到一个新的位置，这时候就要进行302跳转，而使用301跳转的场景就是之前的网站因为某种原因需要移除掉，然后要到新的地址访问，是永久性的。

清晰明确而言：使用301跳转的大概场景如下：

&emsp; 域名到期不想续费（或者发现了更适合网站的域名），想换个域名。
在搜索引擎的搜索结果中出现了不带www的域名，而带www的域名却没有收录，这个时候可以用301重定向来告诉搜索引擎我们目标的域名是哪一个。
空间服务器不稳定，换空间的时候。



<br/>
<br/>


><h2 id=''></h2>



<br/>
<br/>


><h2 id=''></h2>



<br/>
<br/>


><h2 id=''></h2>



<br/>
<br/>


><h2 id=''></h2>





<br/>

***
<br/>


> <h1 id=''></h1>





<br/>

***
<br/>


> <h1 id=''></h1>





<br/>

***
<br/>


> <h1 id=''></h1>




<br/>

***
<br/>


> <h1 id=''></h1>













