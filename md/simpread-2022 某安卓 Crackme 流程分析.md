> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIxNjMwMzkwOA==&mid=2247485239&idx=1&sn=be7cb8e18cad26ba242a44d5de496bde&chksm=978a577ca0fdde6a274bc79a4eb6d386fddf48c0085b084e715020ed4e8d414b5dcced3d1f97&mpshare=1&scene=1&srcid=0216MEd0pVJ6ywEAa986ZKU5&sharer_sharetime=1644973105735&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

安卓题目指路：https://www.52pojie.cn/thread-1582582-1-1.html

jadx 打开 apk

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaOPzToMvcJRyrkHtOIsXuYgXmZTZWAgNdlaiahy1x8wXer6wADfL6AKA/640?wx_fmt=png)

锁定函数  

```
public native boolean checkSn(String str);

```

根据输入的值来校验返回值，当输入正确的 flag 时返回 true

打开 lib52pojie.so，JniOnload 动态加载 checkSn

unidbg 代码获取下函数地址

```
package com.pojie52;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.TraceHook;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.listener.TraceWriteListener;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.trace.KingTrace;
import java.io.*;
public class uniStart extends AbstractJni implements IOResolver {
   private AndroidEmulator emulator;
   private VM vm;
   private Module module;
   private Boolean logging = true;
   private String processName = "com.wuaipojie.crackme2022";
   private DvmClass mainActivity;
   private Debugger debugger;
   public uniStart(boolean logging) throws IOException {
       this.logging = logging;
       emulator = AndroidEmulatorBuilder.for64Bit().setProcessName(processName).build();
       final Memory memory = emulator.getMemory();
       memory.setLibraryResolver(new AndroidResolver(23));
       emulator.getSyscallHandler().addIOResolver(this);
       vm = emulator.createDalvikVM(new File("unidbg-android/src/main/resources/android/apk/524.apk"));
       DalvikModule dm = vm.loadLibrary("52pojie", true);
       vm.setVerbose(logging);
       vm.setJni(this);
       dm.callJNI_OnLoad(emulator);
       module = dm.getModule();
       mainActivity = vm.resolveClass("com/wuaipojie/crackme2022/MainActivity");
       debugger = emulator.attach();
  }
   public static void main(String[] args) throws IOException {
       uniStart uniStart = new uniStart(true);
  }
   @Override
   public FileResult resolve(Emulator emulator, String pathname, int oflags) {
       System.out.println("pathname: " + pathname);
       return null;
  }
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiayziaD8HLnnOBsAqQq6A4BdiaJdF1kSe1F4ic9Q7QpP7fXWefsXqbP6nqg/640?wx_fmt=png)

拿到函数地址：0x6f74  

lib52pojie.so 静态分析该地址处的函数，开发过程使用了内联汇编，静态基本上得不到什么有效信息

直接使用 unidbg 来 trace 分析

KingTrace 指路：https://github.com/dqzg12300/unidbg_tools/tree/main/unidbg-api/src/main/java/trace

```
public class uniStart extends AbstractJni implements IOResolver {
   private AndroidEmulator emulator;
   private VM vm;
   private Module module;
   private Boolean logging = true;
   private String processName = "com.wuaipojie.crackme2022";
   private DvmClass mainActivity;
   private Debugger debugger;
   public uniStart(boolean logging) throws IOException {
       this.logging = logging;
       emulator = AndroidEmulatorBuilder.for64Bit().setProcessName(processName).build();
       final Memory memory = emulator.getMemory();
       memory.setLibraryResolver(new AndroidResolver(23));
       emulator.getSyscallHandler().addIOResolver(this);
       vm = emulator.createDalvikVM(new File("unidbg-android/src/main/resources/android/apk/524.apk"));
       DalvikModule dm = vm.loadLibrary("52pojie", true);
       vm.setVerbose(logging);
       vm.setJni(this);
       dm.callJNI_OnLoad(emulator);
       module = dm.getModule();
       String StringFile = "unidbg-android/src/main/java/com/pojie52/trace1.txt";
       KingTrace.traceCode(emulator, module, StringFile);
       mainActivity = vm.resolveClass("com/wuaipojie/crackme2022/MainActivity");
       debugger = emulator.attach();
  }
   public static void main(String[] args) throws IOException {
       uniStart uniStart = new uniStart(true);
       uniStart.checkSn();
  }
   public void checkSn() {
       boolean success = mainActivity.callStaticJniMethodBoolean(emulator, "checkSn(Ljava/lang/String;)Z",
               new StringObject(vm, "adair"));
       System.out.println("success: " + success);
  }
   @Override
   public FileResult resolve(Emulator emulator, String pathname, int oflags) {
       System.out.println("pathname: " + pathname);
       return null;
  }
}

```

最终只有 75 行，猜测入参长度问题导致在入口处就直接返回了

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaWGxayu8oexgSKu2qNVKtc8icjB2VErpfUy8GQV90RJyZiceeA3KwXxicg/640?wx_fmt=png)

在 trace.log 中搜索 cmp 指令  

```
[lib52pojie.so] [0x06fd8] [ 1f 40 00 71 ] 0x40006fd8: cmp w0, #0x10 >>>   w0=0x5//w0=0x5

```

判断长度是否为 0x10，由此判断输入字符串长度应为 16 字节才会继续往下执行（从 java 代码中也可获取该信息），修改入参为：`52pojieairadaira`

重新 trace

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaMMs7ncO5nnO2uYZs0qmblXY5icwZG5jCwvPzUMsm3iaeJWszRS21JXMQ/640?wx_fmt=png)

39000 + 行，trace 应该是正常了，当然这个入参函数返回的结果肯定是 false，根据题目要求，传入参数，经过计算得到值，那么这个值一定会与内存中某个值做异同判断，再根据判断的结果返回到 java 中的 boolean 中，那么就还是从 trace 日志中的 cmp 指令下手，全局搜索 cmp 关键字从后往前找，  

```
[lib52pojie.so] [0x07848] [ 1f 00 00 71 ] 0x40007848: cmp w0, #0 >>>   w0=0xffffffa2//w0=0xffffffa2
[lib52pojie.so] [0x0785c] [ ea 17 9f 1a ] 0x4000785c: cset w10, eq >>>   w10=0xd87d7949//w10=0x0
[lib52pojie.so] [0x078e4] [ 40 01 00 12 ] 0x400078e4: and w0, w10, #1 >>>   w0=0xffffffa2w10=0x0//w0=0x0
[lib52pojie.so] [0x07904] [ c0 03 5f d6 ] 0x40007904: ret

```

贴近 trace 尾部的位置，在一个函数中 cmp 之后 ret 了 0x0 出去，那么大概率这个地方是最终判断两值是否相等的位置，其中 cset 指令的意思时通过判断上一条 cmp 指令的结果对结果寄存器赋值，在该例中为

```
cmp w0, #0 >>> w0 != 0x0 -> Z标志位变成0
cset w10, eq >>> w10 = 0x0 # 如果上面cmp之后之后Z标志位的值为1，则此处w10 = 0x1

