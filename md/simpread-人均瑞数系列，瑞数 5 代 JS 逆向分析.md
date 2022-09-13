> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Li88GVe49Q4DBbq3ti3YlA)

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

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx56FRty9oyzTsGDia7App22eh9fYueQuOEXHNfNzibfkO6K6DIDrRmY32Q/640?wx_fmt=png)

瑞数动态安全 Botgate（机器人防火墙）以 “动态安全” 技术为核心，通过动态封装、动态验证、动态混淆、动态令牌等技术对服务器网页底层代码持续动态变换，增加服务器行为的“不可预测性”，实现了从用户端到服务器端的全方位“主动防护”，为各类 Web、HTML5 提供强大的安全保护。

在 K 哥往期的文章[《人均瑞数系列，瑞数 4 代 JS 逆向分析》](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247488564&idx=1&sn=0d0faad2a8a6354e10b5b112a37af3d1&scene=21#wechat_redirect)中，详细介绍了瑞数的特征、如何区分不同版本、瑞数的代码结构以及各自的作用，本文就不再赘述了，不了解的同志可以先去看看之前的文章。

Cookie 入口定位
-----------

本文案例中瑞数 5 代网站为：`aHR0cHM6Ly93d3cubm1wYS5nb3YuY24vZGF0YXNlYXJjaC9ob21lLWluZGV4Lmh0bWw=`

定位 Cookie，首选 Hook 来的最快，通过 Fiddler 插件、油猴脚本、浏览器插件等方式注入以下 Hook 代码：

```
(function() {
    var cookieTemp = "";
    Object.defineProperty(document, 'cookie', {
        set: function(val) {
            console.log('Hook捕获到cookie设置->', val);
            debugger;
            cookieTemp = val;
            return val;
        },
        get: function() {
            return cookieTemp;
        }
    });
})();


```

断下之后往上跟栈，可以看到组装 Cookie 后赋值给 `document.cookie` 的代码，类似如下结构：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5kv7A3k3ew48fzj4KibxNu3U2z3SqgzukYUHlZHIicAZj3XW27ticDRyXA/640?wx_fmt=png)

继续往上跟栈，和 4 代瑞数类似，`(772, 1)` 的位置是入口，4 代有一次生成假 cookie 的过程，5 代就没有了，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx50pAjn4GUVLIHzOk3r2dtazyRNCha6cibhtmHKHKiaUJs8lqAvtJicxyyQ/640?wx_fmt=png)

再往前跟栈，来到首页代码，这里就是我们熟悉的 call 位置了，图中 `_$ug` 实际上是 eval 方法，传入的第一个参数 `_$Cs` 是 Window 对象，第二个对象 `_$Dm` 是我们前面看到的 VM 虚拟机中的 IIFE 自执行代码。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx54dClzvu8pnzVmnibjkFHpztBXEsRHkBcVAOMDpcLwTia5h7ayZgjWGSQ/640?wx_fmt=png)

VM 代码以及 $_ts 变量获取
-----------------

获取 VM 代码和 `$_ts` 变量是第一步，和 4 代类似，复制外链 JS（例如 `fjtvkgf7LVI2.a670748.js`）的代码和 412 页面的自执行代码到文件，本地直接运行即可，需要轻度补一下环境，缺啥补啥，大致补一下 window、location、document 就行了，补的具体内容可以直接在浏览器控制台使用 `copy()` 命令复制过来，然后 VM 代码我们就可以直接 Hook eval 的方式得到，这里 `$_ts` 变量的获取和 4 代有点儿区别，4 代我们的做法是运行完代码后直接取 `window.$_ts` 就行了，5 代运行完代码后会有一个清空 `$_ts` 的操作，可以自己跟栈看一下逻辑，要么把清空的逻辑删了，要么定义一个全局变量，然后直接在 call 的地方将 `$_ts` 的值导出来：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5kledemaQKa6bGiapiav6TfSVWVgiamGTeeU8iaU5orDlkVRyFueNcAGIRw/640?wx_fmt=png)

大致的补环境代码如下：

```
var eval_js = ""
var rs_ts = ""

window = {
    $_ts: {},
    eval: function (data) {
        eval_js = data
    }
}

location = {
    "ancestorOrigins": {},
    "href": "https://脱敏处理/datasearch/home-index.html",
    "origin": "https://脱敏处理",
    "protocol": "https:",
    "host": "www.脱敏处理.cn",
    "hostname": "www.脱敏处理.cn",
    "port": "",
    "pathname": "/datasearch/home-index.html",
    "search": "",
    "hash": ""
}

document = {
    "scripts": ["script", "script"]
}


```

获取 VM 代码以及 `$_ts` 变量：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5OjiaEqoyxcDTNhZDyATicINppXSJaPZKsEZsLplmYDDmM6JguhGe7iabQ/640?wx_fmt=png)

善用 Watch 跟踪功能
-------------

在跟栈分析之前，有必要了解一下浏览器开发者工具的 Watch 功能，它能够持续跟踪某个变量的值，对于瑞数这种控制流很多的情况，设置相应的变量跟踪，能够让你知道你现在处于哪个控制流中，以及生成的数组的变化，不至于跟着跟着不知道到哪一步了。如下图所示，`_$S8` 表示目前正处于第 279 号大控制流，`_$5x` 表示大控制流下的哪个分支，`_$mz` 表示 128 位大数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5B04GECLsUjCwXuncq1ricAHQHZ7BYx9jkV6SNsMickl0Nz64AbR6FG9A/640?wx_fmt=png)

