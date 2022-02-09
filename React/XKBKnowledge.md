> <h1 id=""></h1>
- [**探索脚手架create-react-app原理**](https://juejin.cn/post/6844903604583661582)
- [ES6语法](#ES6语法)
	- [=> 箭头函数](#箭头函数)
	- [匿名函数](#匿名函数)
		- [自调用函数](#自调用函数)
	- [对象](#对象)
		- [Object.setPrototypeOf()](#Object.setPrototypeOf())
	- [Generator](#Generator)
		- [作为对象属性的Generator函数](#作为对象属性的Generator函数)
	- [Promise对象](#Promise对象)
	- [Module模块](#Module模块)
		- [require和import](#require和import)
			- [export用法](#export用法)
			- [import用法](#import用法)
			- [require用法](#require用法)
- [**React**](#React)
	- [ReactDom](#ReactDom)
		- [unmountComponentAtNode()](#unmountComponentAtNode())  
- [**标准库**](#标准库)
	- [Math.max()](#Mathmax)
	- [Object.keys()](#Objectkeys)
	- [Object.assign()](#Object.assign) 
- [**三方库**](#三方库)
	- [minirefresh](#minirefresh)
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
	- [滚动大小](#滚动大小)
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


># <h1 id="ES6语法">ES6语法</h1>

<br/>

># <h2 id="箭头函数">[=> 箭头函数](https://juejin.cn/post/6844903573428371464)是ES6语法</h2>

> [**基础语法**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

```
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```

<br/>

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```
//情况一:没有参数,函数应该写成一对圆括号
/*
	() => { statements }
*/

var f = () => 5;
// 等同于
var f = function () { return 5 };



//情况二: 当只有一个参数时，圆括号是可选的
/*
	(singleParam) => { statements }
	singleParam => { statements }
*/

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```


<br/>

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。

```
var sum = (num1, num2) => { return num1 + num2; }
```


<br/>

箭头函数可以与变量解构结合使用

```
const full = ({ first, last }) => first + ' ' + last;

// 等同于
function full(person) {
  return person.first + ' ' + person.last;
}
```


<br/>

箭头函数的一个用处是简化回调函数。

```
// 普通函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
```


另一个例子是

```
// 普通函数写法
var result = values.sort(function (a, b) {
  return a - b;
});

// 箭头函数写法
var result = values.sort((a, b) => a - b);
```

下面是 rest 参数与箭头函数结合的例子。

```
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```


<br/>

**使用注意点:**

（1）箭头函数没有自己的this对象（详见下文）。

（2）不可以当作构造函数，也就是说，不可以对箭头函数使用new命令，否则会抛出一个错误。

（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用yield命令，因此箭头函数不能用作 Generator 函数。


上面四点中，最重要的是第一点。对于普通函数来说，内部的this指向函数运行时所在的对象，但是这一点对箭头函数不成立。它没有自己的this对象，内部的this就是定义时上层作用域中的this。也就是说，箭头函数内部的this指向是固定的，相比之下，普通函数的this指向是可变的。

```
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```
上面代码中，setTimeout()的参数是一个箭头函数，这个箭头函数的定义生效是在foo函数生成时，而它的真正执行要等到 100 毫秒后。如果是普通函数，执行时this应该指向全局对象window，这时应该输出21。但是，箭头函数导致this总是指向函数定义生效时所在的对象（本例是{id: 42}），所以打印出来的是42。


<br/>

下面例子是回调函数分别为箭头函数和普通函数，对比它们内部的this指向。

```
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数
  setInterval(() => this.s1++, 1000);
  // 普通函数
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
```
上面代码中，Timer函数内部设置了两个定时器，分别使用了箭头函数和普通函数。前者的this绑定定义时所在的作用域（即Timer函数），后者的this指向运行时所在的作用域（即全局对象）。所以，3100 毫秒之后，timer.s1被更新了 3 次，而timer.s2一次都没更新。


<br/>

箭头函数实际上可以让this指向固定化，绑定this使得它不再可变，这种特性很有利于封装回调函数。下面是一个例子，DOM 事件的回调函数封装在一个对象里面。

```
var handler = {
  id: '123456',

  init: function() {
    document.addEventListener('click',
      event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
};
```
上面代码的init()方法中，使用了箭头函数，这导致这个箭头函数里面的this，总是指向handler对象。如果回调函数是普通函数，那么运行this.doSomething()这一行会报错，因为此时this指向document对象。


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

> <h2 id="对象">对象</h2>

CommonJS 模块输出一组变量，就非常合适使用简洁写法。


```
let ms = {};

function getItem (key) {
  return key in ms ? ms[key] : null;
}

function setItem (key, value) {
  ms[key] = value;
}

function clear () {
  ms = {};
}

module.exports = { getItem, setItem, clear };
// 等同于
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```



<br/>

> <h3 id = 'Object.setPrototypeOf()'> Object.setPrototypeOf()</h3>

&emsp; Object.setPrototypeOf方法的作用与__proto__相同，用来设置一个对象的原型对象（prototype），返回参数对象本身。

```
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40

```
上面代码将proto对象设为obj对象的原型，所以从obj对象可以读取proto对象的属性。

<br/>

上面代码将proto对象设为obj对象的原型，所以从obj对象可以读取proto对象的属性。

如果第一个参数不是对象，会自动转为对象。但是由于返回的还是第一个参数，所以这个操作不会产生任何效果。

```
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true
```
由于undefined和null无法转为对象，所以如果第一个参数是undefined或null，就会报错。

```
Object.setPrototypeOf(undefined, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
```


<br/>
<br/>

> <h3 id = 'Object.getPrototypeOf()'> Object.getPrototypeOf()</h3>

`Object.getPrototypeOf(obj);`

下面是一个例子。

```
function Rectangle() {
  // ...
}

const rec = new Rectangle();

Object.getPrototypeOf(rec) === Rectangle.prototype
// true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype
// false
```

如果参数不是对象，会被自动转为对象。

```
// 等同于 Object.getPrototypeOf(Number(1))
Object.getPrototypeOf(1)
// Number {[[PrimitiveValue]]: 0}

// 等同于 Object.getPrototypeOf(String('foo'))
Object.getPrototypeOf('foo')
// String {length: 0, [[PrimitiveValue]]: ""}

// 等同于 Object.getPrototypeOf(Boolean(true))
Object.getPrototypeOf(true)
// Boolean {[[PrimitiveValue]]: false}

Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
```

如果参数是undefined或null，它们无法转为对象，所以会报错。

```
Object.getPrototypeOf(null)
// TypeError: Cannot convert undefined or null to object

Object.getPrototypeOf(undefined)
// TypeError: Cannot convert undefined or null to object
```








<br/>
<br/>
<br/>

> <h2 id="Generator">Generator</h2>

>  <h3 id="作为对象属性的Generator函数">作为对象属性的 Generator 函数</h3>



如果一个对象的属性是 Generator 函数，可以简写成下面的形式。

```
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```

上面代码中，myGeneratorMethod属性前面有一个星号，表示这个属性是一个 Generator 函数。

它的完整形式如下，与上面的写法是等价的。

```
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```



<br/>
<br/>
<br/>


> <h2 id="Promise对象">Promise对象</h2>

&emsp;Promise是一个对象,用来传递异步操作消息.其构造函数接受一个函数作为参数,改函数的两个参数分别是resolve和reject.它们是2个函数,由Javascript引擎提供,不用自己部署.

```
var promise = new Promise(function(resolve, reject) {
	// .....操作逻辑
	
	if(/*异步操作成功*/) {
		resolve(value)
	}else {
		reject(error)
	}
})
```
Promise实例完成以后可以通过then方法分别指定Resolved状态和Rejected状态的回调函数:

```
promise.then(function(value){
	// success
}, function(value){
	//failure
})
```




<br/>
<br/>
<br/>


> <h2 id="Module模块">Module模块</h2>

&emsp; ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```
// CommonJS模块
let { stat, exists, readfile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;

```

&emsp; 上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

&emsp; ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。

```
// ES6模块
import { stat, exists, readFile } from 'fs';
```

&emsp; 上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。



<br/>
<br/>
<br/>


> <h2 id="require和import"> [require和import](https://www.cnblogs.com/libin-1/p/7127481.html) </h2>

> **调用时间**

- require 是运行时调用，所以理论上可以运作在代码的任何地方
- import 是编译时调用，所以必须放在文件的开头

<br/>


> **本质**

- **`require 是赋值过程`**，其实require的结果就是对象、数字、字符串、函数等，再把结果赋值给某个变量。它是普通的值拷贝传递。

- **`import 是解构过程。使用import导入模块的属性或者方法是引用传递`**。且import是read-only的，值是单向传递的。default是ES6 模块化所独有的关键字，export default {} 输出默认的接口对象，如果没有命名，则在import时可以自定义一个名称用来关联这个对象




<br/>
<br/>

> <h3 id="export用法"> export用法 </h3>

&emsp; 一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果希望外部能够读取模块内部的某个变量，就必须使用`export`关键字输出该变量。下面是一个 JS 文件，里面使用export命令输出变量。

```
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

&emsp; 上面代码是profile.js文件，保存了用户信息。ES6 将其视为一个模块，里面用export命令对外部输出了三个变量。

export的写法，还可以等价下面的:

```
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export { firstName, lastName, year };
```

&emsp; 上面代码在export命令后面，使用大括号指定所要输出的一组变量。它与前一种写法（直接放置在var语句前）是等价的，但是应该优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量,个人建议使用这个.

&emsp; export命令除了输出变量，还可以输出函数或类（class）。

```
export function multiply(x, y) {
  return x * y;
};
```

上面代码对外输出一个函数multiply。

&emsp; 通常情况下，export输出的变量就是本来的名字，但是可以使用as关键字重命名。

```
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

&emsp; 上面代码使用as关键字，重命名了函数v1和v2的对外接口。重命名后，v2可以用不同的名字输出两次。


<br/>

> default默认导出和非默认导出

```
// 第一组
export default function crc32() { // 输出
  // ...
}
// crc32 不需要加 {}
import crc32 from 'crc32'; // 输入




// 第二组
export function crc32() { // 输出
  // ...
};
// crc32 需要加 {}
import {crc32} from 'crc32'; // 输入
```


&emsp; `export default`命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此export default命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能唯一对应export default命令。

&emsp; 本质上，export default就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';
```


<br/>
<br/>

> default导出和普通导出混合

在一条import语句中，同时输入默认方法和其他接口，可以写成下面这样。

```
import _, { each, forEach } from 'lodash';
```


对应上面代码的export语句如下。

```
// 对应 import _ from 'lodash'
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach };
```
上面代码的最后一行的意思是，暴露出forEach接口，默认指向each接口，即forEach和each指向同一个方法。





<br/>
<br/>


> <h3 id="import用法"> import用法 </h3>


> 整体模块加载

```
// circle.js


export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

```


加载所需要的模块和方法:

```
// main.js

import { area, circumference } from './circle';

console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));
上面写法是逐一指定要加载的方法，整体加载的写法如下。


//等价于下面的

import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```





<br/>
<br/>

> <h3 id="require用法"> require用法 </h3>

&emsp; 在CommonJS中，有一个全局性方法require()，用于加载模块。假定有一个数学模块math.js，就可以像下面这样加载。

|:--|:--|
| 1 | var math = require('math'); |

然后，就可以调用模块提供的方法：

| 1 | var math = require('math'); |
|:--|:--|
| 2 | math.add(2,3); // 5 |

或者

**写法一:**

```
// module.js
module.exports = {
    a: function() {
        console.log('exports from module');
    }
}
```

使用:

```
// sample.js
var obj = require('./module.js');
obj.a()  // exports from module
```


<br/>

**写法二:**

当我们不需要导出模块中的全部数据时，使用大括号包含所需要的模块内容。

```
// module.js
function test(str) {
  console.log(str); 
}
module.exports = {
 test
}
```

使用:

```
// sample.js
let { test } =  require('./module.js');
test ('this is a test');
```








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


># <h1 id="三方库">三方库</h1>


<br/>

>## <h2 id="minirefresh">[minirefresh](https://github.com/minirefresh/minirefresh)</h2>

&emsp; [优雅的H5下拉刷新。零依赖，高性能，多主题，易拓展.](http://www.minirefresh.com/minirefresh-doc/)

![react42](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react42.png)


<br/>
<br/>
<br/>

> <h2 id=""></h2>


<br/>
<br/>
<br/>

> <h2 id=""></h2>


<br/>
<br/>
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


># <h1 id="Dom对象">[Dom对象](https://aqingya.cn/articl/a3b68a60.html#题目4)</h1>


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

> <h2 id="滚动大小">滚动大小</h2>

&emsp; 滚动大小 (scroll dimension), 指的是包含滚动内容的元素的大小。有些元素 (例如元素)，即使没有执行任何代码也能自动地添加滚动条；但另外一些元素，则需要通过 CSS 的 overflow 属性进行设置才能滚动。以下是 4 个与滚动大小相关的属性。

- scrollHeight: 在没有滚动条的情况下，元素内容的总高度。
- scrollwidth: 在没有滚动条的情况下，元素内容的总宽度。
- scrollLeft: 被隐藏在内容区域左侧的像素数。通过设置这个属性可以改变元素的滚动位置。
- scrollTop: 被隐藏在内容区域上方的像素数。通过设置这个属性可以改变元素的滚动位置。

![滚动四属性](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react28.png)


&emsp; 在确定文档的总高度时 (包括基于视口的最小高度时), 必须取得 scrollwidth/clientwidth 和 scrollHeight/clientHeight 中的最大值，才能保证在跨浏览器的环境下得到精确的结果。下面就是这样一个例子。

&emsp; 注意，对于运行在混杂模式下的 IE, 则需要用 document . body 代替 document . documentElement。

```
var docHeight = Math.max(document.documentElement.scrollHeight, document.documentElement.clientHeight);
var docWidth = Math.max(document.documentElement.scrollWidth, document.documentElement.clientWidth);

console.log(docHeight);
console.log(docWidth);
```

&emsp; 通过 scrollLeft 和 scrollTop 属性既可以确定元素当前滚动的状态，也可以设置元素的滚动位置。在元素尚未被滚动时，这两个属性的值都等于 0。如果元素被垂直滚动了，那么 scrollTop 的值会大于 0，且表示元素上方不可见内容的像素高度。如果元素被水平滚动了，那么 scrollLeft 的值会大于 0，且表示元素左侧不可见内容的像素宽度。这两个属性都是可以设置的，因此将元素的 scrollLeft 和 scrollTop 设置为 0, 就可以重置元素的滚动位置。下面这个函数会检测元素是否位于顶部，如果不是就将其回滚到顶部。

```
function scrollToTop(element) {
    if (element.scrollTop != 0) {
        element.scrollTop = 0;
    }
}
```




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



