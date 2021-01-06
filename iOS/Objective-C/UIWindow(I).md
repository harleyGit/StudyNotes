> #继承关系图
![UIResponder 继承关系图](https://upload-images.jianshu.io/upload_images/2959789-c59555dbd67c93c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


># UIWindow 简单介绍

&emsp;&emsp; 一个iOS程序之所以能显示到屏幕上，完全是因为它有UIWindow。也就说，没有UIWindow，就看不见任何UI界面。

#`系统状态栏其实就是一个window ,程序启动的时候创建的默认的window ，弹出键盘也是一个window ，alterView 弹框也是window 。`

|UIWindow 作用|
|:---|
|①传递触摸消息和键盘事件给UIView；|
|②作为UIView的最顶层容器，包含应用显示所有的UIView|


官方层次图：
![UIScreen 层次图](https://upload-images.jianshu.io/upload_images/2959789-728c36563385ed02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>`UIScreen：`  代表了一个显示屏，iOS设备至少有一个自身的主显示屏，同时也有可能外接其他的显示设备。如果想将内容显示到屏幕上，需要设置UIWindow实例的screen属性为对应的UIScreen对象。<br/>
`UIWindow：`  是一种特殊的UIView，通常在一个App中只会有一个UIWindow。


&emsp;&emsp;  iOS程序启动完毕后，创建的第一个视图控件就是UIWindow，接着创建视图控制器，最后将控制器的view添加到UIWindow上，于是控制器的view就显示在屏幕上了。

![视图控制器附加到Window上，视图控制器自动的加入它的View作为window的子视图](https://upload-images.jianshu.io/upload_images/2959789-38559f39b4d05859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***`四大对象的关系`***
|对象                   |属性                                                         |
|:---:|:---:|
|UIApplication   |delegate属性                                           |
|AppDelegate                  |window属性                              |
| UIWindow                      |rootViewController属性           |
| UIViewController          |view属性                                    |

```
①第一步创建的对象是UIApplication
②UIApplication绑定一个AppDelegate对象
③AppDelegate对象中有一个window属性(UIWindow)
④UIWindow对象中有一个rootViewController的属性
⑤rootViewController设置成要显示的UIViewController
⑥最后 显示出UIViewController对象的view

```
![关系示意图](https://upload-images.jianshu.io/upload_images/2959789-d953b87a08914d5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
***
<br/>


> #添加View到UIWindow

```

//创建一个控制器，把view添加到uiwindow上面（有两种方式）

//①直接将控制器的view添加到UIWindow中，并不理会它对应的控制器

[self.window  addsubview:vc.view];

//②设置uiwindow的根控制器，自动将rootviewcontroller的view添加到window中，负责管理rootviewcontroller的生命周期

[self.window.rootviewcontroller=vc];

```

`两个方法的区别：`

&emsp;&emsp;  以后的开发中，建议使用②。因为方法①存在一些问题，比如说控制器上面可能由按钮，需要监听按钮的点击事件，如果是①，那么按钮的事件应该由控制器来进行管理。但控制器是一个局部变量，控制器此时已经不存在了，但是控制器的view还在，此时有可能会报错。注意：方法执行完，这个控制器就已经不存在了。

>`问题描述1：`当view发生一些事件的时候，通知控制器，但是控制器已经销毁了，所以可能出现未知的错误。<br/><br/>
>`问题描述2：`添加一个开关按钮，让屏幕360度旋转（两者的效果不一样）。当发生屏幕旋转事件的时候，UIapplication对象会将旋转事件传递给uiwindow,uiwindow又会将旋转事件传递给它的根控制器，由根控制器决定是否需要旋转<br/><br/>
UIapplication->uiwindow->根控制器（第一种方式没有根控制器，所以不能跟着旋转）。<br/><br/>
`提示：`不通过控制器的view也可以做开发，但是在实际开发中，不要这么做，不要直接把view添加到UIWindow上面去。因为，难以管理。


<br/>
***
<br/>


># 通知
|通知名|用法|
|:---|:---:|
|UIWindowDidBecomeVisibleNotification|当window激活时并展示在界面的时候触发，返回空|
|UIWindowDidBecomeHiddenNotification|当window隐藏的时候触发，暂时没有实际测，返回空|
|UIWindowDidBecomeKeyNotification|当window被设置为keyWindow时触发，返回空|
| UIWindowDidResignKeyNotification |当window的key位置被取代时触发，返回空|


<br/>
***
<br/>

># 方法和属性

<br/>


```
//默认是[UIScreen mainScreen]。修改屏幕的操作代价非常大
@property(nonatomic,retain) UIScreen *screen ; 

// default = 0.0，窗口级别越高，显示越靠前
//在windows数组里面，window是根据windowLevel来排列的，最后一个覆盖在最上面。这里的windows数组不包括系统提供的window,比如说状态栏就是在一个系统创建的window里面。
@property(nonatomic) UIWindowLevel windowLevel;

//是否是主窗口                    
@property(nonatomic,readonly,getter=isKeyWindow)BOOLkeyWindow;


//为子类提供。不要直接调用；当window变为keyWindow时会被自动调用来通知window。可以继承UIWindow重写此方法来实现功能
- (void)becomeKeyWindow; 

//为子类提供。不要直接调用；当window不再是keyWindow时(例如其他window实例调用了- makeKeyWindow或- makeKeyAndVisible方法)会被自动调用来通知window。可以继承UIWindow重写此方法来实现功能。
- (void)resignKeyWindow;  


 //成为主窗口
- (void)makeKeyWindow;

//简化方法，让窗口成为主窗口并且可见，如果要不可见，可以使用view的hidden属性
- (void)makeKeyAndVisible;


//让窗口成为主窗口，并且显示出来。用这个方法，才能把信息显示到屏幕上。
//UIWindow有makekeyandvisible这个方法，可以让这个Window凭空的显示出来，而其他的view没有这个方法，所以它只能依赖于Window，Window显示出来后，view才依附在Window上显示出来。
[self.window makekeyandvisible];

  
//UIWindow成为主窗口，但不显示。
[self.window makeKeyWindow];


//UIApplication调用window的该方法给window分发事件，window再将事件分发到合适的目标，比如将触摸事件分发到真正触摸的view上。可以直接调用该方法分发自定义事件。
- (void)sendEvent:(UIEvent *)event;     
              
// 把该window中的一个坐标转换成在目标window中时的坐标值
- (CGPoint)convertPoint:(CGPoint)point toWindow:(nullable UIWindow *)window; 
  
// 把目标window中的一个坐标转换成在该window中时的坐标值
- (CGPoint)convertPoint:(CGPoint)point fromWindow:(nullable UIWindow *)window;  

// 把该window中的一个矩阵转换成在目标window中时的矩阵值
- (CGRect)convertRect:(CGRect)rect toWindow:(nullable UIWindow *)window;

// 把目标window中的一个矩阵转换成在该window中时的矩阵值
- (CGRect)convertRect:(CGRect)rect fromWindow:(nullable UIWindow *)window;
```

<br/>
细说 windowLevel

>&emsp;&emsp;  ①我们生成的normalWindow虽然是在第一个默认的window之后调用makeKeyAndVisible，但是仍然没有显示出来。这说明当Level层级相同的时候，只有第一个设置为KeyWindow的显示出来，后面同级的再设置KeyWindow也不会显示。　<br/>　
&emsp;&emsp;   ②statusLevelWindow在alertLevelWindow之后调用makeKeyAndVisible，仍然只是显示在alertLevelWindow的下方。这说明UIWindow在显示的时候是不管KeyWindow是谁，都是Level优先的，即Level最高的始终显示在最前面。




<br/>
***
<br/>

># 获取UIWindow
```

//在应用中打开的UIWindow列表，这样就可以接触应用中的任何一个UIView对象(平时输入文字弹出的键盘，就处在一个新的UIWindow中)
[UIApplication sharedApplication].windows ;


//(获取应用程序的主窗口也就是说这个窗口是keyWindow)用来接收键盘以及非触摸类的消息事件的UIWindow，而且程序中每一个时刻只能有一个window是keyWindow
[UIApplication sharedApplication].keyWindow  

//提示：如果某个UIWindow内部的文本框不能输入文字，可能是因为这个UIWindow不是keyWindow


//获得某个UIView所在的UIWindow
self.view.window

```
`注意：keyWindow不是一成不变的，当你创建alertView或者ActionSheet的时候，它们所在的window会变成keyWindow。也就是说系统默认创建的window首先变成keywindow，而当弹框的时候，alertView所在的window变成keywindow，默认的keywindow变成非keywindow。`

<br/>
***
<br/>


> #Demo
```

- (void) clickAction:(UIButton *)sender {
    window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    window.backgroundColor = [UIColor redColor];
    window.windowLevel = UIWindowLevelAlert - 1;
    window.hidden = NO;
    UITapGestureRecognizer * tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(dismissWindow)];
    [window addGestureRecognizer:tap];

}


-(void)dismissWindow{
    window.hidden = YES;
    window = nil;
}

```
效果图：
![window 效果](https://upload-images.jianshu.io/upload_images/2959789-0dd05f79a657e822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`注意:  UIWindow不需要像View那样进行addSubview添加到视图层次中，当Window创建的时候，就自动显示，通过WindowLevel来决多个window同时存在时候，哪个在上层，哪个在下层。`



window在创建的时候，默认是UIWindowLevelNormal(0.0),这个值越大，层次越靠上。

windowLevel大于0，小于1000的时候，在statusbar之下，在默认的window之上
windowLevel大于1000的时候，就在statusbar之上了。`


<br/>
***
<br/>
参考资料：
 [点击状态栏回到顶部的功能失效的解决办法](https://www.cnblogs.com/junhuawang/p/6003191.html)
