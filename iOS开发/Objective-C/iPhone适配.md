# iPhone机型屏幕尺寸
| 手机型号 | 屏幕尺寸 | 物理尺寸(逻辑分辨率pt) | 像素尺寸分辨率(px) | 倍图 |
|:--|:--|:--|:--|:--|
|iPhone 5 / 5c / 5S	  | 4英寸 |  320 * 568pt | 	640 * 1136px |  @2x|
| iPhone 6 / 6s | 4.7英寸 | 375 * 667pt | 750 * 1334 px | @2x |
| iPhone  6+ / 6s+ | 5.5英寸 | 414 * 736pt |  1242 * 2208px		| @3x |
| iPhone 7 | 4.7英寸 | 375 * 667pt | 750 * 1334 px | @2x |
| iPhone 7+ | 5.5英寸 | 414 * 736pt |  1242 * 2208px		| @3x |
| iPhone 8 | 4.7英寸 | 375 * 667pt | 750 * 1334 px | @2x |
| iPhone 8+ | 5.5英寸 | 414 * 736pt |  1242 * 2208px		| @3x |
| iPhone X	 | 5.8英寸 | 375 * 812pt | 1125 * 2436px	 | @3x |


<br/>

***
<br/>

# 全面屏和非全面屏显示

- 非全面屏状态栏（iPhoneX 之前机型导航栏） 

![非全面屏状态栏](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a8.png)


- 全面屏状态栏（iPhoneX 之后机型导航栏）
![全面屏状态栏](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a9.png)


&emsp;	从以上两图，我们可以看出全面屏的顶部`Statusbar`变高了，其他部分没变。`Largetitle`是iOS11中新加入的特性。当然我们开发中很少用到Largetitle。



<br/>

***

-	全面屏底部多出区域

&emsp;	全面屏底部多出了高度为34的Home Indicator 区域。


<br/>

***


-	iPhone 分辨率

![iPhone分辨率](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a6.png)



<br/>

***


-	iPhone 适配

	-	在iOS7以后，苹果给UIViewController引入了topLayoutGuide 和 bottomLayoutGuide两个属性。用于表示顶部或底部的高度。根据有无显示状态栏、导航栏、tabBar返回高度。如果VC内嵌VC，内嵌的VC将视为另起的顶底坐标体系，不受原状态栏等影响。你可能听都没听过这两属性，因为开发中我们习惯直接用宏写上数值，几乎不用这两个属性。更何况那时iPhone还没有全面屏这种东西。

	-	iOS11.0 之前只能通过iPhone的尺寸和分辨率来辨别全面屏和非全面屏，但是这有一个问题：不同的全面屏的size和分辨率是不同的，怎么办？

	-	好在iOS11.0 之后，苹果弃用了topLayoutGuide和bottomLayoutGuide两个属性，苹果推出了`SafeArea`概念。
官方的建议是 `不能被遮挡的内容和控件在安全区域范围内显示`。如果视图底部有按钮，在全面屏下，`请约束底部距离34`，不要影响到Home功能。



全面屏版本11.0之前和之后的代码：

```

// 单纯根据分辨率
#define K_iPhoneXStyle ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) : NO)

// 单纯根据屏幕宽高
#define  K_iPhoneXStyle (KScreenWidth == 375.f && KScreenHeight == 812.f ? YES : NO)

#define KStatusBarHeight (K_iPhoneXStyle ? 44.f : 20.f)


[testView mas_makeConstraints:^(MASConstraintMaker *make) {
    
    make.height.equalTo(@44);
    if (@available(iOS 11.0,*)) {
        make.top.equalTo(self.view.mas_safeAreaLayoutGuideTop);
        make.left.equalTo(self.view.mas_safeAreaLayoutGuideLeft);
        make.right.equalTo(self.view.mas_safeAreaLayoutGuideRight);
    } else {
        make.top.equalTo(self.view).offset(KStatusBarHeight);
        make.left.equalTo(self.view);
        make.right.equalTo(self.view);
    }
}];




```

<br/>

***


- safeAreaInsets

此属性适用于手动计算frame。iPhoneX竖屏时控制器view的safeAreaInsets是（44，0，34，0）；横屏(0, 44, 21, 44)。


