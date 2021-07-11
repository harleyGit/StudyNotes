> <h1 id=""></h1>
- [**生命周期**](#生命周期)
	- [生命周期方法](#生命周期方法) 
- [**顶层API**](#顶层API)
- [**性能优化**](#性能优化)
- **参考资料：**
	- [**JavaScript优秀教程**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)



<br/>

***
<br/>


># <h1 id="生命周期">[生命周期](https://www.jianshu.com/p/c9bc994933d5)</h1>


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
