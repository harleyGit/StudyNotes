># 以下是三种是获取当前上下文的方式，而绘图的API有两种（***Core Graphics和OpenGL ES***）。
<br/>
<br/>



># 上下文的获取：***UIGraphicsBeginImageContextWithOptions 和 UIGraphicsEndImageContext***

`
这两个方法是成对出现的，函数块内就是在当前上下文中进行绘制的。


```Swift
UIGraphicsBeginImageContextWithOptions(CGMake(100,100),NO,0);

//这里执行绘图操作

//当绘制完成后，需要从当前上下文中获取绘制的内容，而这个内容是一个UIImage对象
UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();//关闭图形上下文
```

`UIGraphicsBeginImageContextWithOptions(CGSize size, BOOL opaque, CGFloat scale)`

&emsp;  ***用法：***用来处理图片的图形上下文,函数不仅仅是创建了一个适用于图形操作的上下文，并且该上下文也属于当前上下文。

>CGSize size:  表示所要创建的图片的尺寸;<br/>
BOOL opaque: 指定所生成图片的背景是否为不透明，如上我们使用YES而不是NO，则我们得到的图片背景将会是黑色;<br/>
CGFloat scale: 指定生成图片的缩放因子，这个缩放因子与UIImage的scale属性所指的含义是一致的。传入0则表示让图片的缩放因子根据屏幕的分辨率而变化。


<br/>

***

># 上下文的获取：***UIGraphicsPushContext和UIGraphicsPopContext***
&emsp;  用法：成对出现的，使用UIGraphicsPushContext来转换已有上下文为当前上下文。

```
//context为已拥有的上下文，必须传入一个CGContextRef的上下文，使用UIGraphicsPushContext把传入的上下文转换为当前View的上下文
UIKIT_EXTERN void UIGraphicsPushContext(CGContextRef context);

//绘图操作

//绘制完成后，需要把转换的上下文恢复，UIGraphicsPopContext函数恢复上下文环境
UIKIT_EXTERN void UIGraphicsPopContext(void);

```


<br/>

***
<br/>


># 上下文的获取：***drawRect:***

&emsp;  `当drawRect:方法被调用时，Cocoa就会为你创建一个图形上下文，此时你对图形上下文的所有绘图操作都是在当前View的上下文中 。`

```
//每个继承自View的类都包含该方法，可以进行继承该方法来获取当前视图上下文
- (void) drawRect: (CGRect) rect {
       
            //直接进行绘图操作
           
}
```


<br/>

***


>#***CGContextSaveGState和CGContextRestoreGState***上下文状态的压栈和出栈

```Swift
//将传入用于绘制的上下文状态压栈栈顶，并保存传入时的上下文所有状态。
void CGContextSaveGState(CGContextRef cg_nullable c)


//将当前的上下文状态恢复到传入的上下文保存时的状态
void CGContextRestoreGState(CGContextRef cg_nullable c)
```

🌰：假如当前上下文默认绘制是黑色的画笔宽度是2，而我们需要先绘制一个绿色且宽度为10的线，然后需要绘制一个绿色且宽度为5的线。这其中就涉及到来修改上下文绘制的画笔颜色和画笔宽度属性，以及恢复上下文属性。

```
- (void) drawView {

    CGRect frame = CGRectMake(80, 80, 300, 188);
    UIImageView *imageView = [[UIImageView alloc] initWithFrame:frame];
    imageView.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        //设置当前上下文中绘制的区域
        UIGraphicsBeginImageContextWithOptions(frame.size, NO, 0);
        //获取当前上下文
        CGContextRef context = UIGraphicsGetCurrentContext();
        
        //修改当前上下文的画笔颜色和宽度
        CGContextSetLineWidth(context, 2);
        [[UIColor blackColor] setStroke];
        
        //保存当前上下文的状态
        CGContextSaveGState(context);
        
        //绘制一个圆
        CGContextAddEllipseInRect(context, CGRectMake(0, 0, frame.size.width, frame.size.height));
        CGContextSetLineWidth(context, 10);
        [[UIColor greenColor] setStroke];
        
        //开始绘制
        CGContextStrokePath(context);
        
        
        CGContextAddEllipseInRect(context, CGRectMake(10, 10, frame.size.width - 20, frame.size.height - 20));
        CGContextSetLineWidth(context, 5);
        [[UIColor orangeColor] setStroke];
        
        CGContextStrokePath(context);
        
        //恢复当前上下文的状态
        CGContextRestoreGState(context);
        CGContextAddEllipseInRect(context, CGRectMake(20, 20, frame.size.width - 40, frame.size.height - 40));
        
        //不设置画笔颜色和宽度，因为原本的上下文有颜色和宽度状态
        CGContextStrokePath(context);
        
        
        //从当前上下文中获取绘制的内容
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        
        //关闭上下文 
        UIGraphicsEndImageContext();
        
        dispatch_async(dispatch_get_main_queue(), ^{
            imageView.image = image;
        });
    });
    
    [self.view addSubview:imageView];
}
```
![绘制效果图](https://upload-images.jianshu.io/upload_images/2959789-5aba6306788302e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***

># ***图片绘制***

>## ***CGImageCreateWithImageInRect***

`用法：根据指定范围截图图片区域，获得一个新的图片，获得的图片是CGImageRef类型的CGImageRef newImageRef = CGImageCreateWithImageInRect(imageRef,size)`
```
CGImageCreateWithImageInRect(CGImageRef cg_nullable image, CGRect rect)

