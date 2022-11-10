> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/p5di2mWVLut7Grv4TDQvkg)

  

      大家好，我是 TheWeiJun；光阴似箭、日月如梭，突然发现又有好长时间没有更新了。还好总有粉丝朋友找我提问，今天更新一篇粉丝 **Robbers** 提到的网站问题，主要涉及 js 逆向和 tls 指纹绕过。欢迎各位读者朋友多多阅读与交流！

特别声明：本公众号文章只用于学术研究，不作为其它不法用途；如有侵权请联系作者删除。

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A6N3l9obtATicYg0vSreUhmEsiciaibJfHjG8MEHndnoJycRKpgeeK3LkxKu6qlz7oyC2Gelexa4W4Bfw/640?wx_fmt=gif)

立即加星标

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6N3l9obtATicYg0vSreUhmEG7mwRfEUOichNPSVJkJl4IDcjeqvJyO5GGxL9CaYcRQqg13wjyXVzicQ/640?wx_fmt=png)

每天看好文

 目录

  

一、前言介绍

二、参数分析

三、断点调试

四、算法分析

五、指纹绕过

六、学习展望  

趣味模块

       Robbers 是一名 spider 工程师，最近 Robbers 遇到了一个棘手的问题：Robbers 在访问某某网站时，遇到了 JS 加密参数。Robbers 凭借自己超高的专业技能对该加密参数逆向还原后，用 requests、httpx、aiohttp 等包去发包，居然认证不通过，提示身份授权失败。这篇文章，我们将和 Robbers 一起并肩作战，去解决这个问题！

一、前言介绍

      我们在以往的文章中都是提到了如何从 params、data、headers、cookies、response 中去还原加密参数，通过还原加密参数的方式即可实现数据采集。而今天我们要分享的文章中，和提到的这几个类型完全没有任何关联，遇到这样的问题，该如何解决这类型的问题？带着这些疑问耐心看完本篇文章，你就豁然开朗了！

二、参数分析  

1、首先打开我们今天要模拟的网站，刷新当前页面，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4ykgD3coXpmnEdBeicknqdozf350nUOAekmKibGLDoILJX9UicOIvnSLV9tIWLMADzLGFPq0uibj4a6A/640?wx_fmt=jpeg)

2、打开开发者工具 DevTools, 选择 Network 栏目, 刷新当前页面，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoTribnhzCIHwRkEOs6fBpVpLZs0CTMSiahqKb6fXY8iaypeIuFdgY6LVog/640?wx_fmt=png)

3、经过分析可以确定该接口即为我们要获取数据的地址，接下来我们进行参数分析：  

**Request 参数分析：**

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoKLXIbLTicw1nLVG9So7CPcX2DeCGzkmgfds8nvibHNS1BBQ3EyEb2jJg/640?wx_fmt=png)

**总结：**该接口都是明文，不需要进行任何还原。

**Headers 参数分析：**

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdocExlQJPgLKH8coLu6ic9CNlJkrnVBv4wSpib7zHd8av5VGCRgFPKBZsw/640?wx_fmt=png)

**总结：**u-sign 目测为 md5 算法加密参数。

通过分析，我们可以确定 u-sign 参数是被加密处理了。经过重放请求包，不能够缺少 u-sign 参数，接下来我们需要进入 JS 段点调试分析加密参数环节一探究竟了。

三、断点调试

1、使用 XHR/fetch 打上断点，当该请求发包的时候，捕获断点如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdojvggOdGb0C7c5jgXBlDuqIwR37ESYdAQDZ2LzGcj9VFmoMA9kCeUPg/640?wx_fmt=png)

2、在 Call Stack 栏目中追溯 headers 参数堆栈，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdohFjZE7Ctib7HN6xTxBVIZCNflhPUpt4oqJZ5zU51O35HKuZkqIWIkCg/640?wx_fmt=png)

3、在 u-sign 参数下面一行打上断点，查看 u-sign 对应的 value 的值，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo27mP7BsmQA6xfxkqgEdWAadSFYJ0GVbek3OvMaRmTsITcJtR2X4EXw/640?wx_fmt=png)

4、确定加密函数为 i 后，我们进入该函数，查看加密逻辑，截图如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo8icSOLxXrHd5ApaoEnfKpRtrgyl0zoGdJ1o2sWosDRgbjT0yw45RMTg/640?wx_fmt=png)

