># Windget 简介
**`StatelessWidget`**
&emsp;  `StatelessWidget`用于不需要维护状态的场景，它通常在build方法中通过嵌套其它Widget来构建UI，在构建过程中会递归的构建其嵌套的Widget。
```
class Echo extends StatelessWidget {

  final String text;
  final Color backgroundColor;

  const Echo({
    Key key,
    @required this.text,
    this.backgroundColor: Colors.grey,
  }): super(key: key);  
//Key: 这个key属性类似于React/Vue中的key;
//主要的作用是决定是否在下一次build时复用旧的widget，决定的条件在canUpdate()方法中

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: backgroundColor,
        child: Text(text),
      ),
    );
  }
}


//调用
Widget build(BuildContext context) {
  return MaterialApp(
    color: Colors.white,
    home: Echo(text: "welcome to 我的家", backgroundColor: Colors.white),
  );
}
```
&emsp;  widget的构造函数参数应使用命名参数，`命名参数中的必要参数要添加@required标注`，这样有利于静态代码分析器进行检查。另外，`在继承widget时，第一个参数通常应该是Key`，另外，如果Widget需要接收子Widget，那么child或children参数通常应被放在参数列表的最后。同样是按照惯例，`Widget的属性应尽可能的被声明为final，防止被意外改变`。

