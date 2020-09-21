># 指针类型
<br/>
## 空指针
&emsp;  指向空，或者说不指向任何东西。在C语言中，我们让指针变量赋值为NULL表示一个空指针，而C语言中，NULL实质是 ((void*)0) ，  在C++中，NULL实质是0。
&emsp;  任何程序数据都不会存储在地址为0的内存块中，它是被操作系统预留的内存块。

摘自 `stddef.h`
```
#ifdef __cplusplus
     #define NULL    0
#else    
     #define NULL    ((void *)0)
#endif
```


<br/>
## 坏指针
&emsp;  指针变量的值是 `NULL`，或者未知的地址值，或者是当前应用程序不可访问的地址值，这样的指针就是`坏指针`，不能对他们做解指针操作，否则程序会出现运行时错误，导致程序意外终止。
&emsp;  任何一个指针变量在做 解地址操作前，都必须保证它指向的是有效的，可用的内存块，否则就会出错。坏指针是造成C语言Bug的最频繁的原因之一。
```
void opp()
{
     int*p = NULL;
     *p = 10;      //Oops! 不能对NULL解地址
}

void foo()
{
     int*p;
     *p = 10;      //Oops! 不能对一个未知的地址解地址
}

void bar()
{
     int*p = (int*)1000; 
     *p =10;      //Oops!   不能对一个可能不属于本程序的内存的地址的指针解地址
}
```


<br/>
## void*类型指针
&emsp;  由于void是空类型，因此`void*`类型的指针只保存了指针的值，而丢失了类型信息，我们不知道他指向的数据是什么类型的，只指定这个数据在内存中的起始地址，如果想要完整的提取指向的数据，程序员就必须对这个指针做出正确的类型转换，然后再解指针。因为，编译器不允许直接对`void*`类型的指针做解指针操作。


<br/>
## 野指针
&emsp; `野指针`：就是指针指向的位置是不可知的（随机的、不正确的、没有明确限制的）很可能触发运行时段错误。

&emsp; `产生原因`：指针变量如果是局部变量，则分配在栈上，本身遵从栈的规律（反复使用，使用完不擦除，所以是脏的，本次在栈上分配到的变量的默认值是上次这个栈空间被使用时余留下来的值），就决定了栈的使用多少会影响这个默认值。
&emsp; 指针变量在定义时如果未初始化，值也是随机的。指针变量的值其实就是别的变量（指针所指向的那个变量）的地址，所以意味着这个指针指向了一个地址是不确定的变量，这时候去解引用就是去访问这个地址不确定的变量，所以结果是不可知的。

&emsp; `危害`:
-  指向不可访问（操作系统不允许访问的敏感地址，譬如内核空间）的地址，结果是触发段错误，这种算是最好的情况了；
-  指向一个可用的、而且没什么特别意义的空间（譬如我们曾经使用过但是已经不用的栈空间或堆空间），这时候程序运行不会出错，也不会对当前程序造成损害，这种情况下会掩盖你的程序错误，让你以为程序没问题，其实是有问题的；
-  指向了一个可用的空间，而且这个空间其实在程序中正在被使用（譬如说是程序的一个变量x），那么野指针的解引用就会刚好修改这个变量x的值，导致这个变量莫名其妙的被改变，程序出现离奇的错误。一般最终都会导致程序崩溃，或者数据被损害。这种危害是最大的。

#`野指针产生`
```

#include <stdio.h>
#include <stdlib.h>
 
int main()
{
	char *p1 = NULL;
    printf("p1:%p, &p1:%p\n",p1,&p1);
    
    p1 = (char*)malloc(100);      //为p1在堆区分配空间
    if(p1 == NULL)                //若为空直接return出程序
    {
        return 0;
    }
    printf("p1:%p, &p1:%p\n",p1,&p1);
    
    if(p1 != NULL)                //目的：释放p1
    {
        free(p1);                 //只释放了p1指向的堆区空间  并没有将指针p1置为空
    }
    printf("p1:%p, &p1:%p\n",p1,&p1);
}
```
打印值：
`p1:0x0, &p1:0x7ffeefbff548`
`p1:0x1006179f0, &p1:0x7ffeefbff548`
`p1:0x1006179f0, &p1:0x7ffeefbff548`

由打印值看：`p1并不为0，也就是在释放p1时，仅仅释放的是p1指向的内存空间，并没有将指针p1指为空，此时p1就成了野指针！`

解决方法：
-  `在定义一个指针时初始化为NULL;`
-  `释放指针指向的内存空间时，将指针重置为NULL;`