```
#define KScreenWidth ([UIScreen mainScreen].bounds.size.width)

#define KScreenHeight ([UIScreen mainScreen].bounds.size.height)

// 单纯根据屏幕宽高
#define  K_iPhoneXStyle (KScreenWidth == 375.f && KScreenHeight == 812.f ? YES : NO)

#define KStatusBarHeight (K_iPhoneXStyle ? 44.f : 20.f)

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor redColor];
    UIView *testView = [[UIView alloc] initWithFrame:CGRectMake(0, KStatusBarHeight, KScreenWidth, 200)];
    testView.backgroundColor = [UIColor blackColor];
    [self.view addSubview:testView];
    self.testView = testView;
}

- (void)viewSafeAreaInsetsDidChange {
    [super viewSafeAreaInsetsDidChange];
    NSLog(@"%s", __func__);
    
    [self updateFrame];
}

- (void)updateFrame {
    if (@available(iOS 11.0, *)) {
        CGRect newFrame = self.testView.frame;
        newFrame.origin.x = self.view.safeAreaInsets.left;
        newFrame.size.width = KScreenWidth - self.view.safeAreaInsets.left - self.view.safeAreaInsets.right;
        self.testView.frame = newFrame;
    }
}

```


<br/>

***

- automaticallyAdjustsScrollViewInsets 和 contentInsetAdjustmentBehavior

	-	automaticallyAdjustsScrollViewInsets
	&emsp;	在iOS7.0以后,相对于ScrollView新增属性,默认为YES,系统会根据所在界面的astatus bar, search bar, navigation bar, toolbar, or tab bar等自动调整ScrollView的inset。

```
	- (void)viewDidLoad {
	   [super viewDidLoad];
	   self.title = @"我是导航条"; 
	   self.view.backgroundColor = [UIColor redColor];

	   UITableView *testTableView = [[UITableView alloc] initWithFrame:CGRectMake(0, 0, KScreenWidth, KScreenHeight) style:UITableViewStylePlain];
	   testTableView.backgroundColor = [UIColor blueColor];
	   testTableView.delegate = self;
	   testTableView.dataSource = self;

	   [self.view addSubview:testTableView];
	}

	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
	   UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"];
	   if (!cell) {
	       cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cell"];
	   }
	   cell.textLabel.text = [NSString stringWithFormat:@"%ld", (long)indexPath.row];
	   return cell;
	}

	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
	   return 20;
	}
```

<br/>

效果图：

<br/>

![iPhone分辨率](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a7.png)

![iPhone分辨率](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a10.png)

&emsp;	可以看到没有改变`tableView的frame`,只是显示范围变了。如果没有这个属性，我们要实现同样的效果，tableView尺寸要这样设置。当然手动修改insets也是可以的。

```
x = 0, 
y = KStatusBarAndNavigationBarHeight, 

width = KScreenWidth, 
height = KScreenHeight - KStatusBarAndNavigationBarHeight

```

`注意：` 这种自动调整是在ScrollView是其根视图添加的的第一个控件的时候,才会出现自动调整的效果。


<br/>


	- contentInsetAdjustmentBehavior
&emsp;	iOS11中废弃了automaticallyAdjustsScrollViewInsets，取而代之的是contentInsetAdjustmentBehavior属性。

&emsp;	该属性原理和automaticallyAdjustsScrollViewInsets原理相似，是为了进一步适应安全区域。



![iPhone分辨率](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a11.png)

如果你需要自定义内边距，代码将变成以下这样

```

if (@available(iOS 11.0, *)) {
        self.tableView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
} else {
        self.automaticallyAdjustsScrollViewInsets = NO;
}



```

-	以下是适配建议：
 -	 对于完全自定义页面的安全区域是个碍事的东西，把ScrollView（及子类）的contentInsetAdjustmentBehavior设置成Never，自己控制每一个细节。
		-	对于没有横屏需求的同学，系统默认的UIScrollViewContentInsetAdjustmentAutomatic是个好选择，就不用使用者自己修改了。
		-	对于有横屏需求的同学，建议使用UIScrollViewContentInsetAdjustmentAlways，横屏的时候不会被“刘海”干扰。



<br/>

***
<br/>


-	UI 在不同尺寸上适配

```

#define KScaleWidth(width) ((width)*(KScreenWidth/375.f))
#define IsIphone6P          SCREEN_WIDTH==414
#define SizeScale           (IsIphone6P ? 1.5 : 1)
#define kFontSize(value)    value*SizeScale
#define kFont(value)        [UIFont systemFontOfSize:value*SizeScale]

```


