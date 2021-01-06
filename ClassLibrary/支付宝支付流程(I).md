![阿里SDK拷入报错](https://upload-images.jianshu.io/upload_images/2959789-425451f4076cb987.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![阿里SDK加入依赖库](https://upload-images.jianshu.io/upload_images/2959789-09b6dd62ca06f8ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;&emsp;  这是许多开发者在使用阿里SDK遇到的坑之一，当我们按照阿里的指示将它所说的SDK宝导入工程中，并按照指示加入依赖库，如：【阿里SDK加入依赖库】所示。等加完依赖库后，我们发现报错继续存在，没有解决。报错仍然如图：【[阿里SDK拷入报错】所示，好像陷入了死循环。
&emsp;&emsp;  How do you do ?
&emsp;&emsp;  这时查百度，发现这样做是可以的。将原阿里SDK的第三方库删除扔掉垃圾篓，将阿里的Demo拷入到项目中，再运行发现完美运行。问题解决！！



<br/>
***
<br/>
参考资料：[支付宝支付集成](https://www.cnblogs.com/demodashi/p/8486502.html)
[支付宝支付集成实现](https://www.jianshu.com/p/a62a4793e84f)
[蚂蚁金服iOS支付集成](https://docs.open.alipay.com/204/105295/)
