> <h1 id=""></h1>
- [**模块导入和导出**](#模块导入和导出)
	- [require](#require)
- [ES6语法](#ES6语法)
	- [=> 箭头函数](#箭头函数)
	- [匿名函数](#匿名函数)
		- [自调用函数](#自调用函数)
- [**React**](#React)
	- [ReactDom](#ReactDom)
		- [unmountComponentAtNode()](#unmountComponentAtNode())  
- [**标准库**](#标准库)
	- [Math.max()](#Mathmax)
	- [Object.keys()](#Objectkeys)
	- [Object.assign()](#Object.assign) 
- [**属性**](#属性)
	- [clientWidth](#clientWidth)
	- [JavaScript的childNodes和children](#JavaScript的childNodes和children)
	- [.innerWidth](#innerWidth)
	- [setTimeout](#setTimeout)
- [**Window**](#Window)
	- [Window.sessionStorage](#sessionStorage)
	- [Window Navigator](#WindowNavigator)
- [**Element**](#Element)
	- [Element.getBoundingClientRect()](#getBoundingClientRect)
- [**Dom对象**](#Dom对象)
	- [Dom元素创建设值(createElement)](#Dom元素创建设值)
- [**Event用法**](#Event用法)
	- [stopPropagation()](#stopPropagation())
	- [preventDefault()](#preventDefault())
	- [initEvent()](#initEvent())
- [**EventTarget**](#EventTarget) 
	- [addEventListener()](#addEventListener)
- [**TouchEvent**](#TouchEvent) 
	- [属性](#属性)
		- [touches](#touches)  
- [**Touch**](#Touch) 
	- [属性](#属性) 
		- [screenX](#screenX)
- [**JSON**](#JSON)
	- [JSON.parse()](#JSON.parse())
	- [JSON.stringify()](#JSON.stringify())
- [**问题**](#问题)
- [**开发心得**](#开发心得)
	- [flex使用](#flex心得)
	- [值为空](#值为空)
- [**配置**](#配置)
	-  [打包](#打包)
-  [代码解读](#代码解读)
	- [tabs滚动](#tabs滚动)  





<br/>

***
<br/>

># <h1 id="模块导入和导出"> [模块导入和导出](https://www.cnblogs.com/libin-1/p/7127481.html) </h1>


<br/>


> <h2 id="require"> require </h2>

&emsp; 在CommonJS中，有一个全局性方法require()，用于加载模块。假定有一个数学模块math.js，就可以像下面这样加载。

|:--|:--|
| 1 | var math = require('math'); |


然后，就可以调用模块提供的方法：

| 1 | var math = require('math'); |
|:--|:--|
| 2 | math.add(2,3); // 5 |





<br/>

***
<br/>


># <h1 id="ES6语法">ES6语法</h1>

<br/>

># <h2 id="箭头函数">[=> 箭头函数](https://juejin.cn/post/6844903573428371464)</h2>

> [**基础语法**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

```
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
//相当于：(param1, param2, …, paramN) =>{ return expression; }

// 当只有一个参数时，圆括号是可选的：
(singleParam) => { statements }
singleParam => { statements }

// 没有参数的函数应该写成一对圆括号。
() => { statements }
```

<br/>

>- **正常语法：**

```
const elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

let array = elements.map(function(element) {
  return element.length;
}); 
console.log(array);

```

打印：

```
> Array [8, 6, 7, 9]
```

<br/>

> **正常语法 改写1**

```
// 上面的普通函数可以改写成如下的箭头函数
const elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

let array = elements.map((element) => {
  return element.length;
}); 
console.log(array);
```

打印：

```
> Array [8, 6, 7, 9]
```

<br/>


> **正常语法 改写2**

```
const elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

// 当箭头函数只有一个参数时，可以省略参数的圆括号
let array = elements.map(element => {
 return element.length;
}); 

console.log(array);
```

打印：

```
> Array [8, 6, 7, 9]
```

<br/>

> **正常语法 改写3**

```
// 上面的普通函数可以改写成如下的箭头函数
const elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

// 当箭头函数的函数体只有一个 `return` 语句时，可以省略 `return` 关键字和方法体的花括号
let array = elements.map(element => element.length);
console.log(array);
```

打印：

```
> Array [8, 6, 7, 9]
```

<br/>

> **正常语法 改写4**

```
// 上面的普通函数可以改写成如下的箭头函数
const elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

// 在这个例子中，因为我们只需要 `length` 属性，所以可以使用参数解构
// 需要注意的是字符串 `"length"` 是我们想要获得的属性的名称，而 `lengthFooBArX` 则只是个变量名，
// 可以替换成任意合法的变量名
let array = elements.map(({ "length": lengthFooBArX }) => lengthFooBArX); // [8, 6, 7, 9]
console.log(array);
```

打印：

```
> Array [8, 6, 7, 9]
```



<br/>
<br/>


> <h2 id="匿名函数"> 匿名函数 </h2>

<br/>

> <h3 id="自调用函数"> 自调用函数 </h3>

<br/>

> <h3 id="window.mapi"> window.mapi </h3>


```
(function (window, api) {
　　var num = 1;

　　function a() {
　　　　console.log(num);
　　}

　　api.sum = function (x, y) {
　　　　console.log("--->> %i", x+y);
　　}
  
  api.version = "1.0.0";
            
})(window, window['api'] || (window['api'] = {}));


window.api.sum(99, 1)

console.log("++++ %s", window.api.version)


```

打印：

```
> "--->> %i" 100
> "++++ %s" "1.0.0"
```

这个`windown['api']`相当于 `window.api`

而 `window['api'] = {}` 是一个对象，而在代码中：

```
function (x, y) {
	console.log("--->> %i", x+y);
}
```

相当于是一个函数对象，所以可以穿入。


衍生下：

```
//生成一个对象
var person = {firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"};

//访问对象属性
//你可以通过两种方式访问对象属性:
//实例 1
person.lastName;



//或者实例 2
//person["lastName"];
```

<br/>

**与其比较：**

```
(function (n1,n2){

console.log("这是匿名函数的自执行的第二种写法，结果为："+(n1+n2))

}(10,100))//110
```

打印：

```
> "这是匿名函数的自执行的第二种写法，结果为：110"
```



<br/>


```
//=====================================================

(function (win, mapi) {
    mapi.registerResponseInterceptor = function (func) {
        mapi['responseInterceptor'] = func;
    }
})(window, window['mapi'] || (window['mapi'] = {}));

window.mapi.registerResponseInterceptor((res, request)=>{
    return res + request;
});



// let aa = window.mapi.registerResponseInterceptor(1, 99);

console.log("--------_>> %i", window.mapi.responseInterceptor(1,99));
//或者下面
//console.log("--------_>> %i", window.mapi["responseInterceptor"](1,99));
```

打印：

```
--------_>> 100
```



<br/>
<br/>








<br/>

***
<br/>


># <h1 id="React">React</h1>


># <h2 id="ReactDOM">ReactDOM</h2>

<br/>


> <h3 id="unmountComponentAtNode()">unmountComponentAtNode()</h3>

```
ReactDOM.unmountComponentAtNode(container)
```

&emsp; 从 DOM 中卸载组件，会将其事件处理器（event handlers）和 state 一并清除。如果指定容器上没有对应已挂载的组件，这个函数什么也不会做。如果组件被移除将会返回 true，如果没有组件可被移除将会返回 false。


> <h2 id=""></h2>


> <h2 id=""></h2>




<br/>

***
<br/>


># <h1 id="标准库">标准库</h1>


<br/>

> <h2 id="Mathmax">Math.max()</h2>

Math.max() 函数返回一组数中的最大值。


```
console.log(Math.max(1, 3, 2));
// expected output: 3
```

&emsp; 返回给定的一组数字中的最大值。如果给定的参数中至少有一个参数无法被转换成数字，则会返回 NaN。

<br/>

> <h2 id="Objectkeys">Object.keys()</h2>

&emsp; Object.keys() 方法会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致 。

```
//一个表示给定对象的所有可枚举属性的字符串数组。
Object.keys(obj)



// simple array
var arr = ['a', 'b', 'c'];
console.log(Object.keys(arr)); // console: ['0', '1', '2']

// array like object
var obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.keys(obj)); // console: ['0', '1', '2']

// array like object with random key ordering
var anObj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.keys(anObj)); // console: ['2', '7', '100']

// getFoo is a property which isn't enumerable
var myObj = Object.create({}, {
  getFoo: {
    value: function () { return this.foo; }
  }
});
myObj.foo = 1;
console.log(Object.keys(myObj)); // console: ['foo']
```


<br/>

> <h2 id="Object.assign">Object.assign() </h2>

&emsp; **Object.assign()** 方法用于将所有可枚举属性的值从一个或多个源对象source复制到目标对象。它将返回目标对象target。

```
const target = { a: 1, b: 2 }
const source = { b: 4, c: 5 }

const returnedTarget = Object.assign(target, source)

target // { a: 1, b: 4, c: 5 }
returnedTarget // { a: 1, b: 4, c: 5 }

```

&emsp; Object.assign方法的第一个参数是目标对象，后面的参数都是源对象。

&emsp; **注意:**如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。


<br/>

> <h2 id=""></h2>

<br/>

> <h2 id=""></h2>






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


># <h1 id="Window">Window</h1>

<br/>

> <h2 id="sessionStorage">Window.sessionStorage</h1>

&emsp; `sessionStorage` 属性允许你访问一个，对应当前源的 session Storage 对象。它与 localStorage 相似，不同之处在于 localStorage 里面存储的数据没有过期时间设置，而存储在 sessionStorage 里面的数据在页面会话结束时会被清除。

```
// 保存数据到 sessionStorage
sessionStorage.setItem('key', 'value');

// 从 sessionStorage 获取数据
let data = sessionStorage.getItem('key');

// 从 sessionStorage 删除保存的数据
sessionStorage.removeItem('key');

// 从 sessionStorage 删除所有保存的数据
sessionStorage.clear();
```


<br/>
- **返回值**

一个 **Storage** 对象。

<br/>

- **示例**

下面的代码访问当前域名的 session Storage 对象，并使用 Storage.setItem() 访问往里面添加一个数据条目。

```
sessionStorage.setItem('myCat', 'Tom');
Copy to Clipboard
```

<br/>



&emsp; 下面的示例会自动保存一个文本输入框的内容，如果浏览器因偶然因素被刷新了，文本输入框里面的内容会被恢复，因此写入的内容不会丢失。

```
// 获取文本输入框
let field = document.getElementById("field");

// 检测是否存在 autosave 键值
// (这个会在页面偶然被刷新的情况下存在)
if (sessionStorage.getItem("autosave")) {
  // 恢复文本输入框的内容
  field.value = sessionStorage.getItem("autosave");
}

// 监听文本输入框的 change 事件
field.addEventListener("change", function() {
  // 保存结果到 sessionStorage 对象中
  sessionStorage.setItem("autosave", field.value);
});
```

<br/>
<br/>

> <h2 id="WindowNavigator"> Window Navigator </h2>

**window.navigator 对象包含有关访问者浏览器的信息**

![navigator信息](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react6.png)

**警告!!!**

- 来自 navigator 对象的信息具有误导性，不应该被用于检测浏览器版本，这是因为：
- navigator 数据可被浏览器使用者更改
- 一些浏览器对测试站点会识别错误
- 浏览器无法报告晚于浏览器发布的新操作系统



<br/>
<br/>

> <h2 id=""></h2>



<br/>
<br/>

> <h2 id=""></h2>








<br/>

***
<br/>


># <h1 id="Dom对象">Dom对象</h1>


<br/>

> <h2 id="Dom元素创建设值">Dom元素创建设值</h2>

```
css文件
.abc {
	background: red;
}


Js文件

//通过指定名称创建一个元素
var div1=document.createElement("div");

//设置或者改变指定属性并指定值。
//tabindex:tabindex 属性 指示其元素是否可以聚焦，以及它是否/在何处着手键盘导航（使用Tab因此键，得名）。参考资料：https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/tabindex
div1.setAttribute('tabindex', '-1');

//设置样式
div1.className = 'abc'
//设置焦点
div1..focus();
```







<br/>

> <h2 id=""></h1>





<br/>

> <h2 id=""></h1>




<br/>

***
<br/>


># <h1 id="Element">Element</h1>



<br/>

> <h2 id="getBoundingClientRect">Element.getBoundingClientRect()</h2>

&emsp; Element.getBoundingClientRect() 方法返回元素的大小及其相对于视口的位置。

&emsp; 如果是标准盒子模型，元素的尺寸等于width/height + padding + border-width的总和。如果box-sizing: border-box，元素的的尺寸等于 width/height。

![位置显示](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react27.png)



<br/>

> <h2 id=""></h2>



<br/>

***
<br/>


># <h1 id="Event用法">Event用法</h1>

- **Event 对象**
	- 代表事件的状态，比如事件在其中发生的元素、键盘按键的状态、鼠标的位置、鼠标按钮的状态。
	- 事件通常与函数结合使用，函数不会在事件发生前被执行


<br/>

> <h2 id="stopPropagation()">stopPropagation()</h2> 

事件句柄　(Event Handlers)
比如当用户点击某个 HTML 元素时启动一段 JavaScript

```
<Menu>
	<Menu.Item>                
	 	<a onClick={e => { e.stopPropagation(); this.add(0) }}>添加模块</a>              </Menu.Item>              
	<Menu.Item>                
		<a onClick={e => { e.stopPropagation(); this.add(1) }}>添加用例</a>              </Menu.Item>            
</Menu>
```
onClick就是一个事件，后面定义这个事件的行为

**下面来看stopPropagation() 方法的定义：**

&emsp; 不再派发事件。终止事件在传播过程的捕获、目标处理或起泡阶段进一步传播。调用该方法后，该节点上处理该事件的处理程序将被调用，事件不再被分派到其他节点
语法（如下）：

```
event.stopPropagation()
```

&emsp; 也就是说该方法会停止事件的传播，阻止它被分派到其他Document 节点。

&emsp; 工程中的的框架是react，在调用这个方法时`componentWillReceiveProps()`遇到了问题是，我不想在触发某一个事件（比如上面代码的onClick）后，把props的数据更新，就可以使用stopPropagation方法阻止数据的分派。


<br/>

> <h2 id="preventDefault()">preventDefault()</h2> 


```
event.preventDefault()
```
&emsp; 通知 Web 浏览器不要执行与事件关联的默认动作（如果存在这样的动作）
例如，如果 type 属性是 “submit”，在事件传播的任意阶段可以调用任意的事件句柄，通过调用该方法，可以阻止提交表单。注意，如果 Event 对象的 cancelable 属性是 fasle，那么就没有默认动作，或者不能阻止默认动作。无论哪种情况，调用该方法都没有作用。


<br/>

> <h2 id="initEvent()">initEvent()</h2> 

```
event.initEvent(eventType,canBubble,cancelable)
```
&emsp; 该方法将初始化 `Document.createEvent()` 方法创建的合成 Event 对象的 type 属性、bubbles 属性和 cancelable 属性。只有在新创建的 Event 对象被 Document 对象或 Element 对象的 dispatchEvent() 方法分派之前，才能调用 Event.initEvent() 方法

```
// 创建事件.
var event = document.createEvent('Event');

// 初始化一个点击事件，可以冒泡，无法被取消
event.initEvent('click', true, false);
```
但是这个方法已经被废弃了，新的方法是：

```
event = new Event(typeArg, eventInit);
eg:

// 创建一个支持冒泡且不能被取消的look事件

var e = new Event("look", {"bubbles":true, "cancelable":false});
document.dispatchEvent(ev);

// 事件可以在任何元素触发，不仅仅是document
myDiv.dispatchEvent(ev);
```





<br/>

***
<br/>


># <h1 id="EventTarget">[EventTarget](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)</h1>


<br/>


> <h2 id="addEventListener">addEventListener()</h2>

&emsp; EventTarget.addEventListener() 方法将指定的监听器注册到 EventTarget 上，当该对象触发指定的事件时，指定的回调函数就会被执行。 事件目标可以是一个文档上的元素 Element,Document和Window或者任何其他支持事件的对象 (比如 XMLHttpRequest)。

&emsp; addEventListener()的工作原理是将实现EventListener的函数或对象添加到调用它的EventTarget上的指定事件类型的事件侦听器列表中。

```
target.addEventListener(type, listener, options);
target.addEventListener(type, listener, useCapture);
target.addEventListener(type, listener, useCapture, wantsUntrusted );  // Gecko/Mozilla only
```


- **参数**
- type
	- 表示监听事件类型的字符串。

- listener
	- 当所监听的事件类型触发时，会接收到一个事件通知（实现了 Event 接口的对象）对象。listener 必须是一个实现了 EventListener 接口的对象，或者是一个函数。有关回调本身的详细信息，请参阅The event listener callback 

- options 可选
	- 	一个指定有关 listener 属性的可选参数对象。可用的选项如下：
	- 	capture:  Boolean，表示 listener 会在该类型的事件捕获阶段传播到该 EventTarget 时触发。
	- 	once:  Boolean，表示 listener 在添加之后最多只调用一次。如果是 true， listener 会在其被调用之后自动移除。
	- 	passive: Boolean，设置为true时，表示 listener 永远不会调用 preventDefault()。如果 listener 仍然调用了这个函数，客户端将会忽略它并抛出一个控制台警告。查看 使用 passive 改善的滚屏性能 了解更多.
	- 	signal：AbortSignal，该 AbortSignal 的 abort() 方法被调用时，监听器会被移除。
	-  mozSystemGroup: 只能在 XBL 或者是 Firefox' chrome 使用，这是个 Boolean，表示 listener 被添加到 system group。

- useCapture  可选
	- Boolean，在DOM树中，注册了listener的元素， 是否要先于它下面的EventTarget，调用该listener。 当useCapture(设为true) 时，沿着DOM树向上冒泡的事件，不会触发listener。当一个元素嵌套了另一个元素，并且两个元素都对同一事件注册了一个处理函数时，所发生的事件冒泡和事件捕获是两种不同的事件传播方式。事件传播模式决定了元素以哪个顺序接收事件。进一步的解释可以查看 事件流 及 JavaScript Event order 文档。 如果没有指定， useCapture 默认为 false 。 

- wantsUntrusted 
- 	如果为 true , 则事件处理程序会接收网页自定义的事件。此参数只适用于 Gecko（chrome的默认值为true，其他常规网页的默认值为false），主要用于附加组件的代码和浏览器本身。





<br/>

***
<br/>


># <h1 id="TouchEvent">[TouchEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/TouchEvent)</h1>



> <h2 id="属性">属性</h2>


> <h3 id="touches">touches</h3>

&emsp; 一个 [TouchList](https://developer.mozilla.org/zh-CN/docs/Web/API/TouchList)，其会列出所有当前在与触摸表面接触的  Touch 对象，不管触摸点是否已经改变或其目标元素是在处于 touchstart 阶段。

```
var touches = touchEvent.touches;
```

**返回值**

&emsp; **touches:** 一个 TouchList，其会列出所有当前在与触摸表面接触的  Touch 对象，不管触摸点是否已经改变或其目标元素是在处于 touchstart 阶段。


&emsp; TouchEvent.touches 属性是一个 TouchList 对象，并包含 Touch 当前接触表面的每个接触点的对象列表。

&emsp; 事件处理程序会检查 TouchEvent.touches 列表的长度，以确定激活的触摸点的数量，然后根据触摸点的数量调用不同的处理程序。

```
someElement.addEventListener('touchstart', function(e) {
   // Invoke the appropriate handler depending on the
   // number of touch points.
   switch (e.touches.length) {
     case 1: handle_one_touch(e); break;
     case 2: handle_two_touches(e); break;
     case 3: handle_three_touches(e); break;
     default: console.log("Not supported"); break;
   }
 }, false);

```




<br/>

***
<br/>


># <h1 id="Touch">[Touch](https://developer.mozilla.org/zh-CN/docs/Web/API/Touch/Touch)</h1>


>## <h2 id="属性">属性</h2>


>### <h3 id="screenX">screenX</h3>

&emsp; 返回触点相对于屏幕左边沿的的X坐标. 不包含页面滚动的偏移量.

```
var x = touchItem.screenX;
```






>### <h3 id=""></h3>



<br/>

***
<br/>


># <h1 id="JSON.stringify()"> [JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) </h1>

&emsp; **JSON.stringify()** 方法将一个 JavaScript 对象或值转换为 JSON 字符串，如果指定了一个 replacer 函数，则可以选择性地替换值，或者指定的 replacer 是数组，则可选择性地仅包含数组指定的属性。


```
console.log(JSON.stringify({ x: 5, y: 6 }));
// expected output: "{"x":5,"y":6}"

console.log(JSON.stringify([new Number(3), new String('false'), new Boolean(false)]));
// expected output: "[3,"false",false]"

console.log(JSON.stringify({ x: [10, undefined, function(){}, Symbol('')] }));
// expected output: "{"x":[10,null,null,null]}"

console.log(JSON.stringify(new Date(2006, 0, 2, 15, 4, 5)));
// expected output: ""2006-01-02T15:04:05.000Z""
```

<br/>

**语法**

```
JSON.stringify(value[, replacer [, space]])
```

- **参数**


	- **value**

	```
	将要序列化成 一个 JSON 字符串的值。
	```

	- **replacer 可选**

	```
	如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；如果该参数为 null 或者未提供，则对象所有的属性都会被序列化。
	
	```

	- **space 可选**

	```
	
	指定缩进用的空白字符串，用于美化输出（pretty-print）；如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；如果该参数没有提供（或者为 null），将没有空格。
	返回值
	一个表示给定值的JSON字符串。
	```
	


<br/>

***
<br/>

> <h1 id="JSON.parse()"> JSON.parse() </h1>

&emsp; **JSON.parse()** 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

```
const json = '{"result":true, "count":42}';
const obj = JSON.parse(json);

console.log(obj.count);
// expected output: 42

console.log(obj.result);
// expected output: true

```

打印：

```
> 42
> true
```


<br/>

**语法**

```
JSON.parse(text[, reviver])
```


<br/>

参数

- text

`要被解析成 JavaScript 值的字符串，关于JSON的语法格式,请参考：JSON。`

- reviver 可选

`转换器, 如果传入该参数(函数)，可以用来修改解析生成的原始值，调用时机在 parse 函数返回之前。`

- 返回值

`Object 类型, 对应给定 JSON 文本的对象/值。`







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


># <h1 id="配置">配置</h1>

- <h2 id="打包">打包</h2>
	- Mac打包

```
npm run pkgv2 treasure

//qa环境的包
//upload=后面的参数是是否自动上传包
//若是dev环境，把qa改为dev就好了
npm run pkgv2 treasure -- -f -qa -upload=true
```

- Window打包

```
yarn  pkgv2 treasure
```

打包的文件是：

![打包的文件](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react8.png)



<br/>

***
<br/>


># <h1 id="代码解读">代码解读</h1>

<br/>

> <h2 id="tabs滚动">tabs滚动</h2>

顶部tabs数据： 

```
//完整一屏幕数据
* let categories = [{ categoryName: '银行1234', categoryType: '1234567890' }, { categoryName: '大厦', categoryType: '1234567890' }, { categoryName: '王者荣耀', categoryType: '1234567890' }, { categoryName: '发现亚东', categoryType: '1234567890' }, { categoryName: '发现了我', categoryType: '1234567890' }, 
//未完整一屏幕数据
* { categoryName: '放几天', categoryType: '1234567890' }, { categoryName: '你从我房', categoryType: '1234567890' }, { categoryName: '老师那个', categoryType: '1234567890' },]
```     

```
_startScroll = (index) => {
    let { tabLayoutContentDiv, hsTabLayoutScroll } = this.refs
    let tabLayoutContentWidth = tabLayoutContentDiv.clientWidth
    let currentTab = tabLayoutContentDiv.childNodes[index];

    //中心点左右边距
    let centerMargin = window.innerWidth / 2 - currentTab.clientWidth / 2;

    console.log('🍎  <<<<<<<<<', '\n 显示窗口宽度window.innerWidth：', window.innerWidth, '\n 其显示窗口宽度1/2: ', window.innerWidth / 2,
        '\n 标签父组件Div的宽度tabLayoutContentWidth:', tabLayoutContentWidth,
        '\n 滚动到第index: ', index, '标签', ' \n左边距离是currentTab.offsetLeft：', currentTab.offsetLeft, '\n宽度为currentTab.clientWidth：', currentTab.clientWidth, ' 它的宽度1/2为：', (currentTab.clientWidth / 2),
        '\n 中心点减去第', index, '标签宽度后距离左边的距离centerMargin：', centerMargin,
    )
    //左边内容
    //这是在第一个屏幕中的判断，主要判断是否在第一屏幕的中间位置需不需要滑动，是左边的内容
    //currentTab.offsetLeft <= centerMargin： 判断选中的tab左边的距离和距离中心减去tab 1/2宽度 的距离值
    //tabLayoutContentWidth <= window.innerWidth 这时判断tab父组件的宽度(其实也是所有tab的宽度)和 中心宽度减去选中tab的1/2宽度值
    if (currentTab.offsetLeft <= centerMargin || tabLayoutContentWidth <= window.innerWidth) {
        console.log('🍎 ======= \n 左边内容(-this.scrollOffset)： ', (-this.scrollOffset))

        hsTabLayoutScroll._scrollTo(-this.scrollOffset, 250)
    } else {//这是判断滑动以后右边的内容
        console.log('🍎 >>>>>>> 右边内容 \n     <<<<<<<<<< 判断: ', tabLayoutContentWidth - currentTab.offsetLeft - currentTab.clientWidth > centerMargin,
            '\n 标签父组件Div的宽度tabLayoutContentWidth: ', tabLayoutContentWidth,
            '\n 选中标签左边距离currentTab.offsetLeft', currentTab.offsetLeft,
            '\n 选中标签宽度currentTab.clientWidth: ', currentTab.clientWidth,
            '\n 中心点减去选中标签1/2centerMargin: ', centerMargin)

		//判断父组件宽度减去（选中tab的左边距离+选中tab的宽度）和 中线点的比较。
		//这里的判断若是true说明父组件减去（选中tab的左边距离+选中tab的宽度）距离父组件最右边大于显示屏幕的宽度，所以可以继续左滑动
		//若是false说明距离最右边的距离不足能显示屏幕的一半了，需要做补充差值忘左滑动，这时要注意滑到最右边时不能再滑了
        if (tabLayoutContentWidth - currentTab.offsetLeft - currentTab.clientWidth > centerMargin) {            let currentTabX = currentTab.getBoundingClientRect().x;
            console.log('🍎 =======  右边内容 true判断：', '\n Math.abs(currentTabX): ', Math.abs(currentTabX), '\n centerMargin: ', centerMargin,
                '\n (Math.abs(currentTabX) > centerMargin): ', (Math.abs(currentTabX) > centerMargin))
                
			//选中tab的x距离中心点的差值判断，进行向左滑动，Math.abs()是绝对值
            if (Math.abs(currentTabX) > centerMargin) {//这里是不足一屏幕时进行的滑动
                hsTabLayoutScroll._scrollTo(-(Math.abs(currentTabX) - centerMargin), 250);
            } else if (Math.abs(currentTabX) < centerMargin) {//这里是超出屏幕以后进行的滑动
                hsTabLayoutScroll._scrollTo(centerMargin - Math.abs(currentTabX), 250);
            }
        } else {
            //右边
            //这里快滑到最右边了，做差值滑动
            let remainWidth = tabLayoutContentWidth - (-this.scrollOffset) - window.innerWidth;
            console.log('🍎 >>>>>>>  右边内容 false判断：', '\n 父组件Div的宽度tabLayoutContentWidth: ', tabLayoutContentWidth, '\n (-this.scrollOffset): ', (-this.scrollOffset),
                '\n 显示窗口宽度window.innerWidth:', window.innerWidth,
                '\n -remainWidth: ', -remainWidth)

            hsTabLayoutScroll._scrollTo(-remainWidth, 250)
        }

    }
}
```



![效果显示](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/js_5.png)

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



