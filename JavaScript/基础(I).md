> <h1 id=""></h1>
- [**import和require**](#import和require)
- [**函数**](#函数)
	- [Function()构造函数](#Function()构造函数)
	- [自调用函数](#自调用函数)
	- [函数是对象](#函数是对象)
	- [箭头函数](#箭头函数)
	- [函数作为方法调用](#函数作为方法调用)
	- [使用构造函数调用函数](#使用构造函数调用函数)
	- [作为函数方法调用函数](#作为函数方法调用函数)
	- [JavaScript闭包](#JavaScript闭包)
- [**对象高级使用**](#对象高级使用)
	- [创建 JavaScript 对象](#创建JavaScript对象)
	- 	[使用Object生成对象](#使用Object生成对象)
	- [	使用字面量创建对象](#使用字面量创建对象)
	- 	[使用对象构造器](#使用对象构造器)
	- [	属性添加到 JavaScript 对象](#属性添加到JavaScript对象)
	- 	[把方法添加到JavaScript对象](#把方法添加到JavaScript对象)
	- 	[JavaScript 类](#JavaScript类)
	- 	[JavaScript 的对象是可变的](#JavaScript的对象是可变的)
	- [	new 和不 new的区别](#new和不new的区别)
- [**HTML的DOM**](#HTML的DOM)
	- [改变CSS样式](#改变CSS样式)
- [**数组**](#数组)
- **参考资料：**



<br/>

***
<br/>

># <h1 id="import和require">[import和require](https://www.cnblogs.com/libin-1/p/7127481.html)</h1>

&emsp; 在es6以前，还没有提出一套官方的规范,从社区和框架推广程度而言,目前通行的javascript模块规范有两种：**CommonJS** 和 **AMD**。

- **CommonJS规范**

&emsp; 2009年，美国程序员Ryan Dahl创造了node.js项目，将javascript语言用于服务器端编程。

&emsp; 这标志”Javascript模块化编程”正式诞生。前端的复杂程度有限，没有模块也是可以的，但是在服务器端，一定要有模块，与操作系统和其他应用程序互动，否则根本没法编程。

&emsp; node编程中最重要的思想之一就是模块，而正是这个思想，让JavaScript的大规模工程成为可能。模块化编程在js界流行，也是基于此，随后在浏览器端，requirejs和seajs之类的工具包也出现了，可以说在对应规范下，require统治了ES6之前的所有模块化编程，即使现在，在ES6 module被完全实现之前，还是这样。

&emsp; 在CommonJS中,暴露模块使用module.exports和exports，很多人不明白暴露对象为什么会有两个,后面会介绍区别

&emsp; 在CommonJS中，有一个全局性方法require()，用于加载模块。假定有一个数学模块math.js，就可以像下面这样加载。

|:--|:--|
| 1 | var math = require('math'); |


然后，就可以调用模块提供的方法：

| 1 | var math = require('math'); |
|:--|:--|
| 2 | math.add(2,3); // 5 |

　
　



<br/>


**import 和 require 是JS模块化编程使用的.**


- **调用时间**
	- require 是运行时调用，所以理论上可以运作在代码的任何地方
	- import 是编译时调用，所以必须放在文件的开头


<br/>


- **本质**
	- require 是赋值过程，其实require的结果就是对象、数字、字符串、函数等，再把结果赋值给某个变量。它是普通的值拷贝传递。
	- import 是解构过程。使用import导入模块的属性或者方法是引用传递。且import是read-only的，值是单向传递的。default是ES6 模块化所独有的关键字，export default {} 输出默认的接口对象，如果没有命名，则在import时可以自定义一个名称用来关联这个对象



<br/>
<br/>


- **语法用法展示**

	- **require的基本语法**

  &emsp; 在导出的文件中使用module.exports对模块中的数据导出，内容类型可以是字符串，变量，对象，方法等不予限定。使用require()引入到需要的文件中即可

  &emsp; 在模块中，将所要导出的数据存放在module的export属性中，在经过CommonJs/AMD规范的处理，在需要的页面中使用require指定到该模块，即可导出模块中的export属性并执行赋值操作（值拷贝）

```
// 在 module.js 文件中
module.exports = {
    a: function() {
        console.log('exports from module');
    }
}



// 在 sample.js 文件中使用
var obj = require('./module.js');
obj.a()  // exports from module
```

<br/>

&emsp; 当我们不需要导出模块中的全部数据时，使用大括号包含所需要的模块内容。

```
// 在 module.js 文件中
function test(str) {
	console.log(str); 
}
module.exports = {
	test
}


//  在 sample.js 文件中使用
let { test } =  require('./module.js');
test ('this is a test');
```


<br/>
<br/>

- **import 的基本语法**

&emsp;  使用import导出的值与模块中的值始终保持一致，即引用拷贝，采用ES6中解构赋值的语法，import配合export结合使用

```
// 在 module.js 文件中
export function test(args) {
  console.log(args);
}
// 定义一个默认导出文件, 一个文件只能定义一次
export default {
  a: function() {
    console.log('export from module');
  }
}

export const name = 'gzc'

```

```
// 使用_导出export default的内容
import _, { test, name } from './a.js'

test(`my name is ${name}`)  // 模板字符串中使用${}加入变量
```




<br/>
<br/>

- **写法形式**
	- require/exports 方式的写法比较统一

	```
	// require module.js 文件
	const module = require('module')
	// exports
	export.fs = fs
	module.exports = fs
	```


	- import/export 方式的写法就相对丰富些

	```
	// import
	import fs  from 'fs';
	import { newFs as fs } from 'fs';  // ES6语法, 将fs重命名为newFs, 命名冲突时常用
	import { part } from fs;
	import fs, { part } from fs;
	```
	
	```
	// export
	export default fs;
	export const fs;
	export function part;
	export { part1, part2 };
	export * from 'fs';
	```


<br/>
<br/>


- **要点总结**
	- 通过require引入基础数据类型时,属于复制该变量
	- 通过require引入复杂数据类型时, 属于浅拷贝该对象
	- 出现模块之间循环引用时, 会输出已执行的模块, 未执行模块不会输出
	- CommonJS规范默认export的是一个对象,即使导出的是基础数据类型




<br/>

***
<br/>

> <h1 id="函数">函数</h1>

> <h2 id="Function()构造函数">**Function()构造函数**</h2>

函数同样可以通过内置的 JavaScript 函数构造器（Function()）定义。

```
var myFunction = new Function("a", "b", "return a * b");

var x = myFunction(4, 3);

console.log(x)


//等价于
var myFunction1 = function (a, b) {return a * b};

var x1 = myFunction1(4, 3);
```
打印为： 12


<br/>


> <h2 id="自调用函数">**自调用函数**</h2>


- 函数表达式可以 "自调用"。

- 自调用表达式会自动调用。

- 如果表达式后面紧跟 () ，则会自动调用。

- 不能自调用声明的函数。

- 通过添加括号，来说明它是一个函数表达式：

```
(function () {
    var x = "Hello!!";      // 我将调用自己
})();
```

<br/>


> <h2 id="函数是对象">**函数是对象**</h2>


在 JavaScript 中使用 typeof 操作符判断函数类型将返回 "function" 。

但是JavaScript 函数描述为一个对象更加准确。

JavaScript 函数有 属性 和 方法。

arguments.length 属性返回函数调用过程接收到的参数个数：

```
<body>

<p> arguments.length 属性返回函数接收到参数的个数：</p>
<p id="demo"></p>
<script>
function myFunction(a, b) {
    return arguments.length;
}
document.getElementById("demo").innerHTML = myFunction(4, 3);
</script>

</body>
```

在HTML的显示效果是：

```
arguments.length 属性返回函数接收到参数的个数：

2
```





<br/>


> <h2 id="箭头函数">**箭头函数**</h2>


ES6 新增了箭头函数。

箭头函数表达式的语法比普通函数表达式更简洁。

```
(参数1, 参数2, …, 参数N) => { 函数声明 }

(参数1, 参数2, …, 参数N) => 表达式(单一)
// 相当于：(参数1, 参数2, …, 参数N) =>{ return 表达式; }
```

当只有一个参数时，圆括号是可选的：

```
(单一参数) => {函数声明}
单一参数 => {函数声明}
```


没有参数的函数应该写成一对圆括号:

```
() => {函数声明}
```

实例：

```
// ES5
var x = function(x, y) {
     return x * y;
}
 
// ES6
const x = (x, y) => x * y;
```



<br/>


> <h2 id="函数作为方法调用">**函数作为方法调用**</h2>

在 JavaScript 中你可以将函数定义为对象的方法。

以下实例创建了一个对象 (myObject), 对象有两个属性 (firstName 和 lastName), 及一个方法 (fullName):

```
var myObject = {
    firstName:"John",
    lastName: "Doe",
    fullName: function () {
        return this.firstName + " " + this.lastName;
    }
}
myObject.fullName(); 

```


<br/>


> <h2 id="使用构造函数调用函数">**使用构造函数调用函数**</h2>

如果函数调用前使用了 new 关键字, 则是调用了构造函数。

这看起来就像创建了新的函数，但实际上 JavaScript 函数是重新创建的对象：

```
// 构造函数:
function myFunction(arg1, arg2) {
    this.firstName = arg1;
    this.lastName  = arg2;
}
 
// This    creates a new object
var x = new myFunction("John","Doe");
x.firstName; 
```



<br/>


> <h2 id="作为函数方法调用函数">**作为函数方法调用函数**</h2>

在 JavaScript 中, 函数是对象。JavaScript 函数有它的属性和方法。

call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身。

```
function myFunction(a, b) {
    return a * b;
}
myObject = myFunction.call(myObject, 10, 2);     // 返回 20


//参数为数组
function myFunction(a, b) {
    return a * b;
}
myArray = [10, 2];
myObject = myFunction.apply(myObject, myArray);  // 返回 20
```


<br/>


> <h2 id="JavaScript闭包">**JavaScript 闭包**</h2>

```
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();
 
add();
add();
add();
 
// 计数器为 3
```





<br/>

***
<br/>

># <h1 id="对象高级使用">[**‌对象高级使用**](https://www.runoob.com/js/js-objects.html)</h1>


<br/>

<h3 id="创建JavaScript对象">**1）.创建 JavaScript 对象**</h3>



通过 JavaScript，您能够定义并创建自己的对象。

- 创建新对象有两种不同的方法：
	- 使用 Object 定义并创建对象的实例。
	- 使用函数来定义对象，然后创建新的对象实例。


- **使用 Object**

在 JavaScript 中，几乎所有的对象都是 Object 类型的实例，它们都会从 Object.prototype 继承属性和方法。

Object 构造函数创建一个对象包装器。

- Object 构造函数，会根据给定的参数创建对象，具体有以下情况：

	- 如果给定值是 null 或 undefined，将会创建并返回一个空对象。
	- 如果传进去的是一个基本类型的值，则会构造其包装类型的对象。
	- 如果传进去的是引用类型的值，仍然会返回这个值，经他们复制的变量保有和源对象相同的引用地址。
	- 当以非构造函数形式被调用时，Object 的行为等同于 new Object()。

语法格式：

```
// 以构造函数形式来调用
new Object([value])
```


value 可以是任何值。

<br/>


<h3 id="使用Object生成对象">**2). 使用Object生成对象**</h3>


以下实例使用 Object 生成布尔对象：

```
// 等价于 o = new Boolean(true);
var o = new Object(true);
```


这个例子创建了对象的一个新实例，并向其添加了四个属性：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
var person=new Object();
person.firstname="John";
person.lastname="Doe";
person.age=50;
person.eyecolor="blue"; 
document.write(person.firstname + " is " + person.age + " years old.");
</script>

</body>
</html>

```

HTML 显示：

**John is 50 years old.**


<br/>

<h3 id="使用字面量创建对象">**3). 使用字面量创建对象**</h3>

也可以使用对象字面量来创建对象，语法格式如下：

```
{ name1 : value1, name2 : value2,...nameN : valueN }
```

案例测试：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
person={firstname:"John",lastname:"Doe",age:50,eyecolor:"blue"}
document.write(person.firstname + " is " + person.age + " years old.");
</script>

</body>
</html>
```

HTML显示：

```
John is 50 years old.
```

**JavaScript 对象就是一个 name:value 集合。**


<br/>


<h3 id="使用对象构造器">**4). 使用对象构造器**</h3>

本例使用函数来构造对象：


```
function person(firstname,lastname,age,eyecolor){
	this.firstname=firstname;
	this.lastname=lastname;
	this.age=age;
    this.eyecolor=eyecolor;
}
myFather=new person("John","Doe",50,"blue");

console.log(myFather.firstname + " is " + myFather.age + " years old.");
```

打印：

```
John is 50 years old.
```

在JavaScript中，this通常指向的是我们正在执行的函数本身，或者是指向该函数所属的对象（运行时）




<br/>

<h3 id="属性添加到JavaScript对象">**5）.属性添加到 JavaScript 对象**</h3>

- 您可以通过为对象赋值，向已有对象添加新属性：

- 假设 person 对象已存在 - 您可以为其添加这些新属性：firstname、lastname、age 以及 eyecolor：

```
person.firstname="John";
person.lastname="Doe";
person.age=30;
person.eyecolor="blue";

x=person.firstname;
console.log(x)
```

打印为：

```
John
```



<br/>

<h3 id="把方法添加到JavaScript对象">**6）.把方法添加到 JavaScript 对象**</h3>

- 方法只不过是附加在对象上的函数。

- 在构造器函数内部定义对象的方法：

```
function person(firstname,lastname,age,eyecolor){
    this.firstname=firstname;
    this.lastname=lastname;
    this.age=age;
    this.eyecolor=eyecolor;
    this.changeName=changeName;
	function changeName(name){
		this.lastname=name;
	}
}

//changeName() 函数 name 的值赋给 person 的 lastname 属性。
myMother=new person("Sally","Rally",48,"green");
myMother.changeName("Doe");
console.log(myMother.lastname);
```

打印为：

```
Doe
```



<br/>

<h3 id="JavaScript类">**7）.JavaScript 类**</h3>

JavaScript 是面向对象的语言，但 JavaScript 不使用类。

在 JavaScript 中，不会创建类，也不会通过类来创建对象（就像在其他面向对象的语言中那样）。

JavaScript 基于 prototype，而不是基于类的。

**JavaScript for...in 循环**

JavaScript for...in 语句循环遍历对象的属性。

```
//注意： for...in 循环中的代码块将针对每个属性执行一次。
for (variable in object)
{
    执行的代码……
}
```

Demo案例：

```
function myFunction(){
	var x;
	var txt="";
	var person={fname:"Bill",lname:"Gates",age:56}; 
	for (x in person){
		txt=txt + person[x];
	}
	console.log(txt);
}

myFunction()
```

打印：

```
BillGates56
```


<br/>

<h3 id="JavaScript的对象是可变的">**8）.JavaScript 的对象是可变的**</h3>


对象是可变的，它们是通过引用来传递的。

以下实例的 person 对象不会创建副本：

```
var x = person;  // 不会创建 person 的副本，是引用

```


案例Demo：

```
var person = {firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"}

var x = person;
x.age = 10;

Console.log(
person.firstName + " is " + person.age + " years old.")
```

打印：

```
John is 10 years old.
```




<h3 id="new和不new的区别">**9）.new 和不 new的区别**</h3>

&emsp; 如果 new 了函数内的 this 会指向当前这个 person 并且就算函数内部不 return 也会返回一个对象。
&emsp; 如果不 new 的话函数内的 this 指向的是 window。


```
function person(firstname,lastname,age,eyecolor)
{
    this.firstname=firstname;
    this.lastname=lastname;
    this.age=age;
    this.eyecolor=eyecolor;
    return [this.firstname,this.lastname,this.age,this.eyecolor,this] 
}

var myFather=new person("John","Doe",50,"blue");
var myMother=person("Sally","Rally",48,"green");
console.log(myFather) 
console.log(myMother) 
```

打印：


```
this 输出一个 person 对象

this 输出 window 对象
```




<br/>

***
<br/>

> <h1 id="HTML的DOM">**‌HTML的DOM**</h1>


<br/>

> <h2 id="改变CSS样式">**改变CSS样式**</h2>


- **改变 HTML 样式**
如需改变 HTML 元素的样式，请使用这个语法：

```
document.getElementById(id).style.property=新样式

```
案例demo：

```
<body>
 
<p id="p1">Hello World!</p>
<p id="p2">Hello World!</p>
<script>
document.getElementById("p2").style.color="blue";
document.getElementById("p2").style.fontFamily="Arial";
document.getElementById("p2").style.fontSize="larger";
</script>
```




<br/>

***
<br/>

> <h1 id="数组">数组</h1>
**1）.** 添加元素

- push()方法是给数组尾巴添加一个或多个元素,返回的是添加数组后该数组长度;
- add()方法是不能操作数组的，不然会报错;

```
//push 操作
var arr = [1,2,3,4];
var arrlength = arr.push(5);
console.log("arr长度为："+arrlength+"\narr值为："+arr);

打印：arr长度为：5， 值为：[1,2,3,4,5]


//add 操作
var arr = [1,2,3,4];
var arrlength = arr.add(5);
console.log("arr长度为："+arrlength+"\narr值为："+arr);

//报错：arr.add add is not a function
```



**add()**主要是针对dom节点进行元素的添加，比如说你要向某个div里面添加一个片段就可以写:
```
$(".div").add("<p>hello,world!</p>");
```

它有一点像clone()函数，只是一个复制(基于原有的)一个添加（原有的或者新增的）； 


<br/>
<br/>

**2.)删除元素:splice()和remove()区别** 

- splice()主要是针对数组的插入，删除和替换元素，由参数决定。 

```
//index_num表示操作的数组下标，num表示操作的个数
splice(index_num,num);
```

```
var arr = [1,2,3,4];
var arrrm = arr.splice(3,1);
console.log("arr删除的数据为："+arrrm+"\narr删除后的值为："+arr);
//arr删除的数据为：4
//arr删除后的值为：1,2,3
```


- Remove()在js中主要是针对dom节点进行操作的，比如说我们要删除某个子节点的话可以这样使用：

```
$(".div select").remove("option:nth-child(2)");

//删除select的第二个孩子节点。
``` 

纵上所述：有关数组元素和DOM元素的删除与新增一定要有所区分哟，切记不能混用！ 
但是要是你实在是觉得splice这个字母不好记，硬要用arr.remove()操作来删除元素的话是未尝不可的，我们可以通过自定义原型方法来实现这个功能，满足你的想法：

```
Array.prototype.remove = function(val) { 
    var index = this.indexOf(val); 
    if (index > -1) { 
        return  this.splice(index, 1); 
    }

};
var arr = [1,2,3,4];
var arrrm = arr.remove(4);
console.log("arr删除的数据为："+arrrm+"\narr删除后的值为："+arr);
//arr删除的数据为：4
//arr删除后的值为：1,2,3
```


<br/>

***
<br/>

> <h1 id=""></h1>
> 



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>



<br/>

***
<br/>

> <h1 id=""></h1>

