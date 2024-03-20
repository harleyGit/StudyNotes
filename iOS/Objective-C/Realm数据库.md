> <h1 id=''></h1>
- [**简介**](#简介)
- [**‌RLMRealmConfiguration配置**](#RLMRealmConfiguration配置)
- [**数据模型RLMObject**](#数据模型RLMObject)
	- [举例使用](#举例使用)
	- [模型新增属性处理](#模型新增属性处理)
	- 


<br/>

***
<br/><br/>简介<br/>

> <h1 id='简介'>简介</h1>

Realm是一种流行的移动数据库，它提供了一种简单而又快速的方法来存储和检索对象。在Realm中，RLMObject是用于表示数据库中的对象的基类


<br/>

***
<br/><br/><br/>

> <h1 id='RLMRealmConfiguration配置'>RLMRealmConfiguration配置</h1>

在 Realm 中，RLMRealmConfiguration 通常用于配置 Realm 数据库的行为，例如指定数据库文件的位置、设置数据库的架构版本等。在数据库迁移的过程中，您可能会利用 RLMRealmConfiguration 来执行一些配置，但这并不是必须的。增加属性后，您主要需要执行数据库迁移，以确保现有的数据库与新的模型属性保持同步。


<br/>

***
<br/><br/><br/>

> <h1 id='数据模型RLMObject'>数据模型RLMObject</h1>

- **RLMObject具有以下作用：**

	- 数据模型定义：RLMObject用于定义数据模型，它充当了对象关系映射（ORM）中的模型对象。您可以创建子类来定义您的数据模型，并在这些子类中定义属性，这些属性将成为数据库中的字段。
	
	- 数据持久化：您可以将RLMObject子类的对象存储到Realm数据库中。Realm数据库将对象持久化到磁盘，以便在应用程序的不同执行周期中保留数据。
	
	- 对象关系管理：RLMObject可以在数据库中建立对象之间的关系。您可以在RLMObject子类中定义其他RLMObject子类的属性，从而表示对象之间的关系，例如一对一、一对多或多对多关系。
	
	- 查询：您可以使用Realm查询语言（RLMQueries）对存储在Realm数据库中的RLMObject对象进行检索。您可以使用查询来查找、过滤、排序和组合对象，以满足您的应用程序需求。
	
	- 事务支持：Realm支持事务，RLMObject的修改必须在事务中完成。这意味着您可以在修改多个对象时保持数据库的一致性，并可以通过撤消操作回滚事务。

<br/>

总的来说，RLMObject是Realm中的核心概念之一，它为您提供了一个轻量级、高效的方式来管理和操作您的应用程序数据。






<br/><br/>

> <h2 id='举例使用'>举例使用</h2>


**模型:**

```
#import <Realm/Realm.h>

// 定义Task类
@interface Task : RLMObject
@property NSString *title;
@property NSString *details;
@property NSDate *dueDate;
@property BOOL completed;
@property Category *category;
@end

@implementation Task
@end
```

<br/>

```
// 创建一个新的任务
Task *newTask = [[Task alloc] init];
newTask.title = @"Finish project";
newTask.details = @"Complete the coding part of the project";
newTask.dueDate = [NSDate dateWithTimeIntervalSinceNow:3600]; // Due date 1 hour from now
newTask.completed = NO;

// 获取默认的Realm实例
RLMRealm *realm = [RLMRealm defaultRealm];

// 在事务中保存任务和类别对象到数据库中
[realm transactionWithBlock:^{
    [realm addObject:newTask];
    [realm addObject:newCategory];
}];
```

我们就创建了一个新的任务和一个新的类别，并将任务分配给了类别。然后，我们使用Realm的事务将这些对象保存到数据库中。


<br/><br/>

> <h2 id='模型新增属性处理'>模型新增属性处理</h2>

在Realm中，当您对已存在的RLMObject添加新的属性时，之前已经保存的对象会遇到一些问题：

- 默认值问题：新添加的属性需要有默认值，否则会导致插入数据库失败。Realm要求新添加的可选属性要么具有默认值，要么是可为空的。

- 迁移问题：即使您提供了默认值，旧数据也不会自动更新以匹配新的模式。因此，您需要执行一个数据迁移过程来更新现有的数据，使其与新的RLMObject模型匹配。

<br/>


下面是一个示例解决方案，展示了如何使用Realm的迁移功能来处理这种情况：

**模型:**

```
// 更新 Person 模型类，添加 email 属性
@interface Person : RLMObject
@property NSString *name;
@property NSInteger age;
@property NSString *email; // 新的属性
@end

@implementation Person
@end

```

<br/>

在这种情况下，您需要执行手动的数据库迁移来更新现有的Realm数据库，以包含新的email属性。您可以使用以下步骤来执行数据库迁移：

```
// 获取默认的 Realm 实例
RLMRealm *realm = [RLMRealm defaultRealm];

// 在一个 Realm 事务中执行数据库迁移
[realm transactionWithBlock:^{
    // 添加新的列到 Person 表中
    [realm.schema enumerateObjectSchema:^(RLMObjectSchema * _Nonnull objectSchema) {
        if ([objectSchema.className isEqualToString:@"Person"]) {
            [objectSchema addProperty:@"email" type:RLMPropertyTypeString];
        }
    }];
}];

// 更新所有已存在的 Person 对象，为新的属性设置默认值
[realm transactionWithBlock:^{
    for (Person *person in [Person allObjects]) {
        person.email = @""; // 可以根据需要设置默认值
    }
}];

```

通过执行以上步骤，您可以手动执行数据库迁移来更新现有的Realm数据库结构，以适应新的模型属性。这样，您就可以确保数据库与模型的变化保持同步，而不会丢失现有数据。
