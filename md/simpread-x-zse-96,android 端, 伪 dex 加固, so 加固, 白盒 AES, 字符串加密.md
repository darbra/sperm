> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DUYHhh3aWDZCoyPGyn0OwA)

x-zse-96,android 端, 伪 dex 加固, so 加固, 白盒 AES, 字符串加密
==================================================

上一篇某招聘软件的 sig 及 sp 参数被和谐掉了, 所以懂得都懂啊!

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kMicBUyrbRzW7ZvFtbXsllARQLxc9Srkiaia5rYqTYtK4ayaLNXaYyhkDw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

因为 web 的 api 没有那么全, 所以来看了下 app 的, ios 的防护几乎没有, 纸糊的一样, android 端的有点复杂了, 到最后我也没能完整的实现整个加密过程, 我也只复现到 DFA 还原出了秘钥, iv 也找到了, 就是结果不对, 也许是魔改 AES 的程度比较高, 后续搞出来了的话再发下文吧. 网上找了下, 都是分析 web 的, 几乎没有分析 app 的, 所以这篇应该算是首篇比较详细的文章了.

声明
==

本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！

流程
==

抓包
--

抓包的话可以发现头部有个 x-zse-96 参数, 和 web 是一样的名字. web 的之前我也搞过, 没什么难度说实话.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kWQffA5ia4bO2wGdiaLMPZ51KYT0ic7CVDDic3OcUArjFdJSOotxypJpMPA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

关键代码定位
------

正常来说是先查壳看看有没有加固, 加固的脱壳, 没加固的直接拖到 jadx 里反编译, 因为我觉得这应该算是个大厂吧, 凭着经验大厂很少加壳, 所以我也没有查壳, 直接扔到 jadx 里反编译了. 尝试搜索一下字符串 x-zse-96

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kLJVgCoZeviaicQSnHRn5icXhZuuZmiaez8A9aFzffGGthFQVPUSHJQlS9w/640?wx_fmt=png&from=appmsg)在这里插入图片描述

什么也没有找到, 其实是字符串被加密了 这个时候就有很多方法可以选择了, 比如 hook java 层的系统 hashmap,x-zse-96 像是 base64 过的, 也可以 hook base64, 又像是 1.0_拼接后面一串得来的, 也可以 hook StringBuilder 的 tostring 方法, 甚至如果你觉得它是 so 层的加密也可以直接 hook NewStringUTF 函数. 这里我根据习惯先 hook 了 hashmap

```
Java.perform(function (){
    var hashMap = Java.use("java.util.LinkedHashMap");  //LinkedHashMap HashMap
hashMap.put.implementation = function (a, b) {
    if(a!=null && a.equals("X-Zse-96")){ //X-Zse-96 x-zse-96
        console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()))
        console.log("hashMap.put: ", a, b);
    }
    return this.put(a, b);
}}

```

什么也没有 hook 到, 看来不是通过 hashmap 来添加的 接着我 hook 了 StringBuilder 的 tostring 方法, 因为做过 web 的就知道, 这个参数就是前面的 1.0_拼接后面的得到的. 事实上 hook base64 也是可以 hook 到的

