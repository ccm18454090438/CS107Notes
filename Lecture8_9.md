## Lecture 8 C语言泛型，void型指针


### 下先来思考一个问题  ： 我们如何编写可以处理任何数据类型的代码


本节内容主要是：
1. 学习编写可以处理任何类型数据的代码
2. 了解void * 的使用，同时避免出现错误


### 泛型概述

泛型代码能够减少代码重复，这意味着在出现错误时间，只需要在一个地方进行修改。一般情况下应用于： 函数排序，搜索任意类型的数组，释放内存等等


### 泛型交换

我们来一个情景再现：

现在，我们要写一个交换int类型数据的函数

    void swap_int(int *a, int *b) {
        int temp = *a;
        *a = *b;
        *b = temp;
    }
    
    int main(int argc, char *argv[]) {
        int x = 2;
        int y = 5;
        swap_int(&x, &y);
        // want x = 5, y = 2
        printf("x = %d, y = %d\n", x, y);	
        return 0;
    }

OK  这是我们的代码

那么现在，我们有不想交换int了 ， 我们要交换short

怎么办呢？？重写！！

    void swap_short(short *a, short *b) {
        short temp = *a;
        *a = *b;
        *b = temp;
    }
    
    int main(int argc, char *argv[]) {
        short x = 2;
        short y = 5;
        swap_short(&x, &y);
        // want x = 5, y = 2
        printf("x = %d, y = %d\n", x, y);	
        return 0;
    }


OK  我们现在发现，我们要交换的其实是字符串 
怎么办 ？？？？ 继续重写

    void swap_string(char **a, char **b) {
        char *temp = *a;
        *a = *b;
        *b = temp;
    }
    
    int main(int argc, char *argv[]) {
        char *x = "2";
        char *y = "5";
        swap_string(&x, &y);
        // want x = 5, y = 2
        printf("x = %s, y = %s\n", x, y);	
        return 0;
    }

OK  现在问题又来了 ，

我们现在有好多种自定义的结构体要进行交换  。 **怎么说  ？？？**

OK现在问题来了。

我们想编写一个函数，能够交换任意类型的两个值

前面我们写的交换全部是同一种模式
1. 传递两个指针进入函数。指向要交换的两个值
2. 创建临时变量，作为交换媒介
3. 进行交换



OK  现在，建模

    void swap(pointer to data1, pointer to data2) {
        store a copy of data1 in temporary storage
        copy data2 to location of data1
        copy data in temporary storage to location of data2
    }

上面是课件版的模型  

OK 先看参数  ，两个指针，不多说

第二行

建立临时空间，存储data1的副本

那么问题哦就来了，不同的数据类型，大小不一样啊，比如，int四个字节，char一个字节，等等  ！！


在前面进行拷贝时，C语言自己知道应该拷贝多少字节，因为它知道数据类型。

那么我们的这种泛型要求的是一种**++不关心数据类型++**制作变量副本，有没有这样的方法呢？？？


处于这样的目的，我们修改参数类型为void * （不关心类型）
这样，我们就不知道数据类型的大小，所以我们增加一个记录大小的参数
为了过得副本，我们再获得对应大小的内存空间


修改后的代码：

    void swap(void *data1ptr, void *data2ptr, size_t nbytes) {
        char temp[nbytes];
        // store a copy of data1 in temporary storage
        temp = *data1ptr; ???
        // copy data2 to location of data1
        // copy data in temporary storage to location of data2
    }


这样就出问题了，我们怎么制作副本，C语言赋值，开玩笑，C语言都不知道什么类型，你怎么赋值

下面就要介绍一个函数了：

### memcpy

函数声明： 

    void *memcpy(void *dest, const void *src, size_t n);


简单说一下该函数的作用，从src指向的地址 向dest指向的地址拷贝
n个字节的内容

memcpy 的参数一定要使用指向要处理的数据的指针，才能正常的进行后续操作，程序才能真正的知道，从哪拷贝，拷贝到那里，拷贝多少内容


### menmove 

    void *memmove(void *dest, const void *src, size_t n);

memmove 和memcpy具有相同的功能，不同的是前者支持重叠区域 

