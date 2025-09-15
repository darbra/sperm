> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uaNC3KV3IExL4d1-SAqShA)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicPzjYzt97VqJgyEYt537Ps6zyZzR9GDCgkQB8NqgsPvOuJpyl3VNzeg/640?wx_fmt=png&from=appmsg#imgIndex=0)

声明
--

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

**本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号【K 哥爬虫】联系作者立即删除！**

逆向目标
----

*   目标：某品会 APP
    
*   apk 版本：v9.42.8
    
*   逆向参数：authorization
    
*   下载地址：`aHR0cHM6Ly93d3cud2FuZG91amlhLmNvbS9hcHBzLzMxNTgzL2hpc3Rvcnlfdjk0MjA4`
    

抓包分析
----

打开 app 随便搜索一样东西，然后通过 charles 配合 SocksDroid 进行抓包，结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicYoCz3tAtJN0XFQNPaEcZQbY2wWdMJ9ZTzLDnGNicljlfV17Lmn1qu6g/640?wx_fmt=png&from=appmsg#imgIndex=1)

python 重放请求，发现请求头需要 authorization 参数。

java 层分析
--------

这个版本有 frida 检测，还是老方法，hook dlopen 查看在哪个 app 闪退了：

```
function hook_dlopen() {
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            var path_ptr = args[0];
            var path = ptr(path_ptr).readCString();
            console.log("[android_dlopen_ext -> enter", path);
        },
        onLeave: function (retval) {
            console.log("android_dlopen_ext -> leave")

        }
    });
}
setImmediate(hook_dlopen)


```

```
frida -U -f com.achievo.vipshop -l .\demo.js


```

上面代码启动，发现还是 `libmsaoaidsec.so` 检测，关于该检测，往期文章中有详细的介绍：

> 某瓣 app 逆向分析：https://mp.weixin.qq.com/s/i_k_QOfgAV33_2u9T4OnxA

把该 so 文件丢到 ida 工具，直接通过交叉引用，找到上层的调用函数 hook 掉就行：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicmOfnNicg7tquzOa3IC7ccnzojmibI6MjCXXYPUm4q5vTRPd1MMNncTFw/640?wx_fmt=png&from=appmsg#imgIndex=2)

hook 检测代码如下：

```
function hook_dlopen() {
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            var path_ptr = args[0];
            var path = ptr(path_ptr).readCString();
            console.log("[android_dlopen_ext -> enter", path);
        },
        onLeave: function (retval) {
            console.log("android_dlopen_ext -> leave")

        }
    });
}

function hook_call_constructors() {
    var linker64_base_addr = Module.getBaseAddress("linker64")
    var call_constructors_func_off = 0x000000000004a174
    var call_constructors_func_addr = linker64_base_addr.add(call_constructors_func_off)
    var listener = Interceptor.attach(call_constructors_func_addr, {
        onEnter: function (args) {
            console.log("call_constructors -> enter")
            varmodule = Process.findModuleByName("libmsaoaidsec.so")
            if (module != null) {
                Interceptor.replace(module.base.add(0x000000000001B924), new NativeCallback(function () {
                    console.log("replace sub_1BEC4")
                }, "void", []))
                listener.detach()
            }
        },
    })
}

setImmediate(hook_dlopen)


```

注意 `call_constructors_func_off` 的地址需要改成大伙自己的，这个地址可以在手机执行以下命令得到：

```
readelf -s /apex/com.android.runtime/bin/linker64 | grep call_constructors


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXiciaEeVib0oTakkKiaMpqDIZHuXLeMibOUDCgNXAmmqsHRBRibibbXjCTEgUicw/640?wx_fmt=png&from=appmsg#imgIndex=3)

这个参数在头部，因此我们优先选择 hook 头部，而不是 hook HashMap，hashMap 输出内容较多，当我们 hook 头部没有相关信息时，可以再考虑 hook hashMap，hook 代码如下：

```
function showStacks() {
    Java.perform(function () {
        console.log("打印堆栈")
        console.log(Java.use("android.util.Log").getStackTraceString(
            Java.use("java.lang.Throwable").$new()
        ));
    })
}

