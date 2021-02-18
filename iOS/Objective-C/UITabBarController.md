
- **介绍**
	- [1](https://www.jianshu.com/p/f1b0be2c9735)
	- [2](https://www.jianshu.com/p/a7c5f4028475)
	- [3](https://www.cnblogs.com/jukaiit/p/5066468.html)
	- [4](https://www.cnblogs.com/code-cd/p/4810408.html)
	- [**详解及其定义**](https://www.jianshu.com/p/2f74a5d93faa)
	- [**translucent 详解**](http://www.cnblogs.com/gfxxbk/p/5999663.html)
- **版本问题解决**
	- 文字和图片出现移位


<br/>

***
<br/>

># 介绍

![UITabBarController 结构图](https://upload-images.jianshu.io/upload_images/2959789-1b77d4693c1742b8.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

![UITabBarController内部构造图](https://upload-images.jianshu.io/upload_images/2959789-782e2483f7e29b4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



> &emsp; 因为UITabBarController类继承自UIViewController类，标签栏控制器有自己的视图，可以通过视图属性访问。标签栏控制器的视图只是标签栏视图和包含自定义内容的视图的容器。选项卡栏视图为用户提供了选择控件，并由一个或多个选项卡栏项组成。图2显示了如何组装这些视图以显示整个选项卡栏接口。尽管选项卡栏和工具栏视图中的项目可以更改，但管理它们的视图不会更改。只有自定义内容视图更改以反映当前选中选项卡的视图控制器。

>&emsp; 您可以使用导航控制器或自定义视图控制器作为选项卡的根视图控制器。如果根视图控制器是一个导航控制器，标签栏控制器会进一步调整显示的导航内容的大小，这样它就不会与标签栏重叠。因此，在选项卡栏接口中显示的任何视图都应该设置其autoresizingMask属性，以便在任何条件下适当调整视图的大小。








<br/>

***
<br/>


># 版本问题解决


- **iOS 12.1 后UITabBar的文字和图片出现移位**

&emsp;出现的情况：使用iOS12.1系统 UINavigationController + UITabBarController（ UITabBar 磨砂），在popViewControllerAnimated 会遇到tabbar布局错乱的问题：

```
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated{
   [super pushViewController:viewController animated:animated];

   if (self.childViewControllers.count > 0) {
       //如果没这行代码，是正常显示的
       viewController.hidesBottomBarWhenPushed = YES;
   }
  
}
```

&emsp;原因：iOS 12.1 Beta 2 引入的问题，只要 UITabBar 是磨砂的，并且 push viewController 时 hidesBottomBarWhenPushed = YES 则手势返回的时候就会触发，出现这个现象的直接原因是 tabBar 内的按钮 UITabBarButton 被设置了错误的 frame，frame.size 变为 (0, 0) 导致的。

解决办法，加入这行代码：

```
//对全局的UITabBar样式进行修改
+ (instancetype)appearance;//返回接收方的外观代理(英文翻译)
//translucent属性能决定UITabBar/UINavigationBar是否为半透明的效果
@property(nonatomic,getter=isTranslucent) BOOL translucent;


[UITabBar appearance].translucent = NO;
```


