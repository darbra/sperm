> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247488564&idx=1&sn=0d0faad2a8a6354e10b5b112a37af3d1&chksm=c06f3d28f718b43ef5c87368a3da8589f411aaf638132da9f12f2c01ef65c82c33ad6a586b23&mpshare=1&scene=1&srcid=0701bMWNjsSIl6CoCi6Vh71b&sharer_sharetime=1656671755023&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

![](https://mmbiz.qpic.cn/mmbiz_gif/kicB09lvgibnnRjv0AAqQxyBODIttZXnQqcTPoF4Pt8tJmnia4CHaYUS3zqicFfKZTWibXTAew2ibFHDjy5Pf8nDnVEQ/640?wx_fmt=gif)

点击上方「**蓝字**」关注我们

![](https://mmbiz.qpic.cn/mmbiz_png/kLQoJJzjYaicxneNzbOg7ynx3TfnIwmNTpJQ7orkaUNrJIV4u7PNdSJ25Mtn6XdRQTamLDDicHnYfdic2bsiaNQjCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ly70P3iaKrDXsvQx2ewlNoLbic2Xg1W2GoCSgz5LzxibowSickHY8MdP8H1FnpALtuxrGWSic9o1hVH1A/640?wx_fmt=png)

声明
--

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

**本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号【K 哥爬虫】联系作者立即删除！**

前言
--

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE10Qajma91A8VAIo66wIWTPWQL7Jq5yYKKKvgjJLRlbO6xNf8p6Tlyw/640?wx_fmt=png)

瑞数动态安全 Botgate（机器人防火墙）以 “动态安全” 技术为核心，通过动态封装、动态验证、动态混淆、动态令牌等技术对服务器网页底层代码持续动态变换，增加服务器行为的“不可预测性”，实现了从用户端到服务器端的全方位“主动防护”，为各类 Web、HTML5 提供强大的安全保护。

瑞数 Botgate 多用于政企、金融、运营商行业，曾一度被视为反爬天花板，随着近年来逆向大佬越来越多，相关的逆向文章也层出不穷，真正到了人均瑞数的时代了，这里也感谢诸如 Nanda、懒神等逆向大佬，揭开了瑞数神秘的面纱，总结的经验让后来人少走了不少弯路。

过瑞数的方法基本上有以下几种：自动化工具（要隐藏特征值）、RPC 远程调用、JS 逆向（硬扣代码和补环境），本文介绍的是 JS 逆向硬扣代码，尽可能多的介绍各种细节。

瑞数特征以及不同版本的区别
-------------

对于绝大多数使用了瑞数的网站来说，有以下几点特征（可能有特殊版本不一样，先仅看主流的）：

1、打开开发者工具（F12）会依次出现两个典型的无限 debugger：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEYEzbsBqFR9ia6gEIsGCHFoMqZ6cfficpc1NaOebh0kteQEl1N7yOmT2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEhCmHHNp1DA0VpNVYia7cZDpKysK00a5MtW9RjZrIankCbLqn2SdZ8eQ/640?wx_fmt=png)

2、瑞数的 JS 混淆代码中，变量、方法名大多类似于 `_$xx`，有众多的 `if-else` 控制流，新版瑞数还可能会有 jsvmp 以及众多三目表达式的情况：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEEmxuAZ4ic1Mf3f4TJak8klvbzibNeiaxMeq7KVvqU9oul63A2icKicqibChQ/640?wx_fmt=png)

  

3、看请求，会有典型的三次请求，首次请求响应码是 202（瑞数 3、4 代）或者 412（瑞数 5 代），接着单独请求一个 JS 文件，然后再重新请求页面，后续的其他 XHR 请求中，都带有一个后缀，这个后缀的值是由 JS 生成的，每次都会变化，后缀的值第一个数字为瑞数的版本，比如 `MmEwMD=4xxxxx` 就是 4 代瑞数，`bX3Xf9nD=5xxxxx` 就是 5 代瑞数：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEkYzaIEpcXgUpLLmTahVyMtXAib2bvrag5QLkhPLKF604CCC4OH1zRYw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrERjX6uuukOnMbTe8AObhkItT0J6ic7HCCkY8LyiczZjxv5MDO5m8QIhvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEs1dGX8G6y5XX5o9lMibwyBib3GO1ludCX0giaMRz5Ino0GdBHlnbaenWQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrElqM9byEmI2FLbricjibFMqyuCrZSiamwt8Y8DyDp5ER0wPcf1TDSN7t6A/640?wx_fmt=png)

