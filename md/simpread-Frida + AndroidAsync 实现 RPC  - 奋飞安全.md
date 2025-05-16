> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [91fans.com.cn](http://91fans.com.cn/post/fridarpctwo/#gsc.tab=0)

> AndroidAsync 启动 Web 服务实现 App 函数的 RPC 调用

我们之前在 [某段子 App 签名计算方法 (一)](http://91fans.com.cn/post/zysignone) 这篇教程里面使用 python 的 Flask 库启动一个 web Server 来实现 App 函数的 RPC 调用。

今天我们介绍一个新盆友，AndroidAsync， 用 AndroidAsync 来启动 web Server，这样 frida 就直接搞定，不需要再请 Python 来帮忙了。

### AndroidAsync[](#_androidasync)

AndroidAsync 的详细介绍大家可以自行谷歌，反正就是一个比较帅的网络库了。

它的老家在这里 [https://github.com/koush/AndroidAsync](https://github.com/koush/AndroidAsync)

把它搞下来，然后编译成 jar 包再转成 dex,frida 可以调用了。准备工作就 ok 了

我们以昨天的 [某资讯 App signature 签名分析 (一)](http://91fans.com.cn/post/onenewsone/) 为例

先把 web 服务跑起来

```
adb push androidAsync.dex /data/local/tmp/

```

*   在 android.app.Application.attach 的时候启动 WebServer

```
var ApplicationCls = Java.use("android.app.Application");
ApplicationCls.attach.implementation = function () {
    try {
        var AsyncHttpServer = Java.use("com.koushikdutta.async.http.server.AsyncHttpServer");
        var androidAsync = AsyncHttpServer.$new();
        androidAsync.get("/", RequestTestCallback.$new());
        androidAsync.listen(8181);
        console.log("reg webServer Ok");
    } catch (e) {
        console.error("reg webServer Error!!!, e:" + e);
    }

    this.attach.apply(this, arguments);
};

```

代码就不用解释了，牛 X 的代码自己会说话。

在 8181 端口启动了 WebServer，然后注册了一个测试用的 RequestTestCallback

打开浏览器 [http://127.0.0.1:8181](http://127.0.0.1:8181/) 木反应？

哦，晕了，这是在手机里监听 8181，不是在电脑上，所以应该是访问手机的 ip， [http://192.168.2.113:8181/](http://192.168.2.113:8181/)

还是木反应？ 看看日志，并没有 **reg webServer Ok** 或者 **reg webServer Error!!!**

原来我们用的 Frida attach 模式，可能并没有跑到 android.app.Application.attach 这个函数。

这就好办了，设置一个开关变量，直接在签名函数 signInternal 里面启动服务

```
var bRunServer = 0;

var SignUtilCls = Java.use("com.yxdxxx.news.util.sign.SignUtil");
SignUtilCls.signInternal.implementation = function(a,b){
                if( bRunServer == 0){
                        bRunServer = 1;
                        runWebServer();
                }

        var rc = this.signInternal(a,b);
        console.log("inStr = " + b);
        console.log(">>> rc = " + rc);
        return rc;
}

```

好了这次可以看到启动成功的提示了

```
[MI NOTE Pro::com.hxxx.yxdxxx]-> reg webServer Ok

```

RunServer 的时候增加一个 /onenewssign 接口

```
OneNewsSignRequestCallback = Java.registerClass({
    name: "OneNewsSignRequestCallback",
    implements: [HttpServerRequestCallback],
    methods: {
        onRequest: function (request, response) {
            
            var InStr = request.getQuery().getString("instr");
            console.log("RPC Str = " + InStr);
            var context1 = Java.use('android.app.ActivityThread').currentApplication().getApplicationContext();
            console.log(context1);

            var SignUtilCls = Java.use("com.yxdxxx.news.util.sign.SignUtil");
            var ret = SignUtilCls.signInternal(context1,InStr);
            response.send("{\"code\":0,\"message\":\"" + ret + "\"}");
        }
    }
});

```

好了直接调用， [http://192.168.2.113:8181/onenewssign?instr=yxdxxx5.7.7.21k6lwwmig_1620967068422_166028401](http://192.168.2.113:8181/onenewssign?instr=yxdxxx5.7.7.21k6lwwmig_1620967068422_166028401) 算下和之前结果对不对。

结果并不一样，但是每次传相同的参数，结果都不一样，估计 so 的算法里面还是加入了随机数。不过应该这个结果是可用的。

不知道是 AndroidAsync 不太稳定还是 Frida+AndroidAsync 不太稳定，反正我崩了好几回。

凑活用吧，说不定是我手机的问题。多个方法总是好的。

android.content.Context 参数获取有两种方法，一种是用 Api 获取全局的 Context，还有一种就是 保存 signInternal 函数的参数。

**注意： response.send 的返回值必须是 Json**

![](http://91fans.com.cn/img/fridarpctwo/ffshow.jpeg) 1:ffshow

绝大多数时候，凑合着做完，比完美地半途而废要好。绝大多数时候，决定要做就直接开始，比自认为准备充分了再开始要好。

![](http://91fans.com.cn/img/ffzsxq.jpg)

关注微信公众号，最新技术干货实时推送

![](http://91fans.com.cn/img/ff330.png)