![UITabBarController 内部构造图](https://upload-images.jianshu.io/upload_images/2959789-782e2483f7e29b4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> &emsp; 因为UITabBarController类继承自UIViewController类，标签栏控制器有自己的视图，可以通过视图属性访问。标签栏控制器的视图只是标签栏视图和包含自定义内容的视图的容器。选项卡栏视图为用户提供了选择控件，并由一个或多个选项卡栏项组成。图2显示了如何组装这些视图以显示整个选项卡栏接口。尽管选项卡栏和工具栏视图中的项目可以更改，但管理它们的视图不会更改。只有自定义内容视图更改以反映当前选中选项卡的视图控制器。
<br/>
&emsp; 您可以使用导航控制器或自定义视图控制器作为选项卡的根视图控制器。如果根视图控制器是一个导航控制器，标签栏控制器会进一步调整显示的导航内容的大小，这样它就不会与标签栏重叠。因此，在选项卡栏接口中显示的任何视图都应该设置其autoresizingMask属性，以便在任何条件下适当调整视图的大小。


https://www.jianshu.com/p/f1b0be2c9735
https://www.jianshu.com/p/a7c5f4028475
https://www.cnblogs.com/jukaiit/p/5066468.html
https://www.cnblogs.com/code-cd/p/4810408.html



