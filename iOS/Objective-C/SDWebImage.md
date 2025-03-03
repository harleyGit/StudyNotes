> </h1>
- [**图片渲染过程**](#图片渲染过程)
- [**SDWebImage的UML图**](#SDWebImage的UML图)
- [**下载顺序枚举**](#下载顺序枚举)
- [**缓存文件清除机制**](#缓存文件清除机制)
- [**图片格式**](#图片格式)
- [**MD5获取文件名**](#MD5获取文件名)
- **资料**
	- [**SDWebImage 图片解压和压缩**](https://www.jianshu.com/p/dfa47380fc05)
	- [**SDWebImage 源码解析**](http://cloverkim.com/SDWebImage-source-code-analysis-1.html)
	- [**SDWebImage相关面试题**](http://cloverkim.com/SDWebImage-interview-question.html)
	- [**SDWebImage知识点精讲**](http://cloverkim.com/SDWebImage-interview-question.html)
	- **[从零开始打造一个iOS图片加载框架](https://juejin.im/post/6844903807667666951)**
	- [**iOS 中图片的解压缩**](http://www.cocoachina.com/ios/20170227/18784.html)
	- [**SDWebImage 4.x版本源码分析**](https://www.jianshu.com/p/ae5c107d2b76)
	- [**从sd_setImageWithURL:方法谈SDWebImage （二)**](https://www.jianshu.com/p/2d96280aa841)



<br/><br/><br/>

***
<br/>
> <h1 id='图片渲染过程'>图片渲染过程</h1>

- 图片文件只有在确认要显示时,CPU才会对齐进行解压缩.因为解压是非常消耗性能的事情.解压过的图片就不会重复解压,会缓存起来；
- 图片渲染到屏幕的过程:
    -   CPU 读取文件->计算Frame->图片解码->解码后纹理图片位图数据通过数据总线交给GPU；
    -   GPU获取图片Frame->顶点变换计算->光栅化->根据纹理坐标获取每个像素点的颜色值(如果出现透明值需要将每个像素点的`颜色*透明度值`)->渲染到帧缓存区->渲染到屏幕;

- 三方库`YYImage`和 `SDWebImage`中，使用的下面的`CGBitmapContextCreate`函数对图片进行强制解压缩：

```diff

+  //data ：如果不为 NULL ，那么它应该指向一块大小至少为 bytesPerRow *  height 字节的内存；如果 为 NULL ，那么系统就会为我们自动分配和释放所需的内存，所以一般指定 NULL 即可；
- //width 和height ：位图的宽度和高度，分别赋值为图片的像素宽度和像素高度即可；
! //bitsPerComponent ：像素的每个颜色分量使用的 bit 数，在 RGB 颜色空间下指定 8 即可；
bytesPerRow：位图的每一行使用的字节数，大小至少为 width * bytes per pixel 字节。当我们指定 0/NULL 时，系统不仅会为我们自动计算，而且还会进行 cache line alignment 的优化
# //space ：就是我们前面提到的颜色空间，一般使用 RGB 即可；
+ //bitmapInfo：位图的布局信息.kCGImageAlphaPremultipliedFirst

CG_EXTERN CGContextRef __nullable CGBitmapContextCreate(void * __nullable data,
    size_t width, size_t height, size_t bitsPerComponent, size_t bytesPerRow,
    CGColorSpaceRef cg_nullable space, uint32_t bitmapInfo)
    CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

```


<br/><br/><br/>

***
<br/>
> <h1 id='SDWebImage的UML图'>SDWebImage的UML图</h1>

**`SDWebImage UML类图`**

&emsp;  若是想看下列的清晰的结构图，最新的版本有的类是没有的，所以需要 `pod 'SDWebImage',  '~>3.8.2' `版本。

**SDWebImage 结构图:**
![ios_oc1_108_31.webp](./../../Pictures/ios_oc1_108_31.webp)

<br/>

SDWebImage的相关类分为以下三种：

- **各种分类：**
	- UIButton+WebCache：为UIButton类添加加载图片的方法。
	- MKAnnotationView+WebCache：为MKAnnotationView类添加各种加载图片的方法。
	- UIImageView+WebCache：为UIImageView类添加加载图片的方法。
	- UIImageView+HighlightedWebCache：为UIImageView类添加高亮状态下加载图片的方法。

<br/>

- **工具类：**
	- NSData+ImageContentType：根据图片数据获取图片的类型，比如GIF、PNG等。
	- UIImage+MultiFormat：根据UIImage的data生成指定格式的UIImage。
	- UIImage+GIF：判断一张图是否为GIF。
	- SDImageResizingTransformer：根据屏幕的分辨倍数成倍放大或者缩小图片的大小。
	- SDImageCacheConfig：图片缓存策略记录。比如是否解压缩、是否允许iCloud、是否允许内存缓存、缓存时间等。
	- SDWebImageCodersManager：编码解码管理器，处理多个图片编码解码任务，编码器是一个优先队列，这意味着后面添加的编码器将具有最高优先级。

<br/>

- **核心类：**

	- UIView+WebCache：所有的UIView及其子类都会调用这个分类的方法来完成图片加载的处理，同时通过UIView+WebCacheOperation分类来管理请求的取消和记录工作。
	- SDImageCache：负责SDWebImage的整个缓存工作，是一个单例对象。缓存路径处理、缓存名字处理、管理内存缓存和磁盘缓存的创建和删除、根据指定key获取图片、存入图片的处理、根据缓存的创建和修改日期来删除缓存等。
	- SDWebImageManager：拥有一个SDImageCache和SDWebImageDownloader属性，分别用于图片的缓存和加载处理。为UIView及其子类提供了加载图片的统一接口。
	- SDWebImageDownloader：图片下载中心，管理下载队列。
	- SDWebImageDownloaderOperation：用于下载图片，管理NSURLRequest对象请求头的封装、缓存、cookie的设置、加载选项的处理等。


<br/>

**UML符号意义图:**
![ios_oc1_108_32.webp](./../../Pictures/ios_oc1_108_32.webp)

<br/>

**关键类的作用**

```
SDWebImageManger:    负责下载和缓存的管理；
SDImgeCache                  负责内存和磁盘缓存的类；
SDWebImageDownloader      负责下载类的管理；
SDWebImageDownloaderOperation        负责单个图片下载的操作由它来做
```

<br/>

**`SDWebImage 调用方法序列图`**
![ios_oc1_108_33.webp](./../../Pictures/ios_oc1_108_33.webp)

![z27](./../../Pictures/z27.jpg)

<br/><br/>

&emsp;  SDWebImage 加载图片的格式是webp的图片格式(这种格式是谷歌提出的，但是这个webp图片格式因为防火墙的原因在国内是不能使用的, 这种格式可以节省服务器1/3的资源。)。

&emsp;  程序运行时通过堆栈的方式进行加载的，可以在 `application: didFinishLaunchingWithOptions: ` 打断点看到。在Xcode左边的线程Thread1 运行可以看到，zai `start` 中是系统内核加载一些静态类库接着是执行`main`方法，然后才是执行 `application: didFinishLaunchingWithOptions: `  方法。


<br/><br/><br/>

***
<br/>
> <h1 id='下载顺序枚举'>下载顺序枚举</h1>

```
typedef NS_ENUM(NSInteger, SDWebImageDownloaderExecutionOrder) {
    /**
     * Default value. All download operations will execute in queue style (first-in-first-out).
     按照队列先进先出的顺序下载
     */
    SDWebImageDownloaderFIFOExecutionOrder,

    /**
     * All download operations will execute in stack style (last-in-first-out).
     按照栈后进先出进行下载(在UITableView中显示是从底部到顶部逐个显示图片的)
     */
    SDWebImageDownloaderLIFOExecutionOrder
};

```


<br/><br/><br/>

***
<br/>
> <h1 id='缓存文件清除机制'>缓存文件清除机制</h1>

- 先清除已超过最大缓存时间(大于一周的图片)的缓存文件；
- 保存文件的大小；
- 判断设置的上限，进行第二轮的清除；
- 默认的清除方式，先删除老的，达到期望缓存的大小，最大缓存的一半；

<br/><br/><br/>

***
<br/>
> <h1 id='图片格式'>图片格式</h1>

-  png: 无损压缩，解压缩性能高；
-  jpg：压缩比很高；
-  gif：动图；
-  webp：压缩比比我们的jpg小 1/3;
-  TIF:  从gig动图中扒了一张图作为tif图；

```
//图片枚举
typedef NS_ENUM(NSInteger, SDImageFormat) {
    SDImageFormatUndefined = -1,
    SDImageFormatJPEG = 0,
    SDImageFormatPNG,
    SDImageFormatGIF,
    SDImageFormatTIFF,
    SDImageFormatWebP,
    SDImageFormatHEIC
};


# NSData+ImageContentType.m 文件

//通过获取图片的16进制的前一位进行判断格式
+ (SDImageFormat)sd_imageFormatForImageData:(nullable NSData *)data {
    if (!data) {
        return SDImageFormatUndefined;
    }
    
    // File signatures table: http://www.garykessler.net/library/file_sigs.html
    uint8_t c;
    [data getBytes:&c length:1];
    switch (c) {
        case 0xFF:
            return SDImageFormatJPEG;
        case 0x89:
            return SDImageFormatPNG;
        case 0x47:
            return SDImageFormatGIF;
        case 0x49:
        case 0x4D:
            return SDImageFormatTIFF;
        case 0x52: {
            if (data.length >= 12) {
                //RIFF....WEBP
                NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(0, 12)] encoding:NSASCIIStringEncoding];
                if ([testString hasPrefix:@"RIFF"] && [testString hasSuffix:@"WEBP"]) {
                    return SDImageFormatWebP;
                }
            }
            break;
        }
        case 0x00: {
            if (data.length >= 12) {
                //....ftypheic ....ftypheix ....ftyphevc ....ftyphevx
                NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(4, 8)] encoding:NSASCIIStringEncoding];
                if ([testString isEqualToString:@"ftypheic"]
                    || [testString isEqualToString:@"ftypheix"]
                    || [testString isEqualToString:@"ftyphevc"]
                    || [testString isEqualToString:@"ftyphevx"]) {
                    return SDImageFormatHEIC;
                }
            }
            break;
        }
    }
    return SDImageFormatUndefined;
}

```
<br/>

**打开图片的16进制data数据:**
![ios_oc1_108_34.webp](./../../Pictures/ios_oc1_108_34.webp)




<br/><br/><br/>

***
<br/>
> <h1 id='MD5获取文件名'>MD5获取文件名</h1>

```

#SDDiskCache.m 文件

#pragma mark - Hash

#define SD_MAX_FILE_EXTENSION_LENGTH (NAME_MAX - CC_MD5_DIGEST_LENGTH * 2 - 1)

//保存缓存文件的文件名：url(MD5)+.扩展名
static inline NSString * _Nonnull SDDiskCacheFileNameForKey(NSString * _Nullable key) {
    const char *str = key.UTF8String;
    if (str == NULL) {
        str = "";
    }
    unsigned char r[CC_MD5_DIGEST_LENGTH];
    CC_MD5(str, (CC_LONG)strlen(str), r);
    NSURL *keyURL = [NSURL URLWithString:key];
    NSString *ext = keyURL ? keyURL.pathExtension : key.pathExtension;
    // File system has file name length limit, we need to check if ext is too long, we don't add it to the filename
    if (ext.length > SD_MAX_FILE_EXTENSION_LENGTH) {
        ext = nil;
    }
    NSString *filename = [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%@",
                          r[0], r[1], r[2], r[3], r[4], r[5], r[6], r[7], r[8], r[9], r[10],
                          r[11], r[12], r[13], r[14], r[15], ext.length == 0 ? @"" : [NSString stringWithFormat:@".%@", ext]];
    return filename;
}

```


&emsp; 强制解压缩的原理就是对图片进行重新绘制，得到一张新的解压缩后的位图。其中，用到的最核心的函数是 CGBitmapContextCreate 。
