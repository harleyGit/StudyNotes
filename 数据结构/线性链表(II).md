- **循环链表**
- **双向链表**
	- 双向链表的插入
	- 双向链表的删除



<br/>


***
<br/>

># 循环链表

![循环链表示意图](https://upload-images.jianshu.io/upload_images/2959789-2eb248ee9f51dc69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

**2个循环链表进行合并**
![循环链表合并](https://upload-images.jianshu.io/upload_images/2959789-83a5326f42466165.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//保存A表头的头结点，即①
p = rearA->next;
//将本是指向B表的第一个节点(不是头结点，是第一个节点)赋值给rearA->next,即②
rearA->next = rearB->next->next;
//将原A表的头结点赋值给rearB->next,即③
rearB->next = p;
//释放p
free(p);

```





<br/>

***
<br/>

># 双向链表


![双向链表的非空链表和空链表](https://upload-images.jianshu.io/upload_images/2959789-e28b820baa20fcd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

- **双向链表的插入**

![双向链表的插入](https://upload-images.jianshu.io/upload_images/2959789-00d156bcd57f5f58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

- **双向链表的删除**
![双向链表的删除](https://upload-images.jianshu.io/upload_images/2959789-3bd2dbc044214af7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//把p->next赋值给p->prior的后继，即①
p->prior->next = p->next;
//把p->prior赋值给p->next的前缀，即②
p->next->prior = p->prior;
//释放节点p
free(p);
```


<br/>

***
<br/>



上一回顾：[线性链表(I)](https://www.jianshu.com/p/60c6d1cc5a89)
