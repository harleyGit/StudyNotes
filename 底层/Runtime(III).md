- **属性列表**



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
