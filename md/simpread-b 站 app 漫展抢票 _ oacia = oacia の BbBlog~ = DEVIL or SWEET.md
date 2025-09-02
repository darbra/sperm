> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [oacia.dev](https://oacia.dev/bilibili-get-ticket/)

> # 前言 最近萌生了想去漫展的想法，毕竟这么久了还没有去过一次漫展嘞，听说 cp30 好像不错，不过问题就是票特别的难抢😔 趁现在 cp30 买票还没开始，今天也没有什么其他的事情，所以就简单分析了一......

最近萌生了想去漫展的想法，毕竟这么久了还没有去过一次漫展嘞，听说 cp30 好像不错，不过问题就是票特别的难抢😔

趁现在 cp30 买票还没开始，今天也没有什么其他的事情，所以就简单分析了一下 b 站的 app，写了个抢票脚本自娱自乐一下

感觉这个 app 还是很有意思的，它把主要的 webview 业务逻辑放到子进程里运行我觉得还是很不错滴，这样 frida 注入到主进程就 hook 不到了（我就被卡了快一个小时！）

但这个抢票脚本写的相当的简陋，我就不好意思放出来啦😂不过相信大家在 github 上应该可以找到更加完善的项目吧 `:)`

还有就是为什么不直接去 b 站会员购网页端分析呢，这样购票接口不是一下子就看到了嘛就不需要绕各种检测了

**just for fun!**

包名: `tv.danmaku.bili`

入口: `tv.danmaku.bili.MainActivityV2`

先看看会员购页面是哪一个 `activity` 的

<table><tbody><tr><td data-num="1"></td><td><pre>PS D:\work\analysis\bilibili&gt; adb shell "dumpsys activity top | grep ACTIVITY"
</pre></td></tr><tr><td data-num="2"></td><td><pre>  ACTIVITY com.google.android.apps.nexuslauncher/.NexusLauncherActivity ad36d3f pid=2139
</pre></td></tr><tr><td data-num="3"></td><td><pre>  ACTIVITY tv.danmaku.bili/.MainActivityV2 e753a0a pid=12505
</pre></td></tr></tbody></table>

用 frida 写个简单的 hook 脚本注入进去看看情况

<table><tbody><tr><td data-num="1"></td><td><pre>function hook(){
</pre></td></tr><tr><td data-num="2"></td><td><pre>    Java.perform(function(){
</pre></td></tr><tr><td data-num="3"></td><td><pre>        const activity = Java.use("com.mall.ui.page.base.MallWebFragmentLoaderActivity");
</pre></td></tr><tr><td data-num="4"></td><td><pre>        activity.onCreate.overload('android.os.Bundle').implementation = function(x){
</pre></td></tr><tr><td data-num="5"></td><td><pre>            const retv = this.onCreate(x);
</pre></td></tr><tr><td data-num="6"></td><td><pre>            return retv;
</pre></td></tr><tr><td data-num="7"></td><td><pre>        }
</pre></td></tr><tr><td data-num="8"></td><td><pre>    })
</pre></td></tr><tr><td data-num="9"></td><td><pre>}
</pre></td></tr><tr><td data-num="10"></td><td></td></tr><tr><td data-num="11"></td><td><pre>setImmediate(hook,0);
</pre></td></tr></tbody></table><table><tbody><tr><td data-num="1"></td><td><pre>oriole:/data/local/tmp 
</pre></td></tr><tr><td data-num="2"></td><td><pre>adb forward tcp:1234 tcp:1234
</pre></td></tr><tr><td data-num="3"></td><td><pre>frida -H 127.0.0.1:1234 -l .\hook.js -f tv.danmaku.bili
</pre></td></tr></tbody></table>

注入进去之后不出意料的卡在主界面不动了

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222856973.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222856973.png)

去看看是在哪一个 so 卡住的

