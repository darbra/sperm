> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aVPlvHkb6ohZ5K_zXVhs0w)

vm 题算是逆向中比较难的一种题型了，在这里详细的记录一下。

```
一





原理

```

程序运行时通过解释操作码（opcode）选择对应的函数（handle）执行。

### vm_init

进行初始化工作。在这个函数里，规定了有几个寄存器，以及有几种不同的操作：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xlop9owntjY0CSV2KIibOib2JalEeicGt7kdTm3KNTJG5gohAdaggSY1ibg/640?wx_fmt=other&from=appmsg)

这样看不明显，创建结构体修复一下，结构体长这个样子：

```
typedef struct
{
    unsigned long R0;      //寄存器
    unsigned long R1;    
    unsigned long R2;   
    unsigned long R4;
    unsigned char *rip;    //指向正在解释的opcode地址
    vm_opcode op_list[OPCODE_N];    //opcode列表，存放了所有的opcode及其对应的处理函数
}vm_cpu;


```

```
typedef struct
{
    unsigned long opcode;
    void (*handle)(void*);
}vm_opcode;


```

修复完的初始化函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XyULq3PU9waNG1IjAZdOgodhQkR5HVksrIm725NFYkLk3efyOER6N1g/640?wx_fmt=other&from=appmsg)

### vm_start

```
void vm_start(vm_cpu *cpu)
{
    
    cpu->eip = (unsigned char*)opcodes;   //这里不是在上面就初始化过了吗？？？
    while((*cpu->eip) != 0xf4)//如果opcode不为RET，就调用vm_dispatcher来解释执行
    {
        vm_dispatcher(*cpu->eip)
    }
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XialACicWvlibAGv8tn1tEtcpOLCSWAYnpq1xic4fnD4SJVyWM4icKkwWXlA/640?wx_fmt=other&from=appmsg)

如果 rip 指向的操作码为 F4 就返回。

### vm_dispatcher

调度器，任务是根据 opcode 选择函数执行。

```
void vm_dispatcher(vm_cpu *cpu)
{
    int i;
    for(i = 0; i < OPCODE_N; i++)
    {    
        if(*cpu->eip == cpu->op_list[i].opcode)
        {
            cpu->op_list[i].handle(cpu);
            break;
        }
    }
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XBtEQ9cyhRmub9SvnRnjKJWhwxCKHSAwNeCbKib3nBg7SclQxujEZxYQ/640?wx_fmt=other&from=appmsg)

循环，找到 opcode 对应的函数。

参考链接：

系统学习 vm 虚拟机逆向_vmware 逆向 - CSDN 博客

https://blog.csdn.net/weixin_43876357/article/details/108570305

```
二





实战1

```

【GWCTF2019babyvm】

### 分析函数

知道了原理之后，就可以分析题目加深理解了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XkCBPs7lrqgQJL9JTibAx1FJicv7tp16WjzJYgGOoJ1tq4ZNibfR1d938A/640?wx_fmt=other&from=appmsg)

主要分析 vm_ini 函数，搞清楚 opcode 对应的操作。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XTczpFs3JzFsvc1Wia5xPyr5fXOedD8gYe8lib7TSiatDHnN12Du5ceViaw/640?wx_fmt=other&from=appmsg)

#### 0xF1

可以把这个当作 mov 指令。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XWJ6XMDURKGZVkCCosibqcnttmCLo93EOib94tsYTWpc1ItNGhiaApDkxg/640?wx_fmt=other&from=appmsg)

```
v2 = (int *)(a1->_rip + 2);


```

v2 指向当前指令往后偏移两个字节的位置。

```
a1->_rip += 6LL;


```

说明这条指令的大小为 6 字节。

可以看出这条指令可以将复制一些值到寄存器，也可以将寄存器的值复制过去。

拿第一组数据来看，a1->rip+1 是 0xE1，那么将 input[*v2] 的值存入寄存器 R0，也就是 R0=input[0]。

```
0xF1, 0xE1, 0x00, 0x00, 0x00, 0x00,


```

#### 0xF2

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9X2G0CgzZa1wicSOickXqUe87G72rD73yB2j49j5JpUmegpp4xbTk59cFg/640?wx_fmt=other&from=appmsg)

R0=R0^R1，指令长度 1 字节。

#### 0xF5

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9X5OndYL56QxpMZ511DS6GjKydUZBxNeVff7jALK5tGXKWp3CJMhWN3Q/640?wx_fmt=other&from=appmsg)

读取输入并判断长度。

**0xF4**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XY5mKnPa60CMZZMyegicGKpMiawheOWF2XM7BhfvHcnA1icEsBX1BxsGGw/640?wx_fmt=other&from=appmsg)

空操作，nop，作用是将 rip+1。

#### 0xF7

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xcq9FxbXAZNOp0k1nib8WdXt88fLNh3sL9TNCC3LPFq8BibQc5d18fMHw/640?wx_fmt=other&from=appmsg)

R0*=R3，指令长度为 1。

#### 0xF8

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xryl0cTo64OkTlEqL6xwL7ESHDNrJQ2lJMflgsO0YNevXov6ncG2EKA/640?wx_fmt=other&from=appmsg)

交换 R0 和 R1 的数值。

#### 0xF6

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XcbUtPsSuFaa2u13raT8K1sbzG5Uk6UekibtXstFmCcV2Tz0CPBxiaeibQ/640?wx_fmt=other&from=appmsg)

R0=R2+2*R1+3*R0

### 翻译

