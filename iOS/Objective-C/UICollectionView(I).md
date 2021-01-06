># 协议方法
```
#pragma mark --  UICollectionViewDatasoure
//显示几个section
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
{
    return 1;
}

//每个section中显示多个item
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return 12;
}

//配置单元格的方法
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    UICollectionViewCell* cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"UICollectionViewCell" forIndexPath:indexPath];
    cell.title = [NSString stringWithFormat:@"%td - %td",indexPath.section,indexPath.item];
    cell.editing = YES;
    return cell;
}


#pragma mark --  UICollectionViewDelegate
//每个单元格的大小size
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
    return CGSizeMake(60, 50);
}


#pragma mark -- UICollectionViewFlowLayoutDelegate
//每个item之间的行间距
- (CGFloat)collectionView:(UICollectionView *)collectionView 
layout:(UICollectionViewLayout*)collectionViewLayout 
minimumInteritemSpacingForSectionAtIndex:(NSInteger)section
{
   return 10;
}

//每个item之间的列间距
- (CGFloat)collectionView:(UICollectionView *)collectionView 
layout:(UICollectionViewLayout*)collectionViewLayout
 minimumLineSpacingForSectionAtIndex:(NSInteger)section
{
   return 10;
}

//每个Item的大小size
- (CGSize)collectionView:(UICollectionView *)collectionView 
layout:(UICollectionViewLayout*)collectionViewLayout 
sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
    return CGSizeMake(80,60);
}


//Section边距设置:整体边距的优先级，始终高于内部边距的优先级
-(UIEdgeInsets)collectionView:(UICollectionView *)collectionView
 layout:(UICollectionViewLayout *)collectionViewLayout 
insetForSectionAtIndex:(NSInteger)section
{
    //分别为上、左、下、右
    return UIEdgeInsetsMake(10, 20, 10, 20);
}


//headerView的size设置
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section{

    return CGSizeMake(20, 40);

}
//footerView的size设置
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section{

     return CGSizeMake(20, 40);
}
```


#`Item 行间距`
![item 行间距：minimumInteritemSpacing](https://upload-images.jianshu.io/upload_images/2959789-388d4762888a8b0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;   `UICollectionViewFlowLayout`实例对象属性`minimumInteritemSpacing`可以对其进行设置，其`UICollectionViewFlowLayoutDelegate`也可以对其进行实现，但需要遵守`UICollectionViewFlowLayoutDelegate`协议 (推荐这个`minimumInteritemSpacing`,尽量少用协议)

<br/>

#`Item 列间距`
![Item 列间距:minimumLineSpacing](https://upload-images.jianshu.io/upload_images/2959789-93213e8d7b4ec9ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; `UICollectionViewFlowLayout`的属性`minimumLineSpacing `可以对其进行设置(推荐这个,尽量少用协议)


<br/>
#`Item size设置`
&emsp;  `UICollectionViewFlowLayout`的属性`itemSize`可以对其进行设置。

<br/>

#`调整每个Section中的内容边距`
![Section 的边距](https://upload-images.jianshu.io/upload_images/2959789-aa06b1756013710a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&emsp;  `UICollectionViewFlowLayout`的属性`sectionInset`可以对其进行设置，也可以用协议设置。













<br/>
***
<br/>





```
#define JScreen_Width  [UIScreen mainScreen].bounds.size.width

UICollectionViewFlowLayout *flowLayout = [[UICollectionViewFlowLayout alloc] init];
   
     
CGFloat   cellSpace   = 6.0;
//每一个行有多少个item
NSInteger cellNumber  = 3;
//item的宽度
CGFloat   cellWidth   = (JScreen_Width - 52) / cellNumber;
//item的高度
CGFloat   cellHeight  = cellWidth;
        
//item的size,这个也可以通过代理方法完成
flowLayout.itemSize = CGSizeMake(cellWidth, cellHeight);
//每个item的水平间距
flowLayout.minimumInteritemSpacing = cellSpace/2;
//每个item的竖直间距
flowLayout.minimumLineSpacing = cellSpace;
//第一行item距离顶部的距离，相当于UITableView的header的高度设置
flowLayout.headerReferenceSize = CGSizeMake(JScreen_Width, 0);
        

UICollectionView *selectView = [[UICollectionView alloc] initWithFrame:CGRectMake(20, 184, JScreen_Width -2*20, 200) collectionViewLayout:flowLayout];


#pragma mark -- UICollectionViewDelegateFlowLayout
//设置其边界(上，左，下，右）
- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout insetForSectionAtIndex:(NSInteger)section {
    if (self.chooseType == ChooseElementTypePicture) {
        return UIEdgeInsetsZero;
    }else{
        return UIEdgeInsetsMake(-20, 20, 0, 20);
    }
}

......
//还有其他代码就不贴出来了
```
效果图：
![效果图](https://upload-images.jianshu.io/upload_images/2959789-f297540429acaa9e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>
[UICollectionView的section设置背景颜色](https://www.jianshu.com/p/8c9c5a125991)
[UICollectionView 自定义section背景](https://www.jianshu.com/p/26791bad530c)
[]()
