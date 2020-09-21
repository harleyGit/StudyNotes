>#判断横竖屏

&emsp;  UIInterfaceOrientation是iOS8之后使用的设备方向属性，在之前可以使用statusBarOrientation来设置和获取设备朝向。

&emsp; iPhone/iPa的Home键盘是固定位置的，判断设备朝向可根据Home键位置来判断。

>Home键在正下方，正向竖屏

>Home键在正上方，反向竖屏

>Home键在正左方，横屏模式

>Home键在正右方，横屏模式

>faceUp

>faceDown


<br/>
**`UIInterfaceOrientation`**
```
UIInterfaceOrientationUnknown
设备的朝向不能确定。

UIInterfaceOrientationPortrait
该设备处于竖屏模式，设备保持直立，底部的Home键。

UIInterfaceOrientationPortraitUpsideDown
该设备处于竖屏模式，但上下颠倒，设备保持直立，顶部的Home键。

UIInterfaceOrientationLandscapeLeft
设备处于横向模式，设备保持直立，右侧Home键。

UIInterfaceOrientationLandscapeRight
该设备处于横向模式，设备保持直立，左侧Home键。
```


<br/>
***
<br/>
参考资料：
[设备朝向 US](https://blog.csdn.net/morris_/article/details/80070255)
