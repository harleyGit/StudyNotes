[SwiftEOS](https://github.com/ProChain/SwiftyEOS)是一个用于与EOS交互的开源框架，用Swift编写,可以在iOS和macOS上使用。


-  **`特点：`**
    -  EOS密钥对生成；
    -  私钥导入；
    -  签名哈希；
    -  基本的RPC API（链/历史）可查询客户端；
    -  交易（EOS token 转账）；
    -  帮助类处理iOS上的脱机钱包；
    -  在iOS上加密/解密导入私钥。

- **`使用`**
a.  将Libraries和Sources(只保留Crypto和Key2个文件夹)文件夹复制到项目中，不需要main.swift;
b.  如果不是针对iOS平台，请删除Sources/Utils/iOS;
c.  将Libraries/include添加到Header搜索路径中;

![添加到搜索路径](https://upload-images.jianshu.io/upload_images/2959789-87928ca23868523c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

d.  将Libraries/include/Bridging-Header.h设置为Objective-C Bridging Header。如果你有自己的bridging header，请复制该文件中的所有导入内容并粘贴到你自己的文件中；

![导入的文件，有的<>需要换为“”，否则编译不通过](https://upload-images.jianshu.io/upload_images/2959789-1d36a8dbebb227f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


e.  编译然后等结果。


<br/>
***
<br/>
># 密钥对的产生
```
private func encryptorTest() {
    
    ///生成随机密钥对
    let (pk, pub) = generateRandomKeyPair(enclave: .Secp256k1)
    guard let privateKey = pk, let publicKey = pub else {
        return
    }
    print("private key: \(privateKey.wif())")
    print("public key : \(publicKey.wif())")
    
    //PVT_K1_和PUB_K1_前缀是标准密钥表示的一部分。但是EOS系统和SwiftyEOS也支持旧方式
    print("去除PVT_K1_的 private key: \(privateKey.rawPrivateKey())")
    print("去除PUB_K1_的public key : \(publicKey.rawPublicKey())")
    
    
    ///导入现有密钥
    //导入不带分隔符和前缀的密钥
    guard let importedPrivateKey = try? PrivateKey(keyString: privateKey.rawPrivateKey()) else { return }
    let importedPulicKey = PublicKey(privateKey: importedPrivateKey)
    print("导入带前缀的密钥：\(importedPulicKey)")
    
    //导入带分隔符和前缀的密钥
    guard let importedPk = try? PrivateKey(keyString: privateKey.wif()) else { return }
    let importedPub = PublicKey(privateKey: importedPk)
    print("导入不带前缀的密钥：\(importedPub)")
    
    
    
    ///RPC API
    
    
}

```
![打印结果](https://upload-images.jianshu.io/upload_images/2959789-ba171e6494bcacea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