该函数的作用是 ： 从src指向的地址想dest指向的地址拷贝n个字节的内容  。同时返回dest


例如  ：

    #include <stdio.h>
    #include<string.h>
    int main()
    {
    	char a[]="123456789";
    	memmove(a,a+4,4);
    	printf("%s\n",a);// 567856789
    	return 0;
    }


再回到泛型

我们在定义指针类型时，不能使用void* 这样C语言不知道自己指向的是什么。

那么，应怎么合理的使用 memcpy 。memmove ，来便于我们编写泛型呢？？？

    void swap(void *data1ptr, void *data2ptr, size_t nbytes) {
        char temp[nbytes];
        // store a copy of data1 in temporary storage
        memcpy(temp, data1ptr, nbytes);
        // copy data2 to location of data1
        // copy data in temporary storage to location of data2
    }

我们可以使用memcpy直接拷贝字节  ，不关心类型，  反正我们已经有了长度

OK  ，下面有同样的方法实现数据交换

    void swap(void *data1ptr, void *data2ptr, size_t nbytes) {
        char temp[nbytes];
        // store a copy of data1 in temporary storage
        memcpy(temp, data1ptr, nbytes);
        // copy data2 to location of data1
        memcpy(data1ptr, data2ptr, nbytes);
        // copy data in temporary storage to location of data2
        memcpy(data2ptr, temp, nbytes);
    }


这样就是，我们直接操作内存，不关心数据类型，进行数据交换
。这样我们只需要知道，大小以及位置即可


### 泛型陷阱

#### void*
void* 是很危险的，对于任何一种数据类型，例如int  ，我们总不能交换半个int吧

所以如果操作稍有不慎就会导致错误


OK，现在 ，梳理几个问题  

    什么样的变量类型代表“泛型指针”？
    
    我们可以使用哪种变量类型在堆栈上创建特定数量的空间？
    
    我们如何将通用内存从一个位置复制到另一个位置？
    
    memcpy和memmove有什么区别？
    
    C语言中泛型函数的好处是什么？挑战是什么？

这些内容前面都有提过，回忆一下



### 泛型数组交换


现在，编写一个函数，交换int 数组的首尾

    void swap_ends_int(int *arr, size_t nelems) {
        int tmp = arr[0];
        arr[0] = arr[nelems – 1];
        arr[nelems – 1] = tmp;
    }
    
    int main(int argc, char *argv[]) {
        int nums[] = {5, 2, 3, 4, 1};
        size_t nelems = sizeof(nums) / sizeof(nums[0]);
        swap_ends_int(nums, nelems);
        // want nums[0] = 1, nums[4] = 5
        printf("nums[0] = %d, nums[4] = %d\n", nums[0], nums[4]);	
        return 0;
    }


现在，我们是不是可以用到刚才我们编写好的泛型交换函数呢

    void swap_ends_int(int *arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }
    
    int main(int argc, char *argv[]) {
        int nums[] = {5, 2, 3, 4, 1};
        size_t nelems = sizeof(nums) / sizeof(nums[0]);
        swap_ends_int(nums, nelems);
        // want nums[0] = 1, nums[4] = 5
        printf("nums[0] = %d, nums[4] = %d\n", nums[0], nums[4]);	
        return 0;
    }

。
现在吧数据类型变一下试一试：

    void swap_ends_int(int *arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }
    
    void swap_ends_short(short *arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }
    
    void swap_ends_string(char **arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }
    
    void swap_ends_float(float *arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }


OK  ，现在看这个代码 

    void swap_ends(void *arr, size_t nelems) {
        swap(arr, arr + nelems – 1, sizeof(*arr));
    }
 
思考一下，这个代码是通用的那？？  是否可以正常使用

，不行！！

1. 前面有说过，指针的运算偏移量的大小取决于其所指向的数据类型的大小。这里没有类型，怎么运算
2. 而且，我没让你也不知道数据类型的大小，后操作也无法进行



所以我们要添加参数，解决上述问题

OK  看修正版

    void swap_ends(void *arr, size_t nelems, size_t elem_bytes) {
        swap(arr, arr + nelems – 1, elem_bytes);
    }

现在还有问题吗？？

当然有，偏移量问题

