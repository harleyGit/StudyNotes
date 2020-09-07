># Stream 分类
-  **`Single-subscriptio`** :  单订阅流;
- **`broadcast`**:   广播式的流（可多订阅）;
- 如果一个流是单订阅模式 却想多次订阅，可以通过`asBroadcastStrea()`方法来修改。

&emsp;  单订阅流只能被订阅一次，重复订阅会报错， 直到设置listen 后才会发送。单订阅流通常用于流式数据块较大的连续数据，如文件I/O;
&emsp;  广播式的流可以订阅多次，在listen之前的数据会丢失。


<br/>
&emsp;  为了方操作 Stream ，官方提供了StreamController,如下图所示，StreamSink来添加流 （入口），同时提供 stream 属性用于对外的监听和变换。 
&emsp;  stream.listen的返回一个StreamSubscription，可以通过它的`pause(), resume(), cancel()`等方法来操作流的订阅。


![Stream 处理流的模型](https://upload-images.jianshu.io/upload_images/2959789-e611b41ee21bd1b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


-  **`StreamController:`**
// 创建一个单订阅流
`StreamController controller = StreamController<String>(); `
// 创建一个广播式的订阅流
`StreamController controller = StreamController.broadcast(); `
` listen ：` 用来设置监听， 它的返回值是 StreamSubscribe。

-  **`StreamSubscribe`**
` pause()` ： 暂停监听（是立即暂停），暂停后的事件流不会丢失，会在resume后一起回调;
`resume()`： 唤醒pause的流
`cancel()`： 取消

<br/>
- ###单订阅
```
         ///定义一个Controller
          StreamController<List<String>> _dataController = StreamController<List<String>>();
          ///获取 StreamSink 做 add 入口
          StreamSink<List<String>> _dataSink = _dataController.sink;
          ///获取 Stream 用于监听
          Stream<List<String>> _dataStream =  _dataController.stream;
          ///事件订阅对象
          StreamSubscription _dataSubscription = _dataStream.listen((value){
            ///do change
            print('监听值为：${value}');
          });
          ///改变事件
          _dataSink.add(["first", "second", "three", "more"]);
```
打印：![输出监听值](https://upload-images.jianshu.io/upload_images/2959789-5ac8833fa3c8c3bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`其他的`
<br/>
```
          StreamController controller = StreamController<String>();
          StreamSink sink = controller.sink;
          Stream stream = controller.stream;

          stream.transform(StreamTransformer<String, String>.fromHandlers(handleData: (String data, EventSink<String> sink) {
            if (!data.contains('数据2')) {
              sink.add(data);
            }
          })).listen((event) {
            print('接受到的数据是： ${event}');
          });
          sink.add('3秒后才设置监听。');
```
打印：
`flutter: 接受到的数据是： 3秒后才设置监听。`


<br/>
```
          StreamController controller = StreamController<String>();
          StreamSink sink = controller.sink;
          Stream stream = controller.stream;


          StreamSubscription subscription = controller.stream.transform(StreamTransformer<String, String>.fromHandlers(handleData: (String data, EventSink<String> sink){
            print('transform');
            if (!data.contains('数据3')) {
              sink.add(data);
            }
          })).listen((event) {
            print('接受到的数据是： ${event}');
          }); 
          sink.add('我是一条新的消息');

```
打印：
`flutter: 接受到的数据是： 我是一条新的消息`





<br/>
- ### 广播类Stream
```
// 初始化一个int类型的广播Stream controller
  final StreamController<int> ctrl = StreamController<int>.broadcast();
  
  // 初始化一个监听，同时通过transform对数据进行简单处理
  final StreamSubscription subscription = ctrl.stream
                          .where((value) => (value % 2 == 0))
                          .listen((value) => print('监听：$value'));

  // 往Stream中添加数据
  for(int i=1; i<11; i++){
    ctrl.sink.add(i);
  }
  
  // StreamController用完后需要释放
  ctrl.close();
```
打印：
![控制台打印](https://upload-images.jianshu.io/upload_images/2959789-811d227510103818.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






