> <h1 id=""></h1>
- [**基本用法**](#基本用法)
	- [生命周期方法](#生命周期方法) 
	- [参数传递](#参数传递)
		- [父->子传参](#父子传参)
		- [子->父组件的通信](#子父组件的通信)
		- [兄弟节点之间的通信](#兄弟节点之间的通信)
		- [订阅模型](#订阅模型)
- [**顶层API**](#顶层API)
- [**性能优化**](#性能优化)
	- [值是否为空或有值](#值是否为空或有值) 
- **参考资料：**
	- [**JavaScript优秀教程**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
	- [Material-UI React组件库](https://v4-2-1.material-ui.com/zh/getting-started/installation/)
	- [React生命周期](https://www.jianshu.com/p/c9bc994933d5)



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

> <h1 id="顶层API">顶层API</h1>

[createElement](https://juejin.cn/post/6844903970876440583)





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