```

理解了 cset 的原理，再通过 unidbg 的 patchCode 功能来验证一下

```
public static void main(String[] args) throws IOException {
   uniStart uniStart = new uniStart(true);
   uniStart.debugger1();
   uniStart.checkSn();
}
public void debugger1() {
   // byte[] patchCode = {(byte) 0xea, 0x07, (byte) 0x9f, (byte) 0x1a}; // EA079F1A -> cset w10, ne 0x785c
   byte[] patchCode = {(byte) 0x1f, (byte) 0x00, (byte) 0x00, (byte) 0x6B};  // 1F00006B -> cmp w0, w0
   // cmp w0, w0结果恒为true
   emulator.getBackend().mem_write(module.base + 0x7848, patchCode);
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiadOic5FdjmicxZMpkzTOkNP5PzydIn4hKStufyMubKan6iaLJjJOFT7KbQ/640?wx_fmt=png)

最终结果返回 true，那么此处位置则可以暂时认为是最终判断的位置，记录此处地址：0x7848  

以该地址为起始点，往上查找 w0 或者 x0 第一次出现的位置

```
[lib52pojie.so] [0x0798c] [ e0 03 15 aa ] 0x4000798c: mov x0, x21 >>>   x0=0x40398058x21=0x40398058//x0=0x40398058

```

通过 unidbg 的 debugger 功能断点调试，查看 x0 和 x1 地址处的内存值

```
private Debugger debugger;
public uniStart(boolean logging) throws IOException {
   debugger = emulator.attach();
}
public static void main(String[] args) throws IOException {
   uniStart uniStart = new uniStart(true);
   uniStart.debugger1();
   uniStart.checkSn();
}
public void debugger1() {
   debugger.addBreakPoint(module.base + 0x798c);
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaxz5WmMOcncGEBiabLGmeGAxNdJ5ibPfVickvUK0ibqmQfUbcW0sS78UwYQ/640?wx_fmt=png)

得到两个 32 字节长度的 hex，初步判断这里可能就是最终需要做判断的两个值，通过修改入参发现，x0 在内存中的值动态变化，x1 在内存中的值是固定不变的，记录下这个地方的值  

```
maybe encrypt result: 56ec3d4c526764d7949fe1f201d9fe45ce3601c8d282129884d8bf985034450e
maybe check result: 6e6649305baf80c49b1b063c0500c80346ccfd42b3063ae7312b52a21cd334d8

```

根据 x1 地址 0x40398018 逆推，找到给这个地址赋值的地方

```
[lib52pojie.so] [0x07208] [ 68 01 00 f0 ] 0x40007208: adrp x8, #0x40036000 >>>   x8=0x0//x8=0x40036000
[lib52pojie.so] [0x0720c] [ 08 11 42 f9 ] 0x4000720c: ldr x8, [x8, #0x420] >>>   x8=0x40036000x8=0x40036000//x8=0xffffffffc5c93154
[lib52pojie.so] [0x07210] [ 89 13 87 52 ] 0x40007210: movz w9, #0x389c >>>   w9=0x0//w9=0x389c
[lib52pojie.so] [0x07214] [ 49 47 af 72 ] 0x40007214: movk w9, #0x7a3a, lsl #16 >>>   w9=0x389c//w9=0x7a3a389c
[lib52pojie.so] [0x07218] [ 01 01 09 8b ] 0x40007218: add x1, x8, x9 >>>   x1=0x40036a70x8=0xffffffffc5c93154x9=0x7a3a389c//x1=0x400369f0
[lib52pojie.so] [0x07598] [ f3 03 01 aa ] 0x40007598: mov x19, x1 >>>   x19=0x3796751bx1=0x400369f0//x19=0x400369f0
[lib52pojie.so] [0x07698] [ e0 03 13 aa ] 0x40007698: mov x0, x19 >>>   x0=0x20x19=0x400369f0//x0=0x400369f0
[lib52pojie.so] [0x0c864] [ 00 00 40 f9 ] 0x4000c864: ldr x0, [x0] >>>   x0=0x400369f0x0=0x400369f0//x0=0x40398018
[lib52pojie.so] [0x077dc] [ f6 03 00 aa ] 0x400077dc: mov x22, x0 >>>   x22=0x4000c884x0=0x40398018//x22=0x40398018
[lib52pojie.so] [0x077fc] [ e1 03 16 aa ] 0x400077fc: mov x1, x22 >>>   x1=0x400369f0x22=0x40398018//x1=0x40398018
[lib52pojie.so] [0x07990] [ e1 03 14 aa ] 0x40007990: mov x1, x20 >>>   x1=0x40398018x20=0x40398018//x1=0x40398018

```

从上往下看，先是 x8 处指向固定地址 0x40036000，接着从该地址 + 0x420 的地址处取出数据赋值给 x8

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWialqHq60zutKOpjica4Dkr6Ps7kKO6iaTZorYaT568NWU8kLJZPxGWBbXw/640?wx_fmt=png)

固定 w9 取值为 0x7a3a389c，与 x8 相加得到 0x400369f0 地址，在此处获取该地址的内存值如下  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiamSpyxu3o7E5tk0oQOwqrNmPtuGkujgI7W44jibaibscj0sMTLWdvkQSg/640?wx_fmt=png)

可得上述 x1 的值是固定写死在内存中的，这个结论极大地加深了该值作为 check result 的可能性，此时记录该内存值  

回过头，我们再看下 encrypt result 的 set 过程

一步步根据汇编回推太慢，使用 unidbg 的 memory trace write 功能定位关键写入位置

```
public void debugger1() {
   emulator.traceWrite(0x40398058, 0x40398058+32);
   emulator.traceWrite(0x40398058, 0x40398058+32, new TraceWriteListener() {
       @Override
       public boolean onWrite(Emulator<?> emulator, long address, int size, long value) {
           emulator.getUnwinder().unwind();
           return false;
      }
  });
   debugger.addBreakPoint(module.base + 0xcc1c);
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaoHfvw2CtHrOX4jGxmmK7iahLe58DxPcsk3IXuQ5VYc1OzN5mNlU1lVw/640?wx_fmt=png)

debugger 调试 0xcc1c 地址  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiatgH73aDaiazxAjDbFCvl6xibia860t2A3MGWsE0hQJSpicdKp32JJCd1lw/640?wx_fmt=png)

此时 0x40398058 地址处还未写入值，可能的 encrypt result 和入参明文都在 x1 寄存器处，trace.log 中向上查找 0xbffff480 的写入位置  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiakKz6YDO2lLHzIs9PpXZ8JEcV3ysB2cibav5q14tSc8D4yz73yH3kVnw/640?wx_fmt=png)

一共 166 条结果，往上翻找  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWia7K3nW4WUBQtUASFKO3c9ibRtLNZ6gx4f60UBXgpnibQL6FDX6Lmzmcbg/640?wx_fmt=png)