#`优化代码`
```
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
    char *p1 = NULL;
    printf("p1:%p, &p1:%p\n",p1,&p1);
    
    p1 = (char*)malloc(100);      //为p1在堆区分配空间
    if(p1 == NULL)                //若为空直接return出程序
    {
        return 0;
    }
    printf("p1:%p, &p1:%p\n",p1,&p1);
    
    if(p1 != NULL)                //目的：释放p1
    {
        free(p1);                 //只释放了p1指向的堆区空间  并没有将指针p1置为空
        p1 = NULL;			      //重新置为空
    }
    printf("p1:%p, &p1:%p\n",p1,&p1);
}

```
打印值：
`p1:0x0, &p1:0x7ffeefbff548`
`p1:0x10058fdd0, &p1:0x7ffeefbff548`
`p1:0x0, &p1:0x7ffeefbff548`




<br/>
***
<br/>

># 函数指针
&emsp;  一个函数总是占用一段连续的内存区域，函数名在表达式中有时也会被转换为该函数所在内存区域的首地址，这和数组名非常类似。我们可以把函数的这个首地址（或称入口地址）赋予一个指针变量，使指针变量指向函数所在的内存区域，然后通过指针变量就可以找到并调用该函数，这种指针就是`函数指针`。


定义：
`returnType (*pointerName)(parameters list);`

-  returnType 为函数返回值类型;
-  pointerNmae 为指针名称;
-  parameters list 为函数参数列表;
-  ( )的优先级高于*，第一个括号不能省略，如果写作returnType *pointerName(paramlist);



#`DEMO`
```
int main(int argc, const char * argv[])
{
    //函数申明
    int  pointerFunc(int x);

    int (*ptr)(int);
    //将pointerFunc函数的首地址赋给指针ptr
    ptr = pointerFunc;  //等价于 ptr = * pointerFunc, ptr = & pointerFunc;
    printf(" pointerFunc 地址：%p,\n &pointerFunc: %p,\n  *pointerFunc:%p", pointerFunc, &pointerFunc, *pointerFunc);
    int n = (*ptr)(10);
    printf("n = %d", n);

}

//函数实现
int pointerFunc(int x){
    return x +10;
}
```
打印结果：
 `ointerFunc 地址：0x100001410,`
` &pointerFunc: 0x100001410,`
` *pointerFunc:0x100001410`
`  n = 20`
 
&emsp;  ptr是指向函数的`指针变量`，所以可把函数pointerFunc赋给ptr作为ptr的值，即把pointerFunc的入口地址赋给ptr,以后就可以用ptr来调用该函数。
&emsp; 实际上ptr和pointerFunc都指向同一个入口地址，不同就是ptr是一个指针变量，不像函数名称那样是死的，它可以指向任何函数。在程序中把哪个函数的地址赋给它，它就指向哪个函数。而后用指针变量调用它，因此可以先后指向不同的函数。




<br/>
***
<br/>

># 结构体指针

结构体指针有特殊的语法：  -> 符号 
如果p是一个结构体指针，则可以使用 p ->【成员】 的方法访问结构体的成员。p->member 等价于 (*p).member。
```
typedef struct
{
    char name[31];
    int age;
    float score;
}Student;


int main(int argc, const char * argv[])
{
     Student stu = {"Bob" , 19, 98.0};
    Student*ps = &stu;
    
    ps->age = 20;
    ps->score = 99.0;
    printf("name:%s age:%d\n",ps->name,ps->age);
    return 0;
}
```
输出：
`name:Bob age:20`



 <br/>
## 结构体变量申明和指针结构体声明

#`指针结构体声明`
```
typedef struct BinaryTreeNode {
    char data;
    struct BinaryTreeNode *leftChild;
    struct BinaryTreeNode *rightChild;
} BinaryTreeNode, *BinaryTree;


void binaryTreeTest(void);


int initBinaryTree(BinaryTree *binaryTree){
    *binaryTree = NULL;
    
    return TRUE;
}

void  setTest(BinaryTree *node){
    *node = (BinaryTreeNode *)malloc(sizeof(BinaryTreeNode));
    printf("\n\n*node:%p, node:%p", *node, node);
    (*node)->data = 'S';
}

//方法调用
void binaryTreeTest(void){
    BinaryTree binaryTree;  //指针结构体声明
    
    int statusCode = 0;
    

    statusCode = initBinaryTree(&binaryTree);
    printf("binaryTree:%p,  &binaryTree:%p", binaryTree, &binaryTree);
    setTest(&binaryTree);
    printf("\n\n-->>data:%c, binaryTree:%p, &binaryTree:%p", binaryTree->data, binaryTree, &binaryTree);
    
}

```
打印：
`binaryTree:0x0,  &binaryTree:0x7ffeefbff4d8`  //binaryTree：指针变量地址，&binaryTree：指针的地址

