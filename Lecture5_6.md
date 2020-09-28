
## Lecture 5  More C Strings
显然  本节介绍更多关于字符串的内容

### 字符串搜索 

示例 1

    char daisy[6];
    strcpy(daisy, "Daisy");
    char *letterA = strchr(daisy, 'a');
    printf("%s\n", daisy);			// Daisy
    printf("%s\n", letterA);		// aisy

这里用到了strchr 函数   前面有介绍


示例 2

    char daisy[10];
    strcpy(daisy, "Daisy Dog");
    char *substr = strstr(daisy, "Dog");
    printf("%s\n", daisy);			// Daisy Dog
    printf("%s\n", substr);			// Dog

这里用到了strstr函数


示例 4

    char daisy[10];
    strcpy(daisy, "Daisy Dog");
    int spanLength = strspn(daisy, "aDeoi");		// 3

  这里用到了  strspn函数 
  
 

示例 5

    char daisy[10];
    strcpy(daisy, "Daisy Dog");
    int spanLength = strcspn(daisy, "driso");		// 2

这里是用到了  strcspn函数

强调一点   不可以用sizeof 求字符串的长度


#### 字符串作为参数

在以字符串为参数进行传递时  ， C语言传递的是地址  以char* 的形式传递

所以 在编写形参时   char n[]    和 char *n 是一个意思



#### 字符串数组 
直接点就是指针数组  

定义方式

    char *stringArray[5];	// 存储五个char *

也可以用

    char *stringArray[] = {
	"Hello",
	"Hi",
	"Hey there"
    };
    
这种方式来定义

我们可以用括号[]对字符串进行访问

    printf("%s\n", stringArray[0]);
打印字符串数组中的第一个数组


当我们将字符串数组当做参数进行传递时

C语言自动转换为双重指针（char **） 进而进行传递


课件上面有一个关于字符串指针

另外课件上提到了一个关于追踪内存读取的工具  Valgrind

感兴趣的可以去试一试


### 指针

指针   就是指存储内存地址的变量
大小为8个字节

后续的堆分配内存还要用到相关内容

#### 先来理解一下内存   
内存 简单说就是一个超级大的字节数组

每个字节都对应一个唯一的地址
通常情况下用十六进制表示

通过指针我们可以在别的函数中修改接受到的参数  

例

    void myFunc(int& num) {
	    num = 3;
    }

    int main(int argc, char *argv[]) {
	    int x = 2;
	    myFunc(x);
	    printf("%d", x);	// 3!
	    ...
    }
    
    
面对复杂的数据传参  指针可以很大的提高效率  ，而且还可以对其进行一定的修改  

C语言   传值传地址


还要提一下 * 运算符  &运算符   简单的说就是取地址

例如

    int x=1;
    int *y=&x;//定义一个指针  赋值为x的地址
    int z=*y;// 将y存储的地址对应的值拿出来赋给z   因为y存储的是x的地址 相当于 int z=x;


多提两嘴  指针的应用很多   
例如  
 
 通过对指针的偏移量获取子串


还有要注意的就是   
1.我们不能对创建的字符数组进行赋值，因为它不说指针，当然他在传参过程中是转换为指针进而进行传递的
2.
对于我们通过char *="****"   进行赋值得到的字符串是不能被改变的  因为它位于计算机的一个特殊的位置，不可改变



更多指针应用的详细图解等  见课件


#### 字符串在内存中

其实有点点上面都有提到

这里总结一下字符串在内存中的一些要点

1. 如果我们是通过char[] 的形式创建的字符串 ，那么我们是可以对其进行那个改变的，它位于电脑的堆栈空间中
2. 对于已经创建的字符数组 我们不能对数组名进行重新赋值 ，它只能对其分配得到的内存块进行引用，同时字符数组的大小一经创建就是确定的了，不可进行修改
3. 在以字符数组为参数进行传参时，是自动转换为字符指针进行传递的
4. 我们用char* 创建的字符串是不可修改的，他存放于数据段中
5. char* 是可以重新赋值的
6. 可以通过对指针添加偏移量获得子串
7. 对于通过字符串指针就行的字符修改 是持久的，可以保持到函数调度外


### OK  本节结束



## Lecture 6 
### More Pointers and Arrays

看标题就应该清楚这节的主要内容了
     ---- 更多关于指针和数组的内容


