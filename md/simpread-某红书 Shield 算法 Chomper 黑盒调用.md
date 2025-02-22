> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fqv8tyurN38UZF9S-O0EnA)

某红书 Shield 算法 Chomper 黑盒调用

从安卓还原算法过程得知需要初始化设置一个 Token 这里选择的是一个老版本作为案例，这里也不需要其他多余的部环境操作。 chomper 地址：https://github.com/sledgeh4w/chomper

初始化
---

从 frida-trace 无法得知初始化做了什么，只能看到具体的算法调用逻辑。

```
           /* TID 0x103 */
   863 ms  +[XYSSHandle reqAuthorityWithHeaders:<CFBasicHash 0x283b31740 [0x201426858]>{type = mutable dict, count = 10,
entries =>
 0 : Accept-Language = zh-Hans;q=1, en;q=0.9
 1 : X-B3-TraceId = a4e86b06fbe691f3
 2 : Connection = Keep-Alive
 3 : Mode = rawIp
 4 : X-raw-ptr = 0
 7 : Host = edith.xiaohongshu.com
 9 : User-Agent = discover/7.1 (iPhone; iOS 16.7.10; Scale/2.00) Resolution/750*1334 Version/7.1 Build/7010202 Device/(Apple Inc.;iPhone10,1) NetType/WiFi
 10 : xy-platform-info = platform=iOS&version=7.1&build=7010202&deviceId=C53A986E-875F-4FE2-9C49-BC1DD280E337&bundle=com.xingin.discover
 11 : Accept-Encoding = br;q=1.0, gzip;q=1.0, compress;q=0.5
 12 : xy-common-params = app_id=ECFAAF02&build=7010202&deviceId=C53A986E-875F-4FE2-9C49-BC1DD280E337&device_fingerprint=2025022115160335e8c03be20ba5b6b300c149f76fc3f50123d7e2b8cb1cf6&device_fingerprint1=2025022115160335e8c03be20ba5b6b300c149f76fc3f50123d7e2b8cb1cf6&fid=1740122163-0-0-970427ae7226e33071bdefb4866c668a&identifier_flag=0&lang=zh-Hans&launch_id=761816146&platform=iOS&project_id=ECFAAF&sid=session.1740122519639795203705&t=1740123589&tz=Asia/Shanghai&uis=light&version=7.1
}
 request:0x282c35e00 bodyData:nil]


```

### dlopen 监控

无法从 frida-trace 得知初始化的值，只能在 dlopen 之后做![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHZrgS70ppeHrvia6Z0h7TW02CKRvTlmCbhtzympIDicicibBrCVCzQtZs1QiaMZia9pLfzYXgkYkwO9emAQ/640?wx_fmt=png&from=appmsg)代码如下：从 frida-trace 得知 XYSSHandle 的相关初始化函数。

```
let dlopen = Module.findExportByName(null, "dlopen");

Interceptor.attach(dlopen, {
    onEnter: function (args) {
        this.hook = false
        if ((args[0].readUtf8String() + '').indexOf("XYSecureShield") > -1) {
            this.hook = true;
        }
    }, onLeave: function (retval) {
        if (this.hook) {
            var methodName = "+ setup:build:key:";
            var hooking = ObjC.classes["XYSSHandle"][methodName];
// console.log(hooking, 'hooking')
            Interceptor.attach(hooking.implementation, {
                onEnter: function (args) {
                    console.log(ObjC.Object(args[2]), ObjC.Object(args[3]), ObjC.Object(args[4]))
                }, onLeave: function (returnValues) {
                }
            });
            var methodName = "+ setToken:";
            var hooking = ObjC.classes["XYSSHandle"][methodName];
// console.log(hooking, 'hooking')
            Interceptor.attach(hooking.implementation, {
                onEnter: function (args) {
                    console.log("setToken", ObjC.Object(args[2]))
                }, onLeave: function (returnValues) {
                }
            });
            let module_name = "XYSecureShield"
            var base_addr = Module.findBaseAddress(module_name);
// console.log(hooking, 'hooking')
            Interceptor.attach(base_addr.add(0x969C), {
                onEnter: function (args) {
                    console.log("0x969C")
                }, onLeave: function (returnValues) {
                }
            });
        }
    }
});


```

