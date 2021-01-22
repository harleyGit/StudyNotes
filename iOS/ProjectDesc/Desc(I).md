
- [**SwiftHub**](https://gitee.com/harelyio/work/tree/master/Project/SwiftHub)
- **[MVVM之从理论到实践](https://www.jianshu.com/p/1c1b6bc557ac)**
	- [DemoCode](https://gitee.com/harelyio/work/tree/master/Project/MVVMReactive)
- **[仿SDWebImage逻辑](https://gitee.com/harelyio/work/tree/master/Project/ImageLoadSDK)**
- **[C++数据结构算法](https://gitee.com/harelyio/work/tree/master/Project/C2)**
- **SwiftEOS**



<br/>

***
<br/>

># SwiftHub

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




<br/>
<br/>

> 库方法使用

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
