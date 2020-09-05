# CocoaPosds(I)之安装和卸载


># 安装CocoaPosds[当前Mac OS X 10.15]

**`①是否安装了RVM，若没有需安装，否则第②步`**
什么是RVM?
&emsp;`Ruby Version Manager`简称`RVM`,是一款非常好用的`ruby`版本管理以及安装工具

**①查看是否安装RVM**
`rvm -v`
```
//没有安装rvm
zsh: command not found: rvm 
```

**②安装RVM**
`curl -L https://get.rvm.io | bash -s stable`
检查RVM是否安装上了
`rvm -v`
```
//没有安装上
zsh: command not found: rvm
```
这是因为没有从终端载入，这时可以这么做：
`source ~/.rvm/scripts/rvm `
然后检查RVM的版本：
`rvm -v`
```
rvm 1.29.9 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```
这时说媒RVM已经安装上了，这时可以进行Ruby的安装了。


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
>②安装Ruby



<br/>

**`更换镜像源`**
-  ruby 默认的原地址是国外网络地址，通过下面命令查看当前的镜像:
`gem sources -l`
```
gem sources -l
*** CURRENT SOURCES ***

https://rubygems.org/
```

-  移除当前镜像
`gem sources --remove https://rubygems.org/`
```
gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
```

-  添加国内的 ruby 镜像
`gem sources -a https://gems.ruby-china.com`

```
gem sources -a https://gems.ruby-china.com
https://gems.ruby-china.com added to sources
```

-  再次查看当前镜像,发现已经替换成功
```
gem sources -l
*** CURRENT SOURCES ***

https://gems.ruby-china.com/
```

<br/>

**`安装最新ruby`**
-  查看Ruby已有的版本，安装最新的：
`rvm list known`

