> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484649&idx=1&sn=42eb5319db1ca830ca81d75218e4c0e4&chksm=fb18f54bcc6f7c5de60395d03650aa7c6a30e37407989c604c31ffa1076d071a32afcb0556c4&mpshare=1&scene=1&srcid=0310Lj90mCDiRI2ZAIpqT7T4&sharer_sharetime=1646874335522&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

前言
--

是的，我重新发了，没想到一不小心就过了这么久了，发现这期间有天赋的大佬们现在对于 tls 指纹研究得比我还透彻了，真的强啊。有种感觉 tls 未来会出现一个单独的派系。

文章较之前有部分改动，对理解 ja3 来说不影响的。因为某大佬看了我的文章后说我的方法其实不是完美突破，所以完美两个字没了。可能有朋友会说 " 你这不是在炒冷饭吗？没意思”，不不，完全没这想法。我想的是**：**

**1. 为了留存备份，等我老了可以给子孙后代看，这个是我写的。**

**2. 以后大家不用再拿着那个 pdf 私下转发了。**

**3. 之前明明答应过给朋友转载的，所以现在重发兑现下承诺。**

**4. 还是有些朋友一搞不定某站就说是 ja3 指纹，所以重发可以拿来参考下什么是 tls，现在用 tls 指纹的网站可真的不多啊。  
**

**5. 其实之前有几个朋友希望我能重新发且**开启付费**，那就发吧，付费就没必要了，毕竟也不是靠这个过日子的。**

之前发过 ja3 相关的文章可以作为初期了解：[JS 逆向之猿人学第十九题突破 ja3 指纹验证](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484137&idx=1&sn=ccfa46a45a09e7fde284dfba281fd719&chksm=fb18f34bcc6f7a5d49ee3050887aa909708ede268cb5046bcd80d43ffdc7c9f948d428c65ec4&scene=21#wechat_redirect) ，再来看本篇文章，篇幅略长，如果赶时间的朋友，可以跳过接下来的理论概念介绍，可以直接从后面的【如何突破 ja3】开始看

Official Account

到底什么是 ja3
---------

### 简介

git:https://github.com/salesforce/ja3

官方解释：

> JA3 是一种创建 SSL/TLS 客户端指纹的方法，它应该易于在任何平台上生成，并且可以轻松共享以用于威胁情报

 更权威的介绍文章：https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967

浓缩一下大概意思就是：

> JA3 是一种对传输层安全应用程序进行指纹识别的方法。它于 2017 年 6 月首次发布在 GitHub 上，是 Salesforce 研究人员 John Althouse、Jeff Atkinson 和 Josh Atkins 的作品。创建的 JA3 TLS/SSL 指纹可以在应用程序之间重叠，但仍然是一个很好的妥协指标 (IoC)。指纹识别是通过创建客户端问候消息的 5 个十进制字段的哈希来实现的，该消息在 TLS/SSL 会话的初始阶段发送。

这.....，2017 年就有了，我是 2021 年才通过阅读青南大佬的文章知道有这么个东西。

然后我猜，ja3 名字的由来，是因为有 3 个研究人员，且他们的姓名缩写都是 ja

### ja3 的由来

其实，John Althouse 自己说过，官方解释：

> 对这个 TLS 客户端进行指纹识别的主要概念来自 Lee Brotherston 2015 年的研究（https://blog.squarelemon.com/tls-fingerprinting/）和他的 DerbyCon 演讲（https://www.youtube.com/watch?v=XX0FRAy2Mec）。如果不是 Lee ，就不会有 JA3 的出现

### ja3 如何工作的

原文解释的很好了，官方解释：

> TLS 及其前身 SSL 用于为常见应用程序和恶意软件加密通信，以确保数据安全，因此可以隐藏在噪音中。要启动 TLS 会话，客户端将在 TCP 3 次握手之后发送 TLS 客户端 Hello 数据包。此数据包及其生成方式取决于构建客户端应用程序时使用的包和方法。服务器如果接受 TLS 连接，将使用基于服务器端库和配置以及 Client Hello 中的详细信息制定的 TLS Server Hello 数据包进行响应。由于 TLS 协商以明文形式传输，因此可以使用 TLS Client Hello 数据包中的详细信息来指纹和识别客户端应用程序

更多更专业的，如果很感兴趣，劳烦移步 John Althouse 发表的文章：

https://engineering.salesforce.com/open-sourcing-ja3-92c9e53c3c41

感觉看了上面的话，是不是有点似懂非懂的感觉，没事，听我慢慢说，来个图吧，要来图的话，先来个三次握手的吧，老图了，相信各位朋友也都看烦了，我也看烦了，因为这图都传包浆了

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqwECf0Z0zwKSbiatYPDkPRELcNIneuS2iasumIG676Picicpia1yBcPIqaZw/640?wx_fmt=png)

