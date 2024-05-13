> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9b9gOJbaFcAO8lbb26TgGw)

接着上一篇文章（https://bbs.kanxue.com/thread-281119.htm），我们继续了解一些不一样 vm。

```
一





实战三

```

**d3sky**

这一题需要了解一些 TLS 相关知识。

### TLS 回调函数

TLS（Thread Local Storage，线程局部存储）是各线程的独立的存储空间，TLS 回调函数是指，每当创建或终止进程的线程时会自动调用执行的函数，且调用执行优先于 EP 代码，该特征使它可以作为一种反调试技术使用。

为什么要讲这东西呢，因为一开始分析 main 函数时感觉静态分析比较吃力就想着动态调试分析一下，然后就停在了这里, 这是触发了一个访问异常，sub_4C1050 是由线程回调函数（TlsCallback_0）调用的。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhTe1ZJxbPP9LnibicYn3iazjibpia50SSKO6TGL4UyPvzGiashZpVo8QaiceWA/640?wx_fmt=other&from=appmsg)

指令在尝试读取地址 0x00 处的数据，导致程序不能正常长运行。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhq3Px0qqJESbL44iao4kAicrA8pXQMRKnicOEq9T85UuJNkqk2PnGye63w/640?wx_fmt=other&from=appmsg)

汇编页面长这个样子：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhiajSYToqNBThoyPlv2PXAFlBoibUhSxWaw7KBfFXiacFCqXulTobVsceQ/640?wx_fmt=other&from=appmsg)

圈起来的那里是判断程序是否处于调试状态，如果是调试状态则执行下面的`mov ecx,[eax]`指令。如果不是处于调试状态则走另一个分支，触发一个除零异常，跳转到处理函数之后程序正常运行。对抗方法很简单，直接在 IsDebuggerPresent 结束之后修改 eax 的值，将 1 修改为 0。

踩了个坑大家注意一下，如果修改的是跳转逻辑也就是将 jz 改为 jnz，在调试状态程序也是不能正常运行的，因为 IsDebuggerPresent 的返回值被存储在了 [ebp+var1C] 处，下面的 idiv 除指令除的就是这个位置，试想我们修改了跳转逻辑成功避开了那段会触发访问异常的指令，但是往下运行却没能触发除零异常，自然不能跳转到程序原本的执行流程上去。

### 分析 main 函数

这个 VM 题和前两个又很大的区别，他没有类似 mov、xor、push、pop 这样的操作，只有一条与非指令 opcode[v6] = ~(opcode[v7] & opcode[v8])，而通过与非指令可以实现所有的逻辑运算。出题师傅的博客有详细的介绍 d3sky 出题小记 - 云之君's Blog (https://www.yunzh1jun.com/2023/05/02/d3sky/)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhWZVA89kqL98YmCxS61dgex1Gf144nG7Ee4kKv4NbqDTzRYTC7bYVcA/640?wx_fmt=other&from=appmsg)

乍一看很唬人，跟着动调几轮就能理清逻辑了，opcode[2]、opcode[7]、opcode[19] 是三个标志位。

通过动态调试很容易判断出当 opcode[2] 为 1 时，会在控制台输出，当 opcode[7] 为 1 时会读取输入，在读取完所有的数据之前，opcode[19] 不会为 1，结合 puts("wrong") 可以推测出他是 flag 检测位。

后面这一组着重分析一下。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhmdw7DR4MC8127Zlic008TIdJBOUiaydYsSqM0BA4sX7eMr1gUODpZ5eA/640?wx_fmt=other&from=appmsg)

注释解释的应该很清楚了，RC4_back 是 RC_4 的加 / 解密部分。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhmdw7DR4MC8127Zlic008TIdJBOUiaydYsSqM0BA4sX7eMr1gUODpZ5eA/640?wx_fmt=other&from=appmsg)

opcode[0] 可以看作是虚拟机的 pc 指针，而每次循环都会解密三个单位的指令（这里第一次 RC4 理解为解密更符合逻辑一点，也就是将原始的 opcode 认为是加密处理过的）。取出来之后，pc 指针加三，可以理解为每条指令长度为 3。之后会将 opcode 进行复原。然后执行核心的与非指令。

在调试的过程中通过十六进制窗口那里能得到输入存储的位置。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrh8f8BnicZRniaW8WBicEsLznDEDOGiayNtWkZleTrrB7hhj19uZGic2LGLLw/640?wx_fmt=other&from=appmsg)

