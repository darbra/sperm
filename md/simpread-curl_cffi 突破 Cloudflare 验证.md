> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/x5sHHONqsYq2NiLKSKMgKg)

 Cloudflare

WAF

 Cloudflare 是一种基于云技术的 Web 应用程序防火墙（WAF），旨在保护网站免受各种 Web 攻击，它能够在 5 秒内检测到并阻止恶意流量。

现在很多 JW 站点都逐步应用 Cloudflare WAF, 导致采集开发成本日益剧增。

本文涉及场景：通过 Python 采集程序访问站点页面时，会跳转至《Cloudflare WAF 連線錯誤頁面》，并提示拒绝访问。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xxA783bXQVuDUtWYWaaTedSWciaE53sLLtiaIs9tTiboVoqjl7z1uxqWPGc2W8hf6VaFq1MuolQ1gqG3GbO4nzdaA/640?wx_fmt=png&from=appmsg)

然而在服务器中通过 Curl 命令可以正常访问。

根据猜测，大概率是请求指纹被识别后拦截。

Tls/Ja3

我尝试随机生成请求库的 JA3 指纹，但是并没有生效，依旧被拦截到。

经过检索，找到了名为 curl-impersonate 的开源项目，通过它可模拟四种浏览器 Chrome、Edge、Safari 和 Firefox，执行与真实浏览器相同的 TLS 和 HTTP 握手。(通过将 curl 中的组件全部替换为浏览器使用库，并且让版本保持一致，从而使 curl 的指纹和浏览器一致）

更多内容大家自行查看，下面说一下基于 curl-impersonate 的 Python 开源库 curl_cffi。

https://github.com/yifeikong/curl_cffi

curl_cffi

curl_cffi 可模拟真实浏览器的 TLS | JA3 指纹。

直接 pip install curl_cffi 安装即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xxA783bXQVuDUtWYWaaTedSWciaE53sLLQYuAs5WggC1yia0K2dcaZQAtncJFQhWo7sockmLv5MIibmg85vaLBsOQ/640?wx_fmt=png&from=appmsg)

使用非常简单，注意 impersonate 填写的版本即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xxA783bXQVuDUtWYWaaTedSWciaE53sLLK4JSfrz0u6xHztVyAZHwb0vbD57SGhhg0DBFdu378BeoJQyYEWdziaA/640?wx_fmt=png&from=appmsg)

也可以支持 requests.Session()

下面是两个用于测试的站点，大家可自行体验：

chinatimes.com

boxun.com

```
from curl_cffi import requests
headers={
    "user-agent":'Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1',
}
def chinatimes():
    sess = requests.Session()
    # 先请求一次取__cf_bm
    sess.get('https://www.chinatimes.com',headers=headers,impersonate="chrome99_android")
    # 第二次携带__cf_bm请求
    d = sess.get('https://www.chinatimes.com/realtimenews/20231130000049-260407',headers=headers,impersonate="chrome99_android")
    print(d.text)

```

项目地址：https://github.com/yifeikong/curl_cffi

实际情况需要大家自行测试，此方案相对于未有其他限制的新闻站点效果尚可，并不适用于所有 cf 站点的访问。