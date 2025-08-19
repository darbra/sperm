> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LzXTndEQpqzXbNwy2z-xtg)

jadx
----

首先找到定位点。

这里直接搜索 zzzghostsigh

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2ib0u32IJMK7AJnE9bJEmhASo2uO2ibmIfexaLkqQLwlV00W4lzcxaKzw/640?wx_fmt=png&from=appmsg&watermark=1)

然后找他引用的位置

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2bPibeBRxucCYFdIaPIWZ7tjYgfJ64Rx9BY6jsQGzfsP8FpBdP1yTMFQ/640?wx_fmt=png&from=appmsg&watermark=1)

继续往下找一下位置。

这里发现是一个接口的实现。![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2PdD47CuhSOC6vBdic6vw3SQfJ9vTq5zaEwZaAaNtj3mCuicCAdQy6vuQ/640?wx_fmt=png&from=appmsg&watermark=1)

那没关系。我们继续找下实现的位置。

这里可以挨个去看。

这里发现是 gh.b()

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2dwibweDZxticVJibx32vXmJIlBG7zfACpIP7iczgpuMg7CY38TN947YJSQ/640?wx_fmt=png&from=appmsg&watermark=1)

继续点两下 就找到了 引用的位置。找到 so 层 是 libmfw.so

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2maHBC1EiatOQIIYje6dVExE8ffd8Wylafib44sd8xtrptXaib4dNibibWgg/640?wx_fmt=png&from=appmsg&watermark=1)

然后我们 hook 一下 看看传餐以及结果值。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2Uu9unAgx6ECsKMSDsG0S5noR4Nicueq6taUyA78j0gN5XK6J1oy0bTA/640?wx_fmt=png&from=appmsg&watermark=1)

unidbg
------

这里直接 unidbg 补环境

封装好请求

直接运行 报错如下

这里看到 直接就运行了没报错 缺少什么环境

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG24lgEiaGAkKYFfA23QfgyAqOkEDSXUomYpMLPoUx6umXs91NwBzSRevg/640?wx_fmt=png&from=appmsg&watermark=1)

```
[10:49:23 460]  WARN [com.github.unidbg.linux.ARM64SyscallHandler] (ARM64SyscallHandler:410) - handleInterrupt intno=2, NR=-128336, svcNumber=0x1a2, PC=unidbg@0xfffe0ab4, LR=RX@0x1203cbac[libmfw.so]0x3cbac, syscall=null
java.lang.ArrayIndexOutOfBoundsException: Index 0 out of bounds for length 0


```

直接去看报错的这部分基址。跳过这部分校验

ida 找一下 3cbac

然后分析下 。这个方法中都是校验。补起来相对还是比较麻烦的。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2uuL9IeGUODhZm1jfMubjsTwsO9P6Wib66jn93totqVIKNq7S6zwS2ibQ/640?wx_fmt=png&from=appmsg&watermark=1)

所以这里我们往上看。这个方法

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG25zFcwYmFSAAxlmhuk8yCkj8T4fjIGzWPZUPK0vliaKWF3xrErib8HQ1A/640?wx_fmt=png&from=appmsg&watermark=1)

按 x 看下调用的地方。这里就是调用环境的地方。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2z0IZuPzUmwOBxDtZ8SsMu5jsLWzUibw8RS9oYd5je0m7khaEGUbcZiaw/640?wx_fmt=png&from=appmsg&watermark=1)

所以这里我们直接跳过这个判断。

这里直接 unibdg 去跳过 这个环境

```
 emulator.attach().addBreakPoint(module.base + 0x3970C, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
//                emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_PC,0x3970C+4);
                emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_PC,address+4);
                emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_X0,1);
                return true;
            }
        });


```

这里返回 true 即断点不会断住，返回 false 即代表断点断住。

然后直接运行。发现可以出值。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2TXmWezhPEcHSrTSW6Wq1hCYYW36q8mVibLZu3SjEUK1kHDVlkic6OrLA/640?wx_fmt=png&from=appmsg&watermark=1)

结果也正如一开始我们 frida 的值一样。

接下来就是 trace 了

纯算分析
----

这里直接搜索 0x4a55

搜索到了 0x4a55d456 拆成了 8 个字节。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2iaUCwnNjabV2BvRN7SpISJNqZnN0SOupt0r6eibxbicz8YEwkDpsSAM0A/640?wx_fmt=png&from=appmsg&watermark=1)

那我们再搜下 0x9606ba47 果然也有 13 个。实际不算结果 12 个。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2WG35yw3NicMtkj7zwicqwCNSWMgpGaOicP3LrCeMTmyvLSTGOyUjxRNpg/640?wx_fmt=png&from=appmsg&watermark=1)

那总计下

4a55d456 9606ba47 87057a46 2b4f926c f9739142

拆成了 五组。 40 位。其实已经可以猜出来是 sha256 了。

先找下 0x4a55d456 的基址 0x1203f360

搜索 一下

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2v9efROLpuXXLO66sCKV3ombicCwWWeVQqT22u1vw3YblKBcyr3gYIPQ/640?wx_fmt=png&from=appmsg&watermark=1)

这里先回去看看 ida

这里把代码丢给 gpt。gpt 直接告诉我们这是个 sha1

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2pkm2ZUyYvEOibicIz5B6QDB0BD7qf39yC89eKKGwFtKdnptV3yClpfyQ/640?wx_fmt=png&from=appmsg&watermark=1)

但是我们实际上去运算。发现好像对不上啊

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2sBN2XQfnKS2LjcCzLPlUibs4a32L4IFVm3CwzlqlWzycYah4TOulutQ/640?wx_fmt=png&from=appmsg&watermark=1)

