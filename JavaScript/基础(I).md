> <h1 id=""></h1>
- [**import和require**](#import和require)
- [**数组**](#数组)
	- [对象数组](#对象数组)
- [undefined vs null哪个好](#undefinedvsnull哪个好)
- [**函数**](#函数)
	- [Function()构造函数](#Function()构造函数)
	- [自调用函数](#自调用函数)
	- [函数是对象](#函数是对象)
	- [箭头函数](#箭头函数)
	- [函数作为方法调用](#函数作为方法调用)
	- [使用构造函数调用函数](#使用构造函数调用函数)
	- [作为函数方法调用函数](#作为函数方法调用函数)
	- [JavaScript闭包](#JavaScript闭包)
	- [举例复杂函数声明demo](#举例复杂函数声明demo)
		- [语法简写特性-返回值参数和行参相同](#语法简写特性-返回值参数和行参相同)
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
- [**Promise**](#Promise)
	- [Promise链式调用demo](#Promise链式调用demo)
	- [延迟Promise代码详解](#延迟Promise代码详解)
- [**HTML的DOM**](#HTML的DOM)
	- [改变CSS样式](#改变CSS样式)
- [**打印Console**](#打印Console)
	- [类型信息输出](#类型信息输出)
	- [格式化输出](#格式化输出)
- **参考资料：**
	- [面试资料I](https://dragon-li.gitee.io/my-wiki/)



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

> <h1 id="数组">数组</h1>

- splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目。该方法会改变原始数组。

```
arrayObject.splice(index,howmany,item1,.....,itemX)


index	必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
howmany	必需。要删除的项目数量。如果设置为 0，则不会删除项目。
item1, ..., itemX	可选。向数组添加的新项目。

```




***
<br/><br/><br/>
> <h2 id="对象数组">对象数组</h2>

**原代码结构**

```tsx
const serviceLists = [
  { label: <FormattedMessage id={'pages.product.list.service.all'} />, value: '' },
  ...globalRegionList.map(res => {
    return {
      label: res.name,
      value: res.region
    }
  })
]
```

---
<br/>

- ✅**语法详解**

- **1.`serviceLists = [...]` 是什么？**

这是一个 **数组**。数组中每个元素都是一个 **对象**，对象里有 `label` 和 `value` 两个字段。

所以整体结构是：

```ts
[
  { label: ..., value: ... },
  { label: ..., value: ... },
  ...
]
```

也就是一个“对象数组”。

<br/>

- **2. 第一项：**

```tsx
{ label: <FormattedMessage id={'pages.product.list.service.all'} />, value: '' }
```

* 这是数组的第一个元素。
* `label` 是一个 React 元素（`<FormattedMessage />`，来自 `react-intl`，用于国际化显示）
* `value` 是空字符串 `''`

用于表示“全部选项”或“默认选项”。

<br/>

- **3.`...globalRegionList.map(...)`**

	* `globalRegionList.map(...)`：对 `globalRegionList` 中的每个元素进行转换（生成对象）
	* 每个 `res` 被转成：

  ```ts
  {
    label: res.name,
    value: res.region
  }
  ```
* `...` 展开运算符：将 map 出来的数组元素“摊开”插入到 serviceLists 数组中。

<br/>

**🔍 例如：**

```ts
globalRegionList = [
  { name: '华东', region: 'cn-east' },
  { name: '华南', region: 'cn-south' },
]
```

<br/>

执行 `globalRegionList.map(...)` 得到：

```ts
[
  { label: '华东', value: 'cn-east' },
  { label: '华南', value: 'cn-south' },
]
```

<br/>

加上 `...` 展开后，`serviceLists` 最终是：

```ts
[
  { label: <FormattedMessage id="..." />, value: '' },
  { label: '华东', value: 'cn-east' },
  { label: '华南', value: 'cn-south' },
]
```

---
<br/>

这种结构常见用于下拉框、单选框、表格过滤器等组件的数据源，例如：

```tsx
<Select options={serviceLists} />
```

或者：

```tsx
<Form.Item name="region" label="区域">
  <Select>
    {serviceLists.map(item => (
      <Select.Option key={item.value} value={item.value}>
        {item.label}
      </Select.Option>
    ))}
  </Form.Item>
</Select>
```



***
<br/><br/><br/>
> <h2 id="undefinedvsnull哪个好">undefined vs null哪个好</h2>

在 React（或 JavaScript）中，**`undefined` vs `null`（你说的 `nil` 应该是 `null`）** 是常见的问题。

如果你定义：

```tsx
state = {
  firmwareMerge: undefined
}
```

你问的核心是：

> ✅ `firmwareMerge` 用 `undefined` 还是 `null` 比较好？怎么选？

---
<br/>

- **✅ 区别一览（重点）**

| 对比项          | `undefined`                        | `null`                     |
| ------------ | ---------------------------------- | -------------------------- |
| 语义           | 变量“未定义”（未赋值）                       | 变量已定义但“无值”                 |
| 使用场景         | 初始状态、延迟赋值、尚未加载的数据                  | 明确表示“空”、“无内容”              |
| 是否被序列化（JSON） | `undefined` 会被忽略                   | `null` 会被保留                |
| 类型           | `typeof undefined === 'undefined'` | `typeof null === 'object'` |
| 推荐用法         | 表示“还没加载/未设置”                       | 表示“已加载，但为空”                |

---
<br/>

- **✅ React 中推荐用法，场景判断：**

- **✅ 用 `undefined`：**
	* **数据尚未加载**
	* **不确定值是否存在（初始状态）**
	* **组件挂载后通过异步请求再赋值**

示例：

```tsx
state = {
  firmwareMerge: undefined // 代表“我还不知道值”
}
```

<br/>

- **✅ 用 `null`：**

	* 明确知道“这个地方就是没有值”
	* 是用户主动取消选择、清空值的场景
	* 接口中该字段有意义地返回了 `null`

示例：

```tsx
state = {
  firmwareMerge: null // 表示“明确为空”
}
```


<br/>

**如何统一判断它“没有值”**

```tsx
if (firmwareMerge == null) {
  // 这个条件同时匹配 null 和 undefined
}
```

<br/>

- **✅ 补充：如果你要传给后端/存储/JSON，建议用 `null`**

```js
JSON.stringify({ a: undefined }) // '{}'
JSON.stringify({ a: null })      // '{"a":null}'
```



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


> <h2 id="函数作为方法调用"> **函数作为方法调用** </h2>

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

***
<br/><br/><br/>
> <h2 id="JavaScript闭包">JavaScript闭包</h2>

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


***
<br/><br/><br/>
> <h2 id="">举例复杂函数声明demo</h2>

```js
在xx.tsx文件中有如下代码：

export async function getInitialState(): Promise<{
  settings?: Partial<LayoutSettings>;
  currentUser?: API.CurrentUser;
  loading?: boolean;
  fetchUserInfo?: () => Promise<T>;
}> {
  const fetchUserInfo = async () => {
    try {
      const msg = await getTenantInfo();
      return msg.result;
    } catch (error) {
      history.push(loginPath);
    }
    return undefined;
  };
  // 如果不是登录页面，执行
  const { location } = history;

	// 省略1万行代码
	....
	..
	,
	
	return {
		fetchUserInfo,
		currentUser,
		settings,
		loading,
  };
}
```


- **解释**
	* `export async function getInitialState()`：声明一个**异步函数**，用于导出供其他模块调用。
	* 返回值是一个 `Promise`，其解析结果是一个对象，包含四个可选字段：
	
		* `settings`: 页面布局设置（Partial 表示只传部分字段也可以）
		* `currentUser`: 当前用户信息
		* `loading`: 加载状态布尔值
		* `fetchUserInfo`: 一个函数，返回 Promise，表示“异步获取用户信息的函数”

<br/>

**函数体,从这里起 都是 getInitialState 函数体内的代码**

```ts
  const fetchUserInfo = async () => {
    try {
      const msg = await getTenantInfo();
      return msg.result;
    } catch (error) {
      history.push(loginPath);
    }
    return undefined;
  };
```

- **✅ 解释：**
	
	* `fetchUserInfo` 是一个定义在 `getInitialState` 函数内部的 **局部函数**。
	* `getTenantInfo()` 是一个 API 调用，返回包含 `result` 字段的数据。
	* 如果发生异常，则跳转到 `loginPath`（即登录页）。
	* 最后，如果出错或返回不成功，返回 `undefined`。


***
<br/><br/><br/>
> <h2 id="语法简写特性-返回值参数和行参相同">语法简写特性-返回值参数和行参相同</h2>

- ✅ **返回值是可以缺损（少返回字段）吗？**

可以，只要这些字段是可选的（加了 `?`）

```ts
Promise<{
  settings?: Partial<LayoutSettings>;
  currentUser?: API.CurrentUser;
  loading?: boolean;
  fetchUserInfo?: () => Promise<T>;
}>
```

所有字段都是可选的（`?`），所以你可以只返回其中一部分，例如：

```ts
return {
  fetchUserInfo,
};
```

这也是合法的，其他字段可以省略（即为 `undefined`）。

<br/>

**✅  `fetchUserInfo` 对应的是哪一部分？**

你问“为什么 fetchUserInfo 对应的没有看到”，其实你已经定义并返回了，只是它是函数体内定义的变量。

```ts
const fetchUserInfo = async () => {
  try {
    const msg = await getTenantInfo();
    return msg.result;
  } catch (error) {
    history.push(loginPath);
  }
  return undefined;
};
```

<br/>

然后你在 return 时写了：

```ts
return {
  fetchUserInfo,
  currentUser,
  settings,
  loading,
};
```

所以 `fetchUserInfo` 是作为属性之一被返回了，和你声明的返回类型中这个字段一一对应：

```ts
fetchUserInfo?: () => Promise<T>;
```

也就是说你返回的是：

```ts
{
  fetchUserInfo: () => Promise<T>,
  // 其他字段可能为 undefined 或未定义
}
```


<br/><br/>

**语法简写：**

✅ 你说的“应该是 `fetchUserInfo: fetchUserInfo`”是完全对的！

当我们写：

```ts
return {
  fetchUserInfo: fetchUserInfo
};
```

这是标准的对象字面量写法，表示返回的对象有一个字段叫 `fetchUserInfo`，它的值是变量 `fetchUserInfo` 的值。

<br/>

**✅ 那为什么能简写成 `fetchUserInfo` 呢？**

这是 JavaScript 中的 **对象属性简写（Property shorthand）** 语法：

当对象属性的 **键（key）和变量名一致时**，你可以省略 `key: value` 的写法，直接写变量名，表示 `key` 和 `value` 是同一个名字。


**✅ 更直观的例子：**

```ts
const name = 'Harley';
const age = 18;

const person = {
  name, // 等价于 name: name
  age   // 等价于 age: age
};
```

最终 `person` 是：

```ts
{
  name: 'Harley',
  age: 18
}
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


<br/><br/><br/>

***
<br/>
> <h1 id="Promise">Promise</h1>

**1️⃣. Promise介绍：**

> `Promise` 是 JavaScript 提供的一种异步编程解决方案，用于表示一个\*\*异步操作最终完成（fulfilled）或失败（rejected）\*\*并返回结果的对象。

<br/>

**✅ 二、基本语法**

```js
const promise = new Promise((resolve, reject) => {
  // 执行异步操作
  if (成功) {
    resolve(成功结果);
  } else {
    reject(失败原因);
  }
});
```

- **说明：**

	* `resolve()`：表示成功，结果会传给 `.then()`
	* `reject()`：表示失败，错误会传给 `.catch()`

<br/>

 **三、常见示例**

```js
const fetchData = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('获取数据成功');
    } else {
      reject('获取数据失败');
    }
  }, 1000);
});

fetchData
  .then((result) => {
    console.log('成功：', result);
  })
  .catch((error) => {
    console.log('失败：', error);
  });
```

- **执行顺序：**

	- 1.创建 `Promise` 时立即执行 `new Promise(...)` 中的代码（同步）
	- 2.1 秒后 `setTimeout` 回调触发，调用 `resolve()` 或 `reject()`
	- 3.`.then()` 或 `.catch()` 接收结果（异步触发，进入微任务队列）

<br/>

 **四、Promise 状态说明**

Promise 有三种状态：

| 状态          | 描述                  |
| ----------- | ------------------- |
| `pending`   | 初始状态，进行中            |
| `fulfilled` | 已成功，调用了 `resolve()` |
| `rejected`  | 已失败，调用了 `reject()`  |

> 一旦从 `pending` 转为 `fulfilled` 或 `rejected`，状态就不能再改变。

<br/>

**五、链式调用与执行顺序**

```js
new Promise((resolve) => {
  console.log('1. 执行 Promise 内代码');
  resolve('OK');
})
  .then((res) => {
    console.log('2. then 第一次', res);
    return 'next';
  })
  .then((res) => {
    console.log('3. then 第二次', res);
  });

console.log('4. 同步代码执行完毕');
```

<br/>

**输出顺序：**

```
1. 执行 Promise 内代码
4. 同步代码执行完毕
2. then 第一次 OK
3. then 第二次 next
```

<br/>

- **原因：**

	* `new Promise()` 内代码立即执行（同步）
	* `.then()` 会被放入“微任务队列”，等同步代码执行完再执行

<br/>

**✅ 六、async / await 是 Promise 的语法糖**

```js
function getData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('数据来了');
    }, 1000);
  });
}

async function main() {
  console.log('开始请求');
  const result = await getData(); // 等待 Promise 完成
  console.log('结果是：', result);
}

main();
```

<br/>

**输出：**

```
开始请求
（1秒后）结果是： 数据来了
```

* `await` 会“暂停” async 函数，直到 Promise 被 resolve。
* `async` 函数总是返回一个 Promise。

<br/>

**✅ 七、错误处理**

```js
async function main() {
  try {
    const result = await fetch('wrong-url');
    console.log(result);
  } catch (err) {
    console.error('捕获错误：', err);
  }
}
```

<br/>


**✅ 八、并发 Promise 示例（常用于多个请求）**

```js
const p1 = Promise.resolve(1);
const p2 = new Promise((resolve) => setTimeout(() => resolve(2), 1000));
const p3 = Promise.resolve(3);

Promise.all([p1, p2, p3]).then((res) => {
  console.log('全部完成', res); // [1, 2, 3]
});
```

<br/>

**✅ 总结一张图理解执行顺序：**

```
1. new Promise(...): 同步执行
2. resolve()/reject(): 注册回调
3. then/catch/finally: 异步执行（微任务队列）
4. await：暂停函数，等 Promise 完成
```

---
<br/>

**✅ 快速测试题：**

```js
console.log('A');

new Promise((resolve) => {
  console.log('B');
  resolve();
}).then(() => {
  console.log('C');
});

console.log('D');
```

输出顺序是：

```
A
B
D
C
```



***
<br/><br/><br/>
> <h2 id="Promise链式调用demo">Promise链式调用demo</h2>

下面是一个\*\*「登录 → 获取用户信息 + 个性化设置 → 渲染首页」\*\*的完整 Demo，既包含 *Promise 链式写法*，也包含 *async/await* 写法，并穿插大量日志来演示 **执行顺序**。

---
<br/>

- **1.场景与接口设计**

| 接口                         | 模拟耗时   | 入参 / 返回值                          | 说明       |
| -------------------------- | ------ | --------------------------------- | -------- |
| `login(credentials)`       | 600 ms | `POST {name, pwd}` → `{token}`    | 登录，返回令牌  |
| `fetchUserProfile(token)`  | 400 ms | `GET /me` → `{id, name, role}`    | 获取个人资料   |
| `fetchUserSettings(token)` | 800 ms | `GET /settings` → `{theme, lang}` | 获取用户偏好   |
| `renderHome(state)`        | —      | 把所有数据渲染到页面                        | 最终 UI 渲染 |

---
<br/>

**2.工具函数：模拟接口返回 & 日志助手**

```ts
// utils.ts -------------------------------------------------------------
export const delay = (ms: number) =>
  new Promise<void>((resolve) => setTimeout(resolve, ms));

export function log(step: string, data?: unknown) {
  console.log(
    `%c${new Date().toISOString().slice(11, 23)} ${step}`,
    'font-weight:bold;color:#4e88fe',
    data ?? ''
  );
}
```

<br/>

**3.Promise 链式版 (一步步 then / catch / finally)**

```ts
// login-chain.ts -------------------------------------------------------
import { delay, log } from './utils';

function login(credentials: { name: string; pwd: string }) {
  log('① 调 login');
  return delay(600).then(() => ({ token: 'mock-jwt-token' as const }));
}

function fetchUserProfile(token: string) {
  log('② 调 fetchUserProfile');
  return delay(400).then(() => ({
    id: 1,
    name: 'Harley',
    role: 'admin',
  }));
}

function fetchUserSettings(token: string) {
  log('③ 调 fetchUserSettings');
  return delay(800).then(() => ({
    theme: 'dark',
    lang: 'zh-CN',
  }));
}

function renderHome(state: { profile: any; settings: any }) {
  log('⑦ 渲染首页', state);
  // 这里可把数据写入 React/Vue 状态管理，再更新 UI
}

// --- 正式流程 --------------------------------------------------------
log('--- Promise 链式 DEMO 开始 ---');

login({ name: 'harley', pwd: '123456' }) // ①
  .then((loginRes) => {
    log('④ 登录成功，拿到 token', loginRes);
    // ② + ③ 并发
    return Promise.all([
      fetchUserProfile(loginRes.token),
      fetchUserSettings(loginRes.token),
    ]);
  })
  .then(([profile, settings]) => {
    log('⑤ 两个请求都 OK');
    renderHome({ profile, settings }); // ⑦
  })
  .catch((err) => {
    log('❌ 流程失败', err);
  })
  .finally(() => {
    log('⑥ finally：隐藏全局 Loading');
  });
```

<br/>

**预期控制台顺序**

```
--- Promise 链式 DEMO 开始 ---
① 调 login
④ 登录成功，拿到 token                 (0.6s)
② 调 fetchUserProfile
③ 调 fetchUserSettings
⑤ 两个请求都 OK                       (0.6s + 0.8s ≈ 1.4s)
⑦ 渲染首页
⑥ finally：隐藏全局 Loading
```

> * **①** 的内部同步代码立刻执行
> * **then** 回调按微任务顺序排队
> * `Promise.all` 确保两个并发请求都完成后再进入下一步
> * **finally** 总在链尾执行

<br/>

**4.async / await 版（更贴近日常写法）**

```ts
// login-async.ts -------------------------------------------------------
import { delay, log } from './utils';

async function login(credentials: { name: string; pwd: string }) {
  log('① 调 login');
  await delay(600);
  return { token: 'mock-jwt-token' as const };
}

async function fetchUserProfile(token: string) {
  log('② 调 fetchUserProfile');
  await delay(400);
  return { id: 1, name: 'Harley', role: 'admin' };
}

async function fetchUserSettings(token: string) {
  log('③ 调 fetchUserSettings');
  await delay(800);
  return { theme: 'dark', lang: 'zh-CN' };
}

async function renderHome(state: { profile: any; settings: any }) {
  log('⑦ 渲染首页', state);
}

export async function main() {
  log('--- async/await DEMO 开始 ---');
  try {
    const { token } = await login({ name: 'harley', pwd: '123456' }); // ①

    // ② & ③ 并发：用 Promise.all + await
    const [profile, settings] = await Promise.all([
      fetchUserProfile(token),
      fetchUserSettings(token),
    ]);

    await renderHome({ profile, settings }); // ⑦
  } catch (err) {
    log('❌ 流程失败', err);
  } finally {
    log('⑥ finally：隐藏全局 Loading');
  }
}

main();
```

> **async / await** 只是语法糖——本质上还是用 `Promise.then` 构建微任务，但极大提升可读性。
> `Promise.all([...])` 保证 **两个请求并发 & 都成功** 才继续；若有一个失败即跳去 `catch`。

<br/>

**5.扩展：失败重试 + 指数退避**

实际开发中网络不稳，我们常给关键接口做重试：

```ts
async function retry<T>(
  fn: () => Promise<T>,
  retries = 3,
  delayMs = 500
): Promise<T> {
  try {
    return await fn();
  } catch (err) {
    if (retries === 0) throw err;
    await delay(delayMs);
    return retry(fn, retries - 1, delayMs * 2); // 指数退避
  }
}

// 用法：三次机会拿设置
const settings = await retry(() => fetchUserSettings(token), 2, 300);
```

<br/>

**6.核心执行顺序再回顾**

```
┌ new Promise(...)   ← 加入微任务前的同步阶段
├ resolve / reject
│   └───► .then/.catch  回调排进「微任务队列」
└ 浏览器 / Node 事件循环
    ├ 先清空同步栈
    ├ 执行所有微任务（Promise 回调）
    └ 再执行宏任务（setTimeout 回调等）
```


***
<br/><br/><br/>
> <h2 id="延迟Promise代码详解">延迟Promise代码详解</h2>

**原代码**

```ts
export const delay = (ms: number) =>
  new Promise<void>((resolve) => setTimeout(resolve, ms));
```

<br/>

**✅ 1.`delay(20)` 为什么能直接调用？它是个变量？**

我们看到的是一个**箭头函数表达式赋值给变量**的写法：

```ts
const delay = (ms) => { ... }
```

也可以写成标准函数：

```ts
function delay(ms) {
  ...
}
```

在这段代码中：

```ts
export const delay = (ms: number) => new Promise(...);
```

就是定义了一个变量 `delay`，它是一个函数，接收 `ms: number` 毫秒，返回一个 Promise。因此你可以像 `delay(20)` 这样调用它，完全合法。

<br/>

**✅ 2.`Promise<void>` 中的 `void` 是什么意思？**

这是 **TypeScript 中 Promise 的泛型参数**，表示：

> 这个 Promise **成功（resolve）时不返回任何值**，即返回值是 `undefined`。

**对比：**

```ts
const a: Promise<number> = Promise.resolve(123); // resolve(123)，返回数字

const b: Promise<void> = Promise.resolve();      // resolve(undefined)，没有值
```

对于 `delay` 来说，它的目的是“等一段时间”，并不返回什么结果值，所以写 `Promise<void>`。

<br/>

**✅ 3.`(resolve) => setTimeout(resolve, ms)` 是什么意思？**

这段代码里是构造 Promise 的关键：

```ts
new Promise((resolve) => {
  setTimeout(resolve, ms);
});
```

<br/>

- **解释：**

	* `new Promise(...)` 需要一个函数作为参数，这个函数会在 Promise 执行时被调用
	* 这个函数有两个参数：`resolve` 和 `reject`（只用了 `resolve`）
	* 你传进去的函数是 `(resolve) => { setTimeout(resolve, ms) }`
	* `setTimeout(resolve, ms)` 的意思是：**等 `ms` 毫秒后调用 `resolve()`**，表示 Promise 成功完成

> 这个技巧常用于「等待一段时间」的场景，比如节流、防抖、动画延迟、等待加载等。


<br/>


**✅ 补充一个完整使用场景：**

```ts
async function main() {
  console.log('开始等待');
  await delay(2000);  // 等 2 秒
  console.log('2 秒过去了');
}
```

<br/>

输出：

```
开始等待
（2 秒后）
2 秒过去了
```

你可以把 `delay` 当成一个通用的“延迟函数”。


<br/><br/><br/>

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

># <h1 id="打印Console">[打印Console](https://www.hangge.com/blog/cache/detail_1735.html)</h1>


<br/>

> <h2 id="类型信息输出">类型信息输出</h2>

- 信息清除

&emsp; 如果想要清空控制台里的日志信息，除了可以点击控制台面板左上角的清空按钮，还可以在 js 代码中执行如下命令：

```
console.clear()
```


- console.log：用于输出普通信息

- console.info：用于输出提示性信息

- console.warn：用于输出告警信息

- console.error：用于输出错误信息




<br/>

> <h2 id="格式化输出">格式化输出</h2>

**具体支持如下四种占位符：**

- 字符（%s）

- 整数（%d 或 %i）

- 浮点数（%f）

- 对象（%o）

```
console.log("%d年%d月%d日",2017,7,26);
console.log("圆周率是%f",3.1415926);
var dog = {};
dog.name = "大黄";
dog.color = "黄色";
console.log("这是我家的小狗：%o", dog);
```


<br/>

> <h2 id=""></h2>




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

