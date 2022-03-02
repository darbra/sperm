> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIxOTQ1OTA0Ng==&mid=2247484164&idx=1&sn=dab3441a211bc568bcc8a94a34857e1f&chksm=97dbbf8da0ac369b34ff82796b00273fb2f9e397b31c98d4a1ecfc2aac6f7bdc2d4a305cda35&mpshare=1&scene=1&srcid=0302sQN7FCkfK824hr7fDsXH&sharer_sharetime=1646203681892&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

> 仅供学习研究 。请勿用于非法用途，本人将不承担任何法律责任。

前言
--

*   app 某某咖啡
    
*   v4.4.0
    

mitmproxy 抓包
------------

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicjccdL8L6WkeY7vEMaLCr1qqmXxYNVG2hZ8tWGuwfPHibNiahSpv0USvg/640?wx_fmt=png)

java 分析
-------

定位到 CryptoHelper 类的名为 md5_crypt 的 native 静态方法。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicqUbian9FmyyvYBHN3T4SJ9NiaHicSicklDq0hkqqicNZBuTp5F1uVx6WDxQ/640?wx_fmt=png)

  

frida hook
----------

脚本如下所示

```
function hook() {
    Java.perform(function() {
    var CryptoHelper = Java.use('com.l*****e.safeboxlib.CryptoHelper');
    CryptoHelper.md5_crypt.implementation = function (x, y) {
        console.log('md5_crypt_x', bytes2str(x));
        console.log('md5_crypt_y', y);
        var result = this.md5_crypt(x, y);
        console.warn('md5_crypt_ret', bytes2str(result));
        return result;
        };
  }
  }


```

我们可以看到，hook 的结果和抓包结果一致

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpiczCo3k1WFSYeef3I3fgoyicnAtoWejRBP2tk2OjCtSV0mo5UvYDQlmFA/640?wx_fmt=png)

  

so 分析
-----

使用 lasting-yang 的脚本 hook_RegisterNatives

脚本地址：https://github.com/lasting-yang/frida_hook_libart

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicZRkq410XCxicOhDf5L8v8O5MwBlxPWYCIHfOEibh2caJdZCk17hApdgw/640?wx_fmt=png)

  

使用开源的 cutter 到 so 去一探究竟，我们搜索上图中的偏移 0x1a981，来到 android_native_md5 函数。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpiclwgufef4e9yAbO66DcRMPa0PrND9Meh91VOswzWJJefaic4EEqM56AA/640?wx_fmt=png)

  

经过一番分析，应该是 md5 加密完之后，还有一个 bytesToInt 的逻辑。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpic850QMsGxmicFMuEbrKnmlQKcttaf9OJ2LIRqGMMOEDKaI5fX639yOuA/640?wx_fmt=png)

  

unidbg
------

去年的文章里用 frida 就能搞定了，这次我们用 lilac、qinless、qxp 等大佬极力推荐的神器 unidbg 进行辅助分析。

首先是搭建架子

```
package com.*;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.linux.android.dvm.array.IntArray;
import com.github.unidbg.linux.android.dvm.wrapper.DvmInteger;
import com.github.unidbg.memory.Memory;

import java.io.File;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

public class Md5Crypt440 extends AbstractJni {
    private final AndroidEmulator emulator;
    private final Module module;
    private final VM vm;

    public String apkPath = "/Users/darbra/Downloads/apk/com.*/temp.apk";
    public String soPath2 = "/Users/darbra/Downloads/apk/com.*/lib/armeabi-v7a/libcryptoDD.so";

    Md5Crypt440() {
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName("com.*").build();
        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));
        vm = emulator.createDalvikVM(new File(apkPath));
        vm.setVerbose(true);
        vm.setJni(this);
        DalvikModule dm2 = vm.loadLibrary(new File(soPath2), true);
        dm2.callJNI_OnLoad(emulator);
        module = dm2.getModule();
    }

    public void call() {
        String methodId = "md5_crypt([BI)[B";
        DvmClass SigEntity = vm.resolveClass("com/luckincoffee/safeboxlib/CryptoHelper");
        String fakeInput1 = "cid=210101;q=j***=;uid=***";
        ByteArray inputByteArray1 = new ByteArray(vm, fakeInput1.getBytes(StandardCharsets.UTF_8));
        DvmObject ret = SigEntity.callStaticJniMethodObject(
                emulator, methodId,
                inputByteArray1,
                1
        );
        byte [] strByte = (byte[]) ret.getValue();
        String strString = new String(strByte);
        System.out.println("callObject执行结果:"+ strString);
    }

    public static void main(String[] args) {
        Md5Crypt440 getSig = new Md5Crypt440();
        getSig.call();
        getSig.destroy();
    }

    private void destroy() {
        try {
            emulator.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}



```