查看这个地址，可以看到他在 tls 回调函数里就被初始化了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhBzicJ6ot92ATMCJ8Mia3n5BO9UFoictkxw8lTxXk2s7ichXYLLpPVDV3Bw/640?wx_fmt=other&from=appmsg)

### 翻译

接下来就是翻译了，和前面两个题目一样，我们需要做的是打印下来，而打印又有两种思路，一是将代码直接 copy 过来，编译执行一下，打印出所有的与非逻辑。二是借助 idapython，由于之前都是用的第一种方法，这道题目就使用 idapython 来打印指令了。

想要实现的效果就是

```
opcode[11]=~(opcode[20]&opcode[20])      opcode[11]=~( 1 & 1 )


```

根据汇编代码编写 idapython 脚本。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhfYDuazaJPwfCEaicWrSUJ9L2McHSfhUbm1MjXnoH1hUFAoaOVOEJkKw/640?wx_fmt=other&from=appmsg)

```
import idc
var_8=-8
var_C=-0x0C
v6=idc.get_reg_value('eax')
ebp_=idc.get_reg_value('ebp')
v7=idc.get_wide_word(ebp_+var_8)
v8=idc.get_wide_word(ebp_+var_C)

opcode_base=0xBE4018
value0=idc.get_wide_word(opcode_base+v7*2)       #这里需要注意，v7要乘2，因为opcode是word数组
value1=idc.get_wide_word(opcode_base+v8*2)
print('opcode[%d]=~(opcode[%d]&opcode[%d])'%(v6,v7,v8),'     opcode[%d]=~( %d & %d )'%(v6,value0,value1))


```

我们只关心对输入的判断，所以前面的逻辑不用理会，使用 idapython 在输入结束时提醒。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhmT5cFhbU4Qowzjx5KGladvtiaenrUTZCHDSk6E6iaaP6ico4yWvj2eTHw/640?wx_fmt=other&from=appmsg)

```
import idc
var_24=-0x24
ebp_=idc.get_reg_value('ebp')
num=idc.get_wide_word(ebp_+var_24)
if(num==0x24):
   print('-------------------------------------END-------------------------------------')
   print('-------------------------------------END-------------------------------------')
   print('-------------------------------------END-------------------------------------')
  


```

输入：123456789012345678901234567890123456~

输出如下：

```
-------------------------------------END-------------------------------------
opcode[7]=~(opcode[8]&opcode[8])      opcode[7]=~( 126 & 126 )                  
opcode[2808]=~(opcode[7]&opcode[7])      opcode[2808]=~( 65409 & 65409 )        
opcode[11]=~(opcode[2772]&opcode[2772])      opcode[11]=~( 49 & 49 )      //取反      
opcode[11]=~(opcode[2773]&opcode[11])      opcode[11]=~( 50 & 65486 )
opcode[12]=~(opcode[2773]&opcode[2773])      opcode[12]=~( 50 & 50 )
opcode[12]=~(opcode[2772]&opcode[12])      opcode[12]=~( 49 & 65485 )
opcode[17]=~(opcode[12]&opcode[11])      opcode[17]=~( 65534 & 65533 )
opcode[11]=~(opcode[2774]&opcode[2774])      opcode[11]=~( 51 & 51 )
opcode[11]=~(opcode[2775]&opcode[11])      opcode[11]=~( 52 & 65484 )
opcode[12]=~(opcode[2775]&opcode[2775])      opcode[12]=~( 52 & 52 )
opcode[12]=~(opcode[2774]&opcode[12])      opcode[12]=~( 51 & 65483 )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( 65532 & 65531 )
opcode[11]=~(opcode[17]&opcode[17])      opcode[11]=~( 3 & 3 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 7 & 65532 )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 7 & 7 )
opcode[12]=~(opcode[17]&opcode[12])      opcode[12]=~( 3 & 65528 )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( 65535 & 65531 )
opcode[11]=~(opcode[2809]&opcode[2809])      opcode[11]=~( 36 & 36 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 4 & 65499 )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 4 & 4 )
opcode[12]=~(opcode[2809]&opcode[12])      opcode[12]=~( 36 & 65531 )
opcode[19]=~(opcode[12]&opcode[11])      opcode[19]=~( 65503 & 65535 )


```

