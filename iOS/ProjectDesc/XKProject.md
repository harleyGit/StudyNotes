


> <h2 id=''></h2>
- [**文件模块**](#文件模块)
	- [路由导航](#路由导航)
	- [网页](#网页) 
		- [WebViewController](#HybridWebViewController)
- [**三方库**](#三方库)
	- [调试解决](#调试解决) 
		- [断点打印为null](#断点打印为null) 
	- [ReactObj](#ReactObj) 
		- [方法](#方法)
	- [FMDB](#FMDB) 



<br/>

***
<br/>

> <h2 id='文件模块'>文件模块</h2>

<br/>

> <h2 id='路由导航'>路由导航</h2>

> ModuleContainer：路由导航类


<br/>
<br/>



> <h2 id='网页'>网页</h2>

<br/>


> <h3 id='HybridWebViewController'>WebViewController</h3>

<br/>

> **方法**

```
//用来调用js中的方法，注意：javaScriptString 就是与前端约好的方法名
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^ _Nullable)(_Nullable id, NSError * _Nullable error))completionHandler;
```

案例：

```
//WKWebViewController.m文件
- (void)getLocation
{
    // 获取位置信息
    
    // 将结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"setLocation('%@')",@"广东省深圳市南山区学府路XXXX号"];
    //调用js里面的方法，这个要与前端写js的约定好： webView evaluateJavaScript:@"JS函数名称('参数1','参数2')" completionHandler:nil]来向JS发送消息
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
   
}



//index.html
 <script language="javascript">
	 function setLocation(location) {
		 //这个location就是：广东省深圳市南山区学府路XXXX号字符串
	    asyncAlert(location);
	    document.getElementById("returnValue").value = location;
	}

 </script>                
```


<br/>
<br/>











<br/>
<br/>



<br/>

***
<br/>

> <h1 id='调试解决'>调试解决</h1>

> <h2 id=''>断点打印为null</h2>

按照下图的提示进行解决：

![修改符号表](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/projectDesc0.png)

![环境配置](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/projectDesc1.png)




<br/>

***
<br/>

> <h1 id='三方库'>三方库</h1>

<br/>

> <h2 id='ReactObj'>ReactObj</h2>

<br/>

> <h3 id='方法'>方法</h3>

- [+(RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe](createSignal:)

- [+(RACSignal *)return:(id)value](#return)

- [-(__kindof RACStream *)map:(id (^)(id value))block](#map:)

- [-(RACSignal *)flattenMap:(__kindof RACSignal * _Nullable (^)(ValueType _Nullable value))block](#flattenMap:)

- [-(RACSignal *)doNext:(void (^)(id x))block](#doNext:)

- 


<br/>
<br/>

> <h3 id='类'>类</h3>
- [RACBehaviorSubject](#RACBehaviorSubject)
- [RACReplaySubject](#RACReplaySubject)
- [RACCommand](#RACCommand)



<br/>
<br/>

>#### <h4 id='createSignal:'>+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe</h4>

```
- (void) testMethod4 {
    //创建信号
    RACSignal *singalA = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        //发送信号
        [subscriber sendNext:@"1"];
        [subscriber sendNext:@(2)];
        [subscriber sendCompleted];
        
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"------this is RACDisposable---------");
        }];
    }];
    //订阅信号
    [singalA subscribeNext:^(id  _Nullable x) {
        NSLog(@"-------singalA value is %@-------",x);
    }];
}

```

打印：

```
2021-08-04 11:36:00.935943+0800 Test1[11228:186004] -------singalA value is 1-------
2021-08-04 11:36:19.339155+0800 Test1[11228:186004] -------singalA value is 2-------
2021-08-04 11:36:34.775731+0800 Test1[11228:186004] ------this is RACDisposable---------

```


<br/>
<br/>

>#### <h4 id='return'>[+(RACSignal *)return:(id)value](https://draveness.me/racsignal/)</h4>


```
+ (RACSignal *)return:(id)value {
	return [RACReturnSignal return:value];
}
```

&emsp; 该方法接受一个 NSObject 对象，并返回一个 RACSignal 的实例，它会将一个 UIKit 世界的对象 NSObject 转换成 ReactiveCocoa 中的 RACSignal。

&emsp; 而 RACReturnSignal 也仅仅是把 NSObject 对象包装一下，并没有做什么复杂的事情：

```
+ (RACSignal *)return:(id)value {
	RACReturnSignal *signal = [[self alloc] init];
	signal->_value = value;
	return signal;
}
```



<br/>

> <h4 id='map:'>- (__kindof RACStream *)map:(id (^)(id value))block </h4>

```
- (__kindof RACStream *)map:(id (^)(id value))block {
	NSCParameterAssert(block != nil);

	Class class = self.class;
	
	return [[self flattenMap:^(id value) {
		return [class return:block(value)];
	}] setNameWithFormat:@"[%@] -map:", self.name];
}
```

&emsp; Map将原信号中的内容映射成新的指定内容。
 通过对比，从map的实现方法中可以看出是基于flattenMap方法的一层封装，但同时又有不同之处。
 
 **案例Demo：**
 
 ```
 - (void) testMethod2 {
    [[self.inputField.rac_textSignal map:^id _Nullable(NSString * _Nullable value) {
        // 当源信号发出，就会调用这个block，修改源信号的内容
        // 返回值：就是处理完源信号的内容。
        return [NSString stringWithFormat:@"hello:%@",value];
    }] subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x); // hello: "x"
    }];
    
}
```

打印：

```
2021-08-03 17:41:58.501753+0800 Test1[21997:481962] hello:
2021-08-03 17:42:04.605039+0800 Test1[21997:481962] hello:
2021-08-03 17:42:07.073937+0800 Test1[21997:481962] hello:1
2021-08-03 17:42:07.429275+0800 Test1[21997:481962] hello:12
2021-08-03 17:42:07.736910+0800 Test1[21997:481962] hello:123
2021-08-03 17:42:08.019351+0800 Test1[21997:481962] hello:1234
```

 &emsp; **map:** 中的闭包参数value就是源信号的内容，直接拿到源信号的内容做处理，把处理好的内容，直接返回就好了，不用包装成信号，返回的值，就是映射的值。



<br/>

> <h4 id='flattenMap:'>- (RACSignal *)flattenMap:(__kindof RACSignal * _Nullable (^)(ValueType _Nullable value))block</h4>


```
- (void) testMethod1 {
    [[self.inputField.rac_textSignal flattenMap:^__kindof RACSignal * _Nullable(NSString * _Nullable value) {
        return  [RACSignal return:[NSString stringWithFormat:@"hello %@", value]];
    }] subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@",x); // hello "x"
    }];
    
    
}
```

打印：

```
2021-08-03 17:36:47.264375+0800 Test1[21384:461035] hello S1
2021-08-03 17:36:48.205486+0800 Test1[21384:461035] hello S12
2021-08-03 17:36:48.748797+0800 Test1[21384:461035] hello S123
2021-08-03 17:36:49.127409+0800 Test1[21384:461035] hello S1234
```

- flattenMap:中闭包参数value就是源信号的内容，拿到源信号的内容做处理；
- value被包装成RACReturnSignal信号，返回出去；


<br/>

**flatternMap和Map的区别**
- FlatternMap中的Block返回信号
- Map中的Block返回对象
- 开发中，如果信号发出的值不是信号，映射一般使用Map
- 开发中，如果信号发出的值是信号，映射一般使用FlatternMap



<br/>

> <h4 id='doNext:'>-(RACSignal *)doNext:(void (^)(id x))block</h4>

```
- (void) testMethod3 {
    
    [[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"<<<<<<<<< 123456"];
        [subscriber sendCompleted];
        return nil;
        
    }] doNext:^(id x) {
        //doNext：执行Next之前，会先执行这个Block： https://blog.csdn.net/s3590024/article/details/76071748
        //doNext: 为一个附加操作，在一个next事件发生时执行的逻辑，而该逻辑并不改变事件本身。
        // 执行 [subscriber sendNext:@"hi"] 之前会调用这个 Block
        NSLog(@"================= 执行 %@", x);
        
    }] doCompleted:^{
        // 执行 [subscriber sendCompleted] 之前会调用这 Block
        NSLog(@">>>>>>>>>>>>>>doCompleted");
        
    }] subscribeNext:^(id x) {
        NSLog(@"%@", x);
    }];
}
```

打印值：

```
2021-08-03 19:27:07.193319+0800 Test1[24609:580637] ================= 执行 <<<<<<<<< 123456
2021-08-03 19:27:15.727391+0800 Test1[24609:580637] <<<<<<<<< 123456
2021-08-03 19:27:28.381956+0800 Test1[24609:580637] >>>>>>>>>>>>>>doCompleted
```



<br/>

> <h4 id='RACBehaviorSubject'>RACBehaviorSubject</h4>

&emsp; `RACBehaviorSubject`在订阅时会向订阅者发送最新的消息。


&emsp;  `RACBehaviorSubject` 最重要的特性就是在订阅时，向最新的订阅者发送之前的消息，这是通过覆写 -subscribe: 方法实现的。

案例Demo：

```
//RACBehaviorSubject *subject = [RACBehaviorSubject behaviorSubjectWithDefaultValue:@0];
RACBehaviorSubject *subject = [RACBehaviorSubject subject];

[subject subscribeNext:^(id  _Nullable x) {
    NSLog(@"1st Sub: %@", x);
}];
[subject sendNext:@1];

[subject subscribeNext:^(id  _Nullable x) {
    NSLog(@"2nd Sub: %@", x);
}];
[subject sendNext:@2];

[subject subscribeNext:^(id  _Nullable x) {
    NSLog(@"3rd Sub: %@", x);
}];
[subject sendNext:@3];
[subject sendCompleted];
```


打印：

```
//使用了 +behaviorSubjectWithDefaultValue:
//2021-08-04 19:04:56.631576+0800 Test1[18063:523610] 1st Sub: 0
2021-08-04 19:00:46.531011+0800 Test1[17933:516457] 1st Sub: (null)
2021-08-04 19:00:53.163748+0800 Test1[17933:516457] 1st Sub: 1


2021-08-04 19:00:59.294161+0800 Test1[17933:516457] 2nd Sub: 1
2021-08-04 19:01:08.451922+0800 Test1[17933:516457] 1st Sub: 2
2021-08-04 19:01:10.061648+0800 Test1[17933:516457] 2nd Sub: 2


2021-08-04 19:01:14.737775+0800 Test1[17933:516457] 3rd Sub: 2
2021-08-04 19:01:20.714692+0800 Test1[17933:516457] 1st Sub: 3
2021-08-04 19:01:21.335019+0800 Test1[17933:516457] 2nd Sub: 3
2021-08-04 19:01:21.977903+0800 Test1[17933:516457] 3rd Sub: 3
```


<br/>

> <h4 id='RACReplaySubject'>RACReplaySubject</h4>

&emsp; `RACReplaySubject`在订阅之后可以重新发送之前的所有消息序列。



<br/>

> <h4 id='RACCommand'>RACCommand</h4>

**`RACCommand:`**RAC中用于处理事件的类，可以把事件如何处理,事件中的数据如何传递，包装到这个类中，他可以很方便的监控事件的执行过程。


**RACCommand使用步骤:**
>- 创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock
- 在signalBlock中，创建RACSignal，并且作为signalBlock的返回值
- 执行命令 - (RACSignal *)execute:(id)input
    
>- RACCommand使用注意:
	- signalBlock必须要返回一个信号，不能传nil.
	- 如果不想要传递信号，直接创建空的信号[RACSignal empty];
	- RACCommand中信号如果数据传递完，必须调用[subscriber sendCompleted]，这时命令才会执行完毕，否则永远处于执行中。
	- RACCommand需要被强引用，否则接收不到RACCommand中的信号，因此RACCommand中的信号是延迟发送的。

>- RACCommand设计思想：内部signalBlock为什么要返回一个信号，这个信号有什么用。
	- 在RAC开发中，通常会把网络请求封装到RACCommand，直接执行某个RACCommand就能发送请求。
	- 当RACCommand内部请求到数据的时候，需要把请求的数据传递给外界，这时候就需要通过signalBlock返回的信号传递了。

>- 如何拿到RACCommand中返回信号发出的数据。
	- RACCommand有个执行信号源executionSignals，这个是signal of signals(信号的信号),意思是信号发出的数据是信号，不是普通的类型。
	- 订阅executionSignals就能拿到RACCommand中返回的信号，然后订阅signalBlock返回的信号，就能获取发出的值。

>- 监听当前命令是否正在执行executing

>- 使用场景,监听按钮点击，网络请求

<br/>

```
- (void) testMethod7 {
    // 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        NSLog(@"执行命令");
        NSLog(@"执行命令--> %@", input);
        // 创建空信号,必须返回信号
        // return [RACSignal empty];
        // 2.创建信号,用来传递数据
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"请求数据"];
            // 注意：数据传递完，最好调用sendCompleted，这时命令才执行完毕。
            [subscriber sendCompleted];
            return nil;
        }];
    }];
    
    // 强引用命令，不要被销毁，否则接收不到数据
    //_loginCommand = command;
    
    // 3.订阅RACCommand中的信号
    [command.executionSignals subscribeNext:^(id x) {
        //订阅executionSignals就能拿到RACCommand中返回的信号，然后订阅signalBlock返回的信号，就能获取发出的值。
        [x subscribeNext:^(id x) {
            NSLog(@"🍎 %@",x);
        }];
    }];
    
    // RAC高级用法
    // switchToLatest:用于signal of signals，获取signal of signals发出的最新信号,也就是可以直接拿到RACCommand中的信号
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {
        NSLog(@"🍊 %@",x);
    }];
    
    // 4.监听命令是否执行完毕,默认会来一次，可以直接跳过，skip表示跳过第一次信号。
    [[command.executing skip:0] subscribeNext:^(id x) {
        // executing：判断当前的block是否在执行，执行完之后会返回NO
        NSLog(@"是否执行： %@", x);
        if ([x boolValue] == YES) {
            // 正在执行
            NSLog(@"正在执行");
        }else{
            // 执行完成
            NSLog(@"执行完成");
        }
    }];
    
    // 5.执行命令（控制器里执行此句代码）
    [command execute:@"✈️ 执行命令！！"];
}

```

打印：

```
2021-08-09 11:40:53.519286+0800 Test1[4783:171682] 是否执行： 0
2021-08-09 11:40:58.701714+0800 Test1[4783:171682] 执行完成
2021-08-09 11:41:04.110605+0800 Test1[4783:171682] 执行命令
2021-08-09 11:41:04.954396+0800 Test1[4783:171682] 执行命令--> ✈️ 执行命令！！
2021-08-09 11:41:15.616669+0800 Test1[4783:171682] 是否执行： 1
2021-08-09 11:41:19.114076+0800 Test1[4783:171682] 正在执行
2021-08-09 11:41:24.337670+0800 Test1[4783:171682] 🍎 请求数据
2021-08-09 11:41:25.591968+0800 Test1[4783:171682] 🍊 请求数据
2021-08-09 11:41:28.640741+0800 Test1[4783:171682] 是否执行： 0
2021-08-09 11:41:29.558142+0800 Test1[4783:171682] 执行完成
```



<br/>

> <h4 id=''></h4>



<br/>

> <h4 id=''></h4>





<br/>

***
<br/>




> <h2 id='FMDB'>FMDB</h2>

```
[FMDatabaseQueue databaseQueueWithPath:path];
```

&emsp; 在App中保持一个FMDatabaseQueue的实例，并在所有的线程中都只使用这一个实例。

&emsp; `FMDatabaseQueue`虽然看似一个队列，实际上它本身并不是，它通过内部创建一个Serial的dispatch_queue_t来处理通过`inDatabase`和`inTransaction`传入的Blocks，所以当我们在主线程（或者后台）调用inDatabase或者inTransaction时，代码实际上是同步的。

&emsp; FMDatabaseQueue这么设计的目的是让我们避免发生并发访问数据库的问题，因为对数据库的访问可能是随机的（在任何时候）、不同线程间（不同的网络回调等）的请求。内置一个Serial队列后，FMDatabaseQueue就变成线程安全了，所有的数据库访问都是同步执行，而且这比使用@synchronized或NSLock要高效得多。

&emsp; 但是这么一来就有了一个问题：如果后台在执行大量的更新，而主线程也需要访问数据库，虽然要访问的数据量很少，但是在后台执行完之前，还是会阻塞主线程。
对此，robertmryan给出了一些想法：

- 如果你是在后台使用的inDatabase来执行更新，可以考虑换成inTransaction，后者比前者更新起来快很多，特别是在更新量比较大的时候（比如更新1000条或10000条）。

- 拆解你的更新数据量，如果有300条，可以分10次、每次更新30条。当然有时不能这么做，因为你可能通过网络请求回来的数据，你希望一次性、完整地写入到数据库中，虽然有局限性，不过这确实能很好地减少每个Block占用数据库的时间。

- 上面两点可以改善问题，但是问题依然是存在的，在大多数时候，你应该把从主线程调用inDatabase和inTransaction放在异步里：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    [self.databaseQueue inDatabase:^(FMDatabase *db) {
        //do something...
    }];
});
```

&emsp; 这种方式能解决不依赖于数据库返回的结果的情况，如果对返回结果有依赖，就需要考虑UI上的体验了，如加一个UIActivityIndicatorView。



<br/>

***
<br/>

> <h2 id=''></h2>



<br/>

***
<br/>

> <h2 id=''></h2>





