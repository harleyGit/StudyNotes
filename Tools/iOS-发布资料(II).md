>#自动管理代码签名
简介：通过配置Xcode环境来自动管理描述文件（profiles）、app IDs（Bundle Identifier）和证书（certificates）。


&emsp;  当我们选中Automatically manage signing(这里简称AMS)选框，并且选择对应Team时，Xcode就会帮我们自动生成对应的描述文件、app id和证书文件（完全不需要我们自己手动去创建这些证书）。

作用：在真机调试的时候，如果我们有新的设备，不需要我们到苹果开发者网站去添加新设备的UDID和更新对应的描述文件，Xcode会自动帮我们创建和更新这些内容，这就是它的方便之处。

<br/>
***



>#自动配置开发证书过程
出现错误：

![自动配置证书错误](https://upload-images.jianshu.io/upload_images/2959789-2d0a2b64a286166f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



&emsp;  存   在：Xcode会在本机钥匙串寻找team对应的开发证书,如果本地钥匙串存在该证书则加载使用。 
&emsp;  不存在：则从开发者中心寻找本机对应的开发证书,如果开发者中心没有则自动生成一个并下载到钥匙串使用;如果AMS已为这台电脑生成过开发证书,则提示如下图中的信息。

![开发者中心证书管理](https://upload-images.jianshu.io/upload_images/2959789-161a67b6c25f878f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

出现上述【自动配置证书错误】解决方法：
MethodOne：前往开发者中心证书管理页面（上图）下载本机的证书导入本机钥匙串。
MethodTwo：直接选择revoke，但revoke会把开发者中心对应本机的证书重新生成覆盖掉，这样可以解决问题，但是会有一个隐患如果profile中存在被覆盖的本机证书对应的profile，且只有这一个证书时，该profile会因不含发布证书而变无效。这个很重要，具体会在文后关于Provisioning展开讨论。

<br/>
***

#>打包上传到AppStore的背后
&emsp;  选择save for app store deployment或者save for Ad Hoc deployment或者upload to App Store发生了什么?

![上传包到AppStore](https://upload-images.jianshu.io/upload_images/2959789-67dd4ba0d62fade8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##Xcode自动加载发布证书。
&emsp;  若存在Xcode会去本机钥匙串寻找苹果账号对应的发布证书，若不存在则从开发者中心寻找发布证书，如果开发者中心不存在则新生成一个，并下载到本机钥匙串，并加载。

下一步，如果存在出现下面的错误。
![未能找到或者匹配的证书资源](https://upload-images.jianshu.io/upload_images/2959789-58fc46d32ba77c00.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####解决方案：
&emsp; 方法一：可选择去开发者中心下载（前提是那个发布证书是本机生成的，如果是其他人的机器生成的，下载也没用，因为没有私钥，私钥要从别人那里导出，这种情况只能去那个人的电脑钥匙串导出p12文件，关于证书和私钥的详细说明请移步后文）

&emsp; 方法二：选择reset，然后try again就可以了。但reset慎重，reset还是有可能会导致Provisioning无效的情况，而且别人的证书不再有效，以后进行和发布证书相关的操作，他就会面临和你现在相同的问题，出现恶性循环。


&emsp;    Xcode正确加载发布证书之后，会根据App id去profile下载Provisioning了，有则下载，无则自动创建一个(绑定当前app id和发布证书)并下载，但是，在这一步并不会检查Provisioning有无效，详细请看文后关于Provisioning话题。

&emsp; 从开发者中心下载其他电脑生成的证书，然后导入到本机中。会在【钥匙串】中发现证书是不包含秘钥的。自然使用不了，结果也的确这样，提示revoke。

![Remoke 错误](https://upload-images.jianshu.io/upload_images/2959789-61f7e41d0a4232c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; 由以上也可证明一点，Xcode帮我们申请的证书（开发、测试证书）都跟电脑绑定的。如果我们想让其他电脑也能用这台电脑的证书，也只有采用老方法，导出p12文件，这样秘钥也导过去了。就可以正常使用。

<br/>
***


>#关于Provisioning Profiles

##  APP IDs、证书、Provisioning Profiles 三者的联系
&emsp; 我们知道最终打包进去ipa的是Provisioning Profiles描述文件，而Provisioning Profiles文件则包含了开发者的开发或者发布证书，以及APP IDs（可以理解为bundle identifier）。做了这么多事情，也是为了得到正确的Provisioning从而成功的打包或者上传我们的app。如果证书或者APP ID出错了，最后的Provisioning自然也是错的，签名过程自然就会出现问题。
<br/>

##  revoke和reset导致的Provisioning Profiles错误
&emsp; 经过观察，Provisioning 无效会有两种情况，要么过期了，要么不含有证书了。这样在开发者中心Provisioning Profiles页面就会显示invalid无效。Xcode加载Provisioning的时候并不会验证Provisioning是否有效。也就是说，显示invalid的Provisioning，Xcode也会加载回来，且不会提示错误。这样就有可能会有问题。

###举个例子
&emsp; 假如开发者中心一片空白，当在mac A打包上传时，开发者中心为该开发者创建了关联mac A的开发证书和发布证书，并且自动生成了app id和Provisioning，此时的Provisioning就是关联了发布证书的，且就一个证书，例如下图。
![Provisioning Profiles](https://upload-images.jianshu.io/upload_images/2959789-12e90be2adca5564.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp; mac A就成功打包上传了，一切都很顺利。然而过了一段日子，需要在mac B打包上传，因为mac B是没有发布证书的，自然会提示reset，如果mac B进行了reset，那么Xcode自动为mac B创建的发布证书就会替代mac A之前的证书了，原证书没了，Provisioning的Certificates就会变成0 total，从而变成invalid，无效了。但是mac B继续打包上传，在Xcode从开发者信息中寻找Provisioning的时候，因为含有同样的APP ID的Provisioning存在了，Xcode不会再为开发者重新创建一个Provisioning，而是直接加载！！而加载这个恰好就是无效的！Xcode也不会提示任何错误。那么之后就GG了。

<br/>
***



>#App IDs
&emsp; 在开发者中心APP IDs有个Wildcard App ID的概念，大概的意思就是使用通配符的App ID可以通配一系列符合的APP ID；比如定义了一个Wildcard App ID：com.，那么这个App ID就可以统配所以以com.开头的bundle identifier，xcode不会再为其生成相同名字的App ID。同理，在第一次使用com. App ID时，Xcode会为其新建Provisioning，Provisioning对应的App ID就是com.，以后再有新项目bundle identifier以com.开头，Xcode不会再为其创建Provisioning，也就是公用com. APP ID的应用共用一个Provisioning。通配符有其好处，也有不好的地方，感兴趣大家可以到Apple官方看看相关内容。

##Xcode自动为项目创建APP ID的过程

说了这么多Wildcard App ID的内容，也是为了说一下，Xcode自动为项目创建APP ID的过程。如果开发者中心有与bundle identifier相同的APP ID则不创建，如果没有，会看有没有适合的Wildcard App ID，如果有也不创建，如果没有才最后创建与bundle identifier相同ID的App ID。




<br/>
***
参考资料：[AutoMatically manager signing背后你不得不知道的事](https://www.jianshu.com/p/2effcd4c4453)















