
> <h2 id=''></h2>
- [**API使用**](#API使用)
	- [浅拷贝和深拷贝](#浅拷贝和深拷贝)
	- [NSComparisonResult枚举](#NSComparisonResult枚举)
	- [sortedArrayUsingComparator](#sortedArrayUsingComparator)
	- [混合排序](#混合排序)
	- [sortUsingComparator](#sortUsingComparator)
- [**参考资料**](#参考资料)
	- [中文排序](https://www.jianshu.com/p/69ff8ed1857f)
	- [中文拼音](https://blog.csdn.net/jabezlee/article/details/9670965)



<br/>

***
<br/>

> <h1 id='API使用'>API使用</h1>

<br/>


> <h2 id='浅拷贝和深拷贝'>浅拷贝和深拷贝</h2>

**测试一:**

```
// strong  类型 NSArray
@property (nonatomic, strong) NSArray  *strongArray;
// copy    类型 NSArray
@property (nonatomic, copy)   NSArray  *copyedArray;

- (void) testNSStringPropertyCopyAndStrong {
    NSArray *tmpArray = [NSArray arrayWithObjects:@"1",@"2", nil];

    self.strongArray = tmpArray;
    self.copyedArray = tmpArray;

    NSLog(@"tmpArray: %@, %p", tmpArray, tmpArray);
    NSLog(@"self.strongArray: %@, %p", self.strongArray, self.strongArray);
    NSLog(@"self.copyedArray: %@, %p", self.copyedArray, self.copyedArray);
}

```

打印:

```
2022-12-08 20:12:43.344174+0800 MLC[8754:319784] tmpArray: (
    1,
    2
), 0x600000194840

2022-12-08 20:12:43.344341+0800 MLC[8754:319784] self.strongArray: (
    1,
    2
), 0x600000194840

2022-12-08 20:12:43.344446+0800 MLC[8754:319784] self.copyedArray: (
    1,
    2
), 0x600000194840
```

&emsp; 针对NSArray使用copy和strong的效果是一样的,都是浅拷贝的.它们的内存地址都是一样的.


<br/>
<br/>


**测试二**


```

// strong  类型 NSArray
@property (nonatomic, strong) NSArray  *strongArray;
// copy    类型 NSArray
@property (nonatomic, copy)   NSArray  *copyedArray;


- (void) testNSStringPropertyCopyAndStrong {

		NSMutableArray *tmpArray = [NSMutableArray arrayWithObjects:@"1",@"2", nil];

    self.strongArray = tmpArray;
    self.copyedArray = tmpArray;

    NSLog(@"tmpArray: %@, %p", tmpArray, tmpArray);
    NSLog(@"self.strongArray: %@, %p", self.strongArray, self.strongArray);
    NSLog(@"self.copyedArray: %@, %p", self.copyedArray, self.copyedArray);

    [tmpArray addObject:@"3"];
    NSLog(@"tmpArray: %@, %p", tmpArray, tmpArray);
    NSLog(@"self.strongArray: %@, %p", self.strongArray, self.strongArray);
    NSLog(@"self.copyedArray: %@, %p", self.copyedArray, self.copyedArray);
}
```

打印:

```
tmpArray: (
    1,
    2
), 0x600001ec26a0

self.strongArray: (
    1,
    2
), 0x600001ec26a0

self.copyedArray: (
    1,
    2
), 0x6000010d1980


tmpArray: (
    1,
    2,
    3
), 0x600001ec26a0

self.strongArray: (
    1,
    2,
    3
), 0x600001ec26a0

self.copyedArray: (
    1,
    2
), 0x6000010d1980

```

&emsp; 当使用NSMutable时,strong是引用.但是copy是进行了深复制,从其打印的日志可以看出.

<br/>

&emsp; 如果确定来源是NSArray类型，这种情况下用strong类型。因为copy修饰的NSArray类型在进行set操作时，底层进行了这样的判断if ([str isMemberOfClass: [NSArray class]])，如果**来源是可变的，就进行一次深拷贝**，如果是不可变的就和strong修饰一样，进行一次浅拷贝，


&emsp; 如果来源数组是NSMutableArray，使用copy修饰的数组会执行一次深拷贝，指向新的内存地址。但是目标数组里面内容的内存地址和来源数组里面内容的内存地址还是一致的，如果改变了来源数组里面内容，目标数组也会跟着改变。




<br/>
<br/>

**测试三**

**PersonModel.h**

```
@interface PersonModel : NSObject
// 姓名
@property (nonatomic, copy) NSString *name;

// 年龄
@property (nonatomic, assign) NSInteger age;

// 地址
@property (nonatomic, copy) NSString *address;
@end



- (void) testNSStringPropertyCopyAndStrong {
		PersonModel *tmpFirstPerModel = [[PersonModel alloc] init];
    tmpFirstPerModel.name = @"Jack";
    tmpFirstPerModel.age = 16;
    tmpFirstPerModel.address = @"深圳市南山区白石洲下白石一坊9巷12号";

    NSLog(@"tmpFirstPerModel: %p", tmpFirstPerModel);
    NSMutableArray *tmpArray = [NSMutableArray arrayWithObjects:tmpFirstPerModel,@"2", nil];

    self.strongArray = tmpArray;
    self.copyedArray = tmpArray;


    NSLog(@"tmpArray: %p, PersonModel: %p", tmpArray, [tmpArray objectAtIndex:0]);
    NSLog(@"self.strongArray: %p, PersonModel: %p", self.strongArray, [self.strongArray objectAtIndex:0]);
    NSLog(@"self.copyedArray: %p, PersonModel: %p", self.copyedArray, [self.copyedArray objectAtIndex:0]);
}
```

打印:

```
tmpFirstPerModel: 0x600001972de0

tmpArray: 0x60000170df50, PersonModel: 0x600001972de0

self.strongArray: 0x60000170df50, PersonModel: 0x600001972de0

self.copyedArray: 0x600001972d80, PersonModel: 0x600001972de0
```

&emsp; 使用copy修饰的self.copyedArray数组进行了深拷贝，指向了新的内存地址，但是数组里面的PersonModel的指针还是和之前初始化的tmpFirstPerModel是一致的，如果你在来源数组中修改了，self.copyedArray里面的内容也会更改.

<br/>
<br/>


**解决方案**

&emsp; 如果要避免这种情况，就必须对模型实现NSCopying和NSMutableCopying(如果支持可变类型)协议,然后实现

`-(id)copyWithZone:(NSZone *)zone `

函数和

`-(id)mutableCopyWithZone:(NSZone *)zone` ,

接着遍历源数组进行copy添加到新数组中或者通过

`-(instancetype)initWithArray:(NSArray<ObjectType> *)array copyItems:(BOOL)flag`

函数，进行初始化拷贝。

**PersonModel.h**

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface PersonModel : NSObject<NSCopying, NSMutableCopying>
// 姓名
@property (nonatomic, copy) NSString *name;

// 年龄
@property (nonatomic, assign) NSInteger age;

// 地址
@property (nonatomic, copy) NSString *address;
@end

NS_ASSUME_NONNULL_END

```

<br/>

**PersonModel.m文件**

```
#import "PersonModel.h"

@implementation PersonModel

#pragma mark ---- NSCopying

- (id)copyWithZone:(NSZone *)zone {
    //一定要通过[self class]方法返回的对象调用allocWithZone:方法。因为指针可能实际指向的是PersonModel的子类。这种情况下，通过调用[self class]，就可以返回正确的类的类型对象
    PersonModel *configModel = [[[self class] allocWithZone:zone] init];
    configModel.name = [self.name copyWithZone:zone];
    configModel.age = self.age;
    configModel.address = [self.address copyWithZone:zone];
    return configModel;
}

#pragma mark ---- NSMutableCopying

- (id)mutableCopyWithZone:(NSZone *)zone {
    return [self copyWithZone:zone];
}

@end

```


<br/>

调用:

```
- (void) testNSStringPropertyCopyAndStrong {
	
	PersonModel *tmpFirstPerModel = [[PersonModel alloc] init];
    tmpFirstPerModel.name = @"Jack";
    tmpFirstPerModel.age = 16;
    tmpFirstPerModel.address = @"深圳市南山区白石洲下白石一坊9巷12号";

    NSLog(@"tmpFirstPerModel: %p", tmpFirstPerModel);
    NSMutableArray *tmpArray = [NSMutableArray arrayWithObjects:tmpFirstPerModel,@"2", nil];

    self.strongArray = tmpArray;
		//self.copyedArray = tmpArray;

    
    //下面2种数组深拷贝方法
    // 遍历 拷贝 添加 方法
	//NSMutableArray *tmpMutableArray = [NSMutableArray array];
	//for (PersonModel *tmpPersonModel in tmpArray) {
			//[tmpMutableArray addObject:[tmpPersonModel copy]];
	//}
	//self.copyedArray = tmpMutableArray;

    // 初始化 方法 进行 内部 拷贝
    self.copyedArray = [[NSArray alloc] initWithArray:tmpArray copyItems:YES];

    tmpFirstPerModel.address = @"厦门市翔安区新店镇朝新路3号";

    NSLog(@"tmpArray: %p, PersonModel: %p", tmpArray, [tmpArray objectAtIndex:0]);
    NSLog(@"self.strongArray: %p, PersonModel: %p, address: %@", self.strongArray, [self.strongArray objectAtIndex:0], ((PersonModel *)[self.strongArray objectAtIndex:0]).address);
    NSLog(@"self.copyedArray: %p, PersonModel: %p, address: %@", self.copyedArray, [self.copyedArray objectAtIndex:0], ((PersonModel *)[self.copyedArray objectAtIndex:0]).address);
    
    
}


```

打印:

```
tmpFirstPerModel: 0x600000948560

MLC[9799:359328] tmpArray: 0x60000075a0d0, PersonModel: 0x600000948560

MLC[9799:359328] self.strongArray: 0x60000075a0d0, PersonModel: 0x600000948560, address: 厦门市翔安区新店镇朝新路3号

MLC[9799:359328] self.copyedArray: 0x6000009b1240, PersonModel: 0x6000009b0c20, address: 深圳市南山区白石洲下白石一坊9巷12号
```





<br/>
<br/>

> <h2 id='NSComparisonResult枚举'>NSComparisonResult枚举</h2>



```
typedef NS_CLOSED_ENUM(NSInteger, NSComparisonResult) {
    NSOrderedAscending = -1L,   // 表示两个比较的对象，前者小于后者
    NSOrderedSame,              //表示比较的对象相等
    NSOrderedDescending         //表示两个比较的对象，前者大于后者
};
```



<br/>
<br/>


> <h2 id='sortedArrayUsingComparator'>sortedArrayUsingComparator</h2>


```

//按递增的方式排序
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr;

```

<br/>
<br/>


> <h2 id='NSOrderedAscending 数字排序'>NSOrderedAscending 数字排序</h2>



```
NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
NSArray *array1 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedAscending;
    }];
NSLog(@"array1: %@", array1);
```

**`执行结果`**

```
2018-11-08 10:01:41.667489+0800 Test[1737:55438] array1: (
    3,
    5,
    8,
    6,
    1,
    0
)
//没有发生交换
```

`结论：`数字排序没有发生交换。

<br/>
<br/>


> <h2 id='NSOrderedSame 数字排序'>NSOrderedSame 数字排序</h2>

```
    NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
 NSArray *array2 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedSame;
    }];
    NSLog(@"array2: %@", array2);
```

**`执行结果`**

```
2018-11-08 10:02:17.067143+0800 Test[1737:55438] array2: (
    3,
    5,
    8,
    6,
    1,
    0
)
```
`排序：`数字排序没有发生交换。



<br/>
<br/>

> <h2 id='NSOrderedDescending  数字排序'>NSOrderedDescending  数字排序</h2>


```
 NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
NSArray *array3 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedDescending;
    }];
NSLog(@"array3: %@", array3);


#打印结果：
2018-11-08 10:02:44.061283+0800 Test[1737:55438] array3: (
    0,
    1,
    6,
    8,
    5,
    3
)
```
结论：`次序发生颠倒`



<br/>
<br/>


> <h2 id='混合排序'>混合排序</h2>

NSOrderedAscending 混合排序

```
NSArray *mixArray = @[@"name", @"3", @"5", @"age", @"sex", @"8", @"6", @"height", @"1", @"0" ];
    NSArray *arrayA = [mixArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedAscending;
    }];
NSLog(@"arrayA: %@", arrayA);


#打印结果---------------------------------------------->
2018-11-08 10:17:15.056657+0800 Test[1737:55438] arrayA: (
    name,
    3,
    5,
    age,
    sex,
    8,
    6,
    height,
    1,
    0
)
#---------------------------------------------->
   
```

NSOrderedSame 混合排序

``` 
NSArray *mixArray = @[@"name", @"3", @"5", @"age", @"sex", @"8", @"6", @"height", @"1", @"0" ];
NSArray *arrayB = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedSame;
    }];
    NSLog(@"arrayB: %@", arrayB);



