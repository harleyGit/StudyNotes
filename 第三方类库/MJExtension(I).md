```
//搜索类库
$ pod search MJExtension
1
//拷贝到Podfile文件
pod 'MJExtension', '~> 3.0.15.1'

//更新
$ pod update --verbose

```

># Method

`- (NSMutableDictionary *)mj_keyValues    //模型 -> 字典`
```
Model  *model = [[Model alloc] init];
model.data1 = data[@"one"];
model.data2 = data[@"two"];
//用一个可变字典做接收，转出来的就是一个以data1和data2为键的字典了。
NSMutableDictionary *params = [model mj_keyValues];
NSLog(@"---> %@", params);    
//打印结果--->{data1 = one;  data2 = two}

```

<br/>
`- (instancetype)mj_setKeyValues:(id)keyValues      //字典 -> 模型`
```
NSLog(@"homeTitle JSON数据为： %@", dicArr);
/*
2019-01-17 21:11:54.659617+0800 HGSWB[23752:244155] homeTitle JSON数据为： {
    category = "news_hot";
    "concern_id" = "";
    "default_add" = 1;
    flags = 0;
    "icon_url" = "";
    name = "\U70ed\U70b9";
    "tip_new" = 0;
    type = 4;
    "web_url" = "";
}
*/

//HGHomeTitleModel 的属性要与dicArr的键相同，否则无法匹配
HGHomeTitleModel *model = [[HGHomeTitleModel new] mj_setKeyValues:dicArr];

// po model
//<HGHomeTitleModel: 0x600000df2670>

```


<br/>
`+ (NSDictionary *)mj_objectClassInArray;  //数组中需要转换的模型类`

```

```



<br/>
***
<br/>
参考资料：[MJExtension的基本使用指导](https://blog.csdn.net/deft_mkjing/article/details/51704898)
