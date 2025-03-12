> <h1 id=""></h1>
- [**‌Then介绍**](#Then介绍)
- [**方法**](#方法)
	- [then()](#then())
	- [do()](#do())
	- [with()](#with())
- **资料**

<br/><br/><br/><br/>

***
<br/>

> <h1 id="Then介绍">Then介绍</h1>
Then 是一个轻量级 Swift 库（一个 Swift 初始化器的语法糖），主要用于简化对象的初始化和配置，使代码更简洁易读。它提供了一种便利的语法，可以在对象创建的同时进行属性的设置，避免重复的代码。

Then 框架非常简单，代码量在 60 行左右。

<br/>

Then 框架一共提供了三个方法，用于减少代码量：

- then() 用在初始化实体时（有返回值）；
- do() 用在多次使用同一个对象，比如设置一个对象的多个属性时（无返回值）；
- with() 对一个值形的实例进行复制，修改新实例的属性，返回修改后的实例。

<br/><br/><br/>

***
<br/>

> <h1 id="then()">then()</h1>

label传统设置属性值：

```swift
let label: UILabel = {
    let label = UILabel()
    label.textAlignment = .center
    label.textColor = .black
    label.text = "Hello, World!"
    return label
}()
```

等价于：

```swift
private let tetLab00 = UILabel().then {
    $0.backgroundColor = .orange
    $0.textAlignment = .center
    $0.textColor = .black
    $0.text = "Hello, world!!"
}
```

<br/><br/>

对于自定义的类、结构体，可以通过扩展 Then 协议，来使用 then():

```swift
class MyThen {
    var really:String?
}

extension MyThen: Then {
    
}



// 使用
let myThen = MyThen().then {
        $0.really = "8888888"
}
```


<br/><br/><br/>
> <h2 id=""></h2>
  
```swift
private func testUserdefaultsDo() {
    UserDefaults.standard.do {
        $0.set("devxoul", forKey: "userName")
        $0.set("devxoul@gmail.com", forKey: "email")
        $0.synchronize()
    }
}
```
do() 可以减少代码的键入量。do() 和 then() 的区别就是，do() 没有返回值，then() 会返回 self。

若是不用`then`，我们要写很多行UserDefaults.standard，除非用一个变量对其进行引用。


<br/><br/><br/>
> <h2 id="with()">with()</h2>

with() 为创建一个新值，对新值进行修改。

```swift
private let tetLab00 = UILabel().then {
    $0.backgroundColor = .orange
    $0.textAlignment = .center
    $0.textColor = .black
    $0.text = "Hello, world!!"
}

private let testBtn00 = UIButton(type: .custom).then {
    $0.frame = CGRect(x: 120, y: 200, width: 100, height: 60)
    $0.setTitle("测试Then的with", for: .normal)
    $0.backgroundColor = .red
}

testBtn00.addTarget(self, action: #selector(testThen_with_method(_:)), for: .touchUpInside)

// 使用
 @objc private func testThen_with_method(_ sender: UIButton) {
    let newFrame = self.tetLab00.frame.with {
        $0.size.width = 400
        $0.size.height = 100
        $0.origin.x = 40
    }
    
    self.tetLab00.do {
        $0.frame = newFrame
    }
}
```


