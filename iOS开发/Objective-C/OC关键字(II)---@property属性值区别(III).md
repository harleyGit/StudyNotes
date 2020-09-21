>#***非容器不变变量***
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
***`修改值`***
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
***`copy 和 strong 置为空`***
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

由此可得：***`非容器不可变量除了copy以外，其他特性同非容器可变变量相同，非容器不可变量copy是浅拷贝。`***

由上实验可得：在不可变容器变量(NSArray)中，容器本身都是浅拷贝包括copy，里面(包括NSMutableArray)的数据也是浅拷贝，同NSString一样。



<br/>
##***总结:***
***`copy，strong，weak，assign的区别。`***

&emsp;&emsp;  `可变变量中`，copy是重新开辟一个内存，strong，weak，assgin后三者不开辟内存，只是指针指向原来保存值的内存的位置，storng指向后会对该内存引用计数+1，而weak，assgin不会。weak，assgin会在引用保存值的内存引用计数为0的时候值为空，并且weak会将内存值设为nil，assign不会，assign在内存没有被重写前依旧可以输出，但一旦被重写将出现奔溃

&emsp;&emsp;  `不可变变量中`，因为值本身不可被改变，copy没必要开辟出一块内存存放和原来内存一模一样的值，所以内存管理系统默认都是浅拷贝。其他和可变变量一样，如weak修饰的变量同样会在内存引用计数为0时变为nil。

***`容器本身遵守上面准则，但容器内部的每个值都是浅拷贝。`***

![内存存储图](https://upload-images.jianshu.io/upload_images/2959789-fab982e68106c332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;&emsp;  综上所述，当创建property构造器创建变量value1的时候，使用copy，strong，weak，assign根据具体使用情况来决定。value1 = value2，如果你希望value1和value2的修改不会互相影响的就用用copy，反之用strong,weak,assign。如果你还希望原来值C(C是什么见【内存存储图】)为nil的时候，你的变量不为nil就用strong,反之用weak和assign。weak和assign保证了不强引用某一块内存，如delegate我们就用weak表示，就是为了防止循环引用的产生。
&emsp;&emsp;  另外，我们上面讨论的是类变量，直接创建局部变量默认是Strong修饰。


### delegate为什么要用weak或者assign而不用strong

&emsp;&emsp;   a创建对象b,b中有C类对象c，所以a对b有一个引用,b对c有一个引用，a.b引用计数分别为1。当c.delegate = b的时候，实则是对b有了一个引用，如果此时c的delegate用strong修饰则会对b的值内存引用计数+1，b引用计数为2。当a的生命周期结束，随之释放对b的引用，b的引用计数变为1，导致b不能释放，b不能释放又导致b对c的引用不能释放，c引用计数还是为1，这样就造成了b和c一直留在了内存中。

&emsp;&emsp;   而要解决这个问题就是使用weak或者assign修饰delegate，这样虽然会有c仍然会对b有一个引用，但是引用是弱引用，当a生命周期结束的时候，b的引用计数变为0，b释放后随之c的引用消失，c引用计数变为0，释放。
