

> <h2 id=''></h2>
- [**方法**](#方法)
	- [addPostFrameCallback](#addPostFrameCallback)
- **参考资料**
	- [**flutter_deer**](https://github.com/simplezhli/flutter_deer)
	- [Deer](https://weilu.blog.csdn.net/article/details/90546727)项目知识点集合




<br/>

***
<br/>


> <h1 id='方法'>方法</h1>


<br/>

> <h2 id='addPostFrameCallback'>addPostFrameCallback</h2>



&nbsp;用法：`addPostFrameCallback`回调方法在Widget渲染完成时触发，所以一般我们在获取页面中的Widget大小、位置时使用到。

&emsp;  若我们在接口请求前弹出loading。例如我将请求方法放在了initState方法中，可能出现的异常如下：

```
inheritFromWidgetOfExactType(_InheritedTheme) or inheritFromElement() was called before initState() completed.
When an inherited widget changes, for example if the value of Theme.of() changes, its dependent widgets are rebuilt. If the dependent widget’s reference to the inherited widget is in a constructor or an initState() method, then the rebuilt dependent widget will not reflect the changes in the inherited widget.
Typically references to inherited widgets should occur in widget build() methods. Alternatively, initialization based on inherited widgets can be placed in the didChangeDependencies method, which is called after initState and whenever the dependencies change thereafter.
```

&emsp;  所以我们需要在build完成后，才可以去创建这个新的组件（这里就是Dialog）。

&emsp;  所以解决方法就是使用addPostFrameCallback回调方法，等待页面build完成后在请求数据：

```
  @override
  void initState() {
    WidgetsBinding.instance.addPostFrameCallback((_){
      /// 接口请求
    });
  }
``` 

&emsp;  导致这类问题的场景很多，但是大体解决思路就是上述的[办法](https://weilu.blog.csdn.net/article/details/94849020)。



<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>





<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>




<br/>

***
<br/>


> <h1 id=''></h1>


<br/>

> <h2 id=''></h2>



<br/>
<br/>


> <h2 id=''></h2>






<br/>
<br/>


> <h2 id=''></h2>



