
<h2 id=''></h2>
- [**RAC 函数式响应式编程框架**](#RAC函数式响应式编程框架)
	- [订阅信号](#订阅信号)
	- [私有方法订阅](#私有方法订阅)
- [**RAC的KVO**](#RAC的KVO)
- [**RAC的UI**](#RAC的UI)
	- [UIButton订阅信号](#UIButton订阅信号)
	- [UITextField的RAC](#UITextField的RAC)
- [**通知的 RAC**](#通知的RAC)
- [**NSTimer的RAC**](#NSTimer的RAC)
- [**宏定义RAC**](#宏定义RAC)
- [**RAC循环引用**](#RAC循环引用)

<br/>

***
<br/>



<h1 id='RAC函数式响应式编程框架'>RAC 函数式响应式编程框架</h1>

**`类似的框架：Masonry、AFNetworking`**

<h2 id='订阅信号'>订阅信号</h2>


```
- (void) reactiveObjTest {
    //1.订阅信号    -- 在实际开发中，就是提供给外界订阅的
    /*
     创建一个可变数组变量 _subscribers
     */
    RACSubject *subject = [RACSubject subject];
    
    //2.订阅信号    -- 代理\通知对比
    //相当于代理 objc.delegate = self;
    /*
     调用这个方法所做的事情
     1. 创建订阅者对象
     2. 将Block放到订阅者对象中
     3. 将订阅者对象放到_subscriber数组里面
     */
    [subject subscribeNext:^(id  _Nullable x) {//调用这个方法的返回值，可以用来取消订阅
        NSLog(@"%@", x);
    }];
    
    //3.注册信号
    /*
     遍历信号对象中的数组，取出订阅者，调用订阅者对象中的Block，然后执行
     */
    [subject sendNext:@"O(∩_∩)O哈！"];
    
    
}
```

输出：

`2019-06-06 15:34:46.583177+0800 Genealogy[6591:256569] O(∩_∩)O哈！`

<br/>

**`源码：`**

```

+ (instancetype)subject {
	return [[self alloc] init];
}

- (instancetype)init {
	self = [super init];
	if (self == nil) return nil;
        
    //取消订阅信号
	_disposable = [RACCompoundDisposable compoundDisposable];
	_subscribers = [[NSMutableArray alloc] initWithCapacity:1];
	
	return self;
}

- (void)dealloc {
    //取消订阅
	[self.disposable dispose];
}


- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock {
	NSCParameterAssert(nextBlock != NULL);
	
	RACSubscriber *o = [RACSubscriber subscriberWithNext:nextBlock error:NULL completed:NULL];
	return [self subscribe:o];
}


+ (instancetype)subscriberWithNext:(void (^)(id x))next error:(void (^)(NSError *error))error completed:(void (^)(void))completed {
	RACSubscriber *subscriber = [[self alloc] init];

    //next block保存在我们新创建的订阅者subscriber中了
	subscriber->_next = [next copy];
	subscriber->_error = [error copy];
	subscriber->_completed = [completed copy];

	return subscriber;
}

```


<br/>

<h2 id='私有方法订阅'>私有方法订阅</h2>

`GFamilyView.h`

```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface GFamilyView : UIView

@end

NS_ASSUME_NONNULL_END
```

`GFamily.m`

```
@interface GFamilyView ()

@property(nonatomic, retain)UIButton *buildFamilyBtn;

@end

@implementation GFamilyView

- (void) buildFamilyAction:(UIButton *)sender {

}

@end
```

`GMyFamilyController.m`调用

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.navigationItem.title = @"我的家族";
    
    [self addSubViews];
    
    //这里使用了运行时，调用其私有的方法
    [[self.familyView rac_signalForSelector:@selector(buildFamilyAction:)] subscribeNext:^(RACTuple * _Nullable x) {//x 是buildFamilyAction参数的集合
        NSLog(@"观察到了按钮点击！");
    }];
}
```

输出：

`2019-06-06 17:13:19.408324+0800 Genealogy[7979:352547] 观察到了按钮点击！`



<br/>

***
<br/>

<h1 id='RAC的KVO'>RAC的KVO</h1>


导入文件：`#import <NSObject+RACKVOWrapper.h>`

`GMyFamilyController.m  文件`

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.navigationItem.title = @"我的家族";
    
	//函数式编程思想
    [self.familyView rac_observeKeyPath:@"frame" options:NSKeyValueObservingOptionNew observer:self block:^(id value, NSDictionary *change, BOOL causedByDealloc, BOOL affectedOnlyLastComponent) {
        //
        NSLog(@"%@", change);
    }];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.familyView.frame = CGRectMake(0, 0, 50, 50);
}
```
输出：

```
2019-06-06 18:04:55.406959+0800 Genealogy[8733:402957] {
    kind = 1;
    new = "NSRect: {{0, 0}, {50, 50}}";
}
```




<br/>

***
<br/>

<h1 id='RAC的UI'>RAC的UI</h1>


<h2 id='UIButton订阅信号'>UIButton订阅信号</h2>

**`UIButton订阅信号`**

```
//包装按钮的点击为信号，然后订阅事件
[[self.buildFamilyBtn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(__kindof UIControl * _Nullable x) {
            NSLog(@"--> %@", x);
        }];
```

输出：

```
2019-06-08 13:03:02.562591+0800 Genealogy[2559:126706] --> <UIButton: 0x7fe04fe66540; frame = (167 271.667; 80 80); opaque = NO; layer = <CALayer: 0x60000024e480>>
```


<br/>
<br/>

<h2 id='UITextField的RAC'>UITextField的RAC</h2>


**`UITextField 的 RAC`**

```
 [[self.textField rac_textSignal] subscribeNext:^(NSString * _Nullable x) {
            NSLog(@"-->> %@", x);
        }];
```

输出：

```
2019-06-08 13:25:44.316737+0800 Genealogy[2772:148790] -->> W
2019-06-08 13:25:44.560117+0800 Genealogy[2772:148790] -->> Ww
2019-06-08 13:25:44.577652+0800 Genealogy[2772:148790] -->> Www
2019-06-08 13:25:44.597110+0800 Genealogy[2772:148790] -->> Wwww
```





<br/>

***
<br/>

<h1 id='通知的 RAC'>通知的 RAC</h1>

```
//包装信号，订阅信号
//程序进入后台
[[[NSNotificationCenter defaultCenter] rac_addObserverForName:UIApplicationDidEnterBackgroundNotification object:nil] subscribeNext:^(NSNotification * _Nullable x) {
            NSLog(@"--> %@",x);
        }];
```

输出：

```
2019-06-08 13:14:18.153235+0800 Genealogy[2690:139236] --> NSConcreteNotification 0x600002bd4a80 {name = UIApplicationDidEnterBackgroundNotification; object = <UIApplication: 0x7f9cf46014a0>}
```




<br/>

***
<br/>


<h1 id='NSTimer的RAC'>NSTimer的RAC</h1>

```
//通过源码可以看到是在GCD的基础上进行封装的NSTimer
[[RACSignal interval:1.0 onScheduler:[RACScheduler scheduler]] subscribeNext:^(NSDate * _Nullable x) {
            NSLog(@"%@, thread: %@", x, [NSThread currentThread]);
        }];
```

输出：

```
2019-06-08 14:53:27.684149+0800 Genealogy[3360:188747] Sat Jun  8 14:53:27 2019, thread: <NSThread: 0x6000013a5600>{number = 4, name = (null)}
2019-06-08 14:53:28.684820+0800 Genealogy[3360:188776] Sat Jun  8 14:53:28 2019, thread: <NSThread: 0x6000013a5800>{number = 5, name = (null)}
2019-06-08 14:53:29.684560+0800 Genealogy[3360:188776] Sat Jun  8 14:53:29 2019, thread: <NSThread: 0x6000013a5800>{number = 5, name = (null)}
```



<br/>

***
<br/>


<h1 id='宏定义RAC'>宏定义RAC</h1>


>**`宏定义信号赋值`**

```
 //用来给某个对象的某个属性绑定信号，只要产生信号内容，就会把内容给属性赋值
 RAC(self.label, text) = self.textField.rac_textSignal;
```

效果图：![UILabel 和 UITextField](https://upload-images.jianshu.io/upload_images/2959789-9fbb5123b2acf374.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

>**`观察者的宏定义`**


```
[RACObserve(self.familyView, frame) subscribeNext:^(id  _Nullable x) {
        NSLog(@"---->> %@", x);
    }];
```

输出：

`2019-06-08 15:20:47.845109+0800 Genealogy[3581:216452] ---->> NSRect: {{0, 0}, {50, 50}}`



代理delegate一般使用weak修饰，但是有一个例外那就是NSURLSession的代理使用的是strong进行修饰的，因为在iOS设计中这个类使用的是单例设计模式，它的实例对象是不需要释放的，它就是循环引用的。


<br/>

***
<br/>

<h1 id='RAC循环引用'>RAC循环引用</h1>


**`GBuildFamilyController.m`**

```
- (void)dealloc {
    NSLog(@"GBuildFamilyController 调用dealloc 方法");
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    @weakify(self);
    RACSignal *signal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {//这个block会保存在这signal中
        
        //没有使用@weakify：在这个block中会给self强制加一个strong，引用技术+1，会造成内存泄漏不会调用dealloc方法
        //使用@weakify后，若是这个controller提交释放掉，这个block就会crash
        //为了避免，给这个self加一个@strongify
        @strongify(self);
        NSLog(@"%@", self);
        
        return nil;
    }];
    
    //signal保存在控制器中,出现循环引用
    _signal = signal;
}

```

当pop时，会输出：

`2019-06-08 16:56:15.727436+0800 Genealogy[4656:316390] GBuildFamilyController 调用dealloc 方法`




<br/>

***
<br/>




<br/>

***
<br/>




<br/>

***
<br/>