```
#include<stdio.h>
void myswap(char*a,char*b);
int main()
{
    unsigned char opcode[575] = {
    0xF5, 0xF1, 0xE1, 0x00, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x20, 0x00, 0x00, 0x00, 0xF1, 0xE1, 
    0x01, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x21, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x02, 0x00, 0x00, 
    0x00, 0xF2, 0xF1, 0xE4, 0x22, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x03, 0x00, 0x00, 0x00, 0xF2, 0xF1, 
    0xE4, 0x23, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x04, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x24, 0x00, 
    0x00, 0x00, 0xF1, 0xE1, 0x05, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x25, 0x00, 0x00, 0x00, 0xF1, 
    0xE1, 0x06, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x26, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x07, 0x00, 
    0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x27, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x08, 0x00, 0x00, 0x00, 0xF2, 
    0xF1, 0xE4, 0x28, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x09, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x29, 
    0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0A, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x2A, 0x00, 0x00, 0x00, 
    0xF1, 0xE1, 0x0B, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x2B, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0C, 
    0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x2C, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0D, 0x00, 0x00, 0x00, 
    0xF2, 0xF1, 0xE4, 0x2D, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0E, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 
    0x2E, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0F, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x2F, 0x00, 0x00, 
    0x00, 0xF1, 0xE1, 0x10, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x30, 0x00, 0x00, 0x00, 0xF1, 0xE1, 
    0x11, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x31, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x12, 0x00, 0x00, 
    0x00, 0xF2, 0xF1, 0xE4, 0x32, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x13, 0x00, 0x00, 0x00, 0xF2, 0xF1, 
    0xE4, 0x33, 0x00, 0x00, 0x00, 0xF4, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
    0xF5, 0xF1, 0xE1, 0x00, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x01, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 
    0x00, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x01, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x02, 0x00, 0x00, 0x00, 
    0xF2, 0xF1, 0xE4, 0x01, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x02, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x03, 
    0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x02, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x03, 0x00, 0x00, 0x00, 
    0xF1, 0xE2, 0x04, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x03, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x04, 
    0x00, 0x00, 0x00, 0xF1, 0xE2, 0x05, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x04, 0x00, 0x00, 0x00, 
    0xF1, 0xE1, 0x05, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x06, 0x00, 0x00, 0x00, 0xF2, 0xF1, 0xE4, 0x05, 
    0x00, 0x00, 0x00, 0xF1, 0xE1, 0x06, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x07, 0x00, 0x00, 0x00, 0xF1, 
    0xE3, 0x08, 0x00, 0x00, 0x00, 0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, 0xF6, 0xF7, 0xF1, 0xE4, 0x06, 
    0x00, 0x00, 0x00, 0xF1, 0xE1, 0x07, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x08, 0x00, 0x00, 0x00, 0xF1, 
    0xE3, 0x09, 0x00, 0x00, 0x00, 0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, 0xF6, 0xF7, 0xF1, 0xE4, 0x07, 
    0x00, 0x00, 0x00, 0xF1, 0xE1, 0x08, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x09, 0x00, 0x00, 0x00, 0xF1, 
    0xE3, 0x0A, 0x00, 0x00, 0x00, 0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, 0xF6, 0xF7, 0xF1, 0xE4, 0x08, 
    0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0D, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x13, 0x00, 0x00, 0x00, 0xF8, 
    0xF1, 0xE4, 0x0D, 0x00, 0x00, 0x00, 0xF1, 0xE7, 0x13, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0E, 0x00, 
    0x00, 0x00, 0xF1, 0xE2, 0x12, 0x00, 0x00, 0x00, 0xF8, 0xF1, 0xE4, 0x0E, 0x00, 0x00, 0x00, 0xF1, 
    0xE7, 0x12, 0x00, 0x00, 0x00, 0xF1, 0xE1, 0x0F, 0x00, 0x00, 0x00, 0xF1, 0xE2, 0x11, 0x00, 0x00, 
    0x00, 0xF8, 0xF1, 0xE4, 0x0F, 0x00, 0x00, 0x00, 0xF1, 0xE7, 0x11, 0x00, 0x00, 0x00,0xf4};
    //翻译
    for (int i = 0; i < 575; )
    {
        if (opcode[i] == 0xf1) // mov
        {
            switch (opcode[i+1])
            {
            case 0xE1:
                //a1->R0 = *((char *)input + *v2);
                printf("R0=input[%d]\n",*(int*)&opcode[i+2]);
                break;
            case 0xE2:
                //a1->R1 = *((char *)input + *v2);
                printf("R1=input[%d]\n",*(int*)&opcode[i+2]);
                break;
            case 0xE3:
                //a1->R2 = *((char *)input + *v2);
                printf("R2=input[%d]\n",*(int*)&opcode[i+2]);
                break;
            case 0xE4:
                //*((_BYTE *)input + *v2) = a1->R0;
                printf("input[%d]=R0\n",*(int*)&opcode[i+2]);
                break;
            case 0xE5:
                //a1->R3 = *((char *)input + *v2);
                printf("R3=input[%d]\n",*(int*)&opcode[i+2]);
                break;
            case 0xE7:
                //*((_BYTE *)input + *v2) = a1->R1;
                printf("input[%d]=R1\n",*(int*)&opcode[i+2]);
                break;
            default:
                printf("mov wrong!!!!!\n");
                break;
            }
            i+=6;
        }
        else if (opcode[i] == 0xf2) // xor
        {
            printf("R0=R0^R1\n");
            i+=1;
        }
        else if (opcode[i] == 0xf5) // scanf
        {
            printf("please input:\n");
            i+=1;
        }
        else if (opcode[i] == 0xf4) // nop
        {
            printf("0xF4 nop\n");
            printf("\n");
            i+=1;
        }
        else if (opcode[i] == 0xf7) //*
        {
            printf("R0*=R3\n");
            i+=1;
        }
        else if (opcode[i] == 0xf8) // change
        {
            printf("change(R0,R1)\n");
            i+=1;
        }
        else if (opcode[i] == 0xf6) //
        {
            printf("R0=R2+2*R1+3*R0\n");
            i+=1;
        }
        else if(opcode[i]==0)
        {
            printf("nop\n");
            i++;
        }
    }
    printf("over!!");
    
    return 0;
}


```

