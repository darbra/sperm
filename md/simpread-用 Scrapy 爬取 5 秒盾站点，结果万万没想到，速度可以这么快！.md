> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4OxCgqNirOPFrSvRY_7aRw)

  

大家好，我是 TheWeiJun，欢迎来到我的公众号。在新的一年里，期待大家在技术征途上不断突破，获得更多的成就。今天，我将与大家分享一段令人振奋的故事，通过对 Scrapy 爬虫的 twisted 源码高并发改造，成功冲破 5 秒盾站点的屏障。让我们一同解锁这个技术谜团，探索爬虫世界的无限可能。祝愿大家在 2024 年迎来更多快乐，事业腾飞！

  

**特别声明：**本公众号文章只作为学术研究，不作为其他不法用途；如有侵权请联系作者删除。

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A6N3l9obtATicYg0vSreUhmEsiciaibJfHjG8MEHndnoJycRKpgeeK3LkxKu6qlz7oyC2Gelexa4W4Bfw/640?wx_fmt=gif)

立即加星标

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6N3l9obtATicYg0vSreUhmEG7mwRfEUOichNPSVJkJl4IDcjeqvJyO5GGxL9CaYcRQqg13wjyXVzicQ/640?wx_fmt=png)

每月看好文

 目录

  

一、前言介绍

二、实战分析

三、源码重写

四、性能对比

五、往期推荐

  

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaawtmicxxynT0xDSjtEBPXy4ibgtWWibh69qBVIeuYlGgWPBV2dm8cosZFxicAgZShfCRQ3u9ZuFbuPicQ/640?wx_fmt=gif)

一、前言介绍

大家好，我是 TheWeiJun。2024 年已来临，我怀揣着对技术的热爱，迫不及待要与大家分享一场关于 Scrapy 爬虫的技术奇遇。在这个数字化飞速发展的时代，我们时刻面临新的技术挑战。在今天的故事中，我将引领大家穿越 Scrapy 的技术迷雾，通过 twisted 源码改造，实现高并发爬取，成功攻克五秒盾站点的技术难关。

如果你对技术探险充满好奇，渴望突破技术的边界，不妨关注我的公众号。在这里，我们将一同探索未知领域，共同启程，为 2024 年的技术征途揭开崭新的篇章！记得点关注，一同踏上这场技术冒险之旅吧！

二、实战分析  

1. 首先，我们需要寻找一个使用了 CloudFlare 的网站。然后，创建一个 Scrapy 项目，并编写以下 Spider 代码：

```
# -*- coding: utf-8 -*-
from urllib.parse import urlencode
import scrapy
class CloudflareSpider(scrapy.Spider):
    name = 'cloudflare_spider'
    def __init__(self):
        super().__init__()
        self.headers = {
            'authority': 'xxxxx',
            'accept': 'application/json, text/plain, */*',
            'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8',
            'cache-control': 'no-cache',
            'pragma': 'no-cache',
            'referer': 'https://xxxxx/feed',
            'sec-ch-ua': '"Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"',
            'sec-ch-ua-mobile': '?0',
            'sec-ch-ua-platform': '"macOS"',
            'sec-fetch-dest': 'empty',
            'sec-fetch-mode': 'cors',
            'sec-fetch-site': 'same-origin',
            'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
        }
        self.url = 'https://xxxx/xxx/xxx/'
    def start_requests(self):
        for page in range(1, 100):
            params = {
                'page': page,
                'posts_to_load': '5',
                'sort': 'top',
            }
            proxies = {
                "http": "http://127.0.0.1:59292",
                "https": "http://127.0.0.1:59292",
            }
            full_url = self.url + '?' + urlencode(params)
            yield scrapy.Request(
                url=full_url,
                headers=self.headers,
                callback=self.parse,
                dont_filter=True,
                meta={"proxies": proxies}
            )
    def parse(self, response, **kwargs):
        print(response.text)

```

2. 代码编写完成后，让我们一起来查看整个 Scrapy 项目的结构。以下是项目目录结构的截图：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RxRZW3pBbCHAjW9YbsuaAWiaCGavHYsicibETHKZLqlpsGmw37blfN0nAA/640?wx_fmt=png&from=appmsg)

3. 接着，我们编写 run_spider.py 文件，并在其中注册我们想要启动的 Spider（使用 spider_name 变量）以下是代码示例：

```
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings
def run_spider():
    process = CrawlerProcess(get_project_settings())
    process.crawl('cloudflare_spider')
    process.start()
if __name__ == "__main__":
    run_spider()

```

4. 通过 run_spider.py 模块运行爬虫，可以看到 403 状态码错误请求，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RXPxjKrf1HXQficAgBuRHAImq4fcqmlIDGBHac7nPeRZ76cKdJiamSibaw/640?wx_fmt=png&from=appmsg)

