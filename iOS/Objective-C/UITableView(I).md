

- **属性**
	- ContentSize的计算
	- 消除选中cell的效果
- **UITableViewCell**
	- 	reloadData
- **样式**
	- 	UITableViewStylePlain
	- 	让段头不停留（取消粘性效果）
	- [MultipleCells](https://github.com/la0fu/MultipleCells)
	- [支持不同类型的Cell](https://www.jianshu.com/p/19fdb2af6228)
	- [UITableView优化之异步绘制](https://blog.csdn.net/mo_xiao_mo/article/details/52622172)
	- [预渲染加速图片显示](https://www.keakon.net/2011/07/26/利用预渲染加速iOS设备的图像显示)
	- [UITableView详解](https://blog.csdn.net/qqqqzxg/article/details/52486282)



<br/>

***
<br/>


>#  属性

- **ContentSize的计算**

&emsp;  UITableView是继承于UIScrollView的，一个UIscrollView能滑动，需要设置contentSize，在iOS11之前，会调用tableView每一个cell的heightForRowAtIndexPath来算出整个高度，从而相加得出contentSize来，这一个步骤非常耗性能！

&emsp; 在 iOS11，默认打开了estimatedRowHeight估算高度功能，当tableView创建完成后，contentSize为estimatedRowHeight（默认值为44）*cell的数量，不需要遍历每一个cell的heightForRowAtIndexPath来计算了。

注意：使用MJRefresh在iOS11下要让estimatedRowHeight=0，因为MJRefresh底部的上拉刷新是根据contentSize来计算的，当数据更新的时候，得出来的contentSize只是预估的。否则会发生UITableView偏移的现象。

<br/>

- **`消除选中cell的效果`**

在数据源协议代理方法：`  - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;                                    `中对cell进行初始化时，设置这个属性：

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"commentCell"];
        if (!cell) {
                cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleValue1 reuseIdentifier:@"commentCell"];
                cell.selectionStyle = UITableViewCellSelectionStyleNone;//设置这个属性
          }
      return cell;
}
```


<br/>

- **`AutoLayout 计算Cell高度`**

&emsp;  在自动布局中提供了 `-systemLayoutSizeFittingSize:` 的 API，在 contentView 中设置约束后，就能计算出准确的值；缺点是计算速度肯定没有手算快，而且这是个实例方法，需要维护专门为计算高度而生的 template layout cell，它还要求使用者对约束设置的比较熟练，要保证 contentView 内部上下左右所有方向都有约束支撑，设置不合理的话计算的高度就成了0。




<br/>

***
<br/>

># UITableViewCell
 
**`UITableViewCell  的 accessoryType 属性`**

```
cell.accessoryType = UITableViewCellAccessoryNone;//cell没有任何的样式
 
cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;//cell的右边有一个小箭头，距离右边有十几像素；
 
cell.accessoryType = UITableViewCellAccessoryDetailDisclosureButton;//cell右边有一个蓝色的圆形button；
 
cell.accessoryType = UITableViewCellAccessoryCheckmark;//cell右边的形状是对号；
```

<br/>

- **`reloadData`**

&emsp;  刷新UITableView的展示数据，这个数据的更新范围包含UITableViewCell、UITableViewHeaderView。




<br/>

***
<br/>


># 样式

UITableViewStyleGrouped 和UITableViewStylePlain 

<br/>

- **`UITableViewStylePlain `** 

```
a. 有多段时(区头,区尾), 段头停留(自带粘性效果)

b. 没有中间的间距和头部间距（要想有的重写UITableViewCell 、UITableViewHeaderFooterView里面的setFrame方法） 
```

<br/>

- **让段头不停留（取消粘性效果）**

```

- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
      CGFloat sectionHeaderHeight = 30;
      if (scrollView.contentOffset.y<=sectionHeaderHeight&&scrollView.contentOffset.y>=0) {
           scrollView.contentInset = UIEdgeInsetsMake(-scrollView.contentOffset.y, 0, 0, 0);
         } else if (scrollView.contentOffset.y>=sectionHeaderHeight) {
           scrollView.contentInset = UIEdgeInsetsMake(-sectionHeaderHeight, 0, 0, 0);
    }
}
```


<br/>

- **`UITableViewStyleGroup`**

![ios_oc1_109_22](./../Pictures/ios_oc1_109_22.png)


在tableview的代理方法：返回组的头／尾视图中设置具体高度时，开头结尾总是默认有一段距离，并且如果设置她们中的某个距离为0，则无效。

**处理方法:**

①设置标头的高度为特小值 （不能为零 为零的话苹果会取默认值就无法消除头部间距了）

```
UIView *view = [[UIView alloc]initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, 0.001)];
view.backgroundColor = [UIColor redColor];
self.tableView.tableHeaderView = view;
```

② UIEdgeInsetsMake 进行处理

```
 self.tableView.contentInset = UIEdgeInsetsMake(-44, 0, 0, 0);
```

③重写UITableViewHeaderFooterView的 frame

```
 -(void)setFrame:(CGRect)frame{
       frame.size.height+=10;
       [super setFrame:frame];
}
```


<br/>
<br/>