得到了两段程序，第一个是简单的异或。

```
please input:       R1=18
R0=input[0]        
R0=R0^R1       //input0^18
input[32]=R0      
R0=input[1]
R0=R0^R1
input[33]=R0
R0=input[2]
R0=R0^R1
input[34]=R0
R0=input[3]
R0=R0^R1
input[35]=R0
R0=input[4]
R0=R0^R1
input[36]=R0
R0=input[5]
R0=R0^R1
input[37]=R0
R0=input[6]
R0=R0^R1
input[38]=R0
R0=input[7]
R0=R0^R1
input[39]=R0
R0=input[8]
R0=R0^R1
input[40]=R0
R0=input[9]
R0=R0^R1
input[41]=R0
R0=input[10]
R0=R0^R1
input[42]=R0
R0=input[11]
R0=R0^R1
input[43]=R0
R0=input[12]
R0=R0^R1
input[44]=R0
R0=input[13]
R0=R0^R1
input[45]=R0
R0=input[14]
R0=R0^R1
input[46]=R0
R0=input[15]
R0=R0^R1
input[47]=R0
R0=input[16]
R0=R0^R1
input[48]=R0
R0=input[17]
R0=R0^R1
input[49]=R0
R0=input[18]
R0=R0^R1
input[50]=R0
R0=input[19]
R0=R0^R1
input[51]=R0
0xF4 nop


```

```
for(int i=0;i<21;i++)
{
    printf("%c",cpdata[i]^18);
}
//This_is_not_flag_233


```

第二段

```
please input:
R0=input[0]
R1=input[1]
R0=R0^R1
input[0]=R0

R0=input[1]
R1=input[2]
R0=R0^R1
input[1]=R0

R0=input[2]
R1=input[3]
R0=R0^R1
input[2]=R0

R0=input[3]
R1=input[4]
R0=R0^R1
input[3]=R0

R0=input[4]
R1=input[5]
R0=R0^R1
input[4]=R0

R0=input[5]
R1=input[6]
R0=R0^R1
input[5]=R0

R0=input[6]          //input[6]=
R1=input[7]
R2=input[8]
R3=input[12]
R0=R2+2*R1+3*R0
R0*=R3
input[6]=R0

R0=input[7]              //input[7]=
R1=input[8]
R2=input[9]
R3=input[12]
R0=R2+2*R1+3*R0
R0*=R3
input[7]=R0

R0=input[8]                  //input[8]=(input[8]/R3-2*input[9]-input[10])/4
R1=input[9]
R2=input[10]
R3=input[12]
R0=R2+2*R1+3*R0
R0*=R3
input[8]=R0

R0=input[13]      //置换  13  19
R1=input[19]
change(R0,R1)
input[13]=R0
input[19]=R1

R0=input[14]       //置换14 18
R1=input[18]
change(R0,R1)
input[14]=R0
input[18]=R1

R0=input[15]       //置换15 17
R1=input[17]
change(R0,R1)
input[15]=R0
input[17]=R1
0xF4 nop   

over!!


```

第二段的比较数据就在第一个比较数据的附近，在得到假 flag 之后，查看该处的数据。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xvn5Wib4olK48dm5wxCA9zfzrGIqrf0E0R4fTv49me94kf528vr2rdlg/640?wx_fmt=other&from=appmsg)

发现上面有一个可疑数据，查看它的交叉引用，可以跟进一个比较函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xwg2B04zCloGywqAyvkLsRqicJeTzoBJr1LODq2JvjbwWthPqXCu5WTA/640?wx_fmt=other&from=appmsg)

但是比较奇怪的是，这个函数没有被调用过，之前做过 actf 的一道题，函数通过栈溢出覆盖了返回值，从而被调用，但是这一题就算输入了真正的 flag，也不会调用这一处函数，所以感觉有点……，虽然有提示，虽然 opcode 有两个 0xf5 调用输入，但还是有些生硬了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xj77BiclW13oEUEK6n2FPDcytcudrZGlDYib0wzU4WScaLo6TM9scO0Ww/640?wx_fmt=other&from=appmsg)

```
#include<stdio.h>
void myswap(char*a,char*b);
int main()
{
    for(int i=30;i<127;i++)
    {
        if(realdata[8]==(unsigned char)((realdata[10]+2*realdata[9]+3*i)*realdata[12]))
        {
            //printf("flag[8]==%c\n",i);
            realdata[8]=i;
        }
    }
    for(int i=30;i<127;i++)
    {
        if(realdata[7]==(unsigned char)((realdata[9]+2*realdata[8]+3*i)*realdata[12]))
        {
            //printf("flag[7]==%c\n",i);
            realdata[7]=i;
        }
    }
    for(int i=30;i<127;i++)
    {
        int a=(realdata[8]+2*realdata[7]+3*i)*realdata[12];
        if(realdata[6]==(unsigned char)((realdata[8]+2*realdata[7]+3*i)*realdata[12]))
        {
            //printf("flag[6]==%c\n",i);
            realdata[6]=i;
        }
    }
    myswap(&realdata[13],&realdata[19]);
    myswap(&realdata[14],&realdata[18]);
    myswap(&realdata[15],&realdata[17]);
    for(int i=0;i<20;i++)
    {
        printf("%c",realdata[i]);
    }
    
    return 0;
}

void myswap(char* a,char* b)
{
   char t=*a;
   *a=*b;
   *b=t;
}

//Y0u_hav3_r3v3rs3_1t!


```

```
三





实战2

```

hgame2023 vm

### 分析结构体

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XugpBMMowVJXVmF9jicJGibYaTAKALIxbbYicKzVicNIgZwGPTlmV6LTKpg/640?wx_fmt=other&from=appmsg)

