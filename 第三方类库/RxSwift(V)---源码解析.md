




># RxSwift 角色定位

-  观察者（Observer）
-  被观察者(可观察的)（Observable）
-  订阅者（Subscriber） 事件的最终处理者
-  管道（Sink） Observer 和 Observable 沟通的桥梁

```
Observable<String>.create { observer -> Disposable in
  observer.onNext("hello")
  return Disposables.create()
}
.subscribe { event in
  print(event.element)
}

```


<br/>
***
<br/>


># Observable(被观察者)继承链

- #`Observable 继承关系图`

![观察类继承链](https://upload-images.jianshu.io/upload_images/2959789-7ee3a253aeebe79a.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
![Observable 继承体系](https://upload-images.jianshu.io/upload_images/2959789-5dbda26f28ecbd32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- #`Observable的核心函数`
a. `subscribe` 订阅操作, `Observable `和 `Observer` 通过订阅建立联系,
`Observable` 好比水源， `Observer` 好比水龙头(永远开着的水龙头)， 订阅的过程就是在`Observable` 和 `Observer` 之间建立管道， 一旦建立管道即是永久性的，只要水源有谁， 水龙头就会有水;
b.  `run` 对用户不可见，隐藏了大量的实现细节， 这个函数就是建立水管的过程;
c.  `asObservable`： 这个协议的存在使得`Observable` 的定义变得更加广泛。

- `asObservable `的函数实现：
```
    public func asObservable() -> Observable<Element> {
        return source
    }

```

-` ObservableType` 协议扩展
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
***
<br/>



># Observer 继承体系

![Observer 继承体系](https://upload-images.jianshu.io/upload_images/2959789-71e7d22bb60e4220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>


![Observer 继承体系](https://upload-images.jianshu.io/upload_images/2959789-eefe7cebfe7b556b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>


># 订阅、序列流程
![订阅调用方法流程图](https://upload-images.jianshu.io/upload_images/2959789-5479356a6f4e3b8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
