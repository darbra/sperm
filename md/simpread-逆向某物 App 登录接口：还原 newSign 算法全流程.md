> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iSuJO8VCwd9MgH8oUEWkjw)

> 免责声明：文章中所有内容仅供学习交流使用，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【CYRUS STUDIO】联系作者

newSign 参数分析
============

通过 Hook Java 层加密算法得到 newSign 参数相关信息如下：

具体参考：逆向某物 App 登录接口：抓包分析 + Frida Hook 还原加密算法 [1]

入参：

```
MD5 update data Utf8: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+/dIN8Kof9Gm2x1kil7S/A+KLRtWKw+AfFWotfKtx+5J+ONciO*********************************************************************************************kK+Xiqtb6FajKK3aJ2vwB5l5lAKIhnvpOWXFWqYSQJy5g7oQ61Vwo+6MVB3U/wBT2CpM7AKDFH2Xj9Krb/0jNsPgNnA==  

```

MD5 加密后的结果：

```
MD5 digest result Hex: 8f03e2117c**********d9b9b18c58

```

调用堆栈：

```
MessageDigest.digest() is called!
java.lang.Throwable
        at java.security.MessageDigest.digest(Native Method)
        at ff.l0.h(RequestUtils.java:3)
        at ff.l0.c(RequestUtils.java:12)
        at lte.NCall.IL(Native Method)
        at com.shizhuang.duapp.common.helper.net.interceptor.HttpRequestInterceptor.intercept(HttpRequestInterceptor.java)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:10)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:1)
        at kb.b.intercept(MergeHostAfterInterceptor.java:11)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:10)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:1)
        at kb.d.intercept(MergeHostInterceptor.java:8)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:10)
        at okhttp3.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:1)
        at okhttp3.RealCall.getResponseWithInterceptorChain(RealCall.java:13)
        at okhttp3.RealCall.execute(RealCall.java:8)
        at retrofit2.OkHttpCall.execute(OkHttpCall.java:18)
        at retrofit2.adapter.rxjava2.CallExecuteObservable.subscribeActual(CallExecuteObservable.java:5)
        at ac2.m.subscribe(Observable.java:7)
        at retrofit2.adapter.rxjava2.BodyObservable.subscribeActual(BodyObservable.java:1)
        at ac2.m.subscribe(Observable.java:7)
        at pc2.j1.subscribeActual(ObservableMap.java:1)
        at ac2.m.subscribe(Observable.java:7)
        at io.reactivex.internal.operators.observable.ObservableRetryWhen$RepeatWhenObserver.subscribeNext(ObservableRetryWhen.java:5)
        at io.reactivex.internal.operators.observable.ObservableRetryWhen.subscribeActual(ObservableRetryWhen.java:7)
        at ac2.m.subscribe(Observable.java:7)
        at io.reactivex.internal.operators.observable.ObservableRetryBiPredicate$RetryBiObserver.subscribeNext(ObservableRetryBiPredicate.java:3)       
        at io.reactivex.internal.operators.observable.ObservableRetryBiPredicate.subscribeActual(ObservableRetryBiPredicate.java:4)
        at ac2.m.subscribe(Observable.java:7)
        at io.reactivex.internal.operators.observable.ObservableSubscribeOn$a.run(ObservableSubscribeOn.java:1)
        at io.reactivex.internal.schedulers.ScheduledDirectTask.call(ScheduledDirectTask.java:3)
        at io.reactivex.internal.schedulers.ScheduledDirectTask.call(ScheduledDirectTask.java:1)
        at java.util.concurrent.FutureTask.run(FutureTask.java:237)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
        at java.lang.Thread.run(Thread.java:761)

```

ff.l0.h
=======

根据调用堆栈 hook 一下 ff.l0.h 方法，打印的传参和返回值

```
/**
 * Hook 指定类的指定方法（包括所有重载）
 * @param {string} className - Java 类的完整名
 * @param {string} methodName - 方法名
 */
function hook_method(className, methodName) {
    Java.perform(function () {
        const Map = Java.use("java.util.Map");
        const MapEntry = Java.use("java.util.Map$Entry"); // 👈 必须显式声明 Map.Entry 类型

        const targetClass = Java.use(className);
        const overloads = targetClass[methodName].overloads;

        for (let i = 0; i < overloads.length; i++) {
            overloads[i].implementation = function () {
                let log = "\n================= HOOK START =================\n";
                log += "🎯 Class: " + className + "\n";
                log += "🔧 Method: " + methodName + "\n";
                log += "📥 Arguments:\n";

                for (let j = 0; j < arguments.length; j++) {
                    const arg = arguments[j];
                    try {
                        // 如果是 Map 类型打印 Map 中的内容
                        if (Map.class.isInstance(arg)) {
                            log += `  [${j}] Map content:\n`;
                            const entrySet = Java.cast(arg, Map).entrySet();
                            const iterator = entrySet.iterator();

                            while (iterator.hasNext()) {
                                const rawEntry = iterator.next();
                                const entry = Java.cast(rawEntry, MapEntry); // 👈 强制转换
                                const k = entry.getKey();
                                const v = entry.getValue();
                                log += `    ${k} => ${v}\n`;
                            }
                        } else {
                            log += `  [${j}]: ${arg.toString()}\n`;
                        }
                    } catch (e) {
                        log += `  [${j}]: ${arg}\n`;
                    }
                }

                const retval = this[methodName].apply(this, arguments);
                log += `📤 Return value: ${retval}\n`;
                log += "================== HOOK END ==================\n";
                console.log(log);
                return retval;
            };
        }
    });
}


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
    // hook_method('ff.l0', 'c')
    hook_all_methods("ff.l0");
});


// frida -H 127.0.0.1:1234 -F -l hook_class_methods.js

```

输出如下：

```
================= HOOK START =================
🎯 Class: ff.l0
🔧 Method: h
📥 Arguments:
  [0]: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+/dIN8Kof9Gm2x1kil7S/A+KLRtWKw+AfFWotfKtx+5J+ONciO*********************************************************************************************iuI9AfGYr9R817W8CfUGlVASAn1T6bq4D7DF1sHPqUITT76LLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==
📤 Return value: 997202002b**********37e2534113
================== HOOK END ==================

```

可以看出 ff.l0.h 的返回值和 md5 加密后的值是一样的，所以 ff.l0.h 其实就是一个 md5 加密方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHmL3nmOaoQd7vicJS1cprUepO82H9Vf2icM9g3PeZMp5FrvsiafO7F7KZw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image1.png

ff.l0.c
=======

再往上层 Hook ff.l0.c，输出如下：

```
================= HOOK START =================
🎯 Class: ff.l0
🔧 Method: c
📥 Arguments:
  [0] Map content:
    cipherParam => userName
    countryCode => 86
    password => 61f209b789**********6ad80b3a00
    type => pwd
    userName => 5f67625e05**********d138c2eb14_1
  [1]: 1750303548243
  [2]:
📤 Return value: 997202002b**********37e2534113
================== HOOK END ==================

```

第一个参数是一个 Map，存放的就是需要加密的请求参数。

dex 脱壳
======

使用 jadx 反编译 apk 并没有找到 ff.l0，应该是加了抽取壳，把 ff.l0 抽取到其他地方，运行时才恢复。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHMrYy7Nuicldx4l3PWy7oBCVbBxdpRbZdy94YCQiaibg8VI5VThNibic843g/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image2.png

使用 frida_dex_dump 脱壳 dex

