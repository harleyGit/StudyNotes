
- **匿名函数Block**
- **Block 分类**
	- 全局block
	- 堆block
	- 栈block
- **Block循环引用**
	- 循环引用3种解决方案 
- **Block 底层原理**
- **Block 高级使用**
- [**Block 本质**](https://www.jianshu.com/p/4e79e9a0dd82)
- [**Block 用法、举例、底层**](http://www.cocoachina.com/cms/wap.php?action=article&id=23147)
- [**Block 详解**](https://www.jianshu.com/p/00a0747740ba)





<br/>

***
<br/>

># 匿名函数Block

Block 会对内部的对象进行强引用，是为了保证它属性的存在。
OC 函数中的局部变量在栈中存放。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //main线程
    void (^block)(void) = ^{
        NSLog(@"welcome to block");
    };//匿名block代码块的定义
    
    block();

     NSLog(@"%@", block);
    
}

```
打印结果：

`welcome to block`

`<__NSGlobalBlock__: 0x10be7c240>`

分析：当前是在主线程中，当运行到定义的匿名block时，会把这段代码由栈区拷贝到堆区，保存到堆区中。一旦进行调用block(),就会来到执行区域进行NSLog打印了。


<br/>

***
<br/>


># Block 分类

<br/>

- NSGlobalBlock(全局block)： block在静态区。

```
void (^block)(void) = ^{
        NSLog(@"welcome to block");
    };//匿名block代码块的定义
    
    block();
```

<br/>

-  NSMallocBlock:  堆block；

```
int a = 10;
void (^block)(void) = ^{
        NSLog(@"welcome to block %d", a);
    };//匿名block代码块的定义
    
    block();
```

若访问了相应的内存空间，访问了外部的变量并且捕获了外部变量或者属性后，由全局block变为堆block。


<br/>

-  NSStackBlock(栈block)

```
int a = 10;
 NSLog(@"第三种block： %@", ^{
        NSLog(@"%d", a);
    });

//打印结果：第三种block： <__NSStackBlock__: 0x7ffee5cee960>

```


<br/>

***
<br/>


># Block循环引用

<br/>

**循环引用出现**

```
//block 的别名
typedef void(^KCBlock)(void);

@property(nonatomic, copy)KCBlock block;
@property(nonatomic, copy)NSString *name;


- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor groupTableViewBackgroundColor];
    
    //循环引用
    self.name = @"will kuang";
    self.block = ^{
        NSLog(@"%@", self.name);//_name也会导致
    };//性能差
 }
```

<br/>

-  `解决方案一：__weak`

```
//第一种解决方案:weakSelf  + weak - Strong -dance  强弱共舞
__weak typeof(self) weakSelf = self;
 self.block = ^{
        NSLog(@"%@", weakSelf.name);

      //当接下来的代码段需要继续使用self时，我们需要对其延长生命周期
      //__strong typeof(self) strongSelf = weakSelf;
    };

```

<br/>

-  `解决方案二： __block`

```
//第二种解决方案：__block
    __block ViewController *weakSelf = self;    //重新拷贝一份：weakSelf
    //weakSelf(被self捕捉了) - self(捕捉了相应的block) - block(捕捉了weakSelf) --weakSelf(若是把weakSelf置为nil，block就不会持有weakSelf)
    self.block = ^{
        NSLog(@"%@", weakSelf.name);    //临时变量被持有,捕获在这个内存区域
        weakSelf = nil;
    };
    self.block();
```

<br/>

- `解决方案三`

```
//在ViewController.m中定义
typedef void(^KCBlock)(ViewController *);

//第三种方式：去想：为什么会产生循环引用
    //self -- block -- self
    self.block = ^(ViewController *vc) {//传入一个self，变成一个临时变量持有，当出了这个作用域后就会释放
        NSLog(@"%@", vc.name);
    };
    self.block(self);

```



<br/>

***
<br/>

># Block 底层原理

<br/>

```
    __block int a = 10; //此时a存放在栈区域
    NSLog(@"前   =   %p", &a);
    void (^block)(void) = ^{
        a ++;
        NSLog(@"中   =   %p", &a);
        NSLog(@"welcome to block %d", a);
    };
    block();
    NSLog(@"后   =   %p", &a);

```

打印结果：

`2019-02-28 18:03:54.337567+0800 Test[10932:362482] 前   =   0x7ffeed6c09b8`
`2019-02-28 18:03:54.337783+0800 Test[10932:362482] 中   =   0x600001138ef8`
`2019-02-28 18:03:54.337951+0800 Test[10932:362482] welcome to block 11`
`2019-02-28 18:03:54.338088+0800 Test[10932:362482] 后   =   0x600001138ef8`

分析:`前`中的a打印来自于栈，站之后被block捕捉，a的地址发生了变化。中 后 中的a被拷贝进入了堆地址中了。

<br/>

 终端建立 C 语言文件
 
```
$ mkdir 003---Block原理探究
$ cd 003---Block原理探究
$ vim block.c  # vim 打开对应的文件名，若没有则创建对应的文件名。

# vim 打开文件后是命令模式，使用 i 或者 a 进入编辑模式。
# 在文件中输入

#include "stdio.h"
int main(){
int a = 10; 
void (^block)(void)=^{

printf("%d",a);
}

block();

return 0;
}

# 完成输入后，按下 esc ，这样就退回 vim 命令模式了，输入 : 号切换到末行模式，输入 x(或者 wq)可以用来保存，若输入 q! 则不保存。

# 对这段代码进行编译，输入：
$gcc block.c
block.c:7:2: error: expected ';' at end of
      declaration
}
 ^
 ;
1 error generated.
# 1 error generated. 报错出现。；
# 进入文件中将block后的{}后添加一个  ;  分号，再次编译，输入：
$gcc block.c
#编译成功后会出现一个可执行文件  a.out ,输入：
$./a.out
#会打印出：10
#使用clang 进行编译，输入：
$ clang -rewrite-objc block.c
#会出现一个block.app的文件，其为c++文件。从这段简单的代码可以看到其实现的过程。



```




<br/>

***
<br/>


># Block 高级使用

<br/>

- `链式编程+函数编程`

```
//block 应用： 链式编程+函数编程
//调用where时会返回一个block，该block需要传递一个字符串参数，并且返回一个字符串。
//所以where这里需要传递一个字符串参数
NSLog(@"%@", self.select.where(@"金科刺青网"));

- (ViewController *)select {
    NSLog(@"111111111111");
    
    return self;
}

- (NSString *(^)(NSString *))where{
    NSLog(@"whree ");
    
    NSString *(^block)(NSString *) = ^(NSString *word){
        return [NSString stringWithFormat:@"Ken say: %@", word];
    };
    return block;
}
```
打印结果：
`2019-03-01 10:29:11.694078+0800 Test[1324:32894] 111111111111`
`2019-03-01 10:29:11.694276+0800 Test[1324:32894] whree`
`2019-03-01 10:29:11.694460+0800 Test[1324:32894] Ken say: 金科刺青网`



<br/>

- `函数式编程`

```
//block 函数
 //函数式编程：y=f(x) ---> y=f(f(x))
 [self functionBlock:^(NSString *word) {
      NSLog(@"%@",word);
}];
  

- (void)functionBlock:(void(^)(NSString *))success{
    if (success) {
        //灵活、异步，在网络请求时可以调用代码块，如AFN中大量使用
        success(@"welcome to 函数式编程");
    }
}


```

打印结果：

`Test[1516:46925] welcome to 函数式编程`



<br/>

***
<br/>
