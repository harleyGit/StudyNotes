> <h2 id=''></h2>
- [**属性**](#属性)
	- [synthesize](#synthesize)
	- [@class](#@class)
	- [@selector](#@selector)
- [**类型**](#类型)
	- [Class和class用法](#Class和class用法)
	- [现成的回调函数](#现成的回调函数)
- [**标识**](#标识)
	- [UDID](#UDID)
	- [UUID](#UUID)
- [**网络请求**](#网络请求)
	- [字典数据NSNumber转化基本类型](#字典数据NSNumber转化基本类型)
- [**异常**](#异常)
	- [NSException](#NSException)




<br/>
<br/>

***
<br/>


> <h1 id='属性'>属性</h1>


<br/>


> <h2 id='synthesize'>synthesize</h2>

&emsp; **@synthesize**表示如果属性没有手动实现setter和getter方法，编译器会自动加上这两个方法.

<br/>

**使用如下:**

**HGTestOne.h**

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@protocol TestDelate <NSObject>

@optional
- (void)logInfo;

@end


@protocol TestProtocal <NSObject>

@property(nonatomic, weak)id <TestDelate> delegate;

@end


@interface HGTestOne : NSObject<TestProtocal>

- (void) testDelegateMethod;

@end

NS_ASSUME_NONNULL_END

```


<br/>

**HGTestOne.m**


```
#import "HGTestOne.h"

@implementation HGTestOne
@synthesize delegate;


- (void) testDelegateMethod {
    NSLog(@">>> 🍎 self.delegate:%@", self.delegate);
    
    if ([self.delegate respondsToSelector:@selector(logInfo)]) {
        [self.delegate logInfo];
    }
}

@end

```

<br/>

**TestRObjCController.m 中使用如下:**

```
@interface TestRObjCController ()<TestDelate>

@property(nonatomic, strong)UIButton *testBtn;

@end

@implementation TestRObjCController

- (UIButton *)testBtn {
    if (!_testBtn) {
        _testBtn = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 200, 120)];
        [_testBtn setTitle:@"测试" forState:UIControlStateNormal];
        [_testBtn addTarget:self action:@selector(testAction) forControlEvents:UIControlEventTouchUpInside];
        _testBtn.backgroundColor = UIColor.purpleColor;
    }
    
    return  _testBtn;
    
}



- (void)viewDidLoad {
    [super viewDidLoad];
    self.navigationItem.title = @"ReactiveObjC-3.1.1";

    
    [self.view addSubview:self.testBtn];
}


- (void) testAction {
    
//    [self testNetWork];
    
//    [self testRACommand];
    
    [self testPropertyDelegate];
    
}



//测试delegate属性
- (void) testPropertyDelegate {
    HGTestOne *to = [HGTestOne new];
    to.delegate =self;
    
    [to testDelegateMethod];
    
}

- (void) logInfo {
    NSLog(@"🍎 打印输出代理输出日志");
}

@end
```

打印输出如下:

```
2022-09-11 16:57:01.119678+0800 MLC[35505:1604465] >>> 🍎 self.delegate:<TestRObjCController: 0x7fd4cf7752a0>
2022-09-11 16:57:11.340421+0800 MLC[35505:1604465] 🍎 打印输出代理输出日志
```




<br/>
<br/>



> <h2 id='@class'>@class</h2>

[**@class**](https://www.jianshu.com/p/4f5081ef9eb3)


<br/>
<br/>


>## <h2 id='@selector'>@selector</h2>

[@selector](https://www.jianshu.com/p/96842c912a04)





<br/>
<br/>


> <h2 id=''></h2>







<br/>
<br/>

***
<br/>


> <h1 id='类型'>类型</h1>

<br/>

> <h2 id='Class和class用法'>Class和class用法</h2>

- Class类型和int、char、id一样也是一种类型，用于存储类对象的指针;
- class类型和id类型一样，本身继承于指针类型，故在声明class类型变量时时不需要用星号的。

<br/>

```
Class aclass = [someClass class]; 
```

&emsp; 代码第一个Class是数据类型，aclass是变量名，等号右边的class是类方法能够获得类的相关信息。

```
/**
 *测试Class和class区别用法
 */
- (void)testClassAnd_classMethod {
    
    NSDate *testDate = [NSDate new];
    
    //①
    Class class1=NSClassFromString(@"NSDate"); //这里的参数只能放类名,不能放实例!
    NSLog(@"🍎class1: %@",class1);
    
    //②
    id test1=[class1 new];           //这两句话效果是一样的
    NSLog(@"🍏test1: %@",test1);     //因为上面class1接收了NSDate类,现在class1就相当于NSDate这个类
    NSDate* test2=[NSDate new];
    NSLog(@"🍏test2: %@",test2);
    
    //③
    NSLog(@"🍐: %@",[testDate class]);   //对一个实例执行class方法(又叫发送class消息)可获取其所属类的名字
    
    
    //④
    NSLog(@"🍊: %@",NSDate.class);
    
    
    //⑤
    NSDate* date=[NSDate new];
    NSLog(@"🍋[date class]: %@",[date class]);                     //返回的是__NSDate ,导致下面的不相等,以后再研究为什么
    NSLog(@"🍋[date class]==NSDate.class: %d",[date class]==NSDate.class);     //不相等
    NSLog(@"🍋[NSDate class]==NSDate.class: %d",[NSDate class]==NSDate.class);  //相等
    
}
```

输出:

```
2022-09-12 21:15:22.418893+0800 MLC[39075:1739316] 🍎class1: NSDate
2022-09-12 21:15:22.420870+0800 MLC[39075:1739316] 🍏test1: 2022-09-12 13:15:22 +0000
2022-09-12 21:15:22.421026+0800 MLC[39075:1739316] 🍏test2: 2022-09-12 13:15:22 +0000
2022-09-12 21:15:22.421122+0800 MLC[39075:1739316] 🍐: __NSTaggedDate
2022-09-12 21:15:22.421171+0800 MLC[39075:1739316] 🍊: NSDate
2022-09-12 21:15:22.421225+0800 MLC[39075:1739316] 🍋[date class]: __NSTaggedDate
2022-09-12 21:15:22.421293+0800 MLC[39075:1739316] 🍋[date class]==NSDate.class: 0
2022-09-12 21:15:22.421346+0800 MLC[39075:1739316] 🍋[NSDate class]==NSDate.class: 1
```

<br/>

&emsp; 所以可以在项目中这样使用,结合运行时:

```
inline static NSDictionary<NSNumber *, Class<Model>> *modeDic(void) {
    static dispatch_once_t onceToken;
    static NSDictionary<NSNumber *, Class<Model>> *_dict;
    dispatch_once(&onceToken, ^{
        _dict = @{
            @(1) : [Model1 class],
            @(2) : [Model2 class],
            @(3) : [Model3 class],
        };
    });
    return _dict;
}
```






<br/>
<br/>


> <h2 id='现成的回调函数'>现成的回调函数</h2>

&emsp; 通常我们在写一个不带参数的块回调函数是这样写的,在 **.h** 头文件中
定义类型

```
typedef void (^leftBlockAction)();
```

在定义一个回调函数

```
-(void)leftButtonAction:(leftBlockAction)leftBlock;
```

<br/>

在 **.m** 文件中

```
-(void)leftButtonAction:(leftBlockAction)leftBlock{
leftBlock();
}

```

<br/>


现在有一个更方便的方法,就是在 **.h** 头文件定义属性方法

```
@property (nonatomic,copy) dispatch_block_t leftBlockAction;

````

<br/>

在 **.m**文件 调用的方法里调用

```
if (self.leftBlockAction) {
    self.leftBlockAction();
}
```

在另个模块里直接

```
MyAlertView *alert = [[MyAlertView alloc]init];
alert.leftBlockAction = ^() {

    NSLog(@"left button clicked");
};
```

这样就比自己定义闭包的方法简便了很多了


<br/>
<br/>




> <h2 id=''></h2>



<br/>
<br/>



> <h2 id=''></h2>



<br/>
<br/>




	
<br/>
<br/>

***
<br/>

> <h1 id='标识'>标识</h1>

>## <h2 id='UDID'>[UDID](https://www.jianshu.com/p/8bc3b1d5323f)</h2>

&emsp; UDID的全名为 Unique Device Identifier :设备唯一标识符。从名称上也可以看出，UDID这个东西是和设备有关的，而且是只和设备有关的，有点类似于MAC地址。需要把UDID这个东西添加到Provisoning Profile授权文件中，也就是把设备唯一标识符添加进去，以此来识别某一台设备。

&emsp; UDID是一个40位十六进制序列，我们可以使用iTunes和Xcode来获取这个值。





<br/>
<br/>


> <h2 id='UUID'>UUID</h2>

&emsp; 英文名称是：Universally Unique Identifier,翻译过来就是通用唯一标识符。是一个32位的十六进制序列，使用小横线来连接：8-4-4-4-12 。UUID在某一时空下是唯一的。比如在当前这一秒，全世界产生的UUID都是不一样的；当然同一台设备产生的UUID也是不一样的。我在很早之前的一篇博客中使用了一种现在看起来非常愚蠢的方式来获取当前的UUID，下面也有读者反映了这个情况，现在最简单获取UUID的代码如下：

```
for (int i = 0; i < 10; i++)
{
    NSString *uuid = [NSUUID UUID].UUIDString;
    NSLog(@"uuid 2 = %@",uuid);
}
```

&emsp; 通过运行程序可以发现，循环10次，每一次打印的值都是不一样的，当然循环的再多，这个值永远不会出现两个一样的值。所以从某种程序上来说，UUID跟你的设备没有什么关系了。




<br/>
<br/>



> <h2 id=''></h2>



<br/>
<br/>





<br/>
<br/>

***
<br/>


> <h1 id='网络请求'>网络请求</h1>


<br/>


> <h2 id='字典数据NSNumber转化基本类型'>字典数据NSNumber转化基本类型</h2>

&emsp; 因为NSArray和NSDictionary都无法存储基本数据类型，所以NSNumber就是用来将基本数据类型转化为对象的。

&emsp; 基本数据类型存入数组需要先将基本数据类型转化为对象，然后存入数组或者字典中。

&emsp; 基本数据类型转化为NSNumber对象

```
+ (NSNumber *)numberWithChar:(char)value;
+ (NSNumber *)numberWithUnsignedChar:(unsigned char)value;
+ (NSNumber *)numberWithShort:(short)value;
+ (NSNumber *)numberWithUnsignedShort:(unsigned short)value;
+ (NSNumber *)numberWithInt:(int)value;
+ (NSNumber *)numberWithUnsignedInt:(unsigned int)value;
+ (NSNumber *)numberWithLong:(long)value;
+ (NSNumber *)numberWithUnsignedLong:(unsigned long)value;
+ (NSNumber *)numberWithLongLong:(long long)value;
+ (NSNumber *)numberWithUnsignedLongLong:(unsigned long long)value;
+ (NSNumber *)numberWithFloat:(float)value;
+ (NSNumber *)numberWithDouble:(double)value;
+ (NSNumber *)numberWithBool:(BOOL)value;
+ (NSNumber *)numberWithInteger:(NSInteger)value;
+ (NSNumber *)numberWithUnsignedInteger:(NSUInteger)value;
```


<br/>

&emsp; 也可以使用@直接向基本数据类型封装为对象

&emsp; @10 代表是1个NSNumber对象，这个对象中包装的是整形的10。如果后面的数据是1个变量，那么这个变量就必须要使用小括弧括起来。

&emsp; NSNumber对象转化为基本数据类型

```
@property (readonly) char charValue;
@property (readonly) unsigned char unsignedCharValue;
@property (readonly) short shortValue;
@property (readonly) unsigned short unsignedShortValue;
@property (readonly) int intValue;
@property (readonly) unsigned int unsignedIntValue;
@property (readonly) long longValue;
@property (readonly) unsigned long unsignedLongValue;
@property (readonly) long long longLongValue;
@property (readonly) unsigned long long unsignedLongLongValue;
@property (readonly) float floatValue;
@property (readonly) double doubleValue;
@property (readonly) BOOL boolValue;
@property (readonly) NSInteger integerValue;
@property (readonly) NSUInteger unsignedIntegerValue;
```

<br/>
<br/>



> <h2 id=''></h2>






<br/>
<br/>

***
<br/>


> <h1 id='异常'>异常</h1>


>## <h2 id='NSException'>[NSException](https://www.cnblogs.com/lxlx1798/articles/10024109.html)</h2>



<br/>


> <h2 id=''></h2>



<br/>
<br/>



> <h2 id=''></h2>




<br/>
<br/>

***
<br/>


> <h1 id=''></h1>


<br/>


> <h2 id=''></h2>



<br/>
<br/>



> <h2 id=''></h2>