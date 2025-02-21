> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTE1NDMwMA==&mid=2247485218&idx=1&sn=20400378564395d60a92dc9ce6d32ffb&chksm=ebf1c9b1dc8640a782b422cdbdcf6a7b921d30ff246f304d07d3b19945aea58e8250451f9b74&cur_album_id=1394397996049317893&scene=189#wechat_redirect)

案例：byd hy

SM4 密码算法详解：分组长度与加密流程 - CSDN 博客

https://github.com/py-gmssl/py-gmssl 可以看源码

https://github.com/guojuntang/sm4_dfa dfa 攻击

（六）国密 SM4 算法 - 知乎 (zhihu.com)

### 分析 SO

这里我没看 java 层，从 so 的 java_开始查看，根据经验可得

```
Java_com_bangcle_comapiprotect_CheckCodeUtil_checkcode 入口
这里是sm4的算法入口，bcda123fcd4d2019 这里为什么是iv？
bangcle_QSM4_cbc_encrypt(v112, v125, v128, &v789, "bcda123fcd4d2019", 16LL, v135, 131076LL, 1);// bcda123fcd4d2019=iv 


```

#### iv 探究

上面代码追踪到如下：

```
v18 = bangcle_CRYPTO_cbc128_encrypt(p, a3, item_count, bcda123fcd4d2019, &v23, off_142FA0);
进去之后发现就是用来异或的，iv的作用就是这个。 这里我做了patch，所以最后的iv=0000000000000000000000000000000
//这里要分为16个字符长度，每个字符跟bcda123fcd4d2019的相应位置异或。得到sm4加密之前的明文输入，1234得到的是SQWU=>?joh8h><=5


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOhj1F6XrZPw5ibwopPqNLO6Tg9Ynib5wb3rm288zSVbI3MV27tD2cQw9w/640?wx_fmt=png&from=appmsg)

#### sm4 加密探究

进入 a6 之后，可以看看算法如下，怎么看 a6? 这里看看 x4 的值。或者 off_142FA0 点进去看看就行

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOQ6xrs6Wtp4DCNsCj56Mric0MAHPbooLOOicXbDCU7byORabHUQzibCDuA/640?wx_fmt=png&from=appmsg)a6=bangcle_WB_QSM4_encrypt(__int64 a1, __int64 a2, __int64 *a3) 最终的方法

根据前序的知识，可以得出非常标准的算法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOxXTFLdv1BibOoxEKzUdkAib51J77X4o2x948LUMLCmH2iaOC5vXsusXKg/640?wx_fmt=png&from=appmsg)

如图，注入位置，后续就开始注入。使用 unidbg 代码如下：

```
package com;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.api.SystemService;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.utils.Inspector;
import com.github.unidbg.virtualmodule.android.AndroidModule;
import com.sun.jna.Pointer;
import keystone.Keystone;
import keystone.KeystoneArchitecture;
import keystone.KeystoneMode;
import unicorn.Arm64Const;

import java.io.*;
import java.util.ArrayList;
import java.util.Random;

/*
 */
