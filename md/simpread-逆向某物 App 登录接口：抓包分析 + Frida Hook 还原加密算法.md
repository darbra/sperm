> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/E7TJX0gZaaP0yKL9cLzSmQ?poc_token=HFuMY2ij7qgQuXb7MUvlnR85yncstf4qLuIC9eKc)

> 免责声明：文章中所有内容仅供学习交流使用，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【CYRUS STUDIO】联系作者

登录接口抓包分析
========

使用 Charles 分析登录接口，打开 APP，输入用户名和密码，点击登录后，可以看到会调用一个 unionLogin 接口

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmGaDf4a7k0ns5u9Kv9Q02FjHkp6npLmaqicjdON7OdlDfvEZkJvk3WzQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image1.png

  
Charles 的详细使用教程参考：安卓抓包实战：使用 Charles 抓取 App 数据全流程详解 [1]

response：

```
{
    "code": 704,
    "data": "3YlD20Mglo7XabT4cSg0p_sFlOH7UrRp2-SV2xAVsXYfLssqWSX1NpUQ9HhlWEnRgXF0phtI-2BfqI9Cl1cGmih8veCmWZU-ItqKSv_gs3stFlw_*****************************************fRv25yucw==",
    "status": 704
}

```

curl 转 request（https://spidertools.cn/#/curl2Request）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmIe9VzicoyAylIIc3YRic1qaZcXI3KmIjh9wKM6PeglKxrqgQ8q8kevag/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image2.png

通过 Python 请求接口

```
import requests
import json


headers = {
    "ipvx": "**************",
    "webua": "Mozilla/5.0 (Linux; Android 7.1.2; MI8 Build/N2G47J; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.158 Mobile Safari/537.36/duapp/5.43.0(android;7.1.2)",
    "app_build": "5.43.0.10",
    "cookieToken": "",
    "ltk": "*****************************************",
    "duplatform": "android",
    "appId": "duapp",
    "duchannel": "xiaomi",
    "humeChannel": "",
    "duv": "5.43.0",
    "duloginToken": "",
    "dudeviceTrait": "MI8",
    "dudeviceBrand": "Xiaomi",
    "timestamp": "1717085035150",
    "shumeiid": "**************************************************************",
    "User-Agent": "duapp/5.43.0(android;7.1.2)",
    "X-Auth-Token": "Bearer eyJhbGciOiJSUzI1NiJ9.eyJpYXQiOjE3MTcwODQ1NzEsImV4cCI6MTc0ODYyMDU3MSwiaXNzIjoiNGY4M**********************************************************************************************JZCI6MjI2NDc5NzQzOCwidXNlck5hbWUiOiLlvpfniallci1XMlI5RzJLNiIsImlzR3Vlc3QiOnRydWV9.mJk2LvNvA69dDq6_eTVwa4Lex0vkDg7RXtR6uAzb9k2z6RGoAJ4Pjy__555ekGRbKwleQtlaHeiRUM1GaS-NnteXAXyz2NBw9tDI7Q08eCLSHiGp5-mC26YFUdml7gHPNp-zXlas4BOsVm3yqJrrjCslU7Uj2U6QIy2th2lEE81uBy1M4mw1OO7anwkIXiJJOHak0KaiPSnyr36Nq0uklqmbRMQLyAtbBKFkooRQ1EGDQ5x_egpeHpN4QZXQn81iHWaqx1JxmZ6U0yeQbn6T_0IW2U7_9lLBlnUQ5d3dL0NMp8-nXdID_Tjy3HiWpwdydwcQkpPBv0NQHYzfOfP0qQ",
    "isRoot": "0",
    "emu": "0",
    "isProxy": "1",
    "SK": "********************************************************************************************",
    "edk": "**************************************************************************",
    "dps": "1",
    "sks": "0,adw2",
    "skc": "*********************",
    "rtk": "****************************",
    "Content-Type": "application/json; charset=utf-8",
    "Host": "************"
}
cookies = {
    "HWWAFSESTIME": "*************"
}
url = "https://************/api/v1/app/user_core/users/unionLogin"
data = {
    "cipherParam": "userName",
    "countryCode": 86,
    "loginToken": "",
    "newSign": "7717ddcec27591d894ef62740c195046",
    "password": "707413eb7907088c8765038d0727ea62",
    "platform": "android",
    "timestamp": "1717085035150",
    "type": "pwd",
    "userName": "36387881aef6d40c65c4825ec0d35868_1",
    "v": "5.43.0"
}
data = json.dumps(data, separators=(',', ':'))
response = requests.post(url, headers=headers, cookies=cookies, data=data)

print(response.text)
print(response)

```

