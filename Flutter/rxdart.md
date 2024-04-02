> <h1 id=''></h1>
- [**简介**](#简介)
- [BehaviorSubject](#BehaviorSubject)
	- [用法](#用法)




<br/>

***

<br/><br/><br/>


> <h1 id='简介'>简介</h1>

BehaviorSubject 不是 Flutter 的库，而是属于 ReactiveX (Rx) 库的一部分，尤其是 RxJS（针对 JavaScript/TypeScript）和 RxDart（针对 Dart/Flutter）。在 Flutter 开发中，如果您需要使用 BehaviorSubject，您应当引用的是 RxDart 库，这是一个为 Dart 语言提供了 Rx 风格的响应式编程特性的库。

RxDart 提供了包括 BehaviorSubject 在内的各种 Observable、Observer、Subject 和操作符，这些可以帮助您在 Flutter 应用中构建复杂的异步数据流和状态管理逻辑。在 Flutter 开发中，BehaviorSubject 通常用于实现状态管理、事件总线（Event Bus）、数据同步等场景，尤其是在那些需要维护和共享一个可变状态，并且希望新订阅者能够立即获取到当前状态的场合。

如果您想在 Flutter 项目中使用 BehaviorSubject，您需要按照以下步骤操作：

```
dependencies:
  rxdart: ^<版本号>  # 替换为当前最新的稳定版本号
```



<br/>

***
<br/><br/><br/>

> <h1 id='BehaviorSubject'>BehaviorSubject</h1>


BehaviorSubject 是 RxDart 库中的一种特殊类型的 Subject，它继承自 Subject 类并实现了 Observable 接口。BehaviorSubject 具有以下核心特点和使用方式：

<br/>

**1.主要特点：**

初始化时需要一个初始值： 创建一个 BehaviorSubject 实例时，需要提供一个初始值。这个值会立即发送给所有已经订阅的观察者，并成为 BehaviorSubject 内部的当前值。


<br/>

**2.保存并提供最新值：** 

BehaviorSubject 会始终保持一个当前值（latest value）。每当有新的订阅者加入时，它们会立即收到这个当前值，而不仅仅是从订阅时刻起的新值。这意味着即使在订阅之后 BehaviorSubject 发出了新值，新订阅者也能立即接收到这个最新的值。

<br/>

**3.值的更新：** 

通过调用 add 或 sink.add 方法向 BehaviorSubject 发送新的值。这会更新 BehaviorSubject 的内部状态，并触发所有已订阅的观察者收到这个新值。

<br/>

**4.完成（Complete）与错误（Error）：** 

同其他 Observables，BehaviorSubject 可以通过调用 close 或 sink.close 方法来标记数据流的完成，此时所有观察者会收到完成通知，之后不再接收任何值。同样，通过调用 addError 或 sink.addError 方法可以发送一个错误通知，观察者会收到错误对象并终止订阅


<br/><br/><br/>

> <h2 id='用法'>用法</h2>


**使用场景：**

**1.状态管理：**

<br/>

在 Flutter 应用中，BehaviorSubject 常用于实现组件的状态管理。它能保存组件的当前状态，并允许其他组件或服务订阅状态变化，即时获取最新的状态。

<br/>

**2.数据同步：**

在多视图或组件间需要共享同一数据源，并确保新加入的组件能立即获得当前数据时，BehaviorSubject 是理想的选择。例如，一个实时更新的计数器，新打开的窗口或面板应显示当前的计数值，而非从零开始。

<br/>

**3.路由参数传递：**

在单页应用（SPA）中，可以利用 BehaviorSubject 来传递路由参数，新激活的路由组件订阅后可以直接获取到当前的路由参数，无需关心数据的发送时机。



<br/>

**Demo:**

```
import 'package:rxdart/rxdart.dart';

void main() {
  // 创建一个 BehaviorSubject，指定初始值为 'Initial Value'
  final BehaviorSubject<String> subject = BehaviorSubject.seeded('Initial Value');

  // 订阅 BehaviorSubject
  subject.stream.listen((value) {
    print('Subscriber A received value: $value');
  });

  // 发送新值
  subject.add('New Value');

  // 新订阅者也会立即收到最近的值（这里是 'New Value'）
  subject.stream.listen((value) {
    print('Subscriber B received value: $value');
  });

  // 发送另一个新值
  subject.add('Another New Value');

  // 输出结果：
  // Subscriber A received value: Initial Value
  // Subscriber A received value: New Value
  // Subscriber B received value: New Value
  // Subscriber A received value: Another New Value
  // Subscriber B received value: Another New Value
}
```

- **在这个例子中：**
	- 我们创建了一个 BehaviorSubject，初始值为 'Initial Value'。
	
	- 订阅者 A 首先订阅了 subject，立即收到了初始值。
	
	- 向 subject 发送 'New Value'，订阅者 A 收到了更新。
	
	- 订阅者 B 在 'New Value' 发送之后订阅，但它依然立即收到了 'New Value'，因为这是 BehaviorSubject 当前保存的最新值。
	
	- 再次向 subject 发送 'Another New Value'，两个订阅者都收到了这个更新。


<br/>


综上所述，BehaviorSubject 是 RxDart 库中用于 Flutter 开发的响应式编程工具之一，它特别适用于那些需要维护和共享一个可变状态，并且希望新订阅者能够立即获取到当前状态的场景。通过订阅 BehaviorSubject 的 stream，各个组件或服务可以实时响应状态变化，从而实现高效的异步数据流和状态管理。