24000 行左右在大量的往 0xbffff480 地址处存储数据，跳至目标处  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaIJvHGibT0SI9yjvicnGqtbAd2d8fhHFAZyXiaXA6SQ5QO5PDp8a2KAC4w/640?wx_fmt=png)

分别向 0xbffff480 的 0x0~0xf 偏移处写入 16 个字节的数据，将写入数据的最低位 hex 取出  

```
strb w9, [x14] >>>   w9=0xa511f056x14=0xbffff480//w9=0xa511f056                 0x56
strb w9, [x14, #1] >>>   w9=0xf0563decx14=0xbffff480//w9=0xf0563dec             0xec
strb w8, [x14, #2] >>>   w8=0x11f0563dx14=0xbffff480//w8=0x11f0563d             0x3d
strb w8, [x14, #3] >>>   w8=0x563dec4cx14=0xbffff480//w8=0x563dec4c             0x4c
strb w9, [x14, #4] >>>   w9=0xd5906852x14=0xbffff480//w9=0xd5906852             0x52
strb w12, [x14, #5] >>>   w12=0x68526467x14=0xbffff480//w12=0x68526467       0x67
strb w11, [x14, #6] >>>   w11=0x90685264x14=0xbffff480//w11=0x90685264       0x64
strb w8, [x14, #7] >>>   w8=0x526467d7x14=0xbffff480//w8=0x526467d7             0xd7
strb w13, [x14, #8] >>>   w13=0x38cf0994x14=0xbffff480//w13=0x38cf0994       0x94
strb w8, [x14, #9] >>>   w8=0x994e19fx14=0xbffff480//w8=0x994e19f             0x9f
strb w9, [x14, #0xa] >>>   w9=0xcf0994e1x14=0xbffff480//w9=0xcf0994e1         0xe1
strb w9, [x14, #0xb] >>>   w9=0x94e19ff2x14=0xbffff480//w9=0x94e19ff2         0xf2
strb w9, [x14, #0xc] >>>   w9=0xde2ab601x14=0xbffff480//w9=0xde2ab601         0x01
strb w9, [x14, #0xd] >>>   w9=0xb601fed9x14=0xbffff480//w9=0xb601fed9         0xd9
strb w8, [x14, #0xe] >>>   w8=0x2ab601fex14=0xbffff480//w8=0x2ab601fe         0xfe
strb w8, [x14, #0xf] >>>   w8=0x1fed945x14=0xbffff480//w8=0x1fed945         0x45

```

是不是和 encrypt_result 的前 16 个字节 56ec3d4c526764d7949fe1f201d9fe45 匹配上了呢

找到位置之后，再向上回溯明文到这一步的整个传输过程，以 0x0 处的 0xa511f056 为例

```
[lib52pojie.so] [0x090f4] [ e8 8d 40 f9 ] 0x400090f4: ldr x8, [x15, #0x118] >>>   x8=0x0x15=0xbffff0e0//x8=0xa511f0563dec4c
[lib52pojie.so] [0x090fc] [ 09 fd 58 d3 ] 0x400090fc: lsr x9, x8, #0x18 >>>   x9=0x0x8=0xa511f0563dec4c//x9=0xa511f056
[lib52pojie.so] [0x09100] [ c9 01 00 39 ] 0x40009100: strb w9, [x14] >>>   w9=0xa511f056x14=0xbffff480//w9=0xa511f056

```

全局搜索 0xa511f0563dec4c

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaJfgbqr0ibrabnJibOcd7gf1E6TfB9nIfTiaS3hbmjSAicmdW851IL7D0oQ/640?wx_fmt=png)

0x8ff4 地址处 x8+0x20=0xbffff1d8+0x20=0xbffff1f8  

0x90f4 地址处 x15+0x118=0xbffffe0e+0x118=0xbffff1f8

因此 0x8ff4 为 0xa511f0563dec4c 数据写入的位置

再从 0x8ff4 往上回溯就能够找到下方关键位置，对应地址 0xacc4

```
[lib52pojie.so] [0x0acc4] [ 00 00 13 ca ] 0x4000acc4: eor x0, x0, x19 >>>   x0=0xf17a354ab377b6x0=0xf17a354ab377b6x19=0x546bc51c8e9bfa//x0=0xa511f0563dec4c

```

全局搜索该地址发现有 64 处引用，且在第 32-33 之间跨度比较大，因此判断 encrypt_result 的前 16 个字节是由前 32 处引用生成，后 16 字节由后 32 处引用生成

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiazIwG3DATcLs4ZATthMSIXb8mwTZze0cwvwhicNWgFlWjFrgroBiaF36g/640?wx_fmt=png)

从第一处引用位置往回推  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaFEIw9wRVrnTEXMGTCvrJWjeg3CsUtzvBu5jwr9AEUIvY4h1LcIicqpw/640?wx_fmt=png)

第一轮运算 ret 之后想 x30 返回地址为 0x8fec，因此可以判断跳转指令 b 的地址为 0x8fec-0x4=0x8fe8，向上查找 0x8fe8 地址  

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWianVnkCuqictTdKaPiawzvwpbXXz8rFSfKenp0AsJqL3pmlU6T5cygTMdA/640?wx_fmt=png)

因此每轮运算即为 0x8fe8~0x8fec 之间的大概 300 多行指令，上图中框出来的则为首轮运算的参数，通过对地址 0x8fd4 全局搜索，查看前 32 轮运算的所有入参  

第一次：

```
# 入参
[lib52pojie.so] [0x08fd4] [ e1 03 15 aa ] 0x40008fd4: mov x1, x21 >>>   x1=0x8d71d734x21=0x37561947//x1=0x37561947    
[lib52pojie.so] [0x08fd8] [ e2 03 16 aa ] 0x40008fd8: mov x2, x22 >>>   x2=0x48x22=0x20155a74//x2=0x20155a74        
[lib52pojie.so] [0x08fdc] [ e3 03 17 aa ] 0x40008fdc: mov x3, x23 >>>   x3=0x10x23=0x99813f01//x3=0x99813f01        
[lib52pojie.so] [0x08fe0] [ e4 03 18 aa ] 0x40008fe0: mov x4, x24 >>>   x4=0x40x24=0x81b236f5//x4=0x81b236f5        
[lib52pojie.so] [0x08fe4] [ e5 03 19 aa ] 0x40008fe4: mov x5, x25 >>>   x5=0x20x25=0x3c3b90e65e1f82//x5=0x3c3b90e65e1f82
# 返回值
[lib52pojie.so] [0x0acc4] [ 00 00 13 ca ] 0x4000acc4: eor x0, x0, x19 >>>   x0=0xb69f0624f6464bx0=0xb69f0624f6464bx19=0x37561947//x0=0xb69f0613a05f0c

```

第二次：