需要分析的大概是下面 3 个参数，其他基本都是固定的参数或者时间戳

*   • "newSign": "7717ddcec27591d894ef62740c195046",
    
*   • "password": "707413eb7907088c8765038d0727ea62",
    
*   • "userName": "36387881aef6d40c65c4825ec0d35868_1",
    

newSign 和 password 看起来很像 md5 算法后的值，都是固定长度 32 位 hex 字符串；userName 是 32 位 hex + _1

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmtD42MyMsiapOJoMOqq9cXLTsTrkZm9CQTxaE3RVXur5EbQNtxOfQJOw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image3.png

解决 Frida 反调试
============

尝试通过 Frida 去 Hook App 后报错，程序运行被终止。

关于 Frida 的详细使用可以参考这篇文章：一文搞懂如何使用 Frida Hook Android App[2]

1. 绕过文件名和端口检测
-------------

从报错信息可以看出 Frida 被一个 C 程序终止了。

```
HashMap.put() called with key: eventName value: networkDetection
Fatal Python error: none_dealloc: deallocating None: bug likely caused by a refcount error in a C extension
Python runtime state: initialized

Current thread 0x00010c04 (most recent call first):
  <no Python frame>

Thread 0x00003d40 (most recent call first):
  File "D:\App\Miniconda\envs\spider\Lib\site-packages\prompt_toolkit\eventloop\win32.py", line 50 in wait_for_handles
  File "D:\App\Miniconda\envs\spider\Lib\site-packages\prompt_toolkit\input\win32.py", line 614 in wait
  File "D:\App\Miniconda\envs\spider\Lib\concurrent\futures\thread.py", line 58 in run
  File "D:\App\Miniconda\envs\spider\Lib\concurrent\futures\thread.py", line 83 in _worker
  File "D:\App\Miniconda\envs\spider\Lib\threading.py", line 982 in run
  File "D:\App\Miniconda\envs\spider\Lib\threading.py", line 1045 in _bootstrap_inner
  File "D:\App\Miniconda\envs\spider\Lib\threading.py", line 1002 in _bootstrap

```

先尝试修改 frida-server 文件名和端口，绕过常规的文件名和端口检测，发现可以正常 Hook 了。

2. 绕过 libmsaoaidsec.so 反调试
--------------------------

但 app 启动不久后 frida 还是被杀掉了。

通过 hook dlopen 函数打印加载的 so 文件，看看是哪里做的反调试

