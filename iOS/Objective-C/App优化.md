> <h2 id=''></h2>
![](./../../Pictures/)
- [**App瘦身**](#App瘦身)
	- [官方AppThinning](#官方AppThinning)
	- [无用图片资源](#无用图片资源)
	- [代码瘦身](#代码瘦身)
		- [LinkMap 结合 Mach-O 找无用代码](#LinkMap结合Mach-O找无用代码)



<br/>

***
<br/>

> <h1 id='App瘦身'>App瘦身</h1>

&emsp; App Store 规定了安装包大小超过 150MB 的 App 不能使用 OTA（over-the-air）环境下载，也就是只能在 WiFi 环境下下载。所以，150MB 就成了 App 的生死线，一旦超越了这条线就很有可能会失去大量用户。

&emsp; App 包过大既损害用户体验，影响升级率，还会导致无法提交 App Store 的情况和非 WiFi 环境无法下载这样可能影响到 App 生死的问题。那么，怎样对包大小进行瘦身和控制包大小的不合理增长就成了重中之重。

<br/>
<br/>


> <h2 id='官方AppThinning'>官方AppThinning</h2>

&emsp; App Thinning 是由苹果公司推出的一项可以改善 App 下载进程的新技术，主要是为了解决用户下载 App 耗费过高流量的问题，同时还可以节省用户 iOS 设备的存储空间。

&emsp; App Thinning 会专门针对不同的设备来选择只适用于当前设备的内容以供下载。比如，iPhone 6 只会下载 2x 分辨率的图片资源，iPhone 6plus 则只会下载 3x 分辨率的图片资源。

&emsp; 在苹果公司使用 App Thinning 之前， 每个 App 包会包含多个芯片的指令集架构文件。以 Reveal.framework 为例，使用 du 命令查看到主文件在 Reveal.framework/Versions/A 目录下，大小有 21MB。

```
$ du -h Reveal.framework/*
  0B  Reveal.framework/Headers
  0B  Reveal.framework/Reveal
 16K  Reveal.framework/Versions/A/Headers
 21M  Reveal.framework/Versions/A
 21M  Reveal.framework/Versions
 ```
 
 <br/>
 
 我们可以再使用 file 命令，查看 Version 目录下的 Reveal 文件：
 
 ```
ming$ file Reveal.framework/Versions/A/Reveal 
Reveal.framework/Versions/A/Reveal: Mach-O universal binary with 5 architectures: [i386:current ar archive] [arm64]
Reveal.framework/Versions/A/Reveal (for architecture i386): current ar archive
Reveal.framework/Versions/A/Reveal (for architecture armv7):  current ar archive
Reveal.framework/Versions/A/Reveal (for architecture armv7s): current ar archive
Reveal.framework/Versions/A/Reveal (for architecture x86_64): current ar archive
Reveal.framework/Versions/A/Reveal (for architecture arm64):  current ar archive
```

可以看到， Reveal 文件里还有 5 个文件：
- x86_64 和 i386，是用于模拟器的芯片指令集架构文件；
- arm64、armv7、armv7s ，是真机的芯片指令集架构文件。


<br/>

&emsp; 使用 App Thinning 后，用户下载时就只会下载一个适合自己设备的芯片指令集架构文件。

&emsp; App Thinning 有三种方式，包括：
- App Slicing，会在你向 iTunes Connect 上传 App 后，对 App 做切割，创建不同的变体，这样就可以适用到不同的设备。
- On-Demand Resources，主要是为游戏多关卡场景服务的。它会根据用户的关卡进度下载随后几个关卡的资源，并且已经过关的资源也会被删掉，这样就可以减少初装 App 的包大小。
- Bitcode ，是针对特定设备进行包大小优化，优化不明显。


<br/>

&emsp; 如何在你项目里使用 App Thinning 呢？

&emsp; 其实，这里的大部分工作都是由 Xcode 和 App Store 来帮你完成的，你只需要通过 Xcode 添加 xcassets 目录，然后将图片添加进来即可。首先，新建一个文件选择 Asset Catalog 模板，如下图所示：

![ios_oc1_5.webp](./../../Pictures/ios_oc1_5.webp)

&emsp; 然后，按照 Asset Catalog 的模板添加图片资源即可，添加的 2x 分辨率的图片和 3x 分辨率的图片，会在上传到 App Store 后被创建成不同的变体以减小 App 安装包的大小。而芯片指令集架构文件只需要按照默认的设置， App Store 就会根据设备创建不同的变体，每个变体里只有当前设备需要的那个芯片指令集架构文件。

&emsp; 使用 App Thining 后，你可以将 2x 图和 3x 图区分开，从而达到减小 App 安装包体积的目的。如果我们要进一步减小 App 包体积的话，还需要在图片和代码上继续做优化。


<br/>
<br/>

> <h2 id='无用图片资源'>无用图片资源</h2>

&emsp; 图片资源的优化空间，主要体现在删除无用图片和图片资源压缩这两方面。而删除无用图片，又是其中最容易、最应该先做的。

<br/>

删除无用图片的过程，可以概括为下面这 6 大步:
- 通过 find 命令获取 App 安装包中的所有资源文件，比如 find /Users/daiming/Project/ -name
- 设置用到的资源的类型，比如 jpg、gif、png、webp
- 使用正则匹配在源码中找出使用到的资源名，比如 pattern = @"@"(.+?)""
- 使用 find 命令找到的所有资源文件，再去掉代码中使用到的资源文件，剩下的就是无用资源了
- 对于按照规则设置的资源名，我们需要在匹配使用资源的正则表达式里添加相应的规则，比如 @“image_%d”
- 确认无用资源后，就可以对这些无用资源执行删除操作了。这个删除操作，你可以使用 NSFileManger 系统类提供的功能来完成

<br/>

如果你不想自己重新写一个工具的话，可以选择开源的工具直接使用。我觉得目前最好用的是[LSUnusedResources](https://github.com/tinymind/LSUnusedResources)，特别是对于使用编号规则的图片来说，可以通过直接添加规则来处理。

![ios_oc1_6.gif](./../../Pictures/ios_oc1_6.gif)



<br/>
<br/>



> <h2 id='图片资源压缩'>图片资源压缩</h2>

&emsp; 对于 App 来说，图片资源总会在安装包里占个大头儿。对它们最好的处理，就是在不损失图片质量的前提下尽可能地作压缩。目前比较好的压缩方案是，将图片转成 WebP。WebP 是 Google 公司的一个开源项目。

首先，我们一起看看选择 WebP 的理由：
- WebP 压缩率高，而且肉眼看不出差异，同时支持有损和无损两种压缩模式。比如，将 Gif 图转为 Animated WebP ，有损压缩模式下可减少 64% 大小，无损压缩模式下可减少 19% 大小。
- WebP 支持 Alpha 透明和 24-bit 颜色数，不会像 PNG8 那样因为色彩不够而出现毛边。

<br/>

**如何把图片转成 WebP？**

Google 公司在开源 WebP 的同时，还提供了一个图片压缩工具 cwebp来将其他图片转成 WebP。[**cwebp**](https://developers.google.com/speed/webp/docs/precompiled) 使用起来也很简单，只要根据图片情况设置好参数就行。


<br/>

cwebp 语法如下：

```
cwebp [options] input_file -o output_file.webp
```

选择无损压缩如下:

```
cwebp -lossless original.png -o new.webp
```

其中，-lossless 表示的是，要对输入的 png 图像进行无损编码，转成 WebP 图片。不使用 -lossless ，则表示有损压缩。

在 cwebp 语法中，还有一个比较关键的参数 -q float。

图片色值在不同情况下，可以选择用 -q 参数来进行设置，在不损失图片质量情况下进行最大化压缩：

- 小于 256 色适合无损压缩，压缩率高，参数使用 -lossless -q 100；
- 大于 256 色使用 75% 有损压缩，参数使用 -q 75；
- 远大于 256 色使用 75% 以下压缩率，参数 -q 50 -m 6。


<br/>

除了 cwebp 工具外，你还可以选择由腾讯公司开发的[**iSparta。iSpart**](http://isparta.github.io/) 是一个 GUI 工具，操作方便快捷，可以实现 PNG 格式转 WebP，同时提供批量处理和记录操作配置的功能。如果是其他格式的图片要转成 WebP 格式的话，需要先将其转成 PNG 格式，再转成 WebP 格式。


<br/>

图片压缩完了并不是结束，我们还需要在显示图片时使用 libwebp 进行解析。这里有一个 iOS 工程使用 libwebp 的范例，你可以点击这个[链接查看。](https://github.com/carsonmcdonald/WebP-iOS-example)

不过，WebP 在 CPU 消耗和解码时间上会比 PNG 高两倍。所以，我们有时候还需要在性能和体积上做取舍。

我的建议是，如果图片大小超过了 100KB，你可以考虑使用 WebP；而小于 100KB 时，你可以使用网页工具 [**TinyPng**](https://tinypng.com/)或者 GUI 工具[**ImageOptim**](https://imageoptim.com/mac)进行图片压缩。这两个工具的压缩率没有 WebP 那么高，不会改变图片压缩方式，所以解析时对性能损耗也不会增加。




<br/>
<br/>

> <h2 id='代码瘦身'>代码瘦身</h2>

&emsp; App 的安装包主要是由资源和可执行文件组成的，所以我们在掌握了对图片资源的处理方式后，需要再一起来看看对可执行文件的瘦身方法。

&emsp; 可执行文件就是 Mach-O 文件，其大小是由代码量决定的。通常情况下，对可执行文件进行瘦身，就是找到并删除无用代码的过程。而查找无用代码时，我们可以按照找无用图片的思路，即：
- 首先，找出方法和类的全集；
- 然后，找到使用过的方法和类；
- 接下来，取二者的差集得到无用代码；
- 最后，由人工确认无用代码可删除后，进行删除即可。



<br/>
<br/>


> <h3 id='LinkMap结合Mach-O找无用代码'>LinkMap 结合 Mach-O 找无用代码</h3>

- **1.找到方法和类的全集**

**我们可以通过分析 LinkMap 来获得所有的代码类和方法的信息。**获取 LinkMap 可以通过将 Build Setting 里的 Write Link Map File 设置为 Yes，然后指定 Path to Link Map File 的路径就可以得到每次编译后的 LinkMap 文件了。设置选项如下图所示：

![ios_oc1_7.webp](./../../Pictures/ios_oc1_7.webp)

LinkMap 文件分为三部分：Object File、Section 和 Symbols。如下图所示：

![ios_oc1_8.webp](./../../Pictures/ios_oc1_8.webp)

其中：
- Object File 包含了代码工程的所有文件；
- Section 描述了代码段在生成的 Mach-O 里的偏移位置和大小；
- Symbols 会列出每个方法、类、block，以及它们的大小。

<br/>
<br/>

- **2.找到使用过的方法和类**

通过 LinkMap ，你不光可以统计出所有的方法和类，还能够清晰地看到代码所占包大小的具体分布，进而有针对性地进行代码优化。

iOS 的方法都会通过 objc_msgSend 来调用。而objc_msgSend 在 Mach-O 文件里是通过 __objc_selrefs 这个 section 来获取 selector 这个参数的。

所以，__objc_selrefs 里的方法一定是被调用了的。__objc_classrefs 里是被调用过的类，__objc_superrefs 是调用过 super 的类。通过 __objc_classrefs 和 __objc_superrefs，我们就可以找出使用过的类和子类。

那么，Mach-O 文件的 __objc_selrefs、__objc_classrefs 和 __objc_superrefs 怎么查看呢？

我们可以使用 [MachOView](https://sourceforge.net/projects/machoview/) 这个软件来查看 Mach-O 文件里的信息。[MachOView 同时也是一款开源软件](https://github.com/gdbinit/MachOView) 。

<br/>

下面让我们看一下一个Demo:
- 我们需要编译一个 App。在这里，我 [clone 了一个 GitHub 上的示例 下来编译](https://github.com/ming1016/GCDFetchFeed)。
- 然后，将生成的 GCDFetchFeed.app 包解开，取出 GCDFetchFeed。
- 最后，我们就可以使用 MachOView 来查看 Mach-O 里的信息了


![ios_oc1_9.webp](./../../Pictures/ios_oc1_9.webp)


如图上所示，我们可以看到 __objc_selrefs、__objc_classrefs 和、__objc_superrefs 这三个 section。


但是，这种查看方法并不是完美的，还会有些问题。原因在于， Objective-C 是门动态语言，方法调用可以写成在运行时动态调用，这样就无法收集全所有调用的方法和类。所以，我们通过这种方法找出的无用方法和类就只能作为参考，还需要二次确认。


<br/>
<br/>

- **3.通过 AppCode 找出无用代码**

&emsp; 如果工程量不是很大的话，我还是建议你直接使用 AppCode 来做分析,若是超过百万行则AppCode的静态分析会“歇菜”。毕竟代码量达到百万行的工程并不多。而那些代码量达到百万行的团队，则会自己通过 Clang 静态分析来开发工具，去检查无用的方法和类。

用 AppCode 做分析的方法很简单，直接在 AppCode 里选择 Code->Inspect Code 就可以进行静态分析。


![ios_oc1_10.webp](./../../Pictures/ios_oc1_10.webp)

静态分析完以后，我们可以在 Unused code 里看到所有的无用代码，如下图所示：

![ios_oc1_11.webp](./../../Pictures/ios_oc1_11.webp)

接下来，我和你说一下这些无用代码的主要类型。

- 无用类：Unused class 是无用类，Unused import statement 是无用类引入声明，Unused property 是无用的属性；
- 无用方法：Unused method 是无用的方法，Unused parameter 是无用参数，Unused instance variable 是无用的实例变量，Unused local variable 是无用的局部变量，Unused value 是无用的值；
- 无用宏：Unused macro 是无用的宏。
- 无用全局：Unused global declaration 是无用全局声明。

<br/>

这下我们把无用的代码找完了吗?

看似 AppCode 已经把所有工作都完成了，其实不然。下面，我再和你列举下 AppCode 静态检查的问题：
- JSONModel 里定义了未使用的协议会被判定为无用协议；
- 如果子类使用了父类的方法，父类的这个方法不会被认为使用了；
- 通过点的方式使用属性，该属性会被认为没有使用；
- 使用 performSelector 方式调用的方法也检查不出来，比如 self performSelector:@selector(arrivalRefreshTime)；
- 运行时声明类的情况检查不出来。比如通过 NSClassFromString 方式调用的类会被查出为没有使用的类，比如 layerClass = NSClassFromString(@“SMFloatLayer”)。还有以[[self class] accessToken] 这样不指定类名的方式使用的类，会被认为该类没有被使用。像 UITableView 的自定义的 Cell 使用 registerClass，这样的情况也会认为这个 Cell 没有被使用。

基于以上种种原因，使用 AppCode 检查出来的无用代码，还需要人工二次确认才能够安全删除掉。


<br/>
<br/>

**运行时检查类是否真正被使用过**

即使你使用 LinkMap 结合 Mach-O 或者 AppCode 的方式，通过静态检查已经找到并删除了无用的代码，那么就能说包里完全没有无用的代码了吗？

实际上，在 App 的不断迭代过程中，新人不断接手、业务功能需求不断替换，会留下很多无用代码。这些代码在执行静态检查时会被用到，但是线上可能连这些老功能的入口都没有了，更是没有机会被用户用到。也就是说，这些无用功能相关的代码也是可以删除的。

**那么，我们要怎么检查出这些无用代码呢？**

通过 ObjC 的 runtime 源码，我们可以找到怎么判断一个类是否初始化过的函数，如下：

```
#define RW_INITIALIZED (1<<29)
bool isInitialized() {
   return getMeta()->data()->flags & RW_INITIALIZED;
}
```

isInitialized 的结果会保存到元类的 class_rw_t 结构体的 flags 信息里，flags 的 1<<29 位记录的就是这个类是否初始化了的信息。而 flags 的其他位记录的信息，你可以参看 objc runtime 的源码，如下：

```
// 类的方法列表已修复
#define RW_METHODIZED         (1<<30)

// 类已经初始化了
#define RW_INITIALIZED        (1<<29)

// 类在初始化过程中
#define RW_INITIALIZING       (1<<28)

// class_rw_t->ro 是 class_ro_t 的堆副本
#define RW_COPIED_RO          (1<<27)

// 类分配了内存，但没有注册
#define RW_CONSTRUCTING       (1<<26)

// 类分配了内存也注册了
#define RW_CONSTRUCTED        (1<<25)

// GC：class有不安全的finalize方法
#define RW_FINALIZE_ON_MAIN_THREAD (1<<24)

// 类的 +load 被调用了
#define RW_LOADED             (1<<23)
```

flags 采用位方式记录布尔值的方式，易于扩展、所用存储空间小、检索性能也好。所以，经常阅读优秀代码，特别有助于提高我们自己的代码质量。

面试应聘的时候，常常会被问到“**苹果公司为什么要设计元类**”这样的开放问题。结果我们都只能说出元类是什么。

因为我们很多人都只是奔着学习 runtime 这个知识点而学习，并没有在学习过程中多想想为什么。比如，为什么类结构要这么设计，为什么一个类要设计两个结构体等等类似的问题。在我看来，没有经过深入思考的学习是不够的，是学不到精髓的，很多优秀的代码可能就会被错过。














<br/>

***
<br/>

> <h1 id=''></h1>



<br/>

***
<br/>

> <h2 id=''></h2>



<br/>

***
<br/>

> <h2 id=''></h2>



<br/>

***
<br/>

> <h2 id=''></h2>



<br/>

***
<br/>

> <h2 id=''></h2>



<br/>

***
<br/>

> <h2 id=''></h2>


<br/>

***
<br/>

> <h2 id=''></h2>


<br/>

***
<br/>

> <h2 id=''></h2>

<br/>

***
<br/>

> <h2 id=''></h2>


<br/>

***
<br/>

> <h2 id=''></h2>

