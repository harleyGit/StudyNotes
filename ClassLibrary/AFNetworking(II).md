

- **类结构图**
	- 网络请求流程图 
- **属性**
- **多路复用**



<br/>

***
<br/>

># 属性

**`@property (nonatomic, assign) BOOL removesKeysWithNullValues;`**

&emsp;  在`AFNetWorking`只要把这个`removesKeysWithNullValues=YES.`后台返回的JSON数据中存在空的键值对,将会被自动删除,可以避免空值做操作,造成崩溃问题。


<br/>

***
<br/>

># 类结构图

![AF类结构图 <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet1.png)

<br/>

> **发送请求**

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet2.png)


<br/>
<br/>



> **接收到响应**

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet3.png)


<br/>
<br/>



> **进度条模块**

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet4.png)


<br/>
<br/>



> **认证模块**

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet5.png)

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet6.png)





<br/>

***
<br/>




># 多路复用

&emsp; AFURLSessionManager和NSURLSession是一对一的关系，AFURLSessionManager会在初始化的时候创建对应的NSURLSession。同样AFURLSessionManager在注释中写明了可以提供一个配置好的manager单例来全局复用。

&emsp; 在iOS9.0之后，session支持http2.0。http2.0的一个特点就是多路复用，可以减少访问同一个服务器时，重新建立tcp连接的耗时和资源。

&emsp; 在不同的场景使用不同的session，比如：一个session处理普通的请求，一个session处理background请求；一个sesison处理浏览器公开的请求，一个session专门处理隐私请求等等场景。