4、看 Cookie，瑞数 3、4 代有以 T 和 S 结尾的两个 Cookie，其中以 S 开头的 Cookie 是第一次的 201 那个请求返回的，以 T 开头的 Cookie 是由 JS 生成的，动态变化的，T 和 S 前面一般会跟 80 或 443 的数字，Cookie 值第一个数字为瑞数的版本（为什么可以通过第一个数字来判断版本？难道相同版本第一个数字不会变吗？这些问题我们在分析 JS 的时候可以找到答案），比如：

*   `FSSBBIl1UgzbN7N80T=37Na97B.nWX3....`：数字 80 是 http 协议的默认端口号，对应 http 请求，其值第一位为 3，表示 3 代瑞数；
    
*   `FSSBBIl1UgzbN7N443T=4a.tr1kEXk.....`：数字 443 是 https 协议的默认端口号，对应 https 请求，其值第一位为 4，表示 4 代瑞数。
    
      
    

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrElSYNhNIdibZRxH6icUaaUHlakYvRdDg5icIBLnzE1GVjqmnoWhLFDor8w/640?wx_fmt=png)

瑞数 5 代也有以 T 和 S 结尾的两个 Cookie，但有些特殊的 5 代瑞数也有以 O 和 P 结尾的，同样的，以 O 开头的是第一次的 412 那个请求返回的，以 P 开头的是由 JS 生成的，Cookie 值第一个数字同样为瑞数的版本，和 3、4 代不同的是，5 代没有加端口号了，比如：

*   `vsKWUwn3HsfIO=57C6DwDUXS.....`：以 O 结尾，其值第一位为 5，表示 5 代瑞数；
    
*   `WvY7XhIMu0fGT=53.9fybty......`：以 T 结尾，其值第一位为 5，表示 5 代瑞数。
    

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE6oxG4oUJRniaobShIqWduTqPy6KkkPM1hqATm0clFxXjdSGxlGIAfqQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEP4ZNggAGciaNKNnvM8nPNGTVLnicmYM2jFGMMR90gA9uKoJEic3HN3jgQ/640?wx_fmt=png)

5、看入口，瑞数有个流程是在虚拟机 VM 中加载 1w+ 行的代码，加载此代码的入口，不同版本也不一样（这个入口具体在哪里？怎么定位？在后续逆向分析中再详细介绍），示例如下：

*   3 代：`_$aW = _$c6[_$l6()](_$wc, _$mo);`，`_$c6` 实际上是 `eval`，`_$l6()` 实际上是 `call`；
    
      
    

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE9Aeic9qZzjqvDpLT1J5Ia1LY405AnHrnEoM99c7dt2OSLt0KtAqhdiaw/640?wx_fmt=png)

*   4 代：`ret = _$DG.call(_$6a, _$YK);`，`_$DG` 实际上是 `eval`，有关键字 `ret`，`call` 是明文；
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEKq4v2Iv5NkzMS4ARlSysf9oImVBuPe5u8drqqAI9uaVs32Eypg38KQ/640?wx_fmt=png)

*   5 代：5 代种类比较多了，最初和 4 代的类似，比如 `ret = _$Yg.call(_$kc, _$mH);`，有关键字 ret，call 是明文，也有没有 ret 关键字的版本，比如 `_$ap = _$j5.call(_$_T, _$gp);`，也有像 3 代那样全部混淆了的，比如：`_$x8 = _$mP[_$nU[15]](_$z3, _$Ec);`，`_$mP` 实际上是 `eval`，`_$nU[15]` 实际上是 `call`，混淆的 `call` 与 3 代的区别就是 5 代是在一个数组里取值得到的；
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEDMKXPc4Vh7sf0icywPRH2ArRwhhcLVpG5YabZoOfLEwjCklnl8SHWfg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE2Bib4sOOzyBtulBpicU2Gy91mh5QiakC1gxtsnUbwuicHCWp1Cct5YcI1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEtnwGy51nAsdLOUDjoaw5W54vmdX1sGY3TXPYVgBGIicUb1j0o0bibicQg/640?wx_fmt=png)

  

当然要想精准区分不同版本，得各个条件结合起来看，最主要的还是得看看内部的实现逻辑，以及页面的代码结构，比如 4 代有一个生成假 Cookie 的步骤，而 5 代没有，有的特殊版本虽然看起来是 5 代，但是加了 jsvmp 和三目表达式，和传统的 5 代又有区别，偶尔愚人节啥的突然来个新版本，也会不一样，各版本在分析一遍之后，就很容易区分了。

Cookie 入口定位
-----------

