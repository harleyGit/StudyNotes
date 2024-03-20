> <h1 id=''></h1>
- [模型新增属性处理](#模型新增属性处理)



<br/>

***
<br/><br/><br/>

> <h1 id=''></h1>


<br/><br/><br/>

> <h2 id='模型新增属性处理'>模型新增属性处理</h2>

当您需要在使用FMDB时更新数据库表以适应新的模型属性时，您可以使用以下步骤：

- 确保您已经在模型类中添加了新的属性。
- 执行数据库迁移以更新表结构。
- 如果新的属性是非可选的，您可能需要为现有数据提供默认值或处理NULL值。

以下是一个在Objective-C中使用FMDB执行数据库迁移的详细示例：

```
// 导入FMDB库
#import "FMDB.h"

// 模型类定义
@interface YourModel : NSObject

@property (nonatomic, strong) NSString *oldProperty; // 已存在的属性
@property (nonatomic, strong) NSString *newProperty; // 新添加的属性

@end

// 数据库管理器类
@interface DatabaseManager : NSObject

@property (nonatomic, strong) FMDatabase *database;

+ (instancetype)sharedInstance;
- (BOOL)performDatabaseMigrationIfNeeded;

@end

@implementation DatabaseManager

+ (instancetype)sharedInstance {
    static DatabaseManager *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[DatabaseManager alloc] init];
    });
    return instance;
}

- (instancetype)init {
    self = [super init];
    if (self) {
        // 初始化数据库
        NSString *databasePath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
        databasePath = [databasePath stringByAppendingPathComponent:@"your_database.sqlite"];
        self.database = [FMDatabase databaseWithPath:databasePath];
    }
    return self;
}

- (BOOL)performDatabaseMigrationIfNeeded {
    if (![self.database open]) {
        NSLog(@"Failed to open database");
        return NO;
    }

    // 检查是否需要迁移数据库
    BOOL migrationNeeded = NO;
    FMResultSet *resultSet = [self.database executeQuery:@"PRAGMA table_info(your_table)"];
    while ([resultSet next]) {
        NSString *columnName = [resultSet stringForColumn:@"name"];
        if ([columnName isEqualToString:@"new_property"]) {
            // 如果表中已存在新的属性，则无需迁移
            migrationNeeded = NO;
            break;
        } else {
            // 否则，需要进行迁移
            migrationNeeded = YES;
        }
    }
    [resultSet close];

    if (migrationNeeded) {
        // 执行数据库迁移
        NSString *migrationSQL = @"ALTER TABLE your_table ADD COLUMN new_property TEXT";
        BOOL success = [self.database executeUpdate:migrationSQL];
        if (!success) {
            NSLog(@"Failed to migrate database");
            [self.database close];
            return NO;
        } else {
            NSLog(@"Database migrated successfully");
        }
    }

    [self.database close];
    return YES;
}

@end
```

在上面的示例中，我们假设有一个名为YourModel的模型类，其中包含了一个已存在的属性oldProperty和一个新添加的属性newProperty。我们还创建了一个名为DatabaseManager的数据库管理器类来执行数据库迁移。

performDatabaseMigrationIfNeeded方法用于执行数据库迁移。它首先打开数据库，然后查询表结构以查看是否需要迁移。如果表中不存在新属性，则执行数据库迁移以添加新列。最后，关闭数据库连接。

您可以在应用程序启动时调用performDatabaseMigrationIfNeeded方法，以确保在模型类发生更改时及时更新数据库表结构。

这样，您就可以在使用FMDB时确保数据库表结构与模型类的属性保持同步，实现了数据库迁移。

