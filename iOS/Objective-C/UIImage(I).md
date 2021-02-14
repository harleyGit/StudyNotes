
- **创建并使用 ImageSource**
- **[**图片的摆正**](http://feihu.me/blog/2015/how-to-handle-image-orientation-on-iOS/)**
- [**UIImage简单操作**](https://www.jianshu.com/p/3baddf100b67)
- [**图片圆角处理**](https://www.jianshu.com/p/a00571de9bca)





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


># 图片的拉伸

![伸展图](https://upload-images.jianshu.io/upload_images/2959789-ad1c4e7c17e89125.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
func originPic() {
    let button:UIButton = UIButton.init(type: .custom)
    button.frame = CGRect.init(x: 100, y: 100, width: 100, height: 100)
    let img = UIImage(named: "safari")
    button.setBackgroundImage(img, for: .normal)
    button.setTitle("orgin", for: .normal)
    view.addSubview(button)
}

func strentchPic() {
    let button:UIButton = UIButton.init(type: .custom)
    button.frame = CGRect.init(x: 100, y: 220, width: 100, height: 100)
    let img = UIImage(named: "safari")?.resizableImage(withCapInsets: UIEdgeInsets(top: 10, left: 0, bottom: 10, right: 0),  resizingMode: .stretch)//指定被拉伸的区域，如果有渐变的图像，建议使用这个拉伸效果比较好
    button.setBackgroundImage(img, for: .normal)
    button.setTitle("stretch", for: .normal)

    view.addSubview(button)
}

func titlePic() {
    let button:UIButton = UIButton.init(type: .custom)
    button.frame = CGRect.init(x: 100, y: 340, width: 100, height: 100)
    var img = UIImage.init(named: "safari")
    img = img?.stretchableImage(withLeftCapWidth: Int((img?.size.width)!/2.0), topCapHeight: Int((img?.size.height)! / 2.0))
    button.setBackgroundImage(img, for: .normal)
    button.setTitle("stretchable", for: .normal)
    view.addSubview(button)
}

```
效果图：
![3 种形态](https://upload-images.jianshu.io/upload_images/2959789-e9292de49cf6f7b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

图片拉伸变形的方法、属性：
&emsp;  `withCapInsets：`通过设置 `UIEdgeInsets` 的 `left、right、top、bottom` 来分别指定这个矩形区域距离左侧、右侧、顶部、底部的距离。 

&emsp;  `resizingMode：`设置矩形区域的拉伸变形模式，具体有如下两种：

*  ` .stretch：`拉伸模式，通过拉伸 `UIEdgeInsets` 指定的矩形区域来填充图片。 
*  ` .tile：`平铺模式，通过重复显示 `UIEdgeInsets` 指定的矩形区域来填充图片。



```
func showPic() {
       let imageView = UIImageView(frame: CGRect(x:10, y:100, width:300, height:66))
       let image = UIImage(named: "safari")?
           .resizableImage(withCapInsets: UIEdgeInsets(top: 0, left: 15, bottom: 0, right: 15),
                           resizingMode: .stretch) //左右15像素的部分不变，中间部分来拉伸
       imageView.image = image
       view.addSubview(imageView)
   }

```

![图片水平拉伸](https://upload-images.jianshu.io/upload_images/2959789-ec9d6e2e2288fd62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

微信的聊天气泡就是可以这样做，我这边加载的是 safar 图标看不出这种效果，可以找一张气泡，效果就会更好。


