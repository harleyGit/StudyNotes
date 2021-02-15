
- **请求配置**
	- 简括
	- Request 请求对象
	- 创建会话对象方式
- **Get**
- **Post**
- **NSURLSessionDataDelegate代理**
	- 代理方法
- **NSURLSessionDownloadTask文件下载**
- **NSURLSessionUploadTask上传文件**
	- Get方法
	- Post方法
- [**网络请求之NSURLSession使用**](https://blog.csdn.net/nd71pbt7/article/details/55095225)
- [**网络之数据请求**](https://www.cnblogs.com/soley/p/5483673.html)



<br/>

***
<br/>

># 请求配置

> **简括**

&emsp;NSURLSession是2013年iOS 7发布的用于替代NSURLConnection的，iOS 9之后NSURLConnection彻底退出历史舞台。

<br/>

NSURLSession一般分别两部操作：
- 第一步:通过NSURLSession的实例创建task；
- 第二步:执行task；


<br/>

对于NSURLSessionTask，也就是task，可以把它当作所谓的任务。

&emsp;&emsp;  NSURLSessionTask是一个抽象子类，它有三个具体的子类是可以直接使用分别是：`NSURLSessionDataTask`，`NSURLSessionUploadTask`和`NSURLSessionDownloadTask`。这三个类应用的三个基本网络任务分别是：获取数据、上传文件、下载文件。与数据有关的NSURLSessionDataTask也可以胜任上传下载的任务，所以经常使用到。

<br/>
<br/>


> **Request 请求对象**

**1）首先构造一个NSURL请求资源地址**

```   
// 构造URL资源地址
NSURL *url = [NSURL URLWithString:@"http://api.nohttp.net/method?name=yanzhenjie&pwd=123"];
```

<br/>

**2）创建一个NSRequest请求对象**

```
// 创建Request请求
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
// 配置Request请求
// 设置请求方法
[request setHTTPMethod:@"GET"];
// 设置请求超时 默认超时时间60s
[request setTimeoutInterval:10.0];
// 设置头部参数
[request addValue:@"gzip" forHTTPHeaderField:@"Content-Encoding"];
//或者下面这种方式 添加所有请求头信息
request.allHTTPHeaderFields=@{@"Content-Encoding":@"gzip"};
//设置缓存策略
[request setCachePolicy:NSURLRequestReloadIgnoringLocalCacheData];

```


**缓存策略：**`根据需求添加不用的设置，比如请求方式、超时时间、请求头信息`

> `NSURLRequestUseProtocolCachePolicy = 0`   //默认的缓存策略， 如果缓存不存在，直接从服务端获取。如果缓存存在，会根据response中的Cache-Control字段判断下一步操作，如: Cache-Control字段为must-revalidata, 则询问服务端该数据是否有更新，无更新的话直接返回给用户缓存数据，若已更新，则请求服务端.  

> `NSURLRequestReloadIgnoringLocalCacheData = 1`  


> `NSURLRequestIgnoringLocalAndRemoteCacheData = 4`     //忽略本地缓存，代理服务器以及其他中介，直接请求源服务端. 

> `NSURLRequestReloadIgnoringCacheData = NSURLRequestReloadIgnoringLocalCacheData` 

> `NSURLRequestReturnCacheDataElseLoad = 2`   //有缓存就使用，不管其有效性(即忽略Cache-Control字段), 无则请求服务端.  

> ` NSURLRequestReturnCacheDataDontLoad = 3`   //只加载本地缓存. 没有就失败. (确定当前无网络时使用)  

> `NSURLRequestReloadRevalidatingCacheData = 5`   //缓存数据必须得得到服务端确认有效才使用  <br/>


<br/>
<br/>

> **创建会话对象方式**

**`第一种方式`**

```
// 采用苹果提供的共享session
NSURLSession *sharedSession = [NSURLSession sharedSession];
```


<br/>

**`第二种方式`**

```
//通过NSURLSessionConfiguration方式配置不同的NSURLSession
// 构造NSURLSessionConfiguration
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
// 构造NSURLSession，网络会话；
NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
```

`通过NSURLSessionConfiguration提供了三种创建NSURLSession的方式`

>  `defaultSessionConfiguration`   //默认配置使用的是持久化的硬盘缓存，存储证书到用户钥匙链。存储cookie到shareCookie。<br/>

>`ephemeralSessionConfiguration`   //不使用永久持存cookie、证书、缓存的配置，最佳优化数据传输。

>`backgroundSessionConfigurationWithIdentifier`   //可以上传下载HTTP和HTTPS的后台任务(程序在后台运行)。在后台时，将网络传输交给系统的单独的一个进程,即使app挂起、退出甚至崩溃照样在后台执行。

可以通过NSURLSessionConfiguration统一设置超时时间、请求头等信息

```
// 构造NSURLSessionConfiguration
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
//设置请求超时为10秒钟
configuration.timeoutIntervalForRequest = 10;

//在蜂窝网络情况下是否继续请求（上传或下载）
configuration.allowsCellularAccess = NO;

//配置请求头
configuration.HTTPAdditionalHeaders =@{@"Content-Encoding":@"gzip"};
```



<br/>

***
<br/>



># Get请求

```
// 1.创建一个网络路径

NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"http://172.16.2.254/php/phonelogin?yourname=%@&yourpas=%@&btn=login",yourname,yourpass]];

// 2.创建一个网络请求

NSURLRequest *request =[NSURLRequest requestWithURL:url];

// 3.获得会话对象

NSURLSession *session = [NSURLSession sharedSession];

// 4.根据会话对象，创建一个Task任务：

NSURLSessionDataTask *sessionDataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {

        NSLog(@"从服务器获取到数据");

         /*

           对从服务器获取到的数据data进行相应的处理：

         */    
        //请求失败，打印错误信息
    if (error) {
        NSLog(@"get error :%@",error.localizedDescription);
    }else {        
        // JSON数据格式解析
        NSDictionary *object = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:&error];
        
        // 判断是否解析成功
        if (error) {
            NSLog(@"get error :%@",error.localizedDescription);
        }else {
             NSLog(@"get success :%@",object);
            // 解析成功，处理数据，通过GCD获取主队列，在主线程中刷新界面。
            dispatch_async(dispatch_get_main_queue(), ^{
                // 刷新界面....
            });
        }
    }    
  }];

    // 5.最后一步，执行任务（resume也是继续执行）:

    [sessionDataTask resume];
```




<br/>

***
<br/>

>## Post请求

```
// 1.创建一个网络路径

NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"http://172.16.2.254/php/phonelogin"]];

// 2.创建一个网络请求，分别设置请求方法、请求参数

NSMutableURLRequest *request =[NSMutableURLRequest requestWithURL:url];

request.HTTPMethod = @"POST";
//设置请求超时
[request  setTimeoutInterval:10.0f]
// 设置头部参数
[request addValue:@"gzip" forHTTPHeaderField:@"Content-Encoding"];

NSString *args = [NSString stringWithFormat:@"yourname=%@&yourpass=%@&btn=login",yourname,yourpass];

//NSString转成NSData数据类型。
request.HTTPBody = [args dataUsingEncoding:NSUTF8StringEncoding];

// 3.获得会话对象
NSURLSession *session = [NSURLSession sharedSession];

// 4.根据会话对象，创建一个Task任务
NSURLSessionDataTask *sessionDataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {

NSLog(@"从服务器获取到数据");

/*

对从服务器获取到的数据data进行相应的处理.

*/

NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:(NSJSONReadingMutableLeaves) error:nil];

}];

//5.最后一步，执行任务，(resume也是继续执行)。

[sessionDataTask resume];
```


<br/>

***
<br/>

># NSURLSessionDataDelegate代理

&emsp;&emsp;    相比NSURLConnection，NSURLSession提供了block方式处理返回数据的简便方式，但是，如果项目需要在网络请求数据的过程中，要做进一步的处理的话，需要调用NSURLSession的代理方法。

&emsp;&emsp;    通常，使用代理方法需要先设置代理对象，但是通过查看NSURLSessionDataDelegate文档，我们可以看到如下，代理属性delegate为只读状态。

`@property (nullable, readonly, retain) id  delegate;`

在类中遵循代理方法后，主要设置代理代码如下：

```

// 1.delegateQueue参数表示协议方法将会在(NSOperationQueue)队列里面执行。（session的delegate属性是只读的,所以使用如下方法设置代理。）

NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc] init]];

// 2.创建任务(因为要使用代理方法,就不需要block方式的初始化了)

NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"http://172.16.2.254/php/phonelogin?yourname=%@&yourpass=%@&btn=login",yourname,yourpass]];

NSURLSessionDataTask *task = [session dataTaskWithRequest:[NSURLRequest requestWithURL:url]];

// 3.执行任务

[task resume];

```


<br/>

> **`代理方法`**

```
#pragma mark -- NSURLSessionDataDelegate
// 1.接收到服务器的响应
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
    
    //此处需要允许处理服务器的响应，才会继续加载服务器的数据。 若在接收响应时需要对返回的参数进行处理(如获取响应头信息等),那么这些处理应该放在这个允许操作的前面
    completionHandler(NSURLSessionResponseAllow);
    
}

// 2.接收到服务器的数据（此方法在接收数据过程会多次调用）

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
    
    // 处理每次接收的数据，例如每次拼接到自己创建的数据receiveData
    [self.receiveData appendData:data];
    
}

// 3.3.任务完成时调用（如果成功，error == nil）
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
    
    if(error == nil){
        
        /*
         
         请求完成,成功或者失败的处理
         
         */
        
    }else{
        
        NSLog(@"请求失败:%@",error);
        
    }
    
}

```


<br/>

***
<br/>

># **NSURLSessionDownloadTask文件下载**

```

// 1.创建网路路径
    NSURL *url = [NSURL URLWithString:@"http://172.16.2.254/php/phonelogin/image.png"] ;
    
    // 2.获取会话
    NSURLSession *session = [NSURLSession sharedSession];
    
    // 3.根据会话，创建任务
    NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
        
        /*
         a.location是沙盒中tmp文件夹下的一个临时url,文件下载后会存到这个位置,由于tmp中的文件随时可能被删除,所以我们需要自己需要把下载的文件移动到其他地方:pathUrl.

         b.response.suggestedFilename是从相应中取出文件在服务器上存储路径的最后部分，例如根据本路径为，最后部分应该为：“image.png”
         */
        
        NSString *path = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:response.suggestedFilename];
        
        NSURL *pathUrl = [NSURL fileURLWithPath:path];
        
        // 剪切文件
        [[NSFileManager defaultManager] moveItemAtURL:location toURL:pathUrl error:nil];
        
    }];
    
    // 4.启动任务
    [task resume];

```

<br/>

***`NSURLSessionDownloadDelegate代理方法`***

***① 在所在的类中遵循代理协议；***

***② 实现代理方法；***

```

// 1.每次写入调用(会调用多次)
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite {
    
    // 可在这里通过已写入的长度和总长度算出下载进度
    CGFloat progress = 1.0 * totalBytesWritten / totalBytesExpectedToWrite; NSLog(@"%f",progress);
}

// 2.下载完成时调用

- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location {
    
    //  这里的location也是一个临时路径,需要自己移动到需要的路径(caches下面)
    NSString *filePath = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:downloadTask.response.suggestedFilename];
    
    [[NSFileManager defaultManager] moveItemAtURL:location toURL:[NSURL fileURLWithPath:filePath] error:nil];
    
}

// 3.请求成功/失败（如果成功，error为nil）
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
    
    if(error == nil){
        
        /*
         
         请求完成,成功或者失败的处理
         
         */
        
    }else{
        NSLog(@"请求失败:%@",error);
    }
    
}


```



<br/>

***
<br/>

># NSURLSessionUploadTask上传文件

<br/>

> **`Get方法`**

```

NSURLSessionUploadTask *task = [NSURLSession sharedSession] uploadTaskWithRequest:request fromFile:fileName completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        //数据处理
    };

```


<br/>


> **`Post方法`**

```

[[NSURLSession sharedSession] uploadTaskWithRequest:request fromData:body completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"-------%@", [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];

```

&emsp;&emsp;  对于Get和Post的不同点在于，用post方法需要添加网络路径的请求体body，而在实际开发中，上传文件一般使用post方式，更加安全可靠。
