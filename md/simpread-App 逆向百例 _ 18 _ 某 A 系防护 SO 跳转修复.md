> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XF-qnLwii9Xt734a2aXOWA)

**观前提示：**  

**本文章仅供学习交流, 切勿用于非法通途, 如有侵犯贵司请及时联系删除**

```
样本：aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMWl5Q21NNVd1WC1NTkNsRkh2X0w3TVE/cHdkPWxpbm4=

```

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFqGnaicm1h4xpn0ibWGM0icuGiaibAMnSFr3WlD4dFtAghO90YJJzkVmPSZA/640?wx_fmt=png)

0x0 前言
======

最近比较忙，拖更了一段时间，不过好在事情在逐步逐步地减少，毕设搞完了，论文答辩了，毕业照也拍了，出来社会当上社畜了，所以更新频率会降低很多，不过我还是会将工作上遇到的东西拿出部分来做案例分享，可能会比较简单甚至出错，大佬们轻点喷。

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFYVNPAjO1jne20bmsGc4oUzhj9zFVFfy4wu2JsWItvJeoyiamAHc1AVQ/640?wx_fmt=png)

0x1 手动分析
========

大姐姐打开`libsgmainso-6.5.55.so`

直接在导出表找到`JNI_OnLoad`进来

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaF0LxibNDLPytibuUE9bl0ZIV7iaSDcWbs8picHGZUEFjicrFkQLPRSH2esoQ/640?wx_fmt=png)

可以看到没有正常的解析出预想的代码 而是出现了

```
__asm { BR              X11 }


```

跳回汇编看看

```
SUB             SP, SP, #0x50
STP             X25, X23, [SP,#0x40+var_30]
STP             X22, X21, [SP,#0x40+var_20]
STP             X20, X19, [SP,#0x40+var_10]
STP             X29, X30, [SP,#0x40+var_s0]
ADD             X29, SP, #0x40
MRS             X21, #3, c13, c0, #2
LDR             X8, [X21,#0x28]
MOV             W9, #0x8A
ADRL            X10, dword_166320
STR             X8, [SP,#0x40+var_38]
STR             W9, [SP,#0x40+var_3C]
LDRB            W9, [X10,#(byte_166352 - 0x166320)]
MOV             X20, X0
ADD             X10, SP, #0x40+var_3C
MOV             W8, #0xCE
ADD             W9, W9, #0x8A
ADRP            X22, #0x166000
STR             W9, [SP,#0x40+var_3C]
ADR             X11, loc_1E10C
LDRSW           X3, =0xFFFFFEF7
LDRSW           X25, [X10]
ADD             X3, X3, X25
ADD             X11, X11, X3
MOV             X9, #0x16
BR              X11


```

根据上面的汇编代码可知 最终需要跳转的地址放在`X11`寄存器里面 而寄存器是通过计算得到的 除去入栈出栈的操作 最终可以简化为

```
X10 = 0x166320
W9  = X10 + 0x32 = 0x166352=>0xab
W9  = W9  + 0x8a = 0x135
X25 = W9
X3  = 0xFFFFFEF7 + X25 = 0x2c
X11 = 0x1E10C + X3 = 0x1e138


```

得到 X11 的地址手动 Patch 一下看看效果

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFFCqWQrGOUQB9QQL4ibScBLMkP6sJOibtickjYUfUU3AIo0mltuenD5PNA/640?wx_fmt=png)

确认后发现对应的地址是一堆常量 那就需要我们手动将对应地址回归 undefined 状态（快捷键 U）

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaF23KbqbHicdhRLicpfNDUIkgu6m5T4MbXdqo8Nz0VQiaicClZocXxB6Xw3w/640?wx_fmt=png)

接着再到对应的地址创建 Function（快捷键 P）

最后回到伪代码界面 按下 F5 让代码重新分析即可

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFIvKu6V4PbY43Ckp0gSkZ96N4K3AQU3KABBJd8db5J7ygJ0kWZhDxaQ/640?wx_fmt=png)

