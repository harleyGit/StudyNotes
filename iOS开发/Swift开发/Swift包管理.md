# Swift Package Manager 使用

&emsp;	Swift Package Manager 是一个苹果官方出的管理源代码分发的工具，目的是更简单的使用别人共享的代码。它会直接处理包之间的依赖管理、版本控制、编译和链接。从总体功能上来说，和 iOS 平台上的 Cocoapods、Carthage 一样。
&emsp;	Swift Package Manager (SwiftPM) 是 Apple 推出的一个包管理工具, 用于创建, 使用 Swift 的库, 以及可执行程序的工具



***
<br/>



- `创建一个可执行的包`
```
mkdir MyPackage

cd MyPackage



/*
type类型：
	empty(空包):	Source 文件夹下什么都没有，也不能编译

	library(静态包):	Source 文件夹下有个和包同名 swift 文件，里面有个空结构体

	executable(可执行包):	Source 文件夹下有个 main.swift 文件，在 build 之后会在 .build/debug/ 目录下生成一个可执行文件，可通过 swift run 或者直接点击运行，从而启动一个进程

	system-module(系统包):	这种包是专门为了链接系统库（例如 libgit、jpeglib、mysql 这种系统库）准备的，本身不需要任何代码，所以也没有 Source 文件夹，但是需要编辑 module.modulemap 文件去查找系统库路径 (Swift 4.2 已经被其他方式取代)
*/

//包的初始化，创建多个包文件
swift package init --type executable


swift build

swift run

```



