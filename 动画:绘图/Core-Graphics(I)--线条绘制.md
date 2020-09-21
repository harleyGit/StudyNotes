
>#***CGContext  图形上下文***
##几种图形处理的基本框架介绍：
&emsp;  Core Graphics/QuartZ 2D:Core Graphics是基于C的一套框架，使用了Quartz作为绘图引擎，所有基于Core Graphics的API都必须在视图的上下文中进行操作，QuartZ 2D是苹果公司开发的一套API，它是Core Graphics Framework的一部分。

&emsp; OpenGL ES:OpenGL ES是跨平台的图形API，属于OpenGL的一个简化版本，OpenGL ES是应用程序编程接口，描述了方法、结构、函数应具有的一些列行为以及应该如何被使用的API。也就是说它只是定义了一套规范，具体的实现由我们自己根据它制定规范去按步骤做，纯C语言。

<br/>
***


># 虚线的绘制
@ parameters  c                上下文；
@ parameters  phase        表示在第一个虚线绘制的时候跳过多少个点；
@ parameters   lengths = {10, 5, 6}     一个数组，表示先绘制10点，再跳过5点，再绘制6点，在跳过10点如此往复；
@ parameters  count        等于lengths数组的长度。
<br/>
`
CGContextSetLineDash(CGContextRef cg_nullable c, CGFloat phase,
                     const CGFloat * __nullable lengths, size_t count)
`

Demo 代码：
`YBStockChartView.m 文件`
```
#import "YBStockChartView.h"

@implementation YBStockChartView

- (void)drawRect:(CGRect)rect {
    [super drawRect:rect];
    
    
    CGContextRef context = UIGraphicsGetCurrentContext();
    // 重新开始画
    CGContextBeginPath(context);
    //设置描绘线的宽度
    CGContextSetLineWidth(context, 2.0);
    //设置描绘线的颜色
    CGContextSetStrokeColorWithColor(context, [UIColor whiteColor].CGColor);
    
    CGFloat lengths[] = {10,5};
    CGContextSetLineDash(context, 0, lengths, 2);
    //设置起点值
    CGContextMoveToPoint(context, 0.0, 20.0);
    //绘制直线
    CGContextAddLineToPoint(context, 310.0, 20.0);
    //直接在图形上下文中渲染路径
    CGContextStrokePath(context);
    
    CGContextSetLineDash(context, 5, lengths, 2);
    CGContextMoveToPoint(context, 0.0, 40.0);
    CGContextAddLineToPoint(context, 310.0, 40.0);
    CGContextStrokePath(context);
    
    CGContextSetLineDash(context, 8, lengths, 2);
    CGContextMoveToPoint(context, 0.0, 60.0);
    CGContextAddLineToPoint(context, 310.0, 60.);
    CGContextStrokePath(context);
    //闭合曲线
    CGContextClosePath(context);
}
@end
```

` ViewController.m 文件 `
```

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];

    [self describeView];
}

- (void) describeView{
    CGSize size   = self.view.frame.size;
    float  width  = size.width;
    float  height = size.height;
    self.scv = [[YBStockChartView alloc] initWithFrame:CGRectMake(0, 64, width, height)];
    
    [self.view addSubview:self.scv];
}
```

效果图：
![虚线绘制效果图](https://upload-images.jianshu.io/upload_images/2959789-080bd60c79121ab8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br>

># 直线绘制
&emsp; 此方法在`- (void)drawRect:(CGRect)rect`可以显示，但是在自己写的其他方法，无法绘制。若知道请留言告知，谢谢！
```
- (void)drawRect:(CGRect)rect {
    [super drawRect:rect];
    
    //获取画布
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    UIColor *strokeColor = [UIColor redColor];
    CGPoint startP = CGPointMake(80, 100);
    CGPoint endP = CGPointMake(300,  100);
    //填充勾画颜色
    CGContextSetStrokeColorWithColor(context, strokeColor.CGColor);
    //线条宽度
    CGContextSetLineWidth(context, 2);
    
    const CGPoint solidPoints[] = {startP, endP};
    //画线
    CGContextStrokeLineSegments(context, solidPoints, 2);
    
}

```

效果图:
![红线效果绘制](https://upload-images.jianshu.io/upload_images/2959789-cffb2d5126aef9de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>

>#股票阴线绘制

`YBStockChartView.m 文件`

```
#import "YBStockChartView.h"

@interface YBStockChartView()
@property(nonatomic, strong) UIColor *lineColor;
@end

@implementation YBStockChartView

- (void)drawRect:(CGRect)rect {
    [super drawRect:rect];
    CGPoint p1 = CGPointMake(100, 30);
    CGPoint p2 = CGPointMake(100, 70);
    CGPoint p3 = CGPointMake(100, 120);
    CGPoint p4 = CGPointMake(100, 170);
    
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetStrokeColorWithColor(context, [UIColor greenColor].CGColor);
    CGContextSetLineWidth(context, 2.0f);
    
    //p1->p2 线段
    CGContextMoveToPoint(context, p1.x, p1.y);
    CGContextAddLineToPoint(context, p2.x, p2.y);
    CGContextStrokePath(context);

    //p3 -> p4 线段
    CGContextMoveToPoint(context, p3.x, p3.y);
    CGContextAddLineToPoint(context, p4.x, p4.y);
    //直接在图形上下文中渲染路径
    CGContextStrokePath(context);
    //画方框
//    CGContextStrokeRect(context, CGRectMake(100-14/2.0, p2.y, 14, p3.y - p2.y));
    CGContextSetFillColorWithColor(context, [UIColor greenColor].CGColor);

    //填充框
    CGContextFillRect(context,CGRectMake(100-14/2.0, p2.y, 14, p3.y - p2.y));
    
}

@end
```

```
#import "ViewController.h"
#import "YBStockChartView.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];

    [self describeView];
}

- (void) describeView{
    CGSize size       = self.view.frame.size;
    float  width      = size.width;
    float  height     = 600;//size.height;
    
    YBStockChartView *scv = [[YBStockChartView alloc] initWithFrame:CGRectMake(0, 64, width, height/2)];
    self.scv.padding  = UIEdgeInsetsMake(10, 10, 0, 10);
    self.scv.prop     = 0.7;
    self.scv.delegate = self;
    
    [self.view addSubview:self.scv];
}
@end
```
效果图：
![股票阴线效果图](https://upload-images.jianshu.io/upload_images/2959789-9f418f266d5a8a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考资料：[绘图API](https://www.jianshu.com/p/e20a2ffc7583)