跟栈分析
----

老样子，本地替换一套 412 页面的代码，固定下来，然后开始跟栈分析。直接从 `(772, 1)` 开始跟（文中说的第多少号控制流、第几步均为作者自己的叫法，第多少步并不代表实际上的步骤，仅表示关键步骤）：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5HjabFDZIjXlgwT94SbJpE21RxSATzUft6JUy0yrNolKNh7J2fia4Ulw/640?wx_fmt=png)

单步进来，`_$qh` 是传进来的参数 1，即将进入 742 号控制流：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Z2FDnEagqZ8lbNNGibJ380niawTRj9YOiaJAibmp1RgCNZgzUMBfjBzbzw/640?wx_fmt=png)

进入 742 号控制流，第 1 步通过一个方法获取了一个时间戳，进入这个方法内部，对时间戳进行了差值计算，会发现有两个变量 `_$tb` 和 `_$t1` 已经生成了值：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5eQ0An8TibRvGL6FgZibib3t3HKBvicW78hfGibVIA4koHnYKuLAREmDWUOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5lpiaib8KRSz04WjhR5BznicudJO7vlFL1YykQ78wLMkcSUpqvbLyBhLVg/640?wx_fmt=png)

这两个值也是时间戳，怎么来的？直接搜索这两个变量，搜索结果有几个全部打上断点，刷新断下后往前跟栈，会发现是最开始走了一遍 703 号控制流：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5IC7o6JC99Cb2nnZE67G2t00EjdY5lmHyp3ib0cnOIDoo2X3RwhxNFTw/640?wx_fmt=png)

先单步跟一遍 703 号控制流，703 号控制流第 1 步是进入 699 号控制流，返回一个数组，没有特别的，直接扣代码即可：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5rApTd0JLd75UQ9s6clOMorbRP9mfQmGCsyd6gbeWOicqvsWzguCd1pA/640?wx_fmt=png)

703 号控制流第 2、3 步分别取数组的值：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5uHW2ZQfAgYydlKZ8t0AuebJ2xQekSvlQuVV6c3ODnllRN1yvQSmVZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Telua6r2qcaY1sEmXVBdH1sqbJEIOwtw0QPB1XJflibOlhwvbj9akZA/640?wx_fmt=png)

703 号控制流第 4、5、6 步生成两个时间戳并赋值给前面提到的 `_$tb`、`_$t1` 变量，涉及到的方法也没有什么特别的，缺啥搜啥补啥即可：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5RENqauPib9syNkRnmXibUheYEFMkGx0trH6bZFo8ZFgw7YOEnibqOmdsg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5VVFIUlCazcWOZDBwfBt1zTZv7bFUvarxuAC8RLB6jHrTEkrG0tTBhw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5C4Iynx8rBtVibFNtg2uBDoXGQXDgyRuw2QjfUTianaWaOQ2EdeOzLGzg/640?wx_fmt=png)

703 号控制流第 7 步，这里修改了 `$_ts` 的某个值（VM 代码中，`$_ts` 被赋值给了另一个变量，下图中是 `_$iw`），`_$iw._$uq` 原本的值是 `_$ou`，修改后的值是 181，这个值也是后面关键 4 位数组中的其中一个，具体逻辑后面再讲。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5KT8LbpGQFqn3J3Ct4JvN3qpHoIcW2FwXKwrdwLjZpYagQkEkqBPFrg/640?wx_fmt=png)

703 号控制流结束，我们继续前面的  742 号控制流，742 号控制流第 2 步，将前面生成的时间戳赋值给另一个变量。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5WWurfe4IULO5myEnjfA96Xicdia56ibTPAafOX18wbQtP8tI6Tib2rMSDA/640?wx_fmt=png)

742 号控制流第 3 步，进入 279 号控制流，279 号控制流是生成 128 位数组的关键。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5RLhqsdj8AHjIYJibgorgefcVLDmYeJery9HVxBfnPGgdQEU8CE5ufAQ/640?wx_fmt=png)

进入 279 号控制流，第 1 步定义了一个变量：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5vnCxibqC0dICiadoNTa4WIFndKkoFTHibbA52ibxcHCJOrAv8R6we8Ufug/640?wx_fmt=png)

279 号控制流，第 2 步，进入 157 号控制流，157 号控制流主要是做自动化检测

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5DStxmq3YIXuIYdFLAfxhZwnH729D7u0QdaUuib31qIfP2QONZVH6qSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5b7VkPTFu4BVyehksyBlUTT8UzwGTUibk9P60SntNyOSmMdoDuu5FzOA/640?wx_fmt=png)

279 号控制流，第 3、4、5 步，做了一些运算，一些全局变量的值会改变，后续的数组里会用到。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Bzh4ZleFxYlEAVmWicDaYZxYCzopB0cEIQH5vTyuX8zwNE8IbDoIwJw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ObsPYx3zCmzYibOXMGfDlyicjvGibZoJwm7PkTlrhUBKKNaNo4kQGKj7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5dZ2AwJrhkXryuzhRoUMspQrhq9TMKoRjljSicolNlvHBgbSCrrYzeZQ/640?wx_fmt=png)

279 号控制流，第 6 步，初始化了一个 128 位的空数组，后续的操作都是为了往这个数组里面填充值。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5TXLxcuKgmPftibL5JmneP0FSicu5gDxI10XLAFEMp5JZIzBtSSXISiaKg/640?wx_fmt=png)

