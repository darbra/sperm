> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.seeflower.dev](https://blog.seeflower.dev/archives/54/)

> Be patient.

*   Lilac，又名：[白龙~](https://blog.csdn.net/qq_38851536)

龙哥往往一语中的，给我带来了莫大的帮助，非常感谢

本文旨在对 getByte 算法进行分析与还原，仅供学习交流

本文不会提供完整算法脚本

本文将涉及以下内容：

*   OLLVM 虚假控制流 (OLLVM-BCF) 反混淆
*   cutter 反编译
*   md5 算法识别
*   SHA256 算法还原——魔数修改
*   Salsa20 算法还原——逻辑魔改
*   trace_natives 与 frida-trace 搭配使用
*   findhash 使用
*   gettimeofday 和 lrand48
*   ~unidbg 模拟调用 (下一篇文章)~

**！！！为了节省版面，文章中重复的 hook 代码会省略掉，复现时请记得自行补充**

<table><thead><tr><th>名称</th><th>物料</th><th>补充</th></tr></thead><tbody><tr><td>目标方法</td><td>getByte</td><td>-</td></tr><tr><td>目标类</td><td>com.tencent.starprotocol.ByteData</td><td>-</td></tr><tr><td>目标 so</td><td>libpoxy_star.so</td><td>md5: 889415fb8e886dfdc3fdd405c105d262</td></tr><tr><td>目标 apk</td><td>com.tencent.qqlive_V8.3.95.26016.apk</td><td>md5: 6d6cd9c0b36c49f17d0f204cf917774e</td></tr><tr><td>frida-server</td><td><a href="https://github.com/frida/frida/releases/tag/14.2.18" target="_blank">frida-server-14.2.18-android-arm64</a></td><td>-</td></tr><tr><td>python</td><td>3.8.5</td><td>由 miniconda 创建</td></tr><tr><td>IDA</td><td>IDA Pro 7.5</td><td><a href="https://down.52pojie.cn/Tools/Disassemblers/IDA_Pro_v7.5_Portable.zip" target="_blank">爱盘地址</a></td></tr><tr><td>CyberChef</td><td>CyberChef</td><td><a href="https://gchq.github.io/CyberChef" target="_blank">在线地址</a></td></tr><tr><td>findhash</td><td>findhash</td><td><a href="https://github.com/Pr0214/findhash" target="_blank">Github 地址</a></td></tr><tr><td>trace_natives</td><td>trace_natives</td><td><a href="https://github.com/Pr0214/trace_natives" target="_blank">Github 地址</a></td></tr><tr><td>jnitrace</td><td>jnitrace</td><td><a href="https://github.com/chame1eon/jnitrace" target="_blank">Github 地址</a></td></tr><tr><td>JNI-Frida-Hook</td><td>JNI-Frida-Hook</td><td><a href="https://github.com/Areizen/JNI-Frida-Hook" target="_blank">Github 地址</a></td></tr><tr><td>hook_RegisterNatives</td><td>hook_RegisterNatives</td><td><a href="https://github.com/lasting-yang/frida_hook_libart/blob/master/hook_RegisterNatives.js" target="_blank">Github 地址</a></td></tr><tr><td>cutter</td><td>cutter</td><td><a href="https://cutter.re/" target="_blank">官方地址</a></td></tr><tr><td>测试 ROM</td><td>QQ1B.200205.002</td><td><del>能跑 frida 就行</del></td></tr></tbody></table>

*   物料 [https://gofile.io/d/bqu3Hd](https://gofile.io/d/bqu3Hd)

算法稳定主动调用
--------

此处稳定意为固定输入、固定输出

### 获取输入输出参数

已知目标方法是`com.tencent.starprotocol.ByteData.getByte`，要触发这个地方的调用很简单，APP 随便打开一个视频即可

首先用 objection 看一眼

```
android hooking watch class_method com.tencent.starprotocol.ByteData.getByte --dump-args --dump-return

```

结果如下（部分参数已和谐）

```
(agent) [466367] Called com.tencent.starprotocol.ByteData.getByte(android.content.Context, long, long, long, long, java.lang.Object, java.lang.Object, java.lang.Object, java.lang.Object)
(agent) [466367] Arguments com.tencent.starprotocol.ByteData.getByte(com.tencent.qqlive.ona.base.QQLiveApplicationWrapper@bdbc6ce, 1, 0, 0, 0, [Ljava.lang.String;@6e859e6, , ce****************************b2, [B@a368b27)
(agent) [466367] Return Value: [object Object]

```

显然 objection 不能直接得到全部参数具体内容，需要手动写个 hook 脚本

首先准备两个 hexdump 函数

```
function jhexdump(array) {
    if(!array) return;
    console.log("---------jhexdump start---------");
    var ptr = Memory.alloc(array.length);
    for(var i = 0; i < array.length; ++i)
        Memory.writeS8(ptr.add(i), array[i]);
    console.log(hexdump(ptr, {offset: 0, length: array.length, header: false, ansi: false}));
    console.log("---------jhexdump end---------");
}
function dumpByteArray(obj){
    console.log("---------dumpByteArray start---------");
    let obj_ptr = ptr(obj.$h).readPointer();
    let buf_ptr = obj_ptr.add(Process.pointerSize * 3);
    let size = obj_ptr.add(Process.pointerSize * 2).readU32();
    console.log(hexdump(buf_ptr, {offset: 0, length: size, header: false, ansi: false}));
    console.log("---------dumpByteArray end---------");
}

```

打印参数

e.g. `frida -U -n com.tencent.qqlive -l agent/poxy_star.js -o agent/poxy_star.log`

```
function getByte_LogArgs(){
    Java.perform(function(){
        let gson = Java.use('com.google.gson.Gson');
        let ByteDataCls = Java.use("com.tencent.starprotocol.ByteData");
        ByteDataCls.getByte.overload("android.content.Context", "long", "long", "long", "long", "java.lang.Object", "java.lang.Object", "java.lang.Object", "java.lang.Object").implementation = function(context, num1, num2, num3, num4, obj1, obj2, obj3, obj4){
            console.log(context, num1, num2, num3, num4);
            console.log("obj1", obj1.$className, gson.$new().toJson(obj1))
            console.log("obj2", obj2.$className, gson.$new().toJson(obj2))
            console.log("obj3", obj3.$className, gson.$new().toJson(obj3))
            console.log("obj4", obj4.$className, gson.$new().toJson(obj4))
            dumpByteArray(obj4);
            let resp = this.getByte(context, num1, num2, num3, num4, obj1, obj2, obj3, obj4)
            jhexdump(resp);
            return resp
        }
    })
}

setImmediate(getByte_LogArgs);

```

点击视频，得到下面的结果

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_00-06-26.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_00-06-26.png)

参数拿到后就可以开始主动调用了

### frida 稳定主动调用

经过测试，发现传入参数固定，返回结果并不固定，这是因为原算法用到了 **lrand48** 和 **gettimeofday**

这是怎么发现的呢

~**当然是根据经验测出来的**~

一般遇到传入参数固定但是结果变化，统统往**时间**、**随机数**上靠

这里先认为是假设，下面进行验证

#### 固定 lrand48 和 gettimeofday 返回

**lrand48** 定义如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_10-58-22.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_10-58-22.png)

那么这里直接将返回设置位`7`

这里提一点，为了和其他可能为 0 的参数进行区分，建议这里**尽量不要设置为 0**

**gettimeofday** 定义如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-12-21.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-12-21.png)

> 定义函数 int gettimeofday (struct timeval _tv, struct timezone_ tz);  
> 函数说明 gettimeofday() 会把目前的时间有 tv 所指的结构返回，当地时区的信息则放到 tz 所指的结构中。