5、确定 n(a) 为 u-sign 参数的值后，我们在 console 输出 a 参数，copy(a) 用前面提到的 md5 去校验看是否等于该参数的值，截图如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoe1054Zdbh1xDofUypgVtttk2Q8MLUW7HRQKv1bRVaHLxGDYUxODLUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdooc6SVD0nSE816Zu2gEBTyviayTic3gX0pp1QzXU2oNphiaqGWaMLH8Srg/640?wx_fmt=png)

**总结：**确定参数算法及整个流程贯通后，接下来，我们只需要对 js 代码加密算法进行还原即可。

四、算法还原

**1、Python 版本算法还原代码如下：**

```
# -*- coding: utf-8 -*-
# --------------------------------------
# @author : 逆向与爬虫故事公众号
# --------------------------------------
import json
import hashlib
def get_sign(data: dict) -> str:
    data_str = json.dumps(data, separators=(',', ':'))
    text = f"{data_str.lower()}&9sasji5owng41irkisvtjhlxhmrysrp1"
    hash_md5 = hashlib.md5()
    hash_md5.update(text.encode())
    sign = hash_md5.hexdigest()
    return sign
if __name__ == '__main__':
    headers = {
        "Accept-Encoding": "gzip, deflate, br",
        "Connection": "keep-alive",
        'Accept': '*/*',
        'Accept-Language': 'zh-CN,zh;q=0.9',
        'Cache-Control': 'no-cache',
        "Content-Length": "132",
        'Pragma': 'no-cache',
        'Sec-Fetch-Dest': 'empty',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Site': 'same-site',
        'sec-ch-ua': '"Chromium";v="106", "Google Chrome";v="106", "Not;A=Brand";v="99"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"macOS"',
        'u-sign': '37635b63e0f1973c61b1444983eab1be',
        'u-token': '',
    }
    json_data = {
        'keyword': '',
        'provinceNames': [],
        'natureTypes': [],
        'eduLevel': '',
        'categories': [],
        'features': [],
        'pageIndex': 5,
        'pageSize': 20,
        'sort': 11,
    }
    sign = get_sign(json_data)
    print(sign)
    print(headers['u-sign'])

```

**2、代码运行后，******Pycharm 打印结果如下：****

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo366QsFPOibh0Z4P4Dsjf8mBjvTQbq0co0dabEvVAGJjzUXN7ZOXRhYw/640?wx_fmt=png)

**3、算法还原后，使用 Python 发送请求包，截图如下：**

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo4yicRgatgsdbdIwh9ib5fbbxfkAfGd65CDdUEMh7msQkJNUMNSsaRk7w/640?wx_fmt=png)

**总结：**参数完全一致的情况无法通过认证，接下来我们进入新的环节解决这个问题吧！

五、指纹绕过

1、在我们参数算法完全还原的情况，请求该网站却提示身份认证失败，我们重新梳理下可能存在的情况如下：  

*   cookies
    
*   http2.0
    
*   tls 指纹
    

**总结：**经过分析，我们可以确认该网站不需要 cookies，故第一种怀疑排除掉；接下来进行 http2.0 验证。  

2、可能比较傻，我确实用 httpx 验证了下该网站，截图如下：  

```
with httpx.Client(http2=True) as req:
    response = req.post('https://xxxxxxxx.cn/xxxxx.basiclib.api.college.query',
                        headers=headers, json=json_data)
    print(response.text)

```

Pycharm 代码运行后，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo8OiaAWmXXrTiaZI1Axpf9zM8yVv0J8S2nyfhU1s9qFibZjt623mzlqEEQ/640?wx_fmt=png)

**总结：**结局总是那么不理想，此刻怀疑的方案只剩下最后一种：该网站对 tls 请求指纹进行验证；接下来我们继续分析。

可能到这里会有人问，什么是 tls 指纹？

**TLS 指纹**，也有人叫 JA3 指纹。在创建 TLS 连接时，根据 TLS 协议在 Client Hello 阶段发送的数据包就是就是 TLS 指纹。不同浏览器、不同版本（不同框架）因为对协议的理解和应用不一样，所以发送的数据包内容也就不一样，所以就形成了 TLS 指纹。

