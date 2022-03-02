> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2NDY4Njc4MQ==&mid=2247483900&idx=1&sn=1caea941f9e01a5d76f05218ed3f64a2&chksm=ce64c62ef9134f38359bc844423825dbd73b4dd11093214c8b6e22df0ad934a09c358cc8127e&mpshare=1&scene=1&srcid=0216eLF1AYSuiWJUbUBt77U9&sharer_sharetime=1645025996150&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

夕夕跟我的羁绊很深，记得我刚转行接触到第一个项目就是爬夕夕的数据。但正当我编好程序爬的不亦乐乎的时候，夕夕突然在接口添加了一个动态的加密参数。这一猝不及防的操作，直接导致公司项目崩盘，项目经理也没能捱到最后，带上俩测试妹子就跑了，从此杳无音讯。

当时，整个公司都笼罩在夕夕的恐怖阴影下，可以说已经到了谈 “夕” 色变人人自危的地步。我们技术总监，为了尽快让项目恢复运营，不惜动用了公司的全部技术团队，对这个加密的参数进行破解，但都无济于事。然而让人感到欣慰的是，通过这次风波总监和我们之间的感情却意外得到了升温，因为他每次被老板骂的时候，都会站在门口对我们喊："你们都是猪吗？！"  

再后来，有人请出了一位广东那边的大佬后，问题才得以解决。但对于破解项目中只有参与权的我来说：虽然我当时也被夕夕折磨的体无完肤，但后期还是凭借自己的努力克敌制胜。从而也明白个道理：**杀不死你的东西，终究会让你强大。**

好了，故事讲完了，我们开始进入今天的正题：夕夕的 **messagePack** 算法。

我们通过网络请求分析，会发现一个名为 **anti_content** 的动态参数

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEDpToUsQ2BjPuysnHfBksvPKWK8zYh4ZP3nibGWd3o0hAdU6XxNM98Aw/640?wx_fmt=png)

如果这个不携带这个参数或者把这个参数写写死进行请求，就会出现请求异常的错误提示  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEIqCPdsXwNnpWJPh3KcPj6bFcA60iciaCjVymY2ibWfqhWnV9wUWoZziaibg/640?wx_fmt=png)

我们直接直接通过 **xhr** 搜索关键的方式进行断点

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEzHaOEcaR8XzyJXDQKRiaqJSLjiaZrQdl9VKdhuhfkiaEYOgqxESSVtQDA/640?wx_fmt=png)

然后我们下拉滚动条，让他加载更多的商品。与此同时，我们的断点也会随着生效，最终它停在了这个地方

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oE0D1qm3rOiabnIWmhvq5VGm7ysKYicC6gIhSVr2UY9IPnExo7CJBZrPcg/640?wx_fmt=png)

然后我们去分析右侧 **call stack** 调用栈，然后我们发现了这样一个可疑的地方

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEHIHMX1N5CSyVbNUzoqh6PuIS1A2PVTxppEtetzATvqGCPw9WziahZeg/640?wx_fmt=png)

我们在这里打上断点后，直接刷新浏览器，会发现这个异步操作，而这个 **pending** 是一种等待的状态  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEWkf2x30T0mTO8d7a2EDaEODu92ib2IsgOyOXcxXHWapxOmicE0e5Jicvg/640?wx_fmt=png)

因为它没有明确的返回值，可以点击下图这个按钮跳出方法，在调用栈从后往前分析它的经过

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEWL2wbpHAbGeLYehcHibMzicHld2azfEKR82InNmroibCeXoA27Gst113A/640?wx_fmt=png)

然后我们可以看到这个方法，最后一次调用的所经历的过程

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEZjYjcJUDPgxUjlv7ibLTBv5LoicCBzM4vic3dNe69PqY4ZWciaiadTcaTTQ/640?wx_fmt=png)

我们直接把断点打在这里，分析一下这个过程  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oE7cEm02510hRZiaicoUxe0sTbDjHibkJ8fUaCIPPic904PHlxZx6OwRJUlQ/640?wx_fmt=png)

可以看到它会运行到这里，而且会把 **t.messagePackSync** 作为返回值

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEoORmgafRC6OjmQvxOVSgIicdbhKow7vWt9E8NSjKBXtQ2B6uTUQgWBA/640?wx_fmt=png)

