> <h2 id=''></h2>
- [**进程与二进制格式**](#进程与二进制格式)
	- [通用二进制格式](#通用二进制格式)







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


<br/>
<br/>


> <h2 id=''></h2>






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










