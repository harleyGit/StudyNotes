># NSString和String
#`共同点`
String保留了大部分NSString的API比如:
`hasPrefix`
`lowercaseString`
`componentsSeparatedByString`
`substringWithRange` 等等。
所以很多常规操作在开发中使用两者之一都是可以的。

<br/>
#`不同点`
NSString是引用类型。Swift String是值类型。
```
//NSString和String都可以使用自己的类名来直接进行初始化
//字符串赋值也是初始化，虽然写法相同，但是NSString的意思是初始化了一个指针指向了这个字符串，但Swift String的意思则是把字符串字面量赋值给变量
var nsString: NSString = NSString()
var swiftString:String = String()        
var nsString: NSString = "dsx"
var swiftString:String = "dsx"
```

<br/>
#`字符串拼接`
NSString需要用append或者stringWithFormat将两个字符串拼接；
Swift String只需要用 + 即可；


<br/>
#`Swift string 实现字符串遍历`
```
for character in "My name is dsx".characters {
  print(character)
}
```
计算字符串的长度，NSString使用`length`属性，Swift使用`characters`属性获得字符数组，然后使用字符数组属性`count`获得长度。


<br/>
`比较字符串相等`
```

et strA: NSString = ""
let strB: NSString = ""
let strC: NSString = "dsx"
let strD: NSString = "dsx"

// NSString 字符串相等
if(strA.isEqualToString(strB as String)){
  print("yes");
}

// String的相等   
if (strC == strD){
  print("yes");
}

```



<br/>
***
<br/>
># 字符串截取
```
var nameStr = "Harley"
print("origin name:\(nameStr)")
let nameStart = nameStr.startIndex
let nameEnd = nameStr.index(nameStr.endIndex, offsetBy: -2)
nameStr.replaceSubrange(nameStart...nameEnd, with: "***")   //字符替换
print("now name: \(nameStr)")
```
打印：
`origin name:Harley`
`now name: ***y`


<br/>
**`字符串索引 String.Index`**

**字符串截取**
```
// 截取前5位
let strPrefix = "一二三四五六七八九十1234567890".prefix(5)
print("--->> 前缀截取到第5位：\(strPrefix)")
```
`--->> 前缀截取到第5位：一二三四五`

**截取到指定位置**
```
// 截取到除了 6 的前几位
if let range = "一二三四五六七八九十1234567890".firstIndex(of: "6") {
    let strUpToPrefix = "一二三四五六七八九十1234567890".prefix(upTo: range)
    print("--->> upTo截取到指定字符串：\(strUpToPrefix)")
}
```
`--->> upTo截取到指定字符串：一二三四五六七八九十12345`

```
// 截取到除了 6 的前几位
if let through_range = "一二三四五六七八九十1234567890".firstIndex(of: "2") {
    let strThroughPrefix = "一二三四五六七八九十1234567890".prefix(through: through_range)
    print("--->> through截取到指定字符串：\(strThroughPrefix)")
}
```
`--->> through截取到指定字符串：一二三四五六七八九十12`


<br/>
**`upTo的 lowerBound 和 upperBound`**
```
if let range_1 = "一二三四五六七八九十1234567890".range(of: "五") {
    let str_upto_lower = "一二三四五六七八九十1234567890".prefix(upTo: range_1.lowerBound)
    print("-->> str_upto: \(str_upto_lower)")  
}
```
`-->> str_upto: 一二三四`


```
if let range_2 = "一二三四五六七八九十1234567890".range(of: 五) {
     let str_upto_upper = "一二三四五六七八九十1234567890".prefix(upTo: range_2.upperBound)
     print("-->> str_upto: \(str_upto_upper)")  
}
```
`-->> str_upto: 一二三四五`


<br/>
**`through的 lowerBound 和 upperBound`**

```
if let range_1 = "一二三四五六七八九十1234567890".range(of: "五") {
    let str_through_lower = "一二三四五六七八九十1234567890".prefix(through: range_1.lowerBound)
    print("-->> str_through: \(str_through_lower)")
}
```
`-->> str_through: 一二三四五`

```
if let range_2 = "一二三四五六七八九十1234567890".range(of:  "五") {
     let str_through_upper = "一二三四五六七八九十1234567890".prefix(through: range_2.upperBound)
     print("-->> str_through: \(str_through_upper)")
}
```
`-->> str_through: 一二三四五六`


<br/>
**` suffix的 lowerBound 和 upperBound`**
```
let str_suffix = "一二三四五六七八九十1234567890".suffix(6)
print("--->> str_suffix: \(str_suffix)")
```
`--->> str_suffix: 567890`


```
let str_suffixFrom = "一二三四五六七八九十1234567890".suffix(from: str_2.firstIndex(of: "5")!)
print("---->> str_suffixFrom: \(str_suffixFrom)")
```
`--->> str_suffixFrom: 567890`


<br/>
```
let str: String = "我最爱北京天安门！"
let range: Range = str.range(of: "北京")!
let location: Int = str.distance(from: str.startIndex, to: range.lowerBound)
/* location = 3 */

let keyLength: Int = str.distance(from: range.lowerBound, to: range.upperBound)
// let key = "北京"; let keyLength = key.count;  //count = 2
/* keyLength = 2 */
print("location = \(location), length = \(keyLength)")
/* location = 3, length = 2 */
```

```
// SubString
let frontStr: Substring = str[str.startIndex ..< range.lowerBound]
print("frontSubStr = \(frontStr)")
/* frontSubStr = 我最爱 */

let frontStr2: Substring = str[str.startIndex ... range.lowerBound]
print("frontSubStr2 = \(frontStr2)")
/* frontSubStr2 = 我最爱北 */
```


```
let frontTest_before: Substring = str[str.startIndex ..< str.index(before: range.lowerBound)]
let frontTest_after: Substring = str[str.startIndex ..< str.index(after: range.lowerBound)]
print("before = \(frontTest_before), after = \(frontTest_after)")
/* before = 我最, after = 我最爱北 */
```




<br/>
***
<br/>
参考资料：
[String.Index如何在Swift中工作 US](http://www.imooc.com/wenda/detail/594277)
[字符串截取、替换、插入 US](https://blog.csdn.net/SSY_1992/article/details/86542445)
[字符串操作（替换、过滤、去掉空格、分割、拼接、字符串截取） US](https://blog.csdn.net/u014599371/article/details/80093022)







