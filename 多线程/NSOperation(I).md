
- **操作和操作队列**
- **NSOperation、NSOperationQueue 使用方法**
- **NSInvocationOperation 的使用**
- **NSBlockOperation**



<br/>

***
<br/>

># 操作和操作队列

**`操作(Operation):`**

&emsp;  执行操作,换句话说就是你在线程中执行的那段代码。
&emsp;  在 GCD 中是放在 block 中的。在 NSOperation 中，我们使用 NSOperation 子类 NSInvocationOperation、NSBlockOperation，或者自定义子类来封装操作。



**`操作队列(Operation Queues)`**

&emsp;  这里的队列指操作队列，即用来存放操作的队列。不同于 GCD 中的调度队列 FIFO（先进先出）的原则。NSOperationQueue 对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的开始执行顺序（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。

&emsp;  操作队列通过设置最大并发操作数（maxConcurrentOperationCount）来控制并发、串行。

&emsp;  NSOperationQueue 为我们提供了两种不同类型的队列：主队列和自定义队列。主队列运行在主线程之上，而自定义队列在后台执行。


<br/>

***
<br/>

># NSOperation、NSOperationQueue  使用方法

&emsp;  `NSOperation` 需要配合 `NSOperationQueue` 来实现多线程。因为默认情况下，`NSOperation` 单独使用时是系统的同步执行操作，配合 NSOperationQueue 可以实现异步执行。

`NSOperation 实现多线程的步骤为：`

&emsp;  创建操作：先将需要执行的操作封装到一个 NSOperation 对象中。

&emsp;  创建队列：创建 NSOperationQueue 对象。

&emsp;  将操作加入到队列中：将 NSOperation 对象添加到 NSOperationQueue 对象中。

之后系统就会自动将 NSOperationQueue 中的 NSOperation 取出来，在新线程中执行操作。


<br/>

`NSOperation的创建操作:`

&emsp; NSOperation 是个抽象类，不能用来封装操作。我们只有使用它的子类来封装操作。我们有三种方式来封装操作。

①.  使用子类 NSInvocationOperation;

②.  使用子类 NSBlockOperation;

③.  自定义继承自 NSOperation 的子类，通过实现内部相应的方法来封装操作;

&emsp; 在不使用 NSOperationQueue，单独使用 NSOperation 的情况下系统同步执行操作，下面我们学习以下操作的三种创建方式。

<br/>

***
<br/>

>#  NSInvocationOperation 的使用

`在主线程使用 NSInvocationOperation `

```

/**
 * 使用子类 NSInvocationOperation
 */
- (void)useInvocationOperation {
    
    // 1.创建 NSInvocationOperation 对象
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];
    
    // 2.调用 start 方法开始执行操作
    [op start];
}

/**
 * 任务1
 */
- (void)task1 {
    for (int i = 0; i < 2; i++) {
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@", [NSThread currentThread]);     // 打印当前线程
    }
}

```

输出：

`2019-05-20 14:22:37.467721+0800 YSC-NSOperation-demo[3051:82251] 1---<NSThread: 0x600001d62680>{number = 1, name = main}`

`2019-05-20 14:22:39.469215+0800 YSC-NSOperation-demo[3051:82251] 1---<NSThread: 0x600001d62680>{number = 1, name = main}`


&emsp;  在没有使用 NSOperationQueue、在主线程中单独使用使用子类 NSInvocationOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。


<br/>

**`在子程使用 NSInvocationOperation `**

```
//    在其他线程使用子类 NSInvocationOperation
    [NSThread detachNewThreadSelector:@selector(useInvocationOperation) toTarget:self withObject:nil];


/**
 * 使用子类 NSInvocationOperation
 */
- (void)useInvocationOperation {
    
    // 1.创建 NSInvocationOperation 对象
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];
    
    // 2.调用 start 方法开始执行操作
    [op start];
}

/**
 * 任务1
 */
- (void)task1 {
    for (int i = 0; i < 2; i++) {
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@", [NSThread currentThread]);     // 打印当前线程
    }
}
```

输出：

 `019-05-20 14:34:38.696854+0800 YSC-NSOperation-demo[3197:87716] 1---<NSThread: 0x600001980600>{number = 3, name = (null)}`
 
`2019-05-20 14:34:40.701441+0800 YSC-NSOperation-demo[3197:87716] 1---<NSThread: 0x600001980600>{number = 3, name = (null)}`

&emsp；在其他线程中单独使用子类 NSInvocationOperation，操作是在当前调用的其他线程执行的，并没有开启新线程。

<br/>

***
<br/>

>#  NSBlockOperation

