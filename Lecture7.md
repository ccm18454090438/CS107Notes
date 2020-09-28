## Lecture 7  堆栈和堆

###  计划
堆栈

堆栈和内存分配

realloc 重新分配


### 堆栈

#### 内存布局

堆栈存储 每次函数调用的变量和参数，函数调用使用的堆栈部分在函数返回是释放

函数调用时，堆栈向下增长  ，函数返回时，堆栈 ，向上收缩

更形象堆栈布局图，调用流程图 见课件

注意，每次函数调用都会产生相应的堆栈空间，唯一的属于自己的空间


还有一点就是当一个函数的堆栈空间使用完毕后，C语言不会对其进行清空处理，而是进行标记，标记该空间是可以使用的。 这样可以提高效率


##### 什么是堆栈溢出


堆栈溢出是说您已经使用了所有的堆栈内存，然而，您还在进行堆栈调度。

例如，函数的递归，如果出口安排不合理，或递归次数过多，都会导致堆栈溢出。


#### 堆栈注意事项

对于需要保留的数据，要存储在主函数的堆栈空间上

    char *create_string(char ch, int num) {
        char new_str[num + 1];
        for (int i = 0; i < num; i++) {
            new_str[i] = ch;
        }
        new_str[num] = '\0';
        return new_str;
    }
    
    int main(int argc, char *argv[]) {
        char *str = create_string('a', 4);
        printf("%s", str);	// want "aaaa"
        return 0;
    }
    
例如上面这个代码
它将需要返回的内容存在了堆栈调用时产生的堆栈空间上，在函数返回后，其就可能会被其他东西是有，导致数据丢失，甚至产生致命错误

尽管在短时间内，你的内容不会改变。
但这种代码终究是炸弹般的存在，

#### 假如我们想要一中在函数调用后不会丢失的内存空间


### 堆和动态内存

除了堆栈之外，还存在一个特殊的内存空间，叫做堆，它需要我们自己进行管理，而且是随着调度向上增长的。

堆是一种动态的内存，在程序的运行过程中，我们可以根据自给的需求，进行内存申请内存释放。


#### malloc 函数


先看一下函数声明  
    
    void *malloc(size_t size);

参数为，所要申请的内存大小，参数返回指向申请得到的内存空间的首地址。

如果没有足够的内存空间，则返回NULL


这样就可以对前面的示例进行修改
得到一个可以放心使用的版本
    
    char *create_string(char ch, int num) {
        char *new_str = malloc(sizeof(char) * (num + 1));
        for (int i = 0; i < num; i++) {
            new_str[i] = ch;
        }
        new_str[num] = '\0';
        return new_str;
    }
    
    int main(int argc, char *argv[]) {
        char *str = create_string('a', 4);
        printf("%s", str);	// want "aaaa"
        return 0;
    }




####  堆中使用断言


    int *array_of_multiples(int mult, int len) {
        int *arr = malloc(sizeof(int) * len);
        assert(arr != NULL);
        for (int i = 0; i < len; i++) {
            arr[i] = mult * (i + 1);
        }
        return arr;
    }
    

这里断言的意思是  
如果发生分配错误，malloc 将返回null 此时，我们直接结束程序

因为，内存分配错误是非常严重的


#### calloc堆分配函数


需要注意的是，calloc在调用时会清空内存，这也是他与malloc的一个重要区别

函数定义

    void *calloc(size_t nmemb, size_t size);
    
注意这里有两个参数，表示申请大小为两数相乘的结果大小的内存空间

课件上的一个应用实例代码

    // allocate and zero 20 ints
    int *scores = calloc(20, sizeof(int));
    // alternate (but slower)
    int *scores = malloc(20 * sizeof(int));
    for (int i = 0; i < 20; i++) scores[i] = 0;



#### strdup堆分配函数

函数声明

    char *strdup(char *s);

此函数需要我们提供一个字符串 ，返回一个以空终止符结尾的堆分配的字符串，

与malloc相比，不需要我们再去专门复制字符串。

示例

    char *str = strdup("Hello, world!");	// on heap
    str[0] = 'h';


#### free清理函数
函数定义

    void free(void *ptr);
    

前面说过，堆分配内存需要我们自己释放

这里就是说的释放方式

在使用时 ，传入要释放的内存的首地址，

    
    	char *bytes = malloc(4);
    	…
    	free(bytes);


#### free 的注意事项

对于一个申请得到的内存块，只能释放一次，即使有多个指针指向他

    char *bytes = malloc(4);
    char *ptr = bytes;
    …
    free(bytes);  //正确
    …
    free(ptr);//错误