观察输出不难发现（49、50、51、52，正是前四个输入 1234 的 ascii），应该是把我们的输入按照四字节一组进行加密的。也能看出, 我们的输入存储在从 opcode[2772] 这个位置。

与非运算十六进制观察更方便一些。

```
-------------------------------------END-------------------------------------
opcode[7]=~(opcode[8]&opcode[8])      opcode[7]=~( 7e & 7e )
opcode[2808]=~(opcode[7]&opcode[7])      opcode[2808]=~( ff81 & ff81 )      //保存最后一位输入至opcode[2808]
     
opcode[11]=~(opcode[2772]&opcode[2772])      opcode[11]=~( 31 & 31 )       //opcode[11]=~(input[0]&input[0])
opcode[11]=~(opcode[2773]&opcode[11])      opcode[11]=~( 32 & ffce )       //opcode[11]=~(input[1]&opcode[11])
opcode[12]=~(opcode[2773]&opcode[2773])      opcode[12]=~( 32 & 32 )       //opcode[12]=~(input[1]&input[1])
opcode[12]=~(opcode[2772]&opcode[12])      opcode[12]=~( 31 & ffcd )       //opcode[12]=~(input[0]&opcode[12])
opcode[17]=~(opcode[12]&opcode[11])      opcode[17]=~( fffe & fffd )       //opcode[17]=~(opcode[12]&opcode[11])

opcode[11]=~(opcode[2774]&opcode[2774])      opcode[11]=~( 33 & 33 )       //opcode[11]=~(input[2]&input[2])
opcode[11]=~(opcode[2775]&opcode[11])      opcode[11]=~( 34 & ffcc )       //opcode[11]=~(input[3]&opcode[11])
opcode[12]=~(opcode[2775]&opcode[2775])      opcode[12]=~( 34 & 34 )       //opcode[12]=~(input[3]&input[3])
opcode[12]=~(opcode[2774]&opcode[12])      opcode[12]=~( 33 & ffcb )       //opcode[12]=~(input[2]&opcode[12])
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( fffc & fffb )       //opcode[18]=~(opcode[12]&opcode[11])

opcode[11]=~(opcode[17]&opcode[17])      opcode[11]=~( 3 & 3 )             //opcode[11]=~(opcode[17]&opcode[17])
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 7 & fffc )          //opcode[11]=~(opcode[18]&opcode[11])
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 7 & 7 )             //opcode[12]=~(opcode[18]&opcode[18])
opcode[12]=~(opcode[17]&opcode[12])      opcode[12]=~( 3 & fff8 )          //opcode[12]=~(opcode[17]&opcode[12])
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffff & fffb )       //opcode[18]=~(opcode[12]&opcode[11])

//注意下面是进行比较
opcode[11]=~(opcode[2809]&opcode[2809])      opcode[11]=~( 24 & 24 )       //opcode[11]=~(cpdata[0]&cpdata[0])
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 4 & ffdb )          //opcode[11]=~(opcode[18]&opcode[11])
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 4 & 4 )             //opcode[12]=~(opcode[18]&opcode[18])
opcode[12]=~(opcode[2809]&opcode[12])      opcode[12]=~( 24 & fffb )       //opcode[12]=~(cpdata[0]&opcode[12])
opcode[19]=~(opcode[12]&opcode[11])      opcode[19]=~( ffdf & ffff )       //opcode[19]=~(opcode[12]&opcode[11])


```

最下面可以推测应该是比较数据，opcode[2809] 距 opcode[2772] 正好相差 37 个单位，是输入的长度。

把与非指令还原成高级指令。

```
opcode[11]=~(input[0]&input[0])                //将input[0]取反 0000 0000 0011 0001--> 1111 1111 1100 1110 
opcode[11]=~(input[1]&opcode[11])              //将input[1]与取反之后的input[0]进行与操作 与操作保留都是1的位，而~input[0]的1是由0变来的
1100 1110 & 0011 0010 -->0000 0010  这一步就是将input[0]为0且input[1]为1的位 置1  0000 0010 然后取反(有点异或的意思)   1111 1101

opcode[12]=~(input[1]&input[1])                //input[1]取反 0011 0010 -- 1100 1101
opcode[12]=~(input[0]&opcode[12])              //将input[0]为1且input[1]为0的位 置1 0000 0001   然后取反 1111 1110
opcode[17]=~(opcode[12]&opcode[11])            //保留都是1的位 1111 1100 然后取反 0000 0011   就是 input[0]^input[1]的结果
最后一条指令就是将input[0]为0且input[1]为1的位置1 同时将input[0]为1且input[1]为0的位置1  合并一下就是相异为1 相同为0

上面六条等价为高级指令
opcode[17]=input[0]^input[1]


```

