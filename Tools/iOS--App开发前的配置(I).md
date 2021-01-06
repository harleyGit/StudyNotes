>#项目是否处在Debug状态

点击项目工程->【TARGETS】下的项目->【Build Settings】，在搜索框中输入：“macros”，如下图。
![Debug 状态查看](https://upload-images.jianshu.io/upload_images/2959789-e9fe0de927cf1309.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来在项目中的宏文件中可以这样做：
```
#ifdef DEBUG
    //do sth.
#else
    //do sth.
#endif
```
  一般Apple已经为我们设置好了 DEBUG 的宏定义，所以，我们只要让 NSLog 在 DEBUG 模式下失效就好了，这样能让我们的程序运行起来更加稳定，同时我们也可以继续使用正规的 NSLog。
```Swift
//put this in prefix.pch

#ifndef DEBUG
#undef NSLog
#define NSLog(args, ...)
#endif
```
<br/>
***

># 开启Debug或Release模式

  一个发布的程序，若带有太多的NSLog输出，肯定对于App性能有所影响，我们可以使用一个宏定义来处理，在开发的时候使用DEBUG模式，在发布的时候使用RELEASE模式。这样，发布的App就不会在程序内部做大量的NSLog输出了。

##Debug和Release模式的区别
 &emsp; Release是发行版本,比Debug版本有一些优化，文件比Debug文件小 Debug是调试版本，Debug和Release调用两个不同的底层库。通俗点讲，我们开发者自己内部真机或模拟器调试时，使用Debug模式就好，等到想要发布时，也就是说需要大众客户使用时，需要build Release版本，具体区别如下：
####① Debug是调试版本，包括的程序信息更多

####② 只有Debug版的程序才能设置断点、单步执行、使用TRACE/ASSERT等调试输出语句

####③ Release不包含任何调试信息，所以文件小、运行速度快

## 查看目标文件生成
![目标文件路径.png](https://upload-images.jianshu.io/upload_images/2959789-9903e5676e763d19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![目标文件.png](https://upload-images.jianshu.io/upload_images/2959789-779bb60bbcb74127.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




```Swift
简单的代码如下，

#if defined(DEBUG)||defined(_DEBUG)

    NSLog(@"------------->>>>>>>  错误的代码段");

    NSLog(@"-------------》》》Test Code");

#endif
```
若定义了Debug模式，就会走逻辑判断中的NSLog的打印，否则不会处理。DEBUG和_DEBUG的判断是来自于Xcode的设置，在Xcode中我们可以取消DEBUG模式，开启RELEASE发布模式如下图：

选择Product一栏>Scheme>Edit Scheme
![1.选择工程文件](https://upload-images.jianshu.io/upload_images/2959789-475a10633f830639.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   

 ![2.选择边界 Scheme](https://upload-images.jianshu.io/upload_images/2959789-dc37df5ac1e4cb5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![3.选择Debug或者Release模式](https://upload-images.jianshu.io/upload_images/2959789-5a20dea8887a2e4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***

>#Archive(打包) 测试版(Debug)或者发布版(Debug)
&emsp; Archive也分为Debug和Release版本，你可以Archive出一个Debug版本的应用也可以Archive出一个Release的应用。直接archive 是系统提供帮助打包的，Archive生成后的文件会小很多。
![打包选择](https://upload-images.jianshu.io/upload_images/2959789-763f8d5252a32fca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要注意一点：
![不选择 Generic iOS Device 无法打包](https://upload-images.jianshu.io/upload_images/2959789-4e9fb9ab78ae5d14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)