```
function getProcessName() {
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


function mkdir(path) {
    var mkdirPtr = Module.getExportByName('libc.so', 'mkdir');
    var mkdir = new NativeFunction(mkdirPtr, 'int', ['pointer', 'int']);


    var opendirPtr = Module.getExportByName('libc.so', 'opendir');
    var opendir = new NativeFunction(opendirPtr, 'pointer', ['pointer']);

    var closedirPtr = Module.getExportByName('libc.so', 'closedir');
    var closedir = new NativeFunction(closedirPtr, 'int', ['pointer']);

    var cPath = Memory.allocUtf8String(path);
    var dir = opendir(cPath);
    if (dir != 0) {
        closedir(dir);
        return 0;
    }
    mkdir(cPath, 755);
    chmod(path);
}

function chmod(path) {
    var chmodPtr = Module.getExportByName('libc.so', 'chmod');
    var chmod = new NativeFunction(chmodPtr, 'int', ['pointer', 'int']);
    var cPath = Memory.allocUtf8String(path);
    chmod(cPath, 755);
}

function readStdString(str) {
    const isTiny = (str.readU8() & 1) === 0;
    if (isTiny) {
        return str.add(1).readUtf8String();
    }

    return str.add(2 * Process.pointerSize).readPointer().readUtf8String();
}

function findSymbolInLib(libname, keywordList) {
    const libBase = Module.findBaseAddress(libname);
    if (!libBase) {
        console.error("[-] Library not loaded:", libname);
        return null;
    }

    const matches = [];
    const symbols = Module.enumerateSymbolsSync(libname);
    for (const sym of symbols) {
        if (keywordList.every(k => sym.name.includes(k))) {
            matches.push(sym);
        }
    }

    if (matches.length === 0) {
        console.error("[-] No matching symbol found for keywords:", keywordList);
        return null;
    }

    const target = matches[0]; // 取第一个匹配的
    console.log("[+] Found symbol:", target.name, " @ ", target.address);
    return target.address;
}

function dumpDexToFile(filename, base, size) {
    // packageName
    var processName = getProcessName();

    if (processName != "-1") {
        // 判断是否以 .dex 结尾
        if (!filename.endsWith(".dex")) {
            filename += ".dex";
        }

        const dir = "/sdcard/Android/data/" + processName + "/dump_dex";
        const fullPath = dir + "/" + filename.replace(/\//g, "_").replace(/!/g, "_");

        // 创建目录
        mkdir(dir);

        // dump dex
        var fd = new File(fullPath, "wb");
        if (fd && fd != null) {
            var dex_buffer = ptr(base).readByteArray(size);
            fd.write(dex_buffer);
            fd.flush();
            fd.close();
            console.log("[+] Dex dumped to", fullPath);
        }
    }
}


function hookDexFileLoaderOpenCommon() {
    const addr = findSymbolInLib("libdexfile.so", ["DexFileLoader", "OpenCommon"]);
    if (!addr) return;

    Interceptor.attach(addr, {
        onEnter(args) {
            const base = args[0]; // const uint8_t* base
            const size = args[1].toInt32(); // size_t size
            const location_ptr = args[4]; // const std::string& location
            const location = readStdString(location_ptr);

            console.log("\n[*] DexFileLoader::OpenCommon called");
            console.log("    base       :", base);
            console.log("    size       :", size);
            console.log("    location   :", location);

            // 文件名
            const filename = location.split("/").pop();

            // 魔数
            var magic = ptr(base).readCString();
            console.log("    magic      :", magic)

            // dex 格式校验
            if (magic.indexOf("dex") !== -1) {
                dumpDexToFile(filename, base, size)
            }
        },
        onLeave(retval) {}
    });
}

setImmediate(hookDexFileLoaderOpenCommon);


// frida -H 127.0.0.1:1234 -l dump_dex_from_open_common.js -f com.cyrus.example

```

参考：

*   • ART 下 Dex 加载流程源码分析 和 通用脱壳点 [2]
    
*   • https://github.com/CYRUS-STUDIO/frida_dex_dump
    

日志输出如下：

```
[+] Found symbol: _ZN3art13DexFileLoader10OpenCommonEPKhmS2_mRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPKNS_10OatDexFileEbbPS9_NS3_10unique_ptrINS_16DexFileContainerENS3_14default_deleteISH_EEEEPNS0_12VerifyResultE  @  0x7be3891c28
Spawned `com.shizhuang.duapp`. Use %resume to let the main thread start executing!
[Remote::com.shizhuang.duapp]-> %resume
[Remote::com.shizhuang.duapp]->
[*] DexFileLoader::OpenCommon called
    base       : 0x7bd87de02c
    size       : 450032
    location   : /system/framework/org.apache.http.legacy.jar
    magic      : dex
039
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/org.apache.http.legacy.jar.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7bd87de02c
    size       : 450032
    location   : /system/framework/org.apache.http.legacy.jar
    magic      : dex
039
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/org.apache.http.legacy.jar.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b75c69000
    size       : 8681372
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b7471e000
    size       : 12888744
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes2.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes2.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b73b1b000
    size       : 12592256
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes3.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes3.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b72f75000
    size       : 12213596
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes4.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes4.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b7254f000
    size       : 10637856
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes5.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes5.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b71d5e000
    size       : 8324572
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes6.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes6.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b71a3e000
    size       : 3273924
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes7.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes7.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b71270000
    size       : 8183732
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes8.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes8.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b706ff000
    size       : 11994176
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes9.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes9.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6fc62000
    size       : 11125808
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes10.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes10.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6f0a2000
    size       : 12319700
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes11.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes11.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6e4e6000
    size       : 12300396
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes12.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes12.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6d8e4000
    size       : 12587972
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes13.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes13.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6cd5e000
    size       : 12081268
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes14.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes14.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b6c15c000
    size       : 12590752
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes15.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes15.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7bd2eb7000
    size       : 1260244
    location   : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/base.apk!classes16.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/base.apk_classes16.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b5e4497fc
    size       : 3782924
    location   : /system/product/app/webview/webview.apk
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/webview.apk.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7bcd38e138
    size       : 77880
    location   : /system/product/app/webview/webview.apk!classes2.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/webview.apk_classes2.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7b5e4497fc
    size       : 3782924
    location   : /system/product/app/webview/webview.apk
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/webview.apk.dex

[*] DexFileLoader::OpenCommon called
    base       : 0x7bcd38e138
    size       : 77880
    location   : /system/product/app/webview/webview.apk!classes2.dex
    magic      : dex
035
[+] Dex dumped to /sdcard/Android/data/com.shizhuang.duapp/dump_dex/webview.apk_classes2.dex

```

把所有 dex 拉取到本地

```
adb pull /sdcard/Android/data/com.shizhuang.duapp/dump_dex

```

反编译 dex
=======

通过 grep 命令查找 "ff/l0" 在哪些 dex 有引用

```
wayne:/sdcard/Android/data/com.shizhuang.duapp/dump_dex # grep -rl "ff/l0" *.dex
base.apk_classes10.dex
base.apk_classes11.dex
base.apk_classes12.dex
base.apk_classes13.dex
base.apk_classes14.dex
base.apk_classes15.dex
base.apk_classes2.dex
base.apk_classes4.dex
base.apk_classes5.dex
base.apk_classes6.dex
base.apk_classes7.dex
base.apk_classes9.dex

```

使用 dex2jar 批量转换 dex

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHiapo3EGHiavx1777Hk9RXgaqHjoIWmic94u1Rr2Uul3EHzGdarAFXqVCw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image3.png

  
参考：FART 脱壳某大厂 App + CodeItem 修复 dex + 反编译还原源码 [3]

使用 jadx 反编译 ff.l0.c 方法得到源码如下：

```
public static String c(Map<String, Object> map, long j, String str) {
    synchronized (l0.class) {
        try {
            if (map == null) {
                return "";
            }
            map.put("uuid", he.a.i.t());
            map.put("platform", "android");
            map.put(NotifyType.VIBRATE, he.a.i.b());
            if (str == null) {
                str = "";
            }
            map.put("loginToken", str);
            map.put("timestamp", String.valueOf(j));
            String i = i(map);
            he.a.m.d(TAG, "StringToSign-body use Gson " + i);
            String doWork = DuHelper.doWork(he.a.h, i);
            map.remove("uuid");
            return h(doWork);
        } finally {
        }
    }
}

```

ff.l0.c 中调用 i 方法把参数拼接成字符串。