279 号控制流，第 7 步，进入 695 号控制流，生成一个 20 位的数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5sydnG7pYib4NZGsLmnhAib1Sb1SyZzZYBAZ12QsCsUic7ymiatURGPcCAw/640?wx_fmt=png)

进入 695 号控制流看一下，第 1 步，取 `$_ts` 的一个值，生成 16 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ZY4xBibRxO1LbQicqN3JYIMmoWW9UyJwIva35ulcRKibWWvXh6PNyReng/640?wx_fmt=png)

695 号控制流，第 2 步，取 `$_ts` 里的四个值，与前面的 16 位数组一起组成 20 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5eDXSeZFx7CqMdVNB2eOhQG0feglezg2tY52Zj0HmfwoZIlXQAQ9TFQ/640?wx_fmt=png)

这里注意这四个值怎么来的，以第二个值 `_$iw._$KI` 为例，搜索发现有一条语句 `_$iw._$KI = _$iw[_$iw._$KI](_$bl, _$n2);`，首先等号右边取 `_$iw._$KI` 的值为 `_$Mo`，然后 `_$iw["_$Mo"]` 实际上就是 `_$iw._$Mo`，前面的定义 `_$iw._$Mo = _$1D`，`_$1D` 是个方法，所以原语句相当于 `_$iw._$KI = _$1D(_$bl, _$n2)`，其他三个值的来源也是类似的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5uVHhsrVwjN0icficPIicaDGL77SRgAGJd9AkFicgKribc9fzIzDRDZmKMew/640?wx_fmt=png)

695 号控制流结束，回到 279 号控制流，第 8 步，将前面的时间戳转换成了一个 8 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5BibauzR5Jx81QibXCEGE0mZQzhFKC9hTOGIObgibKADbsC0Jxn4j1zmcQ/640?wx_fmt=png)

279 号控制流，第 9 步，往 128 位数组里面添加了一个值。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ZGicrzfbyWlCEqMrIicqn0AFrDib5icyUwbial04DjTaf174NxH9LJqR8yg/640?wx_fmt=png)

`_$ae` 这个值怎么来的？搜索下断点并跟栈，发现是开头走了第 178 号控制流得来的，跟着走一遍即可。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5f8myDxto6QBMrSTUXCv0eVrAlE3DlLZIiaOBEH6OvVUa4VmsE3H0MDQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5QYnzAAswyT7aMQWJ8iazw9qdaic4icG4Zr6t2UOC7DNFicZ28bo8xCEZjw/640?wx_fmt=png)

279 号控制流，第 10 步，又往 128 位数组里面添加了一个值，这个值是开始 279 号控制流传过来的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5JPEiaD0rLjQZ6iaHmp4ynoKwicq7Z831QuNWVJzxtRqXPYViaW5wSMNib2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5falqbmRGzJFCQ6Bn33nIbSyuWom99YencSMIC9dW6LaZ6TZgQVMlIg/640?wx_fmt=png)

279 号控制流，第 11、12、13、14 步，时间戳相关计算，然后生成两个 2 位数组。注意这里面的两个变量，`_$ll` 和 `_$ed`，在刷新 cookie、生成后缀的时候可能是有值的，仅访问主页没有值不影响。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5lPDiagUPXUfsGcIkh5wjVVcz2Cqkyc3QjZS72cMPRzLDUCf7UAgtKkQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx565xibfM04Rg3icql7oJC3y5iakI6CibON9Ypy9MAPicicbDqCSETAJHPCCPg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx51hcqMzpPj3m1732twEYr4mLO2piaRrzCtCPXY4AOq2R6dtOv3YLa1rQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5vib1Uvjgk2W6dWMhWjHCibtiavia1kOQ0pdu4gZVkWBsbSH9nBremiciaia7Q/640?wx_fmt=png)

279 号控制流，第 15 步，往 128 位数组里面添加了一个 4 位数组 `_$bl`，搜索也可以找到是通过 723 号控制流得来的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5FGS5WcU1gNsCYwybib4YtZUe2kNbjvAPhPTsH2XfRvns3cJEQDw9SNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5RTUWaRPicOFMASciaCianvVpVpgdZGTSnTbL2EVcxH0LAQQHgPOI5e8rQ/640?wx_fmt=png)

这里的 723 号控制流，实际上是取了 `$_ts` 某个值进行运算，生成 16 位数组，然后截取前 4 位数组返回的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5RLfEqM4RXIGRJuDjiazSISruB8fUxWlKgicyJHociaWNxk1OvpqTmxsGQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5AFmW9ia3B5AlQWOADia3Yp49BuibPlE7vXlfn5vl4LELZykgh5oribvOpQ/640?wx_fmt=png)

279 号控制流，第 16 步，往 128 位数组里面添加了一个 8 位数组 `_$Yb`。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5uud0mznUsayVg7gqXhqZGVhH2E5u3tRW9g3y841rTDPE98GjSOC6lA/640?wx_fmt=png)

8 位数组 `_$Yb` 同样搜索打断点，可以在一个赋值语句断下：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5A6kic8PU2NqibwcO7xgTpESoCRSZb1p9Rpw2VBKPsFB8T6fiaeu928zpQ/640?wx_fmt=png)

