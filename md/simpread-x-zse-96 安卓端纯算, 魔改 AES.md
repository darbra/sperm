> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SaOL71YL-pi9pXc5hUzMjg)

两天前发了一个 x-zse-96 的文章, 当时遇到了点问题, 只分析到了最后一个白盒 AES 函数里面, 并且当时用 dfa 攻击还原出了秘钥, IV 也确定了, 但是加密结果不对, 本来打算把下文鸽掉的, 因为当时 unidbg 没跑起来, 用 frida 去 hook 白盒 AES 中的每一行汇编有点麻烦, 没有 unidbg 方便. 后来小白大佬说 unidbg 可以跑通, 并把还原算法的任务交给了我. 我拿着小白提供的 unidbg 代码, 开始了算法还原之旅. 还原算法, 篇幅有点长, 会介绍 AES 的 10 轮加密以及如何从汇编代码和伪 c 代码中还原出魔改的 AES, 由于 so 是从内存中 dump 下来的, 反编译出来的伪代码准确性很差, 主要看的就是汇编. 严格来说, 这个 AES 魔改的已经完全不能算是 AES 了, 但是他 c 中的函数名称还是叫 wb_laes_encrypt, 白盒也不算.

unidbg 代码
=========

```
package com.zh2;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.backend.Unicorn2Factory;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.linux.file.ByteArrayFileIO;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;
import keystone.Keystone;
import keystone.KeystoneArchitecture;
import keystone.KeystoneEncoded;
import keystone.KeystoneMode;
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
public class zh extends AbstractJni implements IOResolver {
    private final AndroidEmulator emulator;
    private final DvmClass CryptoTool;
    private final VM vm;
    private final Module module;
    public zh() throws IOException {
        emulator = AndroidEmulatorBuilder
                .for32Bit()
                .addBackendFactory(new Unicorn2Factory(true))
                .setProcessName("com.zhihu.android")
                .build();
        Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));
        vm = emulator.createDalvikVM(new File("unidbg-android/apks/zh2/zh9.29.0.apk"));
        vm.setJni(this);
        vm.setVerbose(true);
        emulator.getSyscallHandler().addIOResolver(this);
        DalvikModule dm2 = vm.loadLibrary("bangcle_crypto_tool", true);
        module = dm2.getModule();
        CryptoTool = vm.resolveClass("com.bangcle.CryptoTool");
        dm2.callJNI_OnLoad(emulator);
    }
    public static String bytesToHex(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (byte b : bytes) {
            int unsignedInt = b & 0xff;
            String hex = Integer.toHexString(unsignedInt);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }
    public void x_zse_96_encrypt() throws IOException {
        List<Object> list = new ArrayList<>(5);
        list.add(vm.getJNIEnv());
        list.add(0);
        byte[] byy1 = {-118,-125,-125,41,41,34,-113,-115,42,35,34,42,-125,-128,40,-125,-125,33,-126,41,40,-118,-117,35,-125,-128,-117,35,42,-115,35,-113};
        byte[] byy2 = {-103,48,58,58,50,52,58,57,-110,-110,58,59,58,-103,-110,-110};
        ByteArray arr1 = new ByteArray(vm,byy1);
        list.add(vm.addLocalObject(arr1));
        list.add(vm.addLocalObject(new StringObject(vm, "541a3a5896fbefd351917c8251328a236a7efbf27d0fad8283ef59ef07aa386dbb2b1fcbba167135d575877ba0205a02f0aac2d31957bc7f028ed5888d4bbe69ed6768efc15ab703dc0f406b301845a0a64cf3c427c82870053bd7ba6721649c3a9aca8c3c31710a6be5ce71e4686842732d9314d6898cc3fdca075db46d1ccf3a7f9b20615f4a303c5235bd02c5cdc791eb123b9d9f7e72e954de3bcbf7d314064a1eced78d13679d040dd4080640d18c37bbde")));
        ByteArray arr2 = new ByteArray(vm,byy2);
        list.add(vm.addLocalObject(arr2));
        Number number = module.callFunction(emulator, 0xa708, list.toArray());
        ByteArray resultArr = vm.getObject(number.intValue());
        System.out.println("result:"+ bytesToHex(resultArr.getValue()));
    }
    public static void main(String[] args) throws IOException {
        zh zh = new zh();
        zh.patchInstruction();
        zh.x_zse_96_encrypt();
    }
    private void patchInstruction() {
        try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {
            KeystoneEncoded assemble = keystone.assemble("nop;nop");
            byte[] machineCode = assemble.getMachineCode();
            emulator.getMemory().pointer(module.base + 0x8EBC).write(0,machineCode,0,machineCode.length);
            KeystoneEncoded encoded = keystone.assemble("mov r3, 1");
            byte[] patchCode = encoded.getMachineCode();
            emulator.getMemory().pointer(module.base + 0x4e70).write(0, patchCode, 0, patchCode.length);
            KeystoneEncoded encoded1 = keystone.assemble("mov r3, 1");
            byte[] patchCode1 = encoded1.getMachineCode();
            emulator.getMemory().pointer(module.base + 0x4e84).write(0, patchCode1, 0, patchCode1.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    @Override
    public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature) {
            case "com/secneo/apkwrapper/H->PKGNAME:Ljava/lang/String;": {
                String packageName = vm.getPackageName();
                if (packageName != null) {
                    return new StringObject(vm, packageName);
                }
                break;
            }
            case "com/secneo/apkwrapper/H->ISMPASS:Ljava/lang/String;": {
                return new StringObject(vm, "###MPASS###");
            }
        }
        return super.getStaticObjectField(vm, dvmClass, signature);
    }
    @Override
    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature) {
            case "android/app/ActivityThread->getSystemContext()Landroid/app/ContextImpl;": {
                return vm.resolveClass("android/app/ContextImpl").newObject(signature);
            }
            case "android/app/ContextImpl->getPackageManager()Landroid/content/pm/PackageManager;": {
                return vm.resolveClass("android/content/pm/PackageManager").newObject(signature);
            }
            case "java/io/File->getPath()Ljava/lang/String;": {
                System.out.println("PATH:" + dvmObject.getValue());
                if (dvmObject.getValue().equals("android/os/Environment->getExternalStorageDirectory()Ljava/io/File;")) {
                    return new StringObject(vm, "/mnt/sdcard");
                }
            }
            case "java/lang/String->intern()Ljava/lang/String;": {
                String string = dvmObject.getValue().toString();
                return new StringObject(vm, string.intern());
            }
            case "java/lang/Class->getDeclaredFields()[Ljava/lang/reflect/Field;": {
                DvmClass c = (DvmClass) dvmObject;
                System.out.println(c.getClassName());
            }
        }
        return super.callObjectMethodV(vm, dvmObject, signature, vaList);
    }
    @Override
    public DvmObject<?> getObjectField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
        switch (signature) {
            case "android/content/pm/PackageInfo->applicationInfo:Landroid/content/pm/ApplicationInfo;": {
                return vm.resolveClass("android/content/pm/ApplicationInfo").newObject(signature);
            }
            case "android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;": {
                return new StringObject(vm, "/data/app/com.zhihu.android-XnTWbKh_JrM9gUKdEn_Wug==/base.apk");
            }
            case "android/content/pm/ApplicationInfo->dataDir:Ljava/lang/String;": {
                return new StringObject(vm, "/data/data/com.zhihu.android-XnTWbKh_JrM9gUKdEn_Wug==");
            }
            case "android/content/pm/ApplicationInfo->nativeLibraryDir:Ljava/lang/String;": {
                return new StringObject(vm, "/data/app/com.zhihu.android-XnTWbKh_JrM9gUKdEn_Wug==/lib/arm");
            }
        }
        return super.getObjectField(vm, dvmObject, signature);
    }
    @Override
    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "android/os/Environment->getExternalStorageDirectory()Ljava/io/File;": {
                return vm.resolveClass("java/io/File").newObject(signature);
            }
        }
        return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);
    }
    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
        if (pathname.equals("/proc/self/cmdline")) {
            return FileResult.success(new ByteArrayFileIO(oflags, pathname, "com.zhihu.android".getBytes()));
        }
        if (pathname.equals("/data/app/com.zhihu.android-XnTWbKh_JrM9gUKdEn_Wug==/base.apk")) {
            return FileResult.success(new SimpleFileIO(oflags, new File("D:\\code\\me\\app\\unidbg-master\\unidbg-android\\src\\test\\resources\\zhihu\\zhihu.apk"), pathname));
        }
        return null;
    }
}

```

