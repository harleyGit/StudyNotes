- **属性列表**
- **[Runtime源码分析](https://juejin.cn/post/6844903952039821320)**
	- [文章介绍](https://juejin.cn/user/3913917126673047/posts)



<br/>

***
<br/>

># 属性列表

`property_getAttributes(objc_property_t _Nonnull property) `:  

获取属性的真实类型

```
const char *attrs = property_getAttributes(property);

//Printing description of attrs:
(const char *) attrs = 0x0000000108cc3c26 "T@\"NSMutableArray<ShowNewsModel>\",&,N,V_list1"
```

[property_getAttributes()](https://www.jianshu.com/p/cefa1da5e775)
