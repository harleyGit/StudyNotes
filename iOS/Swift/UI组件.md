

> <h2 id=''></h2>
- [**UIViewController初始化**](#UIViewController初始化)
	- [自定义指定构造器方法](#自定义指定构造器方法)
	- [便利构造器方法](#便利构造器方法)
- [**UICollectionView**](#UICollectionView)
	- [**UICollectionViewDelegateFlowLayout代理**](#UICollectionViewDelegateFlowLayout代理)
		- [item的尺寸](#item的尺寸)
		- [每个分区的内边距](#每个分区的内边距)
		- [最小行间距](#最小行间距)
		- 	[最小item间距](#最小item间距)
		- 	[每个分区区头尺寸](#每个分区区头尺寸)
	- [**UICollectionViewDataSource**](#UICollectionViewDataSource)
		- [	每个分区含有的 item 个数](#每个分区含有的item个数)
		- 	 [返回 cell](#返回cell)
		- 	[分区数](#分区数)
		- [	返回自定义HeadView或者FootView](#返回自定义HeadView或者FootView)
- [**UILabel**](#UILabel)
	- [宽度自适应](#宽度自适应)
- [**‌弹窗播放视频**](#弹窗播放视频)



<br/>

***
<br/>
<br/>

># <h1 id='UIViewController初始化'>UIViewController初始化</h1>


<br/>
<br/>

> <h2 id='自定义指定构造器方法'>自定义指定构造器方法</h2>



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
<br/>

> <h2 id='便利构造器方法'>便利构造器方法</h2>

```
var studentName : String = ""
var age : Int  = 20

convenience init(studentName: String, age: Int) {
    self.init()

    self.studentName = studentName
    self.age = age
}
```








<br/>

***
<br/>
<br/>

># <h1 id='UICollectionView'>UICollectionView</h1>

<br/>
<br/>

> <h2 id='UICollectionViewDelegateFlowLayout代理'>UICollectionViewDelegateFlowLayout代理</h2>

<br/>

> <h3 id='item的尺寸'>item的尺寸</h3>


```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
    return CGSize(width: ScreenWidth / 4.0 - 50 / 4.0, height: ScreenWidth / 4.0 - 50 / 4.0)

}
```



<br/>
<br/>

> <h3 id='每个分区的内边距'>每个分区的内边距</h3>

```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
    return UIEdgeInsetsMake(10, 10, 10, 10);

}
```


<br/>
<br/>

> <h3 id='最小行间距'>最小行间距</h3>



```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
    return 10;
}
```


<br/>
<br/>

> <h3 id='最小item间距'>最小item间距</h3>



```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
    return 10;
}
```


<br/>
<br/>

> <h3 id='每个分区区头尺寸'>每个分区区头尺寸</h3>


```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, referenceSizeForHeaderInSection section: Int) -> CGSize {
    return CGSize (width: ScreenWidth, height: 40)
}
```


<br/>
<br/>

> <h2 id='UICollectionViewDataSource'>UICollectionViewDataSource</h2>
 
<br/>
<br/>

> <h3 id='每个分区含有的item个数'>每个分区含有的 item 个数</h3>

```
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return 200;
}
```

<br/>
<br/>

> <h3 id='返回cell'>返回 cell</h3>

```
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: CELL_ID, for: indexPath) as! TSMHomeCollectionCell;
    return cell;
}
```

<br/>
<br/>

> <h3 id='分区数'>分区数</h3>

```
func numberOfSections(in collectionView: UICollectionView) -> Int {
        return self.sectionViewModels.count
}
```


<br/>
<br/>

> <h3 id='返回自定义HeadView或者FootView'>返回自定义HeadView或者FootView</h3>

```
func collectionView(collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, atIndexPath indexPath: NSIndexPath) -> UICollectionReusableView{
    var v = Home_HeadView()
    if kind == UICollectionElementKindSectionHeader{

    v = colltionView!.dequeueReusableSupplementaryViewOfKind(kind, withReuseIdentifier: "headView", forIndexPath: indexPath) as! Home_HeadView
    }
    return v
}
```



<br/><br/><br/>

***
<br/>

> <h1 id="UILabel">UILabel</h1>


***
<br/>
> <h2 id="宽度自适应">宽度自适应</h2>

我的需求是：

* `UILabel` 最多显示 2 行
* 超出部分末尾显示省略号（`...`）
* 宽度根据内容自动伸展

<br/>

在 Swift 里你可以这样做：

```swift
let tipLab = UILabel()
tipLab.text = tip
tipLab.font = UIFont.systemFont(ofSize: 12.fontFit)
tipLab.numberOfLines = 2
tipLab.lineBreakMode = .byTruncatingTail // 末尾省略号
tipLab.textAlignment = .center
tipLab.backgroundColor = .cyan

// 让宽度自适应
tipLab.setContentHuggingPriority(.required, for: .horizontal)
tipLab.setContentCompressionResistancePriority(.required, for: .horizontal)

// 约束
tipLab.snp.makeConstraints { make in
    make.top.equalTo(titleLab.snp.bottom).offset(12)
    make.height.greaterThanOrEqualTo(20) // 不固定死高度，让它能自适应两行高度
}
```

<br/>

**核心点**

1. **`numberOfLines = 2`** → 限制最多两行
2. **`lineBreakMode = .byTruncatingTail`** → 超出部分用省略号
3. **不要固定 `height.equalTo(20)`** → 因为固定死高度会强制单行高度，无法显示两行内容；可以改为

   ```swift
   make.height.lessThanOrEqualTo(tipLab.font.lineHeight * 2)
   ```

<br/>

   这样即使是两行，也能正常显示且不超出
4. **宽度自适应** → 用 `hugging` 和 `compressionResistance` 让 Label 根据内容撑开宽度

<br/>

如果你希望 **宽度最多不超过父视图的一部分**（比如 80%），可以加：

```swift
make.width.lessThanOrEqualToSuperview().multipliedBy(0.8)
```

这样长文字会在两行后被截断加省略号。


<br/><br/><br/>

***
<br/>

> <h1 id="弹窗播放视频">弹窗播放视频</h1>

<br/>

 **1️⃣ 视频播放弹出视图**

```swift
import UIKit
import AVKit
import SnapKit

class VideoPopupView: UIView {
    private var player: AVPlayer?
    private var playerLayer: AVPlayerLayer?

    private let containerView = UIView()
    private let closeButton = UIButton(type: .system)

    init(videoURL: URL) {
        super.init(frame: .zero)
        setupUI()
        setupVideoPlayer(url: videoURL)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupUI() {
        // 背景不透
        backgroundColor = UIColor.black

        // 容器
        containerView.backgroundColor = .black
        addSubview(containerView)
        containerView.snp.makeConstraints { make in
            make.center.equalToSuperview()
            make.width.equalToSuperview().multipliedBy(0.9)
            make.height.equalTo(containerView.snp.width).multipliedBy(9.0/16.0) // 16:9 视频比例
        }

        // 关闭按钮
        closeButton.setTitle("关闭", for: .normal)
        closeButton.setTitleColor(.white, for: .normal)
        closeButton.addTarget(self, action: #selector(closeTapped), for: .touchUpInside)
        addSubview(closeButton)
        closeButton.snp.makeConstraints { make in
            make.top.equalTo(containerView.snp.bottom).offset(10)
            make.centerX.equalToSuperview()
        }
    }

    private func setupVideoPlayer(url: URL) {
        player = AVPlayer(url: url)
        player?.automaticallyWaitsToMinimizeStalling = true // 性能优化

        playerLayer = AVPlayerLayer(player: player)
        playerLayer?.videoGravity = .resizeAspect
        playerLayer?.frame = containerView.bounds
        if let playerLayer = playerLayer {
            containerView.layer.addSublayer(playerLayer)
        }
    }

    func show(in parentView: UIView) {
        parentView.addSubview(self)
        self.snp.makeConstraints { make in
            make.edges.equalToSuperview()
        }
        player?.play()
    }

    @objc private func closeTapped() {
        player?.pause()
        removeFromSuperview()
    }

    override func layoutSubviews() {
        super.layoutSubviews()
        playerLayer?.frame = containerView.bounds
    }
}
```

<br/>

**2️⃣ 给 UIView 添加点击手势弹出视频弹窗**

```swift
class ViewController: UIViewController {
    private let videoTriggerView = UIView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white

        // 点击区域
        videoTriggerView.backgroundColor = .systemBlue
        view.addSubview(videoTriggerView)
        videoTriggerView.snp.makeConstraints { make in
            make.center.equalToSuperview()
            make.size.equalTo(CGSize(width: 200, height: 100))
        }

        // 添加点击手势
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(videoViewTapped))
        videoTriggerView.isUserInteractionEnabled = true
        videoTriggerView.addGestureRecognizer(tapGesture)
    }

    @objc private func videoViewTapped() {
        guard let url = URL(string: "https://example.com/video.mp4") else { return }
        let popup = VideoPopupView(videoURL: url)
        popup.show(in: view)
    }
}
```

<br/>

✅ **特点**

* 使用 **SnapKit** 布局，布局清晰。
* **背景不透明**（黑色遮罩层）。
* **性能优化**：`automaticallyWaitsToMinimizeStalling` 减少卡顿，播放器在弹出后才播放。
* **点击手势触发**，可以绑定到任何 `UIView`。