```
var sb = Java.use("java.lang.StringBuilder");
sb.toString.implementation = function () {
var retval = this.toString();
if (retval.indexOf("1.0_") != -1) {
    console.log("StringBuilder.toString: ", retval);
    console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()))
}
return retval;
}

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kv0icIqNYCxgGSzicu48saAJicRFFKMKQicL7kxzJPCibzTn3lTlD1tiace7g/640?wx_fmt=png&from=appmsg)在这里插入图片描述

从堆栈来看这个参数的生成是最后通过拦截器添加上去的, tostring 的下面一行点进去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kSwyw9hcOqDHW8M6jD7j0YG7oYpbb8bSuOOeibLuZGNichcaCWiaLXoAXw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

按堆栈的意思就是说这个 sb.tostring 就是结果了, hook 了下 b 函数看看传进去的是不是 x-zse-96, 结果并不是, 是一个 32 位的 md5 的结果, 堆栈后面有一句 native method, 这里我也是比较困惑为什么按照堆栈里找的和反编译出来的结果不一样. 这里我以为是反编译出了什么问题, 扔到 pkid 里查个壳看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kHQpR4qoMxHlF50uVOq0BFnLuiaiaW4MIrIP3H73ibHamrH22713DiarAzQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

好家伙, 不按常理出牌, 正常来说大厂都不加壳的, 看了一眼 so 的名字, dexhelper 确实是梆梆企业版的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kbzYwUBib41RkiaR5TVN3GrqfOoYsRUyqImKdPJI4FIxVhzicXbjN8S7kA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这个时候我拿出了我的 px4 定制的 fart 脱壳机, 结果直接运行不起来, 由于 fart 脱壳机太热门了, 很多厂商对这个都有检测, 我这个还是去过特征版的! 但是我记得这个梆梆加固主要是指令抽取, 正常来说反编译的代码都是空的函数, 为什么和 jadx 中的不一样, 并且里面的 dex 都是有数据的.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3k9WT9Iwiby5s1siarIiaZssqnuSKWgNibiaGj1ng8flgPkl8ibibCIhAGewicRw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

用 mt 管理器看一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kn6uVQncLQIJmC5NY0LDoHs0GY8gReesBv6WfTBQicPKaFwlKnUAWjuw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

发现是个伪加固, app 虚晃一枪, 要是正常人估计还在想办法怎么脱壳, 这样的话也不用脱壳了, 正好挺省事的.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kSwyw9hcOqDHW8M6jD7j0YG7oYpbb8bSuOOeibLuZGNichcaCWiaLXoAXw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

接着上面的逻辑, b2 不为空, 所以走了下面的逻辑, 并且有一个 addHeader 的方法, 这个像是往请求头里添加键值对. 并且可以看到是添加了两组, hook 一下

```
Java.perform(function(){
let Builder = Java.use("okhttp3.Request$Builder");
Builder["addHeader"].implementation = function (str, str2) {
    let result = this["addHeader"](str, str2);
    if(str=='X-Zse-96' || str=='X-Zse-93'){
        console.log(`Builder.addHeader is called: str=${str}, str2=${str2}`);
        console.log(`Builder.addHeader result=${result}`);
    }
    return result;
};
})

