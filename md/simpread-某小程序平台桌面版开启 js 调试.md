> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hjzYVflkK2Azi0wyjeKcTA)

网上有很多大同小异的文章介绍如何逆向此类小程序的 js，工作流大体如下：

1.  安装小而美 app 到 root 过的 Android（或模拟器）
    
2.  使用小程序
    
3.  在 /data 目录下查找 wxpkg
    
4.  使用有人实现好的 nodejs 库解包
    
5.  将解包导入 IDE，实现 js 单步调试
    

这个流程相当于做了重打包，在导入 IDE 之后无法通过开发者的 appid 检查，涉及到网络请求的部分会出错。

能不能直接在原汁原味的生产环境上开启 js 单步调试呢？

![](https://mmbiz.qpic.cn/mmbiz_png/6N4b2yN3FOKApCcwLcb4QtmhibKqmFK9CRxquk6X0cUzAZsicFNAU8alica1yZ3QQmlsKwhGEO2nA8KRDPsMPKENQ/640?wx_fmt=png)

在 mac 上的小程序使用 JavaScriptCore 实现。JSC 自带了单步调试，只是默认没有开启。另外由于小程序渲染界面的引擎并不是简单的 WKWebView，本文仅限单步调试 js，不能审查界面元素。

之前公众号发过一个插件，也介绍了原理，但是并没有点破可以直接拿来调小程序：[全局开启 iOS / mac 的 WebView 调试](http://mp.weixin.qq.com/s?__biz=Mzk0NDE3MTkzNQ==&mid=2247483775&idx=1&sn=dfa5cf10a82521cf6502810c04cca6c3&chksm=c329ff8ff45e7699f3a916389ebea225fee84e3618e7723f0aab4e7fa87a9ed41ab029b5e187&scene=21#wechat_redirect)

最近 Safari 更新到 16.4，把我的插件搞不兼容了。想起这一茬，顺手水了篇推送。之前的插件实现方式是注入代码到 webinspectord，修改系统检查 entitlement 的逻辑。

引用的文章链接中提供的 frida 脚本直接可以用在 mac 上，然后再打开某小而美 mac 版，就可以看到各种可以审查的页面：

![](https://mmbiz.qpic.cn/mmbiz_png/6N4b2yN3FOKApCcwLcb4QtmhibKqmFK9CP9ZdsquKGTiaWYTFyQcGszYcb9reCxYmxhQCs7clg9tGWmIicAibdRq6w/640?wx_fmt=png)

居然还用了 JSPatch（注意里面的 _OC_callC）。

在新版的 macOS 上这段脚本不能用了，来看 WebKit 博客怎么说的。  

![](https://mmbiz.qpic.cn/mmbiz_png/6N4b2yN3FOKApCcwLcb4QtmhibKqmFK9CXFeYQyFabzGcGKibrgrdEF95yMTCicNGAV7qeTp75VYHFyFibPcuic6vUg/640?wx_fmt=png)

之前 app 的 WKWebView 和 JSContext 能否被调试，取决于应用是否是 Xcode 调试版（具有 get-task-allow），或者使用其他特殊的 entitlement。

现在 Safari 将决定权全权交给开发者，直接在 app 当中设置 WKWebView / JSContext 的 isInspectable 属性为 @YES，就可以细粒度地控制某个页面允许用 F12。

例如 WebKit 官方给的开启 JSContext 调试的示例：

```
JSContext *jsContext = [JSContext new];
jsContext.inspectable = YES;

```

那么我们只要注入 app（而不是之前的 webinspectord）就可以修改了。

除了 hook 类的初始化方法，frida 里正好有一个 ObjC.chooseSync 函数，可以根据 class 在内存中搜索已经初始化好的对象。

因此我们只需要启动好对应的小程序，然后 frida 附加到 Mini Program 上执行一行 js 即可：

```
['WKWebView', 'JSContext'].forEach(
    clazz => ObjC.chooseSync(ObjC.classes[clazz]).forEach(
        v => v.setInspectable_(ptr(1))
    )
)

```

小而美的小程序和主程序不在一个进程当中运行，通常会看到两个 Mini Program 进程。

用 python 自动筛选一下：

```
import frida
mac = frida.get_local_device()
if mac.query_system_parameters()['os']['id'] != 'macos':
    raise RuntimeError('This script is only for Mac OS')
for proc in mac.enumerate_processes():
    if proc.name not in ['WeChat', 'Mini Program']:
        continue
    print('Patching %s (%d)' % (proc.name, proc.pid))
    session = mac.attach(proc.pid)
    script = session.create_script('''
        ['WKWebView', 'JSContext'].forEach(
            clazz => ObjC.chooseSync(ObjC.classes[clazz]).forEach(
                v => v.setInspectable_(ptr(1))
            )
        )
    ''')
    script.load()
    script.unload()
    session.detach()

```

这个脚本需要关闭 macOS 的 SIP（rootless），将显著降低系统安全性。在 Apple Silicon 上除了关掉 sip，还需要开启 am64e abi，然后重启。  

```
sudo nvram boot-args="-arm64e_preview_abi"

```

来看效果。

直接调试某社交网站的小程序：

![](https://mmbiz.qpic.cn/mmbiz_png/6N4b2yN3FOIcRZ4iccQwCEFuoGt0kQSyft1VrDU8ibOoGNxSfZwj5usgPZSS1X7jPVeqMrEGgutjX8S4zmpXw3nw/640?wx_fmt=png)

单步进入请求参数，都抓下来了

![](https://mmbiz.qpic.cn/mmbiz_png/6N4b2yN3FOIcRZ4iccQwCEFuoGt0kQSyfhnm7kLVNSI1zNpFMFOkL1hjgShchPEZqWUiaeGbBZeL1G7GicGmvy9yw/640?wx_fmt=png)

本文仅限技术讨论。如遇到账号被风控甚至产生其他后果，请自行承担…