详细的三次握手流程解释我就不说了，网上能查到的太多了，我就不献丑解释了，本篇文章的重点也不是它

那么按照 ja 开发者自述，是在三次握手之后，客户端向服务端发起 client hello 包，这个包里带了客户端这边的一些特征发给服务端，服务端拿来解析数据包，然后回发一个 hello 给客户端，之后再进行 ssl 数据交互，下面这个图，就是 John Althouse 自己画的

 ![](https://mmbiz.qpic.cn/mmbiz_gif/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqc6R25IHBd7ORubcC0zHwGNqBKN3ibuhHI0pBGPaMNLr7pWibToTNzKIw/640?wx_fmt=gif)

怎么说？画的很传神很易懂对吧？

更多原文解释，请看这里：https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967

John Althouse 有关 ja3 的 ppt 讲义，上面的动图就是 ppt 文件里的，费了老大劲搞到的：链接: https://pan.baidu.com/s/1pfQwRwg3tbOT2Y2nQZYoAA  提取码: ptj6 （链接失效了可以联系我）

### 识别原理

以下 John Althouse 的文章里的：

> 1.JA3 不是简单地查看使用的证书，而是解析在 SSL 握手期间发送的 TLS 客户端 hello 数据包中设置的多个字段。然后可以使用生成的指纹来识别、记录、警报和 / 或阻止特定流量。  
> 2.JA3 在 SSL 握手中查看客户端 hello 数据包以收集 SSL 版本和支持的密码列表。如果客户端支持，它还将使用所有支持的 SSL 扩展、所有支持的椭圆曲线，最后是椭圆曲线点格式。这些字段以逗号分隔，多个值用短划线分隔（例如，每个支持的密码将在它们之间用短划线列出）
> 
> 3. JA3 方法用于收集 Client Hello 数据包中以下字段的字节的十进制值：版本、接受的密码、扩展列表、椭圆曲线和椭圆曲线格式。然后按顺序将这些值连接在一起，使用 “,” 分隔每个字段，使用 “-” 分隔每个字段中的每个值

其中第一条，也解释了我前一篇请求时尝试提交一个 ssl 证书为啥没有用的，第二条，服务端会在 3 次握手之后，收到客户端过来的 hello 包，然后解包，收集版本、接受的密码、扩展列表、椭圆曲线和椭圆曲线格式，在这时候就可以拿着 ja3 指纹去比对，哪些是限制了，哪些没有限制的，当确实有在限制名单里，就针对处理，当没有在限制名单里也返回一个 hello，接着再继续 ssl

#### ja3 已收录指纹 / 黑名单

*   https://sslbl.abuse.ch/blacklist/sslblacklist.csv  这个没有更新了只有上百条
    
*   https://github.com/salesforce/ja3/blob/master/lists/osx-nix-ja3.csv  这个不是很全，只有上百条
    
*   https://ja3er.com/getAllUasJson  这个一直在更新，十几万条
    
*   https://ja3er.com/getAllHashesJson  同上，只是给定标注字段不同
    

我猜测，ja3er.com 里的十几万 ja3 指纹，就是所有人访问过该网站的客户端（浏览器或者语言请求库）的指纹，有一条算一条的收集

### 案例解释

前面说了一堆理论概念，我们做开发的，如果只有概念没有实操或者例子解释是不够的，所以，来个案例，还是猿人学 19 题，同时继续祭出 wireshark 方便分析，先浏览器访问网站，拿到 ip，**提示一下，你们可别拿着 ip 去干人网站哈，挺好的一个爬虫练习平台，否则后果自负**

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqG9Dqax9hBj8iaQQxDT44g0WC1nRJ9ImscRtXDEic8lcIv5z10CeqGSag/640?wx_fmt=png)

这边 wireshark 显示：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqx7IGMblK3Oeicb0TbU8lEGDdZE6ZP6IyFotav9QyMKrt18WHyx0FWVg/640?wx_fmt=png)

此时你会看到很多的数据数据包，过滤一下，直接在过滤器位置输入 ip.，就会有很多指令提示和历史记录

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqrYm2vZrH8carSDRRSfxnmv456DHNqQpVOlicAZa9Ie9ZLFIQrBLcUiag/640?wx_fmt=png)

全部的命令是啥意思这里就不多说了，更多的可以百度，只介绍几个这里会用到的命令：

> ip.dst_host 就是目标地址，这里你可以理解为就是访问网站的服务端地址
> 
> ip.src_host 就是源地址，这里你可以理解为就是你正在操作的电脑的 ip，这个 ip 大多是局域网 ip
> 
> ip.host 跟上面一样，但是不会区分是 src 还是 dst，只要有指定地址的都会过滤出来

