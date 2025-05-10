> <h1 id= ""></h1>
- [**管理Node版本的NVM**](#管理Node版本的NVM)
- [**新建一个React项目**](#新建一个React项目)
	- ss
- [Flex布局](#Flex布局)
- [清理缓存导致的错误](#清理缓存导致的错误)
- [跨域造成无法请求解决](#跨域造成无法请求解决)




<br/><br/><br/>

***
<br/>

> <h1 id= "管理Node版本的NVM">管理Node版本的NVM</h1>

NVM（Node Version Manager）是一个用来**管理多个 Node.js 版本的工具**。它在前端开发、特别是用 React 编码时非常有用，原因如下：

---

## 🔧 NVM 是做什么的？

NVM 可以让你：

1. **安装多个 Node.js 版本**（比如 14、16、18、20……）；
2. **按项目或全局切换 Node.js 版本**；
3. 避免系统安装冲突，不需要使用 `sudo` 来装包；
4. 卸载和更新 Node.js 版本也变得简单；

---

## 🚀 为什么用 React 时建议用 NVM？

### 1. **React 项目依赖特定的 Node 版本**

一些 React 项目（特别是老项目或用某些工具链，如 Vite、Next.js、Create React App 等）可能**只支持某个范围内的 Node 版本**。
如果你直接用系统的 Node 版本，很可能会出现：

* npm/yarn 安装报错；
* 构建失败；
* 插件或依赖兼容性问题。

👉 有了 NVM，你就可以快速切换到兼容的版本来运行项目。

---

### 2. **不同项目用不同版本的 Node**

你可能有多个 React 项目，它们依赖不同版本的 Node，比如：

* 项目 A 要求 Node 16
* 项目 B 用的是 Node 20

用 NVM，进入某个项目目录时可以执行：

```bash
nvm use 16
```

甚至你可以在项目中放一个 `.nvmrc` 文件，里面写上需要的版本，比如：

```bash
16
```

然后运行：

```bash
nvm use
```

就会自动切换。

---

### 3. **开发环境更干净**

使用 NVM 安装 Node，不会影响系统默认环境，也不需要 `sudo` 权限，更安全。

---

## 🧪 总结一句话：

> **NVM 是前端开发必备工具之一，能帮你灵活地切换 Node.js 版本，确保 React 项目在正确的环境下运行。**

***
<br/><br/><br/>
> <h2 id="安装NVM">安装NVM</h2>

在 Mac 上使用 Homebrew 安装 NVM 非常简单。下面是详细的步骤：

---

## 🍺 使用 Homebrew 安装 NVM（Node Version Manager）

### ✅ 第 1 步：确认已安装 Homebrew

打开终端，输入：

```bash
brew --version
```

如果输出版本号，说明你已经安装了。如果没有，请先去 [https://brew.sh](https://brew.sh) 安装 Homebrew。

---

### ✅ 第 2 步：用 Brew 安装 NVM

```bash
brew install nvm
```

---

### ✅ 第 3 步：创建 NVM 的目录

NVM 需要一个目录来存储你安装的 Node.js 版本。可以用这个命令：

```bash
mkdir ~/.nvm
```

---

### ✅ 第 4 步：配置 shell（zsh 或 bash）

你需要在你的 shell 配置文件中加入以下内容（取决于你用的是 zsh 还是 bash）：

#### 对于 Zsh 用户（默认 macOS 用 zsh）：

```bash
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"' >> ~/.zshrc
```

然后运行：

```bash
source ~/.zshrc
```


<br/>

**或者在`.bash_profile`中这样安装：**

~/.bash_profile 的完整内容（适用于 macOS M 系列，如 M1/M2/M3）：

```sh
# NVM 配置开始
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"  # 加载 nvm 主程序
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"  # 可选：启用自动补全
# NVM 配置结束
```



<br/><br/>

#### 如果你用的是 Bash：

把上面那几行加到 `~/.bash_profile` 或 `~/.bashrc`，然后执行 `source ~/.bash_profile`

---

### ✅ 第 5 步：验证 NVM 是否安装成功

```bash
nvm --version
```

如果你看到版本号，说明安装成功！

---

### ✅ 第 6 步：使用 NVM 安装 Node.js

```bash
nvm install 18
```

然后切换使用它：

```bash
nvm use 18
```

设置为默认版本：

```bash
nvm alias default 18
```


<br/><br/><br/>

***
<br/>

> <h1 id= "新建一个React项目">新建一个React项目</h1>

安装好 Node.js 和 NVM 后，你就可以非常方便地创建并启动一个 React 项目了。下面是 **从零开始用 `npm` 创建并启动 React 应用的完整详细步骤**：

---

## ✅ 前提：你已经完成了这些

* ✅ 已经通过 `nvm install` 安装了 Node.js（建议 Node 版本 ≥ 18）
* ✅ `npm` 已随 Node 一起安装（可通过 `npm -v` 检查）

---

## 🚀 创建并启动 React 项目的完整步骤

### 📦 第 1 步：创建 React 项目

使用官方推荐的脚手架工具 `create-react-app`（一次性生成完整项目结构）：

```bash
npx create-react-app my-app
```

解释：

* `npx` 是 `npm` 附带的一个执行器，会自动下载并运行最新的 `create-react-app`；
* `my-app` 是你的项目文件夹名（你可以改成任何名字）；
* 它会自动下载依赖、创建文件夹、配置项目等。

👉 如果你看到过程中有提示 `[✔] Success! Created my-app...`，说明创建成功。

---

### 📁 第 2 步：进入项目目录

```bash
cd my-app
```

---

### 🚀 第 3 步：启动开发服务器

```bash
# 安装一些依赖
npm install

#启动服务器
npm start
```

这条命令会：

* 启动 React 的开发服务器（基于 Webpack）；
* 打开浏览器，访问 `http://localhost:3000/`；
* 实时热更新你写的代码。

---

### 🔁 项目目录结构大概长这样：

```bash
my-app/
├── node_modules/        # 所有依赖包
├── public/              # 静态资源目录
├── src/                 # 你的源码目录（写 React 的地方）
│   ├── App.js           # 默认首页组件
│   ├── index.js         # 应用入口
├── .gitignore
├── package.json         # 项目信息和依赖列表
├── README.md
```

---

## 🔨 开发建议

* 你可以编辑 `src/App.js` 来修改页面内容；
* 保存后浏览器会自动刷新；
* 你可以用任意编辑器（如 VS Code）打开这个目录开始开发：

```bash
code .
```

---

## ✅ 常用命令汇总

| 命令              | 作用               |
| --------------- | ---------------- |
| `npm start`     | 启动开发服务器          |
| `npm run build` | 打包项目用于生产环境       |
| `npm test`      | 运行测试             |
| `npm install`   | 安装依赖（比如团队项目拉下来后） |

---

是否需要我也演示一下用 Vite 创建更轻量更快的 React 项目？（更现代、更推荐）



<br/><br/><br/>

***
<br/>

># <h1 id="Flex布局">[Flex布局](https://cloud.tencent.com/developer/article/1620305)</h1>

横轴和纵轴布局通常指的是使用 **Flexbox** 或 **Grid** 布局实现的一维（横向或纵向排列）或二维（同时控制横轴和纵轴）布局。最常用的两种方式是 **Flexbox** 布局和 **CSS Grid** 布局。

### 1. **Flexbox 布局**

**Flexbox** 布局是一个一维的布局模型，它可以在横向或纵向上排列子元素。通过设置 `flex-direction` 来控制横轴（主轴）和纵轴（交叉轴）的方向。`flex-direction` 可以是以下几种值：

* `row`（默认值）：元素按横向（从左到右）排列。
* `column`：元素按纵向（从上到下）排列。
* `row-reverse`：元素按横向（从右到左）排列。
* `column-reverse`：元素按纵向（从下到上）排列。

**Flexbox 的基础属性**：

* `justify-content`: 控制主轴（横轴方向）上的对齐方式。
* `align-items`: 控制交叉轴（纵轴方向）上的对齐方式。
* `flex-direction`: 设置主轴方向。
* `flex-wrap`: 控制子元素是否换行。

### 示例：横轴和纵轴布局（Flexbox）

```jsx
import React from 'react';

class FlexboxExample extends React.Component {
  render() {
    return (
      <div style={{
        display: 'flex',
        flexDirection: 'row', // 横轴布局
        justifyContent: 'space-between', // 横轴上的项目之间有间距
        alignItems: 'center', // 纵轴居中对齐
        height: '200px',
        border: '1px solid black'
      }}>
        <div style={{ width: '100px', height: '50px', backgroundColor: 'lightblue' }}>Item 1</div>
        <div style={{ width: '100px', height: '50px', backgroundColor: 'lightgreen' }}>Item 2</div>
        <div style={{ width: '100px', height: '50px', backgroundColor: 'lightcoral' }}>Item 3</div>
      </div>
    );
  }
}

export default FlexboxExample;
```

### 解释：

* `display: 'flex'`：启用 Flexbox 布局。
* `flexDirection: 'row'`：设置主轴为横轴，子项将按横向排列。
* `justifyContent: 'space-between'`：设置横轴上的项目之间的间距均匀分布。
* `alignItems: 'center'`：设置纵轴（交叉轴）上的项目居中对齐。

### 2. **CSS Grid 布局**

**Grid 布局** 是一个二维的布局模型，能够同时控制横轴和纵轴上的元素排列。通过设置 `grid-template-columns` 和 `grid-template-rows` 来定义网格的列和行。

**Grid 的基本属性**：

* `grid-template-columns`: 定义网格列的数量和宽度。
* `grid-template-rows`: 定义网格行的数量和高度。
* `grid-gap`: 定义行与列之间的间距。

### 示例：横轴和纵轴布局（CSS Grid）

```jsx
import React from 'react';

class GridExample extends React.Component {
  render() {
    return (
      <div style={{
        display: 'grid',
        gridTemplateColumns: 'repeat(3, 1fr)', // 定义三列，每列宽度相等
        gridTemplateRows: '100px 100px', // 定义两行，每行100px高
        gap: '10px', // 列和行之间的间距
        border: '1px solid black'
      }}>
        <div style={{ backgroundColor: 'lightblue' }}>Item 1</div>
        <div style={{ backgroundColor: 'lightgreen' }}>Item 2</div>
        <div style={{ backgroundColor: 'lightcoral' }}>Item 3</div>
        <div style={{ backgroundColor: 'lightyellow' }}>Item 4</div>
        <div style={{ backgroundColor: 'lightpink' }}>Item 5</div>
        <div style={{ backgroundColor: 'lightgray' }}>Item 6</div>
      </div>
    );
  }
}

export default GridExample;
```

### 解释：

* `display: 'grid'`：启用 Grid 布局。
* `gridTemplateColumns: 'repeat(3, 1fr)'`：设置三列，每列宽度相等（`1fr` 表示等分的宽度）。
* `gridTemplateRows: '100px 100px'`：设置两行，每行的高度为 100px。
* `gap: '10px'`：设置行和列之间的间隔。

### 总结

* **Flexbox**：适用于一维布局（横轴或纵轴），可以控制单行或单列的排列，常用于处理简单的排列，如水平或垂直居中。
* **Grid**：适用于二维布局，能够同时控制横轴和纵轴上的元素布局，适合复杂的布局，如网格布局、杂志样式的内容。

### 选择使用哪个布局？

* 如果你只需要控制一行或一列，Flexbox 是最简单的选择。
* 如果你需要更复杂的布局（例如，多个行和列的排列），Grid 是更合适的选择。


<br/><br/><br/>

***
<br/>

> <h1 id="清理缓存导致的错误">清理缓存导致的错误</h1>

#### 1. **删除构建缓存**

```bash
rm -rf node_modules/.cache
```

#### 2. **重启开发服务器**

先关闭当前运行的 `npm run dev` 或 `npm start`，然后重新启动：

```bash
npm run start
# 或者
npm run dev
```

#### 3. **清理浏览器缓存（如果有问题出现在浏览器）**

可以尝试强制刷新页面：

* Windows: `Ctrl + Shift + R`
* Mac: `Cmd + Shift + R`

#### 4. **确保编辑器也没缓存语法分析错误**

有些编辑器（如 VS Code）可能自己缓存了 ESLint 报错提示，重启编辑器可以排除这个问题。

#### 5. **检查 ESLint 缓存（如果项目用了 ESLint）**

运行：

```bash
npx eslint . --cache --fix
```


<br/><br/><br/>

***
<br/>

> <h1 id="跨域造成无法请求解决">跨域造成无法请求解决</h1>

在项目目录中的`package.json中添加‌`

```json
devDependencies": {
    "http-proxy-middleware": "^3.0.5"
  }
```

<br/><br/>

然后新建`setup.js`在根目录:

```
const { createProxyMiddleware } = require('http-proxy-middleware');
 
module.exports = function(app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'https://itango.tencent.com/out/itango/myip',
      changeOrigin: true,
    })
  );

	app.use(
	    '/api1',
	    createProxyMiddleware({
	      target: 'https://itango.tencent.com/out/itango/myip1',
	      changeOrigin: true,
	    })
	  );
};
```

<br/>

然后在`httpManager.js`中编码:

```
class httpManager {
  async get(url) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error("Network response was not ok");
      const data = await response.json(); // 如果返回的是 JSON
      return data;
    } catch (error) {
      console.error("GET 请求出错:", error);
      throw error;
    }
  }
}

const HttpManagerInstance = new httpManager();
export default HttpManagerInstance;
```

然后在需要的地方调用即可:

```
async fetchIP() {
    try {
      const data = await HttpManager1.get(
        "/api"
      );
      this.setState({ ipData: data });
    } catch (error) {
      this.setState({ error: "获取 IP 失败" });
    }
  }
```



