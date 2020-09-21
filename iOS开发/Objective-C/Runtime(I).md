># 获取类的成员和属性变量
#`class_copyIvarList(Class _Nullable cls, unsigned int * _Nullable outCount) `

```

    unsigned int icount = 0;
    id classObject = objc_getClass([@"AppDelegate" UTF8String]);
    Ivar *ivars = class_copyIvarList(classObject, &icount);
    NSLog(@"AppDelegate %d 个成员变量", icount);
    for (int i = 0; i < icount; i ++) {
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(ivars[i])];
        NSLog(@"第 %d 个成员变量是：%@", i, ivarName);
    }

```
输出：
`2019-05-15 11:40:47.309695+0800 Genealogy[3423:49651] AppDelegate 1 个成员变量
`
`2019-05-15 11:40:55.189206+0800 Genealogy[3423:49651] 第 0 个成员变量是：_window
`



<br/>
***
<br/>
># 获得类的属性列表
#`class_copyPropertyList(Class _Nullable cls, unsigned int * _Nullable outCount)`

```

    unsigned int icount = 0;
    id classObject = objc_getClass([@"AppDelegate" UTF8String]);
    objc_property_t *properties = class_copyPropertyList(classObject, &icount);
    NSLog(@"AppDelegate %d 个属性变量", icount);
    for (int i = 0; i < icount; i ++) {
        objc_property_t property = properties[i];
        
        NSString *propertyName = [[NSString alloc] initWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
        NSLog(@"第 %d 个成员变量是：%@", i, propertyName);
    }

```

输出：
`2019-05-15 14:36:34.108732+0800 Genealogy[6014:112910] AppDelegate 5 个属性变量`
`2019-05-15 14:36:42.488907+0800 Genealogy[6014:112910] 第 0 个成员变量是：window`
`2019-05-15 14:36:45.498659+0800 Genealogy[6014:112910] 第 1 个成员变量是：hash`
`2019-05-15 14:36:45.499019+0800 Genealogy[6014:112910] 第 2 个成员变量是：superclass`
`2019-05-15 14:36:45.499275+0800 Genealogy[6014:112910] 第 3 个成员变量是：description`
`2019-05-15 14:36:45.499532+0800 Genealogy[6014:112910] 第 4 个成员变量是：debugDescription`



<br/>
***
<br/>
>#



<br/>
***
<br/>
>#
