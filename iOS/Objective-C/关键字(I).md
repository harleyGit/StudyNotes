
- **DEPRECATED_MSG_ATTRIBUTE**
- **Const**
- **NS_ENUM 和 NS_OPTIONS** 
- **__bridge**
- **FOUNDATION_EXTERN**
- **FOUNDATION_EXTERN_INLINE**
- **VA_ARGS**
- **[pragma 处理警告](https://www.jianshu.com/p/4720fc9e991a)**
- [**常见的宏**](https://www.jianshu.com/p/9f7a37989b79)


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


># pragma



 <br/>

***
<br/>


># NS_ENUM 和 NS_OPTIONS

```

//整型枚举 :只能单个使用，在C++ 不可以转换
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {
    UIViewAnimationTransitionNone,
    UIViewAnimationTransitionFlipFromLeft,
    UIViewAnimationTransitionFlipFromRight,
    UIViewAnimationTransitionCurlUp,
    UIViewAnimationTransitionCurlDown,
};


// 位移枚举：使用这种可以多个连接使用 | 进行按位后连接。c++可以转换位 NSUInteger
//[self.view setAutoresizingMask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleTopMargin];
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};

```




<br/>

***
<br/>

># __bridge

`(__bridge id) 是桥接，把非OC转化为OC使用的。`


<br/>

***
<br/>

># FOUNDATION_EXTERN

```
#if defined(__cplusplus)
#define FOUNDATION_EXTERN extern "C"
#else
#define FOUNDATION_EXTERN extern
#endif
```

表示 extern 全局变量，此时并没有分配内存，需要在.m文件中实现，此时为了支持C和C++混编（__cplusplus 是C++编译器内部定义的宏，在C++中，需要加
extern"C" 或包含在 extern "C" 块中），注意，此时外界是可以修改这个值，详细 extern 用法可自行查询相关资料，本文不详谈。
用法如下：

```
FOUNDATION_EXTERN NSString *name;// h文件
const NSString *name = @"gitKong";// m文件
```




<br/>

***
<br/>

># FOUNDATION_EXTERN_INLINE

表示全局的内联函数



<br/>

***
<br/>

># __VA_ARGS__

- **描述**

C99 编译器标准允许定义可变参数宏(variadic macros)，这样就使用拥有可以变化的参数表的宏。

```
#define FYFLog(format, ...) NSLog(format, __VA_ARGS__)
```

缺省号代表一个可以变化的参数表。使用保留名 '__VA_ARGS__' 把参数传递给宏。当宏的调用展开时，实际的参数就传递给 NSLog() 了。



<br/>

***
<br/>

参考资料：
[^断言NSAssert][断言NSAssert](https://www.jianshu.com/p/d7498657d550)

[^断言(NSAssert)的使用][断言(NSAssert)的使用](https://www.jianshu.com/p/6e444981ab45)

[CADisplayLink做逐帧动画](https://www.jianshu.com/p/0eeb21244caa)
