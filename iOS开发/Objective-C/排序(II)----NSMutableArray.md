```
- (void)sortUsingComparator:(NSComparator NS_NOESCAPE)cmptr
```


<br/>

###公共代码
```
NSArray *arrayNum = @[@"3", @"5", @"8", @"6", @"1", @"0"];
    NSArray *mixArray = @[@"name", @"3", @"5", @"age", @"sex", @"8", @"6", @"height", @"1", @"0" ];
    
    NSMutableArray *arrayMuNum = [NSMutableArray arrayWithArray:arrayNum];
    NSMutableArray *arrayMuMix = [NSMutableArray arrayWithArray:arrayNum];

```

<br/>
### NSOrderedSame 和 NSOrderedAscending 

```
[arrayMuNum sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedAscending;
    }];
    
NSLog(@"arrayMuNum: %@", arrayMuNum);

#打印结果---------------------------------------------------------->
2018-11-08 11:08:56.465967+0800 Test[2571:124393] arrayMuNum: (
    3,
    5,
    8,
    6,
    1,
    0
)
#---------------------------------------------------------->
```
`结论：`输出结果一样，不改变排序结果。


<br/>
### NSOrderedDescending

```
[arrayMuNum sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return NSOrderedDescending;
    }];
    
    NSLog(@"arrayMuNum: %@", arrayMuNum);
    
#打印结果---------------------------------------------------------->
2018-11-08 11:17:56.640987+0800 Test[2635:131096] arrayMuNum: (
    0,
    1,
    6,
    8,
    5,
    3
)

#---------------------------------------------------------->


```
结论：次序颠倒了！


<br/>
```
[arrayMuNum sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        if ([obj1 integerValue] < [obj2 integerValue])
        {
            //降序
            return NSOrderedAscending;
        }else {
            //升序
            return NSOrderedDescending;
        }
    }];

    NSLog(@"arrayMuNum: %@", arrayMuNum);


### 综合使用
#打印结果---------------------------------------------------------->

2018-11-08 11:58:06.974184+0800 Test[3015:170704] arrayMuNum: (
    8,
    6,
    5,
    3,
    1,
    0
)
#---------------------------------------------------------->
```


<br/>
### compare 排序

```
//compare是对oc中的一些对象做比较的方法，根据返回值NSComparisonResult判断比较对象的大小。这里我们主要会用到NSNumer和NSString。NSNumber可以理解，就是直接比较其对应值的大小。而NSString则是对字符串中的字符，根据其ASCII码逐个进行比较
    
  [arrayMuMix sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    
    NSLog(@"arrayMuMix: %@", arrayMuMix);

#打印结果---------------------------------------------------------->
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
#---------------------------------------------------------->

```

<br/>
### 汉字排序
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


#打印结果---------------------------------------------------------->
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
#---------------------------------------------------------->
```













<br/>
***
<br/>
***`参考资料:`***
[中文排序](https://www.jianshu.com/p/69ff8ed1857f)
[中文拼音](https://blog.csdn.net/jabezlee/article/details/9670965)
