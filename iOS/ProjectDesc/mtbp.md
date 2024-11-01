> <h1 id=""></h1>
- [**RxSwift使用**](#RxSwift使用)
	- [BehaviorRelay使用](#BehaviorRelay使用)
		- [BehaviorRelay数组](#BehaviorRelay数组)
	- [**PublishRelay使用**](#PublishRelay使用)
	- [**Reactive使用**](#Reactive使用)
- [**语言本地化**](#语言本地化)
- [**ObjectMapper库**](#ObjectMapper库)
	- [接受网络数据转化为数据模型](#接受网络数据转化为数据模型)
		- [不继承Then协议,如何使用Then的链式语法](#不继承Then协议,如何使用Then的链式语法)
- [**Then库**](#Then库)
- [PixDefines库](#PixDefines库)
	- [PixNameSpace使用](#PixNameSpace使用)
		- [关键字where约束和范型的联合](#关键字where约束和范型的联合)


<br/>

***
<br/><br/><br/>

> <h1 id="RxSwift使用">RxSwift使用</h1>

<br/><br/><br/>

> <h2 id="BehaviorRelay使用">BehaviorRelay使用</h2>

`BehaviorRelay` 是 `RxSwift` 中的一种特殊的 `Relay` 类型，它包装了一个可以存储当前值的状态，并且能够让观察者订阅它以接收最新的值和随后的所有更改。它是 `BehaviorSubject` 的一种变体，但没有 `onError` 和 `onCompleted` 事件的概念，因此更适用于状态管理。

下面是 `BehaviorRelay` 的一些常用用法：

- **1.创建和初始化 `BehaviorRelay`**
初始化时需要传入一个默认值（初始值）：

```swift
// 创建一个 BehaviorRelay，初始值为 0
let relay = BehaviorRelay<Int>(value: 0)
```

- **2.读取和获取当前值**

`BehaviorRelay` 保存了一个当前值，可以通过 `.value` 访问：

```swift
print(relay.value)  // 输出: 0
```

- **3.监听值的变化**
可以通过 `Observable` 来监听 `BehaviorRelay` 的值变化：

```swift
relay.asObservable()
    .subscribe(onNext: { value in
        print("值发生变化：\(value)")
    })
    .disposed(by: disposeBag)
```

- **4.更新值**

`BehaviorRelay` 没有 `onNext` 方法，因此要用 `.accept` 方法更新值。每次更新值时，订阅者会自动收到新值：

```swift
relay.accept(10)  // 更新值为 10
relay.accept(20)  // 更新值为 20
```

<br/><br/>

- **5.常见示例**

例如，在 UI 中使用 `BehaviorRelay` 来管理状态：

```swift
// 定义一个字符串类型的 BehaviorRelay，初始值为空字符串
let textRelay = BehaviorRelay<String>(value: "")

// 将 Relay 绑定到 UITextField 上，实时监听文本变化
textRelay
    .bind(to: textField.rx.text.orEmpty)
    .disposed(by: disposeBag)

// 当需要更改文本时使用 accept()
textRelay.accept("Hello, RxSwift!")
```

在这个示例中，当 `textRelay` 的值被更新时，`UITextField` 的显示内容会自动同步变化。

<br/><br/>

- 使用 `.value` 获取当前值。
- 使用 `.accept()` 更新值。
- 使用 `asObservable()` 进行订阅，监听值的变化。


<br/><br/><br/>

> <h2 id="BehaviorRelay数组">BehaviorRelay数组</h2>

```
import RxSwift
import RxCocoa

// 创建一个 Dispose Bag，用于管理订阅的生命周期
let disposeBag = DisposeBag()

// 创建一个 BehaviorRelay，持有一个字符串数组，初始值为空数组
let stringArrayRelay = BehaviorRelay<[String]>(value: [])

// 订阅 stringArrayRelay，以便每次值发生变化时接收更新
stringArrayRelay.asObservable()
    .subscribe(onNext: { array in
	    // 任何订阅了 aa 的观察者都会收到这个新的值。
        print("Array updated: \(array)")
    })
    .disposed(by: disposeBag)

// 更新数组内容
// accept() 是 BehaviorRelay 的方法，用来更新它持有的值。
stringArrayRelay.accept(["Hello", "RxSwift"])
// 输出: Array updated: ["Hello", "RxSwift"]

// 再次更新数组内容
stringArrayRelay.accept(["Hello", "RxSwift", "is", "awesome"])
// 输出: Array updated: ["Hello", "RxSwift", "is", "awesome"]
```

**输出：**

```
Array updated: ["Hello", "RxSwift"]
Array updated: ["Hello", "RxSwift", "is", "awesome"]
```

<br/><br/><br/>

> <h2 id="PublishRelay使用">PublishRelay使用</h2>



<br/><br/><br/>

> <h2 id="Reactive使用">Reactive使用</h2>

下面代码如何使用？你能解释吗？

```
// 这段代码是在扩展 Reactive，并限定只有 EEPanel 类型的对象可以使用这个扩展。
// Reactive 是 RxSwift 提供的一个协议，允许我们为特定类型添加响应式扩展，通常和 Binder 或其他操作符一起使用
extension Reactive where Base: EEPanel {
	// 在扩展里定义了一个 items 属性，它的类型是 Binder<[Expression.UIItem]>。
	// Binder 是 RxCocoa 中的一个类，专门用于在响应式绑定中简化数据流向 UI 的绑定。
	var items: Binder<[Expression.UIItem]> {
				/*
				Binder 构造函数接收两个参数：
				
				第一个参数是 self.base，即扩展的 EEPanel 实例，它是绑定的目标。
				第二个参数是一个闭包 (owner, value)，在每次数据更新时执行。owner 是 EEPanel 实例，value 是 [Expression.UIItem] 类型的数组。
				*/
	    return Binder(self.base) { owner, value in
	        owner.items = value
	        owner.cv.reloadData()
	    }
	}
}
```

上面的代码中，我们定义了一个 Reactive 扩展，用来给 EEPanel 添加一个 items 属性，它是一个 Binder<[Expression.UIItem]>。这个 items 属性允许我们将一个 [Expression.UIItem] 数据源绑定到 EEPanel 的 items 属性，并在数据更新时自动触发 reloadData 方法。

<br/><br/>


**使用**

```
owner.items = value
owner.cv.reloadData()
```

每当有新的值传递到 items 时，闭包就会执行，将 value（数据源）赋给 owner.items，然后调用 owner.cv.reloadData() 重新加载数据。在这里，cv 可能是 EEPanel 中的一个 UICollectionView 或 UITableView。

<br/>

- **Reactive 和 Binder 的作用**
	- Reactive：Reactive 是 RxSwift 提供的一种方式，用于为已有类型（如 EEPanel）添加响应式扩展。通过扩展 Reactive，可以将 RxSwift 的操作符和自定义响应式属性应用到已有类中。

	- Binder：Binder 是一种轻量级的绑定工具，常用于 UI 组件的绑定，特别适合不需要处理错误或完成事件的场景。Binder 通过定义一个处理闭包，在接收到新的值时更新 UI，简化了 UI 和数据之间的绑定。


<br/><br/>

**下面是一个完整的Demo**


```
import UIKit
import RxSwift
import RxCocoa

// 模拟数据模型
struct Expression {
    struct UIItem {
        let title: String
    }
}

// 自定义 UIView 组件 EEPanel
class EEPanel: UIView {
    
    // UICollectionView 用于显示数据项
    let cv: UICollectionView
    var items: [Expression.UIItem] = [] {
        didSet {
            cv.reloadData()
        }
    }
    
    override init(frame: CGRect) {
        // 初始化布局
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 100, height: 50)
        self.cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        
        super.init(frame: frame)
        
        addSubview(cv)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        // 使用 frame 布局 UICollectionView，使其填满整个视图
        cv.frame = self.bounds
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

// 扩展 EEPanel，使其符合 UICollectionViewDataSource
extension EEPanel: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return items.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        cell.contentView.backgroundColor = .lightGray
        
        // 添加文本标签来显示标题
        let label = UILabel(frame: cell.contentView.bounds)
        label.textAlignment = .center
        label.text = items[indexPath.item].title
        cell.contentView.addSubview(label)
        
        return cell
    }
}

// 扩展 Reactive，使 EEPanel 支持 RxSwift 响应式绑定
extension Reactive where Base: EEPanel {
    var items: Binder<[Expression.UIItem]> {
        return Binder(self.base) { owner, value in
            owner.items = value
            owner.cv.reloadData()
        }
    }
}

// 使用示例
let disposeBag = DisposeBag()

// 创建一个 EEPanel 实例
let panel = EEPanel(frame: CGRect(x: 0, y: 0, width: 300, height: 500))
panel.cv.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")

// 创建一个数据源 Observable
let itemsObservable = Observable.just([
    Expression.UIItem(title: "Item 1"),
    Expression.UIItem(title: "Item 2"),
    Expression.UIItem(title: "Item 3")
])

// 绑定数据源到 panel 的 items 属性
itemsObservable
    .bind(to: panel.rx.items)
    .disposed(by: disposeBag)
```

**数据源绑定：** 在 itemsObservable 中定义了一组数据，并使用 bind(to:) 将其绑定到 panel.rx.items。每次数据更新时，EEPanel 的 items 会自动更新，并触发 cv.reloadData() 刷新集合视图。

<br/><br/>

```
import UIKit
import RxSwift
import RxCocoa

// 模拟数据模型
struct Expression {
    struct UIItem {
        let title: String
    }
}

// 自定义 UIView 组件 EEPanel
class EEPanel: UIView {
    
    // UICollectionView 用于显示数据项
    let cv: UICollectionView
    var items: [Expression.UIItem] = [] {
        didSet {
            cv.reloadData()
        }
    }
    
    override init(frame: CGRect) {
        // 初始化布局
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 100, height: 50)
        self.cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        
        super.init(frame: frame)
        
        addSubview(cv)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        // 使用 frame 布局 UICollectionView，使其填满整个视图
        cv.frame = self.bounds
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

// 扩展 EEPanel，使其符合 UICollectionViewDataSource
extension EEPanel: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return items.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        cell.contentView.backgroundColor = .lightGray
        
        // 添加文本标签来显示标题
        let label = UILabel(frame: cell.contentView.bounds)
        label.textAlignment = .center
        label.text = items[indexPath.item].title
        cell.contentView.addSubview(label)
        
        return cell
    }
}

// 扩展 Reactive，使 EEPanel 支持 RxSwift 响应式绑定
extension Reactive where Base: EEPanel {
    var items: Binder<[Expression.UIItem]> {
        return Binder(self.base) { owner, value in
            owner.items = value
            owner.cv.reloadData()
        }
    }
}

// 使用示例
let disposeBag = DisposeBag()

// 创建一个 EEPanel 实例
let panel = EEPanel(frame: CGRect(x: 0, y: 0, width: 300, height: 500))
panel.cv.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")

// 创建一个数据源 Observable
let itemsObservable = Observable.just([
    Expression.UIItem(title: "Item 1"),
    Expression.UIItem(title: "Item 2"),
    Expression.UIItem(title: "Item 3")
])

// 绑定数据源到 panel 的 items 属性
itemsObservable
    .bind(to: panel.rx.items)
    .disposed(by: disposeBag)
```



<br/>

***
<br/><br/><br/>

> <h1 id="语言本地化">语言本地化</h1>

当然可以！下面是一个完整的例子，展示如何在 iOS 应用中使用 `NSLocalizedString` 和自定义的本地化 bundle。

### 步骤 1: 创建本地化文件

首先，确保你的项目中有本地化文件。以下是基本的步骤：

1. **添加本地化文件**：
   - 创建一个新的 `Localizable.strings` 文件（通常在你的项目中）。
   - 为不同语言添加翻译，例如英语和中文。

**Localizable.strings (English)**

```plaintext
"hello" = "Hello";
"welcome_message" = "Welcome to the app!";
```

**Localizable.strings (Chinese)**

```plaintext
"hello" = "你好";
"welcome_message" = "欢迎使用本应用！";
```

### 步骤 2: 创建自定义 Bundle

如果你需要使用自定义的 bundle 来存储本地化文件，可以按照以下步骤进行：

1. 在项目中创建一个新的文件夹，命名为 `Localizations`。
2. 将 `Localizable.strings` 文件移动到这个新创建的文件夹中。
3. 将文件夹添加到 Xcode 项目的 `Copy Bundle Resources` 中。

### 步骤 3: 定义自定义 Bundle 方法

接下来，我们需要在代码中添加一个方法来获取这个自定义的 bundle。可以通过扩展 `Bundle` 来实现：

```swift
import Foundation

extension Bundle {
    static func localizableBundle() -> Bundle {
        // 根据需要替换为你的本地化 bundle 路径
        return Bundle(path: Bundle.main.path(forResource: "Localizations", ofType: "bundle")!)!
    }
}
```

### 步骤 4: 使用 NSLocalizedString 加载本地化字符串

然后，你可以在你的视图控制器中使用 `NSLocalizedString` 来加载本地化字符串。例如：

```swift
import UIKit

class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 创建一个标签
        let label = UILabel()
        label.text = NSLocalizedString("welcome_message", bundle: Bundle.localizableBundle(), comment: "Welcome message for users")
        label.textAlignment = .center
        label.frame = CGRect(x: 0, y: 100, width: view.frame.width, height: 50)
        
        // 添加标签到视图
        view.addSubview(label)
        
        // 设置背景颜色
        view.backgroundColor = .white
    }
}
```

### 运行应用

- 当你运行应用时，它将根据设备的语言设置显示适当的欢迎信息。如果设备语言设置为中文，则标签上将显示“欢迎使用本应用！”；如果是英语，则显示“Welcome to the app!”。

### 总结

通过上述步骤，我们创建了一个完整的示例，展示了如何使用 `NSLocalizedString` 从自定义 bundle 中加载本地化字符串。这使得应用能够灵活支持多种语言，而不仅限于默认的 main bundle 中的字符串。



<br/>

***
<br/><br/><br/>

> <h1 id="ObjectMapper库">ObjectMapper库</h1>

ObjectMapper：这是一个用于将 JSON 数据映射到 Swift 对象的库。通过实现 Mappable 协议，你可以轻松地将网络响应解析为 Swift 对象。

<br/><br/><br/>

> <h2 id="接受网络数据转化为数据模型">接受网络数据转化为数据模型</h2>

在定义数据模型时，我们可以同时遵循 Mappable 和 Then 协议。Mappable 协议要求实现 mapping(map:) 方法，用于将 JSON 数据映射到对象的属性。Then 协议允许我们在初始化对象时使用链式语法。

<br/>

**定义一个模型**

```
import ObjectMapper
import Then

// 定义数据模型
class AppleModel: Mappable, Then {
    var name: String?
    var color: String?
    var weight: Double?

    required init?(map: Map) {
        // 初始化时执行的操作
    }

    // mapping 方法将 JSON 字段映射到属性
    func mapping(map: Map) {
        name    <- map["name"]
        color   <- map["color"]
        weight  <- map["weight"]
    }
}
```

<br/>

**网络请求数据**

从网络获取苹果数据，并使用 ObjectMapper 将其映射到 AppleModel 对象中。

```
import Alamofire

// 定义网络请求
func fetchAppleData() {
    let url = "https://example.com/api/apples" // 替换为你的 API 地址

    AF.request(url).responseJSON { response in
        switch response.result {
        case .success(let value):
            // 使用 ObjectMapper 解析 JSON 数据
            if let json = value as? [[String: Any]] {
                let apples = json.compactMap { AppleModel(JSON: $0) }
                // 处理解析后的数据
                for apple in apples {
                    print("Name: \(apple.name ?? ""), Color: \(apple.color ?? ""), Weight: \(apple.weight ?? 0)")
                }
            }
        case .failure(let error):
            print("Error fetching apple data: \(error)")
        }
    }
}

```

<br/> <br/>

使用 Then 进行链式初始化

```
let apple = AppleModel()
    .then {
        $0.name = "Granny Smith"
        $0.color = "Green"
        $0.weight = 150.0
    }

print("Apple name: \(apple.name ?? ""), color: \(apple.color ?? ""), weight: \(apple.weight ?? 0)")
```

在这个示例中，我们定义了一个 AppleModel 类，它遵循 Mappable 和 Then 协议。通过 ObjectMapper 实现的 mapping(map:) 方法，可以将 JSON 数据映射到对象属性。而 Then 协议允许我们在初始化对象时使用链式语法，使代码更简洁。

<br/><br/>

> <h3 id="不继承Then协议,如何使用Then的链式语法"> 不继承Then协议,如何使用Then的链式语法</h3>

在 Swift 中，使用链式语法通常依赖于实现一些特定的协议。对于 `Then` 协议来说，它是为了简化对象的初始化和属性设置而设计的。

如果你没有让 `AppleModel` 继承 `Then` 协议，依然可以使用链式语法，但需要手动实现链式设置的方法。这是通过返回 `self` 实现的，例如：

### 使用链式语法的实现示例

如果不继承 `Then` 协议，可以通过自定义初始化器或方法来实现链式语法。

#### 1. 定义数据模型

```swift
import ObjectMapper

class AppleModel: Mappable {
    var name: String?
    var color: String?
    var weight: Double?

    required init?(map: Map) {
        // 初始化时执行的操作
    }

    // mapping 方法将 JSON 字段映射到属性
    func mapping(map: Map) {
        name    <- map["name"]
        color   <- map["color"]
        weight  <- map["weight"]
    }
    
    // 自定义方法以支持链式语法
    @discardableResult
    func setName(_ name: String?) -> AppleModel {
        self.name = name
        return self
    }

    @discardableResult
    func setColor(_ color: String?) -> AppleModel {
        self.color = color
        return self
    }

    @discardableResult
    func setWeight(_ weight: Double?) -> AppleModel {
        self.weight = weight
        return self
    }
}
```

#### 2. 使用链式语法

现在你可以使用这些自定义的方法进行链式调用：

```swift
let apple = AppleModel()
    .setName("Granny Smith")
    .setColor("Green")
    .setWeight(150.0)

print("Apple name: \(apple.name ?? ""), color: \(apple.color ?? ""), weight: \(apple.weight ?? 0)")
```

### 总结

即使没有实现 `Then` 协议，你仍然可以通过返回 `self` 的自定义方法来实现类似的链式设置效果。虽然 `Then` 协议简化了这个过程，但通过自定义方法也可以达到类似的目的。这使得对象的初始化更加灵活且可读性强。






<br/>

***
<br/><br/><br/>

> <h1 id="Then库">Then库</h1>
Then：这是一个简化对象初始化和配置的库，允许你在创建对象时链式调用属性设置。这样可以提高代码的可读性和简洁性。



<br/>

***
<br/><br/><br/>

> <h1 id="PixDefines库">PixDefines库</h1>

PixDefines 是一个用于简化和组织代码的库，通常在一些图形或图像处理相关的项目中使用。


<br/><br/><br/>

> <h2 id="PixNameSpace使用">PixNameSpace使用</h2>


PixNameSpace 是该库中提供的一个命名空间，用于管理常量、类型和其他定义，使得代码更具可读性和可维护性。下面将详细讲解 PixNameSpace 的作用及其用法。

<br/>

`PixDefines` 是一个用于简化和组织代码的库，通常在一些图形或图像处理相关的项目中使用。`PixNameSpace` 是该库中提供的一个命名空间，用于管理常量、类型和其他定义，使得代码更具可读性和可维护性。下面将详细讲解 `PixNameSpace` 的作用及其用法。

<br/>

- **1.`PixNameSpace` 的作用**
	- **组织代码**：通过命名空间，开发者可以将相关的常量和类型放在一起，避免名称冲突，提升代码的结构化程度。
	- **提高可读性**：命名空间使得代码更易读，特别是在大型项目中，可以让其他开发者快速了解各部分的功能。
	- **管理常量**：可以在命名空间中定义各种常量，例如颜色、尺寸等，以便在整个项目中统一使用，避免硬编码。

<br/>

- **2.使用 `PixNameSpace`**

下面是一个如何定义和使用 `PixNameSpace` 的示例。

- **2.1 定义 `PixNameSpace`**

首先，我们需要在项目中定义 `PixNameSpace`，并在其中添加一些常量和类型。

```swift
import Foundation

// 定义命名空间
enum PixNameSpace {
    // 常量定义
    enum Colors {
        static let primaryColor = "#FF5733"
        static let secondaryColor = "#33CFFF"
    }
    
    enum Dimensions {
        static let defaultMargin: CGFloat = 16.0
        static let defaultPadding: CGFloat = 8.0
    }
    
    // 其他可能的定义
    struct ImageNames {
        static let logo = "app_logo"
        static let placeholder = "placeholder_image"
    }
}
```

在这个例子中，我们定义了一个 `PixNameSpace` 枚举，包含了 `Colors`、`Dimensions` 和 `ImageNames` 这几个子命名空间，分别用于管理颜色常量、尺寸常量和图片名称。

<br/><br/>

- 2.2 使用 `PixNameSpace`

在使用这些定义时，可以方便地引用命名空间中的常量，增强代码的可读性。

```swift
import UIKit

class ExampleViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 使用 PixNameSpace 中的常量
        view.backgroundColor = UIColor(hex: PixNameSpace.Colors.primaryColor)
        
        let margin = PixNameSpace.Dimensions.defaultMargin
        let button = UIButton(frame: CGRect(x: margin, y: margin, width: 200, height: 50))
        button.backgroundColor = UIColor(hex: PixNameSpace.Colors.secondaryColor)
        button.setTitle("Click Me", for: .normal)
        
        view.addSubview(button)
    }
}

// UIColor 扩展，用于支持 hex 颜色
extension UIColor {
    convenience init?(hex: String) {
        var hexString = hex.trimmingCharacters(in: .whitespacesAndNewlines)
        hexString = hexString.replacingOccurrences(of: "#", with: "")

        var rgb: UInt64 = 0
        Scanner(string: hexString).scanHexInt6464(&rgb)

        let red = CGFloat((rgb >> 16) & 0xFF) / 255.0
        let green = CGFloat((rgb >> 8) & 0xFF) / 255.0
        let blue = CGFloat(rgb & 0xFF) / 255.0

        self.init(red: red, green: green, blue: blue, alpha: 1.0)
    }
}
```

<br/><br/>

- **3.总结**

通过定义 `PixNameSpace`，我们能够清晰地组织常量和类型，提升代码的可读性和可维护性。在实际项目中，使用命名空间是一个很好的编程实践，特别是在团队合作和大型项目开发中。

- **组织结构**：所有相关的常量都集中在一个地方，便于管理和查找。
- **避免命名冲突**：即使在大型项目中，也可以有效避免不同模块之间的命名冲突。
- **增强可读性**：通过使用命名空间，代码的意图更清晰，其他开发者更容易理解。 


<br/><br/><br/>

> <h3 id="关键字where约束和范型的联合">关键字where约束和范型的联合</h3>


可以看下使用关键字where约束和范型的一个联合Demo，如下：
















