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
