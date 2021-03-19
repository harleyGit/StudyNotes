
- **NSURLSession**
	- Block接受数据 
	- Delegate接受数据
- **AF类结构**
	- 目录结构
	- AFURLSessionManager
- **类结构图**
	- 网络请求流程图 
- **属性**
- **多路复用**






<br/>

***
<br/>

># **NSURLSession**

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet7.jpeg)



NSURLSession中比较重要的几个对象: NSURLSessionTask、NSURLSessionConfiguration、为它请求时执行的代理方法。


![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet8.png)


NSURLSession的使用共分两步:

- 通过NSURLSession的实例创建task
- 通过task选择自己所需要的方法，有Delegate方法和Block方法执行task
> task一共有4个delegate，只要设置了一个，就代表四个全部设置，有时候一些delegate不会被触发的原因在于这四种delegate是针对不同的URLSession类型和URLSessionTask类型来进行响应的，也就是说不同的类型只会触发这些delegate中的一部分，而不是触发所有的delegate。

<br/>

**Block接受数据**

使用Block很简单，只需要传入请求的Request就可以直接获取到数据。

```
- (IBAction)orginalDownLoadAction:(UIButton *)sender {
    __weak typeof(self) weakself = self;
    NSURLSession *session = [self createASession];
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:[self creatRequest:downURL] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        dispatch_async(dispatch_get_main_queue(), ^{
            weakself.showImageView.image = [UIImage imageWithData:data];
        });
    }];
    [dataTask resume];
}

//  创建Session对象
- (NSURLSession *)createASession {
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSOperationQueue *operationQueue = [[NSOperationQueue alloc] init];
    operationQueue.maxConcurrentOperationCount = 1;
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:operationQueue];
    return session;
}

- (NSURLRequest *)creatRequest:(NSString *)url {
    NSURLRequest *requset = [NSURLRequest requestWithURL:[NSURL URLWithString:url]];
    return requset;
}
```



<br/>

**‌Delegate接受数据**

可以真正的观测到数据的获取和请求的发送.

```
- (IBAction)orginalDownLoadAction:(UIButton *)sender {
    NSURLSession *session = [self createASession];
    
    NSURLSessionDataTask *dataTask = [session dataTaskWithURL:[NSURL URLWithString:downURL]];
    
    [dataTask resume];
}
/**
 接收到服务器的响应
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler{ 
 
    _mutableData = [[NSMutableData alloc] init];
     // 允许处理服务器的响应，才会继续接收服务器返回的数据
    if (completionHandler) {
        completionHandler(NSURLSessionResponseAllow);
    }
}

/**
 接收到服务器的数据（可能调用多次）
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data {
    NSLog(@"接收数据返回");
    [_mutableData appendData:data];
}

/**
 请求成功或者失败（如果失败，error有值）
 所有的代理都会执行此代理方法
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    if (error) {
        NSLog(@"下载有误 %@",[error localizedDescription]);
    }
    else {
        NSLog(@"完成下载");
        __weak typeof(self) weakself = self;
        dispatch_async(dispatch_get_main_queue(), ^{
            weakself.showImageView.image = [UIImage imageWithData:_mutableData];
        });
    }
}
```




<br/>

***
<br/>

># AF类结构

<br/>

> **[目录结构](https://www.jianshu.com/p/e4ff363da7f7)**
- **AFNetWorking** 这个文件是一个头文件。啥也没做，就是引入了其他文件方便使用。
- **AFURLSessionManager** 是核心类，基本上通过它来实现了大部分核心功能。负责请求的建立、管理、销毁、安全、请求重定向、请求重启等各种功能。他主要实现了NSURLSession和NSRULSessionTask的封装。
- **AFHTTPSessionManager** 是AFURLSessionManager的子类对外面做一个接口的显示, 主要实现了对HTTP请求的优化。
- **AFURLRequestSerialization** 这个主要用于请求头的编码解码、序列化、优化处理、简化请求拼接过程等。
- **AFURLResponseSerialization** 这个主要用于网络返回数据的序列化、编码解码、序列化、数据处理等。
- **AFSecurityPolicy** 这个主要用于请求的认证功能。比如https的认证模式等。
- **AFNetworkReachabilityManager** 这个主要用于监听网络请求状态变化功能。

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet10.png)


<br/>
<br/>

> **AFURLSessionManager**

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet9.jpeg)


**AFURLSessionManager.h头文件属性**:

```

interface AFURLSessionManager : NSObject <NSURLSessionDelegate, NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate, NSSecureCoding, NSCopying>

//指定的初始化方法、通过他来初始化一个Manager对象。
- (instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)configuration 

//AFURLSessionManager通过session来管理和创建网络请求。一个manager就实现了对这个session的管理，他们是一一对应的关系。
@property (readonly, nonatomic, strong) NSURLSession *session;

//处理网络请求回调的操作队列,就是我们初始化session的时候传入的那个OperationQueue参数。如果不传入，默认是MainOperationQueue。
@property (readonly, nonatomic, strong) NSOperationQueue *operationQueue;

//对返回数据的处理都通过这个属性来处理，比如数据的提取、转换等。默认是一个`AFJSONResponseSerializer`对象用JSON的方式解析。
@property (nonatomic, strong) id <AFURLResponseSerialization> responseSerializer;

//用于指定session的安全策略。用于处理信任主机和证书认证等。默认是`defaultPolicy`。
@property (nonatomic, strong) AFSecurityPolicy *securityPolicy;

//观测网络状态的变化，具体可以看我的Demo用法。
@property (readwrite, nonatomic, strong) AFNetworkReachabilityManager *reachabilityManager;

@end

```

<br/>

类中使用的通知:

在外面通过监听这些通知，可以获取到这些通知的信息

> - AFNetworkingTaskDidResumeNotification
- AFNetworkingTaskDidCompleteNotification
- AFNetworkingTaskDidSuspendNotification
- AFURLSessionDidInvalidateNotification
- AFURLSessionDownloadTaskDidFailToMoveFileNotification
- AFNetworkingTaskDidCompleteResponseDataKey
- AFNetworkingTaskDidCompleteSerializedResponseKey
- AFNetworkingTaskDidCompleteResponseSerializerKey
- AFNetworkingTaskDidCompleteAssetPathKey
- AFNetworkingTaskDidCompleteErrorKey






























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



<br/>

***
<br/>

># 属性

**`@property (nonatomic, assign) BOOL removesKeysWithNullValues;`**

&emsp;  在`AFNetWorking`只要把这个`removesKeysWithNullValues=YES.`后台返回的JSON数据中存在空的键值对,将会被自动删除,可以避免空值做操作,造成崩溃问题。





># 多路复用

&emsp; AFURLSessionManager和NSURLSession是一对一的关系，AFURLSessionManager会在初始化的时候创建对应的NSURLSession。同样AFURLSessionManager在注释中写明了可以提供一个配置好的manager单例来全局复用。

&emsp; 在iOS9.0之后，session支持http2.0。http2.0的一个特点就是多路复用，可以减少访问同一个服务器时，重新建立tcp连接的耗时和资源。

&emsp; 在不同的场景使用不同的session，比如：一个session处理普通的请求，一个session处理background请求；一个sesison处理浏览器公开的请求，一个session专门处理隐私请求等等场景。