5. 此时，Scrapy 的 parse 解析函数可能无法获取到失败的 response，因为 Scrapy 默认只处理状态码在 200 范围内的请求。为了能够查阅失败的请求结果，我们需要设置允许通过的状态码参数。以下是相应的代码设置：

```
HTTPERROR_ALLOWED_CODES = [403]

```

  
6. 接下来讲解一下为什么要这么设置? 我们打开 scrapy 源码，截图如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RibMG12k9e0RuIVkiagbeGsBrGTJy42ryztLl8f6foqoN6j8QdMowSAlA/640?wx_fmt=png&from=appmsg)

7. 在 spidermiddlewares 中间件中，我们可以观察到 HttpErrorMiddleware 模块。当 Scrapy 启动时，各个模块会被注册到 spidermiddlewares 中间件。现在让我们深入了解它是如何运行的，以下是相关代码截图：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RreyJhLg5oWGJc9Fibz7vicNkeaIR7D46SXosUere6uIQeUcUSoyLkqVw/640?wx_fmt=png&from=appmsg)

**总结：**观察上述代码，我们可以注意到 Scrapy 的作者默认会过滤掉状态码在 200 以内的请求，因为在作者看来，以 200 开头的请求都是成功的。然而，如果我们想要自定义允许通过的请求状态码，就需要设置 HTTPERROR_ALLOWED_CODES。

8. 我们知道 spider 中间件原理并设置 403 状态码后，重新运行代码，截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RZKh0JyZISSa0GM5mKoItDzEgGibs4bpNfdBkIvxOfzVBdFeLsb0QJ4Q/640?wx_fmt=png&from=appmsg)

**总结：**这张页面截图对于那些已经接触过 5 秒盾的 Spider 开发者来说应该不陌生。接下来，我们将使用 tls_client 包来绕过 5 秒盾机制。  

9. 此外，大家应该都使用过 download middlewares 中间件。下面我们将在下载器中间件中处理 5 秒盾请求，相关代码如下：  

```
from scrapy.http import HtmlResponse
from tls_client import Session
class DownloaderMiddleware(object):
    def __init__(self):
        self.session: Session = Session(
            client_identifier="chrome_104"
        )
    def process_request(self, request, spider):
        proxies = request.meta.get("proxies") or None
        headers = request.headers.to_unicode_dict()
        if request.method == "GET":
            response = self.session.get(
                url=request.url,
                headers=headers,
                proxy=proxies,
                timeout_seconds=60,
            )
        else:
            response = self.session.post(
                url=request.url,
                headers=headers,
                proxy=proxies,
                timeout_seconds=60,
            )
        return HtmlResponse(
            url=request.url,
            status=response.status_code,
            body=response.content,
            encoding="utf-8",
            request=request,
        )

```

10. 将上面的模块注册到下载器中间件后，我们启动爬虫观察请求结果，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2Rbp68K5Vd9YzjHWXkdygdMDAn7KL0n2PZwI3yvhNklHsDqx2IYpEYKA/640?wx_fmt=png&from=appmsg)

**总结：**尽管 5 秒盾已经能够成功解决并返回结果，我们却发现 Scrapy 并没有充分发挥 Twisted 的异步机制。这是因为我们在下载器中间件中处理请求时，实际上是在同步的环境下运行的。如果我们希望 Scrapy 能够实现高并发，就必须修改 Twisted 的请求模块。我们可以通过重写 Twisted 请求组件或者兼容 tls_client 模块来实现高并发。在这里，我们选择后者的方式，以达到 Scrapy 高并发的目标。接下来，我们将进入源码重写的环节。

三、源码重写

1. 首先，我们来了解一下 Scrapy 的运行机制，然后找到相应的模块，并查看 Scrapy 源码的实现。以下是相应的截图：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2Ric8kNRQsPDU696AP2xSFplMsa8oqxwEqSzq7PhyB4qZQgABcb8KqGjg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RyTS26yphUIraLSopIdRlpxN2GmKicOcC0pM5tk5kwS5vux1nEhicic89g/640?wx_fmt=png&from=appmsg)

**总结：**在 Scrapy 启动时，它通过 downloader_handlers 中的 download_request 方法加载 Twisted 模块，从而进行请求的异步处理。一旦我们获得了灵感，就可以开始继承并重写 Scrapy 的源码。

2. 我们继承重写 downloader hanlders 中的模块，重写后源码如下：