本文案例中瑞数 4 代网站为：`aHR0cDovL3d3dy5mYW5nZGkuY29tLmNuL25ld19ob3VzZS9uZXdfaG91c2VfZGV0YWlsLmh0bWw=`

首先过掉无限 debugger（过不过其实无所谓，后面的分析其实这个基本上没影响），直接右键 `Never pause here` 永不在此处断下即可：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEJdpUgWtU7oYFEnAq87V5VaWKuhicnP7gndoaobnKqEianvogcwlbSvag/640?wx_fmt=png)

  

定位 Cookie，首选 Hook 来的最快，通过 Fiddler 等抓包工具、油猴脚本、浏览器插件等方式注入以下 Hook 代码：

```
(function() {
    // 严谨模式 检查所有错误
    'use strict';
    // document 为要hook的对象 这里是hook的cookie
 var cookieTemp = "";
    Object.defineProperty(document, 'cookie', {
  // hook set方法也就是赋值的方法 
  set: function(val) {
    // 这样就可以快速给下面这个代码行下断点
    // 从而快速定位设置cookie的代码
    console.log('Hook捕获到cookie设置->', val);
                debugger;
    cookieTemp = val;
    return val;
  },
  // hook get 方法也就是取值的方法 
  get: function()
  {
   return cookieTemp;
  }
    });
})();


```

Hook 发现会有生成两次 Cookie 的情况，断下之后往上跟栈，可以看到组装 Cookie 的代码，类似如下结构：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEdQqjSFPKxOuR7zzHzaBjOWvtw5YhXMmBKC8s6PlQyYtrVs1qEiawAXQ/640?wx_fmt=png)

  

仔细观察这两次 Cookie 生成的地方，分别往上跟栈，你就会发现两个 Cookie 分别是经过了两个不同方法得到的，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrECfOxCGpG6vKIOozv5M1HhlyGMyeeSWR4MV55dlsxmljf1wp5hicxQ9Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE0L7Sicddp3ohvFAPTNXS57wUP4xDGss2kHkjzl7tjoz9gBZFqWoguLA/640?wx_fmt=png)

  

这里的代码存在于 VM 虚拟机中，且是 IIFE 自执行代码，我们还得往前跟栈看看这些 VM 代码是从哪里加载出来的，跟栈来到首页（202 页面）带有 call 的位置：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrErbGFBqb1jkQUlQXGAnzbCAyoxzRFpsY2GbsngHlu6AdcUTprEDTBIw/640?wx_fmt=png)

我们在文章开头介绍的这个位置就是这么分析得来的，这个位置通常在分析瑞数的时候作为入口，图中 `_$te` 实际上是 eval 方法，传入的第一个参数 `_$fY` 是 Window 对象，第二个对象 `_$F8` 是我们前面看到的 VM 虚拟机中的 IIFE 自执行代码。

在知道了瑞数大致的入口之后，我们也可以使用事件监听中的 Script 断点，一直下一个断点（F8）就可以走到 202 页面，然后搜索 call 关键字就能快速定位到入口，Script 断点中的两个选项，第一个表示运行 JS 脚本的第一条语句时断下，第二个表示 JS 因为内容安全政策而被屏蔽时断下，一般选择第一个就可以了，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEn3JYHXVqIicXLa2B7yTatIhCiczyoRVSj84q1ZibRLLvGTBVibiagxic12Vw/640?wx_fmt=png)

文件结构与逻辑
-------

想要后续分析 Cookie 的生成，我们不得不要观察一下 202 页面的代码，meta 标签有个 content 内容，引用了一个类似于 `c.FxJzG50F.dfe1675.js` 的 JS 文件，接着跟一个自执行的 JS，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEJ5UOAPTjLZGr0Oic3CoHY46zKgNX9EUtqTFeRYjHpf7vRia0G7g6YdVA/640?wx_fmt=png)

  

第 1 部分 meta 标签的 content 内容，每次都是变化的，第 2 部分引用的这个外部 JS 在不同页面也有所差别，但是同一个网站同一个页面 JS 里的内容一般是固定不会变的，第 3 部分自执行代码每次变化的只是变量名，整体逻辑不变，后续我们在扣代码的时候，也会用到这里的部分方法。自执行代码里同样也是有很多 `if-else` 控制流，开头的那个数组，比如上图中的 `_$Dk` 就是用来控制后续的控制流的。

引用的 `c.FxJzG50F.dfe1675.js` 直接打开看是乱码的，而自执行 JS 的主要作用是将这 JS 乱码还原成 VM 里的 1w+ 行的正常代码，并且定义了一个全局变量 `window.$_ts` 并赋了许多值，这个变量在后续 VM 中作用非常大，meta 标签的 content 内容同样也会在 VM 里用到。