```
function get_self_process_name() {
    var openPtr = Module.getExportByName('libc.so', 'open');
    var open = new NativeFunction(openPtr, 'int', ['pointer', 'int']);

    var readPtr = Module.getExportByName("libc.so", "read");
    var read = new NativeFunction(readPtr, "int", ["int", "pointer", "int"]);

    var closePtr = Module.getExportByName('libc.so', 'close');
    var close = new NativeFunction(closePtr, 'int', ['int']);

    var path = Memory.allocUtf8String("/proc/self/cmdline");
    var fd = open(path, 0);
    if (fd != -1) {
        var buffer = Memory.alloc(0x1000);

        var result = read(fd, buffer, 0x1000);
        close(fd);
        result = ptr(buffer).readCString();
        return result;
    }

    return "-1";
}

function hook_dlopen() {

    // Hook dlopen 函数
    const dlopenAddr = Module.findExportByName(null, "dlopen");
    const dlopen = new NativeFunction(dlopenAddr, 'pointer', ['pointer', 'int']);
    Interceptor.attach(dlopen, {
        onEnter(args) {
            // 在 Android 中 进程名通常与包名一致
            let processName = get_self_process_name()
            // 获取传递给 dlopen 的参数（SO 文件路径）
            const soPath = Memory.readUtf8String(args[0]);
            // 如果是目标 app SO 文件，则打印路径
            if (processName != "-1") {
                if (soPath.includes(processName)) {
                    // 打印信息
                    console.log("dlopen() - Loaded SO file:", soPath);
                }
            }else{
                console.log("dlopen() - Loaded SO file:", soPath);
            }
        }, onLeave(retval) {
        }
    });

    // Hook android_dlopen_ext 函数
    const android_dlopen_extAddr = Module.findExportByName(null, "android_dlopen_ext");
    const android_dlopen_ext = new NativeFunction(android_dlopen_extAddr, 'pointer', ['pointer', 'int', 'pointer']);
    Interceptor.attach(android_dlopen_ext, {
        onEnter(args) {
            // 在 Android 中 进程名通常与包名一致
            let processName = get_self_process_name()
            // 获取传递给 dlopen 的参数（SO 文件路径）
            const soPath = Memory.readUtf8String(args[0]);
            // 如果是目标 app SO 文件，则打印路径
            if (processName != "-1") {
                if (soPath.includes(processName)) {
                    // 打印信息
                    console.log("android_dlopen_ext() - Loaded SO file:", soPath);
                }
            }else{
                console.log("android_dlopen_ext() - Loaded SO file:", soPath);
            }
        }, onLeave(retval) {
        }
    });
}


setImmediate(function () {
    hook_dlopen()
});

// frida -H 127.0.0.1:1234 -l dlopen.js -f com.shizhuang.duapp

```

发现当加载到 libmsaoaidsec.so 时，frida 就停止了，日志信息如下

```
Spawned `com.shizhuang.duapp`. Resuming main thread!                
[Remote::com.shizhuang.duapp ]-> android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/oat/arm64/base.odex
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libmmkv.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libxcrash.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libdulog.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libduhook.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libszstone.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libdewuhelper.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libdusanwa.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libc++_shared.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libdu_mediacache.so
android_dlopen_ext() - Loaded SO file: /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libmsaoaidsec.so
Process terminated

```

直接删除 libmsaoaidsec.so 试试看

```
adb shell rm /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libmsaoaidsec.so

```

重新启动 frida，能正常运行了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYm2l97gI805xl026sOHApGHiaPX2J0EuYibKEaE6CgmjcDF1TibYbibmb3DA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image4.png

使用 Frida 分析参数
=============

1. Hook HashMap
---------------

因为在 Java 中参数大多数放在 HashMap 里面，所以可以 Hook 一下 HashMap 看看里面有没有这些参数。

编写 Frida Hook 脚本

```
/**
 * Hook java.util.HashMap.put 方法，打印满足条件的 key 和 value。
 *
 * @param {string[]} filters - 字符串数组，用于过滤 key/value。为空时表示不过滤，打印所有。
 */
function hookHashMap(filters) {
    Java.perform(function () {
        // 获取 java.util.HashMap 类引用
        var HashMap = Java.use('java.util.HashMap');

        // Hook put(Object key, Object value) 方法
        HashMap.put.implementation = function (key, value) {
            // 将 key 和 value 转为字符串，避免 null 报错
            var keyStr = key ? key.toString() : "null";
            var valueStr = value ? value.toString() : "null";

            // 默认是否打印为 false，filters 为空时直接设为 true（打印所有）
            var shouldPrint = filters.length === 0;

            // 遍历 filters，检查 key 或 value 是否包含任一关键字
            if (!shouldPrint) {
                for (var i = 0; i < filters.length; i++) {
                    var keyword = filters[i];
                    if (keyStr.indexOf(keyword) !== -1 || valueStr.indexOf(keyword) !== -1) {
                        shouldPrint = true;
                        break;  // 只要匹配一个即可
                    }
                }
            }

            // 打印日志
            if (shouldPrint) {
                console.log("HashMap.put() called:");
                console.log("  [key]   =", keyStr);
                console.log("  [value] =", valueStr);
            }

            // 调用原始 put 方法，保持功能不变
            return this.put(key, value);
        };
    });
}


setImmediate(function () {
    // 打印所有 put 操作
    // hookHashMap([]);

    // 只打印 key 或 value 包含 "cipherParam" 或 "userName" 的 put 操作
    hookHashMap(["cipherParam", "userName"]);
});

// 延迟hook
// setTimeout(function () {
//     hookHashMap([]);
// }, 10000);  // 10000 毫秒 = 10 秒

// frida -H 127.0.0.1:1234 -F -l hook_hashmap.js -o log.txt

```