```

有结果, 并且入参是 x-zse-96 的值

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kuuibteT9jAicnYINp1csRuNr9ibHxXzgPE8TVltK68LpIicEle3tXoFQ5Q/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这里的 H.d("G51CEEF09BA7DF27F") 其实就是字符串加密了, 执行后结果就是 x-zse-96, 点进去是一个 native 方法. 在 so 层加密了, 现在很多 app 都弄成这样了, 通过搜索几乎定位不到关键代码. 接着看 H.d("G38CD8525") 就是 1.0_了, hook H.d 方法同样可以得到结果. 真正的加密结果来自 new String(this.c.a(a2)),a2 是查询参数 md5 后的结果转为了字节数组. 接着一路跟下来到了 native 方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kdWia4xGgDkLmJDx3UeDQz5ibOdqiaSmlqpPOQEl5rQaoBOiaB51H09atPw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

看名字应该是一个 aes. 中间的 str 是一个很长的字符串, 这个字符串每次都是一样的, 但加密结果不一样, 所以不用管这个, 主要是 1 和 3 参数, 分别是两个字节数组, 1 是 params md5 后的字节数组, 第 3 个数组初步可以当成是 aes 的秘钥, 如果是 ecb 模式的话 (初步认为, 其实是 iv, 一般人都会认为是秘钥的, 因为标题说了白盒 AES, 秘钥内嵌在程序里)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kVWvZDI2ZNdjXiakpqklqheK5XiaMH2b406AOBeCGgIscXjj0aL2NFngQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

so 模块没有出现在代码里, 前面我 hook 了 jni 方法 NewStringUTF 发现是 libbangcle_crypto_tool.so, 还可以 hook libart.so 来找动态注册以及 libdl.so 找静态注册. 接下来看 libbangcle_crypto_tool.so, 拖到 ida 里反汇编

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3knIsPVDwvxoJyXW5iaaNatecypGa3XoelH2k5icwo97eUFEJniaXps8F6g/640?wx_fmt=png&from=appmsg)在这里插入图片描述

可以看到都是 seg 段, 没有 text 段

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kjtibHWFpWrexEkgOwHjq2J2yubn6nvAaMwBzQSApzSy4ZDIg71ibzEkQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

并且导出函数中也是有 java 中的静态注册函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kgwe6MqCxNhj7jXLzYcGDRgRnsp7YHGvCSueauSR5ZXHQgia2Va5HMdw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

点过去也是这样, 这里你应该感觉到不对劲了, 似乎 ida 无法识别这个 so, 其实是 so 被加固了.

dump so
-------

so 加固的碰到的比较少, 这里我能想到的办法就是 dump 内存中的 so, 再修复一下.

```
function dump_so(so_name) {
    Java.perform(function () {
        var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
        var dir = currentApplication.getApplicationContext().getFilesDir().getPath();
        var libso = Process.getModuleByName(so_name);
        console.log("[name]:", libso.name);
        console.log("[base]:", libso.base);
        console.log("[size]:", ptr(libso.size));
        console.log("[path]:", libso.path);
        var file_path = dir + "/" + libso.name + "_" + libso.base + "_" + ptr(libso.size) + ".so";
        var file_handle = new File(file_path, "wb");
        if (file_handle && file_handle != null) {
            Memory.protect(ptr(libso.base), libso.size, 'rwx');
            var libso_buffer = ptr(libso.base).readByteArray(libso.size);
            file_handle.write(libso_buffer);
            file_handle.flush();
            file_handle.close();
            console.log("[dump]:", file_path);
        }
    });
}
dump_so("libbangcle_crypto_tool.so")

```

dump 下来后拖到电脑上修复一下就好了, 工具 https://github.com/F8LEFT/SoFixer 不过这样 dump 下来的效果不是很好, unidbg 压根跑不起来这个 so, 里面还是有些数据无法访问, 偏偏这些就是白盒 aes 中的关键数据 s 盒. 折腾了大半天, 一路打 patch, 最后还是放弃了 unidbg 这条路. 还是老老实实看 so 里的内容把, 修复后的 so 至少可以看到代码段了.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kQOaLMFXkIP3CGST1IgQtRrx1jfPqMbsmCcOaAB8b4IaVibc0DvcjLrw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

进来后先转 jnienv 对象, 这样代码会好分析些. 这个函数稍微有点控制流混淆以及虚假指令

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kNTyrQx05R7cFYs07iaj9UsHrm34nBkPAdtWeVq6LMo3d8cztHsE7WWw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

不过从后往前推就可以看出来走的那个函数了, v12 是返回值, v12 来自 v10 创建的字节数组, v10 来着上面的 8E74 函数, 这个函数就是核心的加密流程了, 点进去瞧瞧 v9 是 a1 赋值过来的, 所以也是 env 对象

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3k0ibeIafiahynTFQ6uf7VyicucOa3QOeMcicLpBLa3fTtk2Wcvu04hFjMmA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

进来后看左下角的流程图, 用了好几个 switch case 来判断走哪个流程, 还好我们知道我们的函数是 laes, 并且从提示来看知道是 cbc 模式, 也就是说需要一个初始化 iv, 这个 laes 的意思应该是 local aes, 字面意思就是本地的 aes. 从这个函数进去看看. 进去后再进入一个 laes 就到了下面这个图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3k15oY5iblDw4JOCvSia7w5vE3yLr8OOBfypLVlNdT5Hb1A1xrGHouwFtw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

前面那些内容函数最好 hook 看一下入参, 其实我都 hook 过了, 只是没写, 写了篇幅太长了, 后面还有白盒 AES, 这里有个 a3>15,a3 是明文的长度, aes 分组长度都是 16 个字节, 所以这里应该是判断明文的长度来决定加密几轮. 这个 a6 是个函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3krehFdCZ5MR0gLibEibOFDvsrP2HO47ZR6DxUzYcN6LAiactUsFa4xNsNA/640?wx_fmt=png&from=appmsg)在这里插入图片描述

直接点进去是这样, 不知道是不是加固的原因, 这里只好看汇编了, 鼠标放到 a6 那, 按 tab 转汇编视图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3ke6kEyRdg1ic0TWtw0TF6PrRJSZanZ6iaJUM8peJ3LEJQULaKdklo2iayg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

blx R12, 意思是跳到 R12 寄存器中的地址执行, 这里直接看看不出跳到哪, 可以用 frida 的 inlinehook 看看

```
function inlineHook() {
    var nativePointer = Module.findBaseAddress("libbangcle_crypto_tool.so");
    var hookAddr = nativePointer.add(0x6140); // 6140    
    Interceptor.attach(hookAddr, {
        onEnter: function (args) {
            console.log(nativePointer)
            console.log("onEnter: ", this.context.r12);
        }, onLeave: function (retval) {
        }
    });
}
inlineHook()

