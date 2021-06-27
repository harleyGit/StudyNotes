> <h1 id=""></h1>
- [**编译器Clang**](#编译器Clang)



<br/>

***
<br/>

ARC 是 iOS 中管理引用计数的技术，帮助 iOS 实现垃圾自动回收，具体实现的原理是由编译器进行管理的，同时运行时库协助编译器辅助完成。主要涉及到 Clang （LLVM 编译器） 和 objc4 运行时库。
本文主要内容由修饰符 __strong 、 __weak 、 __autorelease 拓展开，分别延伸出引用计数、弱引用表、自动释放池等实现原理。在阅读本文之前，你可以看看下面几个问题：


- [在 ARC 下如何存储引用计数？](#ARC下如何存储引用计数)


- [如[NSDictionary dictionary]方法创建的对象在 ARC 中有什么不同之处](#NSDictionary创建对象与ARC的不同)。


- [弱引用表的数据结构](#弱引用表的数据结构)。


- [解释一下自动释放池中的 Hot Page 和 Cold Page](#自动释放池中的HotPage和ColdPage)。


<br/>


> <h1 id="编译器Clang">编译器Clang</h1>


&emsp; 在 Objective-C 中，对象的引用关系由引用修饰符来决定，如__strong、__weak、__autorelease等等，编译器会根据不同的修饰符生成不同逻辑的代码来管理内存。

使用下面命令将OC代码转化为LLVM中间码：

```
// 切换到你文件路径下
cd Path
// 利用 main.m 生成中间码文件 main.ll
clang -S -fobjc-arc -emit-llvm main.m -o main.ll 

```

在main.m文件中加入defaultFunction方法，然后利用的命令行命令将其转换成中间码：

```
void defaultFunction() {
    id obj = [NSObject new];
}
```

经过在终端输入命令后它会在我们的指定的文件夹下生成一个main.ll的文件，打开后我们会看到其内容比较多，大致是以下内容:

```
void defaultFunction() {
	id obj = obj_msgSend(NSObject, @selector(new));
	objc_storeStrong(obj, null);
}

```


obj_msgSend(NSObject, @selector(new))非常好理解，就是新建一个对象，而objc_storeStrong是 objc4 库中的方法，具体逻辑如下：

```
void objc_storeStrong(id *location, id obj)
{
    id prev = *location;
    //检查输入的 obj 地址 和指针指向的地址是否相同
    if (obj == prev) {
        return;
    }
    //持有对象，引用计数 + 1
    objc_retain(obj);
    //指针指向 obj
    *location = obj;
    //原来指向的对象引用计数 - 1
    objc_release(prev);
}
```

其中objc_retain和objc_release也是 objc4 库中的方法，在本文后面分析 objc4 库的章节会详细讲


<br/>

***
<br/>

> <h1 id="isa指针">isa指针</h1>




<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>


<br/>

***
<br/>

> <h1 id=""></h1>


<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>