给定过滤指令，回车，此时有可能你输入过滤命令之后，看不到有任何数据包，小问题，可能你打开 wireshark 完了，没抓到包，重新刷新下网页就行了，此时就看到如下的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqPHBMgPmyn7iaW5CG59CYVq8u26ib3vNTRibmAoqSDzn3UBMcPGf2ibeAWw/640?wx_fmt=png)

注意看，最开始的 6 个数据包就是上面说的流程，前三个是三次握手，第四个开始到第 6 个就是 ja3 的 hello 数据包，个人感觉这段过程也很像三次握手，再后续的数据包就是实质的 ssl 数据交互了，

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqVS6uDnknFSc8WX2KxjGovK9tAUMqibK0vl9GwKNoe0Xk9H1NNVwhzyQ/640?wx_fmt=png)

有了这个，看看 ja3 指纹到底是啥样的，双击这个 client hello，然后服务端返回的 sever hello 就不看了，此时不需要看，忘掉它吧

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqsU3W4YC5cgteqjRdWCsDEI13zMsbmuyCibY3PUd8rEPsj7WkZ6ibTTRw/640?wx_fmt=png)

 展开 client hello：

 ![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqlkn6eUymDqic6KHVgvDwoQSQpgiamPkBaJIsYQ6uicHcRichPicyNErGIBw/640?wx_fmt=png)

滑到最后，就能看到 ja3 指纹了

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqzcWwzuusORbJRicd7TlJU8agY5521Hb6lyR8UdlFE4o5fWX7n2B3oBQ/640?wx_fmt=png)

然后，看 JA3 Fullstring ：

> 771,43690-4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,64250-0-23-65281-10-11-35-16-5-13-18-51-45-43-27-17513-51914-21,51914-29-23-24,0

对应：

> TLSVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats

以【,】分割，第一个数，771，根据官方解释，就是 ssl/tls 版本，也就是这个：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqOsdGx3FXBjfq10pGicXWrG4KYzgJTNlNoI0Mm9CkLBVP5HJAX8XxWfQ/640?wx_fmt=png)

python 终端跑一下，0x0303 就是 771，对上了：

 ![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqg5tmajkAiayeYq8mWdhBYOc6iaTFzhV0pJQLwRlV3e5vdcs45GceiagfA/640?wx_fmt=png)

后续的就不细说了，直接看

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqlslZwdjZzPf48XUcW8iaicHovINlsAdjgtoLnNbdyibSTwEjVgS3B40iaw/640?wx_fmt=png) 

 ja3 图 1

细心的你发现了，我截图这里是没有第四个数相关的套件， 但是 ja 自己的文章里是有第四个数 EllipticCurves 相关的：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqeB6dn9wcgPaFTjpvpKT4HrQbiaW9JybOXq2l48gclzcmsONkIQStctg/640?wx_fmt=png)

其实不是没有的，是现在第四个参数由 EllipticCurves 改成了 supported_groups

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqWa5sgggeMwViaXVUu47Olhiaiac2E5HYoSpHictHDhcqhSibambBrnztUdw/640?wx_fmt=png)

ja3 图 2

注：ja3 图 1 是公司电脑，ja3 图 2 是家里电脑，所以看着 ja3 字段有点不一样，tls 版本也不一样

说到版本，纠正下我前一篇给的说明，**ja3 指纹不是一定在 tls1.3 上支持的，上面的截图各位朋友应该也看得到，tls1.2 也支持的，不过也确实是需要 wireshark 的最新版才能看到 ja3 指纹**

有关第三个数 Extensions 扩展列表，感兴趣的可以看看更详细的解释：https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml

实际的案例作为 ja3 概念解释就到这里了。

如何突破 ja3
--------

终于到了大家都很感兴趣的环节，怎么破它了。

完全突破 tls，我知道的有两种**，一种是 hello 包 hook 改写，一种是自编译 openssl 重新改默认的指纹**

### python 半突破

在上一篇的解决方案里，修改 cipher 里的加密算法即可，也就是 TLSVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats 里的【Ciphers】，

有关 ciphers 相关套接字的，可以看看 openssl 的官方解释：https://www.openssl.org/docs/man1.1.1/man1/ciphers.html，，并且华为官网的 waf 介绍里也有不同 tls 版本对应的套接字：https://support.huaweicloud.com/bestpractice-waf/waf_06_0012.html

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqDGD9mUBN2Af3C6jgdByJBANTX2PrFA6gGwpPOicCnReI0F5MJKLrTTg/640?wx_fmt=png)

也就是说，我们要想改，可以直接复制上面的套件覆盖下即可，例如：

> ```
> urllib3.util.ssl_.DEFAULT_CIPHERS = 'EECDH+AESGCMEDH+AESGCM'
> 
> ```

