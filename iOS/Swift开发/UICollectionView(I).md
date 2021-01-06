># UICollectionViewDelegateFlowLayout 代理方法
<br/>
-  item 的尺寸
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
    return CGSize(width: ScreenWidth / 4.0 - 50 / 4.0, height: ScreenWidth / 4.0 - 50 / 4.0)

}
```

<br/>
- 每个分区的内边距
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
    return UIEdgeInsetsMake(10, 10, 10, 10);

}
```


<br/>
- 最小行间距
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
    return 10;
}
```


<br/>
-  最小 item 间距
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
    return 10;
}
```


<br/>
-  每个分区区头尺寸
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, referenceSizeForHeaderInSection section: Int) -> CGSize {
    return CGSize (width: ScreenWidth, height: 40)
}
```


<br/>
***
<br/>

># UICollectionViewDataSource 
<br/>
-  每个分区含有的 item 个数
```
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return 200;
}
```

<br/>
-  返回 cell
```
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: CELL_ID, for: indexPath) as! TSMHomeCollectionCell;
    return cell;
}
```

<br/>
- 分区数
```
func numberOfSections(in collectionView: UICollectionView) -> Int {
        return self.sectionViewModels.count
}
```


<br/>
- 返回自定义HeadView或者FootView
```
func collectionView(collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, atIndexPath indexPath: NSIndexPath) -> UICollectionReusableView{
    var v = Home_HeadView()
    if kind == UICollectionElementKindSectionHeader{

    v = colltionView!.dequeueReusableSupplementaryViewOfKind(kind, withReuseIdentifier: "headView", forIndexPath: indexPath) as! Home_HeadView
    }
    return v
}
```