unidbg 跑不起来的原因是有几个初始化的函数无法执行起来, 当时我也是打了几个 patch 下去, 最终没能跑起来, 也许是版本有差异, 我弄的是最新版的, 这个 unidbg 中的版本是 9.29.0.

还原算法
====

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAb1iaT1RsESRzPp369vwuRvDc3GaeQD9Dj7b99kDS3nD0wORExOwYxqg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这个就是目标函数了, 其实它有 5 个参数, ida 识别出错了, 参数一和参数 2 都是入参, 结果通过 a2 返回回去

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAQx371weZwMicqXwjLs2PQKaDoSVVMls2LW0r2HjAzWYyx0UdicicwGKZw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

来看下函数最开始的地方, v3 被赋值成了入参, v53 = *a3,v52 = a3[3] / 32 + 6 这里 ida 反汇编出来的有些问题, 不过根据 aes 的算法特征也可以大概猜出这里是在做什么, v52 = a3[3] / 32 + 6, 我们知道 AES 有 10 轮加密, 第 10 轮没有 mixcolumns(列混淆), 并且 aes 的分组长度 16 字节 128 位, 128/32+6=10, 下面的从 1 开始到 10 结束正好是 9 轮.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAsOMyIhXMlyjEIjib2soAuqibn2UlpEryYCxsGL70e2k79d5aCIxVLHgw/640?wx_fmt=png&from=appmsg)v52 = a3\[3\] / 32 + 6

