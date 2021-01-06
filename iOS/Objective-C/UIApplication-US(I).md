```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
       application.idleTimerDisabled=true;
}
```

<br/>
**`var idleTimerDisabled: Bool`**
> ① 是一个布尔值，用来控制这个App在空闲的时候是否禁用；
② 这个属性的默认值是false。大多数应用程序在用户长时间内没有触动时，系统将设备放置到一个“休眠”的状态,屏幕变暗。这样做是为了节约资源。这个属性设置为true时，禁用“idle timer”，避免系统进入休眠；
③ 在大多数情况时我们应该将它设置为false，包括音频应用程序，但是有些比如游戏等应用程序需要将它设置为true；


<br/>
***
<br/>
&emsp;  Appdelegate 所持有唯一的 Window 对象是全局的，所以在 Appdelegate 文件中设置屏幕旋转也是全局有效的。
&emsp;  与info.plist中设置的屏幕方向参数，当使用了下面的函数的时候info.plist中的方向即无效（无下面函数的时候，即用plist中的设置），此处提供的方向是controller内可用方向的集合，当controller使用了超出集合外的方向，即报错！
```
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
     //支持竖屏和右旋转
    return  UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscapeLeft;
 
}
```
[iOS强制横屏和竖屏](https://blog.csdn.net/ghl2318560278/article/details/51579814)
[UIApplication sharedapplication用法总结 US](https://blog.csdn.net/huang2009303513/article/details/39501225)
[使用系统打开](https://blog.csdn.net/potato512/article/details/43968375)
