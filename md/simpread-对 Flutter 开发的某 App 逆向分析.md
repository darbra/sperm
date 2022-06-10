> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2NzUzNzk1Mw==&mid=2247492329&idx=2&sn=8dbf8e6292d094217e97f2056ae31e4d&chksm=ceb8a7a7f9cf2eb1085c554d939be7abac67f36863bae660ef5c06144e1574cdebad948e690b&mpshare=1&scene=1&srcid=0610jyosBxoUxDMn9H1oDPEt&sharer_sharetime=1654839849678&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

哆啦安全 ，赞 30

前言  

-----

最近总感觉时间不够用，很多东西都堆着，答应给朋友看的某 app，也没来得及看。所以也就一直没有空发文章，当然我也承诺过，尽量保证文章的质量，所以不能随便水文章，要嘛就不发，要嘛就发质量我觉得过得去的。

这不刚好有个 app，可以记录一下

app 名：

```
dGVj{deleteme}aC5lY{deleteme}2hvaW5nLm{deleteme}t1cmls

```

开始抓包
----

一开始用 charles 链接 wifi 代理去抓包这个 app，发现只能抓到一点点无关紧要的包，需要的数据接口是抓不到的：

抓包工具显示的是 connect:

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE0PWSwQiaPWVwBUV4ccwQYNTg8vIqREEcjBiaiayA5kJOBZZEUsy4z12icg/640?wx_fmt=png)

app 显示界面也是空的：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEZn52G7icW7icYqZMFYtLpwj7Os94u3RMzWib2P5iaC71IayPHHQjvXmFag/640?wx_fmt=png)

所以以为 APP 可能设置了 no_proxy，使用 postern 开启 VPN，因为 VPN 抓包的方式可以解决 no_proxy 的问题，但结果 toast 提示【null】，以为 APP 开启了双向验证，逐一去尝试解决，情况跟上面一样，都没有抓到包。

那么用 r0capture 抓，也没有，很骚啊

抓不到包，感觉直接一头雾水啊  

尝试 hook 打印堆栈，突然看到了 flutter 的字样，才意识到这有可能是用 flutter 开发的 APP。

什么是 flutter
-----------

Flutter 是 Google 使用 Dart 语言开发的移动应用开发框架，使用一套 Dart 代码就能快速构建高性能、高保真的 iOS 和 Android 应用程序。

由于 Dart 使用 Mozilla 的 NSS 库生成并编译自己的 Keystore，导致我们就不能通过将代理 CA 添加到系统 CA 存储来绕过 SSL 验证。

flutter 打包的 apk，会把核心逻辑放在 so 层，且 SSL Pining 也在 Native 层，这就导致没法抓包

怎么判定 app 是 flutter 开发的
----------------------

把 apk 包复制一份，后缀改为 zip，然后解压，进入 lib 目录，如果看到有 libflutter.so，那就是 flutter 开发的了，否则则不是

而今天这个 app，查看，确实是 flutter 了  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEAeYjSmdgwKLlvHJ3jp2LXNibajk2o7kiaAQnQABUBZhabqicOBg2M76yg/640?wx_fmt=png)

反抓包分析
-----

为了解决这个问题，就必须要研究 libflutter.so 了

**当然我搞得这个 app，不是很难那种**

一开始是在看雪里找到这个帖子：  

https://bbs.pediy.com/thread-261941.htm  

然后跟着操作了一下，在打开 app 的包文件，用 IDA 去打开 libflutter.so（注意 32 位与 64 位）

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEvicEgyVvyOpeLGZcm2x1MQjyCKKakODXibTH94WiakRiaEKP5iar40lenkA/640?wx_fmt=png)

然后点 search->text  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEr6BgGIpHle1INrCnasia6lbBBv78oA5bHUGdpITs1Fq8OsSHtKKxsiag/640?wx_fmt=png)

输入 ssl_client，

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEdfdugCZlibMKYtyRUkNFgOvyNyOxIIsNcbJibicBpfeoXWTmJj00xAHbw/640?wx_fmt=png)