然后我们在 **console** 控制台进行调用，发现返回的参数正是我们我们所要破解的 **anti_content** **不过这个异步调用过程，后期我们会分析一下用** **python** **对它的调用方式**  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEvC4hkaoVc0fm3iaZgKGtdMouD3MeEwXEhxuFB5YZvrn22Hg01JZ7MZg/640?wx_fmt=png)

而 **t.messagePackSync** 指向了这个方法  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEWUOdAoOyfGeicwyBu7nzX5RosI19kYhSIsxlKjSeqB5nYxibefSRJHfw/640?wx_fmt=png)

我们在这个方法的返回值处，打上断点就会对它的加密位置更加明确  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEnQhfiaevibnDBRll7UzaicL1MYkl57AInxfQgz2lgLF6XLibBolBmcv6zw/640?wx_fmt=png)

至此，我们已经明确了这个加密的最终生成位置，下面要做的事就是将这个方法逐一找出来。但这里我们也一定要明确一个概念：对于这种 **webpack** 打包的代码我们一定要有全局意识：**要把整个包作为一个整体，而不是单一的方法**  

我们新建一个 **html** 文件并在文件内部创建 **script** 标签

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AntiContent</title>
</head>
<body>
<script type="text/javascript">
    // todo 填写代码
</script>
</body>
</html>

