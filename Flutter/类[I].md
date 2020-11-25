`Dart` 中所有的类都继承自 `Object`类。[Flutter 中文官方文档](https://book.flutterchina.club/chapter14/flutter_app_startup.html)
- **`类的方法`**
- **`子类(类的继承)`**
- **`Flutter特殊语法`**
- **``**
- **``**


># 类的方法

- **`构造方法`**

```
class Person {
  String name;
  int age;

  //默认构造函数只能写一个
  Person(this.name, this.age);
  /*
    ///实例化之前做的操作，实例化列表
    //遇上面的实例化只能存在一个
    Person(): name = "李白", age = 28 {
    print("实例化之前的操作：name: ${this.name}, age: ${this.age}");
  }
  */

  //命名构造函数可以写多个
  Person.info(){
    print("这个是命名构造函数");
  }

  void printInfo() {
      print("姓名： ${this.name}\n,年龄：${this.name},${20+80}");
    }
}


///调用
void testCustomClass() {
    print("<-----------------------------构造方法：start------------------------------>");
    Person person = Person("荆轲", 27);
    person.printInfo();
    Person person1 = Person.info();
    print("<-----------------------------构造方法：end------------------------------>");

}
```
打印：
```
flutter: <-----------------------------构造方法：start------------------------------>
flutter: 姓名： 荆轲,
         年龄：荆轲,100
flutter: 这个是命名构造函数
flutter: <-----------------------------构造方法：end------------------------------>
```


<br/>
- **`get、 set方法`**

```
/// get 方法
  get getInfo{
    print("get 方法的书写格式, 要把()去掉，这是一个计算属性，一般是返回一个计算值");
    print("姓名： ${this.name},\n         年龄：${this.name},${20+80}");
    return 20 + 80;
  }
  
  
  /// set方法
  set userName(name) {
    print("set 方法: 设置 name");
    this.name = name;
  }


    ///调用get、set方法
    print("<-----------------------------start------------------------------>");
    person.userName = "嬴政";
    var length = person.getInfo;
    print("get 计算返回的长度是： ${length}\n\n");
    print("<-----------------------------end------------------------------>");
```

打印：

```
flutter: <-----------------------------start------------------------------>
flutter: set 方法: 设置 name
flutter: get 方法的书写格式, 要把()去掉，这是一个计算属性，一般是返回一个计算值
flutter: 姓名： 嬴政,
         年龄：嬴政,100
flutter: get 计算返回的长度是： 100
flutter: <-----------------------------end------------------------------>
```





<br/>

- **`连缀书写`**

```
///连缀书写
    print("<-----------------------------连缀书写：start------------------------------>");
    Person person = Person("荆轲", 27);
    person..name = "盘古"
          ..age = 30
          ..getInfo;
    print("<-----------------------------连缀书写：end------------------------------>");
```

打印：

```
flutter: <-----------------------------连缀书写：start------------------------------>
flutter: get 方法的书写格式, 要把()去掉，这是一个计算属性，一般是返回一个计算值
flutter: 姓名： 盘古,
         年龄：盘古,100
flutter: <-----------------------------连缀书写：end------------------------------>
```




<br/>

***
<br/>
<br/>

># 子类(类的继承)
- **`类的继承构造方法`**

```
class Student extends Person {
  String sex;
  Student(String name, int age, String sex): super(name, age){
    this.sex = sex;
  }
  get getStudentInfo {
    print("学生信息：name：${this.name},  age：${this.age}, sex：${this.sex}");
  }
}

///调用
    print("<-------------------------------get 方法：start------------------------------->");
    Student student = Student("李白", 49, "中性");
    student.getInfo;
    print("<-------------------------------get 方法：start------------------------------->");
```
打印：
```
flutter: <-------------------------------get 方法：start------------------------------->
flutter: get 方法的书写格式, 要把()去掉，这是一个计算属性，一般是返回一个计算值
flutter: 姓名： 李白,
         年龄：李白,100
flutter: <-------------------------------get 方法：start------------------------------->
```







<br/>

***
<br/>

># Flutter 特殊语法
<br/>

- **`匿名方法`**

```
    ///匿名方法
    var fn = (){
      print("我是匿名方法");
    };
    fn();

    var printNum = (int n){
      print("匿名方法带参数：($n+100)");
    };
    printNum(100);
```
打印：
`flutter: 我是匿名方法`
`flutter: 匿名方法带参数：(100+100)`


<br/>

- **`箭头函数`**

```
    ///箭头函数
    List listStr = ["吕布", "貂蝉", "诸葛亮", "曹操", "司马懿"];
    List listNum = [1, 3, 5, 8, 9, 12, 14, 18];

    listStr.forEach((value){
      print("$value");
    });
    //箭头后只有一行代码
    listStr.forEach((value)=> print("---->> $value"));
    listStr.forEach((value)=>{
      print("++++++>> $value")
    });

    var newListNum2 = listNum.map((value) => value > 2 ? value * 3 : value);
    print(newListNum2);
```

打印：

```
flutter: 吕布
flutter: 貂蝉
flutter: 诸葛亮
flutter: 曹操
flutter: 司马懿
flutter: ---->> 吕布
flutter: ---->> 貂蝉
flutter: ---->> 诸葛亮
flutter: ---->> 曹操
flutter: ---->> 司马懿
flutter: ++++++>> 吕布
flutter: ++++++>> 貂蝉
flutter: ++++++>> 诸葛亮
flutter: ++++++>> 曹操
flutter: ++++++>> 司马懿
flutter: (1, 9, 15, 24, 27, 36, 42, 54)
```



<br/>

- **`自执行方法`**

```
     ///自执行方法
    ((int n){
      print("自执行方法: $n");
    })(12);
```
打印：
<br/>
`flutter: 自执行方法: 12`



<br/>

- **`闭包`**

```
    ///闭包
    fn1(){
      var a = 123;
      return(){
        print("闭包： $a");
      };
    }

    var b = fn1();//相当于把return内的函数赋值给了b
    b();
    b();
    b();
```
打印：
```
flutter: 闭包： 123
flutter: 闭包： 123
flutter: 闭包： 123
```


<br/>

- **`闭包`**

```
 T getData<T>(T value){
    return value;
  }


    ///范型
    print(this.getData("范型：value"));
    print(this.getData(1212341));
    print(getData<String>("泛型 傻逼"));

```
打印：
```
flutter: 范型：value
flutter: 1212341
flutter: 泛型 傻逼
```