类型信息

```
typedef long __kernel_long_t;
typedef __kernel_long_t __kernel_time_t;
typedef __kernel_long_t __kernel_suseconds_t;
struct timeval {
    __kernel_time_t tv_sec;
    __kernel_suseconds_t tv_usec;
};

```

根据上述信息，那么只需要在 **gettimeofday** 返回时向参数一写入两个 long 类型的数即可，分别代表秒数、微秒数

#### 验证

另外对于图中马赛克的参数，经过测试，确定改变它们不影响最终结果，后面使用特定固定值

于是编写如下完整的稳定主动调用脚本

```
function freeze_funcs(){
    let lrand48_addr = Module.findExportByName("libc.so", "lrand48");
    Interceptor.attach(lrand48_addr, {onLeave: function(retval){retval.replace(7)}});
    let tm_s = 1626403551;
    let tm_us = 5151606;
    let gettimeofday_addr = Module.findExportByName("libc.so", "gettimeofday");
    Interceptor.attach(gettimeofday_addr, {
        onEnter: function(args) {
            this.tm_ptr = args[0];
        },
        onLeave:function(retval){
            this.tm_ptr.writeLong(tm_s);
            this.tm_ptr.add(0x4).writeLong(tm_us);
        }
    });
}

function call_getByte(){
    Java.perform(function(){
        let LongCls = Java.use("java.lang.Long");
        let StringCls = Java.use("java.lang.String");
        let ReflectArrayCls  = Java.use('java.lang.reflect.Array')
        let ByteDataCls = Java.use("com.tencent.starprotocol.ByteData");
        let ctx = Java.use('android.app.ActivityThread').currentApplication().getApplicationContext();
        let num_1 = LongCls.$new(1).longValue();
        let num_2 = LongCls.$new(0).longValue();
        let num_3 = LongCls.$new(0).longValue();
        let num_4 = LongCls.$new(0).longValue();
        let obj1 = ReflectArrayCls.newInstance(StringCls.class, 9);
        ReflectArrayCls.set(obj1, 0, "dl_10303");
        ReflectArrayCls.set(obj1, 1, "1");
        ReflectArrayCls.set(obj1, 2, "66666666666666666666666666666666");
        ReflectArrayCls.set(obj1, 3, "getCKey");
        ReflectArrayCls.set(obj1, 4, "888888888888888888888888888888888888");
        ReflectArrayCls.set(obj1, 5, "1626403551515");
        ReflectArrayCls.set(obj1, 6, "");
        ReflectArrayCls.set(obj1, 7, "8.3.95.26016");
        ReflectArrayCls.set(obj1, 8, "com.tencent.qqlive");
        let obj2 = StringCls.$new("");
        let obj3 = StringCls.$new("66666666666666666666666666666666");
        let obj4 = Java.array('B', [49,54,50,54,52,48,51,53,53,49,44,110,48,48,51,57,101,121,49,109,109,100,44,110,117,108,108]);
        let ByteDataIns = ByteDataCls.getInstance()
        let byte = ByteDataIns.getByte(ctx, num_1, num_2, num_3, num_4, obj1, obj2, obj3, obj4);
        jhexdump(byte);
        Interceptor.detachAll();
    })
}

freeze_funcs();
call_getByte();

```

现在得到固定的返回了

```
---------jhexdump start---------
b6abddd8  68 65 68 61 00 00 00 01 01 00 00 00 00 00 00 00  heha............
b6abdde8  00 00 00 03 08 00 00 00 00 44 a7 2c da 00 00 00  .........D.,....
b6abddf8  01 00 00 00 96 00 01 00 08 00 00 01 7a ad 34 cb  ............z.4.
b6abde08  37 00 02 00 0a 68 68 68 68 68 68 69 69 69 68 00  7....hhhhhhiiih.
b6abde18  03 00 04 01 00 00 01 00 05 00 04 01 00 00 01 00  ................
b6abde28  04 00 04 00 00 00 00 00 06 00 04 01 00 00 04 00  ................
b6abde38  07 00 04 01 00 00 02 00 08 00 04 01 00 00 03 00  ................
b6abde48  09 00 20 45 4f 2a db 77 40 90 33 9f e2 58 8c 2f  .. EO*.w@.3..X./
b6abde58  c6 4a 3d ce 7a c7 30 7d ce ac 0b aa b6 18 b3 6f  .J=.z.0}.......o
b6abde68  41 74 d3 00 0a 00 10 cd b5 df 89 f5 2d 25 05 4b  At..........-%.K
b6abde78  85 81 f6 cf af 1e fb 00 0b 00 10 fb 08 ad 5f af  .............._.
b6abde88  a8 93 35 2d 13 cb 93 27 e8 f7 fd                 ..5-...'...
---------jhexdump end---------

```

so 分析准备
-------

通过反编译 apk，可以知道`getByte`是一个 native 函数，熟练打开 IDA、拖入 so、搜索导出函数

不好意思，没有

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-36-11.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-36-11.png)

再看看字符串

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-37-07.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-37-07.png)

动态注册跑不了了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-37-58.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-37-58.png)

直接就是 [**hook_RegisterNatives**](https://github.com/lasting-yang/frida_hook_libart/blob/master/hook_RegisterNatives.js)

```
[RegisterNatives] java_class: com.tencent.starprotocol.ByteData name: getByte sig: (Landroid/content/Context;JJJJLjava/lang/Object;Ljava/lang/Object;Ljava/lang/Object;Ljava/lang/Object;)[B fnPtr: 0x90c969ad module_name: libpoxy_star.so module_base: 0x90c89000 offset: 0xd9ad
[RegisterNatives] java_class: com.tencent.starprotocol.ByteData name: putByte sig: (Landroid/content/Context;JJJJLjava/lang/Object;Ljava/lang/Object;Ljava/lang/Object;Ljava/lang/Object;)I fnPtr: 0x90caa995 module_name: libpoxy_star.so module_base: 0x90c89000 offset: 0x21995

```

好了，让我们看看`sub_D9AC`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-49-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_11-49-09.png)

似乎有点麻烦，很多分支......

此刻是不是头都大了，再看看 so 大小，828KB，发现了什么没有

就动态注册了两个方法，并且没有其他静态注册的方法，so 文件却如此之大

显然 so 有混淆

好在反编译代码中有迹可循，可以看到反编译结果有很多下面这样的判断

```
((dword_C2C04 ^ dword_C2C08 ^ 0x100011AA) & 0x5120FBAA) == -1964041611

```

如果你经验丰富，一眼就能看出来这是 **BCF(Bogus Control Flow)**，即虚假控制流

~吐槽：我是算法还原完了才知道这个点，哎，心累...~

### OLLVM-BCF 反混淆

这烦人的 BCF 自然是有方案处理的，具体请参考下面这篇文章

*   [Hex-Rays: 十步杀一人，两步秒 OLLVM-BCF](https://bbs.pediy.com/thread-257213.htm)

简单来说就是将 data 段数据的`Write`属性去掉，然后重新 F5 即可，IDA 会自动完成 BCF 的优化

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-02-25.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-02-25.png)

这是优化后的结果，是不是非常非常清晰了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-05-08.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-05-08.png)

知道这个操作后，简直事半功倍，基本上不用去手动 hook 确定走向了

看雪还有一些关于 BCF 反混淆的文章，经过测试还是 IDA 来的快，效果相对更好...

### cutter 搭配使用

即使进行了 BCF 反混淆操作

IDA 反编译出来的代码阅读有些问题，比如看不懂变量传递关系

