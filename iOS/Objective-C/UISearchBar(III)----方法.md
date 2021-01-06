>#设置是否动画效果的显示或隐藏取消按钮
```// 设置是否动画效果的显示或隐藏取消按钮
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
***
<br/>
***`设置 (获取) 搜索框背景图片、选择按钮视图的背景图片、文本框的背景图片`***

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
***
<br/>
***`设置（获取）搜索框的图标（包括搜索图标、清除输入的文字的图标、图书图标、搜索结果列表图标）    `***

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
***
<br/>
***`设置（获取）选择按钮视图的分割线图片、按钮上显示的标题样式 `***

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
***
<br/>
***`搜索框四种图标的偏移量`***

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
***
<br/>
>#***UISearchBarDelegate代理方法***
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
***
<br/>
>#***UISearchBar 控件组成图***
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

##修改UISearchBar宽高

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


`参考资料`：[UISearchBar 属性、方法详解及应用](http://www.code4app.com/blog-822724-1543.html)