<br/>

***


-	UILabel的适配
&emsp;	开发过程中都遇到过字体UIFont适配问题.

	-	App如果要设置全局字体，可以通过Swizzing修改。或者像以上宏一样，传进参数，修改字体大小(淘宝在Plus机型的字体都加大成1.5倍)。
		-	宏定义适配字体大小（根据屏幕尺寸判断）

```
//宏定义
#define SCREEN_WIDTH ([UIScreen mainScreen].bounds.size.width)
#define FONT_SIZE(size) ([UIFont systemFontOfSize:FontSize(size))

//字体适配 我在PCH文件定义了一个方法
static inline CGFloat FontSize(CGFloat fontSize){
	Ï
    if (SCREEN_WIDTH==320) {
        return fontSize-2;
    }else if (SCREEN_WIDTH==375){
        return fontSize;
    }else{
        return fontSize+2;
    }
}
```

或者：

```
#define IsIphone6P          SCREEN_WIDTH==414
#define SizeScale           (IsIphone6P ? 1.5 : 1)
#define kFontSize(value)    value*SizeScale
#define kFont(value)        [UIFont systemFontOfSize:value*SizeScale]
```

- runTime给UIFont写分类 替换系统自带的方法

```
class_getInstanceMethod得到类的实例方法
class_getClassMethod得到类的类方法

1. 首先需要创建一个UIFont的分类
2. 自己UI设计原型图的手机尺寸宽度
#define MyUIScreen  375 // UI设计原型图的手机尺寸宽度(6), 6p的--414
```

```
UIFont+runtime.m

#import "UIFont+runtime.h"
#import <objc/runtime.h>
@implementation UIFont (runtime)

+ (void)load {
    // 获取替换后的类方法
    Method newMethod = class_getClassMethod([self class], @selector(adjustFont:));
    // 获取替换前的类方法
    Method method = class_getClassMethod([self class], @selector(systemFontOfSize:));
    // 然后交换类方法，交换两个方法的IMP指针，(IMP代表了方法的具体的实现）
    method_exchangeImplementations(newMethod, method);
}

+ (UIFont *)adjustFont:(CGFloat)fontSize {
    UIFont *newFont = nil;
    newFont = [UIFont adjustFont:fontSize * [UIScreen mainScreen].bounds.size.width/MyUIScreen];
    return newFont;
}
@end
```

外部调用：

```
//Controller类中正常调用就行了：

UILabel *label = [[UILabel alloc]initWithFrame:CGRectMake(0, 150, [UIScreen mainScreen].bounds.size.width, 60)];
label.text = @"iOS字体大小适配";
label.font = [UIFont systemFontOfSize:16];
[self.view addSubview:label];

```
		
-	注意：
	-	load方法只会走一次，利用runtime的method进行方法的替换
	-	替换的方法里面（把系统的方法替换成我们自己写的方法），这里要记住写自己的方法，不然会死循环
	-	之后凡是用到systemFontOfSize方法的地方，都会被替换成我们自己的方法，即可改字体大小了
	-	注意：此方法只能替换 纯代码 写的控件字号，如果你用xib创建的控件且在xib里面设置的字号，那么替换不了！你需要在xib的
awakeFromNib方法里面手动设置下控件字体




***
<br/>

># 参考资料：
* [iPhone X适配实战总结](https://www.dazhuanlan.com/2019/09/23/5d8885965922d/?__cf_chl_jschl_tk__=07a284263e36412b3125579256be07b9ac748989-1599791161-0-Aey9NTMS0sUtBDxemVxMkfQcbyDkuVjk3u4CgEtLieaHYGSP2TpZH0NaOBRI4qWdquNM5V963-uDKo51ZQyL3pA7VPP_l0Dm-VnBU3UVNNJYKrz4Zi8XFFu1eQs5K4N2aUFuTNYjul50ibni_qCMWwunC1jdUZne5Csg6BMgK0VacTWmNh8TyWTdJA6MReA38h_tUrHi38HHfZvSvBqeLoCnZyBAEQGI9CSu8oft_0TDlZ1YKmQuLRNWGukBP-TQxkMiO0zhauesSRoMsuRbPU3kCKfF-SGFefqr8IDYB-pNG05bSLle5FDnSdY1kzE2ZA)

























