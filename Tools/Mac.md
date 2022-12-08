> <h1 id=''></h1>
- [**相对路径**](#相对路径)
- [**快捷键**](#快捷键)
- [**配置**](#配置)
	- [解决source保存配置](#解决source保存配置)
	- [软链接](#软链接)
- [**Xcode**](#Xcode)
	[**调试**](#调试)
	- [配置](#配置)
		- [导航显示开发页面](#导航显示开发页面)
		- [跳转到定义方法](#跳转到定义方法)
	- [清理缓存文件](#清理缓存文件)
		- [Xcode完全卸载](#Xcode完全卸载)
	- **参考资料**
		- [Yapi](https://hellosean1025.github.io/yapi/index.html)
		- [Xcode清理垃圾文件](https://www.jianshu.com/p/4540d34431db)
		- [控制台调试](https://www.jianshu.com/p/75688613c6f4)
- [**开发必备工具**](#开发必备工具)
	- [Charles使用](#Charles使用)
	- [Wireshark🦈SIP](#Wireshark🦈SIP)
	- [MacVim](#MacVim)
	- [微信小助手扩展](#微信小助手扩展)
	- [美区AppleID](#美区AppleID)
- [**下载**](#下载)
- [**iPhone纯色图片**](#纯色图片)
- [**电子产品**](#电子产品)
	- [iPad](#iPad)
- **兴趣工具**
	- [**正则表达式图观**](https://jex.im/regulex/?from_wecom=1#!flags=&re=)
	- [**Window系统下载**](https://msdn.itellyou.cn)
	- [**LookAE(影视后期特效)**](https://www.lookae.com)
	- [**美国地址生成器**](https://www.meiguodizhi.com/usa-address/hot-city-New-York)
	- [**免费FQ**](https://github.com/Alvin9999/new-pac/wiki/ss免费账号)
	- [**免费节点**](https://lncn.org)
	- [**BetaProfiles(阻止苹果系统软件更新及证书)**](https://betaprofiles.com)
	- [**AppleOpenSource(苹果系统资源下载)**](https://opensource.apple.com)
	- [**反斗限免**](http://free.apprcn.com)
	- [**邮箱账号获取(10分钟邮箱，免费邮箱账号注册获取验证码)**](https://10minutemail.net)
- [**修改管理员UserName和Password**](#修改管理员UserName和Password)
- [**Mac问题解决**](#Mac问题解决)
	- [Mac蓝牙不可用](#Mac蓝牙不可用)
	- [Mac清理内存](#Mac清理内存)
	- [可删除的文件](#可删除的文件)
	- [破解软件无法安装](#破解软件无法安装)
- [**DNS**](#DNS)
	- [DNS缓存清除](#DNS缓存清除)

<br/>

***
<br/>


> <h1 id='相对路径'>相对路径</h1>

**2种路径**
- **绝对（或完整）路径**以诸如 D: 之类的盘符开头，后跟一个冒号。
- **相对路径:** 指相对于当前目录的位置。相对路径使用两种特殊符号，单点 (.) 和双点 (..)，通过它们可以转换到当前目录或父目录。双点用于在目录等级中上移。单点表示当前目录。

<br/>

&emsp; 在以下目录结构示例中，假定您使用 Windows 资源管理器导航至 D:\Data\Shapefiles\Soils。导航到此目录之后，相对路径将使用 **D:\Data\Shapefiles\Soils 作为当前目录（如果导航到新目录，此新目录将成为当前目录）**。当前目录有时被称为根目录。‌


![tool0_0.gif](./../Pictures/tool0_0.gif)

<br/>

&emsp; 如果要从当前目录 (Soils) 浏览至 Landuse 目录，可以在 Windows 资源管理器地址框中输入以下内容：

```
..\Landuse
```


<br/>


&emsp; Windows资源管理器将浏览至 **D:\Data\Shapefiles\Landuse**。使用 **D:\Data\Shapefiles\Landuse** 作为当前目录的其他几个示例如下：

![tool0_1.png](./../Pictures/tool0_1.png)



<br/>
<br/>

***
<br/>


> <h1 id='快捷键'>快捷键</h1>

- **锁屏：** `Command+Control+Q`
- **关机终端命令：** `sudo halt` 或者 `sudo shutdown -h now`
- **重启终端命令:** `sudo reboot` 或者 `sudo shutdown -r now`
- **休眠终端命令：** `sudo shutdown -s now`
- [x] **Finder显示隐藏文件:** `Command + shift + “ . ”`


<br/>

***
<br/>



> <h1 id='配置'>配置</h1>

<h2 id='解决source保存配置'>解决source保存配置</h1>


```
vim ~/.zshrc

///输入 i 

///文件输入:
source ~/.bash_profile

///输入: esc 键

/// 输入: wq!
```



<br/>
<br/>

<h2 id='软链接'>软链接</h2>

```
// a 就是源文件，b是链接文件名,其作用是当进入b目录，实际上是链接进入了a目录
// 值得注意的是执行命令的时候,应该是a目录已经建立，目录b没有建立。我最开始操作的是也把b目录给建立了, 结果就不对了
ln -s a b


// 进入 /home/gamestat/
cd /gamestat/

// 删除软链接：
rm -rf  b  注意不是rm -rf  b/


// Mac的M1后需要软链接到 /usr/local/bin/ 这个目录里,通过brew安装不会软链到这个目录
ln -s /opt/homebrew/Cellar/doxygen/1.9.4/bin/doxygen /usr/local/bin/doxygen

//查看软链软件真实目录位置
realpath /opt/homebrew/bin/doxygen
```



<br/>

***
<br/>


># <h1 id = "Xcode">Xcode</h1>

> <h2 id='调试'>调试</h2>


- 控制台**`bt`**命令
	- 当xcode工具区打印的堆栈信息不全时，可以在控制台通过“bt”指令打印完整的堆栈信息


<br/>
<br/>
<br/>
<br/>



> <h2 id='快捷键'>快捷键</h2>

- 代码格式对齐：选中要对齐的代码，然后快捷键 `Contorl + i`
* 单行注释：`command + /`，即`⌘ + /`;
* 注释文档：`command + Option + /`，即`⌘ + ⌥ + /`;
- [**常用快捷键总结**](https://yanhooit.gitbooks.io/ios_study_note/content/xcodekuai_jie_jian.html)






<br/>
<br/>
<br/>
<br/>


> <h2 id='配置'>配置</h2>




<br/>

> <h3 id='导航显示开发页面'>导航显示开发页面</h3>

打开`Xcode->Preferebces`,选择如下图：

![z46](./../Pictures/z46.png)

<br/>
<br/>

> <h3 id='跳转到定义方法'>跳转到定义方法</h3>

Command+点击

![z47](./../Pictures/z47.png)


<br/>
<br/>
<br/>
<br/>




> <h2 id='清理缓存文件'>清理缓存文件</h2>


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
<br/>


> <h3 id='Xcode完全卸载'>Xcode完全卸载</h3>

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







<br/>

***
<br/>


># <h1 id = "下载">下载</h1>

- 安装任何应用配置

1️⃣ 打开终端；

2️⃣输入：`sudo spctl --master-disable`；

3️⃣根据提示输入管理员的密码；

4️⃣【系统偏好设置】->【安全与隐私】->【通用】->【允许从以下位置下载的应用】。

<br/>

下载后即可安装应用了。


<br/>

- **下载资源**

1⃣️ 切换到下载路径

`cd /Users/harleyhuang/Desktop`

2⃣️ 资源下载

下载一首`十年.mp3`

`curl -o 十年.mp3 ‘http://ev.sycdn.kuwo.cn/29ab24ac0a6f70c790a234422e377a29/60813d14/resource/n3/11/0/2695339732.mp3'`



<br/>

- 软件安装

使用 brew 安装软件

`brew install wget `

使用 brew 卸载软件

`brew uninstall wget `

使用 brew 查询软件

`brew search /wge*/ `

其他brew命令

```
brew list           列出已安装的软件 
brew update     更新brew 
brew home       用浏览器打开brew的官方网站 
brew info         显示软件信息 
brew deps        显示包依赖
```


<br/>

****
<br/>



> <h1 id='纯色图片'>纯色图片</h1>

-  黑色背景确认是否有漏光，七彩屏；

-  白色背景确认是否暖屏或阴阳屏；

-  浏览所有背景色确认是否有坏点，亮点；

```
![Red](https://upload-images.jianshu.io/upload_images/2959789-bd4cd7776049a9ba.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Yellow](https://upload-images.jianshu.io/upload_images/2959789-85d892d4ea3fa950.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Green](https://upload-images.jianshu.io/upload_images/2959789-91a358310a836c5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Blue](https://upload-images.jianshu.io/upload_images/2959789-7d2e465a30daad06.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Black](https://upload-images.jianshu.io/upload_images/2959789-e7e43b1f1342f19d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![White](https://upload-images.jianshu.io/upload_images/2959789-9aa4d49da9b0330a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```


<br/>

****
<br/>


> <h1 id='电子产品'>电子产品</h1>

<br/>

> <h2 id='iPad'>iPad</h2>


| iPad型号 | 发布时间 | 屏幕尺寸 | 芯片性能 | CPU | GPU | 电池 | 网络 | 功能 |
|:--|:--|:--|:--|:--|:--|:--|:--|
| iPad | 2010年1月27日 |  |  |   |  |  |  |  |




<br/>

***
<br/>


># <h1 id='开发必备工具'>[开发必备工具](https://juejin.cn/post/7088473126996181028)</h1>



<br/>

>## <h2 id='Charles使用'>[Charles 使用](https://www.jianshu.com/p/633ac6221028)</h2>
>


<br/>
<br/>
<br/>


># <h2 id='Wireshark🦈SIP'>Wireshark🦈SIP</h2>

- [Wireshark使用教程](https://www.cnblogs.com/hls-code/p/16054209.html)
- [抓取真机数据](https://blog.csdn.net/yulianlin/article/details/79095413)



<br/>
<br/>

># <h2 id='MacVim'>[MacVim](https://www.jianshu.com/p/923aec861af3)</h2>


<br/>
<br/>


># <h2 id='微信小助手扩展'>[微信小助手扩展](https://github.com/harleyGit/WeChatExtension-ForMac)</h2>

**慎重:使用后容易被封!!**

- **1.安装**

```
curl -o- -L https://omw.limingkai.cn/install.sh | bash -s

omw -n

omw -g
```


<br/>
<br/>

- **2.自动卸载**

`bash <(curl -sL https://git.io/JUO6r)`



<br/>
<br/>


># <h2 id='美区AppleID'>[美区AppleID](https://www.ifanr.com/app/1367245)</h2>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



> <h1 id=''></h1>

<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>



<br/>

> <h2 id=''></h2>


<br/>
<br/>

> <h2 id=''></h2>




<br/>

***
<br/>

># <h1 id='修改管理员UserName和Password'>修改管理员 UserName 和 Password</h1>

1️⃣重启iMac、Mac Air、Mac Pro；

2️⃣当听到电脑启动声音时，按下组合键：Command + S；

3️⃣当看到好像进入终端时，松开组合键；

4️⃣输入：`sbin/mount -uaw`;

5️⃣输入：`rm /var/db/.AppleSetupDone`,此时注意：rm 和后面的字符串一个空格；

6️⃣再次输入：reboot，对电脑进行重启即可；

7️⃣重启后，跟当初才激活电脑一样，根据激活配置，一步一步重新对其进行配置，在此过程中会要求我们重新创建一个管理员账号。


<br/>

***
<br/>


># <h1 id='Mac问题解决'>Mac问题解决</h1>



<br/>

> <h2 id='Mac蓝牙不可'>**Mac蓝牙不可用**</h2>


`①  拔出与Mac连接的设备线；  `<br/>
`② 关机;  `<br/>
`③ 同时按下shift+control+option+power,保持5秒左右；`<br/>
`④  先按下power键，紧接着同时按下option+command+p+r,等待   mac发出4声Duang~的声音后松手，系统会自动开机；`
`⑤  蓝牙菜单恢复正常；`<br/>


<br/>
<br/>

> <h2 id='Mac清理内存'>Mac清理内存</h2>


**`查看文件夹内的内存占用情况`**

&emsp;  使用下面命令系统就会自动统计该目录下所有文件的占用情况，一般等待一两分钟后就能得到结果了。若是想分析其它位置，你需要首先键入cd /文件夹路径移驾，然后再次运行`sudo du -sh *`开始扫描。

```
sudo du -sh *
```

<br/>

&emsp;   查看Mac内文件所占的Size，然后用`Command + shift + G`粘贴地址到(`/Users/用户名/Library/其他文件夹目录`)所在大文件夹。

```
$ du -hd 5 |grep -n '\dG' |sort
```

<br/>
<br/>

> <h2 id='可删除的文件'>可删除的文件</h2>

```
//该目录下的内容是Xcode在编译过程中产生的中间件，并且文件还特别大，在编译完成后能够被删除
~/Library/Developer/Xcode/DerivedData


//该目录下的内容是Xcode在做模拟器调试时生成的模拟器的数据，如果模拟器不再使用也可以删除
~/Library/Developer/CoreSimulator/Devices/
```
[Xcode 空间的清理](https://www.jianshu.com/p/7b39a31c312d)

[清理内存空间](https://www.jianshu.com/p/8fac91ff3453)

[手动清理内存](https://www.jianshu.com/p/41c736860925)

<br/>

**`删除所有系统日志——可以节省出100MB-2GB硬盘空间`**

&emsp;  随着你使用Mac的时间越来越长，系统日志文件也会越来越多，根据电脑的用量、错误和服务，这些文件会越来越多。这些系统日志文件是用来调试和排除故障的，如果你感觉没有用，可以使用下面的命令删除：

```
sudo rm -rf /private/var/log/*
```

<br/>

**`删除快速查看生成的缓存文件——可以节省出100MB-300MB硬盘空间`**

&emsp;  快速查看功能是OS X系统内置的文件预览功能，在Finder中选择任何文件后都可以点击空格来查看文件的详情。不过快速查看功能依靠缓存功能才能更流畅，而且这些缓存文件会一直增加，通过下面的命令移除缓存：

```
sudo rm -rf /private/var/folders/
```

<br/>

**`删除临时文件——可以节省500MB-5GB硬盘空间`**

&emsp;  `/private/var/tmp/`是存放系统缓存的文件夹，通常情况下会在系统重启时清楚，不过有时确不会。而且如果你长时间不关闭Mac，也不重启的话，缓存文件会越来越多。使用下面的命令清楚这些临时文件：

```
cd /private/var/tmp/

rm -rf TM*
```

<br/>

**`清除缓存文件——可以节省1GB-10GB硬盘空间`**
&emsp;  缓存文件有很多种，比如网页浏览记录，应用meta数据等等。这些缓存文件的容量究竟多大跟用户使用的应用有关，也与Mac重启的频率有关。此外，很多在线音乐播放app也会产生大量的缓存文件，我们可以通过下面的命令删除这些缓存文件：

```
cd ~/Library/Caches/

rm -rf ~/Library/Caches/*
```

[**Mac 清理软件集合**](https://blog.csdn.net/mandagod/article/details/89339544)




<br/>
<br/>

> <h2 id='破解软件无法安装'>**破解软件无法安装**</h2>


- 【安全隐私】没有权限安装来路不明的 App

![a27](./../Pictures/a27.jpg)

打开权限，在终端输入：`sudo spctl --master-disable`

- 文件损坏无法安装

![a28](./../Pictures/a28.png)

在终端输入：`sudo xattr -d com.apple.quarantine /Applications/[App的名字].app`， 这个可以从文件夹中将app拖入终端，然后再输入密码解决了，亲测可用。



<br/>

***
<br/>

> <h1 id=''></h1>




<br/>

***
<br/>

> <h1 id=''></h1>


># <h1 id = "DNS">[DNS](https://zhuanlan.zhihu.com/p/28273783)</h1>

- 域名劫持特征

一些小服务商以及小地方的服务商非常喜欢干这个事情。根据腾讯给出的数据，DNS劫持率7%，恶意劫持率2%。网速给的劫持率是10-15%。

- 把你的域名解析到竞争对手那里，然后哭死都不知道，为什么流量下降了。
- 在你的代码当中，插入广告或者追踪代码。这就是为什么在淘宝或者百度搜索一下东西，很快就有人联系你。
- 下载APK文件的时候，替换你的文件，下载一个其他应用或者山寨应用。
- 打开一个页面，先跳转到广告联盟，然后跳转到这个页面。无缘无故多花广告钱，以及对运营的误导。


<br/>

- <h2 id = "DNS缓存清除">DNS缓存清除</h2>

`sudo killall -HUP mDNSResponder`

<br/>

- 查看当前dns

`nslookup domain`

<br/>

- Mac的DNS服务IP配置

![<br/>](./../Pictures/tool_mac_0.png)


<br/>

***
<br/>