点 ok，等一会儿，找到有【Rx,PC ："ssl_client"】之类的字眼

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE9FgH9ibTxTbhziaz1BsmAen4rhmCBOiaNb3JdeOUXfl5QQ8QspUTGg33w/640?wx_fmt=png)

点进去，进入这里：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEDWvUUicHU4mEA0BLkmapNWJQ0dddW0r6ACvIuTRVADVGSdVJjbYibX2w/640?wx_fmt=png)

按下空格键

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEElYF8ezJ9ZGTmBMep6Ww1K0icbWNPGMru50JDqGLiatMuIM3ib4MRp5YA/640?wx_fmt=png)

这时候，点 ida 里的菜单栏，options->General，把这个由 0 改成 4（这个步骤我找了很久，请教了奋飞大佬找到的，ida 操作不熟没办法）

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEgjuibLBXDiaNBDhWkCZwOn5vSBbEicoYKVF0MTroPDWjdBhMibac4s7rJA/640?wx_fmt=png)

Number of opcode bytes 设置为 4

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEgWmVov8WJYficFTL800R6lGkDqT8lvKicEGFXb6lGSIGDU4LeibMiaxUTA/640?wx_fmt=png)

点 ok，现在再看，已经多了点东西了

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEU8YtsicqXeIraOIvjOovOoLN6GlGToK3861Idxk5Mg2B2Do5T7d1Nnw/640?wx_fmt=png)

然后当前位置往上找，找到有_unwind 开头的，停下来，从_unwind 开始，拿到字符串前面 10 个字符（也可以大于 10），照这个前 10 个字符，我尝试好久才知道是要从这个 unwind 开始拿字符串，而不是在找到 ssl_client 位置开始拿前 10 个字符串，上面看雪那个帖子也没说清楚（也可能是我菜，没读懂）。

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE0Nqg3VO2ah7T08ohwsfiaPlxQQ0QDIOxMg29nZJX0391aZDRRibYPPGw/640?wx_fmt=png)

用那几个字符串，加上 flutter 的 hook 脚本，配合 drony 或者 postern 就可以抓包了（亲测过，postern 一样可以抓，配置起来没有 drony 繁琐）：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEncNFBh7zLtAcXBAhsvMImUPKY8Yzxmqv6fIsMrJIRwQL2TtPepXUEw/640?wx_fmt=png)

用 frida hook attach 模式启动，同时重新刷新页面

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE08ia7JOOrWTFjSibedNYDWzlxBurwzaia9JQbAxXQheURKpicVia5ibHMWaQ/640?wx_fmt=png)

果然抓到了包，而且 app 页面也正常显示数据，不好意思打的码有点厚，不过不重要，这种结果就是有数据的意思

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEzUG5ibEkwlFhCjhrvicRanjp21Goicib0PMLicFDsNNXmzbLgrvfaSibYM1g/640?wx_fmt=png)

ok，抓包搞定了，现在就看看有没有啥加密参数了

加密参数逆向
------

再看刚才的接口，就一个加密字段 sign:

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYErceFkbDxXwwcXiboL6dA6We63ESCbXH3L4YQ1nrdumThBmkkBicPia35g/640?wx_fmt=png)

所以，找找这个怎么生成的即可，打开 jadx，搜索：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEtc4TepO46VDuqUKwHHIh6VzkyZqpZa1JeNLE8F23F1VoQrA6w6AEWQ/640?wx_fmt=png)

就两个地方可疑，点进去：

第一个

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEI2lRtXBAlwgjKotFh4gqGWyDptG2vsffvIWVaNDQmoicNuWM3b93PPQ/640?wx_fmt=png)

第二个：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEZz94hmuj5cFygiadc2AMy2vIydibmiblVvQ8pk2SaKZUSicnpnfTu7Km9g/640?wx_fmt=png)

