>#  闭包变量
```
var mySecondClosure:(_ a: Int, _ b: Int) -> Int = {
        (a: Int, b: Int) -> Int in
        return a * b
    }


let c = mySecondClosure(3, 5)
print("闭包变量 mySecondClosure 的闭包值：\(c)")

```
打印：
`闭包变量 mySecondClosure 的闭包值：15`


<br/>
***
<br/>
>#  闭包参数
```
func myOperation(_ a: Int, _ b: Int, operation: (_ oa: Int, _ ob: Int) -> Int) -> Int {
        let res = operation(a, b)
        return res
    }


//使用
/*

//被捕获的参数列表中，含有a、b，下标从0开始，可通过"$"获取。
//编译器亦可通过，捕获列表自行推断出参数。
//故可省略参数列表 （a, b）和 关键字 in 
let multipyClosure:(Int, Int) -> Int = {
      $0 * $1
}
  
//若函数体只包含一句 return 代码，可省略 return      
let multipyClosure = {
      (a: Int, b: Int) in
       a * b
 }
 */

//上述两个闭包的写法与下同样        
let multipyClosure = {
      (a: Int, b: Int) in
       return a * b
}
        
let d = myOperation(9, 10, operation: multipyClosure)
print("参数闭包 operation 返回值：\(d)")


//闭包的展开-----------------------------------------------

func myOperation(_ a: Int, _ b: Int, operation: (_ oa: Int, _ ob: Int) -> Int) -> Int {
        let res = operation(a, b)
        return res
}

let d = self.myOperation(9, 10) { (a: Int, b: Int) -> Int in
      return a * b
 }
print("参数闭包 operation 返回值:  \(d)")        

```
打印：
`参数闭包 operation 返回值：90`

<br/>
***
<br/>
># 闭包捕获
```
        var count = 2
        let incrementCount = {
            count += 8
        }

        incrementCount()
        print("第 1 次计算: \(count)")

        incrementCount()
        print("第 2 次计算: \(count)")
```
打印：
`第 1 次计算: 10`
`第 2 次计算: 18`

&emsp;  由于闭包定义和变量count在同一作用域中，故闭包可以捕获并访问变量count。对变量counter做的任何改变，对闭包来说都是透明可见的。

<br/>
#`函数捕获值`
```
      func countingClosure() -> () -> Int {
        var counter = 0
        let incrementCounter: () -> Int = {
            counter += 1
            return counter
        }
        return incrementCounter
       }
    

        //该例子中，闭包捕获了封闭空间（函数实体内）的内部变量counter。
        let counter1 = countingClosure()  //返回的是一个 () -> Int 闭包 
        let counter2 = countingClosure()
        
        let count_1 = counter1() // 1, 执行 () -> Int 闭包中的函数定义
        print("\(String(describing: count_1))")
        
        let count_2 = counter2() // 1
        print("\(String(describing: count_2))")
        
        let count_3 = counter1() // 2
        print("\(String(describing: count_3))")
        
        let count_4 = counter1() // 3
        print("\(String(describing: count_4))")
        
        let count_5 = counter2() // 2
        print("\(String(describing: count_5))")
```
打印：
`1`
`1`
`2`
`3`
`2`



<br/>
***
<br/>
># 闭包的柯西特性
```
func add(_ num: Int) -> (_ second: Int) -> Int {
        return { val in
            return num + val
        }
  }


let addTwo = self.add(2)(3)

print("\(String(describing: addTwo))")
```
打印：
`5`





<br/>
***
<br/>
[属性闭包](https://blog.csdn.net/weixin_34001430/article/details/87976286)
[闭包表达式、尾随、闭包、闭包](https://blog.csdn.net/weixin_33883178/article/details/86784698)