内存释放时，必须释放分配所得的地址，同时也不能只释放其中的一部分


    char *bytes = malloc(4);
    char *ptr   = malloc(10);
    …
    free(bytes);	//正确	
    …
    free(ptr + 1);//错误

#### 内存泄漏

内存泄漏是指，你在堆上申请内存，但是没有及时释放，事实上，你的程序有责任去清理申请的内存

这样就会导致内存耗尽。 进而导致内存泄漏，程序崩溃

其实真正的程序运行中，内存泄漏不常见，
只要你按照要求，及时释放掉无用内存空间，几乎不用担心内存泄漏的发生。


###  realloc 重新分配

    void *realloc(void *ptr, size_t size);


先来看一个示例

    char *str = strdup("Hello");
    assert(str != NULL);
    …
    
    // want to make str longer to hold "Hello world!"
    char *addition = " world!";
    str = realloc(str, strlen(str) + strlen(addition) + 1);
    assert(str != NULL);
    
    strcat(str, addition);
    printf("%s", str);
    free(str);


realloc 接受一个现有的指针，并将其放大到请求大小。
并返回最新的指针

这个过程：   如果现有的后面剩余的长度够用，直接延长，不够就转移到申请的新的足够大的空间，并释放掉旧的空间。

在以前，只接受malloc返回的指针
    


同时，我们要确保不要吧指针传递到对分配内存

不要讲指针传递到堆栈内存

（这两句我也不是特别明白，后面要查查资料）



在使用realloc时，我们只需要释放重新分配得到的内存，旧的版本已经被释放了


和上面的例子一样

    
    char *str = strdup("Hello");
    assert(str != NULL);
    …
    // want to make str longer to hold "Hello world!"
    char *addition = " world!";
    str = realloc(str, strlen(str) + strlen(addition) + 1);
    assert(str != NULL);
    strcat(str, addition);
    printf("%s", str);
    free(str);
    


### 总结

    
    void *malloc(size_t size);
    void *calloc(size_t nmemb, size_t size);
    void *realloc(void *ptr, size_t size);
    char *strdup(char *s);
    void free(void *ptr);


比较这几个函数的异同



#### 堆内存分配保证：

失败时为NULL，因此请使用assert(断言)检查

内存是连续的；申请得到的内存要自行管理;除非调用free，否则不会回收

重新分配保留现有数据,释放旧的,保留新的

calloc    zero初始化字节，malloc和realloc不初始化




#### 违法行为：

如果溢出（即访问超出分配的字节）

如果你在free之后使用，或者free在一个位置被调用两次。

如果重新分配/释放非堆地址


#### 堆栈和堆的对比


分配:

堆栈: 快速分配可以特别大,

堆: 可以过大,



灵活程度: 

堆栈: 自动分配,释放.一步声明/初始化

堆: 非常灵活,自己觉得何时分配,分配大小,可重新分配调整大小

安全性:

堆栈: 安全性由编译器予以保证

堆: 出现错误的机会很多,低类型安全性，在完成之前忘记分配/释放，分配错误的大小等，内存泄漏

其余方面


堆栈:
不是特别大,总堆栈大小固定，默认8MB

有些不灵活,无法在运行时添加/调整大小，范围由函数的控制流指定


堆: 控制下的作用域可以精确地确定生存期

#### 补充
在一般情况下,除非必要,一般使用堆栈内存

##### 首选堆分配的情况

1. 你需要一个非常大的分配，堆栈空间可能不足

2. 您需要控制内存的生存期，或者内存必须在函数调用之外保持正常使用

3. 您需要在初始分配内存后调整内存大小

 
 
由于这节函数有点多,多加几个 ,课程后面练习的代码


strdup的实现

    char *myStrdup(char *str) {
        char *heapStr = malloc(strlen(str) + 1);
        assert(heapStr != NULL);
        strcpy(heapStr, str);
        return heapStr;
    }



对于空间释放的合理位置的选择


待填充代码

    char *str = strdup("Hello");
    assert(str != NULL);
    char *ptr = str + 1;
    for (int i = 0; i < 5; i++) {
	     int *num = malloc(sizeof(int));
         assert(num != NULL);
	     *num = i;
	     printf("%s %d\n", ptr, *num);
    }
    printf("%s\n", str);


填充后


    char *str = strdup("Hello");
    assert(str != NULL);
    char *ptr = str + 1;
    for (int i = 0; i < 5; i++) {
	     int *num = malloc(sizeof(int));
         assert(num != NULL);
	     *num = i;
	     printf("%s %d\n", ptr, *num);
         free(num);
    }
    printf("%s\n", str);
    free(str);



### OK本节结束  
