- <h2 id=''></h2>
- [ swift 语法糖 ？ ！的本质]( #swift语法糖的本质)
- [@propertyWrapper](#@propertyWrapper)
- [Task使用](#task使用)



<br/>

***
<br/><br/>

> <h1 id='swift语法糖的本质'>swift 语法糖 ？ ！的本质</h1>

![<br/>](./../../Pictures/ios_pd3.png)

这是底层可选项的代码，可以看出本质是enum

`var age: Int? = 10 `

等价于以下四种：

```
var age0: Optional<Int> = Optional<Int>.some(10)
var age1: Optional = .some(10)
var age2 = Optional.some(10)
var age3 = Optional(10)

```

```
？为optional的语法糖
optional 是一个包含了nil 和普通类型的枚举，确保使用者在变量为nil的情况下处理

！为optional 强制解包的语法糖

```


<br/><br/>

> <h2 id='@propertyWrapper'>@propertyWrapper</h2>
是 Swift 语言中的一个属性包装器（Property Wrapper）特性，引入于 Swift 5.1 版本。属性包装器允许你定义包装器类型，将通用的代码用于属性的存取和设置，使代码更具可读性、可维护性，并提供了一种在属性上应用自定义行为的方式。

```
@propertyWrapper
struct TwelveOrLess {
    private var value: Int

    init() {
        self.value = 0
    }

    var wrappedValue: Int {
        get { return value }
        set { value = min(newValue, 12) }
    }
}
```

在这个例子中，TwelveOrLess 是一个属性包装器，它确保属性值不超过 12。然后，你可以将这个包装器应用于其他属性：

```
struct MyStruct {
    @TwelveOrLess var number: Int
}

var myStruct = MyStruct()

myStruct.number = 15
print(myStruct.number)  // 输出: 12
```

在这个示例中，number 属性被 @TwelveOrLess 包装器包装，当试图将其设置为大于 12 的值时，包装器会将其截断为 12。

<br/>

高级用法：

自定义初始化和其他方法：

你可以在属性包装器中定义自定义初始化方法以及其他方法，以满足特定的需求。

```
@propertyWrapper
struct CustomWrapper {
    var wrappedValue: Int

    init(initialValue: Int) {
        self.wrappedValue = initialValue
    }

    func doSomething() {
        // 在这里可以执行其他操作
        print("Doing something with \(wrappedValue)")
    }
}

struct MyStruct {
    @CustomWrapper(initialValue: 42) var number: Int
}

var myStruct = MyStruct()
myStruct.$number.doSomething()  // 输出: Doing something with 42
包装器设置和投影：
```
你可以通过在包装器中实现 projectedValue 属性，为属性包装器添加额外的设置或投影。

```
@propertyWrapper
struct WrapperWithProjection {
    var wrappedValue: Int
    var projectedValue: SomeOtherType

    init(initialValue: Int) {
        self.wrappedValue = initialValue
        self.projectedValue = SomeOtherType()
    }
}

struct MyStruct {
    @WrapperWithProjection(initialValue: 42) var number: Int
}

var myStruct = MyStruct()
print(myStruct.$number)  // 输出: SomeOtherType()
```
属性包装器为属性提供了一种声明性的、可复用的方式来添加额外的逻辑和行为。这样的结构使得代码更清晰、更易维护。


<br/><br/>

> <h2 id='task使用'>Task使用</h2>


Task 是 Swift Concurrency 中的一部分，它用于处理异步编程，包括协程（coroutines）和异步/等待（async/await）模式。Swift Concurrency 是在 Swift 5.5 版本中引入的，并通过 async 和 await 关键字提供了一种更直观和可读的异步编程模型。

基本用法：

异步函数声明和调用：

```
func fetchData() async -> String {
    return "Data fetched successfully"
}

async {
    let result = await fetchData()
    print(result)
}
```
在上述例子中，fetchData 函数被声明为异步函数，返回一个 String。在调用这个函数时，使用 await 关键字等待异步操作的完成，并获取异步操作的结果。

异步闭包：

```
func fetchData(completion: @escaping (String) -> Void) {
    DispatchQueue.global().async {
        let result = "Data fetched successfully"
        completion(result)
    }
}

func useFetchData() {
    fetchData { data in
        print(data)
    }
}
```
在上述例子中，fetchData 函数使用 DispatchQueue.global().async 异步执行操作，通过闭包回调返回数据。这是传统的异步编程方式，不涉及 Task。

<br/>

使用 Task 进行异步编程：

创建和运行 Task：

```
func fetchData() async -> String {
    return "Data fetched successfully"
}

let myTask = Task {
    let result = await fetchData()
    print(result)
}

// 或者更简洁的写法
Task {
    let result = await fetchData()
    print(result)
}
```
Task 可以用于创建异步任务。在上述例子中，通过 Task 创建一个异步任务，其中使用 await 关键字等待异步函数 fetchData 的完成。

<br/>

Task 组：

```
func fetchData(id: Int) async -> String {
    return "Data for id \(id) fetched successfully"
}

func fetchAllData() async {
    await Task.withGroup(resultType: String.self) { group in
        for id in 1...3 {
            group.addTask {
                return await fetchData(id: id)
            }
        }

        for try await data in group {
            print(data)
        }
    }
}
```
在这个例子中，Task.withGroup 创建了一个任务组，允许并发执行多个任务。group.addTask 添加了多个异步任务到任务组中，然后使用 for try await 循环等待所有任务的完成。




