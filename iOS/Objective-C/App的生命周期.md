>#	APP的启动流程

[类库 思维导图](https://github.com/harleyGit/StudyNotes/blob/master/Ohters/类库.mindnode)

<br/>

-	从源代码到app
当我们点击了 build 之后，做了什么事情呢？
	-	预处理（Pre-process）：把宏替换，删除注释，展开头文件，产生 .i 文件。
	-	编译（Compliling）：把之前的 .i 文件转换成汇编语言，产生 .s文件。
	-	汇编（Asembly）：把汇编语言文件转换为机器码文件，产生 .o 文件。
	-	链接（Link）：对.o文件中的对于其他的库的引用的地方进行引用，生成最后的可执行文件（同时也包括多个 .o 文件进行 link）。


-	APP的生命流程
	-	APP的启动流程(pre-main)
```
1.iOS系统首先会加载解析该APP的Info.plist文件，因为Info.plist文件中包含了支持APP加载运行所需要的众多Key，value配置信息，例如APP的运行条件(Required device capabilities)，是否全屏，APP启动图信息等。

2.创建沙盒(iOS8后，每次启动APP都会生成一个新的沙盒路径)

3.根据Info.plist的配置检查相应权限状态

4.加载Mach-O文件读取dyld路径并运行dyld动态连接器(内核加载了主程序，dyld只会负责动态库的加载)
	4.1 首先dyld会寻找合适的CPU运行环境
	4.2 然后加载程序运行所需的依赖库和我们自己写的.h.m文件编译成的.o可执行文件，并对这些库进行链接。
	4.3 加载所有方法(runtime就是在这个时候被初始化的)
	4.4 加载C函数
	4.5 加载category的扩展(此时runtime会对所有类结构进行初始化)
	4.6 加载C++静态函数，加载OC+load
	4.7 最后dyld返回main函数地址，main函数被调用
```

*dyld说明:*
&emsp;	dyld叫做动态链接器，主要的职责是完成各种库的连接。dyld是苹果用C++写的一个开源库，可以在苹果的[git上直接查看源代码](https://github.com/opensource-apple/dyld)。
&emsp;	当系统从xnu内核态把控制权转交给dyld变成用户态后dyld首先初始化程序环境，将可执行文件以及相应的系统依赖库与我们自己加入的库加载进内存中，生成对应的ImageLoader类对应的image对象(镜像文件)，对这些image进行链接，调用各image的初始化方法等等(注:这里多数情况都是采用的递归，从底向上的方法调用)，其中runtime就是在这个过程中被初始化的，这些事情大多数在`dyld:_mian`方法中被发生。



	-	APP的初始化流程(main)

```
1.main 函数

2.执行UIApplicationMain
	2.1 创建UIApplication对象
	2.2 创建UIApplication的delegate对象
	2.3 创建MainRunloop
	2.4 delegate对象开始处理(监听)系统事件(没有storyboard)

3.根据Info.plist获得最主要storyboard的文件名,加载最主要的storyboard(有storyboard)
程序启动完毕的时候, 就会调用代理的application:didFinishLaunchingWithOptions:方法
在application:didFinishLaunchingWithOptions:中创建UIWindow
创建和设置UIWindow的rootViewController
最终显示第一个窗口

```
<br/>

\*main.m文件说明\*
```

import <UIKit/UIKit.h>
#import "AppDelegate.h"

//main函数是整个程序的入口
int main(int argc, char * argv[]) {
    //参数argc说明:命令行总的参数个数。
    //参数argv说明:是参数的数组，argv中第一个参数为app的路径＋全名。
    printf("argc = %d\n", argc);
    char *argChar = argv[0];
    printf("index = %i ,argv = %s\n", 0, argChar);
    @autoreleasepool {
        //UIApplicationMain函数说明
        //第一个参数argc:参数是main函数C语言中传入的，保持与main函数相同。
        //第二个参数argv:同argc参数一样
        //第三个参数nil:该参数为principalClassName (主要类名) 
        //    如果principalClassName是nil，那么它的值将从Info.plist去获取，如果Info.plist没有，则默认为UIApplication。
        //    principalClass这个类除了管理整个程序的生命周期之外什么都不做，它只负责监听事件然后交给delegateClass去做。
        //第四个参数NSStringFromClass([AppDelegate class]):委托代理类的类名，UIApplication创建的delegate对象的类名
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

<br/>

*UIApplication代理方法说明：*
```

//app启动完毕后就会调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    //打印方法名
    //本地通知的Key，UIApplicationDidFinishLaunchingNotification
    NSLog(@"--- %s ---",__func__);
}

//比如当有电话进来或短信进来或锁屏等情况下，这时应用程序挂起进入非活动状态。
//也就是手机界面还是显示着你当前的应用程序的窗口，只不过被别的任务强制占用了，也可能是即将进入后台状态(因为要先进入非活动状态然后才会进入后台状态)
- (void)applicationWillResignActive:(UIApplication *)application
{
    NSLog(@"--- %s ---",__func__);
}

//指当前窗口不是你的App,大多数程序进入这个后台会在这个状态上停留一会，时间到之后会进入挂起状态(Suspended)。
//如果你程序特殊处理后可以长期处于后台状态也可以运行。
//Suspended (挂起): 程序在后台不能执行代码。系统会自动把程序变成这个状态而且不会发出通知。
//当挂起时，程序还是停留在内存中的，当系统内存低时，系统就把挂起的程序清除掉，为前台程序提供更多的内存
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    /本地通知的Key，/UIApplicationDidEnterBackgroundNotification
    NSLog(@"--- %s ---",__func__);
    //当用户按下home键后，程序进入后台运行状态，如果内存不足被系统关闭或者用户手动杀掉程序，都不会调用applicationWillTerminate函数。
    //在程序进入后台时，添加一beginBackgroundTaskWithExpirationHandler(后台运行通知函数)，程序进入后台10分钟内，程序还在运行，并可以响应一些消息
    [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        NSLog(@"程序关闭");
    }];
}

