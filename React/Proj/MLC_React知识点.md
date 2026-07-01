- [React 知识点](#React知识点)
	- [AbortController 请求中断](#AbortController请求中断)
	- [请求超时自动取消](#请求超时自动取消)
	- [React 组件卸载取消请求](#React组件卸载取消请求)
	- [AbortError 错误处理](#AbortError错误处理)


***
<br/><br/><br/>
> <h2 id="React知识点">React 知识点</h2>

本文整理 `AbortController` 在 React / Fetch 请求中的常见用法：**用 `signal` 绑定请求，用 `abort()` 主动中断请求，常用于接口超时、组件卸载清理、用户取消和防重复请求**。


***
<br/><br/><br/>
> <h3 id="AbortController请求中断">AbortController 请求中断</h3>

`AbortController` 是浏览器原生 API，用来手动中断异步任务，最常见场景是中断 `fetch` 请求。

核心由两部分组成：

| 成员 | 作用 |
|------|------|
| `controller.signal` | 中断信号，传给 `fetch` 作为请求的中断开关 |
| `controller.abort()` | 主动触发中断，通知绑定了该 `signal` 的任务停止 |

核心写法：

```js
const controller = new AbortController();

const fetchOptions = {
  method: upperMethod,
  headers: mergedHeaders,
  signal: controller.signal, // 把中断信号绑定给 fetch
};

const response = await fetch(url, fetchOptions);
```

`fetch` 只负责接收 `signal`，真正触发取消的是 `controller.abort()`。


***
<br/><br/><br/>
> <h3 id="请求超时自动取消">请求超时自动取消</h3>

请求超时取消的核心逻辑：**发起请求时绑定 `signal`，同时启动定时器；超过 `timeout` 还没返回，就调用 `abort()` 终止请求**。

```js
const controller = new AbortController();

// 超时后自动调用 abort，中断请求
const timer = setTimeout(() => controller.abort(), timeout);

const fetchOptions = {
  method: upperMethod,
  headers: mergedHeaders,
  signal: controller.signal,
};

try {
  const response = await fetch(url, fetchOptions);
  return response;
} finally {
  clearTimeout(timer);
}
```

执行流程：

```text
创建 AbortController
      │
      ▼
fetch(url, { signal: controller.signal })
      │
      ├── 请求在 timeout 前完成 → clearTimeout(timer)
      │
      └── 请求超过 timeout
              │
              ▼
        controller.abort()
              │
              ▼
        fetch 立即终止并抛出 AbortError
```

关键点：**请求正常成功或失败后都要 `clearTimeout(timer)`**，否则定时器到点后仍会执行 `abort()`，产生不必要的中断行为。

常见用途：

- **接口超时自动断开：** 避免接口长时间无响应导致页面一直等待。
- **用户主动取消：** 例如点击 `取消加载` 按钮时调用 `controller.abort()`。
- **防止重复请求：** 短时间多次查询时，中断上一次未完成请求，只保留最新请求。


***
<br/><br/><br/>
> <h3 id="React组件卸载取消请求">React 组件卸载取消请求</h3>

React 中常见问题：组件已经卸载，但请求稍后才返回，回调里继续 `setState`，可能触发无效状态更新。

```text
Can't perform a React state update on an unmounted component
```

解决方式是在 `useEffect` 中创建 `AbortController`，并在清理函数里取消请求：

```js
useEffect(() => {
  const controller = new AbortController();

  async function loadData() {
    try {
      const response = await fetch('/api/list', {
        signal: controller.signal,
      });
      const data = await response.json();
      setList(data);
    } catch (err) {
      if (err.name === 'AbortError') {
        return;
      }
      console.log('真实接口报错', err);
    }
  }

  loadData();

  return () => {
    controller.abort(); // 组件卸载，直接中断请求
  };
}, []);
```

流程：

```text
组件挂载
  │
  ▼
创建 controller 并发起 fetch
  │
  ├── 请求正常返回 → 更新 state
  │
  └── 组件先卸载
          │
          ▼
    cleanup 执行 controller.abort()
          │
          ▼
    fetch 抛出 AbortError，不再更新已卸载组件
```

结论：**React 组件内发起的请求，要么在请求返回前组件仍然存在，要么在卸载时主动取消，避免无效回调继续执行**。


***
<br/><br/><br/>
> <h3 id="AbortError错误处理">AbortError 错误处理</h3>

调用 `controller.abort()` 后，正在进行的 `fetch` 会被终止，并抛出 `name === 'AbortError'` 的异常。业务中应该把主动取消和真实接口错误区分开。

```js
try {
  await fetch(url, fetchOptions);
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('请求被主动取消/超时中断');
  } else {
    console.log('真实接口报错');
  }
} finally {
  clearTimeout(timer);
}
```

注意点：

- `abort()` 只能中断仍在进行中的请求；请求已经结束后再调用没有实际效果。
- 一个 `signal` 可以绑定多个 `fetch`，一次 `abort()` 会中断所有绑定该信号的请求。
- Axios 也支持请求取消，现代版本同样可以使用 `AbortController`。
- 现代浏览器基本都支持 `AbortController`，IE 不支持。

总结：**`AbortController` 是 Fetch 的请求中断控制器；在 React 中最常用于接口超时取消和组件卸载清理请求，避免无限等待、重复请求堆积和无效状态更新**。
