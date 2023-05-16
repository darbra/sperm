> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bKq8x6m2u0Gx6xoPSO3JuA)

一、目标

李老板： 奋飞呀，x-sign 你都水了好几篇了，一直在 Apk 里面打转，咱们啥时候分析分析它的 so？

奋飞: 循序渐进嘛，我们上次刚定位了它的 so，今天我们来分析分析。

App 版本: v4.15.1

二、步骤
----

### Native 层的入口

先回忆下这个堆栈

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

堆栈会说话的，他告诉我们

1、jni 函数叫 com.txxxao.wireless.security.adapter.JNICLibrary.doCommandNative。

2、doCommandNative 的实现在 libsgmainso-6.5.49.so 中，可能在偏移 0x121c8 附近。

### 先 Hook jni 函数一把

jni 函数会告诉我们入参和返回值的类型，所以不能放过。

这个 jni 函数的声明在 libsgmain.so 这个假 so 里面

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBNnnt5ZVThzJhlAib4yaDv9ceG4FLickKFvIJZw2zzFOffict9YSmFExRA/640?wx_fmt=png)1:docmd

这个 jni 函数有两个参数，第一个参数是 int 型，第二个参数是 Object 数组

我们先上 frida 看看它是不是我们的目标。

```
Java.enumerateClassLoaders({
    "onMatch": function(loader) {
        if (loader.toString().indexOf("libsgmain.so") >= 0 ) {
            Java.classFactory.loader = loader; // 将当前class factory中的loader指定为我们需要的
            console.log("loader = ",loader.toString());

        }
    },
    "onComplete": function() {
        console.log("success");
    }
});

// 此处需要使用 Java.classFactory.use
var  signCls =  Java.classFactory.use('com.txxxao.wireless.security.adapter.JNICLibrary');
signCls.doCommandNative.implementation = function(a,b){
    var retval = this.doCommandNative(a,b);
    console.log(" #### >>> a = " + a);

    if( a == 70102){
        console.log(" #### >>> Obj = " + b);
    }

    console.log(" #### >>> rc= " + retval)  // .entrySet().toArray());

    // var stack = threadinstance.currentThread().getStackTrace();
    // console.log("#### >>> Rc Full call stack:" + Where(stack));

    return retval;
}
// */

```

这里先解释下这个 70102 的来历，doCommandNative 明显承担了很多功能，我们全部打印出来太乱了。

从之前的堆栈 com.axxbxxx.wireless.security.middletierplugin.c.d.a.a 这个类里面知道做 x-sign 签名的时候使用的命令参数是 70102 (对应的代码在 libsgmiddletier.so 这个假 so 里面)

跑一下

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBfjL32wWPbjAm2qAItFGgp1hhicZ9QltvxYn6ss08a64Mgqnh7g9rBIw/640?wx_fmt=png)1:frc1

确认过眼神，就是它了。

###### Tip:

Frida spawn 模式跑这个脚本的时候， loader 没有输出， 这时候把脚本任意修改个空格，然后再保存。Frida 会重新自动载入，这时候才能 有输出。

### ida 一下 libsgmainso-6.5.49.so

这个 so 还是很有料的。

首先他的导出函数里面找不到 doCommandNative 说明它是动态注册的。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBXv4alE4xjaKtD1d4QSEKtRYyuzTSawINkt7jESLMM44EHXn1fBN60Q/640?wx_fmt=png)1:ida1

其次是 so 里面稍微有点身份的函数都是这种动态跳转。有效的抵抗了 ida 的 F5。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBia6WgOcKOfe0NllesePk3wsLXZrjVPdXzXsye0pEzsiavKia66MyoEY6Q/640?wx_fmt=png)1:ida2

我们一个一个来解决。

动态注册我们不怕，Hook RegisterNatives 就可以搞定它

```
[RegisterNatives] java_class: com.txxxao.wireless.security.adapter.JNICLibrary name: doCommandNative sig: (I[Ljava/lang/Object;)Ljava/lang/Object; fnPtr: 0x7637c25ba4 module_name: libsgmainso-6.5.49.so module_base: 0x7637c07000 offset: 0x1eba4

```

结果出来，我们的目标是 0x1eba4

比较尴尬的是，ida 中的 0x1eba4 的位置怎么看都不像函数的样子。

怎么办？

从这个 so 的种种表现来看，会不会它在运行时有些自修改之类的玩法？

先不管那么多了，我们把这个 so 从运行时 dump 出来再说。

###### Tip:

dump so 的方法参考 http://91fans.com.cn/post/carcommunitytwo/

我的测试手机是 64 位的，所以 dump 出来了一个 64 位的 so

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBcGYHyKqvRCxXuyy1HYCicIpcic59nK69GMsQGyZJRa9ynqKV3AQ117Iw/640?wx_fmt=png)1:idasub

这次有那么点意思了，不过由于烦人的 BR X11 动态跳转，导致我们还是不能愉快的 f5

### 修复一下

如果我们知道这个 BR X11 指令的 x11 的值，然后把它改成一个静态跳转，是不是可以修复可怜的 F5？

说干就干

```
var mbase = Module.getBaseAddress('libsgmainso-6.5.49.so');
Interceptor.attach(mbase.add(0x1EC18),{
		onEnter:function(args){
		console.log('Context  : ' + JSON.stringify(this.context));
	}
});

```

