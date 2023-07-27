> 
- [**项目开始**](#项目开始)
	- [环境配置](#环境配置)
	- [项目初始化](#项目初始化)
	- [打包](#打包)
- [**iOS原生模块**](#iOS原生模块)
	- [原生模块给JS发送消息](#原生模块给JS发送消息)
	- [RCT_EXPORT_MODULE](#RCT_EXPORT_MODULE)
	- [RN调用原生](#RN调用原生)
		- 
- [**语法基础**](#语法基础)
	- [样式声明](#样式声明)
	- [弹性布局](#弹性布局)
- [**组件**](#组件)
	- [3大平台组件对比](#3大平台组件对比)
- [**架构原理**](#架构原理)
	- [线程模型](#线程模型)
	- [启动过程](#启动过程)
	- [新旧架构对比](#新旧架构对比)






<br/>

***
<br/><br/>

> <h1 id='项目开始'>项目开始</h1>

<br/><br/>

> <h2 id='环境配置'>环境配置</h2>

- 基础环境:
	- 安装Node
	- 安装yarn(其实npm也可以,不过yarn速度会更快点)

- iOS环境配置:
	- Xcode
	- CocoaPods
	- watchman安装
		- 主要是用来调试:
			- Command+B:弹窗选择刷新、debug
			- `brew install watchman`


<br/><br/>

> <h2 id='项目初始化'>项目初始化</h2>



- 初始化项目
	- react-native init xxxproject
	- cd xxxproject
	- Android:yarn android
	- iOS
		- cd iOS
		- pod install
		- cd ..
		- yarn ios



- 好用的插件:
	- vscdoe
![rn0_00.png](./../Pictures/rn0_00.png)
	- 快捷命令:
		- rnc(react native class): 快速创建rn类
			- ![rn0_01.png](./../Pictures/rn0_01.png)
		- rnf(react native function): 类组件
			- ![rn0_02.png](./../Pictures/rn0_02.png)



- 调试
	- 模拟器调试
		- Android:点击模拟器后,然后快捷键:ctrl+B,然后选中debug,会跳转到浏览器
		- iOS: Command+B(M)





<br/><br/>


> <h2 id='打包'>打包</h2>


![rn0_15.png](./../Pictures/rn0_15.png)

在上线时分别运行命令:

```
npm run ios-start 
```

注意要修改渠道号,如下:

![rn0_16.png](./../Pictures/rn0_16.png)


会将代码打包进RN项目中iOS文件夹中的一个bunle文件夹中!

<br/>

这个时候,iOS打包就像往常一样,打正式包了!







<br/>

***

<br/><br/>

> <h1 id='iOS原生模块'>iOS原生模块</h1>


<br/><br/>

> <h2 id='原生模块给JS发送消息'>原生模块给JS发送消息</h2>


[**给 JavaScript 端发送事件**](https://www.jianshu.com/p/7973a2b91f24)

&emsp; 即使没有被 JavaScript 调用，原生模块也可以给 JavaScript 发送事件通知。最好的方法是继承**RCTEventEmitter**，实现suppportEvents方法并调用self sendEventWithName:。




<br/><br/>



># <h2 id='RCT_EXPORT_MODULE'>[RCT_EXPORT_MODULE](https://reactnative.cn/docs/native-modules-ios)</h2>

&emsp; 有时候 App 需要访问平台 API，但 React Native 可能还没有相应的模块封装；或者你需要复用 Objective-C、Swift 或 C++代码，而不是用 JavaScript 重新实现一遍；又或者你需要实现某些高性能、多线程的代码，譬如图片处理、数据库、或者各种高级扩展等等。

&emsp; 我们把 React Native 设计为可以在其基础上编写真正的原生代码，并且可以访问平台所有的能力。这是一个相对高级的特性，我们并不认为它应当在日常开发的过程中经常出现，但具备这样的能力是很重要的。如果 React Native 还不支持某个你需要的原生特性，你应当可以自己实现该特性的封装。

<br/>

我们就可以使用上述的宏将iOS中的类作为模块导出来给rn使用!







<br/><br/>

># <h2 id='RN调用原生'>[RN调用原生](https://juejin.cn/post/6965082621801955364)</h2>



<br/>

我们在`NativeAcc.h`中有一段代码:

```
#import <Foundation/Foundation.h>
#import <React/RCTBridgeModule.h>

NS_ASSUME_NONNULL_BEGIN

///遵守RCTBridgeModule协议后,在.m文件导出后就可以在RN中的js进行引用然后使用了
@interface NativeAcc : NSObject<RCTBridgeModule>

@end

NS_ASSUME_NONNULL_END

```

<br/>

然后在原生iOS的`NativeAcc.m`文件中有

```
RCT_EXPORT_MODULE(AccModule)
```

其实这是一个导出模块,可以理解为注册到到RN中的一个类或者模块,然后在RN中的js文件中就可以通过一个模块直接使用了,如下:

```
///在AccModule.js中的使用
const startGetNetworkSpeetTestInfo =()=>{
    NativeModules.AccModule.startGetNetworkSpeetTestInfo(); //开始测速
}
```



<br/>



- **iOS端如何操作**
	- 创建一个类，然后遵循协议
	- 使用RCT_EXPORT_MODULE导出模块
	- 使用RCT_EXPORT_METHOD导出异步方法
	- RCT_EXPORT_SYNCHRONOUS_TYPED_METHOD导出同步方法






<br/>

***
<br/><br/>

> <h1 id=''></h1>


<br/><br/>

> <h2 id='样式声明'>样式声明</h2>

![rn0_03.png](./../Pictures/rn0_03.png)



<br/><br/>

> <h2 id='弹性布局'>弹性布局</h2>


- web的弹性布局

![rn0_04.png](./../Pictures/rn0_04.png)


<br/>

- Rn的弹性布局:

![rn0_05.png](./../Pictures/rn0_05.png)



<br/>

***
<br/><br/>

> <h1 id='组件'>组件</h1>
 
 
 <br/><br/>
 
 > <h2 id='3大平台组件对比'>3大平台组件对比</h2>
 
 RN生成的组件会生成对应的android和iOS的组件,如下图:
 
![rn0_06.png](./../Pictures/rn0_06.png)



<br/>

3大平台组件比对:

![rn0_07.png](./../Pictures/rn0_07.png)





<br/>

***
<br/><br/>

> <h1 id='架构原理'>架构原理</h1>


![rn0_08.png](./../Pictures/rn0_08.png)



<br/>


<br/><br/>

># <h2 id='线程模型'>**线程模型**</h2>




![rn0_09.png](./../Pictures/rn0_09.png)





<br/><br/>

> <h2 id='启动过程'>启动过程</h2>

![rn0_10.png](./../Pictures/rn0_10.png)



<br/><br/>

> <h2 id='新旧架构对比'>新旧架构对比</h2>

![rn0_11.png](./../Pictures/rn0_11.png)


<br/>

- **1.核心模块对比:**

![rn0_12.png](./../Pictures/rn0_12.png)


<br/>

- **2.CodeGen**

![rn0_13.png](./../Pictures/rn0_13.png)


<br/>

<br/>


<br/>













