> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7wU8almDyjwXeEvVbEWgvw)

前言
--

这次和紫星大佬组队，获得了猿人学 - Android 端爬虫比赛的第一名。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTvnxgiaq62wBfpTHoV5mvj4pE4uI5rKwkkaatZIUEB9E4x5IuRxNibM5g/640?wx_fmt=png)

在此先说一句：紫星大佬牛逼！

比赛链接：https://appmatch.yuanrenxue.com/

获取 key
------

第五题是双向认证，拿出珍藏的脚本 tracer-keystore.js 试试。

```
function hookKeystoreGetInstance() {
    var keyStoreGetInstance = Java.use('java.security.KeyStore')['getInstance'].overload("java.lang.String");
    keyStoreGetInstance.implementation = function (type) {
        //console.log("[Call] Keystore.getInstance(java.lang.String )")
        console.log("[Keystore.getInstance()]: type: " + type);
        var tmp = this.getInstance(type);
        keystoreList.push(tmp); // Collect keystore objects to allow dump them later using ListAliasesRuntime()
        return tmp;
    }
}



```

下载地址：https://github.com/FSecureLABS/android-keystore-audit/blob/master/frida-scripts/tracer-keystore.js

打开 app，点击第五题，就出来 bks 证书的密码了。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTX5yOEiavESLYtD31a3AkNMcWGJI3BCzuhciaq2YwfBicthYk29OXGU75g/640?wx_fmt=png)

bks 到 p12 的转换
-------------

接着打开神器 keystore-explorer，进行 bks 到 p12 的转换。

下载链接：https://keystore-explorer.org/downloads.html

打开 clientCA.bks

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTVMhd4KtibGcbW3c2h7RPUmdwAdsL1VmxfFtNj2pND3xTPTZMg5LsiaicA/640?wx_fmt=png)

输入前面 hook 到的密码

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTL3y6Nxc2r0Yprg3vJAAl1g2Uzs7YpYHlmXOxqTu8fDnVawabDibrDIw/640?wx_fmt=png)

转成 p12

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTe5HaIku3lXjy1IdLX7zfZFkhxnD35bl9hRaJMICwZzNRv3US1VicGEw/640?wx_fmt=png)

导出证书

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTvM704Zr1a2AWtFc0K4FI2qPpCD1bia2K2Ejna4icLMYFoQBMFyCaJomw/640?wx_fmt=png)

抓包
--

效仿以前抓 soul 包的方式，将较早之前生成的 p12 证书导入 charles。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTuibqrUEyHhRyrnfu7LRXmibCL09GVGSvafzSh8n5yic4LULVjlQQnARDg/640?wx_fmt=png)

发现提示密码错误。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTt0kNT739tG5OibhW7rI9tnG0hGKLicPzYwSsOw5l2r9JxYV6DGUADAmw/640?wx_fmt=png)

【PS: 就在我还在对密码错误怀疑人生的时候，紫星巨佬说到：为啥一定要抓包？然后他就搞出结果了。。】

那换种思路，用神器 r0capture 试试。

下载链接：https://github.com/r0ysue/r0capture

运行神器后，请求流程自吐了出来。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTfb7ZT231eC7Jq6CwKMBy7jTicH6Fr9XEPapVjd2ic3mhAFV6qnOvTrew/640?wx_fmt=png)

POST 请求

url 是 *.*.*.*:*/api/app5

Content-Type 是 application/x-www-form-urlencoded

User-Agent 是 okhttp/3.14.9

data 是 page=1

脚本书写
----

带上之前转化成功的 p12 证书，构造刚刚得到的请求，结果就呼之欲出了。

```
import requests_pkcs12

def get_page(page):
    url = 'https://*.*.*.*:*/api/app5'
    hd = {
        'Content-Type':'application/x-www-form-urlencoded',
        'user-agent': 'okhttp/3.14.9'
    }
    data = {
        'page': page
    }
    resp = requests_pkcs12.post(url, 
                headers=hd, data=data, pkcs12_filename='1.p12', 
                pkcs12_password='**********', verify=False)
    print(resp.json())

get_page(1)


```

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1YxPOT8Etich3jQREXTgYXXTNCVwCGiaQPibFqhq0GhSSKL1MHgibico033PnLnAxMLzYJ6ib9jOvhB6emw/640?wx_fmt=png)

答案呼之欲出了。

队友紫星大佬的 52pojie：https://www.52pojie.cn/home.php?mod=space&uid=358970

紫星大佬的 github：https://github.com/zixing131

本人收藏了很多优秀文章：https://github.com/darbra/sperm

本人还有些可复现的案例：https://github.com/darbra/sign