># Observer(观察者)、Observable(可观察序列)
&emsp;  核心理解就是一个`观察者(Observer)`订阅一个`可观察序列(Observable)`,`观察者(Observer)`对`可观察序列(Observable)`发射的数据或数据序列作出响应。

&emsp; 可观察序列存在三种情况：发射数据(Next)、遇到问题(Error)、发射完成(Completed),也就是3个事件
```
enum Event<Element>  {
    case Next(Element)      // 序列的下一个元素
    case Error(ErrorType)   // 序列因为某些错误终止
    case Completed          // 正常的序列技术
}
```

<br/>
**`观察者 (Observer)`**
观察者需要一个订阅序列的功能,如下：
```
class Observable<Element> {
    func subscribe(observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(event: Event<Element>)
}
```
通过这个 `subscribe` 来订阅序列。这里就涉及到序列的“冷”“热”：

- 冷：只有当有观察者订阅这个序列时，序列才发射值；
- 热：序列创建时就开始发射值。



<br/>
#`AsyncSubject`
```
        let disposeBag = DisposeBag()
        let subject = AsyncSubject<String>()
        
        subject.subscribe{
            print("subscription: 1 Event:", $0)
        }.disposed(by: disposeBag)
        
        subject.onNext("🐩")
        subject.onNext("🐶")
        subject.onNext("🐱")
        subject.onNext("🥜")
        subject.onCompleted()
```
打印：
`subscription: 1 Event: next(🥜)`
`subscription: 1 Event: completed`


<br/>
***
<br/>
># Subject
&emsp;  我们把 Subject 当作一个桥梁（或者说是代理）， Subject 既是 Observable 也是 Observer :
- 作为一个 Observer ，它可以订阅序列。
- 同时作为一个 Observable ，它可以转发或者发射数据。


