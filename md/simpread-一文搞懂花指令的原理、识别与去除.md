> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Qv9Rm4oW5rWhVe1cHs6RIg)

花指令的原理、识别、去除
============

一、花指令的原理
--------

花指令的本质是干扰反汇编引擎正常工作的指令片段，不影响程序本身的执行结果。花指令可以干扰 IDA 等反汇编工具生成正确的汇编代码、CFG（控制流图）、进一步干扰生成中间代码（IR）及伪代码。 对于只会使用 IDA F5 功能进行逆向的人来说，简直就是致命杀手。按可执行性可分为**不可执行花指令**和可执行花指令。但即使是可执行花指令，也不会改变程序的原功能。在 x86 平台，常见的`junk_code`如下：

```
      指令                          操作码
call immed16     ---->    E8    // 3字节指令，immed16为2字节，代表跳转指令的目的地址(16位)与下一条指令地址的偏移
call immed32     ---->    9A    // 5字节指令，immed32为4字节，代表跳转指令的目的地址(32位)与下一条指令地址的偏移
jmp  immed8      ---->    EB 
jmp  immed16     ---->    E9 
jmp  immed32     ---->    EA 
loop immed8      ---->    E2 
ret              ---->    C2
retn             ---->    C3

```

为了能够有效的迷惑反汇编器，同时又保证代码的正确运行，花指令必须满足以下两个基本特征：

*   • 垃圾数据必须是某个合法指令的一部分。
    
*   • 程序运行时，垃圾数据必须位于实际不可执行的代码路径。
    

1 反汇编算法与设计缺陷
------------

反汇编算法分为**线性扫描算法**和**递归下降算法** (IDA)。

### 1.1 线性扫描算法

*   • 从入口开始，依次解析每一条指令，遇到分支指令不会递归进入分支。
    
*   • 线性扫描算法的缺点在于：在冯诺依曼体系结构下，**无法区分数据与代码，将代码段中嵌入的数据误解释为指令操作码，最后得到错误的反汇编**。
    
*   • 假如有一个函数 disAsm (addr) , 该函数可以对指定地址 addr 处反汇编一条指令，并将结果自动输出到屏幕，返回值是当前反汇编指令的长度。 你要如何驱动 disAsm 对一个完整的函数反汇编？
    

```
target = getFunctionAddress(mian)
targetEnd = getFunctionEnd(mian)
currentAddr = target
while currentAddr < targetEnd:
    currentLen = disAsm(addr)
    currentAddr += currentLen

```

### 1.2 递归下降算法

*   • 从入口开始，依次解析每一条指令，遇到分支指令时递归进入分支。
    
*   • 递归下降算法**强调控制流的概念**。控制流根据一条指令是否被另一条指令**引用**来决定是否对其进行反汇编。
    
*   • 递归下降算法的缺点在于：可以构造**必然条件**或者**互补条件**，使得反汇编出错。
    

### 1.3 如何构造欺骗采用递归下降方法的反汇编引擎？

x86 指令集的长度不是固定的，有一些指令很短，只有 1 字节，有一些指令比较长，可能达到 5 字节， 甚至更长。不同的指令，其指令长度不是固定的。如果通过巧妙的构造，引导反汇编引擎解析一条错误的指令，扰乱解析指令的长度，就能使反汇编引擎无法按照正确的指令长度依次解析邻接未解析的指令，最终使反汇编引擎输出错误的反汇编结果。

二、 花指令的识别、实现、去除
---------------

### 2.1 无条件转移花指令

*   • 单次无条件转移：这种`jmp`单次跳转只能骗过线性扫描算法，**会被 IDA 识别（递归下降）**。
    

```
jmp LABEL1
  db junk_code;
LABEL1:

```

*   • 多次无条件转移：和单次跳转一样，这种也会被 IDA 识别。
    

```
__asm {
    jmp LABEL1;
    _emit 68h;
LABEL1:
    jmp LABEL2;
    _emit 0CDh;
    _emit 20h;
LABEL2:
    jmp LABEL3;
    _emit 0E8h;
LABEL3:
}

```