<table><tbody><tr><td data-num="1"></td><td><pre>function hook_dlopen() {
</pre></td></tr><tr><td data-num="2"></td><td><pre>    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
</pre></td></tr><tr><td data-num="3"></td><td><pre>        {
</pre></td></tr><tr><td data-num="4"></td><td><pre>            onEnter: function (args) {
</pre></td></tr><tr><td data-num="5"></td><td><pre>                var pathptr = args[0];
</pre></td></tr><tr><td data-num="6"></td><td><pre>                if (pathptr !== undefined &amp;&amp; pathptr != null) {
</pre></td></tr><tr><td data-num="7"></td><td><pre>                    var path = ptr(pathptr).readCString();
</pre></td></tr><tr><td data-num="8"></td><td><pre>                    console.log("load " + path);
</pre></td></tr><tr><td data-num="9"></td><td><pre>                }
</pre></td></tr><tr><td data-num="10"></td><td><pre>            }
</pre></td></tr><tr><td data-num="11"></td><td><pre>        }
</pre></td></tr><tr><td data-num="12"></td><td><pre>    );
</pre></td></tr><tr><td data-num="13"></td><td><pre>}
</pre></td></tr><tr><td data-num="14"></td><td></td></tr><tr><td data-num="15"></td><td><pre>setImmediate(hook_dlopen)
</pre></td></tr></tbody></table>

哈哈没想到是 `libmsaoaidsec.so` , 这个 so 我在很多的 apk 里面都看到过了，按照以前逆向的经验这个 so 里面没有任何的业务代码，在 `init_proc` 里面是纯的检测逻辑

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222907116.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222907116.png)

我对 `libmsaoaidsec.so` 里面的控制流平坦化还是很感兴趣的，之后会去专门分析一下这个 so:)

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222916985.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222916985.png)

现在我们就用最简单的方法去反调试好啦，就是不让 app 加载这个 so, 具体做法就是在打开这个 so 时，把要加载的 so 的字符串置空

<table><tbody><tr><td data-num="1"></td><td><pre>function hook_dlopen() {
</pre></td></tr><tr><td data-num="2"></td><td><pre>    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
</pre></td></tr><tr><td data-num="3"></td><td><pre>        onEnter: function (args) {
</pre></td></tr><tr><td data-num="4"></td><td><pre>            var pathptr = args[0];
</pre></td></tr><tr><td data-num="5"></td><td><pre>            if (pathptr !== undefined &amp;&amp; pathptr != null) {
</pre></td></tr><tr><td data-num="6"></td><td><pre>                var path = ptr(pathptr).readCString();
</pre></td></tr><tr><td data-num="7"></td><td><pre>                if(path.indexOf('libmsaoaidsec.so') &gt;= 0){
</pre></td></tr><tr><td data-num="8"></td><td><pre>                    ptr(pathptr).writeUtf8String("");
</pre></td></tr><tr><td data-num="9"></td><td><pre>                }
</pre></td></tr><tr><td data-num="10"></td><td><pre>                console.log("load " + path);
</pre></td></tr><tr><td data-num="11"></td><td><pre>            }
</pre></td></tr><tr><td data-num="12"></td><td><pre>        }
</pre></td></tr><tr><td data-num="13"></td><td><pre>    });
</pre></td></tr><tr><td data-num="14"></td><td><pre>}
</pre></td></tr><tr><td data-num="15"></td><td></td></tr><tr><td data-num="16"></td><td></td></tr><tr><td data-num="17"></td><td><pre>setImmediate(hook_dlopen_anti)
</pre></td></tr></tbody></table>

再次注入代码之后就 hook 成功了

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222925964.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222925964.png)

用 `Device Monitor` 看一下购票的页面，发现是套了一个 `WebView`

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222934485.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222934485.png)

想要在 android 中使用 chrome 的 devtool 开启 webview debug, 需要注入下面的 frida 脚本

<table><tbody><tr><td data-num="1"></td><td><pre>function webview_debug() {
</pre></td></tr><tr><td data-num="2"></td><td><pre>    Java.perform(function () {
</pre></td></tr><tr><td data-num="3"></td><td><pre>        var WebView = Java.use('android.webkit.WebView');
</pre></td></tr><tr><td data-num="4"></td><td></td></tr><tr><td data-num="5"></td><td><pre>        WebView.$init.overloads.forEach(function(init) {
</pre></td></tr><tr><td data-num="6"></td><td><pre>            init.implementation = function() {
</pre></td></tr><tr><td data-num="7"></td><td><pre>                
</pre></td></tr><tr><td data-num="8"></td><td><pre>                var instance = init.apply(this, arguments);
</pre></td></tr><tr><td data-num="9"></td><td></td></tr><tr><td data-num="10"></td><td><pre>                
</pre></td></tr><tr><td data-num="11"></td><td><pre>                WebView.setWebContentsDebuggingEnabled(true);
</pre></td></tr><tr><td data-num="12"></td><td></td></tr><tr><td data-num="13"></td><td><pre>                console.log('[*] WebView debug open~');
</pre></td></tr><tr><td data-num="14"></td><td></td></tr><tr><td data-num="15"></td><td><pre>                
</pre></td></tr><tr><td data-num="16"></td><td><pre>                return instance;
</pre></td></tr><tr><td data-num="17"></td><td><pre>            };
</pre></td></tr><tr><td data-num="18"></td><td><pre>        });
</pre></td></tr><tr><td data-num="19"></td><td><pre>    });
</pre></td></tr><tr><td data-num="20"></td><td><pre>}
</pre></td></tr></tbody></table>

