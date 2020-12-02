
- **nonatomic和atomic**
- **retain**
- **copy**
- **assign**
- **weak**




<br/>

***
<br/>


># nonatomic和atomic

- nonatomic：不使用同步锁，非原子性
- atomic：使用同步锁，原子性

>属性声明为atomic时，在该属性在调用getter和setter方法时，会加上同步锁(也叫互斥锁@synchronized)。
即属性在调用getter和setter方法时，保证同一时刻只能有一个线程调用属性的读/写方法。保证了读和写的过程是可靠的。
但并不能保证数据一定是可靠的。

理由如下图：
![数据不可靠](https://upload-images.jianshu.io/upload_images/2959789-1ef27095cf1e0b8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/>

># retain

&emsp;  retain到另外一个对象之后，地址是不会变化的，地址也为0x1111，实质上是建立一个指针，也就是指针拷贝，内容也是相同的，retain值会加1。

```
- (void)setName:(NSString *)name{
    if (_name != name) {
        [ _name release];
        _name = [name retain];
    }
}
- (NSString *)name{
    return [[ _name retain] autorelease];
}
```




<br/>

***
<br/>

># copy

&emsp;  copy建立一个相同的对象，如果一个NSString对象，假如地址为0x1111，内容为@"hello"，通过Copy到另一个对象之后，地址为0x2322，内容也相同，而新的对象retain为1，旧的对象是不会发生变化。

内部实现

```
- (void)setName:(NSString *)name{
    if (_name != name) {
        [ _name release];
        _name = [name copy];
    }
}
- (NSString *)name{
    return [[ _name retain] autorelease];
}
```



<br/>

***
<br/>

># assign

内部实现

```
- (void)setName:(NSString *)name{
    _name = name;
}


- (NSString *)name{
    return _name;
}
```



<br/>

***
<br/>

># weak

- **用法：**

&emsp; weak是弱引用，用weak来修饰、描述所引用对象的计数器并不会加1，而且weak会在引用对象被释放的时候自动置为nil，这也就避免了野指针访问坏内存而引起奔溃的情况，另外weak也可以解决循环引用。

>问题：
`为什么修饰代理使用weak而不是用assign？`

>&emsp; assign可用来修饰基本数据类型，也可修饰OC的对象，但如果用assign修饰对象类型指向的是一个强指针，当指向的这个指针释放之后，它仍指向这块内存，必须要手动给置为nil，否则会产生野指针，如果还通过此指针操作那块内存，会导致EXC_BAD_ACCESS错误，调用了已经被释放的内存空间；

>&emsp; 而weak只能用来修饰OC对象，而且相比assign比较安全，如果指向的对象消失了，那么它会自动置为nil，不会导致野指针。

<br/>

> **原理概括：**

> &emsp; weak表其实是一个哈希表，key是所指对象的指针，value是**`weak指针的地址数组`**(注意：这个数组是weak指针的地址，不是对象的指针地址)。（value是数组的原因是：因为一个对象可能被多个弱引用指针指向）
> 
> &emsp; Runtime维护了一张weak表，用来存储某个对象的所有的weak指针。


<br/>

**[weak 实现原理](https://www.cnblogs.com/guohai-stronger/p/10161870.html)**

