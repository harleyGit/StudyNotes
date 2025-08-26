

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
- [**UIButton**](#UIButton)
	- [文字和图片布局](#文字和图片布局)
		- [示意图详解布局](#示意图详解布局) 
		- [内部局部示意图](#内部局部示意图) 
		- [直接设置右边距](#直接设置右边距) 
		- [同时有文字+图片](#同时有文字+图片) 
		- [通用公式（动态计算）](#通用公式（动态计算）)



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


<br/><br/><br/>

***
<br/>

> <h1 id="UIButton">UIButton</h1>


***
<br/><br/><br/>
> <h2 id="文字和图片布局">文字和图片布局</h2>

**比如有如下的一段代码：**

```swift
button.imageEdgeInsets = UIEdgeInsetsMake(0, 130, 0, -130);
```

结果：图片 紧贴最右边。

- **原因：**
	- **UIButton** 的 `imageEdgeInsets` 其实是 相对 `contentRect` 的偏移量，并不是绝对位置。
	- 所以像你这种大数值`130`会直接把图片推到最右侧。

<br/><br/><br/> 

**UIButton默认排版**

- 一个 `UIButton` 内部有两个子视图：
	* `imageView` → 显示图片
	* `titleLabel` → 显示文字

它们默认是 **左右并排** 的，按钮会把它们整体居中显示。

<br/>

**`UIEdgeInsets` 的作用**

`UIEdgeInsets` 的四个参数（top, left, bottom, right），其实就是在 **原本的位置基础上再加偏移量**。

* `imageEdgeInsets` → 控制 `imageView` 相对于它原本位置的偏移
* `titleEdgeInsets` → 控制 `titleLabel` 相对于它原本位置的偏移

**注意 ⚠️：**
这些 inset **不会改变按钮本身的大小**，只是调整子控件（图片、文字）在按钮里的位置。

<br/><br/>

**假设按钮宽度是 `200pt`，文字宽度 `50pt`，图片宽度 `20pt`，间距 4pt。**

**默认效果**

```
[   image   ][  spacing  ][   title   ]   ← 中间居中
```

<br/>

**设置**

```objc
_deviceAddButton.imageEdgeInsets = UIEdgeInsetsMake(0, 20, 0, -20);
```

意思是：

* 图片的 **左边界 +20** → 图片往右移
* 图片的 **右边界 -20** → 效果等价于「整体往右推」

<br/><br/>

**为什么要两个 inset 都改？**

因为 inset 是相对于「按钮 contentRect」算的，不是单独水平偏移量。
要想「整体往左 / 往右」移动，必须同时调节 `left` 和 `right`，这样大小不变，只是平移。

<br/><br/>

 **我的场景（右边距 10pt）**

当你写：

```objc
_deviceAddButton.imageEdgeInsets = UIEdgeInsetsMake(0, 165-5, 0, -165+5);
```

本质就是在 **原来居中的基础上，图片整体往左挪 5pt**。
如果原来贴在右边，现在就会距离右边 10pt 左右。

<br/>

- **✅ 总结：**

	* `imageEdgeInsets` / `titleEdgeInsets` 是控制 **相对于按钮内容区域的偏移**。
	* `left/right` 是加减距离，组合起来决定图片 / 文字在按钮里的最终位置。
	* 如果你要「距离右边固定 10pt」，更推荐用 **按钮宽度 - 图片宽度 - 间距 - 文字宽度 - 10** 来算，这样不会受文字长短影响。


<br/><br/>
> <h3 id="示意图详解布局">示意图详解布局</h3>

**默认情况下（UIButton 内部布局）**

假设按钮宽 200pt，左边是图片（20pt），右边是文字（50pt），两者之间 4pt 间距：

```
|<----------- 200pt (button) ----------->|

       [img 20pt]--4pt--[title 50pt]       
```

👉 默认它们会整体居中。

<br/>

**如果设置**

```objc
button.imageEdgeInsets = UIEdgeInsetsMake(0, 10, 0, -10);
```

意思就是：

* **left = 10** → 图片相对原来往右移动 10pt
* **right = -10** → 维持大小不变，只是整体平移

结果：

```
|<----------- 200pt (button) ----------->|

             [    img 20pt ]--4pt--[title 50pt]       
```

（图片比原来更靠右）

<br/>

**如果你想让 图片距离右边正好 10pt**

你就要 **用按钮总宽度来计算**：

```
imageX = buttonWidth - rightMargin - imageWidth
```

例如按钮宽 200pt，图片宽 20pt，想要右边距 = 10pt：

```
imageX = 200 - 10 - 20 = 170pt
```

所以你的 inset 要设置成「让图片最终放到 x=170」的位置。

<br/>

✅ 推荐做法：
与其写死 `165+2-5` 这种 magic number，不如：

```objc
CGFloat buttonWidth = _deviceAddButton.bounds.size.width;
CGFloat imageWidth = _deviceAddButton.imageView.bounds.size.width;
CGFloat rightMargin = 10;

CGFloat move = buttonWidth - rightMargin - imageWidth - _deviceAddButton.imageView.frame.origin.x;
_deviceAddButton.imageEdgeInsets = UIEdgeInsetsMake(0, move, 0, -move);
```

这样就无论按钮宽度还是图片大小变不变，始终保持 **图片距右边 10pt**。


<br/><br/>
> <h3 id="内部局部示意图">内部局部示意图</h3>


**按钮宽度 = 200pt，高度忽略**

默认布局（左边图片 20pt，右边文字 50pt，中间 4pt）：

```
┌─────────────────────────────── UIButton (200pt) ───────────────────────────────┐
|                                                                               |
|   x=0             x=20        x=24                          x=74              |
|   ┌───────┐       ↑           ↑                             ↑                 |
|   |  IMG  | (20pt)|   4pt      |       ┌─────────────┐      | TITLE (50pt)    |
|   └───────┘       |           |       |   "Title"    |      |                 |
|                   |           |       └─────────────┘      |                 |
|                   |<-- imageRight -->|                                      |
|                                                                               |
└───────────────────────────────────────────────────────────────────────────────┘
```

👉 默认情况下：

* 图片起点 `x=0`，宽度 20pt
* 文本起点 `x=24`，宽度 50pt

<br/>

**如果你设置**

```objc
button.imageEdgeInsets = UIEdgeInsetsMake(0, 10, 0, -10);
```

解释：

* `left = 10` → 图片相对原始位置 **整体右移 10pt**
* `right = -10` → 保持对称（否则图片会被压缩）

结果：

```
┌─────────────────────────────── UIButton (200pt) ───────────────────────────────┐
|                                                                               |
|             x=10             x=30        x=34                 x=84            |
|             ┌───────┐        ↑           ↑                    ↑               |
|             |  IMG  |        |           |   ┌─────────────┐   | TITLE         |
|             └───────┘        |   4pt     |   |   "Title"    |   |             |
|                               |           └─────────────┘   |                 |
|                               |<-- imageRight (缩短) -->|                       |
|                                                                               |
└───────────────────────────────────────────────────────────────────────────────┘
```

📌 效果：图片往右 10pt，和标题的距离还是 4pt。


<br/><br/>

 **如果要让图片 距离按钮右边正好 10pt**

假设按钮宽 200pt，图片宽 20pt：

```
目标位置 = 按钮宽度 - 图片宽度 - 右边距 = 200 - 20 - 10 = 170pt
```

<br/>

那么你要计算一个 `move` 值：

```objc
CGFloat buttonWidth = button.bounds.size.width;
CGFloat imageWidth = button.imageView.bounds.size.width;
CGFloat rightMargin = 10;

// 当前图片起点位置
CGFloat currentX = button.imageView.frame.origin.x;

// 需要移动的量
CGFloat move = (buttonWidth - rightMargin - imageWidth) - currentX;

button.imageEdgeInsets = UIEdgeInsetsMake(0, move, 0, -move);
```

👉 这样写，无论按钮宽度、文字长度怎么变，**图片都能保持距右边 10pt**。



<br/><br/>
> <h3 id="直接设置右边距">直接设置右边距</h3>


如果你按钮宽度固定，可以这样：

```objc
button.contentHorizontalAlignment = UIControlContentHorizontalAlignmentRight;
button.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, 9);
```

这样图片就会 **靠右对齐，距离右边 9pt**。

<br/><br/>
> <h3 id="同时有文字+图片">同时有文字 + 图片</h3>


如果按钮还有文字，需要让文字靠左，图片靠右：

```objc
button.contentHorizontalAlignment = UIControlContentHorizontalAlignmentFill;

// 让图片距离右边 9pt
button.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, 9);

// 让文字距离左边 9pt
button.titleEdgeInsets = UIEdgeInsetsMake(0, 9, 0, 0);
```

<br/><br/>
> <h3 id="通用公式（动态计算）">通用公式（动态计算）</h3>

如果你想精确控制「图片距离右边 X」：

```objc
CGFloat spacing = 9;
CGFloat imageWidth = button.imageView.frame.size.width;
CGFloat buttonWidth = button.bounds.size.width;

CGFloat leftInset = buttonWidth - imageWidth - spacing;
button.imageEdgeInsets = UIEdgeInsetsMake(0, leftInset, 0, -leftInset);
```

---

⚠️ **推荐写法**：
如果只是要「右边留 9pt」，直接用 **方法一** 就够了。
如果按钮里同时有文字，建议用 **方法二**。

---

要不要我给你整理一个 **UIButton 分类**（比如 `setImageRight(padding:)`），以后你只要写一句就能把图片放右边并控制间距？













