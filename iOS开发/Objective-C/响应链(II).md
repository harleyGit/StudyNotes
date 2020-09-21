># 扩大UIButton的响应区
&emsp;  通过重载UIButton的 `-(BOOL) pointInside: withEvent: `方法，让Point即使落在Button的Frame外围也返回YES。

创建一个UIButton的分类：

#`UIButton+ ExtendResponseArea.h 文件`
```
#import <UIKit/UIKit.h>

@interface UIButton (ExtendResponseArea)

@property (nonatomic) CGFloat responseAreaWidth;
@property (nonatomic) CGFloat responseAreaHeight;

@end
```

#`UIButton+ ExtendResponseArea.m 文件`
```
#import "UIButton+ ExtendResponseArea.h"
#import <objc/runtime.h>

@implementation UIButton (ExtendResponseArea)


- (CGFloat)minHitTestWidth {
    NSNumber * width = objc_getAssociatedObject(self, @selector(responseAreaWidth));
    return [width floatValue];
}

- (void)setMinHitTestWidth:(CGFloat) responseAreaWidth {
    objc_setAssociatedObject(self, @selector(responseAreaWidth), [NSNumber numberWithFloat: responseAreaWidth], OBJC_ASSOCIATION_ASSIGN);
}

- (CGFloat)minHitTestHeight {
    NSNumber * height = objc_getAssociatedObject(self, @selector(responseAreaHeight));
    return [height floatValue];
}

- (void)setMinHitTestHeight:(CGFloat) responseAreaHeight {
    objc_setAssociatedObject(self, @selector(responseAreaHeight), [NSNumber numberWithFloat: responseAreaHeight], OBJC_ASSOCIATION_ASSIGN);
}

- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event {
    
    return CGRectContainsPoint(NewBounds(self.bounds, self. responseAreaWidth, self. responseAreaHeight), point);
}

CGRect NewBounds(CGRect bounds, CGFloat responseAreaWidth, CGFloat responseAreaHeight) {
    
    CGRect clickBounds = bounds;
    if (responseAreaWidth > bounds.size.width) {
        clickBounds.size.width = responseAreaWidth;
        clickBounds.origin.x -= (responseAreaWidth.size.width - bounds.size.width)/2;
    }
    if (responseAreaHeight > bounds.size.height) {
        clickBounds.size.height = responseAreaHeight;
        clickBounds.origin.y -= (responseAreaHeight.size.height - bounds.size.height)/2;
    }
    return hitTestingBounds;
}

@end

```

调用
```
    UIButton * button = [UIButton buttonWithType:UIButtonTypeCustom];
    button.frame = CGRectMake(100, 100, 20, 20);
    button.backgroundColor = [UIColor yellowColor];
    [button addTarget:self action:@selector(testButtonAction:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:button];
    
    button. responseAreaWidth = 80;
    button. responseAreaHeight = 80;


- (void)testButtonAction:(UIButton *)button {
    NSLog(@"我要点击了 !!!!");
}

```








<br/>
***
<br/>







<br/>
***
<br/>




