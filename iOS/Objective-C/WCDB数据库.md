> <h1 id=''></h1>
[模型新增属性处理](#模型新增属性处理)

<br/>

***
<br/><br/><br/>

> <h1 id=''></h1>


<br/><br/><br/>

> <h2 id='模型新增属性处理'>模型新增属性处理</h2>

在WCDB中，模型新增属性会自动迁移。WCDB会自动根据您定义的新模型进行数据库结构的更新，无需手动编写迁移代码。这种自动迁移功能使得处理模型的更改变得非常方便。

下面是一个详细的Objective-C示例，演示了如何在WCDB中进行模型属性的自动迁移：

假设我们有一个名为 Person 的模型类，具有 name 和 age 两个属性：

**Person模型:**

假设我们想要向 Person 类中添加一个新的属性 email。我们只需在模型类中添加新的属性即可，WCDB会自动处理数据库结构的更新。

```
#import <WCDB/WCDB.h>

@interface Person : WCTTableCoding

@property NSString *name;
@property NSInteger age;
@property NSString *email; // 新的属性

@end

@implementation Person

WCDB_PROPERTY(name)
WCDB_PROPERTY(age)
WCDB_PROPERTY(email)// 新的属性

@end
```


<br/>

在这种情况下，当您使用新的模型类时，WCDB会自动检测到模型的更改，并在运行时自动更新数据库结构以适应新的属性。这意味着您可以继续使用WCDB操作数据库，而不必担心手动处理数据库迁移的细节。

```
// 使用WCDB操作数据库
WCTDatabase *database = [WCTDatabase databaseWithFile:pathToDatabase];
WCTTable<Person *> *table = [database getTable:Person.class];

// 添加一个新的 Person 对象到数据库
Person *person = [[Person alloc] init];
person.name = @"Alice";
person.age = 30;
person.email = @"alice@example.com"; // 使用新的属性

[table insertRecord:person];
```

&emsp; 通过这种方式，WCDB会自动处理数据库结构的更新，使得您可以方便地对模型进行更改，而无需手动执行迁移操作。这样，您就能够以更高效和简便的方式管理数据库结构。
