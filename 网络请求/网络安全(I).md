- 介绍
&emsp;  在`http`请求中某些`url`访问需要具有权限认证,否则会返回`401`错误码。这时需要你在`请求头中附带授权的用户名,密码`，或者`使用https协议第一次与服务端建立连接的时候,服务端会发送iOS客户端一个证书`，这时我们需要验证服务端证书链(certificate keychain)。
&emsp;  当我们遇到以上情况的时候NSURLSession中的一个delegate会被调用:
```
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler
```
&emsp;  上述代理就是为了处理从服务端传回的`NSURLAuthenticationChallenge(认证)`,在`NSURLAuthenticationChallenge`的`protectionSpace`中存放了服务端返回的认证信息,我们需要根据这些信息来创建对应的`NSURLCredential(证书/凭证)`,并且设置对应的`NSURLSessionAuthChallengeDisposition(处理方式)`来告诉`NSURLSession`来如何处理处服务端返回的个认证。

**`NSURLSessionAuthChallengeDisposition中`**包含三种类型:
```
NSURLSessionAuthChallengePerformDefaultHandling：默认方式处理
NSURLSessionAuthChallengeUseCredential：使用指定的证书
NSURLSessionAuthChallengeCancelAuthenticationChallenge：取消
```


<br/>
***
<br/>
># 验证服务器通信

