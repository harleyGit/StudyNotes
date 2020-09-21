```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
id obj = [[NSObject alloc] init];
[obj autorelease];
[pool drain];  //等同于[obj release];
```

&emsp;&emsp;  若大量产生autorelease对象，不废弃NSAutoreleasePool对象，那么生成的对象就不能释放，会产生内存不足现象！典型的例子是读入大量图片改变其尺寸。图像文件读入到NSData对象，并从中生成UIImage对象，改变对象尺寸生成新的UIImage对象，会产生大量autorelease对象。

&emsp;&emsp;  在使用变量时，默认在变量前加 __strong;
&emsp;&emsp;  如:
```
 id __strong  test = [[Test alloc] init];
```

<br/>
****
<br/>


&emsp;&emsp;  ***`使用 init/new/copy/mutableCopy 方法返回值取得的对象是自己生成并持有的。但除了这些方法外生成的对象，是生成但不持有，可以通过一些修饰词来解决。如：__strong`***
<br/>

>#***__strong修饰符***
&emsp;&emsp;  `id类型和对象类型的所有权修饰符默认为__strong修饰符，所以不需要再写上“__strong”。`
&emsp;&emsp;  通过__strong修饰符，不必再次键入retain或者release，完美的满足了“引用技术式内存管理的思考方式”
```
id __strong        obj0;
id __weak          obj1;
id __autoreleasing obj2;

//上面的赋值等同于下面的代码段
id __strong        obj0 = nil;
id __weak          obj1 = nil;
id __autoreleasing obj2 = nil;
```


<br/>
****
<br/>


>#***__weak修饰符***
循环引用容易发生内存泄漏。
***`内存泄漏`***就是应当废弃的对象在超出其生存周期后继续存在。


<br/>
****
<br/>


>#***__unsafe_unretained修饰符***
附有__unsafe_unretained修饰符的变量不属于编译器的内存管理对象。
