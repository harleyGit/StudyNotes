> <h2 id=""></h2>
- [**字符串操作**](#字符串操作)
	- [计算文本Rect](#计算文本Rect)
	- [NSCharacterSet字符串分割](#NSCharacterSet字符串分割)
- [**参考资料**](#参考资料)
	- [玩转 NSString](https://www.jianshu.com/p/d3f343b71cc2)


<br/>

***
<br/>

> <h1 id="字符串操作">字符串操作</h1>

<br/>
<br/>

> <h2 id="计算文本Rect">计算文本Rect</h2>

```
//计算一段文本的尺寸大小,在iOS7被弃用
- (CGSize)sizeWithFont:(UIFont *)font constrainedToSize:(CGSize)size lineBreakMode:(NSLineBreakMode)lineBreakMode NS_DEPRECATED_IOS(2_0, 7_0, "Use -boundingRectWithSize:options:attributes:context:") __TVOS_PROHIBITED;


//iOS7新出了一个新的方法替代上面的方法
/**
 返回文本绘制所占据的矩形空间。

@param size  宽高限制，用于计算文本绘制时占据的矩形块,用于在绘制文本时作为参考。
@param options  文本绘制时的附加选项
typedef NS_OPTIONS(NSInteger, NSStringDrawingOptions) {
        
        NSStringDrawingUsesLineFragmentOrigin = 1 << 0,
        // 整个文本将以每行组成的矩形为单位计算整个文本的尺寸
        // The specified origin is the line fragment origin, not the base line origin
        
        NSStringDrawingUsesFontLeading = 1 << 1,
        // 使用字体的行间距来计算文本占用的范围，即每一行的底部到下一行的底部的距离计算
        // Uses the font leading for calculating line heights
        
        NSStringDrawingUsesDeviceMetrics = 1 << 3,
        // 将文字以图像符号计算文本占用范围，而不是以字符计算。也即是以每一个字体所占用的空间来计算文本范围
        // Uses image glyph bounds instead of typographic bounds
        
        NSStringDrawingTruncatesLastVisibleLine
        // 当文本不能适合的放进指定的边界之内，则自动在最后一行添加省略符号。如果NSStringDrawingUsesLineFragmentOrigin没有设置，则该选项不生效
        // Truncates and adds the ellipsis character to the last visible line if the text doesn't fit into the bounds specified. Ignored if NSStringDrawingUsesLineFragmentOrigin is not also set.
        
    }


@param context  上下文。包括一些信息，例如如何调整字间距以及缩放。最终，该对象包含的信息将用于文本绘制。该参数可为 nil 。
@param attributes  文字属性
 @return 一个矩形，大小等于文本绘制完将占据的宽和高。
 */
- (CGRect)boundingRectWithSize:(CGSize)size options:(NSStringDrawingOptions)options attributes:(nullable NSDictionary<NSAttributedStringKey, id> *)attributes context:(nullable NSStringDrawingContext *)context NS_AVAILABLE(10_11, 7_0);
```


<br/>
<br/>

><h2 id='NSCharacterSet字符串分割'>NSCharacterSet字符串分割</h2> 

`//根据一个给定的字符串获取一个NSCharacterSet对象`

` +(NSCharacterSet *)characterSetWithCharactersInString:(NSString *)aString;`

```
NSCharacterSet *set = [NSCharacterSet characterSetWithCharactersInString:@"0123456789"];
NSString *str = @"7sjf78sf990s";
NSLog(@"set----%@",[str componentsSeparatedByCharactersInSet:set]);  
```

打印：
![正常过滤](https://upload-images.jianshu.io/upload_images/2959789-289fca2e6d87dac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

`//相反字符串限制 【具体见接下的例子】`

`@property (readonly, copy) NSCharacterSet *invertedSet;`

```
NSCharacterSet *invertedSet = [[NSCharacterSet characterSetWithCharactersInString:@"0123456789"] invertedSet];
NSLog(@"invertedSet----%@",[str componentsSeparatedByCharactersInSet:invertedSet]);
```

打印：
![invertedSet 属性反向过滤](https://upload-images.jianshu.io/upload_images/2959789-698ac3d395836325.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**[NSCharacterSet 详解](https://www.zybuluo.com/chinese-ppmt/note/609656)**




<br/>

***
<br/>


