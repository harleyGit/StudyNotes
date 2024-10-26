> <h2 id=""></h2>
- [**语法**](#语法)
	- [CocoaPods 的默认官方仓库地址](#CocoaPods的默认官方仓库地址)
		- [:git参数获取仓库代码](#:git参数获取仓库代码) 
		- [:tag参数指定标签版本](#:tag参数指定标签版本) 
		- [:branch指定Git仓库的分支](#:branch指定Git仓库的分支) 
		- [:commit指定某个特定提交](#:commit指定某个特定提交) 
		- [:path指定本地仓库地址](#:path指定本地仓库地址) 
		- [:podspec指定spec文件路径](#:podspec指定spec文件路径) 
		- [:binary用于某些特殊库分发二进制格式](#:binary用于某些特殊库分发二进制格式) 
		- [综合示例](#综合示例)
	- [自己私有库内的库名与三方库一样，如何解决](#自己私有库内的库名与三方库一样，如何解决)
		- [使用 :branch、:tag 或 :commit 来锁定特定版本或分支](#使用:branch、:tag或:commit来锁定特定版本或分支)
			- [方案一](#方案一)
			- [方案二](#方案二)
				- [post_install钩子代码理解](#post_install钩子代码理解)
- [**问题解决**](#问题解决)
	- [libarclite_iphoneos.a找不到](#libarclite_iphoneos.a找不到)
	- [**.bashrc和和.zshrc区别**](#.bashrc和和.zshrc区别)
- [**使用**](#使用)
	- [工作基本指令](#工作基本指令)
	- [pod update 和 pod install](#podupdate和podinstall)
		- [下载库失败：error: RPC failed; curl 18 transfer closed with outstanding read data remaining](#下载库失败：errorRPC)
	- [脚本命令](#脚本命令)
- [**RVM安装**](#RVM安装)
- [**安装Ruby**](#安装Ruby)
	- [安装最新ruby](#安装最新ruby)
	- [移除OpenSSL3](#移除OpenSSL3)
- [**CocoaPosds**](#CocoaPosds)
	- [**安装**](#安装) 
		- [检测pod是否安装成功](#检测pod是否安装成功)
			- [出错: Unable to xxx](#出错:Unabletoxxx)
				- [在Intel芯片OS14.4错误:Unable to find a pod with name](#在Intel芯片OS14.4错误:Unabletofindapodwithname)
	- [M1安装](#M1安装)
	- [安装ruby](#安装ruby)
		- [问题1:CFLAGS=-O3](#问题1:CFLAGS=-O3)
		- [问题2:__rvm_make-j8](#问题2:__rvm_make-j8)
			- [问题2.1:Intel系列OS14.4——__rvm_make-j8](#问题2.1:Intel系列OS14.4——__rvm_make-j8)
		- [安装cocoapods(macOS10.11之后使用)](#安装cocoapods(macOS10.11之后使用))
	- [**错误解决方案**](#错误解决方案)
	- [**卸载CocoaPosds**](#卸载CocoaPosds)
- **资料**
	- [CocoaPods使用指南](https://juejin.cn/post/7179231344147300412)
	- [Cocoapods 使用](https://hanleylee.com/articles/usage-of-cocoapods/)
- [**pynev:python的版本管理工具**](#pynev:python的版本管理工具)
- [**Python安装‌**](#Python安装)




<br/>

***
<br/><br/><br/>

> <h1 id="语法">语法</h1>

<br/><br/><br/>

> <h2 id="CocoaPods的默认官方仓库地址">CocoaPods的默认官方仓库地址</h2>

[`source 'https://github.com/CocoaPods/Specs.git'` ](https://github.com/CocoaPods/Specs/tree/master)是 CocoaPods 的默认官方仓库地址，其中包含了许多开源库的 spec 文件，用于指定库的依赖和版本信息。

除了 `source` 参数指定的默认仓库，CocoaPods 中还有其他的参数可以用来自定义 Pod 的来源，以下是一些常见的用法：

<br/><br/>

> <h3 id=":git参数获取仓库代码">:git参数获取仓库代码</h3>

- **`:git`**  
   用于指定一个 Git 仓库地址，直接从这个仓库中获取库代码。通常搭配 `:tag`、`:branch` 或 `:commit` 使用来锁定版本。

   ```ruby
   pod 'LibraryName', :git => 'https://github.com/username/LibraryName.git'
   ```

<br/><br/>

> <h3 id=":tag参数指定标签版本">:tag参数指定标签版本</h3>

- **`:tag`**
   指定一个特定的 Git 标签版本。
   
   ```ruby
   pod 'LibraryName', :git => 'https://github.com/username/LibraryName.git', :tag => '1.0.0'
   ```


<br/><br/>

> <h3 id=":branch指定Git仓库的分支">:branch指定Git仓库的分支</h3>

- **`:branch`**
   
   指定一个 Git 仓库的分支。
   
```ruby
pod 'LibraryName', :git => 'https://github.com/username/LibraryName.git', :branch => 'develop'
```

<br/><br/>

> <h3 id=":commit指定某个特定提交">:commit指定某个特定提交</h3>


- **`:commit`**
  
  指定 Git 仓库中的某个特定提交（commit ID）。
 
```ruby
pod 'LibraryName', :git => 'https://github.com/username/LibraryName.git', :commit => 'abcd1234'
```

<br/><br/>

> <h3 id=":path指定本地仓库地址">:path指定本地仓库地址</h3>


- **`:path`**
   如果库位于本地，可以使用 `:path` 参数指定路径。这个方法适合在开发库时测试本地版本。
  
```ruby
pod 'LibraryName', :path => '~/Documents/LocalPods/LibraryName'
```

<br/><br/>

> <h3 id=":podspec指定spec文件路径">:podspec指定spec文件路径</h3>


- **`:podspec`**
   如果你有一个 podspec 文件（spec 文件），可以使用 `:podspec` 参数指定文件路径。
   
```ruby
pod 'LibraryName', :podspec => '~/Documents/LocalPods/LibraryName/LibraryName.podspec'
```

<br/><br/>

> <h3 id=":binary用于某些特殊库分发二进制格式">:binary用于某些特殊库分发二进制格式</h3>


- **`:binary`**

   使用二进制文件安装。适用于某些特殊库分发二进制格式（如 xcframeworks）。

```ruby
pod 'LibraryName', :binary => true
```

<br/><br/>

> <h3 id="综合示例">综合示例</h3>

以下是一个包含不同参数的 `Podfile` 示例：

```ruby
platform :ios, '13.0'
use_frameworks!

source 'https://github.com/CocoaPods/Specs.git'

target 'YourApp' do
  # 从指定仓库的分支安装
  pod 'LibraryName', :git => 'https://github.com/username/LibraryName.git', :branch => 'main'

  # 从指定的 tag 版本安装
  pod 'AnotherLibrary', :git => 'https://github.com/username/AnotherLibrary.git', :tag => '2.1.0'

  # 从本地路径安装
  pod 'LocalLibrary', :path => '~/Documents/LocalPods/LocalLibrary'

  # 从指定的提交版本安装
  pod 'CommitLibrary', :git => 'https://github.com/username/CommitLibrary.git', :commit => 'abcd1234'
end
``` 




<br/><br/><br/>

> <h2 id="自己私有库内的库名与三方库一样，如何解决">自己私有库内的库名与三方库一样，如何解决</h2>

```
 pod 'SwiftLint', '0.57.0'
```

就会出现如下提示错误：

![ios0.0.60.png](./../../Pictures/ios0.0.60.png)


原因是当时在YSHoby项目中，公司的一套私有库中有一个SwiftLint库，其实是对SwiftLint裹了一层，但是也叫SwiftLint库。

我是怎么发现的呢？,通过如下命令：

```
pod search SwiftLint
```

![ios0.0.61.png](./../../Pictures/ios0.0.61.png)


这时我建立一个Demo，用到了SwiftLint库，但是再Pod install时，下载的还是YSHoby中私有库中的SwiftLint类，不是从网上GitHub下载的，那该如何做呢？

<br/>

这时我们可以我们的库指定从哪个地方下载就好了。

```cocoapod
pod 'SwiftLint', :git => 'https://github.com/realm/SwiftLint.git', :tag =>'0.57.0'
```

这个`:git`后的地址，通过如下图获取：

![ios0.0.62.png](./../../Pictures/ios0.0.62.png)

<br/><br/><br/>

> <h3 id="使用:branch、:tag或:commit来锁定特定版本或分支">使用 :branch、:tag 或 :commit 来锁定特定版本或分支</h3>

在 Podfile 中指定 SwiftLint 使用 GitHub 上的特定仓库代码，可以使用 :git 参数指定仓库的地址，并且可以使用 :branch、:tag 或 :commit 来锁定特定版本或分支。例如：

```
platform :ios, '11.0'
use_frameworks!

# 指定 CocoaPods 的官方 Specs 仓库
source 'https://github.com/CocoaPods/Specs.git'

target 'YourApp' do
  # 使用指定的 SwiftLint 仓库和版本
  pod 'SwiftLint', :git => 'https://github.com/realm/SwiftLint.git', :tag => '0.50.0'
  
  # 使用特定分支安装 SwiftLint
  # pod 'SwiftLint', :git => 'https://github.com/realm/SwiftLint.git', :branch => 'main'
  
  # 或者，使用特定 commit 安装 SwiftLint
  # pod 'SwiftLint', :git => 'https://github.com/realm/SwiftLint.git', :commit => 'abcd1234'  
end
```

- **参数说明：**
	- :git：指定 GitHub 仓库的 URL。
	- :tag：指定一个版本标签，例如 0.50.0。
	- :branch：指定分支名，例如 main 或 develop。


<br/>

***
<br/><br/><br/>

> <h1 id="问题解决">问题解决</h1>

<br/><br/><br/>

> <h2 id="libarclite_iphoneos.a找不到">libarclite_iphoneos.a找不到</h2>

Xcode 14.3 运行项目报错`File not found libarclite_iphonesimulator.a or libarclite_iphoneos.a`

**问题描述**

升级到 Xcode14.3 后编译报错：

真机运行

```
ld: file not found: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/arc/libarclite_iphoneos.a
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

模拟器运行：

```
ld: file not found: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/arc/libarclite_iphonesimulator.a
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```


<br/><br/>

> <h3 id=""></h3>


**方案一**

下载**`arc`**文件夹或者从旧版本 Xcode 复制并将 arc文件夹 拷贝到

```
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/
```

![ios0.0.58.png](./../Pictures/ios0.0.58.png)


我当时没有这么做，我是直接从网上下载的`‌libarclite_iphoneos.a`文件，因为我在真机运行的时候出现的这个错误。当时我在上述的路径中没有找到**arc**文件夹，就自己创建了一个，将上述文件放在里面就好了。😂哈哈

<br/><br/>

> <h3 id=""></h3>

**方案二**

还有就是直接在Podfile加入，如下脚本命令：

```
post_install do |installer|
  installer.generated_projects.each do |project|
    project.targets.each do |target|
      target.build_configurations.each do |config|
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
      end
    end
  end
end
```

完整如下图：

![ios0.0.59.png](./../../Pictures/ios0.0.59.png)

是可以的。

<br/><br/><br/>

> <h4 id="post_install钩子代码理解">post_install钩子代码理解</h4>

上面的一段代码是post——install的一段钩子，用于在 CocoaPods 安装完所有依赖后执行额外的配置。在这里，它将每个生成的项目中的所有目标（target）的 IPHONEOS_DEPLOYMENT_TARGET 版本设置为 13.0。下面逐步解析每一部分的作用：


```ruby
post_install do |installer|
  ...
end
```

- **`post_install`**：这是 CocoaPods 的一个钩子（hook），允许你在 `pod install` 执行完成后运行自定义脚本。`installer` 参数表示当前安装的上下文对象。
- **`installer`**：包含 CocoaPods 安装过程中生成的所有信息，包括项目和目标的配置等，便于进一步调整配置。

<br/> <br/>

```ruby
installer.generated_projects.each do |project|
  ...
end
```

- **`installer.generated_projects`**：指的是所有由 CocoaPods 生成的 Xcode 项目文件。这些文件包含你的项目依赖库的相关配置和构建信息。
- **`each do |project|`**：遍历每一个生成的项目（`project`）。在 `post_install` 钩子中，可以对每个项目进行配置。

<br/><br/>

```ruby
project.targets.each do |target|
  ...
end
```

- **`project.targets`**：指当前项目的所有目标（targets）。每个 CocoaPods 依赖项会被配置成一个目标，你可以在这些目标上应用构建设置。
- **`each do |target|`**：遍历每一个 `target`，允许你进一步操作每个目标的配置。

<br/><br/>

```ruby
target.build_configurations.each do |config|
  ...
end
```

- **`target.build_configurations`**：指目标的所有构建配置（如 `Debug` 和 `Release` 配置）。
- **`each do |config|`**：遍历每个 `build_configuration`，以便对每个构建配置（例如 `Debug` 和 `Release`）应用相同的设置。

```ruby
config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
```

- **`config.build_settings['IPHONEOS_DEPLOYMENT_TARGET']`**：访问构建配置的 `build_settings` 字典，并设置 `IPHONEOS_DEPLOYMENT_TARGET` 的值。这个值用于指定 iOS 部署目标版本，在这里设置为 `13.0`。
- **效果**：确保所有项目中的依赖库都将最低支持版本设置为 iOS 13.0，这样可以避免 CocoaPods 默认的最低版本配置不符合项目要求的问题。

<br/><br/>

**代码总结**

这段 `post_install` 钩子代码的作用是强制所有生成的 CocoaPods 依赖库的 `IPHONEOS_DEPLOYMENT_TARGET` 版本设置为 iOS 13.0，确保它们符合项目的最低部署目标。


<br/><br/>


> <h2 id=".bashrc和和.zshrc区别">.bashrc和和.zshrc区别</h1>

`.bashrc` 和 `.zshrc` 是两个不同的配置文件，用于不同的命令行解释器（shell）。具体区别如下：

- **1.解释器类型**
	- **`.bashrc`**：这是 Bash（Bourne Again SHell）的配置文件，主要用于交互式非登录 shell。它通常用于设置环境变量、别名、函数等。
	- **`.zshrc`**：这是 Zsh（Z shell）的配置文件。Zsh 是一个功能更强大的 shell，相较于 Bash 提供了更多的功能和灵活性。`.zshrc` 也用于交互式非登录 shell。

<br/>

- **2.使用场景**
	- 如果你在终端中使用 Bash 作为默认 shell，应该编辑 `.bashrc` 文件。
	- 如果你使用的是 Zsh，应该编辑 `.zshrc` 文件。

<br/>

- **3.在 macOS 上的默认 shell**
	- 从 macOS Catalina（10.15）开始，Zsh 成为默认的 shell，因此，如果你没有特别更改，应该配置 `.zshrc`。
	- 在较早的版本中，Bash 是默认的 shell。

<br/>

- **4.文件位置**
这两个文件通常位于用户的主目录中（`~`），并且都是隐藏文件。

<br/>

- **总结**
	- 使用 `.bashrc` 时，适用于 Bash shell。
	- 使用 `.zshrc` 时，适用于 Zsh shell，且在现代 macOS 版本中，Zsh 是默认 shell。



<br/>

***
<br/><br/><br/>

> <h1 id="使用">使用</h1>

<br/>

> <h2 id="工作基本指令">工作基本指令</h2>


定位到工程目录文件，在Mac终端输入：

```
pod init
```

然后再次在终端输入：

```
open -e Podfile
```

打开Podfile文件，输入要导入第三方类库(包含版本号)


<br/>

**下载第三方类库**

```
pod install
```


**升级SDK版本**

```
pod update
```

**如果出现搜索不到最新库，可以执行下面代码更新索引库**

```
pod repo update
```
更新一下。


<br/><br/><br/>

> <h2 id ='podupdate和podinstall'>pod update 和 pod install</h2>

**`update 和 install 区别：`**

- update:

- install:


<br/>

**`update 失败解决：`**

<br/>

终端输入：

```
pod update 
```

<br/>
错误提示，如下图：
<br/>

![无法更新，错误提示](https://upload-images.jianshu.io/upload_images/2959789-9df1e959b994c491.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`解决方法`**

<br/>

终端输入：

```
//pod repo update命名是用来更新本地cocoapods的spec资源配置信息
//安装完cocoapods后，在用户根目录下有个隐藏文件夹，/Users/harleyhuang[用户名]/.cocoapods，里面是cocoapods收录的所有库的配置信息；
// /Users/harleyhuang[用户名]/.cocoapods/repos/master/Specs/；
// 比如Kingfisher就是/Users/harleyhuang[用户名]/.cocoapods/repos/master/Specs/a/a/6/Kingfisher，内部分版本包含多个文件夹，每个文件夹内包含一个配置文件，比如Kingfisher.podspec.json

 pod repo update --verbose
```

<br/>

如：Kingfisher.podspec.json 所在文件夹路径图：

![.cocoapods 下的隐藏文件](https://upload-images.jianshu.io/upload_images/2959789-bb6ac0921fb5330c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Kingfisher 的5.13.1 版本](https://upload-images.jianshu.io/upload_images/2959789-70c00e4f79dc1239.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![更新整个.cocoapods下的所有库](https://upload-images.jianshu.io/upload_images/2959789-ea752f45f3805eca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后再次在终端输入：`pod update`即可。


<br/><br/>

- `pod deintegrate`

方法很简单：

1；安装cocoapods-deintegrate命令: sudo gem install cocoapods-deintegrate

2；然后到工程目录下面执行命令：pod deintegrate，就可以了，然后手动删除.xcworkspace，libPods.a，Podfile，Podfile.lock文件就好了。如果想要重装的话保留Podfile，再执行命令：pod install 就好了，很简单。



<br/><br/><br/>


> <h2 id='下载库失败：errorRPC'>下载库失败：error: RPC failed; curl 18 transfer closed with outstanding read data remaining</h2>

**安装Firebase的子模块：**

```
pod 'Firebase/Crashlytics', '10.25.0'
```

在终端下载的时候出错，如下所示：

```
Installing Firebase 10.25.0 (was 9.1.0)

[!] Error installing Firebase
[!] /usr/bin/git clone https://github.com/firebase/firebase-ios-sdk.git /var/folders/2z/dxhnl1vd6jzdg_70q_2h00bh0000gn/T/d20240508-52861-n43mvb --template= --single-branch --depth 1 --branch CocoaPods-10.25.0

Cloning into '/var/folders/2z/dxhnl1vd6jzdg_70q_2h00bh0000gn/T/d20240508-52861-n43mvb'...
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
error: 4907 bytes of body are still expected
fetch-pack: unexpected disconnect while reading sideband packet
fatal: early EOF
fatal: fetch-pack: invalid index-pack output
```


<br/> <br/>

 **出现以上错误有以下原因:**

- **1.缓存区溢出curl的postBuffer的默认值太小，需要增加缓存**

使用git命令增大缓存（单位是b，524288000B也就500M左右）

```
git config --global http.postBuffer 524288000
```

使用`git config --list`查看是否生效

此时重新克隆即可

<br/>

**2.🛜网络下载速度缓慢**

修改下载速度

```
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

<br/>

**3.以上两种方式依旧无法clone下，尝试以浅层clone，然后更新远程库到本地**

```
git clone --depth=1 http://xxx.git
git fetch --unshallow
```






<br/><br/><br/>

> <h2 id='脚本命令'>脚本命令</h2>

<br/><br/><br/>

> <h2 id=''>拷贝隐私清单文件到指定Pod下的库文件内</h2>

在Podfile文件夹中写下如下命令:

```
post_install do |installer|
  flutter_post_install(installer)
  
  # 获取 Pods 项目文件的路径所在文件夹
   pods_project_path = File.dirname(installer.pods_project.path);
   
   #隐私清单原有文件路径
   Qiniu_privacy_path = './ThirdPodPrivacy/Qiniu/PrivacyInfo.xcprivacy'
   #目标Qiniu库的Resources文件夹路径
   Qiniu_privacy_target_path = pods_project_path + '/Qiniu/Resources/'
   
   #根据Qiniu_privacy_target_path文件路径创建Qiniu库下的Resources问件夹
   FileUtils.mkdir_p(Qiniu_privacy_target_path)
   #将Qiniu_privacy_path下的文件复制到Qiniu_privacy_target_path路径下
   FileUtils.cp_r(Qiniu_privacy_path, Qiniu_privacy_target_path)
   #打印 Pods 项目路径到控制台,File.dirname获取当前文件所在的文件夹
   #puts "🍎 pods_project_path path: #{File.dirname(pods_project_path)}"
   
    
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'NO'
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '11.0'
      config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
      config.build_settings['CLANG_WARN_QUOTED_INCLUDE_IN_FRAMEWORK_HEADER'] = "NO"
    end
  end
end
```

然后在终端执行:

```
pod install
```




<br/>

***
<br/><br/><br/>

># <h1 id = "RVM安装">RVM安装</h1>

**`条件：安装CocoaPosds[当前Mac OS X 10.15]`**

**`①是否安装了RVM，若没有需安装，否则第②步`**

什么是RVM?

&emsp;`Ruby Version Manager`简称`RVM`,是一款非常好用的`ruby`版本管理以及安装工具

**①查看是否安装RVM**

```
rvm -v

//没有安装rvm
zsh: command not found: rvm 
```

**②安装RVM**

```
curl -L https://get.rvm.io | bash -s stable
```

<br/>

检查RVM是否安装上了

```
rvm -v

//没有安装上
zsh: command not found: rvm
```

这是因为没有从终端载入，这时可以这么做：

```
source ~/.rvm/scripts/rvm 
```

然后检查RVM的版本：



```
rvm -v

rvm 1.29.9 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```
这时说媒RVM已经安装上了，这时可以进行Ruby的安装了。

<br/><br/>

&emsp; 但有时像上述进行安装RVM可能一直失败，所有我们需要换另一种方式进行下载`离线下载`[官网离线下载](https://rvm.io/rvm/offline)，如下面：

```
// 离线包
$ curl -sSL https://github.com/rvm/rvm/tarball/stable -o rvm-stable.tar.gz

// 创建文件夹
$ mkdir rvm && cd rvm

// 解包
$ tar --strip-components=1 -xzf ../rvm-stable.tar.gz

// 安装 
$ ./install --auto-dotfiles

// 加载
$ source ~/.rvm/scripts/rvm

```



<br/>

***
<br/><br/><br/>

> <h2 id='安装Ruby'>安装Ruby</h2>

**②安装Ruby**

<br/>

**`更换镜像源`**

-  ruby 默认的原地址是国外网络地址，通过下面命令查看当前的镜像:


```
gem sources -l
*** CURRENT SOURCES ***

https://rubygems.org/
```

<br/>

-  移除当前镜像

```
gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
```


<br/>

-  添加国内的 ruby 镜像

```
gem sources -a https://gems.ruby-china.com

https://gems.ruby-china.com added to sources
```

<br/>

-  再次查看当前镜像,发现已经替换成功

```
gem sources -l
*** CURRENT SOURCES ***

https://gems.ruby-china.com/
```


<br/><br/><br/>

> <h2 id='安装最新ruby'>安装最新ruby</h2>


**`安装最新ruby`**

-  查看Ruby已有的版本，安装最新的：

```
rvm list known
```

![Ruby的版本](https://upload-images.jianshu.io/upload_images/2959789-9c334a8624c135e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  选择最新的版本进行安装,安装可能需要一点时间：

`rvm install 2.7`

&emsp;  在漫长的下载，编译过程，完成以后，Ruby, Ruby Gems 就安装好了。这期间若`Honebrew`没有安装，则在此过程中会进行自动安装。

![安装HomeBrew 失败](https://upload-images.jianshu.io/upload_images/2959789-1365b1d4b7c13293.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  即使`HomeBrew `没有安装好也可以安装CocoaPods(这是错的，必须安装否则无法安装Ruby)，检查Ruby是否安装好了(查看自己Mac的ruby源)：

``` 
ruby -v

ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
```

-	传说 CocoaPods 支持的 ruby 最低版本是2.2.2，如果自己电脑版本低于这个版本就升级 ruby ，上面显示我的电脑版本不需要升级，可以忽略下面的升级操作
`sudo gem update --system`


<br/> <br/>

**`查询已经安装的ruby`**

```
rvm list

//提示rvm rubies还没有安装
# No rvm rubies installed yet. Try 'rvm help install'.
```

按他的提示，在终端输入：

```
rvm help install
```


在输出的内容中找到下图的版本号，然后找到指定的版本号进行安装

![指定版本号](https://upload-images.jianshu.io/upload_images/2959789-65d49892477c169d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是上图的版本号有点旧，我们可以用命令：

```
rvm list known
```

<br/>

查看版本号，然后选中一个版本号进行安装(这里我选择2.6.3版本)：

```
rvm install 2.6.3 --default
```

会报下面的错误

![报错](https://upload-images.jianshu.io/upload_images/2959789-aa1b863c9dd52bd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

若出现如上问题说明Mac上没有安装Homebrew(若是安装了，这一步可以直接跳过) ,需要先安装：

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

在这里需要等待大概要2小时，可以看会剧比如：`海贼王、庆余年等`，安装成功后提示:

![安装成功提示](https://upload-images.jianshu.io/upload_images/2959789-2368f35b3b34d5d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/><br/>

在终端输入：

```
brew help
```

终端提示

![提示](https://upload-images.jianshu.io/upload_images/2959789-d09d056de35bc769.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/><br/>

强制退出终端，然后重新打开终端，输入：

```
rvm install 2.6.3
```

<br/>

安装后，提示

```
Install of ruby-2.6.3 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
```

终端输入：

```
rvm docs generate-ri


RVM version 1.29.9-next (master) is installed, yet version 1.29.9 (latest) is loaded.

Please open a new shell or run one of the following commands:

    rvm reload
    echo rvm_auto_reload_flag=1 >> ~/.rvmrc # OR for auto reload with msg
    echo rvm_auto_reload_flag=2 >> ~/.rvmrc # OR for silent auto reload
```

按照提示重新打开一个Shell或者输入命令，这里输入命令：

```
rvm reload


RVM reloaded!
```

更新Ruby版本

```
rvm install 2.7.0-preview1
```



<br/><br/><br/>

> <h2 id="移除OpenSSL3">移除OpenSSL3</h2>

由于新版的brew默认安装OpenSSL3，导致Ruby无法安装，因此需要移除：

```
brew uninstall --ignore-dependencies openssl@3
```





<br/>

***
<br/><br/><br/>

># <h1 id = "CocoaPosds">CocoaPosds</h1>

<br/>

># <h2 id = "安装">安装</h2>

-  根据系统版本选择指令

Mac为 OS X 10.11之前系统的安装cocoapods 指令

` sudo gem install cocoapods`

Mac为 OS X 10.11以后系统的安装cocoapods 指令

`sudo gem install -n /usr/local/bin cocoapods `

由于我的系统版本是 OS X 10.15，所以选择：

`sudo gem install -n /usr/local/bin cocoapods`

&emsp;  到了这里就成功安装CocoaPods了!如果你这样想，我只能说你太年轻了，后面的坑还多着呢！！！

- 安装本地库

```
pod setup
```

<br/>

- 查看 Cocoapods 版本

```
pod --version


1.9.0.beta.3
```


<br/><br/>

-  **对安装后的CocoaPods进行测试：**

`pod search RxSwift` 或者 `pod install`,会出现：

![pod search 失败提示](https://upload-images.jianshu.io/upload_images/2959789-7a3f49197f6a92cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![pod install 失败提示](https://upload-images.jianshu.io/upload_images/2959789-88b40a784489d24b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/> <br/>

-  **查看repo**

`pod repo list`

```
master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/Riber/.cocoapods/repos/master

trunk
- Type: CDN
- URL:  https://cdn.cocoapods.org/
- Path: /Users/Riber/.cocoapods/repos/trunk

2 repos
```

会发现有**`2 repo`**，针对于上面的 CDN 错误，我们要删除一个。更新使用`pod repo update `

-  解决[!]CDN: 错误:

在 podfile文件中添加source源:
`source 'https://github.com/CocoaPods/Specs.git'`

<br/> <br/>

&emsp;  podfile文件中添加source源后，pod install和pod update可以正常操作，但是pod search有些库却不正常,此时可以在终端执行:
`pod repo remove trunk`

&emsp;  Trunk 是用来自己写了一些类库上传到CocoaPods，但是这里我暂时用不到所以用不到，说以删除了。若是以后用到，可以自己重新添加进行配置Trunk。要重新创建请看[这里](https://www.jianshu.com/p/2572935ee006)


<br/><br/>

> <h2 id='检测pod是否安装成功'>检测pod是否安装成功</h2>


- **再次检测是否成功，终端输入**

```
pod search AFNetworking
```

<br/><br/><br/>

> <h2 id='出错:Unabletoxxx'>出错:Unable to xxx</h2>

报错：

```
[!] Unable to find a pod with name, author, summary, or description matching `AFNetworking`
```

<br/><br/>


-  不幸的是又失败了，使出撒手锏

```
git clone https://git.coding.net/CocoaPods/Specs.git ~/.cocoapods/repos/master
```

这时候`pod serarch Snapkit` 事没问题了


<br/><br/><br/>

> <h2 id='在Intel芯片OS14.4错误:Unabletofindapodwithname'>在Intel芯片OS14.4错误:Unable to find a pod with name</h2>

```
pod repo remove master
 
cd ~/.cocoapods/repos

git clone --depth 1 https://github.com/CocoaPods/Specs.git master
 
rm ~/Library/Caches/CocoaPods/search_index.json
 
pod search MJ
```

注意上述要在良好的网络环境下才可以,否则会出错!如下:

```
[!] /usr/local/bin/git clone https://github.com/CocoaPods/Specs.git -- master

正克隆到 'master'...
错误：RPC 失败。curl 92 HTTP/2 stream 5 was not closed cleanly: CANCEL (err 8)
错误：预期仍然需要 4915 个字节的正文
fetch-pack: unexpected disconnect while reading sideband packet
致命错误：过早的文件结束符（EOF）
致命错误：fetch-pack：无效的 index-pack 输出
```

<br/>
若是出现下面的错误:

```
正克隆到 'master'...
remote: Enumerating objects: 1458219, done.
remote: Counting objects: 100% (1458219/1458219), done.
remote: Compressing objects: 100% (844228/844228), done.
错误：RPC 失败。curl 92 HTTP/2 stream 5 was not closed cleanly: CANCEL (err 8)
错误：预期仍然需要 6021 个字节的正文
fetch-pack: unexpected disconnect while reading sideband packet
致命错误：过早的文件结束符（EOF）
致命错误：fetch-pack：无效的 index-pack 输出
```

**请执行:**

```
repos % git config --global http.version HTTP/1.1
```

然后再执行:

```
pod repo remove master
 
cd ~/.cocoapods/repos

git clone --depth 1 https://github.com/CocoaPods/Specs.git master
 
rm ~/Library/Caches/CocoaPods/search_index.json
 
pod search MJ
```



<br/><br/>


-  终端输入安装出错

```
pod install
```

如下错误：

```
[!] Unable to add a source with url [https://github.com/CocoaPods/Specs.git](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%253A%252F%252Fgithub.com%252FCocoaPods%252FSpecs.git) named master. You can try adding it manually in ~/.cocoapods/repos or via pod repo add 
```

在终端依次输入然后回车：

```
rm -rf ~/.cocoapods

mkdir -p ~/.cocoapods/repos

cd ~/.cocoapods/repos

git clone https://github.com/CocoaPods/Specs.git master
```

&emsp;  这时再次使用pod install 和 pod update ，如丝滑般流畅，就这破东西整了一天，记住不要轻易升级系统和新的版本Cocoapods否则后面的坑一个接着一个来。


<br/><br/><br/>



>## <h2 id="M1安装">[M1安装](https://www.jianshu.com/p/66eefaaada7a)</h2>

&emsp; 因为苹果的M1芯片与英特尔的芯片不同导致安装时也与以前不同导致出现各种的坑，现在开始M1芯片的Cocoapods安装。


<br/>

- **安装 Homebrew 过程**

去[Homebrew官网复制下载命令](https://brew.sh/index_zh-cn)终端输入：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

运行安装失败后的提示：

![安装失败](./../Pictures/tool_pod3.png)

<br/>

选择国内的镜像下载地址进行下载(最好是国外的,国内的容易出问题)：

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

![国内源地址选择](./../Pictures/tool_pod4.png)


选择一个地址后输入 y 开始执行脚本，然后输入你的开机密码，后面会下载很多文件

![下载效果](./../Pictures/tool_pod5.png)


不需要你操作，静静的等待安装完就行了，安装完成后的界面如下

![下载完成](./../Pictures/tool_pod6.png)


<br/>

重启终端后输入命令 `brew -v` 即可，如下图所示:

![](./../Pictures/tool_pod7.png)

更新 Homebrew

```
brew update
```

<br/>

卸载 Homebrew

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
```



<br/><br/><br/>

在终端输入：

```
/usr/bin/ruby -e "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install)"
```

&emsp; 若你的电脑系统是macOS big sur 12.3系统会提示你命令已废弃请使用

```
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
```

&emsp; 激动人心的时刻到了，终于下载了！但是要注意，在下载完后要注意 **Warning** 否则会导致意想不到的问题，一定按照其 **Warning** 提示进行操作。

<br/> <br/>

&emsp; 执行环境变量，添加 **Homebrew** 到 **/Users/zhoujiebing/.zprofile**文件的**PATH**中，执行下面命令即可:

```
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/zhoujiebing/.zprofile
 
eval "$(/opt/homebrew/bin/brew shellenv)"
```

&emsp; 如果你不确定可以先试试命令brew help ,如果提示没有此命令,那就需要执行上面的两行命令了.执行了之后再次输入brew help 就会列出帮助信息了,那就说明你的homebrew安装好了,可以用了!


<br/><br/><br/>

> <h2 id='安装ruby'>安装ruby</h2>

```
rvm install 3.0.0
```


<br/><br/><br/><br/>

> <h2 id='问题1:CFLAGS=-O3'>问题1:CFLAGS=-O3</h2>


- **若出现问题1,没有Skip**

```
Error running ' CFLAGS=-O3 *****(一堆路径)

please read /Users/harleyhuang/.rvm/log/1653793499_ruby-3.0.0/configure.log
There has been an error while running configure. Halting the installation.
```

解决:

```
//安装openssl
brew install openssl@1.1

//查看安装路径
brew --prefix openssl@1.1
#路径:/opt/homebrew/opt/openssl@1.1

//安装ruby
rvm install 3.0.0 --with-openssl-dir=/opt/homebrew/opt/openssl@1.1

//有时它会提示已经安装,可以用下面的命令
rvm reinstall 3.0.0 --with-openssl-dir=`brew --prefix openssl`
```


<br/><br/><br/>

> <h2 id='问题2:__rvm_make-j8'>问题2:__rvm_make -j8</h2>


- **若出现问题2,没有Skip**


```
Error running '__rvm_make -j8',
please read /Users/harleyhuang/.rvm/log/1623324364_ruby-2.5.0/make.log

There has been an error while running make. Halting the installation.

```

&emsp; 在网上搜索`Error running '__rvm_make -j8'`,发现寥寥无几或者根本没有搜到.那就只能复制这个地址`/Users/harleyhuang/.rvm/log/1623324364_ruby-2.5.0/make.log`然后到Finder中Command+F+G搜索这个文件.打开这个文件后发现经常出现这几个问题:

```
warning: no debug symbols in executable (-arch x86_64)


readline.c:1904:37: error: use of undeclared identifier 'username_completion_function'; did you mean 'rl_username_completion_function'?
                                    rl_username_completion_function);
                                    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                    rl_username_completion_function
readline.c:79:42: note: expanded from macro 'rl_username_completion_function'
# define rl_username_completion_function username_completion_function
                                         ^
/opt/homebrew/opt/readline/include/readline/readline.h:485:14: note: 'rl_username_completion_function' declared here
extern char *rl_username_completion_function PARAMS((const char *, int));
             ^
compiling psych_parser.c
1 error generated.
```


然后想到可能是M1用的arm架构导致的,所以可以在命令前加入**`arch -arm64`**记忆区别:


```
brew install openssl

rvm install 2.5.0 --with-openssl-dir=`brew --prefix openssl`

//或者下面安装-x86_64架构的rvm
arch -x86_64 rvm install 3.0.0 --with-openssl-dir=/usr/local/opt/openssl@1.0

// M1芯片的64架构:
// arch -arm64 rvm install 3.0.0 --with-openssl-dir=`brew --prefix openssl`

```

然后就下载成功了!

<br/>


```
//检查安装的的ruby版本
rvm list 

//检查ruby源 
gem sources -l

//移除不需要的ruby源
gem sources --remove https://rubygems.org/

//添加ruby源
gem sources --add https://gems.ruby-china.com

//指定系统默认的ruby版本
rvm use 3.0.0 --default 
```


出现这个问题：`Requirements installation failed with status: 1.`

<br/>

解决：

```
rvm autolibs read-only

//安装指定版本
rvm install ruby-3.0.0

//检查安装的的ruby版本
rvm list 

//指定系统默认的ruby版本
rvm use 3.0.0 --default 


//查看下载的的ruby版本号
ruby -v
```


<br/><br/><br/>

> <h2 id='问题2.1:Intel系列OS14.4——__rvm_make-j8'>问题2.1:Intel系列OS14.4——__rvm_make-j8</h2>

执行完下面命令:

```
 brew --prefix openssl@3 
 
 rvm install ruby-3.0.0 --with-openssl-dir=/usr/local/opt/openssl@3 
```

然后会出现如下终端提示错误:

```
Error running '__rvm_make -j8',
please read /Users/dwd/.rvm/log/1713417672_ruby-3.0.0/make.log

There has been an error while running make. Halting the installation.
```


然后后面搜索这个问题`x86_64 OS14.4 Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date. Requirements installation successful. 解决 `找到了github的一个解决[RVM - Error running '__rvm_make -j8' while running rvm install 3.2.2 on MacOs #5367](https://github.com/rvm/rvm/issues/5365)的解决方法:

```
brew uninstall --ignore-dependencies openssl

brew reinstall openssl@3

rvm install ruby-3.2.2 --reconfigure --enable-yjit --with-openssl-dir=$(brew --prefix openssl@3)
```

但是还是会报错,如下:

![tool.0.0.3.jpg](./../../Pictures/tool.0.0.3.jpg)

但是将这个命令:

```
rvm install ruby-3.2.2 --reconfigure --enable-yjit --with-openssl-dir=$(brew --prefix openssl@3)
```

**替换成:**

```
rvm install 3.2.2 -C --with-openssl-dir=/opt/homebrew/opt/openssl@3.1
```

就可以了.







<br/><br/><br/>

> <h2 id='安装cocoapods(macOS10.11之后使用)'>安装cocoapods(macOS10.11之后使用)</h2>


```
// cocoapods 安装
sudo gem install -n /usr/local/bin cocoapods

// 设置
pod setup

// 测试
pod search snapkit
```

有出错了!:

```
### Error LoadError - dlopen(/Users/harleyhuang/.rvm/rubies/ruby-3.0.0/lib/ruby/gems/3.0.0/gems/ffi-1.15.5/lib/ffi_c.bundle, 9): no suitable image found.  Did find:
	/Users/harleyhuang/.rvm/rubies/ruby-3.0.0/lib/ruby/gems/3.0.0/gems/ffi-1.15.5/lib/ffi_c.bundle: mach-o, but wrong architecture
	/Users/harleyhuang/.rvm/rubies/ruby-3.0.0/lib/ruby/gems/3.0.0/gems/ffi-1.15.5/lib/ffi_c.bundle: mach-o, but wrong architecture - /Users/harleyhuang/.rvm/rubies/ruby-3.0.0/lib/ruby/gems/3.0.0/gems/ffi-1.15.5/lib/ffi_c.bundle
<internal:/Users/harleyhuang/.rvm/rubies/ruby-3.0.0/lib/ruby/3.0.0/rubygems/core_ext/kernel_require.rb>:85:in 'require'
......
.....
...

Don't forget to anonymize any private data!

Looking for related issues on cocoapods/cocoapods...
Searching for inspections failed: undefined method `map' for nil:NilClass
```

然后进行了一番大搜索,发现又是架构的锅,日了狗了!命令撸起来:

```
sudo arch -arm64 gem install ffi
 
sudo arch -arm64 gem install cocoapods

//测试
pod search afnetworking     
```

坎坷又来了:!

`[!] CDN: trunk URL couldn't be downloaded: https://cdn.cocoapods.org/all_pods_versions_5_7_d.txt Response: Couldn't connect to server`

```
//在本地Podfile替代
source 'https://cdn.cocoapods.org/'
//用:
source 'https://github.com/CocoaPods/Specs.git'


//在本地移除久的引用,用新的
pod repo remove trunk

pod install --repo-update
```



<br/>

***
<br/><br/>

># <h2 id = "错误解决方案">错误解决方案</h2>


<br/>

- 错误：

```
/usr/local/lib/ruby/gems/2.7.0/gems/cocoapods-core-1.9.1/lib/cocoapods-core/source/metadata.rb:15:in `initialize': undefined method `with_indifferent_access' for false:FalseClass (NoMethodError)
```

终端输入：

```
$ sudo gem update cocoapods 
$ rm -rf ~/.cocoapods/repos/trunk/
```


<br/>

-  repos 为 0

```
pod repo list

0 repos
```

- a. 先移除掉本地的master,在终端输入pod repo remove master;

- b. 再切换到：`cd ~/.cocoapods/repos`;

- c. `git clone --depth 1 https://github.com/CocoaPods/Specs.git master`,等待下载完毕；

- d. `rm ~/Library/Caches/CocoaPods/search_index.json`

- e. `pod search afnetworking`, 检测是否可以使用了；

<br/><br/>

[解决方案参考]([https://www.cnblogs.com/shuilangyizu/p/10935728.html](https://www.cnblogs.com/shuilangyizu/p/10935728.html)
)



<br/><br/><br/>


- rvm系统路径出错

问题：电脑重启后，pod无法使用，会报错。需要每次在系统终端`rvm reload`后pod才能使用，否则pod无法使用。

![<br/>](./../Pictures/tool_pod1.png)

如上图，执行【步骤1】就会出现问题，执行【步骤2】出现【步骤3】警告，解决方案：

执行【步骤4】，注意这个命令需要输入**5次**

让后执行【步骤5】没有警告了，然后电脑再重启，没有任何问题。



<br/>

![tool_pod2.png](./../Pictures/tool_pod2.png)

在上图中执行`rvm -h`来查看rvm指令，上图只是截取的一部分。





<br/>

***
<br/><br/><br/>



> <h2 id = "卸载CocoaPosds">卸载CocoaPosds</h2>

**`第一种卸载方法：`**

**①卸载老版本**

`sudo gem uninstall cocoapods`

出现

```
Remove executables:
	pod, sandbox-pod

in addition to the gem? [Yn]  Yn          //输入 Yn，确定删除    
Removing pod
Removing sandbox-pod
Successfully uninstalled cocoapods-1.8.4

```

<br/>

**②查看本地安装过的cocopods相关东西**

` gem list --local | grep cocoapods`

```
cocoapods-core (1.8.4)
cocoapods-deintegrate (1.0.4)
cocoapods-downloader (1.3.0)
cocoapods-plugins (1.0.0)
cocoapods-search (1.0.0)
cocoapods-stats (1.1.0)
cocoapods-trunk (1.4.1)
cocoapods-try (1.1.0)
```

使用命令逐个删除：

```
sudo gem uninstall cocoapods-core
//Successfully uninstalled cocoapods-core-1.8.4


sudo gem uninstall cocoapods-deintegrate
//Successfully uninstalled cocoapods-deintegrate-1.0.4

sudo gem uninstall cocoapods-downloader 
//Successfully uninstalled cocoapods-downloader-1.3.0

.....
...
..
```

或者使用脚本命令进行全部的删除：

```
sudo rm -rf /usr/local/bin/pod ; gem list | grep cocoapods | awk '{print $1}' | while read line; do sudo gem uninstall $line;done
```

如下图：

![CocoaPods相关的东西](https://upload-images.jianshu.io/upload_images/2959789-2b4d3d8859ca5e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/> <br/>

**`第二种卸载方法：`**

**①查找目前版本的pod路径**

```
which pod
```

**②移除现有pod**

```
rm -rf /usr/local/bin/pod
```


&emsp; 这样CocoaPods 就卸载干净了。


<br/><br/>


<br/>

***

<br/><br/><br/>

> <h1 id="pynev:python的版本管理工具">pynev:python的版本管理工具</h1>

**安装**

```
brew install pyenv
```


<br/>


**根据自身环境，将下方内容加到对应文件中： [.bashrc / .zshrc](#.bashrc和和.zshrc区别)**

**.bashrc的配置到 ~/.bash_profile文件中，然后记得 source ~/.bash_profile**

这里只示范一个。打开`open ~/.zshrc`,将下面的配置写入

```
export PYENV_ROOT=/usr/local/var/pyenv
export PATH="$PYENV_ROOT/bin:$PATH"
export PATH="$PYENV_ROOT/shims:$PATH"
#导出python的环境变量。否则python命令找不到
export PATH=/Users/ganghuang/.pyenv/versions/3.9.9/bin:$PATH
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
```

<br/>

**验证是否安装成功**

```
pyenv
```

![tool.0.0.12.png](./../Pictures/tool.0.0.12.png)



<br/>

***
<br/><br/><br/>

> <h1 id="Python安装">Python安装</h1>

[Python安装](#Python安装)

**安装**

```
pyenv install 3.9.9 //安装的python版本
```

遇到下面的问题：

```
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Downloading Python-3.9.9.tar.xz...
-> https://www.python.org/ftp/python/3.9.9/Python-3.9.9.tar.xz
Installing Python-3.9.9...
python-build: use readline from homebrew
python-build: use zlib from xcode sdk

BUILD FAILED (OS X 14.6.1 using python-build 20180424)

Inspect or clean up the working tree at /var/folders/82/356fyp3j3kq2x8b03msz5w3w0000gn/T/python-build.20241009102336.54421
Results logged to /var/folders/82/356fyp3j3kq2x8b03msz5w3w0000gn/T/python-build.20241009102336.54421.log

Last 10 log lines:
changing mode of build/scripts-3.9/idle3 from 644 to 755
changing mode of build/scripts-3.9/2to3 from 644 to 755
renaming build/scripts-3.9/pydoc3 to build/scripts-3.9/pydoc3.9
renaming build/scripts-3.9/idle3 to build/scripts-3.9/idle3.9
renaming build/scripts-3.9/2to3 to build/scripts-3.9/2to3-3.9
Creating directory /usr/local/var/pyenv/versions/3.9.9/bin
ginstall: cannot create directory ‘/usr/local/var’: Permission denied
Creating directory /usr/local/var/pyenv/versions/3.9.9/lib
ginstall: cannot create directory ‘/usr/local/var’: Permission denied
make: *** [altbininstall] Error 1
```

这里主要注意的是苹果为了安全，不允许其他软件访问不分文件。可以修改终端读取所有磁盘文件在【系统设置】->【隐私安全】里修改。

因为根据终端命令提示，是因为权限不够导致无法对部分文件夹进行读写，所以用下面命令：

```
#将该目录及其所有子目录的拥有者更改为当前用户
sudo chown -R $(whoami) /usr/local/var

sudo pyenv install 3.9.9  
```

然后

```
pyenv global 3.9.9  #设置全局版本
```

<br/>

这里要等一会，或者重启终端，否则比如会出现python命令找不到。

**查看安装目录**

```
 pyenv which python3
```

<br/>

更新默认目录（$PATH替换为上一步提到的**安装目录**， --path不需要替换）

```
export PATH="$HOME/.pyenv/bin:$PATH"

#如：export PATH="$HOME/.pyenv/bin:usr/bin/python3"


#下面2行命令要关闭终端，重新打开才行
eval "$(pyenv init --path)"

eval "$(pyenv init -v)"
```

<br/>

**关闭终端重新打开**

```
source ~/.zshrc #或者source ~/.bash_profile
```

会遇到如下错误：

```
mkdir: /usr/local/var: Permission denied
mkdir: /usr/local/var: Permission denied
```

输入如下命令：

```
# 将 /usr/local/var 及其所有子目录的拥有者更改为当前用户，$(whoami) 会动态获取当前登录用户的用户名。
#不用担心这个文件权限太宽泛了，关闭重新打开后这个文件夹下又会回复之前的权限
sudo chown -R $(whoami) /usr/local/var

chown: /usr/local/var: No such file or directory



#因为提示没有这个/usr/local/var路径下的文件夹，那我们就创建
sudo mkdir -p /usr/local/var


#创建完这个问件夹后，我们再重新对其开启访问权限
sudo chown -R $(whoami) /usr/local/var

source ~/.zshrc #就可以了
```


<br/>

**验证：**

```
#系统默认运行的python版本
which python3

/usr/bin/python3



#python关联的openssl版本
python3 -c "import ssl; print(ssl.OPENSSL_VERSION)"

LibreSSL 2.8.3

```