所以肯定是魔改了一些参数。这里继续回去看 010 里的 trace。

这里就相对简单了。我们对这源码去看实现就好。

这里找着入口对着看。

这里直接找 sha1 的特征值。

找一段源码（最好找 python 的。或者 Go 的 只要不是 JS 的, 因为我就是 js 改的。千奇百怪）

这里我们找下 sha1 的轮常数： 1518500249，1859775393，1894007588（-1894007588）

然后把它转成 16 进制去 trace 搜索。

这里分别为 0x5A827999，0x6ED9EBA1，0x8F1BBCDC

这里去搜搜

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2zqjA5uiaTwgviciaDeTY1opicLrYJChljITwSFx9iaG7IHDTamicASFwNouQ/640?wx_fmt=png&from=appmsg&watermark=1)

这里可以发现 trace 的地方有 703 个很明显不合理

找到 w29=0x5A827999 做下切割。

只拿一段。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2JOgNrFBJWRbD8hX11ISBVFpGRa5KrMfLWQ6nBdAyAOd5cHYPfl0FTw/640?wx_fmt=png&from=appmsg&watermark=1)

这里找到 movk 的地方。

发现一共 4 轮。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2lc8FhaNafbaiaicjy7I7HNCiaOricXicT5K7pP7J7qYnBB4CY8sLOgCI2JQ/640?wx_fmt=png&from=appmsg&watermark=1)

但是这这里 movk 直接就是就是加载常量了 所以得往前看一点。看看谁那些值。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2nI9BZeF0uTrsCjdc7tn2klvm3f4KibS7KiaAj6BzjxIvWvzrcXhHDbwg/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG25lYYPicqiarZlbRbP9858jz2schfZK6x6iatDVMHt31aCiaQoUoicibUH01w/640?wx_fmt=png&from=appmsg&watermark=1)

具体看看 trace 中的值。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2qPZx7z3meB5zHQj9NxzYJSEibbzG9BBeYo5rgaYgiaNmxUzF7jbrtgfA/640?wx_fmt=png&from=appmsg&watermark=1)

这里可以推断出 SHA1 的魔值

```
v18 = 0x67452301
v19 = 0xEFCDAB89
V20 = 0x98BADCFE
v21 = 0x5E4A1F7C
v22 = 0x10325476


```

但是好像和标准值有差别

标准的 “v21”与 “v22” 为 "0x10325476" 以及 "0xC3D2E1F0"

这里虽然改了。但是结果仍然不一致。

继续往下看吧。

分析下具体的代码

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2bl3kWFTOFJZ9JmibCibeqBhRyGWEUG0EYwO4OjVJraMIPcE5Kg2Nlz7w/640?wx_fmt=png&from=appmsg&watermark=1)

如下

以上依次为

movk、rev、and、add、eor、add、ror、add、ror、add、

_、字节反转、与、加、异或、右移、加、右移、加

这里可以看到这里基本和算法都差不多没有魔改。这一段对这去扣。

剩余是否一致也是一样的。往下看。

这里只讲差异点吧。

在开头搜过 0x5a827999 这个值。但是正常这个值。其实标准中应该只参与过 16 轮。而稳重有 36 轮。

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG27EddlQhbLGRgf4KSJ8HHwur2OM8VviaGDms8Pl0AmggxHSDOdMfqcDQ/640?wx_fmt=png&from=appmsg&watermark=1)

所以还有 20 轮肯定是改了轮数。

所以下文开始就是 16 轮之后的改的论数

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2qRbtf14dNTlsMeLGxIFp6cD0zHGtDibHa3KFlVld1gXZ76eC6IBNdbw/640?wx_fmt=png&from=appmsg&watermark=1)

对应的伪 c 如下

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2aKBXSzpD492Vc5HKvA3ZHmqUTHxxMburDHp377c6TkLHTv56JKTibpA/640?wx_fmt=png&from=appmsg&watermark=1)

继续问下 ai

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2kETXbn4Zr91iaRWTFbQ6Arjy91wCOr22Wurb2oNxsKhwiaXR0W6oLDSQ/640?wx_fmt=png&from=appmsg&watermark=1)

所以代码应该如下：

```
if (i < 16) {
    f = (b & c) | ((~b) & d);
    k = 0x5A827999;
} else if (i < 20) {
    f = b ^ c ^ d;
    k = 0x6ED9EBA1;
} else if (i < 40) {
    f = (b & c) | (b & d) | (c & d);
    k = 0x8F1BBCDC;
} else if(i < 60){
    f = (b & c) | ((~b) & d);
    k = 0x5A827999;
}else{
    f = b ^ c ^ d;
    k = 0xCA62C1D6;
}


```

这里既是改了 这些值还是对不上。

走到最后

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2Brt3iamvQxsBdVU9lAgYUNibWv5TuIfK02RRyoZ7F8KHHtnEAA9A1UQQ/640?wx_fmt=png&from=appmsg&watermark=1)

这里对源码看看

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG29QLtOoAYoXNQZ9W24v1dvZcz0pBI2C4ADHuhbAC6G7GEjWCbsMtFCQ/640?wx_fmt=png&from=appmsg&watermark=1)

那正常 应该是

4 3 2 1 0

而我们的 是

4 2 3 1 0

所以这样调转下位置。 然后运行

![](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEODsPzbaQoNLlW4orahdcG2ksyBlApP77vJEcibaqRpvj5YnyEWtic6CxTq7HOia3mJBqf89FVDD9cIw/640?wx_fmt=png&from=appmsg&watermark=1)

发现终于一模一样了。

结语
--

也算搞了好几天。还原了一个比较简单的算法。看到这里的同学记得点赞转发，感谢🙏