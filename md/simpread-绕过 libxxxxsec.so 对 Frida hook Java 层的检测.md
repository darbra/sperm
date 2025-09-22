> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5o45U-3I81iG1Mzev2RjYw)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DuibU3GqmxVmRsdItbBVRKegNHicHQvAHDdZsGpLVU7touSU1AU1twHTfRjG3Vu5aUh0RnPPllfVUhs4qdWF5QYQ/640?wx_fmt=png#imgIndex=0)

声明：文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用，任何人不得将其用于非法用途给予盈利等目的，否则后果自行承担！如有侵权烦请告知，我会立即删除并致歉。谢谢！

文章有疑问的，可以公众号发消息问我，或者留言。我每天都会看的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9zYJrD2VibHmqgf4y9Bqh9nDynW5fHvgbgkSGAfRboFPuCGjVoC3qMl6wlFucsx3Y3jt4gibQgZ6LxpoozE0Tdow/640?wx_fmt=png#imgIndex=1)

> 字数 575，阅读大约需 3 分钟

前言
--

回答一下昨天文章被删除的留言，react 也有类似于路由守卫的功能，绕过其实都差不多。

这主要还是涉及前端，刷漏洞的前提还是后端有相关的漏洞。可以起到辅助的作用，比如绕过路由守卫访问指定页面，而那个页面恰好有羡慕文件上传，xxe 啥的。

这种比单纯看 js 文件找接口，构造参数要方便很多。

今天看公众号文章的时候，看到了一篇讲 Frida 绕过的文章。  
[https://mp.weixin.qq.com/s/kZPYm_Ir-39cg7_Y7Ise3A](https://mp.weixin.qq.com/s?__biz=MzkyMzcxMDM0MQ==&mid=2247506587&idx=1&sn=b8a8498e4459afba262f1708aeb19ee5&scene=21#wechat_redirect)

提到的方法能过 native 的检测，但是并未绕过 Java 层的检测，当 Frida hook Java 层函数的时候，就会：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzG0SugB8gJ7flovxJYlicKWgwq5u2rg4FzXUia1YM9t7TnecvI6mt3Wr0g/640?&wx_fmt=png#imgIndex=2)4149726d51dcc24625b0edbf742c83ad.png

本文在这篇文章的基础上，补充一下如何绕过 libxxxxsec.so 对 Frida hook Java 层的检测。

过 libxxxxsec.so 检测
------------------

libxxxxsec.so 不单单会检测 Frida 特征，还会检测 prettyMethod 是否被 hook。

当 Frida 使用 Java.use(xxx) 或者使用 Java.perform(xxx) 的时候，其实对`prettyMethod`函数进行 inlinehook，导致该函数开头的几行汇编语句改变了。

libxxxxsec.so 就能通过这个变化，判断 prettyMethod 是否被 hook，如果被 hook 就直接退出。

比如

```
function checkJava() {
    Java.perform(function() {
        console.log("hello")
    });
}

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzG0SugB8gJ7flovxJYlicKWgwq5u2rg4FzXUia1YM9t7TnecvI6mt3Wr0g/640?&wx_fmt=png#imgIndex=3)4149726d51dcc24625b0edbf742c83ad.png

针对这种情况，有两种解决办法。

### 方法 1

第一种非常的简单，直接进入该 APP 的路径下，把相关 so 文件给删了，然后打开 APP，看是否能正常运行，如果可以，那就绕过了。如果不行，那就想其他办法。

我们通过 hook dlopen 函数，打印 so 的路径即可，路径特征如下

```
cd /data/app/~~一串字符/APP包名-一串字符/lib/arm64/libxxxxsec.so

rm -rf libxxxxsec.so

```

删除 libxxxxsec.so 后，APP 如果可以正常启动，说明没问题  
再次执行 frida 脚本  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzGVazM9jtLe9zYLw796CsP5sbFqlRtYTicmEElviaEjpOWdwssXsscA1Lg/640?&wx_fmt=png#imgIndex=4) d1e86f2429001bacb786b1e1d97d690d.png

  
不再被检测，完全没问题

### 方法 2

方法 2 来自 kanxue 的`wx_小友`师傅。  
文章链接：

*   • https://bbs.kanxue.com/thread-287913.htm
    

相关脚本在文章中有，这里就贴出来了。

`wx_小友`师傅的办法是在 so 加载的时候，hook so 文件的`.init_proc`和`JNI_OnLoad`，使其成为空函数，来实现绕过的。

而这两个函数的加载实际比较早，所以需要 hook `linker64` 中的 `__dl__ZN6soinfo17call_constructorsEv`方法。

在 android13 当中，该函数不是导出函数，所以需要通过地址来进行 hook。  
该方法的地址在不同 android 当中可能不同，因此建议将 linker64 下载到本地，IDA 反编译查看。

```
adb pull /system/bin/linker64 .

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzGrzicaMPWdk9nQJg6Ezcynmmyswh5KjpicRZbGvt8ic9RZtaMejvXRfJmw/640?&wx_fmt=png#imgIndex=5)73e392a3b6d0e99338eaa3ea24de47f2.png

替换  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzGQtqrH8NU7ZjfiaPCjgw1VpCkVJybWdicc3tnrKI3mNQBTr22R10uCAVQ/640?&wx_fmt=png#imgIndex=6) a62438dfa95f927b3d11ae3d3fab4a29.png

至于`.init_proc`和`JNI_OnLoad`两个方法，可以解压 APK，在 lib 下找到 libxxxxsec.so 文件，IDA 反编译查看相关地址，替换到脚本中对应位置。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhnKZt0QmX262AyUps4tlzGVazM9jtLe9zYLw796CsP5sbFqlRtYTicmEElviaEjpOWdwssXsscA1Lg/640?&wx_fmt=png#imgIndex=7)d1e86f2429001bacb786b1e1d97d690d.png

同样可以绕过检测。

参考资料
----

*   • frida 反调试 so 文件删除 https://www.jianshu.com/p/e3e955dd9cce
    
*   • https://bbs.kanxue.com/thread-287913.htm
    
*   • https://bbs.kanxue.com/thread-287058.htm
    

欢迎加入知识星球，领取优惠券，仅需 25~

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbria6KN80dMHzudmBQuUfsjkrQ5MkoIbSR9USqojTtHZcicKIZTwU2Tbuib8iafcZNQg0VbyM6ozWTTTSA/640?wx_fmt=png#imgIndex=8)