由于很多值、变量都是动态变化的，肯定不利于我们的分析，所以我们需要固定一套代码到本地，打断点、跟栈都会更加方便，随便保存一份 202 页面的代码，以及该页面对应的外链 JS 文件，如 `c.FxJzG50F.dfe1675.js` 到本地，使用浏览器自带的 overrides 重写功能、或者浏览器插件 ReRes、或者抓包工具的响应替换功能（如 Fiddler 的 AutoResponder）进行替换。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEXPsuMdcFItkF5mmHHWomdR60ZlGbSTkdKe8uxdBm3raQzouaWdEGcw/640?wx_fmt=png)

VM 里面的代码是生成 Cookie 的主要代码，包含众多的 `if-else` 控制流，无疑增加了我们分析代码的成本，这里就可以使用 AST 技术做一下反混淆，比如 Nanda 就将 `if-else` 控制流转换成了 `switch-case` 的，同一个控制流下的代码放在了同一个 `case` 下，然后在 `call` 入口那个地方，将 VM 代码做一下本地替换，具体可以参考 Nanda 的文章：[《某数 4 代逻辑分析》](https://mp.weixin.qq.com/s?__biz=MzkwMTE3NzI5NA==&mid=2247483705&idx=1&sn=fa2d58d4f504bd5a370748b42adb6838&scene=21#wechat_redirect)，感兴趣的可以试试，不了解 AST 的可以看看以前的文章[《逆向进阶，利用 AST 技术还原 JavaScript 混淆代码》](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247488349&idx=1&sn=7a485dc2d9d902042a49e1d3183a9bca&scene=21#wechat_redirect)，后续有时间 K 哥再写写 AST 还原瑞数代码的实战，本文咱们选择硬刚！

![](https://mmbiz.qpic.cn/mmbiz_jpg/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEtnHu0M1eOHficKlcuEWrJnMRY90K8RiadpsfXBibmgNjamlqb1yNxdApA/640?wx_fmt=jpeg)

VM 代码以及 $_ts 变量获取
-----------------

前面我们了解了 VM 代码和 `$_ts` 的重要性，所以我们第一步是要想办法拿到他们，至于在什么时候有用到，文章后续再说，复制外链 JS，即  `c.FxJzG50F.dfe1675.js` 的代码和 202 页面的自执行代码到文件，本地直接运行即可，需要轻度补一下环境，缺啥补啥，大致补一下 window、location、document 就行了，补的具体内容可以直接在浏览器控制台使用 `copy()` 命令复制过来，然后 VM 代码我们就可以直接 Hook eval 的方式得到，大致的补环境代码如下：

```
var eval_js = ""

window = {
    $_ts:{},
    eval:function (data) {
        eval_js = data
    }
}

location = {
    "ancestorOrigins": {},
    "href": "http://www.脱敏处理.com.cn/new_house/new_house_detail.html",
    "origin": "http://www.脱敏处理.com.cn",
    "protocol": "http:",
    "host": "www.脱敏处理.com.cn",
    "hostname": "www.脱敏处理.com.cn",
    "port": "",
    "pathname": "/new_house/new_house_detail.html",
    "search": "",
    "hash": ""
}

document = {
    "scripts": ["script", "script"]
}


```

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrExmtgxbTwAYR38KSM7OohjL5sIMHfE8sAUxHBCd1jOTvxh3yc93WbLw/640?wx_fmt=png)

  

观察 `$_ts` 的 key 和 value，和浏览器中得到的是一样的：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEiaDF5XnoH8lR0LYZtw84Ig2RWMMCnRpkXknziad2rjRwRsKicgFXic30icw/640?wx_fmt=png)

  

注意事项：`c.FxJzG50F.dfe1675.js` 外链 JS 如果你直接下载下来用编辑器打开可能会被自动编码，和原始数据有出入，导致运行报错，这里建议直接在浏览器在线访问这个文件，手动复制过来，或者在抓包软件里将响应内容复制过来，观察以下两种情况，第一种情况就可能会导致运行出错，第二种是正常的：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEaYghp0ZhmgEoB4BroaTOXQsfesDgkicIGYbzA3vB9suBcvdXCp2icKEg/640?wx_fmt=png)

  

扣代码
---

**前面说了这么多，现在终于可以进入主题了，那就是扣代码，找个好椅子，准备把屁股坐穿，此时你的键盘只有 F11 有用，不断单步调试，只需要亿点点细节，就完事儿了！**

