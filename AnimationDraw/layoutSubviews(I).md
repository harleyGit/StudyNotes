

- **layoutSubviews**
	- layoutSubviews 会调用的情况
	- layoutSubviews 不会调用的情况




<br/>

***
<br/>


># layoutSubviews

简介：

&emsp;  iOS5.1和之前的版本，此方法的缺省实现不会做任何事情(实现为空)，iOS5.1之后(iOS6开始)的版本，此方法的缺省实现是使用你设置在此view上面的constraints(Autolayout)去决定subviews的position和size。

&emsp;  UIView的子类如果需要对其subviews进行更精确的布局，则可以重写此方法。只有在autoresizing和constraint-based behaviors of subviews不能提供我们想要的布局结果的时候，我们才应该重写此方法。可以在此方法中直接设置subviews的frame。 我们不应该直接调用此方法，而应当用setNeedsLayout、layoutIfNeeded 这两个方法。


<br/>

**TestView.h 文件**

```
#import <UIKit/UIKit.h>

@interface TestView : UIView

@end
```


**TestView.m 文件**

```
#import "TestView.h"

@implementation TestView


- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self)
    {
        NSLog(@"initWithFrame:%@" ,NSStringFromCGRect(frame));
    }
    return self;
}

//layoutSubviews方便数据计算, 调用先于drawRect
- (void)layoutSubviews
{
    NSLog(@"-------触发了layoutSubviews方法： %@", self);
    [super layoutSubviews];
    
    UIView *innerView = [[UIView alloc] initWithFrame:CGRectMake(self.center.x/2, self.center.y/2, 100, 100)];
    innerView.backgroundColor = [UIColor cyanColor];
    
    [self addSubview:innerView];
}


- (void)drawRect:(CGRect)rect {

    NSLog(@"-----调用了drawRect 方法了");
}

@end

```



- **layoutSubviews 会调用的情况**

<br/>

- ① addSubview会触发layoutSubviews；

```
 TestView *test1 = [[TestView alloc] init];
 [self.view addSubview:test1];
```
打印：

```
2018-08-12 18:29:52.793801+0800 StruggleSwift[9298:3032966] -------调用了initWithFrame:{{0, 0}, {0, 0}}
2018-08-12 18:29:55.594053+0800 StruggleSwift[9298:3032966] -------触发了layoutSubviews方法： <TestView: 0x7f903fe3a300; frame = (0 0; 0 0); layer = <CALayer: 0x600000432b40>>
```
	
<br/>

- ② 设置frame值和addSubview 会触发 layoutSubviews 和 drawRect 方法

```
TestView *test1 = [[TestView alloc] initWithFrame:CGRectMake(70, 70, 100, 100)];
[self.view addSubview:test1];
```
打印：

```
2018-08-12 18:52:00.897356+0800 StruggleSwift[10348:3256349] -------调用了initWithFrame:{{70, 70}, {100, 100}}
2018-08-12 18:52:00.938708+0800 StruggleSwift[10348:3256349] -------触发了layoutSubviews方法： <TestView: 0x7f81e2d05ac0; frame = (70 70; 100 100); layer = <CALayer: 0x604000225680>>
2018-08-12 18:52:00.939959+0800 StruggleSwift[10348:3256349] -----调用了drawRect 方法了
```

