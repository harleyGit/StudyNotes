<br/>

***
<br/>

># 图片加载
图片的显示分为三步：加载、解码、渲染。
通常，我们操作的只有加载，解码和渲染是由UIKit进行。

![z21](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z21.jpg)




<br/>

***
<br/>


># 图片压缩
&emsp;   对于大量上传的图片，我们需要对其进行优化，对一些大图片进行压缩后再进行上传
[](https://www.jianshu.com/p/76f7eb00ef13)

[iOS性能优化——图片加载和处理](https://www.jianshu.com/p/7d8a82115060)



<br/>

***
<br/>




># 内存图片与显示的图片Size不符
&emsp;  假如内存里有一张400x400的图片，要放到100x100的`UIImageView`里，如果不做任何处理，直接丢进去，问题就大了。
&emsp;  这意味着，GPU需要对大图进行缩放到小的区域显示，需要做像素点的sampling，这种smapling的代价很高，又需要兼顾pixel alignment。计算量会飙升。
OpenGL ES是直接调用底层的GPU进行渲染；Core Graphics是一个基于CPU的绘制引擎；

# `缩放图片Code`

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
