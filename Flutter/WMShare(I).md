> <h2 id=''></h2>
- [**环境配置**](#环境配置)
- [**介绍**](#介绍)
	- [优势](#优势)
	- [组件分类](#组件分类)
	- [生命周期](#生命周期)
- [**架构原理**](#架构原理)
	- [渲染三颗树](#渲染三颗树)



<br/>

***
<br/>




> <h1 id='介绍'>介绍</h1>

<br/>

> <h2 id='优势'>**优势**</h2>
- 提高开发效率
	- 同一份代码开发iOS和Android，不过需要对其风格进行适配
		- Android风格： **Material Design**
		- iOS风格： **Cupertino**
	- 用更少的代码做更多的事情
- 轻松迭代
	- 在应用程序运行时更改代码并重新加载（通过热重载），这个需要重新刷新下就好了，就像网站刷新一样
	- 修复崩溃并继续从应用程序停止的地方进行调试


<br/>
<br/>

&emsp; 就像iOS的Swift提出的万物皆对象，在Flutter中提出的则是万物皆Widget(也就是组件)，Widget是界面的基本构建元素。

组合 > 集成

许多功能强大的Widget通常由许多更小的、单一用途widget组成，比如：Container在Flutter很常用（相当于**`HTML中的div标签`**）， 它也由多个子widget组成，这些子widget负责布局、绘制、定位和调整大小。具体来说，Container由 LimitedBox、 ConstrainedBox、 Align、 Padding、 DecoratedBox、 和Transform组成。



<br/>
<br/>


> <h2 id='组件分类'>组件分类</h2>

![有无状态组件](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter2.png)

- **StatefulWidget:** 有状态组件，定义交互逻辑和业务数据，可以理解为具有动态可交互的内容界面，会根据数据的变化进行多次渲染。使用**`setState`**进行页面的类容的更新和刷新，这个和React一样。

![StatefulWidget组件类图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter3.png)


<br/>


- **StatelessWidget：** 无状态组件，外部传入的数据转化为界面展示的内容，只会渲染一次；


![StatelessWidget组件类图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter4.png)

<br/>

- **RenderObjectWidget（渲染对象）：**是RenderObjectElement的配置信息，是个抽象类；
	
	- RenderObjectElement也是个抽象类，其包装了RenderObject，RenderObject为应用程序提供真正的渲染，其系统子类：
		
		- LeafRenderObjectElement：叶子渲染对象对应的元素，处理没有children的renderObject;
			
			- MultiChildRenderObjectWidget(父级组件)
				
				- Flex/Wrap/Flow/Stack  
		
		- SingleChildRenderObjectElement：处理只有单个child的renderObject;
			
			- SingleChildRenderObjectWidget (父级组件)
				
				- RawImage(Imaget)/ErrorWidget 
		
		- MultiChildRenderObjectElement： 处理有多个children的渲染对象;
			
			- SingleChildRenderObjectWidget (父级组件)
				
				- Offstage/SizedBox/Align/Padding

![RenderObjectWidget组件类图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter5.png)


<br/>


- **ProxyWidget：** 提供一个提供子部件的部件，而不是构建新的部件；
	
	- [InheritedWidget](https://juejin.cn/post/6855129006707228680)：用于做数据共享，比如：`Theme/ThemeData, Text/DefaultTextStyle`等等都是通过InheritedWidget
    来实现的数据共享，并且Flutter中的状态管理框架也都是通过它实现的，例如最为知名其中之一的状态管理框架Provider。
    
    &emsp; 那么这个数据共享是什么意思呢？，如下面的Code：
    
    Main.dart文件
    
```
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: InheritedWidgetTestRoute(),
    );
  }
}
```

<br/>

**InheritedWidgetTestRoute.dart文件**

```
class InheritedWidgetTestRoute extends StatefulWidget {
  @override
  _InheritedWidgetTestRouteState createState() =>
      new _InheritedWidgetTestRouteState();
}

class _InheritedWidgetTestRouteState extends State<InheritedWidgetTestRoute> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("InheritedWidget"),
      ),
      body: Center(
        child: ShareDataWidget(
          //父widget
          data: count,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Padding(
                padding: const EdgeInsets.only(bottom: 20.0),
                //注意：如果不使用builder或者单独抽取成一个组件类，
                //而是像注释这样直接使用是错误的；
                //child: Text(ShareDataWidget.of(context).data.toString()),//错误用法，非子widget
                //用法1：
                // child: Builder(//子widget中依赖ShareDataWidget
                //   builder: (context) {
                //     return Text(ShareDataWidget.of(context).data.toString());
                //   },
                // ),

                //用法2：
                child: _TestWidget(),
              ),
              RaisedButton(
                child: Text("增加1"),
                //每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新
                onPressed: () => setState(() => ++count),
              )
            ],
          ),
        ),
      ),
    );
  }
}

class _TestWidget extends StatefulWidget {
  @override
  __TestWidgetState createState() => new __TestWidgetState();
}

class __TestWidgetState extends State<_TestWidget> {
  @override
  Widget build(BuildContext context) {
    //使用InheritedWidget中的共享数据
    return Text(ShareDataWidget.of(context).data.toString());
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("Dependencies change");
  }
}
```
    
<br/>

**ShareDataWidget.dart文件**

```
class ShareDataWidget extends InheritedWidget {
  ShareDataWidget({@required this.data, Widget child}) : super(child: child);

  //需要在子树中共享的数据，保存点击次数
  final int data;

  //定义一个便捷方法，方便子树中的widget获取共享数据
  static ShareDataWidget of(BuildContext context) {
    //dependOnInheritedWidgetOfExactType用法:https://juejin.cn/post/6855129006707228680
    return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
  }

  //该回调决定当data发生变化时，是否通知子树中依赖data的Widget
  @override
  bool updateShouldNotify(ShareDataWidget old) {
    //如果返回true，则子树中依赖(build函数中有调用)本widget
    //的子widget的`state.didChangeDependencies`会被调用
    return old.data != data;
  }
}
```
    
![效果图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter7.png)
	 
<br/>

 - ParentDataWidget
    
![ProxyWidget组件类图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter6.png)

**Element提供渲染的方法，而Widget只是它的配置而已。**





<br/>
<br/>


> <h2 id='生命周期'>生命周期</h2>

- [**createState**](#createState)
- [**initState**](#initState)
- [**didChangeDependencies**](#didChangeDependencies)
- [**build**](#build)
- [**addPostFrameCallback**](#addPostFrameCallback)
- [**didUpdateWidget**](#didUpdateWidget)
- [**deactivate**](#deactivate)
- [**dispose**](#dispose)


<br/>
<br/>


![z16](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z16.png)

上图是Flutter生命周期的示意图，其各个方法依次执行的分别是：

<br/>
<br/>

> <h3 id='createState'>createState</h3>

&emsp; createState 是 StatefulWidget 里创建 State 的方法，当要创建新的 StatefulWidget 的时候，会立即执行 createState，而且只执行一次，createState 必须要实现：

```
class MyScreen extends StatefulWidget {
@override
_MyScreenState createState() => _MyScreenState();
}
```


<br/>
<br/>

> <h3 id='initState'>initState</h3>


&emsp; 前面的 `createState` 是在创建 StatefulWidget 的时候会调用，initState 是 StatefulWidget 创建完后调用的第一个方法，而且只执行一次，类似于 **`Android 的 onCreate`**、**`iOS 的 viewDidLoad()`**，所以在这里 View 并没有渲染，但是这时 StatefulWidget 已经被加载到渲染树里了。

&emsp; 这时 StatefulWidget 的 mount 的值会变为 true，直到 dispose 调用的时候才会变为 false。可以在 initState 里做一些初始化的操作

&emsp; 在 override initState 的时候必须要调用 super.initState()：

```
@override
void initState() {
  super.initState();
  ...
}
```





<br/>
<br/>



> <h3 id='didChangeDependencies'>didChangeDependencies</h3>

&emsp; 当 StatefulWidget 第一次创建的时候，didChangeDependencies 方法会在 initState 方法之后立即调用，之后当 StatefulWidget 刷新的时候，就不会调用了，除非你的 StatefulWidget 依赖的 InheritedWidget 发生变化之后，didChangeDependencies 才会调用，所以 didChangeDependencies 有可能会被调用多次





<br/>
<br/>

> <h3 id='build'>build</h3>


&emsp; 在 StatefulWidget 第一次创建的时候，build 方法会在 didChangeDependencies 方法之后立即调用，另外一种会调用 build 方法的场景是，每当 UI 需要重新渲染的时候，build 都会被调用，所以 build 会被多次调用，然后 返回要渲染的 Widget。千万不要在 build 里做除了创建 Widget 之外的操作，因为这个会影响 UI 的渲染效率。






<br/>
<br/>


> <h3 id='addPostFrameCallback'>addPostFrameCallback</h3>


&emsp; addPostFrameCallback 是 StatefulWidge 渲染结束的回调，只会被调用一次，之后 StatefulWidget 需要刷新 UI 也不会被调用，addPostFrameCallback 的使用方法是在 initState 里添加回调：

```
import 'package:flutter/scheduler.dart';
@override
	void initState() {
	super.initState();
	SchedulerBinding.instance.addPostFrameCallback((_) => {
		
	});
}

```

&emsp； 渲染完成后，在这个方法里我们可以在[获取页面中Widget大小和位置](https://juejin.cn/post/6844903950257242119)，而且还可以进行网络接口请求




<br/>
<br/>




> <h3 id='didUpdateWidget'>didUpdateWidget</h3>


&emsp; `didUpdateWidget` 这个生命周期我们一般不会用到，只有在使用 key 对 Widget 进行复用的时候才会调用。

&emsp; 这个Key是Widget、Element和[SemanticsNode的](https://juejin.cn/post/6844904167085965326)标识符，只有当新的Widget的Key与当前Element中Widget的Key相同时，它才会被用来更新现有的Element。 Key在具有相同父级的Element之间必须是唯一的。

在案例枚举值**`KeyTest`**中可以看到。

而Widget的是否能够更新是根据它的一个源码方法：

```
static bool canUpdate(Widget oldWidget, Widget newWidget) {
return oldWidget.runtimeType == newWidget.runtimeType
    && oldWidget.key == newWidget.key;
}
```

&emsp; 来进行判断是否要更新，这里的`runtimeType`是其组件类型，而在上例中起类型都相同的，所以要根据其key的不同来进行判断。


&emsp; 这里有涉及到Flutter中的[3颗渲染树](#渲染三颗树)🌲，其分别是：**Widget Tree**、**Element Tree**、RenderObject Tree。

- Widget： Element的配置信息，与Element的关系可以是一对多，一份配置可以创造多个Element实例；

- Element：Widget 的实例化，内部持有Widget和RenderObject；

- RenderObject：负责渲染绘制。

类比下：
- Widget有点像是产品经理，负责规划产品、整理需求；
- Element则是UI设计师，根据原型整理出最终设计图；
- RenderObject就是我们程序开发者，负责具体的落地实现。





<br/>

> <h3 id='deactivate'>deactivate</h3>


&emsp; 当要将 State 对象从渲染树中移除的时候，就会调用 deactivate 生命周期，这标志着 StatefulWidget 将要销毁，但是有时候 State 不会被销毁，而是重新插入到渲染树种。



<br/>
<br/>



> <h3 id='dispose'>dispose</h3>

&emsp; 当 View 不需要再显示，从渲染树中移除的时候，State 就会永久的从渲染树中移除，就会调用 dispose 生命周期，这时候就可以在 dispose 里做一些取消监听、动画的操作，和 initState 是相反的





<br/>

**）.**



<br/>

**）.**



<br/>
<br/>



> <h2 id=''></h2>





<br/>
<br/>


> <h2 id=''></h2>





<br/>
<br/>


> <h2 id=''></h2>



在Flutter的类中

您可以通过实现widget的build返回widget树（或层次结构）来定义widget的独特特征 。 这棵树更具体地表示了用户界面的widget层次。例如，工具栏widget的build函数可能返回一个包含一些文本和各种按钮的水平布局。 然后，框架递归地构建widget，直到该所有widget构建完成，然后framework将他们一起添加到树中。

widget的构建函数一般没有副作用。每当它被要求构建时，widget应该返回一个新的widget树，无论widget以前返回的是什么。 Framework会将之前的构建与当前构建进行比较并确定需要对用户界面进行哪些修改。

这种自动比较非常有效，可以实现高性能的交互式应用程序。而构建函数的设计则着重于声明widget是由什么构成的，而不是将用户界面从一个状态更新到另一个状态的(这很复杂性)，从而简化了代码。









<br/>

***
<br/>




> <h1 id='架构原理'>架构原理</h1>



> <h2 id='渲染三颗树'>渲染三颗树</h2>

![3棵🌲](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter12.png)





<br/>
<br/>



> <h2 id='渲染三颗树'>渲染三颗树</h2>



![源码路径图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter0.png)
源码路径图


<br/>

![Flutter框架图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter1.png)
分层框架图






<br/>

***
<br/>




> <h1 id=''></h1>




<br/>

***
<br/>




> <h1 id=''></h1>
