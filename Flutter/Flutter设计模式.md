> <h4/>
- [**Stream管道流-响应式编程**](#Stream管道流-响应式编程)
- []()
	- [Bloc模式](#Bloc模式)
	- [MVVM模式](#MVVM模式)



<br/><br/><br/>

***
<br/>

> <h1 id='Stream管道流-响应式编程'>管道流-响应式编程</h1>
&emsp; **StreamController** 是 Dart 中用于创建和管理 Dart 流（stream）的类。流是一系列异步事件的序列，可以用于处理异步操作，例如处理用户输入、网络请求或其他非同步事件。

<br/>
- **三元素**
	- StreamController：数据流管道
	- Stream.sink：发出消息
	- Stream：收到消息

- **Stream 和 StreamController 的联系**
	- **Stream：** 可以看作是异步数据的“管道”，允许你监听一系列异步事件，比如用户输入、网络请求、计时器事件等。
	- **StreamController：** 则是用来创建和管理Stream的工具。它允许你手动添加数据、错误或结束事件到Stream中。
	- 简单来说，StreamController 负责“控制”或“驱动”一个 Stream：
		- 你可以通过 StreamController.sink 将数据添加到 Stream 中；
		- 然后通过 StreamController.stream 获取到这个 Stream 供其它部分（如 UI 层）监听和响应。

<br/>

- **步骤**
	- 首先在 StreamControlState 里实现 StreamController;
	- StreamSink 通过 _streamController.sink 获取;
	- Stream 通过 _streamController.stream 获取;

**代码:**

```dart
import 'dart:async';

void main() {
  // 创建一个 StreamController，用于处理整数类型的事件
  var controller = StreamController<int>();

  // 获取 StreamSink，用于将事件添加到流中
  var sink = controller.sink;

  // 监听流事件
  var subscription = controller.stream.listen((data) {
    print('Received data: $data');
  });

  // 使用 StreamSink 添加事件到流中
  sink.add(1);
  sink.add(2);
  sink.add(3);

  // 关闭 StreamSink，表示没有更多的事件将被添加
  sink.close();

  // 关闭 StreamSubscription
  subscription.cancel();
}
```

**总结:** 通过 sink 来添加事件到流中。sink 提供了一个类似于 add 的方法，即 sink.add(event)。使用 sink.add 和 sink.close 是为了更清晰地表示我们正在操作 StreamController 的输入端，向流中添加事件。

&emsp; 需要注意的是，当你关闭 sink 时，它也会关闭相应的 stream，这意味着不再允许往流中添加新的事件。在实际应用中，使用 sink 主要是为了在更复杂的异步处理中提供更多的控制和灵活性。

<br/><br/>

&emsp; 然后就可以通过 _sink 发送消息，在 _stream 处接受消息，这里你肯定会比较迷惑，发送一个消息，为什么搞的这么麻烦？

&emsp; 这正是响应式编程的魅力所在，如果直接发送，那么就是同步的，如果要实现异步发送，按照正常的实现，就必须要写很多监听和回调，很容易陷入回调陷阱，而在响应式编程里，我们只需关心 _sink 和 _stream，在 _sink 里发送消息，在 _stream 处接受消息，不需要写额外的监听和回调.

&emsp; StreamController 会帮我们处理，而且在 StreamController 里也可以对接受到的数据处理后在发送。

<br/><br/><br/>

***
<br/>

> <h1 id="Bloc和MVVM">Bloc和MVVM</h1>


> <h2 id='Bloc模式'>Bloc模式</h2>

**1.Bloc图**
![flutter1_5.png](./../Pictures/flutter1_5.png)

BLoC模式由来自Google的Paolo Soares和Cong Hui设计，并在2018年DartConf期间（2018年1月23日至24日）首次展示。

<br/>

- **业务逻辑（Business Logic ）**

	- 转移到一个或几个BLoC，
	- 尽可能从表现层中删除。 换句话说，UI组件应该只关心UI事物而不关心业务
	- 依赖Streams独家使用输入（Sink）和输出（流）
	- 保持平台独立
	- 保持环境独立
	- BLoC模式最初被设想为允许独立于平台重用相同的代码：Web应用程序，移动应用程序，后端。
- Widgets通过Sinks向BLoC发送事件
- BLoC通过Stream通知Widgets，
- 由BLoC实现的业务逻辑不是他们关注的问题。

![flutter1_7.png](./../Pictures/flutter1_7.png)

<br/>

下图是 BLoC 模式里的事件和状态流向图：

![flutter1_8.png](./../Pictures/flutter1_8.png)

- Widget 向 BLoC 发送事件
- 事件会触发 BLoC 里的 sink
- 然后 Stream 会把 State 通知给 Widget

&emsp; 这里的 Event 是为了把 Widget 和具体的业务逻辑分离抽象出来的东西，State 就是 Widget 显示需要用到的数据，也是和业务逻辑分离的。

<br/>

**BLoC实现的业务逻辑层，具有以下的特点：**
- BLoC依赖响应式编程
- 有 Event 和 State

<br/><br/>

BLoC 实现了业务逻辑层和 UI 逻辑的分离，为此带来了巨大的好处：
- 可以用对 App 影响最小的方式修改业务逻辑
- 可以修改 UI，而不用担心影响业务逻辑
- 更加方便单元测试

<br/>

![flutter1_9.png](./../Pictures/flutter1_9.png)

BLoC 模式的架构图，看到这里你觉得和某个模式很像，没错就是 MVVM

共有四层，从上到下分别是：
- UI Screen
- BLoC
- Repository
- Network Provider

Widget 对应的是 MVVM 里的 View，BLoC 对应的是 MVVM 里的 ViewModel，Repository 和 Network Provider 对应的是 MVVM 里的 Model。

<br/>

**简单Demo**

```dart
import 'dart:async';

import 'package:flutter/material.dart';

///Bloc + Stream
class BlocPage extends StatefulWidget {
  @override
  _BlocPageState createState() => _BlocPageState();
}

class _BlocPageState extends State<BlocPage> {
  final PageBloc _pageBloc = new PageBloc();

  @override
  void dispose() {
    _pageBloc.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: new Text("bloc"),
      ),
      body: Container(
        child: new StreamBuilder(
            initialData: 0,
            stream: _pageBloc.stream,
            builder: (context, snapShot) {
              return new Column(
                children: <Widget>[
                  new Expanded(
                      child: new Center(
                          child: new Text(snapShot.data.toString()))),
                  new Center(
                    child: new FlatButton(
                        onPressed: () {
                          _pageBloc.add();
                        },
                        color: Colors.blue,
                        child: new Text("+")),
                  ),
                  new Center(
                    child: new FlatButton(
                        onPressed: () {
                          _pageBloc.dec();
                        },
                        color: Colors.blue,
                        child: new Text("-")),
                  ),
                  new SizedBox(
                    height: 100,
                  )
                ],
              );
            }),
      ),
    );
  }
}


class PageBloc {
  int _count = 0;

  ///StreamController
  StreamController<int> _countController = StreamController<int>();

  ///对外提供入口
  StreamSink<int> get _countSink => _countController.sink;

  ///提供 stream StreamBuilder 订阅
  Stream<int> get stream => _countController.stream;

  void dispose() {
    _countController.close();
  }

  void add() {
    _count++;
    _countSink.add(_count);
  }

  void dec() {
    _count--;
    _countSink.add(_count);
  }
}

```

<br/><br/>
> <h2 id='MVVM模式'>MVVM模式</h2>

![flutter1_6.png](./../Pictures/flutter1_6.png)






---
注释: 0,5339 SHA-256 5eac1aa87b3039ec7aa4fd842ee1d3ac  
@HuangGang <harley.smessage@icloud.com>: 4 7,12 28,11 48,12 129 134 140,6 169 286 332,2 357,6 392,2 395,5 407,3 458,5 480,3 529,4 576,5 625,5 755 841 856,4 1882,40 1931,2 1942,8 3141,4 5259 5282  
...
