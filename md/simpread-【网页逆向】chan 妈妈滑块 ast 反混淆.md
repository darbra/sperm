> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rImqhwTBnsdOiaXWqnyBDw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGeficG3ekgCErFHtLxX6UyTKlzy6tmoVib5r3QrR2GdUz0FCHzNA2lEBQ/640?wx_fmt=jpeg&from=appmsg&randomid=skwo23y2&watermark=1)

  

集美语录
----

后续文章会开个某书平台集美的 “红学” 语录：

> ❝
> 
> “等你开口要才给的男人是没有付出意识的”

  

解混淆
---

跟栈发现主要的逻辑都在这个混淆的 js 文件里面![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGNIxqibtspJjQILKatNPCTufiaqMb3futIgdKKajLfHPqQ5SfFlD2HmEw/640?wx_fmt=png&from=appmsg&randomid=5lhlla95&watermark=1)

简单理下组成，很典型具备解密、数组、自执行的 ob 混淆

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGqezXAp8Cely9RWsrOiaALtRwLTCWyB4AnojKdxH3RlmrbBDbgIZNkwA/640?wx_fmt=png&from=appmsg&randomid=ixcqu5mq&watermark=1)

### 解密函数多次套用

难点在于业务代码中对于解密函数进行了多次引用

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGT7wLe4dBmVibRvtjqxzxfftICp0tVibcyptpK87oglicCzqyfF2a890sw/640?wx_fmt=png&from=appmsg&randomid=hlrsdhg4&watermark=1)

有的甚至进行了三层引用![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGWBtxMz64ib1mNLh2Z2icmy0e8RCX2GYBUCPq9ApH15HuV99yPaU4BHjQ/640?wx_fmt=png&from=appmsg&randomid=d5hc14d9&watermark=1)

先搭框架

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBG7TOMcFEvjpuD1oBoSEib2U9MbIaZkIfWdz9JBb2zHr5JYnic6b4caPjw/640?wx_fmt=png&from=appmsg&randomid=b04kpsos&watermark=1)

```
traverse(ast, {
    FunctionDeclaration(path) {
        const node = path.node;
        const body = node.body;
        if (node.id.name !== 'a0_0x4d16' || !body.body || body.body.length !== 2) return;
        console.log(`>>>: 查找到解密函数, 导出成功`)
        global["decrypt"] = path.toString()
    }
})
traverse(ast, {
    FunctionDeclaration(path) {
        const node = path.node;
        const body = node.body;
        if (node.id.name !== 'a0_0x8afe' || !body.body || body.body.length !== 3) return;
        console.log(`>>>: 查找到大数组, 导出成功`)
        global["largeArray"] = path.toString();
    }
})
traverse(ast, {
    CallExpression(path) {
        const node = path.node;
        const callee = node.callee;
        if (!callee) return;
        const body = callee.body
        if (!body) return;
        if (!body.body || body.body.length !== 3 || body.body[2].type !== 'WhileStatement') return;
        if (!node.arguments || node.arguments.length !== 2 || node.arguments[1].type !== 'NumericLiteral') return;
        let changeAst = parser.parse("!" + path.toString());
        let ChangeArrayOrder = generator(changeAst, opts = {"compact": true}).code;
        console.log(ChangeArrayOrder)
        console.log(`>>>: 查找到自执行函数, 执行成功`)
        global["ChangeArrayOrder"] = ChangeArrayOrder;
    }
})

eval(global["largeArray"]);
eval(decrypt); 
eval(global["ChangeArrayOrder"])


```

### 方案

下面来解决下解密函数套用的问题，首先这里了解下第一性原则：

> ❝
> 
> “第一性原则”（First Principles）是一个源于哲学、后被广泛应用于科学、工程、商业等领域的思维工具，核心是回归事物最本质的起点，避开经验和类比的干扰，从根本上推导解决方案

无论业务代码中调用的是哪个套用后的解密函数，比如下面_0x40fdb3，_0x5a580d，我们最终都希望的是改为调用最跟的哪个解密函数: a0_0x4d16

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGdrpicBkkXpu75WIM9eZl6jy60iaDql14pR5az2YYiaAvk9N3Uh9G6ZZlA/640?wx_fmt=png&from=appmsg&randomid=7fqzlw2k&watermark=1)

然后就面临两个问题：如何替换以及参数怎么传递

先来解决参数的问题：

举例子来讲：

调用：

```
_0x82fdf1('2(gU', 0x65f, 0x473, 0x6da, 0x301)


```

原函数:

```
function _0x82fdf1(_0x2fa324, _0x2aec72, _0x54f87c, _0x2f9b57, _0x54311f) {
 return a0_0xe336(_0x54f87c - -0x179, _0x2fa324);
}


```

希望变为:

```
a0_0xe336(0x473 - -0x179, '2(gU')


```

过程：

```
// 先维护一个_0x82fdf1实参和形参的对应关系
params1 = [
    "_0x2fa324", "_0x2aec72", "_0x54f87c", "_0x2f9b57", "_0x54311f"
]

params2 = [
    '2(gU', 0x65f, 0x473, 0x6da, 0x301
]

const _maps = {}
for (let i = 0; i < params1.length; i++) {
    _maps[params1[i]] = params2[i];
}


// 最后替换解密函数a0_0x4d16参数
params3 = ['_0x54f87c - -0x179', '_0x2fa324']

for (let i = 0; i < params3.length; i++) {
    if (type.isBinaryExpression(params3[i])) {
        params3[i].left = paramMap[params3[i].left.name];
    } else {
        params3[i] = paramMap[params3[i].name];
    }
}


```