执行脚本并输出日志到文件

```
frida -H 127.0.0.1:1234 -F -l hook_hashmap.js -o log.txt

```

输出如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmTP7oA8aibm4QUQDGtJ9g6KHatyVhZsYLKDfLpkPFrye5vmI9eM0eKxQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image5.png

得到请求参数

```
{"cipherParam":"userName","countryCode":86,"loginToken":"","newSign":"8210cfbcbac039dcd7ebf7e2107ab4f5","password":"61f209b7895b603769bf036ad80b3a00","platform":"android","timestamp":"1750247282535","type":"pwd","userName":"5f67625e05dd530fd1e026d138c2eb14_1","v":"5.43.0"}

```

2. Hook Java 层加密算法
------------------

hook 一下 java 层加密类并打印使用的加密算法、参数、调用堆栈信息。

```
/**
 * 打印调用堆栈
 */
function showStacks() {
    console.log(
        Java.use("android.util.Log")
            .getStackTraceString(
                Java.use("java.lang.Throwable").$new()
            )
    );
}


/**
 * hook java层常见加密算法并打印参数和调用堆栈
 */
function hookCrypto() {

    var ByteString = Java.use("com.android.okhttp.okio.ByteString");

    function toBase64(tag, data) {
        console.log(tag + " Base64: " + ByteString.of(data).base64());
    }

    function toHex(tag, data) {
        console.log(tag + " Hex: " + ByteString.of(data).hex());
    }

    function toUtf8(tag, data) {
        console.log(tag + " Utf8: " + ByteString.of(data).utf8());
    }

    var messageDigest = Java.use("java.security.MessageDigest");
    messageDigest.update.overload('byte').implementation = function (data) {
        console.log("MessageDigest.update('byte') is called!");
        showStacks();
        return this.update(data);
    }
    messageDigest.update.overload('java.nio.ByteBuffer').implementation = function (data) {
        console.log("MessageDigest.update('java.nio.ByteBuffer') is called!");
        showStacks();
        return this.update(data);
    }
    messageDigest.update.overload('[B').implementation = function (data) {
        console.log("MessageDigest.update('[B') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " update data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        console.log("=======================================================");
        return this.update(data);
    }
    messageDigest.update.overload('[B', 'int', 'int').implementation = function (data, start, length) {
        console.log("MessageDigest.update('[B', 'int', 'int') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " update data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        console.log("=======================================================", start, length);
        return this.update(data, start, length);
    }
    messageDigest.digest.overload().implementation = function () {
        console.log("MessageDigest.digest() is called!");
        showStacks();
        var result = this.digest();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " digest result";
        toHex(tag, result);
        toBase64(tag, result);
        console.log("=======================================================");
        return result;
    }
    messageDigest.digest.overload('[B').implementation = function (data) {
        console.log("MessageDigest.digest('[B') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " digest data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        var result = this.digest(data);
        var tags = algorithm + " digest result";
        toHex(tags, result);
        toBase64(tags, result);
        console.log("=======================================================");
        return result;
    }
    messageDigest.digest.overload('[B', 'int', 'int').implementation = function (data, start, length) {
        console.log("MessageDigest.digest('[B', 'int', 'int') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " digest data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        var result = this.digest(data, start, length);
        var tags = algorithm + " digest result";
        toHex(tags, result);
        toBase64(tags, result);
        console.log("=======================================================", start, length);
        return result;
    }

    var mac = Java.use("javax.crypto.Mac");
    mac.init.overload('java.security.Key', 'java.security.spec.AlgorithmParameterSpec').implementation = function (key, AlgorithmParameterSpec) {
        console.log("Mac.init('java.security.Key', 'java.security.spec.AlgorithmParameterSpec') is called!");
        return this.init(key, AlgorithmParameterSpec);
    }
    mac.init.overload('java.security.Key').implementation = function (key) {
        console.log("Mac.init('java.security.Key') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " init Key";
        var keyBytes = key.getEncoded();
        toUtf8(tag, keyBytes);
        toHex(tag, keyBytes);
        toBase64(tag, keyBytes);
        console.log("=======================================================");
        return this.init(key);
    }
    mac.update.overload('byte').implementation = function (data) {
        console.log("Mac.update('byte') is called!");
        return this.update(data);
    }
    mac.update.overload('java.nio.ByteBuffer').implementation = function (data) {
        console.log("Mac.update('java.nio.ByteBuffer') is called!");
        return this.update(data);
    }
    mac.update.overload('[B').implementation = function (data) {
        console.log("Mac.update('[B') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " update data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        console.log("=======================================================");
        return this.update(data);
    }
    mac.update.overload('[B', 'int', 'int').implementation = function (data, start, length) {
        console.log("Mac.update('[B', 'int', 'int') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " update data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        console.log("=======================================================", start, length);
        return this.update(data, start, length);
    }
    mac.doFinal.overload().implementation = function () {
        console.log("Mac.doFinal() is called!");
        var result = this.doFinal();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " doFinal result";
        toHex(tag, result);
        toBase64(tag, result);
        console.log("=======================================================");
        return result;
    }

    var cipher = Java.use("javax.crypto.Cipher");
    cipher.init.overload('int', 'java.security.cert.Certificate').implementation = function () {
        console.log("Cipher.init('int', 'java.security.cert.Certificate') is called!");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.Key', 'java.security.SecureRandom').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key', 'java.security.SecureRandom') is called!");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.cert.Certificate', 'java.security.SecureRandom').implementation = function () {
        console.log("Cipher.init('int', 'java.security.cert.Certificate', 'java.security.SecureRandom') is called!");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.Key', 'java.security.AlgorithmParameters', 'java.security.SecureRandom').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key', 'java.security.AlgorithmParameters', 'java.security.SecureRandom') is called!");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.Key', 'java.security.spec.AlgorithmParameterSpec', 'java.security.SecureRandom').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key', 'java.security.spec.AlgorithmParameterSpec', 'java.security.SecureRandom') is called!");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.Key', 'java.security.AlgorithmParameters').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key', 'java.security.AlgorithmParameters') is called!");
        return this.init.apply(this, arguments);
    }

    cipher.init.overload('int', 'java.security.Key').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " init Key";
        var className = JSON.stringify(arguments[1]);
        if (className.indexOf("OpenSSLRSAPrivateKey") === -1) {
            var keyBytes = arguments[1].getEncoded();
            toUtf8(tag, keyBytes);
            toHex(tag, keyBytes);
            toBase64(tag, keyBytes);
        }
        console.log("=======================================================");
        return this.init.apply(this, arguments);
    }
    cipher.init.overload('int', 'java.security.Key', 'java.security.spec.AlgorithmParameterSpec').implementation = function () {
        console.log("Cipher.init('int', 'java.security.Key', 'java.security.spec.AlgorithmParameterSpec') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " init Key";
        var keyBytes = arguments[1].getEncoded();
        toUtf8(tag, keyBytes);
        toHex(tag, keyBytes);
        toBase64(tag, keyBytes);
        var tags = algorithm + " init iv";
        var iv = Java.cast(arguments[2], Java.use("javax.crypto.spec.IvParameterSpec"));
        var ivBytes = iv.getIV();
        toUtf8(tags, ivBytes);
        toHex(tags, ivBytes);
        toBase64(tags, ivBytes);
        console.log("=======================================================");
        return this.init.apply(this, arguments);
    }

    cipher.doFinal.overload('java.nio.ByteBuffer', 'java.nio.ByteBuffer').implementation = function () {
        console.log("Cipher.doFinal('java.nio.ByteBuffer', 'java.nio.ByteBuffer') is called!");
        showStacks();
        return this.doFinal.apply(this, arguments);
    }
    cipher.doFinal.overload('[B', 'int').implementation = function () {
        console.log("Cipher.doFinal('[B', 'int') is called!");
        showStacks();
        return this.doFinal.apply(this, arguments);
    }
    cipher.doFinal.overload('[B', 'int', 'int', '[B').implementation = function () {
        console.log("Cipher.doFinal('[B', 'int', 'int', '[B') is called!");
        showStacks();
        return this.doFinal.apply(this, arguments);
    }
    cipher.doFinal.overload('[B', 'int', 'int', '[B', 'int').implementation = function () {
        console.log("Cipher.doFinal('[B', 'int', 'int', '[B', 'int') is called!");
        showStacks();
        return this.doFinal.apply(this, arguments);
    }
    cipher.doFinal.overload().implementation = function () {
        console.log("Cipher.doFinal() is called!");
        showStacks();
        return this.doFinal.apply(this, arguments);
    }

    cipher.doFinal.overload('[B').implementation = function () {
        console.log("Cipher.doFinal('[B') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " doFinal data";
        var data = arguments[0];
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        var result = this.doFinal.apply(this, arguments);
        var tags = algorithm + " doFinal result";
        toHex(tags, result);
        toBase64(tags, result);
        console.log("=======================================================");
        return result;
    }
    cipher.doFinal.overload('[B', 'int', 'int').implementation = function () {
        console.log("Cipher.doFinal('[B', 'int', 'int') is called!");
        showStacks();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " doFinal data";
        var data = arguments[0];
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        var result = this.doFinal.apply(this, arguments);
        var tags = algorithm + " doFinal result";
        toHex(tags, result);
        toBase64(tags, result);
        console.log("=======================================================", arguments[1], arguments[2]);
        return result;
    }

    var signature = Java.use("java.security.Signature");
    signature.update.overload('byte').implementation = function (data) {
        console.log("Signature.update('byte') is called!");
        return this.update(data);
    }
    signature.update.overload('java.nio.ByteBuffer').implementation = function (data) {
        console.log("Signature.update('java.nio.ByteBuffer') is called!");
        return this.update(data);
    }
    signature.update.overload('[B', 'int', 'int').implementation = function (data, start, length) {
        console.log("Signature.update('[B', 'int', 'int') is called!");
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " update data";
        toUtf8(tag, data);
        toHex(tag, data);
        toBase64(tag, data);
        console.log("=======================================================", start, length);
        return this.update(data, start, length);
    }
    signature.sign.overload('[B', 'int', 'int').implementation = function () {
        console.log("Signature.sign('[B', 'int', 'int') is called!");
        return this.sign.apply(this, arguments);
    }
    signature.sign.overload().implementation = function () {
        console.log("Signature.sign() is called!");
        var result = this.sign();
        var algorithm = this.getAlgorithm();
        var tag = algorithm + " sign result";
        toHex(tag, result);
        toBase64(tag, result);
        console.log("=======================================================");
        return result;
    }
}


setImmediate(function () {
    Java.perform(function () {
        hookCrypto()
    })
})

// frida -H 127.0.0.1:1234 -F -l hook_java_crypto.js -o log.txt

```