在不同的数据类型下，偏移量的字节大小是不同的。
很明显，这里错了，这里没有类型，

所以我们要知道元素的大小，才能正常使用指针运算，才能完成泛型函数

OK 新版来了

    void swap_ends(void *arr, size_t nelems, size_t elem_bytes) {
        swap(arr, arr + (nelems – 1) * elem_bytes, elem_bytes);
    }

那么问题来了：  前面有说过C语言不能对void* 指针进行运算

怎么解决？？？？

因为我们偏移的是字节量   

所以我们用  char *


    void swap_ends(void *arr, size_t nelems, size_t elem_bytes) {
        swap(arr, (char *)arr + (nelems – 1) * elem_bytes, elem_bytes);
    }


OK  这个代码就能够是实现任何数据类型的数组交换

OK 看几个应用示例：



    short nums[] = {5, 2, 3, 4, 1};
    size_t nelems = sizeof(nums) / sizeof(nums[0]);
    swap_ends(nums, nelems, sizeof(nums[0]));


    short nums[] = {5, 2, 3, 4, 1};
    size_t nelems = sizeof(nums) / sizeof(nums[0]);
    swap_ends(nums, nelems, sizeof(nums[0]));


    char *strs[] = {"Hi", "Hello", "Howdy"};
    size_t nelems = sizeof(strs) / sizeof(strs[0]);
    swap_ends(strs, nelems, sizeof(strs[0]));


    mystruct structs[] = …;
    size_t nelems = …;
    swap_ends(structs, nelems, sizeof(structs[0]));


### 泛型堆栈

OK，现在尝试用泛型实现一个堆栈

先来熟悉一下堆栈


堆栈是表示一堆东西的数据结构。

对象可以被推到堆栈顶部或从堆栈顶部弹出。

只能访问堆栈的顶部；堆栈中的其他对象不可见。

主要业务：

push（value）：将元素添加到堆栈顶部

pop（）：移除并返回堆栈中的顶层元素

peek（）：返回（但不要移除）堆栈中的顶层元素




一把情况下，C语言使用链表实现堆栈


看一下，一般堆栈的实现

    typedef struct int_node {
        struct int_node *next;
        int data;
    } int_node;
    
    typedef struct int_stack {
        int nelems;
        int_node *top; 
    } int_stack;


由于我们不知道的数据类型是不确定的，所有我们要做修改

同时，我们需要知道数据类型的字节大小

    typedef struct int_node {
        struct int_node *next;
        void *data;
    } int_node;
    
    typedef struct stack {
        int nelems;
        int elem_size_bytes;
        node *top; 
    } stack;

下面看一般堆栈的功能

    int_stack_create（）：在堆上创建新堆栈并返回指向它的指针
    
    int_stack_push（int_stack*s，int data）：将数据推送到堆栈上
    
    int_stack_pop（int_stack*s）：弹出并返回最顶层的堆栈元素


#### int_stack_create

    int_stack *int_stack_create() {
        int_stack *s = malloc(sizeof(int_stack));
        s->nelems = 0;
        s->top = NULL;
        return s;
    }

我们要怎们修改来实现我们的泛型要求呢


    stack *stack_create(int elem_size_bytes) {
        stack *s = malloc(sizeof(stack));
        s->nelems = 0;
        s->top = NULL;
        s->elem_size_bytes = elem_size_bytes;
        return s;
    }



OK， 第一个功能实现

#### int_stack_push


一般实现

    void int_stack_push(int_stack *s, int data) {
        int_node *new_node = malloc(sizeof(int_node));
        new_node->data = data;
    
        new_node->next = s->top;
        s->top = new_node;
        s->nelems++;
    }

由于我们的数据类型是不确定的，所以不能传递数据本身，因为太多变了。


所以传递指针
，同时，我们要自己制作数据的副本，不能直接赋值传递得到的指针
因为，该指针的生命周期我们无法管理，

所以，我们的实现如下

    void stack_push(stack *s, const void *data) {
        node *new_node = malloc(sizeof(node));
        new_node->data = malloc(s->elem_size_bytes);
        memcpy(new_node->data, data, s->elem_size_bytes);
    
        new_node->next = s->top;
        s->top = new_node;
        s->nelems++;
    }


