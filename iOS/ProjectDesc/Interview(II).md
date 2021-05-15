- [**Swift基础**](#Swift基础)
	- [面向协议编程](#面向协议编程)
	- [协议中实用泛型](#协议中实用泛型)
	- [路由导航](#路由导航)
- [**组件化**](#组件化)
	- [路由导航](#路由导航)
- [**安全**](#安全)

	- [MD5](#MD5)
	- [防止反编译](#防止反编译)
- [**多线程**](#多线程)
	- [GCD控制线程数量](#GCD控制线程数量) 


<br/>

***
<br/>





> <h1 id = "Swift基础">Swift基础</h1>


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

***
<br/>


> <h1 id="组件化">组件化</h1>
[iOS组件化实践](https://www.jianshu.com/p/510ee1290ab4)

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


![<br/>](https://user-gold-cdn.xitu.io/2018/1/11/160e332699f100d4?imageView2/0/w/1280/h/960/ignore-error/1
)



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