//app程序程序从后台回到前台就会调用
- (void)applicationWillEnterForeground:(UIApplication *)application
{
    //本地通知的Key，UIApplicationWillEnterForegroundNotification
    NSLog(@"--- %s ---",__func__);
}


//app程序获取焦点就会调用
- (void)applicationDidBecomeActive:(UIApplication *)application
{
    //本地通知的Key，UIApplicationDidBecomeActiveNotification
    NSLog(@"--- %s ---",__func__);
}

// 内存警告，可能要终止程序，清除不需要再使用的内存
- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application
{
    NSLog(@"--- %s ---",__func__);
}

// 程序即将退出调用
- (void)applicationWillTerminate:(UIApplication *)application
{
    //UIApplicationWillTerminateNotification
    NSLog(@"--- %s ---",__func__);
}
```

<br/>

*生命周期*

*ViewController的生命周期方法说明:(详细说明都在代码注释中)*

```
#pragma mark --- sb相关的life circle
//执行顺序1
// 当使用storyBoard时走的第一个方法。这个方法而不走initWithNibName方法。
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
     NSLog(@"%s", __func__);
    if (self = [super initWithCoder:aDecoder])
     {
          //这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
     }
    return self;
}
#pragma mark --- life circle
//执行顺序1
// 当控制器不是SB时，都走这个方法。(xib或纯代码都会走这个方法)
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    NSLog(@"%s", __func__);
    if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) 
    {
        //这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
    }
    return self;
}

//执行顺序2
// xib加载完成时调用，纯代码不会调用。系统自行调用
- (void)awakeFromNib {
    [super awakeFromNib];
     //当awakeFromNib方法被调用时，所有视图的outlet和action已经连接，但还没有被确定。
     NSLog(@"%s", __func__);
}

//执行顺序3
// 加载控制器的self.view视图。(默认从nib)
- (void)loadView {
    //该方法一般开发者不主动调用，应该由系统自行调用。
    //系统会在self.view为nil的时候调用。当控制器生命周期到达需要调用self.view的时候会自行调用。
    //或者当我们设置self.view=nil后，下次需要用到self.view时，系统发现self.view为nil，则会调用该方法。
    //该方法一般会首先根据nibName去找对应的nib文件然后加载。
    //如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)
    NSLog(@"%s", __func__);
    //该方法比较特殊，如果重写不能调用父类的方法[super loadView];
    self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
}

//执行顺序4
//视图控制器中的视图加载完成，viewController自带的view加载完成后会第一个调用的方法
- (void)viewDidLoad {
    //当self.view被创建后，会立即调用该方法。一般用于完成各种初始化操作
    NSLog(@"%s", __func__);
    [super viewDidLoad];
}

//执行顺序5
//视图将要出现
- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewWillAppear:animated];
}

//执行顺序6
// view 即将布局其 Subviews
- (void)viewWillLayoutSubviews {
    //view即将布局它的Subviews子视图。 当view的的属性发生了改变。
    //需要要调整view的Subviews子视图的位置，在调整之前要做的工作都可以放在该方法中实现
    NSLog(@"%s", __func__);
    [super viewWillLayoutSubviews];
}

//执行顺序7
// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {
    //view已经布局其Subviews，这里可以放置调整完成之后需要做的工作
    NSLog(@"%s", __func__);
    [super viewDidLayoutSubviews];
}

//执行顺序8
//视图已经出现
- (void)viewDidAppear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewDidAppear:animated];
}

//执行顺序9
//视图将要消失
- (void)viewWillDisappear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewWillDisappear:animated];
}

//执行顺序10
//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewDidDisappear:animated];
}

//执行顺序11
// 视图被销毁
- (void)dealloc {
    //系统会在此时释放掉init与viewDidLoad中创建的对象
    NSLog(@"%s", __func__);
}

//执行顺序12
//出现内存警告  //模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
    //在内存足够的情况下，app的视图通常会一直保存在内存中，但是如果内存不够，一些没有正在显示的viewController就会收到内存不足的警告。
    //然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常我们需要在这里将不需要显示在内存中保留的对象释放它的所有权，将其指针置nil。
    NSLog(@"%s", __func__);
    [super didReceiveMemoryWarning];
}

```




	-	APP的运行时生命周期	 




<br/>

***
<br/>


>#	参考资料：

[US AppDelegate程序生命运行过程及使用时机](https://blog.csdn.net/zhw521411/article/details/52956055)
[静态库和动态库的制作和研究](https://www.jianshu.com/p/d643c1368c9d)
[视频 静态库和动态库基础要点](https://www.bilibili.com/video/BV1of4y1Q7cy?from=search&seid=4557030534076664019)




