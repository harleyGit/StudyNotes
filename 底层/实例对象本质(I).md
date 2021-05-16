

- **分类**
- **isa指针的指向**



<br/>


***
<br/>



># 分类
OC对象分为三种：
>- 实例对象(instance对象);
>- 类对象(class对象);
>- 元类对象（meta-class对象）

<br/>

**`实例对象`**

&emsp;  实例对象（instance对象）就是通过类的alloc出来的对象，每次调用alloc都会产生新的实例对象。例如：

```
NSObjcet *obj1 = [[NSObject alloc] init];
NSObjcet *obj2 = [[NSObject alloc] init];
```

&emsp;  obj1和obj2都是NSObject的实例对象，但是它们是不同的两个实例对象，分别占用两块不同的内存地址。

&emsp;  实例对象在内存中存储的信息包括：
- isa指针
- 其他成员变量

![实例对象存储的信息](https://upload-images.jianshu.io/upload_images/2959789-96d7237f1137f373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

**`类对象(class对象)`**

&emsp;  `类对象（class对象）`就是通过class方法或者runtime的object_getClass方法得到的class对象。

&emsp;  `注意：`class 方法只是获取类，并不能获取真正获取其类对象。在这里因为下面的obj1的类就是NSObject所以其类对象和类是一样的。若换成其他的结果可能不一样。

```
Class objClass1 = [obj1 class];
Class objClass2 = [obj2 class];
Class objClass3 = [NSObject class];

// runtime方法
Class objClass4 = object_getClass(obj1);
Class objClass5 = object_getClass(obj2);

NSLog(@"objClass1= %@,\n objClass2= %@,\n objClass3= %@,\n objClass4= %@,\n objClass5= %@,\n ", obj1, obj2, objClass1, objClass2, objClass3, objClass4, objClass5);
```
打印：

 `objClass1= NSObject,`

 `objClass2= NSObject,`
 
 `objClass3= NSObject,`
 
 `objClass4= NSObject,`
 
 ` objClass5= NSObject,`

&emsp;  objClass1-objClass5都是NSObject的类对象（class对象），且它们是同一个对象。

>&emsp;  **`每个类在内存中有且只有一个class对象`**

类对象在内存中存储的信息包括：
- `isa`指针
- `superClass`指针
- 类的`属性`信息（`@property`），类的成员变量信息（`ivar`）
- 类的`对象方法`信息（`instance method`），类的`协议`信息（`protocol`）

![类对象存储图 <br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/principle0.png)


<br/>

**`元类对象（meta-class对象）`**

&emsp;  **`元类对象（meta-class对象）`**就是通过RunTime的`object_getClass`方法得到的对象

```
//通过RunTIme的API获得元类对象
Class objectMetaClass = object_getClass([NSObject class]);
```

objectMetaClass就是NSObject的元类对象

> **`每个类在内存中有且只有一个元类对象`**

元类对象和类对象的内存结构是一样的，但是用途不一样，元类对象在内存中存储的信息包括：
- `isa`指针
- `superClass`指针
- 类的`类方法`信息（`class method`）

![元类对象存储信息 <br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/principle1.png)

以上我们了解了`实例对象`、`类对象`和`元类对象`的含义以及包含的内容，那么它们当中的`isa`指针和`superClass`指针分别指向哪里呢?


<br/>

***
<br/>

># isa指针的指向

isa指针指向用一张示意图来简单概括一下：

![isa 指针指向图](https://upload-images.jianshu.io/upload_images/1760191-9f0d589ac77c87b1.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

&emsp;  实例对象（instance对象）的`isa`指针指向`class`。当调用对象方法时，通过实例对象的`isa`找到`class`，最后找到对象方法的实现进行调用。

&emsp;  类对象（class对象）的`isa`指针指向meta-class。当调用类方法时，通过类对象的`isa`找到meta-class，最后找到类方法的实现进行调用。



<br/>

***
<br/>

># class的superClass指针的指向

类(class)的superClass指针指向用一张示意图来简单概括一下：

![类的superClass指针指向图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/principle2.jpeg)

&emsp;  图中举例Student继承自Person，Person继承自NSObject。

&emsp;  当Student的实例对象要调用父类Person的对象方法时，会先通过`isa`找到Student的`class`，然后通过`class`中的superClass找到父类Person的`class`，最后找到对象方法的实现进行调用。


<br/>

***
<br/>

># meta-class对象的superClass指针指向

![image](https://upload-images.jianshu.io/upload_images/1760191-cf604bd592f17c58.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

&emsp;  同上，当Student的class要调用Person的类方法时，会先通过isa找到Student的meta-class，然后通过superClass找到Person的meta-class，最后找到类方法的实现进行调用。

这里当然要提一下非常经典的isa指向图，做进一步的总结：

![image](https://upload-images.jianshu.io/upload_images/1760191-16c6237e389628a0.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/937)

> 1、instance的isa指向class
> 
> 2、class的isa指向meta-class
> 
> 3、meta-class的isa指向基类的meta-class，基类的isa指向自己
> 
> 4、class的superClass指向父类的class，如果没有父类，则superClass指针为nil
> 
> 5、meta-class的superClass指向父类的meta-class，基类的meta-class的superClass指向基类的class
> 
> 6、instance调用对象方法的轨迹：通过isa找到class，方法不存在，就通过superclass逐层到父类里找，有就实现，如果找到基类仍没有找到，就会抛出`unrecognized selector sent to instance`异常
> 
> 7、class调用类方法的轨迹：通过isa找到meta-class，方法不存在，就通过superClass逐层父类里找。



**补充：**

相信很多人在查看源码或者看一些底层博客的时候，经常会看到下面一段代码，来讲述class的内部结构：

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

这段源码其实讲述的也是class内部结构，包含成员变量列表、方法列表、方法缓存以及协议列表。细心的人可能会发现，这段代码里面是有`if`判断条件的:

```
#if !__OBJC2__

```

判断条件是非OC2.0版本，也就是说在OC2.0之前的版本中，class底层的结构体中包含上面代码所讲述的，但我们现在所用的最新版肯定是OC2.0版本了，所以这段代码就不再使用了。

新的class底层的objc_class结构体：

![objc_class 结构体](https://upload-images.jianshu.io/upload_images/2959789-cec2b3026ae79e5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  可以看到结构体中只包含`isa`、`superclass`、`cache`和`bits`。而`bits`经过`& FAST_DATA_MASK`之后，会得到`struct class_rw_t`这样一个结构体：

![struct class_rw_t](https://upload-images.jianshu.io/upload_images/2959789-da044d73461d618d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> rw代表readwrite，可读可写
> t代表table，列表

`struct class_rw_t`结构体中就包含了方法列表、属性列表以及协议列表，这些都是可读可写的。其中还包含一个`struct class_ro_t`的结构体：

![struct class_ro_t](https://upload-images.jianshu.io/upload_images/2959789-c019d88138c9f97e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> ro代表readonly，只读

`struct class_ro_t`的结构体中包含了instance对象占用的内存空间、类名以及成员变量列表，当然这些都是只读的。

**总结：**

**一个NSObject对象占用多少内存？**

答：系统会为一个NSObject对象分配最少16个字节的内存空间。一个指针变量所占用的大小（64bit占8个字节，32bit占4个字节）

**对象的isa指针指向哪里？**

答：instance对象的`isa`指针指向class对象，class对象的`isa`指针指向meta-class对象，meta-class对象的`isa`指针指向基类的meta-class对象，基类自己的`isa`指针指向自己。


**OC的类信息存放在哪里？**


答：成员变量的具体值存放在实例对象（instance对象）；对象方法，协议，属性，成员变量信息存放在类对象（class对象）；类方法信息存放在元类对象（meta-class对象）。







<br/>

***
<br/>

**`参考资料：`**

[OC对象本质](https://www.jianshu.com/p/ffd742041946)








































































