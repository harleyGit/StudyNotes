># 简介
&emsp;&emsp;   NSCache是苹果官方提供的缓存类，用法与NSMutableDictionary的用法很相似，在AFNetworking和SDWebImage中，使用它来管理缓存。

&emsp;&emsp;   NSCache在系统内存很低时，会自动释放一些对象（出自苹果官方文档，不过在模拟器中模拟内存警告时，不会做缓存的清理动作） 为了确保接收到内存警告时能够真正释放内存，最好调用一下removeAllObjects方法。

&emsp;&emsp;   NScache是线程安全的，在多线程操作中，不需要对Cache加锁。

&emsp;&emsp;   NScache的key只是做强引用，不需要实现NScopying协议。

<br/>
***
<br/>

># 属性
```
delegate:  代理属性;

totalCostLimit：缓存空间的最大成本，超出上限会自动回收对象。默认值是0没有限制;

countLimit：能够缓存对象的最大数量，默认值也是0（默认没有限制,当超出缓存最大成本或数量时，NSCache会把前面的数据即最开始存的给清除掉);

evictsObjectsWithDiscardedContent：标示是否回收废弃的内容，默认值是YES（自动回收);
```



<br/>
***
<br/>



>#NSCache的方法

```
-objectForKey:  返回与键值关联的对象;

-setObject: forKey:  在缓存中设置指定键名对应的值。与可变字典不同的是，缓存对象不会对键名做copy操作 0成本;

-setObject: forKey: cost:   在缓存中设置指定键名对应的值，并且指定该键值对的成本。成本cost用于计算记录在缓冲中所有对象的总成本。当出现内存警告，或者超出缓存的成本上限时，缓存会开启一个回收过程，删除部分元素。

-removeObjectForKey：删除缓存中指定键名的对象;

-removeAllObjects：删除缓存中的所有对象;
```



<br/>
***
<br/>


># 委托方法
```
//缓存将要删除对象时调用，不能在此方法中修改缓存。仅仅用于后台的打印，以便于程序员的测试;
- (void)cache:(NSCache *)cache willEvictObject:(id)obj  {} 

```

