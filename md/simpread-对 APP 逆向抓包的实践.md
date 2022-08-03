> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzk0MjIxNzgwNg==&mid=2247485847&idx=1&sn=3853fc474f20ee8458337d00576c33c9&chksm=c2c7c6b6f5b04fa02305526ca1bf3cffbd9060df0deed070cced16202b3cd82b9ab4c9892266&mpshare=1&scene=1&srcid=0803sb0RwLZL1lK5lAtpglyY&sharer_sharetime=1659491361355&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

 ![](http://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wrIJ0a6Vfic8AnrKAUMuscw7NI0MZJHvddaGYdjArhsDvESBdpiaPlicyXTD5ZNpNicJo9JWHGZAkxOJQ/0?wx_fmt=png&wx_head=1) ** 编码安全 ** 专注于编程安全技术的分享。 48 篇原创内容  公众号

  

  

背景基础

  

  

在对某音国际版的 APP 抓包分析数据过程中，**发现该 APP 使用了自定义的 SSL 框架**，这就大大提高直接利用现有的框架抓包分析的成本。

现成的 Xposed 框架的 JustTrustMe 模块、frida 框架的 r0capture 脚本均无法正常抓包。因为在于这些模块脚本，主要针对的是系统原生的 SSL 框架，而该 APP 采用自定义了自己的 SSL 框架。那么如果我们需要对 APP 进行抓包的话，就需要过掉该 APP 的检测机制从而实现抓包数据分析。

下面从 SSL 框架原理，以及通过两个方式实现对 APP 正常抓取数据包功能。

  

  

SSL 框架原理

  

  

**SSL 全称是 Secure Scoket Layer 安全套接层，它是一个安全协议，它提供使用 TCP/IP 的通信应用程序间的隐私与完整性，**它是一种为网络通信提供安全以及数据完整性的安全协议，再传输层对网络进行加密，因特网的超文本传输协议（HTTP）使用 SSL 来实现安全的通信。

SSL 记录协议：为高层协议提供安全封装，压缩，加密等基本功能

SSL 握手协议：用与再数据传输开始前进行通信双方的身份验证、加密算法协商、交换秘钥。

在客户端与服务器间传输的数据是通过使用对称算法（如 DES 或 RC4）进行加密的。公用密钥算法（通常为 RSA）是用来获得加密密钥交换和数字签名的，此算法使用服务器的 SSL 数字证书中的公用密钥。有了服务器的 SSL 数字证书，客户端也可以验证服务器的身份。

SSL 协议的 1 和 2 版本 ，它只提供服务器认证，而第 3 版本添加了客户端认证，此认证同时需要客户端和服务器的数字证书。

  

  

SO 文件修改法

  

  

**该 APP 防止抓包主要通过自定义 SSL 校验来实现，那么我们只要将 APP 中检验的最终结果修改成校验通过就可以正常的抓包。**

通过对 APP 组成结构和代码的分析，该 APP 中自定义 SSL 校验是在 libsscronet.so 的文件中，那我们就可以直接将 APP 重命名为 zip 后缀，然后直接解压。解压后得到很多文件，这些 so 文件都是存在 lib 目录。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BzW56Qia0OybZW0fTwRYUdHP0rGgyeJh1YfSx84eFemoQ9pS4OxVX0BQ/640?wx_fmt=png&random=0.5107817715937286&random=0.4437142885558565&random=0.8643985682626978&random=0.18358382178224653&random=0.9200225867679888&random=0.8675189315024432)

接下来就需要通过利用 IDA 静态分析工具分析和 Hex_Editor 工具结合进行对 so 文件做修改。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BGPVicbRwliac3XnaYLShvydkbJl7OyUZiamXoxfqogibR2fTwQRGADgVPA/640?wx_fmt=png&random=0.35517003889110077&random=0.6553432919727913&random=0.7273709749339956&random=0.8510310523947129&random=0.5518901220988996&random=0.8760782990948659)

在 IDA 中通过字符串窗口中搜索关键词 verifycert，通过上图可以搜索到该关键词相关的，那么接下来就是通过代码交互引用然后跳转到代码调用的地方。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5Ba8he27kWkqfGYZHQpruPDSxA37jMouq3MUIsLhpJE0lXWJVGecWjicg/640?wx_fmt=png&random=0.9094629515821568&random=0.4895706973170193&random=0.9244700129345349&random=0.7000395376667508&random=0.6568506727514654&random=0.520025817421617)

上图的伪代码中，可以看到该函数中的调用关键字符串的地方，那么在这个调用的地方 return 0 的时候就表示校验成功，

那么我们只需要把图中的 return 1 修改为 0 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BySAvahj7ib0RcGyRL7lg2lCfXwfyFtbicrvpXHS8UNsNNDwiaTcY17Ofw/640?wx_fmt=png&random=0.8129390935107774&random=0.6351110302036673&random=0.24495117746823247&random=0.6141147575659427&random=0.840192550082514&random=0.3619511396648625)

