># 开源接口网站
&emsp;  [万维易源](https://www.showapi.com)开源接口，当然有的需要收费的。本来想用Charles进行抓取接口，但是发现抓不到。后来通过Safari进行查看网络请求终于可以了。如下图：
![调用请求中的 接口查看](https://upload-images.jianshu.io/upload_images/2959789-e81b3d8bf101d229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
- **Dio Get请求**

```
///Get请求
  static getTest() async {
   
    try{
      Response response;
      Dio dio = new Dio();
      Map<String, dynamic> headers = new Map();
      //万维易源：https://www.showapi.com/apiGateway/onlineTest?apiCode=894&pointCode=5
      //cookie: 用来进行判断是否登录了
      headers['Cookie'] = ' Hm_lpvt_2b8b8260aa5e70de7faf1de4f9221ee0=1589516451; Hm_lvt_2b8b8260aa5e70de7faf1de4f9221ee0=1589512856; showApiAuth=KY71p_fk9pk0oQdV2of793G5l0CD4-bk3UF_x-UYtIp8MTZnD7OwpVF1GPQcs9Id3NRCQNOVEtQ; JSESSIONID=372AC98B88D5EAE14137E8AD67C36AEE';
      Options options = new Options(
        headers: headers
      );

      var data={'showapi_pointId':'5dd7975c0cf247735cd30a31', 'showapi_pointCode': '5', 'showapi_appid': '206561', 'showapi_sign': '479562cbc0dea0a2f14e27c661a8e6af', 'showapi_timestamp': '20200515130459', 'day': '20200509'};
      response = await Dio().get(
        "https://www.showapi.com/apiGateway/generateSignUrl?showapi_apiCode=894",
          queryParameters:data,
          options: options
      );
      return print('Get 请求数据：$response');
    }catch(e){
      return print(e);
    }

  }
```

打印：
![Get 数据打印](https://upload-images.jianshu.io/upload_images/2959789-994e56f543863216.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






















