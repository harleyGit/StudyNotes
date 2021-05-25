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


循环链表：

```
//
//  main.c
//  DataStruct
//
//  Created by Harley Huang on 20/5/2021.
//

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TRUE 1
#define FALSE 0

typedef int Status; // Status是函数结果状态，成功返回TRUE,失败返回FALSE
typedef int ElemType;

/* 线性表的循环链表存储结构 */
typedef struct node
{
    ElemType data;
    struct node *next;
}Node, LinkList;

// 初始化链表操作
void initList(LinkList **pList);
// 插入元素操作
Status insertList(LinkList *pList, int i, const ElemType e);
// 删除元素操作
Status deleteList(LinkList *pList, int i, ElemType *e);
// 获取元素操作
Status getElem(LinkList *pList, int i, ElemType *e);
// 头部后插入元素操作
Status insertListHead(LinkList *pList, const ElemType e);
// 尾部后插入元素操作
Status insertListTail(LinkList *pList, const ElemType e);
// 清空链表操作
Status clearList(LinkList *pList);
// 遍历链表操作
void traverseList(LinkList *pList);
// 获取链表长度操作
int getLength(LinkList *pList);

int main(int argc, const char * argv[]) {
    
    
    LinkList *pList;
    
    // 初始化链表
    initList(&pList);
    printf("初始化链表!\n\n");
    
    // 尾部后插入结点
    insertListTail(pList, 1);
    printf("尾部后插入元素1\n");
    insertListTail(pList, 2);
    printf("尾部后插入元素2\n\n");
    
    // 插入结点
    insertList(pList, 1, 4);
    printf("位置1插入元素4\n\n");
    
    // 删除结点
    int val;
    deleteList(pList, 3, &val);
    printf("删除位置3的结点，删除结点的数据为： %d\n", val);
    printf("\n");
    
    return 0;

    // 遍历链表并显示元素操作
    printf("遍历链表：");
    traverseList(pList);
    printf("\n");
    
    // 获得链表长度
    printf("链表长度: %d\n\n", getLength(pList));
    
    // 销毁链表
    clearList(pList);
    printf("销毁链表\n\n");
    
    printf("\n\nHello, World!\n");
    return 0;
}


// 初始化链表操作
// 必须使用双重指针，一重指针申请会出错
void initList(LinkList **pList)
{
    *pList = (LinkList *)malloc(sizeof(LinkList));
    if (!pList)
    {
        printf("malloc error!\n");
        return;
    }
    
    (*pList)->data = 0;
    // 因为是循环链表，所以尾指针指向头节点
    (*pList)->next = *pList;
}


// 插入元素操作
Status insertList(LinkList *pList, int i, const ElemType e)
{
    // 指向位置i所在的前一个结点
    Node *front;
    // 计数器
    int j;
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    // 只能在位置1以及后面插入，所以i至少为1
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的前一个结点
    front = pList;
    if (i != 1) // 对i!=1的情况特殊处理
    {
        front = pList->next; // 指向第2个结点的前一个结点，与j对应
        for (j = 2; j < i; j++) // j为计数器，赋值为2，对应front指向的下一个结点
        {
            front = front->next;
            if (front == pList)
            {
                printf("dont find front!\n");
                return FALSE;
            }
        }
    }
    
    
    // 创建一个空节点，存放要插入的新元素
    Node *temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return FALSE;
    }
    temp->data = e;
    
    // 插入结点
    temp->next = front->next;
    front->next = temp;
    
    return TRUE;
}


// 删除元素操作
Status deleteList(LinkList *pList, int i, ElemType *e)
{
    Node *front; // 指向位置i所在的前一个结点
    int j; // 计数器
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    // 只能删除位置1以及以后的结点
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的前一个结点
    front = pList;
    if (i != 1) // 对i=1的情况特殊处理
    {
        front = pList->next; // 指向第2个结点的前一个结点，与j对应
        for (int j = 2; j < i; j++) // j为计数器，赋值为2，对应front指向的下一个结点
        {
            front = front->next;
            if (front->next == pList)
            {
                printf("dont find front!\n");
                return FALSE;
            }
        }
    }
    
    // 提前保存要删除的结点
    Node *temp = front->next;
    *e = temp->data; // 将要删除结点的数据赋给e
    
    // 删除结点
    front->next = front->next->next;
    
    // 销毁结点
    free(temp);
    temp = NULL;
    
    return TRUE;
}

// 获取元素操作
Status getElem(LinkList *pList, int i, ElemType *e)
{
    Node *cur;
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    // 只能获取位置1以及以后的元素
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的结点
    cur = pList->next; // 这里是让cur指向链表的第1个结点
    int j = 1; // j为计数器，赋值为1，对应cur指向结点
    while (cur != pList && j < i)
    {
        cur = cur->next;
        j++;
    }
    // 未找到i位置所在的前一个结点
    if (cur == pList)
    {
        printf("dont find front!\n");
        return FALSE;
    }
    
    // 取第i个结点的数据
    *e = cur->data;
    
    return TRUE;
}

// 头部后插入元素操作
Status insertListHead(LinkList *pList, const ElemType e)
{
    Node *head;
    Node *temp;
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    // 让head指向链表的头结点
    head = pList;
    
    // 创建存放插入元素的结点
    temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return FALSE;
    }
    temp->data = e;
    
    // 头结点后插入结点
    temp->next = head->next;
    head->next = temp;
    
    return TRUE;
}

// 尾部后插入元素操作
Status insertListTail(LinkList *pList, const ElemType e)
{
    Node *cur;
    Node *temp;
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    // 找到链表尾节点
    cur = pList;
    while (cur->next != pList)
    {
        cur = cur->next;
    }
    
    // 创建存放插入元素的结点
    temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return -1;
    }
    temp->data = e;
    
    // 尾结点后插入结点
    temp->next = cur->next;
    cur->next = temp;
    
    return TRUE;
}

// 清空链表操作
Status clearList(LinkList *pList)
{
    Node *cur; // 当前结点
    Node *temp; // 事先保存下一结点，防止释放当前结点后导致“掉链”
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    cur = pList->next; // 指向头结点后的第一个结点
    while (cur != pList)
    {
        temp = cur->next; // 事先保存下一结点，防止释放当前结点后导致“掉链”
        free(cur); // 释放当前结点
        cur = NULL;
        cur = temp; // 将下一结点赋给当前结点p
    }
    pList->next = NULL; // 头结点指针域指向空
    
    return TRUE;
}

// 遍历链表操作
void traverseList(LinkList *pList)
{
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return;
    }
    
    Node *cur = pList->next;
    while (cur != pList)
    {
        printf("%d ", cur->data);
        cur = cur->next;
    }
    printf("\n");
}

// 获取链表长度操作
int getLength(LinkList *pList)
{
    Node *cur = pList;
    int length = 0;
    
    while (cur->next != pList)
    {
        cur = cur->next;
        length++;
    }
    
    return length;
}
```

