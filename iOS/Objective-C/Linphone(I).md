><h2 id=''></h2>
- [**概念**](#概念)
	- [IP话机](#IP话机)
	- [SIP协议](#SIP协议)
		- [IP电话协议族分类](#IP电话协议族分类)
		- [SIP协议功能](#SIP协议功能)
		- [SIP网络构成](#SIP网络构成)
		- [SIP协议采用的地址](#SIP协议采用的地址)
		- [SIP消息的功能类型](#SIP消息的功能类型)
		- [SIP消息的结构和重要头域](#SIP消息的结构和重要头域)
		- [SIP消息流程](#SIP消息流程)
- [**Linphone官方API文档**](https://www.linphone.org/technical-corner/liblinphone)
	- [资料](#资料)
- [**知识点**](#知识点)
- [**语法**](#语法)
	- [输出<<](#输出<<)
	- [void指针(void*)](https://blog.csdn.net/qq_39583450/article/details/109715890)
- [**参考资料**]()
	- [Linphone使用说明](https://www.mptablet.com/post/12045.html)
	- [Linphone SDK编译说明](https://gitlab.linphone.org/BC/public/linphone-sdk/blob/master/README.md)
	- [Linphone C API文档](https://download.linphone.org/releases/docs/liblinphone/latest/c/)
	- [SDP消息格式](https://cxybb.com/article/aiwusheng/104723834)
	- [SIP技术介绍](http://www.h3c.com/cn/d_200805/605897_30003_0.htm)



<br/>

***
<br/>


><h1 id='概念'>概念</h1>


<br/>

> <h2 id='IP话机'>IP话机</h2>

&emsp; IP电话机是以IP网络协议为主要通信协议，以网口（或WIFI）为主要接口，具备拨号、通话功能的独立电话设备，通常需要配合IPPBX电话交换机使用或者配合IMS使用。IP话机常用的通信协议有SIP， IMS， MGCP, SCCP等，其中SIP较为常用,也被称作VoIP。



<br/>
<br/>

> <h2 id='SIP协议'>SIP协议</h2>

> <h3 id='IP电话协议族分类'>IP电话协议族分类</h3>


IP电话协议族分类图:

![ios_oc1_32.png](./../../Pictures/ios_oc1_32.png)

<br/>

SIP和H.323协议比较:

![ios_oc1_33.png](./../../Pictures/ios_oc1_33.png)



<br/>
<br/>

> <h3 id='SIP协议功能'>SIP协议功能</h3>

SIP协议功能:

![ios_oc1_34.png](./../../Pictures/ios_oc1_34.png)



<br/>
<br/>

> <h3 id='SIP网络构成'>SIP网络构成</h3>

SIP网络构成:

![ios_oc1_35.png](./../../Pictures/ios_oc1_35.png)

- **Use Agets:**
	- 用户代理,是一个软终端或者是一个支持SIP协议	的电话;
	- UA:对接收到的行为进行代理,发送到SIP网络中;
	- 一个发起和终止会话的实体,包含2个功能实体:
		- User Agent Clients(UAC)-发起SIP事务请求的功能实体
		
		![ios_oc1_36.png](./../../Pictures/ios_oc1_36.png)
	- User Agent Server(UAS)-接收SIP事务请求的功能实体
	
		![ios_oc1_37.png](./../../Pictures/ios_oc1_37.png)
		
		这个相应的动作指的是比如:电话振铃、网页上显示有新呼叫.
		
		![ios_oc1_38.png](./../../Pictures/ios_oc1_37.png)
		
		

<br/>

- **Proxy Server - 代理服务器**
	- 对收到的请求消息进行翻译和处理后,传递给其他的服务器;
	- 为其他的客户代理,进行SIP消息的转接和转发的功能;
	- 对SIP请求及响应进行路由;


<br/>

- **Location Server -位置服务器**
	- 是一个数据库,用于存放终端用户当前的位置信息,为SIP重定向服务器或代理服务器提供被叫用户可能的位置信息


<br/>

- **Redirect Server - 重定向服务器**
	- 将用户新的位置返回给呼叫方,呼叫方可根据得到的新位置重新呼叫;
	- Redirect Server只是对请求消息进行响应,不产生请求消息


<br/>

- **Registrar Server - 登记服务器**
	- 接受REGISTER 请求完成用户地址的注册
	- 可以支持鉴权的功能


<br/>

**通信流程:**

![ios_oc1_44.png](./../../Pictures/ios_oc1_44.png)

<br/>
<br/>

> <h3 id='SIP协议采用的地址'>SIP协议采用的地址</h3>


![ios_oc1_39.png](./../../Pictures/ios_oc1_39.png)





<br/>
<br/>

> <h3 id='SIP消息的功能类型'>SIP消息的功能类型</h3>

![ios_oc1_40.png](./../../Pictures/ios_oc1_40.png)

![ios_oc1_41.png](./../../Pictures/ios_oc1_41.png)

<br/>

消息请求回应状态码:

![ios_oc1_42.png](./../../Pictures/ios_oc1_42.png)

<br/>

扩展请求消息:

![ios_oc1_43.png](./../../Pictures/ios_oc1_43.png)




<br/>
<br/>

> <h3 id='SIP消息的结构和重要头域'>SIP消息的结构和重要头域</h3>



<br/>
<br/>

> <h3 id='SIP消息流程'>SIP消息流程</h3>





<br/>
<br/>

> <h3 id=''></h3>




<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>

> <h3 id=''></h3>






<br/>
<br/>

> <h2 id=''></h2>




<br/>
<br/>

> <h2 id=''></h2>



<br/>
<br/>

> <h2 id=''></h2>






<br/>
<br/>

***
<br/>

><h1 id='Linphone官方API文档'>Linphone官方API文档</h1>

<br/>

><h2 id='资料'>资料</h2>

[linphone-iphone的代码](https://github.com/BelledonneCommunications/linphone-iphone)

[Linphone iOS 编译](https://www.jianshu.com/p/ec5cfb72c9e7)

[Linphone集成到自己的iOS项目](https://string.quest/read/10861716)







<br/>
<br/>

***
<br/>

><h1 id='知识点'>知识点</h1>
- [X3DH加密协议](https://www.aechina.io/wiki/1422.html)






<br/>

***
<br/>

><h1 id='语法'>语法</h1>

<br/>

><h2 id='输出<<'>输出<<</h2>

C++引入了`ostringstream、istringstream、stringstream`这三个类，要使用他们创建对象就必须包含<sstream>头文件:
- istringstream类用于执行C++风格的串流的输入操作。  
- ostringstream类用于执行C风格的串流的输出操作。  
- strstream类同时可以支持C风格的串流的输入输出操作。 

```
#include <sstream>

int main(int argc, char* argv[])
{
	std::ostringstream ostr;
	ostr << "string:"<< "str_test"
		 << " int:" << 123
		 << " char:" << 'a'
		 << " bool:" << true;

	std::string s = ostr.str();

	printf("%s\n", s.c_str());

	return 0;
}
```

运行输出:

```
string:str_test int:123 char:a bool:1
```

