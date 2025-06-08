> <h1 id=""></h1>
- [**await和async**](#await和async)
	- [初步使用async和await](#初步使用async和await)
	- [async和await中使用演员进入主队列](#async和await中使用演员进入主队列)
	- [任务优先级](#任务优先级)
- [**多个任务处理**](#多个任务处理)
	- [多个图片下载简单版](#多个图片下载简单版)
	- [简单任务组](#简单任务组)
- [**逃逸闭包转换为支持async/await的现代异步方法**](#逃逸闭包转换为支持async/await的现代异步方法)
- [**线程安全类Actor**](#线程安全类Actor)
	- [结构体值类型新理解](#结构体值类型新理解)



<br/><br/><br/>

***
<br/>

> <h1 id="await和async">await和async</h1>

***
<br/><br/><br/>
> <h2 id="初步使用async和await">初步使用async和await</h2>

- await基本上是任务的一个暂停点;
- 添加await后我们相当于高速编译器,我们得在此状态下暂停任务,等待响应后再继续;
- 通过Task进入异步环境;

***

```swift
private static func fetchImage() async {
    print("☘️>>>>>>>>> 异步请求前")
    
    // 这里我们调用await它会瞬间发生,但是我们不知道它需要多久,await并不会改变耗时是一秒还是一小时,只是我们在等待响应,等待响应的到来
    let image = try? await self.downloadWithAsync()
    
    print("☘️========= 异步请求结束")
    
    // 一旦我们处于异步环境,我们需要回到主线程一个是: 使用dispatch回到主队列,第二个是: 我们可以使用演员
    // 主演员或多或少就是在主线程
    // 使用异步等待关键字实际上是调用主演员的运行方法,需要加入await.然后将要执行执行主队列的代码放到run方法里
    await MainActor.run {
        // 将image赋值给图片view
        // self.image = image
        print("🍎 网络请求数据来了: \(image ?? "-.-")")
    }

    print("☘️》〉》〉》〉》〉》 异步请求继续执行下面代码....")
}

public static func downloadWithAsync() async throws -> String? {
    
    do {
        if #available(iOS 15.0, *) {
            print("🍎 ++++++++ 开始发起模拟请求")
           
            try await Task.sleep(nanoseconds: 2 * 1_000_000_000) // 延迟2秒
            
            //let (data, response) = try await URLSession.shared.data(from: URL(string: "https://picsum.photos/200")!, delegate: nil)
            //return handleResponse(data: data, response: response)
            
            print("🍎 ¥¥¥¥¥¥¥ 模拟请求完成")
            return "网络请求json数据来了"
        } else {
            Thread.sleep(forTimeInterval: 2.0)
            return "----------->> json📊数据来了"
        }
    }catch {
        throw error
    }
}

private static func handleResponse(data: Data?, response: URLResponse?) -> UIImage? {
    guard
        let data = data,
        let image = UIImage(data: data),
        let response = response as? HTTPURLResponse,
        response.statusCode >= 200 && response.statusCode < 300 else {
        return nil
    }
    return image
}
```

<br/>

**调用:**

```swift
// async 若是们需要支持并发的函数,需要添加async
public static func practiceException02() {
    // await标记代码的暂停点,表示执行这个操作比较耗时
    
    // 注意: 只能在异步函数里调用异步函数,那到底怎么进入支持并发的环境呢? 答案是: 创建或者启动一个任务Task(我们通过一个任务来进入异步上下文),然后在里面才能调用异步函数
    Task {
        await self.fetchImage()
    }
}
```

***
**打印:**

```sh
☘️>>>>>>>>> 异步请求前
🍎 ++++++++ 开始发起模拟请求
🍎 ¥¥¥¥¥¥¥ 模拟请求完成
☘️========= 异步请求结束
🍎 网络请求数据来了: 网络请求json数据来了
☘️》〉》〉》〉》〉》 异步请求继续执行下面代码....
```

***
<br/><br/><br/>
> <h2 id="async和await中使用演员进入主队列">async和await中使用演员进入主队列</h2>

```swift
public static func practiceException03()   {
    print("😁<<<<<<<<<<<<<<<<<")
    
    Task {
        await self.practiceException03_00()
        
    }
    print("😂+++++++++++++++++")
}

@MainActor //必须在主线程上运行，不允许在其他线程执行
public static func practiceException03_00() async {
    print("☘️>>>>>>>>> 异步请求前")
    
    let author1 = "Author1 : \(Thread.current)"
    print("🍎 当前作者Author1: \(author1)")
    
    // Task.sleep 是一个异步函数,所以要加入 await
    // 我们没有进入后台线程,但是却进入了后台线程了.原因是 Task.sleep 进入了后台线程并等待了,注意: 这里它有时会进入后台线程,有时不会
    // 然后它下面的代码则表示在后台线程,所以后面若是有关于UI方面的操作会报警告
    // 所以我们需要在异步等待后回到 主线程中去,我们只需要跳回到主执行器即可,跳到主执行器可以: await Main.actor.run 执行就好了
    try? await Task.sleep(nanoseconds: 2_000_000_000) //(换成: try? doSomething() 就不会发生进入到后台线程了,会一直在主线程)
    
    let author2 = "Author2 : \(Thread.current)"
    print("🍎 当前作者Author2: \(author2)")
    await MainActor.run(body: {
        let author3 = "Author3 : \(Thread.current)"
        print("🍎 当前作者Author3: \(author3)")
    })
    
    print("🌵 程序代码继续执行..........")
}

static func doSomething() async throws {
    print("+++++ HI")
}
```

***

**调用:**

```swift
HGPraceMulThread.practiceException03()
```

打印:

```sh
😁<<<<<<<<<<<<<<<<<
😂+++++++++++++++++
☘️>>>>>>>>> 异步请求前
🍎 当前作者Author1: Author1 : <_NSMainThread: 0x300e2c040>{number = 1, name = main}
🍎 当前作者Author2: Author2 : <_NSMainThread: 0x300e2c040>{number = 1, name = main}
🍎 当前作者Author3: Author3 : <_NSMainThread: 0x300e2c040>{number = 1, name = main}
🌵 程序代码继续执行..........
```

- **⚠️注意:**
	- Task是用来进入异步环境上下文的;
	- 我们在`practiceException03_00()‌`方法内调用了`‌try? await Task.sleep(nanoseconds: 2_000_000_000)`这个方法,这个需要注意,因为在UI环境中也就是它之前若是主线程后,经过它调用进入后台等待,可能是主线程等待,也可能是子线程等待,所以后面用主演员,表示在主线程操作UI.

***
<br/><br/><br/>
> <h2 id="任务优先级">任务优先级</h2>


- **1.任务优先级简单测试**

```swift
 public static func practiceException04() {
        
    print("🍟 <<<<<<<<<<<")
    Task{
        print("++最近的线程: \(Thread.current)")
        print("++线程优先级: \(Task.currentPriority)")
    }
    
    Task{
        print("\n\n--最近的线程: \(Thread.current)")
        print("--线程优先级: \(Task.currentPriority)")
    }
    print("🚫 ##############")
}


/// 调用
HGPraceMulThread.practiceException04()
```

***
**打印:**

```sh
🍟 <<<<<<<<<<<
🚫 ##############
++最近的线程: <NSThread: 0x302e95a80>{number = 8, name = (null)}


--最近的线程: <NSThread: 0x302e98000>{number = 3, name = (null)}
++线程优先级: TaskPriority.high
--线程优先级: TaskPriority.high
```

- **说明:**
	- 这里调用其实在cell点击后调用的,所以是一般在主线程,但是现在没有打印在当前主线程有点奇怪的;
	- 这2个任务都是在主队列异步的,所以看到打印显示都是在执行完后才执行任务内的函数的;
	- 可以看到2个任务互不干扰,没有先后的执行顺序的;
	- 在加载图片若是没有先后顺序可以这么做的;


***
<br/>

**多个优先级处理测试:**

```swift
public static func practiceException04_01() {
        
    print("💯 <<<<<<<<<<<<<<<<<<<")
    Thread.current.name = "我命名的🍎"
    Task(priority: .high) { // 优先级高的任务优先处理,但并不意味着它们首先完成
        //                try? await Task.sleep(nanoseconds: 2_000_000_000)
        await Task.yield() // 进行让步,让其他比较紧急的任务排在它前面
        print("high : \(Thread.current) : \(Task.currentPriority)")
    }
    Task(priority: .userInitiated) {
        print("userInitiated : \(Thread.current) : \(Task.currentPriority)")
    }
    Task(priority: .medium) {// 中等优先级
        print("medium : \(Thread.current) : \(Task.currentPriority)")
    }
    Task(priority: .low) {
        print("low : \(Thread.current) : \(Task.currentPriority)")
    }
    Task(priority: .utility) {
        print("utility : \(Thread.current) : \(Task.currentPriority)")
    }
    Task(priority: .background) {
        print("background : \(Thread.current) : \(Task.currentPriority)")
    }
    
    
    Task(priority: .low) {
        print("父任务: low : \(Thread.current) : \(Task.currentPriority)")
        
        
//            Task.detached { // detached: 使子任务脱离父任务,分离任务
//                print("子任务detached : \(Thread.current) : \(Task.currentPriority)")
//            }
        Task { // detached: 使子任务脱离父任务,分离任务
            print("子任务detached : \(Thread.current) : \(Task.currentPriority)")
        }
    }
    
    let taskObj: Task<(), Never> = Task{
        // 在这里可以起一个获取图片的任务
        
    }
    // 当不需要了,比如我进入一个导航页面,然后立马退出,剩下的20个图片任务不需要加载了,就可以调用 self.taskObj.cancel()
    
    print("🔚>>>>>>>>>>>>>>>>>>>>>>")
}


/// 调用
HGPraceMulThread.practiceException04_01()

```

打印:

```sh
💯 <<<<<<<<<<<<<<<<<<<
🔚>>>>>>>>>>>>>>>>>>>>>>
userInitiated : <NSThread: 0x301872140>{number = 4, name = (null)} : TaskPriority.high
high : <NSThread: 0x301879780>{number = 6, name = (null)} : TaskPriority.high
medium : <NSThread: 0x301864480>{number = 5, name = (null)} : TaskPriority.medium
父任务: low : <NSThread: 0x30186e700>{number = 9, name = (null)} : TaskPriority.low
子任务: <NSThread: 0x30186e700>{number = 9, name = (null)} : TaskPriority.low
background : <NSThread: 0x30186c680>{number = 10, name = (null)} : TaskPriority.background
low : <NSThread: 0x30186e6c0>{number = 7, name = (null)} : TaskPriority.low
utility : <NSThread: 0x301875640>{number = 8, name = (null)} : TaskPriority.low
```

- **说明:**
	- 优先级只是决定谁最有可能最先调用,但是并不决定谁最先完成任务;
	- 比如上述的最高优先级,加入了睡眠,那么它完成的时候可能就是最后的;
	- 子任务可以从父任务那里继承优先级;
	- 可以使用一个变量引用任务,对任务取消;
		- 对于任务比如需要很长时间才能执行完的,你需要检查一下它,有相关的方法进行检测,然后判断再取消是最好的.


<br/><br/><br/>

***
<br/>

> <h1 id="多个任务处理">多个任务处理</h1>

***
<br/><br/><br/>
> <h2 id="多个图片下载简单版">多个图片下载简单版</h2>

```swift

func practiceFetchImages() {
	
	Task {
        do {
            async let fetchImage1 = fetchImage()
            async let fetchTitle1 = fetchTitle()
            
            let (image, title) = await (try fetchImage1, fetchTitle1)
            
            
//                        async let fetchImage2 = fetchImage()
//                        async let fetchImage3 = fetchImage()
//                        async let fetchImage4 = fetchImage()
//
//                        let (image1, image2, image3, image4) = await (try fetchImage1, try fetchImage2, try fetchImage3, try fetchImage4)
//                        self.images.append(contentsOf: [image1, image2, image3, image4])
            
//                        let image1 = try await fetchImage()
//                        self.images.append(image1)
//
//                        let image2 = try await fetchImage()
//                        self.images.append(image2)
//
//                        let image3 = try await fetchImage()
//                        self.images.append(image3)
//
//                        let image4 = try await fetchImage()
//                        self.images.append(image4)

        } catch {
            
        }
    }

}


func fetchTitle() async -> String {
    return "NEW TITLE"
}

func fetchImage() async throws -> UIImage {
    do {
        let (data, _) = try await URLSession.shared.data(from: url, delegate: nil)
        if let image = UIImage(data: data) {
            return image
        } else {
            throw URLError(.badURL)
        }
    } catch {
        throw error
    }
}

```


如上所述,可以将任务中的`async`和`await`放入到类似元组中进行处理,但是少的下载任务还行,但是当多的时候,就不行了.这就需要下面的**任务组**了.


***
<br/><br/><br/>
> <h2 id="简单任务组"> 简单任务组</h2>


```swift

func test00() {

	Task {
	
		await self.getImages()
	}
}

func getImages() async {
    if let images = try? await fetchImagesWithAsyncLet() {
        self.images.append(contentsOf: images)
    }
}

 func fetchImagesWithAsyncLet() async throws -> [UIImage] {
    async let fetchImage1 = fetchImage(urlString: "https://picsum.photos/300")
    async let fetchImage2 = fetchImage(urlString: "https://picsum.photos/300")
    async let fetchImage3 = fetchImage(urlString: "https://picsum.photos/300")
    async let fetchImage4 = fetchImage(urlString: "https://picsum.photos/300")
    
    let (image1, image2, image3, image4) = await (try fetchImage1, try fetchImage2, try fetchImage3, try fetchImage4)
    return [image1, image2, image3, image4]
}

```

![swift0.0.0.1.png](./../../Pictures/swift0.0.0.1.png)

***

上述若是简单的几张图片还好,若是多张图片就不行了,下面介绍任务组:


```swift
 private func fetchImage(urlString: String) async throws -> UIImage {
    guard let url = URL(string: urlString) else {
        throw URLError(.badURL)
    }
    
    do {
        let (data, _) = try await URLSession.shared.data(from: url, delegate: nil)
        if let image = UIImage(data: data) {
            return image
        } else {
            throw URLError(.badURL)
        }
    } catch {
        throw error
    }
}
    
func fetchImagesWithTaskGroup() async throws -> [UIImage] {
    let urlStrings = [
        "https://picsum.photos/300",
        "https://picsum.photos/300",
        "https://picsum.photos/300",
        "https://picsum.photos/300",
        "https://picsum.photos/300",
    ]
    
    // 使用带抛出错误的任务组
    return try await withThrowingTaskGroup(of: UIImage?.self) { group in
        var images: [UIImage] = []
        // 告诉编译器预留多少空间,一点性能提升
        images.reserveCapacity(urlStrings.count)
        
        for urlString in urlStrings {
		        // 添加子任务
		        group.addTask {
                try? await self.fetchImage(urlString: urlString)
            }
        }
        
        // 这个for循环会等待每次的for循环
        for try await image in group {
            if let image = image {
                images.append(image)
            }
        }
        
        return images
    }
}


func getImages() async {
    if let images = try? await manager.fetchImagesWithTaskGroup() {
        self.images.append(contentsOf: images)
    }
}


/// 调用
func test01() {
	Task {
			await viewModel.getImages()
	}
}
```

![swift0.0.0.0.png](./../../Pictures/swift0.0.0.0.png)


<br/><br/><br/>

***
<br/>
> <h1 id="逃逸闭包转换为支持async/await的现代异步方法">逃逸闭包转换为支持async/await的现代异步方法</h1>

这是将传统基于“逃逸闭包”的异步 API 转换为支持 `async/await` 语法的现代异步方法。以下是详细解释：

---

**🔹 `withCheckedThrowingContinuation` 是什么？**

这是 Swift 提供的一个 API，用来将使用 **回调（逃逸闭包）风格的异步函数**，转换为支持 `async/await` 的 **结构化并发函数**。

**定义如下：**

```swift
func withCheckedThrowingContinuation<T>(
    _ body: (CheckedContinuation<T, Error>) throws -> Void
) async throws -> T
```

* `T` 是要返回的结果类型
* `CheckedContinuation<T, Error>` 是你从 `body` 闭包中获得的一个 **控制对象**，你可以通过它来 **resume（恢复）这个异步流程**

---

**🔸 传统异步函数是这样写的：**

```swift
func fetchData(completion: @escaping (Result<Data, Error>) -> Void)
```

你想把它封装为 `async` 形式：

```swift
func fetchData() async throws -> Data
```

这时候就可以用 `withCheckedThrowingContinuation` 来做桥接，避免逃逸闭包的复杂性。

---

**🔹 什么是 `resume`？**

`resume(returning:)` 和 `resume(throwing:)` 是 `CheckedContinuation` 的两个方法，用来手动恢复当前挂起的 `async` 调用：

* `resume(returning:)`：当你得到了期望的结果
* `resume(throwing:)`：当你遇到了错误并希望抛出

**⚠️ 注意：`resume` 只能调用一次！否则会崩溃。**

---

**🔸 示例：封装传统 API**

假设你有这个传统异步方法：

```swift
func oldStyleNetworkCall(completion: @escaping (Data?, Error?) -> Void)
```

你可以封装为：

```swift
func fetchData() async throws -> Data {
    return try await withCheckedThrowingContinuation { continuation in
        oldStyleNetworkCall { data, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let data = data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: NSError(domain: "UnknownError", code: -1))
            }
        }
    }
}
```

---

## ✅ 为什么这样做是“避免逃逸闭包”？

你用 `withCheckedThrowingContinuation` 包裹了逃逸闭包式的 API，但在你的 `async` 调用者看来：

```swift
let data = try await fetchData()
```

这只是一个普通的结构化异步函数，而不是基于逃逸闭包的写法，提升了代码的安全性、可读性和顺序逻辑。

---

**🔒 `withChecked...` 有什么“检查”功能？**

* 它会在调试环境中检测 `resume` 是否被调用
* 如果没有 resume 或多次 resume，会在运行时报错

这有助于发现错误用法。

---

**总结**

| 术语                                | 说明                                    |
| --------------------------------- | ------------------------------------- |
| `withCheckedThrowingContinuation` | 桥接传统逃逸闭包异步 API 与现代 `async/await` 异步函数 |
| `CheckedContinuation.resume(...)` | 手动恢复暂停的异步流程                           |
| 用处                                | 提升代码可读性、结构化，简化异步控制流程                  |


***
<br/>

**如下一个简单网络请求:**

```swift
struct NetworkClient {
    
    // 增加解析支持（泛型 + JSON 解码）
    static func requestDecodable<T: Decodable>(from url: URL) async throws -> T {
        let data = try await requestData(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
    
    static func requestData(from url: URL) async throws -> Data {
        return try await withCheckedThrowingContinuation({ continuation in
            let task = URLSession.shared.dataTask(with: url, completionHandler: { data,response,err in
                if let error = err {
                    continuation.resume(throwing: error)
                } else if let data = data {
                    continuation.resume(returning: data)
                } else {
                    continuation.resume(throwing: URLError(.unknown))
                }
            })
            task.resume() // 必须调用以启动任务
        })
    }
}
struct Todo: Decodable {
    let id: Int
    let title: String
    let completed: Bool
}


/// 闭包转换成async/await
public static func practiceMulThread00() async {
    print("🍎 <<<<<<<<< 闭包转换成async/await")
    do {
        let url = URL(string: "https://jsonplaceholder.typicode.com/todos/1")!
        let todo: Todo = try await NetworkClient.requestDecodable(from: url)
        print("🍎 数据解析Todo title: \(todo.title), completed: \(todo.completed)")
    } catch {
        print("❌ 失败 Failed to fetch todo: \(error)")
    }
    print("🍎 ========== 闭包转换成async/await")
}
```

<br/>

**调用:**

```swift
Task{
    do {
        await HGPraceMulThread.practiceMulThread00()
    } catch {
        print(error)
    }
    
}
```

打印:

```sh
🍎 <<<<<<<<< 闭包转换成async/await
🍎 数据解析Todo: delectus aut autem, completed: false
🍎 ========== 闭包转换成async/await
```


<br/><br/><br/>

***
<br/>

> <h1 id="线程安全类Actor">线程安全类Actor</h1>

Actor 是 Swift 5.5（随着 Swift 并发引入）提供的一个 结构化并发模型中的线程安全引用类型。它用于解决并发环境下的数据竞争问题。

**它为什么是安全的?**
> 对 actor 内部状态的访问是 **串行的（serial）**，即任何时候只有一个执行体可以访问它的属性或方法。

**✅ 特性：**

| 特性      | 说明                        |
| ------- | ------------------------- |
| ✅ 串行访问  | 同一时间只能一个任务访问其内部数据         |
| ✅ 自动同步  | 无需使用 `DispatchQueue` 或锁   |
| ✅ 编译器保护 | 不能直接跨线程访问内部属性，除非用 `await` |

***

可以把 actor 看作一种特殊的类（class），它的**内部状态只能被一个线程在某一时刻访问，从而自动实现线程安全（thread-safe）**。


```swift
actor Counter {
    public var value = 0
    
    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}
```

***

- **📌 调用方式1️⃣**

```swift
public func testActor00(){
	let counter = Counter()
	
	Task {
	    await counter.increment()
	    let current = await counter.getValue()
	    print("Counter is \(current)")
	}
}
```

<br/>
**📌 调用方式2️⃣**

```
// 或者
public func testActor01() async {

	let counter = Counter()
	// 编译报错,因为actor要求是线程安全的只能在类的内部修改,外部修改会编译报错.所以actor是线程安全的
	// actor确保了参与者内部的所有操作都在线程安全的环境中进行的
	// counter.value = 22 
	await counter.increment()
	let current = await counter.getValue()
	print("Counter is \(current)")
}
```

- **注意到上述区别了吗?**
	- `‌testActor00()`和`‌testActor01()`方法后一个带了`async`一个后面没有携带;
		- 原因: `actor`必须是在异步环境中,进入异步环境(也就是上下文)要么是函数末尾带**async关键字**,要么使用**Task**进入,否则编译报错	
		- 因为actor具有线程安全性,所以需要我们等待进入,所以**访问属性需要等待**;
	- **actor和类差不多**,只是多了线程安全.一个线程访问一个actor时,另一个就需要等待,保证了线程安全
***

**⏱️ actor 适合用于什么时候？**

| 场景           | 示例                                          |
| ------------ | ------------------------------------------- |
| ✅ 管理共享状态     | 全局缓存、配置对象、连接池等                              |
| ✅ 并发写操作      | 多任务同时更新一个对象                                 |
| ✅ 替代锁机制      | 不再需要手动写 `DispatchQueue`、`NSLock`            |
| ✅ 和结构化并发结合使用 | Swift Concurrency（`Task`、`await`）结构中保护状态很自然 |

---

- **🚫 actor 不适合的场景：**
	* 对性能极高要求，且自己精细控制线程同步（如音频、图形高频刷新等）
	* 极度简单的数据结构，只用于单线程访问

---

**🧪 示例：缓存系统**

```swift
actor ImageCache {
    private var cache: [URL: Data] = [:]
    
    func set(_ data: Data, for url: URL) {
        cache[url] = data
    }
    
    func get(for url: URL) -> Data? {
        return cache[url]
    }
}
```

使用：

```swift
let cache = ImageCache()

Task {
    if let data = await cache.get(for: imageURL) {
        print("Loaded from cache")
    } else {
        let data = try await fetchImageData(from: imageURL)
        await cache.set(data, for: imageURL)
    }
}
```


***

![ios0.0.67.png](./../../Pictures/ios0.0.67.png)

- **上图是截自于其他博客上的图,由上图可得:**
	- 值类型比引用类型快的多,因为值距离线程距离近的多;
	- 每个线程都有自己的栈,所以线程里的东西默认都是线程安全的.
		- 当传递值类型时,我们传递的是数据副本,不是原始数据的引用.所以当赋值或传递值类型时会创建一个新的数据副本,相反还有引用类型.
	- 类、函数、Actor都是引用类型,不是存在某个特定线程上,所以它们不是线程安全的;
	- 如上面的图: 堆离线程更远,所以访问栈比访问堆快得多,因为堆会在所有线程同步数据(所以读写数据较慢 )
		- 堆是在线程间共享的
		 - 每个线程都有自己的栈,每个线程都没有自己的堆,线程共享堆.因为堆需要和那些类进行同步,和那些actor进行同步.

***
<br/><br/><br/>
> <h2 id="结构体值类型新理解">结构体值类型新理解</h2>

```swift
struct MyStruct {
    var title: String
}

struct CustomStruct { 
    let title: String
    
    func updateTitle(newTitle: String) -> CustomStruct {
        CustomStruct(title: newTitle)
    }
}




let objectA = MyStruct(title: "Starting title!")
objectA.title = "Second title!" // 当我们把title变量由let改为var其实就是将这个旧的objectA废弃了,重新生成了一个结构体objectA,所以这个objectA也必须声明为var类型而不是let,若是你一开始就声明声明为let,然后改动其属性就会编译报错

// 上述的 objectA.title = "Second title!" 等价于下面的 struct2 = CustomStruct(title: "Title2")
var struct2 = CustomStruct(title: "Title1")
struct2 = CustomStruct(title: "Title2")
```


