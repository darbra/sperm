> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6xO97nNO4Az981kaDUwfMA)

0x00 前言
-------

作为我离职后的第一篇文章，我决定将`解决无限debugger`作为本篇文章的主题，方便读者朋友们以后在遇到无限 debugger 时能快速解决。

在很久之前我发过一篇 [JS 逆向系列 14-Bypass Debugger](https://mp.weixin.qq.com/s?__biz=MzkzNTcwOTgxMQ==&mid=2247485008&idx=1&sn=22e3872914a4590e79e1142cad305d1a&scene=21#wechat_redirect)，但是这篇文章写的还是不太全，我决定在本篇文章中一次性将所有可能遇到的情况都写出来，具体请看下文。

注：本文不涉及修改 chrome 内核绕过 debugger。

0x01 正文
-------

实现前端无限 debugger 的三种核心方式为：

```
1. eval
2. Function
3. Function.prototype.constructor

```

当然除了上面这三种方法外，也有别的方法可以实现，例如下面这种定时器：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIalnvicshBZ3q0S7PeHZIsrWwbZsQUmBwCkoL8O1yWemnicfDEE0VloBA/640?wx_fmt=png&from=appmsg&watermark=1)

图上的代码使用了 setTimeout 定时每 0.05 秒调用一次 debuggerProtection 方法，方法内部有个 debugger，通过这种方式就能引起无限 debugger。

这种显式的将 debugger 写到前端代码里用来实现无限 debugger 的方法我称之为**显式无限 debugger**，这种是可以直接绕过的。我提供了三种解决方案供大家参考：

1. 条件断点 2. 替换 3.hook

前两个我都在之前的文章中讲过，现在我将那些内容搬到这篇文章中重新讲一遍。首先讲一下如何实现条件断点：右键执行 debugger 的那行代码的左侧，点击添加`条件断点`：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIoqdUOscpgCW48XdpugS6UyTKMQg2s8Ov7iaUbMI7qMjzoRfyVghq9NA/640?wx_fmt=png&from=appmsg&watermark=1)

条件设置为 false：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIXWS9tXFo1wzWOVJUqtAVyvM8bOnEo21ETK9j7ZsS1Gc58SdTyoxfpA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIu6nracMRoIwHoTcibsA95UTAAtILPN00RAfh1RkBUgaBP8f1ufyGYiaA/640?wx_fmt=png&from=appmsg&watermark=1)

现在这个 debugger 就不会生效了，但是这东西只能生效一次，意思就是你刷新网站之后这个条件断点就没了，所以这时候就可以考虑使用替换或者 hook 的方式进行绕过，下面我继续讲一下该如何进行替换。

F12 打开 devtools 后，点击源代码，左上方有一些板块：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIGXgx0ZK0HdzaKFjn4dWX6ZW92Sbc8fPSCZsGDiaiawgdgKpib9RriaibGtw/640?wx_fmt=png&from=appmsg&watermark=1)

展开点击替换：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIVtQTm7iassjN7fQqcmX7pcbAibLbcibwyYSDLsgdCLoZg04GcNR3rYmRA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSI7GkarkC44wJrJn1amDwcpDGzF0uz6SrpfcJkiaS5epC2FNvF4DNVIVw/640?wx_fmt=png&from=appmsg&watermark=1)

点击上面的`选择放置替换项的文件夹`，选择一个存放替换后的 js 文件的文件夹，并点击允许：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIkibv60naw8Ueh2KbTZtFPzgsOrPVNg3zMyoBian0Wib6ucGG3KGpsK8Dw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIUjxOlUnhf50M5Gso7j6kubLq3BUBrIOChTqic1m3kqXNl9HNhvOdj6g/640?wx_fmt=png&from=appmsg&watermark=1)

此时回到 js 中，找到执行 debugger 的位置：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIt8KtibDaKqn3FYlIPAicwmd1aT4GKiaiaYETZMUkrM8qvcstibwI9AibHffQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIBoS04M7COeq0ur65ibRxJgG39ff0qPVnqjswrx5At6jeEiag9a6V888w/640?wx_fmt=png&from=appmsg&watermark=1)

右键点击替换内容：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIE1myf3j6Lo1erAbTYl5P9duopzwlTkibMbxJ3nfzzqwE1662wWIMX6Q/640?wx_fmt=png&from=appmsg&watermark=1)