可以看到 `_$EJ` 的值就是 `_$Yb`，往前跟栈，会发现先后经过了 657 号、10 号、777 号控制流，其中 777 号控制流是入口：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5RIoibcf3icDhXhHa1aocJnR92vC7opc8NNraPAY5bYSKuGELibQNGuvcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5F8aOFK0S7CJOVHHgI7gWYwtwPLTuhVOfCM6xTsQMicYA97jACXhsGBQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5h9FfOl2CpOMicSBRLLjWK73dnI5oJ91bcjw7umysXDVYlFiaDmiaezPTg/640?wx_fmt=png)

如果单步跟 777 号控制流，你会发现步骤较多，中间有些语句不好处理，且容易跟丢，所以我们这里就直接关注 657 号控制流就行了，777 号控制流直接到 10 号控制流，再到 657 号控制流，中间的一些过程暂时不管，跟到缺什么的时候再说（后续有很多取值赋值等操作都是在 777 号控制流里实现的，可以注意一下），这段逻辑在本地表现的代码如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5w54ia1qB4elztKvgLtlqRjia0zHSia2Q6GicBLFR7J3evbXZNO8pLgbNpg/640?wx_fmt=png)

这里直接单步跟一下 657 号控制流，第 1、2 步 new 了一个方法。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5AgclQ7uUTcvVAjfyicEHlYOWAUKYb9R2lcMiahkYHpg2bHLaRWcDaf8A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5EX6ofsg6v6ENqusJRFL6QPSKQrc87vLJmVNgBgkbbRPRKU9OSSQ5UA/640?wx_fmt=png)

这里就要注意了，容易跟丢，先进入 `_$bH` 方法打上断点，然后下一个断点就走到里面了，接着在单步调试，会进到另一个小的控制流里面，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5b6NUmAMcbs6iantYU9ib4gSoSzbnJ6hZryp7XgzjgyvpVicRnXnGicIoIw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5gghNlHaJkcb0XyNgoLb7Trs5QRlnF6FwkXdVaK6hbDxiazddjpqfFGA/640?wx_fmt=png)

开始单步跟第 96 号小控制流，第 1 步定义了一个变量。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5d9oLH1IsFkktGf4buNIibkA5SmwGibYaoRNcwibFaoAN69w5PaSOYhdiag/640?wx_fmt=png)

96 号小控制流，第 2 步将 `_$PI` 的值赋值给了 `_$fT`，而 `_$PI` 的值其实是 `window.localStorage.$_YWTU`，`window.localStorage` 里面有很多值，这个东西我们文章最后再讲，其中一些值与浏览器指纹相关，这里先知道他是取值就行了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Jz48fIb2d0jWqq97XlFGTgZPRSWVpeklwtMkpAAGFDtldkew9lCeag/640?wx_fmt=png)

96 号小控制流，第 3 步，进入第 94 号小控制流，最终生成的是一个 8 位数组，这个其实就是前面我们想要的 `_$Yb` 的值了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx55adzViaSqrLm4KYc5ClZJpVhrYWEdIJ1LKWthoz71v98Ua3wDtvFSQQ/640?wx_fmt=png)

后面没有什么特别的，中间几步我就省略了，照着扣代码就行了，然后 96 号小控制流，第 4 步，就将 `_$EJ` 的值赋值给 `_$Yb` 了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5TgicpUNyF37ABoOiadhT7w8a6hPVKzqyKTWB76Lzsy6cLP6r8lKJQobA/640?wx_fmt=png)

到这里先别急着结束，后面还有关键的几步，96 号小控制流，第 5 步，又遇到了和前面类似的写法。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5hib1Cgdqc2cOEw4tRjbst8sqnHcKbHuguAOmYzLpibSqoRhM7oOzbDyQ/640?wx_fmt=png)

同样的，先进 `_$pu` 打断点，再单步跟。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ujQNxJdf6omzntGm3h6ko2QxG6coaU8WEIo1DFU65ZqJ2849oqpO0w/640?wx_fmt=png)

来到另一个小控制流，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5pkpZ685R6vfjv3sr4pd9bUeformtTf0PpHdcnXicFRCGQFkeojzQdbw/640?wx_fmt=png)

10 号小控制流第 1 步，取 `window.localStorage.$_cDro` 的值，转为 int 类型，赋值给 `_$5s`，这个 `_$5s` 后续也会加到 128 位大数组里面。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5WbgYSxicYticvVnAAPNFy3lxtv9S8Sibic7wopicR33T08Ia6dBNvamNtpg/640?wx_fmt=png)

10 号小控制流后续还有几步，没啥用可以省略，最后一步返回 96 号小控制流。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5gYqOhtKsOvxFu70X38FIUJOxIBtMcUfOQdblUicJPNickvEuxXZ0Dqyg/640?wx_fmt=png)

然后 96 号小控制流后续也没啥了，返回 657 号控制流。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5a0yQxDEaBfexLVC6ZYnIfmibJDia9b4whia6yRZGm6s0DPwba3dn2zsrg/640?wx_fmt=png)

此时我们已经拿到  `_$Yb` 了，777 号控制流就先不管了，后续还有些代码先不管不用扣，等用到的时候再说，返回 279 号控制流，接着前面的步骤，来到第 17 步，变量 `_$5s` 经过 264 号控制流后，生成了一个值并添加到 128 位大数组里面，而 `_$5s` 的值正是前面我们跟 `_$Yb` 时，通过 777 号控制流拿到的，实际上也就是取 `window.localStorage.$_cDro` 的值，转为了 int 类型。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5mwOCQ1oHeaBnSShJaibsIoJwJcMviaJ8MEwvniaSOKdf458VX7JfbaMOA/640?wx_fmt=png)