function hook_Header(){
    var Builder = Java.use("okhttp3.Request$Builder");
    Builder["addHeader"].implementation = function (str, str2) {
        console.log("key: " + str)
        console.log("val: " + str2)
        // showStacks()
        if (str ==="Authorization"){
            showStacks()
        }
        var result = this["addHeader"](str, str2);
        return result;
    };
}


```

```
frida -U -f com.achievo.vipshop -l .\demo.js -o 1.txt


```

找到我们需要的参数的值：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicVKYEvZ03aIB5BLQkSBeXTq2mjR3oElb3kiccr4NAKdicMTHF247EIvhg/640?wx_fmt=png&from=appmsg#imgIndex=4)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicicLDRmsvvNY7HuJtGdEX7Z9ib0xEhGmyaJ5iatHp5KU0I7zqFJ7KxBzfw/640?wx_fmt=png&from=appmsg#imgIndex=5)

apk 文件放入 jadx，根据第二层堆栈找到如下位置：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicfx1aGDFoatdcZLjm3yCKOOc5icqoq8iaviawo6djwXaK95lpUPqXYvDrA/640?wx_fmt=png&from=appmsg#imgIndex=6)

值是 str，str 又通过 `b.b` 方法得到的，我们这里是 post 方法就走 if 逻辑，hook 一下：

```
let b = Java.use("a9.b");
b["b"].implementation = function (context, treeMap, str, str2) {
    console.log(`b.b is called: context=${context}, treeMap=${treeMap}, str=${str}, str2=${str2}`);
    let result = this["b"](context, treeMap, str, str2);
    console.log(`b.b result=${result}`);
    return result;
};


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicmYhmVQPibmSiaia8l2RvMLHhLibLn4u47gQj6pzLnSrGic9LmQy5PGIJZWA/640?wx_fmt=png&from=appmsg#imgIndex=7)

位置没错，进入方法：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicBDsRIqflMv68MpgwqfnFr1gVk62zIa9x3FWEmic0ycloV96t8tYicUYA/640?wx_fmt=png&from=appmsg#imgIndex=8)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXic0tff2psDXypwCQheeHlgJFOMJic28B2TIYZt8qhyzrf84icgQRE0v9icQ/640?wx_fmt=png&from=appmsg#imgIndex=9)

一直走，最后定位到了这里：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicE8XK0en08u4WaksYBz8pDnnGp6szqiaGJftgAS2HEEia9ZZcJ4q3ocww/640?wx_fmt=png&from=appmsg#imgIndex=10)

进入 getSignHash 方法然后调用了 gs 方法，但是 gs 方法是报错，按道理 gs 函数应该有返回值，返回我们的加密结果，我们可以直接把代码丢到 GPT 让他帮忙分析一下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicWTxlfX88SyzibDcKoQhiapuJZtY6nnrNM7KZpLmdQNWzPicDBGa7xKrpA/640?wx_fmt=png&from=appmsg#imgIndex=11)

gpt 给了我们答案，说是通过反射调用了一个隐藏的方法，那我们先了解一下 java 反射是什么：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicKhianSFuHErNtWXsR6dTo4XWgMxXneKskEf9XCYCk4cYY3pZNTW0OGw/640?wx_fmt=png&from=appmsg#imgIndex=12)

上面也说了 java 的反射创建类不通过 new，而是通过 newInstance 方法创建，那我们向下翻，会看到 initInstance 方法：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicZzNVoPzW084O8icPqoicFWvTLZ2iaBWRO5JibUgoSNicTXLP0sIKdSAXcdA/640?wx_fmt=png&from=appmsg#imgIndex=13)

根据上面了解到的反射的相关知识，这里很明显是在创建对象，我们直接点击 KeyInfo 这个类，找到真正的加密函数 gs：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicYNiacSrr89RedpG3Y53ezGnKvZ92HTetVWAEfP2jVH3ZicrSAJwUMWXw/640?wx_fmt=png&from=appmsg#imgIndex=14)

最后调用了 native 方法，so 的名称为 keyinfo：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXictMQ3LAenw9MWNicpnWmnibTxxIjFTnoofrMhlibLiaeTXt4libibwFwOGMYA/640?wx_fmt=png&from=appmsg#imgIndex=15)

