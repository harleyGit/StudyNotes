![推送通知关系图](https://upload-images.jianshu.io/upload_images/2959789-79dca67e3ef5821a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


&emsp;  在推送通知时，可以让通知栏目中有按钮，需要在AppDelegate中实现：`application: handleActionWithIdentifier:  forRemoteNotification:  completionHandler:`方法。
用到的按钮初始化对象类： `UIMutableUserNotificationCategory()` (通知类别对象)