思考一个问题，我们如和在内存中管理所有类型的内存？？？？？


### 指针和参数

在重提一下   
指针----存储内存地址的变量，大小8个字节（忘记的往回看）



指针的好多内容前面已经说过了，重复是就略过了

C语言传递参数的方式 
一般情况下就是拷贝一个副本

通常情况下 ，如果你正在进行某些操作，同时，并不关心对传入数据进行的任何更改，可以传递数据本身

反之如果我们想要修改某部分内容，那么应该传递其地址，这样可以对其进行持久性的修改


如果我们的参数是一个地址 ，那么当我们取值的时候 ，实际上是先到达其指定的地址，然后将地址内的内容取出


其实  char* 真正意义上是指向单个字符的指针，
不过我们通常是让他后面跟着一堆字符，然后以一个\0为终止标志 进行表示字符串的

如果我们在函数调用中修改了字符串的部分内容，那么，这些修改将持续到函数调用以外


#### 在字符串作为参数时
1. 如果传递的是char*  拷贝其指向的地址
2. 如果传递的是字符数组传递字符数组中首元素地址


简单说 ：：
1. 不关心修改 随便传
2. 想要修改 传地址


### 双重指针


有时，我们想要修改的是指针本身 ，而非 他说指向的数据

这时，我们就要用到双重指针了，也就是指向指针的指针 

这里有一个具体的流程实例   （见课件  ）


### 内存中的数组

数组的内存地址是连续的
声明数组后  堆栈上分配连续的内存空间
，   数组是存储在堆栈上的


数组变量引用指定的内存块，数组不能进行重新分配
数组创建后，大小不可改变

数组作为参数时，会传递首元素地址
所以，在函数中，我们不能用sizeof获取 数组的长度，他其实是一个指针

    void myFunc(char *myStr) {
	    int size = sizeof(myStr); // 8
    }

    int main(int argc, char *argv[]) {
	    char str[3];
	    strcpy(str, "hi");
    	int size = sizeof(str);   // 3
    	myFunc(str);
    	...
    }



（sizeof 返回数组的长度或者指针的8  ）


我们可以使用一个指针等于一个数组，让指针指向其首元素地址

    int main(int argc, char *argv[]) {
    	char str[3];
    	strcpy(str, "hi");
    	char *ptr = str;	
	    ...
    }
    
    


我们也可以创建一个指针数组，将多个字符串组合到一块




### 指针运算

先看一个例子

    char *str = "apple";	// e.g. 0xff0
    char *str1 = str + 1;	// e.g. 0xff1
    char *str3 = str + 3;	// e.g. 0xff3
    
    printf("%s", str);		// apple
    printf("%s", str1);	// pple
    printf("%s", str3);	// le
    
    
注意  指针运算  不是以字节为单位的
而是 其所指向的数据类型的大小


    // nums 指向一个int数组
    int *nums = …			// e.g. 0xff0
    int *nums1 = nums + 1;	// e.g. 0xff4
    int *nums3 = nums + 3;	// e.g. 0xffc
    
    printf("%d", *nums);	// 52
    printf("%d", *nums1);	// 23
    

  \*   取消引用  
  例
 
    char *str = "apple";	// e.g. 0xff0

    // both of these add two places to str,
    // and then dereference to get the char there.
    // E.g. get memory at 0xff2.
    char thirdLetter = str[2];		// 'p'
    char thirdLetter = *(str + 2);	// 'p' 课件上的示例

 
 两个指针的运算结果  返回的不是字节差  
 
 例如 
 
     // nums points to an int array
    int *nums = …				// e.g. 0xff0
    int *nums3 = nums + 3;		// e.g. 0xffc
    int diff = nums3 - nums;	// 3
    
这里的3的意思是这两个指针偏移了三个单位（其所指向的数据类型）


关于为什么指针在运算时知道自己应该偏移几个字节  ，
这里直接看课件上给定解释吧

    At compile time, C can figure out the sizes of different data types, and the sizes of what they point to.
    For this reason, when the program runs, it knows the correct number of bytes to address or add/subtract for each data type.

    翻译: 
    在编译时，C可以计算出不同数据类型的大小，以及它们所指向的数据类型的大小    。
    因此，当程序运行时，它知道每个数据类型要寻址或加/减的正确字节数。
    
    
### 其他内容   ： const, struct and ternary

#### const 
使用const 声明全局常量   这样声明的常量创建后不可修改

例

    const double PI = 3.1415;
    const int DAYS_IN_WEEK = 7;
    
    int main(int argc, char *argv[]) {
    	…
    	if (x == DAYS_IN_WEEK) {
    		…
    	}
    	…
    }
    

使用const标志的数据不能修改

例

    char str[6];
    strcpy(str, "Hello");
    const char *s = str;
    
    // 无法使用s更改它指向的字符
    s[0] = 'h';
    

注意   const 声明的指针   其指向不可修改 ，但其本身可以修改


课件上的一个例子

    // This function promises to not change str’s characters
    int countUppercase(const char *str) {
    	int count = 0;
	    for (int i = 0; i < strlen(str); i++) {
	    	if (isupper(str[i])) {
	    		count++;
	    	}
	    }
	    return count;
    }
    


对于不同的变量类型你  const 的解释是不同的

    // cannot modify this char
    const char c = 'h';
    
    // cannot modify chars pointed to by str
    const char *str = … 
    
    // cannot modify chars pointed to by *strPtr
    const char **strPtr = …
    

#### 结构体

结构体是定义一组有其他变量类型组成的新的变量类型的方法

这一块比较简单  

直接看书上的例子

    struct date {		// declaring a struct type
        int month;
        int day;		// members of each date structure
    };
    …
    
    struct date today;	               // construct structure instances
    today.month = 1;
    today.day = 28;
    
    struct date new_years_eve = {12, 31};   // shorter initializer syntax
    
    
##### 关于 typedef的用法
简单说就是改名   给某个变量类型起个新名字

例子

    typedef struct date {		
        int month;
        int day;		
    } date;
    …
    
    date today;	
    today.month = 1;
    today.day = 28;
    
    date new_years_eve = {12, 31};  

在结构体作为参数的时候 采用传值的方式   拷贝整个副本


结构体也是存在指针的

例

    void advance_day(date *d) {
    	(*d).day++;
    }
    
    int main(int argc, char *argv[]) {
    	date my_date = {1, 28};
    	advance_day(&my_date);
    	printf("%d", my_date.day);	// 29
    	return 0;
    }
    
对于结构体指针的应用  可以使用* 也可以采用另一种方式  ->


    void advance_day(date *d) {
    	d->day++;		// equivalent to (*d).day++;
    }
    
    int main(int argc, char *argv[]) {
    	date my_date = {1, 28};
    	advance_day(&my_date);
    	printf("%d", my_date.day);	// 29
    	return 0;
    }


同时   ，我们自己定义的结构体也是可以作为函数返回值类型的


    date create_new_years_date() {
    	date d = {1, 1};
         return d;		// or return (date){1, 1};
    }
    
    int main(int argc, char *argv[]) {
    	date my_date = create_new_years_date();
     	printf("%d", my_date.day);	// 1
    	return 0;
    }


对于结构体  ，使用sizeof  返回结构体类型的大小
即所有组成部分的大小和


    typedef struct date {
    	    int month;
    	    int day;       
    	} date;
    
    int main(int argc, char *argv[]) {
    	int size = sizeof(date);	// 8	
         	return 0;
    }

同时 结构体也是可以像基本类型一样创建数组

    typedef struct my_struct {
    	int x;
    	char c;
    } my_struct;
    
    …
    
    my_struct array_of_structs[5];
    

需要注意的是  结构体数组  定义之后 要进行初始化  也别说对于有字符串等的情况

对于结构体数组可以一块初始化 ，也可以对于单个字段初始化


同时初始化

    typedef struct my_struct {
    	int x;
    	char c;
    } my_struct;
    
    …
    
    my_struct array_of_structs[5];
    array_of_structs[0] = (my_struct){0, 'A'};



逐个初始化

    typedef struct my_struct {
    	int x;
    	char c;
    } my_struct;
    
    …
    my_struct array_of_structs[5];
    array_of_structs[0].x = 2;
    array_of_structs[0].c = 'A';


##### 三目运算符

三目运算符  ：   是否为真？ 为真取值:为假取值


更具体的代码示例

    int x;
    if (argc > 1) {
    	x = 50;
    } else {
    	x = 0;
    }
    
    // equivalent to
    int x = argc > 1 ? 50 : 0;

### OK  本节结束
