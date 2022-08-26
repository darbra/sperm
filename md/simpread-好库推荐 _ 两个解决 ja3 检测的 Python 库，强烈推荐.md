> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2kZHmQpGkq89wNaz5xBwyw)

**关注它，不迷路。**

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！

某佬丢过来一个网站，我用 requests 库请求会报错:

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1Zbneyt19g2s7OyBTpq3cDNQYNLLDtib0ZtSDHXm6TNLYCk1o6ehAY95e6ECW9yAbQ3iaFgjcsTYNw/640?wx_fmt=png)

先说下我的环境: Win10 + Python 3.10 + requests  2.27.1, 直接请求的话报错了，我猜测是检测了 tls，听说降低 Python 和 requests 的版本可以正常请求，或许检测不那么严格。

下面介绍两个库，过掉它的检测。  

1. Pyhttpx
----------

项目地址:

```
https://github.com/zero3301/pyhttpx

```

你可以将整个项目下载下来，也可以直接安装这个库:  

```
pip install pyhttpx

```

我选择的是将整个项目下载下来测试，根据他的 demo，写下请求代码:

```
import pyhttpx
sess = pyhttpx.HttpSession()
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36",
}
url = "打码了"
response = sess.get(url)
print(response.text)

```

可以正常返回了:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1Zbneyt19g2s7OyBTpq3cDcECeRoZJbZaxmuriaYibJQTRAQqcn5odhBxTNGyqCwHO2c6GxBDdqiaBw/640?wx_fmt=png)

完美解决！

这个项目的优秀之处在于可以修改成指定的 ja3 加密套件，还能在 Windows 下运行，非常的 nice！当然我在测试某些网站的时候报错了，项目还不是那么完美，期待解决。

2. Pycurl
---------

目前这个库在 Windows 上还没办法解决 ja3 的问题，因此我选择了 Liunx.

环境:  

```
Distributor ID:  Ubuntu
Description:  Ubuntu 18.04.4 LTS
Release:  18.04
Codename:  bionic

```