```
/**
 * 将传入的 Map<String, Object> 转换成字符串形式（key + value 拼接），
 * 处理数组、集合、JsonArray 等特殊类型；其中特殊类型会转成以 "," 分隔的字符串。
 */
public static String i(Map<String, Object> map) {

    // 最终用于拼接所有 key-value 的结果字符串
    StringBuilder sb3 = new StringBuilder();

    // 遍历 map 中的每一项
    for (Map.Entry<String, Object> entry : map.entrySet()) {
        sb3.append(entry.getKey()); // 拼接 key

        Object value = entry.getValue(); // 获取 value

        // 如果是 org.json.JSONArray 类型，打印警告（不建议使用此类型）
        if (value instanceof org.json.JSONArray) {
            a.l lVar = he.a.m; // 获取日志工具（推测）
            StringBuilder d = a.d.d("Please Not use this params type: ");
            d.append(value.getClass());
            // 打印错误日志 + 堆栈信息
            lVar.c(d.toString(), new Throwable());
        }

        // 如果是 Java 原生数组类型
        if (value != null && value.getClass().isArray()) {
            int length = Array.getLength(value);
            ArrayList arrayList = new ArrayList();
            for (int i = 0; i < length; i++) {
                // 使用 id.e.o() 方法处理数组每个元素后加入列表
                arrayList.add(id.e.o(Array.get(value, i)));
            }
            // 将所有元素用 "," 拼接后追加到结果中
            sb3.append(TextUtils.join(",", arrayList));
        }
        // 如果是集合类（List、Set）或 JsonArray
        else if ((value instanceof Collection) || (value instanceof JsonArray)) {
            Iterable iterable = (Iterable) value;
            ArrayList arrayList2 = new ArrayList();
            Iterator it = iterable.iterator();
            while (it.hasNext()) {
                // 使用 id.e.o() 方法处理每个元素后加入列表
                arrayList2.add(id.e.o(it.next()));
            }
            // 将所有元素用 "," 拼接后追加到结果中
            sb3.append(TextUtils.join(",", arrayList2));
        }
        // 其他类型，直接处理后拼接
        else {
            sb3.append(id.e.o(value));
        }
    }

    // 返回拼接好的字符串
    return sb3.toString();
}

```

id.e.o() 方法，将任意对象 obj 转换成 String 类型

```
@NonNull
public static String o(@Nullable Object obj) {
    String n = n(obj);
    String str = n;
    if (n == null) {
        str = "";
    }
    return str;
}

@Nullable
public static String n(@Nullable Object obj) {
    if (obj == null) {
        return null;
    }
    // 判断对象是否需要用 Gson 转 JSON
    Class<?> cls = obj.getClass();
    if (!(obj instanceof CharSequence) && !f.a(cls)) {
        try {
            return k().toJson(obj);
        } catch (Exception e2) {
            HashMap hashMap = new HashMap();
            hashMap.put("class", cls.toString());
            hashMap.put("json", obj.toString());
            he.a.j.c(e2, "app_error_GsonHelper_toJson", hashMap);
            return null;
        }
    }
    // 如果是 Float 或 Double，并且数值是整数（如 3.0），就转为整数形式字符串（"3" 而不是 "3.0"）
    if ((obj instanceof Float) || (obj instanceof Double)) {
        Number number = (Number) obj;
        if (number.doubleValue() == number.longValue()) {
            valueOf = String.valueOf(number.longValue());
        }
    }
    // 其余类型直接用 String.valueOf 转换
    valueOf = String.valueOf(obj);
    return valueOf;
}

```

参数最终是调用了 DuHelper.doWork 进行加密。

```
String doWork = DuHelper.doWork(he.a.h, i);

```

he.a.h 是 Context，i 是拼接后的参数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHhhkH1uBO6ph32aBic9oSaGXMXv7y6483g0TtdMvYGksHSMiacFvricPDQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image4.png

doWork 返回的字符串再调用 ff.l0.h（md5） 方法加密。

```
public static String h(String str) {
    try {
        MessageDigest messageDigest = MessageDigest.getInstance("MD5");
        messageDigest.update(str.getBytes());
        byte[] digest = messageDigest.digest();
        StringBuilder sb3 = new StringBuilder();
        for (byte b : digest) {
            String hexString = Integer.toHexString(b & 255);
            while (hexString.length() < 2) {
                hexString = PushConstants.PUSH_TYPE_NOTIFY + hexString;
            }
            sb3.append(hexString);
        }
        return sb3.toString();
    } catch (NoSuchAlgorithmException e2) {
        e2.printStackTrace();
        return "";
    }
}

```

DuHelper.doWork
===============

查找 DuHelper 所在的 dex

```
wayne:/sdcard/Android/data/com.shizhuang.duapp/dump_dex # grep -rl "DuHelper" .
./base.apk_classes9.dex
./base.apk.dex

```

使用 jadx 反编译

```
package com.shizhuang.duapp.common.helper.ee;

import com.meituan.robust.ChangeQuickRedirect;
import lte.NCall;

/* loaded from: base.apk_classes9.jar:com/shizhuang/duapp/common/helper/ee/DuHelper.class */
public class DuHelper {
    public static ChangeQuickRedirect changeQuickRedirect;

    static {
        NCall.IV(new Object[]{282});
    }

    public static native int checkSignature(Object obj);

    public static String doWork(Object obj, String str) {
        return (String) NCall.IL(new Object[]{283, obj, str});
    }

    public static native String encodeByte(byte[] bArr, String str);

    public static native String getByteValues();

    public static native String getLeanCloudAppID();

    public static native String getLeanCloudAppKey();

    public static native String getWxAppId(Object obj);

    public static native String getWxAppKey();
}

```

DuHelper.doWork 是调用 lte.NCall.IL 进行加密

```
return (String) NCall.IL(new Object[]{283, obj, str});

```

lte.NCall.IL
============

查找 NCall 所在的 dex

```
1|wayne:/sdcard/Android/data/com.shizhuang.duapp/dump_dex # grep -rl "NCall" .
./base.apk_classes5.dex
./base.apk_classes9.dex
./base.apk_classes10.dex
./base.apk_classes11.dex

```

使用 jadx 反编译

```
package lte;

/* loaded from: base.apk_classes11.jar:lte/NCall.class */
public class NCall {
    static {
        System.loadLibrary("GameVMP");
    }

    public static native byte IB(Object[] objArr);

    public static native char IC(Object[] objArr);

    public static native double ID(Object[] objArr);

    public static native float IF(Object[] objArr);

    public static native int II(Object[] objArr);

    public static native long IJ(Object[] objArr);

    public static native Object IL(Object[] objArr);

    public static native short IS(Object[] objArr);

    public static native void IV(Object[] objArr);

    public static native boolean IZ(Object[] objArr);

    public static native int dI(int i);

    public static native long dL(long j);

    public static native String dS(String str);
}

```

hook lte.NCall.IL 方法并打印参数和结果

```
function hook_NCall_IL() {
    // 获取类 lte/NCall
    var NCall = Java.use("lte.NCall");

    // Hook 静态方法 IL([Ljava/lang/Object;)Ljava/lang/Object;
    NCall.IL.overload('[Ljava.lang.Object;').implementation = function (args) {

        // 合并日志
        var logMessage = "Hooked lte/NCall->IL() method\n";

        // 打印传入的参数
        logMessage += "Arguments: [";
        for (var i = 0; i < args.length; i++) {
            logMessage += args[i].toString();
            if (i < args.length - 1) {
                logMessage += ", ";
            }
        }
        logMessage += "]\n";

        // 调用原始方法并获取返回值
        var result = this.IL(args);

        // 打印返回值
        logMessage += "Result: " + result;

        // 输出合并的信息
        console.log(logMessage);

        // 返回结果
        return result;
    };
}

setImmediate(function () {
    Java.perform(hook_NCall_IL);
})

```

执行脚本

```
frida -H 127.0.0.1:1234 -F -l hook_NCall.js

```

输出如下：

```
Hooked lte/NCall->IL() method
Arguments: [283, com.shizhuang.duapp.modules.app.DuApplication@e7edb59, loginTokenplatformandroidtimestamp1728414660226****************fb63v5.43.0]     
Result: knQQXR0br7Lqn4eabvJsdZ4D96wrRcYi2zPW************************************uZgrvFlZJ0mCmQBrhQQOR1PtwTx8iu3Yfc4=

```

