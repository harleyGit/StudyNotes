> <h2 id=""></h2>
- [**基本用法**](#基本用法)
	- [ReactDOM](#ReactDOM)
		- [ReactDom.render](#ReactDom.render)
	- [React视图渲染](#React视图渲染)
	- [项目文件描述](#项目文件描述)
		- [import和require](#import和require)
	- [生命周期方法](#生命周期方法) 
	- [构造函数constructor](#构造函数constructor)
	- [React Hooks](#ReactHooks)
		- [useState](#useState)
		- [useEffect](#useEffect)
	- [React Router](#ReactRouter)
		- [路由Demo](#路由Demo)
	- [参数传递](#参数传递)
		- [父->子传参](#父子传参)
		- [子->父组件的通信](#子父组件的通信)
		- [兄弟节点之间的通信](#兄弟节点之间的通信)
		- [订阅模型](#订阅模型)
- [**ES6基础**](#ES6基础)
	- [异步编程](#异步编程)
- [**顶层API**](#顶层API)
	- [createElement](#createElement)
	- [cloneElement](#cloneElement)
- [**性能优化**](#性能优化)
	- [值是否为空或有值](#值是否为空或有值) 
- [JavaScript语法](#JavaScript语法)
	- [数组splice()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)
- [**创建到打包**](#创建到打包)
- **参考资料：**
	- [**JavaScript优秀教程**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
	- [Material-UI React组件库](https://v4-2-1.material-ui.com/zh/getting-started/installation/)
	- [React生命周期](https://www.jianshu.com/p/c9bc994933d5)
	- [hook](https://juejin.cn/post/6844903999083118606)



<br/>

***
<br/>


![react31](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react31.png)




># <h1 id="基本用法">基本用法</h1>


<br/>


> <h2 id="ReactDOM">React DOM</h2>

- react.js是React的核心文件，如组件、Hooks、虚拟DOM等，都在这个文件中。
- react-dom.js则是对真实DOM的相关操作，如将虚拟DOM渲染到真实DOM里，或者从真实DOM中获取节点。


<br/>

**介绍：**ReactDOM对象是react-dom.js提供的一个用于进行DOM操作的对象，在该对象下有一系列API用于操作DOM。在React中需要和真实的DOM打交道时都需要通过ReactDOM的API进行。当然也可以使用一些原生DOM的API，但并不推荐这么做。



<br/>

> <h3 id='ReactDom.render'>ReactDom.render</h3>

```
ReactDOM.render(element, container[, callback])
```

- render方法是ReactDOM在开发时唯一常用的API。render方法用于将React生成的虚拟DOM生成到真实的DOM中去。
- element是React生成的虚拟DOM，也叫作ReactElement或ReactNode。除此之外也可以使用字符串去实现。
- element要放置在container的容器中，它必须是一个已经存在的真实DOM节点。
- callback是将ReactNode渲染到container之后的回调函数。


**index.js文件**
```
ReactDOM.render(
  // <h1>Hello World</h1>,
  'Hello React',
  document.getElementById('root'),
  () => {
    console.log('渲染完成了')
  }
);
```

效果：

![react32](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react32.png)


这里将“Hello React”这段字符串渲染到了#root这个div中，当然也可以利用React-Node做更复杂的结构渲染。

render方法通常用来渲染整个项目的根组件，其他组件都在根组件中一层层调用。在使用render方法时要注意container中如果有其他子内容都会被替换掉。另外render方法并不会修改container的其他特性，只是修改container的内容。


<br/>

> <h3 id=''></h3>



<br/>
<br/>


> <h2 id="React视图渲染">React视图渲染</h2>




<br/>

> <h3 id='React.createElement'>React.createElement</h3>


当需要用React创建虚拟DOM时，React专门提供了一个方法createElement()。注意该方法并非是原生DOM中的createElement。

```
React.createElement(type, config, children)
```


**具体参数如下:**

1）type要创建的标签类型。如要创建的是个div标签，则写React.createElement（＂div＂），一定注意type的类型是一个字符串。

2）congfig参数是设置生成的节点的相关属性，这里要注意congfig的类型是一个纯对象;

3）children代表该元素的内容或者子元素,可以放字符串，数组等

示例如下：

```
let h1 = React.createElement('h1', null, 'Hello React')
let p = React.createElement('p', null, '欢迎吃🍎')
React.createElement("h1", {
    id: "title",
    className: "title",
    title: "港青",
    style: {
        width: '100px',
        height: '100px'
    }
}, [h1, p])
```

<br/>

> <h3 id=''></h3>





<br/>
<br/>


> <h2 id="项目文件描述">项目文件描述</h2>


![react33](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react33.png)

- node_modules，在项目中安装的依赖都会放在这个文件夹下

- package.json，是整个项目的描述文件，里边有两块内容这里详细说一下。

	- ①dependencies项目安装的依赖名称及版本信息。可以看到在构建完的项目中，已经帮开发者安装好了一些基本的依赖：＂react＂:＂^16.13.1＂，＂react-dom＂:＂^16.13.1＂，＂react-scripts＂:＂3.4.1＂。react和react-dom不需要再复述了。react-scripts是什么？create-react-app会把webpack、Babel、ESLint配置好合并在一个包里，方便开发人员使用，这个包就是react-scripts。
	
	- ②scripts中定义的是在命令行工具中可以使用到的一些命令。在当前目录my-app中，启动命令行工具，一起来测试一下这些命令

- public文件夹。用来存放html模板。public文件夹中的index.html就是项目的html模板，不建议读者修改名字，否则需要重新配置html文件。
- src文件夹。该文件夹中index.js是整个项目的入口文件。为了加快重新构建的速度，webpack只处理src中的文件。注意要将JS和CSS文件放在src中，否则该文件不会被webpack打包。


<br/>


> React.StrictMode

&emsp; StrictMode是用来检查项目中是否有潜在风险的检测工具，类似于JavaScript中的严格模式。StrictMode跟Fragment类似，不会渲染任何真实的DOM。只是为后代元素触发额外的检查和警告。

&emsp; StrictMode可以在代码中的任意地方使用，当然也可以直接用在index.js中，开启全局检测。除上述描述的特征外，StrictMode检查只在开发模式下运行，不会与生产模式冲突。

<br/>

**检测的项目如下：**

1）识别具有不安全生命周期的组件。

2）有关旧式字符串ref用法的警告。

3）关于已弃用的findDOMNode用法的警告。

4）检测意外的副作用。

5）检测遗留的context API。


在StrictMode模式下，如果检测到代码有以上问题，React会在控制台中打印出相应的警告。



<br/>
<br/>
<br/>

> <h3 id='import和require'>import和require</h3>

- **调用时间**
	- require 是运行时调用，所以理论上可以运作在代码的任何地方
	- import 是编译时调用，所以必须放在文件的开头

<br/>

- **本质区别：**
	- require 是赋值过程，其实require的结果就是对象、数字、字符串、函数等，再把结果赋值给某个变量。它是普通的值拷贝传递。

	- import 是解构过程。使用import导入模块的属性或者方法是引用传递。且import是read-only的，值是单向传递的。default是ES6 模块化所独有的关键字，export default {} 输出默认的接口对象，如果没有命名，则在import时可以自定义一个名称用来关联这个对象

<br/>

**require用法展示**


&emsp; 在导出的文件中使用module.exports对模块中的数据导出，内容类型可以是字符串，变量，对象，方法等不予限定。使用require()引入到需要的文件中即可

&emsp; 在模块中，将所要导出的数据存放在module的export属性中，在经过CommonJs/AMD规范的处理，在需要的页面中使用require指定到该模块，即可导出模块中的export属性并执行赋值操作（值拷贝）

```
// module.js
module.exports = {
    a: function() {
        console.log('exports from module');
    }
}
```

```
// sample.js
var obj = require('./module.js');
obj.a()  // exports from module
```

&emsp; 当我们不需要导出模块中的全部数据时，使用大括号包含所需要的模块内容

```
// module.js
function test(str) {
  console.log(str); 
}
module.exports = {
 test
}
```


```
// sample.js
let { test } =  require('./module.js');
test ('this is a test');
```



<br/>

**import 的基本语法**

&emsp; 使用import导出的值与模块中的值始终保持一致，即引用拷贝，采用ES6中解构赋值的语法，import配合export结合使用

```
// module.js
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

> <h3 id=''></h3>






<br/>
<br/>

>## <h2 id="生命周期方法">[生命周期方法](https://www.runoob.com/react/react-component-life-cycle.html)</h2>


![37](![react36](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react37.png))

- **componentWillMount** 在渲染前调用,在客户端也在服务端。

- **componentDidMount :** 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

- **componentWillReceiveProps：** 这个方法在初始化render时不会被调用，在组件接收到一个新的 prop (更新后)时被调用。一般用于父组件状态更新时子组件的重新渲染。这个东西十分好用，但是一旦用错也会造成十分严重的后果。

- **shouldComponentUpdate** 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。可以在你确认不需要更新组件时使用。

- **componentWillUpdate**在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。

- **componentDidUpdate** 在组件完成更新后立即调用。在初始化时不会被调用。
- **componentWillUnmount**在组件从 DOM 中移除之前立刻被调用。


<br/>

- 组件的构造

```
import React,{ Component } from 'react';

class Demo extends Component {
  constructor(props,context) {
      super(props,context)
      this.state = {
          //定义state
      }
  }
componentWillMount () {}

componentDidMount () {}

//这种方式十分适合父子组件的互动，通常是父组件需要通过某些状态控制子组件渲染亦或销毁...
componentWillReceiveProps (nextProps) {//componentWillReceiveProps方法中第一个参数代表即将传入的新的Props
	if (this.props.sharecard_show !== nextProps.sharecard_show){
        //在这里我们仍可以通过this.props来获取旧的外部状态
        //通过新旧状态的对比，来决定是否进行其他方法
        if (nextProps.sharecard_show){
            this.handleGetCard();
        }
}

shouldComponentUpdate (nextProps,nextState) {}

componentWillUpdate (nextProps,nextState) {}

componentDidUpdate (prevProps,prevState) {}

render () {
    return (
        <div></div>
    )
}

componentWillUnmount () {}
}

export default Demo;
```


<br/>
<br/>

**测试Demo：**

```

class Button extends React.Component {
  constructor(props) {
      super(props);
      this.state = {data: 0};
      this.setNewNumber = this.setNewNumber.bind(this);
  }
  
  setNewNumber() {
    this.setState({data: this.state.data + 1})
  }
  render() {
      return (
         <div>
            <button onClick = {this.setNewNumber}>INCREMENT</button>
            <Content myNumber = {this.state.data}></Content>
         </div>
      );
    }
}
 
 
class Content extends React.Component {
  componentWillMount() {
      console.log('Component WILL MOUNT!')
  }
  componentDidMount() {
       console.log('Component DID MOUNT!')
  }
  componentWillReceiveProps(newProps) {
        console.log('Component WILL RECEIVE PROPS!')
  }
  shouldComponentUpdate(newProps, newState) {
        return true;
  }
  componentWillUpdate(nextProps, nextState) {
        console.log('Component WILL UPDATE!');
  }
  componentDidUpdate(prevProps, prevState) {
        console.log('Component DID UPDATE!')
  }
  componentWillUnmount() {
         console.log('Component WILL UNMOUNT!')
  }
 
    render() {
      return (
        <div>
          <h3>{this.props.myNumber}</h3>
        </div>
      );
    }
}
ReactDOM.render(
   <div>
      <Button />
   </div>,
  document.getElementById('example')
);

```


效果图：

![测试生命周期](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react7.png)


控制台打印：

```
Component WILL MOUNT!
Component DID MOUNT!
```


点击按钮进行+1，控制台打印：

```
Component WILL RECEIVE PROPS!
Component WILL UPDATE!
Component DID UPDATE!
```

<br/>
<br/>





> <h2 id="构造函数constructor">构造函数constructor</h2>

```
Constructor(props){  
     super(props);  
}  
```

- 在React中，构造函数主要用于两个目的：
	- 它用于通过向this.state分配对象来初始化组件的本地状态。
	- 它用于绑定组件中出现的事件处理程序方法。

**注意：** 如果你的React组件既不初始化状态，也不绑定方法，那么就不需要实现React组件的构造函数。

&emsp; 不能在构造函数()中直接调用setState()方法。如果组件需要使用本地状态，则需要直接使用’this.state ‘，在构造函数中分配初始状态。构造函数只使用这个来分配初始状态，所有其他方法都需要使用set.state()方法。

App.js

```
import React, { Component } from 'react';  
  
class App extends Component {  
  constructor(props){  
    super(props);  
    this.state = {  
         data: 'www.srcmini.com'  
      }  
    this.handleEvent = this.handleEvent.bind(this);  
  }  
  handleEvent(){  
    console.log(this.props);  
  }  
  render() {  
    return (  
      <div className="App">  
    <h2>React构造函数例子</h2>  
    <input type ="text" value={this.state.data} />  
        <button onClick={this.handleEvent}>点击</button>  
      </div>  
    );  
  }  
}  
export default App;  
```

main.js

```
import React from 'react';  
import ReactDOM from 'react-dom';  
import App from './App.js';  
  
ReactDOM.render(<App />, document.getElementById('app'));  
```



<br/>

> 有必要在构造函数中调用super()吗？

&emsp; 是的，有必要在构造函数中调用super()。如果需要在组件的构造函数中设置属性或访问’this’，则需要调用super()。

```
class App extends Component {  
    constructor(props){  
        this.fName = "Jhon"; // this'在super()之前是不允许的  
    }  
    render () {  
        return (  
            <p> Name: { this.props.name }</p>  
        );  
    }  
}  
```

&emsp; 当你运行上面的代码时，会得到一个错误，在super()之前不允许使用’this’。因此，如果需要访问构造函数中的道具，需要调用super(props)。



<br/>

> 箭头函数

&emsp;  箭头函数是ES6标准的新特性。如果需要使用箭头函数，则没有必要将任何事件绑定到’this ‘。在这里，“this”的范围是全局的，不局限于任何调用函数。因此，如果你使用的是箭头函数，就不需要在构造函数中绑定’this’。


```
import React, { Component } from 'react';  
  
class App extends Component {  
  constructor(props){  
    super(props);  
    this.state = {  
         data: 'www.srcmini.com'  
      }  
  }  
  handleEvent = () => {  
    console.log(this.props);  
  }  
  render() {  
    return (  
      <div className="App">  
    <h2>React构造函数例子</h2>  
    <input type ="text" value={this.state.data} />  
        <button onClick={this.handleEvent}>点击</button>  
      </div>  
    );  
  }  
}  
export default App;  
```




<br/>
<br/>
<br/>

> <h2 id='ReactHooks'>React Hooks</h2>


<br/>
<br/>

> <h3 id='useState'>useState</h3>


```
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];

// 简化
const [state, setState] = useState(initialState)
```

&emsp; 调用该方法时传入state的初始值，该方法会返回一个数组，数组的第0位是state具体的值，而第1位是修改该state的方法，同类组件的setState方法一样，调用该方法会更新state，然后引起视图更新。


**HOC.js**

```
export function TestHOC1() {
    const [name, setName] = useState('kkb')
    return <div>
        <p>{name}</p>
        <button
            onClick={() => {
                setName('开课吧')
            }}>
            显示全称
        </button>
    </div>
}
```


效果图: 

![react34](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react34.png)

**在使用useState时，有三个问题要注意:**

1）useState返回的setState方法同类组件的setState一样，也是一个异步方法，需要组件更新之后state的值才会变成新值;

2）useState返回的setState并不具有类组件的setState合并多个state的作用，如果state中有个多state，在更新时，其他值一同更新.



```
export function TestHOC2() {
    const [data, setData] = useState({ name: 'kkb', age: 10 })
    return <div>
        <p>{data.name}</p>
        <button
            onClick={() => {
                setData({
                    ...data, name: '开课吧'
                })

            }}>
            显示全称
        </button>
    </div >
}
```

效果图: 

![react35](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react35.png)








<br/>
<br/>

> <h3 id='useEffect'>useEffect</h3>


&emsp; Effect翻译成专业术语称之为副作用。网络请求、DOM操作都是副作用的一种，useEffect就是专门用来处理副作用的。在类组件中副作用通常在componentDid-Mount和componentDidUpdate中进行处理，而useEffect就相当于componentDidMount、componentDidUpdate和componentWillUnmount的集合体。useEffect包括两个参数执行时的回调函数和依赖参数，并且回调函数还有一个返回函数


```
function TestHOC3_1() {
    const [course, setCourse] = useState('Web高级工程师')
    const [num, setNum] = useState(1)

    useEffect(() => {
        console.log('✈️组件挂载或更新')
        return () => {
            console.log('✈️清理更新前的一些全局类容， 或检测组件即将卸载')
        };
    }, [num]) // 只有num更新时才会执行回调函数
					    // 一定注意依赖参数要传入一个空数组，不传的话组件的任何更新都会调用该副作用


    return <div>
        <div>
            选择课程：
            <select
                value={course}
                onChange={({ target }) => { setCourse(target.value) }}
            >
                <option value='Web 全栈工程师'>Web 全栈工程师</option>
                <option value='Web 高级工程师'>Web 高级工程师</option>

            </select>
        </div>
        <div>
            购买数量：
            <input
                type='number'
                value={num}
                min={1}
                onChange={({ target }) => { setNum(target.value) }}
            >

            </input>
        </div>
    </div>
}

export function TestHOC3() {
    const [show, setShow] = useState(true)

    return <div>
        {show ? <TestHOC3_1 /> : ''}
        <button onClick={() => {
            setShow(!show)
        }}>
            {show ? '隐藏课程' : '显示课程'}
        </button>
    </div>
}


```



效果图: 

![react36](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react36.png)


依赖参数，其本身是一个数组，在数组中放入要依赖的数据，当这些数据有更新时，就会执行回调函数。整个组件的生命周期过程如下：

组件挂载→执行副作用（回调函数）→组件更新→执行清理函数（返还函数）→执行副作用（回调函数）→组件准备卸载→执行清理函数（返还函数）→组件卸载。

上文讲过useEffect是componentDidMount、componentDidUpdate和componentWillUnmount的集合体，如果单纯只想要在挂载后、更新后、卸载前其中之一的阶段执行，可以参考以下操作。

①componentDidMount。如果只想要在挂载后执行，可以把依赖参数置为空，这样在更新时就不会执行该副作用了。

②componentWillUnmount。如果只想要在卸载前执行，同样把依赖参数置为空，该副作用的返还函数就会在卸载前执行。

③componentDidUpdate。只检测更新相对比较麻烦，需要区分更新还是挂载需要检测依赖数据和初始值是否一致，如果当前的数据和初始数据保持一致就说明是挂载阶段，当然安全起见应和上一次的值进行对比，若当前的依赖数据和上一次的依赖数据完全一样，则说明组件没有更新


<br/>


> Hooks的使用规则


&emsp； Hooks使用规则了解Hooks的使用之后，还需要具体了解一下Hooks的使用规则，主要为以下两点：

1）只能在函数式组件和自定义Hooks之中调用Hooks，普通函数或者类组件中不能使用Hooks；

2）只能在函数的第一层调用Hooks。如果函数中还嵌套了流程控制语句如if或者for，这些地方是不能再调用Hooks的，当然函数中嵌套了子函数，子函数中也一样不能使用Hooks。Hooks的设计极度依赖其定义时候的顺序，如组件更新时Hooks的调用顺序变了，就会出现不可预知的问题。Hooks的使用则是为了保证Hooks调用顺序的稳定性。为此React提供一个ESLint plugin来做静态代码检测。eslint-plugin-react-hooks新版的脚手架中，也内置了这套检测。当代码中的Hooks使用不符合上述规范时，在开发环境中会有错误警告。

**注意：**

&emsp； 在使用自定义Hook时，同样需要遵守Hooks的使用规则，另外注意React要求自定义Hook的命名也必须以use开始，以区别于其他函数。


<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>
<br/>

> <h2 id='ReactRouter'>React Router</h2>

&emsp; SPA（Single Page Application，单页面应用），整个应用只加载一个页面（入口页面），后续在与用户的交互过程中，通过DOM操作在这个单页上动态生成结构和内容。使用SPA可以使项目有更好的用户体验，减少请求、渲染和页面跳转产生的等待与空白，另外前端在项目中更具重要性，数据和页面内容都由异步请求（AJAX）+DOM操作来完成，前端则更多地去处理业务逻辑。

&emsp; React-Router就是一套前端路由库，在项目中安装好React-Router之后，开发者就可以基于React开发单页面多视图的应用。

&emsp; React Router Dom 就只是基於 React Route 更進階的套件，核心都還是 React Route，相同地，react-router-native 也是基於 React Route 更進階的套件。


React-Router的安装命令如下：npm i react-router-dom

&emsp; React Router Dom 其實只是多了四個 React Component：BrowserRouter、 HashRouter、Link、NavLink。


WEB端的Router中提供了两种不同的模式**＜HashRouter/＞**和**＜BrowserRouter/＞**:

- HashRouter是基于hash实现的一种路由方式，URL变化时主要是hash值进行变化。如http://127.0.0.1:3000/#/about、http://127.0.0.1:3000/#/user，可以看到URL里一定会有一个#号，也就是hash标识。hash模式的好处是一定不会向服务端发送请求，但URL里一定会有一个#号。
- BrowserRouter则是基于H5 history API的一种路由方式。history的URL切换基于history提供的pushState方法，好处是URL和之前直接请求后端的URL没有什么区别，如http://127.0.0.1:3000/about、http://127.0.0.1:3000/user。当然问题也同样突出，在部署线上时，要注意直接输入URL还是会发起后端请求，所以后端一样也要做处理。


&emsp; 在实际开发时，不管采取哪种路由模式，都需要在项目最外层配置好路由，告诉React该项目使用的是哪种路由，具体代码如下：

**index.js**

```
import { BrowserRouter as Router, HashRouter } from 'react-router-dom';

ReactDOM.render(
  <React.StrictMode>
    <Router>
      <App />
    </Router>
  </React.StrictMode>,
  document.getElementById('root')
);
```


<br/>

**App.js**

```
render() {
    return <TestHOC4 />
}
```


<br/>

**HOC.js**

```
export function TestHOC4() {

    return <div style={{ width: '100%', height: '100%' }}>
        <React.Fragment>
            {/* <Link to='/about'>数字</Link> */}
            <Route path="/about" component={About} />
        </React.Fragment>
    </div>
}

export function TestHOC4_1() {
    return <React.Fragment>
        <Link to='/about'>首页</Link>
        <Route path='/about' render={() => <div>1234567890</div>} />
    </React.Fragment>
}
```

效果图：

![38](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react38.png)

&emsp; 如果使用的是HashRouter，则把BrowserRouter替换成HashRouter即可。


<br/>
<br/>

> <h3 id='路由Demo'>路由Demo</h3>

**RouterAbout.js**

```
export default function RouterAboutView() {
    return <div>
        <h1>关于视图</h1>
        <p>关于视图内容</p>
    </div>
}
```

<br/>

**RouterIndexView.js**

```
export default function RouterIndexView() {
    return <div>
        <h1>首页视图</h1>
        <p>首页视图内容</p>
    </div>
}
```


<br/>

**RouterListView.js**

```
export default function RouterListView() {
    const pageLength = Math.ceil(Data.length / 3)

    let { page = 1 } = useParams()
    return <div>
        <h1>列表视图</h1>
        <p>列表视图内容</p>
        <List activePage={page} />
        <Pagation activePage={page}
            pageLength={pageLength}
        />
    </div>
}

const pageLen = 3 //每页多少条
export function List(props) {
    // 接受父级传递的props，props中必须包含activePage
    // activePage代表当前实现第几页
    let { activePage } = props
    // 从当前页第几条开始，注意页码从1开始计数，但是js从0开始计数，所以减1
    let start = (activePage - 1) * pageLen
    let end = activePage * pageLen
    // 当前页到第几条结束
    let nowData = Data.filter((item, index) => index >= start && index < end)

    return <ul>
        {nowData.map(item => {
            return (<li key={item.id}>
                <h2>{item.title}</h2>
                <p>{item.describe}</p>
            </li>)
        })}
    </ul>
}
```



<br/>

**RouterRouter.js**

```
let Routers = [
    {
        path: '/',
        exact: true,
        render(props) {
            return <RouterIndexView {...props} />
        }
    },
    {
        path: '/about',
        exact: true,
        render(props) {
            return <RouterAboutView {...props} />
        }
    },
    {
        path: ['/list', '/list/:page'],
        exact: true,
        render(props) {
            let { page = 1 } = props.match.params
            if (page >= 1) {// 解析页码，如果没有传递页面则设置默认值为1
                // 若/list/a等不是数字的情况下，则显示404视图
                return <RouterListView {...props} />
            }
            return <RouterUndefinedView {...props} />
        }
    },
    {
        path: '',
        exact: false,
        render(props) {
            return <RouterUndefinedView {...props} />
        }
    }
]


let Navs = [
    {
        to: '/',
        exact: true,
        title: '首页'
    },
    {
        to: '/about',
        exact: true,
        title: '关于'
    },
    {
        to: '/list',
        title: '课程列表',
        isActive(url) {
            let urlData = url.split('/')
            if (url === '/list' || (urlData.length === 3 && urlData[1] === 'list' && urlData[2] > 0)) {
                // 判断url为‘/list’或‘/list/大于1的数字时’，选中当前项，否则不选中
                return true
            }
            return false
        }
    },

]

export { Routers, Navs }



// 🍎<==========================================>🍎

export function Nav() {
    let { pathname } = useLocation()

    return <nav>
        <span> | </span>
        {
            Navs.map(item => {
                return <Fragment key={item.to}>
                    <NavLink
                        to={item.to}
                        exact={item.exact}
                        isActive={item.isActive ? () => {
                            return item.isActive(pathname)
                        } : null
                        }
                        activeStyle={{
                            color: 'red'
                        }}
                    >
                        {item.title}
                        <span> | </span>
                    </NavLink>
                </Fragment>
            })
        }
    </nav>
}


// 🍎<==========================================>🍎

/**
 * @description: 翻页导航组件
 */
export function Pagation(props) {
    // activePage   当前第几页， pageLength总共多少页
    let { activePage, pageLength } = props
    console.log(activePage)

    return <nav>
        {
            [...('.'.repeat(pageLength))].map((item, index) => {
                index++
                return <Fragment key={index}>
                    <span> | </span>
                    <Link
                        to={'/list/' + index}
                        style={{
                            color: activePage == index ? 'red' : '#000'
                        }}>
                        {index}
                    </Link>
                </Fragment>
            })
        }
        <span> | </span>
    </nav>
}




// 🍎<==========================================>🍎


/**
 * @description: 列表数据
 */

export const Data = [{
    id: 0,
    title: ' iOS 架构师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 1,
    title: ' Web 架构师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 2,
    title: ' Go 架构师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 3,
    title: ' Go 金牌就业班',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 4,
    title: ' iOS 架构师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 5,
    title: ' 百万架构师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
}, {
    id: 6,
    title: ' JavaEE 金牌就业班',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 7,
    title: ' 数据分析全栈工程师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 8,
    title: ' Android 开发师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
},
{
    id: 9,
    title: '测试师',
    describe: '授课深度对标阿里 P6等级，进入BAT等一线大厂，成为大厂进阶稀缺人才'
}
]

```


<br/>

**RouterUndefinedView.js**

```
export default function RouterUndefinedView() {
    return <div>
        <h1>404视图</h1>
        <p>404视图内容</p>
    </div>
}
```

调用：

```
/**
   * @description: 路由导航综合练习
   * 导航组件
   */
  _testRouter2 = () => {
    return <Fragment>
      <Nav />
      <Switch>
        {Routers.map(item => {
          return <Route
            key={item.path}
            path={item.path}
            exact={item.exact}
            render={item.render} />
        })}
      </Switch>
    </Fragment>
  }
```

效果图：

![39](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react39.png)


<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>

> <h3 id=''></h3>



<br/>
<br/>

> <h3 id=''></h3>




<br/>
<br/>
<br/>

> <h2 id="参数传递">参数传递</h2>

<br>

> <h3 id="父子传参">父->子传参</h3>

&emsp; 父组件通过props将数据传递给子组件，子组件通过this.props.属性名调用


![Code](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react9.gif)

<br/>

效果：

![效果](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react10.png)



<br>
<br>

> <h3 id="子父组件的通信">子->父组件的通信</h3>

- 父组件通过props传递一个修改自身数据state的方法给子组件

- 子组件调用父组件传递过来的方法，传递参数，修改父组件的属性。相当于子组件调用父组件的方法来更新父组件的数据，或者向父组件传递数据


![Code](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react11.png)

<br/>

![效果](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react12.gif)




<br>
<br>

> <h3 id="兄弟节点之间的通信">兄弟节点之间的通信</h3>

&emsp; 在一个兄弟节点中修改父组件的数据，然后父组件会同步到另一个需要通信的子组件，就完成了一次通信

![Code](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react13.png)




<br>
<br>

> <h3 id="订阅模型">订阅模型</h3>

- addEventListener 就是一个发布订阅模型

- 订阅：element.addEventListener('click',callback)

- 发布：当绑定元素的click事件被触发的时候，就会执行发布流程，调用订阅绑定的callback


**原理图解：**

![模型图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react14.png)

<br/>

![模型图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react15.png)





<br/>

***
<br/>

> <h1 id="ES6基础">ES6基础</h1>

<br/>

> <h2 id='异步编程'>异步编程</h2>

&emsp; JavaScript引擎是基于事件循环的概念实现的，JavaScript引擎会把任务放在一个任务队列中，通过事件循环机制一一执行任务队列里的任务，从第一个依次执行到最后一个。有些任务执行可能时间会比较长，如果等待时间比较长的任务执行完成之后再执行下一个任务就会影响用户体验，所以JavaScript在设计的时候就有了同步和异步。异步任务不进入主线程，而进入任务队列中的任务，只有任务队列通知主线程，某个异步任务可以执行了，这个任务才会进入主线程执行。异步任务在ES5标准中通过回调来解决执行顺序问题.

```
/**
* @description: 异步编程
*/
_asynchronousTest = () => {
	asyncFn(function () {
	  console.log('执行完之后的回调打印。。。。。')
	})
}

function asyncFn(cb) {
  setTimeout(() => {
    console.log('🍎 异步逻辑')
    cb && cb()
  }, 1000);
}


```

输出：

```
🍎 异步逻辑
node_modules/vconsole/dist/vconsole.min.js:10
执行完之后的回调打印。。。。。
node_modules/vconsole/dist/vconsole.min.js:10
```

上述只是模拟异步过程，但是回调函数多了容易嵌套造成“回调地狱”，造成代码阅读困难。所以引入Promise语法。


<br/>

> **Promise基本语法**

&emsp; 首先Promise是系统中预定义的类，通过实例化可以得到Promise对象。Promise对象会有三种状态，分别是pending、resolved、rejected

```
_asynchronousTest1 = () => {
    let p1 = new Promise(function () {

    })
    console.log('🍎 p1：', p1)

    let p2 = new Promise(function (resolve, reject) {
      resolve('success...')
    })
    console.log('🍎 p2：', p2)


    let p3 = new Promise(function (resolve, reject) {
      resolve('reject...')
    })
    console.log('🍎 p3：', p3)

}
```

输出：

```
🍎 p1： Promise {[[PromiseState]]: 'pending', [[PromiseResult]]: undefined}
🍎 p2： Promise {[[PromiseState]]: 'fulfilled', [[PromiseResult]]: 'success...'}
🍎 p3： Promise {[[PromiseState]]: 'fulfilled', [[PromiseResult]]: 'reject...'}
```


&emsp; 上述代码中Promise回调函数里如果没有调取resolve或者reject，那么就会返还一个Pending状态的Promise对象。如果调取了resolve函数就会返还一个resolved状态的Promise对象（在火狐上略有不同，会显示fullfilled状态的Promise对象）。如果调取了reject函数则会返还一个rejected状态的对象。每一个Promise对象都会有一个then方法，then方法里会接收两个参数（可选）.


<br/>

&emsp; 如下代码在执行then的时候会执行到第一个成功的回调中去，如果调取reject函数则会执行到then的第二个错误的回调中去。当然Promise也提供catch方法来捕捉reject错误

```
let p2 = new Promise(function (resolve, reject) {
      // resolve('success...')
      reject('失败')
    })
    console.log('🍎 p2：', p2)
    p2.then(res => {
      console.log('🍎 ♨️ res：', res)
    }, err => {
      console.log('💣 res：', err)
    })

```


输出:

```
🍎 p2： Promise {[[PromiseState]]: 'rejected', [[PromiseResult]]: '失败'}
💣 res： 失败
node_modules/vconsole/dist/vconsole.min.js:10
```



<br/>

> 






<br/>
<br/>

> <h2 id=''></h2>



<br/>
<br/>

> <h2 id=''></h2>






<br/>

***
<br/>

> <h1 id="顶层API">顶层API</h1>


<br/>

> <h2 id="createElement">[createElement](https://juejin.cn/post/6844903970876440583)</h2>




<br/>

> <h2 id="cloneElement">cloneElement</h2>

```
React.cloneElement(
 element,
 [props],
 [...children]
)

克隆原来的元素，返回一个新的 React 元素；
保留原始元素的 props，同时可以添加新的 props，两者进行浅合并；
key 和 ref 会被保留，因为它们本身也是 props ，所以也可以修改；
根据 react 的源码，我们可以从第三个参数开始定义任意多的子元素，如果定义了新的 children ，会替换原来的 children ；

第一个参数：react组件或者dom,这dom是真实的dom结构也可以是自定义的；

第二个参数：当前element的props、key、ref，也可以添加新的props；

第三个参数：是props.children，不指定默认展示我们调用时添加的子元素，如果指定会覆盖我们调用克隆组件时里面包含的元素。
```


<br/>

> **验证1:组件复制**

CloneElementTest.js文件

```
export function CloneDemo(props) {
    console.dir(props)
    console.table({ 'text: %s': props.children.props.children, 'keyValue': props.keyValue})
    return React.cloneElement(<div style={{ backgroundColor: 'red', display: "flex", justifyContent: "center", alignItems: "center" }} />,
        props
    )
}
export function ContainerBox() {
    return <CloneDemo keyValue={'CloneDemo的Key'}><h1>ContainerBox: 这是在父组件添加的元素</h1></CloneDemo>
}

```


Index.js文件

```
import { ContainerBox } from './Test/CloneElementTest';




ReactDOM.render(
  <React.StrictMode>
   {<ContainerBox />}
   
	</React.StrictMode>,
  document.getElementById('root')
);


reportWebVitals();



```

效果图：

![效果图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react16.png)



<br/>

> **验证2:传参数**

CloneElementTest.js

```

export function CloneDemo1({ dom = <div />, ...props }) {
    console.dir(props)
    console.table({ '🍎 text: ': props.children.props.children})

    return React.cloneElement(dom, { ...props })
}
export function ContainerBox1() {
    return <CloneDemo1 dom={<p></p>}><h1>这是在父组件添加的元素ContainerBox1</h1></CloneDemo1>
}
```

Index.js

```
//组件导入
import { ContainerBox1 } from './Test/CloneElementTest';




ReactDOM.render(
  <React.StrictMode>
  {<ContainerBox1/>}
  
  </React.StrictMode>,
  document.getElementById('root')
);



reportWebVitals();
```

![标签元素排列](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react17.png)



![props打印](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react18.png)





<br/>

> **验证2:传参数**




<br/>

> **验证2:传样式没有效果**

CloneElementTest.js

```
const Exam2 = (props) => <div>这是一个自定义的ReactElement元素{props.children}</div>
function CloneDemo2({ dom = <div />, ...rest }) {
    console.log('🍎 <<<<<<<<<<<<<<<')
    console.dir(dom)
    console.log('🍎 ===============')
    console.dir(rest)
    console.log('🍎 >>>>>>>>>>>>>>>')

    return React.cloneElement(dom, { ...rest })
}
export function ContainerBox2() {
    return <CloneDemo2 dom={<Exam2 style={{ color: "red", textAlign: "center" }} />}><h1>这是在父组件添加的元素</h1></CloneDemo2>
}

```


index.js

```
import { ContainerBox2 } from './Test/CloneElementTest';


ReactDOM.render(
  <React.StrictMode>
  {<ContainerBox2 />}

  </React.StrictMode>,
  document.getElementById('root')
);



reportWebVitals();
```


![控制台打印](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react19.png)

![标签显示](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react20.png)




<br/>

> **验证2:传样式有效果**


CloneElementTest.js

```
const Exam2_1 = (props) => {
    console.log('🍎 <<<<<<<<<<<<<<<')
    console.dir(props)
    return <div style={{ ...props.styles, ...props.style }}>这是一个自定义的ReactElement元素{props.children}</div>
}
function CloneDemo2_1({ dom = <div />, ...rest }) {
    const styles = {
        color: "blue",
        minWidth: "1200px",
        margin: "100px auto",
        textAlign: "left"
    }
    console.log('🍎 ===============')
    console.dir(dom)
    console.log('🍎 >>>>>>>>>>>>>>>')
    console.dir(rest)
    console.log('🍎 --------------->end')


    return React.cloneElement(dom, { styles, ...rest })
}
export function ContainerBox2_1() {
    return <CloneDemo2_1 dom={<Exam2_1 style={{ color: "red", textAlign: "center" }} />}><h1>这是在父组件添加的元素props.children</h1></CloneDemo2_1>
}
```


index.js

```
import { ContainerBox2_1 } from './Test/CloneElementTest';


ReactDOM.render(
  <React.StrictMode>
	{<ContainerBox2_1 />}


  </React.StrictMode>,
  document.getElementById('root')
);



reportWebVitals();

```

![标签元素展示](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react23.png)


![属性打印](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react22.png)




<br/>

> **验证2:样式优先级**



```
const Exam2_2 = (props) => {
    console.table({'🍎 Exam2_2 >>> props:%o': props,})
    return <div style={{ ...props.style }}>这是一个自定义的ReactElement元素{props.children}</div>
}
function CloneDemo2_2({ dom = <div />, ...rest }) {
    console.table({'🍊 CloneDemo2_2 >>> dom:%o': dom, 'rest: %o': rest})

    const styles = {
        color: "blue",
        minWidth: "1200px",
        margin: "100px auto",
        textAlign: "center"
    }
    return React.cloneElement(dom, {
        style: Object.assign({}, styles, dom.props.style), //将传入的样式放到最后提高他的优先级
        ...rest
    })
}
export function ContainerBox2_2() {
    return <CloneDemo2_2 dom={<Exam2_2 style={{ color: "red", textAlign: "center" }} />}><h1>这是在父组件添加的元素2_2</h1></CloneDemo2_2>
}
```



index.js

```
import { ContainerBox2_2 } from './Test/CloneElementTest';


ReactDOM.render(
  <React.StrictMode>
  {<ContainerBox2_2 />}


  </React.StrictMode>,
  document.getElementById('root')
);



reportWebVitals();




```


![标签元素展示](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react24.png)


![属性打印01](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react25.png)


![属性打印02](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react26.png)






<br/>

> <h2 id=""></h2>




<br/>

> <h2 id=""></h2>



<br/>

> <h2 id=""></h2>





<br/>

***
<br/>

> <h1 id="性能优化">性能优化</h1>

<br/>

> <h2 id="值是否为空或有值">值是否为空或有值</h2>

&emsp; 所以在判断是否为空前，应预判、确定数据的类型，如果期望类型不清晰，则可能会导致错误的判断或考虑情况不周全。

<br/>

- null、为定义、0判断

```
var a1 = null
var a2
var a3 = 0
Var a4= 1//或者 a4=-11

if (!a1) {
    console.log('a1 为null')
}

if (!a2) {
    console.log('a2 没有定义')
}

if (!a3) {
    console.log('a3 为0')
}

if (!a4) {
    console.log('a4 为0')
}
```

打印：

```
a1 为null
a2 没有定义
a3 为0
//a4 没有打印
```


<br/>

- a2没有定义判断

```
var a2

if (!a2) {
    console.log('a2 没有定义')
}

if (!a2.length) {
    console.log('a3 没有leng属性')
}

会报错：：
typeError: undefined is not an object(evaluating 'a2.length')
//因为a2没有定义，你又使用它的length属性，所以会报错



var a2
 if (!a2 || !a2.length) {
    console.log('a2 没有定义')
}    
打印： a2 没有定义

==================================   

var a2
  if (!a2.length) {
    console.log('a3 没有leng属性')
}

会报错：：
typeError: undefined is not an object(evaluating 'a2.length')

```




<br/>

&emsp; 确定数据类型后，然后根据不同的数据类型使用不同的方法来判断，例

```
function isEmpty(v) {
    switch (typeof v) {
    case 'undefined':
        return true;
    case 'string':
        if (v.replace(/(^[ \t\n\r]*)|([ \t\n\r]*$)/g, '').length == 0) return true;
        break;
    case 'boolean':
        if (!v) return true;
        break;
    case 'number':
        if (0 === v || isNaN(v)) return true;
        break;
    case 'object':
        if (null === v || v.length === 0) return true;
        for (var i in v) {
            return false;
        }
        return true;
    }
    return false;
}


```


测试案例：

```
isEmpty()              //true
isEmpty([])            //true
isEmpty({})            //true
isEmpty(0)             //true
isEmpty(Number("abc")) //true
isEmpty("")            //true
isEmpty("   ")         //true
isEmpty(false)         //true
isEmpty(null)          //true
isEmpty(undefined)     //true

```

**空值有：** undefined、 null、 ''、 NaN、false、0、[]、{} 、空白字符串，这些都返回true。



<br/>

***
<br/>

> <h1 id="JavaScript语法">JavaScript语法</h1>



<br/>

***
<br/>

> <h1 id="创建到打包">创建到打包</h1>

&emsp; 搭建项目使用的是 `create-react-app (已自动集成webpack相关配置)`官方提供的脚手架命令，目录结构如下：

![33](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react33.png)

asq: 为什么 没有webpack配置文件？

webpack的配置需要 输入 npm run eject 命令将所有内建的配置暴露出来。

create-react-app 已经为做了绝大部分事情，配好了webpack

现在就能使用 npm run start 开始写react 项目了

<br/>
<br/>

> <h2 id='打包'>打包</h2>

在package.json中又这么一段快捷指令，其是：

```
"scripts": {
    "build": "node scripts/build.js",
    "start": "node scripts/start.js",
    "test": "node scripts/test.js --env=jsdom",
  },
```

分别对应项目scripts下的文件：

![40](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react40.webp)

build文件就是打包项目打包的一系列配置.


运行 npm run build 后项目中会多出一个build的文件夹

![41](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react41.webp)


我们只需要把 build这个文件夹 放到云服务器上 即可






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
