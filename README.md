> <h1 id = "常用">常用</h1>

<br/>

> ## **Git提交规范**

```
- fix: 修复改bug;
- docs: 文档注释;
- style: 代码格式(不影响代码运行的变动);
- refactor: 重构(既不增加新功能，也不是修复bug);
- perf: 性能优化;
- test: 增加测试;
- chore: 构建过程或辅助工具的变动;
- revert: 回滚到上一个版本;
- merge：代码合并;
- sync：同步主线或分支的Bug;
- build: 打包;
- deps: 升级依赖;
- feat: 新功能;

⌘ -> command
⇧ -> shift
⌥ -> option
⬆ -> 上箭头
⬇ -> 下箭头
⌃ -> Control
```

<br/>
<br/>

> <h2 id='模块图片相对路径'>模块图片相对路径</h2>

- **iOS**
	- Objective-C
	- Swift

```
![](./../../Pictures/)
```

- **其他文件夹**

```
![](./../Pictures/)
```

- **标题**

```
<br/>

***
<br/>

># <h1 id=''></h1>

<br/>

> <h2 id=''></h2>
```


<br/>
<br/>

***
<br/>


> <h1 id="纲目">纲目</h1>
- [**Git提交规范**](#Git提交规范)
- [**C语言**](#C语言)
- [**iOS开发**](#iOS开发)
	- [**Objective-C**](#Objective-C)
	- [**逆向**](#逆向)
	- [**Swift**](#Swift)
	- [**错误总结**](#错误总结)
	- [**ProjectDesc**](#ProjectDesc)
- [**Go**](#Go)
- [**Flutter**](#Flutter)
- [**JavaScript**](#JavaScript)
- [**React**](#React)
- [**CSS**](#CSS)
- [**Egret Engine**](#EgretEngine)
- [**软件测试**](#软件测试)
- [**底层**](#底层)
- [**优化**](#优化)
- [**动画/绘图**](#动画/绘图)
- [**加密**](#加密)
- [**类库**](#类库)
- [**多线程**](#多线程)
- [**数据存储**](#数据存储)
- [**Network**](#Network)
- [**数据结构**](#数据结构)
- [**工具**](#工具)
- [**读书笔记**](#读书笔记)
- [**优秀资料**](#优秀资料)
- [**优秀文章**](#优秀文章)
- [**参考资料**](#参考资料)



<br/>

***
<br/>
￼

>#  <h1 id = "C语言">C语言</h1>

><h2 id=''> C</h2>
*	[基础](./C/C基础.md)
*	[指针](./C/C指针.md)

<br/>
<br/>

>## <h2 id = "C++">[C++](https://www.bookstack.cn/read/cppreference-language/b1364a863c4e10b2.md)</h2>
*	[**基础**](./C/基础.md)
*	[作用域](./C/作用域.md)
*	[基础语法](./C/基础语法.html)
*	[运算符](./C/C2运算符.md)
*	[类](./C/C2类.md)
*	[继承与多态](./C/C2继承与多态.md)
*	[类(III)](./C/类(III).md)
*	[类(IV)](./C/类(IV).md)
*	[命名空间](./C/命名空间.md)
*	[文件处理](./C/C2文件处理.md)
*	[修饰变量的存储类型关键字](./C/修饰变量的存储类型关键字.md)
*	[引用](./C/引用.md)
*	[类的高级应用](./C/C2类的高级应用.md)
*	[异常处理](./C/C2异常处理.md)
*	[C2Exercise(I)](./C/C2Exercise(I).md)
*	[排序算法](./C/排序算法.md)
*	[LeetCode(I)](./C/LeetCode(I).md)



<br/>
<br/>

***
<br/>



># <h1 id = "iOS开发">[iOS开发](https://github.com/apple)</h1>

<br/>

![进阶](./Pictures/ios_oc0_0.png)

<br/>
<br/>

> <h2 id = "Objective-C">Objective-C</h2>
* [**OC资料**](./iOS/Objective-C/OC资料.md)
* [**‌知识体系**](./iOS/Objective-C/知识体系.md)
*	[**iOS类继承**](./iOS/Objective-C/iOS类继承.md)
*	[**消息转发**](./iOS/Objective-C/消息转发.md)
*	[**App启动**](./iOS/Objective-C/App启动.md)
*	[**App优化**](./iOS/Objective-C/App优化.md)
*	[**App发布**](./iOS/Objective-C/App发布.md)
*	[**组件化(I)**](./iOS/Objective-C/组件化(I).md)
*	[**静态库动态库**](./iOS/Objective-C/静态库动态库.md)
*	[**动态化和热更新**](./iOS/Objective-C/动态化和热更新.md)
*	[**链接器**](./iOS/Objective-C/链接器.md)
*	[**‌开发协助**](./iOS/Objective-C/开发协助.md)
* [**iPhone适配**](./iOS/Objective-C/iPhone适配.md)
* [**埋点**](./iOS/Objective-C/埋点.md)
* [**问题解决**](./iOS/Objective-C/问题解决.md)
* [**多线程(I)**](./iOS/Objective-C/多线程(I).md)
* [**动画**](./iOS/Objective-C/动画.md)
* [**数据存储**](./iOS/Objective-C/数据存储.md)
* [**JSON数据**](./iOS/Objective-C/JSON数据.md)
* [路由导航](./iOS/Objective-C/路由导航.md)
* [**字符串**](./iOS/Objective-C/字符串.md)
* [**变量**](./iOS/Objective-C/变量.md)
* [**关键字**](./iOS/Objective-C/关键字.md)
* [**Block**](./iOS/Objective-C/Block.md)
* [**异常**](./iOS/Objective-C/异常.md)
* [iPhone设计规范](https://zhuanlan.zhihu.com/p/127780364)
* [本地化(I)](./iOS/Objective-C/本地化(I).md)
* [闭包](./iOS/Objective-C/闭包.md)
* [分类和扩展](./iOS/Objective-C/分类和扩展.md)
* [键盘弹出(I)](./iOS/Objective-C/键盘弹出(I).md)
* [**NSArray**](./iOS/Objective-C/NSArray.md)
* [响应链(I)](./iOS/Objective-C/响应链(I).md)
* [响应链(II)](./iOS/Objective-C/响应链(II).md)
* [远程推送(US)](./iOS/Objective-C/远程推送(US).md)
* [坐标系(I)](./iOS/Objective-C/坐标系(I).md)
* [App跳转](./iOS/Objective-C/App跳转.md)
* [keyChain](./iOS/Objective-C/keychain.md)
* [NSTimer(I)](./iOS/Objective-C/NSTimer(I).md)
* [Runtime(I)](./iOS/Objective-C/Runtime(I).md)
* [Video(I)](./iOS/Objective-C/Video(I).md)
* [App签名原理](./iOS/Objective-C/App签名原理.md)
* [单元测试(I)](./iOS/Objective-C/单元测试(I).md)
* [语法精练(I)](./iOS/Objective-C/语法精练(I).md)
* [WKWebView(I)](./iOS/WKWebView(I).md)
* [**GUI框架**](./iOS/Objective-C/GUI框架.md)
	* [UIButton](./iOS/Objective-C/UIButton.md)
	* [UICollectionView(I)](./iOS/Objective-C/UICollectionView(I).md)
	* [UIImage(I)](./iOS/Objective-C/UIImage(I).md)
	* [视图(I)](./iOS/Objective-C/视图(I).md)
	* [UIScrollView(I)](./iOS/Objective-C/UIScrollView(I).md)
	* [UISearchBar](./iOS/Objective-C/UISearchBar.md)
	* [UITabBarController](./iOS/Objective-C/UITabBarController.md)
	* [UITableView(I)](./iOS/Objective-C/UITableView(I).md)
	* [UITextField(I)](./iOS/Objective-C/UITextField(I).md)
	* [UIViewController(I)](./iOS/Objective-C/UIViewController(I).md)
	* [UIWindow(I)](./iOS/Objective-C/UIWindow(I).md)
* **优化**
	*	[内存优化](./iOS/Objective-C/内存优化.md)
	*	[性能优化](./iOS/Objective-C/性能优化.md)
* [**底层原理**](#底层原理)
	* [ARC原理](./iOS/Objective-C/ARC原理.md) 
	* [RunLoop(I)](./iOS/Objective-C/RunLoop(I).md)
*	**类库**
	*	[Linphone(I)](./iOS/Objective-C/Linphone(I).md)
	*	[YYCache](./iOS/Objective-C/YYCache.md)
	*	[AFNetworking](./iOS/Objective-C/AFNetworking.md)
- **项目开发**
	- [DevSum](./iOS/Objective-C/DevSum.md)


<br/>
<br/>



> <h2 id = "逆向">逆向</h2>
* [**资料集**](./iOS/Reverse/资料集.md)
* [**逆向(I)**](./iOS/Reverse/逆向(I).md)
* [**Mach-O格式**](./iOS/Reverse/Mach-O格式.md)







<br/>
<br/>

	       
>## <h2 id = "Swift">[Swift](https://swiftgg.gitbook.io/swift/yu-yan-can-kao)</h2>
* [资料集](./iOS/Swift/资料集.md)
* [**数据**](./iOS/Swift/数据.md)
* [Swif包管理](./iOS/Swift/Swift包管理.md)
* [SwiftUI(I)](./iOS/Swift/SwiftUI(I).md)
* [SwiftUI 官方文档地址](https://developer.apple.com/documentation/swiftui/)
* [SwiftUI 官方教程](https://developer.apple.com/tutorials/swiftui/)
* [SwiftUI 官方Demo](https://github.com/Jinxiansen/SwiftUI/tree/doc)
* [SwiftUI 翻译官方Demo](https://gitee.com/TheAlgorithms/SwiftUI)
* [闭包](./iOS/Swift/闭包.md)
* [枚举和结构体](./iOS/Swift/枚举和结构体.md)
* [枚举值](./iOS/Swift/枚举值.md)
* [构造器(I)](./iOS/Swift/构造器(I).md)
* [异常捕获](./iOS/Swift/异常捕获.md)
* [函数](./iOS/Swift/函数.md)
* [关键字(II)](./iOS/Swift/关键字(II).md)
* [关键字(I)](./iOS/Swift/关键字(I).md)
* [值引用类型](./iOS/Swift/值引用类型.md)
* [UIViewController](./iOS/Swift/UIViewController.md)
* [UICollectionView(I)](./iOS/Swift/UICollectionView(I).md)
* [Swift-服务端](./iOS/Swift/Swift-服务端.md)
* [String(I)](./iOS/Swift/String(I).md)
* [Swift 原理底层探索](https://blog.csdn.net/u013480070/article/details/103702845)
[](./iOS/Swift/.md)


<br/>
<br/>

	       
> <h2 id = "错误总结">错误总结</h2>
* [上架错误](./iOS/Error/上架错误.md)
* [审核被拒](./iOS/Error/审核被拒.md)
* [Error(I)](./iOS/Error/Error(I).md)


<br/>
<br/>

	       
> <h2 id = "ProjectDesc">ProjectDesc</h2>
* [XKProject](./iOS/ProjectDesc/XKProject.md)
* [**Desc(I)**](./iOS/ProjectDesc/Desc(I).md)
* [**Desc(II)**](./iOS/ProjectDesc/Desc(II).md)
* [**项目实战优化**](https://www.jianshu.com/u/751ece4bc422)
* [**Interview(I)**](./iOS/ProjectDesc/Interview(I).md)
* [**Interview(II)**](./iOS/ProjectDesc/Interview(II).md)
* [FlutterInterview](./iOS/ProjectDesc/FlutterInterview.md)
* [**Swift知识点(I)**](./iOS/ProjectDesc/Swift知识点(I).md)
* **设计模式**
	* [**单例类**](./iOS/ProjectDesc/单例类.md)
	* [**协议代理**](./iOS/ProjectDesc/协议代理.md)
	* [**KVC和KVO**](./iOS/ProjectDesc/KVC和KVO.md)
	* [**‌MVC和MVVM**](./iOS/ProjectDesc/MVC和MVVM.md)
	



<br/>

***
<br/>


> <h1 id = "Go">Go</h1>
- [语法](./Go/语法.md)
- [语法(II)](./Go/语法(II).md)


<br/>

***
<br/>


> <h1 id = "Flutter">Flutter</h1>
*	[资料集](./Flutter/资料集.md)
*	[类[I]](./Flutter/类[I].md)
*	[类[II]](./Flutter/类[II].md)
*	[组件(I)](./Flutter/组件(I).md)
*	[组件(II)](./Flutter/组件(II).md)
*	[布局(I)](./Flutter/布局(I).md)
*	[Deer项目解析(I)](./Flutter/Deer项目解析(I).md)
*	[Play(I)](./Flutter/BRPlay(I).md)
	*	[BRPlay配置说明](./Others/BRPlay/BRPlay配置说明.md)
*	[生命周期(I)](./Flutter/生命周期(I).md)
*	[常用命令](./Flutter/常用命令.md)
*	[性能优化(I)](./Flutter/性能优化(I).md)
*	[多线程和异步任务(I)](./Flutter/多线程和异步任务(I).md)
*	[多线程和异步任务(II)](./Flutter/多线程和异步任务(II).md)
*	[常用插件](./Flutter/常用插件.md)
*	[插件Dio](./Flutter/插件Dio.md)
*	[Stream](./Flutter/Stream.md)
*	[状态(I)](./Flutter/状态(I)**.md**)
*	[Flutter-系统方法调用顺序](./Flutter/Flutter-系统方法调用顺序.md)
*	[Flutter-使用-Charles进行抓包](./Flutter/Flutter-使用-Charles进行抓包.md)
*	[Flutter-路由导航](./Flutter/Flutter-路由导航.md)
*	[Flutter-插件(II)---oktoast](./Flutter/Flutter-插件(II)---oktoast.md)
*	[开发环境配置](./Flutter/开发环境配置.md)
*	[异常抛出](./Flutter/异常抛出.md)
*	[前端环境配置](./Flutter/前端环境配置.md)
*	[跨组件状态共享](./Flutter/跨组件状态共享.md)
*	[**分享**](#分享)
	* [WMShare(I)](./Flutter/WMShare(I).md)





<br/>

***
<br/>


> <h1 id = "JavaScript">JavaScript</h1>
- [**基础(I)‌**](./JavaScript/基础(I).md)
- [**字体自适应**](./JavaScript/字体自适应.md)




<br/>

***
<br/>


> <h1 id = "React">React</h1>
- [**资料集**](./React/资料集.md)
- [**Node**]()
	- [Node(I)](./React/Node(I).md)
- [**基础(I)**](./React/基础(I).md)
- [**基础(II)**](./React/基础(II).md)
- [**插件发布**](./React/插件发布.md)
- [create-react-app流程分析](./React/create-react-app流程分析.md)
- [性能优化]()
	- [性能优化(I)](./React/性能优化(I).md) 
- **知识综合**
	- [**ES6入门教程**](https://es6.ruanyifeng.com/)
	- [XKBKnowledge(I)](./React/XKBKnowledge(I).md)
	- [XKBKnowledge(II)](./React/XKBKnowledge(II).md)





<br/>

***
<br/>


> <h1 id = "CSS">CSS</h1>
- [布局](./CSS/布局.md)



<br/>

***
<br/>

> <h1 id = "EgretEngine">Egret Engine</h1>
* [位图使用](./EgretEngine/位图使用.md)
* [配置](./EgretEngine/ee配置.md)
* [项目伊始](./EgretEngine/ee项目伊始.md)
* [engine](./EgretEngine/engine.md)



<br/>

***
<br/>


> <h1 id = "软件测试">软件测试</h1>
-  	[**测试笔记(I)**](./SoftwareTest/SoftTest/测试笔记(I).md)







<br/>

***
<br/>

> <h1 id = "底层">底层</h1>
 *	[资料](./底层/资料.md)
 *	[自动释放池](./底层/自动释放池.md)
 *	[Class(I)](./底层/Class(I).md)
 *	[实例对象本质(I)](./底层/实例对象本质(I).md)
 *	[Runtime(I)](./底层/Runtime(I).md)
 *	[Runtime(II)](./底层/Runtime(II).md)
 *	[Runtime(III)](./底层/Runtime(III).md)
- [**内存管理**](./底层/内存管理.md)
- [**浅谈Mach-O**](./底层/浅谈Mach-O.md)
[](./底层/.md)




<br/>

***
<br/>

> <h1 id = "动画/绘图">动画/绘图</h1>
*	[离屏渲染](./AnimationDraw/离屏渲染.md)
*	[线条绘制(I)](./AnimationDraw/线条绘制(I).md)
*	[图片绘制(I)](./AnimationDraw/图片绘制(I).md)
*	[CoreAnimation(I)](./AnimationDraw/CoreAnimation(I).md)
*	[drawRect(I)](./AnimationDraw/drawRect(I).md)
*	[layoutSubviews(I)](./AnimationDraw/layoutSubviews(I).md)
*	[layoutSubviews(II)](./AnimationDraw/layoutSubviews(II).md)
[](./AnimationDraw/.md)







<br/>

***
<br/>



> <h1 id = "加密">加密</h1>
*	[加密(I)](./加密/加密(I).md)
*	[加密(II)](./加密/加密(II).md)
*	[Https[证书生成]](./加密/Https[证书生成].md)








<br/>

***
<br/>

> <h1 id = "类库">类库</h1>
*	[阿里云(I)](./ClassLibrary/阿里云(I).md)
*	[Podfile文档](./ClassLibrary/Podfile文档.md)
*	[极光推送](./ClassLibrary/极光推送.md)
*	[加密](./ClassLibrary/加密.md)
*	[今日头条(I)](./ClassLibrary/今日头条(I).md)
*	[类库常用关键字](./ClassLibrary/类库常用关键字.md)
*	[融云](./ClassLibrary/融云.md)
* 支付
	*	[微信分享(I)](./ClassLibrary/微信分享(I).md)
	*	[微信支付](./ClassLibrary/微信支付.md)
	*	[支付宝集成](./ClassLibrary/支付宝集成.md)
	*	[ApplePay(I)](./ClassLibrary/ApplePay(I).md)
*	图片
	*	[SDWebImage(I)](./ClassLibrary/SDWebImage(I).md)
	*	[Kingfisher(I)](./ClassLibrary/Kingfisher(I).md)
	*	[图片选择(I)](./ClassLibrary/图片选择(I).md)
	*	[RSwift(I)](./ClassLibrary/RSwift(I).md)
* 网络
	*	[Alamofire(I)](./ClassLibrary/Alamofire(I).md)
	*	[Moya(I)](./ClassLibrary/Moya(I).md)
*	[即时通讯](./ClassLibrary/即时通讯.md)
*	UI约束
	*	[FlexLib(I)](./ClassLibrary/FlexLib(I).md)
	*	[Masonry](./ClassLibrary/Masonry.md)
	*	[Snapkit](./ClassLibrary/Snapkit.md)
* JSON解析
	*	[HandyJSON(I)](./ClassLibrary/HandyJSON(I).md)
	*	[SwiftyJSON(I)](./ClassLibrary/SwiftyJSON(I).md)
	*	[JSONModel(I)](./ClassLibrary/JSONModel(I).md)
	*	[MJExtension(I)](./ClassLibrary/MJExtension(I).md)
*	响应式SDK
	*	[ReactiveCocoa](./ClassLibrary/ReactCocoa.md)
	*	[ReactiveCocoa(I)](./ClassLibrary/ReactiveCocoa(I).html)
	*	[ReactiveCocoa(II)](./ClassLibrary/ReactiveCocoa(II).html)
	*	[ReactiveCocoa(III)](./ClassLibrary/ReactiveCocoa(III).html)
	*	[RxSwift(I)](./ClassLibrary/RxSwift(I).md)
	*	[RxSwift(II)](./ClassLibrary/RxSwift(II).md)
	*	[RxSwift(III)](./ClassLibrary/RxSwift(III).md)
	*	[ReactiveNative(I)](./ClassLibrary/ReactiveNative(I).md)
	*	[ReactiveNative(II)](./ClassLibrary/ReactiveNative(II).md)
	*	[ReactiveObjc(I)](./ClassLibrary/ReactiveObjc(I).md)
	*	[ReactiveObjc(II)](./ClassLibrary/ReactiveObjc(II).md)
	*	[ReactiveObjc(III)](./ClassLibrary/ReactiveObjc(III).md)
	*	[ZFPlayer(I)](./ClassLibrary/ZFPlayer(I).md)
	*	[RxTheme](./ClassLibrary/RxTheme.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
*	[MJRefresh(I)](./ClassLibrary/MJRefresh(I).md)
*	[SVProgressHUD(I)](./ClassLibrary/SVProgressHUD(I).md)
*	**Bug收集**
	*	[plcrashreporter](https://github.com/microsoft/plcrashreporter)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)
[](./ClassLibrary/.md)





<br/>

***
<br/>



> <h1 id = "多线程">多线程 </h1>
*	[GCD(I)](./多线程/GCD(I).md) 
*	[GCD(II)](./多线程/GCD(II).md)    
* [NSOperation(I)](./多线程/NSOperation(I).md)    
* [NSOperation(II)](./多线程/NSOperation(II).md)    





<br/>

***
<br/>



> <h1 id = "数据存储">数据存储 </h1>
*	[数据持久化](./DataStorage/数据持久化.md)
*	[相册和视频处理(I)](./DataStorage/相册和视频处理(I).md)
*	[CoreData(I)](./DataStorage/CoreData(I).md)
*	[NSCache(I)](./DataStorage/NSCache(I).md)



<br/>

***
<br/>



> <h1 id = "Network">Network</h1>
*	[**数据解析**](./Network/数据解析.md)
*	[**网络安全(I)**](./Network/网络安全(I).md)
*	[**网络安全(II)**](./Network/网络安全(II).md)
*	[**HTTPS安全(I)**](./Network/HTTPS安全(I).md)
*	[**Header详解**](./Network/Header详解.md)
*	[**网络通信**](./Network/网络通信.md)
*	[**NSURLSession(I)**](./Network/NSURLSession(I).md)
*	[**NSURLSession(II)**](./Network/NSURLSession(II).md)
*	[**NSURLSession(III)**](./Network/NSURLSession(III).md)
*	[**Socket网络编程**](./Network/Socket网络编程.md)
*	[](./Network/.md)




<br/>

***
<br/>

> <h1 id = "数据结构">数据结构</h1>
- [数据结构](https://www.cnblogs.com/linuxAndMcu/category/1084577.html)
*	[递归](./数据结构/递归.md)
*	[线性表(I)](./数据结构/线性表(I).md)
*	[线性链表(II)](./数据结构/线性链表(II).md)
*	[队列](./数据结构/队列.md)
*	[二叉树](./数据结构/二叉树.md)
*	[栈](./数据结构/栈.md)
*	[](./数据结构/.md)



<br/>

***
<br/>

> <h1 id = "工具">工具</h1>
- [SourceTree](./Tools/SourceTree.md)
* [CocoaPods](./Tools/CocoaPods.md)
* [快捷键](./Tools/快捷键.md)
* [Mac](./Tools/Mac.md)
* [Podfile书写格式](./Tools/Podfile-书写格式.md)
* [Git](./Tools/Git.md)
* [VSCode](./Tools/VSCode.md)
* [Postman](./Tools/Postman.md)
* [IntelliJ系列软件](./Tools/IntelliJ系列软件.md)
* [Unix命令](./Tools/Unix命令.md)
* [Linux命令](./Tools/Linux命令.md)
* [codemagic](https://codemagic.io/apps)
	 * [Codemagic 持续部署 Flutter 应用](https://coldstone.fun/post/2020/02/03/flutter-cicd/)
* [Android 模拟器 For Mac](./Tools/Android-模拟器-For-Mac.md)
* [JAVA 和 Android 配置](./Tools/JAVA-和-Android-配置.md)



<br/>

***
<br/>





> <h1 id = "读书笔记">读书笔记</h1>
* 金融投资	
	*	[**金融理财(I)**](./ReadNotes/金融理财(I).md)
	*	[**金融理财(II)**](./ReadNotes/金融理财(II).md)
	*	[Financia(I)](./ReadNotes/Financia(I).md)
* 生活哲理
	*	[LifePhilosophy(I)](./ReadNotes/LifePhilosophy(I).md)
	*	[LifePhilosophy(II)](./ReadNotes/LifePhilosophy(II).md)
	*	[Emotion(I)](./ReadNotes/Emotion(I).md)
*	**其他**
	*	[others(I)](./ReadNotes/others(I).md)
* 思维导图
	*	[汽车史 思维导图](./ReadNotes/汽车史.xmind)








<br/>

***
<br/>




> <h1 id = "优秀资料">优秀资料</h1>
- **资源**
	- [**脚本之家电子书PDF下载**](https://www.jb51.net/books/)
	- [**Jiumo文档搜索引擎**](https://www.jiumodiary.com)
	- [**矢量图Flaticon**](https://www.flaticon.com)
	- [**矢量图iconfont**](https://www.iconfont.cn)
	-	[**矢量图 ICONS**](https://icons8.cn/icons)
	-	[**图标icons8**](https://icons8.cn/icon/set/foodstuff/ios)
	-	[**UI设计教程**](https://www.xueui.cn/tutorials/app-tutorials/how-to-design-app-icon.html)
	-	[**图库piabay**](https://pixabay.com)
-	**[Go 语言圣经](https://docs.hacknode.org/gopl-zh/ch0/ch0-01.html)**
-	[**极客学院[iOS高级开发]**](https://wiki.jikexueyuan.com/list/ios/)
-	[**Git教程**](https://bingohuang.gitbooks.io/progit2/content/01-introduction/sections/about-version-control.html)
-	[**极客方舟**](https://zhuanlan.zhihu.com/p/82986496)
-	[**算法网**](http://ddrv.cn)
-	[**看云[iBook]**](https://www.kancloud.cn/explore)
-	[**计算机语言教程**](http://codingdict.com/tutorials)
-	[**易百教程**](https://www.yiibai.com)
-	[**OC代码转Swift代码**](https://swiftify.com/home)
-	[**玩Android**](https://wanandroid.com)
-	[**Fuchsia中文社区**](https://fuchsia-china.com)





<br/>

***

<br/>




> <h1 id = "优秀文章">优秀文章</h1>
-	[iOS 底层安全分析(李斌同学)](https://juejin.im/user/3438928103236920/posts)
-	[Flutter 基础知识](http://www.xmamiga.com/3428/)
- 	[Cooci 老师博客(潭州课堂)](https://www.jianshu.com/u/5981a4f71db5)
-	[第五章 运输层（UDP和TCP三次握手，四次挥手分析）](https://www.cnblogs.com/whgk/p/6118206.html)
-	[官员同事Blog](https://zhangdinghao.cn)


<br/>

***
<br/>


> <h1 id = "参考资料">参考资料</h1>

* [GitHub Markdown 语法说明](https://github.com/riku/Markdown-Syntax-CN/blob/master/syntax.md#p)

<br/>

- **HTML标签定义**
<dl>
  <dt>定義列表</dt>
  <dd>有時候，人們偶爾會用到。</dd>
  <dt>在 HTML 中撰寫 Markdown</dt>
  <dd>*無法* 運作的 **非常** 好。改用 HTML<em>標籤</em>。</dd>
</dl>

<br/>

- **图片链接到影片**

</a>
	<a href="https://www.bilibili.com/video/BV1ux411Z797?from=search&seid=12561028895783817121" target="_blank">
	<img src="https://pic.17qq.com/uploads/plokohonlv.jpeg" alt="死神" width="260" height="333" border="30">
</a>

</a>
	<img src="./../Pictures/desktop2.tiff" alt="Mac桌面图片" width="960" height="540" border="30">
</a>


[![死神 BLEACH-w150](https://pic.17qq.com/uploads/plokohonlv.jpeg)](https://www.bilibili.com/video/BV1ux411Z797?from=search&seid=12561028895783817121)

<br/>

* **字体颜色测试**

```diff

- red

+ green

! orange

# gray

```






