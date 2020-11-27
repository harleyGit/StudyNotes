>#  Block 定义

`returnType    (^  BlockName)(arguments, arguments,……);`

Forexample:  `void (^ completeBlock)(int value, NSString *string);`


<br/>

***
<br/>

>#  Block 的使用

**有返回值的 Block 使用**

```
//申明
typedef int (^firstCompleteBlock) (int argumentOne, int argumentTwo);
//下面的类型，不推荐
//申明类型，省略形参：typedef int (^firstCompleteBlock) (int , int );


//定义为属性
@property(nonatomic, copy)firstCompleteBlock fcblock;

```

block 的实现：

![Block 的申明](https://upload-images.jianshu.io/upload_images/2959789-dd3b462208e3775b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

#`Blcok 作为属性,有返回值有参数`

```
typedef int (^firstCompleteBlock) (int argumentOne, int argumentTwo);
@property(nonatomic, copy)firstCompleteBlock fcblock;

self.fcblock = ^int(int argumentOne, int argumentTwo) {
        return  argumentOne * argumentTwo;
    };
    
int resulst = self.fcblock(10, 5);
NSLog(@"值为：%d", resulst);
```

输出：

`2019-05-16 16:41:31.280026+0800 Test[1282:68696] 值为：50`


<br/>

**`Block 作为方法参数,有返回值有参数`**

![Block 有返回值](https://upload-images.jianshu.io/upload_images/2959789-b6280f1ba468903b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
typedef int (^firstCompleteBlock) (int argumentOne, int argumentTwo);
@property(nonatomic, copy)firstCompleteBlock fcblock;


//方法实现
- (int) setOtherValue:(NSString *)value block:(firstCompleteBlock)compleBlock {
    NSLog(@"block %@ 返回值", value);
    int a = compleBlock(2, 8);
    NSLog(@"completeBlock 值为：%d", a);
    return a;
}



//调用方法
int b = [self setOtherValue:@"有" block:^int(int argumentOne, int argumentTwo) {
        return argumentOne + argumentTwo;
    }];
NSLog(@"block 回调有返回值： %d", b);
```

输出：

`2019-05-16 16:41:31.280309+0800 Test[1282:68696] block 有 返回值`

`2019-05-16 16:41:31.280472+0800 Test[1282:68696] completeBlock 值为：10`

`2019-05-16 16:41:31.280642+0800 Test[1282:68696] block 回调有返回值： 10`



<br/>

**`Blcok 作为方法的返回值,有返回值有参数`**

```
typedef int (^firstCompleteBlock) (int argumentOne, int argumentTwo);
@property(nonatomic, copy)firstCompleteBlock fcblock;


//实现
- (firstCompleteBlock) setOtherValue:(NSString *)value successBlock:(firstCompleteBlock)completeBlock {
    NSLog(@"block %@ 方法的返回值", value);
    int b = completeBlock(8, 32);
    NSLog(@"completeBlock 值实现后是：%d", b);
    
    
    return ^(int one, int two){
        int c= one * two;
        NSLog(@"-------->> %d", c);
        return c;
    };
}


//调用, 这里对block的调用有没有想到响应式编程的感觉
int d =  [self setOtherValue:@"是" successBlock:^int(int argumentOne, int argumentTwo) {
        return  argumentOne + argumentTwo;
    }](4, 5);

    
NSLog(@"block 的值是：%d", d);
```

输出：

`2019-05-17 12:14:25.221920+0800 Test[3712:60491] block 是 方法的返回值`

`2019-05-17 12:14:25.222350+0800 Test[3712:60491] completeBlock 值实现后是：40`

`2019-05-17 12:14:25.222582+0800 Test[3712:60491] -------->> 20`

`2019-05-17 12:14:25.222756+0800 Test[3712:60491] block 的值是：20`


<br/>

**`有返回值，无参数`**

```
typedef int (^BlockTest) (void);
@property(nonatomic, copy)BlockTest blockTest;


self.blockTest = ^int(){
        NSLog(@"blockTest 计算值为：%@", @"🈶值");
        return 10;
    };
int c = self.blockTest();
NSLog(@"blockTest 无参数，有返回值是：%d", c);
```

输出：

`2019-05-17 12:50:53.533606+0800 Test[4121:74646] blockTest 计算值为：🈶值`

`2019-05-17 12:50:56.547803+0800 Test[4121:74646] blockTest 无参数，有返回值是：10`


<br/>

**无返回值的 Block**

`Block 无返回值，但有参数`

```
typedef void (^BlockTest) (int argumentOne, int argumentTwo);
@property(nonatomic, copy)BlockTest blockTest;


self.blockTest = ^(int argumentOne, int argumentTwo) {
        NSLog(@"blockTest 计算值为：%d", argumentOne + argumentTwo);
    };
self.blockTest(20, 80);

```

输出：

`2019-05-17 12:36:54.927784+0800 Test[3983:69324] blockTest 计算值为：100`

<br/>

`无返回值，无参数`

```
typedef void (^BlockTest) (void);
@property(nonatomic, copy)BlockTest blockTest;



self.blockTest = ^void(){
        NSLog(@"blockTest 计算值为：%@", @"没🈶值");
    };
//简化形式
//self.blockTest = ^{
//        NSLog(@"blockTest 计算值为：%@", @"没🈶值");
//    };
self.blockTest();
```

输出：

`2019-05-17 12:39:42.019343+0800 Test[4025:70776] blockTest 计算值为：没🈶值`



<br/>

***
<br/>




>#  Block 简单实践

**Block属性传值**

```
@property(nonatomic, copy) void (^sencondCompleteBlcok)(NSString *value);

// block 属性内部传值
- (void) blockInObjcMethod {
    __weak typeof(self) weakSelf = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        __strong typeof(weakSelf) strongSelf = weakSelf;
        strongSelf.sencondCompleteBlcok(@"属性 block");
    });
}