有点像 VMP 壳，283 是 index。

libGameVMP.so 脱壳
================

lte.NCall.IL 是一个 native 方法，具体实现在 libGameVMP.so

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHuicn3sqaZpAu81qQZUib06MZpHUwiaQzVDXeLCZj5MRvqxicTcFcCKJ61A/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image5.png

用 IDA 打开 so 会报错，说明 so 应该是加了壳。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhH8UcFyra7aibhLBtNAjdRMoCDibsJqsbeYUqBTcLPno3J0Z9AEpk5wS6A/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image6.png![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhH0wyFcibfpvFOf1TbbdUSicDOfXfTCZicaAfMhPUJP0azw95K5IicwPClJA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image7.png

通过 frida_dump[4] 脱掉 so 的壳，并用 SoFixer 修复 so。

具体参考：一文搞懂 SO 脱壳全流程：识别加壳、Frida Dump、原理深入解析 [5]

找到 lte.NCall.IL 的 JNI 方法入口
==========================

脱壳后的 so 中找不到 NCall.IL 方法，说明是动态注册的。

通过 frida 打印 lte.NCall 类中所有 JNI 方法信息如下：

```
[+] Found native method: _Z32android_os_Process_getUidForNameP7_JNIEnvP8_jobjectP8_jstring @ 0x7c65c0d648
========== [ JNI Method Info Dump ] ==========
[*] Target class: lte.NCall
[*] entry_point_from_jni_ offset = 24 bytes

------------ [ #1 Native Method ] ------------
Method Name     : public static native byte lte.NCall.IB(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a590
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #2 Native Method ] ------------
Method Name     : public static native char lte.NCall.IC(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a5b8
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #3 Native Method ] ------------
Method Name     : public static native double lte.NCall.ID(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a5e0
Native Addr     : 0x7b6b455028
Module Name     : libGameVMP.so
Module Offset   : 0xe028
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #4 Native Method ] ------------
Method Name     : public static native float lte.NCall.IF(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a608
Native Addr     : 0x7b6b454fe8
Module Name     : libGameVMP.so
Module Offset   : 0xdfe8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #5 Native Method ] ------------
Method Name     : public static native int lte.NCall.II(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a630
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #6 Native Method ] ------------
Method Name     : public static native long lte.NCall.IJ(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a658
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #7 Native Method ] ------------
Method Name     : public static native java.lang.Object lte.NCall.IL(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a680
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #8 Native Method ] ------------
Method Name     : public static native short lte.NCall.IS(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a6a8
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #9 Native Method ] ------------
Method Name     : public static native void lte.NCall.IV(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a6d0
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #10 Native Method ] ------------
Method Name     : public static native boolean lte.NCall.IZ(java.lang.Object[])
ArtMethod Ptr   : 0x9f63a6f8
Native Addr     : 0x7b6b454fa8
Module Name     : libGameVMP.so
Module Offset   : 0xdfa8
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #11 Native Method ] ------------
Method Name     : public static native int lte.NCall.dI(int)
ArtMethod Ptr   : 0x9f63a720
Native Addr     : 0x7b6b45293c
Module Name     : libGameVMP.so
Module Offset   : 0xb93c
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #12 Native Method ] ------------
Method Name     : public static native long lte.NCall.dL(long)
ArtMethod Ptr   : 0x9f63a748
Native Addr     : 0x7b6b452ad0
Module Name     : libGameVMP.so
Module Offset   : 0xbad0
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

------------ [ #13 Native Method ] ------------
Method Name     : public static native java.lang.String lte.NCall.dS(java.lang.String)
ArtMethod Ptr   : 0x9f63a770
Native Addr     : 0x7b6b452aec
Module Name     : libGameVMP.so
Module Offset   : 0xbaec
Module Base     : 0x7b6b447000
Module Size     : 462848 bytes
Module Path     : /data/app/com.shizhuang.duapp-fTxemmnM8l6298xbBELksQ==/lib/arm64/libGameVMP.so
------------------------------------------------

[*] Total native methods found: 13
===============================================

```

参考：逆向 JNI 函数找不到入口？动态注册定位技巧全解析 [6]

找到 lte.NCall.IL(java.lang.Object[]) 方法在 libGameVMP.so 偏移 0xdfa8 的位置

```
__int64 __fastcall sub_DFA8(JNIEnv *env, jclass clazz, jobjectArray arr)
{
  return sub_17EB8((__int64)env, (__int64)arr);
}

```

OLLVM bcf（虚假控制流）
================

NCall.IL 实际调用的是 sub_17EB8 函数，而且函数内部大量引用了 x y 开头的全局变量。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHnUjTSGXdibrOodia3bYudA37csYxkxJpVqfwCtY01VNLRy0CNibGNc6tA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image8.png

  
这个其实是做了 OLLVM 虚假控制流（bcf）混淆，通过伪条件隐藏真实的代码执行流。

关于 OLLVM 具体参考：

*   • 移植 OLLVM 到 LLVM 18，C&C++ 代码混淆 [7]
    
*   • 移植 OLLVM 到 Android NDK，Android Studio 中使用 OLLVM[8]
    
*   • OLLVM 增加 C&C++ 字符串加密功能 [9]
    

使用 Frida 反 OLLVM
================

1. 固定参数
-------

为了方便分析，先固定一下 lte.NCall.IL 方法的调用参数

```
function NCall_IL() {
    Java.perform(() => {
        const Integer = Java.use("java.lang.Integer");
        const String = Java.use("java.lang.String");
        const DuApplication = Java.use("com.shizhuang.duapp.modules.app.DuApplication");
        const NCall = Java.use("lte.NCall");

        // 1. 创建 Integer 对象（包装 int）
        const arg0 = Integer.valueOf(283);

        // 2. 获取静态字段 instance
        const arg1 = DuApplication.instance.value;

        // 3. 构造字符串参数
        const arg2 = String.$new(
            "cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0"
        );

        // 构造 Object[] 参数数组
        const argsArray = Java.array("java.lang.Object", [arg0, arg1, arg2]);

        // 5. 调用 NCall.IL(Object[])
        const result = NCall.IL(argsArray);

        // 6. 打印结果
        console.log("NCall.IL 返回值：", result);
    });
}

```

调用返回结果如下：

```
[Remote::**]-> NCall_IL()
NCall.IL 返回值： dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+/dIN8Kof9Gm2x1kil7S/VILfEPi7ImlGxmmwj6+taHk6jQ4T********************************************************************************************rp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

2. 获取加密串调用堆栈
------------

通过 Hook jstring 相关 JNI 接口，快速定位加密算法的具体位置，越过 OLLVM 混淆 + VMP 壳

参考：破解 VMP+OLLVM 混淆：通过 Hook jstring 快速定位加密算法入口 [10]

得到调用堆栈如下：

```
[Remote::cyrus]-> NCall_IL()

====== 🧪 NewStringUTF Hook ======
📥 Input C String: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+**********************************************************************************************************************************************xrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==
🔍 Backtrace:
0x7b627e185c libdewuhelper.so!encode+0x138!+0x185c
0x7b6ca0f388 base.odex!0x808388!+0x808388
📤 Returned Java String: 0x99
====== ✅ Hook End ======


====== 🧪 GetStringChars Hook ======
📥 jstring: 0x15
📥 isCopy: 0x0
📤 UTF-16 String: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ*********************************fEPi7ImlGxmmwj6+taHk6jQ4Tog7XzBbL
====== ✅ Hook End ======

NCall.IL 返回值： dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+/dIN8Kof9Gm2x1kil7S/VILfEPi7ImlGxmmwj6+taHk6jQ4T********************************************************************************************rp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

从日志输出可以知道：NewStringUTF 在 libdewuhelper.so 的 encode 函数中被调用，在 so 偏移 0x185c 处。

libdewuhelper.so
================

使用 frida dump 脱壳 libdewuhelper.so

```
python dump_so.py libdewuhelper.so

```

使用 IDA 反汇编 libdewuhelper.so 的 encode 方法如下：

```
jstring __fastcall encode(JNIEnv *a1, __int64 a2, jbyteArray a3, jstring a4)
{
  const char *v7; // x23
  void *Value; // x20
  unsigned int v9; // w25
  jbyte *v10; // x24
  jbyte *v11; // x0
  jbyte *v12; // x26
  __int64 v13; // x9
  jbyte *v14; // x10
  jbyte *v15; // x11
  __int64 v16; // x8
  jbyte v17; // t1
  char *v18; // x25
  jstring v19; // x19
  __int128 *v21; // x10
  _OWORD *v22; // x11
  __int64 v23; // x12
  __int128 v24; // q0
  __int128 v25; // q1

  v7 = (*a1)->GetStringUTFChars(a1, a4, 0LL);
  Value = (void *)j_getValue();
  v9 = (*a1)->GetArrayLength(a1, a3);
  v10 = (*a1)->GetByteArrayElements(a1, a3, 0LL);
  v11 = (jbyte *)malloc(v9 + 1);
  v12 = v11;
  if ( (int)v9 >= 1 )
  {
    if ( v9 <= 0x1F || v11 < &v10[v9] && v10 < &v11[v9] )
    {
      v13 = 0LL;
LABEL_6:
      v14 = &v11[v13];
      v15 = &v10[v13];
      v16 = v9 - v13;
      do
      {
        v17 = *v15++;
        --v16;
        *v14++ = v17;
      }
      while ( v16 );
      goto LABEL_8;
    }
    v13 = v9 & 0x7FFFFFE0;
    v21 = (__int128 *)(v10 + 16);
    v22 = v11 + 16;
    v23 = v9 & 0xFFFFFFE0;
    do
    {
      v24 = *(v21 - 1);
      v25 = *v21;
      v21 += 2;
      v23 -= 32LL;
      *(v22 - 1) = v24;
      *v22 = v25;
      v22 += 2;
    }
    while ( v23 );
    if ( v13 != v9 )
      goto LABEL_6;
  }
LABEL_8:
  v11[v9] = 0;
  v18 = (char *)j_AES_128_ECB_PKCS5Padding_Encrypt(v11, Value);
  free(v12);
  (*a1)->ReleaseStringUTFChars(a1, a4, v7);
  (*a1)->ReleaseByteArrayElements(a1, a3, v10, 0LL);
  v19 = (*a1)->NewStringUTF(a1, v18);
  if ( v18 )
    free(v18);
  if ( Value )
    free(Value);
  return v19;
}

```

encode 方法中用到的 JNI 函数如下，可以根据 JNI 函数原型去还原 encode 方法中的参数类型。

```
    const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
    jsize       (*GetArrayLength)(JNIEnv*, jarray);
    jbyte*      (*GetByteArrayElements)(JNIEnv*, jbyteArray, jboolean*);
    void        (*ReleaseStringUTFChars)(JNIEnv*, jstring, const char*);
    void        (*ReleaseByteArrayElements)(JNIEnv*, jbyteArray, jbyte*, jint);
    jstring     (*NewStringUTF)(JNIEnv*, const char*);

```

https://cs.android.com/android/platform/superproject/+/android10-release:libnativehelper/include_jni/jni.h;l=378

返回值 v19 来自于 v18，是 j_AES_128_ECB_PKCS5Padding_Encrypt 方法的返回值

```
v18 = (char *)j_AES_128_ECB_PKCS5Padding_Encrypt(v11, Value);

```

v11 通过与 v10 的相关计算得到，而 v10 的值来自于 a3。

Value 的值是一个通用类型指针

```
Value = (void *)j_getValue();

```

来自于 getValue_ptr() 的调用

```
// attributes: thunk
__int64 j_getValue(void)
{
  return getValue_ptr();
}

```

getValue_ptr 是一个函数指针，指向 getValue()，偏移为 0x5FB8，类型为：__int64 (*getValue_ptr)(void)

```
.data:0000000000005FB8                               ; __int64 (*getValue_ptr)(void)
.data:0000000000005FB8 0C 16 00 00 00 00 00 00       getValue_ptr DCQ getValue               ; DATA XREF: j_getValue↑o
.data:0000000000005FB8                                                                       ; j_getValue+4↑r
.data:0000000000005FB8                                                                       ; j_getValue+8↑o

```

encode 函数分析
===========

使用 frida 打印一下 encode 的参数和返回值看看

```
[+] encode 函数地址: 0x7b62808724
[Remote::**]-> NCall_IL()
[>] a2 pointer: 0x7b625c5ea4
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7b625c5ea4  48 f2 7a 9e 40 32 30 14 70 31 30 14 02 00 00 00  H.z.@20.p10.....
7b625c5eb4  00 00 00 00 90 28 30 14 00 00 00 00 00 00 00 00  .....(0.........
7b625c5ec4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7b625c5ed4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[>] jbyteArray (length=195):
00000000  63 69 70 68 65 72 50 61 72 61 6d 75 73 65 72 4e  cipherParamuserN
00000010  61 6d 65 63 6f 75 6e 74 72 79 43 6f 64 65 38 36  amecountryCode86
00000020  6c 6f 67 69 6e 54 6f 6b 65 6e 70 61 73 73 77 6f  loginTokenpasswo
00000030  72 64 36 37 31 36 63 35 38 64 63 33 32 65 39 36  rd6716c58dc32e96
00000040  66 38 38 39 61 30 33 35 64 30 63 31 37 34 39 30  f889a035d0c17490
00000050  62 65 70 6c 61 74 66 6f 72 6d 61 6e 64 72 6f 69  beplatformandroi
00000060  64 74 69 6d 65 73 74 61 6d 70 31 37 34 34 30 34  dtimestamp174404
00000070  32 31 39 35 37 34 33 74 79 70 65 70 77 64 75 73  2195743typepwdus
00000080  65 72 4e 61 6d 65 66 33 37 62 66 61 31 34 30 35  erNamef37bfa1405
00000090  37 63 66 30 31 38 30 31 31 64 62 36 37 63 39 36  7cf018011db67c96
000000a0  33 63 64 37 33 33 5f 31 75 75 69 64 34 63 33 61  3cd733_1********
000000b0  39 62 33 38 31 38 32 38 66 62 36 33 76 35 2e 34  9b381828fb63v5.4
000000c0  33 2e 30                                         3.0
[>] jstring a4: "0101101000100010100100100000110001110010111010101010001011101110****************************************************************1111001011100010101000100100110010110010100010101011110010111100"
[<] encode 返回值: "dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQX******************************************k6jQ4Tog7XzBbLATwfAwFewviaX1/8WS4J271k/SPo
cXykU4wnASDm+kFk63OxOynX9B1wA42cTOy3rHZ3W/ll1gBxtH5hmdGpnYqxrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA=="
NCall.IL 返回值： dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC*******************************************6jQ4Tog7XzBbLATwfAwFewviaX1/8WS4J271k/SPocX
ykU4wnASDm+kFk63OxOynX9B1wA42cTOy3rHZ3W/ll1gBxtH5hmdGpnYqxrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

从日志可以知道

*   • jbyteArray a3 就是原始的参数数据
    
*   • encode 返回值 和 NCall.IL 返回值 是一样的
    

getValue 函数分析
=============

IDA 反汇编代码中 getValue 函数原型如下：

```
__int64 __fastcall getValue(const char *a1)

```

getValue 函数最后调用的是 j_b64_decode 函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHMzWIa4Jlx4TjPM2VQ7Md0RcFTRs9MECe8L9ibdKd306RXjj9hVDljcA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image9.png

按 X 查找 j_b64_decode 函数的交叉引用，找到 j_b64_decode 的返回值类型其实是 char *

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHsicb7WOGoO7m1TQVa9JuLO22v5ibqsV5dBnlsoib34PVeibphMow94OMTA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image10.png

所以 getValue 的真实函数原型应该如下：

```
char* getValue(const char *a1)

```

hook getValue 函数并打印传参和返回值

```
/**
 * hook getValue 函数并打印参数和返回值
 */
function hookGetValue() {
    const moduleName = "libdewuhelper.so";
    const funcOffset = 0x160C;

    // 获取模块基址
    const base = Module.findBaseAddress(moduleName);
    if (!base) {
        console.error("[!] 模块未加载:", moduleName);
        return;
    }

    const funcAddr = base.add(funcOffset);
    console.log("[+] getValue 函数地址:", funcAddr);

    // Hook 函数
    Interceptor.attach(funcAddr, {
        onEnter(args) {
            this.argStr = Memory.readCString(args[0]);
            console.log(`[*] getValue called with arg: "${this.argStr}"`);
        },
        onLeave(retval) {
            const retStr = Memory.readCString(retval);
            console.log(`[+] getValue returned: ${retval} -> "${retStr}"`);
        }
    });
}


// Java 调用 native 方法示例
function NCall_IL() {
    Java.perform(() => {
        const Integer = Java.use("java.lang.Integer");
        const String = Java.use("java.lang.String");
        const DuApplication = Java.use("com.shizhuang.duapp.modules.app.DuApplication");
        const NCall = Java.use("lte.NCall");

        const arg0 = Integer.valueOf(283);
        const arg1 = DuApplication.instance.value;
        const arg2 = String.$new("cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0");

        const argsArray = Java.array("java.lang.Object", [arg0, arg1, arg2]);
        const result = NCall.IL(argsArray);
        console.log("NCall.IL 返回值：", result);
    });
}


setImmediate(getValue)

// frida -H 127.0.0.1:1234 -F -l getValue.js -o log.txt

```

输出如下：

```
[+] getValue 函数地址: 0x7b6280860c
[Remote::**]-> NCall_IL()
[*] getValue called with arg: "0101101000100010100100100000110001110010111010101010001011101110****************************************************************1111001011100010101000100100110010110010100010101011110010111100"
[+] getValue returned: 0x7bd7646280 -> "****************"
NCall.IL 返回值： dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+/dIN8Kof9Gm2x1kil7S/VILfEPi7ImlGxmmwj6+taHk6jQ4T********************************************************************************************rp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

得到 AES 加密密钥：****************

AES_128_ECB_PKCS5Padding_Encrypt 函数分析
=====================================

j_AES_128_ECB_PKCS5Padding_Encrypt 实际调用的是 AES_128_ECB_PKCS5Padding_Encrypt 函数

```
__int64 __fastcall AES_128_ECB_PKCS5Padding_Encrypt(__int64 a1, __int64 a2)
{
  ...
  do
  {
    j_AES128_ECB_encrypt(&v8[v30], a2, &v29[v30]);
    --v31;
    v30 += 16LL;
  }
  while ( v31 );
LABEL_68:
  j_b64_encode(v29, v28);
  return init_proc(v8);
}

```

AES_128_ECB_PKCS5Padding_Encrypt 里面调用 j_AES128_ECB_encrypt 加密数据

```
__int64 __fastcall AES128_ECB_encrypt(unsigned __int8 *a1, __int64 a2, int8x16_t *a3)

```

并使用 j_b64_encode 编码

```
void *__fastcall b64_encode(char *a1, __int64 a2)

```

通过分析 AES_128_ECB_PKCS5Padding_Encrypt 汇编代码得知：

*   • a1 是需要加密的参数，类型是 char*
    
*   • a2 是一个固定的数字，而且在加密方法里面没有用到
    
*   • a3 加密输出的 buffer
    
*   • 返回值是加密串的长度
    

所以 AES128_ECB_encrypt 方法原型实际上应该是这样：

```
__int64 AES128_ECB_encrypt(char *a1, __int64 a2, char *a3)

```

hook AES128_ECB_encrypt 方法并打印参数和返回值看看：

```
function AES128_ECB_encrypt() {
    const soName = "libdewuhelper.so";
    const funcName = "AES128_ECB_encrypt";

    const funcAddr = Module.getExportByName(soName, funcName);
    console.log("[+] AES128_ECB_encrypt 地址:", funcAddr);

    Interceptor.attach(funcAddr, {
        onEnter(args) {
            this.inputPtr = args[0];
            this.a2 = args[1].toInt32();
            this.outputPtr = args[2];
            this.log = "";

            this.log += "\n======= AES128_ECB_encrypt =======\n";
            this.log += `[>] 明文地址 a1 = ${this.inputPtr}\n`;
            this.log += `[>] a2 = ${this.a2}\n`;
            this.log += `[>] 输出缓冲区地址 a3 = ${this.outputPtr}\n`;
            this.log += "[>] 明文内容:\n";
            this.log += hexdump(this.inputPtr, {
                offset: 0,
                length: 256,
                header: true,
                ansi: false
            }) + "\n";
        },

        onLeave(retval) {
            const encryptedLen = retval.toInt32();
            this.log += `[<] 返回值：加密结果长度 = ${encryptedLen}\n`;
            this.log += "[<] 密文内容:\n";
            this.log += hexdump(this.outputPtr, {
                offset: 0,
                length: Math.min(encryptedLen, 256),
                header: true,
                ansi: false
            }) + "\n";
            this.log += "======= AES128_ECB_encrypt END =======\n";
            console.log(this.log);
        }
    });
}


// Java 调用 native 方法示例
function NCall_IL() {
    Java.perform(() => {
        const Integer = Java.use("java.lang.Integer");
        const String = Java.use("java.lang.String");
        const DuApplication = Java.use("com.shizhuang.duapp.modules.app.DuApplication");
        const NCall = Java.use("lte.NCall");

        const arg0 = Integer.valueOf(283);
        const arg1 = DuApplication.instance.value;
        const arg2 = String.$new("cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0");

        const argsArray = Java.array("java.lang.Object", [arg0, arg1, arg2]);
        const result = NCall.IL(argsArray);
        console.log("NCall.IL 返回值：", result);
    });
}

setImmediate(function () {
    Java.perform(function () {
        AES128_ECB_encrypt()
    });
})

// frida -H 127.0.0.1:1234 -F -l AES128_ECB_encrypt.js -o log.txt

```

输出如下：

```
[+] AES128_ECB_encrypt 地址: 0x7b628093d0
[Remote::**]-> NCall_IL()

======= AES128_ECB_encrypt =======
[>] 明文地址 a1 = 0x7bd768cf00
[>] a2 = -681286304
[>] 输出缓冲区地址 a3 = 0x7bd768d0c0
[>] 明文内容:
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7bd768cf00  63 69 70 68 65 72 50 61 72 61 6d 75 73 65 72 4e  cipherParamuserN
7bd768cf10  61 6d 65 63 6f 75 6e 74 72 79 43 6f 64 65 38 36  amecountryCode86
7bd768cf20  6c 6f 67 69 6e 54 6f 6b 65 6e 70 61 73 73 77 6f  loginTokenpasswo
7bd768cf30  72 64 36 37 31 36 63 35 38 64 63 33 32 65 39 36  rd6716c58dc32e96
7bd768cf40  66 38 38 39 61 30 33 35 64 30 63 31 37 34 39 30  f889a035d0c17490
7bd768cf50  62 65 70 6c 61 74 66 6f 72 6d 61 6e 64 72 6f 69  beplatformandroi
7bd768cf60  64 74 69 6d 65 73 74 61 6d 70 31 37 34 34 30 34  dtimestamp174404
7bd768cf70  32 31 39 35 37 34 33 74 79 70 65 70 77 64 75 73  2195743typepwdus
7bd768cf80  65 72 4e 61 6d 65 66 33 37 62 66 61 31 34 30 35  erNamef37bfa1405
7bd768cf90  37 63 66 30 31 38 30 31 31 64 62 36 37 63 39 36  7cf018011db67c96
7bd768cfa0  33 63 64 37 33 33 5f 31 75 75 69 64 34 63 33 61  3cd733_1********
7bd768cfb0  39 62 33 38 31 38 32 38 66 62 36 33 76 35 2e 34  9b381828fb63v5.4
7bd768cfc0  33 2e 30 0d 0d 0d 0d 0d 0d 0d 0d 0d 0d 0d 0d 0d  3.0.............
7bd768cfd0  6e 54 34 47 5a 30 6f 6e 62 5a 4c 38 34 42 38 38  nT4GZ0onbZL84B88
7bd768cfe0  00 04 6b d7 7b 00 00 00 c0 2d 50 d8 7b 00 00 00  ..k.{....-P.{...
7bd768cff0  00 00 00 00 00 00 00 00 1a 61 70 70 53 74 61 74  .........appStat
[<] 返回值：加密结果长度 = 223
[<] 密文内容:
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7bd768d0c0  75 65 a8 5e 56 d1 dc af 3b 8f 63 76 ec 39 2f e2  ue.^V...;.cv.9/.
7bd768d0d0  e3 8f 52 73 ac 87 4c 6b 27 9b 7e 6a db 22 41 70  ..Rs..Lk'.~j."Ap
7bd768d0e0  be fd d2 0d f0 aa 1f f4 69 b6 c7 59 22 97 b4 bf  ........i..Y"...
7bd768d0f0  54 82 df 10 f8 bb 22 69 46 c6 69 b0 8f af ad 68  T....."iF.i....h
7bd768d100  79 3a 8d 0e 13 a2 0e d7 cc 16 cb 01 3c 1f 03 01  y:..........<...
7bd768d110  5e c2 f8 9a 5f 5f fc 59 2e 09 db bd 64 fd 23 e8  ^...__.Y....d.#.
7bd768d120  71 7c a4 53 8c 27 01 20 e6 fa 41 64 eb 73 b1 3b  q|.S.'. ..Ad.s.;
7bd768d130  29 d7 f4 1d 70 03 8d 9c 4c ec b7 ac 76 77 5b f9  )...p...L...vw[.
7bd768d140  65 d6 00 71 b4 7e 61 99 d1 a9 9d 8a b1 ae 9d 83  e..q.~a.........
7bd768d150  59 5c cc 7c 65 e9 db 8d 3c da fa c8 9d 3e 06 67  Y\.|e...<....>.g
7bd768d160  4a 27 6d 92 fc e0 1f 3c 58 d0 d2 a8 5d ec 8f e4  J'm....<X...]...
7bd768d170  cb 36 84 9d 9f 7d 56 99 21 8f f2 07 55 2f 40 ae  .6...}V.!...U/@.
7bd768d180  00 a0 c5 1f 65 e3 f4 aa db ff 48 cd b0 f8 0d 9c  ....e.....H.....
7bd768d190  6c 00 61 00 6d 00 62 00 64 00 61 00 24 00 32     l.a.m.b.d.a.$.2
======= AES128_ECB_encrypt END =======

```

使用 CyberChef 验证参数和算法
====================

a1 就是要加密的参数，和输出参数是一致的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhH8JHkbibWQ480RiakLcYIFLSj7iaGOgibJpwHCHTKPswSALZnYZ9u2CMT1w/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image11.png

AES128_ECB_encrypt 函数返回值的 hex

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHibTUicF83fjRbJEMxU9iciaJ1dkdNWyg3wVkqKC5z6tAWYXhYMribGrlk1g/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image12.png

使用 AES ECB 加密得到一样的结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHpe8e7CrQp1EibBYjHibPibdevo5dbvicQ6YXv5kfDeyu4yqwzXVXC5ic09w/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image13.png

再通过 base64 编码加密串

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhH4CfNOMcjoib1paQhpJFQz4r0gIlLvFThyO48moicP039zr0wYQTueHFw/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image14.png

编码后的结果与 app 中返回的加密串结尾部分有点不一样

```
// 通过标准 Base64 编码得到加密串
dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+*************************************************g7XzBbLATwfAwFewviaX1/8WS4J271k/SPocXykU4wnASDm+kFk63OxOynX9B1wA42cTOy3rHZ3W/ll1gBxtH5hmdGpnYqxrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

// app 返回的加密串
dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+*************************************************g7XzBbLATwfAwFewviaX1/8WS4J271k/SPocXwnASDm+kFk63OxOynX9B1wA42cTOy3rHZ3W/ll1gBxtH5hmdGpnYqxrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

b64_encode 函数分析
===============

b64_encode 函数原型如下：

```
char *b64_encode(char *a1, __int64 a2)

```

使用 frida hook 一下 b64_encode 函数 并打印参数和返回值：

```
function hook_b64_encode() {
    const soName = "libdewuhelper.so";
    const funcName = "b64_encode";

    const funcAddr = Module.getExportByName(soName, funcName);
    console.log("[+] b64_encode 地址:", funcAddr);

    Interceptor.attach(funcAddr, {
        onEnter(args) {
            this.a1 = args[0];
            this.a2 = args[1].toInt32(); // 转成 JS number
            this.log = "";

            this.log += "\n======= b64_encode =======\n";
            this.log += `[>] 原始数据地址 a1 = ${this.a1}\n`;
            this.log += `[>] 数据长度 a2 = ${this.a2}\n`;

            this.log += "[>] 原始数据内容:\n";
            this.log += hexdump(this.a1, {
                offset: 0,
                length: Math.min(this.a2, 256),
                header: true,
                ansi: false
            }) + "\n";
        },

        onLeave(retval) {
            this.log += `[<] 返回值（Base64字符串地址）= ${retval}\n`;
            const b64Str = Memory.readCString(retval);
            this.log += `[<] Base64 编码结果: ${b64Str}\n`;
            this.log += "======= b64_encode END =======\n";
            console.log(this.log);
        }
    });
}


// Java 调用 native 方法示例
function NCall_IL() {
    Java.perform(() => {
        const Integer = Java.use("java.lang.Integer");
        const String = Java.use("java.lang.String");
        const DuApplication = Java.use("com.shizhuang.duapp.modules.app.DuApplication");
        const NCall = Java.use("lte.NCall");

        const arg0 = Integer.valueOf(283);
        const arg1 = DuApplication.instance.value;
        const arg2 = String.$new("cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0");

        const argsArray = Java.array("java.lang.Object", [arg0, arg1, arg2]);
        const result = NCall.IL(argsArray);
        console.log("NCall.IL 返回值：", result);
    });
}


setImmediate(function () {
    Java.perform(function () {
        hook_b64_encode();
    });
})

// frida -H 127.0.0.1:1234 -F -l b64_encode.js -o log.txt

```

输出如下：

```
[+] b64_encode 地址: 0x7b6280a5c8

======= b64_encode =======
[>] 原始数据地址 a1 = 0x7bd768d440
[>] 数据长度 a2 = 208
[>] 原始数据内容:
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7bd768d440  75 65 a8 5e 56 d1 dc af 3b 8f 63 76 ec 39 2f e2  ue.^V...;.cv.9/.
7bd768d450  e3 8f 52 73 ac 87 4c 6b 27 9b 7e 6a db 22 41 70  ..Rs..Lk'.~j."Ap
7bd768d460  be fd d2 0d f0 aa 1f f4 69 b6 c7 59 22 97 b4 bf  ........i..Y"...
7bd768d470  54 82 df 10 f8 bb 22 69 46 c6 69 b0 8f af ad 68  T....."iF.i....h
7bd768d480  79 3a 8d 0e 13 a2 0e d7 cc 16 cb 01 3c 1f 03 01  y:..........<...
7bd768d490  5e c2 f8 9a 5f 5f fc 59 2e 09 db bd 64 fd 23 e8  ^...__.Y....d.#.
7bd768d4a0  71 7c a4 53 8c 27 01 20 e6 fa 41 64 eb 73 b1 3b  q|.S.'. ..Ad.s.;
7bd768d4b0  29 d7 f4 1d 70 03 8d 9c 4c ec b7 ac 76 77 5b f9  )...p...L...vw[.
7bd768d4c0  65 d6 00 71 b4 7e 61 99 d1 a9 9d 8a b1 ae 9d 83  e..q.~a.........
7bd768d4d0  59 5c cc 7c 65 e9 db 8d 3c da fa c8 9d 3e 06 67  Y\.|e...<....>.g
7bd768d4e0  4a 27 6d 92 fc e0 1f 3c 58 d0 d2 a8 5d ec 8f e4  J'm....<X...]...
7bd768d4f0  cb 36 84 9d 9f 7d 56 99 21 8f f2 07 55 2f 40 ae  .6...}V.!...U/@.
7bd768d500  00 a0 c5 1f 65 e3 f4 aa db ff 48 cd b0 f8 0d 9c  ....e.....H.....
[<] 返回值（Base64字符串地址）= 0x7bd83d1840
[<] Base64 编码结果: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+**********************************************************************************************************************************************xrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==
======= b64_encode END =======

