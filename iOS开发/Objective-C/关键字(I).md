
- **DEPRECATED_MSG_ATTRIBUTE**
- Const


<br/>

***
<br/>

># DEPRECATED_MSG_ATTRIBUTE

方法版本迭代的时候使用，抛弃旧的方法，提示使用新的方法

使用：

```

- (void)storeImage2:(UIImage *_Nullable)image forKey:(NSString *)key DEPRECATED_MSG_ATTRIBUTE("please use storeImage: imageData: forKey: completion:");
```

![z22](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z22.png)



<br/>

***
<br/>


># Const

const 前缀声明指定类型的常量，如下所示：  

`const type variable = value;`


**const 修饰位置的变化**

 ``` //变量a被const修饰，就成为了只读，不能被修改赋值了 int const a = 10; //等价于 const int a = 10; // Wrong a = 20;//错误代码

int const *p // *p只读 ;p变量

int * const p // *p变量 ; p只读

const int * const p //p和*p都只读

int const * const p //p和*p都只读
 ```


**const** 常用用法 

``` 
//定义一个全局只读变量 NSString * const Kname = @"appkey";

//static修饰后此全局变量只能本文件访问 static NSString *const Key = @"hddjj”; 
```

参考资料：
[^关键字]:[关键字](http://www.cocoachina.com/ios/20171226/21653.html)







<br/>

***
<br/>

参考资料：
[^断言NSAssert][断言NSAssert](https://www.jianshu.com/p/d7498657d550)

[^断言(NSAssert)的使用][断言(NSAssert)的使用](https://www.jianshu.com/p/6e444981ab45)
