- **自定义打印**
- **Chapter Six**
	- [x] 翻转字符串
	- [x] 和为S的数字
- [x] [**快速排序**](http://c.biancheng.net/view/198.html)
- [x] [基本算法](https://juejin.cn/post/6844903885904019470)
- [x] [**LeetCode Cookbook**](https://books.halfrost.com/leetcode/)


<br/>

***
<br/>


># 自定义打印

```

/**
 *打印格式1
 */
#define PrintFormat1(format, ...) \
printf("🍎 🍎 🍎 🍎"\
"\n"\
"文件名: %s"\
"\n"\
"ANSI标准: %d 时间：%s %s  行数: %d 函数:%s \n"\
"参数值：" format "\n", __FILE__, __STDC__, __DATE__, __TIME__,  __LINE__, __FUNCTION__, ##__VA_ARGS__)

/**
 *打印格式2
 */
#define PrintFormat2(format, ...) \
printf("\n🍎 🍎 🍎 🍎\n%d %s %s,  %s[%d]: " format "\n" "🍊 🍊 🍊 🍊\n\n", __STDC__, __DATE__, __TIME__, __FUNCTION__, __LINE__,  ##__VA_ARGS__)


```

**[printf格式化输出](https://blog.csdn.net/xiexievv/article/details/6831194)**

<br/>

***
<br/>


># Chapter Six

<br/>

> - **翻转字符串**

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

    char *reverseSentence(char *questionName,char *sentence);

};

#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"

void reverseString(char *beginP, char *endP);


void reverseString(char *beginP, char *endP) {
    if (beginP == nullptr || endP == nullptr) {
        PrintFormat2("beginP 为 nullptr，或者 endP 为 null");
        return;
    }
    
    while (beginP < endP) {
        char tempVar = *beginP;
        *beginP = *endP;
        *endP = tempVar;
        
        beginP++;
        endP--;
    }
}

char * Chapter6:: reverseSentence(char *questionName,char *sentence) {
   
    if (questionName != nullptr) {
        PrintFormat2("题目： %s", questionName);
    }
    
    if (sentence == nullptr) {
        return  nullptr;
    }
    
    
    char *beginP = sentence;
    char *endP = sentence;
    
    while (*endP != '\0') {
        //指针地址偏移
        endP ++;
    }
    
    endP--;
    
    //翻转整个字符串
    reverseString(beginP, endP);
    
    PrintFormat2("---->> %s", sentence);
    
    //重置为字符串的首字母
    beginP= endP = sentence;
    
    
    while (*beginP != '\0') {//首指针不为空，则句子没结束
        if (*beginP == ' ') {//为空
            beginP ++;//指向下一个元素地址
            endP ++;//指向下一个元素地址
        }else if (*endP == ' ' || *endP == '\0') {
            reverseString(beginP, --endP);//翻转一个单词
            beginP = ++endP;//首指针和尾指针指向下一个单词首字母
        }else {
            endP ++;//指向下一个元素地址
        }
    }
    
    return  sentence;
    
    
}




```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
    char sentence[] = "I am a student.";
    char questionName[] = "page284， 翻转字符串";
     
    Chapter6 chapter6;
    char* result = chapter6.reverseSentence(questionName, sentence);
    PrintFormat2("翻转后的字符串是： %s", result);

    return 0;
}

```

<br/>

输出：

```

🍎 🍎 🍎 🍎
1 Dec  3 2020 23:21:07,  reverseSentence[168]: 题目： page284， 翻转字符串
🍊 🍊 🍊 🍊


🍎 🍎 🍎 🍎
1 Dec  3 2020 23:21:07,  reverseSentence[189]: ---->> .tneduts a ma I
🍊 🍊 🍊 🍊


🍎 🍎 🍎 🍎
1 Dec  3 2020 23:21:08,  main[106]: 翻转后的字符串是： student. a am I
🍊 🍊 🍊 🍊



```



<br/>

>- **和为S的数字**

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {
    void question57_1(char* questionName, int array[], int value, int size = 6);

};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"


void Chapter6:: question57_1(char* questionName, int array[], int value, int size) {
    
    PrintFormat2("%s", questionName);
    
    /*
     C/C++ 传递数组，虽然传递的是首地址地址，但是参数到了函数内
     就成了普通指针，不再是数组首地址了，所以试图在别的函数中得到
     传递数组的长度是行不通的。
     只能先计算好长度后再传过去，进行其他的运算。
     */
    //int count = sizeof(array)/sizeof(int) - 1;
    int count = size - 1;
    
    if (array == nullptr && count < 2) {
        return;
    }
    
    int *firstP = nullptr;
    int *lastP = nullptr;
    int i = 0;
    
    firstP = array;
    //数组值 = 数组首地址 + 个数
    lastP = array+count;
    
    while (i < count) {
        int sum = *firstP + *lastP;
        if (sum < value) {
            ++i;
            firstP = (array+i);
        }else if (sum > value) {
            --count;
            lastP = (array+count);
        }else {
            PrintFormat2("第一个： %d", *firstP);
            PrintFormat2("第二个： %d", *lastP);
            
            break;
        }
    }
    
}



```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
	  Chapter6 chapter6;
	  
	  char questionName[] = "和为S的数字";
    int array[] = {1, 2, 4, 7, 11, 15};
    //计算数组长度或者 int length = sizeof(array)/sizeof(array[0]);
    int length = sizeof(array)/sizeof(int);
    chapter6.question57_1(questionName, array, 15);
    
    
    return 0;
}

```

<br/>

输出：

```
🍎 🍎 🍎 🍎
1 Dec  3 2020 23:22:46,  question57_1[146]: 和为S的数字
🍊 🍊 🍊 🍊


🍎 🍎 🍎 🍎
1 Dec  3 2020 23:22:46,  question57_1[178]: 第一个： 4
🍊 🍊 🍊 🍊


🍎 🍎 🍎 🍎
1 Dec  3 2020 23:22:46,  question57_1[179]: 第二个： 11
🍊 🍊 🍊 🍊

```



<br/>


> - **数组中数字出现的次数**


Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

		/// page275: 56 数组中数字出现的次数
    /// 用来在num的二进制表示中找到最右边是1的位
    /// @param data 数组
    /// @param length 长度
    /// @param num1 数字1
    /// @param num2 数字2
    void findNumsAppearOnce(const char *name, int data[], int length, int *num1, int *num2);
    //返回无符号整型
    unsigned int findFirstBitIs1(int num);
    
};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"


/// 判断在num的二进制表示中从右边数起的indexBit位是不是1
/// @param num <#num description#>
/// @param indexBit <#indexBit description#>
bool isBit1(int num, unsigned int indexBit) {
    num = num >> indexBit;
    return (num & 1);
}

unsigned int Chapter6::findFirstBitIs1(int num) {
    
    int indexBit = 0;
    while (((num & 1) == 0) && (indexBit < 8 * sizeof(int))) {
        num = num >> 1;
        ++ indexBit;
    }
    return  indexBit;
}

void Chapter6:: findNumsAppearOnce(const char *name, int data[], int length, int *num1, int *num2) {
    
    if (name != nullptr) {
        PrintFormat2("%s", name);
    }
    
    if (data == nullptr || length < 2) {
        return;
    }
    
    //异或运算：0 ^ 0 = 0, 0 ^ 1 = 1, 1 ^ 0 = 1, 1 ^ 1 = 0
    int resultExclusiveOR = 0;
    for (int i = 0; i < length; i ++) {
        resultExclusiveOR ^= data[i];
        PrintFormat1("%d", resultExclusiveOR);
    }
    
    unsigned int indexOf1 = findFirstBitIs1(resultExclusiveOR);
    
    *num1 = *num2 = 0;
    for (int j = 0; j < length;  ++j) {
        if (isBit1(data[j], indexOf1)) {
            *num1 ^= data[j];
        }else {
            *num2 ^= data[j];
        }
    }
    
}


```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {

	  Chapter6 chapter6;
    int result1, result2;
    int data[8] = {2, 4, 3, 6, 3, 2, 5, 5};
    chapter6.findNumsAppearOnce("面试题56：数组中数字出现次数", data, sizeof(data)/sizeof(int), &result1, &result2);
    PrintFormat2("result1: %d, result2: %d", result1, result2);
    
    return 0;
}

```

<br/>

输出：

```
🍎 🍎 🍎 🍎
1 Dec  3 2020 23:31:01,  findNumsAppearOnce[150]: 面试题56：数组中数字出现次数
🍊 🍊 🍊 🍊

🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：2
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：6
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：5
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：3
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：0
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：2
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：7
🍎 🍎 🍎 🍎
文件名: /Users/huanggang/Documents/GitHub/MLC/C2/C2/Chapter6.cpp
ANSI标准: 1 时间：Dec  3 2020 23:31:01  行数: 161 函数:findNumsAppearOnce 
参数值：2

🍎 🍎 🍎 🍎
1 Dec  3 2020 23:31:02,  main[100]: result1: 6, result2: 4
🍊 🍊 🍊 🍊

```


<br/>

> - **二叉树的深度**

![z24](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z24.png)

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

public:
    //二叉树结构体
    typedef struct BinaryTree {
        char value;
        struct BinaryTree *leftChild;
        struct BinaryTree *rightChild;
    }BinaryTree, *BinaryTreeNode;
    char characters[10] = "AB#D##C##";
    //起始变量值
    int number = 0;

	/// page272: 55 获取二叉树的深度
    /// @param rootNode 根结点
    int getBinaryTreeDepth(BinaryTree *rootNode);

};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"


//创建二叉树
void Chapter6:: createBinaryTree(BinaryTreeNode *binaryTree, int index) {
	char data = characters[number++];
	
	if (data == '#' || data == '\0') {
	    //当其设置为NULL时，其右子树的指针容易变为野指针导致错误
	    *binaryTree = NULL;
	}else {
	    //if (*binaryTree == NULL) {//不要加判断否则程序crash
	    //malloc 函数返回的是 void * 类型，如果你写成：p = malloc (sizeof(int)); 则程序无法通过编译，报错：“不能将 void* 赋值给 int * 类型变量”。所以必须通过 (int *) 来将强制转换。
	    *binaryTree = (BinaryTree*)malloc(sizeof(BinaryTree));
	    //}
	    if (!(*binaryTree)) {
	        exit(OVERFLOW);
	    }
	    
	    (*binaryTree)->value = data;
	    createBinaryTree(&(*binaryTree)->leftChild,++index);
	    //当 (*binaryTree)->rightChild 递归设置为NULL，返会再次打印 (*binaryTree)->rightChild 其值时，发现已经有值了，它的指针变为野指针了
	    createBinaryTree(&(*binaryTree)->rightChild, ++index);
	}
	
}

//二叉树深度
int Chapter6:: getBinaryTreeDepth(BinaryTree *rootNode) {
	if (rootNode == nullptr) {
	    return  0;
	}
	
	int leftDep= this->getBinaryTreeDepth(rootNode->leftChild);
	int rightDep = this->getBinaryTreeDepth(rootNode->rightChild);
	
	
	return (leftDep>rightDep) ? (leftDep+1) : (rightDep+1);
}



```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
    /**
     *二叉树的深度
     */
     Chapter6 chapter6;
     BinaryTree *binaryTree = nullptr;
     chapter6.createBinaryTree(&binaryTree);
     int depth = this->getBinaryTreeDepth(binaryTree);
     PrintFormat2("二叉树的深度为： %d", depth);
     
    return 0;
}

```

<br/>

输出：

```
🍎 🍎 🍎 🍎
1 Dec  4 2020 18:41:52,  chapter6Run[95]: 二叉树的深度为： 3
🍊 🍊 🍊 🍊
```


<br/>


> - **二叉搜索树的第 K 大节点**

![z24](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z24.png)


Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

	/// 二叉树创建
    /// @param binaryTree 根结点指针
    void createBinaryTree(BinaryTreeNode *binaryTree, int index = 0);
    
	 /// P269 算法54: 二叉搜索树的第 K 大节点
    /// @param k 第 k 节点
    /// @param pRoot 根结点指针变量
    BinaryTree *kthNode(BinaryTree *pRoot, unsigned int k);
    
};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"



