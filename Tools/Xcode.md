

- **配置**
	- 导航显示开发页面 
	- Command+点击 跳转到定义的方法
- **清理缓存文件**
- **Xcode完全卸载**
- **参考资料**




<br/>

***
<br/>

># 配置

<br/>

> 导航显示开发页面

打开`Xcode->Preferebces`,选择如下图：

![z46](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z46.png)

<br/>
<br/>

> Command+点击 跳转到定义的方法

![z47](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z47.png)


<br/>

***
<br/>

># 清理缓存文件

<br/>

-	删除Xcode中多余的证书provisioning profile 

```
手动删除： 
Xcode6 provisioning profile path： 
~/Library/MobileDevice/Provisioning Profiles
```

<br/>

-	清理Xcode编译项目产生的缓存垃圾 

```
(Xcode永久了，会产生很多项目编译缓存，占用一大堆硬盘空间，此时需要对该目录进行清理） 
手动删除： 
Xcode编译项目缓存垃圾的目录： 
~/Library/Developer/Xcode/DerivedData
```


<br/>

- 真机/模拟器运行项目非常慢的解决方法

```

删除 ~/Library/Developer/Xcode/iOS DeviceSupport/该目录下，所有文件夹;


选择Xcode --> Window-->Devices and Simulators，找到真机设备，鼠标右键选择unpair the device

然后重启Xcode、重新连接设备、重新运行应用程序

```


<br/>

- 清理运行时产生的文件

Command+shift+G 在Finder中，删除项目运行时产生的文件

```
/Users/harleyhuang/Library/Developer/Xcode/DerivedData
```
亲测，删除了3个项目文件。清了3.3G的内存

<br/>

***
<br/>


># 终端卸载

```
% sudo rm -rf /Applications/Xcode.app
Password:

% sudo rm -rf /Library/Preferences/com.apple.dt.Xcode.plist

% rm -rf ~/Library/Preferences/com.apple.dt.Xcode.plist

% rm -rf ~/Library/Caches/com.apple.dt.Xcode

% rm -rf ~/Library/Application\ Support/Xcode

% rm -rf ~/Library/Developer/Xcode

% rm -rf ~/Library/Developer/CoreSimulator

% rm -rf ~/Library/Developer/XCPGDevices

% rm -rf ~/Library/Developer

% rm -rf ~/Library/Developer/Xcode/DerivedData
```


<br/>

***
<br/>

**参考资料：**

[Xcode清理垃圾文件](https://www.jianshu.com/p/4540d34431db)
