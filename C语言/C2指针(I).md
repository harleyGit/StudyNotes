- **空指针**
	- NULL
	- nullptr
	- 0





<br/>

***
<br/>


># 空指针

- **NULL**


&emsp; 在C语言中，我们使用NULL表示空指针，也就是我们可以写如下代码：
	
```
int *i = NULL;
foo_t *f = NULL;
```

<br/>

实际上在C语言中，NULL通常被定义为如下：

```
define NULL ((void *)0);
```

&emsp; 也就是说NULL实际上是一个void *的指针，然后吧void *指针赋值给int *和foo_t *的指针的时候，隐式转换成相应的类型。而如果换做一个C++编译器来编译的话是要出错的，因为C++是强类型的，void *是不能隐式转换成其他指针类型的，所以通常情况下，编译器提供的头文件会这样定义NULL：

`#ifdef __cplusplus ---简称：cpp c++ 文件`

<br/>

`#define NULL 0`

<br/>

`#else`

<br/>

`#define NULL ((void *)0)`

<br/>

`#endif`


<br/>
<br/>

- **nullptr**



<br/>
<br/>


- **0**

&emsp; C++中不能将void *类型的指针隐式转换成其他指针类型，而又为了解决空指针的问题，所以C++中引入0来表示空指针（注：0表示，还是有缺陷不完美），这样就有了类似上面的代码来定义NULL。实际上C++的书都会推荐说C++中更习惯使用0来表示空指针而不是NULL，尽管NULL在C++编译器下就是0。