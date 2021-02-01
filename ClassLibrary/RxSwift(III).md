- **网络请求**
- **UITableView**

<br/>

***
<br/>

># 网络请求

[RxSwift 网络请求封装](http://www.manongjc.com/article/3755.html)

网络安全请求合并：

```
 public func testRxZip() {
        
        let oneOb = Observable<Int>.create { observer -> Disposable in
            observer.on(.next(20))
            observer.on(.completed)
            return Disposables.create()
        }
        let twoOb = Observable<Int>.create { observer -> Disposable in
            observer.on(.next(80))
            observer.on(.completed)
            return Disposables.create()
        }
        
        Observable.zip(oneOb, twoOb).subscribe(onNext: {(one, two) in
            print("获取信息成功: \(one)")
            print("获取订单成功: \(two) 条")

            }).disposed(by: DisposeBag())
    }

///调用
self.testRxZip()
```
打印：

```
获取信息成功: 20
获取订单成功: 80 条
```


[**观察序列创建**](https://www.jianshu.com/p/2a45b502279a)



<br/>

***
<br/>

># UITableView

&emsp; 将 data 属性变成一个可观察序列对象（Observable Squence）;

&emsp; `可观察序列对象`，简单点来说就是“序列”，我们可以对这些数值进行“订阅（Subscribe）”，有点类似于“通知（NotificationCenter）”

- **`Model`**

```
//歌曲结构体
struct Music {
    let name: String //歌名
    let singer: String //演唱者
     
    init(name: String, singer: String) {
        self.name = name
        self.singer = singer
    }
}

//实现 CustomStringConvertible 协议，方便输出调试
extension Music: CustomStringConvertible {
    var description: String {
        return "name：\(name) singer：\(singer)"
    }
}
```

<br/>

- **`ShowCell`**

```
class ShowCell: UITableViewCell {
    lazy var mainTitle: UILabel = {
        let label = UILabel()
        label.font = UIFont.systemFont(ofSize: 16.0)
        
        return label
    }()
    
    lazy var subTitle: UILabel = {
        let label = UILabel()
        label.font = UIFont.systemFont(ofSize: 12.0)
        label.textColor = UIColor.lightGray
        
        return label
    }()
    
    class func cell(_ tableView: UITableView) -> ShowCell {
        let identifier = NSStringFromClass(self.classForCoder())
        var cell = tableView.dequeueReusableCell(withIdentifier: identifier) as? ShowCell
        if cell == nil {
            cell = ShowCell.init(style: .default, reuseIdentifier: identifier)
        }
        
        return cell ?? ShowCell()
    }
    
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        self.addSubViews()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func addSubViews() {
        
        self.contentView.addSubview(self.mainTitle)
        self.contentView.addSubview(self.subTitle)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        self.mainTitle.snp.makeConstraints { (make) in
            make.left.equalToSuperview().offset(16)
            make.top.equalToSuperview().offset(8)
        }
        self.subTitle.snp.makeConstraints { (make) in
            make.left.equalTo(self.mainTitle)
            make.top.equalTo(self.mainTitle.snp.bottom).offset(8)
        }
    }
}

```

<br/>

- **`ViewModel`**

```
//歌曲列表数据源
struct MusicListViewModel {
    let data = Observable.just([
        Music(name: "无条件", singer: "陈奕迅"),
        Music(name: "你曾是少年", singer: "S.H.E"),
        Music(name: "从前的我", singer: "陈洁仪"),
        Music(name: "在木星", singer: "朴树"),
    ])
}
```

<br/>

`ViewController` 中写一下响应式代码

```

private lazy var contentView: UITableView = {
        let tableView = UITableView()
        
        return tableView
    }()
    
let musicVM = MusicListViewModel()


override func viewDidLoad() {
        super.viewDidLoad()
        
        self.addTableView()
        self.layoutOfTableView()
        self.rxSwiftBindTableView()
        

    }


extension HGRegisterController: UITableViewDelegate {
    
    func rxSwiftBindTableView() {
        
        self.contentView.register(ShowCell.self, forCellReuseIdentifier: "ShowCell")
        //数据绑定
        self.musicVM.data.bind(to: self.contentView.rx.items(cellIdentifier: "ShowCell", cellType: ShowCell.self)){(row, model, cell) in
            cell.mainTitle.text = model.singer
            cell.subTitle.text = model.name
        }.disposed(by: disposeBag)
        
        //点击cell绑定
        self.contentView.rx.itemSelected.bind { (indexPath) in
            print("--->>: \(indexPath)")
        }.disposed(by: disposeBag)
        
        //代理绑定
        self.contentView.rx.setDelegate(self).disposed(by: disposeBag)
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 64
    }
    
    func addTableView() {
        self.view.addSubview(self.contentView)
    }
    
    func layoutOfTableView() {
        self.contentView.snp.makeConstraints { (make) in
            make.left.bottom.right.top.equalToSuperview()
            
        }
    }
}

```
**`rx.items(cellIdentifier:）`**这是 Rx 基于 `cellForRowAt` 数据源方法的一个封装。传统方式中我们还要有个` numberOfRowsInSection` 方法，使用 Rx 后就不再需要了（Rx 已经帮我们完成了相关工作）。

**`rx.modelSelected：`**这是 Rx 基于` UITableView `委托回调方法 `didSelectRowAt` 的一个封装

效果图：

![RxSwift 效果图](https://upload-images.jianshu.io/upload_images/2959789-6074eb1afab8a393.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打印：
![点击打印](https://upload-images.jianshu.io/upload_images/2959789-578a4b179461d423.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)














