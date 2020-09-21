># ***引子***

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

***`原理示意图`***
![原理](https://upload-images.jianshu.io/upload_images/2959789-10650da701e0f04c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

A=C其实是在内存中创建了一个A，然后又开辟了一个内存C，C里面存放的着值B。不懂这句话，可以看看C语言的指针这一章。

***`原理：`***
此处tempMStr就是A，值地址就是C，“strValue”就是B，而引用计数这个概念是针对C的，赋值给其他变量或者指针设置为nil，如tempStr = nil，都会使得引用计数有所增减。当内存区域引用计数为0时就会将数据抹除。而我们使用copy,strong,retain,weak,assign区别就在：

>1.是否开辟新的内存<br/>
2.是否对地址C有引用计数增加

***`需要注意的是property修饰符是在被赋值时起作用。`***

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

***`可见之前引用计数2是mstrOrigin和strongMStr添加的。`***
`结论：`
&emsp;&emsp;  copy会重新开辟新的内存来保存一份相同的数据。被赋值对象和原值修改互不影响。strong和weak虽然都指向原来数据地址，原值修改的时候storng和weak会随之变化。区别是前者会对数据地址进行引用计数+1防止原地址值被释放，但后者不会，当其他值都不在指向值地址时，值地址被释放，weak的值也就是为nil了。我们称会对数据地址增加引用计数的为强引用，不改变引用计数的为弱引用。


<br/>
***
<br/>


>#***retain 和 assign 的区别***
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
***
<br/>

