```
"""Download handlers for http and https schemes"""
import logging
from time import time
from urllib.parse import urldefrag
from tls_client import Session
from twisted.internet import threads
from twisted.internet.error import TimeoutError
from scrapy.http import HtmlResponse
from scrapy.core.downloader.handlers.http11 import HTTP11DownloadHandler
logger = logging.getLogger(__name__)
class CloudFlareDownloadHandler(HTTP11DownloadHandler):
    def __init__(self, settings, crawler=None):
        super().__init__(settings, crawler)
        self.session: Session = Session(
            client_identifier="chrome_104"
        )
    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings, crawler)
    def download_request(self, request, spider):
        from twisted.internet import reactor
        timeout = request.meta.get("download_timeout") or 10
        # request details
        url = urldefrag(request.url)[0]
        start_time = time()
        # Embedding the provided code asynchronously
        d = threads.deferToThread(self._async_download, request)
        # set download latency
        d.addCallback(self._cb_latency, request, start_time)
        # check download timeout
        self._timeout_cl = reactor.callLater(timeout, d.cancel)
        d.addBoth(self._cb_timeout, url, timeout)
        return d
    def _async_download(self, request):
        timeout = int(request.meta.get("download_timeout"))
        proxies = request.meta.get("proxies") or None
        headers = request.headers.to_unicode_dict()
        if request.method == "GET":
            response = self.session.get(
                url=request.url,
                headers=headers,
                proxy=proxies,
                timeout_seconds=timeout,
            )
        else:
            response = self.session.post(
                url=request.url,
                headers=headers,
                proxy=proxies,
                timeout_seconds=timeout,
            )
        return HtmlResponse(
            url=request.url,
            status=response.status_code,
            body=response.content,
            encoding="utf-8",
            request=request,
        )
    def _cb_timeout(self, result, url, timeout):
        if self._timeout_cl.active():
            self._timeout_cl.cancel()
            return result
        raise TimeoutError(f"Getting {url} took longer than {timeout} seconds.")
    def _cb_latency(self, result, request, start_time):
        request.meta["download_latency"] = time() - start_time
        return result

```

**总结：**源码重写工作已经圆满完成，此时我们迫不及待地期待着 Scrapy 在高并发环境下的表现。怀揣这个疑问，让我们迅速进入性能对比环节。在进行最后的步骤时，请确保将重写的代码注册到 DOWNLOAD_HANDLERS 中间件模块。

四、性能对比

为了进行性能对比，我们按照以下规则进行测试：

*   执行 100 个请求的情况下使用下载器中间件方案。
    
*   执行 100 个请求的情况下使用 Twisted 源码重写方案。
    
*   执行 500 个请求的情况下使用下载器中间件方案。
    
*   执行 500 个请求的情况下使用 Twisted 源码重写方案。
    

1. 阅读完对比流程后，我们先执行下载器中间件方案，scrapy 输出日志如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RfmH6FFTHRt6niaSzFXsdc2YQcr8bUtTic2vOW9DEPqyh1Naic2d2oiab4Q/640?wx_fmt=png&from=appmsg)

2. 接着，在相同的环境中，执行源码重写方案，Scrapy 输出的日志如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RJZYTkqqaVoqw9wwDcm3hvMpTiac2n0dvR39fFZLmfGBqiaXxv7dhsLGw/640?wx_fmt=png&from=appmsg)

**总结：**通过对比两张截图的 elapsed_time_seconds 字段，明显可以观察到 Scrapy Twisted 源码重写方案在执行 100 次请求时，爬取速度提升了 6 倍。为了确保性能对比的权威性，接下来我们将分别执行 500 次请求。

3. 在执行 500 次请求时，仍然首先采用下载器中间件方案，Scrapy 输出的日志如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2R3WFrNYbich0SGLr7PsS1U999qTd3Y3PRNAPtlVLpI7rwib9RdY0nk90w/640?wx_fmt=png&from=appmsg)

4. 紧接着，我们执行 500 次请求，采用 twisted 源码重写方案，Scrapy 输出的日志如下：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2RnxWdqLwKh15Mnap7oouqV0mRddTsF9AUTh5ktNKp1U1ibEwGiaghgypQ/640?wx_fmt=png&from=appmsg)

**总结：**通过比较 500 次请求的两张截图，我们可以观察到，在 elapsed_time_seconds 方面，Scrapy Twisted 源码重写方案明显优于下载器中间件方案。在同时执行 500 次请求的情况下，爬取速度提升约为 9 倍。基于这个结果，我相信在请求量足够大的场景下，采用 Scrapy Twisted 源码重写方案能够显著提升爬取效率。

五、往期推荐

