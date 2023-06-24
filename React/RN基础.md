> 
- [**项目开始**](#项目开始)
	- [环境配置](#环境配置)
	- [项目初始化](#项目初始化)
- [**语法基础**](#语法基础)
	- [样式声明](#样式声明)
	- [弹性布局](#弹性布局)
- [**组件**](#组件)
	- 






<br/>

***
<br/><br/>

> <h1 id='项目开始'>项目开始</h1>

<br/><br/>

> <h2 id='环境配置'>环境配置</h2>

- 基础环境:
	- 安装Node
	- 安装yarn(其实npm也可以,不过yarn速度会更快点)

- iOS环境配置:
	- Xcode
	- CocoaPods
	- watchman安装
		- 主要是用来调试:
			- Command+B:弹窗选择刷新、debug
			- `brew install watchman`


<br/><br/>

> <h2 id='项目初始化'>项目初始化</h2>



- 初始化项目
	- react-native init xxxproject
	- cd xxxproject
	- Android:yarn android
	- iOS
		- cd iOS
		- pod install
		- cd ..
		- yarn ios



- 好用的插件:
	- vscdoe
![rn0_00.png](./../Pictures/rn0_00.png)
	- 快捷命令:
		- rnc(react native class): 快速创建rn类
			- ![rn0_01.png](./../Pictures/rn0_01.png)
		- rnf(react native function): 类组件
			- ![rn0_02.png](./../Pictures/rn0_02.png)



- 调试
	- 模拟器调试
		- Android:点击模拟器后,然后快捷键:ctrl+B,然后选中debug,会跳转到浏览器
		- iOS: Command+B(M)




<br/>

***
<br/><br/>

> <h1 id=''></h1>


<br/><br/>

> <h2 id='样式声明'>样式声明</h2>

![rn0_03.png](./../Pictures/rn0_03.png)



<br/><br/>

> <h2 id='弹性布局'>弹性布局</h2>


- web的弹性布局

![rn0_04.png](./../Pictures/rn0_04.png)


<br/>

- Rn的弹性布局:

![rn0_05.png](./../Pictures/rn0_05.png)



<br/>

***
<br/><br/>

> <h1 id='组件'>组件</h1>
 
 
 RN生成的组件会生成对应的android和iOS的组件,如下图:
 
![rn0_06.png](./../Pictures/rn0_06.png)



<br/>

3大平台组件比对:

![rn0_07.png](./../Pictures/rn0_07.png)





