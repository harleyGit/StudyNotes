- **NSURLSessionTask**
- **NSURLSessionStreamTask**


<br/>

***
<br/>

**`简要介绍：`**

**NSURLSessionDataTask, NSURLSessionDownloadTask,  NSURLSessionStreamTask,  NSURLSessionUploadTask这四个类继承自NSURLSessionTask这个核心类。<br/>
上传类NSURLSessionUploadTask继承自NSURLSessionDataTask<br/>
如下图：**

![核心Task类 与 四大类的关系图](https://upload-images.jianshu.io/upload_images/2959789-f47c269affb2c126.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`NSURLSessionTask 方法和属性`**

```
@property (readonly)                 NSUInteger    taskIdentifier;
@property (nullable, readonly, copy) NSURLRequest  *originalRequest; 
@property (nullable, readonly, copy) NSURLRequest  *currentRequest;  
//服务器对当前活动请求的响应
@property (nullable, readonly, copy) NSURLResponse *response; 
//进度
@property (readonly, strong) NSProgress *progress

//网络负载应该开始的最早日期
//对于从后台NSURLSession实例创建的任务，此属性表示网络负载不应该在此日期之前开始。 设置此属性并不能保证加载将从指定的日期开始，而只是它不会马上开始。 如果未指定，则不使用启动延迟。
//此属性对从非后台会话创建的任务没有影响。
//只适用于后台NSURLSession
@property (nullable, copy) NSDate *earliestBeginDate；


//客户期望发送的字节数的期待上限。
//为此属性设置的值应考虑HTTP头和正文数据或正文流的大小。如果未指定值，则系统将使用NSURLSessionTransferSizeUnknown。该属性由系统用来优化URL会话任务的调度。强烈建议开发人员尽可能提供近似的上限或确切的字节数，而不是接受默认值。
@property int64_t countOfBytesClientExpectsToSend;
//客户期望接收的字节数的期待上限。
//为此属性设置的值应考虑HTTP响应头和响应主体的大小。如果未指定值，则系统将使用NSURLSessionTransferSizeUnknown。该属性由系统用来优化URL会话任务的调度。强烈建议开发人员尽可能提供近似的上限或确切的字节数，而不是接受默认值。
@property int64_t countOfBytesClientExpectsToReceive；


// 实际发送和接受的字节
@property (readonly) int64_t countOfBytesReceived;
@property (readonly) int64_t countOfBytesSent;


//预计发送的数据，和 Content-Length of the HTTP request 有关
@property (readonly) int64_t countOfBytesExpectedToSend;
//预计接收，通常来自HTTP响应的内容长度标题。
@property (readonly) int64_t countOfBytesReceived;



// 任务描述
@property (nullable, copy) NSString *taskDescription;

// 将任务标记为取消
- (void)cancel;

 //当前任务的状态
@property (readonly) NSURLSessionTaskState state;

 //错误
@property (nullable, readonly, copy) NSError *error;

//暂停/恢复
- (void)suspend;
- (void)resume;

// 任务优先级
@property float priority;


/*优先级的值*/
FOUNDATION_EXPORT const float NSURLSessionTaskPriorityDefault API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
FOUNDATION_EXPORT const float NSURLSessionTaskPriorityLow API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
FOUNDATION_EXPORT const float NSURLSessionTaskPriorityHigh API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
```

<br/>
***
<br/>

&emsp; &emsp; 上面我们说了NSURLSessionTask,还有三个子类，在API中前面的两个子类没有什么还能说的了，因为都包含在了NSURLSessionTask中，唯一有一个需要说一下的，就是在NSURLSessionDownloadTask中有一个方法是父类中没有的，我们看看对这个方法的理解：

```
 //取消下载: 但取消的下载资源我们还能继续下载（恢复数据以供以后使用
//只有满足以下条件时才能恢复下载：
 //a、请求资源后，资源并未发生变化
 //b、该任务是一个HTTP或HTTPS GET请求
// c、服务器在其响应中提供ETag或Last-Modified标头（或两者都有）
 //d、服务器支持字节范围请求
// e、系统为响应磁盘空间压力而未删除临时文件

- (void)cancelByProducingResumeData:(void (^)(NSData * _Nullable resumeData))completionHandler;
```


<br/>

***
<br/>

># **NSURLSessionStreamTask 方法**

**`@interface NSURLSessionStreamTask : NSURLSessionTask`**

```
//异步地从流中读取若干个字节，并在完成时调用处理程序。
//读取minBytes或最多maxBytes字节，并在会话委托队列中调用数据或错误的完成处理程序。如果发生错误，任何未完成的读取也将失败，并且新的读取请求将立即出错。
- (void)readDataOfMinLength:(NSUInteger)minBytes maxLength:(NSUInteger)maxBytes timeout:(NSTimeInterval)timeout completionHandler:(void (^) (NSData * _Nullable data, BOOL atEOF, NSError * _Nullable error))completionHandler;


//将指定的数据异步写入流，并在完成时调用处理程序
- (void)writeData:(NSData *)data timeout:(NSTimeInterval)timeout completionHandler:(void (^) (NSError * _Nullable error))completionHandler;


//获取流
 //完成所有已排队的读取和写入，然后调用URLSession：streamTask：didBecomeInputStream：outputStream：delegate消息。
// 收到该消息时，任务对象被视为已完成，并且不会再收到任何委托消息。
- (void)captureStreams;


//排队请求以关闭底层Socket的写入结束。 所有未完成的IO将在Socket的写入端关闭之前完成。
//但是，服务器可能会继续将字节写回到客户端，因此最佳做法是从服务器继续读取，直到你收到EOF。
- (void)closeWrite;


// 排队请求以关闭底层Socket的读取端。 所有未完成的IO将在读取端关闭之前完成。 你可以继续写入服务器。
- (void)closeRead;

//开始加密握手。 握手在所有待处理IO完成后开始。 TLS认证回调被发送到会话-URLSession：task：didReceiveChallenge：completionHandler：
- (void)startSecureConnection;


//完成所有挂起的安全IO后，干净地关闭安全连接。
- (void)stopSecureConnection;
```


<br/>

 [Demo 连接点击这里](https://github.com/geniusZhangXu/AVFoundation):(【和以前的参考资料作者写的关于AVFoundation】的Demo是在一起的，这部分的demo在NSURLSession文件夹下面)



<br/>

***
<br/>

[NSURLSession 所有的都在这里(一)](https://www.cnblogs.com/taoxu/p/8962778.html)

[NSURLSession 所有的都在这里(二)](https://www.cnblogs.com/taoxu/p/9003457.html)

[深入了解NSURLSession](https://www.jianshu.com/p/94d214129d4d)

 [网络请求之NSURLSession(api篇)](https://www.jianshu.com/p/2ab336db172c)

[网络请求和及时通讯](https://www.cnblogs.com/taoxu/category/1021605.html)










