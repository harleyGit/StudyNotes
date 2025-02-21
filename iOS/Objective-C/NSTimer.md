
- 简介
- [**iOS定时器NSTimer内存泄露原理分析+解决方案**](https://c.m.163.com/news/a/DSN3GH9V053809XK.html?spss=newsapp&spsw=2&spssid=ab2c55f2dafa9bc7d17d102d3fd98666)
- [**NSTimer详解**](https://www.jianshu.com/p/d4589134358a)


<br/>

***
<br/>



># 简介


&emsp;  NSTimer是iOS开发执行定时任务时常用的类，它支持定制定时任务的开始执行时间、任务时间间隔、重复执行、RunLoopMode等。

&emsp; NSTimer必须与RunLoop搭配使用，因为其定时任务的触发基于RunLoop，NSTimer使用常见的Target-Action模式。由于RunLoop会强引用timer，timer会强引用Target，容易造成循环引用、内存泄露等问题。



<br/>

***
<br/>