```
# 入参
[lib52pojie.so] [0x08fd4] [ e1 03 15 aa ] 0x40008fd4: mov x1, x21 >>>   x1=0x8d71d734x21=0x20155a74//x1=0x20155a74
[lib52pojie.so] [0x08fd8] [ e2 03 16 aa ] 0x40008fd8: mov x2, x22 >>>   x2=0x48x22=0x99813f01//x2=0x99813f01
[lib52pojie.so] [0x08fdc] [ e3 03 17 aa ] 0x40008fdc: mov x3, x23 >>>   x3=0x10x23=0x81b236f5//x3=0x81b236f5
[lib52pojie.so] [0x08fe0] [ e4 03 18 aa ] 0x40008fe0: mov x4, x24 >>>   x4=0x40x24=0xb69f0613a05f0c//x4=0xb69f0613a05f0c
[lib52pojie.so] [0x08fe4] [ e5 03 19 aa ] 0x40008fe4: mov x5, x25 >>>   x5=0x20x25=0x72c025ec20d5a2//x5=0x72c025ec20d5a2
# 返回值
[lib52pojie.so] [0x0acc4] [ 00 00 13 ca ] 0x4000acc4: eor x0, x0, x19 >>>   x0=0x7fbd3dd7591b0ex0=0x7fbd3dd7591b0ex19=0x20155a74//x0=0x7fbd3df74c417a

```

第三次：

```
# 入参
[lib52pojie.so] [0x08fd4] [ e1 03 15 aa ] 0x40008fd4: mov x1, x21 >>>   x1=0x8d71d734x21=0x99813f01//x1=0x99813f01
[lib52pojie.so] [0x08fd8] [ e2 03 16 aa ] 0x40008fd8: mov x2, x22 >>>   x2=0x48x22=0x81b236f5//x2=0x81b236f5
[lib52pojie.so] [0x08fdc] [ e3 03 17 aa ] 0x40008fdc: mov x3, x23 >>>   x3=0x10x23=0xb69f0613a05f0c//x3=0xb69f0613a05f0c
[lib52pojie.so] [0x08fe0] [ e4 03 18 aa ] 0x40008fe0: mov x4, x24 >>>   x4=0x40x24=0x7fbd3df74c417a//x4=0x7fbd3df74c417a
[lib52pojie.so] [0x08fe4] [ e5 03 19 aa ] 0x40008fe4: mov x5, x25 >>>   x5=0x20x25=0x4145992b8ee3ef//x5=0x4145992b8ee3ef
# 返回值
[lib52pojie.so] [0x0acc4] [ 00 00 13 ca ] 0x4000acc4: eor x0, x0, x19 >>>   x0=0x4e37555f6d7650x0=0x4e37555f6d7650x19=0x99813f01//x0=0x4e3755c6ec4951

```

至第 32 次计算结束后的值就可以得到最终结果 0xa511f0563dec4c 就可以作为上述最终赋值 56ec3d4c526764d7949fe1f201d9fe45 的起始数据

```
# 返回值
[lib52pojie.so] [0x0acc4] [ 00 00 13 ca ] 0x4000acc4: eor x0, x0, x19 >>>   x0=0xf17a354ab377b6x0=0xf17a354ab377b6x19=0x546bc51c8e9bfa//x0=0xa511f0563dec4c

```

再根据上述列出的几次运算入参和返回过程，得出运算规则如下图所示，其中参数 5 为固定 32 长度数组，每轮运算取出数组中对应索引的值

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaK59Fia0Ruhj5NQoIghzV9zGc9djPEibkfMH6h2R12hcaWzHic3b4BFEYA/640?wx_fmt=png)

如此我们只需要知道第一轮传入的参数，就能拿到经过 32 轮运算之后的结果了  

第一轮入参如下

```
[lib52pojie.so] [0x08fd4] [ e1 03 15 aa ] 0x40008fd4: mov x1, x21 >>>   x1=0x8d71d734x21=0x37561947//x1=0x37561947

```

全局搜索 0x37561947，找到关键位置

```
[lib52pojie.so] [0x08de8] [ 31 9e 68 d3 ] 0x40008de8: lsl x17, x17, #0x18 >>>   x17=0x37x17=0x37//x17=0x37000000
[lib52pojie.so] [0x08dec] [ ff 05 00 71 ] 0x40008dec: cmp w15, #1 >>>   w15=0x1//w15=0x1
[lib52pojie.so] [0x08df0] [ 51 1e 78 b3 ] 0x40008df0: bfi x17, x18, #8, #8 >>>   x17=0x37000000x18=0x19//x17=0x37001900
[lib52pojie.so] [0x08df4] [ 41 01 00 54 ] 0x40008df4: b.ne #0x40008e1c
[lib52pojie.so] [0x08df8] [ 80 9d 68 d3 ] 0x40008df8: lsl x0, x12, #0x18 >>>   x0=0xbffff0e0x12=0x20//x0=0x20000000
[lib52pojie.so] [0x08dfc] [ 32 42 0a aa ] 0x40008dfc: orr x18, x17, x10, lsl #16 >>>   x18=0x19x17=0x37001900x10=0x56//x18=0x37561900
[lib52pojie.so] [0x08e00] [ a0 1d 78 b3 ] 0x40008e00: bfi x0, x13, #8, #8 >>>   x0=0x20000000x13=0x5a//x0=0x20005a00
[lib52pojie.so] [0x08e04] [ 52 02 0b aa ] 0x40008e04: orr x18, x18, x11 >>>   x18=0x37561900x18=0x37561900x11=0x47//x18=0x37561947

```

发现是由 0x37,0x56,0x19,0x47 通过移位操作，直接搜索这些 hex 会有很多响应结果，暂且不在往上回溯，到现在我们都没关心过真正的入参`52pojieairadaira`，对应 hex `35,32,70,6f,6a,69,65,61,69,72,61,64,61,69,72,61`

全局搜索`0x35`，搜索到的第一个响应结果如下

```
[lib52pojie.so] [0x09b34] [ 69 69 6a 38 ] 0x40009b34: ldrb w9, [x11, x10] >>>   w9=0x0x11=0xbffff4a0x10=0x0//w9=0x35
[lib52pojie.so] [0x09b38] [ 8c 01 40 f9 ] 0x40009b38: ldr x12, [x12] >>>   x12=0xbffff2d0x12=0xbffff2d0//x12=0xbffff480
[lib52pojie.so] [0x09b3c] [ 08 01 09 4a ] 0x40009b3c: eor w8, w8, w9 >>>   w8=0x2w8=0x2w9=0x35//w8=0x37

```

可以看到在 0x9b3c 地址处让 0x35 ^ 0x2 得到 0x37，全局搜索 0x9b3c 地址

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiadLQDA0eDTJ7t2NLia5fOrMkFiaK8B1xPHnrEwPgSl9yaP2wWDDJj42pg/640?wx_fmt=png)

