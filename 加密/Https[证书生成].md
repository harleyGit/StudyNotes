
># Https 安全简介

-   http使用的端口是80；

-   https使用端口是443，端口不能改，否则浏览器不知道怎么访问，这是双方协议的。
443端口是用发送请求进行密钥连接的，但是拿数据和发送数据时还使用80端口的。

- https的443端口安全原因
```
1. 浏览器 向443端口请求加密算法；
    1.1 传输加密算法时-> 保证安全，不被拦截；
        解决方案：采用RSA加密算法，服务器上存放私钥，然后把公钥传给浏览器，但是要注意的是把其传给浏览器

        问题1: 可能会被第三方截取也就是招到第三方的攻击，这个时候怎么办？ 
        所以这里有用到CA机构证书(获取服务器传来的公钥，然后CA机构使用自己的私钥进行加密来自服务器传来的公钥，然后再把加密的服务器公钥密文和CA机构的公钥传给浏览器，这就可以防止篡改了。)，也就是第三方权威证书验证机构。

        问题2: 但是又有一个问题，就是这个CA有没有可能是坏蛋扮演的，怎么办？

        放心，第三方CA在我们装系统时就已经把CA的公钥内置到我们的操作系统了，自己在可以在操作系统查到。
        CA传给浏览器是证书，包含：服务器端加密的公钥+服务器域名+公钥的摘要，这样就可以防伪了。


2. 浏览器拿出从443端口拿到的加密算法，向80端口发送请求得到一段密文；
3. 浏览器从拿到的加密算法 解密decode()这段密文，然后浏览器进行展示。
```

- https 连接概要：
```
1. 浏览器提交支持的加密算法列表， hi 在吗；
2. 服务器收到请求，从中挑选一个自己有的，然后向浏览器下发一个加密算法， hello 在～；
上面是2次握手；
3. 签名就是防止篡改的，使用摘要算法比如：sha1（）、hash。对于比较重要的内容，对其使用摘要算法，这样获取到的值就是‘签名’，这要就可以防篡改。但是要注意这中间有第三方的攻击，这个‘签名‘并没有防伪功能，所以不是签名。
```