初始轮秘钥加
======

这样的话框中的应该就是第一个轮秘钥加了. 说是加其实是异或, 比如秘钥 k0 是 0x11111111111111111111111111111111, 明文是 0x22222222222222222222222222222222, 异或就是他两直接异或就好了.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAHRriakBpud8icIrLat2ylSlyicFAanD8VwnOJ7Mve3OdLibXibnicE5vX23w/640?wx_fmt=png&from=appmsg)在这里插入图片描述

因为 c 语言只能一个字节一个字节的异或, 所以 16 个字节他这里有 16 轮. result 是第一个入参, 和第二个入参是同一个地址, 这里我 hook 过了 result 经过 9 轮加密始终没变, 所以上面 result 赋值的结果不用管, 只需要看

(&v36 + i) = (byte_D914[

(v53 + i) & 0xF ^ (16 *

(v3 + i))] >> 4) ^ (16 * (byte_D914[(

(v53 + i) >> 4) & 0xF ^ (16 * (

(v3 + i) >> 4))] >> 4)); 这行就可以了, 正常来说两个字节异或直接 0x11^0x22 就可以了, 他这里那么长肯定是改了 AES 的, v3 是入参, v53 来自上面的

a3, 接下来 unidbg 中看下这个 v53 是多少, 鼠标选中这个星号, 按 tab 转文本汇编

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAoGf7Emvy8bXWiaVAp2vy9T02qqSSgTHxzjsBQXquemEk3I6YjODTlTA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

