> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xUQFi-Ro_ojNQnhgMTgScQ)

⚠️ 免责声明

本文内容仅供学习与交流，不用于任何商业或非法用途。

若涉及侵权，请通过微信联系删除。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsu77OKhO4EGsiaC1sOCeBBE4OxtHxNHen2JhRUlrn0AFrmlDJ5Lnia1yeA/640?wx_fmt=jpeg&watermark=1#imgIndex=0)

一、 如何判断五秒盾的类型

五秒盾一共有三个类型：challenge、turnstile 和 Invisible 无感模式

像这种点进去网页开屏就是一个请判断您是否是真人的全屏网页，然后验证成功返回的是一条名字为 cf_clearance 的 cookie 的就是 challenge 类型，特征为 “继续之前，xxx.com 需要先检查您的连接的安全性”。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuicGsrYuup4gHHoX670szA2ut85FcL0ibUDaApdT0Y1JBV81iaAVjd5iaKA/640?wx_fmt=jpeg&watermark=1#imgIndex=1)

下面这种嵌入到其他网页中间的，出现 cloudflare 图标的就是 turnstile 类型，也就是旋转门，验证结束后一般是带着生成的 token 去请求登录或者注册等接口。  

也有一些特殊的，会同时生成两条不一样的 token，一条给注册登录等接口携带，另一条 token 请求 cloudflare 的域名成功后又设置了一个名字为 cf_clearance 的 cookie，这个需要注意下。

貌似还有一种无感的旋转门，唯一的区别只是没有图标，难度和流程跟正常的旋转门都是一样的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuPUO8gnYPG9pKJS8tricdfUMTO827AibESia2u3rL25mFjRZKvMqRrmmcQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)  

turnstile 和 challenge 类型都可能出现点击或者不需要点击的，是由网站的后台控制的，最后是无感，打开网页没有任何提示，但是去浏览器开发者工具查看会有名字为 cf_clearance 的 cookie，那就是无感了。

二、指纹分析

整体的发包流程就不赘述了，很多大佬的文章都有写过，到我写文章的时候也没变的。

这几个类型整体逆向难度从难到易为 challenge>turnstile>Invisible，但实际上 js 的逻辑都大差不差，challenge 和 turnstile 主要都是检测了 30 条左右的指纹，turnstile 少了几步 dom 检测的流程，Invisible 只有一个请求，就能拿到 cookie。

这是 Invisible 的发包指纹:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuhaHztHDcXAIjJicH946vTLUkz1rKR1p6RBMsibDHPTkCz6QfONbS6Nfg/640?wx_fmt=png&watermark=1#imgIndex=3)

而下面是 challenge 和 turnstile 拿到的指纹:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuY60RkDS6Lmy0gNPRouFEib08hbsMMxu60z7wZnfAfIFicRcexZibh18Pw/640?wx_fmt=png&watermark=1#imgIndex=4)

箭头所指的数组就是 Invisible 类型拿到的指纹。只是 challenge 和 turnstile 指纹中的一条，可以想象到 cloudflare 的环境检测有多么恐怖，但实际上有些指纹校验的也不是很严格，下面说一下检测点。

第一二条指纹:

也就是上面图片上的数组 1 和 2，这两条先后顺序是随机的，但一定是 1 和 2，也是检测比较严格的点，尽量要做到和浏览器一模一样。

主要是检测了 location screen navigation navigator document window 这些函数的值，tostring 函数这些要保护好，还有是否继承 Function 或者 Object，也就是原型链要补全。

代码运行时间差检测:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsupWkiaTdaWjTllz6ssaSemgFLAVp155bZGXuyPTPicrulSWcxkjH5ofXw/640?wx_fmt=png&watermark=1#imgIndex=5)

检测代码如下:

for (var a = 1, b = 1, c, d = 0; 5E3> d; d++) { let e = performance.now(), f = performance.now(); e < f && (c = f - e, c > a && c < b ? b = c : c < a && (b = a, a = c))}console.log({EUPiv1: a});

css 检测:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuwHgcZPZ4BxvyKVRTr8moSw2BnyXrbTxo28AzPWVLUoQBFISQyo9Ldg/640?wx_fmt=png&watermark=1#imgIndex=6)

这条检测了 cssRules 等，拿到了 css 的 text，并且还做了哈希，这条哈希可以固定，并且这一条也不是很严格，我代码里是做了动态获取的。

performance.getEntries() 检测：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuXSmx9PWH9wia3ovVscDSAfErrmNWnsABhtiahwSJzRF91FgcsrKdE2icA/640?wx_fmt=png&watermark=1#imgIndex=7)

时区:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsubhA06UNxKwqOmZy5PGIk8TicicDcu4xP8D1vJbbwyWyn0CQicPKN63jVw/640?wx_fmt=png&watermark=1#imgIndex=8)

报错检测：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuUfAkPVoavNgwT4oAb2MSG3jjU0ObYuIh3uibSRbg75W7dqXLPE51lPA/640?wx_fmt=png&watermark=1#imgIndex=9)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuOTicsbAxX87qa97mQ4fy3zdnEkeds1uCCR5TJvbYhNvLPe64TnBRbEg/640?wx_fmt=png&watermark=1#imgIndex=10)

还有一个比较严格的地方，cloudflare 会在响应头去设置 script-src 指令限制脚本来源和执行方式。若策略中 未包含'unsafe-eval'，浏览器会阻止所有通过 eval()、Function 构造器、setTimeout/setInterval 字符串参数等动态执行代码的行为，可以看一下下面的介绍 https://developer.mozilla.org/de/docs/Web/Security/Practical_implementation_guides/CSP

在 js 这里检测点主要是 iframe 页面的 contentWindow 下面的 eval 执行任何代码都是会报错的，而父页面的 eval 是正常的，这里需要注意一下，检测的也很严格，并且是随机禁止 eval 能否执行代码。

还有就是音频，图像，dom 的各种各样的检测了。实在是太多了，很多检测点都不是很严格，可以写死。还是那句话，缺啥补啥，还要耐心去一点一点补，总会补出来的。

三、成果展示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuibjOpicUicW44XfSCg3OD8Ft8OiaQfXrRrtgEcf2IWfgmJe6WlKBIW88DQ/640?wx_fmt=png&watermark=1#imgIndex=11)

challenge 4s，Invisible 1s，turnstile 3s

并发也是一点问题没有

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuIiaFXiaV7ftpM40Plxj7icYbTJNh1PucTleIESyicArGDiclibUmia2uPKcYA/640?wx_fmt=png&watermark=1#imgIndex=12)

技术交流，手机贴膜，电脑维修，上门通下水道请添加微信

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuxRdmkenW4D1BIQjXvsl4jK40YsicnjxlVfPccp1LzaIEnEU1y9KDDVA/640?wx_fmt=jpeg&from=appmsg&watermark=1#imgIndex=13)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/5Hx2mgUKPGkzczIPzTSVMgzHQKNcZfsuXJzqIuzibNVTfubmQQD5fMPcbXfic1khoib1rKicvuIianTRtahmqYuMdDA/640?wx_fmt=jpeg&watermark=1#imgIndex=14)