// image:  需要截取的原图CGImageRef类型
// rect :  截取的区域

/*
注意:该接口ARC无效，获得的图片需要手动释放.
CGImageRelease(newImageRef)
*/

```

<br/>

>##***CGContextDrawImage***

`用法：在当前的上下文中把图片内容绘制到指定区域`
```
CGContextDrawImage(CGContextRef cg_nullable c, CGRect rect,
    CGImageRef cg_nullable image)

// c:      内容上下文
// rect :  绘制的区域
// image: 需要绘制上去的图片，CGImageRef类型

```
<br/>

>## ***drawAtPoint***

`用法：该方法是UIImage类的对象方法，用于把当前图片按照指定的抛锚点在当前上下文中开始绘制`

```
- (void)drawAtPoint:(CGPoint)point;

//point:  图片开始绘制的起始点

```

<br/>

🌰：

```
- (void) drawView {
    UIImageView *view = [[UIImageView alloc] initWithFrame:CGRectMake(40, 80, 180, 120)];
    view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    UIImage *image = [UIImage imageNamed:@"icon.png"];
    UIGraphicsBeginImageContextWithOptions(CGSizeMake(40, 40), NO, 0);
    [image drawAtPoint:CGPointMake(0,0)];
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    view.image = newImage;
    [self.view addSubview:view];
    UIGraphicsEndImageContext();
}
```

![起始点绘制效果图](https://upload-images.jianshu.io/upload_images/2959789-b606e752bbd1ea5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

>## ***drawInRect***

`用法：该方法是UIImage类的对象方法，用于把当前图片按照指定Rect在当前上下文中绘制，可以缩放可以位移。用法与drawAtPoint一样`

```
- (void)drawInRect:(CGRect)rect;

// rect:  指定的绘制Rect

```

<br/>


>##***imageWithCGImage:  scale:  orientation:***

`用法：该方法是UIImage的类方法，用于把CGImageRef类型的图片 按照scale与方向转换成对应的UIImage对象。`


```
+ (UIImage *)imageWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation;

// cgImage:      需要转换的CGImageRef类型的图片
// scale:        图片的比例
// orientation:  图片的方向

```

***`UIImageOrientationUp, //默认方向  `***
![默认方向](https://upload-images.jianshu.io/upload_images/2959789-8de922f66dbc81a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

***`UIImageOrientationDown,// 180 度旋转，向下 `***
![180 度旋转，向下](https://upload-images.jianshu.io/upload_images/2959789-dc988e2902291e8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>


***`UIImageOrientationLeft,//向左90度旋转 `***
![向左90度旋转](https://upload-images.jianshu.io/upload_images/2959789-1f0e8c88ffd1442b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>


***`UIImageOrientationRight,//向右90 度旋转 `***
![向右90 度旋转](https://upload-images.jianshu.io/upload_images/2959789-fc0f67ae57400758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br/>



***` UIImageOrientationUpMirrored, //按照图片镜像样子水平旋转
`***
![水平旋转](https://upload-images.jianshu.io/upload_images/2959789-415464a1906124c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br/>

***`UIImageOrientationDownMirrored, // 水平翻转 `***
![水平翻转](https://upload-images.jianshu.io/upload_images/2959789-579068ae0e85d5b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>


***` UIImageOrientationLeftMirrored, //向左垂直翻转`***
![向左垂直翻转](https://upload-images.jianshu.io/upload_images/2959789-3634d2f928fad059.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>


***` UIImageOrientationRightMirrored, //向右垂直翻转`***
![向右垂直翻转](https://upload-images.jianshu.io/upload_images/2959789-de358d125ad97db0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>




<br/>






