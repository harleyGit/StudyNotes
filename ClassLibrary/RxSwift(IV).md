># 网络请求合并
[RxSwift 网络请求封装](http://www.manongjc.com/article/3755.html)
网络安全请求合并：
```
 public func testRxZip() {
        
        let oneOb = Observable<Int>.create { observer -> Disposable in
            observer.on(.next(20))
            observer.on(.completed)
            return Disposables.create()
        }
        let twoOb = Observable<Int>.create { observer -> Disposable in
            observer.on(.next(80))
            observer.on(.completed)
            return Disposables.create()
        }
        
        Observable.zip(oneOb, twoOb).subscribe(onNext: {(one, two) in
            print("获取信息成功: \(one)")
            print("获取订单成功: \(two) 条")

            }).disposed(by: DisposeBag())
    }

///调用
self.testRxZip()
```
打印：
```
获取信息成功: 20
获取订单成功: 80 条
```


<br/>
***
<br/>
#**`参考资料：`**
[观察序列创建](https://www.jianshu.com/p/2a45b502279a)
