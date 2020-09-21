># iOS 原生系统架构
![系统架构层级](https://upload-images.jianshu.io/upload_images/2959789-93d89cf51262be20.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#`架构层级细分`
![原生架构细分](https://upload-images.jianshu.io/upload_images/2959789-08c98af35d821099.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原生架构细分接受，请看[这里](https://www.jianshu.com/p/80a27d111605)


```
NS函数归属于Cocoa Foundation框架；

CF函数归属于Core Foundation框架；

CG函数归属于CoreGraphics.frameworks框架；

CA函数归属于CoreAnimation.frameworks框架；

UI函数归属于UIKit框架。

UIKit继承自 Core Animation 和Core Graphics；
Core Animation：核心动画； 
Core Graphics：核心绘制 ；
UIkit：iOS基础视图框架.

```



<br/>
***
<br/>



># NSObject 继承关系图


![NSObject 关系图](https://upload-images.jianshu.io/upload_images/2959789-f6e8202c3b3d6d13.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![NSObject 继承关系图](https://upload-images.jianshu.io/upload_images/2959789-96979f65ce2ce6ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

># UIResponder 继承关系图

![UIResponder 继承关系图](https://upload-images.jianshu.io/upload_images/2959789-a64e5c6fc5717c30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>


>#手势继承关系图
![手势继承关系图](https://upload-images.jianshu.io/upload_images/2959789-40c947129ce0518e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

