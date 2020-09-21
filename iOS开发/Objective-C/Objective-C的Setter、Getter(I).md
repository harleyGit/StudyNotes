
># 属性重写Setter、Getter

##`若只重写setter和getter其中之一，可以直接重写`
```
//点赞列表
@property(nonatomic, copy) NSMutableArray<PraiseModel *> *topicZanPoList;

//重写Setter方法
- (void)setTopicZanPoList:(NSMutableArray<PraiseModel *> *)topicZanPoList {
    _topicZanPoList = topicZanPoList;
    NSMutableArray *praises = [NSMutableArray arrayWithCapacity:1];
    if (_topicZanPoList.count != 0) {
        for (NSDictionary *pmDic in _topicZanPoList) {
            PraiseModel *pm = [[PraiseModel alloc] initWithDictionary:pmDic error:nil];
            [praises addObject:pm];
        }
    }else {
        return;
    }
    
    _topicZanPoList = praises;
}

````

若这时还要重写Getter方法，则会报以下错误，如图：
![同时写Setter、Getter报错图](https://upload-images.jianshu.io/upload_images/2959789-0c0f379248393c9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
//重写Getter方法
- (NSMutableArray<PraiseModel *> *)topicZanPoList {
    
    return _topicZanPoList;
}

```

注意点：
##`set、get书写错误导致死循环`
```
//在set方法中：
self.age=age;   //相当于是[self setAge:age];

在get方法中:
 return self.age;  //相当于是[self age];
```



<br/>
***
<br/>
参考资料：
[Setter、Getter方法](https://www.cnblogs.com/tangaofeng/p/4858120.html)
[重写Setter、Getter方法，犯得低级错误](https://www.jianshu.com/p/7d0b87d65df4)
