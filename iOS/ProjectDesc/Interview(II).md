
- [**安全**](#安全)
	- [MD5](#MD5)


<br/>

***
<br/>

> <h1 id="安全">安全</h1>

<br/>

> <h2 id="MD5">MD5</h2>

<br/>

- 使用场景：

&emsp; MD5常用在密码加密中，一般为了保证用户密码的安全，在数据库中存储的都是用户的密码经过MD5加密后的值，在客户端用户输入密码后，也会使用MD5进行加密，这样即使用户的网络被窃听，窃听者依然无法拿到用户的原始密码，并且即使用户数据库被盗，没有存储明文的密码对用户来说也多了一层安全保障。

&emsp;MD5签名技术还常用于防止信息的篡改。使用MD5可以对进行进行签名，接收者拿到信息后只要重新计算签名和原始签名进行对比，即可知道数据信息是否中途被篡改了。

<br/>

- MD5 加密参数可能会发生丢失？解释下原理？

[MD5原理](https://cloud.tencent.com/developer/article/1402024)


OC实现MD5加密：

获得MD5加密

```
- (NSString *)md5:(NSString *)str {

	NSString *resultStr = nil;
	
	const char *cStr = [str UTF8String];//指针不能变，cStr指针变量本身可以变化
	
	unsigned char result[16];//这里可以CC_MD5_DIGEST_LENGTH宏代替16，这里不明白为什么写16后面我有解释的！
	
	CC_MD5(cStr, (unsigned int)strlen(cStr), result);
	
	resultStr = [NSString stringWithFormat:
	
	@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
	
	result[0], result[1], result[2], result[3],
	
	result[4], result[5], result[6], result[7],
	
	result[8], result[9], result[10], result[11],
	
	result[12], result[13], result[14], result[15]
	
	];
	
	return [resultStr uppercaseString];
	
}
```
值得注意的是：

&emsp; 其中%02x是格式控制符：‘x’表示以16进制输出，‘02’表示不足两位，前面补0；如‘f’输出为0f，‘1f3’则输出1f3;还有就是为什么是result[16]呢，这是因为MD5算法最后生成的是128位，而在计算机的最小存储单位为字节，1个字节是8位，对应一个char类型，计算可得需要16个char。所以result是[16]。那么为什么输出的格式一定是%02x呢，而不是其它呢。这也是有原因的：因为约定MD5一般是以16进制的格式输出，那么其实这个问题就转换为把128个0和1以16进制来表示，每4位二进制对应一个16进制的元素，则需要32个16进制的元素，如果元素全部为0，放到char的数组中，正常是不会输出，如00001111，以%x输出，则是f,那么就会丢失0；但如果以%02x表示则输出结果是0f，正好是转换的正确结果。

所以以上就是char[16]和%02x的来历。

