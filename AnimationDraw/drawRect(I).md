>#简介

&emsp;  iOS的绘图操作是在UIView类的drawRect方法中完成的，所以如果我们要想在一个UIView中绘图，需要写一个扩展UIView 的类，或者写一个UIVIew的子类，并重写drawRect方法，在这里进行绘图操作，程序会自动调用此方法进行绘图。

&emsp; 重绘操作是在drawRect方法中完成，但是苹果不建议直接调用drawRect方法，当然如果你坚持直接调用此方法，是没有效果的。

&emsp;  如果UIView初始化时没有设置rect大小，将直接导致drawRect不被自动调用。drawRect 调用是在Controller->loadView, Controller->viewDidLoad 两方法之后调用的。

调用drawRect方法的途径：
&emsp; ① 调用UIView类中的setNeedsDisplay方法，或者setNeedsDisplayInRect:触发drawRect:，但是有个前提条件是rect不能为0；

&emsp; ②在改变UIView的frame值的时候，会调用此方法；

&emsp; ③ 在改变UIView的backgroundColor属性时也会调用此方法(前提条件是该UIView的frame值已经设置了，否则不会调用drawRect此方法)。

&emsp; ④在调用sizeToFit后被调用，所以可以先调用sizeToFit计算出size。然后系统自动调用drawRect:方法。

>sizeToFit会自动调用sizeThatFits方法；<br/>
sizeToFit不应该在子类中被重写，应该重写sizeThatFits;<br/>
sizeThatFits传入的参数是receiver当前的size，返回一个适合的size;<br/>
sizeToFit可以被手动直接调用;<br/>
sizeToFit和sizeThatFits方法都没有递归，对subviews也不负责，只负责自己.

&emsp; ⑤通过设置contentMode属性值为UIViewContentModeRedraw。那么将在每次设置或更改frame的时候自动调用drawRect:。

代码：
`
Custom_OneView.m 文件中
`
```
#import "Custom_OneView.h"

@implementation Custom_OneView

- (void)drawRect:(CGRect)rect {
    //获得处理的上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    //设置线条样式
    CGContextSetLineCap(context, kCGLineCapSquare);
    //设置线条粗细宽度
    CGContextSetLineWidth(context, 1.0);
    //设置颜色
    CGContextSetRGBStrokeColor(context, 1.0, 0.0, 0.0, 1.0);
    
     //开始一个起始路径
    CGContextBeginPath(context);
    //起始点设置为(0,0)(也就是说画笔移动到该点开始画线,通俗点说落笔点是(0,0)):注意这是上下文对应区域中的相对坐标
    CGContextMoveToPoint(context, 0, 0);
    //设置下一个坐标点(画直线到该点)
    CGContextAddLineToPoint(context, 100, 100);
    CGContextStrokePath(context);
    
}
@end

```

##执行setNeedsLayout方法，打断点在drawRect方法中挥发现该方法执行
`
ViewController.m 文件
`

```
@interface ViewController ()

@property(nonatomic, strong) Custom_OneView *oneView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    self.oneView = [[Custom_OneView alloc] init];
    self.oneView.backgroundColor = [UIColor cyanColor];
    self.oneView.frame = CGRectMake(100, 100, 300, 300);

    [self.view addSubview:self.oneView];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.oneView.backgroundColor = [UIColor yellowColor];
    [self.oneView setNeedsLayout];
}

```

没点击空白处，没执行`setNeedsLayout`方法，效果图：

![没执行setNeedsLayout方法效果图](https://upload-images.jianshu.io/upload_images/2959789-482b565daf97be3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击空白处，执行`setNeedsLayout`方法，效果图：
![点击空白处执行`setNeedsLayout`方法效果图](https://upload-images.jianshu.io/upload_images/2959789-36ff35f13d357175.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##设置frame值，打断点在drawRect方法中挥发现该方法执行

`
ViewController.m 文件
`
```
@interface ViewController ()

@property(nonatomic, strong) Custom_OneView *oneView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    self.oneView = [[Custom_OneView alloc] init];
    [self.view addSubview:self.oneView];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.oneView.backgroundColor = [UIColor purpleColor];
    self.oneView.frame = CGRectMake(100, 100, 300, 300);
}
```

没点击空白处，没执行drawRect方法效果图：

![没点击空白处，没执行drawRect方法效果图](https://upload-images.jianshu.io/upload_images/2959789-d7e3fd3ba55a2a74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



点击空白处，执行drawRect方法效果图：

![点击空白处，执行drawRect方法效果图](https://upload-images.jianshu.io/upload_images/2959789-336d022ec02f2183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




##设置backgroundColor，执行drawRect方法

`
ViewController.m 文件
`
```
@interface ViewController ()

@property(nonatomic, strong) Custom_OneView *oneView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    self.oneView = [[Custom_OneView alloc] init];
    self.oneView.backgroundColor = [UIColor cyanColor];
    self.oneView.frame = CGRectMake(100, 100, 300, 300);

    [self.view addSubview:self.oneView];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.oneView.backgroundColor = [UIColor greenColor];
}
```

没点击空白地方，没有执行`touchesBegan:  withEvent:`方法。
效果图：
![没有点击空白处](https://upload-images.jianshu.io/upload_images/2959789-131a8af07fd218ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击空白后的效果图：
![运行效果图](https://upload-images.jianshu.io/upload_images/2959789-f83f940b5055f061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***


#自定义一个TextField
`
Custom_TextField.m文件
`
```
#import "Custom_TextField.h"

@implementation Custom_TextField

- (void)drawRect:(CGRect)rect {
    CGContextRef context = UIGraphicsGetCurrentContext();
    //设置填充颜色
    CGContextSetFillColorWithColor(context, [UIColor grayColor].CGColor);
    //补充当前填充颜色的rect
    CGContextFillRect(context, CGRectMake(0, CGRectGetHeight(self.frame) - 0.5, CGRectGetWidth(self.frame), 0.5));
}
@end

```

`
ViewController.m文件
`
```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];

    
    [self addTextField];
}

- (void) addTextField {
    self.userNameTF = [[Custom_TextField alloc] initWithFrame:CGRectMake(100, 100, 200, 40)];
    self.userNameTF.placeholder = @"请输入用户名";
    [self.view addSubview:self.userNameTF];
    
    self.userPWDTF = [[Custom_TextField alloc] initWithFrame:CGRectMake(100, 160, 200, 40)];
    self.userPWDTF.placeholder = @"请输入密码";
    [self.view addSubview:self.userPWDTF];
}
```

效果图：
![自定义TextField](https://upload-images.jianshu.io/upload_images/2959789-e190f1143626ba8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>#drawRect方法使用注意事项

&emsp;  ① 若使用UIView绘图，只能在drawRect：方法中获取相应的contextRef并绘图。如果在其他方法中获取将获取到一个invalidate 的ref并且不能用于画图。drawRect：方法不能手动显示调用，必须通过调用setNeedsDisplay 或 者 setNeedsDisplayInRect，让系统自动调该方法。

&emsp;  ②若使用calayer绘图，只能在drawInContext: 中（类似鱼drawRect）绘制，或者在delegate中的相应方法绘制。同样也是调用setNeedDisplay等间接调用以上方法

&emsp;  ③若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来掉用setNeedsDisplay实时刷新屏幕.