第一组 idapython 打印的指令就等价为：

```
opcode[17]=input[0]^input[1]
opcode[18]=input[2]^input[3]
opcode[18]=opcode[17]^opcode[18]
opcode[19]=cpdata[0]^opcode[18]


```

结合这里：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhGoaMqvYWUqFwQoJibLnVWSo1JquMRCd2L5MAIoryU2sQFAhz67W3BzQ/640?wx_fmt=other&from=appmsg)

检测时要保证 opcode 为 0，两个数相同异或结果为 0，所以最后一步 opcode[19]=cpdata[0]^opcode[18] 就是检测 cpdata[0] 和 opcode[18] 是否相等。

那么合理的推测就是:

```
cpdata[0]==input[0]^input[1]^input[2]^input[3]
cpdata[1]==input[1]^input[2]^input[3]^input[4]
…………
cpdata[33]==input[33]^input[34]^input[35]^input[36]
?????这里数据不够了 不知道是循环到input[0]那里还是继续往后取数据，先写一个解出前面的试试
cpdada[36]=input[]


```

z3 脚本

```
from z3 import *

# 创建符号变量
input = [BitVec('input_%d' % i, 16) for i in range(37)]

# 已知的结果
cpdata = [0x0024, 0x000B, 0x006D, 0x000F, 0x0003, 0x0032, 0x0042, 0x001D, 
    0x002B, 0x0043, 0x0078, 0x0043, 0x0073, 0x0030, 0x002B, 0x004E, 
    0x0063, 0x0048, 0x0077, 0x002E, 0x0032, 0x0039, 0x001A, 0x0012, 
    0x0071, 0x007A, 0x0042, 0x0017, 0x0045, 0x0072, 0x0056, 0x000C, 
    0x005C, 0x004A, 0x0062, 0x0053, 0x0033]

# 添加异或约束
constraints = []
for i in range(34):
    constraints.append(cpdata[i] == (input[i] ^ input[i+1] ^ input[i+2] ^ input[i+3]))

# 创建解决器
solver = Solver()

# 添加约束
solver.add(constraints)

# 求解
if solver.check() == sat:
    model = solver.model()
    solution = [model[input[i]].as_long() for i in range(37)]
    print("Solution found:")
    print(solution)
else:
    print("No solution found.")

#[80, 65, 50, 7, 127, 39, 80, 11, 78, 87, 15, 61, 38, 108, 52, 13, 101, 119, 81, 32, 78, 72, 8, 60, 69, 107, 0, 95, 78, 83, 85, 13, 121, 119, 15, 93, 111]


```

明显是错误, 有可能是推测错了，经过验证 z3 打印的结果是满足 cpdata[0]==input[0]^input[1]^input[2]^input[3] 的，那我们将其 patch 进去，就能得到第二组比较的方法，将得到的全部数据 patch 进去就能打印到与 cpdata[34] 的比较。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhdoUc5w8KypI3XrJBCgpBicr9loSSxAYM2jpAzK9YPKCkZabKIrJIt7A/640?wx_fmt=other&from=appmsg)

