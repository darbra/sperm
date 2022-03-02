> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-270228.htm)

> [原创] 某东到家 app so signkeyV1 参数分析

目录

*   [前言](#前言)
*   charles 抓包
*   java 层分析
*   frida hook
*   [unidbg](#unidbg)
*   IDA 动态调试
*   GDB 动态调试

前言
==

> **_京东到家 app signKeyv1 参数分析，版本 8.14.0_**

charles 抓包
==========

![](https://bbs.pediy.com/upload/attach/202111/939409_TBKCKYY3HXF5ACK.png)

 

就是 `djencrypt` 这个参数，是一个加密串，分析一下

java 层分析
========

![](https://bbs.pediy.com/upload/attach/202111/939409_PBRS4ETJFAP7XJT.png)

 

全局搜索定位到这个函数 `base.net.volley.BaseStringRequest.getParams`，跟进 `DaojiaAesUtil.encrypt` 看看

 

![](https://bbs.pediy.com/upload/attach/202111/939409_EB8MD53JEPV7EVV.png)

 

在跟进 `AesCbcCrypto.encrypt` 这个函数

 

![](https://bbs.pediy.com/upload/attach/202111/939409_PHRNDFWPYEFZXCC.png)

 

里面写的还是比较清楚的，加密方式是 `AES/CBC/PKCS5Padding` 使用 `cyberchef` 解密试试

 

![](https://bbs.pediy.com/upload/attach/202111/939409_282A3ESVSA3YQUH.png)

 

解密成功，主要是分析这个 `signKeyV1` 参数，看长度是 `64`，猜测是 `sha256` 加密，分析一下

 

![](https://bbs.pediy.com/upload/attach/202111/939409_NK52EZJ5PYX9M2T.png)

 

全局搜索定位到这里，调用 `k2 native` 函数获取的，在 `libjdpdj.so` 文件里

 

![](https://bbs.pediy.com/upload/attach/202111/939409_6U3RN5DEW6UEKC9.png)

 

打开 `so` 进入 `JNI_OnLoad` 函数，这里是动态注册的，点击 `off_117004`

 

![](https://bbs.pediy.com/upload/attach/202111/939409_RTAYCAZJVGKGYZD.png)

 

这里看到，函数注册列表，点进 `gk2` 函数

 

![](https://bbs.pediy.com/upload/attach/202111/939409_2AAVEU9H5ERS47P.png)

 

前面是一些数据处理，下面有个 `j_hmac_sha256` 函数，可以确实是用的 `hmac sha256` 算法

 

![](https://bbs.pediy.com/upload/attach/202111/939409_N3KTDGJ94GF5ZVY.png)

 

这里的符号都没去掉，`init update final` 函数都能看到，这里应该是标准的算法，因为使用的是 `openssl` 库，先写个 `frida hook` 一下

frida hook
==========

```
function hook() {
    var javaString = Java.use('java.lang.String');
    var zCls = Java.use('jd.net.z');
 
    zCls.k2.implementation = function (a) {
        console.log('zCls.k2.a: ', javaString.$new(a));
 
        var res = this.k2(a);
        console.log('zCls.k2.res: ', res);
 
        return res;
    }
}

```

这里 `hook java k2` 函数的输入输出

```
function hookSo1() {
    var hmac_sha256 = Module.findExportByName('libjdpdj.so', 'hmac_sha256')
    var HMAC_CTX_init = Module.findExportByName('libjdpdj.so', 'HMAC_CTX_init')
    var HMAC_Update = Module.findExportByName('libjdpdj.so', 'HMAC_Update')
    var HMAC_Init_ex = Module.findExportByName('libjdpdj.so', 'HMAC_Init_ex')
 
    Interceptor.attach(hmac_sha256, {
        onEnter: function (args) {
            console.log('hmac_sha256 参数 1: ', hexdump(args[0]));
            console.log('hmac_sha256 参数 2: ', hexdump(args[1]));
            console.log('hmac_sha256 参数 3: ', hexdump(args[2]));
            console.log('hmac_sha256 参数 4: ', hexdump(args[3]));
        },
        onLeave: function (retValue) {
        }
    })
 
    Interceptor.attach(HMAC_CTX_init, {
        onEnter: function (args) {
            console.log('HMAC_CTX_init 参数 1: ', hexdump(args[0]));
        },
        onLeave: function (retValue) {
        }
    })
 
    Interceptor.attach(HMAC_Update, {
        onEnter: function (args) {
            console.log('HMAC_Update 参数 1: ', hexdump(args[0]));
            console.log('HMAC_Update 参数 2: ', hexdump(args[1], {length: 1200}));
            console.log('HMAC_Update 参数 3: ', hexdump(args[2]));
        },
        onLeave: function (retValue) {
        }
    })
 
    Interceptor.attach(HMAC_Init_ex, {
        onEnter: function (args) {
            console.log('HMAC_Init_ex 参数 1: ', hexdump(args[0]));
            console.log('HMAC_Init_ex 参数 2: ', hexdump(args[1]));
            console.log('HMAC_Init_ex 参数 3: ', hexdump(args[2]));
            console.log('HMAC_Init_ex 参数 4: ', hexdump(args[3]));
            console.log('HMAC_Init_ex 参数 5: ', hexdump(args[4]));
        },
        onLeave: function (retValue) {
        }
    })
}

```

这里在 `hook` 一些 `so` 函数，`hamc` 会有个 `key` 一般在 `init` 的时候初始化

```
function main() {
    Java.perform(function () {
        hook();
        hookSo1();
    })
}

```

启动脚本 `frida -UF -l hook.js | tee hook.log`

 

![](https://bbs.pediy.com/upload/attach/202111/939409_9RN79D6VPM8S6QT.png)

 

`k2` 函数的输入是请求参数

 

![](https://bbs.pediy.com/upload/attach/202111/939409_QAXQB43VAVJJXDZ.png)

 

`HMAC_Init_ex so` 函数的参数二，长度 `32` 猜测是 `hamc key`

 

![](https://bbs.pediy.com/upload/attach/202111/939409_RSEFP43MVF5RZUT.png)

 

`HMAC_Update` 函数是请求参数

 

![](https://bbs.pediy.com/upload/attach/202111/939409_QTGMXH2JZ8UEEPW.png)

 

最后的加密结果 `ae07cde50402ef91660dea93dc196f7f82e7bc04322baf4022dc2879434f3fae` 是这个，来验证一下

 

![](https://bbs.pediy.com/upload/attach/202111/939409_P2HK7F8NPAP43QU.png)

 

使用 `cyberchef` 加密，结果一样，正是 `hmac sha256`

 

**_Tips: 下面在使用 unidbg 跑起来，毕竟多掌握一些工具总有用处_**

unidbg
======

```
package com.xiayu.jingdongdaojia;
 
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.LibraryResolver;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.debugger.DebuggerType;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.memory.Memory;
 
import java.io.File;
import java.io.IOException;
 
public class SignKeyV1Test extends AbstractJni {
    private final AndroidEmulator emulator;
    private final Module module;
    private final VM vm;
 
    public String apkPath = "apk path";
    public String soPath = "so path";
 
    private static LibraryResolver createLibraryResolver() {
        return new AndroidResolver(23);
    }
 
    private static AndroidEmulator createARMEmulator() {
        return AndroidEmulatorBuilder.for32Bit().build();
    }
 
    public SignKeyV1Test() {
        emulator = createARMEmulator();
        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(createLibraryResolver());
        vm = emulator.createDalvikVM(new File(apkPath));
        vm.setVerbose(true);
 
        DalvikModule dm = vm.loadLibrary(new File(soPath), false);
        vm.setJni(this);
 
        dm.callJNI_OnLoad(emulator);
        module = dm.getModule();
    }
 
    public void callGetSignKeyV1() {
        DvmClass zClass = vm.resolveClass("jd/net/z");
 
        DvmObject strRc = zClass.callStaticJniMethodObject(
                emulator,
                "k2([B)Ljava/lang/String;",
                new ByteArray(vm, "参数".getBytes())
        );
 
        System.out.println("callGetSignKeyV1: " + strRc.getValue());
    }
 
    public static void main(String[] args) throws IOException {
        SignKeyV1Test signKeyV1 = new SignKeyV1Test();
 
        signKeyV1.callGetSignKeyV1();
        signKeyV1.destroy();
    }
 
    private void destroy() throws IOException {
        emulator.close();
    }
}

```

代码写完跑起来

 

![](https://bbs.pediy.com/upload/attach/202111/939409_2S4K3PYER8PK4RN.png)

 

这报错了，缺少函数 `jd/utils/StatisticsReportUtil->getSign()Ljava/lang/String;` 调用的是京东到家 `apk` 的 `java` 代码

 

![](https://bbs.pediy.com/upload/attach/202111/939409_YE9K648QCNUR5TU.png)

 

点进来看一下，打开逻辑是获取 `apk` 的签名之类的，这里的依赖比较多，不是很好补，一般签名啥的都是固定，直接 `frida call` 一下，获取返回值

```
function callGetSign() {
    var StatisticsReportUtil = Java.use('jd.utils.StatisticsReportUtil');
 
    var res = StatisticsReportUtil.getSign();
    console.log(res)
}

```

运行成功，获取返回值

 

![](https://bbs.pediy.com/upload/attach/202111/939409_C8KYJH7MR2D2ZZP.png)

 

`unidbg` 补一下，直接写死字符串

 

![](https://bbs.pediy.com/upload/attach/202111/939409_RZFQNBJFG3TQPE2.png)

 

再次运行结果出来了，结果相同

 

**_Tips: 后面在打算学习学习 ida gdb 动态调试，暂时留空_**

IDA 动态调试
========

// TODO

GDB 动态调试
========

// TODO

> 原文链接：[某东到家 app so signkeyV1 参数分析](https://www.qinless.com/752)

[【公告】看雪团队招聘安全工程师，将兴趣和工作融合在一起！看雪 20 年安全圈的口碑，助你快速成长！](https://job.kanxue.com/position-read-1104.htm)

最后于 2021-11-12 00:28 被爬山的小脑虎编辑 ，原因：

[#逆向分析](forum-161-1-118.htm)