表现为函数传入参数在确定要走的那个分支里面没有被直接或者间接调用

好在 cutter 的反编译结果至少是能看出来的

以`sub_AA924`为例

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-19-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-19-09.png)

大致可以推测出来数据处理在最后一个 case 进行

但是`*(_BYTE *)(*v10 + *v8)`和 a1 什么关系实在是看不懂，v5 和 a1 又有什么关系？

这也许要直接阅读汇编代码才能看出其中关系了，或者这是某种特定的形式

实在不行也可以 hook 来确定，毕竟动态的是准确的

或者这是一个我还没学习到的知识点

好在 cutter 可以帮上忙

根据下图的标注，可以知道 a1 就是`piStack44`了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-37-11.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-37-11.png)

而对于后面的`puStack52`，也能根据下面的标注推测出每一轮 + 1，等于是挨个取 a1 处的内容

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-39-46.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_12-39-46.png)

如果只看 IDA，可能就比较懵了...

所以搭配使用效果更佳

返回结果追踪定位
--------

按照常规思路，一般是看 native 函数的返回，然后追踪结果的生成过程

先打印一波 so 的结果

```
function jbhexdump(array) {
    console.log("---------jbhexdump start---------");
    let env = Java.vm.getEnv();
    let size = env.getArrayLength(array);
    let data = env.getByteArrayElements(array);
    console.log(hexdump(data, {offset: 0, length: size, header: false, ansi: false}));
    env.releaseByteArrayElements(array, data, 0);
    console.log("---------jbhexdump end---------");
}

function inline_hook(){
    let base_addr = Module.getBaseAddress("libpoxy_star.so");
    Interceptor.attach(base_addr.add(0xD9AC).add(1), {
        onLeave: function (retval) {
            console.log(`onLeave sub_D9AC`);
            jbhexdump(retval);
        }
    });
}

freeze_funcs();
inline_hook();
call_getByte();

```

日志如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_13-24-29.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_13-24-29.png)

**一 模 一 样**

是否反向查看函数，挨个跟踪定位算法呢，答案是否

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_13-29-11.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_13-29-11.png)

在已经有 BCF 反混淆的加持下，依然可以看到函数嵌套函数，如果没有足够的精力，可能是难以定位算法的

既然 so 结尾返回的是 jbyteArray，那么可以通过 hook SetByteArrayRegion 实现快速定位

### jnitrace 定位 SetByteArrayRegion 调用

jnitrace 如果以 spawn 模式可能难以追踪，这里采用 attach 模式

具体操作是打开 APP 后一小会儿执行命令，同时注意脚本中主动调用时机

```
jnitrace -l libpoxy_star.so -m attach -a poxy_star.js com.tencent.qqlive

```

[![](https://blog.seeflower.dev/images/oCam_2021_07_16_13_56_04_790.gif)](https://blog.seeflower.dev/images/oCam_2021_07_16_13_56_04_790.gif)

可以看到有一个`SetByteArrayRegion`内容和最终结果一样，地址是 **0x13d79**

另外可以通过`-o`选项设置追踪结果

这就是调用的位置了，位于`sub_13C7C`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-03-00.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-03-00.png)

### 更优雅的 jni 追踪

前面提到可以直接用 jnitrace 进行 jni 函数调用追踪，但是不太方便

只想在主动调用某个函数之间进行追踪怎么办呢

参考`JNI-Frida-Hook`这个项目，可以发现实现原理很简单，拿到 jni 函数的地址然后 hook 就行了

下面是一个简单的案例，这样会灵活很多

```
let jni_struct_array = [
    "reserved0", "reserved1", "reserved2", "reserved3", "GetVersion", "DefineClass", "FindClass", "FromReflectedMethod", "FromReflectedField", "ToReflectedMethod", "GetSuperclass", "IsAssignableFrom", "ToReflectedField", "Throw", "ThrowNew",
    "ExceptionOccurred", "ExceptionDescribe", "ExceptionClear", "FatalError", "PushLocalFrame", "PopLocalFrame", "NewGlobalRef", "DeleteGlobalRef", "DeleteLocalRef", "IsSameObject", "NewLocalRef", "EnsureLocalCapacity", "AllocObject", "NewObject",
    "NewObjectV", "NewObjectA", "GetObjectClass", "IsInstanceOf", "GetMethodID", "CallObjectMethod", "CallObjectMethodV", "CallObjectMethodA", "CallBooleanMethod", "CallBooleanMethodV", "CallBooleanMethodA", "CallByteMethod", "CallByteMethodV",
    "CallByteMethodA", "CallCharMethod", "CallCharMethodV", "CallCharMethodA", "CallShortMethod", "CallShortMethodV", "CallShortMethodA", "CallIntMethod", "CallIntMethodV", "CallIntMethodA", "CallLongMethod", "CallLongMethodV", "CallLongMethodA",
    "CallFloatMethod", "CallFloatMethodV", "CallFloatMethodA", "CallDoubleMethod", "CallDoubleMethodV", "CallDoubleMethodA", "CallVoidMethod", "CallVoidMethodV", "CallVoidMethodA", "CallNonvirtualObjectMethod", "CallNonvirtualObjectMethodV",
    "CallNonvirtualObjectMethodA", "CallNonvirtualBooleanMethod", "CallNonvirtualBooleanMethodV", "CallNonvirtualBooleanMethodA", "CallNonvirtualByteMethod", "CallNonvirtualByteMethodV", "CallNonvirtualByteMethodA", "CallNonvirtualCharMethod",
    "CallNonvirtualCharMethodV", "CallNonvirtualCharMethodA", "CallNonvirtualShortMethod", "CallNonvirtualShortMethodV", "CallNonvirtualShortMethodA", "CallNonvirtualIntMethod", "CallNonvirtualIntMethodV", "CallNonvirtualIntMethodA",
    "CallNonvirtualLongMethod", "CallNonvirtualLongMethodV", "CallNonvirtualLongMethodA", "CallNonvirtualFloatMethod", "CallNonvirtualFloatMethodV", "CallNonvirtualFloatMethodA", "CallNonvirtualDoubleMethod", "CallNonvirtualDoubleMethodV",
    "CallNonvirtualDoubleMethodA", "CallNonvirtualVoidMethod", "CallNonvirtualVoidMethodV", "CallNonvirtualVoidMethodA", "GetFieldID", "GetObjectField", "GetBooleanField", "GetByteField", "GetCharField", "GetShortField", "GetIntField",
    "GetLongField", "GetFloatField", "GetDoubleField", "SetObjectField", "SetBooleanField", "SetByteField", "SetCharField", "SetShortField", "SetIntField", "SetLongField", "SetFloatField", "SetDoubleField", "GetStaticMethodID",
    "CallStaticObjectMethod", "CallStaticObjectMethodV", "CallStaticObjectMethodA", "CallStaticBooleanMethod", "CallStaticBooleanMethodV", "CallStaticBooleanMethodA", "CallStaticByteMethod", "CallStaticByteMethodV", "CallStaticByteMethodA",
    "CallStaticCharMethod", "CallStaticCharMethodV", "CallStaticCharMethodA", "CallStaticShortMethod", "CallStaticShortMethodV", "CallStaticShortMethodA", "CallStaticIntMethod", "CallStaticIntMethodV", "CallStaticIntMethodA", "CallStaticLongMethod",
    "CallStaticLongMethodV", "CallStaticLongMethodA", "CallStaticFloatMethod", "CallStaticFloatMethodV", "CallStaticFloatMethodA", "CallStaticDoubleMethod", "CallStaticDoubleMethodV", "CallStaticDoubleMethodA", "CallStaticVoidMethod",
    "CallStaticVoidMethodV", "CallStaticVoidMethodA", "GetStaticFieldID", "GetStaticObjectField", "GetStaticBooleanField", "GetStaticByteField", "GetStaticCharField", "GetStaticShortField", "GetStaticIntField", "GetStaticLongField",
    "GetStaticFloatField", "GetStaticDoubleField", "SetStaticObjectField", "SetStaticBooleanField", "SetStaticByteField", "SetStaticCharField", "SetStaticShortField", "SetStaticIntField", "SetStaticLongField", "SetStaticFloatField",
    "SetStaticDoubleField", "NewString", "GetStringLength", "GetStringChars", "ReleaseStringChars", "NewStringUTF", "GetStringUTFLength", "GetStringUTFChars", "ReleaseStringUTFChars", "GetArrayLength", "NewObjectArray", "GetObjectArrayElement",
    "SetObjectArrayElement", "NewBooleanArray", "NewByteArray", "NewCharArray", "NewShortArray", "NewIntArray", "NewLongArray", "NewFloatArray", "NewDoubleArray", "GetBooleanArrayElements", "GetByteArrayElements", "GetCharArrayElements",
    "GetShortArrayElements", "GetIntArrayElements", "GetLongArrayElements", "GetFloatArrayElements", "GetDoubleArrayElements", "ReleaseBooleanArrayElements", "ReleaseByteArrayElements", "ReleaseCharArrayElements", "ReleaseShortArrayElements",
    "ReleaseIntArrayElements", "ReleaseLongArrayElements", "ReleaseFloatArrayElements", "ReleaseDoubleArrayElements", "GetBooleanArrayRegion", "GetByteArrayRegion", "GetCharArrayRegion", "GetShortArrayRegion", "GetIntArrayRegion",
    "GetLongArrayRegion", "GetFloatArrayRegion", "GetDoubleArrayRegion", "SetBooleanArrayRegion", "SetByteArrayRegion", "SetCharArrayRegion", "SetShortArrayRegion", "SetIntArrayRegion", "SetLongArrayRegion", "SetFloatArrayRegion",
    "SetDoubleArrayRegion", "RegisterNatives", "UnregisterNatives", "MonitorEnter", "MonitorExit", "GetJavaVM", "GetStringRegion", "GetStringUTFRegion", "GetPrimitiveArrayCritical", "ReleasePrimitiveArrayCritical", "GetStringCritical",
    "ReleaseStringCritical", "NewWeakGlobalRef", "DeleteWeakGlobalRef", "ExceptionCheck", "NewDirectByteBuffer", "GetDirectBufferAddress", "GetDirectBufferCapacity", "GetObjectRefType"
]

function getJNIFunctionAdress(func_name){
    
    let jnienv_addr = Java.vm.getEnv().handle.readPointer()
    let offset = jni_struct_array.indexOf(func_name) * Process.pointerSize;
    return Memory.readPointer(jnienv_addr.add(offset))
}

function hook_jni(func_name){
    let listener = null;
    switch (func_name){
        case "SetByteArrayRegion":
            listener = Interceptor.attach(getJNIFunctionAdress(func_name), {
                onEnter: function(args){
                    console.log(`env->${func_name} called from ${Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join("\n")}`);
                    this.arg_array = args[1];
                },
                onLeave: function(retval){
                    jbhexdump(this.arg_array);
                    console.log("SetByteArrayRegion onLeave");
                }
            })
        default:
            listener = Interceptor.attach(getJNIFunctionAdress(func_name), {
                onEnter: function(args){
                    console.log(`env->${func_name} called from ${Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join("\n")}`);
                }
            })
    }
    return listener;
}

function inline_hook(){
    let base_addr = Module.getBaseAddress("libpoxy_star.so");
    Interceptor.attach(base_addr.add(0xD9AC).add(1), {
        onEnter: function(args){
            this.hook_jni_interceptor = hook_jni("SetByteArrayRegion");
        },
        onLeave: function (retval) {
            this.hook_jni_interceptor.detach();
            console.log(`onLeave sub_D9AC`);
            jbhexdump(retval);
        }
    });
}

freeze_funcs();
inline_hook();
call_getByte();

```

这样追踪更方便，符合需求，输出结果如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-26-48.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-26-48.png)