```

这个 so 是 32 位的, 正常情况下地址是要加 1 的, 但是因为这个 so 是从内存中 dump 下来的, 所以不需要加 1 了, 这个需要注意下. 结果是 0x6420. 这个 app 的好几年都是用的这个 so, 一点都没变, 偏移也没变, 属于是 so 加固了以为高枕无忧了.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3k126cR99lJ2O2aObqnM1DzbarI9TFUuB7GVpbl5kaMmHicEqGYmia010g/640?wx_fmt=png&from=appmsg)在这里插入图片描述

进来后看到一个名字 WB white box, 白盒的意思, 现在的安卓逆向 java 层的加密已经几乎没有了, so 的逆向上来就是各种白盒, 各种魔改算法以及自写算法, 不如直接去搞 ios 逆向, 现在的 ios 逆向很多都是调用的系统函数, 可以直接 hook 整个系统函数, 类似安卓的那个算法助手, 而且 ios 逆向的人少, 对抗少, 系统也闭源, 风控也比安卓小得多, 现在从 0 开始搞安卓逆向难度很大, 就算你有天赋至少也需要半年的时间 (ios 的可以看看沐阳老哥的课程, 当然只是题外话, 后面也会出几篇文章从 0 开始搞 ios 逆向.) 之前我发过一篇白盒 aes 的文章, 很多 js 逆向的反馈压根没听过这个词, 感觉很高级的样子, 正常, js 逆向你可以直接扣 js 啊, ida 中的这个伪 c 代码扣下来很难跑通, 对 c 的要求有点高. 我是扣了一下, 跑不起来, 放弃了. 看下面的内容前你最好了解下什么是白盒 AES, 这是我从网上找的一篇文章 https://blog.csdn.net/qq_37638441/article/details/128968233 接着正题, 前面 java 层传入了一个参数 3,16 个字节, 如果是白盒 aes 的话, 那这个大概率就是 iv 了.

```
function callAES(){
    var base_addr = Module.findBaseAddress("libbangcle_crypto_tool.so");
    var real_addr = base_addr.add(0x6420);
    var wbaes_encrypt_ecb_func = new NativeFunction(real_addr, "void", ["pointer", "pointer", "pointer", "pointer","pointer"]);
    inputPtr = Memory.alloc(0x10);
    var inputArray = hexToBytes("b11812121a8886852e2e868786252e2e");
    Memory.writeByteArray(inputPtr, inputArray)
    var inputPtr3 = Memory.alloc(0x10);
    // var inputArray = hexToBytes("803d17b5b00000000400000080000000");
    var inputArray = hexToBytes("401948d4b00000000400000080000000");
    Memory.writeByteArray(inputPtr3, inputArray)
    var inputPtr4 = Memory.alloc(0x10);
    var inputArray = hexToBytes("00000000000000000000000000000000");
    Memory.writeByteArray(inputPtr4, inputArray)
    var inputPtr5 = Memory.alloc(0x10);
    var inputArray = hexToBytes("00000000000000000000000000000000");
    Memory.writeByteArray(inputPtr5, inputArray)
    wbaes_encrypt_ecb_func(inputPtr, inputPtr, inputPtr3, inputPtr4,inputPtr5);
    console.log(hexdump(inputPtr,{length: 0x10}));
}