#### int_stack_pop


正常的实现

    int int_stack_pop(int_stack *s) {
        if (s->nelems == 0) {
            error(1, 0, "Cannot pop from empty stack"); 
        }
        int_node *n = s->top;
        int value = n->data;
    
        s->top = n->next; 
    
        free(n);
        s->nelems--;
    
        return value;
    }



怎么修改？

我们不能直接返回数据  ，因为数据类型不确定

如果传递一个指针回去的话，那么，谁来负责释放它呢？？
使用者才不管你那么多，

所以我们要调用者再给我们一个地址，我们传递到那个地址上，这样就没我们的事了，就算需要释放也是使用者的问题了

实现：

    void stack_pop(stack *s, void *addr) {
        if (s->nelems == 0) {
            error(1, 0, "Cannot pop from empty stack"); 
        }
        node *n = s->top;
        memcpy(addr, n->data, s->elem_size_bytes);
        s->top = n->next; 
    
        free(n->data);
        free(n);
        s->nelems--;
    }

    

OK  ，我们来使用一下，泛型堆栈

正常的堆栈使用

    int_stack *intstack = int_stack_create();
    for (int i = 0; i < TEST_STACK_SIZE; i++) {
        int_stack_push(intstack, i); 
    }

由于泛型堆栈不能直接传递数据本身

所以 ，传递地址

    stack *intstack = stack_create(sizeof(int));
    for (int i = 0; i < TEST_STACK_SIZE; i++) {
        stack_push(intstack, &i); 
    }




对于正常的使用

    int_stack *intstack = int_stack_create();
    int_stack_push(intstack, 7);


我们都要转换为

    stack *intstack = stack_create(sizeof(int));
    int num = 7;
    stack_push(intstack, &num);


还有  ，在输出元素时，  正常为

    // Pop off all elements
    while (intstack->nelems > 0) {
        printf("%d\n", int_stack_pop(intstack));
    }

我们要做修改，因为，我们要传递指针接受数据，进而输出

    // Pop off all elements
    int popped_int;
    while (intstack->nelems > 0) {
        int_stack_pop(intstack, &popped_int);
        printf("%d\n", popped_int);
    }

### 要点
C语言有一种指针类型，void*

对于void* 的指针，我们不能对其进行取消引用操作和指针运算操作

我们可以使用mencpy， memmove ，直接从内存位置进行拷贝

对于void* 的指针运算，我们一般先将其转换为char*，不过由于类型多变的原因，使用时要多多注意。

### OK  本节结束


## Lecture 9 C 泛型--函数指针


###  目前为止的泛型

关于void* 就不多介绍了，以及泛型交换，泛型数组交换，泛型注意事项等的前面都有。 我只写前面没有提到的内容


#### memset
函数声明

    void *memset(void *s, int c, size_t n);

它的作用是将一个指定地址的指定字节数设置为指定值。

带入里面就是说，用c 去填充从s地址开始的m个字节


    int counts[5];
    memset(counts, 0, 3);	// zero out first 3 bytes at counts
    memset(counts + 3, 0xff, 4)	// set 3rd entry’s bytes to 1s

#### 为什么void * 很强大

1. 对于需要接受多种数据类型的函数参数，它需要能够接收不同大小的数据类型。但是指针的大小都是一样的（八个字节），不管他指向的数据类型是什么，都可以通过指针传递。
2. 这样就需要我们在定义形参数，设置为指针，但是，不同数据类型的指针定义又不一样，又该怎么办呢。
3. void* 就解决了这个问题，它可以指向任何类型，只要在接受到后，强制转换类型即可。

这就显示出了void*的 强大功能。

### 泛型冒泡排序
现在我们来写一个函数对整型进行排序，我们使用冒泡排序。

原理不对说。


    void bubble_sort_int(int *arr, int n) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                if (arr[i - 1] > arr[i]) {
                    swapped = true;
                    swap_int(&arr[i - 1], &arr[i]);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }

现在，能不能把它变成能够给任意数据类型排序的函数呢。

第一步，修改形参。


    void bubble_sort(void *arr, int n, int elem_size_bytes) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                if (arr[i - 1] > arr[i]) {
                    swapped = true;
                    swap(&arr[i - 1], &arr[i], elem_size_bytes);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }
    
