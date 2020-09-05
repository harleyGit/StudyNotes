># pod update 和 pod install
**`update 和 install 区别：`**
- update:

- install:


<br/>
**`update 失败解决：`**
终端输入：
```
pod update 
```
错误提示，如下图：
![无法更新，错误提示](https://upload-images.jianshu.io/upload_images/2959789-9df1e959b994c491.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`解决方法`**
终端输入：
```
//pod repo update命名是用来更新本地cocoapods的spec资源配置信息
//安装完cocoapods后，在用户根目录下有个隐藏文件夹，/Users/harleyhuang[用户名]/.cocoapods，里面是cocoapods收录的所有库的配置信息；
// /Users/harleyhuang[用户名]/.cocoapods/repos/master/Specs/；
// 比如Kingfisher就是/Users/harleyhuang[用户名]/.cocoapods/repos/master/Specs/a/a/6/Kingfisher，内部分版本包含多个文件夹，每个文件夹内包含一个配置文件，比如Kingfisher.podspec.json

 pod repo update --verbose
```
如：Kingfisher.podspec.json 所在文件夹路径图：
![.cocoapods 下的隐藏文件](https://upload-images.jianshu.io/upload_images/2959789-bb6ac0921fb5330c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Kingfisher 的5.13.1 版本](https://upload-images.jianshu.io/upload_images/2959789-70c00e4f79dc1239.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![更新整个.cocoapods下的所有库](https://upload-images.jianshu.io/upload_images/2959789-ea752f45f3805eca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后再次在终端输入：`pod update`即可。












