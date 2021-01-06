
- **ARC的关闭**
- **引用计数**
- **property 属性值**
	-  点语法
	-  readWrite
	-  retain和assign区别
		- 非NSMutableString的情况 
		- copy 和 strong 置为空
		- copy，strong，weak，assign的区别。
		- delegate为什么要用weak或者assign而不用strong
- **野指针**
- **僵尸对象**



<br/>

***
<br/>



># ARC的关闭

关闭整个工程的ARC

>objective-C Automatic Reference Counting 设置为NO,关闭ARC,YES为开启ARC模式

![关闭整个工程的 ARC 模式](https://upload-images.jianshu.io/upload_images/2959789-5dcc3c0c3ec1c1c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**关闭或开启部分文件的ARC**

>关闭ARC：-fno-objc-arc<br/>
开启ARC：-fobjc-arc

![关闭 ViewController 文件的 ARC 模式](https://upload-images.jianshu.io/upload_images/2959789-aac8c91b35231361.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>

># **引用计数**

```
NSMutableString *tempMStr = [[NSMutableString alloc]initWithString:@"strValue"];
NSLog(@"tempMStr值地址:%p,\ntempMStr值%@,\n tempMStr值引用计数%@\n", tempMStr,tempMStr,[tempMStr valueForKey:@"retainCount"]);
```

打印结果：

```
2018-09-12 07:58:44.033175+0800 Test[935:102832] tempMStr值地址:0x60400005ba80,
tempMStr值strValue,
tempMStr值引用计数1
```

**`原理示意图`**

![原理](https://upload-images.jianshu.io/upload_images/2959789-10650da701e0f04c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

A=C其实是在内存中创建了一个A，然后又开辟了一个内存C，C里面存放的着值B。不懂这句话，可以看看C语言的指针这一章。

**`原理：`**

此处tempMStr就是A，值地址就是C，“strValue”就是B，而引用计数这个概念是针对C的，赋值给其他变量或者指针设置为nil，如tempStr = nil，都会使得引用计数有所增减。当内存区域引用计数为0时就会将数据抹除。而我们使用copy,strong,retain,weak,assign区别就在：


>1.是否开辟新的内存
<br/>
2.是否对地址C有引用计数增加

**`需要注意的是property修饰符是在被赋值时起作用`**

<br/>

```

@property(copy,nonatomic)NSMutableString    *aCopyMStr;
@property(strong,nonatomic)NSMutableString  *strongMStr;
@property(weak,nonatomic)NSMutableString    *weakMStr;
@property(assign,nonatomic)NSMutableString  *assignMStr;

- (void)memoryTest {
    
    NSMutableString *mstrOrigin = [[NSMutableString alloc]initWithString:@"mstrOriginValue"];
    NSLog(@"mstrOrigin输出:%p,%@\n", mstrOrigin,mstrOrigin);
    NSLog(@"1. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
    
    self.aCopyMStr = mstrOrigin;
    NSLog(@"aCopyMStr输出:%p,%@\n",_aCopyMStr,_aCopyMStr);
    NSLog(@"2. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
    
    self.strongMStr= mstrOrigin;
    NSLog(@"strongMStr输出:%p,%@\n",_strongMStr,_strongMStr);
    NSLog(@"3. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
    
    self.weakMStr  = mstrOrigin;
    NSLog(@"weakMStr输出:%p,%@\n",_weakMStr,_weakMStr);
    NSLog(@"4. 引用计数%@",[mstrOrigin valueForKey:@"retainCount"]);
}
```

打印结果：

```
2018-09-12 08:28:45.733698+0800 Test[1242:386455] mstrOrigin输出:0x600000250b00,mstrOriginValue

2018-09-12 08:28:45.734030+0800 Test[1242:386455] 1. 引用计数1

2018-09-12 08:28:45.734529+0800 Test[1242:386455] aCopyMStr输出:0x600000250a40,mstrOriginValue

2018-09-12 08:28:45.734720+0800 Test[1242:386455] 2. 引用计数1

2018-09-12 08:28:45.734877+0800 Test[1242:386455] strongMStr输出:0x600000250b00,mstrOriginValue

2018-09-12 08:28:45.735038+0800 Test[1242:386455] 3. 引用计数2

2018-09-12 08:28:45.735205+0800 Test[1242:386455] weakMStr输出:0x600000250b00,mstrOriginValue

2018-09-12 08:28:45.735486+0800 Test[1242:386455] 4. 引用计数2
```

`原理：`

&emsp;&emsp;  strongMStr和weakMStr指针指向的内存地址都和mstrOrigin相同,但mstrOrigin内存引用计数为2，不为3，因为weakMStr虽然指向了数据内存地址（之后用C简称，见【原理】图），但不会增加C计数。copy修饰的的aCopyMStr，赋值后则是自己单独开辟了一块内存，内存上保存“mstrOrigin”字符串，并指向。

拷贝示意图如下:

![指针指向示意图](https://upload-images.jianshu.io/upload_images/2959789-d40bef0ba013f15c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;&emsp;  可见当我修改mstrOrigin的值的时候，必然不会影响aCopyMStr,只会影响strongMStr和weakMStr，是因为指向的内存地址不同。

```
NSLog(@"------------------修改原值后------------------------\n");
 
[mstrOrigin appendString:@"*******"];
NSLog(@"mstrOrigin输出:%p,%@\n", mstrOrigin,mstrOrigin);
NSLog(@"aCopyMStr输出:%p,%@\n",_aCopyMStr,_aCopyMStr);
NSLog(@"strongMStr输出:%p,%@\n",_strongMStr,_strongMStr);
NSLog(@"weakMStr输出:%p,%@\n",_weakMStr,_weakMStr);
```

打印结果：

```
2018-09-12 08:58:58.924352+0800 Test[1510:668804] ------------------修改原值后------------------------

2018-09-12 08:58:58.924492+0800 Test[1510:668804] mstrOrigin输出:0x60000005a340,mstrOriginValue*******

2018-09-12 08:58:58.924752+0800 Test[1510:668804] aCopyMStr输出:0x604000255300,mstrOriginValue

2018-09-12 08:58:58.924925+0800 Test[1510:668804] strongMStr输出:0x60000005a340,mstrOriginValue*******

2018-09-12 08:58:58.925122+0800 Test[1510:668804] weakMStr输出:0x60000005a340,mstrOriginValue*******
```

&emsp;emsp;  copy会重新开辟新的内存来保存一份相同的数据。被赋值对象和原值修改互不影响。strong和weak赋值都指向原来数据地址，区别是前者会对数据地址进行引用计数+1，后者不会。

&emsp;&emsp;  引用计数是否+1有什么实质区别呢？
![原理](https://upload-images.jianshu.io/upload_images/2959789-8761a4fa77c035b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;&emsp;  如果知道“值地址的引用计数为0时，地址上保存的值就会被释放”。那么区别就不难理解，weak修饰的指针A指向的值地址C，那么地址上当其他指向他的指针被释放的时候，这个值地址引用计数也就变为0了，这个A的值也就为nil了。换句话说当值地址C上没有其他强引用指针修饰的时候C就会被立即释放，A的值就变为nil了。
&emsp;&emsp;  `换句话说，当一个强引用和一个弱引用指向值地址C时，强引用释放了，那么弱引用指向的值为nil`

&emsp;&emsp;  这里我们来初始化mstrOrigin和并将strongMStr设置为nil让C的引用计数为0，然后输出weakMStr，看是否为nil.
注：初始化和设为nil都可以将指针所指向的数据地址引用计数减少1。

```
- (void)memoryTest {
    
    NSMutableString*mstrOrigin = [[NSMutableString alloc]initWithString:@"mstrOriginValue*******"];
    
    self.strongMStr = mstrOrigin;
    self.weakMStr   = mstrOrigin;
    
    mstrOrigin = [[NSMutableString alloc]initWithString:@"mstrOriginChange3"];

    NSLog(@"mstrOrigin输出:%p,%@\n", mstrOrigin,mstrOrigin);
    NSLog(@"strongMStr输出:%p,%@\n",_strongMStr,_strongMStr);
    NSLog(@"weakMStr输出:%p,%@\n",_weakMStr,_weakMStr);
    NSLog(@"1. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
    
    NSLog(@"------------------------置为nil-------------------------");
    self.strongMStr = nil;
    NSLog(@"mstrOrigin输出:%p,%@\n", mstrOrigin,mstrOrigin);
    NSLog(@"strongMStr输出:%p,%@\n",_strongMStr,_strongMStr);
    NSLog(@"weakMStr输出:%p,%@\n",_weakMStr,_weakMStr);
    NSLog(@"2. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
}
```

打印结果：

```
2018-09-12 11:15:48.712824+0800 Test[2971:1923860] mstrOrigin输出:0x600000247fb0,mstrOriginChange2

2018-09-12 11:15:48.713060+0800 Test[2971:1923860] strongMStr输出:0x600000247ad0,mstrOriginValue*******

2018-09-12 11:15:48.713288+0800 Test[2971:1923860] weakMStr输出:0x600000247ad0,mstrOriginValue*******

2018-09-12 11:15:48.713790+0800 Test[2971:1923860] 1. 引用计数1

2018-09-12 11:15:48.714106+0800 Test[2971:1923860] ------------------------置为nil-------------------------
2018-09-12 11:15:48.714564+0800 Test[2971:1923860] mstrOrigin输出:0x600000247fb0,mstrOriginChange2

2018-09-12 11:15:48.715070+0800 Test[2971:1923860] strongMStr输出:0x0,(null)

2018-09-12 11:15:48.715226+0800 Test[2971:1923860] weakMStr输出:0x0,(null)

2018-09-12 11:15:48.715373+0800 Test[2971:1923860] 2. 引用计数1
```

**`可见之前引用计数2是mstrOrigin和strongMStr添加的。`**

`结论：`

&emsp;&emsp;  copy会重新开辟新的内存来保存一份相同的数据。被赋值对象和原值修改互不影响。strong和weak虽然都指向原来数据地址，原值修改的时候storng和weak会随之变化。区别是前者会对数据地址进行引用计数+1防止原地址值被释放，但后者不会，当其他值都不在指向值地址时，值地址被释放，weak的值也就是为nil了。我们称会对数据地址增加引用计数的为强引用，不改变引用计数的为弱引用。


<br/>

***
<br/>


># property 属性值

<br/>


- **点语法:** self. 调用property自动生成getter 和 setter 方法，而 `_` 则是直接调用实例例变量。



<br/>


- **readwrite:** 可以使用`setValue:  forKey:`方法对其值进行修改；

```
//访问器寻找名称的成员变量
+ (Bool)  accessInstanceVariablesDierctly {
                return NO;
}

``` 


<br/>


- **assign：** 修饰OC基本数据类型，不会使对象的引用类型计数 +1。



<br/>

- **retain 和 assign 的区别**

**retain同strong，就是指针指向值地址，同时进行引用计数加1。**



```
- (void)memoryTest {
    
    NSMutableString*mstrOrigin = [[NSMutableString alloc]initWithString:@"mstrOriginValue*******"];
    
    self.assignMStr = mstrOrigin;
    self.weakMStr   = mstrOrigin;
    
    mstrOrigin = [[NSMutableString alloc]initWithString:@"mstrOriginChange3"];

    NSLog(@"mstrOrigin输出:%p,%@\n", mstrOrigin,mstrOrigin);
    NSLog(@"assignMStr输出:%p,%@\n",self.assignMStr,self.assignMStr);
    NSLog(@"weakMStr输出:%p,%@\n",_weakMStr,_weakMStr);
    NSLog(@"1. 引用计数%@\n",[mstrOrigin valueForKey:@"retainCount"]);
```
运行报错
![运行报错.png](https://upload-images.jianshu.io/upload_images/2959789-07ad6f53283a1217.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;&emsp;  可以发现在输出assignMStr时会出现奔溃的情况。原因是发送了野指针的情况。assign同weak，指向C并且计数不+1，但当C地址引用计数为0时，assign不会对C地址进行B数据的抹除操作，只是进行值释放。这就导致野指针存在，即当这块地址还没写上其他值前，能输出正常值，但一旦重新写上数据，该指针随时可能没有值，造成崩溃。

注释掉野指针代码段
![注掉野指针代码段.png](https://upload-images.jianshu.io/upload_images/2959789-0f80d733de4115d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打印结果为：
![注掉野指针代码段后.png](https://upload-images.jianshu.io/upload_images/2959789-025bce1b66451b67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> **非NSMutableString的情况**

&emsp;&emsp; 上面我们讨论了典型的例子NSMutableString，即非容器可变变量。也就是说还存在其他三种类型需要讨论：

&emsp;&emsp; a.非容器不可变变量NSSting
&emsp;&emsp; b.容器可变变量NSMutableArray
&emsp;&emsp; c.容器不可变变量NSArray

```
@property(copy,nonatomic)NSMutableArray     *aCopyMArr;
@property(strong,nonatomic)NSMutableArray   *strongMArr;
@property(weak,nonatomic)NSMutableArray     *weakMArr;


- (void)memoryTest {
    NSMutableArray  *mArrOrigin = [[NSMutableArray alloc]init];
    NSMutableString *mstr1 = [[NSMutableString alloc]initWithString:@"value1"];
    NSMutableString *mstr2 = [[NSMutableString alloc]initWithString:@"value2"];
    NSMutableString *mstr3 = [[NSMutableString alloc]initWithString:@"value3"];
    
    [mArrOrigin addObject:mstr1];
    [mArrOrigin addObject:mstr2];
    
    //将mArrOrigin拷贝给aCopyMArr，strongMArr，weakMArr
    self.aCopyMArr= mArrOrigin;
    self.strongMArr= mArrOrigin;
    self.weakMArr= mArrOrigin;
    
    NSLog(@"mArrOrigin输出:%p,%@\n", mArrOrigin,mArrOrigin);
    NSLog(@"aCopyMArr输出:%p,%@\n",_aCopyMArr,_aCopyMArr);
    NSLog(@"strongMArr输出:%p,%@\n",_strongMArr,_strongMArr);
    NSLog(@"weakMArr输出:%p,%@\n",_weakMArr,_weakMArr);
    NSLog(@"weakMArr输出:%p,%@\n",_weakMArr[0],_weakMArr[0]);
    NSLog(@"mArrOrigin中的数据引用计数%@", [mArrOrigin valueForKey:@"retainCount"]);
    NSLog(@"%p %p %p %p",&mArrOrigin,mArrOrigin,mArrOrigin[0],mArrOrigin[1]);
}
```

打印结果：

![打印结果.png](https://upload-images.jianshu.io/upload_images/2959789-67a1937f524d00b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

**添加一个元素**

```
NSLog(@"\n\n<-------------------给原数组添加一个元素----------------->");
    //给原数组添加一个元素
    [mArrOrigin addObject:mstr3];
    
    NSLog(@"mArrOrigin输出:%p,%@\n", mArrOrigin,mArrOrigin);
    NSLog(@"aCopyMArr输出:%p,%@\n",_aCopyMArr,_aCopyMArr);
    NSLog(@"strongMArr输出:%p,%@\n",_strongMArr,_strongMArr);
    NSLog(@"weakMArr输出:%p,%@\n",_weakMArr,_weakMArr);
    NSLog(@"mArrOrigin中的数据引用计数%@", [mArrOrigin valueForKey:@"retainCount"]);
}
```
打印结果：
![添加一个元素.png](https://upload-images.jianshu.io/upload_images/2959789-91a7844702f139af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

**修改数组元素**

```
    NSLog(@"\n\n<-------------------修改数组一个元素----------------->");
    //修改原数组中的元素，看是否有随之变化
    [mstr1 appendFormat:@"aaa"];
    
    NSLog(@"mArrOrigin输出:%p,%@\n", mArrOrigin,mArrOrigin);
    NSLog(@"aCopyMArr输出:%p,%@\n",_aCopyMArr,_aCopyMArr);
    NSLog(@"strongMArr输出:%p,%@\n",_strongMArr,_strongMArr);
    NSLog(@"weakMArr输出:%p,%@\n",_weakMArr,_weakMArr);
    NSLog(@"mArrOrigin中的数据引用计数%@", [mArrOrigin valueForKey:@"retainCount"]);
}
```

打印结果：
![修改后 copy数组值改变.png](https://upload-images.jianshu.io/upload_images/2959789-d187092ef3efe161.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***`综合上述：`***
&emsp;&emsp;  上面3段代码所做的操作是mArrOrigin（value1,value2）赋值给copy,strong,weak修饰的aCopyMArr,strongMArr,weakMArr。通过给原数组增加元素，修改原数组元素值，然后输出mArrOrigin的引用计数，和数组地址，查看变化。

&emsp;&emsp;  发现其中数组本身指向的内存地址除了aCopyMArr重新开辟了一块地址，strongMArr,weakMArr和mArrOrigin指针指向的地址是一样的。

&emsp;&emsp;  可以看出容器可变变量中容器本身和非容器可变变量是一样的，copy深拷贝，strongMArr,weakMArr和assign都是浅拷贝(数组变量和字符串变量对于copy都是深拷贝)。

&emsp;&emsp;  另外我们发现被拷贝对象mArrOrigin中的数据引用计数居然不是1而是3。也就是说容器内的数据拷贝都是进行了浅拷贝。同时当我们修改数组中的一个数据时strongMArr,weakMArr，aCopyMArr中的数据都改变了，说明容器可变变量中的数据在拷贝的时候都是浅拷贝(数组中的元素数据是浅拷贝，在copy，stong，weak中)。

容器可变变量的拷贝结构如下图
![copy，strong， weak 在容器可变变量中的内存情况](https://upload-images.jianshu.io/upload_images/2959789-d108a50e25a9095a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> **`非容器不变变量`**

```
@property(copy,nonatomic)NSString   *aCopyStr;
@property(strong,nonatomic)NSString *strongStr;
@property(weak,nonatomic)NSString   *weakStr;
@property(assign,nonatomic)NSString *assignStr;

- (void)memoryTest {
    
    
    NSString*strOrigin = [[NSString alloc]initWithUTF8String:"strOrigin0123456"];
    
    self.aCopyStr  = strOrigin;
    self.strongStr = strOrigin;
    self.weakStr= strOrigin;
    
    NSLog(@"strOrigin输出:%p,%@\n", strOrigin,strOrigin);
    NSLog(@"aCopyStr输出:%p,%@\n",_aCopyStr,_aCopyStr);
    NSLog(@"strongStr输出:%p,%@\n",_strongStr,_strongStr);
    NSLog(@"weakStr输出:%p,%@\n",_weakStr,_weakStr);
}

```

打印结果：

![原值](https://upload-images.jianshu.io/upload_images/2959789-b342f0efad7036df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

**`修改值`**

```
    NSLog(@"------------------修改原值后\n------------------------");
    
    strOrigin =@"aaa";

    NSLog(@"strOrigin输出:%p,%@ \n", strOrigin,strOrigin);
    NSLog(@"aCopyStr输出:%p,%@ \n",_aCopyStr,_aCopyStr);
    NSLog(@"strongStr输出:%p,%@ \n",_strongStr,_strongStr);
    NSLog(@"weakStr输出:%p,%@ \n",_weakStr,_weakStr);
}
```

打印结果：

![修改后的属性变量值](https://upload-images.jianshu.io/upload_images/2959789-0091b5e18dc2ecde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> **`copy 和 strong 置为空`**

```
self.aCopyStr=nil;
    self.strongStr=nil;
    
    NSLog(@"strOrigin输出:%p,%@ \n", strOrigin,strOrigin);
    NSLog(@"aCopyStr输出:%p,%@ \n",_aCopyStr,_aCopyStr);
    NSLog(@"strongStr输出:%p,%@ \n",_strongStr,_strongStr);
    NSLog(@"weakStr输出:%p,%@ \n",_weakStr,_weakStr);
}
```

打印结果为：

![copy 和 strong 置为 nil](https://upload-images.jianshu.io/upload_images/2959789-7adb70b4fee719ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

综上可得：
&emsp;&emsp;  NSString和NSMutableString（非容器可变变量）基本相同，除了copy。NSString为浅拷贝，NSMutableString是深拷贝。那么为什么NSString的copy是浅拷贝呢，也就是说为什么aCopyStr不自己开辟一个独立的内存出来呢。答案很简单，因为不可变量的值不会改变，既然都不会改变，所以没必要重新开辟一个内存出来让aCopyStr指向他，直接指向原来值位置就可以了。示意图如下
![非容器可变变量](https://upload-images.jianshu.io/upload_images/2959789-d91489bf06b10edd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由此可得：**`非容器不可变量除了copy以外，其他特性同非容器可变变量相同，非容器不可变量copy是浅拷贝。`**

由上实验可得：在不可变容器变量(NSArray)中，容器本身都是浅拷贝包括copy，里面(包括NSMutableArray)的数据也是浅拷贝，同NSString一样。



<br/>

**总结:**

> **`copy，strong，weak，assign的区别。`**

&emsp;&emsp;  `可变变量中`，copy是重新开辟一个内存，strong，weak，assgin后三者不开辟内存，只是指针指向原来保存值的内存的位置，storng指向后会对该内存引用计数+1，而weak，assgin不会。weak，assgin会在引用保存值的内存引用计数为0的时候值为空，并且weak会将内存值设为nil，assign不会，assign在内存没有被重写前依旧可以输出，但一旦被重写将出现奔溃

&emsp;&emsp;  `不可变变量中`，因为值本身不可被改变，copy没必要开辟出一块内存存放和原来内存一模一样的值，所以内存管理系统默认都是浅拷贝。其他和可变变量一样，如weak修饰的变量同样会在内存引用计数为0时变为nil。

**`容器本身遵守上面准则，但容器内部的每个值都是浅拷贝。`**


![内存存储图](https://upload-images.jianshu.io/upload_images/2959789-fab982e68106c332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;&emsp;  综上所述，当创建property构造器创建变量value1的时候，使用copy，strong，weak，assign根据具体使用情况来决定。value1 = value2，如果你希望value1和value2的修改不会互相影响的就用用copy，反之用strong,weak,assign。如果你还希望原来值C(C是什么见【内存存储图】)为nil的时候，你的变量不为nil就用strong,反之用weak和assign。weak和assign保证了不强引用某一块内存，如delegate我们就用weak表示，就是为了防止循环引用的产生。
&emsp;&emsp;  另外，我们上面讨论的是类变量，直接创建局部变量默认是Strong修饰。


> **delegate为什么要用weak或者assign而不用strong**

&emsp;&emsp;   a创建对象b,b中有C类对象c，所以a对b有一个引用,b对c有一个引用，a.b引用计数分别为1。当c.delegate = b的时候，实则是对b有了一个引用，如果此时c的delegate用strong修饰则会对b的值内存引用计数+1，b引用计数为2。当a的生命周期结束，随之释放对b的引用，b的引用计数变为1，导致b不能释放，b不能释放又导致b对c的引用不能释放，c引用计数还是为1，这样就造成了b和c一直留在了内存中。

&emsp;&emsp;   而要解决这个问题就是使用weak或者assign修饰delegate，这样虽然会有c仍然会对b有一个引用，但是引用是弱引用，当a生命周期结束的时候，b的引用计数变为0，b释放后随之c的引用消失，c引用计数变为0，释放。



<br/>


***
<br/>



># 野指针

什么是野指针？
`"野指针"不是nil指针，是指向"垃圾"内存（不可用内存）的指针。野指针是非常危险的。`

![野指针崩溃](https://upload-images.jianshu.io/upload_images/2959789-12eb2ba4a548efee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对上图的代码进行改动一下下

![指针置为 nil](https://upload-images.jianshu.io/upload_images/2959789-769dec7d9419e6ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**[野指针，僵尸对象，空指针详解](https://juejin.im/entry/5c930d746fb9a070d20f1c4d)**

**`说明：`**

&emsp;&emsp; ①.  model对象接收到release消息后，会马上被销毁，所占用的内存会被回收。” 这里执行release只是标记对象占用的那块内存可以被释放，但是具体的释放的时间是不可控的，如果在release之后执行[stu setLow:10.0f];不一定会野指针crash，如果对象内存已经被其他对象覆写占用，那么会crash，如果没有没覆写，调用依然可以正确执行。
&emsp;&emsp; ②. 向空指针发送消息不会报错，但是给野指针发送消息会报错



<br/>

***
<br/>


># 僵尸对象

***`定义:一个已经被释放的对象 就叫做僵尸对象`***

&emsp;&emsp;  遇到exc_bad_access这类问题一般都是僵尸对象引起的，可以开启僵尸模式定位，我们并没有保留他，只是在程序运行到该对象的时候会产生问题，没有谁会运用他，只会定位他然后解决掉。

<br/>
**`内存回收的本质`**

&emsp;&emsp;  ①.  申请一块空间,实际上是向系统申请一块别人不再使用的空间.

&emsp;&emsp;  ②.  释放一块空间,指的是占用的空间不再使用,这个时候系统可以分配给别人去使用.

&emsp;&emsp;  ③.  在这个空间分配给别人之前，数据还是存在的.

   &emsp;&emsp;   a.  OC对象释放以后,表示OC对象占用的空间可以分配给别人.

   &emsp;&emsp;   b.  但是再分配给别人之前 这个空间仍然存在 对象的数据仍然存在.

<br/>
***`野指针访问僵尸对象,有时候会出问题,有时候不会出问题`***
①. 当野指针指向的僵尸对象所占用的空间还没有分配给别人的时候,这个时候其实是可以访问的.

因为这个对象的数据还在.

②. 当野指针指向的对象所占用的空间分配给了别人的时候 这个时候访问就会出问题.

③. 所以,你不要通过一个野指针去访问一个僵尸对象.

       a.虽然可以通过野指针去访问已经被释放的对象,但是我们不允许这么做.

<br/>

**`僵尸对象无法复活`**

①. 当一个对象的引用计数器变为0以后 这个对象就被释放了.

②. 就无法取操作这个僵尸对象了. 所有对这个对象的操作都是无效的.

③. 因为一旦对象被回收 对象就是1个僵尸对象 而访问1个僵尸对象 是没有意义.









<br/>

***
<br/>




















