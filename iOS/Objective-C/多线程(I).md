> <h2 id=''></h2>
- [GCD](#GCD)
	- [并发](#并发)
	- [线程组和依赖](#线程组和依赖)
		- 	[线程组处理](#线程组处理)
- [**NSOperation**](#NSOperation)
	- [常驻线程](#常驻线程)
	- 	[依赖](#依赖)
	- [	多任务(任务是异步)](#多任务(任务是异步))
	- [并发](#并发)
- [**内存问题**](#内存问题)
- **资料**
	- [GCD](https://www.cnblogs.com/EchoHG/p/8609685.html)


<br/>
<br/>

***
<br/>
<br/>


><h1 id='GCD'>GCD</h1>

<br/>


![GCD](./../../Pictures/ios_oc105.png)

<br/>

![GCD分类](./../../Pictures/ios_oc1_30.webp)



<br/>

><h2 id='并发'>并发</h2>


&emsp; 在 iOS 并发编程技术中，GCD 的使用率是最高的。所以，在这篇文章中，我就以 GCD 为例和你说说多线程的并发问题。


&emsp; GCD（Grand Central Dispatch）是由苹果公司开发的一个多核编程解决方案。它提供的一套简单易用的接口，极大地方便了并发编程。同时，它还可以完成对复杂的线程创建、释放时机的管理。但是，GCD 带来这些便利的同时，也带来了资源使用上的风险。


&emsp; 例如，在进行数据读写操作时，总是需要一段时间来等待磁盘响应的，如果在这个时候通过 GCD 发起了一个任务，那么 GCD 就会本着最大化利用 CPU 的原则，会在等待磁盘响应的这个空档，再创建一个新线程来保证能够充分利用 CPU。

&emsp; 而如果 GCD 发起的这些新任务，都是类似于数据存储这样需要等待磁盘响应的任务的话，那么随着任务数量的增加，GCD 创建的新线程就会越来越多，从而导致内存资源越来越紧张，等到磁盘开始响应后，再读取数据又会占用更多的内存。结果就是，失控的内存占用会引起更多的内存问题。


&emsp; 这种情况最典型的场景就是数据库读写操作。[FMDB](https://github.com/ccgus/fmdb)是一个开源的第三方数据库框架，通过 FMDatabaseQueue 这个核心类，将与读写数据库相关的磁盘操作都放到一个串行队列里执行，从而避免了线程创建过多导致系统资源紧张的情况。


<br/>


为了能够支持以后可能更大的并发量，下面我将其中“已读”功能的数据库操作改成 FMDatabaseQueue。这样，我就可以将并行队列转化为串行队列来执行，避免大并发读写磁盘操作造成内存问题，改写代码如下：

```
// 标记文章已读
- (RACSignal *)markFeedItemAsRead:(NSUInteger)iid fid:(NSUInteger)fid{
    @weakify(self);
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        @strongify(self);
        // 改写成 FMDatabaseQueue 串行队列进行数据库操作
        FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:self.feedDBPath];
        [queue inDatabase:^(FMDatabase *db) {
            FMResultSet *rs = [FMResultSet new];
            // 读取文章数据
            if (fid == 0) {
                rs = [db executeQuery:@"select * from feeditem where isread = ? and iid >= ? order by iid desc", @(0), @(iid)];
            } else {
                rs = [db executeQuery:@"select * from feeditem where isread = ? and iid >= ? and fid = ? order by iid desc", @(0), @(iid), @(fid)];
            }
            NSUInteger count = 0;
            while ([rs next]) {
                count++;
            }
            // 更新文章状态为已读
            if (fid == 0) {
                [db executeUpdate:@"update feeditem set isread = ? where iid >= ?", @(1), @(iid)];
            } else {
                [db executeUpdate:@"update feeditem set isread = ? where iid >= ? and fid = ?", @(1), @(iid), @(fid)];
            }
            
            [subscriber sendNext:@(count)];
            [subscriber sendCompleted];
            [db close];
        }];
        return nil;
    }];
}
```

如代码所示，你只需要将数据库的操作放到 FMDatabaseQueue 的 inDatabase 方法入参 block 中，就可以在 FMDatabaseQueue 维护的串行队列里排队等待执行了.



<br/>
<br/>
<br/>


> <h2 id='线程组和依赖'>线程组和依赖</h2>  

<br/>

<h3 id='线程组处理'>线程组处理</h3>

```
{
    // 创建队列组
    dispatch_group_t group =  dispatch_group_create();
    // 创建并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
 
    // 开子线程，任务1
    dispatch_group_async(group, queue, ^{
        [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://img-blog.csdn.net/20180421152137506"]];
        NSLog(@"任务1 完成，线程：%@", [NSThread currentThread]);
    });
 
    // 开子线程，任务2
    dispatch_group_async(group, queue, ^{
        [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://img-blog.csdn.net/20170112145924755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGVyb193cWI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center"]];
        NSLog(@"任务2 完成，线程：%@", [NSThread currentThread]);
    });
 
    // 全部完成
    dispatch_group_notify(group, queue, ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"全部完成，线程：%@", [NSThread currentThread]);
        });
    });
}
```

<br/>

输出结果：

```
AsyTaskTest[5963:308229] 任务1 完成，线程：<NSThread: 0x604000263380>{number = 3, name = (null)}

AsyTaskTest[5963:308228] 任务2 完成，线程：<NSThread: 0x60400007c4c0>{number = 4, name = (null)}

AsyTaskTest[5963:308103] 全部完成，线程：<NSThread: 0x604000070600>{number = 1, name = main}

```




<br/>
<br/>
<br/>

***
<br/>




> <h1 id='NSOperation'>NSOperation</h2>

&emsp; 在 AFNetworking 2.0 中，把每个请求都封装成了单独的 NSOperationQueue，再由 NSOperationQueue 根据当前的 CPU 数量和系统负载来控制并发。那么，为什么 AFNetworking 2.0 没有为每个请求创建一个线程，而只是创建了一个队列，用来接收 NSOperationQueue 的回调呢？

&emsp; FMDB 只通过 FMDatabaseQueue 开启了一个线程队列，来串行地操作数据库。这又是为什么呢？

<br/>


&emsp; 这大概就是因为多线程技术有坑。特别是 UIKit 干脆就做成了线程不安全，只能在主线程上操作。

&emsp; 而写 UIKit、AFNetworking、FMDB 这些库的“大神”们，并不是解决不了多线程技术可能会带来的问题，而相反正是因为他们非常清楚这些可能存在的问题，所以为避免使用者滥用多线程，亦或是出于性能考虑，而选择了使用单一线程来保证这些基础库的稳定可用。


举一个在开发中可能遇到的一个例子：以照片处理为例，当选择一张照片后，你希望能够看到不同滤镜处理后的效果。如果这些效果图都是在一个队列里串行处理的话，那么你就得等着这些滤镜一个一个地来处理。这么做的话，不仅会影响用户体验，也没能充分利用硬件资源，可以说是把高端手机当作低端机来用了。换句话说就是，用户花大价钱升级了手机硬件，操作 App 的体验却没有得到提升。

所以，我们不能因为多线程技术有坑就不去用，正确的方法应该是更多地去了解多线程会有哪些问题，如果我们能够事先预见到那些问题的话，那么避免这些问题的发生也就不在话下了。

接下来，我们就一起来看看多线程技术常见的两个大坑，常驻线程和并发问题，分别是从何而来，以及如何避免吧。


<br/>
<br/>


> <h2 id='常驻线程'>常驻线程</h2>

&emsp; 常驻线程，指的就是那些不会停止，一直存在于内存中的线程。在 AFNetworking 2.0 专门创建了一个线程来接收 NSOperationQueue 的回调，这个线程其实就是一个常驻线程。接下来，我们就看看常驻线程这个问题是如何引起的，以及是否有对应的解决方案。

在AFNetworking 2.0 创建常驻线程的代码，来看一下这个线程是怎么创建的：

```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        // 先用 NSThread 创建了一个线程
        [[NSThread currentThread] setName:@"AFNetworking"];
        // 使用 run 方法添加 runloop
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
```

AFNetworking 2.0 先用 NSThread 创建了一个线程，并使用 NSRunLoop 的 run 方法给这个新线程添加了一个 runloop。


<br/>


通过 NSRunLoop 添加 runloop 的方法有三个：
- run 方法。通过 run 方法添加的 runloop ，会不断地重复调用 runMode:beforeDate: 方法，来保证自己不会停止。
- runUntilDate: 和 runMode:beforeDate 方法。这两个方法添加的 runloop，可以通过指定时间来停止 runloop。

但是若是过多的创建常驻线程，不但不能提高 CPU 的利用率，反而会降低程序的执行效率。常驻线程是很耗费资源的。



<br/>

&emsp; 既然常线程是个坑，那为什么 **AFNetworking 2.0 库还要这么做呢？**

&emsp; 其实，这个问题的根源在于 AFNetworking 2.0 使用的是 NSURLConnection，而 NSURLConnection 的设计上存在些缺陷。接下来，我和你说说它的设计上有哪些缺陷，了解了这些缺陷后你也就能够理解当时 AFNetworking 2.0 为什么明知常驻线程有坑，还是使用了常驻线程。这样，你以后再碰到类似的情况时，也可以跟 AFNetworking 2.0 一样使用常线程去解决问题，只要不滥用常驻线程就可以了。

&emsp; NSURLConnection 发起请求后，所在的线程需要一直存活，以等待接收 NSURLConnectionDelegate 回调方法。但是，网络返回的时间不确定，所以这个线程就需要一直常驻在内存中。既然这样，AFNetworking 2.0 为什么没有在主线程上完成这个工作，而一定要新创建一个线程来做呢？


&emsp; 这是因为主线程还要处理大量的 UI 和交互工作，为了减少对主线程的影响，所以 AFNetworking 2.0 就新建了一个常驻线程，用来处理所有的请求和回调。AFNetworking 2.0 的线程设计如下图所示：

![ios_oc1_21.webp](./../../Pictures/ios_oc1_21.webp)

&emsp; 通过上面的分析我们可以知道，如果不是因为 NSURLConnection 的请求必须要有一个一直存活的线程来接收回调，那么 AFNetworking 2.0 就不用创建一个常驻线程出来了。虽然说，在一个 App 里网络请求这个动作的占比很高，但也有很多不需要网络的场景，所以线程一直常驻在内存中，也是不合理的。


<br/>
<br/>


&emsp; AFNetworking 在 3.0 版本时，使用苹果公司新推出的 NSURLSession 替换了 NSURLConnection，从而避免了常驻线程这个坑。NSURLSession 可以指定回调 NSOperationQueue，这样请求就不需要让线程一直常驻在内存里去等待回调了。实现代码如下：

```
self.operationQueue = [[NSOperationQueue alloc] init];
self.operationQueue.maxConcurrentOperationCount = 1;
self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
```

&emsp; 从上面的代码可以看出，NSURLSession 发起的请求，可以指定回调的 delegateQueue，不再需要在当前线程进行代理方法的回调。所以说，NSURLSession 解决了 NSURLConnection 的线程回调问题。


<br/>

&emsp; 如果你需要确实需要保活线程一段时间的话，可以选择使用 NSRunLoop 的另外两个方法 runUntilDate: 和 runMode:beforeDate，来指定线程的保活时长。让线程存活时间可预期，总比让线程常驻，至少在硬件资源利用率这点上要更加合理。


或者，你还可以使用 CFRunLoopRef 的 CFRunLoopRun 和 CFRunLoopStop 方法来完成 runloop 的开启和停止，达到将线程保活一段时间的目的。








<br/>
<br/>

> <h2 id='依赖'>依赖</h2>


```
{
    // 创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 任务1
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://img-blog.csdn.net/20180421152137506"]];
        NSLog(@"任务1 完成，线程：%@", [NSThread currentThread]);
    }];
 
    // 任务2
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://img-blog.csdn.net/20170112145924755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGVyb193cWI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center"]];
        NSLog(@"任务2 完成，线程：%@", [NSThread currentThread]);
    }];
 
    // 添加操作依赖，注意不能循环依赖
    [op1 addDependency:op2];
 
    op1.completionBlock = ^{
        NSLog(@"全部完成，线程：%@", [NSThread currentThread]);
    };
 
    // 添加操作到队列
    [queue addOperation:op1];
    [queue addOperation:op2];
}


```

输出结果：

```
AsyTaskTest[6009:309365] 任务2 完成，线程：<NSThread: 0x600000277c80>{number = 3, name = (null)}

AsyTaskTest[6009:309362] 任务1 完成，线程：<NSThread: 0x60400046c0c0>{number = 4, name = (null)}

AsyTaskTest[6009:309364] 全部完成，线程：<NSThread: 0x600000277d40>{number = 5, name = (null)}


```




<br/>
<br/>



><h2 id='多任务(任务是异步)'>多任务(任务是异步)</h2>


若是直接使用上述的线程组来执行，可能会出问题，如下问题的模拟：

```
{
    NSURLSession *session = [NSURLSession sharedSession];
 
    // 创建队列组
    dispatch_group_t group =  dispatch_group_create();
    // 创建并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
 
    // 任务1
    dispatch_group_async(group, queue, ^{
        NSURLSessionDataTask *task1 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/hero/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
            NSLog(@"任务1 完成，线程：%@", [NSThread currentThread]);
        }];
        [task1 resume];
    });
 
    // 任务2
    dispatch_group_async(group, queue, ^{
        NSURLSessionDataTask *task2 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/thumbnails/erin-sarofsky/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
            NSLog(@"任务2 完成，线程：%@", [NSThread currentThread]);
        }];
        [task2 resume];
    });
 
    // 全部完成
    dispatch_group_notify(group, queue, ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"全部完成，线程：%@", [NSThread currentThread]);
        });
    });
}
 
! 输出结果：

AsyTaskTest[6048:310326] 全部完成，线程：<NSThread: 0x60000007f480>{number = 1, name = main}

AsyTaskTest[6048:310361] 任务2 完成，线程：<NSThread: 0x604000468a80>{number = 3, name = (null)}

AsyTaskTest[6048:310364] 任务1 完成，线程：<NSThread: 0x60400046a900>{number = 4, name = (null)}

```

- 通过dispatch_group_enter、dispatch_group_leave解决异步问题：

```
{
    NSURLSession *session = [NSURLSession sharedSession];
 
    // 创建队列组
    dispatch_group_t group = dispatch_group_create();
 
    // 任务1
    dispatch_group_enter(group);
    NSURLSessionDataTask *task1 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/hero/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"任务1 完成，线程：%@", [NSThread currentThread]);
        dispatch_group_leave(group);
    }];
    [task1 resume];
 
    // 任务2
    dispatch_group_enter(group);
    NSURLSessionDataTask *task2 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/thumbnails/erin-sarofsky/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"任务2 完成，线程：%@", [NSThread currentThread]);
        dispatch_group_leave(group);
    }];
    [task2 resume];
 
    // 全部完成
    dispatch_group_notify(group, dispatch_get_main_queue(), ^(){
        NSLog(@"全部完成，线程：%@", [NSThread currentThread]);
    });

    /*或者这样做
    dispatch_group_wait(taskGroup, DISPATCH_TIME_FOREVER);

        dispatch_async(dispatch_get_main_queue(), ^{

        //执行mainTask

    });
    */
}

```
 
输出结果：

```
AsyTaskTest[6102:311543] 任务2 完成，线程：<NSThread: 0x60400046bd00>{number = 3, name = (null)}

AsyTaskTest[6102:311539] 任务1 完成，线程：<NSThread: 0x60400046c700>{number = 4, name = (null)}

AsyTaskTest[6102:311509] 全部完成，线程：<NSThread: 0x60400007cdc0>{number = 1, name = main}

```


- 使用信号量+线程组来处理

```

{
    // 初始化信号量
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
 
    NSURLSession *session = [NSURLSession sharedSession];
 
    // 创建队列组
    dispatch_group_t group =  dispatch_group_create();
    // 创建并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
 
    // 任务1
    dispatch_group_async(group, queue, ^{
        NSURLSessionDataTask *task1 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/hero/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
            NSLog(@"任务1 完成，线程：%@", [NSThread currentThread]);
            // 发送信号，使信号量+1
            dispatch_semaphore_signal(semaphore);
        }];
        [task1 resume];
    });
    // 信号量等于0时会一直等待，大于0时正常执行，并让信号量-1
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
 
    // 任务2
    dispatch_group_async(group, queue, ^{
        NSURLSessionDataTask *task2 = [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/105/media/us/imac-pro/2018/d0b63f9b_f0de_4dea_a993_62b4cb35ca96/thumbnails/erin-sarofsky/large.mp4"] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
            NSLog(@"任务2 完成，线程：%@", [NSThread currentThread]);
            dispatch_semaphore_signal(semaphore);
        }];
        [task2 resume];
    });
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
 
    // 全部完成
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"全部完成，线程：%@", [NSThread currentThread]);
    });
}

```
 
输出结果：

```
AsyTaskTest[6149:312567] 任务1 完成，线程：<NSThread: 0x600000468040>{number = 3, name = (null)}

AsyTaskTest[6149:312567] 任务2 完成，线程：<NSThread: 0x600000468040>{number = 3, name = (null)}

AsyTaskTest[6149:312507] 全部完成，线程：<NSThread: 0x60000007d680>{number = 1, name = main}
 
```



<br/>
<br/>

***
<br/>


> <h2 id='内存问题'>内存问题</h2>


&emsp; 创建线程的过程，需要用到物理内存，CPU 也会消耗时间。而且，新建一个线程，系统还需要为这个进程空间分配一定的内存作为线程堆栈。堆栈大小是 4KB 的倍数。在 iOS 开发中，主线程堆栈大小是 1MB，新创建的子线程堆栈大小是 512KB。

&emsp; 除了内存开销外，线程创建得多了，CPU 在切换线程上下文时，还会更新寄存器，更新寄存器的时候需要寻址，而寻址的过程还会有较大的 CPU 消耗。

&emsp; 所以，线程过多时内存和 CPU 都会有大量的消耗，从而导致 App 整体性能降低，使得用户体验变成差。CPU 和内存的使用超出系统限制时，甚至会造成系统强杀。这种情况对用户和 App 的伤害就更大了。