```
opcode[7]=~(opcode[8]&opcode[8])      opcode[7]=~( 7e & 7e )
opcode[2808]=~(opcode[7]&opcode[7])      opcode[2808]=~( ff81 & ff81 )
opcode[11]=~(opcode[2772]&opcode[2772])      opcode[11]=~( 50 & 50 )        //input[0]
opcode[11]=~(opcode[2773]&opcode[11])      opcode[11]=~( 41 & ffaf )        //input[1]
opcode[12]=~(opcode[2773]&opcode[2773])      opcode[12]=~( 41 & 41 )
opcode[12]=~(opcode[2772]&opcode[12])      opcode[12]=~( 50 & ffbe )        
opcode[17]=~(opcode[12]&opcode[11])      opcode[17]=~( ffef & fffe )
opcode[11]=~(opcode[2774]&opcode[2774])      opcode[11]=~( 32 & 32 )        //input[2]
opcode[11]=~(opcode[2775]&opcode[11])      opcode[11]=~( 7 & ffcd )         //input[3]
opcode[12]=~(opcode[2775]&opcode[2775])      opcode[12]=~( 7 & 7 )
opcode[12]=~(opcode[2774]&opcode[12])      opcode[12]=~( 32 & fff8 )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffcf & fffa )
opcode[11]=~(opcode[17]&opcode[17])      opcode[11]=~( 11 & 11 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 35 & ffee )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 35 & 35 )
opcode[12]=~(opcode[17]&opcode[12])      opcode[12]=~( 11 & ffca )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffff & ffdb )
opcode[11]=~(opcode[2809]&opcode[2809])      opcode[11]=~( 24 & 24 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 24 & ffdb )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 24 & 24 )
opcode[12]=~(opcode[2809]&opcode[12])      opcode[12]=~( 24 & ffdb )
opcode[19]=~(opcode[12]&opcode[11])      opcode[19]=~( ffff & ffff )

opcode[11]=~(opcode[2773]&opcode[2773])      opcode[11]=~( 41 & 41 )          //input[1]
opcode[11]=~(opcode[2774]&opcode[11])      opcode[11]=~( 32 & ffbe )          //input[2]
opcode[12]=~(opcode[2774]&opcode[2774])      opcode[12]=~( 32 & 32 )  
opcode[12]=~(opcode[2773]&opcode[12])      opcode[12]=~( 41 & ffcd )
opcode[17]=~(opcode[12]&opcode[11])      opcode[17]=~( ffbe & ffcd )
opcode[11]=~(opcode[2775]&opcode[2775])      opcode[11]=~( 7 & 7 )            //input[3]
opcode[11]=~(opcode[2776]&opcode[11])      opcode[11]=~( 35 & fff8 )          //input[4]
opcode[12]=~(opcode[2776]&opcode[2776])      opcode[12]=~( 35 & 35 )
opcode[12]=~(opcode[2775]&opcode[12])      opcode[12]=~( 7 & ffca )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( fffd & ffcf )
opcode[11]=~(opcode[17]&opcode[17])      opcode[11]=~( 73 & 73 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 32 & ff8c )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 32 & 32 )
opcode[12]=~(opcode[17]&opcode[12])      opcode[12]=~( 73 & ffcd )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffbe & ffff )
opcode[11]=~(opcode[2810]&opcode[2810])      opcode[11]=~( b & b )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 41 & fff4 )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 41 & 41 )
opcode[12]=~(opcode[2810]&opcode[12])      opcode[12]=~( b & ffbe )
opcode[19]=~(opcode[12]&opcode[11])      opcode[19]=~( fff5 & ffbf )

…………

opcode[11]=~(opcode[2806]&opcode[2806])      opcode[11]=~( 60 & 60 )          //input[34]
opcode[11]=~(opcode[2807]&opcode[11])      opcode[11]=~( 43 & ff9f )            //input[35]
opcode[12]=~(opcode[2807]&opcode[2807])      opcode[12]=~( 43 & 43 )     
opcode[12]=~(opcode[2806]&opcode[12])      opcode[12]=~( 60 & ffbc )
opcode[17]=~(opcode[12]&opcode[11])      opcode[17]=~( ffdf & fffc )
opcode[11]=~(opcode[2808]&opcode[2808])      opcode[11]=~( 7e & 7e )          //input[36]
opcode[11]=~(opcode[2772]&opcode[11])      opcode[11]=~( 41 & ff81 )           //input[0]
opcode[12]=~(opcode[2772]&opcode[2772])      opcode[12]=~( 41 & 41 )
opcode[12]=~(opcode[2808]&opcode[12])      opcode[12]=~( 7e & ffbe )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffc1 & fffe )
opcode[11]=~(opcode[17]&opcode[17])      opcode[11]=~( 23 & 23 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 3f & ffdc )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 3f & 3f )
opcode[12]=~(opcode[17]&opcode[12])      opcode[12]=~( 23 & ffc0 )
opcode[18]=~(opcode[12]&opcode[11])      opcode[18]=~( ffff & ffe3 )
opcode[11]=~(opcode[2843]&opcode[2843])      opcode[11]=~( 62 & 62 )
opcode[11]=~(opcode[18]&opcode[11])      opcode[11]=~( 1c & ff9d )
opcode[12]=~(opcode[18]&opcode[18])      opcode[12]=~( 1c & 1c )
opcode[12]=~(opcode[2843]&opcode[12])      opcode[12]=~( 62 & ffe3 )
opcode[19]=~(opcode[12]&opcode[11])      opcode[19]=~( ff9d & ffe3 )


```

