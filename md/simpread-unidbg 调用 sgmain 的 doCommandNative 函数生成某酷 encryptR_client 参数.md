> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1680816-1-1.html)

> [md]@[TOC](unidbg 调用 sgmain 的 doCommandNative 函数生成优酷 encryptR_client 参数)## 前置工作 1. 雷电 4 模拟器（安装 frida-server）2.Get......

![](https://avatar.52pojie.cn/data/avatar/000/55/71/95_avatar_middle.jpg) 漁滒

@[TOC](unidbg调用sgmain的doCommandNative函数生成优酷encryptR_client参数)

前置工作
----

1. 雷电 4 模拟器（安装 frida-server）  
2.GetVideo（MD5: 72641720EC9D489FC9AB4B90BBA1D100）  
3.HttpCanary 或者 Fiddler（可以随意，非必须）  
4.jadx-gui  
5.IDEA（拉取 unidbg 项目）  
6.python（安装 frida 相关依赖）

app 抓包
------

本次使用的并非官方 app，而是使用的一款名为 GetVideo 的视频解析软件。所以里面也设置到了这个参数的生成，模拟器使用的是雷电 4。关于雷电 4 模拟器安装 frida 可以参考我之前的帖子。  
↓↓↓↓

[雷电模拟器安装 frida-server 教程](https://www.52pojie.cn/thread-1344344-1-1.html)

app 安装完成后，需要点击右上角的三个点，进去设置。然后往下拉到优酷的设置中，把所有开关打开

![](https://img-blog.csdnimg.cn/5289b81645cd44f29de2b246fdd42ae3.jpeg#pic_center)  
然后打开 HttpCanary，选中 GetVideo 进行抓包，然后在优酷的解析界面点击 TV 的解析

![](https://img-blog.csdnimg.cn/4c0b76fc36304666a6d82a1b93d9da3d.jpeg#pic_center)  
等待下方出现解析结果后，返回 HttpCanary，里面路径【get.json】结尾的就是需要的包  
![](https://img-blog.csdnimg.cn/279060102a6141c8bc8aa3620860c938.jpeg#pic_center)  
点开这个包继续查看，里面的这个 encryptR_client 参数，就是本次需要生成的参数

![](https://img-blog.csdnimg.cn/0112fa0e21514863b40c85c50c57eeee.jpeg#pic_center)

java 层代码分析
----------

使用 jadx-gui 反编译目标 app，直接搜索【encryptR_client】

![](https://img-blog.csdnimg.cn/d98262d4b5494947a9d7fef5d4928103.jpeg#pic_center)  
非常好，直接三个结果。第二个结果是定值，第一第三个是一样的，那就点第一个结果进去。

![](https://img-blog.csdnimg.cn/dbfc7900aaac4993af38843924ecd556.jpeg#pic_center)  
然后又是调用了 staticSafeEncrypt 函数，继续点进去。

![](https://img-blog.csdnimg.cn/fb268c76130f4b5b9f9517ab39f8f96f.jpeg#pic_center)

这时就来到了一个抽象类，那么必须找到实现类才有真正的函数体，再看包名，是来自于【com.alibaba.wireless.security】，这个是阿里聚安全 SDK（貌似三方 SDK 的包名是不能够被混淆的），那么这个的实现类是在【main】这个插件里面。

打开 app 的【lib\armeabi-v7a】目录，并把里面的【libsgmain.so】解压出来

![](https://img-blog.csdnimg.cn/1c71da5cfb69400c9fb07c9cf2ee6817.jpeg#pic_center)

使用 jadx-gui 的添加文件来添加这个 so

![](https://img-blog.csdnimg.cn/0b53eaa8f38c434ea67d5259d4672f7d.jpeg#pic_center)  
等待重新加载后再次搜索【staticSafeEncrypt】

![](https://img-blog.csdnimg.cn/1a651962dacf4483a1b44b95f42a77c2.jpeg#pic_center)  
这次就可以搜索到较多的结果了，经过分析，实现类在【com.alibaba.wireless.security.open.staticdataencrypt】中。

![](https://img-blog.csdnimg.cn/dac2b10d49f4401ba0b74e6232ebbd47.jpeg#pic_center)  
调用了里面的 a 方法，继续跟进去

![](https://img-blog.csdnimg.cn/fc94403ae7634df9adf89e9514014bea.jpeg#pic_center)  
然后是调用了【doCommand】方法，继续往里面跟

![](https://img-blog.csdnimg.cn/597b3e3255fa4344acb7dc23dde3285a.jpeg#pic_center)  
这时和之前一样，又来到了一个抽象类，继续查找它的实现类，搜索【implements IRouterComponent】

![](https://img-blog.csdnimg.cn/67a18db436f143d59eb40d73c12b75d5.jpeg#pic_center)  
很舒服，只有一个结果，直接进去看

![](https://img-blog.csdnimg.cn/6fd7473594cb400fb04e98fbc0cbfda9.jpeg#pic_center)  
这里接着调用了【doCommandNative】方法，继续走

![](https://img-blog.csdnimg.cn/bfe8a8997e45410c85bf819fad594286.jpeg#pic_center)  
来到这里就没法继续走了，到了 so 层的函数了，那么使用 unidbg 调用的就是这个函数了。

frida hook 加 unidbg 调用
----------------------

拉 unidbg 后简单写一个 demo

```
package com.taobao.wireless.security.adapter;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;
import java.io.File;
import java.io.IOException;

public class JNICLibrary_test{
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private final DvmClass JNICLibrary;
    private final boolean logging;

    public JNICLibrary_test(boolean logging) {
        this.logging = logging;
        emulator = AndroidEmulatorBuilder.for32Bit().setRootDir(new File("root")).setProcessName("com.wuxianlin.getvideo").build();
        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/java/com/taobao/wireless/security/adapter/GetVideo.apk"));
        vm.setVerbose(logging);

        DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/java/com/taobao/wireless/security/adapter/libsgmainso-6.4.170.so"), true);
        dm.callJNI_OnLoad(emulator);
        module = dm.getModule();

        JNICLibrary = vm.resolveClass("com/taobao/wireless/security/adapter/JNICLibrary");
    }

    public void destroy() throws IOException {
        emulator.close();
        if (logging) {
            System.out.println("destroy");
        }
    }

    public static void main(String[] args) throws Exception {
        JNICLibrary_test CLibrary = new JNICLibrary_test(true);
        CLibrary.encrypt();
        CLibrary.destroy();
    }

    public void encrypt() {

    }

}

这里加载的就不是我们从apk拉出来的【libsgmain.so】，而是名字类似的【libsgmainso-6.4.170.so】，

```

![](https://img-blog.csdnimg.cn/4a9508660282483c9ce6205a2801054e.jpeg#pic_center)  
不出意外，会出现补环境相关的问题，这里自行补全。

![](https://img-blog.csdnimg.cn/b6d8958ed90142f3a8c285ff6a712ca7.jpeg#pic_center)  
完成初始化后就尝试调用目标函数，那么调用的参数是什么呢？这时就需要用 frida 帮助我们了。

根据前面的内容，我们知道目标函数是第一个整数是 10601 的函数，使用 frida 可以 hook 到下面内容

```
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.open.staticdataencrypt.a.a(Unknown Source)
    at com.alibaba.wireless.security.open.staticdataencrypt.a.staticSafeEncrypt(Unknown Source)
    at d.h.a.e.m.a()
    at d.h.a.e.l.d(:2)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10601
传入参数：b
['<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: [B>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: [B>
<instance: java.lang.Object, $className: java.lang.String>
1
16
1
23570660
长  度：6
字节集：[49, 50, 51, 52, 53, 54]
字符串：b'123456'
Base64：MTIzNDU2
HEX   ：313233343536

结果：
<instance: java.lang.Object, $className: [B>
长  度：24
字节集：[68, 78, 89, 76, 47, 78, 66, 47, 120, 56, 103, 112, 122, 74, 102, 75, 121, 83, 66, 52, 56, 81, 61, 61]
字符串：b'DNYL/NB/x8gpzJfKySB48Q=='
Base64：RE5ZTC9OQi94OGdwekpmS3lTQjQ4UT09
HEX   ：444e594c2f4e422f783867707a4a664b7953423438513d3d
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

```

可以看到入参的类型和结果，尝试使用 unidbg 运行，在自己写的 encrypt 函数内增加下面内容

```
        DvmObject<?> dvmObject = JNICLibrary.callStaticJniMethodObject(
                emulator,
                "doCommandNative(I[Ljava/lang/Object;)Ljava/lang/Object;",
                10601,
                vm.addLocalObject(new ArrayObject(
                        DvmInteger.valueOf(vm, 1),
                        DvmInteger.valueOf(vm, 16),
                        DvmInteger.valueOf(vm, 1),
                        new StringObject(vm, "23570660"),
                        new ByteArray(vm, "123456".getBytes()),
                        new StringObject(vm, "")
                ))
        );
        System.out.println("staticSafeEncrypt: " + dvmObject.getValue());

```

![](https://img-blog.csdnimg.cn/284efd4b955644618ddecaef5c8a6948.jpeg#pic_center)  
这里出现报错了，虽然这是一个补 jni 环境，但是这是在 java 层抛出一个异常。但是参数是没有错的，怎么会这样呢。

在这里就要考虑是不是有一些初始化的东西没有做了，但是 so 这里就只有一个函数，没有 init 名字相关的函数，那这时就要考虑初始化函数是不是自己本身，那么就要 hook 从 so 加载一直到调用目标函数的所有调用

```
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.mainplugin.SecurityGuardMainPlugin.onPluginLoaded(Unknown Source)
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.d()
    at com.alibaba.wireless.security.framework.d.getPluginInfo()
    at com.alibaba.wireless.security.open.initialize.b.a()
    at com.alibaba.wireless.security.open.initialize.a.loadLibrarySync()
    at com.alibaba.wireless.security.open.initialize.a.loadLibrarySync()
    at com.alibaba.wireless.security.open.initialize.a.initialize()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInstance()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInstance()
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10101
传入参数：b
['<instance: java.lang.Object, $className: com.wuxianlin.getvideo.MainApplication>', '<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: com.wuxianlin.getvideo.MainApplication>
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
3

/data/user/0/com.wuxianlin.getvideo/app_SGLib

结果：
<instance: java.lang.Object, $className: java.lang.Integer>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.d()
    at com.alibaba.wireless.security.framework.d.getPluginInfo()
    at com.alibaba.wireless.security.open.initialize.b.a()
    at com.alibaba.wireless.security.open.initialize.a.loadLibrarySync()
    at com.alibaba.wireless.security.open.initialize.a.loadLibrarySync()
    at com.alibaba.wireless.security.open.initialize.a.initialize()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInstance()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInstance()
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10102
传入参数：b
['<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
main
6.4.170
/data/app/com.wuxianlin.getvideo-1/lib/arm/libsgmainso-6.4.170.so
结果：
<instance: java.lang.Object, $className: java.lang.Integer>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.d()
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.d()
    at com.alibaba.wireless.security.framework.d.getPluginInfo()
    at com.alibaba.wireless.security.framework.d.getInterface()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInterface()
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10102
传入参数：b
['<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
securitybody
6.4.99
/data/app/com.wuxianlin.getvideo-1/lib/arm/libsgsecuritybodyso-6.4.99.so
结果：
<instance: java.lang.Object, $className: java.lang.Integer>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.framework.d.a()
    at com.alibaba.wireless.security.framework.d.d()
    at com.alibaba.wireless.security.framework.d.getPluginInfo()
    at com.alibaba.wireless.security.framework.d.getInterface()
    at com.alibaba.wireless.security.open.SecurityGuardManager.getInterface()
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10102
传入参数：b
['<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
avmp
6.4.36
/data/app/com.wuxianlin.getvideo-1/lib/arm/libsgavmpso-6.4.36.so
结果：
<instance: java.lang.Object, $className: java.lang.Integer>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.avmpplugin.a.a$a.a(Unknown Source)
    at com.alibaba.wireless.security.avmpplugin.a.a.createAVMPInstance(Unknown Source)
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
60901
传入参数：b
['<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.String>
mwua_uuid
sgcipher
结果：
<instance: java.lang.Object, $className: java.lang.Long>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.avmpplugin.a.a$a.invokeAVMP(Unknown Source)
    at d.h.a.e.l.d(:1)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
60902
传入参数：b
['<instance: java.lang.Object, $className: java.lang.Long>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: java.lang.Class>', '<instance: java.lang.Object, $className: [Ljava.lang.Object;>']
<instance: java.lang.Object, $className: java.lang.Long>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: java.lang.Class>
<instance: java.lang.Object, $className: [Ljava.lang.Object;>
2955493818
sign
class [B
<instance: java.lang.Object, $className: [Ljava.lang.Object;>
[2,[99,99,111,100,101,61,48,49,48,51,48,49,48,49,48,50,38,99,108,105,101,110,116,95,105,112,61,49,57,50,46,49,54,56,46,49,46,49,38,99,108,105,101,110,116,95,116,115,61,49,54,54,49,53,55,48,51,51,51,38,117,116,105,100,61,89,119,109,78,72,82,66,65,113,74,119,68,65,78,99,117,53,84,86,117,49,118,55,75,38,118,105,100,61,88,77,106,107,52,79,68,65,121,77,122,73,121,79,65,61,61],111,"",[0,0,0,0],0]
结果：
<instance: java.lang.Object, $className: [B>
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
java.lang.Exception
    at com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
    at com.alibaba.wireless.security.mainplugin.a.doCommand(Unknown Source)
    at com.alibaba.wireless.security.open.staticdataencrypt.a.a(Unknown Source)
    at com.alibaba.wireless.security.open.staticdataencrypt.a.staticSafeEncrypt(Unknown Source)
    at d.h.a.e.m.a()
    at d.h.a.e.l.d(:2)
    at com.wuxianlin.getvideo.ui.YoukuFragment$d$a.run(:2)
    at java.lang.Thread.run(Thread.java:761)

传入参数：a
10601
传入参数：b
['<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.Integer>', '<instance: java.lang.Object, $className: java.lang.String>', '<instance: java.lang.Object, $className: [B>', '<instance: java.lang.Object, $className: java.lang.String>']
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.Integer>
<instance: java.lang.Object, $className: java.lang.String>
<instance: java.lang.Object, $className: [B>
<instance: java.lang.Object, $className: java.lang.String>
1
16
1
23570660
长  度：6
字节集：[49, 50, 51, 52, 53, 54]
字符串：b'123456'
Base64：MTIzNDU2
HEX   ：313233343536

结果：
<instance: java.lang.Object, $className: [B>
长  度：24
字节集：[68, 78, 89, 76, 47, 78, 66, 47, 120, 56, 103, 112, 122, 74, 102, 75, 121, 83, 66, 52, 56, 81, 61, 61]
字符串：b'DNYL/NB/x8gpzJfKySB48Q=='
Base64：RE5ZTC9OQi94OGdwekpmS3lTQjQ4UT09
HEX   ：444e594c2f4e422f783867707a4a664b7953423438513d3d
↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑


```

调用的次数不少，整理一下就是如下图

![](https://img-blog.csdnimg.cn/26d1921467a64980a69cf13fecc96f10.jpeg#pic_center)  
按照上面的逻辑顺序去调用 doCommandNative 函数

![](https://img-blog.csdnimg.cn/e281e4ec1ff9463192ed0b9bdfb644be.jpeg#pic_center)  
出现一些补环境的问题，这里就跳过了

![](https://img-blog.csdnimg.cn/b7ed785a1e0c496eb1e179ab28af81d8.jpeg#pic_center)  
接着会出现 libc 的 pthread_create failed，那这里就要开启多线程

![](https://img-blog.csdnimg.cn/a4444bf415064bc6b158859630efe989.jpeg#pic_center)  
接着会出现一个【com/uc/crashsdk/JNIBridge】的类，但是我在 java 层并没有找到这个类，也就是说这个类很有可能是不存在的，但是 unidbg 没有办法知道 java 层是否真的有这个类，默认都是找到类的，那么应该怎么处理呢

在全局搜索【FindClass】，跳转到 DalvikVM 这个类里面

![](https://img-blog.csdnimg.cn/34f49ca9f70a4ea49218109503d65e97.jpeg#pic_center)

可以看到上面还有一个【notFoundClassSet】的处理，点击跳转进去，来到 BaseVM

![](https://img-blog.csdnimg.cn/173b40d1668543bbb3189a784fee6e1a.jpeg#pic_center)  
可以看到有一个 addNotFoundClass 的函数给我们进行调用，那么就可以在生成 vm 后进行下面调用

```
vm.addNotFoundClass("com/uc/crashsdk/JNIBridge");

```

此时再运行，就不会出现这个补环境，并且从 jni 日志看到下面内容

```
JNIEnv->FindNoClass(com/uc/crashsdk/JNIBridge) was called from RX@0x40048297[libmain.so]0x48297

```

![](https://img-blog.csdnimg.cn/496ccbc0183849cbbce571989387ef46.jpeg#pic_center)  
so 的初始化就完成，接着处理插件的初始化

![](https://img-blog.csdnimg.cn/6a064f8f90504641832db36cbf497afc.jpeg#pic_center)  
下一步就是生成 vmp 的函数

![](https://img-blog.csdnimg.cn/71d65257259b4b8cbc031ad573adbca3.jpeg#pic_center)  
这里出现一个 1905 的报错，在 app 中查看这个报错的原因

![](https://img-blog.csdnimg.cn/9221b807504b48a5b45b03444cc39e4b.jpeg#pic_center)  
这里说了是一个找不到文件的错误，也就是说我们需要处理文件 io

![](https://img-blog.csdnimg.cn/27291d3faaf048c2a7c73242244e9864.jpeg#pic_center)  
处理好文件访问后，就可以调用成功了，跟着调用 invokeAVMP 函数

```
long createAVMPInstance = (Long) dvmObject.getValue();
        System.out.println("createAVMPInstance: " + createAVMPInstance);
        System.out.println("createAVMPInstance: " + Long.toHexString(createAVMPInstance));
        String ccode = "ccode=0103010102&client_ip=192.168.1.1&client_ts=" + (System.currentTimeMillis() / 1000) + "&utid=YwUOJY66/6oDANcu5TV0KQWW&vid=XMjk4ODAyMzIyOA==";
        dvmObject = JNICLibrary.callStaticJniMethodObject(
                emulator,
                "doCommandNative(I[Ljava/lang/Object;)Ljava/lang/Object;",
                60902,
                vm.addLocalObject(new ArrayObject(
                        DvmLong.valueOf(vm, createAVMPInstance),
                        new StringObject(vm, "sign"),
                        vm.resolveClass("[B"),
                        new ArrayObject(
                                DvmInteger.valueOf(vm, 2),
                                new ByteArray(vm, ccode.getBytes()),
                                DvmInteger.valueOf(vm, ccode.length()),
                                new StringObject(vm, ""),
                                new ByteArray(vm, new byte[4]),
                                DvmInteger.valueOf(vm, 0)
                        )
                ))
        );

```

![](https://img-blog.csdnimg.cn/2a03e805838f48d2b6fd77f29a60d9f9.jpeg#pic_center)  
这次出现的是 1900 的错误

![](https://img-blog.csdnimg.cn/a01999fccfb34acda7f042ae47d62890.jpeg#pic_center)  
这里只说了是 VMP 错误，但是没有详细信息。我也不知道怎么搞了，但是经过分析发现，这个并非是目标函数的前置函数，也就是说这个函数可以不调用。

如果哪位大佬知道这里报错 1900 的源码，麻烦评论区回复一下，先感谢了

那么接下来直接调用目标函数

![](https://img-blog.csdnimg.cn/b496142a482f4e4989d43626f527445c.jpeg#pic_center)  
这次终于调用成功了，并且结果与 hook 的是一致的，说明调用没有错误。完美，散花！！！

![](https://avatar.52pojie.cn/data/avatar/001/43/64/20_avatar_middle.jpg) _ 本帖最后由 OVVO 于 2022-8-27 12:32 编辑_  
![](https://attach.52pojie.cn/forum/202208/27/123237hiqo0oou30hy12zm.jpg)

**QQ 图片 20220827123236.jpg** _(1.99 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU0OTQ4MnwwYTM3YTkwZnwxNjYxNjA4NDAxfDE0MTAxOTh8MTY4MDgxNg%3D%3D&nothumb=yes)

2022-8-27 12:32 上传

![](https://avatar.52pojie.cn/data/avatar/001/43/64/20_avatar_middle.jpg)OVVO  ![](https://attach.52pojie.cn/forum/202208/27/123431k3ha3shbu4es2vay.png)

**QQ 图片 20220827123436.png** _(4.2 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU0OTQ4M3wyMzg4ODNhYXwxNjYxNjA4NDAxfDE0MTAxOTh8MTY4MDgxNg%3D%3D&nothumb=yes)

2022-8-27 12:34 上传

![](https://attach.52pojie.cn/forum/202208/27/123451eeg5ys5qq5ery5zy.jpg)

**QQ 图片 20220827123456.jpg** _(1.65 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU0OTQ4NHxmNmY0OGZlZnwxNjYxNjA4NDAxfDE0MTAxOTh8MTY4MDgxNg%3D%3D&nothumb=yes)

2022-8-27 12:34 上传

![](https://avatar.52pojie.cn/data/avatar/001/10/94/58_avatar_middle.jpg)OVVO ![](https://img.lxtx.top/i/2022/08/27/6309aacece01c.png)![](https://avatar.52pojie.cn/data/avatar/000/35/89/70_avatar_middle.jpg)正己 看到某酷我以为是酷安。。。![](https://avatar.52pojie.cn/data/avatar/000/15/24/83_avatar_middle.jpg) 好久没人分享大作了![](https://avatar.52pojie.cn/data/avatar/001/62/57/99_avatar_middle.jpg) Light 紫星 看到某酷我以为是酷安。。。![](https://avatar.52pojie.cn/data/avatar/000/37/54/56_avatar_middle.jpg)不苦小和尚 这么麻烦，还是放弃吧  
![](https://avatar.52pojie.cn/data/avatar/000/69/36/16_avatar_middle.jpg)im 热情 鱼大，GetVideo 呢![](https://avatar.52pojie.cn/data/avatar/000/69/36/16_avatar_middle.jpg) wakichie _ 本帖最后由 ofo 于 2022-8-27 16:38 编辑_  
![](https://attach.52pojie.cn/forum/202208/27/163352qi1p0tj514i71p2p.png)

**QQ 图片 20220827163329.png** _(32.33 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU0OTUyOHxlYzNiNTc3MnwxNjYxNjA4NDAxfDE0MTAxOTh8MTY4MDgxNg%3D%3D&nothumb=yes)

2022-8-27 16:33 上传

  
我做错什么了？![](https://static.52pojie.cn/static/image/smiley/default/24.gif)