-  **[Interview(II)](https://github.com/harleyGit/StudyNotes/blob/master/iOS/ProjectDesc/Interview(II).md)** 
-  [**iOS面试集锦**](https://github.com/ChenYilong/iOSInterviewQuestions)

<br/>

***
<br/>


- **[OC基础](#oc基础)**
	- [UILabel多适应](#UILabel多适应)
	- [不同账号内购后崩溃处理异常](#不同账号内购后崩溃处理异常)
	- [适配器、响应元是什么](#适配器响应元是什么)
	- [热更新](#热更新)
	- [代码管理](#代码管理)
- [**Swift基础**](#Swift基础)
	- [面向协议编程](#面向协议编程)
- [**性能优化**](#性能优化)
	- [循环引用解决](#循环引用解决)
	- [NSTimer循环引用解决](#NSTimer循环引用解决)
	- [NSTimer计时不准确怎么办](#NSTimer计时不准确怎么办)
	- [图片优化](#图片优化)
		 - [图片的处理](#图片的处理)
			 - [图片异步绘制](#图片异步绘制)
			 - [图片格式判断](#图片格式判断) 
	- [图片上传](#图片上传)
	- [界面保持流畅](#界面保持流畅)
	- [启动优化](#启动优化)
	- [UI卡顿优化](#UI卡顿优化)
	- [内存优化](#内存优化)
	- [包体积优化](#包体积优化)
	- [内存暴涨解决](#内存暴涨解决)
	- [文件存储优化](#文件存储优化)
- [**架构与设计模式**](#架构与设计模式)
	- [面向协议编程](#面向协议编程)
- [**底层**](#底层)
	- [RunLoop与自动释放池关系，什么时侯释放](#runloop与自动释放池关系什么时侯释放)
	- [RunLoop原理和与线程的联系](#RunLoop原理和与线程的联系)
	- [main函数之前会做什么](main函数之前会做什么)
	- [响应者链](#响应者链)
	- [OC的引用计数是存放](#OC的引用计数是存放)
	- [类库没有导入](#类库没有导入)
- [**网络**](#网络)
	- [NSURLSession与RunLoop的联系](#NSURLSession与RunLoop的联系)
	- [网络性能优化](#网络性能优化)
- [**类库** ](#类库)
	- [FishHook](#fishhook) 
	- [静态库和动态库](#静态库和动态库)
		- [库引用错误解析](#库引用错误解析)
- [**Quesion(I)**](https://rencheng.cc/2020/04/30/ios/general/iOS高级面试题/) 




<br/>

***
<br/>

># <h1 id="OC基础">[OC基础](https://my.oschina.net/antsky/blog/1475173)</h1>

<br/>

> <h2 id="UILabel多适应">UILabel多适应?</h2>

Label自适应宽度

```
+ (float)widthAuto:(NSString *)content fontSize:(CGFloat)fontSize contentheight:(CGFloat)height{

		// 列寬
		//    CGFloat contentHeight = height;
		
		// 用何種字體進行顯示
		UIFont *font = [UIFont systemFontOfSize:fontSize];
		
		// 計算出顯示完內容需要的最小尺寸
		CGSize size = CGSizeMake(100, 100);
		
		// 計算出顯示完內容需要的最小尺寸
		NSDictionary * tdic = [NSDictionary dictionaryWithObjectsAndKeys:font,NSFontAttributeName,nil];
		
		//ios7方法，获取文本需要的size，限制宽度
		CGSize  actualsize =[content boundingRectWithSize:size options:NSStringDrawingUsesLineFragmentOrigin |NSStringDrawingUsesFontLeading attributes:tdic context:nil].size;
		
		return actualsize.width;

}

```


<br/>
<br/>


> <h3 id= "不同账号内购后崩溃处理异常">不同账号内购后崩溃处理异常<h3>
 

IAP支付的过程：

- 调用IAP接口发起支付
- 支付成功，获取App Receipt票据，调用充值接口验证
- 验证通过，给用户充值虚拟货币并回调给 App

&emsp; 这里需要了解一下IAP支付的机制，每次支付行为或每笔交易被认为是一个SKPaymentTransation，只有当SKPaymentTransation被finishTransaction:，这次支付（交易）行为才算是正常结束了。即使这次支付途中被中断，其实也并没有丢失。假设支付没有完成 App 就退出了（比如崩溃），那么当下次 App 重启之后，只要设置了监听addTransactionObserver:，之前被中断的支付就会接着进行。

**掉单**可能会出现在哪些环节：
- 第1步，这个过程中 App 进程因为某种原因被 kill 了，其实支付行为还在系统后台进行着，苹果自己做的，很有可能扣款成功。但是这时候没法为用户充值虚拟货币。
- 第2步，App 端与自己服务器端通信失败；自己服务器端与 AppStore 服务器之间的通信失败。

&emsp; 针对第一种情况，可以在 App 一启动就设置监听，如果有未完成的支付，则会回调`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions;`这个方法，在这个方法里调用接口充值。
至于第二种情况，App 端需要做接口重试，设置一个重试的逻辑。

&emsp; 下面来描述一下我的具体做法。

&emsp; 我使用了RMStore，稍作了一些修改。新建一个数据表（字段有：`transactionIdentifier、productIdentifier、uid、transactionDate(支付时间)、rechargeDate(充值到账时间)、state`(「已支付」和「已充值」)），存每一笔交易。

&emsp; 在`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions;`这个方法里，首先根据`SKPaymentTransaction的transactionIdentifier`去本地数据表中判断这条交易：

- 如果没有，则在数据表中创建一条交易，状态为「已支付」，接着去调用充值接口；
- 如果有这条交易，并且状态为「已支付」，则不需要创建该条交易，只需要去调用充值接口；
- 如果有这条交易，并且状态为「已充值」（虚拟货币到了用户账上），则不需要创建，也不需要去充值，直接结束交易（finishTransaction:）。

&emsp; 充值接口的设置，App 端需要传这些参数：交易id、商品id、用户uid、票据，该接口在服务器端需要做一些操作：transactionIdentifier需要入库，和App Receipt对应起来，每次访问接口都要先做去重判断，如果是已经验证通过的transactionIdentifier，也返回成功。只要是访问成功，该接口都需要带上transactionIdentifier返回给 App 端，可能不止一个，因为一个App Receipt票据验证成功后返回的交易列表可能有多个

<br/>

**漏单：**

&emsp; 以上流程中可能会出现漏单的情况：当客户端向苹果支付成功后，准备向服务端发起验证，此时若网络发生抖动或者人为退出app等情况，就会出现用户充值成功，但是没加金币的情况。
解决方案：在paymentQueue:updatedTransactions:代理方法中获取到支付成功的回调，此时马上缓存订单信息（`包括支付凭证、订单号、用户id等；担心用户卸载app的，可以直接缓存到钥匙串`），然后再向服务端验证订单。若订单验证成功，则删除该项订单缓存；否则不做处理。另外，每次app启动的时候，可以延时几秒钟，去检测本地是否有未验证成功的苹果订单；如果有，就一个个重新验证。

<br/>

**刷单：**

&emsp; 我们公司的app最早的时候，服务端还是用的Http请求，也没有考虑到刷单这一块，然后被某些用户钻了空子；他们先发起苹果支付，获取一个订单号，然后拿这个订单号和之前支付成功的凭证，向服务端发起验证；然后服务端发现订单号也对得上，凭证也验证有效，就给用户加金币了；用户用同一个凭证不断的换订单号。。。。。
解决方案：首先服务端每次生成的订单号都缓存起来，客户端在请求完订单号后，将该订单号记录在内购订单对象里（赋值给SKPayment的applicationUsername属性）；然后当客户端支付成功后，把订单id和支付凭证传给服务端，服务端先拿凭证去苹果校验，苹果返回的结果中有一个transactionId（跟前面记录的applicationUsername是一致的） ，这时候服务端判断该transactionId和订单id是否一致，若一致，且该订单id是一个新id，之前没有缓存过；才算是完全的校验成功。


<br/>

**订单重复：**

&emsp; 订单重复的情况就是支付成功后，从本地拿到了错误的支付凭证（之前的凭证），去校验的时候，会发现该订单已经完成了。
我这边发现这个问题的情况是这样的：客户端发起苹果支付的时候调用addPayment方法，在paymentQueue:updatedTransactions:回调中获取支付的状态，支付成功、支付失败和重复购买的状态下要结束订单finishTransaction。我当时是支付失败的情况，没有结束订单，然后下次支付成功的时候，获取的支付凭证是上次失败的凭证，然后不仅校验会失败，订单也是重复的。

<br/>

[**内购全面实战**](https://www.jianshu.com/p/2f98b7937b6f)





<br/>

> <h3 id ="适配器响应元是什么">适配器、响应元是什么？</h3>

- **[适配器](https://www.jianshu.com/p/61ade7936c93)**
	- [**适配器Demo**](https://github.com/idage/iOSAdapterDemo)
	- [**适配器模式实战**](https://www.jianshu.com/p/5a45fe49f8b6)




**定义：** 将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以在一起工作。

&emsp; 适配器其实就是在视图控制器和子视图之间的一个媒介，这个媒介其实用来解耦两者之间的耦合度的。
当然这个子视图控件是被封装好的一个公有视图类，比如一个特有风格的按钮、tabelViewCell等，虽然这个控件只是一个展示类，并没有什么交互。但是在项目好几个地方都用到了这个控件了，你在给这个视图控件负值的时候是怎么做的呢？

这个时候就需要建立一个适配器了。

适配器使用场景：
- 将一个类的接口转换成客户希望的另外一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作；
- 对于一些第三方的统计分析、错误收集等SDK应该都不陌生。就目前而言市面上也有许多相同功能的产品，眼花缭乱，让人无法抉择选哪一款SDK才是最靠谱的；
- 已经存在的类的接口不符合我们的需求；
- 创建一个可以复用的类，使得该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作；
- 在不对每一个都进行子类化以匹配它们的接口的情况下，使用一些已经存在的子类。

**优点：**解耦合，让视图类不合数据类产生耦合，使视图类更加独立。 新增加数据类的时候不需要修改视图类；
**缺点：**会新增加很多类，使系统更凌乱，代码可读性更弱了。



<br/>

- **响应元**



<br/>
<br/>

> <h2 id="热更新">[热更新](https://juejin.cn/post/6844904144411574285)</h2>




<br/>
<br/>

> <h2 id="代码管理">代码管理</h2>

<br/>

> <h3 id="pod instal和pod update区别">**pod install**和**pod update**区别
</h3>

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd4.jpg)

- **pod install:**执行该命令时，如果Podfile.lock文件存在, 则直接从此文件中读取框架信息并且它会只下载Podfile.lock文件中指定的版本安装。对于不在Podfile.lock文件中的pod库，pod install命令会搜索这个pod库在Podfile文件中指定的版本来安装；如果Podfile.lock不存在, 则会读取Podfile文件内的框架信息，然后执行下载并且根据下载好的框架信息, 生成Podfile.lock文件。

- **pod update:**只有当你想要更新pod库的版本时才使用pod update；它不管Podfile.lock是否存在, 都会读取Podfile文件的的框架信息去下载安装，下载好之后, 再根据下载好的框架信息, 生成Podfile.lock文件

当多人合作开发项目时，最开始把项目下载下来时最好都执行**`pod install`**，以后为了保持类库版本一致统一用**`pod update`**命令。



<br/>









<br/>

***
<br/>




> <h1 id = "Swift基础">Swift基础</h1>


<br/>

> <h2 id = "面向协议编程">面向协议编程</h2>

- **OOP**：面向对象编程（英文Object Oriented Programming）；
- **POP**：面向协议编程（Protocol Oriented Programming），是Swift的一种编程范式；


&emsp; 在Swift的协议中定义属性永远不要用 `let` 关键字。只读属性规定使用 `var` 关键字，并在后面单独跟上 `{ get }`。如果有一个方法改变了一个或多个属性，你需要标记它为 `mutating`。


<br/>


在[面向协议编程与 Cocoa 的邂逅 (上)](https://onevcat.com/2016/11/pop-cocoa-1/)的文章里POP解决了面向对象编程的**OC动态派发安全性、横切关注点、菱形缺陷**问题。

协议扩展：
对于 P，可以在 extension P 中为 myMethod 添加一个实现：


```
protocol P {
    func myMethod()
}

extension P {
    func myMethod() {
        doWork()
    }
}
```


有了这个协议扩展后，我们只需要简单地声明 ViewController 和 AnotherViewController 遵守 P，就可以直接使用 myMethod 的实现了：

```
extension ViewController: P { }
extension AnotherViewController: P { }

viewController.myMethod()
anotherViewController.myMethod()
```
不仅如此，除了已经定义过的方法，我们甚至可以在扩展中添加协议里没有定义过的方法。在这些额外的方法中，我们可以依赖协议定义过的方法进行操作。我们之后会看到更多的例子。总结下来：

- **协议定义**
	- 提供实现的入口
	- 遵循协议的类型需要对其进行实现
- **协议扩展**
	- 为入口提供默认实现
	- 根据入口提供额外实现


这样一来，横切点关注的问题也简单安全地得到了解决。
- ✅ 动态派发安全性
- ✅ 横切关注点
- 菱形缺陷


<br/>

在[**面向协议编程与 Cocoa 的邂逅 (下)**](https://onevcat.com/2016/12/pop-cocoa-2/)中我们把POP运用到实际的项目中去，比如：网络的封装。很赞的，可以把其运用到项目中去，重构项目中的网络层。


<br/>

***
<br/>





># <h1 id = "性能优化">性能优化</h1>




<br/>
<br/>

> <h3 id = "循环引用解决">循环引用解决</h1>

[♻️解决循环引用框架：FBRetainCycleDetector](https://draveness.me/retain-cycle1/)








<br/>
<br/>

> <h3 id = "NSTimer循环引用解决">NSTimer循环引用解决？</h1>

&emsp; 解决NSTimer循环引用的更佳方案:NSProxy直接消息转发，不会像继承于NSObject的对象去父类里面搜索，降低效率



<br/>

> <h3 id = "NSTimer计时不准确怎么办">NSTimer计时不准确怎么办？</h1>


&emsp; 不准确是因为当线程切换到主线程时会阻碍它计时，当用的是NSRunloopCommentModels时，可以用GCD来计时。

&emsp; GCD的定时器不依赖于runloop，而是和内核挂钩的，会比较准时，定时器的接口设计

接口定义：

```
+ (NSString *)execTask:(void(^)(void))task
           start:(NSTimeInterval)start
        interval:(NSTimeInterval)interval
         repeats:(BOOL)repeats
           async:(BOOL)async;

+ (NSString *)execTask:(id)target
              selector:(SEL)selector
                 start:(NSTimeInterval)start
              interval:(NSTimeInterval)interval
               repeats:(BOOL)repeats
                 async:(BOOL)async;

+ (void)cancelTask:(NSString *)name;

```


<br/>

接口实现

```

static NSMutableDictionary *timers_;
dispatch_semaphore_t semaphore_;
+ (void)initialize
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        timers_ = [NSMutableDictionary dictionary];
        semaphore_ = dispatch_semaphore_create(1);
    });
}

+ (NSString *)execTask:(void (^)(void))task start:(NSTimeInterval)start interval:(NSTimeInterval)interval repeats:(BOOL)repeats async:(BOOL)async
{
    if (!task || start < 0 || (interval <= 0 && repeats)) return nil;
    
    // 队列
    dispatch_queue_t queue = async ? dispatch_get_global_queue(0, 0) : dispatch_get_main_queue();
    
    // 创建定时器
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    
    // 设置时间
    dispatch_source_set_timer(timer,
                              dispatch_time(DISPATCH_TIME_NOW, start * NSEC_PER_SEC),
                              interval * NSEC_PER_SEC, 0);
    
    
    dispatch_semaphore_wait(semaphore_, DISPATCH_TIME_FOREVER);
    // 定时器的唯一标识
    NSString *name = [NSString stringWithFormat:@"%zd", timers_.count];
    // 存放到字典中
    timers_[name] = timer;
    dispatch_semaphore_signal(semaphore_);
    
    // 设置回调
    dispatch_source_set_event_handler(timer, ^{
        task();
        
        if (!repeats) { // 不重复的任务
            [self cancelTask:name];
        }
    });
    
    // 启动定时器
    dispatch_resume(timer);
    
    return name;
}

+ (NSString *)execTask:(id)target selector:(SEL)selector start:(NSTimeInterval)start interval:(NSTimeInterval)interval repeats:(BOOL)repeats async:(BOOL)async
{
    if (!target || !selector) return nil;
    
    return [self execTask:^{
        if ([target respondsToSelector:selector]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            [target performSelector:selector];
#pragma clang diagnostic pop
        }
    } start:start interval:interval repeats:repeats async:async];
}

+ (void)cancelTask:(NSString *)name
{
    if (name.length == 0) return;
    
    dispatch_semaphore_wait(semaphore_, DISPATCH_TIME_FOREVER);
    
    dispatch_source_t timer = timers_[name];
    if (timer) {
        dispatch_source_cancel(timer);
        [timers_ removeObjectForKey:name];
    }

    dispatch_semaphore_signal(semaphore_);
}

```



<br/>
<br/>

> <h2 id="图片优化">图片优化</h2>

<br/>

**图片性能瓶颈：**

大部分格式的图片，都需要被首先解码为bitmap，然后才能渲染到UI上。

UIImageView 显示图片，也有类似的过程。实际上，一张图片从在文件系统中，到被显示到 UIImageView，会经历以下几个步骤：

- 分配内存缓冲区和其它资源。
- 从磁盘拷贝数据到内核缓冲区
- 从内核缓冲区复制数据到用户空间
- 生成UIImageView，把图像数据赋值给UIImageView
- 将压缩的图片数据，解码为位图数据（bitmap），如果数据没有字节对齐，Core Animation会再拷贝一份数据，进行字节对齐。
- CATransaction捕获到UIImageView layer树的变化，主线程Runloop提交CATransaction，开始进行图像渲染
- GPU处理位图数据，进行渲染。

由于 UIKit 的封装性，这些细节不会直接对开发者展示。实际上，当我们调用`[UIImage imageNamed:@"xxx"]`后，UIImage 中存储的是未解码的图片，而调用 `[UIImageView setImage:image]`后，会在主线程进行图片的解码工作并且将图片显示到 UI 上，这时候，UIImage 中存储的是解码后的 bitmap 数据。

而图片的解压缩是一个非常消耗 CPU 资源的工作，如果我们有大量的图片需要展示到列表中，将会大大拖慢系统的响应速度，降低运行帧率。这就是 UIImageView 的一个性能瓶颈。

<br/>
<br/>

> <h2 id = "图片的处理">图片的处理</h2>

<br/>

- <h3 id="图片异步绘制">图片异步绘制</h3>

```
- (void)image
{
    UIImageView *imageView = [[UIImageView alloc] init];
    imageView.frame = CGRectMake(100, 100, 100, 56);
    [self.view addSubview:imageView];
    self.imageView = imageView;
    
	//在子线程中对图片原始数据进行强制解码，再将解码后的图片抛回主线程继续使用，从而提高主线程的响应速度。
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 获取CGImage
        CGImageRef cgImage = [UIImage imageNamed:@"timg"].CGImage;

		//+colorSpaceForImageRef: 方法来获取原始图片的颜色空间参数
		//CGColorSpaceRef colorspaceRef = [UIImage colorSpaceForImageRef:imageRef];
        // alphaInfo
        CGImageAlphaInfo alphaInfo = CGImageGetAlphaInfo(cgImage) & kCGBitmapAlphaInfoMask;
        BOOL hasAlpha = NO;
        if (alphaInfo == kCGImageAlphaPremultipliedLast ||
            alphaInfo == kCGImageAlphaPremultipliedFirst ||
            alphaInfo == kCGImageAlphaLast ||
            alphaInfo == kCGImageAlphaFirst) {
            hasAlpha = YES;
        }

        // bitmapInfo
        CGBitmapInfo bitmapInfo = kCGBitmapByteOrder32Host;
        bitmapInfo |= hasAlpha ? kCGImageAlphaPremultipliedFirst : kCGImageAlphaNoneSkipFirst;

        // size
        size_t width = CGImageGetWidth(cgImage);
        size_t height = CGImageGetHeight(cgImage);

        // context
        //CGBitmapContextCreate() 方法，生成一个空白的图片绘制上下文，我们传入了上述的一些参数，指定了图片的大小、颜色空间、像素排列等等属性
        CGContextRef context = CGBitmapContextCreate(NULL, width, height, 8, 0, CGColorSpaceCreateDeviceRGB(), bitmapInfo);

        // draw
        //CGContextDrawImage() 方法，将未解码的 imageRef 指针内容，写入到我们创建的上下文中，这个步骤，完成了隐式的解码工作。
        CGContextDrawImage(context, CGRectMake(0, 0, width, height), cgImage);

        // get CGImage
        //从 context 上下文中创建一个新的 imageRef，这是解码后的图片了
        cgImage = CGBitmapContextCreateImage(context);

        // into UIImage
        /*
        从 imageRef 生成供UI层使用的 UIImage 对象，同时指定图片的 scale 和orientation 两个参数。

				scale 指的是图片被渲染时需要被压缩的倍数，为什么会存在这个参数呢，因为苹果为了节省安装包体积，允许开发者为同一张图片上传不同分辨率的版本，也就是我们熟悉的@2x，@3x后缀图片。不同屏幕素质的设备，会获取到对应的资源。为了绘制图片时统一，这些图片会被set自己的scale属性，比如@2x图片，scale 值就是2，虽然和1x图片的绘制宽高一样，但是实际的长是width * scale。

				orientation 很好理解，就是图片的旋转属性，告诉设备，以哪个方向作为图片的默认方向来渲染。
        */
        //UIImage *newImage = [UIImage imageWithCGImage:newImageRef scale:image.scale orientation:image.imageOrientation];
	     UIImage *newImage = [UIImage imageWithCGImage:cgImage];

        // release
        CGContextRelease(context);
        CGImageRelease(cgImage);

        // back to the main thread
        dispatch_async(dispatch_get_main_queue(), ^{
            self.imageView.image = newImage;
        });
    });
}
```


<br/>

- <h3 id="图片格式判断">图片格式判断</h3>

根据图片不同压缩格式的原始二进制数据的编码特征，就可以进行简单的分类了，如下Code：

```
+ (XRImageFormat)imageFormatForImageData:(nullable NSData *)data {
    if (!data) {
        return XRImageFormatUndefined;
    }
    
    uint8_t c;
    [data getBytes:&c length:1];
    switch (c) {
        case 0xFF:
            return XRImageFormatJPEG;
        case 0x89:
            return XRImageFormatPNG;
        case 0x47:
            return XRImageFormatGIF;
        case 0x49:
        case 0x4D:
            return XRImageFormatTIFF;
        case 0x52:
            if (data.length < 12) {
                return XRImageFormatUndefined;
            }
            
            NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(0, 12)] encoding:NSASCIIStringEncoding];
            if ([testString hasPrefix:@"RIFF"] && [testString hasSuffix:@"WEBP"]) {
                return XRImageFormatWebP;
            }
    }
    return XRImageFormatUndefined;
}
```



<br/>
<br/>

> <h2 id="图片上传">图片上传</h2>

<br/>


- 上传图片时出现的问题？需要注意的事项

**事项一：**上传图片到服务器一般是将图片NSData上传到服务器，服务器返回一个图片NSString地址，之后再将NSString的路径转为url并通过url请求去更新图片（图片此时更新的便是NSString）

上传的图片一般压缩方式是：

```
NSData *eachImgData = UIImageJPEGRepresentation(image, 0.5);
```

但是你可能会发现这样设置之后，上传还是比较慢，即使是把压缩比例设置为0.1 结果还是一样。你也许会看到图片大小大概为200-300KB  为何这么小了  还是上传比较慢，或者加载比较慢，这是因为你忽略了一个图片分辨率的压缩过程，所以此时你要压缩的不只是图片大小，还是图片分辨率。图片分辨率压缩代码：

```
//等比例压缩

+ (UIImage *) imageCompressForSize:(UIImage *)sourceImage targetSize:(CGSize)size {

    UIImage *newImage = nil;
    CGSize imageSize = sourceImage.size;
    CGFloat width = imageSize.width;
    CGFloat height = imageSize.height;

    CGFloat targetWidth = size.width;
    CGFloat targetHeight = size.height;
    CGFloat scaleFactor = 0.0;

    CGFloat scaledWidth = targetWidth;
    CGFloat scaledHeight = targetHeight;

    CGPoint thumbnailPoint = CGPointMake(0.0, 0.0);

	//Size是否相等
    if(CGSizeEqualToSize(imageSize, size) == NO){
		//宽度因子
        CGFloat widthFactor = targetWidth / width;
        CGFloat heightFactor = targetHeight / height;

        if(widthFactor > heightFactor){
            scaleFactor = widthFactor;
        }
        else{
            scaleFactor = heightFactor;
        }

		//缩放宽度
        scaledWidth = width * scaleFactor;
        scaledHeight = height * scaleFactor;

        if(widthFactor > heightFactor){
			//缩图坐标
            thumbnailPoint.y = (targetHeight - scaledHeight) * 0.5;
        }else if(widthFactor < heightFactor){
            thumbnailPoint.x = (targetWidth - scaledWidth) * 0.5;
        }

    }

   
    UIGraphicsBeginImageContext(size);
 
	  CGRect thumbnailRect = CGRectZero;
    thumbnailRect.origin = thumbnailPoint;
    thumbnailRect.size.width = scaledWidth;
    thumbnailRect.size.height = scaledHeight;
    
	[sourceImage drawInRect:thumbnailRect];

    newImage = UIGraphicsGetImageFromCurrentImageContext();

    if(newImage == nil){
        NSLog(@"scale image fail");
    }

    UIGraphicsEndImageContext();

    return newImage;
}
```
这样双重压缩，完美解决图片压缩问题。


<br/>

NSData的length是bytes格式需要进行除以1024进行单位转换,格式判断Code：

```
- (BOOL)imgeData:(NSData *)data andImage:(UIImage *)image{
    
    double dataLength = [data length] * 1.0;
    
    NSArray *typeArray = @[@"bytes",@"KB",@"MB",@"GB",@"TB",@"PB", @"EB",@"ZB",@"YB"];
        NSInteger index = 0;
        while (dataLength > 1024) {
            dataLength /= 1024.0;
            index ++;
        }
    NSString *str = [NSString stringWithFormat:@"%.3f %@\n",dataLength,typeArray[index]];
   
    if(index >=2){
        //大于1M
        return YES;
    }
    
    if(index ==1 && dataLength > 850){
        //大于850KB
        return YES;
    }
    
    NSLog(@"str=== 无压缩====%@",str);
    
    return NO;
}

```

对于非常大的图片和视频请看[网络优化(I)](https://github.com/harleyGit/StudyNotes/blob/master/Optimization/网络优化(I).md)中的**上传优化**。



<br/>
<br/>

> <h2 id ="界面保持流畅">[界面保持流畅](https://xilankong.github.io/ios性能优化/2017/10/29/iOS如何保持界面流畅.html)</h2>

<br/>

文本绘制

> [TextKit 最佳实践](https://juejin.cn/post/6844903621553831944)

> [CoreText使用说明书](https://www.jianshu.com/p/1c3d0936bba6)


图片
> [图片解码](https://www.cnblogs.com/dins/p/ios-tu-pian.html)

Alpha（不透明度）：
 - 属性为浮点类型的值，取值范围从0到1.0，表示从完全透明到完全不透明，其特性有当前UIView的alpha值会被其所有subview继承。
 - alpha值会影响到UIView跟其所有subview，alpha具有动画效果。
 - 当alpha为0时，跟hidden为YES时效果一样，但是alpha主要用于实现隐藏的动画效果，在动画块中将hidden设置为YES没有动画效果。


&emsp; 事实上，解压缩后的图片大小与原始文件大小之间没有任何关系，而只与图片的像素有关：
`解压缩后的图片大小(3600) = 图片的像素宽(30) * 图片的像素高(30) * 每个像素所占的字节数(4)`


&emsp; 当未解压缩的图片将要渲染到屏幕时，系统会在主线程对图片进行解压缩，而如果图片已经解压缩了，系统就不会再对图片进行解压缩。因此，也就有了业内的解决方案，在子线程提前对图片进行强制解压缩。

&emsp; 而强制解压缩的原理就是对图片进行重新绘制，得到一张新的解压缩后的位图。其中，用到的最核心的函数是 

```
/*

data ：如果不为 NULL ，那么它应该指向一块大小至少为 bytesPerRow * height 字节的内存；如果 为 NULL ，那么系统就会为我们自动分配和释放所需的内存，所以一般指定 NULL 即可；

width 和 height ：位图的宽度和高度，分别赋值为图片的像素宽度和像素高度即可；

bitsPerComponent ：像素的每个颜色分量使用的 bit 数，在 RGB 颜色空间下指定 8 即可；

bytesPerRow ：位图的每一行使用的字节数，大小至少为 width * bytes per pixel 字节。有意思的是，当我们指定 0 时，系统不仅会为我们自动计算，而且还会进行 cache line alignment 的优化，更多信息可以查看 what is byte alignment (cache line alignment) for Core Animation? Why it matters? 和 Why is my image’s Bytes per Row more than its Bytes per Pixel times its Width?

space ：就是我们前面提到的颜色空间，一般使用 RGB 即可；

bitmapInfo ：就是我们前面提到的位图的布局信息。
到这里，你已经掌握了强制解压缩图片需要用到的最核心的函数



*/



CG_EXTERN CGContextRef __nullable CGBitmapContextCreate(void * __nullable data,
    size_t width, size_t height, size_t bitsPerComponent, size_t bytesPerRow,
    CGColorSpaceRef cg_nullable space, uint32_t bitmapInfo)
    CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
```

<br/>

- Pixel Format（像素格式）

位图其实就是一个像素数组，而像素格式则是用来描述每个像素的组成格式，它包括以下信息：

	- Bits per component：一个像素中每个独立的颜色分量使用的 bit 数；
	- Bits per pixel：一个像素使用的总 bit 数；
	- Bytes per row：位图中的每一行使用的字节数。
	

<br/>

- Color and Color Spaces（颜色空间）

在 Quartz 中，一个颜色是由一组值来表示的，比如 (0, 0, 1)。而颜色空间则是用来说明如何解析这些值的，离开了颜色空间，它们将变得毫无意义。比如,下面的值都表示蓝色：

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc0.png)


<br/>

- Color Spaces and Bitmap Layout（颜色空间和位图格式）

像素格式是用来描述每个像素的组成格式的，比如每个像素使用的总 bit 数。而要想确保 Quartz 能够正确地解析这些 bit 所代表的含义，我们还需要提供位图的布局信息 CGBitmapInfo：


```

typedef CF_OPTIONS(uint32_t, CGBitmapInfo) {
    kCGBitmapAlphaInfoMask = 0x1F,

    kCGBitmapFloatInfoMask = 0xF00,
    kCGBitmapFloatComponents = (1 << 8),

    kCGBitmapByteOrderMask     = kCGImageByteOrderMask,
    kCGBitmapByteOrderDefault  = (0 << 12),
    kCGBitmapByteOrder16Little = kCGImageByteOrder16Little,
    kCGBitmapByteOrder32Little = kCGImageByteOrder32Little,
    kCGBitmapByteOrder16Big    = kCGImageByteOrder16Big,
    kCGBitmapByteOrder32Big    = kCGImageByteOrder32Big
} CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

```





<br/>
<br/>


> <h2 id ="启动优化">[启动优化](https://juejin.cn/post/6844904127068110862#heading-3)</h2>

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc3.png)


- **冷启动流程**

&emsp; Apple 官方的《WWDC Optimizing App Startup Time》 将 iOS 应用的启动可分为 pre-main 阶段和 main 两个阶段，最佳的启动速度是400ms以内，最慢不得大于20s，否则会被系统进程杀死（最低配置设备）。

&emsp; 为了更好的区分，笔者将整个启动流程分为三个阶段， **App总启动流程 = pre-main + main函数代理（didFinishLaunchingWithOptions）+ 首屏渲染（viewDidAppear），后两个阶段都属于 main函数  执行阶段**。




<br/>

[**高德启动耗时优化**](https://cloud.tencent.com/developer/news/616444)


<br/>
<br/>


> <h2 id = "UI卡顿优化">UI卡顿优化</h2>

卡顿的优化，哪几个方面？从哪可以检测到？





<br/>
<br/>


> <h2 id = "内存优化">[**内存优化**](https://juejin.cn/post/6864492188404088846)</h2>




<br/>
<br/>




> <h2 id = "包体积优化">[包体积优化](https://juejin.cn/post/6844904169938092045)</h2>

<br/>

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc1.png)


- **资源瘦身**
	
	<br/>
	
- 移除无用图片资源

&emsp； 这里推荐使用工具 [LSUnusedResources](https://github.com/tinymind/LSUnusedResources)，可以根据项目实际情况定义查找文件的正则表达式。另外建议勾选 Ignore similar name ，避免扫描出图片组。


![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc2.png)


<br/>

- 压缩图片等资源文件


&emsp; 建议使用无损压缩，例如 ImageOptim、pngquant命令、tinypng ，如果涉及有损压缩最好要求设计介入进行资源检查。


&emsp; 还可以使用 Webp 格式的图片，Webp 是由 Google 推出的图片格式，有损压缩模式下图片体积只有 jpeg 格式的 1/3，无损压缩也能减小 1/4，可以使用 cwebp 进行格式压缩转换，目前 SDWebImage、Kingfisher 都用支持该格式解析的拓展。无损压缩命令如下：

```
// 语法
cwebp [options] input_file -o output_file.webp
// 无损压缩
cwebp -lossless original.png -o new.webp
复制代码
除了终端命令，还可以使用 iSparta 进行批量转换格式。
```


<br/>

- Xcode 本身也提供压缩图片的编译选项
	- Compress PNG Files
打包的时候基于 pngcrush 工具自动对图片进行无损压缩，如果我们已自行对图片进行压缩，该选项最好关闭。
	
	- Remove Text Medadata From PNG Files
移除 PNG 资源的文本字符，比如图像名称、作者、版权、创作时间、注释等信息。

***


<br/>

- **删除重复文件**

&emsp; 通过校验所有资源的 MD5，筛选出项目中的重复资源，推荐使用 fdupes 工具进行重复文件扫描，[fdupes ](https://github.com/adrianlopezroche/fdupes)是 Linux 平台的一个开源工具，由 C 语言编写 ，文件比较顺序是`大小对比 > 部分 MD5 签名对比 > 完整 MD5 签名对比 > 逐字节对比`。

通过 `Homebrew 安装 fdupes：`

```
brew install fdupes

```

查看目标文件夹下的重复文件：

```
fdupes -Sr 文件夹   // 查看文件夹下所有子目录中的重复文件及大小
fdupes -Sr 文件夹 > 输出地址.txt  // 将信息输出到txt文件中
```

输出内容如下，一般情况下，相同文件仅保留一份，修改对应的引用即可。

```
4474 bytes each:
Test/Images.xcassets/TabBarImage/tabBar_2.imageset/tabBar_2@2x.png
Test/Resource/TabBarImage/tabBar_2@2x.png

3912 bytes each:
Test/Images.xcassets/TabBarImage/tabBar_3.imageset/tabBar_3@2x.png
Test/Resource/TabBarImage/tabBar_3@2x.png
```





<br/>
<br/>


> <h2 id ="内存暴涨解决">内存暴涨解决</h2>

&emsp; 在列表滑动的情况内存莫名的增长，频繁访问图片的时候内存莫名的增长，频繁的打开和关闭数据库的时候内存莫名的增长……。 这些都是拜iOS的autorelease机制所赐；具体分析如下：

1.滑动列表的时候，内存出现莫名的增长，原因可能有如下可能：

	a. 没有使用UITableView的reuse机制； 导致每显示一个cell都用autorelease的方式重新alloc一次；导致cell的内存不断的增加；

	b. 每个cell会显示一个单独的UIView， 在UIView发生内存泄漏，导致cell的内存不断增长；


<br/>

2.频繁访问图片的时候，内存莫名的增长；

&emsp; 频繁的访问网络图片，导致iOS内部API，会不断的分配autorelease方式的buffer来处理图片的解码与显示； 利用图片NSCache可以缓解一下此问题；

**NSCache使用：**

```
// 创建对象
NSCache *cache = [[NSCache alloc] init];
// 设置缓存数量限制，默认值是 0，表示没有限制
cache.countLimit = 10;
// 设置缓存总成本限制，默认值是 0，表示没有限制
cache.totalCostLimit = 1024 * 1024;
// 设置是否自动清理缓存，默认为 YES，表示自动清理
cache.evictsObjectsWithDiscardedContent = YES;
// 设置代理
cache.delegate = self;
// 设置缓存
/*
	0 成本，与可变字典不同，缓存对象不会对键名做 copy 操作，只是做强引用
*/
[cache setObject:str forKey:@(i)];
// 设置缓存
/*
	指定成本
*/
[cache setObject:str forKey:@(i) cost:1024];
// 查看缓存内容
/*
	NSCache 没有提供遍历的方法，只支持用 key 来取值，NSCache 的 Key 只是做强引用，不需要实现 NSCopying 协议
*/
NSString *string = [cache objectForKey:@(i)];
// 删除指定缓存
[cache removeObjectForKey:@8];
// 删除所有缓存
/*
	一旦调用了 removeAllObjects，就无法给 cache 添加对象，关于 NSCache 的内存管理，交给他自己就行
*/
[cache removeAllObjects];
// 缓存协议方法
/*
	须遵守 <NSCacheDelegate> 协议，obj 就是要被清理的对象
	当缓存中的对象被清除的时候，会自动调用，不建议平时开发时重写！仅供调试使用
*/
- (void)cache:(NSCache *)cache willEvictObject:(id)obj {
}
	
```

 
<br/>


3.频繁打开和关闭SQLite，导致内存不断的增长；

&emsp; 在进行SQLite频繁打开和关闭操作，而且读写的数据buffer较大，那么SQLite在每次打开与关闭的时候，都会利用autorelease的方式分配51K的内存； 如果访问次数很多，内存马上就会顶到几十兆，甚至上百兆！ 所以针对频繁的读写数据库且数据buffer较大的情况， 可以设置SQLite的长连接方式；避免频繁的打开和关闭数据库；

**SQLite操作封装-长连接方式**

```
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SQLite;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace Acon.UrineAnalyzerPlatform.DataAccess
{
    /// <summary>
    /// SQLite数据库操作类，长连接模式
    /// </summary>
    public sealed class SQLiteHelper
    {
        public static string ConnectionString { get; set; }
 
        private static SQLiteConnection Connection;
 
        static SQLiteHelper()
        {
            string fileName = Path.Combine(Application.StartupPath, @"Data\\UrineMachine.db");
            ConnectionString = string.Format("Data Source={0};{1}", fileName, ";foreign keys=true;");
        }
 
        #region 静态方法
 
        private static SQLiteConnection CreateConnection()
        {
            if (Connection == null)
                Connection = new SQLiteConnection(ConnectionString);
            
            return Connection;
        }
 
        public static SQLiteTransaction GetTransaction()
        {
            return CreateConnection().BeginTransaction();
        }
 
        private static void PrepareCommand(SQLiteCommand command, SQLiteConnection connection, SQLiteTransaction transaction, CommandType commandType, string commandText, SQLiteParameter[] parms)
        {
            if (connection.State != ConnectionState.Open) connection.Open();
 
            command.Connection = connection;
            //command.CommandTimeout = CommandTimeOut;
            // 设置命令文本(存储过程名或SQL语句)
            command.CommandText = commandText;
            // 分配事务
            if (transaction != null)
            {
                command.Transaction = transaction;
            }
            // 设置命令类型.
            command.CommandType = commandType;
            if (parms != null && parms.Length > 0)
            {
                //预处理SqlParameter参数数组，将为NULL的参数赋值为DBNull.Value;
                foreach (SQLiteParameter parameter in parms)
                {
                    if ((parameter.Direction == ParameterDirection.InputOutput || parameter.Direction == ParameterDirection.Input) && (parameter.Value == null))
                    {
                        parameter.Value = DBNull.Value;
                    }
                }
                command.Parameters.AddRange(parms);
            }
        }
 
        #region ExecuteNonQuery
        public static int ExecuteNonQuery(string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteNonQuery(CreateConnection(), commandText, parms);
        }
   
        public static int ExecuteNonQuery(SQLiteConnection connection, string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteNonQuery(connection, CommandType.Text, commandText, parms);
        }
 
        /// <summary>
        /// 执行SQL语句,返回影响的行数
        /// </summary>
        /// <param name="connection">数据库连接</param>
        /// <param name="commandType">命令类型(存储过程,命令文本, 其它.)</param>
        /// <param name="commandText">SQL语句或存储过程名称</param>
        /// <param name="parms">查询参数</param>
        /// <returns>返回影响的行数</returns>
        public static int ExecuteNonQuery(SQLiteConnection connection, CommandType commandType, string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteNonQuery(connection, null, commandType, commandText, parms);
            //try
            //{
            //    return ExecuteNonQuery(connection, null, commandType, commandText, parms);
            //}
            //finally
            //{
            //    connection.Close();
            //}
        }
 
        /// <summary>
        /// 执行SQL语句,返回影响的行数
        /// </summary>
        /// <param name="transaction">事务</param>
        /// <param name="commandType">命令类型(存储过程,命令文本, 其它.)</param>
        /// <param name="commandText">SQL语句或存储过程名称</param>
        /// <param name="parms">查询参数</param>
        /// <returns>返回影响的行数</returns>
        public static int ExecuteNonQuery(SQLiteTransaction transaction, CommandType commandType, string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteNonQuery(transaction.Connection, transaction, commandType, commandText, parms);
 
        }
 
        /// <summary>
        /// 执行SQL语句,返回影响的行数
        /// </summary>
        /// <param name="connection">数据库连接</param>
        /// <param name="transaction">事务</param>
        /// <param name="commandType">命令类型(存储过程,命令文本, 其它.)</param>
        /// <param name="commandText">SQL语句或存储过程名称</param>
        /// <param name="parms">查询参数</param>
        /// <returns>返回影响的行数</returns>
        private static int ExecuteNonQuery(SQLiteConnection connection, SQLiteTransaction transaction, CommandType commandType, string commandText, params SQLiteParameter[] parms)
        {
            SQLiteCommand command = new SQLiteCommand();
            PrepareCommand(command, connection, transaction, commandType, commandText, parms);
            int retval = command.ExecuteNonQuery();
            command.Parameters.Clear();
            return retval;
        }
 
        #endregion ExecuteNonQuery
 
        #region ExecuteScalar
        /// <summary>
        /// 执行SQL语句,返回结果集中的第一行第一列
        /// </summary>
        /// <param name="connectionString">数据库连接字符串</param>
        /// <param name="commandText">SQL语句</param>
        /// <param name="parms">查询参数</param>
        /// <returns>返回结果集中的第一行第一列</returns>
        public static object ExecuteScalar(string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteScalar(CreateConnection(), commandText, parms);
        }
 
        public static object ExecuteScalar(SQLiteConnection connection, string commandText, SQLiteParameter[] parms)
        {
            return ExecuteScalar(connection, CommandType.Text, commandText, parms);
        }
 
        public static object ExecuteScalar(SQLiteConnection connection, CommandType commandType, string commandText, SQLiteParameter[] parms)
        {
            SQLiteCommand command = new SQLiteCommand();
            PrepareCommand(command, connection, null, commandType, commandText, parms);
            object retval = command.ExecuteScalar();
            command.Parameters.Clear();
            return retval;
        }
 
 
        #endregion
 
        #region ExecuteDataSet
        public static DataSet ExecuteDataSet(string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteDataSet(CreateConnection(), commandText, parms);
        }
 
        public static DataSet ExecuteDataSet(SQLiteConnection connection, string commandText, SQLiteParameter[] parms)
        {
            return ExecuteDataSet(connection, CommandType.Text, commandText, parms);
        }
 
        public static DataSet ExecuteDataSet(SQLiteConnection connection, CommandType commandType, string commandText, SQLiteParameter[] parms)
        {
            return ExecuteDataSet(connection, null, commandType, commandText, parms);
        }
 
        /// <summary>
        /// 执行SQL语句,返回结果集
        /// </summary>
        /// <param name="transaction">事务</param>
        /// <param name="commandType">命令类型(存储过程,命令文本, 其它.)</param>
        /// <param name="commandText">SQL语句或存储过程名称</param>
        /// <param name="parms">查询参数</param>
        /// <returns>返回结果集</returns>
        public static DataSet ExecuteDataSet(SQLiteTransaction transaction, CommandType commandType, string commandText, params SQLiteParameter[] parms)
        {
            return ExecuteDataSet(transaction.Connection, transaction, commandType, commandText, parms);
        }
 
        private static DataSet ExecuteDataSet(SQLiteConnection connection, SQLiteTransaction transaction, CommandType commandType, string commandText, SQLiteParameter[] parms)
        {
            SQLiteCommand command = new SQLiteCommand();
            PrepareCommand(command, connection, transaction, commandType, commandText, parms);
            SQLiteDataAdapter adapter = new SQLiteDataAdapter(command);
            DataSet ds = new DataSet();
            adapter.Fill(ds);
            command.Parameters.Clear();
            return ds;
        }
        #endregion
 
        #endregion 静态方法
 
        public static void Close()
        {
            if (Connection != null && Connection.State == ConnectionState.Open)
            {
                Connection.Close();
                System.Data.SQLite.SQLiteConnection.ClearPool(Connection);//清除连接池，否则数据库文件不会被释放。
            }
        }
    }
}


```



<br/>
<br/>

> <h2 id = "文件存储优化">文件存储优化</h2>

<br/>

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc3.png)


- Documents：保存应用运行时生成的需要持久化的数据,iTunes会自动备份该目录。苹果建议将在应用程序中浏览到的文件数据保存在该目录下。

- tmp：临时文件目录，在程序重新运行的时候，和开机的时候，会清空tmp文件夹。

- Library:
	- Caches：一般存储的是缓存文件，例如图片视频等，此目录下的文件不会再应用程序退出时删除。在手机备份的时候，iTunes不会备份该目录。例如:音频,视频等文件存放其中
	- Preferences：保存应用程序的所有偏好设置iOS的Settings(设置)，我们不应该直接在这里创建文件，而是需要通过NSUserDefault这个类来访问应用程序的偏好设置。iTunes会自动备份该文件目录下的内容。比如说:是否允许访问图片,是否允许访问地理位置......






<br/>

***
<br/>




># <h1 id = "架构与设计模式">架构与设计模式</h1>

<br/>

- [开发架构设计思考](https://www.jianshu.com/p/1e1009ba061c)
- [组件化开发实施](https://www.jianshu.com/p/599e97b63af7)
- [组件优化](https://casatwy.com/iOS-Modulization.html?hmsr=toutiao.io)










<br/>

***
<br/>


># <h1 id = "底层">底层</h1>

<br/>

> <h2 id ="runloop与自动释放池关系什么时侯释放">RunLoop与自动释放池关系，什么时侯释放?</h2>


- **分两种情况：手动干预释放和系统自动释放**
	- 手动干预释放就是指定autoreleasepool,当前作用域大括号结束就立即释放
	- 系统自动去释放:不手动指定autoreleasepool,Autorelease对象会在当前的 runloop 迭代结束时释放
	- kCFRunLoopEntry(1):第一次进入会自动创建一个autorelease
	- kCFRunLoopBeforeWaiting(32):进入休眠状态前会自动销毁一个autorelease,然后重新创建一个新的autorelease
	- kCFRunLoopExit(128):退出runloop时会自动销毁最后一个创建的autorelease

<br/>

- **系统自动释放时机：**

&emsp; 每个runloop中都创建一个Autorelease Pool，并在runloop的末尾进行释放。所以，一般情况下，每个接受autorelease消息的对象，都会在下个runloop开始前被释放。也就是说，在一段同步的代码中执行过程中，生成的对象接受autorelease消息后，一般是不会在作用域结束前释放的。Autorelease对象出了作用域之后，会被添加到最近一次创建的自动释放池中，并会在当前的 runloop 迭代结束（RunLoop 在第一次获取时创建, 在线程结束时销毁）时释放,至于何时runloop结束并没有固定的duration。

&emsp; Runloop是一个运行循环，但是它不可能一直运行着，所以它会有一个休眠期。Runloop维护着一个AutoreleasePool，当它进入休眠前它会把这个释放池里的对象进行释放。

&emsp; UIKit通过RunLoopObserver在RunLoop两次Sleep间对AutoreleasePool进行pop和push,将这次Loop中产生的Autorelease对象释放。


<br/>

- **在UIKit中Runloop什么时候进入休眠？**

先看一个简单Demo：

```
- (void)viewDidLoad{
    [super viewDidLoad];
    NSString *string = [NSString stringWithFormat:@"齐滇大圣"];
}

```

&emsp; 这里[NSString stringWithFormat:@”齐滇大圣”];创建对象时这个对象的引用计数为 1 。当使用局部变量 string 指向这个对象时，这个对象的引用计数 +1 ，变成了 2 。而当 viewDidLoad 方法返回时，局部变量 string 被回收，指向了 nil 。因此，其所指向对象的引用计数 -1 ，变成了 1 。

&emsp; 然后我们的这个对象是一个autorelease的实例，是被系统自动添加到了当前的 autoreleasepool 中的。所以会当Runloop一次迭代结束即将进入休眠的时候autoreleasepool drain对象引用计数 -1，对象释放。

下面我们配合着堆栈信息看一下这个Runloop什么时候结束一次迭代。

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd0.png)


&emsp; 这里我们加入一个weak的全局变量reference来指向我们的对象。因为weak引用不持有我们的对象，不会影响所指向对象的生命周期，所以我们用它来输出以判断我们的对象什么时候释放。

&emsp; 我们能看到reference在viewDidLoad和viewWillAppear的时候有输出，而在viewDidAppear的时候为null，说明被释放了。那我们来猜测一下runloop的迭代周期。

&emsp; viewWillAppear很容易理解是即将进入页面嘛，那runloop肯定是还有事要做的嘛，当viewDidAppear的时候表示已经进入页面了。那就表示没事做了，进入睡眠，等待用户动作的时候再次唤醒。你可能会觉得我口说无凭不靠谱，好那我就拿出证据来，我们来看下面两张图。

断点1和断点2:

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd1.png)

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_pd2.png)

我们能看到断点1和断点2 runloop还是在执行的，断点3表示runloop一个迭代已经结束了，即将进入睡眠。

```
//触发 Source0 (非基于port的) 回调。
__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__(source0);
//out_to_block表示从block跳出，block执行完毕，即将进入睡眠。
__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
```


<br/>

- **自动释放池使用时机：**
	- 写给予命令行的程序时，就是没有UI框架；
	- 写循环，循环里边包含了大量临时创建的对象；
	- 创建了新的线程；
	- 长时间在后台运行的任务；
	- 合理运用自动释放池，可以降低程序的内存峰值，异步的方式将文件保存在磁盘（SDWebimage里边异步保存图片到磁盘，类似的占用内存的操作）；


<br/>
<br/>


 > <h2 id ="RunLoop原理和与线程的联系">RunLoop原理和与线程的联系？说一下你对他的了解</h2>

1. RunLoop 的作用就是来管理线程的，当线程的 RunLoop开启后，线程就会在执行完任务后，处于休眠状态，随时等待接受新的任务，而不是退出。

2. 只有主线程的RunLoop是默认开启的，所以程序在开启后，会一直运行，不会退出。其他线程的RunLoop如果需要开启，就手动开启，


3. **runloop的mode作用:**

	- model 主要是用来指定事件在运行循环中的优先级的，分为：

	```
	* NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
	* UITrackingRunLoopMode                       ：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
	* UIInitializationRunLoopMode                  ：在刚启动 App 时进入的第一个 Mode ，启动完成后就不再使用
	* GSEventReceiveRunLoopMode                  ：接受系统事件的内部 Mode，通常用不到
  	* kCFRunLoopCommonModes                      ：这是一个占位的 Mode，没有实际作用
	```

	- 苹果公开提供的 Mode有两个：

	```
	NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
	NSRunLoopCommonModes（kCFRunLoopCommonModes）
	```



<br/>
<br/>

> <h2 id ="main函数之前会做什么">[main函数之前会做什么](https://juejin.cn/post/6844903783160348685#heading-11)</h2>

main()函数调用之前，其实是做了很多准备工作，主要是dyld这个动态链接器在负责，核心流程如下:

- 程序执行从_dyld_star开始
	- 读取macho文件信息，设置虚拟地址偏移量，用于重定向;
	- 调用dyld::_main方法进入macho文件的主程序;

<br/>

- 配置一些环境变量
	- 设置的环境变量方便我们打印出更多的信息。
	- 调用getHostInfo()来获取machO头部获取当前运行架构的信息

<br/>

- 实例化主程序,即macho可执行文件

<br/>

- 加载共享缓存库

<br/>


- 插入动态缓存库

<br/>

- 链接主程序

<br/>

- 初始化函数
	- 经过一系列的初始化函数最终调用notifSingle函数
	-  此回调是被运行时_objc_init初始化时赋值的一个函数load_image(后面的总结是：先类文件load,再分类文件load,然后再是C++构造函数，最后就进入了我们的main主程序)
	-  load_images里面执行call_load_methods函数，循环调用所用类以及分类的load方法
		-  call_load_methods方法调用,在call_load_methods中，通过doWhile循环来调用call_class_loads加载`每个类的load方法`,然后再加`载分类的loads方法`;
	- doModInitFunctions函数，内部会调用全局C++对象的构造函数，即_ _ attribute_ _((constructor))这样的函数
		- 在调用完notifySigin后，继续调用了doInitialization，doModInitFunctions会调用machO文件中_mod_init_func段的函数，也就是我们在文件中所定义的全局C++构造函数

	
<br/>


- 	返回主程序的入口函数，开始进入主程序的main（）函数





<br/>
<br/>

> <h2 id = "响应者链">响应者链</h2>

<br/>

- [响应者链（I）](https://github.com/harleyGit/StudyNotes/blob/master/iOS/Objective-C/响应链(I).md)




<br/>
<br/>



> <h2 id = "OC的引用计数是存放">OC的引用计数是存放在哪？</h2>

在64bit操作系统中,apple对对象中的isa进行了优化使用isa_t结构来保存关于对象的更多信息.

```
# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                      \
      uintptr_t nonpointer        : 1;   //指针是否优化过                                   \
      uintptr_t has_assoc         : 1;   //是否有设置过关联对象，如果没有，释放时会更快                                   \
      uintptr_t has_cxx_dtor      : 1; 	 //是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快                                     \
      uintptr_t shiftcls          : 33; //存储着Class、Meta-Class对象的内存地址信息 \
      uintptr_t magic             : 6;  //用于在调试时分辨对象是否未完成初始化                                     \
      uintptr_t weakly_referenced : 1;  //是否有被弱引用指向过，如果没有，释放时会更快                                     \
      uintptr_t deallocating      : 1;  //对象是否正在释放                                     \
      uintptr_t has_sidetable_rc  : 1;  //引用计数器是否过大无法存储在isa中                                     \
      uintptr_t extra_rc          : 19  //里面存储的值是引用计数器减1
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)
 
# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)
 
# else
#   error unknown architecture for packed isa
# endif


```

其中：extra_rc和has_sidetable_rc两个标志就是用来存储对象引用计数的.


isa中不同的位域代表不同的含义。


- nonpointer
	- 0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
	- 1，代表优化过，使用位域存储更多的信息



- has_assoc
	- 是否有设置过关联对象，如果没有，释放时会更快


- has_cxx_dtor
	- 是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快


- shiftcls
	- 存储着Class、Meta-Class对象的内存地址信息

- magic
	- 用于在调试时分辨对象是否未完成初始化


- weakly_referenced
	- 是否有被弱引用指向过，如果没有，释放时会更快


- deallocating
	- 对象是否正在释放


- extra_rc
	- 里面存储的值是引用计数器减1


- has_sidetable_rc
	- 引用计数器是否过大无法存储在isa中
	- 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中


<br/>

**引用计数的存储**

引用计数的增加主要通过retain()函数来实现.

```
// Equivalent to calling [this retain], with shortcuts if there is no override
inline id 
objc_object::retain()
{
    assert(!isTaggedPointer());
 
    if (fastpath(!ISA()->hasCustomRR())) {
        //在ARC中直接调用
        return rootRetain();
    }
 
    return ((id(*)(objc_object *, SEL))objc_msgSend)(this, SEL_retain);
}
 
ALWAYS_INLINE id 
objc_object::rootRetain()
{
    return rootRetain(false, false);
}
 
ALWAYS_INLINE id 
objc_object::rootRetain(bool tryRetain, bool handleOverflow)
{
    if (isTaggedPointer()) return (id)this;
 
    bool sideTableLocked = false;
    //是否将引用计数结果转存到sidetable中
    bool transcribeToSideTable = false;
 
    isa_t oldisa;
    isa_t newisa;
 
    do {
        transcribeToSideTable = false;
        oldisa = LoadExclusive(&isa.bits);
        newisa = oldisa;
        //不支持nonpointer：引用计数保存在sidetable中
        if (slowpath(!newisa.nonpointer)) {
            ClearExclusive(&isa.bits);
            //不尝试增加引用计数且sidetable被锁，则打开sidetable锁定
            if (!tryRetain && sideTableLocked) sidetable_unlock();
            //尝试增加引用计数:如果尝试成功则返回当前对象;如果尝试失败则返回nil
            if (tryRetain) return sidetable_tryRetain() ? (id)this : nil;
            //直接进行retain操作:在sidetable中使引用计数增加1
            else return sidetable_retain();
        }
        
        //支持nonpointer
 
        // 尝试增加引用计数且对象正在释放
        if (slowpath(tryRetain && newisa.deallocating)) {
            ClearExclusive(&isa.bits);
            if (!tryRetain && sideTableLocked) sidetable_unlock();
            return nil;
        }
        uintptr_t carry;
        newisa.bits = addc(newisa.bits, RC_ONE, 0, &carry);  // extra_rc++
 
        //extra_rc溢出
        if (slowpath(carry)) {
            // newisa.extra_rc++ overflowed
            if (!handleOverflow) {
                ClearExclusive(&isa.bits);
                return rootRetain_overflow(tryRetain);
            }
            // Leave half of the retain counts inline and 
            // prepare to copy the other half to the side table.
            if (!tryRetain && !sideTableLocked) sidetable_lock();
            //sidetable加锁
            sideTableLocked = true;
            //引用计数转存到sidetable中
            transcribeToSideTable = true;
            newisa.extra_rc = RC_HALF;
            newisa.has_sidetable_rc = true;
        }
    } while (slowpath(!StoreExclusive(&isa.bits, oldisa.bits, newisa.bits)));
 
    if (slowpath(transcribeToSideTable)) {
        // Copy the other half of the retain counts to the side table.
        sidetable_addExtraRC_nolock(RC_HALF);
    }
 
    if (slowpath(!tryRetain && sideTableLocked)) sidetable_unlock();
    return (id)this;
}


```


- 在不支持nonpointer的对象中,引用计数只存储在sidetable中;
- 可以看出在支持nonpointer的情况下：
	- 在isa的位域extra_rc足以存储的情况,引用计数会优先选择存储在isa的extra_rc位域中;
	- 若isa位域extra_rc溢出，则会选择将将引用计数的RC_HALF(如果extra_rc占据8bit，则RC_HALF=2^7)保存在isa中，另一半RC_HALF叠加保存在sidetable中.之所以这样选择是因为isa_t的存取理论上会比sidetable的操作效率上快很多,这样做既可以使超出extra_rc存储范围的引用计数得到有效存储,又可以确保引用计数的增减足够快速(存取都以extra_rc优先).


<br/>

**引用计数的获取**

因为对象的引用计数主要存储在isa位域extra_rc和散列表中,所以要获取对象的引用计数也需要从两个位置进行获取.

```
uintptr_t
_objc_rootRetainCount(id obj)
{
    assert(obj);
 
    return obj->rootRetainCount();
}
 
inline uintptr_t 
objc_object::rootRetainCount()
{
    if (isTaggedPointer()) return (uintptr_t)this;
 
    //添加sidetable操作锁
    sidetable_lock();
    //加载对象isa
    isa_t bits = LoadExclusive(&isa.bits);
    ClearExclusive(&isa.bits);
    //如果对象开启了nonpointer
    if (bits.nonpointer) {
        //获取isa中存储的引用计数
        uintptr_t rc = 1 + bits.extra_rc;
        if (bits.has_sidetable_rc) {
            //获取散列表中存储的引用计数
            rc += sidetable_getExtraRC_nolock();
        }
        sidetable_unlock();
        return rc;
    }
    //对象未开启nonpointer
    sidetable_unlock();
    return sidetable_retainCount();
}
 
 
 
uintptr_t
objc_object::sidetable_retainCount()
{
    SideTable& table = SideTables()[this];
 
    size_t refcnt_result = 1;
    
    table.lock();
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it != table.refcnts.end()) {
        // this is valid for SIDE_TABLE_RC_PINNED too
        refcnt_result += it->second >> SIDE_TABLE_RC_SHIFT;
    }
    table.unlock();
    return refcnt_result;
}


```

- 在开启nonpointer的对象中,对象的引用计数包括两部分:
	- 存储在isa中的引用计数;
	- 存储在sidetable中的引用计数.
- 在未开启nonpointer的对象中,对象的引用计数全部存储在sidetable中，只需要从sidetable中获取就可以.


这里有一个有意思的问题：

- 在开启nonpointer的对象中,引用计数retainCount=1+isa.extra_rc+sidetable_getExtraRC_nolock();
- 在未开启nonpointer的对象中,引用计数retainCount=1+table.refcnts.find(this)->second >> SIDE_TABLE_RC_SHIFT;


&emsp; 所以其实在创建对象的过程中(例如alloc和copy)，存储的引用计数并没有进行+1操作,而是默认对象的最小引用计数为1.在release操作时才会判断引用计数-1是否为0，如果为0则进行释放;否则进行引用计数-1.

[引用计数说起](https://juejin.cn/post/6844903919127101448)


<br/>
<br/>


> <h2 id="类库没有导入">类库没有导入</h2>

- 当一个类库没有进行导入文件时，但是我想使用它，如何做到？
	使用runtime


<br/>

***
<br/>

># 网络

[**iOS状态码解读**](https://github.com/ChenYilong/iOSDevelopmentTips/blob/master/Tips/HTTP状态码汇总.md)

<br/>

> <h2 id="NSURLSession与RunLoop的联系">NSURLSession与RunLoop的联系</h2>


AFNet2.0网络请求常驻线程机制

```
//使用GCD开启一个子线程来发送网络请求
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    //使用非自动发送网络请求模式,发送请求OK
    /*
    //创建NSURLConnection对象，设置代理，暂不发送
	  NSURLConnection *connect =  [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
    //设置代理方法的执行队列
    [connect setDelegateQueue:[[NSOperationQueue alloc]init]];

    //调用start发送网络请求
    [connect start];
    */

    //使用自动发送网络请求模式，发送请求失败（需要改造代码）
    //WHY?
    /*1⃣️ 网络请求发送和数据接收是否成功，和一些因素相关，比如客户端的网速、服务器端的查询速度等等。
      2⃣️ 而在子线程中创建的NSURLConnection对象是一个临时变量，当请求发送完成之后就被释放了，所以这个时候它的代理方法不会调用用。
      3⃣️ 为什么使用非自动发送网络请求模式是OK的。
        因为在该模式中，调用了start来开始发送网络请求，该方法内部会自动将当前的connect作为一个Source添加到当前线程所在的Runloop中
        如果当前线程是子线程（即当前线程的runloop并未创建），那么该方法内部会默认先创建当前线程的Runloop,设置在runloop的默认模式下运行。
        此时runloop会对这个Connect对象进行强引用，保证了代理方法被调用的前提
     */
    NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self];
    [connect setDelegateQueue:[[NSOperationQueue alloc]init]];
    
    //创建当前线程的runloop，并开启runloop
    [[NSRunLoop currentRunLoop] run];
});

```

&emsp; 在 SDWebImage 的 v3.7.0 版本及以前，并没有引入 NSURLSession 而是采用的 NSURLConnection。而后者往往是需要与 Runloop 协同使用，因为每个 Connect 会作为一个 Source 添加到当前线程所在的 Runloop 中，并且 Runloop 会对这个 Connect 对象强引用，以保证代理方法可以调用。

<br/>

iOS 中，关于网络请求的接口自下至上有如下几层:

```
CFSocket
CFNetwork       ->ASIHttpRequest
NSURLConnection ->AFNetworking
NSURLSession    ->AFNetworking2, Alamofire
```


CFSocket 是最底层的接口，只负责 socket 通信。
- CFNetwork 是基于 CFSocket 等接口的上层封装，ASIHttpRequest 工作于这一层。
- NSURLConnection 是基于 CFNetwork 的更高层的封装，提供面向对象的接口，AFNetworking 工作于这一层。
- NSURLSession 是 iOS7 中新增的接口，表面上是和 NSURLConnection 并列的，但底层仍然用到了 NSURLConnection 的部分功能 (比如 com.apple.NSURLConnectionLoader 线程)，AFNetworking2 和 Alamofire 工作于这一层。

下面主要介绍下 NSURLConnection 的工作过程。

&emsp; 通常使用 NSURLConnection 时，你会传入一个 Delegate，当调用了 [connection start] 后，这个 Delegate 就会不停收到事件回调。实际上，start 这个函数的内部会会获取 CurrentRunLoop，然后在其中的 DefaultMode 添加了4个 Source0 (即需要手动触发的Source)。CFMultiplexerSource 是负责各种 Delegate 回调的，CFHTTPCookieStorage 是处理各种 Cookie 的。

&emsp; 当开始网络传输时，我们可以看到 NSURLConnection 创建了两个新线程：com.apple.NSURLConnectionLoader 和 com.apple.CFSocket.private。其中 CFSocket 线程是处理底层 socket 连接的。NSURLConnectionLoader 这个线程内部会使用 RunLoop 来接收底层 socket 的事件，并通过之前添加的 Source0 通知到上层的 Delegate。

<br/>

![ <br/> ](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/AFNet11.png)



**RunLoop实际应用举例**

AFNetworking
AFURLConnectionOperation 这个类是基于 NSURLConnection 构建的，其希望能在后台线程接收 Delegate 回调。为此 AFNetworking 单独创建了一个线程，并在这个线程中启动了一个 RunLoop：


```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
 
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```

RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，所以 AFNetworking 在 [runLoop run] 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 NSMachPort (mach_port) 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。

```
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}
```

当需要这个后台线程执行任务时，AFNetworking 通过调用 [NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的 RunLoop 中。


<br/>
<br/>





> <h3 id="网络性能优化">[**网络性能优化**](https://www.jianshu.com/p/a470ab485e39)</h3>


<br/>

- [**网络缓存（URL缓存）**](https://www.jianshu.com/p/fb5aaeac06ef)

&emsp; 如果打算在項目中加入網絡請求緩存，你並不需要自己造一個輪子，瞭解一下NSURLCache就足夠，這是一個Apple已經為你準備好了的網絡請求緩存類。

```
public enum NSURLRequestCachePolicy : UInt {
case UseProtocolCachePolicy // 默認值
case ReloadIgnoringLocalCacheData // 不使用緩存數據
case ReloadIgnoringLocalAndRemoteCacheData // Unimplemented
public static var ReloadIgnoringCacheData: NSURLRequestCachePolicy { get } //忽略缓存直接从原始地址下载
case ReturnCacheDataElseLoad // 無論緩存是否過期都是用緩存，沒有緩存就進行網絡請求
case ReturnCacheDataDontLoad // 無論緩存是否過期都是用緩存，沒有緩存也不會進行網絡請求
case ReloadRevalidatingCacheData // Unimplemented
}



//使用NSURLCache之前需要在AppDelegate中緩存空間的設置：
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	NSURLCache *URLCache = [[NSURLCache alloc] initWithMemoryCapacity:4 * 1024 * 1024
	diskCapacity:20 * 1024 * 1024
	diskPath:nil];
	[NSURLCache setSharedURLCache:URLCache];
}
```

&emsp; 文件的缓存是否有效是通过`Last-Modified`和`ETag`来进行判断的。

<br/>

- **`Last-Modified`**顾名思义，是资源最后修改的时间戳，往往与缓存时间进行对比来判断缓存是否过期。

&emsp; 在浏览器第一次请求某一个URL时，服务器端的返回状态会是200，内容是你请求的资源，同时有一个Last-Modified的属性标记此文件在服务期端最后被修改的时间，格式类似这样：

`Last-Modified: Fri, 12 May 2006 18:53:33 GMT`

&emsp; 客户端第二次请求此URL时，根据 HTTP 协议的规定，浏览器会向服务器传送 If-Modified-Since 报头，询问该时间之后文件是否有被修改过：

`If-Modified-Since: Fri, 12 May 2006 18:53:33 GMT`


<br/>

- ETag 是的功能与 Last-Modified 类似：服务端不会每次都会返回文件资源。

&emsp; 客户端每次向服务端发送上次服务器返回的 ETag 值，服务器会根据客户端与服务端的 ETag 值是否相等，来决定是否返回 data，同时总是返回对应的 HTTP 状态码。客户端通过 HTTP 状态码来决定是否使用缓存。比如：服务端与客户端的 ETag 值相等，则 HTTP 状态码为 304，不返回 data。服务端文件一旦修改，服务端与客户端的 ETag 值不等，并且状态值会变为200，同时返回 data。


总结： 在官方给出的文档中提出 ETag 是首选的方式，优于 Last-Modified 方式。因为 ETag 是基于 hash ，hash 的规则可以自己设置，而且是基于一致性，是“强校验”。 Last-Modified 是基于时间，是弱校验，弱在哪里？比如说：如果服务端的资源回滚客户端的 Last-Modified 反而会比服务端还要新。


<br/>

- [网络模块优化(失败、缓存请求重发)](https://www.cnblogs.com/ziyi--caolu/p/8176331.html)


<br/>

- [HTTPDNS实践](https://www.jianshu.com/p/fb54405dbdca)

[HTTPDNS在App开发中实践总结](http://www.iosfly.com/2016/12/03/HTTPDNS/)

为什么有些公司的网络接口使用的英文域名而不是IP地址？

这是因为使用IP地址就可以防止你的域名被DNS劫持，造成网络差和其他运营商给你加各种内容。

- [HttpDns 在 iOS 端的接入方案](https://juejin.cn/post/6844904144705339400)

- [即时通讯性能调优](https://github.com/ChenYilong/iOSBlog/issues/6)
- [网络请求优化之取消请求](https://www.jianshu.com/p/20f6172524d6)
- [iOS网络层设计](https://www.jianshu.com/p/fe0dd50d0af1)






<br/>

***
<br/>

># <h2 id = "类库">类库<h3>

<br/>
<br/>

> <h2 id = "fishhook">FishHook<h3>

hook系统函数，一个faceBook写的三方框架


<br/>
<br/>

> <h2 id = "静态库和动态库">[静态库和动态库](https://www.jianshu.com/p/a200b593696b)<h3>

介绍
> 静态库：链接时完整地拷贝至可执行文件中，被多次使用就有多份冗余拷贝。利用静态函数库编译成的文件比较大，因为整个 函数库的所有数据都会被整合进目标代码中，他的优点就显而易见了，即编译后的执行程序不需要外部的函数库支持，因为所有使用的函数都已经被编译进去了。当然这也会成为他的缺点，因为如果静态函数库改变了，那么你的程序必须重新编译。



<br/>

> 动态库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。由于函数库没有被整合进你的程序，而是程序运行时动态的申请并调用，所以程序的运行环境中必须提供相应的库。动态函数库的改变并不影响你的程序，所以动态函数库的升级比较方便。

&emsp; 静态库和动态库都是闭源库，只能拿来满足某个功能的使用，不会暴露内部具体的代码信息，而从github上下载的第三方库大多是开源库


<br/>

静态库优点：
>- 模块化，分工合作
- 避免少量改动经常导致大量的重复编译连接
- 也可以重用，注意不是共享使用


动态库优点：

>- 可以将最终可执行文件体积缩小
- 多个应用程序共享内存中得同一份库文件，节省资源
- 可以不重新编译连接可执行程序的前提下，更新动态库文件达到更新应用程序的目的。
- 将整个应用程序分模块，团队合作，进行分工，影响比较小。


<br/>
格式区别：
>静态库：.a和.framework (windows:.lib , linux: .a)

>动态库：.dylib和.framework（系统提供给我们的framework都是动态库！）(windows:.dll , linux: .so)

注意：两者都有framework的格式，但是当你创建一个framework文件时，系统默认是动态库的格式，如果想做成静态库，需要在buildSetting中将Mach-O Type选项设置为Static Library就行了！


<br/>
<br/>

- <h2 id = "库引用错误解析">库引用错误解析</h2>

framework和[host工程](https://www.jianshu.com/p/d25f9465cfa8)(可执行的工程项目)资源共用

第方三库

Class XXX is implemented in both XXX and XXX. One of the two will be used. Which one is undefined.

这是当 framework 工程和 host 工程链接了相同的第三方库或者类造成的。

为了让打出的 framework 中不包含 host 工程中已包含的三方库（如 cocoapods 工程编译出的 .a 文件），可以这样：

删除 Build Phases > Link Binary With Libraries 中的内容（如有）。此时编译会提示三方库中包含的符号找不到。

在 framework 的 Build Settings > Other Linker Flags 添加 -undefined dynamic_lookup。必须保证 host 工程编译出的二进制文件中包含这些符号。


<br/>
<br/>

> <h2 id = "MJRefresh">**MJRefresh**<h3>

- MJRefresh使用时有什么问题？有哪些需要注意的？

&emsp;
**事项1：**在很多情况下，如果endRefreshing 先执行  reloadData后执行 会出现indexPath.row不准确（值为4）这时候你的数组中很有可能不到四个成员，这时候就会造成数组越界。

所以一定要先执行reloadData 方法 再执行endRefreshing方法

```
 [_tableView reloadData];

 [_tableView.header endRefreshing];

 [_tableView.footer endRefreshing];
```


<br/>

**事项2：**使用上拉加载更多， 当数据加载完调用`[ableView.mj_footer endRefreshingWithNoMoreData]`方法显示已经全部加载完毕这个方法，一定要放在`[tableView.mj_footer endRefreshing];`停止加载方法的后面，不然`[ableView.mj_footer endRefreshingWithNoMoreData]`这个方法会失效。

```
[Request requestGET:parameter view:nil urlPre:CourseCenterURL funItem:HuFuncItem_DiscoverCourseLibraryList success:^(NSDictionary *dic) {
    NSArray *dataListArray =  [HospitalShareRightModel mj_objectArrayWithKeyValuesArray:dic[@"data"][@"result"]];
    //下拉刷新
    if (weakSelf.pageNum == kPageNum) {
        [weakSelf.rightTableView.mj_header endRefreshing];
        [weakSelf.hospitalShareRightArray removeAllObjects];
        if (dataListArray.count <= 0) {
            [weakSelf.rightTableView removeFromSuperview];
            [weakSelf defauleView];
        }
    }
    [weakSelf.rightTableView.mj_footer endRefreshing];
    [weakSelf.hospitalShareRightArray addObjectsFromArray:dataListArray];
    if (dataListArray.count < 10) {
        [weakSelf.rightTableView.mj_footer endRefreshingWithNoMoreData];//放到停止加载方法后面 不然会失效
    }
    [weakSelf.rightTableView reloadData];
} error:^(NSDictionary *dic) {
    [MBProgressHUD showError:dic[@"errmsg"]];
} failure:^{
}];

```






















