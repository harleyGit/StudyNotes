# `1.  访问隐私信息，比如：位置、相机、照片，没有对访问这些信息进行说明，导致审核失败`
```
Guideline 5.1.1 - Legal - Privacy - Data Collection and Storage


We noticed that your app requests the user’s consent to access their location but does not clarify the use of the location in the applicable purpose string.

Next Steps

Please revise the relevant purpose string in your app’s Info.plist file to specify why the app is requesting access to the user's location. You can modify your app's Info.plist file using the property list editor in Xcode.

To help users understand why your app is requesting access to their personal data, all permission request alerts in your app should specify how your app will use the requested feature.

```

在info.plist 文件中，访问位置信息的提示语：
` <key>NSLocationAlwaysUsageDescription </key>`
`<string>太极.道需要您的同意，才能始终访问位置</string>`

应该为：
`<string>太极.道会在定位、搜索服务中使用您的位置信息</string>`


<br/>
***
<br/>

#`2. 视频购买VIP,把IAP的项目通通申请为一次性购买，因为在后台控制这个VIP的时间不用苹果这边来控制，但是我错了`
```
Guideline 3.1.1 - Business - Payments - In-App Purchase


We noticed that your in-app purchase product is set to an incorrect product type.

五星月VIP, 一星月VIP, 一星终身 VIP, 一星年 VIP, 二星月 VIP, 二星年 VIP, 二星终身 VIP, 三星月 VIP, 三星年 VIP, 四星月 VIP, 三星终身 VIP, and 四星年 VIP is set to Consumable.

Next Steps

Since the service offered by your app requires the user to make an advance payment to access the content or receive the service, please use the non-renewable subscription in-app purchase product type. Non-renewable subscription content must be made available to all iOS devices owned by a single user, as indicated in guideline 3.1.2 of the App Store Review Guidelines.

Note: The product type cannot be changed once an in-app purchase product has been created. Therefore, you will need to create a new in-app purchase product with the correct product type.

```
&emsp;  上网查了下悲剧的原因，原来我把本应该【非续期订阅】选择为【非续期订阅】类型了，导致了被拒。感叹：还是得本本分分的做人，守规矩啊！

&emsp;  知道原因后，把原来的IAP被拒审核删除掉，重新选择类型添加IAP项目，共修改了18个！再对项目中的18个进行修改，那个累啊！



<br/>
***
<br/>


#`3.  对于一些不需要用户进行注册、登录的功能，在App中强制用户进行注册、登录导致了被拒，这是设计的缺陷`

```
### Guideline 5.1.1 - Legal - Privacy - Data Collection and Storage

We noticed that your app requires users to register with personal information to purchase non account-based in-app purchase products, which does not comply with the App Store Review Guidelines.

Apps cannot require user registration prior to allowing access to app content and features that are not associated specifically to the user.

**Next Steps**

User registration that requires the sharing of personal information must be optional or tied to account-specific functionality.

To resolve this issue, please make it clear to the user that registering will enable them to access the content from any of their iOS devices and provide them a way to register at any time, if they wish to later extend access to additional iOS devices.

Please note that although guideline 3.1.2 of the [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) requires an app to make subscription content available to all the iOS devices owned by a single user, it is not appropriate to force user registration to meet this requirement; such user registration must be made optional.

```
&emsp;  在太极道的视频中，因为不是注册、登录的用户无法进行观看VIP视频、普通用户视频。但是在一些知名播放App，如：腾讯视频、优酷视频他都能让游客用户观看，但对于VIP它也是让用户先看6分钟，再提示用户登录、充值。