279 号控制流，第 18、19、20 步，往 128 位数组里面添加了两个定值、一个 8 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ibXlT2icXyehnaOPcsITkD2ANACjoQ3WFOQ3y6uHGEyZuB4CMDSAt8jg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ykNZicZOojLZRB6xiaT6eefOuyv3icAicdn5fc5dw045jjpQ4bjooEXgpA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5VbkO32O380zTJFHgIViaOOF57fdn8XCsgsISUgnW2OqKkAicYd9Hpmpw/640?wx_fmt=png)

279 号控制流，第 21 步，往 128 位数组里面添加了一个 `undefined` 占位，后续会有操作将其填充值。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5kTp7UbuKv6M2YHIhlwxpBFTUj7bM3s01h8lDs4RmCCA4Ku6puhPkNA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5s6RiaIwlxhR4AqBy31WibwM4JwmZ0B94xXDpT37W4UsonQpqL2oukiaNw/640?wx_fmt=png)

279 号控制流，第 22 步，进入 58 号控制流，58 号控制流与 `window.localStorage.$_fb` 的值有关，如果有这个值，就会生成 20 位数组，如果没有就是 undefined。58 号控制流就只有一步，返回一个变量，本文中是 `_$0g`。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5icZzTNUoshmKDpMaW0fVhQ4kuHIZlNRwsniaXaa9oahwS2GdL8HiaXasA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5MlPbzpsxC8YNRmCLmJ1UAdNf0uECTtvhI74Myu2h109TxMy2gll4Fw/640?wx_fmt=png)

这个 `_$0g` 是咋来的呢？同样的直接搜索，下断点，发现是通过 112 号控制流得来的，往前跟栈，同样是先经过了 777 号控制流，和之前的情况类似，中间的过程就不看了，直接看这个 112 号控制流。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ory16XjGsl7tOb7BAsMnWGuXPS8FaylwS6icAwc2DgHSic316GmWjFzQ/640?wx_fmt=png)

本文中，112 号控制流传的参是 `_$bd[279]` 即 `$_fb`，112 号控制流第 1 步，进入 247 号控制流。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx524IYkB2pib8zQIAsibcliaLuKQOld6aRGHgAXE7rSyPIsHEshc2WZ1hVw/640?wx_fmt=png)

247 号控制流就 3 步，先将 `window.localStorage` 赋值给一个变量，然后取其中 `$_fb` 的值再返回。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5yfNykLfTIG5YzO8XOWRqicjlAib8Kic4xq4IXWiazDFEliaYcQwrrhq49GQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5gz1aQibOBV9xYATrfXJLMiaryo15D0S5kWibIlgF6A7wD2a1JSx3PpNXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5UlsiclnpcRPicKFYKwtguE7RzxLqic7Qh6x81BnTHzzV4Q3ZuGNfogFfQ/640?wx_fmt=png)

112 号控制流第 2、3 步，一个 `try-catch` 语句，取 `window.localStorage.$_fb` 计算得到 25 位数组，然后取前 20 位并返回，这就是前面我们需要的 `_$0g` 的值了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx52mELoIHowx9fWt0oOHpg5o8SVF2zL47sMAuRtxx1OxPviaN469ibq20A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5V5lSsiaAV3oJK94kX4PXwbiaDK28KamA0VDbvwL2LDUXnVHb6aGdrRJw/640?wx_fmt=png)

279 号控制流，第 23 步，将前面 `window.localStorage.$_fb` 计算得到的 20 位数组添加到 128 位大数组里面，注意这一步如果没有 `window.localStorage.$_fb` 值的话，是不会添加的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5BI5JojHOSq9YNa0PdNxjnw5R9cGlMb9rqsfHKS9wt9y1BMibeaxViaxA/640?wx_fmt=png)

279 号控制流，第 24 步，对一个变量进行位运算，然后取 `window.localStorage.$_f0` 进行运算，如果 `$_f0` 为空的话是不会往 128 位大数组里添加值的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5icjDVuic01hWUTqZ275RRLcv0saf0F80xuhDOKRGe3mPiaYPsKgfmrPDw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Qlv1ZTYVnK47ordhf7rFWo1OoqUCw3uFYTFicFOWZLo3XRAV3mFz0ww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5dKRZ7Txy8SDdWTWmlbWtsGIwJ35V5uACTWmHqLuhm0egQKfyLicpLqg/640?wx_fmt=png)

279 号控制流，第 25 步，对一个变量进行位运算，然后取 `window.localStorage.$_fh0` 进行运算，如果 `$_fh0` 为空的话是不会往 128 位大数组里添加值的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5dk9iaNxhst86GKqdLKGBWFFDhSbib09VOYHxtA1UvEI47eibqxeoibu21A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5v3EunEoicnNLcOUQdzkib8MQdDWsNYYuRQo2NBCicrjc3lZiaey7IOsO7g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx57ia0slVo6ics2IncrHicXYoeD2FnicTpGBWC2v6zjGL7bDXNgX3qPh1iafA/640?wx_fmt=png)

279 号控制流，第 26 步，对一个变量进行位运算，然后取 `window.localStorage.$_f1` 进行运算，如果 `$_f1` 为空的话是不会往 128 位大数组里添加值的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx55EPiaLBJzwP7jibicYphlypcJOUjpYIx9QMj26yTibZeciaZOhBISyqH6EA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5qRibcZXiae9Y4GXpbPXe7MHRoEicymzIItzyEVQPdoHd6XoU8Hu5vMkRA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5trmWib6kkDy9jmuVygn7iaNNLM8HZQV5MlGmicbic8bEviczjBuiajl4W3jQ/640?wx_fmt=png)

