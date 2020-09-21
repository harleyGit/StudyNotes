>#***ARC的关闭***
#关闭整个工程的ARC
>objective-C Automatic Reference Counting 设置为NO,关闭ARC,YES为开启ARC模式

![关闭整个工程的 ARC 模式](https://upload-images.jianshu.io/upload_images/2959789-5dcc3c0c3ec1c1c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#关闭或开启部分文件的ARC
>关闭ARC：-fno-objc-arc<br/>
开启ARC：-fobjc-arc

![关闭 ViewController 文件的 ARC 模式](https://upload-images.jianshu.io/upload_images/2959789-aac8c91b35231361.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***
<br/>

>#***野指针***
什么是野指针？
`"野指针"不是nil指针，是指向"垃圾"内存（不可用内存）的指针。野指针是非常危险的。`
![野指针崩溃](https://upload-images.jianshu.io/upload_images/2959789-12eb2ba4a548efee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对上图的代码进行改动一下下
![指针置为 nil](https://upload-images.jianshu.io/upload_images/2959789-769dec7d9419e6ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***`说明：`***
&emsp;&emsp; ①.  model对象接收到release消息后，会马上被销毁，所占用的内存会被回收。” 这里执行release只是标记对象占用的那块内存可以被释放，但是具体的释放的时间是不可控的，如果在release之后执行[stu setLow:10.0f];不一定会野指针crash，如果对象内存已经被其他对象覆写占用，那么会crash，如果没有没覆写，调用依然可以正确执行。

&emsp;&emsp; ②. 向空指针发送消息不会报错，但是给野指针发送消息会报错

<br/>
>#***僵尸对象***
***`定义:一个已经被释放的对象 就叫做僵尸对象`***
&emsp;&emsp;  遇到exc_bad_access这类问题一般都是僵尸对象引起的，可以开启僵尸模式定位，我们并没有保留他，只是在程序运行到该对象的时候会产生问题，没有谁会运用他，只会定位他然后解决掉。

<br/>
***`内存回收的本质`***

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
***`僵尸对象无法复活`***
①. 当一个对象的引用计数器变为0以后 这个对象就被释放了.

②. 就无法取操作这个僵尸对象了. 所有对这个对象的操作都是无效的.

③. 因为一旦对象被回收 对象就是1个僵尸对象 而访问1个僵尸对象 是没有意义.



># ***retain***
***retain同strong，就是指针指向值地址，同时进行引用计数加1。***


<br/>
***
<br/>


>#***非NSMutableString的情况***
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
***添加一个元素***
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

***修改数组元素***
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





























