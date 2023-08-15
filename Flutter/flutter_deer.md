

> <h2 id=''></h2>
- [**方法**](#方法)
	- [addPostFrameCallback](#addPostFrameCallback)
	- [showGeneralDialog](#showGeneralDialog)
	- [函数作为参数](#函数作为参数)
	- [Theme](#Theme)
- [**工具类**](#工具类)
	- [图片工具](#图片工具)
- [**组件库**](#组件库)
	- [Dio网络库](#Dio网络库)
		- [扩展拦截器类](#扩展拦截器类)
	- [fluro路由组件](#fluro路由组件)
	- [cached_network_image-图片加载](#cached_network_image-图片加载)
	- 	[flutter_ume调试工具](#flutter_ume调试工具)
	- [common_utils-工具库](#common_utils-工具库)
	- [keyboard_actions处理键盘事件](#keyboard_actions处理键盘事件)
	- [sp_util-保存对象](#sp_util-保存对象)
	- [sprintf格式化字符串](#sprintf格式化字符串)
- **参考资料**
	- [**flutter_deer**](https://github.com/simplezhli/flutter_deer)
	- [Flutter web真机测试](https://juejin.cn/post/7091184835842015268)
	- [Deer](https://weilu.blog.csdn.net/article/details/90546727)项目知识点集合




<br/>

***
<br/>


> <h1 id='方法'>方法</h1>


<br/>

> <h2 id='addPostFrameCallback'>addPostFrameCallback</h2>



&nbsp;用法：`addPostFrameCallback`回调方法在Widget渲染完成时触发，所以一般我们在获取页面中的Widget大小、位置时使用到。

&emsp;  若我们在接口请求前弹出loading。例如我将请求方法放在了initState方法中，可能出现的异常如下：

```
inheritFromWidgetOfExactType(_InheritedTheme) or inheritFromElement() was called before initState() completed.
When an inherited widget changes, for example if the value of Theme.of() changes, its dependent widgets are rebuilt. If the dependent widget’s reference to the inherited widget is in a constructor or an initState() method, then the rebuilt dependent widget will not reflect the changes in the inherited widget.
Typically references to inherited widgets should occur in widget build() methods. Alternatively, initialization based on inherited widgets can be placed in the didChangeDependencies method, which is called after initState and whenever the dependencies change thereafter.
```

&emsp;  所以我们需要在build完成后，才可以去创建这个新的组件（这里就是Dialog）。

&emsp;  所以解决方法就是使用addPostFrameCallback回调方法，等待页面build完成后在请求数据：

```
  @override
  void initState() {
    WidgetsBinding.instance.addPostFrameCallback((_){
      /// 接口请求
    });
  }
``` 

&emsp;  导致这类问题的场景很多，但是大体解决思路就是上述的[办法](https://weilu.blog.csdn.net/article/details/94849020)。



<br/><br/>

># <h2 id='showGeneralDialog'>[showGeneralDialog](https://blog.csdn.net/Calvin_zhou/article/details/115768013)</h2>

展示弹窗


<br/><br/>

> <h2 id='函数作为参数'>函数作为参数</h2>


- 1.在调用页面，声明一个作为参数的函数tap

```
tap(int i){
    print('选择了....'+i.toString());
}
```


<br/>

- 2、声明自定义的函数,设置形参,点击调用参数

```
InkWell formateItem( Function(int) tapAction){
  child: Container(),
  onTap: () {
      tapAction(1688);
    }
}
```

<br/>

- 3、调用，将tapAction传入

```
formateItem(tap)
```

<br/>

- 4、打印

```
Flutter: 选择了 1688
```




<br/><br/>


> <h2 id='Theme'>Theme</h2>

主题统一颜色和字体风格

使用:

```
extension ThemeExtension on BuildContext {
  bool get isDark => ThemeUtils.isDark(this);
  //Theme 主题: https://flutter.cn/docs/cookbook/design/themes
  Color get backgroundColor => Theme.of(this).scaffoldBackgroundColor;
  Color get dialogBackgroundColor => Theme.of(this).canvasColor;
}
```




<br/>

***
<br/><br/>

> <h1 id='工具类'>工具类</h1>


<br/><br/>

> <h2 id='图片工具'>图片工具</h2>

[**AssetImage:**](https://doc.flutterchina.club/assets-and-images/) 加载图片;

获取图片:

```
new AssetImage('graphics/background.png'),
```


<br/>

网络图片加载:

[CachedNetworkImageProvider](#cached_network_image-图片加载)加载网络图片

```
static ImageProvider getImageProvider(String? imageUrl, {String holderImg = 'none'}) {
    if (TextUtil.isEmpty(imageUrl)) {//判断是否为空
      return AssetImage(getImgPath(holderImg));
    }
    //网络图片加载:https://blog.csdn.net/qq_33224517/article/details/118079879
    return CachedNetworkImageProvider(imageUrl!);
  }
```


<br/>

***
<br/>


> <h1 id='组件库'>组件库</h1>


<br/><br/>

># <h2 id='Dio网络库'>[Dio网络库](https://github.com/cfug/dio/blob/main/README-ZH.md)</h2>




<br/><br/>

> <h2 id='fluro路由组件'>fluro路由组件</h2>


使用:

```
FluroRouter().navigateTo(context, path,
  replace: replace,
  clearStack: clearStack,
  transition: TransitionType.native,//原生形式
  routeSettings: RouteSettings(
    arguments: arguments,
  ),
);
```



<br/><br/>

> <h3 id='扩展拦截器类'>扩展拦截器类</h3>

我们可以创建一个拦截器类来覆盖onRequest、onError、onResponse方法。

```
class AppInterceptors extends Interceptor {
  @override
  FutureOr<dynamic> onRequest(RequestOptions options) async {
    if (options.headers.containsKey("requiresToken")) {
      //remove the auxiliary header
      options.headers.remove("requiresToken");

      SharedPreferences prefs = await SharedPreferences.getInstance();
      var header = prefs.get("Header");

      options.headers.addAll({"Header": "$header${DateTime.now()}"});

      return options;
    }
  }

  @override
  FutureOr<dynamic> onError(DioError dioError) {
    if (dioError.message.contains("ERROR_001")) {
      // this will push a new route and remove all the routes that were present
      navigatorKey.currentState.pushNamedAndRemoveUntil(
          "/login", (Route<dynamic> route) => false);
    }

    return dioError;
  }

  @override
  FutureOr<dynamic> onResponse(Response options) async {
    if (options.headers.value("verifyToken") != null) {
      //if the header is present, then compare it with the Shared Prefs key
      SharedPreferences prefs = await SharedPreferences.getInstance();
      var verifyToken = prefs.get("VerifyToken");

      // if the value is the same as the header, continue with the request
      if (options.headers.value("verifyToken") == verifyToken) {
        return options;
      }
    }

    return DioError(request: options.request, message: "User is no longer active");
  }
}
```


该类可以被轻松的添加到dio对象的拦截器集合中：

```
Dio addInterceptors(Dio dio) {
  dio.interceptors.add(AppInterceptors());
}
```



<br/><br/>

> <h3 id='cached_network_image-图片加载'>cached_network_image-图片加载</h3>

```
//添加占位图
CachedNetworkImage(
	imageUrl: "http://via.placeholder.com/350x150",
	placeholder: (context, url) => CircularProgressIndicator(),
	errorWidget: (context, url, error) => Icon(Icons.error),
),
   
   
     
//进度条展示
CachedNetworkImage(
    imageUrl: "http://via.placeholder.com/350x150",
    progressIndicatorBuilder: (context, url, downloadProgress) => 
            CircularProgressIndicator(value: downloadProgress.progress),
    errorWidget: (context, url, error) => Icon(Icons.error),
),
```






<br/><br/>

> <h3 id=''></h3>









<br/><br/>

> <h2 id='flutter_ume调试工具'>[flutter_ume调试工具](https://github.com/bytedance/flutter_ume/blob/master/README_cn.md)</h2>










<br/><br/>

># <h2 id='common_utils-工具库'>[common_utils-工具库](https://github.com/Sky24n/common_utils)</h2>


Dart常用工具类库。包含日期，正则，倒计时，时间轴等工具类。如果你有好的工具类欢迎PR.


Dart常用工具类库 **common_utils**

- TimelineUtil : 时间轴.
- TimerUtil : 倒计时，定时任务.
- MoneyUtil : 精确转换，元转分，分转元，支持格式输出.
- LogUtil : 简单封装打印日志.
- DateUtil : 日期转换格式化输出.
- RegexUtil : 正则验证手机号，身份证，邮箱等等.
- NumUtil : 保留x位小数, 精确加、减、乘、除, 防止精度丢失.
- ObjectUtil : 判断对象是否为空(String List Map),判断两个List是否相等.
- EncryptUtil : 异或对称加/解密，md5加密，Base64加/解密.
- TextUtil : 银行卡号每隔4位加空格，每隔3三位加逗号，隐藏手机号等等.
- JsonUtil : 简单封装json字符串转对象.


<br/>
<br/>


Flutter常用工具类库 [flustars](https://github.com/Sky24n/flustars)

- SpUtil : 单例"同步"SharedPreferences工具类。支持get传入默认值，支持存储对象，支持存储对象数组。
- ScreenUtil : 屏幕适配，获取屏幕宽、高、密度，AppBar高，状态栏高度，屏幕方向.
- WidgetUtil : 监听Widget渲染状态，获取Widget宽高，在屏幕上的坐标，获取网络/本地图片尺寸.
- ImageUtil : 获取网络/本地图片尺寸.
- DirectoryUtil : 文件目录工具类.
- DioUtil : 单例Dio网络工具类(已迁移至此处DioUtil)。







<br/>

># <h2 id='keyboard_actions处理键盘事件'>[keyboard_actions处理键盘事件](https://github.com/diegoveloper/flutter_keyboard_actions)</h2>





<br/>
<br/>


> <h2 id='sp_util-保存对象'>sp_util-保存对象</h2>



<br/>
<br/>


> <h2 id='sprintf格式化字符串'>sprintf格式化字符串</h2>


[格式化字符串](https://www.jianshu.com/p/78ca5e599361)




<br/>
<br/>


> <h2 id=''></h2>





<br/>
<br/>


> <h2 id=''></h2>




<br/>
<br/>


> <h2 id=''></h2>





<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>




<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>






<br/>
<br/>


> <h2 id=''></h2>