然后使用 USB 将电脑和手机相连

在电脑端的 chrome 打开 `chrome://inspect/#devices`

之后我们点击这个 `Port forwarding` 按钮配置端口转发

[![](https://oacia.dev/bilibili-get-ticket/image-20240713222956316.png)](https://oacia.dev/bilibili-get-ticket/image-20240713222956316.png)

然后 选一个端口点击 `Done`

[![](https://oacia.dev/bilibili-get-ticket/image-20240713223022350.png)](https://oacia.dev/bilibili-get-ticket/image-20240713223022350.png)

但是这样做并没有什么网页可以 `inspect`

[![](https://oacia.dev/bilibili-get-ticket/image-20240713223028761.png)](https://oacia.dev/bilibili-get-ticket/image-20240713223028761.png)

通过 `device monitor` 来看，这个 b 站肯定是调用了 `Webview` 的，那为什么这个脚本没有 hook 到 Webview 的创建呢？我觉得可能的原因就是 webview 并不是在主进程被创建的，而是在子进程被创建的，我们打印一下 bilibili 建立的进程来看看情况

[![](https://oacia.dev/bilibili-get-ticket/image-20240713223040869.png)](https://oacia.dev/bilibili-get-ticket/image-20240713223040869.png)

的确，除了主进程之外，还多了其他的四个进程，感觉这个 `web` 进程很可疑呀

那写个 python 脚本让 frida 也去注入这个 `tv.danmaku.bili:web` 子进程好啦

<table><tbody><tr><td data-num="1"></td><td><pre>import codecs
</pre></td></tr><tr><td data-num="2"></td><td><pre>import frida
</pre></td></tr><tr><td data-num="3"></td><td><pre>import sys
</pre></td></tr><tr><td data-num="4"></td><td><pre>import threading
</pre></td></tr><tr><td data-num="5"></td><td></td></tr><tr><td data-num="6"></td><td><pre>device = frida.get_device_manager().add_remote_device("127.0.0.1:1234")
</pre></td></tr><tr><td data-num="7"></td><td><pre>pending = []
</pre></td></tr><tr><td data-num="8"></td><td><pre>sessions = []
</pre></td></tr><tr><td data-num="9"></td><td><pre>scripts = []
</pre></td></tr><tr><td data-num="10"></td><td><pre>event = threading.Event()
</pre></td></tr><tr><td data-num="11"></td><td></td></tr><tr><td data-num="12"></td><td><pre>jscode = open('./hook.js', 'r', encoding='utf-8').read()
</pre></td></tr><tr><td data-num="13"></td><td><pre>pkg = "tv.danmaku.bili"  
</pre></td></tr><tr><td data-num="14"></td><td></td></tr><tr><td data-num="15"></td><td></td></tr><tr><td data-num="16"></td><td><pre>def spawn_added(spawn):
</pre></td></tr><tr><td data-num="17"></td><td><pre>    event.set()
</pre></td></tr><tr><td data-num="18"></td><td><pre>    if spawn.identifier == pkg or spawn.identifier == f"{pkg}:web":
</pre></td></tr><tr><td data-num="19"></td><td><pre>        print('spawn_added:', spawn)
</pre></td></tr><tr><td data-num="20"></td><td><pre>        session = device.attach(spawn.pid)
</pre></td></tr><tr><td data-num="21"></td><td><pre>        script = session.create_script(jscode)
</pre></td></tr><tr><td data-num="22"></td><td><pre>        script.on('message', on_message)
</pre></td></tr><tr><td data-num="23"></td><td><pre>        script.load()
</pre></td></tr><tr><td data-num="24"></td><td><pre>    device.resume(spawn.pid)
</pre></td></tr><tr><td data-num="25"></td><td></td></tr><tr><td data-num="26"></td><td></td></tr><tr><td data-num="27"></td><td><pre>def spawn_removed(spawn):
</pre></td></tr><tr><td data-num="28"></td><td><pre>    print('spawn_removed:', spawn)
</pre></td></tr><tr><td data-num="29"></td><td><pre>    event.set()
</pre></td></tr><tr><td data-num="30"></td><td></td></tr><tr><td data-num="31"></td><td></td></tr><tr><td data-num="32"></td><td><pre>def on_message(spawn, message, data):
</pre></td></tr><tr><td data-num="33"></td><td><pre>    print('on_message:', spawn, message, data)
</pre></td></tr><tr><td data-num="34"></td><td></td></tr><tr><td data-num="35"></td><td></td></tr><tr><td data-num="36"></td><td><pre>def on_message(message, data):
</pre></td></tr><tr><td data-num="37"></td><td><pre>    if message['type'] == 'send':
</pre></td></tr><tr><td data-num="38"></td><td><pre>        print("[*] {0}".format(message['payload']))
</pre></td></tr><tr><td data-num="39"></td><td><pre>    else:
</pre></td></tr><tr><td data-num="40"></td><td><pre>        print(message)
</pre></td></tr><tr><td data-num="41"></td><td></td></tr><tr><td data-num="42"></td><td></td></tr><tr><td data-num="43"></td><td><pre>device.on('spawn-added', spawn_added)
</pre></td></tr><tr><td data-num="44"></td><td><pre>device.on('spawn-removed', spawn_removed)
</pre></td></tr><tr><td data-num="45"></td><td></td></tr><tr><td data-num="46"></td><td><pre>device.enable_spawn_gating()
</pre></td></tr><tr><td data-num="47"></td><td></td></tr><tr><td data-num="48"></td><td><pre>event = threading.Event()
</pre></td></tr><tr><td data-num="49"></td><td><pre>print('Enabled spawn gating')
</pre></td></tr><tr><td data-num="50"></td><td></td></tr><tr><td data-num="51"></td><td><pre>pid = device.spawn([pkg])
</pre></td></tr><tr><td data-num="52"></td><td></td></tr><tr><td data-num="53"></td><td><pre>session = device.attach(pid)
</pre></td></tr><tr><td data-num="54"></td><td><pre>print("[*] Attach Application id:", pid)
</pre></td></tr><tr><td data-num="55"></td><td><pre>device.resume(pid)
</pre></td></tr><tr><td data-num="56"></td><td><pre>sys.stdin.read()
</pre></td></tr></tbody></table>

这样就 hook 到子进程啦

[![](https://oacia.dev/bilibili-get-ticket/image-20240713223048889.png)](https://oacia.dev/bilibili-get-ticket/image-20240713223048889.png)

打开了 webview 的 debug 之后，终于可以 inspect 了！

[![](https://oacia.dev/bilibili-get-ticket/image-20240713223059764.png)](https://oacia.dev/bilibili-get-ticket/image-20240713223059764.png)

之后的过程就很简单啦

1.  点击漫展详情，抓个包
2.  点击立即购票，抓个包
3.  点击提交订单，抓个包

我写了一个小脚本还挺好用的，用这个 js 脚本可以读取剪切板里的 curl 请求并转换为 python 的 request 请求复制到剪切板上 `:)`

剪切板里的 curl 请求是从这里来的

[![](https://oacia.dev/bilibili-get-ticket/image-20240713225910636.png)](https://oacia.dev/bilibili-get-ticket/image-20240713225910636.png)

记得安装一下包就好了

```
npm i copy-paste
```

<table><tbody><tr><td data-num="1"></td><td><pre>import * as curlconverter from 'curlconverter';
</pre></td></tr><tr><td data-num="2"></td><td><pre>import ncp from 'copy-paste'
</pre></td></tr><tr><td data-num="3"></td><td><pre>let cmd = ncp.paste()
</pre></td></tr><tr><td data-num="4"></td><td><pre>var res = curlconverter.toPython(cmd);
</pre></td></tr><tr><td data-num="5"></td><td><pre>console.log(res)
</pre></td></tr><tr><td data-num="6"></td><td></td></tr><tr><td data-num="7"></td><td><pre>ncp.copy(res, function () {
</pre></td></tr><tr><td data-num="8"></td><td><pre>  console.log("OK")
</pre></td></tr><tr><td data-num="9"></td><td><pre>})
</pre></td></tr></tbody></table>

分析一下接口，写一下脚本就抢票成功啦

[![](https://oacia.dev/bilibili-get-ticket/image-20240713225457049.png)](https://oacia.dev/bilibili-get-ticket/image-20240713225457049.png)

`waiting for cp30!`