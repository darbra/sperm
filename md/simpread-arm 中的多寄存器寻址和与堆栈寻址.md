> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cuNhcE9VvlCj1mxqPlIthw)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

最近在分析代码时, 感觉对堆栈和多数据寻址有一些感悟, 故总结一下, 做个记录。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

  

  

  

  

 目录

  

⊙一. 什么是多寄存器寻址和堆栈寻址

⊙二. 多寄存寻址和堆栈寻址各有几种形态

⊙三. 多寄存寻址和堆栈寻址过程中发生了什么

⊙四. 多寄存寻址和堆栈寻址的异同点

 **一. 什么是多寄存器寻址和堆栈寻址**  

多寄存器寻址是指一条指令可以对对各寄存器值的传送, 这种寻址方式中用一条指令最多可传送 16 个通用寄存器的值, 连续的寄存器用 "-" 来连接, 否则用 "," 分隔离。  

```
LDMIA RO! ,{R1 -R4}

```

堆栈是一种数据结构, 堆栈寻址意味着进行出栈入栈的操作。  

```
STMFD R13!,{R0-R4};
LDMFD R13!,{RO-R4};

```

**二. 多寄存寻址和堆栈寻址各有几种形态**  

1 . 多寄存器寻址:

    从存值和取值:  

    ①从内存中取出数据对寄存器赋值 (load much)

    ②把多寄存器的数据放入到内存中 (store much)

    从堆栈指针增加和减少:

    ①进行一个操作后可以增加堆栈指针的值 (increase)

    ②进行一个操作后可以减少堆栈指针的值 (decrease)

    从先后顺序上:

    ①可以在操作 (取值或存值) 前 (before) 进行堆栈指针的 (增加或减少)

    ②可以在操作 (取值或存值) 后 (after) 进行堆栈指针的 (增加或减少)  
多寄存器寻址的排列组合数为: 2*2*2 = 8 种

<table><tbody><tr><td width="103" valign="top">STMDA<br></td><td width="103" valign="top">从寄存器取值到内存后目标指针再减<br></td><td width="103" valign="top">LDMDA<br></td><td width="103" valign="top"><strong>从内存取值到寄存器</strong><strong>后</strong><strong>目标指针减</strong><br></td></tr><tr><td width="103" valign="top">STMIA<br></td><td width="103" valign="top">从寄存器取值到内存后目标指针再增</td><td width="103" valign="top">LDMIA<br></td><td width="103" valign="top"><strong><strong>从内存取值到寄存器后目标指针增</strong></strong><br></td></tr><tr><td width="103" valign="top">STMDB<br></td><td width="103" valign="top"><strong>从寄存器取值到内存</strong><strong>前</strong><strong>目标指针</strong><strong>减</strong><br></td><td width="103" valign="top">LDMDB<br></td><td width="103" valign="top"><strong><strong><strong>从内存取值到寄存器前</strong></strong></strong><strong><strong><strong>目标指针减</strong></strong></strong><br></td></tr><tr><td width="103" valign="top">STMIB<br></td><td width="103" valign="top">从寄存器取值到内存前目标指针增</td><td width="103" valign="top">LDMIB<br></td><td width="103" valign="top"><strong>从内存取值到寄存器前</strong><strong>目标指针增</strong><br></td></tr></tbody></table>

2. 堆栈寻址:  

  从入栈和出栈:  

    ①从寄存器中把数据放入堆栈为入栈 (store much)

    ②把堆栈中的值放入到寄存器中为出栈 (load much)

    从堆栈的状态:

    ①满 (full) 堆栈: 当前栈顶指针指向的堆栈内存是有值的。

    ②空 (empty) 堆栈: 当前栈顶指针指向的堆栈内存是无值的。

    从出栈或入栈方向上:

    ①(出栈或入栈) 的栈顶指针向上增加

    ②(出栈或入栈) 的栈顶指针向下减少

堆栈寻址的排列组合数为: 2*2*2 =8  