public class bydhy extends AbstractJni implements IOResolver {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
        System.out.println("get path:" + pathname);
        if ("/proc/self/maps".equals(pathname) || ("/proc/" + emulator.getPid() + "/maps").equals(pathname)) {
            return FileResult.success(new SimpleFileIO(oflags, new File("unidbg-android/src/test/resources/byd/bydmpas"), pathname));
        }
        return null;
    }

    bydhy() {
// 创建模拟器实例
        emulator = AndroidEmulatorBuilder.for64Bit().setProcessName("com.byd.sea").build();
// 获取模拟器的内存操作接口
        final Memory memory = emulator.getMemory();
// 设置系统类库解析
        memory.setLibraryResolver(new AndroidResolver(23));
        emulator.getSyscallHandler().addIOResolver(this);
// 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/byd/bydhy.apk"));
// 设置JNI
        vm.setJni(this);
// 打印日志
        vm.setVerbose(true);
        new AndroidModule(emulator, vm).register(memory);
        ;
// 加载目标SO
// DalvikModule dm = vm.loadLibrary("encrypt", true);
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/resources/byd/libencrypt_bydhy.so"), true);
//获取本SO模块的句柄,后续需要用它
        module = dm.getModule();
// 调用JNI OnLoad
        dm.callJNI_OnLoad(emulator);
    }

    ;

    @Override
    public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
        switch (signature) {
            case "android/app/ActivityThread->currentActivityThread()Landroid/app/ActivityThread;": {
                return dvmClass.newObject(null);
            }
            case "android/os/SystemProperties->get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;": {
                String arg0 = varArg.getObjectArg(0).getValue().toString();
                String arg1 = varArg.getObjectArg(1).getValue().toString();
                System.out.println(arg0 + "====" + arg1);
                if (arg0.equals("ro.serialno")) {
                    return new StringObject(vm, "unknown");
                }
                return new StringObject(vm, "");
            }
        }
        return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);
    }

    @Override
    public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
        switch (signature) {
            case "android/app/ActivityThread->getSystemContext()Landroid/app/ContextImpl;": {
                return vm.resolveClass("android/app/ContextImpl").newObject(null);
            }
            case "android/app/ContextImpl->getPackageManager()Landroid/content/pm/PackageManager;": {
                DvmClass clazz = vm.resolveClass("android/content/pm/PackageManager");
                return clazz.newObject(signature);
            }
            case "android/app/ContextImpl->getSystemService(Ljava/lang/String;)Ljava/lang/Object;": {
                StringObject serviceName = varArg.getObjectArg(0);
                assert serviceName != null;
                System.out.println(serviceName.toString());
                return new SystemService(vm, serviceName.getValue());
            }
            case "android/net/wifi/WifiManager->getConnectionInfo()Landroid/net/wifi/WifiInfo;": {
                return vm.resolveClass("android/net/wifi/WifiInfo").newObject(null);
            }
            case "android/net/wifi/WifiInfo->getMacAddress()Ljava/lang/String;": {
                return new StringObject(vm, "da:c4:13:ef:95:aa");
            }
        }
        return super.callObjectMethod(vm, dvmObject, signature, varArg);
    }

    @Override
    public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {

        switch (signature) {
            case "android/os/Build->MODEL:Ljava/lang/String;":
                return new StringObject(vm, "google");
            case "android/os/Build->MANUFACTURER:Ljava/lang/String;":
                return new StringObject(vm, "google1");
            case "android/os/Build$VERSION->SDK:Ljava/lang/String;":
                return new StringObject(vm, "20.1");
        }
        return super.getStaticObjectField(vm, dvmClass, signature);

    }


    public String checkcode() throws FileNotFoundException {
//        emulator.attach().addBreakPoint(module.base + 0x29E6C); //28轮
//        emulator.attach().addBreakPoint(module.base + 0x29E9C); //29轮
//        emulator.attach().addBreakPoint(module.base + 0x29ECC); //30轮
//        emulator.attach().addBreakPoint(module.base + 0x29efc); //31轮
//        emulator.attach().addBreakPoint(module.base + 0x2580C); //结果下断

//        先把这里nop掉，好分析 不进行nop的值=XsdchC8eiT4MgMiS0PMHtQ== nop之后HzsW71C+Z3r+/Y1tWAsffg== 这里是iv
        UnidbgPointer pointer = UnidbgPointer.pointer(emulator, module.base + 0x29384); //这里要分为16个字符长度，每个字符跟bcda123fcd4d2019的相应位置异或。得到sm4加密之前的明文输入，1234得到的是SQWU=>?joh8h><=5
        Keystone keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian);
        String s = "nop";
        byte[] machineCode = keystone.assemble(s).getMachineCode();
        pointer.write(machineCode);
        pointer = UnidbgPointer.pointer(emulator, module.base + 0x29388);
        keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian);
        machineCode = keystone.assemble(s).getMachineCode();
        pointer.write(machineCode);
        pointer = UnidbgPointer.pointer(emulator, module.base + 0x2938C);
        keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian);
        machineCode = keystone.assemble(s).getMachineCode();
        pointer.write(machineCode);
//
//arg listm
        ArrayList<Object> params = new ArrayList<>(10);
//jnienv
        params.add(vm.getJNIEnv());
//jclazz
        params.add(0);
        //str1 参数//用4个字符刚好就是
        StringObject str1 = new StringObject(vm, "F1234");
        params.add(vm.addLocalObject(str1));
