
**前奏**
**注册URL Scheme**
**App在App Store的下载链接**




<br/>

***
<br/>

># **前奏**

&emsp;  我们都知道苹果手机中的APP都有一个沙盒，APP就是一个信息孤岛，相互是不可以进行通信的。但是iOS的APP可以注册自己的URL Scheme，URL Scheme是为方便app之间互相调用而设计的。我们可以通过系统的OpenURL来打开该app，并可以传递一些参数。

&emsp;  **URL Scheme**必须能唯一标识一个APP，如果你设置的URL Scheme与别的APP的URL Scheme冲突时，你的APP不一定会被启动起来。因为当你的APP在安装的时候，系统里面已经注册了你的URL Scheme。一般情况下，是会调用先安装的app。但是iOS的系统app的URL Scheme肯定是最高的。所以我们定义URL Scheme的时候，尽量避开系统app已经定义过的URL Scheme。

&emsp;  URL Schemes 有两个单词：

&emsp;  &emsp;  ①.  URL，我们都很清楚，`http://www.apple.com `就是个 URL，我们也叫它链接或网址；

&emsp;  &emsp;  ②. Schemes，表示的是一个 URL 中的一个位置——最初始的位置，即 ://之前的那段字符。比如 `http://www.apple.com` 这个网址的 Schemes 是 http。


<br/>

***

<br/>

># 注册**URL Scheme**

