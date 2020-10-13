##  Lecture 10 汇编介绍

### 接下来的这几节主要学习汇编

1.   Moving data around ： The Lecture  (这节课--移动数据  )
2. Arithmetic and logical operations  ： Lecture 11（算术和逻辑运算）
3. Control flow ： Lecture 12  （控制流）
4. Function calls ： Lecture 13  （函数调用）

### GCC 和 汇编

#### 数据表示

1. 整数（无符号整数，有符号整数（补码））
2. 字符（ASC码）
3. 地址（无符号长整型）
4. 集合（数组，结构体。）


#### GCC

GCC 可以将您的可读代码转换为机器可读指令。

不管是C语言还是其他语言，计算机都是看不懂的，纯正的机器代码全是0和1，但是我们可以以另一种方式去阅读它。称之为汇编。

下面看看汇编是什么样的。


### 汇编示例

    int sum_array(int arr[], int nelems) {
       int sum = 0;
       for (int i = 0; i < nelems; i++) {
          sum += arr[i];
       }
       return sum;
    }


转换为汇编 

    00000000004005b6 <sum_array>:
      4005b6:    ba 00 00 00 00       mov    $0x0,%edx
      4005bb:    b8 00 00 00 00       mov    $0x0,%eax
      4005c0:    eb 09                jmp    4005cb <sum_array+0x15>
      4005c2:    48 63 ca             movslq %edx,%rcx
      4005c5:    03 04 8f             add    (%rdi,%rcx,4),%eax
      4005c8:    83 c2 01             add    $0x1,%edx
      4005cb:    39 f2                cmp    %esi,%edx
      4005cd:    7c f3                jl     4005c2 <sum_array+0xc>
      4005cf:    f3 c3                repz retq 
    

这是函数的名称，以及该函数代码开始的内存地址。

    00000000004005b6 <sum_array>: 


这是每条指令的内存地址，顺序指令在内存中也是顺序的。


     4005b6:  
     4005bb:  
     4005c0:  
     4005c2:    
     4005c5:    
     4005c8:  
     4005cb:  
     4005cd:   
     4005cf: 


这是汇编代码，每个指令都是可读版本。

    mov    $0x0,%edx
    mov    $0x0,%eax
    jmp    4005cb <sum_array+0x15>
    movslq %edx,%rcx
    add    (%rdi,%rcx,4),%eax
    add    $0x1,%edx
    cmp    %esi,%edx
    jl     4005c2 <sum_array+0xc>
    repz retq 

这是机器代码：原始十六进制指令，表示计算机读取的二进制。不同的指令可能具有不同的字节长度。


    ba 00 00 00 00
    b8 00 00 00 00 
    eb 09
    48 63 ca 
    03 04 8f
    83 c2 01
    39 f2
    7c f3
    f3 c3



每一条指令都有一个操作名称

每个指令还有参数


$[number]  表示一个常熟，或称作立即数。

％[name]表示寄存器，即CPU上的存储位置（例如edx） %edx

### 寄存器和汇编的抽象等级


#### 汇编抽象

C抽象了机器代码的底层细节，所以我们能够使用变量，和其他更高级的抽象。

不过，汇编代码只有字节，没有变量等等的一些概念，汇编作用于特定的处理器，
他的抽象级别是寄存器。
或者说，它的操作单位是寄存器。


#### 寄存器

什么是寄存器，

寄存器是CPU上的可以快速读写的小型存储区域。


1.   寄存器有64位，也就是8字节。
2.   一共有16个寄存器。每一个都有唯一的名称
3.   寄存器就像处理器的“草稿纸”。正在计算或处理的数据首先移至寄存器。操作在寄存器上执行。
4.   寄存器还保存参数和函数的返回值。
5.   寄存器非常快！
6.   处理器指令主要包括将数据移入/移出寄存器并对其进行算术运算。这是程序必须执行的逻辑级别！


汇编指令就是操作这些寄存器，
例如，在寄存器中加两个数字，将数据从寄存器传输到内存，将数据从内存移到寄存器。等等。



关于计算机的架构，课件上有更形象的配图，


#### GCC和汇编


GCC将编译你的程序，在堆栈和堆上分配内存，并生成汇编指令以访问这些内存位置并进行计算。

例如   ：

C语言   

    int sum = x + y;


汇编级别将抽象为

    Copy x into register 1
    Copy y into register 2
    Add register 2 to register 1
    Write register 1 to memory for sum


这个就不翻译了。



#### 下面将要学习的指令为 x86-64指令集体系结构


该指令集由Intel和AMD处理器使用。