<br/>
**`PublishSubject`**
`PublishSubject` 只发射给观察者订阅后的数据。
![PublishSubject 监听事件](https://upload-images.jianshu.io/upload_images/2959789-cd667f277fabf6fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```        
        let disposeBag = DisposeBag()
        let subject = PublishSubject<String>()
        
        subject.subscribe{
            print("subscription: 1 Event:", $0)
        }.disposed(by: disposeBag)
        
        subject.onNext("❤️")
        subject.onNext("🐶")
        
        subject.subscribe{
            print("subscription: 2 Event:", $0)
            }.disposed(by: disposeBag)
        subject.onNext("🐱")
        subject.onNext("🥜")
```
打印：
`subscription: 1 Event: next(❤️)`
`subscription: 1 Event: next(🐶)`
`subscription: 1 Event: next(🐱)`
`subscription: 2 Event: next(🐱)`
`subscription: 1 Event: next(🥜)`
`subscription: 2 Event: next(🥜)`



<br/>
#`ReplaySubject`
和 PushblishSubject 不同，不论观察者什么时候订阅， ReplaySubject 都会发射完整的数据给观察者。

![PushblishSubject 监听](https://upload-images.jianshu.io/upload_images/2959789-b1416dbcfddcb02b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
        let disposeBag = DisposeBag()
        let subject = ReplaySubject<String>.create(bufferSize: 1)
        
        subject.subscribe{
            print("Subscription: 1 Event:", $0)
        }.disposed(by: disposeBag)
        
        subject.onNext("🐶")
        subject.onNext("🐱")
        
        subject.subscribe{
            print("Subscription: 2 Event:", $0)
        }.disposed(by: disposeBag)
        
        
        subject.onNext("1️⃣")
        subject.onNext("❤️")
```
打印：
`Subscription: 1 Event: next(🐶)`
`Subscription: 1 Event: next(🐱)`
`Subscription: 2 Event: next(🐱)`
`Subscription: 1 Event: next(1️⃣)`
`Subscription: 2 Event: next(1️⃣)`
`Subscription: 1 Event: next(❤️)`
`Subscription: 2 Event: next(❤️)`


<br/>
#`BehaviorSubject`
&emsp;  当观察者对 BehaviorSubject 进行订阅时，它会将源 Observable 中最新的元素发送出来（如果不存在最新的元素，就发出默认元素）。然后将随后产生的元素发送出来。

![BehaviorSubject 监听](https://upload-images.jianshu.io/upload_images/2959789-ace8af127dbccedd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
        let disposeBag = DisposeBag()
        let subject = BehaviorSubject.init(value: "🎈")
        
        subject.subscribe{
            print("Subcription: 1 Event:", $0)
            }.disposed(by: disposeBag)
        
        subject.onNext("🦁")
        subject.onNext("🐲")
        
        subject.subscribe{
            print("Subcription: 2 Event:", $0)
            }.disposed(by: disposeBag)
        
        subject.onNext("💰")
        subject.onNext("💵")
        
        
        subject.subscribe{
            print("Subcription: 3 Event:", $0)
            }.disposed(by: disposeBag)
        
        subject.onNext("🍍")
        subject.onNext("🍉")
```
打印：
`Subcription: 1 Event: next(🎈)`
`Subcription: 1 Event: next(🦁)`
`Subcription: 1 Event: next(🐲)`
`Subcription: 2 Event: next(🐲)`
`Subcription: 1 Event: next(💰)`
`Subcription: 2 Event: next(💰)`
`Subcription: 1 Event: next(💵)`
`Subcription: 2 Event: next(💵)`
`Subcription: 3 Event: next(💵)`
`Subcription: 1 Event: next(🍍)`
`Subcription: 2 Event: next(🍍)`
`Subcription: 3 Event: next(🍍)`
`Subcription: 1 Event: next(🍉)`
`Subcription: 2 Event: next(🍉)`
`Subcription: 3 Event: next(🍉)`




<br/>
**`Variable `**
Variable 是 BehaviorSubject 的一个封装。相比 BehaviorSubject ，它不会因为错误终止也不会正常终止，是一个无限序列。

`（注意：由于 Variable 在之后版本中将被废弃，建议使用 Varible 的地方都改用下面介绍的 BehaviorRelay 作为替代。）`
```
func variableTest() {
        let varible = BehaviorRelay<String>(value: "OnePice：最强 Z")
        varible.asObservable().subscribe { (event) in
            print("Subscription: 1, event: \(event)")
        }.disposed(by: disposeBag)
        
        varible.accept("路飞")
        varible.accept("索大")
        
        
        
        varible.asObservable().subscribe { (event) in
            print("----> Subscription: 2, event: \(event)")
        }.disposed(by: disposeBag)
        
        varible.accept("鸣人：风遁螺旋手里⚔")
        varible.accept("二柱子：天照")
    }
```
打印：
`Subscription: 1, event: next(OnePice：最强 Z)`
`Subscription: 1, event: next(路飞)`
`Subscription: 1, event: next(索大)`
`----> Subscription: 2, event: next(索大)`
`Subscription: 1, event: next(鸣人：风遁螺旋手里⚔)`
`----> Subscription: 2, event: next(鸣人：风遁螺旋手里⚔)`
`Subscription: 1, event: next(二柱子：天照)`
`----> Subscription: 2, event: next(二柱子：天照)`

&emsp;  我们最常用的 Subject 应该就是 Variable 。Variable 很适合做数据源，比如作为一个 UITableView 的数据源，我们可以在这里保留一个完整的 Array 数据，每一个订阅者都可以获得这个 Array 。


<br/>
***
<br/>
># Operator 操作符
&emsp;  **操作符**可以帮助大家创建新的序列，或者变化组合原有的序列，从而生成一个新的序列。

&emsp;  我们之前在[输入验证](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/first_app.html)例子中就多次运用到操作符。例如，通过 [map](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/decision_tree/map.html) 方法将**输入的用户名**，转换为**用户名是否有效**。然后用这个转化后来的序列来控制红色提示语是否隐藏。我们还通过 [combineLatest](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/decision_tree/combineLatest.html) 方法，将**用户名是否有效**和**密码是否有效**合并成**两者是否同时有效**。然后用这个合成后来的序列来控制按钮是否可点击。

#`filter`
一个序列的温度列表,你可以用 [filter](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/decision_tree/filter.html) 创建一个新的序列。这个序列只发出温度大于 10 度的元素。

```
        let disposeBag = DisposeBag()
        
        Observable.of(2, 32, 22, 5, 60, 1).filter { (number) -> Bool in
            number > 10
            }.subscribe { (number) in
                print("温度： \(number)")
        }.disposed(by: disposeBag)
```
`温度： next(32)`
`温度： next(22)`
`温度： next(60)`
`温度： completed`


<br/>
#`map - 转换`
&emsp;  可以用 [map](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/decision_tree/map.html) 创建一个新的序列。这个序列将原有的 **JSON** 转换成 **Model** 。这种转换实际上就是解析 **JSON** 。
```
       let disposeBag = DisposeBag()
        Observable.of(1, 2, 3)
            .map { $0 * 10 }
            .subscribe(onNext: { print($0) })
            .disposed(by: disposeBag)
        
```
打印:
`10`
`20`
`30`

<br/>
#`zip - 组合`
&emps;  通过一个函数将多个 Observables 的元素(最多8个)组合起来，然后将每一个组合的结果发出来
```
        let disposeBag = DisposeBag()
        let first = PublishSubject<String>()
        let second = PublishSubject<String>()
        
        Observable.zip(first, second) { $0 + $1 }
            .subscribe(onNext: {
                print($0)
            }).disposed(by: disposeBag)
        
        first.onNext("1")
        second.onNext("A")
        first.onNext("2")
        second.onNext("B")
        second.onNext("C")
        second.onNext("D")
        first.onNext("3")
        first.onNext("4")
```
打印：
`1A`
`2B`
`3C`
`4D`




<br/>
***
<br/>
># 线程切换

**`Schedulers(调度器)`**
&emsp;  `Schedulers` 是 Rx 实现多线程的核心模块，它主要用于控制任务在哪个线程或队列运行。

<br/>
**`切换线程`**
```
sequence1.observeOn(backgroundScheduler) // 切换到后台线程
  .map { n in
      print("在 background scheduler 执行")
  }
  .observeOn(MainScheduler.instance) // 切换到主线程
  .map { n in
      print("在 main scheduler")
  }
```
线程的切换支持 GCD 和 NSOperation，主要使用两个操作符：observeOn 和 subscribeOn ，常用的还是 observeOn 。
- 调用 observeOn 指定接下来的操作在哪个线程；
- 调用 subscribeOn 决定订阅者的操作执行在哪个线程。

若我们没有明确调用这两个操作，后面的操作都是在当前线程执行的。

&emsp;  [subscribeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/subscribeOn.html) 来决定数据序列的构建函数在哪个 **Scheduler** 上运行。网络请求中，由于获取 `Data` 需要花很长的时间，所以用 [subscribeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/subscribeOn.html) 切换到 **后台 Scheduler** 来获取 `Data`。这样可以避免主线程被阻塞。

&emsp;  [observeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/observeOn.html) 来决定在哪个 **Scheduler** 监听这个数据序列。UI刷新数据，通过使用 [observeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/observeOn.html) 方法切换到主线程来监听并且处理结果。

&emsp;  一个比较典型的例子就是，在后台发起网络请求，然后解析数据，最后在主线程刷新页面。你就可以先用 [subscribeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/subscribeOn.html) 切到后台去发送请求并解析数据，最后用 [observeOn](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/observeOn.html) 切换到主线程更新页面。

<br/>
#`MainScheduler`

&emsp;`MainScheduler` 代表主线程。如果你需要执行一些和 UI 相关的任务，就需要切换到该 Scheduler 运行。
```
public class func ensureExecutingOnScheduler() 
```
&emsp;  可以保证代码一定执行在主线程的地方调用 `MainScheduler.ensureExecutingOnScheduler() `，特别是在线程切换来切换去的情况下，或者是调用其他的库，我们不确定当前是否在执行在主线程。毕竟 UI 的更新还是要在主线程执行的。


<br/>

#`SerialDispatchQueueScheduler`

&emsp;`SerialDispatchQueueScheduler `(串行的调度器)抽象了串行 DispatchQueue, MainScheduler 就是继承于它。如果你需要执行一些串行任务，可以切换到这个 Scheduler 运行。


<br/>

#`ConcurrentDispatchQueueScheduler`

&emsp;`ConcurrentDispatchQueueScheduler` 抽象了并行 DispatchQueue。如果你需要执行一些并发任务，可以切换到这个 Scheduler 运行。

<br/>

#`OperationQueueScheduler`

&emsp;`OperationQueueScheduler` 抽象了 NSOperationQueue。它具备 NSOperationQueue 的一些特点，例如，你可以通过设置maxConcurrentOperationCount，来控制同时执行并发任务的最大数量。



<br/>
***
<br/>
># Error Handling - 错误处理
&emsp;  一旦序列里面产出了一个 `error` 事件，整个序列将被终止。`RxSwift` 主要有两种错误处理机制：

*   retry - 重试
*   catch - 恢复

<br/>
#`retryWhen `
&emsp;  `retryWhen `操作符，这个操作符主要描述应该在何时重试，并且通过闭包里面返回的 Observable 来控制重试的时机


<br/>
#`catchError - 恢复`
&emsp;  [catchError](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree/catchError.html) 可以在错误产生时，用一个备用元素或者一组备用元素将错误替换掉




<br/>
***
<br/>
参考资料：
[线程切换](http://t.swift.gg/d/31-015)