#打印结果------------------------------------------------>
2018-11-08 10:25:22.967135+0800 Test[2096:80796] arrayB: (
    name,
    3,
    5,
    age,
    sex,
    8,
    6,
    height,
    1,
    0
)

#------------------------------------------------------->
    
```


NSOrderedDescending混合排序

```   
NSArray *mixArray = @[@"name", @"3", @"5", @"age", @"sex", @"8", @"6", @"height", @"1", @"0" ];
NSArray *arrayC = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedDescending;
    }];
    NSLog(@"arrayC: %@", arrayC);


#打印结果---------------------------------------------->

2018-11-08 10:26:53.926848+0800 Test[2096:80796] arrayC: (
    0,
    1,
    height,
    6,
    8,
    sex,
    age,
    5,
    3,
    name
)

#------------------------------------------------------->
```

`由此可以得出结论：`

当返回值是：NSOrderedAscending ，NSOrderedSame 排序不变，但是当是NSOrderedDescending 排序颠倒。


<br/>

***
<br/>

```
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr

```

<br/>

**NSOrderedAscending 和 NSOrderedSame**

```
NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
NSArray *sort1 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedAscending;
    }];
    
NSLog(@"arrayC: %@", sort1);



#打印结果---------------------------------------------->
2018-11-08 10:44:54.420773+0800 Test[2324:99796] sort1: (
    3,
    5,
    8,
    6,
    1,
    0
)
#---------------------------------------------->
```
`结论：`NSOrderedAscending和 NSOrderedSame的排序一样，这就不一一验证结果了。


<br/>

NSOrderedDescending

```
 NSArray *sort3 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedDescending;
    }];
    
    NSLog(@"sort3: %@", sort3);