### Chomper 调用初始化

基础框架请看上一篇文章 某手 sig3-ios 算法 Chomper 黑盒调用 [1] 如下代码就初始化完成

```
objc.msg_send("XYSSHandle", "setup:build:key:", pyobj2nsobj(emu, "ECFAAF02"), pyobj2nsobj(emu, "7010202"),
              pyobj2nsobj(emu, "C53A986E-875F-4FE2-9C49-BC1DD280E337"))
objc.msg_send("XYSSHandle", "setToken:", pyobj2nsobj(emu,
                                                     "snFxyd29qfTBy1PRP4vNvhD2NU4pon75wsIrrZQTR3n6MgwmC3vh9hSLJTqbC4kM8+AdHzW93wplGjU+zZbwiGYUQE8nbB49FTj0alqXaX0zJ+Msj3dNPNPoTcpXB34G"))


```

## 主动调用

### 构造 NSMutableURLRequest 对象

这里还是比较复杂一点，根据 frida-trace 的代码得知，需要传入一个 NSMutableURLRequest 对象，需要构造。问下 ai![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHZrgS70ppeHrvia6Z0h7TW02C2eWtAFvzph4lyZchVHR5EImW88ibPayK9SvouWqZvjYiatx1Oq2IaLA/640?wx_fmt=png&from=appmsg) 如下代码就构造好 req 对象。

```
NSURL_addr = objc.msg_send("NSURL", "URLWithString:", pyobj2nsobj(emu,
                                                                  "https://www.xiaohongshu.com/api/im/v2/messages/unread"))
req_obj = objc.msg_send("NSMutableURLRequest", "requestWithURL:", NSURL_addr)
# req_obj = objc.msg_send(req_obj, "path:", NSURL_addr)


```

### 构造 Headers

直接使用作者的 pyobj2nsobj，如下代码就是将 py 的 dict 转为 oc 里面的 dic

```
  if isinstance(obj, dict):
        ns_obj = objc.msg_send("NSMutableDictionary", "dictionary")

        for key, value in obj.items():
            ns_key = pyobj2nsobj(emu, key)
            ns_value = pyobj2nsobj(emu, value)

            objc.msg_send(ns_obj, "setObject:forKey:", ns_value, ns_key)


```

下面就是算法入参

```
 pyobj2nsobj(emu, {
                                    "xy-platform-info": "2",
                                    "xy-common-params": "1",
                                })


```

### 算法调用

上边的构造传入，并主动调用之后就会得出结果。

```
XYSSHandle_addr = objc.msg_send("XYSSHandle", "reqAuthorityWithHeaders:request:bodyData:",
                                pyobj2nsobj(emu, {
                                    "xy-platform-info": "2",
                                    "xy-common-params": "1",
                                }), req_obj, pyobj2nsobj(emu, "111".encode()))
print("Shield result: %s", emu.read_string(objc.msg_send(XYSSHandle_addr, "cStringUsingEncoding:", 4)))


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHZrgS70ppeHrvia6Z0h7TW02sTInQHbs0scHAXFXVrdXiaDll2cpuZ3Zbqjd6ZMECYb9bAVp7Dibzeog/640?wx_fmt=png&from=appmsg)file

参考资料

[1] 

某手 sig3-ios 算法 Chomper 黑盒调用: _https://zhaoxincheng.com/index.php/2025/02/20/%E6%9F%90%E6%89%8Bsig3-ios%E7%AE%97%E6%B3%95%E4%BD%BF%E7%94%A8chomper%E8%B0%83%E7%94%A8/_