前 16 个结果的 w9 为入参`52pojieairadaira`对应的 hex，指令执行结束 w8 对应的结果为【'0x37', '0x19', '0x56', '0x47', '0x20', '0x5a', '0x15', '0x74', '0x99', '0x3f', '0x81', '0x1', '0x81', '0x36', '0xb2', '0xf5'】，与上述我们需求的第一轮运算入参的所有 16 个字节匹配  

再看匹配到的后 16 个结果中，w9 的 hex 组装为【0x0, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10】，w8 的参数组装为 0x56,0xec,0x3d,0x4c,0x52,0x67,0x64,0xd7,0x94,0x9f,0xe1,0xf2,0x1,0xd9,0xfe,0x45，正好对应前 32 轮的计算结果 56ec3d4c526764d7949fe1f201d9fe45

因此整个流程大概是

①入参 hex 异或 xor_key_table（【0x2, 0x2b, 0x26, 0x28, 0x4a, 0x33, 0x70, 0x15, 0xf0, 0x4d, 0xe0, 0x65, 0xe0, 0x5f, 0xc0, 0x94】），对应索引异或得到【0x37,0x19,0x56,0x47,0x20,0x5a,0x15,0x74,0x99,0x3f,0x81,0x01,0x81,0x36,0xb2,0xf5】

```
# 52pojieairadaira
hex_param = [0x35, 0x32, 0x70, 0x6f, 0x6a, 0x69, 0x65, 0x61, 0x69, 0x72, 0x61, 0x64, 0x61, 0x69, 0x72, 0x61]
xor_key = [0x2, 0x2b, 0x26, 0x28, 0x4a, 0x33, 0x70, 0x15, 0xf0, 0x4d, 0xe0, 0x65, 0xe0, 0x5f, 0xc0, 0x94]
real_param = []
for i in range(len(hex_param)):
   a = hex_param[i] ^ xor_key[i]
   real_param.append(a)

```

②将上述异或之后得到的 hex 移位拼接成【0x37561947，0x20155a74，0x99813f01，0x81b236f5】

```
params = []
for i in range(0, len(real_param), 4):
   param = (real_param[i] << 24) + (real_param[i + 2] << 16) + (real_param[i + 1] << 8) + real_param[i + 3]
   params.append(param)

```

③经过 32 轮运算（汇编转 c/python，无捷径）

```
hex_list = [0xd6, 0x90, 0xe9, 0xfe, 0xcc, 0xe1, 0x3d, 0xb7, 0x16, 0xb6, 0x14, 0xc2, 0x28, 0xfb, 0x2c, 0x05, 0x2b, 0x67,
           0x9a, 0x76, 0x2a, 0xbe, 0x04, 0xc3, 0xaa, 0x44, 0x13, 0x26, 0x49, 0x86, 0x06, 0x99, 0x9c, 0x42, 0x50, 0xf4,
           0x91, 0xef, 0x98, 0x7a, 0x33, 0x54, 0x0b, 0x43, 0xed, 0xcf, 0xac, 0x62, 0xe4, 0xb3, 0x1c, 0xa9, 0xc9, 0x08,
           0xe8, 0x95, 0x80, 0xdf, 0x94, 0xfa, 0x75, 0x8f, 0x3f, 0xa6, 0x47, 0x07, 0xa7, 0xfc, 0xf3, 0x73, 0x17, 0xba,
           0x83, 0x59, 0x3c, 0x19, 0xe6, 0x85, 0x4f, 0xa8, 0x68, 0x6b, 0x81, 0xb2, 0x71, 0x64, 0xda, 0x8b, 0xf8, 0xeb,
           0x0f, 0x4b, 0x70, 0x56, 0x9d, 0x35, 0x1e, 0x24, 0x0e, 0x5e, 0x63, 0x58, 0xd1, 0xa2, 0x25, 0x22, 0x7c, 0x3b,
           0x01, 0x21, 0x78, 0x87, 0xd4, 0x00, 0x46, 0x57, 0x9f, 0xd3, 0x27, 0x52, 0x4c, 0x36, 0x02, 0xe7, 0xa0, 0xc4,
           0xc8, 0x9e, 0xea, 0xbf, 0x8a, 0xd2, 0x40, 0xc7, 0x38, 0xb5, 0xa3, 0xf7, 0xf2, 0xce, 0xf9, 0x61, 0x15, 0xa1,
           0xe0, 0xae, 0x5d, 0xa4, 0x9b, 0x34, 0x1a, 0x55, 0xad, 0x93, 0x32, 0x30, 0xf5, 0x8c, 0xb1, 0xe3, 0x1d, 0xf6,
           0xe2, 0x2e, 0x82, 0x66, 0xca, 0x60, 0xc0, 0x29, 0x23, 0xab, 0x0d, 0x53, 0x4e, 0x6f, 0xd5, 0xdb, 0x37, 0x45,
           0xde, 0xfd, 0x8e, 0x2f, 0x03, 0xff, 0x6a, 0x72, 0x6d, 0x6c, 0x5b, 0x51, 0x8d, 0x1b, 0xaf, 0x92, 0xbb, 0xdd,
           0xbc, 0x7f, 0x11, 0xd9, 0x5c, 0x41, 0x1f, 0x10, 0x5a, 0xd8, 0x0a, 0xc1, 0x31, 0x88, 0xa5, 0xcd, 0x7b, 0xbd,
           0x2d, 0x74, 0xd0, 0x12, 0xb8, 0xe5, 0xb4, 0xb0, 0x89, 0x69, 0x97, 0x4a, 0x0c, 0x96, 0x77, 0x7e, 0x65, 0xb9,
           0xf1, 0x09, 0xc5, 0x6e, 0xc6, 0x84, 0x18, 0xf0, 0x7d, 0xec, 0x3a, 0xdc, 0x4d, 0x20, 0x79, 0xee, 0x5f, 0x3e,
           0xd7, 0xcb, 0x39, 0x48]
hex_params = [0x3c3b90e65e1f82, 0x72c025ec20d5a2, 0x4145992b8ee3ef, 0x69cff9c3113c5e, 0x73a0bdc7e35bb5,
             0x6587f6717f5417,
             0x33f15055cc7021, 0x54d89634fd983e, 0x7fb7780a7fadbb, 0x4f4f92ef4f948e, 0x5f72ac76224645,
             0x64fedb2ae2d1ac,
             0x5aa8969e2ac4bc, 0x680fea99ef2053, 0x6c1e634f07843e, 0x677673b06c935, 0x3d1b6be4557920, 0x1b04f5e1299f9c,
             0x326a513916a44c, 0x1ff10831d1d12e, 0x72fb7bdb5a88e, 0x6c3867e930366f, 0x4f884102f7796b, 0x7005c6149330c,
             0x284c586c9b290, 0x19ca128d08d882, 0x544b18a95a6bde, 0x6a14600a6179ed, 0x1d4be54074b82b, 0x7581763c59130d,
             0x4bb3c753b8931b, 0x668bd94d4abaa7]
def bfi(dst, src, length, width):
   ls = 0xFFFFFFFF >> (32 - width)
   lsb = length
   ls = ~((ls << lsb) & 0xff)
   if ls < -1:
       # 针对特例
       ls = -1
   dst = dst & ls
   dst = dst | (src & ls) << lsb
   return dst
all_encrypt_hex = []
def encrypt(*args):
   kwargs = args[0]
   x1 = kwargs[0]  # 0x37561947
   x2 = kwargs[1]  # 0x20155a74
   x3 = kwargs[2]  # 0x99813f01
   x4 = kwargs[3]  # 0x81b236f5
   x5 = kwargs[4]  # 0x3c3b90e65e1f82 # 0x08fe4
   x29 = [0] * 1024
   xx19 = x1  # 0x0abd4
   x9 = x3 ^ x2  # 0x0ac14
   x9 = x9 ^ x4  # 0x0ac2c
   x21 = x9 ^ x5  # 0x0ac30
   x1 = x21  # 0x0ac90
   x29[-0x60] = x1
   x8 = x21
   x21 = x8 >> 0x8  # 0x0ad80
   x22 = x8 >> 0x10  # 0x0ad84
   x9 = x8
   x25 = x9 >> 0x18  # 0x0ad98
   x1 = x25
   x10 = x1 & 0xff
   x0 = hex_list[x10]  # 0xb4
   x29[-0x68] = x0  # 0x0ab94
   x1 = x21 & 0xffffffff  # 0x0ade8
   x10 = x1 & 0xff
   x0 = hex_list[x10]  # 0xe6
   x29[-0x70] = x0
   x1 = x22 & 0xffffffff
   x10 = x1 & 0xff
   x0 = hex_list[x10]  # 0x4c
   x27 = x0
   x1 = x29[-0x60]
   x10 = x1 & 0xff  # 0x2
   x0 = hex_list[x10]
   x9 = x29[-0x70]
   x12 = x29[-0x68]
   x10 = x27 & 0xff  # 0x0afb4
   x8 = x12 & 0xff
   x9 = x9 & 0xff
   x8 = x8 << 0x18
   # bfi x8, x9, #8, #8
   x8 = bfi(x8, x9, 0x8, 0x8)
   x8 = bfi(x8, x10, 0x10, 0x8)
   x9 = x8 >> 0xe
   x10 = x8 >> 0x16
   x11 = x8 >> 8
   x8 = x8 + x0
   x12 = x12 >> 6
   # bfi x10, x8, #0xa, #0x20
   x10 = bfi(x10, x8, 0xa, 0x20)
   x12 = bfi(x12, x8, 0x2, 0x20)
   x9 = bfi(x9, x8, 0x12, 0x20)
   x11 = bfi(x11, x8, 0x18, 0x20)  # 0x0afec
   x8 = x10 ^ x8
   x8 = x8 ^ x9
   x8 = x8 ^ x11
   x0 = x8 ^ x12
   x0 = x0 ^ xx19  # 0x0acc4
   # print(hex(x0))
   all_encrypt_hex.append(x0)
   result_params = [x2, x3, x4, x0]
   return result_params
for i in range(32):
   params.append(hex_params[i])
   params = encrypt(params)

```