### 2.2 互补跳转花指令

```
__asm {
    jz Label;
    jnz Label;
    _emit 0xC7;
Label:
}

```

简单的实现（msvc、32 位）：

```
#include<cstdio>
int main() {
    __asm {
        jz s;
        jnz s;
        _emit 0xe9;
    s:
    }
    // 这段asm主要的影响是0xe9会作为下一条指令的起始
    printf("hello world!\n");
    return 0;
}

```

编译结果：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODCKqqxiaM61HwPpppPLmynDvR0W6jDw0deia1I34hMkx49axGVpxSt3vQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

这种互补跳转让 ida 也识别错误，只需要将`E9`改为单字节指令，并且改为单字节指令后不影响程序正常功能即可。最好的单字节指令：**nop**。之后 ida 就能正常分析。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODmy7BpB17WJZ5SRsTkOtLg2B1CxoggE24zR6Hh6AGziazZNc1vwFAGmg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)

### 2.3 跳转构造花指令

```
__asm {
    push ebx;
    xor ebx, ebx;
    test ebx, ebx;
    jnz LABEL7;
    jz    LABEL8;
LABEL7:
    _emit 0xC7;
LABEL8:
    pop ebx;
}

```

简单的实现（msvc、32 位）：

```
#include<stdio.h>
int main() {
    __asm {
        push ebx;
        xor ebx, ebx;
        test ebx, ebx;
        jnz s1;
        jz  s2;
    s1:
        _emit 0xe9;
    s2:
        pop ebx;一定要恢复ebx
    }
    printf("hello world!\n");
    return 0;
}

```

ida 将`s2`识别为`s1+1`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODRdugkmS5L4g7lSnD2xuhia1e74gUtmStHIbTXSNGgV3rsNuJjOclZbQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

同样 patch 掉 E9 就能正常识别。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODHfXLvlPUt24Ypen0SSLQy5qv5nWw1ADMAia2QPI3UmibMvscbZ54CquQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3)

### 2.4 call&ret 花指令

```
__asm {
    call LABEL9;
    _emit 0x83;                        ;1字节
LABEL9:
    ;ret后不会跳转到_emit 0x83;而是跳转到汇编__emit 0xF3; 之后, 换句话说就是改变了返回地址
    add dword ptr ss : [esp] , 8;    ;5字节
        ret;                        ;1字节
    __emit 0xF3;                    ;1字节
}

```

简单的实现（msvc、32 位）：

```
#include<stdio.h>
int main() {
    __asm {
        call s;
        _emit 0x83;
    s:
        add dword ptr ss : [esp] , 8;
        ret;
        _emit 0xf3;
    }
    printf("hello world!\n");
    return 0;
}

```

结合 ida 反汇编来解释为什么是 add 指令的第二个操作数是 8。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODeyiaqUQuRqjKFicJp5ibm7yGXAZWoBZLXstVqpJkkFCjZMs7G2mg9XuiaA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)

call 指令会将返回地址 (**0x41188C**) 压栈，此时`[esp]`中就是返回地址，构造花指令一共使用了`1+5+1+1=8`字节的指令，因此在花指令中需要 ret 到正确的 eip，而正确的 eip 是`[esp]+8`，所以 **add 指令的第二个操作数是多少取决于用多少个字节来构造花指令**。此处是 8 字节，花指令中 ret 时，[esp] 已经是正确的返回地址 (**0x41188C+8=0x411894**)。

patch 的时候只需将所有的花指令 (**[0x41188C,0x411894)**) 改为 nop 即可。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODpibNv1Kxuw8sdJXWMcic9wFLv9LvBBR1VSWkDbmZ2VMyYJdOc9icLDKYA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5)

### 2.5 裸函数花指令

这种裸函数，**能构造更复杂的花指令，实现和去除的代价也更高**，下面给一个最简单的实现，函数的功能是给指针 a 指向的地址写入整数 5。

