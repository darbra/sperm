> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HMrhWmB4k_c90WWPOyBDkA)

言
-

又好久没更新了。最近一直都在知识的海洋里苦苦挣扎，也算是学了一些东西，自以为技术还可以了，结果遇到几个难搞的 app，又把我干废了。

还是得多学多练啊  

这个 app 是好友 @Artio 给我的，我本着好奇想玩一下，结果还挺有意思。  

分析
--

打开 app，wifi 访问，习惯性的开 postern+charles，准备抓个包看看：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6MOC0wntZG3BIibliatibl7HcHS7VtI2KKkK3J4LueE2KjfHjHuS19icDJg/640?wx_fmt=png)

然后就有这个提示，我开流量访问，就很正常的访问数据，我就觉得很奇怪了，难道这个 app 禁止 wifi 访问？如果禁止 wifi 访问的话，是不是也算一种另类的反抓包办法啊哈哈

搞逆向的兄弟们应该都知道，像这种提示，是 toast 实现的（有关 toast 文档在这里：https://developer.android.google.cn/guide/topics/ui/notifiers/toasts?hl=en#java），那就跟下逻辑，找堆栈，看他是怎么识别我用的 wifi 的

objection 启动，内存漫游分析

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6E3VicnC3ga1j56CrpJrsey8pmfZwEO0X0j68ftoXh8mKEKtG6L7jdAg/640?wx_fmt=png)

卧槽，一启动就挂了，看来有 frida 检测，用脚本过掉 frida 检测，这里我用的脚本是针对两个的，一个字符串，一个 pthread 的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6hXFFN8QITkmyZ4mlpGJaaX4g7cF8eKWvdf2daz4YUIsz0zbW2dMCew/640?wx_fmt=png)

先只开针对 str 的：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6oCtQVo1kGqmLhtIaZ2YNdWmu1pdKuyZcOleZKCaebzlV8URgw16Xrg/640?wx_fmt=png)

ok，app 正常打开，frida 也没掉，那就算过了（后面我才知道并没有这么简单）  

那么要 objection 分析的话，就开两个终端  

先执行 objection -g xxx explore  

另一个终端马上执行  frida -U -l hook.js   xxx  

好了，现在就可以在 objection 里分析了，用 search  wifi，找到了 3000 多个

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6jY9owF0rI9Fibs2y6I5gk86GCAuWDMvNnbjJicxcTdCDxsFc8gPXhADg/640?wx_fmt=png)

我人麻了，哪个才是啊，后面一步步跟逻辑，发现了 toast 的类，

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6vqU7Fjmib9eq85mL8dIPxMW9NnHXCk2pKtuZicHiaynXNXNe8baoI2pZQ/640?wx_fmt=png)

然后在 toast 基础之上他自己封装了一个类：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia62nOzMUFn75tBukCWMZWED4EfuG3eRqxSFMDV3CmPcKaJibjoW9paJuA/640?wx_fmt=png)

然后用 show 调的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6ibsNBrV3rX4d0BJCUemEnoNH8Q0n09X3MZNNSLKzJWnianZVjMhO2bOg/640?wx_fmt=png)

那就一步步回溯堆栈看他是怎么判断的了。思路是没问题的对吧。我调试了老半天，愣是没找到怎么判断的，找到下面几个比较可疑的地方，结果 hook 发现根本没走到这里

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6ibibCgAoMuxUzk4AQOD6j026R2ky9CHODUS4SoFbibzEUfD08icCenTx3A/640?wx_fmt=png)

迷茫了  

好了，今天的技术分享到此结束，遇到问题睡大觉，要学会放弃，菜狗就得认命。。。。

![](https://mmbiz.qpic.cn/mmbiz_jpg/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6CMMXNtAM5uJYYEmVOcwiczNCgXjxyVr7v6l5iasqxgcJN7wxJljubBKQ/640?wx_fmt=jpeg)

好了，不扯了。后面冷静之后再分析，发现其实是 frida 检测到了，只是对 strhook 是不够的，还得有 pthread 的，所以我把上面 frida 对抗的脚本两个都开上了，用 spwan 模式再次启动

然后就有了，wifi 可以使用了：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6dVsZfMDNmXiawyiaxCL9VUVChYbJ50vic73LH3gMugG584PfF5aelFFRA/640?wx_fmt=png)

为了验证这个，我把 frida-server 停了再重启 app，wifi 可以正常打开，还真的是被检测了  

好的，现在再打开 postern 看看，发现还是提示网络错误那个，那就是有 vpn 检测了  

对于 vpn 检测的怎么搞呢，肉师傅星球里有相关的，同时里面也有个练习 app，Ipconfig，可以安装打开看看：  

这是啥都不开的状态 ：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6ToRtAgeO6EK0icX00zZ79M2OCKiby4npInhE27QK22muqUs0pelk2Mng/640?wx_fmt=png)

