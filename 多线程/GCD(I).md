># GCD 三种队列内型
**`The main queue`**: 与主线程功能相同。实际上，提交至main queue的任务会在主线程中执行。main queue可以调用dispatch_get_main_queue()来获得。因为main queue是与主线程相关的，所以这是一个串行队列。

**`Global queues`**: 全局队列是并发队列，并由整个进程共享。进程中存在三个全局队列：高、中（默认）、低三个优先级队列。可以调用dispatch_get_global_queue函数传入优先级来访问队列。

**`用户队列`**: 用户队列 (GCD并不这样称呼这种队列, 但是没有一个特定的名字来形容这种队列，所以我们称其为用户队列) 是用函数 dispatch_queue_create 创建的队列. 这些队列是串行的。正因为如此，它们可以用来完成同步机制, 有点像传统线程中的mutex。

&emsp;  这个有没有开启新线程，需要看这种队列和dispatch_async和dispatch_sync的搭配，请看[这里](https://www.cnblogs.com/EchoHG/p/8609685.html)
![3种队列和2种任务(同步、异步)的搭配](https://upload-images.jianshu.io/upload_images/2959789-74ba672570998f83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>



>#  dispatch_get_global_queue()
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



<br/>
参考资料：
[]()











