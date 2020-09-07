># 配置

**①配置镜像环境变量**
打开**`.bash_profile`** 文件配置镜像地址,`在终端输入：
```
 vim ~/.bash_profile
```
敲击键盘上的`A`键进入编辑模式，然后在其中输入环境变量如下：
```
export PUB_HOSTED_URL=https://pub.flutter-io.cn

export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

```
输入完成后，按`ESC`退出编辑模式，输入`:wq`返回至命令行,最后在命令行输入一下命令进行文本保存：
```
source ~/.bash_profile
```
**`注意：`** 此镜像为临时镜像，并不能保证一直可用，读者可以参考详情请参考 [Flutter 中文文档](https://flutterchina.club/setup-macos/) 以获得有关镜像服务器的最新动态。


<br/>
**②获取Flutter SDK**
`第一种：获取Flutter SDK的方法`
下载Flutter SDK可用安装包，下载页最新请到[GitHub 最新安装包](https://github.com/flutter/flutter/releases)

解压安装包到你想安装的目录，如：
```
cd /Users/harleyhuang/Documents/Gitee
unzip ~/Downloads/flutter_macos_v0.5.1-beta.zip
```

使用上述命令进行解压 `zip`的包时会报错，如下图：
![无法进行解压](https://upload-images.jianshu.io/upload_images/2959789-d0bbc6a8f1006ac5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;这是由于该压缩文件由pkzip算法压缩而成，所以必须先安装p7zip包，用p7zip来解压，这里我用brew来安装p7zip的
```
 brew install p7zip
```
安装成功后运行以下命令就可以解压你想要解压的文件:
```
7za x xxxxx.zip 
```
然后下载Flutter SDK的按转包，进行解压到你想安装的目录中。

<br/>
`第二种：获取Flutter SDK的方法`
终端输入命令：
```
git clone -b beta https://github.com/flutter/flutter.git
```
这种方法需要在网络好的环境才能下载，否则会失败。在好的网络环境需要大概 **`2小时`**。

添加flutter相关工具到path中：
```
export PATH=${PATH}:/Users/harleyhuang/flutter/bin
```
如下图：
![获取flutter SDK解压安装包后的地址](https://upload-images.jianshu.io/upload_images/2959789-b293c5e6f0056878.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
复制上述的地址，然后在终端打开`.bash_profile`文件，配置flutter环境变量如下：
```
export PATH=${PATH}:/Users/harleyhuang/flutter/bin
```
完成后的3个flutter环境变量如下：
```
export PUB_HOSTED_URL=https://pub.flutter-io.cn

export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

//export PATH=/Users/harleyhuang/flutter/bin:$PATH ，或者如下命令：
export PATH=${PATH}:/Users/harleyhuang/flutter/bin
```
截图如下图：
![Mac 完成配置的3个环境变量配置](https://upload-images.jianshu.io/upload_images/2959789-a38cc0c6b9c878a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
完成后一定要输入以下命令，进行文件保存，否则再次使用flutter命令无法生效，需要输入：
```
source ~/.bash_profile
```

此时Flutter的环境变量配置也就完成，然后在终端进行检查是否配置成功，输入：
```
flutter doctor
```
然后终端显示：
```
harleyhuang@@Harley-Pro ~ % flutter doctor        
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel unknown, v1.12.13+hotfix.5, on Mac OS X 10.15.2 19C57,
    locale zh-Hans-HK)
[✗] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from:
      https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK
      components.
      (or visit https://flutter.dev/setup/#android-setup for detailed
      instructions).
      If the Android SDK has been installed to a custom location, set
      ANDROID_HOME to that location.
      You may also want to add it to your PATH environment variable.

[✓] Xcode - develop for iOS and macOS (Xcode 11.3)
[!] Android Studio (not installed)
[✓] Connected device (2 available)

! Doctor found issues in 2 categories.
harleyhuang@@Harley-Pro ~ % 

```
这些显示可以通过**`Command + S `**输出终端的显示文件内容。

若输入后有反应与[官网检查](https://flutterchina.club/setup-macos/)差不多一致，则配置成功！

**`注意：`**当你退出终端时，然后再次打开终端输入 `flutter doctor`,终端会输出如下提示：
```
zsh: command not found: flutter
```    
只有当你在终端再次输入：`source ~/.bash_profile`后然后回车，这个`flutter doctor`才起作用，不报上述的提示。

**`原因分析：`**
&emsp;  字面意思是`相关命令没有没有找到`，其实就是`bash shell `以及`zsh shell` 是两种读取系统环境变量（使用flutter的前提是你肯定已经在bash的 .bash_profile 已经配置相关`flutter`的环境变量了，从而才能使用`flutter`命令）
&emsp; 然而在使用`zsh shell`的时候，你并没有把相关的环境变量的配置设置到 `.zshrc` 中(功能上类似bash 的.bash_profile)

**`解决办法：`**
-  命令行终端输入 `open .zshrc` 回车；
- 在打开的文件中输入**`source ~/.bash_profile`**,然后使用`command+s`组合键进行保存;
- 在终端输入 **`source .zshrc`** 回车,这样就解决了。
&emsp; 这样再次退出终端，然后再次打开终端问题就解决了。

![zshrc 文件中添加 source ~/.bash_profile ](https://upload-images.jianshu.io/upload_images/2959789-e4e554de2818f6cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
***
<br/>
># 安装Visual Studio Code

<br/>
根据大神提示，编码Flutter最好的工具是**`Visual Studio Code`**其次是**`Android Studio`**，其下载地址是：[Visual Studio Code 安装包](https://code.visualstudio.com/docs/?dv=osx)

下载好后需要下载两个插件，分别是**`Dart`**和**`Flutter`**,如下图：
**`1⃣️Flutter 插件下载`**
![Flutter 插件截图](https://upload-images.jianshu.io/upload_images/2959789-3be0a07175c8e460.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
**`2⃣️ Dart 插件下载`**
![Dart 插件](https://upload-images.jianshu.io/upload_images/2959789-6e52d6ec9413e5b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
**`3⃣️Android iOS Emulator插件下载`**
![模拟器下载](https://upload-images.jianshu.io/upload_images/2959789-ed27e1d517c49933.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开控制面板
![按图中手动或者 command + shif + p组合键进行打开](https://upload-images.jianshu.io/upload_images/2959789-90bb70f8825162df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
下载好后，重启一下**`Visual Studio Code`** 就好了

<br/>
创建一个Flutter工程：**`Command + shift + P`**,如下图：
![项目名称一律小写](https://upload-images.jianshu.io/upload_images/2959789-434ef6420b066c3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
**`Visual Studio Code`** 调出模拟器进行调试，如下图：
![Debug 模式调试](https://upload-images.jianshu.io/upload_images/2959789-76ca11764f04a02a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>
># Android 的配置
**`1⃣️ 配置 JDK, 使用brew下载`**

**`2⃣️ 下载 android sdk, 使用brew下载`**




















