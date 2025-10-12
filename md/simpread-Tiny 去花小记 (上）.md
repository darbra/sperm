> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/W4c8ftPnNrzuT6seaHh5eg)

失踪人口回归

最近都在硬钢某书，也是学习了很多大厂的加密混淆设备指纹的方法，没去花指令的话算法可能直接 trace 能看但是逆设备指纹就很痛苦了，去了花指令能看清楚很多逻辑。

还是师从龙哥，在龙哥星球学习了去花指令的基础和 IDA 脚本就上手干这个 tiny.so 了，龙哥 yyds。

app 版本 884090

环境

安卓 14 + frida16.5.2 魔改（用 Florida 应该就行 能过 msao 就可以不用写脚本 patch 了）

IDA9.2(IDA9.0 应该也行)

直接 IDA 打开 tiny.so 的 JNI_OnLoad 看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUibsQXqQNmvzo52nEZZpq1gicEC0txud31meTHkh6pxXkEiatXwLIFOicww/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

看不了一点转汇编看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydU8vwTWtZT0TibRONibL9KoIflUx8Hlas0TtZOY938jeC0T5PDiawHlw87Q/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)

IDA 不知道这个 br x9 的地址是哪里 所以解析成这样了

那么找一下 x9 是怎么计算出来的

根据 x9 的来源往上追溯即可找到如何运算出 x9

```
.text:0000000000199F14 77 24 00 D0                 ADRP            X23, #off_6278A0@PAGE            ;debug 这里ida优化了 按h可以看原来的反汇编代码
.text:0000000000199F18 78 1E 8B 92                 MOV             X24, #0xFFFFFFFFFFFFA70C         ;debug x24 = 0xFFFFFFFFFFFFA70C
.text:0000000000199F1C AB 03 1F F8                 STUR            X11, [X29,#var_10]
.text:0000000000199F20 69 A6 01 A9                 STP             X9, X9, [X19,#0x130+var_118]
.text:0000000000199F24 6B 0E 40 F9                 LDR             X11, [X19,#0x130+var_118]
.text:0000000000199F28 EB 52 44 F9                 LDR             X11, [X23,#off_6278A0@PAGEOFF]   ;debug x11 = [0x6278A0] 按h可以看原来的反汇编代码 就是读 0x6278A0这个偏移的值
                                                                                                    ;debug x11 = 0x3F7FD34
.text:0000000000199F2C 38 84 BF F2                 MOVK            X24, #0xFC21,LSL#16              ;0xFC21 << 16 = 0xfc210000 X24 = 0xfffffffffc21a70c
.text:0000000000199F30 08 7C 92 52                 MOV             W8, #0x93E0
.text:0000000000199F34 0A 00 00 B0                 ADRP            X10, #sub_19A60C@PAGE
.text:0000000000199F38 08 78 A0 72                 MOVK            W8, #0x3C0,LSL#16
.text:0000000000199F3C 4A 31 18 91                 ADD             X10, X10, #sub_19A60C@PAGEOFF
.text:0000000000199F40 1A 00 00 B0                 ADRP            X26, #sub_19A900@PAGE
.text:0000000000199F44 69 01 18 8B                 ADD             X9, X11, X24 ;debug x9 = x11 + x24 = 0x3F7FD34 + 0xfffffffffc21a70c = 0x19A440
.text:0000000000199F48 7B 24 00 D0                 ADRP            X27, #0x627000
.text:0000000000199F4C 7C 24 00 D0                 ADRP            X28, #0x627000
.text:0000000000199F50 76 24 00 D0                 ADRP            X22, #0x627000
.text:0000000000199F54 75 24 00 D0                 ADRP            X21, #0x627000
.text:0000000000199F58 74 24 00 D0                 ADRP            X20, #0x627000
.text:0000000000199F5C 5A 03 24 91                 ADD             X26, X26, #sub_19A900@PAGEOFF
.text:0000000000199F60 79 24 00 D0                 ADRP            X25, #0x627000
.text:0000000000199F64 48 01 08 8B                 ADD             X8, X10, X8
.text:0000000000199F68 20 01 1F D6                 BR              X9           ;debug b 0x19A440

```

这里可以看到跳转的地址是 0x19A440

这里我们就把汇编指令用 Keypatch 插件 patch 成 B 0x19A440

ida 反编译就成了这样了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydU2jGPMuJiafbSzELVrtkicBVJBCIMHjDtXckKQ4iaWz6GVGKHZwywYiav5A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

