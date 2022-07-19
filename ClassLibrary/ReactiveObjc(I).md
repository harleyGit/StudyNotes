> <h2 id=''></h2>
- [**ReactiveObjc下载**](#ReactiveObjc下载)
- [**类**](#类)
	- [RACSignal(冷信号)](#RACSignal)
	-  [RACSubject(热信号)](#RACSubject)
	-  [RACReplaySubject](#RACReplaySubject)
	-  [RACCommand](#RACCommand)
		-  [属性](#属性)
	-  [RACMulticastConnection(多播)](#RACMulticastConnection(多播))
	-  [RACScheduler](#RACScheduler)
	-  [RACUnit](#RACUnit)
	-  [RACEvent](#RACEvent)
	-  
- [属性](#属性2)
- [方法](#方法)
	- [rac_signalForSelector(替代代理)](#rac_signalForSelector)
	- [rac_valuesAndChangesForKeyPath](#rac_valuesAndChangesForKeyPath)
	- [rac_signalForControlEvents](#rac_signalForControlEvents)
- **参考资料**
	- [ReactiveCocoa之RACCommand使用(五)](https://blog.csdn.net/majiakun1/article/details/52937770)
	- [操作符](https://juejin.cn/post/6934238669578436615)
	- [ReactiveObjC快速入门](https://juejin.cn/post/6953808004307222564#heading-10)
	- [MVVM on iOS (ReactiveX)](https://sunxiaoshan.medium.com/mvvm-on-ios-reactivex-417b012b5d81)
 

<br/>

***
<br/>


># <h2 id='ReactiveObjc下载'>ReactiveObjc下载</h2>

&emsp; 首先你的Mac 设备需要下载好CocoaPod、配置好Git，这个我就不细说了，在百度上很多。

```
① 终端输入一下指令
//搜索ReactiveObjc的最新版本 
pod search ReactiveObjc

②把最新版本，CocoaPod考入到Podfile文件中
pod 'ReactiveObjC', '~> 3.1.0'

③ 在终端 切换到当前项目文件中
cd  /Users/huanggang/Documents/HGSWB

④ 执行下载更新指令，在下载之前最好把 Podfile.lock文件删除，若没有就不用删除了
pod update --verbose
```

`完成安装以后，在需要使用的项目文件中导入文件：#import <ReactiveObjC.h>`


<br/>

***
<br/>


> <h1 id='类'>类</h1>

- 响应式编程思想(KVO)
- 链式编程思想(block做参数, block返回对象本身)
- ReactiveCocoa主要由以下四大核心组件构成
	- 信号源: [RACStream]及其子类最核心
	- 订阅者: [RACSubscriber]的实现类及其子类
	- 调度器: [RACScheduler]及其子类
	- 清洁工: [RACDisposable]及其子类



<br/>

> <h2 id='RACSignal'>RACSignal(冷信号)</h2>

- 只有数据改变时, 信号内部才会发出数据, 它本身不具备发送信号的能力, 而是交给内部一个订阅者去发出
- 默认一个信号都是冷信号, 也就是值改变了,也不会触发, 只有订阅了这个信号, 这个信号才会变成热信号, 值改变才会触发
- 不要分开订阅next, error, completed这样的话会把前面的订阅者清除掉, 应该使用直接返回这三种的订阅方法
- **RACSignal底层实现大致原理：**
	- 创建信号: `+ (RACSignal<ValueType> *)createSignal:(RACDisposable * _Nullable (^)(id<RACSubscriber> subscriber))didSubscribe `	，首先把**didSubscribe**保存到信号中，还不会触发;
	- 信号被订阅: `- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock`, 也就是调用signal的subscribeNext:方法
		- subscribeNext内部会创建订阅者subscriber，并且把nextBlock保存到subscriber中。
		- subscribeNext内部会调用siganl的didSubscribe
	- siganl的didSubscribe中调用[subscriber sendNext:@1];
	- sendNext底层其实就是执行subscriber的nextBlock


**Demo**

```
- (void) testMethod0 {
    //1.创建信号
    RACSignal *siganal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        // block调用时刻：每当有订阅者订阅信号，就会调用block。
                
        // 2.发送信号
        [subscriber sendNext:@"发送数据"];
        
        //数据发送完要调用sendCompleted，内部会自动调用[RACDisposable disposable]取消订阅信号。
        [subscriber sendCompleted];
//        NSError *error = [NSError new];
//        [subscriber sendError:error];  //error会自动销毁信号
        return [RACDisposable disposableWithBlock:^{
            // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。
                        
            // 执行完Block后，当前信号就不在被订阅了。
            NSLog(@"信号被销毁");
        }];
    }];
    
    // 3.订阅信号,才会激活信号
    [siganal subscribeNext:^(id  _Nullable x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"%@",x); //接受数据
    }];
    
    
}
```

打印：

```
2021-08-16 13:39:12.056700+0800 Test1[12895:210417] 发送数据
2021-08-16 13:39:17.833233+0800 Test1[12895:210417] 信号被销毁
```


<br/>

- **RACSubscriber(订阅者), 用于发送信号, 这是一个协议不是一个类**
- **RACDisposable用于取消订阅或者清理资源, 当信号发送完成或发送错误的时候, 就会自动触发它**
- **RACSubject: 遵循了RACsubscriber协议的RACSignal,所以自己是信号, 也可以订阅其他信号**
- **RACReplaySubject: RACSubject的子类, 重复提供信号, 先把发送过的信号缓存起来, 再次订阅时直接发送缓存的值**

<br/>
<br/>

> <h2 id='RACSubject'>RACSubject(热信号)</h2>

&emsp; `RACSubject（热信号）`继承于`RACSignal`，一个`subject`可认为是一个信号，你可以手动控制`sendNext、sendError、sendCompleted`事件。可以帮助你将非RAC桥接到RAC上。
热信号特点：

- 可变；

- 主动；

- 可以桥接RAC和非RAC;


<br/>

&emsp; RACSubject 对外公开的接口只有两个，一个是创建一个热**`信号对象`**，另一个是发送消息的方法。

```
/// Returns a new subject.
+ (instancetype)subject;

// Redeclaration of the RACSubscriber method. Made in order to specify a generic type.
- (void)sendNext:(nullable ValueType)value;
```

创建热信号的接口：
`+ (instancetype)subject;`

&emsp; 这个方法创建的时候，会初始化两个私有属性，一个是subscribers数组 、另一个是disposable。
- subscribers：数组里包含了所有的当前订阅者，在self的同步操作时候会使用它遍历所属的订阅者。
- disposable：包含接收者对其他信号的所有订阅。主要是用于取消订阅和清理资源的操作。

<br/>

**订阅信号：** 只是把订阅者保存起来，并且订阅者的nextBlock已经赋值了

`- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock error:(void (^)(NSError *error))errorBlock;`

**发送信号：** 遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock

`- (void)sendNext:(nullable ValueType)value;`

重新定义了RACSubscriber协议的sendNext方法。

<br/>

**RACSubject与RACCommand的区别：**

![RACSubject 与 RACCommand 的区别 图示](https://upload-images.jianshu.io/upload_images/2959789-6056772ea7ea6433.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp; **RACSubject只能单向发送事件，发送者将事件发送出去让接收者接收事件后进行处理。所以，RACSubject可代替代理，被监听者可利用subject发送事件，监听者接收事件然后进行相应的监听处理，不过，事件的传递方向是单向的。**

&emsp; **对于RACCommand，我觉得用HTTP请求能够更形象地说明其原理，HTTP请求是由请求者向服务器发送一条网络请求，而服务器接收到请求然后经过相应处理后再向请求者返回处理过后的结果，数据流是双向的，RACCommand正是如此，让我想让某个部件进行某种会产生结果的操作时，利用RACCommand向此部件发送执行事件，部件接收到执行事件后进行相应操作处理并也通过RACCommand将操作结果回调到上层，使得事件得以双向流通。**


```
- (void)testReactive {
    RACSubject *searchSubjuct = [RACSubject subject];
    
    [searchSubjuct subscribeNext:^(id  _Nullable x) {
        NSLog(@"订阅信号： %@", x);
    }];
    
    [searchSubjuct sendNext:@"welcome to my home"];
}
```

打印结果：

```
2018-11-21 11:26:31.507642+0800 HGSWB[3233:107079] 订阅信号： welcome to my home
```




<br/>

***
<br/>


> <h2 id='RACReplaySubject'>RACReplaySubject</h2>

**RACReplaySubject使用步骤:**
- 创建信号 [RACReplaySubject subject]，跟RACSiganl不一样，创建信号时没有block。
- 可以先订阅信号，也可以先发送信号。
	- 订阅信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
	- 发送信号 sendNext:(id)value

<br/>

**RACReplaySubject:底层实现和RACSubject的区别：**
- 调用sendNext发送信号，把值保存起来，然后遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock。
- 调用subscribeNext订阅信号，遍历保存的所有值，一个一个调用订阅者的nextBlock
- 如果想当一个信号被订阅，就重复播放之前所有值，需要先发送信号，在订阅信号。
- 也就是先保存值，在订阅值。

```
- (void)testMethod13 {
    // 1.创建信号
    RACReplaySubject *replaySubject = [RACReplaySubject subject];

    // 2.发送信号
    [replaySubject sendNext:@1];
    [replaySubject sendNext:@2];

    // 3.订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第一个订阅者接收到的数据%@",x);
    }];

    // 订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第二个订阅者接收到的数据%@",x);
    }];
}
```

打印：

```
2021-08-16 19:29:41.423055+0800 Test1[23710:536357] 第一个订阅者接收到的数据1
2021-08-16 19:29:43.493856+0800 Test1[23710:536357] 第一个订阅者接收到的数据2
2021-08-16 19:29:48.676178+0800 Test1[23710:536357] 第二个订阅者接收到的数据1
2021-08-16 19:29:49.399736+0800 Test1[23710:536357] 第二个订阅者接收到的数据2
```


<br/>

- **RACTuple: 类似NSArray, 用来包装值**

- **RACSequence: RAC中的集合类, 用于代替NSArray, NSDictionary, 可以使用它来快速来便利数组和字典**

- **注意RACSequence会开启子线程, 要在completed做U更新**

- **RACCommand: RAC中用于处理事件的类, 可以把事件如何处理, 事件中的数据如何传递, 包装到这个类中, 可以很方便的监控事件的执行过程**




<br/>

***
<br/>


> <h2 id='RACCommand'>RACCommand</h2>

![What is RACCommand](https://upload-images.jianshu.io/upload_images/2959789-3af4f09a4bb3dbec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**`两种创建方式`**

```
//①
//通过传进一个用于构建RACSignal的block参数来初始化RACCommand，而block中的参数input为执行command时传入的数据，
//另外，创建出的signal可在里面完成一些数据操作，如网络请求，本地数据库读写等等
- (instancetype)initWithSignalBlock:(RACSignal<ValueType> * (^)(InputType _Nullable input))signalBlock;

//使用规则：
//#a.创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock

//#b.在signalBlock中，创建RACSignal，并且作为signalBlock的返回值

//#c.执行命令 - (RACSignal *)execute:(id)input


//②
//传进一个能传递BOOL事件的RACSignal，这个signal的作用相当于过滤，当传递的布尔事件为真值时，command能够执行，反之则不行。
- (instancetype)initWithEnabled:(nullable RACSignal<NSNumber *> *)enabledSignal signalBlock:(RACSignal<ValueType> * (^)(InputType _Nullable input))signalBlock;

```

<br/>


> <h3 id='属性'>属性</h3>

 &emsp; RACCommand 通常用来表示某个Action的执行，比如点击Button。它有几个比较重要的属性：executionSignals、errors、executing。 
  
 **a、`executionSignals:`** 是signal of signals(信号中的信号)，如果直接subscribe的话会得到一个signal，而不是我们想要的value，所以一般会配合switchToLatest。 
  
 **b、`errors:`**  跟正常的signal不一样，RACCommand的错误不是通过sendError来实现的，而是通过errors属性传递出来的。 
  
 **c、`executing:`**  表示该command当前是否正在执行。 
 

<br/>

**`Demo`**

```
- (void)testReactive {
    //注意：当前命令内部发送数据完成，一定要主动发送完成
    // 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        // block调用：执行命令的时候就会调用
        NSLog(@"%@", input);
        // 这里的返回值不允许为nil
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            // 发送数据
            [subscriber sendNext:@"执行命令产生的数据"];
            
            // *** 发送完成 **
            [subscriber sendCompleted];
            return nil;
        }];  
    }];
    // 监听事件有没有完成
    [command.executing subscribeNext:^(id x) {
        if ([x boolValue] == YES) { // 正在执行
            NSLog(@"当前正在执行%@", x);
        }else {
            // 执行完成/没有执行
            NSLog(@"执行完成/没有执行");
        }
    }];
    
    // 2.执行命令
    [command execute:@"welcome  to  my home!!"];
    
}
```

打印：

```
2021-08-11 11:18:58.336069+0800 Test1[17449:167012] 执行完成/没有执行
2021-08-11 11:18:58.336497+0800 Test1[17449:167012] welcome  to  my home!!
2021-08-11 11:19:04.124016+0800 Test1[17449:167012] 当前正在执行1
2021-08-11 11:19:04.124378+0800 Test1[17449:167012] 执行完成/没有执行
```


<br/>


**Demo2**

**`RACCommand使用注意点`**
- 必须返回一个信号
- executionSignals:RACCommand返回的信号时信号中的信号, 有两种方式可以获取最新的信号
	- switchToLatest:获取内部的最新信号(被动执行)
	- execute:获取内部信号(这个是主动执行)
- executing:用来判断是否正在执行, 第一次不准确需要跳过
- 一定要记得sendCompleted, 否则永远不会执行完成
     
     

```
- (void)testMethod11 {
    
    
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
        NSLog(@"<<<<<< 🍎 input: %@", input);
        return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
            NSLog(@"====== 🍎");
            //input为传入的参数
            [subscriber sendNext:@"发送数据"];
            //NSError *error = [NSError new];
            //[subscriber sendError:error];  发送error会自动调用
            //每次都会创建一个信号，发送完数据，此处记得销毁消息，不然会一直处于执行中。
            [subscriber sendCompleted];
            return [RACDisposable disposableWithBlock:^{
                NSLog(@"信号被销毁");
            }];
        }];
    }];
    
    [command.executionSignals subscribeNext:^(id  _Nullable x) {
        NSLog(@"获取一个信号---%@",x);
        [x subscribeNext:^(id x) {
            
            NSLog(@"%@",x);
        }];
    }];
    
    [command.executing subscribeNext:^(NSNumber * _Nullable x) {
        NSLog(@"监听完成---%@",x); //x为监听状态 0代表执行完毕，1代表执行中
    }];
    [command.errors subscribeNext:^(NSError * _Nullable x) {
        NSLog(@"错误"); // 接受错误信息
    }];
    [command.executionSignals.switchToLatest subscribeNext:^(id  _Nullable x) {
        NSLog(@"接受信号中的信号x为数据:%@",x);//x为发送的数据
    }];
    [command execute:@"11111"]; //调用command`
    
    
}

//switchToLatest的测试
- (void) testMethod10 {
    RACSubject *signalOfSignals = [RACSubject subject];
    RACSubject *signal = [RACSubject subject];
    // 获取信号中信号最近发出信号，订阅最近发出的信号。
    // 注意switchToLatest：只能用于信号中的信号
    [signalOfSignals.switchToLatest subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
    [signalOfSignals sendNext:signal];
    [signal sendNext:@1];
    [signal sendNext:@"123"];
}
```

打印：

```
2021-08-11 11:22:09.768168+0800 Test1[17530:171230] 监听完成---0
2021-08-11 11:22:09.768341+0800 Test1[17530:171230] <<<<<< 🍎 input: 11111
2021-08-11 11:22:09.768804+0800 Test1[17530:171230] 监听完成---1
2021-08-11 11:22:09.768873+0800 Test1[17530:171230] 获取一个信号---<RACDynamicSignal: 0x60000093d400> name: 
2021-08-11 11:22:09.769019+0800 Test1[17530:171230] ====== 🍎
2021-08-11 11:22:09.769130+0800 Test1[17530:171230] 发送数据
2021-08-11 11:22:09.769229+0800 Test1[17530:171230] 接受信号中的信号x为数据:发送数据
2021-08-11 11:22:09.769329+0800 Test1[17530:171230] 信号被销毁
2021-08-11 11:22:09.769433+0800 Test1[17530:171230] 监听完成---0
```


<br/>

`普通做法`

```
- (void)testReactive {
    // RACCommand: 处理事件
    // 不能返回空的信号
    // 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
        
        //block调用，执行命令的时候就会调用
        NSLog(@"%@",input); // input 为执行命令传进来的参数
        // 这里的返回值不允许为nil ,如果不想要传递信号，直接创建空的信号[RACSignal empty];
        return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
            [subscriber sendNext:@"执行命令产生数据"];
            //操作完成后发送完成消息以表示其执行完了,否则永远处于执行中
            [subscriber sendCompleted];
            return nil;
        }];
    }];
    
    
    // 如何拿到执行命令中产生的数据呢？
    // 订阅命令内部的信号
    // ** 方式一：直接订阅执行命令返回的信号
    
    // 2.执行命令
    RACSignal *signal = [command execute:@"welcome to my Home"];// 这里其实用到的是replaySubject 可以先发送命令再订阅
    
    // 在这里就可以订阅信号了
    [signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@", x);
    }];
}

```

打印结果：

```
2018-11-21 17:09:12.896750+0800 HGSWB[10066:392674] welcome to my Home
2018-11-21 17:09:41.202123+0800 HGSWB[10066:392674] 执行命令产生数据
```

<br/>

**`一般做法`**

```
- (void)testReactive {
    // 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        //block调用，执行命令的时候就会调用
        NSLog(@"%@",input); // input 为执行命令传进来的参数
        // 这里的返回值不允许为nil
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"执行命令产生的数据"];
            [subscriber sendCompleted];
            return nil;
        }];
    }];
    
   
    /*
    // 方式二：
    //一般做法
    // 订阅信号
    // 注意：这里必须是先订阅才能发送命令
    // executionSignals：信号源，信号中信号，signalofsignals:信号，发送数据就是信号
    [command.executionSignals subscribeNext:^(RACSignal *x) {
        [x subscribeNext:^(id x) {
            NSLog(@"%@", x);
        }];
    }];

    */

    //方法三
    //高级做法
    // switchToLatest获取最新发送的信号，只能用于信号中信号。
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {
        NSLog(@"%@", x);
    }];

    // 2.执行命令
    [command execute:@"welcome  to  my home!!"];

}
```

打印结果：

```
2018-11-21 17:17:05.006235+0800 HGSWB[10145:399746] welcome  to  my home!!
2018-11-21 17:17:25.073069+0800 HGSWB[10145:399746] 执行命令产生的数据
```

`注意：`方法二和方法三必须县订阅才能执行，而方法一可以先执行，再订阅。

<br/>
`注意:` 伴随着command一起构建的signal，记得要在操作完成后发送完成消息以表示其执行完了
```
[subscriber sendCompleted];
```
否则不能再执行此command。

&emsp; UIButton中有属性`rac_command`用于绑定一个已经创建好的command（其使用在后面讲到），当你使用第二种方式创建command时，button的enable属性会随command的可执行性而改变，意思是当传递布尔事件的信号传递了真值事件，按钮才可使用。另外，当你按下按钮，command开始执行时，按钮的enable被自动设置成了NO，除非command执行完了，怎么判断command执行完成了呢？就是当其伴随的signal发送完成事件的时候（上面提及到）。

注意: 当button的rac_command已经绑定了某个command，而这个command又是以第二种方式初始化，那么你就不能动态改变button的enable，如:

```
RAC(self.button, enable) = someSignal;
```
这样子运行起来会报错。（自己曾踩过的坑）

**`执行 RACCommand`**

调用函数：

```
- (RACSignal *)execute:(id)input;
```
在上面已经提到，input会作为创建command时其内部signal的构建block中的参数，用于传递数据。



<br/>

***
<br/>

> <h2 id='RACMulticastConnection(多播)'>RACMulticastConnection(多播)</h2>

&emsp; 用于当一个信号, 被多次订阅时, 为保证创建信号时, 避免多次调用创建信号中的block, 造成副作用, 可以使用这个类处理

**&emsp; 封装在底层, 开发中很少使用! 就是用来解决RACSignal的副作用(副作用就是不管你订阅多少次我就只会执行一次)**

```
- (void) testMethod14 {
    
    RACSignal *signal1 = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
           [subscriber sendNext:@"--->> signal1🍺🍺🍺🍺🍺🍺🍺"];
           [subscriber sendCompleted];
           return [RACDisposable disposableWithBlock:^{
               NSLog(@"signal1销毁了");
           }];
       }];
    
    //创建多播
    RACMulticastConnection *connection = [signal1 publish];
    //订阅多播
    [connection.signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"拿到多播数据1 %@", x);
    }];
    
    [connection.signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"拿到多播数据2 %@", x);
    }];
    
    [connection.signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"拿到多播数据3 %@", x);
    }];
    
    //千万不能忘记的进行连接
    [connection connect];
}
```

打印：

```
2021-08-18 11:48:48.366201+0800 Test1[11504:210180] 🍎 日志信息测试
2021-08-18 11:49:13.758727+0800 Test1[11504:210180] 拿到多播数据1 --->> signal1🍺🍺🍺🍺🍺🍺🍺
2021-08-18 11:49:15.727729+0800 Test1[11504:210180] 拿到多播数据2 --->> signal1🍺🍺🍺🍺🍺🍺🍺
2021-08-18 11:49:17.091577+0800 Test1[11504:210180] 拿到多播数据3 --->> signal1🍺🍺🍺🍺🍺🍺🍺
2021-08-18 11:49:21.832482+0800 Test1[11504:210180] signal1销毁了

```



<br/>

创建的信号block订阅了2次，调用了2次，如下：

```
- (void) testMethod0 {
    //1.创建信号
    RACSignal *siganal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        // block调用时刻：每当有订阅者订阅信号，就会调用block。
        
        // 2.发送信号
        [subscriber sendNext:@"发送数据"];
        
        //数据发送完要调用sendCompleted，内部会自动调用[RACDisposable disposable]取消订阅信号。
        [subscriber sendCompleted];
        //        NSError *error = [NSError new];
        //        [subscriber sendError:error];  //error会自动销毁信号
        return [RACDisposable disposableWithBlock:^{
            // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。
            
            // 执行完Block后，当前信号就不在被订阅了。
            NSLog(@"信号被销毁");
        }];
    }];
    
    // 3.订阅信号,才会激活信号
    [siganal subscribeNext:^(id  _Nullable x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"%@",x); //接受数据
    }];
    
    [siganal subscribeNext:^(id  _Nullable x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"订阅2: %@",x); //接受数据
    }];
    
    
}
```

打印：

```
2021-08-18 11:45:47.833306+0800 Test1[11351:203977] 发送数据
2021-08-18 11:45:51.432998+0800 Test1[11351:203977] 信号被销毁
2021-08-18 11:45:55.143469+0800 Test1[11351:203977] 订阅2: 发送数据
2021-08-18 11:45:57.378283+0800 Test1[11351:203977] 信号被销毁
```

[](https://www.jianshu.com/p/62c2729c4a72)
[操作符](https://juejin.cn/post/6953808004307222564#heading-7)



<br/>

***
<br/>

> <h2 id='RACScheduler'>RACScheduler</h2>

**RAC中的队列, 用GCD封装的**




<br/>

***
<br/>

> <h2 id='RACUnit'>RACUnit</h2>

**表示stream不包含有意义的值! 可以直接理解为nil**



<br/>

***
<br/>

> <h2 id='RACEvent'>RACEvent</h2>

**把数据包装成信号事件**



<br/>

***
<br/>

> <h2 id='属性2'>属性</h2>


<br/>


> <h3 id=''></h3>




<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>





<br/>

***
<br/>

> <h2 id='方法'>方法</h2>


<br/>


> <h3 id='rac_signalForSelector'>rac_signalForSelector(替代代理)</h3>

**用法：** 返回一个信号, 判断是否调用某个方法



<br/>
<br/>

> <h3 id='rac_valuesAndChangesForKeyPath'>rac_valuesAndChangesForKeyPath</h3>

**用法：** 监听某个对象的属性变化(KVO)

```
//注意:keypath(<#...#>) 的使用, 可以避免在观察者里面使用字符串, @keypath(self, age) == @"age"
[self rac_valuesForKeyPath:keypath(self, age) observer:self];
    
//移除观察者还需要使用OC的方法
```



<br/>
<br/>

> <h3 id='rac_signalForControlEvents'>rac_signalForControlEvents</h3>

```
//    监听登录按钮点击
[[_nextBtn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(__kindof UIControl * _Nullable x) {
   //执行登录事件
    [self.loginVM.loginCommand execute:nil];
}];
```




<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>

> <h3 id=''></h3>




<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>

> <h3 id=''></h3>