># NSTimer
&emsp; **`NSTimer`**是不准确的，当程序执行的时候，遇到cpu忙碌的时候，NSTimer会被放到一边不执行，就会造成该执行的事件不执行，会造成事件的叠加。

&emsp; 所以说NSTimer通常会用来做一些有一定时间跨度的时间处理。

解决办法：（1.多线程，2.CADdisplay link）不好意思还没学到，学到在更新……
[三种精准计时](https://www.cnblogs.com/XYQ-208910/p/6590829.html)


<br/>
***
<br/>


># RunLoop
&emsp;  RunLoop在OC中有两种获取方式：`[NSRunLoop currentRunLoop] `或者C的：`CFRunLoopRef`。

&emsp; ` NSRunLoop` 是基于 `CFRunLoopRef` 的OC封装，提供了面向对象的 API，但`不是线程安全`的，`CFRunLoopRef` 是在 `CoreFoundation` 框架内的，它提供了纯 C 函数的 API，是线程安全的，`CoreFoundation`是开源的([CoreFoundation 源码地址](https://link.jianshu.com?t=https://opensource.apple.com/tarballs/CF/))


&emsp;  每一个线程都有唯一的一个与之对应的`RunLoop`对象。`RunLoop`保存现在一个全局的Dictionary里面，线程作为key，RunLoop作为value。线程创建并没有RunLoop对象，`会在第一次获取的时候，系统创建(获取的时候系统才开始创建，不需要我们自己创建)`。线程结束时销毁`RunLoop`。


**`CF的内存管理(Core Foundation)`**
```
凡是带有Create、Copy、Retain等字眼的函数，创建出来的对象，都需要在最后做一次release,比如 CFRunLoopObserverCreate

release函数：CFRelease(对象);
```






&emsp;  `RunLoop`是一个相对抽象的概念，在程序运行过程中循环做一些事情，主要应用于：定时器（Timer）、PerformSelector、GCD Async To Main Queue、事件详情、手势识别、界面刷新、网络请求、AutoreleasePool。

&emsp;  一个线程的消息事件处理都是依赖于NSRunLoop来驱动,所以要知道线程正在调用什么方法,就需要从NSRunLoop来入手。


<br/>
&emsp;  `RunLoop`只处理两种源：`输入源、时间源`。而输入源又可以分为：`NSPort、自定义源、performSelector,我们常用搭到的`performSelector`方法有：
```
// 主线程
performSelectorOnMainThread:withObject:waitUntilDone:
performSelectorOnMainThread:withObject:waitUntilDone:modes:
/// 指定线程
performSelector:onThread:withObject:waitUntilDone:
performSelector:onThread:withObject:waitUntilDone:modes:
/// 针对当前线程
performSelector:withObject:afterDelay:         
performSelector:withObject:afterDelay:inModes:
/// 取消，在当前线程，和上面两个方法对应
cancelPreviousPerformRequestsWithTarget:
cancelPreviousPerformRequestsWithTarget:selector:object:
```




<br/>
***
<br/>


># NSRunLoop
&emsp;  ` NSRunLoop` 是基于 `CFRunLoopRef` 的OC封装，提供了面向对象的 API，但`不是线程安全`的。

<br/>
#`RunLoop 在主线程`
```
NSTimer * timer = [NSTimer timerWithTimeInterval:1.f repeats:YES block:^(NSTimer * _Nonnull timer) {
           static int count = 0;
           [NSThread sleepForTimeInterval:1];
           //休息一秒钟，模拟耗时操作
           NSLog(@"%s - %d",__func__,count++);
       }];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
![RunLoop 加在主线程中](https://upload-images.jianshu.io/upload_images/2959789-9bd54428c78a928f.gif?imageMogr2/auto-orient/strip)

&emsp;  当把NSTimer直接加入主线程中时，我们拖动UITextView上下滑动时会发现，有卡顿现象，当我们把RunLoop注销掉时，再次拖动没有出现卡顿现象。



<br/>
#`RunLoop 在子线程`
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSTimer * timer = [NSTimer timerWithTimeInterval:1.f repeats:YES block:^(NSTimer * _Nonnull timer) {
               static int count = 0;
               [NSThread sleepForTimeInterval:1];
               //休息一秒钟，模拟耗时操作
               NSLog(@"%s - %d",__func__,count++);
           }];
        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
    });
```
&emsp;   通过观察，我们在拖动UITextView控件时，我们发现子线程中的RunLoop并没有打印。
&emsp;   这是因为当进入block的时候，先创建了timer，并且也把timer也把timer加入到runloop中，但是很重要的一点子线程中Runloop不会自动运行，需要手动运行，因为这里没有运行Runloop，所以timer就被释放掉了，所以导致了啥都没有。



<br/>
#`RunLoop 在子线程手动开启`
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSTimer * timer = [NSTimer timerWithTimeInterval:1.f repeats:YES block:^(NSTimer * _Nonnull timer) {
               static int count = 0;
               [NSThread sleepForTimeInterval:1];
               //休息一秒钟，模拟耗时操作
               NSLog(@"%s - %d",__func__,count++);
           }];
        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
        
        //子线程需要手动开启Runloop
        [[NSRunLoop currentRunLoop] run];
        NSLog(@"当前线程: %@", [NSThread currentThread]);
    });
```
![子线程手动开启RunLoop](https://upload-images.jianshu.io/upload_images/2959789-1f926c39389ff696.gif?imageMogr2/auto-orient/strip)



<br/>

#`子线程实现常驻线程`
```
//运行runloop
[[NSRunLoop currentRunLoop] run]; 

//指定runloop在NSDefaultRunLoopMode模式下，设置开始时间,开启成功会返回YES
[[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];  

//运行runloop直到一个时间
[[NSRunLoop currentRunLoop] runUntilDate:[NSDate distantFuture]]; 
```
&emsp;  这样子就可以实现在子线程处理耗时操作，并且常驻线程了。






<br/>
***
<br/>


># RunLoop的 C 语言的的结构体 CFRunLoopRef

&emsp;  `CFRunLoopRef` 是在 `CoreFoundation` 框架内的，它提供了纯 C 函数的 API，是线程安全的。

**`CFRunLoop的结构`**
```

//RunLoop 的运行模式
typedef struct __CFRunLoopMode *CFRunLoopModeRef


typedef struct CF_BRIDGED_MUTABLE_TYPE(id) __CFRunLoop * CFRunLoopRef;

//RunLoop要处理的事件源
typedef struct CF_BRIDGED_MUTABLE_TYPE(id) __CFRunLoopSource * CFRunLoopSourceRef;

//RunLoop的观察者(监听者)
typedef struct CF_BRIDGED_MUTABLE_TYPE(id) __CFRunLoopObserver * CFRunLoopObserverRef;

//Timer 事件
typedef struct CF_BRIDGED_MUTABLE_TYPE(NSTimer) __CFRunLoopTimer * CFRunLoopTimerRef;

```








<br/>
***
<br/>

>#RunLoop Mode

![RunLoop的内部结构图](https://upload-images.jianshu.io/upload_images/2959789-a2691bb0e7f205fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




`RunLoop Mode`可以视为事件的管家，一个Mode管理着各种事件，它的结构如下：
```
struct __CFRunLoopMode {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;	/* must have the run loop locked before locking this */
    CFStringRef _name;
    Boolean _stopped;
    char _padding[3];
    CFMutableSetRef _sources0;      //处理一些触摸事件和 perform Selectors方法等。
    CFMutableSetRef _sources1;      //基于Port的线程间的通信
    CFMutableArrayRef _observers;   //监听器(通知)，监听RunLoop的一些状态的
    CFMutableArrayRef _timers;      //处理定时器任务
    CFMutableDictionaryRef _portToV1SourceMap;  //字典， key是mach_port_t,value是CFRunLoopSourceRef
    __CFPortSet _portSet;   //保存所有需要监听的port，比如_weakeUpPort, _timerPort都保存在这个数组中
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
```

&emsp;  一个CFRunLoopMode对象有一个_name，若干 _source0、_source1、_timers、observer和若干port，`可见事件都是由CFRunLoopMode在管理，而RunLoop管理CFRunLoopMode `。

&emsp; 从源码的结构体可以很容易看出，Runloop总是运行在某种特定的`CFRunLoopModeRef`下(每次运行 `__CFRunLoopRun()` 函数时必须指定Mode)。而通过CFRunloopRef对应结构体的定义可以很容易知道每种Runloop都可以包含若干个Mode，每个Mode又包含 `Source/Timer/Observer` 。每次调用Runloop的主函数`__CFRunLoopRun()`时必须指定一种Mode，这个Mode称为` _currentMode`，当切换Mode时必须退出当前Mode，然后重新进入Runloop以保证不同Mode的Source/Timer/Observer互不影响。


下图是苹果文档中提到的5种Mode，第2个、第3个在iOS中没有暴露出来。

- `NSRunLoopCommonModes` 实际上是一个 Mode 的集合，默认包括 `NSDefaultRunLoopMode` 和 `NSEventTrackingRunLoopMode`（注意：并不是说Runloop会运行在kCFRunLoopCommonModes这种模式下，而是相当于分别注册了 NSDefaultRunLoopMode和 UITrackingRunLoopMode。当然你也可以通过调用CFRunLoopAddCommonMode()方法将自定义Mode放到 kCFRunLoopCommonModes组合）

- `NSDefaultRunLoopMode` 默认模式，一般处理timer\网络等事件；
- `UITrackingRunLoopMode` UI模式，专门处理UI事件；

![5 种RunLoop Mode](https://upload-images.jianshu.io/upload_images/2959789-9624d88d118abea5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>

>#RunLoop Source
`RunLoop Source `分为Source、Observer、Timer三种，他们统称为ModeItem。

按照官方文档分类，Source 分类：
- `Port-Based Sources` (跟其他线程交互，或通过线程内核发送的消息)
- `Custom Input Sources` (自动已输入源，几乎不用)
- `Cocoa Perform Selector Sources` (比如performSelector等方法)


<br/>
按照函数调用栈，Source分类：
- `Source0`: 非基于Port的,处理内部app事件，如：UIEvent；
- `Soruce1`： 基于Port的，通过内核和其他线程通信、接收、分发系统事件的，主动唤醒runloop线程；


<br/>

**` CFRunLoopSource `**

&emsp; `CFRunLoopSource` 是对input sources的抽象。CFRunLoopSource分`source0`和`source1`两个版本，它的结构如下：
```
struct __CFRunLoopSource {
    CFRuntimeBase _base;
    uint32_t _bits; //用于标记Signaled状态，source0只有在被标记为Signaled状态，才会被处理
    pthread_mutex_t _lock;
    CFIndex _order;			/* immutable */
    CFMutableBagRef _runLoops;
    union {//source0 和 source1 两个版本
	CFRunLoopSourceContext version0;	/* immutable, except invalidation */
        CFRunLoopSourceContext1 version1;	/* immutable, except invalidation */
    } _context;
};
```


<br/>
`source0 的详解`
&emsp;  `source0`是App内部事件，由App自己管理的`UIEvent、CFSocket`都是source0。当一个source0事件准备执行的时候，必须要先把它标记为signal状态，以下是source0的结构体：
```
typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
    Boolean	(*equal)(const void *info1, const void *info2);
    CFHashCode	(*hash)(const void *info);
    void	(*schedule)(void *info, CFRunLoopRef rl, CFRunLoopMode mode);
    void	(*cancel)(void *info, CFRunLoopRef rl, CFRunLoopMode mode);
    void	(*perform)(void *info);
} CFRunLoopSourceContext;
```
&emsp;  `source0`是非基于Port的。只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 `CFRunLoopSourceSignal(source)`，将这个 Source 标记为待处理，然后手动调用 `CFRunLoopWakeUp(runloop)` 来唤醒 RunLoop，让其处理这个事件。


<br/>
`source1 的详解`
&emsp;  `source1`由 `RunLoop` 和内核管理，`source1` 带有`mach_port_t`，可以接收内核消息并触发回调，以下是source1的结构体:
```
typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
    Boolean	(*equal)(const void *info1, const void *info2);
    CFHashCode	(*hash)(const void *info);
#if TARGET_OS_OSX || TARGET_OS_IPHONE
    mach_port_t	(*getPort)(void *info);
    void *	(*perform)(void *msg, CFIndex size, CFAllocatorRef allocator, void *info);
#else
    void *	(*getPort)(void *info);
    void	(*perform)(void *info);
#endif
} CFRunLoopSourceContext1;
```
&emsp;  `source1`除了包含回调指针外包含一个`mach port`，`source1`可以监听系统端口和通过内核和其他线程通信，接收、分发系统事件，它能够主动唤醒RunLoop(由操作系统内核进行管理，例如CFMessagePort消息)。官方也指出可以自定义Source，因此对于CFRunLoopSourceRef来说它更像一种协议，框架已经默认定义了两种实现，如果有必要开发人员也可以自定义，详细情况可以查看[官方文档](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fcontent%2Fdocumentation%2FCocoa%2FConceptual%2FMultithreading%2FRunLoopManagement%2FRunLoopManagement.html)。







<br/>
***
<br/>

># CFRunLoopObserverRef
&emsp;  `CFRunLoopObserver`是观察者，可以观察RunLoop的各种状态，并抛出回调。其结构体如下：
```
struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;		/* immutable */
    CFIndex _order;			/* immutable */
    CFRunLoopObserverCallBack _callout;	/* immutable */
    CFRunLoopObserverContext _context;	/* immutable, except invalidation */
};
```

<br/>
`CFRunLoopObserver 可观察的6种状态如下：`
```
/* Run Loop Observer Activities *///RunLoop 的状态的变化
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),   //即将进入 loop
    kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理 timer
    kCFRunLoopBeforeSources = (1UL << 2),   //即将处理 source
    kCFRunLoopBeforeWaiting = (1UL << 5),   //即将 sleep
    kCFRunLoopAfterWaiting = (1UL << 6),    //刚被唤醒，退出 sleep
    kCFRunLoopExit = (1UL << 7),            //即将退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU   //全部的活动
};
```

<br/>
&emsp;  `Runloop` 通过监控 Source 来决定有没有任务要做，除此之外，我们还可以用 `Runloop Observer` 来监控 Runloop 本身的状态。` Runloop Observer `可以监控上面的 Runloop 事件，具体流程如下图。
![RunLoop 执行逻辑图](https://upload-images.jianshu.io/upload_images/2959789-cff1d145c64a7a29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>

># CFRunLoopTimerRef

&emsp;  `CFRunLoopTimer`是定时器，可以在设定的时间点抛出回调，它的结构体如下：
```
struct __CFRunLoopTimer {
    CFRuntimeBase _base;
    uint16_t _bits; //标记fire状态
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;      //添加该timer的runloop
    CFMutableSetRef _rlModes;   //存放所有 包含该timer的 mode的 modeName，意味着一个timer可能会在多个mode中存在
    CFAbsoluteTime _nextFireDate;
    CFTimeInterval _interval;		/* immutable */ //理想时间间隔
    CFTimeInterval _tolerance;          /* mutable *///时间偏差
    uint64_t _fireTSR;			/* TSR units */
    CFIndex _order;			/* immutable */
    CFRunLoopTimerCallBack _callout;	/* immutable */
    CFRunLoopTimerContext _context;	/* immutable, except invalidation */
};
```
&emsp;  所以CFRunLoopTimer具有以下特性：
- CFRunLoopTimer 是定时器，可以在设定的时间点抛出回调
- CFRunLoopTimer和NSTimer是toll-free bridged的，可以相互转换


<br/>
***
<br/>



![RunLoop 运行流程](https://upload-images.jianshu.io/upload_images/2959789-dfca2e040b3b6ee1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


RunLoop内部函数作用：

- `Souces0`:  处理一些触摸事件和 perform Selectors方法等;
- `Sources1`:  基于Port的线程间的通信;
- `timer`:  处理定时器任务;
- `observers`:   监听器，监听RunLoop的一些状态的。

&emsp;  RunLoop的运行机制就是运用一个do while循环，根据返回值来保持是否持续来进行运行处理某些事件。还有当无任务时，会进入休眠来，防止CPU的空转，等待消息的唤醒。


![RunLoop 运行流程图](https://upload-images.jianshu.io/upload_images/2959789-6a13be4bc8f12c99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;  先判断`source0`是否为空,若为空退出,然后判断`source1`是否为空,若为空退出,然后判断是否`有timer`,所以`runloop`如果要跑起来,必须有`source`或者`timer`的其中一个。Source共在2种类型：`Source0和Source1`，`Source0只包含了一个回调（函数指针）`，它并不能主动触发事件。使用时，你需要先调用 `CFRunLoopSourceSignal(rlms)`方法将这个Source标记为待处理，然后手动调用`CFRunLoopWakeUp(rl)`方法来唤醒RunLoop，让其处理这个事件。
&emsp; `Source1`包含了一个`mach_port`和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种类型的Source能主动唤醒RunLoop的线程。


<br/>

![RunLoop 工作流程](https://upload-images.jianshu.io/upload_images/2959789-68332ffa5a1717b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`上图显示了线程的输入源`
	A：基于端口的输入源（Port Sources）
	B：自定义输入源（Custom Sources）
	C：Cocoa执行Selector的源（"performSelector...方法" Sources）
	D：定时源（Timer Sources ）

`线程针对上面不同的输入源，有不同的处理机制`
	A：handlePort－－－处理基于端口的输入源
	B：customSrc－－－处理用户自定义输入源
	C：mySelector－－－处理Selector的源
	D：timerFired－－－处理定时源

&emsp;  `注：`线程除了处理输入源，Run Loops也会生成关于Run Loop行为的通知（notification）。Run Loop观察者（Run-Loop Observers）可以收到这些通知，并在线程上面使用他们来作额为的处理。


<br/>
**`CFRunLoop 监控卡顿流程`**
```
int32_t __CFRunLoopRun()
{
    // 通知即将进入runloop
    __CFRunLoopDoObservers(KCFRunLoopEntry);
    
    do
    {
        // 通知将要处理timer和source
        __CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
        __CFRunLoopDoObservers(kCFRunLoopBeforeSources);
        
        // 处理非延迟的主线程调用
        __CFRunLoopDoBlocks();
        // 处理UIEvent事件
        __CFRunLoopDoSource0();
        
        // GCD dispatch main queue
        CheckIfExistMessagesInMainDispatchQueue();
        
        // 即将进入休眠
        __CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);
        
        // 等待内核mach_msg事件
        mach_port_t wakeUpPort = SleepAndWaitForWakingUpPorts();
        
        // Zzz...
        
        // 从等待中醒来
        __CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
        
        // 处理因timer的唤醒
        if (wakeUpPort == timerPort)
            __CFRunLoopDoTimers();
        
        // 处理异步方法唤醒,如dispatch_async
        else if (wakeUpPort == mainDispatchQueuePort)
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()
            
        // UI刷新,动画显示
        else
            __CFRunLoopDoSource1();
        
        // 再次确保是否有同步的方法需要调用
        __CFRunLoopDoBlocks();
        
    } while (!stop && !timeout);
    
    // 通知即将退出runloop
    __CFRunLoopDoObservers(CFRunLoopExit);
}
```

<br/>
***
<br/>
[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