本篇文章分享到这里就结束了，欢迎大家关注下期文章，我们不见不散![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2R5eNiaJWETUDCKtBHcZL5T8kbPWd3cuXiaHPeGdxTAjWBBw9p1nEQslcg/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2R5eNiaJWETUDCKtBHcZL5T8kbPWd3cuXiaHPeGdxTAjWBBw9p1nEQslcg/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2R5eNiaJWETUDCKtBHcZL5T8kbPWd3cuXiaHPeGdxTAjWBBw9p1nEQslcg/640?wx_fmt=png&from=appmsg)  
文章结尾给大家准备了彩蛋，可以领取微信红包，提前祝大家新春快乐![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2Rx7woGMkyKmrnttPphTEB2IqMicM5aYw1Wa5NEIpYZ3C9bwsFWzAePqg/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2Rx7woGMkyKmrnttPphTEB2IqMicM5aYw1Wa5NEIpYZ3C9bwsFWzAePqg/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6yelSOV4CAGF5TMxHUxr2Rx7woGMkyKmrnttPphTEB2IqMicM5aYw1Wa5NEIpYZ3C9bwsFWzAePqg/640?wx_fmt=png&from=appmsg)  
**粉丝福利：**公众号后台回复 **888** 即可获取 github 完整开源仓库。

往期推荐

[

某云滑块验证码别乱捅！一不小心就反爬了。



](https://mp.weixin.qq.com/s?__biz=Mzg5ODA3OTM1NA==&mid=2247491688&idx=1&sn=3c7c98709ad36032f8b8ade92f7881c2&chksm=c06aaf7df71d266b77999518c53af8dbb04b44d431d9aa70852257b95cd973e402475b944b08&scene=21#wechat_redirect)

[

某美滑块验证码别乱捅！一不小心就反爬了。



](https://mp.weixin.qq.com/s?__biz=Mzg5ODA3OTM1NA==&mid=2247491526&idx=1&sn=55c3085d1052c169f29d42f384e8d16e&chksm=c06950d3f71ed9c5986b4653c11db2662fdf3781666899be6abc114703acbf8d734ee5eae3cc&scene=21#wechat_redirect)

[

深入探索 Go 语言 net/http 包源码：从爬虫的视角解析 HTTP 客户端



](https://mp.weixin.qq.com/s?__biz=Mzg5ODA3OTM1NA==&mid=2247490768&idx=1&sn=c02b065b8e4a96904833d3b80b62835c&chksm=c06953c5f71edad384aedb70b911b2d43f940c74ec821a6c86c0c5da3cdca785130ed7815714&scene=21#wechat_redirect)

[

探秘迷雾背后：逆向短视频弹幕系统的奇妙之旅



](https://mp.weixin.qq.com/s?__biz=Mzg5ODA3OTM1NA==&mid=2247490764&idx=1&sn=3ffe639940160433ffa3e5195a127997&chksm=c06953d9f71edacf0d8f168f5886c7c4e4690e5209677e10bebdc214043f24ffe540544daa1b&scene=21#wechat_redirect)

[

DX 滑块验证码别乱捅！一不小心就反爬了。



](https://mp.weixin.qq.com/s?__biz=Mzg5ODA3OTM1NA==&mid=2247489011&idx=1&sn=3244b1d79fdfe4c7fb6f858d0576bc8b&chksm=c0695ae6f71ed3f0db05a936a3692d2013c85a55a058fef4a4c500e2705eae739f18f13e4c46&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A47tBINwSQml9LBbhoHC1zXx7gzfJ3gvfXgbCervO9niaprHP6aiae9Dl7icz7SCsp9qDfkchax1Exsw/640?wx_fmt=gif)

龙年大吉

  

**  
如果想要获得更多精彩内容可以关注我朋友:**

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7k7QiaRDzFktiaSbkClw0pXa6NV1Q9ge9a6D5nxGOojicqVQUQqQK0NOHg/640?wx_fmt=png)

_**END**_

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7zpB5TQPCHMeJVyX5BicWRibtHzfCIJvrJRAiaLC9akyJxXrfKVMnUS6rw/640?wx_fmt=png)

  

  

  

  

  

  

作者简介  

  

  

我是 **TheWeiJun**，有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎扫我微信与我交朋友💕

分享日常学习中关于爬虫及逆向分析的一些思路，文中若有错误的地方，欢迎大家多多交流指正💕

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4QPIsiaD2F0kQdBViby9s1UsyIkT8kzGfPRzeVicUqrcPJgjI1JcYrmukxRTbYR2VxH5zrx09ct5TEg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLwXGFXXOX978IvJSokfVzoN9L2EUB8bUAC0QqT1WJmckZfBfZJl3FFA/640?wx_fmt=gif)

**点分享**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLAWgUzHv0vx1STQhKykWTpicN12F4UdUeNXS1WsXYicqeBzmEUQbB3dAg/640?wx_fmt=gif)

**点收藏**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLJzbzyLgP8z7NIwqluryicesaw2PBoEPNvQ8K0jpyqn7AlA3vHGq1n3Q/640?wx_fmt=gif)

**点点赞**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLXdRibvzbwpGJb8wcTtFaUYTx1WXpUiaaD9TYy6Rk7jYhSwAL7c7BsSbA/640?wx_fmt=gif)

**点在看**