```
- (UIButton *)loadNetBtn {
    if (!_loadNetBtn) {
        _loadNetBtn = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 200, 120)];
        _loadNetBtn.backgroundColor = UIColor.cyanColor;
        [_loadNetBtn setTitle:@"加载网路" forState:UIControlStateNormal];
        [_loadNetBtn setTitleColor:UIColor.redColor forState:UIControlStateNormal];
        
        //资料：验证服务器通信：https://www.jianshu.com/p/feffa2d14536
        [_loadNetBtn addTarget:self action:@selector(loadNetworkAction:) forControlEvents:UIControlEventTouchUpInside];
        
    }
    return  _loadNetBtn;

}

- (void)viewDidLoad {
    [super viewDidLoad];

    [self.view addSubview:self.loadNetBtn];
}

- (void) loadNetworkAction:(UIButton *)sender {
     NSURL *url = [NSURL URLWithString:@"https://api.weibo.com/2/common/get_city.json"];
       NSURLRequest *request = [NSURLRequest requestWithURL:url];
       NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
       NSURLSessionDataTask *task = [session dataTaskWithRequest:request];
       [task resume];
}

//验证服务器通信
//通过验证服务器的域名、端口号、协议
+ (void)verificationServerCommunication:(NSURLAuthenticationChallenge *)challenge {
    
    //创建保护空间
    NSURLProtectionSpace *defaultSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:NULL authenticationMethod:NSURLAuthenticationMethodDefault];
    
    NSURLProtectionSpace *certificateSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:@"iPhone" authenticationMethod:NSURLAuthenticationMethodClientCertificate];
    
    NSURLProtectionSpace *trustSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:NULL authenticationMethod:NSURLAuthenticationMethodServerTrust];
    
    NSArray *validSpaces = [NSArray arrayWithObjects:defaultSpace, certificateSpace, trustSpace, nil];

    //判断 validSpaces 数组中保护空间是不是存在，因为trustSpace和challenge.protectionSpace一样，所以存在
    if (![validSpaces containsObject:challenge.protectionSpace]) {
        NSLog(@"我们不能确认这是安全的连接，请检查网络");
        
        dispatch_async(dispatch_get_main_queue(), ^{
            //在主线程中使用 AlertView进行提示
        });
        
        [challenge.sender cancelAuthenticationChallenge:challenge];
    }
}


#pragma mark -- NSURLSessionDelegate

/// 处理从服务端传回的认证
/// @param session <#session description#>
/// @param challenge 服务端返回的权限认证类
/// @param completionHandler NSURLSessionAuthChallengeDisposition: 告诉NSURLSession来如何处理处服务端返回的个认证
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler {
    
    NSLog(@"++++++++++NSURLAuthenticationChallenge 信息++++++++++");
    //ProtectionSpace，类中描述服务器中希望的认证方式以及协议，主机端口号等信息
    NSLog(@"protectionSpace: %@", challenge.protectionSpace);
    //建议使用的证书
    NSLog(@"proposedCredential: %@", challenge.proposedCredential);
    //用户密码输入失败的次数
    NSLog(@"previousFailureCount: %ld", (long)challenge.previousFailureCount);
    //授权失败的响应头的详细信息
    NSLog(@"failureResponse: %@", challenge.failureResponse);
    //最后一次授权失败的错误信息
    NSLog(@"error: %@\n", challenge.error);
    NSLog(@"\n");
    
    
    
    
    ///验证服务器通信
    [ViewController verificationServerCommunication:challenge];
        
    // 1.从服务器返回的受保护空间中拿到证书的类型
    // 2.判断服务器返回的证书是否是服务器信任的
    //AFNetworking中的处理方式
    //默认方式处理认证
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    __block NSURLCredential *credential = nil;
    //判断服务器返回的证书是否是服务器信任的
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        NSLog(@"------->>>>>>是服务器信任的证书");
        // 3.根据服务器返回的受保护空间创建一个证书
        //void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *)
        //代理方法的completionHandler block接收两个参数:
        //第一个参数: 代表如何处理证书
        //第二个参数: 代表需要处理哪个证书
        
        //创建证书
        credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        
        /*
         disposition：如何处理证书
         NSURLSessionAuthChallengePerformDefaultHandling:默认方式处理
         NSURLSessionAuthChallengeUseCredential：使用指定的证书    NSURLSessionAuthChallengeCancelAuthenticationChallenge：取消请求
         */
        
        if (credential) {//使用指定的证书
            disposition = NSURLSessionAuthChallengeUseCredential;
        } else {//取消
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
    } else {
        disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
    }
    
    //安装证书
    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}

// 接收到服务器的响应
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    NSLog(@"+++++++++++didReceiveResponse");
    completionHandler(NSURLSessionResponseAllow);
}

// 接收到服务器返回的数据
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    
//    NSString * str =  [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
//    NSData *receiveData = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
//    NSString *str1 = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

    NSLog(@"----------->>>>>didReceiveData--%lu-", (unsigned long)data.length);
    
}

// 请求完毕
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"++++++++++++++didCompleteWithError=---%@", error);
}
```
打印：
![打印](https://upload-images.jianshu.io/upload_images/2959789-c0cd0e05ce8091d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
***
<br/>
># HTTP 认证

#`HTTP Basic、HTTP Digest与NTLM认证`

&emsp;  `NSURLCredential` 代表的是一个身份证证书，`URL Loading`系统支持3种类型的证书：`password-based user credentials`, `certificate-base`。
&emsp;  `NSURLCredential`适合大多数的认证请求，因为它可以表示由用户名/密码组合、客户端证书及服务器信任创建的认证信息。

-  认证信息有三种持久化选项：
&emsp;  a. `NSURLCredentialPersistenceNone `： 要求URL载入系统 “在用完相应的认证信息后立即丢弃”;
&emsp;  b.  `NSURLCredentialPersistenceForSession` ： 要求URL载入系统“在应用终止时，丢弃相应的crediential”;
&emsp;  c.  `NSURLCredentialPersistencePermanent`：要求URL载入系统“将相应的认证信息载入钥匙串(keychain)”,以便其他应用也能使用.

&emsp;  为了认证， 要创建一个`NSURLCedential`对象，在提供的`authentication challenge`的`protection space` 上调用`authenticationMethod`方法，可以获取服务器的认证方法

-  NSURLCredential支持的认证方法
    - `HTTP basic authentication (NSURLAuthenticationMethodHTTPBasic)` 需要一个用户和密码，使用`credentialWithUser:password:persistence:`方法创建`NSURLCredential`对象;

    -  `HTTP digest authentication (NSURLAuthenticationMethodHTTPDigest)`, 与`basic authentication`类似， 也需要一个用户名和密码，使用`credentialWithUser:password:persistence:`方法创建NSURLCredential对象;

    -   `Client certificate authentication (NSURLAuthenticationMethodClientCertificate)` 需要`system identity `和需要与server进行身份验证的所有证书，使用`credentialWithIdentity:certificates:persistence:`创建NSURLCredential对象;

    -  `Server trust authentication (NSURLAuthenticationMethodServerTrust) `需要`authentication challenge的protection space`提供一个trust。使用`credentialForTrust:`来创建NSURLCredential对象。
(Basic、Digest与NTLM认证都是基于用户名/密码的认证。他们认证的响应逻辑是相同的。)


-  身份认证原理
&emsp;  在代码需要向认证的服务器请求资源时，服务器会使用http状态码401进行响应，即访问被拒绝需要验证。NSURLConnection会接收到响应并立刻使用认证challenge的一份副本来发送一条willSendRequestForAuthenticationChallenge:委托消息。


<br/>

# `AFNetworking 中的具体使用`
```

+ (void)post:(NSString *)url params:(NSDictionary *)params success:(void (^)(id json))success failure:(void (^)(NSError *error))failure
{
    
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    mgr.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json",@"text/html",@"text/plain", nil];
    [mgr.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    
    [mgr setTaskDidReceiveAuthenticationChallengeBlock:^NSURLSessionAuthChallengeDisposition(NSURLSession *session, NSURLSessionTask *task, NSURLAuthenticationChallenge *challenge, NSURLCredential *__autoreleasing *credential) {
        if (challenge.previousFailureCount == 0) {
            NSURLCredential *cre = [NSURLCredential credentialWithUser:@"账号"
                                                              password:@"密码" persistence:NSURLCredentialPersistenceForSession];
            *credential = cre;
            return NSURLSessionAuthChallengeUseCredential;
        } else {
            return NSURLSessionAuthChallengeCancelAuthenticationChallenge;
        }
    }];
    
    [mgr POST:url parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
        if (success) {
            success(responseObject);
        }
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        if (failure) {
            failure(error);
        }
    }];
}

```

自己验证：
```
- (UIButton *)loadNetBtn {
    if (!_loadNetBtn) {
        _loadNetBtn = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 200, 120)];
        _loadNetBtn.backgroundColor = UIColor.cyanColor;
        [_loadNetBtn setTitle:@"加载网路" forState:UIControlStateNormal];
        [_loadNetBtn setTitleColor:UIColor.redColor forState:UIControlStateNormal];
        
        //资料：验证服务器通信：https://www.jianshu.com/p/feffa2d14536
        [_loadNetBtn addTarget:self action:@selector(loadNetworkAction:) forControlEvents:UIControlEventTouchUpInside];
        
    }
    return  _loadNetBtn;

}

- (void)viewDidLoad {
    [super viewDidLoad];

    [self.view addSubview:self.loadNetBtn];
}



- (void) loadNetworkAction:(UIButton *)sender {
     NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];

     NSURLSessionDataTask *task =  [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com"] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
         NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
     }];
     [task resume];
}

//验证服务器通信
//通过验证服务器的域名、端口号、协议
+ (void)verificationServerCommunication:(NSURLAuthenticationChallenge *)challenge {
    
    //创建保护空间
    NSURLProtectionSpace *defaultSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:NULL authenticationMethod:NSURLAuthenticationMethodDefault];
    
    NSURLProtectionSpace *certificateSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:@"iPhone" authenticationMethod:NSURLAuthenticationMethodClientCertificate];
    
    NSURLProtectionSpace *trustSpace = [[NSURLProtectionSpace alloc] initWithHost:challenge.protectionSpace.host port:challenge.protectionSpace.port protocol:NSURLProtectionSpaceHTTPS realm:NULL authenticationMethod:NSURLAuthenticationMethodServerTrust];
    
    NSArray *validSpaces = [NSArray arrayWithObjects:defaultSpace, certificateSpace, trustSpace, nil];

    //判断 validSpaces 数组中保护空间是不是存在，因为trustSpace和challenge.protectionSpace一样，所以存在
    if (![validSpaces containsObject:challenge.protectionSpace]) {
        NSLog(@"我们不能确认这是安全的连接，请检查网络");
        
        dispatch_async(dispatch_get_main_queue(), ^{
            //在主线程中使用 AlertView进行提示
        });
        
        [challenge.sender cancelAuthenticationChallenge:challenge];
    }
}


#pragma mark -- NSURLSessionDelegate

/// 处理从服务端传回的认证
/// @param session <#session description#>
/// @param challenge 服务端返回的权限认证类
/// @param completionHandler NSURLSessionAuthChallengeDisposition: 告诉NSURLSession来如何处理处服务端返回的个认证
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler {
    
    NSLog(@"++++++++++++++++++++start++++++++++++++++++++");
    NSLog(@"++++++++++NSURLAuthenticationChallenge 信息++++++++++");
    //ProtectionSpace，类中描述服务器中希望的认证方式以及协议，主机端口号等信息
    NSLog(@"protectionSpace: %@", challenge.protectionSpace);
    //建议使用的证书
    NSLog(@"proposedCredential: %@", challenge.proposedCredential);
    //用户密码输入失败的次数
    NSLog(@"previousFailureCount: %ld", (long)challenge.previousFailureCount);
    //授权失败的响应头的详细信息
    NSLog(@"failureResponse: %@", challenge.failureResponse);
    //最后一次授权失败的错误信息
    NSLog(@"error: %@\n", challenge.error);
    NSLog(@"\n");
    NSLog(@"++++++++++++++++++++end++++++++++++++++++++");
    
    
    
    ///验证服务器通信
    [ViewController verificationServerCommunication:challenge];
    
    if (challenge.previousFailureCount == 0) {
        NSURLCredential *creds = [[NSURLCredential alloc] initWithUser:@"userName" password:@"password" persistence:NSURLCredentialPersistenceForSession];
        
        [challenge.sender useCredential:creds forAuthenticationChallenge:challenge];
    }else {
        [challenge.sender cancelAuthenticationChallenge:challenge];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"无效的信任");
        });
    }
 }
```
![打印结果](https://upload-images.jianshu.io/upload_images/2959789-36341eee914475a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
***
<br/>
