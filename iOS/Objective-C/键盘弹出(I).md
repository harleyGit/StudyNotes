- [解决页面因为键盘遮挡](#解决页面因为键盘遮挡)
- **资料**
	- [**键盘弹出调整输入框位置(TextView或者TextFiled)**](https://www.jianshu.com/p/4cf85b642979)
	- [**键盘上方的view随着键盘的弹出、收起、键盘输入法改变而移动**](https://blog.csdn.net/crystal_198874/article/details/43954741)
	- [**UITextField输入框随键盘弹出界面上移**](https://blog.csdn.net/walkerwqp/article/details/77881499)






<br/><br/><br/>

***
<br/>

> <h1 id= "解决页面因为键盘遮挡">解决页面因为键盘遮挡</h1>

```swift
private extension HGRegisterController {
    
    private weak var activeTextField: UITextField?
    confirmPasswordInputView.delegate = self
    addKeyboardObservers()
   
    deinit {
        NotificationCenter.default.removeObserver(self)
        
    }
    
    func textFieldDidBeginEditing(_ textField: UITextField) {
        activeTextField = textField
    }
    func textFieldDidEndEditing(_ textField: UITextField) {
        if activeTextField == textField {
            activeTextField = nil
        }
    }
}

private extension HGRegisterController {
    
    func addKeyboardObservers() {
        
        NotificationCenter.default.addObserver(self, selector:#selector(keyboardWillChangeFrame(_:)), name: UIResponder.keyboardWillChangeFrameNotification, bject: nil)
        NotificationCenter.default.addObserver(self, elector: #selector(keyboardWillHide(_:)), ame: UIResponder.keyboardWillHideNotification, bject: nil)
    }
    
    @objc func keyboardWillChangeFrame(_ notification: Notification) {
        
        guard let activeTextField = activeTextField,
              activeTextField == passwordInputView || activeTextField == confirmPasswordInputView else {
            restoreKeyboardAvoidance(using: notification)
            return
        }
        guard let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
            return
        }
        
        view.transform = .identity
        let keyboardFrameInView = view.convert(keyboardFrame, from: nil)
        let inputFrame = activeTextField.convert(activeTextField.bounds, to: view)
        let requiredBottomSpacing = 12.fit
        let overlap = inputFrame.maxY + requiredBottomSpacing - keyboardFrameInView.minY
        let offset = max(0, overlap)
        animateKeyboardAvoidance(offset: offset, using: notification)
    }
    
    @objc func keyboardWillHide(_ notification: Notification) {
        
        restoreKeyboardAvoidance(using: notification)
    }
    
    func restoreKeyboardAvoidance(using notification: Notification) {
        
        animateKeyboardAvoidance(offset: 0, using: notification)
    }
    
    func animateKeyboardAvoidance(offset: CGFloat, using notification: Notification) {
        
        let duration = notification.userInfo?[UIResponder.keyboardAnimationDurationUserInfoKey] as? TimeInterval ?? 0.25
        let rawCurve = notification.userInfo?[UIResponder.keyboardAnimationCurveUserInfoKey] as? UInt ?? UIView.AnimationOptions.curveEaseInOut.rawValue
        let options = UIView.AnimationOptions(rawValue: rawCurve << 16)
        
        UIView.animate(withDuration: duration,
                       delay: 0,
                       options: options,
                       animations: {
            self.view.transform = offset > 0 ? CGAffineTransform(translationX: 0, y: -offset) : .identity
        })
    }
}
```

这段代码本质上是在做：**键盘弹出时，自动把界面往上移动，避免输入框被键盘挡住。**

- 属于 iOS 中非常经典的机制：
	* Keyboard Avoidance
	* 键盘避让
	* 输入框防遮挡

***
<br/>

在注册页面大致UI控件布局：

```text
账号输入框

密码输入框

确认密码输入框
```

当你点击`‌确认密码输入框` 时，iPhone 键盘会从底部弹出。但 `‌确认密码输入框` 可能刚好在屏幕底部。于是`‌键盘把输入框挡住了`，用户根本看不到自己输入什么。

所以必须：`‌键盘弹出时，整个页面向上移动`。这就是`‌键盘避让`

***
<br/>

## 代码大致流程：

```text
1. 用户点击输入框
2. textFieldDidBeginEditing
3. 记录当前激活输入框 activeTextField

4. 键盘即将弹出
5. 收到 keyboardWillChangeFrameNotification

6. 计算：
   当前输入框是否会被挡住

7. 如果会挡住：
   view 往上移动

8. 用户收起键盘
9. keyboardWillHide

10. 页面恢复原位
```

---
<br/>

### `activeTextField` 是什么

```swift
// 当前正在输入的输入框
private weak var activeTextField: UITextField?
```

- 比如：
	* 用户点了密码框
	* activeTextField = passwordInputView

- 或者：
	* 用户点了确认密码框
	* activeTextField = confirmPasswordInputView

<br/>

###  为什么 weak

因为`UITextField`已经被 view 强引用了。这里`‌activeTextField`只是临时记录，避免循环引用。`weak`是正确写法。

<br/>

###  开始监听键盘通知

```swift
// 注册键盘通知
func addKeyboardObservers() {}
```

<br/>

#### 第一段监听： 监听键盘 frame 改变

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(keyboardWillChangeFrame(_:)),
    name: UIResponder.keyboardWillChangeFrameNotification,
    object: nil
)
```

- **什么时候会触发：**
	* 键盘弹出
	* 键盘收起
	* 键盘高度变化
	* 第三方键盘切换
	* 键盘浮动
	* iPad 分离键盘

都会触发。

<br/>

#### 第二段

```swift
// 监听键盘消失
keyboardWillHideNotification
```

<br/>

#### 为什么 deinit 要 removeObserver

```swift
deinit {
    NotificationCenter.default.removeObserver(self)
}
```

意思：`‌对象销毁时， 取消通知监听`。否则`对象已经释放，通知还在回调`会崩溃。虽然**iOS 9 后很多 Notification 自动管理了**。但`removeObserver`仍然是好习惯。

---
<br/>

### 输入框开始编辑

```swift
func textFieldDidBeginEditing(_ textField: UITextField) {
    activeTextField = textField
}
```

意思：`当前哪个输入框正在输入`.比如：`点击密码框`,则：

```swift
activeTextField = passwordInputView
```

<br/>

```swift
func textFieldDidEndEditing(_ textField: UITextField) {
    if activeTextField == textField {
        activeTextField = nil
    }
}
```

意思：`‌输入结束后清空`,`记录失效输入框`


---
<br/>

### 最核心部分：keyboardWillChangeFrame

#### 第一部分

```swift
guard let activeTextField = activeTextField,
      activeTextField == passwordInputView ||
      activeTextField == confirmPasswordInputView else {
    restoreKeyboardAvoidance(using: notification)
    return
}
```

意思：**`只有密码输入框和确认密码输入框,才做键盘避让`**,否则`恢复原位`

<br/>

比如：`账号输入框在上方,不会被挡住`,就不需要移动页面。

<br/>

### 第二部分

```swift
guard let keyboardFrame =
notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey]
as? CGRect else {
    return
}
```

意思：`‌获取键盘最终位置`,比如：`‌键盘高度 = 336`

---
<br/>

### 最关键数学计算

-  **1.重置 transform**

```swift
view.transform = .identity
```

意思：`‌先恢复原位，再重新计算`，否则`‌连续叠加位移`，会越来越偏。

<br/>

- **2.键盘坐标转换**

```swift
let keyboardFrameInView =
view.convert(keyboardFrame, from: nil)
```

意思：`把键盘坐标,转换到当前 view 坐标系`。因为**`‌键盘 frame 默认是屏幕坐标`**，必须转换。

<br/>

- **3.获取输入框位置**

```swift
let inputFrame =
activeTextField.convert(activeTextField.bounds, to: view)
```

意思`‌获取输入框在 view 中的位置`，比如`‌输入框底部 Y = 700`

<br/>

- **4.留出安全间距**

```swift
let requiredBottomSpacing = 12.fit
```

意思**`‌输入框距离键盘顶部,至少保留 12`**,否则太贴边。

<br/>

- **5.计算重叠区域**

```swift
let overlap =
inputFrame.maxY +
requiredBottomSpacing -
keyboardFrameInView.minY
```

这是核心公式。

---
<br/>

### 举实际例子（最重要）

比如输入框位置`‌输入框底部 = 720`,即：

```swift
inputFrame.maxY = 720
```
<br/>

**键盘顶部**`‌键盘顶部 Y = 650`,即：

```swift
keyboardFrameInView.minY = 650
```
<br/>

**安全距离**

```swift
requiredBottomSpacing = 12
```
<br/>

**代入公式**`‌overlap = 720 + 12 - 650`，结果`‌82`。意思：`‌输入框被挡住了 82pt`，所以`‌页面需要上移 82pt`

---
<br/>

###  `max(0, overlap)`

```swift
let offset = max(0, overlap)
```

**意思:** `‌如果 overlap 小于 0,说明没被挡住,不需要移动`.比如`‌输入框在上面`,则`‌overlap = -200`,最终`‌offset = 0`


---
<br/>

### 开始动画移动

**`animateKeyboardAvoidance`**

```swift
UIView.animate(
    withDuration: duration,
    delay: 0,
    options: options
)
```

**意思：** `‌跟随系统键盘动画,同步移动.`这样`键盘升起,页面也同步丝滑上移`

---
<br/>

### 为什么从 notification 获取 duration

```swift
let duration =
notification.userInfo?[
UIResponder.keyboardAnimationDurationUserInfoKey
]
```

因为`‌必须和系统键盘动画一致`,否则`‌键盘已经弹完,页面才开始动`体验很差。

---
<br/>

### 为什么 `rawCurve << 16`

```swift
let options =
UIView.AnimationOptions(rawValue: rawCurve << 16)
```

这是 UIKit 历史遗留设计。通知里`‌拿到的是 UIViewAnimationCurve`,而`‌UIView.animate` 需要`‌UIView.AnimationOptions`,所以**‌ 必须左移 16 位**,这是官方标准写法。

---

### 真正移动页面

```swift
self.view.transform =
offset > 0
? CGAffineTransform(translationX: 0, y: -offset)
: .identity
```

- **意思:** 
	- 有遮挡`‌y: -offset` 页面向上移动。
	- 无遮挡 `‌.identity` 恢复原位。

---
<br/>

### 为什么用 transform

很多人这里不懂。

- **transform** 的本质
	- 视觉变换
	- 不会
		- 修改 frame
		- 修改约束
		- 修改布局
	- 只是`‌把整个 view 渲染时移动`
	- 所以
		- 性能很好
		- 动画流畅
		- 不破坏 AutoLayout
**这是现代推荐方案。**

---
<br/>

## 完整运行流程

### 场景

**页面：**

```text
账号输入框

密码输入框

确认密码输入框
```

<br/>

- **用户点击确认密码**,触发`‌textFieldDidBeginEditing`

- 记录`activeTextField = confirmPasswordInputView`

- 键盘弹出,系统发送：`keyboardWillChangeFrameNotification `

<br/>

### 代码开始计算

- 发现：`确认密码框被挡住 90pt`
- 执行动画 `‌view.transform = translationY(-90)`
- 结果：
	- 整个页面上移
	- 输入框露出来

<br/>

### 用户收起键盘

- 触发：`‌keyboardWillHide`
- 恢复 `‌view.transform = .identity`
- 页面回归原位。

这是 iOS 键盘避让最经典的方案之一。

