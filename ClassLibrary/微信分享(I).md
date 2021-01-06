>#***分享文本***
## ***AppDelegate 的配置***

在`AppDelegate.h`文件中，导入 ：***`#import WXApi 文件`***
```
#import "WXApi.h"
```

***`AppDelegate.m 文件中`***
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AppDelegate weixinShareRegister];
    
    return YES;
}

#pragma mark -- 微信分享
+ (void) weixinShareRegister {
        //注册appID
    [WXApi registerApp:WXShareAppID enableMTA:false];
}


//从微信分享过后点击返回应用的时候调用
- (void)onResp:(BaseResp *)resp {
    
    SendMessageToWXResp *sendResp = (SendMessageToWXResp *)resp;
    
    NSString *str = [NSString stringWithFormat:@"%d", sendResp.errCode];
    UIAlertController *ac = [UIAlertController alertControllerWithTitle:@"回调信息" message:[NSString stringWithFormat:@"onResp errCode:  %@", str] preferredStyle:UIAlertControllerStyleAlert];
    
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        [self.window.rootViewController dismissViewControllerAnimated:YES completion:nil];
    }];
    [ac addAction:okAction];
    [self.window.rootViewController presentViewController:ac animated:YES completion:nil];
}


- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    return [WXApi handleOpenURL:url delegate:self];

}

@end

```

***`HGMyController.m 文件中`***
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self testShareFunction];
}


- (void) testShareFunction {
    CGRect frame = CGRectMake(50, 100, 100, 60);
    UIButton *button = [[UIButton alloc] initWithFrame:frame];
    button.backgroundColor = [UIColor greenColor];
    [button addTarget:self action:@selector(shareButtonClick:) forControlEvents:UIControlEventTouchUpInside];
    
    [self.view addSubview:button];
    
}


//分享文本到微信
- (void)shareButtonClick:(id)sender {
    SendMessageToWXReq *req = [[SendMessageToWXReq alloc]init];
    req.bText = YES;           // 指定为发送文本
    req.text = @"hello world"; // 要发送的文本
    req.scene = WXSceneSession;// 指定发送到会话
    [WXApi sendReq:req];
}

```


