- [解决页面因为键盘遮挡](#解决页面因为键盘遮挡)
- [iOS 键盘避让与收起](#iOS键盘避让与收起)
	- [键盘避让](#键盘避让)
	- [contentInset、滚动条 Insets 与 contentOffset](#contentInset、滚动条Insets与contentOffset)
	- [点击空白处收起键盘](#点击空白处收起键盘)
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



<br/>

***
<br/><br/><br/>
> <h1 id="iOS键盘避让与收起">iOS 键盘避让与收起</h1>

登录、注册等表单页面通常需要同时处理三件事：键盘出现后避免遮挡输入框、支持拖动收起键盘、点击输入区域以外的位置收起键盘。

```swift
private let formScrollView = UIScrollView()

private extension ArgusAppAccountPassword {
    func configureFormScrollView() {
        formScrollView.keyboardDismissMode = .interactive
        formScrollView.showsVerticalScrollIndicator = false
    }

    func addKeyboardObservers() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(keyboardWillChangeFrame(_:)),
            name: UIResponder.keyboardWillChangeFrameNotification,
            object: nil
        )

        NotificationCenter.default.addObserver(
            self,
            selector: #selector(keyboardWillHide(_:)),
            name: UIResponder.keyboardWillHideNotification,
            object: nil
        )
    }

    @objc func keyboardWillChangeFrame(_ notification: Notification) {
        guard let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
            return
        }

        let keyboardFrameInView = view.convert(keyboardFrame, from: nil)
        let keyboardOverlap = max(0, view.bounds.maxY - keyboardFrameInView.minY)
        updateFormForKeyboard(bottomInset: keyboardOverlap, using: notification)
    }

    @objc func keyboardWillHide(_ notification: Notification) {
        updateFormForKeyboard(bottomInset: 0, using: notification)
    }

    func updateFormForKeyboard(bottomInset: CGFloat, using notification: Notification) {
        let duration = notification.userInfo?[UIResponder.keyboardAnimationDurationUserInfoKey] as? TimeInterval ?? 0.25
        let rawCurve = notification.userInfo?[UIResponder.keyboardAnimationCurveUserInfoKey] as? UInt
            ?? UIView.AnimationOptions.curveEaseInOut.rawValue
        let options = UIView.AnimationOptions(rawValue: rawCurve << 16)

        UIView.animate(
            withDuration: duration,
            delay: 0,
            options: options,
            animations: {
                self.formScrollView.contentInset.bottom = bottomInset
                self.formScrollView.verticalScrollIndicatorInsets.bottom = bottomInset
            }
        )
    }
}
```

**核心逻辑：**键盘 frame 改变时，计算键盘与当前页面的重叠高度，再将该高度设置为 `UIScrollView` 的底部 inset，使内容可以滚动到键盘上方；键盘隐藏后将 inset 恢复为 `0`。

***
<br/>

> <h2 id="键盘避让">键盘避让</h2>

## `keyboardDismissMode`

```swift
formScrollView.keyboardDismissMode = .interactive
```

`keyboardDismissMode` 是 `UIScrollView` 的属性，用于控制用户滚动时如何收起键盘。

| 枚举值 | 说明 |
| --- | --- |
| `.none` | 滚动时不收起键盘 |
| `.onDrag` | 开始拖动 ScrollView 时收起键盘 |
| `.interactive` | 键盘跟随拖动手势交互式移动，用户可中途反向拖动 |

`.interactive` 主要用于垂直方向的键盘交互，体验与系统表单页面一致。它只负责**如何收起键盘**，不负责处理键盘遮挡。

```swift
formScrollView.showsVerticalScrollIndicator = false
```

`showsVerticalScrollIndicator = false` 用于隐藏右侧垂直滚动条，与键盘收起逻辑无关。

---
<br/>

## 注册键盘通知

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(keyboardWillChangeFrame(_:)),
    name: UIResponder.keyboardWillChangeFrameNotification,
    object: nil
)

NotificationCenter.default.addObserver(
    self,
    selector: #selector(keyboardWillHide(_:)),
    name: UIResponder.keyboardWillHideNotification,
    object: nil
)
```

- `keyboardWillChangeFrameNotification`：键盘位置或大小即将改变时触发，包括弹出、frame 变化、输入法切换等场景，比只监听 `keyboardWillShowNotification` 更完整。
- `keyboardWillHideNotification`：键盘即将隐藏时触发，用于恢复底部 inset。

使用 selector 方式注册的观察者在现代 iOS 中会随对象释放而自动失效，通常不会因此产生内存泄漏。不过，如果页面可能重复注册，或希望明确管理监听生命周期，仍可在适当时机调用：

```swift
NotificationCenter.default.removeObserver(self)
```

---
<br/>

## 计算键盘遮挡高度

```swift
@objc func keyboardWillChangeFrame(_ notification: Notification) {
    // 键盘动画结束时的 frame，坐标属于屏幕坐标系
    guard let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
        return
    }

    // 转换为当前 view 的坐标系
    let keyboardFrameInView = view.convert(keyboardFrame, from: nil)

    // 当前页面与键盘的重叠高度
    let keyboardOverlap = max(0, view.bounds.maxY - keyboardFrameInView.minY)

    updateFormForKeyboard(bottomInset: keyboardOverlap, using: notification)
}
```

**核心公式：**

```swift
keyboardOverlap = max(0, view.bounds.maxY - keyboardFrameInView.minY)
```

- `keyboardFrameEndUserInfoKey` 表示键盘动画结束后的最终 frame，应避免使用起始位置 `keyboardFrameBeginUserInfoKey`。
- 键盘通知提供的是屏幕坐标，必须通过 `view.convert(_:from:)` 转换到当前 View 坐标系。
- `max(0, ...)` 避免键盘位于页面外时得到负数。

例如：页面底部 `y = 844`，键盘顶部 `y = 546`，则重叠高度为 `844 - 546 = 298pt`。

---
<br/>

## 同步更新 ScrollView

```swift
func updateFormForKeyboard(bottomInset: CGFloat, using notification: Notification) {
    let duration = notification.userInfo?[UIResponder.keyboardAnimationDurationUserInfoKey] as? TimeInterval ?? 0.25
    let rawCurve = notification.userInfo?[UIResponder.keyboardAnimationCurveUserInfoKey] as? UInt
        ?? UIView.AnimationOptions.curveEaseInOut.rawValue
    let options = UIView.AnimationOptions(rawValue: rawCurve << 16)

    UIView.animate(
        withDuration: duration,
        delay: 0,
        options: options,
        animations: {
            self.formScrollView.contentInset.bottom = bottomInset
            self.formScrollView.verticalScrollIndicatorInsets.bottom = bottomInset
        }
    )
}
```

键盘通知提供动画时长和曲线。使用同一组参数更新 ScrollView，可以让键盘动画和页面布局变化保持同步，避免错位或卡顿。

`rawCurve << 16` 是将键盘通知中的动画曲线值转换为 `UIView.AnimationOptions` 所需的位布局。

键盘隐藏时将底部 inset 恢复为 `0`：

```swift
@objc func keyboardWillHide(_ notification: Notification) {
    updateFormForKeyboard(bottomInset: 0, using: notification)
}
```

**注意：**增加 `contentInset` 只是为内容提供可滚动空间，不保证当前输入框自动进入可见区域。需要自动定位时，还应结合 `scrollRectToVisible(_:animated:)` 等方法。

***
<br/>

> <h2 id="contentInset、滚动条Insets与contentOffset">contentInset、滚动条 Insets 与 contentOffset</h2>

三者作用不同：

| API | 作用 | 是否直接指定滚动位置 | 影响对象 |
| --- | --- | --- | --- |
| `contentInset.bottom = x` | 为滚动内容增加底部边距 | 否 | 内容可滚动区域 |
| `verticalScrollIndicatorInsets.bottom = x` | 调整垂直滚动条的底部边距 | 否 | 右侧滚动条 |
| `setContentOffset(.zero, animated: false)` | 将滚动位置设为原点 | 是 | ScrollView 当前可视位置 |

可以将 ScrollView 理解为通过一个窗口查看长内容：

- `contentInset`：在内容边缘增加留白，改变可滚动边界。
- `contentOffset`：移动窗口当前查看的位置。
- `verticalScrollIndicatorInsets`：只调整滚动条的显示范围。

---
<br/>

## `contentInset.bottom`

```swift
formScrollView.contentInset.bottom = bottomInset
```

键盘弹出且 `bottomInset = 300` 时，ScrollView 底部会增加 `300pt` 的边距，使底部内容能够继续向上滚动到键盘上方。它不会像 `setContentOffset` 一样主动要求页面滚到某个固定位置，但系统可能为了维持合法滚动范围而校正 offset。

---
<br/>

## `verticalScrollIndicatorInsets.bottom`

```swift
formScrollView.verticalScrollIndicatorInsets.bottom = bottomInset
```

该属性只调整垂直滚动条的显示区域，使滚动条底部停在键盘上方，不会主动滚动页面内容。

---
<br/>

## `setContentOffset(.zero, animated: false)`

```swift
formScrollView.setContentOffset(.zero, animated: false)
```

该方法会把 `contentOffset` 设为 `.zero`，即主动滚动到偏移原点；`animated: false` 表示瞬间完成，不播放滚动动画。

原代码在键盘隐藏时执行：

```swift
if bottomInset == 0 {
    formScrollView.setContentOffset(.zero, animated: false)
}
```

这会在收起键盘时强制改变用户的滚动位置。用户若正在填写页面中部或底部的表单，页面可能突然跳回顶部，因此一般应删除该逻辑，除非产品明确要求收起键盘后回到顶部。

### 完整场景

假设内容高度为 `1000pt`，可视高度为 `600pt`，用户当前位于 `contentOffset.y = 200`：

1. 键盘弹出，设置 `contentInset.bottom = 300`，底部新增可滚动空间，但没有主动指定新的 offset。
2. 键盘消失，设置 `contentInset.bottom = 0`，底部空间恢复。
3. 如果再调用 `setContentOffset(.zero, animated: false)`，则 `contentOffset` 被设为原点，页面立即回到顶部。

**结论：**`contentInset` 用于调整可滚动边界，`contentOffset` 用于指定当前滚动位置，两者不能互相替代。

***
<br/>

> <h2 id="点击空白处收起键盘">点击空白处收起键盘</h2>

以下代码实现：点击页面空白区域时收起键盘；点击账号、密码输入区域及其子视图时，不触发收起手势。

```swift
final class LoginViewController: UIViewController, UIGestureRecognizerDelegate {
    private let accountInputView = UIView()
    private let passwordInputView = UIView()

    override func viewDidLoad() {
        super.viewDidLoad()

        let tap = UITapGestureRecognizer(target: self, action: #selector(dismissKeyboard))
        tap.delegate = self
        view.addGestureRecognizer(tap)
    }

    @objc private func dismissKeyboard() {
        view.endEditing(true)
    }

    func gestureRecognizer(
        _ gestureRecognizer: UIGestureRecognizer,
        shouldReceive touch: UITouch
    ) -> Bool {
        guard let touchedView = touch.view else { return true }

        let touchedAccountInput = touchedView === accountInputView
            || touchedView.isDescendant(of: accountInputView)
        let touchedPasswordInput = touchedView === passwordInputView
            || touchedView.isDescendant(of: passwordInputView)

        return !touchedAccountInput && !touchedPasswordInput
    }
}
```

---
<br/>

## `dismissKeyboard()`

```swift
@objc private func dismissKeyboard() {
    view.endEditing(true)
}
```

- `view.endEditing(true)`：让当前 View 层级中的第一响应者结束编辑，从而收起键盘。
- `@objc`：使方法可被 Objective-C selector 调用。对于 `#selector(dismissKeyboard)`，编译器会检查方法是否能够暴露给 Objective-C；不满足要求通常会产生编译错误，而不是留到运行时崩溃。

---
<br/>

## 手势代理过滤点击区域

```swift
func gestureRecognizer(
    _ gestureRecognizer: UIGestureRecognizer,
    shouldReceive touch: UITouch
) -> Bool {
    guard let touchedView = touch.view else { return true }

    let touchedAccountInput = touchedView === accountInputView
        || touchedView.isDescendant(of: accountInputView)
    let touchedPasswordInput = touchedView === passwordInputView
        || touchedView.isDescendant(of: passwordInputView)

    return !touchedAccountInput && !touchedPasswordInput
}
```

- 返回 `true`：手势接收本次触摸，随后执行 `dismissKeyboard()`。
- 返回 `false`：手势忽略本次触摸，不执行收起键盘逻辑。
- `===`：判断点击的是否正是输入容器本身。
- `isDescendant(of:)`：判断点击的是否为输入容器内部的子视图，例如密码可见按钮、图标或 Label。

必须设置 `tap.delegate = self`，否则 `gestureRecognizer(_:shouldReceive:)` 不会参与判断。

### 执行结果

| 点击位置 | `shouldReceive` 返回值 | 结果 |
| --- | --- | --- |
| 账号或密码输入区域 | `false` | 手势不触发，键盘保持显示 |
| 输入区域内的子视图 | `false` | 手势不触发，键盘保持显示 |
| 页面空白区域 | `true` | 调用 `view.endEditing(true)` 收起键盘 |

***
<br/>

## 常见问题

- 忘记设置 `tap.delegate = self`：过滤方法不会执行。
- 只判断 `touchedView === inputView`：点击输入容器内部的按钮、图标时可能被误判为空白区域，应结合 `isDescendant(of:)`。
- 给 ScrollView 使用 `.onDrag` 或 `.interactive`：只能实现拖动时收起键盘，不能替代“点击空白收起”的手势逻辑。
- 只设置 `contentInset`：仅提供避让所需的滚动空间，不会自动将目标输入框滚动到可见区域。
- 键盘隐藏时强制 `setContentOffset(.zero, ...)`：会改变用户当前浏览位置，应根据产品需求谨慎使用。

## 工作流程

1. 输入框成为第一响应者，系统发出 `keyboardWillChangeFrameNotification`。
2. 读取键盘最终 frame，并转换到当前 View 坐标系。
3. 计算键盘与页面的重叠高度 `keyboardOverlap`。
4. 使用系统键盘动画参数更新 `contentInset.bottom` 和 `verticalScrollIndicatorInsets.bottom`。
5. 用户可通过 `.interactive` 拖动键盘，或点击输入区域外的空白位置收起键盘。
6. 键盘隐藏后将底部 inset 恢复为 `0`，但不强制重置用户的滚动位置。


