
- **属性**
	- 搜索框风格
	- 搜索框占位符
	- 搜索框右边的按钮风格
	- 搜索框颜色、背景颜色
	- 搜索框 样式
	- 搜索框附件选择按钮视图
	- 搜索框 背景图片和附属栏背景图片设置
	- 搜索框 文本框的背景偏移量和文本偏移量
- **方法**
	- 设置是否动画效果的显示或隐藏取消按钮
	- 设置 (获取) 搜索框背景图片、选择按钮视图的背景图片、文本框的背景图片
	- 设置（获取）搜索框的图标
	- 设置（获取）选择按钮视图的分割线图片、按钮上显示的标题样式
	- 搜索框四种图标的偏移量
	- UISearchBarDelegate代理方法
	- UISearchBar 控件组成图
	- [**UISearchBar 属性、方法详解及应用**](http://www.code4app.com/blog-822724-1543.html)
	- Demo

<br/>

***
<br/>

># 属性

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 45.0f)];
        
    [self.view addSubview:search];
}

```


<br/>

- ***`搜索框风格 后两种已被禁用`***

```
typedef NS_ENUM(NSInteger, UIBarStyle) {
    UIBarStyleDefault         //白色搜索框，灰色背景
    UIBarStyleBlack           //黑色搜索框,
     
    UIBarStyleBlackOpaque      = 1, // 禁用. Use UIBarStyleBlack
    UIBarStyleBlackTranslucent = 2, // 禁用. Use UIBarStyleBlack and set the translucent property to YES
}
```

![search.barStyle = UIBarStyleBlack;](https://upload-images.jianshu.io/upload_images/2959789-f27dc5d114fec927.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![search.barStyle = UIBarStyleDefault;](https://upload-images.jianshu.io/upload_images/2959789-db6311e873996ff0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

- ***`搜索框占位符`***

```
// 搜索的文本
@property(nullable,nonatomic,copy)   NSString *text;  
 // 搜索框顶部的提示信息 
@property(nullable,nonatomic,copy)   NSString *prompt;    
// 占位符，默认nil, 若有值则在输入文本后消失
@property(nullable,nonatomic,copy)   NSString *placeholder;
```

`Demo`

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.prompt = @"search‘s prompt";
    search.placeholder = @"search's placeholder";
    
    
    [self.view addSubview:search];
}
```

