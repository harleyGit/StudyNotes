

- **ApplePay内购集成**
	- 沙箱测试账号创建
- **后记**
	- 沙盒账号注意事项
	- 关于证书的问题
- **苹果内购两种模式**
	- 服务器模式
	- 本地模式
- **其他支付**
- **参考资料**




<br/>

***
<br/>

[Apple Pay 内购官方文档](https://help.apple.com/app-store-connect/#/devae49fb316)

># **ApplePay内购集成**

![内购配置图](https://upload-images.jianshu.io/upload_images/2959789-19fe47b5f5540bdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

&emsp;  在Apple 内购中，一个内购项目需要申请一个，比如：我申请月、年、终身VIP，就需要增加3个内购项目。如下图：

![3个内购项目申请月、年、终身](https://upload-images.jianshu.io/upload_images/2959789-3eb73663fe9f234c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  在每个内购项目中，其定价只能按照苹果给的来进行设值，自己不能随便进行设值的。如下图：

![内购价格设值](https://upload-images.jianshu.io/upload_images/2959789-43e4c55c7aac623f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
<br/>

- **沙箱测试账号创建**

![选择箭头所指的选项](https://upload-images.jianshu.io/upload_images/2959789-057184762f6d01e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![创建沙箱账号](https://upload-images.jianshu.io/upload_images/2959789-819a980017a0e86f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> `沙盒账号使用流程`

①  在iPhone上安装测试包（必须是adhoc签名证书或者develop签名证书打的包，不能是从App Store上下载的）；

②  退出自己iPhone的App Store账号（因为我们需要使用沙盒账号登录）：

&emsp;  a.打开App Store应用首页滑到最下方--选中AppleID--注销;
      
&emsp;   b.设置--iTunes Store与App Store--选中AppleID--注销;

&emsp; &emsp;   在商城退出自己的Apple 账号后，不要使用刚才在沙盒申请的账号(沙盒账号是一个假的AppleID账号，不能直接登录的。)，那样是登录不进去的。否则会出现以下错误：
![无法使用沙盒测试账号测试](https://upload-images.jianshu.io/upload_images/2959789-97c7ad8e09466ef6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  `沙盒账号`是在安装测试包内购买商品时，系统会让你进行登录，这时我们点击“使用现有的AppleID”就可以输入刚才创建好的沙盒测试账号进行登录了。

> `测试使用沙盒测试账号登录使用`

&emsp;  在测试包里面购买商品VIP时，系统会让你进行登录，这里我们点击“使用现有的AppleID”就可以输入刚才创建好的沙盒测试账号进行登录了。

![使用申请的沙盒账号登录](https://upload-images.jianshu.io/upload_images/2959789-908bbae69d3f64a8.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

&emsp;  输入账号之后，有可能会出现提示`" 此Apple ID 只能在iTunes Store 中国店面购买，您将被跳转至该页面"`，点击确定之后会跳转到App Store，导致这次购买失败。没关系，我们再次回到测试包，然后购买商品就好了。

&emsp;  原因：出现提示的原因：因为AppleID是分地区的。之前我们创建沙盒账号的时候就看到了，需要选择地区。App Store也是分地区的，对应的AppleID只能在App Store对应的地区进行下载和购买东西。我们刚才创建的[harleyTest@123.com](这个账号的地区是中国，所以只能在中国店面登录。由于我之前的登录的账号越南的，所以此时AppStore店面是越南店面。所以我们这次登录，系统会跳转到AppStore应用将店面切换到中国。另外，App Store应用切换地区的时候，会报【Your request produced an error】。这个不需要管。

<br/>

**`点击购买视频VIP之后，成功的会出现如下相应提示`**

![使用申请的沙盒账号密码进行付款](https://upload-images.jianshu.io/upload_images/2959789-21350e55f0f0c6d1.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意事项：在App Store Connect上创建商品了之后，除了需要填商品ID，商品名称，商品描述，价格等之外，还要上传一张图片，图片就是上面这个界面。


<br/>

***
<br/>

># 后记

<br/>

- **`沙盒账号注意事项:`**
&emsp; a. BudleID，证书，商品ID等内容一致，才能进行接下来的储值测试;

&emsp; b. 测试设备需要使用不越狱的真机（越狱机不能进行沙盒储值，模拟器也不能进行沙盒储值）

&emsp; c. 沙盒账号是不能直接在App Store进行登录的，只能在点击了购买商品之后，在弹出的登录框进行登录。

&emsp; d. 真实的AppleID不能在adhoc证书和develop证书打出来的包进行沙盒储值测试，所以在沙盒测试之前，需要退出真实的AppleID账号

&emsp; e. 从App Store上面下载的包不能使用沙盒账号进行储值


<br/>

- **`关于证书的问题：`**
&emsp; a .使用develop签名证书和adhoc签名证书打的ipa包，我把他们叫做测试包，测试包只能使用沙盒账号进行储值，不能使用真实的AppleID进行储值
&emsp; b.从App Store应用下载的包，我把他们叫做线上包，线上包只能使用真实的AppleID进行储值，不能使用沙盒账号进行储值


<br/>

- **`小窍门：`**

&emsp; ① 平常我们上传包的时候是打包了ipa包之后，使用Xcode里面的Application Loader应用上传ipa包的；
&emsp; ① 虽然很多人上传包使用的是appstore的签名证书，但是，其实使用adhoc的证书打包的ipa包也是可以正常上传并且送审上线的。我平常就是用adhoc的证书打包成ipa包，给测试妹子测试，测试完直接用这个包上传送审了。😆。


<br/>

***
<br/>

>#  苹果内购两种模式

- **`服务器模式`**


>a.  调用服务器接口创建一个商品的订单;

>b.  请求Apple的商品列表;

>c.  选取商品调用苹果支付;

>d.  支付成功（会返回凭证）;

>e.  把支付成功的返回凭证上传到APP服务器（带上订单的ID，有利于后台判断是哪个订单支付成功）;

>f.  APP服务器保存该凭证等数据并像苹果服务器发起凭证验证，验证成功则发送商品。


<br/>

- **`本地模式`**

>a.  请求Apple的商品列表;

>b.  选取商品调用苹果支付;

>c.  支付成功（会返回凭证）;

>d.  把凭证与商品发送状态保存到一个本地的数据库;

>e.  app调用apple服务器的验证API;

>f.  验证成功发送商品并改变数据库的物品发送状态;


<br/>

- **`核心代码`**

```
//导入文件：
#import <StoreKit/StoreKit.h>
#pragma mark -- ApplePay
//产品ID号
#define ApplePayInPurchase          @"com.tianYingXinXi.XXX"

//遵守SKPaymentTransactionObserver、SKProductsRequestDelegate协议
@interface VIPMemberController ()<UITextViewDelegate, SKPaymentTransactionObserver, SKProductsRequestDelegate>



//设置支付服务
[[SKPaymentQueue defaultQueue] addTransactionObserver:self];

[self applePayWithVIP];

//结束后一定要销毁
- (void)dealloc
{
    [[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
}

- (void) applePayWithVIP {
    if ([SKPaymentQueue canMakePayments]) {
        NSLog(@"yes");
        
        [self getRequestAppleProductID:@"ProductID"];
    }else {
        NSLog(@"not");
    }
}


//请求苹果后台商品
- (void) getRequestAppleProductID:(NSString *)productID {
    //ApplePayInPurchase对应着苹果后台的商品ID,他们是通过这个ID进行联系的
    NSArray *product = [[NSArray alloc] initWithObjects:ApplePayInPurchase , nil];
    NSSet *nsset = [NSSet setWithArray:product];
    
    //初始化请求
    SKProductsRequest *request = [[SKProductsRequest alloc] initWithProductIdentifiers:nsset];
    request.delegate = self;
    
    //开始请求
    [request start];
}


#pragma mark -- SKProductsRequestDelegate
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response {
    NSArray *product = response.products;
    
    //服务器没有产品
    if ([product count] == 0) {
        NSLog(@"服务器没有产品");
        return;
    }
    
    SKProduct *requestProduct = nil;
    for (SKProduct *pro in product) {
        NSLog(@"%@", [pro description]);
        NSLog(@"%@", [pro localizedTitle]);
        NSLog(@"%@", [pro localizedDescription]);
        NSLog(@"%@", [pro price]);
        NSLog(@"%@", [pro productIdentifier]);
        
        //如果后台消费条目的ID与我这里需要请求的一样（用于确保订单的正确性）
        if ([pro.productIdentifier isEqualToString:ApplePayInPurchase]) {
            requestProduct = pro;
        }
    }
    
    
    //发送购买请求
    SKPayment *payment = [SKPayment paymentWithProduct:requestProduct];
    [[SKPaymentQueue defaultQueue] addPayment:payment];
}



#pragma mark -- SKRequestDelegate
//请求失败
- (void)request:(SKRequest *)request didFailWithError:(NSError *)error {
    NSLog(@"请求失败error: %@", error);
}
//反馈请求的产品信息结束后
- (void)requestDidFinish:(SKRequest *)request{
    NSLog(@"信息反馈结束");
}



#pragma mark -- SKPaymentTransactionObserver
//监听购买结果
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions {
    for (SKPaymentTransaction *tran in transactions) {
        switch (tran.transactionState) {
            case SKPaymentTransactionStatePurchased://购买成功
                NSLog(@"交易完成");
                [self completeTransaction:tran];
                break;
                
            case SKPaymentTransactionStatePurchasing://正在处理
                NSLog(@"商品添加进列表");
                break;
                
            case SKPaymentTransactionStateRestored://恢复购买
                NSLog(@"已经购买过商品");
                [[SKPaymentQueue defaultQueue] finishTransaction:tran];
                break;
                
            case SKPaymentTransactionStateFailed:
                NSLog(@"交易失败");
                [[SKPaymentQueue defaultQueue] finishTransaction:tran];
                break;
                
            default:
                break;
        }
    }
}

//交易结束,当交易结束后还要去appstore上验证支付信息是否都正确,只有所有都正确后,我们就可以给用户方法我们的虚拟物品了
- (void) completeTransaction:(SKPaymentTransaction *)transaction {
    NSString *str = [[NSString alloc] initWithData:transaction.transactionReceipt encoding:NSUTF8StringEncoding];
    NSString *environment = [self environmentForReceipt:str];
    
    NSLog(@"----- 完成交易调用的方法completeTransaction 1--------%@",environment);
    
    //验证凭据，获取到苹果返回的交易凭据
    //验证凭据，获取到苹果返回的交易凭据
    NSURL *receiptURL = [[NSBundle mainBundle] appStoreReceiptURL];
    //从沙盒中获取到购买凭据
    NSData *receiptData = [NSData dataWithContentsOfURL:receiptURL];
    

    //验证返回的数据，防止粤语(安全性不高的可以不用看下面的代码)
    //封装的验证方法
    /**
        NSString *receipt = [receiptData base64EncodedStringWithOptions:NSDataBase64Encoding64CharacterLineLength];
        if ([receipt length] > 0 && [productIdentifier length] > 0) {
        [SVProgressHUD showInfoWithStatus:@"支付成功!"];
             //可以将receipt发给服务器进行购买验证
             [self dl_validateReceiptWiththeAppStore:receipt];
         }
    */

    //BASE64 常用的编码方案，通常用于数据传输，以及加密算法的基础算法，传输过程中能够保证数据传输的稳定性
    //BASE64是可以编码和解码的
    NSString *encodeStr = [receiptData base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithLineFeed];
    NSString *sendString = [NSString stringWithFormat:@"{\"receipt-data\" : \"%@\"}", encodeStr];
    NSLog(@"_____%@",sendString);
    NSURL *StoreURL=nil;
    if ([environment isEqualToString:@"environment=Sandbox"]) {
        StoreURL = [[NSURL alloc] initWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"];
    }else {
        StoreURL = [[NSURL alloc] initWithString:@"https://buy.itunes.apple.com/verifyReceipt"];
    }
    
    //这个二进制数据由服务器进行验证；zl
    NSData *postData = [NSData dataWithBytes:[sendString UTF8String] length:[sendString length]];
    
    NSLog(@"++++++%@",postData);
    NSMutableURLRequest *connectionRequest = [NSMutableURLRequest requestWithURL:StoreURL];
    
    [connectionRequest setHTTPMethod:@"POST"];
    [connectionRequest setTimeoutInterval:50.0];//120.0---50.0zl
    [connectionRequest setCachePolicy:NSURLRequestUseProtocolCachePolicy];
    [connectionRequest setHTTPBody:postData];
    
    //开始请求
    NSError *error=nil;
    NSData *responseData=[NSURLConnection sendSynchronousRequest:connectionRequest returningResponse:nil error:&error];
    if (error) {
        NSLog(@"验证购买过程中发生错误，错误信息：%@",error.localizedDescription);
        return;
    }
    NSDictionary *dic=[NSJSONSerialization JSONObjectWithData:responseData options:NSJSONReadingAllowFragments error:nil];
    NSLog(@"请求成功后的数据:%@",dic);
    
    //这里可以等待上面请求的数据完成后并且state = 0 验证凭据成功来判断后进入自己服务器逻辑的判断,也可以直接进行服务器逻辑的判断,验证凭据也就是一个安全的问题。楼主这里没有用state = 0 来判断。
    // [[SKPaymentQueue defaultQueue] finishTransaction: transaction];
    
    NSString *product = transaction.payment.productIdentifier;
    
    NSLog(@"transaction.payment.productIdentifier++++%@",product);
    
    if ([product length] > 0)
    {
        NSArray *tt = [product componentsSeparatedByString:@"."];
        
        NSString *bookid = [tt lastObject];
        
        if([bookid length] > 0)
        {
            
            NSLog(@"打印bookid%@",bookid);
            //这里可以做操作吧用户对应的虚拟物品通过自己服务器进行下发操作,或者在这里通过判断得到用户将会得到多少虚拟物品,在后面（[self getApplePayDataToServerRequsetWith:transaction];的地方）上传上面自己的服务器。
        }
    }
    //此方法为将这一次操作上传给我本地服务器,记得在上传成功过后一定要记得销毁本次操作。调用[[SKPaymentQueue defaultQueue] finishTransaction: transaction];
//    [self getApplePayDataToServerRequsetWith:transaction];
    NSLog(@"--->> %@", transaction);
    

}

-(NSString * )environmentForReceipt:(NSString * )str
{
    str= [str stringByReplacingOccurrencesOfString:@"\r\n" withString:@""];
    
    str = [str stringByReplacingOccurrencesOfString:@"\n" withString:@""];
    
    str = [str stringByReplacingOccurrencesOfString:@"\t" withString:@""];
    
    str=[str stringByReplacingOccurrencesOfString:@" " withString:@""];
    
    str=[str stringByReplacingOccurrencesOfString:@"\"" withString:@""];
    
    NSArray * arr=[str componentsSeparatedByString:@";"];
    
    //存储收据环境的变量
    NSString * environment=arr[2];
    return environment;
}


-(void)dl_validateReceiptWiththeAppStore:(NSString *)receipt
{
    NSError *error;
    NSDictionary *requestContents = @{@"receipt-data": receipt};
    NSData *requestData = [NSJSONSerialization dataWithJSONObject:requestContents options:0 error:&error];
    
    if (!requestData) {
    
    }else{
        
    }
    NSURL *storeURL;
#ifdef DEBUG
    storeURL = [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"];
#else
    storeURL = [NSURL URLWithString:@"https://buy.itunes.apple.com/verifyReceipt"];
#endif
    
    NSMutableURLRequest *storeRequest = [NSMutableURLRequest requestWithURL:storeURL];
    [storeRequest setHTTPMethod:@"POST"];
    [storeRequest setHTTPBody:requestData];
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    [NSURLConnection sendAsynchronousRequest:storeRequest queue:queue
                           completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
                               if (connectionError) {
                                  /* 处理error */
                               } else {
                                   NSError *error;
                                   NSDictionary *jsonResponse = [NSJSONSerialization JSONObjectWithData:data options:0 error:&error];
                                   if (!jsonResponse) {
                                       /* 处理error */
                                   }else{
                                       /* 处理验证结果 */
                                   }
                               }
                           }];

}

```


<br/>

***
<br/>

># 其他支付

[开发支付篇](https://www.cnblogs.com/TheYouth/p/6847014.html?utm_source=itdadao&utm_medium=referral)

[iOS集成ApplePay](https://www.cnblogs.com/zhouxihi/p/6528133.html)

[Apple Pay-苹果支付](https://www.cnblogs.com/feiyu-mdm/p/5802111.html)

[Apple Pay开发流程](https://www.jianshu.com/p/7ea225928726)

[Apple Pay 与 银联 的集成](https://www.jianshu.com/p/a3d03688fc5a)



<br/>

***
<br/>


># 参考资料

[Apple Pay 集成(I)](https://www.jianshu.com/p/dd81d7ade732)

[App Store无法成功添加沙箱技术测试员账号](http://ju.outofmemory.cn/entry/365183)

[苹果IAP(内购)中沙盒账号使用注意事项](https://www.jianshu.com/p/1ef61a785508)

[申请流程](http://baijiahao.baidu.com/s?id=1586198840331975300&wfr=spider&for=pc)

[IAP应用内购详细步骤和问题总结指南](https://www.jianshu.com/p/307a3b7cfd6c)

