>#UIApplication显示和隐藏状态栏的网络活动标志
```
//在向服务端发送请求状态栏显示网络活动标志
[[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:YES]; 

//这里是发送服务端请求的代码 //请求结束状态栏隐藏网络活动标志
[[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:NO];
```