STR R3, [R11,#-16] (Store Register), 存储指令, 结合着 c 代码, 就是把寄存器 r3 的值存到 R11 偏移 - 16 的位置, 对应着 v53=*a3,unidbg 中下断看下 R3 是多少

```
public void HookByConsoleDebugger() {
        Debugger debugger = emulator.attach();
        debugger.addBreakPoint(module.base+0x6444);
    }
需要encrypt函数调用前执行

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAxyasXcTcHz8dj85Vp5eSeXJ80q2nsdjsJMqRq0xxJJScjqlicWicRPzw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

m 某个寄存器可以直接看内存中的数据, 默认是 112 字节, 可以在后面加数组指定 dump 多少字节, 好像后面还有

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAficia2oGY1m3Xc5EhIWWjtMIq7aUUhAibeupM5vMO7mmhJbicqJUXJ5AoA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

长度是 16

11, 这个 11 好像是 aes 的 11 个轮秘钥, 但是他的 11 个轮秘钥是没有联系的, 标准的 aes 是第一轮可以推出后面的轮秘钥的. *(&v36 + i) = (byte_D914[

(v53 + i) & 0xF ^ (16 *

(v3 + i))] >> 4) ^ (16 * (byte_D914[(

(v53 + i) >> 4) & 0xF ^ (16 * (

(v3 + i) >> 4))] >> 4)); 16 轮循环下来, v53 只消耗第一个轮秘钥, v3 是入参, v36 是结果, byte_D914 是一个数组. 这里看上去写的有点复杂, 其实不然, 就是 byte_D914[

(v53 + i) & 0xF ^ (16 *

(v3 + i))] >> 4 异或 (16 * (byte_D914[(

(v53 + i) >> 4) & 0xF ^ (16 * (

(v3 + i) >> 4))] >> 4)), 想要获得结果就需要知道 byte_D914 是多少

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudA5VoqCnwQbRD7g0zSQkhUc8jBkaYibF0K43rqjqVxLIYHSopBRJEkXhQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

点过去只有 100 字节, 但是

(v53 + i) & 0xF ^ (16 *

(v3 + i)) 或者 (

(v53 + i) >> 4) & 0xF ^ (16 * (*(v3 + i) >> 4)) 的结果在 0x0 到 0xff 之间有 256 个, 所以一直带下面的 256 个应该都是 byte_D914 的. 但是当你把这 256 个字节扣出来并且加密一遍后你会发现结果有几个字节是正确的, 有几个字节是错误的, 应该是因为 dump 下来的 so 有些问题. 这里 ida 中的静态 byte_D914 是不准确的, 真正准确的是内存中的, 所以可以从 unidbg 中把这 256 个字节 dump 下来.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAK50KMKdaYpdBgRTjxVDG7xLWhYEWFT5rPf0NFQgeB1uE3K2CRrHqUA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

鼠标选中 byte_D914 按 tab 转到文本汇编, 后面的一些操作都需要汇编的基础, 因为 c 的代码不准确, 想要搞 so 的算法还原必须要懂汇编

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAtCjCCALd9xuYRxleJBkk0FIcSI3vGpNB3ibkwEdL4NNnW0Wib3Z69Wkw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

LDRB R3, [R2, R3](Load Register Byte) 字面意思, 从 R2+R3 的地址加载一个字节到 R3 中, 对应着 c 代码就是从 byte_D914 中取数据呗.

```
debugger.addBreakPoint(module.base+0x64c0);

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudACKAM39wxRbicKC7OshqvdGXy8XduretrbcD1KN4h7G5cARiaVcOoibPWw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

r3 0x18 就是偏移, 偏移是从 0x0 到 0xff, 所以 r2 中的数据应该就是 byte_D914

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAjXMsFfaFSphH0PjI6PqicRRh3TnS5XqSsiavlmwhO5ia2dvufBtF90iaVw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这些字节都对的上, 后面的就有些偏差了, 这里我们只要 unidbg 中的结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudADtIWqyhZcCpv82Sx3nvbO5R7iauCTdyt2tpueSCvuOEmyKec5vAyMng/640?wx_fmt=png&from=appmsg)在这里插入图片描述

还原算法的话就把 byte_D914 转成数组就好了, 用到的时候从里面取值. 经过上面一系列操作的话, 入参和第一个秘钥加密的结果就好了. 这个结果很重要

9 轮循环
=====

标准 aes 中的 9 轮循环里的内容是字节替换, 循环左移, 列混淆, 轮秘钥加. 轮秘钥加上面我们说过了, 字节替换就是以自身数值为索引取出 s 盒里的内容, s 盒 256 字节, 类似上面的 byte_D914. 循环左移和列混淆这个魔改的 aes 不涉及, 所以这里不说了, 稍微有些麻烦, 感兴趣的自行网上检索. 对照着这 4 个步骤来看这 9 轮循环 因为是魔改的, 字节替换, 循环左移, 列混淆, 轮秘钥加这 4 个步骤都改了, 所以就按照标准 aes 中的叫法来给下面的 4 个循环取上对应的名字

#字节替换
=====

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAPUDn8KoWvic3dqzx4wf0EjIicalarshhic3VHX3Ur0EeEhjTiaoREBFh3w/640?wx_fmt=png&from=appmsg)在这里插入图片描述

循环左移
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudA3QnJ7sAbRcqDrozDtiajYqPMFP683vqd1huRtHy2cI7Biarmn5rcw0AA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