```
#include<stdio.h>
#include<stdlib.h>

//naked:裸函数，编译器不维护该函数的栈帧，由程序员自己维护。
void _declspec(naked)_cdecl example5(int* a){
    __asm{
        push ebp
        mov ebp, esp
        sub esp, 0x40;为局部变量分配空间。
        push ebx
        push esi
        push edi

        ;模拟初始化
        mov eax, 0xCCCCCCCC
        mov ecx, 0x10
        ;edi指向栈顶
        lea edi, dword ptr ds : [ebp - 0x40]
        ;使用stosd指令将EAX中的值（0xCCCCCCCC）复制到EDI指向的内存地址，共复制ECX（0x10）次。
        rep stos dword ptr es : [edi] 
    }
    *a = 5;
    __asm{
        call LABEL9;
        ;等价于 call [eip+1]
        _emit 0xE8;
        _emit 0x01;
        _emit 0x00;
        _emit 0x00;
        _emit 0x00;
    LABEL9:
        push eax;
        push ebx;
        lea eax, dword ptr ds : [ebp - 0x0] ; //将ebp的地址存放于eax
        add dword ptr ss : [eax - 0x50] , 26; //该地址存放的值正好是函数返回值,
        //不过该地址并不固定,根据调试所得。加26正好可以跳到下面的mov指令,该值也是调试计算所得
        pop eax;
        pop ebx;
        pop eax;
        jmp eax;
        ;等价于 call [eip+3]
        _emit 0xE8;
        _emit 0x03;
        _emit 0x00;
        _emit 0x00;
        _emit 0x00;
        mov eax, dword ptr ss : [esp - 8] ; //将原本的eax值返回eax寄存器
    }
    __asm{
        pop edi
        pop esi
        pop ebx
        mov esp, ebp
        pop ebp
        ret
    }
}
int main() {
    printf("hello world!\n");
    int *b = (int*)malloc(sizeof(int));
    example5(b);
    printf("b = %d\n", *b);
    free(b);
    return 0;
}

```

在 ida9.2 中，无需 patch 就能反汇编，因为其中的花指令是由**无 ret 的 call 和 jmp** 实现的，这对 ida 的递归扫描算法没有任何影响。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODth1hy3mpic0eGOVNHpjUW6jm7kNYL8LnkzHXC68mh8k3mb5MicuOfJQw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)

### 2.6 函数返回值花指令

有些函数在特定情况下返回值是固定的，可以用以构造花指令。比如我们自己写的函数，返回值可以是**任意非零整数**，就可以自己**构造永恒跳转**。

还有些 API 函数也是如此，比如在 Win 下`HMODULE LoadLibraryA(LPCSTR lpLibFileName);`函数，如果我们故意传入一个**不存在**的模块名称，那么他就会返回一个**确定的值 NULL**，此时就可以通过这个函数来构造永恒跳转。

这种花指令和跳转构造的花指令原理一致，只是需要显式的`xor ebx, ebx`，因此也不要还原`ebx`，更加隐蔽。

```
#include<stdio.h>
#include<Windows.h>

int main() {
    LoadLibrary(L"./deadbeef");
    __asm {
        cmp eax, 0;
        jc LABEL6_1;
        jnc LABEL6_2;
        LABEL6_1:
            _emit 0xE8;
        LABEL6_2:
    }
    printf("Hello World!\n");
    return 0;
}

```

代码中存在`jc/jnc`，但是实际上不存在`deadbeef`这个模块，所以`LoadLibrary`返回值一点是`NULL`，只有`jnc`起作用。**这种花指令的识别需要熟悉`api`函数的返回值，以及正确分析程序所需的资源和运行时环境**。识别与去除需要一定的综合判断和细节。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODmFnqHSHIY0hSn5wUKQGZX6WGPXONGUpliaiaNbHG2luK6N29Xl3lPBpQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=7)

动态调试看程序的执行流实际跳转到什么位置，其余的 junk_code 直接 patch 即可。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODVnvmta8BMlbfnEibRdHYIQdtlgibtBrH1D54ibJhLRuAJyvyyta69ekHQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=8)

