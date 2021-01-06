># 滚动图片
```
import React, { Component } from 'react';
import {
    AppRegistry,  //注册组件
    StyleSheet, //样式组件
    Text, //文本组件
    View,  //视图组件
    ScrollView,
    Image,
} from 'react-native';


// 导入图片的json数据
var ImageData = require('./Pictures.json');
// 引入定时器
//react-timer-mixin 是一个库文件
//$cd 文件地址(该项目的地址): cd /Users/huanggang/Desktop/Demo
//$ npm i react-timer-mixin --save
var TimerMixin = require('react-timer-mixin');

var Dimensions = require('Dimensions');
var {width} = Dimensions.get('window');

export default class Demo extends Component {

    //注册定时器
    mixins: [TimerMixin];

    state = {
        //当前页面小点点
        currentPage: 0,
    }

    //初始化固定值
    static  defaultProps = {
        // 间隔时间ms
        duration: 1000
    }

    //初始化固定值
    static defaultProps ={
        //间隔时间  单位是毫秒
        duration:1000
    }


    render() {
        return (
            <View style={styles.container}>
                <ScrollView
                    ref={'scrollView'}
                    horizontal={true}
                    showsHorizontalScrollIndicator={false}
                    pagingEnabled={true}
                    //一帧动画结束之后
                    onMomentumScrollEnd ={(e)=>this.onScrollAnimationEnd(e)}
                    onScrollBeginDrag = {()=>this.ScrollBeginDrag()}
                    onScrollEndDrag ={()=>this.startTimer()}
                >
                    {this.renderAllImage()}
                </ScrollView>

                {/*指示器*/}
                <View style={styles.pageViewStyle}>
                    {/*5个小点*/}
                    {this.renderPage()}
                </View>
            </View>

        );
    }


    // UI加载完毕
    componentDidMount() {
        // 开启定时器
        this.startTimer()
    }

    //开启定时器
    startTimer(){
        //逻辑代码
        //拿到scrollView
        var sctollView = this.refs.scrollView;
        //没有循环引用，但是有内存泄漏
        var imgCount = ImageData.data.length;
        var obj = this;

        //设置定时器， 在这个方法 setInterval 中this 指的是window
        setInterval(function () {
            //设置当前页
            var currentPage = 0;
            // 判断,若是大于imgCount说明超越了，要清零
            if ((obj.state.currentPage +1)>=imgCount) {
                //清零
                currentPage = 0;
            }else {
                currentPage = obj.state.currentPage +1;
            }

            //更新状态机
            obj.setState({
                currentPage: currentPage
            })

            // 滚起来
            var  offsetX =  currentPage * width;
            sctollView.scrollTo(offsetX, 0, true)
        }, this.props.duration)
    }


    //滚动完毕,相当于OC的代理方法
    onScrollAnimationEnd(e){
        //拿出偏移量
        var offsetX = e.nativeEvent.contentOffset.x;
        // 求出第几页
        var currentPage = Math.floor(offsetX/width);
        //更新状态机
        this.state.currentPage = currentPage;
    }

    // 加载所有图片
    renderAllImage(){

        var allImage = [];
        //拿到图片数据
        var imgsArr = ImageData.data;

        //遍历
        for (var i =0; i<imgsArr.length; i++){
            //取出单个图片数据
            var  imgItem = imgsArr[i];
            //创建图片组件到数组里面
            allImage.push(
                <Image
                    key={i}
                    // uri获取的图片必须设置size，否则看不到
                    source={{uri:imgItem.img}}
                    style={{width:width, height:150}}
                />
            )
        }
        //返回所有的图片
        return allImage;
    }


    //滚动完毕
    onScrollAnimationEnd(e){
        //拿到偏移量
        var offsetX = e.nativeEvent.contentOffset.x;

        //求出当前第几页
        var currentPage = Math.floor(offsetX/width);

        //更新状态机
        this.setState({
            currentPage:currentPage
        })
    }

    ScrollBeginDrag(){
        clearInterval(this.timer);
    }

    //开启定时器
    startTimer(){
        var scrollView = this.refs.scrollView;
        var imgCount = ImageData.data.length;
        var obj = this;

        //设置定时器
        this.timer = setInterval(function () {
            //设置当前页
            var  currentPage = 0;

            //判断
            if ((obj.state.currentPage +1) >= imgCount) {
                //清零
                currentPage = 0;
            }else {
                currentPage = obj.state.currentPage +1;
            }

            console.log(currentPage);

            //更新状态机
            obj.setState({
                currentPage:currentPage
            })

            //滚起来
            var offsetX = currentPage * width;
            scrollView.scrollTo({x:offsetX, y:0, animated:true});
        }, this.props.duration);
    }

    //返回小点点
    renderPage(){
        var  style;
        //点一个装点点的数组
        var pageArr =[];
        //拿到图片数组
        var  imgsArr = ImageData.data;

        //遍历
        for (var i =0;i<imgsArr.length; i++){
            //判断
            style = (i==this.state.currentPage)?{color:'orange'}:{color:'#ffffff'};
            //给page加小先对象
            //&bull; 点的转义字符
            pageArr.push(
                <Text
                    key={i}
                    // 多个样式用数组
                    style={[{fontSize:25}, style]}
                    > &bull; </Text>
            )
        }
        return pageArr;
    }


}


const styles = StyleSheet.create({
    container: {

    },

    pageViewStyle:{
        width: width,
        height: 25,
        backgroundColor:'rgba(0,0,0,0.2)',
        position:'absolute',
        bottom:0,

        //主轴方向
        flexDirection:'row',//横的
        justifyContent:'flex-end',//对齐方式
        alignItems:'center'
    }
})

//输出到ios的app中去
AppRegistry.registerComponent('Demo', () => Demo);

```
![滚动条图片显示](https://upload-images.jianshu.io/upload_images/2959789-43974c7c5bf08bd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
#`自定义的空件，导入到另一个View控件中`
如：在`index.ios.js`中使用`ScrollTest.js`的空件，可以这样做：
在`index.ios.js`中写入
```
import ScrollTest from "./ScrollTest";

//在其return的方法中写入：
<ScrollTest/>
``` 
调整合适的样式，就可以把View放入进来。



<br/>
***
<br/>
># ListView

#`单节的数据`
```
var Heros = require('./heros.json')

export default class ListViewHeros extends Component{

    // 构造函数初始化
    constructor(props) {
        super(props);

        var  ds = new ListView.DataSource({rowHasChanged:(r1, r2)=>r1 != r2});
        this.state = {
            dataSource: ds.cloneWithRows(Heros)
        };
    }
}
```

<br/>
#`多节的数据显示`
Car.json
```
{
  "data": [
    {
      "cars": [
        {
          "icon": "m_180_100.png",
          "name": "AC Schnitzer"
        },
        {
          "icon": "m_92_100.png",
          "name": "阿尔法·罗密欧"
        },
        {
          "icon": "m_9_100.png",
          "name": "奥迪"
        },
        {
          "icon": "m_97_100.png",
          "name": "阿斯顿·马丁"
        }
      ],
      "title": "A"
    },
    //很多行数据
    .  .  .  .
    .  .  .  .
}
```

```
var Car = rqueire('./Car.json');

constructor(props){
        super(props);

        var getSectionData = (dtaBlob, sectionID)=>{
            return dataBlob[sectionID];
        };
        var getRowData = (dataBlob, sectionID, rowID)=>{
            return dataBlob[sectionID+ ':' +rowID];
        };

        this.state ={
            dataSource: new ListView.DataSource({
                getSectionData: getSectionData, //获取组数据
                getRowData:getRowData,  //获取数据
                rowHasChanged:(r1, r2) => r1!== r2,
                sectionHeaderHasChanged:(s1, s2) =>s1 !== s2,
            })
        }

    }



    //发送网络请求的生命周期方法
    componentDidMount() {
        this.loadJson();
    }

    loadJson(){
        //拿到Json
        var jsonData = Car.data;
        // 定义数据源重要变量
        var dtaBlob = {},
            sectionIDs = [],
            rowIDs = [],    //二维数组
            cars = [];

        for (var i =0; i<= jsonData.length; i++){
            // 1.组ID拿到
            sectionIDs.push(i);
            // 2.dataBlob
            dataBlob[i] = jsonData[i].title;
            // 3.取出这一组的所有车
            cars = jsonData[i].cars;
            rowIDs[i] = [];
            for (var j=0; j<=cars.length; j++){
                // i组的行 那么这一行的ID 就是 j
                rowIDs[j].push(j);
                // 每一行的内容放到dataBlob里面了
                dataBlob[i+':'+j] = cars[j];
            }

        }

        // 更新状态机
        this.setState({
            dataSource:this.state.dataSource.cloneWithRowsAndSections(dataBlob, sectionIDs, rowIDs)
        })
    }
```



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
[]()

