

- NSString和String
	- 共同点String保留了大部分NSString的API比如
	- 不同点 NSString是引用类型。Swift String是值类型
	- 字符串拼接
	- Swift string 实现字符串遍历
	- 比较字符串相等
- 字符串截取
	- 截取到指定位置
	- upTo的 lowerBound 和 upperBound
	- through的 lowerBound 和 upperBound

- 金额显示


<br/>

***
<br/>

># NSString和String

- **`共同点String保留了大部分NSString的API比如:`**

`hasPrefix`

`lowercaseString`

`componentsSeparatedByString`

`substringWithRange` 等等。
所以很多常规操作在开发中使用两者之一都是可以的。

<br/>

- **`不同点`**
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

- **`字符串拼接`**

NSString需要用append或者stringWithFormat将两个字符串拼接；
Swift String只需要用 + 即可；


<br/>

- **`Swift string 实现字符串遍历`**

```
for character in "My name is dsx".characters {
  print(character)
}
```
计算字符串的长度，NSString使用`length`属性，Swift使用`characters`属性获得字符数组，然后使用字符数组属性`count`获得长度。


<br/>

- **`比较字符串相等`**

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

- **`字符串索引 String.Index`**

**字符串截取**

```
// 截取前5位
let strPrefix = "一二三四五六七八九十1234567890".prefix(5)
print("--->> 前缀截取到第5位：\(strPrefix)")
```
打印：

`--->> 前缀截取到第5位：一二三四五`

<br/>

- **截取到指定位置**

```
// 截取到除了 6 的前几位
if let range = "一二三四五六七八九十1234567890".firstIndex(of: "6") {
    let strUpToPrefix = "一二三四五六七八九十1234567890".prefix(upTo: range)
    print("--->> upTo截取到指定字符串：\(strUpToPrefix)")
}
```
打印：

`--->> upTo截取到指定字符串：一二三四五六七八九十12345`

<br/>


```
// 截取到除了 6 的前几位
if let through_range = "一二三四五六七八九十1234567890".firstIndex(of: "2") {
    let strThroughPrefix = "一二三四五六七八九十1234567890".prefix(through: through_range)
    print("--->> through截取到指定字符串：\(strThroughPrefix)")
}
```
打印：

`--->> through截取到指定字符串：一二三四五六七八九十12`


<br/>

- **`upTo的 lowerBound 和 upperBound`**

```
if let range_1 = "一二三四五六七八九十1234567890".range(of: "五") {
    let str_upto_lower = "一二三四五六七八九十1234567890".prefix(upTo: range_1.lowerBound)
    print("-->> str_upto: \(str_upto_lower)")  
}
```
打印：

`-->> str_upto: 一二三四`


```
if let range_2 = "一二三四五六七八九十1234567890".range(of: 五) {
     let str_upto_upper = "一二三四五六七八九十1234567890".prefix(upTo: range_2.upperBound)
     print("-->> str_upto: \(str_upto_upper)")  
}
```
`-->> str_upto: 一二三四五`


<br/>

- **`through的 lowerBound 和 upperBound`**

```
if let range_1 = "一二三四五六七八九十1234567890".range(of: "五") {
    let str_through_lower = "一二三四五六七八九十1234567890".prefix(through: range_1.lowerBound)
    print("-->> str_through: \(str_through_lower)")
}
```
打印：

`-->> str_through: 一二三四五`

```
if let range_2 = "一二三四五六七八九十1234567890".range(of:  "五") {
     let str_through_upper = "一二三四五六七八九十1234567890".prefix(through: range_2.upperBound)
     print("-->> str_through: \(str_through_upper)")
}
```
打印：

`-->> str_through: 一二三四五六`


<br/>

- **` suffix的 lowerBound 和 upperBound`**

```
let str_suffix = "一二三四五六七八九十1234567890".suffix(6)
print("--->> str_suffix: \(str_suffix)")
```
打印：

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


># 金额显示

金额显示要求：

