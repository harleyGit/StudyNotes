><h2 id=''></h2>
- [**Linphone官方API文档**](https://www.linphone.org/technical-corner/liblinphone)
- [**知识点**](#知识点)
- [**语法**](#语法)
	- [输出<<](#输出<<)
	- [void指针(void*)](https://blog.csdn.net/qq_39583450/article/details/109715890)
- [**参考资料**]()
	- [Linphone使用说明](https://www.mptablet.com/post/12045.html)
	- [SDP消息格式](https://cxybb.com/article/aiwusheng/104723834)
	- [SIP技术介绍](http://www.h3c.com/cn/d_200805/605897_30003_0.htm)


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

