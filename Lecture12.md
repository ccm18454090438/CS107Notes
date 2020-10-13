[200~## Lecture 12  汇编：控制流

主要讲程序如何将比较和运算结果存储在条件判断代码中，  
以及如何实现判断和循环


### 汇编执行和%rip

我们知道程序的数据存储在内存或寄存器中，  
同时，汇编指令也在内存和寄存器之间来回读写，  
而且汇编指令也是存储在内存中的。 


但是，谁来控制程序对应指令的执行呢？？
程序如何知道下一步要干什么呢？？

此时我们需要 程序计数器：  %rip


前面有说过：

一些寄存器在程序执行期间承担特殊责任。

%rax存储返回值

%rdi将第一个参数存储到函数中

%rsi将第二个参数存储到函数中

%rdx将第三个参数存储到函数中

%rip存储下一条要执行的指令的地址

%rsp存储当前栈顶的地址


#### 循环。

如何用这种%rip存储下一个指令的方式来执行循环？？  
调整%rip！


#### jmp


看一个例子

    00000000004004ed <loop>:
    4004ed: 55                    push   %rbp
    4004ee: 48 89 e5              mov    %rsp,%rbp
    4004f1: c7 45 fc 00 00 00 00  movl   $0x0,-0x4(%rbp)
    4004f8: 83 45 fc 01           addl   $0x1,-0x4(%rbp)
    4004fc: eb fa                 jmp    4004f8 <loop+0xb>


这是一个死循环。  

类似：

    while (true) {…}

jmp是作用是无条件跳转，无须判断直接更改程序计数器（%rip）


使用方式：

jmp 标签（直接跳转）

jmp *操作数（间接跳转）


目的地可以硬编码到指令中（直接跳转）：

    jmp 404f8<loop+0xb>
跳转到0x404f8处的指令

目标也可以是常用的操作数形式之一（间接跳转）：

jmp *%rax  #跳转到地址为%rax的指令


    jmp [target]

程序碰到上述指令就直接跳转 

### 控制流

怎么进行条件跳转呢？？ 

#### 条件代码
怎么实现？？
1. 计算结果，  
2. 根据结果选择a方向，  


指令实现包括
一条计算结果，一条条件跳转

例如

    cmp S1, S2      // compare two values
    je [target]   or   jne [target]   or   jl [target]   or  ...        // 
解释一下：  

je ： 表示相等，  
jne： 表示不相等，  
jl： 表示小于


更多变形请看：



![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5jxzgGrZVXXTFJMpTWR2qScmSR7NXdWZM0H3lZjLWrI8k*Q9eazhoktxfoKwSQdna2SHKxP3q.uAlvbuXLJ1qvI!/b&bo=jAgQBAAAAAABB7A!&rf=viewer_4)



    cmp S1，S2

实际上是拿S2 与S1进行比较


示例代码

    // Jump if %edi > 2
    cmp $2, %edi
    jg [target]
    
    // Jump if %edi != 3
    cmp $3, %edi
    jne [target]
    
    
    // Jump if %edi == 4
    cmp $4, %edi
    je [target]
    
    // Jump if %edi <= 1
    cmp $1, %edi
    jle [target]


程序是如何知道前面比较的结果的呢？？？  


CPU里面有一个特别的寄存器， 叫做条件码。

cmp 通过进行减法运算 然后查看结果并将结果存储在条件代码中。 

OK  ，什么是条件代码？？   

最常见的有:


CF：进位标志。最近的操作对最高有效位的产生了进位。用于检测无符号操作的溢出。

零标志：ZF。最近的零操作产生了最新的零。

SF：标志旗。最近的操作产生了一个负值。

OF：溢出标志。最近的一次操作导致了一个2的补码溢出，要么是负的，要么是正的。

##### 设置条件代码

例如：

如果我们计算t=a+b，则条件代码的设置依据：

CF：进位标志（无符号溢出）。（无符号）t<（无符号）a

ZF：零标志（零）。（t==0）

SF：符号标志（负）。（t<0）

OF：溢出标志（有符号溢出）。（a<0==b<0）&&（t<0！=a<0）


cmp指令与减法类似，不过它不存储结果，值设置条件代码，  
注意：

    CMP S1, S2		 S2 – S1


同时cmp也是可以使用后缀的

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5rVeHN1Pzrc3zreOaCsmiJUKQn.oSc*Z4U6*QgsI3ZGqdQYFk9hBT2lLCDDzrs*VbEtvNAZi9BkSqoCrdZiJ1ew!/b&bo=VgfYAQAAAAABB6g!&rf=viewer_4)