![金额显示要求](https://upload-images.jianshu.io/upload_images/2959789-9294a9235f456505.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
///产品原始测试数据
let test1 = "6774215635.055265745674"   //9267亿多 整数位12位 小数位12位
let test2 = "774215635.055265745674"
let test3 = "215635.055265745674"
let test4 = "234.76847345264"
let test5 = "0.777889998762"              //小数位12位

/// 内部测试数据
let ctest1 = "12345678901234567000.123456789012"//12345678901234567000.00"
let ctest2 = "-0.0"
let ctest3 = "1.0"
let ctest4 = "0.1234"
let ctest5 = "1.1234"
let ctest6 = "12.1234"
let ctest7 = "123.1234"
let ctest8 = "1234.1234"
let ctest9 = "12345.1234"
let ctest10 = "123456.1234"
let ctest11 = "1234567.1234"
let ctest12 = "12345678.1234"
let ctest13 = "123456789.1234"
let ctest14 = "1234567890.1234"
let ctest15 = "11047130.279"



@objc func showPrit() {
        
	let t1_0 = amountShow(amount: test1, type: 0)
	let t1_1 = amountShow(amount: test1, type: 1)
	let t1_2 = amountShow(amount: test1, type: 2)
	print("原数据：\(t1_0)->    1级显示\(t1_1), 2级显示：\(t1_2)")
	
	let t2_0 = amountShow(amount: test2, type: 0)
	let t2_1 = amountShow(amount: test2, type: 1)
	let t2_2 = amountShow(amount: test2, type: 2)
	print("原数据：\(t2_0)->    1级显示\(t2_1), 2级显示：\(t2_2)")
	
	let t3_0 = amountShow(amount: test3, type: 0)
	let t3_1 = amountShow(amount: test3, type: 1)
	let t3_2 = amountShow(amount: test3, type: 2)
	print("原数据：\(t3_0)->    1级显示\(t3_1), 2级显示：\(t3_2)")
	
	let t4_0 = amountShow(amount: test4, type: 0)
	let t4_1 = amountShow(amount: test4, type: 1)
	let t4_2 = amountShow(amount: test4, type: 2)
	print("原数据：\(t4_0)->    1级显示\(t4_1), 2级显示：\(t4_2)")
	
	let t5_0 = amountShow(amount: test5, type: 0)
	let t5_1 = amountShow(amount: test5, type: 1)
	let t5_2 = amountShow(amount: test5, type: 2)
	let t5_3 = amountShow(amount: test5, type: 3)
	print("test原数据：\(t5_0)->    1级显示\(t5_1), 2级显示：\(t5_2), 贸易显示：\(t5_3)")
	
	let ctArray:[String] = [ctest1, ctest2, ctest3, ctest4, ctest5, ctest6, ctest7, ctest8, ctest9, ctest10, ctest11, ctest12, ctest13, ctest14, ctest15]
	
	print("\n\n\n")
	for (index, ctest) in ctArray.enumerated() {
	    let ct1_0 = amountShow(amount: ctest, type: 0)
	    let ct1_1 = amountShow(amount: ctest, type: 1)
	    let ct1_2 = amountShow(amount: ctest, type: 2)
	    print("ctest\(index + 1)原数据：\(ct1_0)->    1级显示\(ct1_1), 2级显示：\(ct1_2)")
	}
    
    /*
    var string = "abcdefgh"
    let str1 = string[string.index(before: string.endIndex)]
    let str2 = string[string.index(after: string.startIndex)]

    print(str1)
    print(str2)


    let startIndext:String.Index = string.startIndex
    let endIndext:String.Index = string.endIndex
    let firstString = string[startIndext]
    print(firstString)
    //输出字符串是a

    let endString = string[endIndext -1]
    //编译器报越界错误，endIndext并不是字符串下标脚本的合法实际参数。
    */
}
    
//MARK: -- 金额显示
//0: 原数据显示， 1： 单位显示，    2: 9位显示
func amountShow(amount: String = "", type: UInt = 0) -> String {
    
    switch type {
    case 0:
        return amount
    case 1:
        return oneLevelAmountShow(amount: amount)
    case 2:
        return twoLevelDataShow(amount: amount)
    case 3:
        return businessAmountShow(amount: amount)
    default:
        break
    }
    
    return "-.-"
}
    
///一级金额显示
public func oneLevelAmountShow(amount: String = "") -> String {
    //保留2位小数
    let decimalNumberH_2 = NSDecimalNumberHandler(roundingMode: NSDecimalNumber.RoundingMode.down, scale: 2, raiseOnExactness: false, raiseOnOverflow: false, raiseOnUnderflow: false, raiseOnDivideByZero: true)
    
    guard Double(amount) != nil else {
        return "-.-"
    }
    
    let billionDN = NSDecimalNumber(string: "1000000000.00")    //十亿
    let millionDN = NSDecimalNumber(string: "1000000.00")   //百万
    let thousandDN = NSDecimalNumber(string: "1000.00")   //千
    let oneDN = NSDecimalNumber(string: "1.00")   //1
    let amountDN = NSDecimalNumber(string: amount)
    
    ///B:十亿显示
    if amountDN.compare(billionDN) == .orderedDescending || amountDN.compare(billionDN) == .orderedSame {
        let multiplyB = amountDN.dividing(by: billionDN, withBehavior: decimalNumberH_2)
        return "\(multiplyB)B"
    }
    
    ///M：百万显示
    if amountDN.compare(billionDN) == .orderedAscending && amountDN.compare(millionDN) == .orderedDescending  || amountDN.compare(billionDN) == .orderedSame {
        let multiplyB = amountDN.dividing(by: millionDN, withBehavior: decimalNumberH_2)
        return "\(multiplyB)M"
    }
    
    ///K： 千显示
    if amountDN.compare(billionDN) == .orderedAscending && amountDN.compare(thousandDN) == .orderedDescending || amountDN.compare(thousandDN) == .orderedSame {
        let multiplyB = amountDN.dividing(by: thousandDN, withBehavior: decimalNumberH_2)
        return "\(multiplyB)K"
    }
    
    ///个： 显示
    if amountDN.compare(thousandDN) == .orderedAscending &&  amountDN.compare(oneDN) == .orderedDescending || amountDN.compare(oneDN) == .orderedSame {
        let decimalNumberH_0 = NSDecimalNumberHandler(roundingMode: NSDecimalNumber.RoundingMode.down, scale: 0, raiseOnExactness: false, raiseOnOverflow: false, raiseOnUnderflow: false, raiseOnDivideByZero: true)
        let multiplyB = amountDN.dividing(by: oneDN, withBehavior: decimalNumberH_0)
        
        return "\(multiplyB)"
    }
    
    ///小数显示
    if amountDN.compare(oneDN) == .orderedAscending {
        let multiplyB = amountDN.dividing(by: oneDN, withBehavior: decimalNumberH_2)
        
        return "\(multiplyB)"
    }
    
    return "-.-"
}

///二级金额显示
public func twoLevelDataShow(amount: String, digit: Int = 9) -> String {
    //判断是否为0
    if Double(amount) == 0.00 {
        return "0"
    }
    //不含小数点
    if !amount.contains(".") {
        return amount
    }//ctest1
    
    //含小数点
    let amountArray = amount.components(separatedBy: ".")
    //整数部分
    let integerPart = amountArray.first ?? "0"
    //小数部分
    let decimalPart = amountArray.last ?? "0"
    //整数位数
    let integerDigit = integerPart.count
    //小数位数
    let decimalDigit = decimalPart.count
    
    if integerDigit >= digit {
        return "\(integerPart)"
    }else {
        //要取的小数位数
        let realDecimalDigit = digit - (integerDigit + 1)
        if Int(decimalDigit) <= 0 || realDecimalDigit == 0 {//只有整数位或者整数位为8位
            return "\(integerPart)"
        }
        //截取前 realDecimalDigit 字符串
        let realDecimal = decimalPart.prefix(realDecimalDigit)
        let lastAmount = zeroManagerOfdecimalSuffix(sourceNum: "\(integerPart).\(realDecimal)")

        return lastAmount
    }
}

///小数点后去0处理
public func zeroManagerOfdecimalSuffix(sourceNum: String) -> String {
    var outNum = sourceNum
    var i = 1
    
    if sourceNum.contains(".") {
        while i < sourceNum.count {
            if outNum.hasSuffix("0") {
                outNum.remove(at: outNum.index(before: outNum.endIndex))
                i += 1
            }else {
                break
            }
        }
        
        if outNum.hasSuffix(".") {
            outNum.remove(at: outNum.index(before: outNum.endIndex))
        }
        return outNum
    }else {
        return sourceNum
    }
}


///交易金额显示
func businessAmountShow(amount: String, decimalPoinDigit: UInt = 6) -> String {
    //不含小数点
    if !amount.contains(".") {
        return amount
    }//ctest1
    
    //含小数点
    let amountArray = amount.components(separatedBy: ".")
    //整数部分
    let integerPart = amountArray.first ?? "0"
    //小数部分
    let decimalPart = amountArray.last ?? "0"
    //小数位数
    let decimalDigit = decimalPart.count
    
    if Int(decimalDigit) > decimalPoinDigit {//大于小数点保留位数
        
        //截取前 realDecimalDigit 字符串
        let realDecimal = decimalPart.prefix(Int(decimalPoinDigit))
        let lastAmount = zeroManagerOfdecimalSuffix(sourceNum: "\(integerPart).\(realDecimal)")

        return lastAmount
    }else {
        let lastAmount = zeroManagerOfdecimalSuffix(sourceNum: "\(integerPart).\(decimalPart)")

        return lastAmount
    }
}

```

后台打印效果：

![测试数据展示](https://upload-images.jianshu.io/upload_images/2959789-3e3ab36f0e1744d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[](https://www.teamleader.cn/ios-shu-ju-jing-du-ji-da-shu-de-chu-li/)

[数字格式化显示](https://www.hangge.com/blog/cache/detail_2086.html)

[NSDecimalNumber的基本使用](https://blog.csdn.net/ziyuzhiye/article/details/88844105)

[Git Demo](https://github.com/yafoolaw/NSNumberFormatterDemo)



<br/>

***
<br/>




参考资料：

[String.Index如何在Swift中工作 US](http://www.imooc.com/wenda/detail/594277)

[字符串截取、替换、插入 US](https://blog.csdn.net/SSY_1992/article/details/86542445)

[字符串操作（替换、过滤、去掉空格、分割、拼接、字符串截取） US](https://blog.csdn.net/u014599371/article/details/80093022)







