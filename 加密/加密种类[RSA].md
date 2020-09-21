># RSA 加密

- 获取证书文件
```
cd /Users/harleyhuang/Documents/GitHub/MLC  

//用openssl生成一个privateKey.pem私钥文件
openssl genrsa -out privateKey.pem 1024

//用privateKey.pem生成一个公钥文件
openssl rsa -in privateKey.pem -pubout -out publicKey.pem


//通过privateKey.pem文件生成一个csr文件
openssl req -new -key privateKey.pem -out rsaPrivateCsr.csr


//通过csr文件生成一个crt文件
openssl x509 -req -days 3650 -in rsaPrivateCsr.csr -signkey privateKey.pem -out rsaPrivateCert.crt


//通过crt文件生成der文件,包含着公钥的信息
openssl x509 -outform der -in rsaPrivateCert.crt -out rsaPublicCert.der


//通过privateKey.pem和rsaPrivateCert.crt文件生成rsaPrivate.p12,包含私钥信息
openssl pkcs12 -export -out rsaPrivate.p12 -inkey privateKey.pem -in rsaPrivateCert.crt
```
完成以上的步骤之后，我们就可以得到一个rsaPrivate.p12私钥文件和rsaPublicCert.der公钥文件，这两个文件是我们在代码中用到的。s
**`注意`**：上述在终端中会使用到密码，在下面的代码文件中可能要用到，若是填错可能无法提取证书文件，导致崩溃。

<br/>
- 证书的使用步骤
&emsp;  下面演示了如何使用证书、密钥和信任服务去导入一个身份（identity），评估证书是否可信，判断信任失败的原因，以及信任失败后的恢复。
  顺序如：
  -  a.  导入一个 identity;
  -  b.  从导入的数据中获得证书;
  -  c.  获得用于证书评估的策略;
  -  d.  校验证书，根据指定策略评估证书是否可信;
  -  e.  测试证书中的可恢复错误;
  -  f.  判断证书是否过期;
  -  g.  改变评估条件,忽略过期证书;
  -  h.  重新评估证书.


<br/>
- 从`.p12`私钥文件提取、评估身份
&emsp;  在iOS设备上使用加密过的identity（一个密钥及其关联的证书）进行客户端认证，比如:可以把`PKCS#12`数据以受密码保护文件的方式安全地传输到这个设备上。

下面显示如何从`PKCS#12`数据中提取`identity`和`trust objects`（可信任对象），并评估其可信度。

从PKCS#12数据中提取identity和trust对象
```
//加载私钥
- (void)loadPrivateKey:(NSString *)privateKeyPath password:(NSString *)password {
    NSAssert( privateKeyPath.length != 0, @"私钥路径为空");
    
    //删除当前私钥
    if (privateKeyRef) {
        CFRelease(privateKeyRef);
    }
    
    NSData *pkcs12Data = [NSData dataWithContentsOfFile:privateKeyPath];
    CFDataRef inPkcs12Data = (__bridge CFDataRef)pkcs12Data;
    CFStringRef passwordRef = (__bridge CFStringRef)password;
    
    //从 PKCS #12 证书中提取标示和证书
    SecIdentityRef myIdentity;
    SecTrustRef myTrust;
    //以PKCS#12格式导出或以PKCS#12格式导入时使用的密码(由CFStringRef对象表示)
    const void *keys[] = {kSecImportExportPassphrase};
    const void *values[] = {passwordRef};
    //创建要传给 SecPKCS12Import的包含密码的字典
    CFDictionaryRef optionsDictionary = CFDictionaryCreate(NULL, keys, values, 1, NULL, NULL);
    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    
    //从PKCS #12数据中导出证书、密钥、信任，放到数组中
    //OSStatus: 系统框架Security/SecureTransport.h中定义的错误码
    OSStatus status = SecPKCS12Import(inPkcs12Data, optionsDictionary, &items);
    
    if (status == 0) {
        //从数组中取出第一个字典，并从这个字典中取出身份和信任
        //SecPKCS12Import方法为PKCS #12数据中的每一个条目（身份或证书）返回一个字典
        //在这个例子中被导出的身份是数组中的第一个(item #0)
        CFDictionaryRef myIdentifityAndTrust = CFArrayGetValueAtIndex(items, 0);
        myIdentity = (SecIdentityRef)CFDictionaryGetValue(myIdentifityAndTrust, kSecImportItemIdentity);
        myTrust = (SecTrustRef)CFDictionaryGetValue(myIdentifityAndTrust, kSecImportItemTrust);
    }
    
    if (optionsDictionary) {
        CFRelease(optionsDictionary);
    }
    
    if (items) {
        CFRelease(items);
    }
    
    NSAssert(status == noErr, @"提取身份和信任失败");
    
    SecTrustResultType trustResult;
    //评估指定证书和策略的信任管理是否有效
    status = SecTrustEvaluate(myTrust, &trustResult);
    NSAssert(status == errSecSuccess, @"信任评估失败");
    
    //提取私钥
    status = SecIdentityCopyPrivateKey(myIdentity, &privateKeyRef);
    NSAssert(status == errSecSuccess, @"私钥创建失败");
    
    //return status
}

```