279 号控制流，第 27 步，进入 611 号控制流，611 号控制流主要是检测 `window.navigator.connection.type`，即 `NetworkInformation` 网络相关信息，里面判断了 `type` 是不是 `bluetooth`、`cellular`、`ethernet`、`wifi`、`wimax`，正常的话应该返回 0。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5uFl0gGAhG3zgLhYbGXVOVMaa5ufiaHsDJLISwSEhMDvm1Rk8grr6jCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5sbmpeCUvRgQZrWjJ0WKDfuMk9eNWxicruxwHZkic8JE0QkGJuWBric8OA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5j4hhSdQCCgvhKBY2CmU2WH7OyBYuZuvLqiaAZ8WvXLsJ4xE4xtAyGibA/640?wx_fmt=png)

279 号控制流，接下来几步都是类似的，这里就直接统称第 28 步了，首先对一个变量进行位运算，然后分别取 `window.localStorage.$_fr`、 `window.localStorage.$_fpn1` 、 `window.localStorage.$_vvCI`、 `window.localStorage.$_JQnh` 进行运算，同样如果这些变量为空的话，也是不会往 128 位大数组里添加值的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5aVPDCc7U8icUl05FPEFZ2dqCOhGG4DOkf5SDCQ3hLxodrocPln5wnzw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5mkMiaIo4bGhtxp1v1LIOjLEFOIhRP10x8FxDaGnviccWId8gOx83rDpg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5EoAB6vc6ZIUCMkvp8P8sThtSm0Lk3swgqic5CWy5Iwib3mkgrA4x9WlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5SMAe0GdaQicNnIuAJP8jic0VNX8sulib451rX8rcYdAtC5LzEU67TneibA/640?wx_fmt=png)

279 号控制流，第 29 步，往 128 位大数组里添加了一个定值 4，本文中该变量名是 `_$kW`。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Pw3FDlaM8GkHaWPYA51X7nlVCbrs3zZib0gzyxSBCD7bnJS64rcRQPw/640?wx_fmt=png)

`_$kW` 这个变量是咋来的，和前面的套路类似，直接搜索下断，同样是经过开头的 777 号控制流得来的，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5UM81IQVq7QK79QP8nUNHxx6dRiaPo47vkBpV5xIfvicickD8seuvicvRibg/640?wx_fmt=png)

继续 279 号控制流，中间有一些变量位运算之类的就省略了，第 30、31 步，取了一个 `https:443` 的长度进行计算，先后往 128 位大数组里添加了一个定值和一个 9 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Aok6qm6IGHbEcmNfue3iaFRDwu1yaSNJhVAV2eb5yncPNNV7P0VNLZg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5rdIbNE14ZcNHa6TjTX5kfbRKsZfiakZvGffAZ2SOCqrVrY9L8rBl9YQ/640?wx_fmt=png)

279 号控制流，接下来几步都是在取值，都差不多，就统称为第 32 步了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5pzpo9S0RDRyVCXcovveLs4uwQViaiaGkeomH6fJnlA7vZbGVBTInlRLw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5pYVLicoemdNlOP4ElckKKPLmW9oGgHVg0oRuBOIOVepC0Fcf9mWZxvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5iaWWicuZXWmDN4u0yGiafD0NsHD5Rvh2Y5pzbrsVJInjORiafemNX9mffw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5d7icianNcN8uJA7Hts0T9POaVc6vxPBSqEib7EK3lzrvDbtWbNkjkW4gw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5AMXnmxr9xwVS9t2OpvDoeUHoxB68XBv0HWNYyLfcc1U21L0BHLgCVQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5NmTiaxQ7R9APfl4r6xhK8cib3H0GNqicy42oMeDZjeicU0NwJE0cfQzkuA/640?wx_fmt=png)

279 号控制流，第 33 步，之前 128 位大数组第 12 位是个 `undefined`，这里就将第 12 位填充上了一个 4 位数组，其中有个变量 `_$8L`，前面我们跟步骤的时候就有一个变量一直在做位运算，此处的 `_$8L` 就是这么来的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5HAMxL8d5Ribe3MREQj3eUWB3EialHCXW9VSvgYk4YrOGsw1Ok56zvghA/640?wx_fmt=png)

279 号控制流，最后两步，原来的 128 位大数组，只取有值的前 21 位，一共有多少位与 `window.localStorage` 的某些值有关，有值的话就长一些，没有就短一些，然后再将数组的每个元素合并成最终的一个大数组并返回，279 号控制流就结束了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx550Zvs8SojYQ5xUMaia2KMq0JBiaTEBMqcbaQdLbcDzHtVjf3szd4V3lQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Z86KJ8ayboveGvuD54B3ZUx2wSo0tHHezRP5ficPpwvYkQyqrKewO8w/640?wx_fmt=png)

返回到文章开头的逻辑，279 号控制流结束，返回到 742 号控制流，第 2 步，定义了一个变量并生成了一个 32 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ZG48dLxTKwg4MSxSSbQgzdGeTPCQq4g2dkp8B43c7fPaOhrpEibQEOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5O4ic5kBGgEaIs2Dsx31iaLxq8lKRBQgBeyZepOZAYAic12NWx0liaqNgEg/640?wx_fmt=png)

