> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KgozptGhKU39Fk7URiIsUw)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

周末闲暇读了一下 arm_v7 的官方文档, 对 arm 存储立即数有一些感悟, 故记录下来。

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

  

  

  

  

 目录

  

⊙ 1. 什么是立即数？

⊙ 2. 立即数是如何存储在指令中的

⊙ 3. 立即数在逆向分析的一些自我心得

**1. 什么是立即数?**  

举个栗子

```
mov r0，0x1234

```

后面的 0x1234，就是表示的立即数!

概念: 通常把在立即寻址方式指令中给出的数称为立即数, 立即数可以是 8 位、16 位或 32 位, 该数值紧跟在操作码之后。

**2. 立即数是如何存储在指令中的？**

首先这个问题得知道指令存储立即数的结构 (从 arm 的官方文档截图):  

下方截图的其实代表一行指令 (四个字节) 例如: mov r0,#0xff

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib3HGuZqfSNdnYVUGzONQAMbKY6zjlZ9jr050zxvVBsFnmAxsmWmSskibibTRLjdS10iaIVHHy4WmAHQw/640?wx_fmt=png)

  

在 arm 指令中若要赋值一个立即数时, 要占用执行指令的 0-11 内存单元, 其中 a-h, 直接存储数值, 而 8-11 存储的为循环右移 2 的倍数的值也就是: ROR((a-h 存储的值),2*rotation)

arm 官方在设计 arm 对于立即寻址这样存储有什么意义呢？  

可以尽可能使用少内存单元的情况下扩大数的表示范围。  

分析下在立即数寻址中可以放置什么样的立即数？  

我们知道 8 位存储的值的范围为 0-0xff, 也就是说在

```
mov r0,#0xff

```

的时候 rotation=0;  

当存储的立即数超过了这个范围那么应该如何存储呢？

```
mov r0，#0xff0

```

首先 a-e 还是存储的为 1111 1111   

而 rorarion 为循环右移的位数 / 2

  
如果要表示一个数：

移位前  

0000 0000 0000 0000 0000 0000 1111 1111  

移位后  

0000 0000 0000 0000 0000 1111 1111 0000

相当于循环右移了 28 位, 此时 rotation=28 /2 =14=1110  

a-h：1111 1111

任何立即数赋值都合法吗?  

```
mov r0,#0xff1

```

它的二进制位表示为 1111 1111 0001

但是根据指令存储立即数的规则来说, 存储八位后通过循环右移并不可能出现 0xff1 这样的数据存在, 事实证明并不是所有的位数低的立即数存储在 arm 的寄存器中都可以的, 什么是合法的呢, 就是根据存储指令的结构的情况下可以通过移位来表示的才叫合法。

分析过程参照以下表格 (PS: 接触汇编的时候其实官方文档解释的更清晰并且没有歧义)：

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib3HGuZqfSNdnYVUGzONQAMb9XHdTQ7amKegX1EWiaZIfSu4NUMczhKw8n34qguhZqE85Kqpz8ibI0sQ/640?wx_fmt=png)

那么这种情况, 如果我们编写代码的话怎么去写呢？

```
mov r0，0xff0
add r0,r0，#1可以采用这种方式让其加1

```

是不是超级 nice!!

3. 立即数在逆向分析的一些自我心得

上文我们讨论了 mov 指令  

```
mov r0，0xffffff00

```

很明显根据我们上文的分析，通过对 a-h 的 2 的整数倍移位是不可以达到这样的, 但是我们在逆向分析中会看到 ida 反编译成这样, 难道 ida 也错了, 并不是的我参考了 idapro 权威指南中说道, ida 反编译的汇编代码其实也是伪代码, 并不是说显示的汇编代码都是准确的, 只不过让逆向分析人员更加方便看。  

那么如果我们要给 r0 寄存器赋值 0xffffff00 要怎么做呢？  

```
mvn r0，0xff

```

先解释下这个语句的意义: 把操作数 2 按位取反后送入 r0 寄存器中, 也就达到了 mov r0,0xffffff00 的目的！

上面我们说的都是一些可以利用移位或者取反能够解决的, 如果我们碰到没有规律的怎么处理呢？  

然后我在 ida 中做了个实现, 看看 ida 是如何搞定的？  

结论如下；

```
ldr r0, LABEL1
LABEL1:
 .word  0xf6688156

```

就是说用标号跳转找到立即数也可以!  

总结: 多看 arm 官网的文档, 有时候 csdn 找到一些知识本来就是错的, 并且大批人搬运转载错误的知识, 随后我会持续更新 arm 官方文档的读书笔记, 感谢阅读!  

我是 BestToYou, 分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误的地方, 恳请大家联系我批评指正。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif)