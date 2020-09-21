self. 调用property自动生成getter 和 setter 方法，而 _ 则是直接调用实例例变量。

使用readOnly 属性，可以使用 setValue:  forKey:  方法对其值进行修改；

```
//访问器寻找名称的成员变量
+ (Bool)  accessInstanceVariablesDierctly {
                return NO;
}

``` 


assign：修饰OC基本数据类型，不会使对象的引用类型计数 +1。
