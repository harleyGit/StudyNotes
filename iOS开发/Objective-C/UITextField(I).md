>#清空按钮
```
typedef NS_ENUM(NSInteger, UITextFieldViewMode) {
    UITextFieldViewModeNever,           //清空按钮永不出现
    UITextFieldViewModeWhileEditing,    //编辑的时候出现
    UITextFieldViewModeUnlessEditing,   //不编辑的时候出现
    UITextFieldViewModeAlways           //只要输入框有内容就出现
};


UITextField *field = [UITextField new];
field.clearButtonMode=UITextFieldViewModeWhileEditing; 
```


<br/>
***
<br/>

```
//申明变量
@property(nonatomic, retain) UITextField *inputText;


- (UITextField *)inputText {
    if (!_inputText) {
        _inputText = [UITextField new];
        _inputText.delegate = self;
        _inputText.textAlignment = NSTextAlignmentLeft;
        _inputText.backgroundColor = [UIColor whiteColor];
        //设置输入起始位置
        [_inputText setValue:[NSNumber numberWithInt:20] forKey:@"paddingLeft"];
    }
    return _inputText;
}


#pragma mark -- UITextFieldDelegate
//输入长度判断
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    //为了获取删除操作，若没有if会造成当达到字数限制后删除键不能使用的操作
    if (range.length == 1 && string.length == 0) {
        return YES;
    }
    
    if (textField.text.length >= 30) {
        textField.text = [textField.text substringToIndex:30];
        return NO;
    }
    
    return YES;
}
```

效果图：
![起始输入和长度设置](https://upload-images.jianshu.io/upload_images/2959789-2130fef6c6675534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
