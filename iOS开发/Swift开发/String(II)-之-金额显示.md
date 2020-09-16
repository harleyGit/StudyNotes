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


<br/>
参考资料：
[](https://www.teamleader.cn/ios-shu-ju-jing-du-ji-da-shu-de-chu-li/)
[数字格式化显示](https://www.hangge.com/blog/cache/detail_2086.html)
[NSDecimalNumber的基本使用](https://blog.csdn.net/ziyuzhiye/article/details/88844105)
[Git Demo](https://github.com/yafoolaw/NSNumberFormatterDemo)