```

so 主动调用这个函数, 其实它有 5 个参数, ida 识别成了 3 个. 前两个是同一个参数, iv 和明文异或的结果, cbc 模式下比 ecb 模式多了个 iv, 后两个参数作用不大, 不用管, 关键是第 3 个参数, 有点奇怪, 现在我也没搞清楚它的作用是什么, 后面再说吧.

DFA 攻击
------

调用后有结果, 结果也是正确的. 接下来寻找 dfa 攻击点, 第 8 次列混淆到第 9 次列混淆之间, 并且需要寻找到 state 块

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kDMydhyhJXLMAcaIM6KbJQfbTMWZbfgmGwl3yv0icYwRILFBm0nkfKsA/640?wx_fmt=png&from=appmsg)在这里插入图片描述![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kVbVtwGgicjRAHQ3EA1KQxbNcNSUjS3QXNsdZsibP6c9aiaW8QenbicicC6g/640?wx_fmt=png&from=appmsg)在这里插入图片描述![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3k2GOMuzR0ETlIb4LZibVKsbYuhGk8APicwlnmY8sdUkvIauv675ysjPoQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kDQZy1vq47ZTOhswbyIrhrAFMbjAic1avY2ia6uA020oVeGRrYPpjGomw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

按 aes 的算法应该是这样, 最后面的第十轮少了列混淆, 所以不在这个循环里面. 加下来寻找哪个是 state 块, 初看选了 result, 每次核心运算都有他, 一 hook 发现经过 9 轮他的值都不变, 直到最后赋值结果的时候才变了. 后来仔细看了下 v20 也有可能, 但是这个 v20 不太好 hook 他的变化, 因为这里没有什么函数调用, 所有没有入参什么的, 要 hook 到这个 v20 的地址有些困难

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3koww9zgib11Y5Vbg3I4leWYP69CVkKmcsGg31JibYdbte7E5tFBvwHvcw/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这里我寻找九轮循环最开始的地方 hook 上, 这个赋值写的也有点复杂. 鼠标放到 v20 的地方, 转汇编更方便看.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kybubkdAmT4ujdmWAzaKISdmgma2DEoCj8Btgh797YaHA5GqjtjkQnA/640?wx_fmt=png&from=appmsg)在这里插入图片描述![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kVXgWnNia5jgGCVyW0pLic4iaJicc1Cj4EiaP9FqMxF2xbGYHxL8G2QOXlYg/640?wx_fmt=png&from=appmsg)在这里插入图片描述

这里鼠标放到 var_34 按 h 可以转成立即数. STRB R3, [R11,#-52] 这条指令的意思就是将寄存器 R3 中的值存储到距离 R11 寄存器所指向的内存位置偏移为 -52 的地方。所以 v20 的地址就是 R11 的地址减去 52 的偏移的位置处.

```
function hookwb(){
    var count = 0;
    var base_addr = Module.findBaseAddress("libbangcle_crypto_tool.so");
    var real_addr = base_addr.add(0x6580)
    Interceptor.attach(real_addr, {
        onEnter: function (args) {
            count += 1;
            console.log("onEnter: ",count, hexdump(this.context.r11.sub(0x24),{length:0x10}));
            console.log("start:"+count);
        }
    });
}