**Demo**
```
@interface ViewController ()<NSCacheDelegate>
@property(nonatomic, retain) NSCache *cache;
@end


@implementation ViewController

- (NSCache *)cache {
    if (_cache == nil) {
       _cache = [[NSCache alloc] init];
       // 设置数量限制,最大限制为10
       _cache.countLimit = 5;
       _cache.delegate = self;
    }
    return _cache;
}


/*
*按钮点击触发方法调用：- (void) testNSCacheMethod{}
*
*/


#pragma mark -- NSCacheDelegate
// MARK: NSCache Delegate
// 当缓存中的对象被清除的时候，会自动调用
// obj 就是要被清理的对象
// 提示：不建议平时开发时重写！仅供调试使用
- (void)cache:(NSCache *)cache willEvictObject:(id)obj {
    [NSThread sleepForTimeInterval:0.5];
    NSLog(@"清除了-------> %@", obj);
}


- (void) testNSCacheMethod {
    for (int i = 0; i < 10; ++i) {
        NSString *str = [NSString stringWithFormat:@"hello - %04d", I];

        NSLog(@"设置 %@", str);
        // 添加到缓存
        [self.cache setObject:str forKey:@(i)];
    }

    // - 查看缓存内容，NSCache 没有提供遍历的方法，只支持用 key 来取值
    for (int i = 0; i < 10; ++i) {
        NSLog(@"缓存中----->%@", [self.cache objectForKey:@(i)]);
    }
}

@end

```
效果：
![控制台打印](https://upload-images.jianshu.io/upload_images/2959789-668592588598d5ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; 由此我们看到，当我们把countLimit = 5时，先存储5条接着会把最开始的4条删除掉然后添加最后的4条，即最终会存储5条数据；

<br/>
**`沙箱读取文件Demo`**
```
#import "AHPersonInfoManager.h"
#import "YYModel.h"
@interface AHPersonInfoManager()
//用户信息模型
@property(nonatomic,strong)AHPersonInfoModel *model;
//个人信息缓存
@property(nonatomic,strong)NSCache *infoModelCache;

@end

@implementation AHPersonInfoManager

+(instancetype)manager{
    static AHPersonInfoManager *personInfoManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        @synchronized (personInfoManager) {
            personInfoManager = [[AHPersonInfoManager alloc]init];
            personInfoManager.infoModelCache = [[NSCache alloc]init];
        }
       
    });
    return personInfoManager;
}

#pragma mark 个人资料

-(AHPersonInfoModel*)getInfoModel{
    //读取缓存
    if (_infoModelCache) {
        return [_infoModelCache objectForKey:@"infoModel"];
    }
    //读取文件
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    //获取完整路径
    NSString *documentsDirectory = [paths objectAtIndex:0];
//    LOG(@"homeDirectory = %@", documentsDirectory);
    NSString *plistPath = [documentsDirectory stringByAppendingPathComponent:PersonInfoFilePath];
    //读出文件
    if (  [[NSFileManager defaultManager] fileExistsAtPath:plistPath]) {
        NSMutableDictionary *infoDic = [[[NSMutableDictionary alloc] initWithContentsOfFile:plistPath] mutableCopy];
        _model = [AHPersonInfoModel yy_modelWithDictionary:infoDic];
    }else{
        //防nil
        _model = [[AHPersonInfoModel alloc]init];
    }
    //写入缓存
    [_infoModelCache setObject:_model forKey:@"infoModel"];
    return _model;
}

-(void)setInfoModel:(AHPersonInfoModel*)infoModel{
    @synchronized (self) {
        //写入缓存
        if (!_infoModelCache) {
            _infoModelCache = [[NSCache alloc]init];
        }
        [_infoModelCache setObject:infoModel forKey:@"infoModel"];
        //写入文件
        NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        //获取完整路径
        NSString *documentsDirectory = [paths objectAtIndex:0];
        //    LOG(@"homeDirectory = %@", documentsDirectory);
        NSString *plistPath = [documentsDirectory stringByAppendingPathComponent:PersonInfoFilePath];
        NSDictionary *dic = [infoModel yy_modelToJSONObject];
        //写入文件
        [dic writeToFile:plistPath atomically:YES];
        _model = infoModel;
    }

}

-(void)setWithJson:(id)json{
    @synchronized (self) {
        NSMutableDictionary *newInfoDic = [json yy_modelToJSONObject];
        NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        //获取完整路径
        NSString *documentsDirectory = [paths objectAtIndex:0];
        //    LOG(@"homeDirectory = %@", documentsDirectory);
        NSString *plistPath = [documentsDirectory stringByAppendingPathComponent:PersonInfoFilePath];
        //原模型字典
        NSMutableDictionary *oldInfoDic= [[[NSMutableDictionary alloc] initWithContentsOfFile:plistPath] mutableCopy];
        //防nil
        if (!([oldInfoDic allKeys].count >0)) {
            oldInfoDic = [NSMutableDictionary dictionary];
        }
        
        //修改对应的key-value
        NSArray *keyArray  = [newInfoDic allKeys];
        if (keyArray.count>0) {
            for (NSString *key in keyArray) {
                if (  [newInfoDic objectForKey:key]) {
                    //去除星座
                    if ([key isEqualToString:@"constellation"]) {
                        
                    }else{
                        [oldInfoDic setObject:newInfoDic[key] forKey:key];
                    }
                    
                }
            }
            [oldInfoDic writeToFile:plistPath atomically:YES];
            //写入缓存
            if (!_infoModelCache) {
                _infoModelCache = [[NSCache alloc]init];
            }
            AHPersonInfoModel *infoModel = [AHPersonInfoModel yy_modelWithJSON:oldInfoDic];
            [_infoModelCache setObject:infoModel forKey:@"infoModel"];
       
        }
    }
}
```




<br/>
***
<br/>

<br/>
参考资料：
