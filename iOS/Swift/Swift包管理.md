


- **SwiftPackageManager使用**
	- 	创建一个可执行的包
- **配置信息**
	- 	添加依赖
	- 	添加依赖的方式
	- 	添加系统依赖包
- [**SwiftPackageManager使用**](https://www.jianshu.com/p/44560fd214d2)

<br/>

***
<br/>

># SwiftPackageManager使用

&emsp;	Swift Package Manager 是一个苹果官方出的管理源代码分发的工具，目的是更简单的使用别人共享的代码。它会直接处理包之间的依赖管理、版本控制、编译和链接。从总体功能上来说，和 iOS 平台上的 Cocoapods、Carthage 一样。
&emsp;	Swift Package Manager (SwiftPM) 是 Apple 推出的一个包管理工具, 用于创建, 使用 Swift 的库, 以及可执行程序的工具




<br/>
<br/>



- **创建一个可执行的包**

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



<br/>

***
<br/>


># 配置信息

<br/>

- **添加依赖**

如果需要依赖其他的包， 需要在 Package.swift 定义依赖项和版本，像下面这样：

```
// swift-tools-version:4.2
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "WYMobileWebsite",
    dependencies: [
        .package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", from: "3.0.0"),
        .package(url:"https://github.com/PerfectlySoft/Perfect-MySQL.git", from: "3.0.0"),
        .package(url:"https://github.com/PerfectlySoft/Perfect-Session.git", from: "3.0.0")
    ],
      targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
            name: "WYMobileWebsite",
            dependencies: ["PerfectHTTPServer","PerfectMySQL","PerfectSession"]),
        .testTarget(
            name: "WYMobileWebsiteTests",
            dependencies: ["WYMobileWebsite"]),
    ]
)

```

&emsp;	Package.dependencies 用于添加包的依赖，一般是包括指向包源的 git 路径和版本环境，或指向依赖包的本地路径.
&emsp;	在执行 Swift build 时会自动执行一个 swift package resolve 命令，该命令会解析 Package.swift 的依赖，并生成对应的 package.resolved 文件


<br/>
<br/>

- **添加依赖的方式**
	- git 源 + 确定的版本号
	- git 源 + 版本区间
	- git 源 + Commit 号
	- git 源 + 分支名
	- 本地路径

如：

```
.package(url: "https://github.com/Alamofire/Alamofire.git", .exact("1.2.3")),
.package(url:"https://github.com/Alamofire/Alamofire.git", .branch("master")),
.package(url:"https://github.com/Alamofire/Alamofire.git", from: "1.2.3"),
.package(url: "https://github.com/Alamofire/Alamofire.git", .revision("e74b07278b926c9ec6f9643455ea00d1ce04a021"),
.package(url: "https://github.com/Alamofire/Alamofire.git", "1.2.3"..."4.1.3"),
.package(path: "../Foo"),


```



<br/>
<br/>


- **添加系统依赖包**

`Package.SupportedPlatform(系统支持版本)` 

这个 Struct 用于设置包的最小依赖平台版本，具体 API 定义可以进入代码文档中查看，下面给出示例：

```
    platforms: [.macOS(.v10_10)],

```

**`注意：`**需要注意的是虽然这个属性是个数组，但是目的是为了让设置不同平台的最小依赖，如果设置了多个同平台的值进去，就会报错，例如这样：`[.macOS(.v10_10), .macOS(.v10_11)]`