列混淆和轮秘钥加
--------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudA1jGJPg2iaMr4WK894Dbc08M81Nib702hIxmt1GvkUa61LFxfjPz1Abicw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这 4 个步骤看上去挺相似的, 但是主要是 v4,v20.v53 这几个值在操作还有中间的数组取值操作, 这个数组的我就不说了, 和上面的数组获取的方式一样. v53 是 11 个轮秘钥, 步骤 4 正好也是用的 v53 中的轮秘钥, v4 和 v20 是主要需要关注的点, 中间记录了他们的变化, 我截图没截上而已. 这两个值的变化说简单也不简单, 说难也难不到哪去. 有那么一丝白盒 AES 查表的意味.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAc8GiaicXial9mjPg4qxhGEHy3F49uYUwAeZzhYwTplSbzKKBuvITjtEQw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这是 python 还原的几个数组还有几张表, 有点大, 折叠了. 接着上面的分析. 我只分析第一个字节替换的过程, 后面的 3 个步骤和这个的分析是一模一样的, 所以我尽可能写详细点.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAzibz2KtE11nHx9KjBVLXR64GksJEohXDOuYJ4ibJAg7VicooiagKEeoGgg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

主要是看 v4 和 v20 是怎么生成的. v36 也就是最开始的第一轮轮秘钥加得到的结果, 先来看下结果是多少. 鼠标选中 v36 转到文本汇编

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAuHWbksJCTbydce9x9udE6D8TaBLwS7TNSM1Jv658g4vQhVxAh17r7A/640?wx_fmt=png&from=appmsg)在这里插入图片描述

LDRB R3, [R11,#-36] LDRB 指令上面介绍过了, 这行指令的意思就是从 R11 偏移 - 36(-0x24) 的地方取下一个字节给 R3, 结合 c 代码, R11 处地址 - 0x24 中的数据就是 v36

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAORFQhb4wqKcKzRrSw8dlnzMvyiaocbntW3M4N0aC9KvHD2rfzXT2PNw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

0xbffff5ac-0x24=0xbffff588

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAzdibKJlHK1U6Lib3hcvFphs9hjApEEiahOXq1k7TXiahzYicWQXBd7pkcTQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

记住这 16 个字节, 每一轮加密中的 4 组加密都需要这个值 (每一轮不一样), 根据这里的索引去查表, 一次用 4 个字节, 一共 4 次刚好 16 个字节, 每次的 4 个字节从表中一次查 4 个字节也是 16 字节, 对应的正好就是 v4 和 v20 的值, 后面会说表怎么拿到.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAu0fNjXTYDNsD0zHjxvfD6dSL9Kl2zR15MU4zFiaq4TI37IbACcQLPQA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

接下来先去看下经过 68-99 行, v20 的值变成了多少.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudA4ubXJAQxqXQYOK7nNVG2rzZe1ibrQVM8icBwicnQvJw6lvoRQfUJ2lBgA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

取地址处看下汇编

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAwA50jI3qt6Qk3BRpprHiafia8QIX0RS4WpIanWCq7LcsY2EL6XOPveYw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

ADD R3, R2, R3 将寄存器 R2 和寄存器 R3 中的值相加，并将结果存储到寄存器 R3 中 unidbg 中下断看下 R2 和 R3 是什么, 根据 & v4 + i 不看断点处大概也能猜到, R3 是 i(偏移),R2 就是 v4

```
debugger.addBreakPoint(module.base+0x69B4);

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAqBHdpcNMMjp4LlgZL7m61KPY9mOpaf6q7quz4shwAZwVzXPdsPQLtg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

结果也正好验证了, 此处刚进入循环所有 r3 是 0,, 来看下 r2 的值, 也就是 v20

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudATaCpfGXWIZhYcYOr8wmGdeVQfpTZ97vxIXTuxZAKibDkUBzmx08Djkw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

ok, 结果我们知道了, 接下来带着结果去看 v20 是怎么生成的. 6574 处

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudApCyv1FuJGv9Shd2CAnUWFhKVS4ibcjIdxYtLoMkZicnTIsm5ULz3TjsA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

LDR R3, [R3,R2,LSL#2] 将寄存器 R2 中的值左移两位 (相当于 * 4)+R3 再赋给 R3,unidbg 中下断看下

```
debugger.addBreakPoint(module.base+0x6574);

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAHnYxv7wichKvOXuBn8aXpu7xSDYiavia2Z61BfmpVKkZfCfLUUeC0UI1A/640?wx_fmt=png&from=appmsg)在这里插入图片描述

r2 这个 0xc5 有没有很眼熟, 请看上面这张图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudADxXedXl5lvO9Tjv8iaSrb2BZRCyU4pUF4lB8GybeKdSJlSaPQcmEfpg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

按照 LDR R3, [R3,R2,LSL#2] 汇编的操作来执行一下, 0xc5

4=0x314,R3=0x4000dc14+0x314=0x4000df28, 看下内存中的值是多少

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAC83lNdz7WZQ1dlbcibHDHJUwaCUJKTKiaCjeHv9fFpVGKic3Xa88vU5uQ/640?wx_fmt=png&from=appmsg)

