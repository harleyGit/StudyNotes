> <h1 id=""></h1>
- [**‌Moya使用介绍**](#Moya使用介绍)
- [Mock数据](#Mock数据)



<br/>

***
<br/><br/><br/>

> <h1 id="Moya使用介绍">Moya使用介绍</h1>

Moya 是一个用于网络请求的 Swift 库，它为网络请求的封装提供了一个简洁而优雅的接口。MoyaProvider 是 Moya 的核心类，用于创建和发送网络请求。下面是一个使用 MoyaProvider 的完整示例，包括定义一个 API，配置 MoyaProvider，并发起一个网络请求。


```
import Moya

// 1. 定义 API 枚举
enum MyAPI {
    case getUser(id: Int)
    case createUser(name: String, age: Int)
    case updateUser(id: Int, name: String, age: Int)
    case deleteUser(id: Int)
}

// 2. 遵守 TargetType 协议
extension MyAPI: TargetType {
    var baseURL: URL {
        return URL(string: "https://jsonplaceholder.typicode.com")!
    }
    
    var path: String {
        switch self {
        case .getUser(let id):
            return "/users/\(id)"
        case .createUser:
            return "/users"
        case .updateUser(let id, _, _):
            return "/users/\(id)"
        case .deleteUser(let id):
            return "/users/\(id)"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUser:
            return .get
        case .createUser:
            return .post
        case .updateUser:
            return .put
        case .deleteUser:
            return .delete
        }
    }
    
    var task: Task {
        switch self {
        case .getUser:
            return .requestPlain
        case .createUser(let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        case .updateUser(_, let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        case .deleteUser:
            return .requestPlain
        }
    }
    
    var headers: [String: String]? {
        return ["Content-Type": "application/json"]
    }
    
    var sampleData: Data {
        return Data()
    }
}

```


使用Moya进行请求调用：

```
// 3. 使用 MoyaProvider 进行请求
let provider = MoyaProvider<MyAPI>()

// 3.1 调用 getUser 请求
provider.request(.getUser(id: 1)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("User Data: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.2 调用 createUser 请求
provider.request(.createUser(name: "John Doe", age: 30)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("Created User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.3 调用 updateUser 请求
provider.request(.updateUser(id: 1, name: "Jane Doe", age: 25)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("Updated User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.4 调用 deleteUser 请求
provider.request(.deleteUser(id: 1)) { result in
    switch result {
    case .success(let response):
        print("User deleted successfully")
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

```

在使用 Moya 的 `MoyaProvider` 时，调用请求的过程是非常简单的。下面我将展示如何调用不同的 API 请求并处理响应的完整例子。

### 完整代码示例

```swift
import Moya

// 1. 定义 API 枚举
enum MyAPI {
    case getUser(id: Int)
    case createUser(name: String, age: Int)
    case updateUser(id: Int, name: String, age: Int)
    case deleteUser(id: Int)
}

// 2. 遵守 TargetType 协议
extension MyAPI: TargetType {
    var baseURL: URL {
        return URL(string: "https://jsonplaceholder.typicode.com")!
    }
    
    var path: String {
        switch self {
        case .getUser(let id):
            return "/users/\(id)"
        case .createUser:
            return "/users"
        case .updateUser(let id, _, _):
            return "/users/\(id)"
        case .deleteUser(let id):
            return "/users/\(id)"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUser:
            return .get
        case .createUser:
            return .post
        case .updateUser:
            return .put
        case .deleteUser:
            return .delete
        }
    }
    
    var task: Task {
        switch self {
        case .getUser:
            return .requestPlain
        case .createUser(let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        case .updateUser(_, let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        case .deleteUser:
            return .requestPlain
        }
    }
    
    var headers: [String: String]? {
        return ["Content-Type": "application/json"]
    }
    
    var sampleData: Data {
        return Data()
    }
}

// 3. 使用 MoyaProvider 进行请求
let provider = MoyaProvider<MyAPI>()

// 3.1 调用 getUser 请求
provider.request(.getUser(id: 1)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("User Data: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.2 调用 createUser 请求
provider.request(.createUser(name: "John Doe", age: 30)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("Created User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.3 调用 updateUser 请求
provider.request(.updateUser(id: 1, name: "Jane Doe", age: 25)) { result in
    switch result {
    case .success(let response):
        do {
            // 解析 JSON 数据
            if let json = try response.mapJSON() as? [String: Any] {
                print("Updated User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 3.4 调用 deleteUser 请求
provider.request(.deleteUser(id: 1)) { result in
    switch result {
    case .success(let response):
        print("User deleted successfully")
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}
```

- **详细说明**

	- **1.定义 API 枚举**: 通过定义 `MyAPI` 枚举，并实现 `TargetType` 协议，定义了与 API 相关的所有端点、HTTP 方法、参数、以及请求头信息。

	- **2.使用 MoyaProvider 发起请求**: `provider.request` 方法用于发起 API 请求，使用枚举值来指定请求的类型。例如，`.getUser(id: 1)` 表示发起一个 GET 请求来获取 ID 为 1 的用户数据。

	- **3.处理响应**: 通过 `switch` 语句处理请求结果，如果请求成功，`result` 会是 `.success`，并且包含响应数据。你可以通过 `response.mapJSON()` 方法将响应转换为 JSON 格式。对于失败的请求，`result` 会是 `.failure`，你可以获取错误信息并进行相应的处理。

	- **4.更多调用实例**: 上面展示了 `getUser`、`createUser`、`updateUser` 和 `deleteUser` 的不同调用方式，展示了如何通过 MoyaProvider 轻松调用不同的 API 端点。




<br/>

***
<br/><br/><br/>

> <h1 id="Mock数据">Mock数据</h1>

```
import Moya

// 1. 定义 API 枚举
enum MyAPI {
    case getUser(id: Int)
    case createUser(name: String, age: Int)
}

// 2. 遵守 TargetType 协议
extension MyAPI: TargetType {
    var baseURL: URL {
        return URL(string: "https://jsonplaceholder.typicode.com")!
    }
    
    var path: String {
        switch self {
        case .getUser(let id):
            return "/users/\(id)"
        case .createUser:
            return "/users"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUser:
            return .get
        case .createUser:
            return .post
        }
    }
    
    var task: Task {
        switch self {
        case .getUser:
            return .requestPlain
        case .createUser(let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        }
    }
    
    var headers: [String: String]? {
        return ["Content-Type": "application/json"]
    }
    
    // 3. 定义模拟数据
    var sampleData: Data {
        switch self {
        case .getUser(let id):
            // 模拟返回的用户数据
            return """
            {
                "id": \(id),
                "name": "John Doe",
                "age": 30
            }
            """.data(using: .utf8)!
            
        case .createUser(let name, let age):
            // 模拟返回的新创建用户数据
            return """
            {
                "id": 100,
                "name": "\(name)",
                "age": \(age)
            }
            """.data(using: .utf8)!
        }
    }
}

// 4. 使用 MoyaProvider 进行测试请求
let provider = MoyaProvider<MyAPI>(stubClosure: MoyaProvider.immediatelyStub)

// 4.1 测试 getUser 请求
provider.request(.getUser(id: 1)) { result in
    switch result {
    case .success(let response):
        do {
            if let json = try response.mapJSON() as? [String: Any] {
                print("Mocked User Data: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 4.2 测试 createUser 请求
provider.request(.createUser(name: "Jane Doe", age: 25)) { result in
    switch result {
    case .success(let response):
        do {
            if let json = try response.mapJSON() as? [String: Any] {
                print("Mocked Created User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

```



是的，你可以使用 `sampleData` 来提供模拟数据，这对于单元测试非常有用。通过 `sampleData`，你可以模拟网络请求的响应数据，而不必真正发起网络请求。这样可以帮助你在没有真实服务器响应的情况下进行测试。

下面是一个包含 `sampleData` 的完整示例，演示如何使用 Moya 进行网络请求的模拟和测试。

### 示例代码

```swift
import Moya

// 1. 定义 API 枚举
enum MyAPI {
    case getUser(id: Int)
    case createUser(name: String, age: Int)
}

// 2. 遵守 TargetType 协议
extension MyAPI: TargetType {
    var baseURL: URL {
        return URL(string: "https://jsonplaceholder.typicode.com")!
    }
    
    var path: String {
        switch self {
        case .getUser(let id):
            return "/users/\(id)"
        case .createUser:
            return "/users"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUser:
            return .get
        case .createUser:
            return .post
        }
    }
    
    var task: Task {
        switch self {
        case .getUser:
            return .requestPlain
        case .createUser(let name, let age):
            return .requestParameters(parameters: ["name": name, "age": age], encoding: JSONEncoding.default)
        }
    }
    
    var headers: [String: String]? {
        return ["Content-Type": "application/json"]
    }
    
    // 3. 定义模拟数据
    var sampleData: Data {
        switch self {
        case .getUser(let id):
            // 模拟返回的用户数据
            return """
            {
                "id": \(id),
                "name": "John Doe",
                "age": 30
            }
            """.data(using: .utf8)!
            
        case .createUser(let name, let age):
            // 模拟返回的新创建用户数据
            return """
            {
                "id": 100,
                "name": "\(name)",
                "age": \(age)
            }
            """.data(using: .utf8)!
        }
    }
}

// 4. 使用 MoyaProvider 进行测试请求
let provider = MoyaProvider<MyAPI>(stubClosure: MoyaProvider.immediatelyStub)

// 4.1 测试 getUser 请求
provider.request(.getUser(id: 1)) { result in
    switch result {
    case .success(let response):
        do {
            if let json = try response.mapJSON() as? [String: Any] {
                print("Mocked User Data: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}

// 4.2 测试 createUser 请求
provider.request(.createUser(name: "Jane Doe", age: 25)) { result in
    switch result {
    case .success(let response):
        do {
            if let json = try response.mapJSON() as? [String: Any] {
                print("Mocked Created User: \(json)")
            }
        } catch {
            print("Failed to map response: \(error)")
        }
    case .failure(let error):
        print("Request failed with error: \(error)")
    }
}
```

- **详细说明**

	- **1.定义模拟数据**: 在 `sampleData` 属性中为每个 API 端点返回不同的模拟数据。这个数据将在测试中使用，而不是从实际的网络请求中获取。

	- **2.使用 MoyaProvider 进行测试请求**: 创建 `MoyaProvider` 时使用 `stubClosure` 参数，将其设置为 `MoyaProvider.immediatelyStub`，这意味着请求会立即返回 `sampleData`，而不会真正发起网络请求。这非常适合单元测试。

	- **3.测试 getUser 和 createUser**: 调用 `getUser` 和 `createUser` 方法时，返回的是 `sampleData` 中定义的模拟数据，而不是从服务器获取的数据。

	- **4.输出结果**: 运行代码时，你将看到模拟的用户数据和创建用户的数据被打印出来。


<br/>

***

<br/><br/><br/>

> <h1 id="网络请求配置自定义">网络请求配置自定义</h1>

形如：

```
let dataProvider = MoyaProvider<xxxxxVideoRequest>(endpointClosure: MoyaProvider.xxxxAuthorizedEndpointMapping)
```


这句代码涉及到使用 Moya 库创建一个 `MoyaProvider` 对象，并且使用了自定义的 `endpointClosure` 来对网络请求进行一些特定的配置。下面是对这句代码的详细解释：

- **1.`MoyaProvider<xxxxxVideoRequest>`**

	- `MoyaProvider` 是 Moya 库的核心类，用于创建和发送网络请求。
	- `<xxxxxVideoRequest>` 是你定义的一个枚举或类，通常遵循 Moya 的 `TargetType` 协议，用来定义 API 请求的相关信息，比如请求的 URL、HTTP 方法、参数等。
	- 通过泛型 `xxxxxVideoRequest`，你告诉 `MoyaProvider` 这个实例将会用于处理 `xxxxxVideoRequest` 类型的 API 请求。

<br/>

- **2. `endpointClosure: MoyaProvider.xxxxAuthorizedEndpointMapping`**

	- `endpointClosure` 是 `MoyaProvider` 的一个初始化参数，用于自定义 `Endpoint` 对象的生成方式。
	- `Endpoint` 对象包含了一个请求的详细信息，包括 URL、参数、HTTP 方法、请求头等。
	- `MoyaProvider.xxxxAuthorizedEndpointMapping` 是一个自定义的闭包或方法，用于生成带有授权信息的 `Endpoint`。这通常用于在请求中添加认证信息，比如 token，或者修改请求的某些部分（例如：添加公共的请求头）。

<br/>

- **3.`endpointClosure` 详解**

默认情况下，Moya 使用一个标准的 `endpointClosure` 来生成 `Endpoint` 对象，但你可以通过自定义 `endpointClosure` 来改变生成的方式。

`MoyaProvider.xxxxAuthorizedEndpointMapping` 很可能是你自己实现的一个闭包，类似于以下代码：

```swift
let xxxxAuthorizedEndpointMapping = { (target: xxxVideoRequest) -> Endpoint in
    var endpoint = MoyaProvider.defaultEndpointMapping(for: target)
    
    // 在这里可以修改 endpoint，比如添加授权信息
    let token = "your_auth_token"
    endpoint = endpoint.adding(newHTTPHeaderFields: ["Authorization": "Bearer \(token)"])
    
    return endpoint
}


//或者这样定义
static func xxxxEndpointMapping(
        for target: Target
    ) -> Endpoint where Target: xxxAuthorizedTargetType {
        let hobbyEndpoint = MoyaProvider.xxxEndpointMapping(for: target)
        var headers = [String: String]()
        let data: Data = (try? xxxEndpoint.urlRequest().httpBody) ?? "".data(using: .utf8)!
        headers["wToken"] = AliTigerTally.sharedInstance().vmpSign(data)
        guard target.needsAuth,
              let token = DependencyContainer.shared.appInfoRepo.accessToken else {
                  return xxxxEndpoint.adding(newHTTPHeaderFields: headers)
              }
        headers["Token"] = token
        return xxxxEndpoint.adding(newHTTPHeaderFields: headers)
    }
```

然后你可以像在代码中一样使用它：

```swift
let dataProvider = MoyaProvider<xxxxVideoRequest>(endpointClosure: xxxxAuthorizedEndpointMapping)
```

这个自定义的 `endpointClosure` 可以让你在每次请求中自动加入授权信息，或者做其他你需要的定制处理。

<br/>

- **总结**

	- **`dataProvider`** 是一个 `MoyaProvider` 实例，用于发送和处理与 `StarCurtainSmartVideoRequest` 相关的网络请求。
	- **`endpointClosure`** 是一个闭包，用来自定义每个请求的 `Endpoint` 对象，在这里你可以添加认证信息或修改请求配置。
	- **`MoyaProvider.hobbyAuthorizedEndpointMapping`** 是这个 `endpointClosure` 的一个实现，用于定制请求的生成，通常用于添加授权信息。















