> <h2 id=''></h2>
- [**React基础**](#React基础)
	- [**props**](#props)
	- [生命周期](#生命周期)
- [**高阶组件**](#高阶组件)





<br/>

***
<br/>


> <h1 id='React基础'>React基础</h1>

<br/>

> <h2 id='props'>props</h2>

子组件prop: 在React中prop有一个内置的属性children,它表示子组件的集合也就是组件的数量.


propTypes: 可以对prop中的属性进行约束,如下:

```
static propTypes = {
	classPrefix: React.PropTypes.string,
	activeIndex: React.PropTypes.number,
}

```



<br/>
<br/>



> <h2 id='生命周期'>生命周期</h2>

生命周期可以分为挂载、渲染、卸载,分成两类:
- 当组件在挂载或卸载时;
- 当组件接收新的数据时,即组件更新时;

```

Class App extends Component {
	
	/**
		* 生命周期方法
	**/
	//类型约束 外部访问: App.propTypes
	static propTypes = {
		//.. ...
	}
	
	// 默认props的属性
	static defaultProps = {
		// .. .. 
	}
	
	// 初始化时的方法
	constructor(props) {
		super(props)
		this.state = {
			// . . .. 	
		}
	}
	
	// 会在render方法之前执行, 组件初始化时运行一次
	componentWillMount() {
		// . ..
	}	
	
	
	// 会在render方法之后执行, 组件初始化时运行一次
	componentDidMount() {
		// ...
	}
	
	// 组件卸载, 执行一些清理方法,如事件回收或是清除定时器
	componentWillUnmount() {
		// . . ..
	}	
	
	
	// 渲染组件
	render() {
		return <div/>
	}
	
	
	
	
	
	/**
		* 数据更新方法
		* 若是state更新了,一次执行shouldComponentUpdate、componentWillUpdate、render、componentDidUpdate
	**/
	
	// 若是由父组件更新props造成子组件更新,则componentWillReceiveProps先执行,接着是shouldComponentUpdate......
	componentWillReceiveProps(nextProps) {
		// this.setState({})
	}
	
	// componentWillReceiveProps 的替代方法
	static getDerivedStateFromProps(nextProps, prevState) {

        console.log(' >> 🍎 nextProps: ', nextProps, '\n prevState:', prevState)

        // 该方法内禁止访问this
        // nextProps: 最近更新的nextProps, prevState: 没有更新的state
        if (nextProps.isAllCheckModeState !== prevState.isAllCheckModeState) {
            return {
                isAllCheckMode: nextProps.isAllCheckModeState
            }
        }

        // 不需要更新状态 返回null
        return null
    }
    
	
	// 可以加一些判断,让它需要时更新,不需时不更新
	shouldComponentUpdate(nextProps, nextState)() {
		// return true
	}
	
	componentWillUpdate(nextPorps, nextState) {
		//...
	}
	
	componentDidUpdate(prevProps, preState) {
		// . . .
	}
	
	
}
```


<br/>
<br/>



> <h2 id=''></h2>





<br/>
<br/>



> <h2 id=''></h2>









<br/>

***
<br/>



> <h1 id='高阶组件'>高阶组件</h1>


**App.js**

```
render() {
    const HightComponentTest = withPersisrentDataHOC(MyComponent)
    return <HightComponentTest />
}
```


**HighOrderComponent.js文件**

```
import React, { Component } from "react/cjs/react.production.min";


/**
 * @description: 高阶组件 localStorage 展示
 * @param {*} WrappedComponent
 * @return {*}
 */
export function withPersisrentDataHOC(WrappedComponent) {
    return class extends Component {
        componentWillMount() {
            localStorage.setItem('data', '高阶组件的使用测试 🚗 13245')

            let data = localStorage.getItem('data')
            this.setState({ data })
        }

        render() {
            // {...this.props} 把传递给当前组件的属性继续传递给被包装的组件
            return <WrappedComponent data={this.state.data} {...this.props} />
        }
    }
}

export class MyComponent extends Component {
    render() {
        return <div>{this.props.data}</div>
    }
}
```



效果：

![29](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react29.png)



<br/>
<br/>

> <h2 id='组件状态提升'>组件状态提升</h2>


&emsp; 无状态组件更容易被复用。高阶组件可以通过将被包装组件的状态及相应的状态处理方法提升到高阶组件自身内部实现被包装组件的无状态化。一个典型的场景是，利用高阶组件将原本受控组件需要自己维护的状态统一提升到高阶组件中。

**App.js**

```
render() {
    return this._renderOfHightLevelComponentTest()
}



/**
* @description: 高阶组件渲染
* @param {*}
* @return {*}
*/
_renderOfHightLevelComponentTest = () => {

	// 组件状态渲染
	const ComponentWithControlledState = withCOntrolledState(SimpleControlledComponent)
	return <ComponentWithControlledState />
}
```


<br/>

**HightLevelComponent.js**

```
/**
 * @description: 组件状态提升
 * @param {*} WrappedCommponent 外部组件
 * @return {*}
 */
export function withCOntrolledState(WrappedCommponent) {
    return class extends React.Component {
        constructor(props) {
            super(props)
            this.state = {
                value: ''
            }

            this.handleValueChange = this.handleValueChange.bind(this)
        }

        handleValueChange(event) {
            console.log('🍎 输入值：', event.target.value)
            this.setState({
                value: event.target.value
            })
        }

        render() {
            // newProps 保存受控组件需要使用的属性和事件处理函数
            const newProps = {
                controlledProps: {
                    value: this.state.value,
                    onChange: this.handleValueChange
                }
            }
            return <WrappedCommponent {...this.props} {...newProps} />
        }
    }
}

export class SimpleControlledComponent extends React.Component {
    render() {
        return <input name='simple' {...this.props.controlledProps} />
    }
}
```


效果图：
![30](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react30.png)



<br/>
<br/>

> <h2 id=''></h2>





<br/>
<br/>

> <h2 id=''></h2>





<br/>

***
<br/>



> <h1 id=''></h1>





<br/>

***
<br/>



> <h1 id=''></h1>





<br/>

***
<br/>



> <h1 id=''></h1>