直接去分析 sub_1400010B0，不难看出这是 vm_start 函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XwP6yNnD1nmtHOvoq8nPN2vIiblxDu7RxDWa8oWR7yYISNp9MXZT82AQ/640?wx_fmt=other&from=appmsg)

byte_140005360 自然就是 opcode，sub_140001940 是调度器。这一题创建结构体比上面那道困难一些，因为我没找到 vm_init 函数，只能根据 dispatcher 调度的函数来判断。从 *(unsigned int *)(a1 + 24) 也不难看出，a1+24 指向的是 eip，大小为四字节。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xvn4SiaQxS1CzNc37k2jtCAPR1kE7bVKBnjctJ90RIImEyp9CB5P3dwA/640?wx_fmt=other&from=appmsg)

从这里看以看出 handle 是八字节的，从第一个函数分析。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XicXcMNLBgztppRLI3Kb8VQA1t8wf04OGBB0S6V4pI4JhTsu5JFnZjCQ/640?wx_fmt=other&from=appmsg)

不难猜测出这是个 mov 之类的指令，从 v2 那里可以看到 opcode[a1[6]+1] 应该就是 sp+1 ，也就是 a[6] 是上面分析的 eip，所以可以判断通用寄存器有 6 个，大小是 dword。上面的 case0~7，说明有八个函数。接着往下看，分析第二个函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XlC0odCVhT8A8kXxcxtD62Nm3hA8A7z3uREBzlfaysE6guGrEJnAsZA/640?wx_fmt=other&from=appmsg)

这个很容易想到 push 指令，存数据之前指针先增加，再结合第三个函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XaI0jTP7CNK0CCQEkdlgdY3waRdZ4SufWXJLwWojPWChmtScUbNprdQ/640?wx_fmt=other&from=appmsg)

取数据之后，指针自减，再加上和上面猜的的 push 是成对出现的，那么可以推测 a[7] 是栈指针 esp。再往下只有一个比较陌生的东西：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9X02Rao9ZmCx9ibT6sQ4RB2zNsK4wYZdnphK08FXHVshNHOQlHe1E1ibvg/640?wx_fmt=other&from=appmsg)

在 vm_cpu 结构体的第 32 字节处，有一个一字节大小的变量，相等赋值 0，不等赋值 1，有点像标志位 zf，接着根据下面的函数分析。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XKzJmDutU37icbvdQ0gYxpxO3uUiaEiac25iaMZ47mUEXEt8oz6jUyqqc8g/640?wx_fmt=other&from=appmsg)

这个函数会判断我们刚才提到的那个一字节的变量从而选择不同的指令，所以可以推测其是一个一字节大小的标志寄存器。

综上：

vm_cpu 有 6 个通用寄存器，一个 eip，一个 esp，还有一个 zf。

创建结构体

```
typedef struct
{
    unsigned int R[6];      
    unsigned int eip;
    unsigned int esp;
    char          zf; 
    vm_opcode op_list[OPCODE_N];    
}vm_cpu;


```

