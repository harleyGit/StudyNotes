> <h1 id= ""></h1>
- [**Native桥接RN**](#Native桥接RN) 
	- [Swift兼容RN代码模版](#Swift兼容RN代码模版) 
	- [RN中的回调原生iOS中使用](#RN中的回调原生iOS中使用) 
	- [RN闭包Block类型包括RCTResponseSenderBlock、RCTPromiseResolveBlock、RCTPromiseRejectBlock区别比较](#RN闭包Block类型包括RCTResponseSenderBlock、RCTPromiseResolveBlock、RCTPromiseRejectBlock区别比较)

<br/><br/><br/>

***
<br/>

> <h1 id= "Native桥接RN">Native桥接RN</h1>

***
<br/><br/><br/>
> <h2 id="RCTEventEmitter作用"> RCTEventEmitter作用</h2>

在 **iOS 使用 Swift 桥接 React Native（RN）** 的过程中，`RCTEventEmitter` 是一个非常重要的类，它用于从 **原生 iOS 模块向 JavaScript 发送事件**。这通常用于通知 JS 某些异步的原生事件，例如蓝牙连接、推送通知、网络变化等。

<br/>

- **✅ `RCTEventEmitter` 是什么？**

`RCTEventEmitter` 是一个来自 `React` 框架的基类，继承自 `RCTBridgeModule`。它专门用于从 Native 端主动向 RN 的 JS 端发送事件（Event）。

<br/>

- **✅ 使用流程（Swift 示例）**

	- **步骤 1：继承 `RCTEventEmitter`**

```swift
import Foundation
import React

@objc(BLEEventEmitter)
class BLEEventEmitter: RCTEventEmitter {

    static var shared: BLEEventEmitter?

    override init() {
        super.init()
        BLEEventEmitter.shared = self
    }

    // MARK: - 事件支持列表
    // 告诉 React Native：原生模块将会向 JS 发送哪些事件（也就是 JS 端可以监听的事件名）
    // 只能向 JS 发送这里列出的事件，否则 sendEvent 不生效，也不会报错
    override func supportedEvents() -> [String] {
        return ["onBleStatusChange"]
    }

    // MARK: - 是否在主线程发送
    // 告诉 React Native：模块初始化是否必须在主线程执行
    // 默认是 false, 模块初始化是在 后台线程;
    // 如果你的 init() 中访问了 UIKit 或其它主线程依赖的内容，就必须返回 true
    override static func requiresMainQueueSetup() -> Bool {
        return true
    }

    // MARK: - 发送事件
    func sendStatusEvent(status: String) {
        sendEvent(withName: "onBleStatusChange", body: ["status": status])
    }
}
```

<br/>

- **步骤 2：在 Swift 的其他模块中发送事件**

```swift
BLEEventEmitter.shared?.sendStatusEvent(status: "connected")
```

<br/>

- **步骤 3：在 JS 端接收事件**

```js
import { NativeEventEmitter, NativeModules } from 'react-native';

const { BLEEventEmitter } = NativeModules;
const emitter = new NativeEventEmitter(BLEEventEmitter);

const subscription = emitter.addListener('onBleStatusChange', (event) => {
  console.log('BLE status changed:', event.status);
});

useEffect(() => {
	return () => subscription.remove();
}, []);
```

---
<br/>

- **🧠 总结**

| 功能                         | 用途                             |
| -------------------------- | ------------------------------ |
| `RCTEventEmitter`          | 从 iOS 原生发事件到 React Native JS 层 |
| `sendEvent(...)`           | 发送一个事件                         |
| `supportedEvents()`        | 告诉 RN 哪些事件是合法的                 |
| JS 中的 `NativeEventEmitter` | 订阅 iOS 发来的事件                   |


***
<br/><br/><br/>
> <h2 id="Swift兼容RN代码模版">Swift兼容RN代码模版</h2>

**🔍 我在 `.m` 文件中看到的：**

```objc
@interface RCT_EXTERN_MODULE (BLEEventEmitter, RCTEventEmitter<RCTBridgeModule>)
RCT_EXTERN_METHOD(connect)
RCT_EXTERN_METHOD(disconnect)
RCT_EXTERN_METHOD(getChannelState:
                  (nonnull RCTResponseSenderBlock)callback)
@end
```

<br/>

**✅ 它的作用：让 Swift 写的模块能被 React Native 识别（桥接暴露接口）**

因为 React Native 的桥接系统是基于 Objective-C runtime 的，**它无法自动识别纯 Swift 方法**，所以需要通过这种方式手动“桥接”。

---
<br/>

- **🧠 逐条解释：**

	- **1. `RCT_EXTERN_MODULE`**

告诉 React Native：
“我有一个原生模块，名字叫 `BLEEventEmitter`，它是用 Swift 写的，继承自 `RCTEventEmitter`。”

等价于 OC 中手动声明模块：

```objc
@interface BLEEventEmitter : RCTEventEmitter <RCTBridgeModule>
@end
```

<br/>

- **2. `RCT_EXTERN_METHOD(...)`**

每个方法你想暴露给 JS，都要在这里声明。否则，JS 无法调用 Swift 中的对应方法,例子：

```swift
@objc public func connect() {
    // 连接蓝牙
}
```

<br/>

**OC 桥接中写：**

```objc
RCT_EXTERN_METHOD(connect)
```

<br/>

- **3.有回调参数的写法**如：

```swift
@objc public func getChannelState(_ callback: @escaping RCTResponseSenderBlock) {
    callback(["connected"])
}
```

桥接声明方式：

```objc
RCT_EXTERN_METHOD(getChannelState:(nonnull RCTResponseSenderBlock)callback)
```

---

**✅ 为什么写在 `.m` 文件中？**

虽然模块是 Swift 写的，**但 React Native 桥接必须通过 Objective-C 头部来声明它们**，所以这部分必须放在 `.m` 文件里，否则桥接不会生效。

---

- **📝 总结**

| 项目                  | 作用                                        |
| ------------------- | ----------------------------------------- |
| `RCT_EXTERN_MODULE` | 声明 Swift 模块供 JS 调用                        |
| `RCT_EXTERN_METHOD` | 显式暴露方法给 React Native                      |
| `.m 文件中写这些桥接声明`     | 因为桥接系统依赖 Objective-C runtime，必须在 OC 作用域声明 |

---

如果你不想自己写这些 `.m` 文件，可以用 [SwiftBridgeGenerator](https://github.com/mrousavy/react-native-swift-bridge-generator) 这类工具自动生成。



***
<br/><br/><br/>
> <h2 id="RN中的回调原生iOS中使用">RN中的回调原生iOS中使用</h2>

	要在 React Native 的 iOS 端用 Swift 实现一个 `checkBluetoothIsEnabled(promise: Promise)` 的方法，以下是完整写法、桥接声明和 JS 端使用方式的详解。

---

- **🧩 一、JS端定义**

```js
NativeModules.BluetoothModule.checkBluetoothIsEnabled().then(isEnabled => {
  console.log('蓝牙是否开启:', isEnabled);
});
```

<br/>

- **🧱 二、Swift 实现（使用 `Promise` 的桥接）**

	- **✅ 1.Swift 模块：BluetoothModule.swift**

```swift
import Foundation
import CoreBluetooth
import React

@objc(BluetoothModule)
class BluetoothModule: NSObject {

    private var centralManager: CBCentralManager!

    override init() {
        super.init()
        centralManager = CBCentralManager(delegate: nil, queue: nil, options: nil)
    }

    @objc
    func checkBluetoothIsEnabled(_ resolve: @escaping RCTPromiseResolveBlock,
                                 rejecter reject: @escaping RCTPromiseRejectBlock) {
        let state = centralManager.state
        if state == .poweredOn {
            resolve(true)
        } else {
            resolve(false)
        }
    }
}
```

---
<br/>

- **🧷 三、桥接声明（必须写在 `.m` 文件中）**

	- **✅ 2. BluetoothModuleBridge.m**

```objc
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(BluetoothModule, NSObject)

RCT_EXTERN_METHOD(checkBluetoothIsEnabled:
                  (RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end
```

> ⚠️ 注意：这段代码必须写在 `.m` 文件中，不能是 `.swift`。

---

- **🧩 四、JS 使用方式**

```js
import { NativeModules } from 'react-native';

const { BluetoothModule } = NativeModules;

BluetoothModule.checkBluetoothIsEnabled()
  .then((isOn) => {
    console.log('蓝牙状态:', isOn);
  })
  .catch((err) => {
    console.error('检查蓝牙失败:', err);
  });
```

---

- **📌 总结**

| 部分       | 内容                                                                  |
| -------- | ------------------------------------------------------------------- |
| Swift 类名 | `BluetoothModule`（需继承自 `NSObject`）                                  |
| 方法名      | `checkBluetoothIsEnabled(resolve:rejecter:)`                        |
| 桥接方式     | 在 `.m` 中用 `RCT_EXTERN_METHOD(...)` 声明                               |
| JS 调用    | `NativeModules.BluetoothModule.checkBluetoothIsEnabled().then(...)` |


---

- **❓1. `rejecter reject: @escaping RCTPromiseRejectBlock` 这个参数没用到？**

是的，在下面这段代码中你**确实没有用到 `reject` 参数**，所以你可以：

* 保留它（将来可能用来处理错误）✅
* 如果你确定永远不会失败，也可以删除（但为了规范，**推荐保留**）

```swift
@objc
func checkBluetoothIsEnabled(_ resolve: @escaping RCTPromiseResolveBlock,
                             rejecter reject: @escaping RCTPromiseRejectBlock) {
    // 你没有用 reject，而是直接调用 resolve
    resolve(true)
}
```

---
<br/>

- **❓2. RN 中的 `(promise: Promise)` 对应 Swift 中的 `RCTPromiseResolveBlock` 吗？**

✅ **没错，就是这样：**

* 在 JS 中你写的：

```js
NativeModules.BluetoothModule.checkBluetoothIsEnabled().then(...) // Promise
```

* 在 Swift 中就要实现为：

```swift
func someFunc(_ resolve: @escaping RCTPromiseResolveBlock,
              rejecter reject: @escaping RCTPromiseRejectBlock)
```

这就是 JS 的 `Promise` 与 Swift 的 `resolve/reject` 映射方式。

---
<br/>

- **❓3. `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 有什么区别？**

| 类型                       | 用途           | 参数                                     | 示例                                                  |
| ------------------------ | ------------ | -------------------------------------- | --------------------------------------------------- |
| `RCTPromiseResolveBlock` | ✅ 表示异步**成功** | 任何返回给 JS 的值                            | `resolve(true)`                                     |
| `RCTPromiseRejectBlock`  | ❌ 表示异步**失败** | `(String, String, Error?)`：错误码、信息、原始错误 | `reject("ERR_BLE", "Bluetooth not available", nil)` |

---

- **✅ 使用示例（含错误处理）：**

```swift
@objc
func checkBluetoothIsEnabled(_ resolve: @escaping RCTPromiseResolveBlock,
                             rejecter reject: @escaping RCTPromiseRejectBlock) {
    let state = centralManager.state
    switch state {
    case .poweredOn:
        resolve(true)
    case .unsupported:
        reject("ERR_UNSUPPORTED", "该设备不支持蓝牙", nil)
    case .unauthorized:
        reject("ERR_NO_PERMISSION", "未授权蓝牙使用权限", nil)
    default:
        resolve(false)
    }
}
```



***
<br/><br/><br/>
> <h2 id="RN闭包Block类型包括RCTResponseSenderBlock、RCTPromiseResolveBlock、RCTPromiseRejectBlock区别比较">RN闭包Block类型包括RCTResponseSenderBlock、RCTPromiseResolveBlock、RCTPromiseRejectBlock区别比较</h2>

在 React Native 的 iOS 原生桥接中，常见的 Block 类型包括 `RCTResponseSenderBlock`、`RCTPromiseResolveBlock`、`RCTPromiseRejectBlock` 等。它们虽然都是回调，但用途和 JS 对应的写法不同。

---

- **🧩 一、主要 Block 类型对比**

| Block 名称                     | 对应 JS             | 用途                 | 使用方式                           |
| ---------------------------- | ----------------- | ------------------ | ------------------------------ |
| **`RCTResponseSenderBlock`** | callback          | ✅ 回调风格函数（callback） | `callback([value])`            |
| **`RCTPromiseResolveBlock`** | `Promise.resolve` | ✅ Promise 成功回调     | `resolve(value)`               |
| **`RCTPromiseRejectBlock`**  | `Promise.reject`  | ✅ Promise 失败回调     | `reject(code, message, error)` |

---

- **🧪 二、使用场景与示例**

	-  **✅ 1. `RCTResponseSenderBlock`：传统 Callback 风格（JS 回调函数）**

	- **🔧 Swift 或 ObjC 端（定义）：**

```swift
@objc
func getDeviceName(_ callback: @escaping RCTResponseSenderBlock) {
    let name = UIDevice.current.name
    callback([name])
}
```

<br/>

- **🧠 JS 使用方式：**

```js
NativeModules.MyModule.getDeviceName((name) => {
  console.log("设备名:", name);
});
```

> ⚠️ callback 总是接收一个数组（即使是一个值也得写 `[value]`）。

---
<br/>

- **✅ 2. `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock`：Promise 风格**

```swift
@objc
func checkStatus(_ resolve: @escaping RCTPromiseResolveBlock,
                 rejecter reject: @escaping RCTPromiseRejectBlock) {
    let success = true
    if success {
        resolve("OK")
    } else {
        reject("ERR_CODE", "失败原因", nil)
    }
}
```

<br/>

- **🧠 JS 使用方式：**

```js
NativeModules.MyModule.checkStatus()
  .then(result => console.log("成功:", result))
  .catch(err => console.warn("失败:", err));
```

---

- **🧾 三、还有哪些常用的 Block？**

| Block 类型                                        | 用途              | 是否异步                         | 说明     |
| ----------------------------------------------- | --------------- | ---------------------------- | ------ |
| `RCTResponseSenderBlock`                        | JS 的 callback   | ✅ 通常用于 callback 回调函数         |        |
| `RCTPromiseResolveBlock`                        | JS 的 Promise 成功 | ✅ 用于异步成功响应                   |        |
| `RCTPromiseRejectBlock`                         | JS 的 Promise 失败 | ✅ 用于异步失败响应                   |        |
| `RCTResponseErrorBlock` *(已废弃)*                 | 错误回调            | ❌ 旧式做法，建议不用                  |        |
| `RCTBubblingEventBlock` / `RCTDirectEventBlock` | 事件发送到 JS        | ✅ 配合 `RCTEventEmitter` 用于发事件 |        |
| `RCTBridgeModule`                               | 桥接模块协议          | ⛔ 不是 Block，是协议               | 定义模块接口 |

