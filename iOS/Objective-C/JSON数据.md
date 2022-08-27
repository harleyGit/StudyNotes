> <h2 id=""></h2>
![](./../../Pictures/)
- [**JSON来源**](#JSON来源)
	- [如何解析JSON](#如何解析JSON)
- [字典转Json](#字典转Json)
- [NSData转字典](#NSData转字典)


<br/>
<br/>

***
<br/>


> <h1 id="JSON来源">JSON来源</h1>

&emsp; JSON是 JavaScript Object Notation 的缩写。其实，JSON 最初是被设计为 JavaScript 语言的一个子集，但最终因为和编程语言无关，所以成为了一种开放标准的常见数据格式。


&emsp; 虽然 JSON 源于 JavaScript，但到目前很多编程语言都有了 JSON 解析的库，包括 C、C++、Java、Perl、Python 等等。除此之外，还有很多编程语言内置了 JSON 生成和解析的方法，比如 PHP 在 5.2 版本开始内置了 json_encode() 方法，可以将 PHP 里的 Array 直接转化成 JSON。

<br/>


**JSON 基于两种结构：**
- 名字 / 值对集合：这种结构在其他编程语言里被实现为对象、字典、Hash 表、结构体或者关联数组。
- 有序值列表：这种结构在其他编程语言里被实现为数组、向量、列表或序列。


<br/>
<br/>

> <h2 id="如何解析JSON">如何解析 JSON</h2>



<br/>
<br/>

***
<br/>


> <h1 id=""></h1>


<br/>

> <h2 id=""></h2>



<br/>
<br/>

***
<br/>


> <h1 id=""></h1>


<br/>

> <h2 id="字典转Json">字典转Json</h2>

```
NSDictionary *muDic = @{@"token": @"123456789", @"name": @"harely"};

NSData *data = [NSJSONSerialization dataWithJSONObject:[muDic copy] options:kNilOptions error:nil];
NSString *jsonS = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

NSDictionary * aa = [NSDictionary dictionaryWithObject:[ViewController encrypt:jsonS] forKey:@"p"] ;

NSLog(@"------>> aa: %@", aa);
```



<br/>

- **代码解析**

&emsp; NSJSONSerialization提供了将JSON数据转换为Foundation对象（一般都是NSDictionary和NSArray）和Foundation对象转换为JSON数据（可以通过调用isValidJSONObject来判断Foundation对象是否可以转换为JSON数据）

NSJSONWritingOptions 包含2种参数：
- NSJSONWritingPrettyPrinted:将生成的json数据格式化输出，这样可读性高，不设置则输出的json字符串
- NSJSONWritingSortedKeys :输出的json字符串就是一整行


打印结果为：

```
po muDic
{
    name = harely;
    token = 123456789;
}

 po data
<7b22746f 6b656e22 3a223132 33343536 37383922 2c226e61 6d65223a 22686172 656c7922 7d>

(lldb) po jsonS
{"token":"123456789","name":"harely"}

2018-08-07 14:15:01.420066+0800 Test[4446:381421] ------>> aa: {
    p = "bGtuenV4Y3dlRw56eymV@@ApaMqJEWD$$fQMMHR2KNZL0od7CADmKNK6h4hGg9OhzI";
}
```


<br/>
<br/>



> <h2 id="NSData转字典">NSData转字典</h2>

**JSON数据(NSData)转化为Foundation对象(Object):**

```
+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;
```

<br/>

```
/*
  NSJSONReadingMutableContainers
  NSJSONReadingMutableLeaves
  NSJSONReadingAllowFragments
 不在乎返回的是什么东西，就用kNilOptions，效率最好
 */

NSData *data = [NSData new];
//解析返回的JSON
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil];
        
NSLog(@"%@", dict[@"error"]);
```

<br/>

