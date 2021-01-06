># 安装
**`Moya 介绍`**
&emsp;  `Moya` 是一个基于 `Alamofire` 的更高层网络请求封装抽象层。`Moya` 也就可以看做我们的网络管理层，用来封装 URL、参数等请求所需要的一些基本信息。使用后我们的客户端代码会直接操作` Moya`，然后 `Moya `去管理请求，而不用跟 Alamofire 进行直接接触。

终端查找
```
$ pod search Moya
```

最新版本和资源信息
```
 pod 'Moya', '~> 14.0.0-beta.2'
   - Homepage: https://github.com/Moya/Moya
   - Source:   https://github.com/Moya/Moya.git
```

&emsp;  我们在使用`Moya`时，需要引入3个第三方库，如：`Moya `, `Alamofire `, `SwiftyJSON(方便解析返回的 JSON 数据)`

```
pod 'Moya', '~> 14.0.0-beta.2'
pod 'Alamofire', '~> 5.0.0-rc.2'
pod 'SwiftyJSON', '~> 5.0.0'
```



<br/>
![使用 Moya 之前和之后](https://upload-images.jianshu.io/upload_images/2959789-bad45770b8d8ec89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Moya 的特点**
- 编译时检查正确的API端点访问。
- 允许您定义具有关联枚举值的不同端点的明确用法。
- 将测试存根视为一等公民，因此单元测试非常容易。


<br/>
***
<br/>

># Moya 实战

<br/>
**`接口枚举`**
```
//初始化请求的provider
let NetProvider = MoyaProvider<NetRequestAPI>()

/**定义请求的endpoints（供provider使用）**/
public enum NetRequestAPI {
    case channels                        //获取频道接口
    case playlist(String)                //获取歌曲接口
    case otherRequest                   // 其他接口,没有参数
}
```

<br/>
**`扩展封装`**
```
//请求配置
/*
 baseURL：服务器地址host 处理
 path：根据不同的接口，确定各个请求的具体路径
 method：根据不同的接口，设置请求方式
 headers：统一配置的请求头信息配置
 task：配置内部参数，以及task信息
 */
extension NetRequestAPI: TargetType {
    
    //服务器地址
    public var baseURL: URL {
        switch self {
        case .channels:
            return URL(string: "https://www.douban.com")!
            
        case .playlist(_):
            return URL(string: "https://douban.fm")!
        case .otherRequest:
            return URL(string: "https://douban.fm/default.html")!
        }

    }
    
    // 各个请求的具体路径
    public var path: String {
        switch self {
        case .channels:
            return "/j/app/radio/channels"
            
        case .playlist(_):
            return "/j/mine/playlist"
            
        case .otherRequest:
            return "/default/otherRequest"
        }
    }
    
    
    //请求方式
    public var method: Moya.Method {
        switch self {
        case .channels:
            return .get
            
        case .playlist(_):
            return .get
            
        default:
            return .post
        }
    }
    
    
    //请求任务事件（这里附带上参数）
    public var task: Task {
        var param: [String: Any] = [:]
        switch self {
        case .playlist(let channel):
            param["channel"] = channel
            param["type"] = "n"
            param["from"] = "mainsite"
            break
        
        case .channels: break
            
        case .otherRequest:
            return .requestPlain
        }
        
        return .requestParameters(parameters: param, encoding: URLEncoding.default)
    }
    
    //是否执行Alamofire验证
    public var validate: Bool {
        return false
    }
    
    //做单元测试模拟的数据，只会在单元测试文件中有作用
    public var sampleData: Data {
        return "{}".data(using: String.Encoding.utf8, allowLossyConversion: true)!
    }
    
    //请求头设置
    public var headers: [String : String]? {
        return nil
    }
    
}
```


<br/>
**`第一种调用方式：在 Controller 中直接调用`**
```
override func viewDidLoad() {
        super.viewDidLoad()
        
        NetProvider.request(.channels) { (result) in
            if case let .success(response) = result { response.data
                //
                let data = try? response.mapJSON()  //请求response转化为JSON格式
                let json = JSON(data!)  //转化为JSON数据格式
                print("channels 接口数据:\(json["channels"].arrayValue)")
            }
            
            //主线程刷新UI
            DispatchQueue.main.async {
                // self.tableView.reloadData()
            }
        }
    }
```
控制台打印：
![属性data和statusCode打印](https://upload-images.jianshu.io/upload_images/2959789-9fe35b788c4600b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![request 属性打印](https://upload-images.jianshu.io/upload_images/2959789-2e2d76fc83c94a9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
**`第二种调用方式：VM 模块`**
**功能 VM 模块处理**
```
/*
 MoyaProvider 是此次网络请求的信息提供者
 MoyaProvider 根据模块 NetRequestAPI 设置的信息绑定数据请求
 MoyaProvider 通过调用 request 方法传出此次请求的接口，但是参数需要应用层提供！
 获取回调信息，然后进行 json 序列化!
 最后利用函数式编程思想回调 携带信息的闭包 给应用层
 */

//登录模块管理
class LoginViewModel: NSObject {
    static let manager = LoginViewModel()
    
    //验证码事件
    func getChannel(username: String?, complete:@escaping((Any)-> Void)) {
        let provider = MoyaProvider<NetRequestAPI>()
        
        provider.request(.channels) { (result) in
            switch result {
            case let .success(response):
                let dict = JSON(response.data)
                complete(dict)
                
            case let .failure(error):
                print(error)
                complete("")
            }
        }
    }
}

```

*  ` .success(Moya.Response)：`成功的情况。我们可以从 Moya.Response 中得到返回数据（data）和状态（status ）；
*   ` .failure(MoyaError)：`失败的情况。这里的失败指的是服务器没有收到请求（例如可达性/连接性错误）或者没有发送响应（例如请求超时）。我们可以在这里设置个延迟请求，过段时间重新发送请求。


&emsp; 对于` .failure(error)` 情况下，我们还可以通过 switch 语句判断具体的 MoyaError 错误类型：
```
case let .failure(error):
    switch error {
    case .imageMapping(let response):
        print("错误原因：\(error.errorDescription ?? "")")
        print(response)
    case .jsonMapping(let response):
        print("错误原因：\(error.errorDescription ?? "")")
        print(response)
    case .statusCode(let response):
        print("错误原因：\(error.errorDescription ?? "")")
        print(response)
    case .stringMapping(let response):
        print("错误原因：\(error.errorDescription ?? "")")
        print(response)
    case .underlying(let error, let response):
        print("错误原因：\(error.errorDescription ?? "")")
        print(error)
        print(response as Any)
    case .requestMapping:
        print("错误原因：\(error.errorDescription ?? "")")
        print("nil")
    }
```

<br/>
**`过滤正确状态码`**
&emsp;  除了连接超时这样的网络问题会返回` .failure`，像是服务器报错（404）、请求未授权（401）等都是返回` .success 的`。这样我们还需要根据状态码来判断返回数据是否是正确的。
`Moya.Response` 本身就提供了下面两个方法来过滤 response 响应：
* `filterSuccessfulStatusCodes()：`返回状态码为 200 - 299 的响应
* `filterSuccessfulStatusAndRedirectCodes()：`返回状态码为 200 - 399 的响应

```
let provider = MoyaProvider<NetRequestAPI>()

provider.request(.channels) { (result) in
    switch result {
    case let .success(response):
        do {
            //过滤成功的状态码响应
            try response.filterSuccessfulStatusCodes()
            let data = try response.mapJSON()
            let dict = JSON(data)
            complete(dict)
            //继续做一些其它事情....
        }
        catch {
            //处理错误状态码的响应...
        }
        
        
    case let .failure(error):
        print(error)
        complete("")
    }
}

```


<br/>
**Controller 模拟调用**
```
override func viewDidLoad() {
        super.viewDidLoad()
        
        //应用层只需要为此次网络提供信息参数
        //在回调闭包拿到信息，处理其他业务就OK!
        //VM层调用
        LoginViewModel.manager.getChannel(username: nil) { [weak self](responseData) in
            //self?.smscodeTF.text = smscode
            print("Controller调用：\(responseData)")
        }
}
```
打印图:
![JSON 数据](https://upload-images.jianshu.io/upload_images/2959789-7b6876be2b1a65ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
**`封装一个请求/结果处理的适配器`**
&emsp;  根据项目需求我们可以自行定义一个`适配器（adapter）`，将请求和结果的判断处理都封装起来。然后通过三个回调函数返回相应的结果，这三个回调函数分别对应三种响应情况：

*   `success：`服务器连接成功，且数据正确返回（同时会自动将数据转换成 JSON 对象，方便使用）
*  ` error：`服务器连接成功，但数据返回错误（同时会返回错误信息）
*   `failure ：`服务器连接不上，网络异常等（同时会返回错误信息。必要的话，还可以在此增加自动重新请求的机制。）


```
struct Network {
    static let provider = MoyaProvider<DouBan>()
     
    static func request(
        _ target: DouBan,
        success successCallback: @escaping (JSON) -> Void,
        error errorCallback: @escaping (Int) -> Void,
        failure failureCallback: @escaping (MoyaError) -> Void
        ) {
        provider.request(target) { result in
            switch result {
            case let .success(response):
                do {
                    //如果数据返回成功则直接将结果转为JSON
                    try response.filterSuccessfulStatusCodes()
                    let json = try JSON(response.mapJSON())
                    successCallback(json)
                }
                catch let error {
                    //如果数据获取失败，则返回错误状态码
                    errorCallback((error as! MoyaError).response!.statusCode)
                }
            case let .failure(error):
                //如果连接异常，则返沪错误信息（必要时还可以将尝试重新发起请求）
                //if target.shouldRetry {
                //    retryWhenReachable(target, successCallback, errorCallback,
                //      failureCallback)
                //}
                //else {
                    failureCallback(error)
                //}
            }
        }
    }
}
```

<br/>
**Moya模型总结：CFNextwork -> Alamofire -> Moya -> 业务层**
![Moya 直接调用网络逻辑流程图](https://upload-images.jianshu.io/upload_images/2959789-7112b5c763fa3427.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`弊端：`**
- 若整个项目是` RxSwift`(毕竟现在函数响应式编程已成为趋势)，显然我们更需要序列，因为序列可以直接绑定响应UI，更便于开发;
- 当前直接使用 Moya ,还缺了很严重的一步即：模型化。

<br/>
***
<br/>
># RxSwift 结合 Aoya
<br/>
`真实的网络架构层的模块希望是下面这样的：`
**Moya模型总结：CFNextwork -> Alamofire -> Moya -> 模型化 -> RxSwift -> 业务层**
![RxSwift 结合 Moya](https://upload-images.jianshu.io/upload_images/2959789-a97e81c23337396e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
首先要使用`CocoaPods` 在 `Podfile` 文件中进行如下配置：
```
pod 'Moya', '~> 14.0.0-beta.2' 
pod 'Alamofire', '~> 5.0.0-rc.2'
pod 'SwiftyJSON', '~> 5.0.0'
pod 'ObjectMapper', '~> 3.5.1'
```
导入文件：
```
@_exported import Alamofire
@_exported import SwiftyJSON
@_exported import ObjectMapper
```

<br/>
实战代码如下：
```
/*
 首先拓展 PrimitiveSequence 实际对象是处理 Moya.Response
 通过调用 SwiftyJSON 把 Response 的 data 解析成 json
 然后调用 ObjectMapper 转成相应模型数据
 数组模型处理差不多，大家只要返回 [T] 就 OK
 */
extension PrimitiveSequence where Trait == SingleTrait, Element == Moya.Response {
    func map<T: ImmutableMappable>(_ type: T.Type) -> PrimitiveSequence<TraitType, T> {
           return self
               .map { (response) -> T in
                 let json = try JSON(data: response.data)
                 guard let code = json[RESULT_CODE].int else { throw RequestError.noCodeKey }
                 if code != StatusCode.success.rawValue { throw RequestError.sysError(statusCode:"\(code)" , errorMsg: json[RESULT_MESSAGE].string) }
                if let data = json[RESULT_DATA].dictionaryObject {
                    return try Mapper<T>().map(JSON: data)
                }else if let data = json[RESULT_RESULT].dictionaryObject {
                    return try Mapper<T>().map(JSON: data)
                }
                 throw RequestError.noDataKey
            }.do(onSuccess: { (_) in
                
            }, onError: { (error) in
                if error is MapError {
                    log.error(error)
                }
            })
        }
}

```

**外界调用**
```
loginService.login().asObservable()
    .subscribe(onNext: {[weak self] (rcmdBranchModel) in
        
        guard let `self` = self else { return }
        self.requestIds = rcmdBranchModel.tab.map{$0.id}
        self.menuTitles += rcmdBranchModel.tab.map{$0.name}
        self.pageController.magicView.reloadData(toPage: 1)
    }).disposed(by: disposeBag)

```


<br/>
***
<br/>
参考资料
[](https://juejin.im/post/5d6fb6ef51882520d46ab27e)
[网络抽象层库Moya的使用详解4](https://www.hangge.com/blog/cache/detail_1807.html)
[ Moya网络层框架的学习笔记 US](https://www.jianshu.com/p/2da7dbeac112)
[ Moya使用理解 US](https://www.jianshu.com/p/219b197a230a)
[moya的使用 US](https://www.jianshu.com/p/2ee5258828ff)
[系统性学习Moya+Alamofire+RxSwift+ObjectMapper的配合使用 US](http://www.cocoachina.com/articles/19827)
