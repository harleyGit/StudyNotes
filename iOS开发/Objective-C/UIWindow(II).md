>#  状态栏覆盖的View

```
#RedController.m

- (void)viewDidLoad {
    [super viewDidLoad];
    
    
    self.view.backgroundColor = [UIColor lightGrayColor];
    CGFloat width = [UIScreen mainScreen].bounds.size.width;
    CGFloat height = [UIScreen mainScreen].bounds.size.height;;
    UIView * subview = [[UIView alloc] initWithFrame:CGRectMake(width/2,0,width/2,height)];
    subview.backgroundColor = [UIColor redColor];
    [self.view addSubview:subview];
}


#ViewController.m

- (void) clickAction:(UIButton *)sender {
    window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    window.backgroundColor = [UIColor redColor];
    window.windowLevel =  UIWindowLevelStatusBar + 1;
    window.hidden = NO;
    
    RedController * rc = [RedController new];
    window.rootViewController = rc;
    
    UITapGestureRecognizer * tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(dismissWindow)];
    [window addGestureRecognizer:tap];

}

-(void)dismissWindow{
    window.hidden = YES;
    window = nil;
}


```

效果图：
![状态栏覆盖掉](https://upload-images.jianshu.io/upload_images/2959789-d48a117e9a50a3ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>


># keyWindow 与 delegate.window 的区别

<br/>
```
#pragma mark -- 二者区别
[UIApplication sharedApplication].keyWindow;


[[UIApplication sharedApplication] delegate].window;
```

`代码`

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    
    NSLog(@"viewDidLoad");
    NSLog(@"--------------");
    NSLog(@"keyWindow: %@",[UIApplication sharedApplication].keyWindow);
    NSLog(@"window: %@", [[UIApplication sharedApplication] delegate].window);
    
    [self.view addSubview:self.clickBtn];
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear: animated];
    
    NSLog(@"viewWillAppear");
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    
    NSLog(@"viewDidAppear");
    
    NSLog(@"***********");
    NSLog(@"keyWindow: %@",[UIApplication sharedApplication].keyWindow);
    NSLog(@"window: %@", [[UIApplication sharedApplication] delegate].window);
}
```
![二者区别](https://upload-images.jianshu.io/upload_images/2959789-4bf7558e9a1e2b0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>&emsp;&emsp;  ` [UIApplication sharedApplication].keyWindow`  这种写法之前一直也觉得是正确的，因为网上大多数的博客或者资料中都是这样写的，但是在最近的项目，发现这样写是不安全的。<br/>
&emsp;&emsp;   如果应用程序没有跳转，这种写法还算是可行的，但是如果应用程序出现了跳转（分享跳转到其他APP，访问系统相册等），这时返回原APP，你会发现加载原窗口上的视图位置会发生明显偏移，查阅了一些资料，发现如果写成  `[[[UIApplication sharedApplication]delegate]window]`  就不会出现上述问题，如果大家在项目中遇到此问题，不妨试试这种写法。





<br/>
***
<br/>

[UIWindow(I)](https://www.jianshu.com/p/5986ebe5005a)

<br/>
参考资料：
[UIWindow与视图调整技巧](https://www.jianshu.com/p/9373a7e0e34e)
