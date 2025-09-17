- [**‌CocoaMQTT5**](#CocoaMQTT5介绍)


<br/><br/><br/>

***
<br/>

> <h1 id="CocoaMQTT5介绍">CocoaMQTT5介绍</h1>

`CocoaMQTT5` 是一个 **用 Swift 实现的 MQTT v5 客户端库**，主要用在 iOS / macOS / tvOS / watchOS 应用里，用来和 MQTT 服务器（Broker）进行通信。

<br/>

**🌐 MQTT 是什么**

* **MQTT（Message Queuing Telemetry Transport）** 是一种轻量级的发布/订阅（pub/sub）消息协议，常用于物联网 (IoT)、实时消息推送等场景。
* 特点：
	* 协议头很小，带宽占用低。
	* 长连接，基于 TCP。
	* 通过「主题（topic）」进行消息路由。
	* 支持 QoS (Quality of Service) 级别，保证消息可靠性。

<br/>

**🍏 CocoaMQTT5 的作用**

`CocoaMQTT5` 是 [CocoaMQTT](https://github.com/emqx/CocoaMQTT) 的 **MQTT 5.0 版本**：

* 封装了 MQTT 5.0 协议的细节，提供易用的 Swift API。
* 让你的 App 轻松：

  * 连接/断开 MQTT Broker。
  * 发布消息（publish）。
  * 订阅/取消订阅主题（subscribe/unsubscribe）。
  * 监听服务器下发的消息。
  * 处理 MQTT 5 的新特性（属性、原因码、会话过期等）。

<br/>

**典型使用场景**

* IoT 设备控制（智能家居 App 控制灯、空调、摄像头等）。
* 实时消息推送/聊天。
* 设备状态上报（温湿度、传感器数据）。
* 移动端作为「轻量客户端」接入 MQTT 云平台（如 EMQX、Mosquitto、HiveMQ）。

<br/>

**示例**

```swift
import CocoaMQTT5

// 1️⃣ 创建客户端
let clientID = "iOSClient-\(Int(Date().timeIntervalSince1970))"
let mqtt = CocoaMQTT5(clientID: clientID, host: "broker.emqx.io", port: 1883)

// 2️⃣ 设置回调
mqtt.didReceiveMessage = { mqtt, message, id in
    print("收到消息: \(message.string ?? "") 来自主题 \(message.topic)")
}

mqtt.didConnectAck = { mqtt, ack in
    print("连接成功: \(ack)")
}

// 3️⃣ 连接 Broker
mqtt.connect()

// 4️⃣ 订阅主题
mqtt.subscribe("home/temperature", qos: .qos1)

// 5️⃣ 发布消息
mqtt.publish("home/temperature", withString: "23.5℃", qos: .qos1)

// 6️⃣ 断开连接
mqtt.disconnect()
```

> 上面代码演示了最核心的流程：**连接 → 订阅 → 发布 → 接收消息 → 断开**。

<br/>

**总结**

| 方面   | CocoaMQTT5 的作用               |
| ---- | ---------------------------- |
| 协议   | 实现 MQTT 5.0 客户端              |
| 运行环境 | iOS / macOS / tvOS / watchOS |
| 功能   | 连接 Broker、订阅/发布消息、接收推送、管理会话  |
| 适用场景 | IoT、实时推送、状态同步、设备监控           |

如果你只需要 MQTT 3.1.1 的功能，可以用 `CocoaMQTT`；
如果要支持 **MQTT v5** 的属性/增强功能，就选 `CocoaMQTT5`。