扣代码步骤太多，不可能每一步都截图写出来，只写一下比较重要的，**如有遗漏的地方，那也没办法![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE7au9w89icNXZdmmicXsOLl8ZXW1Zww3SYQGsCRVPVdl69VTNAlTpxJUA/640?wx_fmt=png)**，首先先在我们替换的 202 页面里，自执行代码开始的地方手动加个 debugger，一进入页面就断下，方便后续的分析：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEVmLAtScRJGTfGw0cnfIbC2JyCLCHg4eDXKYRdticmTgjmAhuib5gYcYw/640?wx_fmt=png)

  

通过前面我们的分析，已经知道了入口在 call 的地方，快速搜索并下断点：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEpfd0UlO7Ifc0BicTCC8D0CBHNCI5uZDu3jyP8z3MzTWtTluyq5hJrzw/640?wx_fmt=png)

通过前面我们的分析，我们也知道了有两次生成 Cookie 的地方，快速搜索 `(5)`，搜索结果第二个即为入口：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE4E6QxQpdicIKYQrnjrSTMVKoBTZsRDRrlDQEq7a8SrxXviacKPzsJuOw/640?wx_fmt=png)

### 假 Cookie 生成逻辑

首先单步跟假 Cookie，虽然是假的，但是后续生成真 Cookie 中会用到，在跟的时候你会走到这个逻辑里面：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEfNuM5vbfL6iaqsuosh06kIMP2S6O0tTyaNcgxCafdC8prfevJbzXclQ/640?wx_fmt=png)

有一步会调用 `_$8e()` 方法，而 `_$8e = _$Q9`，`_$Q9` 又嵌套在 `_$d0` 里的，搜索一下哪里调用了 `_$d0`，发现是代码开头：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEmFm6DBeyFoO5hJIwau49Yus8SnwVVVkMpvic6VAb75C3rfhsttiaosjw/640?wx_fmt=png)

那么传入的参数 `_$Wn` 是啥呢？单步跟入，是一个方法，作用就是取 202 页面的 content 内容，那么我们在本地就直接删掉这个 `_$Wn` 方法，直接传入 content 的值即可，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEEoZS7sLy73z3IY85SrG5ZrgicquzHkxYf3FJ2TP89J0h9hH1iaJPaFDQ/640?wx_fmt=png)

  

另外，我们发现，代码有非常多的在数组里面按索引取值的情况，比如上图中的 `_$PV[68]` 的值，实际上就是字符串 content，很显然我们要把这个数组的来源找到，直接搜索 `_$PV =`，可以找到疑似定义和赋值的地方：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEibm9fiahX7tCzj8KkqXwplnLICfjn5JU4PeDBGSY3dY2bEib4Ku2UQ0DQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEygKC8Pntv5VYv2Km6hod1qwB4jH6NVwFbIb715FZotj6IO0qrLdlXA/640?wx_fmt=png)

  

所以我们得看看这个 `_$iL` 方法，传入了一个非常长的字符串，打断点进去看看，果然生成了 `_$PV`，是一个 725 位的数组：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE606uVuMHJZc4UPzpahLFDPicV04N56cb33QADRPmGEVrs99IQeKClOA/640?wx_fmt=png)

接下来在扣代码的过程中，你会经常遇到一个变量，在本文中是 `_$sX`：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE5turJTS2ibTQvY2ibcaPIQaicwjshrYa9XmXSTYkFAFvehKJzKHLr44Ng/640?wx_fmt=png)

  

有没有很熟悉？这个值就是我们前面拿到的 `$_ts` 变量，在开头就可以看到是将 `window.$_ts` 赋值给了 `_$sX`：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrERvuia7Mziapvy0lziaxicj88OmAWKMjE7ZbgTvICXSPO2nj4Aa2GRTZMKA/640?wx_fmt=png)

继续走，会走到以下逻辑中：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEo3uLXz4XlbW0rBSTQpDdIHmAPztwbibFj9IT1uoYG5mbkOSA3Wc2DIw/640?wx_fmt=png)

这里会遇到六个数组，他们都已经有值了，所以我们得找到他们是咋来的，任意搜索其中一个数组名称，会找到定义和赋值的地方：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEYDVh6ZxOk9AAlMBzbhTgCdMwIw1Ocp1JlkRnbCOFyenWMfP66OCm6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrECNuEI4UeHucbvRy84gTU7d1FWXA5yv9MMCdFysKvqhTCACiceZIwm2Q/640?wx_fmt=png)

  

赋值明显是调用了 `_$rv` 方法，再搜 `_$rv` 方法，发现是开头就调用了：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEe4tjGGykTicqd8sDUiaVSUcLJ3ibv0Qnic29MXqzsMBicicibIToibM4tx7cXg/640?wx_fmt=png)

