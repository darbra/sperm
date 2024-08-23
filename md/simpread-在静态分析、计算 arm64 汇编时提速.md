> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2YQmxIm4dR6HdDrvyLURZA)

![](https://mmbiz.qpic.cn/mmbiz_png/gY5PqCV2kYZJ4sKUniaZPr7YRoHTLnbM8ZCNohcGM2hjy5tMJyRd9RicbR9HrMkoJiaGrBrV7JjW09zqHtCHI8q8g/640?&wx_fmt=png)

在分析汇编代码的时候，我们要计算寄存器的值的来源，举个例子

```
LDRSW X3, [X27,X10,LSL#2] 这个时候想要得到x3的值，需要搞清楚x27与x10

```

```
的来源，然后你在多行代码中进行寻找改变x27、x10的代码，后续还

```

```
遇到x27和x10的来源，找起来就很费事，实在很让人恼火，那么如何办呢,首先找到

```

```
目标操作寄存器，然后找到第一个源和第二个源寄存器，判断我们想要寻找到对应指令的源寄存器，根据

```

```
源寄存器逐级递归遍历，找到所有使用寄存器的index，然后把对应指令打印出来。缺点：目前的代码对于出入栈的时候stp会存在问题，还有对于一些通过出入栈给内存

```

```
空间赋值的也没法处理，但是在局部分析汇编代码时，有一些用，如果其他大佬有好的

```

```
解决方案，欢迎提供~下方实现代码为下抛砖引玉。

```

![](https://mmbiz.qpic.cn/mmbiz_png/ib93efMPP0actGbYAbS23TrIXMKA4Hfb3wAjuNLrpnMP1iarC8oTugBZFgEibxVv0uVSvt4F6wwMxoDdCWddEL6sw/640?&wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib2v1Kmw5V47MCKMNA9trqSLVl7q1bibnc7yT5icStAMCekdndJTMUAcecibkw8oibQ3Uiaa8E6AxTsjkJg/640?wx_fmt=png&from=appmsg)

如果想要分析 w11, 需要找 w14 和 w11, 然后按照箭头就可以找到，然后再找 w14 和 w11 的来源，逐级递归处理；手动是不太现实的，把其实现为代码，代码用了 capstone 和 keystone 去找寻目标寄存器和操作数；代码如下。

```
from capstone import *
from capstone.arm64 import *
from keystone import *
import copy
# 示例汇编代码
assembly_code = """
LSR W11, W12, #0xB
AND W14, W11, #2
LDRSW X3, [X27,X10,LSL#2]
AND W16, W12, #0x10000000
AND W2, W11, #4
BFXIL W14, W12, #31, #1
ORR W14, W14, W2
AND W2, W11, #8
AND W10, W11, #0x10
LSR W11, W16, #0x1A
AND W15, W12, #0x20000000
ORR W2, W14, W2
BFXIL W11, W12, #0x1A, #2
AND W14, W12, #0x40000000
ORR W10, W2, W10
ORR W11, W11, W15,LSR#26
ADD X2, X3, X27
UBFX W9, W12, #0x15, #5
UBFX W8, W12, #0x10, #5
AND W13, W12, #0x80000000
AND W18, W12, #0x4000000
AND W17, W12, #0x8000000
ORR W11, W11, W14,LSR#26
"""
instructions = [line.strip() for line in assembly_code.strip().split('\n') if line.strip()]
# 使用 Keystone 汇编示例代码
ks = Ks(KS_ARCH_ARM64, KS_MODE_LITTLE_ENDIAN)
encoding, count = ks.asm('\n'.join(instructions))
machine_code = bytearray(encoding)
md = Cs(CS_ARCH_ARM64, CS_MODE_LITTLE_ENDIAN)
md.detail = True  # 启用详细信息
allTarSour = []
for instruction in md.disasm(machine_code, 0x4):
    target = None
    src1 = None
    src2 = None
    if instruction.operands:
        target = instruction.reg_name(instruction.operands[0].reg)
        if len(instruction.operands) > 1:
            if instruction.operands[1].type == ARM64_OP_REG:
                src1 = instruction.reg_name(instruction.operands[1].reg)
            elif instruction.operands[1].type == ARM64_OP_MEM:
                src1_base = instruction.reg_name(instruction.operands[1].mem.base)
                src1_index = instruction.reg_name(instruction.operands[1].mem.index)
                src1_offset = instruction.operands[1].mem.disp
                if src1_index:
                    src1 = f"{src1_base}, {src1_index}, LSL #{src1_offset >> src1_index.shift}" if src1_offset else f"{src1_base}, {src1_index}"
                else:
                    src1 = f"{src1_base}, #{src1_offset}" if src1_offset else src1_base
        if len(instruction.operands) > 2:
            if instruction.operands[2].type == ARM64_OP_IMM:
                src2 = f"#{instruction.operands[2].imm}"
            elif instruction.operands[2].type == ARM64_OP_REG:
                src2 = instruction.reg_name(instruction.operands[2].reg)
        src = []
        if src1:
            if ","  in src1:
                src.extend(  [s if ("x" in s or  "w" in s) else None for s in src1.split(",")])
            else:
                src.append(src1)
        if src2:
            if ","  in src2:
                src.extend(  [s if ("x" in s or  "w" in s) else None for s in src2.split(",")])
            else:
                src.extend([src2] if "x" in src2 or "w" in src2 else [])
        allTarSour.append({"target": target,"source":src})
index_list = []
def findall(data):
    index_list.append(len(data)-1)
    end_target = data[-1]['target']
    sources = data[-1]['source']
    for source in  sources:
        temp_len = len(data)-2
        if(temp_len >-1):
            while(temp_len>-1):
                if(data[temp_len].get('target')[1:] == source[1:]):
                    copied_list = copy.deepcopy(data[:temp_len+1])
                    if(len(copied_list)>1):
                        findall(copied_list )
                    else:
                        if(copied_list[0].get('target')[1:] == source[1:]):
                            index_list.append(0)
                    break
                else:
                    temp_len -= 1
findall(allTarSour)
index_list.sort()
assembly_lines = assembly_code.strip().split('\n')
for index in index_list:
    print(assembly_lines[index])

```

结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib2v1Kmw5V47MCKMNA9trqSLe9ia5yL0UPwicR0P9CvmwuJCf48r98cEnicNVzZSHRSFOibaaB4vEjfupg/640?wx_fmt=png&from=appmsg)

学习 js 逆向可以关注我朋友：

我是 BestToYou, 分享工作或日常学习中关于 Android、iOS 逆向及安全防护的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误或者不足的地方, 恳请大家联系我批评指正。