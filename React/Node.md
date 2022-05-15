> <h2 id=''></h2>
- [**Node介绍**](#Node介绍)
	- [AMD简介](#AMD简介)
	- [CommonJS简介](#CommonJS简介)
	- [ES 6简介](#ES6简介)
	- [异步I/O和事件驱动](#异步I/O和事件驱动)
- [**HTTP简介**](#HTTP简介)





<br/>

***
<br/>

> <h1 id='Node介绍'>Node介绍</h1>


常见的后端开发语言和技术有多种：
- 基于Java的Spring（https://spring.io/）；
- 基于Ruby的Ruby on Rails（http://rubyonrails.org/）；
- 基于Python的Django（https://www.djangoproject.com/）；
- 基于PHP的Laravel（https://laravel.com/）；
- 除此之外还有一些常用的建站技术，例如ASP.NET（https://www.asp.net/）。


&emsp; 相比其他后端技术，我们应该如何理解Node及其作用呢？众所周知，JavaScript是一门脚本语言，脚本语言都需要一个解析器才能运行。对于HTML页面中的JavaScript程序，`浏览器充当了解析器的角色`。而对于独立运行的JavaScript代码，`Node就是它的解析器`。

&emsp;简单来说，Node就是一个让JavaScript运行在服务器端的开发平台，它让JavaScript成为脚本语言世界的“一等公民”，在后端与Java、Ruby、Python和PHP“平起平坐”。Node基于V8引擎开发，V8引擎执行JavaScript程序的速度非常快，性能也非常好.


<br/>

&emsp; **目前流行的JavaScript模块化规范有AMD、CommonJS和ES 6模块系统。**

<br/>
<br/>

> <h2 id='AMD简介'>AMD简介</h2>


&emsp; AMD（Asynchronous Module Definition）规范，即异步模块定义。它采用异步方式加载模块，模块的加载不影响后面语句的运行。所有依赖该模块的语句都定义在一个回调函数中，等加载完成之后，这个回调函数才会运行。RequireJS是对这个异步加载模块方法的实现。它的基本思想是：·通过define方法将代码定义为模块；·通过require方法实现代码的模块加载。使用require.js的示例项目结构如下：\

![目录结构](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/node0.png)

<br/>

index.html Code

![index.html](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/node1.png)

<br/>


math.js Code：

![math.js](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/node2.png)

RequireJS源码可以从[官方网站下载](https://requirejs.org/)



<br/>
<br/>

> <h2 id='CommonJS简介'>CommonJS简介</h2>

&emsp; CommonJS是JavaScript模块化的一种规范,根据这个规范，每一个文件都是一个模块，其内部定义的变量是属于这个模块的，不会对外暴露，也就是说不会污染全局变量。\

&emsp; CommonJS的核心思想就是通过require()方法同步加载所要依赖的其他模块，然后通过exports或者module.exports来导出需要暴露的接口。
使用CommonJS规范的示例项目结构如下：

```
01  .
02  ├── index.js
03  └── math.js
04  
05  0 directories, 2 files
```


其中，index.js代码如下：

```
01  var math = require('./math.js');
02
03  console.log(math.add(10, 20)); // 30
```


math.js代码如下：

```
01  var add = function (a, b) {
02      return a + b;
03  };
04  
05  module.exports.add = add;
```
然后执行index.js脚本：

node index.js

输出结果如下：

```
30
```



<br/>
<br/>

> <h2 id='ES6简介'>ES 6简介</h2>


&emsp; 上面介绍了模块加载方案AMD和CommonJS规范，而ES 6的模块化方案才是真正的标准规范。

&emsp; ES 6在语言标准的层面上实现了模块功能，它完全可以取代AMD和CommonJS规范，成为浏览器和服务器通用的模块化解决方案。但是ES 6目前无法在所有的浏览器中执行，需要通过`Babel将不被支持的import语法编译为当前受到广泛支持的require语法`。

&emsp; ES 6模块的设计思想是尽量静态化，使得编译而非运行时就能确定模块的依赖关系，以及输入和输出的变量。

使用ES 6模块化的示例项目结构如下：


```
01  .
02  ├── .babelrc
03  ├── index.js
04  └── math.js
05  
06  0 directories, 3 files
```

首先需要配置Babel文件.babelrc，代码如下：

```
01  {
02      "presets": [
03          "@babel/preset-env"
04      ]
05  }
```

提示：在Linux和macOS系统中，以“.”开头的文件是隐藏文件，如果使用ls命令查看的话，需要添加参数-a。

其中，index.js代码如下：

```
01  import { add } from './math.js';
02  
03  console.log(add(10, 20));
```

math.js代码如下：


```
01  function add(x, y) {
02      return x + y;
03  }
04  
05  export { add };
```

在运行代码前还需要安装Babel命令行工具，以支持上述ES 6模块化的语法。命令如下：

`npm install --save-dev @babel/core @babel/node @babel/preset-env`

执行index.js脚本：

```
npx babel-node index.js
```

输出结果如下：

```
30
```



> <h2 id='异步I/O和事件驱动'>异步I/O和事件驱动</h2>


**单线程:**Node的运行时（Runtime）环境基于V8引擎。V8引擎是Chrome浏览器中的JavaScript代码解析引擎，其最大的特点是单线程，因此Node也是单线程。

对于耗时的操作,我们应该怎么办? 这就需要引出Node的另一个知识点,那就是**异步**


<br/>

>**异步下的几个概念:**
- 阻塞I/O（blocking I/O）:当需要执行I/O操作读取硬盘和网络等数据时，线程会被阻塞，直到要读取的数据全部准备好返回给用户，这时线程才会解除阻塞状态。


>- 非阻塞I/O（non-blocking I/O）:当需要执行I/O操作读取硬盘和网络等数据时，线程可以在发起I/O处理请求后，不用等请求完成，继续做其他事情。但是程序如何知道要读取的数据已经准备好了呢？除了存在效率问题的轮询方法外，现在通常的做法是I/O多路复用的方式，即用一个阻塞函数同时监听多个文件描述符，当其中有一个文件描述符准备好了，就立刻返回。Linux系统提供了select、poll和epoll等实现I/O多路复用的功能。



>- 同步IO（synchronous I/O）:同步I/O做I/O操作的时候会阻塞线程，因此阻塞I/O、非阻塞I/O及I/O多路复用都是同步I/O。


>- 异步IO（asynchronous I/O）:异步I/O做I/O操作的时候不会造成任何阻塞。


&emsp; 非阻塞I/O都不阻塞了为什么不是异步I/O呢？其实，当非阻塞I/O准备好数据以后还是要阻塞进程去内核读取数据的，因此不算是异步I/O。



<br/>

> **事件驱动**

&emsp; 事件驱动就是通过监听事件的状态变化来做出相应的操作。例如，读取一个文件，文件读取完毕或者文件读取错误都会触发对应的状态，然后调用对应的回调函数进行处理。示例代码如下：

```
01  var fs = require('fs');
02  
03  fs.readFile('./test.txt', { 'encoding': 'utf8' }, function (err, data) {
04      if (err) {
05          console.log(err);
06      } else {
07          console.log(data);
08      }
09 });
10  
11 console.log('event driver');
```





<br/>

***
<br/>


> <h1 id='HTTP简介'>HTTP简介</h1>

![请求模式](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/node3.png)


<br/>

>**Content-Type头常用MIME类型：**
- application/json类型，
- text/plain：普通文本；
- text/html：HTML格式的文本；
- image/png：png格式的图片；
- application/pdf：PDF文档；
- application/x-www-form-urlencoded：form表单数据格式；
- multipart/form-data：表单上传文件。


<br/>
<br/>



> <h2 id=''></h2>



<br/>
<br/>







<br/>

***
<br/>


> <h2 id=''></h2>



<br/>
<br/>


<br/>

***
<br/>


> <h2 id=''></h2>



<br/>
<br/>