我们就成功手动完成了一次去花指令

那么接下来

![](https://mmbiz.qpic.cn/mmbiz_jpg/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUCt0D2ep6qmibes99BaM0ibZiaLIB8Z9ibAiaujTYtbJjBLgD1mSiaibZlkfIQ/640?wx_fmt=jpeg&from=appmsg&watermark=1#imgIndex=3)

继续到这个 0x19A440 地址看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUoyYsiaIqAOrGETv04X6pHRANSLcbMYP0qbYbT0sGvE1fXCux5ibbMI7g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)

又是 x8 的运算 这样手动算太麻烦了 我们知道一个原理就行 接下来我们可以直接用 frida hook 或者 unidbg hook 去直接看此时的地址是什么不自己手动算了

（unidbg 跑一个 so 的 jni_onload 应该都会吧）

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUY6SOaPQG74Sj3XHia497BrDr7n4z0Rzt6MZIRHzSicOEOHD4DNGicCx8g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5)

此时 x8=0x1219a5a4 减去基址就是 0x19a5a4（实际上这里 br x8 会跳不同的地址 unidbg 这里只是其中一个分支）

patch 这个 br x8 为 b 0x19a5a4

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUeKzUpCZCU6ltWVic4zrC2zvBVqZbxibSpTv7DApqMMB7g6mAM7h4Dodw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)

继续来到 0x19a5a4 这个地址

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUEJwV2V49AQ4njyzk7B2c3SZqLdbDBayu3WicoOhTB1DnLktcSmbShRg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=7)

这里十进制数字不好看给他默认转为 16 进制 设置 Hex-Rays 的 Default radix 为 16

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xh6PVKL2kGz6yGqaN698psSvibAKl0NL59GV9ibt6icBE7g4jMdW01TgwLA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=8)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhUXNOg7mibGjicWMPAFfZZbLy3MjEWm0j0TTDmzAeib6Z7ulial3ibFStrDw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=9)

发现反编译还是一坨 继续观察汇编指令 

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUr7ZibIpYEcicHbfsIzpicuPbvPyh7h2jgmM7wrezKChS4qzBLwlmEQ9Bg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=10)

这里可以发现多了一个 blr 指令 这个地址不知道是什么 用 undibg 模拟跑一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydU3vX3nWHvG5ia7GdFwsMOp70h1rROtT3RRyialCFQSeHjicJlnd7Fxe7vQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11)

这个 x8 明显不是 tiny.so 的地址 有经验的同学就会知道这个其实 unidbg 的 jni 函数地址

怎么知道是哪个 jni 函数呢

unidbg-android/src/main/java/com/github/unidbg/linux/android/dvm/DalvikVM64.java

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydUmublHDVibDxov2Wrmq8CgB6EoicQO62EiaibKHexTCdMnufcoicZ6GUDT0A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=12)

这个代码里面就有注册 jni 函数，我们在他 impl.setPointer 方法里面加一个日志不就知道了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpS2IiaDPSxjWqZxpdC6o1ydU1t9G1NibbOkdzQwuicE4ib5hmCUXZL7pxaL27VBn4nF39k78yb5sb6ibnQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=13)

对应 impl.setPointer(0x6d8, _GetJavaVM);

那就是调用的 GetJavaVM 

手动标注一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xhm0iaic17FLAhZdpFMdKgJuia0KDOiaSqFyj6JLTReYmkQicTkLWGjkxjWicw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=14)

但是这里对 v2 - 0x38 这个整体进行转 JNIEnv * 类型不好转就先不管了（会的 dddd）

然后看下一个 br x9 实际地址是 0x19a160 patch 一下 重新反汇编

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhnNbwMB0PMBRKfZXZhvSOicSLNWicUPQgiaibFjE7m8fMCepaVN4PE8ib3sg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=15)

然后这里有个 v4 = (char *)off_6278E0 + 0x22EF2790;

其实就是读 6278E0 这个地址的值 +0x22EF2790 计算出来新的一个地址，这个 tiny 有很多变量都被 ida 识别成这个了 也算是一种混淆，它实际上就是指向 0x670780 这个地址的变量

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XheCOibUOjChVYtCgD4ww7fhVZC5cvpVgmer9Pibh3kt8n8icdtvDicCQzQQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=16)

