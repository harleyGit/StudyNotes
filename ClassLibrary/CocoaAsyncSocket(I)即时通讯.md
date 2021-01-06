#`CocoaPod 下载 CocoaAsyncSocket `
```
# 第三方开源即时通讯SDK
pod 'CocoaAsyncSocket', '~> 7.6.3'
```

<br/>

#`获取IP信息`
```
struct ifaddrs {
	struct ifaddrs  *ifa_next;  //指向链表的下一个成员(也可以是下一个struct ifaddr结构)
	char		*ifa_name;  //接口名称，以0结尾的字符串，比如eth0，lo
	unsigned int	 ifa_flags;  //接口的标识位(比如当IFF_BROADCAST或IFF_POINTOPOINT设置到此标识位时，影响联合体变量ifu_broadaddr存储广播地址或ifu_dstaddr记录点对点地址）
	struct sockaddr	*ifa_addr;  //指向一个包含网络地址的sockaddr结构
	struct sockaddr	*ifa_netmask;  //存储该接口的子网掩码；结构体变量存储广播地址或点对点地址(见括弧介绍ifa_flags）
	struct sockaddr	*ifa_dstaddr;  //如果(ifa_flags&IFF_POINTOPOINT)有效，ifu_dstaddr指向一个包含p2p目的地址的结构
	void		*ifa_data;  //指向一个缓冲区(存储了该接口协议族的特殊信息)，其中包含地址族私有数据,没有私有数据,通常是NULL(一般不关注他)
};


//获取本地网络接口信息，将之存储于链表中，链表头结点指针存储于__ifap中带回，函数执行成功返回0，失败返回-1，且为errno赋值。
//函数getifaddrs用于获取本机接口信息，比如最典型的获取本机IP地址
getifaddrs（int getifaddrs (struct ifaddrs **__ifap)）


struct sockaddr_in {
	__uint8_t       sin_len;
	sa_family_t     sin_family;   //地址族(Address Family)
	in_port_t       sin_port;        //16位TCP/UDP端口号
	struct  in_addr sin_addr;    //32位IP地址
	char            sin_zero[8];      // 不使用
};

//区分相同驱动的接口
IFF_UP

```

[sockaddr和sockaddr_in详解 US](https://blog.csdn.net/qingzhuyuxian/article/details/79736821)
[Socket & CocoaAsyncSocket介绍与使用 US](https://www.jianshu.com/p/7c3045776f9d)
[即时通讯IM US](https://www.jianshu.com/p/c0778466e099)
[iOS 使用 socket 即时通信（非第三方库）US](https://www.cnblogs.com/Free-Thinker/p/10435519.html)
[SocketRocket US](https://github.com/facebook/SocketRocket)