此时删去 debugger：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSI9qDwA0eWd7iaUwjdHg4uThXZSGlWeWswCgeibia0W6geu8iaJDjr39p2Ig/640?wx_fmt=png&from=appmsg&watermark=1)

然后再 ctrl+s 保存即可：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSI9755Wcn5PJSB7C76ZexZLtHI7poWMMJ36dKxz0ZgvBxDLya1HYhyrg/640?wx_fmt=png&from=appmsg&watermark=1)

文件名左边没有 * 号就代表已经保存成功了，此时刷新就会加载我们替换过的 JS：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIsef8UuYE1caqVgh3OVr8ggUPNMEIia2u0t0DLRsVWKfxF1mfa4S9Wew/640?wx_fmt=png&from=appmsg&watermark=1)

现在就不会再有 debugger 的干扰了。

这里我强调一下替换的两个注意事项：

1.`替换js后需要开着f12访问网站才会执行替换后的js，否则加载的还是原本服务端的js。`2. 如果发现修改后网站依然没有我们想要的效果，此时可以在替换后的 js 打个断点，检验一下网站是否加载的是我们替换后的 js。

针对第二点我可以举个例子，例如下面这种情况：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSI9HHfuUg8HIicm9m8I7tJZdnlfYL8IyKnt30hiariciaVeUB6Deb0kwwjAw/640?wx_fmt=png&from=appmsg&watermark=1)

可以看到目标网站的 js 文件名后都加了一个时间戳，这个时间戳是会影响到我们替换 js 的，例如我现在替换一下 adb 文件，删除该文件的 debugger 字段并重新进入网站：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSI9dd4I7fq6xBPDv0myUu3CxPoacepiaoJ4BuWia5XhBFOKswHg3whlKlw/640?wx_fmt=png&from=appmsg&watermark=1)

可见网站并没有加载我们替换的 js，依然弹出了 debugger，原因很简单，就是因为网站导入 js 时往文件名后面加了个时间戳：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIhE8KYyqNLw6Ep8cvALfcgRDk9MeSjmn6QibfTLkoEk1NiaiaIRR3IHapA/640?wx_fmt=png&from=appmsg)

这就导致替换的 js 与刷新网站后加载的 js 文件名不一致，导致替换不成功。解决这种问题很简单，那就是删除`new Date().getTime()`：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSInULzpjiaRfrAd1RllWKCwPGIy1j1OZr83EMT2DNBhl6fJXLs8TX2sfg/640?wx_fmt=png&from=appmsg&watermark=1)

现在加载的 js 就不会带时间戳：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIlrWuic9bVAEiblKibcibTONpMPgSMEYmGdCaWjwNFficBfuHy1xDib9BNu8A/640?wx_fmt=png&from=appmsg&watermark=1)

替换的时候就没问题了：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIUKhV2U7g22yLYtDy7W5wSZOia8SFXdJOic0H24lKL3Be4G4hrXA7cJaA/640?wx_fmt=png&from=appmsg&watermark=1)

成功加载了我们替换后的 js。

我在上面还提供了一个方案，那就是 hook，首先 hook 肯定能解决使用 setTimeout 引起无限 debugger 的问题，但是我这里就不提供相关 hook 脚本了，主要是因为我对 hook setTimeout 不太放心，有些开发者可能会利用此类方法去实现动画渲染或者其他用处，所以我担心 hook 此类方法后会造成不必要的异常，并且解决此类无限 debugger 也很简单，所以我在这里也就不提供相关代码了。

讲完使用定时器引起无限 debugger 的解决方案后，现在我就来讲讲如何绕过我文章中最开始所说的那三种引起无限 debugger 的核心方式 (目前市面上的主流方式)，首先可以直接使用我的插件 ----AntiDebug_Breaker：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIT3icJl1IPTTTTpmPfAyZ3za1cVhG3L3bibZ5MQRKwYKAr5IMqkmqNz2g/640?wx_fmt=png&from=appmsg&watermark=1)

github 地址：`https://github.com/0xsdeo/AntiDebug_Breaker`，该脚本可以绕过大部分存在无限 debugger 的网站，但是在某些网站可能会出现异常，例如下面这种情况：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIPTpGpia7OkaTZ8edIzYwzT8aQ1k4xicqnR1ibjV0qhgxIby58F9vXHn6w/640?wx_fmt=png&from=appmsg&watermark=1)