执行脚本并输出日志到 log.txt

```
frida -H 127.0.0.1:1234 -F -l hook_java_crypto.js -o log.txt

```

userName
========

userName 参数：

```
输入：13833854654
输出：36387881aef6d40c65c4825ec0d35868_1

```

通过日志输出可以知道：userName = AES ECB(手机号, 密钥) + _1

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmaiccDlHNq99f11HyuowTLIz76ic3KfEYccoWDtnvvzb0wpr8PYrxOPXQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image6.png

Cipher.init('int', 'java.security.Key') 方法初始化密钥，密钥是 ****************

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmPlCNlgTXQiaZIr39iazre0H9iaCOKe3weibzibiaZkDWkbwRK9IiboQORj5zQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image7.png

使用 CyberChef 验证一下：结果一致。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmnowK5GUWBbl0ZppVxoK25ia6c5VnIcxpibIHyfDJSiadWoB7icMcGjsY0w/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image8.png

password
========

password 参数：

```
输入：90909090
输出：f807a8a14205e5cdc6533430138c1179

```

password = md5(密码 + du)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmRsbEOzf7h18hm5m8UibbCS02be3yILc6icTZjp6mAzSY4r8pW7OxtrLw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image9.png

加密结果是：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYmiav61MukYRsD2HzvbSydgr6XRsP5TvlNgROONu3IIUvPd6TLgicibul2Q/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image10.png