返回结果反向推导与追踪
-----------

那就这个时候开始反向追踪了吗

不，让我们把它的调用流程图也弄出来

祭出 [**trace_natives**](https://github.com/Pr0214/trace_natives)，导入 IDA 插件，拿到要 trace 的函数列表

**frida-trace** 一把梭，注意，为了得到调用层次分明的日志，这里直接重定向到文件

```
frida-trace -UF -O libpoxy_star_1626418239.txt > star_trace.log

```

建议先执行一次上面的命令，让它自动生成 js，停掉

然后去目标 js 加一个返回时的日志打印，再重新运行

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-58-37.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-58-37.png)

第一次执行命令的时候比较慢，因为要生成 js，后面就很快了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-55-24.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_14-55-24.png)

然后执行主动调用脚本

得到记录后，找到添加的返回日志，把进入`sub_D9AD`到这里之间的内容单独拿出来

为什么这里是`sub_D9AD`而不是`sub_D9AC`请查阅 trace_natives 的 README

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-03-43.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-03-43.png)

去除不相关的记录后依然有 1w + 行

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-10-00.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-10-00.png)

当然还有办法精简，前面已经知道在进入`sub_13C7C`的时候，就已经产生最终结果了

那么只要`sub_13C7C`之前的记录即可

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-12-41.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-12-41.png)

很好，现在只剩 1000 + 的记录了

根据调用记录，可以大致确定`sub_13C7C`由`sub_11AC0`调用

另外发现`sub_11AC0`所在位置非常靠前，说明它应该是运算的核心位置

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-15-03.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-15-03.png)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-18-46.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-18-46.png)

现在可以反向追踪了，先看下最后这几个函数

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-23-35.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-23-35.png)

并没有什么有用的信息，再往前是一个比较长的调用

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-25-21.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-25-21.png)

`sub_ABD9C`中只有两个函数调用

*   `sub_ABDBC`
*   `sub_AAE88`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-27-42.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-27-42.png)

这个时候，先看看参数内容，因为很有可能`sub_AAE88`之前就已经出现最终结果了

如果是这样就可以不分析`sub_AAE88`了

hook 代码如下

```
function inline_hook(){
    let hook_flag = false;
    let base_addr = Module.getBaseAddress("libpoxy_star.so");
    Interceptor.attach(base_addr.add(0xD9AC).add(1), {
        onEnter: function(args){
            hook_flag = true;
            this.hook_jni_interceptor = hook_jni("SetByteArrayRegion");
        },
        onLeave: function (retval) {
            hook_flag = false;
            this.hook_jni_interceptor.detach();
            console.log(`onLeave sub_D9AC`);
            jbhexdump(retval);
        }
    });

    Interceptor.attach(base_addr.add(0xAAE88).add(1), {
        onEnter: function (args) {
            if(hook_flag){
                console.log(`call sub_AAE88`);
                this.arg_0 = args[0];
                console.log("sub_AAE88 arg_0", hexdump(args[0].readPointer()));
            }
        },
        onLeave: function (retval) {
            console.log("sub_AAE88 onLeave arg_0", hexdump(this.arg_0.readPointer()));
        }
    });
    Interceptor.attach(base_addr.add(0xABDBC).add(1), {
        onEnter: function (args) {
            if(hook_flag){
                console.log(`call sub_ABDBC`);
                this.arg_0 = args[0];
                console.log("sub_ABDBC arg_0", hexdump(args[0].readPointer()));
            }
        },
        onLeave: function (retval) {
            console.log("sub_ABDBC onLeave arg_0", hexdump(this.arg_0.readPointer()));
        }
    });
}

```

