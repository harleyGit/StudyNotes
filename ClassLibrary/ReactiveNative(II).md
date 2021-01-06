



>#  文件 index.ios.js 介绍
```
import React, { Component } from 'react';
import {
  AppRegistry,  //注册组件
  StyleSheet, //样式组件
  Text, //文本组件
  View,  //视图组件
    Image,
    TextInput,
} from 'react-native';

export default class Demo extends Component {
  // render 初始化方法,渲染界面 相当于 viewDidLoad
  render() {
    return (
            //视图组件布局Code
        );
    }
}


const styles = StyleSheet.create({
  container: {
    flex: 1,  //子控件占父控件的比例
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
});

//输出到ios的app中去
AppRegistry.registerComponent('Demo', () => Demo);
```

<br/>
***
<br/>
># Flex Box 布局
```
render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          欢迎来到王者荣耀!
        </Text>

        <Text style={styles.instructions}>
          To get started, edit index.ios.js
        </Text>
        <Text style={styles.instructions}>
          Press Cmd+R to reload,{'\n'}
          Cmd+D or shake for dev menu
        </Text>

      </View>
    );
  }

const styles = StyleSheet.create({
  container: {
    flex: 1,  //子控件占父控件的比例
    //改变主轴的方向 --> 默认是垂直方向
    //column 从上到下
    //column-verbose 从下到上
    //row 从左到右
    //row-reverse 从右往左
    flexDirection: 'column',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },

});
```
![column 从上往下布局](https://upload-images.jianshu.io/upload_images/2959789-1b1a01674e93fe46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


默认是从下往上的布局，当我们把父视图的`container`样式的`flexDirection`改为`row  (flexDirection: 'row')`,如下图：
![row  从左往右布局](https://upload-images.jianshu.io/upload_images/2959789-fa68b16fc65f9b70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)











<br/>
***
<br/>
> # 样式
```
//设置背景颜色：transparent 背景色为透明
background: 'transparent'

//距离顶部的距离
marginTop: 

//主轴的方向设置，在flexBox中的布局
//flex布局，布局内部的子组件，但对于Text子组件，因为文本是Text的一部分，所以子组件Text的内部文本并不会改变
flexDirection: 

//设置主轴的对齐方式: flex-start 对齐主轴起点位置， flex-start 主轴对齐尾部位置， space-bettween 平均分配主轴的空间两端对齐， space-around 平均分配
justiyContent:'flex-start'


//侧轴的对齐方式: center 居中显示
//默认值：stretch  若没有设置高度，或者高度为auto，子控件就占满父控件
alignItems: 'center'

//主轴不显示，自动换行
flexWrap:'wrap'

//子控件自己布局
alignSelf: 'flex-start'
````

#`查看屏幕的宽、高、分辨率`
```
//引入
var Dimensions = require('Dimensions')
<Text style={styles.instructions}>
          当前屏幕宽度：{Dimensions.get('window').width}
          当前屏幕高度：{Dimensions.get('window').height}
          当前屏幕分辨率：{Dimensions.get('window').scale}
</Text>

instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  }
```
![iPhone8 plus 的显示](https://upload-images.jianshu.io/upload_images/2959789-294b44e288844984.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
***
<br/>
># 真机调试
- 文件配置，【工程名】->【Libraries】->【RCTWebSocket.xcodeproj】。现在最新的版本是不需要的，在0.59版本的ReactiveNative；
![文件配置](https://upload-images.jianshu.io/upload_images/2959789-1db961d0fa354efd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&emsp;上图的代码正常运行是不运行的，需要点击模拟器然后`Command+ D`,然后选择`Debug  JS Remotely`在电脑的浏览器进行调试。

- 修改`Bundle identifier`，否则会报错；










<br/>
***
<br/>
># 九宫格
```
//引入
var Dimensions = require('Dimensions');
//json 数组字典
var data = require('./app.json');
//获得屏幕宽度
var Dimensions = require('Dimensions');
var {width} = Dimensions.get('window');
var cols = 3;
var boxW = 100;
var vMargin = (width - cols * boxW) / (cols + 1);//九宫格算法....
var hMargin = 25;


export default class Demo extends Component {
  // render 初始化方法 相当于 viewDidLoad
  render() {
    return (
            <View style={styles.container}>
                <View style={styles.viewStyle}>
                    {this.renderAllBaobao()}
                </View>
            </View>
        );
  }


 //返回View中所有包包的函数
    renderAllBaobao(){
        //1.装包包的数组
        var images = [];
        //2.遍历数据
        for(var i=0;i<Data.length;i++){
            //3.取出Data中的Json
            var dataItem = Data[i];
            images.push(
                <View key={i} style={styles.outViewStyle}>
                    <Image source={{uri:dataItem['icon']}} style={styles.imageStyle}></Image>
                    <Text>{dataItem['name']}</Text>
                </View>
            );
        }
        //最后返回这个数组
        return images;
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F5FCFF',
    },
    viewStyle:{

        //确定主轴方向
        flexDirection:'row',
        flexWrap:'wrap'
    },
    imageStyle:{
        width:80,
        height:80,
    },
    outViewStyle:{
        backgroundColor:'red',
        alignItems:'center',
        width:boxW,
        height:boxW,
        marginLeft:vMargin,
        marginTop:hMargin,
    }

});

AppRegistry.registerComponent('Test1', () => Test1);
```

<br/>
***
<br/>
># 属性
-  Props 属性 相当于OC中的ReadOnly ,只读属性!!
-  state 状态 每个组件有一个系统的setState方法,用来改变状态,而且会刷新界面，调用render函数；













<br/>
***
<br/>
># 方法
-  调用Render()函数；
-  componentWillMount() 相当于OC中的ViewWillAppear；
- componentDidMount() 在render函数执行完后执行，用于UI显示完成后，进行网络请求(第一次加载数据)；
- componentDidUpdate()   刷新UI之后调用,第一次加载UI不会调用

```
/*
*变量
*/
//引入
var Dimensions = require('Dimensions');

export default class Demo extends Component {

  state = {
    //当前页面
    title: '无尽战刃'
  }

  // render 初始化方法 相当于 viewDidLoad
  render() {
    return (
      <View   ref="topView"  style={styles.container}>
        <Text style={styles.welcome}>
          欢迎来到王者荣耀!
          {this.state.title}
        </Text>
        <Button title={'拿起你的⚔吧'}
                color={'red'}
                onPress={()=>this.click('我拿起了⚔')}/> // 方法在外: onPress={btnClick}

        <Text style={styles.instructions}>
          当前屏幕宽度：{Dimensions.get('window').width}
          当前屏幕高度：{Dimensions.get('window').height}
          当前屏幕分辨率：{Dimensions.get('window').scale}
        </Text>
        <Text style={styles.instructions}>
          Press Cmd+R to reload,{'\n'}
          Cmd+D or shake for dev menu
        </Text>

      </View>
    );
  }


  click(event){
    console.log(event)
    this.setState({
      title:event
    })

     // 拿到view
    console.log(this.refs.topView)
  }

}

const btnClick = ()=>{
  AlertIOS.alert('哥们我来了')
}

```
![UI 显示](https://upload-images.jianshu.io/upload_images/2959789-e43161f91905f4f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)












<br/>
***
<br/>










<br/>
***
<br/>










<br/>
***
<br/>










<br/>
***
<br/>
