这里被 IDA 识别为 sub 方法 不过问题不大 只需保存 Patch 文件 然后 IDA 重新打开即可

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFkecyT19ibYzknjqaKm3cB5Cfk4tV6Wa9AvzgU6WdhOYSTfPC0R9D5pw/640?wx_fmt=png)

接回原来的话 手动修复跳转后 可以看到大段的代码 说明目的已经达到了

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFYicicF1nc6ptaVVdpeQNKfaJXtSWpRSwe3To566ibpFsAuYD8ZVnQYGFQ/640?wx_fmt=png)

但是一个 so 里面不止一处的出现跳转 况且还有好几个 so 需要跳转修复 这将是一处大工程 手动还原是不现实的

0x2 脚本修复
========

这里我将尝试使用`flare_emu`代替手动计算工作 为此创建一个脚本

将范围限制在`[0x1E0B8-0x1E120]`

```
import flare_emu
myEH= flare_emu.EmuHelper()
myEH.emulateRange(
  startAddr=0x1E0B8,
  endAddr=0x1E120
)
myEH.getRegVal("X11")


```

将脚本放到 IDA 里面运行 得到`X11`的结果为`0x1e08d`

这和我们前面计算的`0x1e138`相差甚远

那么问题出现在哪里呢？

通过分析发现 手动获取`0x166352`得到

```
myEH.getEmuBytes(0x166352,1)
Out[9]: bytearray(b'\x00')


```

而在 unidbg 中拿到`0x166352`却是有值的![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFX72860OSc55HZw0ME4biaL11rIoQNXrsTfGnmTkJGWmHud2icgXm8G0Q/640?wx_fmt=png)

那么问题就显而易见了 如果要解决可以使用`dump_so`去拿到初始化后的 so 文件

但是我并没有那么做 这次我使用 unidbg 去做跳转修复（有点杀鸡用牛刀的感觉了）

0x3 unidbg 跳转修复
===============

前面已经分析了那么多 我就直接上脚本了（学习借鉴了一篇看雪的文章）

首先你得补出一份可以跑得通的 unidbg 环境

主要思想就是 tracecode 并实时保存 10 条汇编数据 只要出现`BR`跳转指令 就进入判断特征 符合我们筛选的条件后 记录地址和真实跳转地址 等待完全执行完后 将记录下来的所有数据替换进原始 SO 文件输出修复后的 SO 文件

