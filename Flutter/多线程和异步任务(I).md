

- **[多线程和异步任务实践](https://blog.happyyun.com/2019/05/26/flutter-多线程和异步任务实践/)**
- **Future**
- 创建Future
	- 打乱顺序执行
	- then函数嵌套使用的执行
	- then链式调用
	- async/await组合
- **协程**




<br/>

***
<br/>

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

&ensp; **then**注册在 Future 完成时调用的回调。
当这个 Future 用一个 value 完成时，将使用该值调用onValue回调。
如果 Future已经完成，则不会立即调用回调，而是将在稍后的microtask（微任务）中调度。
如果回调返回Future，那么then返回的future将与callback返回的future结果相同。

onError回调必须接受一个参数或两个参数，后者是[StackTrace]。

如果onError接受两个参数，则使用错误和堆栈跟踪时调用它，否则仅使用错误对象时候调用它。

onError回调必须返回一个可用于完成返回的future的值或future，因此它必须是可赋值给FutureOr <R>的东西。

返回一个新的Future，该Future是通过调用onValue(如果这个Future是通过一个value完成的)或'onError(如果这个Future是通过一个error完成的)的结果完成的。

如果调用的回调抛出异常，返回的future将使用抛出的错误和错误的堆栈跟踪完成。在onError的情况下，如果抛出的异常与onError的错误参数“相同(identical)”，则视为重新抛出，并使用原始堆栈跟踪替代

如果回调返回Future，则then返回的Future将以与回调返回的Future相同的结果完成。

如果未给出onError，并且后续程序走了刚出现了错误，则错误将直接转发给返回的Future。

在大多数情况下，单独使用catchError更可读，可能使用test参数，而不是在单个then调用中同时处理value和error。

请注意，在添加监听器(listener)之前，future不会延迟报告错误。如果第一个then或catchError调用在future完成后发生error，那么error将报告为未处理的错误。


<br/>

***
<br/>

-  创建Future

```
void testFuture() {
    Future future = new Future(() => null);
    future.then((_) {
      print('then');
    }).then((_){
      print('when Complete');
    }).catchError((_){
      print('catchError');
    });
  }
```
打印：

`flutter: then`

`flutter: when Complete`

&emsp;  `Future`直接new就可以了。没有具体的返回数据，可以用匿名函数代替，` Future future = new Future(() => null)`; 相当于 `Future<Null> future = new Future(() => null);` 泛型如果为null可以省略不写，为了便于维护和管理，开发中建议加上泛型。


<br/>
- Future 输出

```
void testFuture2() {
    Future f1 = Future(()=> print('1'));
    Future f2 = Future(() => print('2'));
    Future f3 = Future(()=> print('3'));
    Future f4 = Future(() => print('4'));
  }
```

打印：
`flutter: 1`

`flutter: 2`

`flutter: 3`

`flutter: 4 `


<br/>

- 相关函数回调
`then`：异步操作逻辑在这里写。
`whenComplete`：异步完成时的回调。
`catchError`：捕获异常或者异步出错时的回调。
因为这里面的异步操作过程中没有遇到什么错误，所以catchError回调不会调用。
> 在我们平时开发中我们是这样用的，首先给我们的函数后面加上async关键字，表示异步操作，然后函数返回值写成Future，然后我们可以new一个Future，逻辑前面加上一个await关键字，然后可以使用future.then等操作。下面是一个示例操作，为了方便演示，这里的返回值的null。平时开发中如果请求网络返回的是json，我们可以把泛型写成String；泛型也可能是实体类（entity/domain），不过要做json转实体类相关操作。
```
 Future asyncDemo() async {
    Future<Null> future = new Future(() => null);
    await future.then((_) {
      print('回调函数 then');
    }).then((_) {
      print('回调函数 when complete');
    }).catchError((_) {
      print('回调函数 cach error');
    });
  }
```

打印：
`flutter: 回调函数 then`
`flutter: 回调函数 when complete`


<br/>
- 多个Future执行
    - 3个Future 执行
    ```
   ///多个Future的执行步骤
   void testFutureCreate1() {
    Future f1 = Future(() => null);
    Future f2 = Future(() => null);
    Future f3 = Future(() => null);

    //都是异步 空实现 顺序进行
    f1.then((_) => print('1'));
    f2.then((_) => print('2'));
    f3.then((_) => print('3'));
  }
    ```
    打印：
    `flutter: 1`
    `flutter: 2`
    `flutter: 3`

   <br/>
   
   - 打乱顺序执行

```
    void testFutureCreate2(){
    Future f2 = new Future(() => null);
    Future f1 = new Future(() => null);
    Future f3 = new Future(() => null);
    
    f1.then((_) => print("1"));
    f2.then((_) => print("2"));
    f3.then((_) => print("3"));
  }
```
    
打印：
`flutter: 2`

`flutter: 1`

`flutter: 3`


<br/>
- 打乱then顺序执行

```
    
  ///打乱顺序执行
  void testFutureCreate3(){

      Future f1 = new Future(() => null);
      Future f2 = new Future(() => null);
      Future f3 = new Future(() => null);

      f3.then((_) => print("3"));
      f1.then((_) => print("1"));
      f2.then((_) => print("2"));

  }
```

打印：

`flutter: 1`

`flutter: 2`

`flutter: 3`

&emsp; 创建多个Future，执行顺序和和创建Future的先后顺序有关，如果只是单独的调用then，没有嵌套使用的话，和调用then的先后顺序无关。


<br/>

- then函数嵌套使用的执行

```
  /// 嵌套执行
  void testThen1 () {
    Future f1 = Future(() => null);
    Future f2 = Future(() => null);
    Future f3 = Future(() => null);

    f1.then((_) => print('f1 -> f2'));

    f2.then((_) {
      print('f2 -> f2');
      f1.then((_) => print('f2.then -> f1'));
    });

    f3.then((_) => print('f3 -> f3'));
  }
```

打印：
`flutter: f1 -> f2`

`flutter: f2 -> f2`

`flutter: f2.then -> f1`

`flutter: f3 -> f3`

<br/>

- 嵌套打乱顺序执行

```
void testThen2() {
    Future f1 = new Future(() => null);
    Future f2 = new Future(() => null);
    Future f3 = new Future(() => null);

    f2.then((_) {
      print("f2 -> f2");
      f1.then((_) => print("f2.then -> f1"));
    });
    f1.then((_) => print("f1 -> f1"));
    f3.then((_) => print("f3 -> f3"));
  }
```

打印：

`flutter: f1 -> f1`

`flutter: f2 -> f2`

`flutter: f2.then -> f1`

`flutter: f3 -> f3`

<br/>

- 嵌套打乱顺序执行2.0

```
void testThen3() {
      Future f1 = new Future(() => null);
      Future f2 = new Future(() => null);
      Future f3 = new Future(() => null);

      f1.then((_) => print("f1 -> f1"));
      f3.then((_) {
        print("f3 -> f3");
        f1.then((_) => print("f3.then -> f1"));
      });
      f2.then((_) => print("f2 -> f2"));
  }
```

`flutter: f1 -> f1`

`flutter: f2 -> f2`

`flutter: f3 -> f3`

`flutter: f3.then -> f1`


<br/>
- 嵌套打乱顺序3.0 

```
void testThen4() {
    Future f1 = new Future(() => null);
    Future f2 = new Future(() => null);
    Future f3 = new Future(() => null);

    f1.then((_) => print("f1 -> f1"));
    f3.then((_) {
      print("f3 -> f3");
      new Future(() => print("f3.then -> new Future"));
      f1.then((_) => print("f3.then -> f1"));
    });
    f2.then((_) => print("f2 -> f2"));
  }
```

打印：

`flutter: f1 -> f1`

`flutter: f2 -> f2`

`flutter: f3 -> f3`

`flutter: f3.then -> f1`

`flutter: f3.then -> new Future`

结论：
首先执行顺序和创建Future的先后顺序有关，如果遇到多个 then 嵌套，先执行外面的  then，然后再执行里面的then，如果then里面还有创建Future，要等到then执行完毕，之后执行Future。


<br/>

- 最后总结Code

```
 ///综合示例
  void testAll() {
    Future f3 = new Future(() => null);
    Future f1 = new Future(() => null);
    Future f2 = new Future(() => null);
    Future f4 = new Future(() => null);
    Future f5 = new Future(() => null);

    f3.then((_) => print("f3.then -> f3"));
    f2.then((_) => print("f2.then -> f2"));
    f4.then((_) => print("f4.then -> f4"));

    f5.then((_) {
      print("f5.then -> f5");
      new Future(() => print("f5.then -> new Future"));
      f1.then((_) {
        print("f1.then -> f1");
      });
    });
  }
```

打印：

`flutter: f3.then -> f3`

`flutter: f2.then -> f2`

`flutter: f4.then -> f4`

`flutter: f5.then -> f5`

`flutter: f1.then -> f1`

`flutter: f5.then -> new Future`

推断出输出结果：

&emsp;  首先按照Future创建顺序应该是 f3 f1 f2 f4 f5依次执行。
&emsp;  我们看到then函数的调用情况，f3先调用，所以它应该先输出。
&emsp;  紧接着是f2调用了，所以f2输出。
&emsp;  紧接着f4调用了，f4应该输出。
&emsp;  紧接着是f5调用then函数，这个比较特殊，它是then函数的嵌套使用，首先是一个打印语句，直接输出，然后是new Future函数，它应该等then执行完毕再去执行，所以这里会去找下面的f1.then里面的逻辑，然后输出f1，到此f5.then都执行完毕，然后就是执行new Future里面的逻辑（如果没有内部嵌套 then的话，它就会直接输出。）



<br/>
- then链式调用

```
Future<String> login(String userName, String pwd) async {
  return '88888888';
}
 
Future<String> getUserInfo(String id) async {
  return '野猿新一是一只Android程序员';
}
 
Future saveUserInfo(String userInfo) async {
  print(userInfo);
}



//调用
void main() {
  login('野猿新一', '1234556').then((id){
    print('登录成功，用户id为${id}');
    return getUserInfo(id);
  }).then((userInfo){
    print('获取用户信息成功，结果为${userInfo}');
    return saveUserInfo(userInfo);
  }).then((data){
    print('保存用户信息成功');
  });
}

```

打印： 

```
登录成功，用户id为88888888
获取用户信息成功，结果为野猿新一是一只Android程序员
野猿新一是一只Android程序员
保存用户信息成功

```


<br/>

- async/await组合

**注意：** 
- 在调用异步方法的前面多加了一个await关键字，表示这是异步返回，等该异步任务执行成功了才会执行下一行代码
- awsit异步调用只能在异步方法里面，否则会报"The await expression can only be used in an async function."错误，如下的task()方法后面有一个async关键字就是一个异步方法
- 异步的任务难免会发生异常，所以建议用try/catch代码块包裹起来，在catch里面处理异常

```
void main() {
  task();
}
 
void task() async {
  try {
    String id = await login('野猿新一', '1234556');
    print('登录成功，用户id为${id}');
    String userInfo = await getUserInfo(id);
    print('获取用户信息成功，结果为${userInfo}');
    saveUserInfo(userInfo);
    print('保存用户信息成功');
  } catch (e) {
    print(e);
  }
}
```

打印：

```
登录成功，用户id为88888888
获取用户信息成功，结果为野猿新一是一只Android程序员
野猿新一是一只Android程序员
保存用户信息成功

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


















