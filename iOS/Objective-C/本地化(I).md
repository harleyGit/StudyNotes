
- **语言本地化**
- **本地推送**
- **设备信息**
- [**应用内切换语言实现本地化**](https://blog.csdn.net/wahaha13168/article/details/82989147)



<br/>

***
<br/>

># 语言本地化

```
获取用户语言偏好设置列表
[NSLocale preferredLanguages];
```

[本地化语言与手动语言切换 US](https://www.jianshu.com/p/73973d31dc82)


<br/>

***
<br/>

># 本地推送


**`1. 本地推送流程:`**

a.  创建一个触发器（trigger）
b. 创建推送的内容（UNMutableNotificationContent）
c. 创建推送请求（UNNotificationRequest）
d. 推送请求添加到推送管理中心（UNUserNotificationCenter）中


<br/>

**2.本地推送类为：**

```
UNTimeIntervalNotificationTrigger;
UNCalendarNotificationTrigger;
UNLocationNotificationTrigger;
```


<br/>

**3.本地推送分类**

>**`UNTimeIntervalNotificationTrigger`**

```
  //timeInterval：单位为秒（s）  repeats：是否循环提醒
  UNTimeIntervalNotificationTrigger *timeTrigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5 repeats:YES];
```


<br/>

>**`UNCalendarNotificationTrigger `**

进行注册,时间点信息用 NSDateComponents.（定期推送）

`+ (instancetype)triggerWithDateMatchingComponents:(NSDateComponents *)dateComponents repeats:(BOOL)repeats;`

```
   //每周五的11点0分提醒
    NSDateComponents *components = [[NSDateComponents alloc] init];
    components.weekday = 7;
    components.hour = 11;
    components.minute = 0;
    
    //日期
    //进行注册,时间点信息用 NSDateComponents.（定期推送）
    UNCalendarNotificationTrigger *calendarTrigger = [UNCalendarNotificationTrigger triggerWithDateMatchingComponents:components repeats:YES];
```


<br/>

>**`UNLocationNotificationTrigger`**

&emsp;  进行注册，地区信息使用CLRegion的子类CLCircularRegion，可以配置region属性 notifyOnEntry和notifyOnExit，是在进入地区、从地区出来或者两者都要的时候进行通知，这个测试过程专门从公司跑到家时刻关注手机有推送嘛，果然是有的（定点推送）

`+ (instancetype)triggerWithRegion:(CLRegion *)region repeats:(BOOL)repeats`


<br/>

**`创建推送内容`**

[UNNotificationContent](https://link.jianshu.com?t=https://developer.apple.com/reference/usernotifications/unnotificationcontent)属性：readOnly;

[UNMutableNotificationContent](https://link.jianshu.com?t=https://developer.apple.com/reference/usernotifications/unmutablenotificationcontent)属性：title、subtitle、body、badge、sound、lauchImageName、userInfo、attachments、categoryIdentifier、threadIdentifier;

本地消息内容展示:

`title |NSString|`: 限制在一行，多出部分省略号;

`subtitle |NSString |`:限制在一行，多出部分省略号;

`body| NSString |`:通知栏出现时，限制在两行，多出部分省略号；预览时，全部展示

**`注意：`**body中printf风格的转义字符，比如说要包含%，需要写成%% 才会显示，\同样。

```
    // 创建通知内容 UNMutableNotificationContent, 注意不是 UNNotificationContent ,此对象为不可变对象。
    UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
    content.title = @"Dely 时间提醒 - title";
    content.subtitle = [NSString stringWithFormat:@"Dely 装逼大会竞选时间提醒 - subtitle"];
    content.body = @"Dely 装逼大会总决赛时间到，欢迎你参加总决赛！希望你一统X界 - body";
    content.badge = @666;
    content.sound = [UNNotificationSound defaultSound];
    content.userInfo = @{@"key1":@"value1",@"key2":@"value2"};
    
```

<br/>

`本地通知Demo`

```
#ifdef NSFoundationVersionNumber_iOS_9_x_Max
#import <UserNotifications/UserNotifications.h>
#endif

//遵守协议 <UNUserNotificationCenterDelegate>

- (void) registerLocalNotification {
    //注册通知设置代理
    UNUserNotificationCenter *unc = [UNUserNotificationCenter currentNotificationCenter];
    unc.delegate = self;
    //用户授权
    [unc requestAuthorizationWithOptions:(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge) completionHandler:^(BOOL granted, NSError * _Nullable error) {
        
    }];
    //获取当前通知设置
    [unc getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
        
    }];
}

//收到本地推送时调用
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler {
    
    NSLog(@"推送内容 %@", notification.request.content.body);
    
    completionHandler(UNNotificationPresentationOptionSound | UNNotificationPresentationOptionAlert);
}

//点击推送弹窗时调用
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)(void))completionHandler {
    
    NSLog(@"点击推送 %@", response.notification.request.content.body);
}

+ (void) localPushNotification {
    
    //timeInterval：单位为秒（s）  repeats：是否循环提醒
    UNTimeIntervalNotificationTrigger *timeTrigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:5.0f repeats:NO];
    
    //通知内容
    UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
    content.title = @"title: 海贼王：狂热行动";
    content.subtitle = [NSString stringWithFormat:@"subtitle: 海贼万博会"];
    content.body = @"body: 路飞大战巴雷特，霸气大开！！！！";
    content.badge = @666;
    content.sound = [UNNotificationSound defaultSound];
    content.userInfo = @{@"key1":@"value1",@"key2":@"value2"};
    
    //通知标示
    NSString *requestIdentifier = @"date.come.time";
    
    // 创建通知请求 UNNotificationRequest 将触发条件和通知内容添加到请求中
    UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:requestIdentifier content:content trigger:timeTrigger];
    
    UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
    
    // 将通知请求 add 到 UNUserNotificationCenter
    [center addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        if (!error) {
            NSLog(@"推送已添加成功 %@", requestIdentifier);
            //你自己的需求例如下面：
            UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"本地通知" message:@"成功添加推送" preferredStyle:UIAlertControllerStyleAlert];
            UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
            [alert addAction:cancelAction];
            [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:alert animated:YES completion:nil];
            //此处省略一万行需求。。。。
        }
    }];
    
}




[self registerLocalNotification];
[AppDelegate localPushNotification];
```

效果：

![本地通知](https://upload-images.jianshu.io/upload_images/2959789-2fc487881afa5c64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>

># 设备信息

**[获取设备的一些信息](https://www.jianshu.com/p/2e524e3d9ae4)**

**[查看包名、版本号、设备信息、签名、进程ID](https://blog.csdn.net/wejfoasdbsdg/article/details/77967874?utm_source=blogxgwz8)**

**[iOS 如何获取设备信息](https://www.jianshu.com/p/4d4c75450ef7)**




<br/>

***
<br/>