④将运算结果取低位移位运算（0x9100）

```
encrypt_hex = [0] * 16
x8 = all_encrypt_hex[-1]
w8 = x8 & 0xffffffff
x9 = x8 >> 0x18
encrypt_hex[0] = x9 & 0xff
x9 = x8 >> 0x8
encrypt_hex[3] = w8 & 0xff
x8 = x8 >> 0x10
w8 = x8 & 0xffffffff
encrypt_hex[1] = x9 & 0xff
encrypt_hex[2] = w8 & 0xff
x8 = all_encrypt_hex[-2]
w9 = x8 >> 0x18
encrypt_hex[4] = w9 & 0xff
x9 = all_encrypt_hex[-3]
x11 = x8 >> 0x10
x12 = x8 >> 0x8
x13 = x9 >> 0x18
w8 = x8 & 0xffffffff
encrypt_hex[7] = w8 & 0xff
w12 = x12 & 0xffffffff
encrypt_hex[5] = w12 & 0xff
w11 = x11 & 0xffffffff
encrypt_hex[6] = w11 & 0xff
w13 = x13 & 0xffffffff
encrypt_hex[8] = w13 & 0xff
w9 = x9 & 0xffffffff
encrypt_hex[0xb] = w9 & 0xff
x8 = x9 >> 0x8
x9 = x9 >> 0x10
w8 = x8 & 0xfffffff
w9 = x9 & 0xfffffff
encrypt_hex[0x9] = w8 & 0xff
encrypt_hex[0xa] = w9 & 0xff
x8 = all_encrypt_hex[-4]
x9 = x8 >> 0x18
w9 = x9 & 0xffffffff
encrypt_hex[0xc] = w9 & 0xff
w8 = x8 & 0xffffffff
encrypt_hex[0xf] = w8 & 0xff
x9 = x8 >> 8
x8 = x8 >> 0x10
w9 = x9 & 0xffffffff
w8 = x8 & 0xffffffff
encrypt_hex[0xd] = w9 & 0xff
encrypt_hex[0xe] = w8 & 0xff
print(",".join([hex(i) for i in encrypt_hex]))

```

以上前 32 轮运算结束，0xbffff480 处的前 16 个字节也已得到

⑤将上述结果【0x56,0xec,0x3d,0x4c,0x52,0x67,0x64,0xd7,0x94,0x9f,0xe1,0xf2,0x1,0xd9,0xfe,0x45】作为新的异或 table 与【0x00,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10,0x10】对应位置异或

```
hex_param_01 = [0x0, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10, 0x10]
xor_key_01 = [0x56, 0xec, 0x3d, 0x4c, 0x52, 0x67, 0x64, 0xd7, 0x94, 0x9f, 0xe1, 0xf2, 0x1, 0xd9, 0xfe, 0x45]
real_param_01 = []
for i in range(len(hex_param_01)):
   a = hex_param_01[i] ^ xor_key_01[i]
   real_param_01.append(a)

```

重复上述②③④步，得到第 64 轮结束后的 16 字节为【0xce,0x36,0x1,0xc8,0xd2,0x82,0x12,0x98,0x84,0xd8,0xbf,0x98,0x50,0x34,0x45,0xe】，正好对应 0xbffff480 处的后 16 字节

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaGruPMP1ibRRaF5yfUULNSIcKYbiaJjmqqbUj2thZkmaV0CbZ4yo7rmWA/640?wx_fmt=png)

至此，整个加密流程则已经完整分析结束，得到最终生成的结果：56ec3d4c526764d7949fe1f201d9fe45ce3601c8d282129884d8bf985034450e  

根据题目要求，我们需要传入明文让这个最终的结果等于：6e6649305baf80c49b1b063c0500c80346ccfd42b3063ae7312b52a21cd334d8，这样才能拿到正确的 flag 并最终让函数执行结果返回 true

