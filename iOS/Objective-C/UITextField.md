- [输入框字数限制](#输入框字数限制)
- [**清空按钮**](#清空按钮)
- **参考资料：**
	- [UITextField的那点事](https://www.jianshu.com/p/4b9957f2afc2)




 
***
<br/><br/><br/>
> <h2 id="输入框字数限制">输入框字数限制</h2>

```swift
func textField(_ textField: UITextField,
               shouldChangeCharactersIn range: NSRange,
               replacementString string: String) -> Bool {
               
		let currentText = textField.text ?? ""
		
		guard let textRange = Range(range, in: currentText) else {
		    return false
		}
		
		let updatedText = currentText.replacingCharacters(in: textRange, with: string)
		
		return areaCodeDigits(from: updatedText).count <= 6
}
```

假设当前输入框内容：`86`,用户继续输入：`1`.那么：

```swift
currentText = "86"

string = "1"

range = {2,0}
```

表示：`‌从索引2开始，替换0个字符`，即插入操作。

---
<br/>

### 第一步

```swift
let currentText = textField.text ?? ""

// 获取输入框当前内容
currentText = "86"
```
如果为空：

```swift
currentText = ""
```

<br/>

### 第二步

```swift
guard let textRange = Range(range, in: currentText) else {
    return false
}
```

**为什么要转换？**

UIKit 给的是`‌NSRange`,例如`‌{2,0}`。而 Swift 字符串操作需要`‌Range<String.Index>`，所以必须转换。

例如：

```swift
NSRange(location: 2, length: 0)
```

转换成：

```swift
currentText.index(...)
```

形式。

<BR/>

当前`‌86`,对应`‌range = {2,0}`转换后`‌textRange`,表示`‌字符串末尾的位置`

<br/>

```swift
let updatedText =
currentText.replacingCharacters(
    in: textRange,
    with: string
)
```

模拟用户输入后的结果。

注意：此时 UITextField 还没真正修改。系统是在问你：

```text
如果允许这次输入
结果会变成什么？
```

<br/>

## 示例1：输入字符

当前：`‌86`,输入：`‌1`,得到：

```swift
updatedText = "861"
```

<br/>

## 示例2：删除字符

当前：`‌861"`,点击删除。此时：

```swift
range = {2,1}
string = ""
```

得到：

```swift
updatedText = "86"
```

<br/>

### 示例3：替换字符

当前：`‌861`。选中：`‌6`。输入：`9`，得到：

```swift
updatedText = "891"
```

***
<br/>

### 第四步

```swift
areaCodeDigits(from: updatedText)
```

这里是自定义方法。大概率类似：

```swift
private func areaCodeDigits(from text: String) -> String {
    text.filter(\.isNumber)
}
```

或者：

```swift
private func areaCodeDigits(from text: String) -> [Character] {
    text.filter { $0.isNumber }
}
```

作用：`‌提取数字`
---
<br/>

## 举例

输入：

```swift
"+86"
```

得到：

```swift
"86"
```

---
<br/>

输入：

```swift
"+1"
```

得到：

```swift
"1"
```

---

输入：

```swift
"+123456"
```

得到：

```swift
"123456"
```

---

# 第五步

```swift
.count <= 6
```

限制数字长度不能超过 6 位。

---

## 示例1

输入：

```text
123456
```

数字数：

```swift
6
```

结果：

```swift
true
```

允许输入。

---

## 示例2

输入：

```text
1234567
```

数字数：

```swift
7
```

结果：

```swift
false
```

拒绝输入。

---

# 整体流程演示

假设区号输入框：

```text
+86
```

用户继续输入：

```text
1
```

执行：

```swift
currentText = "+86"

updatedText = "+861"

areaCodeDigits(from: updatedText)
```

结果：

```swift
"861"
```

长度：

```swift
3
```

判断：

```swift
3 <= 6
```

返回：

```swift
true
```

允许输入。

---

再输入：

```text
23456
```

变成：

```text
+86123456
```

数字：

```swift
86123456
```

长度：

```swift
8
```

判断：

```swift
8 <= 6
```

返回：

```swift
false
```

输入被拦截。

---

# 这段代码实际等价于

```swift
func textField(_ textField: UITextField,
               shouldChangeCharactersIn range: NSRange,
               replacementString string: String) -> Bool {

    // 预测修改后的内容
    let newText = 当前文本 + 本次输入

    // 提取数字
    let digits = 只保留数字(newText)

    // 最多允许6位数字
    return digits.count <= 6
}
```

常见用于：

* 国际电话区号（+86、+1、+852）
* 国家代码
* 验证码输入
* 纯数字输入框长度限制

其中最关键的一句是：

```swift
let updatedText = currentText.replacingCharacters(in: textRange, with: string)
```

它是在**提前模拟本次输入后的最终结果**，然后根据结果决定是否允许这次输入。这样无论是输入、删除、粘贴、替换，都能正确处理。



<br/><br/><br/>

***
<br/>

> <h1 id="清空按钮">清空按钮</h1>


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
