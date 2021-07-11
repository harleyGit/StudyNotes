> <h1 id=""></h1>
- [**文件**](#文件)
- [**属性**](#属性)
	- [clientWidth](#clientWidth)
	- [JavaScript的childNodes和children](#JavaScript的childNodes和children)
	- [.innerWidth](#innerWidth)
	- [setTimeout](#setTimeout)
- [**问题**](#问题)
- [**开发心得**](#开发心得)
	- [flex使用](#flex心得)
	- [值为空](#值为空)





<br/>

***
<br/>
># <h1 id="文件">文件</h1>

<br/>

- 网络请求类： src > components-hybrid > mapi.js 
- 路由模块：src > router > config > config.js 
- 路由配置模块： WRouter.js
- CheckPathInterceptor： 检查路径，对路径进行处理拦截
- WRouterInterceptorChain： 路由导航拦截器
- ParamInterceptor： 路由参数


- WRouter： 路由导航处理

```
//根据路径获取路由信息
//从config.js中获取加的路由信息，放入allRouteMap数组中
//从而获取加入的路由信息比如路径、标题、导出的方法等
//其加载config.js中的路由信息是通过index.js中的 window.WRouter.init(config, WrappedComp.wrapped); 来进行获取的配置表的
getRouteInfo:function(path){}

```


- Mapi：执行一些原生的一些调用
- Window的调用：MapiExts类的调用


在sys-about.js 中 有这行代码：

```
_userAgreement(){
        this.props.navigateTo({
            pathname: '/system-setting/user-agreement'
        });
    }
```
在 app.js中找到 `cloneElement` 这个方法，是这个方法把它带入的 ，详解：[React.Children与React.cloneElement杂谈](https://juejin.cn/post/6844903983975235592)

[React.Children详解](https://www.cnblogs.com/chen-cong/p/10371329.html)

[React.cloneElement()给子组件传值](https://www.shuzhiduo.com/A/pRdBK9P2zn/)
[修改CHILDREN(](https://www.freesion.com/article/2673922398/)


在seek-areasure-home.js 的  window.getLoginInfo 调用的这个方法是 enterprise-chage.js 中的 _getLoginInfo 方法吗？


- [ ] 在【新增企业】中点开【地区】是在组件FilterAreap【filter-area.js】文件中，其地区文件时存在一个【area.js】文件中。





<br/>

***
<br/>


># <h1 id="属性">属性</h1>

<br/>


> <h2 id="clientWidth">**.clientWidth**</h2>


&emsp; 内联元素以及没有 CSS 样式的元素的 clientWidth 属性值为 0。Element.clientWidth 属性表示元素的内部宽度，以像素计。该属性包括内边距 padding，但不包括边框 border、外边距 margin 和垂直滚动条（如果有的话）。


![图解](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_3.png)


<br/>


> <h2 id="JavaScript的childNodes和children">**JavaScript的childNodes和children**</h2>


&emsp; childNodes用来获取一个元素的所有子元素，这个包括元素节点和文本节点。

&emsp; children用来获取一个元素的子元素节点，注意只是元素节点

- 其中DOM中常见的三种节点分别如下：

	- 元素节点：<body>，<p>，<a>，<div>，<head>.....等等这些标签，都是元素节点

	- 属性节点：title，value，href，id，class等等这些标签的属性，都是属性节点

	- 文本节点：文本节点是包含在在标签之内的内容（双标签）比如`<p>this is text node</p>`，中间的文字就是文本节点；注意包含在标签中间的不一定是文本节点，如`<div><p></p></div>`当中的<p>是元素节点

　　&emsp; 所以在使用childNodes来获取到一个元素的子元素集合，这是一个数组。

　　&emsp; 其中最常用的是通过childNodes获得的对象数组中，第一个子元素对象（也即下标为0），这个元素是text，即文本节点，也可以使用firstChild来替代childNodes[0]，然后通过nodeValue属性来获取text的值。

　　&emsp; 如果在标签之中只有一个文本节点，而没有其他元素节点时，也可以使用innerHTML来获取这个节点的文本内容。
　　
```
	<body>
		<div id="test">
			this is test
			<p>this is other test</p>
		</div>
	</body>
	<script>
		var div = document.getElementById("test");
		
		alert(div.childNodes.length);        //显示3
		
		alert(div.childNodes[0]);   //[object Text]
		
		alert(div.childNodes[0].nodeValue);   //显示this is test
		//等价
		alert(div.firstChild.nodeValue);      //显示this is test
		
		alert(div.childNodes[1]);   //[object HTMLParagraphElement]
		
		alert(div.childNodes[1].childNodes[0].nodeValue); //this is other test
		//等价   
		alert(div.childNodes[1].firstChild.nodeValue); //this is other test
		//p标签中无其他节点，可以使用innderHTML
		alert(div.childNodes[1].innerHTML);  ////this is other test
	
	</script>
　　
```



<br/>

> <h2 id="innerWidth">.innerWidth</h1>


只读的 Window 属性 innerWidth 返回以像素为单位的窗口的内部宽度。如果垂直滚动条存在，则这个属性将包括它的宽度。

更确切地说，innerWidth 返回窗口的 layout viewport (en-US) 的宽度。 窗口的内部高度——布局视口的高度——可以从 innerHeight 属性中获取到。



<br/>

> <h2 id="setTimeout">setTimeout</h2>


```

<body>

	<p>点击按钮，等待 3 秒后弹出 "Hello" 。</p>
	<p>点击第二个按钮来阻止弹出函数 myFunction 的执行。 (你必须在 3 秒前点击)</p>
	
	<button onclick="myFunction()">先点我</button>
	<button onclick="myStopFunction()">阻止弹出</button>
	
	<script>
		var myVar;
		
		function myFunction() {
		    myVar = setTimeout(function(){ alert("Hello") }, 3000);
		}
		
		function myStopFunction() {
		    clearTimeout(myVar);
		}
	</script>

</body>

```


![弹框延迟](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_4.png)

- clearTimeout() 方法可取消由 setTimeout() 方法设置的定时操作。
- clearTimeout() 方法的参数必须是由 setTimeout() 返回的 ID 值。

注意: 要使用 clearTimeout() 方法, 在创建执行定时操作时要使用全局变量：



<br/>

> <h2 id=""></h1>



<br/>

> <h2 id=""></h1>


<br/>

***
<br/>


># <h1 id="问题">问题</h1>

- ReceivedListButtonManager.post(ReceivedListButtonManager.EVENT_TYPE.EXPAND) 这个是有什么作用？push方法的值是什么？

<br/>

***
<br/>


># <h1 id="开发心得">开发心得</h1>

<br/>

> <h2 id="flex心得">flex心得</h2>

- 没有使用display: flex效果

```
test.css
.div1 {
    margin-top: 20px;
    width: 100%;
    height: 10%;
    background-color: red;
}


.div2 {
    background-color: yellow;
}

//test.js 文件
_renderIndustryTemplate = () => {
	return <div className='div1'>
				    <div className='div2'>
					    search-custom-template-industry-categorysearch-custom-template-industry-category
				    </div>
		   </div>
}

```

效果图：

![父组件没有使用display: flex效果](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_0.png)

没有使用display: flex情况下div是块级元素，独占一行。


<br/>


- **flex 布局对其自身影响**

```

//test.css

.div1 {
    display: flex;
    background-color: red;
}
.div2 {
    background-color: yellow;
}


//test.js 文件
//企业模板
_test1 = () => {
    return <div className='div1'>
        <div className='div2'>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
		        search-custom-template-industry-category
		        <br/>
        </div>

    </div>
}


```

效果：![自身使用display: flex效果](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_2.png)

- 其内容是被子组件填充的，其高度是随着子组件的增加而增加，但是其宽度可以达到100%;
- 若是没有子组件，也没有设其宽度、高度，则它的内容是看不到的，需要设置其宽、高的；



<br/>


- flex布局对其子组件影响

```
test.css
.div1 {
    margin-top: 30px;
    display: flex;
    width: 100%;
    height: 10%;
    background-color: red;
}


.div2 {
    background-color: yellow;
}

//test.js 文件
_test1 = () => {
	return <div className='div1'>
				    <div className='div2'>
					    search-custom-template-industry-categorysearch-custom-template-industry-category
				    </div>
		   </div>
}

```


![父组件使用display: flex效果](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_1.png)


结论：当父组件使用了 flex后，块级元素的宽度是随着内容的扩充而扩充，但他的高度相当于父级组件是100%充满的。

&emsp; 我当时也测试了，块级元素的父组件没有使用`display: flex`可以独占一行可以设置设置其宽度、高度。

&emsp; 当块级元素的父组件使用`display: flex`后页可以设置设置其宽度、高度。


<br/>
<br/>

># <h2 id="值为空">值为空</h2>

- 数组元素不存在

```
let array = [1, 2, 3, 4]

let arr5 = array[5]
if(arr5 === null ){
	console.log('第5个元素不存在')
}
//判断arr5其为null是不对的，因为arr5不存在相当于没有定义，所以是undefined,所以正确的判断是：

if(arr5 === null  || arr){
	console.log('第5个元素不存在')
}

```



<br/>

***
<br/>


># <h1 id=""></h1>



<br/>

***
<br/>


># <h1 id=""></h1>



<br/>

***
<br/>


># <h1 id=""></h1>



<br/>

***
<br/>


># <h1 id=""></h1>



<br/>

***
<br/>


># <h1 id=""></h1>




<br/>

***
<br/>



