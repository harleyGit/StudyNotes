> <h2 id=''></h2>
- [**进程与二进制格式**](#进程与二进制格式)
	- [通用二进制格式](#通用二进制格式)
	- [Mach-O 文件格式](#Mach-O文件格式)
		- [Mach-O 头](#Mach-O头)
		- [Mach-O Data](#Mach-OData)







<br/>

***
<br/>



> <h1 id='进程与二进制格式'>[进程与二进制格式](https://www.desgard.com/macos%20kernel/objective-c/iosre/2017/10/07/iosre-1.html)</h1>


&emsp; 进程在众多操作系统中都有提及，它是作为一个正在执行的程序的实例，这是 UNIX 的一个基本概念。而进程的出现是特殊文件在内存中加载得到的结果，这种文件必须使用操作系统可以认知的格式，这样才对该文件引入依赖库，初始化运行环境以及顺利地执行创造条件。

&emsp; Mach-O（Mach Object File Format）是 macOS 上的可执行文件格式，类似于 Linux 和大部分 UNIX 的原生格式 ELF（Extensible Firmware Interface）。为了更加全面的了解这块内容，我们看一下 macOS 支持的三种可执行格式：解释器脚本格式、通用二进制格式和 Mach-O 格式。


<br/>

> **Mach-O 格式的文件类型:**

|**类型**|**名称**|
|:--|:--|
| Mach-O Object | 目标文件  .o |
| Mach-O Ececutable | 可执行文件 |
| Mach-O Dynamically | 动态库文件 .a 、 .dylib |
| Mach-O dynamic linker | 动态链接器文件 |
| Mach-O dSYM companion | 符号表文件 |
| Bundle  | 自己创建的动态库，运行在沙盒中，无法被 dyld 链接，只能通过 dlopen() 加载 |
| Framework | 包含Dylib、资源文件和头文件的文件夹 |
 

<br/>

> **Mach-O文件格式和用途**


|  **可执行格式**  |  **magic** |  **用途** |
|:--|:--|:--|
|  脚本 | \x7FELF  | 主要用于 shell 脚本，但是也常用语其他解释器，如 Perl, AWK 等。也就是我们常见的脚本文件中在 #! 标记后的字符串，即为执行命令的指令方式，以文件的 stdin 来传递命令  |
| 通用二进制格式  | 0xcafebabe<br/>0xbebafeca  | 包含多种架构支持的二进制格式，只在 macOS 上支持  |
|  Mach-O |  0xfeedface（32 位） <br/>0xfeedfacf（64 位） | macOS 的原生二进制格式  |





<br/>
<br/>


> <h2 id='通用二进制格式'>通用二进制格式</h2>


<br/>



&emsp; 这个格式在有些资料中也叫胖二进制格式（Fat Binary），Apple 提出这个概念是为了解决一些历史原因，macOS（更确切的应该说是 OS X）最早是构建于 PPC 架构之上，后来才移植到 Intel 架构（从 Mac OS X Tiger 10.4.7 开始），通用二进制格式的二进制文件可以在 PPC 和 x86 两种处理器上执行。

&emsp; 说到底，通用二进制格式只不过是对多架构的二进制文件的打包集合文件，而 macOS 中的多架构二进制文件也就是适配不同架构的 Mach-O 文件。

&emsp; Fat Header 的数据结构在 <mach-o/fat.h> 头文件中有定义，可以参看 /usr/include/mach-o/fat.h 找到定义头：

```
#define FAT_MAGIC	0xcafebabe
#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */

struct fat_header {
	uint32_t	magic;		/* FAT_MAGIC 或 FAT_MAGIC_64 */
	uint32_t	nfat_arch;	/* 结构体实例的个数 */
};

struct fat_arch {
	cpu_type_t	cputype;	/* cpu 说明符 (int) */
	cpu_subtype_t	cpusubtype;	/* 指定 cpu 确切型号的整数 (int) */
	uint32_t	offset;		/* CPU 架构数据相对于当前文件开头的偏移值 */
	uint32_t	size;		/* 数据大小 */
	uint32_t	align;		/* 数据内润对其边界，取值为 2 的幂 */
};

```


&emsp;对于 cputype 和 cpusubtype 两个字段这里不讲述，可以参看 /usr/include/mach/machine.h 头中对其的定义，另外 Apple 官方文档中也有简单的描述。

&emsp;在 fat_header 中，magic 也就是我们之前在表中罗列的 magic 标识符，也可以类比成 UNIX 中 ELF 文件的 magic 标识。加载器会通过这个符号来判断这是什么文件，通用二进制的 magic 为 0xcafebabe。nfat_arch 字段指明当前的通用二进制文件中包含了多少个不同架构的 Mach-O 文件。fat_header 后会跟着多个 fat_arch，并与多个 Mach-O 文件及其描述信息（文件大小、CPU 架构、CPU 型号、内存对齐方式）相关联。


&emsp; 在运行iOS会生成一个 .app 文件，位于 工程目录/Product 文件夹下，.app 文件实际上是一个文件夹，我们可以右键，显示包内容，就可以查看文件夹中的内容了，其中有一个与工程同名的 Unix 可执行文件，这就是一个 Mach-O 格式的文件。我们可以使用 **`file、lipo -info `** 命令来查看,如下:

![查看cpu类型](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc79.png)


<br/>

使用Mac 自带的 **`otool`** 工具来打印其 fat_header 详细信息: `otool -f -V Desktop/xxx.app/xxx`

<br/>

&emsp; 之后我们用 Synalyze It! Pro 来查看 WeChat 的 Mach64 Header 的效果：

![没法看](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc80.png)




<br/>
<br/>


> <h2 id='Mach-O文件格式'>Mach-O 文件格式</h2>


![>Mach-O 文件格式](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc81.png)


- **Mach-O 主要由 3 部分组成:**

	- Mach-O 头（Mach Header）：这里描述了 Mach-O 的 CPU 架构、文件类型以及加载命令等信息；
	- 加载命令（Load Command）：描述了文件中数据的具体组织结构，不同的数据类型使用不同的加载命令表示；
	- 数据区（Data）：Data 中每一个段（Segment）的数据都保存在此，段的概念和 ELF 文件中段的概念类似，都拥有一个或多个 Section ，用来存放数据和代码。


<br/>


> <h3 id='Mach-O头'>Mach-O 头</h3>


&emsp; 与 Mach-O 文件格式有关的结构体定义都可以从 /usr/include/mach-o/loader.h 中找到，也就是 <mach-o/loader.h> 头。以下只给出 64 位定义的代码，因为 32 位的区别是缺少了一个预留字段：

```
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */

struct mach_header_64 {
	uint32_t	magic;		/* mach magic 标识符 */
	cpu_type_t	cputype;	/* CPU 类型标识符，同通用二进制格式中的定义 */
	cpu_subtype_t	cpusubtype;	/* CPU 子类型标识符，同通用二级制格式中的定义 */
	uint32_t	filetype;	/* 文件类型 */
	uint32_t	ncmds;		/* 加载器中加载命令的条数 */
	uint32_t	sizeofcmds;	/* 加载器中加载命令的总大小 */
	uint32_t	flags;		/* dyld 的标志 */
	uint32_t	reserved;	/* 64 位的保留字段 */
};
```


由于 Mach-O 支持多种类型文件，所以此处引入了 filetype 字段来标明，这些文件类型定义在 loader.h 文件中同样可以找到。

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


> <h3 id='Mach-OData'>Mach-O Data</h3>


&emsp; 加载命令在 Mach-O 文件加载解析时，会被内核加载器或者动态链接器调用。这些指令都采用 Type-Size-Value 这种格式，即：32 位的 cmd 值（表示类型），32 位的 cmdsize 值（32 位二级制位 4 的倍数，64 位位 8 的倍数），以及命令本身（由 cmdsize 指定的长度）。内核加载器使用的命令可以参看 xnu 源码来学习，其他命令则是由动态链接器处理的。

在正式进入加载命令这一过程之前，先来学习一下 Mach-O 的 Data 区域，其中由 Segment 段和 Section 节组成。先来说 Segment 的组成，以下代码仍旧来自 loader.h：

 ```
#define	SEG_PAGEZERO	"__PAGEZERO" /* 当时 MH_EXECUTE 文件时，捕获到空指针 */
#define	SEG_TEXT	"__TEXT" /* 代码/只读数据段 */
#define	SEG_DATA	"__DATA" /* 数据段 */
#define	SEG_OBJC	"__OBJC" /* Objective-C runtime 段 */
#define	SEG_LINKEDIT	"__LINKEDIT" /* 包含需要被动态链接器使用的符号和其他表，包括符号表、字符串表等 */
```

&emsp; 进而来看一下 Segment 的数据结构具体是什么样的（同样这里也只放出 64 位的代码，与 32 位的区别就是其中 uint64_t 类型的几个字段取代了原先 32 位类型字段）：

```
struct segment_command_64 { 
	uint32_t	cmd;		/* LC_SEGMENT_64 */
	uint32_t	cmdsize;	/* section_64 结构体所需要的空间 */
	char		segname[16];	/* segment 名字，上述宏中的定义 */
	uint64_t	vmaddr;		/* 所描述段的虚拟内存地址 */
	uint64_t	vmsize;		/* 为当前段分配的虚拟内存大小 */
	uint64_t	fileoff;	/* 当前段在文件中的偏移量 */
	uint64_t	filesize;	/* 当前段在文件中占用的字节 */
	vm_prot_t	maxprot;	/* 段所在页所需要的最高内存保护，用八进制表示 */
	vm_prot_t	initprot;	/* 段所在页原始内存保护 */
	uint32_t	nsects;		/* 段中 Section 数量 */
	uint32_t	flags;		/* 标识符 */
};
```

<br/>


&emsp; 部分的 Segment （主要指的 __TEXT 和 __DATA）可以进一步分解为 Section。之所以按照 Segment -> Section 的结构组织方式，是因为在同一个 Segment 下的 Section，可以控制相同的权限，也可以不完全按照 Page 的大小进行内存对其，节省内存的空间。而 Segment 对外整体暴露，在程序载入阶段映射成一个完整的虚拟内存，更好的做到内存对齐（可以继续参考 OS X & iOS Kernel Programming 一书的第一章内容）。下面给出 Section 具体的数据结构：

```
struct section_64 { 
	char		sectname[16];	/* Section 名字 */
	char		segname[16];	/* Section 所在的 Segment 名称 */
	uint64_t	addr;		/* Section 所在的内存地址 */
	uint64_t	size;		/* Section 的大小 */
	uint32_t	offset;		/* Section 所在的文件偏移 */
	uint32_t	align;		/* Section 的内存对齐边界 (2 的次幂) */
	uint32_t	reloff;		/* 重定位信息的文件偏移 */
	uint32_t	nreloc;		/* 重定位条目的数目 */
	uint32_t	flags;		/* 标志属性 */
	uint32_t	reserved1;	/* 保留字段1 (for offset or index) */
	uint32_t	reserved2;	/* 保留字段2 (for count or sizeof) */
	uint32_t	reserved3;	/* 保留字段3 */
};
```


<br/>

下面列举一些常见的Section:

| **section** | **用途** |
|:--|:--|
| __TEXT.__text | 	主程序代码 |
| __TEXT.__cstring | C 语言字符串 |
| __TEXT.__const | const 关键字修饰的常量 |
| __TEXT.__stubs | 用于 Stub 的占位代码，很多地方称之为桩代码。 |
| __TEXT.__stubs_helper | 当 Stub 无法找到真正的符号地址后的最终指向 |
| __TEXT.__objc_methname | Objective-C 方法名称 |
| __TEXT.__objc_methtype | Objective-C 方法类型 |
| __TEXT.__objc_classname | Objective-C 类名称 |
| __DATA.__data | 初始化过的可变数据 |
| __DATA.__la_symbol_ptr	 | lazy binding 的指针表，表中的指针一开始都指向 __stub_helper |
| __DATA.nl_symbol_ptr | 非 lazy binding 的指针表，每个表项中的指针都指向一个在装载过程中，被动态链机器搜索完成的符号 |
| __DATA.__const	 | 没有初始化过的常量 |
| __DATA.__cfstring | 程序中使用的 Core Foundation 字符串（CFStringRefs） |
| __DATA.__bss | BSS，存放为初始化的全局变量，即常说的静态内存分配 |
| __DATA.__common | 没有初始化过的符号声明 |
| __DATA.__objc_classlist | Objective-C 类列表 |
| __DATA.__objc_protolist	 | Objective-C 原型 |
| __DATA.__objc_imginfo | Objective-C 镜像信息
 |
| __DATA.__objc_selfrefs | Objective-C self 引用 |
| __DATA.__objc_protorefs	 | Objective-C 原型引用 |
| __DATA.__objc_superrefs | Objective-C 超类引用
 |

<br/>


**验证实验**

&emsp; 当了解了 Segment 和 Section 的定义之后，我们可以简单的探索一下 LC_SEGMENT 这个命令的过程。用 helloworld 来做个试验：

```
// main.cpp
#import <stdio.h>

int main() {
    printf("hello");
    return 0;
}
```

&emsp; 使用 clang -g main.cpp -o main 生成执行文件。然后拖入到 MachOView 中来查看一下加载 Segment 的结构（当然使用 Synalyze It! 也能捕捉到这些信息的，但是 MachOView 更对结构的分层更加一目了然）：

![图一](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc82.jpg)

<br/>

![图二](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc83.jpg)


&emsp; 在 LC_SEGMENT_64 中有四个元素，分别是 __PAGEZERO、__TEXT、__DATA、__LINKEDIT 这四个 Segment。其中，__TEXT 的 __text Section 的加载是可以验证到的，我们从 Section 的实例中取出其 addr 来对比汇编之后代码的起始地址即可。使用 otool -vt main 来获取其汇编代码：




```
$ otool -vt main
main:
(__TEXT,__text) section
_main:
0000000100000f60	pushq	%rbp
0000000100000f61	movq	%rsp, %rbp
0000000100000f64	subq	$0x10, %rsp
0000000100000f68	leaq	0x3b(%rip), %rdi
0000000100000f6f	movl	$0x0, -0x4(%rbp)
0000000100000f76	movb	$0x0, %al
0000000100000f78	callq	0x100000f8a
0000000100000f7d	xorl	%ecx, %ecx
0000000100000f7f	movl	%eax, -0x8(%rbp)
0000000100000f82	movl	%ecx, %eax
0000000100000f84	addq	$0x10, %rsp
0000000100000f88	popq	%rbp
0000000100000f89	retq

```

对比 Synalyze It! 的分析结果中 SEG__TEXT.__text 中的 addr 观察：

![图三](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc84.jpg)

其对应的物理地址均为 0x100000F60，说明其 LC_SEGMENT 对于 Segment 和 Section 的加载与我们的预期完全一致。


<br/>
<br/>

**__TEXT.__stubs 的探究**

&emsp; Stub 是指用来替换一部分功能的程序段。桩程序可以用来模拟已有程序的行为（比如一个远端机器的过程）或是对将要开发的代码的一种临时替代。


将 Calculator 应用拖入到 Synalyze It! 和 Hopper Disassembler 中。首先使用 Synalyze It! 来查找一个 __stubs 地址：


![图四](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc85.jpg)

取出地址 0x100016450 并在 Hopper 中查找对应的代码，并以此双击进入：

![图五](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc86.jpg)


![图6](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc87.jpg)

到达第二幅图的位置的时候，我们发现无法继续进入，因为 _CFRelease 中的代码是没有意义的。我们拿出 0x100031000 这个首地址，在 MachOView 中查找：
	
![图8](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc88.jpg)

发现其低 32 位的值为 0x00000010001663E，将这个地址继续在 hopper 中搜索：

![图9](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc89.jpg)

发现这一系列操作都会跳到 0x000000010001650c 这个位置，而这里就是 __TEXT.__stub_helper 的表头。

![图10](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/ios_oc90.jpg)

&emsp; 也就是说，__la_symbol_ptr 里面的所有表项的数据在开始时都会被 binding 成 __stub_helper。而在之后的调用中，虽然依旧会跳到 __stub 区域，但是 __la_symbol_ptr 中由于在之前的调用中获取到了对应方法的真实地址，所以无需在进入 dyld_stub_binder 阶段，并直接调用函数。这样就完成了一次近似于 lazy 思想的延时 binding 过程。（这个过程可以用 lldb 来加以验证，在之后会补充。）

总结一下 Stub 机制。其实和 wikipedia 上的说法一致，设置函数占位符并采用 lazy 思想做成延迟 binding 的流程。在 macOS 中也是如此，外部函数引用在 __DATA 段的 __la_symbol_ptr 区域先生产一个占位符，当第一个调用启动时，就会进入符号的动态链接过程，一旦找到地址后，就将 __DATA Segment 中的 __la_symbol_ptr Section 中的占位符修改为方法的真实地址，这样就完成了只需要一个符号绑定的执行过程。




	

	
	
	
	

		

	






<br/>

***
<br/>



> <h1 id=''></h1>




<br/>
<br/>


> <h2 id=''></h2>




<br/>
<br/>


> <h2 id=''></h2>