<table><tbody><tr><td width="103" valign="top">STMED<br></td><td width="103" valign="top">从寄存器取值到堆栈后目标指针减<br></td><td width="103" valign="top">LDMFA<br></td><td width="103" valign="top"><strong>从堆栈取值到寄存器后</strong><strong>堆栈</strong><strong>指针减</strong><br></td></tr><tr><td width="103" valign="top">STMEA<br></td><td width="103" valign="top">从寄存器取值到堆栈后堆栈指针增</td><td width="103" valign="top">LDMFD<br></td><td width="103" valign="top"><strong><strong>从堆栈取值到寄存器后</strong></strong><strong><strong>堆栈</strong></strong><strong><strong>指针增</strong></strong><br></td></tr><tr><td width="103" valign="top">STMFD<br></td><td width="103" valign="top"><strong>从寄存器取值到堆栈</strong><strong>前</strong><strong>堆栈</strong><strong>指针</strong><strong>减</strong><br></td><td width="103" valign="top">LDMEA<br></td><td width="103" valign="top"><strong><strong><strong>从堆栈取值到寄存器前</strong></strong></strong><strong><strong><strong>堆栈</strong></strong></strong><strong><strong><strong>指针减</strong></strong></strong><br></td></tr><tr><td width="103" valign="top">STMFA<br></td><td width="103" valign="top">从寄存器取值到堆栈前堆栈指针增</td><td width="103" valign="top">LDMED<br></td><td width="103" valign="top"><strong>从堆栈取值到寄存器前</strong><strong>堆栈</strong><strong>指针</strong><strong>增</strong><br></td></tr></tbody></table>

**三. 多寄存寻址和堆栈寻址过程中发生了什么**

多寄存器寻址:

```
STMDA R0!,{R1-R2};
[RO]=R2
R0=RO-4
[RO]=R1
R0=RO-4]
LDMIB RO!,{R1-R2};
RO=RO+4
R1 =[R0]
RO=RO+4
R2=[RO]

```

堆栈寻址:  

```
STMED sp!,{R1-R2};
[SP] = R2
SP = SP-4
[SP] = R1
SP=SP-4
LDMED sp!,{R1-R2};
SP = SP+4
R1 = [SP]
SP=SP+4
R2 = [SP]

```

有没有发现多寄存寻址的 STMDA 和 LDMIB 都是根据最后两个字母进行变换的, 但是堆栈寻址的 LDMED, 最后两个字母明明是 ED 为什么是加呢?  

这里回答下因为多寄存器寻址和堆栈寻址指令意义明显表示的不同, 在出栈入栈中 st 代表的就是入栈了, 而 ld 代表的是出栈了，在这个情况下, 自然以入栈的最后两个字母为主, 而加上 ld 后后面的字母就被反转回来, 就很有意思。  

可能看完这篇文章还有人问, 你怎么知道上方的 STMED sp!,{R1-R2} 时从先把 R2 放到内存中, 然后把 R1 放到内存中呢？为什么不是先把 R1 放到内存中, 然后把 R2 放到内存中呢 (堆栈寻址中同理)？

目前 arm 中是这样的如果在目标指针是减的情况下, 就会从右到左进行处理, 如果目标指针是增加的情况下, 那么就是从左到右处理！

**四. 多寄存寻址和堆栈寻址的异同点**

 相同点:

①都是操作内存和寄存器之间进行读写.  

②进行操作的时候都会进行目标指针的移动, 只不过数据块传送为目标指针, 而堆栈是堆栈指针。

不同点:  

①表示的具体意义不同  

那么有一个问题, 多寄存器寻址能够替代堆栈寻址吗？  

在 " 三. 多寄存寻址和堆栈寻址过程中发生了什么 " 中我们看到了这两组不同的寻址方式, 但是你会发现他们对于数据的处理操作是相同的, 于是我做了个实验, 写了

```
STMDA R13!,{R1-R2};

```

并编译为二进制文件, 然后用 ida 打开反编译下, 你会发现 ida 竟然将其翻译为

```
STMED sp!,{R1-R2};

```

这也太神奇了！

于是我就试了其他的指令发现, 堆栈寻址和多寄存寻址确实存在一一对应的关系, 也就是说可以被转换替代, 具体对应关系如下

<table><tbody><tr><td width="228" valign="top">STMDA<br></td><td width="228" valign="top">STMED<br></td></tr><tr><td width="228" valign="top">STMIA<br></td><td width="228" valign="top">STMEA</td></tr><tr><td width="228" valign="top">STMDB<br></td><td width="228" valign="top">STMFD<br></td></tr><tr><td width="228" valign="top">STMIB<br></td><td width="228" valign="top">STMFA<br></td></tr></tbody></table><table><tbody><tr><td width="228" valign="top">LDMDA<br></td><td width="228" valign="top">LDMFA<br></td></tr><tr><td width="228" valign="top">LDMIA<br></td><td width="228" valign="top">LDMFD<br></td></tr><tr><td width="228" valign="top">LDMDB<br></td><td width="228" valign="top">LDMEA</td></tr><tr><td width="228" valign="top">LDMIB</td><td width="228" valign="top">LDMED<br></td></tr></tbody></table>

我是 BestToYou, 分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误的地方, 恳请大家联系我批评指正。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif)