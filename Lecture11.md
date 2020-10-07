## Lecture 11 汇编：运算和逻辑运算



很显然，本节主要说 在汇编中执行算术运和逻辑运算。

课件里面提供了一些资源：


我就不做翻译了，

Course textbook (reminder: see relevant readings for each lecture on the Schedule page, http://cs107.stanford.edu/schedule.html)


CS107 Assembly Reference Sheet: http://cs107.stanford.edu/resources/x86-64-reference.pdf 

CS107 Guide to x86-64: http://cs107.stanford.edu/guide/x86-64.html 


原模原样奉上、


### 截止到目前的mov指令
前面的只是初步介绍这里详细的介绍。


这里对内存位置在复习一下。


后面的表格都是直接上图，简单的英文还是要会的，太复杂的我会翻译
![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5hkt1K4M*epWXdfY1ls2HkjQrWm2i5DVLp8TYWTIoJ4odqmciTRruPu7tPq6IVL5lG*0M4ceTT0NV2rb8kEBe1E!/b&bo=AAWKAgAAAAABB60!&rf=viewer_4)


还有对所有格式的介绍

![image](https://pic2.zhimg.com/80/v2-4c361cecb83705bc2bf90099b1ed81bd_1440w.jpg)


### 数据和寄存器大小

在汇编中数据大小的表示略有不同。

原文是

    A byte is 1 byte.
    A word is 2 bytes.
    A double word is 4 bytes.
    A quad word is 8 bytes.

翻译过来是


一个字节是1个字节。

一个字是2个字节。

双字是4字节。

四字是8字节。


汇编可以使用后缀表示大小

后缀有：
1. b表示一个字节
2. w表示一个字
3. l表示一个双字
4. q表示一个四字

更详细内容见下图




![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5jmUMcqwv0oydXfrEfMM5qG..0IMrJFBpS5W2W8Su1W*BEOgkHBvuVQRT1UPzBRYXlpS6Tap*B3rUxMhioXV6Ss!/b&bo=JAiWAyQIlgMBByA!&rf=viewer_4)



![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5tknl1jRHU0Izv6f3pJW9gfAjIQ9k44zFjDLvh2SN9oWZNYFOIRu5TKfiefgDnkYk1TabLqryCjYas6kXC65bf0!/b&bo=JAjiAwAAAAABB.0!&rf=viewer_4)


![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcRhjwNUhy49dScuqAyELlNotPYaARf0wk9cxYqLJidVqYwK.TVm9R6rFm3fjt7HgRVMg4d.2k3FXk7NByIP*PHg!/b&bo=bghaAwAAAAABFw8!&rf=viewer_4)


#### 有一些寄存器在程序执行过程中有特殊任务

1. %rax存储返回值
2. %rdi将第一个参数存储到函数中
3. %rsi将第二个参数存储到函数中
4. %rdx将第三个参数存储到函数中
5. %rip存储下一条要执行的指令的地址
6. %rsp存储当前栈顶的地址


#### mov的变形

mov可以添加后缀来表示所要移动的数据的大小。

例如： movb，movw,movl,movq.

这样指令就只会更新指定的寄存器字节或内存位置。

例如： movl 在将数据写入寄存器的同时也会将高四位的字节归零。

#### 练习，给下面的mov添加合适的后缀

    mov__ %eax, (%rsp)
    mov__ (%rax), %dx
    mov__ $0xff, %bl
    mov__ (%rsp,%rdx,4),%dl
    mov__ (%rdx), %rax
    mov__ %dx, (%rax)


答案：

    movl %eax, (%rsp)
    movw (%rax), %dx
    movb $0xff, %bl
    movb (%rsp,%rdx,4),%dl
    movq (%rdx), %rax
    movw %dx, (%rax)

#### movabsq

movabsq指令常用于写入64位的立即数。

例如

    movabsq $0x0011223344556677, %rax

因为，正常情况下，movq只接受32位立即数。  
64位的话会被当做一个目的地而非立即数。

#### movz和movs

这两个指令用于将较小的数据赋给较大的目的地。

movz用零填充剩余的字节

movs通过符号扩展源中最高有效位来填充剩余的字节。


同时，使用时，来源必须是寄存器或内存地址，目的地必须是寄存器。


下面是使用方法总结

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcWa3jN3rYwHZ*FyyDptWUdWiCDS*fAM1vRSFdetN3HCwmHFcaIVZMZkNo2xm.IwVpou34CRFfzBctTiarOU6B6o!/b&bo=Rgi2AQAAAAABF8k!&rf=viewer_4)



![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcWa3jN3rYwHZ*FyyDptWUdX5TrwbUXwLElfy1hq*t18ly2KcRzX2ZXLoxAAWjqx1ooiQ2N6wS4G3pemBkwE1Fg8!/b&bo=JghQAgAAAAABF0w!&rf=viewer_4)



#### lea指令


    lea		src,dst


lea指令的操作方式与mov类似，区别在于如歌处理src。  

mov是操作src地址处的数据，而lea是直接操作src。

同时，他们的目标也就是dst操作是一样的。


操作具体解释

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/ruAMsa53pVQWN7FLK88i5mXn6KHlL2IcWMazlol0NeF6ygQrn37ROYBQk5cf6N.xO9i8UE2H8qb6OZscd1qJl.AGbMo2yOxi4uAfaq74KLY!/b&bo=RAiyAgAAAAABB9w!&rf=viewer_4)


### 逻辑运算和算术运算


#### 一元指令

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcecd6Fzjfb6jlFL8Zr7OG8KOARCh3Y6Dh2RJT0bM*zOvoKndaS1uwQ9noJKj1DmpREnSwlsfxmUUEWHp.xalVpI!/b&bo=qAVqAQAAAAABF*Y!&rf=viewer_4)


效果就不多说了吧。 

示例：

    incq 16(%rax)
    dec %rdx
    not %rcx



#### 二元指令


下面的指令要有两个操作数


需要注意的是，两个操作数可以是寄存器或内存，源数也可以是立即数，但是不能都是内存地址。


OK：  
看图



![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mccZx6yLskHmvS7ep8ZLIn9wFzT3mPaxSaaRZedcds9h.LlFvjXaZAX.Vm1c*H.iUfMJJ4sw3uHSBUt.eK2gPgDk!/b&bo=rAXYAQAAAAABF0A!&rf=viewer_4)



示例：  

    addq %rcx,(%rax)
    xorq $16,(%rax, %rdx, 8)
    subq %rdx,8(%rax)



#### 大数乘法
如果64位的数相乘会得到128位的结果，但是在x86里面只有64位寄存器。    
那怎么办呢？？

    imul 

如果为imul指定两个操作数，它会将乘得的结果截断，直到找到结果合适64
为寄存器。

    imul S, D		D ← D * S


如果指定一个操作数， 将会用该数乘以%rax。 
然后将结果拆分到两个寄存器内，  
规则是：
1. 高64位放入%rdx。  
2. 低64位放入%rax。  


![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcdHdLMNrx90KXb9VonixtxkmzAgHF7pPOH8SjlA.3W3Ljouqjchm5y4bGMvQNTqdZEnFWEEC3YZLgdER90aJGdQ!/b&bo=lgfsAAAAAAABF00!&rf=viewer_4)



#### 除法和余数

被除数/除数=商+余数


x86-64支持128位除以64位

格式要求如下
1. 被除数的高位64位存在%rdx中，
2. 被除数的低位64位存在%rax中。
3. 除数是指令的操作数。
4. 商存储在%rax中，余数存储在%rdx中。

具体见下图
![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mcdHdLMNrx90KXb9VonixtxlN.btezUohjAIQgMRRlyuJdQfIqwePuhxqOECBpYv79sFuN6gNnLv6EzC832YzhQc!/b&bo=jAc4AQAAAAABF4I!&rf=viewer_4)


大多数的除法只支持64位，所以有了cqto指令

cqto指令符号将%rax中的64位值扩展到%rdx，以便达到除法指令的要求，用被除数填充两个寄存器。

如图

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mccZx6yLskHmvS7ep8ZLIn9xII3hNDu**iaL*gLTYH2RG.voFGbQPvAxbZennYQej*A7SGRxWSTgTw1VgkTC5O*I!/b&bo=hgeYAQAAAAABFyg!&rf=viewer_4)




#### 位运算指令


具体的看图吧，很好理解

![image](http://m.qpic.cn/psc?/V534mZDk33nCkV2y6HUr32SC1A1b0hEN/45NBuzDIW489QBoVep5mccZx6yLskHmvS7ep8ZLIn9yY9uyBs9KRvCr7T9GGSpgURj2Y8AEOkEE7xKCGtlvnHXtDdAjKYIgfBDNSfZ6fqt0!/b&bo=lAVKAQAAAAABF.o!&rf=viewer_4)
使用示例

	shll $3,(%rax)
	shrl %cl,(%rax,%rdx,8)
	sarl $4,8(%rax)


### 汇编实例


https://godbolt.org/z/NLYhVf 
该网站能够快速查看代码对应的汇编


直接上实例
    
    // Returns the sum of x and the first element in arr
    int add_to_first(int x, int arr[]) {
        int sum = x;
        sum += arr[0];
        return sum;
    }
    
    
    ----------
    add_to_first:
      movl %edi, %eax
      addl (%rsi), %eax
      ret





再来一个

    // Returns x/y, stores remainder in location stored in remainder_ptr
    long full_divide(long x, long y, long *remainder_ptr) {
        long quotient = x / y;
        long remainder = x % y;
        *remainder_ptr = remainder;
        return quotient;
    }
    
    -------
    
    full_divide:
      movq %rdx, %rcx
      movq %rdi, %rax
      cqto
      idivq %rsi
      movq %rdx, (%rcx)
      ret
    

还有由汇编匹配源代码的练习，更多的在课件


### OK 本节结束


