> <h1 id=""></h1>
- [**生命周期**](#生命周期)
- **参考资料：**
	- [**JavaScript优秀教程**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)



<br/>

***
<br/>

># <h1 id="生命周期">[生命周期](https://www.jianshu.com/p/c9bc994933d5)</h1>

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

componentWillReceiveProps (nextProps) {}

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


