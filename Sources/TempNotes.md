- ⭕️
- [保护静态共享实例defaultManager]()
- [数组通过“+”拼接](#数组通过“+”拼接)
- [串行同步队列](#串行同步队列)
- [AKBLEDeviceRepository 串行队列死锁保护](#AKBLEDeviceRepository串行队列死锁保护)
- [DispatchWorkItem管理延迟或超时任务](#DispatchWorkItem管理延迟或超时任务) 
- [swift位移枚举使用](#swift位移枚举使用)
- [对象身份标识符](#对象身份标识符)
- [状态机枚举使用](#状态机枚举使用)
- [字符串中提取数字](#字符串中提取数字)










<br/><br/><br/>

***
<br/>

> <h1 id= "保护静态共享实例defaultManager">保护静态共享实例 defaultManager </h1>

```
public final class AKDeviceDiscoveryManager: NSObject {

    private static var defaultManager: AKDeviceDiscoveryManager?
    private static let defaultManagerLock = NSLock()
    
    private static func defaultDiscoveryManager() -> AKDeviceDiscoveryManager {

        defaultManagerLock.lock()
        defer { defaultManagerLock.unlock() }
        if let defaultManager = self.defaultManager {
            return defaultManager
        }
        let manager = AKDeviceDiscoveryManager()
        self.defaultManager = manager
        return manager
    }
    
      public static func releaseResources() {

        defaultManagerLock.lock()
        let manager = defaultManager
        defaultManager = nil
        defaultManagerLock.unlock()
        manager?.releaseResources()
    }
}
```

**为什么加入 defaultManagerLock**

加 defaultManagerLock 是为了保护静态共享实例 defaultManager 的创建和释放，避免多线程同时访问时出现竞态。
主要原因：
1. defaultManager 是全局共享状态
private static var defaultManager: AKDeviceDiscoveryManager?
静态变量会被所有调用方共享。比如不同页面、蓝牙状态回调、主线程/后台线程都可能调用：

```sh
AKDeviceDiscoveryManager.startDiscovery(scanRequest)
AKDeviceDiscoveryManager.stopDiscovery()
AKDeviceDiscoveryManager.releaseResources()
```
如果没有锁，多线程同时访问 defaultManager 时不安全。

2. 避免重复创建默认 manager
没有锁时可能发生：
```
Thread A: 发现 defaultManager == nil
Thread B: 发现 defaultManager == nil
Thread A: 创建 managerA
Thread B: 创建 managerB
Thread A: defaultManager = managerA
Thread B: defaultManager = managerB
```
结果是创建了两个 manager，可能导致：
- BLE/LAN 通道重复启动
- delegate 绑定混乱
- 旧 manager 仍持有扫描资源
- 超时任务和停止逻辑不一致
3. 避免释放和启动并发冲突
比如一个线程正在：
AKDeviceDiscoveryManager.startDiscovery(scanRequest)
另一个线程同时调用：
AKDeviceDiscoveryManager.releaseResources()
如果没有同步保护，可能出现：
- 刚创建的 manager 立刻被置空
- releaseResources() 释放的是旧 manager
- 新 manager 启动扫描但静态引用丢失，后续无法 stop
- BLE/LAN 扫描生命周期不可控
4. Swift 静态变量本身不保证这种读改写流程原子
static let 初始化是线程安全的，但这里是：
private static var defaultManager: AKDeviceDiscoveryManager?
并且有“读取 nil -> 创建 -> 赋值”和“读取 -> 置空 -> release”的复合操作。这个流程需要额外同步。
所以加锁保护的是这两段：
private static func defaultDiscoveryManager() -> AKDeviceDiscoveryManager
和：
public static func releaseResources()
这样可以保证默认实例创建、复用、置空释放是串行的。
如果你确认所有调用都只会发生在主线程，也可以不用 NSLock，改成更简单的主线程约束写法。但从 SDK 公共 API 角度，静态入口最好不要假设调用线程。


<br/><br/><br/>

***
<br/>

> <h1 id= "蓝牙底层库设计">蓝牙底层库设计</h1>

AKBleUtil
- **功能**
	- 开始扫描设备
		- 扫描次数8次，每次扫描时间为30s，然后停止6秒。接着再扫描，再停止。直到8次为止。
	- 停止设备扫描
	- 查询某个蓝牙设备（Peripheral）支持哪些 Service（服务）。
	- 订阅蓝牙服务
	- 蓝牙设备发送消息
	- 接收蓝牙设备发来的消息


场景一：
	业务层AController页面扫描蓝牙设备，调用蓝牙扫描BLEScannerChannel类，BLEScannerChannel调用AKBleUtil扫描，返回扫描到的设备通过广播报数据发现设备，设备信息中含有MAC，MAC地址类似：AC:CL:D0:G9:J1
；
	
场景二： 通过设备的二维码，扫描到蓝牙设备的mac地址，地址类似：AB:JL:D8:G9:J1
	
紧跟着来到

场景三：
- AController中选择扫描到的蓝牙设备或者二维码扫描到的设备，然后经过选择Wi-Fi配置页面：账号、密码，接着来到蓝牙激活页面。先是通过连接设备方法，通过BLEScannerChannel类传递设备MAC地址，然后BLEScannerChannel类调用AKBleUtil方法查看有没有扫描到的蓝牙设备，若是有则直接连接，若是没有，先扫描，在规定时间内进行扫描，若是超出时间没有扫描到，则停止扫描。若是扫描到设备，则停止扫描再进行连接。
- 连接设备后，在ConnectManager中通过设备Mac地址，调用AKBleUtil中的方法进行查询服务、订阅服务、设备发送消息、设备接受消息。


请问如何设计，要求使用swift编码，使用代理进行扫描、
接受、发送消息给出伪代码，说一说大致设计思路，要求蓝牙设备连接能经受压力连接、接受、发送消息，符合主流大厂设计。




<br/><br/><br/>

***
<br/>

> <h1 id= "数组通过“+”拼接">数组通过“+”拼接</h1>

```swift
 private var devicesByIdentityKey: [String: AKDiscoveredDevice] = [:]
    private var duplicateDevices: [AKDiscoveredDevice] = []

let devices = duplicateDevices + Array(devicesByIdentityKey.values)
```

什么意思？

你这段代码本质上是在 **组合两个不同来源的设备列表**，形成最终的 devices 数组，具体解释如下：

---

# 1. 两个存储结构

```swift
private var devicesByIdentityKey: [String: AKDiscoveredDevice] = [:]
```

* 这是一个 **字典**（Dictionary）。
* key 是 `String`，通常是 **设备的唯一标识符**。
* value 是 `AKDiscoveredDevice`。
* 用途：**快速通过唯一标识查找设备**，保证每个 key 对应的设备唯一，不会重复。

---

```swift
private var duplicateDevices: [AKDiscoveredDevice] = []
```

* 这是一个 **数组**。
* 用于存储 **重复出现的设备** 或某些无法唯一标识的设备。
* 例如，在扫描 BLE 或 LAN 设备时，同一个设备可能被发现多次，如果不覆盖原有字典，就存到这里。

---

# 2. 合并设备列表

```swift
let devices = duplicateDevices + Array(devicesByIdentityKey.values)
```

解释：

* `devicesByIdentityKey.values`：字典中所有设备对象。
* `Array(...)`：把字典的 values 转成数组。
* `duplicateDevices + ...`：用 `+` 运算符把两个数组合并。

所以最终：

```swift
devices = duplicateDevices + Array(devicesByIdentityKey.values)
```

含义：

> “把重复设备和唯一设备合并，形成一个完整设备列表。”

---

# 3. 使用场景举例

假设：

```swift
devicesByIdentityKey = [
    "abc123": deviceA,
    "def456": deviceB
]

duplicateDevices = [deviceC]
```

合并后：

```swift
devices = [deviceC, deviceA, deviceB]
```

* **顺序**：先重复设备，再唯一设备。
* 方便后续对所有设备进行 **排序、过滤、显示**。

---

# 4. 为什么这么设计

* **字典**保证了唯一性（同一个 key 不会重复）。
* **数组 duplicateDevices** 处理一些无法用 key 区分的设备。
* 合并后统一处理，便于 UI 显示或者排序。





<br/><br/><br/>

***
<br/>

> <h1 id= ""></h1>


```swift
private let stateQueue = DispatchQueue(label: "com.argus.akAdder.discovery.manager")

let shouldNotify = stateQueue.sync { () -> Bool in
return true
}
```

这个是什么队列？串行同步队列吗？

是的，你这段代码创建的队列是**串行队列（Serial Queue）**，并且你对它使用了 **同步执行（sync）**。具体解释如下：

```swift
private let stateQueue = DispatchQueue(label: "com.argus.akAdder.discovery.manager")
```

* 这是一个 **串行队列**（默认情况 `DispatchQueue(label:)` 创建的就是串行队列）。
* 所有提交到 `stateQueue` 的任务会**按顺序执行**，同一时间只会有一个任务在执行。

```swift
let shouldNotify = stateQueue.sync { () -> Bool in
    return true
}
```

* `sync` 表示 **同步执行**：当前线程会等待这个闭包执行完成后再继续。
* 因为队列是串行的，这可以确保对共享状态的访问是线程安全的。
* 这种用法常用于保护某个资源或状态，避免多线程并发修改。

⚠️ 注意事项：

* **不要在该队列内部再次调用 `sync` 到同一个队列**，会导致死锁。
* 如果想异步执行而不阻塞当前线程，应该用 `async`。

总结：你这里是**串行同步队列**，通常用于线程安全地读取/修改状态。


***
<br/><br/><br/>
> <h2 id="AKBLEDeviceRepository串行队列死锁保护">AKBLEDeviceRepository 串行队列死锁保护</h2>

问题：

`AKBLEDeviceRepository` 中这些方法都使用了串行队列同步访问字典：

```swift
public func upsert(_ device: AKBlePeripheralModel)
public func find(mac: String) -> AKBlePeripheralModel?
public func find(identifier: String) -> AKBlePeripheralModel?
public func remove(identifier: String)
public func removeAll()
```

里面都有类似代码：

```swift
queue.sync {
    devicesByMAC[mac] = device
}
```

### `queue.sync` 的作用

`AKBLEDeviceRepository` 内部维护的是 `Dictionary`：

```swift
private var devicesByMAC: [String: AKBlePeripheralModel] = [:]
```

Swift 的 `Dictionary` 不是线程安全的。BLE 扫描回调、连接回调、业务读取、释放资源可能来自不同线程，如果同时读写同一个字典，可能出现数据不一致甚至崩溃。

因此使用串行队列保护读写：

```swift
private let queue = DispatchQueue(label: "com.argus.ble.deviceRepository")
```

串行队列保证同一时间只有一个字典操作在执行。

`sync` 表示调用方等待字典操作完成后再继续：

```swift
repository.upsert(device)
let device = repository.find(mac: mac)
```

这样 `upsert` 返回时，设备已经写入；`find` 能读取到已串行化后的最新结果。

### 会不会死锁

普通从外部调用这些方法不会死锁：

```swift
repository.upsert(device)
repository.find(mac: mac)
repository.removeAll()
```

但如果已经处在同一个串行队列内部，又调用这些公共方法，就会死锁。

错误示例：

```swift
queue.sync {
    repository.find(mac: mac)
}
```

原因：

`queue.sync` 正在占用这个串行队列，闭包里再次调用 `find`，`find` 内部又执行一次 `queue.sync`。同一个串行队列必须等当前任务执行完才能执行下一个任务，但当前任务又在等待下一个 `sync` 完成，于是互相等待，形成死锁。

### 本次解决方案

给 repository 的串行队列设置身份标记：

```swift
private let queue = DispatchQueue(label: "com.argus.ble.deviceRepository")
private let queueKey = DispatchSpecificKey<ObjectIdentifier>()

public override init() {
    super.init()
    queue.setSpecific(key: queueKey, value: ObjectIdentifier(self))
}
```

然后所有字典访问不再直接调用 `queue.sync`，而是走统一封装：

```swift
private func syncOnQueue<T>(_ work: () -> T) -> T {
    if DispatchQueue.getSpecific(key: queueKey) == ObjectIdentifier(self) {
        return work()
    }
    return queue.sync(execute: work)
}
```

逻辑：

- 如果当前不在 repository 的队列上，使用 `queue.sync`，保证线程安全。
- 如果当前已经在这个 repository 的队列上，直接执行 `work()`，避免同队列重入 `sync` 死锁。
- 使用 `ObjectIdentifier(self)` 区分不同 repository 实例，避免测试或多实例场景误判。

修改后：

```swift
syncOnQueue {
    devicesByMAC[mac] = device
}
```

替代：

```swift
queue.sync {
    devicesByMAC[mac] = device
}
```

### 为什么不用 `async`

如果 `upsert` 使用 `async`：

```swift
queue.async {
    devicesByMAC[mac] = device
}
```

调用方无法确认写入何时完成。下面代码可能读不到刚写入的设备：

```swift
repository.upsert(device)
let found = repository.find(mac: device.macAddr)
```

所以这里仍然保持同步语义，只是在同步前增加“当前是否已经在队列上”的判断。

### 最终收益

- 保留线程安全。
- 保留 `upsert`、`find`、`removeAll` 的同步可预测语义。
- 避免未来内部方法重入调用导致死锁。
- 不改变外部调用方式。


***
<br/><br/><br/>
> <h2 id="">DispatchWorkItem管理延迟或超时任务</h2>

```swift
    private var timeoutWorkItem: DispatchWorkItem? 
```
这个是什么？如何使用？详细解读和举例

`private var timeoutWorkItem: DispatchWorkItem?` 在 Swift 中表示一个 **可选的 `DispatchWorkItem` 类型变量**，通常用来管理延迟或超时任务。下面我详细解读它的作用、用法和举例。

---

## 1. 什么是 `DispatchWorkItem`

* `DispatchWorkItem` 是 **封装一段可执行代码的对象**，可以提交给 `DispatchQueue` 执行。
* 与直接调用 `queue.async { ... }` 不同，你可以：

  * 随时 **取消** 任务
  * **延迟执行**
  * **在不同队列之间传递**任务
* 常用场景：超时处理、延迟操作、定时任务。

---

## 2. 你的声明

```swift
private var timeoutWorkItem: DispatchWorkItem?
```

* `private`：只能在当前类/结构体中访问。
* `var`：可以在运行时赋值或重新创建。
* `DispatchWorkItem?`：可选，意味着可能有值，也可能为 `nil`（例如任务未创建或已取消）。

用途：通常用于记录一个超时任务，这样你可以在需要的时候 **取消或替换**。

---

## 3. 如何使用

### 示例 1：简单延迟执行

```swift
timeoutWorkItem = DispatchWorkItem {
    print("超时任务执行")
}

DispatchQueue.main.asyncAfter(deadline: .now() + 5, execute: timeoutWorkItem!)
```

解释：

* 创建一个 `DispatchWorkItem`，封装要执行的代码。
* 使用 `asyncAfter` 延迟 5 秒执行。
* 你可以保留 `timeoutWorkItem` 引用，以便后续取消。

---

### 示例 2：取消超时任务

```swift
// 用户操作完成，取消超时任务
timeoutWorkItem?.cancel()
timeoutWorkItem = nil
```

说明：

* 调用 `cancel()` 后，任务会 **标记为已取消**，即使延迟到时间到了，闭包也不会执行。
* 清空引用，防止重复使用或内存泄漏。

---

### 示例 3：结合网络请求的超时处理

```swift
func startNetworkRequest() {
    // 创建超时任务
    timeoutWorkItem = DispatchWorkItem {
        print("请求超时，执行超时处理")
        // 可以更新 UI 或重置状态
    }
    
    // 5 秒后执行超时任务
    DispatchQueue.main.asyncAfter(deadline: .now() + 5, execute: timeoutWorkItem!)
    
    // 假设发起网络请求
    performNetworkCall { success in
        if success {
            // 网络请求成功，取消超时任务
            timeoutWorkItem?.cancel()
            timeoutWorkItem = nil
            print("请求成功")
        }
    }
}
```

解释：

* 如果网络请求在 5 秒内完成，取消 `timeoutWorkItem`，超时逻辑不会触发。
* 如果网络请求超过 5 秒未完成，超时任务会执行。

---

✅ **总结**

`timeoutWorkItem` 是一个**可控、可取消的延迟任务**。适合：

* 延迟执行某些操作
* 超时处理
* 避免重复任务或内存泄漏


***
<br/><br/><br/>
> <h2 id="swift位移枚举使用">swift位移枚举使用</h2>

```
@objc public enum AKDiscoveryType: Int {
  
    case none = 0
    case ble = 1
    case lan = 2
}

public func containsDiscoveryType(_ type: AKDiscoveryType) -> Bool {
        (typeMask & type.rawValue) != 0
    }
}

有点不懂解释下
```

你这里本质上是在用 **位掩码（BitMask）** 来表示“支持哪些发现方式”，这是很多底层框架、蓝牙、权限、网络库里非常常见的设计。

你现在有点迷惑，主要是因为：

```swift
case ble = 1
case lan = 2
```

它不是普通枚举用途，而是当成“二进制位”使用。

我一步一步拆开。

---

# 一、先看你的枚举

```swift
@objc public enum AKDiscoveryType: Int {

    case none = 0
    case ble = 1
    case lan = 2
}
```

等价于：

```swift
none = 0   // 0000
ble  = 1   // 0001
lan  = 2   // 0010
```

注意：

这里的值不是随便写的。

而是：

| 类型  | 二进制  |
| --- | ---- |
| ble | 0001 |
| lan | 0010 |

因为后面要做：

# 位运算（Bit Operation）

---

# 二、核心方法

```swift
public func containsDiscoveryType(_ type: AKDiscoveryType) -> Bool {
    (typeMask & type.rawValue) != 0
}
```

这里最关键：

```swift
&
```

这是：

# 按位与运算

---

# 三、typeMask 是什么

你虽然没贴：

```swift
private var typeMask: Int
```

但它大概率是：

```swift
private var typeMask: Int = 0
```

它用来：

# 保存当前支持哪些发现方式

比如：

---

## 情况1：只支持 BLE

```swift
typeMask = 1
```

二进制：

```text
0001
```

---

## 情况2：只支持 LAN

```swift
typeMask = 2
```

二进制：

```text
0010
```

---

## 情况3：同时支持 BLE + LAN

```swift
typeMask = 1 | 2
```

即：

```text
0001
0010
----
0011
```

所以：

```swift
typeMask = 3
```

这里：

```swift
|
```

是：

# 按位或运算

用于“组合能力”。

---

# 四、containsDiscoveryType 如何判断

---

# 示例1：当前支持 BLE + LAN

```swift
typeMask = 3
```

即：

```text
0011
```

---

现在判断：

```swift
containsDiscoveryType(.ble)
```

即：

```swift
(typeMask & 1) != 0
```

变成：

```text
0011
0001
----
0001
```

结果：

```text
0001 != 0
```

所以：

```swift
true
```

说明：

✅ 支持 BLE

---

# 示例2：判断是否支持 LAN

```swift
(typeMask & 2) != 0
```

即：

```text
0011
0010
----
0010
```

结果不为0。

所以：

```swift
true
```

说明：

✅ 支持 LAN

---

# 示例3：只支持 BLE 时

```swift
typeMask = 1
```

即：

```text
0001
```

判断：

```swift
containsDiscoveryType(.lan)
```

变成：

```text
0001
0010
----
0000
```

结果：

```swift
0
```

所以：

```swift
false
```

说明：

❌ 不支持 LAN

---

# 五、为什么不用数组

很多人会想：

为啥不用：

```swift
[.ble, .lan]
```

因为：

# 位掩码性能更高

尤其：

* C
* Objective-C
* iOS底层
* 蓝牙协议
* socket
* Linux
* 驱动
* 游戏引擎

非常常见。

它优点：

| 方式      | 内存 | 性能 |
| ------- | -- | -- |
| 数组      | 较大 | 较慢 |
| BitMask | 极小 | 极快 |

---

# 六、这是 Apple 官方大量使用的设计

比如：

## UIViewAutoresizing

```swift
flexibleWidth
flexibleHeight
```

本质就是位掩码。

---

## UIView.AnimationOptions

```swift
.curveEaseIn
.repeat
.autoreverse
```

也是位掩码。

---

# 七、你的代码更规范的写法

Swift 里更推荐：

```swift
struct AKDiscoveryType: OptionSet {

    let rawValue: Int

    static let ble = AKDiscoveryType(rawValue: 1 << 0)
    static let lan = AKDiscoveryType(rawValue: 1 << 1)
}
```

使用：

```swift
let types: AKDiscoveryType = [.ble, .lan]

types.contains(.ble)
```

这是：

# Swift 官方位掩码方案

比 enum 更标准。

---

# 八、你现在这个 contains 本质

你的代码：

```swift
(typeMask & type.rawValue) != 0
```

翻译成人话：

> “当前能力集合里，是否包含这个能力”

即：

* 是否支持 BLE
* 是否支持 LAN
* 是否支持某种功能

---

# 九、一个完整例子

```swift
@objc public enum AKDiscoveryType: Int {

    case none = 0
    case ble = 1   // 0001
    case lan = 2   // 0010
}

class Manager {

    private var typeMask: Int = 0

    func addType(_ type: AKDiscoveryType) {
        typeMask |= type.rawValue
    }

    func containsDiscoveryType(_ type: AKDiscoveryType) -> Bool {
        (typeMask & type.rawValue) != 0
    }
}
```

使用：

```swift
let manager = Manager()

manager.addType(.ble)
manager.addType(.lan)

print(manager.containsDiscoveryType(.ble)) // true
print(manager.containsDiscoveryType(.lan)) // true
```

此时：

```text
0001 | 0010 = 0011
```

---

# 十、你真正需要记住的核心

位掩码核心只有一句：

| 运算  | 作用     |      |
| --- | ------ | ---- |
| `   | `      | 添加能力 |
| `&` | 判断是否包含 |      |
| `^` | 切换     |      |
| `~` | 取反     |      |

其中最常用：

```swift
mask |= value
```

添加。

```swift
(mask & value) != 0
```

判断是否存在。


<br/>

## 如果还有更多枚举，并且你继续想用：

# BitMask（位掩码）

那每个枚举值必须是：

# 2 的幂

即：

```text
1
2
4
8
16
32
64
128
...
```

因为：

它们的二进制只有一位是 1，才不会互相冲突。

---

# 一、正确写法

假设你有 8 种发现方式：

```swift
@objc public enum AKDiscoveryType: Int {

    case none       = 0

    case ble        = 1       // 00000001
    case lan        = 1 << 1  // 00000010 = 2
    case wifi       = 1 << 2  // 00000100 = 4
    case cloud      = 1 << 3  // 00001000 = 8
    case zigbee     = 1 << 4  // 00010000 = 16
    case matter     = 1 << 5  // 00100000 = 32
    case thread     = 1 << 6  // 01000000 = 64
    case ethernet   = 1 << 7  // 10000000 = 128
}
```

---

# 二、为什么推荐 `1 << n`

因为：

```swift
1 << 0 = 1
1 << 1 = 2
1 << 2 = 4
1 << 3 = 8
```

这是：

# 左移运算

即：

```text
00000001 << 1
=
00000010
```

这样：

* 不容易写错
* 可读性强
* 一眼看出是 BitMask

这是行业标准写法。

---

# 三、千万不要这样写

错误：

```swift
case ble = 1
case lan = 2
case wifi = 3
case cloud = 4
```

因为：

```text
3 = 0011
```

已经包含：

```text
0001
0010
```

会导致：

```swift
(mask & 3) != 0
```

判断错乱。

---

# 四、位掩码核心原则

每个值只能占：

# 一个二进制位

即：

| 值  | 二进制      |
| -- | -------- |
| 1  | 00000001 |
| 2  | 00000010 |
| 4  | 00000100 |
| 8  | 00001000 |
| 16 | 00010000 |

这样才能：

* 自由组合
* 不冲突
* 正确判断

---

# 五、组合效果

比如：

```swift
var mask = 0

mask |= AKDiscoveryType.ble.rawValue
mask |= AKDiscoveryType.lan.rawValue
mask |= AKDiscoveryType.wifi.rawValue
```

结果：

```text
00000001
00000010
00000100
---------
00000111
```

即：

```swift
mask = 7
```

---

# 六、判断

判断：

```swift
(mask & AKDiscoveryType.wifi.rawValue) != 0
```

即：

```text
00000111
00000100
---------
00000100
```

不为0：

```swift
true
```

说明支持 wifi。

---

# 七、更推荐的 Swift 官方写法

实际上：

# 位掩码不推荐用 enum

而推荐：

# OptionSet

这是 Apple 官方标准方案。

---

# 八、标准 OptionSet 写法

```swift
struct AKDiscoveryType: OptionSet {

    let rawValue: Int

    static let ble      = AKDiscoveryType(rawValue: 1 << 0)
    static let lan      = AKDiscoveryType(rawValue: 1 << 1)
    static let wifi     = AKDiscoveryType(rawValue: 1 << 2)
    static let cloud    = AKDiscoveryType(rawValue: 1 << 3)
    static let zigbee   = AKDiscoveryType(rawValue: 1 << 4)
}
```

使用：

```swift
let types: AKDiscoveryType = [.ble, .lan, .wifi]

types.contains(.ble)
```

这是 Swift 最规范的方式。

---

# 九、为什么 Apple 发明 OptionSet

因为 enum 本质：

# 只能表示“一个值”

例如：

```swift
direction = .left
```

但位掩码需要：

# 同时拥有多个值

例如：

```swift
[.ble, .lan, .wifi]
```

所以：

Apple 专门设计了：

# OptionSet

---

# 十、你这个场景非常适合 OptionSet

比如：

设备支持：

* BLE
* LAN
* Matter
* Thread

这种：

# “多个能力组合”

就是 OptionSet 最经典场景。

---

# 十一、OC 里对应什么

Objective-C 里：

```objc
typedef NS_OPTIONS(NSUInteger, AKDiscoveryType) {
    AKDiscoveryTypeBLE      = 1 << 0,
    AKDiscoveryTypeLAN      = 1 << 1,
    AKDiscoveryTypeWiFi     = 1 << 2,
};
```

Swift 的 OptionSet 本质就是它。


***
<br/><br/><br/>
> <h2 id="对象身份标识符">对象身份标识符</h2>

```swift
 private var runningChannelIds: Set<ObjectIdentifier> = []
```
 中的 ObjectIdentifier 是什么？为什么要这么用？看不懂 ObjectIdentifier 没有用过，请举例相接
 
 
 
 <br/><br/><br/>

***
<br/>

> <h1 id= "字符串中提取数字">字符串中提取数字</h1>


这段 Swift 代码：

```swift
text?.filter { $0.isNumber } ?? ""
```

本质是在做一件事：**从字符串中提取所有数字字符，如果 text 为 nil 就返回空字符串**。

---

## 1. 整体结构拆解

```swift
text?                 // 可选链：text 是 Optional<String>
    .filter { ... }   // 过滤字符
?? ""                 // nil 合并
```

等价逻辑：

```swift
if let text = text {
    return text.filter { $0.isNumber }
} else {
    return ""
}
```

---

## 2. filter 做了什么？

```swift
.filter { $0.isNumber }
```

### 含义：

对字符串中的每一个字符进行判断，只保留“数字字符”。

---

### `$0` 是什么？

`$0` = 当前遍历的字符（Character）

例如：

```swift
text = "a1b2c3"
```

遍历过程：

| 字符 | isNumber | 是否保留 |
| -- | -------- | ---- |
| a  | false    | ❌    |
| 1  | true     | ✅    |
| b  | false    | ❌    |
| 2  | true     | ✅    |
| c  | false    | ❌    |
| 3  | true     | ✅    |

结果：

```swift
"123"
```

---

### `isNumber` 是什么？

```swift
Character.isNumber
```

表示：

* 0~9
* 以及一些 Unicode 数字（比如阿拉伯数字等）

---

## 3. `text?` 可选链是什么意思？

```swift
text?.filter(...)
```

如果：

### text 有值

```swift
text = "a1b2"
```

执行 filter

---

### text 是 nil

```swift
text = nil
```

整个表达式直接返回 nil，不会 crash。

---

## 4. `?? ""` 的作用

```swift
?? ""
```

叫 **nil 合并运算符**

意思是：

> 如果前面结果是 nil，就返回 ""

---

所以整体逻辑：

| text 状态 | 输出   |
| ------- | ---- |
| "a1b2"  | "12" |
| nil     | ""   |

---

## 5. 实际用途

这行代码常用于：

### 1. 提取手机号 / 验证码 / 数字输入

```swift
let input = "验证码：1234"
let code = input?.filter { $0.isNumber } ?? ""
// "1234"
```

---

### 2. 去除格式字符

```swift
let phone = "138-0013-8000"
let pure = phone.filter { $0.isNumber }
// "13800138000"
```

---

## 6. 等价写法（更清晰版）

如果你想更可读：

```swift
let result: String

if let text = text {
    result = String(text.filter { $0.isNumber })
} else {
    result = ""
}
```

---

## 7. 更进阶写法（推荐）

如果你明确知道只需要 ASCII 数字：

```swift
text?.filter { $0 >= "0" && $0 <= "9" } ?? ""
```

优点：

* 更严格
* 不受 Unicode 影响
* 性能略优

---

## 总结一句话

```swift
text?.filter { $0.isNumber } ?? ""
```

就是：

> “从字符串中提取所有数字，如果字符串为空就返回空字符串”。