再看下 v20 的前 4 个字节

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudALum537GfadEWekw78k2UUY1pKAtsdXRMLHDSYojKqUcJQ4NCTcGxlw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

00 BC BC 03 变成 03 BC BC 00 怎么变得相信不需要我多说了, 后面的 12 个字节也是同理. 接下来说一下怎么获取这张大表, unidbg 中 hook 即可. 因为 LDR R3, [R3,R2,LSL#2],r2 在 0x0 到 0xff 之间,

4 后就是 0x0 到 1020, 以间隔 4dump 下 unidbg 中的内存即可

```
debugger.addBreakPoint(module.base+0x6574
        , new BreakPointCallback() {
            int num=0;
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                int i;
                Backend backend = emulator.getBackend();
                if(num==0){
                    for (i=0;i<=1020;i+=4){
                    int x =0x4000dc14+Integer.parseInt(Integer.toHexString(i), 16);
                    byte[] bytes = backend.mem_read(x, 0x04);
                    StringBuilder hexString = new StringBuilder();
                    for (byte b : bytes) {
                        hexString.append(String.format("%02X ", b & 0xFF));
                    }
                    System.out.println(Integer.parseInt(Integer.toHexString(i), 16)+":"+ hexString.toString().trim());
                };
                }
                num+=1;
                return true;
            }
        });

```

结果就是这样的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAKDplEuAonS7rfxqUicEv8TdCaaqmiaskF0cWpvlkMPE8mUFE60dWUDQg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

尝试搜索一下 0x314 也就是 788

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAty2vJ9Nt2E01EVL8icFCIYcXn2zquhTcEbqMjW1PQn49vcmiaHGibYb7Q/640?wx_fmt=png&from=appmsg)在这里插入图片描述

可以看到结果也是对的上的, 后面的流程都是这样, 自此, 几个数和表的获取都说完了, 接下来的流程就是细心点就没问题了.

第 10 轮
------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudANHDP1qUI10VSehXQBRASgdm2icA6oZhpdm8QbJbSDxDXg0WZpibbztcg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

第 10 轮标准 AES 有字节替换, 循环左移, 轮秘钥加. 这里它只有一个循环. 这个表的获取方式和上面的略有不同, 不过我相信你如果把前 9 轮加密能搞定的话, 最后一轮肯定是没问题的了.

总结
==

这样下来的话, 这个白盒 AES 中的算法就搞定了, 这里还有几个注意事项 1 传进去的明文是 32 位的, 刚好是两个分组长度, 所以会再填充一个分组, 这个填充模式也是魔改的, 具体可以看下方的图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAs6swichDXeSKbIOJsvRuCVyN3pB661vC9iaPznlVhxlUQ62fQSKJP93A/640?wx_fmt=png&from=appmsg)在这里插入图片描述

因为传进去的 md5 值 32 位, 所以只需要再填充一组就可以了, 也就是 9b9b9b9b9b9b9b9b9b9b9b9b9b9b9b9b, 所以总的分组数就是 3 组. 2 因为是 cbc 模式, 每个分组加密的结果会和下一轮的明文进行异或. 初始 iv 是 99303a3a32343a3992923a3b3a999292 3python 纯算代码太长了, 有好几张表, 这里我放不下就放到下方知识星球了, 感兴趣的可以加一下.

最后
==

微信公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VVlcG4MNLdWFiczfABgNtDcd9neWmhudANibVZM3V8iaqucfYbuULqSz1gT9PJ2uvA8thcymVTwLYCKVR4LyRz04Q/640?wx_fmt=jpeg&from=appmsg)在这里插入图片描述

知识星球

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VVlcG4MNLdWFiczfABgNtDcd9neWmhudAClBhLMcbTrXWOicR2SZDNPtnIU5lic8yeFtgdv7ibLsVjGGhQPyfG52Yg/640?wx_fmt=jpeg&from=appmsg)在这里插入图片描述

如果你觉得这篇文章对你有帮助也可请作者喝一杯咖啡

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdWFiczfABgNtDcd9neWmhudASOWgBkqAgjsPpeNkVrjl1dA42ncGP7LADGdSiaFZMvghwgWrUlSWjmA/640?wx_fmt=png&from=appmsg)在这里插入图片描述