`*node:0x1007156a0, node:0x7ffeefbff4d8`  //变量地址不同了，但是指针的地址相同

`-->>data:S, binaryTree:0x1007156a0, &binaryTree:0x7ffeefbff4d8`  //binaryTree->data中的binaryTree指向的地址都是相同的。


<br/>
<br/>
## 结构体变量申明

```
typedef struct BinaryTreeNode {
    char data;
    struct BinaryTreeNode *leftChild;
    struct BinaryTreeNode *rightChild;
} BinaryTreeNode, *BinaryTree;


void binaryTreeTest(void);



int initBinaryTree(BinaryTreeNode *binaryTree){
    binaryTree = NULL;
    
    return TRUE;
}


void  setTest(BinaryTreeNode *node){
    node = (BinaryTreeNode *)malloc(sizeof(BinaryTreeNode));
    printf("\n\n*node:%p, &node:%p", node, &node);
    node->data = 'S';
}

//方法调用
void binaryTreeTest(void){
    BinaryTreeNode binaryTree;
    
    int statusCode = 0;

    statusCode = initBinaryTree(&binaryTree);
    printf("binaryTree:%p,  &binaryTree:%p", binaryTree, &binaryTree);
    setTest(&binaryTree);
    printf("\n\n-->>data:%c, binaryTree:%p, &binaryTree:%p", binaryTree.data, binaryTree, &binaryTree);

    
}

```
打印：
`binaryTree:0x7ffeefbff4c8,  &binaryTree:0x0`

`*node:0x1006609d0, &node:0x7ffeefbff488`  //可以看到与上述的结构体变量地址完全不同，是两个不同的结构体。相当于类初始化2个对象

`-->>data:, binaryTree:0x7ffeefbff4c8, &binaryTree:0x7ffeefbff4c8(lldb) `



<br/>
***
<br/>

># 数组和指针
#`数组名作为右值的时候，就是第一个元素的地址`
```
    int arr[3] = {1,2,3};
    
    int*p_first = arr;
    printf("%d\n",*p_first);  /输出：1
```

<br/>

#`指向数组元素的指针支持 递增、递减运算`
```
    int arr[3] = {1,2,3};
    
    int*p = arr;
    for(;p!=arr+3;p++){
        printf("%d\n",*p);
    }
    printf("arr size:%lu", sizeof(arr));
    printf("sizeof(p): %lu\n",sizeof(p));
```
输出：
`1`
`2`
`3`
`arr size:12`
`izeof(p): 8`

总结：
-  p= p+1 意思是，让p指向原来指向的内存块的下一个相邻的相同类型的内存块。
      同一个数组中，元素的指针之间可以做减法运算，此时，指针之差等于下标之差。

-    p[n]    == *(p+n)
     p[n][m]  == *(  *(p+n)+ m )

-  对数组名使用sizeof时，返回的是整个数组占用的内存字节数。当把数组名赋值给一个指针后，再对指针使用sizeof运算符，返回的是指针的大小。


 

<br/>
***
<br/>

># 函数和指针

&emsp;  C语言中，实参传递给形参，是按值传递的，也就是说，函数中的形参是实参的拷贝。形参和实参只是在值上一样，而不是同一个内存数据对象。这就意味着：这种数据传递是单向的，即从调用者传递给被调函数，而被调函数无法修改传递的参数达到回传的效果。

```
void change(int a)
{
    a++;      //在函数中改变的只是这个函数的局部变量a，而随着函数执行结束，a被销毁。age还是原来的age，纹丝不动。
    printf("change 方法中a=%d， a 地址：%p\n\n", a, &a);
}
int main(void)
{
    int age = 19;
    printf("age 地址：%p\n\n", &age);
    change(age);
    printf("age = %d\n",age);   // age = 19
    return 0;
}
```
输出：
`age 地址：0x7ffeefbff4fc`

`change 方法中a=20， a 地址：0x7ffeefbff4dc`

&emsp;  由此我们可以看到，这是一种深拷贝，age和方法change中的变量地址是两块不同的内存区域。

&emsp;  有时候我们可以使用函数的返回值来回传数据，在简单的情况下是可以的，但是如果返回值有其它用途（如返回函数的执行状态量），或者要回传的数据不止一个，返回值就解决不了。


<br/>