```
//    在当前线程使用 NSBlockOperation
    [self useBlockOperation];


/**
 * 使用子类 NSBlockOperation
 */
- (void)useBlockOperation {
    
    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    
    // 2.调用 start 方法开始执行操作
    [op start];
}
```

输出：

`2019-05-20 17:02:04.186312+0800 YSC-NSOperation-demo[21633:174028] 1---<NSThread: 0x600001554140>{number = 1, name = main}`

`2019-05-20 17:02:06.187878+0800 YSC-NSOperation-demo[21633:174028] 1---<NSThread: 0x600001554140>{number = 1, name = main}`

&emsp;  在没有使用 NSOperationQueue、在主线程中单独使用 NSBlockOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。

&emsp;  与 NSInvocationOperation 使用一样。因为代码是在主线程中调用的，所以打印结果为主线程。如果在其他线程中执行操作，则打印结果为其他线程。



<br/>

```
//    使用 NSBlockOperation 的 AddExecutionBlock: 方法
    [self useBlockOperationAddExecutionBlock];


/**
 * 使用子类 NSBlockOperation
 * 调用方法 AddExecutionBlock:
 */
- (void)useBlockOperationAddExecutionBlock {
    
    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    
    // 2.添加额外的操作
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];          // 模拟耗时操作
            NSLog(@"5---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    // 3.调用 start 方法开始执行操作
    [op start];
}

```

输出：

```
2019-05-20 17:18:39.602123+0800 YSC-NSOperation-demo[22041:183103] 3---<NSThread: 0x60000138af00>{number = 1, name = main}

2019-05-20 17:18:39.602129+0800 YSC-NSOperation-demo[22041:183155] 1---<NSThread: 0x6000013d0140>{number = 4, name = (null)}

2019-05-20 17:18:39.602128+0800 YSC-NSOperation-demo[22041:183154] 2---<NSThread: 0x6000013d9440>{number = 5, name = (null)}

2019-05-20 17:18:39.602130+0800 YSC-NSOperation-demo[22041:183162] 4---<NSThread: 0x6000013d4700>{number = 3, name = (null)}

2019-05-20 17:18:41.602619+0800 YSC-NSOperation-demo[22041:183155] 1---<NSThread: 0x6000013d0140>{number = 4, name = (null)}

2019-05-20 17:18:41.602619+0800 YSC-NSOperation-demo[22041:183103] 3---<NSThread: 0x60000138af00>{number = 1, name = main}

2019-05-20 17:18:41.602619+0800 YSC-NSOperation-demo[22041:183154] 2---<NSThread: 0x6000013d9440>{number = 5, name = (null)}

2019-05-20 17:18:41.602619+0800 YSC-NSOperation-demo[22041:183162] 4---<NSThread: 0x6000013d4700>{number = 3, name = (null)}

2019-05-20 17:18:43.603699+0800 YSC-NSOperation-demo[22041:183162] 5---<NSThread: 0x6000013d4700>{number = 3, name = (null)}

2019-05-20 17:18:45.604132+0800 YSC-NSOperation-demo[22041:183162] 5---<NSThread: 0x6000013d4700>{number = 3, name = (null)}

```


&emsp;  NSBlockOperation 还提供了一个方法 addExecutionBlock:，通过`addExecutionBlock: `就可以为 NSBlockOperation 添加额外的操作。这些操作（包括 blockOperationWithBlock 中的操作）可以在不同的线程中同时（并发）执行。只有当所有相关的操作已经完成执行时，才视为完成。

&emsp; 如果添加的操作多的话，blockOperationWithBlock: 中的操作也可能会在其他线程（非当前线程）中执行，这是由系统决定的，并不是说添加到 blockOperationWithBlock: 中的操作一定会在当前线程中执行。（可以使用 addExecutionBlock: 多添加几个操作试试）。

&emsp;  使用子类 NSBlockOperation，并调用方法 AddExecutionBlock: 的情况下，blockOperationWithBlock:方法中的操作 和 addExecutionBlock: 中的操作是在不同的线程中异步执行的。

&emsp;  而且，这次执行结果中 blockOperationWithBlock:方法中的操作也不是在当前线程（主线程）中执行的。从而印证了blockOperationWithBlock: 中的操作也可能会在其他线程（非当前线程）中执行。

&emsp;  一般情况下，如果一个 NSBlockOperation 对象封装了多个操作。NSBlockOperation 是否开启新线程，取决于操作的个数。如果添加的操作的个数多，就会自动开启新线程。当然开启的线程数是由系统来决定的。



<br/>

***
<br/>

参考资料：

**[iOS 多线程：『NSOperation、NSOperationQueue』详尽总结](https://www.jianshu.com/p/4b1d77054b35)**