我们可以 hook 验证一下：

```
let KeyInfo = Java.use("com.vip.vcsp.KeyInfo");
KeyInfo["gsNav"].implementation = function (context, map, str, z10) {
    console.log(`KeyInfo.gsNav is called: context=${context}, map=${map}, str=${str}, z10=${z10}`);
    let result = this["gsNav"](context, map, str, z10);
    console.log(`KeyInfo.gsNav result=${result}`);
    return result;
};


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicicHS7ic19jrO6j9LP3ia0JLvAUA7SowdFsOD8VR7DOpiaPmsECW1e2HcDA/640?wx_fmt=png&from=appmsg#imgIndex=16)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXic4ka29pkb0wx6wDia1bvVYCibTweASUFiabT22wF9AWOUPAjWnEzYMeYxw/640?wx_fmt=png&from=appmsg#imgIndex=17)

最后改成主动调用的形式，方便我们后续分析：

```
function hook_zhu() {
    Java.perform(function () {
        const KeyInfo = Java.use("com.vip.vcsp.KeyInfo");
        const TreeMap = Java.use("java.util.TreeMap");
        constString = Java.use('java.lang.String');
        // 构造 context
        const currentApplication = Java.use("android.app.ActivityThread").currentApplication();
        const context = currentApplication.getApplicationContext();
        let map = TreeMap.$new();
        map.put(String.$new("api_key"), String.$new("23e7f28019e8407b98b84cd05b5aef2c"));
        map.put(String.$new("app_name"), String.$new("shop_android"));
        map.put(String.$new("app_version"), String.$new("9.42.8"));
        map.put(String.$new("bigSaleTagIds"), String.$new(""));
        map.put(String.$new("brandIds"), String.$new(""));
        map.put(String.$new("brandStoreSns"), String.$new(""));
        map.put(String.$new("categoryId"), String.$new(""));
        map.put(String.$new("channelId"), String.$new("1"));
        map.put(String.$new("channel_flag"), String.$new("0_1"));
        map.put(String.$new("clickFrom"), String.$new("list_userword"));
        map.put(String.$new("client"), String.$new("android"));
        map.put(String.$new("client_type"), String.$new("android"));
        map.put(String.$new("couponIds"), String.$new(""));
        map.put(String.$new("darkmode"), String.$new("0"));
        map.put(String.$new("deeplink_cps"), String.$new(""));
        map.put(String.$new("device_model"), String.$new("Google Pixel 3"));
        map.put(String.$new("did"), String.$new("0.0.488b303f75e2323522a8ed6c9da1d86f.2e28ab"));
        map.put(String.$new("elder"), String.$new("0"));
        map.put(String.$new("evgid"), String.$new("itg0e6C7desc9WOI7whauxkrce1Tnmectkj0f9gbLh7l8Yy8fZS8ASg24uPgOT1UB+J9Pn37EebSRw/foKbqomTNnj0y5EDKLAXFfooxVlQ="));
        map.put(String.$new("extParams"), String.$new("{\"priceVer\":\"2\",\"video_playable\":\"1\",\"mclabel\":\"1\",\"cmpStyle\":\"1\",\"statusVer\":\"2\",\"ic2label\":\"1\",\"video\":\"2\",\"uiVer\":\"2\",\"preheatTipsVer\":\"4\",\"floatwin\":\"1\",\"superHot\":\"1\",\"exclusivePrice\":\"1\",\"router\":\"1\",\"coupons\":\"5\",\"needVideoExplain\":\"1\",\"rank\":\"2\",\"needVideoGive\":\"1\",\"attr\":\"2\",\"bigBrand\":\"2\",\"couponVer\":\"v2\",\"videoExplainUrl\":\"1\",\"live\":\"1\",\"sellpoint\":\"1\",\"reco\":\"1\",\"vreimg\":\"1\",\"search_tag\":\"2\",\"tpl\":\"1\",\"ads\":\"2\",\"stdSizeVids\":\"\",\"labelVer\":\"2\",\"preheatView\":\"1\"}"));
        map.put(String.$new("fdc_area_id"), String.$new("104104"));
        map.put(String.$new("functions"), String.$new("RTRecomm,flagshipInfo,couponBarV2,lowPriceTabs,feedbackV2,otdAds,zoneCode,slotOp,survey,outfit,aiRealtime,floaterParams,tabGroupV2,bsAndSeason,propAndOpTag,parallelCall"));
        map.put(String.$new("harmony_app"), String.$new("0"));
        map.put(String.$new("harmony_os"), String.$new("0"));
        map.put(String.$new("height"), String.$new("2028"));
        map.put(String.$new("isMultiTab"), String.$new("0"));
        map.put(String.$new("is_default_area"), String.$new("1"));
        map.put(String.$new("keyword"), String.$new("纸"));
        map.put(String.$new("lastPageProperty"), String.$new("{\"isBgToFront\":\"0\",\"suggest_text\":\"纸\",\"scene_entry_id\":\"-99\",\"refer_page_id\":\"page_te_globle_classify_search_1749796272747\",\"isSimple\":\"0\",\"text\":\"纸\",\"tag\":\"1\",\"module_name\":\"com.achievo.vipshop.search\",\"type\":\"all\",\"typename\":\"全部\",\"is_back_page\":\"1\"}"));
        map.put(String.$new("maker"), String.$new("GOOGLE"));
        map.put(String.$new("mars_cid"), String.$new("2760c2a5-07c5-3a1c-a409-66e37ebaf574"));
        map.put(String.$new("mobile_channel"), String.$new("rjx5hknt:::"));
        map.put(String.$new("mobile_platform"), String.$new("3"));
        map.put(String.$new("net"), String.$new("WIFI"));
        map.put(String.$new("operator"), String.$new(""));
        map.put(String.$new("os"), String.$new("Android"));
        map.put(String.$new("osv"), String.$new("11"));
        map.put(String.$new("otddid"), String.$new(""));
        map.put(String.$new("other_cps"), String.$new(""));
        map.put(String.$new("page_id"), String.$new("page_te_commodity_search_1749796274004"));
        map.put(String.$new("page_init_ts"), String.$new("1749796253199"));
        map.put(String.$new("phone_brand"), String.$new("google"));
        map.put(String.$new("phone_model"), String.$new("pixel 3"));
        map.put(String.$new("priceMax"), String.$new(""));
        map.put(String.$new("priceMin"), String.$new(""));
        map.put(String.$new("props"), String.$new(""));
        map.put(String.$new("province_id"), String.$new("104104"));
        map.put(String.$new("referer"), String.$new("com.achievo.vipshop.search.activity.SearchActivity"));
        map.put(String.$new("rom"), String.$new("Dalvik/2.1.0 (Linux; U; Android 11; Pixel 3 Build/RQ1D.210205.004)"));
        map.put(String.$new("sd_tuijian"), String.$new("0"));
        map.put(String.$new("service_provider"), String.$new(""));
        map.put(String.$new("session_id"), String.$new("2760c2a5-07c5-3a1c-a409-66e37ebaf574_shop_android_1749796345730"));
        map.put(String.$new("skey"), String.$new("6692c461c3810ab150c9a980d0c275ec"));
        map.put(String.$new("sort"), String.$new("0"));
        map.put(String.$new("source"), String.$new("app"));
        map.put(String.$new("source_app"), String.$new("android"));
        map.put(String.$new("standby_id"), String.$new("rjx5hknt:::"));
        map.put(String.$new("sys_version"), String.$new("30"));
        map.put(String.$new("tabFields"), String.$new("gender,tabs,priceTabs,discountTabs,tabGroupV2"));
        map.put(String.$new("tfs_fp_token"), String.$new("Ba1bX4G5LVCvZX/keQ+jKakGAtkmcd5uF3mqx1MgFxED+ki6Tt8pN7kHPc01QhDUUEKPZujxXveye0E3jIKnUvA=="));
        map.put(String.$new("timestamp"), String.$new("1749796274"));
        map.put(String.$new("union_mark"), String.$new("blank&_&blank&_&rjx5hknt:::&_&blank&_&blank&_&blank"));
        map.put(String.$new("vipService"), String.$new(""));
        map.put(String.$new("warehouse"), String.$new("VIP_NH"));
        map.put(String.$new("width"), String.$new("1080"));

        let result = KeyInfo["gsNav"](context, map, null, false);
        console.log("java---->", result);
    })
}


