># 清理缓存文件

-	删除Xcode中多余的证书provisioning profile 
```
手动删除： 
Xcode6 provisioning profile path： 
~/Library/MobileDevice/Provisioning Profiles
```

-	清理Xcode编译项目产生的缓存垃圾 
```
	(Xcode永久了，会产生很多项目编译缓存，占用一大堆硬盘空间，此时需要对该目录进行清理） 
手动删除： 
Xcode编译项目缓存垃圾的目录： 
~/Library/Developer/Xcode/DerivedData
```

- 真机/模拟器运行项目非常慢的解决方法
```

删除 ~/Library/Developer/Xcode/iOS DeviceSupport/该目录下，所有文件夹;


选择Xcode --> Window-->Devices and Simulators，找到真机设备，鼠标右键选择unpair the device

然后重启Xcode、重新连接设备、重新运行应用程序

```

<br/>

***
<br/>

参考资料：
<br/>
[Xcode清理垃圾文件](https://www.jianshu.com/p/4540d34431db)
