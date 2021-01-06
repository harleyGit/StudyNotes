># UI交互
**`UIButton 点击`**
```
self.registerBtn.rx.tap.subscribe { [weak self] (next) in
            self?.registerBtn.backgroundColor = UIColor.red
            print("\("注册了")")
        }.disposed(by: disposeBag)
```
打印：
`注册了`




<br/>
**`UITextField 文本响应`**

```
let disposeBag = DisposeBag()

self.userNameInput.rx.text.orEmpty.changed.subscribe(onNext: { (text) in
            
            print("监听到了：- \(text)")
        }).disposed(by: disposeBag)
       
self.userNameInput.rx.text.orEmpty.bind(to: self.registerBtn.rx.title()).disposed(by: disposeBag)
```
打印：
`监听到了：- 2`
`监听到了：- 22`
`监听到了：- 222`

效果：
![文本响应](https://upload-images.jianshu.io/upload_images/2959789-8b1bdc603af0480a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
**`UIScrollView 滚动响应`**
```
scrollView.rx.contentOffset.subscribe(onNext: { [weak self] (content) in
        self?.view.backgroundColor = UIColor.init(red: content.y/255.0*0.8, green: content.y/255.0*0.3, blue: content.y/255.0*0.6, alpha: 1);
        print(content.y)
    }).disposed(by: disposeBag)
```




<br/>
**`手势响应`**
```
let tap = UITapGestureRecognizer()
    self.label.addGestureRecognizer(tap)
    self.label.isUserInteractionEnabled = true
    tap.rx.event.subscribe { (event) in
        print("我点击了label")
    }.disposed(by: disposeBag)  
```
打印：
`我点击了label`





<br/>
***
<br/>
># KVO 响应
```
class Student: NSObject {
    
     //这里一定要用@objc，因为KVO用到了OC的runtime，否则发现不到这个属性numberID
    //这里还要用到dynamic，否则改变属性值时无法观察到变化
    @objc public dynamic var numberID: String?// = "RxSwift 的 KVO"

}



let student = Student()
student.rx.observeWeakly(String.self, "numberID").subscribe(onNext: { (change) in
            print(change ?? "numberOne")
            }).disposed(by: disposeBag)        
        
student.numberID = "OneByOne"      
```
打印：
`numberOne`//为空，打印为这个值
`OneByOne`




<br/>
***
<br/>
># 通知响应
```
NotificationCenter.default.rx.notification(UIResponder.keyboardWillShowNotification).takeUntil(self.rx.deallocated)//页面销毁自动移除通知监听
            .subscribe { (event) in
            print("键盘出现通知来了: \(event)")
        }.disposed(by: disposeBag)
```
打印：
`键盘出现通知来了: next(name = UIKeyboardWillShowNotification,`
` object = nil, `
`userInfo = Optional([AnyHashable("UIKeyboardCenterEndUserInfoKey"):NSPoint: {207, 723}, AnyHashable("UIKeyboardAnimationCurveUserInfoKey"): 7, AnyHashable("UIKeyboardCenterBeginUserInfoKey"): NSPoint: {207, 1069}, AnyHashable("UIKeyboardFrameBeginUserInfoKey"): NSRect: {{0, 896}, {414, 346}}, AnyHashable("UIKeyboardBoundsUserInfoKey"): NSRect: {{0, 0}, {414, 346}}, AnyHashable("UIKeyboardAnimationDurationUserInfoKey"): 0.25, AnyHashable("UIKeyboardFrameEndUserInfoKey"): NSRect: {{0, 550}, {414, 346}}, AnyHashable("UIKeyboardIsLocalUserInfoKey"): 1]))`





<br/>
***
<br/>
>#Timer 定时器
```
 let timer = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        timer.subscribe(onNext: { (number) in
            print("定时器：\(number)")
            }).disposed(by: disposeBag)
```
打印：
`定时器：0`
`定时器：1`
`定时器：2`
我返回上一级的ViewController，定时器就销毁了，停止了打印

&emsp; 使用RxSwift免去了定时器的一些不必要的麻烦，如：runloop影响、销毁问题、线程问题。








<br/>
***
<br/>
># 网络请求

```
let url = URL(string: "https://www.jianshu.com/p/5533c99bfa8e")
        URLSession.shared.rx.response(request: URLRequest(url: url!))
            .subscribe(onNext: { (response, data) in
                print("------->> response: \(response)")
                print("~~~~~~~>> data: \(data)")
            }, onError: { (error) in
                print("error is: \(error)")
            }, onCompleted: {
                print("结束了")
            }).disposed(by: disposeBag)
```
打印：
![请求打印](https://upload-images.jianshu.io/upload_images/2959789-016babe8679dab55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)










<br/>
***
<br/>


<br/>
#`UITextField Demo`
```

@IBOutlet weak var number1: UITextField!
@IBOutlet weak var number2: UITextField!
@IBOutlet weak var number3: UITextField!

@IBOutlet weak var result: UILabel!

let disposeBag = DisposeBag()

Observable.combineLatest(number1.rx.text.orEmpty, number2.rx.text.orEmpty, number3.rx.text.orEmpty) { text1, text2, text3 -> Int in
            let aa = (Int(text1) ?? 0) + (Int(text2) ?? 0)
            return aa + (Int(text3) ?? 0)
            }
            .map{$0.description}
            .bind(to: result.rx.text)
            .disposed(by: disposeBag)

```
效果：
![计算](https://upload-images.jianshu.io/upload_images/2959789-b646f659baba6e07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
