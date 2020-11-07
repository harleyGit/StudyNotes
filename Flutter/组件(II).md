&emsp; 官方说的一通看了以后有点不太了解，通过把代码敲了一通，然后打断点跑了一遍终于知道是咋回事了。
&emsp; 开发做多了，完全跟`iOS的设计模式单例`差不多在存储数据方面。但是在`单例上`又加了`通知`和`全局刷新整个app组件`，相当简单暴力。
好了，先看第一个代码Demo：

```
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

class ShareDataWidget extends InheritedWidget {

  ///需要在子树中共享的数据，保存点击次数
  final int data;

  ShareDataWidget({
    @required this.data,
    Widget child
  }) : super(child: child);

  ///定义一个便捷方法，方便子树中的widget获取共享数据 
  static ShareDataWidget of(BuildContext context) {
    //return context.inheritFromWidgetOfExactType(ShareDataWidget);// 废弃了
    return context.dependOnInheritedWidgetOfExactType(aspect: ShareDataWidget);
  }

  ///该回调决定当data发生变化时，是否通知子树中依赖data的Widget
  @override
  bool updateShouldNotify(ShareDataWidget oldWidget) {
    // TODO: implement updateShouldNotify
    // 如果返回true，则子树中依赖(build函数中有调用)本widget
    // 的子widget的`state.didChangeDependencies`会被调用
    return oldWidget.data != data;
  }
  
}


/*
  * 实现一个子组件_TestWidget，在其build方法中引用ShareDataWidget中的数据。
  * 同时，在其didChangeDependencies() 回调中打印日志
*/
class  _TestWidget extends StatefulWidget {
  @override
  _TestWidgetState createState() => new _TestWidgetState();
  
}

class  _TestWidgetState extends State<_TestWidget> {
  
  @override
  Widget build(BuildContext context) {
    print("共享数据： ${ShareDataWidget.of(context).data.toString()}");
    //使用InheritedWidget中的共享数据，也就产生了依赖
    return Text(
      "依赖产生：" + ShareDataWidget.of(context)
                      .data
                      .toString()
                      
    );
  }


@override
  void didChangeDependencies() {
    // TODO: implement didChangeDependencies
    super.didChangeDependencies();
    //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("Dependencies change【依赖发生了改变】");
  }
  
}



class  InheritedWidgetTestRoute extends StatefulWidget {

  @override
  _InheritedWidgetTestRouteState createState() => new _InheritedWidgetTestRouteState();
  
}

class  _InheritedWidgetTestRouteState extends State<InheritedWidgetTestRoute> {

  int count = 0;

  @override
  Widget build(BuildContext context) {

    return Center(
      child: ShareDataWidget(
        data: count,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children:<Widget>[
            Padding(padding: const EdgeInsets.only(bottom: 20.0),
            child: _TestWidget(),// 子widget中依赖ShareDataWidget
            ),
            RaisedButton(
              ////每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新
              onPressed: () => setState(() => ++count),
              child: Text("Increment"),
              )
          ]
        ),
        ),
    );
  }
}

```

<br/>
# **`调用`**

```
void main() {
  runApp(MyApp());
  // runApp(Text("hellosjglajgl"));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home:Scaffold(
        appBar: AppBar(
          title: Text('Hello Flutter'),
        ),
        body: InheritedWidgetTestRoute(),
      )
    );
  }
  
}
```

运行效果：
![依赖刷新](https://upload-images.jianshu.io/upload_images/2959789-2350a473d6ad7095.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&emsp;  感觉这个可以做app的用户信息展示，感觉很棒。

&emsp;  一般来说，子widget很少会重写`didChangeDependencies()`方法，因为在依赖改变后framework也都会调用build()方法。但是，如果你需要在依赖改变后执行一些昂贵的操作，比如网络请求，这时最好的方式就是在`didChangeDependencies()`方法中执行，这样可以避免每次build()都执行这些昂贵操作。

