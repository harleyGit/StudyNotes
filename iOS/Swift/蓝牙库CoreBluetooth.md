
> <h1 id=""></h1>
- [**BLE核心类联系**](#BLE核心类联系)
	- [蓝牙中央设备CBCentralManager](#蓝牙中央设备CBCentralManager)
		- [`‌didDiscoverCharacteristicsFor`代理调用顺序](#didDiscoverCharacteristicsFor代理调用顺序)
	- [蓝牙外设CBPeripheral](#蓝牙外设CBPeripheral) 
	- [外设服务CBService](#外设服务CBService)
	- [服务特征CBCharacteristic](#服务特征CBCharacteristic)
	- [描述蓝牙UUID的服务特征类CBUUID](#描述蓝牙UUID的服务特征类CBUUID)
- [**连接成功的蓝牙外围设备-Peripheral**](#连接成功的蓝牙外围设备-Peripheral)
	- [代理方法CBPeripheralDelegate](#代理方法CBPeripheralDelegate)
	- [启动连接蓝牙外设](#启动连接蓝牙外设)
	- [轮询获取蓝牙开启状态](#轮询获取蓝牙开启状态)
- [**连接蓝牙外设**](#连接蓝牙外设)
	- [连接温度计设备](#连接温度计设备)
	- [向蓝牙外设发送数据](#向蓝牙外设发送数据)
	- [接收蓝牙外设发送数据](#接收蓝牙外设发送数据)
- [**密钥**](#密钥)
	- [生成共享密钥对](#生成共享密钥对)
- [**解析字节数据**](#解析字节数据)
	- [大端和小端区别](#大端和小端区别)
	- [Byte数组构造BLE蓝牙发送指令](#Byte数组构造BLE蓝牙发送指令)
	- [自描述式二进制数据包格式](#自描述式二进制数据包格式)
	- [结构体解包数据](#结构体解包数据)
	- [小端字节序还原成一个uint16_t类型的整数](#小端字节序还原成一个uint16_t类型的整数)
	- [读取一段字节数据范围中的某个字节的某一位bool值](#读取一段字节数据范围中的某个字节的某一位bool值)
	- [字节范围取值](#字节范围取值)
	- [判断某一个字节是否为1](#判断某一个字节是否为1)
	- [连续Byte某段bit的十进制值](#连续Byte某段bit的十进制值)
	- [16字节转化为16进制的数字符串](#16字节转化为16进制的数字符串)
	- [蓝牙数据分包](#蓝牙数据分包)
	- [多个字节转成UInt类型数字](#多个字节转成UInt类型数字)
- [蓝牙外设写入数据调度器](#蓝牙外设写入数据调度器)
- [**调试**](#调试)
	- [MobaXterm在Windows系统上的集成终端工具](#MobaXterm在Windows系统上的集成终端工具)
	- [screen终端工具串口调试](#screen终端工具串口调试)
	- [minicom终端串口工具](#minicom终端串口工具)
- [硬件自带Wi-Fi](#硬件自带Wi-Fi)
	- [App加入设备Wi-Fi网络](#App加入设备Wi-Fi网络)
- [iot配网简单架构](#iot简单配网架构)
- **借鉴资料**
	- [iOS 蓝牙（中心模式连接外设）](https://juejin.cn/post/7129891777783267342)

<br/><br/><br/>

***
<br/>

> <h1 id="BLE核心类联系">BLE核心类联系</h1>

**核心 BLE 类之间的关系图:**

```sh
CBCentralManager       // 蓝牙中央设备（App 作为主控）
 └── CBPeripheral      // 外设（你连接的蓝牙设备）
      └── CBService    // 外设的服务（功能模块，如心率、血压）
           └── CBCharacteristic // 服务的特征（具体数据点，如心率值）
```


***
<br/><br/><br/>

> <h2 id="蓝牙中央设备CBCentralManager">蓝牙中央设备CBCentralManager</h2>

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

<br/>

**🛠️ 它有什么用？**

- 常见的用途包括：
	* 连接智能手环、蓝牙体温计、蓝牙耳机等外设
	* 从外设读取数据（如心率、温度）
	* 向外设发送指令（如设置参数）

<br/>

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
            
            let serviceUUID = CBUUID(string: "180D") // 示例：心率服务
            // 若是options: nil,则即使蓝牙外设多次发广播（比如你按了 K3 键），iOS 只会在第一次发现时触发一次 didDiscoverPeripheral，后续不会再收到，即使广播内容变了
            // 只扫描包含特定 Service Data（如 FE95）的 BLE 设备,需要设置[serviceUUIDs],如小米的:[@"FE95"]
            // [self.centralManager scanForPeripheralsWithServices:serviceUUIDs
                                            options:@{CBCentralManagerScanOptionAllowDuplicatesKey: @(YES)}];
                                            


            centralManager.scanForPeripherals(withServices: [serviceUUID], options: nil)
        } else {
            print("蓝牙不可用")
        }
    }

    // 2. 扫描到设备
    // peripheral: 蓝牙设备标识; RSSI: 信号强度; advertisementData: 广播包数据(iOS无法扫描出蓝牙Mac,要把Mac放到广播里面)
    // advertisementData 是广播数据字典，包含当前蓝牙外设广播出来的内容。
    func centralManager(_ central: CBCentralManager,
                        didDiscover peripheral: CBPeripheral,
                        advertisementData: [String : Any],
                        rssi RSSI: NSNumber) {
        print("发现设备: \(peripheral.name ?? "未知设备")")
        // 保存外设并连接
        discoveredPeripheral = peripheral
        // 停止扫描
        centralManager.stopScan()
        // 开始连接蓝牙外设
        centralManager.connect(peripheral, options: nil)
    }

    // 3. 连接成功(蓝牙外设连接成功后会调用这个代理方法)
    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
        print("已连接 \(peripheral.name ?? "设备")")
        // 设置代理接收数据
        peripheral.delegate = self
        // 开始发现服务
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
            print("发现特征: \(characteristic.uuid), 发现属性: \(characteristic.properties)")
            
            // 读取特征值
            peripheral.readValue(for: characteristic)
            
            if characteristic.properties.contains(.notify) {
	            // 订阅通知,若是不订阅无法接受蓝牙外设发来的数据
	            peripheral.setNotifyValue(true, for: characteristic)            
            }
        }
    }

    // 6. 接收特征值
    func peripheral(_ peripheral: CBPeripheral,
                    didUpdateValueFor characteristic: CBCharacteristic,
                    error: Error?) {
        if let data = characteristic.value {
	        let let hexString = value.map { String(format: "%02x", $0) }.joined(separator: " ")
	        print("接收到数据: \(hexString)")
	        DispatchQueue.main.async {
                // self.dataTextView.text += "\n收到：\(hexString)"
                // 若这里是UI组件,可以将蓝牙返回值进行赋值
            }
        }
    }
}



// 小扩展：把 Data 转成 16 进制字符串方便打印
extension Data {
    func hexEncodedString() -> String {
        return map { String(format: "%02hhx", $0) }.joined(separator: " ")
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


<br/><br/>
> <h3 id="didDiscoverCharacteristicsFor代理调用顺序">didDiscoverCharacteristicsFor`代理调用顺序</h3>

**代理方法：**

```swift
func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?)
```

是 CoreBluetooth 框架中用来处理 **发现特征** 的代理方法。

<br/>

- 🔍 它 **何时触发？什么时候会走这个方法？**

当你调用：

```swift
peripheral.discoverCharacteristics(nil, for: service)
```

之后，系统会异步搜索该 `service` 下所有的特征（characteristics），并在搜索完成后，调用这个代理方法。

<br/><br/>

**✅ 方法触发流程梳理**

整个流程是：

- **1.连接成功后，调用：**

```swift
peripheral.discoverServices(nil)
```

<br/>

- **2.回调到：**

```swift
func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?)
```

<br/>

在这里你会遍历每一个 `service`，继续发现特征：

```swift
for service in peripheral.services ?? [] {
   peripheral.discoverCharacteristics(nil, for: service)
}
```

<br/>

- **3.然后就会触发你提到的这个方法：**

   ```swift
   func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?)
   ```

<br/>

**🧠 方法内部逻辑详解**

```swift
guard let characteristics = service.characteristics else { return }

for char in characteristics {
    print("发现特征: \(char.uuid)")

    // 主动读取一次特征的当前值（如果需要）
    peripheral.readValue(for: char)

    // 如果该特征支持 Notify，则订阅它以便接收外设主动推送的数据
    if char.properties.contains(.notify) {
        peripheral.setNotifyValue(true, for: char)
    }
}
```

| 行                            | 作用                             |
| ---------------------------- | ------------------------------ |
| `readValue(for:)`            | 主动读取一次当前特征值（例如读取设备电量、设备名）      |
| `setNotifyValue(true, for:)` | 订阅该特征，之后只要设备有数据变化，会自动通知并触发代理回调 |

<br/>

- **✅ 通知数据从哪里来？**

只要你订阅了 `.notify` 的特征，外设更新该特征时，你就会收到回调：

```swift
func peripheral(_ peripheral: CBPeripheral, 
                didUpdateValueFor characteristic: CBCharacteristic, 
                error: Error?) {
    let data = characteristic.value
    print("接收到新数据: \(String(describing: data))")
    // 这里你可以解析 data
}
```

<br/>

- **✅ 实战示例代码（完整）**

```swift
class BLEManager: NSObject, CBCentralManagerDelegate, CBPeripheralDelegate {
    var centralManager: CBCentralManager!
    var discoveredPeripheral: CBPeripheral?

    override init() {
        super.init()
        centralManager = CBCentralManager(delegate: self, queue: nil)
    }

    func centralManagerDidUpdateState(_ central: CBCentralManager) {
        if central.state == .poweredOn {
            central.scanForPeripherals(withServices: nil)
        }
    }

    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral,
                        advertisementData: [String : Any], rssi RSSI: NSNumber) {
        central.stopScan()
        discoveredPeripheral = peripheral
        peripheral.delegate = self
        central.connect(peripheral)
    }

    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
        print("连接成功: \(peripheral.name ?? "未知设备")")
        peripheral.discoverServices(nil)
    }

    func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
        for service in peripheral.services ?? [] {
            peripheral.discoverCharacteristics(nil, for: service)
        }
    }

    func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {
        for char in service.characteristics ?? [] {
            print("发现特征: \(char.uuid)")

            peripheral.readValue(for: char)

            if char.properties.contains(.notify) {
                peripheral.setNotifyValue(true, for: char)
            }
        }
    }

    func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {
        guard let data = characteristic.value else { return }
        print("收到新数据: \(data as NSData)")
        // 可以在此解析数据
    }
}
```

---

## ✅ 总结重点

| 步骤    | 方法                                   | 用途          |
| ----- | ------------------------------------ | ----------- |
| 连接设备后 | `discoverServices`                   | 发现服务        |
| 服务发现后 | `discoverCharacteristics`            | 发现特征        |
| 特征发现后 | `readValue` / `setNotifyValue(true)` | 主动读取 / 订阅数据 |
| 数据更新后 | `didUpdateValueForCharacteristic`    | 接收设备推送数据    |




***
<br/><br/><br/>

> <h2 id="蓝牙外设CBPeripheral"> 蓝牙外设CBPeripheral </h2>

代表一个蓝牙 **外设**（Peripheral），通过 `CBCentralManager` 扫描到设备后获得。

常用属性和方法：

| 属性 / 方法                                              | 说明             |
| ---------------------------------------------------- | -------------- |
| `name`                                               | 设备名称           |
| `services`                                           | 连接后发现的所有服务（数组） |
| `discoverServices([CBUUID]?)`                        | 发起服务发现         |
| `discoverCharacteristics([CBUUID]?, for: CBService)` | 发现服务下的特征       |
| `readValue(for: CBCharacteristic)`                   | 读取某个特征的值       |
| `setNotifyValue(true/false, for:)`                   | 订阅通知特征, **让外设有数据更新时自动推送给你（App）**         |


***
<br/><br/><br/>

> <h2 id="外设服务CBService">外设服务CBService</h2>

- **🧠 什么是“服务（Service）”？**
	- 比如: **心率监测、血压测量、设备信息**等

<br/>

- **在 BLE 蓝牙中：**
	* 每个设备都定义了一组“服务”（Service）；
	* 每个服务中包含若干个“特征”（Characteristic）；
	* 特征才是真正承载“功能”或“数据”的单元。

🔧 你可以把结构类比为：

```
外设（Peripheral）
└── 服务（Service） → 比如电池服务 / 心率服务
    └── 特征（Characteristic） → 电量值 / 心率数值
```

<br/>

**获取服务:**

```swift
CBService *service = ...; 
NSArray *characteristics = service.characteristics;
```

<br/>

**常用属性和方法：**

| 属性 / 方法           | 说明                                           |
| ----------------- | -------------------------------------------- |
| `uuid`            | 服务的 UUID                                     |
| `isPrimary`       | 是否为主服务                                       |
| `characteristics` | 当前服务下的所有特征（需要调用 discoverCharacteristics 后才有） |

 ✅ 注意：必须连接设备后并调用 `discoverServices(: )`，并在回调中使用 `peripheral.discoverCharacteristics(nil, for: service)` 才能拿到 `characteristics`

<br/>

```objc
// 传入 nil 表示：发现该设备提供的所有服务（Service）；
[peripheral discoverServices:nil];

// 或者是:
// 这里的 UUID 是你已知的某个设备提供的自定义服务
NSArray *uuids = @[@"4a6e4000-5e16-459a-9e5b-86f23a314181"];

// 将字符串 UUID 转为 CoreBluetooth 可识别的 CBUUID 类型
// 作用: 将 NSString 类型的 UUID 转换为 CBUUID 类型（CoreBluetooth 框架中使用的 UUID 类型），方便接下来的服务发现操作
//   CBUUID 是苹果专门定义的 BLE UUID 类型
//   UUIDWithString: 可以将字符串 "4a6e4000-..." 转成 CBUUID 对象。
NSMutableArray<CBUUID *> *cbUUIDs = [NSMutableArray array];
[uuids enumerateObjectsUsingBlock:^(NSString *obj, NSUInteger idx, BOOL *stop) {
    [cbUUIDs addObject:[CBUUID UUIDWithString:obj]];
}];
// 此语句尝试去发现蓝牙外设中的指定服务 UUID（而不是全部服务），这是正确的 API 调用方法
// cbPeripheral 是你通过 scanForPeripherals 扫描并连接成功的设备实例。
[cbPeripheral discoverServices:cbUUIDs];

```

- **参数解析:**
	* 传入 `nil`：表示**发现该外设的所有服务**。
	* 你也可以传入一个数组，指定只查找某些 UUID 的服务，例如：

```objc
[peripheral discoverServices:@[[CBUUID UUIDWithString:@"180D"]]]; // 查找心率服务
```


`discoverServices:` 是 CoreBluetooth 中的一个非常重要的方法，它的作用是：**向指定的 `CBPeripheral` 发送请求，查找它所提供的服务（Services）**。

<br/>

**📌 `discoverServices:`调用后会触发异步回调：**

```objc
- (void)peripheral:(CBPeripheral *)peripheral 
didDiscoverServices:(NSError *)error;
```

你可以在这个方法中遍历 `peripheral.services`，然后继续调用：

```objc
[peripheral discoverCharacteristics:nil forService:service];
```

去查找该服务下的特征。

<br/>

**✅ 示例代码简化**

```objc
// 连接成功后
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
    peripheral.delegate = self;
    [peripheral discoverServices:nil]; // 🔍 请求发现所有服务
}

// 服务发现回调
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error {
    for (CBService *service in peripheral.services) {
        NSLog(@"发现服务：%@", service.UUID);
        [peripheral discoverCharacteristics:nil forService:service];
    }
}
```

<br/>

- **✅总结一句话：**

```objc
[peripheral discoverServices:nil];
```

&emsp; 是你开始“探索设备能力”的**第一步**，相当于敲开设备的门，看看它能做哪些事情（通过服务/特征暴露）。




***
<br/><br/><br/>

> <h2 id="服务特征CBCharacteristic">服务特征CBCharacteristic</h2>

代表一个服务下的一个 **特征值**（数据通道），比如：温度值、心率值、设备电量等。

常用属性和方法：

| 属性 / 方法       | 说明                 |
| ------------- | ------------------ |
| `uuid`        | 特征 UUID            |
| `properties`  | 支持的操作类型（可读、可写、可订阅） |
| `value`       | 当前读取到的数据（二进制）      |
| `isNotifying` | 是否已订阅该特征           |
| `descriptors` | 特征的描述符（较少用）        |

常见操作：

* `.read`：读取数据
* `.write`：写入数据
* `.notify`：订阅通知


<br/><br/>

- **📌 特别说明：UUID 是什么？**
	* 每个服务或特征都有一个唯一标识符（UUID）
	* 标准蓝牙协议的服务/特征使用 **16位 UUID**（如：`180D` 心率）
	* 厂商自定义服务/特征使用 **128位 UUID**



***
<br/><br/><br/>

> <h2 id="描述蓝牙UUID的服务特征类CBUUID">描述蓝牙UUID的服务特征类CBUUID</h2>

当然可以，我们重点来详细解释这句代码：

```objc
CBUUID *fe95UUID = [CBUUID UUIDWithString:@"FE95"];
```

<br/>

 **✅ 这句代码的作用**

> 创建一个表示蓝牙 UUID 为 `FE95` 的 `CBUUID` 对象，用于后续匹配蓝牙设备的 Service 或广播中的数据。

<br/> 

**🔍 分析每个部分**

- **📌 `CBUUID`**
	* `CBUUID` 是 CoreBluetooth 框架中专门用来描述 **蓝牙 UUID（服务或特征）** 的类。
	* 它支持 16-bit、32-bit 和 128-bit 的 UUID 格式。

<br/>

- **📌`+ (CBUUID *)UUIDWithString:(NSString *)theString`**
	* 这是一个类方法，用来根据字符串创建一个 `CBUUID` 实例。
	* 比如：

```objc
CBUUID *uuid = [CBUUID UUIDWithString:@"180D"]; // 心率服务
CBUUID *uuid = [CBUUID UUIDWithString:@"FE95"]; // 厂商自定义服务（如小米）
```

<br/>

- **📌 `"FE95"`**
	* 这是一个**标准的 16 位蓝牙 UUID**，以 16 进制表示。
	* 前缀 `"FE"` 开头的 UUID 属于 **厂商专用（Vendor Specific）** 范围。
	* `FE95` 是\*\*小米（Xiaomi）\*\*在其设备中广泛使用的 Service UUID，用于广播设备信息、传感器数据等。

<br/> 

**🚩这句代码的实际效果**

```objc
CBUUID *fe95UUID = [CBUUID UUIDWithString:@"FE95"];
```

- 你获得了一个 `CBUUID` 实例，这个对象之后可以用来：
	* 查找广播数据中的 `"Service Data"` 是否包含该 UUID
	* 连接后匹配 peripheral 中的服务
	* 与某个 `CBCharacteristic` 的 `UUID` 进行比较

<br/> 

**🧠 补充知识：UUID 是蓝牙中识别“服务”和“特征”的唯一标识**

* 举几个常见的 UUID 对应用途：

  | UUID                                                     | 含义         |
  | -------------------------------------------------------- | ---------- |
  | `180D`                                                   | 心率服务       |
  | `180F`                                                   | 电池服务       |
  | `FE95`                                                   | 小米厂商定义服务   |
  | `FEE0`/`FEE1`                                            | 华为厂商定义服务   |
  | `UUIDWithString:@"XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"` | 128 位自定义服务 |

<br/> 

**✅ 总结**

| 部分                       | 含义                                |
| ------------------------ | --------------------------------- |
| `CBUUID`                 | CoreBluetooth 框架用于描述服务/特征 UUID 的类 |
| `UUIDWithString:@"FE95"` | 创建一个用于匹配小米蓝牙服务的 UUID 实例           |
| 应用场景                     | 用于识别广播、查找服务、匹配特征等操作               |

<br/>

如果你看到后续有：

```objc
NSData *data = serviceData[fe95UUID];
```

就是表示在广播中找是否包含 UUID 为 `FE95` 的那块数据，并取出它来解析。


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

***
<br/><br/><br/>

> <h2 id="代理方法CBPeripheralDelegate">代理方法CBPeripheralDelegate</h2>


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

```sh
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
            // 若是options: nil,则即使蓝牙外设多次发广播（比如你按了 K3 键），iOS 只会在第一次发现时触发一次 didDiscoverPeripheral，后续不会再收到，即使广播内容变了
            // [self.centralManager scanForPeripheralsWithServices:serviceUUIDs
                                            options:@{CBCentralManagerScanOptionAllowDuplicatesKey: @(YES)}];

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

<br/><br/>

**📘 小贴士**

* 某些蓝牙设备只开放特定的 UUID 服务和特征，建议开发前看设备说明文档。
* 如果你只想连接某一类设备（如体温计），可以在 `scanForPeripherals(withServices:)` 中指定 Service UUID。
* `CBPeripheralDelegate` 和 `CBCentralManagerDelegate` 要设置好代理，不然不会触发回调。



***
<br/><br/><br/>
> <h2 id="启动连接蓝牙外设">启动连接蓝牙外设</h2>

```swift
func retrievePeripherals(withIdentifiers identifiers: [UUID]) -> [CBPeripheral]
```

**上述方法**是 CoreBluetooth 框架中 `CBCentralManager` 的一个方法，用于：

> 🔍 **根据设备的 UUID 获取之前连接过的蓝牙外设对象（CBPeripheral）**。

---

- **🧠 使用场景**
	* **不再扫描周围设备**；
	* **直接恢复并获取你之前连接过的目标蓝牙设备**；
	* 适用于“已知设备”，如你在 App 中保存了该设备的 `UUID`；
	* 可以配合后台重连、断线重连、App 重启恢复连接等场景。

---

**📦 示例用法（Swift）**

```swift

**✅ 搭配使用建议**

你可以在连接成功后保存设备 UUID：

```swift
let uuidString = peripheral.identifier.uuidString
UserDefaults.standard.set(uuidString, forKey: "LastConnectedPeripheral")
```

下次启动时用：

```swift
if let uuidString = UserDefaults.standard.string(forKey: "LastConnectedPeripheral"),
   let uuid = UUID(uuidString: uuidString) {
    let peripherals = centralManager.retrievePeripherals(withIdentifiers: [uuid])
   
    // 连接逻辑
    if let targetPeripheral = peripherals.first {
	    targetPeripheral.delegate = self
	    centralManager.connect(targetPeripheral, options: nil)
    }
}
```

---

**📌 注意事项**

| 点        | 说明                                             |
| -------- | ---------------------------------------------- |
| UUID 来源  | 这个 UUID 必须是 iOS 系统曾经为你连接过的设备生成的（不能是设备的 MAC 地址） |
| 跨设备      | 不支持跨 iPhone 使用（UUID 是 iOS 为当前设备生成的）            |
| 扫描 vs 恢复 | 它不会主动扫描，只是从系统的“缓存”中恢复已知设备                      |
| 常用于      | App 重启后自动重连、后台唤醒自动重连等场景                        |

---

**🚫 不能做的事**

| 想法             | 是否可行                                                |
| -------------- | --------------------------------------------------- |
| 用设备 MAC 地址恢复连接 | ❌ iOS 不支持公开 MAC 地址,但是你的蓝牙设备你可以加上MAC地址                                  |
| 从未连接过的设备恢复     | ❌ 系统无缓存，无法返回                                        |
| 获取所有已连接设备      | ✅ 请使用 `retrieveConnectedPeripherals(withServices:)` |


***
<br/>

在 Objective-C 中，可以使用 `retrievePeripheralsWithIdentifiers:` 方法来恢复连接之前已知 UUID 的蓝牙设备。

**✅ 示例：Objective-C 恢复已知设备并连接**

```objc
#import <CoreBluetooth/CoreBluetooth.h>

- (void)restorePeripheralWithUUID {
    // 创建一个已知的 NSUUID（假设你之前保存过）
    NSUUID *uuid = [[NSUUID alloc] initWithUUIDString:@"E2C56DB5-DFFB-48D2-B060-D0F5A71096E0"];

    // 调用 CBCentralManager 的 retrieve 方法
    NSArray<CBPeripheral *> *peripherals = [self.centralManager retrievePeripheralsWithIdentifiers:@[uuid]];

    if (peripherals.count > 0) {
        CBPeripheral *targetPeripheral = peripherals.firstObject;
        targetPeripheral.delegate = self;
        [self.centralManager connectPeripheral:targetPeripheral options:nil];
        NSLog(@"🔗 尝试连接已知设备 %@", targetPeripheral.name);
    } else {
        NSLog(@"⚠️ 未找到已知 UUID 对应的设备");
    }
}
```

---

**NSUserDefaults保存 UUID（首次连接成功时）**

你可以在连接成功的回调中保存 `peripheral.identifier`：

```objc
- (void)centralManager:(CBCentralManager *)central
  didConnectPeripheral:(CBPeripheral *)peripheral {
    NSLog(@"✅ 已连接设备：%@", peripheral.name);

    NSString *uuidString = peripheral.identifier.UUIDString;
    [[NSUserDefaults standardUserDefaults] setObject:uuidString forKey:@"SavedPeripheralUUID"];
}
```

---

**✅ 启动时恢复连接**

在初始化 `CBCentralManager` 并状态变为 `PoweredOn` 时，执行：

```objc
- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
    if (central.state == CBManagerStatePoweredOn) {
        NSString *uuidString = [[NSUserDefaults standardUserDefaults] stringForKey:@"SavedPeripheralUUID"];
        if (uuidString) {
            NSUUID *uuid = [[NSUUID alloc] initWithUUIDString:uuidString];
            NSArray<CBPeripheral *> *peripherals = [central retrievePeripheralsWithIdentifiers:@[uuid]];
            if (peripherals.count > 0) {
                CBPeripheral *peripheral = peripherals.firstObject;
                peripheral.delegate = self;
                [central connectPeripheral:peripheral options:nil];
            }
        }
    }
}
```

---

**📌 小结**

| 步骤        | 方法                                                     |
| --------- | ------------------------------------------------------ |
| 保存设备 UUID | `peripheral.identifier.UUIDString` 存入 `NSUserDefaults` |
| 恢复设备对象    | `retrievePeripheralsWithIdentifiers:`                  |
| 连接设备      | `connectPeripheral:options:`                           |


***
<br/><br/><br/>
> <h2 id="轮询获取蓝牙开启状态">轮询获取蓝牙开启状态</h2>

轮询查看蓝牙是否开启，因为蓝牙的状态是异步的，获取的时候并不能准确获取。为了获取到准确的状态，只能通过比较准确的轮询进行获取其状态。

```swift
func pollBluetoothStateOnce(timeout: TimeInterval = 1.0, check: @escaping (Bool) -> Void) {
    let interval: TimeInterval = 0.2 // 1 秒查询 5 次
    var elapsedTime: TimeInterval = 0
    let timer = DispatchSource.makeTimerSource(queue: DispatchQueue.global())
    
    timer.schedule(deadline: .now(), repeating: interval)
    timer.setEventHandler {
        let state = CBCentralManager().state // 改成你自己的获取蓝牙状态的方法
        
        if state == .poweredOn {
            timer.cancel()
            DispatchQueue.main.async { check(true) }
        } else if state == .poweredOff {
            timer.cancel()
            DispatchQueue.main.async { check(false) }
        } else {
            elapsedTime += interval
            if elapsedTime >= timeout {
                timer.cancel()
                DispatchQueue.main.async { check(false) }
            }
        }
    }
    
    timer.resume()
}
```

用法：

```swift
pollBluetoothStateOnce { isOn in
    print("蓝牙状态：\(isOn ? "开" : "关")")
}
```

特点：

* 每秒检查 5 次（间隔 0.2 秒）
* 总时长 1 秒（可调）
* 一旦检测到蓝牙**开**或**关**就会立刻结束
* 超时后返回 **false**


***
<br/><br/><br/>
> <h2 id="优化轮询连接蓝牙外设">优化轮询连接蓝牙外设</h2>


在开发时遇到这样的场景：**先扫描→按 MAC 存字典→立刻尝试连接**。若此时还没扫到该外设，就要**用 GCD 轮询**去找并连接；一旦**连接成功就停止轮询**；若**超时**则**停止扫描和连接**并回调结果。

下面给你一套可直接用的 **Objective-C** 实现（纯 `dispatch_queue_t/dispatch_source_t`，不使用 `NSTimer`）。


<br/>

**头文件（示例）**

```objc
// AKBLEConnector.h
#import <Foundation/Foundation.h>
#import <CoreBluetooth/CoreBluetooth.h>

typedef void(^AKBLEConnectCompletion)(BOOL success, CBPeripheral * _Nullable peripheral, NSError * _Nullable error);

@interface AKBLEConnector : NSObject <CBCentralManagerDelegate>

@property (nonatomic, strong, readonly) CBCentralManager *central;
/// 扫描阶段把发现的设备按“规范化后的 MAC”放这里（例如去掉冒号并大写）
@property (nonatomic, strong, readonly) NSMutableDictionary<NSString *, CBPeripheral *> *peripheralsByMAC;

- (instancetype)initWithQueue:(dispatch_queue_t _Nullable)queue;

/// 轮询按 MAC 连接
/// @param mac 目标设备 MAC（支持带冒号或不带）
/// @param timeout 超时时长（秒）
/// @param retriesPerSecond 每秒轮询次数（传 5 即每 0.2s 一次）
/// @param completion 结果回调（主线程）
- (void)connectByMAC:(NSString *)mac
             timeout:(NSTimeInterval)timeout
    retriesPerSecond:(NSInteger)retriesPerSecond
          completion:(AKBLEConnectCompletion)completion;

/// 外部扫描时在 didDiscover 里调用，把发现的外设放到字典
- (void)cacheDiscoveredPeripheral:(CBPeripheral *)peripheral mac:(NSString *)mac;

@end
```

<br/>

 **实现文件**

```objc
// AKBLEConnector.m
#import "AKBLEConnector.h"

@interface AKBLEConnector ()
@property (nonatomic, strong) CBCentralManager *central;
@property (nonatomic, strong) NSMutableDictionary<NSString *, CBPeripheral *> *peripheralsByMAC;

@property (nonatomic, strong) dispatch_source_t connectTimer;
@property (nonatomic, copy)   AKBLEConnectCompletion connectCompletion;

@property (nonatomic, strong) CBPeripheral *connectingPeripheral;
@property (atomic, assign)    BOOL isConnecting;
@property (atomic, assign)    BOOL didFinish; // 防重复回调
@end

@implementation AKBLEConnector

#pragma mark - Init

- (instancetype)initWithQueue:(dispatch_queue_t)queue {
    self = [super init];
    if (self) {
        _peripheralsByMAC = [NSMutableDictionary dictionary];
        _central = [[CBCentralManager alloc] initWithDelegate:self queue:queue];
    }
    return self;
}

#pragma mark - Public

- (void)cacheDiscoveredPeripheral:(CBPeripheral *)peripheral mac:(NSString *)mac {
    if (!peripheral || mac.length == 0) return;
    NSString *key = [self.class normalizedMAC:mac];
    @synchronized (self.peripheralsByMAC) {
        self.peripheralsByMAC[key] = peripheral;
    }
}

- (void)connectByMAC:(NSString *)mac
             timeout:(NSTimeInterval)timeout
    retriesPerSecond:(NSInteger)retriesPerSecond
          completion:(AKBLEConnectCompletion)completion
{
    if (mac.length == 0) {
        if (completion) {
            NSError *err = [NSError errorWithDomain:@"AKBLE"
                                               code:-1000
                                           userInfo:@{NSLocalizedDescriptionKey:@"MAC 为空"}];
            dispatch_async(dispatch_get_main_queue(), ^{ completion(NO, nil, err); });
        }
        return;
    }

    // 准备
    self.didFinish = NO;
    self.isConnecting = NO;
    self.connectingPeripheral = nil;
    self.connectCompletion = completion;

    // 若蓝牙没开，直接失败
    if (self.central.state != CBManagerStatePoweredOn) {
        [self finishWithSuccess:NO peripheral:nil
                          error:[NSError errorWithDomain:@"AKBLE"
                                                   code:-1001
                                               userInfo:@{NSLocalizedDescriptionKey:@"蓝牙未开启"}]];
        return;
    }

    // 开始扫描（允许重复回调，尽快拿到外设）
    NSDictionary *scanOpts = @{ CBCentralManagerScanOptionAllowDuplicatesKey : @YES };
    [self.central scanForPeripheralsWithServices:nil options:scanOpts];

    // 轮询定时器（GCD）
    NSTimeInterval interval = (retriesPerSecond > 0) ? (1.0 / retriesPerSecond) : 0.2; // 默认 5 次/秒
    __block NSTimeInterval elapsed = 0;

    dispatch_queue_t timerQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    self.connectTimer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, timerQueue);
    dispatch_source_set_timer(self.connectTimer,
                              dispatch_time(DISPATCH_TIME_NOW, 0),
                              (uint64_t)(interval * NSEC_PER_SEC),
                              0);

    __weak typeof(self) weakSelf = self;
    dispatch_source_set_event_handler(self.connectTimer, ^{
        __strong typeof(self) self_ = weakSelf;
        if (!self_ || self_.didFinish) return;

        // 已连接？（更快：依赖 delegate；这里做兜底）
        if (self_.connectingPeripheral.state == CBPeripheralStateConnected) {
            [self_ finishWithSuccess:YES peripheral:self_.connectingPeripheral error:nil];
            return;
        }

        // 查找目标外设
        NSString *key = [self.class normalizedMAC:mac];
        __block CBPeripheral *target = nil;
        @synchronized (self_.peripheralsByMAC) {
            target = self_.peripheralsByMAC[key];
        }

        // 拿到外设后，若尚未发起连接 → 发起连接
        if (target) {
            if (self_.connectingPeripheral != target) {
                self_.connectingPeripheral = target;
                self_.connectingPeripheral.delegate = nil; // 如需用到，外部自行设置
                self_.isConnecting = YES;

                NSDictionary *opts = @{
                    CBConnectPeripheralOptionNotifyOnConnectionKey: @YES,
                    CBConnectPeripheralOptionNotifyOnDisconnectionKey: @YES
                };
                [self_.central connectPeripheral:target options:opts];
            } else {
                // 已经在连，如果处于断开状态可重试发起
                if (!self_.isConnecting &&
                    (target.state == CBPeripheralStateDisconnected))
                {
                    self_.isConnecting = YES;
                    [self_.central connectPeripheral:target options:nil];
                }
            }
        }

        // 超时
        elapsed += interval;
        if (elapsed >= timeout) {
            // 停止扫描与连接
            if (self_.connectingPeripheral) {
                [self_.central cancelPeripheralConnection:self_.connectingPeripheral];
            }
            [self_ finishWithSuccess:NO peripheral:self_.connectingPeripheral
                               error:[NSError errorWithDomain:@"AKBLE"
                                                        code:-1002
                                                    userInfo:@{NSLocalizedDescriptionKey:@"连接超时"}]];
        }
    });

    dispatch_resume(self.connectTimer);
}

#pragma mark - Helpers

+ (NSString *)normalizedMAC:(NSString *)mac {
    // 统一成大写且去掉非十六进制字符（: - 空格等）
    NSCharacterSet *set = [[NSCharacterSet characterSetWithCharactersInString:@"0123456789ABCDEFabcdef"] invertedSet];
    NSString *raw = [[mac componentsSeparatedByCharactersInSet:set] componentsJoinedByString:@""];
    return raw.uppercaseString;
}

- (void)finishWithSuccess:(BOOL)success
               peripheral:(CBPeripheral * _Nullable)peripheral
                    error:(NSError * _Nullable)error
{
    if (self.didFinish) return;
    self.didFinish = YES;

    // 停止轮询 & 扫描
    if (self.connectTimer) {
        dispatch_source_cancel(self.connectTimer);
        self.connectTimer = nil;
    }
    [self.central stopScan];

    // 回到主线程回调一次
    if (self.connectCompletion) {
        AKBLEConnectCompletion block = self.connectCompletion;
        self.connectCompletion = nil;
        dispatch_async(dispatch_get_main_queue(), ^{
            block(success, peripheral, error);
        });
    }

    // 清理状态
    self.isConnecting = NO;
}

#pragma mark - CBCentralManagerDelegate

- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
    // 可按需处理蓝牙状态变化
}

- (void)centralManager:(CBCentralManager *)central
  didDiscoverPeripheral:(CBPeripheral *)peripheral
      advertisementData:(NSDictionary<NSString *,id> *)advertisementData
                   RSSI:(NSNumber *)RSSI
{
    // 如果你能从广播中解析出 MAC，就放进字典（这里留接口给外部）
    // NSString *mac = [self parseMACFromAdvertisement:advertisementData];
    // if (mac) { [self cacheDiscoveredPeripheral:peripheral mac:mac]; }
}

- (void)centralManager:(CBCentralManager *)central
 didConnectPeripheral:(CBPeripheral *)peripheral
{
    // 连接成功 → 立即结束轮询并回调
    if (self.didFinish) return;
    if (peripheral == self.connectingPeripheral) {
        [self finishWithSuccess:YES peripheral:peripheral error:nil];
    }
}

- (void)centralManager:(CBCentralManager *)central
didFailToConnectPeripheral:(CBPeripheral *)peripheral
                 error:(NSError *)error
{
    // 连接失败：不立刻回调失败，继续在超时时间内轮询重试
    if (peripheral == self.connectingPeripheral) {
        self.isConnecting = NO; // 允许下一轮轮询再次发起连接
    }
}

- (void)centralManager:(CBCentralManager *)central
didDisconnectPeripheral:(CBPeripheral *)peripheral
                 error:(NSError *)error
{
    // 若在连接窗口内发生断开，也允许轮询逻辑重新尝试（不直接回调失败）
    if (!self.didFinish && peripheral == self.connectingPeripheral) {
        self.isConnecting = NO;
    }
}

@end
```

<br/>

**使用示例**

```objc
AKBLEConnector *connector = [[AKBLEConnector alloc] initWithQueue:nil];

// 1) 开始扫描（示例）
[connector.central scanForPeripheralsWithServices:nil
                                          options:@{ CBCentralManagerScanOptionAllowDuplicatesKey:@YES }];

// 2) 在 didDiscover 里解析出 MAC 后缓存
// [connector cacheDiscoveredPeripheral:peripheral mac:macFromADV];

// 3) 立刻按 MAC 发起“轮询连接”
[connector connectByMAC:@"AA:BB:CC:11:22:33"
                timeout:10
       retriesPerSecond:5
             completion:^(BOOL success, CBPeripheral * _Nullable p, NSError * _Nullable err) {
    if (success) {
        NSLog(@"✅ 已连接：%@", p.name);
    } else {
        NSLog(@"❌ 失败：%@", err.localizedDescription);
    }
}];
```

<br/>

**说明与要点**

* **轮询频率**：`retriesPerSecond=5` → 每 0.2s 一次；你可传 2、10 等自定义频率。
* **连接策略**：

  * 未发现目标外设 → 持续扫描 + 轮询字典；
  * 一旦发现外设 → 发起一次连接；若失败，在超时窗口内允许再次发起；
  * 一旦 `didConnect` 到达 → 立刻**停止扫描与轮询**并回调成功；
  * 超时 → **停止扫描与连接**并回调失败。
* **线程安全**：`peripheralsByMAC` 用 `@synchronized` 简单保护；如果你有更高并发需求可换成自定义并发队列或 `NSLock`。
* **MAC 规范化**：调用 `normalizedMAC:`，确保字典与查询键一致（去符号、转大写）。
* 你现有解析广播报中 MAC 的逻辑保持不变，只需在 `didDiscover` 里调用 `cacheDiscoveredPeripheral:mac:` 即可。


当然上述方法只是具体轮询连接蓝牙外设，实际项目需要结合自己的项目和架构。



<br/><br/><br/>

***
<br/>

> <h1 id="连接蓝牙外设">连接蓝🦷外设</h1>

**🔧 方法定义**

```objc
- (void)connectPeripheral:(CBPeripheral *)peripheral 
                  options:(nullable NSDictionary<NSString *, id> *)options;
```

这是 `CBCentralManager` 的实例方法。它告诉系统：**“请尝试连接我扫描到的这个外设。”**

<br/>

**📌 所属类**

这个方法是 **`CBCentralManager`（中心设备管理器）** 的方法，不是 `CBPeripheral` 自己的。

<br/> 

**🧱 参数详解**

- **1.`peripheral`**
	* 类型：`CBPeripheral *`
	* 意义：你希望连接的蓝牙外设，通常来源于 `scanForPeripheralsWithServices:` 的扫描回调：

```objc
- (void)centralManager:(CBCentralManager *)central 
 didDiscoverPeripheral:(CBPeripheral *)peripheral 
     advertisementData:(NSDictionary<NSString *, id> *)advertisementData 
                  RSSI:(NSNumber *)RSSI
```

你需要保存这个 `peripheral` 并传入连接方法中。

<br/> 

- **2.`options`**
	* 类型：`NSDictionary<NSString *, id> *`（可选）
	* 意义：连接时的附加参数，常用的键包括：

| 键名（NSString）                                          | 意义                                 | 推荐用法     |
| ----------------------------------------------------- | ---------------------------------- | -------- |
| `CBConnectPeripheralOptionNotifyOnConnectionKey`      | 是否在后台连接成功时发本地通知（`NSNumber`，YES/NO） | 一般设为 YES |
| `CBConnectPeripheralOptionNotifyOnDisconnectionKey`   | 断开连接是否通知                           | 一般设为 YES |
| `CBConnectPeripheralOptionNotifyOnNotificationKey`    | 收到通知是否提醒                           | 少用       |
| `CBConnectPeripheralOptionEnableTransportBridgingKey` | iOS/macOS 之间桥接（很少用）                | -        |

<br/>

示例：

```objc
NSDictionary *options = @{
    CBConnectPeripheralOptionNotifyOnConnectionKey: @YES,
    CBConnectPeripheralOptionNotifyOnDisconnectionKey: @YES
};
[centralManager connectPeripheral:peripheral options:options];
```

<br/>

**🔄 连接结果回调**

**✅ 连接成功：**

```objc
- (void)centralManager:(CBCentralManager *)central 
  didConnectPeripheral:(CBPeripheral *)peripheral {
    NSLog(@"连接成功: %@", peripheral.name);
    
    // 必须设置 delegate，否则收不到后续回调
    peripheral.delegate = self;
    
    // 发起服务发现
    [peripheral discoverServices:nil];
}
```

<br/> 

**❌ 连接失败：**

```objc
- (void)centralManager:(CBCentralManager *)central 
didFailToConnectPeripheral:(CBPeripheral *)peripheral 
                 error:(NSError *)error {
    NSLog(@"连接失败: %@，错误: %@", peripheral.name, error);
}
```


<br/> 

**🚫 连接断开（包括手动断开或外设掉线）：**

```objc
- (void)centralManager:(CBCentralManager *)central 
 didDisconnectPeripheral:(CBPeripheral *)peripheral 
                 error:(NSError *)error {
    NSLog(@"已断开连接: %@，错误: %@", peripheral.name, error ?: @"无错误");
}
```

<br/> 

**🧠 总结一句话：**

- `connectPeripheral:options:` 是你主动连接蓝牙设备的入口方法，必须在设备扫描出来后调用，并配合后续的连接、服务发现、特征操作流程一起使用。




***
<br/><br/><br/>
> <h2 id="连接温度计设备">连接温度计设备</h2>

假设你的设备有一个温度服务 `UUID: FFE0`，服务下的特征是温度值 `UUID: FFE1`：

你会在连接成功后这样查找并读取：

```swift
let serviceUUID = CBUUID(string: "FFE0")
let charUUID = CBUUID(string: "FFE1")

peripheral.discoverServices([serviceUUID])

func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
    for service in peripheral.services! {
        if service.uuid == serviceUUID {
            peripheral.discoverCharacteristics([charUUID], for: service)
        }
    }
}

func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {
    for char in service.characteristics! {
        if char.uuid == charUUID {
            peripheral.readValue(for: char)
        }
    }
}
```

---

## 🧠 总结

| 类                  | 作用              | 示例                                            |
| ------------------ | --------------- | --------------------------------------------- |
| `CBPeripheral`     | 表示蓝牙外设          | `peripheral.name`、`discoverServices()`        |
| `CBService`        | 外设提供的功能模块       | `service.uuid`、`service.characteristics`      |
| `CBCharacteristic` | 实际的数据点（可读/写/通知） | `characteristic.value`、`setNotifyValue(true)` |

***

<br/><br/><br/>

> <h2 id="向蓝牙外设发送数据">向蓝牙外设发送数据</h2>

**🔧 方法定义**

```objc
- (void)writeValue:(NSData *)data 
 forCharacteristic:(CBCharacteristic *)characteristic 
              type:(CBCharacteristicWriteType)type;
```

这是 `CBPeripheral` 的一个实例方法，用于将数据写入某个特征（`CBCharacteristic`）中。

该方法是用于 **向蓝牙外设发送数据** 的核心方法之一，是蓝牙通信中实现「**写数据**」操作的关键。

<br/>

🧱 各参数详解

| 参数               | 类型                          | 说明                      |
| ---------------- | --------------------------- | ----------------------- |
| `data`           | `NSData *`                  | 要写入的数据（通常是你想传给外设的命令或指令） |
| `characteristic` | `CBCharacteristic *`        | 目标特征（必须是可写的）            |
| `type`           | `CBCharacteristicWriteType` | 写入类型：是否需要回调确认           |

<br/>

**📌 写入类型说明（`CBCharacteristicWriteType`）**

| 枚举值                                    | 含义              | 特点               |
| -------------------------------------- | --------------- | ---------------- |
| `CBCharacteristicWriteWithResponse`    | 写入并等待外设响应（可靠写入） | 会触发代理回调，适合关键指令   |
| `CBCharacteristicWriteWithoutResponse` | 写入不等待响应（快速写入）   | 没有回调，适合高频数据或实时控制 |

<br/>

**✅ 使用示例（Objective-C）**

```objc
NSData *dataToSend = [@"01FFAA" dataUsingEncoding:NSUTF8StringEncoding];

// 写入数据（带响应）
[peripheral writeValue:dataToSend 
      forCharacteristic:writeCharacteristic 
                   type:CBCharacteristicWriteWithResponse];
```

<br/>

**🎯 写入成功的回调（仅适用于 WithResponse）**

```objc
- (void)peripheral:(CBPeripheral *)peripheral 
didWriteValueForCharacteristic:(CBCharacteristic *)characteristic 
             error:(NSError *)error {
    if (error) {
        NSLog(@"写入失败: %@", error.localizedDescription);
    } else {
        NSLog(@"写入成功到特征: %@", characteristic.UUID);
    }
}
```

<br/>

**❗ 使用注意事项**

| 点                                                         | 说明 |
| --------------------------------------------------------- | -- |
| ✅ 确保特征 `properties` 包含 `.write` 或 `.writeWithoutResponse` |    |
| ❌ 否则调用会失败，不会写入成功                                          |    |
| ⏳ `.withResponse` 写入有回调，适合需要确认是否写入成功的命令（如控制开关）            |    |
| ⚡ `.withoutResponse` 无回调，适合高速频繁数据，如运动轨迹、实时数据上传            |    |

<br/> 

**🧠 特征支持检查示例**

```objc
if (characteristic.properties & CBCharacteristicPropertyWrite) {
    // 支持写入带响应
}

if (characteristic.properties & CBCharacteristicPropertyWriteWithoutResponse) {
    // 支持写入不带响应
}
```

<br/>

**🚀 场景举例**

| 场景            | 建议使用方式                        |
| ------------- | ----------------------------- |
| 开关设备（关键指令）    | `.writeWithResponse`          |
| 发送大批量数据（如传图片） | `.writeWithoutResponse`（结合分包） |
| 实时游戏/手柄控制     | `.writeWithoutResponse`（低延迟）  |


***
<br/><br/><br/>

> <h2 id="接收蓝牙外设发送数据">接收蓝牙外设发送数据</h2>

在接受**蓝牙外设发送数据**的前,你需要先对特征进行订阅,如下:

```objc
if (characteristic.properties & CBCharacteristicPropertyNotify) {
    [peripheral setNotifyValue:YES forCharacteristic:characteristic];
}
```

**✅ 这段代码的核心作用是：**

**订阅这个特征的“通知”（Notify）功能，让外设有数据更新时自动推送给你（App）。**

若是你不订阅,则无法接收到**蓝牙外设**发送到的数据.

<br/>

**🔍 背景知识：BLE 通信的 3 种方式**

**在 CoreBluetooth 中，读取特征值的方式有三种：**

| 方式         | 方法                                                   | 场景              |
| ---------- | ---------------------------------------------------- | --------------- |
| Read       | `[peripheral readValueForCharacteristic:]`           | 主动读取一次数据        |
| Write      | `[peripheral writeValue:forCharacteristic:type:]`    | 主动发送数据到外设       |
| Notify（通知） | `[peripheral setNotifyValue:YES forCharacteristic:]` | 外设有数据就推送给你（自动）✅ |


<br/>

**🧠 那么订阅（notify）有什么用？**

- **📌 1. 实时接收外设数据（最常见）**
- **比如：**
	* 体温计：每 1 秒发送一次温度
	* 心率监测带：每秒发送一次心率
	* 蓝牙秤：量完体重后发送一次数据
	* 蓝牙键盘：按键之后立刻推送数据

你一旦调用了 `setNotifyValue:YES`，一旦外设推送新值，你就会收到：

```objc
- (void)peripheral:(CBPeripheral *)peripheral 
didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic 
             error:(NSError *)error
```

👉 在这个代理方法里读取 characteristic.value，就是最新的数据。

<br/>

**📌 2. 省电节省通信流量**

如果你不订阅，而是用 `readValueForCharacteristic:` 定时轮询，一方面效率低、响应慢，还会耗电。

订阅是**BLE 官方推荐的实时数据传输方式**。

<br/>

**✅ 示例流程**

1. 扫描并连接设备
2. 发现服务和特征
3. 遍历特征，看是否包含 `.notify`
4. 如果支持，则订阅：

```objc
if (characteristic.properties & CBCharacteristicPropertyNotify) {
    [peripheral setNotifyValue:YES forCharacteristic:characteristic];
}
```

<br/>

5.等待接收回调数据：

```objc
- (void)peripheral:(CBPeripheral *)peripheral 
didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic 
             error:(NSError *)error {
    NSData *data = characteristic.value;
    // 解析数据
}
```

<br/> 

- **⚠️ 注意：**
	* 并不是所有特征都支持 Notify，需要通过 `characteristic.properties` 判断是否有 `CBCharacteristicPropertyNotify`。
	* 订阅是建立一个长期的数据推送通道，直到你断开或调用 `setNotifyValue:NO`。



<br/><br/><br/>

***
<br/>

> <h1 id="解析字节数据">解析字节数据</h1>

***
<br/><br/><br/>
> <h2 id="大端和小端区别">大端和小端区别</h2>

- **🧠 简单定义**
	* **大端（Big Endian）**：高位字节存在前面（低地址），低位字节在后面。
	* **小端（Little Endian）**：低位字节存在前面（低地址），高位字节在后面。

---

**✅ 举个例子：数字 `0x1234`**

我们以一个 `UInt16` 数字 `0x1234`（十进制是 `4660`）为例，看它在内存中如何表示：

| 表示方式 | 内存字节顺序（从低地址到高地址） |
| ---- | ---------------- |
| 大端   | `0x12 0x34`      |
| 小端   | `0x34 0x12`      |

---

**🧪 Swift 示例**

```swift
import Foundation

let number: UInt16 = 0x1234

// 转成大端和小端字节序
let bigEndian = number.bigEndian
let littleEndian = number.littleEndian

// 将 UInt16 转成 2 字节的 Data
let bigEndianData = withUnsafeBytes(of: bigEndian) { Data($0) }
let littleEndianData = withUnsafeBytes(of: littleEndian) { Data($0) }

// 打印
print("原始数值: \(number)") // 4660
print("大端字节: \(bigEndianData as NSData)") // 输出: <1234>
print("小端字节: \(littleEndianData as NSData)") // 输出: <3412>
```

---

**🖨️ 输出示例：**

```text
原始数值: 4660
大端字节: <1234>
小端字节: <3412>
```

---

**🧩 补充说明**

| 场景                 | 使用的大/小端       |
| ------------------ | ------------- |
| 网络协议               | **大端（网络字节序）** |
| Intel CPU（x86/x64） | 小端            |
| ARM（如 iPhone）      | 默认小端，但可配置     |

---

**✅ 小结**

| 项目          | 大端          | 小端                |
| ----------- | ----------- | ----------------- |
| 说明          | 高位字节在前      | 低位字节在前            |
| 示例 (0x1234) | `12 34`     | `34 12`           |
| 网络协议        | 常用（“网络字节序”） | 否                 |
| Swift 默认存储  | 小端（与平台一致）   | 小端（在 iOS/macOS 上） |


***
<br/><br/><br/>
> <h2 id="Byte数组构造BLE蓝牙发送指令">Byte数组构造BLE蓝牙发送指令</h2>

 **Byte 数组（`Byte[]`）** 在 iOS（Objective-C）中通常用来处理 **原始的二进制数据**，它有很多实际用途，尤其在以下场景中非常常见：

---

**✅ 常见用途**

1. **构造或解析通信协议**（如蓝牙 BLE、串口、TCP、UDP 协议）
2. **操作二进制文件数据**（图片、音频、压缩包）
3. **与 C 库交互时进行内存操作**
4. **自定义加密、校验算法时处理字节数组**

---

**🎯 举个详细例子：构造 BLE 蓝牙发送指令**

假设你要通过 BLE 发送如下格式的命令数据包给一个硬件设备：

```
[Header][Command][PayloadLength][Payload][Checksum]
```

* Header: 1 字节，固定为 `0xAA`
* Command: 1 字节，比如 `0x01` 表示“请求设备状态”
* PayloadLength: 1 字节，比如 `0x00` 表示没有额外数据
* Payload: 可变，这里为空
* Checksum: 1 字节，前面所有字节相加后的低字节

---

**✅ OC 代码构造这段数据包**

```objc
Byte header = 0xAA;
Byte command = 0x01;
Byte payloadLength = 0x00;

// 计算校验和（这里是简单累加：0xAA + 0x01 + 0x00 = 0xAB）
Byte checksum = (header + command + payloadLength) & 0xFF;

// 构造最终数据数组
Byte packet[] = {header, command, payloadLength, checksum};

// 转成 NSData 用于发送
NSData *dataToSend = [NSData dataWithBytes:packet length:sizeof(packet)];

// 打印结果
NSLog(@"发送数据: %@", dataToSend); // 输出: <aa0100ab>
```

---

**✅ 输出结果**

```
发送数据: <aa0100ab>
```

这正是我们要发给 BLE 外设的原始指令包，包含了协议头、命令、长度、校验等。

---

✅ 小结

| 项目       | 说明                                |
| -------- | --------------------------------- |
| `Byte[]` | 是处理原始二进制的最基础方式                    |
| 应用       | 蓝牙、网络、音频、图片处理、自定义协议               |
| 搭配使用     | 一般与 `NSData`、`NSMutableData` 搭配使用 |
| 优势       | 精确控制每个字节，适用于嵌入式通信或 C 库交互          |


***
<br/><br/><br/>
> <h2 id="自描述式二进制数据包格式">自描述式二进制数据包格式</h2>
如下是一个非常常见的**自描述式二进制数据包格式**，结构如下所示（我们以 ASCII 图示来表达）：

```
| aLen: UInt8 | aData: [UInt8] | bLen: UInt8 | bData: [UInt8] | ...
```

也就是说：

* 每段数据前面 **先跟一个长度字段**（`1 字节 UInt8`），
* 然后跟对应长度的数据内容。
* 重复这个结构。

---

**✅ Swift 完整示例代码**

```swift
import Foundation

func parsePacket(_ packetData: Data) -> [Data] {
    var result: [Data] = []
    var index = 0

    while index < packetData.count {
        // 1. 先取长度字节（UInt8）
        let length = Int(packetData[index])
        index += 1

        // 2. 检查是否还有足够的数据
        guard index + length <= packetData.count else {
            print("❌ 数据长度不足，无法继续解析")
            break
        }

        // 3. 取出这一段数据
        let subData = packetData.subdata(in: index ..< index + length)
        result.append(subData)

        // 4. 移动指针
        index += length
    }

    return result
}
```

---

**✅ 举个例子:**

```swift
// 模拟二进制数据： aLen = 3, a = 0x01 0x02 0x03
//                 bLen = 2, b = 0xAA 0xBB
let rawBytes: [UInt8] = [
    0x03, 0x01, 0x02, 0x03,
    0x02, 0xAA, 0xBB
]
let packetData = Data(rawBytes)

let sections = parsePacket(packetData)

for (i, section) in sections.enumerated() {
    print("Section \(i): \(section as NSData)")
}
```

**输出：**

```
Section 0: <010203>
Section 1: <aabb>
```

***
<br/><br/><br/>
> <h2 id="结构体解包数据">结构体解包数据</h2>


如果你知道你最多有几个字段（例如 a, b, c），你也可以改为返回命名字段：

```swift
struct Packet {
    var aData: UInt8 = 0
    var bData: UInt16 = 0
    var cData: UInt32 = 0
}
```

<br/>

-  **✅ 1.通用解包方法**

```swift
import Foundation

struct Packet {
    var aData: UInt8
    var bData: UInt16
    var cData: UInt32

    init?(from data: Data) {
        guard data.count >= MemoryLayout<UInt8>.size +
                            MemoryLayout<UInt16>.size +
                            MemoryLayout<UInt32>.size else {
            return nil
        }

        var offset = 0

        // 解析 UInt8
        self.aData = data.withUnsafeBytes {
            $0.load(fromByteOffset: offset, as: UInt8.self)
        }
        offset += MemoryLayout<UInt8>.size

        // 解析 UInt16（小端字节序）
        self.bData = data.withUnsafeBytes {
            $0.load(fromByteOffset: offset, as: UInt16.self).littleEndian
        }
        offset += MemoryLayout<UInt16>.size

        // 解析 UInt32（小端字节序）
        self.cData = data.withUnsafeBytes {
            $0.load(fromByteOffset: offset, as: UInt32.self).littleEndian
        }
    }
}
```

---

**🔁 示例用法：**

```swift
let rawBytes: [UInt8] = [
    0x01,                   // aData: UInt8
    0x34, 0x12,             // bData: UInt16 -> 0x1234 = 4660
    0x78, 0x56, 0x34, 0x12  // cData: UInt32 -> 0x12345678 = 305419896
]

let data = Data(rawBytes)

if let packet = Packet(from: data) {
    print("aData = \(packet.aData)") // 1
    print("bData = \(packet.bData)") // 4660
    print("cData = \(packet.cData)") // 305419896
} else {
    print("数据不足或格式错误")
}
```

---

- **📌 注意事项：**

- 1.**字节序问题（Endian）**：
	* 通常 BLE、网络协议中用的是小端（Little Endian），Swift 默认读取的是系统字节序，所以建议调用 `.littleEndian`。
	* 若是大端（Big Endian）则使用 `.bigEndian`。

- 2.**Data 长度校验**：
	* 注意你要先校验 `Data.count` 是否足够再读取。

---

**✅ 可选：更通用的解包器函数**

如果你有多个不同结构体，可以写一个通用读取函数：

```swift
extension Data {
    func read<T>(from offset: inout Int) -> T {
        let value = self.withUnsafeBytes {
            $0.load(fromByteOffset: offset, as: T.self)
        }
        offset += MemoryLayout<T>.size
        return value
    }
}
```

然后在结构体中这样用：

```swift
init?(from data: Data) {
    var offset = 0
    guard data.count >= 7 else { return nil }
    self.aData = data.read(from: &offset)
    self.bData = data.read(from: &offset).littleEndian
    self.cData = data.read(from: &offset).littleEndian
}
```



***
<br/><br/><br/>
> <h2 id="小端字节序还原成一个uint16_t类型的整数">小端字节序还原成一个uint16_t类型的整数</h2>

我们模拟一个包含 4 个字节的 `NSData`，其第 2 和第 3 字节表示一个数字：

```objc
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // 构造一个 NSData 对象，假设内容如下（十六进制）:
        // 索引:   0     1     2     3
        // 数据:  0x00, 0x00, 0x34, 0x12  => 表示数值 0x1234（小端模式）
        Byte bytes[] = {0x00, 0x00, 0x34, 0x12}; 
        NSData *data = [NSData dataWithBytes:bytes length:4];

        // 获取 Byte 指针
        // data.bytes 是返回 NSData 内部的原始字节指针（const void *）
        // (Byte*) 将其强制类型转换为 Byte *（unsigned char *），便于按字节访问
        Byte *dataBytes = (Byte*)data.bytes;

        // 提取小端 16 位整数（从索引 2 和 3）
        // 读取 dataBytes[3] 和 dataBytes[2]，按小端模式组合成 uint16_t
        // dataBytes[2] 是低字节（低位）
        // dataBytes[3] 是高字节（高位）
        // 将高位字节左移 8 位后与低位字节进行按位或，得到 16 位无符号整数
        uint16_t value = (dataBytes[3] << 8) | dataBytes[2];

        // 打印结果
        NSLog(@"value = %d (0x%04X)", value, value);  // 打印十进制和十六进制
    }
    return 0;
}
```

---

**🖨️ 输出结果**

```text
value = 4660 (0x1234)
```

---

📌 总结

| 内容   | 说明                       |                        |
| ---- | ------------------------ | ---------------------- |
| 原始数据 | `0x00 0x00 0x34 0x12`    |                        |
| 读取位置 | 第 2、3 字节：`0x34` 和 `0x12` |                        |
| 组合方式 | \`(0x12 << 8)            | 0x34 = 0x1234 = 4660\` |
| 字节序  | 小端（低位在前）                 |                        |

---

如果你想改成 **大端解析**（高位在前），只需反过来取字节：

```objc
uint16_t value = (dataBytes[2] << 8) | dataBytes[3];  // 大端处理方式
```

---

🔹 问题 1：`dataBytes[3]` 是数组里面的哪个？

数组内容如下（按索引标注）：

| 索引 | 值（十六进制） | 值（十进制） |
| -- | ------- | ------ |
| 0  | `0x00`  | 0      |
| 1  | `0x00`  | 0      |
| 2  | `0x34`  | 52     |
| 3  | `0x12`  | 18     |

所以：

```objc
dataBytes[3] == 0x12  // 即十进制 18
```

---

**🔹 问题 2：`(dataBytes[3] << 8)` 的结果是多少？**

```objc
(dataBytes[3] << 8)
= (0x12 << 8)
= 0x1200
= 4608（十进制）
```

为什么？因为：

* `0x12` 是十进制 18，二进制是 `0001 0010`
* 左移 8 位后变成 `0001 0010 0000 0000`（即 0x1200）

---

**✅ 小结**

| 表达式                 | 说明            | 值（十六进制）         | 值（十进制）   |      |
| ------------------- | ------------- | --------------- | -------- | ---- |
| `dataBytes[3]`      | 第 4 个字节（索引 3） | `0x12`          | 18       |      |
| `dataBytes[3] << 8` | 高位字节左移 8 位    | `0x1200`        | 4608     |      |
| `dataBytes[2]`      | 第 3 个字节（索引 2） | `0x34`          | 52       |      |
| 组合结果                | \`0x1200      | 0x34 = 0x1234\` | `0x1234` | 4660 |


***
<br/><br/><br/>
> <h2 id="">读取一段字节数据范围中的某个字节的某一位bool值</h2>

```swift
/// 从 Data 中读取某字节的某一位(bit)，并返回该位是否为 1
/// - Parameters:
///   - data: 原始数据
///   - byteIndex: 要读取的字节索引（从 0 开始）
///   - bitIndex: 位索引（0 = 最右边的位，7 = 最左边的位）
/// - Returns: Bool 表示该位是否为 1
func isBitSet(in data: Data, byteIndex: Int, bitIndex: Int) -> Bool {
    guard byteIndex < data.count, bitIndex >= 0, bitIndex < 8 else {
        return false // 越界或非法位索引
    }
    
    let byte = data[byteIndex]
    let mask: UInt8 = 1 << bitIndex
    return (byte & mask) != 0
}
```

---

**示例场景**

- **我们假设你要：**
	- 读取 Data 的第 0～2 字节（共 3 字节）；
	- 检查这些字节中 某个 bit 是否为 1（比如第 1 字节的第 4 位）；
	- 可以传入字节索引和位索引来判断某一位是否为 1。

**🧪 示例使用**

```swift
let bytes: [UInt8] = [0b00001111, 0b10100000, 0b01010101] // 共3字节
let data = Data(bytes)

print("第 0 字节，第 0 位是否为 1:", isBitSet(in: data, byteIndex: 0, bitIndex: 0)) // true
print("第 0 字节，第 4 位是否为 1:", isBitSet(in: data, byteIndex: 0, bitIndex: 4)) // false
print("第 1 字节，第 7 位是否为 1:", isBitSet(in: data, byteIndex: 1, bitIndex: 7)) // true
print("第 2 字节，第 1 位是否为 1:", isBitSet(in: data, byteIndex: 2, bitIndex: 1)) // false
```

---

 **🖨️ 输出结果**

```text
第 0 字节，第 0 位是否为 1: true     // 0b00001111 的最低位是 1
第 0 字节，第 4 位是否为 1: false    // 第 5 位是 0
第 1 字节，第 7 位是否为 1: true     // 0b10100000 的最高位是 1
第 2 字节，第 1 位是否为 1: false    // 0b01010101 的第 2 位是 0
```

<br/><br/>

**改进一步:** 让 `isBitSet(...)` 返回 `0 或 1`

```swift
/// 获取某个字节某个 bit 的值（0 或 1）
public func bitValue(byteIndex: Int, bitIndex: Int, data: Data) -> Int {
    guard data.count > byteIndex, bitIndex >= 0, bitIndex < 8 else {
        print("错误：索引越界")
        return 0
    }

    let byte = data[byteIndex]
    return (byte & (1 << bitIndex)) != 0 ? 1 : 0
}
```

***

示例调用:

```swift
let data = Data([0b10101010, 0b11001100, 0xFF, 0x00] + Array(repeating: 0, count: 12))
let analyzer = AKAnalyByteData()

let bits = analyzer.bitsInRange(startByteIndex: 0, endByteIndex: 1, data: data)
print("Bits: \(bits)")  // 输出类似：[0,1,0,1,0,1,0,1,0,0,1,1,0,0,1,1]

let bit3 = analyzer.bitValue(byteIndex: 0, bitIndex: 3, data: data)
print("byte[0] 第3位 bit 值：\(bit3)")  // 输出：1
```


***
<br/><br/><br/>
> <h2 id="字节范围取值">字节范围取值</h2>

- **方法需求:**
	- 从 16 字节的 `Data` 中读取一个连续的字节范围（比如第 0 到第 1 字节）；
	* 然后从这个子范围中，**按位解析**出其中每一位的值（共 16 位）；
	* 返回这些位的布尔数组 `[Bool]`（true 表示 1，false 表示 0）；
	* 方法定义在 `@objcMembers public class AKAnalyByteData: NSObject` 中，以供 Swift 和 Objective-C 都可以调用。

---

**✅ 完整实现（Swift，可供 OC 调用）**

```swift
import Foundation

@objcMembers
public class AKAnalyByteData: NSObject {
    
    /// 判断某字节的某位是否为 1
    public func isBitSet(byteIndex: Int, bitIndex: Int, data: Data) -> Bool {
        guard data.count == 16 else {
            print("错误：输入数据不是 16 字节")
            return false
        }
        guard byteIndex >= 0, byteIndex < 16, bitIndex >= 0, bitIndex < 8 else {
            print("错误：索引越界")
            return false
        }
        
        let byte = data[byteIndex]
        let mask: UInt8 = 1 << bitIndex
        return (byte & mask) != 0
    }
    
    /// 获取某字节范围内所有 bit 的值
    /// - Parameters:
    ///   - startByteIndex: 起始字节索引（包含）
    ///   - endByteIndex: 结束字节索引（包含）
    ///   - data: 输入的 16 字节数据
    /// - Returns: [Bool]，每一位的值（低位在前）
    public func bitsInRange(startByteIndex: Int, endByteIndex: Int, data: Data) -> [Bool] {
        guard data.count == 16 else {
            print("错误：输入数据不是 16 字节")
            return []
        }
        guard startByteIndex >= 0, endByteIndex < 16, startByteIndex <= endByteIndex else {
            print("错误：字节索引无效")
            return []
        }
        
        var bits: [Bool] = []
        for byteIndex in startByteIndex...endByteIndex {
            let byte = data[byteIndex]
            for bitIndex in 0..<8 {
                let isSet = (byte & (1 << bitIndex)) != 0
                bits.append(isSet)
            }
        }
        return bits
    }
}
```

---

**✅ 示例：Swift 中调用**

```swift
let bytes: [UInt8] = [
    0b00001111, // byte[0]
    0b10100000, // byte[1]
    // 剩余14字节填 0
] + Array(repeating: 0, count: 14)

let data = Data(bytes)
let analyzer = AKAnalyByteData()
let bits = analyzer.bitsInRange(startByteIndex: 0, endByteIndex: 1, data: data)

for (index, bit) in bits.enumerated() {
    print("Bit[\(index)] = \(bit ? 1 : 0)")
}
```

---

**🖨️ 输出（按位顺序）**

```text
Bit[0] = 1
Bit[1] = 1
Bit[2] = 1
Bit[3] = 1
Bit[4] = 0
Bit[5] = 0
Bit[6] = 0
Bit[7] = 0
Bit[8] = 0
Bit[9] = 0
Bit[10] = 0
Bit[11] = 0
Bit[12] = 0
Bit[13] = 1
Bit[14] = 0
Bit[15] = 1
```

说明：

* 第 0 字节是 `0b00001111` → 低位到高位依次是：1111 0000；
* 第 1 字节是 `0b10100000` → 低位到高位依次是：0000 0101；

---

**✅ Objective-C 中也可以调用**

虽然 `bitsInRange(...)` 返回的是 `[Bool]`，在 OC 中它会被桥接为 `NSArray<NSNumber *> *`，你可以这样调用：

```objc
AKAnalyByteData *analyzer = [[AKAnalyByteData alloc] init];
NSData *data = ... // 16字节数据
NSArray<NSNumber *> *bitValues = [analyzer bitsInRangeWithStartByteIndex:0 endByteIndex:1 data:data];

for (int i = 0; i < bitValues.count; i++) {
    NSLog(@"Bit[%d] = %@", i, bitValues[i].boolValue ? @"1" : @"0");
}
```

***
<br/><br/>

**修改: 将 [Bool] 改为 [Int]（返回 0 或 1）**

**方法一:**

```
@objcMembers
open class AKAnalyByteData: NSObject {

    /// 提取指定字节范围内所有 bit（以 0/1 表示）
    public func bitsInRange(startByteIndex: Int, endByteIndex: Int, data: Data) -> [Int] {
        guard data.count == 16 else {
            print("错误：输入数据不是 16 字节")
            return []
        }

        guard startByteIndex >= 0, endByteIndex < 16, startByteIndex <= endByteIndex else {
            print("错误：字节索引无效")
            return []
        }

        var bits: [Int] = []

        for byteIndex in startByteIndex...endByteIndex {
            let byte = data[byteIndex]
            for bitIndex in 0..<8 {
                let bit = (byte & (1 << bitIndex)) != 0 ? 1 : 0
                bits.append(bit)
            }
        }

        return bits
    }
}
```



***
<br/><br/><br/>
> <h2 id="判断某一个字节是否为1">判断某一个字节是否为1</h2>

**判断某一字节（`byte`）中第 `bitIndex` 位是否为 1** 的标准写法:

```swift
(byte & (1 << bitIndex)) != 0
```

---

- **✅ 分解解释：**

| 表达式             | 作用                                                                |
| --------------- | ----------------------------------------------------------------- |
| `1 << bitIndex` | 将 `1` 左移 `bitIndex` 位，生成一个掩码。例如 `bitIndex = 3` 时，结果是 `0b00001000` |
| `byte & 掩码`     | 和掩码按位与，仅当 `byte` 中该位为 1，结果才非零                                     |
| `!= 0`          | 最终判断该位是否为 1（true）或 0（false）                                       |

---

**🔍 举个例子**

假设：

```swift
let byte: UInt8 = 0b00001101 // 十进制 13
```

对应二进制：

```
bit7  bit6  bit5  bit4  bit3  bit2  bit1  bit0
 0     0     0     0     1     1     0     1
```

逐位判断：

| bitIndex | 掩码 (1 << bitIndex) | byte & 掩码  | 是否为 1 (`!= 0`) |
| -------- | ------------------ | ---------- | -------------- |
| 0        | `00000001`         | `00000001` | ✅ true         |
| 1        | `00000010`         | `00000000` | ❌ false        |
| 2        | `00000100`         | `00000100` | ✅ true         |
| 3        | `00001000`         | `00001000` | ✅ true         |
| 4\~7     | ...                | `0`        | ❌ false        |

***
<br/><br/><br/>
> <h2 id="">连续Byte某段bit的十进制值</h2>

**🔢 如果你想获取一个连续 bit 段的“十进制整数值”怎么做？**

**✅ 举例说明**

假设你有以下 `Data`（2 字节）：

```swift
let bytes: [UInt8] = [0b10101100, 0b11010010]
let data = Data(bytes)
```

现在你要取从第 0 字节开始，到第 1 字节结束的所有位，共 16 位，然后取其中 **第 4 到第 11 位**，转为十进制整数。

---

**✅ 实现方法（Swift）**

```swift
@objcMembers
open class AKAnalyByteData: NSObject {

    /// 从某段字节范围中提取 bit 范围内的值（返回十进制整数）
    /// - Parameters:
    ///   - startByteIndex: 起始字节（含）
    ///   - endByteIndex: 结束字节（含）
    ///   - startBitIndex: 起始 bit（相对于整个段，例如第 4 位）
    ///   - bitLength: 提取的位数（如 8）
    ///   - data: 输入数据（必须 >= endByteIndex）
    /// - Returns: 十进制整数值
    public func extractBitsAsInt(startByteIndex: Int, endByteIndex: Int, startBitIndex: Int, bitLength: Int, data: Data) -> Int {
        guard data.count >= endByteIndex + 1 else {
            print("错误：数据长度不足")
            return 0
        }

        // 提取指定字节范围所有 bits（低位在前）
        var allBits: [Bool] = []
        for byteIndex in startByteIndex...endByteIndex {
            let byte = data[byteIndex]
            for bitIndex in 0..<8 {
                let isSet = (byte & (1 << bitIndex)) != 0
                allBits.append(isSet)
            }
        }

        // 获取指定 bit 范围
        let bitsToUse = Array(allBits[startBitIndex..<startBitIndex + bitLength])

        // 转成整数（低位在前）
        var value = 0
        for (i, bit) in bitsToUse.enumerated() {
            if bit {
                value |= (1 << i)
            }
        }
        return value
    }
}
```

---

**✅ 示例调用**

```swift
let bytes: [UInt8] = [0b10101100, 0b11010010] // 0xAC, 0xD2
let data = Data(bytes)

let analyzer = AKAnalyByteData()
let result = analyzer.extractBitsAsInt(startByteIndex: 0, endByteIndex: 1, startBitIndex: 4, bitLength: 8, data: data)
print("提取值（十进制）: \(result)")
```

---

**✅ 解析过程（可视化）**

从这 16 个 bit 中：

```
byte[0] = 0b10101100 → bits[0~7]  = [0, 0, 1, 1, 0, 1, 0, 1]
byte[1] = 0b11010010 → bits[8~15] = [0, 1, 0, 0, 1, 1, 0, 1]
                ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
             第4位→bit[4] 到 bit[11]（共8位） = 01101001 → 十进制 105
```

---

**✅ 输出结果：**

```text
提取值（十进制）: 105
```

***

**但是`bit[4]` 到 `bit[11]` 是 `01101001` 是怎么得到的?**

**首先是如何编号的？**

如下2个字节编号:

```byte
byte[0] = 0b10101100
byte[1] = 0b11010010
```

我们按惯例从 **左到右（高位到低位）编号 bit：**

| Byte     | 二进制        | Bit 索引      | 对应 bit 值（从左到右）    |
| -------- | ---------- | ----------- | ----------------- |
| byte\[0] | `10101100` | bit\[0\~7]  | `1 0 1 0 1 1 0 0` |
| byte\[1] | `11010010` | bit\[8\~15] | `1 1 0 1 0 0 1 0` |

***

**bit[4] 到 bit[11] 是哪些值？**

按上面顺序取出 bit[4] 到 bit[11] 共 8 位：

```arduino
bit[4]  = byte[0] 的第5位（从左数） → 1
bit[5]  = byte[0] 的第6位           → 1
bit[6]  = byte[0] 的第7位           → 0
bit[7]  = byte[0] 的第8位           → 0
bit[8]  = byte[1] 的第1位           → 1
bit[9]  = byte[1] 的第2位           → 1
bit[10] = byte[1] 的第3位           → 0
bit[11] = byte[1] 的第4位           → 1
```
<br/>
组合在一起得到:

```
bit[4~11] = 1 1 0 0 1 1 0 1 = 0b11001100 = 105
```


***
<br/><br/><br/>
> <h2 id="16字节转化为16进制的数字符串">16字节转化为16进制的数字符串</h2>

将一个 16 字节的 `Data` 对象转为 16 进制字符串非常简单且常用。


**✅ 方法一：使用 `map` 和 `String(format:)`**

```swift
let data: Data = ... // 16 字节的 Data
let hexString = data.map { String(format: "%02X", $0) }.joined()
print(hexString)
```

---

🔍 示例代码

```swift
let bytes: [UInt8] = [
    0x12, 0x34, 0x56, 0x78,
    0x9A, 0xBC, 0xDE, 0xF0,
    0x11, 0x22, 0x33, 0x44,
    0x55, 0x66, 0x77, 0x88
]
let data = Data(bytes)

let hexString = data.map { String(format: "%02X", $0) }.joined()
print("16字节十六进制字符串：\(hexString)")
```

---

🧾 输出结果

```
16字节十六进制字符串：123456789ABCDEF01122334455667788
```

---

✅ 方法封装（可复用）

你也可以封装成一个通用方法：

```swift
@objcMembers
open class AKAnalyByteData: NSObject {
    
    /// 将 Data 转换为十六进制字符串（大写）
    /// - Parameter data: 输入的 Data
    /// - Returns: 十六进制字符串
    public func hexString(from data: Data) -> String {
        return data.map { String(format: "%02X", $0) }.joined()
    }
}
```

***
<br/><br/><br/>
> <h2 id="蓝牙数据分包">蓝牙数据分包</h2>

在与蓝牙外设（BLE）通信时，**数据包的大小是非常关键的**，因为 BLE 通常有最大传输限制（一般为 20 字节／MTU 为 23 时），而你说的“65 字节数据包（第一字节是长度，后 64 字节是内容）”是一个非常典型的“**自定义分包协议**”。

---

- **🧾 需求分析**
	- 有一段字符串内容（最长 64 字节）。
	- 打包成 65 字节数据：
		- 第 1 字节表示“内容长度”，如：64。
		- 后 64 字节是真正的 UTF-8 编码字符串。
	* 这个数据需要发给蓝牙外设。

---

## 🧠 BLE 通信限制简述

* 通常默认单次 BLE 传输最大 **20 字节**（实际 MTU 为 23，去掉协议开销后可用 20）。
* 如果你要传输 > 20 字节，需要**手动分包**，并且蓝牙外设要支持“连续接收”。

---

## ✅ Swift 中构建 65 字节数据的方法（使用 `Data` 最合适）

```swift
func buildBLEPacket(from string: String) -> Data? {
    // 限制最大长度 64 字节（以 UTF-8 编码计）
    guard let stringData = string.data(using: .utf8), stringData.count <= 64 else {
        return nil
    }

    var packet = Data()
    
    // 第一个字节：长度
    packet.append(UInt8(stringData.count))
    
    // 后续字节：字符串数据
    packet.append(stringData)
    
    // 如果不足 64 字节，补 0
    if stringData.count < 64 {
        let padding = Data(repeating: 0, count: 64 - stringData.count)
        packet.append(padding)
    }

    return packet
}
```

📦 这样你构造出了一个固定 **65 字节** 的数据包：

* `[长度(1 byte)] + [字符串内容(≤64 bytes)]`
* 字符串内容不足 64 字节时，用 `0x00` 补齐

---

**🔁 BLE 分包发送（每次 20 字节）**

你可以这样将这个 Data 按 20 字节一段发送出去：

```swift
func sendBLEPacket(_ packet: Data, write: (Data) -> Void) {
    let mtu = 20
    var offset = 0

    while offset < packet.count {
        let end = min(offset + mtu, packet.count)
        let chunk = packet.subdata(in: offset..<end)
        write(chunk) // 你的蓝牙 write 方法，如 peripheral.writeValue
        offset = end
    }
}
```


***
<br/><br/><br/>
> <h2 id="多个字节转成UInt类型数字">多个字节转成UInt类型数字</h2>

任意 Data 中按指定范围读取 1~4 个字节，并根据字节序（大端/小端）转换为 UInt 类型。

***

- **✅ 通用字节读取工具`（支持 UInt8 / UInt16 / UInt32）`**

```swift
enum Endian {
    case little
    case big
}

/// 从 Data 中读取整数（支持 1～4 字节，小端或大端）
/// - Parameters:
///   - data: 原始 Data
///   - range: 要读取的字节范围（如 0..<2）
///   - endian: 字节序（默认小端）
/// - Returns: UInt 值（UInt8 / UInt16 / UInt32）
func readUInt(from data: Data, range: Range<Int>, endian: Endian = .little) -> UInt {
    let subdata = data[range]
    
    guard (1...4).contains(subdata.count) else {
        fatalError("只支持 1~4 字节读取")
    }
    
    var value: UInt = 0
    let bytes = Array(subdata)
    
    switch endian {
    case .little:
        for (i, byte) in bytes.enumerated() {
            value |= UInt(byte) << (8 * i)
        }
    case .big:
        for (i, byte) in bytes.reversed().enumerated() {
            value |= UInt(byte) << (8 * i)
        }
    }
    
    return value
}
```

***

- **✅ 使用示例：**

```sh
let data = Data([0x78, 0x56, 0x34, 0x12])

// 读取第0~1字节，小端 → UInt16
let val1 = readUInt(from: data, range: 0..<2, endian: .little)
print("前2字节（小端） UInt16: \(val1)")  // 输出：22136

// 读取第2~4字节，大端 → UInt24（转为 UInt）
let val2 = readUInt(from: data, range: 1..<4, endian: .big)
print("第1~3字节（大端） UInt24: \(val2)")  // 输出：5649428

// 读取全部4字节，小端 → UInt32
let val3 = readUInt(from: data, range: 0..<4)
print("全部4字节（小端） UInt32: \(val3)")  // 输出：305419896
```

***

- **✅ 支持范围：**

| 字节数 | 返回类型       | 实际 Swift 类型      |
| --- | ---------- | ---------------- |
| 1   | `UInt8`    | `UInt`（<= 255）   |
| 2   | `UInt16`   | `UInt`（<= 65535） |
| 3   | `UInt24`模拟 | `UInt`（<= 16M）   |
| 4   | `UInt32`   | `UInt`（<= 4G）    |

> ❗如果你需要严格区分返回类型（如必须是 `UInt16`、`UInt32`），可以再加一个泛型版本。

***

**如下更通用的方式:**

```swift
let valueA = readUInt(from: data, range: 0..<2)     // 第1、2字节
let valueB = readUInt(from: data, range: 2..<3)     // 第3字节
let valueC = readUInt(from: data, range: 3..<4)     // 第4字节
let valueD = readUInt(from: data, range: 9..<13)    // 第10~13字节

```

**输出为二进制 / 十六进制 / 十进制：**

```swift
print("十进制: \(valueA)")
print("十六进制: \(String(valueA, radix: 16))")
print("二进制: \(String(valueA, radix: 2))")
```

<br/><br/><br/>

***
<br/>
> <h1 id="密钥">密钥</h1>

***
<br/><br/><br/>
> <h2 id="生成共享密钥对">生成共享密钥对</h2>

你想在 Objective-C 中调用 `mbedtls_ecc_shared_secret_compute`，它的签名是：

```c
int ecc_shared_secret_pair_key(void *p_ctx, uint8_t const *p_sk, uint8_t const *p_pk, uint8_t *p_secret);
```

而你已经在 Swift 中拿到了 `privateKeyData` 和 `devicePublicKey`，它们是 `Data` 类型，你希望在 Objective-C 的类方法中传入这些 `NSData *` 类型参数，最终返回一个 `NSData *` 共享密钥。

下面是一个完整且可行的 **Objective-C 方法实现**：

---

### ✅ Objective-C 方法实现

```objc
+ (NSData *)createSharedKeyPairWithPrivateKey:(NSData *)privateKeyData
                                devicePublicKey:(NSData *)devicePublicKey {

    // 1. 准备参数指针
    const uint8_t *p_sk = (const uint8_t *)privateKeyData.bytes;
    const uint8_t *p_pk = (const uint8_t *)devicePublicKey.bytes;

    // 2. 创建 buffer 用于接收共享密钥，长度取决于你使用的 ECC 曲线（通常 32 字节）
    uint8_t sharedSecret[32] = {0};  // 假设使用的是 secp256r1 曲线，输出为 256bit（32字节）

    // 3. 上下文指针（可为 NULL，或实际上下文）
    void *ctx = NULL;

    // 4. 调用共享密钥计算函数
    int ret = ecc_shared_secret_pair_key(ctx, p_sk, p_pk, sharedSecret);

    if (ret != 0) {
        NSLog(@"计算共享密钥失败，错误码: %d", ret);
        return nil;
    }

    // 5. 转换为 NSData 并返回
    return [NSData dataWithBytes:sharedSecret length:sizeof(sharedSecret)];
}
```

---

### ✅ Swift 调用方式

如果你已经用 Swift 拿到了私钥、公钥：

```swift
let sharedKey = YourClass.createSharedKeyPairWithPrivateKey(withPrivateKey: privateKey, devicePublicKey: devicePubKey)
```

确保你在 `@objc` 的类中公开了这个方法，方法定义写成：

```objc
// privateKeyData: app自己使用椭圆曲线函数生成的公钥、私钥中的私钥数据
// devicePublicKey: 蓝牙外设发来的公钥数据
// 返回的是2者生成的共享密钥
+ (NSData *)createSharedKeyPairWithPrivateKey:(NSData *)privateKeyData devicePublicKey:(NSData *)devicePublicKey;
```

---

### 🔐 提示：

* 如果你用的是其他椭圆曲线（如 Curve25519），共享密钥可能是 32、48 字节甚至更大，请根据实际情况调整 `sharedSecret` 长度。
* `p_ctx` 通常是用于自定义 ECC 实现的上下文，如果你不使用它，可以安全传 `NULL`。

---

如果你需要对 Swift/ObjC 互调方式、数据格式（Data to NSData）也封装成 Swift-friendly 接口，我也可以帮你完成 👍


***
<br/><br/><br/>
> <h2 id="蓝牙外设写入数据调度器">蓝牙外设写入数据调度器</h2>

下面给一个 **BLE 写入队列调度器** 的 **Objective-C** 版本示例，包含：

1. **AKBLEWriteItem** — 写入请求模型
2. **AKBLEWriteQueue** — 写入队列调度器
3. 示例用法（ViewController 里调用）

> ⚠️ 仅演示核心思路，生产代码可再按需要做日志／错误码封装、线程安全加强等。

<br/>

**1️⃣  AKBLEWriteItem.h / .m**

```objc
//  AKBLEWriteItem.h
#import <Foundation/Foundation.h>
#import <CoreBluetooth/CoreBluetooth.h>

typedef void (^AKBLEWriteCompletion)(NSError * _Nullable error);

@interface AKBLEWriteItem : NSObject
@property (nonatomic, strong) NSData              *data;
@property (nonatomic, strong) CBCharacteristic    *characteristic;
@property (nonatomic)         CBCharacteristicWriteType type;
@property (nonatomic)         NSUInteger          retryCount;   // 剩余重试次数
@property (nonatomic, copy)   AKBLEWriteCompletion completion;
+ (instancetype)itemWithData:(NSData *)data
               characteristic:(CBCharacteristic *)charac
                        type:(CBCharacteristicWriteType)type
                  retryCount:(NSUInteger)retry
                  completion:(AKBLEWriteCompletion)block;
@end
```

<br/>

```objc
//  AKBLEWriteItem.m
#import "AKBLEWriteItem.h"
@implementation AKBLEWriteItem
+ (instancetype)itemWithData:(NSData *)data
               characteristic:(CBCharacteristic *)charac
                        type:(CBCharacteristicWriteType)type
                  retryCount:(NSUInteger)retry
                  completion:(AKBLEWriteCompletion)block
{
    AKBLEWriteItem *item = [AKBLEWriteItem new];
    item.data           = data;
    item.characteristic = charac;
    item.type           = type;
    item.retryCount     = retry;
    item.completion     = block;
    return item;
}
@end
```

<br/>

**2️⃣  AKBLEWriteQueue.h / .m**

```objc
//  AKBLEWriteQueue.h
#import <Foundation/Foundation.h>
#import <CoreBluetooth/CoreBluetooth.h>
#import "AKBLEWriteItem.h"

@interface AKBLEWriteQueue : NSObject <CBPeripheralDelegate>
@property (nonatomic, weak,   readonly) CBPeripheral *peripheral;
+ (instancetype)queueWithPeripheral:(CBPeripheral *)peripheral;
- (void)enqueueWrite:(AKBLEWriteItem *)item;
- (void)clearQueue;          // 可手动清空
@end
```

```objc
//  AKBLEWriteQueue.m
#import "AKBLEWriteQueue.h"

@interface AKBLEWriteQueue ()
@property (nonatomic, weak)   CBPeripheral   *peripheral;
@property (nonatomic, strong) NSMutableArray<AKBLEWriteItem *> *queue;
@property (nonatomic, assign) BOOL              busy;
@property (nonatomic, strong) dispatch_queue_t  syncQ;   // 串行同步队列
@end

@implementation AKBLEWriteQueue

+ (instancetype)queueWithPeripheral:(CBPeripheral *)peripheral {
    AKBLEWriteQueue *q  = [AKBLEWriteQueue new];
    q->_peripheral      = peripheral;
    q->_queue           = [NSMutableArray array];
    q->_syncQ           = dispatch_queue_create("ak.ble.writequeue", DISPATCH_QUEUE_SERIAL);
    peripheral.delegate = q;      // ⚠️ 若已有 delegate，需要转发
    return q;
}

- (void)enqueueWrite:(AKBLEWriteItem *)item {
    dispatch_async(self.syncQ, ^{
        [self.queue addObject:item];
        [self tryWriteNext];
    });
}

- (void)clearQueue {
    dispatch_async(self.syncQ, ^{
        [self.queue removeAllObjects];
        self.busy = NO;
    });
}

#pragma mark - Internal
- (void)tryWriteNext {
    if (self.busy || self.queue.count == 0) return;
    self.busy = YES;
    AKBLEWriteItem *item = self.queue.firstObject;
    [self.peripheral writeValue:item.data
               forCharacteristic:item.characteristic
                            type:item.type];
    // 无响应写需人为 call finish
    if (item.type == CBCharacteristicWriteWithoutResponse) {
        [self finishWriteWithError:nil];
    }
}

- (void)finishWriteWithError:(NSError *)err {
    AKBLEWriteItem *item = self.queue.firstObject;
    if (!item) { self.busy = NO; return; }
    
    if (err && item.retryCount > 0) {          // 失败且可重试
        item.retryCount--;
        // 简易重试：0.2 秒后重写
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.2 * NSEC_PER_SEC)), self.syncQ, ^{
            self.busy = NO;
            [self tryWriteNext];
        });
        return;
    }
    // 完成或放弃
    if (item.completion) item.completion(err);
    [self.queue removeObjectAtIndex:0];
    self.busy = NO;
    [self tryWriteNext];                       // 继续下一条
}

#pragma mark - CBPeripheralDelegate
- (void)peripheral:(CBPeripheral *)peripheral
didWriteValueForCharacteristic:(CBCharacteristic *)characteristic
             error:(NSError *)error
{
    dispatch_async(self.syncQ, ^{
        [self finishWriteWithError:error];
    });
}
@end
```

<br/>

**3️⃣  使用示例（ViewController）**

```objc
// 假设 self.peripheral 已连接 & self.writeChar 已发现
self.writeQueue = [AKBLEWriteQueue queueWithPeripheral:self.peripheral];

// 发送一条 带响应 写
NSData *data = [NSData dataWithBytes:(uint8_t[]){0x01,0x02} length:2];
AKBLEWriteItem *item = [AKBLEWriteItem itemWithData:data
                                      characteristic:self.writeChar
                                               type:CBCharacteristicWriteWithResponse
                                         retryCount:2
                                         completion:^(NSError * _Nullable err) {
    if (err) {
        NSLog(@"❌ 写入失败 %@", err);
    } else {
        NSLog(@"✅ 写入成功");
    }
}];
[self.writeQueue enqueueWrite:item];

// 无响应写
AKBLEWriteItem *item2 = [AKBLEWriteItem itemWithData:data
                                       characteristic:self.writeChar
                                                type:CBCharacteristicWriteWithoutResponse
                                          retryCount:0
                                          completion:nil];
[self.writeQueue enqueueWrite:item2];
```

<br/>

**✨ 关键点回顾**

| 功能             | 实现要点                                       |
| -------------- | ------------------------------------------ |
| **串行队列**       | `dispatch_queue_t syncQ` 保证线程安全            |
| **busy 标记**    | 同时只发 1 条写请求                                |
| **带响应写**       | 等 `didWriteValueForCharacteristic` 回调再发下一条 |
| **无响应写**       | 写后立即 `finishWriteWithError:nil`            |
| **重试机制**       | `retryCount` 控制剩余重试次数                      |
| **completion** | 单条写结果回调，成功 `error=nil`                     |

> 生产环境建议再：
> • 把 `delegate` 转发给业务层，避免覆盖；
> • 根据特征属性自动选择 `.type`（前面已讲）；
> • 对长包自动分片。




<br/><br/><br/>

***
<br/>
> <h1 id="调试">调试</h1>

***
<br/><br/><br/>
> <h2 id="MobaXterm在Windows系统上的集成终端工具">MobaXterm在Windows系统上的集成终端工具</h2>

**MobaXterm** 是一款功能强大的终端工具，广泛用于连接远程服务器或设备，**但它本身并不直接支持串口通信（Serial Terminal）**。不过有些用户会通过内置的 `minicom` 或者通过插件方式曲线实现串口通信。

---
<br/>

**🔍 一、MobaXterm 是什么？**

**MobaXterm** 是一个运行在 Windows 系统上的集成终端工具，主要特点包括：

* 支持多种协议：SSH、X11、RDP、VNC、FTP、SFTP、Serial（有限支持）等
* 内置 Linux 环境（Cygwin）
* 多标签终端
* 图形界面友好
* 支持 SSH 登录远程服务器
* 可以执行远程桌面等操作

<br/>

**二、它支持串口终端（Serial Terminal）吗？**

**默认并不强项在串口通信上**，但：

* 可以通过内置的 `minicom` 或 `screen` 等 Linux 工具来进行串口通信（需要一定设置）
* 更多情况下，用户会结合外部串口工具，如 PuTTY 或 TeraTerm 实现串口通信

<br/> 

**🍏 三、Mac 能安装 MobaXterm 吗？**

**不能直接安装**，因为：

* MobaXterm 是专为 **Windows 平台** 开发的应用
* Mac 没有对应的版本
* Mac 上有替代方案：
	* [CoolTerm（推荐串口终端工具）：图形界面，适合与串口设备通信（如 Arduino、嵌入式设备）](https://freeware.the-meiers.org/)
	* **screen** 命令（内建工具）
	* **minicom** 终端命令工具；



***
<br/><br/><br/>
> <h2 id="screen终端工具串口调试">screen终端工具串口调试</h2>

&emsp; 在开发蓝牙外设的时候,可以能与硬件同学进行联调,这个时候可能就需要查看蓝牙外设传给你的数据了.

&emsp; 因为我用的是Mac,所以需要用到Mac已经提供好的终端命令工具了,就是**`screen`**,下面查看下如何进行使用:

***

- **1.查找串口设备名:**

```sh
ls /dev/tty.*
```

***

- **2.使用 `screen` 命令连接串口**

```sh
screen <设备路径> <波特率>
```

比如：

```sh
screen /dev/cu.HC-05-DevB 9600
```

这里 9600 是串口通信的波特率，取决于你的蓝牙模块设置的速率（常见有 9600、115200 等）

注意这里可能遇到屏幕显示乱码问题:这可能是**波特率（Baud Rate）** 不匹配造成的是最常见原因！

&emsp; **蓝牙串口模块（如 HC-05/HC-06）默认波特率**可能是 `9600、38400、115200` 等。

我最开始用的是`‌9600`,但是发现乱码后,就用了`115200‌`,发现是可以正常显示的.

***

- **3.退出 screen**

	- 按下以下键顺序退出：**`Ctrl + A`，然后按 `K`，再按 `Y`  确认退出**

注意：如果你直接关闭终端，可能会导致串口设备被占用，导致之后无法再次连接。



***
<br/><br/><br/>
> <h2 id="minicom终端串口工具">minicom终端串口工具</h2>
---

## ✅ 三、使用 `minicom`（终端串口工具）

### 🧪 安装 minicom（需 Homebrew）：

```bash
brew install minicom
```

---

### 🔌 查看串口设备路径：

```bash
ls /dev/tty.*
```

---

### ⚙️ 初次配置：

```bash
sudo minicom -s
```

进入设置菜单后：

1. 选择 `Serial port setup`
2. 设置：

   * A: `/dev/tty.usbserial-1410`
   * E: `115200` 波特率
3. 保存并退出：选择 `Save setup as dfl`（默认配置）

---

### ▶️ 启动 minicom 串口：

```bash
sudo minicom
```

开始串口通信。

---

### 🔚 退出 minicom：

按下：`Ctrl + A`，然后按 `Z`
再按 `X` 退出 minicom

---

## ✅ 推荐对比总结：

| 工具       | 类型   | 优点          | 适用人群 |
| -------- | ---- | ----------- | ---- |
| screen   | 终端   | 系统自带，无需安装   | 快速测试 |
| CoolTerm | 图形界面 | 直观、易用，适合初学者 | 推荐新手 |
| minicom  | 终端   | 更强功能，支持菜单配置 | 高级用户 |

---

如果你告诉我你使用的是哪种设备，我可以帮你确认设备路径并给出专门命令！也可以给你录屏演示操作方式。是否要继续？


***
<br/><br/><br/>
> <h2 id="图形工具CoolTerm">图形工具CoolTerm</h2>

**📥 安装 CoolTerm：**

- 官网下载页面：
	- 👉 [https://freeware.the-meiers.org/](https://freeware.the-meiers.org/)
- 下载安装后直接运行。

<br/>

- **📡 配置串口通信：**
	- 1.打开 CoolTerm
	- 2.点击 `Options`（选项）
	- 3.在 **Serial Port** 下拉列表中，选择你的串口设备
   （如 `/dev/tty.usbserial-1410`）
	- 4.设置波特率（Baudrate），一般是 `115200`
	- 5.点击 `OK`
	- 6.点击 `Connect` 开始通信

<br/>

-  **🔄 使用说明：**
	* 你可以看到串口打印信息
	* 可以手动发送指令或调试命令
	* 支持自动重连、换行配置等




<br/><br/><br/>

***
<br/>

> <h1 id="硬件自带Wi-Fi">硬件自带Wi-Fi</h1>


***
<br/><br/><br/>
> <h2 id="App加入设备Wi-Fi网络"> App加入设备Wi-Fi网络 </h2>

 **`NEHotspotConfiguration` 与 `NEHotspotConfigurationManager.shared.apply(...)`**

* 用来**在 App 内为设备添加/应用 Wi-Fi 配置**（SSID、密码、EAP 等），由系统弹出同意对话后尝试加入该网络（无需用户手动去设置 App 切换到系统 Wi-Fi 设置）。([Apple Developer][1])

<br/>

- **常用场景**
	* 配置并连接到设备自带的 Wi-Fi（IoT 配件）、在 App 引导下自动加入用户指定热点、把临时热点（`joinOnce = true`）只用于本次连接等。([Apple Developer][1])

<br/>

- **重要属性/方法**
	* `NEHotspotConfiguration(ssid: String, passphrase: String, isWEP: Bool)` — 构造器
	* `config.joinOnce = true` 
		* 只在本次会话使用（系统不持久保存）；`false` 则会保存到系统已知网络中。([Apple Developer][1])
	* `NEHotspotConfigurationManager.shared.apply(config) { error in ... }` 
		* 发起配置并尝试加入。完成处理器的 `error == nil` 并不总是代表“已经完全连上并可通行”，只是表示系统接受了请求 / 没有立即返回错误（实际连接状态仍建议额外验证）。
	* `NEHotspotConfigurationManager.shared.removeConfiguration(forSSID:)` 
		* 删除**本 App**之前添加的配置（不能删除用户/其他 App 添加的）。([Apple Developer][4])

<br/>

**常见错误和处理（示例）**

* `NEHotspotConfigurationError.userDenied`：用户拒绝授权。
* `NEHotspotConfigurationError.alreadyAssociated`：设备已经连接该 SSID。
* `NEHotspotConfigurationError.internal` 等（有时会出现“internal error”或 helper 通信错误，需调试或重启设备）。([Apple Developer][5], [Stack Overflow][6])

**示例（Swift）**

```swift
import NetworkExtension

func connectToWifi(ssid: String, passphrase: String) {
    let config = NEHotspotConfiguration(ssid: ssid, passphrase: passphrase, isWEP: false)
    config.joinOnce = true // 或 false，按需求

    NEHotspotConfigurationManager.shared.apply(config) { error in
        if let error = error as NSError? {
            // NEHotspotConfigurationError.code 可比较 enum rawValue
            if error.domain == NEHotspotConfigurationErrorDomain {
                let code = NEHotspotConfigurationError(rawValue: error.code)
                switch code {
                case .userDenied:
                    print("用户拒绝")
                case .alreadyAssociated:
                    print("已经连接")
                default:
                    print("hotspot 配置错误：", error)
                }
            } else {
                print("其它错误：", error)
            }
        } else {
            print("apply 没有返回错误 —— 系统接受了请求（不代表已完全可用），可进一步验证连接状态。")
            // 建议用下面的 fetchCurrent 或 NWPathMonitor 等方式再次确认当前连接
        }
    }
}
```

**调试建议 / 常见坑**

* 在模拟器不可用：必须在真机测试。
* 即使 `apply` completion 没有错误，也不总是立刻能通信，最好在 `apply` 后用 `NEHotspotNetwork.fetchCurrent` 或 `NWPathMonitor` /实际 socket 测试来确认。

<br/>

**2) `NEHotspotNetwork.fetchCurrent(...)`**

* 在 iOS 14+（以及较新 SDK）中，用来**读取当前 Wi-Fi 网络的信息**（`ssid`, `bssid`, `signalStrength` 等），返回 `NEHotspotNetwork?`。这是替代旧的 CaptiveNetwork/CNCopyCurrentNetworkInfo 的现代 API。([Apple Developer][8])

<br/>

**重要限制（关键）**
`fetchCurrent` **并不总会返回非 nil**；Apple 文档和社区说明：只有当你的 App **满足至少一个特定条件** 时才会返回信息（例如：App 有访问 Wi-Fi 信息的能力，且**请求了精确位置权限**；或 App 是使用 `NEHotspotConfiguration` 配置过当前连接；或设备上有活动的 VPN / NEDNSSettingsManager 等）。也就是说：**需要权限/条件**才能拿到 SSID/BSSID 等敏感 Wi-Fi 信息。

<br/>

**常见满足条件的几项（概括）：**

* App 已在 Xcode 中启用 **Access Wi-Fi Information** entitlement（`com.apple.developer.networking.wifi-info`）。
* App 已取得位置权限（iOS 要求，精确位置通常需要）。
* 或该网络是 App 通过 `NEHotspotConfiguration` 配置过的（或满足其它少数系统条件如活动 VPN）。([Apple Developer][10], [seacode.uk][9])

<br/>

**示例（闭包与 async 包装）**

```swift
NEHotspotNetwork.fetchCurrent { network in
    if let network = network {
        print("SSID:", network.ssid)
        print("BSSID:", network.bssid ?? "nil")
        print("signal:", network.signalStrength) // 0.0..1.0
    } else {
        print("无法获取当前网络信息（可能缺权限或条件不满足）")
    }
}
```

<br/>

用 Swift 的 async/await 包装（方便在 async 环境里用）：

```swift
func fetchCurrentNetwork() async -> NEHotspotNetwork? {
    await withCheckedContinuation { cont in
        NEHotspotNetwork.fetchCurrent { net in
            cont.resume(returning: net)
        }
    }
}
```

**注意**：`signalStrength` 的值是 `0.0..1.0` 的范围（不是 dBm），且很多人报告在某些情形下会得到 0（Apple 的论坛回复也给出类似说明）。


<br/>

**3) `CNCopySupportedInterfaces()` / `CNCopyCurrentNetworkInfo(...)`（CaptiveNetwork）**

* 这是较早的 SystemConfiguration CaptiveNetwork API，用于获取当前连接 Wi-Fi 的 SSID/BSSID（`CNCopySupportedInterfaces()` 列出接口，`CNCopyCurrentNetworkInfo` 获取具体 info）。在 iOS 的早期常用。

<br/>

**现在的状态 & 隐私限制**

* `CNCopyCurrentNetworkInfo` 在 iOS 13 之后对访问做了严格限制（因为 Wi-Fi 信息能泄露位置信息）：要使用它你必须满足某些条件（如：启用了 Access Wi-Fi Information entitlement 并获得位置许可，或满足其它条件），并且从 iOS 14 开始 Apple 推动用 `NEHotspotNetwork.fetchCurrent` 作为替代，`CNCopyCurrentNetworkInfo` 标注为 Deprecated（在文档里可见）。因此**不推荐在新代码里继续依赖该接口**。([Apple Developer][14], [Stack Overflow][15])

<br/>

**示例（仅作历史参考 — 新项目请用 NEHotspotNetwork）**

```swift
import SystemConfiguration.CaptiveNetwork

func currentSSID() -> String? {
    guard let interfaces = CNCopySupportedInterfaces() as? [String] else { return nil }
    for iface in interfaces {
        if let dict = CNCopyCurrentNetworkInfo(iface as CFString) as? [String:AnyObject],
           let ssid = dict[kCNNetworkInfoKeySSID as String] as? String {
            return ssid
        }
    }
    return nil
}
```

**注意**：在 iOS 13+ 这段代码经常返回 nil 来反映系统权限/隐私变化。([Stack Overflow][15])

<br/>

**4) NEHotspotHelper（顺便说明）**

* `NEHotspotHelper` 是另一个更强大的 Wi-Fi 热点辅助 API（能拦截/评估热点列表、在后台处理登录认证等），但**需要向 Apple 申请特殊的 Network Extension Entitlement（`com.apple.developer.networking.HotspotHelper`）并且审批严格**。普通 App 通常拿不到（Apple 会审核使用理由）。如果你只是想连接 Wi-Fi，优先用 `NEHotspotConfiguration`（更简单、无需额外审批）。([Apple Developer][16], [Medium][17])

<br/>

**权限 & Capabilities 一览（实践要点）**

1. **Access Wi-Fi Information**（`com.apple.developer.networking.wifi-info`）

   * 在 Xcode Target → Signing & Capabilities 加 “Access Wi-Fi Information”，或在 entitlements 添加 `com.apple.developer.networking.wifi-info = YES`。没有这个 entitlement，读取 SSID/BSSID 很可能失败。([Apple Developer][10], [Better Programming][18])

2. **Location Permission**

   * 从 iOS 13/14 后访问 Wi-Fi 信息可能要求 App 请求并获得位置授权（通常是当你使用 `CNCopyCurrentNetworkInfo` / `NEHotspotNetwork.fetchCurrent` 时）。请在 Info.plist 提供合适的说明键（如 `NSLocationWhenInUseUsageDescription`、`NSLocationAlwaysAndWhenInUseUsageDescription`），并在运行时请求权限。([seacode.uk][9])

3. **Network Extensions / Hotspot entitlements**

   * `NEHotspotHelper` 需要特殊审批；`NEHotspotConfiguration` 通常只要在 Capabilities 打开 “Hotspot Configuration” 就能工作（但偶有系统/设备 bug 情形）。([Medium][17], [Stack Overflow][6])

4. **真机测试**：上述 Wi-Fi APIs 大多数都 **在模拟器不可用**，必须用真机测试。

<br/>

**推荐实战流程（配置 + 验证）**

1. 在 Xcode 打开 `Signing & Capabilities`：添加 **Hotspot Configuration**（用于 apply）和 **Access Wi-Fi Information**（用于读取 SSID/BSSID）。
2. 请求并确认用户的位置权限（若你需要读取 SSID）。
3. 使用 `NEHotspotConfigurationManager.shared.apply(config)` 发起连接请求，处理 `error`（判断 `NEHotspotConfigurationError`）。
4. `apply` 后不要完全依赖 `error == nil` 为“已连通”，用 `NEHotspotNetwork.fetchCurrent` 或 `NWPathMonitor` / 实际网络请求去确认真实连通性。([Marko Engelman][3], [Apple Developer][8])


<br/><br/>

**`NEHotspotConfiguration`、`NEHotspotConfigurationManager`、`NEHotspotNetwork.fetchCurrent`、`CNCopySupportedInterfaces`** 这些 API 有沙箱限制，部分需要特殊的 **Entitlement（网络扩展权限）**，不是所有 App 都能上架使用。


<br/>

**1.`NEHotspotConfiguration` & `NEHotspotConfigurationManager`**

👉 用来连接指定的 Wi-Fi。

```swift
import NetworkExtension

func connectToWiFi(ssid: String, password: String) {
    let config = NEHotspotConfiguration(ssid: ssid, passphrase: password, isWEP: false)
    config.joinOnce = false  // 设置为 true 表示仅本次加入，false 表示记住 Wi-Fi

    NEHotspotConfigurationManager.shared.apply(config) { error in
        if let error = error {
            if (error as? NEHotspotConfigurationError)?.code == .alreadyAssociated {
                print("已连接到该 Wi-Fi")
            } else {
                print("连接失败：\(error.localizedDescription)")
            }
        } else {
            print("Wi-Fi 连接成功！")
        }
    }
}
```

使用时调用：

```swift
connectToWiFi(ssid: "MyWiFi", password: "12345678")
```

<br/>

**2.`NEHotspotNetwork.fetchCurrent`**

👉 获取当前连接的 Wi-Fi 信息。

```swift
import NetworkExtension

func getCurrentWiFi() {
    NEHotspotNetwork.fetchCurrent { network in
        if let network = network {
            print("当前 Wi-Fi SSID: \(network.ssid)")
            print("信号强度 RSSI: \(network.signalStrength)") // 0.0 ~ 1.0
            print("BSSID: \(network.bssid)")
        } else {
            print("未连接到 Wi-Fi 或无权限获取")
        }
    }
}
```

<br/>

**3.`CNCopySupportedInterfaces`（SystemConfiguration 框架）**

👉 获取设备支持的 Wi-Fi 接口，并查看当前 Wi-Fi 信息。
这个方法比较旧，很多时候只能获取 **SSID 和 BSSID**。

```swift
import SystemConfiguration.CaptiveNetwork

func getWiFiInfo() {
    if let interfaces = CNCopySupportedInterfaces() as? [String] {
        for interface in interfaces {
            if let dict = CNCopyCurrentNetworkInfo(interface as CFString) as? [String: AnyObject] {
                print("接口: \(interface)")
                print("SSID: \(dict[kCNNetworkInfoKeySSID as String] ?? "" as AnyObject)")
                print("BSSID: \(dict[kCNNetworkInfoKeyBSSID as String] ?? "" as AnyObject)")
            }
        }
    } else {
        print("无法获取 Wi-Fi 接口信息")
    }
}
```

<br/>

**总结**

* **`NEHotspotConfiguration`** → 连接 Wi-Fi
* **`NEHotspotConfigurationManager`** → 管理 Wi-Fi 配置（添加/移除）
* **`NEHotspotNetwork.fetchCurrent`** → 获取当前 Wi-Fi 详细信息（信号强度、SSID、BSSID）
* **`CNCopySupportedInterfaces`** → 获取 Wi-Fi SSID/BSSID（老 API，功能有限）


```swift
// MARK: - 定义配网方式
enum DeviceConfigMethod {
    case bluetooth
    case lan
    case softAP
    case ethernet
}

// MARK: - 定义设备基础模型
class Device {
    let id: String
    let productId: String
    var name: String?
    
    init(id: String, productId: String, name: String? = nil) {
        self.id = id
        self.productId = productId
        self.name = name
    }
}
```


<br/><br/><br/>

***
<br/>

> <h1 id= "iot简单配网架构">iot简单配网架构</h1>


// MARK: - 把不同配网方式需要的参数拆开
protocol DeviceConfigRequest {}

struct BluetoothConfigRequest: DeviceConfigRequest {
    let peripheralID: String
}

struct LANConfigRequest: DeviceConfigRequest {
    let ip: String
    let port: UInt16
}

struct SoftAPConfigRequest: DeviceConfigRequest {
    let ssid: String
    let password: String
    let targetWiFiSSID: String
    let targetWiFiPassword: String
}

struct EthernetConfigRequest: DeviceConfigRequest {
    let host: String
    let port: UInt16
}


protocol DeviceConfigStrategy {
    associatedtype Request: DeviceConfigRequest
    
    var method: DeviceConfigMethod { get }
    
    func startConfig(
        for device: Device,
        request: Request
    ) async throws -> DeviceConfigResult
}

struct DeviceConfigResult {
    let deviceID: String
    let success: Bool
    let message: String
}



// MARK: -- 各种方式分别实现
// MARK: -- 蓝牙
final class BluetoothConfigStrategy: DeviceConfigStrategy {
    let method: DeviceConfigMethod = .bluetooth
    
    func startConfig(
        for device: Device,
        request: BluetoothConfigRequest
    ) async throws -> DeviceConfigResult {
        print("开始蓝牙配网: \(device.id), peripheralID: \(request.peripheralID)")
        return DeviceConfigResult(
            deviceID: device.id,
            success: true,
            message: "Bluetooth config success"
        )
    }
}


// MARK: -- LAN
final class LANConfigStrategy: DeviceConfigStrategy {
    let method: DeviceConfigMethod = .lan
    
    func startConfig(
        for device: Device,
        request: LANConfigRequest
    ) async throws -> DeviceConfigResult {
        print("开始局域网配网: \(device.id), ip: \(request.ip)")
        return DeviceConfigResult(
            deviceID: device.id,
            success: true,
            message: "LAN config success"
        )
    }
}



// MAKR： - AP
final class SoftAPConfigStrategy: DeviceConfigStrategy {
    let method: DeviceConfigMethod = .softAP
    
    func startConfig(
        for device: Device,
        request: SoftAPConfigRequest
    ) async throws -> DeviceConfigResult {
        print("开始 SoftAP 配网: \(device.id), AP: \(request.ssid)")
        return DeviceConfigResult(
            deviceID: device.id,
            success: true,
            message: "SoftAP config success"
        )
    }
}

// - 网线
final class EthernetConfigStrategy: DeviceConfigStrategy {
    let method: DeviceConfigMethod = .ethernet
    
    func startConfig(
        for device: Device,
        request: EthernetConfigRequest
    ) async throws -> DeviceConfigResult {
        print("开始网线配网: \(device.id), host: \(request.host)")
        return DeviceConfigResult(
            deviceID: device.id,
            success: true,
            message: "Ethernet config success"
        )
    }
}


// MARK： - 工厂模式：统一创建策略
enum DeviceConfigFactory {
    static func makeBluetoothStrategy() -> BluetoothConfigStrategy {
        BluetoothConfigStrategy()
    }

    static func makeLANStrategy() -> LANConfigStrategy {
        LANConfigStrategy()
    }

    static func makeSoftAPStrategy() -> SoftAPConfigStrategy {
        SoftAPConfigStrategy()
    }

    static func makeEthernetStrategy() -> EthernetConfigStrategy {
        EthernetConfigStrategy()
    }
}


//final class BluetoothConfigStrategy {
//    private let bleManager: BLEManager
//    private let logger: Logger
//
//    init(bleManager: BLEManager, logger: Logger) {
//        self.bleManager = bleManager
//        self.logger = logger
//    }
//}

// 再往上包一层 Facade，给业务侧更好用的入口
final class DeviceConfigurator {

    func configBluetooth(
        device: Device,
        request: BluetoothConfigRequest
    ) async throws -> DeviceConfigResult {
        let strategy = DeviceConfigFactory.makeBluetoothStrategy()
        return try await strategy.startConfig(for: device, request: request)
    }

    func configLAN(
        device: Device,
        request: LANConfigRequest
    ) async throws -> DeviceConfigResult {
        let strategy = DeviceConfigFactory.makeLANStrategy()
        return try await strategy.startConfig(for: device, request: request)
    }

    func configSoftAP(
        device: Device,
        request: SoftAPConfigRequest
    ) async throws -> DeviceConfigResult {
        let strategy = DeviceConfigFactory.makeSoftAPStrategy()
        return try await strategy.startConfig(for: device, request: request)
    }

    func configEthernet(
        device: Device,
        request: EthernetConfigRequest
    ) async throws -> DeviceConfigResult {
        let strategy = DeviceConfigFactory.makeEthernetStrategy()
        return try await strategy.startConfig(for: device, request: request)
    }
}

// 调用Demo
class TestClass {
    func testaa() async {
        let device = Device(id: "123", productId: "ABC")
        let request = SoftAPConfigRequest(
            ssid: "Device_AP",
            password: "12345678",
            targetWiFiSSID: "HomeWiFi",
            targetWiFiPassword: "home_password"
        )

        let configurator = DeviceConfigurator()
        let result = try? await configurator.configSoftAP(device: device, request: request)
        print(result?.message)
    }
    
    func testSoftAPConfigSuccess() async throws {
        let strategy = SoftAPConfigStrategy()
        let device = Device(id: "123", productId: "ABC")
        let request = SoftAPConfigRequest(
            ssid: "Device_AP",
            password: "12345678",
            targetWiFiSSID: "HomeWiFi",
            targetWiFiPassword: "11111111"
        )

        let result = try await strategy.startConfig(for: device, request: request)

//        XCTAssertTrue(result.success)
    }
}

```











