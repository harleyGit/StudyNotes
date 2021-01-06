- **NSComparisonResult枚举**
- **sortedArrayUsingComparator**
- **混合排序**
- **sortUsingComparator**

<br/>

***
<br/>

>#  NSComparisonResult枚举

```
typedef NS_CLOSED_ENUM(NSInteger, NSComparisonResult) {
    NSOrderedAscending = -1L,   // 表示两个比较的对象，前者小于后者
    NSOrderedSame,              //表示比较的对象相等
    NSOrderedDescending         //表示两个比较的对象，前者大于后者
};
```



<br/>

***
<br/>


># sortedArrayUsingComparator


```

//按递增的方式排序
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr;

```

<br/>

**NSOrderedAscending 数字排序**

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

***
<br/>

**NSOrderedSame 数字排序**

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

***
<br/>

NSOrderedDescending  数字排序

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

***
<br/>

># 混合排序

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

***
<br/>

># sortUsingComparator

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
[中文排序](https://www.jianshu.com/p/69ff8ed1857f)

[中文拼音](https://blog.csdn.net/jabezlee/article/details/9670965)


