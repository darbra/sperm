> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PyKtsRV0WJaqm8APpJNOMw)

**观前提示：**  

**本文章仅供学习交流, 切勿用于非法通途, 如有侵犯贵司请及时联系删除**

```
样本：aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMTRWbXVfeHlDN2lGYk9qYjBtOHpYc3c/cHdkPWxpbm4=

```

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclp7Su1076WwNFQDZEHrmzNaoibEAgy4VZnibt8XPNURvCDsKuABgI2EWg/640?wx_fmt=png)

0x1 花指令分析
=========

大姐姐打开`libsoulpower.so`

直接跳转到`0xaa73c`

如果发现这里没被正常识别

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icltD4ibldGiaMeYmA407UoYpjYfaHyChfianj8DD0nCxlxRytrgLCUb2aow/640?wx_fmt=png)

直接在`0xaa73c`处快捷键`P`即可

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclLaiaG2F8zro4LkjTBVRmKaIwicJMJ5tzOQMVaWlj6NqWek40NQYSkLQw/640?wx_fmt=png)

直接 F5 反编译看伪代码

然后就会发现不明代码

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclKGPPcdEOdsrE2sCM9z5iatVPhZdY39VicuvfDicqWaHu2p51Wvyv4JchQ/640?wx_fmt=png)

转到汇编

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclW630MZBnTTalbGRErExgj0iblvEHlWRj5xlO0q33Knak3cvY1Ur5v3w/640?wx_fmt=png)

这里是将`R0`的值作为地址 跳转到该地址处

```
BX              R0


```

那么`R0`从哪里来呢 继续往上看

这里调用了`sub_AECF0`并且将返回值放在了`R0`

```
BL              sub_AECF0


```

所以真实的跳转地址可能就是在`sub_AECF0`中计算得出的

继续往上看还能看到入参的操作

```
LDR             R0, =(off_2C1B64 - 0xAA7A4)
MOV             R1, #0
STR             R1, [SP,#160]
MOV             R1, #1
ADD             R0, PC, R0 ; off_2C1B64


```

这里 IDA 已经帮我们识别好了伪代码 直接看就行

```
sub_AECF0(&off_2C1B64, 1)


```

进`sub_AECF0`看是啥操作

```
int __fastcall sub_AECF0(int a1, int a2)
{
  return *(_DWORD *)(a1 + 4 * a2);
}


```

很简单啊 直接复现一下

0x2C1B64 + 4 * 1 = 0x2C1B68

得到`0x2C1B68`后 跳转过去

```
002C1B68                 DCD sub_AA7B8


```

可以看到`0x2C1B68`对应的就是`sub_AA7B8`

所以 R0 = 0xAA7B8

而 IDA 并不会去执行这些操作 所以达到花指令的目标

0x2 手动去花
========

前面计算了`R0`的真实跳转地址

拿到真实地址后就可以在`0xAA7B4`处手动 Patch 了

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icltzC2ZdwHEal5WJRse3cic8Ms865PGGOTqvWms0Sp6rc0AnHDz72SNug/640?wx_fmt=png)

Patch 完事之后再次`F5`反编译

反编译完发现代码仅仅多了一行

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclQOr6e4fF4FROf3RmbZcxiaGwoRIosic3zwzKZ3vNcUwzn5HZELhfle0w/640?wx_fmt=png)

转到汇编 这里同样出现`BX R0`

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclchXBAlYwWCficvQ4rzTsOKITsS73iaEoibkphUhr4CgqibkA2GNWFAWNTQ/640?wx_fmt=png)

所以这个样本应该是多处插花 那就继续还原

按上述操作 找到俩个传入参数 然后手动计算出真实地址

0x2C1B64 + 4 * 2 = 0x2C1B6C

```
002C1B6C                 DCD sub_AA9F8


```

R0 = 0xAA9F8

继续重复操作 Patch

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclyvDOqmwnLkRR1yIJlob9f3RzNLfYMQ4fJgR2SPsbEvLoZ2y1DPLNCA/640?wx_fmt=png)

代码又多了一点 然后不出意料 又是一处插花

```
LDR             R0, =(off_2C1B64 - 0xAAA08)
MOV             R1, #3
ADD             R0, PC, R0 ; off_2C1B64
BL              sub_AECF0
BX              R0


```

继续重复几次操作后 代码多了这么多 但是还没完全去花

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icldn9HYM4NTcr4hUGZXLsp5XqPfI22uUqtiaMKmuick32gVIGJUsBL4pVw/640?wx_fmt=png)

