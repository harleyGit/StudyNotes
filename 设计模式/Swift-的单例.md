
&emsp; 使用一个静态类型属性创建简单的单例对象，它保证懒初始化一次，即使在多个线程同时访问时也是如此:
**`创建一个单例`**
```
class Singleton {
    static let sharedInstance = Singleton()
}
```

&emsp; 如果需要在初始化之外执行其他设置，可以将闭包调用的结果分配给全局常量:
```
class Singleton {
    static let sharedInstance: Singleton = {
        let instance = Singleton()
        // setup code
        return instance
    }()
}
```
