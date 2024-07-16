> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GisxR8W3Ge_7Xrlrwxf9yw)

前言
--

又有段时间没更新了，没啥大事，其实就是道心有点不稳了，所以考虑很多，焦虑，紧张，难受，每天回家就躺着，啥也不想干，原因大家都懂。

另外也遇到些技术以外的糟心事......

唉.....，不提了都过去了，以后还是不定期更新吧

相信各位佬，其实逆向调试多了，自会留下点什么复用的，所以我就把我用到的整理下，能分享的，就发出来吧，不能分享的就留着了。

所以以下操作，对大佬来说，可以忽略，相信大佬们都会

同时，以下的整理我也不敢说有多完整，至少常规的操作基本是够用了。

另外前两天 junge 大佬才发了算法助手 plus 版，支持安卓 14，应该也有一些功能重合，不过不重要，看个人喜好选择吧，能用，能帮到他人就行

**不过以下所有涉及的相关操作，都建立在你能过 root，能打开 app，能过反调试的情况下，才能做的对应操作**

常规操作总结
------

### 1. 合并 dex

由于安卓 app 的 dex 文件的 MultiDex 技术，一个 dex 不能太大，也就是俗称的 “64K 引用限制”。谷歌官方介绍

> https://developer.android.google.cn/studio/build/multidex?hl=zh-cn

所以会导致很多时候，一个 app 的 dex 有多个，比如如下：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVr5lqYiaBr1yI75ibVYajZ0qxg6LnwBaXkbdex6zNDOJvbibFw4uS3pcNsyHEyK0ujceQ5q4Cf4f8AA/640?wx_fmt=png)

而这种 dex，对于逆向分析来说，如果是 jadx，选中全部 dex 拖进 jadx 就完了，不过由于 jadx 有时候反编译效果不太好，所以我们会选用 jeb 或者 GDA 来分析，但是 jeb 和 GDA 又只能分析单个 dex。

所以，合并 dex 的需求就来了。由于 jadx 可以通过多个 dex 进行反编译

首先，配置环境变量，windows:

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVr5lqYiaBr1yI75ibVYajZ0qd02BjeibtvjEcHZ0t5mib0Lb62iamv1mJIQDicAzDViapaPteuXyYPWbsicw/640?wx_fmt=png)

*nix，就是用 ln -s 建立软链接就行。具体的自行百度  

方法一：  

不是太推荐这个方法，原因看下面  

用如下合并代码运行即可：

两个代码都可以：

```
import os, sys
    path = r'E:\BaiduNetdiskDownload\meituan'  # 文件夹目录
    files = os.listdir(path)  # 得到文件夹下的所有文件名称
    out_path = r'./'  # 输出文件夹
    # 路径上不要有中文!!!!!
    s = []
    for file in files:  # 遍历文件夹
        if file.find("dex") > 0:  ## 查找dex 文件
            sh = f'jadx -j 1 -r -d {out_path} {path}\\{file}'
            print(sh)
            os.system(sh)

```

```
    if len(sys.argv) < 3:
        print("start error")
        sys.exit()
    print(sys.argv[1], sys.argv[2])
    path = sys.argv[1]  # 文件夹目录
    files = os.listdir(path)  # 得到文件夹下的所有文件名称
    s = []
    for file in files:  # 遍历文件夹
        if file.find("dex") > 0:  ## 查找dex 文件
            sh = 'jadx -j 1 -r -s -d ' + sys.argv[2] + " " + path + file
            print(sh)
            os.system(sh)

```

执行之后：

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVr5lqYiaBr1yI75ibVYajZ0qYlpulialnECEXDfAa5JuWReuNpicf0nVoHTNCkvrckJE2f0OosgQNo1Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVr5lqYiaBr1yI75ibVYajZ0qFMoB8V4bIZ2K0y0XVjwea9K1PGvkjvdZU0mfGIcFfSV72CAZzArsrg/640?wx_fmt=png)

跟我们预期的好像不太一样，这就很尴尬，它直接反编译成文件夹了...  

所以上面我不推荐这个方法

方法二（推荐）：

如果是脱壳出来的 dex，文件肯定不是 classes 之类，你需要把文件该成 classes.dex classes2.dex classes3.dex 之类的，然后去 apk 里把 meta-inf 拿到，重新压缩为 zip，改后缀为 apk，这样 jeb 和 GDA 就可以分析了，拖进去就完了。

方法三：  

np 管理器 / mt 管理器的 dex 合并：  

> githubXiaowangzi/NP-Manager

![](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXVr5lqYiaBr1yI75ibVYajZ0q2G3mTiararic2wdBchg1Uiaiam9qxdmXQvVvwp6E67AqQ8xDvUCcM24xcg/640?wx_fmt=png)

### 2.dex 转 apk

这个有时候也有需求，用于将脱壳下来的众多 dex 重新组装成 apk  

```
import os
import zipfile
import argparse
def rename_class(path):
    files = os.listdir(path)
    dex_index = 0
    if path.endswith('/'):
        path = path[:-1]
        print(path)
    for i in range(len(files)):
        if files[i].endswith('.dex'):
            old_name = path + '/' + files[i]
            if dex_index == 0:
                new_name = path + '/' + 'classes.dex'
            else:
                new_name = path + '/' + 'classes%d.dex' % dex_index
            dex_index += 1
            if os.path.exists(new_name):
                continue
            os.rename(old_name, new_name)
    print('[*] 重命名完毕')
def extract_META_INF_from_apk(apk_path, target_path):
    r = zipfile.is_zipfile(apk_path)
    if r:
        fz = zipfile.ZipFile(apk_path, 'r')
        for file in fz.namelist():
            if file.startswith('META-INF'):
                fz.extract(file, target_path)
    else:
        print('[-] %s 不是一个APK文件' % apk_path)
def zip_dir(dirname, zipfilename):
    filelist = []
    if os.path.isfile(dirname):
        if dirname.endswith('.dex'):
            filelist.append(dirname)
    else:
        for root, dirs, files in os.walk(dirname):
            for dir in dirs:
                # if dir == 'META-INF':
                # print('dir:', os.path.join(root, dir))
                filelist.append(os.path.join(root, dir))
            for name in files:
                # print('file:', os.path.join(root, name))
                filelist.append(os.path.join(root, name))
    z = zipfile.ZipFile(zipfilename, 'w', zipfile.ZIP_DEFLATED)
    for tar in filelist:
        arcname = tar[len(dirname):]
        if ('META-INF' in arcname or arcname.endswith('.dex')) and '.DS_Store' not in arcname:
            # print(tar + " -->rar: " + arcname)
            z.write(tar, arcname)
    print('[*] APK打包成功，你可以拖入APK进行分析啦！'  )
    z.close()
def parse_args():
    parser = argparse.ArgumentParser(description='repackage dumped dex to apk for jeb/jadx analysis.')
    parser.add_argument('-a', '--apk_path', required=True, type=str,
                        help='apk for extracting META-INF')
    parser.add_argument('-i', '--dex_path', required=True, type=str,
                        help='path of dumped dex')
    parser.add_argument('-o', '--output', type=str, default="repacked.apk",
                        help='apk path after zip')
    args = parser.parse_args()
    #print(args.apk_path)
    return args
if __name__ == '__main__':
    args = parse_args()
    rename_class(args.dex_path)
    extract_META_INF_from_apk(args.apk_path, args.dex_path)
    # zip_file(args.dex_path)
    zip_dir(args.dex_path, args.output)

```

### 2. 算法助手快速定位加密位置

这个是 xposed 插件，所以需要依赖 root+xposed 环境，这个工具在之前的文章里已经多次出现，具体操作就不再演示了

### 3.np 管理器

这个不用多言，你不妨试试，也许比你手动分析快很多  

> githubXiaowangzi/NP-Manager

### 4.mt 管理器

这个跟 np 管理器很类似，很多功能基本一样，但是也有不一样的，同样的，你可以尝试用用，感受一下。  

> 下载 | MT 管理器 (mt2.cn)

### 5. 抓包

这个其实在之前的文章有介绍 [移动安全逆向分析流程](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484968&idx=1&sn=5dfc98c11270394b85e085cfc94590ff&chksm=fb18f78acc6f7e9cb23c077424e009b52738ebf4beb9123f5fdfec9ba812e5487b2ecfad5873&scene=21#wechat_redirect)

所以这里就不再展示，只是再加一个脸哥的透明代理抓包

https://blog.seeflower.dev/archives/207/

https://blog.seeflower.dev/archives/210/

### 6. 脱壳

这个比较敏感，毕竟你懂的，所以这里就不展示了。感兴趣私聊我即可。  

### 7. 弹窗拦截

什么时候会有强制弹窗  

最常见的弹窗就以下几种

> 强制升级  
> 
> 某些授权登录
> 
> 广告

这个我也记得在以前的文章好像也说过。可以利用算法助手里的弹窗拦截功能拦截，mt 管理器也有 activity 记录，然后可以定位是哪里的弹窗，对其进行 hook 设置为 false，即可解除弹窗。

frida 相关
--------

### 1.Java.use 和 Java.choose 的区别

这个很早的文章有提过，Java.use 就是通过拿到类，然后 hook 类的某个指定的方法。

Java.choose 就是在内存已加载的类里找需要的类，然后 hook 执行的前后  

### 2. 启动 hook 脚本失败或者一段时间退出

这种情况大概率是有 frida 对抗，如果是该 app 的壳自带的对 frida 对阿对抗，可以用 setTimeout(main,2000)。如果是有更强的 frida 对抗。那么就得针对性得去分析了。

### 3.hook context 上下文（寄存器）

```
Java.perform(function () {
     var ActivityThread = Java.use('android.app.ActivityThread')
     var currentActivityThread = null
     Java.choose('android.app.ActivityThread', {
       onMatch: function (ins) {
         console.log('found instance', ins)
         currentActivityThread = ins
       }, onComplete: function () {
       }
     })
     // var currentActivityThread =  ActivityThread.currentActivityThread()
     // var currentActivityThread_obj = Java.cast(currentActivityThread,ActivityThread)
     var application = currentActivityThread.getApplication()
     // var Context = Java.use('android.content.Context')
     // var context_obj = Java.cast(application,Context)
     console.log('context is ',application)
     return application;
   })

```

### 4.hook constructor 构造方法

```
function hook_constructor() {
    if (Process.pointerSize == 4) {
        var linker = Process.findModuleByName("linker");
    } else {
        var linker = Process.findModuleByName("linker64");
    }
    var addr_call_function =null;
    var addr_g_ld_debug_verbosity = null;
    var addr_async_safe_format_log = null;
    if (linker) {
        var symbols = linker.enumerateSymbols();
        for (var i = 0; i < symbols.length; i++) {
            var name = symbols[i].name;
            if (name.indexOf("call_function") >= 0){
                addr_call_function = symbols[i].address;
            }
            else if(name.indexOf("g_ld_debug_verbosity") >=0){
                addr_g_ld_debug_verbosity = symbols[i].address;
                ptr(addr_g_ld_debug_verbosity).writeInt(2);
            } else if(name.indexOf("async_safe_format_log") >=0 && name.indexOf('va_list') < 0){
                addr_async_safe_format_log = symbols[i].address;
            } 
        }
    }
    if(addr_async_safe_format_log){
        Interceptor.attach(addr_async_safe_format_log,{
            onEnter: function(args){
                this.log_level  = args[0];
                this.tag = ptr(args[1]).readCString()
                this.fmt = ptr(args[2]).readCString()
                if(this.fmt.indexOf("c-tor") >= 0 && this.fmt.indexOf('Done') < 0){
                    this.function_type = ptr(args[3]).readCString(), // func_type
                    this.so_path = ptr(args[5]).readCString();
                    var strs = new Array(); //定义一数组 
                    strs = this.so_path.split("/"); //字符分割
                    this.so_name = strs.pop();
                    this.func_offset  = ptr(args[4]).sub(Module.findBaseAddress(this.so_name)) 
                     console.log("func_type:", this.function_type,
                        '\nso_name:',this.so_name,
                        '\nso_path:',this.so_path,
                        '\nfunc_offset:',this.func_offset 
                     );
                }
            },
            onLeave: function(retval){
            }
        })
    }
}
function main() {
    hook_constructor();
}
setImmediate(main);

```

### 5.hook classloader 类加载器

```
Java.perform(function () {
        var target = Java.registerClass({
            name: 'com.xxx.xxx.xxxx.xx.RegisterClass',
            // implements: [Six],
            // superClass: Java.use('java.lang.Object'),
            methods: {
                $init: function () {
                    console.log('Constructor called');
                },
                next: {
                    returnType: 'java.lang.Boolean',
                    // returnType: 'java.lang.Boolean',
                    //   argumentTypes: [],
                    argumentTypes: [],
                    implementation: function () {
                        console.log('call next')
                        return Java.use('java.lang.Boolean').$new('true');
                        // return Java.use('java.lang.Object').$new('true');
                        // return true;
                    },
                },
            }
        })
        var Six = Java.use('com.xxx.xxxx.xxx.xxx.xxx')
        var pathLoader = Six.class.getClassLoader()
        var dexLoader = target.class.getClassLoader()
        console.log('activtity loader->',pathLoader)
        var parent = pathLoader.parent.value
        pathLoader.parent.value = dexLoader
        dexLoader.parent.value = parent
    })

```

### 6. 打印调用栈

```
//第一种
function printStack(name) {
    Java.perform(function () {
        var Exception = Java.use("java.lang.Exception");
        var ins = Exception.$new("Exception");
        var straces = ins.getStackTrace();
        if (straces != undefined && straces != null) {
            var strace = straces.toString();
            var replaceStr = strace.replace(/,/g, "\\n");
            console.log("=============================" + name + " Stack strat=======================");
            console.log(replaceStr);
            console.log("=============================" + name + " Stack end=======================\r\n");
            Exception.$dispose();
        }
    });
}
//第二种
function showStacks() {
    Java.perform(function () {
        console.log(
            Java.use("android.util.Log")
                .getStackTraceString(
                    Java.use("java.lang.Throwable").$new()
                )
        );
    })
}

```

### 7.dump dex

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
function dump_dex() {
    var libart = Process.findModuleByName("libart.so");
    var addr_DefineClass = null;
    var symbols = libart.enumerateSymbols();
    for (var index = 0; index < symbols.length; index++) {
        var symbol = symbols[index];
        var symbol_name = symbol.name;
        //这个DefineClass的函数签名是Android9的
        //_ZN3art11ClassLinker11DefineClassEPNS_6ThreadEPKcmNS_6HandleINS_6mirror11ClassLoaderEEERKNS_7DexFileERKNS9_8ClassDefE
        if (symbol_name.indexOf("ClassLinker") >= 0 &&
            symbol_name.indexOf("DefineClass") >= 0 &&
            symbol_name.indexOf("Thread") >= 0 &&
            symbol_name.indexOf("DexFile") >= 0) {
            console.log(symbol_name, symbol.address);
            addr_DefineClass = symbol.address;
        }
    }
    var dex_maps = {};
    var dex_count = 1;
    console.log("[DefineClass:]", addr_DefineClass);
    if (addr_DefineClass) {
        Interceptor.attach(addr_DefineClass, {
            onEnter: function(args) {
                var dex_file = args[5];
                //ptr(dex_file).add(Process.pointerSize) is "const uint8_t* const begin_;"
                //ptr(dex_file).add(Process.pointerSize + Process.pointerSize) is "const size_t size_;"
                var base = ptr(dex_file).add(Process.pointerSize).readPointer();
                var size = ptr(dex_file).add(Process.pointerSize + Process.pointerSize).readUInt();
                if (dex_maps[base] == undefined) {
                    dex_maps[base] = size;
                    var magic = ptr(base).readCString();
                    if (magic.indexOf("dex") == 0) {
                        var process_name = get_self_process_name();
                        if (process_name != "-1") {
                            var dex_dir_path = "/data/data/" + process_name + "/files/dump_dex_" + process_name;
                            mkdir(dex_dir_path);
                            var dex_path = dex_dir_path + "/class" + (dex_count == 1 ? "" : dex_count) + ".dex";
                            console.log("[find dex]:", dex_path);
                            var fd = new File(dex_path, "wb");
                            if (fd && fd != null) {
                                dex_count++;
                                var dex_buffer = ptr(base).readByteArray(size);
                                fd.write(dex_buffer);
                                fd.flush();
                                fd.close();
                                console.log("[dump dex]:", dex_path);
                            }
                        }
                    }
                }
            },
            onLeave: function(retval) {}
        });
    }
}
var is_hook_libart = false;
function hook_dlopen() {
    Interceptor.attach(Module.findExportByName(null, "dlopen"), {
        onEnter: function(args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                //console.log("dlopen:", path);
                if (path.indexOf("libart.so") >= 0) {
                    this.can_hook_libart = true;
                    console.log("[dlopen:]", path);
                }
            }
        },
        onLeave: function(retval) {
            if (this.can_hook_libart && !is_hook_libart) {
                dump_dex();
                is_hook_libart = true;
            }
        }
    })
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
        onEnter: function(args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                //console.log("android_dlopen_ext:", path);
                if (path.indexOf("libart.so") >= 0) {
                    this.can_hook_libart = true;
                    console.log("[android_dlopen_ext:]", path);
                }
            }
        },
        onLeave: function(retval) {
            if (this.can_hook_libart && !is_hook_libart) {
                dump_dex();
                is_hook_libart = true;
            }
        }
    });
}
setImmediate(dump_dex);

