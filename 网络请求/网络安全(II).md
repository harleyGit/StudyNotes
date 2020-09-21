># 客户端证书认证
&emsp;  从服务器返回的证书被编码成了 `PKCS #12(.12)`文件格式，这是由 RSA Laboratories 发布的一种常用标准。

```
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    
    // successful authentication
    if (self.statusCode == 200) {
        
        // unpack service response
        NSError *error = nil;
        NSDictionary *response = [NSJSONSerialization JSONObjectWithData:self.responseData 
                                                                 options:0 
                                                                   error:&error];
        
        NSString *token = [response objectForKey:@"token"];
        [Model sharedModel].token = token;
        
        // create our client certificate if necessary, and store in the credential store
        if (_registerDevice == YES) {
            
            NSString *certString = [response objectForKey:@"certificate"];
            //64位解码
            NSData *certData = [[NSData alloc]initWithBase64EncodedString:certString];
        
            SecIdentityRef identity = NULL;
            SecCertificateRef certificate = NULL;
            [Utils identity:&identity andCertificate:&certificate fromPKCS12Data:certData withPassphrase:_passphrase];
            
            // store the certificate for future authentication challenges
            if (identity != NULL) {
                
                // store the certificate and identity in the keychain
                NSArray *certArray = [NSArray arrayWithObject:(__bridge id)certificate];
                NSURLCredential *credential = [NSURLCredential credentialWithIdentity:identity
                                                                         certificates:certArray
                                                                          persistence:NSURLCredentialPersistencePermanent];
                
                NSURLProtectionSpace *certSpace = [[NSURLProtectionSpace alloc] initWithHost:@"yourbankingdomain.com"
                                port:443
                            protocol:NSURLProtectionSpaceHTTPS
                               realm:@""
                authenticationMethod:NSURLAuthenticationMethodClientCertificate];
                //NSURLCredentialStorage：管理凭据存储的单例（共享对象）
                //sharedCredentialStorage：返回共享的URL证书存储仓库
                //setDefaultCredential: forProtectionSpace: 给指定保护空间的设置默认凭据，如果仓库在指定的保护空间中不包含凭据，那么此证书将被添加。
                [[NSURLCredentialStorage sharedCredentialStorage] setDefaultCredential:credential
                                                             forProtectionSpace: certSpace];

                // we got what we needed, save it for later
                [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"registeredDevice"];
                [[NSUserDefaults standardUserDefaults] synchronize];
            }
        }

        
        // issue command to retrieve users accounts
        [[Model sharedModel] fetchAccounts];
        
        // alert observers
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:kNormalLoginSuccessNotification object:nil];
        });
        
    // authentication failed
    } else {
        
        // alert observers
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:kNormalLoginFailedNotification object:nil];
        });
    }

    [self finish];
}
```

<br/>
`Utils.m 文件`
&emsp;  使用 Security Framework 中的 SecPKCS12Import() 函数导入身份和信任，然后提取出证书的过程。
```
+ (void)identity:(SecIdentityRef*)identity 
  andCertificate:(SecCertificateRef*)certificate 
  fromPKCS12Data:(NSData*)certData 
  withPassphrase:(NSString*)passphrase {
    
    // adapted from https://developer.apple.com/library/IOs/#documentation/Security/Conceptual/CertKeyTrustProgGuide/iPhone_Tasks/iPhone_Tasks.html#//apple_ref/doc/uid/TP40001358-CH208-SW13
    
    // bridge our import data to foundation objects
    CFStringRef importPassphrase = (__bridge CFStringRef)passphrase;
    CFDataRef importData = (__bridge CFDataRef)certData;
    
    // create dictionary of options for the PKCS12 import
    const void *keys[] = { kSecImportExportPassphrase };
    const void *values[] = { importPassphrase };
    CFDictionaryRef importOptions = CFDictionaryCreate(NULL, keys,
                                                       values, 1,
                                                       NULL, NULL);
    
    // create array to store our import results
    CFArrayRef importResults = CFArrayCreate(NULL, 0, 0, NULL);
    OSStatus pkcs12ImportStatus = errSecSuccess;
    
    //返回PKCS #12格式的blob中的身份和证书
    pkcs12ImportStatus = SecPKCS12Import(importData,
                                         importOptions,
                                         &importResults);
    
    // check if we import successfully
    if (pkcs12ImportStatus == errSecSuccess) {
        CFDictionaryRef identityAndTrust = CFArrayGetValueAtIndex (importResults, 0);
        
        // retrieve the identity from the certificate we imported
        const void *tempIdentity = NULL;
        tempIdentity = CFDictionaryGetValue (identityAndTrust,
                                             kSecImportItemIdentity);
        *identity = (SecIdentityRef)tempIdentity;
        
        // extract the certificate from the identity
        SecCertificateRef tempCertificate = NULL;
        OSStatus certificateStatus = errSecSuccess;
        certificateStatus = SecIdentityCopyCertificate (*identity,
                                                        &tempCertificate);
        *certificate = (SecCertificateRef)tempCertificate;
    }
    
    // clean up
    if (importOptions) {
        CFRelease(importOptions);
    }
}

```


