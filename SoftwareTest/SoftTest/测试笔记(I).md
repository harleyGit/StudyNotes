> <h2 id=''></h2>
- [快捷键](#快捷键)
- [视频笔记](#视频笔记)
	- [简单与非门](#简单与非门)
	- [质量模型](#质量模型)
	- [如何开展软件测试的工作](#如何开展软件测试的工作)
	- [用例的八大要素](#用例的八大要素)
	- [缺陷定义](#缺陷定义)
	- [接口测试](#接口测试)
	- [HTTP协议简介](#HTTP协议简介)
	- [接口测试流程](#接口测试流程)
- [**MySQL数据库**](#MySQL数据库)
	- [数据库安装](#数据库安装)
	- [MySQL数据库启动](#MySQL数据库启动)
	- [启动问题解决](#启动问题解决)
	- [数据库操作](#数据库操作)
- [面试题](#面试题)




<br/>

***
<br/>


![文件放置](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_1.png)


<br/>

><h1 id='快捷键'>快捷键</h1>

- 小点：command+L
- 链接[](): command+K
- 换行： `<br/>`
- 加粗`** **`：command+B
- 图片显示：
	- 图片前缀：![图片名称](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/)
	- 图片名称： xxx.png 或者 xxx.jpg 或者 xxx.webp 。。。很多格式图片
	- 组合成图片： ![图片名称（可以起任何名字）](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/xxx.png)


<br/>

***
<br/>

> <h1 id='视频笔记'>视频笔记</h1>

> <h3 id='简单与非门'>简单与非门</h3>

**或（||）：** 

```
// || 或连接符
yes || yes || yes || no = yes

Yes || yes || yes = yes 
```

<br/>

**与（&&）:**

```
// && 连接符

yes && yes && yes && no = no

yes && yes && yes = yes

```


<br/>

**认识测试行业**

![st_2](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_2.png)

<br/>

> <h2 id='质量模型'>质量模型</h2>

```
功能、性能、兼容、易用、安全、可靠性、移植性、维护性

```


<br/>
<br/>

> <h2 id='如何开展软件测试的工作'>**如何开展软件测试的工作**</h2>


1：需求评审

2：编写测试计划

3:用例设计

4：用例执行

5：缺陷管理

6:测试报告


<br/>
<br/>

> <h2 id='用例的八大要素'>用例的八大要素 </h2>


1、用例编号 

2、用例标题 

3、模块/项目 

4、优先级（p0～p4，p0最高）

5、前置条件 

6、测试步骤 

7、测试数据 

8、预期结果


<br/>
<br/>


**解决穷举场景：等价类划分法**

正向用例：一条尽可能覆盖多条

逆向用例：每一条数据，都是一条单独用例

<br/>
<br/>


**解决边界限制问题：**

<br/>
<br/>

**优化策略：**

```
结论：7个点优化为5个点
上点：必选（不考虑区间开闭）
内点：必选（建议选择中间范围）
离点：开内闭外（考虑开闭区间，开区间选包含的点，闭区间选不包含的点）

```

<br/>
<br/>



> <h2 id='缺陷定义'>缺陷定义</h2>

软件中存在的各种问题，称为缺陷，简称bug


<br/>
<br/>


**缺陷标准：**	
	
1：少功能 

2:功能错误 

3：多功能 

4：缺少隐形功能 

5:易用性（软件测试人员专业角度）
	
	
<br/>
<br/>


**缺陷产生的原因：**
	
```
1:需求阶段：需求描述不易理解，有歧义，错误等
2:设计阶段：设计文档存在错误或缺陷
3:编码阶段：代码出现错误
4：运行系统：软硬件系统本身故障导致软件缺陷
```
	
	
<br/>
<br/>

**描述缺陷重点：**	

```
缺陷标题，前置条件，复现步骤，预期结果，实际结果，附近备注
```

<br/>
<br/>

**提交缺陷信息**

```
指派人，缺陷等级，修复优先级，类型，状态（统计缺陷）
```

<br/>
<br/>

**缺陷管理工具**

```


```

测试所需技能：

- 使用jira记录不同类型bug
- 分析接口文档：使用jmeter，postman，python 等工具进行接口测试或压力测试
- 使用Charles 进行抓包
- Mysql,MongoDB数据库进行增删改查


<br/>
<br/>

> <h2 id='接口测试'>接口测试</h2>


概念：

```
接口测试就是，对系统或组件之间的接口进行测试，校验传递的数据正确性和逻辑依赖关系的正确性！
```

原理：

```
接口测试主要针对的测试目标--服务器
怎么测
   模拟客户端，向服务器发送请求
用什么测
   工具：fiddler ,postman ,jester 
   代码：python+unittest框架+requests框架
测什么
   测试服务器针对客户端请求，回发的响应数据是否与预期结果一致！
 ```

<br/>
<br/>

**特点**

```
符合质量控制前移的理念
可以发现一些页面操作发现不了的问题
接口测试低成本高收益
接口测试是从用户的角度对系统进行检测
```

<br/>
<br/>

**实现方式：**	

```
工具：JMeter,postman,fiddler
代码：python+Requests+UnitTest
```

<br/>
<br/>

**什么是自动化接口测试**

```
借助工具，代码，模拟客户端发送请求给服务器，借助断言自动判断预期结果和实际结果是否一致！
```


<br/>
<br/>

> <h2 id='HTTP协议简介'>**HTTP协议简介**</h2>

```
HTTP:
(超文本传输协议)，是一个基于请求与响应模式的，应用层的协议，也是互联网上应用最为广泛的一种网络协议
```

特征：

```
 1，支持客户端/服务器模式
 2，简单快速
 3，灵活
 4，无连接
 5，无状态
 ```
 
<br/>
<br/>

**URL格式**
 
 概念：（Uniform Resource Locator)统一资源定位符
 作用：在网络环境中，唯一的定义一个数据资源
 
 
<br/>
<br/>

**URL语法格式组成**
 
 ```
 协议：http,规定数据传输的方式
 域名（IP)在网络环境中找到主机--用：//与协议隔开
 端口（port）：（常省略）在网络主机上，标识一个进程（应用程序）--用：与域名
 资源路径：标识网络资源（文件、图片、音频量、变量...）--用：/与端口隔分
 查询参数：传递给资源路径对应的数据。--用？与资源路径隔分。查询参数内部 用&隔分多个kv对
 ```
 
 
<br/>
 
**HTTP请求**
 
作用：
 
 ```
 客户端（app，浏览器），发送请求给服务器时，使用的协议-- http请求协议
 规定发送给服务器的数据传输的语法格式
 ```
 
整体格式
 
```
请求行：http请求第一行。请求方法（空格）URL(空格）协议版本
请求头：语法格式：k：v
  User-Agent：描述请求发送端的浏览器类型
  Content-Type：描述请求体的数据类型！
空行：代表http请求头结束
请求体：请求发送时携带的数据。数据类型Content-Type的值！
 post和put有请求体
 get和delete没有请求体

```

![图片名称（可以起任何名字）](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_5.png)
 
 
 
 fiddler抓包验证：
 
![图片名称（可以起任何名字）](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_6.png)

<br/>

![图片名称（可以起任何名字）](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_7.png)



 
 <br/>
 
 
 ```
 请求行
   http请求方法：大小写无所谓
       GET:查询
       POST：添加（常用在登陆）
       PUT：修改
       DELETE：删除
 请求头
   语法格式：k：v
      User-Agent：产生请求的浏览器类型
      Content-Type：请求体数据的类型
      application/json：JSON数据格式
      application/x-www- form-urlencoded：form表单数据
 请求体
  GET、DELETE方法没有情求体
  PUT、POST方法有请求体
  数据值的组织形式、受Content-Type的值影响
  ```
  
**HTTP响应**

作用：

```
   服务器端，针对客户端发送的http请求，回发响应数据--http应答！
   规定回发给客户端的数据组织格式
```
   
整体格式：

```
  响应行（状态行）：协议版本（空格）状态码（空格）状态描述
  相应头：语法格式：k：v
     Content-Type：描述响应体中数据类型
  空行：代表响应头结束
  相应体：绝大多数不为空（请求成功；回发数据，失败：回发错误信息）
     数据类型受Content-Type值影响。
 ```


![整体格式](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_8.png)


<br/>

状态行

```
状态码：
   1xx：代表指示信息。表请求已经被接收，等待继续处理。
   2xx：代表请求成功被处理、接收。常见：200、201
   3xx：重定向，待访问的资源，需求重新指定路径访问。
   4xx：代表客户端错误。常见：404、403
   5xx：访问器端错误。
   状态码描述：一般与状态码唯一对应。200--OK；404--file not found
   
响应头
   语法格式：k：v
   Concent-Type：值为响应体的数据类型
   Concent-Length：响应体的大小。可以不写，浏览器会自动求取。一旦写，必须准确！
响应体
   回发给客户端的消息内容，常见的有html网页、xml、json
   ```
 
<br/> 
   
**传统风格接口**

```
特点：
   请求方法，只使用get和post即可
   URL不唯一。同一个操作可以对应不同的URL
   状态码的使用较单一。200最常见
   ```

<br/>
   
**RESTFUI**

```
特点：
  1，每一个URL代表一种资源；
  2，客户端和服务器之间，传递这种资源的某种表现层；
  3，客户端通过四个HTTP动词（GET、post、delete、put），对服务器端资源进行操作，实现“表现层状态转化”
  4，接口之间传递的数据最常用格式为JSON.
```

<br/>


> <h2 id='接口测试流程'>接口测试流程</h2>
  


```
1.分析需求，产生需求文档（文档）。
2.开发产生接口文档。解析接口文档 。
3.产生测试用例（送审）
4.执行测试用例
  。工具：postman、jmeter
  。代码：python+Rrequests+UnitTest
5.提交、跟踪缺陷
6.生成测试报告
7.（可选）接口自动化持续集成！
```

<br/>

**接口文档**

什么是接口文档

```
由开发人员编写，描述接口信息的文档。开发团队按接口文档进行开发工作，并要一直维护遵守。
```

作用

```
1，能够让前端开发与后台开发人员更好的配合，提高工作效率。（有一个统一参考的文件）
2，项目迭代或者项目人员更迭时，方便后期人员维护
3，方便测试人员进行接口测试
```

<br/>

**展现形式**

```
Word文档形式
Excel表现式形式
Pdf文档形式
```

* 结构

```
基本信息：
   资源路径（协议和域名在“系统信息”中）
   请求方法
   接口描述
请求参数：
   请求头：Content-Type。描述请求体的数据类型！
   请求体：实现该接口使用的数据及对应类型。
返回数据：
   状态码200
   错误码（自定义状态码）
      码值
      描述信息
``` 

 
<br/>
<br/> 
   
   
> <h2 id='接口文档解析'>接口文档解析</h2>


```
接口文档的解析本质：在接口文档中，找出http请求所需要的数据信息
   主要包含：请求方法、URL、请求头、请求体、响应状态码、描述。
 ```  

<br/>

**接口用例设计** 

接口测试的测试点


![接口用例设计）](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_9.png)



* 设计方法与思路

```
与手工设计相同之处：
   手工测试对应的功能测试点，与接口测试对应的功能完全一致。
与手工设计不同之处：
   1，手工测试，测写入到输入框中的数据是否正确。接口测试测参数对应的参数值是否正确。
   2，接口测试，不单单针对参数值进行，还可以针对参数本身进行测试。
         正向参数：
            必选参数： 所有的必选（必填）都包含。
            组合参数：所有的必选+任意一个或多个可选参数。
            全部参数：所有的必选+所有的可选参数
         反向参数：
            多参：多出一个或多个必选参数（可以任意指定）
            少参：缺少一个或多个必选参数
            无参：没有必选参数
            错误参数；参数名输入错误
```


<br/>





* 单接口测试用例

```
接口测试用例文档10要素：
编号、用例名称（标题）、模块、优先级、预置条件、请求方法、URL、请求头、请求体（请求数据）、预期结果
```  


<br/>
<br/>	

> <h2 id='postman断言'>postman断言</h2>
   


<br/>

***
<br/>

> <h1 id='MySQL数据库'>MySQL数据库</h1>

<br/>

> <h2 id='数据库安装'>数据库安装</h2>

- 1）.安装Homebrew，详细步骤参见[Homebrew官网](https://brew.sh/);
- 2）.`brew doctor`确认brew在正常工作;
- 3）.`brew update`更新包;
- 4）.`brew install mysql` 安装mysql;

<br/>

![brew安装MySQL](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_16.png)

<br/>

> <h2 id='MySQL数据库启动'>MySQL数据库启动</h2>

- 5）.mysql_secure_installation 可以设置密码(这个是可选的，可以不设置密码，我是直接回车的);
- 6）. 启动MySQL：
	- 后台启动MySQL：`brew services start mysql` ；
	- 前台启动mysql(关闭控制台，服务停止)：`mysql.server start` ；
	- 若是出现： **Error: Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)** 则可能是MySQL没有启动造成的，启动以后就可以了；
	- 若是使用后台启动：`brew services start mysql` 和前台启动：`mysql.server start`  没有成功，则可以： **`sudo mysql.server start`**
- 连接MySQL：**mysql -u root -p**


<br/>
<br/>

> <h2 id='启动问题解决'>启动问题解决</h2>

<br/>

> **问题1:** 连接Mysql出现ERROR 2002(HY000):Can‘t connect to local MySQL server through socket ‘/tmp/mysql.sock

![问题解决](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/SoftwareTest/Pictures/st_17.png)

 
 
 
 
 <br/>
<br/>

> <h2 id='数据库操作'>数据库操作</h2>

<br/>

|  **命令**  |  **用法说明**  |
|:--|:--|
| `brew services start mysql` ； | 后台启动MySQL |
| mysql -u root -p | 连接数据库 |
| create DATABASE Database1; | 创建Database1数据库 |
| use Database1; | 使用 Database1数据库 |
| CREATE TABLE c (<br/>id int,<br/>name VARCHAR(20),<br/>age TINYINT UNSIGNED<br/>); | 创建表c |
| show tables;  | 展示有哪几张表 |
| desc Database1; |  展示Database1数据库有哪几张表 |
| exit; |  退出 |
| select * from c; | 查看c表中的所有数据 |
     
     
	
	
 
  
  

 




	
	
	
		
	
	


	
	
	






































<br/>

***
<br/>

><h1 id='面试题'>面试题</h1>

> 面试题一：ssjlslsl

答案：

>面试题二：jjgdgggg

答案：

