但是，相信对这个 tls 有过相关经验的大佬来说，其实在调试 requests 请求时，调试到这个 http/client.py 文件库里时，就能看到，在开始 connect 时 http 版本直接写死成了 1.0，还有这个建立连接的 tunnel_headers，代码给了个空值

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqEU5Ks4oYSjlsWe4UN3iaYZ0eibv3b8ia3xWfwT10tDox8VVDqZ4dEPQjA/640?wx_fmt=png)

 ![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqbzmLgMy5QUnhzkxfEKA1nU4Ef8icTrtREhckoN3lUicIlsCNyiaIUqMew/640?wx_fmt=png)

如果你再配上 fiddler 或者 charles 抓包看的时候，明显能看到，在 python 发送实际请求前的 CONNECT 请求如下：

 ![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqJ3FJuxAnazmUVH15ibLOQbickUTj23ibYX1fciag53tlZvoV2DiaYBnweUg/640?wx_fmt=png)

而浏览器的这个 CONNECT 请求是 http1.1，且 headers 是有值的，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqHEqSYXUj4HZJs5lsaqoEcAw217AJFjggBIMiaRmUcRvFjn3Kd4ghlcA/640?wx_fmt=png)

所以，这很明显的差异好吗？是很容易被识别到的

而且根据我多方查阅，加上咨询了各位大佬之后，**python 目前只能改 Ciphers** **里面的算法套件，来生成非默认的 ja3 指纹，然后可以骗过检测不是太高的反爬机制。**

但是其他的 Extensions,EllipticCurves,EllipticCurvePointFormats 是没法改的，原因是  **python 跟 openssl 没有很直接的联系，python 发 https 请求最后还是借助 openssl 库暴露出来的方法，**也就是的 ssl_.py 里的方法 create_urllib3_context**，因为 openssl 库对外提供的方法或者接口是没办法这么高度自定义的，****Ciphers 部分也最多能改改算法，都不能给个自己定义的算法进去的，**而 chrome 可以访问是因为 chrome 有自己的 ssl，且 chrome 肯定是不会被禁止的（闹呢，禁止了浏览器正常用户怎么访问？）

也就是说这是 python 自身的缺陷了？所以我之前测试时不管用 requests，httpx，还是 aiohttp 都不行，因为这三个库底层都借助了 openssl 库发请求。

假如这种反爬手段满天飞的时候，python 层面还没有成型的方案解决怎么办？

WTF？python 的业务方向中引以为傲的爬虫居然有缺陷？而且这个问题非常致命啊，感觉被降维打击了，伤心的是目前还没法改变局面，你说焦虑不焦虑？

那有朋友估计会想问，“那为啥之前猿人学 19 题可以过？” ，那是因为 19 题检测的不严，**只要 ja3 指纹长度小于等于浏览器的指纹长度都可以过**，但其实还有很多特征的可以检测到的，来，看图，还是上次的图，看最后一个数有很大区别的，这个我在上一篇的结尾里也把这个问题抛出来了

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqoL3dhy57aic0ONRBZeahmHt5DJt9icG1d3GNpvgIgKcv5HaPwR8s4g0w/640?wx_fmt=png)

这么明显的特征，如果是一个实际的网站案例，你觉得他会放过你吗？

所以目前 python 针对 tls 指纹的有两个缺陷，**第一个发起 CONNECT 请求时 http1.0 被写死，headers 为空（当然这个可以改源码临时解决），第二个指纹没法完全自定义，有很多特征被识别**  

说到这里，有朋友可能不信，口说无凭，这次来个实际的案例，一个大佬给我的某网站

浏览器打开（抱歉，我必须马赛克马到位），正常访问是这样的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqfYMbQH8r3NW3603rqRzgrngF6yfAEWWIiahrBREAGxSZibuG2NgmAnBg/640?wx_fmt=png)

python 代码，这里我用之前介绍的简写形式了，只加这两行，其他地方不用改就行的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqY1QTqvNgiaEwozq4Czk6lS3nZ7QibvBQCFethEw8IqG3TcjT6u2vwr4A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXXInn8icXtHoGQflRGUCiasAqGickic1AkRJibzKb5U23QONMzb99yhicvOfqfXknCrLFKapLn9mBFEuvuw/640?wx_fmt=png)

运行时发现程序卡住且一直没有响应的状态，测试得知原因是这个网站强制验证了 http2.0 的问题，requests 暂不支持 http2 被该服务器识别一直不响应结果导致卡住。

那换成 httpx，（有关 httpx 详细的代码如何突破 ja3 的，可以看我之前写的这篇：python 爬虫 - requests、httpx、aiohttp、scrapy 突破 ja3 指纹识别 ）

发现运行还是不行的。

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq9ne8DBcXANf7ZUicI75pScMc0PxgicjzQCdGwZOO7M5yKRZQuqC2x0hg/640?wx_fmt=png)