#打印结果---------------------------------------------->
2018-11-08 10:49:01.112905+0800 Test[2324:99796] sort3: (
    0,
    1,
    6,
    8,
    5,
    3
)
#---------------------------------------------->
```
结论：次序颠倒了。


<br/>

**使用 compare 排序**

```
//compare是对oc中的一些对象做比较的方法，根据返回值NSComparisonResult判断比较对象的大小。这里我们主要会用到NSNumer和NSString。NSNumber可以理解，就是直接比较其对应值的大小。而NSString则是对字符串中的字符，根据其ASCII码逐个进行比较
    
NSArray *sort5 = [mixArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    
NSLog(@"sort5: %@", sort5);



#打印结果---------------------------------------------->

2018-11-08 10:53:10.171084+0800 Test[2324:99796] sort5: (
    0,
    1,
    3,
    5,
    6,
    8,
    age,
    height,
    name,
    sex
)
#---------------------------------------------->
```
结论：正常排序。


<br/>
<br/>

> <h2 id='sortUsingComparator'>sortUsingComparator</h2>



`- (void)sortUsingComparator:(NSComparator NS_NOESCAPE)cmptr`


公共代码 

``` 
NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"]; 

NSArray *mixArray = @[@"name", @"3", @"5", @"age", @"sex", @"8", @"6", @"height", @"1", @"0" ];

NSMutableArray *arrayMuNum = [NSMutableArray arrayWithArray:arrayNum];
NSMutableArray *arrayMuMix = [NSMutableArray arrayWithArray:arrayNum];

//NSOrderedSame 和 NSOrderedAscending 

[arrayMuNum sortUsingComparator:NSComparisonResult(id _Nonnull obj1, id _Nonnull obj2) { return NSOrderedAscending; }];

NSLog(@"arrayMuNum: %@", arrayMuNum);

```

打印结果

```
2018-11-08 11:08:56.465967+0800 Test[2571:124393] arrayMuNum: ( 3, 5, 8, 6, 1, 0 )
```

结论：`输出结果一样，不改变排序结果`。



**NSOrderedDescending**

```
[arrayMuNum sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedDescending;
    }];
    
    NSLog(@"arrayMuNum: %@", arrayMuNum);
    
