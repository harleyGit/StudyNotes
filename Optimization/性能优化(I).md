
- **性能优化概要**
- **内存**
	- 	内核
	- 	栈
	- 	堆
	- 	静态存储区
	- 	内存泄漏
	- 	内存溢出
	- 	指针置为nil和对象release的作用
	- 	[malloc与free本质(内存的释放与回收)](https://www.cnblogs.com/shiweihappy/p/4246372.html)
	- [静态存储区、栈区、堆区的区别](https://blog.csdn.net/u010977122/article/details/53099425)
	- 	BSS段
	- 	Data段
	- 	text代码段
	- 	变量区别
	- 	全局变量（成员变量）和局部变量的区别
	- 	全局变量（成员变量）和静态变量的区别
	- 	retain
	- 	weak
- [**性能优化**](https://www.cnblogs.com/guohai-stronger/p/10430106.html)
- [**UI流畅**](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
- **引用计数**
- **优化角度**
	- 解码
	- 性能优化
- **NSTimer内存泄漏**
	- 	视图控制器判断
	- 	运行时解决
	- 	方法重定向解决
- **内存检测**
	- 	静态检测
	- 	几种内存泄漏情况
	- 	动态检测(instrument检测，MLeakFinder[第三方方法])
	- 	析构方法打印



<br/>

***
<br/>

># 性能优化概要

**iOS性能优化，主要围绕以下问题解决的：**
- 内存
	- 内存布局
	- retain
	- weak
- Runloop
	- NSTimer
	- 面试-Runloop
- 界面
	- 内存泄露
	- TableView优化


<br/>

***
<br/>

># 内存

代码的文件是可执行的二进制文件，在二进制文件中，这些文件呢如下图分布：


![Code 文件分布](https://upload-images.jianshu.io/upload_images/2959789-4b13a1a887e60282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

- **`内核`**

>&emsp;  内核是操作系统最关键的组成部分。内核的功能是负责接触底层，所以大部分会用到C语音进行编写的，有的甚至使用到汇编语言。iOS的核心是XNU内核。 

>&emsp;  XNU内核是混合内核，其核心是叫Mach的微内核，其中Mach中亦是消息传递机制，但是使用的是指针形式传递。因为大部分的服务都在XNU内核中。Mach没有昂贵的复制操作，只用指针就可以完成的消息传递。


<br/>

- **`栈`**

>栈主要存放局部变量和函数参数等相关的变量，如果超出其作用域后也会自动释放。栈区：是向低字节扩展的数据结构，也是一块连续的内存区域。


<br/>

- **`堆（heap）`**

>堆区存放new，alloc等关键字创造的对象，我们在之前常说的内存管理管理的也是这部分内存。堆区：是向高地址扩展的数据结构，不连续的内存区域，会造成大量的碎片。


<br/>

- **`静态存储区`**

&emsp;  内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。它主要存放静态数据、全局数据和常量。

<br/>

- **`内存泄漏：`**

&emsp;  是指申请的内存空间使用完毕之后未回收。一次内存泄露危害可以忽略，但若一直泄漏，无论有多少内存，迟早都会被占用光，最终导致程序crash。

<br/>

- **`内存溢出：`**

&emsp;  是指程序在申请内存时，没有足够的内存空间供其使用。通俗理解就是内存不够用了，通常在运行大型应用或游戏时，应用或游戏所需要的内存远远超出了你主机内安装的内存所承受大小，就叫内存溢出。最终导致机器重启或者程序crash。

<br/>

- **`野指针：`**

&emsp;  指向一个已删除的对象或者受限内存区域的指针，[野指针and空指针](https://www.cnblogs.com/mjios/archive/2013/04/22/3034788.html)，

<br/>

- **`指针置为nil和对象release的作用`**

&emsp; `nil`就是把一个对象的指针置为空，只是切断了指针与内存中对象的联系；

&emsp; `release`是通知内存释放这个对象，但是在IOS中其实也不会立马释放内存，而是将内存计数器剪去1，直到计数器变为0，才会释放掉内存，所以release的目的是为了释放内存，而self.object = nil，是清空指针。


 




<br/>

**`BSS段`**
>BSS段存放未初始化的全局变量以及静态变量，一旦初始化就会从BSS段去掉，转到数据段中。


<br/>

- **`Data段`**

>Data段存储已经初始化好的静态变量和全局变量，以及常量数据，直到程序结束之后才会被立即收回


<br/>

- **` text代码段`**

>text段是用来存放程序代码执行的一块内存区域。这一块内存区域的大小在程序运行前就已经确定，通常也是只读属性。


<br/>

- **`变量区别`**

变量分为：`全局变量（成员变量）、局部变量、实例属性和静态变量以及类属性`

**变量按作用范围可以分为全局变量和局部变量，其中全局变量也就是成员变量。成员变量按调用的方式可以分为类属性和实例属性。类属性是用static修饰的成员变量，也就是静态变量。实例属性是没有用static修饰的成员变量，也叫作非静态变量。**

![<br/>](https://upload-images.jianshu.io/upload_images/2959789-622d60286031ccbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; 如果局部变量和全局变量的名字是一样的，局部变量的作用范围区域内全局变量就会被隐藏；但是如果在局部变量的范围内想要访问成员变量，必须要使用关键字self来引用全局变量（成员变量）。

<br/>

- **`全局变量（成员变量）和局部变量的区别`**
	- 内存中位置不同：全局变量（成员变量）在堆内存，全局变量（成员变量）属于对象，对象进入堆内存；局部变量属于方法，方法进入栈内存
	- 生命周期不同：全局变量（成员变量）随着对象的创建而存在的，对象消失也随之消失；局部变量随着方法调用而存在，方法调用完毕而消失
	- 初始化不同：全局变量（成员变量）有默认的初始化值；局部变量是没有默认初始化的，必须定义，然后才能使用。

<br/>

- **`全局变量（成员变量）和静态变量的区别：`**
	- 内存位置不同：静态变量也就是类属性，存放在静态区；成员变量存放在堆内存
	- 调用方式不同：静态变量可以通过对象调用，也可以通过类名调用；成员变量就只能用对象名调用


<br/>

- **`retain`**

&emsp;  对于retain，如果经过taggerPointer修饰过的，就直接return，如果不是的话，就调用当前的retain-rootRetain方法 。需要关注当前引用计数什么时候加1-----通过sideTable方法，加一个偏移量refcntStorage。这就是内部实现的过程。

拓展：retain与copy有什么区别？

**copy**：建立索引计数为1的对象，然后释放对象；copy建立一个相同的对象，如果一个NSString对象，假如地址为0x1111，内容为@"hello"，通过Copy到另一个对象之后，地址为0x2322，内容也相同，而新的对象retain为1，旧的对象是不会发生变化；

**retain**：retain到另外一个对象之后，地址是不会变化的，地址也为0x1111，实质上是建立一个指针，也就是指针拷贝，内容也是相同的，retain值会加1。

<br/>

- **`weak`**
 [**weak实现原理**](https://www.cnblogs.com/guohai-stronger/p/10161870.html)



<br/>

***
<br/>

># 引用计数
sideTable: hashTable 引用计数，弱引用，锁
是一个哈希表，



<br/>

***
<br/>

># 优化角度

- **解码**
	- CPU 把就算好的内容提交给GPU;
	- GPU 进行变换、合成、渲染；

<br/>
<br/>

- **性能优化：**
	-  透明度：消耗CPU，若有透明度，CPU需要计算；
	-  圆角、阴影离屏渲染会消耗CPU，这是因为有太多的layer导致不停地切换上下文消耗时间、性能；
	-  重绘；
	-  图片解压：无论是从磁盘加载还是从网络下载的图片都是没经过解压，在渲染的时候解压，在主线程解压；


自动打包的时候使用shell脚本。


<br/>

***
<br/>

># NSTimer内存泄漏

[**循环引用**](https://www.cnblogs.com/guohai-stronger/p/9011806.html)

```
__weak typeof(self) weakSelf = self;
_timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:self selector:@selector(fire) userInfo:nil repeats:YES];
```
上面的代码依然不可以解决循环引用。`__weak typeof(self) weakSelf = self;`这个weakSelf指向的地址就是当前self指向的地址；如果再使用`strong--->weakSelf`，也会使self连带指针retain（加1）操作，没有办法避免当前VC引用计数加1。如下图：

![NSTimer 循环引用原理](https://upload-images.jianshu.io/upload_images/2959789-5074788d077f5df4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- **`NSTimer循环引用解决，目前有以下几种:`**
	- 类方法
	- GCD方法
	- weakProxy


<br/>
<br/>

> **视图控制器判断**

DEMO:
viewController -----Push------>LeakedViewController


**`LeakedViewController.m`**

```
- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");
}


- (void)viewDidLoad {
    [super viewDidLoad];
    
    //会造成内存泄漏，因为这个NSTimer在主线程中运行，它强引用了self，只要它运行主控制器就不会释放掉，造成了内存的泄漏。
    //用weak也不行，因为在他的内部使用了strong强引用
    self.timer = [NSTimer timerWithTimeInterval:1.0f target:self selector:@selector(fire) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSDefaultRunLoopMode];
}

- (void)fire {
    NSLog(@"fire 调用了");
}



//当向父VC添加子VC之后，该方法不会被自动调用，需要显示调用告诉编译器已经完成添加（事实上不调用该方法也不会有问题，不太明白）;
//从父VC移除子VC之后，该方法会自动调用，传入的参数为nil,所以不需要显示调用。
- (void)didMoveToParentViewController:(UIViewController *)parent {
    if (parent == nil) {
        [self.timer invalidate];
        self.timer = nil;
    }
}

//可以解决内存泄漏问题，调用dealloc析构方法

```

输出：

```
2019-06-08 23:54:20.937609+0800 Genealogy[7703:553726] fire 调用了
2019-06-08 23:54:21.938562+0800 Genealogy[7703:553726] fire 调用了
2019-06-08 23:54:22.179312+0800 Genealogy[7703:553726] LeakedViewController 调用dealloc 方法
```



<br/>
<br/>

> **运行时解决**

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //不让当前timer强制引用当前的vc,self持有_target
    _target = [NSObject new];
    class_addMethod([_target class], @selector(fire), (IMP)fireImp, "v@:");
    
    //self.timer持有了_target,timer没有强引用self
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_target selector:@selector(fire) userInfo:nil repeats:YES];
}

void fireImp(id self, SEL _cmd){
    NSLog(@"fireImp 调用了 fire ");
}

- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");


    [self.timer invalidate];
    self.timer = nil;
}

```
输出：

```
2019-06-09 10:58:13.603251+0800 Genealogy[10606:696072] fireImp 调用了 fire 
2019-06-09 10:58:14.603689+0800 Genealogy[10606:696072] fireImp 调用了 fire 
2019-06-09 10:58:14.684340+0800 Genealogy[10606:696072] GBuildFamilyController 调用dealloc 方法
```



<br/>
<br/>

> **方法重定向解决**

下面我们主要看使用 [NSProxy](https://www.jianshu.com/p/923f119333d8)类来解决：

**`GProxy.h 文件`**

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface GProxy : NSProxy

@property(nonatomic, weak)id target;

@end

NS_ASSUME_NONNULL_END
```

**`GProxy.m 文件`**

```
#import "GProxy.h"
#import <objc/runtime.h>

@implementation GProxy

//方法签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [self.target methodSignatureForSelector:sel];
}

//消息转发
- (void)forwardInvocation:(NSInvocation *)invocation {
    [invocation invokeWithTarget:self.target];
}

@end

```

**`GBuildFamilyController.m 文件`**

```
- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");
    
    [self.timer invalidate];
    self.timer = nil;
}


- (void)viewDidLoad {
    [super viewDidLoad];
    
    _gproxy = [GProxy alloc];
    //_gproxy.target弱引用了self,self可以正常走dealloc析构方法
    _gproxy.target = self;
    
    //self.timer强引用了_gproxy.target
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_gproxy.target selector:@selector(fire) userInfo:nil repeats:YES];
}

- (void)fire {
    NSLog(@"fire 调用了");
}
```
输出：

`2019-06-09 13:14:14.188451+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:15.188988+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:16.187918+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:17.111693+0800 Genealogy[11849:767419] GBuildFamilyController 调用dealloc 方法`


<br/>


![<br/>](https://upload-images.jianshu.io/upload_images/2959789-296953927f40b615.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

使用weakProxy解决循环引用的原因是：

&emsp;  weakProxy是利用runtime消息转发机制来断开NSTimer对象与视图的强引用关系。初始NSTimer时把触发事件的target对象替换成了另一个单独的对象，紧接着对象中NSTimer的SEL方法触发时让这个方法在当前视图中实现。
![NSProxy 循环引用解决方法](https://upload-images.jianshu.io/upload_images/2959789-67a0868a6729b9b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

`Runloop`

[RunLoop 底层原理](https://www.cnblogs.com/guohai-stronger/p/9190220.html)




<br/>

***
<br/>

>#  内存检测


<br/>

- **`静态检测`**
【Product】-> 【Analyze】


<br/>

- **`几种内存泄漏情况`**

```
//block 传空值
 [self sceneO3:nil];

//CG C 语言需要我们手动释放
- (void) sceneO1 {
    CGPathRef shadowPah = CGPathCreateWithRect(CGRectMake(0, 2, 0, 20), NULL);
}

//文件操作，打开文件流但没关闭文件流会发生内存泄漏
- (void) sceneO2 {
    FILE *f;
    f = fopen("info.plist", "f");
}


- (void) sceneO3:(void(^)(void))callback {
    //需要对这个block进行判空，否则会发生内存泄漏
    callback();
}

- (NSArray *) sceneO4 {
    NSString *str = nil;
    //array中插入空值，会发生内存泄漏
    return @[@"sceneo1", str, @"scence02"];
}
```

<br/>

![设置编译时检测内存泄漏](https://upload-images.jianshu.io/upload_images/2959789-e8ed173365fbc179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

- **`动态检测(instrument检测，MLeakFinder[第三方方法])`**



- **`析构方法打印`**





<br/>

***
<br/>