怎么改都不成功的，这个就是实际例子说明了，这是 python 语言层面的问题啊，难道 python 真的有缺陷？

### golang 之 ja3transport 库突破 ja3

之后在蔡老板朋友圈里，青南大佬评论中说了是可以替换 ja3 指纹的，我请教青南大佬之后，他给了我一个方案，用 golang 突破，原文链接：[一日一技：Golang 如何突破 JA3?](https://mp.weixin.qq.com/s?__biz=MzI2MzEwNTY3OQ==&mid=2648981779&idx=1&sn=fda89c1ad8237456e734b4d6ac06df1b&scene=21#wechat_redirect)

核心就是，代码里加了第三方库，ja3transport，这个库可以直接伪造 ja3，刺不刺激？

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqs0BEwHNE1p78Qckjobwm9g6TDdtOoiaHCcV9wjKnssEic5RRESJoNYYg/640?wx_fmt=png)

一执行，一看，就发现真的改变了 ja3 指纹，惊呆！激动！刺激！兴奋！

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqsRUtR7o5CRGdsT35DmtnX68HY897qBicv52iaOm6ibbaCj10mPFYKRKfw/640?wx_fmt=png)

大概的看了下 ja3transport 的介绍，它在发起请求的时候，会将请求的 client hello 数据包里的 ja3 指纹修改为我们自己的给定的，这样就达到了修改 JA3 指纹的目的，妙啊

#### ja3transport 简介

这个大佬给了一段解释：

> JA3 的问题在于它仅比基于用户代理字符串的指纹识别客户端好一点。到目前为止，用户代理字符串比 TLS ClientHello 数据包更容易更改。JA3 签名的参数仍由 TLS 客户端控制，因此不能作为可信的信息来源。在 Jeff 和 John 的 ShmooCon 谈话中，他们提到 JA3 不是灵丹妙药。他们提出的颠覆 JA3 检测的方法是使用操作系统的 HTTPS 客户端绕过 TLS 客户端特定的 JA3 签名。我们提出了另一种通过制作与其他 TLS 客户端（如浏览器）匹配的 ClientHello 数据包来破坏 JA3 检测的方法

#### ja3transport 突破原理

官方解释：

> 要想颠覆 JA3，需要修改 5 个 JA3 参数，可以在 Refraction Networking 的 utls 项目 ClientHelloSpec 提供的 struct 中修改。该包允许用户构建和执行 ClientHello 握手。第一个参数，TLS 版本，可以用和成员修改。第二个参数，可用密码套件，可以通过更新成员来更改。第三个参数 TLS 扩展可以通过更新成员来更改。参数的一个问题是所有参数都必须遵循 TLSVersionMinTLSVersionMaxCipherSuitesExtensionsExtensionsTLSExtension 界面。第四个和第五个参数，椭圆曲线和椭圆曲线点格式，分别是 TLS 扩展 SupportedCurvesExtension 和 的一部分 SupportedPointsExtension
> 
>   
> 我们不是根据 JA3 字符串创建客户端，而是可以生成与 Web 浏览器等良性签名匹配的 JA3 签名。有一些预设允许 JA3 签名匹配 Chrome 或 Firefox，甚至更多。我们仅限于屏蔽使用 Go 可用的相同扩展的应用程序。例如，如果 Chrome 使用 Go 不支持的扩展程序，我们无法屏蔽它。屏蔽密码套件比较棘手，因为任何密码套件都可以进行广告，即使它没有实现。如果执行握手的服务器接受客户端通告但实际上并不支持的密码套件，则会出现问题。只要服务器接受实际支持的密码套件，虚假宣传比可用密码套件多的密码套件就不是问题
> 
> Go 的 net/http 库有一个名为 Transport. 传输结构负责编写如何将数据包发送到目标服务器。由于 JA3 的签名是基于 ClientHello 数据包的，我们可以进行 TLS 握手，让 Go 完成剩下的工作。该 Transport 对象是一个参数 http.Client 结构其中大部分进入开发人员都很熟悉。通过生成 Transport 结构体，我们的库应该可以与任何现有的 Go 项目无缝协作

浓缩下梗概，意思就是在 go 构建请求，三次握手之后，到实际要发起 client hello 包之前，ja3transport 把数据包拦截了，即上面说的 hello 包 hook 的方法，然后把原来的 ja3 指纹修改成了自传递的 ja3 字段发出 client hello，服务端就认了，然后就通过了。

太牛逼了，他不是直接修改的 tls 里的那五个数组套件，因为 ja3transport 作者自己也说了，要想突破 ja3，他认为的方法和选择的方法也是避开了直接改 5 个 JA3 参数，而是**在 5 个 JA3 参数创建好之后进行拦截替换**

