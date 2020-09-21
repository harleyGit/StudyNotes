解码：
CPU 把就算好的内容提交给GPU;
GPU 进行变换、合成、渲染；

性能优化：
-  透明度：消耗CPU，若有透明度，CPU需要计算；
-  圆角、阴影离屏渲染会消耗CPU，这是因为有太多的layer导致不停地切换上下文消耗时间、性能；
-  重绘；
-  图片解压：无论是从磁盘加载还是从网络下载的图片都是没经过解压，在渲染的时候解压，在主线程解压；


自动打包的时候使用shell脚本。


<br/>
***
<br/>
># NSTimer 内存泄漏
#`解决方法一`
DEMO:
viewController -----Push------>LeakedViewController


#`LeakedViewController.m`
```
- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");
}


- (void)viewDidLoad {
    [super viewDidLoad];
    
    //会造成内存泄漏，因为这个NSTimer在主线程中运行，它强引用了self，只要它运行主控制器就不会释放掉，造成了内存的泄漏。
    //用weak也不行，因为在他的内部使用了strong强引用
    self.timer = [NSTimer timerWithTimeInterval:1.0f target:self selector:@selector(fire) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSDefaultRunLoopMode];
}

- (void)fire {
    NSLog(@"fire 调用了");
}

- (void)didMoveToParentViewController:(UIViewController *)parent {
    if (parent == nil) {
        [self.timer invalidate];
        self.timer = nil;
    }
}

//可以解决内存泄漏问题，调用dealloc析构方法

```
输出：
`2019-06-08 23:54:20.937609+0800 Genealogy[7703:553726] fire 调用了`
`2019-06-08 23:54:21.938562+0800 Genealogy[7703:553726] fire 调用了`
`2019-06-08 23:54:22.179312+0800 Genealogy[7703:553726] LeakedViewController 调用dealloc 方法`


![引用关系](https://upload-images.jianshu.io/upload_images/2959789-296953927f40b615.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

#`解决方案二`
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //不让当前timer强制引用当前的vc,self持有_target
    _target = [NSObject new];
    class_addMethod([_target class], @selector(fire), (IMP)fireImp, "v@:");
    
    //self.timer持有了_target,timer没有强引用self
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_target selector:@selector(fire) userInfo:nil repeats:YES];
}

void fireImp(id self, SEL _cmd){
    NSLog(@"fireImp 调用了 fire ");
}

- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");


    [self.timer invalidate];
    self.timer = nil;
}

```
输出：
`2019-06-09 10:58:13.603251+0800 Genealogy[10606:696072] fireImp 调用了 fire `
`2019-06-09 10:58:14.603689+0800 Genealogy[10606:696072] fireImp 调用了 fire `
`2019-06-09 10:58:14.684340+0800 Genealogy[10606:696072] GBuildFamilyController 调用dealloc 方法`



<br/>
#`解决方案三`

#`GProxy.h 文件`
```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface GProxy : NSProxy

@property(nonatomic, weak)id target;

@end

NS_ASSUME_NONNULL_END
```

#`GProxy.m 文件`
```
#import "GProxy.h"
#import <objc/runtime.h>

@implementation GProxy

//方法签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [self.target methodSignatureForSelector:sel];
}

//消息转发
- (void)forwardInvocation:(NSInvocation *)invocation {
    [invocation invokeWithTarget:self.target];
}

@end

```

#`GBuildFamilyController.m 文件`
```
- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");
    
    [self.timer invalidate];
    self.timer = nil;
}


- (void)viewDidLoad {
    [super viewDidLoad];
    
    _gproxy = [GProxy alloc];
    //_gproxy.target弱引用了self,self可以正常走dealloc析构方法
    _gproxy.target = self;
    
    //self.timer强引用了_gproxy.target
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_gproxy.target selector:@selector(fire) userInfo:nil repeats:YES];
}

- (void)fire {
    NSLog(@"fire 调用了");
}
```
输出：
`2019-06-09 13:14:14.188451+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:15.188988+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:16.187918+0800 Genealogy[11849:767419] fire 调用了`
`2019-06-09 13:14:17.111693+0800 Genealogy[11849:767419] GBuildFamilyController 调用dealloc 方法`


<br/>
***
<br/>
>#  内存检测

#`静态检测`
【Product】-> 【Analyze】


`几种内存泄漏情况`
```
//block 传空值
 [self sceneO3:nil];

//CG C 语言需要我们手动释放
- (void) sceneO1 {
    CGPathRef shadowPah = CGPathCreateWithRect(CGRectMake(0, 2, 0, 20), NULL);
}

//文件操作，打开文件流但没关闭文件流会发生内存泄漏
- (void) sceneO2 {
    FILE *f;
    f = fopen("info.plist", "f");
}


- (void) sceneO3:(void(^)(void))callback {
    //需要对这个block进行判空，否则会发生内存泄漏
    callback();
}

- (NSArray *) sceneO4 {
    NSString *str = nil;
    //array中插入空值，会发生内存泄漏
    return @[@"sceneo1", str, @"scence02"];
}
```
![设置编译时检测内存泄漏](https://upload-images.jianshu.io/upload_images/2959789-e8ed173365fbc179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

#`动态检测(instrument检测，MLeakFinder[第三方方法])`



#`析构方法打印`





<br/>
***
<br/>



<br/>
***
<br/>
