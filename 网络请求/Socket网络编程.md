
># UDP 和 TCP 
![UDP 和 TCP 的区别](https://upload-images.jianshu.io/upload_images/2959789-2cdee56d98ac5c41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  掉帧就是掉数据了，发生UDP也就是网络层，比如玩王者荣耀游戏和视频直播(网络流媒体)时出现马赛克。
&emsp;  TCP的应用场景比如是下载文件，若是使用UDP可能会出现丢包现象。比如你若是用UDP下载几个G的Xcode，结果完成后发现丢包，Xcode就会打不开，那是你的内心是崩溃的。

#`Scocket通信机制`
![Socket(套接字、插座-AT&T)](https://upload-images.jianshu.io/upload_images/2959789-f687c4d7e23f836b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  知道了IP地址(知道了设备)、端口号(使用哪种服务，如:发邮件是一种服务，它使用了一个端口)、传输讯协议(UDP/TCP，知道如何传输数据)，两台不同的设备就可以通信了。





上一篇：[^fn1]
[^fn1]: [网络请求](https://www.jianshu.com/p/987100a583c3)