效果图：
![调用了layoutSubviews 和 drawRect 方法](https://upload-images.jianshu.io/upload_images/2959789-cc8f3c3d686ffea0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)

`结论：`当设置了frame值后，drawRect 方法会被系统调用。

<br/>

- ③ 调用 setNeedsLayout 方法，会调用 layoutSubviews

```
TestView *test1 = [TestView new];
[test1 setNeedsLayout];

[self.view addSubview:test1]; 
```
打印：

```
2018-08-12 19:07:22.155301+0800 StruggleSwift[10985:3407121] -------调用了initWithFrame:{{0, 0}, {0, 0}}
2018-08-12 19:07:22.197568+0800 StruggleSwift[10985:3407121] -------触发了layoutSubviews方法： <TestView: 0x7fd22c549b00; frame = (0 0; 0 0); layer = <CALayer: 0x604000223f20>>
2018-08-12 19:07:22.198063+0800 StruggleSwift[10985:3407121] -------触发了layoutSubviews方法： <TestView: 0x7fd22c549b00; frame = (0 0; 0 0); layer = <CALayer: 0x604000223f20>>
```

`结论`

&emsp;  由打印结果看 layoutSubviews 被执行了两次，第一次是因为 setNeedsLayout ，第二次是因为 addSubview 执行。layoutSubviews 被执行的前提条件是该视图被父视图调用 addSubview 否则，不会执行layoutSubviews方法。`这两次的执行是在 addSubview 方法执行完以后调用的。`

原因：setNeedsLayout 在receiver标上一个需要被重新布局的标记，异步调用layoutIfNeeded刷新布局，不立即刷新。在系统runloop的下一个周期自动调用layoutSubviews。

layoutSubviews 可以处理子视图中的一些数据。


<br/>

- ④ 同时使用 setNeedsLayout、layoutIfNeeded 后会立马调用layoutSubviews

```
TestView *test1 = [TestView new];
[test1 setNeedsLayout];
[test1 layoutIfNeeded];
[self.view addSubview:test1];
```
打印：

```
2018-08-12 19:14:13.678237+0800 StruggleSwift[11150:3473719] -------调用了initWithFrame:{{0, 0}, {0, 0}}
2018-08-12 19:14:23.766366+0800 StruggleSwift[11150:3473719] -------触发了layoutSubviews方法： <TestView: 0x7fbdd5d02aa0; frame = (0 0; 0 0); layer = <CALayer: 0x6040000372c0>>
2018-08-12 19:14:27.242705+0800 StruggleSwift[11150:3473719] -------触发了layoutSubviews方法： <TestView: 0x7fbdd5d02aa0; frame = (0 0; 0 0); layer = <CALayer: 0x6040000372c0>>
```

结论：

&emsp;  `通过打断点发现，在执行了 setNeedsLayout、layoutIfNeeded 方法后会立马调用layoutSubviews，`然后在 addSubview 后又调用了一次 layoutSubviews 方法。

&emsp;  layoutIfNeeded方法：在receiver标上一个需要被重新绘图的标记，在下一个draw周期自动重绘(iphone device的刷新频率是60hz，也就是1/60秒后重绘)，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）。

&emsp; 如果要立即刷新，要先调用[view setNeedsLayout]，把标记设为需要布局，然后马上调用[view layoutIfNeeded]，实现布局。

&emsp; 因为视图在第一次显示之前，标记总是“需要刷新”的，所以可以直接调用[view layoutIfNeeded]。

layoutIfNeeded遍历的不是superview链，应该是subviews链。 

drawRect是对receiver的重绘，能获得context


<br/>

⑤ 调用 setNeedsDisplay

```
TestView *test1 = [[TestView alloc] initWithFrame: CGRectZero];
[test1 setNeedsDisplay]; 
[self.view addSubview:test1];
```
打印：

```
2018-08-12 19:26:50.034083+0800 StruggleSwift[11791:3601469] -------调用了initWithFrame:{{0, 0}, {0, 0}}
2018-08-12 19:26:52.449600+0800 StruggleSwift[11791:3601469] -------触发了layoutSubviews方法： <TestView: 0x7f8aacf07c00; frame = (0 0; 0 0); layer = <CALayer: 0x600000038820>>
```

结论：

&emsp; 在调用addSubview 后，系统调用了layoutSubviews 方法。

<br/>

- ⑥ 滚动一个UIScrollView会触发layoutSubviews；

<br/>

- ⑦ 旋转Screen会触发父UIView上的layoutSubviews事件；

<br/>

- ⑧ 改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件；

<br/>
<br/>

- **layoutSubviews 不会调用的情况**

<br/>

- ①  init初始化不会触发layoutSubviews；

```
TestView *test1 = [[TestView alloc] init];
```
打印：

```
2018-08-12 18:27:25.653266+0800 StruggleSwift[9172:3008950] -------调用了initWithFrame:{{0, 0}, {0, 0}}
```

<br/>


- ② initWithFrame: 设置frame的值，但没有使用addSubView，也不会调用layoutSubviews；

```
TestView *test1 = [[TestView alloc] initWithFrame:CGRectMake(10, 10, 100, 100)];
```
打印：

```
2018-08-12 18:40:43.667931+0800 StruggleSwift[9499:3138994] -------调用了initWithFrame:{{10, 10}, {100, 100}}
```
