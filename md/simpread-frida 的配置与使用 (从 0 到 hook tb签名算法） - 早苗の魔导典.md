> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [moe.me](https://moe.me/2021/08/16/frida%E7%9A%84%E4%BD%BF%E7%94%A8%E4%B8%8E%E9%85%8D%E7%BD%AE/)

> 概述在 app 逆向中常用 frida 作为 hook 框架对目标程序进程进行调试等操作，虽然关于 frida 的教程有不少，但似乎没有一篇从安装配置到实际使用的教程，于是便厚着脸皮写这一篇。

[](#概述 "概述")概述
--------------

在 app 逆向中常用 frida 作为 hook 框架对目标程序进程进行调试等操作，虽然关于 frida 的教程有不少，但似乎没有一篇从安装配置到实际使用的教程，于是便厚着脸皮写这一篇。

[frida GitHub 地址](https://github.com/frida)

[](#给太懒不看只想要一个api的 "给太懒不看只想要一个api的")给太懒不看只想要一个 api 的
----------------------------------------------------

目前只开放一个淘口令解析的 API，地址为 [https://api.57liuliu.com/taobao/api/sharepassword_query](https://api.57liuliu.com/taobao/api/sharepassword_query)

提交数据格式 JSON，内容为 `{"password":"淘口令"}` 示例如下

![](https://moe.me/images/pasted-12.png)

注：本 API 随时可能坏掉（

[](#安装frida "安装frida")安装 frida
------------------------------

### [](#1-安装Python "1.安装Python")1. 安装 Python

官方要求 Python3 版本且为 Python3.7 以上，建议直接安装 Python3.7.0  
[Python 3.7.0 下载](https://www.python.org/downloads/release/python-370/)

### [](#2-安装frida的python模块 "2.安装frida的python模块")2. 安装 frida 的 python 模块

如果已经将 Python 相关目录添加到了 PATH 中，则执行`pip install frida`，否则的话需要执行`python -m pip install frida`

同上方法，安装 frida-tools 模块：`pip install frida-tools`或者`python -m pip install frida-tools`

完成安装之后，打开 cmd 输入 frida，查看是否成功执行

![](https://moe.me/images/pasted-1.png)

### [](#3-在被调试的机器上安装frida-server "3.在被调试的机器上安装frida-server")3. 在被调试的机器上安装 frida-server

**被调试的机器必须 root**

前往 frida 的官方 github release 页面下载对应版本的 frida-server  
[frida release](https://github.com/frida/frida/releases)，下载时注意选择与被调试机器架构相同的版本（执行`adb shell getprop ro.product.cpu.abi`查询）

注意要下载的文件名为 frida-server-xxxxxx(版本)-android - 架构. xz

下载解压之后，使用命令`adb push frida-server在你电脑解压出来的位置 /data/local/tmp`将 frida-server 传输到目标机器中

接下来执行`adb shell`打开目标机器的 shell，执行如下命令给 frida-server 赋予执行权限并启动 frida-server

```
sudo su
cd /data/loca/tmp
chmod +x frida-server
./frida-server
```

完成以上步骤之后，不要关闭 adb shell 的窗口，在电脑上继续执行如下命令进行端口转发

```
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

全部完成之后，在电脑上执行`frida-ps -U`，正常情况下此时应该会列出当前被调试的机器上的进程列表。

![](https://moe.me/images/pasted-2.png)

当看到这一步的时候，说明 frida-server 已经成功安装并启动

下次使用时可以通过如下 bat 脚本快速启动 frida-server 并配置端口转发 (将其中 adb 目录替换实际情况)

```
"D:\Program Files\Nox64\bin\nox_adb" shell "nohup /data/local/tmp/frida-server &"
"D:\Program Files\Nox64\bin\nox_adb" forward tcp:27042 tcp:27042
"D:\Program Files\Nox64\bin\nox_adb" forward tcp:27043 tcp:27043
frida-ps -U
pause
```

[](#使用frida "使用frida")使用 frida
------------------------------

### [](#1-编写一个注入脚本 "1.编写一个注入脚本")1. 编写一个注入脚本

frida 使用的脚本语言为 Javascript，提供的 api 参考如下文档 [frida javascript api](https://frida.re/docs/javascript-api/)

新建一个 js 脚本文件，输入如下内容

```
function test(){
    console.log(Java.classFactory.loader);	
}
function main(){
    Java.perform(function(){	
        test();
    });
}
setImmediate(main);
```

通过如下命令，将拉起目标进程并挂起，将脚本注入进去并恢复目标进程，用这种方法可以避免部分进程的反调试导致无法注入。

> frida -U -f com.taobao.taobao -l a.js –no-pause

该命令中拉起的是手机淘宝客户端，注入 a.js 文件，并且在注入之后自动恢复目标进程。如果不加–no-pause 参数则需要之后输入 %resume 命令恢复目标进程。

正常情况下输出如下：

![](https://moe.me/images/pasted-3.png)

可以看到当前的 class loader 信息已经被打印出来了。

### [](#2-对目标类进行hook "2.对目标类进行hook")2. 对目标类进行 hook

使用 [jdax](https://github.com/skylot/jadx) 或其他程序对 apk 进行反编译并寻找要 hook 的类，在这里我们以 hook 淘宝的网络类以便展示其进行的所有由 mtop 类发送的 https 请求明文为例

首先 jdax 反编译目标 apk，找到要 hook 的类，本文中使用的手机淘宝 9.23.0 版本中 mtop 网络类位于

> mtopsdk.network.impl.b

![](https://moe.me/images/pasted-4.png)  
查看目标类构造函数的签名，可以知道他需要传入两个参数，分别为`mtopsdk.network.domain.Request`和`android.content.Context`类型

修改我们之前的注入脚本，将 test 函数中 console.log 修改为 hook 内容，如下

```
function test(){
    var ANetworkCallImpl = Java.use('mtopsdk.network.impl.b');
    ANetworkCallImpl.$init.overload('mtopsdk.network.domain.Request', 'android.content.Context').implementation = function(){
        console.log("\nANetworkCallImpl "+arguments[0])
        var ret = this.$init.apply(this, arguments);
        return ret
    }
}
function main(){
    Java.perform(function(){
        test();
    });
}
setImmediate(main);
```

![](https://moe.me/images/pasted-5.png)  
可以看到，所有 mtop sdk 发出的 https 请求明文在这里都可以看到了。

### [](#3-hook签名方法 "3.hook签名方法")3.hook 签名方法

通过某些不可描述的方法 (百度与 google)，我们知道了对参数进行签名的关键函数在

> com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative

函数声明如下

`public static native Object doCommandNative(int arg0, Object[] arg1);`

于是针对其编写 hook 脚本

```
function test(){
    Java.use("com.taobao.wireless.security.adapter.JNICLibrary").doCommandNative.implementation = function(m,n){
        var result = this.doCommandNative(m,n);
        for (var j = 0; j < arguments.length; j++) {
            console.log("arg[" + j + "]: " + arguments[j] + " => " + JSON.stringify(arguments[j]));
        }
        console.log("doCommandNative => ",m,n,result);
        return result;
    }
}

function main(){
    Java.perform(function(){
        test();
    });
}

setImmediate(main);
```

执行该脚本，发现如下返回

![](https://moe.me/images/pasted-6.png)  
报错找不到该类，nmd wsm.jpg

![](https://moe.me/images/pasted-7.png)

### [](#4-定位class-loader "4.定位class loader")4. 定位 class loader

仔细观察上面的报错，可以发现如下一条

> zip file “/data/app/com.taobao.taobao-0XVGcUBeC_rav8IvTjFXZw==/base.apk”

即当前的 class loader 为系统默认 class loader，而我们希望 hook 的目标类

> com.taobao.wireless.security.adapter.JNICLibrary

是由程序启动加载时动态加载进来的，所以系统默认的 class loader 中没有该类，这种情况下我们可以使用`Java.enumerateClassLoaders(callbacks)` 来遍历目前所有的 class loader 来找到目标类所在的 class loader。

```
function test(){
    Java.enumerateClassLoaders({
        onMatch: function (loader){
            try{
                if(loader.findClass("com.taobao.wireless.security.adapter.JNICLibrary")){
                    console.log("found com.taobao.wireless.security.adapter.JNICLibrary loader");
                    console.log(loader);
                }
            }catch(error){
            }
        },
        onComplete: function(){
            console.log("enumerateClassLoaders complete");
        }
    });
}
function main(){
    Java.perform(function(){
        setTimeout(() => {	
            test();
        }, 1000);	
    });
}
setImmediate(main);
```

执行结果如下，可见我们要寻找的目标类在 libsgmain.so 之中

![](https://moe.me/images/pasted-8.png)

找到目标类所在的 class loader 之后，可以通过修改`Java.classFactory.loader`的值，手动指定当前使用的 class loader 为前一步找到的 loader。

```
var loaderSwitched = false;
function test(){
    Java.enumerateClassLoaders({
        onMatch: function (loader){
            try{
                if(loader.findClass("com.taobao.wireless.security.adapter.JNICLibrary")){
                    console.log("found com.taobao.wireless.security.adapter.JNICLibrary loader");
                    console.log(loader);
                    Java.classFactory.loader = loader;
                    loaderSwitched = true;
                }
            }catch(error){
            }
        },
        onComplete: function(){
            console.log("enumerateClassLoaders complete");
        }
    });
}
function starthook(){
    if(!loaderSwitched){
        console.log("loader not switched, return");
        return;
    }
    Java.use("com.taobao.wireless.security.adapter.JNICLibrary").doCommandNative.implementation = function(m,n){
        var result = this.doCommandNative(m,n);
        for (var j = 0; j < arguments.length; j++) {
            console.log("arg[" + j + "]: " + arguments[j] + " => " + JSON.stringify(arguments[j]));
        }
        console.log("doCommandNative => ",m,n,result);
        return result;
    }
}
function main(){
    Java.perform(function(){
        setTimeout(() => {
            test();	
        }, 1000);
        setTimeout(() => {
            starthook();	
        }, 2000);
    });
}
setImmediate(main);
```

执行该脚本，返回如下

![](https://moe.me/images/pasted-9.png)  
可以看到目标函数已经被成功 hook 并且打印出了输入和返回值，观察打印的内容我们发现，调用 xsign 签名算法的时候第一个参数为 70102  
修改以上脚本使其只打印 arg[0] 为 70102 的部分，效果如下

![](https://moe.me/images/pasted-10.png)

其传入和传出的参数，正是我们提供签名的内容和签名出来的 x-sign。

[](#使用frida-rpc "使用frida-rpc")使用 frida-rpc
------------------------------------------

### [](#1-frida-rpc的配置 "1.frida-rpc的配置")1.frida-rpc 的配置

为了将我们发现的目标函数导出以便电脑其他程序调用，我们需要利用 frida 提供的 frida-rpc 框架

首先需要修改注入的脚本，将要导出的目标函数传递给 rpc.exports

```
var loaderSwitched = false;
function test(){
    Java.enumerateClassLoaders({
        onMatch: function (loader){
            try{
                if(loader.findClass("com.taobao.wireless.security.adapter.JNICLibrary")){
                    console.log("found com.taobao.wireless.security.adapter.JNICLibrary loader");
                    console.log(loader);
                    Java.classFactory.loader = loader;
                    loaderSwitched = true;
                }
            }catch(error){
            }
        },
        onComplete: function(){
            console.log("enumerateClassLoaders complete");
        }
    });
}
function main(){
    Java.perform(function(){
        setTimeout(() => {
            test();
        }, 1000);
    });
}
rpc.exports = {
    sign: function(func, input, data){
        if(loaderSwitched){
            var Integer = Java.use("java.lang.Integer");
            var Boolean = Java.use("java.lang.Boolean");
            var String = Java.use("java.lang.String");
            var argList = Java.array("Ljava.lang.Object;", 
                [
                    String.$new("21646297"),
                    String.$new(input), 
                    Boolean.$new(false),
                    Integer.$new(0),    
                    String.$new(func),  
                    String.$new(data),  
                    null,               
                    null,               
                    null,               
                    String.$new("r_1")  
                ]
            );
            var Map = Java.use('java.util.HashMap');
            var ret = Java.use("com.taobao.wireless.security.adapter.JNICLibrary").doCommandNative(70102, argList);	
            var args_map = Java.cast(ret, Map);
            var jsonRet = {"x-sgext":args_map.get("x-sgext").toString(), "x-umt":args_map.get("x-umt").toString(), "x-mini-wua":args_map.get("x-mini-wua").toString(), "x-sign":args_map.get("x-sign").toString()}
            return {"Success":true,"Data":jsonRet};
        }else{
            return {"Success":false,"Reason":"Not loaded"};
        }
    }
};
setImmediate(main);
```

同时我们还需要编写对应的 Python 脚本将上面的 JavaScript 脚本注入并用来调用导出的函数

```
import frida
from time import sleep

device = frida.get_device_manager().enumerate_devices()[-1] 
pid = device.spawn(["com.taobao.taobao"])   
session = device.attach(pid)    
spt = open("F:\\a.js", encoding="utf-8").read()
script = session.create_script(spt)
script.load() 
device.resume(pid) 
sleep(2)    
print(script.exports.sign("test", "test", "test"))
```

Python 执行该脚本，查看返回

![](https://moe.me/images/pasted-11.png)  
可以看到 Python 已经成功远程调用了我们导出的签名函数。

[](#其他说明 "其他说明")其他说明
--------------------

### [](#1-关于导出的签名函数需要的三个参数具体内容 "1.关于导出的签名函数需要的三个参数具体内容")1. 关于导出的签名函数需要的三个参数具体内容

hook 导出的签名函数  
`sign: function(func, input, data)`

> 第一个参数 func，值为所调用的 mtop api 名称，例如解析淘口令时 api 名称为 mtop.taobao.sharepassword.querypassword

> 第二个参数 input，值为请求 header 中参与签名的那部分（deviceId,utdid,ttid,extdata 等）经过特定排序组合起来的值，其排序方法为

```
def params2signdata(params):
    names = ["uid", "reqbiz-ext", "appKey", "data", "t", "api", "v", "sid", "ttid", "deviceId", "lat", "lng", "extdata", "x-features", "routerId", "placeId", "open-biz","mini-appkey", "req-appkey", "accessToken", "open-biz-data"]
    signStr = ""
    if "utdid" in params:
        signStr += params["utdid"]
    for name in names:
        if name == "data":
            signStr += "&" + hashlib.md5(params[name].encode("utf-8")).hexdigest()
            continue
        if name == "extdata" and name in params and params[name] == "":
            continue
        if name not in params or params[name] == None:
            params[name] = ""
        signStr += "&" + str(params[name])
    return signStr
```

> 第三个参数 data，值为类似 “pageId=xxx&pageName=xxx” 的标识目前所在页面的内容，其中 pageId 的值需要进行一次 urlencode

### [](#2-淘宝的反调试 "2.淘宝的反调试")2. 淘宝的反调试

淘宝运行起来之后会定期通过读取 /proc/{PID}/status 文件检查自己是否被调试，虽然目前不对其进行处理也能正常运行和 hook，但稳妥起见可以对这个读取行为进行 hook 让他无法通过这个检查到自己是否被调试，代码如下（这段代码来自 [r0tracer 项目](https://github.com/r0ysue/r0tracer)）：

```
var ByPassTracerPid = function () {
    var fgetsPtr = Module.findExportByName("libc.so", "fgets");
    var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
    Interceptor.replace(fgetsPtr, new NativeCallback(function (buffer, size, fp) {
        var retval = fgets(buffer, size, fp);
        var bufstr = Memory.readUtf8String(buffer);
        if (bufstr.indexOf("TracerPid:") > -1) {
            Memory.writeUtf8String(buffer, "TracerPid:\t0");
            console.log("tracerpid replaced: " + Memory.readUtf8String(buffer));
        }
        return retval;
    }, 'pointer', ['pointer', 'int', 'pointer']));
};
setImmediate(ByPassTracerPid);
```