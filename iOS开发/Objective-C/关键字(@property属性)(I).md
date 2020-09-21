># nonatomic和atomic
- nonatomic：不使用同步锁，非原子性
- atomic：使用同步锁，原子性
>属性声明为atomic时，在该属性在调用getter和setter方法时，会加上同步锁(也叫互斥锁@synchronized)。
即在属性在调用getter和setter方法时，保证同一时刻只能有一个线程调用属性的读/写方法。保证了读和写的过程是可靠的。
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
[weak 实现原理](https://www.cnblogs.com/guohai-stronger/p/10161870.html)

[野指针，僵尸对象，空指针详解](https://juejin.im/entry/5c930d746fb9a070d20f1c4d)
