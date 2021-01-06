># 动画框架图介绍
![动画框架](https://upload-images.jianshu.io/upload_images/2959789-ab6a0ba9b3fcaa60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  UIKit、AppKit:  属于应用层，处理与用户的交互，包括：CGImageRef、CGColorRef;
-  CoreAnimation:  在OpenGL、OpenGL ES 和CoreGraphics基础上封装了一套动画的API;
      -  Core Graphics：底层框架，提供了一套2D绘图的API，使用C语言来写的。最重要的对象是`上下文(graphicsContext)`相当于一个画布，用来存储绘图状态。
-  OpenGL、OpenGL ES 和CoreGraphics，提供一些接口来访问GPU;
-  Graphics Hardware:  最底层是图形硬件 GPU、CPU，是跨平台的.


#`核心动画类结构`
![动画类结构](https://upload-images.jianshu.io/upload_images/2959789-a4ed14863205a659.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  CAAnimation 动画计时类；
    -     CATransition 转场动画，界面之间跳转（切换）；
    -     CAAnimationGroup 是个动画组，可以同时进行缩放，旋转；
    -     CAPropertyAnimation 也是个抽象类，本身不具备动画效果，只有子类才有。
-  CAConstraint 布局约束类；
-  CALayer 图层类；
- CAMediaTimingFunction: 一种函数，它将动画的节奏定义为时间曲线,如：二维码的[扫描线](https://www.cnblogs.com/YouXianMing/p/4421492.html);
-  CATransaction 事物类，可以同时设置多个layer层的动画效果。可以通过隐式和显式两种方式来进行动画操作；
-  CAConstraintLayoutManager:提供基于约束的布局管理器的对象;
-  CARenderer:  一种允许应用渲染一个图层树到核心OpenGL的上下文中的图层。


#`类继承关系`
![类继承关系图](https://upload-images.jianshu.io/upload_images/2959789-edc3c2af0cd66cd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)












<br/>
***
<br/>


># 图层 CALayer
&emsp;  使用Core  Animation 有三组图层对象。每一组图层对象在使应用程序的内容出现时都有不同的作用：

-  模型图层树中的对象(或者简单地说是“图层树”)是应用程序中交互最多的对象。此树中的对象存储任何动画的目标值。无论何时更改图层的属性，都使用图层树中的其中一个图层。
- 呈现图层树中的对象包含任何运行中的当前值。图层树对象包含动画的目标值，而展现的图层树中的图层对象则反映当前的值。你不应该修改此树中的对象。相反，您可以使用这些对象读取当前的动画值，可能是从这些值开始创建一个新的动画。
-  呈现树中的对象执行实际的回缩，并且对Core是私有的。

&emsp;  每一组图层对象都被组织成一个分层结构，就像应用程序中的视图一样。事实上，对于为其所有视图启用层的应用程序，每个树的初始结构与视图层次结构完全匹配。但是，应用程序可以根据需要将额外的层对象-将与视图无关的图层-添加到需要层次的图层。在不需要视图的所有开销的情况下，您可能会这样做，以优化应用程序对于不需要所有开销的内容的性能。图1-9显示了一个简单的iOS应用程序中的分层情况。示例中的窗口包含一个内容视图，它本身包含一个按钮视图和两个高级层对象。每个视图都有一个对应的层对象，该对象构成层次结构的一部分。
![图层结构](https://upload-images.jianshu.io/upload_images/2959789-1bc6783de5287194.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![图层交互联系](https://upload-images.jianshu.io/upload_images/2959789-0bf4df9cc8b41c7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>
># 贝塞尔曲线


![贝塞尔曲线绘制点](https://upload-images.jianshu.io/upload_images/2959789-a80388c2c56ad4a3.gif?imageMogr2/auto-orient/strip)

&emsp;  通过控制点来绘制曲线的，Control point 控制点是用来控制曲线弧度的，Current point、Endpoint 被称为数据点。

CAShapelayer:用来绘制各种图形的；
`ratationMode = KCCAAnimationRotateAuto //表示一个常量让它沿着线走，不至于离开线` 






<br/>
***
<br/>
># 动画
#`APP内部：`
  -  布局：属性设置frame、color；
  -  创建：drawRect、contnets；
  -  准备：各种渲染参数，解压图片。下载下来的都是压缩图片，渲染时才开始解压；
  -  提交：coreAnimation，把数据通过IPC进程间的通信，传递给Renderserver。

 `APP外部`：通过CPU计算好现实内容，布局计算，文本绘制，图片渲染。





<br/>
***
<br/>





<br/>
***
<br/>
资料：
[[CAKeyframeAnimation](https://blog.csdn.net/xy_26207005/article/details/51505837)
](https://www.jianshu.com/p/5158b0a33f48)
[让你的应用动起来](http://www.cnblogs.com/kenshincui/p/3972100.html#autoid-3-0-0)

[关键帧动画 CAKeyframeAnimation](https://blog.csdn.net/xy_26207005/article/details/51505837)
