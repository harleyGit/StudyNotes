> <h2 id=''></h2>
- [**代理**](#代理)
	- [WKNavigationDelegate](#WKNavigationDelegate)
	- [WKUIDelegate](#WKUIDelegate)
- [**方法**](#方法)
- [**性能优化**](#性能优化)
	-[内存泄漏](#内存泄漏) 
- **参考资料**
	- [WKWebView使用总结](https://www.jianshu.com/p/20cfd4f8c4ff)
	- [WebKit源码调试与分析](https://jishuin.proginn.com/p/763bfbd5f874)
	- [UIWebView & WKWebView 详解上](https://www.jianshu.com/p/d346b86839a5) 





<br/>

***
<br/>

> <h1 id='代理'>代理</h1>


> <h2 id='WKNavigationDelegate'>WKNavigationDelegate</h2>

```
// 内容开始加载（当内容开始返回时调用）
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation;

// 页面加载完成
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation;

// 页面加载失败
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error;

// 收到服务器重定向请求（主机地址被重定向时调用）
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation;

// 在请求开始加载之前，决定是否跳转（用户点击网页上的链接，需要打开新页面时，将先调用这个方法）
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;

// 页面开始加载时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation;


// 在收到响应开始加载后，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler;
 
// 当主文档已committed时，如果发生错误将进行调用
- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error; 


// 如果需要证书验证，进行验证，一般使用默认证书策略即可 
- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *__nullable credential))completionHandler; 


// 9.0才能使用，web内容处理中断时会触发，可针对该情况进行reload操作，可解决部分白屏问题 
- (void)webViewWebContentProcessDidTerminate:(WKWebView *)webView NS_AVAILABLE(10_11, 9_0); 
```

<br/>

**WKNavigationDelegate方法加载流程图**

![oc27](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc27.png)

- **代理方法内部剖析：**
	- **decidePolicyForNavigationAction 剖析**
		- 当 WebContent 即将创建 DocumentLoader 加载器时，会首先触发 decidePolicyForNavigationAction 代理方法。如果我们选择 cancel ,那么浏览内核会完全忽略这一操作，后续也不再继续执行其他操作，我们可以放心的使用 cancel 取消掉我们不想加载的主文档请求，而无需担忧任何异常。但当我们选择 alllow 后，我们会进入一个稍微复杂的逻辑判断，内核代码首先判断该该链接是否是 universalLink 类型的链接，如果判断是 universalLink 类型的链接，会尝试去调起三方 app，如果能调起，则会 cancel 当前请求，否则才会走到正常的网络加载逻辑（如果需要统计 universalLink 调起情况与或建设屏蔽能力，可以再仔细阅读该处源码）
	
	- **didStartProvisionalNavigation**
		- decidePolicyForNavigationAction 方法中选择 allow 并且判断为非 universalLink 链接后，会立即触发 didStartProvisionalNavigation 方法，表示即将开始加载主文档。这个方法看似只是对 decidePolicyForNavigationAction 方法的确认，但是值得思考的问题是方法名中的 Provisional 究竟是什么意思。其实，页面开始页面加载后为了更好的区分加载的各阶段，会将网络加载的初始阶段命名为临时状态，此时的页面是不会记入历史的，直到接收到首个数据包，才会对当前页面进行 committed 提交，并触发didCommitNavigation 方法通知 UIProcess 进程该事件，同时将网络 data 提交给 WebContent 进行渲染树生成。我们可由此引申出下一个问题，即 didFailProvisionalNavigation 与 didFailNavigation 的关系。

	- **didFailProvisionalNavigation 与 didFailNavigation 的分别在什么时候执行？他们之间有什么关系？**
		- 当 NetworkProcess 进程发生网络错误时，错误首先由 NSURLSession 回调到 WebContent 层。WebContent 会判断当前主文档加载状态，如果处于临时态，则错误会回调给 didFailProvisionalNavigation 方法；如果处于提交态，则错误会回调给 didFailNavigation 方法。


			![oc28](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc28.png)
			
			主文档加载状态图



	- **didFinishNavigation 究竟什么时候执行？与页面上屏是否有关？**
		- 我们已经理解了 NetworkProcess 层也是使用 NSURLSession 加载主文档的。当 NSURLSession 接收到 finish 事件时，会将该消息通过进程通信方式传递给 WebContent 进程，WebContent 进程再传递给 UIProcess 进程，直到被我们的代理方法响应。因此 didFinishNavigation 在 NSURLSession 的网络加载结束时就会触发，但因为跨了两次进程通信，因此对比网络层，实际上是有一定的延迟的。与子资源加载和页面上屏无时间先后关系




<br/>
<br/>
<br/>


> <h2 id='WKUIDelegate'>WKUIDelegate</h2>

&emsp; JS交互时会用到这个代理，本文讨论的需求不涉及JS交互.









<br/>

***
<br/>

> <h1 id='方法'>方法</h1>

<br/>

> 1.**URL拦截**

```
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;



//案例
//针对一次action来决定是否允许跳转，允许与否都需要调用decisionHandler，比如decisionHandler(WKNavigationActionPolicyCancel);
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler
{
    //可以通过navigationAction.navigationType获取跳转类型，如新链接、后退等
    NSURL *URL = navigationAction.request.URL;
    //判断URL是否符合自定义的URL Scheme
    if([URL.scheme isEqualToString:SHWebViewDemoScheme]){
        //根据不同的业务，来执行对应的操作，且获取参数
        if([URL.host isEqualToString:SHWebViewDemoHostSmsLogin]){
            NSString *param = URL.query;
            NSLog(@"短信验证码登录, 参数为%@", param);//短信验证码登录, 参数为username=12323123&code=892845
            decisionHandler(WKNavigationActionPolicyCancel);
            return;
        }
    }
    decisionHandler(WKNavigationActionPolicyAllow);
}
```



<br/>

> 2.**WKUserContentController中新增方法**

```
//注册回调 
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;


//js中调用方法 window.webkit.messageHandlers.<name>.postMessage(<messageBody>)

//oc中将会收到WKScriptMessageHandler的回调
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message;

//移除
- (void)removeScriptMessageHandlerForName:(NSString *)name;
```


<br/>

***
<br/>

> <h1 id='性能优化'>性能优化</h1>

[体验优化](https://www.jianshu.com/p/5c7df5079458)


<br/>
<br/> 

> <h2 id='内存泄漏'>内存泄漏</h2>

**WebController.h**

```
#import <UIKit/UIKit.h>
#import <WebKit/WebKit.h>


NS_ASSUME_NONNULL_BEGIN

@interface WebController : UIViewController<WKUIDelegate, WKNavigationDelegate, WKScriptMessageHandler>

@property(nonatomic, strong) WKWebView *webView;
@property(nonatomic, strong) WKUserContentController *userContentController;

@end

NS_ASSUME_NONNULL_END
```


**WebController.m**

```
#import "WebController.h"

@interface WebController ()

@end

@implementation WebController

- (WKUserContentController *)userContentController {
    if (!_userContentController) {
        _userContentController =[[WKUserContentController alloc]init];
    }
    
    return  _userContentController;
}

- (WKWebView *)webView {
    if (!_webView) {
        WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc]init];
        configuration.userContentController= self.userContentController;
        
        _webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:configuration];
        
        _webView.UIDelegate=self;
        
        _webView.navigationDelegate=self;
    }
    
    return  _webView;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    //注册一个name为sayhello的js方法
    [self.userContentController addScriptMessageHandler:self name:@"sayhello"];
    
    [self.view addSubview:self.webView];
    
    [self.webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"https://juejin.cn/post/6844903951049949197"]]];
    
}


- (void)dealloc{
    
    //这里需要注意，前面增加过的方法一定要remove掉。
    [self.userContentController removeScriptMessageHandlerForName:@"sayhello"];
    
}

#pragma mark - WKScriptMessageHandler

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    
        NSLog(@"name:%@\\\\n body:%@\\\\n          frameInfo:%@\\\\n",message.name,message.body,message.frameInfo);
    
}



@end

```

&emsp; 像上述使用 `- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;` 这个方法存在内存泄漏的问题，那该如何解决(通过dealloc这个析构方法没有调用)？


&emsp；这里`WKUserContentController`的实例对象方法`addScriptMessageHandler`在`scriptMessageHandler`参数传入控制器本身(猜测addScriptMessageHandler将会对scriptMessageHandler参数传入的对象做强引用,这点开发文档没有说明),而控制器又强引用了webView,然后webView又强引用了configuration,configuration又强引用了WKUserContentController对象,所以导致了引用循环,从而导致控制器不被释放的问题.）因此还需要进一步改进。


**解决方法：**

&emsp；只需要将`[userContentController removeScriptMessageHandlerForName:@"sayhello"];`这句话挪一下位置就可以了。我们可以在WebController里加一个方法：

```
 -(void)popController {

        [self.userContentController removeScriptMessageHandlerForName:@"sayhello"];

        [self.navigationController popViewControllerAnimated:YES];
}
```
在控制器pop(或者dismiss)回去的时候remove就可以了。      

由于日常开发中用到webView的界面大部分都是二级以上的界面 ，所以在pop时remove是可行的。




<br/>
<br/> 

> <h2 id=''></h2>



<br/>
<br/> 

> <h2 id=''></h2>




<br/>

***
<br/>

> <h1 id=''></h1>






<br/>

***
<br/>

> <h1 id=''></h1>





<br/>

***
<br/>

> <h1 id=''></h1>




<br/>

***
<br/>

> <h1 id=''></h1>




<br/>

***
<br/>

> <h1 id=''></h1>


<br/>

***
<br/>

> <h1 id=''></h1>












