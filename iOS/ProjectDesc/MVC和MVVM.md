
- **MVC**
	- MVC介绍
	- V对C的交流方式
	- MVC实战
- **MVVM**
- [**MVC和MVVM设计模式的那些事**](https://www.jianshu.com/p/caaa173071f3)
-  [**MVC、MVP和MVVM理解**](https://www.cnblogs.com/GJ-ios/p/13139565.html)
- [ **MVVM案例解读**](https://www.jianshu.com/p/db8400e1d40e)

<br/>

***
<br/>


># MVC

<br/>



> MVC介绍

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z52.png)

 
- **黄线**意味着你不能 穿越这黄线，任何一个方向都不行，即M和V完全分离。
- **白色的虚线**，它意味着你可以自由的穿越它，只要是安全的。
- **白色的实线**代表你可以穿越，但你必须要买票，或者交点过路费。

<br/>
- **C和M之间的绿色箭头**

&emsp;C和M之间的绿色箭头，这箭头的方向就代表着“发起对话”的方向，也就是说，发起对话的是C，而做出回答的是M。C可以问M各种各样的问题，但M 只是回答C的问题或要求，它不可以主动的向C要求什么。还记得虚线是畅通无阻的意思吧，所以，C知道M的所有的事情,如果用代码来说明这件事情，就是 说，C可以导入M的头文件或是M的接口（API）。因为C可以通过M的API，所以它就可以肆无忌惮的向M要求这要求那了。


<br/>

- **C和V之间绿色箭头**

&emsp;我们再来看看另外的一个绿色箭头，它是在C和V之间，和前一个绿色箭头的意义一样，它代表C可以直接地向V进行交流。你可以想想，C要把V放到屏幕 上，并设置V的属性，告诉它们什么时候从屏幕上消失，把它们分成组等等。如果C不能自由的向V发号施令的话，程序的显示将会多么的困难,所以，C可以毫无 限制地向V说话。

&emsp;可能你已经注意到了，这个箭头上还有outlet(输出口)，outlet可以看作是从C指向V的指针，它在C中被定义。outlet给我们提供了很大的 方便，它使我们在C的内部就可以轻松准确地向V施令。C可以拥有很多的outlet，可以不止一个，这也使它可以更高效的和V进行交流。


<br/>
- **M和V之间黄线隔开**

&emsp;那M和V之间可以交流么？还记得黄线的意思么？完全不可以通过，所以我们是不允许M和V进行交流的。这是因为我们不希望这三部分之间有过多的交流，你想想，假如V在显示时出现了问题，比如有一个图形没有显示出来，我们就要去查找错误，因为C可以和V交流，M也可以和V交流的话，我们就要去检查两个部分。 相反的，只有C可以和V交流的话，在出错时，我们就只需要去C那里查找原因，这样查找错误不就很是简单了么？所以，我们不允许M和V之间有直接的联系，这 也是在它们两之间有两根黄线的原因。 总结下来也就是以下三点：

（1）、Model和View永远不能相互通信，只能通过Controller传递。

（2）、Controller可以直接与Model对话（读写调用Model），Model通过Notification和KVO机制与Controller间接通信。

（3）、Controller可以直接与View对话，通过outlet,直接操作View,outlet直接对应到View中的控件,View通过action向Controller报告事件的发生(如用户Touch我了)。Controller是View的直接数据源（数据很可能是Controller从Model中取得并经过加工了）。Controller是View的代理（delegate),以同步View与Controller。


<br/>

> **V对C的交流方式：**

- 第一种我们称为目标操作(target-action)。

&emsp; 它是这样工作的，C会在自己的内部“悬挂”一个目标(target)，如图中的红白相间的 靶子，对应的，它还会分发一个操作(action，如图中的黄色箭头)给将要和它交流的视图对象(可能是屏幕上的一个按钮)，当按钮被按时，action 就会被发送给与之对应的target，这样V就可以和C交流了。但是在这种情况下，V只是知道发送action给对应的target,它并不知道C中的 类，也不知道它到底发送了什么。target-action是我们经常使用的方法。

<br/>

- 第二种方式我们叫做委托(delegate)。

&emsp; 有时候，V需要和C进行同步，你知道，用户交互不仅仅是什么按按钮，划滑块，还有很多种形式。好了， 让我们来看看图中的delegate黄色箭头，你发现箭头上又分出了四个小箭头：should，did，will，还有一个没标注的。绝大部分的 delegate信息都是should，will，did这三种形式。和英文意思相对应，should代表视图对象将询问C中的某个对象“我应该这么做 么？”，举个例子，有一个web视图，有人点击了一个链接，web视图就要问“我应该打开这个链接么？这样做安全么？”。这就是should信息。那 will和did呢？will就是“我将要做这件事了”，did就是“我已经做了这件事”。C把自己设置为V的委托(delegate),它让V知道：如 果V想知道更多的关于将如何显示的信息的话，就向C发送delegate信息。通过接受V发过来的delegate信息，C就会做出相应的协调和处理。还 有一点，每个V只能有一个delegate。

<br/>

- 第三种方式就是数据源(datasource)

&emsp; V不能拥有它所要显示的数据，记住这点非常重要。V希望别人帮助它管理将要显示的数据，当 它需要数据时，它就会请求别人的帮助,把需要的数据给它。再者，iphone的屏幕很小，它不能显示包含大量信息的视图。看图中的datasource箭 头，和delegate类似，V会发送cout，data at信息给C来请求数据。

&emsp; 对于不同的UIView，有相应的UIViewController，对应MVC中的C。例如在iOS上常用的UITableView，它所对应的Controller就是UITableViewController。


<br/>


![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z53.png)


- M：模型model的对象通常非常的简单。根据Apple的文档，model应包括数据和操作数据的业务逻辑。而在实践中，model层往往非常薄，不管怎样，model层的业务逻辑不应被拖入到controller。

- V：视图view通常是UIKit控件（component，这里根据习惯译为控件）或者编码定义的UIKit控件的集合。View的如何构建（PS：IB或者手写界面）何必让Controller知晓，同时View不应该直接引用model（PS：现实中，你懂的！），并且仅仅通过IBAction事件引用controller。业务逻辑很明显不归入view，视图本身没有任何业务。

- C：控制器controller。Controller是app的“胶水代码”：协调模型和视图之间的所有交互。控制器负责管理他们所拥有的视图的视图层次结构，还要响应视图的loading、appearing、disappearing等等，同时往往也会充满我们不愿暴露的model的模型逻辑以及不愿暴露给视图的业务逻辑。

&emsp; 网络数据的请求及后续处理，本地数据库操作，以及一些带有工具性质辅助方法都加大了Massive View Controller的产生。



<br/>


> MVC实战

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z54.png)

