> <h2 id=""></h2>
- [**项目**](#项目)
	- [**NSBundle**](#NSBundle)
	- [ **类库**](#类库)
		- 	[IQKeyboardManager](#IQKeyboardManager)
		- 	[MJExtension](#MJExtension)
		- [SAMkeychain](#SAMkeychain)
		- [**RAC**](#RAC)
			- [**ReactiveCocoa**](#ReactiveCocoa)
			- [**ReactiveSwift**](#ReactiveSwift)
			- [ReactiveObjC](#ReactiveObjC)
			- [ReactiveObjCBridge](#ReactiveObjCBridge)
	- [**宏定义**](#宏定义)



<br/>

***
<br/>


> <h1 id="项目">项目</h1>

<br/>

> <h2 id="NSBundle">NSBundle</h2>

NSBundle介绍：它是一个单例类,用来加载资源.

&emsp; bundle是一个目录,其中包含了程序会使用到的资源. 这些资源包含了如图像,声音,编译好的代码,nib文件(用户也会把bundle称为plug-in). 对应bundle,cocoa提供了类NSBundle.

&emsp; 我们的程序是一个bundle. 在Finder中,一个应用程序看上去和其他文件没有什么区别. 但是实际上它是一个包含了nib文件,编译代码,以及其他资源的目录. 我们把这个目录叫做程序的main bundle

&emsp; bundle中的有些资源可以本地化.例如,对于foo.nib,我们可以有两个版本: 一个针对英语用户,一个针对法语用户. 在bundle中就会有两个子目录:English.lproj和French.lproj,我们把各自版本的foo.nib文件放到其中. 当程序需要加载foo.nib文件时,bundle会自动根据所设置的语言来加载. 我们会在16章再详细讨论本地化

&emsp; 通过使用下面的方法得到程序的main bundle

```
NSBundle *myBundle = [NSBundle mainBundle];
```


<br/>
<br/>

> <h2 id="类库">类库</h2>

<br/>


> <h3 id="IQKeyboardManager">IQKeyboardManager</h3>

```
//点击背景收回键盘
[IQKeyboardManager sharedManager].shouldResignOnTouchOutside = YES;
 ```




<br/>
<br/>


> <h3 id="MJExtension">MJExtension</h3>

**核心代码：**

```
//JSON|字典转模型的各类方法，最终都会调用
- mj_setKeyValues:(id)keyValues context:

//模型转JSON|字典的各类方法，最终都会调用
- (NSMutableDictionary *)mj_keyValuesWithKeys: ignoredKeys:

```

**性能方面:**

&emsp; 使用runtime动态生成类的属性信息，并通过缓存机制进行性能提优。

**容错方面**

&emsp; 在JSON|字典转模型最后赋值之前，会对值和属性的类型进行一致性的判断。如果不匹配，value会被置为nil，避免潜在的Crash风险。





<br/>
<br/>


> <h3 id="SAMkeychain">SAMkeychain</h3>

SAMkeychain可以用来储存用户的隐私信息。


<br/>

**储存数据**

```
+ (BOOL)setPasswordData:(NSData *)password forService:(NSString *)serviceName account:(NSString *)account;
+ (BOOL)setPasswordData:(NSData *)password forService:(NSString *)serviceName account:(NSString *)account error:(NSError **)error __attribute__((swift_error(none)));
+ (BOOL)setPassword:(NSString *)password forService:(NSString *)serviceName account:(NSString *)account;
+ (BOOL)setPassword:(NSString *)password forService:(NSString *)serviceName account:(NSString *)account error:(NSError **)error __attribute__((swift_error(none)));
```



<br/>

**获取数据**

```
+ (NSData *)passwordDataForService:(NSString *)serviceName account:(NSString *)account;
+ (NSData *)passwordDataForService:(NSString *)serviceName account:(NSString *)account error:(NSError **)error __attribute__((swift_error(none)));
+ (NSString *)passwordForService:(NSString *)serviceName account:(NSString *)account;
+ (NSString *)passwordForService:(NSString *)serviceName account:(NSString *)accou;
```

<br/>


**删除数据**

```
+ (BOOL)deletePasswordForService:(NSString *)serviceName account:(NSString *)account;
+ (BOOL)deletePasswordForService:(NSString *)serviceName account:(NSString *)account error:(NSError **)error __attribute__((swift_error(none)));
```



<br/>


**获取所有帐号信息**

```
+ (NSArray<NSDictionary<NSString *, id> *> *)allAccounts;
//NSArray *allAccounts = [SAMKeychain allAccounts];
+ (NSArray<NSDictionary<NSString *, id> *> *)allAccounts:(NSError *__autoreleasing *)error __attribute__((swift_error(none)));

+ (NSArray<NSDictionary<NSString *, id> *> *)accountsForService:(NSString *)serviceName;
+ (NSArray<NSDictionary<NSString *, id> *> *)accountsForService:(NSString *)serviceName error:(NSError *__autoreleasing *)error __attribute__((swift_error(none)));
//通过下面方法可以配置筛选的信息,
- (NSArray<NSDictionary<NSString *, id> *> *)fetchAll:(NSError **)error;
```



<br/>
<br/>

> <h2 id="RAC">RAC</h2>

&emsp; RAC 5.0 相比于 4.0 有了巨大的变化，不仅是受 swift 3.0 大升级的影响，RAC 对自身项目结构的也进行了大幅度的调整。这个调整就是将 RAC 拆分为四个库：**`ReactiveCocoa、ReactiveSwift、ReactiveObjC、ReactiveObjCBridge`**。

<br/>

**使用RAC项目里需要引入哪些类库？**

>如果你只是纯 swift 项目，你继续使用 ReactiveCocoa 。但是 RAC 依赖于 ReactiveSwift ，等于你引入了两个库。

>如果你的项目是纯 OC 项目，你需要使用的是 ReactiveObjC 。这个库里面包含原来 RAC 2 的全部代码。

>如果你的项目是 swift 和 OC 混编，你需要同时引用 ReactiveCocoa 和 ReactiveObjCBridge 。但是 ReactiveObjCBridge 依赖于 ReactiveObjC ，所以你就等于引入了 4 个库。




<br/>
<br/>

> <h3 id="ReactiveCocoa">ReactiveCocoa</h3>

&emsp; 现在的 RAC 注意力主要集中在 Swift 和 UI 层上，将原来一个基于 RAC 面向 UI 层的扩展库 Rex 合并进了 RAC 。

&emsp; RAC 3 和 4 的主要精力在围绕 Swift 重新打造一个响应式编程库。因为这部分的核心 API 已经很成熟，所以现在将重心放在为 AppKit 和 UIKit 提供一些更好用的扩展上。



<br/>


**ReactiveCocoa主要由以下四大核心组件构成**

- 信号源: [RACStream]及其子类最核心
- 订阅者: [RACSubscriber]的实现类及其子类
- 调度器: [RACScheduler]及其子类
- 清洁工: [RACDisposable]及其子类




<br/>
<br/>

> <h3 id="ReactiveSwift">ReactiveSwift</h3>

&emsp; 原来 RAC 中只和 Swift 平台相关的核心代码被单独抽取成了一个新框架：ReactiveSwift 。

&emsp; Swift 正在快速成长并且成长为一个跨平台的语言。把只和 Swift 相关的代码抽取出来后，ReactiveSwift 就可以在其他平台上被使用，而不只是局限在 CocoaTouch 和 Cocoa 中。






<br/>
<br/>

> <h3 id="ReactiveObjC">ReactiveObjC</h3>


&emsp; 在 RAC 3 和 4 中，RAC 也包含了 RAC 2 中的 OC 代码。现在这部分代码被移到了 ReactiveObjC 。

&emsp; 这样做的原因是因为两个库虽然有着一样的核心编程范式，实际上却是完全独立的两套 API 。实际的使用中，RAC 4 和 RAC 2 是完全不同的两组用户群，并且维护的团队其实也是两组。之前混在一个库里也增加了管理的复杂度。拆分出去后也可以更加自由的维护 ReactiveObjC 。


<br/>

**方法使用**

需要导入文件：**`#import <ReactiveObjC.h>`**

**1). createSignal**

```
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe 
```

测试：

```
- (void) testAction {
    
    RACSignal *siganal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        [subscriber sendNext:@"发送数据"];
        [subscriber sendCompleted];
        //数据发送完要调用sendCompleted，信号才会销毁
//        NSError *error = [NSError new];
//        [subscriber sendError:error];  //error会自动销毁信号
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"信号被销毁");
        }];
    }];
    
    [siganal subscribeNext:^(id  _Nullable x) {
        NSLog(@"订阅到的数据----->>>");
        NSLog(@"%@",x); //接受数据
    }];
    
}
```
打印：

```
2021-06-14 20:28:43.870935+0800 RACTest[13779:1641679] 订阅到的数据----->>>
2021-06-14 20:28:44.644955+0800 RACTest[13779:1641679] 发送数据
2021-06-14 20:28:46.491466+0800 RACTest[13779:1641679] 信号被销毁
```


<br/>

**2). retry**

`- (RACSignal *)retry:(NSInteger)retryCount`

```
- (void) testMethod2 {
    RACSignal *sourceSignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@1];
        [subscriber sendNext:@2];
        
        NSLog(@"++++++++");
        
        [subscriber sendError:[NSError errorWithDomain:@"domain" code:-1 userInfo:nil]];
        
        NSLog(@"--------");
        
        [subscriber sendNext:@3];
        
        NSLog(@">>>>>>>>>");

        
        [subscriber sendCompleted];
        return nil;
    }];
    
    RACSignal *retrySignal = [sourceSignal retry:2];
    [retrySignal subscribeNext:^(id  _Nullable x) {
        NSLog(@"value = %@", x);
    }];
}
```

打印：

```
2021-06-14 20:57:51.332699+0800 RACTest[14869:1670831] value = 1
2021-06-14 20:57:51.332922+0800 RACTest[14869:1670831] value = 2
2021-06-14 20:57:51.333007+0800 RACTest[14869:1670831] ++++++++
2021-06-14 20:57:51.333156+0800 RACTest[14869:1670831] --------
2021-06-14 20:57:51.333230+0800 RACTest[14869:1670831] >>>>>>>>>
2021-06-14 20:57:51.333764+0800 RACTest[14869:1670831] value = 1
2021-06-14 20:57:51.333852+0800 RACTest[14869:1670831] value = 2
2021-06-14 20:57:51.333919+0800 RACTest[14869:1670831] ++++++++
2021-06-14 20:57:51.333989+0800 RACTest[14869:1670831] --------
2021-06-14 20:57:51.334051+0800 RACTest[14869:1670831] >>>>>>>>>
2021-06-14 20:57:51.334181+0800 RACTest[14869:1670831] value = 1
2021-06-14 20:57:51.334253+0800 RACTest[14869:1670831] value = 2
2021-06-14 20:57:51.334328+0800 RACTest[14869:1670831] ++++++++
2021-06-14 20:57:51.334401+0800 RACTest[14869:1670831] --------
2021-06-14 20:57:51.334505+0800 RACTest[14869:1670831] >>>>>>>>>
```


&emsp; **-retry:** 实现与 **-repeat** 实现类似，基于 subscribeForever。-retry: 是内部维护一个 currentRetryCount 变量，当原始信号发送 sendError 时判断重试次数 currentRetryCount 是否小于 retryCount，若是则重试，如果重试依旧收到 sendError，超过 retryCount 之后就会停止重试。

&emsp; 如果原信号没有发生错误，那么原信号在发送结束，当原信号发送 sendCompleted，subscribeForever 也就接受了，所以 -retry: 操作对于没有任何error的信号 和 直接订阅原信号表现一样。



<br/>

**2). catch**

`- (RACSignal *)retry:(NSInteger)retryCount`

- 返回一个新信号N
- 当receiver发生error时，catchBlock中收到相应错误信息，并返回一个新的信号M，同时订阅M，自此以后新信号N中的数据便是M中的数据了
- 当receiver不发生error时，数据仍会通过N进行发送
- 还有一个精简的方法是- (RACSignal *)catchTo:(RACSignal *)signal，意思是相同的
	- 比如[A catchTo: B]表示，返回一个新信号，其中的数据是，如果A不发生错误就是A的数据，如果A发生错误，则数据边来源于B






<br/>
<br/>

> <h3 id="ReactiveObjCBridge">ReactiveObjCBridge</h3>

&emsp; 在把 Swift 和 OC 的库拆分之后问题来了，并不是所有的库都是纯 OC 和 Swift 的。有相当大一部分项目处于 OC 迁移到 Swift 过程中，其中可能使用 Swift 调用了 RAC 2 中基于 OC 写的 API。为了解决这部分用户的问题，所以有了 ReactiveObjCBridge 。




<br/>

***
<br/>


> <h2 id="宏定义">宏定义</h2>

<br/>

> <h3 id="clang">clang</h3>

&emsp; 在项目中有些时候不得已需要通过Selector的方式执行方法的时候，又不想因为找不到Selector而导致unrecognized selector sent to instance的崩溃问题，就可以将找不到的Selector识别为错误：

```
// clang诊断push
#pragma clang diagnostic push

// 将undeclared selector警告识别为error
#pragma clang diagnostic error "- Wundeclared-selector"

// clang诊断pop，如果不pop，下面写的代码，也会将undeclared selector识别为error
#pragma clang diagnostic pop

```

<br/>

**忽略警告**

&emsp; 同样如果我们希望一个警告在编译的时候，不被识别为警告，我们就可以对警告进行忽略，下面同样以undeclared selector警告为例：

```
#pragma clang diagnostic push

// 忽略undeclared selector的警告
#pragma clang diagnostic ignored "-Wundeclared-selector"
[self performSelector:@selector(noMethod) withObject:nil];

#pragma clang diagnostic pop
```

<br/>

**组合**

&emsp; 实际上，clang diagnostic并不只有上面的两种固定用法，error：警告识别为错误还是ignored：忽略警告都可以根据自己的需求进行选择。

&emsp; 而警告的类型也不止-Wundeclared-selector：undeclared selector一种，其他的比如：

```
-Wdeprecated-declarations      废弃的方法
-Wincompatible-pointer-types  指针类型不匹配
-Warc-retain-cycles                  Block的循环引用
-Wunused-variable                   未使用的变量
```

&emsp; 同样#pragma clang diagnostic也可以写成#pragma GCC diagnostic，clang和GCC都是前端编译器，不过clang是苹果专门为mac系统做的。