//创建二叉树
void Chapter6:: createBinaryTree(BinaryTreeNode *binaryTree, int index) {
    char data = characters[number++];
    
    if (data == '#' || data == '\0') {
        //当其设置为NULL时，其右子树的指针容易变为野指针导致错误
        *binaryTree = NULL;
    }else {
        //if (*binaryTree == NULL) {//不要加判断否则程序crash
        //malloc 函数返回的是 void * 类型，如果你写成：p = malloc (sizeof(int)); 则程序无法通过编译，报错：“不能将 void* 赋值给 int * 类型变量”。所以必须通过 (int *) 来将强制转换。
        *binaryTree = (BinaryTree*)malloc(sizeof(BinaryTree));
        //}
        if (!(*binaryTree)) {
            exit(OVERFLOW);
        }
        
        (*binaryTree)->value = data;
        createBinaryTree(&(*binaryTree)->leftChild,++index);
        //当 (*binaryTree)->rightChild 递归设置为NULL，返会再次打印 (*binaryTree)->rightChild 其值时，发现已经有值了，它的指针变为野指针了
        createBinaryTree(&(*binaryTree)->rightChild, ++index);
    }
    
}


Chapter6::BinaryTree * Chapter6:: kthNode(Chapter6::BinaryTree *pRoot, unsigned int k) {
    if (pRoot == NULL || k == 0) {
        return nullptr;
    }
    
    return KthNodeCore(pRoot, k);
}