NCall.IL 返回值： dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+**********************************************************************************************************************************************xrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==

```

所以加密数据的实际长度是 208，并不是 223。

把 hexdump 复制到 CyberChef 使用标准 base64 编码结果 和 NCall.IL 返回值是一样的，也就是说 b64_encode 就是一个标准的 base64 编码方法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHZPtGPOMGvP7IiahvE48hwsG29z8rZ2ylibibl4iaHLucSz4Z4aTFpMOKPg/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image15.png

使用 CyberChef 验证算法
=================

所以 encode 方法的算法逻辑是：AES ECB 加密 + 标准 Base64 编码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jmOEtISbGxA2fVDScoqNrhHGlPdEaUGjzUun9cykAwLZA0hl9wsmBJOleicib4AMPiaBSUbSiciaYON6xA/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image16.png

  
对比 NCall.IL 方法的返回值是一致的。

使用 Python 还原 newSign 算法
=======================

下面是使用 Python 实现的完整加密流程，包括：

*   • aes_ecb_encrypt(plaintext, key)：AES ECB 模式加密（PKCS7 padding）
    
*   • base64_encode(data)：标准 Base64 编码
    
*   • md5_hash(data)：MD5 哈希
    
*   • newSign(text, key)：整合上面函数：先 AES-ECB 加密，再 base64 编码，最后 md5 哈希
    

安装依赖（如未安装）：

```
pip install pycryptodome

