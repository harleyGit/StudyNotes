- [**冲突处理**](#冲突处理)
- [**源码解析**](#源码解析)
- 参考资料
	- [JSONModel高级使用](https://www.jianshu.com/p/ac44a75d4f75)
	- [JSONModel的使用](https://www.jianshu.com/p/606f421bc8aa)
	- [JSONModel的使用](https://www.jianshu.com/p/606f421bc8aa)
	- [IOS-JSONModel使用](https://www.jianshu.com/p/ae05d9410d3c)
	- [iOS中JSONModel的使用](https://www.jianshu.com/p/3cce56f374b4)


<br/>

***
<br/>

> <h1 id="冲突处理">冲突处理</h1>

**`id、description冲突处理`**

- **方法一**

`json 字段展示`

![description 冲突](https://upload-images.jianshu.io/upload_images/2959789-b9eff7f5e785f6df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**正常解析，使用 po 打印得：**

![因为description在JOSNMondel源码中为一个方法，所以不能为一个字段](https://upload-images.jianshu.io/upload_images/2959789-a7a6a60f00689ec7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



`HGMyInformationModel.h 文件`

```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface HGMyInformationModel : HGMyModel

//用户高清头像
@property(nonatomic, copy) NSString *avatar_hd;
//用户大图头像180*180
@property(nonatomic, copy) NSString *avatar_large;
//性别
@property(nonatomic, copy) NSString *gender;
//用户个人描述
@property(nonatomic, copy) NSString *introduction;  //description

//......还有其他字段，不便都列举 
@end

NS_ASSUME_NONNULL_END
```

`HGMyInformationModel.m文件`

```
#import "HGMyInformationModel.h"


@implementation HGMyInformationModel

//initWithDictionary 这个方法在 JSONMondel 1.8.0 版本已弃用，建议使用initWithModelToJSONDictionary，但是这个方法对于关键字段id有用，对于descripton无用，解析为空。
+ (JSONKeyMapper *)keyMapper {
      return [[JSONKeyMapper alloc] initWithDictionary:@{@"description":@"introduction"}];
}

//设置所有的属性为可选(所有属性值可以为空),这里这段注释是不需要的，加上也无所谓
+(BOOL)propertyIsOptional:(NSString *)propertyName {
    return YES;
}

@end

```

![字段替换成功](https://upload-images.jianshu.io/upload_images/2959789-fbd04ec9d84169f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

- **方法二**

![id 关键字](https://upload-images.jianshu.io/upload_images/2959789-947b85e2801cd1ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Order_details.h`

![属性字段](https://upload-images.jianshu.io/upload_images/2959789-627f166d57511847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Order_details.m`

![新定义的字段处理](https://upload-images.jianshu.io/upload_images/2959789-da9eba3c925b8130.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`网络请求字段进行解析`

![调用方法进行解析](https://upload-images.jianshu.io/upload_images/2959789-b1e72ec761479343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








<br/>

***
<br/>


> <h1 id="源码解析">源码解析</h1>
[**‌源码解析**](https://blog.csdn.net/u014644610/article/details/80407267)

数据解析步骤：
- 先是判断传入的字典是否为空，如果为空返回为空的错误 
- 再判断传入的数据是否是字典类型，如果不是字典类型不正确的错误 
- 核心的代码，通过init方法初始化映射property，（核心代码，等会再下面解释） 
- 获取当前类的keyMapper 
- 检查映射结构是否能从我们传入的dict中找到对应的数据，如果不能找到，就返回nil，并且抛出错误 
- 根据传入的dict进行数据的赋值，如果赋值没有成功，就返回nil，并且抛出错误 
- 根据本地的错误来判断是否有错误，如果有错误，就返回nil 
- 返回self


```
-(id)initWithDictionary:(NSDictionary*)dict error:(NSError**)err
{
    //check for nil input
  // 第一步： 先是判断传入的字典是否为空，如果为空返回为空的错误
    if (!dict) {
        if (err) *err = [JSONModelError errorInputIsNil];
        return nil;
    }

    //invalid input, just create empty instance
    //第二步：再判断传入的数据是否是字典类型，如果不是字典类型不正确的错误
    if (![dict isKindOfClass:[NSDictionary class]]) {
        if (err) *err = [JSONModelError errorInvalidDataWithMessage:@"Attempt to initialize JSONModel object using initWithDictionary:error: but the dictionary parameter was not an 'NSDictionary'."];
        return nil;
    }

    //create a class instance
    //第三步：核心的代码，通过init方法初始化映射property
    self = [self init];
    if (!self) {

        //super init didn't succeed
        if (err) *err = [JSONModelError errorModelIsInvalid];
        return nil;
    }
    //第四步：获取当前类的keyMapper
    //key mapping
    JSONKeyMapper* keyMapper = [self __keyMapper];
    //第五步：检查映射结构是否能从我们传入的dict中找到对应的数据，如果不能找到，就返回nil，并且抛出错误
    //check incoming data structure
    if (![self __doesDictionary:dict matchModelWithKeyMapper:keyMapper error:err]) {
        return nil;
    }

    //import the data from a dictionary
    //第六步：根据传入的dict进行数据的赋值，如果赋值没有成功，就返回nil，并且抛出错误。
    if (![self __importDictionary:dict withKeyMapper:keyMapper validation:YES error:err]) {
        return nil;
    }
    //第七步：根据本地的错误来判断是否有错误，如果有错误，就返回nil，并且抛出错误。
    //run any custom model validation
    if (![self validate:err]) {
        return nil;
    }
    //第八步：返回self
    //model is valid! yay!
    return self;
}

```


<br/>


**第三步：核心代码的解析–通过init方法初始化映射property**


调用init方法，init方法中点用了“setup“方法。`–[self setup]。`

```
-(id)init
{
    self = [super init];
    if (self) {
        //do initial class setup
        [self __setup__];
    }
    return self;
}

```


<br/>

解析**“setup”**方法 
- 先是通过AssociateObject来判断是否进行过映射property的缓存，如果没有就使用“__inspectProperties”方法进行映射property的缓存
- –(后面我会解析一下“__inspectProperties”方法，介绍怎么进行映射property的缓存) 
- 获取当前类的keyMapper映射。 
- 判断一下，当前的keyMapper是否存在和是否进行映射过，如果没有进行映射就使用AssociateObject方法进行映射。 
- 进行AssociateObject映射。

```
-(void)__setup__
{
    //if first instance of this model, generate the property list
   //第一步： 先是通过AssociateObject来判断是否进行过映射property的缓存，如果没有就使用“__inspectProperties”方法进行映射property的缓存
    if (!objc_getAssociatedObject(self.class, &kClassPropertiesKey)) {
    //进行映射property的缓存
        [self __inspectProperties];
    }

    //if there's a custom key mapper, store it in the associated object
    //第二步：获取keyMapper
    id mapper = [[self class] keyMapper];
    //第三步：判断一下，当前的keyMapper是否存在和是否进行映射过，如果没有进行映射就使用AssociateObject方法进行映射
    if ( mapper && !objc_getAssociatedObject(self.class, &kMapperObjectKey) ) {
    //第四步：进行AssociateObject映射
        objc_setAssociatedObject(
                                 self.class,
                                 &kMapperObjectKey,
                                 mapper,
                                 OBJC_ASSOCIATION_RETAIN // This is atomic
                                 );
    }
}
```


<br/>

**`__inspectProperties`**方法解析，介绍怎么进行映射property的缓存 

注意：NSScanner是用于在字符串中扫描指定的字符，特别是把它们翻译/转换为数字和别的字符串。可以在创建NSScaner时指定它的string属性，然后scanner会按照你的要求从头到尾地扫描这个字符串的每个字符。 

```
1、先是获取当前class的property列表和个数 
2、然后再遍历这些property 
3、把我们的property通过一个局部变量进行赋值–JSONModelClassProperty，这个是JSONModel提供的类，来解析每个property。 
4、获取property的名称给当前这个局部变量 
5、获取这个property的属性 
6、判断这个property的属性值里面是否包含”Tc,”，如果包含就设置structName为BOOL 
7、扫描property属性 
8、设置property的类型 
9、判断并设置property的是否是可变的 
10、判断property的是否我们允许的json类型 
11、解析protocol的string 
12、检查property是否为structure 
13、判断property是不是Optional 
14、判断property是不是Ignored 
15、判断property是不是只读属性 
16、通过kvc去设置相应的值 
17、使用AssociateObject进行缓存
```

```
-(void)__inspectProperties
{
    //JMLog(@"Inspect class: %@", [self class]);

    NSMutableDictionary* propertyIndex = [NSMutableDictionary dictionary];

    //temp variables for the loops
    Class class = [self class];
    NSScanner* scanner = nil;
    NSString* propertyType = nil;

    // inspect inherited properties up to the JSONModel class
    while (class != [JSONModel class]) {
        //JMLog(@"inspecting: %@", NSStringFromClass(class));
       //第一步：先是获取当前class的property列表和个数
        unsigned int propertyCount;
        objc_property_t *properties = class_copyPropertyList(class, &propertyCount);
        //第二步：遍历property
        //loop over the class properties
        for (unsigned int i = 0; i < propertyCount; i++) {
//第三步：创建一个解析和判断每个property的局部变量JSONModelClassProperty
            JSONModelClassProperty* p = [[JSONModelClassProperty alloc] init];

            //get property name
            objc_property_t property = properties[i];
            const char *propertyName = property_getName(property);
            //第四步：获取property的名称给当前这个局部变量
            p.name = @(propertyName);

            //JMLog(@"property: %@", p.name);

            //get property attributes
            //第五步：获取这个property的属性
            const char *attrs = property_getAttributes(property);
            NSString* propertyAttributes = @(attrs);
            //第六步：判断这个property的属性值里面是否包含"Tc,"，如果包含就设置structName为BOOL
            if ([propertyAttributes hasPrefix:@"Tc,"]) {
                //mask BOOLs as structs so they can have custom convertors
                p.structName = @"BOOL";
            }
            //第七步：扫描property属性
            scanner = [NSScanner scannerWithString: propertyAttributes];

            //JMLog(@"attr: %@", [NSString stringWithCString:attrs encoding:NSUTF8StringEncoding]);
            [scanner scanUpToString:@"T" intoString: nil];
            [scanner scanString:@"T" intoString:nil];

            //check if the property is an instance of a class
            //解析一个类，检查属性是否为类的实例
            if ([scanner scanString:@"@\"" intoString: &propertyType]) {

                [scanner scanUpToCharactersFromSet:[NSCharacterSet characterSetWithCharactersInString:@"\"<"]
                                        intoString:&propertyType];

                //JMLog(@"type: %@", propertyClassName);
                //第八步：设置property的类型
                p.type = NSClassFromString(propertyType);
                //第九步：判断并设置property的是否是可变的
                p.isMutable = ([propertyType rangeOfString:@"Mutable"].location != NSNotFound);
                //第十步：判断property的是否我们允许的json类型
                p.isStandardJSONType = [allowedJSONTypes containsObject:p.type];

                //read through the property protocols

            //第十一步：解析protocol的string
                while ([scanner scanString:@"<" intoString:NULL]) {

                    NSString* protocolName = nil;

                    [scanner scanUpToString:@">" intoString: &protocolName];
                    if ([protocolName isEqualToString:@"Optional"]) {
                        p.isOptional = YES;
                    } else if([protocolName isEqualToString:@"Index"]) {
                        p.isIndex = YES;
                        objc_setAssociatedObject(
                                                 self.class,
                                                 &kIndexPropertyNameKey,
                                                 p.name,
                                                 OBJC_ASSOCIATION_RETAIN // This is atomic
                                                 );
                    } else if([protocolName isEqualToString:@"ConvertOnDemand"]) {
                        p.convertsOnDemand = YES;
                    } else if([protocolName isEqualToString:@"Ignore"]) {
                        p = nil;
                    } else if ([self propertyIsReadOnly:p.name]) {
                        p = nil;
                    } else {
                        p.protocol = protocolName;
                    }

                    [scanner scanString:@">" intoString:NULL];
                }

            }
            //check if the property is a structure
            //第十二步：检查property是否为structure
            else if ([scanner scanString:@"{" intoString: &propertyType]) {
                [scanner scanCharactersFromSet:[NSCharacterSet alphanumericCharacterSet]
                                    intoString:&propertyType];

                p.isStandardJSONType = NO;
                p.structName = propertyType;

            }
            //the property must be a primitive
            else {

                //the property contains a primitive data type
                [scanner scanUpToCharactersFromSet:[NSCharacterSet characterSetWithCharactersInString:@","]
                                        intoString:&propertyType];

                //get the full name of the primitive type
                propertyType = valueTransformer.primitivesNames[propertyType];

                if (![allowedPrimitiveTypes containsObject:propertyType]) {

                    //type not allowed - programmer mistaked -> exception
                    @throw [NSException exceptionWithName:@"JSONModelProperty type not allowed"
                                                   reason:[NSString stringWithFormat:@"Property type of %@.%@ is not supported by JSONModel.", self.class, p.name]
                                                 userInfo:nil];
                }

            }

            NSString *nsPropertyName = @(propertyName);
            //第十三步：判断property是不是Optional
            if([[self class] propertyIsOptional:nsPropertyName]){
                p.isOptional = YES;
            }
                        //第十四步：判断property是不是Ignored
            if([[self class] propertyIsIgnored:nsPropertyName]){
                p = nil;
            }
                                    //第十五步：判断property是不是只读属性

            if([self propertyIsReadOnly:nsPropertyName]) {
                p = nil;
            }

            //add the property object to the temp index
            //第十六步：通过kvc去设置相应的值
            if (p) {
                [propertyIndex setValue:p forKey:p.name];
            }
        }

        free(properties);

        //ascend to the super of the class
        //(will do that until it reaches the root class - JSONModel)
        class = [class superclass];
    }

    //finally store the property index in the static property index
    //第十七步：使用AssociateObject进行缓存
    objc_setAssociatedObject(
                             self.class,
                             &kClassPropertiesKey,
                             [propertyIndex copy],
                             OBJC_ASSOCIATION_RETAIN // This is atomic
                             );
}

```



