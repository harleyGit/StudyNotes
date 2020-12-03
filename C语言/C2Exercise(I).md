- **自定义打印**
- **Chapter Six**
	- [x] 翻转字符串
	- [x] 和为S的数字


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

==**[printf格式化输出](https://blog.csdn.net/xiexievv/article/details/6831194)**==

<br/>

***
<br/>


># Chapter Six

<br/>

- ==**翻转字符串**==

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


void Chapter6:: chapter6Run() {

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

- ==**和为S的数字**==

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

void Chapter6:: chapter6Run() {

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

void Chapter6:: chapter6Run() {

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