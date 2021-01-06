># 下载MJRefresh

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
<br/>
># 下拉刷新

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
<br/>


>#上拉加载






#`注意：`
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
参考资料：
[MJRefresh的使用](https://www.cnblogs.com/xvewuzhijing/p/4904629.html)
[MJRefresh篇一](https://www.jianshu.com/p/9b7b86d23a63)