像这种出现一大堆异常的只有一种原因，那就是因为 **eval 的作用域问题**，如有兴趣的读者朋友们可以去看一下我这篇文章：[JS 逆向系列 15 - 深入探究 Hook eval 后存在的作用域问题](https://mp.weixin.qq.com/s?__biz=MzkzNTcwOTgxMQ==&mid=2247485069&idx=1&sn=a1cd135c0709f736beac6b388526854d&scene=21#wechat_redirect)，首先我先说明一下，这个问题**无解**，如果有人能解出，那么从代码层面上就能彻底解决无限 debugger 的问题了，但至少从我目前的视角来看，这种语法限制的问题绝对是解决不了的，至少我解决不了，所以如果遇到这种情况可以先开启下面这个设置：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSILyJWsAMqWMqVc4pznPMEE3yibzv3BxaRaEucJDDxL2oMcm3mTB1DByA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSILQ88kkCb4GbdaDqQhgjR06iaj5gNa5icZnlH32xrs17sXVWm17leT6mg/640?wx_fmt=png&from=appmsg&watermark=1)

开启这个设置后就能彻底忽略掉由 eval 引起的无限 debugger(注：**此时必须将插件开启的绕过无限 debugger 的脚本关闭**)，并且经实测，该选项也可忽略由 Function 实例的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIBwSsRicEjhuicibicEMPzuKh25s56GPGA0mOG4ZicXUibXQJekfKvqyATeXQ/640?wx_fmt=png&from=appmsg&watermark=1)

没有引起任何 debugger，所以开启这个选项即可绕过我上述所说的那三种核心方式引起的无限 debugger。

其实我这一年来一直都在致力于实现无感绕过无限 debugger，之所以这样做是因为我在上文并没有提到的一个原因，那就是有些在 vm 中的代码也很重要，如果直接忽略掉来自 eval 和 Function 的匿名脚本，可能会影响到我们正常逆向，所以如果我们遇到了 eval 的作用域问题，不一定非要开启上面那个设置，也可以考虑使用下面这段备用脚本：

![](https://mmbiz.qpic.cn/mmbiz_png/AkYofsd175M6xXicU2K7eaAkady2hUwSIQ6PXDGdKU7k2ajWHl728FEkIk0HzMNoWghU33u1h5st8FIYibcrPryA/640?wx_fmt=png&from=appmsg&watermark=1)

github 地址：`https://github.com/0xsdeo/Hook_JS/blob/main/hook_debugger/Bypass_Debugger/Bypass_Debugger(%E5%A4%87%E7%94%A8).js`，这段脚本只 hook 了 Function 和 Function.prototype.constructor，没有 hook eval，也就是说如果目标网站是由 eval 引起的无限 debugger 就没办法了，但是这样能过滤掉由 Function 和 constructor 引起的无限 debugger，所以需要读者朋友们自行判断。

0x02 总结
-------

<table><thead><tr><td><section>类型</section></td><td><section>特征</section></td><td><section>解决方案</section></td></tr></thead><tbody><tr><td><section>显式无限 debugger</section></td><td><section>将 debugger 写死到代码中</section></td><td><section>1. 条件断点<br>2. 替换<br>3. hook</section></td></tr><tr><td><section>虚拟机无限 debugger<br>（eval/Function/<br>Function.prototype.constructor）</section></td><td><section>debugger 在 vm 虚拟机中</section></td><td><section>1. 使用 AntiDebug_Breaker 插件或在油猴中注入 Bypass_Debugger 脚本<br>2. 如果发现报异常就开启 chrome 设置里的<strong>来自 eval 或控制台的匿名脚本</strong>或使用<strong> Bypass_Debugger(备用)</strong>。前者可以直接绕过，但是看不到 eval 和 Function 实例的脚本；后者不能绕过由 eval 引起的 debugger。</section></td></tr></tbody></table>

本文所涉及到的所有插件或脚本的 github 地址：

AntiDebug_Breaker：`https://github.com/0xsdeo/AntiDebug_Breaker` 

Bypass_Debugger：`https://github.com/0xsdeo/Bypass_Debugger/blob/main/Bypass_Debugger.js` 

Bypass_Debugger(备用)：`https://github.com/0xsdeo/Bypass_Debugger/blob/main/Bypass_Debugger(%E5%A4%87%E7%94%A8).js`