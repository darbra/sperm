> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIxOTQ1OTA0Ng==&mid=2247484368&idx=1&sn=c743dd5f76188d44402af4db21069707&chksm=97dbbf59a0ac364f46f284b7d7d862c4adc3129a574663da7a15bb87cd68d100244ef47c43f2&mpshare=1&scene=1&srcid=0622sFj8mD27Sjl5PB26hGbU&sharer_sharetime=1655877089339&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

堂主以前只用过 sql、spark、hive 做过数据处理，最近学习了一下 excel 数据处理，发觉功能很强悍，特此通过某买菜 app 的数据进行相应演示，希望对小伙伴们有所帮助。

1 app 抓包体验
----------

经测试，一共进行了三个参数的校验，分别是 sign，sesi，nars

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUNRNTiaZt4cbccFjic5ZI4Pk7rOPCyibcFUT68a0dWZibErIeMoPibjjLXKw/640?wx_fmt=png)

2 逆向体验
------

使用 https://github.com/lasting-yang/frida_dump 进行脱壳，jadx 打开。

sign 参数在 java 层，位置是 e51。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAU0qKKsFDLhJJTv8QF2u6IqCcYj0HeyFqdKDXprGeT9wlFMBe9DS45Ng/640?wx_fmt=png)

sesi、nars 参数在 so 层，位置是 libli11li1.so

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAU2CqSuN1Fqy9s51K0iazgQp7Op6Tx91ln1dJeITib90aEsLX5NZBFNUjg/640?wx_fmt=png)

使用 frida 以及 unidbg，成功地还原出算法。浅浅地总结下，sign 是加个 privatekey 后的 md5，sesi 涉及到两次魔改 md5 及 base64，nars 涉及到一次魔改 md5 和一次 aes-cbc 和一次 base64。

这次主要是讲 excel 的初体验，对参数感兴趣的小伙伴可以自己尝试下。

3 excel 体验
----------

### 3.1 透视表

我们用 wps 打开获取的数据，随便在数据中点击一下。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUTWc14FkorQ3cvZgnbib5YtYibOPGWC74nib23zia9eht5Tqv1Ufnw9HCBg/640?wx_fmt=png)

接着点击模块【插入】中的【数据透视表】。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUkdqRgM4kdqzlOdoLtZz51rUxTYa4RUZ6ibOpP4Mw52LhtQUYHg1qI1g/640?wx_fmt=png)

点击弹出框中的确定按钮。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUtS24bo1vKUodZbvnTdyDwFN53b0o62dsRazlaHrguViaCYYZl9rntjA/640?wx_fmt=png)

我们可以看到下图中的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUyl4InSg2LDm1jga1micdwmvONjJ4k7cicPicZFNp15oucxdgNDuCicicbPQ/640?wx_fmt=png)

接着我们将 city 移入行，cate_name_1 移入列，price 移入值，我们欣喜地发现，透视表（如下所示）映入眼前。其中的值是每个城市的每个列表的商品计数。一行代码不用写，就出来这么完美的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUMyxjIXbtBcE9GbibcKndbRicvahGMUl1lWFwsVOeYA4kBticxYiaV0xC4g/640?wx_fmt=png)

随后，我们进行值的探索，我们点击计数项：price 下拉按钮。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUnsuD4opEmnzZiaOMlDqzDzdeQpIFqQRde4rJpHjPFeWcIoPEsRx17iaA/640?wx_fmt=png)

选择 “值字段设置”。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUibcwBbeURnHgribTsNnN5dfjjs7EyMB8myOHv42sYVC04wiaUXP0cbKibQ/640?wx_fmt=png)

点击平均值。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUeamQvtwWGueYjVLDXfsfiaPl7IfrZiag2LzRoZYr8WDiazIPjL42kSzRQ/640?wx_fmt=png)

我们发现，值已经变成了每个城市的每个列表的商品价格平均数，效果蛮不错的哦，啥代码都不用写。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUPiaCJ7u7icARciaCoUK3q7I6UcT2HT2mkMzkOmALbzRUrX4Fbw92ObxUQ/640?wx_fmt=png)

透视表更多处理小伙伴们可以自行尝试。

### 3.2 VLOOKUP

我们发觉数据中的城市是拼音，比如 shanghai，hangzhou。我们想替换成中文，怎么操作呢？这时就介绍神奇函数 vlookup。

我们首先添加拼音以及对应中文的映射关系，如下所示。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUfGRSTfJWxdlV3aoJTgTXrM8u6m5tMnIrKOHEuZIpEKwDHI98ooe2pg/640?wx_fmt=png)

接着我们在新的一列中输入 =VLOOKUP(A2,K:L,2,FALSE) 回车键，直接跳出中文了。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAU7UdzQmexPbcnmJGhice7lsgMA1Vf6DRibSLUx5zF8clvOMNzH6q9piaxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUxGBxSkw0MD1lQUic0Ccx4BnIqDPDUzEjAmQxqDH3FJ5VoCXvX1DsvxA/640?wx_fmt=png)

下拉一下，预期的中文映入眼帘。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1Z7h19LicjD3LwJuaSMibvQAUVxZxfRJMrjuRoBOBIBxDye00f0icCH3ZFBjZpwu6PXMbuzIYJ4Dk4PA/640?wx_fmt=png)

4 总结
----

数据处理 excel 相当强大，大家都可以去学习下，没必要都搞成 dataframe 处理。

更多精彩尽在 https://github.com/darbra