前面有关于数组首位的泛型交换函数，现在把它推广到任意第i个位置的交换。



    void swap_ends(void *arr, size_t nelems, size_t elem_bytes) {
        swap(arr, (char *)arr + (nelems – 1) * elem_bytes, elem_bytes);
    }


OK ，修改为

    void *ith_elem = (char *)arr + i * elem_bytes;

这样就可以实现第i个的交换。



修正版：

    void bubble_sort(void *arr, int n, int elem_size_bytes) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                void *p_prev_elem = (char *)arr + (i - 1) * elem_size_bytes;
                void *p_curr_elem = (char *)arr + i * elem_size_bytes;
                if (*p_prev_elem > *p_curr_elem) {
                    swapped = true;
                    swap(p_prev_elem, p_curr_elem, elem_size_bytes);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }


发现问题没有？？？？
1. 前面说过，不能对void* 进行取消引用。
2. void*的指向 不可能一直是int等的这种可以直接用>进行比较的类型。


所以现在遇到一个问题，我们无法对一个不确定的数据类型进行比较，无法比较，我们的代码就不能正常使用。

怎么办？？？

虽然泛型函数不知道类型，但是身为调用者肯定是知道的对吧。

所以，我们要去问调用者，


### 函数指针

好的，我们说了，要询问调用者怎么进行数据比较。

那么具体怎么弄？？

因为调用者知道数据类型，以及比较方案，那么，我们要调用者给我们一个比较函数。

    compare_fn(p_prev_elem, p_curr_elem)


现在有了比较方案，排序函数，怎么去调用这个函数呢？？？

我们可以搞一个函数指针，调用者吧这个函数作为参数传递给我们。

    void bubble_sort(void *arr, int n, int elem_size_bytes, 
                     bool (*compare_fn)(void *a, void *b)) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                void *p_prev_elem = (char *)arr + (i - 1) * elem_size_bytes;
                void *p_curr_elem = (char *)arr + i * elem_size_bytes;
                if (compare_fn(p_prev_elem, p_curr_elem)) {
                    swapped = true;
                    swap(p_prev_elem, p_curr_elem, elem_size_bytes);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }


下面，怎样声明参数的类型。


通用语法如下

    [return type] (*[name])([parameters])


OK  现在来个具体的。

    bool (*compare_fn)(void *a, void *b)

上面这个声明一个返回值为bool，名字叫compare_fn，参数为两个void* 的一个函数的函数指针


OK 现在，我们的泛型排序函数就完成了。

    void bubble_sort(void *arr, int n, int elem_size_bytes, 
                     bool (*compare_fn)(void *a, void *b)) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                void *p_prev_elem = (char *)arr + (i - 1) * elem_size_bytes;
                void *p_curr_elem = (char *)arr + i * elem_size_bytes;
                if (compare_fn(p_prev_elem, p_curr_elem)) {
                    swapped = true;
                    swap(p_prev_elem, p_curr_elem, elem_size_bytes);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }


只不过没我们需要调用者给我们一个比较的方法。


看几个应用实例

    bool integer_compare(void *ptr1, void *ptr2) {
        ...   
    }
    
    int main(int argc, char *argv[]) {
        int nums[] = {4, 2, -5, 1, 12, 56};
        int nums_count = sizeof(nums) / sizeof(nums[0]);
        bubble_sort(nums, nums_count, sizeof(nums[0]), integer_compare);
        ...
    }



    bool string_compare(void *ptr1, void *ptr2) {
        ...   
    }
    
    int main(int argc, char *argv[]) {
        char *classes[] = {"CS106A", "CS106B", "CS107", "CS110"};
        int arr_count = sizeof(classes) / sizeof(classes[0]);
        bubble_sort(classes, arr_count, sizeof(classes[0]), string_compare);
        ...
    }


在回顾一下，泛型排序的函数声明

    void bubble_sort(void *arr, int n, int elem_size_bytes, 
                     bool (*compare_fn)(void *a, void *b))


还有一点就是： 由于我们泛型的数据大小是不确定的，我们要传给比较函数的数据也是不定大小的，

那么，怎么让比较函数可以接收任意大小的数据呢，——-用指针，