![效果图](https://upload-images.jianshu.io/upload_images/2959789-1365e10ebed05a27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
**属性： context**
&emsp;  `build`方法有一个`context`参数，它是`BuildContext`类的一个实例，表示当前widget在widget树中的上下文，每一个widget都会对应一个context对象（因为每一个widget都是widget树上的一个节点）。实际上，context是当前widget在widget树中位置中执行”相关操作“的一个句柄，比如它提供了从当前widget开始向上遍历widget树以及按照widget类型查找父级widget的方法。

```

@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text("Context 测0000000试"),
    ),
    body: Container(
      child: Builder(builder: (context) {
        Scaffold scaffold = context.findAncestorWidgetOfExactType();
        return (scaffold.appBar as AppBar).title;
      }),
    ),
  );
}
```
![context 测试](https://upload-images.jianshu.io/upload_images/2959789-195d30cddc2dd37c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

**`StatefulWidget `**
**方法:**
- `createState()` 用于创建和Stateful widget相关的状态，它在Stateful widget的生命周期中可能会被多次调用。例如，当一个Stateful widget同时插入到widget树的多个位置时，Flutter framework就会调用该方法为每一个位置生成一个独立的State实例，其实，本质上就是一个StatefulElement对应一个State实例。
> &emsp;  在说`“widget树”`时它可以指`widget结构树`，但由于`widget与Element有对应关系（一可能对多）`，在有些场景（Flutter的SDK文档中）也代指“UI树”的意思。而在`stateful widget`中，`State对象也和StatefulElement具有对应关系（一对一）`，所以在Flutter的SDK文档中，可以经常看到“从树中移除State对象”或“插入State对象到树中”这样的描述。其实，无论哪种描述，其意思都是在描述“一棵构成用户界面的节点元素的树”。



<br/>

**`State `**
&emsp;  一个`StatefulWidget`类会对应一个`State类`，State表示与其对应的`StatefulWidget`要维护的状态，State中的保存的状态信息有：
- 在widget 构建时可以被同步读取。
- 在widget生命周期中可以被改变，当State被改变时，可以手动调用其 `setState()`方法通知Flutter framework状态发生改变，Flutter framework在收到消息后，会重新调用其build方法`重新构建widget树`，从而达到更新UI的目的。

State中有两个常用属性：
- `widget`，它表示与该`State实例关联的widget实例`，由Flutter framework动态设置。`注意`，这种关联并非永久的，因为在应用生命周期中，UI树上的某一个节点的widget实例在重新构建时可能会变化，但State实例只会在第一次插入到树中时被创建，当在重新构建时，`如果widget被修改了，Flutter framework会动态设置State.widget为新的widget实例`。
- `context`, `StatefulWidget`对应的`BuildContext`，作用同`StatelessWidget的BuildContext`。



<br/>
**`Flutter SDK自带组件库`**
**基础组件**
`import 'package:flutter/widgets.dart';`
&emsp;  `Flutter` 提供了一套丰富、强大的基础组件，在基础组件库之上`Flutter`又提供了一套`Material风格（Android默认的视觉风格）`和一套`Cupertino风格（iOS视觉风格）的组件库`。

**Material组件**
`import 'package:flutter/material.dart';`
&emsp;  `Material`组件，它可以帮助我们构建遵循Material Design设计规范的应用程序。Material应用程序以[`MaterialApp`](https://docs.flutter.io/flutter/material/MaterialApp-class.html) 组件开始， 该组件在应用程序的根部创建了一些必要的组件，比如`Theme`组件，它用于配置应用的主题。


**Cupertino组件**
`import 'package:flutter/cupertino.dart';`
&emsp;  `Cupertino风格的组件`，尽管目前还没有Material 组件那么丰富，但是它仍在不断的完善中。值得一提的是在Material 组件库中有一些组件可以根据实际运行平台来切换表现风格，比如MaterialPageRoute，在路由切换时，如果是Android系统，它将会使用Android系统默认的页面切换动画(从底向上)；如果是iOS系统，它会使用iOS系统默认的页面切换动画（从右向左）。

`cupertinoTestRoute.dart 文件`
```
import 'package:flutter/cupertino.dart';

class cupertinoTestRoute extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return CupertinoPageScaffold(
      navigationBar: CupertinoNavigationBar(
        middle: Text("Cupertino 组件库测试"),
      ),
      child: Center(
        child: CupertinoButton(
            color: CupertinoColors.activeOrange,
            child: Text("点击"),
            onPressed: () {}),
      ),
    );
  }
}

```

`main.dart 文件`
```
//自定义类文件
import 'package:flutter_demo/customWidget/cupertinoWidget.dart';

//调用
Widget build(BuildContext context) {

  return MaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(
      primarySwatch: Colors.cyan,
    ),
    home: cupertinoTestRoute(),//初始化
  );
}

```

效果：
![组件库效果图](https://upload-images.jianshu.io/upload_images/2959789-84c6ca8d05ccdc65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)


<br/>
***
<br/>
># 状态管理
`stateWidgets.dart 文件`
```
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

class TapboxA extends StatefulWidget {
  TapboxA({Key key}): super(key: key);

  @override
  _TapboxAState createState() => new _TapboxAState();
}

class _TapboxAState extends State<TapboxA> {
  bool _active = false;

  void _handleTap() {
    setState(() {
      _active = !_active;
    });
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new GestureDetector(
      onTap: _handleTap,
      child: new Container(
        child: new Center(
          child: new Text(
            _active ? "激活了" : "未激活",
            style: new TextStyle(
              fontSize: 32.0,
              color: Colors.white
            ),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: new BoxDecoration(
          color: _active ? Colors.lightGreen[700] : Colors.grey[600],
        ),
      ),
    );
  }

}
```

`main.dart 文件`
```
Widget build(BuildContext context) {

  return MaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(
      primarySwatch: Colors.cyan,
    ),
    home: TapboxA(),
  );
}
}
```
效果：
![状态管理](https://upload-images.jianshu.io/upload_images/2959789-0b87349542f9aa86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)






<br/>
***
<br/>


># 路由管理
打开一个简单的路由Controller
```
//创建一个点击按钮
FlatButton(
  child: Text("打开新路由"),
  textColor: Colors.blue,
  onPressed: (){
    //导航到新路由
    Navigator.push(context, MaterialPageRoute(builder: (context) {
      return NewRoute();
    }));
  },
)



//路由controller
class NewRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //Scaffold 是 Material 库中提供的页面脚手架，
    //它提供了默认的导航栏、标题和包含主屏幕widget树（后同“组件树”或“部件树”）的body属性，组件树可以很复杂。
    return Scaffold(
      appBar: AppBar(
        title: Text("👋首页"),
      ),
      body: Center(
        child: Text("这是新的路由"),
      ),
    );
    
  }
}
```
效果图：
![Simulator Screen Shot - iPhone 11 - 2020-01-14 at 22.18.38.png](https://upload-images.jianshu.io/upload_images/2959789-ea3c3924bc502b6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/220)

&emsp; `MaterialPageRoute`继承自`PageRoute`类，`PageRoute`类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。
&emsp; `MaterialPageRoute` 是`Material组件库提供的组件`，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画。
&emsp; `MaterialPageRoute` 构造函数的各个参数的意义：
```
  MaterialPageRoute({
    WidgetBuilder builder,
    RouteSettings settings,
    bool maintainState = true,
    bool fullscreenDialog = false,
  })
```
- `builder` 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。我们通常要实现此回调，返回新路由的实例。
- `settings` 包含路由的配置信息，如路由名称、是否初始路由（首页）。
- `maintainState`：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置maintainState为false。
- `fullscreenDialog` 表示新的路由页面是否是一个全屏的模态对话框，在iOS中，如果fullscreenDialog为true，新页面将会从屏幕底部滑入（而不是水平方向）。