完成上述的代码步骤后，你需要完成下面这些：
   a.  释放包含新数据的CFDataRef对象;
  b.  返回的信任对象通过调用SecTrustEvaluate 或 SecTrustEvaluateAsync评估信任。
处理信任结果;
  c.  如果信任结果是kSecTrustResultInvalid, kSecTrustResultDeny, kSecTrustResultFatalTrustFailure，你不能继续，以失败结束;
  d.  如果信任结果是kSecTrustResultRecoverableTrustFailure，你应该从信任失败中恢复。
下面的代码显示了如何从身份中获取证书，如何展示证书的信息。

**`显示证书信息`**
```
NSString *copySummaryString(SecIdentityRef identity) {
    SecCertificateRef myReturnedCertificate = NULL;
    //从证书中提取身份
    OSStatus status = SecIdentityCopyCertificate(identity, &myReturnedCertificate);
    if (status) {
        SLog(@"SecIdentityCopyCertificate failed.\n");
        return  NULL;
    }
    //从证书中获取概要信息
    CFStringRef certSummary = SecCertificateCopySubjectSummary(myReturnedCertificate);
    //转换string为NSString对象
    NSString *summaryString = [[NSString alloc] initWithString:(__bridge NSString *)certSummary];
    
    CFRelease(certSummary);
    
    return  summaryString;
}
```


<br/>
- 获取和使用持久化的钥匙串
&emsp;  当在钥匙串中添加或查找一个条目时，需要有一个持久化的引用。因为持久化引用能保证在程序从启动到能写入磁盘这段时间内，始终可用。当需要反复在钥匙串中查找条目时，使用持久化引用更加容易。以下演示了如何获取一个identity 的持久化引用。

**`获取钥匙串的持久化引用`**
```
//钥匙串的持久化引用
CFDataRef persistentRefForIdentity(SecIdentityRef identity) {
    OSStatus status = errSecSuccess;
    CFTypeRef persistent_ref = NULL;
    const void *keys[] = {kSecReturnPersistentRef, kSecValueRef};
    const void *values[] = {kCFBooleanTrue, identity};
    
    CFDictionaryRef dict = CFDictionaryCreate(NULL, keys, values, 2, NULL, NULL);
    status = SecItemAdd(dict, &persistent_ref);
    
    if (dict) {
        CFRelease(dict);
    }
    
    return  (CFDataRef)persistent_ref;
}
```

**`从持久化引用的钥匙串中检索身份对象`**
```
//从持久化引用的钥匙串中检索身份对象
SecIdentityRef identityForPersistentRef(CFDataRef persistent_ref) {
    CFTypeRef  identity_ref = NULL;
    
    const void *keys[] = {kSecClass, kSecReturnRef, kSecValuePersistentRef};
    const void *values[] = {kSecClassIdentity, kCFBooleanTrue, persistent_ref};
    
    CFDictionaryRef dict = CFDictionaryCreate(NULL, keys, values, 3, NULL, NULL);
    SecItemCopyMatching(dict, &identity_ref);
    
    if (dict) {
        CFRelease(dict);
    }
    
    return (SecIdentityRef)identity_ref;
}
```