后续没有什么特别的，一直单步，最后有个 `join('')` 操作，就生成了假 Cookie：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE7B8T4ia79P8E8096D8icibAZ9iaSpk3FVic9onXm4vGTwMYD6icJuxXFleRA/640?wx_fmt=png)

  
接下来是生成 Cookie 的名字 `FSSBBIl1UgzbN7N80T`，然后将 Cookie 赋值给 `document.cookie`，然后又向 `localStorage` 里面的 `$_ck` 赋了个值，`localStorage` 的内容可以直接复制下来，没有太大影响。  

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEXrl2Iic4KzKWfWiar9Rceia9DNqCynhJIVmVaj1K6oA52RlRIRGxo0ncQ/640?wx_fmt=png)

  

### 真 Cookie 生成逻辑

单步跟真 Cookie，在本文中也就是 `_$ZN(768, 1);`，可以看到开始进入了无穷无尽的 `if-else` 控制流：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEb0A0LeAja2iboJpRyZZBBZpD6aAuhhfwnEnZrXyMiazU5bLxukg5DXEQ/640?wx_fmt=png)

这里本地应该怎样处理呢？我的做法是以 `_$Hn` 和其值命名函数，`function _$Hn768(){}` 就表示所有走 768 号控制流的方法，继续跟，生成真 Cookie 的方法基本上在 747 号控制流，后续我们主要以 747 号控制流的各个步骤来看，747 号控制流扣出来的代码大致如下：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEl7GTG4FAfI0rvnW0uuMUtXF3Gch0xMR7ibgMZkyVZ2gL6H0rRMIpV8w/640?wx_fmt=png)

### 取假 Cookie

单步跟 747 号控制流，会有个进入第 709 号控制流的步骤，会取先前生成的假 Cookie，经过一系列操作之后返回一个数组：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEic7T4s1xVWeAgTWcLg4bm6ia96FdicyFe7ibKwAhnSnhme3FXd8NzswCXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrESMpsPNPjnHpuaCyF3bLEu8ialibPWUFGT1fAgD8aD27qQMpfb2SdoJoQ/640?wx_fmt=png)

  

至此我们在本地同步扣的代码，如果正常的话，返回的数组也应该是一样的（后续的数据就不一样了，有一些时间戳之类的参数参与运算）：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEiagIezBe5eexJ0sWQ1Cich4XAeelSRjeNteBB4M0W5GsUtqhdPzyh28g/640?wx_fmt=png)

### 自动化工具检测

继续跟 747 号控制流，会进入 268 号控制流，接着进入 154 号控制流，这里面会针对自动化工具做一些检测，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEIL23szcOTrnmhLZANA0QoiaTwMP8ZmZIHueD57JwWmk0icaRcH9gXf9Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEs2kjtT5vwaibfIRw3LsSLSDLn4TJxym8ecytmv9R1aEhPEt42NoAeQg/640?wx_fmt=png)

  

这里定义了一个变量 `_$iL`，检测不通过就是 1，后续又把这个变量赋值给了 `_$aW`，所以我们本地保持一致，也为 false 即可（其实我们不用自动化工具的话，这一段检测就不用管直接返回 false 就行）：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE07hBqQJmz0ObvmBib06JK7LaFA1Kzicz9ecJfq4UVmbKoaWdYjTqCs0A/640?wx_fmt=png)

### 20 位核心数组

继续跟 268 号控制流，会进入 668 号控制流，668 号控制流就两个操作，一是生成一个 16 位数组，二是取 `$_ts` 里面的 4 个变量，加到前面的 16 位后面，组成一个 20 位数组，这 20 位数组的最后 4 位是瑞数核心，其中的映射关系搞错了请求是通不过的，在五代中这部分的处理逻辑会更加复杂。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEWQnHamAcjpGalEyv1ibBhn349fFaDEDxxuoXIBzvtHImqxzPLUEk4CA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE9hJoJ72bXlI3LpR3X5h5chgb1UfGMhLWYebRKH5PvmXTic8pHv8w8ZQ/640?wx_fmt=png)

  

这里不是单纯的取 `$_ts` 里的键值对，你在扣代码的时候，你也许会发现怎么本地到这里取值的时候，取出来的不是数字，而是字符串呢？就像下面这种情况：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEthXzkhSR2bBZqlakHFichyZ3zFW6Cy8NlpUuwVDJiakx2PNVMJJZviaow/640?wx_fmt=png)