```
public void find_junk(long base_addr, long so_size) {
        patchlist = new ArrayList<>();
        emulator.getBackend().hook_add_new(new CodeHook() {
            int count_code = 0;
            String[] opcode = new String[10];
            Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM64, Capstone.CS_MODE_ARM);
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                byte[] bytes = emulator.getBackend().mem_read(address, 4);
                Instruction[] disasm = capstone.disasm(bytes, 0);
                String mnemonic = disasm[0].getMnemonic();
                String opStr = disasm[0].getOpStr();
                opcode[(count_code % 10)] = mnemonic;
                count_code++;
                if (mnemonic.indexOf("br") != -1) {
                    List<String> list = Arrays.asList(opcode);
                    if (list.contains("add") && list.contains("ldrsw")) {
                        int reg = 0;
                        switch (opStr) {
                            case "x0":
                                reg = 199;
                                break;
                            case "x1":
                                reg = 200;
                                break;
                            case "x2":
                                reg = 201;
                                break;
                            case "x3":
                                reg = 202;
                                break;
                            case "x4":
                                reg = 203;
                                break;
                            case "x5":
                                reg = 204;
                                break;
                            case "x6":
                                reg = 205;
                                break;
                            case "x7":
                                reg = 206;
                                break;
                            case "x8":
                                reg = 207;
                                break;
                            case "x9":
                                reg = 208;
                                break;
                            case "x10":
                                reg = 209;
                                break;
                            case "x11":
                                reg = 210;
                                break;
                            case "x12":
                                reg = 211;
                                break;
                            case "x13":
                                reg = 212;
                                break;
                            case "x14":
                                reg = 213;
                                break;
                            case "x15":
                                reg = 214;
                                break;
                            case "x16":
                                reg = 215;
                                break;
                            case "x17":
                                reg = 216;
                                break;
                            case "x18":
                                reg = 217;
                                break;
                            case "x19":
                                reg = 218;
                                break;
                            case "x20":
                                reg = 219;
                                break;
                            case "x21":
                                reg = 220;
                                break;
                            case "x22":
                                reg = 221;
                                break;
                            case "x23":
                                reg = 222;
                                break;
                            case "x24":
                                reg = 223;
                                break;
                            case "x25":
                                reg = 224;
                                break;
                            case "x26":
                                reg = 225;
                                break;
                            case "x27":
                                reg = 226;
                                break;
                            case "x28":
                                reg = 227;
                                break;
                        }
                        long value = (long) emulator.getBackend().reg_read(reg);
                        int TRUE_JUMP = (int) (value - base_addr);
                        if (TRUE_JUMP < so_size) {
                            PatchIns patchIns = new PatchIns();
                            patchIns.setAddr(address - base_addr);
                            patchIns.setIns("b 0x" + Integer.toHexString(TRUE_JUMP));
                            patchlist.add(patchIns);
                        }
                    }
                }
            }

            @Override
            public void onAttach(UnHook unHook) {}

            @Override
            public void detach() {}
        }, base_addr, base_addr + so_size, null);

    }

    public void save_fix(String so_path, String so_fix_path) {
        Keystone keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian);
        try {
            File sgmain = new File(so_path);
            FileInputStream fileInputStream = new FileInputStream(sgmain);
            byte[] data = new byte[(int) sgmain.length()];
            fileInputStream.read(data);
            fileInputStream.close();
            for (PatchIns patchins : patchlist) {
                KeystoneEncoded assemble = keystone.assemble(patchins.ins, (int) patchins.addr);
                for (int i = 0; i < assemble.getMachineCode().length; i++) {
                    data[(int) patchins.addr + i] = assemble.getMachineCode()[i];
                }
            }
            File sgmain_fix = new File(so_fix_path);
            FileOutputStream fileOutputStream = new FileOutputStream(sgmain_fix);
            fileOutputStream.write(data);
            fileOutputStream.flush();
            fileOutputStream.close();
            System.out.println("fix finish,fix count->" + patchlist.toArray().length);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }


```

最终修复效果![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFoRS6BOvFsxkZMW23Z8qfybkKhibBo9Z6r4I8uiaPRiaAKMYXqE83FsPmw/640?wx_fmt=png)

还原效果还是很不错的 只要运行到了的 基本被修复好了![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaF7hiaBtWptweucWeHC4tXYfzUarKpn38rscicNfLiapmPDBibROh6nOThGA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFjmiatL27Sslf2sojD2oEoEf8vYgc46PWQK06ZvxXsgx8tZvAW9qO6Dg/640?wx_fmt=png)

看看`doCommandNative`入口 效果也可以![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFebyBCOtQtBkFCd7OMfklGfEfmAFkPZiaTZtKlY2LMeMjpq3LbwZGxjg/640?wx_fmt=png)

不过也会出现例外的情况 例如极少量的误判没有还原到 不过更大的可能性是 没有执行到这个位置 所以没做出修复操作 例如下面这种分支

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMId94n7220HF6apIZYqhaFVtsNicNzrD0draRymKXoXIbYia0xENWfNoWjLfC8VHGxmLG9r6Ud4Z3g/640?wx_fmt=png)

不过总的来说效果还是 ok 的

0x3 参考文章
========

《记一次基于 unidbg 模拟执行的去除 ollvm 混淆》 https://mp.weixin.qq.com/s/KuWi39Grl9lrhI8iY_S8pw