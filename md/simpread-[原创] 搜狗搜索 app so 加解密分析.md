> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-270177.htm)

> [原创] 搜狗搜索 app so 加解密分析

仅供学习研究 。请勿用于非法用途，本人将不承担任何法律责任。
==============================

目录

*   仅供学习研究 。请勿用于非法用途，本人将不承担任何法律责任。
*   const 小白龙_龙哥 = function () {}
*   [前言](#前言)
*   [样本](#样本)
*   [一、简单分析](#一、简单分析)
*                    0x1 jadx
*                    0x2 ida
*                    0x3 frida hook
*                    0x4 unidbg
*   二、so rsa base64
*                    [0x1](#0x1)
*                    0x2 rsa
*                    0x3 EncryptWall::WallKey::WallKey
*                    0x4 RSA_Encrypt
*                    0x5 base64Encode
*                    0x6 unidbg
*                    0x7 unidbg unicorn inline hook
*   三、so zip aes
*                    0x1 so 分析
*                    0x2 unidbg
*                    0x3 unidbg console debugger zip_compress
*                    0x4 unidbg console debugger aes
*   四、response 解密
*                    [0x1](#0x1)
*                    [0x2](#0x2)
*                    [AES_Decrypt](#aes_decrypt)
*                    0x3 zip_uncompress
*                    [0x4](#0x4)
*                    0x5 unidbg
*                    0x6 unidbg console debugger
*                    0x7 python
*   [最后](#最后)

const 小白龙_龙哥 = function () {}
=============================

> const a = 龙哥是全能的傻宝，每天都会换一种姿势来分享各种骚操作  
> const b = 10 月份开始准备密码学课程，11 月份准备更完 aes 密码学系列课程  
> const c = 除了密码学，还有各种 unidng 骚操作，反 unidbg，反反 unidbg，全是出自傻宝  
> const d = [赶紧点击加入星球吧，买到就是赚到 !](https://t.zsxq.com/BeeyBIQ)  
> return 无敌的龙哥，全能的傻宝

前言
==

> 最近在学习 `so` 加解密逆向分析，算法还原，就拿星主分析过的一个样本来练手，那个星主？  
> 当然是 `call 小白龙_龙哥();`

样本
==

> **_搜狗搜索 app_** 版本 `7.9.6.6` 主要分析微信公众号文章请求参数的加解密

[](#一、简单分析)一、简单分析
=================

![](https://bbs.pediy.com/upload/attach/202111/939409_QKEHC4KWAUNEPMP.png)

 

以上是抓包截图，请求跟响应都是加密串，主要是分析这些

### 0x1 jadx

![](https://bbs.pediy.com/upload/attach/202111/939409_8V87ES7V6PX8WNM.png)

 

`jadx-gui` 打开之后，开始搜索，这里因为请求没啥关键字，搜索 `url` 即可，出来一个结果点进去看看

 

![](https://bbs.pediy.com/upload/attach/202111/939409_HKKYZ2FS9K4QGJ7.png)

 

这里调用了两个函数 `encrypt dencrypt` 应该就是加解密了，跳转到声明函数的地方

 

![](https://bbs.pediy.com/upload/attach/202111/939409_REQHP5BUDJXBFPD.png)

 

进来发现全部是 `navite` 函数，这里加载了一个 `so` 文件，`navite` 的实现应该就在这里  
使用 `ida` 打开看一下

### 0x2 ida

![](https://bbs.pediy.com/upload/attach/202111/939409_HQUG4M9M8KM3RD7.png)

 

打开 `ida -> exports` 窗口，搜索 `java` 发现这里是静态注册的函数，刚好对应 `java` 层的四个函数  
先分析到这里，下篇文章在分析 `so` 具体的算法实现，下面使用 `frida hook` 验证一下

### 0x3 frida hook

```
function hook() {
    var ScEncryptWall = Java.use('com.sogou.scoretools.ScEncryptWall');
 
    ScEncryptWall.decrypt.implementation = function (a) {
        console.log('decrypt.a: ', a);
 
        var res = this.decrypt(a);
        console.log('decrypt.res: ', res);
        console.log('decrypt.res: ', Java.use('java.lang.String').$new(res));
 
        return res;
    }
 
    ScEncryptWall.encrypt.implementation = function (a, b, c) {
        console.log('encrypt.a: ', a);
        console.log('encrypt.b: ', b);
        console.log('encrypt.c: ', c);
 
        var res = this.encrypt(a, b, c);
        console.log('encrypt.res: ', res);
 
        return res;
    }
}
 
 
function main() {
    Java.perform(function () {
        hook();
    })
}
 
 
setImmediate(main);

```

![](https://bbs.pediy.com/upload/attach/202111/939409_9C83TWDHPN9BXD3.png)

 

![](https://bbs.pediy.com/upload/attach/202111/939409_GEHQP3Y8HUZ43FK.png)

 

结果全部 `hook` 出来了，跟抓包结果对比是一样的，在使用 `unidbg` 跑起来

### 0x4 unidbg

```
package com.xiayu.sougou;
 
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.LibraryResolver;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.hook.hookzz.HookZz;
import com.github.unidbg.hook.hookzz.IHookZz;
import com.github.unidbg.hook.xhook.IxHook;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.XHookImpl;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;
import com.sun.jna.Pointer;
 
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
 
public class SouGou extends AbstractJni {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
 
    public String apkPath = "/Users/admin/Desktop/android/file/sougou-7.9.6.6.apk";
    public String soPath = "unidbg-android/src/test/resources/test_so/sougou/libSCoreTools-7.9.6.6.so";
 
    private static LibraryResolver createLibraryResolver() {
        return new AndroidResolver(23);
    }
 
    private static AndroidEmulator createARMEmulator() {
        return AndroidEmulatorBuilder.for32Bit().build();
    }
 
    public SouGou() {
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
 
    public void encrypt() {
        List list = new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
 
        list.add(vm.addLocalObject(new StringObject(vm, "http://app.weixin.sogou.com/api/searchapp1")));
        list.add(vm.addLocalObject(new StringObject(vm, "type=2&ie=utf8&page=2&query=python&select_count=0&usip=1")));
        list.add(vm.addLocalObject(new StringObject(vm, "")));
        Number number = module.callFunction(emulator, 0x9CA0 + 1, list.toArray())[0];
 
        System.out.println("encrypt: " + vm.getObject(number.intValue()).getValue().toString());
    }
 
    public static void main(String[] args) throws IOException {
        SouGou sougou = new SouGou();
        sougou.encrypt();
        sougou.destroy();
    }
 
    private void destroy() throws IOException {
        emulator.close();
    }
} 
```

以上就全部代码跑起来

 

![](https://bbs.pediy.com/upload/attach/202111/939409_WM9X5WP9SKZH96M.png)

 

![](https://bbs.pediy.com/upload/attach/202111/939409_HETCUX2SFXNJA4H.png)

 

代码跑起来但是结果为空，应该是哪里有问题  
回去看下 `so` 发现有个 `init` 函数，应该就是初始化，先调用 `init`

```
public void init() {
    DvmClass Context = vm.resolveClass("android/content/Context");
 
    List list = new ArrayList<>(10);
    list.add(vm.getJNIEnv());
    list.add(0);
    list.add(vm.addLocalObject(Context.newObject(null)));
    Number number = module.callFunction(emulator, 0x9564 + 1, list.toArray())[0];
 
    System.out.println("init： " + number.intValue());
} 
```

![](https://bbs.pediy.com/upload/attach/202111/939409_XVVVS952W6GXVEB.png)

 

在运行结果就正常出来了, 接着在来分析下 `rsa base64` 的加密逻辑

二、so rsa base64
===============

### 0x1

![](https://bbs.pediy.com/upload/attach/202111/939409_STFH944TENX33AU.png)

 

也就是这里点进去

 

![](https://bbs.pediy.com/upload/attach/202111/939409_RU7DZ97ZK53577T.png)

 

一共五个参数，前两个是默认的，分别是 `JNIEnv jObject or jClass`，后面三个就是我们传进来的  
上面是做了一些字符串初始化然后调用了 `j_Sc_EncryptWallEncode` 这个函数，点进去看看

 

![](https://bbs.pediy.com/upload/attach/202111/939409_GV4VZPG6DQVUQN9.png)

 

进来之后有三个函数需要我们关注，前两经过分析都不是很重要，主要分析最后一个

 

![](https://bbs.pediy.com/upload/attach/202111/939409_ACFQ7CD3FMH5T24.png)

 

要感谢代码符号没去掉，才能看到里面密密麻麻的算法

 

![](https://bbs.pediy.com/upload/attach/202111/939409_455BAUWT52CWA7P.png)

 

在往下分析，就看到我们熟悉字符串，不正是前面请求包的请求参数吗，这里就是全部算法的加密逻辑了

### 0x2 rsa

![](https://bbs.pediy.com/upload/attach/202111/939409_VC29FEHV2FMBXGT.png)

 

前面是一些常规的判断赋值操作

> **_`v10 = operator new(48u);`_**  
> 这一行就是创建一个大小 48 字节 的内存，v10 就只内存地址
> 
> **_`EncryptWall::WallKey::WallKey(v10);`_**  
> 这里调用一个函数，把内存地址进去，点进去看看

### 0x3 EncryptWall::WallKey::WallKey

![](https://bbs.pediy.com/upload/attach/202111/939409_6G4N6RM86AU3837.png)

 

点进来逻辑也比较简单，两个 `do while` 一共循环 48 次，刚好把 48 字节大小的 内存填空

> **_`linux 下的 lrand48()` 函数_**  
> 随机生成 `0 - 2 ^ 31` 之间的长整数

 

**_Tips: 继续往下_**

> **_`v12 = RSA_Encrypt(v10 + 16, 32u, &v60, &v59);`_**  
> 这里就是调用 `rsa` 加密函数，四个参数分别是 `v10 的首地址加上 16`，`32数字`，`v60 的地址`，`v59 的地址`

### 0x4 RSA_Encrypt

![](https://bbs.pediy.com/upload/attach/202111/939409_PXEES4EGZ5QASRR.png)

> **_`n_crypto::SetSignPubKey`_**  
> 这里是生成 `public_key`，模数，指数都传进去了
> 
> **_`n_crypto::PublicEnc(v8, v9, v6, &v11, v4);`_**  
> 调用这个函数开始加密，一共有五个参数  
> 分别是 `1：32字节的地址`，`2：32数字`，`3：128字节大小的内存`，`4：内存地址`，`5：80字节大小的内存地址`

 

**_Tips: rsa 分析结束_**

### 0x5 base64Encode

![](https://bbs.pediy.com/upload/attach/202111/939409_FHSPCVUNVRN8UT7.png)

 

这里就是对 `rsa` 的加密结果，再来一层 `base64` 编码，稍后可以验证下是否是标准的 `base64`

### 0x6 unidbg

> **_算法分析的差不多了，在使用 `unidbg 的 console debugger 跟 hook` 功能，验证下数据，不熟悉的可以看下上面的推荐文章_**

### 0x7 unidbg unicorn inline hook

> **_先 `EncryptWall::WallKey::WallKey` 函数看看生成的啥数据_**

```
public void hookUnicornWallKey() {
    emulator.getBackend().hook_add_new(new CodeHook() {
        @Override
        public void onAttach(Unicorn.UnHook unHook) {
        }
 
        @Override
        public void detach() {
        }
 
        @Override
        public void hook(Backend backend, long address, int size, Object user) {
            if (address == (module.base + 0xB332)) {
                System.out.println("Hook By Unicorn hookUnicornWallKey");
                RegisterContext ctx = emulator.getContext();
                Pointer input1 = ctx.getPointerArg(0);
                Inspector.inspect(input1.getByteArray(0, 0x100), " 参数1");
            }
            if (address == (module.base + 0xB336)) {
                RegisterContext ctx = emulator.getContext();
                Inspector.inspect(ctx.getPointerArg(0).getByteArray(0, 0x100), " Unicorn hook hookUnicornWallKey");
            }
        }
    }, module.base + 0xB332, module.base + 0xB336, null);
}

```

![](https://bbs.pediy.com/upload/attach/202111/939409_R72SJ56ZXH36NF9.png)

 

运行结果可以看到，刚好是 48 字节 的数据

> **_接着分别 `hook rsa base64` 函数_**

```
public void hookUnicornRsa() {
    emulator.getBackend().hook_add_new(new CodeHook() {
        @Override
        public void onAttach(Unicorn.UnHook unHook) {
        }
 
        @Override
        public void detach() {
        }
 
        @Override
        public void hook(Backend backend, long address, int size, Object user) {
            if (address == (module.base + 0xB34E)) {
                RegisterContext ctx = emulator.getContext();
                Pointer input1 = ctx.getPointerArg(0);
                Inspector.inspect(input1.getByteArray(0, 0x100), " hookUnicornRsa 参数1");
            }
            if (address == (module.base + 0xB352)) {
                RegisterContext ctx = emulator.getContext();
                Inspector.inspect(ctx.getPointerArg(0).getByteArray(0, 0x100), " Unicorn hook hookUnicornRsa");
            }
        }
    }, module.base + 0xB34E, module.base + 0xB352, null);
}
 
public void hookUnicornBase64Encode() {
    emulator.getBackend().hook_add_new(new CodeHook() {
        @Override
        public void onAttach(Unicorn.UnHook unHook) {
        }
 
        @Override
        public void detach() {
        }
 
        @Override
        public void hook(Backend backend, long address, int size, Object user) {
            if (address == (module.base + 0xB372)) {
                RegisterContext ctx = emulator.getContext();
                Pointer input1 = ctx.getPointerArg(0);
                Inspector.inspect(input1.getByteArray(0, 0x100), " hookUnicornBase64Encode 参数1");
            }
            if (address == (module.base + 0xB376)) {
                RegisterContext ctx = emulator.getContext();
                Inspector.inspect(ctx.getPointerArg(0).getByteArray(0, 0x100), " Unicorn hook hookUnicornBase64Encode");
            }
        }
    }, module.base + 0xB372, module.base + 0xB376, null);
}

```

代码写完跑起来

 

![](https://bbs.pediy.com/upload/attach/202111/939409_SHYJU8JC55JVUJ6.png)

> **_`rsa` 的第一个参数，是 32 个字节，也就是前面随机生成 48 字节数据的后 32 个字节数据，返回值是 128 个字节_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_VVPSD2Y3J5AUBMY.png)

> **_`base64` 的第一个参数，刚好是前面 `rsa` 的返回值_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_DRYRGTHRY7M2CQP.png)

> **_使用 `CyberChef` 网站来验证，是否是标准的 `base64` ，结果不出所料，就是标的_**  
> **_`rsa base64` 就分析完了，下一篇文章在分析 `zip + aes` 算法_**

三、so zip aes
============

### 0x1 so 分析

![](https://bbs.pediy.com/upload/attach/202111/939409_CDCACCPG7CA7EC9.png)

> **_v46 = sub_B178(s, v17, v11, v57);_**  
> 先来看看这段代码，`v46` 刚好是 `u` 参数，调用 `sub_B178` 函数获取的

 

![](https://bbs.pediy.com/upload/attach/202111/939409_9DKDV6M5B38XXNR.png)

 

点进来可以看到一些算法，分别是 `zip_compress aes base64`

> 对 `aes` 算法了解的同学应该知道需要关注那些内容，不了解的可以 `call 龙哥 ()` 跟着一起学习密码学  
> 咱们需要关注的是  
> **_1、使用的密码长度：AES-128 or AES-192 or AES-256_**  
> **_2、明文是啥_**  
> **_3、密钥 key 是啥_**  
> **_4、加密模式是啥：ECB or CBC，如果非 ECB，那么 IV 是啥_**  
> **_5、填充方式是啥：No or PKCS5 or PKCS7_**

 

**_Tips: 下面一个一个来分析，先来看看 sub_B178 函数的入参是啥_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_JHTFXGHEWUP82FK.png)

> **_1、s 是参数 a1_**  
> **_2、v17 是 s 的长度_**  
> **_3、v11 是 48 字节大小的首地址加 16 的地址_**  
> **_3、v57 48 字节大小的地址_**

 

**_Tips: 进入 sub_B178 函数继续分析_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_KQCM6XFRMTWYJ6V.png)

> 这里的 `zip_compress` 需要注意，是用的是 `deflate` 压缩算法  
> 其他的 `zlib gzip` 实际在压缩的时候会添加上头和尾，使用这两个算法需要去掉头和尾才是真正的数据  
> [参考链接 gzip,deflate,zlib 辨析](https://blog.csdn.net/wy_2012/article/details/6304946)

 

继续分析 `AES_Encrypt` 函数，`Base64Encode` 函数可以暂时忽略，看下参数

> **_1、v10 = v9 也就是 v9 的内存地址_**  
> **_2、v11 是 zip_compress 函数的结果_**  
> **_3、&v16 地址_**  
> **_4、v5 参数 a3，也就是前面 48 字节的 后 32 位_**  
> **_5、32u 32 数字_**  
> **_6、v4 参数 a4 前面的 48 字节_**  
> **_7、16u 16 数字_**

 

**_Tips: 参数分析完，点击去继续分析_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_SBK575DGYHUNEXA.png)

> 先分析 `n_crypto::SetEncKeySym(&v20, v10, 8 * a5);` 这一行代码，看像是密钥相关函数  
> **_1、&v20 char 类型的地址_**  
> **_2、v10 是参数 a4_**  
> **_3、8 x a5，a5 是参数 a5 8 x a5 = 256_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_Z2AFVYV7ZJHSGRE.png)

> `SetEncKeySym` 函数点进来，看到这个判断，就可以判断是 `AES-256`, `在AES-256中，密钥是32字节`  
> **_这里的 a2 就是参数 3，而参数 1 = a4 = 32 字节，这里的参数 1 应该就是密钥_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_WDRJMRBC5QT2UPQ.png)

 

再往下分析 `padding` 填充方式，目前看不出来是啥方式，不过通过参数 `16u` 可以猜测这个应该是 `block` 大小，`Pcks#5 默认是 block = 5`, `Pcks#7 可变的 1-255`，这里应该是 `Pcks#7` 填充，后面可以验证下

 

**_Tips: 继续分析 n_crypto::EncSym 函数_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_88QHJDGQQBMNVX6.png)

> 先来看下参数  
> **_1、&v21 是个内存地址。v21 在上面有初始化，a6 就是前面的 48 字节，这里应该是取前 16 个字节，猜测：前 16 是 IV，后面 32 是密钥_**  
> **_2、v9 内存地址_**  
> **_3、v18 内存地址_**  
> **_4、v14 内存地址_**  
> **_5、&v20 应该是密钥的内存地址_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_2F2A6QY2XQMAD6E.png)

 

EncSym 函数点进去，通过符号可以发现是 CBC 加密模式

 

![](https://bbs.pediy.com/upload/attach/202111/939409_5Q7BUDNE3JQ3FQA.png)

 

前面的猜测确实正确，`CBC` 模式下的 `IV` 在运算会先跟密钥进行异或，这里应该就是异或的逻辑了

### 0x2 unidbg

> 又到了 `unidbg debugger hook` 阶段，验证一些数据

### 0x3 unidbg console debugger zip_compress

> 先在 `0xB1B0` 下个断点

 

![](https://bbs.pediy.com/upload/attach/202111/939409_7TYBDGVXEXHTVP3.png)

 

运行代码，成功断下来，查看 `mr0` 的数据，就是我们传进来的 `url`

 

![](https://bbs.pediy.com/upload/attach/202111/939409_H5RA8QX9UF6V5ME.png)

 

![](https://bbs.pediy.com/upload/attach/202111/939409_2SZM7A4GACU34QT.png)

 

输入 `n` 指令单步执行，在查看 `r6` 的值，使用 `cyberchef` 转成 `bas64` 稍后再使用 `python` 来验证

 

![](https://bbs.pediy.com/upload/attach/202111/939409_VJX9S78EP2SR97S.png)

 

![](https://bbs.pediy.com/upload/attach/202111/939409_4V3AZYQGDB8HNMC.png)

 

这里为啥看 `r6` 的值呢，看下 `so` 就明白了 `operator new[]` 申请一块内存，存到 `r6` 寄存器，`zip_compress` 的加密结果存到了 `v9` 里也就是 `r6`

 

![](https://bbs.pediy.com/upload/attach/202111/939409_TT6H4NCDXURPWQZ.png)

 

使用 `python zlib` 模块，头和尾分别是 `2-4` 个字节去掉即可

### 0x4 unidbg console debugger aes

> 最后再来看下 `aes cbc` 使用的啥填充模式，`0x12970` 下断点

 

![](https://bbs.pediy.com/upload/attach/202111/939409_5DW8DGSU4P2GXW9.png)

 

查看 `mr0` 的值，第三行后面 11 都是填充的，模式就是 `pkcs#7`

四、response 解密
=============

> **_前面分析了，参数加密的逻辑，下面就来分析下 `response` 的解密_**

### 0x1

![](https://bbs.pediy.com/upload/attach/202111/939409_MNQV8EFRA25AYPX.png)

 

解密逻辑就是这个 `decrypt` 函数，点进去

 

![](https://bbs.pediy.com/upload/attach/202111/939409_7V999WC93BF2EXK.png)

 

需要一个参数，就是我们传进来的加密字符串然后调用 `j_Sc_EncryptWallDecode` 函数完成解密

 

![](https://bbs.pediy.com/upload/attach/202111/939409_XXG4CGCGZ5PUCUC.png)

 

这里先有个 `if` 判断，看看 `byte_3A0C0` 这是啥

 

![](https://bbs.pediy.com/upload/attach/202111/939409_H7H98U7WDRW2NGF.png)

 

按 `table` 键查看汇编代码，这里点击去

 

![](https://bbs.pediy.com/upload/attach/202111/939409_Y4XU4RYYPYEA3FZ.png)

 

这个变量是在加密逻辑返回的，所以后面在使用 `unidbg` 调用的时候还需要先调用加密逻辑才行

 

![](https://bbs.pediy.com/upload/attach/202111/939409_CA3H7TUMWZVU7JA.png)

 

进到 `EncryptWall::DecryptHttpRequest3` 函数看一下，一大堆密密麻麻的解密函数，这函数一共有 6 个参数，先来分析一下

> **_1、v4 = dword_3A0C4 = 也就是加密逻辑的返回值_**  
> **_1、v2 = 传进来的加密串_**  
> **_1、v6 = 加密字符串的长度_**  
> **_1、&v8 = 地址_**  
> **_1、0 = 0_**  
> **_1、v7 = 变量_**

### 0x2

> **_这里分析下 else 的逻辑，经过分析得知这里走的是 else 分支_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_CYTBG2PV5RFK8TD.png)

> **_v9 = (4_ a3);***
> 
> > v9 = 4 * 字符串的长度
> 
> **_v10 = operator new[](4_ a3);***
> 
> > v10 = 内存地址，大小是 字符串长度的 3 倍
> 
> **_v14 = n_crypto::Decode_Base64(v10, v9, v11, v13);_**
> 
> > 这里是 base64 decode 逻辑
> 
> **_v8 = AES_Decrypt(v12, v14, v20, v7 + 16, 32u, v7, 16u);_**
> 
> > 这里是 aes 解密，参数有 7 个分析一下  
> > **_1、v12 = 内存地址_**  
> > **_2、v14 = base64 解密结果_**  
> > **_3、v20 = v14_**  
> > **_4、v7 + 16 = 参数 1 + 16_**  
> > **_5、32u = 32 字节_**  
> > **_6、v7 = 参数一_**  
> > **_7、16u = 16 字节_**

### AES_Decrypt

> **_分析下 AES_Decrypt 的逻辑，看看 key iv 分别是啥_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_XVDMXQUB7KSS428.png)

 

有了前面分析加密逻辑的经验这里看起来大概就了解了

> **__aeabi_memcpy(&v16, a6, 16);_**
> 
> > v16 = 内存地址，a6 = 参数 6，16 = 16 个字节，这里应该是从 a6 里取出前 16 个字节的数据存到 v16 里
> 
> **_n_crypto::SetDecKeySym(&v15, v11, 8_ a5);***
> 
> > 这个应该是生成 key 的逻辑  
> > v15 = 内存地址，v11 = 参数 4， 8 _a5 = 8_ 32 = 256，这里应该是 AES-256 解密
> 
> **_n_crypto::DecSym(&v16, v9, v8, v7, &v15);_**
> 
> > 这里开始解密  
> > v16 = 内存地址，值是 16 个字节数据，v9 = 内存地址，v8 = 参数 1，v7 = 参数 2，v15 = 应该是 key，这里可以猜测 v16 = iv

 

`AES Decrypt` 的解密逻辑就分析到这里

### 0x3 zip_uncompress

> **_最后在来看下 zip_uncompress 解压缩数据的逻辑_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_3G5Y92HAYFVFFSY.png)

> **_zip_uncompress(v8 + 1,_ v20 - 4, v15, v18, 0);***  
> 主要看这行代码  
> v8 = aes 解密结果，v8 + 1 也就是地址 + 1，16 位的 1 地址 = 2 字节，32 位的 1 地址 = 4 字节，64 位的 1 地址 = 8 字节  
> 所以这里是 aes 解密结果 去掉前 四个字节 在 解压缩

 

**_Tips: so 解密逻辑分析完了，下面使用 unidbg 验证一些数据_**

### 0x4

> **_先使用 unidbg 把 解密函数跑起来_**

### 0x5 unidbg

```
public void decrypt() {
    List list = new ArrayList<>(10);
    list.add(vm.getJNIEnv());
    list.add(0);
    list.add(vm.addLocalObject(new StringObject(vm, "加密字符串")));
    Number number = module.callFunction(emulator, 0x9DA0 + 1, list.toArray())[0];
 
    System.out.println("decrypt: " + new String((byte[]) vm.getObject(number.intValue()).getValue()));
} 
```

![](https://bbs.pediy.com/upload/attach/202111/939409_GXUDKYHT5GRQNY2.png)

 

代码跑起来，发现报错了，具体因为啥，这里先不说我们继续往下分析

### 0x6 unidbg console debugger

> **_使用 console debugger 下端点调试_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_3XV4VCYE3VRYFB7.png)

 

现在分析 `byte_3A0C0` 看看加密逻辑返回的是个啥

 

![](https://bbs.pediy.com/upload/attach/202111/939409_AGMSFWER5PWZU8P.png)

 

在 `0xA35E` 地址下端点，查看 `r6` 的值，前 48 个字节 的数据事不是有些熟悉，没错正是加密时随机生成的数据  
这 `48个` 字节的 前 `16个` 是 `aes` 加密的 `iv`，后 `32个` 是 `aes` 加密的 `key`，所以猜测 `aes` 解密时用的也是这个 `key iv`，下面来验证一下

 

![](https://bbs.pediy.com/upload/attach/202111/939409_G4E5YBWRMB432R7.png)

 

`0x1158C` 下端点，查看 `r1` 寄存器的数据，前 `32个` 字节刚好是，`encrypt` 返回的结果

> **_v11 = v4 = 参数 4 = 解密逻辑结果 + 16_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_D3VFWWHRJP294A5.png)

 

在 `0x115A6` 下端点，查看 `r0` 寄存器的数据，前 `16个` 字节的数据也正好是 `encrypt` 返回的结果

> **_现在就真想大白了，参数加密用的 key iv，跟响应解密用的是同一套，如果不对的话，解密结果就不对_**  
> **_经过测试每次执行程序 加密逻辑的结果都是一样，这里随机生成的 key iv 应该也是固定的，就拿着这个加密结果去请求一次，获取到响应结果再来调用解密函数_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_WAJ6GQWSYVJERHQ.png)

 

加解密的 `key iv` 对应起来之后，`decrypt` 函数也就正常执行了

### 0x7 python

> **_再来看看 使用 python 解密_**

 

![](https://bbs.pediy.com/upload/attach/202111/939409_X8GTCPWTR5J5V3U.png)

 

这里在 `zlib.decompress` 报错了

 

![](https://bbs.pediy.com/upload/attach/202111/939409_7X8MFKN4S4XUNZW.png)

 

加个参数 `-15` 就可以了: [参考文章](https://stackoverflow.com/questions/1838699/how-can-i-decompress-a-gzip-stream-with-zlib)

> **_zlib 库可以解以下的格式_**  
> deflate 需要加上 wbits = -zlib.MAX_WBITS  
> zlib 需要加上 wbits = zlib.MAX_WBITS  
> gzip 需要加上 wbits = zlib.MAX_WBITS | 16

最后
==

> **_整个搜狗搜索 app 的加解密文章就算写完了_**  
> 原文地址: [搜狗搜索 app so 加解密分析](https://www.qinless.com/483)

[【公告】看雪团队招聘安全工程师，将兴趣和工作融合在一起！看雪 20 年安全圈的口碑，助你快速成长！](https://job.kanxue.com/position-read-1104.htm)

[#逆向分析](forum-161-1-118.htm)