- [**‌CocoaMQTT5**](#CocoaMQTT5介绍)
- [MQTT核心概念](#MQTT核心概念)
- [CocoaMQTT5Delegate代理方法](#CocoaMQTT5Delegate代理方法)
- GPT资料：https://chatgpt.com/c/6965ee06-3a08-8323-9f66-246d7a755526


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

 ***
<br/>

**为什么不用 HTTP / WebSocket，而用 MQTT**

<br/>

- **与 HTTP 对比**

| 维度     | HTTP       | MQTT    |
| ------ | ---------- | ------- |
| 通信模型   | 请求-响应      | 发布-订阅   |
| 连接方式   | 短连接 / 半长连接 | **长连接** |
| 流量消耗   | 较大         | **极小**  |
| 实时性    | 一般         | **高**   |
| 适合 IoT | ❌          | ✅       |

HTTP 不适合 **高频、小数据、实时性强** 的场景。


<br/>

- **与 WebSocket 对比**

| 维度         | WebSocket | MQTT             |
| ---------- | --------- | ---------------- |
| 协议复杂度      | 中         | **非常简单**         |
| QoS（消息可靠性） | 无内建       | **内建 QoS 0/1/2** |
| 离线消息       | 需自实现      | **原生支持**         |
| 断线重连       | 自己处理      | **协议级支持**        |

**结论**：
WebSocket 偏通用实时通信；
MQTT 是 **为设备通信量身定制**。

<br/>

**典型使用场景**

* IoT 设备控制（智能家居 App 控制灯、空调、摄像头等）。
* 实时消息推送/聊天。
* 设备状态上报（温湿度、传感器数据）。
* 移动端作为「轻量客户端」接入 MQTT 云平台（如 EMQX、Mosquitto、HiveMQ）。

***
<br/><br/><br/>
> <h2 id="MQTT核心概念">MQTT 的核心概念</h2>

<br/>

- **1️⃣ 发布 / 订阅模型**
	* **发布（Publish）**：向某个 Topic 发送消息
	* **订阅（Subscribe）**：监听某个 Topic 的消息

> 发布者和订阅者互不感知，全部由 Broker 中转

```text
iOS App ----> Broker ----> Device
        publish           subscribe
```

<br/> 

- **2️⃣ Topic（主题）**

类似路径：

```text
home/livingroom/light
device/12345/status
user/1001/notify
```

支持通配符：

* `+`：单层
* `#`：多层

<br/>

-  **3️⃣ QoS（消息可靠级别）**

| QoS | 含义         |
| --- | ---------- |
| 0   | 最多一次（可能丢）  |
| 1   | 至少一次（可能重复） |
| 2   | 仅一次（最可靠）   |

---
<br/>

**MQTT 是怎么用的⁉️⁉️**

<br/> 

**Swift 并无官方 MQTT，通常使用第三方库**

常见库：

* **CocoaMQTT（最常用）**
* MQTTClient（偏底层）
<br/>

**在 iOS 中典型职责**

Swift MQTT 客户端一般负责：

* 建立与 Broker 的 TCP / TLS 连接
* 订阅 Topic
* 发布控制指令
* 监听设备消息
* 断线重连
* 心跳保活

---
<br/>

**简化理解一句话**

> **Swift 中的 MQTT，就是让 iOS App 像“设备一样”，通过 MQTT 协议与 IoT 后端或设备进行实时通信。**

---
<br/>

**一个典型 iOS + MQTT 架构**

```text
┌─────────┐        MQTT         ┌──────────┐
│ iOS App │ <----------------> │  Broker  │
└─────────┘                     └──────────┘
                                      ↑
                                      │
                               ┌──────────┐
                               │  Device  │
                               └──────────┘
```



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



<br/><br/><br/>

***
<br/>

> <h1 id="CocoaMQTT5Delegate代理方法">CocoaMQTT5Delegate代理方法</h1>


```swift
extension MQTTManager: CocoaMQTTDelegate {

    func mqtt(_ mqtt: CocoaMQTT, didConnectAck ack: CocoaMQTTConnAck) {
        print("MQTT connected: \(ack)")
        
        // 示例：订阅设备状态
        subscribe(topic: "device/+/status")
    }

    func mqtt(_ mqtt: CocoaMQTT, didReceiveMessage message: CocoaMQTTMessage, id: UInt16) {
        let topic = message.topic
        let payload = message.string ?? ""

        print("📩 [\(topic)] \(payload)")

        // 在这里分发给 ViewModel / Combine / Notification
    }

    func mqtt(_ mqtt: CocoaMQTT, didDisconnect reason: CocoaMQTTDisconnectReason, error: Error?) {
        print("MQTT disconnected")
    }

    func mqtt(_ mqtt: CocoaMQTT, didPublishMessage message: CocoaMQTTMessage, id: UInt16) {}
    func mqtt(_ mqtt: CocoaMQTT, didSubscribeTopics success: NSDictionary, failed: [String]) {}
    func mqtt(_ mqtt: CocoaMQTT, didUnsubscribeTopics topics: [String]) {}
    func mqttDidPing(_ mqtt: CocoaMQTT) {}
    func mqttDidReceivePong(_ mqtt: CocoaMQTT) {}
}
```


<br/>

- **1. didConnectAck`**

```swift
func mqtt(_ mqtt: CocoaMQTT, didConnectAck ack: CocoaMQTTConnAck)
```

- 触发时机
	* **客户端与 MQTT Broker 建立 TCP + MQTT 协议连接完成后**
	* 收到 **CONNACK** 报文时触发
	* 表示“连接结果确认”，不是开始 connect 的回调

- **关键参数**
	* `ack`: `CocoaMQTTConnAck`
	  * `.accept`：连接成功
	  * `.badUsernameOrPassword`
	  * `.notAuthorized`
	  * `.serverUnavailable`
	  * 等

- **典型用途**
	* 判断是否真正连上 Broker
	* **在此之后才能：**
	  * `subscribe`
	  * `publish`
	* 初始化会话状态

<br/>

**生产级实践**

```swift
if ack == .accept {
    subscribe(topic: "device/+/status")
    subscribe(topic: "user/\(userId)/notify")
} else {
    // 登录失败、鉴权失败、账号异常
}
```

> ⚠️ 不要在 `connect()` 之后立刻订阅，必须等到 `didConnectAck(.accept)`

<br/>

- **2.`didReceiveMessage`**

```swift
func mqtt(_ mqtt: CocoaMQTT,
          didReceiveMessage message: CocoaMQTTMessage,
          id: UInt16)
```
- **触发时机**
	* 客户端 **收到 Broker 推送的 PUBLISH 消息**
	* 来自你订阅过的 Topic

- **关键参数**
	* `message.topic`：消息主题
	* `message.payload` / `message.string`
	* `message.qos`
	* `message.retained`

- **典型用途**
	* 处理设备上报
	* 处理状态变化
	* 处理告警 / 控制回执

- **生产级实践**
	- **只做分发，不直接写业务逻辑**

```swift
mqttEventSubject.send(
    MQTTEvent(topic: message.topic, payload: payload)
)
```

然后：

* ViewModel（Combine）
* 或 NotificationCenter
* 或 Redux / Store

> ⚠️ 不要在这里做 JSON 大解析或耗时操作

<br/>

- **3. `didDisconnect`**

```swift
func mqtt(_ mqtt: CocoaMQTT,
          didDisconnect reason: CocoaMQTTDisconnectReason,
          error: Error?)
```

- **触发时机**
* MQTT 连接断开
	* 网络断开
	* Broker 主动断开
	* 心跳超时
	* 调用 `disconnect()`

- **关键参数**
	* `reason`：
		* `.normal`
		* `.error`
		* `.timeout`
		* `.keepAliveTimeout`
		* `error`：底层 Socket 错误

- **典型用途**
	* 重连策略
	* 更新 UI 状态
	* 设备离线处理

- **生产级实践**

```swift
scheduleReconnectWithBackoff()
```

常见策略：

* 指数退避（1s / 2s / 4s / 8s）
* 前后台切换暂停重连
* 网络可达性恢复后再重连

<br/>

- **4. `didPublishMessage`**

```swift
func mqtt(_ mqtt: CocoaMQTT,
          didPublishMessage message: CocoaMQTTMessage,
          id: UInt16)
```

- **触发时机**
	* **客户端成功将消息发送给 Broker**
	* 仅表示“已发出”，不是对端已处理

- **典型用途**
	* 调试日志
	* QoS1 / QoS2 状态追踪
	* 消息发送统计

- **生产级实践**
	* 很多项目 **可以留空**
	* 对 IPC 控制类消息有用

<br/>

- **5. `didSubscribeTopics`**

```swift
func mqtt(_ mqtt: CocoaMQTT,
          didSubscribeTopics success: NSDictionary,
          failed: [String])
```

- **触发时机**
	* Broker 对 SUBSCRIBE 请求返回 SUBACK

- **关键参数**
	* `success`：订阅成功的 topic → qos
	* `failed`：订阅失败的 topic

- **典型用途**
	* 确认关键 topic 是否成功订阅
	* 容错处理

- **生产级实践**

```swift
if failed.contains("device/+/status") {
    // 记录错误，可能权限不足
}
```

<br/>

- **`didUnsubscribeTopics`**

```swift
func mqtt(_ mqtt: CocoaMQTT,
          didUnsubscribeTopics topics: [String])
```

- **触发时机**
	* UNSUBSCRIBE 成功后

- **典型用途**
	* 页面销毁时取消订阅
	* 切换设备 / 用户

- **生产级实践**
	* 多设备切换时清理旧订阅

<br/> 

- **`mqttDidPing`**

```swift
func mqttDidPing(_ mqtt: CocoaMQTT)
```

- **触发时机**
	* 客户端发送 **PINGREQ**（心跳）

- **典型用途**
	* 调试
	* 连接存活监控

- **生产级建议**
	* 一般不需要处理
	* 不要写业务逻辑

<br/>

- **`mqttDidReceivePong`**

```swift
func mqttDidReceivePong(_ mqtt: CocoaMQTT)
```

- **触发时机**
	* Broker 回复 **PINGRESP**

- **典型用途**
	* 判断连接是否健康

- **生产级实践**
	* 可作为“活跃连接”标记
	* 心跳异常 → 触发重连

<br/>

- **企业级 IPC / IoT 项目中的推荐分层**

```text
CocoaMQTTDelegate
        ↓
MQTTManager（只做连接 & 分发）
        ↓
Combine / Notification
        ↓
ViewModel / DeviceManager
        ↓
UI / 设备状态机
```


<br/><br/><br/>

***
<br/>

> <h1 id=""></h1>



<br/><br/><br/>

***
<br/>

> <h1 id=""></h1>



<br/><br/><br/>

***
<br/>

> <h1 id=""></h1>



<br/><br/><br/>

***
<br/>

> <h1 id=""></h1>