实际上我们最开始得到的 `$_ts` 值，是经过了二次处理的，我们以第一个 `_$sX._$Xb` 为例，直接搜索 `_$sX._$Xb`，可以发现这么一个地方：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEX23q2oKric6OYklAcHCbCKmX2O2BBq4ALWBT62npa1dLgpwRk75zs3Q/640?wx_fmt=png)

很明显这里给  `_$sX._$Xb` 重新赋值了一遍，我们可以看到等号右边，先取了一次 `_$sX._$Xb`，其值为 `_$Rm`，这和我们初始 `$_ts` 里面对应的值是一样的，然后我们就得再看看 `_$sX["_$Rm"]` 又是何方神圣，直接搜索发现是开头赋值了一个方法，通过调用这个方法来生成新的值：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEt8Sib87doMlIRvtj2QkJvzpiaP6Bd00GbtBT7iaJG8yGMPxPgggm8KQQg/640?wx_fmt=png)

另外其他三个值也是同样的套路，赋值的代码分别为：

```
_$sX._$Xb = _$sX[_$sX._$Xb](_$BH, _$DP);
_$sX._$oI = _$sX[_$sX._$oI](_$ZJ, _$DS)
_$sX._$EN = _$sX[_$sX._$EN]();
_$sX._$D9 = _$sX[_$sX._$D9](_$iL);


```

实际上应该是：

```
_$sX._$Xb = _$sX["_$Rm"](_$BH, _$DP);
_$sX._$oI = _$sX["_$Nw"](_$ZJ, _$DS)
_$sX._$EN = _$sX["_$Uh"]();
_$sX._$D9 = _$sX["_$ci"](_$iL);


```

进一步来说，实际上是：

```
_$sX._$Xb = _$1k(_$BH, _$DP);
_$sX._$oI = _$jH(_$ZJ, _$DS)
_$sX._$EN = _$9M();
_$sX._$D9 = _$oL(_$iL);


```

静态分析没问题，我们可以先固定下来，但是实际应用当中这些值都是动态的，那我们应该怎么处理呢？先来多看几个对比一下找找规律：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEecRC9Mic72oUJNYrmd7mLWbYcBEqmOzlEV3NciaKJibHhz3ID7NCyibaGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE7LgxhWIm0ANyYoEfjPyLVAZoBTt9RLutyXfsjQXvtAhbTmiaJ9uPl3Q/640?wx_fmt=png)

  

可以发现每次对应的位次都不一样，但是实际上相同位置的方法点进去都是一样的，也就是说，变的只有方法名和变量名，实现的逻辑是不变的，所以我们只要知道了这四个值分别对应的位置，就能够拿到正确的值，在本地，我们就可以这样做：

1、先利用正则匹配出这四个值，如：`[_$sX._$Xb, _$sX._$oI, _$sX._$EN, _$sX._$D9]`；

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEiaV2F4JYy9cpOWzq59cv1DYXVDBDSoTGExR5RxkIicwKNqIicRuGfiavfQ/640?wx_fmt=png)

  

2、再匹配出 VM 代码开头的 20 个赋值的语句，如：`_$sX._$RH = _$wI; _$sX._$i5 = _$n5;` 等；

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEBFFKvW8vRZcGq7kQjamjqUOpWRu4DZ9QeImoPmf0XY0wUMgvBRD3ag/640?wx_fmt=png)

  

3、然后通过 `$_ts` 取这四个值对应的值，相当于：`_$sX._$Xb = _$ts._$Xb = _$Rm`；然后再找这四个值所定义的方法在 20 个赋值语句中的位置，相当于：查找 `_$sX._$Rm = _$1k;` 在 20 个赋值语句中的位置为 7（索引从 0 开始）

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEjGazgjdaO9ROfefibickY5f8zgCeEKqd1nTOSJNyBqics149wNRrHwuicw/640?wx_fmt=png)

4、我们知道了这四个方法在 20 个赋值语句中的位置，那么我们直接匹配本地对应位置的名称，进行动态替换即可，当然前提是咱们本地已经扣了一套代码出来了：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEib4icXOLibcOh6Ric7AXUPkBxMnQeolWWIO0dG8VB8lhzxLMeJuCgaWFGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEcr208C2icZ86FBqKpwjD4p8L7ib0w2J46iaD9H8PictRpNLBRFOryanP0A/640?wx_fmt=png)

  

经过这样处理后，就能够保证这四个值的准确性了。

### 其他用到 $_ts 值的地方

