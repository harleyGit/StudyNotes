

- UIView视图更新与重绘
	- 对应的事件序列
	- 通知 Controller 有数据变化
- 方法调用逻辑
- View Drawing Cycle


<br/>

***
<br/>

># UIView视图更新与重绘

```
public func setNeedsLayout()
public func layoutSubviews()
public func layoutIfNeeded()
 
public func setNeedsDisplay()
public func setNeedsDisplayInRect(rect: CGRect)
public func drawRect(rect: CGRect)
```

<br/>

>- **对应的事件序列如下：**<br/>
① 用户触摸屏幕<br/>
②硬件报告触摸事件给 UIKit 框架<br/>
③UIKit 框架将触摸事件打包成 UIEvent 对象，然后分发给合适的视图<br/>
④事件处理代码会对相应事件作出响应，代码可以是这样的：
-更改 frame、bounds、alpha 等属性
-调用 setNeedsLayout 方法以标记该视图（或者它的子视图）为需要进行布局更新
-调用 setNeedsDisplay 或者 setNeedsDisplayInRect: 方法以标记该视图（或者它的子视图）需要进行重画

> - 通知 Controller 有数据变化<br/>
⑤ 如果一个视图的几何结构改变了，UIKit 会更新它的子视图<br/>
⑥ 如果任何视图的任何部分被标记为需要重画，UIKit 会要求视图重画自身<br/>
⑦任何已经更新的视图会与应用余下的可视内容组合在一起，同时被发送到图形硬件去显示<br/>
⑧图形硬件将已解释内容转化到屏幕上<br/>

<br/>

***
<br/>



># 方法调用逻辑

在上面的过程中，我们可以接触到开头提到的方法，调用逻辑是这样的：

>setNeedsLayout 会给当前 UIView 立一个 flag，以表示后续应该调用 layoutSubviews 方法，以调整当前视图及其子视图的布局。<br/>
setNeedsDisplayInRect: 会给当前 UIView 立一个 flag，以表示后续应该调用 drawRect: 方法，以进行视图重绘。


<br/>

***
<br/>

># View Drawing Cycle

>个人对 View Drawing Cycle 的理解是这样的：<br/>

&emsp;  UIKit 需要处理非常多的事件，这些事件组合起来变成了一个非常复杂的事件序列，在这个序列中有些特定的点是 UIKit 专门提供给 UIView 来进行视图更改的。

<br/>

&emsp;  如上所述，在当前 run loop 结束之前，我们有机会做各种视图更改，并且这些更改会在下一个 run loop 体现出来，所以 View Drawing Cycle 就是一次次 run loop 中我们通过 UIKit 得到的 UIView 重布局、重绘机会所组成的循环。

**如何善用 View Drawing Cycle?**

&emsp;  一个很常见的例子是，一个 iPad App，横屏和竖屏时界面布局不一样，那么你可以监听设备旋转，在设备旋转时执行 setNeedsLayout 方法，然后在 layoutSubviews 里面通过判断接下来是横屏还是竖屏来进行不一样的布局设置。基本上你不可能只在这个方法里只进行了单个 UIView 的布局修改，而是多项修改，那么 App 会在下一个 View Drawing Cycle 到来时，把这些修改一起执行，这是最正常的情况。