TLS:(Transport Layer Security)为安全传输层协议，所以属于传输层;
<br/>
[SSL与TLS的区别以及介绍](https://blog.csdn.net/anningzhu/article/details/77517432)

<br/>

***
<br/>


># CA 生成
#   **` 自动生成`**
-  在OpenSSL的安装目录下的misc目录下，运行脚本
`/usr/local/etc/openssl@1.1/misc/CA.pl -newca`

#   **`手动生成`**
`无法进行下去了，因为只从中间CA生成开始而根CA没生成，需要先生成根CA，没尝试过。`
<br/>
[根CA的生成](https://www.cnblogs.com/Security-Darren/p/4078867.html)
<br/>
[中间CA生成](https://www.cnblogs.com/Security-Darren/p/4079605.html)

&emsp;  创建root CA私钥和证书然后进一步创建中间CA。为了便于区分，我们将创建中间`CA（intermediate CA）`的CA称为根`CA（root CA）`。
&emsp;  中间CA是root CA的代理，其证书由root CA签发，同时中间CA能够代表根CA签发用户证书，由此建立起信任链。
&emsp;  创建中间CA的好处是即使中间CA的私钥泄露，造成的影响也是可控的，我们只需要使用root CA撤销对应中间CA的证书即可。此外root CA的私钥可以脱机妥善保存，只需要在撤销和更新中间CA证书时才会使用。


<br/>

- 生成 CA 目录
```
//mkdir [-p] dirName
//-p 确保目录名称存在，不存在的就建一个
mkdir -p /Users/harleyhuang/Documents/Gitee/SSL/CA

//生成 certs、 crl、 newcerts、 private四个文件夹
mkdir certs crl newcerts private
```
![ certs、 crl、 newcerts、 private四个文件夹](https://upload-images.jianshu.io/upload_images/2959789-4dac1629dc4aee90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
//chmod abc file: 文件使用权限
//a,b,c各为一个数字，分别表示User、Group、及Other的权限。
//若r=4，w=2，x=1，要rwx属性则4+2+1=7
chmod 777 private


// touch命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件
touch index.txt

//echo： 指令与 PHP 的 echo 指令类似，都是用于字符串的输出
//echo "It is a test" > myfile：显示结果定向至文件
echo 1000 > serial

```

![文件夹生成](https://upload-images.jianshu.io/upload_images/2959789-51b6cc921b7cb428.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




-  创建中间CA的私钥，采用AES-256算法加密中间CA的私钥，中途会让我们输入加密密钥，最后修改中间CA的私钥访问权限
```
openssl genrsa -aes256 -out private/cakey.pem 4096
chmod 400 private/cakey.pem
```
![cakey.pem 生成](https://upload-images.jianshu.io/upload_images/2959789-4b8e690fadc799bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


-  中间CA要向root CA申请公钥证书，就要首先产生一个CSR（证书签名请求，Certificate Signing Request都有作用）格式的请求文件，将其发送给root CA后等待其对中间CA的审查。

&emsp;  将创建root CA时使用的配置文件拷贝到中间CA证书目录下，该配置文件在生成CSR文件和后续签发用户证书时都有用。
&emsp;  创建并编辑intermediate_CA.cnf,若是demoCA中没有rootCA.cnf文件夹可以去`/System/Library/OpenSSL/openssl.cnf `进行拷贝一份：
```
cp /Users/harleyhuang/Documents/Gitee/SSL/demoCA/rootCA.cnf /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA/intermediateCA.cnf


cd cd /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA

vim intermediateCA.cnf
//编辑
...
[ CA_default ]
dir                = /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA
```
今后我们每次使用中间CA创建新的证书时，以`”-config /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA/intermediateCA.cnf“` 的形式告诉OpenSSL中间CA的信息。

`intermediateCA.cnf.cnf`默认申请的有效期是365天，如果想要修改这个时长，可以在`[ CA_default ]的"default_days"`字段进行修改。

- 生成CSR文件
```
cd /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA

openssl req -config intermediateCA.cnf -sha256 -new -key private/cakey.pem -out cacsr.pem
```
![cacsr.pem 生成](https://upload-images.jianshu.io/upload_images/2959789-2c47280475e446ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;  随后系统会要求我们输入中间CA的私钥密码，设置中间CA的一些身份信息等等，注意`”Organization Name“`一项一定要与root CA时设置的相同。
&emsp;  正确输入中间CA的身份信息后我们就得到了中间CA的CSR。
&emsp;  接下来我们用root CA同意中间CA的请求，因为我们将使用root CA的私钥签名中间CA的证书，这时系统会要求我们输入root CA的私钥密码，选择签名证书如下：
```
cd /Users/harleyhuang/Documents/Gitee/SSL/demoCA 

openssl ca -config rootCA.cnf -extensions v3_ca -notext -md sha256 -in /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA/cacsr.pem -out /Users/harleyhuang/Documents/Gitee/SSL/intermediateCA/cacert.pem
```




<br/>

***
<br/>


># 根证书生成

-  新建一个SSL的文件夹
-  终端定位到这个文件夹

` cd /Users/harleyhuang/Documents/Gitee/SSL`

-  创建根证书密钥文件(自己做CA)root.key
`openssl genrsa -des3 -out root.key`

![根证书秘钥创建](https://upload-images.jianshu.io/upload_images/2959789-a3b55f6ff046d6f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![root 密钥key完成创建](https://upload-images.jianshu.io/upload_images/2959789-ca8240f9709cb080.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建根证书的申请文件root.csr
`openssl req -new -key root.key -out root.csr`
![截屏2020-03-15上午9.53.04.png](https://upload-images.jianshu.io/upload_images/2959789-be699386900ebd72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


-  创建一个自当前日期起为期十年的根证书root.crt
`openssl x509 -req -days 3650 -sha1 -extensions v3_ca -signkey root.key -in root.csr -out root.crt`

![root.crt 生成](https://upload-images.jianshu.io/upload_images/2959789-8cc94ff6157d41f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3 个证书生成](https://upload-images.jianshu.io/upload_images/2959789-6078190bfca61c88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/>



># 服务器SSL证书生成

-  创建服务器证书密钥文件(在SSL文件夹中生成私钥)
  使用openssl工具生成一个RSA私钥，
`openssl genrsa -des3 -out server.key 2048`

![生成私钥](https://upload-images.jianshu.io/upload_images/2959789-b1b3b597022dfd99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![私钥文件](https://upload-images.jianshu.io/upload_images/2959789-d79a3576e5ad5585.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; 生成rsa私钥，des3算法，2048位强度，server.key是秘钥文件名。
&emsp; `注意`：生成私钥，需要提供一个至少4位的密码。

-  创建服务器证书的申请文件root.csr(生成CSR[证书签名请求])
&emsp; 生成私钥之后，便可以创建csr文件了。
&emsp; 此时可以有两种选择。理想情况下，可以将证书发送给证书颁发机构（CA），CA验证过请求者的身份之后，会出具签名证书（很贵）。另外，如果只是内部或者测试需求，也可以使用OpenSSL实现自签名，具体操作如下：
```
openssl req -new -key server.key -out server.csr

//或者生成如下证书，注意2者不同，这里使用上面的
//openssl req -new -sha256 -x509 -days 365 -key server.key -out server.crt
```
![输入信息](https://upload-images.jianshu.io/upload_images/2959789-677efc0acc44e92a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
说明：需要依次输入国家，地区，城市，组织，组织单位，Common Name和Email。其中Common Name，可以写自己的名字或者域名，如果要支持https，Common Name应该与域名保持一致，否则会引起浏览器警告。


- 删除私钥中的密码
在【创建根证书密钥文件】过程中，由于必须要指定一个密码。而这个密码会带来一个副作用，那就是在每次Apache启动Web服务器时，都会要求输入密码，这显然非常不方便。要删除私钥中的密码，操作如下：
```
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
```

-  生成自签名证书(创建一个自当前日期起为期十年的根证书)
&emsp;  如果你不想花钱让CA签名，或者只是测试SSL的具体实现。那么，现在便可以着手生成一个自签名的证书了。
&emsp;  需要注意的是，在使用自签名的临时证书时，浏览器会提示证书的颁发机构是未知的。
`openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt`
![自签名证书生成](https://upload-images.jianshu.io/upload_images/2959789-e037bb51e07e8d87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

证书查看
![证书查看](https://upload-images.jianshu.io/upload_images/2959789-9eae159aca20007f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

说明：crt上有证书持有人的信息，持有人的公钥，以及签署者的签名等信息。当用户安装了证书之后，便意味着信任了这份证书，同时拥有了其中的公钥。证书上会说明用途，例如服务器认证，客户端认证，或者签署其他证书。当系统收到一份新的证书的时候，证书会说明，是由谁签署的。如果这个签署者确实可以签署其他证书，并且收到证书上的签名和签署者的公钥可以对上的时候，系统就自动信任新的证书。

- 或者使用根证书生成crt
`openssl x509 -req -days 730 -sha1 -extensions v3_req -CA root.crt -CAkey root.key -CAserial root.csr -CAcreateserial -in server.csr -out server2.crt `

![根证书生成 server2.crt](https://upload-images.jianshu.io/upload_images/2959789-9c1bebcc447b1861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![server2.crt 文件](https://upload-images.jianshu.io/upload_images/2959789-3bef6b91c40a1ea3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



-  安装私钥和证书
&emsp;  将私钥(server.key)和证书文件(server.crt)复制到Apache的配置目录下即可，在Mac 10.10系统中，复制到/etc/apache2/目录中即可。


- 客户端利用AF3.0使用自定义证书
第一步：
```
// 1.初始化单例类
     AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.securityPolicy.SSLPinningMode = AFSSLPinningModeCertificate;
    // 2.设置证书模式
    NSString * cerPath = [[NSBundle mainBundle] pathForResource:@"xxx" ofType:@"cer"];
    NSData * cerData = [NSData dataWithContentsOfFile:cerPath];
    manager.securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate withPinnedCertificates:[[NSSet alloc] initWithObjects:cerData, nil]];
    // 客户端是否信任非法证书
    mgr.securityPolicy.allowInvalidCertificates = YES;
    // 是否在证书域字段中验证域名
    [mgr.securityPolicy setValidatesDomainName:NO];
```

第二步：使用AFNetworking进行请求

AFNetworking首先需要配置AFSecurityPolicy类，AFSecurityPolicy类封装了证书校验的过程
```
/**
 AFSecurityPolicy分三种验证模式：
 AFSSLPinningModeNone:只是验证证书是否在信任列表中
 AFSSLPinningModeCertificate：该模式会验证证书是否在信任列表中，然后再对比服务端证书和客户端证书是否一致
 AFSSLPinningModePublicKey：只验证服务端证书与客户端证书的公钥是否一致
*/
 
AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];
    securityPolicy.allowInvalidCertificates = YES;//是否允许使用自签名证书
    securityPolicy.validatesDomainName = NO;//是否需要验证域名，默认YES
 
    AFHTTPSessionManager *_manager = [AFHTTPSessionManager manager];
    _manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    _manager.securityPolicy = securityPolicy;
    //设置超时
    [_manager.requestSerializer willChangeValueForKey:@"timeoutinterval"];
    _manager.requestSerializer.timeoutInterval = 20.f;
    [_manager.requestSerializer didChangeValueForKey:@"timeoutinterval"];
    _manager.requestSerializer.cachePolicy = NSURLRequestReloadIgnoringCacheData;
    _manager.responseSerializer.acceptableContentTypes  = [NSSet setWithObjects:@"application/xml",@"text/xml",@"text/plain",@"application/json",nil];
  
    __weak typeof(self) weakSelf = self;
    [_manager setSessionDidReceiveAuthenticationChallengeBlock:^NSURLSessionAuthChallengeDisposition(NSURLSession *session, NSURLAuthenticationChallenge *challenge, NSURLCredential *__autoreleasing *_credential) {
         
        SecTrustRef serverTrust = [[challenge protectionSpace] serverTrust];
        /**
         *  导入多张CA证书
         */
        NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"ca" ofType:@"cer"];//自签名证书
        NSData* caCert = [NSData dataWithContentsOfFile:cerPath];
        NSArray *cerArray = @[caCert];
        weakSelf.manager.securityPolicy.pinnedCertificates = cerArray;
         
        SecCertificateRef caRef = SecCertificateCreateWithData(NULL, (__bridge CFDataRef)caCert);
        NSCAssert(caRef != nil, @"caRef is nil");
         
        NSArray *caArray = @[(__bridge id)(caRef)];
        NSCAssert(caArray != nil, @"caArray is nil");
         
        OSStatus status = SecTrustSetAnchorCertificates(serverTrust, (__bridge CFArrayRef)caArray);
        SecTrustSetAnchorCertificatesOnly(serverTrust,NO);
        NSCAssert(errSecSuccess == status, @"SecTrustSetAnchorCertificates failed");
         
        NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        __autoreleasing NSURLCredential *credential = nil;
        if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
            if ([weakSelf.manager.securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
                credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
                if (credential) {
                    disposition = NSURLSessionAuthChallengeUseCredential;
                } else {
                    disposition = NSURLSessionAuthChallengePerformDefaultHandling;
                }
            } else {
                disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
         
        return disposition;
    }];

```
上述代码通过给AFHTTPSessionManager重新设置证书验证回调来自己验证证书，然后将自己的证书加入到可信任的证书列表中，即可通过证书的校验。


<br/>

***
<br/>

- 创建客户端证书密钥文件client.key
`openssl genrsa -des3 -out client.key 2048`

-  创建客户端证书的申请文件client.csr
`openssl req -new -key client.key -out client.csr`

-  创建一个自当前日期起有效期为两年的客户端证书client2.crt
`openssl x509 -req -days 730 -sha1 -extensions v3_req -CA server.crt -CAkey server.key -CAserial server.csr -CAcreateserial -in client.csr -out client2.crt`
![客户端证书client2.crt 生成](https://upload-images.jianshu.io/upload_images/2959789-7a9a52c201da76a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
发现它`unable to load CA Private Key`

-  将客户端证书文件client.crt和客户端证书密钥文件client.key合并成客户端证书安装包client.pfx
`openssl pkcs12 -export -in client2.crt -inkey client.key -out client.pfx`

-  保存生成的文件备用，其中server.crt和server.key是配置单向SSL时需要使用的证书文件，client.crt是配置双向SSL时需要使用的证书文件，client.pfx是配置双向SSL时需要客户端安装的证书文件

     .crt文件和.key可以合到一个文件里面，把2个文件合成了一个.pem文件（直接拷贝过去就行了）




<br/>

***
<br/>

**`参考资料:`**
<br/>
[HTTPS 原理解析](https://juejin.im/entry/6844903506537611271)
<br/>
[OpenSSL证书生成及Mac上Apache服务器配置HTTPS](https://www.jianshu.com/p/b2a9655fe687)
<br/>
[搭建CA服务器 US](https://www.cnblogs.com/zhaojiedi1992/p/zhaojiedi_linux_011_ca.html)
<br/>
[https 原理解析](https://juejin.im/entry/6844903506537611271)