使用 CyberChef 验证一下 ，就是用 md5 算法加密 (密码 + du)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jk9XcwbmvGsth8CMkL2nfYm8ibkuNVmlmx9okO8QhpXp2bc6EyVGibyM7TKgjOPu8BklbgPPvricwFyA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image11.png

timestamp
=========

使用 Python 获取 timestamp

```
def get_current_timestamp() -> str:
    """
    获取当前时间的 13 位毫秒级时间戳，并返回为字符串格式。

    返回:
        字符串形式的时间戳，例如 "1717085035150"
    """
    timestamp_ms = int(time.time() * 1000)
    return str(timestamp_ms)

```

使用 Python 还原加密算法
================

```
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import hashlib

def aes_ecb_encrypt(plaintext: str, key: str) -> bytes:
    key_bytes = key.encode('utf-8')
    data_bytes = pad(plaintext.encode('utf-8'), AES.block_size)  # PKCS7 padding
    cipher = AES.new(key_bytes, AES.MODE_ECB)
    encrypted = cipher.encrypt(data_bytes)
    print(f"[AES] 原文: {plaintext}")
    print(f"[AES] 密钥: {key}")
    print(f"[AES] 加密结果（Hex）: {encrypted.hex()}")
    return encrypted

def md5_hash(data: str) -> str:
    md5_result = hashlib.md5(data.encode('utf-8')).hexdigest()
    print(f"[MD5] Hash 结果: {md5_result}")
    return md5_result

def encrypt_username(username):
    return aes_ecb_encrypt(username, "****************").hex() + "_1"

def encrypt_password(password):
    return md5_hash(password + "du")

print(encrypt_username("13833854654"))
print(encrypt_password("90909090"))

```

输出如下：

```
[AES] 原文: 13833854654
[AES] 密钥: ****************
[AES] 加密结果（Hex）: 36387881aef6d40c65c4825ec0d35868
36387881aef6d40c65c4825ec0d35868_1
[MD5] Hash 结果: f807a8a14205e5cdc6533430138c1179
f807a8a14205e5cdc6533430138c1179

```

可以看到和 App 的参数加密结果是一致的。

#### 引用链接

`[1]` 安卓抓包实战：使用 Charles 抓取 App 数据全流程详解: _https://cyrus-studio.github.io/blog/posts/%E5%AE%89%E5%8D%93%E6%8A%93%E5%8C%85%E5%AE%9E%E6%88%98%E4%BD%BF%E7%94%A8-charles-%E6%8A%93%E5%8F%96-app-%E6%95%B0%E6%8D%AE%E5%85%A8%E6%B5%81%E7%A8%8B%E8%AF%A6%E8%A7%A3/_  
`[2]` 一文搞懂如何使用 Frida Hook Android App: _https://cyrus-studio.github.io/blog/posts/%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-frida-hook-android-app/_