![URL Scheme 步骤图](https://upload-images.jianshu.io/upload_images/2959789-431ebe3f85e44562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

右键单击info.plist,按下图所示：
![info.plist生成的的代码](https://upload-images.jianshu.io/upload_images/2959789-e92d788fe8aedb81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`URL Scheme: Harley109`

根据[微信开发平台](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417694084&token=b2efd44becf512cd1ae681b36801c951a3c4240b&lang=zh_CN)对数据进行接入

`AppDelegate.h  增加 WXApiDelegate 协议。`

```
#import <UIKit/UIKit.h>
#import "WXApi.h"

@interface AppDelegate : UIResponder <UIApplicationDelegate, WXApiDelegate>

@property (strong, nonatomic) UIWindow *window;


@end
```

`AppDelegate.m  代码中使用开发工具包`

```
#pragma mark -- 微信分享
+ (void) weixinShareRegister {
    //wxd930ea5d5a258f4f 微信SDK demo里的appID
    //注册appID
    [WXApi registerApp:@"wxd930ea5d5a258f4f"];
}

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    // 接受传过来的参数
    NSString *text = [[url host] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    
    UIAlertController *ac = [UIAlertController alertControllerWithTitle:@"URL Scheme" message:text preferredStyle: UIAlertControllerStyleAlert];
    UIAlertAction *action1 = [UIAlertAction actionWithTitle:@"标题" style:UIAlertActionStyleDefault handler:nil];
    [ac addAction:action1];
    
    [self.window.rootViewController presentViewController:ac animated:YES completion:nil];

    return YES;

}
```

`HGMyController.m  添加分享事件`

```
//分享文本到微信
- (void)shareButtonClick:(id)sender {
    SendMessageToWXReq *req = [[SendMessageToWXReq alloc]init];
    req.bText = YES;           // 指定为发送文本
    req.text = @"hello world"; // 要发送的文本
    req.scene = WXSceneSession;// 指定发送到会话
    [WXApi sendReq:req];
}

//分享网页到微信
-(void)sendUrl:(NSString*)url To:(enum WXScene)scene{
    SendMessageToWXReq *req = [[SendMessageToWXReq alloc]init];
    req.bText = NO;
    req.scene = WXSceneSession;// 分享到会话
    WXMediaMessage *medMessage = [WXMediaMessage message];
    medMessage.title = @"分享网页的标题"; // 标题
    medMessage.description = @"这个就是描述啦";// 描述
    WXWebpageObject *webPageObj = [WXWebpageObject object];
    [medMessage setThumbImage:[UIImage imageNamed:@"kitty"]];// 缩略图
    webPageObj.webpageUrl = @"http://www.baidu.com";
    medMessage.mediaObject = webPageObj;
    req.message = medMessage;
    [WXApi sendReq:req];
}


- (void) testShareFunction {
    CGRect frame = CGRectMake(50, 100, 100, 60);
    UIButton *button = [[UIButton alloc] initWithFrame:frame];
    button.backgroundColor = [UIColor greenColor];
    [button addTarget:self action:@selector(shareButtonClick:) forControlEvents:UIControlEventTouchUpInside];
    
    [self.view addSubview:button];
    
}
```

![没有完善微信开发信息，完善就好](https://upload-images.jianshu.io/upload_images/2959789-e125c3d56ee92be5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![点击上图【返回 Wechat SDK Demo】事件](https://upload-images.jianshu.io/upload_images/2959789-1e561512d4901cb6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`重写AppDelegate的 application: openURL: options: 方法`

```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    return [WXApi handleOpenURL:url delegate:self];
}
```

`程序要实现和微信终端交互的具体请求与回应，因此需要实现WXApiDelegate协议的两个方法`

```
-(void) onReq:(BaseReq*)reqonReq
//是微信终端向第三方程序发起请求，要求第三方程序响应。第三方程序响应完后必须调用sendRsp返回。在调用sendRsp返回时，会切回到微信终端程序界面。
```

```
-(void) onResp:(BaseResp*)resp
//如果第三方程序向微信发送了sendReq的请求，那么onResp会被回调。sendReq请求调用后，会切到微信终端程序界面。`
//具体在此两方法中所要完成的内容由你定义，具体可参考微信开发工具包中的SDK Sample Demo源码。
```
` 如果你的程序要发消息给微信，那么需要调用WXApi的sendReq函数`
```
-(BOOL) sendReq:(BaseReq*)req
//其中req参数为SendMessageToWXReq类型。
//需要注意的是，SendMessageToWXReq的scene成员，如果scene填WXSceneSession，那么消息会发送至微信的会话内。
//如果scene填WXSceneTimeline，那么消息会发送至朋友圈。
//如果scene填WXSceneFavorite,那么消息会发送到“我的收藏”中。
//scene默认值为WXSceneSession。
```

**参考资料:**

 &emsp;&emsp;&emsp;&emsp; [微信SDK Sample Demo源码](https://open.weixin.qq.com/zh_CN/htmledition/res/dev/download/sdk/WeChatSDK_sample_iOS_1.4.2.1.zip)
 
 &emsp;&emsp;&emsp;&emsp; [打开另一个APP](https://www.jianshu.com/p/0811ccd6a65d)
 
 &emsp;&emsp;&emsp;&emsp; [使用URLScheme进行App跳转](https://blog.csdn.net/shimazhuge/article/details/79450412)
 
&emsp;&emsp;&emsp;&emsp; [分享到微信简明教程](https://www.jianshu.com/p/db2751ff334a)

<br/>

***
<br/>

># **App在App Store的下载链接**

&emsp;  基本格式是: `https://itunes.apple.com/cn/app/idxxxxxxxxxx?mt=8 `
&emsp;  这里的"xxxxxxxxxx"就是自己在[itunesconnect.apple.com](https://itunesconnect.apple.com/)中创建自己的App之后iTunesConnect为你的App创建的一个Apple ID。

&emsp;  ***查看已发布的AppID:***
![已发布App ID](https://upload-images.jianshu.io/upload_images/2959789-279557aad6185d1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，我的App 下载链接是： https://itunes.apple.com/cn/app/id1234897980?mt=8 ，在微信中也可以点开：![跳转到AppStore](https://upload-images.jianshu.io/upload_images/2959789-2f939371c3b2664a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/>

参考资料：

[微信跳转到appstore下载App](https://blog.csdn.net/sinat_35861727/article/details/70282504)

