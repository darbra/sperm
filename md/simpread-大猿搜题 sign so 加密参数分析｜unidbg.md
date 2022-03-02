> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2NjcxNzE3NQ==&mid=2247483906&idx=1&sn=712447f635fb957b0f7d47d7333b865a&chksm=ce47de9af930578c2c3e4181b64de465be78bcc9096731eb6f3eb599768e69f70755bb3bff2e&mpshare=1&scene=1&srcid=0302EmxYsDsZprkUyCAuJRqP&sharer_sharetime=1646203747659&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

前言
==

> ❝
> 
> 「样本: https://www.wandoujia.com/apps/6526244」  
> 「版本: 11.4.3」
> 
> ❞

charles
=======

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3ia14OibwPCCXjEYiaoy6qQT6SdicUibSWGia7hHsYS6HYkIqAiaHC3pL7Nl5Q/640?wx_fmt=png)

随便一个接口 `sign` 参数

「这个 app 有 360 加固，需要先脱壳」

dump dex
========

「使用的 yang 神的 frida-dump https://github.com/lasting-yang/frida_dump」

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3vhaSbCvKCicqNHKczbkzibml3OpNPnkwodVRK2icggHwuQqJZ0Fica7vicA/640?wx_fmt=png)

主要如果报了这个错误，需要给 `app` 文件的读权限

```
import os
import zipfile
import argparse


def rename_class(path):
    files = os.listdir(path)
    dex_index = 0
    if path.endswith('/'):
        path = path[:-1]
        print(path)
    for i in range(len(files)):
        if files[i].endswith('.dex'):
            old_name = path + '/' + files[i]
            if dex_index == 0:
                new_name = path + '/' + 'classes.dex'
            else:
                new_name = path + '/' + 'classes%d.dex' % dex_index
            dex_index += 1
            if os.path.exists(new_name):
                continue
            os.rename(old_name, new_name)
    print('[*] 重命名完毕')


def extract_META_INF_from_apk(apk_path, target_path):
    r = zipfile.is_zipfile(apk_path)
    if r:
        fz = zipfile.ZipFile(apk_path, 'r')
        for file in fz.namelist():
            if file.startswith('META-INF'):
                fz.extract(file, target_path)
    else:
        print('[-] %s 不是一个APK文件' % apk_path)


def zip_dir(dirname, zipfilename):
    filelist = []
    if os.path.isfile(dirname):
        if dirname.endswith('.dex'):
            filelist.append(dirname)
    else:
        for root, dirs, files in os.walk(dirname):
            for dir in dirs:
                # if dir == 'META-INF':
                # print('dir:', os.path.join(root, dir))
                filelist.append(os.path.join(root, dir))
            for name in files:
                # print('file:', os.path.join(root, name))

                filelist.append(os.path.join(root, name))

    z = zipfile.ZipFile(zipfilename, 'w', zipfile.ZIP_DEFLATED)
    for tar in filelist:
        arcname = tar[len(dirname):]

        if ('META-INF' in arcname or arcname.endswith('.dex')) and '.DS_Store' not in arcname:
            # print(tar + " -->rar: " + arcname)
            z.write(tar, arcname)
    print('[*] APK打包成功，你可以拖入APK进行分析啦！')
    z.close()


if __name__ == '__main__':
    args = {
        'dex_path': '/Users/admin/Desktop/project/user/user-frida-hook/hook/frida_dump_dex/unpack/xiaoyuansouti',
        'apk_path': '/Volumes/T7/android/android-file/xiaoyuansouti-11.4.3.apk',
        'output': '/Volumes/T7/android/android-file/xiaoyuansouti-new-11.4.3.apk'
    }

    rename_class(args['dex_path'])
    extract_META_INF_from_apk(args['apk_path'], args['dex_path'])
    zip_dir(args['dex_path'], args['output'])


```

脱完壳之后再使用以上脚本，合并 `dex` 生成新的 `apk` 文件，在反编译

java 层分析
========

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3vH4wUGvx5TFiax3uVibibatdDhqb8zK3r9DALFsOsLTYgx8wM7AZIialzg/640?wx_fmt=png)

全局搜索 `sign` 关键词，定位到这个函数 `com.fenbi.android.leo.imgsearch.sdk.network.intercepto.SignInterceptor.intercept`

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3oMNTOicel2n2Rb7dtPeFFtMzllNQVia2ZctF4m3gXmCJiakzxstxY7ZdQ/640?wx_fmt=png)

