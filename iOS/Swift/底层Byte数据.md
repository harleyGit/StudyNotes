> <h1 id=""></h1>
- [**3个底层字节处理数据类**](#3个底层字节处理数据类)
	- [开发蓝牙数据工具选择](#开发蓝牙数据工具选择)
	- [UnsafeMutableRawPointer和Data的关系](#UnsafeMutableRawPointer和Data的关系)
- [**类型Data**](#类型Data)
	- [不同字节数的数字拼接](#不同字节数的数字拼接)
		- [多个字节拼接](#多个字节拼接)
		- [`Data(repeating:count:)`用法详解](#`Data(repeating:count:)`用法详解)
	- [自描述式二进制数据包格式拆包解析](#自描述式二进制数据包格式拆包解析)
	- [传入的值为十进制数转成不同字节数值](#传入的值为十进制数转成不同字节数值)
	- [将十进制64存储为4字节](#将十进制64存储为4字节)
	- [固定范围字节转化成UInt类型](#固定范围字节转化成UInt类型)
	- [SHA256字节签名](#SHA256字节签名)
	- [byte以16进制打印](#byte以16进制打印)
- [**类型UInt8**](#类型UInt8)
	- [数组类型截取二进制data数据](#数组类型截取二进制data数据)
	- [类型UInt64变量转化为UInt8如何做?](#类型UInt64变量转化为UInt8如何做?)
		- [将4字节写入Data](#将4字节写入Data)
		- [`withUnsafeBytes(of:)` 方法详解](#`withUnsafeBytes(of:)`方法详解)
- [结构体进行拼包](#结构体进行拼包)
- [UInt32转成Data类型](#UInt32转成Data类型)
- [移位详解](#移位详解)
	- [UInt32类型的10进制时间戳需要移位吗？](#UInt32类型的10进制时间戳需要移位吗？)
- [Uint16、UInt32、UInt64转成Data类型](#Uint16、UInt32、UInt64转成Data类型)
- [Data在Int、字符串、结构体、json、图片、音频文件转化](#Data在Int、字符串、结构体、json、图片、音频文件转化)
	- [结构体进行封装字节数据并拼接](#结构体进行封装字节数据并拼接)
- **资料**
	- [Swift内存布局与指针操作](https://www.cnblogs.com/zbblog/p/15125741.html)
	- [Swift操作字节数据的方法](https://www.cnblogs.com/jamiechoo/articles/18448765)
	- [swift Data数据转换成其它数据](https://www.cnblogs.com/jamiechoo/articles/18548825)
	- [玩转socket之字节流操作--拼包、拆包](https://juejin.cn/post/6844903854077640711#heading-18)



<br/><br/><br/>

***
<br/>

> <h1 id="3个底层字节处理数据类">3个底层字节处理数据类</h1>

在 Swift 中，`UnsafeMutableRawPointer`、`storeBytes(of:as:)` 和 `Data` 都是处理底层数据（特别是字节数据）时的重要工具。它们之间有一定的关系，但各自用途和抽象层次不同。

---

**🔧 一、三者概览**

| 工具                        | 作用               | 抽象层级 | 是否安全          |
| ------------------------- | ---------------- | ---- | ------------- |
| `Data`                    | 存储任意字节数据的安全、高层封装 | 高    | ✅ 安全          |
| `UnsafeMutableRawPointer` | 指向原始内存地址的指针（无类型） | 低    | ❌ 不安全（需要小心使用） |
| `storeBytes(of:as:)`      | 向原始内存写入某个值的字节表示  | 低    | ❌ 不安全，和指针一起使用 |

---

**🧠 二、各自用途**

- **1.`Data`**
	* 是 Swift 提供的 **安全字节缓冲区**。
	* 适合用于网络、文件操作、加密等需要处理二进制数据的场景。
	* 支持很多便捷的 API，如 `append(_:)`、`subdata(in:)`、`withUnsafeBytes {}` 等。

```swift
let number: UInt32 = 42
var data = Data()
data.append(contentsOf: withUnsafeBytes(of: number.littleEndian, Array.init))
// 存储 4 个字节进 Data
```

---
<br/>

**2.`UnsafeMutableRawPointer`**

* 指向未绑定类型的原始内存（可以是任意类型的数据）。
* 通常在进行 **高性能内存操作** 或与 C 接口交互时使用。
* 需要自己管理内存分配、释放、对齐、生命周期等，容易出错。

```swift
let size = MemoryLayout<UInt32>.size
let pointer = UnsafeMutableRawPointer.allocate(byteCount: size, alignment: 4)
defer { pointer.deallocate() }

pointer.storeBytes(of: 42, as: UInt32.self)
let value = pointer.load(as: UInt32.self)
print(value)  // 42
```

---
<br/>

- **3.`storeBytes(of:as:)`**
	* 是 `UnsafeMutableRawPointer` 的方法。
	* 用于把某个值的字节 **按指定类型** 存储进指针指向的内存。
	* 常用于和裸内存打交道时的写操作。

```swift
pointer.storeBytes(of: myValue, as: MyType.self)
```

---

## 🔗 三者关系与配合

它们可以结合使用：

```swift
let value: UInt32 = 12345

// 1. 用 UnsafeMutableRawPointer 写入字节
let pointer = UnsafeMutableRawPointer.allocate(byteCount: 4, alignment: 4)
pointer.storeBytes(of: value, as: UInt32.self)

// 2. 将 pointer 内容封装成 Data
let data = Data(bytes: pointer, count: 4)

// 3. 读取 Data 内容
let readValue = data.withUnsafeBytes {
    $0.load(as: UInt32.self)
}
print(readValue)  // 12345

pointer.deallocate()
```

---

- **🧩 总结类比（通俗理解）**
	* `Data` 就像是安全的 **字节容器（数组）**，你可以放心存取；
	* `UnsafeMutableRawPointer` 是原始的 **裸内存指针**，要自己处理一切；
	* `storeBytes(of:as:)` 是一种 **原始写入工具**，配合指针将 Swift 值变成字节流写入内存。

***
<br/><br/><br/>
> <h2 id="开发蓝牙数据工具选择">开发蓝牙数据工具选择</h2>

蓝牙开发中，你确实会频繁处理原始字节数据（尤其是 GATT 通信中通过特征发送和接收的数据）。在这种场景下：

---

**✅ 建议使用工具选择**

| 场景                     | 推荐工具                                                  | 原因         |
| ---------------------- | ----------------------------------------------------- | ---------- |
| 拼装/解析蓝牙数据包             | `Data` + `withUnsafeBytes` / `withUnsafeMutableBytes` | 安全高效，易于处理  |
| 构造复杂二进制结构（如命令包、校验码）    | `Data` + `append(contentsOf:)`                        | 类似字节数组操作   |
| 从结构体或整数写入字节            | `withUnsafeBytes(of:)` → 转为 `[UInt8]` 或 `Data`        | 安全封装原始值    |
| 高性能连续内存写入（如组合多个字段到特征值） | 可选 `UnsafeMutableRawPointer`，但建议避免除非性能瓶颈              | 容易出错，不推荐常用 |
| 与 C API（如外部蓝牙库）交互      | 可能需要 `UnsafeMutableRawPointer`                        | 需要用指针传递内存  |

---

**📦 实用示例：拼装蓝牙数据包（如写入 64 字节公钥）**

```swift
func constructBLEPacket(publicKey: [UInt8]) -> [Data] {
    var packets: [Data] = []
    let chunkSize = 20
    let totalChunks = Int(ceil(Double(publicKey.count) / Double(chunkSize)))

    for i in 0..<totalChunks {
        let start = i * chunkSize
        let end = min(start + chunkSize, publicKey.count)
        let chunk = publicKey[start..<end]

        var packet = Data()
        packet.append(UInt8(i))  // 序号字节
        packet.append(contentsOf: chunk)
        packets.append(packet)
    }

    return packets
}
```

---

 **🔄 实用示例：解析蓝牙接收数据（解析特征值）**

```swift
func parseBLEData(_ data: Data) {
    guard data.count >= 1 else { return }
    let sequenceNumber = data[0]
    let payload = data.dropFirst()

    print("序号：\(sequenceNumber)，有效负载：\(payload.map { String(format: "%02x", $0) })")
}
```

---

**🔍 小技巧：结构体转 Data**

如果你需要将某个结构体打包成字节发出：

```swift
struct Command {
    var id: UInt8
    var value: UInt16
}

var command = Command(id: 0x01, value: 0x1234)
let data = withUnsafeBytes(of: &command) { Data($0) }
// 注意字节序！
```

如果需要特定字节序（大端/小端），建议使用 `littleEndian` 或 `bigEndian` 明确表示。

---

**✅ 小结建议（蓝牙方向）：**

* **优先用 `Data` 来拼装和解析字节数据**；
* **使用 `withUnsafeBytes(of:)` 将整数、结构体转成 Data**；
* **尽量避免 `UnsafeMutableRawPointer`，除非你真的需要裸内存操作（如跨语言调用 C）**。

***
<br/><br/><br/>
> <h2 id="UnsafeMutableRawPointer和Data的关系">UnsafeMutableRawPointer和Data的关系</h2>

它们是 Swift 中都常用的数据处理工具：🔗 `UnsafeMutableRawPointer` 和 `Data` 

它们的关系在于： **`Data` 是对底层内存字节的封装，而 `UnsafeMutableRawPointer` 是直接操作内存的方式**。

***

**✅ 1. `Data` 是一个字节缓冲区（byte buffer）**

```swift
var data = Data()
data.append(contentsOf: [0x01, 0x02, 0x03, 0x04])
```

这就是在顺序存储 4 个字节，Data 是连续内存的封装，你可以认为它就像是一个 **`UInt8`** 数组。

---

**✅ 2. `UnsafeMutableRawPointer` 是直接控制内存的工具**

你用的是：

```swift
let ptr = malloc(24) // 申请 24 字节内存
ptr?.storeBytes(of: 100, as: Int.self)
ptr.storeBytes(of: 200, toByteOffset: 8, as: Int.self)
ptr.storeBytes(of: 300, toByteOffset: 16, as: Int.self)

let first = ptr.load(fromByteOffset: 0, as: Int.self)     // 100
let second = ptr.load(fromByteOffset: 8, as: Int.self)    // 200
let third = ptr.load(fromByteOffset: 16, as: Int.self)    // 300

print(first, second, third)
```

输出:

```sh
100 200 300
```

这操作的是一段 **原始裸内存（raw memory）**，你手动控制内存内容。

---

**🤝 它们如何互相转换？**

**✅ 从 `UnsafeMutableRawPointer` 创建 `Data`**

你可以把这段内存“封装”为一个 `Data`：

```swift
let data = Data(bytes: ptr!, count: 24)
```

这就表示：从 `ptr` 开始，读取连续的 24 个字节作为 Data。

你前面写入了 3 个 Int（100、200、300），每个占 8 字节，所以这段 `Data` 实际上是这三个 Int 的二进制拼接。

---

### ✅ 从 `Data` 读取为指定类型

```swift
let value = data.withUnsafeBytes { rawBuffer in
    rawBuffer.load(fromByteOffset: 8, as: Int.self) // 从偏移8字节位置读取 Int（即 200）
}
```

这个就和你在指针里：

```swift
ptr?.storeBytes(of: 200, toByteOffset: 8, as: Int.self)
```

是一样的道理。`Data` 其实只是封装后的内存，逻辑上等价于你操作的裸指针。

---

**✅ 从 `Data` 创建一个指针（不推荐长期使用）**

```swift
data.withUnsafeBytes { rawBuffer in
    let ptr = rawBuffer.baseAddress!
    let intValue = ptr.load(fromByteOffset: 0, as: Int.self)
}
```

你也可以把 Data 暂时当成内存指针使用，但这通常是临时用途，**不要越界访问**。

---

**✍️ 示例：完整流程对比**

```swift
let ptr = malloc(24)!
ptr.storeBytes(of: 100, as: Int.self)
ptr.storeBytes(of: 200, toByteOffset: 8, as: Int.self)
ptr.storeBytes(of: 300, toByteOffset: 16, as: Int.self)

let data = Data(bytes: ptr, count: 24)

let values = (0..<3).map { i in
    data.withUnsafeBytes { rawBuffer in
        rawBuffer.load(fromByteOffset: i * 8, as: Int.self)
    }
}
print(values) // [100, 200, 300]
```

---

**✅ 总结一句话：**

> `UnsafeMutableRawPointer` 是直接操作裸内存，`Data` 是封装后的字节缓冲。你写入指针的内容，完全可以通过 `Data` 来访问，二者可以互相转换，字节布局一致。


<br/><br/><br/>

***
<br/>

> <h1 id="类型Data">类型Data</h1>

- **主要功能：**
	- Data转化为无符号数字，包括`UInt16,UInt32,UInt64`，以及字节数组`[UInt8]`。
	- Data的初始化方法`init(number: Integer)`可传入一个任意类型的整数，此方法会将传入的整数转化为Data数据。
	- Data的初始化方法`init(hexString: String)`可将传入的16进制字符串转化成对应Data数据，当然也同样提供了将Data转化为16进制字符串的方法`data.hexString()`。
	- 另外还有一个将数字转化为字节数组的方法`bytes()`，该方法是对`BinaryInteger协议`的扩展，因此遵循BinaryInteger协议的数字类型`(如：Int,UInt,Int16,Int32,UInt32等)`都能直接使用此方法将数字转化为对应的[UInt8]。

***
<br/>

**`Data` 是按字节（Byte = 8 bits）来存储的,里面的每个元素都表示一个字节,并且元素可以是16进制、10进制数据**

***

**原例:**

```swift
var data = Data()
data.append(contentsOf: [0x01, 0x02, 0x03, 0x04])

// 或者
Data([0x01, 0x02, 0x03, 0x04])
```

表示在内存中顺序保存了这 4 个字节：

```
[00000001] [00000010] [00000011] [00000100]
```

也可以表示为：

```
十六进制：01 02 03 04
十进制：  1  2  3  4
```

***
<br/>

- **💡 进制只是“表示法”，不是“存储方式”**
	* 在计算机中，**所有数据最终都以二进制（0/1）存储**。
	* 所谓的 `0x01` 是写代码时用的 **十六进制语法**，只是“好看、方便”。
	* 存储到内存中，它就是 1 个字节：`00000001`。

**🔧 举个例子：**

```swift
// let d3 = Data([0b00000001, 0b00000010]) // 二进制
// let data = Data([65, 66, 67]) //10进制,等价如下
let data = Data([0x41, 0x42, 0x43])  // 16进制

if let str = String(data: data, encoding: .utf8) {
    print(str) // 输出 "ABC"
}
```

* `0x41 = 65` 是字母 `'A'` 的 ASCII 编码
* `0x42 = 66` 是 `'B'`
* `0x43 = 67` 是 `'C'`

***

**✅ 总结一句话：**

> `0x01` 是 16 进制的写法，表示的是一个字节（UInt8 类型），实际存储是二进制，`Data` 会按顺序把它们直接写入字节序列中。

你看到的进制只是方便开发者阅读。内存里只存在“位”的排列（0 和 1）。

***

**十六进制初始化 Data:**

```swift
let data = Data([0x01, 0xAB, 0xFF, 0x10])

// 下面三种写法在 `Data` 中效果一模一样
let d1 = Data([0x01, 0x02])            // 十六进制
let d2 = Data([1, 2])                  // 十进制
let d3 = Data([0b00000001, 0b00000010]) // 二进制
```

打印这些数据，结果都一样：

```swift
print(d1 as NSData) // <0102>
print(d2 as NSData) // <0102>
print(d3 as NSData) // <0102>
```

`0xXX` 写法是十六进制的**字面值**（literal），写法是十六进制，但本质上代表的是 `UInt8` 数值，比如：
* `0x01` = 1
* `0xAB` = 171
* `0xFF` = 255
* `0x10` = 16
* `<0102>` 是用十六进制打印出来的，说明字节值就是 1 和 2。

---

**✅ 总结重点**

| 方面    | 说明                              |
| ----- | ------------------------------- |
| 存储内容  | `Data` 存的是“字节”，每个字节是 8 位（UInt8） |
| 写法限制？ | ❌ 没有限制。你可以用 2、10、16 进制来表示这些字节   |
| 显示格式  | 打印时常用十六进制（如 `<FF00AB>`），便于查看    |
| 本质    | 所有内容都以 **二进制** 存储，只是写法/查看不同     |


***
<br/><br/><br/>
> <h2 id="不同字节数的数字拼接">不同字节数的数字拼接</h2>

**🔧 Swift 中将不同字节长度的数值拼接到 `Data`，比如：**

* 第 1 字节：存数字 64（占 1 字节）
* 第 2～3 字节：存一个 2 字节值
* 第 4～6 字节：存一个 3 字节值
* …

---

**✅ 首先：你需要手动控制“每个值的字节数”，也就是所谓的“小端 / 大端”问题。**

```swift
var packet = Data()

// 1️⃣ 写入 1 字节（UInt8）
let value1: UInt8 = 64
packet.append(value1)

// 2️⃣ 写入 2 字节（UInt16）——小端
let value2: UInt16 = 0x1234
packet.append(contentsOf: withUnsafeBytes(of: value2.littleEndian) { Array($0) })

// 3️⃣ 写入 3 字节（UInt32 的低 3 字节）
let value3: UInt32 = 0xABCDEF
let value3Bytes = withUnsafeBytes(of: value3.littleEndian) { Array($0.prefix(3)) }
packet.append(contentsOf: value3Bytes)

// 4️⃣ 写入 4 字节（UInt32）
let value4: UInt32 = 0x11223344
packet.append(contentsOf: withUnsafeBytes(of: value4.littleEndian) { Array($0) })

// 5️⃣ 写入 5 字节（UInt64 的低 5 字节）
let value5: UInt64 = 0x5566778899
let value5Bytes = withUnsafeBytes(of: value5.littleEndian) { Array($0.prefix(5)) }
packet.append(contentsOf: value5Bytes)

// 6️⃣ 写入 6 字节（UInt64 的低 6 字节）
let value6: UInt64 = 0xAABBCCDDEEFF
let value6Bytes = withUnsafeBytes(of: value6.littleEndian) { Array($0.prefix(6)) }
packet.append(contentsOf: value6Bytes)
```

---

**📦 解释每一步：**

| 步骤 | 类型       | 值                  | 字节数  | 说明                     |
| -- | -------- | ------------------ | ---- | ---------------------- |
| 1  | `UInt8`  | `0x40`             | 1 字节 | 简单 append              |
| 2  | `UInt16` | `0x1234`           | 2 字节 | 用 `withUnsafeBytes` 提取 |
| 3  | `UInt32` | `0x00ABCDEF`       | 3 字节 | 取前 3 字节 `.prefix(3)`   |
| 4  | `UInt32` | `0x11223344`       | 4 字节 | 正常追加                   |
| 5  | `UInt64` | `0x005566778899`   | 5 字节 | 取前 5 字节                |
| 6  | `UInt64` | `0x00AABBCCDDEEFF` | 6 字节 | 取前 6 字节                |

🔁 **小端（little-endian）** 表示低位字节在前，高位字节在后，是 BLE 通讯中常用字节序。

---

**🧪 打印结果（十六进制）**

你可以打印看效果：

```swift
print(packet.map { String(format: "%02X", $0) }.joined(separator: " "))
```

可能输出类似：

```
40 34 12 EF CD AB 44 33 22 11 99 88 77 66 55 FF EE DD CC BB AA
```

---

**✅ 总结**

| 问题                | 答案                                      |
| ----------------- | --------------------------------------- |
| 如何存 1～6 字节不同长度数据？ | 用 `withUnsafeBytes + prefix(n)` 构造你要的字节 |
| 用什么类型拼接？          | `Data` 是最方便的。你可以随意 append 任意长度数据片段      |
| 字节序要注意什么？         | 通常使用小端（little endian），特别是 BLE 这类通信协议里   |

<br/><br/>
> <h3 id="多个字节拼接">多个字节拼接</h3>

**比如实现如下字节拼接:**

```swift
newPacketData = [signatureLength] + signatureData + [randNumLength] + randNum
```

```swift
let signatureLength: UInt8 = 32
let randNumLength: UInt8 = 84  // 注意：类型应该是 UInt8，不是 Uint8

let signatureData = Data(repeating: 0xAB, count: Int(signatureLength)) // 示例数据
let randNum = Data(repeating: 0xCD, count: Int(randNumLength))         // 示例数据

// 构建数据包
let newPacketData = Data([signatureLength]) + signatureData + Data([randNumLength]) + randNum
```

---

**🔍 分析说明：**

| 部分                        | 内容           | 类型     |
| ------------------------- | ------------ | ------ |
| `Data([signatureLength])` | 将长度作为第一个字节写入 | `Data` |
| `signatureData`           | 签名内容（32字节）   | `Data` |
| `Data([randNumLength])`   | 再写入随机数的长度    | `Data` |
| `randNum`                 | 随机数内容（84字节）  | `Data` |

---

**✅ 最终结果：**

你得到的 `newPacketData` 会是这样的结构（总共 `1 + 32 + 1 + 84 = 118` 字节）：

```swift
[1字节 signatureLength] + [32字节 signatureData] + [1字节 randNumLength] + [84字节 randNum]


// 检查拼接结果
print("newPacketData count: \(newPacketData.count)")  // 应该是 118
print("newPacketData (hex): \(newPacketData.map { String(format: "%02x", $0) }.joined())")
```



👉 **如何使用一个 `UInt8` 数值创建一个只包含一个字节的 `Data` ?**

**✅ 举例说明：**

```swift
let randNumLength: UInt8 = 84
let lengthData = Data([randNumLength])
```

这段代码的含义是：

* `randNumLength` 是一个 `UInt8` 类型的值，值为 `84`
* `[randNumLength]` 是一个数组，包含一个元素 `[84]`
* `Data([randNumLength])` 把这个数组变成一个 **Data 对象**，也就是一个长度为 1 的字节序列

---

**🧾 内存表示（十六进制）：**

```swift
print(lengthData as NSData)  // <54>
```

其中 `<54>` 是十六进制的 84（即 0x54），表示这个 `Data` 对象中只有一个字节，值为 84。

---

**✅ 总结：**

| 表达式                     | 含义                 | 类型           |
| ----------------------- | ------------------ | ------------ |
| `[randNumLength]`       | 创建一个 `UInt8` 数组    | `[UInt8]`    |
| `Data([randNumLength])` | 用这个数组生成 1 字节的二进制数据 | `Data`，长度为 1 |

***
<br/><br/><br/>
> <h2 id="自描述式二进制数据包格式拆包解析">自描述式二进制数据包格式拆包解析</h2>

下面是一个非常常见的**自描述式二进制数据包格式**，结构如下所示（我们以 ASCII 图示来表达）：

```
| aLen: UInt8 | aData: [UInt8] | bLen: UInt8 | bData: [UInt8] | ...
```

也就是说：

* 每段数据前面 **先跟一个长度字段**（`1 字节 UInt8`），
* 然后跟对应长度的数据内容。
* 重复这个结构。

---

 **✅ 从 `Data` 类型的 `packetData` 中按顺序解析出 `aData`, `bData`, `cData` 代码:**

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

**✅ 模拟数据测试：**

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

---

 **✅ 提示**

如果你知道你最多有几个字段（例如 a, b, c），你也可以改为返回命名字段：

```swift
struct Packet {
    let aData: Data
    let bData: Data
    let cData: Data
}
```

> 这个格式常见于 BLE、串口、嵌入式通信、或自定义数据协议中，解包时务必注意长度校验。




<br/><br/>
> <h3 id="`Data(repeating:count:)`用法详解">`Data(repeating:count:)`用法详解</h3>


```swift
Data(repeating: 0xAB, count: Int(signatureLength))
```

这行代码是在创建一个 **固定内容、固定长度的 Data（二进制数据）对象**，我们逐个拆开讲：

---

**🧱 拆解说明**

| 部分                            | 含义                                                    |
| ----------------------------- | ----------------------------------------------------- |
| `0xAB`                        | 一个十六进制的字节值（即十进制的 171），用于填充                            |
| `repeating: 0xAB`             | 表示要重复这个值来填充 Data                                      |
| `count: Int(signatureLength)` | 指定重复多少次。这里 `signatureLength` 是 `UInt8`，需要转换成 `Int` 类型 |
| `Data(...)`                   | 创建一个 Swift 的 `Data` 对象，用这些字节填充                        |

---

**✅ 举例**

假设：

```swift
let signatureLength: UInt8 = 4
```

那么执行：

```swift
let signatureData = Data(repeating: 0xAB, count: Int(signatureLength))
```

就等价于创建了这样的二进制数据：

```
<ab ab ab ab>
```

也就是 4 个字节，每个字节都是十六进制的 AB（十进制 171）。




<br/><br/>
> <h3 id="传入的值为十进制数转成不同字节数值">传入的值为十进制数转成不同字节数值</h3>

要求存储的是**十进制的数值本身**，不是十六进制写法，不想写 `0x1234`、`0xABCDEF`，我写的就是十进制的值，比如 `1234`、`11259375`、`2864434397` 等，然后要以占 **1 \~ 6 个字节的形式**存储进 `Data` 中。

---

写的是 **十进制数值**，但数据最终还是要按“二进制字节序列”存储（因为 `Data` 只能存 byte）。

所以你可以 **完全不写十六进制**，这样写就行：

```swift
var packet = Data()

// 👇全是十进制数值，但会转换成对应字节数的小端数据写入 Data

let value1: UInt8 = 64                  // 1 字节
packet.append(value1)

let value2: UInt16 = 1234              // 2 字节
packet.append(contentsOf: withUnsafeBytes(of: value2.littleEndian) { Array($0) })

let value3: UInt32 = 11259375          // 3 字节（其实是 0x00ABCDEF）
packet.append(contentsOf: withUnsafeBytes(of: value3.littleEndian) { Array($0.prefix(3)) })

let value4: UInt32 = 2864434397        // 4 字节（就是 0xAABBCCDD）
packet.append(contentsOf: withUnsafeBytes(of: value4.littleEndian) { Array($0) })

let value5: UInt64 = 366503875925      // 5 字节
packet.append(contentsOf: withUnsafeBytes(of: value5.littleEndian) { Array($0.prefix(5)) })

let value6: UInt64 = 187723572702975   // 6 字节
packet.append(contentsOf: withUnsafeBytes(of: value6.littleEndian) { Array($0.prefix(6)) })
```

---

- **🔍 说明**
	* 你传入的是**十进制数值**，如 `1234`，只是写法，不影响存储。
	* 存入 `Data` 时，Swift 会根据类型（`UInt16`, `UInt32`...）转成对应的**字节数组(注意:一定要传入类型,否则默认是Int类型了)**，用的是小端格式（BLE 通常这样要求）。
	* `prefix(n)` 控制只取你需要的字节数，比如只用 `UInt64` 的低 5 字节。

---

**🔧 工具函数（推荐你封装一下）**

为了写得更简洁，你可以这样封装一个通用函数：

```swift
func appendInteger<T: FixedWidthInteger>(
    _ value: T,
    byteCount: Int,
    to data: inout Data
) {
    let bytes = withUnsafeBytes(of: value.littleEndian) { Array($0.prefix(byteCount)) }
    data.append(contentsOf: bytes)
}
```

然后用法非常优雅：

```swift
var packet = Data()

appendInteger(64 as UInt8, byteCount: 1, to: &packet)
appendInteger(1234 as UInt16, byteCount: 2, to: &packet)
appendInteger(11259375 as UInt32, byteCount: 3, to: &packet)
appendInteger(2864434397 as UInt32, byteCount: 4, to: &packet)
appendInteger(366503875925 as UInt64, byteCount: 5, to: &packet)
appendInteger(187723572702975 as UInt64, byteCount: 6, to: &packet)
```

---

**✅ 总结**

| 你输入的十进制值 | ✅ 支持，完全没问题                      |
| -------- | ------------------------------- |
| 怎么控制字节数？ | 用 `withUnsafeBytes + prefix(n)` |
| 最适合的容器？  | `Data` 是最合适的二进制容器               |
| 最终存储方式？  | 都转成二进制字节序列（小端）                  |

---

**🔹解析如下代码 ：**

```swift
let value2: UInt16 = 1234              // 2 字节

acket.append(contentsOf: withUnsafeBytes(of: value2.littleEndian) { Array($0) })
```

把一个整数值（十进制 `1234`，类型是 `UInt16`）转换成 **2 字节的二进制表示**（小端），并添加到 `packet` 中。

<br/>

**✅ 步骤拆解：**

```swift
value2.littleEndian
```

把 `value2` 按 **小端字节序**表示（例如，1234 十进制是 `0x04D2`，小端顺序是 `[0xD2, 0x04]`）。

```swift
withUnsafeBytes(of: value2.littleEndian)
```

把 `UInt16` 看作一个 2 字节内存块，返回一个 `UnsafeRawBufferPointer` —— 本质上是一个字节数组视图。

```swift
{ Array($0) }
```

把字节视图转成 `Array<UInt8>`，比如 `[0xD2, 0x04]`。
<br/>

```swift
packet.append(contentsOf: ...) 
```

把这两个字节 **追加到最终的 `Data` 中**。

**🔍 例子：**

```swift
let value2: UInt16 = 1234      // 十进制
value2.littleEndian            // -> 0x04D2，小端是 [0xD2, 0x04]
```

---
<br/>

**🔹代码 2：**

```swift
packet.append(contentsOf: withUnsafeBytes(of: value3.littleEndian) { Array($0.prefix(3)) })
```

**🚀 目的：**

把一个 `UInt32` 整数（十进制 `11259375`）的低 3 个字节写入 `Data`（也就是只取低 24 位）。

 **✅ 步骤拆解：**

```swift
value3.littleEndian
```

例如：11259375 → 十六进制是 `0x00ABCDEF` → 小端字节 `[0xEF, 0xCD, 0xAB, 0x00]`

```swift
withUnsafeBytes(of: value3.littleEndian)
```

把这个 4 字节整数的原始字节表示拿出来。

```swift
$0.prefix(3)
```

只保留前 3 个字节 → `[0xEF, 0xCD, 0xAB]`

```swift
Array(...) + append(contentsOf:)
```

转换成 `Array<UInt8>` 并追加到 `Data` 中。

**🔍 例子：**

```swift
let value3: UInt32 = 11259375       // 十进制
value3.littleEndian = 0xEFCDAB00    // 内存中按 [EF, CD, AB, 00]
prefix(3) => [EF, CD, AB]
```

---

### 🧠 总结区别：

| 项目      | 第一行 (`value2`) | 第二行 (`value3`)                 |
| ------- | -------------- | ------------------------------ |
| 数据类型    | `UInt16`（2 字节） | `UInt32`（4 字节）                 |
| 添加多少字节？ | 添加全部 2 字节      | 只添加前 3 字节（低 3 字节）              |
| 是否裁剪字节？ | ❌ 不裁剪          | ✅ 用 `.prefix(3)` 裁剪            |
| 示例结果    | `[0xD2, 0x04]` | `[0xEF, 0xCD, 0xAB]`（丢弃第 4 字节） |

<br/><br/>
> <h3 id="将十进制64存储为4字节">将十进制64存储为4字节</h3>

**✅ 示例代码（将十进制 64 存储为 4 字节）：**

```swift
let value4: UInt32 = 64
packet.append(contentsOf: withUnsafeBytes(of: value4.littleEndian) { Array($0) })
```

---

- **🧠 解释：**
	* `value4` 是 `UInt32` 类型，即使你存的是小数如 `64`，它也会用 **4 个字节表示**。
	* `64` 十六进制是 `0x00000040`
	* 小端模式下，内存中字节顺序是：

  ```
  [0x40, 0x00, 0x00, 0x00]
  ```

---

**✅ 验证输出**

如果你打印 `packet` 内容：

```swift
print(packet.map { String(format: "%02X", $0) }.joined(separator: " "))
```

会显示：

```
40 00 00 00
```

---

**🧩 总结**

| 内容         | 说明                         |
| ---------- | -------------------------- |
| 十进制值       | `64`                       |
| 类型         | `UInt32`                   |
| 字节序        | 小端（低位在前）                   |
| 存储结果（4 字节） | `[0x40, 0x00, 0x00, 0x00]` |



***

**如下代码:**



```swift
packet.append(contentsOf: withUnsafeBytes(of: 64.littleEndian) { Array($0) })

```

在 Swift 中编译器会推断 **64 是 Int 类型（通常是 Int64 on 64-bit 系统, 也就是默认Int类型是Int64）**。这就引发了一个问题：你可能不确定最终用了多少个字节来存储它。

<br/>

```
// 显式指定类型
packet.append(contentsOf: withUnsafeBytes(of: UInt32(64).littleEndian) { Array($0) })
```

- **这样：**
	- 明确是 UInt32（占 4 字节）
	- .littleEndian 保证了字节顺序是 BLE/网络中常用的小端格式
	- withUnsafeBytes 获取的是 UInt32 的 4 字节内存视图



***
<br/><br/><br/>
> <h2 id="SHA256字节签名"> SHA256字节签名</h2>

使用 `HMAC-SHA256` 算法和指定的共享密钥 `sharedKeyStr` 对字符串进行签名。

---

**✅ 示例：Swift 中的 `String` 扩展（HMAC-SHA256 签名）**

```swift
import Foundation
import CommonCrypto

extension String {
    /// 使用 HMAC-SHA256 加密字符串，返回十六进制字符串签名
    func hmacSHA256(key sharedKeyStr: String) -> String? {
        guard let keyData = sharedKeyStr.data(using: .utf8),
              let messageData = self.data(using: .utf8) else {
            return nil
        }

        var digest = Data(count: Int(CC_SHA256_DIGEST_LENGTH))

        digest.withUnsafeMutableBytes { digestBytes in
            keyData.withUnsafeBytes { keyBytes in
                messageData.withUnsafeBytes { messageBytes in
                    CCHmac(CCHmacAlgorithm(kCCHmacAlgSHA256),
                           keyBytes.baseAddress, keyData.count,
                           messageBytes.baseAddress, messageData.count,
                           digestBytes.baseAddress)
                }
            }
        }

        // 返回 hex 格式的字符串
        return digest.map { String(format: "%02hhx", $0) }.joined()
    }
}
```

---

**✅ 用法示例：**

```swift
let message = "This is a message"
let sharedKey = "my_secret_key"

if let signature = message.hmacSHA256(key: sharedKey) {
    print("签名结果：\(signature)")
}
```

---

**✅ 输出格式说明**

这个方法返回的是 **十六进制字符串格式的签名**，可根据需要修改为 `Base64` 编码：

```swift
return digest.base64EncodedString()
```

***
<br/><br/><br/>
> <h2 id="字符串扩充到指定字节数,不够补0">字符串扩充到指定字节数,不够补0</h2>

- 任意字符串字节长度（比如 16、18、20、32 等）不足对其进行：
	* 选择补零（`0x00`）或截断；
	* 输出十六进制字符串查看。

---

**✅ 封装方法：将字符串转换为固定长度的 Data（自动补 0）**

```swift
extension String {
    /// 将字符串转为固定长度的 Data，不足补 0x00，超出则截断
    /// - Parameter length: 目标字节数
    /// - Returns: 固定长度的 Data
    func toFixedLengthData(_ length: Int) -> Data {
        var data = self.data(using: .utf8) ?? Data()
        if data.count < length {
            data.append(contentsOf: Array(repeating: 0x00, count: length - data.count))
        } else if data.count > length {
            data = data.prefix(length)
        }
        return data
    }
}

extension Data {
    /// 十六进制字符串打印（每字节空一格）
    func hexString(spaced: Bool = true, uppercased: Bool = true) -> String {
        let format = uppercased ? "%02X" : "%02x"
        return self.map { String(format: format, $0) }.joined(separator: spaced ? " " : "")
    }
}
```

---

**🧪 使用示例：**

```swift
let str = "123456"

// 转成 16 字节 Data
let data16 = str.toFixedLengthData(16)
print("16字节: \(data16.hexString())")

// 转成 18 字节 Data
let data18 = str.toFixedLengthData(18)
print("18字节: \(data18.hexString())")

// 转成 32 字节 Data
let data32 = str.toFixedLengthData(32)
print("32字节: \(data32.hexString())")
```

---

**🔎 示例输出：**

```
16字节: 31 32 33 34 35 36 00 00 00 00 00 00 00 00 00 00
18字节: 31 32 33 34 35 36 00 00 00 00 00 00 00 00 00 00 00 00
32字节: 31 32 33 34 35 36 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```


***
<br/><br/><br/>
> <h2 id="byte以16进制打印">byte以16进制打印</h2>

```swift
extension Data {
    func hexString(spaced: Bool = true, uppercased: Bool = true) -> String {
        let format = uppercased ? "%02X" : "%02x"
        return self.map { String(format: format, $0) }.joined(separator: spaced ? " " : "")
    }
}
```

- **📌 说明：**
	* `map { String(format: "%02X", $0) }`: 把每个字节（`UInt8`）格式化为两位大写十六进制字符串；
	* `joined(separator: " ")`: 用空格连接每个字符串；
	* `%02X`: 表示十六进制，大写，不足两位补零, 用 `%02x` 输出小写结果。
***

**用法：**

```swift
let data: Data = Data([0x01, 0xAB, 0x34, 0xFF])

print(data.hexString()) // 默认大写并加空格
```

🧾 输出结果：

```
01 AB 34 FF
```



<br/><br/><br/>

***
<br/>

> <h1 id="类型UInt8">类型UInt8</h1>

 **✅ 1. `UInt8` 的存储本质**

`UInt8` 表示的是 **0\~255 的无符号整数**，也就是 8 位二进制可以表示的范围。
存储在内存里时，是 **二进制** 存储的，不是十六进制也不是八进制。计算机的内存只懂二进制。

---

**✅ 2. `[0x01, 0x02, 0x03, 0x04]` 是什么？**

**原代码:**

```swift
var data = Data()
data.append(contentsOf: [0x01, 0x02, 0x03, 0x04])
```

这是用 **十六进制写法** 表示的 `UInt8` 数组：

```swift
[0x01, 0x02, 0x03, 0x04]
```

* `0x01`、`0x02` 等是**16 进制（Hex）字面量**。
* 它们代表的是 **一个个字节（8 位，UInt8）** 的数值。
* 所以这 4 个数各自是：

| 表达式  | 十进制值 | 二进制值     |
| ---- | ---- | -------- |
| 0x01 | 1    | 00000001 |
| 0x02 | 2    | 00000010 |
| 0x03 | 3    | 00000011 |
| 0x04 | 4    | 00000100 |

<br/>

**等价于：**

```swift
[1, 2, 3, 4] // 十进制写法
```

每个元素的数值是一样的，只是你写代码时使用了不同进制的“表示法”而已。

---

**✅ 3. 总结：你问的“是多少进制的数据”含义如下**

| 内容          | 答案说明                        |
| ----------- | --------------------------- |
| **写法上的进制**  | 你写的是 **十六进制**（因为用了 `0x` 前缀） |
| **数据的本质存储** | 是 **二进制**，计算机永远以二进制存储数据     |
| **数据类型**    | 是 `UInt8`，值范围是 0\~255       |

---

**📌 举个例子：**

```swift
let byte: UInt8 = 0xFF
print(byte) // 输出 255
```

虽然你用 `0xFF` 写出来，但它就是一个 255 的 `UInt8` 值，内存里就是二进制 `11111111`。


***
<br/><br/><br/>
> <h2 id="数组类型截取二进制data数据">数组类型截取二进制data数据</h2>

通过 **「数组下标+范围」** 的方式提取 `Data` 字节字段，而不是位移（`<<`、`|`）操作。适合你后续维护、调试和查错。

---

 ✅ 假设数据为**一个 16 字节的 Data 包（小端格式）** 结构（示例）：

| 字节范围   | 字段名       | 字节数 | 说明        |
| ------ | --------- | --- | --------- |
| 0..1   | version   | 2   | UInt16，小端 |
| 2      | type      | 1   | UInt8     |
| 3..6   | timestamp | 4   | UInt32，小端 |
| 7..10  | deviceID  | 4   | UInt32，小端 |
| 11..15 | reserved  | 5   | 保留/其他字段   |

使用 `Data.subdata(in:)` 提取子字节段，然后用 `withUnsafeBytes {}` 加载为整数值。**不需要移位操作。**

```swift
import Foundation

enum Endian {
    case little
    case big
}

struct MyParsedPacket {
    let version: UInt16
    let type: UInt8
    let timestamp: UInt32
    let deviceID: UInt32
    let reserved: Data

    init?(data: Data) {
        guard data.count >= 6 else {
            print("数据长度不足")
            return nil
        }

        // version: bytes 0..2, 使用2个字节的UInt16表示
        let versionData = data.subdata(in: 0..<2)
        self.version = versionData.withUnsafeBytes { $0.load(as: UInt16.self) }

        // type: byte 2
        self.type = data[2]

        // timestamp: bytes 3..7 使用4个字节的UInt32表示
        let timestampData = data.subdata(in: 3..<7)
        self.timestamp = timestampData.withUnsafeBytes { $0.load(as: UInt32.self) }

        // deviceID: bytes 7..11
        let deviceIDData = data.subdata(in: 7..<11)
        self.deviceID = deviceIDData.withUnsafeBytes { $0.load(as: UInt32.self) }
	  }

    func printFields() {
        print("Version: \(version)")
        print("Type: \(type)")
        print("Timestamp: \(timestamp)")
        print("Device ID: \(deviceID)")
    }
}
```

---

**✅ 调用**：

```swift
let packetData = Data([
    0x01, 0x00,       // version = 1
    0x02,             // type = 2
    0x78, 0x56, 0x34, 0x12, // timestamp = 0x12345678
    0xDE, 0xAD, 0xBE, 0xEF, // deviceID = 0xEFBEADDE
    0x11, 0x22, 0x33, 0x44, 0x55 // reserved
])

if let parsed = MyParsedPacket(data: packetData) {
    parsed.printFields()
}
```

---

**输出结果（十进制为主）：**

```
Version: 1
Type: 2
Timestamp: 305419896
Device ID: 4022250974
Reserved (Hex): 11 22 33 44 55
```

- **✅ 说明总结：**	
	* 使用 `.subdata(in:)` 提取范围，避免了位运算逻辑。
	* 用 `.withUnsafeBytes { $0.load(as:) }` 转换为整数（自动按小端）。
	* 结构清晰，字段位置一目了然。
	* Reserved 这种保留字段保留为原始 Data，你可以根据需要转为字符串、Hex、Base64 等。

***

&emsp; 若是想从一个 `Data` 对象中 **截取第 45 到 100 字节之间的数据**，在 Swift 中你可以直接使用 **`subdata(in:)`** 或 **`data[range]`** 来操作

如直接用**数组切片（Swift 5+）**

```swift
let subData = data[45..<101]
```


***
<br/><br/><br/>
> <h2 id="private(set)访问控制+自定义方法">private(set)访问控制+自定义方法 </h2>

在 Swift 中，如果你想实现一个属性：

* ✅ **可以通过点语法读取（get）**；
* ❌ **不能通过点语法写入（set）**；
* ✅ **只能通过类内特定方法赋值**；

你可以通过 **`private(set)` 访问控制 + 自定义方法** 来实现。

---

## ✅ 示例：属性只能通过方法赋值

```swift
class MyClass {
    // 外部和本类都可以读取，但只有本类可以写入
    private(set) var status: String = "initial"
    
    // 本类中提供专门的赋值方法
    func updateStatus(to newValue: String) {
        status = newValue
    }
}
```

---

## ✅ 使用方式：

```swift
let obj = MyClass()

// ✅ 外部可以读取
print(obj.status)

// ❌ 外部不能直接赋值，会报错
// obj.status = "new value"  // ❌ Error

// ✅ 通过方法修改
obj.updateStatus(to: "updated")
print(obj.status)
```

---

## ✅ 限制更严格：只能本类内写入，连子类也不能赋值？

将属性声明为 `private var`（而不是 `private(set)`），就只有当前类内可以访问和修改：

```swift
class MyClass {
    private var status: String = "initial"

    var currentStatus: String {
        return status // 公开只读接口
    }

    func updateStatus(to newValue: String) {
        status = newValue
    }
}
```

---

需要我补充这个设计模式在更复杂对象或枚举状态下的用法吗？例如状态机、只读配置、网络连接状态等。


***
<br/><br/><br/>
> <h2 id="类型UInt64变量转化为UInt8如何做?">类型UInt64变量转化为UInt8如何做?</h2>

```swift
enum RegisterCode: UInt64 {
    case start         = 0x10
    case success       = 0x11
    case failed        = 0x12
    case verifySuccess = 0x13
    case verifyFail    = 0x14
}


let result: RegisterCode = .success
print(result.rawValue) // 输出 0x11
```

***

Uint64转换成UInt8字节的,会报错,如下:

```swift
// 将result转换为一个字节,如下:
let status = RegisterCode.start 
let value = status.rawValue  // 是 UInt64 类型
let msgData = Data([RegisterCode.success.rawValue])   // 期望的是 [UInt8]
// 报错: Cannot convert value of type 'UInt64' to expected element type 'UInt8
```


**解决方法：显式类型转换**
你需要将 UInt64 转换为 UInt8，前提是值不能超过 UInt8 的范围（0~255）：

```swift
let byte: UInt8 = UInt8(status.rawValue & 0xFF) // 只取低 8 位
let msgData = Data([byte])
```

或者更直接写成：

```swift
let msgData = Data([UInt8(status.rawValue & 0xFF)])
```

<br/>

但是要注意的是,上述转化不太准确因为你要确定`status.rawValue` 的值在 `0...255` 之间，这样做是安全的。

如果你希望将整个 UInt64（8 字节）编码进 Data，你应该用：

```swift
var value = status.rawValue
let msgData = withUnsafeBytes(of: &value) { Data($0) }
```
这会生成一个包含完整 8 字节数据的 Data。

<br/><br/>
> <h3 id="将4字节写入Data">将4字节写入Data</h3>

如果你要将一个 UInt32 写入 Data：

```swift
var num: UInt32 = 0x12345678
let data = withUnsafeBytes(of: &num) { Data($0) }
// data.count == 4
```
⚠️ 注意这个是 小端字节序，如果需要固定字节序（比如大端），可以用：

```swift
let bigEndianValue = num.bigEndian
let data = withUnsafeBytes(of: bigEndianValue) { Data($0) }
```

<br/><br/>
> <h3 id="`withUnsafeBytes(of:)`方法详解">`withUnsafeBytes(of:)` 方法详解</h3>

- **这个方法做什么的？**

**`withUnsafeBytes(of:)` 是 Swift 提供的一种把任意值的内存表示转换为 `UnsafeRawBufferPointer` 的方法。**

- **通常用于：**
	* 将整数、结构体等 **转成 `Data`**；
	* 用于进行 **低层数据访问（比如蓝牙/网络包）**；
	* 避免自己手动 byte-by-byte 拼数据。

---

- **✅ 基本语法：**

```swift
var value: SomeType = ...
let data = withUnsafeBytes(of: &value) { Data($0) }

// 完整写法
let data = withUnsafeBytes(of: &type) { rawBuffer in
    return Data(rawBuffer)
}
```
-  “把变量 `value` 占用的内存区域原封不动复制为 `Data` 类型，适合用于序列化或底层通信”。
	* `&value`：传的是变量地址（必须是 `var`）。
	* `$0`：是 `UnsafeRawBufferPointer`，可以像 byte 数组一样访问。
	* `Data($0)`：初始化 `Data`，把这些原始字节收集成 Swift 的 `Data` 对象。

---

**✅ 举个例子：将一个 `UInt32` 写入 `Data`**

```swift
var number: UInt32 = 0x12345678
let data = withUnsafeBytes(of: &number) { Data($0) }

print(data as NSData) // 输出: <78563412> （小端序）
```

👉 输出的 `Data` 为：

```
0x78 0x56 0x34 0x12
```

这说明 Swift 默认用 **小端字节序（little-endian）** 存储整数。

---

**✅ 如果想用大端字节序：**

```swift
var number = UInt32(0x12345678).bigEndian
let data = withUnsafeBytes(of: &number) { Data($0) }

print(data as NSData) // 输出: <12345678>
```

---

**✅ 常见用途**

多个字段拼接成 `Data`

```swift
var type: UInt8 = 0x01
var length: UInt16 = 0x0020
var checksum: UInt8 = 0xFF

var data = Data()
data.append(contentsOf: withUnsafeBytes(of: &type) { Data($0) })
data.append(contentsOf: withUnsafeBytes(of: &length) { Data($0) })
data.append(contentsOf: withUnsafeBytes(of: &checksum) { Data($0) })
```

> 最终 `data` 就是一个按字段顺序拼好的二进制流。

---

**⚠️ 注意事项**

| 注意点           | 说明                                         |
| ------------- | ------------------------------------------ |
| 必须是 `var`     | 因为需要拿变量的内存地址                               |
| 不适用于 class 类型 | 只适合值类型，比如 `struct`、整数等                     |
| 字节序           | 默认是小端，小心跨平台或通信协议需要                         |
| 安全性           | `withUnsafeBytes` 是“安全地使用不安全内存”，但不等于你能越界访问 |



***
<BR/><BR/><BR/>
> <H2 ID="结构体进行拼包">结构体进行拼包</H2>

在 Swift 中，如果你要将一个 C 风格的结构体（如你给的 `lock_event_s`）编码为字节（`[UInt8]` 或 `Data`），我们通常会使用 **`withUnsafeBytes`** 或 **`Data(buffer:)`** 等方法，将结构体内存内容转为二进制。

---
<BR/>

**✅ 目标结构体（Swift 等价定义）**

**C 结构体如下：**

```c
struct lock_event_s {
    uint8_t  action;
    uint8_t  method;
    uint32_t user_id;
    uint32_t time;
} __PACKED;
```

<BR/>

在 Swift 中，可以这样定义（确保使用 Packed 对齐方式）：

```swift
import Foundation

struct LockEvent: Codable {
    var action: UInt8
    var method: UInt8
    var user_id: UInt32
    var time: UInt32
}
```

> ⚠️ 但 Swift 结构体默认不是 packed，需要手动处理大小端和拼接顺序，来模拟“紧凑结构体”。

---
<br/>

- **✅ 方法一：手动将字段转为字节数组再拼接（最安全方式）**

```swift
extension LockEvent {
    /// 转换为字节数组（[UInt8]），手动控制大小端
    func toBytes() -> [UInt8] {
        var result = [UInt8]()
        result.append(action)
        result.append(method)

        result.append(contentsOf: user_id.littleEndianBytes)
        result.append(contentsOf: time.littleEndianBytes)

        return result
    }
}
```

<br/>

- **✅ UInt32 转为字节数组扩展**

```swift
extension UInt32 {
    var littleEndianBytes: [UInt8] {
        let le = self.littleEndian
        return [
            UInt8((le >> 0) & 0xff),
            UInt8((le >> 8) & 0xff),
            UInt8((le >> 16) & 0xff),
            UInt8((le >> 24) & 0xff)
        ]
    }
}
```

<br/>

**调用：**

```swift
let event = LockEvent(action: 1, method: 2, user_id: 12345678, time: 98765432)
let bytes = event.toBytes()

print(bytes)  // [1, 2, x, x, x, x, y, y, y, y]
let data = Data(bytes)  // 如需要转 Data
```

<br/>

- **✅ 输出验证**得到：

	* 第 1 字节：`action`
	* 第 2 字节：`method`
	* 第 3\~6 字节：`user_id`（小端）
	* 第 7\~10 字节：`time`（小端）

和 C 中按 packed struct 的存储顺序一致（如果目标平台是小端）。

<br/>

**🧠 可选方法：Unsafe 指针法（快速但受对齐限制）**

```swift
var event = LockEvent(action: 1, method: 2, user_id: 12345678, time: 98765432)
let data = withUnsafeBytes(of: &event) { Data($0) }
```

⚠️ 这种方法效率高，但你需要保证结构体内没有 padding，而且要考虑大小端问题，否则结果可能和 C 不一致。

---
<br/>

**✅ 建议**

| 要求                 | 方式                           |
| ------------------ | ---------------------------- |
| 安全、明确字节序           | ✅ 手动拼接（推荐）                   |
| 追求效率（结构体没 padding） | ✅ `withUnsafeBytes`          |
| 从外部设备/蓝牙获取数据还原结构体  | 我也可以给你 `init(bytes:)` 的反解析方法 |




***
<br/><br/><br/>
> <h2 id="UInt32转成Data类型">UInt32转成Data类型</h2>
- **将 `UInt32` 转换为 `Data` 类型，最常见的方式是：**

	* 将它转成字节数组（按照大端或小端字节序）
	* 然后用 `Data` 初始化即可。

<br/>

- **✅ 方法一：使用 `withUnsafeBytes(of:)`（标准做法）**

```swift
let number: UInt32 = 12345678
let data = withUnsafeBytes(of: number.littleEndian) { Data($0) }
```

<br/>

- **🔹说明：**
	* `littleEndian`：确保按小端存储（BLE 通常用小端）
	* `Data($0)`：把内存内容转换成 Data

<br/>

- **✅ 方法二：手动转成字节数组再转成 Data（更可控）**

```swift
let number: UInt32 = 12345678
let bytes: [UInt8] = [
    UInt8((number >> 0) & 0xFF),
    UInt8((number >> 8) & 0xFF),
    UInt8((number >> 16) & 0xFF),
    UInt8((number >> 24) & 0xFF)
]
let data = Data(bytes)
```

如果你想转为 **大端序**，就把位移顺序改成反的。

<br/>

- **✅ 方法三：扩展 UInt32（推荐）**

```swift
extension UInt32 {
    func toData(littleEndian: Bool = true) -> Data {
        let value = littleEndian ? self.littleEndian : self.bigEndian
        return withUnsafeBytes(of: value) { Data($0) }
    }
}
```

**用法：**

```swift
let value: UInt32 = 987654321
let data = value.toData()  // 默认小端
```

<br/>

**✅ 输出示例**

```swift
let value: UInt32 = 0x12345678
let data = value.toData()
print(data as NSData)  // <78563412> （小端）
```



***
<br/><br/><br/>
> <h2 id="移位详解">移位详解</h2>

```swift
let bytes: [UInt8] = [
    UInt8((number >> 0) & 0xFF),
    UInt8((number >> 8) & 0xFF),
    UInt8((number >> 16) & 0xFF),
    UInt8((number >> 24) & 0xFF)
]
```

**提问‼️：** 这段代码什么意思？

<br/>

**🎯 将 `UInt32` 拆解成 4 个字节（`UInt8`）**

一个 `UInt32` 是 4 个字节（32 位）组成的整数：

```
| byte3 | byte2 | byte1 | byte0 |
|  24~31位 | 16~23位 | 8~15位 | 0~7位 |
```

<br/>
例如：

```swift
let number: UInt32 = 0x12345678
```

<br/>
二进制是：

```
00010010 00110100 01010110 01111000
   ↑         ↑       ↑       ↑
 byte3    byte2   byte1   byte0
```

---
<br/>

- **✅ 分步解释：**

	- **`number >> 0`：**
		* 不偏移，保留最低 8 位（即 byte0）
		* 然后 `& 0xFF`（只保留最低的 8 位）
		* 结果是 byte0（`0x78`）

<br/>

- **`number >> 8`：**
	* 向右偏移 8 位 → 原本 byte1 现在变成最低位
	* 再 `& 0xFF` 取最低 8 位 → 得到 byte1（`0x56`）

**依此类推：**

| 表达式                     | 得到    | 说明  |
| ----------------------- | ----- | --- |
| `(number >> 0) & 0xFF`  | byte0 | 最低位 |
| `(number >> 8) & 0xFF`  | byte1 | 次低位 |
| `(number >> 16) & 0xFF` | byte2 | 次高位 |
| `(number >> 24) & 0xFF` | byte3 | 最高位 |

<br/>

**✅ 最终得到的字节顺序**

```swift
[byte0, byte1, byte2, byte3]
```

也就是 **小端字节序（Little Endian）** —— 低位在前，高位在后。
这在蓝牙、网络协议、设备通信中很常见。

---
<br/>

🧠 举个例子：

```swift
let number: UInt32 = 0x12345678
```

拆出来是：

```swift
[0x78, 0x56, 0x34, 0x12]  // 小端序
```



***
<br/><br/><br/>
> <h2 id="UInt32类型的10进制时间戳需要移位吗？">UInt32类型的10进制时间戳需要移位吗？</h2>
**需要！**

因为移位操作跟这个数字是不是“10进制”无关，它操作的是**内存中真实的二进制结构**。

---
<br/>

**🧠 原因解释**

```swift
let timestamp: UInt32 = 1718783172  // 10进制
```

<br/>
Swift 内部都会将它存成二进制：

```
1718783172 (10进制) = 0x668D97B4 (16进制) = 01100110100011011001011110110100
```

<br/>
我们要的目标是把这个 **32 位整数拆成 4 个字节**（因为 BLE / 通信协议是按字节通信的），比如小端顺序：

```
[0xB4, 0x97, 0x8D, 0x66]
```

<br/> 

**✅ 所以移位 + `& 0xFF` 是必须的步骤**

这是把这个时间戳打包为 `[UInt8]` 的方式：

```swift
let timestamp: UInt32 = 1718783172
let bytes: [UInt8] = [
    UInt8((timestamp >> 0) & 0xFF),
    UInt8((timestamp >> 8) & 0xFF),
    UInt8((timestamp >> 16) & 0xFF),
    UInt8((timestamp >> 24) & 0xFF)
]
print(bytes)  // [180, 151, 141, 102]
```

<br/> 

**✅ 或者用 withUnsafeBytes（更优雅）**

```swift
let timestamp: UInt32 = 1718783172
let data = withUnsafeBytes(of: timestamp.littleEndian) { Data($0) }
print(data as NSData)  // <b4978d66>
```



***
<br/><br/><br/>
> <h2 id="Uint16、UInt32、UInt64转成Data类型">Uint16、UInt32、UInt64转成Data类型</h2>

可以为 `UInt16`、`UInt32`、`UInt64` 等整数类型封装一个统一的方法，通过 Swift 的 **协议扩展 + 泛型** 实现通用的字节序转换方法。把它们转换为字节数组或 `Data`。


<br/>

**✅ 推荐封装方式（以小端字节为例）**

```swift
import Foundation

protocol ByteConvertible {
    func toBytes(littleEndian: Bool) -> [UInt8]
    func toData(littleEndian: Bool) -> Data
}

extension ByteConvertible {
    func toData(littleEndian: Bool = true) -> Data {
        return Data(self.toBytes(littleEndian: littleEndian))
    }
}

extension UInt16: ByteConvertible {
    func toBytes(littleEndian: Bool = true) -> [UInt8] {
        let value = littleEndian ? self.littleEndian : self.bigEndian
        return [
            UInt8((value >> 0) & 0xFF),
            UInt8((value >> 8) & 0xFF)
        ]
    }
}

extension UInt32: ByteConvertible {
    func toBytes(littleEndian: Bool = true) -> [UInt8] {
        let value = littleEndian ? self.littleEndian : self.bigEndian
        return [
            UInt8((value >> 0) & 0xFF),
            UInt8((value >> 8) & 0xFF),
            UInt8((value >> 16) & 0xFF),
            UInt8((value >> 24) & 0xFF)
        ]
    }
}

extension UInt64: ByteConvertible {
    func toBytes(littleEndian: Bool = true) -> [UInt8] {
        let value = littleEndian ? self.littleEndian : self.bigEndian
        return (0..<8).map { i in
            UInt8((value >> (i * 8)) & 0xFF)
        }
    }
}
```

<br/>

**用法示例：**

```swift
let u16: UInt16 = 0x1234
let u32: UInt32 = 0x12345678
let u64: UInt64 = 0x1234567890ABCDEF

print(u16.toBytes())  // [0x34, 0x12]
print(u32.toData())   // <78563412>
print(u64.toBytes())  // [0xEF, 0xCD, 0xAB, 0x90, 0x78, 0x56, 0x34, 0x12]
```

<br/>
你也可以传 `littleEndian: false` 得到大端：

```swift
print(u16.toBytes(littleEndian: false)) // [0x12, 0x34]
```

<br/>

**✅ Bonus：统一反解析方法？**

当然也可以定义统一的 `init(fromBytes:)` 方法。








<br/><br/><br/>

***
<br/>

> <h1 id="Data在Int、字符串、结构体、json、图片、音频文件转化">Data在Int、字符串、结构体、json、图片、音频文件转化</h1>

**✅ 1. 整数 <-> `Data`**

**Int 转 Data（以小端方式存储）：**

```swift
let intValue: UInt32 = 4096
let intData = withUnsafeBytes(of: intValue.littleEndian) { Data($0) }
```

**Data 转 Int：**

```swift
let extractedInt = intData.withUnsafeBytes { $0.load(as: UInt32.self).littleEndian }
```

---

**✅ 2. 字符串 <-> `Data`**

**字符串转 `Data`（UTF-8 编码）：**

```swift
let str = "hello"
let strData = str.data(using: .utf8)!
```

**`Data` 转字符串：**

```swift
let backToString = String(data: strData, encoding: .utf8) ?? "<Invalid UTF-8>"
```

---

**✅ 3. 结构体 <-> `Data`**

你可以将结构体作为二进制打包，适用于 BLE 或 Socket：

```swift
struct Point: Codable {
    var x: Int32
    var y: Int32
}

let point = Point(x: 10, y: 20)

// 编码为二进制 Data（注意字节序）
let pointData = withUnsafeBytes(of: point) { Data($0) }

// 还原为结构体
let restoredPoint = pointData.withUnsafeBytes { $0.load(as: Point.self) }
```

> ⚠️ 注意：结构体成员必须是 **原始类型（C Layout Compatible）**。

---

**✅ 4. JSON 对象 <-> `Data`**

适合用在蓝牙传输小 JSON 配置、网络上传输等：

```swift
let dict: [String: Any] = ["name": "BLE", "version": 1]

// Dictionary -> Data
let jsonData = try JSONSerialization.data(withJSONObject: dict, options: [])

// Data -> Dictionary
let decoded = try JSONSerialization.jsonObject(with: jsonData, options: []) as? [String: Any]
```

---

**✅ 5. 图片 <-> `Data`**

```swift
import UIKit

let image = UIImage(named: "example")!
let imageData = image.pngData()!  // 或 jpegData(compressionQuality:)

let backToImage = UIImage(data: imageData)
```

---

**✅ 6. 音频文件 <-> `Data`**

```swift
let url = Bundle.main.url(forResource: "sound", withExtension: "wav")!
let soundData = try Data(contentsOf: url)

// 可通过 AVAudioPlayer 播放或蓝牙传输
```

---

**🔚 总结：常见类型与 `Data` 转换速查表**

| 类型      | 转为 Data 示例                                | 从 Data 还原                            |
| ------- | ----------------------------------------- | ------------------------------------ |
| Int     | `withUnsafeBytes(of: val)`                | `.withUnsafeBytes { $0.load(...) }`  |
| String  | `str.data(using: .utf8)`                  | `String(data: ..., encoding: .utf8)` |
| 结构体     | `withUnsafeBytes(of: struct)`             | `.load(as: Struct.self)`             |
| JSON    | `JSONSerialization.data(withJSONObject:)` | `JSONSerialization.jsonObject(...)`  |
| UIImage | `image.pngData()`                         | `UIImage(data: ...)`                 |
| 音频文件    | `try Data(contentsOf: url)`               | 播放或传输时使用                             |



<br/><br/>
> <h3 id="结构体进行封装字节数据并拼接">结构体进行封装字节数据并拼接</h3>

**`需求:`** 定义一个结构体表示这 6 个字节的数据格式，将结构体转换为字节数据用于蓝牙发送。

- **✅ 步骤概览：**
	1. 定义结构体 `BLEHeader` 表示 6 字节字段。
	2. 提供一个方法，将 `BLEHeader` 转换为 `Data`。
	3. 封装为 `@objcMembers` 的类，使其能在 OC 中调用。

---

 **✅ Swift 中定义结构体并封装转换逻辑**

```swift
import Foundation

// 定义 BLEHeader 结构体，表示数据结构
struct BLEHeader {
    var sequence: UInt16
    var type: UInt8
    var mode: UInt8
    var length: UInt16
}

// 封装为 OC 可调用类
@objcMembers
public class BLEPacketBuilder: NSObject {
    
    public static func buildPacket(sequence: UInt16, type: UInt8, mode: UInt8, length: UInt16) -> Data {
        let header = BLEHeader(sequence: sequence, type: type, mode: mode, length: length)
        return serialize(header)
    }
    
    private static func serialize(_ header: BLEHeader) -> Data {
        var data = Data()
        
        var seq = header.sequence.littleEndian
        var len = header.length.littleEndian
        
        data.append(UnsafeBufferPointer(start: &seq, count: 1))       // 2 字节 sequence
        data.append(header.type)                                      // 1 字节 type
        data.append(header.mode)                                      // 1 字节 mode
        data.append(UnsafeBufferPointer(start: &len, count: 1))       // 2 字节 length
        
        return data
    }
}
```

---

**Swift 中调用：**

```swift
let packet = BLEPacketBuilder.buildPacket(sequence: 0x1234, type: 0x01, mode: 0x02, length: 0x0040)
// 发送 packet 到蓝牙
```

<br/>

**Objective-C 中调用：**

```objc
NSData *packet = [BLEPacketBuilder buildPacketWithSequence:0x1234
                                                      type:0x01
                                                      mode:0x02
                                                    length:0x0040];
// 发送 packet 到蓝牙设备
```

---

**✅ 结构体的作用:**

虽然结构体 `BLEHeader` 本身不是 Objective-C 兼容类型（它不继承自 NSObject），但它作为内部封装的结构，可以清晰地表达数据模型，同时封装为类方法提供 OC 调用方式。

---

如果你希望在 OC 中也能直接定义和使用 `BLEHeader` 结构体（比如跨语言直接构造结构体），那就需要定义成 C 结构体并结合 Swift 的桥接。但多数情况下，用 Swift 管理内部结构体再封装为类方法是最实用的做法。


***

你也可以传入**十进制**数据，和传入十六进制本质上是一样的，Swift 中的 `UInt8`、`UInt16` 等整数类型都支持十进制、十六进制、八进制、二进制等多种写法。

---

**这两种写法是完全等效的：**

```swift
// 十六进制写法
let packet1 = BLEPacketBuilder.buildPacket(sequence: 0x1234, type: 0x01, mode: 0x02, length: 0x0040)

// 十进制写法（等效于上面的 0x1234 = 4660，0x40 = 64）
let packet2 = BLEPacketBuilder.buildPacket(sequence: 4660, type: 1, mode: 2, length: 64)

// 任意十进制写法都可以，比如你写的：
let packet3 = BLEPacketBuilder.buildPacket(sequence: 86, type: 12, mode: 34, length: 66)
```



***
<br/><br/><br/>
> <h2 id="固定范围字节转化成UInt类型">固定范围字节转化成UInt类型</h2>

**从任意 `Data` 中按指定范围读取 1\~4 个字节，并根据字节序（大端/小端）转换为 `UInt` 类型。**

---

**✅ 通用字节读取工具（支持 UInt8 / UInt16 / UInt32）**

```swift
enum Endian {
    case little
    case big
}

/// 从 Data 中读取整数（支持 1～4 字节，小端或大端）
/// - Parameters:
///   - data: 原始 Data
///   - range: 要读取的字节范围（如 0..<2）
///   - endian: 字节序（默认小端）
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

---

**✅ 使用示例：**

```swift
let data = Data([0x78, 0x56, 0x34, 0x12])

// 读取第0~1字节，小端 → UInt16
let val1 = readUInt(from: data, range: 0..<2, endian: .little)
print("前2字节（小端） UInt16: \(val1)")  // 输出：22136

// 读取第2~4字节，大端 → UInt24（转为 UInt）
let val2 = readUInt(from: data, range: 1..<4, endian: .big)
print("第1~3字节（大端） UInt24: \(val2)")  // 输出：5649428

// 读取全部4字节，小端 → UInt32
let val3 = readUInt(from: data, range: 0..<4)
print("全部4字节（小端） UInt32: \(val3)")  // 输出：305419896
```

---

## ✅ 支持范围：

| 字节数 | 返回类型       | 实际 Swift 类型      |
| --- | ---------- | ---------------- |
| 1   | `UInt8`    | `UInt`（<= 255）   |
| 2   | `UInt16`   | `UInt`（<= 65535） |
| 3   | `UInt24`模拟 | `UInt`（<= 16M）   |
| 4   | `UInt32`   | `UInt`（<= 4G）    |

> ❗如果你需要严格区分返回类型（如必须是 `UInt16`、`UInt32`），可以再加一个泛型版本，我也可以为你写。