值得注意的是，这个 app 不用补任何环境，非常轻松地就运行出了结果。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpic9bNKxxFpZ0EQ6VLKPT2GcHz9ficdbkPCQgrBkNjljLhGIxpmicj3paVg/640?wx_fmt=png)

  

结果和抓包、frida 完全一致。

我们在 md5 函数打个断点。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicq13rMv8DMjy2rUIbKhFY0AvypXt4bOxaeiavy519N8DM1Z5yDeHjPIw/640?wx_fmt=png)

```
public void HookByConsoleDebugger(){
    Debugger debugger = emulator.attach();
    debugger.addBreakPoint(module.base+0x13E3C);
}


```

  
  

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicW9iaUZoVjarou5mHG56Hy7pj8qiapAQJDZ09eft7oKuaicIVlzJFIU2kg/640?wx_fmt=png)

输入 mr0，我们可以看到第一个参数就是明文，但不够完整。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicBCgITQmoibrLF7ianKHamPThz18PoaRfP9g2WcjiaE3fGGFuLJEAl7icdA/640?wx_fmt=png)

  

接着输入 mr0 0x100，后面可以跟的是大小。我们欣喜地发现，之前的那坨明文后面加了个 salt 值，d******9

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpiciaeiaQ5Vpvo7VMicG4CicZR1hSNn7Ds2BYPgXVxMibqjT6ZSzMMoJpoErng/640?wx_fmt=png)

  

参数 2，r1=0xef=239，我们去 cyberchef 看看这个明文长度是否为 239

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicIKpTN1BuXD6VBx36418grWJXPZDndO8XKpxsZia0lEnHVByhWsmAMFg/640?wx_fmt=png)

  

果不其然是的。

接着输入 mr2，查看第三个参数。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicrwU6C0lviaAAaEQh478y7Uz1QVjsSNgnGe3p0OBmMcZDBvVYLQAHdSw/640?wx_fmt=png)

  

看情况参数 3 不出重大意外是 buffer，所以我们需要 hook 它的返回值。在这里我们先记下 r2 的地址 --->>> 0xbffff5d8。

我们使用 blr 命令用于在函数返回时设置一个一次性断点，然后 c 运行函数，它在返回处断下来。

接着输入刚刚记录下来的 m0xbffff5d8

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1acjmOUUvfhcLB6de0OCba2n84qxWpT7NO89jsib4bVGLibwNW3TWYSOxYCl1iar6GL0ahdNfxXJGjCg/640?wx_fmt=png)

  

我们去 cyberchef 进行验证，完全正确！

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1awrIic90ricvhRkstywCbTpicibEoicicuu5cFk4BL0znibUqFbD54whFmqbCokXtwhBiaib04icePuDSIbn9A/640?wx_fmt=png)

  

md5 步骤解决了，接着查看 bytesToInt。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1acjmOUUvfhcLB6de0OCba2TZCPPyxnzpJSAN3qdXpwWxU6hWl85FGxlCd83bmIL2y2gtIZoBpugA/640?wx_fmt=png)

  

我们在 0x13924 那里进行 hook 操作

```
public void hookBytesToInt() {
        IHookZz hookZz = HookZz.getInstance(emulator);

        hookZz.wrap(module.base + 0x13924 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Inspector.inspect(ctx.getR0Pointer().getByteArray(0, 0x10), "参数1");
                System.out.println("参数2->>" + ctx.getR1Int());
            };
            @Override
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                System.out.println("返回值->>" + ctx.getR0Int());
            }

        });
    }


```

我们看一下输出结果了，四个输出值都能对应上最后的 sign 结果。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1acjmOUUvfhcLB6de0OCba29YghiaA0PWIXmASUOicKMIgOLY6kLJl207b8zFnDicHCY9qfvcRPxCTfA/640?wx_fmt=png)

  

其中的正负号处理应该是对应的如下逻辑

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1acjmOUUvfhcLB6de0OCba2rK63Wl0ibewXSvHz4eV5ZELhO5fkJSkiaJxXN7WBPMWibibeEW9VmtOv3A/640?wx_fmt=png)

  

我们写个小脚本还原下，与之前的抓包、frida、unidbg 都一致，大功告成。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1acjmOUUvfhcLB6de0OCba2T5AuxrMzyOuXw7EZBnHfG3nuTXKO9HoiayBibqDKZwp8MorOaKwW4T5Q/640?wx_fmt=png)

  

在本人 github.com/darbra/sign 有更多的一些思路交流，如果对朋友们有所帮助，不甚欣喜。