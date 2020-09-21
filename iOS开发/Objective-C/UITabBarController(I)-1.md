![UITabBarController 结构图](https://upload-images.jianshu.io/upload_images/2959789-1b77d4693c1742b8.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***
<br/>


>#  iOS 12.1 后UITabBar的文字和图片出现移位

&emsp;&emsp;  出现的情况：使用iOS12.1系统 UINavigationController + UITabBarController（ UITabBar 磨砂），在popViewControllerAnimated 会遇到tabbar布局错乱的问题：
```
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated{
   [super pushViewController:viewController animated:animated];

   if (self.childViewControllers.count > 0) {
       //如果没这行代码，是正常显示的
       viewController.hidesBottomBarWhenPushed = YES;
   }
  
}
```

&emsp;&emsp; 原因：iOS 12.1 Beta 2 引入的问题，只要 UITabBar 是磨砂的，并且 push viewController 时 hidesBottomBarWhenPushed = YES 则手势返回的时候就会触发，出现这个现象的直接原因是 tabBar 内的按钮 UITabBarButton 被设置了错误的 frame，frame.size 变为 (0, 0) 导致的。

解决办法，加入这行代码：
```
//对全局的UITabBar样式进行修改
+ (instancetype)appearance;//返回接收方的外观代理(英文翻译)
//translucent属性能决定UITabBar/UINavigationBar是否为半透明的效果
@property(nonatomic,getter=isTranslucent) BOOL translucent;


[UITabBar appearance].translucent = NO;
```





<br/>
***
<br/>


<br/>
参考资料：
[详解及其定义](https://www.jianshu.com/p/2f74a5d93faa)
[translucent 详解](http://www.cnblogs.com/gfxxbk/p/5999663.html)