那么也就是说，其实 golang 跟 python 比也并没有在语言上占有很大优势，唯一就是 golang 多了这么一群人提前就研究并写好了这个第三方库，而 python 没法解决只是还没有人写出能够拦截数据包并替换 ja3 指纹的库，也就是这个并不是 python 的缺陷啊。当然，如果你急于求得结果，我建议你还是学一下 golang 来解决问题。

更多的解释请看原文：https://medium.com/cu-cyber/impersonating-ja3-fingerprints-b9f555880e42

#### ja3transport 后续问题

作者也说了，如果你随意地伪造 ja3，假如服务端通过一些方式得知你客户端访问进程跟实际的 ja3 不匹配，那应该也会屏蔽你

> 我们能想到的最简单的检测改进是将 JA3 签名与生成 TLS ClientHello 数据包的进程映像配对。如果有客户端生成与 Firefox 匹配的 JA3 签名，但该进程不是 Firefox，则可能会发生一些奇怪的事情

#### ja3transport 缺陷

但是，用这个库 ja3transport 执行的刚才那个案例网站的时候发现报了如下错，提示就是说这个服务器不支持版本 304 的协议

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq2IBJAT441arTEeeTlruVibdunmqYQWcqOGBeBPJuF3sNtRqO0L6UibUA/640?wx_fmt=png)

> tls: server selected unsupported protocol version 304

一搜这个报错，就看到有个大佬说是因为不支持 http2.0 导致的：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq6nCZWLu5UebRRYRYfYX8NquoGdV6oKx9AHqoc5H5XZibz8TjjVg0Tew/640?wx_fmt=png)

对啊，这个网站上面它强制 http2 了，那也就是说，**ja3transport 不支持 http2.0，这就是他的缺陷啊**

### golang 之 CycleTLS 库 http2.0+ja3 指纹突破

那只要能解决 http2 这个问题，是不是就破了这个站呢？再看上面这个大佬发的日期：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqeKoNm3SLU6O9LkJC0UaLzZAibUjNrRXe8ibw46g919e3V4l2w5v8KP2A/640?wx_fmt=png)

7 May，那现在已经过去这么久了，大胆猜测一下，是不是这个大佬已经解决了呢？此时估计有朋友应该回想，“你这个人怎么这么不严谨，什么都靠猜吗？上一篇解决猿人学 19 题删一大片 cipher 加密算法解决问题也是猜的”

哈哈哈，是啊，因为我个人觉得，爬虫跟前后端开发还是有点区别的，前后端开发，按照我的理解，讲究的逻辑严密，事先考虑各个层面的问题，尽量少 bug 然后服务能够长期稳定运行，但是爬虫的话，有时候没思路的情况下真的是大胆猜测出来的，我现在的思维已经转向这个方面了，虽然以前也干过后端开发，但是思维已经回不去了。

我开始尝试了这个库：git 地址：https://github.com/Danny-Dasilva/CycleTLS，真的就是抱着试一试的心态，执行，结果真的跟浏览器访问一样的，搞定，牛逼！

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq04XWV4SQe8xvGKkiaRTuhekoKSnDF4LnbJ6bBfkhks3uLAPEhrdz3kQ/640?wx_fmt=png)

代码就不贴了，git 地址上有。

所以，也就是说，之前用的库 github.com/CUCyber/ja3transport，不够适合当下场景 http2+ja3 双重验证

这辈子没想到搞了几年爬虫逆向开发会为了解决 ja3 问题去学 golang![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq5uicMXicZDSoTq2I4w0UoD6B5MDibvM2RVAwFbKLVKaUxYyP11kkVf5gg/640?wx_fmt=png)

### nodejs

还是用上面大佬 Danny-Dasilva 的 CycleTLS 库即可，他也开发了 node 的库：https://github.com/Danny-Dasilva/CycleTLS#example-cycletls-request-jsts

直接 npm install cycletls，然后照着案例的代码来就行了，可能对于大部分爬虫开发来说，node 相对 golang 来说会更熟悉一点的。

### 验证 python 指纹

既然已经可以随意改了对吧，那我改成 python 默认的指纹访问下这个网站看看呢？我下面只改了 ja3 指纹，看运行结果，果然是被拉黑了，哈哈哈哈

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqEJFgYvg9Ljib3GGNQdCwpwCzQPo8t4wYIvGmASWDdfYvALOU5qStZtw/640?wx_fmt=png)

### 换接口验证

为了验证真的成功了，哈哈，该严谨的地方确实得严谨一点，还是用的 golang，我换了另一个接口地址执行，确实有结果了。

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqsWORicR3PLvtpMg05EZhtoXFMDm6iaaBias3QcNtd5NDdA3JSTUC90wlA/640?wx_fmt=png)