再次修改下这个函数

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhuwFcBGwVhEpzl5vZwL7DJwbRsgOCXTibPziauCFV7J59qP3fmMOyCDyg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=17)

也可以发现他很多是通过 x19 寄存器给下一个函数传值的

继续 unidbg 往下看他下一步跳的地址然后 patch

19A16C                 B               sub_19A880

然后就到了这个函数 我们先把下一步要走的地址 patch 上 ida 会给我们优化反编译函数更好看逻辑

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhISZWlWAwcDVRXDznCrpsfU4HgWo61Mq5zGWgxpG0JHUZK5Gn6w5CJg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=18)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhX2iceOcqpzVrpCKHWEJmQOQcV0dhpLGib20A9MYHiaAE4vmXrcJ1KMwBQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=19)

实际上就是对 x19+0x38(就是上面标注的 0x670780 这个地址变量) 复制到 x19+0x40

继续往下 (如果觉得 unidbg 断点一个个看下一个地址很累 可以直接 trace 看地址也行)

跳了几次后就到了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhmOYZcIhqsicsh4WPudAaia0OPse8FbKtl0y9ibQic0pyR6qQw1zO47DOFw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=20)

atomic_load 函数的目的是执行一个**原子读取**操作。

**原子操作 (Atomic Operation)** 意味着整个操作（在这个案例中是读取内存中的一个变量）是**不可中断**的。无论有多少个处理器核心或线程同时尝试读写同一个内存位置，该操作都会像一个**单一的、瞬间的步骤**完成。

这里就是读取地址上的值 这地址都没赋值所以是 0

v1 = [v0+0x40] = [0x670780] = 0

在把这个值存到 x19 + 0x4C 上

继续来到这个地方

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xhy7guhT8zDC9iaW1BzogSibib0R7RLDenlkuDNXu5jAecqGkykcx5IFZpw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=21)

这里如果把下面的 BR   X9 patch 成跳转地址 这里的逻辑就消失了 其实这里就是计算下一个跳转地址根据不同的条件 所以这也是 patch 这种花指令的难点 不是所有的 br 寄存器指令都是跳转固定地址的  所以没法直接静态写脚本处理直接 patch 龙哥星球里有个案例就是直接写 ida 脚本就能 patch 我一开始搞这个 tiny 以为也是一样的套路 后面写了一半脚本之后感觉太痛苦了直接换了一个懒人暴力方案

思路就是：直接用 ida 脚本获取所有 br blr 指令的地址 指令 然后用 unidbg 或者 frida 去 hook 此时的地址 拿到真实经过的寄存器 记录下来后用 ida 脚本 set 回去 如果出现一个指令跳多个地址的就不 patch 了 加一个注释在后面

脚本直接给了, 其实这个思路实践下来还是有坑的 比如还得处理 jni 和调用 libc 等函数的 blr 这种就不 patch 就行 脚本里都处理了这些坑

```
# 获取所有br blr
import capstone
import ida_bytes
import ida_ida
import ida_segment
import idaapi
import idautils
import idc
import keystone
# 初始化架构和模式
CS = capstone.Cs(capstone.CS_ARCH_ARM64, capstone.CS_MODE_ARM)
# 设置为详细反汇编模式
CS.detail = True
# 设置反汇编跳过数据
CS.skipdata = True
ks = keystone.Ks(keystone.KS_ARCH_ARM64, keystone.KS_MODE_LITTLE_ENDIAN)
def getAddrRange():
    start = ida_ida.inf_get_min_ea()
    size = ida_ida.inf_get_max_ea() - start
    # 尝试将地址范围限定于text节
    for seg in idautils.Segments():
        seg = idaapi.getseg(seg)
        segName = ida_segment.get_segm_name(seg)
        if segName == ".text":
            start = seg.start_ea
            size = seg.size()
    return start, size
# 获取代码段的起始地址和长度
start, size = getAddrRange()
# 获取对应的数据，字节数组形式
codebytes = idc.get_bytes(start, size)
# 反汇编，参数1是待反汇编的数据，参数2是基地址，参数3是反汇编长度
disasmCodes = list(CS.disasm_lite(codebytes, start, size))
matches = []
debug = False
for i in range(len(disasmCodes)):
    # address, size, mnemonic, op_str
    address, size, mnemonic, op_str = disasmCodes[i]
    if (mnemonic == "blr" or mnemonic == "br") and (op_str == "x8" or op_str == "x9" or op_str == "x10"):
        matches.append(hex(address)+";" + op_str+";"+mnemonic)
print(len(matches))
with open(rf"b_addr_884.txt", "w") as f:
    for m in matches:
        f.write(m+"\n")

```

