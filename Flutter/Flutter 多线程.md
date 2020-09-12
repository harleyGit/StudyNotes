# Flutter 多线程

># Future 

&emsp;	Future就是延时操作的一个封装，可以将异步任务封装为Future对象。获取到Future对象后，最简单的方法就是用await修饰，并等待返回结果继续向下执行。正如上面async、await中讲到的，使用await修饰时需要配合async一起使用。

&emsp;	在Dart中，和时间相关的操作基本都和Future有关，例如延时操作、异步操作等。
```
loadData() {
    // DateTime.now()，获取当前时间
    DateTime now = DateTime.now();
    print('request begin $now');
    Future.delayed(Duration(seconds: 1), (){
      now = DateTime.now();
      print('request response $now');
    });
}

```

&emsp;	Dart还支持对Future的链式调用，通过追加一个或多个then方法来实现，这个特性非常实用。例如一个延时操作完成后，会调用then方法，并且可以传递一个参数给then。调用方式是链式调用，也就代表可以进行很多层的处理。这有点类似于iOS的RAC框架，链式调用进行信号处理。

```
Future.delayed(Duration(seconds: 1), (){
  int age = 18;
  return age;
}).then((onValue){
  onValue++;
  print('age $onValue');
});


```


<br/>

***
<br/>


># 协程

&emsp;	如果想要了解async、await的原理，就要先了解协程的概念，async、await本质上就是协程的一种语法糖。协程，也叫作coroutine，是一种比线程更小的单元。如果从单元大小来说，基本可以理解为`进程->线程->协程。`

- isolate
&emsp;	isolate是Dart平台对线程的实现方案，但和普通Thread不同的是，isolate拥有独立的内存，isolate由线程和独立内存构成。正是由于`isolate线程之间的内存不共享，所以isolate线程之间并不存在资源抢夺的问题，所以也不需要锁。`
&emsp;	通过isolate可以很好的利用多核CPU，来进行大量耗时任务的处理。isolate线程之间的通信主要通过port来进行，这个port消息传递的过程是异步的。通过Dart源码也可以看出，实例化一个isolate的过程包括，实例化isolate结构体、在堆中分配线程内存、配置port等过程。
&emsp;	isolate看起来其实和进程比较相似，'isolate的整体模型我自己的理解其实更像进程，而async、await更像是线程'。若对比一下isolate和进程的定义，会发现确实isolate很像是进程。

&emsp; 创建 isolate，并对其进行网络请求demo：
```
loadData() async {
    // 通过spawn新建一个isolate，并绑定静态方法
    ReceivePort receivePort =ReceivePort();
    await Isolate.spawn(dataLoader, receivePort.sendPort);
    
    // 获取新isolate的监听port
    SendPort sendPort = await receivePort.first;
    // 调用sendReceive自定义方法
    List dataList = await sendReceive(sendPort, 'https://jsonplaceholder.typicode.com/posts');
    print('dataList $dataList');
}

// isolate的绑定方法
static dataLoader(SendPort sendPort) async{
    // 创建监听port，并将sendPort传给外界用来调用
    ReceivePort receivePort =ReceivePort();
    sendPort.send(receivePort.sendPort);
    
    // 监听外界调用
    await for (var msg in receivePort) {
      String requestURL =msg[0];
      SendPort callbackPort =msg[1];
    
      Client client = Client();
      Response response = await client.get(requestURL);
      List dataList = json.decode(response.body);
      // 回调返回值给调用者
      callbackPort.send(dataList);
    }    
}

// 创建自己的监听port，并且向新isolate发送消息
Future sendReceive(SendPort sendPort, String url) {
    ReceivePort receivePort =ReceivePort();
    sendPort.send([url, receivePort.sendPort]);
    // 接收到返回值，返回给调用者
    return receivePort.first;
}


```

-	任务调度
&emsp;	在弄懂协程之前，首先要明白并发和并行的概念，并发指的是由系统来管理多个IO的切换，并交由CPU去处理。并行指的是多核CPU在同一时间里执行多个任务。
&emsp;	并发的实现由非阻塞操作+事件通知来完成，事件通知也叫做“中断”。操作过程分为两种，一种是CPU对IO进行操作，在操作完成后发起中断告诉IO操作完成。另一种是IO发起中断，告诉CPU可以进行操作。
&emsp;	线程本质上也是依赖于中断来进行调度的，线程还有一种叫做“阻塞式中断”，就是在执行IO操作时将线程阻塞，等待执行完成后再继续执行。但线程的消耗是很大的，并不适合大量并发操作的处理，而通过单线程并发可以进行大量并发操作。当多核CPU出现后，单个线程就无法很好的利用多核CPU的优势了，所以又引入了线程池的概念，通过线程池来管理大量线程。

-	协程
&emsp;	在程序执行过程中，离开当前的调用位置有两种方式，继续调用其他函数和return返回离开当前函数。但是执行return时，当前函数在调用栈中的局部变量、形参等状态则会被销毁。
&emsp;	协程分为无线协程和有线协程，无线协程在离开当前调用位置时，会将当前变量放在堆区，当再次回到当前位置时，还会继续从堆区中获取到变量。所以，一般在执行当前函数时就会将变量直接分配到堆区，而async、await就属于无线协程的一种。有线协程则会将变量继续保存在栈区，在回到指针指向的离开位置时，会继续从栈中取出调用。


















