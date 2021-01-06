># Foundation 对象转化为 JSON对象

```
    NSDictionary *muDic = @{@"token": @"123456789", @"name": @"harely"};

NSData *data = [NSJSONSerialization dataWithJSONObject:[muDic copy] options:kNilOptions error:nil];
    NSString *jsonS = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    
    NSDictionary * aa = [NSDictionary dictionaryWithObject:[ViewController encrypt:jsonS] forKey:@"p"] ;
    
    NSLog(@"------>> aa: %@", aa);
```


##代码解析
NSJSONSerialization提供了将JSON数据转换为Foundation对象（一般都是NSDictionary和NSArray）和Foundation对象转换为JSON数据（可以通过调用isValidJSONObject来判断Foundation对象是否可以转换为JSON数据）

>NSJSONWritingOptions 包含2种参数：
<br/>NSJSONWritingPrettyPrinted    将生成的json数据格式化输出，这样可读性高，不设置则输出的json字符串就是一
<br/>NSJSONWritingSortedKeys     输出的json字符串就是一整行


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
***


#JSON数据(NSData)转化为Foundation对象(Object)
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
***