```

### 8.hook log 日志

```
function hook_log() {
    const Log = Java.use('android.util.Log');
    Log.d.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log d', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
    Log.v.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log v', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
    Log.i.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log i', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
    Log.w.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log w', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
    Log.e.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log e', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
    Log.wtf.overload('java.lang.String', 'java.lang.String')
        .implementation = function (tag, content) {
        DMLog.i('Log wtf', 'tag: ' + tag + ', content: ' + content);
        return 0;
    };
}

```

### 9.hook event 事件

这个就是监听点击操作，找到相关的逻辑，进而跟栈找目标位置的  

```
var jclazz = null;
var jobj = null;
function getObjClassName(obj) {
    if (!jclazz) {
        var jclazz = Java.use("java.lang.Class");
    }
    if (!jobj) {
        var jobj = Java.use("java.lang.Object");
    }
    return jclazz.getName.call(jobj.getClass.call(obj));
}
function watch(obj, mtdName) {
    var listener_name = getObjClassName(obj);
    var target = Java.use(listener_name);
    if (!target || !mtdName in target) {
        return;
    }
    // send("[WatchEvent] hooking " + mtdName + ": " + listener_name);
    target[mtdName].overloads.forEach(function (overload) {
        overload.implementation = function () {
            //send("[WatchEvent] " + mtdName + ": " + getObjClassName(this));
            console.log("[WatchEvent] " + mtdName + ": " + getObjClassName(this))
            return this[mtdName].apply(this, arguments);
        };
    })
}
function OnClickListener() {
    Java.perform(function () {
        //以spawn启动进程的模式来attach的话
        Java.use("android.view.View").setOnClickListener.implementation = function (listener) {
            if (listener != null) {
                watch(listener, 'onClick');
            }
            return this.setOnClickListener(listener);
        };
        //如果frida以attach的模式进行attch的话
        Java.choose("android.view.View$ListenerInfo", {
            onMatch: function (instance) {
                instance = instance.mOnClickListener.value;
                if (instance) {
                    console.log("mOnClickListener name is :" + getObjClassName(instance));
                    watch(instance, 'onClick');
                }
            },
            onComplete: function () {
            }
        })
    })
}
setImmediate(OnClickListener);

```

### 10.hook 网络类，过掉 vpn 检测

第一种

```
function main(){
    Java.perform(function (){
        Java.use("android.net.NetworkInfo").isConnected.implementation = function(){
            console.log("first called!")
            return false
        }
        Java.use("java.net.NetworkInterface").getName.implementation = function(){
            console.log("second called!")
            return ""
            // return null
        }
        Java.use("android.net.NetworkCapabilities").hasTransport.implementation=function(){
            console.log("third called!")
            return false
        }
        //android.net.ConnectivityManager.getNetworkCapabilities
    })
}
// setImmediate(main)
setTimeout(main, 2000);

```

第二种  

```
function main() {
    Java.perform(function () {
        var NetworkInfo = Java.use("android.net.NetworkInfo")
        NetworkInfo.isConnected.implementation = function(){
            console.log("first called!")
            var result = this.isConnected()
            console.log('result',result)
            return false
            // return result
        }
        var network = Java.use('java.net.NetworkInterface')
        network.getName.implementation = function () {
            console.log("second called!")
            var name = this.getName()
            console.log('name:' + name)
            if (name == "tun0") {
                // var result = Java.use('java.lang.String').$new('rmnet_data0')
                var result = Java.use('java.lang.String').$new('ccmni1')
                console.log('hook result:' + result)
                // return result
                return ''
            }
            // else if (name == "wlan0") {
            //     // var result = Java.use('java.lang.String').$new('rmnet_data0')
            //     var result = Java.use('java.lang.String').$new('ccmni2')
            //     console.log('hook result:' + result)
            //     return result
            // }
            else {
                return name
            }
        }
        var NetworkCapabilities = Java.use("android.net.NetworkCapabilities")
        NetworkCapabilities.hasTransport.overload('int').implementation = function (arg) {
            console.log("third called!",arg)
            var result = this.hasTransport(arg)
            console.log('result',result)
            // return result
            return false
        }
        var ConnectivityManager = Java.use("android.net.ConnectivityManager")
        ConnectivityManager.getNetworkCapabilities.implementation = function (arg) {
            console.log("four called!",arg)
            var result = this.getNetworkCapabilities(arg)
            console.log('result',result)
            return result
            // return false
        }
        // android.net.ConnectivityManager.getNetworkCapabilities
    })
}
setTimeout(main, 2000);

