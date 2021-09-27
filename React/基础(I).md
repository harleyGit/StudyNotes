> <h1 id=""></h1>
- [**基本用法**](#基本用法)
	- [生命周期方法](#生命周期方法) 
	- [构造函数constructor](#构造函数constructor)
	- [参数传递](#参数传递)
		- [父->子传参](#父子传参)
		- [子->父组件的通信](#子父组件的通信)
		- [兄弟节点之间的通信](#兄弟节点之间的通信)
		- [订阅模型](#订阅模型)
- [**ES6基础**](#ES6基础)
- [**顶层API**](#顶层API)
	- [createElement](#createElement)
	- [cloneElement](#cloneElement)
- [**性能优化**](#性能优化)
	- [值是否为空或有值](#值是否为空或有值) 
- [JavaScript语法](#JavaScript语法)
	- [数组splice()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)
- **参考资料：**
	- [**JavaScript优秀教程**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
	- [Material-UI React组件库](https://v4-2-1.material-ui.com/zh/getting-started/installation/)
	- [React生命周期](https://www.jianshu.com/p/c9bc994933d5)
	- [hook](https://juejin.cn/post/6844903999083118606)



<br/>

***
<br/>


># <h1 id="基本用法">基本用法</h1>


<br/>

>## <h2 id="生命周期方法">[生命周期方法](https://www.runoob.com/react/react-component-life-cycle.html)</h2>



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
