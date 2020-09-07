[MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) 是我们app开发中常用的符合MaterialApp Design设计理念的入口Widget，从源码可以看出该widget的构造方法中有多个参数，但是基本上大多数参数是可以省略的。


```
MaterialApp({
  Key key,
  this.title = '', // 设备用于为用户识别应用程序的单行描述
  this.home, // 应用程序默认路由的小部件,用来定义当前应用打开的时候，所显示的界面
  this.color, // 在操作系统界面中应用程序使用的主色。
  this.theme, // 应用程序小部件使用的颜色。
  this.routes = const <String, WidgetBuilder>{}, // 应用程序的顶级路由表
  this.navigatorKey, // 在构建导航器时使用的键。
  this.initialRoute, // 如果构建了导航器，则显示的第一个路由的名称
  this.onGenerateRoute, // 应用程序导航到指定路由时使用的路由生成器回调
  this.onUnknownRoute, // 当 onGenerateRoute 无法生成路由(initialRoute除外)时调用
  this.navigatorObservers = const <NavigatorObserver>[], // 为该应用程序创建的导航器的观察者列表
  this.builder, // 用于在导航器上面插入小部件，但在由WidgetsApp小部件创建的其他小部件下面插入小部件，或用于完全替换导航器
  this.onGenerateTitle, // 如果非空，则调用此回调函数来生成应用程序的标题字符串，否则使用标题。
  this.locale, // 此应用程序本地化小部件的初始区域设置基于此值。
  this.localizationsDelegates, // 这个应用程序本地化小部件的委托。
  this.localeListResolutionCallback, // 这个回调负责在应用程序启动时以及用户更改设备的区域设置时选择应用程序的区域设置。
  this.localeResolutionCallback, // 
  this.supportedLocales = const <Locale>[Locale('en', 'US')], // 此应用程序已本地化的地区列表 
  this.debugShowMaterialGrid = false, // 打开绘制基线网格材质应用程序的网格纸覆盖
  this.showPerformanceOverlay = false, // 打开性能叠加
  this.checkerboardRasterCacheImages = false, // 打开栅格缓存图像的棋盘格
  this.checkerboardOffscreenLayers = false, // 打开渲染到屏幕外位图的图层的棋盘格
  this.showSemanticsDebugger = false, // 打开显示框架报告的可访问性信息的覆盖
  this.debugShowCheckedModeBanner = true, // 在选中模式下打开一个小的“DEBUG”横幅，表示应用程序处于选中模式
}) 
```


<br/>
#`main.dart ` Code
```
import 'package:flutter/material.dart';
import 'package:flutter_navigaiton/tool/tool.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    //MaterialApp 是我们app开发中常用的符合MaterialApp Design设计理念的入口Widget
    return MaterialApp(
      title: 'Flutter 路由导航', //设备用于为用户识别应用程序的单行描述
      theme: ThemeData( //// 应用程序小部件使用的颜色
        primarySwatch: Colors.blue,
      ),
      home: NamedRouter.initApp(),  // 应用程序默认路由的小部件,用来定义当前应用打开的时候，所显示的界面
    );
  }
}





class NamedRouter {
  static Map<String, WidgetBuilder> routes;
//初始化App
  static Widget initApp() {
    return MaterialApp(
      initialRoute: '/',
      routes: NamedRouter.initRoutes(),// 应用程序的顶级路由表
    );
  }

//初始化路由
  static initRoutes() {
    routes = {
      '/': (context) => FirstScreen(),
      '/second': (context) => SecondScreen(),
      '/toolWidget': (context) => toolWidget()
    };
    return routes;
  }


}

class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Screen'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Launch screen'),
          onPressed: () {
            // Navigate to the second screen using a named route
            Navigator.pushNamed(context, '/second');
          },
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Screen"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // Navigate back to the first screen by popping the current route
            // off the stack
            // Navigator.pop(context);
            Navigator.pushNamed(context, "/toolWidget");
          },
          child: Text('push to toolWidget 组件'),
        ),
      ),
    );
  }
}

```

<br/>
#`tool.dart` Code
```
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

class toolWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("toolWidget"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // Navigate back to the first screen by popping the current route
            // off the stack
            Navigator.pop(context);
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

![Demo 文件目录](https://upload-images.jianshu.io/upload_images/2959789-c66896039d43f483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
效果：


![导航效果](https://upload-images.jianshu.io/upload_images/2959789-20aba33d1caf873f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