还有许多其他指令集：ARM，MIPS等。


### mov 指令

mov 指令将字节从一个位置复制到另一个位置，类似‘=’。


例如   

    mov		src,dst




其中  ：

1.  只有src可以是立即数，也就是说常数。
2.  二者皆可以是寄存器
3.  二者最多可以有一个是内存位置。


下面是示例。

    mov		$0x104,_____
    
将值0x104复制到某个位置。

    mov		%rbx,____
    
将寄存器%rbx中的值复制到某个位置。

    mov    ____,%rbx

将值从某个来源复制到寄存器%rbx。


    mov		0x104,_____

将地址0x104处的值复制到某个目的地


    mov    _____,0x104

将值从某个来源复制到地址0x104的内存中



课件上的一个练习

以下移动指令（单独执行）的结果是什么？对于这个问题，假设值5存储在地址0x42，而值8存储在%rbx中。

    mov   $0x42,%rax
    
    mov    0x42,%rax
    
    mov    %rbx,0x55



带括号的用法

    mov		(%rbx),_____


将存储在%rbx存储的地址处的值拷贝到某个目标，


    mov     _____,(%rbx)

从一个源头往%rbx存储的内存地址处拷贝数据。



    mov		0x10(%rax),_________

拷贝%rax存储的地址加上0x10对应的地址处的值拷贝到某个目标处。

    mov    __________,0x10(%rax)

从某个源头向%rax存储的值加上0x10对应是地址处拷贝数据。



    mov		(%rax,%rdx),__________

将 %rax和%rdx处存储的值的和所对应的地址处 的值拷贝到目标处。



    mov    ___________,(%rax,%rdx)

从某个源头向%rax和%rdx处存储的值的和所对应的地址拷贝数据。


    mov    0x10(%rax,%rdx),______

将0x10加上寄存器%rax和%rdx中的值之和对应的地址处的值复制到某个目的地。


    mov    _______,0x10(%rax,%rdx)

从某个源向地址址是（0x10加上寄存器%rax和%rdx中的值之和）的位置拷贝数据。



#### 练习

以下移动指令（单独执行）的结果是什么？对于这个问题，假设值0x11存储在地址0x10C，0xAB存储在地址0x104，0x100存储在寄存器%rax中，0x3存储在%rdx中。

    mov   $0x42,(%rax)
    mov   4(%rax),%rcx
    mov   9(%rax,%rdx),%rcx



    Imm(rb, ri) is equivalent to address Imm + R[rb] + R[ri]

Imm 正或负或缺失

上面是公式。  


rb  如果缺失就当0算。ri也是。




另一个公式。

这个的示例就不给了，自己理解。

Imm(rb,ri,s) is equivalent to…  Imm + R[rb] + R[ri]*s


其中  ，Imm为正或负，缺失为0

rb 缺失当0，ri也是。

s必须为1、2、4或8（如果缺少，则=1）




![image](https://pic2.zhimg.com/80/v2-4c361cecb83705bc2bf90099b1ed81bd_1440w.jpg)



OK  上面是更具体的介绍。




#### 练习

以下移动指令（单独执行）的结果是什么？对于这个问题，假设值0x1存储在寄存器%rcx中，值0x100存储在寄存器%rax中，值0x3存储在寄存器%rdx中，值0x11存储在地址0x10C。

    mov   $0x42,0xfc(,%rcx,4)
    
    mov   (%rax,%rdx,4),%rbx




提示：  
公式： 

    Imm(rb, ri, s) is equivalent to address Imm + R[rb] + R[ri]*s





OK  现在回顾一下


    int sum_array(int arr[], int nelems) {
       int sum = 0;
       for (int i = 0; i < nelems; i++) {
          sum += arr[i];
       }
       return sum;
    }
    
    00000000004005b6 <sum_array>:
      4005b6:    ba 00 00 00 00       mov    $0x0,%edx
      4005bb:    b8 00 00 00 00       mov    $0x0,%eax
      4005c0:    eb 09                jmp    4005cb <sum_array+0x15>
      4005c2:    48 63 ca             movslq %edx,%rcx
      4005c5:    03 04 8f             add    (%rdi,%rcx,4),%eax
      4005c8:    83 c2 01             add    $0x1,%edx
      4005cb:    39 f2                cmp    %esi,%edx
      4005cd:    7c f3                jl     4005c2 <sum_array+0xc>
      4005cf:    f3 c3                repz retq 


现在看看能看懂什么？？？

当然以后还会讲这个例子、

OK  本节结束，下一节，深入了解汇编。。