<br/>
- 从钥匙串中查找证书
&emsp;  在钥匙串中查找使用名称识别的证书，找到一个持久化引用的条目。
```
//钥匙串中查找证书
void certificateInKeychain() {
    OSStatus status = errSecSuccess;
    CFTypeRef certificateRef = NULL;
    const char *certLabelString = "Remeo Montague";
    
    CFStringRef certLabel = CFStringCreateWithCString(NULL, certLabelString, kCFStringEncodingUTF8);
    const void *keys[] = {kSecClass, kSecAttrLabel, kSecReturnRef};
    const void *values[] = {kSecClassCertificate, certLabel, kCFBooleanTrue};
    CFDictionaryRef dict = CFDictionaryCreate(NULL, keys, values, 3, NULL, NULL);
    status = SecItemCopyMatching(dict, &certificateRef);
    
    if (status == errSecSuccess) {
        CFRelease(certificateRef);
        certificateRef = NULL;
    }
    
    if (dict) {
        CFRelease(dict);
    }
}

```


<br/>
- 获取策略对象并评估可信度
&emsp;  评估证书可信度之前，必需获取到一个证书对象的引用。可以从一个identity中提取一个证书对象，也可以从`der`证书数据中创建证书对象(使用SecCertificateCreateWithData函数)，或者从钥匙串中查找证书。
&emsp;  评估信任度的标准由信任策略（trust policy）指定，在iOS中有两种策略可用：`Basic X509`和`SSL`。可以用`SecPolicyCreateBasicX509`或者`SecPolicyCreateSSL`函数获取策略对象。

```
//加载公钥
- (void) loadPublicKey:(NSString *)publicKeyPath {
    //NSAssert:捕获一个错误, 让程序崩溃, 同时报出错误提示
    NSAssert(publicKeyPath.length != 0, @"公钥路径为空");
    
    //删除当前公钥
    if (publicKeyRef) {
        CFRelease(publicKeyRef);
    }
    
    //根据公钥证书der文件制作成证书对象
    NSData *certificateData = [NSData dataWithContentsOfFile:publicKeyPath];
    //根据二进制数据生成一个SecCertificateRef类型的证书
    //证书对象引用
    SecCertificateRef certificateRef = SecCertificateCreateWithData(kCFAllocatorDefault, (__bridge CFDataRef)certificateData);
    NSAssert(certificateRef != NULL, @"公钥文件错误");
    
    //信任策略 SecPolicyCreateBasicX509 用来获取取策略对象
    SecPolicyRef policyRef = SecPolicyCreateBasicX509();
    //评估信任的对象
    SecTrustRef trustRef;
    

    //用证书和策略创建信任对象（trust）。
    //如果存在中间证书或者锚证书，应把这些证书都包含在certificate数组中并传递给SecTrustCreateWithCertificates函数。
    //这样会加快评估的速度
    OSStatus status = SecTrustCreateWithCertificates(certificateRef, policyRef, &trustRef);
    NSAssert(status == errSecSuccess, @"创建信任管理对象失败");
    
    //信任结果
    SecTrustResultType trustResult;
    //评估一个信任对象：评估指定证书和策略的信任管理是否有效
    status = SecTrustEvaluate(trustRef, &trustResult);
    NSAssert(status == errSecSuccess, @"信任评估失败");
    
    //评估之后返回公钥子证书
    publicKeyRef = SecTrustCopyPublicKey(trustRef);
    NSAssert(publicKeyRef != NULL, @"公钥创建失败");
    
    //处理信任结果（trust result）。如果信任结果是kSecTrustResultInvalid，kSecTrustResultDeny，kSecTrustResultFatalTrustFailure，你无法进行处理。如果信任结果是kSecTrustResultRecoverableTrustFailure，你可以恢复这个错误
    if (trustResult == kSecTrustResultRecoverableTrustFailure) {
        //从信任中恢复
        recoverFromTrustFailure(trustRef);
        //NSAssert(trustResult == kSecTrustResultRecoverableTrustFailure, @"信任失败");
    }
    
    if (certificateRef) {
        CFRelease(certificateRef);
    }
    if (policyRef) {
        CFRelease(policyRef);
    }
    if (trustRef) {
        CFRelease(trustRef);
    }
}
```


<br/>

- 从信任失败中恢复
&emsp;  信任评估的结果有多个，这取决于证书链中所有证书是否都能找到并全都有效，以及用户对这些证书的信任设置是什么。信任结果怎么处理则由你的程序来决定。如果信任结果是`kSecTrustResultConfirm`，你可以显示一个对话框，询问用户是否允许继续。

