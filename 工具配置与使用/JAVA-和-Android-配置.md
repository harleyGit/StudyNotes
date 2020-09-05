># 下载JDK
&emsp; 要想下载方便，请在Mac上先下载 brew，然后使用这个命令下载一些东西十分方便。
```
//默认应该会下载jdk7
 brew cask install java
//也可以指定下载版本
//brew cask install java6
```





<br/>
***
<br/>


># Android SDK 的下载和环境变量配置
```
//安装sdk
brew cask install android-sdk
//配置Path环境
vim ~/.bash_profile
//设置路径
export ANDROID_SDK_ROOT=/usr/local/share/android-sdk
export PATH="${PATH}:${ANDROID_SDK_ROOT}/tools:${ANDROID_SDK_ROOT}/platform-tools"
```

&emsp;  若是不想进行上面的下载和配置，需要先下载[Java SE](https://www.oracle.com/technetwork/java/javase/downloads/jdk13-downloads-5672538.html)然后再下载[Android strudio](https://developer.android.google.cn/studio/)一揽子把其下载和配置完,方便快捷。


<br/>
***
<br/>
[Mac下adb以及Android studio夜神模拟器的配置](https://www.jianshu.com/p/04905a59d2a9)
[Mac配置adb命令](https://www.jianshu.com/p/4b5452f54206)
 [MAC环境Android SDK环境变量配置](https://www.cnblogs.com/itgezhu/p/11086785.html)
[android sdk 下载](https://blog.csdn.net/qq_37443229/article/details/87627385)





