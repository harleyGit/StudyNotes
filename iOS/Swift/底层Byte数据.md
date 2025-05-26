> <h1 id=""></h1>
- [**3个底层字节处理数据类**](#3个底层字节处理数据类)
	- [开发蓝牙数据工具选择](#开发蓝牙数据工具选择)
	- [UnsafeMutableRawPointer和Data的关系](#UnsafeMutableRawPointer和Data的关系)
- [**类型Data**](#类型Data)
	- [不同字节数的数字拼接](#不同字节数的数字拼接)
		- [传入的值为十进制数转成不同字节数值](#传入的值为十进制数转成不同字节数值)
		- [将十进制64存储为4字节](#将十进制64存储为4字节)
		- [固定范围字节转化成UInt类型](#固定范围字节转化成UInt类型)
- [**类型UInt8**](#类型UInt8)
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