```

  
我们先将 **Jt** 相关的整个对象进行拷贝并粘贴到标签中，并且给他定义一个名字  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEtt8Cb8QdxxWC76dhpsNzok8vB6ic1cph6t4zQO3ENC9Q1OIc10XZfrw/640?wx_fmt=png)

然后我们以浏览器方式打开这个文件，并在 **console** 控制台调用我们定义到的 **messagePack** 方法，顿时出现了个错误

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEMibCPSzMt0AvtrVQRDYb0JZI7ic61ngplia948DElu20v5hJpZ1XiaX94A/640?wx_fmt=png)

根据错误提示，我们定位的错误的位置

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEjW8ABQrBynGFHqA2KtIrNCt9xVenjsx4I8MJ6J8xT0TldxQXuiaEMqA/640?wx_fmt=png)

是这个 **r(3)** 未定义，跟先前一样，我们需要跟官方对照，然后模拟出 **r(3)** 对象的所包含的信息。这是官方 **r(3)** 对象所包含的信息

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oENhCSCUGEmIbB3GVC51SHCsc4ib9GWwNJgVRLDvVjibIsHYmZkxjftqDg/640?wx_fmt=png)

然后我们从其中一个方法进行切入，就拿其中的 **addListener** 举个例子  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEvccgticDc0HzC4WibgMyTNrEXvSDg9earkErK7iaY5JocBqTBoNZxM8dw/640?wx_fmt=png)

然后我们将这个整体函数进行拷贝，并以 **r3** 进行命名

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEfzPoW0ZzW8N6q9sGic3iaqt2uQrS2FFDQzjGMd7sTBWfehA0D6jV402g/640?wx_fmt=png)

当我们 **console** 调用 **r3** 时候却出现一个 **exports** 不能赋值的问题

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oE4OLlKsrTvMvRValFzHTvxicibLyubsZnryY0PVfibm30Nnd4hZibGqIGicA/640?wx_fmt=png)

这时我们可以将 **exports** 作为一个空对象进行传入就可以解决此问题。但却发现 **r3** 没有返回任何东西

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEzicOGcnozHQRxwFDD9ULtdot1Wyvgk9oxtyCKM28fLsibIGLD3koOkWw/640?wx_fmt=png)

这是因为你本身没有给 **r****3** 返回任何东西，所以我们要将 **a** 对象进行返回

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oESbyXwY4JVu01rg21EzI1N1GoQ0HxD6akMcUdAhia2PpDLHLZibKibaicTQ/640?wx_fmt=png)

再次调用就会发现，**r3** 对象有了所对应的信息

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEeOjaZBXUVXMiaxoYJX04s76aaKj6P79Vd7mdJmbCib3Lh7LRyXH01rPw/640?wx_fmt=png)

别忘了将 **r3** 进行传入 **messagePack** 函数

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEqhJa9ibn8QILAZE9XddIZH8eaE81pcktDZI9hpy8nxsHvCVG3JicbQQg/640?wx_fmt=png)

我们打开浏览器再次调用 **messagePack** 函数时又发现 **r(0)** 未定义。我们如法炮制即可。这是 **r(0)(t)**  所对应的对象

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEkKmAjsnuTjtwZo699R1InlKJ5xaTlOjUyfzswMlXGnfmdsv914tXBA/640?wx_fmt=png)

我们找到 **r(0)** 所对应的方法并给他增加返回值  

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEaI7nk2aESqhTXAlXWrEwX5IUTcsmM8hdPQTkFzXfXpeoYeD9P9oCuQ/640?wx_fmt=png)

然而它的调用方式跟先前的有点不太一样，但好在能力所及

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEQUjQtuwagAe2l36iakm32e0JribCVtibiaKLzMonrax6ME2GJKyicvBbV6Q/640?wx_fmt=png)

然后我们再对主函数进行调用，发现以下这些对象都存在未定义的情况

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaFNt60Ob5E9hh9XMBuW05oEqGvmSWphPXASaRwwGRdzHL6dgs2JkxJp4P7espaHsc31XehV3K5NFg/640?wx_fmt=png)

这里面除了 **r(1****5)** 返回个方法外，其余的返回了一个对象。以下是 **r(15)** 返回所对应的函数，我们着重研究一下

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodCXWcj9WMmFLhEFHB4NEr9c201ql5M5MY1sNWE00A3hRLOpmK6wXvCw/640?wx_fmt=png)

我们先看 **d("0x3f", "&$Jn")** 代表什么意思

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodj9WddJIMBI0lHbMwTUq99cgRp0mu1bQL5zrBtufqDSPOyG2sUsPvrw/640?wx_fmt=png)

也就是说 **t.exports** 代表中 **r(15)** 返回的方法，所以我们可以这样做，将其返回

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodoemAp7fdoWicm2UjAjibT1iaxXiaFBHj14wYhdbPU3dKLScgictBJntbMfg/640?wx_fmt=png)

调用之后就跟官方有了一模一样的结果

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodaaQr1kR60ObTMAmxozGnhB7G0sm2g1AUibmNwV1eTWUibVkbK49lbJxQ/640?wx_fmt=png)

当我们把所有对象和方法补全后，再次调用 **messagePack** 方法，却发现并不是我们想要的结果

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodMJJLK94s4uSrnMBLur6kiajgZpuEyjVb5Pp8QEUCia6BoVYsL3p5ic2kg/640?wx_fmt=png)

然而，它的所有秘密隐藏在了它的 **prototype** 属性中

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodmvq9sIH06fcVS04Jzl4rUqvicia8wVp79PCrfnsgezRp5k3TIJr7qmEA/640?wx_fmt=png)

  
我们直接调用其中的 **messagePackSync** 方法，确实是我们想要的结果

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbodnibJicyl2XvibO9COT8Y1mZnaDffcADryBSxlkrceUcSIewvbgmlDDTEw/640?wx_fmt=png)

不过有个问题也会应运而生，那就是有关异步调用的问题，因为在 **pyExecJs** 模块中是不支持 **python** 调用 **node** 的异步函数的。这时我们可以独辟蹊径，将 **node** 的异步返回值打印出来 

```
// code for javascript

```

```
messagePack.prototype.messagePackSync().then(function (anti_content) {
    console.log(anti_content);
})

```

然后利用 **python** **os.open** 进行读取，就可以解决此类问题  

```
# code for python


```

```
def get_anti_content():
    cmd = "node antiContent.js"
    anti_content = os.popen(cmd).read().strip()
    return anti_content


```

**最终的效果图：**

![](https://mmbiz.qpic.cn/mmbiz_png/7QYmakzsJiaEFhV3ibxwXsff6ZnsTYbbod9OGckwmfREXzXU3BntZWXy63bLvykToiaQtRqz12765Bh66rUQnOLWg/640?wx_fmt=png)

**最后声明：**

******✧✧** **本篇文章只用于学习交流，切勿用于商业用途，如果侵犯权益，请与作者联系，本人会在第一时间将其删除。**✧✧********