### 用 wireshark 做最后的验证

那边 go 程序在执行，还是这个被马死的目标，这边 wireshark 抓包看结果，是的，相信你也看出猫腻了，它居然发了两次 client hello 包

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqM0jQFHrb5ZQF0CVKlVqfQLXZnfWMW6APCnQBKnibnmAhiaRtic8RXQrUw/640?wx_fmt=png)

而之前我们看到的，比如猿人学 19 题，内部题 22，29 题的那个，实际只会发一次的，这里发两次就很奇怪，然后，这两次发出去的 ja3 指纹还不一样：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq4dIJicB1bdNBEXXaVl1662zC6ibO7Bv9H2CjRcNDagWaYStdyPSIssQA/640?wx_fmt=png)

哪里不一样，以逗号作为分割，除了第一个数和最后一个数，其他的字段的开头一个数组都不一样（以【-】作为分割）

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqRbQKZyswD9yNIJ9y5IVCNR04KaxsjWnTJibtzLicGu8JEWyO8MScnicwg/640?wx_fmt=png)

然后，此时此刻，我们先去 ja3 官网看看浏览器本来的指纹呢？

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqWwyUBrXdCh6fqzEnWuZvUga9fLAsp7iaicHHaKzs8qa6Y2ecVkPnjQww/640?wx_fmt=png)

> chrome 访问 ja3 官网返回得到的：  
> 　　　　771,4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,0-23-65281-10-11-35-16-5-13-18-51-45-43-27-21,29-23-24,0
> 
> chrome 访问目标网站用 wireshark 抓包得到：
> 
> 　　第一个：  
> 　　　　771,43690-4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,60138-0-23-65281-10-11-35-16-5-13-18-51-45-43-27-17513-51914-41,14906-29-23-24,0
> 
> 　　第二个：  
> 　　　　771,10794-4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,14906-0-23-65281-10-11-35-16-5-13-18-51-45-43-27-17513-2570-21,60138-29-23-24,0

仔细对比之后，也就刚好多了标记的

  
![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqHrSBG6OXesb0hAQs7N6WaW4pVia73zCdMLoqpDbdlmLKNjxTiaZAWscg/640?wx_fmt=png)

去掉上面标记出来的，其实第一个和第二个是一样的，仅仅是这个被我马赛克马死的平台是这样，我猜应该是这个平台自己多加了一层握手流程，所以会有 2 个 client hello，具体为啥有两个不纠结了，我也不是该平台的开发，也没法得知具体原因，不重要了，能获取数据就行了。

不过等等，突然有个疑问？为啥 wireshark 的 ja3 指纹，同一个 golang 脚本啊，就算两个 client hello，也应该是两个一样的 ja3 指纹啊，为了进一步分析，先用浏览器访问 ja3 后，同时看看 wireshark 抓包的是啥，这？我该信哪个？怎么 wireshark 多了点其他的东西

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqYUG9eEwb4Nv5xMOFoBxDTkWJJo4xVpd75jhtHD4qwJSnzDm3tD9aaQ/640?wx_fmt=png)

这引出了一个新的问题，哪个才是对的

### ja3 到底存在吗？ 

按理来说，应该信 ja3 官方，因为这玩意儿就是别人搞出来的，那既然如此，是 wireshark 在搞事咯？（此时此刻突然想起了学生时代的一个数学逻辑题，假如你是侦探，现在有几个嫌疑人，已知里面有个人说了谎，从几个嫌疑人的供词里找出真凶。。。扯远了）

为了进一步确认这个问题，根据青南大佬给文章的方案，可以伪造 ja3 指纹，那此时此刻，试下吧，现在的逻辑就是，我用一个我已知的指纹去请求 ja3，然后看看 ja3 官方的返回，再看 wireshark 的 ja3 指纹对比，如果 wireshark 里的 ja3 不全等于我们自定义的 ja3，那就确认是 wireshark 在他喵的搞事了

首先澄清，没作弊哈，我用的第一个，跟我浏览器里的 ja3 是不一样的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqiafibYI8QFVvuJyjJjb0KYMBxtdAI9NEg8V5YqvoibFB4FZRPMoMvenJQ/640?wx_fmt=png)

这里我用的 ja3transport 库了，因为 ja3 官网没有强制 HTTP2 了，运行下面的代码，发现返回的确实改变了的，且值正是我们给定的

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOq5uYfA6L1yLXRTaunIOwv0gNEnC3r1QpflJZibuhuYqJeH6XDu9PUic4A/640?wx_fmt=png)

再看 wireshark 这边：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOql3I1L5TLCoyPyIU28lkVQOl3tt1AeiceH5a5JXYGF4unrWk0AE7cgiaQ/640?wx_fmt=png)