传递变量的指针可解决上述的问题：
```
void change(int* pa)
{
    (*pa)++;   //因为传递的是age的地址，因此pa指向内存数据age。当在函数中对指针pa解地址时，
               //会直接去内存中找到age这个数据，然后把它增1。
    printf("change 方法中a=%d， pa 地址：%p\n\n", *pa, pa);
}
int main(void)
{
    int age = 19;
    printf("age 地址：%p\n\n", &age);
    change(&age);
    printf("age = %d\n",age);   // age = 20
    return 0;
}
```
输出：
`age 地址：0x7ffeefbff4fc`

`change 方法中a=20， pa 地址：0x7ffeefbff4fc`

`age = 20`

&emsp;  从指针传递变量我们可以看到，这是一种浅拷贝，age和方法change中的变量地址是两块相同的内存区域。

&emsp;  若传过去的是地址，在函数中必须要通过`" * "`对指针进行解引用，除非有其他用途。
<br/>


`函数交换变量`
```
void swap_bad(int a,int b);
void swap_ok(int*pa,int*pb);


int main(int argc, const char * argv[])
{
    
    int a = 5, b = 9;
    int c = 10, d = 40;
    
    swap_bad(a, b);
    swap_ok(&c, &d);
    
    printf("a = %d, b = %d\n", a,b);
    printf("c = %d, d = %d", c, d);


    return 0;
}

//错误的写法
void swap_bad(int a,int b)
{
    int t;
    t=a;
    a=b;
    b=t;
}

//正确的写法：通过指针
void swap_ok(int*pa,int*pb)
{
    int t;
    t=*pa;
    *pa=*pb;
    *pb=t;
}

```
输出：
`a = 5, b = 9`
`c = 40, d = 10`

![函数内部赋值详解](https://upload-images.jianshu.io/upload_images/2959789-9c6def9614376298.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

#`参数为指针的结构体`
```
typedef struct {
    char name[31];
    int age;
    float score;
}Student;

void show(const Student *student);

void show(const Student *student){
    printf("name:%s , age:%d , score:%.2f\n",student->name,student->age,student->score);
}



int main(int argc, const char * argv[])
{
    Student student = {"Harley", 27, 750};
    
    show(&student);
     printf("student size:%lu", sizeof(student));
    return 0;
}

```
输出：
`name:Harley , age:27 , score:750.00`
`student size:40`

&emsp;   从定义的结构看出，Student变量的大小至少是39个字节，那么通过函数直接传递变量，实参赋值数据给形参需要拷贝至少39个字节的数据，极不高效。而传递变量的指针却快很多，因为在同一个平台下，无论什么类型的指针大小都是固定的：X86指针4字节，X64指针8字节，远远比一个Student结构体变量小。

<br/>
***
<br/>

># const 和指针

&emsp;  如果const 后面是一个类型，则跳过最近的原子类型，修饰后面的数据。（原子类型是不可再分割的类型，如int, short , char，以及typedef包装后的类型）

&emsp;  如果const后面就是一个数据，则直接修饰这个数据。

```
typedef int* pint_t;  //将 int* 类型 包装为 pint_t,则pint_t 现在是一个完整的原子类型


int main(int argc, const char * argv[])
{
    int a = 1;
    
    int const *p1 = &a;        //const后面是*p1，实质是数据a，则修饰*p1，通过p1不能修改a的值
    const int*p2 =  &a;        //const后面是int类型，则跳过int ，修饰*p2， 效果同上
    
    int* const p3 = NULL;      //const后面是数据p3。也就是指针p3本身是const .
    
    const int* const p4 = &a;  // 通过p4不能改变a 的值，同时p4本身也是 const
    int const* const p5 = &a;  //效果同上
    
    
    const pint_t p6 = &a;  //同样，const跳过类型pint_t，修饰p1，指针p1本身是const
    pint_t const p7 = &a;  //const 直接修饰p，同上
    
    return 0;
}

```




<br/>
***
<br/>

># 深拷贝和浅拷贝

![深拷贝和浅拷贝讲解图](https://upload-images.jianshu.io/upload_images/2959789-cb7c889ff3b06192.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  如果2个程序单元（例如2个函数）是通过拷贝 他们所共享的数据的 指针来工作的，这就是浅拷贝，因为真正要访问的数据并没有被拷贝。如果被访问的数据被拷贝了，在每个单元中都有自己的一份，对目标数据的操作相互 不受影响，则叫做深拷贝。


#`浅拷贝Demo`
```
    int *pp;
    int arr[3] = {1,2,3};
    
    pp = &arr[2];
    arr[2] = 100;
    
    printf("pp= %p, *pp = %d\n", pp, *pp);

```
输出：
`pp= 0x7ffeefbff4e4, *pp = 100`




 

<br/>
***
<br/>
回顾：
[指针(I)](https://www.jianshu.com/p/f533ad9302dc)

<br/>
参考资料：
