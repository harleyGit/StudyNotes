- [**知识点**](#知识点)



<br/>

***
<br/>


> <h1 id="知识点">知识点</h1>

<br/>

[**std::**](https://blog.csdn.net/Calvin_zhou/article/details/78440145)是个名称空间标识符，C++标准库中的函数或者对象都是在命名空间std中定义的，所以我们要使用标准库中的函数或者对象都要用std来限定。


<br/>

[**Vector向量**](https://www.runoob.com/w3cnote/cpp-vector-container-analysis.html)是一个封装了动态大小数组的顺序容器（Sequence Container）。跟任意其它类型容器一样，它能够存放各种类型的对象。可以简单的认为，向量是一个能够存放任意类型的动态数组。

<br/>

**(::)范围解析运算符**：在前面的类声明范例中，我们都把成员函数定义在类内。事实上，类中成员函数的程序代码不一定要写在类内，我们也可以在类中事先声明成员函数的原型，然后在类外面再编写成员函数的程序代码部分。如果是在类外面编写成员函数，只要在外部定义时函数名称前面加上类名称与范围解析运算符（::）即可。范围解析运算符的主要作用就是指出成员函数所属的类。


<br/>

**#include**

```

 #include并不是什么申请指令，只是将指定文件的内容，原封不动的拷贝进来
 *.h文件做的是类的声明，包括类成员的定义和函数的声明
 *.cpp文件做的类成员函数的具体实现（定义）
 在*.cpp文件的第一行一般也是#include"*.h"文件，其实也相当于把*.h文件里的东西复制到*.cpp文件的开头
 */
```