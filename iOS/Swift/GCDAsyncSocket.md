- [**‌介绍**](#介绍)
- [**‌简单用法**](#简单用法)


<br/><br/><br/>

***
<br/>

> <h1 id="介绍">介绍</h1>

`GCDAsyncSocket` 是一个第三方的 **Socket 网络通信库**（来自 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)），主要用于 **TCP/UDP 通信**。

它基于 **GCD（Grand Central Dispatch）** 封装，使用起来比原生的 BSD Socket 更简单，适合做 **即时通讯、局域网通信、IoT 设备对接** 等。

<br/>

- **用途：**
	* 建立 TCP 连接（客户端 / 服务端）
	* 发送、接收数据（异步、非阻塞）
	* 设置超时、自动重连
	* 使用 Delegate 回调方式获取事件（连接成功、断开、收到消息等）




<br/><br/><br/>

***
<br/>

> <h1 id="简单用法">简单用法</h1>

**1.安装**

```ruby
pod 'CocoaAsyncSocket'
```

<br/>

**2.导入**

```swift
import CocoaAsyncSocket
```

<br/>

**客户端连接服务器并收发数据**

```swift
import UIKit
import CocoaAsyncSocket

class TCPClient: NSObject, GCDAsyncSocketDelegate {
    var socket: GCDAsyncSocket!

    override init() {
        super.init()
        socket = GCDAsyncSocket(delegate: self, delegateQueue: DispatchQueue.main)
    }

    // 连接服务器
    func connect(host: String, port: UInt16) {
        do {
            try socket.connect(toHost: host, onPort: port, withTimeout: 5)
            print("正在连接 \(host):\(port)...")
        } catch {
            print("连接失败: \(error)")
        }
    }

    // 发送数据
    func send(message: String) {
        if let data = message.data(using: .utf8) {
            socket.write(data, withTimeout: 5, tag: 0)
        }
    }

    // MARK: - GCDAsyncSocketDelegate
    func socket(_ sock: GCDAsyncSocket, didConnectToHost host: String, port: UInt16) {
        print("连接成功: \(host):\(port)")
        // 连接后开始读取数据（-1 表示不超时）
        socket.readData(withTimeout: -1, tag: 0)
    }

    func socket(_ sock: GCDAsyncSocket, didRead data: Data, withTag tag: Int) {
        if let msg = String(data: data, encoding: .utf8) {
            print("收到消息: \(msg)")
        }
        // 继续监听下一条消息
        socket.readData(withTimeout: -1, tag: 0)
    }

    func socketDidDisconnect(_ sock: GCDAsyncSocket, withError err: Error?) {
        print("断开连接: \(err?.localizedDescription ?? "无错误信息")")
    }
}
```

<br/>

**使用示例**

```swift
let client = TCPClient()
client.connect(host: "127.0.0.1", port: 8888)

// 连接成功后就可以发消息了
client.send(message: "Hello Server!")
```

<br/>

**总结**

* `GCDAsyncSocket` = 封装好的异步 **Socket 通信库**
* 主要用来 **TCP/UDP 连接、收发消息**
* 使用 **Delegate 回调** 方式，简单高效


