> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n1wHb-RtbbvENRhtyE4naw)

**“** 字节对齐对于计算机和逆向分析有很大的意义**”**  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3Cw6e08Gt75hEH5Gl0niaFicdznia2nk0HzPtDxLEQxwicibDK9D2yKP7ZNUwIYMibQfuIDBicSj0jdeWgQ/640?wx_fmt=gif)

提起字节对齐的印象, 就是大学课程中站在讲台中间的老师所讲述的 “字节对齐是字节按照一定规则在空间上排列，字节 (Byte) 是计算机信息技术用计量存储容量和传输容量的一种计量单位，一个字节等于 8 位二进制数，在 UTF-8 编码中，一个英文字符等于一个字节”, 上面比较官方的介绍了字节对齐的意义, 对于逆向分析过程中还有其他的理解方式。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3Cw6e08Gt75hEH5Gl0niaFicdznia2nk0HzPtDxLEQxwicibDK9D2yKP7ZNUwIYMibQfuIDBicSj0jdeWgQ/640?wx_fmt=gif)

以下操作方式均在 windows xp,vc6.0 编译,32 位的机器上, pack 指令默认是 8。

01

—

什么是字节对齐

声明一个结构体:  

```
  struct MyStruct  
  {
    int a;
    char b;
    char c;
    int d;
  }t1;

```

按照人类的思维: sizeof(MyStruct)==（4+1+1+4）==10

按照人类的思维: a 的 offset 为 0,b 的 offset 为 4,c 的 offset 为 5,d 的 offset 为 6

但是按照编译器的思维：

```
#include "stdafx.h"
#include <stdlib.h>
#define offsetof(TYPE, MEMBER)  ((int) &((TYPE *)0)->MEMBER)
int main(int argc, char* argv[])
{
  struct MyStruct  
  {
    int a;
    char b;
    char c;
    int d;
  }t1;
  t1.a =1;
  t1.b = '2';
  t1.c = '3';
  t1.d = 4;
  printf("%d\n",sizeof(t1)); //12
  printf("a offset%d\n",offsetof(MyStruct,a));//a offset 0
  printf("b offset%d\n",offsetof(MyStruct,b));//b offset 4
  printf("c offset%d\n",offsetof(MyStruct,c));//c offset 5
  printf("d offset%d\n",offsetof(MyStruct,d));//d offset 8
  system("pause");
  return 0;
}

```

有没有发现和编译器的不同, 那什么是字节对齐？  

计算机中内存大小的基本单位是字节（byte），理论上来讲，可以从任意地址访问某种基本数据类型，但是实际上，计算机并非逐字节大小读写内存，而是以 2,4, 或 8 的 倍数的字节块来读写内存，如此一来就会对基本数据类型的合法地址作出一些限制，即它的地址必须是 2，4 或 8 的倍数。那么就要求各种数据类型按照一定的规则在空间上排列，这就是对齐。

02

—  

计算机是如何字节对齐的

字节对齐需要遵从这两条规则:  

(1)  结构体第一个成员的偏移量（offset）为 0，以后每个成员相对于结构体首地址的 offset 都是该成员大小与有效对齐值中较小那个的整数倍，如有需要编译器会在成员之间加上填充字节，也就是说在 windows xp 默认下

offset 必须满足

offset % min(pack(8),sizeof(member type)) == 0

(3)  结构体的总大小为 有效对齐值 的整数倍，如有需要编译器会在最末一个成员之后加上填充字节。

Align 必须满足

结构体的总大小 %min(max(sizeof(a)....sizeof(d)),pack(8)) = 0

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3Cw6e08Gt75hEH5Gl0niaFicpeQPNuRuQdWBn3nNQaGNEgkfaszRS3HOyiazAxBibibN5GWcFgU6ia12UQ/640?wx_fmt=gif)

拿上面的例子来看: 首先在 windows xp 中 pack 指令为 8, 意思是默认 cpu 默认取值为 8，  

```
struct MyStruct  
  {
    int a;
    char b;
    char c;
    int d;
  }t1;

```

1.  首先 a 的 offset 为 0  
    
2.  b 的 offset%min (8,sizeof(b))=0 , 因为 a 的类型为 int 所以 b 的 offset 从 4 记数, 4%1=0, 那么 b 的 offset 为 4
    
3.  c 的 offset%min (8,sizeof(c))=0, 因为 b 的 offset 类型为 char 所以 c 的 offset 从 5 记数, 因为 5%1=0, 那么 b 的 offset 为 5
    
4.  d 的 offset%min (8,sizeof(d))=0, 因为 c 的 offset 类型为 char 所以 d 的 offset 从 6 记数, 6%4！=0,7%4！=0,8%4=0, 所以 d 的 offset 为 8
    

根据上面的公式  x %min(,max(4,1,1,4)8)=0 ，x 必须大于等于 12(因为 d 偏移为 8,d 又为 4 字节)  

x 为 12

03

—  

为什么要字节对齐

内存是以字节为单位存放的, 但是在一些 cpu 处理器, 并不是按照一个字节来存取内存的, 处理器一般取字节的个数为 2,4,8,16,32 为单位来存取内存, 我们将这些存取单位成为内存颗粒度。  

现在我们认为 cpu 是以四个字节作为颗粒度来取字节, 那么该处理器只能够从地址为 4 的倍数开始读取数据。

假如没有内存对齐的规则, 数据可以任意存放到内存之中, 假如一个结构体为

```
struct MyStruct
{
char a;
int b;
}

```

那么 a 的 offset 为 0,b 的 offset 为 1, 如果你是 cpu 你想取 b 该如何取呢?  

首先取 0-3 地址的内存, 然后抛弃 0 地址的内存, 留下 1-3 地址的内存, 同时取 4-7 然后用抛弃 5-7 留 4 地址的内存, 这样 b 才被取出来。

是不是特别复杂了！

如果我们有内存对齐的机制的话: a 的 offset 为 0, b 的 offset 为 4，cpu 仅需要取 4-7 地址的内存可以将 b 取出来。

04

—  

不同的 pack 对齐值有什么影响

上面我们所说的都是 pack 为 8 的情况下, 如果我们人为的可以将 pack 修改为 1 和 2 的情况, 还是用上面的例子  

```
struct MyStruct
{
int a ;
char b;
char c;
int d;
}

```

1.   #pragma pack(1) 时: 
    
    offset:
    
    a =0: 初始地址为 0
    

        b=4: 4%min(1,1)=0

        c=5: 5%min(1,1)=0

        d=6: 6%min(1,1)=0

2. #pragma pack(2) 时: 

        offset:

        a =0: 初始地址为 0 

        b=4: 4%min(1,2)=0

        c=5: 5%min(1,2)=0

        d=6: 6%min(4,2)=0