通过对比可以发现进入`sub_AAE88`时数据头部还不完整，但是后面的部分是一致的

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-37-27.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-37-27.png)

仔细观察头部数据发现三个可疑点

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-41-53.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-41-53.png)

首先是前四个字节，这会不会是什么校验呢

答案是否，可以通过变化传入参数确定这个位置是固定值

其次是 0x96，为何怀疑它，因为根据经验，一字节或两字节不为 0 前面为 0 的数据，很可能是后面数据的长度

> 0x96 = 150 = 11 + 8 * 16 + 11

`96`起到`f7 d0`这里，长度刚好是 150，那没跑了，就是长度值

剩下一个`44 a7 2c da`，这有可能是什么呢

逆向常常需要靠经验和猜测，既然确定 0x96 是长度，那它后面应该就是一个完整的数据块

那么猜测它是后面数据的校验，提到数据校验，相信大家都会想到 **CRC32**

用 **CyberChef** 验证一下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-51-39.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-51-39.png)

猜测正确

那么头部前面还有一些`01 02 03 00`可能是什么呢

首先在最开始已经将随机数固定为`7`了，那么基本可以排除它们和随机数无关

可能是时间吗，显然可能性不大，而且可以通过变化传入参数，可以发现除了 CRC32 部分，其他内容不变

这样一来可以推测是一些固定或者变化比较简单的运算，那么先把它放一边吧

现在分析`sub_ABDBC`函数

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-57-43.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_15-57-43.png)

可以看到里面有好几个重复函数，不多说，直接 hook 看参数内容

先看`sub_AC214`，参数内容都比较简单

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-00-47.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-00-47.png)

然后是`sub_AD1D0`，参数内容也是最终结果里面的

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-21-36.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-21-36.png)

根据以上信息，除去头部，对照内容后整理如下

<table><thead><tr><th>tag</th><th>length</th><th>payload</th><th>func</th></tr></thead><tbody><tr><td>00 01</td><td>00 08</td><td>00 00 01 7a ad 34 cb 37</td><td>sub_AC214</td></tr><tr><td>00 02</td><td>00 0a</td><td>68 68 68 68 68 68 69 69 69 68</td><td>sub_AD1D0</td></tr><tr><td>00 03</td><td>00 04</td><td>01 00 00 01</td><td>sub_AC214</td></tr><tr><td>00 05</td><td>00 04</td><td>01 00 00 01</td><td>sub_AC214</td></tr><tr><td>00 04</td><td>00 04</td><td>00 00 00 00</td><td>sub_AC214</td></tr><tr><td>00 06</td><td>00 04</td><td>01 00 00 04</td><td>sub_AC214</td></tr><tr><td>00 07</td><td>00 04</td><td>01 00 00 02</td><td>sub_AC214</td></tr><tr><td>00 08</td><td>00 04</td><td>01 00 00 03</td><td>sub_AC214</td></tr><tr><td>00 09</td><td>00 20</td><td>45 4f 2a db 77 40 90 33 9f e2 58 8c 2f c6 4a 3d ce 7a c7 30 7d ce ac 0b aa b6 18 b3 6f 41 74 d3</td><td>sub_AD1D0</td></tr><tr><td>00 0a</td><td>00 10</td><td>cd b5 df 89 f5 2d 25 05 4b 85 81 f6 cf af 1e fb</td><td>sub_AD1D0</td></tr><tr><td>00 0b</td><td>00 10</td><td>fb 08 ad 5f af a8 93 35 2d 13 cb 93 27 e8 f7 fd</td><td>sub_AD1D0</td></tr></tbody></table>

**第一个参数**是什么呢，通过测试可以发现当`gettimeofday`返回固定，这个值就固定

是不是和时间有关系呢，答案是 yes

> 0x0000017aad34cb37 = 1626403556151

固定的时间代码片段如下

```
let tm_s = 1626403551;
let tm_us = 5151606;

```

Q: 似乎有些对不上，这是什么问题呢  
A: tm_us 是微秒，显然这里是 5.151606s，超过 1s 了

将`tm_us`改为`151606`，重新整理

<table><thead><tr><th>tag</th><th>length</th><th>payload</th><th>func</th></tr></thead><tbody><tr><td>00 01</td><td>00 08</td><td>00 00 01 7a ad 34 b7 af</td><td>sub_AC214</td></tr><tr><td>00 02</td><td>00 0a</td><td>68 68 68 68 68 68 69 69 69 68</td><td>sub_AD1D0</td></tr><tr><td>00 03</td><td>00 04</td><td>01 00 00 01</td><td>sub_AC214</td></tr><tr><td>00 05</td><td>00 04</td><td>01 00 00 01</td><td>sub_AC214</td></tr><tr><td>00 04</td><td>00 04</td><td>00 00 00 00</td><td>sub_AC214</td></tr><tr><td>00 06</td><td>00 04</td><td>01 00 00 04</td><td>sub_AC214</td></tr><tr><td>00 07</td><td>00 04</td><td>01 00 00 02</td><td>sub_AC214</td></tr><tr><td>00 08</td><td>00 04</td><td>01 00 00 03</td><td>sub_AC214</td></tr><tr><td>00 09</td><td>00 20</td><td>ab a3 29 3c be 3f 85 56 42 c6 91 8c 0c e8 e2 a6 06 16 ff 83 47 44 ca c3 26 6d 6f 0c e4 3e c6 64</td><td>sub_AD1D0</td></tr><tr><td>00 0a</td><td>00 10</td><td>ea 36 73 53 c0 ac 2f 60 c7 96 68 ee f1 36 2d 3b</td><td>sub_AD1D0</td></tr><tr><td>00 0b</td><td>00 10</td><td>23 9e 8e 64 9b 2d 17 c7 56 af 13 57 d9 46 5d 41</td><td>sub_AD1D0</td></tr></tbody></table>

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-42-44.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_16-42-44.png)

**第二个参数**是什么呢，通过变化传入参数等，观察结果可以知道是 lrand48 相关的

即原始数据应该是`aaaaaabbba`，它们的 ascii 码加上随机数就是第二个参数了

**第三到八个参数**是什么呢，通过变化传入参数，可以确定任何参数都与它们无关，它们都是固定值

**第九个参数**是什么呢，观察长度，发现它是 64 位的，并且变化随机数、时间都会引起改变

*   修改时间的末尾三位不影响第九个参数
*   修改 getByte 除`obj4`外的参数不影响第九个参数

那么说明这个参数和随机数、时间、obj4 相关

obj4 长度 27，随机数看第二个参数应该是有 10 个，时间长度是 8，加起来长度 45

结果是 64 位，那么是 AES 吗，答案是否，因为 AES 结果必然是 16 整数倍，而

> 45 + 16 < 64

所以基本排除掉 AES，那么还有什么是 32 位呢

如果你经验丰富，那么肯定会想到 **SHA256** 在输入参数小于 64 的时候结果必然是 64 位

当然并不排除其他可能性，但目前看 **SHA256** 可能性最高

**第十和十一个参数又是什么呢**，对剩下的函数一一查看，结果如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-12-30.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-12-30.png)

可以看出基本上和最后三个参数没有什么关系，32 位的话那么可能的算法就很多了，最值得怀疑的当然是最常见的 md5 了

虽然暂时不知道第九、十、十一参数算法，但是到这里`sub_ABDBC`完成分析了

那么现在应该看`sub_ABD9C`之前的函数了，即`sub_139A4`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-17-20.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-17-20.png)