&emsp;  信任结果`kSecTrustResultRecoverableTrustFailure`的意思是：信任被否决，但可以通过改变设置获得不同结果。例如，如果证书签发过期，你可以改变评估日期以判断是否证书是有效的同时文档是已签名的。注意 `CFDateCreate`函数使用绝对时间。你可以用`CFGregorianDateGetAbsoluteTime`函数把日历时间转换为绝对时间。
```
///信任失败恢复
void recoverFromTrustFailure(SecTrustRef myTrust) {
    SecTrustResultType trustResult;
    //评估证书可信度。参考“获取策略对象并评估可信度”
    OSStatus status = SecTrustEvaluate(myTrust, &trustResult);
    
    CFAbsoluteTime trustTime, currentTime, timeIncrement, newTime;
    CFDateRef newDate;
    //检查信任评估结果是否是可恢复的失败
    if (trustResult == kSecTrustResultRecoverableTrustFailure) {
        //取得证书的评估时间（绝对时间）。如果证书在评估时已经过期了，则被认为无效
        trustTime = SecTrustGetVerifyTime(myTrust);
        //设置时间的递增量为1年（以秒计算）
        timeIncrement = 31536000;
        //取得当前时间的绝对时间
        currentTime = CFAbsoluteTimeGetCurrent();
        //设置新时间（第2次评估的时间）为当前时间减一年
        newTime = currentTime - timeIncrement;
        
        //检查评估时间是否大于1年前（最近一次评估是否1年前进行的）。
        //如果是，使用新时间（1年前的时间）进行评估，看证书是否在1年前就已经过期
        if (trustTime - newTime) {
            //把新时间转换为CFDateRef。
            //也可以用NSDate，二者是完全互通的，方法中的NSDate*参数，可以用CFDateRef进行传递；反之亦可
            newDate = CFDateCreate(NULL, newTime);
            //设置信任评估时间为新时间（1年前）
            SecTrustSetVerifyDate(myTrust, newDate);
            //再次进行信任评估。如果证书是因为过期（到期时间在1年内）导致前次评估失败，那么这次评估应该成功
            status = SecTrustEvaluate(myTrust, &trustResult);
        }
    }
    
    //再次检查评估结果。如果仍不成功，则需要做更进一步的操作，比如提示用户安装中间证书，或则友好地告知用户证书校验失败
    if (trustResult != kSecTrustResultProceed) {
        
    }
}
```


<br/>
- 加密和解密数据
&emsp;  证书，密钥和信任API包含了生产不对称密钥并用于数据加密和解密的函数。如果您想要使用此功能来对你不想在备份数据中访问的数据进行加密。或者，你可能想使用公钥/私钥在你的iOS应用和桌面应用间通过网络发送加密数据。下面的代码都使用了cocoa对象（如NSMutableDictionary），而不是像本章其他示例那样使用了Core Foundation对象（如CFMutableDictionaryRef），Cocoa对象和对应的Core Foundation完全相同，免费桥接。如果在方法中有个`NSMutableDictionary *`参数，你可以转化为`CFMutableDictionaryRef`，方法中的`CFMutableDictionaryRef`参数，你可以转换为`NSMutableDictionary`实例。