3、使用 Postman 进行发包测试，不再使用 Python 第三方包，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoARaL4LmrHyia579AFiadVyafyFLujqCQ3ibMmpXBnKsXMR1MAJNDf8FhA/640?wx_fmt=png)

**总结：**看到此处后一下豁然开朗了，可以肯定对方服务端会对请求指纹进行校验，如果是我们刚刚使用的第三方包，都会被服务端给识别到，最后返回身份授权失败错误。那么我们如何过 TLS 指纹呢？

**怎么过 TLS 指纹？这是一个黑客大佬总结的几种方法：**

*   **代理中转请求**
    
*   **使用 Go 语言爬虫库**
    
*   **魔改 requests**
    
*   **访问 ip 指定 host 绕过 waf  
    **
    

4、接下来，我们使用 Golang 编写代码及还原算法如下：  

```
// Package main -----------------------------
// @author    : 逆向与爬虫的故事
// -------------------------------------------
package main
import (
  "crypto/md5"
  "fmt"
  "io/ioutil"
  "log"
  "net/http"
  "strings"
  "time"
)
func main() {
  client := &http.Client{}
  dataStr := `{"keyword":"","provinceNames":[],"natureTypes":[],"eduLevel":"","categories":[],"features":[],"pageIndex":2,"pageSize":20,"sort":11}`
  var data = strings.NewReader(dataStr)
  req, err := http.NewRequest("POST", "https://xxxxxx/xxxxx.query", data)
  if err != nil {
    log.Fatal(err)
  }
  sign := fmt.Sprintf("%x", md5.Sum([]byte(strings.ToLower(dataStr)+"&9sasji5owng41irkisvtjhlxhmrysrp1")))
  fmt.Println(sign)
  req.Header.Set("Accept", "*/*")
  req.Header.Set("Accept-Language", "zh-CN,zh;q=0.9")
  req.Header.Set("Connection", "keep-alive")
  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("Origin", "https://xxxxx.cn")
  req.Header.Set("Referer", "https://xxxxxx.cn/")
  req.Header.Set("Sec-Fetch-Dest", "empty")
  req.Header.Set("Sec-Fetch-Mode", "cors")
  req.Header.Set("Sec-Fetch-Site", "same-site")
  req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36")
  req.Header.Set("sec-ch-ua", `"Google Chrome";v="107", "Chromium";v="107", "Not=A?Brand";v="24"`)
  req.Header.Set("sec-ch-ua-mobile", "?0")
  req.Header.Set("sec-ch-ua-platform", `"Windows"`)
  req.Header.Set("u-sign", sign)
  req.Header.Set("u-token", "")
  resp, err := client.Do(req)
  if err != nil {
    log.Fatal(err)
  }
  defer resp.Body.Close()
  bodyText, err := ioutil.ReadAll(resp.Body)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Printf("%s\n", bodyText)
  fmt.Println(string(time.Now().Weekday()))
}

```

5、执行编辑好的代码，GoLand 打印如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoPxkvqTU2IpicXlImTaRkRfjDSDZCdNdgVzmAsawryFSFnF3W3sgU60w/640?wx_fmt=png)

**总结：**到此我们已经能够解决 Robbers 粉丝遇到的问题了，这也让我意识到随着反爬策略的升级，服务端可能会对爬虫最常用的第三方包进行请求指纹检测。同时也说明了，爬虫除了 Python，用 Go 其实也是一个不错的选择。

六、学习展望

笔者对 GoLang、Python 代码分别设置代理后，用 charles 抓包分析如下：

*   GoLang 设置代理发包
    

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoKtDcFaicA6qwZXvjnFFzicickp1XwnanW38hFAl6JcyNiarzk0DSibFVLAA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo6diaiaOWZwuNdk6Q42Dla2LyxCk6tEYLgBsUSEkX0icysDiawJnGPmXAcw/640?wx_fmt=png)

*   Python 设置代理发包
    

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoVMibguqfxTZZAsiaicNKSfHHhmTEPFkErxibx3nmbzjAwkWEXFmyn1KLUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoQlMtpXa0Vc9z8IBSBsyUbJvGmeyE9tkMMDicLfudzrJfzbibvqjltGqQ/640?wx_fmt=png)

我们对 GoLang、Python 两次发包后的请求参数对比分析，截图如下所示：

