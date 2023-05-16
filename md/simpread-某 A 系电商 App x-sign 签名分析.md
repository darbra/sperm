> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/c-9K_QByPH3xrQc9cPkI_Q)

一、目标

前不久 (我去，都大半年了) 分析过 某二手电商 App x-sign 签名分析 类成员变量的分析 我们找到了几个伪装成 so 的 jar 包。

虽然 rpc 调用 ok 了，但是它的实际运算过程还是在 so 里面的。

今天我们从它们同族的 App 来入手，利用 Native 层字符串定位的方式来定位下在哪个 so 中去做的运算。

App 版本: v4.15.1

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCicFQFj0jgCRInkWBFhhJboG5P7LgoQSxqFnZ6OGtVULbvPXMVk6zjnKQ/640?wx_fmt=png)1:main

二、步骤
----

### 特征字符串定位

一开始选择的特征字符串是 x- , 后来发现没有 x-sign 好使

```
Interceptor.attach(addrGetStringUTFChars, {
    onEnter: function (args) {},
    onLeave: function (retval) {
        if (retval != null) {
            var bytes = Memory.readCString(retval);
            if(bytes != null) {
                if(bytes.toString().indexOf("x-sign") >= 0 )
                {
                    console.log("[GetStringUTFChars] result:" + bytes);
                    var threadef = Java.use('java.lang.Thread');
                    var threadinstance = threadef.$new();

                    var stack = threadinstance.currentThread().getStackTrace();
                    console.log("Rc Full call stack:" + Where(stack));

                    // Native 层 堆栈
                    console.log(Thread.backtrace(this.context, Backtracer.FUZZY)
                    .map(DebugSymbol.fromAddress).join("\n"))

                }
            }

        }
    }
});

```

跑一下

```
[NewStringUTF] bytes:x-sign
Rc Full call stack:dalvik.system.VMStack.getThreadStackTrace(Native Method)
 tt: java.lang.Thread.getStackTrace(Thread.java:1538)
 tt: com.txxxao.wireless.security.adapter.JNICLibrary.doCommandNative(Native Method)
 tt: com.axxbxxx.wireless.security.mainplugin.а.doCommand(Unknown Source:0)
 tt: com.axxbxxx.wireless.security.middletierplugin.c.d.a.a(Unknown Source:280)
 tt: com.axxbxxx.wireless.security.middletierplugin.c.d.a$a.invoke(Unknown Source:56)
 tt: java.lang.reflect.Proxy.invoke(Proxy.java:913)
 tt: $Proxy12.getSecurityFactors(Unknown Source)
 tt: mtopsdk.security.d.a(lt:620)
 tt: mtopsdk.mtop.a.a.a.a.a(lt:218)
 tt: mtopsdk.framework.a.b.d.b(lt:45)
 tt: mtopsdk.framework.b.a.a.a(lt:60)

0xcb434e10 libsgmiddletierso-6.5.50.so!0x33e10
0xcb404e28 libsgmiddletierso-6.5.50.so!0x3e28
0xc9dd5536 libsgmainso-6.5.49.so!0x10536
0xc9dd71c8 libsgmainso-6.5.49.so!0x121c8
0xf365607a libart.so!art_quick_generic_jni_trampoline+0x29
0xf364068a libart.so!MterpAddHotnessBatch+0x29
0xf3651b76 libart.so!art_quick_invoke_stub_internal+0x45

```

有之前分析的基础，我们在 java 层的堆栈，重点关注 com.axxbxxx.wireless.security.middletierplugin.c.d.a.a 这个类， Native 层的堆栈就必须是 libsgmiddletierso-6.5.50.so 和 libsgmainso-6.5.49.so

### 缩小范围

jadx 打开 apk，搜索一下 com.axxbxxx.wireless.security.middletierplugin.c.d.a.a， 奇怪，这个类搜不到。

在 某二手电商 App x-sign 签名分析 类成员变量的分析 的文章里面，我们是通过 类成员变量的分析来定位的。现在我们知道了 这个类大概率是在那两个假的 so 里面

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCicb9jHKNuwQXib3AKrfnr6yibuzXIjKfyHH5LDSEoueuzMJ2Xy4iazNDvFw/640?wx_fmt=png)1:so

是的，他俩是假的 so，本质上是 jar 包， jadx 伺候

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCicy5bdj72AvwYKic1zZhu6EAtq4KtI1WQ5q7NR6ANvWYKJJyH4AohI16Q/640?wx_fmt=png)1:find

在 libsgmiddletier.so 这个 jar 包里面找到了。

### 上 Frida

