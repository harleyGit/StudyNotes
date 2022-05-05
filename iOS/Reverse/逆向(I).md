
><h2 id=''></h2>
- [**禁止调试和破解**](#禁止调试和破解)



<br/>

***
<br/>


><h1 id='禁止调试和破解'>禁止调试和破解</h1>


&emsp; Mac->Xcode->Xcode上集成了LLDB-> 问题:真机上上是如何进行应用程序的开发的呢?

&emsp; 实际上是LLDB上追加了一个debugServer,这个debugServer附加trace进程,通过这个trace进程我们就可以在我们的真机上进行调试了.Xcode运行的时候,是默认开启了debugServer.

&emsp; 根据这个我们可以在我们的程序上禁止别人对我们开发的程序进行调试,可以通过`trace()函数`,但是这个函数我们是我无法直接使用的.需要我们先把头文件`MyTraceHeader.h`拖到我们的项目文件里.

```
//
trace(PT_DENY_ATTACH, 0, 0,);
```

&emsp; 通过这个函数,用户是可以正常使用我们的函数,但是其他像调试我们的人是不行的.只要他通过xcode运行在真机或者模拟器就会直接闪退.原因是trace这个函数检测到debugServer后会直接杀掉trace进程.


<br/>

><h2 id=''></h2>




<br/>

***
<br/>


><h1 id=''></h1>


<br/>

><h2 id=''></h2>





<br/>

***
<br/>


><h1 id=''></h1>


<br/>

><h2 id=''></h2>





<br/>

***
<br/>


><h1 id=''></h1>


<br/>

><h2 id=''></h2>



<br/>

***
<br/>


><h1 id=''></h1>


<br/>

><h2 id=''></h2>
