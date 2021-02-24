- **GCD队列类型**
	- GCD和NSOperaiton 
	- 3种队列类型
	- dispatch_get_global_queue()
- **线程组**
- **线程安全**
- **时间延迟**
	- dispatch_after
	- dispatch_time_t
	- dispatch_walltime	
- [**多线程**](https://meniny.cn/posts/iOS_Primer_017/)



<br/>

***
<br/>

># GCD队列类型

<br/>
<br/>

> GCD和NSOperaiton 
 - 能用GCD的地方就用GCD 尽量减少NSOperation的使用,因为GCD在多核CPU上线程切换的时间比较短 效率相对高些；
 - NSOperation是建立在GCD之上的，虽然使用起来比较复杂，但是在线程并发管理、优先级、上有着GCD无法比拟的优势。


<br/>
<br/>

> 3种队列类型

**`The main queue`**: 与主线程功能相同。实际上，提交至main queue的任务会在主线程中执行。main queue可以调用dispatch_get_main_queue()来获得。因为main queue是与主线程相关的，所以这是一个串行队列。

**`Global queues`**: 全局队列是并发队列，并由整个进程共享。进程中存在三个全局队列：高、中（默认）、低三个优先级队列。可以调用dispatch_get_global_queue函数传入优先级来访问队列。

**`用户队列`**: 用户队列 (GCD并不这样称呼这种队列, 但是没有一个特定的名字来形容这种队列，所以我们称其为用户队列) 是用函数 dispatch_queue_create 创建的队列. 这些队列是串行的。正因为如此，它们可以用来完成同步机制, 有点像传统线程中的mutex。

&emsp;  这个有没有开启新线程，需要看这种队列和dispatch_async和dispatch_sync的搭配，请看[这里](https://www.cnblogs.com/EchoHG/p/8609685.html)

![3种队列和2种任务(同步、异步)的搭配](https://upload-images.jianshu.io/upload_images/2959789-74ba672570998f83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
<br/>


> dispatch_get_global_queue()

```

NSLog(@"11111------->>Post 线程请求： %@", [NSThread currentThread]);

dispatch_async(dispatch_get_global_queue(), ^{
	//网络请求，耗时操作
	NSLog(@"22222------->>Post 线程请求： %@", [NSThread currentThread]);
	
	dispatch_async(dispatch_get_main_queue(), ^{
	        //网络请求到数据，进入主线程，开始把数据放到UI上
	        NSLog(@"~~~~222------->>Post 线程请求： %@", [NSThread currentThread]);
	
	    });

});

```

打印结果：

```

2018-12-03 10:52:00.338757+0800 TaiChi_iOS[12042:777252] 11111------->>Post 线程请求： <NSThread: 0x281ebef00>{number = 1, name = main}
2018-12-03 10:52:00.618865+0800 TaiChi_iOS[12042:777252] 22222------->>Post 线程请求： <NSThread: 0x281ebef00>{number = 1, name = main}
2018-12-03 10:52:00.629733+0800 TaiChi_iOS[12042:777252] ~~~~222------->>Post 线程请求： <NSThread: 0x281ebef00>{number = 1, name = main}

```

`结论：`dispatch_get_global_queue()这个线程队列，开启线程队列会视情况而定，有时会开启线程有时不会开启线程。



<br/>

***
<br/>


># 线程组

**`dispatch_group常用方法：`**
- `dispatch_group_enter` :通知 group,下个任务要放入 group 中执行了
- `dispatch_group_leave`: 通知 group,任务成功完成,要移除,与 enter成对出现
- `dispatch_group_wait`: 在任务组完成时调用，或者任务组超时是调用（完成指的是enter和leave次数一样多）
- `dispatch_group_notify`: 只要任务全部完成了,就会在最后调用


[**dispatch_group_enter和dispatch_group_leave 网络请求嵌套**](https://www.jianshu.com/p/71fbc0415b5d)




<br/>

***
<br/>


># 线程安全

> dispatch_semaphore

[**网络请求依次执行**](https://www.jianshu.com/p/71fbc0415b5d)


<br/>

***
<br/>




># 时间延迟

<br/>

> 类型

- **dispatch_after**

&emsp;  `dispatch_after`是在指定的时间将任务加入到队列中，而不是在指定的时间执行，如果所在的队列上有耗时任务在执行，那么时间上可能出现误差。

```
void  dispatch_after(dispatch_time_t when, dispatch_queue_t queue, dispatch_block_t block);
```


```

NSLog(@"2秒后开始执行");
dispatch_after( dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(),^{
    NSLog(@"等2秒，开始执行任务");
    sleep(2);
    NSLog(@"任务执行完成");
} );

```

输出：

`2019-05-14 11:51:26.890458+0800 Genealogy[1897:44256] 2秒后开始执行`

`2019-05-14 11:51:35.784266+0800 Genealogy[1897:44256] 等2秒，开始执行任务`

`2019-05-14 11:51:40.692939+0800 Genealogy[1897:44256] 任务执行完成`



<br/>
- **dispatch_time_t**

`dispatch_time_t`:用来指定时间，传入的是dispatch_time_t类型的值，通过dispatch_time和dispatch_walltime函数生成;

<br/>

`dispatch_time_t
dispatch_time(dispatch_time_t when, int64_t delta);`


`dispatch_time_t when`:时间点;
`int64_t delta`时间长度;

&emsp;  函数的作用就是获取时间点经过时间长度之后的时间点。第一个参数我们经常使用的是DISPATCH_TIME_NOW，表示现在这个时间点。第二个参数表示的时间长度使用数组* NSEC_PER_SEC的方式获得。

`NSEC_PER_SEC` 表示秒

`NSEC_PER_MSEC` 表示毫秒

`NSEC_PER_USEC` 表示微秒

<br/>

- **dispatch_queue_t queue**

`dispatch_queue_t queue`:任务添加的队列;


<br/>

- **dispatch_walltime**

`dispatch_time_t
dispatch_walltime(const struct timespec *_Nullable when, int64_t delta);`

`const struct timespec *_Nullable when`:  一个struct timespec类型的值;
`int64_t delta`:  以纳秒为单位的延迟时间;


```

//设置时间点为2秒后
NSTimeInterval iT = [[NSDate dateWithTimeInterval:2 sinceDate:[NSDate date]] timeIntervalSince1970];

struct timespec time;
time.tv_sec = (NSInteger)iT;
//比时间点再晚10秒
dispatch_time_t timer = dispatch_walltime(&time, 10*NSEC_PER_SEC);
NSLog(@"12秒后开始任务");
dispatch_after(timer, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
    NSLog(@"任务开始");
    
    sleep(1);

    NSLog(@"任务完成");
});

```

输出：

`2019-05-14 12:06:25.194589+0800 Genealogy[2330:55634] 6秒后开始任务`

`2019-05-14 12:06:33.000349+0800 Genealogy[2330:55725] 任务开始`

`2019-05-14 12:06:34.003165+0800 Genealogy[2330:55725] 任务完成`



<br/>

***
<br/>

参考资料：
[]()











