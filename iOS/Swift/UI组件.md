

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










