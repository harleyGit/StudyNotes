- [**界面流畅**](#界面流畅)
- **图片加载**
- **图片压缩**



<br/>

***
<br/>

> <h1 id = "界面流畅">[界面流畅](https://xilankong.github.io/ios性能优化/2017/10/29/iOS如何保持界面流畅.html)</hr>


<br/>

- 绘制渲染流程

UIView从Draw到Render的过程有如下几步：

&emsp; 每一个UIView都有一个layer，每一个layer都有个content，这个content指向的是一块缓存，叫做backing store。

&emsp; UIView的绘制和渲染是两个过程，当UIView被绘制时，CPU执行drawRect，通过context将数据写入backing store。

&emsp; 当backing store写完后，通过render server交给GPU去渲染，将backing store中的bitmap数据显示在屏幕上。

&emsp; 下图就是从CPU到GPU的过程：

![<br/>](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/optimization0.jpg)

<br/>

- 离屏渲染

<br/>

![](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/optimization1.jpg)

iOS中图形绘制框架的大致结构


<br/>

&emsp; UIKit是iOS中用来管理用户图形交互的框架，但是UIKit本身构建在CoreAnimation框架之上，CoreAnimation分成了两部分OpenGL ES和Core Graphics，OpenGL ES是直接调用底层的GPU进行渲染；Core Graphics是一个基于CPU的绘制引擎；

&emsp; 我们平时所说的硬件加速其实都是指OpenGL，Core Animation / UIKit 基于GPU之上对计算机图形合成以及绘制的实现，由于CPU是渲染能力要低于GPU，所以当采用CPU绘制时动画时会有明显的卡顿。

&emsp; 有些绘制会产生离屏渲染，额外增加GPU以及CPU的绘制渲染。

&emsp; GPU屏幕渲染有以下两种方式：

- On-Screen Rendering 当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行。
- Off-Screen Rendering 离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。


离屏渲染的代价主要包括两方面内容：

- 创建新的缓冲区
- 上下文的切换，离屏渲染的整个过程，需要多次切换上下文环境：先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上有需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的。



为什么需要离屏渲染？

&emsp; 目的在于当使用圆角，阴影，遮罩的时候，图层属性的混合体被指定为在未预合成之前不能直接在屏幕中绘制，即当主屏的还没有绘制好的时候，所以就需要屏幕外渲染，最后当主屏已经绘制完成的时候，再将离屏的内容转移至主屏上。


离屏渲染的触发方式：

- shouldRasterize（光栅化）
- masks（遮罩）
- shadows（阴影）
- edge antialiasing（抗锯齿）
- group opacity（不透明）


CPU渲染:

&emsp; 以上所说的都是离屏渲染发生在OpenGL SE也就是GPU中，但是CPU也会发生特殊的渲染，我们的CPU渲染，也就是我们使用Core Graphics的时候，但是要注意的一点的是只有在我们重写了drawRect方法，并且使用任何Core Graphics的技术进行了绘制操作，就涉及到了CPU渲染。整个渲染过程由CPU在App内 同步地 完成，渲染得到的bitmap最后再交由GPU用于显示。理论上CPU渲染应该不算是标准意义上的离屏渲染，但是由于CPU自身做渲染的性能也不好，所以这种方式也是需要尽量避免的。

&emsp; 所以对于当屏渲染，离屏渲染和CPU渲染的来说，当屏渲染永远是最好的选择，但是考虑到GPU的浮点运算能力要比CPU强，但是由于离屏渲染需要重新开辟缓冲区以及屏幕的上下文切换，所以在离屏渲染和CPU渲染的性能比较上需要根据实际情况作出选择。








<br/>

***
<br/>

># 图片加载
图片的显示分为三步：加载、解码、渲染。
通常，我们操作的只有加载，解码和渲染是由UIKit进行。



<br/>

***
<br/>


># 图片压缩
&emsp;   对于大量上传的图片，我们需要对其进行优化，对一些大图片进行压缩后再进行上传

[不失真的图片压缩](https://www.jianshu.com/p/76f7eb00ef13)

[图片加载和处理](https://www.jianshu.com/p/7d8a82115060)



<br/>

***
<br/>




># 内存图片与显示的图片Size不符
&emsp;  假如内存里有一张400x400的图片，要放到100x100的`UIImageView`里，如果不做任何处理，直接丢进去，问题就大了。
&emsp;  这意味着，GPU需要对大图进行缩放到小的区域显示，需要做像素点的sampling，这种smapling的代价很高，又需要兼顾pixel alignment。计算量会飙升。
OpenGL ES是直接调用底层的GPU进行渲染；Core Graphics是一个基于CPU的绘制引擎；

**`缩放图片Code`**

```
//重新绘制图片
//按照imageWidth, imageHeight指定宽高开始绘制图片
UIGraphicsBeginImageContext(CGSizeMake(imageWidth, imageHeight));
//把image原图绘制成指定宽高
[image drawInRect:CGRectMake(0,0,imageWidth,  imageHeight)];
//从绘制中获取指定宽高的图片
UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
//结束绘制
UIGraphicsEndImageContext();
```
&emsp;  RunLoop开始，RunLoop是一个60fps的回调，也就是说每16.7ms绘制一次屏幕，也就是我们需要在这个时间内完成view的缓冲区创建，view内容的绘制这些是CPU的工作；然后把缓冲区交给GPU渲染，这里包括了多个View的拼接(Compositing),纹理的渲染(Texture)等等，最后Display到屏幕上。但是如果你在16.7ms内做的事情太多，导致CPU，GPU无法在指定时间内完成指定的工作，那么就会出现卡顿现象，也就是丢帧。



<br/>

***
<br/>

># 上传图片优化





<br/>

***
<br/>

># 加载大量图片优化

```
#import "HGWeiBoController.h"

@interface HGWeiBoController ()<UITableViewDelegate, UITableViewDataSource>
@property(nonatomic, retain) NSArray *wbModels;
@property(nonatomic, retain) NSOperationQueue *queue;

@end
@implementation HGWeiBoController

- (NSOperationQueue *)queue {
    if (!_queue) {
        _queue = [[NSOperationQueue alloc] init];
    }
    return _queue;
}

- (NSArray *)wbModels {
    if (!_wbModels) {
        NSURL *url = [[NSBundle mainBundle] URLForResource:@"apps.plist" withExtension:nil];
        NSArray *array = [NSArray arrayWithContentsOfURL:url];
        //2.遍历数组
        NSMutableArray *arrayM = [NSMutableArray array];
        [array enumerateObjectsUsingBlock: ^(id obj, NSUInteger idx, BOOL *stop) {
            //数组中存放的是字典, 转换为app对象后再添加到数组
            [arrayM addObject:[MicroblogModel appWithDict:obj]];
        }];
        _wbModels = [arrayM copy];
    }
    return _wbModels;
}

#pragma mark -- UITableViewDataSource
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *cellID = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellID];
    if (!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellID];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
    }
    
    MicroblogModel *mm = self.wbModels[indexPath.row];
    cell.textLabel.text = mm.name;  //设置文字
    
    //设置图像: 模型中图像为nil时用默认图像,并下载图像. 否则用模型中的内存缓存图像.
    if (!mm.image) {
        cell.imageView.image = [UIImage imageNamed:@"user_default"];
        
        [self downloadImg:indexPath];
    }
    else {
        //直接用模型中的内存缓存
        cell.imageView.image = mm.image;
    }

    return cell;
}


//始终记住, 通过模型来修改显示. 而不要试图直接修改显示
- (void)downloadImg:(NSIndexPath *)indexPath {
    MicroblogModel *mm  = self.wbModels[indexPath.row]; //取得改行对应的模型
    
    [self.queue addOperationWithBlock: ^{
        NSData *imgData = [NSData dataWithContentsOfURL:[NSURL URLWithString:mm.icon]]; //得到图像数据
        UIImage *image = [UIImage imageWithData:imgData];
        
        //在主线程中更新UI
        [[NSOperationQueue mainQueue] addOperationWithBlock: ^{
            //通过修改模型, 来修改数据
            mm.image = image;
            //刷新指定表格行
            [self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
        }];
    }];
}


- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.wbModels.count;
}

@end

```
