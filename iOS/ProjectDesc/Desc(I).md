
- [**优化项目方案**](https://philm.gitbook.io/philm-ios-wiki/)
- [**SwiftHub**](https://gitee.com/harelyio/work/tree/master/Project/SwiftHub)
	- 项目解析
	- 类库使用
	- 库方法使用
- **[MVVM之从理论到实践](https://www.jianshu.com/p/1c1b6bc557ac)**
	- [DemoCode](https://gitee.com/harelyio/work/tree/master/Project/MVVMReactive)
- **[仿SDWebImage逻辑](https://gitee.com/harelyio/work/tree/master/Project/ImageLoadSDK)**
- **[C++数据结构算法](https://gitee.com/harelyio/work/tree/master/Project/C2)**
- **SwiftEOS**
- [**开源库整理**](http://blogwenbo.com/2019/09/03/iOS优秀Swift开源库整理)



<br/>

***
<br/>

># SwiftHub

<br/>

> 项目解析

- [**SwiftHub项目解析**](https://juejin.cn/post/6844904127307186189)

<br/>

> 类库使用

- [**ObjectMapper**](https://www.jianshu.com/p/609fbdb62274)
	- Json数据解析
	- `pod 'ObjectMapper', '~> 4.2.0'`
- RxRelay 
	- `pod 'RxRelay', '~> 5.1.1'`
-  RxSwift
	- `pod 'RxSwift', '~> 5.1.1'`
-  RxTheme
	- ` pod 'RxTheme', '~> 4.1.1'`
	- AXThemeKit (0.0.3): 这是一个简单的主题框架，可以更换主题色、字体、icon，支持主题商店. `pod 'AXThemeKit', '~> 0.0.3'`
	- JXTheme (0.0.6): 主题、换肤、暗黑模式, `pod 'JXTheme', '~> 0.0.6'`
-  [**R.swift**](https://www.jianshu.com/p/727acc03f7b1) 
	-  iOS资源引入框架
-  [**CocoaLumberjackDemo**](https://www.jianshu.com/p/7b799bef0107)
	- 将程序运行过程中产生的Log保存起来或者发送到自己服务器，为了以后方便分析。
	- ` pod 'CocoaLumberjack-RemoteAccess', '~> 0.1.0'`
	- [**全量日志捕获CocoaLumberjack**](https://blog.csdn.net/shengpeng3344/article/details/105148752)
	- [**源码浅析 - CocoaLumberjack 3.6 之 DDLog**](https://juejin.cn/post/6844904147511164942#heading-25)
- **Toast-Swift**：提示框
	- [自定义Toast提示框](https://www.jianshu.com/p/bdfb174ddcf9) 




<br/>
<br/>

> **库方法使用**

<br/>
- ReplaySubject和skip

```
 let disposeBag = DisposeBag()
    
//创建一个bufferSize为2的ReplaySubject
let subject = ReplaySubject<String>.create(bufferSize: 1)

func replaySubjectTest() {
    
    //连续发送3个next事件
    subject.onNext("111")

    //skip: https://www.hangge.com/blog/cache/detail_1933.html
    //skip: 忽略1个序列，但是若是缓存序列则是对它无用
    subject.skip(1).subscribe(onNext: { (connected) in
        print("连接状态： \(connected)")
        
    }).disposed(by: disposeBag)
}
```

打印：

`连接状态： 111`

<br/>


- RxController类库

BaseViewModel.swift

```
import Foundation
import RxSwift
import RxCocoa

class BaseViewModel: NSObject {
    
    struct Input {
        let modules: Observable<Void>
    }
    
    struct Output {
        //Driver：https://www.hangge.com/blog/cache/detail_1942.html
        let titles: Driver<[String]>
        let values: Driver<[String]>
    }

    func transform(input: Input) -> Output {
        
        let titles = Observable.just(true).map { (status) -> [String] in
            if status {
                return ["首页", "新闻", "资产", "设置"]
            }else {
                return ["1", "2", "3"]
            }
        }.asDriver(onErrorJustReturn: [])
        
        
        
        let values = input.modules.take(1).map{ _ -> [String] in
            
            return ["one", "two","there","four"]
            
        }.asDriver(onErrorJustReturn: [])
        
        return Output(titles: titles, values: values)
    }
    
    
}



//RxSwfit Extensions
extension ObservableType {
    
    func mapToVoid() -> Observable<Void> {
        return map { _ in }
    }
}


```

ViewController.swift

```

override func viewDidLoad() {
    super.viewDidLoad()
    
    
    
    rxTest1()
}

//rxTest1 方法必须放在viewDidLoad方法中，否则rx.viewWillAppear.mapToVoid()无法其作用
func rxTest1() {
    let baseVM = BaseViewModel()

    //rx: 进入RxSwift世界入口，https://iweiyun.github.io/2018/11/01/rxcocoa-code/
    let input = BaseViewModel.Input(modules: rx.viewWillAppear.mapToVoid())
    let output = baseVM.transform(input: input)
    
    output.titles.delay(.milliseconds(50)).drive(onNext: {[weak self] (titles) in
        if let strongSelf = self {
            print(">>>>>>>> titles: \(titles), strongSelf: \(strongSelf)")
        }
    }).disposed(by: disposeBag)
    
    
    output.values.drive(onNext: {(values) in
        print("========== values: \(values)")
    }).disposed(by: disposeBag)
    
    
}
```

打印：

```
========== values: ["one", "two", "there", "four"]
>>>>>>>> titles: ["首页", "新闻", "资产", "设置"], strongSelf: <SwiftTest.ViewController: 0x7f806be09230>

```

<br/>



```




let searchModeSelection: Observable<SearchModeSegments>
//SearchModeSegments 是一个枚举，.trending 该枚举的一种情况
let searchMode = BehaviorRelay<SearchModeSegments>(value: .trending)

//input是结构体，sortRepositorySelection 是 searchModeSelection 类型
input.sortRepositorySelection.bind(to: sortRepositoryItem).disposed(by: rx.disposeBag)
```



```

//Observable.combineLatest ？？
//filter ？？
//flatMapLatest ？？

Observable.combineLatest(keyword, currentLanguage, sortRepositoryItem)
            .filter({ (keyword, currentLanguage, sortRepositoryItem) -> Bool in
                return keyword.isNotEmpty || currentLanguage != nil
            })
            .flatMapLatest({ [weak self] (keyword, currentLanguage, sortRepositoryItem) -> Observable<RxSwift.Event<RepositorySearch>> in
                guard let self = self else { return Observable.just(RxSwift.Event.next(RepositorySearch())) }
                self.repositoriesPage = 1
                let query = self.makeQuery()
                let sort = sortRepositoryItem.sortValue
                let order = sortRepositoryItem.orderValue
                return self.provider.searchRepositories(query: query, sort: sort, order: order, page: self.repositoriesPage, endCursor: nil)
                    .trackActivity(self.loading)
                    .trackActivity(self.headerLoading)
                    .trackError(self.error)
                    .materialize()
            }).subscribe(onNext: { [weak self] (event) in
                switch event {
                case .next(let result):
                    //BehaviorRelay ？？
					//let repositorySearchElements = BehaviorRelay(value: RepositorySearch())
					//accept？？
					self?.repositorySearchElements.accept(result)
                default: break
                }
            }).disposed(by: rx.disposeBag)
```


<br/>

```

class TableViewCellViewModel: NSObject {

}

class DefaultTableViewCellViewModel: TableViewCellViewModel {
    let title = BehaviorRelay<String?>(value: nil)
    let detail = BehaviorRelay<String?>(value: nil)
    let secondDetail = BehaviorRelay<String?>(value: nil)
    let attributedDetail = BehaviorRelay<NSAttributedString?>(value: nil)
    let image = BehaviorRelay<UIImage?>(value: nil)
    let imageUrl = BehaviorRelay<String?>(value: nil)
    let badge = BehaviorRelay<UIImage?>(value: nil)
    let badgeColor = BehaviorRelay<UIColor?>(value: nil)
    let hidesDisclosure = BehaviorRelay<Bool>(value: false)
}


class SettingCellViewModel: DefaultTableViewCellViewModel {

    init(with title: String, detail: String?, image: UIImage?, hidesDisclosure: Bool) {
        super.init()
        self.title.accept(title)
        self.secondDetail.accept(detail)
        self.image.accept(image)
        self.hidesDisclosure.accept(hidesDisclosure)
    }
}


let logoutCellViewModel = SettingCellViewModel(with: R.string.localizable.settingsLogOutTitle.key.localized(), detail: nil,image: R.image.icon_cell_logout()?.template, hidesDisclosure: true)


```





<br/>

***
<br/>

># MVVM之从理论到实践

RACSignal的几个操作:

```
map：使用提供的block来转换事件数据。

filter：使用提供的block来觉得事件是否往下传递。

combineLatest:reduce:：组合源信号数组中的信号，并生成一个新的信号。每次源信号数组中的一个输出新值时，reduce块都会被执行，而返回的值会作为组合信号的下一个值。

RAC宏：将信号的输入值指派给对象的属性。它带有两个参数，第一个参数是对象，第二个参数是对象的属性名。每次信号发送下一个事件时，其输出值都会指派给给定的属性。

rac_signalForControlEvents：从按钮的参数事件中创建一个信号。

注意：上面的操作输出仍然是一个RACSignal对象，所以不需要使用变量就可以构建一个管道。

doNext：附加操作，并不返回一个值。
```



<br/>

***
<br/>

># [仿SDWebImage逻辑](https://juejin.im/post/6844903807667666951)





<br/>

***
<br/>

># [C++数据结构算法](https://github.com/harleyGit/StudyNotes/blob/master/C语言/C2Exercise(I).md)



<br/>

***
<br/>

># SwiftEOS
加密库
