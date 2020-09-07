># Future 源码说明
我们来看看Future的源码说明文档

我们重点看看then函数的文档说明：

then注册在 Future 完成时调用的回调。
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