除了上面说的 20 位数组里用到了 4 个 `$_ts` 的值以外，还有其他地方有 7 个值也用到了，直接搜索就能定位，这 7 个值相对较简单，每次都是固定取 `$_ts` 里面的第 2、3、4、15、16、17、19 位的值，同样的，找到对应位置，进行动态替换即可：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE7VZoTic8madrbwuPkx3iazy2ibL7qsp8CgeMbGD0m3kpzOmNxuibljYicnA/640?wx_fmt=png)

### 注意事项

特别注意 VM 代码开头，会直接调用执行一些方法，某些变量的值就是通过这些方法生成的，当你一步一步跟的时候发现某些参数不对，或者没有，那么就得注意开头这些方法了，可能一开始就已经生成了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE9yWUA5AQWcQP32XeraibzSoBXBLia6vuf9n1TEbILqzBTia5eibmqxQx8A/640?wx_fmt=png)

### 后缀 MmEwMD 生成逻辑

后续的其他 XHR 请求中，都带有一个后缀，这个后缀的值同样是由 JS 生成的，每次都会变化，当然不同网站，后缀名不一定都是一样的，本例中是 `MmEwMD`，先下一个 XHR 断点，当 XHR 请求中包含了 `MmEwMD=` 时就断下，然后刷新网页：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrElOtyUZWKNdt8vIKO3UyJLiaFOa0gI8q4FYwAML6guxibFweUy63sJaOg/640?wx_fmt=png)

可以看到后传入 `l.open()` 的 URL 还是正常的，断下后到 `l.send()` 就带有后缀了，再看 `l.open()` 其实就是 `xhr.open()`，明显和正常的有区别，同样这个方法也在 VM 代码里，应该是重写了方法，可以和正常的做对比：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE1eGjOs85TdQ2yeQNBs2rXImrFCFgU3y2MfmEnee8Kr8fj6B0VCIgrQ/640?wx_fmt=png)

跟到 VM 代码里去看看，经过了 `_$sd(arguments[1])` 方法就变成了带有后缀的完整链接了：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrE1dbFeWrJdCu7KMk4pj7Hzz4XgibrzpGFpBFwWnEy795nLfZKRjB2Ikg/640?wx_fmt=png)

跟进 `_$sd` 方法，前面都是对 url 做一些处理，后面有个进入第 779 号控制流的流程，实际上就是原来我们生成 Cookie 的步骤，跟一下就行了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEPfhqncelYjYFWfQFthrneIxf4dWcDe9aPOB70U4hiaRzsukNE3rtj5w/640?wx_fmt=png)

善用 Watch 跟踪功能
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEIjtQvD2uNWGp5QF5j2pmXeibfJrvz1LlgSFcIr6QOJtJJ1LRqePaxOg/640?wx_fmt=png)

  

开发者工具的 Watch 功能能够持续跟踪某个变量的值，对于这种控制流很多的情况，设置相应的变量跟踪，能够让你知道你现在处于哪个控制流中，以及生成的数组的变化，不至于跟着跟着不知道到哪一步了。

结果验证
----

如果整个流程没问题，代码也扣得正确，携带正确的 Cookie 和正确的后缀，就能成功访问：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEib5GPz7Ogp294ADdAmKFknH57CvDyfuGalGxmZPygWd0zwMG1JoZQsA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Ku3lLibSrsEiaicSN7AwSTjrEmeuicicM7CBf41bxprL7PVazpctCk4Q5nRKHFjiceCB5Aib3cLdMolcFyw/640?wx_fmt=png)

END
---

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4L2BEhohiciaycd6acftNGOyxLZkbZal8LW2qPk5MXg11hB0qia0s0mdecYM7sLzaezgIzkkiaiahIRY5g/640?wx_fmt=png)

 ![](http://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4Jnu94oD2zQpYMGwz40zvhO68uyYSicKNwvPsbQDZ1AcI110aboibfwUhA9OvOrsKGA4GneeSRqyFFA/0?wx_fmt=png) ** K 哥爬虫 ** 分享有深度、有细节的爬虫技术。 52 篇原创内容  公众号

  

![](https://mmbiz.qpic.cn/mmbiz_png/7VAgNKQgCMFM8ia5BA9MLZhlCnRr8Er4gR4Rjr7WBmby6jKvlqpH7jZITFBYBIYbibfOgHRCF5obiaJn6yzC321qw/640?wx_fmt=png)

点个在看你最好看

![](https://mmbiz.qpic.cn/mmbiz_png/NtgFk2rGpiaOPxvr7Ls916UDdGAibFN8ObxF6VKc8qCT18luCwKTUgHicBiaMYJE9SIdicQHL7ouCt8xk7tMtsxKayA/640?wx_fmt=png)