```
typedef struct
{
    unsigned int opcode;
    void (*handle)(void*);
}vm_opcode;


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XLw2LbtCO0GALP5cuwichPV7EaLmj0y6GICtUIbpzqfDicqoFWpxKWXKQ/640?wx_fmt=other&from=appmsg)

### 分析函数

逐个分析函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XJ47WicEHtjYu1EcNZMBKdQicEZo7iah2HuqIozHF1Nv76u2mOxAjF6ejg/640?wx_fmt=other&from=appmsg)

#### fun0

都在注释上了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XtNRhhcBRjLcEGHZzUHhXtZIyC3xWNtos4IiaxMoZzXz8FFzeaOgMhsg/640?wx_fmt=other&from=appmsg)

#### fun1

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xaa0aauPbhUHgrhnyawjKkW7rqvHicX7yZIHFXaI61Bebnja8ZEtzibkg/640?wx_fmt=other&from=appmsg)

#### fun2

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XActZaHLt9j06ZjnGbOI60UYSDoSgLFcJNVvMJPXBRnlsOsChx8ndSg/640?wx_fmt=other&from=appmsg)

#### fun3

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XwiagAQosuNGBy6Rq3GB8DcZgjN3yETfWKh0N8RQ6JJspyPZeFefBsng/640?wx_fmt=other&from=appmsg)

这个函数就是实现的 +、-、*、^、<< 等操作。

#### fun4

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XQiaDPm3v37Dzic1DYCNVTOY5UI5I7IkyKkMabcD1h51t4JKRk8A0x6mA/640?wx_fmt=other&from=appmsg)

更改标志寄存器。

#### fun5

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9X2TBXdw0EwOhHKS8uVvWahL6p9RNuqXjZibdgRKEfWG0Qniara4twZGOw/640?wx_fmt=other&from=appmsg)

更改 eip 的值，而且没有任何条件，可以推测出这是条 jmp 指令。

#### fun6

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XGBOCnibIZH0HPVPfREmibiagC1RRmq9QRJEeia7wQ0hlgJeHOTamYz5MZw/640?wx_fmt=other&from=appmsg)

标志寄存器为 0 则发生跳转，所以是条 jnz 指令。

#### fun7

和 fun6 对应，是 jz 指令。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9Xn4ks2zLxwdOeJ5OtswVt4k1B8mewqvGt6VuNCib45J98qfRKC7gOyJA/640?wx_fmt=other&from=appmsg)

### 翻译

```
#include<stdio.h>
int main()
{
    unsigned int R[6]={0,0,0,0,0,0};
    int zf;
    int esp=0;
    unsigned int mystack[80]={0};
    unsigned char opcode[137] = {
    0x00, 0x03, 0x02, 0x00, 0x03, 0x00, 0x02, 0x03, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x01, 0x00, 
    0x00, 0x03, 0x02, 0x32, 0x03, 0x00, 0x02, 0x03, 0x00, 0x00, 0x00, 0x00, 0x03, 0x00, 0x01, 0x00, 
    0x00, 0x03, 0x02, 0x64, 0x03, 0x00, 0x02, 0x03, 0x00, 0x00, 0x00, 0x00, 0x03, 0x03, 0x01, 0x00, 
    0x00, 0x03, 0x00, 0x08, 0x00, 0x02, 0x02, 0x01, 0x03, 0x04, 0x01, 0x00, 0x03, 0x05, 0x02, 0x00, 
    0x03, 0x00, 0x01, 0x02, 0x00, 0x02, 0x00, 0x01, 0x01, 0x00, 0x00, 0x03, 0x00, 0x01, 0x03, 0x00, 
    0x03, 0x00, 0x00, 0x02, 0x00, 0x03, 0x00, 0x03, 0x01, 0x28, 0x04, 0x06, 0x5F, 0x05, 0x00, 0x00, 
    0x03, 0x03, 0x00, 0x02, 0x01, 0x00, 0x03, 0x02, 0x96, 0x03, 0x00, 0x02, 0x03, 0x00, 0x00, 0x00, 
    0x00, 0x04, 0x07, 0x88, 0x00, 0x03, 0x00, 0x01, 0x03, 0x00, 0x03, 0x00, 0x00, 0x02, 0x00, 0x03, 
    0x00, 0x03, 0x01, 0x28, 0x04, 0x07, 0x63, 0xFF, 0xFF   };
    unsigned int input[200] = {
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x0000009B, 0x000000A8, 0x00000002, 0x000000BC, 0x000000AC, 0x0000009C, 
    0x000000CE, 0x000000FA, 0x00000002, 0x000000B9, 0x000000FF, 0x0000003A, 0x00000074, 0x00000048, 
    0x00000019, 0x00000069, 0x000000E8, 0x00000003, 0x000000CB, 0x000000C9, 0x000000FF, 0x000000FC, 
    0x00000080, 0x000000D6, 0x0000008D, 0x000000D7, 0x00000072, 0x00000000, 0x000000A7, 0x0000001D, 
    0x0000003D, 0x00000099, 0x00000088, 0x00000099, 0x000000BF, 0x000000E8, 0x00000096, 0x0000002E, 
    0x0000005D, 0x00000057, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x000000C9, 0x000000A9, 0x000000BD, 0x0000008B, 
    0x00000017, 0x000000C2, 0x0000006E, 0x000000F8, 0x000000F5, 0x0000006E, 0x00000063, 0x00000063, 
    0x000000D5, 0x00000046, 0x0000005D, 0x00000016, 0x00000098, 0x00000038, 0x00000030, 0x00000073, 
    0x00000038, 0x000000C1, 0x0000005E, 0x000000ED, 0x000000B0, 0x00000029, 0x0000005A, 0x00000018, 
    0x00000040, 0x000000A7, 0x000000FD, 0x0000000A, 0x0000001E, 0x00000078, 0x0000008B, 0x00000062, 
    0x000000DB, 0x0000000F, 0x0000008F, 0x0000009C, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00004800, 0x0000F100, 
    0x00004000, 0x00002100, 0x00003501, 0x00006400, 0x00007801, 0x0000F900, 0x00001801, 0x00005200, 
    0x00002500, 0x00005D01, 0x00004700, 0x0000FD00, 0x00006901, 0x00005C00, 0x0000AF01, 0x0000B200, 
    0x0000EC01, 0x00005201, 0x00004F01, 0x00001A01, 0x00005000, 0x00008501, 0x0000CD00, 0x00002300, 
    0x0000F800, 0x00000C00, 0x0000CF00, 0x00003D01, 0x00004501, 0x00008200, 0x0000D201, 0x00002901, 
    0x0000D501, 0x00000601, 0x0000A201, 0x0000DE00, 0x0000A601, 0x0000CA01, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000};
    for(int i=0;i<137;)
    {
        if(opcode[i]==0)
        {
            int v2=opcode[i+1];
            if(v2)
            {
                switch (v2)
                {
                case 1:
                    input[R[2]]=R[0];
                    printf("input[%d]=R[0](%d)\n",R[2],R[0]);
                    break;
                case 2:
                    printf("R[%d]=R[%d](%d)\n",opcode[i+2],opcode[i+3],R[opcode[i+3]]);
                    R[opcode[i+2]]=R[opcode[i+3]];
                    //printf("R[%d]=R[%d]\n",opcode[i+2],opcode[i+3]);
                    break;
                case 3:
                    printf("R[%d]=opcode[%d](%d)\n",opcode[i+2],i+3,opcode[i+3]);
                    R[opcode[i+2]]=opcode[i+3];
                    //printf("R[%d]=opcode[%d]\n",opcode[i+2],i+3);
                    break;
                default:
                    break;
                }
            }
            else
            {
                printf("R0=input[%d](%d)\n",R[2],input[R[2]]);
                R[0]=input[R[2]];
                //printf("R0=input[R2]\n");
            }
            i+=4;
        }
        else if(opcode[i]==1)
        {
            int v2=opcode[i+1];
            if (v2)
            {
                switch (v2)
                {
                case 1u:
                    //stack[++a1->_esp] = a1->R0; // push R0
                    mystack[++esp]=R[0];
                    printf("push R0(%d)\n",R[0]);
                    break;
                case 2u:
                    //stack[++a1->_esp] = a1->R2; // push R2
                    mystack[++esp]=R[2];
                    printf("push R2(%d)\n",R[2]);
                    break;
                case 3u:
                    //stack[++a1->_esp] = a1->R3; // push R3
                    mystack[++esp]=R[3];
                    printf("push R3(%d)\n",R[3]);
                    break;
                }
            }
            else
            {
                //stack[++a1->_esp] = a1->R0; // push R0
                 mystack[++esp]=R[0];
                 printf("push R0(%d)\n",R[0]);
            }
            i+=2;
        }
        else if(opcode[i]==2)
        {
            int v2 =opcode[i+1];
            if (v2)
            {
                 switch (v2)
                 {
                 case 1u:
                    //a1->R1 = stack[a1->_esp--]; // pop R1
                    R[1]=mystack[esp--];
                    printf("pop R1(%d)\n",R[1]);
                    break;
                 case 2u:
                    //a1->R2 = stack[a1->_esp--]; // pop R2
                    R[2]=mystack[esp--];
                    printf("pop R2(%d)\n",R[2]);
                    break;
                 case 3u:
                    //a1->R3 = stack[a1->_esp--]; // pop R3
                    R[3]=mystack[esp--];
                    printf("pop R3(%d)\n",R[3]);
                    break;
                 }
            }
            else
            {
                 //a1->R0 = stack[a1->_esp--]; // pop R0
                 R[0]=mystack[esp--];
                 printf("pop R0(%d)\n",R[0]);
            }
            i+=2;
        }
        else if(opcode[i]==3)
        {
            switch (opcode[i+1])
            {
            case 0u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) += *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]+=R[%d]       %d+=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]+=R[opcode[i+3]];
                 //printf("R[%d]+=R[%d]       %d+=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 break;
            case 1u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) -= *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]-=R[%d]       %d-=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]-=R[opcode[i+3]];
                 //printf("R[%d]-=R[%d]       %d-=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 break;
            case 2u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) *= *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]*=R[%d]     %d*=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]*=R[opcode[i+3]];
                 //printf("R[%d]*=R[%d]\n     %d*=%d",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 break;
            case 3u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) ^= *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]^=R[%d]     %d^=%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]^=R[opcode[i+3]];
                 //printf("R[%d]^=R[%d]\n",opcode[i+2],opcode[i+3]);
                 break;
            case 4u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) <<= *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]<<=R[%d]        %d<<%d\n",opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]<<=R[opcode[i+3]];
                 //printf("R[%d]<<=R[%d]\n",opcode[i+2],opcode[i+3]);
                 //*(&a1->R0 + opcode[a1->_eip + 2]) &= 0xFF00u;
                 printf("R[%d]&=0xff00;     %d&=0xff00\n",opcode[i+2],R[opcode[i+2]]);
                 R[opcode[i+2]]&=0xff00;
                 //printf("R[%d]&=0xff00;\n",opcode[i+2]);
                 break;
            case 5u:
                 //*(&a1->R0 + opcode[a1->_eip + 2]) = (unsigned int)*(&a1->R0 + opcode[a1->_eip + 2]) >> *(&a1->R0 + opcode[a1->_eip + 3]);
                 printf("R[%d]=R[%d]>>R[%d]     %d=%d>>%d\n",opcode[i+2],opcode[i+2],opcode[i+3],R[opcode[i+2]],R[opcode[i+2]],R[opcode[i+3]]);
                 R[opcode[i+2]]=R[opcode[i+2]]>>R[opcode[i+3]];
                 //printf("R[%d]=R[%d]>>R[%d]\n",opcode[i+2],opcode[i+2],opcode[i+3]);
                 break;
            default:
                 break;
            }
            i+=4;
        }
        else if(opcode[i]==4)
        {
            if (R[0] == R[1])                      
            {
               zf=0;
               printf("R0=R1-->zf=0\n");
            }
            if (R[0] != R[1])
            {
               // a1->_ezf = 1;
               zf = 1;
               printf("R0!=R1-->zf=1\n");
            }
            i+=1;
        }
        else if(opcode[i]==5)
        {
            i=opcode[i+1];
            printf("jmp to fun%d\n",i);
        }
        else if(opcode[i]==6)
        {
            if (zf==1)  
            {
                 //result = (unsigned int)(a1->_eip + 2); // 相当于不跳转
                 i+=2;
                 printf("zf=1,eip+2\n");
            }                             // 如果等于zf==1
            else
            {
                i=opcode[i+1];
                printf("zf!=1,jmp to fun%d\n",i);
            }
        }
        else if(opcode[i]==7)
        {
            if(zf==1)
            {
                i=opcode[i+1];
                printf("zf=1,jmp to fun%d",i);
            }
            else
            {
                i+=2;
                printf("zf!=1,eip+2\n");
            }
        }
        else if(opcode[i]==0xff)
        {
            printf("结束\n");
            return 0;
        }
        else
        {
            printf("maybe wrong:%d\n",i);
        }
    }
    return 0;
}