这是开了流量的状态：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6p4JzPvIXiaR6wNGgDBReaQHmS37B7mGZ5ZFECYnaur9sBLY76TlVK8Q/640?wx_fmt=png)

这是开了 wifi 的状态（开不开流量都是如下）：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6LA8L5j5vqXBiaz4AHbO6NDdxNHfWbCtG2wGQeY1Shcoyicu68beHmIVQ/640?wx_fmt=png)

这是开了 postern vpn 的状态：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6MFDRD4ibNt7maF7vQJlWP7ZgVSuAVOH91WbMla9Aia4kOo1pwcEF0ARQ/640?wx_fmt=png)

这区别是不是很大啊，所以，其实，大部分的 vpn 检测就是对 tun0 的检测，那我们再搞个 hook vpn 检测的 frida 脚本是不是就搞定了

好像说得通，那么，还愣着干嘛，搞起来啊，这下面的就是主要的逻辑  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6icv9F76G69fJ7JslJmfnmlibsic0ouTG4KK0iaF1OPJ2ibbbIuRl00Gia4Jg/640?wx_fmt=png)

现在跑一下这个过 vpn 的脚本，然后看看 Ipconfig 是不是被修改了就行了：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6EDqmQ36dpiafwhjUeHVYqEGicvFevSasbk8v1BH7WRmEMbjWulZVicAow/640?wx_fmt=png)

Ipconfig 显示：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6m7BmNdEaXxe26zJ40DpJQoWomIH3u3n9DhxP4iaNSRY1I9JQPcdSicbg/640?wx_fmt=png)

两个图片对比下：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6pOs8dsQJaEHExsNiaZ5x4Jzuk4yWGrQmkVzaSibCALwN3Dq6OEy81W3Q/640?wx_fmt=png)

好的，至少是过了，注意 ccmni2 的值 wlan0，ccmni1 的值还是 tun0，如果你想改的更彻底一点的话，可以再把这个值也改了，或者把值置空，这里我就不去操作了，自己研究了。  

好的，把这段过 vpn 的代码封装成函数，跟上面过 frida 检测的放在一个 js 文件里，spwan 模式启动：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6iaW2ibgfvlmQ1sRyxnfsGKxdv73B3gEV5Wialr2ykMlez2GTrA4K7zrmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6a9drwEFqdvSTCzexWyaj4yz23ZeX1suZESoqWnFo430k5f0tkZ4enQ/640?wx_fmt=png)

选择两个地方  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6Rewc3m0nPTmeXmJiapUicIvRDVibvZ6VSa3TVbp07FicxbEiba2gbia8R8LQ/640?wx_fmt=png)

然后 charles 看结果，ok，完美抓到包，返回的结果刚好就是刚才就是手机页面上显示的推荐路线

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6pHgD8rbG3nVbhFic8sMBpmDCEsVT2zgSL9HBHdE86lQzh6I0TMNicNYA/640?wx_fmt=png)

接下来就是处理加密参数了，卧槽，等会儿，还有 wtoken（嘻嘻，有点意思）

除了 wtoken 以外，其他就这几个参数需要传

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6mVcpe4wrhAZy8LaSlzCUOLQGETibica9ibFIyPnwCPt1sddnooN9IribAg/640?wx_fmt=png)

发现，time-stamp 是啥就不说了，request-id 是 time-stamp 加了几位数  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6eX4S5p7lvhJ0J9IVxV0yhK1Xw1m1KiaZavpOEQ8ZOgpbRgx4QGpYH0Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6uN3tJmWNTC0QTKo5kARCaLlicvB8iciaE480uXBcVYV4ETFiaN8jE3ia66Q/640?wx_fmt=png)

hmac 逻辑在这里：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6fJcraibDfMxbxYscuEM5RaGpWCoqZnbQY5IFLktgKsBMnusrllZibicxA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXW1yAnbkjfD1sOsqDmSzuia6YcXCDBph3iayoyzGbZs1czZrrhucmgSnFWKf04Ou7ribAHtZv4zqbpHg/640?wx_fmt=png)

ok，都是在 java 层里直接可以看到的参数 ，这里就不再继续了，over

结语
--

搞逆向还是得沉住气啊，慢慢来，别慌，不然思维容易卡住，一点点来，抽丝剥茧

当我 hook 掉之后，再次打开这个 app，就不再限制 wifi 访问了，好神奇，具体细节我就没去研究了。

至于他怎么判断 wifi 的，我也没去抠细节了，可以看看相关的文章：

https://blog.csdn.net/qiqigeermumu/article/details/110429819

https://blog.csdn.net/cn_yaojin/article/details/76854159

Ipconfig 想玩一下的，可以在这里下载：  

> 链接：https://pan.baidu.com/s/1K22pbglrt4Fawt_T_F8O1g
> 
> 提取码：ipjc