通过借助 Hex_Editor 工具，将刚才 IDA 工具分析到到 return1 改成 return0，那么重新保存下就可以了。

最后将修改后的 SO 文件替换到 APP 安装的原目录下那么就可以开始抓包分析了。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BZiauSIYYtBJs4c9UlqPL2B7xpQT0of9c9WSjWJtRy9HHnyYjv5U9Vfw/640?wx_fmt=png&random=0.008995321819032398&random=0.37359076756167586&random=0.01813576684058238&random=0.8781896600599726&random=0.39897848343171827&random=0.4570350258307949)

上图是通过 Fiddler 抓包工具进行抓包分析的，Fiddler 抓包还得进行证书安装才能正常抓包。

  

  

HOOK 功能函数法

  

  

直接将 APP 拖进 jadx 反编译工具，**从代码中发现 APP 使用的网络请求框架是 Retrofit，而 Retrofit 网络请求框架底层使用依旧是 Okhttp，**那么 Hook 的切入点可以从拦截器或者初始化时传入的 OkClient 入手。

通过 jadx 反编译，从代码中确认了 APP 初始化 Retrofit 的位置

是 com.bytedance.ies.ugc.aweme.network.RetrofitFactory。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5Bm3YCadAKhchIcgl32MyHkiczOCtm9vz98EqPNKjRVDQNpIxbpq0pDBA/640?wx_fmt=png&random=0.790947925377137&random=0.5817932383017681&random=0.20772493595176234&random=0.5732521932351302&random=0.5372199084136549&random=0.7273032160455042)

Hook 代码的方式目前常见的就是用 frida 和 xposed 这两个框架进行 hook 功能

这边主要以 frida 框架进行 hook 功能，经过对这个类多次代码逻辑梳理及 Hook，找到了 X.MBJ 的类，该类中的 LIZ 方法的返回值 c 内有请求的所有数据，猜测可能是 response。

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BXp0G4ZF8qyPdXRYJXzDgaGL5XRMIqpLXaPzVqvNjbiax8IBKcGvbqQQ/640?wx_fmt=png&random=0.9630573088008707&random=0.11637566545947164&random=0.23388326649390057&random=0.09243183104271413&random=0.8422957919620995&random=0.6422487997237531)

下面开始基于 frida 进行编写 python 和 js 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BjyRZJKBTdfaMxwpWib31icuDSdouAC4AvGsEfnUQ0MV6k7OO0G5pJO0Q/640?wx_fmt=png&random=0.1427284593095821&random=0.4692418505925531&random=0.9860466656377482&random=0.466158383641174&random=0.4523998712972561&random=0.6003925241125072)

上图是 python 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BqgkVJ1xCa6jibA9NfG0j9kd4AcN5TXdKd7r9asfNWJbQE08m2bZqHqg/640?wx_fmt=png&random=0.4047366034347857&random=0.9930011759853525&random=0.10415683551432875&random=0.9125371619075107&random=0.696552286917453&random=0.5230606460216505)

上图是 javascript 脚本

通过启动 frida 运行后的效果如下图

![](https://mmbiz.qpic.cn/mmbiz_png/haqlQiars3wqFDiaaHbXheIJJQ2jM4bB5BSkhE1Mf3Xt7m5KuO4bkRYlrSW5yiafUMicKLOH0AVcu5ty5S9DzRSstw/640?wx_fmt=png&random=0.17586450933662023&random=0.6164356599830527&random=0.9437772144385197&random=0.5097234741424235&random=0.328122564821667&random=0.6488563689915949)

从下面数据中以及可以看到解析到通信数据了，这样的 hook 功能效果就达到，接下来就可以开始用抓包工具进行抓包分析了。

参考借鉴

https://www.jianshu.com/p/efd56ad78499

https://juejin.cn/post/7122723176387346469

结束

**【精彩阅读】**

[**android 逆向渗透测试必备清单**](http://mp.weixin.qq.com/s?__biz=Mzk0MjIxNzgwNg==&mid=2247485813&idx=1&sn=e61530dfd289ea5effed0c8f817a27c9&chksm=c2c7c654f5b04f42afc7740c5f3f7497b0536e7ac03926b18c0d3989eb715cf59dbd4089bc9c&scene=21#wechat_redirect)  

[**android 安全之 frida 框架**](http://mp.weixin.qq.com/s?__biz=Mzk0MjIxNzgwNg==&mid=2247485822&idx=1&sn=1a8dd86af405e845963b68ae199b478a&chksm=c2c7c65ff5b04f490643cd60781bd63202084e67b8b6aeddab51f050eada6ffa17404d2ac4a8&scene=21#wechat_redirect)  

[**对某 APP 加固逆向的分析**](http://mp.weixin.qq.com/s?__biz=Mzk0MjIxNzgwNg==&mid=2247485784&idx=1&sn=9ae66a1ab7e2cae738f0c62d9c8d5795&chksm=c2c7c679f5b04f6fd1baf685bb83f90200c69bf121cf18481c7890c3f7282f70a8c3aeb1c7bf&scene=21#wechat_redirect)