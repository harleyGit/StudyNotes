># Future(I)
***

<br/>

## async和await

案例Demo代码段：
```
  //HTTP的get请求返回值为Future<String>类型，即其返回值未来是一个String类型的值
  //async关键字声明该函数内部有代码需要延迟执行
  getData() async {    
  	//await关键字声明运算为延迟执行，然后return运算结果
    return await http.get(Uri.encodeFull(url), headers: {"Accept": "application/json"}); 
  }

  //调用这个函数，想获取其结果
  String data = getData();

```
会报错：
![报错图片](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Flutter/Pictures/future1.png)

原因：因为data是String类型，而函数getData()是一个异步操作函数，其返回值是一个await延迟执行的结果。`在Dart中，有await标记的运算，其结果值都是一个Future对象，Future不是String类型，所以就报错了。`

解决：
```
String data;
setData() async {
  //getData()延迟执行后赋值给data
  data = await getData();    
}

```

*`记住两点：`*

-	await关键字必须在async函数内部使用
-	调用async函数必须使用await关键字




















<br/>

***

<br/>

># 参考资料
-	[异步async、await和Future的使用技巧](https://juejin.im/post/6844903591409352718)