现在加密函数分析出来了，但是要怎么拿到对应的解密函数呢

整理下上述过程，为了使结果为 6e6649305baf80c49b1b063c0500c80346ccfd42b3063ae7312b52a21cd334d8，我们手中已经掌握的数据有

共 64 轮，以每 32 轮运算分为上下两部分

```
# 上部分
传参明文：未知
计算传参：未知
异或keyTable（常量Table）：022b26284a337015f04de065e05fc094
最终结果：6e6649305baf80c49b1b063c0500c803
# 下部分
传参明文为：00101010101010101010101010101010
异或keyTable为（此为上部分最终计算的结果）：6e664930 5baf80c4 9b1b063c 0500c803
计算传参：0x6e597620, 0x4b90bfd4, 0x8b160b2c, 0x15d81013
最终计算结果：46ccfd42b3063ae7312b52a21cd334d8

```

那么如果我们根据下部分计算过程成功逆推出来计算传参，是不是就能拿到下部分传参的明文了呢

首先说明下如果通过计算过程的传参推出明文

```
0x6e ^ 0x6e = 0x00
0x59 ^ 0x49 = 0x10
0x76 ^ 0x66 = 0x10
0x20 ^ 0x30 = 0x10
0x5b ^ 0x4b = 0x10
0x90 ^ 0x80 = 0x10
0xbf ^ 0xaf = 0x10
0xd4 ^ 0xc4 = 0x10
...
0x15 ^ 0x05 = 0x10
0xd8 ^ 0xc8 = 0x10
0x10 ^ 0x00 = 0x10
0x13 ^ 0x03 = 0x10

```

回过头我们再看下具体的 encrypt 函数逻辑

```
def encrypt(*args):
   kwargs = args[0]
   x1 = kwargs[0]  # 未知
   x2 = kwargs[1]  # 0xe8c7201c34d3d8
   x3 = kwargs[2]  # 0x42da1431522ba2
   x4 = kwargs[3]  # 0xfb05dab33a06e7
   x5 = kwargs[4]  # 0x668bd94d4abaa7 -> 常量table最后一个元素
   x29 = [0] * 1024
   xx19 = x1
   x9 = x3 ^ x2
   x9 = x9 ^ x4
   x21 = x9 ^ x5
   x1 = x21
   x29[-0x60] = x1
   x8 = x21
   x21 = x8 >> 0x8
   x22 = x8 >> 0x10
   x9 = x8
   x25 = x9 >> 0x18
   x1 = x25
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x29[-0x68] = x0
   x1 = x21 & 0xffffffff
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x29[-0x70] = x0
   x1 = x22 & 0xffffffff
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x27 = x0
   x1 = x29[-0x60]
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x9 = x29[-0x70]
   x12 = x29[-0x68]
   x10 = x27 & 0xff
   x8 = x12 & 0xff
   x9 = x9 & 0xff
   x8 = x8 << 0x18
   x8 = bfi(x8, x9, 0x8, 0x8)
   x8 = bfi(x8, x10, 0x10, 0x8)
   x9 = x8 >> 0xe
   x10 = x8 >> 0x16
   x11 = x8 >> 8
   x8 = x8 + x0
   x12 = x12 >> 6
   x10 = bfi(x10, x8, 0xa, 0x20)
   x12 = bfi(x12, x8, 0x2, 0x20)
   x9 = bfi(x9, x8, 0x12, 0x20)
   x11 = bfi(x11, x8, 0x18, 0x20)
   x8 = x10 ^ x8
   x8 = x8 ^ x9
   x8 = x8 ^ x11
   x0 = x8 ^ x12# x0 = 0x8A26C2E1034B06
   x0 = x0 ^ xx19# x0 = x0 ^ xx19 = 0x9c426546fdcc42 -> xx19 = 0x8A26C2E1034B06 ^ 0x9c426546fdcc42 = 0x1664a7a7fe8744 -> x1 = 0x1664a7a7fe8744
   print(hex(x0))

```

最终我们是得到一个 x0 的值，即是每轮计算的结果，这个结果又会与 xx19 进行异或，而 xx19 是从 x1 赋值得来的，x1 即是我们传入的第一个参数，且在后续过程中并没有参与其他的运算，经过最后一步我们是能拿到最后一轮 x0 的值，现在想拿到最后一轮 xx19 的值，将 x2, x3, x4, x5 这四个值正常传入，在最后一步 x0 = x8 ^ x12 仍能拿到正确计算的的 x0 值，该值再与 xx19 进行异或即可得到传入的 x1 的值，以上述最后一轮入参和出参计算过程，代码注释为例，最终推出 x1 的值为 0x1664a7a7fe8744，正向加密过程输出可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaOYt1yfZmEenzrrdzLagmBfiabmpJVO0freX89sHLatnZOKyGs5IuibZQ/640?wx_fmt=png)

是能够匹配上最后一轮传入的第一个参数，另外根据图中还能得到的另一个信息就是下部分计算的后 16 字节结果是能够 encrypt 最后一轮拿到的（同上述第四步），如下规则  

```
0xe8c7201c34d3d8 & 0xffffffff -> 0x1c34d3d8 -> 0x1c,0xd3,0x34,0xd8 -> 12-15字节
0x42da1431522ba2 & 0xffffffff -> 0x31522ba2 -> 0x31,0x2b,0x52,0xa2 -> 8-11字节
0xfb05dab33a06e7 & 0xffffffff -> 0xb33a06e7 -> 0xb3,0x06,0x3a,0xe7 -> 4-7字节
0x9c426546fdcc42 & 0xffffffff -> 0x46fdcc42 -> 0x46,0xcc,0xfd,0x42 -> 0-3字节

```

另外我们考虑到另一个问题就是，后续所有计算的值都是高位，但是最终我们拿到的 x1 一定是低 8 位的值，因此后续计算都以低八位的数据来传入参数

现在知道了 decrypt 的方法，那么据此方法再执行 32 轮不就能够推出上述加密过程下部分的计算传参了吗