如何安装并解决 ja3，可以参考华哥的这篇文章 [python 完美突破 tls/ja3](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484904&idx=1&sn=cfdc0fe3cbce3c4f2662eb480c654e5c&chksm=fb18f44acc6f7d5cbc680d6ffe8be2e844d492faff5a105437dcc911c260854b4ef3535d7c44&scene=21#wechat_redirect)  

也可以参考我在星球里写的:  

```
https://t.zsxq.com/052z3rzN3

```

如果你在服务器上输入下面的命令可以正常返回，说明安装成功了.

```
curl_chrome100 https://www.baidu.com

```

把下面的请求 demo 代码上传到服务器:  

```
import pycurl
import copy
from io import BytesIO
import re
import io
import random
class Response:
    def __init__(self, status_code, body, headers):
        self.status_code = status_code
        self.body = body
        self.headers = headers
    @property
    def text(self, encode="utf-8"):
        return self.body.decode(encode)
class CurlClient:
    def __init__(self):
        c = pycurl.Curl()
        # 自动维护cookie
        c.setopt(pycurl.COOKIEFILE, "")
        c.setopt(pycurl.TIMEOUT, 30)
        # 开启alpn
        c.setopt(pycurl.SSL_ENABLE_ALPN, 1)
        c.setopt(pycurl.SSL_ENABLE_NPN, 0)
        # 跳转
        c.setopt(pycurl.FOLLOWLOCATION, 1)
        # 处理gzip
        c.setopt(pycurl.ENCODING, "gzip,deflate")
        # 是否验证ssl
        c.setopt(pycurl.SSL_VERIFYPEER, 1)
        c.setopt(pycurl.SSLVERSION, pycurl.SSLVERSION_TLSv1_2)
        try:
            c.setopt(pycurl.SSL_CERT_COMPRESSION, "brotli")
            c.setopt(pycurl.SSL_ENABLE_ALPS, 1)
        except:
            pass
        # 设置代理
        # c.setopt(pycurl.PROXY, "http://127.0.0.1:9091")
        # c.setopt(pycurl.PROXY, "http://127.0.0.1:7890")
        # 加密套件
        c.setopt(
            pycurl.SSL_CIPHER_LIST,
            "TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256,ECDHE-ECDSA-AES128-GCM-SHA256,ECDHE-RSA-AES128-GCM-SHA256,ECDHE-ECDSA-AES256-GCM-SHA384,ECDHE-RSA-AES256-GCM-SHA384,ECDHE-ECDSA-CHACHA20-POLY1305,ECDHE-RSA-CHACHA20-POLY1305,ECDHE-RSA-AES128-SHA,ECDHE-RSA-AES256-SHA,AES128-GCM-SHA256,AES256-GCM-SHA384,AES128-SHA,AES256-SHA",
        )
        self.c = c
        self._cookies = {}
    def get(self, url, headers=[]):
        self.c.setopt(pycurl.POST, 0)
        r = self.send(url, headers)
        return r
    def post(self, url, data, isjson=False, headers=[]):
        self.c.setopt(pycurl.POST, 1)
        self.c.setopt(pycurl.POSTFIELDS, data)
        h = copy.deepcopy(headers)
        if isjson:
            h.append("Content-Type: application/json")
        r = self.send(url, h)
        return r
    def send(self, url, headers):
        body = BytesIO()
        resp_header = BytesIO()
        h = self.default_headers
        h.extend(headers)
        self.c.setopt(pycurl.HTTPHEADER, h)
        self.c.setopt(pycurl.URL, url)
        self.c.setopt(pycurl.WRITEDATA, body)
        self.c.setopt(pycurl.WRITEHEADER, resp_header)
        self.c.perform()
        r = Response(
            self.c.getinfo(pycurl.HTTP_CODE),
            body.getvalue(),
            resp_header.getvalue().decode(),
        )
        self.save_cookies(resp_header.getvalue().decode())
        return r
    def save_cookies(self, resp_header):
        cookies = re.findall("Set-Cookie: (.*?);", resp_header, re.IGNORECASE)
        for cookie in cookies:
            k, v = cookie.split("=", 1)
            self._cookies[k] = v
    def set_proxy(self, proxy):
        self.c.setopt(pycurl.PROXY, proxy)
    @property
    def cookies(self):
        return self._cookies
    @property
    def default_headers(self):
        h = [
            'sec-ch-ua: ".Not/A)Brand";v="99", "Microsoft Edge";v="103", "Chromium";v="103"',
            "sec-ch-ua-mobile: ?0",
            'sec-ch-ua-platform: "Windows"',
            "upgrade-insecure-requests: 1",
            "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.66 Safari/537.36 Edg/103.0.1264.44",
            "accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
            "sec-fetch-site: none",
            "sec-fetch-mode: navigate",
            "sec-fetch-user: ?1",
            "sec-fetch-dest: document",
            "accept-encoding: gzip, deflate, br",
            "sccept-language: en-US,en;q=0.9",
        ]
        return h
def test_curl():
    c = CurlClient()
    resp = c.get("打码了")
    print(resp.text)
if __name__ == "__main__":
    test_curl()

```

运行后，也返回了正常的数据 l  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1Zbneyt19g2s7OyBTpq3cDficdDsyBibN0ic8J5PUrIbic7bk8pxBhVyYyic4Xiayibaa5rYicdFhddjbejg/640?wx_fmt=png)

今天的文章就分享到这里，后续分享更多的技巧，敬请期待。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Lll8tx0MDR0xtvs5q4zuW5BXvXzbAAAdAxXH7PSebBWJT3U9dXG1XtOSKRVDQqictGWKznl3rusg5MAsGO0D6Lw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

欢迎加入知识星球，学习更多 AST 和爬虫技巧。