```

### 11.hook 隐私合规相关 api 调用

```
var line_flag = 0 // 为避免异步输出可能导致的数据混乱，加一个标号
function get_tmp_index() {
    line_flag = line_flag + 1;
    return line_flag
}
function log_with_index(index, data) {
    console.log(index + "|" + data)
}
function showStacks(tmp_index) {
    var Exception = Java.use("java.lang.Exception");
    var ins = Exception.$new("Exception");
    var straces = ins.getStackTrace();
    if (undefined == straces || null == straces) {
        return;
    }
    log_with_index(tmp_index, "============================= Stack strat=======================");
    log_with_index(tmp_index, "");
    for (var i = 0; i < straces.length; i++) {
        var str = "   " + straces[i].toString();
        log_with_index(tmp_index, str);
    }
    log_with_index(tmp_index, "");
    log_with_index(tmp_index, "============================= Stack end=======================\r\n");
    Exception.$dispose();
}
// 获取ro.serialno信息
function hookGetSystemInfo() {
    try {
        var SP = Java.use("android.os.SystemProperties");
        if (SP.get != undefined) {
            SP.get.overload('java.lang.String').implementation = function (p1) {
                if (p1.indexOf("ro.serialno") < 0) return this.get(p1);
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                var temp = this.get(p1);
                log_with_index(tmp_index, "[*]" + p1 + " : " + temp);
                return temp;
            }
            SP.get.overload('java.lang.String', 'java.lang.String').implementation = function (p1, p2) {
                if (p1.indexOf("ro.serialno") < 0) return this.get(p1, p2);
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                var temp = this.get(p1, p2)
                log_with_index(tmp_index, "[*]" + p1 + "," + p2 + " : " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetSystemInfo-android.os.SystemProperties failed. reason:" + e)
    }
}
// 获取应用请求权限信息
function hookRequestPermission() {
    try {
        var AC = Java.use("android.support.v4.app.ActivityCompat")
        if (AC.requestPermissions != undefined) {
            AC.requestPermissions.overload('android.app.Activity', '[Ljava.lang.String;', 'int').implementation = function (p1, p2, p3) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - requestPermissions=======================\r\n");
                var temp = this.requestPermissions(p1, p2, p3);
                log_with_index(tmp_index, "requestPermissions: " + p2);
                return temp
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookRequestPermission-android.support.v4.app.ActivityCompat failed. reason:" + e)
    }
    try {
        var AC2 = Java.use("androidx.core.app.ActivityCompat")
        if (AC2.requestPermissions != undefined) {
            AC2.requestPermissions.overload('android.app.Activity', '[Ljava.lang.String;', 'int').implementation = function (p1, p2, p3) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - requestPermissions=======================\r\n");
                var temp = this.requestPermissions(p1, p2, p3);
                log_with_index(tmp_index, "requestPermissions: " + p1 + p2 + p3);
                return temp
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookRequestPermission-androidx.core.app.ActivityCompat failed. reason:" + e)
    }
    try {
        var AC3 = Java.use("android.app.Activity")
        if (AC3.requestPermissions != undefined) {
            AC3.requestPermissions.overload('[Ljava.lang.String;', 'int').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - requestPermissions=======================\r\n");
                var temp = this.requestPermissions(p1, p2);
                log_with_index(tmp_index, "requestPermissions: " + p1 + p2);
                return temp
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookRequestPermission-android.app.Activity failed. reason:" + e)
    }
}
// 获取AndroidId信息
function hookGetAndroidId() {
    try {
        var Secure = Java.use("android.provider.Settings$Secure");
        if (Secure.getString != undefined) {
            Secure.getString.implementation = function (p1, p2) {
                if (p2.indexOf("android_id") < 0) return this.getString(p1, p2);
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - get android_ID=======================param is" + p2 + "\r\n");
                var temp = this.getString(p1, p2);
                log_with_index(tmp_index, "get android_ID: " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetAndroidId-android.provider.Settings$Secure failed. reason:" + e)
    }
}
// 获取IMSI信息
function hookGetIMSI() {
    try {
        var TelephonyManager = Java.use("android.telephony.TelephonyManager");
        if (TelephonyManager.getSimSerialNumber != undefined) {
            // 获取单个IMSI的方法
            TelephonyManager.getSimSerialNumber.overload().implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - getSimSerialNumber(String)=======================\r\n");
                var temp = this.getSimSerialNumber();
                log_with_index(tmp_index, "getSimSerialNumber(String): " + temp);
                return temp;
            };
            // 应该也是获取IMSI的方法
            TelephonyManager.getSubscriberId.overload('int').implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - getSubscriberId(int)=======================\r\n");
                var temp = this.getSubscriberId();
                log_with_index(tmp_index, "getSubscriberId(int): " + temp);
                return temp;
            }
            // 获取多个IMSI的方法
            TelephonyManager.getSimSerialNumber.overload('int').implementation = function (p) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - getSimSerialNumber(int)==============param is" + p + "\r\n");
                var temp = this.getSimSerialNumber(p);
                log_with_index(tmp_index, "getSimSerialNumber(int) " + temp);
                return temp;
            };
            TelephonyManager.getLine1Number.overload('int').implementation = function (p) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "=============================[*]Called - getLine1Number(int)==============param is" + p + "\r\n");
                var temp = this.getLine1Number(p);
                log_with_index(tmp_index, "getLine1Number(int) " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIMSI-android.telephony.TelephonyManager failed. reason:" + e)
    }
}
// 获取IMEI信息
function hookGetIMEI() {
    try {
        var TelephonyManager = Java.use("android.telephony.TelephonyManager");
        if (TelephonyManager.getDeviceId != undefined) {
            // getDeviceId was deprecated in API level 26
            //获取单个IMEI
            TelephonyManager.getDeviceId.overload().implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getDeviceId()=======================\r\n");
                log_with_index(tmp_index, "getDeviceId: " + 'Dawnnnnnn');
                // var temp = this.getDeviceId();
                // log_with_index(tmp_index, "getDeviceId: " + temp);
                // 这里可能因为API LEVEL的关系导致调用getdeviceId时应用闪退，那就不调用了
                return 'Dawnnnnnn';
            };
            //获取多个IMEI的方法
            TelephonyManager.getDeviceId.overload('int').implementation = function (p) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getDeviceId()=======================param is" + p + "\r\n");
                var temp = this.getDeviceId(p);
                log_with_index(tmp_index, "getDeviceId " + p + ": " + temp);
                return temp;
            };
            //API LEVEL26以上的获取单个IMEI方法
            TelephonyManager.getImei.overload().implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getImei()=======================\r\n");
                var temp = this.getImei();
                log_with_index(tmp_index, "getImei: " + temp);
                return temp;
            };
            // API LEVEL26以上的获取多个IMEI方法
            TelephonyManager.getImei.overload('int').implementation = function (p) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getImei()====================param is" + p + "\r\n");
                var temp = this.getImei(p);
                log_with_index(tmp_index, "getImei: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIMEI-android.telephony.TelephonyManager failed. reason:" + e)
    }
}
// 获取MEID信息
function hookGetMEID() {
    try {
        var TelephonyManager = Java.use("android.telephony.TelephonyManager");
        if (TelephonyManager.getMeid != undefined) {
            TelephonyManager.getMeid.overload().implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getMeid()=======================\r\n");
                var temp = this.getMeid();
                log_with_index(tmp_index, "getDeviceId: " + temp);
                return temp;
            };
            TelephonyManager.getMeid.overload('int').implementation = function (p) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getMeid()=======================param is" + p + "\r\n");
                var temp = this.getMeid(p);
                log_with_index(tmp_index, "getMeid " + p + ": " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetMEID-android.telephony.TelephonyManager failed. reason:" + e)
    }
}
// 获取Mac地址信息
function hookGetMacAddress() {
    try {
        var wifiInfo = Java.use("android.net.wifi.WifiInfo");
        if (wifiInfo.getMacAddress != undefined) {
            wifiInfo.getMacAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getMacAddress()=======================\r\n");
                var temp = this.getMacAddress();
                log_with_index(tmp_index, "getMacAddress: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetMacAddress-android.net.wifi.WifiInfo failed. reason:" + e)
    }
    try {
        var networkInterface = Java.use("java.net.NetworkInterface");
        if (networkInterface.getHardwareAddress != undefined) {
            networkInterface.getHardwareAddress.overload().implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHardwareAddress()=======================\r\n");
                var temp = this.getHardwareAddress();
                log_with_index(tmp_index, "getHardwareAddress: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetMacAddress-java.net.NetworkInterface failed. reason:" + e)
    }
}
// 获取IP地址信息
function hookGetIPAddress() {
    // This method was deprecated in API level 31.
    try {
        var wifiInfo = Java.use("android.net.wifi.WifiInfo");
        if (wifiInfo.getIpAddress != undefined) {
            wifiInfo.getIpAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getIpAddress()=======================\r\n");
                var temp = this.getIpAddress();
                log_with_index(tmp_index, "getIpAddress: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIPAddress-android.net.wifi.WifiInfo failed. reason:" + e)
    }
}
// 获取RunningAppProcesses信息
function hookGetRunningAppProcesses() {
    try {
        var AM = Java.use("android.app.ActivityManager");
        if (AM.getRunningAppProcesses != undefined) {
            AM.getRunningAppProcesses.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getRunningAppProcesses()=======================\r\n");
                var temp = this.getRunningAppProcesses();
                log_with_index(tmp_index, "getRunningAppProcesses: " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetRunningAppProcesses-android.app.ActivityManager failed. reason:" + e)
    }
}
// 获取WIFI状态信息
function hookGetWifiState() {
    try {
        var wifiManager = Java.use("android.net.wifi.WifiManager");
        if (wifiManager.getWifiState != undefined) {
            wifiManager.getWifiState.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getWifiState()=======================\r\n");
                var temp = this.getWifiState();
                log_with_index(tmp_index, "getWifiState: " + temp);
                return temp;
            };
        if (wifiManager.getSSID != undefined) {
            wifiManager.getSSID.implementation = function () {
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getSSID()=======================\r\n");
                var temp = this.getSSID();
                log_with_index(tmp_index, "getSSID: " + temp);
                return temp;
            }
        }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetWifiState-android.net.wifi.WifiManager failed. reason:" + e)
    }
}
// 获取网络状态信息
function hookGetActiveNetworkInfo() {
    // This method was deprecated in API level 29.
    try {
        var CM = Java.use("android.net.ConnectivityManager");
        if (CM.getActiveNetworkInfo != undefined) {
            CM.getActiveNetworkInfo.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getActiveNetworkInfo()=======================\r\n");
                var temp = this.getActiveNetworkInfo();
                log_with_index(tmp_index, "getActiveNetworkInfo: " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetActiveNetworkInfo-android.net.ConnectivityManager failed. reason:" + e)
    }
}
// 获取hostaddress、hostname信息
function hookGetHostInfo() {
    try {
        var socketAddress = Java.use("java.net.InetSocketAddress");
        if (socketAddress.getHostAddress != undefined) {
            socketAddress.getHostAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHostAddress()=======================\r\n");
                var temp = this.getHostAddress();
                log_with_index(tmp_index, "getHostAddress: " + temp);
                return temp;
            };
            socketAddress.getAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getAddress()=======================\r\n");
                var temp = this.getAddress();
                log_with_index(tmp_index, "getAddress: " + temp);
                return temp;
            };
            socketAddress.getHostName.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHostName()=======================\r\n");
                var temp = this.getHostName();
                log_with_index(tmp_index, "getHostName: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetHostInfo-java.net.InetSocketAddress failed. reason:" + e)
    }
    try {
        var inetAddress = Java.use("java.net.InetAddress");
        if (inetAddress.getHostAddress != undefined) {
            inetAddress.getHostAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHostAddress()=======================\r\n");
                var temp = this.getHostAddress();
                log_with_index(tmp_index, "getHostAddress: " + temp);
                return temp;
            };
            inetAddress.getAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getAddress()=======================\r\n");
                var temp = this.getAddress();
                log_with_index(tmp_index, "getAddress: " + temp);
                return temp;
            };
            inetAddress.getHostName.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHostName()=======================\r\n");
                var temp = this.getHostName();
                log_with_index(tmp_index, "getHostName: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetHostInfo-java.net.InetAddress failed. reason:" + e)
    }
    try {
        var inet4Address = Java.use("java.net.Inet4Address");
        if (inet4Address.getHostAddress != undefined) {
            inet4Address.getHostAddress.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getHostAddress()=======================\r\n");
                var temp = this.getHostAddress();
                log_with_index(tmp_index, "getHostAddress: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetHostInfo-java.net.Inet4Address failed. reason:" + e)
    }
    try {
        var inet6Address = Java.use("java.net.Inet6Address");
        inet6Address.getHostAddress.implementation = function () {
            var tmp_index = get_tmp_index();
            showStacks(tmp_index);
            log_with_index(tmp_index, "============================= [*]Called - getHostAddress()=======================\r\n");
            var temp = this.getHostAddress();
            log_with_index(tmp_index, "getHostAddress: " + temp);
            return temp;
        };
    } catch (e) {
        log_with_index(-1, "Function hookGetHostInfo-java.net.Inet6Address failed. reason:" + e)
    }
}
// 获取网络连接信息
function hookGetSocketConnect() {
    try {
        var socket = Java.use("java.net.Socket");
        // 这里有个bug，当p1的值中不包含域名时(猜测)，this.connect会失败导致应用和Frida闪退，原因不明，该函数暂时弃用
        // https://stackoverflow.com/questions/56146503/can-anyone-help-me-how-to-hook-java-net-socket-connectjava-net-socketaddress-i
        if (socket.connect != undefined) {
            socket.connect.overload('java.net.SocketAddress', 'int').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - connect(p1,p2)=======================\r\n");
                log_with_index(tmp_index, p1, p2)
                var temp = this.connect(p1, p2);
                return temp;
            }
            socket.connect.overload('java.net.SocketAddress').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - connect(p1)=======================\r\n");
                log_with_index(tmp_index, p1)
                var temp = this.connect(p1);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetSocketConnect-java.net.Socket failed. reason:" + e)
    }
}
// 获取定位信息
function hookGetLocation() {
    try {
        var geocoder = Java.use("android.location.Geocoder");
        if (geocoder.getFromLocation != undefined) {
            // This method was deprecated in API level 33.
            geocoder.getFromLocation.overload('double', 'double', 'int').implementation = function (p1, p2, p3) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getFromLocation(p1,p2,p3)=======================\r\n");
                var temp = this.getFromLocation(p1, p2, p3);
                log_with_index(tmp_index, "getFromLocation: " + temp);
                return temp;
            }
            // This method was deprecated in API level 33.
            geocoder.getFromLocationName.overload('java.lang.String', 'int', 'double', 'double', 'double', 'double').implementation = function (p1, p2, p3, p4, p5, p6) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getFromLocationName(p1,p2,p3,p4,p5,p6)=======================\r\n");
                var temp = this.getFromLocationName(p1, p2, p3, p4, p5, p6);
                log_with_index(tmp_index, "getFromLocationName: " + temp);
                return temp;
            }
            geocoder.getFromLocationName.overload('java.lang.String', 'int').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getFromLocationName(p1,p2)=======================\r\n");
                var temp = this.getFromLocationName(p1, p2);
                log_with_index(tmp_index, "getFromLocationName: " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetLocation-android.location.Geocoder failed. reason:" + e)
    }
    try {
        var LocationManager = Java.use("android.location.LocationManager");
        if (LocationManager.requestLocationUpdates != undefined) {
            LocationManager.requestLocationUpdates.overload('android.location.LocationRequest', 'android.app.PendingIntent').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('android.location.LocationRequest', 'android.location.LocationListener', 'android.os.Looper').implementation = function (p1, p2, p3) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('android.location.LocationRequest', 'java.util.concurrent.Executor', 'android.location.LocationListener').implementation = function (p1, p2, p3) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('long', 'float', 'android.location.Criteria', 'android.app.PendingIntent').implementation = function (p1, p2, p3, p4) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('java.lang.String', 'long', 'float', 'android.app.PendingIntent').implementation = function (p1, p2, p3, p4) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('java.lang.String', 'long', 'float', 'android.location.LocationListener').implementation = function (p1, p2, p3, p4) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('long', 'float', 'android.location.Criteria', 'android.location.LocationListener', 'android.os.Looper').implementation = function (p1, p2, p3, p4, p5) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4,p5)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4, p5);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('long', 'float', 'android.location.Criteria', 'java.util.concurrent.Executor', 'android.location.LocationListener').implementation = function (p1, p2, p3, p4, p5) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4,p5)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4, p5);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('java.lang.String', 'long', 'float', 'android.location.LocationListener', 'android.os.Looper').implementation = function (p1, p2, p3, p4, p5) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4,p5)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4, p5);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('android.location.LocationRequest', 'android.location.LocationListener', 'android.os.Looper', 'android.app.PendingIntent').implementation = function (p1, p2, p3, p4) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.requestLocationUpdates.overload('java.lang.String', 'long', 'float', 'java.util.concurrent.Executor', 'android.location.LocationListener').implementation = function (p1, p2, p3, p4, p5) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - requestLocationUpdates(p1,p2,p3,p4,p5)=======================\r\n");
                var temp = this.requestLocationUpdates(p1, p2, p3, p4, p5);
                log_with_index(tmp_index, "requestLocationUpdates: " + temp);
                return temp;
            }
            LocationManager.getLastKnownLocation.overload('java.lang.String').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getLastKnownLocation(p1)=======================\r\n");
                var temp = this.getLastKnownLocation(p1);
                log_with_index(tmp_index, "getLastKnownLocation: " + temp);
                return temp;
            }
            LocationManager.getCurrentLocation.overload('java.lang.String', 'android.location.LocationRequest', 'android.os.CancellationSignal', 'java.util.concurrent.Executor', 'java.util.function.Consumer').implementation = function (p1, p2, p3, p4, p5) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getCurrentLocation(p1,p2,p3,p4,p5)=======================\r\n");
                var temp = this.getCurrentLocation(p1, p2, p3, p4, p5);
                log_with_index(tmp_index, "getCurrentLocation: " + temp);
                return temp;
            }
            LocationManager.getCurrentLocation.overload('java.lang.String', 'android.os.CancellationSignal', 'java.util.concurrent.Executor', 'java.util.function.Consumer').implementation = function (p1, p2, p3, p4) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getCurrentLocation(p1,p2,p3,p4)=======================\r\n");
                var temp = this.getCurrentLocation(p1, p2, p3, p4);
                log_with_index(tmp_index, "getCurrentLocation: " + temp);
                return temp;
            }
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetLocation-android.location.LocationManager failed. reason:" + e)
    }
}
// 获取GetInstallPackages信息
function hookGetInstallPackages() {
    try {
        var pmPackageManager = Java.use("android.content.pm.PackageManager");
        if (pmPackageManager.getInstalledPackages != undefined) {
            pmPackageManager.getInstalledPackages.overload('int').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - pm-getInstalledPackages()=======================\r\n");
                var temp = this.getInstalledPackages(p1);
                log_with_index(tmp_index, "getInstalledPackages: " + temp);
                return temp;
            };
            pmPackageManager.getInstalledApplications.overload('int').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - pm-getInstalledApplications()=======================\r\n");
                var temp = this.getInstalledApplications(p1);
                log_with_index(tmp_index, "getInstalledApplications: " + temp);
                return temp;
            };
            pmPackageManager.getInstalledModules.overload('int').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-getInstalledModules()=======================\r\n");
                var temp = this.getInstalledModules(p1);
                log_with_index(tmp_index, "getInstalledModules: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetInstallPackages-android.content.pm.PackageManager failed. reason:" + e)
    }
    try {
        var appPackageManager = Java.use("android.app.ApplicationPackageManager");
        if (appPackageManager.getInstalledPackages != undefined) {
            appPackageManager.getInstalledPackages.overload('int').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-getInstalledPackages()=======================\r\n");
                var temp = this.getInstalledPackages(p1);
                log_with_index(tmp_index, "getInstalledPackages: " + temp);
                return temp;
            };
            appPackageManager.getInstalledApplications.overload('int').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-getInstalledApplications()=======================\r\n");
                var temp = this.getInstalledApplications(p1);
                log_with_index(tmp_index, "getInstalledApplications: " + temp);
                return temp;
            };
            appPackageManager.queryIntentActivities.implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-queryIntentActivities()=======================\r\n");
                var temp = this.queryIntentActivities(p1, p2);
                log_with_index(tmp_index, "queryIntentActivities: " + temp);
                return temp;
            };
            appPackageManager.getInstalledApplicationsAsUser.overload('int', 'int').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-getInstalledApplicationsAsUser(p1,p2)=======================\r\n");
                var temp = this.getInstalledApplicationsAsUser(p1, p2);
                log_with_index(tmp_index, "getInstalledApplicationsAsUser: " + temp);
                return temp;
            };
            appPackageManager.getInstalledPackagesAsUser.overload('int', 'int').implementation = function (p1, p2) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - app-getInstalledPackagesAsUser(p1,p2)=======================\r\n");
                var temp = this.getInstalledPackagesAsUser(p1, p2);
                log_with_index(tmp_index, "getInstalledPackagesAsUser: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetInstallPackages-android.app.ApplicationPackageManager failed. reason:" + e)
    }
}
// 获取国内特色信息
// 这个需要在特定品牌手机上才能测试，原生系统不存在对应SDK,以下为小米品牌SDK
// 参考 https://www.ichdata.com/wp-content/uploads/2020/06/2021032423172817.pdf
// https://github.com/gzu-liyujiang/Android_CN_OAID
//xiaomi
function hookGetIdProvider() {
    try {
        var IdProvider = Java.use("com.android.id.impl.IdProviderImpl");
        if (IdProvider.getUDID != undefined) {
            IdProvider.getUDID.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getUDID()=======================\r\n");
                var temp = this.getUDID();
                log_with_index(tmp_index, "getUDID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getOAID != undefined) {
            IdProvider.getOAID.overload('android.content.Context').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getOAID(p1)=======================\r\n");
                var temp = this.getOAID(p1);
                log_with_index(tmp_index, "getOAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getVAID != undefined) {
            IdProvider.getVAID.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getVAID()=======================\r\n");
                var temp = this.getVAID();
                log_with_index(tmp_index, "getVAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getAAID != undefined) {
            IdProvider.getAAID.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getAAID()=======================\r\n");
                var temp = this.getAAID();
                log_with_index(tmp_index, "getAAID: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIdProvider-com.android.id.impl.IdProviderImpl failed. reason:" + e)
    }
}
// samsung
function hookGetIDeviceIdService() {
    try {
        var IdProvider = Java.use("com.samsung.android.deviceidservice.IDeviceIdService$Stub$Proxy");
        if (IdProvider.getOAID != undefined) {
            IdProvider.getOAID.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getOAID()=======================\r\n");
                var temp = this.getOAID();
                log_with_index(tmp_index, "getOAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getVAID != undefined) {
            IdProvider.getVAID.overload('java.lang.String').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getVAID(p1)=======================\r\n");
                var temp = this.getVAID(p1);
                log_with_index(tmp_index, "getVAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getAAID != undefined) {
            IdProvider.getAAID.overload('java.lang.String').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getAAID(p1)=======================\r\n");
                var temp = this.getAAID(p1);
                log_with_index(tmp_index, "getAAID: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIDeviceIdService-com.samsung.android.deviceidservice.IDeviceIdService$Stub$Proxy failed. reason:" + e)
    }
    try {
        var IdProvider = Java.use("repeackage.com.samsung.android.deviceidservice.IDeviceIdService$Stub$Proxy");
        if (IdProvider.getOAID != undefined) {
            IdProvider.getOAID.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getOAID()=======================\r\n");
                var temp = this.getOAID();
                log_with_index(tmp_index, "getOAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getVAID != undefined) {
            IdProvider.getVAID.overload('java.lang.String').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getVAID(p1)=======================\r\n");
                var temp = this.getVAID(p1);
                log_with_index(tmp_index, "getVAID: " + temp);
                return temp;
            };
        }
        if (IdProvider.getAAID != undefined) {
            IdProvider.getAAID.overload('java.lang.String').implementation = function (p1) {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getAAID(p1)=======================\r\n");
                var temp = this.getAAID(p1);
                log_with_index(tmp_index, "getAAID: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetIDeviceIdServicerepeackage.com.samsung.android.deviceidservice.IDeviceIdService$Stub$Proxy failed. reason:" + e)
    }
}
function hookGetICCID() {
    try {
        var UC = Java.use("android.telephony.UiccCardInfo");
        if (UC.getUDID != undefined) {
            IdProvider.getIccId.implementation = function () {
                var tmp_index = get_tmp_index();
                showStacks(tmp_index);
                log_with_index(tmp_index, "============================= [*]Called - getIccId()=======================\r\n");
                var temp = this.getIccId();
                log_with_index(tmp_index, "getIccId: " + temp);
                return temp;
            };
        }
    } catch (e) {
        log_with_index(-1, "Function hookGetICCID-android.telephony.UiccCardInfo failed. reason:" + e)
    }
}
Java.perform(function () {
    hookGetSystemInfo();
    hookRequestPermission();
    hookGetAndroidId();
    hookGetIMSI();
    hookGetIMEI();
    hookGetMEID();
    hookGetMacAddress();
    hookGetIPAddress();
    hookGetRunningAppProcesses();
    hookGetWifiState();
    hookGetActiveNetworkInfo();
    hookGetHostInfo();
    hookGetLocation();
    hookGetInstallPackages();
    hookGetIdProvider();
    hookGetIDeviceIdService();
    hookGetICCID()
})

```

### 12.hook 去除禁止截屏

```
function main() {
    Java.perform(function () {
        var v = Java.use("android.view.Window")
        v.setFlags.overload('int', 'int').implementation = function (arg1, arg2) {
            console.log('arg1', arg1)
            console.log('arg2', arg2)
            this.setFlags(1, 1)
        }
        // 有的会用这个
        // var s = Java.use("android.view.SurfaceView")
        // s.setSecure.overload('java.lang.Boolean').implementation = function (arg) {
        //     this.setSecure(false)
        // }
    })
}
setTimeout(main, 2000)

```

### 13.byte，byte[] 类型的字符打印

```
function byte2string(buf) {
    return Java.use("java.lang.String").$new(buf);
}

```

```
function printByteArray(jbytes) {
    // return JSON.stringify(jbytes);
    var result = "";
    for (var i = 0; i < jbytes.length; ++i) {
        result += " ";
        result += jbytes[i].toString(16);
    }
    return result;
}

```

```
function jbyteArray2Array(jbyteArray) {
    var ret;
    Java.perform(function () {
        var b = Java.use('[B');
        var buffer = Java.cast(jbyteArray, b);
        ret = Java.array('byte', buffer);
    });
    return ret;
}

```

```
function bytesToHex(arr) {
    var str = "";
    var k, j;
    for (var i = 0; i < arr.length; i++) {
        k = arr[i];
        j = k;
        if (k < 0) {
            j = k + 256;
        }
        if (j < 16) {
            str += "0";
        }
        str += j.toString(16);
    }
    return str;
}

```

### 14.hook_SharedPreferences

有时候有些 app，喜欢把一些资源数据存放在这里面，所以 hook 这个也可以拿到一些东西

```
function hook_SharedPreferencesImpl$EditorImpl_putString() {
    var editorImpl = Java.use("android.app.SharedPreferencesImpl$EditorImpl");
    editorImpl.putString.overload("java.lang.String", "java.lang.String").implementation = function (key, value) {
        console.log("putString key：" + key + " value:" + value);
        if (key == "uuid") {
            printCallback();
        }
        return this.putString(key, value);
    };
}

```

```
function hook_SharedPreferencesImpl_getString() {
    var sharedImpl = Java.use("android.app.SharedPreferencesImpl");
    sharedImpl.getString.overload("java.lang.String", "java.lang.String").implementation = function (key, def) {
        var value = this.getString(key, def);
        send("getString key：" + key + " value:" + value);
        return value;
    };
}

```

### 15.hook url

有时候抓包可能很难搞，可以通过这个 hook

```
function hook_url(bShowStacks) {
    // java.net.URL;
    const URL = Java.use('java.net.URL');
    URL.$init.overload('java.lang.String').implementation = function (url) {
        DMLog.i('hook_url', 'url: ' + url);
        if (bShowStacks) {
            showStacks();
        }
        return this.$init(url);
    }
}
function hook_url2(){
  var android_net_Uri_clz = Java.use('android.net.Uri');
  var android_net_Uri_clz_method_parse_u5rj = android_net_Uri_clz.parse.overload('java.lang.String');
   android_net_Uri_clz_method_parse_u5rj.implementation = function(url) {
        var executor = 'Class';
        var beatText = url + '\npublic static android.net.Uri android.net.Uri.parse(java.lang.String)';
        var beat = newMethodBeat(beatText, executor);
        var ret = android_net_Uri_clz_method_parse_u5rj.call(android_net_Uri_clz, url);
        if (check(url)) {
            printBeat(beat);
        }
        return ret;
    };
}

```

### 16.hook okhttp 的 body 以及自吐

```
function hookokhttpBody() {
    Java.perform(function () {
        var RequestBody = Java.use('okhttp3.RequestBody');
        console.log('-----------------> 有',RequestBody)
        RequestBody.contentLength.implementation = function () {
            // 打印请求Body内容长度
            showStacks()
            console.log('okhttp3.RequestBody', this.content.size());
            return this.contentLength();
        };
        RequestBody.writeTo.implementation = function (sink) {
            // 截获请求Body内容
            console.log(this.content.snapshot());
            return this.writeTo(sink);
        };
    });
}
function okhttp3AutoOut(){
    // okhttp3自吐
    Java.perform(function(){
        // 在内存中获取okhttp3.OkHttpClient$Builder内部句柄
        var Builder = Java.use("okhttp3.OkHttpClient$Builder")
        // 为创建代理对象做准备
        var Proxy = Java.use("java.net.Proxy")
        var TYPE = Java.use("java.net.Proxy$Type")
        var InetSocketAddress = Java.use("java.net.InetSocketAddress")
        var String = Java.use("java.lang.String")
        console.log(Builder)
        // hook okhttp3.OkHttpClient$Builder内部类的build方法代表最终构造OkHttpClient对象
        Builder.build.implementation = function(){
            // 设置代理服务器IP地址
            var ip_str = String.$new("192.168.30.244")
            // 创建代理服务器对象
            var Proxy_IP_PORT = InetSocketAddress.$new(ip_str, 8888)
            // 调用当前对象的proxy强行设置代理
            this.proxy(Proxy.$new(TYPE.HTTP.value, Proxy_IP_PORT))
            console.log("设置代理成功：协议：", TYPE.HTTP.value)
            return this.build()
        }
    })
}

```

### 17.hook json 相关

```
function hook_JSONObject_getString(pKey) {
    const JSONObject = Java.use('org.json.JSONObject');
    JSONObject.getString.implementation = function (key) {
        if (key == pKey) {
            DMLog.i('hook_JSONObject_getString', 'found key: ' + key);
            showStacks();
        }
        return this.getString(key);
    }
}
function hook_fastJson(pKey) {
    const fastJson = Java.use('com.alibaba.fastjson.JSONObject');
    fastJson.getString.implementation = function (key) {
        if (key == pKey) {
            DMLog.i('hook_fastJson getString', 'found key: ' + key);
            showStacks();
        }
        return this.getString(key);
    };
    fastJson.getJSONArray.implementation = function (key) {
        if (key == pKey) {
            DMLog.i('hook_fastJson getJSONArray', 'found key: ' + key);
            showStacks();
        }
        return this.getString(key);
    };
    fastJson.getJSONObject.implementation = function (key) {
        if (key == pKey) {
            DMLog.i('hook_fastJson getJSONObject', 'found key: ' + key);
            showStacks();
        }
        return this.getString(key);
    };
    fastJson.getInteger.implementation = function (key) {
        if (key == pKey) {
            DMLog.i('hook_fastJson getJSONObject', 'found key: ' + key);
            showStacks();
        }
        return this.getString(key);
    };
}
function toJSONString(obj) {
    if (null == obj) {
        return "obj is null";
    }
    let resstr = "";
    let GsonBuilder = null;
    try {
        GsonBuilder = Java.use('com.google.gson.GsonBuilder');
    } catch (e) {
        FCAnd.registGson();
        GsonBuilder = Java.use('com.google.gson.GsonBuilder');
    }
    if (null != GsonBuilder) {
        try {
            const gson = GsonBuilder.$new().serializeNulls()
                .serializeSpecialFloatingPointValues()
                .disableHtmlEscaping()
                .setLenient()
                .create();
            resstr = gson.toJson(obj);
        } catch (e) {
            DMLog.e('gson.toJson', 'exceipt: ' + e.toString());
            resstr = FCAnd.parseObject(obj);
        }
    }
    return resstr;
}
function parseObject(data) {
    try {
        const declaredFields = data.class.getDeclaredFields();
        let res = {};
        for (let i = 0; i < declaredFields.length; i++) {
            const field = declaredFields[i];
            field.setAccessible(true);
            const type = field.getType();
            let fdata = field.get(data);
            if (null != fdata) {
                if (type.getName() != "[B") {
                    fdata = fdata.toString();
                } else {
                    fdata = Java.array('byte', fdata);
                    fdata = JSON.stringify(fdata);
                }
            }
            res[field.getName()] = fdata;
        }
        return JSON.stringify(res);
    } catch (e) {
        return "parseObject except: " + e.toString();
    }
}

```

### 18.Map 类型打印

```
function hook_Map(pKey, accurately) {
    const Map = Java.use('java.util.Map');
    Map.put.implementation = function (key, val) {
        var bRes = false;
        if (accurately) {
            bRes = (key + "") == (pKey);
        } else {
            bRes = (key + "").indexOf(pKey) > -1;
        }
        if (bRes) {
            DMLog.i('map', 'key: ' + key);
            DMLog.i('map', 'val: ' + val);
            showStacks();
        }
        this.put(key, val);
    };
    const LinkedHashMap = Java.use('java.util.LinkedHashMap');
    LinkedHashMap.put.implementation = function (key1, val) {
        var bRes = false;
        if (accurately) {
            bRes = (key1 + "") == (pKey);
        } else {
            bRes = (key1 + "").indexOf(pKey) > -1;
        }
        if (null != key1 && bRes) {
            DMLog.i('LinkedHashMap', 'key: ' + key1);
            DMLog.i('LinkedHashMap', 'val: ' + val);
            showStacks();
        }
        return this.put(key1, val);
    };
}

```

### 19. 获取当前 app 包名

```
// 获取当前app的包名
function getPackageName() {
    var packageName = null;
    Java.performNow(function () {
        if (is_spawn) {
            packageName = get_self_process_name();
        } else {
            packageName = context.getPackageName()
            var packageName1 = context.getBasePackageName()
            console.log("context.getBasePackageName()=", packageName1)
        }
    })
    return packageName
}

```

### 20. 获取所有 activities

```
function objection_getActivities() {
    Java.performNow(function () {
        const packageManager = Java.use("android.content.pm.PackageManager");
        const GET_ACTIVITIES = packageManager.GET_ACTIVITIES.value;
        var ActivityThreadClass = Java.use("android.app.ActivityThread");
        var context = ActivityThreadClass.currentActivityThread().getApplication().getApplicationContext()
        context.getPackageManager().getPackageInfo(context.getPackageName(), GET_ACTIVITIES).activities.value.map((activityInfo) => {
            console.log(activityInfo.name.value)
        })
    })
}

```

### 21. 抓包绕过、sslpining disable

这个是简单版

```
function sslpinning_disable_system() {
    Java.performNow(function () {
        var PlatformClass = Java.use("com.android.org.conscrypt.Platform")
        var targetMethodName = "checkServerTrusted"
        var overloadsLength = PlatformClass[targetMethodName].overloads.length;
        for (var i = 0; i < overloadsLength; ++i) {
            PlatformClass[targetMethodName].overloads[i].implementation = function () {
                console.log("com.android.org.conscrypt.Platform.checkServerTrusted->i=" + i + " arguments=" + JSON.stringify(arguments))
            }
        }
    })
}
function sslpinning_disable_okhttp3() {
    Java.performNow(function () {
        var CertificatePinnerClass = Java.use("okhttp3.CertificatePinner")
        var targetMethodName = "check"
        var overloadsLength = CertificatePinnerClass[targetMethodName].overloads.length;
        for (var i = 0; i < overloadsLength; ++i) {
            CertificatePinnerClass[targetMethodName].overloads[i].implementation = function () {
                console.log("okhttp3.CertificatePinner.check->i=" + i + " arguments=" + JSON.stringify(arguments))
            }
        }
    })
}
function sslBypass1() {
    var array_list = Java.use("java.util.ArrayList");
    var ApiClient = Java.use('com.android.org.conscrypt.TrustManagerImpl');
    ApiClient.checkTrustedRecursive.implementation = function (a1, a2, a3, a4, a5, a6) {
        // console.log('Bypassing SSL Pinning');
        var k = array_list.$new();
        return k;
    }
}
function sslBypass2() {
    Java.perform(function () {
        console.log("");
        console.log("[.] Cert Pinning Bypass/Re-Pinning");
        var CertificateFactory = Java.use("java.security.cert.CertificateFactory");
        var FileInputStream = Java.use("java.io.FileInputStream");
        var BufferedInputStream = Java.use("java.io.BufferedInputStream");
        var X509Certificate = Java.use("java.security.cert.X509Certificate");
        var KeyStore = Java.use("java.security.KeyStore");
        var TrustManagerFactory = Java.use("javax.net.ssl.TrustManagerFactory");
        var SSLContext = Java.use("javax.net.ssl.SSLContext");
        // Load CAs from an InputStream
        console.log("[+] Loading our CA...")
        var cf = CertificateFactory.getInstance("X.509");
        try {
            var fileInputStream = FileInputStream.$new("/data/local/tmp/cert-der.crt");
        } catch (err) {
            console.log("[o] " + err);
        }
        var bufferedInputStream = BufferedInputStream.$new(fileInputStream);
        var ca = cf.generateCertificate(bufferedInputStream);
        bufferedInputStream.close();
        var certInfo = Java.cast(ca, X509Certificate);
        console.log("[o] Our CA Info: " + certInfo.getSubjectDN());
        // Create a KeyStore containing our trusted CAs
        console.log("[+] Creating a KeyStore for our CA...");
        var keyStoreType = KeyStore.getDefaultType();
        var keyStore = KeyStore.getInstance(keyStoreType);
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", ca);
        // Create a TrustManager that trusts the CAs in our KeyStore
        console.log("[+] Creating a TrustManager that trusts the CA in our KeyStore...");
        var tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
        var tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
        tmf.init(keyStore);
        console.log("[+] Our TrustManager is ready...");
        console.log("[+] Hijacking SSLContext methods now...")
        console.log("[-] Waiting for the app to invoke SSLContext.init()...")
        SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").implementation = function (a, b, c) {
            console.log("[o] App invoked javax.net.ssl.SSLContext.init...");
            SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").call(this, a, tmf.getTrustManagers(), c);
            console.log("[+] SSLContext initialized with our custom TrustManager!");
        }
    });
}

```

加强版

```
setTimeout(function() {
  Java.perform(function() {
    console.log('');
    console.log('======');
    console.log('[#] Android Bypass for various Certificate Pinning methods [#]');
    console.log('======');
    var errDict = {};
    // TrustManager (Android < 7) //
    ////////////////////////////////
    var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    var TrustManager = Java.registerClass({
      // Implement a custom TrustManager
      name: 'dev.asd.test.TrustManager',
      implements: [X509TrustManager],
      methods: {
        checkClientTrusted: function(chain, authType) {},
        checkServerTrusted: function(chain, authType) {},
        getAcceptedIssuers: function() {return []; }
      }
    });
    // Prepare the TrustManager array to pass to SSLContext.init()
    var TrustManagers = [TrustManager.$new()];
    // Get a handle on the init() on the SSLContext class
    var SSLContext_init = SSLContext.init.overload(
      '[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom');
    try {
      // Override the init method, specifying the custom TrustManager
      SSLContext_init.implementation = function(keyManager, trustManager, secureRandom) {
        console.log('[+] Bypassing Trustmanager (Android < 7) pinner');
        SSLContext_init.call(this, keyManager, TrustManagers, secureRandom);
      };
    } catch (err) {
      console.log('[-] TrustManager (Android < 7) pinner not found');
      //console.log(err);
    }
    // OkHTTPv3 (quadruple bypass) //
    /////////////////////////////////
    try {
      // Bypass OkHTTPv3 {1}
      var okhttp3_Activity_1 = Java.use('okhttp3.CertificatePinner');
      okhttp3_Activity_1.check.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {
        console.log('[+] Bypassing OkHTTPv3 {1}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] OkHTTPv3 {1} pinner not found');
      //console.log(err);
      errDict[err] = ['okhttp3.CertificatePinner', 'check'];
    }
    try {
      // Bypass OkHTTPv3 {2}
      // This method of CertificatePinner.check is deprecated but could be found in some old Android apps
      var okhttp3_Activity_2 = Java.use('okhttp3.CertificatePinner');
      okhttp3_Activity_2.check.overload('java.lang.String', 'java.security.cert.Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing OkHTTPv3 {2}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] OkHTTPv3 {2} pinner not found');
      //console.log(err);
      //errDict[err] = ['okhttp3.CertificatePinner', 'check'];
    }
    try {
      // Bypass OkHTTPv3 {3}
      var okhttp3_Activity_3 = Java.use('okhttp3.CertificatePinner');
      okhttp3_Activity_3.check.overload('java.lang.String', '[Ljava.security.cert.Certificate;').implementation = function(a, b) {
        console.log('[+] Bypassing OkHTTPv3 {3}: ' + a);
        return;
      };
    } catch(err) {
      console.log('[-] OkHTTPv3 {3} pinner not found');
      //console.log(err);
      errDict[err] = ['okhttp3.CertificatePinner', 'check'];
    }
    try {
      // Bypass OkHTTPv3 {4}
      var okhttp3_Activity_4 = Java.use('okhttp3.CertificatePinner');
      //okhttp3_Activity_4['check$okhttp'].implementation = function(a, b) {
      okhttp3_Activity_4.check$okhttp.overload('java.lang.String', 'kotlin.jvm.functions.Function0').implementation = function(a, b) {
        console.log('[+] Bypassing OkHTTPv3 {4}: ' + a);
        return;
      };
    } catch(err) {
      console.log('[-] OkHTTPv3 {4} pinner not found');
      //console.log(err);
      errDict[err] = ['okhttp3.CertificatePinner', 'check$okhttp'];
    }
    // Trustkit (triple bypass) //
    //////////////////////////////
    try {
      // Bypass Trustkit {1}
      var trustkit_Activity_1 = Java.use('com.datatheorem.android.trustkit.pinning.OkHostnameVerifier');
      trustkit_Activity_1.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function(a, b) {
        console.log('[+] Bypassing Trustkit {1}: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Trustkit {1} pinner not found');
      //console.log(err);
      errDict[err] = ['com.datatheorem.android.trustkit.pinning.OkHostnameVerifier', 'verify'];
    }
    try {
      // Bypass Trustkit {2}
      var trustkit_Activity_2 = Java.use('com.datatheorem.android.trustkit.pinning.OkHostnameVerifier');
      trustkit_Activity_2.verify.overload('java.lang.String', 'java.security.cert.X509Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing Trustkit {2}: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Trustkit {2} pinner not found');
      //console.log(err);
      errDict[err] = ['com.datatheorem.android.trustkit.pinning.OkHostnameVerifier', 'verify'];
    }
    try {
      // Bypass Trustkit {3}
      var trustkit_PinningTrustManager = Java.use('com.datatheorem.android.trustkit.pinning.PinningTrustManager');
      trustkit_PinningTrustManager.checkServerTrusted.overload('[Ljava.security.cert.X509Certificate;', 'java.lang.String').implementation = function(chain, authType) {
        console.log('[+] Bypassing Trustkit {3}');
      };
    } catch (err) {
      console.log('[-] Trustkit {3} pinner not found');
      //console.log(err);
      errDict[err] = ['com.datatheorem.android.trustkit.pinning.PinningTrustManager', 'checkServerTrusted'];
    }
    // TrustManagerImpl (Android > 7) //
    ////////////////////////////////////
    try {
      // Bypass TrustManagerImpl (Android > 7) {1}
      var array_list = Java.use("java.util.ArrayList");
      var TrustManagerImpl_Activity_1 = Java.use('com.android.org.conscrypt.TrustManagerImpl');
      TrustManagerImpl_Activity_1.checkTrustedRecursive.implementation = function(certs, ocspData, tlsSctData, host, clientAuth, untrustedChain, trustAnchorChain, used) {
        console.log('[+] Bypassing TrustManagerImpl (Android > 7) checkTrustedRecursive check for: '+ host);
        return array_list.$new();
      };
    } catch (err) {
      console.log('[-] TrustManagerImpl (Android > 7) checkTrustedRecursive check not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.TrustManagerImpl', 'checkTrustedRecursive'];
    }
    try {
      // Bypass TrustManagerImpl (Android > 7) {2} (probably no more necessary)
      var TrustManagerImpl_Activity_2 = Java.use('com.android.org.conscrypt.TrustManagerImpl');
      TrustManagerImpl_Activity_2.verifyChain.implementation = function(untrustedChain, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
        console.log('[+] Bypassing TrustManagerImpl (Android > 7) verifyChain check for: ' + host);
        return untrustedChain;
      };
    } catch (err) {
      console.log('[-] TrustManagerImpl (Android > 7) verifyChain check not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.TrustManagerImpl', 'verifyChain'];
    }
    // Appcelerator Titanium PinningTrustManager //
    ///////////////////////////////////////////////
    try {
      var appcelerator_PinningTrustManager = Java.use('appcelerator.https.PinningTrustManager');
      appcelerator_PinningTrustManager.checkServerTrusted.implementation = function(chain, authType) {
        console.log('[+] Bypassing Appcelerator PinningTrustManager');
        return;
      };
    } catch (err) {
      console.log('[-] Appcelerator PinningTrustManager pinner not found');
      //console.log(err);
      errDict[err] = ['appcelerator.https.PinningTrustManager', 'checkServerTrusted'];
    }
    // Fabric PinningTrustManager //
    ////////////////////////////////
    try {
      var fabric_PinningTrustManager = Java.use('io.fabric.sdk.android.services.network.PinningTrustManager');
      fabric_PinningTrustManager.checkServerTrusted.implementation = function(chain, authType) {
        console.log('[+] Bypassing Fabric PinningTrustManager');
        return;
      };
    } catch (err) {
      console.log('[-] Fabric PinningTrustManager pinner not found');
      //console.log(err);
      errDict[err] = ['io.fabric.sdk.android.services.network.PinningTrustManager', 'checkServerTrusted'];
    }
    // OpenSSLSocketImpl Conscrypt (double bypass) //
    /////////////////////////////////////////////////
    try {
      var OpenSSLSocketImpl = Java.use('com.android.org.conscrypt.OpenSSLSocketImpl');
      OpenSSLSocketImpl.verifyCertificateChain.implementation = function(certRefs, JavaObject, authMethod) {
        console.log('[+] Bypassing OpenSSLSocketImpl Conscrypt {1}');
      };
    } catch (err) {
      console.log('[-] OpenSSLSocketImpl Conscrypt {1} pinner not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.OpenSSLSocketImpl', 'verifyCertificateChain'];
    }
    try {
      var OpenSSLSocketImpl = Java.use('com.android.org.conscrypt.OpenSSLSocketImpl');
      OpenSSLSocketImpl.verifyCertificateChain.implementation = function(certChain, authMethod) {
        console.log('[+] Bypassing OpenSSLSocketImpl Conscrypt {2}');
      };
    } catch (err) {
      console.log('[-] OpenSSLSocketImpl Conscrypt {2} pinner not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.OpenSSLSocketImpl', 'verifyCertificateChain'];
    }
    // OpenSSLEngineSocketImpl Conscrypt //
    ///////////////////////////////////////
    try {
      var OpenSSLEngineSocketImpl_Activity = Java.use('com.android.org.conscrypt.OpenSSLEngineSocketImpl');
      OpenSSLEngineSocketImpl_Activity.verifyCertificateChain.overload('[Ljava.lang.Long;', 'java.lang.String').implementation = function(a, b) {
        console.log('[+] Bypassing OpenSSLEngineSocketImpl Conscrypt: ' + b);
      };
    } catch (err) {
      console.log('[-] OpenSSLEngineSocketImpl Conscrypt pinner not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.OpenSSLEngineSocketImpl', 'verifyCertificateChain'];
    }
    // OpenSSLSocketImpl Apache Harmony //
    //////////////////////////////////////
    try {
      var OpenSSLSocketImpl_Harmony = Java.use('org.apache.harmony.xnet.provider.jsse.OpenSSLSocketImpl');
      OpenSSLSocketImpl_Harmony.verifyCertificateChain.implementation = function(asn1DerEncodedCertificateChain, authMethod) {
        console.log('[+] Bypassing OpenSSLSocketImpl Apache Harmony');
      };
    } catch (err) {
      console.log('[-] OpenSSLSocketImpl Apache Harmony pinner not found');
      //console.log(err);
      errDict[err] = ['org.apache.harmony.xnet.provider.jsse.OpenSSLSocketImpl', 'verifyCertificateChain'];
    }
    // PhoneGap sslCertificateChecker //
    ////////////////////////////////////
    try {
      var phonegap_Activity = Java.use('nl.xservices.plugins.sslCertificateChecker');
      phonegap_Activity.execute.overload('java.lang.String', 'org.json.JSONArray', 'org.apache.cordova.CallbackContext').implementation = function(a, b, c) {
        console.log('[+] Bypassing PhoneGap sslCertificateChecker: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] PhoneGap sslCertificateChecker pinner not found');
      //console.log(err);
      errDict[err] = ['nl.xservices.plugins.sslCertificateChecker', 'execute'];
    }
    // IBM MobileFirst pinTrustedCertificatePublicKey (double bypass) //
    ////////////////////////////////////////////////////////////////////
    try {
      // Bypass IBM MobileFirst {1}
      var WLClient_Activity_1 = Java.use('com.worklight.wlclient.api.WLClient');
      WLClient_Activity_1.getInstance().pinTrustedCertificatePublicKey.overload('java.lang.String').implementation = function(cert) {
        console.log('[+] Bypassing IBM MobileFirst pinTrustedCertificatePublicKey {1}: ' + cert);
        return;
      };
      } catch (err) {
      console.log('[-] IBM MobileFirst pinTrustedCertificatePublicKey {1} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.api.WLClient', 'pinTrustedCertificatePublicKey'];
    }
    try {
      // Bypass IBM MobileFirst {2}
      var WLClient_Activity_2 = Java.use('com.worklight.wlclient.api.WLClient');
      WLClient_Activity_2.getInstance().pinTrustedCertificatePublicKey.overload('[Ljava.lang.String;').implementation = function(cert) {
        console.log('[+] Bypassing IBM MobileFirst pinTrustedCertificatePublicKey {2}: ' + cert);
        return;
      };
    } catch (err) {
      console.log('[-] IBM MobileFirst pinTrustedCertificatePublicKey {2} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.api.WLClient', 'pinTrustedCertificatePublicKey'];
    }
    // IBM WorkLight (ancestor of MobileFirst) HostNameVerifierWithCertificatePinning (quadruple bypass) //
    ///////////////////////////////////////////////////////////////////////////////////////////////////////
    try {
      // Bypass IBM WorkLight {1}
      var worklight_Activity_1 = Java.use('com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning');
      worklight_Activity_1.verify.overload('java.lang.String', 'javax.net.ssl.SSLSocket').implementation = function(a, b) {
        console.log('[+] Bypassing IBM WorkLight HostNameVerifierWithCertificatePinning {1}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] IBM WorkLight HostNameVerifierWithCertificatePinning {1} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning', 'verify'];
    }
    try {
      // Bypass IBM WorkLight {2}
      var worklight_Activity_2 = Java.use('com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning');
      worklight_Activity_2.verify.overload('java.lang.String', 'java.security.cert.X509Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing IBM WorkLight HostNameVerifierWithCertificatePinning {2}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] IBM WorkLight HostNameVerifierWithCertificatePinning {2} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning', 'verify'];
    }
    try {
      // Bypass IBM WorkLight {3}
      var worklight_Activity_3 = Java.use('com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning');
      worklight_Activity_3.verify.overload('java.lang.String', '[Ljava.lang.String;', '[Ljava.lang.String;').implementation = function(a, b) {
        console.log('[+] Bypassing IBM WorkLight HostNameVerifierWithCertificatePinning {3}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] IBM WorkLight HostNameVerifierWithCertificatePinning {3} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning', 'verify'];
    }
    try {
      // Bypass IBM WorkLight {4}
      var worklight_Activity_4 = Java.use('com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning');
      worklight_Activity_4.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function(a, b) {
        console.log('[+] Bypassing IBM WorkLight HostNameVerifierWithCertificatePinning {4}: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] IBM WorkLight HostNameVerifierWithCertificatePinning {4} pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.wlclient.certificatepinning.HostNameVerifierWithCertificatePinning', 'verify'];
    }
    // Conscrypt CertPinManager //
    //////////////////////////////
    try {
      var conscrypt_CertPinManager_Activity = Java.use('com.android.org.conscrypt.CertPinManager');
      conscrypt_CertPinManager_Activity.checkChainPinning.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {
        console.log('[+] Bypassing Conscrypt CertPinManager: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Conscrypt CertPinManager pinner not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.CertPinManager', 'checkChainPinning'];
    }
    // Conscrypt CertPinManager (Legacy) //
    ///////////////////////////////////////
    try {
      var legacy_conscrypt_CertPinManager_Activity = Java.use('com.android.org.conscrypt.CertPinManager');
      legacy_conscrypt_CertPinManager_Activity.isChainValid.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {
        console.log('[+] Bypassing Conscrypt CertPinManager (Legacy): ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Conscrypt CertPinManager (Legacy) pinner not found');
      //console.log(err);
      errDict[err] = ['com.android.org.conscrypt.CertPinManager', 'isChainValid'];
    }
    // CWAC-Netsecurity (unofficial back-port pinner for Android<4.2) CertPinManager //
    ///////////////////////////////////////////////////////////////////////////////////
    try {
      var cwac_CertPinManager_Activity = Java.use('com.commonsware.cwac.netsecurity.conscrypt.CertPinManager');
      cwac_CertPinManager_Activity.isChainValid.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {
        console.log('[+] Bypassing CWAC-Netsecurity CertPinManager: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] CWAC-Netsecurity CertPinManager pinner not found');
      //console.log(err);
      errDict[err] = ['com.commonsware.cwac.netsecurity.conscrypt.CertPinManager', 'isChainValid'];
    }
    // Worklight Androidgap WLCertificatePinningPlugin //
    /////////////////////////////////////////////////////
    try {
      var androidgap_WLCertificatePinningPlugin_Activity = Java.use('com.worklight.androidgap.plugin.WLCertificatePinningPlugin');
      androidgap_WLCertificatePinningPlugin_Activity.execute.overload('java.lang.String', 'org.json.JSONArray', 'org.apache.cordova.CallbackContext').implementation = function(a, b, c) {
        console.log('[+] Bypassing Worklight Androidgap WLCertificatePinningPlugin: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Worklight Androidgap WLCertificatePinningPlugin pinner not found');
      //console.log(err);
      errDict[err] = ['com.worklight.androidgap.plugin.WLCertificatePinningPlugin', 'execute'];
    }
    // Netty FingerprintTrustManagerFactory //
    //////////////////////////////////////////
    try {
      var netty_FingerprintTrustManagerFactory = Java.use('io.netty.handler.ssl.util.FingerprintTrustManagerFactory');
      //NOTE: sometimes this below implementation could be useful
      //var netty_FingerprintTrustManagerFactory = Java.use('org.jboss.netty.handler.ssl.util.FingerprintTrustManagerFactory');
      netty_FingerprintTrustManagerFactory.checkTrusted.implementation = function(type, chain) {
        console.log('[+] Bypassing Netty FingerprintTrustManagerFactory');
      };
    } catch (err) {
      console.log('[-] Netty FingerprintTrustManagerFactory pinner not found');
      //console.log(err);
      errDict[err] = ['io.netty.handler.ssl.util.FingerprintTrustManagerFactory', 'checkTrusted'];
    }
    // Squareup CertificatePinner [OkHTTP<v3] (double bypass) //
    ////////////////////////////////////////////////////////////
    try {
      // Bypass Squareup CertificatePinner  {1}
      var Squareup_CertificatePinner_Activity_1 = Java.use('com.squareup.okhttp.CertificatePinner');
      Squareup_CertificatePinner_Activity_1.check.overload('java.lang.String', 'java.security.cert.Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing Squareup CertificatePinner {1}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Squareup CertificatePinner {1} pinner not found');
      //console.log(err);
      errDict[err] = ['com.squareup.okhttp.CertificatePinner', 'check'];
    }
    try {
      // Bypass Squareup CertificatePinner {2}
      var Squareup_CertificatePinner_Activity_2 = Java.use('com.squareup.okhttp.CertificatePinner');
      Squareup_CertificatePinner_Activity_2.check.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {
        console.log('[+] Bypassing Squareup CertificatePinner {2}: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Squareup CertificatePinner {2} pinner not found');
      //console.log(err);
      errDict[err] = ['com.squareup.okhttp.CertificatePinner', 'check'];
    }
    // Squareup OkHostnameVerifier [OkHTTP v3] (double bypass) //
    /////////////////////////////////////////////////////////////
    try {
      // Bypass Squareup OkHostnameVerifier {1}
      var Squareup_OkHostnameVerifier_Activity_1 = Java.use('com.squareup.okhttp.internal.tls.OkHostnameVerifier');
      Squareup_OkHostnameVerifier_Activity_1.verify.overload('java.lang.String', 'java.security.cert.X509Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing Squareup OkHostnameVerifier {1}: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Squareup OkHostnameVerifier check not found');
      //console.log(err);
      errDict[err] = ['com.squareup.okhttp.internal.tls.OkHostnameVerifier', 'verify'];
    }
    try {
      // Bypass Squareup OkHostnameVerifier {2}
      var Squareup_OkHostnameVerifier_Activity_2 = Java.use('com.squareup.okhttp.internal.tls.OkHostnameVerifier');
      Squareup_OkHostnameVerifier_Activity_2.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function(a, b) {
        console.log('[+] Bypassing Squareup OkHostnameVerifier {2}: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Squareup OkHostnameVerifier check not found');
      //console.log(err);
      errDict[err] = ['com.squareup.okhttp.internal.tls.OkHostnameVerifier', 'verify'];
    }
    // Android WebViewClient (quadruple bypass) //
    //////////////////////////////////////////////
    try {
      // Bypass WebViewClient {1} (deprecated from Android 6)
      var AndroidWebViewClient_Activity_1 = Java.use('android.webkit.WebViewClient');
      AndroidWebViewClient_Activity_1.onReceivedSslError.overload('android.webkit.WebView', 'android.webkit.SslErrorHandler', 'android.net.http.SslError').implementation = function(obj1, obj2, obj3) {
        console.log('[+] Bypassing Android WebViewClient check {1}');
      };
    } catch (err) {
      console.log('[-] Android WebViewClient {1} check not found');
      //console.log(err)
      errDict[err] = ['android.webkit.WebViewClient', 'onReceivedSslError'];
    }
    // Not working properly temporarily disused
    //try {
    //  // Bypass WebViewClient {2}
    //  var AndroidWebViewClient_Activity_2 = Java.use('android.webkit.WebViewClient');
    //  AndroidWebViewClient_Activity_2.onReceivedHttpError.overload('android.webkit.WebView', 'android.webkit.WebResourceRequest', 'android.webkit.WebResourceResponse').implementation = function(obj1, obj2, obj3) {
    //    console.log('[+] Bypassing Android WebViewClient check {2}');
    //  };
    //} catch (err) {
    //  console.log('[-] Android WebViewClient {2} check not found');
    //  //console.log(err)
    //  errDict[err] = ['android.webkit.WebViewClient', 'onReceivedHttpError'];
    //}
    try {
      // Bypass WebViewClient {3}
      var AndroidWebViewClient_Activity_3 = Java.use('android.webkit.WebViewClient');
      //AndroidWebViewClient_Activity_3.onReceivedError.overload('android.webkit.WebView', 'int', 'java.lang.String', 'java.lang.String').implementation = function(obj1, obj2, obj3, obj4) {
      AndroidWebViewClient_Activity_3.onReceivedError.implementation = function(view, errCode, description, failingUrl) {
        console.log('[+] Bypassing Android WebViewClient check {3}');
      };
    } catch (err) {
      console.log('[-] Android WebViewClient {3} check not found');
      //console.log(err)
      errDict[err] = ['android.webkit.WebViewClient', 'onReceivedError'];
    }
    try {
      // Bypass WebViewClient {4}
      var AndroidWebViewClient_Activity_4 = Java.use('android.webkit.WebViewClient');
      AndroidWebViewClient_Activity_4.onReceivedError.overload('android.webkit.WebView', 'android.webkit.WebResourceRequest', 'android.webkit.WebResourceError').implementation = function(obj1, obj2, obj3) {
        console.log('[+] Bypassing Android WebViewClient check {4}');
      };
    } catch (err) {
      console.log('[-] Android WebViewClient {4} check not found');
      //console.log(err)
      errDict[err] = ['android.webkit.WebViewClient', 'onReceivedError'];
    }
    // Apache Cordova WebViewClient //
    //////////////////////////////////
    try {
      var CordovaWebViewClient_Activity = Java.use('org.apache.cordova.CordovaWebViewClient');
      CordovaWebViewClient_Activity.onReceivedSslError.overload('android.webkit.WebView', 'android.webkit.SslErrorHandler', 'android.net.http.SslError').implementation = function(obj1, obj2, obj3) {
        console.log('[+] Bypassing Apache Cordova WebViewClient check');
        obj3.proceed();
      };
    } catch (err) {
      console.log('[-] Apache Cordova WebViewClient check not found');
      //console.log(err);
    }
    // Boye AbstractVerifier //
    ///////////////////////////
    try {
      var boye_AbstractVerifier = Java.use('ch.boye.httpclientandroidlib.conn.ssl.AbstractVerifier');
      boye_AbstractVerifier.verify.implementation = function(host, ssl) {
        console.log('[+] Bypassing Boye AbstractVerifier check for: ' + host);
      };
    } catch (err) {
      console.log('[-] Boye AbstractVerifier check not found');
      //console.log(err);
      errDict[err] = ['ch.boye.httpclientandroidlib.conn.ssl.AbstractVerifier', 'verify'];
    }
    // Apache AbstractVerifier (quadruple bypass) //
    ////////////////////////////////////////////////
    try {
      var apache_AbstractVerifier_1 = Java.use('org.apache.http.conn.ssl.AbstractVerifier');
      apache_AbstractVerifier_1.verify.overload('java.lang.String', 'java.security.cert.X509Certificate').implementation = function(a, b) {
        console.log('[+] Bypassing Apache AbstractVerifier {1} check for: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Apache AbstractVerifier {1} check not found');
      //console.log(err);
      errDict[err] = ['org.apache.http.conn.ssl.AbstractVerifier', 'verify'];
    }
        try {
      var apache_AbstractVerifier_2 = Java.use('org.apache.http.conn.ssl.AbstractVerifier');
      apache_AbstractVerifier_2.verify.overload('java.lang.String', 'javax.net.ssl.SSLSocket').implementation = function(a, b) {
        console.log('[+] Bypassing Apache AbstractVerifier {2} check for: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Apache AbstractVerifier {2} check not found');
      //console.log(err);
      errDict[err] = ['org.apache.http.conn.ssl.AbstractVerifier', 'verify'];
    }
        try {
      var apache_AbstractVerifier_3 = Java.use('org.apache.http.conn.ssl.AbstractVerifier');
      apache_AbstractVerifier_3.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function(a, b) {
        console.log('[+] Bypassing Apache AbstractVerifier {3} check for: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Apache AbstractVerifier {3} check not found');
      //console.log(err);
      errDict[err] = ['org.apache.http.conn.ssl.AbstractVerifier', 'verify'];
    }
        try {
      var apache_AbstractVerifier_4 = Java.use('org.apache.http.conn.ssl.AbstractVerifier');
      apache_AbstractVerifier_4.verify.overload('java.lang.String', '[Ljava.lang.String;', '[Ljava.lang.String;', 'boolean').implementation = function(a, b, c, d) {
        console.log('[+] Bypassing Apache AbstractVerifier {4} check for: ' + a);
        return;
      };
    } catch (err) {
      console.log('[-] Apache AbstractVerifier {4} check not found');
      //console.log(err);
      errDict[err] = ['org.apache.http.conn.ssl.AbstractVerifier', 'verify'];
    }
    // Chromium Cronet //
    /////////////////////
    try {
      var CronetEngineBuilderImpl_Activity = Java.use("org.chromium.net.impl.CronetEngineBuilderImpl");
      // Setting argument to TRUE (default is TRUE) to disable Public Key pinning for local trust anchors
      CronetEngine_Activity.enablePublicKeyPinningBypassForLocalTrustAnchors.overload('boolean').implementation = function(a) {
        console.log("[+] Disabling Public Key pinning for local trust anchors in Chromium Cronet");
        var cronet_obj_1 = CronetEngine_Activity.enablePublicKeyPinningBypassForLocalTrustAnchors.call(this, true);
        return cronet_obj_1;
      };
      // Bypassing Chromium Cronet pinner
      CronetEngine_Activity.addPublicKeyPins.overload('java.lang.String', 'java.util.Set', 'boolean', 'java.util.Date').implementation = function(hostName, pinsSha256, includeSubdomains, expirationDate) {
        console.log("[+] Bypassing Chromium Cronet pinner: " + hostName);
        var cronet_obj_2 = CronetEngine_Activity.addPublicKeyPins.call(this, hostName, pinsSha256, includeSubdomains, expirationDate);
        return cronet_obj_2;
      };
    } catch (err) {
      console.log('[-] Chromium Cronet pinner not found')
      //console.log(err);
    }
    // Flutter Pinning packages http_certificate_pinning and ssl_pinning_plugin (double bypass) //
    //////////////////////////////////////////////////////////////////////////////////////////////
    try {
      // Bypass HttpCertificatePinning.check {1}
      var HttpCertificatePinning_Activity = Java.use('diefferson.http_certificate_pinning.HttpCertificatePinning');
      HttpCertificatePinning_Activity.checkConnexion.overload("java.lang.String", "java.util.List", "java.util.Map", "int", "java.lang.String").implementation = function (a, b, c ,d, e) {
        console.log('[+] Bypassing Flutter HttpCertificatePinning : ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Flutter HttpCertificatePinning pinner not found');
      //console.log(err);
      errDict[err] = ['diefferson.http_certificate_pinning.HttpCertificatePinning', 'checkConnexion'];
    }
    try {
      // Bypass SslPinningPlugin.check {2}
      var SslPinningPlugin_Activity = Java.use('com.macif.plugin.sslpinningplugin.SslPinningPlugin');
      SslPinningPlugin_Activity.checkConnexion.overload("java.lang.String", "java.util.List", "java.util.Map", "int", "java.lang.String").implementation = function (a, b, c ,d, e) {
        console.log('[+] Bypassing Flutter SslPinningPlugin: ' + a);
        return true;
      };
    } catch (err) {
      console.log('[-] Flutter SslPinningPlugin pinner not found');
      //console.log(err);
      errDict[err] = ['com.macif.plugin.sslpinningplugin.SslPinningPlugin', 'checkConnexion'];
    }
    // Unusual/obfuscated pinners bypass //
    ///////////////////////////////////////
    try {
      // Iterating all caught pinner errors and try to overload them
      for (var key in errDict) {
        var errStr = key;
        var targetClass = errDict[key][0]
        var targetFunc = errDict[key][1]
        var retType = Java.use(targetClass)[targetFunc].returnType.type;
        //console.log("errDict content: "+errStr+" "+targetClass+"."+targetFunc);
        if (String(errStr).includes('.overload')) {
          overloader(errStr, targetClass, targetFunc,retType);
        }
      }
    } catch (err) {
      //console.log('[-] The pinner "'+targetClass+'.'+targetFunc+'" is not unusual/obfuscated, skipping it..');
      //console.log(err);
    }
    // Dynamic SSLPeerUnverifiedException Bypasser                               //
    // An useful technique to bypass SSLPeerUnverifiedException failures raising //
    // when the Android app uses some uncommon SSL Pinning methods or an heavily //
    // code obfuscation. Inspired by an idea of: https://github.com/httptoolkit  //
    ///////////////////////////////////////////////////////////////////////////////
    try {
      var UnverifiedCertError = Java.use('javax.net.ssl.SSLPeerUnverifiedException');
      UnverifiedCertError.$init.implementation = function (reason) {
        try {
          var stackTrace = Java.use('java.lang.Thread').currentThread().getStackTrace();
          var exceptionStackIndex = stackTrace.findIndex(stack =>
            stack.getClassName() === "javax.net.ssl.SSLPeerUnverifiedException"
          );
          // Retrieve the method raising the SSLPeerUnverifiedException
          var callingFunctionStack = stackTrace[exceptionStackIndex + 1];
          var className = callingFunctionStack.getClassName();
          var methodName = callingFunctionStack.getMethodName();
          var callingClass = Java.use(className);
          var callingMethod = callingClass[methodName];
          console.log('\x1b[36m[!] Unexpected SSLPeerUnverifiedException occurred related to the method "'+className+'.'+methodName+'"\x1b[0m');
          //console.log("Stacktrace details:\n"+stackTrace);
          // Checking if the SSLPeerUnverifiedException was generated by an usually negligible (not blocking) method
          if (className == 'com.android.org.conscrypt.ActiveSession' || className == 'com.google.android.gms.org.conscrypt.ActiveSession') {
            throw 'Reason: skipped SSLPeerUnverifiedException bypass since the exception was raised from a (usually) non blocking method on the Android app';
          }
          else {
            console.log('\x1b[34m[!] Starting to dynamically circumvent the SSLPeerUnverifiedException for the method "'+className+'.'+methodName+'"...\x1b[0m');
            var retTypeName = callingMethod.returnType.type;
            // Skip it when the calling method was already bypassed with Frida
            if (!(callingMethod.implementation)) {
              // Trying to bypass (via implementation) the SSLPeerUnverifiedException if due to an uncommon SSL Pinning method
              callingMethod.implementation = function() {
                console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+className+'.'+methodName+'" via Frida function implementation\x1b[0m');
                returner(retTypeName);
              }
            }
          }
        } catch (err2) {
          // Dynamic circumvention via function implementation does not works, then trying via function overloading
          if (String(err2).includes('.overload')) {
            overloader(err2, className, methodName, retTypeName);
          } else {
            if (String(err2).includes('SSLPeerUnverifiedException')) {
              console.log('\x1b[36m[-] Failed to dynamically circumvent SSLPeerUnverifiedException -> '+err2+'\x1b[0m');
            } else {
              //console.log('\x1b[36m[-] Another kind of exception raised during overloading  -> '+err2+'\x1b[0m');
            }
          }
        }
        //console.log('\x1b[36m[+] SSLPeerUnverifiedException hooked\x1b[0m');
        return this.$init(reason);
      };
    } catch (err1) {
      //console.log('\x1b[36m[-] SSLPeerUnverifiedException not found\x1b[0m');
      //console.log('\x1b[36m'+err1+'\x1b[0m');
    }
  });
}, 0);
function returner(typeName) {
  // This is a improvable rudimentary fix, if not works you can patch it manually
  //console.log("typeName: "+typeName)
  if (typeName === undefined || typeName === 'void') {
    return;
  } else if (typeName === 'boolean') {
    return true;
  } else {
    return null;
  }
}
function overloader(errStr, targetClass, targetFunc, retType) {
  // One ring to overload them all.. ;-)
  var tClass = Java.use(targetClass);
  var tFunc = tClass[targetFunc];
  var params = [];
  var argList = [];
  var overloads = tFunc.overloads;
  var returnTypeName = retType;
  var splittedList = String(errStr).split('.overload');
  for (var n=1; n<splittedList.length; n++) {
    var extractedOverload = splittedList[n].trim().split('(')[1].slice(0,-1).replaceAll("'","");
    // Discarding useless error strings
    if (extractedOverload.includes('<signature>')) {
      continue;
    }
    console.log('\x1b[34m[!] Found the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"\x1b[0m');
    // Check if extractedOverload is empty
    if (!extractedOverload) {
      // Overloading method withouth arguments
      tFunc.overload().implementation = function() {
        var printStr = printer();
        console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
        returner(returnTypeName);
      }
    } else {
      // Check if extractedOverload has multiple arguments
      if (extractedOverload.includes(',')) {
        argList = extractedOverload.split(', ');
      }
      // Considering max 8 arguments for the method to overload (Note: increase it, if needed)
      if (argList.length == 0) {
        tFunc.overload(extractedOverload).implementation = function(a) {
          var printStr = printer();
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      } else if (argList.length == 2) {
        tFunc.overload(argList[0], argList[1]).implementation = function(a,b) {
          var printStr = printer(a);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      } else if (argList.length == 3) {
        tFunc.overload(argList[0], argList[1], argList[2]).implementation = function(a,b,c) {
          var printStr = printer(a,b);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      } else if (argList.length == 4) {
        tFunc.overload(argList[0], argList[1], argList[2], argList[3]).implementation = function(a,b,c,d) {
          var printStr = printer(a,b,c);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      }  else if (argList.length == 5) {
        tFunc.overload(argList[0], argList[1], argList[2], argList[3], argList[4]).implementation = function(a,b,c,d,e) {
          var printStr = printer(a,b,c,d);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      }  else if (argList.length == 6) {
        tFunc.overload(argList[0], argList[1], argList[2], argList[3], argList[4], argList[5]).implementation = function(a,b,c,d,e,f) {
          var printStr = printer(a,b,c,d,e);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      }  else if (argList.length == 7) {
        tFunc.overload(argList[0], argList[1], argList[2], argList[3], argList[4], argList[5], argList[6]).implementation = function(a,b,c,d,e,f,g) {
          var printStr = printer(a,b,c,d,e,f);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      }  else if (argList.length == 8) {
        tFunc.overload(argList[0], argList[1], argList[2], argList[3], argList[4], argList[5], argList[6], argList[7]).implementation = function(a,b,c,d,e,f,g,h) {
          var printStr = printer(a,b,c,d,e,f,g);
          console.log('\x1b[34m[+] Bypassing the unusual/obfuscated pinner "'+targetClass+'.'+targetFunc+'('+extractedOverload+')"'+printStr+'\x1b[0m');
          returner(returnTypeName);
        }
      }
    }
  }
}
function printer(a,b,c,d,e,f,g,h) {
  // Build the string to print for the overloaded pinner
  var printList = [];
  var printStr = '';
  if (typeof a === 'string') {
    printList.push(a);
  }
  if (typeof b === 'string') {
    printList.push(b);
  }
  if (typeof c === 'string') {
    printList.push(c);
  }
  if (typeof d === 'string') {
    printList.push(d);
  }
  if (typeof e === 'string') {
    printList.push(e);
  }
  if (typeof f === 'string') {
    printList.push(f);
  }
  if (typeof g === 'string') {
    printList.push(g);
  }
  if (typeof h === 'string') {
    printList.push(h);
  }
  if (printList.length !== 0) {
    printStr = ' check for:';
    for (var i=0; i<printList.length; i++) {
      printStr += ' '+printList[i];
    }
  }
  return printStr;
}

```

### 22. hook jni_onload

```
function hook_JNI_OnLoad() {
    // jint JNI_OnLoad(JavaVM *vm, void *reserved)
    // var JNI_Onload_addr = DebugSymbol.fromName("JNI_OnLoad").address;
    var JNI_Onload_addr = Module.findExportByName("xxx.so", "JNI_OnLoad");
    // var JNI_Onload_addr = Module.findExportByName(null,"JNI_OnLoad");
    console.log("JNI_Onload_addr=", JNI_Onload_addr)
    Interceptor.attach(JNI_Onload_addr, {
        onEnter: function (args) {
            // console.log("enter JNI_Onload_addr")
        },
        onLeave: function (retval) {
            // console.log("leave JNI_Onload_addr")
        }
    })
}

```

### 23. hook dlopen

```
function hook_dlopen() {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
        {
            onEnter: function (args) {
                var pathptr = args[0];
                if (pathptr !== undefined && pathptr != null) {
                    var path = ptr(pathptr).readCString();
                    console.log("load " + path);
                    // if (path.indexOf("libmsaoaidsec.so") > -1) {
                    //     hook_dlsym()
                    // }
                }
            }
        }
    );
}

```

这个可用来追踪 frida 对抗的相关方法或者类在哪一个 so 里面

下面还有个 hook open，跟 open 相关的 hook

```
function hook_open() {
    var pth = Module.findExportByName(null, "open");
    Interceptor.attach(ptr(pth), {
        onEnter: function (args) {
            this.filename = args[0];
            console.log("", this.filename.readCString())
            if (this.filename.readCString().indexOf(".so") != -1) {
                args[0] = ptr(0)
            }
        }, onLeave: function (retval) {
            return retval;
        }
    })
}

```

### 24. hook native

```
function hook_native() {
    var module_libart = Process.findModuleByName("libart.so");
    var symbols = module_libart.enumerateSymbols();
    var ArtMethod_Invoke = null;
    for (var i = 0; i < symbols.length; i++) {
        var symbol = symbols[i];
        var address = symbol.address;
        var name = symbol.name;
        var indexArtMethod = name.indexOf("ArtMethod");
        var indexInvoke = name.indexOf("Invoke");
        var indexThread = name.indexOf("Thread");
        if (indexArtMethod >= 0
            && indexInvoke >= 0
            && indexThread >= 0
            && indexArtMethod < indexInvoke
            && indexInvoke < indexThread) {
            console.log(name);
            ArtMethod_Invoke = address;
        }
    }
    if (ArtMethod_Invoke) {
        Interceptor.attach(ArtMethod_Invoke, {
            onEnter: function (args) {
                var method_name = prettyMethod(args[0], 0);
                if (!(method_name.indexOf("java.") == 0 || method_name.indexOf("android.") == 0)) {
                    console.log("ArtMethod Invoke:" + method_name + '  called from:\n' +
                        Thread.backtrace(this.context, Backtracer.ACCURATE)
                            .map(DebugSymbol.fromAddress).join('\n') + '\n');
                }
            }
        });
    }
}

```

### 24. hook art

```
function hook_libart() {
    var symbols = Module.enumerateSymbolsSync("libart.so");
    var addrGetStringUTFChars = null;
    var addrNewStringUTF = null;
    var addrFindClass = null;
    var addrGetMethodID = null;
    var addrGetStaticMethodID = null;
    var addrGetFieldID = null;
    var addrGetStaticFieldID = null;
    var addrRegisterNatives = null;
    var so_name = "lib";      //TODO 这里写需要过滤的so
    for (var i = 0; i < symbols.length; i++) {
        var symbol = symbols[i];
        if (symbol.name.indexOf("art") >= 0 &&
            symbol.name.indexOf("JNI") >= 0 &&
            symbol.name.indexOf("CheckJNI") < 0 &&
            symbol.name.indexOf("_ZN3art3JNIILb0") >= 0
        ) {
            if (symbol.name.indexOf("GetStringUTFChars") >= 0) {
                addrGetStringUTFChars = symbol.address;
                console.log("GetStringUTFChars is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("NewStringUTF") >= 0) {
                addrNewStringUTF = symbol.address;
                console.log("NewStringUTF is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("FindClass") >= 0) {
                addrFindClass = symbol.address;
                console.log("FindClass is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("GetMethodID") >= 0) {
                addrGetMethodID = symbol.address;
                console.log("GetMethodID is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("GetStaticMethodID") >= 0) {
                addrGetStaticMethodID = symbol.address;
                console.log("GetStaticMethodID is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("GetFieldID") >= 0) {
                addrGetFieldID = symbol.address;
                console.log("GetFieldID is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("GetStaticFieldID") >= 0) {
                addrGetStaticFieldID = symbol.address;
                console.log("GetStaticFieldID is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("RegisterNatives") >= 0) {
                addrRegisterNatives = symbol.address;
                console.log("RegisterNatives is at ", symbol.address, symbol.name);
            } else if (symbol.name.indexOf("CallStatic") >= 0) {
                console.log("CallStatic is at ", symbol.address, symbol.name);
                Interceptor.attach(symbol.address, {
                    onEnter: function (args) {
                        var module = Process.findModuleByAddress(this.returnAddress);
                        if (module != null && module.name.indexOf(so_name) == 0) {
                            var java_class = args[1];
                            var mid = args[2];
                            var class_name = Java.vm.tryGetEnv().getClassName(java_class);
                            if (class_name.indexOf("java.") == -1 && class_name.indexOf("android.") == -1) {
                                var method_name = prettyMethod(mid, 1);
                                console.log("<>CallStatic:", DebugSymbol.fromAddress(this.returnAddress), class_name, method_name);
                            }
                        }
                    },
                    onLeave: function (retval) { }
                });
            } else if (symbol.name.indexOf("CallNonvirtual") >= 0) {
                console.log("CallNonvirtual is at ", symbol.address, symbol.name);
                Interceptor.attach(symbol.address, {
                    onEnter: function (args) {
                        var module = Process.findModuleByAddress(this.returnAddress);
                        if (module != null && module.name.indexOf(so_name) == 0) {
                            var jobject = args[1];
                            var jclass = args[2];
                            var jmethodID = args[3];
                            var obj_class_name = Java.vm.tryGetEnv().getObjectClassName(jobject);
                            var class_name = Java.vm.tryGetEnv().getClassName(jclass);
                            if (class_name.indexOf("java.") == -1 && class_name.indexOf("android.") == -1) {
                                var method_name = prettyMethod(jmethodID, 1);
                                console.log("<>CallNonvirtual:", DebugSymbol.fromAddress(this.returnAddress), class_name, obj_class_name, method_name);
                            }
                        }
                    },
                    onLeave: function (retval) { }
                });
            } else if (symbol.name.indexOf("Call") >= 0 && symbol.name.indexOf("Method") >= 0) {
                console.log("Call<>Method is at ", symbol.address, symbol.name);
                Interceptor.attach(symbol.address, {
                    onEnter: function (args) {
                        var module = Process.findModuleByAddress(this.returnAddress);
                        if (module != null && module.name.indexOf(so_name) == 0) {
                            var java_class = args[1];
                            var mid = args[2];
                            var class_name = Java.vm.tryGetEnv().getObjectClassName(java_class);
                            if (class_name.indexOf("java.") == -1 && class_name.indexOf("android.") == -1) {
                                var method_name = prettyMethod(mid, 1);
                                console.log("<>Call<>Method:", DebugSymbol.fromAddress(this.returnAddress), class_name, method_name);
                            }
                        }
                    },
                    onLeave: function (retval) { }
                });
            }
        }
    }
    if (addrGetStringUTFChars != null) {
        Interceptor.attach(addrGetStringUTFChars, {
            onEnter: function (args) {
            },
            onLeave: function (retval) {
                if (retval != null) {
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var bytes = Memory.readCString(retval);
                        console.log("[GetStringUTFChars] result:" + bytes, DebugSymbol.fromAddress(this.returnAddress));
                    }
                }
            }
        });
    }
    if (addrNewStringUTF != null) {
        Interceptor.attach(addrNewStringUTF, {
            onEnter: function (args) {
                if (args[1] != null) {
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var string = Memory.readCString(args[1]);
                        console.log("[NewStringUTF] bytes:" + string, DebugSymbol.fromAddress(this.returnAddress));
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrFindClass != null) {
        Interceptor.attach(addrFindClass, {
            onEnter: function (args) {
                if (args[1] != null) {
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var name = Memory.readCString(args[1]);
                        console.log("[FindClass] name:" + name, DebugSymbol.fromAddress(this.returnAddress));
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrGetMethodID != null) {
        Interceptor.attach(addrGetMethodID, {
            onEnter: function (args) {
                if (args[2] != null) {
                    var clazz = args[1];
                    var class_name = Java.vm.tryGetEnv().getClassName(clazz);
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var name = Memory.readCString(args[2]);
                        if (args[3] != null) {
                            var sig = Memory.readCString(args[3]);
                            console.log("[GetMethodID] class_name:" + class_name + " name:" + name + ", sig:" + sig, DebugSymbol.fromAddress(this.returnAddress));
                        } else {
                            console.log("[GetMethodID] class_name:" + class_name + " name:" + name, DebugSymbol.fromAddress(this.returnAddress));
                        }
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrGetStaticMethodID != null) {
        Interceptor.attach(addrGetStaticMethodID, {
            onEnter: function (args) {
                if (args[2] != null) {
                    var clazz = args[1];
                    var class_name = Java.vm.tryGetEnv().getClassName(clazz);
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var name = Memory.readCString(args[2]);
                        if (args[3] != null) {
                            var sig = Memory.readCString(args[3]);
                            console.log("[GetStaticMethodID] class_name:" + class_name + " name:" + name + ", sig:" + sig, DebugSymbol.fromAddress(this.returnAddress));
                        } else {
                            console.log("[GetStaticMethodID] class_name:" + class_name + " name:" + name, DebugSymbol.fromAddress(this.returnAddress));
                        }
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrGetFieldID != null) {
        Interceptor.attach(addrGetFieldID, {
            onEnter: function (args) {
                if (args[2] != null) {
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var name = Memory.readCString(args[2]);
                        if (args[3] != null) {
                            var sig = Memory.readCString(args[3]);
                            console.log("[GetFieldID] name:" + name + ", sig:" + sig, DebugSymbol.fromAddress(this.returnAddress));
                        } else {
                            console.log("[GetFieldID] name:" + name, DebugSymbol.fromAddress(this.returnAddress));
                        }
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrGetStaticFieldID != null) {
        Interceptor.attach(addrGetStaticFieldID, {
            onEnter: function (args) {
                if (args[2] != null) {
                    var module = Process.findModuleByAddress(this.returnAddress);
                    if (module != null && module.name.indexOf(so_name) == 0) {
                        var name = Memory.readCString(args[2]);
                        if (args[3] != null) {
                            var sig = Memory.readCString(args[3]);
                            console.log("[GetStaticFieldID] name:" + name + ", sig:" + sig, DebugSymbol.fromAddress(this.returnAddress));
                        } else {
                            console.log("[GetStaticFieldID] name:" + name, DebugSymbol.fromAddress(this.returnAddress));
                        }
                    }
                }
            },
            onLeave: function (retval) { }
        });
    }
    if (addrRegisterNatives != null) {
        Interceptor.attach(addrRegisterNatives, {
            onEnter: function (args) {
                console.log("[RegisterNatives] method_count:", args[3], DebugSymbol.fromAddress(this.returnAddress));
                var env = args[0];
                var java_class = args[1];
                var class_name = Java.vm.tryGetEnv().getClassName(java_class);
                var methods_ptr = ptr(args[2]);
                var method_count = parseInt(args[3]);
                for (var i = 0; i < method_count; i++) {
                    var name_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3));
                    var sig_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3 + Process.pointerSize));
                    var fnPtr_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3 + Process.pointerSize * 2));
                    var name = Memory.readCString(name_ptr);
                    var sig = Memory.readCString(sig_ptr);
                    var find_module = Process.findModuleByAddress(fnPtr_ptr);
                    console.log("[RegisterNatives] java_class:", class_name, "name:", name, "sig:", sig, "fnPtr:", fnPtr_ptr, "module_name:", find_module.name, "module_base:", find_module.base, "offset:", ptr(fnPtr_ptr).sub(find_module.base));
                }
            },
            onLeave: function (retval) { }
        });
    }
}

```

### 25. hook exit、kill 方法

用来防止 app 崩溃的，一般是用于一些检测到环境异常崩溃的，加上打印调用栈的，就可以定位是哪里检测的

```
const STD_STRING_SIZE = 3 * Process.pointerSize;
class StdString {
    constructor() {
        this.handle = Memory.alloc(STD_STRING_SIZE);
    }
    dispose() {
        const [data, isTiny] = this._getData();
        if (!isTiny) {
            Java.api.$delete(data);
        }
    }
    disposeToString() {
        const result = this.toString();
        this.dispose();
        return result;
    }
    toString() {
        const [data] = this._getData();
        return data.readUtf8String();
    }
    _getData() {
        const str = this.handle;
        const isTiny = (str.readU8() & 1) === 0;
        const data = isTiny ? str.add(1) : str.add(2 * Process.pointerSize).readPointer();
        return [data, isTiny];
    }
}
function prettyMethod(method_id, withSignature) {
    const result = new StdString();
    Java.api['art::ArtMethod::PrettyMethod'](result, method_id, withSignature ? 1 : 0);
    return result.disposeToString();
}
function hook_libc_exit() {
    var exit = Module.findExportByName("libc.so", "exit");
    console.log("native:" + exit);
    Interceptor.attach(exit, {
        onEnter: function (args) {
            console.log(Thread.backtrace(this.context, Backtracer.FUZZY).map(DebugSymbol.fromAddress).join("\n"));
        },
        onLeave: function (retval) {
            //send("gifcore so result value: "+retval);
        }
    });
}
function anti_exit() {
    const exit_ptr = Module.findExportByName(null, '_exit');
    DMLog.i('anti_exit', "exit_ptr : " + exit_ptr);
    if (null == exit_ptr) {
        return;
    }
    Interceptor.replace(exit_ptr, new NativeCallback(function (code) {
        if (null == this) {
            return 0;
        }
        var lr = FCCommon.getLR(this.context);
        DMLog.i('exit debug', 'entry, lr: ' + lr);
        return 0;
    }, 'int', ['int', 'int']));
}
function anti_kill() {
    const kill_ptr = Module.findExportByName(null, 'kill');
    DMLog.i('anti_kill', "kill_ptr : " + kill_ptr);
    if (null == kill_ptr) {
        return;
    }
    Interceptor.replace(kill_ptr, new NativeCallback(function (ptid, code) {
        if (null == this) {
            return 0;
        }
        var lr = FCCommon.getLR(this.context);
        DMLog.i('kill debug', 'entry, lr: ' + lr);
        FCAnd.showNativeStacks(this.context);
        return 0;
    }, 'int', ['int', 'int']));
}

```

### 26. hook java 的 string 类

这个很容易导致 app 崩，会打印很多日志出来，所以最好在 attach 模式下操作或者进入 app 后，终端里主动执行这个方法

```
function hookjavalangString() {
    Java.perform(function () {
        var JavaString = Java.use('java.lang.String');
        JavaString.$init.overload('java.lang.String').implementation = function (content) {
            console.log('JavaString.$init.overload(\'java.lang.String\')->' + content);
            var returnValue = this.$init(content);
            return returnValue;
        };
        JavaString.$init.overload('[C').implementation = function (content) {
            console.log("JavaString.$init.overload('[C')->" + content);
            var returnValue = this.$init(content);
            return returnValue;
        };
        var StringFactory = Java.use('java.lang.StringFactory');
        StringFactory.newStringFromString.implementation = function (arg0) {
            console.log("java.lang.StringFactory.newStringFromString->" + arg0);
            var returnValue = this.newStringFromString(arg0);
            return returnValue;
        };
        JavaString.toString.implementation = function () {
            var str = this.toString();
            // 自定义处理
            console.log("java.lang.StringFactory.toString->" + str);
            // 返回修改后的字符串
            return str;
        };
        JavaString.getBytes.overload().implementation = function () {
            console.log('java.lang.StringFactory.getBytes overload before->', this.toString())
            var returnValue = this.getBytes();
            // 添加自定义处理
            console.log('java.lang.StringFactory.getBytes overload ->', returnValue)
            // 修改返回值
            // returnValue = [1, 2, 3];
            return returnValue
        };
        JavaString.getBytes.overload('java.lang.String').implementation = function (arg) {
            console.log('java.lang.StringFactory.getBytes overload(\'java.lang.String\') arg->', arg)
            console.log('java.lang.StringFactory.getBytes overload(\'java.lang.String\') before->', this.toString())
            var returnValue = this.getBytes(arg);
            // 添加自定义处理
            console.log('java.lang.StringFactory.getBytes overload(\'java.lang.String\')->', returnValue)
            // 修改返回值
            // returnValue = [1, 2, 3];
            return returnValue
        };
        JavaString.getBytes.overload('java.nio.charset.Charset').implementation = function (arg) {
            console.log('java.lang.StringFactory.getBytes .overload(\'java.nio.charset.Charset\') arg->', arg)
            console.log('java.lang.StringFactory.getBytes .overload(\'java.nio.charset.Charset\') before->', this.toString())
            var returnValue = this.getBytes(arg);
            // 添加自定义处理
            console.log('java.lang.StringFactory.getBytes .overload(\'java.nio.charset.Charset\')->', returnValue)
            // 修改返回值
            // returnValue = [1, 2, 3];
            return returnValue
        };
        JavaString.getBytes.overload('int', 'int', '[B', 'int').implementation = function (arg1, arg2, arg3, arg4) {
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\') arg1->', arg1)
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\') arg2->', arg2)
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\') arg3->', arg3)
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\') arg4->', arg4)
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\') before->', this.toString())
            var returnValue = this.getBytes(arg1, arg2, arg3, arg4);
            // 添加自定义处理
            console.log('java.lang.StringFactory.getBytes .overload(\'int\', \'int\', \'[B\', \'int\')->', returnValue)
            // 修改返回值
            // returnValue = [1, 2, 3];
            return returnValue
        };
    })
}

```

### 27. hook  list 类

```
function hookList() {
    Java.perform(function () {
        // 拦截 ArrayList 实现类
        var ArrayList = Java.use('java.util.ArrayList');
        ArrayList.add.overload('java.lang.Object').implementation = function (obj) {
            // 自定义操作
            console.log('list add ', obj);
            // 调用原方法
            return this.add(obj);
        };
        ArrayList.add.overload('int', 'java.lang.Object').implementation = function (obj1, obj2) {
            // 自定义操作
            console.log('list add obj1', obj1);
            console.log('list add obj2', obj2);
            // 调用原方法
            return this.add(obj1, obj2);
        };
        ArrayList.get.implementation = function (index) {
            var retVal = this.get(index);
            // 修改返回值
            console.log('list get ', retVal)
            return retVal;
        };
    })
}

```

### 28. hook  tls/ssl 的 key

对于用 wireshark 或者 tcpdump 抓包，需要解密 ssl 的时候使用

```
function startTLSKeyLogger(SSL_CTX_new, SSL_CTX_set_keylog_callback) {
    console.log("start----")
    function keyLogger(ssl, line) {
        console.log(new NativePointer(line).readCString());
    }
    const keyLogCallback = new NativeCallback(keyLogger, 'void', ['pointer', 'pointer']);
    Interceptor.attach(SSL_CTX_new, {
        onLeave: function(retval) {
            const ssl = new NativePointer(retval);
            const SSL_CTX_set_keylog_callbackFn = new NativeFunction(SSL_CTX_set_keylog_callback, 'void', ['pointer', 'pointer']);
            SSL_CTX_set_keylog_callbackFn(ssl, keyLogCallback);
        }
    });
}
startTLSKeyLogger(
    Module.findExportByName('libssl.so', 'SSL_CTX_new'),
    Module.findExportByName('libssl.so', 'SSL_CTX_set_keylog_callback')
)

```

### 29. hex 转 utf8 字符串

```
function hexToUtf8(hex) {
    try {
        return decodeURIComponent('%' + hex.match(/.{1,2}/g).join('%'));
    } catch (error) {
        return "hex[" + hex + "]";
    }
}

```

### 30. hook 通用算法

这个太长了就不占篇幅了，感兴趣私聊

xposed 相关
---------

xposed 的话，可整理成工具库的不多，所以就不单开文章梳理了，另外其实上面的所有 frida 操作，理论上用 xposed 都是可以实现的，只是写起来放不方便而已了

另外针对 so 层的 hook，xposed 原生是不支持的，所以得借助一些工具

### 1. 获取真实的 classloader

对于加壳的 app，需要用以下代码获取真实的 classloader，才能实际的 hook

```
public static void getRealClassLoader(XC_LoadPackage.LoadPackageParam lpparam) {
        // 针对加壳应用，获取真实的classloader
        XposedBridge.log("inner  => " + lpparam.processName);
        Class<?> ActivityThread = XposedHelpers.findClass("android.app.ActivityThread", lpparam.classLoader);
        XposedBridge.hookAllMethods(ActivityThread, "performLaunchActivity", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
                Object mInitialApplication = (Application) XposedHelpers.getObjectField(param.thisObject, "mInitialApplication");
                ClassLoader realClassLoader = (ClassLoader) XposedHelpers.callMethod(mInitialApplication, "getClassLoader");
                XposedBridge.log(TAG + ": " + "find classload is => " + realClassLoader.toString());
            }
        });
    }
public static void getRealClassLoader2(XC_LoadPackage.LoadPackageParam lpparam) {
        // 针对加壳应用，获取真实的classloader
        XposedBridge.log("inner  => " + lpparam.processName);
        XposedHelpers.findAndHookMethod(Application.class, "attach", Context.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                Context context = (Context) param.args[0];
                ClassLoader classLoader = context.getClassLoader();
                // 实际的代码逻辑
            }
        });
    }

```

### 2. 过掉调试模式检测

```
public static void anti_antiDebug(XC_LoadPackage.LoadPackageParam lpparam) {
        // 对抗反调试
        XposedBridge.log("inner  => " + lpparam.processName);
        XposedHelpers.findAndHookMethod(Debug.class, "isDebuggerConnected", Context.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                param.setResult(false);
            }
        });
    }

```

### 3. 对抗被杀死

```
public static void anti_killapp() {
        // 对抗被杀死
        XposedHelpers.findAndHookMethod(System.class, "exit", Context.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
            }
        });
    }

```

### 3. hook 多个 dex

```
public static void hookMutilDex(XC_LoadPackage.LoadPackageParam lpparam) {
        // hook多个dex
        XposedHelpers.findAndHookMethod(Application.class, "attach", Context.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                ClassLoader cl = ((Context) param.args[0]).getClassLoader();
                Class<?> hookclass = null;
                try {
                    hookclass = cl.loadClass("类名");
                } catch (Exception e) {
                    Log.e(TAG, "未找到类", e);
                    return;
                }
                XposedHelpers.findAndHookMethod(hookclass, "方法名", new XC_MethodHook() {
                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    }
                });
            }
        });
    }

```

### 4. 打印调用栈

```
private static void printStackTrace() {
        // 打印调用栈
        Throwable ex = new Throwable();
        StackTraceElement[] stackElements = ex.getStackTrace();
        for (int i = 0; i < stackElements.length; i++) {
            StackTraceElement element = stackElements[i];
            loge(TAG, element.getClassName() + "." + element.getMethodName() + "(" + element.getFileName() + ":" + element.getLineNumber() + ")");
        }
    }
private static void printStackTrace2() {
        // 打印调用栈
        loge(TAG, "Stack:" + new Throwable("Stack dump"));
    }

```

### 5. hook 指定字符串

```
private static void hookTargetString(XC_LoadPackage.LoadPackageParam lpparam, String targetStr) {
        // hook指定的字符串
        XposedHelpers.findAndHookMethod("android.widget.TextView", lpparam.classLoader, "setText", CharSequence.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                super.beforeHookedMethod(param);
                loge(TAG, param.args[0].toString());
                if (param.args[0].equals(targetStr)) {
                    printStackTrace(); // 打印调用栈
                }
            }
        });
    }

```

### 6. 点击事件监听

```
private static void hookClick(XC_LoadPackage.LoadPackageParam lpparam, String targetStr) {
        // 点击事件监听
        Class<?> clazz = XposedHelpers.findClass("android.view.View", lpparam.classLoader);
        XposedBridge.hookAllMethods(clazz, "performClick", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
                Object listenerInfoObject = XposedHelpers.getObjectField(param.thisObject, "mListenerInfo");
                Object mOnClickListenerObject = XposedHelpers.getObjectField(listenerInfoObject, "mOnClickListener");
                String callbackType = mOnClickListenerObject.getClass().getName();
                loge(TAG, callbackType);
            }
        });
    }

```

### 7. 遍历所有类的方法

```
public static void getClasssAllMethods(String ClassName) {
        // 遍历所有类下的所有方法
        XposedHelpers.findAndHookMethod(ClassLoader.class, "loadClass", String.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
                Class clazz = (Class) param.getResult();
                String clazzName = clazz.getName();
                //排除非包名的类
                if (clazzName.contains(ClassName)) {
                    Method[] mds = clazz.getDeclaredMethods();
                    for (int i = 0; i < mds.length; i++) {
                        final Method md = mds[i];
                        int mod = mds[i].getModifiers();
                        //去除抽象、native、接口方法
                        if (!Modifier.isAbstract(mod) && !Modifier.isNative(mod) && !Modifier.isAbstract(mod)) {
                            XposedBridge.hookMethod(mds[i], new XC_MethodHook() {
                                @Override
                                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                                    super.beforeHookedMethod(param);
                                    loge(TAG, md.toString());
                                }
                            });
                        }
                    }
                }
            }
        });
    }

```

### 8. 简单的过 xp 检测

```
public static void antiXposed() {
        // 过掉xp检测
        XposedHelpers.findAndHookMethod(StackTraceElement.class, "getClassName", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                String result = (String) param.getResult();
                if (result != null) {
                    if (result.contains("de.robv.android.xposed.")) {
                        param.setResult("l.m.p");
                         Log.i(TAG, "替换了，字符串名称 " + result);
                    } else if (result.contains("com.android.internal.os.ZygoteInit")) {
                        param.setResult("");
                    }
                }
                super.afterHookedMethod(param);
            }
        });
    }
    public static void antiXposedPlus() {
        // 对抗xposed检测
        XposedHelpers.findAndHookMethod(ClassLoader.class, "loadClass", String.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                if (param.args != null && param.args[0] != null && param.args[0].toString().startsWith("de.robv.android.xposed.")) {
                    // 改成一个不存在的类
                    param.args[0] = "l.p.m";
                }
                super.beforeHookedMethod(param);
            }
        });
//        XposedHelpers.findAndHookMethod(Method.class, "getModifiers", new XC_MethodHook() {
//            @Override
//            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
//                Log.i(TAG, "调用了");
////                Method method = (Method) param.thisObject;
////                String[] array = new String[]{"getDeviceId"};
////                String method_name = method.getName();
////                if (Arrays.asList(array).contains(method_name)) {
////                    modify = 0;
////                } else {
////                    modify = (int) param.getResult();
////                }
//
//                super.afterHookedMethod(param);
//            }
//        });
        XposedHelpers.findAndHookMethod(Modifier.class, "isNative", int.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                param.args[0] = modify;
                super.beforeHookedMethod(param);
            }
        });
        XposedHelpers.findAndHookMethod(BufferedReader.class, "readLine", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                String result = (String) param.getResult();
                if (result != null) {
                    if (result.contains("/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar")) {
                        param.setResult("");
                        new File("").lastModified();
                    }
                }
                super.afterHookedMethod(param);
            }
        });
    }

```

### 9. 字符串转 hex 的字符串

```
public static String stringToHexString(String s) {
        String str = "";
        for (int i = 0; i < s.length(); i++) {
            int ch = s.charAt(i);
            String s4 = Integer.toHexString(ch);
            str = str + s4;
        }
        return str;
    }

```

### 10. 加强版日志打印

```
public static void loge(String tag, String msg) {
        // 加强版日志，可以完整输出日志，不会被截断
        if (tag == null || tag.length() == 0 || msg == null || msg.length() == 0) {
            return;
        }
        int segmentSize = 3 * 1024;
        long length = msg.length();
        if (length <= segmentSize) {// 长度小于等于限制直接打印
            Log.i(tag, msg);
        } else {
            while (msg.length() > segmentSize) {// 循环分段打印日志
                String logContent = msg.substring(0, segmentSize);
                msg = msg.replace(logContent, "");
                Log.i(tag, logContent);
            }
            Log.i(tag, msg);// 打印剩余日志
        }
    }

```

### 11. 简单版的过无障碍检测

```
public static void BypassAccessibilityCheck(XC_LoadPackage.LoadPackageParam lpparam){
        XposedHelpers.findAndHookMethod("android.provider.Settings$Secure", lpparam.classLoader, "getString", ContentResolver.class, String.class, new XC_MethodHook() {
            protected void beforeHookedMethod(MethodHookParam methodHookParam) throws Throwable {
                if ("enabled_accessibility_services".equals(methodHookParam.args[1])) {
                    methodHookParam.setResult("");
                }
            }
        });
        XposedHelpers.findAndHookMethod("android.provider.Settings$Secure", lpparam.classLoader, "getInt", ContentResolver.class, String.class, new XC_MethodHook() {
            protected void beforeHookedMethod(MethodHookParam methodHookParam) throws Throwable {
                if ("accessibility_enabled".equals(methodHookParam.args[1])) {
                    methodHookParam.setResult(0);
                }
            }
        });
    }

```

### 12. hook 通用算法

同样的，这个太长了就不占篇幅了，感兴趣私聊吧

结语
--

技术交流、商务合作、技术交流群

扫码或者搜 ID：**geekbyte**

![](https://mmbiz.qpic.cn/mmbiz_jpg/l4m5icTfxSXVdsh5VoUbWiaqGic0IY8QrOqR727qdSQgVicfeHLQHperX9dV9Yv20BlxRFibWkkcb6e5Xy1SHk55wDQ/640?wx_fmt=jpeg)