发现也是自定义的那个值，没有奇怪的数组了

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqrSMgFOdWiab31w2LicqAibftWxnGviaBFSbmT3XRwAQicMG7ibrYbxicys9Rg/640?wx_fmt=png)

相信有朋友会问，这就奇怪了，为啥这里又一样了？难道 wireshark 没有骗我们？啊这。。。，不是吧，忘了吗？**ja3transport 这个库会在在三次握手后，即将发起 client hello 数据包时，拦截这个包，然后把自定义的 ja3 指纹替换原有的啊**，这是 ja3transport 库的原理，所以，这里 wireshark 能跟我们给定的 ja3 指纹对应上

那么，也就是说，**wireshark 有一套的自己的 ja3 指纹解析套件解析并显示了，但是这个解析套件跟 ja3 官网是不一样的，所以显示不一样**。当然这是我经过多次分析推理得出来的结论，查了 wireshark 的文档，暂时没找到相关的介绍和解释，姑且这么理解吧，肯定信 ja3 官方不能信 wireshark 啊，毕竟这 ja3 是别人搞出来的。

而且仔细看下面这个图，我鼠标放到 ja3 上面的时候，下面的进制数据并不会对应显示

 _![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqk4Qbd5X6IVuSVnaANkicJDAH9yfstpXcicviayt7pKLgXfd1NIjFEF1Jg/640?wx_fmt=png)_ 

用微信好友 chao 的话，"所有真实的信息都有二进制的数据，而 wireshark 的 ja3 都没有对应的二进制数据"

所以我跟 chao 讨论之后，认为 **ja3 其实并不存在**（上一次这么醍醐灌顶还是读到《三体》里的那句台词 "物理学不存在"，不好意思又扯远了。。。），或者说 ja3 并不是实质性存在的字符，而是通过 TLSVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats 这五个 tls 组件根据自己的加密算法另类存在的，因为 wireshark 都能通过自己的解析逻辑解析显示啊。以上推论仅代表个人观点，有误请指正

需要注意的点
------

1. 有没有发现，其实 ja3 在 2017 年就有了，据我了解，有很多公司的开发正在研究中。 

2. 有了一个 ja3，那我觉得后续肯定会出现升级版或者替代版了，红蓝对抗，反爬与反反爬，一直在对抗中进步，是好是坏，只能用时间来判定了。很多新东西靠自己一个人摸索是没法走到更远的，好比这个 ja3，如果一开始没有 Lee Brotherston 大佬在 2015 公开讲解，也就不会有这么牛逼的 ja3 指纹出现了，很喜欢微信好友 Regionover 在一篇文章里的一句话：【**故事要留给过去，但成长要用于分享**】

3. 另外，希望国内外能有大神可以仿照这个 ja3transport 库或者 CycleTLS 库写一个 python 的库出来，唉，我自己也想写出来啊，过去这么久了，我也在看原理，还得花时间研究啊。

后续怎么继续学习 tls 指纹
---------------

1.  练习题：猿人学外部题 19 题，内部题 22，29，32 题，32 题是 app 版的 tls 指纹校验，强的一批，网洛者练习平台第 9 题（这个题正常的浏览器都会返回假数据，作者看了一会儿我的文章一晚上就搞了这道题出来，强的一批），只提示下，有几页假数据。 
    
2.  志远大佬的最新课程里有 tls 指纹的，感兴趣可以整一个。他的方法就是自编译 openssl
    
3.  感兴趣可以读下 openssl 源码  
    

之前说的 tls 指纹研究怎么样了
-----------------

说来惭愧，搞了这么久，也没搞出个所以然来，我越去了解，就越觉得这个东西不是那么容易的，资料太少，全靠摸索，只能慢慢来了，因为我想实现的是完全自定义 tls 指纹，这个想法说实话，越来越觉得难实现了，不过一有空就在研究的。不仅研究突破 tls，也在研究怎么利用它更好实现反爬，欢迎有兴趣的朋友一起讨论。

结语
--

1. 真诚感谢一路下来认识的大佬们。以上都是个人见解，如果有误还望指正，有任何问题，我很乐意跟各位大佬们交流，我微信 id：**geekbyte，**备注来意

2. 另外应有些朋友的建议建了个群，里面很多大佬，安全渗透，爬虫，web+app 逆向，前后端开发，ios 开发的大佬都有，已满 200 人，想进群的可以加我微信  

3. 推荐下蔡老板的星球，web 逆向必须学的 ast 技术。他虽然不在爬虫圈，但是在爬虫圈一直有他的传说，我能有今天，能认识这么多大佬，很大一部分原因是因为他。真的很感谢蔡老板。

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqVIMaqte2XCqqSic1Ly70VdlwQ6qZc9zJcBM1nicxwIu5ibZgyWmCxibuGQ/640?wx_fmt=png)