> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [kevinspider.github.io](https://kevinspider.github.io/frida/frida-hook-so/)

> 感谢 frida envhttps://github.com/frida/frida-java-bridge/blob/master/lib/env.js IDA 判断 Thumb 指令集和 Arm ......

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084552.png)

[https://github.com/frida/frida-java-bridge/blob/master/lib/env.js](https://github.com/frida/frida-java-bridge/blob/master/lib/env.js)

*   IDA - Options - General - number of opcode bytes - 设置为 4
*   此时查看 IDA VIew 中 opcode 的长度, 如果出现 2 个字节和 4 个字节的, 说明为 thumb 指令集
*   如果都是 4 个字节的, 说明是 arm 指令集;
*   在 Thumb 指令集下, inline hook 的偏移地址需要进行 +1 操作;

通过 hook ArtMethod 的 RegisterNative 函数, 可以监控所有的静态注册和动态注册的 JNI 函数的地址;

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

function readStdString(str) {
    if ((str.readU8() & 1) === 1) { 
        return str.add(2 * Process.pointerSize).readPointer().readUtf8String();
    }
    return str.add(1).readUtf8String();
}

function attach(addr) {
    Interceptor.attach(addr, {
        onEnter: function (args) {
            this.arg0 = args[0]; 
        },
        onLeave: function (retval) {
            var modulemap = new ModuleMap()
            modulemap.update()
            var module = modulemap.find(retval)
            
            
            if (module != null) {
                console.log('<' + module.name + '> method_name =>',
                    prettyMethod(this.arg0, 1),
                    ',offset=>', ptr(retval).sub(module.base), ',module_name=>', module.name)
            } else {
                console.log('<anonymous> method_name =>', readStdString(string), ', addr =>', ptr(retval))
            }
        }
    });
}

function hook_RegisterNative() {
    var libart = Process.findModuleByName('libart.so')
    var symbols = libart.enumerateSymbols()
    for (var i = 0; i < symbols.length; i++) {
        if (symbols[i].name.indexOf('RegisterNative') > -1 && symbols[i].name.indexOf('ArtMethod') > -1 && symbols[i].name.indexOf('RuntimeCallbacks') < 0) {
            
            attach(symbols[i].address)
        }
    }

}

function main() {
    hook_RegisterNative()
}

setImmediate(main)
```

用于查看目标 module 是否被正常加载, 使用 `Process.enumerateModules()` 将当前加载的所有 so 文件打印出来

```
function hook_native(){
    var modules = Process.enumerateModules();
    for (var i in modules){
        var module = modules[i];
        console.log(module.name);
        if (module.name.indexOf("target.so") > -1 ){
            console.log(module.base);
        }
    }
}
```

```
function hook_module() {
    var baseAddr = Module.findBaseAddress("libnative-lib.so");
    console.log("baseAddr", baseAddr);
}
```

[](#通过导出函数名定位-native-方法 "通过导出函数名定位 native 方法")通过导出函数名定位 native 方法
-----------------------------------------------------------------

```
function hook_func_from_exports(){
    var add_c_addr = Module.findExportByName("libnative-lib.so", "add_c");
    console.log("add_c_addr is :",add_c_addr);
}
```

[](#通过-symbols-符号定位-native-方法 "通过 symbols 符号定位 native 方法")通过 symbols 符号定位 native 方法
-----------------------------------------------------------------------------------

```
function find_func_from_symbols() {
    var NewStringUTF_addr = null;
    var symbols = Process.findModuleByName("libart.so").enumerateSymbols();
    for (var i in symbols) {
        var symbol = symbols[i];
        if (symbol.name.indexOf("art") >= 0 &&
            symbol.name.indexOf("JNI") >= 0 &&
            symbol.name.indexOf("CheckJNI") < 0
        ){
            if (symbol.name.indexOf("NewStringUTF") >= 0) {
                console.log("find target symbols", symbol.name, "address is ", symbol.address);
                NewStringUTF_addr = symbol.address;
            }
        }
    }

    console.log("NewStringUTF_addr is ", NewStringUTF_addr);

    Interceptor.attach(NewStringUTF_addr, {
        onEnter: function (args) {
            console.log("args0",args[0])
            console.log("args0", args[0], hexdump(args[0]));
            console.log("args1", args[1], hexdump(args[1]));
            var env = Java.vm.tryGetEnv();
            if (env != null) {
                
                console.log("Memory readCstring is :", Memory.readCString(args[1]));
            }else{
                console.log("get env error");
            }
        },
        onLeave: function (returnResult) {
            console.log("result: ", Java.cast(returnResult, Java.use("java.lang.String")));
            var env = Java.vm.tryGetEnv();
            if (env != null) {
                var jstring = env.newStringUtf("修改返回值");
                returnResult.replace(ptr(jstring));
            }
        }
    })
}
```

[](#通过地址偏移-inline-hook-任意函数 "通过地址偏移 inline-hook 任意函数")通过地址偏移 inline-hook 任意函数
-----------------------------------------------------------------------------

```
function main(){
    
    var libnative_lib_addr = Module.findBaseAddress("libnative-lib.so");
    console.log("base module addr ->", libnative_lib_addr);
    if (libnative_lib_addr){
        var add_addr1 = Module.findExportByName("libnative-lib.so", "_Z5r0addii");
        var add_addr2 = libnative_lib_addr.add(0x94B2 + 1); 
        console.log(add_addr1);
        console.log(add_addr2);
    }

    
    var add1 = new NativeFunction(add_addr1, "int", ["int", "int"]);
    var add2 = new NativeFunction(add_addr2, "int", ["int", "int"]);

    console.log("add1 result is ->" + add1(10, 20));
    console.log("add2 result is ->" + add2(10, 20));

}

setImmediate(main);
```

*   `onEnter`: 函数 (args) : 回调函数, 给定一个参数 args, 用于读取或者写入参数作为 `NativePointer` 对象的指针;
*   `onLeave`: 函数 (retval) : 回调函数给定一个参数 retval, 该参数是包含原始返回值的 NativePointer 派生对象; 可以调用 `retval.replace(1234)` 以整数 1234 替换返回值, 或者调用`retval.replace(ptr("0x1234"))` 以替换为指针;
*   注意: `retval` 对象会在 `onLeave` 调用中回收, 因此不要将其存储在回调之外使用, 如果需要存储包含的值, 需要制作深拷贝, 如 `ptr(retval.toString())`

```
function find_func_from_exports() {
    var add_c_addr = Module.findExportByName("libnative-lib.so", "add_c");
    console.log("add_c_addr is :",add_c_addr);
    
    Interceptor.attach(add_c_addr,{
        
        onEnter: function (args) {
            console.log("add_c called");
            console.log("arg1:",args[0].toInt32());
            console.log("arg2", args[1].toInt32());
        },
        
        onLeave: function (returnValue) {
            console.log("add_c result is :", returnValue.toInt32());
            
            returnValue.replace(100);
        }
    })
}
```

```
function frida_Interceptor() {
    Java.perform(function () {
       
       
       var add_method = new NativeFunction(Module.findExportByName('libhello.so', 'c_getSum'), 
       'int',['int','int']);
       
       console.log("result:",add_method(1,2));
       
       Interceptor.replace(add_method, new NativeCallback(function (a, b) {
           
            return 123;
       }, 'int', ['int', 'int']));
       
       console.log("result:",add_method(1,2));
    });
}
```

通俗点说, inline hook 就是通过内存地址, 进行 hook;

```
function inline_hook() {
    var libnative_lib_addr = Module.findBaseAddress("libnative-lib.so");
    if (libnative_lib_addr) {
        console.log("libnative_lib_addr:", libnative_lib_addr);
        var addr_101F4 = libnative_lib_addr.add(0x102BC);
        console.log("addr_101F4:", addr_101F4);

        Java.perform(function () {
            Interceptor.attach(addr_101F4, {
                onEnter: function (args) {
                    console.log("addr_101F4 OnEnter :", this.context.PC, 
                                this.context.x1, this.context.x5, 
                                this.context.x10);
                },
                onLeave: function (retval) {
                     console.log("retval is :", retval) 
                }
            }
            )
        })
    }
}
```

```
new NativeFunction(address, returnType, argTypes[, options])
```

*   `address` : 函数地址
*   `returnType` : 指定返回类型
*   `argTypes` : 数组指定参数类型
*   类型可选: void, pointer, int, uint, long, ulong, char, uchar, float, double, int8, uint8, int16, int32, uint32, int64, uint64; 参照函数所需的 type 来定义即可;

```
function invoke_native_func() {
    var baseAddr = Module.findBaseAddress("libnative-lib.so");
    console.log("baseAddr", baseAddr);
    var offset = 0x0000A28C + 1;
    var add_c_addr = baseAddr.add(offset);
    var add_c_func = new NativeFunction(add_c_addr, "int", ["int","int"]);
    var result = add_c_func(1, 2);
    console.log(result);
}
Java.perform(function () {
    
    var base = Module.findBaseAddress("libnative-lib.so");
    
    var sub_834_addr = base.add(0x835) 
    
    var sub_834 = new NativeFunction(sub_834_addr, 'pointer', ['pointer']);
    
    var arg0 = Memory.alloc(10);
    ptr(arg0).writeUtf8String("123");
    var result = sub_834(arg0);
    console.log("result is :", hexdump(result));
})
```

`jni` 全部定在在 `/system/lib(64)/libart.so` 文件中, 通过枚举 `symbols` 筛选出指定的方法

```
function hook_libart() {
    var GetStringUTFChars_addr = null;

    
    var module_libart = Process.findModuleByName("libart.so");
    var symbols = module_libart.enumerateSymbols();
    for (var i = 0; i < symbols.length; i++) {
        var name = symbols[i].name;
        if ((name.indexOf("JNI") >= 0) 
            && (name.indexOf("CheckJNI") == -1) 
            && (name.indexOf("art") >= 0)) {
            if (name.indexOf("GetStringUTFChars") >= 0) {
                console.log(name);
                
                GetStringUTFChars_addr = symbols[i].address;
            }
        }
    }

    Java.perform(function(){
        Interceptor.attach(GetStringUTFChars_addr, {
            onEnter: function(args){
                
                
                console.log("native args[1] is :",Java.vm.getEnv().getStringUtfChars(args[1],null).readCString());
                console.log('GetStringUTFChars onEnter called from:\n' +
                    Thread.backtrace(this.context, Backtracer.FUZZY)
                    .map(DebugSymbol.fromAddress).join('\n') + '\n');
                
                
            }, onLeave: function(retval){
                
                console.log("GetStringUTFChars onLeave : ", ptr(retval).readCString());
            }
        })
    })
}
```

`/system/lib(64)/libc.so` 导出的符号没有进行 `namemanline` , 直接过滤筛选即可

```
var pthread_create_addr = null;



var symbols = Process.findModuleByName("libc.so").enumerateSymbols();
for (var i = 0; i < symbols.length; i++){
    if (symbols[i].name === "pthread_create"){
        
        
        pthread_create_addr = symbols[i].address;
    }
}

Interceptor.attach(pthread_create_addr,{
    onEnter: function(args){
        console.log("args is ->" + args[0], args[1], args[2],args[3]);
    },
    onLeave: function(retval){
        console.log(retval);
    }
});
```

`libc.so` 中方法替换

```
function main() {
    
    
    

    var pthread_create_addr = null;
    var symbols = Process.getModuleByName("libc.so").enumerateSymbols();
    for (var i = 0; i < symbols.length; i++) {
        var symbol = symbols[i];
        if (symbol.name === "pthread_create") {
            pthread_create_addr = symbol.address;
            console.log("pthread_create name is ->", symbol.name);
            console.log("pthread_create address is ->", pthread_create_addr);
        }
    }

    Java.perform(function(){
        
        var pthread_create = new NativeFunction(pthread_create_addr, 'int', ['pointer', 'pointer','pointer','pointer'])
        Interceptor.replace(pthread_create_addr,new NativeCallback(function (a0, a1, a2, a3) {
            var result = null;
            var detect_frida_loop = Module.findExportByName("libnative-lib.so", "_Z17detect_frida_loopPv");
            console.log("a0,a1,a2,a3 ->",a0,a1,a2,a3);
            if (String(a2) === String(detect_frida_loop)) {
                result = 0;
                console.log("阻止frida反调试启动");
            } else {
                result = pthread_create(a0,a1,a2,a3);
                console.log("正常启动");
            }
            return result;
        }, 'int', ['pointer', 'pointer','pointer','pointer']));
    })
}
```

```
Interceptor.attach(f, {
  onEnter: function (args) {
    console.log('RegisterNatives called from:\n' +
        Thread.backtrace(this.context, Backtracer.ACCURATE)
        .map(DebugSymbol.fromAddress).join('\n') + '\n');
  }
});
```

[](#安装 "安装")安装
--------------

`jnitrace`: [https://github.com/chame1eon/jnitrace](https://github.com/chame1eon/jnitrace)

```
python` : `pip install jnitrace
```

[](#基础用法 "基础用法")基础用法
--------------------

`ndk` 开发是没有办法脱离 `[libc.so](http://libc.so)` 和 `[libart.so](http://libart.so)` 进行开发, 所以只要降维打击, 通过 `trace` 的方式就可以监控到 `so` 层

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084549.jpg)

### [](#启动命令 "启动命令")启动命令

```
jnitrace [options] -l libname packagename
```

例如: `jnitrace -l [libnative-lib.so](http://libnative-lib.so) com.example.myapplication`

### [](#必要参数 "必要参数")**必要参数**

*   `-l libname` : 指定要`trace`的`.so`文件, 可以同时`trace`多个`.so`文件, 直接使用 `*`来`trace`所有的`.so`文件; 如: `-l libnative-lib.so -l libanother-lib.so` or `-l *`
*   `packagename` : 指定要`trace`的`package name`

### [](#可选参数 "可选参数")**可选参数**

*   `-m`: 指定是`spawn`还是`attach`
*   `-b`: 指定是`fuzzy`还是`accurate`
*   `-i <regex>`: 指定一个正则表达式来过滤出方法名, 例如: `-i Get -i RegisterNatives` 就只会打印出名字里包含`Get`或者`RegisterNatives`的`JNI methods`
*   `-e <regex>`和`i`相反，同样通过正则表达式来过滤，但这次会将指定的内容忽略掉
*   `-I <string>`trace 导出的方法，jnitrace 认为导出的函数应该是从 Java 端能够直接调用的函数，所以可以包括使用 RegisterNatives 来注册的函数，例如`I stringFromJNI -I nativeMethod([B)V`，就包括导出名里有 stringFromJNI，以及使用 RegisterNames 来注册，并带有 nativeMethod([B)V 签名的函数。
*   `-o path/output.json`，导出输出到文件里。
*   `-p path/to/script.js`，用于在加载 jnitrace 脚本之前将指定路径的 Frida 脚本加载到目标进程中，这可以用于在 jnitrace 启动之前对抗反调试。
*   `-a path/to/script.js`，用于在加载 jnitrace 脚本之后将指定路径的 Frida 脚本加载到目标进程中
*   `-ignore-env`，不打印所有的 JNIEnv 函数
*   `-ignore-vm`，不打印所有的 JavaVM 函数

### [](#启动方式 "启动方式")启动方式

默认使用 `spawn` 启动, 可以通过 `-m attach` 设置通过 `attach` 启动

```
jnitrace -m attach -l[libnative-lib.so](http://libnative-lib.so) com.kevin.demoso1
```

### [](#设置回溯器 "设置回溯器")设置回溯器

默认情况下使用 `accurate`的精确模式来进行回溯, 可以通过 `-b fuzzy` 修改为模糊模式

```
jnitrace -l [libnative-lib.so](http://libnative-lib.so) -b fuzzy com.kevin.demoso1
```

### [](#监控指定规则的方法 "监控指定规则的方法")监控指定规则的方法

用于指定应该跟踪的方法名, 该选项可以多次提供;

```
jnitrace -l libnative-lib.so -i RegisterNatives com.kevin.demoso1
```

只过滤出`RegisterNatives`相关的内容

### [](#忽略指定规则的方法 "忽略指定规则的方法")**忽略指定规则的方法**

用于指定在跟踪中应被忽略的方法名, 这个选项可以被多次提供;

忽略以`Find`开头的所有方法;

```
jnitrace -l libnative-lib.so -e ^Find com.kevin.demoso
```

[](#jnitace-计算偏移地址 "jnitace 计算偏移地址")jnitace 计算偏移地址
--------------------------------------------------

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084555.png)

`0x8e4f3b1` 是方法 `initSN` 方法的绝对地址

`0xd8e4e000` 是 `[libmyjni.so](http://libmyjni.so)` 基地址

使用使用 `initSN()V`的绝对地址 `0xd8e4f3b1` 减去 `[libmyjni.so](http://libmyjni.so)` 的基地址 `0xd8e4e000` , 得到偏移 `0x13B1`

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084551.png)

`g` 进行跳转到 `0x13B1` 即可进入方法

文档地址: [https://frida.re/docs/frida-trace/](https://frida.re/docs/frida-trace/)

[](#options "options")options
-----------------------------

```
Usage: frida-trace [options] target

 Options:
   --version             show program's version number and exit
   -h, --help            show this help message and exit
   -D ID, --device=ID    connect to device with the given ID
   -U, --usb             connect to USB device
   -R, --remote          connect to remote frida-server
   -H HOST, --host=HOST  connect to remote frida-server on HOST
   -f FILE, --file=FILE  spawn FILE
   -F, --attach-frontmost     attach to frontmost application
   -n NAME, --attach-name=NAME     attach to NAME
   -p PID, --attach-pid=PID     attach to PID
   --stdio=inherit|pipe      stdio behavior when spawning (defaults to “inherit”)
   --runtime=duk|v8          script runtime to use (defaults to “duk”)
   --debug                   enable the Node.js compatible script debugger
   -I MODULE, --include-module=MODULE       include MODULE
   -X MODULE, --exclude-module=MODULE       exclude MODULE
   -i FUNCTION, --include=FUNCTION    include FUNCTION
   -x FUNCTION, --exclude=FUNCTION   exclude FUNCTION
   -a MODULE!OFFSET, --add=MODULE!OFFSET    add MODULE!OFFSET
   -T, --include-imports    include program's imports
   -t MODULE, --include-module-imports=MODULE      include MODULE imports
   -m OBJC_METHOD, --include-objc-method=OBJC_METHOD    include OBJC_METHOD
   -M OBJC_METHOD, --exclude-objc-method=OBJC_METHOD    exclude OBJC_METHOD
   -s DEBUG_SYMBOL, --include-debug-symbol=DEBUG_SYMBOL    include DEBUG_SYMBOL
   -q, --quiet           do not format output messages
   -d, --decorate        Add module name to generated onEnter log statement
   -o OUTPUT, --output=OUTPUT    dump messages to file
```

[](#基础使用 "基础使用")基础使用
--------------------

```
frida-trace [options] packagename
```

### [](#启动模式 "启动模式")启动模式

默认使用 `attach` 模式, 可以指定 `-f packageName` 使用 `spawn` 模式启动

```
frida-trace -U -i strcmp -f com.gdufs.xman
```

### [](#文件输出 "文件输出")文件输出

```
frida-trace -U -i "strcmp" -f com.gdufs.xman -o xman.json
```

`-o filepath` 指定输出的文件路径, 方便内容过多时进行查看

### [](#trace-任意-function "trace 任意 function")trace 任意 function

```
frida-trace -U -i "strcmp" com.example.demoso1
```

### [](#trace-任意-module "trace 任意 module")trace 任意 module

```
frida-trace -U -I "libnative-lib.so" com.example.demoso1
```

### [](#根据地址进行-trace "根据地址进行 trace")根据地址进行 trace

```
frida-trace -U -a "libnative-lib.so!0x9281" com.example.demoso1
```

[](#批量-trace "批量 trace")批量 trace
--------------------------------

源码地址: [https://github.com/Pr0214/trace_natives](https://github.com/Pr0214/trace_natives)

ps: 需要切换到 frida14 版本

1. 将 traceNatives.py 丢进 IDA plugins 目录中

*   在 ida 的 python console 中运行如下命令即可找到 plugins 目录：`os.path.join(idaapi.get_user_idadir(), "plugins")`

2.IDA 中，Edit-Plugins-traceNatives –> IDA 输出窗口就会显示如下字眼：**使用方法如下： frida-trace -UF -O C:\Users\Lenovo\Desktop\2021\mt\libmtguard.txt**

下载地址: [https://github.com/lasting-yang/frida_hook_libart](https://github.com/lasting-yang/frida_hook_libart)

[](#hook-art "hook art")hook art
--------------------------------

```
frida -U --no-pause -f package_name -l hook_art.js
```

[](#hook-RegisterNatives "hook_RegisterNatives")hook_RegisterNatives
--------------------------------------------------------------------

```
frida -U --no-pause -f package_name -l hook_RegisterNatives.js
```

[](#hook-artmethod "hook_artmethod")hook_artmethod
--------------------------------------------------

### [](#init-libext-first-time "init libext first time")init libext first time

```
adb push lib/libext64.so /data/local/tmp/libext64.so
adb push lib/libext.so /data/local/tmp/libext.so
adb shell su -c "cp /data/local/tmp/libext64.so /data/app/libext64.so"
adb shell su -c "cp /data/local/tmp/libext.so /data/app/libext.so"
adb shell su -c "chown 1000.1000 /data/app/libext*.so"
adb shell su -c "chmod 777 /data/app/libext*.so"
adb shell su -c "ls -al /data/app/libext*"
```

### [](#use-hook-artmethod-js "use hook_artmethod.js")use hook_artmethod.js

```
frida -U --no-pause -f package_name -l hook_artmethod.js

frida -U --no-pause -f package_name -l hook_artmethod.js > hook_artmethod.log
```

[](#frida-fart-hook "frida-fart-hook")frida-fart-hook
-----------------------------------------------------

首先拷贝 fart.so 和 fart64.so 到 / data/app 目录下，并使用 chmod 777 设置好权限, 然后就可以使用了。

如果目标 app 没有 sdcard 权限则需要手动添加; 或者可以修改 frida_fart_hook.js 中的源码, 将 savepath 改为 `/data/data/应用包名/`;

该 frida 版 fart 是使用 hook 的方式实现的函数粒度的脱壳，仅仅是对类中的所有函数进行了加载，但依然可以解决绝大多数的抽取保护

需要以 spawn 方式启动 app，等待 app 进入 Activity 界面后，执行 fart() 函数即可。如 app 包名为 com.example.test, 则

`frida -U -f com.example.test -l frida_fart_hook.js --no-pause` ，然后等待 app 进入主界面, 执行 fart()

高级用法：如果发现某个类中的函数的 CodeItem 没有 dump 下来，可以调用 dump(classname), 传入要处理的类名，完成对该类下的所有函数体的 dump,dump 下来的函数体会追加到 bin 文件当中。

`frida api` 写入文件

```
function writeFile(){
  var file = new File("/sdcard/reg.dat", "w");
  file.write("content from frida");
  file.flush();
  file.close();
}
```

`frida 定义 NativeFunction` 写入文件

```
function writeFileNative(){
  var addr_fopen = Module.findExportByName("libc.so", "fopen");
  var addr_fputs = Module.findExportByName("libc.so", "fputs");
  var addr_fclose = Module.findExportByName("libc.so", "fclose");
  
  
  var fopen = new NativeFunction(addr_fopen, "pointer", ["pointer", "pointer"]);
  var fputs = new NativeFunction(addr_fputs, "int", ["pointer", "pointer"]);
  var fclose = new NativeFunction(addr_fclose, "int", ["pointer"]);

  
  
  var filename = Memory.allocUtf8String("/sdcard/reg.dat");
  var open_mode = Memory.allocUtf8String("w");
  var file = fopen(filename, open_mode);
  
  var buffer = Memory.allocUtf8String("content from frida");
  var result = fputs(buffer, file);
  console.log("fputs ret: ", result);

  
  fclose(file);
  
}
```

```
function readStdString(str){
  var isTiny = (str.readU8 & 1) === 0;
  if (isTiny){
    return str.add(1).readUtf8String();
  }
  return str.add(2 * Process.pointerSize).readPointer().readUtf8String();
}



function writeStdString(str, content){
  var isTiny = (str.readU8() & 1) === 0;
  if (isTiny){
    str.add(1).writeUtf8String(content);
  }else{
    str.add(2 * Process.pointerSize).readPointer().writeUtf8String(content);
  }
}
```

```
var dlopen = Module.findExportByName(null, "dlopen");
console.log(dlopen);
if(dlopen != null){
    Interceptor.attach(dlopen,{
        onEnter: function(args){
            var soName = args[0].readCString();
            console.log(soName);
            if(soName.indexOf("libc.so") != -1){
                this.hook = true;
            }
        },
        onLeave: function(retval){
            if(this.hook) { 
                dlopentodo();
            };
        }
    });
}


var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
console.log(android_dlopen_ext);
if(android_dlopen_ext != null){
    Interceptor.attach(android_dlopen_ext,{
        onEnter: function(args){
            var soName = args[0].readCString();
            console.log(soName);
            if(soName.indexOf("libc.so") != -1){
                this.hook = true;
            }
        },
        onLeave: function(retval){
            if(this.hook) {
                dlopentodo();
            };
        }
    });
}

function dlopentodo(){
    
}
```

```
function replaceKILL(){
  var kill_addr = Module.findExportByName("libc.so","kill");
  Interceptor.replace(kill_addr, new NativeCallback(function(arg0, arg1){
    console.log("arg0=> ", arg0);
    console.log("arg1=> ", arg1);
    console.log('libc.so!kill called from:\n' +
        Thread.backtrace(this.context, Backtracer.ACCURATE)
        .map(DebugSymbol.fromAddress).join('\n') + '\n');
  },"int",["int","int"]))
}
```

```
function get_call_function() {
    var call_function_addr = null;
    var symbols = Process.getModuleByName("linker").enumerateSymbols();
    for (var m = 0; m < symbols.length; m++) {
        if (symbols[m].name == "__dl__ZL13call_functionPKcPFviPPcS2_ES0_") {
            call_function_addr = symbols[m].address;
            console.log("found call_function_addr => ", call_function_addr)
            hook_call_function(call_function_addr)
        }
    }
}

function hook_call_function(_call_function_addr){
    console.log("hook call function begin!hooking address :=>",_call_function_addr)
    Interceptor.attach(_call_function_addr,{
        onEnter:function(args){
            if(args[2].readCString().indexOf("base.odex")<0){
                console.log("============================")
                console.log("function_name =>",args[0].readCString())
                var soPath = args[2].readCString()
                console.log("so path : =>",soPath)
                var soName = soPath.split("/").pop();
                console.log("function offset =>","0x"+(args[1]-Module.findBaseAddress(soName)).toString(16))
                console.log("============================")
            }
        },onLeave:function(retval){
        }
    })
}

setImmediate(get_call_function)

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
                    this.function_type = ptr(args[3]).readCString(), 
                    this.so_path = ptr(args[5]).readCString();
                    var strs = new Array(); 
                    strs = this.so_path.split("/"); 
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

document: [https://github.com/lasting-yang/frida_dump](https://github.com/lasting-yang/frida_dump)

[](#frida-dump-so "frida dump so")frida dump so
-----------------------------------------------

```
function dump_so(so_name) {
    Java.perform(function () {
        var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
        var dir = currentApplication.getApplicationContext().getFilesDir().getPath();
        var libso = Process.getModuleByName(so_name);
        console.log("[name]:", libso.name);
        console.log("[base]:", libso.base);
        console.log("[size]:", ptr(libso.size));
        console.log("[path]:", libso.path);
        var file_path = dir + "/" + libso.name + "_" + libso.base + "_" + ptr(libso.size) + ".so";
        var file_handle = new File(file_path, "wb");
        if (file_handle && file_handle != null) {
            Memory.protect(ptr(libso.base), libso.size, 'rwx');
            var libso_buffer = ptr(libso.base).readByteArray(libso.size);
            file_handle.write(libso_buffer);
            file_handle.flush();
            file_handle.close();
            console.log("[dump]:", file_path);
        }
    });
}
```

[](#frida-dump-dex "frida dump dex")frida dump dex
--------------------------------------------------

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

[](#frida-dump-dex-class "frida dump dex class")frida dump dex class
--------------------------------------------------------------------

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

function load_all_class() {
    if (Java.available) {
        Java.perform(function () {

            var DexFileclass = Java.use("dalvik.system.DexFile");
            var BaseDexClassLoaderclass = Java.use("dalvik.system.BaseDexClassLoader");
            var DexPathListclass = Java.use("dalvik.system.DexPathList");

            Java.enumerateClassLoaders({
                onMatch: function (loader) {
                    try {
                        var basedexclassloaderobj = Java.cast(loader, BaseDexClassLoaderclass);
                        var pathList = basedexclassloaderobj.pathList.value;
                        var pathListobj = Java.cast(pathList, DexPathListclass)
                        var dexElements = pathListobj.dexElements.value;
                        for (var index in dexElements) {
                            var element = dexElements[index];
                            try {
                                var dexfile = element.dexFile.value;
                                var dexfileobj = Java.cast(dexfile, DexFileclass);
                                console.log("dexFile:", dexfileobj);
                                const classNames = [];
                                const enumeratorClassNames = dexfileobj.entries();
                                while (enumeratorClassNames.hasMoreElements()) {
                                    var className = enumeratorClassNames.nextElement().toString();
                                    classNames.push(className);
                                    try {
                                        loader.loadClass(className);
                                    } catch (error) {
                                        console.log("loadClass error:", error);
                                    }
                                }
                            } catch (error) {
                                console.log("dexfile error:", error);
                            }
                        }
                    } catch (error) {
                        console.log("loader error:", error);
                    }
                },
                onComplete: function () {

                }
            })
            console.log("load_all_class end.");
        });
    }
}
var dex_maps = {};

function print_dex_maps() {
    for (var dex in dex_maps) {
        console.log(dex, dex_maps[dex]);
    }
}

function dump_dex() {
    load_all_class();

    for (var base in dex_maps) {
        var size = dex_maps[base];
        console.log(base);

        var magic = ptr(base).readCString();
        if (magic.indexOf("dex") == 0) {
            var process_name = get_self_process_name();
            if (process_name != "-1") {
                var dex_path = "/data/data/" + process_name + "/files/" + base.toString(16) + "_" + size.toString(16) + ".dex";
                console.log("[find dex]:", dex_path);
                var fd = new File(dex_path, "wb");
                if (fd && fd != null) {
                    var dex_buffer = ptr(base).readByteArray(size);
                    fd.write(dex_buffer);
                    fd.flush();
                    fd.close();
                    console.log("[dump dex]:", dex_path);

                }
            }
        }
    }
}

function hook_dex() {
    var libart = Process.findModuleByName("libart.so");
    var addr_DefineClass = null;
    var symbols = libart.enumerateSymbols();
    for (var index = 0; index < symbols.length; index++) {
        var symbol = symbols[index];
        var symbol_name = symbol.name;
        
        
        if (symbol_name.indexOf("ClassLinker") >= 0 &&
            symbol_name.indexOf("DefineClass") >= 0 &&
            symbol_name.indexOf("Thread") >= 0 &&
            symbol_name.indexOf("DexFile") >= 0) {
            console.log(symbol_name, symbol.address);
            addr_DefineClass = symbol.address;
        }
    }

    console.log("[DefineClass:]", addr_DefineClass);
    if (addr_DefineClass) {
        Interceptor.attach(addr_DefineClass, {
            onEnter: function (args) {
                var dex_file = args[5];
                
                
                var base = ptr(dex_file).add(Process.pointerSize).readPointer();
                var size = ptr(dex_file).add(Process.pointerSize + Process.pointerSize).readUInt();

                if (dex_maps[base] == undefined) {
                    dex_maps[base] = size;
                    console.log("hook_dex:", base, size);
                }
            },
            onLeave: function (retval) {}
        });
    }

}

var is_hook_libart = false;

function hook_dlopen() {
    Interceptor.attach(Module.findExportByName(null, "dlopen"), {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                
                if (path.indexOf("libart.so") >= 0) {
                    this.can_hook_libart = true;
                    console.log("[dlopen:]", path);
                }
            }
        },
        onLeave: function (retval) {
            if (this.can_hook_libart && !is_hook_libart) {
                hook_dex();
                is_hook_libart = true;
            }
        }
    })

    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                
                if (path.indexOf("libart.so") >= 0) {
                    this.can_hook_libart = true;
                    console.log("[android_dlopen_ext:]", path);
                }
            }
        },
        onLeave: function (retval) {
            if (this.can_hook_libart && !is_hook_libart) {
                hook_dex();
                is_hook_libart = true;
            }
        }
    });
}


setImmediate(hook_dex);
```

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084554.jpg)

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-84550.jpg)

[](#hook-so-readPointer "hook so readPointer()")hook so readPointer()
---------------------------------------------------------------------

```
Java.perform(function () {
        var libc_addr = Process.findModuleByName("libc.so").base;
        console.log("libc address is " + libc_addr);
        
        console.log(libc_addr.readByteArray(0x10));
        
        console.log("pointer size", Process.pointerSize);
        console.log("readPointer() is " + libc_addr.readPointer());
        console.log("Memory.readPointer()" + Memory.readPointer(libc_addr.add(Process.pointerSize)));
    })
```

[](#hook-so-writePointer "hook so writePointer()")hook so writePointer()
------------------------------------------------------------------------

```
Java.perform(function () {
        var libc_addr = Process.findModuleByName("libc.so").base;
        console.log("libc_addr : " + libc_addr);
        
        const r = Memory.alloc(4);
        
        r.writePointer(libc_addr);
        
        var buffer = Memory.readByteArray(r, 4);
        console.log(buffer);
    })
```

[](#hook-so-readS32-readU32 "hook so readS32(), readU32()")hook so readS32(), readU32()
---------------------------------------------------------------------------------------

从指定内存地址读取有符号或者无符号 8/16/21/etc 或浮点数 / 双精度值, 并将其作为数字返回;

```
Java.perform(function () {
        var libc_addr = Process.findModuleByName("libc.so").base;
        console.log(hexdump(libc_addr));
        console.log(libc_addr.readS32(), (libc_addr.readS32()).toString(16));
        console.log(libc_addr.readU32(), (libc_addr.readU32()).toString(16));
    })
```

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084553.png)

[](#hook-so-writeS32-writeU32 "hook so writeS32(), writeU32()")hook so writeS32(), writeU32()
---------------------------------------------------------------------------------------------

将有符号或无符号 8/16/32 / 等或浮点数 / 双精度值写入此内存位置

```
Java.perform(function () {
        
        const r = Memory.alloc(4);
        r.writeS32(0x12345678);
        console.log(r.readByteArray(0x10));
    })

<!--  0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  78 56 34 12 7d 00 00 00 98 c0 bb a8 7d 00 00 00  xV4.}.......}...
 -->
```

[](#hook-so-readByteArray-writeByteArray "hook so readByteArray(), writeByteArray()")hook so readByteArray(), writeByteArray()
------------------------------------------------------------------------------------------------------------------------------

```
Java.perform(function () {
        
        var arr = [ 0x72, 0x6F, 0x79, 0x73, 0x75, 0x65];
       
        var r = Memory.alloc(arr.length);
        
        r.writeByteArray(arr);
        
        console.log("memory readbyteArray: ")
        console.log(r.readByteArray(arr.length));
        console.log(Memory.readByteArray(r, arr.length));
    })
```

[](#hook-so-readCString-writeUtf8String "hook so readCString(), writeUtf8String()")hook so readCString(), writeUtf8String()
---------------------------------------------------------------------------------------------------------------------------

```
Java.perform(function () {
        
        var r = Memory.allocUtf8String("你好,世界");
        
        console.log(hexdump(r));
        console.log(r.readCString());
        
        r.writeUtf8String("Hello,World");
        console.log(hexdump(r));
        console.log(r.readCString())
    })
```

```
var arg1Ptr = Java.vm.getEnv().getByteArrayElements(this.arg1, null) 

console.log("arg1Ptr",hexdump(arg1Ptr));

console.log("arg1Ptr",arg1Ptr.readCString());
```

1.  将 ida 中的 android-server 推入到手机中 `adb push /Applications/IDA\ Pro\ 7.0/ida.app/Contents/MacOS/dbgsrv/android_server /data/local/tmp/as` android_server 负责调试 32 位的 app, android_server64 负责调试 64 位的 app, 改名为 as 可以防止部分 android_server 名称检测
2.  给 android_server 增加权限

```
adb shell
su
chmod +x /data/local/tmp/as
```

1.  进行端口转发 `adb forward tcp:11678 tcp:11678`
2.  启动 android_server 并指定端口为 11678

```
adb shell
su
/data/local/tmp/as -p11678
```

1.  调试之前先注入 frida
2.  新建一个 ida 界面
3.  Debugger - Remote ARMLinux/Android debugger
4.  hostname: localhost; Port: 11678; 勾选 Save network settings as default
5.  frida 打印出目标 function 最终的地址, ida 中 g 到目标 function 地址, 查看是否是 thumb 指令集, 如果是 thumb 则 option + g, 将 T 修改为 1, 再按 c;
6.  File - Script file - 读取 ida trace 脚本, 需要更改目标 so 文件和 function 的起始地址和结束地址; 读取之后会出现断点; 快捷键: `option F7`;
7.  frida 主动调用脚本, 检测断点是否触发
8.  断点检测正常后, ida 中执行 starthook() 命令, 进行 hook 操作; 执行`suspend_other_thread()`挂起其他线程 (可选择)
9.  ida 中 Debugger - tracing - tracing options - 设置 Trace file 路径 和 取消 Trace over debugger segments 的勾选
10.  ida 中 Debugger - tracing - 勾选 instruction tracing
11.  frida 主动调用触发断点
12.  ida 点击运行按钮进行执行, 此时 ida 中黄色部分为已经执行完的指令, 点击 Debugger -tracing - tracing window 可以看到当前执行进度;

修改路径下的文件 : /Applications/IDA Pro 7.0/ida.app/Contents/MacOS/python/init.py

```
lib_dynload = os.path.join(
    sys.executable,
    IDAPYTHON_DYNLOAD_BASE,
    "python", "lib", "python2.7", "lib-dynload")


sys.path.insert(0, "/Users/zhangyang/anaconda3/envs/py2/lib/python2.7/site-packages")
```

在 pycharm 中开启调试

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-84553.png)

在需要调试的脚本中添加断点

```
import pydevd_pycharm

pydevd_pycharm.settrace('localhost', port=12345, stdoutToServer=True, stderrToServer=True)
```

在 idapython 的`__init__.py`文件中添加自有的 python 路径;

先在 pycharm 中打开调试监听, 在 ida 中运行要调试的脚本即可;

1.  进入目录 `~/androidFxxk/idaTools/jni_helper`
2.  `java -jar JadxFindJNI/JadxFindJNI.jar <apk.path> <output.json>`
3.  ida 中`Script File`运行 `jni_help`脚本, 路径 `~/androidFxxk/idaTools/jni_helper/ida/jni_helper.py`
4.  导入刚才生成的 `output.json` 文件即可自动识别

给应用提供 sdcard 权限, 方便将 dump 下来的内容直接写入 sdcard 中;

1.  `adb shell dumpsys window | grep mCurrentFocus` 查看当前 app 的包名
2.  根据包名查看进程 pid

```
✔  adb shell
sailfish:/ $ su
sailfish:/ 
u0_a136      29251   641 0 16:10:14 ?     00:00:00 com.kevin.sotest
root         29845 29827 0 16:17:46 pts/3 00:00:00 grep com.kevin.sotest
```

1.  通过对应进程 id 和 so 名称, 查看其在虚拟地址空间中的映射

```
sailfish:/ 
7593f4a000-7593f7a000 r-xp 00000000 103:13 1180186                       /data/app/com.kevin.sotest-Wmu_QuZYJCLa0CPho2HTHw==/lib/arm64/libnative-lib.so
7593f7b000-7593f7f000 r--p 00030000 103:13 1180186                       /data/app/com.kevin.sotest-Wmu_QuZYJCLa0CPho2HTHw==/lib/arm64/libnative-lib.so
7593f7f000-7593f80000 rw-p 00034000 103:13 1180186                       /data/app/com.kevin.sotest-Wmu_QuZYJCLa0CPho2HTHw==/lib/arm64/libnative-lib.so
```

1.  frida 脚本将该内存中的内容全部 dump 下来

```
var global_so_name = "libnative-lib.so";

function catMaps() {
    Java.perform(function () {
        console.log("执行 -> ", "hook starting dump");
        var ProcessClass = Java.use("android.os.Process");
        var pid = ProcessClass.myPid();
        console.log("pid", ProcessClass.myPid());
        console.log("cat /proc/" + pid + "/maps | grep " + global_so_name);
    })
}


function dump() {
    var inputArgs = arguments;
    Java.perform(function () {
        for (var i in inputArgs) {
            var fileHandle = null;
            if (i == 0) {
                fileHandle = new File("/sdcard/" + global_so_name, "wb");
            } else {
                fileHandle = new File("/sdcard/" + global_so_name, "ab");
            }
            if (fileHandle && fileHandle != null) {
                var base = "0x" + inputArgs[i].split("-")[0];
                var end = "0x" + inputArgs[i].split("-")[1];
                var size = parseInt(parseInt(end) - parseInt(base));
                console.log("size", size, typeof size);
                
                console.log("base", base, typeof base);
                var libso_buffer = ptr(base).readByteArray(size);
                console.log("buffer", libso_buffer);
                fileHandle.write(libso_buffer);
                fileHandle.flush();
                fileHandle.close();
            }
        }
    })
}
```

```
var targetAddr = null;



var symbols = Process.findModuleByName("libc.so").enumerateSymbols();
for (var i = 0; i < symbols.length; i++){
    if (symbols[i].name === "__system_property_get"){
        targetAddr = symbols[i].address;
    }
}


Interceptor.attach(targetAddr, {
    onEnter: function (args) {
        this._name = args[0].readCString();
        this._value = args[1];
    },
    onLeave: function (retval) {
        console.log(JSON.stringify({
            result_length: retval,
            name: this._name,
            val: this._value.readCString()
        }))
    }
});
```

[](#Process-id "Process.id")[Process.id](http://process.id/)
------------------------------------------------------------

返回附加目标进程的 PID

```
function Process_pid() {
    console.log(Process.id);
}
```

调用的结果和在 android 中通过 `ps -ef | grep hellojni_sign2`的 pid 相同

```
u0_a139      10190   639 0 15:03:40 ?     00:00:10 com.example.hellojni_sign2
```

[](#Process-isDebuggerAttached "Process.isDebuggerAttached()")Process.isDebuggerAttached()
------------------------------------------------------------------------------------------

检测当前是否对目标程序已经附加

[](#Process-enumerateModules "Process.enumerateModules()")Process.enumerateModules()
------------------------------------------------------------------------------------

枚举当前加载的模块，返回模块对象的数组。 Process.enumerateModules() 会枚举当前所有已加载的 so 模块，并且返回了数组 Module 对象; 可以通过 `JSON.stringify()`打印 module 对象;

```
function ProcessTest() {
    var modules = Process.enumerateModules();
    for (var i = 0; i < modules.length; i++) {
        console.log(JSON.stringify(modules[i]));
        
        var module = modules[i];
        console.log("module.name", module.name);
        console.log("module.base", module.base);
        console.log("module.size", module.size);
        console.log("module.path", module.path);
    }
}
```

其中 module 对象中含有的属性为:

*   name: so 的名称
*   base: so 加载的基础地址
*   size: so 文件的大小
*   path: so 文件的路径

[](#Process-enumerateThreads "Process.enumerateThreads()")Process.enumerateThreads()
------------------------------------------------------------------------------------

`Process.enumerateThreads()`：枚举当前所有的线程，返回包含以下属性的对象数组：

<table><thead><tr><th>索引</th><th>属性</th><th>含义</th></tr></thead><tbody><tr><td>1</td><td>id</td><td>线程 id</td></tr><tr><td>2</td><td>state</td><td>当前运行状态有 running, stopped, waiting, uninterruptible or halted</td></tr><tr><td>3</td><td>context</td><td>带有键 pc 和 sp 的对象，它们是分别为 ia32/x64/arm 指定 EIP/RIP/PC 和 ESP/RSP/SP 的 NativePointer 对象。也可以使用其他处理器特定的密钥，例如 eax、rax、r0、x0 等</td></tr></tbody></table>

[](#Module-对象的属性 "Module 对象的属性")Module 对象的属性
--------------------------------------------

索引 属性 含义 1 name 模块名称 2 base 模块地址，其变量类型为 NativePointer 3 size 大小 4 path 完整文件系统路径

[](#Module-load "Module.load()")Module.load()
---------------------------------------------

主要用于加载指定 so 文件, 返回一个 Module 对象;

```
function frida_Module() {
    Java.perform(function () {
         
         const hooks = Module.load('libhello.so');
         
         console.log("模块名称:",hooks.name);
         console.log("模块地址:",hooks.base);
         console.log("大小:",hooks.size);
         console.log("文件系统路径",hooks.path);
    });
}
setImmediate(frida_Module,0);

输出如下：
模块名称: libhello.so
模块地址: 0xdf2d3000
大小: 24576
文件系统路径 /data/app/com.roysue.roysueapplication-7adQZoYIyp5t3G5Ef5wevQ==/lib/arm/libhello.so
```

[](#Module-enumerateImports "Module.enumerateImports()")Module.enumerateImports()
---------------------------------------------------------------------------------

枚举模块中的所有 import 函数;

```
function frida_Module() {
    Java.perform(function () {
        const hooks = Module.load('libhello.so');
        var Imports = hooks.enumerateImports();
        for(var i = 0; i < Imports.length; i++) {
            
            console.log("type:",Imports[i].type);
            
            console.log("name:",Imports[i].name);
            
            console.log("module:",Imports[i].module);
            
            console.log("address:",Imports[i].address);
         }
    });
}
setImmediate(frida_Module,0);

输出如下：
[Google Pixel::com.roysue.roysueapplication]-> type: function
name: __cxa_atexit
module: /system/lib/libc.so
address: 0xf58f4521
type: function
name: __cxa_finalize
module: /system/lib/libc.so
address: 0xf58f462d                                                                                                                                           
type: function
name: __stack_chk_fail
module: /system/lib/libc.so
address: 0xf58e2681
...
```

[](#Module-enumerateExports "Module.enumerateExports()")Module.enumerateExports()
---------------------------------------------------------------------------------

获取指定 so 的所有导出函数

```
function frida_Module() {
    Java.perform(function () {
        const hooks = Module.load('libhello.so');
        var Exports = hooks.enumerateExports();
        for(var i = 0; i < Exports.length; i++) {
            
            console.log("type:",Exports[i].type);
            
            console.log("name:",Exports[i].name);
            
            console.log("address:",Exports[i].address);
         }
    });
}
setImmediate(frida_Module,0);

输出如下：
[Google Pixel::com.roysue.roysueapplication]-> type: function
name: Java_com_roysue_roysueapplication_hellojni_getSum
address: 0xdf2d411b
type: function
name: unw_save_vfp_as_X
address: 0xdf2d4c43
type: function
address: 0xdf2d4209
type: function
...
```

[](#Module-enumerateSymbols "Module.enumerateSymbols()")Module.enumerateSymbols()
---------------------------------------------------------------------------------

获取指定 so 的所有符号表

```
function frida_Module() {
    Java.perform(function () {
        const hooks = Module.load('libc.so');
        var Symbol = hooks.enumerateSymbols();
        for(var i = 0; i < Symbol.length; i++) {
            console.log("isGlobal:",Symbol[i].isGlobal);
            console.log("type:",Symbol[i].type);
            console.log("section:",JSON.stringify(Symbol[i].section));
            console.log("name:",Symbol[i].name);
            console.log("address:",Symbol[i].address);
         }
    });
}
setImmediate(frida_Module,0);

输出如下：
isGlobal: true
type: function
section: {"id":"13.text","protection":"r-x"}
name: _Unwind_GetRegionStart
address: 0xf591c798
isGlobal: true
type: function
section: {"id":"13.text","protection":"r-x"}
name: _Unwind_GetTextRelBase
address: 0xf591c7cc
...
```

[](#Module-findExportByName-exportName-Module-getExportByName-exportName "Module.findExportByName(exportName), Module.getExportByName(exportName)")Module.findExportByName(exportName), Module.getExportByName(exportName)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

```
function frida_Module() {
    Java.perform(function () {
        Module.getExportByName('libhello.so', 'c_getStr')
        console.log("Java_com_roysue_roysueapplication_hellojni_getStr address:",Module.findExportByName('libhello.so', 'Java_com_roysue_roysueapplication_hellojni_getStr'));
        console.log("Java_com_roysue_roysueapplication_hellojni_getStr address:",Module.getExportByName('libhello.so', 'Java_com_roysue_roysueapplication_hellojni_getStr'));
    });
}
setImmediate(frida_Module,0);

输出如下：
Java_com_roysue_roysueapplication_hellojni_getStr address: 0xdf2d413d
Java_com_roysue_roysueapplication_hellojni_getStr address: 0xdf2d413d
```

[](#Module-findBaseAddress-name-、Module-getBaseAddress-name "Module.findBaseAddress(name)、Module.getBaseAddress(name)")Module.findBaseAddress(name)、Module.getBaseAddress(name)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

返回 name 模块的绝对基地址

```
function frida_Module() {
    Java.perform(function () {
        var name = "libhello.so";
        console.log("so address:",Module.findBaseAddress(name));
        console.log("so address:",Module.getBaseAddress(name));
    });
}
setImmediate(frida_Module,0);

输出如下：
so address: 0xdf2d3000
so address: 0xdf2d3000
```

Memory 的一些 API 通常是对内存处理, 譬如 `Memroy.copy()`用来复制内存; 又或者`Memory.writeByteArray`写入字节到指定内存中;

[](#Memory-scan搜索内存数据 "Memory.scan搜索内存数据")Memory.scan 搜索内存数据
------------------------------------------------------------

其主要功能是搜索内存中以`address`地址开始，搜索长度为`size`，需要搜是条件是`pattern，callbacks`搜索之后的回调函数；此函数相当于搜索内存的功能。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

如果我想搜索在内存中`112A`地址的起始数据要怎么做，代码示例如下。

```
function frida_Memory() {
    Java.perform(function () {
        
        var module = Process.findModuleByName("libhello.so"); 
        
        var pattern = "03 49 ?? 50 20 44";
        
        console.log("base:"+module.base)
        
        var res = Memory.scan(module.base, module.size, pattern, {
            onMatch: function(address, size){
                
                console.log('搜索到 ' +pattern +" 地址是:"+ address.toString());  
            }, 
            onError: function(reason){
                
                console.log('搜索失败');
            },
            onComplete: function()
            {
                
                console.log("搜索完毕")
            }
          });
    });
}
setImmediate(frida_Memory,0);
```

先来看看回调函数的含义，`onMatch：function(address，size)`：使用包含作为`NativePointer`的实例地址的`address`和指定大小为数字的`size`调用，此函数可能会返回字符串`STOP`以提前取消内存扫描。`onError：Function(Reason)`：当扫描时出现内存访问错误时使用原因调用。`onComplete：function()`：当内存范围已完全扫描时调用。

我们来来说上面这段代码做了什么事情：搜索`libhello.so`文件在内存中的数据，搜索以`pattern`条件的在内存中能匹配的数据。搜索到之后根据回调函数返回数据。

![](https://kevinspider-1258012111.cos.ap-shanghai.myqcloud.com/2022-02-18-084808.png)

[](#搜索内存数据Memory-scanSync "搜索内存数据Memory.scanSync")搜索内存数据 Memory.scanSync
------------------------------------------------------------------------

功能与`Memory.scan`一样，只不过它是返回多个匹配到条件的数据。代码示例如下。

```
function frida_Memory() {

  Java.perform(function () {

    var module = Process.findModuleByName("[libhello.so](http://libhello.so)"); 

    var pattern = "03 49 ?? 50 20 44";

    var scanSync = Memory.scanSync(module.base, module.size, pattern);

    console.log("scanSync:"+JSON.stringify(scanSync));

  });

}

setImmediate(frida_Memory,0);



scanSync:[{"address":"0xdf2d412a","size":6}]
```

[](#Memory-alloc "Memory.alloc()")Memory.alloc()
------------------------------------------------

在目标进程中的堆上申请`size`大小的内存，并且会按照`Process.pageSize`对齐，返回一个`NativePointer`，并且申请的内存如果在`JavaScript`里面没有对这个内存的使用的时候会自动释放的。也就是说，如果你不想要这个内存被释放，你需要自己保存一份对这个内存块的引用。

```
function frida_Memory() {
    Java.perform(function () {
        const r = Memory.alloc(10);
        console.log(hexdump(r, {
            offset: 0,
            length: 10,
            header: true,
            ansi: false
        }));
    });
}
setImmediate(frida_Memory,0);
```

[](#Memory-allocUtf8String-Memory-allocUtf16String-Memory-allocAnsiString "Memory.allocUtf8String(),Memory.allocUtf16String(),Memory.allocAnsiString()")Memory.allocUtf8String(),Memory.allocUtf16String(),Memory.allocAnsiString()
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

`Memory.allocUtf8String(str)` 分配 utf 字符串

`Memory.allocUtf16String` 分配 utf16 字符串

`Memory.allocAnsiString` 分配 ansi 字符串

[](#内存复制Memory-copy "内存复制Memory.copy")内存复制 Memory.copy
------------------------------------------------------

如同`c api memcp`一样调用，使用案例如下。

从`module.base`中复制`10`个字节的内存到新年申请的`r`内

```
function frida_Memory() {
    Java.perform(function () {
        
        var module = Process.findModuleByName("libhello.so"); 
        
        var pattern = "03 49 ?? 50 20 44";
        
        var scanSync = Memory.scanSync(module.base, module.size, pattern);
        
        const r = Memory.alloc(10);
        
        Memory.copy(r,module.base,10);
        console.log(hexdump(r, {
            offset: 0,
            length: 10,
            header: true,
            ansi: false
        }));
    });
}
setImmediate(frida_Memory,0);
```

[](#写入内存Memory-writeByteArray "写入内存Memory.writeByteArray")写入内存 Memory.writeByteArray
------------------------------------------------------------------------------------

将字节数组写入一个指定内存，代码示例如下:

```
function frida_Memory() {     
    Java.perform(function () {
        
        var arr = [ 0x72, 0x6F, 0x79, 0x73, 0x75, 0x65];
        
        const r = Memory.alloc(arr.length);
        
        Memory.writeByteArray(r,arr);
        
        console.log(hexdump(r, {
            offset: 0,
            length: arr.length,
            header: true,
            ansi: false
        }));  
    });
}
setImmediate(frida_Memory,0);

输出如下。
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  72 6f 79 73 75 65                                roysue
```

[](#读取内存Memory-readByteArray "读取内存Memory.readByteArray")读取内存 Memory.readByteArray
---------------------------------------------------------------------------------

将一个指定地址的数据按照 c 中 bytearray 进行读取

```
function frida_Memory() {     
    Java.perform(function () {
        
        var arr = [ 0x72, 0x6F, 0x79, 0x73, 0x75, 0x65];
        
        const r = Memory.alloc(arr.length);
        
        Memory.writeByteArray(r,arr);
        
        var buffer = Memory.readByteArray(r, arr.length);
        
        console.log("Memory.readByteArray:");
        console.log(hexdump(buffer, {
            offset: 0,
            length: arr.length,
            header: true,
            ansi: false
        }));
      });  
    });
}
setImmediate(frida_Memory,0);


[Google Pixel::com.roysue.roysueapplication]-> Memory.readByteArray:
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  72 6f 79 73 75 65                                roysue
```

```
var anti_fgets = function () {
    var fgetsPtr = Module.findExportByName("libc.so", "fgets");
    var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
    Interceptor.replace(fgetsPtr, new NativeCallback(function (buffer, size, fp) {
        var retval = fgets(buffer, size, fp);
        var bufstr = Memory.readUtf8String(buffer);
        if (bufstr.indexOf("TracerPid:") > -1) {
            console.log(bufstr); 
            Memory.writeUtf8String(buffer, "TracerPid:\t0");
        }
        return retval;
    }, 'pointer', ['pointer', 'int', 'pointer']));
};
```

1.  使用 native_trace 对关键函数进行批量 trace, 获取 trace.log
2.  使用 python 脚本提取其中的地址, 编辑了模板快速使用 `trace`

```
import re

native_trace = """
16071 ms     | sub_1494c4()                  ;debug 最终结果出现位置 (md5resultPtr, len, resultPtr)
 16071 ms     | sub_2546c()
 16071 ms     | sub_28d18()
 16071 ms     |    | sub_bc028()
 16071 ms     |    |    | sub_cf970()
 16071 ms     |    |    |    | sub_d4680()
"""

addr_list = re.findall(r"sub_(.*)\(\)", native_trace)
addr_list = list(map(lambda x:"0x" + x, addr_list))
print(addr_list)
```

1.  使用 trace 出来的地址使用 frida 批量进行 hook; 编辑了模板快速使用 `print_arg`和 `trace`

```
var base = Process.findModuleByName("libhello-jni.so").base;

function print_arg(addr) {
    var range = Process.findRangeByAddress(addr);
    if (range != null) {
        return hexdump(addr) + '\r\n';
    }else{
        return ptr(addr) + "\r\n";
    }
}

function native_trace(addr) {
    var offset = addr;
    addr = base.add(addr);
    Interceptor.attach(addr, {
        onEnter: function (args) {
            
            console.log("======================onenter sub_" + offset + "==========================");
            for (var i = 0; i < 5; i++) {
                eval("this.arg" + i + "=args[" + i + "]");
                console.log("this.arg" + i, print_arg(eval("this.arg" + i)));
            }

        },
        onLeave: function (returnValue) {
            console.log("======================onleave sub_" + offset + "==========================");
            for (var i = 0; i < 5; i++) {
                console.log("this.arg" + i, print_arg(eval("this.arg" + i)))
            }
            console.log("returnVlaue", print_arg(returnValue));
        }
    })
}

function main() {
    var trace_list = ['0x1494c4'];
    for (var i = 0; i < trace_list.length; i++) {
        native_trace(trace_list[i]);
     }
}

setImmediate(main);
```

*   r0ysue
*   yang