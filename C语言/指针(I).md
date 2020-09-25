> #  指针(I)


## 物理内存
&emsp;    维修电脑师傅眼中的内存：内存在物理上是由一组DRAM芯片组成的.

<br/>

## 软件内存
&emsp;    操作系统将`RAM`(分为两大类：`SRAM`和`DRAM`)等硬件和软件结合起来，给程序员提供的一种对内存使用的抽象。这种抽象机制使得程序使用的是虚拟存储器,而不是直接操作和使用真实存在的物理存储器,所有的虚拟地址形成的集合就是虚拟地址空间。如下图：
![程序员眼中的内存](https://upload-images.jianshu.io/upload_images/2959789-a0a756e7c82b43e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统中的实际内存排列：

<br/>

![系统中的内存排列](https://upload-images.jianshu.io/upload_images/2959789-3d3b7ac1278bfa04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  内存是一个很大的，线性的字节数组（平坦寻址）。每一个字节都是固定的大小，由8个二进制位组成。最关键的是，每一个字节都有一个唯一的编号,编号从0开始，一直到最后一个字节。如上图中，这是一个256M的内存，他一共有256x1024x1024=268435456个字节，那么它的地址范围就是 0 ~ 268435455(在内存中，每一个字节（byte）都有一个内存编号，也就是地址，如同现实生活中的地址一样，每一个地址都可存储二进制编码（bit）的数据.)。

&emsp;  `系统中的内存每一个字节都有一个唯一的编号。因此，在程序中使用的变量，常量，甚至函数等数据，当它们被载入到内存中后，都有自己唯一的编号，这个编号就是这个数据的地址，指针就是这样形成的。`


<br/>

-  bit(位):   一个二进制数据0或1，是`1bit`；

-  byte(字节):  存储空间的基本计量单位,`1byte  =  8  bit`;

-  1 字母 = 1 byte = 8 bit;

-  1 汉字 = 2 byte = 16 bit;


# `+ 64 位的编译器各类型字节数显示`
```
//不同的编译器位数，其字节数显示是不同的。通过在程序中打印：
printf("%lu", sizeof(int));   //在Xcode中输出：4，说明是64位的编译器


char：      1个字节

short：     2个字节

int：       4个字节

long：      8个字节

float:      4个字节

double：    8个字节

```
&emsp;   查看操作系统位数，在终端输入：`uname -a` 在末尾有` x86_64`说明是64位的操作系统。





 

<br/>

***
<br/>

> # 指针介绍

代码Demo：
```
include <iostream>

int main(int argc, const char * argv[]) {
    
    std::cout << "<<<<<<<<<<<<<<<<<<    Start\n";

    int num1 = 10;
    char ch1[2] = "A";
    
    std::cout<<"变量名称    变量值 内存地址"<<std::endl;
    std::cout<<"-----------------------"<<std::endl;

    std::cout<<"num1"<<"\t"<<num1<<"\t"<<&num1 <<std::endl;
    std::cout<<"ch1"<<"\t""\t"<<ch1<<"\t"<<&ch1 <<std::endl;



    std::cout << ">>>>>>>>>>>>>>>>>>    End!\n";
    system("pause");
    return 0;
}

```
输出：
![变量地址输出](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a12.png)



```
int a = -12;
char b = M;
float c = 3.14
```
![a、b、c内存存放示意图](https://upload-images.jianshu.io/upload_images/2959789-0ca97c9c1a77e31b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#`变量寻址`
![3.png](https://upload-images.jianshu.io/upload_images/2959789-6060181ae8a79638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```diff
+ 指针的申明：
- 首先必须定义指针的数据类型，并在数据类型后加上“*”符号，再赋予指针名称，即可声明一个指针变量。

- 由于指针属于系统底层的存取功能，因此通过指针可以存取内存中所指向的内存区内容。假如赋予指针错误的地址，而该地址又刚好是系统数据存储的内存区，此时若覆盖（override）该内存区的内容，很可能会造成系统不稳定或者宕机的情况。另外，如果声明指针时未指定初值，就经常会让指针指向未知的内存地址。
```
![指针变量声明](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a13.png)

<br/>
第一种方式：

![第一种方式](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a15.png)


第二种方式：

![指针示例](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a14.png)

&emsp;  第一种方式在声明时即赋予了初值；
&emsp;  第二种方式则在声明时先设置初值为0，日后需要使用指针时再把变量地址赋值给指针变量。在此要特别说明一点，此处的初值0代表NULL，而不是数值0。因此在声明指针变量时，万万不能直接将指针变量的初值设置为数值，这样会使指针变量指向不合法的地址，因而造成不可预期的错误。

例如：`int *piVal = 10; // 不合法语句`
以下的声明也会造成指针变量指向不合法地址，请大家小心：
```diff
! int *piVal;
*piVal = 10; // 不合法语句
```
&emsp;如果指针变量已经事先指向了一个定义或声明过的变量地址，那么程序可以通过`*`（取值运算符，或引用运算符）来存取重新指向此指针变量的数据内容，使用格式如下：

`*指针变量 = 数值; // 此指针变量已指向合法地址`




编译器将变量名和地址进行挂钩，内存中没有存放变量的名字；


&emsp;  **`指针的值实质是内存单元（即字节）的编号，所以指针单独从数值上看，也是整数，他们一般用16进制表示。指针的值（虚拟地址值）使用一个机器字的大小来存储,也就是说,对于一个机器字为w位的电脑而言,它的虚拟地址空间是0~2w － 1 ,程序最多能访问2w个字节。`**
&emsp;  **`所以可以把指针看做是一个变量，其值为另一个变量的地址，即内存位置的直接地址。`**

```
//地址是计算机内存中的某个位置，而指针是专门用来存放地址的特殊类型变量
type *pointer;

type：      指针的基类型；
*   ：      申明指针；
pointer：   指针变量；//作用：是用来存放地址和对地址进行索引
```




#`DEMO图`

![指针各类型介绍](https://upload-images.jianshu.io/upload_images/2959789-c585ca5c7f591da3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;  要掌握指针的四方面的内容：指针的类型、指针所指向的类型、指针的值(或者叫指针所指向的内存区)、指针本身所占据的内存区。

- 指针的类型： 从语法的角度看，把指针声明语句里的指针名字去掉，剩下的部分就是指针的类型；
-  指针所指向的类型：决定了编译器将把那片内存区里的内容当做什么来看待。从语法上看，你只须把指针声明语句中的指针名字和名字左边的指针声明符`*`去掉，剩下的就是指针所指向的类型；

-  指针的值：是指针本身存储的数值，这个值被编译器当作一个地址，而不是一个一般的数值。在64 位程序里，所有类型指针的值都是一个64 位整数，因为64 位程序里内存地址全都是64 位长。`指针的值`代表那个内存地址的开始，长度为`sizeof(数据类型)`的一片内存区。以后，我们说一个指针的值是XX，就相当于说该指针指向了以XX 为首地址的一片内存区域；我们说一个指针指向了某块内存区域，相当于说该指针的值是这块内存区域的首地址。`指针的值`和`指针所指向的类型`是两个完全不同的概念。在上图中，指针所指向的类型已经有了，但由于指针还未初始化，所以它所指向的内存区是不存在的，或者说是无意义的。

-  指针本身所占据的内存区：指针本身占用的内存可以用函数`sizeof(数据类型)`测一下就知道了。在64 位平台里，指针本身占据了8 个字节的长度。指针本身占据的内存这个概念在判断一个指针表达式是否是左值时很有用。



<br/>

```

//普通的整型变量
int p; 

//首先从P 处开始,先与*结合,所以说明P 是一个指针,然后再与int 结合,说明指针所指向的类型为int 型.所以P是一个返回整型数据的指针
int *p;  

//首先从P 处开始,先与[]结合,说明P 是一个数组,然后与int 结合,说明数组里的元素是整型的,所以P 是一个由整型数据组成的数组  
int p[3]; 

//首先从P 处开始,先与[]结合,因为其优先级比*高,所以P 是一个数组,然后再与*结合,说明数组里的元素是指针类型,
//然后再与int 结合,说明指针所指向的类型是整型的,所以P 是一个返回整型数据的指针所组成的数组  
int *p[3]; 

//首先从P 处开始,先与*结合,说明P 是一个指针然后再与[]结合,说明指针所指向的内容是一个数组,
//然后再与int 结合,说明数组里的元素是整型的.所以P 是一个指向整型数据组成的数组的指针  
int (*p)[3]; 

//首先从P 开始,先与*结合,说是P 是一个指针,然后再与*结合,说明指针所指向的元素是指针,然后再与int 结合,说明该指针所指向的元素是整型数据.
//由于二级指针以及更高级的指针极少用在复杂的类型中,所以后面更复杂的类型我们就不考虑多级指针了,最多只考虑一级指针.  
int **p; 

//从P 处起,先与()结合,说明P 是一个函数,然后进入()里分析,说明该函数有一个整型变量的参数,然后再与外面的int 结合,说明函数的返回值是一个整型数据 
int p(int); 

//从P 处开始,先与指针结合,说明P 是一个指针,然后与()结合,说明指针指向的是一个函数,然后再与()里的int 结合,说明函数有一个int 型的参数,
//再与最外层的int 结合,说明函数的返回类型是整型,所以P 是一个指向有一个整型参数且返回类型为整型的函数的指针  
Int (*p)(int); 

//从P 开始,先与()结合,说明P 是一个函数,然后进入()里面,与int 结合,说明函数有一个整型变量参数,然后再与外面的*结合,说明函数返回的是一个指针,
//然后到最外面一层,先与[]结合,说明返回的指针指向的是一个数组,然后再与*结合,说明数组里的元素是指针,然后再与int 结合,说明指针指向的内容是整型数据.
//所以P 是一个参数为一个整数据且返回一个指向由整型指针变量组成的数组的指针变量的函数. 
int *(*p(int))[3]; 

```
 

<br/>
***
<br/>
># 指针的操作

![指针的基本使用](https://upload-images.jianshu.io/upload_images/2959789-6cf5ec4145e30650.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打印：
`a = 10, &a = 0x7ffeefbff4fc`
`b = 0x7ffeefbff4fc, *b = 10, &b = 0x7ffeefbff4f0`
上图可以看到，当打印 c 时就会Crash，报错看图。


```

void contactAddressAndPointer()
{
    int num = 97;
    //定义一个指针pointer
    int *pointer;
    //对  num  取地址(&num, 使用连字号&运算符访问的地址), 赋值给指针变量pointer
    pointer = &num;

    /*
      int num=97;
      //在定义指针pointer的同时将num的地址赋给指针pointer
      int *pointer=&num;
    */

    //pointer：指向的变量num的地址；
    // &pointer：指针pointer的地址，因为指针也是一个变量自己也有地址的；
    //&num：变量 num 的地址；
    printf("地址：\np:%p,  &p:%p,  &num:%p\n\n",pointer, &pointer, &num);

    //*pointer：变量 num 的值；
    //i：变量 i 的值；
    printf("值：\n指针变量*pointer：%d,  num的值：%d\n",*pointer,num);
}

```
打印值：
`地址：
p:0x7ffeefbff57c,  &p:0x7ffeefbff570,  &num:0x7ffeefbff57c`

`值：
指针变量*pointer：97,  num的值：97`


&emsp;  用来保存 指针 的变量，就是指针变量。如果指针变量pointer保存了变量 num的地址，则就说：pointer指向了变量num，也可以说pointer指向了num所在的内存块 ，这种指向关系，在图中一般用 箭头表示。
![1.png](https://upload-images.jianshu.io/upload_images/2959789-4cb8f26486f3d139.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  指针变量pointer指向了num所在的内存块 ，即从地址0028FF40开始的4个byte 的内存块。


<br/>
![变量和指针](https://upload-images.jianshu.io/upload_images/2959789-7e844f7b0c2e5445.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![变量和指针的存放区别](https://upload-images.jianshu.io/upload_images/2959789-c32192f7a48d9d6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>

## 取址
```
    int pointerFunc(int x, int b){
        return x + b;
    }


    int num = 97;
    float scrore = 10.00F;
    int arr[3] = {1, 2, 3};
    int (*ptr)(int, int);
    
    int *p_num = &num;
    float *p_scrore = &scrore;
    int (*p_arr)[3] = &arr; //指向含有3个int元素的数组的指针 
    ptr = *pointerFunc;
    const char *msg = "Hello Pointer";
    
    printf(" p_num: %p, *p_num: %d, &num:%p\n", p_num, *p_num, &num);
    printf(" p_scrore: %p,  *p_scrore: %f, &num:%p\n", p_scrore, *p_scrore, &scrore);
    printf(" p_arr: %p, *p_arr:%p, &arr:%p\n", p_arr, *p_arr, &arr);
    printf(" pointerFunc 地址：%p, &pointerFunc: %p, *pointerFunc:%p\n", pointerFunc, &pointerFunc, *pointerFunc);
    printf(" msg: %p,  *msg:%c,&msg:%p\n", msg, *msg, &msg);
    int n = (*ptr)(10, 5);
    printf(" n = %d", n);

```
打印：
 `p_num: 0x7ffeefbff534, *p_num: 97, &num:0x7ffeefbff534`
 `p_scrore: 0x7ffeefbff530,  *p_scrore: 10.000000, &scrore:0x7ffeefbff530`
 ` p_arr: 0x7ffeefbff54c, *p_arr:0x7ffeefbff54c, &arr:0x7ffeefbff54c`
 ` pointerFunc 地址：0x100001360, &pointerFunc: 0x100001360, *pointerFunc:0x100001360`
 `msg: 0x100001c60,  *msg:H,&msg:0x7ffeefbff508`
 `n = 15`


特殊的情况，他们并不一定需要使用&取地址：
-  数组名的值就是这个数组的第一个元素的地址。
-  函数名的值就是这个函数的地址。
-  字符串字面值常量作为右值时，就是这个字符串对应的字符数组的名称,也就是这个字符串在内存中的地址。 


<br/>

## 解址

&emsp;  实质是：从指针指向的内存块中取出这个内存数据。
&emsp;  对一个指针解地址，就可以取到这个内存数据，解地址 的写法，就是在指针的前面加一个`*`号。

```
    int age = 19;
    int：*p_age = &age;
    *p_age  = 20;  //通过指针修改指向的内存数据
    
    printf("age = %d\n",*p_age);   //通过指针读取指向的内存数据
    printf("age = %d\n",age);

```
打印：
`age = 20`
`age = 20`


![解引用获取值](https://upload-images.jianshu.io/upload_images/2959789-411cc7fdcc8b4ad6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#`注意：`
-  指针所保留的是内存中一个地址，它并不保存指向的数据的值本身。因此，务必确保指针对应一个已经存在的变量或者一块已经分配了的内存；

-  星号有两种用途：
    * 用于创建指针：`int *myPointer = &myInt;`
    * 对指针进行解引用：`*myPointer = 3998;`






<br/>

##指针之间的赋值
&emsp;  指针赋值和int变量赋值一样，就是将地址的值拷贝给另外一个。指针之间的赋值是一种浅拷贝，是在多个编程单元之间共享内存数据的高效的方法。

```
//通过指针 p1 、 p3 都可以对内存数据 num 进行读写，如果2个函数分别使用了p1 和p3，那么这2个函数就共享了数据num。
int num = 10;
int* p1  = & num;
int* p3 = p1;


printf("num值没改变：num= %d, *p1= %d, *p3= %d", num, *p1, *p3);
    
num = 100;
printf("\n\nnum值改变后：num= %d, *p1= %d, *p3= %d", num, *p1, *p3);

```
输出值：
`num值没改变：num= 10, *p1= 10, *p3= 10`

`num值改变后：num= 100, *p1= 100, *p3= 100`

![浅拷贝示意图](https://upload-images.jianshu.io/upload_images/2959789-947423062950b939.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
## 指针的属性

```
int main(void)
{
    int num = 97;
    int *p1  = &num;
    char* p2 = (char*)(&num);

    printf("%d\n",*p1);    //输出  97
    putchar(*p2);          //输出  a
    return 0;
}
```

`指针的值`：很好理解，如上面的num 变量 ，其地址的值就是0028FF40 ，因此 p1的值就是0028FF40。数据的地址用于在内存中定位和标识这个数据，因为任何2个内存不重叠的不同数据的地址都是不同的。
`指针的类型`：指针的类型决定了这个指针指向的内存的字节数并如何解释这些字节信息。一般指针变量的类型要和它指向的数据的类型匹配。
 
 
由于num的地址是0028FF40，因此p1  和  p2的值都是0028FF40
`*p1`  :  将从地址0028FF40
开始解析，因为p1是int类型指针，int占4字节，因此向后连续取4个字节，并将这4个字节的二进制数据解析为一个整数 97。
`*p2`  :  将从地址0028FF40 开始解析，因为p2是char类型指针，char占1字节，因此向后连续取1个字节，并将这1个字节的二进制数据解析为一个字符，即`a`。
 
同样的地址，因为指针的类型不同，对它指向的内存的解释就不同，得到的就是不同的数据。


<br/>

***
<br/>

下一篇：[指针(II)](https://www.jianshu.com/p/95c787a634f9)

<br/>

参考资料：
[C 语言指针详解](https://www.cnblogs.com/lulipro/p/7460206.html)




 