```

然后分析打印出来的伪指令，不要被一千多行的指令吓得头皮发麻，仔细看下来发现大量的重复，也就是循环。

```
//part1
R[2]=opcode[3](0)
R[2]+=R[3]       0+=0
R0=input[0](0)
R[1]=R[0](0)
R[2]=opcode[19](50)
R[2]+=R[3]       50+=0
R0=input[50](155)
R[1]+=R[0]       0+=155
R[2]=opcode[35](100)
R[2]+=R[3]       100+=0
R0=input[100](201)
R[1]^=R[0]     155^=201
R[0]=opcode[51](8)
R[2]=R[1](82)
R[1]<<=R[0]        82<<8
R[1]&=0xff00;     20992&=0xff00
R[2]=R[2]>>R[0]     82=82>>8
R[1]+=R[2]       20992+=0
R[0]=R[1](20992)
push R0(20992)
R[0]=opcode[77](1)
R[3]+=R[0]       0+=1
R[0]=R[3](1)
R[1]=opcode[89](40)
R0!=R1-->zf=1
zf=1,eip+2
jmp to fun0

//part2
R[2]=opcode[3](0)
R[2]+=R[3]       0+=1
R0=input[1](0)
R[1]=R[0](0)
R[2]=opcode[19](50)
R[2]+=R[3]       50+=1
R0=input[51](168)
R[1]+=R[0]       0+=168
R[2]=opcode[35](100)
R[2]+=R[3]       100+=1
R0=input[101](169)
R[1]^=R[0]     168^=169
R[0]=opcode[51](8)
R[2]=R[1](1)
R[1]<<=R[0]        1<<8
R[1]&=0xff00;     256&=0xff00
R[2]=R[2]>>R[0]     1=1>>8
R[1]+=R[2]       256+=0
R[0]=R[1](256)
push R0(256)
R[0]=opcode[77](1)
R[3]+=R[0]       1+=1
R[0]=R[3](2)
R[1]=opcode[89](40)
R0!=R1-->zf=1
zf=1,eip+2
jmp to fun0
R[2]=opcode[3](0)
R[2]+=R[3]       0+=2
R0=input[2](0)
R[1]=R[0](0)
R[2]=opcode[19](50)
R[2]+=R[3]       50+=2
R0=input[52](2)
R[1]+=R[0]       0+=2
R[2]=opcode[35](100)
R[2]+=R[3]       100+=2
R0=input[102](189)
R[1]^=R[0]     2^=189
R[0]=opcode[51](8)
R[2]=R[1](191)
R[1]<<=R[0]        191<<8
R[1]&=0xff00;     48896&=0xff00
R[2]=R[2]>>R[0]     191=191>>8
R[1]+=R[2]       48896+=0
R[0]=R[1](48896)
push R0(48896)
R[0]=opcode[77](1)
R[3]+=R[0]       2+=1
R[0]=R[3](3)
R[1]=opcode[89](40)
R0!=R1-->zf=1
zf=1,eip+2
jmp to fun0