```
# 把hook到的地址set回IDA的脚本
import re
import capstone
import ida_bytes
import ida_ida
import ida_segment
import idaapi
import idautils
import idc
import keystone
# 初始化架构和模式
CS = capstone.Cs(capstone.CS_ARCH_ARM64, capstone.CS_MODE_ARM)
# 设置为详细反汇编模式
CS.detail = True
# 设置反汇编跳过数据
CS.skipdata = True
ks = keystone.Ks(keystone.KS_ARCH_ARM64, keystone.KS_MODE_LITTLE_ENDIAN)
def generate(code, addr):
    encoding, _ = ks.asm(code, addr)
    return encoding
#
with open(rf"frida_b_addr_884.txt") as f:
    content = f.read()
b_dict = {}
b_value_dict = {}
b_list = content.split("\n")
for info in b_list:
    if info:
        temp = info.split(";")
        if len(temp) < 5:
            continue
        b_addr = temp[0]
        b_addr = int(b_addr, 16)
        b_value = temp[1]
        b_value = int(b_value, 16)
        b_reg = temp[2]
        b_so = temp[3]
        b_symbol_name = temp[4]
        if temp[1] == b_symbol_name:
            b_symbol_name = ""
        info = {
            "b_value": hex(b_value),
            "reg": b_reg,
            "so": b_so,
            "symbol_name": b_symbol_name
        }
        if b_dict.get(b_addr) is None:
            b_dict[b_addr] = []
            b_dict[b_addr].append(info)
            b_value_dict[b_addr] = []
            b_value_dict[b_addr].append(b_value)
        else:
            if b_value not in b_value_dict[b_addr]:
                b_dict.get(b_addr).append(info)
                b_value_dict[b_addr].append(b_value)
print(b_dict.__len__())
with open(rf"b_addr_884.txt") as f:
    content = f.read()
raw_b_dict = {}
b_list = content.split("\n")
for info in b_list:
    if info:
        temp = info.split(";")
        b_addr = temp[0]
        b_mnemonic = temp[2]
        raw_b_dict[b_addr] = b_mnemonic
with open(rf"import_functions_884.txt") as f:
    content = f.read()
ida_func_dict = {}
b_list = content.split("\n")
for info in b_list:
    if info:
        temp = info.split(";")
        import_name_and_so_name = temp[0]
        symbol_name = import_name_and_so_name.split("@@")[0]
        offset = temp[1]
        ida_func_dict[symbol_name] = offset
counts = 0
for k,v in b_dict.items():
    if len(v) > 1:
        now_addr = k
        comm = "调用多个地址:"
        for _ in v:
            comm += _["b_value"] + ";"
        idaapi.set_cmt(now_addr, comm, 0)
        print(hex(now_addr), comm)
    else:
        # 当前地址
        now_addr = k
        # 跳转地址
        b_value = v[0]["b_value"]
        # 跳转地址存的寄存器
        b_reg = v[0]["reg"]
        b_so = v[0]["so"]
        b_symbol_name = v[0]["symbol_name"]
        if b_so != "libtiny.so" and b_so != "libart.so" and b_so != "libandroid.so":
            # 处理跳转不是tiny.so 的跳转
            ida_offset = ida_func_dict.get(b_symbol_name.replace("64",""))
            if raw_b_dict.get(hex(now_addr)) == "blr":
                b_code = f"bl #{ida_offset}"
            else:
                b_code = f"b #{ida_offset}"
            print(hex(now_addr), f"当前跳转到{b_so} {b_symbol_name} {b_code}")
            b_bytes = bytes(generate(b_code, now_addr))
            ida_bytes.patch_bytes(now_addr, b_bytes)
        elif b_so == "no pointerModule":
            ...
        else:
            # 得看之前是br还是blr
            if raw_b_dict.get(hex(now_addr)) == "blr":
                b_code = f"bl #{b_value}"
            else:
                b_code = f"b #{b_value}"
            b_bytes = bytes(generate(b_code, now_addr))
            ida_bytes.patch_bytes(now_addr, b_bytes)

```

上面有个 import_function_884.txt 文件就是获取 ida 中导入系统函数的地址 生成脚本如下