打印：

```
初始化链表!

尾部后插入元素1
尾部后插入元素2

位置1插入元素4

删除位置3的结点，删除结点的数据为： 2

Program ended with exit code: 0
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

代码案例：

```
//
//  main.c
//  DataStruct
//
//  Created by Harley Huang on 20/5/2021.
//

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TRUE 1
#define FALSE 0

// Status是函数结果状态，成功返回TRUE,失败返回FALSE
typedef int Status;
typedef int ElemType;


typedef int ElemType;
// 双向非循环链表的结构定义
typedef struct Node
{
    ElemType data;  //数据域
    struct Node *prior;   //指向前驱结点的指针
    struct Node *next;    //指向后继结点的指针
}Node, DulList;

// 初始化链表操作
void initList(DulList **pList);
// 插入元素操作
Status insertList(DulList *pList, int i, const ElemType e);
// 删除元素操作
Status deleteList(DulList *pList, int i, ElemType *e);
// 获取元素操作
Status getElem(DulList *pList, int i, ElemType *e);
// 头部后插入元素操作
Status insertListHead(DulList *pList, const ElemType e);
// 尾部后插入元素操作
Status insertListTail(DulList *pList, const ElemType e);
// 清空链表操作
Status clearList(DulList *pList);
// 遍历链表操作
void traverseList(DulList *pList);
// 获取链表长度操作
int getLength(DulList *pList);




// 初始化链表操作
void initList(DulList **pList) // 必须使用双重指针，一重指针申请会出错
{
    *pList = (DulList *)malloc(sizeof(Node));
    if (!pList)
    {
        printf("malloc error!\n");
        return;
    }
    
    (*pList)->data = 0;
    (*pList)->prior = NULL;
    (*pList)->next = NULL;
}

// 插入元素操作
Status insertList(DulList *pList, int i, const ElemType e)
{
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    // 只能在位置1以及后面插入，所以i至少为1
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的前一个结点
    Node *front = pList; // 这里是让front与i不同步，始终指向j对应的前一个结点
    for (int j = 1; j < i; j++) // j为计数器，赋值为1，对应front指向的下一个结点，即插入位置结点
    {
        front = front->next;
        if (front == NULL)
        {
            printf("dont find front!\n");
            return FALSE;
        }
    }
    
    // 创建一个空节点，存放要插入的新元素
    Node *temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return FALSE;
    }
    temp->data = e;
    
    // 插入结点
    temp->prior = front;
    temp->next = front->next;
    // 当空链表第一次插入结点时，此时head->next = NULL，调用NULL->prior会出错
    if (front->next != NULL)
        front->next->prior = temp;
    front->next = temp;
    
    return TRUE;
}

// 删除元素操作
Status deleteList(DulList *pList, int i, ElemType *e)
{
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    // 只能删除位置1以及以后的结点
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的前一个结点
    Node *front = pList; // 这里是让front与i不同步，始终指向j对应的前一个结点
    for (int j = 1; j < i; j++) // j为计数器，赋值为1，对应front指向的下一个结点，即插入位置结点
    {
        front = front->next;
        if (front->next == NULL)
        {
            printf("dont find front!\n");
            return FALSE;
        }
    }
    
    // 提前保存要删除的结点
    Node *temp = front->next;
    *e = temp->data; // 将要删除结点的数据赋给e
    
    // 删除结点
    if (front->next->next != NULL) // 删除的不是尾结点，才进入
    {
        front->next->prior = front;
    }
    front->next = front->next->next;
    
    // 销毁结点
    free(temp);
    temp = NULL;
    
    return TRUE;
}

