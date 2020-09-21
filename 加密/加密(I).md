># 网络安全
&ensp; 为了满足安全相关的需求，Apple向开发者提供了` Security` 框架与 `CommonCrypto` 接口，可以在应用中使用。`Security` 框架是一套C API 的集合，用于`管理证书、信任策略以及对设备安全数据存储的访问`。`CommonCrypto` 是一套接口的集合，用于数据的加解密、生成常见的密码散列(比如`MD5和SHA1`),计算消息认证码，生成密码或是基于密码的秘钥等。

问题: 如何验证与正确的服务器通信成功呢？
搜索`initialization vector `，使用 `initialization vector (IV)`来对数据进行加密？

共享秘钥HMAC:
[iOS中的网络加密(HMac加密方法)](https://www.jianshu.com/p/9ce6e8f6b529)

`CommonCrypto`库的使用：[关于CommonCrypto框架](https://www.jianshu.com/p/032665b3cffd)

`NSURLProtectionSpace`：的对象表示需要身份验证的服务器或服务器的一部分。 保护空间定义了一系列匹配约束，用于确定应提供哪个凭证。
SecPKCS12Import() 函数导入身份和信任： [使用Security.framework进行RSA 加密解密签名和验证签名](https://www.cnblogs.com/cocoajin/p/6183443.html)

消息认证码(MAC)是如何实现的？

**`HMAC`**:经常被称作秘钥消息认证码，由`RFC 2104`定义的。`HMAC `可以使用任何哈希函数，通常会使用`MD5`或`SHA-1`。


 &emsp;  若以用户输入密钥为基础，可以考虑使用 `CommonCrypto/CommonKeyDerivation`库的`CCKeyDerivationPBKDF()`函数。`CCKeyDerivationPBKDF()`会返回密钥值、推导算法、推导次数以及用户指定的输出密钥长度。还可以通过`SecRandomCopyBytes()`函数生成随机的字节数组。

这个CCKeyDerivationPBKDF和SecRandomCopyBytes如何使用？


完整的iOS加密过程包括创建加密上下文、处理消息、接收剩余的数据以及释放上下文。这个过程如何操作，具体代码是什么？


使用SecRandomCopyBytes()函数来创建加密安全的随机字节数组。
[](https://www.jianshu.com/p/2e90de97bf9c)
它与其他的随机数有什么不同？为什么?


&emsp;  在iOS设备上安全存储信息，Apple 提供了Keychain Services API(作为Security Framework 的一部分)来完成这项工作。Keychain 是一种在设备上安全存储少量数据的机制，比如密码、密钥、证书和身份信息等。Keychain 并不适合于通用目的的加密和数据存储，而是用来存储需要保护的信息，比如密码与私钥就会以加密的形式存储起来。
&emsp;  Keychain 数据存储在应用沙箱之外，这样就可以通过应用委托事件来持久化数据了。

Keychain如何使用的？















<br/>
***
<br/>

#`加密库 CommonCrypto`
```
//搜索 CommonCrypto
pod search CommonCrypto
pod 'CommonCrypto', '~> 1.1'

```


```
//盐最好不要写死，否则若是泄漏，app就完了
static NSString *salt = @"(*DJSF*YUD)";
NSString *pwd = @"12345";
pwd = [pwd stringWithAppendingString:salt];
//加密
pwd = pwd.md5String;

//死盐不行，可以用随机盐
//用Hash中的HMAC(散列函数)，使用一个密钥加密数据，并且做2次散列
//在实际开发中，密钥来自服务器

```
没有绝对的安全，只有相对的安全。
破解所获得利益 !< 破解所需要成本，称为相对安全

base64编码： 私钥。

![加密过程示意图](https://upload-images.jianshu.io/upload_images/2959789-22eab7f8d8f2d845.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>

>#DES 加密




<br/>
***
<br/>

># SHAI 加密




<br/>
***
<br/>

>#  MD5 加密
&ensp; 同样的数据，MD5 值一样的(`MD5只是摘抄数据的一部分，相当于指纹代表一个人一样，不可用于加密接口`)， Hash不可逆。

&ensp; 找回密码：有密码存放在服务器的这样的软件千万不要使用。所以现在没有这个功能，只有重置密码。



<br/>
***
<br/>

># 越狱情况判断
