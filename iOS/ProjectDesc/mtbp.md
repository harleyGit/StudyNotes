> <h1 id=""></h1>
- [**基础**](#基础)
	- [立即执行闭包](#立即执行闭包)
- [**RxSwift使用**](#RxSwift使用)
	- [Observer](#Observer)
		- [Binder关于UI绑定](#Binder关于UI绑定)
	- [BehaviorRelay使用](#BehaviorRelay使用)
		- [BehaviorRelay数组](#BehaviorRelay数组)
	- [**PublishRelay使用**](#PublishRelay使用)
	- [**Reactive使用**](#Reactive使用)
	- [Signal使用](#Signal使用)
- [**语言本地化**](#语言本地化)
- [**ObjectMapper库**](#ObjectMapper库)
	- [接受网络数据转化为数据模型](#接受网络数据转化为数据模型)
		- [不继承Then协议,如何使用Then的链式语法](#不继承Then协议,如何使用Then的链式语法)
- [**Then库**](#Then库)
- [PixDefines库](#PixDefines库)
	- [PixNameSpace使用](#PixNameSpace使用)
		- [关键字where约束和范型的联合](#关键字where约束和范型的联合)
- [CACurrentMediaTime媒体时间](#CACurrentMediaTime媒体时间)
- [NSObjectProtocol协议](./../Swift/基础.md#NSObjectProtocol协议)
- [枚举关联值](./../Swift/基础.md#枚举关联值)
- [**UI动画效果**](#UI动画效果)
	- [弹窗隐藏](#弹窗隐藏)
	- [CAKeyframeAnimation详解](./动画.md#CAKeyframeAnimation详解)





<br/>

***
<br/><br/><br/>

> <h1 id="基础">基础</h1>

<br/><br/><br/>

> <h2 id="立即执行闭包">立即执行闭包</h2>

```
Swift中这个 

lazy var path = { AClass.Path() }()

什么语法？ 怎么回事
```

[请看这里](./../Swift/基础.md#立即执行闭包)




<br/>

***
<br/><br/><br/>

> <h1 id="RxSwift使用">RxSwift使用</h1>


<br/><br/><br/>

> <h2 id="Observer">Observer</h2>

<br/><br/>

> <h3 id="Binder关于UI绑定">Binder关于UI绑定</h3>

`Binder` 是 `RxSwift` 中的一种绑定机制，它被设计为专门用于 UI 绑定，确保线程安全，同时帮助我们简洁地实现数据流绑定到 UI 元素的操作。`Binder` 是一种 observer，它只会监听并处理主线程上的事件，适合用于更新 UI 元素的属性，比如文本、颜色等。

<br/>

**`Binder` 的核心概念**

`Binder` 是 `RxSwift` 中的一种类型安全的 observer，专门用于绑定 UI 组件。其设计主要考虑了以下几点：

1. **线程安全**：`Binder` 会确保事件在主线程上执行，适合用于 UI 更新。
2. **不会传播错误**：`Binder` 无法处理 error 事件，它只处理 `next` 事件，避免了将错误传播到 UI 层。
3. **简洁的绑定方式**：`Binder` 可以将 observable 的输出直接绑定到 UI 元素的属性，简化了代码逻辑。

<br/>

 **`Binder` 的基本使用**

`Binder` 的构造函数接收两个参数：
1. `target`：表示绑定的目标对象，通常是 UI 元素。
2. `binding`：表示事件触发时执行的闭包操作，接收目标对象 `target` 和新事件值。

<br/>

**使用示例**

假设我们有一个 `UILabel`，我们希望当 observable 发出新值时，自动更新其文本内容。

```swift
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {
    let label = UILabel()
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()

        let observable = Observable.just("Hello, RxSwift!")

        // 绑定 observable 的输出到 label 的文本
        observable.bind(to: label.rx.text)
            .disposed(by: disposeBag)
    }
}
```

在这个例子中，我们直接将 observable 的输出绑定到 `label.rx.text`。`label.rx.text` 就是一个 `Binder<String?>`，用于简洁地实现 observable 数据流到 UI 的绑定。

<br/>

### 自定义 `Binder`

如果我们想自定义一个 `Binder`，例如实现一个用于控制按钮启用状态的 `Binder`，可以这样实现：

```swift
extension Reactive where Base: UIButton {
    var isEnabled: Binder<Bool> {
        return Binder(self.base) { button, isEnabled in
            button.isEnabled = isEnabled
        }
    }
}
```

然后在使用时，可以将 observable 直接绑定到按钮的 `isEnabled` 属性：

```swift
let isEnabledObservable = Observable.just(true)

isEnabledObservable
    .bind(to: myButton.rx.isEnabled)
    .disposed(by: disposeBag)
```

<br/>

**`Binder` 的构造函数详解**

构造一个 `Binder` 一般遵循以下模式：

```swift
Binder<T>(self.base) { target, value in
    // 更新 target 的 UI 属性
}
```

- `target`：绑定的 UI 元素。
- `value`：来自 observable 的新值，传递给 `Binder` 用于更新 UI。

例如，以下 `Binder` 将 `Int` 类型的 observable 绑定到 `UILabel` 上，将整数转换为文本：

```swift
extension Reactive where Base: UILabel {
    var textFromInt: Binder<Int> {
        return Binder(self.base) { label, value in
            label.text = "\(value)"
        }
    }
}
```

然后可以使用 `Observable<Int>` 来绑定 `UILabel`：

```swift
let intObservable = Observable.just(42)
intObservable
    .bind(to: label.rx.textFromInt)
    .disposed(by: disposeBag)
```

### 总结

`Binder` 是 `RxSwift` 中用于 UI 绑定的 observer，可以简洁、安全地将数据流绑定到 UI 控件的属性上。其优点包括线程安全、不会处理错误事件以及简化的绑定操作。`Binder` 的构造函数接受 `target` 和 `binding`，可以灵活定义目标对象和更新逻辑，适用于大部分 UI 数据绑定场景。


<br/><br/>

> <h2 id="binder的按钮点击事件">binder的按钮点击事件</h2>

```
let button: UIButton = UIButton()

button.rx.controlEvent(.touchUpInside)
            .bind(to: rx.edit)
            .disposed(by: rx.disposeBag)

var edit: Binder<Void> {
        return Binder(self.base) { owner, action in
           print("-------------")
        }
    }
```

**代码解析：**

- `.rx.controlEvent(.touchUpInside): ` RxSwift 中用于监听按钮点击事件的方法。.touchUpInside 表示按钮被点击并在其内部释放（即通常的点击效果）。

- `bind(to: rx.edit)`: 将 btnEdit 的点击事件绑定到 edit 属性，这里 edit 是一个 Binder<Void> 类型，它定义了在点击事件发生时要执行的操作。

- `disposed(by: rx.disposeBag)`: 通过 disposeBag 管理订阅的生命周期，确保在 disposeBag 被释放时，订阅也会自动取消，防止内存泄漏。

- `edit` 是一个 Binder<Void>，用于定义点击事件的响应。

- `Binder(self.base)`: Binder 是 RxSwift 中的一种类型安全的包装器，用于定义 UI 事件和观察者。self.base 是 edit 的持有对象，即 Binder 的上下文环境，它指向当前对象。

- `{ owner, action in ... }`: 闭包参数中的 owner 是 self.base（持有者），action 是点击事件触发时传入的参数（Void 类型）。在这里，点击触发时会执行 print("-------------")，在控制台输出一个分隔线以示事件触发。

<br/>

**`Binder<Void>`**是什么？ 为什么定义成这个类型？

`Binder<Void>` 类型用于表示一个不携带任何数据的事件绑定。当我们不关心事件的具体值时，使用 Void 类型就足够了。这种模式特别适用于按钮点击等操作，因为点击事件本身不需要附带任何数据，只要能够触发相应的响应操作即可。

<br/>

- **为什么使用 `Binder<Void>`？**
	- 简化绑定逻辑：对于按钮点击等事件，数据内容并不重要，重要的是事件的发生。Void 表示“什么也不需要传递”，这种设计简化了代码，只关注触发操作，而无需处理额外的数据。

	- 提高代码的可读性：使用 Void 明确表示事件没有负载，帮助开发者快速理解事件不携带数据。例如，如果看到 Binder<String>，则意味着绑定时会传递一个 String 类型的值；而 Binder<Void> 则表明事件仅用于触发，没有数据。

	- 避免冗余数据传递：在不需要传递数据的情况下，用 Void 避免了无意义的数据传输和处理。这样可以减少内存和 CPU 开销，因为 Void 只是一个空事件，不占用额外资源。


这里的 rx.edit 被定义为 Binder<Void>，因为点击事件本身并不需要数据，只是为了触发响应操作。




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

PublishRelay：这是 RxSwift 中的一种特殊的 Relay（即不会终止的 Observable），它能够发布事件，但不会将事件缓存下来。因此，只有订阅者能够实时接收到事件，并且新的订阅者不会接收到之前已经发布的事件。

<br/>

假设我们有一个按钮点击时需要更新的 selected 状态。我们可以通过 BtnItem 的 selected 属性来管理选中状态

- **定义数据模型和 ViewModel**

```
import RxSwift
import RxCocoa

// 数据模型 BtnItem
class BtnItem: ItemType {
    var type: Int
    var selected: Bool

    init(type: Int, selected: Bool = false) {
        self.type = type
        self.selected = selected
    }
}

// 定义 ViewModel
class ViewModel {
    // 用于发布按钮点击事件
    let clickAction = PublishRelay<BtnItem>()
    
    // 订阅并处理点击事件
    init() {
        clickAction.subscribe(onNext: { item in
            // 处理点击事件：这里可以更改选中状态
            item.selected.toggle()
            print("Button item type: \(item.type), selected: \(item.selected)")
        }).disposed(by: disposeBag)
    }

    private let disposeBag = DisposeBag()
}
```

- **1.代码解析**

- **1.1** `‌clickAction.accept(item)`： 这里将 item 事件发布给 clickAction，所有订阅 clickAction 的观察者都会收到这个 item，并触发相应的响应操作。

这样做的目的是：使用`clickAction` 响应按钮点击的 Relay。当按钮点击时，可以调用 `clickAction.accept(item)`，将当前按钮的 BtnItem 数据对象作为事件发送出去。这样，每次点击按钮时，事件就会被发布，订阅者接收到该事件后可以进行进一步的处理，比如更改 UI 或者更新状态。

<br/>

- **1.2** 在vm中处理订阅的按钮点击事件

要处理按钮点击事件，可以订阅 clickAction。这样，当 accept 方法被调用时，所有订阅 clickAction 的代码块都会被执行。

在上述代码中，我们在 viewModel 中处理逻辑，通过订阅 clickAction 来实现点击事件的响应。

<br/><br/>

- **在视图控制器中绑定按钮点击**

```
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {
    let viewModel = ViewModel()
    let disposeBag = DisposeBag()
    let button = UIButton(type: .system)
    var item = BtnItem(type: 1) // 初始化 BtnItem

    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupButton()
        
        // 将按钮点击绑定到 clickAction
        button.rx.tap
            .bind { [weak self] in
                guard let self = self else { return }
                self.viewModel.clickAction.accept(self.item)
            }
            .disposed(by: disposeBag)
    }
    
    func setupButton() {
        button.setTitle("Click Me", for: .normal)
        button.frame = CGRect(x: 100, y: 100, width: 100, height: 50)
        view.addSubview(button)
    }
}
```

**解析：**

- `button.rx.tap：`这是一个 RxCocoa 提供的扩展，用于将按钮点击事件转化为 RxSwift 的 ControlEvent<Void>，允许你对按钮点击事件进行响应。
- `bind { ... }：`每次点击按钮时，闭包内的代码会被执行，将当前的 BtnItem 实例（item）传递给 clickAction。
- 在 **`ViewModel`** 中：clickAction 被订阅，当按钮点击时触发 accept 方法，触发 onNext 闭包。在这里，我们切换了 selected 状态，模拟了一个选中和取消选中的效果。

<br/><br/><br/>

更近一步，是否可以用下面的代码替换subscribe订阅呢？

```
clickAction.withUnretained(self)
            .flatMap { owner, item00 in
                owner.vm.callMethod(item00)
            }
            .map { _ in () }
            .bind(to: rx.reload)
            .disposed(by: rx.disposeBag)
```


**代码解析**

- 1.**`withUnretained(self)`**

	- **作用**：`withUnretained(self)` 操作符在事件流中附加当前对象（`self`，即 `owner`），并在每次事件发生时检查 `self` 是否仍然存在，从而避免了内存泄漏。
	- **工作方式**：将 `self` 和事件值（这里是 `BtnItem` 实例 `item00`）作为参数传递给后续闭包中的操作。若 `self` 已经释放，则不会继续传递，避免因循环引用而导致的内存泄漏。

<br/>

- 2.**`flatMap`**

	- **作用**：`flatMap` 用于将一个值映射到另一个 `Observable` 对象中，通常用于触发异步操作或调用其他需要返回 `Observable` 的方法。
	- **这里的作用**：在闭包 `{ owner, item00 in }` 中，调用 `owner.vm.callMethod(item00)`，处理点击事件。这可能是一个异步操作，也可能会返回一个与更新 `UI` 相关的信号（比如需要刷新列表的信号）。

<br/>

-  3.**`map { _ in () }`**

	- **作用**：将事件流的输出转化为 `Void` 类型（即空值），以便与后续需要 `Void` 类型的 `Observer`（比如 `rx.reload`）绑定。
	- **用途**：由于 `callMethod(item00)` 可能返回的并不是 `Void` 类型的数据（比如某个结果或处理状态），`map { _ in () }` 可以将其转化为 `Void` 类型，确保类型符合后续的绑定需求。

<br/>

- 4.**`bind(to: rx.reload)`**

	- **作用**：将事件流的输出绑定到 `rx.reload`，触发 `UICollectionView` 或其他视图的刷新操作。
	- **这里的意义**：在 `callMethod` 执行完成后，发出一个空事件并绑定到 `rx.reload`，这样视图就会根据 `rx.reload` 的响应来更新显示内容。

<br/>

- **5.`disposed(by: rx.disposeBag)`**

	- **作用**：保证事件绑定在 `rx.disposeBag` 释放时自动解除，防止因为重复绑定导致的资源占用或内存泄漏。

<br/>


**为什么选择这种写法？**

这种写法通过链式调用使代码更具可读性，并且减少了不必要的 `subscribe(onNext:)`，避免了强引用问题（通过 `withUnretained(self)` 确保 `self` 的生命周期管理）。此外，使用 `bind(to:)` 也更符合 RxSwift 的响应式编程风格，使得操作意图更清晰（即事件触发后直接影响 `rx.reload`）。

<br/>

**总结**

- 这种写法主要优势在于：
	- **减少内存泄漏**：避免循环引用，确保对象在事件流中安全使用。
	- **链式风格**：使用链式 API 提高代码可读性和易维护性。
	- **响应式思维**：直接绑定到 `rx.reload`，更加直观地表达事件与视图更新的关联。

<br/><br/>

在使用上述的代码后，我们发现有一个`‌rx.reload`，这个表示刷新什么呢？ 是什么在刷新？

<br/>

在这个代码片段中，`rx.reload` 作为事件绑定的目标，用于触发 `UICollectionView` 或其他视图的刷新操作。让我们更详细地看下如何触发刷新：

<br/>

 **`rx.reload` 的作用**

`rx.reload` 是 `UICollectionView` 的一个自定义 `Binder`，用于监听事件并调用视图的刷新方法（例如 `reloadData()`）。在 RxSwift 中，我们可以使用 `Binder` 来创建自定义属性，使得它们能够响应事件流中的数据变化，从而更新视图。

<br/>

**实现 `rx.reload`**

假设我们要为 `UICollectionView` 创建一个 `rx.reload`，我们可以通过扩展 `Reactive` 为 `UICollectionView` 增加这个自定义的 `Binder`：

```swift
import RxSwift
import RxCocoa
import UIKit

extension Reactive where Base: UICollectionView {
    var reload: Binder<Void> {
        return Binder(self.base) { collectionView, _ in
            collectionView.reloadData()
        }
    }
}
```

<br/>

- **解析**

	- **`extension Reactive where Base: UICollectionView`**：为 `UICollectionView` 扩展 `Reactive`，使我们可以创建自定义的 `rx` 属性。
	- **`var reload: Binder<Void>`**：声明 `rx.reload`，类型为 `Binder<Void>`，这意味着它将响应 `Void` 类型的事件。
	- **`Binder(self.base) { collectionView, _ in collectionView.reloadData() }`**：在事件发生时调用 `collectionView.reloadData()`，从而刷新整个集合视图。

<br/>

**触发刷新**

当 `clickAction` 发布事件，流经 `.bind(to: rx.reload)` 时，`rx.reload` 会监听到这个事件，并执行 `reloadData()` 方法，从而刷新 `UICollectionView` 的数据。

<br/>

**示例应用**

以下是完整代码，展示如何使用 `rx.reload` 来绑定刷新操作：

```swift
class ViewController: UIViewController {
    let disposeBag = DisposeBag()
    let viewModel = ViewModel()
    let collectionView = UICollectionView(frame: .zero, collectionViewLayout: UICollectionViewFlowLayout())
    var item = BtnItem(type: 1)

    override func viewDidLoad() {
        super.viewDidLoad()

        // 假设点击按钮时调用 clickAction
        viewModel.clickAction.accept(item)
        
        // 订阅 clickAction 以触发视图的刷新
        viewModel.clickAction
            .withUnretained(self)
            .flatMap { owner, item00 in
                owner.viewModel.callMethod(item00) // 调用方法进行数据更新
            }
            .map { _ in () }
            .bind(to: collectionView.rx.reload) // 绑定到 rx.reload
            .disposed(by: disposeBag)
    }
}
```

这样，每次 `clickAction` 有事件发出时，`rx.reload` 就会被触发，`UICollectionView` 将会执行 `reloadData()` 来刷新内容。




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

> <h1 id="Signal使用">Signal使用</h1>


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

```
import UIKit

// 这里我们简单实现一个包装器，可以用来扩展 UICollectionView 的功能。
// 定义一个 PixWrapper 泛型类
public class PixWrapper<Base> {
    public let base: Base
    
    public init(_ base: Base) {
        self.base = base
    }
}

// 方便使用的扩展
public extension UICollectionView {
    var pix: PixWrapper<UICollectionView> {
        return PixWrapper(self)
    }
}

```

- **`PixWrapper<Base>`** 是一个泛型类。这里的 Base 是一个类型占位符，可以代表任何类型。泛型类允许我们在使用 PixWrapper 时指定具体的类型。
- **`base`** 是 PixWrapper 的一个属性，它的类型是 Base。构造器 init 接受一个参数，初始化 base 属性。

<br/>

 **扩展 UICollectionView**
 
接下来，在 PixWrapper 的扩展中添加一些与 UICollectionView 相关的方法，例如设置默认的布局。

```
public extension PixWrapper where Base: UICollectionView {
    // 设置默认的布局
    func setDefaultLayout() {
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .vertical
        layout.minimumLineSpacing = 10
        layout.minimumInteritemSpacing = 10
        base.collectionViewLayout = layout
    }
    
    // 其他自定义方法
    func registerCell<T: UICollectionViewCell>(cellType: T.Type) {
        base.register(cellType, forCellWithReuseIdentifier: String(describing: cellType))
    }
    
    func dequeueCell<T: UICollectionViewCell>(cellType: T.Type, indexPath: IndexPath) -> T {
        return base.dequeueReusableCell(withReuseIdentifier: String(describing: cellType), for: indexPath) as! T
    }
}
```

<br/>


```
public extension PixWrapper where Base: UICollectionView {

}
```

&emsp; 是 Swift 中的一种扩展（extension）写法，旨在为 UICollectionView 添加一些功能或方法。具体来说，它是一个泛型扩展，用于在 PixWrapper 的上下文中增加对 UICollectionView 的支持。

- **代码解析**
	- `public extension：`表示这是一个公开的扩展，可以在模块外部访问。
	- `PixWrapper：`这是一个包装类，通常用于封装原始对象（在本例中是 UICollectionView），以便在其基础上添加额外的功能。
	- where Base: UICollectionView：这部分限制了扩展只能应用于 Base 为 UICollectionView 的情况下。这意味着在这个扩展中定义的方法只能被 UICollectionView 或其子类调用。

<br/><br/>

- **where 子句的作用**

**`where Base: UICollectionView `**是一个类型约束，它指定了该扩展只适用于 Base 是 UICollectionView 或其子类的情况。换句话说，只有当 PixWrapper 的类型参数 Base 满足这一条件时，扩展中的方法和功能才会生效。

**举例说明**

例如，如果你有如下代码：

```
let myCollectionViewWrapper = PixWrapper<UICollectionView>(UICollectionView())
```
此时 Base 被推断为 UICollectionView，因此你可以使用以下扩展中的方法：


```
public extension PixWrapper where Base: UICollectionView {
    func setDefaultLayout() {
        // 这里可以使用 base 属性，它是 UICollectionView 类型
        let layout = UICollectionViewFlowLayout()
        base.collectionViewLayout = layout
    }
}
```

但是，如果你尝试创建一个 PixWrapper 实例，传递一个非 UICollectionView 类型的对象，例如 UILabel，则你将无法使用这个扩展，因为 Base 不是 UICollectionView。

<br/><br/>

- **`<Base>` 是什么？**

- `<Base>` 不是一个枚举，而是一个泛型参数。这是 Swift 的一种特性，允许你在类、结构体或函数中使用占位符类型。它在具体使用时会被替换为实际类型。例如：

	- 在 PixWrapper<Base> 中，Base 是一个泛型参数，具体类型可以在实例化时指定，比如 UICollectionView、UIView 或其他类型。
	- 当你声明一个泛型类型时，它可以在其内部使用，也可以通过条件约束（如 where 子句）来限定。

<br/>

- **总结**

	- 泛型类：PixWrapper<Base> 是一个泛型类，允许使用任意类型作为 Base。
	- where 子句：指定扩展只在 Base 为 UICollectionView 或其子类时有效，这提供了类型安全和灵活性。
	- 泛型参数：<Base> 是一种类型占位符，用于允许类型的灵活性，使得类、结构体或函数能够适应多种数据类型。




<br/><br/>

**使用示例**

```
class MyCollectionViewController: UIViewController, UICollectionViewDelegate, UICollectionViewDataSource {
    
    private var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 初始化 UICollectionView
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: UICollectionViewFlowLayout())
        collectionView.delegate = self
        collectionView.dataSource = self
        
        // 使用 PixWrapper 扩展的方法
        collectionView.pix.setDefaultLayout()
        collectionView.pix.registerCell(cellType: MyCollectionViewCell.self)

        view.addSubview(collectionView)
    }
    
    // UICollectionViewDataSource
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return 20 // 示例数据量
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell: MyCollectionViewCell = collectionView.pix.dequeueCell(cellType: MyCollectionViewCell.self, indexPath: indexPath)
        cell.configure(with: "Item \(indexPath.item + 1)") // 配置单元格
        return cell
    }
}

// 自定义 UICollectionViewCell
class MyCollectionViewCell: UICollectionViewCell {
    private let titleLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        contentView.addSubview(titleLabel)
        titleLabel.frame = contentView.bounds
        titleLabel.textAlignment = .center
        titleLabel.textColor = .black
        contentView.backgroundColor = .lightGray
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func configure(with text: String) {
        titleLabel.text = text
    }
}
```

在这个示例中，我们创建了一个 PixWrapper 用于封装 UICollectionView，并在其中添加了一些方法来简化对 UICollectionView 的使用。通过 PixWrapper，我们能够使用链式调用来设置布局、注册和 dequeue 单元格，增强了代码的可读性和可维护性。


<br/>

***
<br/><br/><br/>

> <h1 id="CACurrentMediaTime媒体时间">CACurrentMediaTime媒体时间</h1>

`CACurrentMediaTime` 是 Core Animation 框架中的一个函数，用来获取当前媒体时间（media time）。它返回的是自系统启动以来的秒数，具有高精度和单调递增的特性。这意味着它不会受系统时间的修改（比如用户手动调整时间）影响，非常适合用于动画和计时的场景。

### 主要用途

1. **动画计时**：`CACurrentMediaTime` 经常用于 Core Animation 的动画同步。它可以精确控制动画的开始时间、持续时间等，确保动画平滑进行。
   
2. **高精度计时**：适用于需要精确时间控制的任务，比如性能测试、游戏开发等，适合用来测量代码块的执行时间。

3. **相对时间的参考**：因为它返回的时间是基于系统启动的，而不是绝对的日期时间，所以在多个线程或不同系统间做时间同步比较有用。

### 示例

例如，如果你想要测量一个操作的执行时间，可以使用 `CACurrentMediaTime`：

```swift
let startTime = CACurrentMediaTime()
// 执行一些操作
let endTime = CACurrentMediaTime()
let executionTime = endTime - startTime
print("操作耗时: \(executionTime) 秒")
```

这种方式可以在 Core Animation 和其他需要精确时间的场景中广泛使用。



<br/>

***

<br/><br/><br/>

> <h1 id="UI动画效果">UI动画效果</h1>

<br/><br/><br/>

> <h2 id="弹窗隐藏">弹窗隐藏</h2>

```
// 显示弹窗view
func showPopView() {
    contentView.layer.removeAllAnimations()
    let anim = CABasicAnimation(keyPath: "transform.scale")
    anim.fromValue = 0.2
    anim.toValue = 1.0
    anim.timingFunction = .init(name: .easeInEaseOut)
    anim.duration = 0.3
    anim.fillMode = .forwards
    anim.isRemovedOnCompletion = false
    contentView.layer.add(anim, forKey: "scale")
}
```

<br/>

```
func hidePopView() {
    contentView.layer.removeAllAnimations()
    let anim = CABasicAnimation(keyPath: "transform.scale")
    anim.fromValue = 1.0
    anim.toValue = 0.0
    anim.timingFunction = .init(name: .easeInEaseOut)
    anim.duration = 0.3
    anim.fillMode = .forwards
    anim.isRemovedOnCompletion = false
    contentView.layer.add(anim, forKey: "scale")
}
```

这段代码实现了一个缩放动画，用于在 `contentView` 上播放从正常大小（1.0 倍）缩小到 0.0 倍的动画效果，同时对动画的执行进行了特定的配置。以下是代码逐步详解：

---

### 3. **`contentView.layer.removeAllAnimations()`**
- **功能**：移除 `contentView` 图层上所有当前正在执行的动画。
- **原因**：
  - 防止之前的动画没有完成或冲突。
  - 确保接下来的动画从一个干净的状态开始。

---

### 4. **`let anim = CABasicAnimation(keyPath: "transform.scale")`**
- **创建动画**：
  - `CABasicAnimation` 是用于动画化图层属性的基本动画类。
  - `keyPath` 设置为 `"transform.scale"`，指定对缩放比例属性进行动画。
  - 动画会让 `contentView` 缩放到指定大小。

---

### 5. **`anim.fromValue = 1.0`**
- 设置动画的起始值。
  - 图层从 `1.0` 开始缩放（即原始大小）。

---

### 6. **`anim.toValue = 0.0`**
- 设置动画的目标值。
  - 图层会缩小到 `0.0`（完全不可见）。

---

### 7. **`anim.timingFunction = .init(name: .easeInEaseOut)`**
- 设置动画的时间曲线。
  - `.easeInEaseOut` 表示动画开始和结束时较慢，中间加速。
  - 让动画看起来更自然。

---

### 8. **`anim.duration = 0.3`**
- **设置持续时间**：
  - 动画的播放时间为 0.3 秒。

---

### 9. **`anim.fillMode = .forwards`**
- **设置动画填充模式**：
  - `.forwards` 表示动画完成后，图层保持动画结束时的状态（即缩小到 0.0）。

---

### 10. **`anim.isRemovedOnCompletion = false`**
- **保持动画效果**：
  - 设置为 `false`，表示动画完成后不要从图层中移除。
  - 结合 `fillMode = .forwards`，确保图层在动画完成后仍然保持最终的缩放状态。

---

### 11. **`contentView.layer.add(anim, forKey: "scale")`**
- 将动画添加到 `contentView` 的图层中。
- **`forKey: "scale"`**：
  - 这是动画的标识符，可以用来管理动画，比如后续获取或移除它。

---

### 动画效果
1. 动画开始时，`contentView` 缩放从正常大小（1.0 倍）逐渐变小。
2. 在 0.3 秒内，最终缩放到 0.0 倍，即完全缩小到看不见。
3. 动画完成后，`contentView` 的缩放状态保持在 0.0，不会恢复到原始大小。

---

### 注意点
1. **`isRemovedOnCompletion = false`** 与 `fillMode = .forwards` 配合：
   - 确保动画完成后状态不被重置。
2. **动画移除逻辑**：
   - 如果在动画完成后需要还原状态，可以手动移除动画或通过其他代码重置。
3. **防止重复触发**：
   - 利用 `isShowing` 的状态控制，避免同一动画被重复启动。

---

### 改进建议
- **动画结束后的回调**：可以利用 `CAAnimationDelegate`，在动画完成时做额外处理，例如清理状态：
  ```swift
  anim.delegate = self
  ```

- **复位状态**：
  如果需要在动画结束后恢复到原始大小，可以手动设置图层的 `transform` 属性：
  ```swift
  contentView.layer.transform = CATransform3DIdentity
  ```