而实际上，第一个里返回的最后也调用了第二个里的，所以实际就是第二个了，写个脚本对它进行 hook 操作：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEkRovTCPOKqBMICEugOAATk6w42icEnjPREk6Vib9zMJico3O4qamR5NLg/640?wx_fmt=png)

看结果，对照下抓包工具里的结果：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE2PEqqiciaXg0E5RVfOIbSVU7cLUfMlicxia00WLCic2mL7vek3aYbwzVZYw/640?wx_fmt=png)

有点长，我复制出来对比下，一模一样，那就是这里了  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYESlsIHxgL3VmTWG1XeNwgNJ8kVjJH9zKviao4HnYiaEjPtmMhDG6gJSXA/640?wx_fmt=png)

相信你发现了，这后续流程跟常规的 app 逆向没有任何区别，那是因为这个 app 并不难，如果是那种所有逻辑都在 so 层，就难搞了，这里的加密逻辑还是在 java 层，只是一开始的抓包就把部分朋友难住了

不废话，继续看，这个看来就是个 rsa 加密了，当我正要分析的时候，程序意外崩溃了，这个不要紧，估计没有返回正常的值导致的，那行，既然找到就这个位置了  

打印堆栈跟下加密逻辑，顺便看到了加密的 privateKey:

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEI8nv7W4KtniabibXRpJDIDvPvQ69H9TTvPes3o4oGmQ21L5UuzibIlDHg/640?wx_fmt=png)

而其实刚才那段源码里，这个 key 其实就在了  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEQicoqicTQXvjNy6E881SylMuibBktsibHFQ7dukgQB26iaYiaiaocqaiboibVhQ/640?wx_fmt=png)

接下来就找传进去的第一个参数了，回到刚才的逻辑，第一个参数是这个，  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEVXFbaWPyDwS4ZoIxmMyhQgibfWmaP98iaiaaALUWeUw0ta9nWfa0BZc2Q/640?wx_fmt=png)

直接对这个方法进行 hook:  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEHTBwaicl7r3wCG5rC82JecJmn5oia06rWVbCkwEFV12NvLY2ac8Buhvg/640?wx_fmt=png)

这 str 不是很眼熟吗，就是一个 url 啊

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE42w9hibuLOf1tf9DEv6tZ8VYmj8sDau1Fg8yT1vGib9n7fzP8qqvFIhw/640?wx_fmt=png)

那这个基本也稳了，不过注意的是，有的 url 里面的参数里就多了个开始时间，结束时间，和当前时间

python 代码还原 + 验证
----------------

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYEU7LWDibJ6sOraXTQFRjN9N7Miawq8WbsAEORIyBFJkax659iaMIcmCdFw/640?wx_fmt=png)

hook 的结果：  

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE4KIgjAfxHiaibApDFWno2DtxBVMoIvmneoBLY3XtfooXM20g4WmQHqSA/640?wx_fmt=png)

对比下，完美，一模一样

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXUjwzq5lCqicPtwIo1GngaYE2TpjLZ7dB7OPic0icDR98qJj9RFodo3FGEfzRHiamk6GA7xux8KKBl6Fw/640?wx_fmt=png)

然后这个接口有的是带了时间的，有时间的加密结果也一样，我就不贴图了

python 代码，（感谢成都逆向天团里颜值最高的徐少给的代码），我就不自己重写了

```
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Hash import SHA256
import base64
import warnings
warnings.filterwarnings("ignore")
def RSA_sign(data, privateKey):
    private_keyBytes = base64.b64decode(privateKey)
    priKey = RSA.importKey(private_keyBytes)
    signer = PKCS1_v1_5.new(priKey,)
    hash_obj = SHA256.new(data.encode('utf-8'))
    signature = base64.b64encode(signer.sign(hash_obj))
    return signature
def main(startTime=None, endTime=None):
    data = '/cactus-api/posts/by-tagtagIds%5B%5D=4&orderBy=updatedAt&offset=10&limit=101648403473200'
    privateKey = '''自己用jadx打开放到这里吧，太长了，占篇幅'''
    res_sign1 = RSA_sign(data, privateKey)
    signature = res_sign1.decode('utf8')
    print(signature)
if __name__ == '__main__':
    startTime = 1648051200 # 有时间的接口这么用
    endTime = 1648137599 # 有时间的接口这么用
    main(startTime, endTime)

```