………………
//part40
R[2]=opcode[3](0)
R[2]+=R[3]       0+=39
R0=input[39](0)
R[1]=R[0](0)
R[2]=opcode[19](50)
R[2]+=R[3]       50+=39
R0=input[89](87)
R[1]+=R[0]       0+=87
R[2]=opcode[35](100)
R[2]+=R[3]       100+=39
R0=input[139](156)
R[1]^=R[0]     87^=156
R[0]=opcode[51](8)
R[2]=R[1](203)
R[1]<<=R[0]        203<<8
R[1]&=0xff00;     51968&=0xff00
R[2]=R[2]>>R[0]     203=203>>8
R[1]+=R[2]       51968+=0
R[0]=R[1](51968)
push R0(51968)
R[0]=opcode[77](1)
R[3]+=R[0]       39+=1
R[0]=R[3](40)
R[1]=opcode[89](40)
R0=R1-->zf=0
zf!=1,jmp to fun95
R[3]=opcode[98](0)
pop R1(51968)
R[2]=opcode[104](150)
R[2]+=R[3]       150+=0
R0=input[150](18432)
R0!=R1-->zf=1
zf=1,jmp to fun136
结束


```

可以发现，以 jmp to fun0 来分割，可以分割成 40 个差别不大的块，拿第一块来分析。

```
     
int R[6]={0};
   // R[2]=opcode[3];   //
   // R[2]+=R[3];       //
    
    R[0]=input[0];
    R[1]=R[0];       //R[1]=input[0]
    
    //R[2]=opcode[19];    //
    //R[2]+=R[3];         //

    R[0]=input[50];
    R[1]+=R[0];      //R[1]+=input[50]

    //R[2]=opcode[35];    //
    //R[2]+=R[3];        //

    R[0]=input[100];
    R[1]^=R[0];      //R[1]^=input[100]

    R[0]=opcode[51];   
    R[2]=R[1];        
    
    R[1]<<=R[0];       // ((input[0]+input[50])^input[100])<<opcode[51];

    R[1]&=0xff00;     //   R1=(((input[0]+input[50])^input[100])<<opcode[51])&0xff

    R[2]=R[2]>>R[0];   //  R[2]=((input[0]+input[50])^input[100])>>opcpde[51]

    R[1]+=R[2];          //R1+=((input[0]+input[50])^input[100])>>opcpde[51]

    R[0]=R[1];
    //push R0;

    R[0]=opcode[77];
    R[3]+=R[0];       //R[3]+=opcode[77]
    R[0]=R[3];        //R[0]=
    R[1]=opcode[89];    // R1=40  ,第一次R0=1  接着2、3、4…………39 、40
    比较R0和R1    


```

转为 c 语言就是：

```
    R[2]=(input[0]+input[50])^input[100];
    R[1]=(((input[0]+input[50])^input[100])<<opcode[51])&0xff00;
    R[2]=((input[0]+input[50])^input[100])>>opcode[51];                                       
    R[1]=(((input[0]+input[50])^input[100])<<opcode[51])&0xff00+(((input[0]+input[50])^input[100])>>opcode[51]);


