># Charles 抓取Flutter网络请求
&emsp; 不知怎么回事，Charles抓取原生网络请求轻而易举，但是在Flutter就是不行。后来在网上搜了下，知道了大概是不走代理造成的，经过多番尝试终于可以了。

- 查找本机的的`localhost`
  注意必须是本电脑的，乱写的是不可以的，在 Mac 终端输入：
```
$ nslookup localhost

//输出：
Server:		192.168.0.1
Address:	192.168.0.1#53

Name:	localhost//这就是我们要的localhost
Address: 127.0.0.1
```


- 使用Flutter的Dio网络请求组件，进行网络请求
```
RaisedButton(onPressed: () async {
          var dio = Dio();
          dio.options.contentType = "text";
          (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
              (HttpClient client) {
            client.findProxy = (uri) {
              //proxy all request to localhost:8888
              return "PROXY 127.0.0.1:8888";
            };
            client.badCertificateCallback =
                (X509Certificate cert, String host, int port) => true;
          };
          
          Response<String> response;
          response = await dio.get("http://www.test1111/blog");
          print(response.data);       
          
        },
      child: Text('Charles： Stream'),
   ),
```

- Charles 进行配置
Proxy -> macOS Proxy

- 选择自己映射到的Json文件

![选取当地映射](https://upload-images.jianshu.io/upload_images/2959789-5c977fea2d4e4a41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选取自己的json文件](https://upload-images.jianshu.io/upload_images/2959789-8bde3758017451ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 控制台打印：
![自定义的json文件](https://upload-images.jianshu.io/upload_images/2959789-4ed8b503482cd222.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


