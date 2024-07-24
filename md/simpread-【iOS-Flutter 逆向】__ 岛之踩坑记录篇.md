> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DLs5R6ibYJmc2-ao8nPoRg)

```
0




前言

```

    文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理。

    严正声明禁止用于商业和非法用途，否则由此产生的一切后果与作者本人无关。若有侵权，请在 vx【amuncocoL】联系作者。

```
1




抓包及定位

```

  

    样本版本:3.3

    Android 的话需要 libflutter.so IDA 静态分析 - 搜索 ssl_client 关键字 - 查看交叉引用 - hook 对应函数的返回值即可。

    iOS 该样本使用 Shadow***ket 开启 VPN 抓包可以直接抓到请求:

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtYLVorZKZRsfQNMvvUian2kVhODSia22AHpoBMibcJC52nvXd9mibfA9fGw/640?wx_fmt=png&from=appmsg)

图中可见 Header 中含有 signature 加密字段长度 32 位，无脑 trace 下 CC_MD5 看看是否触发~

使用 frida-trace：‍‍‍

```
frida-trace -UF com.xxx.xxx -i CC_MD5

```

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmt4oCVzibeAmnibHYOTENiclrTstHDt8xibxRIeRiaiaicRZLxHVLfXkC1iazTwQ/640?wx_fmt=png&from=appmsg)

上图无论怎么点击都没有触发 CC_MD5，考虑该样本可能没使用 CC_MD5 库 。

ps: 能直接 trace 到就没必要记录咯 家银们... 

即将开启踩坑之旅

```
2




脱壳




```

使用 frida-ios-dump 对该样本进行脱壳  

```
python dump.py xxxx

```

将 xxx.ipa 修改为 xx.zip 进行解压，Frameworks 目录下有 App.framework、 Flutter.framework 两个框架或者 mach-o 名为 Runner、user-agent 含有 Dart，基本上可以确定这个 App 是 Flutter 开发的。

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmt6ibAwY4xrwibutmlJ3gLCicKfBLickOMJJqibrLSsCpRd4Cia1WP9NTH88Ig/640?wx_fmt=png&from=appmsg)

```
3




reFlutter重打包




```

这里参考 Yang 神的文章，致敬前辈！

[https://bbs.kanxue.com/thread-273545.htm]

reFlutter 也很好安装直接 pip3 install flutter 即可。  

使用如下: 

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmticSA4AZvhsicia0YxoFrZ1b52lD6LfcXLPlgIAdtRnKlG6C5ibAuDNtexg/640?wx_fmt=png&from=appmsg)

我这里没有用到流量拦截，所以 BurpSuite 透明代理的设置直接略过。

可以从上图看出生成了 release.RE.ipa，接着对该 ipa 进行签名安装。  

我这里是直接使用的轻松签，不再赘述。

```
5




大坑-reFlutter重打包后app无法正常运行




```

排查日志如下：

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtyP5nYJeyCMhAE7TC9QMZzMibhzEX8SdysK0S728BfOTWRppnl6jcSOw/640?wx_fmt=png&from=appmsg)

 天塌了啊 家银们...

        遂又使用 blutter 对 Android 该样本进行分析  

  

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtQs1vrKvWJl1fyZVY1CzxtVibrpicRoFhz2GCBD2458ib0ibsQicwSOuohow/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtItyCp3sX7wZnibYKj2RIWxsGSSmYtBI6cn635bZ8nwcUtGEXjXtibYSg/640?wx_fmt=png&from=appmsg)

然鹅还是有很大部分缺失... signature 关键字也都没有搜到... 

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmt1FYFm7VubbWfmXtBKvMzq9Zicl5IE2q4ML7pM4eMuJLVicqhRykh6tMA/640?wx_fmt=png&from=appmsg)

天塌了 家银们...

无奈还是接着继续分析 iOS 吧

```
7




iOS-解析dump.dart

又回到了第⑤步

既然reFlutter重打包的ipa无法正常工作，

那就不管了。

signature看着是md5不麻烦，

直接dump出dart文件。

先看看函数名和偏移~说干就干！

    Mac打开活动监视器，手机USB连接Mac:




把dump.dart从手机scp下来分析函数

绝对地址和偏移~

搜索到md5、hash等关键字




```

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmthwkBaYEPTia5rP22TG8HAccKmGyqsHfad0uv018qOA1zUQUH8gEj7xg/640?wx_fmt=png&from=appmsg)

        最终筛出来_MD5Sink、updateHash、startChunkedConversion

        用 frida 分别 hook 下看看

        最终 hook method: updateHash 的入参发现明文的端倪~

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtUSMvylslyLapDhS9zzMqKtFrrZgmCmGapFbKoPgBib2oiag0LyMxhAibA/640?wx_fmt=png&from=appmsg)

    明文: 排序后的 k+v+salt 进行标准的 md5 加密

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8ribD7AqszEp4sjiaic9GBcNmtBaM3zaH6EZokcbibauwibcw7BNNic4mIveI5qwW7BsJANnGF3JeMvbX6w/640?wx_fmt=png&from=appmsg)

```
8




总结

```

本文旨在踩坑记录，大佬轻喷~  

文中有错误的地方，还请各位大佬斧正~

撒花~ ✿✿ヽ (°▽°) ノ✿下次再会。

`‍`