> <h2 id=''></h2>
- [**环境配置**](#环境配置)
- [**介绍**](#介绍)
	- [优势](#优势)
	- [组件分类](#组件分类)
	- [生命周期](#生命周期)
- [**架构原理**](#架构原理)



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
	
	- InheritedWidget：用于做数据共享，比如：`Theme/ThemeData, Text/DefaultTextStyle`等等都是通过InheritedWidget
    来实现的数据共享，并且Flutter中的状态管理框架也都是通过它实现的，例如最为知名其中之一的状态管理框架Provider。
    
    &emsp; 那么这个数据共享是什么意思呢？
    
	 
	 - ParentDataWidget
    
![ProxyWidget组件类图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter6.png)

**Element提供渲染的方法，而Widget只是它的配置而已。**





<br/>
<br/>


> <h2 id='生命周期'>生命周期</h2>





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