```

代码实现如下：

```
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import base64
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

def base64_encode(data: bytes) -> str:
    encoded = base64.b64encode(data).decode('utf-8')
    print(f"[Base64] 编码结果: {encoded}")
    return encoded

def md5_hash(data: str) -> str:
    md5_result = hashlib.md5(data.encode('utf-8')).hexdigest()
    print(f"[MD5] Hash 结果: {md5_result}")
    return md5_result

def newSign(text: str, key: str) -> str:
    print("\n======= newSign 开始 =======")
    encrypted = aes_ecb_encrypt(text, key)
    b64 = base64_encode(encrypted)
    md5_result = md5_hash(b64)
    print("======= newSign 结束 =======\n")
    return md5_result


# 示例调用
if __name__ == "__main__":
    text = "cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0"
    key = "****************"  # 16字节 AES 密钥
    result = newSign(text, key)
    print("newSign 结果:", result)

```

运行输出如下：

```
======= newSign 开始 =======
[AES] 原文: cipherParamuserNamecountryCode86loginTokenpassword6716c*******************************************************42195743typepwduserNamef37bfa14057cf018011db67c963cd733_1********9b381828fb63v5.43.0
[AES] 密钥: ****************
[AES] 加密结果（Hex）: 7565a85e56d1dcaf3b8f6376ec392fe2e38f5273ac874c6b279b7e6adb224170befdd20df0aa1ff469b6c7592297b4bf5482df10f8bb226946c669b08fafad68793a8d0e13******************************************************************************************************************************************8ab1ae9d83595ccc7c65e9db8d3cdafac89d3e06674a276d92fce01f3c58d0d2a85dec8fe4cb36849d9f7d5699218ff207552f40ae00a0c51f65e3f4aadbff48cdb0f80d9c
[Base64] 编码结果: dWWoXlbR3K87j2N27Dkv4uOPUnOsh0xrJ5t+atsiQXC+**********************************************************************************************************************************************xrp2DWVzMfGXp24082vrInT4GZ0onbZL84B88WNDSqF3sj+TLNoSdn31WmSGP8gdVL0CuAKDFH2Xj9Krb/0jNsPgNnA==
[MD5] Hash 结果: 92d2d46c07**********c281ccaa4c
======= newSign 结束 =======

