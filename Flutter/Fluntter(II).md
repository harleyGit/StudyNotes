># Flutter 常用命令
```
//模拟器列表
$ flutter emulators

//启动模拟器,只有启动模拟器才可以运行模拟器
$ flutter emulators --launch <emulator id>

//启动安卓模拟器
$ flutter emulators --launch Nexus_5X_API_28
//例如未启动模拟器列表:
Nexus_5X_API_28     • Nexus 5X      • Google • Nexus 5X API 28
apple_ios_simulator • iOS Simulator • Apple

//运行所有模拟器
$ flutter run -d all


运行指定模拟器
$ flutter run -d <deviceId>
例如模拟器列表:
Android SDK built for x86 • emulator-5554             • android-x86 • Android 9 (API 28) (emulator)
xxx的 iPhone               • 00008020-001838491169002E • ios         • iOS 12.2

//运行安卓模拟器
$ flutter run -d emulator-5554

//运行IOS真机
$ flutter run -d 00008020-001838491169002E

//运行模拟器过程中命令
//热更新直接刷新
$ r

//热更新重启刷新
$ R

//退出运行模拟器
$ q

//创建一个新的Flutter项目
create

//显示相关安装工具的信息
doctor

//为当前项目运行Flutter驱动程序测试
drive

//在Fushia上进行热重载
fuchsia_reload

//在附加设备删除安装Flutter应用程序
install 

//显示用于运行Flutter应用程序的日志输出
logs

//在设备上运行Flutter应用程序
run

//从一个链接的设备截图
screenshot

//停止在附加设备上的Flutter应用程序
stop

//升级Flutter副本
upgrade

```



<br/>
***
<br/>
># 模拟器调试
&emsp;  在Visual Studio Code 中调出控制台，选择终端，使鼠标的焦点聚焦在终端上然后输入如下命令进行调试：
```
// 输入后进行热加载，也就是重新加载，修改后的东西重新同步到到模拟器上(相当于网页刷新，方便快捷)
r 

//显示网格，掌握布局情况
p

//切换android和iOS的模拟器预览模式
o

//退出调试预览模式
q
```
![模拟器的调试](https://upload-images.jianshu.io/upload_images/2959789-9f6d2e24d2d6a221.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
**`调试`**
![1.png](https://upload-images.jianshu.io/upload_images/2959789-bf6d5a28345f84af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
