```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
  
   Chapter6 chapter6;
	 // 二叉搜索树的第 K 大节点
	 //结构体指针需要申请内存空间才可以使用
	 BinaryTree *binaryTree = NULL;
	 
	 chapter6.createBinaryTree(&binaryTree);
	 PrintFormat2("赋值后：value: %c, leftNode:%p, rightNode: %p", binaryTree->value, binaryTree->leftChild, binaryTree->rightChild);
	 
	 BinaryTree *searchNode = this->kthNode(binaryTree, 4);
	 PrintFormat2("%c", searchNode->value);
    return 0;
}

```

<br/>

输出：

```

```

<br/>

>- 数字在排序数组中出现的次数

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {
	/// P263  算法53： 数字在排序数组中出现的次数
    /// @param array 传递的数组
    /// @param num 指定的值
    /// @param endIndex 数组终止索引
    /// @param startIndex 数组起始索引，默认值为0，放在最后规定
    int getSpecifyNumCount(int *array, int num, int endIndex, int startIndex = 0);
    int getLastNumCount(int *array, int num, int endIndex, int startIndex = 0);

};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"


//实现处把默认值省略
//左边的索引
int Chapter6:: getSpecifyNumCount(int *array, int num, int endIndex, int startIndex) {
    
    //数组违法，不存在
    if (endIndex < startIndex) {
        return  -1;
    }
    
    //sizeof() 获取数组元素个数
    int middle = (endIndex + startIndex)/2;
    
    if (*(array + middle) == num) {//指针获取数组的值
        //array 表示是数组的首位地址
        //*(array+middle - 1) 取第 middle - 1 位元素
        //(*(array+middle - 1) < num && middle > 0) 表示当数组元素为中间时我们要取到最开始的元素，所以要进行判断
        //middle == 0 当数组是第 0 位时，那它就是第一位
        if ((*(array+middle - 1) < num && middle > 0) || middle == 0) {
            return  middle;
        }else {
            //当索引是中间或者最后相同的元素时
            endIndex = middle - 1;
        }
    }else if (*(array+middle) > num) {
        //取上限的下一位，因为middle已经比过了
        endIndex = middle - 1;
    }else if(*(array+middle) < num) {
        //取下限的上一位，因为middle已经比过了
        startIndex = middle + 1;
    }
    
    return  getSpecifyNumCount(array, num, endIndex, startIndex);
}