- Controller ->Model:单向通信，Controller需要将Model呈现给用户，因此需要知道模型的一切，还需要有同Model完全通信的能力，并且能任意使用Model的API。
- 2、Controller ->View:单向通信，Ccontroller通过View来布局用户界面。
- 3、Model ->View：永远不要直接通信，Model是独立于UI的，并不需要和View直接通信，View通过Controller获取Model数据。


<br/>

创建一个名为APP的model

**App.h**

```
#import <Foundation/Foundation.h>
@interface App : NSObject
@property (copy, nonatomic) NSString *name;
@property (copy, nonatomic) NSString *image;
@end
```


**App.m**

```
#import "App.h"
 
@implementation App
 
@end
```


**创建一个APPView.h**

```
#import <UIKit/UIKit.h>
@class App, AppView;
 
@protocol AppViewDelegate <NSObject>
@optional
- (void)appViewDidClick:(AppView *)appView;
@end
 
@interface AppView : UIView
@property (strong, nonatomic) App *app;
@property (weak, nonatomic) id<AppViewDelegate> delegate;
@end
```

**APPView.m**

```
#import "AppView.h"
#import "App.h"
 
@interface AppView()
@property (weak, nonatomic) UIImageView *iconView;
@property (weak, nonatomic) UILabel *nameLabel;
@end
 
@implementation AppView
 
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        UIImageView *iconView = [[UIImageView alloc] init];
        iconView.frame = CGRectMake(0, 0, 100, 100);
        [self addSubview:iconView];
        _iconView = iconView;
         
        UILabel *nameLabel = [[UILabel alloc] init];
        nameLabel.frame = CGRectMake(0, 100, 100, 30);
        nameLabel.textAlignment = NSTextAlignmentCenter;
        [self addSubview:nameLabel];
        _nameLabel = nameLabel;
    }
    return self;
}
 
- (void)setApp:(App *)app
{
    _app = app;
    self.iconView.image = [UIImage imageNamed:app.image];
    self.nameLabel.text = app.name;
}
 
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    if ([self.delegate respondsToSelector:@selector(appViewDidClick:)]) {
        [self.delegate appViewDidClick:self];
    }
}
 
@end
```

