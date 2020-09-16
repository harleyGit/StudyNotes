># 初始化
**`① 自定义指定构造器方法`**
```
var studentName : String = ""
var age : Int  = 20


init(studentName: String, age: Int) {
     //调用 init?(coder aDecoder: NSCoder) 或者下面的方法
    super.init(nibName: nil, bundle: nil)

    self.studentName  = studentName
    self.age = age
}

//必须实现下面的这个方法
required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```

<br/>
**`② 便利构造器方法`**
```
var studentName : String = ""
var age : Int  = 20

convenience init(studentName: String, age: Int) {
    self.init()

    self.studentName = studentName
    self.age = age
}
```