//右边的索引
int Chapter6:: getLastNumCount(int *array, int num, int endIndex, int startIndex) {
    
    //数组违法，不存在
    if (endIndex < startIndex) {
        return  -1;
    }
    
    //sizeof() 获取数组元素个数
    int middle = (endIndex + startIndex)/2;
    
    if (*(array + middle) == num) {//指针获取数组的值
        //array 表示是数组的首位地址
        //*(array+middle - 1) 取第 middle - 1 位元素
        //(*(array+middle - 1) < num && middle > 0) 表示当数组元素为中间时我们要取到最开始的元素，所以要进行判断
        //middle == 0 当数组是第 0 位时，那它就是第一位
        if ((*(array+middle + 1) > num && middle < (endIndex + startIndex)) || middle == (endIndex + startIndex)) {
            return  middle;
        }else {
            //当索引是中间或者最后相同的元素时
            startIndex = middle + 1;
        }
    }else if (*(array+middle) > num) {
        endIndex = middle - 1;
    }else if(*(array+middle) < num) {
        startIndex = middle + 1;
    }
    
    return  getLastNumCount(array, num, endIndex, startIndex);
}







```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
    Chapter6 chapter6;
    
    int array[8] = {1, 2, 3, 3, 3, 3, 4, 5};
    int count = 0;
    int leftCount = chapter6.getSpecifyNumCount( array, 3, 7);
    int rightCount = chapter6.getLastNumCount(array, 3, 7);
    if (leftCount > -1 && rightCount > -1) {
        count = rightCount - leftCount + 1;
    }
    PrintFormat2("次数：%d", count);
    
    return 0;
}

```

<br/>

输出：

```

1 Dec 14 2020 19:21:59,  main[39]: 次数：4
🍊 🍊 🍊 🍊

sh: pause: command not found
Program ended with exit code: 0

```

<br/>

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"

void Chapter6:: chapter6Run() {



}

```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
    
    return 0;
}

```

<br/>

输出：

```

```


<br/>

Chapter6.hpp

```
#ifndef Chapter6_hpp
#define Chapter6_hpp
#include <stdio.h>

class Chapter6 {

};



#endif /* Chapter6_hpp */
```

Chapter6.cpp

```
#include "Chapter6.hpp"
#include <stdio.h>
#include <stdlib.h>
#include "math.h"
#include "string.h"

void Chapter6:: chapter6Run() {



}

```


Main.cpp

```

#include <iostream>
#include "Chapter6.hpp"

int main(int argc, const char * argv[]) {
  
    
    return 0;
}

```

<br/>

输出：

```

```