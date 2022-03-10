> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5NTY3MTc2Mg==&mid=2247483704&idx=1&sn=8e08162f0768fa6a793de5d761eff369&chksm=c00d8d85f77a04939dd5ce0481c8a8ca4426c25c7a1cd037f4568803f479bef2b355b80e6ee5&mpshare=1&scene=1&srcid=0201FpqYNSfgJft3lH6ugVUI&sharer_sharetime=1646873400933&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

前言
--

*   小小白在这里给各位大佬拜个年了，祝各位大佬新年快乐，大吉大利，身体健康，技术更上一层楼
    
*   本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！
    

网站
--

aHR0cHM6Ly93d3cuemhpaHUuY29tL3NlYXJjaD90eXBlPWNvbnRlbnQmcT1weXRob24=

加密定位

需要分析的接口以及加密参数 x-zse-96 如下图

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqP261tbMCynIBPQt7MibYoTg7w6oaJE391znh2kuGgvV1geA26cmTuicQ/640?wx_fmt=png)

直接搜关键字 x-zse-96，发现一个文件，点进去

![](https://mmbiz.qpic.cn/mmbiz_jpg/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqianSPE7iccYfRAmChrT8Pb5fr7uh6ASNwrvuudHicaPP5kuOcTMMSkCGw/640?wx_fmt=jpeg)

格式化后再搜索 x-zse-96，发现两个可疑入口，分别是这两个

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqibEE3P1z6zLKZgPX2wv25BwZxibQHQHV49ux1B4oKicGUEOnDou35IvUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqsyAnqgTljo9eq82vpN3EsulzjxAFK65IoLJQlibFoRrcg2cF5dwaWVQ/640?wx_fmt=png)

重新刷新网站，可以看到断点断在下图这个地方，选中执行函数可以发现加密入口就是这里

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqibF7O8e1MYGeHWDJ30Anzhbuc3RVVjSRJK0xLEyfJwiaMZqkXwc7l4Ow/640?wx_fmt=png)

加密分析

s 明文数据：

```
s = '101_3_2.0+'(版本号) +
    '/api/v4/search_v3?t=general&q=python&correction=1&offset=0&limit=20&filter_fields=&lc_idx=0&show_all_topics=0&search_source=Normal+'(接口后缀) +
    '"AIAf1GdgbRSPTtoYPsJrpvRp_MB-_8SxwGQ=|1643717522"'(dc0 cookie)

```

第一层加密

```
 l()(s) = '7dd6414484df2c210ddbb996c55cf64c'  ->  md5_str

```

根据 js 逆向的一些小经验，32 位密文很容易想到 md5 签名加密，拿到在线 md5 加密测试网站去试试

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqmDktpMQiakWPUtWAbZ5sxcHJV5hLXSXQuIs8WkHUz5WFRPJgViaImNpA/640?wx_fmt=png)

perfect，完美对应上，由此可知，第一层加密就是原生的 md5 加密

第二层加密  

```
a()(md5_str) = 'a0xqSQe8cLYfSHYyThxBHD90k0xxN9xqf_tqr6UqH9Op'

```

选中 a()，点进去，会进入到这个函数，加密参数正是第一层加密得到的 md5 字符串，如图所示

  

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqicQ18dvO6q7hh3Nkh67FnK8N4pezrRpR6Q3MJLlnaIzCyk8gzwz8exQ/640?wx_fmt=png)

然后就单步跟，跟到这里就先别跟了，这里正是 jsvmp 循环一条条执行指令的地方，下面的代码会根据时间来检测调试

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqElkf0BictGG3jAZ4xicVZmX4icAyibp5yUMH2puf8pBHBZfyx6Od8Jiboqg/640?wx_fmt=png)

  

然后就在 35865 行打上 log 输出断点，把 this.C 索引值跟 this 对象里面的值都给打印出来，注意要把 window 除去，不然会缺数据，索引值方便后序调试可用