**`生成密钥对`**
```
///生成密钥对
- (void) generateKeyPair:(NSUInteger)keySize {
    OSStatus  sanityCheck = noErr;
    publicKeyRef = NULL;
    privateKeyRef = NULL;
    
    NSAssert1(keySize == 512 || keySize == 1024 || keySize == 2048, @"密钥尺寸无效 %tu", keySize);
    
    //删除当前密钥对
    [self deleteAsymmetricKeys];
    
    //为SecKeyGeneratePair方法中的属性分配容器字典
    NSMutableDictionary *privateKeyAttr = [[NSMutableDictionary alloc] init];
    NSMutableDictionary *publicKeyAttr  = [[NSMutableDictionary alloc] init];
    NSMutableDictionary *keyPairAttr = [[NSMutableDictionary alloc] init];
    
    //定义的标识符字符串的NSData对象
    self.publicTag = [NSData dataWithBytes:publicKeyIdentifier length:strlen((const char *) publicKeyIdentifier)];
    self.privateTag = [NSData dataWithBytes:privateKeyIdentifier length:strlen((const char *) privateKeyIdentifier)];
    
    //设置密钥对的顶级字典
    //设置密钥对类型属性为RSA
    [keyPairAttr setObject:(__bridge id)kSecAttrKeyTypeRSA forKey:(__bridge id)kSecAttrKeyType];
    //设置密钥对长度为keySize(可以为1024)字节
    [keyPairAttr setObject:[NSNumber numberWithUnsignedInteger:keySize] forKey:(__bridge id)kSecAttrKeySizeInBits];
    
    //设置私钥字典
    //设置私钥的持久化属性（即是否存入钥匙串）为YES
    [privateKeyAttr setObject:[NSNumber numberWithBool:YES] forKey:(__bridge id)kSecAttrIsPermanent];
    //identifier放到私钥的dictionary中
    [privateKeyAttr setObject:self.privateTag forKey:(__bridge id)kSecAttrApplicationTag];
    
    //设置公钥字典
    //设置公钥的持久化属性（即是否存入钥匙串）为YES
    [publicKeyAttr setObject:[NSNumber numberWithBool:YES] forKey:(__bridge id)kSecAttrIsPermanent];
    //identifier放到公钥的dictionary中
    [publicKeyAttr setObject:self.publicTag forKey:(__bridge id)kSecAttrApplicationTag];
    
    //设置顶级字典属性
    //把私钥的属性集（dictionary）加到密钥对的属性集（dictionary）中
    [keyPairAttr setObject:privateKeyAttr forKey:(__bridge id)kSecPrivateKeyAttrs];
    //把公钥的属性集（dictionary）加到密钥对的属性集（dictionary）中
    [keyPairAttr setObject:publicKeyAttr forKey:(__bridge id)kSecPublicKeyAttrs];
    
    
    //产生密钥对
    sanityCheck = SecKeyGeneratePair((__bridge CFDictionaryRef)keyPairAttr, &publicKeyRef, &privateKeyRef);
    NSAssert((sanityCheck == noErr && publicKeyRef != NULL && privateKeyRef != NULL), @"生成密钥对失败");
    
    //释放不用的对象内存
    if (publicKeyRef) {
        CFRelease(publicKeyRef);
    }
    
    if (privateKeyRef) {
        CFRelease(privateKeyRef);
    }

}
```
&emsp;  可以将您的公钥发送给任何人,谁可以使用它来加密数据。如果你保持你的私钥安全,那么只有你能够解密数据。以下代码示例展示了如何使用公钥加密数据。这可以在设备上生成一个公钥或者从发送给你或钥匙链中的证书中提取公钥。您可以使用`SecTrustCopyPublicKey`函数来从证书中提取公钥。在下面的代码中,假定密钥已经在设备上生成并放置在钥匙串中。

