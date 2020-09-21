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



<br/>
***
<br/>
参考资料：
[NSCharacterSet 详解](https://www.zybuluo.com/chinese-ppmt/note/609656)