**ViewController.m**

```
#import "ViewController.h"
#import "App.h"
#import "AppView.h"
 
@interface ViewController () <AppViewDelegate>
 
@end
 
@implementation ViewController
 
- (void)viewDidLoad {
    [super viewDidLoad];
     
    // 创建view
    AppView *appView = [[AppView alloc] init];
    appView.frame = CGRectMake(100, 100, 100, 150);
    appView.delegate = self;
    [self.view addSubview:appView];
     
    // 加载模型数据
    App *app = [[App alloc] init];
    app.name = @"QQ";
    app.image = @"QQ";
     
    // 设置数据到view上
    appView.app = app;
}
 
#pragma mark - AppViewDelegate
- (void)appViewDidClick:(AppView *)appView
{
    NSLog(@"控制器监听到了appView的点击");
}
 
@end
```

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z55.png)






<br/>

***
<br/>




># MVVM

- **MVVM 的基本概念**
	- 在MVVM 中，view 和 view controller正式联系在一起，我们把它们视为一个组件
	- view 和 view controller 都不能直接引用model，而是引用视图模型（viewModel）
	- viewModel 是一个放置用户输入验证逻辑，视图显示逻辑，发起网络请求和其他代码的地方
	- 使用MVVM会轻微的增加代码量，但总体上减少了代码的复杂性


<br/>

- **MVVM 的注意事项**
	- view 引用viewModel ，但反过来不行（即不要在viewModel中引入#import UIKit.h，任何视图本身的引用都不应该放在viewModel中）（PS：基本要求，必须满足）
	- viewModel 引用model，但反过来不行

<br/>

- **MVVM 的使用建议**
	- MVVM 可以兼容你当下使用的MVC架构。
	- MVVM 增加你的应用的可测试性。
	- MVVM 配合一个绑定机制效果最好（PS：ReactiveCocoa你值得拥有）。
	- viewController 尽量不涉及业务逻辑，让 viewModel 去做这些事情。
	- viewController 只是一个中间人，接收 view 的事件、调用  viewModel 的方法、响应 viewModel 的变化。
	- viewModel 绝对不能包含视图 view（UIKit.h），不然就跟 view 产生了耦合，不方便复用和测试。
	- viewModel之间可以有依赖。
	- viewModel避免过于臃肿，否则重蹈Controller的覆辙，变得难以维护。


<br/>

![<br/>](http://upload-images.jianshu.io/upload_images/1874977-83316d550a75ca16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; `Controller`夹在`View`和`ViewModel`之间做的其中一个主要事情就是将`View`和`ViewModel`进行绑定。在逻辑上，`Controller`知道应当展示哪个`View`，`Controller`也知道应当使用哪个`ViewModel`来提供数据，然而`View`和`ViewModel`它们之间是互相不知道的，所以Controller仅关注于用 `view-model 的数据配置`和`管理各种各样的视图`。

<br/>

![MVVM模块层级图<br/>](https://upload-images.jianshu.io/upload_images/1874977-f8c57cfd22113349.png?imageMogr2/auto-orient/strip|imageView2/2/w/675)

Controller可以引用ViewModel和View；

ViewModel千万不要主动对视图控制器C以任何形式直接起作用或直接通告其变化，而是等待视图控制器C来主动获取；










