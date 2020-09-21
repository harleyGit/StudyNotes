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

>#数字排序

```
//按递增的方式排序
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr
```

####NSOrderedAscending 数字排序
```
NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
NSArray *array1 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedAscending;
    }];
NSLog(@"array1: %@", array1);
```
***`执行结果`***
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
#### NSOrderedSame 数字排序

```
    NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
 NSArray *array2 = [arrayNum sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedSame;
    }];
    NSLog(@"array2: %@", array2);
```

***`执行结果`***
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
#### NSOrderedDescending  数字排序

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

###NSOrderedAscending 混合排序
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

###NSOrderedSame 混合排序
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


###NSOrderedDescending混合排序
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

#`由此可以得出结论，当返回值是：NSOrderedAscending ，NSOrderedSame 排序不变，但是当是NSOrderedDescending 排序颠倒。`


<br/>
***
<br/>
```
- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr

```

<br/>
###NSOrderedAscending 和 NSOrderedSame
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
###NSOrderedDescending

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


###使用 compare 排序

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