参数解决了最后就是套用解密函数

```
1 先找到所有最初解密函数a0_0x4d16的引用
2 如果引用的父级的父级仍然是return，继续寻找父级引用
3 如果非return，且不是a0_0x4d16，替换该引用的父级CallExpression节点为函数的return节点的参数


```

举个例子

```
_0x3fc5a5(0x123, 'SqSF', -0x34, 0xc8, -0x1f)

function _0x3fc5a5(_0x4a9d3f, _0x2562e6, _0x3b4e65, _0x2fb756, _0x168173) {
return _0x19e53f(_0x4a9d3f - 0x1d1, _0x168173 - -0x3dc, _0x3b4e65 - 0xff, _0x2fb756 - 0xa8, _0x2562e6);
}
function _0x19e53f(_0x31a276, _0x4c662a, _0x4b38bd, _0x4f8c37, _0x29502d) {
return _0x490c45(_0x31a276 - 0x1c2, _0x4c662a - 0x171, _0x29502d, _0x4f8c37 - 0xf9, _0x4c662a - 0x18);
}


1 

function _0x490c45(_0x158666, _0x5a59d2, _0x2acf8c, _0x2b3a97, _0x4ebef9) {
return a0_0x4d16(_0x4ebef9 - 0x15e, _0x2acf8c);
}

2 找_0x490c45的调用
  _0x490c45(_0x31a276 - 0x1c2, _0x4c662a - 0x171, _0x29502d, _0x4f8c37 - 0xf9, _0x4c662a - 0x18);


  判断其parentPath 是否还是return如何是就递归decode

  找_0x19e53f的引用

  _0x19e53f(_0x4a9d3f - 0x1d1, _0x168173 - -0x3dc, _0x3b4e65 - 0xff, _0x2fb756 - 0xa8, _0x2562e6);
  判断其parentPath 是否还是return如何是就递归decode

  _0x3fc5a5(0x123, 'SqSF', -0x34, 0xc8, -0x1f)  

  判断其parentPath 是否还是return 不是，然后开始替换

  1 _0x3fc5a5(0x123, 'SqSF', -0x34, 0xc8, -0x1f)
  2 _0x19e53f(0x123 - 0x1d1, -0x1f - -0x3dc, -0x34 - 0xff, 0xc8 - 0xa8, 'SqSF')
  3 _0x490c45(0x123 - 0x1d1 - 0x1c2, -0x1f - -0x3dc - 0x171, 'SqSF', 0xc8 - 0xa8 - 0xf9, -0x1f - -0x3dc - 0x18)
  4 a0_0x4d16 直到替换出a0_0x4d16


```

然后就出结果了![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGkzSjpLCZWNNzMyRYemxt7rTFjXokELEI1FRdtxWreCAicnMd33xqiafQ/640?wx_fmt=png&from=appmsg&randomid=06poichj&watermark=1)

后面滑块部分就简单了

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGvONCoKGhgViaHhae9GPtSNEAKEPlOGAsL9eMagLo2T9QECpCcGSje3w/640?wx_fmt=png&from=appmsg&randomid=2dro76p2&watermark=1)

主要分析下 position、和 path 怎么来的，跟着栈往前回溯，可以找到 onDragging 和 setRawList 地方，然后打上日志断点

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGs2oZUEengI5ZKY9F7vPOYBE02YcXPe0jL6NFgJsTM51a64pMTSaMsQ/640?wx_fmt=png&from=appmsg&randomid=wdttbfx0&watermark=1)

可以看到在 onDragging 地方对 x 进行了缩放，x1 = 移动到的 distance / 340 * 100

t 就是当前时间减去开始时间

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGNtlhH8dAQ8icdXsqtA6ibOalljXKlicWBicstW6066a1ibXvArLAabctib1g/640?wx_fmt=png&from=appmsg&randomid=w5y0m53a&watermark=1)

最后加密就是个 aes，av 和 key 也都是版本固定的

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGOto0og6yJZ9ftmwzPm597BzncmkhAzWEnJFMFUjIfdu43D00gQxpUw/640?wx_fmt=jpeg&from=appmsg&randomid=2g511uwb&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGL0Dica2fcYVJ94JdquU7pVlDI9lZ7RhHEI0VhpLnmf3QBr1MSMCqHqA/640?wx_fmt=png&from=appmsg&randomid=tpmgxhiv&watermark=1)

完事！![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGw7dwfB15sjno8Ruuh4JiaOJYtb25F0ZrDl9SCcQ3YJ2zHQw5uRseFmQ/640?wx_fmt=png&from=appmsg&randomid=r4b53nph&watermark=1)

END
---

> ❝
> 
> 有需要用于爬虫、逆向、抓包的测试机又不想花时间折腾 Root、刷机、环境配置可以来看下下面联系方式，或者 B 站视频评论区见

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuycO19D2ficibXlFOFNM8VPBGSZicWevias8RibD1EkFfW67CfoLHnoBfWbIBefq9lLzBH01aqiccrSBKJw/640?wx_fmt=png&from=appmsg&randomid=ylclqrrn&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuycO19D2ficibXlFOFNM8VPBG2TiaLqnSwlYRkoWPmRQSRUvYTnAD2iakNabShhMQkdbMIPgvRicFLndLA/640?wx_fmt=jpeg&from=appmsg&randomid=8uyctyzt&watermark=1)