<br/>
***
<br/>
># 使用哈希与加密确保消息完整性


![负载结构的加密](https://upload-images.jianshu.io/upload_images/2959789-cad7054c47df4462.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- JSON 负载结构
```
{
    "mac": "Message Authentication Code",
    "iv": "Initialization Vector",
    "payload": {
        "toAccount": "123123456456",
        "fromAccount": "654654123123",
        "amount": 23.23,
        "transferDate": "2012-02-27",
        "transferNotes": "Book advance to savings."
    }
}
```
payload 属性是请求中唯一被加密的元素。


<br/>
- 哈希
  对于给定的数据块，密码哈希与摘要会生成固定大小的位序列(对一些字符串进行摘要，生成不规则字符串)。使用哈希值可以简化数据块的比较和排序，应用场景为：追踪文件变更、下载校验、数据混淆以及数据库存储、验证请求数据的完整性。摘要如：`MD5、 SHA-1、 SHA-256`


<br/>
- 消息认证码
  消息认证码(MAC)可以检测到负载是否被修改验证其真实性，通过对请求来的数据(或预定好的请求数据子集)生成哈希值，将哈希值与随负载一同发送的预先计算好的MAC进行对比。


<br/>
- 加密

**`网络请求加密`**
```
- (void)start {
    
    if (![NSThread isMainThread]) {
        [self performSelectorOnMainThread:@selector(start) 
                               withObject:nil 
                            waitUntilDone:NO];
        return;
    }
    
    [self willChangeValueForKey:@"isExecuting"];
    isExecuting = YES;
    [self didChangeValueForKey:@"isExecuting"];
    
    // alert observers of the failed attempt
    dispatch_async(dispatch_get_main_queue(), ^{
        [[NSNotificationCenter defaultCenter] postNotificationName:kFundsTransferStartNotification object:nil];
    });
    
    // clean up any nil values
    _toAccount = _toAccount ? _toAccount : @"";
    _fromAccount = _fromAccount ? _fromAccount : @"";
    NSString *amount = _amount ? [NSString stringWithFormat:@"%.02f",_amount] : @"0.00";
    NSString *date = _transferDate ? [NSString stringWithFormat:@"%@", _transferDate] :[NSString stringWithFormat:@"%@", [NSDate date]]; 
    _transferDate = _transferDate ? _transferDate : [NSDate date];
    _transferNotes = _transferNotes ? _transferNotes : @"";
    
    // build our transfer data
    NSDictionary *transfer = [NSDictionary dictionaryWithObjectsAndKeys:
                              _toAccount, @"toAccount",
                              _fromAccount, @"fromAccount",  
                              date, @"transferDate",
                              _transferNotes, @"transferNotes",
                              amount, @"amount", nil];
    
    // create the json representation of our transfer data using ios5 API
    NSError *error = nil;
    NSData *transferData = [NSJSONSerialization dataWithJSONObject:transfer
                                                           options:0 error:&error];
    NSString *transferString = [[NSString alloc] initWithData:transferData
                                                     encoding:NSUTF8StringEncoding];
    
    // generate our initialization vector
    //生成随机数：创建加密安全的随机字节数组
    NSData *iv = [Utils blockInitializationVectorOfLength:kCCBlockSizeAES128];

    // because we generate random bytes, it may not be proper UTF8 encoding.
    // because of this, we can't just init a string with data. instead,
    // we encode it for transmission. the IV is then decoded on the service
    // and used in the decryption process
    NSString *ivString = [iv base64Encoding];
    
    // encrypt our transfer data using AES and a randomly generated IV (this IV needs to be what we send to the service)
    //aes 加密
    NSString *encryptedString = [transferString encryptedWithAESUsingKey:kAESEncryptionKey 
                                                                   andIV:iv];
    
    // calculate our message authentication code
    NSString *macCandidate = [NSString stringWithFormat:@"%@%@%@%@",
                              _toAccount,       // to account
                              _fromAccount,     // from account
                              amount,           // amount
                              _transferDate];   // transfer date
    
    //使用hmac来生成摘要
    NSString *mac = [macCandidate hmacWithKey:kMACKey];
    
    // construct our payload
    NSMutableDictionary *payload = [NSMutableDictionary dictionaryWithObjectsAndKeys:
                                    ivString, @"iv", 
                                    mac, @"mac", 
                                    encryptedString, @"payload", nil];
    
    // build our funds transfer request and issue it
    NSString *url;
    url = [kResourceBaseURL stringByAppendingString:kEndpoint];
    
    // build our funds transfer request and issue it
    NSMutableURLRequest *request = [self buildRequestWithURL:url
                                                  httpMethod:kHTTPMethod
                                                     payload:payload
                                                     timeout:kOperationTimeout
                                               signingOption:NJRequestSigningOptionHeader];
    
    connection = [[NSURLConnection alloc] initWithRequest:request
                                                 delegate:self];
    
    [UIApplication sharedApplication].networkActivityIndicatorVisible = YES;
    
}



```

**`Utils.m`**
生成随机数
```
+ (NSData*)blockInitializationVectorOfLength:(size_t)ivLength {
    // default to AES block size
    if (ivLength == 0) {
        //使用aes块长度作为默认值
        ivLength = kCCBlockSizeAES128;
    }
    
    NSMutableData *iv = [NSMutableData dataWithLength:ivLength];
    
    int ivResult = SecRandomCopyBytes(kSecRandomDefault,
                                      ivLength,
                                      iv.mutableBytes);
    
    if (ivResult == noErr) {
        return iv;
    }
    
    return nil;
}
```


<br/>
**`网络请求解密`**
```
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    
    // unpack service response
    NSError *error = nil;
    NSDictionary *response = [NSJSONSerialization JSONObjectWithData:self.responseData
                                                             options:0
                                                               error:&error];
    
    // decrypt the payload
    NSString *inboundMAC = [response objectForKey:@"mac"];
    NSData *ivData = [[response objectForKey:@"iv"] dataUsingEncoding:NSUTF8StringEncoding];
    NSString *encryptedResponse = [response objectForKey:@"payload"];
    NSString *decryptedResponse = [encryptedResponse decryptedWithAESUsingKey:kAESEncryptionKey
                                                                        andIV:ivData];
    
    if (decryptedResponse != nil) {
        // create our JSON array of account info
        NSError *accountError = nil;
        NSArray *accounts = [NSJSONSerialization JSONObjectWithData:[decryptedResponse dataUsingEncoding:NSUTF8StringEncoding]
                                                            options:0
                                                              error:&accountError];
        
        // validate the MAC
        NSString *generatedMAC = [decryptedResponse hmacWithKey:kMACKey];
        if ([inboundMAC isEqualToString:generatedMAC]) {
            
            // validation passed, create accounts
            for (NSDictionary *accountData in accounts) {
                Account *account = [[Account alloc] initWithData:accountData];
                [[Model sharedModel].accounts addObject:account];
            }
            
            // post notification
            dispatch_async(dispatch_get_main_queue(), ^{
                [[NSNotificationCenter defaultCenter] postNotificationName:kAccountOperationSuccessful object:nil];
            });
            
        } else {
            // dispatch alert view message to the main thread
            dispatch_async(dispatch_get_main_queue(), ^{
                [[[UIAlertView alloc] initWithTitle:@"Unsecure Connection"
                                            message:@"We're unable to verify account data validity. Please check your network connection and try again."
                                           delegate:nil
                                  cancelButtonTitle:@"OK"
                                  otherButtonTitles:nil] show];
            });
            
            // post notification
            dispatch_async(dispatch_get_main_queue(), ^{
                [[NSNotificationCenter defaultCenter] postNotificationName:kAccountOperationError object:nil];
            });
            
        }
    } else {
        // dispatch alert view message to the main thread
        dispatch_async(dispatch_get_main_queue(), ^{
            [[[UIAlertView alloc] initWithTitle:@"Unsecure Connection"
                                        message:@"We're unable to interpret account data validity. Please check your network connection and try again."
                                       delegate:nil
                              cancelButtonTitle:@"OK"
                              otherButtonTitles:nil] show];
        });
        
    }
    
    [self finish];
}
```













