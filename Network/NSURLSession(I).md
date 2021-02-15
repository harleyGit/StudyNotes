
- **NSURLRequest**
- **NSURLRequest 的分类NSHTTPURLRequest**
- **NSURLSession API**
- [**NSURLSession 篇一**](https://www.cnblogs.com/taoxu/p/8962778.html)
- [**NSURLSession 篇二**](https://www.cnblogs.com/taoxu/p/9003457.html)



<br/>

***
<br/>


># **NSURLRequest** 

- **`策略类型`**

```
typedef NS_ENUM(NSUInteger, NSURLRequestCachePolicy)
{
    //默认缓存策略
    NSURLRequestUseProtocolCachePolicy = 0,
    
    //URL应该加载源端数据，不使用本地缓存数据
    NSURLRequestReloadIgnoringLocalCacheData = 1,

    //本地缓存数据、代理和其他中介都要忽视他们的缓存，直接加载源数据
    NSURLRequestReloadIgnoringLocalAndRemoteCacheData = 4, // Unimplemented

    //从服务端加载数据，完全忽略缓存。和NSURLRequestReloadIgnoringLocalCacheData一样
    NSURLRequestReloadIgnoringCacheData = NSURLRequestReloadIgnoringLocalCacheData,


    //使用缓存数据，忽略其过期时间；只有在没有缓存版本的时候才从源端加载数据
    NSURLRequestReturnCacheDataElseLoad = 2,
    //只使用cache数据，如果不存在cache，就请求失败，不再去请求数据  用于没有建立网络连接离线模式
    NSURLRequestReturnCacheDataDontLoad = 3,

    //指定如果已存的缓存数据被提供它的源段确认为有效则允许使用缓存数据响应请求，否则从源段加载数据。
    NSURLRequestReloadRevalidatingCacheData = 5, // Unimplemented
};
```

<br/>

- ***`属性`***

```
//缓存策略
@property (readonly) NSURLRequestCachePolicy cachePolicy;

//请求超时时间
 @property (readonly) NSTimeInterval timeoutInterval;

//主文档地址 这个地址用来存放缓存
@property (nullable, readonly, copy) NSURL *mainDocumentURL;

//获取网络请求的服务类型 枚举如下
@property (readonly) NSURLRequestNetworkServiceType networkServiceType API_AVAILABLE(macos(10.7), ios(4.0), watchos(2.0), tvos(9.0));

//获取是否允许使用服务商蜂窝网络
@property (readonly) BOOL allowsCellularAccess  API_AVAILABLE(macos(10.8), ios(6.0), watchos(2.0), tvos(9.0));

```

<br/>

***
<br/>

># NSURLRequest的分类NSHTTPURLRequest

`@interface NSURLRequest (NSHTTPURLRequest) `

**`属性和方法`**

```
//HTTP请求方式  POST GET 等
@property (nullable, readonly, copy) NSString *HTTPMethod;


//得到一个字典数据，设置的 HTTP请求头的键值数据
@property (nullable, readonly, copy) NSDictionary<NSString *, NSString *> *allHTTPHeaderFields;


//设置http请求头中的字段值，这个我们待会在看看一些Demo代码和AF的源码比较一下
- (nullable NSString *)valueForHTTPHeaderField:(NSString *)field;


//请求体
@property (nullable, readonly, copy) NSData *HTTPBody;


//http请求体的输入流
@property (nullable, readonly, retain) NSInputStream *HTTPBodyStream;


//设置发送请求时是否发送cookie数据
@property (readonly) BOOL HTTPShouldHandleCookies;


//设置的请求时是否按顺序收发 默认禁用 在某些服务器中设为YES可以提高网络性能
@property (readonly) BOOL HTTPShouldUsePipelining API_AVAILABLE(macos(10.7), ios(4.0), watchos(2.0), tvos(9.0));

 //设置http请求头中的字段值
- (void)setValue:(nullable NSString *)value forHTTPHeaderField:(NSString *)field;

//向http请求头中添加一个字段
- (void)addValue:(NSString *)value forHTTPHeaderField:(NSString *)field;
```

从上述的方法和属性可以看到，这个类别是关于请求体和请求头的一些设置。

<br/>

下面来说一说一些请求头的含义：
> Host: 目标服务器的网络地址；<br/>
Accept: 让服务端知道客户端所能接收的数据类型，如text/html；<br/>
Content-Type: body中的数据类型，如application/json; charset=UTF-8<br/>
Accept-Language: 客户端的语言环境，如zh-cn<br/>
Accept-Encoding: 客户端支持的数据压缩格式，如gzip<br/>
 User-Agent: 客户端的软件环境，我们可以更改该字段为自己客户端的名字，比如QQ music v1.11，比如浏览器Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/600.8.9 (KHTML, like Gecko) Maxthon/4.5.2<br/>
Connection: keep-alive，该字段是从HTTP 1.1才开始有的，用来告诉服务端这是一个持久连接，“请服务端不要在发出响应后立即断开TCP连接”。关于该字段的更多解释将在后面的HTTP版本简介中展开。<br/>
 Content-Length: body的长度，如果body为空则该字段值为0。该字段一般在POST请求中才会有。<br/>


<br/>

- ***`请求体的封装图`***
![上传文件时可以用到](https://upload-images.jianshu.io/upload_images/2959789-3fb21eede3fae86e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

[**iOS上传文件**](https://www.jianshu.com/p/db0e1843c425)

[**文件上传**](https://www.cnblogs.com/wendingding/p/3949966.html)

<br/>

***
<br/>

># NSURLSession 关系图

![Session关系图](https://upload-images.jianshu.io/upload_images/2959789-2cb824aaa2e80977.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

***
<br/>



># **NSURLSession API**

**`NSURLSession 默认是挂起的状态，若需要网络请求则去开启`**

```
//获取全局的NSURLSession对象。在iPhone的所有app共用一个全局session。
@property (class, readonly, strong) NSURLSession *sharedSession;


//利用NSURLSessionConfiguration对象得到NSURLSession的方法，区别在于代理的设置
+ (NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration;
+ (NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration delegate:(nullable id <NSURLSessionDelegate>)delegate delegateQueue:(nullable NSOperationQueue *)queue;


//不明白可以到网易查含义
@property (readonly, retain) NSOperationQueue *delegateQueue;
@property (nullable, readonly, retain) id <NSURLSessionDelegate> delegate;
@property (readonly, copy) NSURLSessionConfiguration *configuration;


//给session添加一个描述信息
@property (nullable, copy) NSString *sessionDescription;


//该方法是任务完成之后调用会释放session
//涉及到的是session和代理之间相互的强引用可能会造成内存泄漏的问题，了解一下！
- (void)finishTasksAndInvalidate;


//这个是取消任务释放session 和前面的任务完成之后是有区别的，上面的注释又给我们解释说让我们注意任务的状态
//可能会给一些结束的任务发送cancel消息

//NOTE:当对 session invalidate 后，就不能再创建新的 task 了，两个方法的不同之处是，- finishTasksAndInvalidate会等到正在执行的 task 执行完成，调用完所有回调或 delegate 后，释放对 delegate 的强引用，而- invalidateAndCancel方法则是直接取消所有正在执行的 task。
- (void)invalidateAndCancel;


//清空所有的 cookie、缓存、证书等，传入的 completionHandler 在上述操作完成后执行。
- (void)resetWithCompletionHandler:(void (^)(void))completionHandler; 


//将内存中的 cookie、证书等写到硬盘，之后的请求会使用新的 TCP 连接，传入的 completionHandler 在上述操作完成后执行。
- (void)flushWithCompletionHandler:(void (^)(void))completionHandler;


//获取 session 中的 task，在获取完 task 列表后会执行传入的 completionHandler 参数，而 task 列表则作为 block 的参数传入。
- (void)getTasksWithCompletionHandler:(void (^)(NSArray<NSURLSessionDataTask *> *dataTasks, NSArray<NSURLSessionUploadTask *> *uploadTasks, NSArray<NSURLSessionDownloadTask *> *downloadTasks))completionHandler;


//
- (void)getAllTasksWithCompletionHandler:(void (^)(NSArray<__kindof NSURLSessionTask *> *tasks))completionHandler

```

<br/>

**`下面这些方法是关于Task初始化的`**


```
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request;


- (NSURLSessionDataTask *)dataTaskWithURL:(NSURL *)url;


- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request fromFile:(NSURL *)fileURL;


- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request fromData:(NSData *)bodyData;


- (NSURLSessionUploadTask *)uploadTaskWithStreamedRequest:(NSURLRequest *)request;


- (NSURLSessionDownloadTask *)downloadTaskWithRequest:(NSURLRequest *)request;


- (NSURLSessionDownloadTask *)downloadTaskWithURL:(NSURL *)url;


- (NSURLSessionDownloadTask *)downloadTaskWithResumeData:(NSData *)resumeData;


- (NSURLSessionStreamTask *)streamTaskWithHostName:(NSString *)hostname port:(NSInteger)port API_AVAILABLE(macos(10.11), ios(9.0), tvos(9.0)) __WATCHOS_PROHIBITED;


- (NSURLSessionStreamTask *)streamTaskWithNetService:(NSNetService *)service API_AVAILABLE(macos(10.11), ios(9.0), tvos(9.0)) __WATCHOS_PROHIBITED;

```

<br/>

**图片下载Demo**

```
- (void) seessionTasdkAciont {
    
    //创建一个URL请求
    NSURLRequest *request = [NSURLRequest requestWithURL:@"self.imageURL"];
    
    //创建一个NSURLSessionConfiguration(会话配置器)，用于对NSSession(会话)设置工作模式
    /**
     *defaultSessionConfiguration: 一般模式（default）,工作模式类似于原来的NSURLConnection，可以使用缓存的Cache，Cookie，鉴权;
     *ephemeralSessionConfiguration: 临时模式（ephemeral）,不使用缓存的Cache、Cookie、鉴权；
     *backgroundSessionConfigurationWithIdentifier: 后台模式（background）：在后台完成上传下载，创建Configuration对象的时候需要给一个NSString的ID用于追踪完成工作的Session是哪一个（后面会讲到）
     */
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
    
    //用指定的 "会话配置器" 创建一个会话, 并且选择运行的网络线程, 系统提供了两个创建方法：
    //1.sessionWithConfiguration:
    //2.sessionWithConfiguration:delegate:delegateQueue:
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration ];
    
    //4.调用 session 的会话方法执行 request 请求(在会话的线程上执行)
    NSURLSessionDownloadTask *task = [session downloadTaskWithRequest:request
                                                    completionHandler:^(NSURL *localfile, NSURLResponse *response, NSError *error) {
        
        if (!error) {
            if ([request.URL isEqual:@"self.imageURL"]) {
                
                UIImage *image = [UIImage imageWithData:[NSData dataWithContentsOfURL:localfile]];
                
                dispatch_async(dispatch_get_main_queue(), ^{
                    //self.image = image;
                    
                });
            }
        }
    }];
    [task resume];
    
}
```