不出所料，第一组 opcode[19] 得到的值是 0，又继续往下执行了一组还是正确的，直接拉到最后可以看到执行的逻辑是：

```
cpdata[34]==input[34]^input[35]^input[36]^input[0]


```

循环执行了，更改一下 z3 脚本即可。

```
from z3 import *

# 创建符号变量
input = [BitVec('input_%d' % i, 8) for i in range(37)]

# 已知的结果
cpdata = [0x0024, 0x000B, 0x006D, 0x000F, 0x0003, 0x0032, 0x0042, 0x001D, 
    0x002B, 0x0043, 0x0078, 0x0043, 0x0073, 0x0030, 0x002B, 0x004E, 
    0x0063, 0x0048, 0x0077, 0x002E, 0x0032, 0x0039, 0x001A, 0x0012, 
    0x0071, 0x007A, 0x0042, 0x0017, 0x0045, 0x0072, 0x0056, 0x000C, 
    0x005C, 0x004A, 0x0062, 0x0053, 0x0033]

# 添加异或约束
constraints = []
for i in range(34):
    constraints.append(cpdata[i] == (input[i] ^ input[i+1] ^ input[i+2] ^ input[i+3]))

# 创建解决器
solver = Solver()

# 添加约束
solver.add(constraints)
solver.add(cpdata[34] == (input[34] ^ input[35] ^ input[36] ^ input[0]))
solver.add(cpdata[35] == (input[35] ^ input[36] ^ input[0] ^ input[1]))
solver.add(cpdata[36] == (input[36] ^ input[0] ^ input[1] ^ input[2]))
solver.add(input[36]==126)    #题目中对最后一位的限制

# 求解
if solver.check() == sat:
    model = solver.model()
    solution = [model[input[i]].as_long() for i in range(37)]
    print("Solution found:")
    print(solution)
else:
    print("No solution found.")

#[65, 95, 83, 105, 110, 57, 49, 101, 95, 73, 110, 83, 55, 114, 85, 99, 116, 105, 48, 78, 95, 86, 105, 82, 84, 117, 97, 49, 95, 77, 52, 99, 104, 105, 110, 51, 126]
#A_Sin91e_InS7rUcti0N_ViRTua1_M4chin3~


```

终于结束了！

这道题似乎完全不同于前面的那种 VM 类型的题目，他没有结构体，没有 vm_init、没有 vm_start 等等，但是思想上是一致的，都是用 vm 提供的指令来还原程序逻辑（只是在这道题目中只有一种指令），都很好的抵抗了静态分析，而且仔细回味一下，opcode[11]、opcode[12]、opcode[17]、opcode[18]，其实就是虚拟机的寄存器，也能找到对应的抽象的 init、start 等。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhxGXAicibpW5H2leAGqpEyKicYAhbf55g09myFLKzRnytdNZckna9FIuVg/640?wx_fmt=other&from=appmsg)

**参考链接：**

出题师傅的博客 d3sky 出题小记 - 云之君's Blog (_https://www.yunzh1jun.com/2023/05/02/d3sky/_)

pz 师傅的视频讲解【D^3CTF2023】D3sky & D3rc4_哔哩哔哩_bilibili（_https://www.bilibili.com/video/BV13P41117B6/?spm_id_from=333.337.search-card.all.click&vd_source=edc820e8f9bd6b2ea43cb8499151dea3_）

```
二





初识VMP

```

通过这三个题目我们对虚拟机保护技术有了一定的了解，这项技术运用到实际的生产工作中就有了 VMP（VMProtect）。

### 什么是 VMProtect

VMProtect 是一个软件保护程序，应用于软件保护和加固领域，VMProtect 通过使应用程序的代码和逻辑变得复杂来对抗逆向，主要的保护机制包括: **虚拟化**、**变异**、**组合保护**。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhuQQibmpteSPiavjkAhZCT8Pne1iaPzQrOVrJniacwogyEbUJ78w8bMMPkg/640?wx_fmt=other&from=appmsg)

我们主要了解一下虚拟化，VMProtect 首先会将受保护的代码转换为等价的虚拟代码片段，然后交由虚拟机执行，该虚拟机是 VMProtect 嵌入到受保护程序中的，因此受保护的程序不需要第三方库活模块即可运行。更为夸张的是，VMProtect 允许使用多个不同的虚拟机来保护同一个应用程序的不同代码片段，这大大增加了破解难度。