```

可以看到，input 经过一番操作被存储在了 R1，随后 push R1 将数据存入了栈中，通过搜索 pop 发现，只有在最后一个块出现了 pop。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XOrcu3icnibpticibKPEuMC8RhSVOK7fv26kwsNq9cMCpfzdvVFk6PmFl0g/640?wx_fmt=other&from=appmsg)

定位到这条指令，他比较了 R0 和 R1，而此时的 R1 存储的是 input[39] 经过加密的值，而 R0 存放的 input[150] 那里就是比较数据了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XMLzqsXt32HfFNQtGy2lhUHkRYSsvlpCbQgQyibmeiaHeLiaeUlCObMQKQ/640?wx_fmt=other&from=appmsg)

可以看到从 input[150] 那里，有连续的四十个有意义的数据，不难判断这就是比较数据了。那么为什么只进行了一次比较呢？因为这一次比较和 cpdata 不相等，input[0]~input[39] 是随便输入的，如果我们把这个 zf 给置为 0，那么接下来就又有一个 pop R1，对比的数据是 input[150+1]。

解密脚本

```
#include<stdio.h>
int main()
{
    unsigned int cpdata[40]={0x00004800, 0x0000F100, 
    0x00004000, 0x00002100, 0x00003501, 0x00006400, 0x00007801, 0x0000F900, 0x00001801, 0x00005200, 
    0x00002500, 0x00005D01, 0x00004700, 0x0000FD00, 0x00006901, 0x00005C00, 0x0000AF01, 0x0000B200, 
    0x0000EC01, 0x00005201, 0x00004F01, 0x00001A01, 0x00005000, 0x00008501, 0x0000CD00, 0x00002300, 
    0x0000F800, 0x00000C00, 0x0000CF00, 0x00003D01, 0x00004501, 0x00008200, 0x0000D201, 0x00002901, 
    0x0000D501, 0x00000601, 0x0000A201, 0x0000DE00, 0x0000A601, 0x0000CA01};
    
    unsigned int input[144] = {
    0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 
    0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 
    0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 
    0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 
    0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 0x00000031, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x0000009B, 0x000000A8, 0x00000002, 0x000000BC, 0x000000AC, 0x0000009C, 
    0x000000CE, 0x000000FA, 0x00000002, 0x000000B9, 0x000000FF, 0x0000003A, 0x00000074, 0x00000048, 
    0x00000019, 0x00000069, 0x000000E8, 0x00000003, 0x000000CB, 0x000000C9, 0x000000FF, 0x000000FC, 
    0x00000080, 0x000000D6, 0x0000008D, 0x000000D7, 0x00000072, 0x00000000, 0x000000A7, 0x0000001D, 
    0x0000003D, 0x00000099, 0x00000088, 0x00000099, 0x000000BF, 0x000000E8, 0x00000096, 0x0000002E, 
    0x0000005D, 0x00000057, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 
    0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x000000C9, 0x000000A9, 0x000000BD, 0x0000008B, 
    0x00000017, 0x000000C2, 0x0000006E, 0x000000F8, 0x000000F5, 0x0000006E, 0x00000063, 0x00000063, 
    0x000000D5, 0x00000046, 0x0000005D, 0x00000016, 0x00000098, 0x00000038, 0x00000030, 0x00000073, 
    0x00000038, 0x000000C1, 0x0000005E, 0x000000ED, 0x000000B0, 0x00000029, 0x0000005A, 0x00000018, 
    0x00000040, 0x000000A7, 0x000000FD, 0x0000000A, 0x0000001E, 0x00000078, 0x0000008B, 0x00000062, 
    0x000000DB, 0x0000000F, 0x0000008F, 0x0000009C, 0x00000000, 0x00000000, 0x00000000, 0x00000000};

    for(int i=0;i<40;i++)
    {
        int data=cpdata[39-i];
        data=((data<<8)&0xff00)+(data>>8);
        data^=input[100+i];
        data-=input[50+i];
        printf("%c",data);
    }
    return 0;

}

//hgame{y0ur_rever5e_sk1ll_i5_very_g0od!!}

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EQxq06s4kjg4SSDDulwT9XPZpAiaJKhTeV2cPawPqAicRApZNic7ial2wibMtt3NicaNsTql6FygfiaFDlA/640?wx_fmt=png&from=appmsg)

  

**看雪 ID：马先越**

https://bbs.kanxue.com/user-home-984774.htm

* 本文为看雪论坛精华文章，由 马先越 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8Ff5NvVAhOPibJvOuu3qznquYSQoia5ic4DUJrd2Y60VNR5Q6FXicicQwV16l7vicmZxefVFXNgiccZMW0xw/640?wx_fmt=jpeg&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550539&idx=1&sn=99d99a504e0b364140e6cfe6c561f0b1&chksm=b18db18186fa389736b29f09c357e9d8c3ecbc37c6411d7664c2876cb7d8d99311810e406a4c&scene=21#wechat_redirect)

**#** **往期推荐**

1、[自定义 Linker 实现分析之路](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550539&idx=2&sn=e3a883e6de9929783e4920b1ae75802d&chksm=b18db18186fa38971cf9a67439421e62a1c3e1dbeb2cdc974c70ab52186fe92738ed759cf003&scene=21#wechat_redirect)

2、[逆向分析 VT 加持的无畏契约纯内核挂](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550427&idx=1&sn=399ad869e9f33b368de123b079ca1ff2&chksm=b18db01186fa390707f03c65e957277ed4eb7d250bbce02130ab2d6324c0c4cd9ab837e01802&scene=21#wechat_redirect)

3、[阿里云 CTF2024 - 暴力 ENOTYOURWORLD 题解](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550386&idx=1&sn=ef197d9dc41313d624d8e297d6cc5f9a&chksm=b18db0f886fa39eedca81d2ebee9e73e689d9db0bfdcb9831d8ebe4a759a5c55f98aff2a771b&scene=21#wechat_redirect)

4、[Hypervisor From Scratch - 基本概念和配置测试环境、进入 VMX 操作](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550275&idx=1&sn=c1b54dc12abbcb627796db92d4f9c2fc&chksm=b18db08986fa399ff036a52bbbe579808ba65111151b31af848628a464efe064e4fbd7c6c1d9&scene=21#wechat_redirect)

5、[V8 漏洞利用之对象伪造漏洞利用模板](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550274&idx=1&sn=83844418c6e1fb22d4d8c2033abdea5e&chksm=b18db08886fa399ee2927fefc6f01c0213e126ef3248a8ecc439231526e9e56e69f937a29a3c&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78txPhfvI9WpuGSCawCN8NJCgzD16Y0IwdUkaI33Qr3DpwRRuvibgRQOg/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多