># Mac 蓝牙不可用


`①  拔出与Mac连接的设备线；  `
`② 关机;  `
`③ 同时按下shift+control+option+power,保持5秒左右；`
`④  先按下power键，紧接着同时按下option+command+p+r,等待   mac发出4声Duang~的声音后松手，系统会自动开机；`
`⑤  蓝牙菜单恢复正常；`


<br/>

***
<br/>

># Mac 清理内存

#`查看文件夹内的内存占用情况`
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

#`可删除的文件`
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

#`删除所有系统日志——可以节省出100MB-2GB硬盘空间`
&emsp;  随着你使用Mac的时间越来越长，系统日志文件也会越来越多，根据电脑的用量、错误和服务，这些文件会越来越多。这些系统日志文件是用来调试和排除故障的，如果你感觉没有用，可以使用下面的命令删除：
```
sudo rm -rf /private/var/log/*
```

<br/>

#`删除快速查看生成的缓存文件——可以节省出100MB-300MB硬盘空间`
&emsp;  快速查看功能是OS X系统内置的文件预览功能，在Finder中选择任何文件后都可以点击空格来查看文件的详情。不过快速查看功能依靠缓存功能才能更流畅，而且这些缓存文件会一直增加，通过下面的命令移除缓存：
```
sudo rm -rf /private/var/folders/
```

<br/>

#`删除临时文件——可以节省500MB-5GB硬盘空间`
&emsp;  `/private/var/tmp/`是存放系统缓存的文件夹，通常情况下会在系统重启时清楚，不过有时确不会。而且如果你长时间不关闭Mac，也不重启的话，缓存文件会越来越多。使用下面的命令清楚这些临时文件：
```
cd /private/var/tmp/

rm -rf TM*
```

<br/>

#`清除缓存文件——可以节省1GB-10GB硬盘空间`
&emsp;  缓存文件有很多种，比如网页浏览记录，应用meta数据等等。这些缓存文件的容量究竟多大跟用户使用的应用有关，也与Mac重启的频率有关。此外，很多在线音乐播放app也会产生大量的缓存文件，我们可以通过下面的命令删除这些缓存文件：

```
cd ~/Library/Caches/

rm -rf ~/Library/Caches/*
```

<br/>

***
<br/>
[Mac 清理软件集合](https://blog.csdn.net/mandagod/article/details/89339544)




<br/>

***
<br/>

# 破解软件无法安装

- 【安全隐私】没有权限安装来路不明的 App
[a27](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a27.jpg)

打开权限，在终端输入：`sudo spctl --master-disable`

- 文件损坏无法安装
[a28](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/a28)

在终端输入：`sudo xattr -d com.apple.quarantine /Applications/[App的名字].app`， 这个可以从文件夹中将app拖入终端，然后再输入密码解决了，亲测可用。


















