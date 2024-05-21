> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7yZzf4V-2fcn-jRwm4uO-w)

Frida 虽然确实调试起来相当方便，但是 Xposed 由于能够安装在用户手机上实现持久化的 hook，至今受到很多人的青睐，对于微信小程序的 wx.request API。

本文将以该 API 作为用例，介绍如何使用 Xposed 来对微信小程序的 js API 进行 hook，首先我们要知道微信小程序跟服务器交互最终都会调用 wx.request 这个 api 跟服务器交互，我们的最终目的是要通过分析这个 api 得到 request 数据和 response 数据，测试的微信版本是 8.0.30。

  
**背景知识**

众所周知，Xposed 主要用于安卓 Java 层的 Hook，而微信小程序则是由 JS 编写的，显然无法直接进行 hook。安卓有一个 WebView 的组件能够用于网页的解析和 js 的执行，并且提供了 JSBridge 可以支持 js 代码调用安卓的 java 代码，微信小程序正是以此为基础开发了它的微信小程序框架，微信小程序特有的 API 则是由这个框架中的 WxJsApiBridge 提供的，因此以 wx. 开头的 API 都能在这个框架中找到对应的 Java 代码，所以我们虽然不能直接 hook js 代码，但是我们可以通过 hook 这些 js api 对应的 Java 代码来实现微信小程序 api 的 hook。

  
**Frida 调试**  
  

在编写 Xposed 插件前，首先先使用 Frida 进行逆向分析以及 hook 调试，确保功能能够实现后在用 Xposed 编写插件，毕竟 Xposed 插件调试起来还是不如 Frida 方便。

  
首先我们要知道，js 代码中的 wx.xxx 字符串一定会在 java 层中出现吗？答案是否定的，wx.getLocation 和其他一些 api 的名字确实会在 java 层出现，java 层的字段表现形式就是 String NAME = “getLocation”，但是我们今天要分析的 api wx.request 在 java 层中不叫这个名字，因此在 jadx 反编译完微信以后，直接搜索该字符串是搜索不到的。既然搜索不到那就换一个思路，微信小程序 wx.xxx 最终都会通过 WxJsApiBridge 调用到 java 层对应的 api 实现，所以我们可以通过 frida hook 这个类相应的方法来确定 wx.request 在 java 层对应的 api 叫什么名字，通过 jadx 搜索发现有一个类叫做 com.tencent.mm.appbrand.commonjni.AppBrandJsBridgeBinding。

  
定位到具体的类以后，我们可以用 Objection 来 hook 整个类来观察这个类中函数的调用情况，以此发现主要的函数。不过在 hook 之前，需要注意的是，微信小程序一般会以新进程的方式启动，其进程名为 com.tencent.mm:appbrand0、com.tencent.mm:appbrand1、com.tencent.mm:appbrand2、com.tencent.mm:appbrand3、com.tencent.mm:appbrand4。微信最多同时运行 5 个小程序进程，所以最多只能同时运行 5 个小程序，如果打开第 6 个小程序，微信会把最近不怎么用的小程序关掉给新打开的小程序运行，因此，如果直接用 frida -U com.tencent.mm -l xxx 或者 objection -g com.tencent.mm explore 来 hook 的话，是无法看到函数调用的，因为你 hook 的进程不是微信小程序的进程而是微信的进程。所以我们要指定 pid 来进行 hook，可以使用 dumpsys activity top | grep ACTIVITY 来得到；也可以使用 frida -UF -l xxx 来 hook 当前最顶层的 Activity。对于 Xposed 则没有这个问题，只需指定微信的包名就会自动 hook 上所有的子进程。

  
结合动态测试的函数调用结果，随便浏览一下被调用的函数的代码，看到了一个主要函数代码如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsVLmO5Q2cJNPTCaChniaYvneuFjBEVUBXwUxbd5Ry95xJvB5eluPjX8w/640?wx_fmt=png)  

这个函数是关键，通过动态 hook 调试发现，每个 wx.xxx 的 api 通过经过 invokeCallbackHandler，由这个函数通过调用 native 函数最终调用到具体的 java 实现，其他的参数我们先不管，我们看最后一个参数 String 类型，通过动态 hook 调试发现这个参数保存的就是具体的 java 层实现的名字，wx.request 对应的 java 层的 api 名字如下图所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsRics1zn4UlGTLG0pZEl4xOfpzc4Qw3haJMlPlIksG7r9Cm7gdLQubZA/640?wx_fmt=png)  
  

我们直接在 jadx 中搜索整个名字看看，如下图所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZs1fXs3iaGibgyR4cInYadR7V4YIKwK6grmjBwnJSo6wxOQeglVUC8oJicQ/640?wx_fmt=png)  
  

搜索发现还有一处，异步调用是走上面这个类，同步调用是走下面这个类 如下图所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsxVx0b1fbQvnMZZPXzpicQwSkYXyS6GtMlKicCNZ4l9Iic0ns6UowtDTmg/640?wx_fmt=png)  
  

这个 a 方法就是关键，废话不多说直接说结果，a 方法的第二个参数就是 request 参数，最终这两个类的 a 方法都会调用 this.yOx.a，点进去如下所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsmcyUlIbah7Q5iarKAGqJrOoeruKS9L6EXsf8JcicRjAH76sKElIJa8fA/640?wx_fmt=png)  
  

