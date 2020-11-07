[Flutter生命周期（组件渲染）](https://juejin.im/post/6844903794879234061)
<br/>
[Flutter的生命周期(交互)](https://juejin.im/post/6844903794883444743)
<br/>
**[Flutter开发实战详解（App的优化都有）](https://wizardforcel.gitbooks.io/gsyflutterbook/content/Flutter-1.html)**


<br/>

- **`WidgetsBinding.instance`**
	- 渲染后的回调
- **`AnnotatedRegion(不使用AppBar)`**
	- AppBar的渐变色
- **`系统字体缩放`**
- **`导航返回拦截（WillPopScope）`**
- **`Scaffold、TabBar、底部导航`**
- **``**
- **``**
- **``**


<br/>

***
<br/>

># `WidgetsBinding.instance` 
&emsp; 用于在App的[生命周期](https://juejin.im/post/6844903794883444743)做一些事件，例如App从后台进入前台，从前台退入后台（或被遮盖），以及需要在确保UI绘制后做一些处理。
&emsp; `WidgetsBinding`控件层和`Flutter`引擎之间的粘合剂。就是这个类 它能监听到第一帧绘制完成，第一帧绘制完成标志着已经Build完成，并交由引擎绘制结束。

&emsp; 只要给WidgetsBinding的单例对象添加WidgetsBindingObserver，然后此类粘合(with)WidgetsBindingObserver抽象类，在初始化State的时候给WidgetsBinding设置Observer为当前类，并在销毁的时候移除Observer，此时我们的State就具备WidgetsBindingObserver的回调了。

```
abstract class WidgetsBindingObserver {

   //路由弹出Future
  Future<bool> didPopRoute() => Future<bool>.value(false);

    //新的路由Future
  Future<bool> didPushRoute(String route) => Future<bool>.value(false);

    //系统窗口相关改变回调，例如旋转
  void didChangeMetrics() { }

    //文字系数变化
  void didChangeTextScaleFactor() { }

    //本地化语言变化
  void didChangeLocales(List<Locale> locale) { }

    //生命周期变化(生命周期回调)
  void didChangeAppLifecycleState(AppLifecycleState state) { }

    //低内存回调
  void didHaveMemoryPressure() { }

    //当前系统改变了一些访问性活动的回调
  void didChangeAccessibilityFeatures() {}
}




//AppLifecycleState的枚举类，我们可以根据它的状态来处理我们的一些任务
enum AppLifecycleState {
  resumed, //应用可见并可响应用户操作
	inactive, //用户可见，但不可响应用户操作
	paused, //已经暂停了，用户不可见、不可操作
	suspending, //应用被挂起，此状态IOS永远不会回调
}


```

<br/>

- **`渲染后的回调`**
	- 单次Frame绘制回调`(addPostFrameCallback)`

```
void initState() {
  super.initState();
  //
  WidgetsBinding.instance.addPostFrameCallback((_){
	  // 在这里可以进行网络请求
    print("Frame has been rendered");
  });
}
```

&emsp; 通过addPostFrameCallback可以做一些安全的操作，在有些时候是很有用的，它会在当前Frame绘制完后进行回调，并只会回调一次，如果要再次监听需要再设置。

- 实时Frame绘制回调(addPersistentFrameCallback)

```
//持久帧的回调
WidgetsBinding.instance.addPersistentFrameCallback((_){
  print("Frame has been rendered");
});
```
这个api在每次绘制Frame结束后都会回调，我们可以利用它做一些帧率检测。
[帧绘制](https://blog.csdn.net/baoolong/article/details/85097318)


<br/>

***
<br/>

># AnnotatedRegion(不使用AppBar)


&emsp; `MediaQuery` 是一个 `InheritedWidget` ，它有一个叫 `MediaQueryData` 的参数，通过源码我们知道，一般情况下 `MediaQueryData` 的 `padding 的 top` 就是状态栏的高度。

获取状态栏高度:
<br/>
`MediaQueryData.fromWindow(WidgetsBinding.instance.window).padding.top`

`AppBar的brightness`属性：`Brightness.light`为黑色，`Brightness.dark`为白色;

&emsp; 不使用AppBar时，可以通过嵌套`
AnnotatedRegion<SystemUiOverlayStyle>`来实现。

```
class StatusDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.white,
      child: AnnotatedRegion<SystemUiOverlayStyle>(
        child: Column(
          children: <Widget>[
            Container(
              height: 200,
              color: Colors.brown,
            ),
          ],
        ),
        value: SystemUiOverlayStyle(
            statusBarColor: Colors.cyan,
            //有Appbar时，会被覆盖
            statusBarIconBrightness: Brightness.light,
            //底部navigationBar背景颜色
            systemNavigationBarColor: Colors.white),
      ),
    );
  }
}
```


<br/>

- AppBar的渐变色


```
appBar: AppBar(
        title: Text('DemoPage'),
        //使用AppBar的flexibleSpace，其中渐变方式还可选择RadialGradient和SweepGradient
        flexibleSpace: Container(
          decoration: BoxDecoration(
              gradient: LinearGradient(
            colors: [Colors.cyan, Colors.brown, Colors.deepOrange],
          )),
        ),
      )
```



![z7](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z7.png)


<br/>

- 返回键监听

&emsp; 使用 `WillPopScope` 嵌套，可以用于监听处理返回键的逻辑。其实`WillPopScope` 并不是监听返回按键，是当前页面将要被pop时触发的回调。

```
class _StatusDemoState extends State<StatusDemo> {
  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      child: StatusWidget(),
      onWillPop: () {
        ///如果返回 return new Future.value(false); popped 就不会被处理
        ///如果返回 return new Future.value(true); popped 就会触发
        ///这里可以通过 showDialog 弹出确定框，在返回时通过 Navigator.of(context).pop(true);决定是否退出
        return showExitDialog(context);
      },
    );
  }

  Future<bool> showExitDialog(BuildContext context) {
    return showDialog(
        context: context,
        builder: (BuildContext content) {
          return AlertDialog(
            title: Text("提示"),
            content: Text("确认退出吗？"),
            actions: <Widget>[
              GestureDetector(
                child: Container(
                  child: Text('退出'),
                ),
                onTap: () {
                  Navigator.of(context).pop(true);
                },
              ),
              GestureDetector(
                child: Container(
                  child: Text('取消'),
                ),
                onTap: () {
                  Navigator.of(context).pop();
                },
              ),
            ],
          );
        });
  }
}

```




<br/>

***
<br/>

># 系统字体缩放
&emsp; Flutter 中字体缩放也是和 MediaQueryData 的 textScaleFactor 有关。所以我们可以在需要的页面，通过最外层嵌套如下代码设置，将字体设置为默认不允许缩放。

```
MediaQuery(
      data: MediaQueryData.fromWindow(WidgetsBinding.instance.window).copyWith(textScaleFactor: 1),
      child: new Container(),
    );
```


<br/>

***
<br/>

># [导航返回拦截（WillPopScope）](https://book.flutterchina.club/chapter7/willpopscope.html)
&emsp; 为了避免用户误触返回按钮而导致APP退出，在很多APP中都拦截了用户点击返回键的按钮，然后进行一些防误触判断，比如当用户在某一个时间段内点击两次时，才会认为用户是要退出（而非误触）。Flutter中可以通过`WillPopScope`来实现返回按钮拦截，我们看看`WillPopScope`的默认构造函数：

```
const WillPopScope({
  ...
  @required WillPopCallback onWillPop,
  @required Widget child
})
```


<br/>

***
<br/>

># [Scaffold、TabBar、底部导航](https://book.flutterchina.club/chapter5/material_scaffold.html)




<br/>

***
<br/>

>#


