<!--
 * @Author: GangHuang harleysor@qq.com
 * @Date: 2025-09-11 09:18:34
 * @LastEditors: GangHuang harleysor@qq.com
 * @LastEditTime: 2026-05-16 15:26:36
 * @FilePath: /MLC_React/README00.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->

> # MLC_React工程介绍：React开发、项目总结、平时使用组件，自己封装的轮子
- [路由跳转和参数传递](#路由跳转和参数传递)
	- [页面传递参数](#页面传递参数)
	- [URLQuery：可刷新、可分享](#URLQuery：可刷新、可分享)
	- [路径参数-REST风格](#路径参数-REST风格)
	- [回传参数-`navigate`+`state`](#回传参数-`navigate`+`state`)
	- [回传参数-回跳时带Query](#回传参数-回跳时带Query) 
	- [全局状态【Redux/Context/Zustand】](#全局状态【Redux/Context/Zustand】)
- [package.json文档](#package.json文档)
- [浏览器标签页图标修改](#浏览器标签页图标修改ido)




<br/><br/><br/>

***
<br/>

> <h1 id="路由跳转和参数传递">路由跳转和参数传递</h1>

***
<br/><br/><br/>
> <h2 id="页面传递参数">页面传递参数</h2>

**`login.js`**

```js
this.props.navigate?.("/register", {
      state: {
        userName: userName,
        registerType: HGRegisterType.PHONE,
        testEmail:'harleysor@qq.com',
        testCode:123456,
      },
    });
```

**‼️ ➡ 第二个参数 必须是 `{ state }`，否则无法传递参数。**

<br/>

**`register.js`**

```js
componentDidMount() {
    const { location } = this.props;
    LogOut(
      "testEmail:",
      location.state.testEmail,
      "testCode:",
      location.state.testCode
    );
  }
```

<br/>

**各个页面必须被`‌withRouter`包裹后才能导出来使用**

```js
import { useNavigate, useLocation, useParams } from "react-router-dom";

export function withRouter(Component) {
  return function (props) {
    return (
      <Component
        {...props}
        navigate={useNavigate()}
        location={useLocation()}
        params={useParams()}
      />
    );
  };
}
```



***
<br/><br/><br/>
> <h2 id="URLQuery：可刷新、可分享">URLQuery：可刷新、可分享</h2>

```js
this.props.navigate("/register?from=login&inviteCode=ABC123");
```
<br/>

 **register 页面接收**

```js
class HGRegisterPage extends React.Component {
  componentDidMount() {
    const search = this.props.location.search;
    const query = new URLSearchParams(search);

    console.log(query.get("from"));       // login
    console.log(query.get("inviteCode")); // ABC123
  }

  render() {
    return <div>注册页</div>;
  }
}

export default withRouter(HGRegisterPage);

```

**⭐ 特点**

| 优点   | 缺点      |
| ---- | ------- |
| 刷新不丢 | URL 变长  |
| 可分享  | 不适合敏感信息 |

***
<br/><br/><br/>
> <h2 id="路径参数-REST风格">路径参数-REST风格</h2>

```js
{
  path: "/register/:inviteCode",
  element: <HGRegisterPage />,
}
```

<br/>

**跳转：**

```js
this.props.navigate("/register/ABC123");
```

<br/>

**接收**

```js
class HGRegisterPage extends React.Component {
  componentDidMount() {
    console.log(this.props.params.inviteCode);
  }

  render() {
    return <div>注册页</div>;
  }
}

export default withRouter(HGRegisterPage);
```

***
<br/><br/><br/>
> <h2 id="回传参数-`navigate`+`state`">回传参数-`navigate`+`state`</h2>
- React Router 没有回传机制
- **回传 = 跳回 + 带参数**

 **从 login 进入 register 时**

```js
this.props.navigate("/register", {
  state: {
    from: "/login",
  },
});
```

<br/>

**register 成功后回去并传数据**

```js
this.props.navigate(-1, {
  state: {
    registerSuccess: true,
    username: "harley",
  },
});
```

<br/>

**login 页面接收**

```js
const location = useLocation();

if (location.state?.registerSuccess) {
  console.log("注册成功", location.state.username);
}
```

✔ **这是 v6 官方认可的“回传”方式**

***
<br/><br/><br/>
> <h2 id="回传参数-回跳时带Query">回传参数-回跳时带Query</h2>

```js
navigate("/login?registered=1&username=harley");
```

***
<br/><br/><br/>
> <h2 id="全局状态【Redux/Context/Zustand】">全局状态【Redux / Context / Zustand】</h2>
适合：

* 多页面共享
* 登录态 / 用户信息

```js
userStore.setRegisterResult({...});
navigate("/login");
```


<br/><br/><br/>

***
<br/>

># <h1 id="package.json文档">[package.json文档](../基础(II).md#package.json文档)</h1>


<br/><br/><br/>

***
<br/>

> <h1 id="浏览器标签页图标修改">浏览器标签页图标修改</h1>

&emsp;&emsp; 修改浏览器标签页图标`（favicon），Vite + React` 工程只需要改` public 目录 + index.html` 即可，不用写 React 代码。

<br/>

- 把图标文件`（favicon.ico / logo.png / icon.svg）`放到项目根目录的 public/ 下。
- 打开根目录 index.html，找到 <head> 里这行：

```html
<!-- 默认可能是这样 -->
<link rel="icon" href="/vite.svg" />
```

- 改成你的图标路径（以 / 开头）：

```html
<!-- .ico 格式 -->
<link rel="icon" href="/favicon.ico" type="image/x-icon" />
<!-- 或 PNG -->
<link rel="icon" href="/logo.png" type="image/png" />
<!-- 或 SVG -->
<link rel="icon" href="/icon.svg" type="image/svg+xml" />
```

重启 npm run dev，浏览器强制刷新（Ctrl+F5 / Cmd+Shift+R）清缓存即可看到新图标。
