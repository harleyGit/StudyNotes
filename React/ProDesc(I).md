> <h2 id=""></h2>
- [**=> 箭头函数**](#箭头函数)
- [window.sessionStorage](#window.sessionStorage)
- [JSON.stringify()](#JSON.stringify())
- [匿名函数](#匿名函数)
	- [自调用函数](#自调用函数)
		- [window.mapi](#window.mapi)
- [Window](#Window)
	- [Window Navigator](#WindowNavigator)
	- [JSON.parse()](#JSON.parse())



<br/>

***
<br/>


># <h1 id="箭头函数">[=> 箭头函数](https://juejin.cn/post/6844903573428371464)</h1>

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

***
<br/>


> <h1 id="window.sessionStorage"> window.sessionStorage </h1>

&emsp; sessionStorage 属性允许你访问一个，对应当前源的 session Storage 对象。它与 localStorage 相似，不同之处在于 localStorage 里面存储的数据没有过期时间设置，而存储在 sessionStorage 里面的数据在页面会话结束时会被清除。

- 页面会话在浏览器打开期间一直保持，并且重新加载或恢复页面仍会保持原来的页面会话。
- 在新标签或窗口打开一个页面时会复制顶级浏览会话的上下文作为新会话的上下文，这点和 session cookies 的运行方式不同。
- 打开多个相同的URL的Tabs页面，会创建各自的sessionStorage。
- 关闭对应浏览器窗口（Window）/ tab，会清除对应的sessionStorage。 

```
应该注意: 存储在sessionStorage或localStorage中的数据特定于页面的协议。也就
是说http://example.com 与 https://example.com的sessionStorage相互隔离。

被存储的键值对总是以UTF-16 DOMString 的格式所存储，其使用两个字节来表示一个字符。对于对象、整数key值会自动转换成字符串形式。
```

<br/>

- **语法**

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

下面的示例会自动保存一个文本输入框的内容，如果浏览器因偶然因素被刷新了，文本输入框里面的内容会被恢复，因此写入的内容不会丢失。

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

> <h1 id="匿名函数"> 匿名函数 </h1>



> <h2 id="自调用函数"> 自调用函数 </h2>



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

***
<br/>


> <h1 id="Window"> Window </h1>

> <h2 id="WindowNavigator"> Window Navigator </h2>

**window.navigator 对象包含有关访问者浏览器的信息**

![navigator信息](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react6.png)

**警告!!!**

- 来自 navigator 对象的信息具有误导性，不应该被用于检测浏览器版本，这是因为：
- navigator 数据可被浏览器使用者更改
- 一些浏览器对测试站点会识别错误
- 浏览器无法报告晚于浏览器发布的新操作系统




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


> <h1 id=""> </h1>


<br/>

***
<br/>


> <h1 id=""> </h1>



<br/>

***
<br/>


> <h1 id=""> </h1>


<br/>

***
<br/>


> <h1 id=""> </h1>


<br/>

***
<br/>


> <h1 id=""> </h1>


<br/>

***
<br/>


> <h1 id=""> </h1>



<br/>

***
<br/>


> <h1 id=""> </h1>



<br/>

***
<br/>


> <h1 id=""> </h1>




<br/>

***
<br/>


> <h1 id=""> </h1>