```
"索引：", this.C, " 值：", JSON.stringify(this, function(key, value) {if (value == window) {return undefined} return value})

```

慢慢等待输出结束，掉头发的逆向之路就可以开始了  

直接在输出的 log 日志里面搜索加密生成的密文

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqqBedKV7Or179TRnwf5tVblVYTiaTyNb7PV77MgliaT6HTExzPCc40BxQ/640?wx_fmt=png)

这个 vmp 生成的加密参数是分成 11 组来计算的，每 4 个一组

这里面有一个固定不变的字符串，就是这个玩意

```
fix_str = "RuPtXwxpThIZ0qyz_9fYLCOV8B1mMGKs7UnFHgN3iDaWAJE-Qrk2ecSo6bjd4vl5"

```

拿第一部分 a0xq 举个例子，看看他是如何生成的，其他的都差不多  

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqnkKicqyvUDjf4jEiabbYtHe2hiaUEQzBzryR0w5E5nm82QibjU7iciaeuhEQ/640?wx_fmt=png)

fix_str[13] = 'q'  

q 就是这样来的，每一个字符都是通过索引那个固定字符串得到的，所以，接下来，就该来寻找索引下标是怎么生成的，向上寻找，看到这个

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPq2WS6CopzpBAXicVGIhzONuUCJmzMy6HtU6VNs6eicyLmwibk4ORLs4vSQ/640?wx_fmt=png)

  
3433258 >> 18 = 13

这里的 18 是固定的，其实判断是不是固定的数字，只需要对比两个控制台里面 log 输出的对应分析就可以很清楚的知道哪些是固定不变的了

再来搜索 3433258 这个数字怎么来的

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqGCD3GB5xTEDU7sYdxKozljpbPYJvrDONwBqia4eD90drKfic4lfiaAhoQ/640?wx_fmt=png)

可以很清晰的看得到，下面这几种运算都可以得到这个数字，那怎么判断是哪一种运算得到的，我的办法是：调试

```
1. 25386 + 3407872 = 3433258
2. 25386 ^ 3407872 = 3433258
3. 25386 | 3407872 = 3433258

```

重新打开一个浏览器打开链接，在 log 输出相同地方换上一个条件断点，断点的条件是 this.C = 342，具体跟进去看看他到底是进行的什么运算

  

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqPFfM2tE8tHVwt5WCCUs24IVqL0fplic07c9KSzCkia2Sv1Xib29vOeBBg/640?wx_fmt=png)

  

这里这一步我就直接说了，是第三种，| 运算得到的，知道运算符之后，又要去分析两个参与运算的值是怎么得到的，这里就需要慢慢去一步步去逆向搜索分析了，所有参与运算的值都可以用这种同样的方法搜索分析得到  

这部分所有的运算流程都在下面了，请各位大佬们自行慢慢对比 log 输出去参悟，理解

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqlSua5j0qpucYCb4k33FK0eVLwQfYPCdy6KYj55phlME30owTVAnVaw/640?wx_fmt=png)

最后验证：

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqJzhLgSsIW2WyP9xgeOPovrUIrwJibwkGuVO81toNlGI3dJu2EQVNTrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqic5ibgIQVlBOjYKCP6VTalmSUMUrxLsXv5P5c2T76feqnZmDyHhaW45A/640?wx_fmt=png)

  

可以发现，算法还原下来只有短短的 50 几行，而且最后本地生成的加密参数跟浏览器生成的完全一致，完结撒花![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqqozA2Q2bROFIyQf9pNECcly4F8z9gvicniciagmbt63mYSdsBKFfyDxFw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqqozA2Q2bROFIyQf9pNECcly4F8z9gvicniciagmbt63mYSdsBKFfyDxFw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k3LlZI6mUqIP5k6v7cPBHPqqozA2Q2bROFIyQf9pNECcly4F8z9gvicniciagmbt63mYSdsBKFfyDxFw/640?wx_fmt=png)