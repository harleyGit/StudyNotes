
创建并使用 ImageSource




<br/>

***
<br/>





># 创建并使用 ImageSource

>&emsp; imageSource抽象了访问数据的任务，它解决了我们自己从内存原始缓冲区中去管理数据的需求。一个imageSource可以包含一个甚至多个图像，缩略图，每一个图像的属性和图像文件。如果我们在工作中用到了图像数据并且程序是运行在OS X v10.4及其以后的，imageSource将会是把图像数据转移到程序中的首选方式。在创建了CGImageSource对象之后，便可以获取图像、缩略图、图像属性和一些其他的图像信息。

>&emsp; 当我们使用imageSource来创建一个图像时，必须要指定一个索引，而且还可以提供一个用于指定是否创建缩略图或者是否运行缓存的属性字典（键值对）。CGImageSource Reference和CGImageProperties Reference列出了所有的key和这些key对应支持的数据类型。

>&emsp; 而且我们需要提供一些索引值（index），因为有些图像文件格式允许多个图像同时存在于一个相同的源文件中。**对于一个图像源文件只包含一个图像时，索引值传入0。如果想要知道一个图像源文件包含了多少个图像，我们调用函数CGImageSourceGetCount来获取。**

<br/>

***
<br/>


参考资料：

[iOS绘图 - UIImage的一些简单操作](https://www.jianshu.com/p/3baddf100b67)

[iOS 图片圆角处理及各种“角”的解决方案](https://www.jianshu.com/p/a00571de9bca)