**`用公钥加密数据`**
```
- (NSData *)encryptWithPublicKey {
    OSStatus status = noErr;
    
    size_t cipherBufferSize;
    // 1 为加密文本分配缓冲区
    uint8_t *cipherBuffer;

    // [cipherBufferSize]
    // 2 指定要加密的文本
    const uint8_t dataToEncrypt[] = "the quick brown fox jumps "
    "over the lazy dog\0";
    size_t dataLength = sizeof(dataToEncrypt)/sizeof(dataToEncrypt[0]);

    // 3 定义SecKeyRef，用于公钥。
    SecKeyRef publicKey = NULL;

    // 4 定义NSData对象，存储公钥的identifier（见列表 2-8 的第1、3、8步），该id在钥匙串中唯一。
    NSData * publicTag = [NSData dataWithBytes:publicKeyIdentifier
                                        length:strlen((const char *)publicKeyIdentifier)];
    // 5 定义dictionary，用于从钥匙串中查找公钥。
    NSMutableDictionary *queryPublicKey =
    [[NSMutableDictionary alloc] init];

    // 6 设置dictionary的键－值属性。
    //属性中指定，钥匙串条目类型为“密钥”，
    //条目identifier为第4步中指定的字符串，密钥类型为RSA，函数调用结束返回查找到的条目引用。
    [queryPublicKey setObject:(__bridge id)kSecClassKey forKey:(__bridge id)kSecClass];
    [queryPublicKey setObject:publicTag forKey:(__bridge id)kSecAttrApplicationTag];
    [queryPublicKey setObject:(__bridge id)kSecAttrKeyTypeRSA forKey:(__bridge id)kSecAttrKeyType];
    [queryPublicKey setObject:[NSNumber numberWithBool:YES] forKey:(__bridge id)kSecReturnRef];


    // 7 调用SecItemCopyMatching函数进行查找。
    status = SecItemCopyMatching
    ((__bridge CFDictionaryRef)queryPublicKey, (CFTypeRef *)&publicKey);


    cipherBufferSize = SecKeyGetBlockSize(publicKey);
    cipherBuffer = malloc(cipherBufferSize);


    if (cipherBufferSize < sizeof(dataToEncrypt)) {
        // Ordinarily, you would split the data up into blocks
        // equal to cipherBufferSize, with the last block being
        // shorter. For simplicity, this example assumes that
        // the data is short enough to fit.
        printf("Could not decrypt.  Packet too large.\n");
        return NULL;
    }

    // 8 加密数据， 返回结果用PKCS1格式对齐。
    status = SecKeyEncrypt(    publicKey,
                           kSecPaddingPKCS1,
                           dataToEncrypt,
                           (size_t) dataLength,
                           cipherBuffer,
                           &cipherBufferSize
                           );

    //  Error handling
    //  Store or transmit the encrypted text

    if (publicKey) CFRelease(publicKey);

    NSData *encryptedData = [NSData dataWithBytes:cipherBuffer length:dataLength];

    free(cipherBuffer);

    return encryptedData;
}

```

&emsp;  下面的代码示例显示了如何解密数据。这个示例使用用来加密数据的公钥对应的私钥,并假设您已经在前面的示例中创建的密文。从钥匙串中获取私钥的技术跟前面的示例中获取公钥的技术相同 。

**`私钥解密`**
```
///私钥解密
- (void) decryptWithPrivateKey: (NSData *)dataToDecrypt {
    OSStatus status = noErr;
    
    size_t cipherBufferSize = [dataToDecrypt length];
    uint8_t *cipherBuffer = (uint8_t *)[dataToDecrypt bytes];

    size_t plainBufferSize;
    uint8_t *plainBuffer;

    SecKeyRef privateKey = NULL;

    NSData * privateTag = [NSData dataWithBytes:privateKeyIdentifier
                                         length:strlen((const char *)privateKeyIdentifier)];

    NSMutableDictionary *queryPrivateKey = [[NSMutableDictionary alloc] init];

    // 1 设置放钥匙串中私钥的字典
    [queryPrivateKey setObject:(__bridge id)kSecClassKey forKey:(__bridge id)kSecClass];
    [queryPrivateKey setObject:privateTag forKey:(__bridge id)kSecAttrApplicationTag];
    [queryPrivateKey setObject:(__bridge id)kSecAttrKeyTypeRSA forKey:(__bridge id)kSecAttrKeyType];
    [queryPrivateKey setObject:[NSNumber numberWithBool:YES] forKey:(__bridge id)kSecReturnRef];

    // 2 从钥匙串中找到私钥
    status = SecItemCopyMatching
    ((__bridge CFDictionaryRef)queryPrivateKey, (CFTypeRef *)&privateKey);

    //  Allocate the buffer
    plainBufferSize = SecKeyGetBlockSize(privateKey);
    plainBuffer = malloc(plainBufferSize);

    if (plainBufferSize < cipherBufferSize) {
        // Ordinarily, you would split the data up into blocks
        // equal to plainBufferSize, with the last block being
        // shorter. For simplicity, this example assumes that
        // the data is short enough to fit.
        printf("Could not decrypt.  Packet too large.\n");
        return;
    }

   // 3 解密数据
    status = SecKeyDecrypt(    privateKey,
                           kSecPaddingPKCS1,
                           cipherBuffer,
                           cipherBufferSize,
                           plainBuffer,
                           &plainBufferSize
                           );

    // 4 释放内存
    if(privateKey) CFRelease(privateKey);
}
```

<br/>
***
<br/>
**`参考资料`**
######[安全之RSA加密/生成公钥、秘钥 pem文件](https://www.cnblogs.com/xuan52rock/p/11023423.html)

