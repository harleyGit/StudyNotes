># Message from debugger: Terminated due to memory issue
内存一直增长，导致无法释放，程序闪退。
&emsp;  使用`Instrument`的`Allocations`检查内存增长，对于一些临时产生的对象使用`@autorelease`进行释放。

**选择Allocations**
![选择 Allocations](https://upload-images.jianshu.io/upload_images/2959789-df5e0c87415c739d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Allocations 配置**
![配置](https://upload-images.jianshu.io/upload_images/2959789-c8dbe5c9370a597c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**选择真机，运行查找内存Boss**
![检测内存增长的 boss](https://upload-images.jianshu.io/upload_images/2959789-77c4ad5c57091780.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**`解决方法：清理内存`**

```
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    
    //bug: 加载太多图片crash
    [SDImageCache sharedImageCache].config.shouldCacheImagesInMemory = NO;
    [[SDImageCache sharedImageCache] clearDiskOnCompletion:nil];
    [[SDImageCache sharedImageCache] clearMemory];
}

```


