
># 半圆边框

```
+ (UIImageView *) drawInputViewWithFrame:(CGRect)frame {
    UIImageView *inputView = [[UIImageView alloc] initWithFrame:frame];
    CGFloat lineW = 1.0f;
    
    //获取上下文，两种情况(从UIView或者UIImageView)
    UIGraphicsBeginImageContext(inputView.frame.size);
    
    //获取图片上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    //线条样式：无端点
    CGContextSetLineCap(context, kCGLineCapButt);
    [[UIColor lightGrayColor] setStroke];
    //设置线条宽度
    CGContextSetLineWidth(context, lineW);
    
    //上部的线
    CGFloat startX = CGRectGetMinX(inputView.frame)+CGRectGetHeight(inputView.frame)/2;
    CGFloat endX = CGRectGetMaxX(inputView.frame)-CGRectGetHeight(inputView.frame)/2;
    CGContextMoveToPoint(context, startX, lineW);
    CGContextAddLineToPoint(context, endX, lineW);
    CGContextStrokePath(context);

    
    //绘制右圆弧
    [GRegisterController drawArcInContext:context superViewFrame:inputView.frame isLeft:NO lineWidth:lineW];
    CGContextStrokePath(context);

    
    //下部的线
    CGFloat lineY = CGRectGetHeight(inputView.frame)-lineW;
    CGContextMoveToPoint(context, startX, lineY);
    CGContextAddLineToPoint(context, endX, lineY);
    CGContextStrokePath(context);

    
    //绘制左圆弧
    [GRegisterController drawArcInContext:context superViewFrame:inputView.frame isLeft:YES lineWidth:lineW];
    CGContextStrokePath(context);

    //获取上下文中的image
    inputView.image = UIGraphicsGetImageFromCurrentImageContext();
    //结束上下文，显示imageView
    UIGraphicsEndImageContext();
    
    return inputView;
}

+ (void) drawArcInContext:(CGContextRef)context superViewFrame:(CGRect)frame isLeft:(BOOL)isLeft lineWidth:(CGFloat)lineW {

    //半径
    CGFloat radius = (CGRectGetHeight(frame)-2*lineW)/2;
    //圆弧中心点X坐标
    CGFloat centerX = 0.0f;
    //圆弧中心点Y坐标
    CGFloat centerY = radius+lineW;
    CGFloat startAngle = 0.0f;
    CGFloat endAngle = 0.0f;
    
    if (isLeft) {
        //圆弧中心点X坐标
        centerX = lineW+radius;
        startAngle = -M_PI_2;
        endAngle = M_PI_2;
    }else {
        centerX = CGRectGetMaxX(frame)-lineW-radius;
        startAngle = M_PI_2;
        endAngle = -M_PI_2;
    }
    
    CGContextAddArc(context, centerX, centerY, radius, startAngle, endAngle, 1);

}


- (UIImageView *)phoneNumberView {
    if (!_phoneNumberView) {
        _phoneNumberView = [GRegisterController drawInputViewWithFrame:CGRectMake(0, 100, 414, 46)];
    }
    return _phoneNumberView;
}

[self.view addSubview:self.phoneNumberView];

```
效果图：
![半圆边框](https://upload-images.jianshu.io/upload_images/2959789-e256dcbb5bc9e54a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

问题：`无法设置绘制区域内的填充色？？`


<br/>

***
<br/>

># 绘制背景色

```
override func draw(_ rect: CGRect) {

      let p = UIBezierPath(rect: rect)
      UIColor.red.setFill()
      p.fill()
}
```

效果：
![绘制背景色](https://upload-images.jianshu.io/upload_images/2959789-32d3e6c1f7c1f288.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>

># 绘制文本

```
override func draw(_ rect: CGRect) {

    let fontName = "HelveticaNeue-Bold"
    let helveticaBold = UIFont(name: fontName, size: 20.0)
    let string = "HUANG HARLEY 2020.01.18"
    string.draw(at: CGPoint(x: 40, y: 40), withAttributes: [NSAttributedString.Key.font : helveticaBold!,NSAttributedString.Key.foregroundColor:UIColor.red ])
}
```
![文本绘制](https://upload-images.jianshu.io/upload_images/2959789-0d98fecd157779fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>

***
<br/>

># 绘制图片

```
override func draw(_ rect: CGRect) {
    
    guard let picURL = URL.init(string: "https://www.apple.com.cn/v/safari/k/images/overview/safari_icon__ep64chrczuky_large_2x.jpg") else { return }
    do {
        let img = UIImage.init(data: try Data.init(contentsOf: picURL))
        //这里是绘制的起始点，会根据图片的大小绘制
        img?.draw(in: CGRect.init(x: 88, y: 64, width: 200, height: 200))//这里指定绘制的起始点和大小，可以自行查看区别
    } catch  {
        
    }
    
    
}

```


![safari 图标绘制](https://upload-images.jianshu.io/upload_images/2959789-4b5fc1267365d3e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>

***
<br/>

># 边框、阴影绘制

```
override func draw(_ rect: CGRect) {
    
    let currentContext = UIGraphicsGetCurrentContext()
    let path = CGMutablePath()
    
    ///绘制边界
    /**绘制矩形的边界*/
    let rectangle = CGRect.init(x: 100, y: 100, width: 80, height: 80)
    /**将矩形添加到路径中*/
    path.addRect(rectangle)
    /**设置线宽*/
    currentContext?.setLineWidth(5)
    /**获取当前上下文句柄*/
    currentContext?.addPath(path)
    
    ///填充颜色
    /**设置填充颜色*/
    UIColor.cyan.setFill()
    /**设置边框颜色*/
    UIColor.red.setStroke()
    
    ///绘制阴影
    let offset = CGSize.init(width: 10, height: 10)
    currentContext?.setShadow(offset: offset, blur: 20, color: UIColor.purple.cgColor)
    
    /**在上下文毛边并填充路径*/
    currentContext?.drawPath(using: .fillStroke)
}

```
![边框、阴影绘制](https://upload-images.jianshu.io/upload_images/2959789-5b28f362e2c0387f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






<br/>

***
<br/>


>#  线条绘制 

```
override func draw(_ rect: CGRect) {
    
     UIColor.red.set()
     /**获取当前的图形上下文*/
    guard let context = UIGraphicsGetCurrentContext() else { return }
     /**设置线宽*/
    (context ).setLineWidth(10)
     /**设置线的划线方式*/
    (context ).setLineCap(.round) //z
     /**设置线的起点*/
    (context ).move(to: CGPoint.init(x: 100, y: 90))
     /**设置线的终点*/
    (context ).addLine(to: CGPoint.init(x: 180, y: 170))
     /**设置两条线直接的链接的方式*/
    (context ).setLineJoin(.round)
    
     /**如果绘制连续颜色，则直接addLine就可以*/
    (context ).addLine(to: CGPoint.init(x: 100, y: 250))
   
     /**使用上下文当前的颜色绘制线条*/
    (context ).strokePath()
}
```
![线条绘制](https://upload-images.jianshu.io/upload_images/2959789-2650ad842e5ca352.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>

***
<br/>


># 绘制渐变

```
override func draw(_ rect: CGRect) {
    
    let currentContext = UIGraphicsGetCurrentContext()
    /**创建颜色空间,用来描述域值范围*/
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    let startColor = UIColor.red
    /**获取颜色组件*/
    let startColorComponents = startColor.cgColor.components

    let endColor = UIColor.purple
    let endColorComponents = endColor.cgColor.components
    ///RGB 颜色分量和 alpha 值写入
    let colorComponents = [
       startColorComponents?[0],
       startColorComponents?[1],
       startColorComponents?[2],
       startColorComponents?[3],
       endColorComponents?[0],
       endColorComponents?[1],
       endColorComponents?[2],
       endColorComponents?[3]
       ] as! [CGFloat]
    ///定义各种颜色的相对位置
    let colorIndices = [0.0,1.0] as [CGFloat]
    ///描述渐变信息
    let gradient = CGGradient.init(colorSpace: colorSpace, colorComponents: colorComponents, locations: colorIndices, count: 2)
    ///渐变将沿纵轴 (vertical axis) 方向绘制
    currentContext?.drawLinearGradient(gradient!, start: CGPoint.init(x: 100, y: 100), end: CGPoint.init(x: 100, y: 200), options: [])
    
}
```
![渐变绘制](https://upload-images.jianshu.io/upload_images/2959789-898f67260399173a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>


># 翻转绘制

```
override func draw(_ rect: CGRect) {
        
         let path = CGMutablePath()
         /**绘制矩形的边界*/
         let rectangle = CGRect.init(x: 200, y: 100, width: 80, height: 80)
         /**将矩形添加到路径中*/
         path.addRect(rectangle)
         /**使用缩放*/
//         let transform = CGAffineTransform.init(scaleX: 1.5, y: 1.5)
//         path.addRect(rectangle, transform: transform)
          /**使用旋转*/
        let transform = CGAffineTransform.init(rotationAngle: CGFloat((45.0 * Double.pi)/180))
         path.addRect(rectangle, transform: transform)
          
         /**获取当前上下文句柄*/
         let currentContext = UIGraphicsGetCurrentContext()
         /**保存上下文，便于后期恢复*/
         currentContext?.saveGState()
 //        /*像右移动10个点*/
         currentContext?.translateBy(x: 10, y: 0)
  
         currentContext?.addPath(path)
         /**设置填充颜色*/
         UIColor.red.setFill()
         /**设置边框颜色*/
         UIColor.yellow.setStroke()
         /**设置线宽*/
         currentContext?.setLineWidth(5)
         /**在上下文毛边并填充路径*/
         currentContext?.drawPath(using: .fillStroke)
         /**回复上下文状态*/
         currentContext?.restoreGState()
    }

```

![翻转绘制](https://upload-images.jianshu.io/upload_images/2959789-d4016835474acdff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>

**`参考资料：`**

[Core Graphics (长篇高能)](https://www.jianshu.com/p/491b50cb19cb)
