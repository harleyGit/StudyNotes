



- **[tsconfig 配置文件](http://developer.egret.com/cn/github/egret-docs/Engine2D/projectConfig/tsconfig/index.html)**
- **[扩展库简介](http://developer.egret.com/cn/github/egret-docs/Engine2D/projectConfig/extendRepSummary/index.html)**
	- res资源加载库
	- eui界面制作库
	- DragonBones龙骨动画库
	- game库
	- socket库
	- egret实验功能库

<br/>

***
<br/>

># tsconfig 配置文件

```


{
    "compilerOptions": {
		//target:编译之后生成的JavaScript文件需要遵循的标准，默认为 es5，兼容性比较好，不建议修改
        "target": "es5",
        
        //outDir:编译出来的js文件，放到哪个目录下，默认编译到 bin-debug 里，目前暂不支持修改
        "outDir": "bin-debug",
        
        //experimentalDecorators:启用实验性质的语法装饰器，引擎里的部分库使用了最新的语法，需要开启这个配置
        "experimentalDecorators": true,
        
        //lib: 编译需要的库文件，默认有3个，你可以根据需求自行添加
        "lib": [
            "es5",
            "dom",
            "es2015.promise"
        ],
        "types": []
    },
    "include": [
        "src",
        "libs"
    ]
    
    /**
	    *其他常用参数:
		"sourceMap": true:把.ts 文件编译成.js 文件时，生成对应的 .js.map 文件，该文件可以让用户直接在浏览器里调试 ts 文件
		
		"removeComments": true: 编译 .js 时删除原本 .ts 文件中的注释。

    */
}


```




<br/>

***
<br/>

># 扩展库简介

<br/>

> **[res资源加载库](http://developer.egret.com/cn/github/egret-docs/extension/RES/newres/index.html)**

- **资源加载配置**

以 **`resource.json`** 为例：

```

{
"resources":
    [
		    /**
		    * name：表示这个资源的唯一短名标识符。
		    * type：表示资源类型。
		    * url：表示当前资源文件的路径。
		    */
        {"name":"bgImage","type":"image","url":"assets/bg.jpg"},
        {"name":"egretIcon","type":"image","url":"assets/egret_icon.png"},
        {"name":"description","type":"json","url":"config/description.json"}
    ],
    
    /**
    * “groups” 是预加载资源组的配置，每项是一个资源组，每一个资源组须包含两个属性：
    * name：表示资源组的组名
    * keys：表示这个资源组包含哪些资源，里面的逗号分隔的每一个字符串，都与“resource”下的资源name对应。同一个资源可以存在于多个资源组里。
    */
"groups":
    [
        {"name":"preload","keys":"bgImage,egretIcon"}
    ]
}

```


<br/>

- **载入资源加载配置**

RES模块对资源加载配置有两种读取方式，一种是通过`配置读取方式`，另一种是通过`路径读取方式`。


<br/>

- 配置读取方式

这是一个json文件，通常我们取名为default.res.json。载入代码如下：

```
RES.addEventListener( RES.ResourceEvent.CONFIG_COMPLETE, this.onConfigComplete, this ); 
RES.addEventListener( RES.ResourceEvent.CONFIG_LOAD_ERROR, this.onConfigLoadErr, this ); 
RES.loadConfig("resource/default.res.json","resource/");

```

loadConfig函数做执行的动作即为初始化RES资源加载模块。该函数包含两个参数，第一个参数是default.res.json文件的完整路径，第二个参数是配置中每个资源项url相对路径的基址。例如配置里的bgImage资源项填的url是assets/bg.jpg，加载时将会拼接为相对路径：resource/assets/bg.jpg。

若需要在初始化完成后再做一些处理，监听ResourceEvent.CONFIG_COMPLETE事件即可。

当然，载入配置也难保证完全不出差错，所以最好监听 ResourceEvent.CONFIG_LOAD_ERROR事件，并在处理函数做一些error log处理之类。


<br/>

- 路径读取方式

路径读取方式就是免去了加载配置文件的过程。直接将资源加载配置内容以参数方式给出。

如果是项目内资源，相对目录为主目录而不是 RES.loadCOnfig 中设置的目录，比如：resource/assets/bg.jpg

如果是外部资源，请使用资源的绝对地址，比如：http://xxx/a.png

```
class Main extends egret.DisplayObjectContainer {
    public constructor() {
        super();
        this.addEventListener(egret.Event.ADDED_TO_STAGE, this.onAddToStage, this);
    }
    private onAddToStage(event:egret.Event) {
        RES.getResByUrl('resource/assets/bg.jpg',this.onComplete,this,RES.ResourceItem.TYPE_IMAGE);
    }
    private onComplete(event:any):void {
        var img: egret.Texture = <egret.Texture>event;
        var bitmap: egret.Bitmap = new egret.Bitmap(img);
        this.addChild(bitmap);
    }
}
```








<br/>


> **[eui界面制作库](http://developer.egret.com/cn/github/egret-docs/extension/EUI/outline/introduction/index.html)**

EUI是一套基于Egret核心显示列表的UI扩展库，它封装了大量的常用UI组件，能够满足大部分的交互界面需求，即使更加复杂的组件需求，您也可以基于EUI已有组件进行组合或扩展，从而快速实现需求。

`Egret UI` 的基础教程，我们假定您已经掌握了Egret核心库的一些基本使用经验，比如显示对象，事件等等，如果您还不具备这些基础知识，建议先从 [**Get Started**](https://docs.egret.com/engine/docs/getStarted/helloWorld) 开始学起。


![z40](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z40.png)






<br/>


> DragonBones龙骨动画库


> game库


> socket库


> egret实验功能库




<br/>

***
<br/>



<br/>

***
<br/>


<br/>

***
<br/>


