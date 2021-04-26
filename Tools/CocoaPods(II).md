># 忽略文件配置

```
# .ignore不起作用解决方案
# git rm -r --cached .
# git add .
# git commit -m "update .ignore"


# iOS 忽略文件
*~
.DS_Store
*.xcworkspace
xcuserdata
*.lock
Pods



# Flutter忽略文件
# Miscellaneous
*.class
*.log
*.pyc
*.swp
.DS_Store
.atom/
.buildlog/
.history
.svn/
# IntelliJ related
*.iml
*.ipr
*.iws
.idea/
# Visual Studio Code related
.vscode/
# Flutter/Dart/Pub related
**/doc/api/
.dart_tool/
.flutter-plugins
.flutter-plugins-dependencies
.packages
.pub-cache/
.pub/
/build/
# Android related
**/android/**/gradle-wrapper.jar
**/android/.gradle
**/android/captures/
**/android/gradlew
**/android/gradlew.bat
**/android/local.properties
**/android/**/GeneratedPluginRegistrant.*
# iOS/XCode related
**/ios/**/*.mode1v3
**/ios/**/*.mode2v3
**/ios/**/*.moved-aside
**/ios/**/*.pbxuser
**/ios/**/*.perspectivev3
**/ios/**/*sync/
**/ios/**/.sconsign.dblite
**/ios/**/.tags*
**/ios/**/.vagrant/
**/ios/**/DerivedData/
**/ios/**/Icon?
**/ios/**/Pods/
**/ios/**/.symlinks/
**/ios/**/profile
**/ios/**/xcuserdata
**/ios/.generated/
**/ios/Flutter/App.framework
**/ios/Flutter/Flutter.framework
**/ios/Flutter/Generated.xcconfig
**/ios/Flutter/app.flx
**/ios/Flutter/app.zip
**/ios/Flutter/flutter_assets/
**/ios/Flutter/flutter_export_environment.sh
**/ios/ServiceDefinitions.json
**/ios/Runner/GeneratedPluginRegistrant.*
# Web related
**/web/**/lib/generated_plugin_registrant.dart
# Service account files
svc-keyfile.json
# Exceptions to above rules.
!**/ios/**/default.mode1v3
!**/ios/**/default.mode2v3
!**/ios/**/default.pbxuser
!**/ios/**/default.perspectivev3
!/packages/flutter_tools/test/data/dart_dependencies_test/**/.packages
**/ios/Flutter/.last_build_id


# Egret游戏 忽略文件
/Lobby/bin-release
/Lobby/bin-debug
/Lobby/node_modules
/Lobby/template
/Lobby/exml.e.d.ts
/Lobby/native_require.js
/.idea/
/Lobby/template/runtime/native_require.js
/Lobby/index.html

```



<br/>

***
<br/>



># pod update 和 pod install

**`update 和 install 区别：`**

- update:

- install:


<br/>

**`update 失败解决：`**

<br/>

终端输入：

<br/>

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

<br/>

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


<br/>
<br/>

- `pod deintegrate`

方法很简单：

1；安装cocoapods-deintegrate命令: sudo gem install cocoapods-deintegrate

2；然后到工程目录下面执行命令：pod deintegrate，就可以了，然后手动删除.xcworkspace，libPods.a，Podfile，Podfile.lock文件就好了。如果想要重装的话保留Podfile，再执行命令：pod install 就好了，很简单。