742 号控制流，第 3 步，取 `$_ts` 里面的某个值并赋值给一个变量。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5qsDP29CG3MThjOcvDKaAm4jymlas17Bj5OgmQ5RvTMIIDicCQHXZVxg/640?wx_fmt=png)

742 号控制流，第 4 步，将前面 279 号控制流得到的大数组与上一步 `$_ts` 里面的某个值进行合并，合并后计算得到一个值。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5jafg5NLz6VRw64mUxOxj6ketqbQibZHKcicU1fhSYj0dQx5aIsR3iamuw/640?wx_fmt=png)

742 号控制流，第 4 步，将上一步得到的值进一步计算得到一个 4 位数组，再将其和大数组合并。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5g3CmX6Fg9YrfDCQe4soHcCHgic9wXg9mL5RVE6gdiayKBzAuWjib1700w/640?wx_fmt=png)

742 号控制流，接下来几步是对时间戳进行各种操作，这里统称为第 5 步。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5v4icPs7x5PC9I0VsIP0OUrsNPkgpmiaeAkvlKZibnoAPoa91zeUl4Lf9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Or5iayibqk2DcUbkSJf8knsP0mNnicsY3ib5OCsO4r13zBJwmf0K3YarcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5JYf5D3mmBJ4wfWbq335jO27sWicVQcfdX7FBaBPcTY02Q1VXBJM6WqQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5HzpaypGgYjYQGlfDfMVjkJicwROWh7OFO72Jx0P9z7jmTDhskKwlUEg/640?wx_fmt=png)

742 号控制流，第 6 步，将上一步得到的 4 个时间戳进行计算，得到一个 16 位数组。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx595YMSlX9xk3ticp7cE7aqBuWjGLU8JzLGrOS0muRae7BJepGIRQ4Flg/640?wx_fmt=png)

742 号控制流，第 7 步，将上一步得到的 16 位数组进行异或运算。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx585qZicIlrvcKQxZcD7Ovls1w2XWWnicb7AYTZqBEhKx1jE6YRjBicicTVw/640?wx_fmt=png)

742 号控制流，第 8 步，将上一步的 16 位数组进行计算，得到一个字符串。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5v3yHpZ0fFV5Uj3J9XRExGXicgAn4OVqp8nq33jibK7WZnCTOxHDibicotg/640?wx_fmt=png)

742 号控制流，第 9 步，正式生成 cookie 值，其中 `_$bd[274]` 定值，一般视为版本号，将上一步得到的字符串、之前得到的大数组和一个 32 位数组进行计算、组合，得到最终结果。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5crY4c5Hibwcslu9DeOrwicJo8NqPuTXoOhJhwBYuFlso1t243KfJibkjQ/640?wx_fmt=png)

742 号控制流结束，返回 772 号控制流，利用了一个方法，组装 cookie，然后赋值给 `document.cookie`，整个流程就结束了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5nrRoWnsJ8lo7t8icQmCdRwmWGAoDLddZIcRfAevm8xYZ6et8F2QRibibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx52QleV5XAWkSwiag2SdJbA1usGp4YxfZdThem5ibjWpoar9JQHSUNoHQw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5UzT8QLmDaZFD6AcicmjzXFxUYn9NibVGS1dlsocV9tFibBlrAIruGVrCg/640?wx_fmt=png)

代码中用到的 `$_ts` 的值需要我们自己去匹配出来，动态替换，这些步骤和 4 代是类似的，本文就不再重复叙述，可以参考 K 哥 4 代的那篇文章进行处理即可。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5tTYJqIBhZnGOIdjzE07HqQXAWX0xSib9HvPYElZuIPrGqDQP4UuAa6Q/640?wx_fmt=png)

后缀生成
----

本例中，请求头中有个 sign 参数，Query String Parameters 有两个后缀参数，这两个后缀和 4 代类似，都是瑞数生成的。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5HxiaGTicl3KNf0PDjuqUicU77Ul2tu2ByEjIrzP9wN2Bia7ibKnTOO2mKSA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5liabyECaj5awQLQb1yYBDiaoaXVPLRiaeZOJNic977G5zYT3r1LSgNl1Yw/640?wx_fmt=png)

和 4 代的处理方法一样，我们下一个 XHR 断点，先让网页加载完毕，然后打开开发者工具，过掉无限 debugger 后，点击搜索就会断下，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5gLLacMM9AVtfwKSdqz41lFv3Mn4mZrOWIIRr2C5uuKmsOh7I8N1GBw/640?wx_fmt=png)

往上跟栈到 `hasTokenGet`，是一个 sojson 旗下的 jsjiami v6 混淆，不值一提，重点是 `jsonMD5ToStr` 方法，先对传进去的参数做了一些编码处理，最后返回的是 `hex_md5`，和在线 MD5 加密的结果是一样的，说明是标准的 MD5。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5GjlJciaPAibCSzKfROfeu3DicialsQKSDwFVnljeSFh21SiaSxUvLaKqlwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5ptG80SjQSLDHsde9ZVLIIH9JOkM7maj7zoldsib8ga8r8icffnkWrANw/640?wx_fmt=png)

