<h1>  react-scripts 流程及源码分析_V3.1.md</h1>

<br/>

> <h2 id=''></h2>
- [**相关版本**](#相关版本)
- [**目录结构**](#目录结构)
- [**流程分析**](#流程分析)
- [**bin/react-scripts.js**](#bin/react-scripts.js)
- [**scripts/init.js**](#scripts/init.js)
- [**scripts/start.js**](#scripts/start.js)
- [**scripts/build.js**](#scripts/build.js)
- [**scripts/eject.js**](#scripts/eject.js)


<br/>

***
<br/>

> <h1 id='相关版本'>相关版本</h1>



该版本直接 fork 官方的[GitHub 仓库](https://www.github.com/facebook/create-react-app)，版本号为 **v3.2.1**。

详细的源代码注释，请查看[GitHub 仓库](https://github.com/FE-Roading/create-react-app)



<br/>

***
<br/>



> <h1 id='目录结构'>目录结构</h1>

```bash
├─bin
│  └─react-scripts.js                # 命令入口文件
├─config
│  └─jest                            # jest配置文件夹
│  ├─env.js                          # 环境变量配置
│  ├─modules.js                      # 返回对应的JS或TS配置
│  ├─paths.js                        # 项目主要的入口文件配置项
│  ├─pnpTs.js                        # pnp相关配置项
│  ├─webpack.config.js               # webpack的配置
│  └─webpackDevServer.config.js      # webpack开发服务器的配置内容
├─fixtures                           # 暂不清楚用途
├─lib                                # TS定义
├─scripts                            # 对应的启动脚本
│  ├─build.js                        # run build的脚本
│  ├─eject.js                        # run eject的脚本
│  ├─init.js                         # create-react-app后的执行脚本
│  ├─start.js                        # run start的脚本
│  └─utils                           # 工具函数封装
├─template                           # 普通的js模板文件夹
│  ├─public
│  └─src
└─template-typescript                # typescript版的模板文件夹
```




<br/>

***
<br/>



> <h1 id='流程分析'>流程分析</h1>


该源代码采用 lerna 工具管理依赖，主要的源码均位于 packages 目录下。在 react-scripts 目录下的 package.json 中，bin 字段内容如下：

```js
  "bin": {
    "react-scripts": "./bin/react-scripts.js"
  },
```

因此我们运行该**react-scripts**命令时，实际是执行该目录下的./bin/react-scripts.js。




![主要流程图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react48.png)
**‌主要流程图**



<br/>

***
<br/>



> <h1 id='bin/react-scripts.js'>bin/react-scripts.js</h1>


该文件在执行时，会读取命令行参数。主要源码如下：

```
javascript
const args = process.argv.slice(2);
```

在日常开发过程中，执行命令的主要格式是：**yarn run start**, 最终在命令行被转为换：**node react-scripts 路径 start**。因此要读取有效的命令行参数，也应该是从第三个参数开始。

在读取到的参数中，查找出有效的命令**build/eject/start/test**对应的序号，如果没有查找到则取第一个参数作为默认值，但最终也会直接输出警告信息后退出程序。

查找到有效命令后，执行**../scripts**目录下对应的脚本文件，并传入除有效参数外的剩余参数。



<br/>

***
<br/>





> <h1 id='scripts/init.js'>scripts/init.js</h1>
 

该文件主要是在执行完 create-react-app 命令后，将项目模板文件复制到新建的项目目录。传入的参数列表如下：

```js
{
  appPath, // 项目根目录
    appName, // 项目名称
    verbose, // 是否打印日志
    originalDirectory, // 命令执行目录
    template; // create-react-app命令中传入的额外参数：指定的模板文件路径
}
```



<br/>

***
<br/>




> <h1 id='scripts/start.js'>scripts/start.js</h1>


对应的执行命令为：**yarn run start**。主要作用：启动开发服务器，执行流程如下：

1. 设置环境变量：BABEL_ENV、NODE_ENV=“development”；
2. 加载自定义的环境变量配置；
3. 必要的入口文件检测：作为入口的 index.html 和 js;
4. 读取 ip 和 port；
5. 检测是否配置 browserslist。如果最终都没有 browserlist 则直接退出；
6. 查找可用端口：先确认默认端口是否可用，不可用则确认是否自动查找可用端口，不查找则直接退出，查找则返回一个可用端口；
7. 配置 createCompiler 的 options 并执行，返回一个 compiler；
8. 载入代理配置，并配置代理服务 prepareProxy；
9. 创建开发服务配置，具体的配置代码放在 webpackDevServer.config.js；
10. 运行 WebpackDevServer，传入 compiler 和 proxyConfig，返回一个 devServer
11. 启动 devServer 服务，如果在交互模式下清理控制台，再打开浏览器



![start流程图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react49.png)

<br/>

***
<br/>




> <h1 id='scripts/build.js'>scripts/build.js</h1>

对应的执行命令为：**yarn run build**。主要作用：源代码构建，执行流程如下：

1. 设置环境变量：BABEL_ENV、NODE_ENV=“production”；
2. 加载自定义的环境变量配置；
3. 必要的入口文件检测：作为入口的 index.html 和 js;
4. 生成 webpack 配置；
5. 检测是否配置 browserslist；
6. 检测是否配置 browserslist。如果最终都没有 browserlist 则直接退出
7. 在构建前，检测出所有文件大小的 map
8. 清空构建的输出目录
9. 复制待构建目录的 public 目录到构建目录：静态文件夹名——paths.appPublic
10. 开始 webpack 构建
11. 构建完成后，输出相关的构建信息：警告/警告/错误、文件大小信息、后续操作的提示


![build流程图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react50.png)


<br/>

***
<br/>


> <h1 id='scripts/eject.js'>scripts/eject.js</h1>


对应的执行命令为：**yarn run eject**。主要作用：暴露 webpack 配置到项目目录中，让用户可以自行修改构建配置，执行流程如下：

1. 确认是否需要继续当前操作——暴露 webpack 等配置信息？是——继续，否——退出
2. 如果有 git 源代码管理工具？否——继续；是——是否有未保存的记录？是——退出，否继续
3. 读取 react-scripts 待复制的文件夹及其下的文件，分别确认所有内容在项目根目录下是否存在，有一个存在则直接退出。检测的文件夹：['config', 'config/jest', 'scripts']
4. 生成相关的 jest 配置：jestConfig
5. 在项目目录下，创建待复制的文件夹
6. 相关文件复制：相关文件复制：在文件复制文件时，删除文件内容@remove-on-eject-begin...@remove-on-eject-end
7. package.json 配置修改并保存：
   > devDependencies、dependencies 中删除 react-scripts
   > dependencies 依赖添加：react-scripts 的 dependencies 中，除了也属于 optionalDependencies 的所有依赖，全部添加到项目依赖中
   > dependencies 依赖是所有字段全部按依赖名称重新排列一次
   > scripts 字段：用 react-scripts 的 bin 字段的 key 去匹配项目 package.json 的 scripts 中所有的 value，将匹配到的结果全部替换为 node scripts/[key].js
   > scripts 字段：删除 eject 项
   > 添加其他字段：jest、babel、eslintConfig
8. 如果类型声明的入口路径 appTypeDeclarations：读取并替换 react-scripts types 后保存
9. 移除 node_modules/.bin/react-scripts 命令
10. 重新安装依赖
11. 如果有 git 则直接提交 git 记录



![eject流程图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/react51.png)


