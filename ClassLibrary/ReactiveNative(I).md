># 搭建环境
#`安装Homebrew`
在终端拷入如下命令：
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
但是这样安装可能报错：
`curl: (7) Failed to connect to raw.githubusercontent.com port 443: Operation timed out`
点击[解决Homebrew报错](https://www.jianshu.com/p/68efabd2e32b)可以解决。

&emsp;  在Max OSX 10.11版本中，Homebrew在安装软件时可能会遇到 `/usr/local` 目录不可写的权限问题，可以使用下面的命令修复：
```
sudo chown -R 'whoami' /usr/local
``` 
①有点变态，权限给的很大
```
sudo chown 777 /usr/local
```
②非常变态，建议用这个，反编译时会用到
&emsp;  为什么对这个 `/usr/local` 进行文件的限制读写，这是因为MacOS 10.11 有一个机制Rootless防止恶意程序的最后防线，采用下面的命令是把这个机制关掉了。
① 将电脑重启按住了 CMD+R 进入恢复模式；
② 进入恢复模式之后，打开终端，输入命令：
```
$csrutil disable  //关闭Rootless
$csrutil enable  //打开Rootless
``` 




<br/>
***
<br/>
># 安装Node.js 和 Watchman
使用已经安装好的Homebrew来进行安装  Node
```
brew install node
brew install watchman
```        
&emsp;  如果你已经安装了 Node，请检查其版本是否在 v10 以上。安装完 Node 后建议设置 npm 镜像以加速后面的过程（或使用科学上网工具）。
注意：`不要使用 cnpm！cnpm 安装的模块路径比较奇怪，packager 不能正常识别！`           
```
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```        
&emsp;  [Watchman](https://facebook.github.io/watchman)则是由 Facebook 提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager 可以快速捕捉文件的变化从而实现实时刷新）。






<br/>
***
<br/>

># Yarn、React Native 命令行工具(react-native-cil)
&emsp;  [Yarn](http://yarnpkg.com/)是 Facebook 提供的替代 npm 的工具，可以加速 node 模块的下载。React Native 的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。
```
npm install -g yarn react-native-cli
```
安装完 yarn 后同理也要设置镜像源：
```
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```
&emsp;  安装完 yarn 之后就可以用 yarn 代替 npm 了，例如用yarn代替npm install命令，用yarn add 某第三方库名代替npm install 某第三方库名。

&emsp;  若是看到`EACCES: permission denied` 这样的权限设置报错，参照上文的Homebre，修复`/usr/local`目录的所有权：
```
sudo chown -R 'whoami' /usr/local
```





<br/>
***
<br/>
># Flow
可以给予一些代码的提示和错误。
```
brew install flow
```






<br/>
***
<br/>
># 创建一个新项目
若是直接使用：
```
$ react-native init Demo
```
&emsp;  在终端进行初始化一个Demo的项目，只会下载一个`package.json`的文件，其他任何东西都没有。
&emsp;  这是因为目前最新的 0.45 及以上版本需要下载 boost 等几个第三方库编译。这些库在国内即便翻墙也很难下载成功，导致很多人`无法运行iOS项目`！！！中文网在论坛中提供了这些库的[国内下载链接](http://bbs.reactnative.cn/topic/4301/)。如果你嫌麻烦，又没有对新版本的需求，那么可以暂时创建`0.44.3`的版本。
在终端输入：
```
$ cd /Users/huanggang/Desktop
$ react-native init Demo --version 0.44.3

```
文件是`99.2MB`大小。
选中`Demo`文件夹下的`iOS`文件夹选中项目进行运行。








<br/>
***
<br/>
># WebStorm 的使用
&emsp;  打开百度，寻找破解版的的WebStorm，然后下载，使用它打开刚刚新建的`Demo`。
目录结构：
![项目目录结构](https://upload-images.jianshu.io/upload_images/2959789-861140a6e87feb5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在WebStorm中，打开设置的快捷键 `Command+ ,` ,对其进行如下配置：
![语言配置_1](https://upload-images.jianshu.io/upload_images/2959789-ebfd8c2e8eae9238.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








<br/>
***
<br/>
># 管理版本
#`查看版本`
```
$ cd /Users/huanggang/Desktop/Demo   //进入当前项目的文件
$ react-native -v      //查看版本

react-native-cli: 2.0.1
react-native: 0.44.3    //版本号
```

在WebStorm中也可以看到，打开`package.json`，可以看到：
```
"dependencies": {
    "react": "16.0.0-alpha.6",
    "react-native": "0.44.3"
  }
```


<br/>
#`更新版本`
npm(Node Package Manager):包管理命令
```
$npm update -g react=native
```

<br/>
#`查询RN版本`
```
$npm info react-native
```


<br/>
#`升级或者降级RN 版本`
```
$npm install --save react-native@0.xx.x  //版本号
```

若是WebStorm中没有代码提示，可以下载插件，去百度搜索吧！