```
Interceptor.attach(base_addr.add(0x139A4).add(1), {
    onLeave: function (retval) {
        console.log("sub_139A4 retval", retval.readPointer());
    }
});

```

通过测试发现它的返回结果是`sub_ABDBC`的参数一，也就是放最终结果的地址

反编译代码中也没有其他特殊处理，那么可以跳过这个函数了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-22-20.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-22-20.png)

然后看`sub_AFAC4`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-25-55.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-25-55.png)

也和结果没有什么关系

再往前看，可以发现是一个比较长的调用，这个函数就是`sub_5BF0`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-26-58.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-26-58.png)

该函数主要调用了三个函数，`sub_6794`和`sub_68DC`和`sub_691C`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-28-53.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-28-53.png)

先简单看下对应的反编译代码，发现前两个是 jni 相关的调用，最后一个`sub_691C`比较复杂

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-37-30.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-37-30.png)

不管那么多，先把`sub_691C`的参数 hook 一下看看

然后并没有发现什么特别的东西...

进入`sub_691C`很快就会进入一个非常长的调用，结合之前猜测的 SHA256 和 md5，这里极有可能是两者之一

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-42-43.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-42-43.png)

是时候祭出 [**findhash**](https://github.com/Pr0214/findhash) 了，运行插件，最终结果如下

```
***************************在二进制文件中检索hash算法常量************************************
0xc9d3c:padding used in hashing algorithms (0x80 0 ... 0)
0xbd8bc:SHA256 / SHA224 K tabke
0x33ff9:函数sub_33FF8疑似哈希函数运算部分。
0x68d09:函数sub_68D08疑似哈希函数，包含初始化魔数的代码。
0x82485:函数sub_82484疑似哈希函数，包含初始化魔数的代码。
0x99a35:函数sub_99A34疑似哈希函数，包含初始化魔数的代码。
0xaae89:函数sub_AAE88疑似哈希函数，包含初始化魔数的代码。
***************************存在以下可疑的字符串************************************
0xbd1a0:/proc/self/cmdline
0xbd56c:/proc/self/cmdline
0xbda25:/proc/self/cmdline
***********************************************************************************
花费 41.270039081573486 秒，因为会对全部函数反编译，所以比较耗时间哈

```

对比调用日志，发现`sub_68D08`和`sub_82484`就在其中

另外`sub_AAE88`已经分析过了，这里检测到的应该是 **CRC32**

先看看大量被调用的`sub_BB1A0`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-51-28.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-51-28.png)

原来和算法无关... 那把它去除掉

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-49-18.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-49-18.png)

`sub_82484`离`sub_691C`不是很远，`sub_68D08`则稍微后面一点

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-55-24.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-55-24.png)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-56-15.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-56-15.png)

那现在优先看看它们，`sub_82484`这显然是 SHA256 的魔数

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-57-30.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_17-57-30.png)

为什么这么说呢，对比标准的 SHA256 算法就知道了嘛...

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-00-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-00-09.png)

经过对比可以知道 init 这里的魔数有些不一样

插件还提示了`0xbd8bc:SHA256 / SHA224 K tabke`，看一下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-01-32.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-01-32.png)

经过对比发现 K 表没有改变，另外发现 K 表就在 init 之后被使用

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-03-24.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-03-24.png)

基本上可以推测是只改了魔数的 SHA256 算法，现在看看`sub_862B4`

对比下一般的调用，是不是一模一样呢

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-08-06.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-08-06.png)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-07-07.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-07-07.png)

把数据和返回 hook 一下

```
Interceptor.attach(base_addr.add(0x82648).add(1), {
    onEnter: function (args) {
        console.log(`call sub_82648`);
        console.log("arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())))
        console.log("arg_2", args[2].toUInt32())
    }
});
Interceptor.attach(base_addr.add(0x84890).add(1), {
    onEnter: function (args) {
        this.arg_1 = args[1];
    },
    onLeave: function (retval) {
        console.log(`sub_84890 onLeave`);
        console.log("arg_1", hexdump(this.arg_1))
    }
});

```

确认无误

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-15-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-15-09.png)

算法还原参考 **SHA256 算法还原**小节，确认是仅修改了魔数的 SHA256 算法

但是传入数据显然还不是明文，来看看`sub_862B4`的调用函数`sub_864F0`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-24-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-24-09.png)

可以看到只有两个 memcpy 动作，应该不涉及数据的修改

再看`sub_864F0`的调用函数`sub_85A40`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-26-09.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-26-09.png)

没错，连续调用了两次`sub_864F0`，而`sub_864F0`内会调用`sub_862B4`等做 SHA256 运算

再看下 hook 结果，可以看出第一轮的 SHA256 结果被放到了第二轮 SHA256 的明文末尾

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-28-30.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-28-30.png)

另外第一轮明文和第二轮明文中有大量重复的字符，是`sub_85A40`里面的明文字符串

```
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\6666666666666666666666666666666666666666666666666666666666666666

```

这两个字符的 ASCII 分别是 0x5C 和 0x36

另外还能观察到第二轮明文里面的重复字符前面是 0x14 长度，刚好是 10 位

综合上述信息，可以推测实际上这是一个`HMAC-SHA256`算法，搜一个标准实现

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-44-37.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-44-37.png)

这个可能看起来不太清晰，看这个

*   [https://github.com/bitcoin/bitcoin/blob/master/src/crypto/hmac_sha256.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/crypto/hmac_sha256.cpp)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-47-11.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-47-11.png)

另外还有部分异或操作，另外可以确定没有走这里的`sub_862B4`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-52-42.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_18-52-42.png)

现在看看传入参数，可以看到`sub_85A40`的 a2 应该是有内容的

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-07-42.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-07-42.png)

并且后面和 v20 相与，根据这一点来看，属于稍稍修改过的`HMAC-SHA256`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-09-25.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-09-25.png)

看看`sub_85A40`的 a2 是什么吧

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-11-36.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-11-36.png)

刚好它是 0x14 长度，这和前面提到的那个内容匹配上了

再看看其他参数

```
Interceptor.attach(base_addr.add(0x85A40).add(1), {
    onEnter: function (args) {
        console.log(`call sub_85A40`);
        console.log("sub_85A40 arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())));
        console.log("sub_85A40 arg_2", args[2]);
        console.log("sub_85A40 arg_3", hexdump(args[3].readByteArray(args[4].toUInt32())));
        console.log("sub_85A40 arg_4", args[4]);
    }
});

```

出现了熟悉的内容，a2 应该是 key，a4 应该是明文

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-25-21.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-25-21.png)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-27-26.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-27-26.png)

综合上述信息，可以进行还原了，具体参见 **HMAC-SHA256 算法还原**小节

至此参数九算法还原完成

上面都是基于`sub_82484`函数所推导的内容，现在可以看`sub_68D08`了，毕竟它也是被怀疑的哈希函数

查看反编译代码，显然这应该是一个 md5 函数，并且没有更改魔数

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-34-13.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-34-13.png)

根据调用记录和已有信息，有理由推断就是这两个位置计算了出 md5 作为最终结果的

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-35-30.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-35-30.png)

查看`sub_9DB88`，又是熟悉的样式

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-39-33.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-39-33.png)

```
Interceptor.attach(base_addr.add(0x68E44).add(1), {
    onEnter: function (args) {
        console.log(`call sub_68E44`);
        console.log("sub_68E44 arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())));
        console.log("sub_68E44 arg_2", args[2]);
    }
});
Interceptor.attach(base_addr.add(0x6D628).add(1), {
    onEnter: function (args) {
        this.arg_0 = args[0];
    },
    onLeave: function (retval) {
        console.log("sub_6D628 retval", hexdump(this.arg_0));
    }
});

```

