> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/E5lbFrVyBR0qumJIiW_3Tg)

> 免责声明：文章中所有内容仅供学习交流使用，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【CYRUS STUDIO】联系作者

搜索接口抓包分析
========

抓包数据如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXcA5icGOCwX3LWUSazOxqFSDnHgbaeicv9aVJHgbhyOPUGLfV0BhwcG9Nw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image1.png

  
Request 和 Response 的 Body 被加密无法直接查看。

请求地址：

```
https://***************************/aweme/v2/search/general/single/?klink_egdi=********************************************************&iid=****************&device_id=****************&ac=****&channel=*******************&aid=****&app_name=*****&version_code=******&version_name=******&device_platform=*******&os=*******&ssmix=*&device_type=*****&device_brand=******&language=**&os_api=**&os_version=**&manifest_version_code=******&resolution=*********&dpi=***&update_version_code=********&_rticket=*************&package=************************&first_launch_timestamp=**********&last_deeplink_update_version_code=*&cpu_support64=****&host_abi=*********&is_guest_mode=*&app_type=******&minor_status=*&appTheme=*****&is_preinstall=*&need_personal_recommend=*&is_android_pad=*&is_android_fold=*&ts=**********&cdid=************************************

```

请求头：

```
:method: POST
:authority: ***************************
:scheme: https
:path: /aweme/v2/search/general/single/?klink_egdi=********************************************************&iid=****************&device_id=****************&ac=****&channel=*******************&aid=****&app_name=*****&version_code=******&version_name=******&device_platform=*******&os=*******&ssmix=*&device_type=*****&device_brand=******&language=**&os_api=**&os_version=**&manifest_version_code=******&resolution=*********&dpi=***&update_version_code=********&_rticket=*************&package=************************&first_launch_timestamp=**********&last_deeplink_update_version_code=*&cpu_support64=****&host_abi=*********&is_guest_mode=*&app_type=******&minor_status=*&appTheme=*****&is_preinstall=*&need_personal_recommend=*&is_android_pad=*&is_android_fold=*&ts=**********&cdid=************************************
content-length: ****
cookie: ****************************************************
cookie: ************************************************************
cookie: ******************
cookie: ********************
cookie: ************************************************************************************************************************************************************************
cookie: ***************************
cookie: ************************************************
x-tt-dt: ******************************************************************************************************************************************************
activity_now_client: *************
compressed-bcm-chain: ******************************************************************************************************************************************************************************************************************************************************************************
x-ss-req-ticket: *************
sdk-version: *
passport-sdk-version: *****
x-vc-bdturing-sdk-version: ********
content-type: ************************************************
x-ss-stub: ******************************
x-bd-content-encoding: ****
x-tt-store-region: *****
x-tt-store-region-src: ***
x-tt-request-tag: ********
x-ss-dp: ****
x-tt-trace-id: *******************************************************
user-agent: ********************************************************************************************************************************************************************
ttzip-version: **********
ttzip-tlb: *
accept-encoding: ************************
x-argus: ********
x-gorgon: ****************************************************
x-helios: ************************************************
x-khronos: **********
x-ladon: ********
x-medusa: ************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
x-soter: ************************************************************************************************************************

(�/�`�=�{

```

curl 请求如下：

```
curl -H "Host: ***************************" -H "Cookie: *********************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************" -H "x-tt-dt: ******************************************************************************************************************************************************" -H "activity_now_client: *************" -H "compressed-bcm-chain: ******************************************************************************************************************************************************************************************************************************************************************************" -H "x-ss-req-ticket: *************" -H "sdk-version: *" -H "passport-sdk-version: *****" -H "x-vc-bdturing-sdk-version: ********" -H "content-type: ************************************************" -H "x-ss-stub: ******************************" -H "x-bd-content-encoding: ****" -H "x-tt-store-region: *****" -H "x-tt-store-region-src: ***" -H "x-tt-request-tag: ********" -H "x-ss-dp: ****" -H "x-tt-trace-id: *******************************************************" -H "user-agent: ********************************************************************************************************************************************************************" -H "ttzip-version: **********" -H "ttzip-tlb: *" -H "x-argus: ********" -H "x-gorgon: ****************************************************" -H "x-helios: ************************************************" -H "x-khronos: **********" -H "x-ladon: ********" -H "x-medusa: ************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************" -H "x-soter: ************************************************************************************************************************" --data-binary "(�/�`�=�{

```

Response：

```
:status: ***
server: Tengine
content-type: application/json; charset=utf-8
date: *****************************
vary: Accept-Encoding
x-tt-logid: **********************************
status_code: 0
bd-tt-error-code: 0
tt_stable: 1
strict-transport-security: ***********************************
tt-idc-switch: ********************
server-timing: ***************
x-tt-trace-host: ************************************************************************************************************************************************************************************************
x-tt-trace-tag: *****************************
x-tt-trace-id: *******************************************************
content-encoding: ttzip
ttzip-version: **********
server-timing: **********************************************
via: **********************
timing-allow-origin: *
eagleid: **************************

(�/��b� i/

```

相关文章：

*   • 如何实现 Android App 的抓包防护？又该如何绕过？一文看懂攻防博弈 [1]
    
*   • 安卓抓包实战：使用 Charles 抓取 App 数据全流程详解 [2]
    

Dex 脱壳与 TTNet 源码分析
==================

使用 frida_dex_dump[3] 脱壳 dex ，再用 dex2jar[4] 把 dex 批量转换成 jar。

TTNet 是 **** 基于 Retrofit 深度改造的闭源网络框架，集成请求加密、拦截器链路、性能埋点和客户端密钥机制，广泛用于其内部 App 的网络通信。

使用 find_in_jars.py[5] 查找 ttnet 包下的类：

```
(anti-app) PS D:\Python\anti-app\dex2jar> python find_in_jars.py "D:\Python\anti-app\app\douyin\dump_dex\jar" "com.*********.ttnet"
[+] Searching for class prefix: com/*********/ttnet
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes2.jar → com/*********/ttnet/TTNetInit$ENV.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes23.jar → com/*********/ttnet/debug/DebugSetting.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/http/HttpRequestInfo.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/http/RequestContext.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/HttpClient.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/ITTNetDepend.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/TTALog.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/TTMultiNetwork.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/TTNetInit.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/clientkey/ClientKeyManager.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/config/AppConfig.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/config/TTHttpCallThrottleControl$DelayMode.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/config/TTHttpCallThrottleControl.class        
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/cronet/AbsCronetDependAdapter.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/diagnosis/TTNetDiagnosisService.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/priority/TTHttpCallPriorityControl$ModeType.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/priority/TTHttpCallPriorityControl.class      
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/retrofit/SsInterceptor.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/retrofit/SsRetrofitClient.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/throttle/TTNetThrottle.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/tnc/TNCManager$TNCUpdateSource.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/tnc/TNCManager.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/utils/RetrofitUtils$CompressType.class        
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes37.jar → com/*********/ttnet/utils/RetrofitUtils.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes47.jar → com/*********/ttnet/diagnosis/IDiagnosisRequest.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes47.jar → com/*********/ttnet/diagnosis/IDiagnosisCallback.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes47.jar → com/*********/ttnet/diagnosis/TTGameDiagnosisService.class        
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes49.jar → com/*********/ttnet/http/IRequestHolder.class
[✓] Found in: D:\Python\anti-app\app\douyin\dump_dex\jar\base.apk_classes7.jar → com/*********/ttnet/INetworkApi.class
[+] Total 29 match(es) found.

```

找到 TTNet 的拦截器：com/*********/ttnet/retrofit/SsInterceptor

jar 文件检索工具：打造自己的 Jar 文件分析工具：类名匹配 + 二进制搜索 + 日志输出全搞定 [6]

SsInterceptor 方法调用分析
--------------------

使用 frida hook 一下 SsInterceptor 中的所有方法

```
/**
 * Hook 指定类的所有方法（每个方法所有重载）
 * @param {string} className - Java 类的完整名
 */
function hook_all_methods(className) {
    Java.perform(function () {
        var clazz = Java.use(className);
        var methods = clazz.class.getDeclaredMethods(); // 反射获取所有声明的方法

        var hooked = new Set(); // 用于避免重复 hook 相同方法名（因为多重载）

        methods.forEach(function (m) {
            var methodName = m.getName();

            // 如果这个方法已经 Hook 过，就跳过
            if (hooked.has(methodName)) return;
            hooked.add(methodName);

            try {
                hook_method(className, methodName);
            } catch (e) {
                console.error("❌ Failed to hook " + methodName + ": " + e);
            }
        });
    });
}


setImmediate(function () {
    hook_all_methods("com.*********.ttnet.retrofit.SsInterceptor");
});


// frida -H 127.0.0.1:1234 -F -l hook_class_methods.js

```

日志输出如下：

```
================= HOOK START =================
🎯 Class: com.*********.ttnet.retrofit.SsInterceptor
🔧 Method: tryFilterDupQuery
📥 Arguments:
  [0]: https://*************/aweme/v1/search/middle_page_tabs/?need_personal_recommend=*&is_login=*&is_vcd=*&request_tag_from=**&klink_egdi=********************************************************&iid=****************&device_id=****************&ac=****&channel=*******************&aid=****&app_name=*****&version_code=******&version_name=******&device_platform=*******&os=*******&ssmix=*&device_type=*****&device_brand=******&language=**&os_api=**&os_version=**&manifest_version_code=******&resolution=*********&dpi=***&update_version_code=********&_rticket=*************&package=************************&first_launch_timestamp=**********&last_deeplink_update_version_code=*&cpu_support64=****&host_abi=*********&is_guest_mode=*&app_type=******&minor_status=*&appTheme=*****&is_preinstall=*&is_android_pad=*&is_android_fold=*&ts=**********&cdid=************************************     
📤 Return value: https://*************/aweme/v1/search/middle_page_tabs/?need_personal_recommend=*&is_login=*&is_vcd=*&request_tag_from=**&klink_egdi=********************************************************&iid=****************&device_id=****************&ac=****&channel=*******************&aid=****&app_name=*****&version_code=******&version_name=******&device_platform=*******&os=*******&ssmix=*&device_type=*****&device_brand=******&language=**&os_api=**&os_version=**&manifest_version_code=******&resolution=*********&dpi=***&update_version_code=********&_rticket=*************&package=************************&first_launch_timestamp=**********&last_deeplink_update_version_code=*&cpu_support64=****&host_abi=*********&is_guest_mode=*&app_type=******&minor_status=*&appTheme=*****&is_preinstall=*&is_android_pad=*&is_android_fold=*&ts=**********&cdid=************************************
================== HOOK END ==================


================= HOOK START =================
🎯 Class: com.*********.ttnet.retrofit.SsInterceptor
🔧 Method: intercept
📥 Arguments:
  [0]: com.*********.retrofit2.client.Request@1d768f
📤 Return value: com.*********.retrofit2.client.Request@c084623
================== HOOK END ==================


================= HOOK START =================
🎯 Class: com.*********.ttnet.retrofit.SsInterceptor
🔧 Method: intercept
📥 Arguments:
  [0]: com.*********.retrofit2.client.Request@c084623
  [1]: com.*********.retrofit2.SsResponse@2049c2f
📤 Return value: undefined
================== HOOK END ==================

```

主要调用了 SsInterceptor 的下面 3 个方法：

*   • tryFilterDupQuery：去除重复 query 参数，构造新的 URL
    
*   • intercept(Request request) 是 “请求拦截器” 方法，用于修改或替换请求内容，添加 headers、加密参数、签名等
    
*   • intercept(Request, SsResponse) 是 “响应拦截器”，用于观察或处理网络响应（如打点统计、异常分析）。
    

SsInterceptor、BaseSsInterceptor 与 Interceptor
---------------------------------------------

SsInterceptor、BaseSsInterceptor 与 Interceptor 之间的关系大概如下：

```
Interceptor (接口)
   ↑
BaseSsInterceptor（抽象类/实现类，实现了一些公共逻辑）
   ↑
SsInterceptor（实现类，实现实际请求拦截与处理）

```

SsInterceptor 代码分析
------------------

```
package com.*********.ttnet.retrofit;

import O.O;
import X.C1465814hj;
import X.C1477714je;
import X.C1478114ji;
import X.C16130FjH;
import android.os.SystemClock;
import android.text.TextUtils;
import android.util.Pair;
import com.GlobalProxyLancet;
import com.*********.common.profilesdk.ProfileManager;
import com.*********.common.utility.StringUtils;
import com.*********.frameworks.baselib.network.http.retrofit.BaseSsInterceptor;
import com.*********.frameworks.baselib.network.http.util.UrlBuilder;
import com.*********.frameworks.baselib.network.http.util.UrlUtils;
import com.*********.frameworks.core.encrypt.RequestEncryptUtils;
import com.*********.qss.common.catcher.CatcherTrace;
import com.*********.retrofit2.RetrofitMetrics;
import com.*********.retrofit2.client.Header;
import com.*********.retrofit2.client.Request;
import com.*********.ttnet.clientkey.ClientKeyManager;
import com.*********.ttnet.http.HttpRequestInfo;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;

// SsInterceptor，用于增强网络请求，包括 query 去重、加密处理、打点统计等
public final class SsInterceptor extends BaseSsInterceptor {

    // 是否启用 query 加密（控制开关）
    public static volatile boolean sEncryptQueryEnabled;

    // 是否已注入 HttpRequestInfo，仅执行一次
    public static volatile boolean sHttpInfoInjectedByInterceptor;

    // 设置是否开启 query 加密功能
    public static void EnableEncryptQuery(boolean z) {
        sEncryptQueryEnabled = z;
    }

    // 去除重复 query 参数，构造新的 URL
    private String tryFilterDupQuery(String str) {
        List list;
        List list2;
        if (StringUtils.isEmpty(str)) {
            return str;
        }
        try {
            LinkedHashMap linkedHashMap = new LinkedHashMap();
            Pair<String, String> parseUrlWithValueList = UrlUtils.parseUrlWithValueList(str, linkedHashMap);
            if (parseUrlWithValueList == null) {
                return str;
            }
            // 去重 query
            if (!linkedHashMap.isEmpty()) {
                for (Map.Entry entry : linkedHashMap.entrySet()) {
                    if (entry != null && (list2 = (List) entry.getValue()) != null && !list2.isEmpty()) {
                        LinkedHashSet linkedHashSet = new LinkedHashSet();
                        Iterator it = list2.iterator();
                        while (it.hasNext()) {
                            String str2 = (String) it.next();
                            if (linkedHashSet.contains(str2)) {
                                it.remove();
                            } else {
                                linkedHashSet.add(str2);
                            }
                        }
                    }
                }
            }
            UrlBuilder urlBuilder = new UrlBuilder(O.C((String) parseUrlWithValueList.first, (String) parseUrlWithValueList.second));
            if (!linkedHashMap.isEmpty()) {
                for (Map.Entry entry2 : linkedHashMap.entrySet()) {
                    if (entry2 != null && entry2.getKey() != null && (list = (List) entry2.getValue()) != null && !list.isEmpty()) {
                        Iterator it2 = list.iterator();
                        while (it2.hasNext()) {
                            urlBuilder.addParam((String) entry2.getKey(), (String) it2.next());
                        }
                    }
                }
            }
            return urlBuilder.build();
        } catch (Throwable th) {
            // 异常上报并打印堆栈
            CatcherTrace.reportThrowable(th, "c.b.ttn.retrofit.SsInterceptor", "tryFilterDupQuery", "java/lang/Throwable", ProfileManager.VERSION);
            GlobalProxyLancet.com_ss_android_ugc_aweme_lancet_ThrowableLancet_thrPrintStackTrace(th);
            return str;
        }
    }

    // 拦截请求：对 Request 做加工处理
    @Override
    public Request intercept(Request request) {
        Request intercept = super.intercept(request); // 先调用父类逻辑
        if (intercept == null) {
            return null;
        }

        // 只注入一次
        if (!sHttpInfoInjectedByInterceptor) {
            HttpRequestInfo.injectCreate();
            sHttpInfoInjectedByInterceptor = true;
        }

        long start = SystemClock.uptimeMillis();
        Request.Builder newBuilder = intercept.newBuilder();

        // 去重 query
        String filteredUrl = tryFilterDupQuery(intercept.getUrl());
        newBuilder.url(filteredUrl);

        if (intercept.getMetrics() != null) {
            intercept.getMetrics().filterDupQueryDuration = SystemClock.uptimeMillis() - start;
        }

        long encryptStart = SystemClock.uptimeMillis();
        ArrayList<Header> headers = new ArrayList<>();
        if (intercept.getHeaders() != null) {
            headers.addAll(intercept.getHeaders());
        }

        // 是否启用 query 加密
        if (sEncryptQueryEnabled) {
            try {
                LinkedList<Pair<String, String>> encryptedHeaders = new LinkedList<>();
                String encryptedUrl = RequestEncryptUtils.tryEncryptRequest(filteredUrl, encryptedHeaders);
                if (encryptedUrl != null) {
                    newBuilder.url(encryptedUrl);
                }
                for (Pair pair : encryptedHeaders) {
                    if (pair != null) {
                        headers.add(new Header((String) pair.first, (String) pair.second));
                    }
                }
            } catch (Throwable th) {
                CatcherTrace.reportThrowable(th, "c.b.ttn.retrofit.SsInterceptor", "int", "java/lang/Throwable", ProfileManager.VERSION);
                GlobalProxyLancet.com_ss_android_ugc_aweme_lancet_ThrowableLancet_thrPrintStackTrace(th);
            }
        }
        if (intercept.getMetrics() != null) {
            intercept.getMetrics().encryptRequestDuration = SystemClock.uptimeMillis() - encryptStart;
        }

        // 添加 X-SS-REQ-TICKET 请求头
        long ticketStart = SystemClock.uptimeMillis();
        String ticket = null;
        try {
            if (C1478114ji.LIZIZ) {
                if (C16130FjH.LIZIZ(filteredUrl).getHost().endsWith(C1477714je.LIZ())) {
                    ticket = String.valueOf(System.currentTimeMillis());
                }
            }
            if (!StringUtils.isEmpty(ticket)) {
                headers.add(new Header("X-SS-REQ-TICKET", ticket));
            }
        } catch (Throwable th3) {
            CatcherTrace.reportThrowable(th3, "c.b.ttn.retrofit.SsInterceptor", "int", "java/lang/Throwable", ProfileManager.VERSION);
            GlobalProxyLancet.com_ss_android_ugc_aweme_lancet_ThrowableLancet_thrPrintStackTrace(th3);
        }
        if (intercept.getMetrics() != null) {
            intercept.getMetrics().genReqTicketDuration = SystemClock.uptimeMillis() - ticketStart;
        }

        // 添加 X-TT-VERIFY-ID 请求头
        long verifyStart = SystemClock.uptimeMillis();
        if (C1465814hj.LIZ) {
            try {
                String verifyId = C1465814hj.LIZ(C16130FjH.LIZIZ(filteredUrl));
                if (!TextUtils.isEmpty(verifyId)) {
                    headers.add(new Header("X-TT-VERIFY-ID", verifyId));
                }
            } catch (Throwable th4) {
                CatcherTrace.reportThrowable(th4, "c.b.ttn.retrofit.SsInterceptor", "preProcessingImpl", "java/lang/Throwable", ProfileManager.VERSION);
                GlobalProxyLancet.com_ss_android_ugc_aweme_lancet_ThrowableLancet_thrPrintStackTrace(th4);
            }
        }
        if (intercept.getMetrics() != null) {
            intercept.getMetrics().preCdnCacheVerifyDuration = SystemClock.uptimeMillis() - verifyStart;
        }

        // 添加 x-bd-client-key 与 x-bd-kmsv
        ClientKeyManager clientKeyManager = ClientKeyManager.LJFF();
        RetrofitMetrics metrics = intercept.getMetrics();
        if (ClientKeyManager.LJIIIIZZ != null && ClientKeyManager.LJIIIZ &&
            !TextUtils.isEmpty(clientKeyManager.LIZJ) &&
            !TextUtils.isEmpty(clientKeyManager.LJFF)) {
            long keyStart = SystemClock.uptimeMillis();
            headers.add(new Header("x-bd-client-key", clientKeyManager.LIZJ));
            headers.add(new Header("x-bd-kmsv", clientKeyManager.LJFF));
            if (metrics != null) {
                metrics.addClientKeyDuration = SystemClock.uptimeMillis() - keyStart;
                metrics.addClientKeySuccess = true;
            }
        }

        newBuilder.headers(headers);
        return newBuilder.build();
    }

    // 响应拦截方法
    @Override
    public void intercept(Request request, com.*********.retrofit2.SsResponse response) {
        ...
    }
}

```

所以，通过 hook intercept(Request, SsResponse) 方法就可以拿到请求和响应的数据。

Request
-------

Request 源码如下：

```
package com.*********.retrofit2.client;

import com.*********.common.profilesdk.ProfileManager;
import com.*********.memoryx.StringBuilderCache;
import com.*********.qss.common.catcher.CatcherTrace;
import com.*********.retrofit2.OptConfig;
import com.*********.retrofit2.RetrofitMetrics;
import com.*********.retrofit2.Utils;
import com.*********.retrofit2.mime.FormUrlEncodedTypedOutput;
import com.*********.retrofit2.mime.TypedOutput;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;
import okhttp3.RequestBody;

/* loaded from: base.apk_classes37.jar:com/*********/retrofit2/client/Request.class */
public final class Request {
    public final boolean addCommonParam;
    public final TypedOutput body;
    public final int commonParamLevel;
    public Object extraInfo;
    public final List<Header> headers;
    public boolean isBodyEncryptEnabled;
    public boolean isQueryEncryptEnabled;
    public final int maxLength;
    public final String method;
    public RetrofitMetrics metrics;
    public final int priorityLevel;
    public int queryFilterPriority;
    public final RequestBody requestBody;
    public final int requestPriorityLevel;
    public final boolean responseStreaming;
    public final String serviceType;
    public final Map<Class<?>, Object> tags;
    public URI uri;
    public final String url;

    ...
    
}

```

SsResponse
----------

Response 源码如下：

```
package com.*********.retrofit2;

import com.*********.retrofit2.client.Header;
import com.*********.retrofit2.client.Response;
import com.*********.retrofit2.mime.TypedInput;
import java.util.List;

/* loaded from: base.apk_classes28.jar:com/*********/retrofit2/SsResponse.class */
public final class SsResponse<T> {
    public final T body;
    public final TypedInput errorBody;
    public final Response rawResponse;
    public RetrofitMetrics retrofitMetrics;

    ...
}

```

smali 层深入分析
===========

** 的 java 层代码量实在太大了，用 jadx 打开内存直接爆了。

使用 dex2smali[7] 把 jar 批量转换为 smali

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXck2siah5w1hww8poYkfIz2R284dxRN13SloDCVTGbaQiaBcIDksIYn3vA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image2.png

把 smali 文件夹导入 Sublime Text，全局查找（Ctrl + Shift + F）"com/google/gson/Gson" 。

找到数据转换工具类 GsonUtils

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXc3RcWKiaghWcWa89pgN7cVk8wDPk2GiagvMdbBXADMwSZriaa8jRrxQMvA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image3.png

其中 LIZJ 方法用于把 Object 转换为 String

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXcvV81odN2mib2pbk5clfShlBR5mRxN9EgMx3LJTMmLaeiahicBGq0MicfGw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image4.png

使用 Gson 把 body 转换成 String
=========================

封装一个 toJson 方法使用 GsonUtils 把 Reqeust Body

```
public final TypedOutput body;

```

和 Response Body

```
public final T body;

```

转换成 String

代码实现如下：

```
function toJson(obj) {
    try {
        const GsonUtils = Java.use("com.*********.helios.sdk.utils.GsonUtils");

        // 直接调用 LIZJ 静态方法
        const jsonString = GsonUtils.LIZJ(obj);

        return jsonString ? jsonString.toString() : "";
    } catch (err) {
        console.error("Error calling GsonUtils.LIZJ:", err);
        return "";
    }
}

```

使用 Frida 实时截取请求响应数据
===================

打印 Request
----------

封装一个 printRequest 方法用于打印 Request：

```
function printRequestContext(context) {
    if (!context) return;

    const RequestContext = Java.use("com.*********.ttnet.http.RequestContext");
    const BaseRequestContext = Java.use("com.*********.frameworks.baselib.network.http.BaseRequestContext");

    const ctx = Java.cast(context, RequestContext);

    const sb = [];

    // 🔷 RequestContext 自身字段
    sb.push("🧩 RequestContext:");
    sb.push(`  - body_is_json: ${ctx.body_is_json.value}`);
    sb.push(`  - cdn_request_num: ${ctx.cdn_request_num.value}`);
    sb.push(`  - decode_time: ${ctx.decode_time.value}`);
    sb.push(`  - force_no_https: ${ctx.force_no_https.value}`);
    sb.push(`  - https_fail_times: ${ctx.https_fail_times.value}`);
    sb.push(`  - https_to_http: ${ctx.https_to_http.value}`);
    sb.push(`  - local_sign: ${ctx.local_sign.value}`);
    sb.push(`  - ss_sign: ${ctx.ss_sign.value}`);
    sb.push(`  - using_https: ${ctx.using_https.value}`);

    // headers 是 JSONObject，需要转换成字符串打印
    const headers = ctx.headers.value;
    if (headers !== null) {
        sb.push(`  - headers: ${headers.toString()}`);
    } else {
        sb.push(`  - headers: null`);
    }

    // 🔶 BaseRequestContext 父类字段
    const baseCtx = Java.cast(context, BaseRequestContext);
    sb.push("🔹 BaseRequestContext:");

    const tryGet = (fieldName) => {
        try {
            return baseCtx[fieldName].value;
        } catch (e) {
            return "<error>";
        }
    };

    sb.push(`  - authCredentials: ${tryGet('authCredentials')}`);
    sb.push(`  - byPassProxy: ${tryGet('byPassProxy')}`);
    sb.push(`  - bypassCookie: ${tryGet('bypassCookie')}`);
    sb.push(`  - enable_http_cache: ${tryGet('enable_http_cache')}`);
    sb.push(`  - force_handle_response: ${tryGet('force_handle_response')}`);
    sb.push(`  - force_use_okhttp: ${tryGet('force_use_okhttp')}`);
    sb.push(`  - ignoreCheckMinInputStreamBufferSize: ${tryGet('ignoreCheckMinInputStreamBufferSize')}`);
    sb.push(`  - input_stream_buffer_size: ${tryGet('input_stream_buffer_size')}`);
    sb.push(`  - isCustomizedCookie: ${tryGet('isCustomizedCookie')}`);
    sb.push(`  - okHttpRequestClientBuilderHook: ${tryGet('okHttpRequestClientBuilderHook')}`);
    sb.push(`  - output_stream_buffer_size: ${tryGet('output_stream_buffer_size')}`);
    sb.push(`  - protect_timeout: ${tryGet('protect_timeout')}`);
    sb.push(`  - read_error_response: ${tryGet('read_error_response')}`);
    sb.push(`  - remoteIp: ${tryGet('remoteIp')}`);
    sb.push(`  - request_flag: ${tryGet('request_flag')}`);
    sb.push(`  - request_type_flags: ${tryGet('request_type_flags')}`);
    sb.push(`  - rotationHostRetryInfoList: ${tryGet('rotationHostRetryInfoList')}`);
    sb.push(`  - socket_connect_timeout: ${tryGet('socket_connect_timeout')}`);
    sb.push(`  - socket_read_timeout: ${tryGet('socket_read_timeout')}`);
    sb.push(`  - socket_write_timeout: ${tryGet('socket_write_timeout')}`);
    sb.push(`  - status: ${tryGet('status')}`);
    sb.push(`  - streaming_force_return_response: ${tryGet('streaming_force_return_response')}`);
    sb.push(`  - throttle_net_speed: ${tryGet('throttle_net_speed')}`);
    sb.push(`  - timeout_connect: ${tryGet('timeout_connect')}`);
    sb.push(`  - timeout_read: ${tryGet('timeout_read')}`);
    sb.push(`  - timeout_write: ${tryGet('timeout_write')}`);
    sb.push(`  - bypass_network_status_check: ${tryGet('bypass_network_status_check')}`);
    sb.push(`  - followRedirectInternal: ${tryGet('followRedirectInternal')}`);
    sb.push(`  - is_need_monitor_in_cancel: ${tryGet('is_need_monitor_in_cancel')}`);
    sb.push(`  - extraInfo: ${tryGet('extraInfo')}`);

    return sb.join("\n");
}


function printRequest(req) {
    if (!req) return '⚠️ Request is null';

    const Request = Java.use('com.*********.retrofit2.client.Request');
    const Header = Java.use('com.*********.retrofit2.client.Header');

    const realReq = Java.cast(req, Request);
    const sb = [];

    sb.push('=== 📦 Request Dump Start ===');
    sb.push('➡️ Method: ' + realReq.getMethod());
    sb.push('🌐 URL: ' + realReq.getUrl());
    sb.push('📌 Path: ' + realReq.getPath());
    sb.push('🔐 QueryEncrypt: ' + realReq.isQueryEncryptEnabled());
    sb.push('🔐 BodyEncrypt: ' + realReq.isBodyEncryptEnabled());
    sb.push('📊 PriorityLevel: ' + realReq.getPriorityLevel());
    sb.push('⚙️ ServiceType: ' + realReq.getServiceType());

    // 🧾 Headers
    const headers = realReq.getHeaders();
    if (headers && headers.size() > 0) {
        sb.push('🧾 Headers:');
        for (let i = 0; i < headers.size(); i++) {
            const h = Java.cast(headers.get(i), Header);
            sb.push(`  - ${h.getName()}: ${h.getValue()}`);
        }
    }

    // 📦 Body info
    const body = realReq.getBody();
    if (body) {
        sb.push('📦 Body mimeType: ' + body.mimeType());
        sb.push('📦 Body length: ' + body.length());
        sb.push('📦 Body.toString(): ' + body.toString());
        sb.push('📦 Body: ' + toJson(body));
    }

    // 🏷️ Tag
    const tag = realReq.tag();
    if (tag) {
        sb.push('🏷️ Tag: ' + tag.toString());
    }

    // 📦 ExtraInfo 判断类型
    const extraInfo = realReq.getExtraInfo();
    if (extraInfo) {
        const className = extraInfo.getClass().getName();
        sb.push('📦 ExtraInfo class: ' + className);
        if (extraInfo.getClass().getName().startsWith("com.*********.ttnet.http.RequestContext")) {
            sb.push(printRequestContext(extraInfo));
        } else {
            sb.push('📦 ExtraInfo.toString(): ' + extraInfo.toString());
        }
    }

    sb.push('=== ✅ Request Dump End ===');
    return sb.join('\n');
}

```

打印 SsResponse
-------------

封装一个 printSsResponse 方法用于打印 SsResponse：

```
function printSsResponse(resp) {
    if (!resp) return '⚠️ SsResponse is null';

    const SsResponse = Java.use('com.*********.retrofit2.SsResponse');
    // const Response = Java.use('com.*********.retrofit2.client.Response');
    const Header = Java.use('com.*********.retrofit2.client.Header');
    // const TypedInput = Java.use('com.*********.retrofit2.mime.TypedInput');

    const ssResp = Java.cast(resp, SsResponse);
    const rawResp = ssResp.raw(); // com.*********.retrofit2.client.Response

    const sb = [];
    sb.push('=== 📥 SsResponse Dump Start ===');

    sb.push('✅ isSuccessful: ' + ssResp.isSuccessful());
    sb.push('🔢 Code: ' + ssResp.code());
    sb.push('📝 Message: ' + ssResp.message());

    // Headers
    const headers = ssResp.headers();
    if (headers && headers.size() > 0) {
        sb.push('🧾 Headers:');
        for (let i = 0; i < headers.size(); i++) {
            const h = Java.cast(headers.get(i), Header);
            sb.push(`  - ${h.getName()}: ${h.getValue()}`);
        }
    }

    // Body
    const body = ssResp.body();
    if (body) {
        sb.push('📦 Body.toString(): ' + body.toString());
        sb.push('📦 Body: ' + toJson(body));
    }

    // Error body
    const errorBody = ssResp.errorBody();
    if (errorBody) {
        sb.push('❗ ErrorBody mimeType: ' + errorBody.mimeType());
        sb.push('❗ ErrorBody length: ' + errorBody.length());
    }

    // RetrofitMetrics
    const metrics = ssResp.getRetrofitMetrics();
    if (metrics) {
        sb.push('📈 RetrofitMetrics: ' + metrics.toString());
    }

    sb.push('=== 📥 SsResponse Dump End ===');
    // console.log(sb.join('\n'));
    return sb.join('\n');
}

```

Hook intercept(Request, SsResponse)
-----------------------------------

使用 Frida Hook intercept(Request, SsResponse) 截取请求和响应数据。

ttnet.js

```
function hookSsInterceptor() {
    ***************************************************************************
    // Hook目标类
    *********************************************************
    ********************************************************************************************************************
        *****************************************

        **************
            **************************************
            *******************
            ***********************
            ****************************
        **************

        // 直接打印日志
        // console.log(log);

        // ✅ 将日志发送给 Python 端而不是 console.log
        *******
            *********************
            *************
        ****

        **********************************
    ***
}


setImmediate(() => {
    Java.perform(function () {
        hookSsInterceptor()
    });
})


// frida -H 127.0.0.1:1234 -F -l ttnet.js
// frida -H 127.0.0.1:1234 -F -l ttnet.js -o log.txt

```

在 Python 端接收到数据并输出到 log.txt

ttnet.py

```
import time

import frida

LOG_FILE = "log.txt"

def on_message(message, data):
    if message["type"] == "send":
        tag = message["payload"].get("tag")
        log = message["payload"].get("payload")
        if tag == "ss_intercept":
            with open(LOG_FILE, "w", encoding="utf-8") as f:
                f.write(f"{log}\n\n")
    elif message["type"] == "error":
        print("❌ Frida script error:", message["stack"])

def main():
    # USB链接
    # device: frida.core.Device = frida.get_usb_device()

    # 远程链接
    device = frida.get_device_manager().add_remote_device("127.0.0.1:1234")

    # 附加到当前前台应用
    pid = device.get_frontmost_application().pid
    session: frida.core.Session = device.attach(pid)

    # 加载脚本
    with open("ttnet.js", "r", encoding="utf-8") as f:
        script = session.create_script(f.read())

    script.on("message", on_message)
    script.load()

    print(f"✅ Script loaded and logging to {LOG_FILE}")
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\n👋 Exit by user.")

if __name__ == "__main__":
    main()

```

测试
--

截取下来的数据如下，Reqeust Body 是经过编码加密的二进制数据。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXc74icWzy87PQwia23CdnynTpqzSwJn99yI1BS4Hrsp18r0tcDWc183Weg/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image5.png

加密字段的解码与数据还原
============

数据格式说明
------

```
{
    "content": {
        "buf": [40, -75,  ...],
        "count": 3509
    },
    "enableRecordFields": false,
    "fields": {},
    "mBodyMd5Stub": "27237ACA0C**********BEBD640337",
    "mIsBodyEncrypted": false,
    "mOriginBodySize": 13440,
    "mType": "zstd"
}

```

content.buf 是一个 byte 数组，包含字节值（范围 -128 到 127），这是 Java 的 byte[] 通过 Gson 序列化后的表现形式。我们可以通过 Python 将它转换为字节数组后再解码成字符串。

该数据结构的实际类型是 FormUrlEncodedTypedOutput，Java 源码如下：

```
package com.*********.retrofit2.mime;

import com.*********.common.profilesdk.ProfileManager;
import com.*********.frameworks.encryptor.EncryptorUtil;
import com.*********.qss.common.catcher.CatcherTrace;
import com.*********.retrofit2.mime.TTRequestCompressManager;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import leakcanary.internal.LeakCanaryFileProvider;

/* loaded from: base.apk_classes37.jar:com/*********/retrofit2/mime/FormUrlEncodedTypedOutput.class */
public final class FormUrlEncodedTypedOutput extends AbsTypedOutput {
    public ByteArrayOutputStream content;
    public final boolean enableRecordFields;
    public final Map<String, List<String>> fields;

    public FormUrlEncodedTypedOutput() {
        this.content = new ByteArrayOutputStream();
        this.fields = new HashMap();
        this.enableRecordFields = false;
    }

    public FormUrlEncodedTypedOutput(boolean z) {
        this.content = new ByteArrayOutputStream();
        this.fields = new HashMap();
        this.enableRecordFields = z;
    }

    public void addField(String str, String str2) {
        addField(str, true, str2, true);
    }

    public void addField(String str, boolean z, String str2, boolean z2) {
        if (str == null) {
            throw new NullPointerException(LeakCanaryFileProvider.ATTR_NAME);
        }
        if (str2 == null) {
            throw new NullPointerException("value");
        }
        if (this.content.size() > 0) {
            this.content.write(38);
        }
        if (this.enableRecordFields) {
            if (this.fields.containsKey(str)) {
                this.fields.get(str).add(str2);
            } else {
                ArrayList arrayList = new ArrayList();
                arrayList.add(str2);
                this.fields.put(str, arrayList);
            }
        }
        String str3 = str;
        if (z) {
            try {
                str3 = URLEncoder.encode(str, "UTF-8");
            } catch (IOException e) {
                CatcherTrace.reportThrowable(e, "c.b.ret.mim.FormUrlEncodedTypedOutput", "afie", "java/io/IOException", ProfileManager.VERSION);
                throw new RuntimeException(e);
            }
        }
        String str4 = str2;
        if (z2) {
            str4 = URLEncoder.encode(str2, "UTF-8");
        }
        this.content.write(str3.getBytes("UTF-8"));
        this.content.write(61);
        this.content.write(str4.getBytes("UTF-8"));
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public TTRequestCompressManager.CompressData compressRequestBody(String str, String str2, boolean z) {
        byte[] byteArray = this.content.toByteArray();
        if (byteArray == null) {
            return null;
        }
        TTRequestCompressManager.CompressData compressBody = TTRequestCompressManager.compressBody(byteArray, byteArray.length, str, str2, z);
        if (compressBody != null && compressBody.data != null) {
            this.mOriginBodySize = byteArray.length;
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream(compressBody.data.length);
            this.content = byteArrayOutputStream;
            byte[] bArr = compressBody.data;
            byteArrayOutputStream.write(bArr, 0, bArr.length);
            this.mType = compressBody.contentEncoding;
        }
        return compressBody;
    }

    public Map<String, List<String>> fields() {
        return this.fields;
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public String fileName() {
        return null;
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public byte[] getOriginBody() {
        return TTRequestCompressManager.decompressDataByType(this.content.toByteArray(), this.mType, this.mOriginBodySize);
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public boolean interceptRequestBody() {
        byte[] LIZ;
        byte[] byteArray = this.content.toByteArray();
        if (byteArray == null || byteArray.length > 102400 || (LIZ = EncryptorUtil.LIZ(byteArray, byteArray.length)) == null) {
            return false;
        }
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream(LIZ.length);
        this.content = byteArrayOutputStream;
        byteArrayOutputStream.write(LIZ, 0, LIZ.length);
        this.mIsBodyEncrypted = true;
        return true;
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public long length() {
        return this.content.size();
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public String md5Stub() {
        this.mBodyMd5Stub = DigestUtil.md5Hex(this.content.toByteArray());
        return this.mBodyMd5Stub;
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public String mimeType() {
        return "application/x-www-form-urlencoded; charset=UTF-8";
    }

    @Override // com.*********.retrofit2.mime.AbsTypedOutput
    public void writeTo(OutputStream outputStream) {
        outputStream.write(this.content.toByteArray());
    }
}

```

buf 解码
------

这是 Retrofit 发送的请求体中的内容，那么 buf 实际是压缩后的二进制数据（注意 "mType": "zstd" 表示使用了 zstd 压缩），我们需要：

1.  1. 把 buf 还原为 Uint8Array 或 Buffer。
    
2.  2. 使用 zstd 解压。
    
3.  3. 然后解码为字符串（如 UTF-8）。
    
4.  4. 使用 URL 解码
    

zstd（Zstandard）[8] 是由 Facebook 开发的一种 无损压缩算法，设计目标是提供：

*   • 比 zlib 更好的压缩比
    
*   • 更快的压缩和解压速度
    

代码实现如下：

```
import zstandard
from urllib.parse import unquote


def decode_zstd_buf_to_text(buf):
    """
    将 Retrofit 中 FormUrlEncodedTypedOutput 的 content.buf 解码为文本字符串

    参数:
        buf (List[int]): 表示 zstd 压缩后的字节数组（有符号 int）

    返回:
        str: 解压缩后的文本内容
    """
    # 将 buf 转换为 bytes 类型（有符号 int8 转换为无符号）
    *******************************************************

    # 使用 zstandard 解压
    ************************************
    *************************************************

    # 解码为字符串（假设 utf-8 编码）
    ************************************

    # URL 解码
    ********************************
    
    ***********************

```

buf 字段还原并重写
-----------

下面是一个完整的 Python 函数，可以：

1.  1. 读取 txt 文件内容
    
2.  2. 识别包含 buf: [数组] 的字段
    
3.  3. 解码 buf 中的 byte 数组并转换为字符串
    
4.  4. 替换原始 buf 字段为可读字符串
    
5.  5. 覆盖写回原始 txt 文件
    

完整代码实现如下：

```
import re

import zstandard


def decode_zstd_buf_to_text(buf):
    """
    将 Retrofit 中 FormUrlEncodedTypedOutput 的 content.buf 解码为文本字符串

    参数:
        buf (List[int]): 表示 zstd 压缩后的字节数组（有符号 int）

    返回:
        str: 解压缩后的文本内容
    """
    # 将 buf 转换为 bytes 类型（有符号 int8 转换为无符号）
    *******************************************************

    # 使用 zstandard 解压
    ************************************
    *************************************************

    # 解码为字符串（假设 utf-8 编码）
    ************************************


def decode_buf_array(match):
    **************************
    # 提取所有整数（支持负数）
    *************************************************************************
    # 解码
    ***********************************************
    *****************************


def replace_buf_array_with_string(file_path: str, encoding='utf-8'):
    """
    自动将文件中类似 "buf":[40, -75, 47, ...] 的内容转换为字符串并替换原 buf 值。

    :param file_path: txt 文件路径
    :param encoding: 读取与写入时的编码（默认 utf-8）
    """
    ***************************************************
        *******************

    # 正则匹配 buf: [xx, xx, ...] 的结构，支持跨行
    ***********************************

    # 替换所有匹配的 buf 字段
    **************************************************************************

    # 写回原文件
    ***************************************************
        *********************

    *****************************


if __name__ == '__main__':
    replace_buf_array_with_string('log1.txt')

```

成功还原 buf 字段数据。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jl8N0d0k9apicF6mWHCIqmXclKx8ZsAgblguLj6UwUVyq8Tz3GBPAQ9MPjtmibmIYHKYCBWsxia44OYw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image6.png

#### 引用链接

`[1]` 如何实现 Android App 的抓包防护？又该如何绕过？一文看懂攻防博弈:_https://cyrus-studio.github.io/blog/posts/%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0-android-app-%E7%9A%84%E6%8A%93%E5%8C%85%E9%98%B2%E6%8A%A4%E5%8F%88%E8%AF%A5%E5%A6%82%E4%BD%95%E7%BB%95%E8%BF%87%E4%B8%80%E6%96%87%E7%9C%8B%E6%87%82%E6%94%BB%E9%98%B2%E5%8D%9A%E5%BC%88/_  
`[2]`安卓抓包实战：使用 Charles 抓取 App 数据全流程详解:_https://cyrus-studio.github.io/blog/posts/%E5%AE%89%E5%8D%93%E6%8A%93%E5%8C%85%E5%AE%9E%E6%88%98%E4%BD%BF%E7%94%A8-charles-%E6%8A%93%E5%8F%96-app-%E6%95%B0%E6%8D%AE%E5%85%A8%E6%B5%81%E7%A8%8B%E8%AF%A6%E8%A7%A3/_  
`[3]`frida_dex_dump:_https://github.com/CYRUS-STUDIO/frida_dex_dump_  
`[4]`dex2jar:_https://github.com/CYRUS-STUDIO/dex2jar_  
`[5]`find_in_jars.py:_https://github.com/CYRUS-STUDIO/dex2jar/blob/main/find_in_jars.py_  
`[6]`打造自己的 Jar 文件分析工具：类名匹配 + 二进制搜索 + 日志输出全搞定:_https://cyrus-studio.github.io/blog/posts/%E6%89%93%E9%80%A0%E8%87%AA%E5%B7%B1%E7%9A%84-jar-%E6%96%87%E4%BB%B6%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%90%8D%E5%8C%B9%E9%85%8D-+-%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%90%9C%E7%B4%A2-+-%E6%97%A5%E5%BF%97%E8%BE%93%E5%87%BA%E5%85%A8%E6%90%9E%E5%AE%9A/_  
`[7]`dex2smali:_https://github.com/CYRUS-STUDIO/dex2smali_  
`[8]`zstd（Zstandard）:_https://github.com/facebook/zstd_