```

输出：

```
2018-11-08 11:17:56.640987+0800 Test[2635:131096] arrayMuNum: (
    0,
    1,
    6,
    8,
    5,
    3
)

```

结论：次序颠倒了！


``` 
[arrayMuNum sortUsingComparator:NSComparisonResult(id _Nonnull obj1, id _Nonnull obj2) { if ([obj1 integerValue] < [obj2 integerValue]) {
 //降序
  return NSOrderedAscending;
   }else {
    //升序
     return NSOrderedDescending;
  } }];

NSLog(@"arrayMuNum: %@", arrayMuNum);
综合使用
```

打印结果

```
2018-11-08 11:58:06.974184+0800 Test[3015:170704] arrayMuNum: ( 8, 6, 5, 3, 1, 0 )

```


<br/>

**compare 排序**

```
//compare是对oc中的一些对象做比较的方法，根据返回值NSComparisonResult判断比较对象的大小。这里我们主要会用到NSNumer和NSString。NSNumber可以理解，就是直接比较其对应值的大小。而NSString则是对字符串中的字符，根据其ASCII码逐个进行比较
    
  [arrayMuMix sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    
    NSLog(@"arrayMuMix: %@", arrayMuMix);
```

打印结果

```
2018-11-08 11:26:19.335257+0800 Test[2751:143683] arrayMuMix: (
    0,
    1,
    3,
    5,
    6,
    8,
    age,
    height,
    name,
    sex
)
```


**汉字排序**

```
- (void) test {
    NSMutableArray *array = [NSMutableArray arrayWithObjects:@"阿门", @"逼格", @"C罗", @"旗木卡卡西", @"旋涡鸣人", @"我们", @"我的", @"佐助", nil];
    [array sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        NSString *word1 = obj1;
        NSString *word2 = obj2;
        //localizedCompare:是Apple提供的根据目前系统语言决定的排序方法，在中文简体时可以进行多音字的排序。
        return [word1 localizedCompare:word2];
    }];
    
    NSLog(@"%@", array);
}
```

打印结果

```
(
C罗,
佐助,
我们,
我的,
旋涡鸣人,
旗木卡卡西,
逼格,
阿门
)
```


<br/>

