# GCD(II) - 时间延迟
<br/>


># GCD 和 NSOperaiton 
 - 能用GCD的地方就用GCD 尽量减少NSOperation的使用,因为GCD在多核CPU上线程切换的时间比较短 效率相对高些；
 - NSOperation是建立在GCD之上的，虽然使用起来比较复杂，但是在线程并发管理、优先级、上有着GCD无法比拟的优势。



<br/>
* * *
<br/>

># dispatch_group
**`Dispatch_group常用方法：`**
- `dispatch_group_enter` :通知 group,下个任务要放入 group 中执行了
- `dispatch_group_leave`: 通知 group,任务成功完成,要移除,与 enter成对出现
- `dispatch_group_wait`: 在任务组完成时调用，或者任务组超时是调用（完成指的是enter和leave次数一样多）
- `dispatch_group_notify`: 只要任务全部完成了,就会在最后调用


[dispatch_group_enter 和 dispatch_group_leave 网络请求嵌套](https://www.jianshu.com/p/71fbc0415b5d)



<br/>
* * *
<br/>
># dispatch_semaphore


[网络请求依次执行](https://www.jianshu.com/p/71fbc0415b5d)


<br/>
* * *
<br/>




># dispatch_after

## 功能：
&emsp;  `dispatch_after`是在指定的时间将任务加入到队列中，而不是在指定的时间执行，如果所在的队列上有耗时任务在执行，那么时间上可能出现误差。

`void  dispatch_after(dispatch_time_t when, dispatch_queue_t queue,
		dispatch_block_t block);`

`dispatch_time_t when`:用来指定时间，传入的是dispatch_time_t类型的值，通过dispatch_time和dispatch_walltime函数生成;

`dispatch_queue_t queue`:任务添加的队列;


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


##   dispatch_time

`dispatch_time_t
dispatch_time(dispatch_time_t when, int64_t delta);`


`dispatch_time_t when`:时间点;
`int64_t delta`时间长度;

&emsp;  函数的作用就是获取时间点经过时间长度之后的时间点。第一个参数我们经常使用的是DISPATCH_TIME_NOW，表示现在这个时间点。第二个参数表示的时间长度使用数组* NSEC_PER_SEC的方式获得。

`NSEC_PER_SEC` 表示秒
`NSEC_PER_MSEC` 表示毫秒
`NSEC_PER_USEC` 表示微秒




<br/>


##   dispatch_walltime

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