```

结果也确实是 9 轮, 接下来进行 dfa 攻击.

```
function hexToBytes(hex) {
    for (var bytes = [], c = 0; c < hex.length; c += 2)
        bytes.push(parseInt(hex.substr(c, 2), 16));
    return bytes;
}
var inputPtr;
function callAES(){
    var base_addr = Module.findBaseAddress("libbangcle_crypto_tool.so");
    var real_addr = base_addr.add(0x6420);
    var wbaes_encrypt_ecb_func = new NativeFunction(real_addr, "void", ["pointer", "pointer", "pointer", "pointer","pointer"]);
    inputPtr = Memory.alloc(0x10);
    var inputArray = hexToBytes("b11812121a8886852e2e868786252e2e");
    Memory.writeByteArray(inputPtr, inputArray)
    var inputPtr3 = Memory.alloc(0x10);
    // var inputArray = hexToBytes("803d17b5b00000000400000080000000");
    var inputArray = hexToBytes("401948d4b00000000400000080000000");
    Memory.writeByteArray(inputPtr3, inputArray)
    var inputPtr4 = Memory.alloc(0x10);
    var inputArray = hexToBytes("00000000000000000000000000000000");
    Memory.writeByteArray(inputPtr4, inputArray)
    var inputPtr5 = Memory.alloc(0x10);
    var inputArray = hexToBytes("00000000000000000000000000000000");
    Memory.writeByteArray(inputPtr5, inputArray)
    wbaes_encrypt_ecb_func(inputPtr, inputPtr, inputPtr3, inputPtr4,inputPtr5);
    // var output = Memory.readByteArray(inputPtr, 0x10);
    // console.log(bufferToHex(output))
    console.log(hexdump(inputPtr,{length: 0x10}));
}
function bufferToHex (buffer) {
    return [...new Uint8Array (buffer)]
        .map (b => b.toString (16).padStart (2, "0"))
        .join ("");
}
function hookwb(){
    var count = 0;
    var base_addr = Module.findBaseAddress("libbangcle_crypto_tool.so");
    var real_addr = base_addr.add(0x6580)
    Interceptor.attach(real_addr, {
        onEnter: function (args) {
            count += 1;
            console.log("onEnter: ",count, hexdump(this.context.r11.sub(0x24),{length:0x10}));
            if(count===9){
                this.context.r11.sub(0x24).add(randomNum(0,15)).writeS8(randomNum(0, 0xff));
                console.log("onEnter: ", hexdump(this.context.r11.sub(0x24),{length:0x10}));
            }
            console.log("start:"+count);
        }
    });
}
function randomNum(minNum,maxNum){
    if (arguments.length === 1) {
        return parseInt(Math.random() * minNum + 1, 10);
    } else if (arguments.length === 2) {
        return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10);
    } else {
        return 0;
    }
}
function dfa(){
    for(var i=0;i<300;i++){
        hookwb()
        callAES()
    }
}

```

连续调用 300 次后取得故障密文再用 phoenixAES 拿到第 10 轮秘钥 E48E8AA0E4449B580CF7ECC13CC17CD3, 接着用 aes_keyschedule 还原出主密钥 6BA6737912D31F3A1B53066645FABEA3

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kYmSJpwRNvibjUdzzRpVTC48O475UO3HlCrCtHiaicRPCictChPKicicxLdRQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

秘钥 6BA6737912D31F3A1B53066645FABEA3,iv99303a3a32343a3992923a3b3a999292(java 层的) 都有了. 事实上只有 1 轮的话, 也可以直接用 ecb 模式, 因为入参 1 就是明文和 iv 异或的结果, 拿去加密一下发现结果不对...... 前面我就说了这个参数 3 不太清楚是什么, 因为这个入参这个时刻下主动调用是这个结果, 重开 app 主动调用逻辑不变结果取变了. 这里有个很大的问号? 这里先分析到这里吧, 先埋个坑, 后面有空再看看吧.

最后
==

微信公众号

在这里插入图片描述

知识星球

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kiavtjvE21FC3ALRxoNqaPT4IQlfPeSiaa1fIv8RXf6I7fAicXE40QH8hA/640?wx_fmt=jpeg&from=appmsg)在这里插入图片描述

如果你觉得这篇文章对你有帮助, 不妨请作者喝杯咖啡吧!

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVmCcL9JibTsHYyJwSap4w3kqWSnFtB0JKH9FmsN5865B0RE37Ct2WhFnubDvmIT913b4KgroUdujQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述