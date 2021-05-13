- [**基础**](#基础)
	- [Flutter与原生平台通信](#Flutter与原生平台通信)
	- [渲染流程](#渲染流程)
	- [线程模型](#线程模型)
	- [Future和isolate区别](#Future和isolate区别)
	- [数据库概念](#数据库概念)


<br/>

***
<br/>

> <h1 id="基础">基础</h1>


<br/>
<br/>

> <h2 id="Flutter与原生平台通信"> Flutter与原生平台通信 </h2>

[**双向通信**](https://blog.csdn.net/zl18603543572/article/details/96043692)三种方式：
- BasicMessageChannel 实现 Flutter 与 原生(Android 、iOS)双向通信
- MethodChannel 实现 Flutter 与 原生原生(Android 、iOS)双向通信
- EventChannel 实现 原生原生(Android 、iOS)向Flutter 发送消息
- [**通信CodeDemo**](https://github.com/zhaolongs/Flutter-Android-iOS)



<br/>
<br/>

> <h2 id="渲染流程"> 渲染流程 </h2>

[**渲染流程**](https://juejin.cn/post/6844904122257276936)

Widget、Element、RenderObject如何渲染的？

![Flutter 的架构<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd5.png)

分别对应：

- Embedder层：操作系统底层适配层。
- Engine层：渲染引擎层。
- Framework层：用Dart实现的上层UI SDK。

<br/>

**Framework层：Dart实现的上层UI SDK**
- 实现了：Animation（动画）、Painting（图形绘制）、Gestures（手势操作）等功能，并包装成对应的 api 提供给上层开发者调用。
- 为了保证Flutter所绘制的控件与原生控件风格类似，Flutter封装了Material（对应Android）、Cupertino（对应iOS）风格的UI组件库，供开发者直接使用。


<br/>

 **Engine层：Skia渲染 + DartVM 引擎**
 
- 这层主要包含三块：Skia、Dart、Text。
- Skia 是渲染引擎，为 Framework 层提供“底层渲染”能力。
- Dart 是 Dart 运行时引擎，为 Framework 层提供运行时调用Dart和渲染能力。
- Text 是文字排版，为 Framework 层提供视图排版能力。

<br/>

**Embedder层：操作系统适配**

- 对不同平台操作系统的适配，包括一些配置：surface、线程、插件等特性。
- 由于Flutter相关特性并不多，因此对不同平台操作系统的适配成本很低。



<br/>
<br/>


> <h2 id="线程模型"> 线程模型 </h2>

[**线程模型**](https://xiongcen.github.io/2020/09/01/flutter-thread-model/)

**摘要:**

&emsp; Flutter Engine层面，有四个Runner各司其职，这里的Runner其实就是线程，不过这四个Runner是由Engine和Native之间的那个嵌入层(Embedder)去赋值的，Engine层只会使用这四个Runner，不会创建新的线程。

![线程管理结构图<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd6.jpg)

四个Runner分别是：

- **Platform Task Runner**
	- 对应平台的线程的主线程；
	- Mobile平台所有Engine实例共享同一个Platform Task Runner和线程；
	- 尽管阻塞该线程并不会影响Flutter渲染管道导致页面卡顿，但主线程建议不要执行耗时操作，否则可能触发Watchdog来结束该应用。比如Android、iOS都是使用平台线程来传递用户输入事件，一旦平台线程被阻塞则会引起手势事件丢失。

<br/>

- **UI Task Runner**
	- 对应平台的线程的子线程；
	- 阻塞这个线程会直接导致Flutter应用卡顿掉帧。为了防止阻塞这个线程，我们可以创建其他的Isolate，创建的Isolate没有绑定Flutter的功能，只能做数据运算，不能调用Flutter的功能，而且创建的Isolate的生命周期受Root Isolate控制，Root Isolate停止，其他的Isolate也会停止，而且创建的Isolate运行的线程，是DartVM里的线程池提供的；

<br/>

- **GPU Task Runner**

- 对应平台的线程的子线程；
- 功能：
	- 主要用于执行设备GPU的指令：在UI Task Runner 创建Layer Tree，在GPU Task Runner将Layer Tree提供的信息转化为平台可执行的GPU指令；

- UI Task Runner和GPU Task Runner跑在不同的线程。GPU Task Runner会根据目前帧执行的进度去向UI Task Runner要求下一帧的数据，在任务繁重的时候可能会告诉UI Task Runner延迟任务，例如基于Layer Tree的处理时长和GPU帧显示到屏幕的耗时，GPU Task Runner可能会延迟下一帧在UI Task Runner的调度。这种调度机制确保GPU Task Runner不至于过载，同时也避免了UI Task Runner不必要的消耗；

- 在此线程耗时太久的话，会造成Flutter应用卡顿，所以在GPU Task Runner尽量不要做耗时的任务，例如加载图片的时候，去读取图片数据，就不应该放在GPU Task Runner，而是放在接下来要讲的IO Task Runner；

- 建议为每一个Engine实例都新建一个专用的GPU Task Runner线程。


<br/>


- **IO Task Runner:**
- 对应平台的线程的子线程；

- 功能：
	- 主要功能是从图片存储（比如磁盘）中读取压缩的图片格式，将图片数据进行处理为GPU Task Runner的渲染做好准备。IO Task Runner首先要读取压缩的图片二进制数据（比如PNG，JPEG），将其解压转换成GPU能够处理的格式然后将数据上传到GPU。
- 加载其他资源文件
- 在IO Task Runner不会阻塞Flutter，虽然在加载图片和资源的时候可能会延迟，但是还是建议为IO Task Runner单独开一个线程。



<br/>
<br/>


> <h2 id="Future和isolate区别">Future和isolate区别</h2>

[**Future和isolate区别**](https://www.jianshu.com/p/252fb36ed13d)

Flutter是用Dart实现的，在Dart中没有线程和进程的概念，我们编程使用多线程一般实现两种场景，一种是异步执行，一种是并行执行。

- [**Future原理**](https://www.jianshu.com/p/890df7ea8f87):
	- Future对象（futures）表示异步操作的结果，进程或者IO会延迟完成
	- 可以在async函数中使用await来挂起执行，返回一个Future对象
	- 在async函数中使用try-catch来捕获异常（或者使用catchError()）
	- await只能在async中使用


<br/>

- **Isolate**
	- Dart是一个单线程语言，它的"线程"概念被称为 Isolate，中文意思是隔离。
	
	- 特点：
		- 它与我们之前理解的 Thread 概念有所不同，各个 isolate 之间是无法共享内存空间。
	
		- Isolate是完全是独立的执行线，每个都有自己的 event loop。只能通过 Port 传递消息，所以它的资源开销低于线程。
	
		- Dart中的线程可以理解为微线程。
	
		- Future实现异步串行多个任务；Isolate可以实现异步并行多个任务
	
	- 作用：
		- Flutter的代码都是默认跑在root isolate上的，将非常耗时的任务添加到event loop后，会拖慢整个事件循环的处理，甚至是阻塞。可见基于Event loop的异步模型仍然是有很大缺点的，这时候我们就需要Isolate。
	
	- 使用场景
		- Dart中使用多线程计算的时候，在创建Isolate以及线程间数据传递中耗时要超过单线程，每当我们创建出来一个新的 Isolate 至少需要 2mb 左右的空间甚至更多，因此Isolate有合适的使用场景，不建议滥用Isolate。那么应该在什么时候使用Future，什么时候使用Isolate呢？
		- 一个最简单的判断方法是根据某些任务的平均时间来选择：
		- 方法执行在几毫秒或十几毫秒左右的，应使用Future
		- 如果一个任务需要几百毫秒或之上的，则建议创建单独的Isolate
		- 一些常见的可以参考的场景
			- JSON 解码
			- 数据加密
			- 图像处理
			- 网络请求：加载资源、图片


<br/>
<br/>


> <h2 id="数据库概念">数据库概念</h2>
[**数据库概念**](https://www.imangodoc.com/52359.html)

Flutter提供了许多高级软件包来处理数据库,最重要的软件包是:
- sqflite-用于访问和操作SQLite数据库;
- firebase_database-用于从Google访问和操纵云托管的NoSQL数据库。






