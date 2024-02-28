

> <h2 id=''></h2>
- [**关键字**](#关键字)
	- [late](#late)
- [**方法**](#方法)
	- [addPostFrameCallback](#addPostFrameCallback)
	- [showGeneralDialog](#showGeneralDialog)
	- [dependOnInheritedWidgetOfExactType](#dependOnInheritedWidgetOfExactType)
	- [函数作为参数](#函数作为参数)
		- [函数回掉](#函数回掉)
- [**Theme**](#Theme)
- [**ChangeNotifierProvider**](#ChangeNotifierProvider)
- [**Stream流**](#Stream流)
	- [Stream.fromFuture](#Stream.fromFuture)
	- [asBroadcastStream方法](#asBroadcastStream方法)
- [**应用状态恢复RestorationMixin**](#应用状态恢复RestorationMixin)
	- [RestorableInt详解](#RestorableInt详解)
- [**PageController()**](#PageController())
- [**打印**](#打印)
	- [debugPrint](#debugPrint)
- [**工具类**](#工具类)
	- [图片工具](#图片工具)
- [**组件库**](#组件库)
	- [Dio网络库](#Dio网络库)
		- [扩展拦截器类](#扩展拦截器类)
		- [CancelToken](#CancelToken)
	- [fluro路由组件](#fluro路由组件)
	- [cached_network_image-图片加载](#cached_network_image-图片加载)
	- 	[flutter_ume调试工具](#flutter_ume调试工具)
	- [common_utils-工具库](#common_utils-工具库)
	- [keyboard_actions处理键盘事件](#keyboard_actions处理键盘事件)
	- [sp_util-保存对象](#sp_util-保存对象)
	- [sprintf格式化字符串](#sprintf格式化字符串)
- **参考资料**
	- [**flutter_deer**](https://github.com/simplezhli/flutter_deer)
	- [Flutter web真机测试](https://juejin.cn/post/7091184835842015268)
	- [Deer](https://weilu.blog.csdn.net/article/details/90546727)项目知识点集合




<br/>

***
<br/><br/>

> <h1 id='关键字'>关键字</h1>


<br/><br/>

> <h2 id='late'>late</h2>

late关键字是Dart语言中的一个特性，它用于标记一个变量的延迟初始化。当你使用late关键字修饰一个变量时，Dart编译器会允许你推迟该变量的初始化，直到该变量首次被访问。这在一些情况下很有用，例如在构造函数中初始化变量可能不方便或不可行的情况下。

```
class Example {
  late String lateInitializedString; // 声明一个延迟初始化的字符串变量

  Example() {
    // 在构造函数中无法立即初始化变量
    // 但是可以在稍后的代码中初始化
    lateInitializedString = 'Initialized later';
  }

  void printString() {
    print(lateInitializedString);
  }
}

void main() {
  var example = Example();
  example.printString(); // 输出：Initialized later
}
```

在上面的例子中，lateInitializedString是一个延迟初始化的字符串变量。在构造函数中，它没有被立即初始化，但是在稍后的代码中进行了初始化。late关键字告诉编译器，虽然变量没有被立即初始化，但在首次访问时将会被初始化。


请注意，如果在访问late变量之前尝试访问它，或者在初始化之前访问它，会导致运行时错误。因此，使用late时要确保在访问变量之前进行初始化。


<br/>

***
<br/><br/>


> <h1 id='方法'>方法</h1>


<br/>

> <h2 id='addPostFrameCallback'>addPostFrameCallback</h2>



&nbsp;用法：`addPostFrameCallback`回调方法在Widget渲染完成时触发，所以一般我们在获取页面中的Widget大小、位置时使用到。

&emsp;  若我们在接口请求前弹出loading。例如我将请求方法放在了initState方法中，可能出现的异常如下：

```
inheritFromWidgetOfExactType(_InheritedTheme) or inheritFromElement() was called before initState() completed.
When an inherited widget changes, for example if the value of Theme.of() changes, its dependent widgets are rebuilt. If the dependent widget’s reference to the inherited widget is in a constructor or an initState() method, then the rebuilt dependent widget will not reflect the changes in the inherited widget.
Typically references to inherited widgets should occur in widget build() methods. Alternatively, initialization based on inherited widgets can be placed in the didChangeDependencies method, which is called after initState and whenever the dependencies change thereafter.
```

&emsp;  所以我们需要在build完成后，才可以去创建这个新的组件（这里就是Dialog）。

&emsp;  所以解决方法就是使用addPostFrameCallback回调方法，等待页面build完成后在请求数据：

```
  @override
  void initState() {
    WidgetsBinding.instance.addPostFrameCallback((_){
      /// 接口请求
    });
  }
``` 

&emsp;  导致这类问题的场景很多，但是大体解决思路就是上述的[办法](https://weilu.blog.csdn.net/article/details/94849020)。



<br/><br/>

># <h2 id='showGeneralDialog'>[showGeneralDialog](https://blog.csdn.net/Calvin_zhou/article/details/115768013)</h2>

展示弹窗




<br/><br/>


> <h2 id='dependOnInheritedWidgetOfExactType'> dependOnInheritedWidgetOfExactType </h2>


BuildContext.dependOnInheritedWidgetOfExactType:该方法用于在 widget 树上获取离当前 widget 最近的一个父级InheritedWidget



<br/><br/>

> <h2 id='函数作为参数'>函数作为参数</h2>


- 1.在调用页面，声明一个作为参数的函数tap

```
tap(int i){
    print('选择了....'+i.toString());
}
```


<br/>

- 2、声明自定义的函数,设置形参,点击调用参数

```
InkWell formateItem( Function(int) tapAction){
  child: Container(),
  onTap: () {
      tapAction(1688);
    }
}
```

<br/>

- 3、调用，将tapAction传入

```
formateItem(tap)
```

<br/>

- 4、打印

```
Flutter: 选择了 1688
```



<br/><br/>

> <h2 id='函数回掉'>函数回掉</h2>

&emsp; 使用 typedef 关键字定义一个回调函数类型，然后将这个类型用作一个函数的参数类型。这允许你传递一个符合指定类型的函数作为参数，实现回调的机制。

```
typedef NetSuccessCallback<T> = void Function(T data);

void fetchData(NetSuccessCallback<String> onSuccess) {
  // 模拟异步操作
  Future.delayed(Duration(seconds: 2), () {
    String result = 'Data from network';
    onSuccess(result); // 调用回调函数，并传递数据
  });
}

void main() {
  // 调用fetchData，并传递回调函数
  fetchData((data) {
    print('Network request successful: $data');
  });
}
```

&emsp; NetSuccessCallback 被定义为一个接受泛型类型 T 的回调函数类型。然后，fetchData 函数接受一个类型为 NetSuccessCallback<String> 的回调函数作为参数。在 fetchData 内部，当异步操作完成时，会调用传递进来的回调函数，并传递数据。

&emsp; 在 main 函数中，我们调用了 fetchData 并传递了一个匿名函数作为回调函数。这个匿名函数打印了成功获取的数据。

&emsp; 这样，通过 typedef 和泛型，你可以定义灵活的回调函数类型，并在需要时传递相应类型的回调函数。



<br/>

***
<br/><br/>




> <h1 id='Theme'>Theme</h1>

主题统一颜色和字体风格

使用:

```
extension ThemeExtension on BuildContext {
  bool get isDark => ThemeUtils.isDark(this);
  //Theme 主题: https://flutter.cn/docs/cookbook/design/themes
  Color get backgroundColor => Theme.of(this).scaffoldBackgroundColor;
  Color get dialogBackgroundColor => Theme.of(this).canvasColor;
}
```






<br/>

***
<br/><br/>

> <h1 id='ChangeNotifierProvider'>ChangeNotifierProvider</h1>


```
class HomeProvider extends RestorableInt {
  //HomeProvider的构造函数通过super(0)调用了父类RestorableInt的构造函数，将初始值设置为0。这是RestorableInt的一种使用方式，它可以用于保存和还原整数类型的状态。
  HomeProvider() : super(0);
}
 
 HomeProvider provider = HomeProvider();

ChangeNotifierProvider<HomeProvider>(
      create: (_) => provider,
);
```

- 1.创建 HomeProvider 实例： 在这段代码的开头，创建了一个 HomeProvider 类的实例，并将其命名为 provider。这个实例是一个用于保存和还原整数状态的RestorableInt子类，初始值为0。

- 2.ChangeNotifierProvider： ChangeNotifierProvider 是 provider 包中的一个类，用于将 ChangeNotifier（包括其子类）引入到 Flutter 的 Provider 架构中，使其能够在小部件树中共享状态。

- 3.create 参数： 在 ChangeNotifierProvider 中，通过 create 参数提供一个回调函数，该回调函数负责创建和返回要提供的 ChangeNotifier 的实例。在这里，使用匿名函数 _（代表不使用传入的 BuildContext）来返回之前创建的 HomeProvider 实例。

- 4.用途和特殊性： 使用 ChangeNotifierProvider 的主要目的是在 Flutter 应用程序中管理状态，并在需要时通知侦听器。HomeProvider 继承自 RestorableInt，这意味着它可以与 RestorationMixin 一起使用，以便在整个应用程序的不同部分之间保存和还原状态。


&emsp; 总体来说，ChangeNotifierProvider 在提供 HomeProvider 实例时，允许在小部件树中共享状态，并且当 HomeProvider 的状态发生变化时，相关的小部件能够得到通知并重新构建。这种方式是一种在 Flutter 中管理状态的常见模式。


<br/>

案例Demo:

```
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 定义一个 ChangeNotifier 类
class CounterNotifier extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  // 定义一个方法，用于增加计数器值
  void increment() {
    _count++;
    // 通知侦听器，状态发生变化
    notifyListeners();
  }
}

void main() {
  runApp(
    // 使用 ChangeNotifierProvider 包裹 MyApp
    ChangeNotifierProvider(
      create: (_) => CounterNotifier(), // 创建 CounterNotifier 实例
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 从 Provider 中获取 CounterNotifier 实例
    CounterNotifier counterNotifier = Provider.of<CounterNotifier>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Counter Example'),
      ),
      body: Center(
        // 使用 Consumer 包裹需要依赖状态的小部件
        child: Consumer<CounterNotifier>(
          builder: (context, counter, child) {
            return Text(
              'Count: ${counter.count}',
              style: TextStyle(fontSize: 24),
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 调用 CounterNotifier 中的方法来增加计数器值
          counterNotifier.increment();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}

```

**详解:**

- 1.定义 CounterNotifier 类： 这是一个继承自 ChangeNotifier 的类，其中包含一个计数器值 _count 和一个方法 increment() 用于增加计数器值。

- 2.main 函数： 在 main 函数中，我们使用 ChangeNotifierProvider 包裹 MyApp，并通过 create 参数创建一个 CounterNotifier 的实例。这样，CounterNotifier 的实例就能够在整个应用程序中共享了。

- 3.MyHomePage 小部件： 在 MyHomePage 中，我们使用 Consumer 小部件包裹了需要依赖状态的部分。在这里，我们依赖于 CounterNotifier 的实例，然后在 builder 回调中根据 CounterNotifier 中的状态构建 UI。

- 4.floatingActionButton： 点击悬浮按钮会调用 counterNotifier.increment() 方法来增加计数器值。由于我们在 increment 方法中调用了 notifyListeners()，Consumer 中的 UI 就会被通知进行更新。

<br/>

&emsp; 这样，通过使用 ChangeNotifierProvider 和 Consumer，我们实现了一个简单的计数器应用程序，其中计数器的状态能够在整个应用程序中共享，并且 UI 在状态发生变化时能够自动更新。


<br/>
<br/>

**疑问1:** 如何通过`CounterNotifier counterNotifier = Provider.of<CounterNotifier>(context);`
就可以获取runApp中创建的CounterNotifier()实例,如何做到的?


**回答:** Provider.of获取实例： 在小部件树的某个地方，你可以使用Provider.of<CounterNotifier>(context)来获取CounterNotifier的实例。这是通过Flutter上下文context向上查找小部件树中最近的ChangeNotifierProvider，并返回其提供的CounterNotifier实例


<br/>
<br/>


**疑问1.1:** 怎么确定通过Flutter上下文context向上查找小部件树中最近的ChangeNotifierProvider就是我们要找的那个? 若是期间有其他的ChangeNotifierProvider实例该怎么办?如何处理

&emsp; 当你使用Provider.of<T>(context)时，Flutter会从给定的BuildContext开始，向上查找小部件树，以找到最近的ChangeNotifierProvider<T>。这是通过InheritedWidget的机制来实现的。ChangeNotifierProvider是InheritedWidget的一个子类，它将ChangeNotifier提供给小部件树，并通过BuildContext的inheritFromWidgetOfExactType方法来查找。

&emsp; 如果在小部件树中有多个ChangeNotifierProvider<T>实例，Provider.of<T>(context)将返回最近的那个。这是因为inheritFromWidgetOfExactType方法会从当前BuildContext开始，向上查找并返回最近的匹配类型的InheritedWidget。

示例代码：

```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<CounterNotifier>(
      create: (_) => CounterNotifier(),
      child: MaterialApp(
        home: MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 获取最近的ChangeNotifierProvider<CounterNotifier>实例
    CounterNotifier counterNotifier = Provider.of<CounterNotifier>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Provider Example'),
      ),
      body: Center(
        child: Text('Counter Value: ${counterNotifier.counter}'),
      ),
    );
  }
}
```

&emsp; 在这个例子中，MyHomePage小部件通过Provider.of<CounterNotifier>(context)获取最近的ChangeNotifierProvider<CounterNotifier>实例。如果在小部件树中有多个ChangeNotifierProvider<CounterNotifier>实例，它将获取到最近的那个。

&emsp; 如果在小部件树中存在多个相同类型的ChangeNotifierProvider，并且你想要访问不同的实例，你可以使用Provider.of<CounterNotifier>(context, listen: false)，这样就不会订阅更新。这样，即使有多个相同类型的ChangeNotifierProvider，它将返回最近的那个实例而不会导致订阅。


<br/>
<br/>


**疑问1.2**: 可以通过某个标识符来找到我要的那个CounterNotifier实例对象吗? 如何做

在使用 ChangeNotifierProvider 的时候，你可以通过给 ChangeNotifierProvider 提供一个独一无二的标识符 key 来创建一个具有唯一标识的实例。这样，在使用 Provider.of<T>(context) 时，你可以使用 key 来确保获取到你想要的特定实例。


```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider<CounterNotifier>(
          key: UniqueKey(), // 使用 UniqueKey() 创建唯一的 key
          create: (_) => CounterNotifier(),
        ),
        // 可以添加其他 ChangeNotifierProvider 或其他 Provider
      ],
      child: MaterialApp(
        home: MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 使用 key 来获取特定的 CounterNotifier 实例
    CounterNotifier counterNotifier = Provider.of<CounterNotifier>(
      context,
      listen: false,
      key: UniqueKey(), // 使用相同的 UniqueKey
    );

    return Scaffold(
      appBar: AppBar(
        title: Text('Provider Example'),
      ),
      body: Center(
        child: Text('Counter Value: ${counterNotifier.counter}'),
      ),
    );
  }
}

```

&emsp; 在这个例子中，ChangeNotifierProvider<CounterNotifier> 的实例通过 UniqueKey() 创建了一个唯一的 key。在 Provider.of<CounterNotifier>(context, key: UniqueKey()) 中，使用相同的 UniqueKey 来获取具有相同标识符的特定 CounterNotifier 实例。

&emsp; 请注意，key 的唯一性是由 UniqueKey 提供的，你也可以使用其他方式来确保唯一性。如果你有其他的标识符需要使用，你也可以使用自定义的标识符。








<br/>

***
<br/><br/>

> <h1 id='Stream流'>Stream流</h1>


- **Stream 是一个重要的概念，用于处理异步事件。以下是一些关键概念和用途：**

	- 异步事件序列： Stream 表示的是一个异步事件序列，可以包含多个事件，这些事件不一定是同时发生的。
	
	- 事件处理： 开发者可以通过监听 Stream 来捕获和处理事件。监听器（Listener）通过注册回调函数，可以在事件到达时执行相应的操作。
	
	- 非阻塞： 使用 Stream 的优势之一是非阻塞式的事件处理。当一个事件到达时，系统不会等待事件处理完毕，而是继续执行其他任务。
	
	- 异步编程： Stream 在处理异步编程任务时非常有用，比如从网络获取数据、文件I/O、定时器等。
	
	- 在Flutter中，许多操作都返回 Stream 对象，比如网络请求、用户输入、定时器等。使用 Stream 可以更灵活地响应这些异步事件。例如，你可以使用 StreamBuilder 在Flutter中构建UI，根据 Stream 中的事件动态更新用户界面。


简单例子:

```
import 'dart:async';

void main() {
  // 创建一个定时器，每秒发送一个事件
  Stream<int> timerStream = Stream.periodic(Duration(seconds: 1), (count) => count);

  // 监听Stream的事件
  timerStream.listen((count) {
    print('Timer count: $count');
  });
}

```

&emsp; Stream.periodic 创建了一个定时器，每秒产生一个事件，然后通过 listen 监听这个 Stream，在每个事件到达时输出计数信息




<br/>
<br/>


<br/><br/>

> <h2 id='Stream.fromFuture'>Stream.fromFuture</h2>

Stream.fromFuture 是一个用于创建 Stream 对象的方法，该方法从一个 Future 对象中生成一个单一事件的 Stream。它允许你将一个异步操作（Future）转换为一个可以监听的事件流（Stream）


```
import 'dart:async';

void main() {
  // 创建一个Future对象，模拟异步操作
  Future<String> fetchData() async {
    await Future.delayed(Duration(seconds: 2));
    return 'Data from Future';
  }

  // 使用Stream.fromFuture创建Stream
  Stream<String> dataStream = Stream.fromFuture(fetchData());

  // 监听Stream的事件
  dataStream.listen((data) {
    print('Received data: $data');
  }, onDone: () {
    print('Stream closed');
  }, onError: (error) {
    print('Error: $error');
  });
}

```

&emsp; 在上面的例子中，fetchData 是一个异步函数，返回一个 Future<String>。然后，使用 Stream.fromFuture 创建一个 Stream<String> 对象，该流将在 Future 完成时发出一个事件，事件的数据即为 Future 的结果。

&emsp; 接着，我们通过 listen 方法来监听 dataStream 的事件。当 Future 完成时，会触发 onData 回调，打印接收到的数据。如果发生错误，将触发 onError 回调。最后，当流关闭时，会触发 onDone 回调。

&emsp; 这是一个简单的例子，实际中你可能会在UI层中使用 StreamBuilder 或者其他方法来更方便地处理 Stream


<br/><br/>

> <h3 id='asBroadcastStream方法'>asBroadcastStream方法</h3>

 Stream 的 .asBroadcastStream() 方法用于将一个单订阅（single-subscription）的 Stream 转换为广播（broadcast）的 Stream。广播流可以被多个监听器（listeners）订阅，而不是单个订阅

&emsp; 一旦通过 .asBroadcastStream() 转换为广播流，你可以使用 .listen 方法来添加多个监听器 

```
import 'dart:async';

void main() {
  // 创建一个单订阅的Stream
  Stream<int> singleSubscriptionStream = Stream.fromIterable([1, 2, 3, 4, 5]);

  // 将单订阅的Stream转换为广播Stream
  Stream<int> broadcastStream = singleSubscriptionStream.asBroadcastStream();

  // 第一个监听器
  broadcastStream.listen(
    (data) => print('Listener 1 received data: $data'),
    onDone: () => print('Listener 1 done'),
  );

  // 第二个监听器
  broadcastStream.listen(
    (data) => print('Listener 2 received data: $data'),
    onDone: () => print('Listener 2 done'),
  );
}

```

&emsp; 在上面的例子中，singleSubscriptionStream 是一个单订阅的 Stream，然后通过 .asBroadcastStream() 方法将其转换为广播流 broadcastStream。

&emsp; 接着，我们添加了两个监听器，分别通过 .listen 方法监听 broadcastStream。这样，两个监听器都可以独立地接收来自 broadcastStream 的事件。当事件触发时，两个监听器的回调函数都会执行。

&emsp; 需要注意的是，使用广播流时要确保适当管理监听器，以避免内存泄漏。当不再需要监听器时，最好使用 cancel 或者 onDone 回调来取消订阅。



<br/>

***
<br/><br/>
> <h1 id='应用状态恢复RestorationMixin'>应用状态恢复RestorationMixin</h1>

```
/**
 * 通过将 RestorationMixin 添加到 _HomeState，你可以利用 Flutter 的状态保存和恢复功能，以便在应用程序重新启动时恢复 widget 的状态。
 * 这对于保留用户界面的滚动位置、文本字段内容等非常有用
 * 
 *  with RestorationMixin： 这表示 _HomeState 类使用了 RestorationMixin mixin。with 关键字用于在 Dart 中混合 mixin，它允许一个类获取另一个类的功能，而不需要继承整个类层次结构。
*/
class _HomeState extends State<Home> with RestorationMixin{

	. . . 
	. . 
	.
}
```


<br/>

&emsp; RestorationMixin 是 Flutter 中的一个 mixin，用于支持应用程序的状态恢复（state restoration）。它提供了一种简单的方法，使您的 Flutter 应用程序能够在发生中断（例如应用程序被销毁并重新创建）时保留其状态。

&emsp; 当用户通过切换应用程序、旋转设备或通过其他手段导致应用程序重新启动时，RestorationMixin 允许应用程序尽可能地恢复到之前的状态，以提供更好的用户体验。

&emsp; 以下是一些关键的概念和方法，以更详细地说明 RestorationMixin 的使用：

**1.状态标识符（RestorationId）**： 使用 restorationId 属性为要保留其状态的组件分配唯一的标识符。这个标识符在整个应用程序中必须是唯一的。

```
class MyStatefulWidget extends StatefulWidget with RestorationMixin {
  @override
  String get restorationId => 'my_stateful_widget';

  // ...
}
```


<br/>

**2.状态保存和恢复**： 使用 registerForRestoration 方法将状态保存和恢复函数注册到组件中。这些函数定义了在状态保存和恢复期间执行的操作。


```
@override
void registerForRestoration(RestorationManager manager, String restorationId) {
  manager.registerForRestoration(this, restorationId);
}

@override
void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
  // 在此处恢复状态
}

@override
void saveState(RestorationBucket? oldBucket) {
  // 在此处保存状态
}

```

在 restoreState 中，您可以从 oldBucket 中提取以前保存的状态。在 saveState 中，您可以将当前状态保存到 oldBucket


<br/>

3.**RestorationScope：** 在应用程序的根级别创建一个 RestorationScope，以便管理整个应用程序的状态。

RestorationMixin 在整个 widget 树中查找 

RestorationScope，以确保状态保存和恢复的一致性。


```
void main() {
  runApp(
    RestorationScope(
      restorationId: 'root_restoration_scope',
      child: MyApp(),
    ),
  );
}

```

这是一个简单的示例，用于说明 RestorationMixin 的基本概念。要使用它，您可能需要更深入地了解 Flutter 的状态管理和组件生命周期。有关更详细的信息，请查看 Flutter 官方文档中关于 RestorationMixin 的内容。



<br/><br/>

> <h2 id='RestorableInt详解'>RestorableInt详解</h2>


&emsp; 在Flutter中，RestorableInt是用于恢复状态的一个类，通常用于保持和还原整数类型的状态。它是RestorableProperty的一个子类，RestorableProperty是用于保存和还原状态的基础类。RestorableInt用于在小部件树中的不同位置传递和保存整数值，并且能够在小部件树重建时保持其值。

以下是RestorableInt的主要特性和使用方法：

**1.创建和初始化：** 可以通过RestorableInt的构造函数创建一个实例，并在构造函数中指定初始值。例如：

```
RestorableInt myRestorableInt = RestorableInt(0);
```


**2.获取和设置值：** 通过value属性可以获取和设置RestorableInt的当前值。

```
int currentValue = myRestorableInt.value;
myRestorableInt.value = 42;
```


**3.保存和恢复状态：** RestorableInt会自动参与Flutter的状态保存和恢复机制。当小部件树需要保存其状态时，Flutter会调用saveState方法来获取当前状态，并在需要恢复状态时调用restoreState方法


```
import 'package:flutter/widgets.dart';

class HomeProvider extends RestorableInt {
	//HomeProvider的构造函数通过super(0)调用了父类RestorableInt的构造函数，将初始值设置为0。
	//这是RestorableInt的一种使用方式，它可以用于保存和还原整数类型的状态。
	HomeProvider() : super(0);
}
```


<br/>

**使用：** 一旦你创建了HomeProvider的实例，你就可以在你的Flutter小部件树中使用它来保存和恢复整数状态。例如：

```
class MyApp extends StatelessWidget {
  final HomeProvider homeProvider = HomeProvider();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: RestorationScope(
        restorationId: 'app',
        child: Scaffold(
          appBar: AppBar(
            title: Text('RestorableInt Example'),
          ),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('Counter Value: ${homeProvider.value}'),
                ElevatedButton(
                  onPressed: () {
                    // Increment the counter
                    homeProvider.value += 1;
                  },
                  child: Text('Increment'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

```


在这个例子中，homeProvider实例被用于保存和恢复计数器的值。每次点击“Increment”按钮时，计数器的值会增加，并且这个值会在应用程序重启或其他状态变化时得以保留



<br/>

在RestorationMixin中的使用:

```
class MyRestorableWidget extends StatefulWidget with RestorationMixin {
  @override
  String get restorationId => 'myRestorableWidget';

  final RestorableInt counter = RestorableInt(0);

  @override
  void restoreState(RestorationBucket oldBucket, bool initialRestore) {
    // 恢复状态
    registerForRestoration(counter, 'counter');
  }

  @override
  void saveState(RestorationBucket bucket, bool saveImmediately) {
    // 保存状态
    counter.saveState();
  }
}
```


&emsp; 在上面的示例中，MyRestorableWidget 混入了 RestorationMixin，并通过重写 restorationId 属性来为该小部件指定一个唯一的标识符。在 restoreState 方法中，通过调用 registerForRestoration 方法来注册需要保存和还原的状态对象。


&emsp; 总体来说，RestorableInt 用于具体的状态值的保存和还原，而 RestorationMixin 则用于在小部件中启用整个状态保存和还原机制。在实际的应用程序中，它们通常一起使用，以便管理和维护整个应用程序的状态。









<br/>

***
<br/><br/>

> <h1 id='PageController()'>PageController()</h1>


&emsp; PageController 是 Flutter 中用于控制 PageView 的控制器类。PageView 是一个可以滑动切换页面的组件，PageController 用于管理和控制页面切换的行为。

以下是一个简单的示例，演示如何使用 PageController：

```
PageController _pageController = PageController();

Widget build(BuildContext context) {
  return PageView(
    controller: _pageController,
    children: [
      // 页面1
      Container(color: Colors.blue),
      
      // 页面2
      Container(color: Colors.green),
      
      // 页面3
      Container(color: Colors.red),
    ],
  );
}


@override
  void dispose() {
    _pageController.dispose();
    super.dispose();
  }
```



在这个例子中：

- 创建了一个 PageController 实例 _pageController。

- 在 PageView 中使用了这个控制器。

- children 列表中包含了三个页面，每个页面都是一个简单的彩色容器。

- 通过 PageController，你可以实现以下一些功能：

- 手动控制页面切换： 使用 jumpTo 或 animateTo 方法可以手动切换到指定的页面。

- 监听页面变化： 通过 addListener 可以监听页面的变化，例如滑动到了哪一页。

- 获取当前页面： 使用 page 属性可以获取当前页面的索引。



<br/>

**释放资源:**

在Flutter中，如果你使用了PageController，特别是在StatefulWidget的dispose方法中手动调用了dispose方法来释放 _pageController，这是一个良好的实践。

在dispose方法中调用_pageController.dispose()是为了确保在小部件销毁时，PageController所使用的资源（例如，动画控制器、监听器等）被适当地释放，从而防止内存泄漏。

当你使用PageController时，如果不手动调用dispose，则可能会导致资源泄漏，尤其是在小部件被销毁或不再需要的情况下。在dispose方法中调用dispose是一种良好的做法，以确保释放相关资源。

<br/>

&emsp; 这使得 PageController 成为管理 PageView 中页面切换的重要工具。


<br/>

***
<br/><br/>

> <h1 id='打印'>打印</h1>


<br/><br/>

> <h2 id='debugPrint'>debugPrint</h2>


&emsp; debugPrint 是 Flutter 提供的用于打印调试信息的函数。它是 print 函数的一种替代，但在开发环境下提供更多的调试信息


**debugPrint 的说明：**

- 调试信息： debugPrint 用于打印调试信息，这对于识别代码中的问题非常有用。

- 多平台支持： debugPrint 在 Flutter 中是一个跨平台的函数，可以在 iOS、Android 等平台上使用。它会根据平台的不同选择合适的输出方式。

- 可配置性： debugPrint 允许你配置一些参数，例如是否在 release 模式下禁用打印、是否输出时间戳等。这样可以更灵活地适应不同的开发场景。

- 在实际使用中，你可以使用 debugPrint 来输出变量的值、对象的内容、错误信息等，以便更好地了解应用程序的运行状态。例如：

```
try {
  // 一些可能抛出异常的代码
} catch (error) {
  debugPrint("An error occurred: ${error.toString()}");
}

```

虽然 debugPrint 在开发环境下提供更多的信息，但在发布版本中，它会被自动替换为 print 函数，以减小应用程序的体积。因此，你可以放心在开发中使用 debugPrint 进行调试，而不必担心在发布版本中产生额外的负担。

<br/>





<br/>

***
<br/><br/>

> <h1 id='工具类'>工具类</h1>


<br/><br/>

> <h2 id='图片工具'>图片工具</h2>

[**AssetImage:**](https://doc.flutterchina.club/assets-and-images/) 加载图片;

获取图片:

```
new AssetImage('graphics/background.png'),
```


<br/>

网络图片加载:

[CachedNetworkImageProvider](#cached_network_image-图片加载)加载网络图片

```
static ImageProvider getImageProvider(String? imageUrl, {String holderImg = 'none'}) {
    if (TextUtil.isEmpty(imageUrl)) {//判断是否为空
      return AssetImage(getImgPath(holderImg));
    }
    //网络图片加载:https://blog.csdn.net/qq_33224517/article/details/118079879
    return CachedNetworkImageProvider(imageUrl!);
  }
```


<br/>

***
<br/>


> <h1 id='组件库'>组件库</h1>


<br/><br/>

># <h2 id='Dio网络库'>[Dio网络库](https://github.com/cfug/dio/blob/main/README-ZH.md)</h2>




<br/><br/>

> <h2 id='fluro路由组件'>fluro路由组件</h2>


使用:

```
FluroRouter().navigateTo(context, path,
  replace: replace,
  clearStack: clearStack,
  transition: TransitionType.native,//原生形式
  routeSettings: RouteSettings(
    arguments: arguments,
  ),
);
```


<br/>

<br/><br/>

> <h2 id='FluroRouter'>FluroRouter</h2>



- **FluroRouter 的详细说明：**

	- 路由定义： FluroRouter 允许你定义应用程序中的不同路由，每个路由可以与一个特定的页面或者视图关联。
	
	- 路由参数传递： 你可以使用 FluroRouter 将参数传递给不同的路由。这些参数可以包含在路由中，以便在目标页面中使用。
	
	- 路由跳转： FluroRouter 提供了方法用于执行路由跳转。这使得在应用程序中进行页面切换变得更加简单。
	
	- 路由拦截： 你可以使用 FluroRouter 实现路由拦截，例如在路由跳转之前执行一些逻辑，或者在路由跳转完成后执行一些操作。
	
	- RESTful 风格的路由： fluro 支持 RESTful 风格的路由，使得定义和使用路由更加灵活和符合约定。



以下是一个简单的示例，演示了如何在 Flutter 中使用 FluroRouter：


```
import 'package:fluro/fluro.dart';
import 'package:flutter/material.dart';

void main() {
  // 创建 FluroRouter 实例
  final FluroRouter router = FluroRouter();

  // 定义路由
  router.define(
    '/home',
    handler: Handler(
      handlerFunc: (BuildContext context, Map<String, dynamic> params) {
        return HomeScreen();
      },
    ),
  );

  // 定义带参数的路由
  router.define(
    '/profile/:id',
    handler: Handler(
      handlerFunc: (BuildContext context, Map<String, dynamic> params) {
        String userId = params['id'][0];
        return ProfileScreen(userId: userId);
      },
    ),
  );

  // 在 MaterialApp 中使用 FluroRouter
  runApp(
    MaterialApp(
      onGenerateRoute: router.generator,
      // other MaterialApp configuration...
    ),
  );
}
```

&emsp; 在上述例子中，我们首先创建了一个 FluroRouter 的实例，并使用 define 方法定义了两个路由，分别是 /home 和 /profile/:id。然后，在 MaterialApp 中使用了这个 FluroRouter 的实例。

&emsp; 在这个简单的示例中，FluroRouter 负责管理应用程序的路由，以及根据路由信息生成相应的页面。在实际应用中，你可能会使用 FluroRouter 来组织和管理整个应用的路由体系。

<br/><br/>

> <h3 id='扩展拦截器类'>扩展拦截器类</h3>

&emsp; 在 Dart 和 Flutter 中，拦截器（Interceptor）通常与网络请求库一起使用，用于在请求发出和响应返回之间插入一些逻辑。这些逻辑可能包括添加请求头、记录日志、处理错误等。一个常见的 Dart 网络请求库是 Dio，下面我会通过 Dio 中的拦截器来说明。

&emsp;Dio 中的拦截器是通过实现 Interceptor 接口来创建的。这个接口包含了两个方法：onRequest 和 onResponse，分别用于在发出请求和收到响应时执行相关逻辑。


<br/>

我们可以创建一个拦截器类来覆盖onRequest、onError、onResponse方法。

```
class AppInterceptors extends Interceptor {
  @override
  FutureOr<dynamic> onRequest(RequestOptions options) async {
    if (options.headers.containsKey("requiresToken")) {
      //remove the auxiliary header
      options.headers.remove("requiresToken");

      SharedPreferences prefs = await SharedPreferences.getInstance();
      var header = prefs.get("Header");

      options.headers.addAll({"Header": "$header${DateTime.now()}"});

      return options;
    }
  }

  @override
  FutureOr<dynamic> onError(DioError dioError) {
    if (dioError.message.contains("ERROR_001")) {
      // this will push a new route and remove all the routes that were present
      navigatorKey.currentState.pushNamedAndRemoveUntil(
          "/login", (Route<dynamic> route) => false);
    }

    return dioError;
  }

  @override
  FutureOr<dynamic> onResponse(Response options) async {
    if (options.headers.value("verifyToken") != null) {
      //if the header is present, then compare it with the Shared Prefs key
      SharedPreferences prefs = await SharedPreferences.getInstance();
      var verifyToken = prefs.get("VerifyToken");

      // if the value is the same as the header, continue with the request
      if (options.headers.value("verifyToken") == verifyToken) {
        return options;
      }
    }

    return DioError(request: options.request, message: "User is no longer active");
  }
}
```


该类可以被轻松的添加到dio对象的拦截器集合中：

```
Dio addInterceptors(Dio dio) {
  dio.interceptors.add(AppInterceptors());
}
```

<br/>
<br/>

**案例Demo**


```
import 'package:dio/dio.dart';

void main() async {
  // 创建 Dio 实例
  Dio dio = Dio();

  // 添加一个请求拦截器
  dio.interceptors.add(MyRequestInterceptor());

  // 添加一个响应拦截器
  dio.interceptors.add(MyResponseInterceptor());

  // 发起网络请求
  try {
    Response response = await dio.get('https://api.example.com/data');
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}

// 自定义请求拦截器
class MyRequestInterceptor extends Interceptor {
  @override
  onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    // 在请求发出之前执行的逻辑
    print('Request Interceptor: ${options.uri}');
    handler.next(options); // 继续请求
  }
}

// 自定义响应拦截器
class MyResponseInterceptor extends Interceptor {
  @override
  onResponse(Response response, ResponseInterceptorHandler handler) async {
    // 在收到响应之后执行的逻辑
    print('Response Interceptor: ${response.requestOptions.uri}');
    handler.next(response); // 继续处理响应
  }
}
```


&emsp; 在上述示例中，我们创建了一个 Dio 实例，并添加了两个拦截器：MyRequestInterceptor 和 MyResponseInterceptor。这两个拦截器分别在请求发起前和收到响应后执行一些逻辑。

&emsp; 拦截器提供了一种有效的方式，允许我们在网络请求的不同阶段插入自定义的逻辑，以便更好地控制和管理网络请求过程中的行为。




<br/><br/>

> <h2 id='CancelToken'>CancelToken</h2>

&emsp; CancelToken 通常与异步任务和取消相关的库（例如 dio 等）一起使用，用于在需要时取消异步操作。CancelToken 包含一个用于取消异步任务的令牌，以及一些与取消相关的方法，如 isCancel。

- CancelToken 中的 isCancel 方法用于检查是否已取消相应的异步任务。让我们来详细解释一下：

	- 	1.CancelToken 介绍： CancelToken 是由 Dart 异步任务取消库提供的一部分。它允许你在进行异步任务时注册取消回调，并在需要时取消任务。这对于处理网络请求、定时器或其他可能需要被中断的异步任务非常有用。
	
	- 	2.isCancel 方法： isCancel 是 CancelToken 类中的一个方法。它用于检查当前的令牌是否已经被取消。方法的定义如下：

```
bool isCancel(dynamic e) {
  // Implementation details
}
```

isCancel 方法接受一个参数 e，通常是捕获到的异常对象。它会检查该异常对象是否与取消相关，如果是取消异常，则返回 true；否则返回 false。这是用于在处理异步任务时检查是否应该取消任务的便捷方法。


- 3.使用 CancelToken： 通常，在发起异步任务之前，你会创建一个 CancelToken 对象，并将其传递给异步任务。当需要取消任务时，你可以调用 cancel 方法，这将触发与该令牌相关的所有回调。在异步任务中，你可以使用 isCancel 方法检查是否应该取消任务。

```
import 'dart:async';

void main() {
  // 创建 CancelToken 对象
  CancelToken cancelToken = CancelToken();

  // 模拟异步任务
  Future<void> asyncTask(CancelToken token) async {
    try {
      // 在异步任务中使用 isCancel 方法检查是否已取消
      if (token.isCancel()) {
        print('Task is cancelled');
        return;
      }

      // 模拟异步操作
      await Future.delayed(Duration(seconds: 2));

      // 在异步任务中再次检查是否已取消
      if (token.isCancel()) {
        print('Task is cancelled');
        return;
      }

      // 完成异步任务
      print('Task completed');
    } catch (e) {
      // 处理异常
      if (token.isCancel(e)) {
        print('Task is cancelled');
      } else {
        print('Error: $e');
      }
    }
  }

  // 启动异步任务
  asyncTask(cancelToken);

  // 在需要的时候取消任务
  cancelToken.cancel();
}
```

&emsp; isCancel 方法用于在异步任务中检查是否已取消任务。请注意，在取消时，可能会抛出 CancelException 或其他与取消相关的异常，isCancel 方法用于检查异常是否与取消有关。



<br/><br/>

> <h3 id='cached_network_image-图片加载'>cached_network_image-图片加载</h3>

```
//添加占位图
CachedNetworkImage(
	imageUrl: "http://via.placeholder.com/350x150",
	placeholder: (context, url) => CircularProgressIndicator(),
	errorWidget: (context, url, error) => Icon(Icons.error),
),
   
   
     
//进度条展示
CachedNetworkImage(
    imageUrl: "http://via.placeholder.com/350x150",
    progressIndicatorBuilder: (context, url, downloadProgress) => 
            CircularProgressIndicator(value: downloadProgress.progress),
    errorWidget: (context, url, error) => Icon(Icons.error),
),
```






<br/><br/>

> <h3 id=''></h3>









<br/><br/>

> <h2 id='flutter_ume调试工具'>[flutter_ume调试工具](https://github.com/bytedance/flutter_ume/blob/master/README_cn.md)</h2>










<br/><br/>

># <h2 id='common_utils-工具库'>[common_utils-工具库](https://github.com/Sky24n/common_utils)</h2>


Dart常用工具类库。包含日期，正则，倒计时，时间轴等工具类。如果你有好的工具类欢迎PR.


Dart常用工具类库 **common_utils**

- TimelineUtil : 时间轴.
- TimerUtil : 倒计时，定时任务.
- MoneyUtil : 精确转换，元转分，分转元，支持格式输出.
- LogUtil : 简单封装打印日志.
- DateUtil : 日期转换格式化输出.
- RegexUtil : 正则验证手机号，身份证，邮箱等等.
- NumUtil : 保留x位小数, 精确加、减、乘、除, 防止精度丢失.
- ObjectUtil : 判断对象是否为空(String List Map),判断两个List是否相等.
- EncryptUtil : 异或对称加/解密，md5加密，Base64加/解密.
- TextUtil : 银行卡号每隔4位加空格，每隔3三位加逗号，隐藏手机号等等.
- JsonUtil : 简单封装json字符串转对象.


<br/>
<br/>


Flutter常用工具类库 [flustars](https://github.com/Sky24n/flustars)

- SpUtil : 单例"同步"SharedPreferences工具类。支持get传入默认值，支持存储对象，支持存储对象数组。
- ScreenUtil : 屏幕适配，获取屏幕宽、高、密度，AppBar高，状态栏高度，屏幕方向.
- WidgetUtil : 监听Widget渲染状态，获取Widget宽高，在屏幕上的坐标，获取网络/本地图片尺寸.
- ImageUtil : 获取网络/本地图片尺寸.
- DirectoryUtil : 文件目录工具类.
- DioUtil : 单例Dio网络工具类(已迁移至此处DioUtil)。







<br/>

># <h2 id='keyboard_actions处理键盘事件'>[keyboard_actions处理键盘事件](https://github.com/diegoveloper/flutter_keyboard_actions)</h2>





<br/>
<br/>


> <h2 id='sp_util-保存对象'>sp_util-保存对象</h2>



<br/>
<br/>


> <h2 id='sprintf格式化字符串'>sprintf格式化字符串</h2>


[格式化字符串](https://www.jianshu.com/p/78ca5e599361)




<br/>
<br/>


> <h2 id=''></h2>





<br/>
<br/>


> <h2 id=''></h2>




<br/>
<br/>


> <h2 id=''></h2>





<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>




<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>






<br/>
<br/>


> <h2 id=''></h2>




