> <h2 id=''></h2>
- [**环境配置**](#环境配置)
- [**介绍**](#介绍)
	- [优势](#优势)
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

***
<br/>




> <h1 id='架构原理'>架构原理</h1>

![源码路径图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter0.png)


<br/>

![Flutter框架图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/flutter1.png)






<br/>

***
<br/>




> <h1 id=''></h1>




<br/>

***
<br/>




> <h1 id=''></h1>