同时在视图中可以发现 ollvm 后面还不知道会出现啥 继续 Patch

#### 停！！！

怎么可能 量很大 年轻人 你把握不住的 该上脚本了！

0x3 脚本去花
========

首先明确要干啥先

1.  找出所有的插花方法
    
2.  找出所有插花方法的调用处
    
3.  拿到俩个参数
    
4.  计算正确地址
    
5.  Patch
    

前面我们知道 每次计算都会进入`sub_AECF0`

但是我猜测可能不止一个`sub_AECF0`

也许是同操作 但是方法名不一样 这里做个验证

拿到方法 HEX`01 01 90 E7 1E FF 2F E1`

```
01 01 90 E7                 LDR             R0, [R0,R1,LSL#2]
1E FF 2F E1                 BX              LR


```

然后`Binary search`

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icloytruGw7dAGiciaLQJ5LibOlQ0uCowtBMSicnAY7OxcRM1ibWZlhevpriciaQ/640?wx_fmt=png)

搜索得出五个结果

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclN3S48epP1QUsic7DMSqOoh6iaE9QHuABc6Or60NDNMbhn7Abr3vSyCZQ/640?wx_fmt=png)

查看每个方法的交叉引用都有很多处

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icl3mS4thjZ1QL8rjAcRkER9Uv78wPNCsjR1jIzyNpiajMxmY3ibn94tA7A/640?wx_fmt=png)

那可以使用`idautils.CodeRefsTo`遍历交叉引用

但是我担心有可能存在漏识别 所以我采用了从头到尾的遍历方法

```
import capstone
from keystone import *

import flare_emu
import ida_ida
import idautils
import idaapi
import ida_segment
import idc
import ida_bytes

# 初始化架构和模式
CS = capstone.Cs(capstone.CS_ARCH_ARM, capstone.CS_MODE_ARM)
# 设置为详细反汇编模式
CS.detail = True
# 设置反汇编跳过数据
CS.skipdata = True


# 获取.text段的范围
def getAddrRange():
    start = ida_ida.inf_get_min_ea()
    size = ida_ida.inf_get_max_ea() - start
    # 将地址范围限定于text节
    for seg in idautils.Segments():
        seg = idaapi.getseg(seg)
        segName = ida_segment.get_segm_name(seg)
        if segName == ".text":
            start = seg.start_ea
            size = seg.size()
    return start, size


def binSearch(start, end, pattern):
    matches = []
    addr = start
    if end == 0:
        end = idc.BADADDR
    if end != idc.BADADDR:
        end = end + 1
    while True:
        addr = ida_bytes.bin_search(addr, end, bytes.fromhex(pattern), None, idaapi.BIN_SEARCH_FORWARD,
                                    idaapi.BIN_SEARCH_NOCASE)
        if addr == idc.BADADDR:
            break
        else:
            matches.append(addr)
            addr = addr + 1
    return matches

# 获取代码段的起始地址和长度
start, size = getAddrRange()
codebytes = idc.get_bytes(start, size)
sub_matches = binSearch(0, 0, "01 01 90 E7 1E FF 2F E1")
disasmCodes = list(CS.disasm_lite(codebytes, start, size))
for i in range(len(disasmCodes)):
    (address, size, mnemonic, op_str) = disasmCodes[i]
    if mnemonic == 'bl':
        sub_addr = int(op_str.replace('#', ''), 16)
        if sub_addr in sub_matches:
            print(hex(address))


```

效果很不错 找出了很多的调用处

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclr3WwFrUyqDchOEz0Mn3PJoIg7OQNT6FV6sumt6wfdFogwR4lsaicuQQ/640?wx_fmt=png)

找出调用处随便跳转几个看能不能找到规律匹配参数

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclRn8RxAezjic7sj4pkI6iaUUxMP6T1IFrufcus7ygopbJajlxgbNsHtqg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclu51yZqjUS26soIZx6Kk9SJcQ0tU98AhTRQcC03RVYPcphxe2Ac4Wlw/640?wx_fmt=png)

这里发现 参数的位置并不固定 不好做匹配

但是又可以发现 参数位置 调用位置 跳转位置 都是处于同一个 block 中

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclADTUgyD0xcxyLJ2oicfepFIrOwtDiatEZJa0ByXky5wVsxmDdFaYrN4w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclDSomkS4Bv22vOgX2qxlVHTtJ80cpokE6Sc9CvDiaibyjdQoXhsofJW0w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclbIvBr1db7UTVdh9ccuHCQRO8V4TeMpjEmXEWsmXNkVupPvCaCIBhow/640?wx_fmt=png)

