> <h1 id=""></h1>
- [递归解析模型](#递归解析模型)
- [YYModel](#YYModel)



<br/>

***

<br/><br/><br/>

> <h1 id="YYKit">YYKit</h1>

- YYKit 是一个功能强大的综合性 iOS 库，集合了多个常用工具类，旨在提高 iOS 开发的效率和代码质量。YYKit 包含以下主要模块：

	- YYModel：用于模型对象的序列化和反序列化。
	- YYCache：用于高效的缓存处理。
	- YYImage：用于处理图像的加载、解码和显示。
	- YYText：用于高级文本处理和排版。
	- YYWebImage：用于网络图像加载和缓存。
	- YYAsyncLayer：用于异步绘制，提升性能。

<br/><br/><br/>

> <h2 id="递归解析模型">递归解析模型</h2>


> <h2 id=""></h2>
[模型转换为字典或者数组](#模型转换为字典或者数组)


<br/><br/><br/>

> <h2 id="模型转换为字典或者数组">模型转换为字典或者数组</h2>

```
- (id)modelToJSONObject
```

NSObject的分类方法，可以递归的将模型中的属性递归解析成数组、字典进行返回；

在控制台进行打印还是不错的。


<br/><br/><br/>

> <h2 id="YYModel">YYModel</h2>

YYModel 是 YYKit 中的一个独立模块，专门用于模型对象与 JSON 之间的转换。它的设计目标是高效、易用且功能强大。YYModel 的主要功能包括：

- JSON 到模型对象的转换：将 JSON 数据解析成 Objective-C 模型对象。
- 模型对象到 JSON 的转换：将模型对象序列化成 JSON 数据。
- 支持复杂的数据结构：包括嵌套模型、数组、字典等。
- 高性能：优化的解析和序列化算法，性能优于许多其他同类库。


<br/><br/>

YYKit 和 YYModel 的联系
- 包容关系：YYModel 是 YYKit 中的一个模块。YYKit 集成了包括 YYModel 在内的多个子模块，为开发者提供了一站式的解决方案。
- 独立使用：尽管 YYModel 是 YYKit 的一部分，但它也可以作为一个独立的库单独使用。如果你只需要进行 JSON 和模型对象之间的转换，可以直接使用 YYModel，而无需引入整个 YYKit。
- 设计理念：两个库的设计理念一致，都是为了提供高效、简洁、易用的工具，提升 iOS 开发的效率。


<br/><br/>


**一个简短的使用YYModel**

```
#import <YYModel/YYModel.h>

@interface Student : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@end

@implementation Student
@end

@interface ClassRoom : NSObject
@property (nonatomic, strong) NSArray<Student *> *students;
@end

@implementation ClassRoom
+ (NSDictionary<NSString *,id> *)modelContainerPropertyGenericClass {
    return @{@"students" : [Student class]};
}
@end

// 使用示例
NSString *jsonString = @"{\"students\": [{\"name\": \"Alice\", \"age\": 20}, {\"name\": \"Bob\", \"age\": 22}]}";
ClassRoom *classRoom = [ClassRoom yy_modelWithJSON:jsonString];
```