看到了 md5 明文和返回结果

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-44-50.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-44-50.png)

经过验证确定是标准的 md5

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-47-38.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-47-38.png)

那么现在的问题就剩 md5 明文怎么来的了，接着看`sub_9DB88`调用

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-50-54.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-50-54.png)

先看`sub_A70F4`，反编译代码如下，那么接着看`sub_A605C`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-51-57.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_19-51-57.png)

对它的参数进行 hook

```
Interceptor.attach(base_addr.add(0xA605C).add(1), {
    onEnter: function (args) {
        console.log(`call sub_A605C`);
        this.arg_6 = args[6];
        console.log("sub_A605C arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())));
        console.log("sub_A605C arg_2", args[2]);
        console.log("sub_A605C arg_3", args[3]);
        console.log("sub_A605C arg_4", args[4]);
        console.log("sub_A605C arg_5", args[5]);
        console.log("sub_A605C arg_6", hexdump(args[6]));
        console.log("sub_A605C arg_7", hexdump(args[7].readByteArray(args[8].toUInt32())));
        console.log("sub_A605C arg_8", args[8]);
    },
    onLeave: function (retval) {
        console.log("sub_A605C retval", hexdump(this.arg_6));
    }
});

```

结果如下，可以看到有一个 0x14 长度的 key，即 HMAC-SHA256 用到的 key

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-01-55.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-01-55.png)

0x23 长度的明文内容，返回结果就是后续 md5 的内容

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-05-12.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-05-12.png)

那么现在参数十一的运算逻辑如下

> 结果 = md5(转换函数 (明文 + key))

`sub_A605C`涉及两个运算，一个是`sub_A4550`，另外是一些位运算

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-08-45.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-08-45.png)

`sub_A4550`反编译结果如下，看起来想某种算法，经过检索，最终可以确定是 **Salsa20** 算法

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-15-12.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-15-12.png)

魔数如下，不过对比标准算法后发现有很大差异

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-17-04.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-17-04.png)

*   [https://botan.randombit.net/doxygen/salsa20_8cpp_source.html](https://botan.randombit.net/doxygen/salsa20_8cpp_source.html)

这个代码看着可能吃力，可以看这个版本的

*   [https://github.com/Daeinar/salsa20/blob/master/salsa.py](https://github.com/Daeinar/salsa20/blob/master/salsa.py)

差异表现为以下几点

*   state 数组的顺序不一样
*   _round 运算内的逻辑不一样，表现为
    
    *   原算法两个数先相加，结果循环左移，左移结果与另外的数异或
    *   魔改算法分两种情况
        
        *   两个数相加 -> 结果与新的数相异或 -> 结果循环左移 -> 最终结果
        *   两个数相加 -> 结果与新的数做`(a & ~b) | (b & ~a)`运算 -> 结果循环左移 -> 最终结果
*   _round 运算结束后，要交换 state 内数据，魔改算法没有这个过程
*   10 轮_round 运算

基于上述结果重写了魔改后的 Salsa20 算法，其余信息参见 **Salsa20 算法还原**小节

至此参数十一算是还原完成了，可能会有疑问，key 怎么来的，不急后面有

现在开始看`sub_9CD48`，也就是对应参数十的计算过程

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-30-56.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-30-56.png)

根据已有信息，可以知道`sub_9DB88`是 md5，那么看`sub_9C1F0`

```
Interceptor.attach(base_addr.add(0x9C1F0).add(1), {
    onEnter: function (args) {
        console.log(`call sub_9C1F0`);
        this.arg_3 = args[3];
        console.log("sub_9C1F0 arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())));
        console.log("sub_9C1F0 arg_2", args[2]);
        console.log("sub_9C1F0 arg_3", hexdump(args[3]));
    },
    onLeave: function (retval) {
        console.log("sub_9C1F0 retval", hexdump(this.arg_3));
    }
});

```

似乎是明文传入，然后直接拿到了后续待 md5 的内容

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-35-45.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-35-45.png)

顺便 hook 下`sub_9B980`

```
Interceptor.attach(base_addr.add(0x9B980).add(1), {
    onEnter: function (args) {
        console.log(`call sub_9B980`);
        this.arg_0 = args[0];
        console.log("sub_9B980 arg_0", hexdump(args[0]));
        console.log("sub_9B980 arg_1", hexdump(args[1].readByteArray(args[2].toUInt32())));
        console.log("sub_9B980 arg_2", args[2]);
    },
    onLeave: function (retval) {
        console.log("sub_9B980 retval", hexdump(this.arg_0));
    }
});

```

又一个熟悉的数据出现了，同时可以看到`sub_9B980`的参数一在函数结束后有了内容

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-39-10.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-39-10.png)

先看看`sub_9C1F0`的转换

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-47-59.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-47-59.png)

可以看到主要运算就是一系列位运算，简单分析如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-50-15.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-50-15.png)

这里比较简单，现在看`sub_9B980`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-52-03.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-52-03.png)

这里和前面类似

综合以上信息编写还原算法如下

```
def init_state(key: list):
    index = 0
    state = [_ for _ in range(256)]
    key = (key * ((256 // len(key)) + 1))[:256]
    for i in range(256):
        index = (index + key[i] + state[i]) % 256
        state[i], state[index] = state[index], state[i]
    return state


def enccypt_data(ori_data: bytes, key: list):
    state = init_state(key)
    index_1 = 0
    index_2 = 0
    _ori_data = [0] * len(ori_data)
    for offset, num in enumerate(list(ori_data)):
        index_1 = (index_1 + 1) % 256
        index_2 = (index_2 + state[index_1]) % 256
        tmp = state[index_1]
        state[index_1] = state[index_2]
        state[index_2] = tmp
        key_index = (state[index_2] + state[index_1]) & 0xff
        _ori_data[offset] = (num & ~state[key_index]) | (state[key_index] & ~num)
    return bytes(_ori_data)


if __name__ == '__main__':
    import binascii
    import hashlib
    key = binascii.a2b_hex('4f274c3f286b54372a40612428095143565e3140')
    data = binascii.a2b_hex('313632363430333535312c6e303033396579316d6d642c6e756c6c0000017aad34b7af')
    enc_data = enccypt_data(data, key)
    assert 'ea367353c0ac2f60c79668eef1362d3b' == hashlib.new('md5', enc_data).hexdigest(), "测试失败"
    print('测试成功')

```

至此参数十也完成还原，现在就剩一个问题，`4f274c3f286b54372a40612428095143565e3140`从何处来

已知进行 HMAC-SHA256 转换时就出现了这个 key 了，key 是`sub_85A40`的传入参数，那么接着它往前看

结果它的调用函数`sub_86CD4`再向前似乎不太一样了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-04-02.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-04-02.png)

这是怎么回事呢

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-05-16.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-05-16.png)

先看看调用日志，`sub_86CD4`同级前一个函数是`sub_8D970`

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-05-57.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-05-57.png)

果然这里暗藏玄机

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-11-08.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-11-08.png)

`sub_8DB6C`内部则是另一个跳转，显然这里没有跳，而是跳转到`sub_86CD4`了

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-12-24.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-12-24.png)

看`sub_8D970`的调用处，结合上面的分析，这里 v16 经过`sub_8D970`后应该是拿到了`sub_86CD4`地址，但是实际没有产生调用

根据下面的 v7 定位到调用处

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-21-52.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-21-52.png)

如图

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-24-04.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-24-04.png)

通过 hook`sub_86CD4`参数可以知道其 a2 就是 key

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-25-53.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-25-53.png)