题外话
---

对了，网洛者爬虫练习平台的站长 LeeGene，他读了 python 的 requests 源码后自己用 golang 实现了个 requests 库：https://github.com/wangluozhe/requests，支持 http2.0+ja3 指纹修改，经作者自己测试，可以过猿人学内部题 22 题 + 29 题，赶紧学起来。

结语
--

终于算是对 flutter 有个大概的逆向操作流程了，其实本次目标并不难，如果是那种很难的就难搞了。这个 app 其实还有某盾加固，但由于不是一线大厂加固，我直接忽略加固一样操作

那遇到那种难的，把大部分的逻辑都直接编译在 so 层的，怎么办？dart 反编译吧，但是反编译的效果不是很好，目前没法通杀，只能多方尝试了

参考链接
----

> http://91fans.com.cn/post/fruittwo/
> 
> https://github.com/google/boringssl
> 
> https://rloura.wordpress.com/2020/12/04/reversing-flutter-for-android-wip/
> 
> https://github.com/rscloura/Doldrums
> 
> https://bbs.pediy.com/thread-261941.htm
> 
> https://mp.weixin.qq.com/s/Ad0v44Bxs1LFy93RT_brYQ
> 
> https://tinyhack.com/2021/03/07/reversing-a-flutter-app-by-recompiling-flutter-engine/
> 
> https://blog.tst.sh/reverse-engineering-flutter-apps-part-1/
> 
> https://blog.tst.sh/reverse-engineering-flutter-apps-part-2/
> 
> https://github.com/hellodword/xflutter/blob/main/snapshot-hash.csv
> 
> https://raphaeldenipotti.medium.com/bypassing-ssl-pinning-on-android-flutter-apps-with-ghidra-77b6e86b9476
> 
> https://github.com/G123N1NJ4/c2hack/blob/0f85a9b0208e9ee05dcbfb4fbcebd1c7babc4047/Mobile/flutter-ssl-bypass.md
> 
> https://github.com/ptswarm/reFlutter?msclkid=41b563c4ab7a11ecad34e0a8139bac19
> 
> https://github.com/mildsunrise/darter
> 
> flutter app 的流量监控和逆向库
> 
> https://github.com/ptswarm/reFlutter
> 
> dart 反编译，只能编译 2018 年前的
> 
> https://github.com/hdw09/darter

哆啦安全 ，赞 16

**推荐阅读**

[Android JNI 动态库逆向](http://mp.weixin.qq.com/s?__biz=Mzg2NzUzNzk1Mw==&mid=2247492242&idx=1&sn=55e534a3701ec1929ebb146807f379d3&chksm=ceb8a7dcf9cf2ecacd30bb469adacadb56308e8a6367c1f8d913bf342f8ed31ad97398832384&scene=21#wechat_redirect)  

[JNI 与 NDK 编程 (基础到精通) 最全总结](http://mp.weixin.qq.com/s?__biz=Mzg2NzUzNzk1Mw==&mid=2247489894&idx=1&sn=59e1f782914a3c46833d563e96d03501&chksm=cebb5c28f9ccd53e81d919fdd5ee5904d2df7edc15e90cf9acc499a44036f665b89ef837fbdc&scene=21#wechat_redirect)

[Android 和 iOS 逆向分析 / 安全检测 / 渗透测试框架 (建议收藏)](http://mp.weixin.qq.com/s?__biz=Mzg2NzUzNzk1Mw==&mid=2247492319&idx=1&sn=3b5396e0d19dd5583466c34116bc0ce7&chksm=ceb8a791f9cf2e871bfe88c9635a13b1ed16be6764eea51bd2c6c53ec85859712aad68cf2053&scene=21#wechat_redirect)