***`配置好 URL types，否则无法返回自己的App`***
![配置分享的App URL types](https://upload-images.jianshu.io/upload_images/2959789-8a406946d8503fcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***效果图***
![准备发送图片](https://upload-images.jianshu.io/upload_images/2959789-c96cc72f327c5b29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信聊天效果图](https://upload-images.jianshu.io/upload_images/2959789-1728be22b7aec527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![返回App 回调的方法](https://upload-images.jianshu.io/upload_images/2959789-43269f39225f199c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
***
<br/>

>#***分享链接***

***`HGMyController.m 文件`***
```
-(void)sendUrl:(NSString*)url To:(enum WXScene)scene{
    SendMessageToWXReq *req = [[SendMessageToWXReq alloc]init];
    //是否是文档
    req.bText = NO;
    req.scene = WXSceneSession;// 分享到会话
    WXMediaMessage *medMessage = [WXMediaMessage message];
    medMessage.title = @"Harley's Main Page For Jian Shu"; // 标题
    medMessage.description = @"Harley 简书的 iOS 开发要点集结!!!";// 描述
    WXWebpageObject *webPageObj = [WXWebpageObject object];
    [medMessage setThumbImage:[UIImage imageNamed:@"icon"]];// 缩略图
    webPageObj.webpageUrl = @"https://www.jianshu.com/u/af13309587fb";
    //完成发送对象实例
    medMessage.mediaObject = webPageObj;
    req.message = medMessage;
    
    //发送分享信息
    [WXApi sendReq:req];
}
```

<br/>
***效果图：***
![准备发送](https://upload-images.jianshu.io/upload_images/2959789-0f86799075120eed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![效果图](https://upload-images.jianshu.io/upload_images/2959789-21be0fb772acb335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
大图为网页的缩略图。



<br/>
***
<br/>


># 分享Music
***`HGMyController.m 文件`***
```
- (void) wxShareMusic {
  SendMessageToWXReq *req1 = [[SendMessageToWXReq alloc]init];
    
    // 是否是文档
    req1.bText =  NO;
    
    //    WXSceneSession  = 0,        /**< 聊天界面    */
    //    WXSceneTimeline = 1,        /**< 朋友圈      */
    //    WXSceneFavorite = 2,
    
    
    req1.scene = WXSceneSession;
    
    //创建分享内容对象
    WXMediaMessage *urlMessage = [WXMediaMessage message];
    urlMessage.title = @"分享一首歌";//分享标题
    urlMessage.description = @"一首小歌,放松一下";//分享描述
    
    [urlMessage setThumbImage:[UIImage imageNamed:@"XXshar"]];//分享图片,使用SDK的setThumbImage方法可压缩图片大小
    
    //创建多媒体对象
    
    static NSString *kLinkURL = @"http://bd.kuwo.cn/yinyue/718535?from=baidu";
    
    WXMusicObject *music = [WXMusicObject object];
    music.musicUrl = kLinkURL;//分享链接
    
    //完成发送对象实例
    urlMessage.mediaObject = music;
    req1.message = urlMessage;
    
    //发送分享信息
    [WXApi sendReq:req1];
    
}
```

<br/>
***效果图***

![分享Music](https://upload-images.jianshu.io/upload_images/2959789-dedd4d1997f035d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>


>#***分享Video***
***`HGMyController.m 文件`***
```
- (void) wxShareVideo {
   SendMessageToWXReq *req1 = [[SendMessageToWXReq alloc]init];
    
    // 是否是文档
    req1.bText =  NO;
    
    //    WXSceneSession  = 0,        /**< 聊天界面    */
    //    WXSceneTimeline = 1,        /**< 朋友圈      */
    //    WXSceneFavorite = 2,
    
    
    req1.scene = 0;
    
    //创建分享内容对象
    WXMediaMessage *urlMessage = [WXMediaMessage message];
    urlMessage.title = @"分享视频";//分享标题
    urlMessage.description = @"小视频";//分享描述
    
    [urlMessage setThumbImage:[UIImage imageNamed:@"XXshar"]];//分享图片,使用SDK的setThumbImage方法可压缩图片大小
    
    //创建多媒体对象
    
    static NSString *kLinkURL = @"http://baidu.wasu.cn/kan/a9OrA?fr=v.baidu.com/browse";
    
    WXVideoObject *video = [WXVideoObject object];
    video.videoUrl = kLinkURL;//分享链接
    
    //完成发送对象实例
    urlMessage.mediaObject = video;
    req1.message = urlMessage;
    
    //发送分享信息
    [WXApi sendReq:req1];

}

```
<br/>
***效果图***
![分享Video](https://upload-images.jianshu.io/upload_images/2959789-ba4ba79b559ee50b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>

>#***分享图片***
***`HGMyController.m 文件`***
```
- (void) wxShareVideo {
       SendMessageToWXReq *req1 = [[SendMessageToWXReq alloc]init];
    
    // 是否是文档
    req1.bText =  NO;
    
    //    WXSceneSession  = 0,        /**< 聊天界面    */
    //    WXSceneTimeline = 1,        /**< 朋友圈      */
    //    WXSceneFavorite = 2,
    
    
    req1.scene = 0;
    
    //创建分享内容对象
    WXMediaMessage *urlMessage = [WXMediaMessage message];
    urlMessage.title = @"分享视频";//分享标题
    urlMessage.description = @"小视频";//分享描述
    
    [urlMessage setThumbImage:[UIImage imageNamed:@"XXshar"]];//分享图片,使用SDK的setThumbImage方法可压缩图片大小
    
    //创建多媒体对象
    
    static NSString *kLinkURL = @"http://baidu.wasu.cn/kan/a9OrA?fr=v.baidu.com/browse";
    
    WXVideoObject *video = [WXVideoObject object];
    video.videoUrl = kLinkURL;//分享链接
    
    //完成发送对象实例
    urlMessage.mediaObject = video;
    req1.message = urlMessage;
    
    //发送分享信息
    [WXApi sendReq:req1];

}

```

<br/>
***效果图***
![分享图片](https://upload-images.jianshu.io/upload_images/2959789-837c4d58aba46acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