newSign 结果: 92d2d46c07**********c281ccaa4c

```

#### 引用链接

`[1]` 逆向某物 App 登录接口：抓包分析 + Frida Hook 还原加密算法:_https://cyrus-studio.github.io/blog/posts/%E9%80%86%E5%90%91%E6%9F%90%E7%89%A9-app-%E7%99%BB%E5%BD%95%E6%8E%A5%E5%8F%A3%E6%8A%93%E5%8C%85%E5%88%86%E6%9E%90-+-frida-hook-%E8%BF%98%E5%8E%9F%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95/_  
`[2]`ART 下 Dex 加载流程源码分析 和 通用脱壳点:_https://cyrus-studio.github.io/blog/posts/art-%E4%B8%8B-dex-%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%92%8C-%E9%80%9A%E7%94%A8%E8%84%B1%E5%A3%B3%E7%82%B9/_  
`[3]`FART 脱壳某大厂 App + CodeItem 修复 dex + 反编译还原源码:_https://cyrus-studio.github.io/blog/posts/fart-%E8%84%B1%E5%A3%B3%E6%9F%90%E5%A4%A7%E5%8E%82-app-+-codeitem-%E4%BF%AE%E5%A4%8D-dex-+-%E5%8F%8D%E7%BC%96%E8%AF%91%E8%BF%98%E5%8E%9F%E6%BA%90%E7%A0%81/_  
`[4]`frida_dump:_https://github.com/lasting-yang/frida_dump_  
`[5]`一文搞懂 SO 脱壳全流程：识别加壳、Frida Dump、原理深入解析:_https://cyrus-studio.github.io/blog/posts/%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82-so-%E8%84%B1%E5%A3%B3%E5%85%A8%E6%B5%81%E7%A8%8B%E8%AF%86%E5%88%AB%E5%8A%A0%E5%A3%B3frida-dump%E5%8E%9F%E7%90%86%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/_  
`[6]`逆向 JNI 函数找不到入口？动态注册定位技巧全解析:_https://cyrus-studio.github.io/blog/posts/%E9%80%86%E5%90%91-jni-%E5%87%BD%E6%95%B0%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%A5%E5%8F%A3%E5%8A%A8%E6%80%81%E6%B3%A8%E5%86%8C%E5%AE%9A%E4%BD%8D%E6%8A%80%E5%B7%A7%E5%85%A8%E8%A7%A3%E6%9E%90/_  
`[7]`移植 OLLVM 到 LLVM 18，C&C++ 代码混淆:_https://cyrus-studio.github.io/blog/posts/%E7%A7%BB%E6%A4%8D-ollvm-%E5%88%B0-llvm-18cc++%E4%BB%A3%E7%A0%81%E6%B7%B7%E6%B7%86/_  
`[8]`移植 OLLVM 到 Android NDK，Android Studio 中使用 OLLVM:_https://cyrus-studio.github.io/blog/posts/%E7%A7%BB%E6%A4%8D-ollvm-%E5%88%B0-android-ndkandroid-studio-%E4%B8%AD%E4%BD%BF%E7%94%A8-ollvm/_  
`[9]`OLLVM 增加 C&C++ 字符串加密功能:_https://cyrus-studio.github.io/blog/posts/ollvm-%E5%A2%9E%E5%8A%A0-cc++-%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8A%A0%E5%AF%86%E5%8A%9F%E8%83%BD/_  
`[10]`破解 VMP+OLLVM 混淆：通过 Hook jstring 快速定位加密算法入口:_https://cyrus-studio.github.io/blog/posts/%E7%A0%B4%E8%A7%A3-vmp+ollvm-%E6%B7%B7%E6%B7%86%E9%80%9A%E8%BF%87-hook-jstring-%E5%BF%AB%E9%80%9F%E5%AE%9A%E4%BD%8D%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E5%85%A5%E5%8F%A3/_