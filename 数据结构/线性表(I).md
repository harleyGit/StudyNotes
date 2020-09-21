># 时间复杂度的计算
#`推导大O阶:`
-  用常数 1 取代运行时间中的所有加法常数；
-  在修改后的运行数函数中，只保留最高阶项；
-  如果最高阶项存在且不是 1 就， 则去除与这个项相乘的常数，得到结果就是大 O 阶。

<br/>
***
<br/>

># 线性的表顺序存储

<br/>
# 插入
![顺序插入如插队](https://upload-images.jianshu.io/upload_images/2959789-cefa6587a0ad65c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
int initList(SqList *list){
    list -> length = 0;
    
    return OK;
}


//线性表顺序插入
int listInsert(SqList *list, int i, int e){
    //顺序线性表已经满
    if (list->length == MAXSIZE ) {
        return ERROR;
    }
    
    //当i比第一位置小或者比最后一位置后一位置还要大时(如：数组长度为9，但是i为10)
    if (i < 1 || i > list->length +1) {
        return ERROR;
    }
    
    //若插入数据位置不在表尾
    if (i <= list->length) {
        for (int k = list->length-1; k >= i-1; k --) {
            list->data[k+1] = list->data[k];
        }
    }
    
    //将新元素插入
    list->data[i -1] = e;
    list->length++;
    
    return TRUE;
}

//初始条件：顺序线性表L已存在
//操作结果：依次对L的每个数据元素输出
int listTraverse(SqList list){
    int i;
    for (i = 0; i < list.length; i ++) {
        int k = visitElement(list.data[i]);
        printf("\n");
    }
    return OK;
}

int visitElement(ElemType c){
    printf("%d", c);
    
    return OK;
}


SqList  L;
    
ElemType e = NULL;
Status  i;
int j;
    
i = initList(&L);
printf("初始化L后：L.length=%d\n",L.length);
    
for (j = 1; j <= 5; j ++) {
    i = listInsert(&L, j, 1);
}
printf("在L的表头依次插入1～5后：L.data=\n");
listTraverse(L);

```
输出：
`初始化L后：L.length=0`
`在L的表头依次插入1～5后：L.data=`
`1`
`1`
`1`
`1`
`1`


<br/>
# 删除
![删除🌰图](https://upload-images.jianshu.io/upload_images/2959789-5bc0d8502b8cb7a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
typedef struct
{
    //数组，存储数据元素
    ElemType data[MAXSIZE];
    //线性表当前长度
    int length;                                
}SqList;

//线性表顺序删除
int listDelete(SqList *list, int i, ElemType *e){
    
    //线性表为空
    if (list->length == 0) {
        return ERROR;
    }
    
    //删除位置不正确
    if (i < 1 || i > list->length) {
        return ERROR;
    }
    
    //取数组地址赋值给指针变量e
    *e = list->data[i-1];
    
    if (i < list->length) {
        for (int k = i-1; k < list->length; k ++) {
            list->data[k] = list->data[k +1];
        }
    }
    
    list->length -=1;
    
    return OK;
}


SqList list = {{1,1,1,1,1}, 5}; 
int k = 5;
int e = NULL;

listDelete(&list, k, &e);
printf("删除第%d个的元素值为：%d\n",k,e);
    
```
输出：
`删除第5个的元素值为：1`




<br/>
***
<br/>

># 线性表的链式存储

![链表基本结构](https://upload-images.jianshu.io/upload_images/2959789-d060f3ee4e965ce2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#`头指针作用：`
-  头指针是指链表指向第一个节点的指针，若链表有头结点，则是指向头结点的指针；

- 头指针具有表示作用，所以常用头指针冠以链表的名字；

-  无论链表是否为空，头指针均不为空。头指针是链表的必要元素；


<br/>
#`头结点作用`
- 头结点是为了操作的的统一个方便而设立的，放在第一元素的节点之前，其数据域一般无意义；

- 有了头结点，对在第一元素节点前插入节点和删除第一节点，其操作与其他节点的操作就统一了。

-  头结点不一定是链表必须的要素。

<br/>
![有、无头结点的3种展示](https://upload-images.jianshu.io/upload_images/2959789-35572aad1850a435.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>

># 结构体节点的定义和申明
```

typedef struct Node {
    int data;
    struct Node *next;
}Node;


typedef  struct Node* LinkList; //Node* 等价于 LinkList

```


<br/>

># 单链表的操作

<br/>

# 单链表的插入
![单链表的插入示意图](https://upload-images.jianshu.io/upload_images/2959789-a0188373ee1f6cf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

//初始条件：顺序线性表L已存在,1≤i≤ListLength(L)
//操作结果：在L中第i个位置之前插入新的数据元素e，L的长度加1，若是i=1，相当于头插法
Status chainedListInsert(LinkList *list, int i, int e){
    int j = 1;
    LinkList insertNode = (LinkList)malloc(sizeof(LinkList));
    LinkList tagNode = *list;
    
    if (!(*list) || j > i) {
        return FALSE;
    }
    
    //寻找第i个结点
    while (tagNode && j < i) {
        tagNode = tagNode->next;
        ++ j;
        
    }
    
    if (tagNode != NULL || i > 1) {
        insertNode->next = tagNode->next;
        tagNode->next = insertNode;
        insertNode->data = e;
       
        return TRUE;
    }
    

    return TRUE;
}


//单链表值的遍历
int chainedListTraverse(LinkList *list){
    if ((*list)->next == NULL) {
        return FALSE;
    }
    
    printf("链表的值是：");
    
    LinkList traverse = (*list)->next;
    while (traverse) {
        printf("%d,  ", traverse->data);
        traverse = traverse->next;
    }
    return TRUE;
}


void chainedListTest(void){
    LinkList list;
    int i, j, k, e;

for (j = 1; j <= 10; j ++) {
        chainedListInsert(&list, j, j);
    }
    printf("\n\n在list的表尾依次插入1～10后：list.data=");
    chainedListTraverse(&list);
}

```
输出：
`在list的表尾依次插入1～10后：list.data=链表的值是：1,  2,  3,  4,  5,  6,  7,  8,  9,  10,  `







<br/>
# 单链表元素的删除

![单链表删除](https://upload-images.jianshu.io/upload_images/2959789-8fa6608975cd7413.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

//初始条件：顺序线性表L已存在,1≤i≤ListLength(L)
//操作结果：在L中第i个位置之前插入新的数据元素e，L的长度加1，若是i=1，相当于头插法
Status chainedListInsert(LinkList *list, int i, int e){
    int j = 1;
    LinkList insertNode = (LinkList)malloc(sizeof(LinkList));
    LinkList tagNode = *list;
    
    if (!(*list) || j > i) {
        return FALSE;
    }
    
    //寻找第i个结点
    while (tagNode && j < i) {
        tagNode = tagNode->next;
        ++ j;
        
    }
    
    if (tagNode != NULL || i > 1) {
        insertNode->next = tagNode->next;
        tagNode->next = insertNode;
        insertNode->data = e;
       
        return TRUE;
    }
    

    return TRUE;
}


//单链表值的遍历
int chainedListTraverse(LinkList *list){
    if ((*list)->next == NULL) {
        return FALSE;
    }
    
    printf("链表的值是：");
    
    LinkList traverse = (*list)->next;
    while (traverse) {
        printf("%d,  ", traverse->data);
        traverse = traverse->next;
    }
    return TRUE;
}

#pragma mark -- 链表方法调用
void chainedListTest(void){
    //删除某一个元素
    for (j = 1; j <= 10; j ++) {
        chainedListInsert(&list, j, j);
    }
    printf("\n\n在list的表尾依次插入1～10后：list.data=");
    chainedListTraverse(&list);
    j=5;
    i = deleteChainedListElement(&list,j,&e); /* 删除第5个数据 */
    printf("\n\n删除元素后的布尔值：%d(1:是 0:否),删除第%d个的元素值为：%d\n",i,j,e);
    printf("\n\n在list的表尾依次插入1～9后：list.data=");
    chainedListTraverse(&list);
    i = deleteChainedList(&list);
    printf("\n\n链表删除后的布尔值：%d(1:是 0:否)", i);
    
    
    //头插法创建链表
    
    printf("\n\n");
    
}


```
输出：
`在list的表尾依次插入1～10后：list.data=链表的值是：1,  2,  3,  4,  5,  6,  7,  8,  9,  10,  `

`删除元素后的布尔值：1(1:是 0:否),删除第5个的元素值为：5`


`在list的表尾依次插入1～9后：list.data=链表的值是：1,  2,  3,  4,  6,  7,  8,  9,  10,  `




<br/>
# 单链表的删除

```

typedef struct Node {
    int data;
    struct Node *next;
}Node;

typedef  struct Node* LinkList; //Node* 等价于 LinkList

/链表的删除
//LinkList *list申明的变量中，*list是一个变量，而且是一个指针变量。相当于 int a，这个a相当于 *list
int deleteChainedList(LinkList *list){
    
    LinkList p = (*list)->next;
    LinkList q = NULL;
    
    while (p) {
        //先判断p是否存在，再对q赋值，否则容易造成崩溃
        q = p->next;
        free(p);
        p = q;
    }
    
    //头结点指针域为空
    (*list)->next = NULL;
    
    return TRUE;
}

```




<br/>
# 链表的创建

#`头插法创建链表`
![头插法创建链表](https://upload-images.jianshu.io/upload_images/2959789-ecc1f1d7f82c3a7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
typedef struct Node {
    int data;
    struct Node *next;
}Node;

typedef  struct Node* LinkList; //Node* 等价于 LinkList

//头插法创建链表
int createListOfHeadInsert(LinkList *list, int length){
    if (length < 0) {
        return  FALSE;
    }
    
    //初始化随机数种子
    srand(time(0));
    //先建立一个带头结点的单链表
    (*list) = (LinkList)malloc(sizeof(Node));
    (*list)->next = NULL;
    
    while (length > 0) {
        LinkList insertNote = (LinkList)malloc(sizeof(Node));//注意：sizeof里的是类型名字，不能是LinkList，它是Node *
        insertNote->data = length;  //insertNote->data = rand()%100+1;
        insertNote->next = (*list)->next;
        //插入到表头
        (*list)->next = insertNote;
        
        length --;
    }
    
    return TRUE;
}


//单链表值的遍历
int chainedListTraverse(LinkList *list){
    if ((*list)->next == NULL) {
        return FALSE;
    }
    
    printf("链表的值是：");
    
    LinkList traverse = (*list)->next;
    while (traverse) {
        printf("%d,  ", traverse->data);
        traverse = traverse->next;
    }
    return TRUE;
}

//链表的删除
//LinkList *list申明的变量中，*list是一个变量，而且是一个指针变量。相当于 int a，这个a相当于 *list
int deleteChainedList(LinkList *list){
    
    LinkList p = (*list)->next;
    LinkList q = NULL;
    
    while (p) {
        //先判断p是否存在，再对q赋值，否则容易造成崩溃
        q = p->next;
        free(p);
        p = q;
    }
    
    //头结点指针域为空
    (*list)->next = NULL;
    
    return TRUE;
}



#pragma mark -- 链表方法调用
void chainedListTest(void){
    LinkList list;
    int i, j, k, e;
    

//头插法创建链表
    createListOfHeadInsert( &list, 20);
    chainedListTraverse(&list);
    i = deleteChainedList(&list);
    printf("\n\n头插法链表删除后的布尔值：%d(1:是 0:否)\n\n", i);
}


```
输出：
`链表的值是：1,  2,  3,  4,  5,  6,  7,  8,  9,  10,  11,  12,  13,  14,  15,  16,  17,  18,  19,  20, ` 

`头插法链表删除后的布尔值：1(1:是 0:否)`







<br/>



#`尾插法创建链表`

![尾插法链表创建](https://upload-images.jianshu.io/upload_images/2959789-c1e8616b8c7f4bef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
typedef struct Node {
    int data;
    struct Node *next;
}Node;

typedef  struct Node* LinkList; //Node* 等价于 LinkList



//尾插法创建链表
int createListOfTailInsert(LinkList *list, int length){
    if (length < 0) {
        return FALSE;
    }
    
    *list = (LinkList)malloc(sizeof(Node));
    //是指针的一种浅拷贝，相当于整型之间的赋值一样
    LinkList tagNode = *list;
    
    while (length > 0) {
        LinkList insertNode = (LinkList)malloc(sizeof(Node));
        insertNode->data = length;
        //将表尾终端结点的指针指向新结点
        tagNode->next = insertNode;
        //将当前的新结点定义为表尾终端结点
        tagNode = insertNode;
        
        length --;
    }
    
    //表示当前链表结束
    tagNode->next = NULL;
    
    return TRUE;
    
}


/单链表值的遍历
int chainedListTraverse(LinkList *list){
    if ((*list)->next == NULL) {
        return FALSE;
    }
    
    printf("链表的值是：");
    
    LinkList traverse = (*list)->next;
    while (traverse) {
        printf("%d,  ", traverse->data);
        traverse = traverse->next;
    }
    return TRUE;
}

//链表的删除
//LinkList *list申明的变量中，*list是一个变量，而且是一个指针变量。相当于 int a，这个a相当于 *list
int deleteChainedList(LinkList *list){
    
    LinkList p = (*list)->next;
    LinkList q = NULL;
    
    while (p) {
        //先判断p是否存在，再对q赋值，否则容易造成崩溃
        q = p->next;
        free(p);
        p = q;
    }
    
    //头结点指针域为空
    (*list)->next = NULL;
    
    return TRUE;
}



#pragma mark -- 链表方法调用
void chainedListTest(void){
    LinkList list;
    int i, j, k, e;
    
 //尾插法创建链表
    createListOfTailInsert(&list, 20);
    chainedListTraverse(&list);
    i = deleteChainedList(&list);
    printf("\n\n尾插法链表删除后的布尔值：%d(1:是 0:否)", i);
    
    
    printf("\n\n");
}


```
输出：
`链表的值是：20,  19,  18,  17,  16,  15,  14,  13,  12,  11,  10,  9,  8,  7,  6,  5,  4,  3,  2,  1,  `

`尾插法链表删除后的布尔值：1(1:是 0:否)`





<br/>
***
<br/>
下一回顾：[线性链表(II)](https://www.jianshu.com/p/1ed7e2d2c761)