因为指针有分类型，多以，用的是void *    

而且，由于不同的比较函数，比较的类型不同，同时，我们也不能对void*进行取消引用。

所以，比较函数在接收到void* 后，需对其进行强制类型转换，转换为要比较的类型。

同时，我们的泛型排序函数所接受的比较函数也必须是泛型的，因为要处理任意类型的数据。

例如： 

    bool integer_compare(void *ptr1, void *ptr2) {
    	// 1) cast arguments to int *s
         int *num1ptr = (int *)ptr1;
         int *num2ptr = (int *)ptr2;
    
         // 2) dereference typed points to access values
         int num1 = *num1ptr;
         int num2 = *num2ptr;
    
         // 3) perform operation
         return num1 > num2;
    }


也可以这样写：

    bool integer_compare(void *ptr1, void *ptr2) {
        return *(int *)ptr1 > *(int *)ptr2;
    }


许多的C语言的标准库都提供了标准函数，它们一般返回

如果第一个值大于第二个值，返回>0

如果第一个值小于第二个值，返回<0

如果相等，返回0

这与strcmp的返回值格式相同！


例如


    int integer_compare(void *ptr1, void *ptr2) {
        return *(int *)ptr1 – *(int *)ptr2;
    }


这样就可以将泛型排序改写成

    void bubble_sort(void *arr, int n, int elem_size_bytes, 
                     int (*compare_fn)(void *a, void *b)) {
        while (true) {
            bool swapped = false;
            for (int i = 1; i < n; i++) {
                void *p_prev_elem = (char *)arr + (i - 1) * elem_size_bytes;
                void *p_curr_elem = (char *)arr + i * elem_size_bytes;
                if (compare_fn(p_prev_elem, p_curr_elem) > 0) {
                    swapped = true;
                    swap(p_prev_elem, p_curr_elem, elem_size_bytes);
                }
            }
            
            if (!swapped) {
                return;
            }
        }
    }


现在，我们来写一个，字符串排序时的比较函数，按照字母表的顺序

    
    int string_compare(void *ptr1, void *ptr2) {
    	// cast arguments and dereference
         char *str1 = *(char **)ptr1;
         char *str2 = *(char **)ptr2;
    
         // perform operation
         return strcmp(str1, str2);
    }

#### 函数指针陷阱

由于将函数作为参数传递个函数后，调用时需要传递符合要求的参数，但是如果传递错了怎么办？？！！！


### 泛型打印

泛型有很多用途

例如

比较给定类型的两个元素的函数

打印给定类型的元素的函数

释放与给定类型相关联的内存的函数


等等还有很多

比较


    int (*cmp_fn)(void *addr1, void *addr2)


输出

    void (*print_fn)(void *addr)

还有更多。。。。



还有一种函数指针的使用方法：

不太好形容，直接看例子好了


    int do_something(char *str) {
        …
    }
    
    int main(int argc, char *argv[]) {
        …
        int (*func_var)(char *) = do_something;
        …
        func_var("testing");
        return 0;   
    }
    


#### 泛型C语言标准库

qsort--
可以对任何类型的数组进行排序！为了做到这一点，需要你提供一个函数，可以比较要求排序的两个元素。


bsearch-–可以使用二进制搜索来搜索任何类型数组中的键！为此，需要提供一个函数，可以比较要求搜索的两个元素。


lfind-–我可以使用线性搜索来搜索任何类型数组中的键！为此，需要提供一个函数，可以比较要求搜索的两个元素。


lsearch--可以使用线性搜索来搜索任何类型数组中的键！如果找不到，也会加上。为了做到这一点，需要提供一个函数，可以比较要求搜索的两个元素。

更具体的英文介绍见课件，我翻译的不太标准

还有好多自己百度


#### 要点

可以将函数做为参数传递给函数，

比较函数比较常用，一般用于比较两个指针所指向的地址处的元素。

在应用泛型数，必须传递指向其所关心的数据类型的指针，因为任何比较函数只对应一个类型。




###  泛型是使用概述


1. 使用memcpy，memmove ，实现数据操作的泛型化。
2. 使用函数指针实现逻辑功能通用化。


###  OK 本节结束
