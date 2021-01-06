># 内存检测
- 静态检测
- 动态检测(Instrumetns / MLeakFinder )
- 析构方法打印

#`项目运行时静态检测配置`
![在Debug模式下静态检测](https://upload-images.jianshu.io/upload_images/2959789-27dd6162d07a3bb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>


># 使用 Autoreleasepool
场景：当程序有大量中间临时变量产生时，避免内存使用峰值过高，及时释放内存的场景(如：图片的存取)。
[Autoreleasepool_1](https://www.jianshu.com/p/e48f41d2144d)



<br/>
***
<br/>


参考资料：
[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/#32)
