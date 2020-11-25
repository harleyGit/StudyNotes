- 抽象类
- 范型限制

<br/>

***
<br/>

># 抽象类

- *`抽象类介绍`*

![抽象类介绍](https://upload-images.jianshu.io/upload_images/2959789-36ff756764386d18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
- *`抽象类`*

```
abstract class Animal {
  //抽象方法子类必须实现
  eat();  

  void animalBaseInfo() {
    print("抽象类的公共普通方法：动物的基本信息");
  }
} 


class Dog extends Animal {
  
  @override
  eat() {
    print("小狗🐶🐶🐶🐶🐶🐶在吃骨头");
  }

}


///调用
    print("<-------------------------------抽象类：start------------------------------->");
    Dog dog = Dog();
    dog.eat();
    dog.animalBaseInfo();
    print("<-------------------------------抽象类：start------------------------------->");

```
打印：
```
flutter: <-------------------------------抽象类：start------------------------------->
flutter: 小狗🐶🐶🐶🐶🐶🐶在吃骨头
flutter: 抽象类的公共方法：动物的基本信息
flutter: <-------------------------------抽象类：start------------------------------->
```







<br/>

***
<br/>

>#  多态

**`多态介绍`**

![Datr 多态介绍](https://upload-images.jianshu.io/upload_images/2959789-f6986eebaafc99dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
abstract class Animal {
  //抽象方法必须实现
  eat();  

  void animalBaseInfo() {
    print("抽象类的公共普通方法：动物的基本信息");
  }

  // 默认实现run方法
  void run() {}
}  



class Dog extends Animal {
  
  @override
  eat() {
    print("小狗🐶🐶🐶🐶🐶🐶在吃骨头");
  }
  
  void run() {
    print("小狗🐶 在画梅花，在❄️地🩸！！！！！！");
  }
}

class Cat extends Animal {
  @override
  eat() {
    print("🐱 在抓🐭，😂");
  }

  void run() {
    print("🐱小猫 在蹦跑！！！！！！");
  }
}





/// 调用
    print("<-------------------------------抽象类：start------------------------------->");
    Dog dog = Dog();
    dog.eat();
    dog.animalBaseInfo();

    Cat cat = Cat();
    cat.eat();
    cat.animalBaseInfo();

    print("<-------------------------------抽象类：start------------------------------->");
```
打印：
```
flutter: <-------------------------------抽象类：start------------------------------->
flutter: 小狗🐶🐶🐶🐶🐶🐶在吃骨头
flutter: 抽象类的公共普通方法：动物的基本信息
flutter: 🐱 在抓🐭，😂
flutter: 抽象类的公共普通方法：动物的基本信息
flutter: <-------------------------------抽象类：start------------------------------->
```

或者如下，通过指针赋值进行调用：
```
///多态指针赋值
    print("<-------------------------------抽象类：start------------------------------->");
    Animal d = new Dog();
    d.eat();

    Animal c = new Cat();
    c.run();
    print("<-------------------------------抽象类：start------------------------------->");
```
打印：
```
flutter: <-------------------------------抽象类：start------------------------------->
flutter: 小狗🐶🐶🐶🐶🐶🐶在吃骨头
flutter: 🐱小猫 在蹦跑！！！！！！
flutter: <-------------------------------抽象类：start------------------------------->
```



<br/>

***
<br/>

># 接口类

![Java 和 Dart 的接口区别](https://upload-images.jianshu.io/upload_images/2959789-7424f8c5b7cf2d52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

**`Code Demo`**

```
/// 接口定义
abstract class  DataBase {

  String url;

  ///当作借口 接口： 就是约定、规范
  void add();
  void delete();
  void update();
  void search();
}


class  MySQL implements DataBase {

  @override
  String url;

  MySQL(this.url);
  
  @override
  void add() {
    // TODO: implement add
    print("MySql 的添加方法");
  }

  @override
  void delete() {
    // TODO: implement delete
    print("MySql 的删除方法");
  }

  @override
  void search() {
    // TODO: implement search
    print("MySql 的查找方法");
  }

  @override
  void update() {
    // TODO: implement update
    print("MySql 的修改方法");
  }

  
}



/// 调用
    print("<-------------------------------接口：start------------------------------->");
    var mysql = MySQL("url 地址");
    mysql.add();
    mysql.delete();
    mysql.update();
    mysql.search();

    print("<-------------------------------接口：start------------------------------->");


```
打印：
```
flutter: <-------------------------------接口：start------------------------------->
flutter: MySql 的添加方法
flutter: MySql 的删除方法
flutter: MySql 的修改方法
flutter: MySql 的查找方法
flutter: <-------------------------------接口：start------------------------------->
```



<br/>

***
<br/>

># Mixins 功能

![mixins 的功能介绍](https://upload-images.jianshu.io/upload_images/2959789-a5d9934c12e62715.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
class A {
  void printAInfo() {
    print("------>>: 打印A的信息");
  }
}

class B {
  void printBInfo() {
    print("++++>>>>>: 打印B的信息");
  }
}

class C with A, B {
  
}


/// 调用
    print("<-------------------------------Minxins：start------------------------------->");
    var c = new C();
    c.printAInfo();
    c.printBInfo();

    print("<-------------------------------Minxins：start------------------------------->");


```
打印：
```
flutter: <-------------------------------Minxins：start------------------------------->
flutter: ------>>: 打印A的信息
flutter: ++++>>>>>: 打印B的信息
flutter: <-------------------------------Minxins：start------------------------------->
```





<br/>

***
<br/>


># 异步和同步

-  #`库的介绍`

![库的介绍](https://upload-images.jianshu.io/upload_images/2959789-dd8d11e8be856772.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- #`async 和 await 关键字介绍`

![async 和 await 关键字介绍](https://upload-images.jianshu.io/upload_images/2959789-b387906ee08ba9ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
class SimpleNetwork {
   testAsync() async {
    return "Hello Flutter!";
  }

  ///API 接口：http://news-at.zhihu.com/api/3/stories/latest
  ///_方法名，在一个类文件中是一个私有方法：_getDataFromZhihuAPI
  getDataFromZhihuAPI() async {
    //创建HttpClient对象
    var httpClient = new HttpClient();
    //创建url对象
    var url = new Uri.http('news-at.zhihu.com', '/api/3/stories/latest');
    //发起请求，等待请求(请求数据是一个异步，异步改为同步需要用到dart中的await关键字，但是awiat必须用到异步方法中，所以_getDataFromZhihuAPI中有async关键字)
    var request = await httpClient.getUrl(url);
    //关闭请求，等待响应
    var response = await request.close();
    //解码响应的内容
    return await response.transform(utf8.decoder).join();
  }
}


/// 调用
//注意： testCustomClass 方法中也要用async，与 await 是一一对应的
void testCustomClass() async {
    print("<-------------------------------Minxins：start------------------------------->");
    var c = new SimpleNetwork();
    var dataJson = await c.getDataFromZhihuAPI();
    print("请求数据是：${dataJson}");

    print("<-------------------------------Minxins：start------------------------------->");
 }

```

打印：

```
flutter: <-------------------------------Minxins：start------------------------------->
flutter: 请求数据是：{"date":"20200406","stories":[{"image_hue":"0x444444","title":"小事 · 精神分裂症患者眼中的世界是什么样的？","url":"https:\/\/daily.zhihu.com\/story\/9722439","hint":"VOL.1185","ga_prefix":"040622","images":["https:\/\/pic2.zhimg.com\/v2-ddaa2be2068324b52c55460e38ff4f21.jpg"],"type":0,"id":9722439},{"image_hue":"0x15181f","title":"你心中古装剧 Top1 是哪一部？","url":"https:\/\/daily.zhihu.com\/story\/9722449","hint":"包茅子 · 3 分钟阅读","ga_prefix":"040620","images":["https:\/\/pic2.zhimg.com\/v2-39046a4132a4fc7eb13fce41d11e1895.jpg"],"type":0,"id":9722449},{"image_hue":"0xb0867b","title":"有哪些高效看文献的方法？","url":"https:\/\/daily.zhihu.com\/story\/9722455","hint":"宝珠道人 · 3 分钟阅读","ga_prefix":"040616","images":["https:\/\/pic2.zhimg.com\/v2-5964966c2419e79532ff78fd715b4e61.jpg"],"type":0,"id":9722455},{"image_hue":"0xb38b7d","title":"美国现在的疫情发展到什么程度了？","url":"http<…>
flutter: <-------------------------------Minxins：start------------------------------->

```


<br/>

- **使用第三方库**


![使用第三方库](https://upload-images.jianshu.io/upload_images/2959789-7d466b662427922d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意： 途中的 终端命令：put get 错误，应该是 **`flutter pub get`**。



<br/>

***
<br/>


># 范型限制

```

/*
*  <T extends BaseStatefulWidget,K extends BaseBloc>: 限制传入的类型是 BaseStatefulWidget(或者是其子类) 和 BaseBloc(或者是其子类)
*
* TemplateBarState<T>： 表示继承自 TemplateBarState(或者是其子类)， <T> 表示限制传入的类型是 BaseStatefulWidget (或者是其子类)
*/
abstract class TemplateBlocBarState<T extends BaseStatefulWidget,
    K extends BaseBloc> extends TemplateBarState<T> {
  K kBloc;

  @override
  void initState() {
    super.initState();
    kBloc = absKBloc();
  }

  K absKBloc();
}



/* 
*  SettingPage 继承自：BaseStatefulWidget
*  UserBloc 继承自：AbsUserBloc， AbsUserBloc 继承自：BaseBloc
*/
class _SettingPageState extends TemplateBlocBarState<SettingPage, UserBloc> { 


}

```