*   **GoLang 版本**
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4ykgD3coXpmnEdBeicknqdoRuy4j7xu8Via4YWCUUt0ibbY9moYnIhpNjLyicgAbkBd2NKWCTJhMbibVw/640?wx_fmt=jpeg)

*   **Python 版本**
    

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4ykgD3coXpmnEdBeicknqdo7xEgoQfn95gFQsVvkfxTLpncq4b5KHLY1euWVmLbUhxuUoIyHhaYEw/640?wx_fmt=png)

**总结：**本来想使用 charles 拦截请求包查看数据包有哪些区别，但是感觉使用 charles 查看不够直观，charles 应用中我们也得到了一些有用信息 (标红部分)，提取服务器的 ip+port 使用 **Wireshark** 打开查看，看看有没有更直观的信息吧。

使用 Wireshark 抓包，再次使用 go、Python 去发包，发包后根据 charles 获取的 ip 信息筛选 tls 指纹相关数据包，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7bGQPjmQVQichZiabKyQrIEkHXQzt4pInZUXYmkbk9lSyK1EvCk5GY1Zq7bDusYroQZPtic5JHkcmMA/640?wx_fmt=png)

紧接着我们点击如下按钮进行参数定位：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7bGQPjmQVQichZiabKyQrIEkZdwJlfXqxl8Epmr9lcHPzlytDdYx19Mw5kdTian74BAFOibofPQ99rHQ/640?wx_fmt=png)

将鼠标拉到最后，可以看到 tls 也就是 JA3 指纹如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7bGQPjmQVQichZiabKyQrIEk7NkYbaFGLmiaKiaBjEK652oEpF1bYXCGuCJCSjibthIhOPm4l04pZ4saA/640?wx_fmt=png)

整理 go、python 发包后的指纹文本对比如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7bGQPjmQVQichZiabKyQrIEkvTxF8tOkgtCyKoLLJ491XPJAkUEx9AXDsiaxM11FwGGicE1oibpEgmJsQ/640?wx_fmt=png)

**总结：**从上图可以看出两个请求包的 JA3 指纹加密算法不一致；如果我们还要继续使用 Python requests 去实现代码，可以尝试使用魔改 requests 修改 TLS 握手特征的代码去实现，也可以去阅读下 tls 指纹相关的文档。简单的网站是可以通过使用魔改 requests 修改 TLS 捂手特征代码通过，难度较大的就不能通过了，还需要新的方案去替代哦。这里突出一点，学无止境啊 ^_^⛽️

本篇分享到这里就结束了，欢迎大家关注下一期，我们不见不散☀️☀️😊

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A43BLL5j8ShhkUSRLT9ayEsmde5yxQmlKq8qZgsltoYaFpliaXKlmJUtNr6uZHibrn2xgIsQJUXu2nA/640?wx_fmt=png)

关注我们获得更多精彩内容

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7k7QiaRDzFktiaSbkClw0pXa6NV1Q9ge9a6D5nxGOojicqVQUQqQK0NOHg/640?wx_fmt=png)

_**END**_

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7zpB5TQPCHMeJVyX5BicWRibtHzfCIJvrJRAiaLC9akyJxXrfKVMnUS6rw/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691Wt6FFglTFeeibsPZT5F1vAiafn06J37WwvPkkGVX2B14Qh3gpPmic5Dpw/640?wx_fmt=gif)

    我是 **TheWeiJun**，有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友。

    分享日常学习中关于爬虫、逆向和分析的一些思路，文中若有错误的地方，欢迎大家多多交流指正☀️

![](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691PicricStHwRzqmIO1cPGTPsCk5SmfU2AZQLL2B6KSpxaHGguqZjXnjiaw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLwXGFXXOX978IvJSokfVzoN9L2EUB8bUAC0QqT1WJmckZfBfZJl3FFA/640?wx_fmt=gif)

**点分享**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLAWgUzHv0vx1STQhKykWTpicN12F4UdUeNXS1WsXYicqeBzmEUQbB3dAg/640?wx_fmt=gif)

**点收藏**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLJzbzyLgP8z7NIwqluryicesaw2PBoEPNvQ8K0jpyqn7AlA3vHGq1n3Q/640?wx_fmt=gif)

**点点赞**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLXdRibvzbwpGJb8wcTtFaUYTx1WXpUiaaD9TYy6Rk7jYhSwAL7c7BsSbA/640?wx_fmt=gif)

**点在看**