第二个参数就是 wx.request 请求参数，所以直接 hook 这个方法就可以得到向服务器发送的数据，那服务器返回的数据在哪里呢！右键点击查找这个参数的所有引用，发现有两处很可疑，如下所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsicHkOEP0hibFibcDA7fQ0y4y6cVIysZVvvJFbYR4mKDPwlR9jMiab3wXMw/640?wx_fmt=png)  

点进去看看这两处的调用，如下所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsVkVIwXSVFD30YEiawE0modOcfrvWQQzoDnJLBU3ldTavGj380wTt63g/640?wx_fmt=png)  

发现他们在一起，通过第一处调用的上一行的 log 可以看出，不验证白名单的 domains 都会走这里，正常的微信小程序都会开启这个验证，所以第二处才是跟服务器发起交互的函数调用，这样就找到了 wx.request 发送的函数调用，因为我们不止要获取发送的数据，还要获取服务器响应的数据，所以还得分析 wx.request callback 在 java 层对应的实现，具体的 callback 的实现是在 so 层，由于时间关系就懒得跟了，这里告诉大家一个简单的获取响应的数据的方法，前面说过 wx.xx 的 api 都要通过 WxJsApiBridge 做桥接实现跟 java 层方法的调用，这样的话可不可以直接 hook com.tencent.mm.appbrand.commonjni.AppBrandJsBridgeBinding 这个类的接受订阅消息的方法来得到响应的数据呢？答案是可以的，如下所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsfo5fZyOqiaibqHibts30NssnWmvVrFUajmwdCLiaJed7IBlCnOsDeRrOTA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZseszl0AWdeoP462zkvWfZ5XWQ76JibXUH1UcnZIOOeeJJtV4bRMw4OOw/640?wx_fmt=png)  
  

Xposed hook wx.request java 层代码得到发送的数据实现如下所示：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZseynY5BpWUcaHr4Prt38geA4hEZyqQUG4IVfhdKwibZ6gJZziaxN9auMw/640?wx_fmt=png)  
  

得到响应数据的 Xposed 代码就不贴了，方法同上。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8EMsUe3pwd36e3I8bgD2PZsGSnIBzOqWjKISWIJjElZxK29bDKRs9KFIRyCXu3G21fsygqJNp079w/640?wx_fmt=png)

  

**看雪 ID：Sharp_Wang**

https://bbs.kanxue.com/user-home-890497.htm

* 本文为看雪论坛优秀文章，由 Sharp_Wang 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8FJjl92zYviaHbyrEZEZzyZFOjibI9RsDgTicPv5dSuE7FmbcRjV9sn7Y7qDnx1icFuO45cIj22DLEZDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458499288&idx=1&sn=b2b9cd6ff7388a8658d254e13c72f9ad&chksm=b18e885286f9014436a590f2531fda167be67e1e227ea395812968e828932bd44eade34b0dbf&scene=21#wechat_redirect)

**#** **往期推荐**

1、[在 Windows 下搭建 LLVM 使用环境](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500602&idx=1&sn=4bcc2af3c62e79403737ce6eb197effc&chksm=b18e8d7086f9046631a74245c89d5029c542976f21a98982b34dd59c0bda4624d49d1d0d246b&scene=21#wechat_redirect)

2、[深入学习 smali 语法](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500599&idx=1&sn=8afbdf12634cbf147b7ca67986002161&chksm=b18e8d7d86f9046b55ff3f6868bd6e1133092b7b4ec7a0d5e115e1ad0a4bd0cb5004a6bb06d1&scene=21#wechat_redirect)

3、[安卓加固脱壳分享](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500598&idx=1&sn=d783cb03dc6a3c1a9f9465c5053bbbee&chksm=b18e8d7c86f9046a67659f598242acb74c822aaf04529433c5ec2ccff14adeafa4f45abc2b33&scene=21#wechat_redirect)

4、[Flutter 逆向初探](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500574&idx=1&sn=06344a7d18a72530077fbc8f93a40d8f&chksm=b18e8d5486f904424874d7308e840523ebfb2db20811d99e4b0249d42fa8e38c4e80c3f622c6&scene=21#wechat_redirect)

5、[一个简单实践理解栈空间转移](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500315&idx=1&sn=19b12ab150dd49325f93ae9d73aef0c4&chksm=b18e8c5186f90547f3b615b160d803a320c103d9d892c7253253db41124ac6993d83d13c5789&scene=21#wechat_redirect)

6、[记一次某盾手游加固的脱壳与修复](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458500165&idx=1&sn=b16710232d3c2799c4177710f0ea6d41&chksm=b18e8ccf86f905d9a0b6c2c40997e9b859241a4d7f798c4aeab21352b0a72b6135afce349262&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FHJ5XNqGmzLUOYeEJc9zylullBt3UKTEQsoxy2icCZlrib0kGSnnibUmPhrtv1ic2HR4SZvjH2PiaQASw/640?wx_fmt=gif)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FHJ5XNqGmzLUOYeEJc9zylullBt3UKTEQsoxy2icCZlrib0kGSnnibUmPhrtv1ic2HR4SZvjH2PiaQASw/640?wx_fmt=gif)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FHJ5XNqGmzLUOYeEJc9zylullBt3UKTEQsoxy2icCZlrib0kGSnnibUmPhrtv1ic2HR4SZvjH2PiaQASw/640?wx_fmt=gif)

**球在看**