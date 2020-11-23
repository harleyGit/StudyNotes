- [**组件可选参数Key的见解**](https://juejin.im/post/5ca2152f6fb9a05e1a7a9a26)
- **Provider**
- **Widget 管理自身状态**



<br/>

***
<br/>

># Provider

![z21](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z21.png)

&emsp; Flutter 实际上在一开始就为我们提供了一种状态管理方式，那就是 StatefulWidget。但是我们很快发现，它正是造成上述原因的罪魁祸首。

&emsp; 在 State 属于某一个特定的 Widget，在多个 Widget 之间进行交流的时候，虽然你可以使用 callback 解决，但是当嵌套足够深的话，我们增加非常多可怕的垃圾代码。

&emsp; 这时候，我们便迫切的需要一个架构来帮助我们理清这些关系，状态管理框架应运而生。


下载 [Provider](https://pub.flutter-io.cn/packages/provider/install)

```
dependencies:
  provider: ^4.3.2+2
```




<br/>

***
<br/>


># Widget 管理自身状态

<br/>

```
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

///TapboxA 类
class TapboxA extends StatefulWidget {
  TapboxA({Key key}) : super(key: key);

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

    return new GestureDetector(
      onTap: _handleTap,
      child: new Container(
        child: new Center(
          child: new Text(
            _active ? 'Active' : 'No Active',
            style: new TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: new BoxDecoration(
          color:_active ? Colors.lightGreen[700] : Colors.grey[600],
        ),
      ),
    );
  }
  
}
```

<br/>

**`调用`**

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
        body: TapboxA(),
      )
    );
  }
  
}
```

效果图：

![效果](https://upload-images.jianshu.io/upload_images/2959789-c6886fdd8e2fa5eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击灰色色块，进行变色。




<br/>

***
<br/>

># 父Widget管理子Widget的状态

```


///------------------------- ParentWidgetState 类 ----------------------------------
///父组件
class ParentWidgetState extends StatefulWidget {
  @override
  _ParentWidgetState createState()  => new _ParentWidgetState();
  
}

class _ParentWidgetState extends State<ParentWidgetState> {

  bool _active = false;

  ///回掉函数
  void _handleTapboxChaned(bool newValue){
    setState(() {
      //修改状态
      _active = newValue;
    });
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new Container(
      child: new TapboxB (
        active: _active,
        //将_handleTapboxChaned函数赋值给TapboxB的onChanged
        onChanged: _handleTapboxChaned,
      ),
    );
  }
  
}

///------------------------- TapboxB ----------------------------------
///子组件
class TapboxB extends StatelessWidget {

  final bool active;
  ///定义接收父类回调函数的指针
  final ValueChanged<bool> onChanged;

  TapboxB({Key key, this.active: false, @required this.onChanged}) : super(key: key);

  void _handleTap() {
    onChanged(!active);
  }



  @override
  Widget build(BuildContext context) {

    return new GestureDetector(
      //调用回调函数传值
      onTap: _handleTap,
      child: new Container(
        child: new Center(
          child: new Text(
            active ? "Active【激活】" : "No Active【没激活】",
            style: new TextStyle(
              fontSize: 18.0,
              color: Colors.white,
            ),
          ),
        ),
        width: 200,
        height: 200,
        decoration: new BoxDecoration(
          color: active ? Colors.lightGreen[700] : Colors.grey[600],
        ),
      ),
    );
  }
  
}

```

<br/>

**`调用`**

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
        body: ParentWidgetState(),
      )
    );
  }
  
}
```

效果图：

![父类管理](https://upload-images.jianshu.io/upload_images/2959789-a2ad04a64d97d054.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