```

so 层分析
------

老样子，keyinfo.so 文件拖到 ida 里面去，在 Exports 表搜索 java 看看是不是静态注册的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicuESiaBpjaPWY97ibsAAbTTvA3MpV8XNeYIOahRaR811UH9IcicC2pWcHQ/640?wx_fmt=png&from=appmsg#imgIndex=18)

是静态注册，点进去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXic8EKfN4U2HIDIEWTYJVuf7MtIEOBQJ5EfvojMDYVD9hHGRcPibjia6Aqw/640?wx_fmt=png&from=appmsg#imgIndex=19)

返回 v7，点进去 `Function_gs`，并按 y 修改 a1 类型：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXica2flAqsaJfwqk4prOXxHsmwYRrCjLtJ6WaMj6JQQowck8wTfPK7FTA/640?wx_fmt=png&from=appmsg#imgIndex=20)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicsUB2bcSvnFxlNlibgdHujBibeeWRbUwQXDXn9SBYVNkZLRRHE6xfEHXw/640?wx_fmt=png&from=appmsg#imgIndex=21)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicKjfx8bo8vGCCxdkHOl5BGYvAph8QYCxNmWeib59ypD2NTAG8ICrfPEQ/640?wx_fmt=png&from=appmsg#imgIndex=22)

代码没有什么混淆，基本上全是明文，遇到这种，我们可以找根据名字定位，也可以 frida trace 看看调用堆栈，因为这代码没多少行，我们可以直接丢给 ai 帮我们分析，这里用的元宝的 deepseek：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicSkjuvI3n6ks1qLdIicTPxfdibLUv5Ul3huzmXlH9OfuqHjxHwkDfHSPw/640?wx_fmt=png&from=appmsg#imgIndex=23)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicLicaAbdqnHhwoXKuHfkfkJJIBH5Pbmkg4heTAy4w8IVWBj253qs6ubA/640?wx_fmt=png&from=appmsg#imgIndex=24)

根据 ai 说的，我们只需要关心 getByteHash 函数做了什么操作，先找到 getByteHash 函数看看，按住 alt + T 搜索 getByteHash，有两处，点进去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicoRFiaGRy3O7JkpNicY2CPp2qIXERCt6mZtacgnGqj5Wft6tibzX6AE5ww/640?wx_fmt=png&from=appmsg#imgIndex=25)

很明显的 sha1 算法，还是直接先 hook 一下 getByteHash 方法，看看传递了什么参数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicicOYJjNnO5CEY5608iaJ5icc3a76Aa7SwKNhKu96p7d1lTyIpf79y7ia7Q/640?wx_fmt=png&from=appmsg#imgIndex=26)

我们重点看第三个参数和第五个参数就行，hook 代码如下：

```
function hook_so_byhash() {
    var addr = Module.findBaseAddress('libkeyinfo.so');
    var func = addr.add(0xF2260)
    // console.log(func)
    let number = 0;
    Interceptor.attach(func, {
        onEnter: function (args) {
            number += 1;
            console.log(`\x1b[31m第${number} 次 hook getByteHash-----------------------------------------------\x1b[0m`);
            this.arg5 = args[4]
            console.log(hexdump(args[2]))
            console.log(args[2].readCString())
        },
        onLeave: function (retval) {
            console.log("this.arg5",hexdump(this.arg5));
        }
    })
}

