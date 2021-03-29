- **[OC基础](#oc基础)** [](https://my.oschina.net/antsky/blog/1475173)
	- [UILabel多适应](#uilabel多适应)
	- [不同账号内购后的崩溃处理异常](#不同账号内购后的崩溃处理异常)


<br/>

***
<br/>

># OC基础

<br/>

> UILabel多适应?

Label自适应宽度

```
+ (float)widthAuto:(NSString *)content fontSize:(CGFloat)fontSize contentheight:(CGFloat)height{

		// 列寬
		//    CGFloat contentHeight = height;
		
		// 用何種字體進行顯示
		UIFont *font = [UIFont systemFontOfSize:fontSize];
		
		// 計算出顯示完內容需要的最小尺寸
		CGSize size = CGSizeMake(100, 100);
		
		// 計算出顯示完內容需要的最小尺寸
		NSDictionary * tdic = [NSDictionary dictionaryWithObjectsAndKeys:font,NSFontAttributeName,nil];
		
		//ios7方法，获取文本需要的size，限制宽度
		CGSize  actualsize =[content boundingRectWithSize:size options:NSStringDrawingUsesLineFragmentOrigin |NSStringDrawingUsesFontLeading attributes:tdic context:nil].size;
		
		return actualsize.width;

}

```


<br/>
<br/>

> 不同账号内购后的崩溃处理异常


IAP支付的过程：

- 调用IAP接口发起支付
- 支付成功，获取App Receipt票据，调用充值接口验证
- 验证通过，给用户充值虚拟货币并回调给 App

&emsp; 这里需要了解一下IAP支付的机制，每次支付行为或每笔交易被认为是一个SKPaymentTransation，只有当SKPaymentTransation被finishTransaction:，这次支付（交易）行为才算是正常结束了。即使这次支付途中被中断，其实也并没有丢失。假设支付没有完成 App 就退出了（比如崩溃），那么当下次 App 重启之后，只要设置了监听addTransactionObserver:，之前被中断的支付就会接着进行。

**掉单**可能会出现在哪些环节：
- 第1步，这个过程中 App 进程因为某种原因被 kill 了，其实支付行为还在系统后台进行着，苹果自己做的，很有可能扣款成功。但是这时候没法为用户充值虚拟货币。
- 第2步，App 端与自己服务器端通信失败；自己服务器端与 AppStore 服务器之间的通信失败。

&emsp; 针对第一种情况，可以在 App 一启动就设置监听，如果有未完成的支付，则会回调`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions;`这个方法，在这个方法里调用接口充值。
至于第二种情况，App 端需要做接口重试，设置一个重试的逻辑。

&emsp; 下面来描述一下我的具体做法。

&emsp; 我使用了RMStore，稍作了一些修改。新建一个数据表（字段有：`transactionIdentifier、productIdentifier、uid、transactionDate(支付时间)、rechargeDate(充值到账时间)、state`(「已支付」和「已充值」)），存每一笔交易。

&emsp; 在`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions;`这个方法里，首先根据`SKPaymentTransaction的transactionIdentifier`去本地数据表中判断这条交易：

- 如果没有，则在数据表中创建一条交易，状态为「已支付」，接着去调用充值接口；
- 如果有这条交易，并且状态为「已支付」，则不需要创建该条交易，只需要去调用充值接口；
- 如果有这条交易，并且状态为「已充值」（虚拟货币到了用户账上），则不需要创建，也不需要去充值，直接结束交易（finishTransaction:）。

&emsp; 充值接口的设置，App 端需要传这些参数：交易id、商品id、用户uid、票据，该接口在服务器端需要做一些操作：transactionIdentifier需要入库，和App Receipt对应起来，每次访问接口都要先做去重判断，如果是已经验证通过的transactionIdentifier，也返回成功。只要是访问成功，该接口都需要带上transactionIdentifier返回给 App 端，可能不止一个，因为一个App Receipt票据验证成功后返回的交易列表可能有多个

<br/>

**漏单：**

&emsp; 以上流程中可能会出现漏单的情况：当客户端向苹果支付成功后，准备向服务端发起验证，此时若网络发生抖动或者人为退出app等情况，就会出现用户充值成功，但是没加金币的情况。
解决方案：在paymentQueue:updatedTransactions:代理方法中获取到支付成功的回调，此时马上缓存订单信息（包括支付凭证、订单号、用户id等；担心用户卸载app的，可以直接缓存到钥匙串），然后再向服务端验证订单。若订单验证成功，则删除该项订单缓存；否则不做处理。另外，每次app启动的时候，可以延时几秒钟，去检测本地是否有未验证成功的苹果订单；如果有，就一个个重新验证。

<br/>

**刷单：**

&emsp; 我们公司的app最早的时候，服务端还是用的Http请求，也没有考虑到刷单这一块，然后被某些用户钻了空子；他们先发起苹果支付，获取一个订单号，然后拿这个订单号和之前支付成功的凭证，向服务端发起验证；然后服务端发现订单号也对得上，凭证也验证有效，就给用户加金币了；用户用同一个凭证不断的换订单号。。。。。
解决方案：首先服务端每次生成的订单号都缓存起来，客户端在请求完订单号后，将该订单号记录在内购订单对象里（赋值给SKPayment的applicationUsername属性）；然后当客户端支付成功后，把订单id和支付凭证传给服务端，服务端先拿凭证去苹果校验，苹果返回的结果中有一个transactionId（跟前面记录的applicationUsername是一致的） ，这时候服务端判断该transactionId和订单id是否一致，若一致，且该订单id是一个新id，之前没有缓存过；才算是完全的校验成功。

**订单重复：**

&emsp; 订单重复的情况就是支付成功后，从本地拿到了错误的支付凭证（之前的凭证），去校验的时候，会发现该订单已经完成了。
我这边发现这个问题的情况是这样的：客户端发起苹果支付的时候调用addPayment方法，在paymentQueue:updatedTransactions:回调中获取支付的状态，支付成功、支付失败和重复购买的状态下要结束订单finishTransaction。我当时是支付失败的情况，没有结束订单，然后下次支付成功的时候，获取的支付凭证是上次失败的凭证，然后不仅校验会失败，订单也是重复的。
