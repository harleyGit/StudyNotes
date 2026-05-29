> <h1 id= ""></h1>
- [**Swift高级用法**](#Swift高级用法)
	- [状态机枚举使用](#状态机枚举使用)
	- [Swift关于OC方法名映射](#Swift关于OC方法名映射)
	- [枚举打印输出](#枚举打印输出)
	- [多代理+弱引用管理](#多代理+弱引用管理)
		- [compactMap等价方法和解读、举例](#compactMap等价方法和解读、举例)
  		- [数组contains解读](#数组contains解读)
	- [类中的访问权限](#类中的访问权限)
	- [异步任务Task](#异步任务Task)
		- [异步网络请求+UI更新小Demo](#异步网络请求+UI更新小Demo)
		- [Task的6种任务场景Demo](#Task的6种任务场景Demo)
	- [Swift中的Codable是什么](#Swift中的Codable是什么)
	- [\`default \`关键字转义用法](#关键字转义用法)
	- [范型中` T? where T: Decodable `](#T?whereT:Decodable)
	- [` DispatchQueue(label:) `队列](#DispatchQueue(label:)队列)
	- [编译器插值#line、#file](#编译器插值#line、#file)
	- [withCheckedContinuation桥接旧的回调式API到async/await模式](#withCheckedContinuation桥接旧的回调式API到async/await模式)
	- [拼接公共域名+接口路径](#拼接公共域名+接口路径)
	- [自定义简易观察者](#自定义简易观察者)
	- [字符串转枚举](#字符串转枚举)
	- [oc对象转换成结构体的工厂方法](#oc对象转换成结构体的工厂方法) 
	- [可用配网过滤](#可用配网过滤)
	- [更新/替换list中相同设备的数据-map、first方法](#更新/替换list中相同设备的数据-map、first方法)
	- [数组集合使用协议hash比较](#数组集合使用协议hash比较)
- [**线程安全**](#线程安全)
	- [barrier实现线程安全](#barrier实现线程安全)
- [**UI组件**](#UI组件)
	- [输入框字数限制](#输入框字数限制)
	- [label宽度自适应](#label宽度自适应)
	- [弹窗播放视频](#弹窗播放视频)
	- [一段代码搞定播放音频，优化内存释放](#一段代码搞定播放音频，优化内存释放)
	- [**线程安全**](#线程安全)
		- [指定队列判断](#指定队列判断)
- [**系统蓝牙库CoreBluetooth**](#系统蓝牙库CoreBluetooth)
	- [精确获取当前蓝牙开启状态](#精确获取当前蓝牙开启状态)
	- [轮询连接蓝牙设备](#轮询连接蓝牙设备)
	- [BLE扫描SessionID](#BLE扫描SessionID)
	- [一段代码检测蓝牙状态，优化内存泄露](#一段代码检测蓝牙状态，优化内存泄露)
	- [蓝牙入口](#蓝牙入口)
		- [蓝牙权限状态获取](#蓝牙权限状态获取)
- [Wi-Fi连接](#Wi-Fi连接)
	- [App内添加Wi-Fi配置](#App内添加Wi-Fi配置)
- [**依赖注入库Swinject**](#依赖注入库Swinject)
- [**实时音视频通信WebRTC**](#实时音视频通信WebRTC)
    - [ICE服务器了解](#ICE服务器了解)
- [**日志库SwiftyBeaver**](#日志库SwiftyBeaver)
	- [完整的日志封装类](#完整的日志封装类)
- **常用库**
	- [**‌alamofire库**](#**‌alamofire库**)
		- [拦截器](#拦截器)
		- [json数据本地库解析](#json数据本地库解析) 
	- [CocoaMQTT5库](#CocoaMQTT5库)
- [**本地数据库RealmSwift**](#本地数据库RealmSwift)
	- [RealmSwift简单实用Demo](#RealmSwift简单实用Demo)
- [GCDAsyncSocket及时通讯](#GCDAsyncSocket及时通讯)
- [**Git操作**](#Git操作)
	- [添加忽略文件](#添加忽略文件)
- [**CocoaPod安装配置**](#CocoaPod安装配置)
- [**概念**](#概念)
	- [p2p连接](#p2p连接)
- [**工程Bug**](#工程Bug)
	- [SYM软件针对于.ips文件进行符号话.DSYM](https://github.com/zqqf16/SYM)
	- [析构调用闭包导致crash](#析构调用闭包导致crash)
	- [线程堆栈分析报告如何查看？](#线程堆栈分析报告如何查看？)
	- [网址缓存导致无法加载](#网址缓存导致无法加载)




<br/><br/><br/>

***
<br/>

> <h1 id= "Swift高级用法">Swift高级用法</h1>


***
<br/><br/><br/>
># <h2 id="状态机枚举使用">[状态机枚举使用](../Swift/swift基础.md#状态机枚举使用)</h2>


***
<br/><br/><br/>
> <h2 id="Swift关于OC方法名映射">Swift关于OC方法名映射</h2>


***
<br/><br/><br/>
> <h2 id="枚举打印输出">枚举打印输出</h2>

`CustomStringConvertible` 用于**控制对象被打印时显示的字符串**。在 SDK 中给 `enum`、`State`、`ErrorCode`、`DeviceType`、`Mode` 等类型实现它，可以让日志更稳定、更清晰。

**工业级推荐写法：**

```swift
import Foundation

// MARK: - OOB Mode

@objc
public enum Mode: Int, CustomStringConvertible {

    /// 自动模式
    case AUTO

    /// 手动模式
    case MANUAL

    /// 默认模式
    case DEFAULT

    /// Swift print()
    public var description: String {
        return stringValue
    }

    /// Swift / OC 通用
    @objc
    public var stringValue: String {
        switch self {
        case .AUTO:
            return "AUTO"

        case .MANUAL:
            return "MANUAL"

        case .DEFAULT:
            return "DEFAULT"
        }
    }
}
```

这里同时保留 `description` 和 `stringValue`：

| 属性 | 用途 |
| ---- | ---- |
| `description` | Swift `print()` / 字符串插值自动使用 |
| `stringValue` | Swift / OC 都可以显式调用 |

---
<br/>

## 为什么要实现 CustomStringConvertible？

默认情况下打印枚举：

```swift
let mode = Mode.AUTO
print(mode)
```

输出可能是：

```text
Mode(rawValue: 0)
```

也可能是：

```text
AUTO
```

具体输出取决于 Swift 版本和 enum 情况，不够稳定。

实现 `CustomStringConvertible` 后：

```swift
print(mode)
```

输出稳定为：

```text
AUTO
```

在日志里：

```swift
AKLog("mode = \(mode)")
```

输出会从不友好的：

```text
mode = Mode(rawValue: 1)
```

变成：

```text
mode = MANUAL
```

<br/>

## 会自动触发 description 的场景

| 场景 | 示例 |
| ---- | ---- |
| `print()` | `print(mode)` |
| 字符串插值 | `"\(mode)"` |
| 日志系统 | `NSLog("%@", "\(mode)")` |
| Debug 输出 | Xcode 控制台 |

`CustomStringConvertible` 类似其他语言里的 `toString()`：

| 语言 | 对应能力 |
| ---- | ---- |
| Swift | `CustomStringConvertible` |
| Java | `toString()` |
| Kotlin | `toString()` |
| OC | `description` |

---
<br/>

## SDK 中推荐实现的类型

| 类型 | 推荐 |
| ---- | ---- |
| `enum` | 必须 |
| `State` | 必须 |
| `ErrorCode` | 必须 |
| `DeviceType` | 必须 |
| `Mode` | 必须 |

例如：

```swift
AddDeviceType
AddState
ErrorCode
BleConnectState
```

这些类型经常出现在 BLE、AP、LAN、OOB Mode、State、ErrorCode 等日志里，实现后日志会更舒服。

---
<br/>

## 它和 @objc 没关系

`CustomStringConvertible` 是纯 Swift 协议，OC 不认识它。OC 对应的是：

```objc
[obj description]
```

Swift 对应的是：

```swift
description
```

如果不声明协议，只写一个普通 `description` 属性：

```swift
@objc
public enum Mode: Int {

    case AUTO

    public var description: String {
        return "AUTO"
    }
}
```

那么：

```swift
mode.description
```

可以输出：

```text
AUTO
```

但：

```swift
print(mode)
```

不会自动调用你的 `description`。因为 Swift 不知道这个普通属性是用于文本描述的。

---
<br/>

## description 来自协议要求

如果你写：

```swift
public var description: String
```

并希望 `print(mode)` 自动使用它，就必须让枚举遵守：

```swift
CustomStringConvertible
```

也就是：

```swift
@objc
public enum Mode: Int, CustomStringConvertible
```

否则 `description` 只是普通属性，不会参与 `print()` 的文本描述逻辑。

---
<br/>

## 更高级的调试协议

除了：

```swift
CustomStringConvertible
```

还有：

```swift
CustomDebugStringConvertible
```

区别：

| 协议 | 用途 |
| ---- | ---- |
| `description` | 用户输出 |
| `debugDescription` | 调试输出 |

一般 SDK 只实现 `description` 就够了。

---
<br/>

## 最终效果

Swift：

```swift
print(mode)
```

输出：

```text
AUTO
```

OC：

```objc
mode.stringValue
```

输出：

```text
AUTO
```

这个方案最适合 SDK、跨平台、日志系统、Swift + OC 混编场景。

***
<br/><br/><br/>
> <h2 id="多代理+弱引用管理">多代理+弱引用管理</h2>

# HGWeakStruct文件

```swift
/// 结构体,内部保存的AnyObject为弱引用
public struct HGWeakStruct<T: AnyObject> {
    public weak var value: T?
    
    public init(value: T? = nil) {
        self.value = value
    }
}
```

# HGManager文件

```swift

public protocol HGManagerDelegate: NSObjectProtocol {
	@objc optional func didConnectDevice(identifier: String?)
}

public class HGManager: NSObject, NSCopying, NSMutableCopying, NSSecureCoding {

private var delegates: [HGWeakStruct<HGManagerDelegate>] = []


private func notifyDelegates(_ handler: (HGManagerDelegate) -> Void) {

	let currentDelegates = self.delegateQueue.sync {
		self.delegates.removeAll { $0.value == nil }
		return self.delegates.compactMap { $0.value }
	}
	currentDelegates.forEach(handler)
}
```

# 调用

## 下面的代码啥意思？

```swift

self.delegates.removeAll { $0.value == nil || $0.value === delegate }
elf.delegates.contains(where: { $0.value === delegate })
self.notifyDelegates { $0. didConnectDevice(identifier: identifier) }
```

***
<br/><br/>

上面的代码片段是一个非常典型的：

```text
Swift 多代理（multicast delegate） + 弱引用管理
```

- 这个在：
	* BLE SDK
	* 网络 SDK
	* 事件总线
	* Notification 分发

里非常常见。

***
<br/>

上述代码片段核心目的是：

```text
一个 manager 可以同时通知多个 delegate，
并且不会造成循环引用。
```

我按底层逻辑给你完整拆解。

---
<br/>

# HGWeakStruct 是干嘛的

```swift 
public struct HGWeakStruct<T: AnyObject> {
    
    public weak var value: T?
    
    public init(value: T? = nil) {
        self.value = value
    }
}
```

## 作用：把对象弱引用包装起来

## 因为：Array 不能直接存 weak,比如：

```swift id="u1ph4l"
weak var delegates: [HGManagerDelegate]
```
Swift 不允许，所以必须包一层：`AKWeakStruct`


***
<br/>

# 为什么要 weak?
# 避免循环引用。

```text
Manager 强引用 delegates
delegate 又强引用 manager
```

就：**永远无法释放**

## BLE SDK 特别容易出现这种问题。

所以：`weak var value`

意味着：**delegate释放后，这里自动变成 nil**

***
<br/>

# delegates 数组结构

```swift id="9d4mz7"
private var delegates: [HGWeakStruct<HGManagerDelegate>] = []
```

实际内存大概：

```sh
delegates
 ├── weak delegateA
 ├── weak delegateB
 └── weak delegateC
```

不是强持有。


***
<br/>

## === 是啥意思

这是：**引用类型对象地址比较**，不是值比较。

**比如：**

```swift id="dx2qz6"
obj1 === obj2
```
表示：**是不是同一个对象实例**

<br/>

```swift id="mjlwmv"
let a = NSObject()
let b = a
let c = NSObject()
```

结果：

```swift id="87p1jf"
a === b // true
a === c // false
```

---
<br/>

# removeAll 那段啥意思

```swift id="ohgsy7"
self.delegates.removeAll {
    $0.value == nil || $0.value === delegate
}
```

**解读：**
- 删除：
  - 1. 已经释放的 delegate
  - 2. 当前指定 delegate

## 等价：

```swift id="0pb26t"
self.delegates.removeAll { item in
    
    if item.value == nil {
        return true
    }
    
    if item.value === delegate {
        return true
    }
    
    return false
}
```

<br/>

### 举例

```sh
# 假设
delegates:
[A, B, nil, C]

# 现在
remove(delegate: B)

# 执行
removeAll {
    $0.value == nil || $0.value === B
}

# 结果
[A, C]
```
因为：
* nil 被清掉
* B 被移除


<br/><br/>
> <h3 id="数组contains解读">数组contains解读</h3>
- 遍历数组，
- 只要有一个满足条件就返回 true

# contains(where:) 啥意思

```swift id="qbjg7k"
// 这种写法是 尾随闭包 + $0 简写
// 检查数组里是否已经存在该 delegate
self.delegates.contains(where: {
	// 是否是同一个对象实例， 比较内存地址
	$0.value === delegate
})

// 完整写法：
self.delegates.contains(where: { item in
    item.value === delegate
})

// 更完整写法
self.delegates.contains(where: { (item: HGWeakStruct<HGManagerDelegate>) -> Bool in
    
    return item.value === delegate
})
```
**等价：**

```swift id="n2m0lg"
for item in delegates {

    if item.value === delegate {
        return true
    }
}

return false


// 或者如下：
var isExist = false

for item in self.delegates {

    if item.value === delegate {

        isExist = true
        break
    }
}
```

---
<br/>

# notifyDelegates 核心机制


```swift id="c2u84h"
private func notifyDelegates(
    _ handler: (HGManagerDelegate) -> Void
) {

    let currentDelegates = self.delegateQueue.sync {

        self.delegates.removeAll { $0.value == nil }

        return self.delegates.compactMap { $0.value }
    }

    currentDelegates.forEach(handler)
}
```
## 线程安全

```swift id="svjlwm"
self.delegateQueue.sync
```

说明：**delegates 访问是串行队列保护的**
**避免：**
* 多线程同时增删
* 崩溃
* 数组越界

<br/>

## 清理已释放对象

```swift id="j8cepo"
self.delegates.removeAll { $0.value == nil }

// 最开始
[A, nil, C]

// 删除空值后
[A, C]
```
**因为 weak 已自动 nil。**

<br/><br/>
> <h3 id="compactMap等价方法和解读、举例">compactMap等价方法和解读、举例</h3>


## 取出有效 delegate

- **`compactMap`：**
	- 自动过滤 nil,并解包
 	- 映射（map）


```swift id="0isjlwm"
self.delegates.compactMap { $0.value }

// 完整写法
self.delegates.compactMap { item in
    return item.value
}

// 更完整写法
self.delegates.compactMap { (item: HGWeakStruct<HGManagerDelegate>) -> HGManagerDelegate? in
    
    return item.value
}
```

**等价**

```swift
var result: [AKBleProcessManagerDelegate] = []

for item in self.delegates {

    if let delegate = item.value {
        result.append(delegate)
    }
}
```

<br/>

**举例**

```sh
// 取出之前
[weak A, weak nil, weak C]

// compactMap执行后
[A, C]
```
<br/>

## 为什么先复制 currentDelegates? 
## 非常重要。这里：

```swift id="zv1xk5"
let currentDelegates = ...
```
是为了：

```text
避免遍历时数组被修改
```
否则：

```text
forEach 时 remove delegate
可能崩溃
```
### 这是经典写法。

---
<br/>

## forEach(handler)

```swift id="4q3qdc"
currentDelegates.forEach(handler)

// 等价
for delegate in currentDelegates {
    handler(delegate)
}
```

***
<br/>

## 最终调用
- 通知所有 delegate：
- 蓝牙连接成功了

```swift id="mxw40f"
self.notifyDelegates {
    $0.didConnectBluetoothPeripheral?(
        identifier: identifier
    )
}

// 完整写法
// 更完整写法：将 { delegate in 换成 {(delegate: AKBleProcessManagerDelegate) in
self.notifyDelegates { delegate in
    
    delegate.didConnectBluetoothPeripheral?(
        identifier: identifier
    )
}
```
**等价**

```swift 
for delegate in currentDelegates {

    delegate.didConnectDevice?(
        identifier: identifier
    )
}
```

---
<br/>

## 总结：整个架构图

```text
HGManager
    │
    ├── weak delegateA
    ├── weak delegateB
    └── weak delegateC
```

### 事件发生：
- **蓝牙连接成功**,
- **Manager：**
	- 遍历所有 `delegate`
  	- 逐个通知
  	- 调用：`delegate.didConnectDevice()`



***
<br/><br/><br/>
> <h2 id="类中的访问权限">类中的访问权限</h2>

若有`BleKit`库中**AClass**属性 ： 

```swift
public struct AClass {
    private(set) var isRegistered: Bool = false
}
```

* 在本模块外，**完全看不到** `isRegistered`（因为属性本身默认是 `internal`）
* 在本模块内，可以读，但不能写


**想让外部模块可以读取，但是不可以修改，如何做？**

<br/>

`private(set)` 的意思是：

* **getter**（读）权限跟属性修饰符一致
* **setter**（写）权限限制在 `private`


---
<br/>

**想让外部模块能读但不能写**

就要加上 **访问级别**：

```swift
public struct AClass {
    public private(set) var isRegistered: Bool = false
    
    public init(isRegistered: Bool = false) {
        self.isRegistered = isRegistered
    }
}
```

这样效果就是：

* 在模块外可以 `device.isRegistered` ✅
* 在模块外不能改 `device.isRegistered = true` ❌
* 只有在 `AKThingBleDeviceModel` 的内部方法里能修改 ✅

<br/>

```swift
let device = AKThingBleDeviceModel()
print(device.isRegistered)  // ✅ 可以读

device.isRegistered = true  // ❌ 报错：'isRegistered' setter is inaccessible
```


[**还有其他用法，请看这里**](./../Swift/关键字.md#模块访问权限举例)


***
<br/><br/><br/>
> <h2 id="异步任务Task">异步任务Task</h2>
在 Swift 中，`Task { }` 是用来创建 **异步任务（concurrent task）** 的方式，是 Swift Concurrency（Swift 并发）的一部分。

---

## ✅ 一句话解释：

`Task { }` 可以在**任何线程中启动一个异步任务**，尤其是你想在非 async 方法中执行 `await` 操作时，它特别有用。

---

## 🔧 语法简介：

```swift
Task {
    // 这里可以写 await 的异步代码
}
```

它会立即开始执行，并且是并发运行的。

---

## 🔍 示例讲解：

```swift
import Foundation

func fetchDataFromServer() async -> String {
    // 模拟网络请求
    try? await Task.sleep(nanoseconds: 2 * 1_000_000_000)  // 2秒
    return "✅ 数据从服务器获取成功"
}

func startNormalFunction() {
    print("🔥 准备开始异步任务")

    Task {
        let result = await fetchDataFromServer()
        print("🎉 异步任务结果：\(result)")
    }

    print("📌 Task 创建之后，函数继续执行（不会阻塞）")
}
```

### 🧠 输出说明：

```bash
🔥 准备开始异步任务
📌 Task 创建之后，函数继续执行（不会阻塞）
🎉 异步任务结果：✅ 数据从服务器获取成功
```

说明：`Task` 是**并发执行**的，主线程继续走，不会卡在 `await`。

---

## 📌 `Task` 的主要作用场景

| 场景                            | 描述             |
| ----------------------------- | -------------- |
| 在非 async 函数中使用 async/await    | 用 `Task` 包一层即可 |
| 在 UI 层调用异步网络请求                | 避免阻塞主线程        |
| 在后台执行耗时操作                     | 提高响应性          |
| 调用 actor、结构化并发等 Swift 5.5+ 特性 | Task 是核心组件之一   |

---

## ✅ Task 的更复杂用法（进阶）

```swift
Task(priority: .background) {
    let imageData = await downloadImage()
    await MainActor.run {
        imageView.image = UIImage(data: imageData)
    }
}
```

* `priority:` 设置任务优先级（`.high`, `.low`, `.background`, `.userInitiated` 等）
* `await MainActor.run` 是指切换回主线程更新 UI

---

## 🔒 捕获错误（带 throws）

```swift
Task {
    do {
        let data = try await riskyNetworkCall()
        print("获取成功: \(data)")
    } catch {
        print("❌ 网络请求失败: \(error)")
    }
}
```

---

## 🧠 小贴士

| 知识点       | 内容                               |
| --------- | -------------------------------- |
| Task 立即启动 | 无需你 `.start()`                   |
| 任务是并发的    | 多个任务可同时运行                        |
| 不阻塞当前线程   | 与传统 GCD 不同，Task 支持结构化并发和 `await` |

---

## 总结口诀（记忆小法）：

> **“异步 await 不好使？Task 包一层全解决。”**

<br/><br/>
<h3 ID="异步网络请求+UI更新小Demo">异步网络请求+UI更新小Demo</h3>
好的！以下是一个完整的 Swift 示例，展示如何使用 `Task {}` 进行 **异步网络请求并更新 UI**，适合你初学 Swift Concurrency 时参考。

---

## ✅ 项目目标：

* 模拟一个网络请求（使用 `URLSession`）
* 使用 `async/await` 在后台下载数据
* 成功后切回主线程更新 UI（`UILabel`）

---

## 🛠 示例：异步请求 + UI 更新

```swift
import UIKit

class ViewController: UIViewController {

    let label = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        fetchDataAndUpdateUI()
    }

    func setupUI() {
        label.text = "⏳ 加载中..."
        label.textAlignment = .center
        label.font = UIFont.systemFont(ofSize: 18)
        label.frame = view.bounds
        view.addSubview(label)
    }

    // 主入口，创建异步任务
    func fetchDataAndUpdateUI() {
        Task {
            do {
                let result = try await fetchServerMessage()
                // 回到主线程更新 UI
                await MainActor.run {
                    label.text = result
                }
            } catch {
                await MainActor.run {
                    label.text = "❌ 网络请求失败：\(error.localizedDescription)"
                }
            }
        }
    }

    // 异步函数：模拟网络请求
    func fetchServerMessage() async throws -> String {
        let url = URL(string: "https://httpbin.org/get")!
        let (data, _) = try await URLSession.shared.data(from: url)
        let string = String(data: data, encoding: .utf8) ?? "⚠️ 无法解析数据"
        return "✅ 请求成功：\n\(string.prefix(80))..."
    }
}
```

---

## 🧠 关键点解释：

| 代码片段                             | 含义                 |
| -------------------------------- | ------------------ |
| `Task { ... }`                   | 启动一个异步任务（非阻塞）      |
| `try await fetchServerMessage()` | 执行真正的异步网络请求        |
| `await MainActor.run { ... }`    | **切回主线程更新 UI**（必须） |
| `label.text = result`            | 显示结果到屏幕            |

---

## ✅ 运行效果：

打开 App 会立即显示 `⏳ 加载中...`，几秒后变为：

```
✅ 请求成功：
{
  "args": {},
  "headers": {
    "Accept": ...
```

---

## ✅ 你可以学到的并发关键技能：

* 使用 `Task` 启动异步任务
* 使用 `async/await` 让代码更清晰
* 正确使用 `MainActor.run` 更新 UI
* 如何处理 `try await` 的错误



<br/><br/>
> <h3 id="Task的6种任务场景Demo">Task的6种任务场景Demo</h3>
当然可以！下面是对你提到的 Swift 中 `Task {}` 使用场景的**详细分类 + 示例**，每一个都配了简单的示例代码，方便你学习和实践。

---

## ✅ 场景 1：**在非 `async` 函数中使用 `await`**

> 如果你在普通函数中需要调用 `await` 方法，直接用 `Task {}` 包裹即可。

### 示例：

```swift
func buttonTapped() {
    print("用户点击了按钮")

    Task {
        let result = await fetchData()
        print("拿到数据：\(result)")
    }
}

func fetchData() async -> String {
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    return "数据加载完成"
}
```

---

## ✅ 场景 2：**避免主线程阻塞（例如 UI 操作）**

> 在 UI 层不要直接调用耗时操作，`Task` 可让你把耗时逻辑放到后台执行，然后再切回主线程更新 UI。

### 示例：

```swift
Task {
    let imageData = await downloadImage()
    await MainActor.run {
        imageView.image = UIImage(data: imageData)
    }
}
```

---

## ✅ 场景 3：**执行后台任务或耗时操作**

> 比如缓存计算、文件处理、大量 JSON 解析等，不应在主线程执行。

### 示例：

```swift
Task {
    let result = await calculateLargeDataSet()
    print("计算完成：\(result)")
}
```

---

## ✅ 场景 4：**结构化并发（TaskGroup）**

> 当你需要并发执行多个任务并等待它们都完成时，可以使用 `TaskGroup`，`Task {}` 是它的基础。

### 示例：

```swift
Task {
    await withTaskGroup(of: String.self) { group in
        group.addTask { await fetchNews() }
        group.addTask { await fetchWeather() }

        for await result in group {
            print("收到数据：\(result)")
        }
    }
}
```

---

## ✅ 场景 5：**调用 Actor 中的方法**

> 如果你使用 `actor` 来保护共享状态，调用它的方法必须在异步环境中 —— `Task` 提供入口。

### 示例：

```swift
actor Counter {
    var value = 0
    func increment() {
        value += 1
    }
}

let counter = Counter()

Task {
    await counter.increment()
}
```

---

## ✅ 场景 6：**响应用户操作，快速反馈再异步处理**

> 比如点击按钮后立即变更 UI，再在后台慢慢处理逻辑。

### 示例：

```swift
@IBAction func loginButtonTapped() {
    loginButton.isEnabled = false
    statusLabel.text = "登录中..."

    Task {
        let success = await performLogin()
        await MainActor.run {
            statusLabel.text = success ? "✅ 登录成功" : "❌ 登录失败"
            loginButton.isEnabled = true
        }
    }
}
```

---

## ✅ 场景 7：**调试用 async 函数**

> 比如在调试时手动调用 async 函数测试，不想改项目主结构时。

```swift
Task {
    print(await testAPI())
}
```

---

## ✅ 总结对照表

| 场景                  | 是否适合用 `Task {}` | 说明                  |
| ------------------- | --------------- | ------------------- |
| 非 async 函数中调用 async | ✅               | 唯一方式                |
| UI 层使用异步网络请求        | ✅               | 避免卡顿                |
| 后台执行复杂逻辑            | ✅               | 高性能                 |
| actor 中异步访问         | ✅               | Swift 并发模式          |
| 并发执行多个异步任务          | ✅               | TaskGroup、async let |
| 同步函数中 debug 测试异步    | ✅               | 开发辅助                |

---

想进一步学 `TaskGroup`、`async let`、或 `MainActor`？我可以按模块继续讲。你想深入哪个部分？




***
<br/><br/><br/>
> <h2 id="Swift中的Codable是什么">Swift中的Codable是什么</h2>

两个问题，我来分别解答：

---

## 🧩 1. Swift 中的 `Codable` 是什么？

`Codable` 是 Swift 中用于 **编码（encode）和解码（decode）数据的协议**，常用于：

* JSON 和模型对象之间的转换
* 网络请求响应数据的解析
* 本地保存/读取结构化数据（如 `UserDefaults`, 文件等）

### 🔁 实际用途举例：

```swift
struct User: Codable {
    let id: Int
    let name: String
}

// JSON → Struct
let jsonData = """
{
  "id": 1,
  "name": "Alice"
}
""".data(using: .utf8)!

let user = try! JSONDecoder().decode(User.self, from: jsonData)
print(user.name)  // 输出: Alice

// Struct → JSON
let encodedData = try! JSONEncoder().encode(user)
```

`Codable` 实际上是 `Encodable & Decodable` 的组合协议。

***
<br/><br/><br/>
> <h2 id="关键字转义用法">\`default \`关键字转义用法</h2>
Swift 中的方法命名 `static func \`default\`()\`

这是 Swift 的 **关键字转义用法**。

### ✅ 意思：

`default` 是 Swift 的**关键字**（比如 switch-case 中的 default 分支），**不能直接用作函数名或变量名**。

如果你想使用这些保留字作为方法名、变量名、属性名等，可以用 **反引号（\`）将它括起来**，就可以合法使用了：

```swift
struct Config {
    static func `default`() -> Config {
        return Config()
    }
}
```

### ✅ 调用方式：

```swift
let config = Config.default()
```

### ⚠️ 和普通方法有啥不同？

仅仅是**用了关键字当作名称**而已，其他行为完全一样。反引号是告诉编译器“这是个合法的标识符，不是关键字用法”。

---

是否需要我给一个结合 `Codable` 和 `default()` 方法的完整示例？



***
<br/><br/><br/>
> <h2 id="T?whereT:Decodable">范型中**T? where T: Decodable**</h2>
这是一个使用 **泛型（Generics）+ 协议约束（Protocol Constraints）+ 返回类型** 的 Swift 函数声明。我们来逐块解释：

---

### 👇 函数定义：

```swift
func aa<T>(bb: T) -> T? where T: Decodable { }
```

---

### 🧠 解释每一部分含义：

| 部分                   | 意思说明                              |
| -------------------- | --------------------------------- |
| `func aa<T>`         | 声明了一个泛型函数，`T` 是类型参数               |
| `(bb: T)`            | 该函数接收一个参数 `bb`，类型为 `T`            |
| `-> T?`              | 返回值是 `T?`（即泛型类型的可选值）              |
| `where T: Decodable` | 泛型约束，表示 `T` 类型必须遵守 `Decodable` 协议 |

---

### ✅ 用人话说：

这是一个 **接收任意遵守 `Decodable` 协议的类型 `T` 的值**，并**返回这个类型的可选值**的函数。

---

### ✅ 示例代码：

```swift
func aa<T>(bb: T) -> T? where T: Decodable {
    // 这里你可以做一些处理，比如解析、校验等
    return bb
}

struct User: Decodable {
    let name: String
}

let user = User(name: "Alice")
let result = aa(bb: user)  // result 是 User?
```

---

### ❓为什么用 `where T: Decodable`？

如果你只写成：

```swift
func aa<T>(bb: T) -> T?
```

就表示可以传入**任何类型**。

但你加上：

```swift
where T: Decodable
```

就 **限制 T 必须支持解码（比如可以从 JSON 中解出）**，这通常用于一些 JSON 解析函数中。

---

需要我举一个这个函数被实际使用在 JSON 解码的例子吗？




***
<br/><br/><br/>
> <h2 id="DispatchQueue(label:)队列">DispatchQueue(label:)队列</h2>

`DispatchQueue(label:)` 是 Swift 中用于创建 **GCD（Grand Central Dispatch）队列** 的方法，主要用于**多线程编程**，实现任务的异步或同步执行、并发控制等。

---

## ✅ 一句话解释

`DispatchQueue(label:)` 创建一个 **自定义串行队列**（默认），你可以把任务扔进这个队列，它会**一个一个顺序执行任务**，**避免多线程冲突**。

---

## 📌 基本用法：

```swift
let queue = DispatchQueue(label: "com.example.myQueue")

queue.async {
    print("任务 1")
}

queue.async {
    print("任务 2")
}
```

输出永远是：

```
任务 1
任务 2
```

因为默认是**串行队列**（serial），**任务按顺序执行**。

---

## 🧠 参数说明：

```swift
DispatchQueue(label: String, qos: ..., attributes: ..., autoreleaseFrequency: ..., target: ...)
```

* `label`: 队列的标识符（通常写成反域名形式）
* `attributes`: 指定串行还是并发队列（`[]` = 串行，`.concurrent` = 并发）

---

## 🚀 创建并发队列的写法：

```swift
let concurrentQueue = DispatchQueue(label: "com.example.concurrent", attributes: .concurrent)

concurrentQueue.async {
    print("并发任务 1")
}

concurrentQueue.async {
    print("并发任务 2")
}
```

这时任务可能是**同时执行**的，打印顺序不固定。

---

## 🆚 与同步队列（sync/async）有啥区别？

很多人会混淆 **同步/异步（sync/async）** 和 **串行/并发（serial/concurrent）**，你可以理解如下：

| 概念           | 控制任务如何加入队列            | 控制任务如何执行            |
| ------------ | --------------------- | ------------------- |
| `async`      | 异步加入，不阻塞当前线程          | 任务会在后台异步执行          |
| `sync`       | 同步加入，**阻塞当前线程直到任务完成** | 任务仍按队列性质执行          |
| `serial`     | 队列中任务 **一个接一个执行**     | 可配合 async 或 sync 使用 |
| `concurrent` | 队列中任务 **可以并发执行**      | 可配合 async 或 sync 使用 |

---

### 📌 示例：串行 vs 并发队列

```swift
// 串行队列：顺序执行
let serialQueue = DispatchQueue(label: "com.example.serial")

serialQueue.async {
    print("任务 A")
}
serialQueue.async {
    print("任务 B")
}

// 并发队列：可能并发执行
let concurrentQueue = DispatchQueue(label: "com.example.concurrent", attributes: .concurrent)

concurrentQueue.async {
    print("任务 X")
}
concurrentQueue.async {
    print("任务 Y")
}
```

---

## 🧯 实用场景

* 保证某些资源访问是**线程安全**的（如数据库、文件操作）
* 自定义后台任务队列
* 在 `ViewModel` 或 `网络请求` 中隔离 UI 与逻辑线程

---

是否想了解如何使用队列防止数据竞争或 UI 卡顿？



***
<br/><br/><br/>
> <h2 id="编译器插值#line、#file">编译器插值#line、#file</h2>
这是 Swift 中非常实用的一套 **编译器魔法指令（或称“编译器插值”）**，用于调试、日志记录和错误追踪等功能。

---

## ✅ 一句话解释：

`#file`、`#line` 等是 **Swift 编译器内建的特殊表达式**，会在**编译时**自动替换为当前代码所在的 **文件路径、行号、函数名等信息**。

---

## 📌 常见的几个指令：

| 表达式          | 含义说明                | 示例值                               |
| ------------ | ------------------- | --------------------------------- |
| `#file`      | 当前文件的完整路径字符串        | `"/Users/you/Project/File.swift"` |
| `#fileID`    | 文件的模块内唯一标识（通常是简化路径） | `"MyApp/File.swift"`              |
| `#line`      | 当前行号（Int）           | `42`                              |
| `#column`    | 当前列号（Int）           | `17`                              |
| `#function`  | 当前函数签名              | `"myFunction(param:)"`            |
| `#dsohandle` | 当前动态共享对象的句柄（很少用）    | `UnsafeRawPointer` 类型             |

---

## 📦 实际用途举例：

### 1. **日志系统：记录出错位置**

```swift
func log(_ message: String,
         file: String = #file,
         line: Int = #line,
         function: String = #function) {
    print("[\(file):\(line)] \(function): \(message)")
}

log("网络请求失败")
```

输出示例：

```
[/Users/you/Project/ViewController.swift:56] fetchData(): 网络请求失败
```

你不需要每次都手动传 `file`、`line`，默认参数会自动插入调用位置。

---

### 2. **断言增强（方便定位问题）**

```swift
func myAssert(_ condition: Bool, message: String,
              file: String = #file,
              line: Int = #line) {
    if !condition {
        print("❌ 断言失败: \(message) at \(file):\(line)")
    }
}
```

---

### 3. **崩溃上报或错误追踪工具中使用**

当你想记录在哪个文件哪一行出现问题、方便线上分析时，这些指令非常有用。

---

## 🆕 Swift 5.3+ 简化版本推荐：

如果你不想记录全路径（太长），可以用：

```swift
#fileID   // 更短，例如：MyApp/NetworkManager.swift
```

---

### 🧪 补充知识：

| 用法              | 特点             |
| --------------- | -------------- |
| `#file`         | 全路径，适合调试       |
| `#fileID`       | 相对路径，适合日志输出    |
| 可作为默认参数         | 可自动传入调用点信息     |
| 与自定义日志、错误处理完美结合 | 调试效率高，减少人工定位时间 |

---

是否想我帮你写一个“简易日志系统”封装？可以直接用于 iOS 项目中调试和打印信息。


***
<br/><br/><br/>
> <h2 id="withCheckedContinuation桥接旧的回调式API到async/await模式">withCheckedContinuation桥接旧的回调式API到async/await模式</h2>

`await withCheckedContinuation` 是 Swift 并发中一个非常重要的工具，用来**桥接旧的回调式 API 到 async/await 模式**。

它的作用是：

> ✅ 将基于回调（completion handler）的异步代码 **转换成 async/await 的形式**

---

## 🔍 使用场景

你在工作中经常会遇到这种旧代码：

```swift
func downloadData(from url: URL, completion: @escaping (Data?, Error?) -> Void)
```

但你想要这样写：

```swift
let data = await downloadData(from: url)
```

这就需要用到 `withCheckedContinuation`。

---

## ✅ 基本语法

```swift
func asyncFunction() async -> ResultType {
    await withCheckedContinuation { continuation in
        // 调用回调式 API
        oldAPICall { result in
            continuation.resume(returning: result)
        }
    }
}
```

---

## ✅ 真实案例：用 continuation 包装回调 API

比如我们有一个老的网络请求函数：

```swift
func fetchData(url: URL, completion: @escaping (Data?, Error?) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        completion(data, error)
    }.resume()
}
```

我们现在想把它转换成 async 风格：

---

### 🔧 封装为 async 函数

```swift
func fetchDataAsync(url: URL) async throws -> Data {
    return try await withCheckedThrowingContinuation { continuation in
        fetchData(url: url) { data, error in
            if let data = data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: error ?? URLError(.badServerResponse))
            }
        }
    }
}
```

---

### 🔍 你现在就可以这样用：

```swift
Task {
    do {
        let url = URL(string: "https://httpbin.org/get")!
        let data = try await fetchDataAsync(url: url)
        print("✅ 收到数据：\(data)")
    } catch {
        print("❌ 网络失败：\(error)")
    }
}
```

---

## 🧠 延伸解释

| 名称                                | 用法                           | 说明                   |
| --------------------------------- | ---------------------------- | -------------------- |
| `withCheckedContinuation`         | 不抛错                          | 用于只返回值的 continuation |
| `withCheckedThrowingContinuation` | 可抛错                          | 用于可能会失败的异步任务         |
| `continuation.resume(...)`        | 必须调用一次                       | 恢复 await 调用，否则会卡住    |
| Checked                           | 调试时会检查是否 resume 多次或忘记 resume |                      |

---

## 🛡️ 安全提示

* `withCheckedContinuation` 会在 **Debug 模式下检测：**

  * 是否调用了 `resume()` 多次 ❌
  * 是否根本没调用 `resume()` ❌
* 如果你不确定是否会 resume 多次，可以使用 `resume(with:)` 或加入保护：

```swift
guard !resumed else { return }
resumed = true
```

---

## ✅ 小结口诀：

> **“老回调，要 async？加个 continuation 一键转。”**

---

是否想我再举一个更贴近 UI 的例子，比如 UIImagePickerController 的 async 化封装？这个很实用。


***
<br/><br/><br/>
> <h2 id="拼接公共域名+接口路径">拼接公共域名+接口路径</h2>
这是 Swift 中 `URL` 的一个构造方法：

```swift
public init?(string: String, relativeTo url: URL?)
```

它的作用是：

> ✅ **将相对路径的字符串补全到一个基础 URL 上**，从而构建一个完整的 URL。

---

## 📌 场景用途：**拼接公共域名 + 接口路径**

当你有很多请求共用一个“根地址”（比如 `https://api.example.com`），你就可以用这个方法动态拼接路径，比如：

```swift
let baseURL = URL(string: "https://api.example.com")!
let fullURL = URL(string: "/v1/user/profile", relativeTo: baseURL)!
print(fullURL.absoluteString) // 👉 https://api.example.com/v1/user/profile
```

---

## ✅ 详细例子：构造多个 API 地址

```swift
let baseURL = URL(string: "https://api.example.com")!

let loginURL    = URL(string: "/v1/auth/login", relativeTo: baseURL)!
let userURL     = URL(string: "/v1/user/info",  relativeTo: baseURL)!
let uploadURL   = URL(string: "/v1/file/upload",relativeTo: baseURL)!

print(loginURL.absoluteString)  // https://api.example.com/v1/auth/login
print(userURL.absoluteString)   // https://api.example.com/v1/user/info
print(uploadURL.absoluteString) // https://api.example.com/v1/file/upload
```

这比直接用字符串拼接更**安全、规范、支持路径编码**。

---

## 🧠 `relativeTo:` 的关键点：

| 参数           | 说明                               |
| ------------ | -------------------------------- |
| `string`     | 可以是相对路径（如 `/v1/user`）或完整 URL     |
| `relativeTo` | 一个基础 URL，如 `https://api.xxx.com` |
| 结果           | 会组合两个 URL，生成一个绝对 URL 对象          |

---

## ❌ 注意事项：

* 相对路径**必须以 `/` 开头**，否则它会和 base 的最后一段直接拼接：

```swift
let base = URL(string: "https://api.com/path")!
URL(string: "abc", relativeTo: base)!.absoluteString
// 👉 https://api.com/pathabc ❌（不是 https://api.com/path/abc）
```


***
<br/><br/><br/>
># <h2 id="字符串转枚举">[字符串转枚举](./../Swift/swift基础.md##字符串转枚举)</h2>


***
<br/><br/><br/>
># <h2 id="oc对象转换成结构体的工厂方法">[oc对象转换成结构体的工厂方法](./../Swift/swift基础.md#OC对象转换成结构体)</h2>




***
<br/><br/><br/>
> <h2 id="可用配网过滤">[可用配网过滤](./../Swift/swift基础.md#数组的compactMap和filter使用)</h2>

**需求包含：**

- ✅ `ProvisioningMethod` 枚举（原 LinkWayType）
- ✅ 返回字符串数组
- ✅ `compactMap` 使用你指定的完整写法
- ✅ 是否展示选择页逻辑
- ✅ App版本支持校验
- ✅ 回调结构清晰可直接接入 UI

<br/>

**ProvisioningManager.swift**

```swift
import Foundation

/// ✅ 配网方式（Provisioning Method）
public enum ProvisioningMethod: String {
    case none = "None"
    case ap = "AP"             // AP模式
    case wired = "WN"          // 有线
    case ble = "BLUETOOTH"     // 蓝牙
    case wifi = "WIFI"         // WiFi
    case sn = "SN"             // SN码

    /// ✅ 当前 App 支持的配网方式类型
    static var supportedMethods: [ProvisioningMethod] {
        return [.ap, .wired, .ble]
    }

    /// ✅ 根据 ways 字符串数组过滤出设备支持 & App支持的方式 → 字符串数组返回
    static func filterSupportedMethods(from ways: [String]) -> [String] {
        let methods = ways.compactMap { way -> ProvisioningMethod? in
            return ProvisioningMethod(rawValue: way)
        }
        .filter { supportedMethods.contains($0) }

        return methods.map { $0.rawValue }
    }
}


/// ✅ 配网方式管理器
class ProvisioningManager {

    /// ✅ 示例：App 是否支持某一种配网方式
    private static func isAppSupported(_ method: ProvisioningMethod) -> Bool {
        // 示例：低版本App暂不支持 Wired (可根据业务逻辑改成版本比较)
        if method == .wired {
            return false
        }
        return true
    }

    /// ✅ 传入设备 ways 字符串数组，自动处理配网逻辑
    /// - Parameters:
    ///   - ways: 设备支持的方式（字符串数组）
    ///   - onSingle: 只支持一种方式 —> 直接进入该方式
    ///   - onShowSelection: 多种方式 —> 展示配网方式选择页
    ///   - onShowUpgradeAlert: App不支持设备方式 —> 提示升级
    static func handleProvisioning(
        ways: [String],
        onSingle: @escaping (String) -> Void,
        onShowSelection: @escaping ([String]) -> Void,
        onShowUpgradeAlert: @escaping () -> Void
    ) {

        // ✅ 过滤 App 支持的方式
        let filteredMethods = ProvisioningMethod.filterSupportedMethods(from: ways)

        // ✅ 若过滤后空 → 都不支持 → 弹窗提示
        guard !filteredMethods.isEmpty else {
            onShowUpgradeAlert()
            return
        }

        // ✅ App是否支持方式进一步判断（转换回枚举操作）
        let validMethods = filteredMethods.compactMap { way -> ProvisioningMethod? in
            return ProvisioningMethod(rawValue: way)
        }
        .filter { isAppSupported($0) }
        .map { $0.rawValue }

        if validMethods.isEmpty {
            onShowUpgradeAlert()
            return
        }

        switch validMethods.count {
        case 1:
            // ✅ 一种方式 → 直接进入配网流程
            onSingle(validMethods.first!)

        default:
            // ✅ 多种方式 → 展示选择页
            onShowSelection(validMethods)
        }
    }
}
```

<br/>

**使用：**

```swift
let deviceSupport = ["BLUETOOTH", "WN"]

ProvisioningManager.handleProvisioning(
    ways: deviceSupport
) { way in
    print("✅ 直接进入：\(way)")
} onShowSelection: { methods in
    print("✅ 展示选择页：\(methods)")
} onShowUpgradeAlert: {
    print("⚠️ 暂不支持，请升级App")
}
```

输出示例：

```
✅ 展示选择页：["BLUETOOTH"]
```

（因为 Wired 当前设定为不支持 → 过滤掉）


这个知识需要看：[数组的compactMap和filter使用](./../Swift/swift基础.md#数组的compactMap和filter使用)

***
<br/><br/><br/>
> <h2 id="更新/替换list中相同设备的数据-map、first方法">更新/替换list中相同设备的数据-map、first方法</h2>

```swift
return list.map { element in
    if let match = data.first(where: { $0.iotId == element.iotId }) {
        return match
    }
    return element
}
```
<br/>

**这段代码做的事情是：**

> 遍历 `list` 中的每一个元素
> 对每个元素，到 `data` 里找有没有 **同样 iotId** 的对象
> 找到 ➜ 用 `data` 的对象替换
> 找不到 ➜ 用原来的对象


> **用 data 里面的一部分数据，更新/替换 list 中相同设备的数据**

<br/>

**举个具体例子**

现有 A 表（设备列表 — list）

| iotId | 名字    |
| ----- | ----- |
| 1     | 老数据_A |
| 2     | 老数据_B |
| 3     | 老数据_C |

<br/>

另有 B 表（扫描回来最新设备信息 — data）

| iotId | 名字    |
| ----- | ----- |
| 2     | 新数据_B |
| 3     | 新数据_C |

<br/>

执行后结果👇

| iotId | 名字    | 来源      |
| ----- | ----- | ------- |
| 1     | 老数据_A | list 保留 |
| 2     | 新数据_B | data 替换 |
| 3     | 新数据_C | data 替换 |

也就是：

* 发现相同编号，就用 **最新的数据覆盖旧数据**
* 没找到，就保持不变

[数组map、first详细用法请看这里](./../Swift/swift基础.md#数组map、first方法详解)


***
<br/><br/><br/>
> <h2 id="数组集合使用协议hash比较">数组集合使用协议hash比较</h2>
# `contains(where:)` 与 `==` 的调用逻辑说明

## 你的核心疑问

在 `AKThingLANScanner.swift` 中用了：

```swift
guard scannedDevices.contains(where: { $0.dedupIdentity == device.dedupIdentity }) == false else {
    return
}
```

你想知道：它是否会调用 `AKThingLANScannedDeviceModel` 里的：

```swift
public static func == (
    lhs: AKThingLANScannedDeviceModel,
    rhs: AKThingLANScannedDeviceModel
) -> Bool
```

## 结论先说

- **不会**调用你贴的 `AKThingLANScannedDeviceModel.==`。
- 这行代码比较的是 **`String`**（`dedupIdentity`）而不是模型对象本身。
- 实际调用的是 **`String` 的 `==`**。

## 为什么不会调用模型的 `==`

### 1) `contains(where:)` 的语义

`contains(where:)` 接收一个闭包：

```swift
(Element) -> Bool
```

数组里每个元素都会传入这个闭包，闭包返回 `true` 就停止并返回 `true`。

### 2) 你当前闭包比较的是 `dedupIdentity`

闭包内容是：

```swift
$0.dedupIdentity == device.dedupIdentity
```

`dedupIdentity` 类型是 `String`，因此这里是：

```swift
String == String
```

不是：

```swift
AKThingLANScannedDeviceModel == AKThingLANScannedDeviceModel
```

所以不会走模型里自定义的 `static func ==`。

## 什么时候才会调用模型的 `==`

只有直接比较模型对象时才会触发，比如：

```swift
lhs == rhs
```

或者：

```swift
scannedDevices.contains(device)
```

这一种 `contains(_:)` 需要 `Element: Equatable`，它内部会用元素的 `==`，所以会调用模型的 `==`。

---

## 对比示例（最直观）

### 示例 A：会调用模型 `==`

```swift
let a = AKThingLANScannedDeviceModel(productId: "p1", mac: "", ip: "192.168.1.10")
let b = AKThingLANScannedDeviceModel(productId: "p2", mac: "", ip: "192.168.1.10")

// 调用 AKThingLANScannedDeviceModel.static func ==
let same = (a == b)
```

### 示例 B：不会调用模型 `==`

```swift
let hasSame = scannedDevices.contains { item in
    // 这里只比较 String，调用 String.==
    item.dedupIdentity == device.dedupIdentity
}
```

---

## 你这段 guard 的执行过程（逐步）

```swift
guard scannedDevices.contains(where: { $0.dedupIdentity == device.dedupIdentity }) == false else {
    return
}
```

按步骤是：

1. 遍历 `scannedDevices`。
2. 对每个元素计算 `$0.dedupIdentity`（String）。
3. 用 `String == String` 与 `device.dedupIdentity` 比较。
4. 只要某次为 `true`，`contains(where:)` 立即返回 `true`。
5. `guard true == false` 不成立，进入 `else { return }`，当前设备不再追加。
6. 如果全都不相等，`contains(where:)` 返回 `false`，`guard` 通过，继续 append。

---

## 举一个完整去重例子

假设已有数组里有：

```text
productId = "p1", mac = "AA-BB-CC-DD-EE-FF", ip = "192.168.1.10"
```

它的 `dedupIdentity` 会是：

```text
mac:aabbccddeeff
```

新扫到设备：

```text
productId = "p2", mac = "aa:bb:cc:dd:ee:ff", ip = "192.168.1.99"
```

其 `dedupIdentity` 也是：

```text
mac:aabbccddeeff
```

因此闭包比较结果为 `true`，`contains(where:)` 返回 `true`，`guard` 触发 `return`，新设备被判定为重复。

这就是“**用 dedupIdentity 字符串做去重**”的完整调用链。


<br/><br/><br/>

***
<br/>

> <h1 id="线程安全">线程安全</h1>


***
<br/><br/><br/>
> <h2 id="barrier实现线程安全">barrier实现线程安全</h2>

[**具体详情移步这里**](./../Swift/swift多线程.md#)





<br/><br/><br/>

***
<br/>

> <h1 id="UI组件">UI组件</h1>


***
<br/><br/><br/>
> <h2 id="输入框字数限制">[输入框字数限制](../Objective-C/UITextField.md#输入框字数限制)</h2>



***
<br/>
># <h2 id="label宽度自适应">[label宽度自适应](./../Swift/UI组件.md#宽度自适应)</h2>


***
<br/><br/><br/>
># <h2 id="弹窗播放视频">[弹窗播放视频](./../Swift/UI组件.md#弹窗播放视频)</h2>


***
<br/><br/><br/>
> <h2 id="自定义简易观察者">自定义简易观察者</h2>

这是一个自定义的“观察者”模式类 `Observable<T>`，它的作用是：当内部的 `value` 发生变化时，通知外部观察者。这个模式在 MVVM 中非常常见，用于数据绑定，类似 SwiftUI 的 `@Published` 或 RxSwift 的 `Observable`。

<br/>

**✅ 先扩展你给的类为一个完整可用的版本：**

```swift
public typealias Observer = (T) -> Void

public class Observable<T> {
    private var observer: Observer?

    public var value: T {
        didSet {
	        DispatchQueue.main.async {
		        observer?(value)
	        }
        }
    }

    public init(_ value: T) {
        self.value = value
    }

    public func bind(observer: @escaping Observer) {
        self.observer = observer
        observer(value) // 初始也触发一次
    }
}
```

<br/>

**🧪 示例：使用这个 `Observable` 做数据绑定**

**✅ 场景：监听用户名变化并更新 UI（控制台模拟）**

```swift
class UserViewModel {
    var username: Observable<String> = Observable("初始用户")
    
    func changeUsername(to newName: String) {
        username.value = newName
    }
}

// 控制器或视图层
let viewModel = UserViewModel()

// 绑定观察者
viewModel.username.bind { newName in
    print("👤 用户名变化：\(newName)")
}

// 改变值，触发观察
viewModel.changeUsername(to: "ChatGPT")
viewModel.changeUsername(to: "OpenAI")
```

<br/>

**🖨 输出：**

```sh
👤 用户名变化：初始用户
👤 用户名变化：ChatGPT
👤 用户名变化：OpenAI
```

---

**🧩 用途总结：**

| 功能    | 说明                            |
| ----- | ----------------------------- |
| 数据绑定  | 视图层自动响应 ViewModel 的变化         |
| 实时监听  | 改变 `value` 时自动通知观察者           |
| 替代复杂库 | 简化版的响应式编程（类似 RxSwift、Combine） |


***
<br/><br/><br/>


**🚀 想进一步拓展？**

你可以：

* 增加多个观察者支持
* 添加 `map`、`filter` 等响应式操作
* 做成双向绑定（双向观察）

---

写一个适配 UIKit 控件（如 `UITextField` + Observable）的实际绑定例子，这在 MVVM 架构中非常常用。


好的，我们来 **拓展 `Observable<T>` 支持多个观察者**，并写一个实际的 UIKit 绑定例子，让你看到它如何在 MVVM 中应用。

---

**✅ 一、增强版 `Observable<T>` —— 支持多个观察者、自动移除**

```swift
public class Observable<T> {
    public typealias Observer = (T) -> Void
    
    private var observers: [UUID: Observer] = [:]

    public var value: T {
        didSet {
            notifyObservers()
        }
    }

    public init(_ value: T) {
        self.value = value
    }

    // 添加观察者，返回一个取消的 token
    @discardableResult
    public func bind(_ observer: @escaping Observer) -> UUID {
        let id = UUID()
        observers[id] = observer
        observer(value) // 初始触发一次
        return id
    }

    public func unbind(_ id: UUID) {
        observers.removeValue(forKey: id)
    }

    private func notifyObservers() {
        for observer in observers.values {
            observer(value)
        }
    }
}
```

---

**✅ 二、实际 UIKit 例子：将 ViewModel 中的值绑定到 UILabel 和 UITextField**

**🧩 ViewModel（带 Observable）**

```swift
class LoginViewModel {
    var username = Observable<String>("")
    
    func updateUsername(_ newValue: String) {
        username.value = newValue
    }
}
```

<br/>

**🧩 ViewController 示例（UIKit）**

```swift
class LoginViewController: UIViewController {
    let viewModel = LoginViewModel()

    private let usernameLabel = UILabel()
    private let usernameTextField = UITextField()

    private var usernameObserverID: UUID?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white

        // 设置 UI
        usernameLabel.frame = CGRect(x: 40, y: 100, width: 300, height: 30)
        usernameTextField.frame = CGRect(x: 40, y: 150, width: 300, height: 30)
        usernameTextField.borderStyle = .roundedRect

        view.addSubview(usernameLabel)
        view.addSubview(usernameTextField)

        // 绑定 ViewModel -> Label
        usernameObserverID = viewModel.username.bind { [weak self] newUsername in
            self?.usernameLabel.text = "👤 当前用户名：\(newUsername)"
        }

        // 监听 TextField -> ViewModel
        usernameTextField.addTarget(self, action: #selector(textFieldChanged(_:)), for: .editingChanged)
    }

    @objc func textFieldChanged(_ sender: UITextField) {
        viewModel.updateUsername(sender.text ?? "")
    }

    deinit {
        if let id = usernameObserverID {
            viewModel.username.unbind(id)
        }
    }
}
```

---

- **🧪 运行效果**
	* 你输入内容到 `UITextField`
	* ViewModel 的 `username.value` 被更新
	* 所有绑定了 `username` 的观察者被通知（例如更新了 `UILabel`）

---

**📌 小结**

| 特性        | 说明                              |
| --------- | ------------------------------- |
| ✅ 多观察者    | 可以绑定多个 UI 或逻辑响应                 |
| 🔄 双向绑定   | TextField 改变 → VM 更新 → Label 更新 |
| 🧼 清理机制   | `UUID` 控制解绑，防止内存泄漏              |
| ✅ 适合 MVVM | 轻量替代 RxSwift、Combine 的简单项目      |


**注意:** 不建议上述这么做,因为当业务复杂的时候将观察者放入数组中后,后面排查问题很难排查了.因为你不知道哪些注册观察者,哪些没有注册观察者.


***
<br/><br/><br/>
> <h2 id="一段代码搞定播放音频，优化内存释放">一段代码搞定播放音频，优化内存释放</h2>

```swift

import Foundation
import AVFoundation
private let TAG = "VoicePlayer"

public class VoicePlayer: NSObject {
    
    private static let shared = VoicePlayer()
    private(set) static var player: AVAudioPlayer?
        
    public static func playScanVoice() {
        
        guard let url = ResourceBundles.scanVoiceFilePath() else {
            return
        }
        
        self.preparePlayVoice(url: url)
    }
    
    public static func playSuggestHandVoice() {
    
        guard let url = ResourceBundles.suggestHandVoiceFilePath() else {
            return
        }
        
        self.preparePlayVoice(url: url)
    }
}

extension BleVoicePlayer: AVAudioPlayerDelegate {
    
    private static func preparePlayVoice(url: URL) {
        
        do {
            self.player = try AVAudioPlayer(contentsOf: url)
            self.player?.delegate = self.shared
            self.player?.prepareToPlay()
            self.player?.play()
        } catch {
            AKLog.e(module: TAG, message: "playScanVoice error: \(error)")
        }
    }
    
    // 播放完成释放
    public func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {// 优化的地方
        BleVoicePlayer.player = nil
    }
}
```

<br/>

**调用**

```swift

VoicePlayer.playScanVoice()
            
DispatchQueue.main.asyncAfter(deadline: .now() + 20) {
    VoicePlayer.playSuggestHandVoice()
}
```


<br/><br/><br/>

***
<br/>

> <h1 id="线程安全">线程安全</h1>


***
<br/><br/><br/>
> <h2 id="指定队列判断">指定队列判断</h2>

```swift
private let workQueueKey = DispatchSpecificKey<Void>()
```

这行代码声明了一个 Key，类型是 DispatchSpecificKey<Void>，意思是：

“我要定义一个 key，用来和某个队列建立映射关系，值类型是 Void。”

它本身不会做任何事情，必须搭配 setSpecific / getSpecific 使用。

[若是不太理解，看这里吧](./../Swift/swift多线程.md#判断当前线程在哪条队列执行)




<br/><br/><br/>

***
<br/>

># <h1 id="系统蓝牙库CoreBluetooth">[系统蓝牙库CoreBluetooth](./../Swift/蓝牙库CoreBluetooth.md)</h1>

***
<br/><br/><br/>
># <h2 id="精确获取当前蓝牙开启状态">[精确获取当前蓝牙开启状态](./../蓝牙库CoreBluetooth.md#轮询获取蓝牙开启状态)</h2>


***
<br/><br/><br/>
> <h2 id="轮询连接蓝牙设备">[轮询连接蓝牙设备](./../蓝牙库CoreBluetooth.md#优化轮询连接蓝牙外设)</h2>


***
<br/><br/><br/>
># <h2 id="BLE扫描SessionID">[./../蓝牙库CoreBluetooth.md# BLE扫描SessionID](#BLE扫描SessionID)</h2>



***
<br/><br/><br/>
> <h2 id="一段代码检测蓝牙状态，优化内存泄露">一段代码检测蓝牙状态，优化内存泄露</h2>


```swift
import AppUIKit
import AAUtil
import ActivatorKit

public class PopupViewUntil {
    
    private static var bleActivator: BleActivator?
    
    deinit{ }
    
    public static func showOpenBlePopupView(onOffBlock:@escaping (_ isOn: Bool)->Void) {
        
        self.checkBleSwitchStatus(completeBlock: { isOnStatus in
            
            if isOnStatus {
                onOffBlock(isOnStatus)
                return
            }
            
            let pop = BottomTipPopView(title:"提示", contentTip: "请先打开手机蓝牙，以便查找附近蓝牙设备",
                                            cancelText:"取消", okText:"前往开启", btnLayout: .horizontal)
            pop.isDismissBack = false
            pop.okClickBlock = { () in
                pop.hidden()
                
                self.openBleSettingsSmartly()
            }
            pop.show()
        })
    }
}

extension PopupViewUntil {
    
    private static func checkBleSwitchStatus(completeBlock:@escaping (_ isOn: Bool) -> Void) {
        
        self.bleActivator = BleActivator(processStatus:.none)
        Permission.bluetooth.request {
            let authorized = Permission.bluetooth.authorized
            if !authorized {
                self.openSystemSettingSmartly()
                completeBlock(false)
                return
            }
                    
            //self.bleActivator = BleActivator(processStatus:.none)
            self.bleActivator?.checkBluetoothSwitchStatus(completionBlock:{ isOn in
                completeBlock(isOn)
                if isOn {
                    // 释放 activator（避免内存泄漏）
                    self.bleActivator?.bleActivatorFree()
                    self.bleActivator = nil
                    return
                }
            }) 
        }
    }
    
    public static func openBleSettingsSmartly() {
        
        if #available(iOS 11.0, *) {
            self.openSystemSettingSmartly()
        } else {
            if let bluetoothURL = URL(string: "App-Prefs:root=Bluetooth"),
               UIApplication.shared.canOpenURL(bluetoothURL) {
                UIApplication.shared.open(bluetoothURL)
            }
        }
    }
    
    private static func openSystemSettingSmartly() {
        if let url = URL(string: UIApplication.openSettingsURLString),
           UIApplication.shared.canOpenURL(url) {
            UIApplication.shared.open(url)
        }
    }
}
```

<br/>

**调用：**

```swift
PopupViewUntil.showOpenBlePopupView { [weak self] isAuth in
    if isAuth {
        self?.scanAction()
    }
}

```



***
<br/><br/><br/>
># <h2 id="蓝牙入口">[蓝牙入口](../Swift/蓝牙库CoreBluetooth.md#蓝牙入口)</h2>


<br/><br/>
># <h3 id="蓝牙权限状态获取">[蓝牙权限状态获取](../Swift/蓝牙库CoreBluetooth.md#蓝牙权限状态获取)</h3>




<br/><br/><br/>

***
<br/>

> <h1 id="Wi-Fi连接">Wi-Fi连接</h1>

***
<br/><br/><br/>
> <h2 id="App内添加Wi-Fi配置">App内添加Wi-Fi配置</h2>

```swift
NEHotspotConfiguration 

NEHotspotConfigurationManager.shared.apply(config) { error in}

NEHotspotNetwork.fetchCurrent { network in}

CNCopySupportedInterfaces()
```

[详情请看这里](./蓝牙库CoreBluetooth.md#App加入设备Wi-Fi网络)




<br/><br/><br/>

***
<br/>

<h1 id= "依赖注入库Swinject">依赖注入库Swinject</h1>

Swinject 是 Swift 中最流行的 **依赖注入（Dependency Injection, DI）框架**之一。它的主要作用是**解耦组件之间的依赖关系**，使代码更模块化、易测试、可维护。

---

### 📌 Swinject 能干什么？

* 管理依赖关系（对象如何彼此依赖）
* 简化构造复杂对象的过程
* 方便单元测试（可替换依赖）
* 避免在代码中硬编码依赖

---

### 🧩 简单例子：使用 Swinject 构建依赖注入

#### 1. 安装 Swinject（用 Swift Package Manager）

在 `Package.swift` 或 Xcode 的 SPM 配置中添加：

```swift
.package(url: "https://github.com/Swinject/Swinject.git", from: "2.8.0")
```

或者

```podfile
target 'YourAppTarget' do
  use_frameworks!

  pod 'Swinject', '~> 2.8'
end
```

---

#### 2. 定义协议和实现

```swift
protocol Animal {
    func sound() -> String
}

class Dog: Animal {
    func sound() -> String {
        return "Woof!"
    }
}
```

---

#### 3. 注册依赖并解析

```swift
import Swinject

// 创建一个容器
let container = Container()

// 注册 Animal 协议的实现
container.register(Animal.self) { _ in Dog() }

// 从容器中解析（获取）实例
let animal = container.resolve(Animal.self)!
print(animal.sound())  // 输出: "Woof!"
```

---

### 🔁 比较一下：不使用 Swinject 时

```swift
let animal = Dog()  // 直接创建，耦合在一起
```

这样就无法轻松替换实现，也不利于测试。而使用 Swinject，你可以很容易替换 `Dog` 为 `Cat`、`MockAnimal` 等。

---

需要一个更贴近实际项目的示例（比如注入到 ViewModel 或 Controller）吗？


<br/><br/><br/>

***
<br/>

<h1 id= "实时音视频通信WebRTC">实时音视频通信WebRTC</h1>

`WebRTC`（全称 Web Real-Time Communication）是一个开源项目，用于实现**实时音视频通信**（包括点对点语音、视频、数据传输）。而在 iOS 中，`WebRTC` 通常是通过 Google 提供的 `WebRTC.framework` 或第三方封装（如 `webrtcn`）来使用的。

---
<br/>

**✅ 在 iOS 中 `WebRTC` / `webrtcn` 的作用**

它主要用于：

1. **视频通话**
2. **语音通话**
3. **实时屏幕共享**
4. **点对点数据通道（data channel）**
5. **无服务器中转的低延迟通信**

---
<br/>

**🧰 webrtcn 是什么？**

> `webrtcn` 是对原生 `WebRTC.framework` 的封装库（比如来自社区或某个开发者发布到 CocoaPods 的），它简化了 WebRTC 在 iOS 上的集成。

它的目标通常是：

* 简化复杂的 PeerConnection 设置
* 更方便地在 iOS 项目中使用 WebRTC（原生 API 较为底层）
* 可能内置一些媒体管理、信令等工具

---

**📦例：iOS 使用 WebRTC（或 webrtcn）可实现的功能**

假设你开发一个“视频聊天 App”，你可以使用 `webrtcn` 来：

* 初始化音视频引擎
* 获取摄像头视频
* 建立 `RTCPeerConnection`（与对端连接）
* 通过信令服务器交换 SDP/ICE 信息
* 实现点对点音视频传输

---
<br/>

**简化理解**

| 功能         | WebRTC/webrtcn 能做吗？ | 用途举例             |
| ---------- | ------------------- | ---------------- |
| 视频通话       | ✅                   | 类似 FaceTime、Zoom |
| 音频通话       | ✅                   | 如语音聊天、对讲功能       |
| 实时数据传输     | ✅                   | 传输游戏状态、聊天消息等     |
| 服务端中转/录制功能 | ❌（需自己实现）            | 通常需要额外服务器实现      |


***
<br/><br/><br/>
> <h2 id="ICE服务器了解">ICE服务器了解>ICE>

在实时音视频与点对点通信领域中，**ICE 服务器**里的 **ICE** 并不是“移民执法机构”，而是一个技术缩写，含义是：

> **ICE（Interactive Connectivity Establishment，交互式连接建立）**

它是 **WebRTC / P2P 实时通信** 中用于**解决 NAT、防火墙穿透问题**的一整套机制与流程，而不是单一的一台服务器。

---
<br/>

**ICE 服务器到底是“什么”？**

严格来说：

> **ICE 不是一种服务器，而是一套连接建立机制**
> 在实现上，通常会配合 **STUN 服务器** 和 **TURN 服务器** 使用
> 业界习惯把 *STUN / TURN* 统称为“ICE 服务器”

<br/>

**为什么需要 ICE？**

现实网络环境中，大多数设备都处在：

* NAT（路由器后面）
* 防火墙
* 内网 / 私网

这会导致：

* 设备 **无法直接互相访问**
* 即使知道对方 IP，也连不上

ICE 的目标只有一个：

> **让两个终端在复杂网络环境下，尽可能建立“直连”，直连失败再走中继**

<br/>

**ICE 的三大核心角色**

**1️⃣ STUN 服务器（“探路者”）**

- **作用：**
	* 帮客户端“探测”自己在公网看到的 IP 和端口
	* 判断 NAT 类型


- **特点：**
	* 不转发媒体数据
	* 流量小、成本低
	* 成功则实现 **P2P 直连**

<br/>

**大白话理解：**

> “我是谁？我在公网长什么样？”

<br/>

**2️⃣ TURN 服务器（“中继站”）**

- **作用：**
	* 当直连失败时，**作为数据中转服务器**
	* 所有音视频数据都会经过 TURN

- **特点：**
	* 成本高（带宽消耗大）
	* 稳定性最好
	* 是 **兜底方案**

**大白话理解：**

> “你俩连不上？那我来帮你们传。”

<br/>

**3️⃣ ICE 本身（“调度者”）**

**作用：**

* 收集各种候选连接方式（Candidate）
* 逐一测试
* 选择**最优可用路径**

候选类型包括：

| 类型               | 说明              |
| ---------------- | --------------- |
| Host             | 本地 IP（同一局域网）    |
| Server Reflexive | 通过 STUN 获得的公网地址 |
| Relay            | 通过 TURN 的中继地址   |

---
<br/>

**ICE 工作流程（简化版）**

```
A 设备                ICE/STUN/TURN                B 设备
  |  获取候选地址  |                          |
  |-------------->|                          |
  |  交换候选信息（信令服务器）              |
  |<---------------------------------------->|
  |  尝试直连（Host / STUN）                |
  |---------------------------------------->|
  |  直连失败 → 使用 TURN 中继               |
  |============ TURN 转发媒体 ===============|
```

> ⚠️ **ICE 本身不负责信令**
> WebRTC 中的 SDP / Candidate 交换，需要你自己的 **信令服务器**

---
<br/>

**五、ICE 在哪些场景中必不可少？**

* WebRTC（浏览器音视频）
* 视频会议（Zoom / Meet / Teams）
* 实时语音 / 视频通话
* P2P 文件传输
* 监控设备远程预览
* 智能硬件点对点通信

---
<br/>

**为什么配置里常看到这样？**

```json
iceServers: [
  { "urls": "stun:stun.l.google.com:19302" },
  {
    "urls": "turn:turn.example.com:3478",
    "username": "user",
    "credential": "pass"
  }
]
```

含义是：

* **优先用 STUN 直连**
* **失败再走 TURN**
* ICE 自动完成策略选择

---
<br/>

**常见误解澄清**

❌ ICE = 一台服务器
✅ ICE = 一套机制 + STUN/TURN 配合

❌ 只要 STUN 就够了
✅ 企业级 / 复杂网络 **必须有 TURN 兜底**

❌ ICE 能替代信令
✅ ICE **不负责信令**

---
<br/>

**总结：**

> **ICE 是 WebRTC 等实时通信中，用来“想尽一切办法让两端连上”的机制**
> **STUN 负责探路，TURN 负责兜底，ICE 负责决策**



<br/><br/><br/>

***
<br/>

> <h1 id= "日志库SwiftyBeaver">日志库SwiftyBeaver</h1>


`SwiftyBeaver` 是 Swift 中一个非常流行的 **日志记录框架（logging framework）**，它的作用是帮助你更好地**打印调试信息、跟踪错误、输出日志到控制台、文件或远程服务器**，比系统的 `print()` 更强大、可控、可扩展。

---

## 🧩 一、SwiftyBeaver 能做什么？

| 功能                | 描述                                                      |
| ----------------- | ------------------------------------------------------- |
| 多日志目标支持           | 控制台（Console）、文件、远程（平台上传）                                |
| 日志级别支持（5 个等级）     | `.verbose` / `.debug` / `.info` / `.warning` / `.error` |
| 日志格式化、美观、带颜色输出    | 支持颜色高亮、显示函数、行号等                                         |
| 自动记录调用点（文件、函数、行号） | 利用了 `#file`、`#function`、`#line`                         |
| 日志过滤、按级别输出控制      | 支持在不同环境下输出不同级别日志                                        |

---

## 🧩 二、语法解释：`private let log = SwiftyBeaver.self`

这行代码：

```swift
private let log = SwiftyBeaver.self
```

的意思是：将 `SwiftyBeaver` 类的**类型本身**赋给 `log`，用于调用类方法。

你可以这样调用：

```swift
log.warning("Something might go wrong")
```

等价于：

```swift
SwiftyBeaver.warning("Something might go wrong")
```

这种写法只是简化后续使用、让代码更整洁。

---

## 🧩 三、方法详解：`warning(...)`

```swift
open class func warning(
    _ message: @autoclosure () -> Any,
    file: String = #file,
    function: String = #function,
    line: Int = #line,
    context: Any? = nil
)
```

### ✅ 参数说明：

| 参数名        | 类型                       | 说明               |
| ---------- | ------------------------ | ---------------- |
| `message`  | `@autoclosure () -> Any` | 日志内容（延迟求值）       |
| `file`     | `String`                 | 自动记录调用日志的文件路径    |
| `function` | `String`                 | 自动记录调用日志的函数名     |
| `line`     | `Int`                    | 自动记录调用日志的行号      |
| `context`  | `Any?`                   | 自定义上下文，便于过滤或远程识别 |

### ✅ 什么是 `@autoclosure`？

它表示 `message` 参数是一个**自动闭包**，意思是你调用时直接写：

```swift
log.warning("Disk is almost full")
```

而不是：

```swift
log.warning({ return "Disk is almost full" })
```

自动闭包的优点是：**只有在日志级别满足条件时才执行 message 内容的计算**，节省性能。

---

## ✅ 举个实际用法：

```swift
log.warning("用户登录失败，token: \(userToken)")
```

SwiftyBeaver 会输出格式类似于：

```
⚠️ 2025-05-07 10:21:01 [ViewController.swift:42 login()] 用户登录失败，token: abc123
```

---

## ✅ 日志等级：

```swift
log.verbose("开发调试信息")
log.debug("调试细节")
log.info("一般运行信息")
log.warning("可能的问题")
log.error("严重错误")
```

你可以按需配置只在 Debug 环境输出 `.debug`，而 Release 环境只记录 `.warning` 以上。

---

## 🛠 配置 SwiftyBeaver（初次使用）：

```swift
import SwiftyBeaver

let console = ConsoleDestination()
SwiftyBeaver.addDestination(console)
```

也可添加文件或云端目标。

---

需要我写一个完整的封装类（比如 `Logger.swift`）让你在项目中统一使用吗？



***
<br/><br/><br/>
> <h2 id="完整的日志封装类">完整的日志封装类</h2>

好的！以下是一个简单实用的 `Logger.swift` 文件封装，用于在 Swift 项目中统一使用 `SwiftyBeaver` 打印日志。支持控制台输出、按日志等级分类，并且代码整洁易维护。

---

## ✅ 第一步：安装 SwiftyBeaver（如使用 CocoaPods）

```ruby
pod 'SwiftyBeaver'
```

安装后 `import SwiftyBeaver` 即可使用。

---

## ✅ 第二步：创建 Logger.swift 封装类

```swift
// Logger.swift
import SwiftyBeaver

let log = Logger.shared

final class Logger {

    static let shared = Logger()
    private let logger = SwiftyBeaver.self

    private init() {
        // 添加 Console 输出目标
        let console = ConsoleDestination()
        console.minLevel = .verbose  // 可修改最低日志等级
        logger.addDestination(console)

        // 可选：添加文件输出（保存到本地）
        /*
        let file = FileDestination()
        file.logFileURL = URL(fileURLWithPath: "/tmp/swifty.log")
        logger.addDestination(file)
        */

        // 可扩展：添加云日志目标（SwiftyBeaver Platform）
    }

    // MARK: - 日志方法封装

    func verbose(_ message: @autoclosure () -> Any,
                 file: String = #file, function: String = #function, line: Int = #line) {
        logger.verbose(message(), file, function, line: line)
    }

    func debug(_ message: @autoclosure () -> Any,
               file: String = #file, function: String = #function, line: Int = #line) {
        logger.debug(message(), file, function, line: line)
    }

    func info(_ message: @autoclosure () -> Any,
              file: String = #file, function: String = #function, line: Int = line) {
        logger.info(message(), file, function, line: line)
    }

    func warning(_ message: @autoclosure () -> Any,
                 file: String = #file, function: String = #function, line: Int = line) {
        logger.warning(message(), file, function, line: line)
    }

    func error(_ message: @autoclosure () -> Any,
               file: String = #file, function: String = #function, line: Int = line) {
        logger.error(message(), file, function, line: line)
    }
}
```

---

## ✅ 使用示例：

```swift
log.debug("用户点击了登录按钮")
log.error("登录失败，错误码：\(errorCode)")
```

输出内容会自动带上调用位置（文件、函数、行号），调试追踪非常方便。

---

是否需要我加上日志等级过滤、颜色输出或写入文件等高级功能？



<br/><br/><br/>

***
<br/>

> <h1 id= "alamofire库">alamofire库</h1>

***
<br/><br/><br/>
> <h2 id="拦截器">拦截器</h2>

`Alamofire` 的 **拦截器（Interceptor）** 主要用于在 **请求发出之前** 和 **响应返回之后** 对网络请求进行**修改、处理**，比如：

* 添加认证 token
* 统一处理请求头
* 日志打印
* 错误重试机制
* 缓存处理

拦截器为我们提供了非常灵活和统一的方式来管理这些操作，而不需要在每个请求中单独配置。

---

## ✅ 拦截器的工作原理：

拦截器通过 **`RequestInterceptor`** 协议来定义。你可以实现这个协议，来控制请求的行为，比如添加请求头、修改请求参数等。

### **核心概念**

* **`RequestInterceptor`**：通过它我们可以在请求发出之前修改请求内容（例如，添加 `Authorization` 头部）。
* **`ResponseInterceptor`**：我们可以在接收到响应之后对响应进行处理（例如，捕获 401 错误并刷新 token）。
* **`Retry`**：提供请求重试的功能。

---

## ✅ 创建自定义拦截器

假设我们需要实现一个拦截器来自动添加 `Authorization` 头，并在响应中捕获 401 错误进行 token 刷新。

### 步骤 1：创建自定义拦截器类

```swift
import Alamofire

class AuthInterceptor: RequestInterceptor {
    private let token: String
    
    // 初始化时接收 token
    init(token: String) {
        self.token = token
    }
    
    // 请求拦截：每次请求都会执行
    func adapt(_ urlRequest: URLRequest, for session: Session) throws -> URLRequest {
        var request = urlRequest
        // 在请求头中添加 Authorization 信息
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return request
    }
    
    // 响应拦截：对响应结果进行处理
    func retry(_ request: Request, for session: Session, dueTo error: Error) -> RetryResult {
        // 如果遇到 401 错误，表示 token 过期，需要重试
        if let afError = error as? AFError, afError.responseCode == 401 {
            print("Token 过期，进行重试处理")
            // 在这里你可以刷新 token，然后重试请求
            // 比如：
            // fetchNewToken { newToken in
            //     // 更新 token 并重试
            // }
            return .retry
        }
        
        // 默认不重试
        return .doNotRetry
    }
}
```

### 步骤 2：设置 Alamofire 会话使用这个拦截器

```swift
import Alamofire

// 创建拦截器
let interceptor = AuthInterceptor(token: "your-token-here")

// 配置 Alamofire 会话
let session = Session(interceptor: interceptor)

// 进行网络请求
session.request("https://api.example.com/data").responseJSON { response in
    switch response.result {
    case .success(let data):
        print("数据：\(data)")
    case .failure(let error):
        print("请求失败：\(error)")
    }
}
```

---

## ✅ 解释每一部分

### `AuthInterceptor` 类：

1. **适配请求（`adapt(_:for:)`）**：

   * 每次请求发出时，`adapt` 会被调用。
   * 在这里，我们添加了 `Authorization` 头部，这个头部包含了认证用的 `token`。

2. **重试机制（`retry(_:for:dueTo:)`）**：

   * 如果响应代码是 401，表示 token 可能已经过期，我们就可以触发 token 刷新并重新发起请求。
   * 可以在这里加上自动刷新 token 的逻辑（比如调用一个接口获取新 token），然后重新尝试请求。

---

## ✅ 使用 Alamofire 会话时的设置：

```swift
let session = Session(interceptor: interceptor)
```

这里我们将自定义的 `AuthInterceptor` 传递给了 `Session` 对象。之后，所有通过这个 `session` 发出的请求都会使用我们的拦截器。

---

## ✅ 完整示例：带有重试逻辑的 API 请求

```swift
import Alamofire

// 创建自定义的拦截器
class AuthInterceptor: RequestInterceptor {
    private var token: String

    init(token: String) {
        self.token = token
    }
    
    // 请求拦截：添加 Authorization 头
    func adapt(_ urlRequest: URLRequest, for session: Session) throws -> URLRequest {
        var request = urlRequest
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return request
    }
    
    // 响应拦截：处理错误（例如 401 错误）
    func retry(_ request: Request, for session: Session, dueTo error: Error) -> RetryResult {
        if let afError = error as? AFError, afError.responseCode == 401 {
            print("Token 过期，进行重试")
            // 这里可以进行 token 刷新逻辑
            return .retry
        }
        return .doNotRetry
    }
}

// 初始化 Alamofire session
let interceptor = AuthInterceptor(token: "your-token-here")
let session = Session(interceptor: interceptor)

// 发起请求
session.request("https://api.example.com/data")
    .validate()
    .responseJSON { response in
        switch response.result {
        case .success(let value):
            print("成功获取数据：\(value)")
        case .failure(let error):
            print("请求失败：\(error)")
        }
    }
```

---

## ✅ 使用场景：

1. **添加认证头**：如果你的 API 都需要通过 token 认证，可以使用拦截器在所有请求中自动添加 `Authorization` 头，而不用每次都手动添加。
2. **请求重试**：比如当 token 过期时，自动触发刷新 token，并重新发起请求。
3. **日志记录**：你可以在拦截器中记录请求和响应的详细日志，帮助调试和监控。
4. **请求统一处理**：比如所有请求需要统一的 header 或是参数，可以通过拦截器统一设置。

---

## ✅ 总结：

`Alamofire` 的拦截器提供了一种强大且灵活的方式来处理请求的生命周期，从请求的适配到响应的处理，尤其适用于认证、重试、缓存等场景。你可以通过实现 `RequestInterceptor` 来拦截并修改请求和响应，提升代码的复用性和维护性。

---

如果你想更深入地了解其他拦截器的用法，比如日志记录、缓存管理等，欢迎告诉我，我可以继续为你讲解。


***
<br/><br/><br/>
> <h2 id="Alamofire的核心链式调用">Alamofire的核心链式调用</h2>

这两个方法是 Alamofire 的核心链式调用方法之一，分别用于：

* `validate(...)`：**验证 HTTP 响应是否有效**（如状态码或 MIME 类型）
* `response(...)`：**接收最终请求结果的回调处理**

下面我通过一个完整的例子来说明它们的作用和用法，并逐行解释：

---

## ✅ 示例代码（结合 `validate` 和 `response`）

```swift
import Alamofire

AF.request("https://jsonplaceholder.typicode.com/posts/1")
    .validate(statusCode: 200..<300) // 验证响应状态码
    .response(queue: .main) { response in
        // 处理原始 Data? 响应结果
        switch response.result {
        case .success(let data):
            if let data = data, let str = String(data: data, encoding: .utf8) {
                print("响应字符串：\n\(str)")
            } else {
                print("无数据或解码失败")
            }
        case .failure(let error):
            print("请求失败：\(error.localizedDescription)")
        }
    }
```

---

## 🔍 解读 `validate(_:)` 和 `response(...)` 方法

### 🔹1. `validate(...)`：对服务器返回做「前置校验」

```swift
.validate(statusCode: 200..<300)
```

它等价于：

```swift
.validate { request, response, data in
    if (200..<300).contains(response.statusCode) {
        return .success(Void())
    } else {
        return .failure(AFError.responseValidationFailed(reason: .unacceptableStatusCode(code: response.statusCode)))
    }
}
```

#### ✅ 它能做什么：

* 状态码校验（默认支持 `200..<300` 范围）
* 响应 Content-Type 校验
* 自定义验证逻辑

#### ❗️如果 `validate` 失败，`response.result` 会直接变成 `.failure(...)`，不会进入 `.success`

---

### 🔹2. `response(queue:completionHandler:)`：获取最终原始响应

```swift
.response(queue: .main) { response in
    // AFDataResponse<Data?> 类型
}
```

#### ✅ 参数含义：

* `queue: DispatchQueue = .main`：你希望在哪个线程回调（主线程、后台等）
* `completionHandler`：闭包回调，传回的是 `AFDataResponse<Data?>`

#### 响应结构（`AFDataResponse<Data?>`）包含：

| 属性         | 说明                                           |
| ---------- | -------------------------------------------- |
| `request`  | 原始 URLRequest                                |
| `response` | URLResponse（可查看 statusCode 等）                |
| `data`     | 原始响应 `Data?`                                 |
| `metrics`  | 网络请求时间等信息                                    |
| `result`   | 成功或失败：`.success(Data?)` or `.failure(Error)` |

---

## ✅ 想获取 JSON？用 `.responseJSON` 或 `.responseDecodable`

如果你不是想手动处理 `Data?`，可以用：

```swift
AF.request("...").validate().responseJSON { response in
    ...
}
```

或

```swift
struct Post: Decodable {
    let id: Int
    let title: String
}

AF.request("...").validate().responseDecodable(of: Post.self) { response in
    ...
}
```

---

## ✅ 小结

| 方法                                     | 功能                             |
| -------------------------------------- | ------------------------------ |
| `.validate(...)`                       | 请求结果前置校验，失败就不会进入 `.success`    |
| `.response(...)`                       | 拿原始 `Data?`，可自定义处理（如手动解析 JSON） |
| `.responseJSON` / `.responseDecodable` | 更高级的 JSON / 模型解码响应处理           |

---

<br/><br/>
**一个完整的 POST 请求 + 校验 + 解码模型的例子：**

好的！下面是一个完整的 Alamofire 使用例子，包括：

* **发送一个 POST 请求**
* **携带 JSON 参数**
* **使用 `.validate()` 校验状态码**
* **使用 `.responseDecodable` 解码响应为 Swift 模型**
* **逐行解释每部分的作用**

---

## ✅ 示例代码：发送 POST 请求并解析 JSON 响应

```swift
import Alamofire

// 1. 定义发送的参数
let parameters: [String: Any] = [
    "title": "Hello",
    "body": "This is a test post",
    "userId": 1
]

// 2. 定义期望返回的模型结构
struct PostResponse: Decodable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

// 3. 发起 POST 请求
AF.request(
    "https://jsonplaceholder.typicode.com/posts", // 模拟 API 地址
    method: .post,
    parameters: parameters,
    encoding: JSONEncoding.default  // 使用 JSON 编码请求体
)
.validate(statusCode: 200..<300) // 4. 校验响应状态码为 2xx
.responseDecodable(of: PostResponse.self) { response in // 5. 解码响应为模型
    switch response.result {
    case .success(let post):
        print("✅ 请求成功：")
        print("Post ID: \(post.id)")
        print("Title: \(post.title)")
    case .failure(let error):
        print("❌ 请求失败：\(error)")
    }
}
```

---

## 🧠 分析解读

| 步骤                                     | 含义                           |
| -------------------------------------- | ---------------------------- |
| 1️⃣ `parameters`                       | 传入 POST 请求的 JSON 参数          |
| 2️⃣ `PostResponse`                     | 结构体模型，符合 `Decodable`，用于解析响应  |
| 3️⃣ `AF.request(...)`                  | 使用默认 Session 发起 POST 请求      |
| 4️⃣ `.validate(statusCode: 200..<300)` | 如果响应不是 2xx，会进入 `.failure` 分支 |
| 5️⃣ `.responseDecodable(of:)`          | 自动将 JSON 响应转换为你定义的结构体        |

---

## 🔁 请求响应示例（模拟 API 的实际返回）

```json
{
  "id": 101,
  "title": "Hello",
  "body": "This is a test post",
  "userId": 1
}
```

---

## ✅ 适用场景

这个模式适合：

* 向 REST API 发送结构化 JSON 请求
* 处理服务器返回的数据模型
* 自动进行错误处理、类型解析、验证状态码

---


<br/><br/>
需要我再给一个带 headers、拦截器或 token 处理的进阶版本吗？


好的，我们现在来写一个**进阶版 Alamofire 网络请求示例**，包括：

---

### ✅ 进阶功能点：

* 携带 **Authorization Token**
* 使用 **自定义拦截器（`RequestInterceptor`）**
* 自动 **重试失败请求（如 401）**
* 返回数据解析为模型（`Decodable`）

---

## 🧱 Step 1：定义模型结构（用于响应解码）

```swift
struct UserProfile: Decodable {
    let id: Int
    let name: String
    let email: String
}
```

---

## 🔐 Step 2：自定义拦截器（自动添加 Header + 401 重试逻辑）

```swift
class AuthInterceptor: RequestInterceptor {
    
    private var retryCount = 0
    private let maxRetryLimit = 2
    
    private var token: String {
        // 真实项目中建议从 Keychain 或 Token 管理器获取
        return "Bearer your_access_token_here"
    }
    
    // 添加 Authorization header
    func adapt(_ urlRequest: URLRequest, for session: Session) async throws -> URLRequest {
        var request = urlRequest
        request.setValue(token, forHTTPHeaderField: "Authorization")
        return request
    }
    
    // 处理重试逻辑（如 token 过期时）
    func retry(_ request: Request, for session: Session, dueTo error: Error) async -> RetryResult {
        // 判断是否是 401 错误，并限制重试次数
        if let response = request.task?.response as? HTTPURLResponse,
           response.statusCode == 401, retryCount < maxRetryLimit {
            retryCount += 1
            print("⚠️ 检测到 401，尝试刷新 token 并重试（第 \(retryCount) 次）")
            return .retryWithDelay(1.0)
        }
        return .doNotRetry
    }
}
```

---

## 🚀 Step 3：使用自定义 `Session` 发送请求

```swift
// 创建一个带拦截器的 Session 实例
let session = Session(interceptor: AuthInterceptor())

session.request("https://api.example.com/profile", method: .get)
    .validate(statusCode: 200..<300)
    .responseDecodable(of: UserProfile.self) { response in
        switch response.result {
        case .success(let profile):
            print("✅ 用户信息：\(profile.name) - \(profile.email)")
        case .failure(let error):
            print("❌ 请求失败：\(error.localizedDescription)")
        }
    }
```

---

## 🧠 重点说明

| 功能       | 代码逻辑                                     |
| -------- | ---------------------------------------- |
| 添加 Token | 在 `adapt(...)` 中为每个请求加 Header            |
| 自动重试     | `retry(...)` 中判断 401，延迟后重新发起请求           |
| 状态码校验    | `.validate(statusCode: 200..<300)`       |
| 解码响应     | `.responseDecodable(of:)` 自动解析 JSON 到结构体 |

---

## 🧪 模拟 JSON 响应（来自 API）

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

## ✅ 总结：适合这类场景

* 有登录验证的 API（需要 Token）
* Token 可能会过期，需要重试
* 返回结构化 JSON，需要解析模型
* 想要集中管理 Header 和错误处理逻辑

---

是否需要我演示一个带 **刷新 token 逻辑** 的完整拦截器？或者用 Swift Concurrency（`async/await`）改写这个流程？

非常好！我们现在来写一个**带刷新 Token 逻辑的 Alamofire 拦截器（AuthInterceptor）**。这个版本适用于：

* API 使用 **访问令牌（Access Token）+刷新令牌（Refresh Token）**
* 当请求返回 **401 Unauthorized** 时，自动尝试刷新 Token 并重试请求

---

## ✅ 总体思路

1. 所有请求自动添加 `Authorization: Bearer xxx`
2. 如果遇到 401 错误：

   * 调用刷新 token 接口
   * 更新本地 token
   * 重新发起原始请求

---

## 🧱 模拟 Token 存储与刷新服务

```swift
class TokenManager {
    static var accessToken = "expired_access_token"
    static var refreshToken = "valid_refresh_token"

    static func refreshAccessToken(completion: @escaping (Bool) -> Void) {
        print("🔄 正在刷新 Token...")

        // 模拟网络延迟与刷新请求
        DispatchQueue.global().asyncAfter(deadline: .now() + 1.0) {
            accessToken = "new_valid_access_token"
            print("✅ Token 刷新成功：\(accessToken)")
            completion(true)
        }
    }
}
```

---

## 🔐 带刷新逻辑的 AuthInterceptor 实现

```swift
class RefreshingAuthInterceptor: RequestInterceptor {

    private var isRefreshing = false
    private var retryQueue: [(RetryResult) -> Void] = []

    // 添加 Authorization 头部
    func adapt(_ urlRequest: URLRequest, for session: Session) async throws -> URLRequest {
        var request = urlRequest
        request.setValue("Bearer \(TokenManager.accessToken)", forHTTPHeaderField: "Authorization")
        return request
    }

    // 处理 401 重试逻辑
    func retry(_ request: Request, for session: Session, dueTo error: Error) async -> RetryResult {
        guard let response = request.response as? HTTPURLResponse,
              response.statusCode == 401 else {
            return .doNotRetry
        }

        return await withCheckedContinuation { continuation in
            retryQueue.append { result in
                continuation.resume(returning: result)
            }

            guard !isRefreshing else { return }

            isRefreshing = true

            TokenManager.refreshAccessToken { success in
                self.isRefreshing = false
                let result: RetryResult = success ? .retry : .doNotRetry
                self.retryQueue.forEach { $0(result) }
                self.retryQueue.removeAll()
            }
        }
    }
}
```

---

## 🚀 使用带刷新功能的拦截器发起请求

```swift
let session = Session(interceptor: RefreshingAuthInterceptor())

session.request("https://api.example.com/user/profile")
    .validate()
    .responseJSON { response in
        switch response.result {
        case .success(let data):
            print("✅ 成功：\(data)")
        case .failure(let error):
            print("❌ 失败：\(error.localizedDescription)")
        }
    }
```

---

## ✅ 整体工作流程图：

```
请求 ➜ 失效的 Token ➜ 响应 401 ➜ interceptor:
                 ↳ 刷新 Token ➜ 更新 token ➜ 重试原请求
```

---

## 🧠 小提示

* 使用 `withCheckedContinuation` 实现 async/await 控制同步刷新行为
* 真实项目中可配合 `Keychain` 管理 `token`，刷新接口返回后存储新 token




***
<br/><br/><br/>
># <h2 id="json数据本地库解析">[json数据本地库解析](./../数据.md#Codable与数据模型的转换)</h2>

当进行网络请求时，除了使用成熟的解析库，比如： **SwiftJson**，我们还可以使用**Swift**自带的解析库`Cotobal`


<br/><br/><br/>

***
<br/>

> <h1 id="CocoaMQTT5库">[CocoaMQTT5库](./../Swift/CocoaMQTT5消息推送)</h1>





<br/><br/><br/>

***
<br/>

> <h1 id= "本地数据库RealmSwift">本地数据库RealmSwift</h1>


***
<br/><br/>RealmSwift简单实用Demo<br/>
> <h2 id="">RealmSwift简单实用Demo</h2>

`realm-swift` 是 Realm 提供的官方 Swift 库，是一个 **高性能、轻量级的本地数据库框架**，用于 iOS、macOS、watchOS 和 tvOS 的数据持久化。

它的特点是：

* 🚀 比 Core Data 更简单、更快
* 📦 支持对象模型存储（ORM），使用 Swift 对象直接存储数据
* 🔄 支持线程切换、安全写入、数据自动刷新
* 🛠 离线可用，适合本地存储用户数据、缓存等场景

---

## ✅ 用途场景

| 场景           | 用途                       |
| ------------ | ------------------------ |
| 本地持久化        | 保存用户信息、设置、收藏、历史记录等       |
| 离线缓存         | API 响应内容缓存本地             |
| 多线程写入        | 支持多线程操作，避免数据冲突           |
| 替代 Core Data | 更易上手、性能优秀，适合替代 Core Data |

---

## 🔧 安装方式（使用 CocoaPods）

```ruby
pod 'RealmSwift'
```

安装后你需要在使用文件中导入：

```swift
import RealmSwift
```

---

## 🧪 示例：使用 Realm 存储一个用户模型

### 1. 定义模型（继承自 `Object`）

```swift
class User: Object {
    @objc dynamic var id: String = UUID().uuidString
    @objc dynamic var name: String = ""
    @objc dynamic var age: Int = 0

    override static func primaryKey() -> String? {
        return "id"
    }
}
```

---

### 2. 插入数据（写入）

```swift
let user = User()
user.name = "Alice"
user.age = 30

let realm = try! Realm()
try! realm.write {
    realm.add(user)
}
```

---

### 3. 查询数据

```swift
let realm = try! Realm()
let users = realm.objects(User.self).filter("age > 25")

for user in users {
    print("👤 \(user.name), 年龄 \(user.age)")
}
```

---

### 4. 更新数据

```swift
if let user = realm.objects(User.self).first {
    try! realm.write {
        user.age += 1
    }
}
```

---

### 5. 删除数据

```swift
if let user = realm.objects(User.self).first {
    try! realm.write {
        realm.delete(user)
    }
}
```

---

## ✅ 与 Core Data 对比

| 特性   | Realm            | Core Data            |
| ---- | ---------------- | -------------------- |
| 上手难度 | ⭐ 简单             | 😵‍ Core Data 配置复杂   |
| 数据模型 | Swift 类/对象       | `.xcdatamodeld` 文件建模 |
| 写入方式 | `realm.write {}` | `context.save()`     |
| 多线程  | ✅ 自动处理           | 🚧 容易出错，需要管理 context |
| 查询语法 | 类似 SQL 的谓词       | NSFetchRequest（更复杂）  |

---

## 🔐 是否安全？

* Realm 支持加密存储（使用 256-bit AES）
* 支持数据迁移、版本控制
* 官方商业服务（Realm Cloud）已整合到 MongoDB Atlas，但本地数据库是免费开源的

---

## 🧩 总结

| 特点   | 说明                   |
| ---- | -------------------- |
| 简洁   | 模型类继承自 `Object`，写法直观 |
| 性能好  | 数据库为 C++ 底层，读写性能优越   |
| 易于维护 | 不需要管理复杂的上下文、模型文件     |
| 支持异步 | 支持多线程查询和写入操作         |

---

需要我帮你写一个完整的“Realm + MVVM”架构的样例项目结构吗？例如：保存登录用户、查询用户列表、响应 UI 更新。


<br/><br/><br/>

***
<br/>

> <h1 id="GCDAsyncSocket及时通讯">[GCDAsyncSocket及时通讯](./../Swift/GCDAsyncSocket.md)</h1>

这个及时通讯用在ap连接的时候，获取设备发来的信息的。




<br/><br/><br/>

***
<br/>

> <h1 id="Git操作">Git操作</h1>

***
<br/><br/><br/>
> <h2 id="添加忽略文件">添加忽略文件</h2>

要给你的 iOS 项目添加一个 `.gitignore` 文件并正确配置，建议按照以下步骤操作：

---

### ✅ 步骤 1：在项目根目录创建 `.gitignore` 文件

在你的 iOS 项目根目录下运行以下命令创建 `.gitignore` 文件：

```bash
touch .gitignore
```

---

### ✅ 步骤 2：添加 iOS 项目常用忽略配置

你可以手动粘贴下面这份 Apple 官方推荐的 `.gitignore` 配置（来自 GitHub 的 [gitignore 模板](https://github.com/github/gitignore/blob/main/Swift.gitignore)）：

```gitignore
## macOS
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon
._*

# Thumbnails
._.DS_Store

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

## Xcode
build/
DerivedData/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata/
*.xccheckout
*.moved-aside
*.xcuserstate

# Swift Package Manager
.swiftpm/
.build/

# CocoaPods
Pods/

# Carthage
Carthage/Build/

# Accio dependency manager
Dependencies/
.accio/

# fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots
fastlane/test_output

# Code coverage
*.xcresult
*.profdata

# Firebase
GoogleService-Info.plist

# Tuist
/.tuist-cache
/Dependencies

# Xcode Cloud
*.xcodecloud/

# Other
*.log
```

你也可以通过命令从 GitHub 获取最新版 Swift 或 Objective-C 的 `.gitignore` 文件：

```bash
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Swift.gitignore
```

---

### ✅ 步骤 3：将 `.gitignore` 应用到已存在的 Git 项目

如果项目已经初始化过 Git，执行以下命令刷新忽略状态：

```bash
git rm -r --cached .
git add .
git commit -m "Update .gitignore and remove ignored files"
```

---

### 🔁 备注：

* 如果你使用的是 **CocoaPods** 或 **Swift Package Manager**，确保忽略了 `Pods/`、`.build/` 等目录。
* **不要** 忽略 `*.xcodeproj` 或 `*.xcworkspace`，除非你明确知道为什么。

是否需要我帮你生成一份适合你项目的 `.gitignore` 文件内容？



<br/><br/><br/>

## **命令 `git rm -r --cached .` 的作用是**：

### 🧩 **从 Git 索引中移除所有已追踪文件，但不删除本地文件**

* `rm` 是 Git 的移除命令；
* `--cached` 表示**只从 Git 的索引中移除，不删除本地磁盘上的文件**；
* `-r` 是递归移除（针对文件夹）；
* `.` 表示当前目录下的所有内容。

---

### 🧠 为什么要这样做？

因为你：

> **在项目已经初始化并提交过 Git 的前提下，才添加了 `.gitignore` 文件。**

这时，Git 仍然追踪那些你现在希望忽略的文件（如 `Pods/`、`build/` 等）。即使你写了 `.gitignore`，**Git 不会自动停止追踪已经加入版本控制的文件**。

#### ✅ 这个命令的作用是：

* 清空 Git 的缓存索引；
* 让 `.gitignore` 生效；
* 重新根据 `.gitignore` 添加新的追踪文件列表。

随后你再执行：

```bash
git add .
git commit -m "Update .gitignore and remove ignored files"
```

就能让 Git 只追踪你希望保留的文件，忽略那些被 `.gitignore` 过滤掉的内容。


<br/><br/><br/>

***
<br/>

> <h1 id="CocoaPod安装配置">CocoaPod安装配置</h1>

为了使Mac的2种shell： bash、zshrc都可以使用`.bash_profile`中的配置需要在 `.zshrc` 中做如下配置（bash中默认加载`.bash_profile`文件）：

```sh
open ~/.zshrc
```

在其中这样配置：

```sh
# 作用： 为了Mac自带的shell的zsh使用bash的 bash_profile 配置
if [ -f ~/.bash_profile ]; then
  source ~/.bash_profile
fi
#End
```

<br/><br/>

然后在终端官网安装**brew**工具，我就不贴了，但是在M4Pro芯片需要做如下配置：

```sh
#brew环境变量配置
export PATH="/opt/homebrew/bin:$PATH"
#End
```

<br/><br/>

然后还需要安装ruby，当然ruby也需要在`.bash_profile`中做如下配置：

```sh
#cocoapods安装需要的配置
#将 Ruby 的路径添加到 PATH 中
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
#为编译器设置正确的路径：
export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"
#如果你使用 pkg-config，还需要设置它的路径
export PKG_CONFIG_PATH="/opt/homebrew/opt/ruby/lib/pkgconfig"
#cocoapoes 安装End
```

<br/><br/><br/>

***
<br/>

> <h1 id="概念">概念</h1>


***
<br/><br/><br/>
> <h2 id="p2p连接">p2p连接</h2>

在摄像头（IPC，**IP Camera**）监控领域里，
“IPC 连接”通常指的是 **网络摄像头与客户端（手机 App、NVR、网页等）之间建立的通信方式**，并不专指某一个协议。
它常常会包含 **直连（P2P）**、**局域网直连**、**转发/中继** 等多种模式，选择哪种要看网络环境和厂商实现。

---

### 📡 IPC 常见的连接方式

| 方式                     | 特点                                                   | 适用场景                               |
| ---------------------- | ---------------------------------------------------- | ---------------------------------- |
| **局域网直连（IP 直连）**       | 客户端直接用摄像头的 IP 地址（`rtsp://`, `http://`, `onvif://`）访问 | 摄像头和客户端在同一局域网，没有 NAT 问题            |
| **P2P 直连**             | 利用服务器做“信令/打洞”，之后客户端和摄像头点对点传输音视频                      | 手机通过 4G/5G 访问家里的摄像头，网络双方都支持 NAT 穿透 |
| **中继/转发（Relay）**       | 如果 P2P 穿透失败，数据走厂商的云服务器转发                             | NAT 太严或设备不支持打洞时                    |
| **RTMP/RTSP over TCP** | 传统流媒体协议，通过公网 TCP 连接摄像头或云服务                           | 直播、录像、回放等                          |
| **WebRTC**             | 现代浏览器/移动端常用方案，基于 UDP，支持 STUN/TURN，延迟低                | 低延迟实时预览（很多新款摄像头走这个）                |

---

### 🔍 P2P 在 IPC 里的含义

* “P2P” 不是协议名，而是 **Point-to-Point** 的缩写。
* 实际实现通常用：

  * UDP 打洞（NAT 穿透）+ 自定义协议
  * 或 WebRTC（它内置 STUN/TURN 服务）
* 优点：延迟低，不占云服务器带宽。
* 缺点：有些网络（公司防火墙、运营商 NAT）会导致 P2P 失败，需要自动回退到中继。

---

### 📌 结论

* **IPC 连接** 是一个泛称：表示客户端如何与摄像头通信。
* 如果厂商文档里提到 “IPC P2P 连接”，通常是：
  设备通过云端做“信令握手”，然后直接建立 **点对点音视频通道**（大多走 UDP / WebRTC）。
* 当 P2P 不可用时，会自动切换到 **Relay/转发** 模式。

---

💡 小贴士

* 如果你的 App/设备 SDK 里看到 `ICE server`、`STUN`、`TURN` 等字样，基本就是在用 WebRTC/P2P 技术。
* 如果只看到 `RTSP`、`RTMP` URL，多半是传统直连或转发，不一定是 P2P。


***
<br/><br/><br/>
> <h2 id="UDP打洞">UDP打洞</h2>


“UDP 打洞”全称是 **UDP hole punching**，中文一般叫 **UDP 打洞**，是做 **NAT 穿透（NAT traversal）** 时最常见的一种方法。

---

### 📌 背景

* 家用路由器、蜂窝网络等大多用 **NAT（网络地址转换）**，把多台设备的内网地址映射到一个公网 IP。
* NAT 会阻止外部设备直接发包到你的内网设备，所以两端都在 NAT 后面时，点对点通信（P2P）就不好建立。

---

### 💡 UDP 打洞是怎么工作的

核心思路：

> 让两端同时“对着”彼此的公网地址/端口发一个 UDP 包，借此在 NAT 上开一个临时映射通道。

流程大致如下：

1. **客户端 A / 客户端 B** 各自连到一个公网的 “信令/协调” 服务器（常叫 rendezvous server）。
2. 服务器记录它们的 **公网 IP:端口**，然后把这个信息告诉对方。
3. A、B 同时尝试用 UDP 向对方的 “公网 IP:端口” 发包。
4. 大多数 NAT 看到“内网设备对某个外部地址发包”后，会建立映射并允许对方回包。
5. 这样 A ↔ B 之间就能直接传 UDP 数据了（不再需要中继）。

> 这个“打洞”其实是 **让 NAT 打开一条回包路径**，并不是在防火墙上真的打一个洞 😂

---

### ✨ 特点

* **优点**：

  * 延迟低，数据不经云服务器转发，省带宽。
* **缺点**：

  * 依赖 NAT 类型，某些对称 NAT（symmetric NAT）下成功率不高，需要 TURN 中继兜底。

---

### 📌 常用场景

* P2P 语音/视频通话（VoIP、WebRTC）。
* 在线游戏（玩家之间的低延迟数据）。
* 智能摄像头 P2P 预览。

---

### 🔎 相关名词

| 名词       | 作用                                  |
| -------- | ----------------------------------- |
| **STUN** | 辅助发现“我在公网看来是什么 IP:端口”               |
| **TURN** | 当打洞失败时，走 TURN 服务器做中继                |
| **ICE**  | WebRTC 里的候选传输方式集合，里面包含 STUN/TURN/直连 |

---

简单来说：

> **UDP 打洞 = 借助公网服务器交换信息 + 双方同时发 UDP 包 → 在 NAT 上开出一条允许彼此通信的通道**。


<br/><br/>

这里是 **UDP 打洞的详细时序图**（配合之前的示意图一起看会更直观）：

---

### 🕒 UDP 打洞时序图

```
Client A             Rendezvous Server           Client B
  |                         |                       |
  |----1. 注册 & 发送本地地址-->                   |
  |                         |<---2. 注册 & 发送本地地址---|
  |<---3. 返回 B 的公网地址---|                       |
  |                         |---4. 返回 A 的公网地址--->|
  |----5. 向 B 的公网地址发 UDP 包 ------------------>|
  |<-------------------6. 向 A 的公网地址发 UDP 包----|
  |                         |                       |
  |<=========== 7. 建立 P2P 通道 ============>|
  |                         |                       |
```

#### 步骤说明

1. **A 注册**：A 向服务器报告自己的“外网可见地址”和本地端口。
2. **B 注册**：B 同样把自己的公网/内网信息发给服务器。
   3 & 4. 服务器把双方的“公网地址:端口”告诉对方。
   5 & 6. A、B 同时尝试对着对方的公网地址发送 UDP 包（“打洞”）。
3. 一旦双方 NAT 都建立了映射，就能直接收发数据，实现 **点对点 UDP 通信**。

> 如果步骤 5/6 失败，通常会退回到 TURN/Relay 转发。

---

这样你就可以同时结合：

* **时序图（步骤 & 消息顺序）**
* **拓扑示意图（NAT 映射关系）**

来完整理解 UDP 打洞的流程了。


<br/><br/><br/>

***
<br/>

> <h1 id="工程Bug">工程Bug</h1>


***
<br/><br/><br/>
> <h2 id="析构调用闭包导致crash">析构调用闭包导致crash</h2>

在iOS中的**BleUtil.m**中有：

```oc
- (void)dealloc {
    
    [self invalidateAndClear];
}
- (void) invalidateAndClear { [self stopScanPeripherals];
    
}

- (void)stopScanPeripherals {
    
    AKLogV(TAG, @"stopScanPeripherals");
    
    if (self.scanTimer) {
        dispatch_source_cancel(self.scanTimer);
        self.scanTimer = nil;
    }
    [self.centralManager stopScan];
    [self clearDiscoverPeripherals];
    
    dispatch_async(dispatch_get_main_queue(), ^{
        if (self.scanBleCallBackBlock) {
            AKBleInfoModel *model = [[AKBleInfoModel alloc] initWithHandStopScan:YES];
            self.scanBleCallBackBlock(model);// ❌ dealloc 时调用，block 内可能访问了 self
        }
    });
    
}
```


执行析构方法`dealloc`后，到了`if (self.scanBleCallBackBlock) {}`直接崩溃报错：**`Thread 1: EXC_BAD_ACCESS (code=1, address=0x731ac6290)`**

<br/>

- **经分析，崩溃的原因是：**
	- `dealloc` 调用时，`self` 已经在释放。
	- 在 `dealloc` 里调了 `invalidateAndClear → stopScanPeripherals`。
	- `stopScanPeripherals` 里用了 异步回调：`dispatch_async(dispatch_get_main_queue(), ^{ ... });`

这会延迟执行 block。

等到 block 真正执行时，self 已经销毁了，
**→ 访问 self.scanBleCallBackBlock 时内存已被回收，导致 EXC_BAD_ACCESS 崩溃。**

<br/>

一开始想通过 **`弱引用weak`【弱引用 self，避免野指针】** 进行解决的，如下：

```oc
__weak typeof(self) weakSelf = self;
dispatch_async(dispatch_get_main_queue(), ^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (!strongSelf) return; // self 已释放，直接退出
    if (strongSelf.scanBleCallBackBlock) {
        AKBleInfoModel *model = [[AKBleInfoModel alloc] initWithHandStopScan:YES];
        strongSelf.scanBleCallBackBlock(model);
    }
});
```

- 崩溃原因：dealloc 后异步 block 仍在访问已释放的 self。
- 最推荐的做法：不要在 dealloc 中再做业务回调，只清理资源即可。
- 如果必须回调，用 weakSelf + nil block 防止野指针。

<br/>

我像上面那样做后，在`__weak typeof(self) weakSelf = self;`报了一个`‌Thread 13: abort with payload or reason `直接就crash了。

这很可能不是 `__weak` 本身的问题，而是**`清理顺序 / 竞态 / 其它对象（比如 CBCentralManager / CBPeripheral / block 内部捕获的外部对象）`**在 `self` 被释放后仍在异步回调里被访问，或者 GCD timer 的取消方式不安全。

<br/>


**拓展：常见原因（为什么 weakSelf 却还崩溃）**

- CBCentralManager / CBPeripheral 仍会异步回调 delegate：如果 centralManager.delegate = self，在你释放 self 之后，系统仍可能异步调用 delegate，导致访问野指针。

- scanBleCallBackBlock 本身捕获了 VC 或其它已释放对象，调用它会触发崩溃（即使你用 weakSelf 检查了 self）。

- dispatch_source_t（timer） 的取消/挂起状态处理不当：若 timer 曾被 dispatch_suspend 而没被 resume，再 dispatch_source_cancel / release 可能出错。

- 异步 block 内部抛异常（未捕获）会导致 abort；日志里能看到异常原因。

- 竞态：你在某线程把 block 置 nil，同时另一个线程又在读取/执行，导致不安全访问。

<br/><br/>
***

解决方案是对闭包回调这块做一些处理：

```oc
// 4) 安全地拷贝并置空回调属性，然后在主线程调用拷贝（避免 race）
void (^callback)(AKBleInfoModel *) = nil;
@synchronized (self) {
    if (self.scanBleCallBackBlock) {
        callback = [self.scanBleCallBackBlock copy];
        self.scanBleCallBackBlock = nil; // 立即置空，防止别处再调用
    }
}

if (callback) {
    dispatch_async(dispatch_get_main_queue(), ^{
        @try {
            AKBleInfoModel *model = [[AKBleInfoModel alloc] initWithHandStopScan:YES];
            callback(model);
        } @catch (NSException *ex) {
            NSLog(@"Exception when calling scan callback in stopScanPeripherals: %@", ex);
        }
    });
}
```

- **需要注意的是：**
	- 先把 `scanBleCallBackBlock` 拷贝到局部变量并立刻把属性置 nil，能防止别处并发访问到已经释放的 `block` 指针。
	- 在主线程上执行回调并用 `@try/@catch` 捕获异常（用于调试和避免直接 abort），但这不是最终解决方案——真正要找出回调里是否访问了已释放对象。
	- 处理 `dispatch_source_t` 要小心 `suspended/resume` 状态，避免在 `suspended 状态下直接 cancel。`

<br/>

**‼️：上面的代码最好不要放在析构方法dealloc中，容易引起crash问题。最好不要做这种在block回调或者线程操作。**


***
<br/><br/><br/>
> <h2 id="线程堆栈分析报告如何查看？">线程堆栈分析报告如何查看？</h2>

**比如这段Swift线程崩溃报告，如何解读？**

```shException Type:  EXC_BAD_ACCESS (SIGBUS)
Exception Codes: 0x0000000000000002, 0x0000000ff6bb5d18
Termination Reason: SIGNAL 10

Triggered by Thread:  15
```


上述崩溃信息是 `iOS` 里比较常见的一类底层异常，内容解析如下：

<br/> 

**1️⃣ `Exception Type:  EXC_BAD_ACCESS (SIGBUS)`**

* **EXC\_BAD\_ACCESS**
  表示你的程序访问了不该访问的内存地址（访问“坏地址”）。
* 括号里的 **SIGBUS**
  指触发异常的具体信号：**Bus Error**。
  这通常意味着：
  
	* 访问了一块已经释放/无效的内存；
	* 或者访问地址未对齐（对齐错误），例如用 `UnsafePointer` 读写未按类型对齐的内存；
	* 或者读写的物理页当前不可用（极少见）。

> 简单说：**程序在尝试读取/写入一段无效或未对齐的内存时被系统杀掉**。

<br/>

**2️⃣ `Exception Codes: 0x0000000000000002, 0x0000000ff6bb5d18`**

* 第一个 `0x2` 通常是异常的子类型。
* 第二个是触发问题的内存地址：`0x0000000ff6bb5d18`
  你可以在调试器里看这个地址是否落在某个对象区域里，帮助确认是哪块内存。

<br/>

**3️⃣ `Termination Reason: SIGNAL 10`**

* **SIGNAL 10** 就是 POSIX 信号 `SIGBUS`，和上面的括号一致。

<br/>

**4️⃣ `Triggered by Thread: 15`**

* 表示在 **线程 15** 上触发的崩溃，不一定是主线程。
  说明有并发或后台任务在操作内存。
* 然后你可以在堆栈报告下找到线程是**15**编号的，然后查看是执行到哪行代码造成的crash

<br/>

**常见成因**

| 场景                | 说明                                            |
| ----------------- | --------------------------------------------- |
| 访问已释放对象           | 比如 `weak` 引用没判空，或 `unowned` 被释放后再访问           |
| 原生指针错误            | `UnsafePointer` / `UnsafeMutablePointer` 使用不当 |
| 对齐错误              | 用 `UnsafeRawPointer` 转换结构体时地址没对齐              |
| 多线程写内存            | 两个线程同时修改对象，没做同步                               |
| CoreData / SQLite | 数据页损坏，或被错误地并发访问                               |
| C/ObjC 库          | 用 C API 写错缓冲区长度，或传了无效地址                       |


<br/><br/>

比如通过查看**`线程编号15`**这段日志，如下：

```sh
Thread 15 name:   Dispatch queue: com.cm.bleQueue
Thread 15 Crashed:
0   libobjc.A.dylib                        0x1883f3b98 objc_retain + 16
1   argus-app-ios-overseas                 0x104f1eb8c -[AKBleUtil stopTimeoutTimer] + 1584012 (AKBleUtil.m:191)
2   argus-app-ios-overseas                 0x104f1ee84 -[AKBleUtil stopScanPeripherals] + 1584772 (AKBleUtil.m:224)
3   argus-app-ios-overseas                 0x104f1c620 -[AKBleManager finishWithSuccess:peripheralMac:error:] + 1574432 (AKBleManager.m:320)
4   argus-app-ios-overseas                 0x104f1c550 -[AKBleManager stopPollConnectBle:] + 1574224 (AKBleManager.m:305)
5   argus-app-ios-overseas                 0x104f1d010 -[AKBleManager connectBluetoothDeviceWithUUID:] + 1576976 (AKBleManager.m:444)
6   argus-app-ios-overseas                 0x104f20864 -[AKBleUtil centralManager:didConnectPeripheral:] + 1591396 (AKBleUtil.m:467)
7   CoreBluetooth                          0x1b472bc60 -[CBCentralManager handlePeripheralConnectionCompleted:] + 456
8   CoreBluetooth                          0x1b470aa88 -[CBCentralManager handleMsg:args:] + 224
9   CoreBluetooth                          0x1b470822c -[CBManager xpcConnectionDidReceiveMsg:args:] + 212
10  CoreBluetooth                          0x1b4708114 __30-[CBXpcConnection _handleMsg:]_block_invoke + 48
11  libdispatch.dylib                      0x192df4aac _dispatch_call_block_and_release + 32
12  libdispatch.dylib                      0x192e0e584 _dispatch_client_callout + 16
13  libdispatch.dylib                      0x192dfd2d0 _dispatch_lane_serial_drain + 740
14  libdispatch.dylib                      0x192dfdde0 _dispatch_lane_invoke + 440
15  libdispatch.dylib                      0x192dfd138 _dispatch_lane_serial_drain + 332
16  libdispatch.dylib                      0x192dfddac _dispatch_lane_invoke + 388
17  libdispatch.dylib                      0x192e081dc _dispatch_root_queue_drain_deferred_wlh + 292
18  libdispatch.dylib                      0x192e07a60 _dispatch_workloop_worker_thread + 540
19  libsystem_pthread.dylib                0x215588a0c _pthread_wqthread + 292
20  libsystem_pthread.dylib                0x215588aac start_wqthread + 8
```

<br/>

**1️⃣ 关键崩溃位置**

```
Thread 15 Crashed:
0   libobjc.A.dylib   objc_retain
1   argus-app-ios-overseas  -[AKBleUtil stopTimeoutTimer]  (AKBleUtil.m:191)
```

* `objc_retain` → Objective-C 在给对象做 `retain`（增加引用计数）的时候访问到了一块无效的内存。
* 出错点在 `AKBleUtil.m` 第 191 行，也就是 `stopTimeoutTimer` 里。

👉 **说明**：`stopTimeoutTimer` 里有一个对象被 `retain` 时，底层地址已经无效（通常是被释放掉了）。

<br/>

**2️⃣ 堆栈上下文**

```
AKBleUtil stopTimeoutTimer
 └─ stopScanPeripherals
     └─ AKBleManager finishWithSuccess:peripheralMac:error:
         └─ stopPollConnectBle:
             └─ connectBluetoothDeviceWithUUID:
                 └─ centralManager:didConnectPeripheral:
```

* 从调用顺序看，这是在 **蓝牙连接成功回调**（`didConnectPeripheral:`）里，调用了 `connectBluetoothDeviceWithUUID:` → `stopPollConnectBle:` → `finishWithSuccess:` → `stopScanPeripherals` → `stopTimeoutTimer`。
* 所以这是 CoreBluetooth 回调线程（你这里是 `com.cm.bleQueue`），在回调里调用了 `stopTimeoutTimer`。

<br/>

**3️⃣ 崩溃成因推测**

根据 `objc_retain` + `EXC_BAD_ACCESS (SIGBUS)` 的组合，常见原因是：

- 1.**对象已经被释放**
   `stopTimeoutTimer` 里可能访问了一个 `NSTimer`、`dispatch_source_t` 或 `self` 的某个属性，但那个对象已提前释放。
- 2.**多线程竞争**
   可能另一个线程把定时器或 `AKBleUtil` 自己释放了，而蓝牙回调线程同时在调用 `stopTimeoutTimer`。
- 3.**Block 循环引用/悬挂指针**
   如果定时器用 block 方式持有 `self`，处理不当也可能在释放时出现野指针。

<br/>

**4️⃣ 建议的排查方法**

| 步骤                     | 说明                                                                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| ① 检查 `AKBleUtil.m:191` | 找到 `stopTimeoutTimer` 的实现，看这一行访问了什么对象（`self.timer`？`dispatch_source`？）。                                                              |
| ② 确认对象生命周期             | 谁创建 / 谁释放 `timer` 或 `AKBleUtil`，是否在 `dealloc`/`invalidate` 中释放太早。                                                                    |
| ③ Zombie Objects       | Xcode → Edit Scheme → Diagnostics → **Enable Zombie Objects**，运行后如果再次崩溃，会显示 “message sent to deallocated instance xxx”，可以直接看到是哪一个对象。 |
| ④ Thread Sanitizer     | Scheme 里开启 Thread Sanitizer，看是否是数据竞争导致。                                                                                              |
| ⑤ 定时器弱引用               | 如果是 `NSTimer scheduledTimer...`，记得在 `invalidate` 之前保持强引用，否则 Timer 会释放掉 target，再回调时出错。                                                |

<br/>

**5️⃣ 结论**

> 崩溃点：
> `AKBleUtil - stopTimeoutTimer` 第 191 行
> 问题类型：访问已释放对象 / 悬挂指针
> 环境：CoreBluetooth 回调线程（`com.cm.bleQueue`）

下一步最重要的是：

* 打开 `AKBleUtil.m`，定位第 191 行，看是哪个对象被访问。
* 打开 “Enable Zombie Objects” 再复现崩溃，系统会告诉你确切的对象类型。

<br/>

后面的解决方法是将之前因为使用NSRunTimer作为定时器因为与其他多线程多次切换造成与其他线程的数据造成**竞态**导致一些对象过早示范。现在换成了使用**GCD**作为定时器，然后统一在**异步串行**统一处理，这样就解决了。




***
<br/><br/><br/>
> <h2 id="网址缓存导致无法加载">网址缓存导致无法加载</h2>

**出现的场合：** 最开始加载是好的，后面因为域名换了，导致无法加载网页。在手机的**Safari**是可以加载的，电脑的**Safari、Chrome**都是可以的。

网页「在浏览器可以打开，但在默认 WKWebView 不行」，大概率就是**缓存 / Cookie / 登录态冲突**导致的。

这个现象在 iOS 开发中其实很常见，跟 WKWebView 的「缓存 / Cookie / 本地存储机制」有关。

后来发现将`WKWebView`的缓存清理了就可以了。如下：


**修改前的代码（默认）**

```swift
let configuration = WKWebViewConfiguration()
// 默认使用 persistent（持久化）DataStore
let webView = WKWebView(frame: .zero, configuration: configuration)
```

**默认情况下：**

* `WKWebsiteDataStore.default()`（持久化）
* WebView 会自动保存 Cookie、LocalStorage、IndexedDB、Service Worker 缓存等
* 这些缓存会在 App 多次启动后持续存在

<br/>

**👉 这意味着：**

* 如果目标网页对 Cookie 或本地状态有依赖（比如登录态或特定的 session 校验），
* 旧的缓存 / Cookie 如果与服务器当前状态不匹配，就可能导致加载失败、跳转错误、白屏或被重定向循环卡住。

<br/>

**修改后的代码**

```swift
let configuration = WKWebViewConfiguration()
configuration.websiteDataStore = WKWebsiteDataStore.nonPersistent()

let webView = WKWebView(frame: .zero, configuration: configuration)
```

- **改成 `nonPersistent` 后的效果：**

	* 不使用任何持久化缓存
	* 每次打开页面，都是“全新的浏览器环境”
	* 不携带旧 Cookie / Session / 缓存
	* 与在浏览器中「无痕模式（隐私模式）」加载网页的效果一致

上述代码虽然可以解决问题，但是**一棒子打死了**。对于性能和优化不是太好了。比如：打开速度、资源等。

<br/>

**👉 所以网页能正常加载了，因为：**

* 清除了之前可能「污染」或「冲突」的 Cookie/Session
* 服务器重新下发新的 Token / Cookie
* 页面逻辑不再被旧状态卡住

<br/>

**可能的根本原因**

1. **旧 Cookie / Session 过期或无效**，服务器拒绝请求。
2. **缓存的 Service Worker / 本地存储异常**，导致页面无法正常更新。
3. **重定向循环或鉴权逻辑异常**，只有干净环境才能进入。
4. Web 页面对 Safari 与 WKWebView 的缓存行为处理不一致（这也是常见坑）。


***
<br/>

个人感觉最好的办法是，当**加载失败的时候然后清理，然后再次加载，如下：**

👉 可以在代理回调 `webView(_:didFailProvisionalNavigation:withError:)` 里调用：

```swift
func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
    print("加载失败：\(error.localizedDescription)")
    clearWebViewCache { [weak self] in
        self?.webView.reload()
    }
}

private func clearWebViewCache(completion: @escaping () -> Void) {
    let dataStore = WKWebsiteDataStore.default()
    let types = WKWebsiteDataStore.allWebsiteDataTypes()
    dataStore.fetchDataRecords(ofTypes: types) { records in
        dataStore.removeData(ofTypes: types, for: records) {
            print("✅ WKWebView 缓存已清除")
            completion()
        }
    }
}
```

这样只有在加载失败时才清理缓存，避免每次启动都清空。