```
import idaapi
import idc
import idautils
def get_all_imports():
    print("Collecting all imported functions...")
    result = []
    # 获取二进制文件基址
    base_addr = idaapi.get_imagebase()
    print(f"Image base: 0x{base_addr:X}")
    # 遍历所有导入项
    nimps = idaapi.get_import_module_qty()
    print(f"Found {nimps} import modules")
    for i in range(nimps):
        module_name = idaapi.get_import_module_name(i)
        if not module_name:
            print(f"Failed to get import module name for index {i}")
            continue
        print(f"Processing module: {module_name}")
        # 使用回调函数获取每个模块的导入函数
        def imp_cb(ea, name, ordinal):
            if name:
                offset = ea - base_addr
                result.append({
                    "name": name,
                    "offset": f"0x{offset:X}",
                    "module": module_name
                })
            return True
        # 应用回调函数到当前模块的所有导入项
        idaapi.enum_import_names(i, imp_cb)
    return result
def main():
    imports = get_all_imports()
    # 打印结果
    print("\n{:<40} {:<16} {:<10}".format("Function Name", "Offset", "Module"))
    print("-" * 70)
    for imp in imports:
        print("{:<40} {:<16} {:<10}".format(
            imp["name"],
            imp["offset"],
            imp["module"]
        ))
    print(f"\nTotal imported functions: {len(imports)}")
    # 将结果保存到文件，使用分号分隔
    with open(rf"import_functions_884.txt", "w") as f:
        for imp in imports:
            f.write(f"{imp['name']};{imp['offset']};{imp['module']}\n")
    print("Results saved to import_functions.txt")
if __name__ == "__main__":
    main()

```

最后是 frida hook 脚本

```
function hook_dlopen(soName = '') {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                if (path.includes('libtiny.so')) {
                    this.isTinySo = true;
                    this.path = path;
                }
            }
        },
        onLeave: function (retval) {
            // Check if this was the libtiny.so load
            if (this.isTinySo) {
                get_b_addr()
            }
        }
    });
}
hook_dlopen();
function get_b_addr(){
        // 假设这个函数会返回你需要监控的地址列表
        function get_addrs() {
            // 在这里实现获取地址的逻辑，或者直接硬编码返回地址列表
            var text = `0x199f68;x9;br
0x199f78;x8;br`;
            return text.split("\n");
        }
        var addrs = get_addrs();
        // 获取目标模块基址
        var baseAddr = Module.findBaseAddress('libtiny.so');
        if (!baseAddr) {
            console.log("[-] Failed to find libtiny.so base address");
            return;
        }
        // 遍历地址列表添加断点
        addrs.forEach(function(text) {
            var items = text.split(';');
            var addr = items[0];
            var op = items[1];
            var addrValue = parseInt(addr.replace("0x", ""), 16);
            // 计算实际地址（UniDBG中是0x12000000L + addr_，这里使用实际模块基址）
            var targetAddress = baseAddr.add(addrValue);
            // 设置断点
            Interceptor.attach(targetAddress, {
                onEnter: function(args) {
                    try {
                        // 读取对应的寄存器值 大多数就这三种
                        var regValue;
                        if (op === "x8") {
                            regValue = this.context.x8;
                        } else if(op == "x9") { // x9
                            regValue = this.context.x9;
                        } else if(op == "x10") {
                            regValue = this.context.x10;
                        } else{
                            console.log("未处理的寄存器:" + op);
                        }
                        var offset;
                        var name = "";
                        var symbolName = "";
                        // 检查指针指向的位置是否在模块范围外
                        var pointerModule = Process.findModuleByAddress(regValue);
                        if (pointerModule) {
                            // 指针指向其他模块
                            offset = regValue.sub(pointerModule.base);
                            name = pointerModule.name;
                            if (name.includes("libart")){
                            }else{
                                // 尝试查找最近的符号
                                try {
                                    var symbol = DebugSymbol.fromAddress(regValue);
                                    if (symbol && symbol.name) {
                                        symbolName = symbol.name;
                                    }
                                } catch (e) {
                                    symbolName = "";
                                }
                                var returnAddr = this.returnAddress;
                                var lr_offset = ptr(returnAddr).sub(baseAddr)
                                console.log(addr + ";0x" + offset.toString(16) + ";" + op + ";" + name + ";" + symbolName+";lr=0x"+lr_offset.toString(16));
                            }
                        } else {
                            // 假设是同一模块内的偏移，相当于UniDBG中的 pointer - 0x12000000L
                            offset = regValue.sub(baseAddr);
                            console.log(addr + ";0x" + offset.toString(16) + ";" + op + ";"+ "no pointerModule:"+ regValue);
                        }
                    } catch (e) {
                        console.log("[-] Error in hook: " + e);
                    }
                }
            });
        });
}
// frida -U -f com.xingin.xhs -l hook_xhs_884_b_addr.js -o frida_b_addr_884.txt

```