示例：
    
    // Jump if %edi > 2
    // calculates %edi – 2
    cmp $2, %edi
    jg [target]

    
    // Jump if %edi != 3
    // calculates %edi – 3
    cmp $3, %edi
    jne [target]

    
    // Jump if %edi == 4
    // calculates %edi – 4
    cmp $4, %edi
    je [target]

    
    // Jump if %edi <= 1
    // calculates %edi – 1 cmp $1, %edi
    jle [target]


    
再看一下结合了条件代码的各种条件跳转：


![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcb2qaj1AMmiS0cjnW8zMJtzA0lbOuWQSqwCk8AxddX4ilCp3hK875Yv3CVTlWv40gqkZkcCWj5CCTYRJ0Y2usMs!/b&bo=ggo4BAAAAAABF4Q!&rf=viewer_4)





还有一个设置条件代码的指令是： TEST


TEST  用于and （&）  也是不存储结果，只设置条件代码。  

    TEST S1, S2		 S2 & S1


![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcb2qaj1AMmiS0cjnW8zMJtwZ2US72*MiQvWluHaT5jx*P0lc2ZTW045CAsk3ttzXcMUEX2oBKkBG9zrn6SnWouo!/b&bo=bAVcAQAAAAABFwQ!&rf=viewer_4)


在回到前面的指令。



了解程序集如何将比较和运算结果存储在条件代码中

了解程序集如何实现循环和控制流



程序集执行和%rip

控制流力学

条件代码

装配说明

If语句

循环

While循环

For循环

取决于条件代码的其他指令


登记职责


一些寄存器在程序执行期间承担特殊责任。

%rax存储返回值

%rdi将第一个参数存储到函数中

%rsi将第二个参数存储到函数中

%rdx将第三个参数存储到函数中

%rip存储下一条要执行的指令的地址

%rsp存储当前栈顶的地址


更多信息，请参阅参考资料网页上的x86-64指南和参考资料表！



指令只是字节！



程序计数器（PC）在x86-64中称为%rip，它将地址存储在下一条要执行的指令的内存中。


除了普通寄存器外，CPU还具有单位条件代码寄存器。它们存储最新的算术或逻辑运算的结果。


最常见的情况代码：

CF：进位标志。最近的操作产生了对最有效位的执行。用于检测无符号操作的溢出。

零标志：ZF。最近的零操作产生了最新的零操作。

SF：标志旗。最近的操作产生了一个负值。

OF：溢出标志。最近的一次操作导致了一个2的补码溢出，要么是负的，要么是正的。



除了普通寄存器外，CPU还具有单位条件代码寄存器。它们存储最新的算术或逻辑运算的结果。



例：如果我们计算t=a+b，则条件代码的设置依据：

CF：进位标志（无符号溢出）。（无符号）t<（无符号）a

ZF：零标志（零）。（t==0）

SF：符号标志（负）。（t<0）

OF：溢出标志（有符号溢出）。（a<0==b<0）&（t<0！=a<0）






cmp指令与减法指令类似，但它不将结果存储在任何地方。它只是设置条件代码。（注意操作数顺序！）


凸轮轴位置S1，S2 S2–S1



将cmp S1、S2读作“比较S2与S1”。它计算S2–S1并用结果更新条件代码。



条件跳转可以查看条件代码的子集，以便检查它们感兴趣的条件。




测试指令类似于cmp，但用于和。它不在任何地方存储结果（&R）。它只是设置条件代码。


试验S1、S2 S2和S1


酷把戏：如果我们为两个操作数传递相同的值，我们可以使用符号标志和零标志条件代码检查该值的符号！


前面讨论的算术和逻辑指令也更新条件代码。不过lea不更新（它只用于地址计算）。

逻辑运算（xor等）将进位和溢出标志设为零。

移位操作将进位标志设置为移出的最后一位，并将溢出标志设置为零。

由于特殊原因原因，inc和dec设置了溢出和零标志，但是保持进位标志不变。 



### if语句



练习： 填空


    00000000004004d6 <if_then>:
      4004d6:	  cmp  $0x6,%edi
      4004d9:	  jne  4004de
      4004db:	  add  $0x1,%edi
      4004de:	  lea  (%rdi,%rdi,1),%eax
      4004e1:	  retq 


    int if_then(int param1) {
      if ( __________ ) {
          _________;
      }
      
      return __________;
    } 

