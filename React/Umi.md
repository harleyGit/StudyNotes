> <h1 id=""></h1>
- [**‌ Umi框架**](#Umi框架)
	- [路由跳转](#路由跳转)


<BR/><BR/><BR/>

***
<BR/>

> <H1 ID= "Umi框架">Umi框架</H1>


***
<br/><br/><br/>
> <h2 id="路由跳转">路由跳转</h2>

```ts
export interface UmiHistory extends History {}
```

使用 Umi（UmiJS 前端框架），扩展原生 `history` 对象的声明方式。`UmiHistory` 是一个 **封装了浏览器导航行为的对象**，你可以通过它来进行页面跳转、返回、获取当前路径等操作。

<br/> 

Umi 中 `history` 用于**控制路由跳转**，类似于浏览器的前进/后退操作。

> 🚀 简单来说：**你想“跳转页面”，就用 `history.push('/xxx')`**

<br/>

**🔹 示例：**

```ts
import { history } from 'umi';

// 跳转到 /login 页面
history.push('/login');

// 跳转并携带参数（URL 查询）
history.push('/product?id=123');

// 返回上一页
history.goBack();

// 替换当前地址（不会记录历史）
history.replace('/dashboard');

// 当前路径名
console.log(history.location.pathname);
```

---
<br/>

**如何“定位”自己要去的页面？**

| 目标            | 用法                                           |
| ------------- | -------------------------------------------- |
| 跳转到某个页面       | `history.push('/目标路径')`                      |
| 替换当前页面        | `history.replace('/目标路径')`                   |
| 携带参数跳转        | `history.push('/xxx?key=value')`             |
| 使用编程跳转（如按钮点击） | 在函数中调用 `history.push(...)`                   |
| 获取当前路径        | `history.location.pathname`                  |
| 获取 URL 参数     | `history.location.query`（需配合 Umi 的约定式 query） |

<br/>

**点击按钮跳转**

```tsx
import React from 'react';
import { history } from 'umi';

const MyComponent = () => {
  const goToDetail = () => {
    history.push('/detail?id=42');
  };

  return (
    <button onClick={goToDetail}>查看详情</button>
  );
};
```

📌 点击按钮后会跳转到 `/detail?id=42` 页面。

<br/>

**`UmiHistory` 类型扩展的意义**

```ts
export interface UmiHistory extends History {}
```

这只是类型扩展语法（TypeScript）：

* Umi 中的 `history` 对象是基于 `history` 库再封装的
* 如果你用 TS，可以让你的代码知道 `history` 对象有哪些方法（如 `.push`, `.goBack`, `.replace`）

你也可以这样声明你的方法参数：

```ts
function navigateTo(h: UmiHistory) {
  h.push('/home');
}
```

---
<br/>

**✅ 总结**

| 功能     | 方法                                        |
| ------ | ----------------------------------------- |
| 跳转页面   | `history.push('/page')`                   |
| 替换页面   | `history.replace('/page')`                |
| 后退一页   | `history.goBack()`                        |
| 获取当前路径 | `history.location.pathname`               |
| 携带参数跳转 | `history.push('/page?id=123')`            |
| 类型扩展   | `UmiHistory extends History` 主要用于 TS 类型补全 |








