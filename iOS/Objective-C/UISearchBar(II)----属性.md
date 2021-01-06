##基础代码
```
- (void)testSearchBar {
    UISearchBar *search = [[UISearchBar alloc] initWithFrame:CGRectMake(0, HGSTATUS_FRAME_HEIGHT, HGSCREEN_WIDTH, 45.0f)];
        
    [self.view addSubview:search];
}

```
>#属性
***`搜索框风格 后两种已被禁用`***
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
***
<br/>

***`搜索框占位符`***
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
***
<br/>

***`搜索框右边的按钮风格`***
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
***
<br/>

***`搜索框 颜色、背景颜色`***
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
***
<br/>

***`搜索框 样式`***
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

![search.searchBarStyle = UISearchBarStyleProminent;
](https://upload-images.jianshu.io/upload_images/2959789-f79bb26420f31993.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![search.searchBarStyle = UISearchBarStyleMinimal;  没有背景](https://upload-images.jianshu.io/upload_images/2959789-29899568affe8414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>

***`搜索框 附件选择按钮视图`***
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
***
<br/>

***`搜索框背景图片、搜索框附属分栏条的背景颜色`***
```
// 搜索框背景图片
@property(nullable, nonatomic,strong) UIImage *backgroundImage;
// 搜索框附属分栏条的背景颜色
@property(nullable, nonatomic,strong) UIImage *scopeBarBackgroundImage;
```



<br/>
***
<br/>

***`搜索框 背景图片和附属栏背景图片设置`***

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
***
<br/>

***`搜索框 文本框的背景偏移量和文本偏移量`***

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
