iOS性能优化，主要围绕以下问题解决的：
- 内存
内存布局
retain
weak

- Runloop
NSTimer
面试-Runloop
- 界面
内存泄露
TableView优化


<br/>
***
<br/>

># 内存
代码的文件是可执行的二进制文件，在二进制文件中，这些文件呢如下图分布：![Code 文件分布](https://upload-images.jianshu.io/upload_images/2959789-4b13a1a887e60282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`内核`**
>&emsp;  内核是操作系统最关键的组成部分。内核的功能是负责接触底层，所以大部分会用到C语音进行编写的，有的甚至使用到汇编语言。iOS的核心是XNU内核。 
&emsp;  XNU内核是混合内核，其核心是叫Mach的微内核，其中Mach中亦是消息传递机制，但是使用的是指针形式传递。因为大部分的服务都在XNU内核中。Mach没有昂贵的复制操作，只用指针就可以完成的消息传递。


<br/>
**`栈`**
>栈主要存放局部变量和函数参数等相关的变量，如果超出其作用域后也会自动释放。栈区：是向低字节扩展的数据结构，也是一块连续的内存区域。


<br/>
**`堆（heap）`**
>堆区存放new，alloc等关键字创造的对象，我们在之前常说的内存管理管理的也是这部分内存。堆区：是向高地址扩展的数据结构，不连续的内存区域，会造成大量的碎片。


<br/>
**`BSS段`**
>BSS段存放未初始化的全局变量以及静态变量，一旦初始化就会从BSS段去掉，转到数据段中。


<br/>
**`Data段`**
Data段存储已经初始化好的静态变量和全局变量，以及常量数据，直到程序结束之后才会被立即收回


<br/>
**` text段`**
text段是用来存放程序代码执行的一块内存区域。这一块内存区域的大小在程序运行前就已经确定，通常也是只读属性。


<br/>
**`变量区别`**
变量分为：`全局变量、成员变量、局部变量、实例属性和静态变量以及类属性`

**变量按作用范围可以分为全局变量和局部变量，其中全局变量也就是成员变量。成员变量按调用的方式可以分为类属性和实例属性。类属性是用static修饰的成员变量，也就是静态变量。实例属性是没有用static修饰的成员变量，也叫作非静态变量。**

![变量划分](https://upload-images.jianshu.io/upload_images/2959789-622d60286031ccbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; 如果局部变量和全局变量的名字是一样的，局部变量的作用范围区域内全局变量就会被隐藏；但是如果在局部变量的范围内想要访问成员变量，必须要使用关键字self来引用全局变量（成员变量）。

`全局变量（成员变量）和局部变量的区别`
- 内存中位置不同：全局变量（成员变量）在堆内存，全局变量（成员变量）属于对象，对象进入堆内存；局部变量属于方法，方法进入栈内存
- 生命周期不同：全局变量（成员变量）随着对象的创建而存在的，对象消失也随之消失；局部变量随着方法调用而存在，方法调用完毕而消失
- 初始化不同：全局变量（成员变量）有默认的初始化值；局部变量是没有默认初始化的，必须定义，然后才能使用。

`全局变量（成员变量）和静态变量的区别：`
- 内存位置不同：静态变量也就是类属性，存放在静态区；成员变量存放在堆内存
- 调用方式不同：静态变量可以通过对象调用，也可以通过类名调用；成员变量就只能用对象名调用


<br/>
**`retain`**

&emsp;  对于retain，如果经过taggerPointer修饰过的，就直接return，如果不是的话，就调用当前的retain-rootRetain方法 。需要关注当前引用计数什么时候加1-----通过sideTable方法，加一个偏移量refcntStorage。这就是内部实现的过程。

拓展：retain与copy有什么区别？

**copy**：建立索引计数为1的对象，然后释放对象；copy建立一个相同的对象，如果一个NSString对象，假如地址为0x1111，内容为@"hello"，通过Copy到另一个对象之后，地址为0x2322，内容也相同，而新的对象retain为1，旧的对象是不会发生变化；

**retain**：retain到另外一个对象之后，地址是不会变化的，地址也为0x1111，实质上是建立一个指针，也就是指针拷贝，内容也是相同的，retain值会加1。

<br/>
**`weak`**
 weak[实现原理](https://www.cnblogs.com/guohai-stronger/p/10161870.html)



<br/>
***
<br/>
># RunLoop
[循环引用](https://www.cnblogs.com/guohai-stronger/p/9011806.html)

```
__weak typeof(self) weakSelf = self;
_timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:self selector:@selector(fire) userInfo:nil repeats:YES];
```
上面的代码依然不可以解决循环引用。`__weak typeof(self) weakSelf = self;`这个weakSelf指向的地址就是当前self指向的地址；如果再使用`strong--->weakSelf`[图片上传中...(image.png-ce9a3-1571548996782-0)]
，也会使self连带指针retain（加1）操作，没有办法避免当前VC引用计数加1。如下图：

![NSTimer 循环引用原理](https://upload-images.jianshu.io/upload_images/2959789-5074788d077f5df4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`NSTimer循环引用的解决方法，目前有以下几种:`
- 类方法
- GCD方法
- weakProxy

下面我们主要看使用 [NSProxy](https://www.jianshu.com/p/923f119333d8)类来解决：
`WeakProxy 类`
```
#import <Foundation/Foundation.h>
NS_ASSUME_NONNULL_BEGIN

@interface WeakProxy : NSProxy
@property(nonatomic , weak)id target;

@end

NS_ASSUME_NONNULL_END


#import "WeakProxy.h"
#import <objc/runtime.h>

@implementation WeakProxy

- (void)forwardInvocation:(NSInvocation *)invocation{
    [self.target forwardInvocation:invocation];
}

- (nullable NSMethodSignature *)methodSignatureForSelector:(SEL)sel{
    return [self.target methodSignatureForSelector:sel];
}

@end
```

使用这个类
```
#import "ViewController.h"
#import "WeakProxy.h"

@interface ViewController ()
@property (strong, nonatomic) NSTimer *timer;
@property(nonatomic,strong)WeakProxy *weakProxy;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    _weakProxy = [WeakProxy alloc];
    _weakProxy.target = self;
    _timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_weakProxy selector:@selector(fire) userInfo:nil repeats:YES];

}

- (void)fire{
    NSLog(@"fire");
}

- (void)dealloc{
    [self.timer invalidate];
    self.timer = nil;
}

@end
```
使用weakProxy解决循环引用的原因是：
&emsp;  weakProxy是利用runtime消息转发机制来断开NSTimer对象与视图的强引用关系。初始NSTimer时把触发事件的target对象替换成了另一个单独的对象，紧接着对象中NSTimer的SEL方法触发时让这个方法在当前视图中实现。
![NSProxy 循环引用解决方法](https://upload-images.jianshu.io/upload_images/2959789-67a0868a6729b9b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
`Runloop`
[RunLoop 底层原理](https://www.cnblogs.com/guohai-stronger/p/9190220.html)


<br/>
***
<br/>



[US](https://www.cnblogs.com/guohai-stronger/p/10430106.html)
[](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

