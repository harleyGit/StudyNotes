- [**RxSwift(II)**](https://github.com/harleyGit/StudyNotes/blob/master/ClassLibrary/RxSwift(II).md)

<br/>

- **RxSwift角色定位**
	- Observable(被观察者)继承链
	- 	Observable的核心函数
	- 	` ObservableType` 协议扩展
	- 	Observer继承体系
	- 	订阅、序列流程
- **Observable：可观察的**
- **基本属性**
- **操作符**
	- **share(replay:)**
	- **empty**
	- **just**	
	- **of**
	- **from**
	- **create**
- **Scheduler 调度器**
- [**Cocoi老师RxSwift源码**](https://www.jianshu.com/u/5981a4f71db5)
- [**RxSwift应用**](https://www.jianshu.com/p/f61a5a988590)
- [**RxSwift中文文档**](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/)
- [**RxSwift特征序列**](http://www.cocoachina.com/articles/29100)
- [**RxSwift销毁者Dispose源码分析**](https://blog.csdn.net/JeffersonZHabc/article/details/98962237)
- [**RxSwift使用详解系列**](https://www.jianshu.com/p/f61a5a988590)
- [**RxSwift使用**](https://www.hangge.com/blog/cache/detail_1917.html)


<br/>

***
<br/>

># RxSwift角色定位

-  观察者（Observer）：执行要做的事
-  被观察者(可观察的)（Observable）：产生的事件，也就是序列
-  订阅者（Subscriber） 事件的最终处理者
-  管道（Sink） Observer 和 Observable 沟通的桥梁

```
Observable<String>.create { observer -> Disposable in
  observer.onNext("hello")//发送序列
  return Disposables.create()
}.subscribe { event in //订阅序列
  print(event.element)
}

```


<br/>
<br/>

- **Observable(被观察者)继承链**

`Observable 继承关系图`

![观察类继承链](https://upload-images.jianshu.io/upload_images/2959789-7ee3a253aeebe79a.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

![Observable继承体系](https://upload-images.jianshu.io/upload_images/2959789-5dbda26f28ecbd32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

- **`Observable的核心函数`**


a. `subscribe` 订阅操作, `Observable `和 `Observer` 通过订阅建立联系,
`Observable` 好比水源， `Observer` 好比水龙头(永远开着的水龙头)， 订阅的过程就是在`Observable` 和 `Observer` 之间建立管道， 一旦建立管道即是永久性的，只要水源有谁， 水龙头就会有水;

b.  `run` 对用户不可见，隐藏了大量的实现细节， 这个函数就是建立水管的过程;

c.  `asObservable`： 这个协议的存在使得`Observable` 的定义变得更加广泛，`asObservable `的函数实现：

```
public func asObservable() -> Observable<Element> {
    return source
}

```


<br/>
<br/>

- **` ObservableType` 协议扩展**

```
extension ObservableType {
	public func subscribe(_ on: @escaping (Event<E>) -> Void)
	    -> Disposable {
	        let observer = AnonymousObserver { e in
	            on(e)
	        }
	        return self.asObservable().subscribe(observer)
	}
}

```
&emsp;  `ObservableType: `扩展 `subscribe `方法， 确保所有的`Observable`行为一致， 都是经由`self.asObservable() `获取`Observable`;




<br/>
<br/>



- **Observer继承体系**

![Observer 继承体系](https://upload-images.jianshu.io/upload_images/2959789-71e7d22bb60e4220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

![Observer 继承体系](https://upload-images.jianshu.io/upload_images/2959789-eefe7cebfe7b556b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
<br/>


- **订阅、序列流程**
![订阅调用方法流程图](https://upload-images.jianshu.io/upload_images/2959789-5479356a6f4e3b8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>




># Observable：可观察的

**`函数式`**

&emsp;  `函数式编程`是种编程范式，它需要我们将函数作为参数传递，或者作为返回值返还。通过组合不同的函数来得到想要的结果。

特点：**`允许把函数本身作为参数传入另一个函数，同时还允许返回一个函数！`**

函数表达式如：`y = f(x) ---> x = f(x) ---> y = f(f(x))`

**`优点：`**
- 灵活
- 高复用
- 简洁
- 易维护
- 适应各种需求变化


<br/>

 **`响应式`**
 
概念：**`对象对某一数据流变化做出响应的这种编码方式称为响应式。`**

<br/>

> **`Observable 概念`**

&emsp; ` Observable` 直译为可观察的，它在 RxSwift 起到了举足轻重的作用，在整个 RxSwift 的使用过程中你会经常使用它。如果你使用过 RAC ，它如同 Signal 一样。RxSwift 中关键点就是在于如何把普通的数据或者事件变成可观察的，这样当某些数据或事件有变化的时候就会通知它的订阅者。


<br/>

**①. Observable<T>**

* **`Observable<T>`**   这个类就是 Rx 框架的基础，我们可以称它为可观察序列。它的作用就是可以异步地产生一系列的 Event（事件），即一个 Observable<T> 对象会随着时间推移不定期地发出 event(element : T) 这样一个东西。
*   而且这些 Event 还可以携带数据，它的泛型 <T> 就是用来指定这个 Event 携带的数据的类型。
*   有了可观察序列，我们还需要有一个 Observer（订阅者）来订阅它，这样这个订阅者才能收到 Observable<T> 不时发出的 Event。


<br/>

**②. Event**

源码

```
public enum Event<Element> {
    /// Next element is produced.
    case next(Element)
 
    /// Sequence terminated with an error.
    case error(Swift.Error)
 
    /// Sequence completed successfully.
    case completed
}
```
&emsp;  可以看到 `Event` 就是一个枚举，也就是说一个` Observable `是可以发出 3 种不同类型的 Event 事件：
* `next` 事件就是那个可以携带数据 `<T> `的事件，可以说它就是一个“最正常”的事件;

* `error` 事件表示一个错误，它可以携带具体的错误内容，一旦` Observable `发出了 `error event`，则这个 `Observable` 就等于终止了，以后它再也不会发出 event 事件了;

* `completed` 事件表示 `Observable `发出的事件正常地结束了，跟 error 一样，一旦 `Observable` 发出了 `completed event`，则这个 Observable 就等于终止了，以后它再也不会发出 event 事件了;



那如何能够让某些数据或事件成为 Observable 呢？

&emsp;  RxSwift 中提供很多种创建 Observable 创建方法。比如：`From`、`never`、`empty` 和 `create` 等。订阅者可以收到 3 个事件，`onNext`、`onError` 和 `onCompleted`，每个 Observable 都应该至少有一个 `onError `或` onCompleted` 事件，`onNext `表示它传给下一个接收者时的数据流。

```
func createObserver() {
        //1. 序列创建
        let observer = Observable<Any>.create { (observer) -> Disposable in

            //3. 信号发送
            observer.onNext("Hello! 我来了！！！")
            observer.onCompleted()
            
            return Disposables.create()
        }
        
        //2. 序列订阅
        observer.subscribe(onNext: { (text) in
            print("---> \(text)")
        }, onError: nil, onCompleted: {
            print("Completed 完成！！")
        }, onDisposed: nil).disposed(by: disposeBag)
    }
```
打印：

`---> Hello! 我来了！！！`

`Completed 完成！！`

<br/>

**方法解析**

点击上述代码中的 `Observable<Any>.create {`中的`create`方法，如下图：

![源码 create 方法](https://upload-images.jianshu.io/upload_images/2959789-0405021f247539be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

点击上图中的`create`发现无法点击，找到`create`方法的实现
![create 方法](https://upload-images.jianshu.io/upload_images/2959789-5e5dc47ade117583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

点击上图中的`AnonymousObservable`跳到如下图：

![AnonymousObservable 类的初始化](https://upload-images.jianshu.io/upload_images/2959789-582fa564f7e3dd9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

点击`②`中的`subscribe`方法，来到其源码：

```
public func subscribe(onNext: ((Element) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
        -> Disposable {
            let disposable: Disposable
            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
            

            //上述开始 ①中序列创建 的闭包中的参数 observer 就是来自于下面的这个observer
            let observer = AnonymousObserver<Element> { event in
                
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }

            //返回值中进行函数调用
            return Disposables.create(
                self.asObservable().subscribe(observer),
                disposable
            )
    }
}
```

一旦进行这句`self.asObservable().subscribe(observer),
                disposable`的调用,就会来到
                
```
let observer = Observable<Any>.create { (observer) -> Disposable in

            //3. 信号发送
            observer.onNext("Hello! 我来了！！！")
            observer.onCompleted()
            
            return Disposables.create()
        }
```
上述闭包中的调用了，就开始走`onNext`、`onCompleted()`方法的调用了。

点击`onNext`方法，来到下图：
![onNext 方法](https://upload-images.jianshu.io/upload_images/2959789-2f30ee855e93a01a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








<br/>

***
<br/>

># 基本属性

**`rx、orEmpty`**

```
let usernameOutlet = UITextField()

//usernameOutlet.rx.text: 观察者初始化, 将usernameOutlet用Reactive封装
//.text: 监听和绑定text filed的值
//orEmpty：其实就是将String?转为String处理，这样就不需要考虑String？的情况，也就不需要考虑String为nil的情况。在Rxswift中，对于所有字符串的监听都是转为orEmpty处理的
//map: 操作符将源 Observable 的每个元素应用你提供的转换方法，然后返回含有转换结果的 Observable(新的可观察序列)。
let usernameValid = usernameOutlet.rx.text.orEmpty
            .map { $0.count >= 5 }
            .share(replay: 1)
```

<br/>

**`Demo`**

```
let disposeBag = DisposeBag()
    
    private lazy var userNameInput: UITextField = {
        let text = UITextField()
        text.placeholder = "用户名输入"
        text.layer.borderColor = UIColor.black.cgColor
        text.layer.borderWidth = 0.5
        text.layer.cornerRadius = 2
        text.layer.masksToBounds = true
        
        return text
    }()
    
    private lazy var reminderUserNameInput: UILabel = {
        let label = UILabel()
        label.font = UIFont.systemFont(ofSize: 14.0)
        label.textColor = UIColor.red
        label.text = "用户名至少3-10个字符组成"
        
        return label
    }()
    
    private lazy var registerBtn: UIButton = {
        let button = UIButton()
        button.backgroundColor = UIColor.cyan
        button.setTitle("注册", for: .normal)
        
        return button
    }()




func inputManager() {
        let inputValid = self.userNameInput.rx.text.orEmpty.map { (text) -> Bool in
            let length = text.count
            return length >= 3 && length <= 10
        }.share(replay: 1)
        
        inputValid.bind(to: self.reminderUserNameInput.rx.isHidden).disposed(by: disposeBag)
        
        inputValid.bind(to: self.registerBtn.rx.isEnabled).disposed(by: disposeBag)
        
        self.registerBtn.rx.tap.subscribe { (next) in
            print("\("注册了")")
        }.disposed(by: disposeBag)
    }
```
效果图：

![](https://upload-images.jianshu.io/upload_images/2959789-5d01ac4c33adbe93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击注册：
`注册了`

**上段Code解读**
- 安装 RxSwift 时会安装 RxSwift(对ReactiveX的实现) 和 RxCocoa(对iOS cocoa 层的实现)；

- orEmpty：主要使 String? 类型变为 String类型；

- map：它属于 Rx 变换操作中的一种，主要对 Observable 发射的数据应用一个函数，执行某种操作，返回经过函数处理过的 Observable。Observable 可观察的对象，用来被观察者(observer)订阅，这样observe可以监听Observable发出的事件；

- share(replay: 1)：只允许监听一次；






<br/>

***
<br/>


>#  操作符

&emsp;  `Observable` 创建后，可能为了满足某些需求需要修改它，这时就需要用到操作符。RxSwift 提供了非常多的操作符，不必要一一掌握这些操作符，使用的时候查一下即可，当然常见的操作符必须要掌握，比如 map、flatMap 、create 、filter 等。

- **`share(replay:)`**
	- 该操作符将使得观察者共享源 Observable，并且缓存最新的 n 个元素，将这些元素直接发送给新的观察者。
	- 简单来说 shareReplay 就是 replay 和 refCount 的组合。


```
      let seq = PublishSubject<Int>()
        let a = seq.map { (i) -> Int in
            print("MAP---\(i)")
            return i * 2
        }
        //.share(replay: 1, scope: .forever)
        
        let _ = a.subscribe(onNext: { (num) in
            print("--1--\(num)")
        }, onError: nil, onCompleted: nil, onDisposed: nil)
        
        seq.onNext(1)
        seq.onNext(2)
        
        let _ = a.subscribe(onNext: { (num) in
            print("--2--\(num)")
        }, onError: nil, onCompleted: nil, onDisposed: nil)
        
        seq.onNext(3)
        seq.onNext(4)
        
        let _ = a.subscribe(onNext: { (num) in
            print("--3--\(num)")
        }, onError: nil, onCompleted: nil, onDisposed: nil)
        
        seq.onCompleted()
        
```

不加 `.share(replay: 1, scope: .forever)`

`MAP---1`

`--1--2`

`MAP---2`

`--1--4`

`MAP---3`

`--1--6`

`MAP---3`

`--2--6`

`MAP---4`

`--1--8`

`MAP---4`

`--2--8`


<br/>


加了`.share(replay: 1, scope: .forever)`

`MAP---1`

`--1--2`

`MAP---2`

`--1--4`

`--2--4`

`MAP---3`

`--1--6`

`--2--6`

`MAP---4`

`--1--8`

`--2--8`

`--3--8`



<br/>

- **`empty`**

`empty`就是创建一个空的sequence,只能发出一个completed事件

```
let disposeBag = DisposeBag()
Observable<Int>.empty().subscribe { event in
    print("--->>\(event)")
}.disposed(by: disposeBag)
```
打印：

`--->>completed`
`

<br/>

- **`just`**

`just` 是创建一个sequence只能发出一种特定的事件，能正常结束

```
let disposeBag = DisposeBag()
	Observable.just("🎈").subscribe { event in
	print("--->>\(event)")
}.disposed(by: disposeBag)
```
打印：

`--->>next(🎈)`

`--->>completed`


<br/>

- **`of`**
`of`是创建一个sequence能发出很多种事件信号

```
let disposeBag = DisposeBag()
Observable.of("🎈", "🐈", "🐩", "🐔", "🐭").subscribe { event in
    print("--->>\(event)")
}.disposed(by: disposeBag)
```
打印：

`--->>next(🎈)`

`--->>next(🐈)`

`--->>next(🐩)`

`--->>next(🐔)`

`--->>next(🐭)`

`--->>completed`

改进加入`onNext`，如下：

```
let disposeBag = DisposeBag()
Observable.of("🎈", "🐈", "🐩", "🐔", "🐭").subscribe(onNext: { (element) in
    print("--->>: \(element)")

}).disposed(by: disposeBag)
```
打印：

`--->>: 🎈`

`--->>: 🐈`

`--->>: 🐩`

`--->>: 🐔`

`--->>: 🐭`

正好对应了我们subscribe中，subscribe只监听事件。

[US](https://www.jianshu.com/p/a1e2665f9a6c)


<br/>

- **`from`**

`from`就是从集合中创建sequence，例如数组，字典或者Set

```
//`from`就是从集合中创建sequence，例如数组，字典或者Set
let disposeBag = DisposeBag()
Observable.from(["🐶", "🐱", "🐭", "🐹"]).subscribe(onNext:{
    print("~~~~>: \($0)")
}).disposed(by: disposeBag)
              
```

打印：

`~~~~>: 🐶`

`~~~~>: 🐱`

`~~~~>: 🐭`

`~~~~>: 🐹`



<br/>

- **`create`**

可以自定义可观察的`sequence`，那就是使用create

```

let createO = { (element: String) -> Observable<String> in
    return Observable.create({ (observer) -> Disposable in
        observer.on(.next(element))
        observer.on(.completed)
        return Disposables.create()
    })
}
createO("🔴").subscribe{
    print("---->> \($0)")
}.disposed(by: disposeBag)


```
打印：

`---->> next(🔴)`

`---->> completed`




<br/>

***
<br/>

># Scheduler 调度器

&emsp; 若你想给 Observable 操作符链添加多线程功能，你可以指定操作符（或者特定的Observable）在特定的调度器(Scheduler)上执行。对于 ReactiveX 中可观察对象操作符来说，它有时会携带一个调度器作为参数，这样可以指定可观察对象在哪一个线程中执行。而默认的情况下，某些可观察对象是在订阅者订阅时的那个线程中执行。SubscribeOn 可以改变可观察对象该在那个调度器中执行。ObserveOn 用来改变给订阅者发送通知时所在的调度器。这样就可以使可观察对象想在那个调度器中执行就在那个调度器中执行，不受约束，而这些细节是不被调用者所关心的。犹如 GCD 一样，你只管使用，底层线程是咋么创建的，你不必关心。




<br/>

***
<br/>