//调用
[self blockInObjcMethod];
self.sencondCompleteBlcok = ^(NSString *value) {
      NSLog(@"回调传的值是： %@", value);
};

```

输出：
`2019-05-16 15:44:21.499789+0800 Test[1088:50961] 回调传的值是： 属性 block`

<br/>

**typedef 定义Block**

#`block 作为方法参数`

```
//申明定义block
typedef void (^firstCompleteBlock) (NSString *value);


//方法实现
+ (void) blockInClassMethod:(firstCompleteBlock)block{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2* NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"--->>> typedef block 打印");
        block(@"typedef Block");
    });
}


//方法调用
 [ViewController blockInClassMethod:^(NSString *value) {
        NSLog(@"类方法回调 %@", value);
    }];


```
输出：

`2019-05-16 15:44:19.101583+0800 Test[1088:50961] --->>> typedef block 打印`

`2019-05-16 15:44:19.101854+0800 Test[1088:50961] 类方法回调 typedef Block`


<br/>

**`作为属性`**

```
typedef void (^firstCompleteBlock) (NSString *value);

@property(nonatomic, copy)firstCompleteBlock fcblock;


//注意：先定义，才能使用，否则Crash
self.fcblock = ^(NSString *value) {
        NSLog(@"%@ 打印", value);
    };
self.fcblock(@"-------->> fcblock");
    
```

输出：
`2019-05-16 16:02:14.104871+0800 Test[1128:56500] -------->> fcblock 打印`





<br/>

***
<br/>



<br/>

***
<br/>




<br/>

***
<br/>


<br/>

***
<br/>


<br/>

***
<br/>











<br/>

***
<br/>

参考资料：

[Block看我就够了【干货】](http://www.cocoachina.com/ios/20190510/26931.html)

[Block导致循环引用的问题](https://www.jianshu.com/p/fc2f4d207d25)
