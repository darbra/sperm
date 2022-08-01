> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3k7_OiThHLpsMkhqtymeQA)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

这个样本在看雪上看到的, 作者 (hczhong) 比较厉害直接省略了去花指令的思路，写了对其中算法分析的过程, 我也对这个花指令感兴趣, 所以分析复现一下, 充当记录贴了，本系列会连续更新, 直至凑齐 50 个花指令样本为止。

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

  

  

  

  

 目录

  

⊙一. 什么是花指令

⊙二.k 手花指令特征

⊙三. 如何用脚本去除花指令

一. 什么是花指令  

垃圾代码、混淆代码、花指令意指在正常的指令前后加上一些不起作用的代码，它能阻碍 DEBUG 或反汇编器正确的分析代码，其中垃圾代码指那些完全没有用的代码，属于比较低级的水平，反汇编时只要掌握其规律，一般很容易去除，混淆代码就是指将那些有用的代码和无用的代码混在一起，使反汇编时非常难分离。花指令是这两种编码的总称，即可以指垃圾代码，也可以指混淆代码, 英文一般称 Junk code，garbager code。

二.k 手花指令特征

下面的是 JNI_OnLoad 部分的二进制代码, 在正常的没被混淆或者没被加花指令的代码, 在 IDA 中可以对其按下 F5, 即可反编译出 C 代码，在这个案例中确是不可以的。  

```
.text:0007B44C             ; __unwind { // __gxx_personality_v0
.text:0007B44C 03 B5                       PUSH            {R0,R1,LR}
.text:0007B44E 01 48                       LDR             R0, =0x10
.text:0007B450 00 F0 04 F8                 BL              loc_7B45C
.text:0007B450             ; ---------------------------------------------------------------------------
.text:0007B454 10 00 00 00 dword_7B454     DCD 0x10                ; DATA XREF: JNI_OnLoad+2↑r
.text:0007B458             ; ---------------------------------------------------------------------------
.text:0007B458 16 88                       LDRH            R6, [R2]
.text:0007B45A CE C3                       STMIA           R3, {R1-R3,R6,R7}
.text:0007B45C
.text:0007B45C             loc_7B45C                               ; CODE XREF: JNI_OnLoad+4↑j
.text:0007B45C                                                     ; JNI_OnLoad+8E↓j ...
.text:0007B45C 91 F7 14 ED                 BLX             sub_CE88
.text:0007B460 C0 01                       LSLS            R0, R0, #7
.text:0007B462 00 00                       MOVS            R0, R0
.text:0007B464 D8 01                       LSLS            R0, R3, #7
.text:0007B466 00 00                       MOVS            R0, R0
.text:0007B468 E8 01                       LSLS            R0, R5, #7
.text:0007B46A 00 00                       MOVS            R0, R0
.text:0007B46C 00 02                       LSLS            R0, R0, #8
.text:0007B46E 00 00                       MOVS            R0, R0

```

```
.text:0007B44C 03 B5                       PUSH            {R0,R1,LR}
.text:0007B44E 01 48                       LDR             R0, =0x10
.text:0007B450 00 F0 04 F8                 BL              loc_7B45C

```

到这儿一直正常, ida 在进行反汇编的时候依然能正常的识别出来二进制代码所走的流程, 当走到下方的代码的时候 ida 这时反汇编不正常  

```
.text:0007B45C 91 F7 14 ED                 BLX             sub_CE88
.text:0007B460 C0 01                       LSLS            R0, R0, #7
.text:0007B462 00 00                       MOVS            R0, R0
.text:0007B464 D8 01                       LSLS            R0, R3, #7
.text:0007B466 00 00                       MOVS            R0, R0

```

那么我们看下 sub_CE88 做了什么  

```
.text:0000CE88             sub_CE88                                ; CODE XREF: .text:loc_7A00↑p
.text:0000CE88                                                     ; .text:loc_A988↑p ...
.text:0000CE88
.text:0000CE88             arg_8           =  8
.text:0000CE88
.text:0000CE88             ; __unwind {
.text:0000CE88 01 10 CE E3                 BIC             R1, LR, #1
.text:0000CE8C 00 11 91 E7                 LDR             R1, [R1,R0,LSL#2]
.text:0000CE90 0E 10 81 E0                 ADD             R1, R1, LR
.text:0000CE94 08 E0 9D E5                 LDR             LR, [SP,#arg_8]
.text:0000CE98 08 10 8D E5                 STR             R1, [SP,#arg_8]
.text:0000CE9C 03 80 BD E8                 LDMFD           SP!, {R0,R1,PC}

```