```
var signCls = Java.use('com.axxbxxx.wireless.security.middletierplugin.c.d.a');
signCls.a.implementation = function(a){
    console.log(">>> sign = " + a);
    var retval = this.a(a);
    console.log(">>> sign Rc = " + retval);
    return retval;
}

```

跑一下，提示这个类找不到？

为啥找不到？因为这个 jar 包是动态加载的，所以他的 Classloader 是不同的，不能使用默认的。

### Hook 动态加载的类

#### 先要遍历一下所有的 ClassLoader

```
Java.enumerateClassLoaders({
    "onMatch": function(loader) {
        console.log(loader);
    },
    "onComplete": function() {
        console.log("success");
    }
});

```

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCick7MyoTnkVJvAMMPI6uHFwsFAdv564qAThvaOibQ5nLKzyicvZhpnhyug/640?wx_fmt=png)1:jar

没毛病就是它了。

#### 指定 ClassLoader

```
Java.enumerateClassLoaders({
    "onMatch": function(loader) {
        if (loader.toString().indexOf("libsgmiddletier.so") > 0 ) {
            Java.classFactory.loader = loader; // 将当前class factory中的loader指定为我们需要的
        }
    },
    "onComplete": function() {
        console.log("success");
    }
});

// 此处需要使用 Java.classFactory.use
var  signCls =  Java.classFactory.use('com.axxbxxx.wireless.security.middletierplugin.c.d.a');
signCls.a.overload('java.util.HashMap').implementation = function(a){
    var retval = this.a(a);
    console.log(" #### >>> a = " + a.entrySet().toArray());
    console.log(" #### >>> rc= " + retval.entrySet().toArray());

    var stack = threadinstance.currentThread().getStackTrace();
    console.log("#### >>> Rc Full call stack:" + Where(stack));

    return retval;
}

```

再跑一下，这下 Ok 了

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCic1EJUibfhLx5G6yNGZWwHLIoiaiambC9ddibI5NAPa2ricKAVLPCnRiaDNSaA/640?wx_fmt=png)1:rc

三、总结
----

我们找到了最接近的 jave 层的接口，也找到了 so 中对应的函数，但是要继续分析这个 so 还是需要费不少功夫的。

frida 提示找不到类的时候不要慌，遍历大法好。

![](https://mmbiz.qpic.cn/mmbiz_jpg/llIox45YGib2kMAz2TozwhnJrNT1X4VCic1OPTF6HnMeGSJp7uczeNg3eGQZOMt3kBlkeJ0JKeWuakmYxx6Ml7ibg/640?wx_fmt=jpeg)1:ffshow

每当年关将至，总会想起刘瑜这段话———— 忙，却似乎也没忙成什么，时间被碾得如此之碎，一阵风吹过，稀里哗啦全都不知去向，以至于我试图回想这一年到底干了些什么，发现自己简直是从一场昏迷中醒来。

###### Tip:

本文的目的只有一个就是学习更多的逆向技巧和思路，如果有人利用本文技术去进行非法商业获取利益带来的法律责任都是操作者自己承担，和本文以及作者没关系，本文涉及到的代码项目可以去 奋飞的朋友们 知识星球自取，欢迎加入知识星球一起学习探讨技术。有问题可以加我 wx: fenfei331 讨论下。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCicFjicMkd3cvRV7UOlwoqrHJ3QjulZAMIKJmnf9frgibQLWAe5SWnpna3g/640?wx_fmt=png)

关注微信公众号，最新技术干货实时推送

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2kMAz2TozwhnJrNT1X4VCicDibZuLt1qxjzjaWrkONDsAfib3aoV2zUnp9CHksJiax9CaKiaMv0rJ0S0A/640?wx_fmt=png)

 手机查看不方便，可以网页看

    http://91fans.com.cn

往期推荐

[

某汽车社区 App 签名和加解密分析 (二) : Frida Dump so



](http://mp.weixin.qq.com/s?__biz=MzI4NzMwNDI3OQ==&mid=2247484542&idx=1&sn=2322a3e5f86aa09d9998eea85e449a28&chksm=ebcefb46dcb972501805f25453b37d9ac6e31e16a9bf2495d0741cfb4c8d6de4fa0d0801c05f&scene=21#wechat_redirect)

[

某二手电商 App x-sign 签名分析 类成员变量的分析



](http://mp.weixin.qq.com/s?__biz=MzI4NzMwNDI3OQ==&mid=2247483987&idx=1&sn=4e8b2556e4ae9cda9ff64b5123c63b5e&chksm=ebcefd6bdcb9747dde38bbcda1fef251dfd5df901dc1f7b2cdd156136720af59b48dc556356b&scene=21#wechat_redirect)