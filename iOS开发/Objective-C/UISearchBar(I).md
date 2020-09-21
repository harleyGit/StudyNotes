>#直接上代码
`HGSearchBar.h`
```
//
//  HGSearchBar.m
//  HGSWB
//
//  Created by Harley on 2018/11/3.
//  Copyright © 2018 Harley'sMac. All rights reserved.
//

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

