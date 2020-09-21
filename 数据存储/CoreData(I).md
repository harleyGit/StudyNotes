>#存储数组、对象数据
![数组对象数组](https://upload-images.jianshu.io/upload_images/2959789-f108d87c1c349c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将其Type设置为Transformable即可，可用来设置iOS对象属性。其生成的属性为：
```
@property (nullable, nonatomic, retain) NSObject *commentHeightArray;
```

##添加属性
在表中添加属性后，可以按照下面的步骤来添加属性
![添加属性后，可以按照这个步骤来对.h文件增加属性](https://upload-images.jianshu.io/upload_images/2959789-10f47a8f6bf7018b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## `Demo 代码`

![数据库和表(包括表的属性)](https://upload-images.jianshu.io/upload_images/2959789-3023ce6b6b5b68c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![表的属性](https://upload-images.jianshu.io/upload_images/2959789-a33234498e9f9313.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
`ManagerDatabase.m 文件`
```

#import "ManagerDatabase.h"

@implementation ManagerDatabase

+ (NSManagedObjectContext *)shareContext {
    //1、创建模型对象
    //获取模型路径
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"TaiJiDaoDatabase" withExtension:@"momd"];
    //根据模型文件创建模型对象
    NSManagedObjectModel *model = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    
    
    //2、创建持久化存储助理：数据库
    //利用模型对象创建助理对象
    NSPersistentStoreCoordinator *store = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];
    
    //数据库的名称和路径
    NSString *docStr = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    NSString *sqlPath = [docStr stringByAppendingPathComponent:@"coreData.sqlite"];
    NSLog(@"数据库 path = %@", sqlPath);
    NSURL *sqlUrl = [NSURL fileURLWithPath:sqlPath];
    
    NSError *error = nil;
    //设置数据库相关信息 添加一个持久化存储库并设置存储类型和路径，NSSQLiteStoreType：SQLite作为存储库
    [store addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:sqlUrl options:nil error:&error];
    
    if (error) {
        NSLog(@"添加数据库失败:%@",error);
    } else {
        NSLog(@"添加数据库成功");
    }
    
    //3、创建上下文 保存信息 操作数据库
    
    NSManagedObjectContext *context = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
    
    //关联持久化助理
    context.persistentStoreCoordinator = store;
    
    return context;
}

@end

```


<br/>
##`DynamicLayout .h 文件`

```
NS_ASSUME_NONNULL_BEGIN

@interface DynamicLayout : NSObject

@property(nonatomic, retain) JSONModel *allCommentM;
@property(nonatomic, assign) CGFloat height;
@property(nonatomic, retain) NSMutableAttributedString *attributeContent;
@property(nonatomic, assign) CGSize  pictureContainerSize;
@property(nonatomic, assign) CGFloat pictureContainerHeight;
@property(nonatomic, assign) CGFloat pictureContainerWidth;
@property(nonatomic, assign) CGFloat praiseHeight;
@property(nonatomic, assign) CGFloat commentHeight;
@property(nonatomic, assign) CGFloat praiseCommentHeight;
//评论高度数组
@property(nonatomic, retain) NSMutableArray *commentHeightArray;

@end

NS_ASSUME_NONNULL_END
```



<br/>
##`写入、查询、修改数据`
##`DynamicLayout.m 文件`

```
//写入数据
- (void) storeCellHeightForTalk:(DynamicLayout *) layout{
    InteractiveZoneCellHeight *izc = [NSEntityDescription insertNewObjectForEntityForName:@"InteractiveZoneCellHeight" inManagedObjectContext:self.managerContext];
    
    izc.topicId = layout.allCommentM.topicId;
    izc.pictureContainerWidth = layout.pictureContainerSize.width;
    izc.pictureContainerHeight = layout.pictureContainerSize.height;
    izc.praiseHeight = layout.praiseHeight;
    izc.commentHeight = layout.commentHeight;
    izc.praiseCommentHeight = layout.praiseCommentHeight;
    izc.commentHeightArray = layout.commentHeightArray;
    izc.height = layout.height;
    
    NSError *error =nil;
    if ([self.managerContext save:&error]) {
        NSLog(@"插入数据成功");
    }else{
        NSLog(@"插入数据失败！");
    }
}


//查询并修改数据
- (BOOL) isStoreCellHeightForTalk:(AllCommentModel *) model {
    BOOL isStore = NO;
    //创建查询请求
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"InteractiveZoneCellHeight"];
    //查询条件
    NSPredicate *pre = [NSPredicate predicateWithFormat:@"topicId = %@", model.topicId];
    request.predicate = pre;
    
    InteractiveZoneCellHeight *izch = [self.managerContext executeFetchRequest:request error:nil].firstObject;

    if (izch != nil) {
        self.height = izch.height;
        self.pictureContainerSize = CGSizeMake(izch.pictureContainerWidth, izch.pictureContainerHeight);
        self.praiseHeight = izch.praiseHeight;
        self.commentHeight = izch.commentHeight;
        self.commentHeightArray = (NSMutableArray *)izch.commentHeightArray;
        self.praiseCommentHeight = izch.praiseCommentHeight;
        isStore = YES;
        
        return isStore;
    }else {
        return isStore;
    }
}


//修改数据
- (void)selectCellHeight:(DynamicLayout *)layout {
   
    //创建查询请求
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"InteractiveZoneCellHeight"];
    //查询条件
    NSPredicate *pre = [NSPredicate predicateWithFormat:@"topicId = %@", layout.allCommentM.topicId];
    request.predicate = pre;
    
    InteractiveZoneCellHeight *izch = [self.managerContext executeFetchRequest:request error:nil].firstObject;
    
    izch.topicId = layout.allCommentM.topicId;
    izch.pictureContainerWidth = layout.pictureContainerSize.width;
    izch.pictureContainerHeight = layout.pictureContainerSize.height;
    izch.praiseHeight = layout.praiseHeight;
    izch.commentHeight = layout.commentHeight;
    izch.praiseCommentHeight = layout.praiseCommentHeight;
    izch.commentHeightArray = layout.commentHeightArray;
    izch.height = layout.height;
    
    NSError *error =nil;
    if ([self.managerContext save:&error]) {
        NSLog(@"修改数据成功");
    }else{
        NSLog(@"修改数据失败！");
    }
}


```

<br/>
***
<br/>


>#查看CoreData中sqlite3存储的数据：

## 查看数据库路径
```
//sqlite3 + 数据库路径
```
![查看数据](https://upload-images.jianshu.io/upload_images/2959789-f7ebdde49d17c5be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##查看数据库的table
![查看数据库中的表单](https://upload-images.jianshu.io/upload_images/2959789-45f2db7648f783f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##查看表中数据
```
select * from [表单名] limit 1;

//退出sqlite3命令
.quit;
```

<br/>
***
<br/>


## 不常用的命令

| 命令 | 含义 |
| --- | --- | 
| .show | 显示格式的配置情况(一会我们显示表中数据的时候就会看到效果)
| .schema | 建表的模式(就是建表语句,个人观点) | 
| .mode | 显示的格式，建议用line分行显示,首先输入查询语句 | 
| .headers on | 显示字段名，注意在column模式才有用 | 









<br/>
***
<br/>
参考资料：
[CoreData 增删改查(I)](https://www.jianshu.com/p/332cba029b95)