setImmediate(hook_so_byhash)


```

在 frida 里面执行我们的主动调用：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicrDWTicUgtXBV4tiaDtZxCCiaicxZaeV7DBDNibDGdtcUrDwwtXObLPnrSYw/640?wx_fmt=png&from=appmsg#imgIndex=27)

打印结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicWaI4MKvyfMCDnu28WXic668WgQSibx5CMibyDnltQUWhrJewHv6m7ibwCQ/640?wx_fmt=png&from=appmsg#imgIndex=28)

第一次，输出的参数暂且不知道是什么，返回的结果为 `1ed562e1e90b23ae3f9a40f8b2a65382b95a4752`：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicA1Fjgj6q6Ydp7o4DLIRoFjz7LwZiaEcgEolXAHXsI5UEk2gsflcJBOA/640?wx_fmt=png&from=appmsg#imgIndex=29)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicAIfjnFYOEwfRyibkfuz6DqtSVt9VzniaADiacQPydibcRZClWOpSOQXvgg/640?wx_fmt=png&from=appmsg#imgIndex=30)

第二次入参为我们的 TreeMap 数据，前面盐值为 `aee4c425dbb2288b80c71347cc37d04b`，经分析，该值写死即可，返回值为 `d98aee7e972029d163709d41538d5bdcb2fe290f`。

我们到 K 哥工具站（https://www.kgtools.cn），对比发现，并没有魔改，标准算法：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicwR0CoBxHq22swnPktvOpQ9sjMb4ib40zM8lpIqqxZGVTEAkkFnFT0fQ/640?wx_fmt=png&from=appmsg#imgIndex=31)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXic4GDlJicVR2SJIQsic2lgibX9xhUoToXyUIkalTia3klao7LbHaT6Bmmo2g/640?wx_fmt=png&from=appmsg#imgIndex=32)

第三次的入参为我们前面的盐加上第二次 sha1 算法加密的返回的结果。

第三次返回的结果为 `0ae492780a1e7aefafa23efae2357faafc98af51`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicSvZltCAkibgTWLUpoxNic6zCYMmKV4zj5vq5oWStQJyhJUG7BU7xpP8Q/640?wx_fmt=png&from=appmsg#imgIndex=33)

和我们主动调用的结果一样，最后用 python 改写发包即可，至此，加密参数分析完毕，相关代码，会放到知识星球中，仅供学习交流。

结果验证
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTpcbibsziaD9IicmNjmeW5vLXicOWe9uQJdQttovSMAFcbQribb0kduFtWXQXEjR8Qnk1anQlH1fyj7ibDw/640?wx_fmt=png&from=appmsg#imgIndex=34)

 ![](https://mmbiz.qpic.cn/sz_mmbiz_gif/iabtD4jabia4KDdF6jxLibSq5ssaiaicicKHf2VWdrkFqrsRuDF7CiaKMxAeua0WeLPFmOIQkgcCt66o7L2uOl1wRVuVw/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=35)

**（ 先别划走 点个关注 Orz ）**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqk3p9iaszC2ibDWOXPQ3e0aCy3zsOLCDOV6ZbGbedyRNqfsqWUODEFC5B4nnbhAiaKmslJL07ruia4og/640?wx_fmt=png&from=appmsg#imgIndex=36)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqRIkUKs0D0Yt8cHdIumAAg9cLcxz0cztZiaWDyxEDjTdbKjruhZNxHJG4IyORoBmMZsUeYqHjlVibA/640?wx_fmt=png&from=appmsg#imgIndex=37)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqNYp1HRxOBplkftyRNyiatRiaicpeopJPGhrmnTedrIwSjnEZbWTR608ibMMDOUFHgINqnXiadxoNWqyw/640?wx_fmt=png&from=appmsg#imgIndex=38)

**[代理 IP 推荐](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)**

**[快 代 理](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)**

[11 年来专注企业代理 IP 云服务！](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)

[先进大数据采集团队和爬虫工程师的优先选择！](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)

[满足](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)[人工智能 数据采集 跨境电商 市场研究 网络安全 媒体矩阵 影音娱乐](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)[等](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)[广泛应用场景 ！](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/7VAgNKQgCMFM8ia5BA9MLZhlCnRr8Er4gR4Rjr7WBmby6jKvlqpH7jZITFBYBIYbibfOgHRCF5obiaJn6yzC321qw/640?wx_fmt=png#imgIndex=39)

**点个****推荐♡****求求你啦**

![](https://mmbiz.qpic.cn/mmbiz_png/NtgFk2rGpiaOPxvr7Ls916UDdGAibFN8ObxF6VKc8qCT18luCwKTUgHicBiaMYJE9SIdicQHL7ouCt8xk7tMtsxKayA/640?wx_fmt=png#imgIndex=40)