
># 预处理指令

- `__LINE__`宏可打印出此宏所出现的行号;

- `__FILE__`宏可打印出正在被编译的文件路径与名称;

- `#if`条件编译指令类似于if条件语句，当此条件成立时，会执行此程序语句区块的程序代码，如果不成立，就略过不执行;

- `#endif`指令是搭配#if等条件编译指令来使用的，作用类似于`}`大括号，有结束的功能。声明语法如下：
```
#if条件表达式


￼    程序语句区块￼   


#endif
```

- `#else`条件编译指令，必须搭配`#if`指令，形成和`if else`条件语句类似的功能，当`#if`指令不成立时会跳过程序语句区块一，执行`#else`下面的程序语句区块二，声明语法如下：
```
￼     #if条件表达式

￼         程序语句区块一

￼     #else条件表达式

￼     程序语句区块二

￼     #endif
```



- `#elif`指令在C++中类似于`if else if`条件语句中的`else if`语句。可以针对多种编译条件来进行验证，只要其中的一个条件成立，就执行该条件的程序区块。
```

  	 #if条件表达式一

  ￼      程序语句区块一

￼     #elif条件表达式二

￼     	程序语句区块二

￼     #elif条件表达式三

￼     	程序语句区块三
￼     	…

￼     #endif
```


- `#ifdef`条件编译指令的条件判断式是由单一的标识符组成的，通过判断`预处理指令#define`是否定义了此标识符来决定是否编译区块内的程序语句。声明语法如下：
```
￼     #ifdef宏名称

￼         程序语句区块

￼     #endif
```

- `#ifndef`条件编译指令与`#ifdef`条件编译的作用正好相反，当宏名称没有被定义时才会执行`#ifndef`的程序语句区块部分。声明语法如下：
```
￼     #ifndef宏名称

￼         程序语句区块

￼     #endif
```


<br/>

***
<br/>


># 预处理命令介绍
![预处理命令简介](https://upload-images.jianshu.io/upload_images/2959789-9224fafa64f6a6bb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#`预处理命令简单使用`
![ifndef 使用](https://upload-images.jianshu.io/upload_images/2959789-725cfe5398296ca1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解释：
![上图预处理器命令意义](https://upload-images.jianshu.io/upload_images/2959789-9e6646d6415016ae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>

***
<br/>