spawn 模式运行就可以 hook 到所有经过的寄存器地址

这里 get_addrs 函数需要 hook 哪个范围的地址呢 一般来说当然是越多越好 一把梭 把所有运行时的地址都拿到回填回去 但是这样有崩溃卡死或者触发未知的反调试风险 我们确定一个地址返回

就拿 JNI_OnLoad 的起始地址 199EE0 当开始

那么结束如何选择

按经验来说我会找 RET 指令和 BL              .__stack_chk_fail 这两个指令同时出现的地方 

全局搜索 RET 指令和 BL              .__stack_chk_fail 指令

找到 JNI_Onload 附近

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhTPRial4FXvBdHTxmLKAmUNzEPgzxmMzSksHuHXlECkGH6dKp77AcGNg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=22)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xhxa51OHtnjZj6AB0o7o4ATsibZrpfLC1fiacrl1YWufnibicCIfRpwyU6zA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=23)

看起来就是 19AF8C

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhOxn9UhsrVsRXZ1sic8haC3WOAHVad2a9gaXr3q0CoGGkxY1ClkhabFQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=24)

选择这个地址当结束

然后在我们获得所有 br blr 的结果里找到这两个地址的中间所有地址拿出来 hook

直接启动拿到结果

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xh5bpPR224zXJDptIcCrjNtBuEHH2hd1rLiatvabdkkXsgy3usv3w9TJA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=25)

直接运行就可以

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhR4SVXXDfNrakaITLXaAHNMpf5hNgO4V6ww15wtR2ZPSusVicxVTRwUQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=26)

然后他就 patch 了所有 br 或者 blr 的指令

可以从 jni onload 开始看逻辑了

继续刚刚手动 patch 到的一个函数

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xh17AtYsXek4Pzcf3f7ibXYRvUqkaEGH2Ak00bic4aVex43L8aUbiaDKZ0A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=27)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xh5GE1fB1qkhV46Y2wlwTRyN1QT6fMMw1f3VibjDWl9tBYTDXPtPicaHMQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=28)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhAow76brFPgJBer7XoR5icfwvGMIiap4vlzuibLQphwbZxk7kat8LrYvmg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=29)

这里可以看到没有 patch 然后转汇编界面看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhKp8ltKWLxtDUc0UWYy9ReBPX3dbExs1FeyUFNr725ltZ1K0llyNibGA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=30)

是调用多个地址的分支 可以一个个去看看做了什么

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhEE8RCZhfbvzwcH28RwbLibTia66uwc5mImm9lehic63lLicSvcku5kscFw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=31)

sub_122380 这个点进去看就知道是线程锁的逻辑丢给 ai 总结下就行

cxa_guard_acquire 实际上就是这个作用

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhVKDrWQFVOr37cKTbjx20icMRGO26xxjiczNy6N5z73CLribYlqwHqqxdQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=32)

然后又回到了这里

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xhbg0qgyj2TPO1VcBNzew4FwOrjqZY6GHLcb2EI4LU6hlc9FxicKLIpFw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=33)

那么就是走的第二个地址 0x199fbc 是一个判断应该是刚刚的线程锁是否加锁成功

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0XhRx9ib1TgAscl1b0ttTJqrr2ttmZfevd28ne3uFc9chxFcJGicLZT7iaTw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=34)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQibWpohrtDLgGQBtPJicR0Xha8MG9RL35Giah1p8pU7hmwgiajcwlfDXnoic2t3yNicfNWic2YzDAHhT73w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=35)

然后一个个跟就行了。。太多了这里 都是体力活儿只要做好记录就能知道他的跳转逻辑和每个函数在干嘛

jni_onload 这里其实就是拿来先观察一下他花指令的逻辑

下一篇我在讲一个去花后逆向设备指纹的实战案例

花指令我也是刚接触搞得不多，angr 什么的是不会的 只能来一手暴力 hook 稍微解决下

感觉 so 的反混淆对比 js 的 ast 还是难很多

太菜了 还得继续学习