### 优缺点

**优点：**

◆保护程度高，极难破解（没有绝对安全的程序，但如果破解程序所需的成本大于程序本身，程序自然就安全了）。

◆保护后，虚拟机和新命令集将内置到受保护的应用程序中，并且不需要任何其他库和模块即可工作。

◆支持多种语言，以及主流操作系统：Windows、macOS、Linux。

**缺点：**

◆执行效率低（参考上面的题目，一条 printf 语句就对应十几条 vm 指令）。

◆过于消耗系统资源。

之后有时间了应该会深入了解一下 VMP，尝试对抗低版本的 vmp 保护，跟着大佬们分析分析源码。（本来这篇文章的后半部分是尝试逆向一个我自己写的受 VMProtect 保护的程序的，可是一个 54kb 的程序直接膨胀到了 13mb，一时间有些恍惚）

**参考链接：**

VMProtect Software » VMProtect » Docs (_https://vmpsoft.com/vmprotect/user-manual_)

【镖客熊猫江】从破解一个 vmp 壳的程序到 vmp 原理详解_哔哩哔哩_bilibili

（_https://www.bilibili.com/video/BV1So4y1H7o4/?spm_id_from=333.337.search-card.all.click&vd_source=edc820e8f9bd6b2ea43cb8499151dea3_）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GI89YLObhqC8xGnamkzMrhI8x5AZeJgqZV8WDDMQE2BQv0dF3pFw4A5nB3d2uWRsVvHkSdwoMq5g/640?wx_fmt=png&from=appmsg)

  

**看雪 ID：马先越**

https://bbs.kanxue.com/user-home-984774.htm

* 本文为看雪论坛精华文章，由 马先越 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8Ey0Z2fLQtGQHrnic5NDLhqHOiawor0NGNYgTypElOIhzbDjjrA4dhmUjZ7EPkia6iblPRianC9Q5OCshw/640?wx_fmt=png&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458549821&idx=2&sn=2a8052c2a32074693dffb319fb7edb54&chksm=b18d4eb786fac7a1c0f3f851a5c344eb0aa0fada20a3bf3d48f9f068d4f9383b2b95fe8f1b4c&scene=21#wechat_redirect)

**#** **往期推荐**

1、[怎么让 IDA 的 F5 支持一种新指令集？](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458552429&idx=1&sn=415d0ec99eeb9224171df3b40d32b0ae&chksm=b18db8e786fa31f16771aa01b1931942b36c595ca6eaedffb73b8bb2ebf021cfc2e6a459db10&scene=21#wechat_redirect)

2、[2024 腾讯游戏安全大赛 - 安卓赛道决赛 VM 分析与还原](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458552129&idx=1&sn=8f962c31693c83cc5088ca426d58efc0&chksm=b18db7cb86fa3edd9008582ce6c530ffb6c97254263e48ae9ac74eab415284286f07c5a9eaff&scene=21#wechat_redirect)

3、[Windows 主机入侵检测与防御内核技术深入解析](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458552007&idx=2&sn=65017982b17cb3930d3276412ecc381a&chksm=b18db64d86fa3f5b47e5fc5724be0a749a1018959bc52e3f332ec1e3a7a428a663f1fc22d425&scene=21#wechat_redirect)

4、[系统总结 ARM 基础](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551831&idx=1&sn=88052805c8c43e2f05a4c86e85aeedea&chksm=b18db69d86fa3f8b34c32693f09b0cbc9bef7ff6b253ef9e60aab30464fb98f9de51416350ff&scene=21#wechat_redirect)

5、[AliyunCTF 2024 - BadApple](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551651&idx=1&sn=91b96cb3efd841999f8381d2f86e2247&chksm=b18db5e986fa3cff8edf467098f6e0dd1bdc8ec3ebb6365ee2ca759e93cd9c509f32c0eb0877&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458550274&idx=1&sn=83844418c6e1fb22d4d8c2033abdea5e&chksm=b18db08886fa399ee2927fefc6f01c0213e126ef3248a8ecc439231526e9e56e69f937a29a3c&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78txPhfvI9WpuGSCawCN8NJCgzD16Y0IwdUkaI33Qr3DpwRRuvibgRQOg/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多