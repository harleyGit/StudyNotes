- **坐标属性**
- **坐标获得**
- **坐标系转换**
- 




<br/>

***
<br/>


># 坐标属性

-  anchorPoint: 锚点，单位(0->1)。比如，把一张纸钉在墙上，这个钉子就是锚点，纸可以围着它转。同时锚点还是图层的把柄；
-  frame、bounds：旋转时它们的宽高不一样，当为竖直是一样的；
-  position：就是锚点在父视图中的位置；




<br/>

***
<br/>


># 坐标获得

<br/>

```
CGRectGetHeight    //返回View本身的高度

CGRectGetMinY      //返回View顶部的坐标

CGRectGetMaxY      //返回View底部的坐标

CGRectGetMinX     //返回View左边缘的坐标

CGRectGetMaxX     //返回View右边缘的坐标

CGRectGetMidX    //表示得到一个View中心点的X坐标

CGRectGetMidY    //表示得到一个View中心点的Y坐标
```

Demo

```
UILabel *label = [[UILabel alloc]initWithFrame:CGRectMake(10, 10, 110, 150)];

label.backgroundColor = [UIColor orangeColor];

[self.view addSubview:label];

NSLog(@"CGRectGetHeight--%f", CGRectGetHeight(label.frame));

NSLog(@"CGRectGetMaxX--%f", CGRectGetMaxX(label.frame));

NSLog(@"CGRectGetMaxY--%f", CGRectGetMaxY(label.frame));

NSLog(@"CGRectGetMidX--%f", CGRectGetMidX(label.frame));

NSLog(@"CGRectGetMidY--%f", CGRectGetMidY(label.frame));

NSLog(@"CGRectGetMinX--%f", CGRectGetMinX(label.frame));

NSLog(@"CGRectGetMinY--%f", CGRectGetMinY(label.frame));
```


<br/>

`打印结果`


```
2015-04-24 15:39:15.577 webView[15743:677046] CGRectGetHeight--150.000000

2015-04-24 15:39:15.577 webView[15743:677046] CGRectGetMaxX--120.000000

2015-04-24 15:39:15.577 webView[15743:677046] CGRectGetMaxY--160.000000

2015-04-24 15:39:15.577 webView[15743:677046] CGRectGetMidX--65.000000

2015-04-24 15:39:15.578 webView[15743:677046] CGRectGetMidY--85.000000

2015-04-24 15:39:15.578 webView[15743:677046] CGRectGetMinX--10.000000

2015-04-24 15:39:15.578 webView[15743:677046] CGRectGetMinY--10.000000
```

<br/>

***
<br/>

># 坐标系转换

<br/>

```
// 将像素point由point所在视图转换到目标视图view中，返回在目标视图view中的像素值
- (CGPoint)convertPoint:(CGPoint)point toView:(UIView *)view;
// 将像素point从view中转换到当前视图中，返回在当前视图中的像素值
- (CGPoint)convertPoint:(CGPoint)point fromView:(UIView *)view;

// 将rect由rect所在视图转换到目标视图view中，返回在目标视图view中的rect
- (CGRect)convertRect:(CGRect)rect toView:(UIView *)view;

/*
[viewB convertRect:viewC.frame toView:viewA];

viewA是目标，viewC是被操作的对象，那么剩下的viewB自然而然就是源了。作用就是计算viewB上的viewC相对于viewA的frame

*/

// 将rect从view中转换到当前视图中，返回在当前视图中的rect
- (CGRect)convertRect:(CGRect)rect fromView:(UIView *)view;

/*
[viewC convertRect:viewB.frame fromView:viewA];

viewA是源，viewB是被操作的对象，那么viewC就是目标。作用就是计算viewA上的viewB相对于viewC的frame。
*/


```

Demo

```
UIWindow *mainWindow = [[UIApplication sharedApplication].delegate window];
    //将rect从view中转换到当前视图中，返回在当前视图中的rect
CGRect absoluteRect = [self.view convertRect:self.view.bounds toView:mainWindow];
```

![效果图](https://upload-images.jianshu.io/upload_images/2959789-512c87c62462fd86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[UIView中的各种坐标转换](https://blog.csdn.net/deft_mkjing/article/details/52213939)

<br/>

***
<br/>
