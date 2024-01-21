> <h2 id=''></h2>
- [**下载MJRefresh**](#下载MJRefresh)
- [**使用**](#使用)
	- [模型->字典](#模型转字典)
	- [字典->模型](#字典转模型)
	- [数组字典转数组模型](#数组字典转数组模型)
	- [数组中字典转化为其他模型](#数组中字典转化为其他模型)
- [**下拉刷新**](#下拉刷新)
- **参考资料:**
	- [MJRefresh的使用](https://www.cnblogs.com/xvewuzhijing/p/4904629.html)
	- [MJRefresh篇一](https://www.jianshu.com/p/9b7b86d23a63)
	- [JSON转Model的库 MJExtension的基本使用指导](https://blog.csdn.net/deft_mkjing/article/details/51704898)
	- [序列化框架MJExtension详解 + iOS ORM框架](https://www.jianshu.com/p/11a8e15f7d2b)




<br/>

***

<br/><br/>

> <h1 id='下载MJRefresh'>下载MJRefresh</h1>


```
//搜索
pod  search MJRefresh

//找到版本
pod 'MJRefresh', '~> 3.1.15.7'


//更新版本
pod update --verbose --no-repo-update
```

<br/>

![MJRefresh 结构图](https://upload-images.jianshu.io/upload_images/2959789-039933455fecb7e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/><br/>

> <h1 id='使用'>使用</h1>

<br/><br/>

> <h2 id='模型转字典
'>模型 -> 字典</h2>

```
Model  *model = [[Model alloc] init];
model.data1 = data[@"one"];
model.data2 = data[@"two"];
//用一个可变字典做接收，转出来的就是一个以data1和data2为键的字典了。
NSMutableDictionary *params = [model mj_keyValues];
NSLog(@"---> %@", params);    
//打印结果--->{data1 = one;  data2 = two}
```


<br/><br/>

> <h2 id='字典转模型'>字典 -> 模型</h2>

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

<br/><br/>

> <h2 id='数组字典转数组模型'>数组字典转数组模型</h2>

```
//数组中需要转换的模型类
+ (NSDictionary *)mj_objectClassInArray;  
```


<br/><br/>

> <h2 id='数组中字典转化为其他模型'>数组中字典转化为其他模型</h2>

**.h文件**

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN
@protocol BaseCCellProtocol;

@interface HGSectionModel : NSObject

/// section标题
@property(nonatomic, copy)NSString *sectionTitle;
@property(nonatomic, copy)NSMutableArray<id <BaseCCellProtocol>> *cells;;

@end

NS_ASSUME_NONNULL_END
```

<br/>

**.m文件**

```
#import "HGSectionModel.h"

@implementation HGSectionModel

///优势:不需要导入BaseCollectionCellM类头文件
+ (NSDictionary *)mj_objectClassInArray {
    return @{
        @"cells": @"BaseCollectionCellM",
    };
}

@end

```



<br/>

***

<br/><br/>

> <h1 id='下拉刷新'>下拉刷新</h1>

```
- (UITableView *)contentView {
    if (!_contentView) {
        _contentView = [UITableView new];
        _contentView.delegate = self;
        _contentView.dataSource = self;
    }
    return _contentView;
}

- (void) startRefreshDataForTable {

    self.contentView.mj_header = [MJRefreshHeader headerWithRefreshingBlock:^{
        //开始刷新请求数据
        NSLog(@"开始刷新请求数据");
    }];
    
  //结束刷新数据
  [self.contentView.mj_header endRefreshing];

}

- (void) startAction {
    [self.contentView.mj_header beginRefreshing];
}


```


<br/>

***

<br/><br/>

> <h1 id='上拉加载'>上拉加载</h1>






**`注意：`**

&emsp;  在iOS11下要注意了，因为UITableView的contentSize，是默认使用estimateRowHeight属性计算的包含Headers, Footers和cells，这样就会造成contentSize和contentOffset值的变化，如果是有动画是观察这两个属性的变化进行的，就会造成动画的异常，因为在估算行高机制下，contentSize的值是一点点地变化更新的，所有cell显示完后才是最终的contentSize值。因为不会缓存正确的行高，tableView reloadData的时候，会重新计算contentSize，就有可能会引起contentOffset的变化。

解决办法：

```

if (@available(iOS 11.0, *)) {
    tableView.estimatedRowHeight = 0;
    tableView.estimatedSectionFooterHeight = 0;
    tableView.estimatedSectionHeaderHeight = 0;
    tableView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
}

```



<br/>
***
<br/>

[]()

<br/>
