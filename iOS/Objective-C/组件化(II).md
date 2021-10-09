> <h2 id=''></h2>
- [**组件化分层**](#组件化分层)
- [**组件化方案**](#组件化方案)
	- [本地组件化](#本地组件化)
	- [cocoapods组件化](#cocoapods组件化)




<br/>

***
<br/>

> <h1 id='组件化分层'>组件化分层</h1>

项目组件化优秀评判，可以通过以下几个方面：
 
- 模块之间没有耦合，模块内部的修改不影响其他模块；

- 模块可以单独编译；

- 模块间数据传递明确；

- 模块可以随时被另一个提供了相同功能的模块替换；

- 模块对外接口清晰且易维护；

- 当模块接口改变时，此模块的外部代码能够被高效重构；

- 尽量用最少的修改和代码，让现有的项目实现模块化；

- 支持OC和Swift，以及混编。
 
&emsp; 前4条主要用于衡量一个模块是否真正解耦，后4条主要用于衡量在项目实践中的易用程度。

<br/>

一般一个项目主要分为三层：业务层、通用层、基础层

![ios_oc25](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc25.webp)


- 组件化封层之后，需要遵循一下原则：
 
	- 只能上层对下层依赖， 不能下层对上层依赖（下层是对上层的抽象）；
	
	- 项目公共代码资源下沉；
	
	- 横向的依赖尽量少有，最好下称到通用模块或者基础模块。







<br/>

***
<br/>

> <h2 id='组件化方案'>组件化方案</h2>

- 目前常用的组件化方案主要有两种：

	- 本地组件化：主要是通过在 工程中创建 library， 利用 cocoapods 的 workspec 进行 本地管理， 不需要将项目上传git，而是直接在项目中以 framework 的方式 进行调用。

	- cocoapods组件化：主要是利用 cocoapods 来进行 模块的远程管理，需要将项目上传 git （这里的组件化模块分为 公有库 和 私有库 ， 对公司而言， 一般是私有库）


<br/>

> <h2 id='本地组件化'>本地组件化</h2>

- **创建主工程**

	- 新建项目主工程
	
	- 集成 cocoapods ， 进行本地管理，执行命令 pod init
	
	- 编辑 Podfile， 并执行 pod install

<br/>

- **创建组件**

- 可以创建 自己的模块：

	- 主工程：主要实现表层业务代码

	- Base：基类封装

	- Tools：工具（字符串，颜色，字体等）

	- Service：服务层，封装业务工具类，例如网络层服务、持久化服务等

	- Pods：第三方依赖

其中，各个模块间的关系如下所示：

![ios_oc26](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc26.webp)





<br/>
<br/>
<br/>


>## <h2 id='cocoapods组件化'>[cocoapods组件化](https://mp.weixin.qq.com/s/wGrjGsodJ1KZfGS2OksQKA)</h2>



<br/>

***
<br/>

> <h2 id=''></h2>






<br/>

***
<br/>

> <h2 id=''></h2>




<br/>

***
<br/>

> <h2 id=''></h2>
