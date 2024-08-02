>
- [**TRTC**](#TRTC)
	- [捕获本地音频数据处理](#捕获本地音频数据处理)
- [**腾讯语音识别器QCloudRealTimeRecognizer**](#腾讯语音识别器QCloudRealTimeRecognizer)
	- [实时处理语音识别为文字](#实时处理语音识别为文字)
- [**YYKit**](./../Objective-C/YYKit.md)
- [**‌YTKNetwork**](#YTKNetwork)



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

- (NSInteger)cacheTimeInSeconds {
    // 缓存时间，单位为秒，这里设置为1分钟
    return 60;
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

