继续往下分析，来到了这里 `com.fenbi.android.solar.util.bn`

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB31V59nHpHbZdh79icP7jdZyK9DZ3hiaOz8pMNaiahtJ0pwe4hRMuwnQoIg/640?wx_fmt=png)

在继续就来到了 `native` 函数处，会调用这个三个函数

frida hook
==========

```
function getProcessId() {
    var androidProcess = Java.use('android.os.Process');
    return 'processId: ' + androidProcess.myTid() + ' - ';
}


function printLog() {
    var result = '';

    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i] + ' ';
    }

    console.log(getProcessId(), result)
}


function hook() {
    var eClass = Java.use('com.fenbi.android.solar.util.e');

    eClass.zcvsd1wr2t.implementation = function (a, b, c, d) {
        printLog('eClass.zcvsd1wr2t.a: ', a);
        printLog('eClass.zcvsd1wr2t.c: ', c);
        printLog('eClass.zcvsd1wr2t.c: ', c);
        printLog('eClass.zcvsd1wr2t.d: ', d);

        var res = this.zcvsd1wr2t(a, b, c, d);
        printLog('eClass.zcvsd1wr2t.res: ', res);

        return res;
    }

    eClass.awi6d8sdfe.implementation = function (a, b, c, d) {
        printLog('eClass.awi6d8sdfe.a: ', a);
        printLog('eClass.awi6d8sdfe.c: ', c);
        printLog('eClass.awi6d8sdfe.c: ', c);
        printLog('eClass.awi6d8sdfe.d: ', d);

        var res = this.awi6d8sdfe(a, b, c, d);
        printLog('eClass.awi6d8sdfe.res: ', res);

        return res;
    }

    eClass.fxc3fs3red.implementation = function (a, b, c, d) {
        printLog('eClass.fxc3fs3red.a: ', a);
        printLog('eClass.fxc3fs3red.c: ', c);
        printLog('eClass.fxc3fs3red.c: ', c);
        printLog('eClass.fxc3fs3red.d: ', d);

        var res = this.fxc3fs3red(a, b, c, d);
        printLog('eClass.fxc3fs3red.res: ', res);

        return res;
    }
}


Java.perform(function () {
    hook();
})


```

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB33xPSvWKseWurvBuqKobNM0cxvWetatl70mF6eYjyH9vP3faGDo0Z9A/640?wx_fmt=png)

执行脚本，结果保存到文件里，函数都 `hook` 成功

unidbg
======

「老规矩，先搭建架子」

```
package com.xiayu.xiaoyuansouti;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.AbstractJni;
import com.github.unidbg.linux.android.dvm.DalvikModule;
import com.github.unidbg.linux.android.dvm.DvmClass;
import com.github.unidbg.linux.android.dvm.VM;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.virtualmodule.android.AndroidModule;

import java.io.File;
import java.io.IOException;

public class GetSign_v1143Test extends AbstractJni {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    public DvmClass EClass;
    public String apkPath = "/Volumes/T7/android/android-file/xiaoyuansouti-11.4.3.apk";

    GetSign_v1143Test() {
        emulator = AndroidEmulatorBuilder.for32Bit().build();
        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));
        vm = emulator.createDalvikVM(new File(apkPath));
        vm.setVerbose(true);
        DalvikModule dm = vm.loadLibrary("RequestEncoder", true);

        vm.setJni(this);
        module = dm.getModule();
        dm.callJNI_OnLoad(emulator);

        EClass = vm.resolveClass("com/fenbi/android/solar/util/e");
    }

    public static void main(String[] args) {
        GetSign_v1143Test getSign_v1143Test = new GetSign_v1143Test();
        
        getSign_v1143Test.destroy();
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

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB32IEubBibQAKXZrK95xU0KOMX22wlicemJ31cDYLDrQbTPMNwyzyhV0sw/640?wx_fmt=png)

跑起来，成功报错，`libandroid.so` 咱们加一下

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3Wxa6ahWhbKDtWovYIPMVtkZYmezMF0rsE0M4Qtqe5mzBHI4wdkeo4g/640?wx_fmt=png)

加上在跑起来，这次报了环境错误，补上就 OK 了

```
public void call_zcvsd1wr2t() {
    String methodId = "zcvsd1wr2t(Ljava/lang/String;Ljava/lang/String;ILjava/lang/String;)Ljava/lang/String;";
    EClass.callStaticJniMethodObject(
            emulator, methodId,
            new StringObject(vm, "/ape-news/android/news"),
            new StringObject(vm, "0"),
            0,
            new StringObject(vm, "")
    );
}