那么应该定位 ptr 来源

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-27-23.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-27-23.png)

然后能找到一个相关的地方

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-28-51.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-28-51.png)

对`sub_87DD8`参数打印，结果如下

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-32-41.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-32-41.png)

好起来了！参数二是明文，参数三在函数结束后就是 key

`sub_87DD8`内两处关键位置，注意到`sub_89BF4`两次调用，且有一个参数是 10

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-35-42.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-35-42.png)

通过 hook`sub_89BF4`的参数，可以发现该函数的功能是将传参数二末尾两位数据移动到头部

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-41-45.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-41-45.png)

然后搜索 hex 发现它是一个硬编码的 key

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-44-59.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-44-59.png)

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-45-59.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-45-59.png)

是的，它在这里

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-46-43.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-46-43.png)

IDA 看半天也看不出来参数怎么来的

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-52-56.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-52-56.png)

用 cutter 看看

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-55-46.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_22-55-46.png)

很好！

结合`sub_87DD8`末尾的位运算，现在可以编写 key 转换代码了

```
def gen_key(rand_data: bytes = b'hhhhhhiiih'):
    data = list(binascii.a2b_hex('286b54372a4061244c3f'))
    init_data_1 = data[-2:] + data[:-2]
    data = list(binascii.a2b_hex('392b3e365829264f4061'))
    init_data_2 = data[-2:] + data[:-2]
    _rand_data = [num_1 ^ num_2 for num_1, num_2 in zip(list(rand_data), list(init_data_2))]
    init_data = _rand_data[-2:] + init_data_1 + _rand_data[:-2]
    print('key', bytes(init_data).hex())
    return init_data

```

### SHA256 算法还原

*   [https://github.com/keanemind/Python-SHA-256](https://github.com/keanemind/Python-SHA-256)

将这个标准 SHA256 算法中的魔数修改，测试一下

修改部分伪代码如下

```
def generate_hash(message: bytearray) -> bytearray:
    
    h0 = 0x6A09E669
    h1 = 0xBB67AE87
    h2 = 0x3C6BF372
    h3 = 0xA54FF53A
    h5 = 0x9B25688C
    h4 = 0x511E527F
    h6 = 0x1F73D9AB
    h7 = 0x5BD0CD19
    

if __name__ == '__main__':
    import binascii
    data = binascii.a2b_hex(b'137b10637437086b761c3d7874550d1f0a026d1c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5caac3b82136d0d55907e5607db20a7da7e996d5d083c1edaa1c27252212c954ff')
    assert 'aba3293cbe3f855642c6918c0ce8e2a60616ff834744cac3266d6f0ce43ec664' == generate_hash(data).hex(), '测试失败'
    print('测试成功')

```

测试通过

### HMAC-SHA256 算法还原

```
def hmac_generate_hash(init_data: bytes, ori_data: bytes):
    init_data_1 = [0x36] * 64
    init_data_2 = [0x5C] * 64
    for index, (num_1, num_2) in enumerate(zip(init_data, init_data_1)):
        init_data_1[index] = num_1 ^ num_2
    for index, (num_1, num_2) in enumerate(zip(init_data, init_data_2)):
        init_data_2[index] = ((num_1 & 0xE5) + (~num_1 & 0x1A)) ^ ((num_2 & 0xE5) + (~num_2 & 0x1A))
    tmp_result = generate_hash(bytes(init_data_1) + ori_data)
    result = generate_hash(bytes(init_data_2) + tmp_result)
    return result

if __name__ == '__main__':
    import binascii
    key = binascii.a2b_hex('4f274c3f286b54372a40612428095143565e3140')
    data = binascii.a2b_hex('313632363430333535312c6e303033396579316d6d642c6e756c6c0000017aad34b7af')
    assert 'aba3293cbe3f855642c6918c0ce8e2a60616ff834744cac3266d6f0ce43ec664' == hmac_generate_hash(key, data).hex(), '测试失败'
    print('测试成功')

```

测试通过

### Salsa20 算法还原

还原不易，相信看到这里的人应该都会了，就贴个关键部分截图吧

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-26-32.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_20-26-32.png)

返回结果构成总结
--------

表中的明文由`getByte传入参数obj4`和`gettimeofday返回结果`构成

hex 如下

> 313632363430333535312c6e303033396579316d6d642c6e756c6c0000017aad34b7af

<table><thead><tr><th>tag</th><th>length</th><th>payload</th><th>说明</th></tr></thead><tbody><tr><td>-</td><td>-</td><td>68 65 68 61</td><td>固定值</td></tr><tr><td>-</td><td>-</td><td>00 00 00 01 01 00 00 00</td><td>固定值 后半部分由前半部分翻转得到</td></tr><tr><td>-</td><td>-</td><td>00 00 00 00 00</td><td>未修改</td></tr><tr><td>-</td><td>-</td><td>00 00 03 08</td><td>固定值 由 sub_175B4 产生</td></tr><tr><td>-</td><td>-</td><td>00 00 00 00</td><td>固定值</td></tr><tr><td>-</td><td>-</td><td>00 00 00 00</td><td>未修改</td></tr><tr><td>-</td><td>-</td><td>4b 46 ba a2</td><td>后续数据 CRC32</td></tr><tr><td>-</td><td>-</td><td>00 00 00 01</td><td>固定值</td></tr><tr><td>-</td><td>-</td><td>00 00 00 96</td><td>后续数据长度</td></tr><tr><td>00 01</td><td>00 08</td><td>00 00 01 7a ad 34 b7 af</td><td>gettimeofday</td></tr><tr><td>00 02</td><td>00 0a</td><td>68 68 68 68 68 68 69 69 69 68</td><td>10 位 (不完全) 随机种子</td></tr><tr><td>00 03</td><td>00 04</td><td>01 00 00 01</td><td>固定值</td></tr><tr><td>00 05</td><td>00 04</td><td>01 00 00 01</td><td>固定值</td></tr><tr><td>00 04</td><td>00 04</td><td>00 00 00 00</td><td>固定值</td></tr><tr><td>00 06</td><td>00 04</td><td>01 00 00 04</td><td>固定值</td></tr><tr><td>00 07</td><td>00 04</td><td>01 00 00 02</td><td>固定值</td></tr><tr><td>00 08</td><td>00 04</td><td>01 00 00 03</td><td>固定值</td></tr><tr><td>00 09</td><td>00 20</td><td>ab a3 29 3c be 3f 85 56 42 c6 91 8c 0c e8 e2 a6 06 16 ff 83 47 44 ca c3 26 6d 6f 0c e4 3e c6 64</td><td>HMAC-SHA256(随机种子 key, 明文)</td></tr><tr><td>00 0a</td><td>00 10</td><td>ea 36 73 53 c0 ac 2f 60 c7 96 68 ee f1 36 2d 3b</td><td>md5(数据交换运算 (随机种子 key, 明文))</td></tr><tr><td>00 0b</td><td>00 10</td><td>23 9e 8e 64 9b 2d 17 c7 56 af 13 57 d9 46 5d 41</td><td>md5(明文 ^ (Salsa20(随机种子 key, 固定数据)))</td></tr></tbody></table>

补充

[![](https://blog.seeflower.dev/images/Snipaste_2021-07-16_23-11-24.png)](https://blog.seeflower.dev/images/Snipaste_2021-07-16_23-11-24.png)

其他部分固定值在`sub_AAE88`产生，就不展开了

*   [https://gist.github.com/SeeFlowerX/373101e86529ae04807f634b87ac4c7c](https://gist.github.com/SeeFlowerX/373101e86529ae04807f634b87ac4c7c)

感谢龙哥！