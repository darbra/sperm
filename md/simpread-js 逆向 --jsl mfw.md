> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDk2MDQ4OQ==&mid=2247483850&idx=1&sn=ccbe2b93cc45906aef8a2293a76423b1&chksm=9ae1b34cad963a5ab87375749cee5065a7e667a22e791eb931038fcc3511c77062cb0e2673c3&mpshare=1&scene=1&srcid=0810QVhIMUyvDDtCmaFcVwed&sharer_sharetime=1660095610511&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

**声明**

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！由于本人水平有限，如有理解或者描述不准确的地方，还望各位大佬指教！！

**目标网站**

aHR0cDovL3d3dy5tYWZlbmd3by5jbi8=

**抓包分析**

好久没更新了。被生活忙的焦头烂额，恰巧在工作中遇到加速乐的案例，觉得逻辑挺不错的。

直接上逻辑：  

同一个页面，请求了三次才成功，前面两次状态码都是返回的 521

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UCugmfiaicbZcwymdSJ2cU4yZicIDXO2gMts0efmJn69Wqx2jpokdWRicQg/640?wx_fmt=jpeg)

第一次 521 请求返回一个 cookie(__jsluid_h) , 并且从返回的 script 代码中可直接提取出请求第二次请求所需的 cookie(__jsl_clearance1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6U2n86NNcqDTSjTudou6XxgmXQ2n8pfV8EMIfIk3mG1FVicrY3F7mgwFA/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UNjKEgZRgVzOccMiadlsyUSn9ASRqibsrv24yJsnwV6YxoT3ZXAbZB7RQ/640?wx_fmt=jpeg)

第二次 521 请求需要的 cookie 为__jsluid_h1 和__jsl_clearance1, 从返回的 js 代码中加密得到第三次所需的__jsl_clearance2

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UtZc21ST4t3Rdu2icffILHTWawnO0K9OJhzW4Bv14Kib7DyZETcKCsRhw/640?wx_fmt=jpeg)

第三次 200 请求携带的 cookie 为__jsluid_h1 和__jsl_clearance2

**加密逻辑**

第一次请求返回的结果如下

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UP2BheVz1IwVsGYuHh4YTCTvwfYfIAiac8YYUnpTtWicZoy41aySMRzlw/640?wx_fmt=jpeg)

使用正则取出 document.cookie 的值，执行一遍，返回第二次请求所需的 cookie(__jsl_clearance1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UibmS44wd93PXlXTzdmYzdJLpKk2jlCcMABIeSrAeu44Un7zLYeJMXVA/640?wx_fmt=jpeg)

第二次请求返回的结果如下

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6UvM8yPAuUicicFIow91ManCicurC8wZxmkUqXekW3SrYmZ2PZh4ecMqOBg/640?wx_fmt=jpeg)

可以看出是 ob 混淆，可以选择使用 ast 解混淆，这里我选择补环境不选择扣算法，不需要解混淆。我们需要对第二次请求的结果进行小处理。在开头补上缺少的环境，即可得到_jsl_clearance_s。

这里我给大家送上通杀大部分加速乐的环境。

```
var window = {
    addEventListener: function() {
    },
    navigator: {
        userAgent: ''
    }
}
var document = {
    addEventListener: function(x, x1, x2) {
        if (x == 'DOMContentLoaded') {
            x1();
        }
    },
    attachEvent: function(x, x1) {
        if (x == 'onreadystatechange') {
            x1();
        }
    },
    cookie: ""
}
var location = {
    href: ''
}
var setTimeout = function(x,x1){} // 可以不用 

```

接下来就是携带第二次请求生成和返回的 Cookie 去请求第三次

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/RJ5yax4pZianY6GJN7CicIClgo8iab6MW6Uia5PHKEjTDdMiaTCRnPvjiaumhN8mcVJmJsEZYQ6tU6HOfwjgpVbbd2aQ/640?wx_fmt=jpeg)

**完整代码：**

```
import requests
import re
import execjs
url = "https://www.mafengwo.cn/"
headers = {
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36"
}
cookie_dict = {}
res_512 = requests.get(url, headers=headers)
print(res_512.text)
if res_512.status_code == 521:
    __jsluid_h = re.findall('__jsluid_s=(.*?);', res_512.headers.get('Set-Cookie'))[0]
    cookie_dict['__jsluid_s'] = __jsluid_h
    js_clearance = re.findall('cookie=(.*?);location', res_512.text)[0]
    js_result = execjs.eval(js_clearance)
    cookie_dict['__jsl_clearance_s'] = js_result.split("=", 1)[1]
    cookie_str = "".join([x+"="+cookie_dict[x]+";" for x in cookie_dict.keys()])
    headers.update({'Cookie': cookie_str})
    # print(headers)
    res_512_next = requests.get(url, headers=headers)
    print(res_512_next.text)
    replace_re = re.findall("(var \S+?=\S+\['ct'\],.+?\)\]\);)", res_512_next.text)
    need_cookie_name = replace_re[0].split("=")[0].split("var")[1].strip()
    eval_js = res_512_next.text.replace(replace_re[0], replace_re[0]+"return {};".format(need_cookie_name)).replace(";go", ";cookie_list = go")
    eval_js = eval_js.replace("<script>", "").replace("</script>", "")
    eval_js = "window = {};window.navigator = {};window.navigator.userAgent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36';" + "function get_cookie(){"+eval_js + "\n return cookie_list}"
    ctx = execjs.compile(eval_js)
    result = ctx.call("get_cookie")
    cookie_dict['__jsl_clearance_s'] = result[0]
    cookie_str = "".join([x + "=" + cookie_dict[x] + ";" for x in cookie_dict.keys()])
    headers.update({'Cookie': cookie_str})
    print(headers)
    res_512_finall = requests.get(url, headers=headers)
    print(res_512_finall.text)

```

**总结**

加速乐相对来说还是很简单的，只需要记住以下步骤还是很容易手撕的。

第一次请求时候，记得获取它的 set-cookie 值。

携带第一次请求生成和返回的 Cookie 去请求第二次。

携带第二次请求生成和返回的 Cookie 去请求第三次。

跟着 上来就是一把梭 将第三次的代码处理即可。

**各位大佬觉得本文写的不错的话，可以一键四连哦。**