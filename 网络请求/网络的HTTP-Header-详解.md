>#HTTP Header 意义
##`Request 响应头`


![屏幕快照 2019-03-02 下午12.57.21.png](https://upload-images.jianshu.io/upload_images/2959789-6fe553775e052b82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



HTTP请求步骤：
-  请求行 + 请求头 + 请求体 == request(这三个组成了request请求)；
-  建立任务task；
-  resume；
-  传输数据；


HTPP传输步骤：
-  域名---> 换取解析为IP；
-  传输层tcp  的三次握手；
-  head数据；
-  空行；
-  请求体body；

server对HTTP的数据进行处理：
封装 head+body：
-  reponse： head + body；
- 传完之后断开连接，也就是4次挥手；




<br/>

token值加入请求头
```

//实例化AFHTTPSessionManagerAFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
//调出请求头
manager.requestSerializer = [AFJSONRequestSerializer serializer];
//将token封装入请求头
[manager.requestSerializer setValue:TOKENforHTTPHeaderField:@"token-id"];

```


<br/>
***
<br/>

>#  断点续传

#`duadi`
```
@property(nonatomic, retain)NSURLSession *session;
@property(nonatomic, retain)NSURLSessionDownloadTask *downloadTask;

//断点续传测试
- (void)breakpointResumeTest {
    if (self.session) {
        //应该知道原来 文件
        NSFileManager *fm = [NSFileManager defaultManager];
        NSData *data = [fm contentsAtPath:@"未下载完的临时路径pathurl"];
        NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"%@--", str);
        //resumeData 编码后是对应的文件是键值对，有 xml 文件 中有 url --> data 写入 strema流 ---> tcp
        //在原来的数据上进行resume
        self.downloadTask = [self.session downloadTaskWithResumeData:data];
        
        //self.resumeData = nil; 这个data可以从沙盒里面取，沙盒比文件速度快，缓存又比沙盒快
    }
}

//取消下载
- (void) cancelDownload {
    __weak typeof(self) weakSelf = self;
    [self.downloadTask cancelByProducingResumeData:^(NSData * _Nullable resumeData) {
        //先存到内存
        //weakSelf.resumeData  = resumeData;
        weakSelf.downloadTask = nil;
        
        //存到沙盒
        [resumeData writeToFile:@"未下载完的临时路径pathurl" atomically:NO];
    }];
}
```




[HTTP Header 详解](https://blog.csdn.net/zzzzzdddddxxxxx/article/details/53261881)

[HTTP 常用 Header 讲解](https://blog.csdn.net/muzhenhua/article/details/47021173)

[HTTP 头组成](https://blog.csdn.net/zhenweicao/article/details/7911525)

[HTTP响应头和请求头信息对照表](http://tools.jb51.net/table/http_header)

[HTTP的请求头和响应头 以及常见的响应状态码](https://blog.csdn.net/love9099/article/details/77539907)

[网络请求简介](https://blog.csdn.net/Bolted_snail/article/details/79870134)