BIC R1，LR，#1；R1 = LR and not #1

; 把下一条指令的最低位, 置为 0, 现在也就是说 R1 的值就是执行完 ce88 函数后执行的地址值  

LDR     R1, [R1,R0,LSL#2]；R1 = R1+R0*4

；RO 在上方赋值过了, R1 又增加了 0x10*4

ADD R1，R1，LR  

; 可以翻译为 R1 = LR + LR(最低位置为 0)+ R0*4

LDR LR，[SP,#8]

; 先从堆栈中保存 LR 寄存器的值

STR R1,[SP,#8]  

; 然后用算出的 R1 覆盖掉堆栈的 LR  

LDMFD SP!,{R0,R1,PC}  

; 恢复堆栈，但是现在 PC 已经被刚才计算出的 R1 覆盖了，下一条被执行的指令将要跳转到一个更远的地方  

对上面代码的综上叙述，也就是根据在跳出 CE88 这个函数的时候会根据 R0 的值, 和 LR 的值, 根据码表计算后赋值给 PC 寄存器从而跳转到一个地址, 但是 ida 线性反汇编算不出来这个要跳转的地址。

我们看下 CE88 被调用的地方有多少, 下图所示，根据蛮力虽然能去除掉花指令，但是通用的处理更适合, 往下看  

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1n9ql2WqzI4Slicu1XtVByqUF30zs788XGhgbN4RntCJfwQru5TWZMt2PWE6ZtCVVC8f1ia2EKk9jQ/640?wx_fmt=png)

  

三. 如何用脚本去除花指令

在我参考的文章中，hczhong 已经给了去花的脚本, 但是被看雪的管理员删除了，如下，代码很易读就是算出跳转的绝对地址从而替换掉原始的代码。

```
import idc
from ida_bytes import patch_word
def put_unconditional_branch(source, destination):
    offset = (destination - source - 4) >> 1
    if offset > 2097151 or offset < -2097152:
        raise RuntimeError("Invalid offset")
    if offset > 1023 or offset < -1024:
        instruction1 = 0xf000 | ((offset >> 11) & 0x7ff)
        instruction2 = 0xb800 | (offset & 0x7ff)
        patch_word(source, instruction1)
        patch_word(source + 2, instruction2)
        patch_word(source + 4, 0xbf00)
        patch_word(source + 6, 0xbf00)
    else:
        instruction = 0xe000 | (offset & 0x7ff)
        patch_word(source, instruction)
        patch_word(source + 2, 0xbf00)
        patch_word(source + 4, 0xbf00)
def patch(ea):
    if idc.get_wide_word(ea) == 0xb503: # PUSH {R0,R1,LR}
        ea1 = ea + 2
# ⽬的寄存器是通⽤寄存器 and ⽬的寄存器是r0 and 源寄存器是⽴即数
        if idc.get_operand_type(ea1, 0) == 1 and idc.get_operand_value(ea1, 0) == 0 and idc.get_operand_type(ea1,1) == 2:
            # get_operand_type 获取寄存器的类型
            index = idc.get_wide_dword(idc.get_operand_value(ea1, 1))
            print("index =", hex(index))
            ea1 += 2
            table = None
            if idc.get_operand_type(ea1, 0) == 7:
                print(222)
                # BL loc_80A30
                # 获取tabel表
                table = idc.get_operand_value(ea1, 0) + 4
            if table is None:
                print("Unable to find table")
            else:
                print("table =", hex(table))
                offset = idc.get_wide_dword(table + (index << 2))
                put_unconditional_branch(ea, table + offset)
                # 0x2f150`
if __name__ == '__main__':
    # so中.text段的起始与结束位置
    for i in range(0x7780, 0x8dda0):
        patch(i)

```

参考:

https://bbs.pediy.com/thread-271489.htm

我是 BestToYou, 分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误的地方, 恳请大家联系我批评指正。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif)