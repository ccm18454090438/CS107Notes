##  Lecture 4 C Strings

显而易见 本节主要介绍字符串

问题 计算机如何表示复杂数据  例如  文本

### 先看大纲

字符

串

常用字符串操作

比较

复制

串联

子字符串

### 字符 （char）

C语言将每个字符表示为一个整数    起名为ASCII值

#### ASCII的一般规则

大写字母按顺序编号

小写字母按顺序编号

数字按顺序编号

小写字母比大写字母多32个


举几个例子

char uppercaseA='A'；//实际上是65

char lowercaseA='a'；//实际上是97

char zeroDigit='0'；//实际为48

bool areEqual = 'A' == 'A';		// true

bool earlierLetter = 'f' < 'c';	// false

char uppercaseB = 'A' + 1;

int diff = 'c' - 'a';	// 2

int numLettersInAlphabet = 'z' – 'a' + 1;

// or

int numLettersInAlphabet = 'Z' – 'A' + 1;

#### 通用函数


| Function      | Description（说明）     |
|---------------|-----------------|
| isalpha\(ch\) |如果ch是'a'到'z'或'A'到'Z' 返回true |
|islower(ch)    |如果ch是'a'到'z' 返回true|
|isupper(ch)    |如果ch是'A'到'Z' 返回true|
|isspace(ch) |如果ch是空格、制表符、换行符等,返回true|
|isdigit(ch)  |如果ch是'0'到'z'或'A'到'9'，返回true|
|toupper(ch)    |返回ch对应的大写字母|
|tolower(ch)    |返回ch对应的小写字母|

##### 注意  
这些方法至是返回一个字符  不会修改现有字符


##### 示例

bool isLetter = isalpha('A');		// true

bool capital = isupper('f');		// false

char uppercaseB = toupper('b');

bool isADigit = isdigit('4');	    // true

### 字符串

C语言中没有字符串的变量类型，他是由有特殊符号的字符数组来表示的         '\0'

在数组中始终要有它的位置

C语言提供了计算字符串长度的函数

int length=strlen（myStr）；//例如13

如果要用这个值要记得保存


当字符串作为参数传入函数中时 ， C传递的是字符串的首地址

对于传递过去的首地址  我们可以像使用字符数组一样使用它


str[0] = 'c';	// modifies original string!

printf("%s\n", str);	// prints cello



#### 常用的string.h函数


| Function      | Description（说明）     |
|---------------|-----------------|
| strlen(str)|返回字符串分（不包括终止字符）|
|strcmp(str1, str2), strncmp(str1, str2, n)|比较两个字符串，如果str1大于str2返回1，如果str小于str2，返回-1，相等返回0，n表示最多在n个字符后停止比较|
|strchr(str, ch)，strrchr(str, ch)|字符查找，strchr 返回str中第一个指向ch字符出现的位置，strrchr 则是返回最后一个，没有找到返回null|
|strstr(haystack, needle)  |字符串搜索，返回一个指针，指向haystack中第一次出现needle的开始，如果在haystack中没有找到needle，则返回NULL |
|strcpy(dst, src),strncpy(dst, src, n)|字符串拷贝将src中的字符串拷贝到dst中，包括终止符，strncpy最多在拷贝n个字符之后停止，并且不添加null终止字符。  |
|strcat(dst, src),strncat(dst, src, n)|字符串连接，一个是全部链接，另一个strncat是最多连接n个字符之后停止连接 总是会在结尾添加终止符  |
|strspn(str, accept),strcspn(str, reject)     |strspn 返回一个长度n，该长度表示在str的前n个字符在accept中都能找到，setcspn也返回一个长度，该长度表示在str中前n个字符在accept中都找不到|

很多字符串函数都假定你给的字符串是符合规范的（有终止符）

不能用<,>，==  等这些符号对字符串进行比较 ，这样比较，其实比较的是地址 

不能 用等号来进行字符串的拷贝   这样拷贝的是地址

关于字符串的拷贝这里提供一个例子

    char str1[6];
    strcpy(str1, "hello");
    
    char str2[6];
    strcpy(str2, str1);
    str2[0] = 'c';
    
    printf("%s", str1);	  // hello
    printf("%s", str2);	  // cello

同时在进行字符串拷贝的时候，我们必须确保目标地址有足够的空间 

例如 

    char str2[6]；//空间不足！
    
    strcpy（strpy，“你好，世界！”）；//覆盖其他内存！


写入超过内存边界称为“缓冲区溢出”。它可以允许安全漏洞！

还有规定数量的拷贝

    // copying "hello"
    char str2[5];				
    strncpy(str2, "hello, world!", 5);	// doesn’t copy '\0'!


当我们的字符串没有空终止符时，我们将无法分辨字符串的结尾在哪里，可能会继续在其他内存地址中进行读取

一般情况下，我们可以自行添加一个空终止符


    //复制“你好”
    
    char str2[6]；//字符串和“\0”的空间
    
    strncpy（str2，“你好，世界！”，5）；//不复制'\0'！
    
    str2[5]='\0'；//添加空终止字符



加一个小题

    int main(int argc, char *argv[]) {
        char str[9];
         strcpy(str, "Hi earth");
        str[2] = '\0';
        printf("str = %s, len = %lu\n",	           str, strlen(str));
        return 0;
    }


会输出什么？？？


不能使用+连接字符串   ，它实际上的是地址

要使用   strcat

特别注意的是  无论是strcat 还是strncat都会去掉旧的空终止符
添加上新的空终止符

举个课件上的例子

    char str1[13]; 		// enough space for strings + '\0' 
    strcpy(str1, "hello ");	
    strcat(str1, "world!");	// removes old '\0', adds new '\0' at end
    printf("%s", str1);	  	// hello world!


你还可以通过自己创建一个char* 指向字符串的某个地址  进而获得其子串 

还是课件上的一个例子


    char chars[8];
    strcpy(chars, "racecar");
    char *str1 = chars;
    char *str2 = chars + 4;
    printf("%s\n", str1);		// racecar
    printf("%s\n", str2);		// car


​    
另外 虽然我们对指针进行了移动  
但还是指向相同的字符串

    char chars[8];
    strcpy(chars, "racecar");
    char *str1 = chars;
    char *str2 = chars + 4;
    str2[0] = 'f';
    printf("%s %s\n", chars, str1);	      // racefar racefar
    printf("%s\n", str2);				// far


我们可以通过char* 指向任何一个字符 ，并对其进行重新赋值，而char[]是不可以的 


例子

    char myString[6];
    strcpy(myString, "Hello");
    myString = "Another string";			// not allowed!



再加一个关于子串的例子

    // Want just "race"
    char str1[8];
    strcpy(str1, "racecar");
    
    char str2[5];
    strncpy(str2, str1, 4);
    str2[4] = '\0';
    printf("%s\n", str1);		// racecar
    printf("%s\n", str2);		// race


​    

还可以通过指针运算来获得子串

    // Want just "ace"
    char str1[8];
    strcpy(str1, "racecar");
    
    char str2[4];
    strncpy(str2, str1 + 1, 3);
    str2[3] = '\0';
    printf("%s\n", str1);		// racecar
    printf("%s\n", str2);		// ace




### OK 本课结束 ·







