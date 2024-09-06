> <h3/>
- [**TRTC**](#TRTC)
	- [捕获本地音频数据处理](#捕获本地音频数据处理)
- [**腾讯语音识别器QCloudRealTimeRecognizer**](#腾讯语音识别器QCloudRealTimeRecognizer)
	- [实时处理语音识别为文字](#实时处理语音识别为文字)
- [**YYKit**](./../Objective-C/YYKit.md)
- [**‌YTKNetwork**](#YTKNetwork)
- [**热修复OCRunnerArm64**](#热修复OCRunnerArm64)
- [**动画库lottie-ios**](#动画库lottie-ios)
- [**推送通知库GTSDK**](#推送通知库GTSDK)
- [**广告标识IDFA**](#广告标识IDFA)
- [**YYKit**](#YYKit)
	- [递归解析模型](#递归解析模型)
	- [将输入框中的表情文字变为图片](#将输入框中的表情文字变为图片)
- [**‌YTKNetwork**](#YTKNetwork)
- [**聊天模块**](#聊天模块)
	- [发送消息滚动到最底部](#发送消息滚动到最底部)
	- [输入框匹配表情包&@字符](#输入框匹配表情包&@字符)
- [**OC高级应用**](#OC高级应用)
	- [UILabel行高计算容器高度](#UILabel行高计算容器高度)
	- [神坑-sectionHeader有段空白](#神坑-sectionHeader有段空白)
- [**Swift应用**](#Swift应用)
	- [数据模型继承Codable](./../Swift/数据.md#继承Codable的结构体)
	- [网络请求配置自定义](#网络请求配置自定义)
	- [double类型判断是否有限值-isFinite](#double类型判断是否有限值-isFinite)




<br/>

***

<br/><br/><br/>

> <h1 id="TRTC">TRTC</h1>



<br/><br/><br/>

> <h2 id="捕获本地音频数据处理">捕获本地音频数据处理</h2>


下面代理方法是腾讯云实时音视频 SDK (TRTC) 中 TRTCAudioFrameDelegate 代理方法的一部分。这个方法用于捕获到本地麦克风采集的音频帧，开发者可以通过实现这个方法来获取和处理本地音频数据。

```
- (void)onCapturedAudioFrame:(TRTCAudioFrame *)frame
```


- 参数说明
	- frame：一个 TRTCAudioFrame 对象，包含当前捕获的音频数据和相关信息。

- TRTCAudioFrame 结构
	- TRTCAudioFrame 是一个音频帧对象，包含以下属性：

		- data：音频数据
		- sampleRate：采样率
		- channel：声道数
		- timestamp：时间戳


**方法作用**

这个方法的主要作用是让开发者可以对本地捕获的音频数据进行处理，例如：

- 实现音频处理算法（如回声消除、降噪等）
- 实现自定义音频效果（如音频混音、变声等）
- 录制本地音频
- 实现音频数据分析

<br/>

**案例：**

```
#import "ViewController.h"
#import <TXLiteAVSDK_TRTC/TXLiteAVSDK.h>

@interface ViewController () <TRTCAudioFrameDelegate>

@property (nonatomic, strong) TRTCEngine *trtcEngine;
@property (nonatomic, strong) NSFileHandle *audioFileHandle;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 初始化 TRTC 实例
    self.trtcEngine = [TRTCEngine sharedInstance];
    self.trtcEngine.delegate = self;
    
    // 设置音频帧回调代理
    [self.trtcEngine setAudioFrameDelegate:self];
    
    // 开始本地音频采集
    [self.trtcEngine startLocalAudio];
    
    // 创建文件用于保存音频数据
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/captured_audio.pcm"];
    [[NSFileManager defaultManager] createFileAtPath:path contents:nil attributes:nil];
    self.audioFileHandle = [NSFileHandle fileHandleForWritingAtPath:path];
}

- (void)onCapturedAudioFrame:(TRTCAudioFrame *)frame {
    // 获取音频数据
    NSData *audioData = [NSData dataWithBytes:frame.data length:frame.length];
    
    // 将音频数据写入文件
    [self.audioFileHandle writeData:audioData];
}

- (void)dealloc {
    [self.audioFileHandle closeFile];
    [self.trtcEngine stopLocalAudio];
    [TRTCEngine destroySharedInstance];
}

@end

```


<br/>

***
<br/><br/><br/>

> <h1 id="腾讯语音识别器QCloudRealTimeRecognizer">腾讯语音识别器QCloudRealTimeRecognizer</h1>

<br/><br/><br/>

> <h2 id="实时处理语音识别为文字">实时处理语音识别为文字</h2>

```
- (void)realTimeRecognizerOnSliceRecognize:(QCloudRealTimeRecognizer *)recognizer result:(QCloudRealTimeResult *)result;

```

这个方法是 QCloud Real-Time Speech Recognition SDK 中的一个回调方法，用于实时处理语音识别结果。每当识别到一段语音（slice）时，该方法会被调用，并提供当前的识别结果


**参数说明**

```
recognizer：当前触发事件的识别器实例。

result：包含当前识别结果的 QCloudRealTimeResult 对象。
```


<br/>

- QCloudRealTimeResult 结构
- QCloudRealTimeResult 是一个包含识别结果的对象，通常包括以下属性：

```
text：识别到的文本内容。
startTime：该段识别结果的起始时间戳。
endTime：该段识别结果的结束时间戳。
isFinal：是否是最终结果。
```

- **方法作用**
	- 这个方法的主要作用是让开发者可以实时处理语音识别结果，例如：

		- 显示实时字幕或转录文本。
		- 处理语音命令。
		- 实现实时语音输入功能。


<br/>

案例代码：

```
#import "ViewController.h"
#import <QCloudRealTimeRecognizer/QCloudRealTimeRecognizer.h>

@interface ViewController () <QCloudRealTimeRecognizerDelegate>

@property (nonatomic, strong) QCloudRealTimeRecognizer *recognizer;
@property (nonatomic, strong) UILabel *resultLabel;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 初始化语音识别器
    self.recognizer = [[QCloudRealTimeRecognizer alloc] init];
    self.recognizer.delegate = self;
    
    // 配置识别器（具体参数根据需求设置）
    QCloudRealTimeRecognizerConfig *config = [[QCloudRealTimeRecognizerConfig alloc] init];
    config.language = @"en-US";
    [self.recognizer setConfig:config];
    
    // 创建并设置显示识别结果的标签
    self.resultLabel = [[UILabel alloc] initWithFrame:CGRectMake(20, 50, self.view.bounds.size.width - 40, 100)];
    self.resultLabel.numberOfLines = 0;
    [self.view addSubview:self.resultLabel];
    
    // 开始语音识别
    [self.recognizer startRecognize];
}

// 实现实时语音识别回调方法
- (void)realTimeRecognizerOnSliceRecognize:(QCloudRealTimeRecognizer *)recognizer result:(QCloudRealTimeResult *)result {
    if (result.isFinal) {
        NSLog(@"Final result: %@", result.text);
    } else {
        NSLog(@"Partial result: %@", result.text);
    }
    
    // 更新显示的识别结果
    dispatch_async(dispatch_get_main_queue(), ^{
        self.resultLabel.text = result.text;
    });
}

- (void)dealloc {
    [self.recognizer stopRecognize];
}

@end

```



<br/>

***

<br/><br/><br/>

> <h1 id="YYKit">YYKit</h1>

<br/><br/><br/>

> <h2 id="递归解析模型">[递归解析模型](./../Objective-C/YYKit#模型转换为字典或者数组)</h2>



<br/><br/><br/>

># <h2 id="将输入框中的表情文字变为图片">[将输入框中的表情文字变为图片](./../Objective-C/YYKit#输入框中的表情文字变为图片)</h2>





<br/>

***
<br/><br/><br/>

> <h1 id="YTKNetwork">YTKNetwork</h1>

YTKNetwork 是一个基于 AFNetworking 的轻量级网络库，用于简化 iOS 应用中的网络请求。它封装了常见的网络请求操作，提供了更高层次的接口，使得网络请求的代码更加简洁和易维护。

<br/>

- **YTKNetwork 的主要特点**
	- 请求的管理：可以方便地管理和组织网络请求。
	- 缓存支持：内置缓存机制，可以自动缓存网络请求的结果。
	- 链式请求：支持多个请求按顺序依次执行。
	- 插件机制：可以添加自定义插件，进行请求前后处理。


<br/> <br/>


**简单案例**

- 1.配置 YTKNetwork

在 AppDelegate 中进行全局配置：

```
#import <YTKNetwork/YTKNetwork.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    YTKNetworkConfig *config = [YTKNetworkConfig sharedConfig];
    config.baseUrl = @"https://api.example.com";
    return YES;
}
```

<br/>

- 2.创建请求类

创建一个继承自 YTKRequest 的请求类，每个请求类对应一个具体的 API 请求


```
#import <YTKNetwork/YTKNetwork.h>

@interface GetUserInfoApi : YTKRequest

- (instancetype)initWithUserId:(NSString *)userId;

@end

@implementation GetUserInfoApi {
    NSString *_userId;
}

- (instancetype)initWithUserId:(NSString *)userId {
    self = [super init];
    if (self) {
        _userId = [userId copy];
    }
    return self;
}

- (NSString *)requestUrl {
    return [NSString stringWithFormat:@"/user/%@", _userId];
}

- (YTKRequestMethod)requestMethod {
    return YTKRequestMethodGET;
}

//若是有参数，配置参数
- (id)requestArgument {
    NSMutableDictionary *dic = [NSMutableDictionary new];
    [dic setValue:self.roomId forKey:@"roomId"];
    [dic setValue:self.pv forKey:@"pv"];
   
    return dic;
}

//缓存：可以通过重载 cacheTimeInSeconds 方法来实现缓存
- (NSInteger)cacheTimeInSeconds {
    // 缓存时间，单位为秒，这里设置为1分钟
    return 60;
}



//自定义请求头：通过重载 requestHeaderFieldValueDictionary 方法可以自定义请求头
- (NSDictionary<NSString *,NSString *> *)requestHeaderFieldValueDictionary {
    return @{
        @"Authorization": @"Bearer token",
        @"Accept": @"application/json"
    };
}



//在请求完成后（无论是成功还是失败）调用。你可以重写这个方法来进行一些统一的后处理工作，例如解析数据、记录日志或更新 UI 等。
- (void)requestCompleteFilter {
    [super requestCompleteFilter];
    
    // 打印请求的 URL
    NSLog(@"Request URL: %@", self.requestUrl);
    
    // 打印返回的 JSON 数据
    NSDictionary *response = self.responseJSONObject;
    NSLog(@"Response JSON: %@", response);
    
    // 可选：将 JSON 数据转化为字符串以便更好地打印
    NSError *error;
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:response options:NSJSONWritingPrettyPrinted error:&error];
    if (!error) {
        NSString *jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
        NSLog(@"Formatted Response JSON: %@", jsonString);
    } else {
        NSLog(@"Error converting JSON to string: %@", error);
    }
    
    // 打印错误信息（如果有）
    if (self.error) {
        NSLog(@"Error: %@", self.error);
    }
}
@end
```

<br/>

- 4.发送请求

在需要发送请求的地方，创建请求对象并发送请求：

```
GetUserInfoApi *api = [[GetUserInfoApi alloc] initWithUserId:@"12345"];
[api startWithCompletionBlockWithSuccess:^(__kindof YTKBaseRequest * _Nonnull request) {
    NSDictionary *response = request.responseJSONObject;
    NSLog(@"User info: %@", response);
} failure:^(__kindof YTKBaseRequest * _Nonnull request) {
    NSLog(@"Failed: %@", request.error);
}];
```



<br/>

***
<br/><br/><br/>

> <h1 id="热修复OCRunnerArm64">热修复OCRunnerArm64</h1>

&emsp; OCRunnerArm64 是一个用于 iOS 的开源库，专注于实现运行时的 Objective-C 代码动态编译和执行。它为 iOS 提供了一种热修复的解决方案，特别是在处理 arm64 架构的设备上。通过使用 OCRunner，你可以在应用运行时动态加载和执行 Objective-C 代码，从而修复或者调整应用的功能，而不需要重新发布整个应用。

<br/>

**主要功能**

- 动态代码执行：OCRunner 可以动态加载并执行 Objective-C 代码，这使得它在需要热修复的场景中非常有用。
- 轻量级：相比于一些其他的热修复解决方案，OCRunner 是一个相对轻量级的实现。
- arm64 支持：OCRunner 专门针对 arm64 架构进行了优化。

<br/>

**使用场景**

&emsp; OCRunner 的主要使用场景是 iOS 应用的热修复，即在应用已经发布后，通过动态加载和执行修复代码，解决潜在的 Bug 或者添加新的功能，而无需重新提交 App Store。


<br/><br/>

```
pod 'OCRunnerArm64'
```


<br/><br/>

- **修复案例**

初始化 OCRunner

首先，在你的 AppDelegate 或者其他适合的位置初始化 OCRunner：

```
#import <OCRunner/OCRunner.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 初始化 OCRunner
    [OCRunner run];
    return YES;
}

```

<br/>

- **编写热修复代码**

假设你在应用中有一个错误的功能，需要在运行时修复它。你可以编写一个修复代码文件（通常是 .txt 或者 .runt 文件），其中包含需要动态执行的 Objective-C 代码。

比如，你有一个原始方法：

```
- (NSString *)sayHello {
    return @"Hello, World!";
}
```

现在你发现这个方法返回的字符串有问题，你想要修复它，使其返回 "Hello, OCRunner!"。

创建一个修复代码文件 fix.txt：

```
%hook YourClassName

- (NSString *)sayHello {
    return @"Hello, OCRunner!";
}

%end
```


<br/>

- **加载和执行修复代码**

你可以将修复代码文件打包在应用内，或者从服务器上下载。在需要时，加载并执行这个文件：

```
NSString *fixFilePath = [[NSBundle mainBundle] pathForResource:@"fix" ofType:@"txt"];
NSString *fixCode = [NSString stringWithContentsOfFile:fixFilePath encoding:NSUTF8StringEncoding error:nil];

[OCRunner eval:fixCode];

```


<br/>

- **测试修复**

在你的应用中，当你调用 sayHello 方法时，它应该会返回 "Hello, OCRunner!" 而不是原来的 "Hello, World!"。

<br/>

- **动态更新修复代码**

为了实现真正的热修复，你可以将修复代码存储在服务器上，并在应用启动时从服务器下载最新的修复代码，然后动态执行。例如：


```
NSURL *url = [NSURL URLWithString:@"https://example.com/fix.txt"];

NSData *data = [NSData dataWithContentsOfURL:url];
NSString *fixCode = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

[OCRunner eval:fixCode];

```


<br/>

***
<br/><br/><br/>


> <h1 id="动画库lottie-ios">动画库lottie-ios</h1>

[**动画库lottie-ios(Github地址)**](https://github.com/airbnb/lottie-ios)


[iOS Lottie动画接入过程详解(掘金)](https://juejin.cn/post/6844903576272125966)

[Lottie-iOS的应用及部分源码分析(掘金)](https://juejin.cn/post/6844903609478414343)



<br/>

***

<br/><br/><br/>

> <h1 id="推送通知库GTSDK">推送通知库GTSDK</h1>

[GTSDK官网iOS集成](https://docs.getui.com/getui/mobile/ios/xcode/)


<br/>

***

<br/><br/><br/>

> <h1 id="广告标识IDFA">广告标识IDFA</h1>


**IDFA介绍：**

&emsp; IDFA（Identifier for Advertising）是苹果为 iOS 设备提供的一个广告标识符。它是苹果从 iOS 6 开始引入的一个功能，用于广告跟踪和营销分析。以下是关于 IDFA 的一些关键点：

- 定义：IDFA 是一个由苹果系统生成的随机字符串，用来唯一标识设备，但它与设备硬件无关，也不包含个人可识别信息。
- 用途：IDFA 主要用于广告定位和效果测量。广告商可以使用 IDFA 来跟踪用户对广告的响应情况，比如查看广告后是否进行了应用安装或其他购买行为。
- 隐私保护：用户可以选择重置 IDFA 或者完全关闭广告跟踪。这意味着用户可以控制他们的数据如何被用于广告目的。
- 获取方式：开发人员可以通过调用苹果提供的 API 获取 IDFA，但在 iOS 14.5 之后，必须先获得用户的明确同意才能读取 IDFA。
- 变化：随着对用户隐私保护的重视增加，苹果在 iOS 14.5 中引入了 App Tracking Transparency（ATT）框架，要求所有 app 必须显示弹窗请求用户的许可才能访问 IDFA 或进行跨 app 跟踪。


&emsp; 总之，IDFA 是苹果为 iOS 设备提供的一个用于广告定位和跟踪的标识符，旨在平衡广告商的需求和个人隐私保护之间的关系。随着隐私政策的变化，获取 IDFA 的过程变得更加严格，需要用户的明确同意。


**重要方法：**

```
[ATTrackingManager requestTrackingAuthorizationWithCompletionHandler:^(ATTrackingManagerAuthorizationStatus status) {

}];
```

ATTrackingManager.requestTrackingAuthorizationWithCompletionHandler: 是一个用于请求用户授权进行跨应用广告跟踪的方法。这个方法是苹果在 iOS 14.5 中引入的 AppTrackingTransparency 框架的一部分，目的是让用户能够选择是否允许应用追踪其数据以便用于个性化广告。


<br/>

Block中的status有以下几种状态：

```
ATTrackingManagerAuthorizationStatus 类型，表示授权状态的结果。

ATTrackingManagerAuthorizationStatusNotDetermined: 授权状态未确定。

ATTrackingManagerAuthorizationStatusRestricted: 授权受到限制（例如，用户是儿童账户）。

ATTrackingManagerAuthorizationStatusDenied: 用户拒绝了跟踪授权。

ATTrackingManagerAuthorizationStatusAuthorized: 用户授权了跟踪。
```


<br/>


- **使用步骤：**
	
- 获取使用权限后，可以获取IDFA的序列号，通过调用方法：

```
NSString *idfa = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
```

若是上述方法获取不到idfa，可以通过获取模拟器的方法进行获取：

```
NSString *simulateIdfa = [SimulateIDFA createSimulateIDFA];
```


<br/>

- 存储

为了可以实时跟踪这个用户可以使用KeyChain这个东西，因为这个keyChain是跟着AppleID的，😝哈哈！这大概也算是活用功能吧！



<br/>

***
<br/><br/><br/>

> <h1 id="聊天模块">聊天模块</h1>


<br/><br/><br/>

> <h2 id="发送消息滚动到最底部">发送消息滚动到最底部</h2>

```
[self.cellArray addObject:cellModel];//模型插入到数组中

//插入到指定的cell在section、row中
[self.tableView beginUpdates];
[self.tableView insertRowAtIndexPath:[NSIndexPath indexPathForRow:self.cellList.count-1 inSection:0] withRowAnimation:UITableViewRowAnimationNone];
[self.tableView endUpdates];

if (needScrollToBottom) {//是否需要滚动到底部
    if (self.tableView.contentSize.height >= self.tableView.height) {
        [self.tableView scrollToBottomAnimated:needScrollAnimation];
    }
}
```


`beginUpdates`和`‌endUpdates`方法，苹果已经不鼓励了，可以使用
`-performBatchUpdates:completion:` 是代替 `-beginUpdates `和 `-endUpdates` 的最佳实践，它不仅是未来兼容性更好的选择，还为你提供了更多的控制和灵活性。建议在所有新的项目中使用这个方法来进行批量更新。


如下举例：

```
- (void)addNewMessage:(NSString *)newMessage {
    // 添加新消息到数据源
    [self.messages addObject:newMessage];
    
    NSIndexPath *newIndexPath = [NSIndexPath indexPathForRow:self.messages.count - 1 inSection:0];
    
    [self.tableView performBatchUpdates:^{
        // 插入新行到UITableView
        [self.tableView insertRowsAtIndexPaths:@[newIndexPath] withRowAnimation:UITableViewRowAnimationAutomatic];
    } completion:^(BOOL finished) {
        if (finished) {
            // 滚动到底部
            [self.tableView scrollToRowAtIndexPath:newIndexPath atScrollPosition:UITableViewScrollPositionBottom animated:YES];
        }
    }];
}
```


<br/><br/><br/>

> <h2 id="输入框匹配表情包&@字符">输入框匹配表情包&@字符</h2>


```
NSString *stirng = @"@你好 [嘻嘻] 很高兴见到你";
NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"\\[(\\S+?)\\]" options:NSRegularExpressionCaseInsensitive error:nil];
NSArray *matches = [regex matchesInString:string options:NSMatchingReportProgress range:NSMakeRange(0, [string length])];

return matches;
```


使用正则表达式从字符串中匹配特定模式的内容，并返回所有匹配结果。具体来说，它尝试从字符串中匹配形如 `[嘻嘻]` 的内容。


<br/>

```objc
NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"\\[(\\S+?)\\]" options:NSRegularExpressionCaseInsensitive error:nil];
```

- **说明**: 创建一个正则表达式对象，用于在字符串中查找匹配的内容。
- **`NSRegularExpression`**: 这是一个用于正则表达式匹配的类，它提供了强大的文本搜索功能。

- **`regularExpressionWithPattern:options:error:`**: 这是 `NSRegularExpression` 的一个类方法，用于生成一个正则表达式对象。
  - `pattern`: 正则表达式的模式字符串，用来定义匹配规则。
    - `\\[` 和 `\\]`：表示匹配方括号 `[` 和 `]`。由于在正则表达式中，`[` 和 `]` 是特殊字符，因此需要用 `\\` 进行转义。
    - `(\\S+?)`：这是一个捕获组，用于匹配方括号内的内容。
      - `\\S`：匹配任何非空白字符。
      - `+?`：表示最小匹配，匹配一个或多个非空白字符，直到遇到 `]` 为止。
  - `options`: 匹配选项，`NSRegularExpressionCaseInsensitive` 表示忽略大小写（但在这个场景中并不影响结果）。
  - `error`: 如果正则表达式语法有误，会通过这个参数返回错误信息，这里传 `nil` 表示不关心错误。

<br/>


```objc
NSArray *matches = [regex matchesInString:string options:NSMatchingReportProgress range:NSMakeRange(0, [string length])];
```

- **说明**: 使用正则表达式在目标字符串中查找所有符合条件的匹配项，并返回一个包含所有匹配结果的数组。

- **`matchesInString:options:range:`**: 这是 `NSRegularExpression` 的实例方法，用于查找所有匹配项。
  - `string`: 需要进行匹配的目标字符串。
  - `options`: 匹配选项，这里使用 `NSMatchingReportProgress` 表示在匹配过程中生成中间结果（通常用于大型文本的匹配），对于普通应用场景这个选项可以忽略。
  - `range`: 指定字符串的范围，`NSMakeRange(0, [string length])` 表示从字符串的第一个字符到最后一个字符。

- **`matches`**: 这是一个 `NSArray` 数组，包含了所有的匹配结果，每一个匹配结果都是一个 `NSTextCheckingResult` 对象。

<br/><br/>

**匹配结果的解释**

- **`matches`**: 每个 `NSTextCheckingResult` 对象包含了匹配到的文本的相关信息，包括匹配的范围 (`range`) 和内容。你可以通过这个对象提取到具体的匹配字符串，或是替换、标记匹配的部分。


<br/><br/>


**总结**

&emsp; 这段代码通过正则表达式从字符串中提取形如 `[嘻嘻]` 的内容，并返回匹配的结果。你可以利用返回的匹配结果进行进一步的处理，比如将 `[嘻嘻]` 这样的字符串替换为对应的表情图片，或者进行其他的文本处理操作。






<br/>

***

<br/><br/><br/>

> <h1 id="OC高级应用">OC高级应用</h1>

<br/><br/><br/>

> <h2 id="UILabel行高计算容器高度">UILabel行高计算容器高度</h2>


<br/><br/><br/>

> <h2 id="">[富文本嵌入图片](./../)</h2>


<br/><br/><br/>

> <h2 id="神坑-sectionHeader有段空白">神坑-sectionHeader有段空白</h2>


```
- (UITableView *)tableView {
    if (!_tableView) {
        _tableView = [[UITableView alloc] initWithFrame:CGRectMake(0, 30, 300, self.frame.size.height - 30 - 90) style:UITableViewStylePlain];
        _tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
        if (@available(iOS 15.0, *)) {//解决设置tableview的header顶部有段空白
            _tableView.sectionHeaderTopPadding = 0.0f;
        }
        _tableView.delegate = self;
        _tableView.dataSource = self;
        _tableView.backgroundColor = [UIColor clearColor];
        _tableView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0);
        _tableView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
    }
    
    return _tableView;
}
```










<br/>

***

<br/><br/><br/>

> <h1 id="Swift应用"> Swift应用 </h1>


<br/><br/><br/>

> <h2 id="网络请求配置自定义">网络请求配置自定义</h2>

```
let dataProvider = MoyaProvider<xxxRequest>(endpointClosure: MoyaProvider.xxxAuthorizedEndpointMapping)
```

是如何解读的？

详细解读，[请看这里](./../Swift/Moya.md#网络请求配置自定义)


<br/><br/><br/>

> <h2 id="double类型判断是否有限值-isFinite">double类型判断是否有限值-isFinite</h2>

在 Swift 中，`isFinite` 是 `FloatingPoint` 协议的一部分，`Double` 类型遵循该协议。`isFinite` 用来判断一个浮点数是否为有限值（finite value）。

具体来说：

- **有限值**：指的是任何非无穷大（`infinity`）或非非数字（`NaN`）的数字。对于 `Double` 类型的值，像 `2.00` 这样的普通数字就是有限值。
- **非有限值**：包括正无穷大（`infinity`）、负无穷大（`-infinity`）以及非数字值（`NaN`，例如 0.0 除以 0.0 的结果）。

所以，`a.isFinite` 会返回一个布尔值 `true` 或 `false`，表示 `a` 是否为有限值。

<br/>

**例子**

```swift
let a: Double = 2.00
print(a.isFinite) // 输出: true

let b: Double = Double.infinity
print(b.isFinite) // 输出: false

let c: Double = Double.nan
print(c.isFinite) // 输出: false
```

在这个例子中：

- `a.isFinite` 返回 `true`，因为 `2.00` 是一个有限的数字。
- `b.isFinite` 返回 `false`，因为 `b` 是无穷大。
- `c.isFinite` 返回 `false`，因为 `c` 是非数字（`NaN`）。

`isFinite` 方法非常有用，特别是在涉及到浮点数运算时，你可能需要确保数值是一个实际的数字而不是无穷大或 `NaN`。














