






- **AFNetworking 架构图**
- **AFURLSessionManager**
- **AFHTTPSessionManager**
- **NSURLSession**
- **AFURLRequestSerialization**
- [ **AFNetworking到底做了什么？(二)**](https://www.jianshu.com/p/f32bd79233da)
- [**AFNetWorking 3.0之前设置请求头**](https://www.jianshu.com/p/45c722f726fd)
- [**请求头的配置用来完成HTTP Basic Auth的鉴权**](https://blog.csdn.net/deft_mkjing/article/details/51900737)
- [**AFNetworking使用技巧与问题**](https://www.jianshu.com/p/37018da11815)
- [**HTTPS认证**](https://www.jianshu.com/p/a84237b07611)






<br/>

***
<br/>



># AFNetworking 架构图

<br/>

- AFNetworking 3.0 的 请求流程线程的关系

&emsp;   我们一开始初始化`sessionManager`的时候，一般都是在主线程，（当然不排除有些人喜欢在分线程初始化...）
&emsp;  然后我们调用`get`或者`post`等去请求数据，接着会进行`request`拼接，AF代理的字典映射，`progress`的`KVO`添加等等，到`NSUrlSession`的`resume`之前这些准备工作，仍旧是在主线程中的。
 然后我们调用`NSUrlSession`的`resume`，接着就跑到`NSUrlSession`内部去对网络进行数据请求了,在它内部是多线程并发的去请求数据的。
&emsp;  紧接着数据请求完成后，回调回来在我们一开始生成的并发数为1的`NSOperationQueue`中，这个时候会是多线程串行的回调回来的。
&emsp;  然后我们到返回数据解析那一块，我们自己又创建了并发的多线程，去对这些数据进行了各种类型的解析。
&emsp;  最后我们如果有自定义的`completionQueue`，则在自定义的`queue`中回调回来，也就是分线程回调回来，否则就是主队列，主线程中回调结束。


<br/>

- HTTPS 认证流程

![<br/> https 认证流程 <br/> ](https://upload-images.jianshu.io/upload_images/2959789-6e492103d356ac3c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

**1\. 客户端发起HTTPS请求**

　　&emsp; 这个没什么好说的，就是用户在浏览器里输入一个https网址，然后连接到server的443端口。

　　
**2\. 服务端的配置**

　　&emsp; 采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面。这套证书其实就是一对公钥和私钥。如果对公钥和私钥不太理解，可以想象成一把钥匙和一个锁头，只是全世界只有你一个人有这把钥匙，你可以把锁头给别人，别人可以用这个锁把重要的东西锁起来，然后发给你，因为只有你一个人有这把钥匙，所以只有你才能看到被这把锁锁起来的东西。
　
　　
**3\. 传送证书**

　　&emsp; 这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等等。
　　
　　
**4\. 客户端解析证书**

　　&emsp; 这部分工作是有客户端的TLS/SSL来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随机值。然后用证书对该随机值进行加密。就好像上面说的，把随机值用锁头锁起来，这样除非有钥匙，不然看不到被锁住的内容。
　　
　　
**5\. 传送加密信息**

　　&emsp; 这部分传送的是用证书加密后的随机值，目的就是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。
　　
　　
**6\. 服务段解密信息**

　　&emsp; 服务端用私钥解密后，得到了客户端传过来的随机值(私钥)，然后把内容通过该值进行对称加密。所谓对称加密就是，将信息和私钥通过某种算法混合在一起，这样除非知道私钥，不然无法获取内容，而正好客户端和服务端都知道这个私钥，所以只要加密算法够彪悍，私钥够复杂，数据就够安全。
　　
　　
**7\. 传输加密后的信息**

　　&emsp; 这部分信息是服务段用私钥加密后的信息，可以在客户端被还原。
　　
　　
**8\. 客户端解密信息**

　　&emsp; 客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容。整个过程第三方即使监听到了数据，也束手无策。

<br/>

这就是整个https验证的流程了。简单总结一下：

*   就是用户发起请求，服务器响应后返回一个证书，证书中包含一些基本信息和公钥。
*   用户拿到证书后，去验证这个证书是否合法，不合法，则请求终止。
*   合法则生成一个随机数，作为对称加密的密钥，用服务器返回的公钥对这个随机数加密。然后返回给服务器。
*   服务器拿到加密后的随机数，利用私钥解密，然后再用解密后的随机数（对称密钥），把需要返回的数据加密，加密完成后数据传输给用户。
*   最后用户拿到加密的数据，用一开始的那个随机数（对称密钥），进行数据解密。整个过程完成。

当然这仅仅是一个单向认证，https还会有双向认证，相对于单向认证也很简单。仅仅多了服务端验证客户端这一步。感兴趣的可以看看这篇：[Https单向认证和双向认证。](https://link.jianshu.com?t=http://blog.csdn.net/duanbokan/article/details/50847612)

###### 了解了https认证流程后，接下来我们来讲讲AFSecurityPolicy这个类，AF就是用这个类来满足我们各种https认证需求。




<br/>
![类结构图](https://upload-images.jianshu.io/upload_images/2959789-f764394a6ddbbe9e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
![封装架构图](https://upload-images.jianshu.io/upload_images/2959789-f40c67aa136faa86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
&emsp;   原生NSURLSession做网络请求Demo:

```
    //创建Configuration对象，并设置各种属性
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    configuration.timeoutIntervalForRequest = 10.0;
    configuration.allowsCellularAccess = YES;
    
    //通过Configuration创建session，一个session可以管理多个task
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186.32812/login"];
    
    //通过URL创建request
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //设置request的请求方法和请求体
    request.HTTPMethod = @"POST";
    request.HTTPBody = [@"username=520it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
    
    //通过session和request来创建task
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        
    }];
    
    [dataTask resume];
```
&emsp;   可以用原生与AF的做对比，其实大致差不多，只不过是AF做了很多细致的优化。

<br/>

&emsp;   一个简单AFNetworking的Get网络请求Demo

```
- (void) afnetworkTextClickAction:(UIButton *)sender {
    AFHTTPSessionManager *manager = [[AFHTTPSessionManager alloc]init];
    [manager GET:@"https://route.showapi.com/341-2?maxResult=2&page=1&showapi_appid=206561&showapi_timestamp=20200501230719&showapi_sign=c5deb2531727443141b89413d89a3147" parameters:nil headers:nil progress:^(NSProgress * _Nonnull downloadProgress) {

    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"相应结果：%@", responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {

    }];
    AFURLSessionManager *mm = [AFURLSessionManager new];
    [mm dataTaskWithRequest:[NSURLRequest new] uploadProgress:nil downloadProgress:nil completionHandler:nil];
    [manager.session dataTaskWithRequest:[NSURLRequest new]];
    
}

```

&emsp; `@interface AFHTTPSessionManager : AFURLSessionManager <NSSecureCoding, NSCopying>`,可以看出 ` AFHTTPSessionManager` 继承自 `AFURLSessionManager`。


<br/>

**序列化**
简介：它就是AFNetworking参数编码的序列化器，它一共有三种编码格式。

-  `AFHTTPRequestSerializer`：第一种是普通的http的编码格式也就是`mid=10&method=userInfo&dateInt=20160818`，这种格式的;

-  `AFJSONRequestSerializer`：第二种也是json编码格式的，也就是编码成`{"mid":"11","method":"userInfo","dateInt":"20160818"}`; 

-  `AFPropertyListRequestSerializer`：第三种没用过，但是看介绍接编码成pislt格式的参数.




<br/>

***
<br/>

># AFURLSessionManager

`- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration`


```

/*
初始化一个session
给manager的属性设置初始值
基本的网络请求步骤：url- request - session - task - resume
*/
- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration {
    self = [super init];
    if (!self) {
        return nil;
    }

    //设置默认的configuration，配置session
    if (!configuration) {
        configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    }

    self.sessionConfiguration = configuration;

    //设置delegate的操作队列并发的线程数量为1，也就是串行队列
    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;


    /*
      完成后若是做复杂的处理，可以选择异步串行队列
      如果更新完成后直接更新UI,选择主队列
      [NSOperationQueue mainQueue];
    */
    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];

    //默认位json响应解析
    self.responseSerializer = [AFJSONResponseSerializer serializer];
  
    //设置默认证书，无条件信任证书的https认证
    self.securityPolicy = [AFSecurityPolicy defaultPolicy];

#if !TARGET_OS_WATCH
    //网络状态监听
    //主机不断发送数据包，证明是有网的。不能判断网络是否连接到你的服务器
    self.reachabilityManager = [AFNetworkReachabilityManager sharedManager];
#endif

    //delegate = value taskid = key
    self.mutableTaskDelegatesKeyedByTaskIdentifier = [[NSMutableDictionary alloc] init];

    //使用NSLock确保线程安全
    self.lock = [[NSLock alloc] init];
    self.lock.name = AFURLSessionManagerLockName;


    //异步获取当前的session的所有未完成的task，其实讲道理来说在初始化中调用这个方法里面应该一个task都没有
    //后台任务重新回来初始化session，可能就会有先前的任务
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        for (NSURLSessionDataTask *task in dataTasks) {
            [self addDelegateForDataTask:task uploadProgress:nil downloadProgress:nil completionHandler:nil];
        }

        for (NSURLSessionUploadTask *uploadTask in uploadTasks) {
            [self addDelegateForUploadTask:uploadTask progress:nil completionHandler:nil];
        }

        for (NSURLSessionDownloadTask *downloadTask in downloadTasks) {
            [self addDelegateForDownloadTask:downloadTask progress:nil destination:nil completionHandler:nil];
        }
    }];

    return self;
}

```



<br/>

***
<br/>


># AFHTTPSessionManager

`- (instancetype)initWithBaseURL:(NSURL *)url  sessionConfiguration:(NSURLSessionConfiguration *)configuration`

```
//url的初始化和基本的配置
- (instancetype)initWithBaseURL:(NSURL *)url
           sessionConfiguration:(NSURLSessionConfiguration *)configuration
{
    self = [super initWithSessionConfiguration:configuration];
    if (!self) {
        return nil;
    }

    // Ensure terminal slash for baseURL path, so that NSURL +URLWithString:relativeToURL: works as expected
    //url 有值且没有'/'，那么在末尾加上'/'
    if ([[url path] length] > 0 && ![[url absoluteString] hasSuffix:@"/"]) {
        url = [url URLByAppendingPathComponent:@""];
    }

    self.baseURL = url;
    //设置请求、相应时的序列化默认值
    self.requestSerializer = [AFHTTPRequestSerializer serializer];
    self.responseSerializer = [AFJSONResponseSerializer serializer];

    return self;
}
```

<br/>

`- (NSURLSessionDataTask *)dataTaskWithHTTPMethod:(NSString *)method  URLString:(NSString *)URLString  parameters:(id)parameters  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgress  downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgress  success:(void (^)(NSURLSessionDataTask *, id))success  failure:(void (^)(NSURLSessionDataTask *, NSError *))failure`

```
//生成request，通过request生成task
- (NSURLSessionDataTask *)dataTaskWithHTTPMethod:(NSString *)method
                                       URLString:(NSString *)URLString
                                      parameters:(id)parameters
                                  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgress
                                downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgress
                                         success:(void (^)(NSURLSessionDataTask *, id))success
                                         failure:(void (^)(NSURLSessionDataTask *, NSError *))failure
{
    NSError *serializationError = nil;
    /*
      先调用AFHTTPRequestSerializer的requestWithMethod函数构建request；
      处理request构建产生的错误 -serializationError
      //relativeToURL表示将URLString拼接到baseURL后面
    */
    NSMutableURLRequest *request = [self.requestSerializer requestWithMethod:method URLString:[[NSURL URLWithString:URLString relativeToURL:self.baseURL] absoluteString] parameters:parameters error:&serializationError];
    if (serializationError) {
        if (failure) {
            dispatch_async(self.completionQueue ?: dispatch_get_main_queue(), ^{
                failure(nil, serializationError);
            });
        }

        return nil;
    }

    //此时的request已经将参数拼接到url后面
    __block NSURLSessionDataTask *dataTask = nil;
    dataTask = [self dataTaskWithRequest:request
                          uploadProgress:uploadProgress
                        downloadProgress:downloadProgress
                       completionHandler:^(NSURLResponse * __unused response, id responseObject, NSError *error) {
        if (error) {
            if (failure) {
                failure(dataTask, error);
            }
        } else {
            if (success) {
                success(dataTask, responseObject);
            }
        }
    }];

    return dataTask;
}
```

<br/>

`- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock  downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock  completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler`

```
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler {

   //为了解决ios8以下的一个bug，调用一个串行队列来创建dataTask
    __block NSURLSessionDataTask *dataTask = nil;
    //创建任务是安全的，那么使用系统调用的是不安全的吗？？？？
    url_session_manager_create_task_safely(^{
        //使用原生的方法， session来创建一个NSURLSessionDataTask对象
        dataTask = [self.session dataTaskWithRequest:request];
    });
    
    //为什么要给task添加一个代理呢？
    [self addDelegateForDataTask:dataTask uploadProgress:uploadProgressBlock downloadProgress:downloadProgressBlock completionHandler:completionHandler];

    return dataTask;
}
```


<br/>

`- (void)addDelegateForDataTask:(NSURLSessionDataTask *)dataTask  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock  downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock  completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler`

```
/*
    addDelegateForDataTask：这个函数并不是AFURLSessionManagerTaskDelegate的函数，
    而是AFURLSessionManager的一个函数。
    这也侧面说明了AFURLSessionManagerTaskDelegate和NSURLSessionTask的关系是由AFURLSessionManager管理的。
*/
- (void)addDelegateForDataTask:(NSURLSessionDataTask *)dataTask
                uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
              downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
             completionHandler:(void (^)(NSURLResponse *response, id responseObject, NSError *error))completionHandler
{
     //初始化delegate
    AFURLSessionManagerTaskDelegate *delegate = [[AFURLSessionManagerTaskDelegate alloc] initWithTask:dataTask];
    delegate.manager = self;
    delegate.completionHandler = completionHandler;


    /*
      taskidentifier=key delegate=value，确保task唯一
      taskDescription自行设置的，区分是否是当前的session创建的
    */
    dataTask.taskDescription = self.taskDescriptionForSessionTasks;
     //函数字面的意思是将一个session task和一个AFURLSessionManagerTaskDelegate类型的delegate变量绑在一起，而这款个绑在一起的工作是由我们的AFURLSessionManager所做。至于绑定的过程，就是以该session task的taskIdentifier为key，delegate为value，赋值给mutableTaskDelegateKeyedByTaskIdentifier这个NSMutableDictionary类型的变量。知道了这两者是关联在一起的话，马上会产生另外的一个问题  --- 为什么要关联以及怎么关联在一起？
    [self setDelegate:delegate forTask:dataTask];
  
      //设置回调块
    delegate.uploadProgressBlock = uploadProgressBlock;
    delegate.downloadProgressBlock = downloadProgressBlock;
}
```


<br/>

***
<br/>

># NSURLSession

<br/>

**` url_session_manager_create_task_safely(dispatch_block_t block)`**

```
//task和block不匹配
//taskid应该是唯一的，并发创建的task，id不唯一。
static void url_session_manager_create_task_safely(dispatch_block_t block) {
    if (NSFoundationVersionNumber < NSFoundationVersionNumber_With_Fixed_5871104061079552_bug) {
        //源代码作者对这个bug在官方做了说明
        // Fix of bug
        // Open Radar:http://openradar.appspot.com/radar?id=5871104061079552 (status: Fixed in iOS8)
        // Issue about:https://github.com/AFNetworking/AFNetworking/issues/2093
        dispatch_sync(url_session_manager_creation_queue(), block);//同步，url_session_manager_creation_queue()创建了一个串行队列
    } else {
        block();
    }
}

```



<br/>

***
<br/>

># AFURLRequestSerialization

**`- (NSMutableURLRequest *)requestWithMethod:(NSString *)method  URLString:(NSString *)URLString  parameters:(id)parameters   error:(NSError *__autoreleasing *)error`**

```
//
- (NSMutableURLRequest *)requestWithMethod:(NSString *)method
                                 URLString:(NSString *)URLString
                                parameters:(id)parameters
                                     error:(NSError *__autoreleasing *)error
{
    //断言若是nil，直接打印出来
    NSParameterAssert(method);
    NSParameterAssert(URLString);

    //传进来的是一个字符串，在这里他帮你转成url
    NSURL *url = [NSURL URLWithString:URLString];

    NSParameterAssert(url);

    NSMutableURLRequest *mutableRequest = [[NSMutableURLRequest alloc] initWithURL:url];
    mutableRequest.HTTPMethod = method;

    //将request的各种属性进行遍历，给NSURLMutableURLRequest自带的属性赋值
    for (NSString *keyPath in AFHTTPRequestSerializerObservedKeyPaths()) {
        //给设置过的属性，添加到request（如：timeout）
        if ([self.mutableObservedChangedKeyPaths containsObject:keyPath]) {
            //通过kvc动态的给mutableRequest添加value
            [mutableRequest setValue:[self valueForKeyPath:keyPath] forKey:keyPath];
        }
    }


    //将传入的参数进行编码，拼接到url后并返回 coount=5&start=1
    mutableRequest = [[self requestBySerializingRequest:mutableRequest withParameters:parameters error:error] mutableCopy];

	return mutableRequest;
}
```

<br/>

`- (NSURLRequest *)requestBySerializingRequest:(NSURLRequest *)request  withParameters:(id)parameters  error:(NSError *__autoreleasing *)error`

```
//通过requestserializtion把参数转化成了查询字符串：请求头设置、默认属性、属性监听
- (NSURLRequest *)requestBySerializingRequest:(NSURLRequest *)request
                               withParameters:(id)parameters
                                        error:(NSError *__autoreleasing *)error
{
    NSParameterAssert(request);

    NSMutableURLRequest *mutableRequest = [request mutableCopy];


    /*
      请求行（状态行）：get，url(资源的路径)/URI(资源的唯一标志符， url属于URI的子集)， http协议1.1
      请求头：content-type， accept-language
      请求体：get/post get参数拼接在url后面 post数据放在body
    */
    [self.HTTPRequestHeaders enumerateKeysAndObjectsUsingBlock:^(id field, id value, BOOL * __unused stop) {
        if (![request valueForHTTPHeaderField:field]) {
            [mutableRequest setValue:value forHTTPHeaderField:field];
        }
    }];

    NSString *query = nil;
    //传入的字典转化字符串
    if (parameters) {
        //自定义解析方式
        if (self.queryStringSerialization) {
            NSError *serializationError;
            query = self.queryStringSerialization(request, parameters, &serializationError);

            if (serializationError) {
                if (error) {
                    *error = serializationError;
                }

                return nil;
            }
        } else {
            //默认解析的方式，dic- count=5&start=1
            switch (self.queryStringSerializationStyle) {
                case AFHTTPRequestQueryStringDefaultStyle:
                    //将parameters传入这个c函数
                    query = AFQueryStringFromParameters(parameters);
                    break;
            }
        }
    }


    //最后判断request中是否包含了GET、HEAD、DELETE(都包含在HTTPMethodsEncodingParametersInURI).
    //因为这几个method的query是拼接到url后面的。而POST、PUT是把query拼接到http body中的。
    if ([self.HTTPMethodsEncodingParametersInURI containsObject:[[request HTTPMethod] uppercaseString]]) {
        if (query && query.length > 0) {
            mutableRequest.URL = [NSURL URLWithString:[[mutableRequest.URL absoluteString] stringByAppendingFormat:mutableRequest.URL.query ? @"&%@" : @"?%@", query]];
        }
    } else {
        // #2864: an empty string is a valid x-www-form-urlencoded payload
        if (!query) {
            query = @"";
        }

        //函数会判断request的Content-Type是否设置了，若果没有，就默认设置为application/x-www-form-urlencoded
        //application/x-www-form-urlencoded是常用的表单发包方式，普通的表单提交，或者js发包，默认都是通过这种方式的。
        if (![mutableRequest valueForHTTPHeaderField:@"Content-Type"]) {
            [mutableRequest setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
        }
        //设置请求体
        [mutableRequest setHTTPBody:[query dataUsingEncoding:self.stringEncoding]];
    }

    return mutableRequest;
}



```

<br/>

***
<br/>

&emsp;  AFNetworking很好用，这是有目共睹的，很多初学者，在每处用到网络请求的地方会直接拿`AFNetworking `实例去请求接口数据，虽然方便，但是为后来的代码维护带来很多的问题.

```
[[AFHTTPSessionManager manager] POST:URLStr parameters:parameters success:^(NSURLSessionDataTask *task, id responseObject) {
        
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        
    }];

```

问题：不知道在开发中，有没有遇到有的类库不在维护，后面出现问题不知如何修改。若是像上面那样做，后面AFNetoworking不在更新维护出问题，你要一个一个修改吗？

解决方法：就是对请求的Post、Get方法进行再次封装，就算后面出现问题，只要对Get、Post封装方法进行修改就好了啊。这就是菜鸟和老鸟的区别！

新建一个网络请求类：`HGNetWorkManager`

![HGNetWorkManager 头文件](https://upload-images.jianshu.io/upload_images/2959789-201e18072d0ea67d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

+ (NSURLSessionDataTask *)postRequestWithURLString:(NSString *)URLString parameters:(NSDictionary *)parameters success:(SuccessHanddle)success failure:(FailureHandle)failure {
    //对汉字进行转码
    URLString = [URLString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    return [self.manager POST:URLString parameters:parameters progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        if (success) {
            success(responseObject);
        }
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        if (failure) {
            failure(error);
        }
    }];
}

+ (NSURLSessionDataTask *)getRequestWithURLString:(NSString *)URLString parameters:(NSDictionary *)parameters success:(SuccessHanddle)success failure:(FailureHandle)failure {
    URLString = [URLString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    return [self.manager GET:URLString parameters:parameters progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        if (success) {
            success(responseObject);
        }
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        if (failure) {
            failure(error);
        }
    }];
}


```



<br/>

***
<br/>