![Ruby的版本](https://upload-images.jianshu.io/upload_images/2959789-9c334a8624c135e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  选择最新的版本进行安装,安装可能需要一点时间：
`rvm install 2.7`
&emsp;  在漫长的下载，编译过程，完成以后，Ruby, Ruby Gems 就安装好了。这期间若`Honebrew`没有安装，则在此过程中会进行自动安装。
![安装HomeBrew 失败](https://upload-images.jianshu.io/upload_images/2959789-1365b1d4b7c13293.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  即使`HomeBrew `没有安装好也可以安装CocoaPods(这是错的，必须安装否则无法安装Ruby)，检查Ruby是否安装好了(查看自己Mac的ruby源)：
`ruby -v`
```
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
```
-	传说 CocoaPods 支持的 ruby 最低版本是2.2.2，如果自己电脑版本低于这个版本就升级 ruby ，上面显示我的电脑版本不需要升级，可以忽略下面的升级操作
`sudo gem update --system`


<br/>

**`查询已经安装的ruby`**
`rvm list`
```
//提示rvm rubies还没有安装
# No rvm rubies installed yet. Try 'rvm help install'.
```
按他的提示，在终端输入：
```
rvm help install
```
在终端输入
`rvm help install`
在输出的内容中找到下图的版本号，然后找到指定的版本号进行安装
![指定版本号](https://upload-images.jianshu.io/upload_images/2959789-65d49892477c169d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是上图的版本号有点旧，我们可以用命令：
`rvm list known`
查看版本号，然后选中一个版本号进行安装(这里我选择2.6.3版本)：
`rvm install 2.6.3 --default`
会报下面的错误
![报错](https://upload-images.jianshu.io/upload_images/2959789-aa1b863c9dd52bd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
若出现如上问题说明Mac上没有安装Homebrew(若是安装了，这一步可以直接跳过) ,需要先安装：
` ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
在这里需要等待大概要2小时，可以看会剧比如：`海贼王、庆余年等`，安装成功后提示:
![安装成功提示](https://upload-images.jianshu.io/upload_images/2959789-2368f35b3b34d5d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在终端输入：
`brew help`
终端提示
![提示](https://upload-images.jianshu.io/upload_images/2959789-d09d056de35bc769.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

强制退出终端，然后重新打开终端，输入：
` rvm install 2.6.3`
安装后，提示
```
Install of ruby-2.6.3 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
```
终端输入：
`rvm docs generate-ri`
```
RVM version 1.29.9-next (master) is installed, yet version 1.29.9 (latest) is loaded.

Please open a new shell or run one of the following commands:

    rvm reload
    echo rvm_auto_reload_flag=1 >> ~/.rvmrc # OR for auto reload with msg
    echo rvm_auto_reload_flag=2 >> ~/.rvmrc # OR for silent auto reload
```
按照提示重新打开一个Shell或者输入命令，这里输入命令：
`rvm reload`
```
RVM reloaded!
```
更新Ruby版本
`rvm install 2.7.0-preview1`

<br/>

> ③安装CocoaPods

-  根据系统版本选择指令
Mac为 OS X 10.11之前系统的安装cocoapods 指令
` sudo gem install cocoapods`
Mac为 OS X 10.11以后系统的安装cocoapods 指令
`sudo gem install -n /usr/local/bin cocoapods `
由于我的系统版本是 OS X 10.15，所以选择：`sudo gem install -n /usr/local/bin cocoapods`

&emsp;  到了这里就成功安装CocoaPods了!如果你这样想，我只能说你太年轻了，后面的坑还多着呢！！！

- 安装本地库
`pod setup`

- 查看 Cocoapods 版本
`pod --version`
```
1.9.0.beta.3
```


-  对安装后的CocoaPods进行测试：
`pod search RxSwift` 或者 `pod install`,会出现：
![pod search 失败提示](https://upload-images.jianshu.io/upload_images/2959789-7a3f49197f6a92cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![pod install 失败提示](https://upload-images.jianshu.io/upload_images/2959789-88b40a784489d24b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  查看repo
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

&emsp;  podfile文件中添加source源后，pod install和pod update可以正常操作，但是pod search有些库却不正常,此时可以在终端执行:
`pod repo remove trunk`

&emsp;  Trunk 是用来自己写了一些类库上传到CocoaPods，但是这里我暂时用不到所以用不到，说以删除了。若是以后用到，可以自己重新添加进行配置Trunk。要重新创建请看[这里](https://www.jianshu.com/p/2572935ee006)


- 再次检测是否成功，终端输入
`pod search AFNetworking`
报错：
```
[!] Unable to find a pod with name, author, summary, or description matching `AFNetworking`
```

-  不幸的事是又失败了，使出撒手锏
`git clone https://git.coding.net/CocoaPods/Specs.git ~/.cocoapods/repos/master`
这时候`pod serarch Snapkit`事没问题了


-  终端输入安装出错
`pod install`
如下错误：
**### [!] Unable to add a source with url [https://github.com/CocoaPods/Specs.git](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%253A%252F%252Fgithub.com%252FCocoaPods%252FSpecs.git) named master. You can try adding it manually in ~/.cocoapods/repos or via pod repo add**
在终端依次输入然后回车：
```
rm -rf ~/.cocoapods

mkdir -p ~/.cocoapods/repos

cd ~/.cocoapods/repos

git clone https://github.com/CocoaPods/Specs.git master
```
&emsp;  这时再次使用pod install 和 pod update ，如丝滑般流畅，就这破东西整了一天，记住不要轻易升级系统和新的版本Cocoapods否则后面的坑一个接着一个来。


<br/>

***
<br/>

># 错误解决方案

<br/>

- 错误：`/usr/local/lib/ruby/gems/2.7.0/gems/cocoapods-core-1.9.1/lib/cocoapods-core/source/metadata.rb:15:in `initialize': undefined method `with_indifferent_access' for false:FalseClass (NoMethodError)`

终端输入：
```
$ sudo gem update cocoapods 
$  rm -rf ~/.cocoapods/repos/trunk/
```


<br/>

-  repos 为 0
```
pod repo list

0 repos
```
a. 先移除掉本地的master,在终端输入pod repo remove master;
b. `git clone --depth 1 https://github.com/CocoaPods/Specs.git master`,等待下载完毕；
[解决方案参考]([https://www.cnblogs.com/shuilangyizu/p/10935728.html](https://www.cnblogs.com/shuilangyizu/p/10935728.html)
)







<br/>

***
<br/>



># 卸载CocoaPosds

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
`
sudo rm -rf /usr/local/bin/pod ; gem list | grep cocoapods | awk '{print $1}' | while read line; do sudo gem uninstall $line;done
`
如下图：
![CocoaPods相关的东西](https://upload-images.jianshu.io/upload_images/2959789-2b4d3d8859ca5e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

**`第二种卸载方法：`**
**①查找目前版本的pod路径**
` which pod`

**②移除现有pod**
`rm -rf /usr/local/bin/pod`


&emsp; 这样CocoaPods 就卸载干净了。


<br/>

***
<br/>



