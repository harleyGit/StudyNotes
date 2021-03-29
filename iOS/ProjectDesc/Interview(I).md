- **[OC基础](#oc基础)** [](https://my.oschina.net/antsky/blog/1475173)
	- [UILabel多适应](#uilabel多适应)


<br/>

***
<br/>

># OC基础

<br/>

> UILabel多适应?

Label自适应宽度

```
+ (float)widthAuto:(NSString *)content fontSize:(CGFloat)fontSize contentheight:(CGFloat)height{

		// 列寬
		//    CGFloat contentHeight = height;
		
		// 用何種字體進行顯示
		UIFont *font = [UIFont systemFontOfSize:fontSize];
		
		// 計算出顯示完內容需要的最小尺寸
		CGSize size = CGSizeMake(100, 100);
		
		// 計算出顯示完內容需要的最小尺寸
		NSDictionary * tdic = [NSDictionary dictionaryWithObjectsAndKeys:font,NSFontAttributeName,nil];
		
		//ios7方法，获取文本需要的size，限制宽度
		CGSize  actualsize =[content boundingRectWithSize:size options:NSStringDrawingUsesLineFragmentOrigin |NSStringDrawingUsesFontLeading attributes:tdic context:nil].size;
		
		return actualsize.width;

}

```

