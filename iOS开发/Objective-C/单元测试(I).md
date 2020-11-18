- **[单元测试和 UI 测试快速入门](https://juejin.im/post/6844903744170098695)**
- **18个测试断言**


<br/>

***
<br/>

># 18个测试断言

```


1.  XCTFail(format…) 生成一个失败的测试；

2.  XCTAssertNil(a1, format…)为空判断，a1为空时通过，反之不通过；

3.  XCTAssertNotNil(a1, format…)不为空判断，a1不为空时通过，反之不通过；

4.  XCTAssert(expression, format…)当expression求值为TRUE时通过；

5.  XCTAssertTrue(expression, format…)当expression求值为TRUE时通过；

6.  XCTAssertFalse(expression, format…)当expression求值为False时通过；

7.  XCTAssertEqualObjects(a1, a2, format…)判断相等，[a1 isEqual:a2]值为TRUE时通过，其中一个不为空时，不通过；

8.  XCTAssertNotEqualObjects(a1, a2, format…)判断不等，[a1 isEqual:a2]值为False时通过；

9.  XCTAssertEqual(a1, a2, format…)判断相等（当a1和a2是 C语言标量、结构体或联合体时使用,实际测试发现NSString也可以）；

10.  XCTAssertNotEqual(a1, a2, format…)判断不等（当a1和a2是 C语言标量、结构体或联合体时使用）；

11.  XCTAssertEqualWithAccuracy(a1, a2, accuracy, format…)判断相等，（double或float类型）提供一个误差范围，当在误差范围（+/-accuracy）以内相等时通过测试；

12.  XCTAssertNotEqualWithAccuracy(a1, a2, accuracy, format…) 判断不等，（double或float类型）提供一个误差范围，当在误差范围以内不等时通过测试；

13.  XCTAssertThrows(expression, format…)异常测试，当expression发生异常时通过；反之不通过；（很变态）

14.  XCTAssertThrowsSpecific(expression, specificException, format…) 异常测试，当expression发生specificException异常时通过；反之发生其他异常或不发生异常均不通过；

15.  XCTAssertThrowsSpecificNamed(expression, specificException, exception_name, format…)异常测试，当expression发生具体异常、具体异常名称的异常时通过测试，反之不通过；

16.  XCTAssertNoThrow(expression, format…)异常测试，当expression没有发生异常时通过测试；

17.  XCTAssertNoThrowSpecific(expression, specificException, format…)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过；

18.  XCTAssertNoThrowSpecificNamed(expression, specificException, exception_name, format…)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过

特别注意下XCTAssertEqualObjects和XCTAssertEqual。

XCTAssertEqualObjects(a1, a2, format…)的判断条件是[a1 isEqual:a2]是否返回一个YES。

XCTAssertEqual(a1, a2, format…)的判断条件是a1 == a2是否返回一个YES。

对于后者，如果a1和a2都是基本数据类型变量，那么只有a1 == a2才会返回YES

```



<br/>

***
<br/>



&emsp;  `OCTest 用RunTime来实现的。`
&emsp;  单元测试每次测试只测试一个，属于白盒测试。

```

//接口暴露，依赖注入
- (int)addNumber:(int)numberOne withOther:(int)numberTow;    //在.h文件中暴露

# 一个基本的测试结构,俗称： 黄金三段式
//1.  Given 准备测试条件，创建测试条件
int a = 1;
int b = 1;

//2.  When 调用我们的测试方法
int c = [self.vc addNumber:a withOther:b];

//3.  Then  断言，是否得到我们的预期值
XCTAssertEqual(c, 2);

```


<br/>

***
<br/>

参考资料：
[XCTestExpectation](https://www.jianshu.com/p/915cc8830bd2)
[](http://www.cocoachina.com/ios/20170426/19129.html)
[](http://www.cocoachina.com/ios/20170718/19930.html)