// 获取元素操作
Status getElem(DulList *pList, int i, ElemType *e)
{
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    // 只能获取位置1以及以后的元素
    if (i < 1)
    {
        printf("i is invalid!\n");
        return FALSE;
    }
    
    // 找到i位置所在的结点
    Node *cur = pList->next; // 这里是让cur指向链表的第1个结点，与j同步
    for (int j = 1; j < i; j++) // j为计数器，赋值为1，对应cur指向结点
    {
        cur = cur->next;
        if (cur == NULL)
        {
            printf("dont find front!\n");
            return FALSE;
        }
    }
    
    // 取第i个结点的数据
    *e = cur->data;
    
    return TRUE;
}

// 头部后插入元素操作
Status insertListHead(DulList *plist, const ElemType e)
{
    Node *head;
    Node *temp;
    
    // 判断链表是否存在
    if (!plist)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    // 让head指向链表的头结点
    head = plist;
    
    // 创建存放插入元素的结点
    temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return FALSE;
    }
    temp->data = e;
    
    // 头结点后插入结点
    temp->prior = head;
    temp->next = head->next;
    // 当空链表第一次插入结点时，此时head->next = NULL，调用NULL->prior会出错
    if (head->next != NULL)
        head->next->prior = temp;
    head->next = temp;
    
    return TRUE;
}

// 尾部后插入元素操作
Status insertListTail(DulList *pList, const ElemType e)
{
    Node *cur;
    Node *temp;
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    // 找到链表尾节点
    cur = pList;
    while (cur->next)
    {
        cur = cur->next;
    }
    
    // 创建存放插入元素的结点
    temp = (Node *)malloc(sizeof(Node));
    if (!temp)
    {
        printf("malloc error!\n");
        return -1;
    }
    temp->data = e;
    
    // 尾结点后插入结点
    temp->prior = cur;
    temp->next = cur->next;
    cur->next = temp; // 尾结点的直接后继指针是NULL，所以不用指定NULL的前驱指针
    
    return TRUE;
}

// 清空链表操作
Status clearList(DulList *pList)
{
    Node *cur; // 当前结点
    Node *temp; // 事先保存下一结点，防止释放当前结点后导致“掉链”
    
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return FALSE;
    }
    
    cur = pList->next; // 指向第一个结点
    while (cur)
    {
        temp = cur->next; // 事先保存下一结点，防止释放当前结点后导致“掉链”
        free(cur); // 释放当前结点
        cur = temp; // 将下一结点赋给当前结点p
    }
    pList->next = NULL; // 头结点指针域指向空
    
    return TRUE;
}

// 遍历链表操作
void traverseList(DulList *pList)
{
    // 判断链表是否存在
    if (!pList)
    {
        printf("list not exist!\n");
        return;
    }
    
    Node *cur = pList->next;
    while (cur != NULL)
    {
        printf("%d ", cur->data);
        cur = cur->next;
    }
    printf("\n");
}

// 获取链表长度操作
int getLength(DulList *pList)
{
    Node *cur = pList;
    int length = 0;
    
    while (cur->next)
    {
        cur = cur->next;
        length++;
    }
    
    return length;
}

int main(int argc, const char * argv[]) {
    
    
    DulList *pList;
    
    // 初始化链表
    initList(&pList);
    printf("初始化链表!\n\n");
    
    // 尾部后插入结点
    printf("尾部后插入元素1、2、3\n\n");
    for (int i = 0; i < 3; i++)
    {
        insertListTail(pList, i+1);
    }
    
    // 头部后插入元素
    insertListHead(pList, 5);
    printf("头部后插入元素5\n\n");
    
    // 插入结点
    insertList(pList, 1, 9);
    printf("在位置1插入元素9\n\n");
    
    // 遍历链表并显示元素操作
    printf("遍历链表：");
    traverseList(pList);
    printf("\n");
    
    // 删除结点
    int val;
    deleteList(pList, 2, &val);
    printf("删除位置2的结点，删除结点的数据为： %d\n", val);
    printf("\n");
    
    // 遍历链表并显示元素操作
    printf("遍历链表：");
    traverseList(pList);
    printf("\n");
    
    // 获得链表长度
    printf("链表长度: %d\n\n", getLength(pList));
    
    // 销毁链表
    clearList(pList);
    printf("销毁链表\n\n");
    
    
    printf("\n\nHello, World!\n");
    return 0;
}


```

打印：

```
初始化链表!

尾部后插入元素1、2、3

头部后插入元素5

在位置1插入元素9

遍历链表：9 5 1 2 3 

删除位置2的结点，删除结点的数据为： 5

遍历链表：9 1 2 3 

链表长度: 4

销毁链表

```

<br/>

***
<br/>



上一回顾：[线性链表(I)](https://www.jianshu.com/p/60c6d1cc5a89)
