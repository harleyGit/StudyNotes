> <h1 id=""></h1>
- [属性](#属性)
- [**控制器添加和移除**](#控制器添加和移除)
	- [添加ViewController](#添加ViewController)
	- [移除ViewController](#移除ViewController)



<br/>

***
<br/>


># <h1 id="属性">属性</h1>


<br/>

***
<br/>


># <h1 id="控制器添加和移除">控制器添加和移除</h1>

&emsp;  在日常的开发中常有这样一种需求，通过切换标签来切换不同的页面，如果在一个 controller 管理这些 view 的话，代码就显得耦合度过高，也与 Apple 的 MVC 模式不符合，Apple 推荐一个 controller 管理一个 view。 

&emsp;  这样就有人建议类似  [self.view addSubview:viewController.view]  的方式用另一个 controller 来控制。

&emsp;  但是这样会产生一个新的问题：直接 add 进去的subView不在 viewController的 view hierarchy 内，事件不会正常传递，如：旋转、触摸等，属于危险操作违背CocoaTouch开发的设计MVC原则，viewController应该且只应该管理一个view hierarchy。

&emsp;  同时我们也要考虑另一个问题：这些子 view 大多数不会一直处于界面上，只是在某些情况下才会出现，例如登陆失败的提示 view，上传附件成功的提示 view，网络失败的提示 view 等。但是虽然这些 view 很少出现，但是我们却常常一直把它们放在内存中。另外，当收到内存警告时，我们如何把这些 view 从 super view 中去掉？

为了解决以上的问题，在iOS5 苹果给出一些方法以及属性用来解决这些问题：

```
@property(nonatomic,readonly) NSArray<__kindof UIViewController *> *childViewControllers;

//添加ViewController，建立父子ViewController关系
//调用该方法时，会自动调用willMoveToParentViewController: 方法，但是不会自动调用didMoveToParentViewController: 方法，要显示调用
- (void)addChildViewController:(UIViewController *)childController;


//通知childController，即将解除父子关系，设置 childController的parentController即将为nil
- (void)willMoveToParentViewController:(nullable UIViewController *)parent;


- (void)didMoveToParentViewController:(nullable UIViewController *)parent;


//在移除childController前不会调用willMoveToParentViewController: ，所以需要显示调用这个方法
- (void)removeFromParentViewController;


- (void)transitionFromViewController:(UIViewController *)fromViewController toViewController:(UIViewController *)toViewController duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^ __nullable)(void))animations completion:(void (^ __nullable)(BOOL finished))completion;


```

<br/>

> <h2 id="添加ViewController">添加ViewController</h2> 

`ViewController.m 文件中`

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    [self.view addSubview:self.clickBtn];
}

- (void) clickAction:(UIButton *)sender {
    NewController *nc = [NewController new];
    
    [self addChildViewController:nc];
    //[nc willMoveToParentViewController:self];
    //将nc的view加到父类的view上去，当然还要确定子view在父类view上的frame，否则不显示
    [self.view addSubview:nc.view];
    //以通知childController，完成了父子关系的建立
    [nc didMoveToParentViewController:self];
}

```

效果：

![未点击Button前](https://upload-images.jianshu.io/upload_images/2959789-ff8535b6be275669.png?imageMogr2/auto-orient/strip%7CimageView2/2/h/240)

![点击Button后](https://upload-images.jianshu.io/upload_images/2959789-b65c77527c3c3673.png?imageMogr2/auto-orient/strip%7CimageView2/2/h/240)

<br/>
<br/>

> <h2 id="移除ViewController">移除ViewController</h2> 


`NewController.m`

```
- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor redColor];
    [self.view addSubview:self.clickBtn];
}

- (void) clickAction:(UIButton *)sender {
    
    //在移除childController前不会调用willMoveToParentViewController: ，所以需要显示调用这个方法
    [self willMoveToParentViewController:nil];
    
    //将childController的view从父类的view中移除
    [self.view removeFromSuperview];
    //解除父子ViewControllet的关系
    [self removeFromParentViewController];
    
}
```

效果图：

![NewController 移除前](https://upload-images.jianshu.io/upload_images/2959789-9ed2ba5db05e26f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

![NewController 移除后](https://upload-images.jianshu.io/upload_images/2959789-3230c7dc311b3025.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

<br/>

**addSubView: 和 addChildController: 先后调用顺序区别**

&emsp;  当childViewController没有被加到任何父视图控制器时，如果把childViewController的view加到别的视图上，viewWillAppear和viewDidAppear会正常调用。但是当childViewController被加到一个父视图控制器上后,要注意区别：

`①先调用addSubView，再调用addChildViewController`

 &emsp;  若先调用addSubView，viewWillAppear和viewDidAppear会各调用一次；，再调用addChildViewController时，与父视图控制器的事件同步，即当父视图控制器的viewDidAppear调用时，childViewController的viewDidAppear方法会再调用一次。`所以viewDidAppear方法被调用了两次`。

`②先调用addChildViewController，再调用addSubView`

 &emsp;   先调用addChildViewController，childViewController的事件与父视图控制器同步，当父视图控制器的viewDidAppear调用时，childViewController的viewDidAppear方法会调用一次，再调用addSubView也不会触发viewWillAppear和viewDidAppear。








<br/>

***
<br/>


<br/>

参考资料：

[addChildViewController 等方法详解](https://www.jianshu.com/p/2148f9cfa010)

[iOS addChildViewController详解](https://blog.csdn.net/shaobo8910/article/details/51453645)