打印出来

```
Context  : {"pc":"0x7637921c18","sp":"0x7639089340","x0":"0x20","x1":"0x76390893e4","x2":"0x2776","x3":"0x28","x4":"0x1","x5":"0x0","x6":"0x4","x7":"0x0","x8":"0x16","x9":"0x7639089350","x10":"0x7637a6cd60","x11":"0x7637921c2c","x12":"0x76390893e8","x13":"0x76390893d8","x14":"0x1","x15":"0x0","x16":"0x76dadbf000","x17":"0x76da67d440","x18":"0x0","x19":"0x76506125e0","x20":"0x0","x21":"0x2776","x22":"0x76390896bc","x23":"0x7650261ddf","x24":"0x8","x25":"0x196","x26":"0x763908d588","x27":"0x2","x28":"0x76390893e8","fp":"0x76390893b0","lr":"0x76dadbf60c"}

```

当前地址是 0x7637921c18 - 0x1EC18 = 0x763793000, 说明 so 基地址是 0x763793000 , x11 的值是 0x7637921c2c - 0x763793000 = 0x1EC2C, 说明这里要跳转到 0x1EC2C

那就把这行指令修改成 b 0x1EC2C

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBrTIAtDaRsuZxXRE4x59wxHucM4CZfYheiau2RMvTgR1jicGLaGwQuSuw/640?wx_fmt=png)1:idabx

再 F5 一下，就比原来漂亮一些了

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBCHqlnOHcRHBYO08YNSrqJd2IMshsXISKibBKaVOUIU2bbZV3ic0yTwUA/640?wx_fmt=png)1:idasubok

### Hook 这个 Native 层的 doCommandNative

这里主要就是介绍下 Hook Native 函数的时候，如何打印 Object[] 类型的参数

```
var mbase = Module.getBaseAddress('libsgmainso-6.5.49.so');
// 1ed4c
Interceptor.attach(mbase.add(0x1EBA4),{
    onEnter:function(args){
        console.log('doCommandNative = ' + args[2].toString(10));


        var Object_javaArray = Java.use('[Ljava.lang.Object;');
        var ArrayArgs_3 = Java.cast(args[3], Object_javaArray);
        var ArrayClz = Java.use("java.lang.reflect.Array");
        var len = ArrayClz.getLength(ArrayArgs_3);

        if( args[2].toString(10) == 70102) {
            for(let i=0;i!=len;i++){
                var objUse = ArrayClz.get(ArrayArgs_3,i);
                if(objUse != null){
                    console.log("args[3] String value:", objUse.toString());
                }
            }
        }

    }
});

```

先用 Java.cast 转一下类型，然后再 java.lang.reflect.Array 来遍历。

结果还是比较漂亮的

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBhV6XE62iaT3nl7libBV0MmjG4y0fNuu1gTO1LoNOOoqmlfibsl59m3FibQ/640?wx_fmt=png)1:rc

三、总结
----

Native 层的保护措施更多，大家都太卷了。

熟练掌握 java 的反射用法，是玩好 frida 的必要条件。

ida 的 F5 也是大家严防死守的，所以修复的方案也要了解下。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBfKRGQByJyZtYMxNLHib2icib4wgzPCJkffqLoKGw5IoLRNrD7WTk45CvQ/640?wx_fmt=png)1:ffshow

本想游戏人间，为何最后反被人间游戏。

###### Tip:

本文的目的只有一个就是学习更多的逆向技巧和思路，如果有人利用本文技术去进行非法商业获取利益带来的法律责任都是操作者自己承担，和本文以及作者没关系，本文涉及到的代码项目可以去 奋飞的朋友们 知识星球自取，欢迎加入知识星球一起学习探讨技术。有问题可以加我 wx: fenfei331 讨论下。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBkanMcfKVkjXYE68JYDfa51tUm5vJiaqesYVPL5MBjdJFQ18RLFxwuiaA/640?wx_fmt=png)

关注微信公众号，最新技术干货实时推送

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib1GTDIkAVibTS1Va9Frwz8tBAwv0OILXQaAHKkr2DMuRHwbxkRtwR3vz3ffM4hOp6Q6IFAjgiczIx9g/640?wx_fmt=png)

手机查看不方便，可以网页看

    http://91fans.com.cn

往期推荐

[

某 A 系电商 App x-sign 签名分析



](http://mp.weixin.qq.com/s?__biz=MzI4NzMwNDI3OQ==&mid=2247484555&idx=1&sn=acd26b99921dd6f1ea68ae6d8b243003&chksm=ebcefbb3dcb972a52bbf29446dc9432ae36dd94fe2d0c2e4ea6bd75bfaad3606b365816ed279&scene=21#wechat_redirect)

[

某汽车社区 App 签名和加解密分析 (二) : Frida Dump so



](http://mp.weixin.qq.com/s?__biz=MzI4NzMwNDI3OQ==&mid=2247484542&idx=1&sn=2322a3e5f86aa09d9998eea85e449a28&chksm=ebcefb46dcb972501805f25453b37d9ac6e31e16a9bf2495d0741cfb4c8d6de4fa0d0801c05f&scene=21#wechat_redirect)