重点来看瑞数的两个后缀生成方式，和 4 代一样，`XMLHttpRequest.send` 和 `XMLHttpRequest.open` 被重写了，如下图所示，在 `XMLHttpRequest.open` 下个断点，也就是图中的 `_$RQ` 方法，`arguments[1]` 就是原始 URL，经过图中的 `_$tB` 方法处理后就能拿到后缀。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5eicOTwo0Xe9jIlWdExpiaianTXJ1fGM6TKEAukrIIHEclvJF1Ccrpa6sg/640?wx_fmt=png)

跟进图中的 `_$tB` 方法，`_$tB` 方法里嵌套了一些其他方法，走一遍逻辑，到图中的 `_$5j` 方法里，前面的一部分都是在对传入的 URL 做处理。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5TKLGuPTYPfkf5Hn3UNmvZuAXNWAYrAXcJZgjdrMrvDPdGgHlQKyjIA/640?wx_fmt=png)

接下来是生成了一个 16 位数组：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5PFPRQ4RsrePBx0SFy0PH4kxG2fibL4OQkZryMbLl3Q5x70a5qYQTQOA/640?wx_fmt=png)

然后这个 16 位数组经过一个方法后就生成了第一个后缀，如下图所示，本文中这个方法是 `_$ZO`。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5mB5PgnSyDIiaVZdSHDo42UJz1Zq0NIIMPGnoWicTyI5s8K8UIpS8K7uA/640?wx_fmt=png)

跟进 `_$ZO` 方法，主要有以下 5 步：

第 1 步：生成了一个 32 位数组；

第 2 步：将之前的 16 位数组以及两个变量拼接生成一个 50 位的数组；

第 3 步：进入 744 控制流，这里你会发现和之前我们跟 cookie 时的 742 号控制流是一样的，重复走了一遍，所以这里就不再跟了；

第 4 步：将生成的第一个后缀值进行处理，得到一个两位的字符串，这个字符串在获取第二个后缀的时候会用到；

第 5 步：将第一个后缀名称和值进行拼接并返回，此时，第一个后缀 `hKHnQfLv` 就生成了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5wGYNG1rwibNfq66iabc6MhFiaVVrTzQ9jnhvZeRNE75iadZvcyLlulqJdw/640?wx_fmt=png)

接着前面的 `_$5j` 方法，图中的 `_$5j` 这一步，就是获取第二个后缀 `8X7Yi61c` 的值：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5C1VfIJp0LHbZTZVt3kVialH7vALECtmVBu0CzeFOAfRqJbicf4Hn8JZw/640?wx_fmt=png)

主要是看一下图中的 `_$UM` 方法，先将前面生成的两位的字符串与 URL 参数进行拼接，然后会经过一个 `_$Nr` 方法就能得到第二个后缀的值了。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5cI8WhEpdeyKYj6rAXAAYD4LQ9aeqvL5n84MnFLG5Cd68P9QrhIJBUw/640?wx_fmt=png)

再来看一下 `_$Nr` 方法，先生成一个类似 53924 的值，然后一个 try 语句，注意这里有个方法，图中的 `_$Js` 方法，里面用到了 `$_ts` 里面的某个值，后面又生成了一个由数字组成的字符串，再次经过组合、计算后得到最终的值。

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5Zsbo7W9t0D5icCicl589icSLWhVXdmoe4P8qZxfQrm7zsoZx8V948poOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5YYCicdSqbL7SAGRqd6UtoeetpjsCuJBkEvEJ34Zc71hoEKXmcf1goFg/640?wx_fmt=png)

回到前面的 `_$UM` 方法，前缀 `8X7Yi61c` 与值组合，自此，两个后缀都拿到了：

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4J9sNCjlzknOicXRTSHyIxx5pXLibrYSia8qoNUXS8ORp6MaJRqShyvns6jwSkkqGiaXStmsqnzWHZkIA/640?wx_fmt=png)

指纹生成
----

我们前面已经分析了，在往 128 位数组里添加值的时候，会有取 `window.localStorage` 里面的某些值进行计算的步骤，这些值就是取浏览器 canvas 等指纹生成的，指纹随机就能并发，通常访问单独的一个 html 页面是不校验指纹的，生成的短 cookie 就能通过，但是一些查询数据接口会校验指纹，通过触发 load 事件来向 cookie 里添加指纹，使得 cookie 长度变长，怎么查找指纹在哪里生成的，这里推荐直接看视频资料，已经讲得很清楚了，篇幅太长，本文就不再赘述了，资料链接：‍‍‍‍‍‍‍‍‍‍‍‍‍https://mp.weixin.qq.com/s/DEUc1K8WaO_Cq1a2r0Ge5g

END
---

![](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4L2BEhohiciaycd6acftNGOyxLZkbZal8LW2qPk5MXg11hB0qia0s0mdecYM7sLzaezgIzkkiaiahIRY5g/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_svg/XYrRG5UShDeGibNoQZgXicJOW4Ss1q8yN1xRqONKKlPnGh7dvAdcvuT8tYuGSeDJibicszI0CZPShu9UtRnEvA9shbglGps4fucP/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/9UjCmequjUickLicqdUmtavXkUejKTHRF28k1CiayichS5TGzHLfAOF0UjWRmTaolibeFRpZQ5XOG0zEvfZZOGeJTVgRYZ3VvDDNZ/640?wx_fmt=svg)

点个在看你最好看

![](https://mmbiz.qpic.cn/mmbiz_svg/ylRhrSjQb8jeDpnF88X2eeSg1lzyKxW6EO1zSCZC3wCLAdPNomrSgTBWpHcGxxGNQTXbTC82mySYiaKThz99VBqX7t3uSBcrU/640?wx_fmt=svg)