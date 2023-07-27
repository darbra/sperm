> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tmDf8xJ8s6_TjN1xAv37hw)

![](https://mmbiz.qpic.cn/mmbiz_png/gY5PqCV2kYZJ4sKUniaZPr7YRoHTLnbM8ZCNohcGM2hjy5tMJyRd9RicbR9HrMkoJiaGrBrV7JjW09zqHtCHI8q8g/640?&wx_fmt=png)

新的最新版 armv8 的，快手花指令脚本，本文为了学习安全思路，切勿做一些非法事情。

![](https://mmbiz.qpic.cn/mmbiz_png/ib93efMPP0actGbYAbS23TrIXMKA4Hfb3wAjuNLrpnMP1iarC8oTugBZFgEibxVv0uVSvt4F6wwMxoDdCWddEL6sw/640?&wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zJkrRpjYPhNfP5zichlATibarrJQVsf9t5nNRBIicAoVVXVx4sZ6RgF24KTG4ibkcFX4vyLITzJosTaCZYPD0iaDicrQ/640?&wx_fmt=png)

  

⊙1.jnionload 不能 f5

⊙2. 手算花指令跳转

⊙3. 写代码用 idapatch

⊙4. 总结

1.jnionload 不能 f5

libkwsgmain.so  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib1DZX6Ria0WZVhn7UmbpthzKUMY7ibCVlofSjVPCvpYkuxZJsbhKETRzliaCPesQlOicNQvrYhakJ8bAw/640?wx_fmt=png)

2. 手算花指令跳转

```

.text:00000000000462C4                 STP             X0, X1, [SP,#-32]!

.text:00000000000462C8                 STP             X2, X30, [SP,#16]

.text:00000000000462CC                 ADR             X1, dword_462EC

.text:00000000000462D0                 SUBS            X1, X1, #0xC

.text:00000000000462D4                 MOV             X0, X1

.text:00000000000462D8                 ADDS            X0, X0, #0x3C ; '<'

.text:00000000000462DC                 STR             X0, [SP,#24]

.text:00000000000462E0                 LDP             X2, X9, [SP,#16]

.text:00000000000462E4                 LDP             X0, X1, [SP],#0x20

.text:00000000000462E8                 BR              X9

```

导致不能 f5 的原因是在地址 0x462E8，x9 寄存器的值没有被 ida 计算出来

我们看 x9 是从 0x462E0，相当于从 [SP,#24], 又因为在 0x462DC 是 x0 给予的。

所以我们搞清 X0 的来源，就可以算出来，x9 的值了。

```
.text:00000000000462CC                 ADR             X1, dword_462EC
.text:00000000000462D0                 SUBS            X1, X1, #0xC
.text:00000000000462D4                 MOV             X0, X1
.text:00000000000462D8                 ADDS            X0, X0, #0x3C 

```

可以得出 X0 = 0x462EC -0xc + 0x3c = 0x4631c

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib1DZX6Ria0WZVhn7UmbpthzKN6RwYjSw6XuxXQ4OsbedeZjEDX3o7viaibjjXlCkhqJxdIe4mV6kLqJQ/640?wx_fmt=png)

这不是逗我吗，怎么全出来了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib1DZX6Ria0WZVhn7UmbpthzK2KQySbwVTvlTibURXqllVnQaAL8enLY2B1cKBusImohDNcTVrdoy5JQ/640?wx_fmt=png)

但是发现后面也有很多的花

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib1DZX6Ria0WZVhn7UmbpthzKsIKV2KaTdel0Lsa0NSWeWYowOL1fqIfbXsiaW6wbMMj3vYqNsPibdJPg/640?wx_fmt=png)

那我们可以先写个 python 脚本

思路遍历所有的指令，如果当前指令是

```

.text:0000000000041688                 ADR             X1, qword_416A8

.text:000000000004168C                 SUBS            X1, X1, #0xC

.text:0000000000041690                 MOV             X0, X1

.text:0000000000041694                 ADDS            X0, X0, #0x3C ; '<'

```

第一条指令为 x1，且下三条指令为 subs mov adds，这个时候就可以计算，x0 的值，然后获取当前 0x41688  +0x7 *0x4 的位置是不是 br

然后直接去替换

我记得之前 ks 花指令比这个复杂来着，不知道为啥变成这样了

附代码

3. 写代码用 idapatch

```
from capstone.arm64_const import *
import idaapi
import idc
import idautils
from capstone import *
from keystone import *
import ida_bytes
import keypatch
ks = Ks(KS_ARCH_ARM64, KS_MODE_LITTLE_ENDIAN)
md = Cs(CS_ARCH_ARM64, KS_MODE_LITTLE_ENDIAN)
kp = keypatch.Keypatch_Asm()
md.detail = True
def get_all_adr(code="ADR"):
    allAddr = []
    adr_count = 0
    for seg in idautils.Segments():
        start = idc.get_segm_start(seg)
        end = idc.get_segm_end(seg)
        for ea in idautils.Heads(start, end):
            insn = idaapi.insn_t()
            if idaapi.decode_insn(insn, ea):
                if insn.get_canon_mnem() == code:
                    adr_count += 1
                    is_fake_adr(ea)
    return allAddr
def is_fake_adr(ea):
    codelist = ida_bytes.get_bytes(ea, 0x100)
    for insn in md.disasm(codelist[0:4], ea):
        if insn.mnemonic == "adr" and len(insn.operands) > 0 and insn.operands[0].type == ARM64_OP_REG:
            if(insn.operands[0].reg - ARM64_REG_X0 == 1):
                subsFlag  =None
                addsFlag = None
                for subs_insn in md.disasm(codelist[4:8], 0):
                    if subs_insn.mnemonic =="subs":
                        subsFlag = subs_insn
                for adds_insn in md.disasm(codelist[3*4:3 *8], 0):
                    if adds_insn.mnemonic == "adds":
                        addsFlag = adds_insn
                if(subsFlag and addsFlag):
                    print(subsFlag.operands[2].type)
                    if(subsFlag.operands[2].type ==ARM64_OP_IMM and addsFlag.operands[2].type ==ARM64_OP_IMM):
                        x9Addr = insn.operands[1].imm -subsFlag.operands[2].imm + addsFlag.operands[2].imm
                        print(hex(x9Addr),hex(ea))
                        for br_insn in md.disasm(codelist[7 * 4:7 * 8], ea + 4*7):
                            if br_insn.mnemonic == "br":
                                print("addr",hex(ea + 4*7),hex(x9Addr))
                                kp.patch_code(ea + 4*7, "b " + hex(x9Addr), None, None, None,
                                                  None)
get_all_adr()

```

为了验证我们的 patch 是否有问题，从手机上替换了他，app 正常运行，完美  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/p5YHVYUwZib1DZX6Ria0WZVhn7UmbpthzKaFA1Ss4bc2ibZNia9z2GHicW15fnqDUTqZZQwnbvEHwO6mO4d5F5pg/640?wx_fmt=jpeg)

4. 总结

后面继续分析 ks 的一些算法。

如果想获取更多知识，欢迎关注我朋友  

我是 BestToYou, 分享工作或日常学习中关于 Android、iOS 逆向及安全防护的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误或者不足的地方, 恳请大家联系我批评指正。

![](https://mmbiz.qpic.cn/mmbiz_jpg/p5YHVYUwZib1LXXkvk6YVoJQJ90OVr4VIkII5veh8SKAFHGiazq7ic7DWSfeTLl6Usp0GMANicVMvC5U0hKkDDg/640?wx_fmt=jpeg)

扫码加我为好友