> <h1 id=""></h1>
- [left、right和leading、trailing区别](#left、right和leading、trailing区别) 
- [makeConstraints和remakeConstraints区别](#makeConstraints和remakeConstraints区别)







<br/>

***
<br/><br/><br/>

> <h1 id=""></h1>

<br/><br/><br/>

> <h2 id="left、right和leading、trailing区别">left、right和leading、trailing区别</h2>


在 **SnapKit** 中，`left` 和 `leading` 是用来指定视图的水平方向布局属性，但它们的使用有所区别，**特别是在处理从左到右（LTR）和从右到左（RTL）语言环境时**。这个在国内倒无所谓，但是对于一款供全世界使用的app，特别是中东阿拉伯地区使用者来说很重要了。他们是**从右到左的**。

<br/>

 - **1.`left` 属性**
	- **定义**: 指视图的左边缘（在标准的从左到右布局中）。
	- **使用场景**: 在不考虑语言方向的布局中，可以直接使用 `left` 来表示左侧对齐。
	- **局限性**: 在支持从右到左（RTL）的语言时，`left` 仍然指代视图的“左侧”，这可能并不符合 RTL 布局的需求。

```swift
view.snp.makeConstraints { make in
    make.left.equalToSuperview()
}
```

<br/>

 - **2.`leading` 属性**
	- **定义**: 表示视图的“前导边缘”，它会根据语言环境自动调整：
	  - 在从左到右（LTR）语言中，`leading` 等同于 `left`。
	  - 在从右到左（RTL）语言中，`leading` 等同于 `right`。
	- **使用场景**: 如果希望支持 RTL 和 LTR 的语言方向，可以使用 `leading`，它会根据当前语言环境自动调整视图的对齐方向。

```swift
view.snp.makeConstraints { make in
    make.leading.equalToSuperview()
}
```

<br/>

 - **3.总结**
	- **`left`**: 始终指视图的左边缘，无论语言方向。
	- **`leading`**: 动态调整，根据语言方向，可以指左边缘或右边缘。

如果你正在开发一个需要支持多语言环境的应用，建议优先使用 `leading` 和 `trailing` 这种更灵活的属性，确保布局在 RTL 和 LTR 环境中都能正确显示。


<br/><br/>

而且还有一点很重要的是，使用left在中东国家地区的时候，若是遵从**RTL**布局很可能造成app的崩溃。



<br/><br/><br/>

> <h2 id="makeConstraints和remakeConstraints区别">makeConstraints和remakeConstraints区别</h2>

时隔多年再一次使用Snapkit时我设置约束时，犯了一个很大的错误。就是我将约束放在了UIView的系统方法`-(void)layouSubViews`方法里了，并且布局使用的还是`makeConstraints`方法，这样可能会导致这个布局方法在`-(void)layouSubViews`多次调用，可能导致程序崩溃。

若是避免你可以用`remakeConstraints`方法，因为这个方法会把之前设置的约束都给清除掉，然后重新设置。但是这会又性能的损耗，所以需要单独一个方法对其进行布局了。