补充完整。


答案：
1. param1 == 6
2. param1++
3. param1 * 2



带有else的：

    Test
    Jump to else-body if test fails
    If-body
    Jump to past else-body
    Else-body
    Past else body
    
    
    
    
    if (arg > 3) {
        ret = 10;
    } else {
        ret = 0;
    }
    
    ret++;

对应汇编：
    
    400552 <+0>:  cmp  $0x3,%edi
    400555 <+3>:  jle  0x40055e <if_else+12>
    400557 <+5>:  mov  $0xa,%eax
    40055c <+10>: jmp  0x400563 <if_else+17>
    40055e <+12>: mov  $0x0,%eax
    400563 <+17>: add  $0x1,%eax



OK 这个就不多说了，就是简单的多个地方进行跳转就是了



### 循环


#### while循环


示例：

    
    void loop() {
        int i = 0;
        while (i < 100) {
            i++;
        }
    }


汇编： 

    0x0000000000400570 <+0>:	mov    $0x0,%eax
    0x0000000000400575 <+5>:	jmp    0x40057a <loop+10>
    0x0000000000400577 <+7>:	add    $0x1,%eax
    0x000000000040057a <+10>:	cmp    $0x63,%eax
    0x000000000040057d <+13>:	jle    0x400577 <loop+7>
    0x000000000040057f <+15>:	repz retq 

#### for 循环



说明：


    for (init; test; update) {
    	body
    }
     

伪代码：

    Init
    Jump to test
    Body
    Update
    Test
    Jump to body if success


类似于while循环：

    init
    while(test) {
        body
        update
    }




示例：


    int sum_array(int arr[], int nelems) {
       int sum = 0;
       for (int i = 0; i < nelems; i++) {
          sum += arr[i];
       }
       return sum;
    }
    
    00000000004005b6 <sum_array>:
      4005b6:        mov    $0x0,%edx
      4005bb<+5>:    mov    $0x0,%eax
      4005c0<+10>:   jmp    4005cb <sum_array+21>
      4005c2<+12>:   movslq %edx,%rcx
      4005c5<+15>:   add    (%rdi,%rcx,4),%eax
      4005c8<+18>:   add    $0x1,%edx
      4005cb<+21>:   cmp    %esi,%edx
      4005cd<+23>:   jl     4005c2 <sum_array+12>
      4005cf<+25>:   repz retq 


课程还提到了一个交gdb的工具，具体的自己去看


### 依靠条件代码的其他指令


有三种使用条件代码的常见指令类型：

jmp指令有条件地跳转到另一条指令

set指令有条件地将字节设置为0或1

新版本的mov指令有条件地移动数据



jmp 前面说过了，


set是有条件分设置指定字节为0或1 ，  
目标为单字节寄存器， 

其他字节不受影响

一般后面有movzbl 进行拓展，将多余字节归零。  


例如： C代码为：  

    
    int small(int x) {
        return x < 16;
    }


使用set做到的汇编为：  

    cmp $0xf,%edi
    setle %al
    movzbl %al, %eax
    retq



具体如图


![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5ocGOqhz6vfu7AZlJm2kM0rvANmlVlXsiyfiVwlyBbeE4VLWKQG0MnB3bnTxYy1EV9uZYiOEs9bxjIcTVRnqlus!/b&bo=1AdyAwAAAAABB4I!&rf=viewer_4)

#### cmov 条件代码


    cmovx src,dst

cmov 有条件的将src中的数据移动到dst中，前者为内存地址或寄存器，后者是寄存器。   

一般用于三目运算符： result = test ? then: else;


示例： 
    
    int max(int x, int y) {
        return x > y ? x : y;
    }



用cmov 的实现是： 

    cmp    %edi,%esi
    mov    %edi, %eax
    cmovge %esi, %eax
    retq
    

更所用法如图：

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcZnFyoCkSSvobgKe*Msf8j3Zl1ZCHwe1PObGj2FLiQMi7tsV4lkkHs9Cjlq0jP1nLeA.GAL48pZyW7ydVFhAIWg!/b&bo=1AdcAwAAAAABF7w!&rf=viewer_4)




### OK  本节结束，  下节 ，汇编的函数调用


