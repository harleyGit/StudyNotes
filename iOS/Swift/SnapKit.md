> <h1 id=""></h1>
- [为什么用320和375作为屏幕适配的分界线](#为什么用320和375作为屏幕适配的分界线)
- [left、right和leading、trailing区别](#left、right和leading、trailing区别) 
- [makeConstraints和remakeConstraints区别](#makeConstraints和remakeConstraints区别)
- [snapkit适配屏幕](#snapkit适配屏幕)
	- [fit适配屏幕](#fit适配屏幕)







<br/>

***
<br/><br/><br/>

> <h1 id=""></h1>


***
<br/><br/><br/>
> <h2 id="为什么用320和375作为屏幕适配的分界线">为什么用320和375作为屏幕适配的分界线</h2>

为什么用**320pt**和**375pt**作为屏幕适配的“分界线”，主要原因跟 iOS 设备的历史屏幕宽度和设计习惯有关，具体分析如下：

---

- **1.iOS 设备经典宽度尺寸**
	
	* **320pt** 是较早 iPhone（如 iPhone 4、5、SE 1代）的屏幕宽度（逻辑点数）。
	* **375pt** 是 iPhone 6、7、8 以及大部分现代主流 iPhone 的宽度。
	* iPhone Plus 系列是 414pt，更大的是 iPhone Max / Pro Max。

开发者通常以这两个宽度为**基准线**划分布局适配策略：

* 小屏（320pt）设备要保证界面不拥挤，字体和控件适当缩小。
* 标准屏（375pt）设备适合设计主流界面，常被视为设计稿基准宽度。
* 大屏（414pt及以上）则可增加边距或字体大小。

---

- **2.iOS 设计稿和 UI 设计习惯**

	* 设计师常用 **375pt 宽度** 设计 iPhone 界面（设计稿尺寸一般是 750px @2x 或 1125px @3x）。
	* 375pt 是苹果官方 iPhone UI 设计的推荐基准，适合绝大多数用户。
	* 320pt 设备在早期非常普及，故经常作为“兼容最小宽度”。

---

- **3.适配的实际意义**

	* 使用这两个分界线，可以方便地做条件判断，比如：

```swift
if UIScreen.main.bounds.width <= 320 {
    // 小屏设备，适当缩小字体或间距
} else if UIScreen.main.bounds.width <= 375 {
    // 标准屏设备，使用默认设计
} else {
    // 大屏设备，增加间距或字体
}
```

* 这种划分简洁且覆盖了大部分 iPhone 机型。

---

- **4.兼容性和历史原因**

* 320pt 屏幕宽度在 iPhone 5 及之前设备长期存在，App 需要兼容这些小屏设备。
* 375pt 是从 iPhone 6 开始的新标准，设计稿也主要以此为基准。
* 其他尺寸（如 414pt）虽然存在，但比例关系可以通过相对缩放或更细致的适配处理。

---

- **总结：**

| 选择理由      | 说明                      |
| --------- | ----------------------- |
| **320pt** | 早期 iPhone 标准小屏，兼容性考虑    |
| **375pt** | iPhone 6+ 后主流屏幕宽度，设计稿标准 |
| **简单划分**  | 方便开发者根据屏幕大小调整布局和字体      |
| **历史延续**  | 保证旧设备与新设备都能良好适配         |




***
<br/><br/><br/>
> <h2 id="left、right和leading、trailing区别">left、right和leading、trailing区别</h2>


在 **SnapKit** 中，`left` 和 `leading` 是用来指定视图的水平方向布局属性，但它们的使用有所区别，**特别是在处理从左到右（LTR）和从右到左（RTL）语言环境时**。这个在国内倒无所谓，但是对于一款供全世界使用的app，特别是中东阿拉伯地区使用者来说很重要了。他们是**从右到左的**。

<br/>

 - **1.`left` 属性**
	- **定义**: 指视图的左边缘（在标准的从左到右布局中）。
	- **使用场景**: 在不考虑语言方向的布局中，可以直接使用 `left` 来表示左侧对齐。
	- **局限性**: 在支持从右到左（RTL）的语言时，`left` 仍然指代视图的“左侧”，这可能并不符合 RTL 布局的需求。

```swift
view.snp.makeConstraints { make in
    make.left.equalToSuperview()
}
```

<br/>

 - **2.`leading` 属性**
	- **定义**: 表示视图的“前导边缘”，它会根据语言环境自动调整：
	  - 在从左到右（LTR）语言中，`leading` 等同于 `left`。
	  - 在从右到左（RTL）语言中，`leading` 等同于 `right`。
	- **使用场景**: 如果希望支持 RTL 和 LTR 的语言方向，可以使用 `leading`，它会根据当前语言环境自动调整视图的对齐方向。

```swift
view.snp.makeConstraints { make in
    make.leading.equalToSuperview()
}
```

<br/>

 - **3.总结**
	- **`left`**: 始终指视图的左边缘，无论语言方向。
	- **`leading`**: 动态调整，根据语言方向，可以指左边缘或右边缘。

如果你正在开发一个需要支持多语言环境的应用，建议优先使用 `leading` 和 `trailing` 这种更灵活的属性，确保布局在 RTL 和 LTR 环境中都能正确显示。


<br/><br/>

而且还有一点很重要的是，使用left在中东国家地区的时候，若是遵从**RTL**布局很可能造成app的崩溃。


***
<br/><br/><br/>
> <h2 id="makeConstraints和remakeConstraints区别">makeConstraints和remakeConstraints区别</h2>

时隔多年再一次使用Snapkit时我设置约束时，犯了一个很大的错误。就是我将约束放在了UIView的系统方法`-(void)layouSubViews`方法里了，并且布局使用的还是`makeConstraints`方法，这样可能会导致这个布局方法在`-(void)layouSubViews`多次调用，可能导致程序崩溃。

若是避免你可以用`remakeConstraints`方法，因为这个方法会把之前设置的约束都给清除掉，然后重新设置。但是这会又性能的损耗，所以需要单独一个方法对其进行布局了。


***
<br/><br/><br/>
> <h2 id="snapkit适配屏幕">snapkit适配屏幕</h2>

snpkit这个库使用时,如下:

```swift
nextButton.snp.makeConstraints { make in
	make.bottom.equalTo(view.snp.bottom).offset((-27).fit)
	make.left.right.equalTo(view).inset(27.fit)
	make.height.equalTo(46.fit)
}
```

* `nextButton` 是你要布局的视图
* `view` 是它的父视图
* `.fit` 是你项目中定义的 **屏幕适配缩放方法**（非 SnapKit 自带）

---

**📌 逐行详解：**

- **1️⃣ `make.bottom.equalTo(view.snp.bottom).offset((-27).fit)`**
	* 让 `someView` 的 **底部对齐到父视图 `view` 的底部**
	* 并**向上偏移** `27.fit` 个单位（负值代表“向上”）

🔍 相当于：

```swift
someView.bottom = view.bottom - 27.fit
```

---

- **2️⃣ `make.left.right.equalTo(view).inset(27.fit)`**
	* 同时设置 **左边距、右边距**
	* 相当于：

```swift
someView.left = view.left + 27.fit
someView.right = view.right - 27.fit
```

> ⚠️ `.inset(...)` 是 SnapKit 中的语法糖，只有在 `.left.right.equalTo(...)`、`.edges.equalTo(...)` 时才可以使用。

---

- **3️⃣ `make.height.equalTo(46.fit)`**
	* 设置固定高度为 `46.fit`，单位根据你的 `.fit` 方法决定

---

- **✅ 那 `.fit` 是什么？**

`.fit` 并不是 SnapKit 的内容，而是你项目中自定义的 **屏幕适配扩展方法**，比如：

```swift
extension Int {
    var fit: CGFloat {
        return CGFloat(self) * (UIScreen.main.bounds.width / 375.0)
    }
}
```

作用是：

> 👉 在不同屏幕宽度下，自动缩放数值来保持相对布局一致。

举例：

* 如果你用的是 iPhone 15 Pro Max（屏宽 ≈ 430），`27.fit` ≈ `27 × 430 / 375 = 30.96`
* 在 iPhone 8（屏宽 375）上 `27.fit = 27`

---

- **🧩 总结整体效果**

```swift
someView.snp.makeConstraints { make in
    // someView 的底部距离 view 底部 “向上” 27 像素（缩放后）
    make.bottom.equalTo(view.snp.bottom).offset(-27.fit)

    // someView 的左右边距为 27（缩放后）
    make.left.right.equalTo(view).inset(27.fit)

    // 固定高度为 46（缩放后）
    make.height.equalTo(46.fit)
}
```

**效果:**
> “把 `someView` 贴在父视图 `view` 的底部，左右边距 27，底部上移 27，固定高度 46，且都经过屏幕适配缩放。”


<br/><br/>
> <h3 id="fit适配屏幕">fit适配屏幕</h3>
封装一套 Swift 的屏幕适配工具，配合 SnapKit 使用非常自然。采用主流的 **以 iPhone 375pt 屏宽为基准**（即 iPhone 6/7/8 尺寸）。

---

- **✅ 一、基础适配工具封装：UIScreen+Adapter.swift**

```swift
import UIKit

/// 适配比例（以 iPhone 375pt 宽度为基准）
let kScreenWidth = UIScreen.main.bounds.width
let kScreenHeight = UIScreen.main.bounds.height
let kWidthRatio = kScreenWidth / 375.0
let kHeightRatio = kScreenHeight / 667.0

/// 支持 Int / CGFloat / Double 的适配扩展
extension Int {
    var fit: CGFloat {
        return CGFloat(self) * kWidthRatio
    }
}

extension CGFloat {
    var fit: CGFloat {
        return self * kWidthRatio
    }
}

extension Double {
    var fit: CGFloat {
        return CGFloat(self) * kWidthRatio
    }
}
```

---

- **✅ 二、用法示例（配合 SnapKit）**

```swift
someView.snp.makeConstraints { make in
    make.bottom.equalTo(view.snp.bottom).offset(-27.fit)
    make.left.right.equalTo(view).inset(27.fit)
    make.height.equalTo(46.fit)
}
```

---

- **📝 说明：**

| 名称             | 作用                |
| -------------- | ----------------- |
| `.fit`         | 横向缩放适配（适用于大多数布局）  |
| `kWidthRatio`  | 屏幕宽度与基准（375）的缩放比例 |
| `kHeightRatio` | 屏幕高度与基准（667）的缩放比例 |

> ❗提示：通常 UI 控件的尺寸使用 `.fit`（基于宽度）；字体适配可以根据需要定义 `fontFit` 或保持不变。

---

- **✅ 三、额外：字体适配（可选）**

```swift
extension UIFont {
    static func fitFont(_ size: CGFloat, weight: UIFont.Weight = .regular) -> UIFont {
        return UIFont.systemFont(ofSize: size.fit, weight: weight)
    }
}
```

用法示例：

```swift
label.font = .fitFont(14, weight: .medium)
```

---

- **✅ 四、封装为工具类（可选）**

你也可以将适配相关的内容封装进一个静态结构体：

```swift
struct ScreenAdapter {
    static let baseWidth: CGFloat = 375.0
    static var widthRatio: CGFloat {
        return UIScreen.main.bounds.width / baseWidth
    }
    static func fit(_ value: CGFloat) -> CGFloat {
        return value * widthRatio
    }
}
```

然后使用：

```swift
let adapted = ScreenAdapter.fit(27)
```

---

- **✅ 总结**

* `.fit` 是用来做横向适配（以屏宽为基础）
* 通常适用于间距、frame、SnapKit 约束等
* 简洁、直观、适配效果好

---

- **✅ 文件结构建议**

```
📁 ScreenAdapter
├── ScreenAdapter.swift         // 主适配核心逻辑
├── UIFont+Fit.swift            // 字体适配封装
├── FitExtensions.swift         // Int/CGFloat/Double 的 .fit 快捷扩展
```

---

- **📄 1. `ScreenAdapter.swift代码`**

```swift
import UIKit

/// 屏幕适配核心工具（以 iPhone 6 为基准 375×667）
public struct ScreenAdapter {

    static let baseWidth: CGFloat = 375.0
    static let baseHeight: CGFloat = 667.0

    /// 当前屏幕宽高
    public static var screenWidth: CGFloat {
        UIScreen.main.bounds.width
    }

    public static var screenHeight: CGFloat {
        UIScreen.main.bounds.height
    }

    /// 横向适配比例
    public static var widthRatio: CGFloat {
        screenWidth / baseWidth
    }

    /// 纵向适配比例（谨慎使用）
    public static var heightRatio: CGFloat {
        screenHeight / baseHeight
    }

    /// 是否是 iPad
    public static var isPad: Bool {
        UIDevice.current.userInterfaceIdiom == .pad
    }

    /// 是否是小屏设备（≤ 320pt）
    public static var isSmallScreen: Bool {
        screenWidth <= 320
    }

    /// 横屏检测
    public static var isLandscape: Bool {
        UIApplication.shared.statusBarOrientation.isLandscape
    }

    /// 安全区域 Insets
    public static var safeInsets: UIEdgeInsets {
        UIApplication.shared.windows.first?.safeAreaInsets ?? .zero
    }

    /// 顶部安全区域高度（如刘海、状态栏）
    public static var safeTop: CGFloat {
        safeInsets.top
    }

    /// 底部安全区域高度（如 Home Indicator）
    public static var safeBottom: CGFloat {
        safeInsets.bottom
    }

    /// 横向适配尺寸
    public static func fitWidth(_ value: CGFloat) -> CGFloat {
        return value * widthRatio
    }

    /// 百分比布局宽度
    public static func percentWidth(_ percent: CGFloat) -> CGFloat {
        return screenWidth * percent
    }

    /// 百分比布局高度
    public static func percentHeight(_ percent: CGFloat) -> CGFloat {
        return screenHeight * percent
    }

    /// 字体适配（iPad 可放大）
    public static func fitFont(_ size: CGFloat) -> CGFloat {
        return isPad ? size * 1.2 : fitWidth(size)
    }
}
```

---

- **📄 2. `UIFont+Fit.swift`**

```swift
import UIKit

extension UIFont {

    /// 自动适配字体（系统字体）
    static func fit(_ size: CGFloat, weight: UIFont.Weight = .regular) -> UIFont {
        return UIFont.systemFont(ofSize: ScreenAdapter.fitFont(size), weight: weight)
    }

    /// 自定义字体适配（如 PingFangSC-Medium）
    static func fit(name: String, size: CGFloat) -> UIFont {
        return UIFont(name: name, size: ScreenAdapter.fitFont(size)) ?? .systemFont(ofSize: size)
    }
}
```

---

- **📄 3. `FitExtensions.swift`**

```swift
import UIKit

extension Int {
    var fit: CGFloat {
        return ScreenAdapter.fitWidth(CGFloat(self))
    }
}

extension CGFloat {
    var fit: CGFloat {
        return ScreenAdapter.fitWidth(self)
    }
}

extension Double {
    var fit: CGFloat {
        return ScreenAdapter.fitWidth(CGFloat(self))
    }
}
```

---

- **✅ 用法示例（UIKit + SnapKit）**

```swift
titleLabel.font = .fit(16, weight: .semibold)

button.snp.makeConstraints { make in
    make.left.right.equalToSuperview().inset(20.fit)
    make.bottom.equalToSuperview().offset(-ScreenAdapter.safeBottom - 12.fit)
    make.height.equalTo(48.fit)
}
```

- **✅ 用法示例（UIKit + Frame）**

```swift
let btn = UIButton()
btn.frame = CGRect(
    x: 20.fit,
    y: ScreenAdapter.safeTop + 10.fit,
    width: ScreenAdapter.screenWidth - 40.fit,
    height: 48.fit
)
```

---

**UIKit界面的小Demo页面 ViewController 示例:**

```swift
//
//  DemoViewController.swift
//  ScreenAdapterDemo
//

import UIKit
import SnapKit

class DemoViewController: UIViewController {

    private let titleLabel = UILabel()
    private let actionButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white

        setupTitleLabel()
        setupActionButton()
    }

    private func setupTitleLabel() {
        titleLabel.text = "欢迎使用适配演示"
        titleLabel.textAlignment = .center
        titleLabel.font = .fit(20, weight: .medium)
        view.addSubview(titleLabel)

        titleLabel.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(ScreenAdapter.safeTop + 40.fit)
            make.centerX.equalToSuperview()
        }
    }

    private func setupActionButton() {
        actionButton.setTitle("点击我", for: .normal)
        actionButton.backgroundColor = .systemBlue
        actionButton.setTitleColor(.white, for: .normal)
        actionButton.titleLabel?.font = .fit(16)
        actionButton.layer.cornerRadius = 10.fit
        view.addSubview(actionButton)

        actionButton.snp.makeConstraints { make in
            make.left.equalToSuperview().offset(30.fit)
            make.right.equalToSuperview().offset(-30.fit)
            make.bottom.equalToSuperview().offset(-ScreenAdapter.safeBottom - 30.fit)
            make.height.equalTo(50.fit)
        }

        actionButton.addTarget(self, action: #selector(didTapButton), for: .touchUpInside)
    }

    @objc private func didTapButton() {
        let alert = UIAlertController(title: "提示", message: "你点击了按钮！", preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "确定", style: .default))
        present(alert, animated: true)
    }
}

```