知道了这一点 就只需要匹配出相对应的 block 就可以了

```
def find_block(blocks, addr):
    for block in blocks:
        start_ea = block.start_ea
        end_ea = block.end_ea
        if start_ea < addr < end_ea:
            return block
    return None
func = idaapi.get_func(address)
  if func:
    block = find_block(idaapi.FlowChart(func), address)


```

即使匹配出了地址块 也不好匹配出参数 那咋办呢

其实 这里我匹配地址块的原因就是不想再写一堆匹配代码找出`R0`和`R1`

所以我打算交给`unicorn`去做

而我又懒得写很多代码 所以 我又使用了封装好的`flare_emu`

例如 我匹配出了一个 block 我仅需要把 block 的起始地址匹配到方法执行前即可

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclowcUDMqe9N3J547Awgk6JkewARaZEeeiaLib0fMbzuia1KGkK7zKqCnTg/640?wx_fmt=png)

测试看结果

```
import flare_emu
myEH = flare_emu.EmuHelper()
myEH.emulateRange(
  startAddr=0x804DC,
  endAddr=0x804FC
)
Out[31]: <unicorn.unicorn.Uc at 0x169c478b908>

myEH.getRegVal("R0")
Out[32]: 0x2c14d0 (OCSP_SIGNATURE_it + 0x2dc)

myEH.getRegVal("R1")
Out[33]: 1


```

效果完美

后面仅需自己写个计算方法 将参数传入计算即可

```
def calcu_addr(a, b):
    return a + 4 * b


```

得到真实跳转地址后 还得匹配出跳转地址存储的地址

而`idc.print_operand` 刚好能实现这个操作

```
idc.print_operand(0x2c14d4,0)
Out[34]: 'sub_80508'


```

拿到这个真实地址 最后 Patch 就行

不过这里需要注意的是 我们不知道后面第几行是需要 Patch 的`BX`处

但是我们前面就得到了块的起始地址 同样的也可以得到块的结束地址

只需要 (结束地址 - 方法调用地址)/4 然后遍历就能找到`BX`所在位置

但是但是 我觉得不优雅

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclbIvBr1db7UTVdh9ccuHCQRO8V4TeMpjEmXEWsmXNkVupPvCaCIBhow/640?wx_fmt=png)

从这张图 可以知道`BX`一直在块的最后一行 那我何必去遍历呢 直接那最后一行的地址不就行了

一下子就优雅起来了

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4icltRpU3JKAicGGXs5S0z2GV2KVWOtxdZEGCIqAO29Ric3DwJkCLf8RnADw/640?wx_fmt=png)

最后写下 patch

```
code = f"B {hex(b_addr)}"
b_code = generate(code, block.end_ea - 4)
# Patch 1 处理花指令
ida_bytes.patch_bytes(block.end_ea - 4, bytes(b_code))


```

将所有代码串起来 执行测试一下效果

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclRNgVFFic8APWgaib6Mk2E4pTKPFkibzUKxZHUVDyE8xyllHEGdl2j4Ung/640?wx_fmt=png)

执行完后应用更改

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclkz4mhdR20giabbqFiamEqeNv1iclzHNpOdrAUmF3lYubDTAwN6KCBI2ZQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclF2OQTgtQAb6YLUJWuiaJhT0E466GKcHAMS6MF2kNAfwPnLx9g1icic33w/640?wx_fmt=png)

最后重新打开 so

跳转到最开始分析的`0xaa73c`

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclHggqibia1fgNZjrQuKK17Qrb25bddTpR7Rw4yrB5GZzpNvIUE4ou1JAw/640?wx_fmt=png)

成功识别到 N 多行代码了

去花工作完成 处理脚本和对应的 so 文件放在样本中

如果你想学习关于 so 反混淆相关的知识

例如 IDA 脚本开发 字符串加密处理 花指令处理 龙龙的星球里都写得很详细

龙龙老卷王了 内容丰富  很多东西我都是跟着龙龙学的 让我受益匪浅

所以 我吹爆！ 

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsNgowvdHibr4SBZS4yxFr4iclURAm76zqaMUibHcmOmSu2hoKygshaJObJFNu3eUbEDfwtFbnrMs38UA/640?wx_fmt=png)

**感谢各位大佬观看**

**感谢大佬们的文章分享**

 **如有错误 还请海涵**

**共同进步 带带弟弟**

点赞 在看 分享是你对我最大的支持  

逆向 lin 狗