### 2.7 指令数据复用花指令（典型的可执行花指令）

正常情况的代码，编译后一个字节只属于一条指令，ida 在反汇编分析时，也遵循这个规。，但是如果是用_emit 精心设计 opcode，可以实现一个字节属于多条指令。比如：

```
#include<stdio.h>
int main() {
    // 可以放在任何位置的花指令
    __asm {
        _emit 0xeb;
        _emit 0xff;
        _emit 0xc0;
        _emit 0x48;
    }
    printf("hello world!\n");
    return 0;
}

```

在 ida 中如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODaTtCaDAnSNiatIpP8spwZqLnoVrnk9dGciccx5bM1jDZ5ibfOe7F6dr6g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=9)

先解释一下指令`EB xx`，含义是：`jmp rel8`，短跳转，`xx`是一个 8 比特的偏移量 (有符号数)。如果`xx = 0xff`，而在计算机中，整数都是以补码表示，8 比特的有符号数 0xff 是 - 1。因此，`EB FF`等价于`jmp [eip - 1]`。以 ida 中地址为 0x411887 处的代码为例。执行`EB FF`前，eip = 0x411889，执行`EB FF`后，eip = 0x411888，等价于将已经执行过的`FF`重新执行一遍，这时候的`FF C0`和`48`分别是`inc eax`和`dec eax`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODErcndHepdPNicuBc8YEyVb1AicYZYicdtDeld2dW8scTlYA9PsmVuk7tA/640?wx_fmt=png&from=appmsg#imgIndex=10)

等价于什么都没做。但是却让 ida 反汇编出错。这种花指令的识别和去除有较大难度。**需要动态调试才能发现四字节的 junk_code**。

### 2.8 间接跳转花指令

将跳转地址存于寄存器中，需要运行时才能知道具体的跳转地址。这类花指令常见于定长指令集的架构，如 arm。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FMiaMMfBpPgCBaItva8LvNf4byLobzdODkyia8lUNIibR2laNqPHbNmNag75RewoibouBPQjZznicwbVGGH7ibTmFs2g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11)

三、花指令的分析方法
----------

### 3.1 调试观察法

花指令不会干扰正常代码：花指令内部如果涉及到寄存器的使用，一般会将其保存在栈中，利用这个弱点，我们可以通过观察 sp 寄存器来判断花指令的入口和出口。大部分情况下，花指令可以直接使用相同长度的 nop （0x90） 替换。

### 3.2 花指令序列批量替换

在上一节中，我们提到如下花指令可以插入在任意位置，以至于很可能大量出现同样的花指令。

```
__asm {
    _emit 0xeb;
    _emit 0xff;
    _emit 0xc0;
    _emit 0x48;
}

```

这大大降低了开发大量花指令的难度，如果我们总是一条一条的分析、patch，会花很多时间。

考虑这种情况，我们可以将 16 进制数据批量查找替换：EBFFC048  ---->  90909090。

推荐 010 Editor 或者 Winhex。

注意：如果花指令的模式太短，不建议批量替换，避免破坏正常指令。

学习资源
====

立即关注【二进制磨剑】公众号

  

👉👉👉[【IDA 技巧合集】](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1Mjk2MTM1OQ==&action=getalbum&album_id=3353659750835535873#wechat_redirect)👈👈👈

👉👉👉[【Github 安全项目合集】](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1Mjk2MTM1OQ==&action=getalbum&album_id=3257219530066477058#wechat_redirect)👈👈👈

  

[![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/FMiaMMfBpPgAHcgEpVwNlwbjBT4E0PMAEaJrrdB5hsn6LWIqIBWP1YYNyyc3ISBVwjRlkQJB2ALvx5plHg4PHmA/640?wx_fmt=jpeg&from=appmsg&watermark=1#imgIndex=12)](https://mp.weixin.qq.com/s?__biz=MzI1Mjk2MTM1OQ==&mid=2247485678&idx=1&sn=2c4735f06b30c42926d0f12f3991d0c7&scene=21#wechat_redirect)