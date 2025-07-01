> <h1 id=""></h1>
- [**‌ Swinject解耦合注入**](#Swinject解耦合注入)
- [注入类加入初始化参数](#注入类加入初始化参数)
	- [解决:可选参数为nil报错](#解决:可选参数为nil报错)




<br/><br/><br/>

***
<br/>

> <h1 id= "Swinject解耦合注入">Swinject解耦合注入</h1>

有一个 `AClass` 类，它需要多个初始化参数，如 `ViewModel`、`CodeInfo`、`Info`、`String` 和 `DeviceBleCodeVCType`。我们通过 `Swinject` 进行依赖注入，在 `Home` 类中进行解析。

- **步骤 1: 定义类和依赖**

首先定义 `AClass` 类和它所需要的依赖项。

```swift
// 模拟的依赖类
class ViewModel {}
class CodeInfo {}
class Info {}
enum DeviceBleCodeVCType {
    case None, SomeType
}

// AClass 类
class AClass {
    let viewModel: ViewModel
    let qrcodeInfo: CodeInfo?
    let userWifiInfo: Info?
    let productId: String
    let type: DeviceBleCodeVCType
    
    init(viewModel: ViewModel, qrcodeInfo: CodeInfo? = nil,
         userWifiInfo: Info? = nil, productId: String, type: DeviceBleCodeVCType = .None) {
        self.viewModel = viewModel
        self.qrcodeInfo = qrcodeInfo
        self.userWifiInfo = userWifiInfo
        self.productId = productId
        self.type = type
    }
}
```

***
**步骤 2: 注册依赖**

接下来，创建一个 `AService` 类，该类负责将所有依赖项（包括 `AClass`）注册到 `Swinject` 容器中。

```swift
import Swinject

// AService 类，注册依赖项
class AService: ContainerServiceProtocol {
    let container: Container
    
    init() {
        container = Container()

        // 注册 ViewModel、CodeInfo、Info 等依赖
        container.register(ViewModel.self) { _ in ViewModel() }
        container.register(CodeInfo.self) { _ in CodeInfo() }
        container.register(Info.self) { _ in Info() }

        // 注册 AClass，使用工厂方法注入依赖项
        container.register(AClass.self) { r in
            let viewModel = r.resolve(ViewModel.self)!
            let qrcodeInfo = r.resolve(CodeInfo.self)   // 可选的参数
            let userWifiInfo = r.resolve(Info.self)     // 可选的参数
            let productId = "someProductId"             // 你可以传入具体的值
            let type: DeviceBleCodeVCType = .None       // 默认值
            
            return AClass(viewModel: viewModel, qrcodeInfo: qrcodeInfo, userWifiInfo: userWifiInfo, productId: productId, type: type)
        }
    }
}
```

***
- **步骤 3: 在 `Home` 类中解析依赖**

现在在 `Home` 类中，使用 `Swinject` 容器来解析 `AClass`，并实例化它。

```swift
// Home 类，使用依赖注入解析 AClass
class Home: ModuleProtocol, DIProtocol {
    let container: Container
    
    // 初始化方法接收一个容器实例
    init(container: Container) {
        self.container = container
    }
    
    // 设置方法，通过解析容器来获取 AClass 的实例
    func setup() {
        if let aClassInstance = container.resolve(AClass.self) {
            // 使用 aClassInstance 进行接下来的操作
            print("AClass instance initialized: \(aClassInstance)")
        } else {
            print("Failed to resolve AClass")
        }
    }
}
```

- **步骤 4: 使用 `Home` 和 `AService`**

最后，你需要确保在应用程序中通过 `AService` 提供容器实例，然后传递给 `Home` 类进行使用。

```swift
// 应用程序中初始化服务和模块
let aService = AService()         // 创建 AService 实例
let home = Home(container: aService.container)  // 创建 Home 实例，并传入容器
home.setup()  // 调用 setup 方法，解析 AClass
```

- **详细流程解释**

	1. **AClass 的初始化**： `AClass` 的初始化方法需要多个参数。我们在 `AService` 类的 `container.register` 方法中注册了 `AClass`，并通过工厂闭包将它所依赖的参数自动解析并传递给 `AClass` 的初始化方法。
	
	2. **Home 类解析依赖**： 在 `Home` 类的 `setup()` 方法中，使用 `container.resolve(AClass.self)` 来解析并获取 `AClass` 的实例。由于在 `AService` 中已经注册了 `AClass` 和它所依赖的所有类（如 `ViewModel`、`CodeInfo` 等），`Swinject` 会自动注入这些依赖并实例化 `AClass`。
	
	3. **容器传递**： `Home` 类的 `container` 是从 `AService` 中传递过来的，它在初始化时就可以访问到所有注册的依赖。这样，`Home` 类就可以顺利使用 `AClass` 的实例，而不需要手动创建它。


***
<br/><br/><br/>
> <h2 id="注入类加入初始化参数">注入类加入初始化参数</h2>

当你传递的参数中，某些值是字符串类型，并且初始化方法期望接收多个参数时，`Swinject` 会按照你传递的顺序匹配这些参数。对于 `Swinject` 解析方法来说，只要你没有传入某个参数（尤其是可选的或有默认值的参数），它会使用默认值，而不会强制要求你传递所有参数。

***
- **示例情况**

假设 `AClass` 有如下的初始化方法：

```swift
class AClass {
    let viewModel: ViewModel
    let productId: String
    let type: DeviceBleCodeVCType
    let extraInfo: String?
    
    init(viewModel: ViewModel, productId: String, type: DeviceBleCodeVCType = .None, extraInfo: String? = nil) {
        self.viewModel = viewModel
        self.productId = productId
        self.type = type
        self.extraInfo = extraInfo
    }
}
```

在上面的例子中，`AClass` 需要一个 `viewModel` 和一个 `productId`，但是 `type` 和 `extraInfo` 有默认值。

如果你在传递参数时，只传递了 `viewModel` 和 `productId`，`Swinject` 会自动使用 `type` 和 `extraInfo` 的默认值。

***

- **传递参数时的问题**

	- **1.如果你只传递一个参数（比如 `viewModel`）：**

`Swinject` 会按照参数的顺序将你传入的 `viewModel` 与初始化方法中的第一个参数 `viewModel` 匹配。接下来，它会查找其余的参数，并使用默认值（比如 `type` 和 `extraInfo`）。

- **2.如果你传递了 `viewModel` 和 `productId`，但是没有传递 `type` 和 `extraInfo`：**

	* `Swinject` 会按顺序匹配并使用默认值。
	* `type` 会使用 `DeviceBleCodeVCType.None`（因为这是它的默认值）。
	* `extraInfo` 会使用 `nil`（因为它是一个可选类型）。

- **代码示例**

```swift
// AClass 初始化方法
class AClass {
    let viewModel: ViewModel
    let productId: String
    let type: DeviceBleCodeVCType
    let extraInfo: String?
    
    init(viewModel: ViewModel, productId: String, type: DeviceBleCodeVCType = .None, extraInfo: String? = nil) {
        self.viewModel = viewModel
        self.productId = productId
        self.type = type
        self.extraInfo = extraInfo
    }
}

// 在容器中注册 AClass
container.register(AClass.self) { (r, viewModel: ViewModel, productId: String, type: DeviceBleCodeVCType, extraInfo: String?) in
    return AClass(viewModel: viewModel, productId: productId, type: type, extraInfo: extraInfo)
}

// 传递参数解析
let viewModel = ViewModel()
let productId = "Product123"

// 只传递了 viewModel 和 productId，后面的参数使用默认值
if let aClassInstance = container.resolve(AClass.self, arguments: viewModel, productId, DeviceBleCodeVCType.SomeType, "Some extra info") {
    // 这里传入了参数 type 和 extraInfo，后续的参数会根据顺序匹配
    print("AClass initialized with \(aClassInstance.productId) and type \(aClassInstance.type)")
} else {
    print("Failed to resolve AClass")
}
```

- **解析：**

	1. **参数传递顺序**：`Swinject` 会根据你传递的参数顺序与初始化方法中的参数类型进行匹配。所以当你传递了 `viewModel` 和 `productId` 后，`Swinject` 会自动将它们分别对应到初始化方法中的第一个和第二个参数。
	2. **默认值**：对于后面的参数 `type` 和 `extraInfo`，如果你没有传递它们，`Swinject` 会使用 `init` 中定义的默认值。例如，`type` 会是 `DeviceBleCodeVCType.None`，`extraInfo` 会是 `nil`。

- **如果你只传递了部分字符串类型的参数**

假设你只传递了一个字符串参数（例如 `productId`）并且缺少其他类型的参数，你依然可以按顺序传递，而不需要为 `type` 和 `extraInfo` 传递值。

```swift
let productId = "CustomProduct" // 假设你只传递了 productId
- if let aClassInstance = container.resolve(AClass.self, arguments: viewModel, productId, DeviceBleCodeVCType.SomeType) {
    print("AClass initialized with productId: \(aClassInstance.productId) and type: \(aClassInstance.type)")
} else {
    print("Failed to resolve AClass")
}
```

***

- **处理混合类型（字符串和其他类型）**

如果你有一些是字符串类型，其他是更复杂的类型，依然可以根据类型和顺序传递参数。比如：

```swift
// 假设你要传入的类型如下：
let viewModel = ViewModel()
let productId = "Product123"
let extraInfo: String? = "Extra information"

// 如果你的初始化方法中存在多个默认值（如 `type` 和 `extraInfo`），可以继续传递相应的值。
if let aClassInstance = container.resolve(AClass.self, arguments: viewModel, productId, DeviceBleCodeVCType.SomeType, extraInfo) {
    print("AClass initialized with productId: \(aClassInstance.productId) and extraInfo: \(aClassInstance.extraInfo ?? "No extra info")")
} else {
    print("Failed to resolve AClass")
}
```

- **总结：**

	* 传递参数时，只要顺序正确，并且每个参数的类型匹配，`Swinject` 会自动解析并将它们传递给初始化方法。
	* 如果某些参数没有传入，`Swinject` 会使用默认值（如果有的话），而不会导致错误。
	* 传递字符串、可选值或者复杂类型时，确保传递的类型和顺序与初始化方法中的一致。


***
<br/><br/><br/>
> <h2 id="解决:可选参数为nil报错"> 解决:可选参数为nil报错</h2>

报错 `Fatal error: Unexpectedly found nil while unwrapping an Optional value` 是因为 `Swinject` 在解析 `AClass` 时出现了某个 `Optional` 为 `nil`，但你没有做正确的处理，导致强制解包时崩溃。

<br/>

- **解决问题的关键：**

	- 1.**解析参数时**，你可能没有为 `qrcodeInfo` 和 `userWifiInfo` 提供默认值，或者在容器解析时没有传递正确的参数。

	- 2.**`Swinject` 解析时未能正确地注入必需的参数**，或者你没有正确处理可选类型的解包。

<br/>

- **详细分析和修改步骤：**

	- 1.**`AClass` 的初始化方法问题**：
   `AClass` 的初始化方法有一些参数是可选的（如 `qrcodeInfo` 和 `userWifiInfo`），并且有默认值。如果你没有传入这些值，它们将默认为 `nil`。你需要确保容器注册和调用时处理了这些可选值。

	- 2.**`container.resolve` 的使用**：
   在 `getAClass` 方法中，你调用了 `container.resolve(AClass.self, arguments: qrcodeInfo, userWifiInfo, productID, DeviceBleCodeVCType.BluetoothDevice)`。其中 `qrcodeInfo` 和 `userWifiInfo` 是可选的，而 `productID` 和 `VCType` 是必传的。`Swinject` 可能在解析时出现问题，导致强制解包失败。

<br/>

- **问题分析：**

	- 1.**解包问题**：
   `container.resolve(AClass.self, arguments: qrcodeInfo, userWifiInfo, (productID ?? "-.-"), DeviceBleCodeVCType.BluetoothDevice)` 中 `qrcodeInfo` 或 `userWifiInfo` 有可能是 `nil`，如果你没有做好防护（例如检查它们是否为 `nil`），在 `AClass` 中解包时可能会崩溃。

	- 2.**容器注册问题**：
   你的容器注册代码如下：

```swift
container.register(AClass.self) { r in
   let viewModel = r.resolve(AClassViewModel.self)!
   return AClass(viewModel: viewModel, productId: "")
}
```

   这里你仅仅传入了 `viewModel` 和一个空的 `productId`。如果 `AClass` 的初始化方法中其他参数没有正确传递，`Swinject` 解析时会出现问题。

<br/>

- **解决方案：**

	- 1.**调整容器注册**：确保你在容器注册时传递了所有必须的参数，并且考虑到可选参数的情况。

	- 2.**处理可选参数**：确保你在 `getAClass` 中正确处理了 `qrcodeInfo` 和 `userWifiInfo` 为 `nil` 的情况，避免强制解包。

<br/>

- **改进后的代码示例：**

	- 1.**注册 `AClass`**：
   在容器注册时，需要确保为 `qrcodeInfo` 和 `userWifiInfo` 提供正确的默认值（可以是 `nil` 或其他合适的值）。同时确保 `productId` 和 `VCType` 是必须传递的。

   ```swift
   container.register(AClass.self) { r, qrcodeInfo, userWifiInfo, productId, vcType in
       let viewModel = r.resolve(AClassViewModel.self)!
       return AClass(viewModel: viewModel, qrcodeInfo: qrcodeInfo, userWifiInfo: userWifiInfo, productId: productId, type: vcType)
   }
   ```

<br/>

2.**调整 `getAClass` 方法**：
   在 `getAClass` 方法中，使用 `resolve` 时提供正确的参数，确保在传递可选参数时检查 `nil`，或者在没有传递时使用默认值。

   ```swift
   public static func getAClass(qrcodeInfo: CodeInfo?, userWifiInfo: WiFiInfo?, productID: String?) -> UIViewController {
       // 确保提供默认值，避免为 nil 时解包失败
       let vc = container.resolve(AClass.self, arguments: qrcodeInfo ?? nil, userWifiInfo ?? nil, productID ?? "-.-", DeviceBleCodeVCType.BluetoothDevice)!
       return vc
   }
   ```

3. **修改 `vc` 解析时的参数传递**：
   你需要传递正确的参数给 `AClass` 的初始化方法。尤其是确保所有的可选参数（如 `qrcodeInfo` 和 `userWifiInfo`）在没有传递时默认为 `nil`。

   ```swift
   let vc = ArgusAppHome.getAClass(qrcodeInfo: nil, userWifiInfo: nil, productID: self.bleIdentifier)
   ```

