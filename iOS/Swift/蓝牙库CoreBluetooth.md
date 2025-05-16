
> <h1 id=""></h1>
- [**CBCentralManager类**](#CBCentralManager类)
- [**连接成功的蓝牙外围设备-Peripheral**](#连接成功的蓝牙外围设备-Peripheral)



<br/><br/><br/>

***
<br/>

> <h1 id=""></h1>


<br/><br/><br/>

***
<br/>

> <h1 id="CBCentralManager类">CBCentralManager类</h1>

`CBCentralManager` 是 **CoreBluetooth 框架** 中的一个核心类，用于在 iOS 中作为 **蓝牙中心设备（Central）** 管理和操作其他 **蓝牙外设（Peripheral）**。

<br/>

**🧠 什么是 CBCentralManager？**

* `CBCentralManager` 代表一个蓝牙中心（iPhone 就是 Central），它可以：
  * 扫描附近的蓝牙外设（Peripheral）
  * 连接蓝牙外设
  * 发现服务、特征
  * 读取/写入特征值
  * 接收通知等

CoreBluetooth 的作用类似于 iOS 中蓝牙通信的 API 层，而 `CBCentralManager` 是操作的起点。

<br/><br/>

**🛠️ 它有什么用？**

常见的用途包括：

* 连接智能手环、蓝牙体温计、蓝牙耳机等外设
* 从外设读取数据（如心率、温度）
* 向外设发送指令（如设置参数）

<br/><br/>

**✅ 如何使用？**

- 一个典型的流程如下：
	- 创建 `CBCentralManager`
	- 扫描蓝牙外设
	- 连接外设
	- 发现服务和特征
	- 与特征交互（读/写/订阅）

<br/><br/>

**📦 示例代码（Swift 版）**

```swift
import CoreBluetooth

class BluetoothManager: NSObject, CBCentralManagerDelegate, CBPeripheralDelegate {
    var centralManager: CBCentralManager!
    var discoveredPeripheral: CBPeripheral?

    override init() {
        super.init()
        centralManager = CBCentralManager(delegate: self, queue: nil)
    }

    // 1. 蓝牙状态变化
    func centralManagerDidUpdateState(_ central: CBCentralManager) {
        if central.state == .poweredOn {
            print("蓝牙开启，开始扫描")
            centralManager.scanForPeripherals(withServices: nil, options: nil)
        } else {
            print("蓝牙不可用")
        }
    }

    // 2. 扫描到设备
    // peripheral: 蓝牙设备标识; RSSI: 信号强度; advertisementData: 广播数据(iOS无法扫描出蓝牙Mac,要把Mac放到广播里面)
    func centralManager(_ central: CBCentralManager,
                        didDiscover peripheral: CBPeripheral,
                        advertisementData: [String : Any],
                        rssi RSSI: NSNumber) {
        print("发现设备: \(peripheral.name ?? "未知设备")")
        discoveredPeripheral = peripheral
        centralManager.stopScan()
        centralManager.connect(peripheral, options: nil)
    }

    // 3. 连接成功
    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
        print("已连接 \(peripheral.name ?? "设备")")
        peripheral.delegate = self
        peripheral.discoverServices(nil)
    }

    // 4. 发现服务
    func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
        guard let services = peripheral.services else { return }
        for service in services {
            print("发现服务: \(service.uuid)")
            peripheral.discoverCharacteristics(nil, for: service)
        }
    }

    // 5. 发现特征
    func peripheral(_ peripheral: CBPeripheral,
                    didDiscoverCharacteristicsFor service: CBService,
                    error: Error?) {
        guard let characteristics = service.characteristics else { return }
        for characteristic in characteristics {
            print("发现特征: \(characteristic.uuid)")
            // 读取特征值
            peripheral.readValue(for: characteristic)
        }
    }

    // 6. 接收特征值
    func peripheral(_ peripheral: CBPeripheral,
                    didUpdateValueFor characteristic: CBCharacteristic,
                    error: Error?) {
        if let data = characteristic.value {
            print("接收到数据: \(data)")
        }
    }
}
```

<br/><br/>

**⚠️ 注意事项**

* 使用蓝牙必须在 `Info.plist` 中添加隐私描述：

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>需要蓝牙权限以连接设备</string>
```

* 真机调试才有效，模拟器不支持 CoreBluetooth。
* 蓝牙外围设备必须在附近并且可被发现。


<br/><br/><br/>

***
<br/>

> <h1 id="连接成功的蓝牙外围设备-Peripheral">连接成功的蓝牙外围设备-Peripheral</h1>

**📌 一、CBPeripheral 是什么？**

**✅ 定义：**

- `CBPeripheral` 表示**一个连接成功的蓝牙外围设备（Peripheral）**，你可以通过它来：
	* 发现服务（Service）
	* 发现特征（Characteristic）
	* 读取/写入特征值
	* 订阅通知等

在你用 `CBCentralManager` 扫描并连接上一个蓝牙设备后，会得到一个 `CBPeripheral` 实例。这个实例就是你用来和设备通信的“句柄”。

<br/><br/>

**🔧 CBPeripheral 的常用方法：**

| 方法                                | 作用             |
| --------------------------------- | -------------- |
| `discoverServices(_:)`            | 搜索设备中的服务       |
| `discoverCharacteristics(_:for:)` | 搜索某个服务中的特征     |
| `readValue(for:)`                 | 读取某个特征的值       |
| `writeValue(_:for:type:)`         | 写入某个特征的值       |
| `setNotifyValue(_:for:)`          | 订阅/取消订阅某个特征的通知 |

<br/>


**📌 二、CBPeripheralDelegate 是什么？**

这是 `CBPeripheral` 的**代理协议**，用于接收设备返回的数据和操作结果，所有“回调”都在这个代理中触发。

**✅ 常见代理方法：**

| 方法                                             | 触发时机              |
| ---------------------------------------------- | ----------------- |
| `didDiscoverServices`                          | 发现设备的服务           |
| `didDiscoverCharacteristicsFor service`        | 发现某个服务的所有特征       |
| `didUpdateValueFor characteristic`             | 接收到设备发来的数据（读值或通知） |
| `didWriteValueFor characteristic`              | 写数据完成后回调          |
| `didUpdateNotificationStateFor characteristic` | 通知订阅状态变化          |

<br/><br/>

**🎯 三、完整使用流程图解**

```mermaid
graph LR
A[CBCentralManager 初始化] --> B{蓝牙开启？}
B -- 是 --> C[开始扫描设备]
C --> D[发现 Peripheral]
D --> E[连接 Peripheral]
E --> F[保存 CBPeripheral 实例]
F --> G[设置 CBPeripheral.delegate]
G --> H[discoverServices()]
H --> I[didDiscoverServices 回调]
I --> J[discoverCharacteristics()]
J --> K[didDiscoverCharacteristics 回调]
K --> L[read/write/subscribe 特征]
```

<br/><br/>

**📦 四、完整 Swift 示例（配套解释）**

```swift
import CoreBluetooth

class BLEManager: NSObject, CBCentralManagerDelegate, CBPeripheralDelegate {
    var centralManager: CBCentralManager!
    var peripheral: CBPeripheral?

    override init() {
        super.init()
        // 初始化中心设备
        centralManager = CBCentralManager(delegate: self, queue: nil)
    }

    // MARK: - 1. 蓝牙状态监听
    func centralManagerDidUpdateState(_ central: CBCentralManager) {
        if central.state == .poweredOn {
            print("蓝牙已开启，开始扫描设备")
            centralManager.scanForPeripherals(withServices: nil, options: nil)
        }
    }

    // MARK: - 2. 扫描到设备
    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral,
                        advertisementData: [String : Any], rssi RSSI: NSNumber) {
        print("发现设备：\(peripheral.name ?? "未知")")
        self.peripheral = peripheral
        self.peripheral?.delegate = self
        centralManager.stopScan()
        centralManager.connect(peripheral, options: nil)
    }

    // MARK: - 3. 连接成功
    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
        print("连接成功，开始发现服务")
        peripheral.discoverServices(nil)
    }

    // MARK: - 4. 发现服务
    func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
        guard let services = peripheral.services else { return }
        for service in services {
            print("发现服务: \(service.uuid)")
            peripheral.discoverCharacteristics(nil, for: service)
        }
    }

    // MARK: - 5. 发现特征
    func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService,
                    error: Error?) {
        guard let characteristics = service.characteristics else { return }
        for char in characteristics {
            print("发现特征: \(char.uuid)")

            // 读取值
            peripheral.readValue(for: char)

            // 如果是 Notify 特征，可以订阅
            if char.properties.contains(.notify) {
                peripheral.setNotifyValue(true, for: char)
            }
        }
    }

    // MARK: - 6. 接收数据
    func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic,
                    error: Error?) {
        if let value = characteristic.value {
            print("接收到数据: \(value.hexEncodedString())")
        }
    }

    // MARK: - 7. 写入数据结果
    func peripheral(_ peripheral: CBPeripheral, didWriteValueFor characteristic: CBCharacteristic, error: Error?) {
        print("写入数据完成: \(characteristic.uuid)")
    }
}

// 小扩展：把 Data 转成 16 进制字符串方便打印
extension Data {
    func hexEncodedString() -> String {
        return map { String(format: "%02hhx", $0) }.joined(separator: " ")
    }
}
```

---

## 📘 小贴士

* 某些蓝牙设备只开放特定的 UUID 服务和特征，建议开发前看设备说明文档。
* 如果你只想连接某一类设备（如体温计），可以在 `scanForPeripherals(withServices:)` 中指定 Service UUID。
* `CBPeripheralDelegate` 和 `CBCentralManagerDelegate` 要设置好代理，不然不会触发回调。

---

如你需要 **Objective-C 版本** 或配合 UI 控件操作（比如按钮触发连接），我也可以帮你写完整示例，要不要我再给个带界面的 demo 版本？
