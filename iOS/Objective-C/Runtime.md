

<br/>

- [**简介**](#简介)
	- [**基本概念**](#基本概念)
- [**类**](#类)
	- [属性列表](#属性列表)
	- [objc_object源码](#objc_object源码)
	- [objc_class结构体](#objc_object源码)
	- [Class(类)](#Class(类))
	- [Object(对象)](#Object(对象))
	- [Meta Class(元类)](#MetaClass(元类))
	- [Method(方法)](#Method(方法))
	- [AssociatedObject关联对象](#AssociatedObject关联对象)
- [**消息转发机制**](#消息转发机制)
	- [消息发送](#消息发送)
- **资料**
	- [class_method 参数](https://www.jianshu.com/p/e4237de0aedb)      
	- 	[Runtime 详解 基础知识](https://www.jianshu.com/p/633e5d8386a8) 
	- 	**[深入了解Category](https://tech.meituan.com/2015/03/03/diveintocategory.html)**
	- 	[iOS runtime——函数/使用方法/使用场景/示例](https://blog.csdn.net/potato512/article/details/51106645)
	- 	**[Runtime源码分析](https://juejin.cn/post/6844903952039821320)**
	- [文章介绍](https://juejin.cn/user/3913917126673047/posts)


<br/>

***
<br/> 

> <h1 id='简介'>简介</h1>

&emsp;  Runtime被称为 `iOS 开发中的黑魔法`，是一套C 语言的API.

&emsp;  写好代码后，command+R 运行，这时代码在内存中运行。Runtime就是动态修改在内存中的对象，把类的方法和属性在内存中进行动态的改变。

&emsp;  C语言函数在调用编译的时候就会决定调用哪个函数，而OC是一种动态语言，它会尽可能把代码从编译链接推迟到运行时，这就是OC运行时的多态。


![z23](./../../Pictures/z23.jpeg)



<br/>
<br/> 

> <h2 id='基本概念'>基本概念</h2>

先来做道[题](https://halfrost.com/objc_runtime_isa_class/)

<br/>

```
Obj *obj = [Obj new];
```

- `实例对象`：通常类的实例化的对象，比如：obj 就是一个实例对象；
- `类对象`：其实`类也是一个对象`，比如: Obj 其实也是一个对象；
- `元类`：其实就是 `类对象的isa`指向的类；
- `isa指针`：在objective-c语言的内部，每一个对象都有一个isa指针，指向该指针的类。每一个类描述了它的实例对象的特点，包括成员变量列表，成员函数列表。每一个实例对象都可以接收消息，而对象接收消息列表保存在它所对应的类中。比如：

```
NSObject *obj=[[NSObject alloc] init];

//调用方法，其实是给对象发送消息，在编译时这句话会翻译成一个C的函数调用
//使用这个函数的需要引入头文件 #import <objc/message.h>
objc_msgSend(objc_msgSend([NSObject class],@selector(alloc)),@selector(init));

```




<br/>
<br/>



&emsp;  Xcode中的NSObject.h和objc.h，我们可以看到，NSObject就是一个包含isa指针的结构体，按照面向对象的设计原则，所有的事物都应该是对象，所以严格的说OC并不是完全面向对象的（应为含有int double 类型的变量）。在OC语言中，每一个类实际上也是一个对象。每一个类也有一个isa指针。每一个类也可以接收消息，例如代码[NSObject alloc],就是向NSObject这个类发送名为 “alloc” 的消息。

> &emsp;  在OC中，因为类也是一个对象，所以也必须是另外一个类的实例，这个类就是元类(metaclass)。`元类保存了类方法的列表`。**当一个类方法被调用的时候，元类会首先查找他本身是否有该方法的实现，如果没有，则元类会向他的父类查找方法，这样就可以一直找到继承链的头。**



<br/>

***
<br/>

> <h1 id='类'>类</h1>


<br/>

```
//用于获取对象的isa指针指向的对象
object_getClass(id _Nullable obj) 

```

<br/>
<br/>


> <h2 id='属性列表'>属性列表</h2>

**property_getAttributes(objc_property_t _Nonnull property)**   

获取属性的真实类型

```
const char *attrs = property_getAttributes(property);

//Printing description of attrs:
(const char *) attrs = 0x0000000108cc3c26 "T@\"NSMutableArray<ShowNewsModel>\",&,N,V_list1"
```

[property_getAttributes()](https://www.jianshu.com/p/cefa1da5e775)



<br/>
<br/>


> <h2 id='objc_object源码'>objc_object源码</h2>



点击 **#import <objc/objc.h>** ，可以发现class与object在Objective-C的定义：

[调试Runtime源码](https://zhuanlan.zhihu.com/p/27786725),[ **Runtime源码**](https://github.com/RetVal/objc-runtime)

```  
/// 代表一个被标志的类
//Class是一个objc_class结构类型的指针
typedef struct objc_class *Class;

/// 代表一个实例类
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

typedef struct objc_object *id;

```


<br/>
<br/>


> <h2 id='objc_class结构体'>objc_class结构体</h2>

点击objc_class这个结构体，可以看到：

```

struct objc_class {
    // objc_class 结构体的实例指针
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY; 

#if !__OBJC2__
    // 指向父类的指针
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    // 类的名字 
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    // 类的版本信息，默认为 0
    long version                                             OBJC2_UNAVAILABLE;
    // 类的信息，供运行期使用的一些位标识  
    long info                                                OBJC2_UNAVAILABLE;
    // 该类的实例变量大小;
    long instance_size                                       OBJC2_UNAVAILABLE;
    // 该类的实例变量列表
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    // 方法定义的列表
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
     // 方法缓存
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    // 遵守的协议列表
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

```

<br/>
<br/>


> <h2 id='Class(类)'>Class(类)</h2>


> &emsp;  `struct objc_classs` 结构体里存放的数据称为`元数据(metadata)`，结构体内包含了isa、super_class(指向父类的指针)、属性(name[类的名字]、version[版本]、info、instance_size[实例大小])、ivars(实例变量列表)、methodLists(方法列表)、cache(缓存)、protocols(遵守的协议列表)。

&emsp;  这些信息就足够创建一个实例了，该结构体的第一个成员变量也是isa指针，这就说明了`Class`本身其实也是一个对象，我们称之为`类对象`，`类对象`在编译期产生用于创建`实例对象`，是`单例`。


<br/>
<br/>


> <h2 id='Object(对象)'>Object(对象)</h2>


&emsp;  这里的 id 被定义为一个指向 objc_object 结构体 的指针。从中可以看出 objc_object 结构体 只包含一个 Class 类型的 isa 指针。

&emsp; 换句话说，一个 Object（对象）唯一保存的就是它所属 Class（类） 的地址。当我们对一个对象，进行方法调用时，比如 [receiver selector];，它会通过 objc_object 结构体的 isa 指针 去找对应的 objc_class 结构体，然后在 objc_class 结构体 的 methodLists（方法列表） 中找到我们调用的方法，然后执行。


<br/>
<br/>


> <h2 id='MetaClass(元类)'>Meta Class(元类)</h2>



&emsp; 对象（objc_object 结构体） 的 isa 指针 指向的是对应的 类对象（objc_class 结构体）。那么 类对象（objc_class 结构体）的 isa 指针 又指向什么呢？

&emsp; objc_class 结构体 的 isa 指针 实际上指向的的是 类对象 自身的 Meta Class（元类）。

&emsp; Meta Class（元类） 就是一个类对象所属的 类。一个对象所属的类叫做 类对象，而一个类对象所属的类就叫做 元类。

 

<br/>
<br/>


> <h2 id='Method(方法)'>Method(方法)</h2>




```
struct objc_method {
    SEL _Nonnull method_name   //方法的名字
    char * _Nullable method_types    //参数的类型
    IMP _Nonnull method_imp	//就是函数的地址
} 

```



<br/>

&emsp;  `类对象`中的`元数据`存储的都是如何创建一个实例的相关信息，那么`类对象`和`类方法`应该从哪里创建呢？就是从isa指针指向的结构体创建，`类对象`的isa指针指向的我们称之为`元类(metaclass)`，元类中保存了创建类对象以及类方法所需的所有信息，因此如简单字符串创建整个结构应该如下图所示:

类之间的关系:

![ios_oc2_21.png](./../../Pictures/ios_oc2_21.png)




&emsp;  通过上图我们可以清晰的看出来一个`实例对象`也就是`struct objc_object结构体`它的isa指针指向`类对象`，`类对象`的isa指针指向了`元类`，`super_class指针`指向了父类的`类对象`，而元类的`super_class指针`指向了父类的`元类`，那元类的isa指针又指向了什么？为了更清晰的表达直接使用一个大神画的图。

![ios_oc2_22.png](./../../Pictures/ios_oc2_22.png)



由上图我们可以得到：
-  整个体系构成了一个自闭环，如果是从NSObject中继承而来的上图中的Root class就是NSObject。至此，整个实例、类对象、元类的概念也就讲清了，接下来我们在代码中看看这些概念该怎么应用。

-  实例对象的isa指向类对象，当调用对象方法，通过实例对象的isa 找到类对象,最终找到对对象方法进行调用;
-  类对象的isa指向元类，调用类方法，通过类对象中的isa找到元类，最终找到元类中的类方法进行调用;
-  当子类的对象要调用`父类的对象方法`，先通过子类的isa找到父类的class 然后通过superClass找到,父类的class 最后找到消息进行调用。

```

@interface Person : NSObject

@property(nonatomic, assign) NSInteger age;
@property(nonatomic, copy)  NSString *name;

@end



//c1是通过一个实例对象获取的Class，实例对象可以获取到其类对象;
//类名作为消息的接受者时代表的是类对象，因此类对象获取Class得到的是其本身，同时也印证了类对象是一个单例的想法。
Person *person = [[Person alloc] init];
NSLog(@"person 对象：%@",person);
//person 对象：<Person: 0x600000c4f260>
    
Class c1 = [person class];
Class c2 = [Person class];
//输出1
NSLog(@"%d", c1 == c2);
    

//class_isMetaClass用于判断Class对象是否为元类
//object_getClass用于获取对象的isa指针指向的对象    
 NSLog(@"2.  %d", [person class] == object_getClass(person));
//输出2.  1
NSLog(@"3.  %d", class_isMetaClass(object_getClass(person)));
//输出3.  0
NSLog(@"4.  %d", class_isMetaClass(object_getClass([Person class])));
//输出4.  1
NSLog(@"5.  %d", object_getClass(person) == object_getClass([Person class]));
//输出5.  0

NSLog(@"6. %@",object_getClass([person class]));
//6. Person
NSLog(@"7. %@",[Person class]);
//7. Person

NSLog(@"8. %@", object_getClass([Person class]));
//8. Person
NSLog(@"9. %@", object_getClass(object_getClass([Person class])));
//9. NSObject
NSLog(@"10. %@", object_getClass(object_getClass(object_getClass([Person class]))));
//10. NSObject

```

`总结：`

- **通过`class`方法获取的Class要分两种情况：**
	- 当obj为实例对象时, 调用[obj class]时，class 就是实例方法，返回的就是obj实例对象isa指针指向的类对象
	- 当[Obj class]调用时，Obj 为类对象（包括元类和根类以及根元类）时，调用的是类方法：+ (Class)class，返回的结果为其本身。

- **使用 object_getClass 方法分析：**
-  当参数obj为Object实例对象
	-  object_getClass(obj)与[obj class]输出结果一直，均获得isa指针，即指向类对象的指针。

- 当参数obj为Class类对象
	- object_getClass(obj)返回类对象中的isa指针，即指向元类对象的指针；[obj class]返回的则是其本身。

- 当参数obj为Metaclass类对象
	- object_getClass(obj)返回元类对象中的isa指针，因为元类对象的isa指针指向根类，所有返回的是根类对象的地址指针；[obj class]返回的则是其本身。

- obj为Rootclass类对象
	- object_getClass(obj)返回根类对象中的isa指针，因为根类对象的isa指针指向Rootclass‘s metaclass(根元类)，即返回的是根元类的地址指针；[obj class]返回的则是其本身。




```
//获取 class为NSObject的元类对象
Class class = object_getClass([NSObject class]);
```

<br/>

建立一个Person 类：

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Person : NSObject

@property(nonatomic, assign) NSInteger age;

@end

NS_ASSUME_NONNULL_END

```


控制台 person 实例对象包含的内容:


![ios_oc2_23.png](./../../Pictures/ios_oc2_23.png)


从上图可以看到person实例对象包含一个isa指针、属性变量。
每个类在内存中有且只有一个类对象，所以每个类在内存中也有且只有一个元类对象;

当你给对象发送消息时，消息是在寻找这个对象的类的方法列表;
当你给类发消息时，消息是在寻找这个类的元类的方法列表。



<br/>
<br/>



> <h2 id='AssociatedObject关联对象'>AssociatedObject关联对象</h2>


简介：关联是指把两个对象相互关联起来,使得其中的一个对象作为另外一个对象的一部分。一般用在分类中，因为在分类中是不可以再次申明定义一个属性变量的，这时可以用关联属性。

```
objc_getAssociatedObject(id _Nonnull object, const void * _Nonnull key);

objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssociationPolicy policy);
```

[AssociatedObject 完全解析](https://www.jianshu.com/p/79479a09a8c0)

[基本用法](https://www.jianshu.com/p/6f1343c7be26)

[demo](https://www.jianshu.com/p/52a28d59ef10)

[Runtime探索](https://www.jianshu.com/u/2de707c93dc4)








<br/>

***
<br/>

> <h1 id='消息转发机制'>消息转发机制</h1>

![ios_oc2_26.png](./../../Pictures/ios_oc2_26.png)


![ios_oc2_27.png](./../../Pictures/ios_oc2_27.png)


![ios_oc2_28.png](./../../Pictures/ios_oc2_28.png)


&emsp;   原理：Objective-C 语言 中，对象方法调用都是类似 `[receiver selector]; `的形式，其本质就是让对象在运行时发送消息的过程。


<br/>


**`编译阶段：`**`[receiver selector];` 方法被编译器转换为:

```
objc_msgSend(receiver，selector) （不带参数）
objc_msgSend(recevier，selector，org1，org2，…)（带参数）
```

<br/>

**`运行时阶段：`**消息接受者 recever 寻找对应的 selector。

> &emsp;  通过 recevier 的 isa 指针 找到 recevier 的 Class（类）；

> &emsp;  在 Class（类） 的 cache（方法缓存） 的散列表中寻找对应的 IMP（方法实现）；

> &emsp;  如果在 cache（方法缓存） 中没有找到对应的 IMP（方法实现） 的话，就继续在 Class（类） 的 method list（方法列表） 中找对应的 selector，如果找到，填充到 cache（方法缓存） 中，并返回 selector；

> &emsp;  如果在 Class（类） 中没有找到这个 selector，就继续在它的 superClass（父类）中寻找；
一旦找到对应的 selector，直接执行 recever 对应 selector 方法实现的 IMP（方法实现）。

> &emsp;  若找不到对应的 selector，消息被转发或者临时向 recever 添加这个 selector 对应的实现方法，否则就会发生崩溃。

消息转发示意图:

![ios_oc2_24.png](./../../Pictures/ios_oc2_24.png)



<br/>

创建一个MessageSend类

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface MessageSend : NSObject

- (void)sendMessage:(NSString *)message;

@end

NS_ASSUME_NONNULL_END

```

调用：


```
- (void)test {
    MessageSend *ms = [MessageSend new];
    
    [ms sendMessage:@"Hello"];
}

```

crash 错误：

```
2019-11-17 11:20:08.972291+0800 HGSWB[49471:2551396] -[MessageSend sendMessage:]: unrecognized selector sent to instance 0x600000e042d0

2019-11-17 11:20:09.047606+0800 HGSWB[49471:2551396] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[MessageSend sendMessage:]: unrecognized selector sent to instance 0x600000e042d0'
```

<br/>

**`①消息动态解析`**

在MessageSend.m 添加方法

```
#import "MessageSend.h"

@implementation MessageSend

void dynamicMethodIMP11(id self, SEL _cmd, NSString *msg) {
    NSLog(@"%@的方法转发到Here: %@ ----> %@", self, NSStringFromSelector(_cmd), msg);
}


//系统方法
+ (BOOL)resolveInstanceMethod:(SEL)sel {    //实例
    
   /*@param cls    给哪个类添加方法
    *@param name   方法列表中的名字，方法编号
    *@param imp    方法实现
    *@param        参数
    * @return      如果添加方法成功返回 YES，否则返回 NO
    */
    //lass_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types)
    if (sel == @selector(sendMessage:)) {
        
        return class_addMethod([self class], sel, (IMP)dynamicMethodIMP11, "v@:");
    }
    
    return [super resolveInstanceMethod:sel];
    
}

@end
```

打印：

```
2019-11-17 11:38:18.171651+0800 HGSWB[50214:2594517] <MessageSend: 0x6000002517a0>的方法转发到Here: sendMessage: ----> Hello
```

查看参数类型，看[这里](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100)

<br/>

**`②消息接受者重定向`**

&emsp;  若上步中 `+resolveInstanceMethod:` 或者` +resolveClassMethod:` 没有添加其他函数实现，运行时就会进行下一步：消息接受者重定向。

&emsp;  若当前对象实现了 `-forwardingTargetForSelector:`，Runtime 就会调用这个方法，允许我们将消息的接受者转发给其他对象。


创建一个 SubMessageSend 类

```
//.h 文件
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface SubMessageSend : NSObject

@end

NS_ASSUME_NONNULL_END



//.m文件
#import "SubMessageSend.h"

@implementation SubMessageSend

- (void)sendMessage:(NSString *)message {
    NSLog(@"SubMessageSend 的 sendMessage 方法");
}

@end
```

MessageSend.m 文件

```
#import "MessageSend.h"
#import "SubMessageSend.h"

@implementation MessageSend

- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSString *methodName = NSStringFromSelector(aSelector);
    if ([methodName isEqualToString:@"sendMessage:"]) {
        return [[SubMessageSend alloc] init];
    }
    
    return [super forwardingTargetForSelector:aSelector];
}
@end

```

打印：
```
2019-11-17 11:51:28.940038+0800 HGSWB[50835:2628501] SubMessageSend 的 sendMessage 方法
```


<br/>

**`③消息重定向`**

&emsp;  如果经过消息动态解析、消息接受者重定向，Runtime 系统还是找不到相应的方法实现而无法响应消息，Runtime 系统会利用 `methodSignatureForSelector: `方法获取函数的参数和返回值类型。

&emsp;  如果 `methodSignatureForSelector: `返回了一个 `NSMethodSignature` 对象（函数签名），Runtime 系统就会创建一个 `NSInvocation` 对象，并通过 `forwardInvocation: `消息通知当前对象，给予此次消息发送最后一次寻找 IMP 的机会。
如果` -methodSignatureForSelector:` 返回 nil。则 Runtime 系统会发出 `-doesNotRecognizeSelector: `消息，程序也就崩溃了。

```
#import "MessageSend.h"
#import "SubMessageSend.h"

@implementation MessageSend


/// 消息重定向(系统方法)
/// @param aSelector 方法名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    
    //获取函数的参数和返回值类型，返回签名
    if ([NSStringFromSelector(aSelector) isEqualToString:@"sendMessage:"]) {
        //调用forwardInvocation:，如果返回nil调用doesNotRecognizeSelector:
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    
    return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    
    //从 anInvocation 中获取消息
    SEL sel = anInvocation.selector;
    
    SubMessageSend *sms = [[SubMessageSend alloc] init];
    
    if ([sms respondsToSelector:sel]) { //判断 SubMessageSend 对象方法是否可以响应 sel
        [anInvocation invokeWithTarget:sms];    //若可以响应，则将消息转发给其他对象处理
    }else {
        [self doesNotRecognizeSelector:sel];    // 若仍然无法响应，则报错：找不到响应方法
    }
}

- (void)doesNotRecognizeSelector:(SEL)aSelector {
    NSLog(@"doesNotRecognizeSelector 不要逼我了，我没办法了，我要 Crash 了！！！");
}

@end
```

&emsp; 因为 `SubMessageSend` 类中有 `sendMessage:` 可以相应，所以可以打印：

```
2019-11-17 12:41:04.617546+0800 HGSWB[52721:2786753] SubMessageSend 的 sendMessage 方法
```

当我在控制台中使用命令 `po anInvocation.methodSignature` 时，打印如下：

![ios_oc2_25.png](./../../Pictures/ios_oc2_25.png)


&emsp;  当我把 `SubMessageSend` 类中 `sendMessage:` 方法注掉后会打印：
`2019-11-17 12:47:29.881606+0800 HGSWB[53142:2878060] doesNotRecognizeSelector 不要逼我了，我没办法了，我要 Crash 了！！！
`

<br/>

消息发送以及转发机制总结：

调用 [receiver selector]; 后，进行的流程：

- `编译阶段：[receiver selector];` 方法被编译器转换为:

  ```
  objc_msgSend(receiver，selector) （不带参数）
  objc_msgSend(recevier，selector，org1，org2，…)（带参数）
  ```

-  `运行时阶段：`消息接受者 recever 寻找对应的 selector。
     
     &emsp;   通过 recevier 的 isa 指针 找到 recevier 的 class（类）；
     
    &emsp;   在 Class（类） 的 cache（方法缓存） 的散列表中寻找对应的 IMP（方法实现）；
    
    &emsp;   如果在 cache（方法缓存） 中没有找到对应的 IMP（方法实现） 的话，就继续在 Class（类） 的 method list（方法列表） 中找对应的 selector，如果找到，填充到 cache（方法缓存） 中，并返回 selector；
    
    &emsp;   如果在 class（类） 中没有找到这个 selector，就继续在它的 superclass（父类）中寻找；
    
    &emsp;   一旦找到对应的 selector，直接执行 recever 对应 selector 方法实现的 IMP（方法实现）。
    
    &emsp;   若找不到对应的 selector，Runtime 系统进入消息转发机制。

- `运行时消息转发阶段`

   &emsp; ` 动态解析：`通过重写 `+resolveInstanceMethod:` 或者 `+resolveClassMethod:`方法，利用 `class_addMethod` 方法添加其他函数实现；
   
 &emsp; `消息接受者重定向：`如果上一步添加其他函数实现，可在当前对象中利用 `-forwardingTargetForSelector:` 方法将消息的接受者转发给其他对象；
 &emsp; `消息重定向：`如果上一步没有返回值为 nil，则利用 `-methodSignatureForSelector:`方法获取函数的参数和返回值类型。
 
 &emsp; 如果 `-methodSignatureForSelector: `返回了一个 `NSMethodSignature` 对象（函数签名），Runtime 系统就会创建一个 `NSInvocation 对象`，并通过 `-forwardInvocation:` 消息通知当前对象，给予此次消息发送最后一次寻找 IMP 的机会。
 
  &emsp; 如果 `-methodSignatureForSelector:` 返回 nil。则 Runtime 系统会发出 `-doesNotRecognizeSelector: `消息，程序也就崩溃了。



<br/>
<br/>

> <h2 id='消息发送'>消息发送</h2>

&emsp;  执行一个方法，有些语言，编译器会执行一些额外的优化和错误检查，因为调用关系很直接也很明显。但对于消息分发来说，就不那么明显了。在发消息前不必知道某个对象是否能够处理消息。你把消息发给它，它可能会处理，也可能转给其他的 Object 来处理。一个消息不必对应一个方法，一个对象可能实现一个方法来处理多条消息。

&emsp;  在Objective-C:中，消息是通过objc_msgSend()这个 runtime 方法及相近的方法来实现的。这个方法需要一个target，selector，还有一些参数。理论上来说，编译器只是把消息分发变成objc_msgSend来执行。比如下面这两行代码是等价的:

```
[array insertObject:foo atIndex:5];

objc_msgSend(array, @selector(insertObject:atIndex:), foo, 5);
```



<br/>

***
<br/>


                                   