//int 参数2
        params.add(0);

        //str2 参数3
        StringObject str2 = new StringObject(vm, "1715591022250");
        params.add(vm.addLocalObject(str2));
        Number number = module.callFunction(emulator, 0x1DDE0, params.toArray());
        StringObject res = vm.getObject(number.intValue());
        return "";
    }

    public static void main(String[] args) throws FileNotFoundException {
        bydhy b = new bydhy();
        System.out.println(b.checkcode());
    }
}



```

这里 F1234 作为明文输入，patch iv 之后的结果为`1f3b16ef 50be677a fefd8d6d 580b1f7e`

分别输入改 x0 就行，手动改，拿 28 轮举例，最终的结果。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOdBK2cicXXr8ZCALg1Srfia6as6IozDBlibvLMLibukF1P1kVMMtHQTvmww/640?wx_fmt=png&from=appmsg)其他轮次依旧这样，最后的结果如下。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOTxOiae79pGicjnZ27TANe21kCeXNurMKIwwa6YvyevlAoqScOzN5qcVg/640?wx_fmt=png&from=appmsg)phoenixSM4 之后为：

```
import phoenixSM4

with open('tracefile', 'wb') as t:
    t.write("""1f3b16ef5****d8d6d580b1f7e
c0d87ff54****fd8d6d580b1f7e
a2cd206a4f222e95fefd8d6d580b1f7e
bc2d8a7c4e****d8d6d580b1f7e 
9e43147076bf48e5b2880a15580b1f7e
5e7c8624****b0b12580b1f7e 
16682a9fa8e47260b38b0b15580b1f7e 
f9c2ca932****c29ea249ee4 
b8025b071088252c133e42ebeb279fe3
d3e683f053****36b7ea249ee3 
397d454b8f19d16f2fd06790e9b61b33
87cc59fa32****4314ddaa83f
65cc8506759bc518c1ad173c64882580
""".encode('utf8'))

phoenixSM4.crack_file('tracefile')
结果：
Round key 32 found:
21****
Round key 31 found:
B****4
Round key 30 found:
68****96
Round key 29 found:
EDF3A9FA


```

```
./sm4_keyschedule EDF3A9FA **** B2D12B44 **** 32   
Key: **** 9A4A5585 **** 142A2B9E
K00: **** CCE066D5 27D246F9 A65A0942
K04: **** B7DE1D60 A35DF2E1 A62C2EDD
K08: **** 493CE50E **** 09D66C42
K12: **** 42782EB5 3104CB06 8C7525F0
K16: **** 2C376B48 FB588F56 6A317921
K20: 9**** D32E709F BBBABC1F FCF3E7F8
K24: **** E9A12B08 **** 90BFEA02
K28: 7**** B2E1AC9 1763BA27 EF90DD2E
K32: **** 682E0F96 B2D12B44 21E6B235

修改端序
./sm4_keyschedule FAA9F3ED **** 442BD1B2 **** 32                                  
Key: **** 89ABCDEF **** 89ABCDEF
K00: **** DF01FEBF 665ED4F0 3BDBEF33
K04: **** 41662B61 C15C6744 527431C6
K08: **** 494B8F3E **** E47F3D90
K12: **** 47CEAA78 76D78969 0CF3CC88
K16: ****354D88D4 666F8E03 A3993068
K20: **** 0A6A9446 **** 73C658F6
K24: ****7F58ED7B C6ED22D9 E8FB6FFF
K28: **** C81955CD 0B24484F 0D3FE944
K32: **** 960F2E68 442BD1B2 35B2E621


```

‍

### 注意点

这里最后的结果我也没找到差异，[原创] 白盒 SM4 的 DFA 方案 - Android 安全 - 看雪 - 安全社区 | 安全招聘 | kanxue.com 这边文章提示了我，第 32 轮注入的话影响 4 字节，我选的位置是没有进行轮秘钥处理的，所以这位置不可取，后续也只有一个反序，所以忽略。大佬也说了有轮密钥端序的问题，最后处理下就行。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RQicOzqf0IHa7zJGoJ1Ul4CDuT3bbmMdOXSo9HwZZWoAnDnMUQw0daKWEg9ianOKevzzoIexFZzicIVRLmj7cKq1Q/640?wx_fmt=jpeg)