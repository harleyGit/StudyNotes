>#UITableView 上下滑动卡顿

①大量加载图片，没有进行缓存和异步绘制；
②使用大量Autolayout以及frame和bounds计算(可以计算后放在Modle中)


<br/>

># 网络请求
对于一些网络请求放在后台，做法是把耗时的请求操作放入后台线程，用GCD、NSOperation开启新线程；

<br/>

># 减少在UI中对象频繁创建
如在朋友圈中的cell，在进行UI操作时才把时间戳转化为一定的时间格式，这个可以在model的set方法进行转化；

<br/>


<br/>
***
<br/>



<br/>
参考资料：
[实战UITableview深度优化](http://www.cocoachina.com/ios/20180207/22197.html)
[性能优化那些事](https://www.cnblogs.com/oc-bowen/p/5999997.html)
[异步绘制Cell](https://blog.csdn.net/mo_xiao_mo/article/details/52622172)
[UITableView性能优化](https://www.jianshu.com/p/0014f736b130)
