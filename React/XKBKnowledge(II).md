># <h2 id=''></h2>
- [**window**](#window)
	- [iframe](#iframe)



># <h1 id="window">window</h1>

<br/>

># <h2 id='iframe'>iframe</h2>



&emsp; 对于iframe嵌入的窗口，document.getElementById方法可以拿到该窗口的 DOM 节点，然后使用contentWindow属性获得iframe节点包含的window对象。

```
var frame = document.getElementById('theFrame');
var frameWindow = frame.contentWindow;
```
&emsp; 上面代码中，frame.contentWindow可以拿到子窗口的window对象。然后，在满足同源限制的情况下，可以读取子窗口内部的属性。

```
// 获取子窗口的标题
frameWindow.title
```


<iframe>元素的contentDocument属性，可以拿到子窗口的document对象。

```
var frame = document.getElementById('theFrame');
var frameDoc = frame.contentDocument;

// 等同于
var frameDoc = frame.contentWindow.document;
```

&emsp; <iframe>元素遵守同源政策，只有当父窗口与子窗口在同一个域时，两者之间才可以用脚本通信，否则只有使用window.postMessage方法。

&emsp; <iframe>窗口内部，使用window.parent引用父窗口。如果当前页面没有父窗口，则window.parent属性返回自身。因此，可以通过window.parent是否等于window.self，判断当前窗口是否为iframe窗口。

```
if (window.parent !== window.self) {
  // 当前窗口是子窗口
}
```

&emsp; <iframe>窗口的window对象，有一个frameElement属性，返回<iframe>在父窗口中的 DOM 节点。对于非嵌入的窗口，该属性等于null。

```
var f1Element = document.getElementById('f1');
var f1Window = f1Element.contentWindow;

f1Window.frameElement === f1Element // true
window.frameElement === null // true
```


<br/>

***
<br/>


># <h1 id=""></h1>





<br/>

***
<br/>


># <h1 id=""></h1>




<br/>

***
<br/>

