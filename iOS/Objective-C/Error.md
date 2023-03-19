> # 没有导入文件或者类库
![缺失文件或没有配置好][image-1]


<br/>
---- 
<br/>
> # 添加新账户或者验证账户是否具有有效凭证

![账户不正确存在问题][image-2]

`解决方案，请看下图：`
![融云demo下载，账户不正确问题的修改][image-3]


<br/>


> ## 模拟器可以运行真机不可以运行(反之模拟器不能运行，真机可以)报错：Undefined symbols for architecture i386:

![报错截图][image-4]

解决方案：
![Build Active Architecture Only 的 Release 模式修改为 Yes][image-5]

`原因：`这是我当初在勾选了自动签名

![选择自动签名][image-6]

修改了自动签名时的代码签名导致的。

![修改代码签名导致][image-7]





<br/>
---- 
<br/>

> # APP 在原有基础上再次添加内购不被审核

&emsp;   项目初建时添加的苹果内购项目，通过提交二进制代码通过审核批准后的内购项目。
![第一次内购审核通过后][image-8]

&emsp;  上图可以看到已通过的内购项目如绿色圆圈显示的。

&emsp;  当第二次更新APP版本并且又增加了内购项目时，我们需要`提前`在Appstore Connect -\> 我的App -\>App Store的
![版本或平台][image-9]上新建一个版本，然后在
![App 内购买项目][image-10]上添加新的内购项目。`若是没有建立新的版本`，苹果会把审核视为上一版本的内购，当他审核时会找不到，就会报错让你再次提交二进制代码。
 
&emsp;  结果就是你傻傻的提交，但是它还是报错，让你抓狂不已，叫天天不应，叫地地不灵！


正确的方法是：
①.  建立新的版本，先放在那；
②.    在![App 内购买项目][image-11]上添加你的内购信息，然后存储(千万不要点击`提交已审核`，若点了他会把这个内购视为你的上一个版本，你又做了无用功);
③.  当你的这个版本开发完成后，提交二进制到App Store了，构建版本成功了，如下图：![2.3.5的版本][image-12]
你往下拉网页，你会看到![添加购买项目][image-13]
就像新添加的一样，你点击那个蓝色加号就会看见你这个版本要添加的IAP了，选择你要这次审核的IAP就好了。成功后的图为：![2.3.3版本的内购项目部分][image-14]


&emsp;  就为了这东西，上下了折腾了3次，烦死了！！！！




<br/>
---- 
<br/>

> # 地理位置隐私数据要求访问，但是没有提示，导致构建失败
`错误提示：`Your delivery was successful, but you may wish to correct the following issues in your next delivery:
Missing Purpose String in Info.plist File - Your app's code references one or more APIs that access sensitive user data. `The app's Info.plist file should contain a NSLocationAlwaysUsageDescription key with a user-facing purpose string explaining clearly and completely why your app needs the data.`Starting Spring 2019, all apps submitted to the App Store that access user data will be required to include a purpose string.

&emsp;  当时我看到了`NSLocationAlwaysUsageDescription`这个，就在APP内的 `Info.plist`文件中添加这个字段，然后打包，结果还是提示上述问题，而且
`NSLocationAlwaysUsageDescription`也没变。后来查了别人的博客才知道，原来使用一些第三方API(如：高德地图SDK)会要求定位，所以按照下图所示：

这几个访问地理字段的都有才可以：
	
	    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
		<string>太极.道需要您的同意，才能使用GPS</string>
	
		<key>NSLocationAlwaysUsageDescription </key>
		<string>太极.道需要您的同意，才能始终访问位置</string>
	
		<key>NSLocationUsageDescription</key>
		<string>太极.道需要您的同意，才能访问位置</string>
	
		<key>NSLocationWhenInUseUsageDescription</key>
		<string>太极.道需要您的同意，才能在使用期间访问位置</string>
&emsp;  添加完以后，再次打包，上传到Appstore，在提交审核的地方等个15分钟，发现了要构建的新版本了。

























[image-1]:	https://upload-images.jianshu.io/upload_images/2959789-c5a5b4882b2146f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-2]:	https://upload-images.jianshu.io/upload_images/2959789-5882fd3cd40e8055.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-3]:	https://upload-images.jianshu.io/upload_images/2959789-15feec72a369a7df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-4]:	https://upload-images.jianshu.io/upload_images/2959789-f56b99b7923d0dd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-5]:	https://upload-images.jianshu.io/upload_images/2959789-4dcf8c80c66273a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-6]:	https://upload-images.jianshu.io/upload_images/2959789-59f4ee92a049c28b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-7]:	https://upload-images.jianshu.io/upload_images/2959789-c1ba393f4cda301d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-8]:	https://upload-images.jianshu.io/upload_images/2959789-06a341638688b839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-9]:	https://upload-images.jianshu.io/upload_images/2959789-4b7b26e583c561ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-10]:	https://upload-images.jianshu.io/upload_images/2959789-7aa044908bc9a770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-11]:	https://upload-images.jianshu.io/upload_images/2959789-7aa044908bc9a770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-12]:	https://upload-images.jianshu.io/upload_images/2959789-e901c1bcfc084515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-13]:	https://upload-images.jianshu.io/upload_images/2959789-f6e935e654844802.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-14]:	https://upload-images.jianshu.io/upload_images/2959789-22f47a262f781570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240