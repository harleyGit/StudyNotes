># 状态 setState方法调用
- `出发组件的 build 方法`；
-  对子、父小部件的影响：
    - 子控件的build方法会被执行，如果key不同，则initState方法也会被执行；
    - 父控件不执行build和initState；


<br/>
***
<br/>

># 导航 Navigation 
-  弹框pop：
    - initState和build都不会被执行；
- 页面pop：
    - pop回来之后,build 方法会被执行；
- 页面push：
    - push的时候,build 方法会被执行；
- 列表滚动：
    - 列表新item的build和initState方法都会被执行；
    - 当列表所在页面setState的时候，如果item的key没变化，则item只执行build方法，如果item的key有变化则执行build和initState；


<br/>
***
<br/>
># TabBarView切换
-  key改变:
    - initState和build都会被执行;