![效果图](https://upload-images.jianshu.io/upload_images/2959789-29d7d522598357dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

- ***`搜索框右边的按钮风格`***

```
//搜索框右侧是否显示图书按钮
@property(nonatomic)  BOOL   showsBookmarkButton __TVOS_PROHIBITED;   // default is NO
//搜索框右侧是否显示取消按钮
@property(nonatomic)  BOOL   showsCancelButton __TVOS_PROHIBITED;     // default is NO
//搜索框右侧是否显示搜索结果按钮
@property(nonatomic)  BOOL   showsSearchResultsButton NS_AVAILABLE_IOS(3_2) __TVOS_PROHIBITED; // default is NO
@property(nonatomic, getter=isSearchResultsButtonSelected) BOOL searchResultsButtonSelected NS_AVAILABLE_IOS(3_2) __TVOS_PROHIBITED;
```

![search.showsBookmarkButton = YES;  效果](https://upload-images.jianshu.io/upload_images/2959789-089cae87c1ef5e9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![search.showsCancelButton = YES;  效果](https://upload-images.jianshu.io/upload_images/2959789-461e07188575e35e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![    search.showsSearchResultsButton = YES;
](https://upload-images.jianshu.io/upload_images/2959789-42fe847569b1268b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![search.showsSearchResultsButton = YES;
    search.searchResultsButtonSelected = YES;  都要设置](https://upload-images.jianshu.io/upload_images/2959789-3e827d5202f053fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- ***`搜索框颜色、背景颜色`***

```
// //设置光标颜色
@property(null_resettable, nonatomic,strong) UIColor *tintColor;
//  //搜索框背景颜色
@property(nullable, nonatomic,strong) UIColor *barTintColor;
```
```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    //设置光标颜色
    search.tintColor = [UIColor redColor];
    //搜索框背景颜色
    search.barTintColor = [UIColor greenColor];
    
    
    [self.view addSubview:search];
}
```
![背景颜色和光标颜色改变](https://upload-images.jianshu.io/upload_images/2959789-e0999b418c4e13e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
<br/>

- ***`搜索框样式`***

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    
    [self.view addSubview:search];
}

```

```
//搜索框样式
@property (nonatomic) UISearchBarStyle searchBarStyle;

typedef NS_ENUM(NSUInteger, UISearchBarStyle) {
    //默认样式和UISearchBarStyleProminent效果一样
    UISearchBarStyleDefault,    // currently UISearchBarStyleProminent
    UISearchBarStyleProminent,  // used my Mail, Messages and Contacts
    //没有背景
    UISearchBarStyleMinimal     // used by Calendar, Notes and Music
} 
```

![search.searchBarStyle = UISearchBarStyleDefault; 效果](https://upload-images.jianshu.io/upload_images/2959789-954002d83dcb28e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

![search.searchBarStyle = UISearchBarStyleProminent;
](https://upload-images.jianshu.io/upload_images/2959789-f79bb26420f31993.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

![search.searchBarStyle = UISearchBarStyleMinimal;  没有背景](https://upload-images.jianshu.io/upload_images/2959789-29899568affe8414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
<br/>

- ***`搜索框附件选择按钮视图`***

```
//选择按钮试图的按钮标题
@property(nullable, nonatomic,copy) NSArray<NSString *>   *scopeButtonTitles ; 
//选中的按钮下标值 ，默认 0. 如果超出范围则忽略
@property(nonatomic) NSInteger  selectedScopeButtonIndex ;
//是否显示搜索栏的附件选择按钮视图
@property(nonatomic) BOOL     showsScopeBar;
```

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.showsScopeBar = YES;
    search.selectedScopeButtonIndex = 1;
    search.scopeButtonTitles = @[@"One",@"Two",@"Three"];
    
    
    [self.view addSubview:search];
}
```

效果图
![效果图](https://upload-images.jianshu.io/upload_images/2959789-3f039a154fd58491.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- ***`搜索框背景图片、搜索框附属分栏条的背景颜色`***

```
// 搜索框背景图片
@property(nullable, nonatomic,strong) UIImage *backgroundImage;
// 搜索框附属分栏条的背景颜色
@property(nullable, nonatomic,strong) UIImage *scopeBarBackgroundImage;
```



<br/>
<br/>

- ***`搜索框 背景图片和附属栏背景图片设置`***

```
// 搜索框背景图片
@property(nullable, nonatomic,strong) UIImage *backgroundImage;
// 搜索框附属分栏条的背景颜色
@property(nullable, nonatomic,strong) UIImage *scopeBarBackgroundImage;
```

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.showsScopeBar = YES;
    search.selectedScopeButtonIndex = 1;
    search.scopeButtonTitles = @[@"One",@"Two",@"Three"];
    
    UIImage *image = [UIImage imageNamed: @"icon"];
    search.backgroundImage = image;
    search.scopeBarBackgroundImage = [UIImage imageNamed:@"my_service"];
    
    
    [self.view addSubview:search];
}
```
![背景图设置](https://upload-images.jianshu.io/upload_images/2959789-8d28418721b2abfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图中可以可以看到，搜索栏无法设置背景图片但是附属栏是可以的，`待解决`


<br/>
<br/>

- ***`搜索框 文本框的背景偏移量和文本偏移量`***

```
// 搜索框中文本框的背景偏移量
@property(nonatomic) UIOffset searchFieldBackgroundPositionAdjustment;
// 搜索框中文本框的文本偏移量
@property(nonatomic) UIOffset searchTextPositionAdjustment;
```

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.searchFieldBackgroundPositionAdjustment = UIOffsetMake(5, 3);
    search.searchTextPositionAdjustment = UIOffsetMake(10, 5);
    
    
    [self.view addSubview:search];
}
```

![偏移量进行了修改](https://upload-images.jianshu.io/upload_images/2959789-de569ff7e3f960e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![默认的样式](https://upload-images.jianshu.io/upload_images/2959789-7750678cd35e5a1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/>

># 方法

<br/>

- **设置是否动画效果的显示或隐藏取消按钮**

```
// 设置是否动画效果的显示或隐藏取消按钮
- (void)setShowsCancelButton:(BOOL)showsCancelButton animated:(BOOL)animated
```

Demo

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.delegate = self;
    
    
    [self.view addSubview:search];
}

- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar {
    //动画取消按钮
    [searchBar setShowsCancelButton:YES animated:YES];
}
```

![开始编辑时，出现取消按钮](https://upload-images.jianshu.io/upload_images/2959789-96cddc6713dc153e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- ***`设置 (获取) 搜索框背景图片、选择按钮视图的背景图片、文本框的背景图片`***

```
// 1.设置搜索框背景图片
- (void)setBackgroundImage:(nullable UIImage *)backgroundImage forBarPosition:(UIBarPosition)barPosition barMetrics:(UIBarMetrics)barMetrics 
//  获取置搜索框背景图片
- (nullable UIImage *)backgroundImageForBarPosition:(UIBarPosition)barPosition barMetrics:(UIBarMetrics)barMetrics 
// 2.设置选择按钮视图的背景图片
- (void)setScopeBarButtonBackgroundImage:(nullable UIImage *)backgroundImage forState:(UIControlState)state 
// 获取选择按钮视图的背景图片
- (nullable UIImage *)scopeBarButtonBackgroundImageForState:(UIControlState)state  
// 3.设置搜索框文本框的背景图片
- (void)setSearchFieldBackgroundImage:(nullable UIImage *)backgroundImage forState:(UIControlState)state 
// 获取搜索框文本框的背景图片
- (nullable UIImage *)searchFieldBackgroundImageForState:(UIControlState)state
```

Demo

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.showsScopeBar = YES;
    search.scopeButtonTitles = @[@"王者荣耀", @"英雄联盟", @"淘宝联盟"];
    search.selectedScopeButtonIndex = 1;
    //设置搜索框背景图片，但是没有效果
    [search setBackgroundImage:[UIImage imageNamed:@"icon"] forBarPosition:UIBarPositionAny barMetrics:UIBarMetricsDefault];
// 设置选择按钮视图的背景图片
    [search  setScopeBarButtonBackgroundImage:[UIImage imageNamed:@"icon"] forState:UIControlStateNormal];
// 设置搜索框文本框的背景图片
    [search setSearchFieldBackgroundImage:[UIImage imageNamed:@"tab_my_nor"] forState:UIControlStateNormal];
    
    search.delegate = self;
    
    
    [self.view addSubview:search];
}
```

![背景图设置](https://upload-images.jianshu.io/upload_images/2959789-c6ec5d0456335012.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- ***`设置（获取）搜索框的图标（包括搜索图标、清除输入的文字的图标、图书图标、搜索结果列表图标）    `***

```
// 设置搜索框的图标
- (void)setImage:(nullable UIImage *)iconImage forSearchBarIcon:(UISearchBarIcon)icon state:(UIControlState)state;
// 获取搜索框的图标
- (nullable UIImage *)imageForSearchBarIcon:(UISearchBarIcon)icon state:(UIControlState)state;


 / / 搜索框图标 UISearchBarIcon 有四个枚举值：
typedef NS_ENUM(NSInteger, UISearchBarIcon) {
    UISearchBarIconSearch, // 搜索框放大镜图标
    UISearchBarIconClear , // 搜索框右侧可用于清除输入的文字的图标x
    UISearchBarIconBookmark , // 搜索框右侧的图书图标
    UISearchBarIconResultsList , // 搜索框右侧的搜索结果列表图标
};
```


Demo

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.showsSearchResultsButton = YES;
    UIImage *searchIcon = [UIImage imageNamed:@"mine_searchFriend"];
     // 设置搜索框放大镜图标
    [search setImage:searchIcon forSearchBarIcon:UISearchBarIconSearch state:UIControlStateNormal];
    // 设置搜索结果列表图标
    [search setImage:searchIcon forSearchBarIcon:UISearchBarIconResultsList state:UIControlStateNormal];
    
    search.delegate = self;
    
    
    [self.view addSubview:search];
}
```
 
![搜索图标和类表图标设置](https://upload-images.jianshu.io/upload_images/2959789-4a2df5b75b74ce6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


- ***`设置（获取）选择按钮视图的分割线图片、按钮上显示的标题样式 `***

```
// 设置选择按钮视图的分割线图片
- (void)setScopeBarButtonDividerImage:(nullable UIImage *)dividerImage forLeftSegmentState:(UIControlState)leftState rightSegmentState:(UIControlState)rightState;
// 获取选择按钮视图的分割线图片
- (nullable UIImage *)scopeBarButtonDividerImageForLeftSegmentState:(UIControlState)leftState rightSegmentState:(UIControlState)rightState;
// 设置选择按钮视图的标题样式
- (void)setScopeBarButtonTitleTextAttributes:(nullable NSDictionary *)attributes forState:(UIControlState)state;
// 获取选择按钮视图的标题样式
- (nullable NSDictionary *)scopeBarButtonTitleTextAttributesForState:(UIControlState)state
```

Demo

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.showsScopeBar = YES;
    search.scopeButtonTitles = @[@"One", @"Two", @"Three", @"Four"];
    // 设置选择按钮视图的分割线图片为一个绿色的线条
    [search setScopeBarButtonDividerImage:[UIImage imageNamed:@"1"] forLeftSegmentState:UIControlStateNormal rightSegmentState:UIControlStateNormal];
    // 设置选择按钮视图的标题样式为20号橘色空心的系统字体
    NSDictionary *attributeDic = @{NSFontAttributeName : [UIFont systemFontOfSize:20] , NSStrokeColorAttributeName : [UIColor orangeColor] , NSStrokeWidthAttributeName : @(3)};
    [search setScopeBarButtonTitleTextAttributes:attributeDic forState:UIControlStateNormal];
    
    
    [self.view addSubview:search];
}
```

![选择图片设置](https://upload-images.jianshu.io/upload_images/2959789-0900ed073b39d3a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


- ***`搜索框四种图标的偏移量`***

```
//  设置搜索框图标的偏移量
- (void)setPositionAdjustment:(UIOffset)adjustment forSearchBarIcon:(UISearchBarIcon)icon;
//  获取搜索框图标的偏移量
- (UIOffset)positionAdjustmentForSearchBarIcon:(UISearchBarIcon)icon;
```

Demo
![图标和placeholder 右移](https://upload-images.jianshu.io/upload_images/2959789-e09e048a7b75038e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![palceholder 右移85个单位](https://upload-images.jianshu.io/upload_images/2959789-ac55c165aba5e839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- ***UISearchBarDelegate代理方法***

```
// 1. 将要开始编辑文本时会调用该方法，返回 NO 将不会变成第一响应者
- (BOOL)searchBarShouldBeginEditing:(UISearchBar *)searchBar;                      
// 2. 开始输入文本会调用该方法
- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar;                     
// 3. 将要结束编辑文本时会调用该方法，返回 NO 将不会释放第一响应者
- (BOOL)searchBarShouldEndEditing:(UISearchBar *)searchBar; 
 // 4. 结束编辑文本时调用该方法
- (void)searchBarTextDidEndEditing:(UISearchBar *)searchBar;     
// 5. 文本改变会调用该方法（包含clear文本）
- (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText; 
// 6. 文字改变前会调用该方法，返回NO则不能加入新的编辑文字
- (BOOL)searchBar:(UISearchBar *)searchBar shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text ;
// 7. 键盘上的搜索按钮点击的会调用该方法
- (void)searchBarSearchButtonClicked:(UISearchBar *)searchBar;     
// 8. 搜索框右侧图书按钮点击会调用该方法
- (void)searchBarBookmarkButtonClicked:(UISearchBar *)searchBar ;
 // 9.点击取消按钮会调用该方法
- (void)searchBarCancelButtonClicked:(UISearchBar *)searchBar;   
// 10.搜索结果列表按钮被按下会调用该方法
- (void)searchBarResultsListButtonClicked:(UISearchBar *)searchBar ; 
// 11. 搜索框的附属按钮视图中切换按钮会调用该方法
- (void)searchBar:(UISearchBar *)searchBar selectedScopeButtonIndexDidChange:(NSInteger)selectedScope;
```


Demo

```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 100.0f)];
    search.barStyle = UIBarStyleDefault;
    search.placeholder = @"search's placeholder";
    
    search.delegate = self;
    
    
    [self.view addSubview:search];
}

- (BOOL)searchBarShouldBeginEditing:(UISearchBar *)searchBar {
    NSLog(@"++++++> searchBarShouldBeginEditing:");
    return YES;
}
- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar {
    NSLog(@"----> searchBarTextDidBeginEditing:");
    searchBar.prompt = @"1.开始编辑文本";
    [searchBar setShowsCancelButton:YES animated:YES]; // 动画效果显示取消按钮
}
- (BOOL)searchBar:(UISearchBar *)searchBar shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text {
    NSLog(@"*****> searchBar:   shouldChangeTextInRange:");

    return YES;
}
- (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText {
    NSLog(@"~~~~~~> searchBar:  textDidChange:");
    searchBar.prompt = @"2.在改变文本过程中。。。";
}
- (void)searchBarSearchButtonClicked:(UISearchBar *)searchBar {
    NSLog(@"@@@@@@@> searchBarSearchButtonClicked:");
    searchBar.prompt = @"3. 点击键盘上的搜索按钮";
}
- (void)searchBarCancelButtonClicked:(UISearchBar *)searchBar {
    NSLog(@"#######> searchBarCancelButtonClicked:");
    searchBar.prompt = @"4. 点击取消按钮";
    searchBar.text = @"";
    [searchBar setShowsCancelButton:NO animated:YES];
    // 如果希望在点击取消按钮调用结束编辑方法需要让加上这句代码
    //[searchBar resignFirstResponder];
}
- (BOOL)searchBarShouldEndEditing:(UISearchBar *)searchBar{
    NSLog(@"%%%%%%% > searchBarShouldEndEditing:");

    return YES;
}
- (void)searchBarTextDidEndEditing:(UISearchBar *)searchBar {
    NSLog(@"^^^^^^> searchBarTextDidEndEditing:");

    searchBar.prompt = @"5.已经结束编辑文本";
}
```

日志打印：

```
2018-11-06 11:07:01.450255+0800 HGSWB[2947:144222] ++++++> searchBarShouldBeginEditing:
2018-11-06 11:07:01.463032+0800 HGSWB[2947:144222] ----> searchBarTextDidBeginEditing:
2018-11-06 11:07:01.625344+0800 HGSWB[2947:144222] [MC] System group container for systemgroup.com.apple.configurationprofiles path is /Users/huanggang/Library/Developer/CoreSimulator/Devices/8B094E91-9EC5-4BDE-8709-789579D85552/data/Containers/Shared/SystemGroup/systemgroup.com.apple.configurationprofiles
2018-11-06 11:07:01.628250+0800 HGSWB[2947:144222] [MC] Reading from private effective user settings.
2018-11-06 11:07:18.061102+0800 HGSWB[2947:144222] *****> searchBar:   shouldChangeTextInRange:
2018-11-06 11:07:18.091087+0800 HGSWB[2947:144222] ~~~~~~> searchBar:  textDidChange:
2018-11-06 11:07:24.105030+0800 HGSWB[2947:144222] ~~~~~~> searchBar:  textDidChange:
2018-11-06 11:07:27.406004+0800 HGSWB[2947:144222] #######> searchBarCancelButtonClicked:
2018-11-06 11:07:35.851062+0800 HGSWB[2947:144222] %%% > searchBarShouldEndEditing:
2018-11-06 11:07:35.861178+0800 HGSWB[2947:144222] ^^^^^^> searchBarTextDidEndEditing:

```

![代理方法效果图](https://upload-images.jianshu.io/upload_images/2959789-17bfed3255cb3fb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

- **UISearchBar 控件组成图**

![UISearchBar 控件组成图](https://upload-images.jianshu.io/upload_images/2959789-eb75d2c6a1db34fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据截图可以获得如下：
![UISearchBar层次结构图](https://upload-images.jianshu.io/upload_images/2959789-ea5d29abc770b5db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`目标样式图`
![1511317665958305.gif](https://upload-images.jianshu.io/upload_images/2959789-825d9a078371f1e5.gif?imageMogr2/auto-orient/strip)

***`实现大致代码`***

```
// 委托
- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar {
    [searchBar setShowsCancelButton:YES animated:YES];
}
- (void)searchBarCancelButtonClicked:(UISearchBar *)searchBar {
    searchBar.text = @"";
    [self setShowsCancelButton:NO animated:YES];
    [searchBar resignFirstResponder];
}
- (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText {
    self.voiceButton.hidden = searchText.length > 0 ? YES : NO;
}
- (void)searchBarTextDidEndEditing:(UISearchBar *)searchBar {
    self.voiceButton.hidden = NO;
}
// 样式设计
- (void)modifyStyleByTraversal {
    // 修改搜索框背景图片为自定义的灰色
    UIColor *backgroundImageColor = [UIColor colorwithHex:0xE3DFDA];
    UIImage* clearImg = [self imageWithColor:backgroundImageColor andHeight:45.0];
    self.backgroundImage = clearImg;
    // 修改搜索框光标颜色
    self.tintColor = [UIColor P2Color];
    // 修改搜索框的搜索图标
    [self setImage:[UIImage imageNamed:@"searchIcon"] forSearchBarIcon:UISearchBarIconSearch state:UIControlStateNormal];
    for (UIView *view in self.subviews.lastObject.subviews) {
        if([view isKindOfClass:NSClassFromString(@"UISearchBarTextField")]) {
            UITextField *textField = (UITextField *)view;
           //添加话筒按钮
            [self addVoiceButton:textField];
            //设置输入框的背景颜色
            textField.clipsToBounds = YES;
            textField.backgroundColor = [UIColor P1Color];
            //设置输入框边框的圆角以及颜色
            textField.layer.cornerRadius = 10.0f;
            textField.layer.borderColor = [UIColor P2Color].CGColor;
            textField.layer.borderWidth = 1;
            //设置输入字体颜色
            textField.textColor = [UIColor P2Color];
            //设置默认文字颜色
            textField.attributedPlaceholder = [[NSAttributedString alloc] initWithString:@" 搜索" attributes:@{NSForegroundColorAttributeName:[UIColor P2Color]}];
        }
        if ([view isKindOfClass:NSClassFromString(@"UINavigationButton")]) {
            UIButton *cancel = (UIButton *)view;
            [cancel setTitle:@"取消" forState:UIControlStateNormal];
            [cancel setTitleColor:[UIColor P2Color] forState:UIControlStateNormal];
        }
    }
}
// 画指定高度和颜色的图片
- (UIImage*) imageWithColor:(UIColor*)color andHeight:(CGFloat)height {
    CGRect rect = CGRectMake(0.0, 0.0, 1.0, height);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
// 添加话筒按钮
- (void)addVoiceButton:(UIView *)view {
    if (!self.voiceButton) {
        self.voiceButton = [UIButton buttonWithType:UIButtonTypeCustom];
    }
    [self.voiceButton setImage:[UIImage imageNamed:@"voice"] forState:UIControlStateNormal];
    [self.voiceButton addTarget:self action:@selector(say) forControlEvents:UIControlEventTouchUpInside];
    [view addSubview:self.voiceButton];
    self.voiceButton.translatesAutoresizingMaskIntoConstraints = NO;
    [self.voiceButton addWidthConstraint:15];
    [self.voiceButton addHeightConstraint:15];
    [self.voiceButton addTopConstraintToView:view constant:8];
    [self.voiceButton addTrailingConstraintToView:view constant:- 15];
}
// 点击话筒触发该方法
- (void)say {
  //  self.prompt = @"在讲话哦！";
}
```

<br/>
<br/>

- **修改UISearchBar宽高**

`UISearchBar 有默认的布局样式，我们想要修改他的宽高，需要重写layoutSubviews 方法。然后在这个方法里面修改UISearchBarTextField的宽高。`

```

- (void)layoutSubviews {
    [super layoutSubviews];
    UITextField *textField = [self valueForKey:@"searchBarTextField"];
    if (textField) {
        textField.frame = CGRectMake(50, 10, self.bounds.size.width - 100, 35);
    }
}
```
效果图：
![效果图](https://upload-images.jianshu.io/upload_images/2959789-e112e378353efd92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


- **Demo**

`HGSearchBar.h`

```
#import "HGSearchBar.h"

@interface HGSearchBar()<UITextFieldDelegate>

//placeholder和icon之间的间隙整体宽度
@property(nonatomic, assign) CGFloat placeholderWidth;

@end

//icon宽度
static CGFloat const searchIconW = 20.0f;
//icon与placeholder间距
static CGFloat const iconSpacing = 10.0f;
//占位文字的字体大小
static CGFloat const placeHolderaFont = 15.0f;

@implementation HGSearchBar


- (void)layoutSubviews {
    [super layoutSubviews];
    
    //设置背景图片
    UIImage *backImage = [UIImage imageWithColor:[UIColor colorWithRed:239/255.0f green:239/255.0f blue:239/255.0f alpha:1.0f]];
    [self setBackgroundImage:backImage];
    
    for (UIView *view in [self.subviews lastObject].subviews) {
        if ([view isKindOfClass:[UITextField class]]) {
            UITextField *field = (UITextField *)view;
            //设置field的frame
            field.frame = CGRectMake(15.0f, 7.5f, self.frame.size.width - 30.0f, self.frame.size.height -  15.0f);
            [field setBackgroundColor:[UIColor whiteColor]];
            field.textColor = [UIColor colorWithRed:51/255.0f green:51/255.0f blue:51/255.0f alpha:1.0f];
            field.borderStyle = UITextBorderStyleNone;
            field.layer.cornerRadius = 2.0f;
            field.layer.masksToBounds = YES;
            
            //设置占位符文字的颜色
            [field setValue:[UIColor colorWithRed:156/255.0 green:156/255.0 blue:156/255.0 alpha:1.0] forKeyPath:@"_placeholderLabel.textColor"];
            [field setValue:[UIFont systemFontOfSize:placeHolderaFont] forKeyPath:@"_placeholderLabel.font"];
            
            if (@available(iOS 11.0, *)) {
                //设置placeholder的位置，为居中(设置搜索框图标的偏移量)
                [self setPositionAdjustment:UIOffsetMake((field.frame.size.width - self.placeholderWidth)/2.0, 0) forSearchBarIcon:UISearchBarIconSearch];
            }
        }
    }
}

//开始编辑时重置为靠左
- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField {
    //继续传递代理方法
    if ([self.delegate respondsToSelector:@selector(searchBarShouldBeginEditing:)]) {
        [self.delegate searchBarShouldBeginEditing:self];
    }
    
    //available 用于声明时，表示该声明的生命周期与特定的平台和操作系统版本有关。
    if (@available(iOS 11.0, *)) {
        [self setPositionAdjustment:UIOffsetZero forSearchBarIcon:UISearchBarIconSearch];
    }
    
    return  YES;
}


- (BOOL)textFieldShouldEndEditing:(UITextField *)textField {
    if ([self.delegate respondsToSelector:@selector(searchBarShouldEndEditing:)]) {
        [self.delegate searchBarShouldEndEditing:self];
    }
    
    if (@available(iOS 11.0, *)) {
        [self setPositionAdjustment:UIOffsetMake((textField.frame.size.width - self.placeholderWidth)/2, 0) forSearchBarIcon:UISearchBarIconSearch];
    }
    
    return YES;
}


//计算placeholder、icon、icon和placeholder间距的总宽度
- (CGFloat)placeholderWidth {
    if (!_placeholderWidth) {
        //返回文本绘制所占据的矩形空间
        CGSize size = [self.placeholder boundingRectWithSize:CGSizeMake(MAXFLOAT, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin|NSStringDrawingUsesFontLeading attributes:@{NSFontAttributeName:[UIFont systemFontOfSize:placeHolderaFont]} context:nil].size;
        _placeholderWidth = size.width + iconSpacing + searchIconW;
    }
    return _placeholderWidth;
}

@end
```

`UIImage扩展类`

`UIImage+Tool.h`

```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UIImage (Tool)

+(UIImage *) imageWithColor:(UIColor *) color;

@end

NS_ASSUME_NONNULL_END

```

`UIImage+Tool.m`

```
#import "UIImage+Tool.h"

@implementation UIImage (Tool)

+ (UIImage *)imageWithColor:(UIColor *)color {
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    //创建一个基于位图的上下文（context）,并将其设置为当前上下文(context)。
    UIGraphicsBeginImageContext(rect.size);
    //获取上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    //设置填充颜色
    CGContextSetFillColorWithColor(context, [color CGColor]);
    //填充矩形
    CGContextFillRect(context, rect);
    //获取图片
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    //结束上下文
    UIGraphicsEndImageContext();
    return theImage;
}

@end

```

`ViewController.h`

```
- (HGSearchBar *)searchBar {
    if (!_searchBar) {
        _searchBar = [[HGSearchBar alloc] initWithFrame:CGRectMake(0, 100, self.view.bounds.size.width, 45.0f)];
        _searchBar.placeholder = @"喜欢的视频";
    }
    return  _searchBar;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self layoutViews];
}

- (void)layoutViews {
    
    [self.view addSubview:self.searchBar];
    [self.searchBar mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.and.right.equalTo(self.view);
        make.top.equalTo(self.view).offset(HGSTATUS_FRAME_HEIGHT);
        make.height.equalTo(@45.0f);
    }];
    
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapAcion:)];
    [self.view addGestureRecognizer:tap];
    
}

- (void) tapAcion:(UITapGestureRecognizer *)gesture {
    //促使UITextFild放弃第一响者
    [self.view endEditing:YES];
}
```

![效果图](https://upload-images.jianshu.io/upload_images/2959789-3756b3c8917dc0d3.gif?imageMogr2/auto-orient/strip)