```

直接来 `call` 目标函数

「中间省略一部分补环境操作。。。。。。」

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3Oia2JX3SyA765gyWCV8JsSZqm71VGE3jFfpW6Ix4gwvLOjR2ZjknQ8g/640?wx_fmt=png)

这个错误，要求返回 `Signature array`，这是获取 `app` 签名，需要返回 `array` 注意不要直接 `vm.resolveClass("[android/content/pm/Signature")` 可以使用 `ArrayObject` 构建返回，这样可以避免很多问题

```
CertificateMeta[] certificateMetas = vm.getSignatures();

DvmObject<?>[] dvmObjects = new DvmObject[certificateMetas.length];

for (int i = 0; i < certificateMetas.length; i++) {
    dvmObjects[i] = vm.resolveClass("android/content/pm/Signature").newObject(certificateMetas[i]);
}

return new ArrayObject(dvmObjects);


```

具体代码如上，补上继续跑起来

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB32UZPoIic41DYnkcPlicPtfqsqQUrqCaEKOD97PBMDsuB9ia3fGIyZibicLw/640?wx_fmt=png)

这个是调用 `Signature.toChars` 函数，返回 `char[]` 数据

```
CertificateMeta certificateMeta = (CertificateMeta) dvmObject.getValue();

byte[] bytes = certificateMeta.getData();

DvmObject<?>[] dvmObjects = new DvmObject[bytes.length];

for (int i = 0; i < bytes.length; i++) {
    dvmObjects[i] = vm.resolveClass("C").newObject((char) bytes[i]);
}

return new ArrayObject(dvmObjects);


```

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3lF8VwpEK7dmjkgksBs1zichV9hkUlhI0iafHENJR5JlMgFenz7EjEPrg/640?wx_fmt=png)

补完，在跑起来，发现又报错了，不知道啥错误，点进 `handler` 里看看

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3ibt3TX2ajTYKrDVSlr4FNHiajLDcwnLf5nKgiahXC3LGLLXm8pvDzs0pg/640?wx_fmt=png)

是个未实现的 `jni` 函数，那咱们就去实现一下，参照 `GetxxxxArrayElements` 代码逻辑，写一个

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3pSLTqITC2tzoIK3hMu0js8DopFCbXuNsUIhkH6UJAstDt6J6AEwX0g/640?wx_fmt=png)

比如这个 `GetIntArrayElements` 函数逻辑，通过 `getObject` 获取对象，在调用 `_GetArrayCritical` 函数，但是 `char[]` 没有实现对应的 `CharArray`，有两种方法

*   1、自己根据 `GetxxxxArrayElements` 逻辑写一个
    
*   2、实现 `CharArray` 跟其他的 `xxxArray` 逻辑保持一致
    

推荐方法 2，这样写后面可以避免很多问题（龙哥的精讲课里也提到过，基本数据的构建最好使用 unidbg 提供的 class）

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3hSgTJUGE1CdiczSLkhoibNeqRicvc4aBlIcbticshGicxbuJdFOKqXWl7GQ/640?wx_fmt=png)

新建个 `CharArray` 文件，代码直接复制其他的 `array` 改改数据类型就 OK 了

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3KbIdtT97Fr87pwxIXhECvibQ4x6wrf0QKh9KgibuIVkd0TpJE80mMgBw/640?wx_fmt=png)

刚刚的环境，咱们使用 `CharArray` 来构建

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3kBq0kTFyO1QLteicbJydQqicxYD7LW5kPnMCibib6KpTGSVBdwQl8eOKqg/640?wx_fmt=png)

`jni` 这里也是使用，同样的逻辑返回，继续跑起来，还会报个错误

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB394iaLpAw8k17nBGxrvb6DZ2viawhywAC7TbUgzncyHXpWuBdK1e50veA/640?wx_fmt=png)

这里在 `write` 的时候，循环调用 `setChar` 函数即可

![](https://mmbiz.qpic.cn/mmbiz_png/MWQJibR3pCZ8e0djy9O5wmvFHHlWWXbB3a0viaemWcw7x8ZWqeFOAfBoLOJQRKNic5FibyOY10l3X8KVuZzzl5aFhg/640?wx_fmt=png)

最后在补个环境，也是成功运行出了结果