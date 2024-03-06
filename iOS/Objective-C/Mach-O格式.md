> <h2 id=''></h2>
- [**Mach-O**](#Mach-O)
- [**‌文件结构**](#文件结构)
	- [header](#header)
	- [查看Mach-o命令](#查看Mach-o命令)
- **资料**
	- Objective-C runtime机制(前传)——Mach-O格式[](https://blog.csdn.net/u013378438/article/details/80353267?spm=1001.2014.3001.5502)


<br/>

***
<br/><br/>

> <h1 id='Mach-O'>Mach-O</h1>

&emsp; Mach-O是Mach Object文件格式的缩写。它是用于可执行文件，动态库，目标代码的文件格式。作为a.out格式的替代，Mach-O格式提供了更强的扩展性，以及更快的符号表信息访问速度。


&emsp; Mach-O格式为大部分基于Mach内核的操作系统所使用的，包括NeXTSTEP, Mac OS X和iOS，它们都以Mach-O格式作为其可执行文件，动态库，目标代码的文件格式。


&emsp; 具体到我们的iOS程序，当用XCode打包后，会生成一个.app为扩展名的文件（位于工程目录/Products文件夹下），其实.app是一个文件夹，我们用鼠标右键选择‘Show Package contents’，就可以查看文件夹的内容，其中会发现有一个和我们工程同名的unix 可执行文件，这个就是iOS可执行文件，它是符合Mach-O格式的。



![ios_oc1_113_41.png](./../../Pictures/ios_oc1_113_41.png)







<br/>

***
<br/><br/>


> <h1 id='文件结构'>文件结构</h1>

<br/>



&emsp; **Mach-O 文件格式是 iOS 和 macOS 使用的一套可执行文件的文件格式**，类似于Windows 平台上使用的 PE 文件格式，以及 Linux 平台上使用的 ELF 文件格式。

<br/>

![ios_oc94.png](./../../Pictures/ios_oc94.png)


**Mach-O 文件主要构成:**
-  header（头部）:对Mach-O文件的一个概要说明，包括Magic Number, 支持的CUP类型等;

<br/>

-  Load command（加载命令）:当系统加载Mach-O文件时，load command会指导苹果的动态加载器(dyld)h或内核，该如何加载文件的Data数据;


<br/>


-  Data 三部分组成:Mach-O文件的数据区，包含代码和数据。其中包含若干Segment块（注意，除了Segment块之外，还有别的内容，包括code signature，符号表之类，不要被苹果的图所误导！），每个Segment块中包含0个或多个seciton。Segment根据对应的load command被dyld加载入内存中。
	-  其中 Data 包含多个 Segment（段），Segment 包含多个Section（节区）.



<br/>
<br/>


&emsp; 查看Mach-O文件,可以使用MachOView软件.不同的CPU架构对应不同的Mach-O文件,比如同时支持 ARMv7 和 ARM64.可以将这2个不同的架构的Mach-O文件进行合并称为fat文件.




<br/><br/>

> <h2 id='header'>header</h2>


一个普通的iOS APP为例，使用MachOView看看Mach-O文件header部分的具体内容。通过MachOView打开可执行文件，可以看到header的结构：

![ios_oc1_113_42.png](./../../Pictures/ios_oc1_113_42.png)

是不是有些懵？下面我们就结合Darwin内核源码，来了解下Mach header的定义。

Mach header的定义位于Darwin源码中的 EXTERNAL_HEADERS/mach-o/loader.h 中：

**32位:**

```
struct mach_header {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
};
```

<br/>

**64位:**

```
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};
```

可以看到，32位和64位的Mach header基本相同，只不过64位header中多了一个保留参数reserved。


<br/>

- **magic：** 魔数，用来标识这是一个Mach-O文件，有32位和64位两个版本：

```
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
```



- **cputype：** 支持的CUP架构类型，如arm。

<br/>


- **cpusubtype：** 在支持的CUP架构类型下，所支持的具体机器型号。在我们的例子中，APP是支持所有arm64的机型的:CUP_SUBTYPE_ARM64_ALL

<br/>

- **filetype:** Mach-O的文件类型。包括：

```
#define	MH_OBJECT	0x1		/* Target 文件：编译器对源码编译后得到的中间结果 */
#define	MH_EXECUTE	0x2		/* 可执行二进制文件 */
#define	MH_FVMLIB	0x3		/* VM 共享库文件（还不清楚是什么东西） */
#define	MH_CORE		0x4		/* Core 文件，一般在 App Crash 产生 */
#define	MH_PRELOAD	0x5		/* preloaded executable file */
#define	MH_DYLIB	0x6		/* 动态库 */
#define	MH_DYLINKER	0x7		/* 动态连接器 /usr/lib/dyld */
#define	MH_BUNDLE	0x8		/* 非独立的二进制文件，往往通过 gcc-bundle 生成 */
#define	MH_DYLIB_STUB	0x9		/* 静态链接文件（还不清楚是什么东西） */
#define	MH_DSYM		0xa		/* 符号文件以及调试信息，在解析堆栈符号中常用 */
#define	MH_KEXT_BUNDLE	0xb		/* x86_64 内核扩展 */
```


<br/>

- **ncmds：**load command的数量

<br/>


- **sizeofcmds:** 所有load command的大小

<br/>

- **flags：** Mach-O文件的标志位。主要作用是告诉系统该如何加载这个Mach-O文件以及该文件的一些特性。有很多值，我们取常见的几种：

```
#define	MH_NOUNDEFS	0x1		/* Target 文件中没有带未定义的符号，常为静态二进制文件 */
#define MH_SPLIT_SEGS	0x20  /* Target 文件中的只读 Segment 和可读写 Segment 分开  */
#define MH_TWOLEVEL	0x80		/* 该 Image 使用二级命名空间(two name space binding)绑定方案 */
#define MH_FORCE_FLAT	0x100 /* 使用扁平命名空间(flat name space binding)绑定（与 MH_TWOLEVEL 互斥） */
#define MH_WEAK_DEFINES	0x8000 /* 二进制文件使用了弱符号 */
#define MH_BINDS_TO_WEAK 0x10000 /* 二进制文件链接了弱符号 */
#define MH_ALLOW_STACK_EXECUTION 0x20000/* 允许 Stack 可执行 */
#define	MH_PIE 0x200000  /* 加载程序在随机的地址空间，只在 MH_EXECUTE中使用 */
#define MH_NO_HEAP_EXECUTION 0x1000000 /* 将 Heap 标记为不可执行，可防止 heap spray 攻击 */
```



<br/><br/><br/>

> <h2 id='查看Mach-o命令'>查看Mach-o命令</h2>


> fat文件

![fat文件结构](./../../Pictures/ios_oc95.png)


查看fat文件,我们可以使用Xcode自带的otool工具

- 查看fat文件的头部:

```
$ otool -f /usr/bin/file
```


<br/>

- 查看Mach文件的头部:

```
$ otool -h /usr/bin/file
```


<br/>

- 查看Load Command信息:


```
$ otool -l /usr/bin/file
```






<br/>

***
<br/>

> <h2 id=''></h2>

<br/>



<br/>

***
<br/>

> <h2 id=''></h2>

<br/>



<br/>

***
<br/>

> <h2 id=''></h2>

<br/>



<br/>

***
<br/>

> <h2 id=''></h2>

<br/>

<br/>

***
<br/>

> <h2 id=''></h2>

<br/>