- KeyChain
- KeyChain Item
- Keychain的用法




<br/>

***
<br/>




># KeyChain

- **NSUserDefaults作用的场所:**

&emsp; NSUserDefaults其实是plist文件中键值存储，并且最大的问题是存在与沙盒中，这就对安全性埋下了隐患。如果攻击者破解app，拿到了沙盒中的数据，就会造成数据泄漏，后果不堪设想。

&emsp; 当然，一般也不会有把密码直接使用NSUserDefaults存储的，都会进行加密、或者是多重加密后再进行NSUserDefaults存储。这么做其实是可行的，前提是加密算法不能泄漏。有个小问题就是，如果用户删掉app重装的话，之前所有存储的敏感信息都会消失。比如，一个用户误删了使用NSUserDefaults存储密码的app，当重新安装之后，由于以前是记住密码免登录，只因为自己操作不当，接下来要进入找回密码功能，重新修改密码才能再次使用app。这对用户来说是一种相当不友好的体验。所以，正确的姿势是使用Keychain服务来存储。


<br/>

- **Keychain使用场所:**

&emsp; Keychain保存的数据不仅仅是加密过的，而且由于Keychain是存在与沙盒之外的，当应用删除之后，app存储的数据并没有被删掉，第二次安装时只要读取Keychain里的数据，即可得到以前存储的信息。

> **存储隐私信息：**

&emsp; iOS的keychain服务提供了一种安全的保存私密信息（密码，序列号，证书等）的方式，每个ios程序都有一个独立的keychain存储。相对于NSUserDefaults、文件保存等一般方式，keychain保存更为安全，而且keychain里保存的信息不会因App被删除而丢失，所以在重装App后，keychain里的数据还能使用。

> **数据共享：**

&emsp; 我们可以把KeyChain理解为一个Dictionary，所有数据都以key-value的形式存储，可以对这个Dictionary进行add、update、get、delete这四个操作。对于每一个应用来说，KeyChain都有两个访问区，私有区和公共区。私有区是一个sandbox，本程序存储的任何数据都对其他程序不可见。而要想在将存储的内容放在公共区，需要先声明公共区的名称，官方文档管这个名称叫“keychain access group”，声明的方法是新建一个plist文件，名字随便起，内容如下：

![z31](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z31.jpeg)

&emsp; “yourAppID.com.yourCompany.whatever”就是你要起的公共区名称，除了whatever字段可以随便定之外，其他的都必须如实填写。这个文件的路径要配置在 Project->build setting->Code Signing Entitlements里，否则公共区无效，配置好后，须用你正式的证书签名编译才可通过，否则xcode会弹框告诉你code signing有问题。所以，苹果限制了你只能同公司的产品共享KeyChain数据，别的公司访问不了你公司产品的KeyChain。


>**设备唯一标示存储：**

&emsp; 在iOS中，为了在苹果的打压下获取唯一标示符，开发者们也是想尽了办法，目前最好的方式就是获取IDFV，并将其存储到keychain中。IDFV是设备区别应用提供商的，一般来说可以作为应用唯一标示符。但是IDFV缺陷就是当设备删除了该所有应用提供商的app之后，IDFV值会发生变化，所以IDFV+Keychain的组合目前被经常用到，来替代UDID的作用。特别是加上Keychain的共享服务，可以使应用提供商下的所有app下获取的IDFV都不会发生变化。这一服务可以说是目前最佳的识别用户的办法。



<br/>

***
<br/>

># KeyChain Item

&emsp; Keychain内部可以保存很多的信息。每条信息作为一个单独的keychain item，keychain item一般为一个字典，每条keychain item包含一条data和很多attributes。举个例子，一个用户账户就是一条item，用户名可以作为一个attribute , 密码就是data。 keychain虽然是可以保存15000条item,每条50个attributes，但是苹果工程师建议最好别放那么多，存几千条密码，几千字节没什么问题。

&emsp; 如果把keychain item的类型指定为需要保护的类型比如password或者private key，item的data会被加密并且保护起来，如果把类型指定为不需要保护的类型，比如certificates，item的data就不会被加密。

- **Item 类型**

```

extern CFTypeRef kSecClassGenericPassword

extern CFTypeRef kSecClassInternetPassword

extern CFTypeRef kSecClassCertificate

extern CFTypeRef kSecClassKey

extern CFTypeRef kSecClassIdentityOSX_AVAILABLE_STARTING(MAC_10_7, __IPHONE_2_0);

```

![z32](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z32.png)




<br/>

***
<br/>

># Keychain的用法

- **首先导入Security.framework 。**

Keychain的API提供以下几个函数来操作Keychain

```
SecItemAdd 添加一个keychain item

SecItemUpdate 修改一个keychain item

SecItemCopyMatching 搜索一个keychain item

SecItemDelete 删除一个keychain item
```

<br/>

- **Keychain Access Group**

&emsp; Keychain通过provisioning profile来区分不同的应用，provisioning文件内含有应用的bundle id和添加的access groups。不同的应用是完全无法访问其他应用保存在Keychain的信息，除非指定了同样的access group。指定了同样的group名称后，不同的应用间就可以分享保存在Keychain内的信息。

- **Keychain Access Group的使用方法：**

&emsp; 首先要在Capabilities下打开工程的Keychain Sharing按钮。然后需要分享Keychain的不同应用添 加相同的Group名称。Xcode6以后Group可以随便命名，不需要加AppIdentifierPrefix前缀，并且Xcode会在以entitlements结尾的文件内自动添加所有Group名称，然后在每一个Group前自动加上$(AppIdentifierPrefix)前缀。虽然文档内提到还需要添加一个包含group的.plist文件，其实它和.entitlements文件是同样的作用，所以不需要重复添加。 但是每个不同的应用第一条Group最好以自己的bundleID命名，因为如果entitlements文件内已经有Keychain Access Groups数组后item的Group属性默认就为数组内的第一条Group,其实keychaingroups的名字你可以随便起，（最好使用app的buddleid），在keychain groups里面加上你想要共享哪个group的数据，想共享几个就可以填写几个，前提是这些group必须的team必须相同，也就是（AppIdentifierPrefix）必须相同！
        
![z33](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z33.png)

 这样设置group会导致存储数据失败，如果不想共享某个group的数据，建议group传入AppIdentifierPrefix+bundleiD！在其他app的keychain group中，只填入想要共享数据的group！例如上图中的MCUUId和MCaccount两个keychain group！在另一个app中加入同样的group，就能实现这两个group中的数据共享，切记代码中传入group的时候要加上appidentifier前缀，否则会存储失败！当传入的group不在entitlements文件内时，此时传入的group的值必须为AppIdentifierPrefix+bundleiD，否则会造成存储失败！查询时也可以指定group查询，但是必须使用真机测试。
 
 
 samkeychain的方法中涉及到的变量主要有三个，分别如这一小节的标题所示，是password、service、account。password、account分别保存的是密码和用户名信息。service保存的是服务的类型，就是用户名和密码是为什么应用保存的一个标志。比如一个用户可以再不同的论坛中使用相同的用户名和密码，那么service保存的信息分别标识不同的论坛。由于包名通常具有一定的唯一性，通常在程序中可以用包的名称来作为service的标识。）




<br/>

***
<br/>
	
># 参考资料：

[**keychain 详解及变化**](https://www.cnblogs.com/junhuawang/p/8194484.html)