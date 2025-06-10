> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/z5cpA8FSIRqkHtUfCZ7Weg)

最近偶尔有朋友问以上 app 怎么抓包, 有的是旧方案, 后来 app 升级了, 有些方案就失效了, 这里重新整理一遍.

#### **本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

首先是字节系

版本: 最新版 34.6.0

之前的 quic 降级方案:  

org.chromium.CronetClient.tryCreateCronetEngine

参数置空即可, 海外的 tiktok 依旧可以用.

从 32 还是 33 某个大版本开始降级方案失效, 应该是只能改 so 才行, 但是特征随着版本是一直变化的. 具体哪个特征就不公开说了.

```
function delay_hook(so_name, hook_func) {
    var dlopen = Module.findExportByName(null, "dlopen");  // 低版本
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext"); // 高版本
    Interceptor.attach(dlopen, {
        onEnter: function (args) {
            var path = args[0].readCString();
            this.path = path;
        }, onLeave: function (retval) {
            if (this.path.indexOf(so_name) !== -1) {
                console.log("[dlopen:]", this.path);
                hook_func();
            }
        }
    });
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            var path = args[0].readCString();
            this.path = path;
        },
        onLeave: function (retval) {
            if (this.path.indexOf(so_name) !== -1) {
                console.log("\nandroid_dlopen_ext加载：", this.path);
                hook_func();
            }
        }
    });
}
function do_hook() {
    var soAddr = Module.findBaseAddress("libsscronet.so");
    var funcAddr = soAddr.add(0x33D05C)  // 34.6.0
    console.log(funcAddr)
    Interceptor.attach(funcAddr,{
        onEnter: function(args){
        },
        onLeave: function(retval){
            console.log('onLeave arg[]: ',retval)
            retval.replace(ptr(0))
            console.log('onLeave result: ',retval)
        }
    });
}
delay_hook("libsscronet.so", do_hook);  // 改so的名字和do_hook的方法体

```

失败

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVw4HaV8FcfibtXZHddfhYnJlJBUozve6jB9IyO5ghZUMLbEl8p7hxCM7jUmcLmziaqDY3CJcMKlOqw/640?wx_fmt=png&from=appmsg&watermark=1)

成功抓到

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdVw4HaV8FcfibtXZHddfhYnJXdBV9XDlNLFrq0n1sqmRU5bAfibPm9nLUas2y9Q01f4QmvjoFdiav0LQ/640?wx_fmt=png&from=appmsg&watermark=1)

ali 系

代表有 tb,xy,elm,tm 等等. hook 让它不走 spdy 即可走 https 抓包

mtopsdk.mtop.global.SwitchConfig.isGlobalSpdySslSwitchOpen

方法名字随版本变化.

```
Log.i(TAG, "进入apk");
Class<?> clazz = XposedHelpers.findClass("mtopsdk.mtop.global.SwitchConfig", lpparam.classLoader);
XposedHelpers.findAndHookMethod(clazz, "isGlobalSpdySslSwitchOpen", new XC_MethodHook() {
    public void beforeHookedMethod(MethodHookParam param) throws Throwable {
        Log.i(TAG, "is_enableSpdy 已被Hook");
    }
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
        param.setResult(false); // 修改返回值
    }
});

```

xy7.21.31

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXhO0Laf6C47zaEsgZViaa6r2MpzKLMtWAvd8qr4ttib3mugUiaHZy3YBibXZXpibIqPYE2gRT94w0Rupw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXhO0Laf6C47zaEsgZViaa6rjTzSjrFfmX6QhQN9nK9VPQZl362xfjyf5s4UdCQNO6vvtsInTdViaeg/640?wx_fmt=png&from=appmsg&watermark=1)

ks

版本 13.3.41.41640

两种方法, vpn 转发或者 quic 降级. 降级的方法是 nativeUpdateConfig, 把入参 enable_quic 改成 false 即可.

```
XposedHelpers.findAndHookMethod("com.**.aegon.Aegon", lpparam.classLoader, "nativeUpdateConfig", String.class, String.class, new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        param.args[0] = "{\"enable_quic\": false, \"preconnect_num_streams\": 2, \"quic_idle_timeout_sec\": 180, \"quic_use_bbr\": true, \"altsvc_broken_time_max\": 600, \"altsvc_broken_time_base\": 60, \"proxy_host_blacklist\": []}";
    }
});

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXhO0Laf6C47zaEsgZViaa6rMyuUAia9XRyqsgYMKQtW39w7O9sAwYaYmvibvrY0koqo7hEKQteW8DxA/640?wx_fmt=png&from=appmsg&watermark=1)

mt 系

mt,dp, 最新的抓包方案还没找到, 先搁一下. 之前的也是降级方案.

pdd

版本: 7.61.0

有长连接 (LongLink) 通道, 在 redmiNote11 上没抓到, 但是详情是加载出来的. 在 pixel4xl 上抓到, 可能是性能太低或者弱网下导致长连接断开了. 如果详情是已售罄, 说明设备被封了, 要改机.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXhO0Laf6C47zaEsgZViaa6r0ZxjNkXVQTRYh1lia0JfNAeaic3OGgkzDNGePO1T4QIv5FpVInXSDYGw/640?wx_fmt=png&from=appmsg&watermark=1)

关注我, 分享更多逆向知识, 尽请期待.