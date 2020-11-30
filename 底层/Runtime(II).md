- **消息转发**
- **AssociatedObject 关联对象**
- **消息发送**
- **[深入了解Category](https://tech.meituan.com/2015/03/03/diveintocategory.html)**


<br/>

***
<br/>


># 消息转发

![](https://upload-images.jianshu.io/upload_images/2959789-a90a63256cbe84ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

![消息处理图](https://upload-images.jianshu.io/upload_images/2959789-35efcf56ecff81f5.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![方法消息发送图](https://upload-images.jianshu.io/upload_images/2959789-33bf2744e2a17617.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

面试题：

-  **当建立一个A类，创建它的分类A+,在A类中有一个方法`- (void)test;`,在分类A+中也有一个方法`- (void)test;`，在初始化A类的实例对象中，调用`test`方法，会调用哪个`test`方法？为什么？**

解析：会调用分类A+中的`test`方法，这是因为分类在添加到类的方法列表中，我们方法列表会移动，腾出存储空间给分类添加到前面的地址空间。当调用这个方法时若分类有这个方法直接调用，它就不会再到原类中在去代调用这个方法。




<br/>

***
<br/>


># AssociatedObject 关联对象

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

># 消息发送

&emsp;  执行一个方法，有些语言，编译器会执行一些额外的优化和错误检查，因为调用关系很直接也很明显。但对于消息分发来说，就不那么明显了。在发消息前不必知道某个对象是否能够处理消息。你把消息发给它，它可能会处理，也可能转给其他的 Object 来处理。一个消息不必对应一个方法，一个对象可能实现一个方法来处理多条消息。

&emsp;  在Objective-C:中，消息是通过objc_msgSend()这个 runtime 方法及相近的方法来实现的。这个方法需要一个target，selector，还有一些参数。理论上来说，编译器只是把消息分发变成objc_msgSend来执行。比如下面这两行代码是等价的:

```
[array insertObject:foo atIndex:5];

objc_msgSend(array, @selector(insertObject:atIndex:), foo, 5);
```


<br/>

***
<br/>


<br/>

***
<br/>


参考资料:

[iOS runtime——函数/使用方法/使用场景/示例](https://blog.csdn.net/potato512/article/details/51106645)