```
def decrypt(*args):
   kwargs = args[0]
   x1 = kwargs[0]
   x2 = kwargs[1]
   x3 = kwargs[2]
   x4 = kwargs[3]
   x5 = kwargs[4]
   x29 = [0] * 1024
   xx19 = x1
   x9 = x3 ^ x2
   x9 = x9 ^ x4
   x21 = x9 ^ x5
   x1 = x21
   x29[-0x60] = x1
   x8 = x21
   x21 = x8 >> 0x8
   x22 = x8 >> 0x10
   x9 = x8
   x25 = x9 >> 0x18
   x1 = x25
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x29[-0x68] = x0
   x1 = x21 & 0xffffffff
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x29[-0x70] = x0
   x1 = x22 & 0xffffffff
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x27 = x0
   x1 = x29[-0x60]
   x10 = x1 & 0xff
   x0 = hex_list[x10]
   x9 = x29[-0x70]
   x12 = x29[-0x68]
   x10 = x27 & 0xff
   x8 = x12 & 0xff
   x9 = x9 & 0xff
   x8 = x8 << 0x18
   x8 = bfi(x8, x9, 0x8, 0x8)
   x8 = bfi(x8, x10, 0x10, 0x8)
   x9 = x8 >> 0xe
   x10 = x8 >> 0x16
   x11 = x8 >> 8
   x8 = x8 + x0
   x12 = x12 >> 6
   x10 = bfi(x10, x8, 0xa, 0x20)
   x12 = bfi(x12, x8, 0x2, 0x20)
   x9 = bfi(x9, x8, 0x12, 0x20)
   x11 = bfi(x11, x8, 0x18, 0x20)
   x8 = x10 ^ x8
   x8 = x8 ^ x9
   x8 = x8 ^ x11
   x0 = x8 ^ x12
   x0 = x0 ^ xx19
   return x0
# 46ccfd42
final_result = 0x46fdcc42
# b3063ae7
final_list1 = 0x1c34d3d8
# 312b52a2
final_list2 = 0x31522ba2
# 1cd334d8
final_list3 = 0xb33a06e7
result = [final_result, final_list1, final_list2, final_list3, hex_params[-1]]
x0 = decrypt(result)
result = [final_list3, x0, final_list1, final_list2, hex_params[-2]]
x1 = decrypt(result)
result = [final_list2, x1, x0, final_list1, hex_params[-3]]
x2 = decrypt(result)
result = [final_list1, x2, x1, x0, hex_params[-4]]
x3 = decrypt(result)
result = [x0, x3, x2, x1, hex_params[-5]]
x4 = decrypt(result)
result = [x1, x4, x3, x2, hex_params[-6]]
x5 = decrypt(result)
result = [x2, x5, x4, x3, hex_params[-7]]
x6 = decrypt(result)
result = [x3, x6, x5, x4, hex_params[-8]]
x7 = decrypt(result)
result = [x4, x7, x6, x5, hex_params[-9]]
x8 = decrypt(result)
result = [x5, x8, x7, x6, hex_params[-10]]
x9 = decrypt(result)
result = [x6, x9, x8, x7, hex_params[-11]]
x10 = decrypt(result)
result = [x7, x10, x9, x8, hex_params[-12]]
x11 = decrypt(result)
result = [x8, x11, x10, x9, hex_params[-13]]
x12 = decrypt(result)
result = [x9, x12, x11, x10, hex_params[-14]]
x13 = decrypt(result)
result = [x10, x13, x12, x11, hex_params[-15]]
x14 = decrypt(result)
result = [x11, x14, x13, x12, hex_params[-16]]
x15 = decrypt(result)
result = [x12, x15, x14, x13, hex_params[-17]]
x16 = decrypt(result)
result = [x13, x16, x15, x14, hex_params[-18]]
x17 = decrypt(result)
result = [x14, x17, x16, x15, hex_params[-19]]
x18 = decrypt(result)
result = [x15, x18, x17, x16, hex_params[-20]]
x19 = decrypt(result)
result = [x16, x19, x18, x17, hex_params[-21]]
x20 = decrypt(result)
result = [x17, x20, x19, x18, hex_params[-22]]
x21 = decrypt(result)
result = [x18, x21, x20, x19, hex_params[-23]]
x22 = decrypt(result)
result = [x19, x22, x21, x20, hex_params[-24]]
x23 = decrypt(result)
result = [x20, x23, x22, x21, hex_params[-25]]
x24 = decrypt(result)
result = [x21, x24, x23, x22, hex_params[-26]]
x25 = decrypt(result)
result = [x22, x25, x24, x23, hex_params[-27]]
x26 = decrypt(result)
result = [x23, x26, x25, x24, hex_params[-28]]
x27 = decrypt(result)
result = [x24, x27, x26, x25, hex_params[-29]]
x28 = decrypt(result)
result = [x25, x28, x27, x26, hex_params[-30]]
x29 = decrypt(result)
result = [x26, x29, x28, x27, hex_params[-31]]
x30 = decrypt(result)
result = [x27, x30, x29, x28, hex_params[-32]]
x31 = decrypt(result)
print(hex(x31), hex(x30), hex(x29), hex(x28))

```

得出结果【0xe8c7206e597620， 0x42da144b90bfd4， 0xfb05da8b160b2c， 0x9c426515d81013】

再分别取每个值的低 8 位：【0x6e597620， 0x4b90bfd4， 0x8b160b2c， 0x15d81013】

是不是就与上面下部分计算最终想要拿到的计算传参一样了呢，整个 decrypt 流程就完整解析出来了

把上述 decrypt 传参修改为【0x6e496630，0x05c80003，0x9b061b3c，0x5b80afc4】

计算得到结果【0x4e12ab59517c7f， 0xc75c9e64420645， 0xa60707c0890756， 0x5457c9ce8e3cc9】

分别取低八位：【0x59517c7f，0x64420645，0xc0890756，0xce8e3cc9】，此即为上部分计算的计算传参

再由上部分的异或 keyTable【0x2, 0x2b, 0x26, 0x28, 0x4a, 0x33, 0x70, 0x15, 0xf0, 0x4d, 0xe0, 0x65, 0xe0, 0x5f, 0xc0, 0x94】

最终拿到上部分计算传参明文及 flag

```
"""
0x59 ^ 0x2 = [
0x7c ^ 0x2b = W
0x51 ^ 0x26 = w
0x7f ^ 0x28 = W
59517c7f
0x64 ^ 0x4a = .
0x06 ^ 0x33 = 5
0x42 ^ 0x70 = 2
0x45 ^ 0x15 = P
64420645
0xc0 ^ 0xf0 = 0
0x07 ^ 0x4d = J
0x89 ^ 0xe0 = i
0x56 ^ 0x65 = 3
c0890756
0xce ^ 0xe0 = .
0x3c ^ 0x5f = c
0x8e ^ 0xc0 = N
0xc9 ^ 0x94 = ]
ce8e3cc9
flag: [WwW.52P0Ji3.cN]
"""

```

将 flag 放置 unidbg 种作为参数传入，返回为 true

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaVS68ztV6fcuWcF3xD4Eic7Y1OrgV1Jr70ia81KcoIre7pp1kcTiag9gKA/640?wx_fmt=png)

写在最后，许久没更新文章了，本篇篇幅较多，连贯性可能不够，感兴趣的小伙伴可自行根据文章流程尝试获取 flag，后台回复 "查查" 获取资源包（本篇文章的 md 文档，观感较好，.py，.java 及 trace 文件等）。  

最后的最后

**查企业，查信用就上企查查~**

![](https://mmbiz.qpic.cn/mmbiz_png/YY4a1FLD4yiaRqhUn1B4oa6IGz5wP2HWiaCIwSqddwdSM8ya4MiaTVqwN72zVlb8lFFDGVxcsicbOos7Hkkfzt6icibQ/640?wx_fmt=png)