> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-273450.htm)

> Frida 工作原理学习（1）

frida 介绍
========

frida 是一款便携的, 自由的, 支持全平台的, hook 框架，可以通过编写 JavaScript,Python 代码来和 frida_server 端进行交互, 还记得当年用 xposed 时那种写了一大堆代码每次修改都要重新打包安装重启手机, 那种调试调到头皮发麻的痛苦百分之 30 的时间都是在那里安装重启安装重启直到有一天遇到了小甜甜。

frida 的代码结构
===========

```
frida-core: Frida 核心库
frida-gum: inline-hook 框架
bindings:
frida-python: python
frida-node: Node.js
frida-qml: Qml
frida-swift: Swift
frida-tools: CLI tools
capstone: instruction disammbler

```

Frida 的核心是 c 编写的有多种语言绑定例如 Node.js、 Python、 Swift、 .NET、 Qml  
一般我们都使用 js 去编写 frida 脚本因为 js 的异常处理机制非常棒相比于其他语言更高效好用。

frida-core
----------

frida-core 的功能有进程注入、进程间通信、会话管理、脚本生命周期管理等功能，屏蔽部分底层的实现细节并给最终用户提供开箱即用的操作接口。而这一切的实现都在 frida-core 之中，正如名字所言，这其中包含了 frida 相关的大部分关键模块和组件，比如 frida-server、frida-gadget、frida-agent、frida-helper、frida-inject 以及之间的互相通信底座。

frida-gum
---------

frida-gum 是基于 inline-hook 实现的他还有很多丰富的功能比如用于代码跟踪 Stalker、用于内存访问监控的 MemoryAccessMonitor，以及符号查找、栈回溯实现、内存扫描、动态代码生成和重定位等。

### Interceptor

Interceptor 是 inline-hook 的封装

```
GumInterceptor * interceptor;
GumInvocationListener * listener;
gum_init ();
interceptor = gum_interceptor_obtain ();
//GumInvocationListener*的接口
listener = g_object_new (EXAMPLE_TYPE_LISTENER, NULL);
 
// 开始 hook `open` 函数
gum_interceptor_begin_transaction (interceptor);
gum_interceptor_attach_listener (interceptor,
      GSIZE_TO_POINTER (gum_module_find_export_by_name (NULL, "open")),
      listener,
      GSIZE_TO_POINTER (EXAMPLE_HOOK_OPEN));
gum_interceptor_end_transaction (interceptor);
 
// 测试 hook 效果
close (open ("/etc/hosts", O_RDONLY));
 
// 结束 hook
gum_interceptor_detach_listener (interceptor, listener);
g_object_unref (listener);
g_object_unref (interceptor);

```

### Stalker

潜行者又称为尾行痴汉，可以实现指定线程中所有函数、所有基本块、甚至所有指令的跟踪但是有很大的缺点比如在 32 位或者 thumb 下问题很大, 一般想使用指令跟踪都是使用内存断点或者 unidbg 模拟执行 so 但是有很多问题，内存断点的反调试倒是很容易解决但是性能是一个很大的缺陷代码触发断点后会先中断到内核态，然后再返回到用户态 (调试器) 执行跟踪回调，处理完后再返回内核态，然后再回到用户态继续执行，这来来回回的黄花菜都凉了。但 Unidbg 的使用门槛动不动就补环境，龙哥说样本和 Unidbg 之间摩擦出的火花才是最迷人的。或者说人话——“他妈的 Unidbg 怎么又报错了，我该怎么办？”

 

Stalker 的简单使用

```
Interceptor.attach(addr, {
       onEnter: function (args) {
           this.args0 = args[0];
           this.tid = Process.getCurrentThreadId();
           //跟随
           Stalker.follow(this.tid, {
               events: {//事件
                   call: true,//呼叫
                   ret: false,//返回
                   exec: true,//执行
                   block: false,//块
                   compile: false//编译
               },
               //接收
               onReceive(events){
                   for (const [index,value] of Stalker.parse(events)) {
                       console.log(index,value);
                       //findModuleByAddress    {"name":"libc.so","base":"0x7d1f0af000","size":3178496,"path":"/apex/com.android.runtime/lib64/bionic/libc.so"}
                       //console.log("tuzi",Process.findModuleByAddress(0x7d1f13adb8));
 
                   }
               }
               // onCallSummary(summay){
               //console.log("onCallSummary"+JSON.stringify(summay));
               // },
           });
       }, onLeave: function (retval) {
           Stalker.unfollow(this.tid);
       }
   });

```

Stalker 也可以用来还原 ollvm 混淆 记录函数的真实执行地址结合 ida 反汇编没执行的代码都 nop 掉可以很大程度上帮助辅助混淆算法分析当然可能不太准确但也是一种非常棒的思路。  
Stalker 的功能实现，在线程即将执行下一条指令前，先将目标指令拷贝一份到新建的内存中，然后在新的内存中对代码进行插桩，如下图所示:  
![](https://bbs.pediy.com/upload/attach/202206/928079_YEHQKX7ZAKAH3UQ.png)  
这其中使用到了代码动态重编译的方法，好处是原本的代码没有被修改，因此即便代码有完整性校验也不影响，另外由于执行过程都在用户态，省去了多次中断内核切换，性能损耗也达到了可以接受的水平。由于代码的位置发生了改变，如前文 Interceptor 一样，同样要对代码进行重定位的修复

### 内存监控

MemoryAccessMonitor 可以实现对指定内存区间的访问监控，在目标内存区间发生读写行为时可以触发用户指定的回调函数。

 

通过阅读源码发现这个功能的实现方法非常简洁，本质上是将目标内存页设置为不可读写，这样在发生读写行为时会触发事先注册好的中断处理函数，其中会调用到用户使用 gum_memory_access_monitor_new 注册的回调方法中。

```
//C 代码
gboolean
gum_memory_access_monitor_enable (GumMemoryAccessMonitor * self,
                                  GError ** error)
{
  if (self->enabled)
    return TRUE;
  // ...
  self->exceptor = gum_exceptor_obtain ();
  gum_exceptor_add (self->exceptor, gum_memory_access_monitor_on_exception,
      self);
  // ...
}

```

```
//js代码
function read_write_break(){
    function hook_dlopen(addr, soName, callback) {
        Interceptor.attach(addr, {
            onEnter: function (args) {
                var soPath = args[0].readCString();
                if(soPath.indexOf(soName) != -1) hook_call_constructors();
            }, onLeave: function (retval) {
            }
        });
    }
    var dlopen = Module.findExportByName("libdl.so", "dlopen");
    var android_dlopen_ext = Module.findExportByName("libdl.so", "android_dlopen_ext");
    hook_dlopen(dlopen, "libaes.so", set_read_write_break);
    hook_dlopen(android_dlopen_ext, "libaes.so", set_read_write_break);
 
    function set_read_write_break(){
        //实现一个异常回调   处理好这个异常就可以正常返回
        Process.setExceptionHandler(function(details) {
            console.log(JSON.stringify(details, null, 2));
            console.log("lr", DebugSymbol.fromAddress(details.context.lr));
            console.log("pc", DebugSymbol.fromAddress(details.context.pc));
            Memory.protect(details.memory.address, Process.pointerSize, 'rwx');
            console.log(Thread.backtrace(details.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n') + '\n');
            return true;
        });
        var addr = Module.findBaseAddress("libaes.so").add(0x6666);
        Memory.protect(addr, 8, '---'); //修改内存页的权限
        /**
         * 比如有一个地址是0x12345678  我想看一下是那个代码去访问了这个地址
         * 我只需要把这个内存地址置空 有函数去访问这个地址时 就会触发非法访问异常
         * 比较鸡肋这种方法 这种方法会一次修改一个内存页  并且触发一次就无效了
         */
    }
}

```

hook 原理
-------

```
1.注入进程
ptrace
dlopen
2.hook 目标函数
2.1 Java Hook
Static Field Hook：静态成员hook
Method Hook：函数hook
2.2 Native So Hook
GOT Hook：全局偏移表hook
SYM Hook：符号表hook
Inline Hook：函数内联hook
执行自身代码
获取敏感信息
修改返回值
etc.

```

frida 注入的主要思路就是找到目标进程, 使用 ptrace 跟踪目标进程获取 mmap，dlpoen，dlsym 等函数库的便宜获取 mmap 在目标进程申请一段内存空间将在目标进程中找到存放 [frida-agent-32/64.so] 的空间启动执行各种操作由 agent 去实现。

### hook java 层

frida 的 hook 区分了 art 模式和 dalvik 模式。

#### dalvik 模式

把 java 函数变成 native 函数，然后修改入口信息为自定义函数信息。  
![](https://bbs.pediy.com/upload/attach/202206/928079_8GHGUZQJKC9BMD5.png)

```
struct Method {  
    ClassObject*    clazz;   /* method所属的类 public、native等*/
    u4              accessFlags; /* 访问标记 */
    u2             methodIndex; //method索引
    //三个size为边界值，对于native函数，这3个size均等于参数列表的size
    u2              registersSize;  /* ins + locals */
    u2              outsSize;
    u2              insSize;
    const char*     name;//函数名称
    /*
     * Method prototype descriptor string (return and argument types)
     */
    DexProto        prototype;
    /* short-form method descriptor string */
    const char*     shorty;
    /*
     * The remaining items are not used for abstract or native methods.
     * (JNI is currently hijacking "insns" as a function pointer, set
     * after the first call.  For internal-native this stays null.)
     */
    /* the actual code */
    const u2*       insns;          /* instructions, in memory-mapped .dex */
    /* cached JNI argument and return-type hints */
    int             jniArgInfo;
    /*
     * Native method ptr; could be actual function or a JNI bridge.  We
     * don't currently discriminate between DalvikBridgeFunc and
     * DalvikNativeFunc; the former takes an argument superset (i.e. two
     * extra args) which will be ignored.  If necessary we can use
     * insns==NULL to detect JNI bridge vs. internal native.
     */
    DalvikBridgeFunc nativeFunc;
    /*
     * Register map data, if available.  This will point into the DEX file
     * if the data was computed during pre-verification, or into the
     * linear alloc area if not.
     */
    const RegisterMap* registerMap;
 
};
 
…
…
…
 
function replaceDalvikImplementation (fn) {
  if (fn === null && dalvikOriginalMethod === null) {
    return;
  }
//备份原来的method,
  if (dalvikOriginalMethod === null) {
    dalvikOriginalMethod = Memory.dup(methodId, DVM_METHOD_SIZE);
    dalvikTargetMethodId = Memory.dup(methodId, DVM_METHOD_SIZE);
  }
 
  if (fn !== null) {
   //自定的代码
    implementation = implement(f, fn);
 
    let argsSize = argTypes.reduce((acc, t) => (acc + t.size), 0);
    if (type === INSTANCE_METHOD) {
      argsSize++;
    }
    // 把method变成native函数
    /*
     * make method native (with kAccNative)
     * insSize and registersSize are set to arguments size
     */
    const accessFlags = (Memory.readU32(methodId.add(DVM_METHOD_OFFSET_ACCESS_FLAGS)) | kAccNative) >>> 0;
    const registersSize = argsSize;
    const outsSize = 0;
    const insSize = argsSize;
 
    Memory.writeU32(methodId.add(DVM_METHOD_OFFSET_ACCESS_FLAGS), accessFlags);
    Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_REGISTERS_SIZE), registersSize);
    Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_OUTS_SIZE), outsSize);
    Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_INS_SIZE), insSize);
    Memory.writeU32(methodId.add(DVM_METHOD_OFFSET_JNI_ARG_INFO), computeDalvikJniArgInfo(methodId));
    //调用dvmUseJNIBridge为这个Method设置一个Bridge,本质上是修改结构体中的nativeFunc为自定义的implementation函数
    api.dvmUseJNIBridge(methodId, implementation);
 
    patchedMethods.add(f);
  } else {
    patchedMethods.delete(f);
 
    Memory.copy(methodId, dalvikOriginalMethod, DVM_METHOD_SIZE);
    implementation = null;
  }
}

```

#### art 模式

art 模式也是需要将 java 函数变成 native 函数但是不同于 dalvik，art 下有两种解释器一种汇编解释器一种 smali 解释器

```
quick code 模式：执行 arm 汇编指令
Interpreter 模式：由解释器解释执行 Dalvik 字节码

```

![](https://bbs.pediy.com/upload/attach/202206/928079_ZRDSMC7S2VH5CDY.png)  
1. 如果函数已经存在 quick code, 则指向这个函数对应的 quick code 的起始地址，而当 quick code 不存在时，它的值则会代表其他的意义。

 

2. 当一个 java 函数不存在 quick code 时，它的值是函数 artQuickToInterpreterBridge 的地址，用以从 quick 模式切换到 Interpreter 模式来解释执行 java 函数代码。

 

3. 当一个 java native（JNI）函数不存在 quick code 时，它的值是函数 art_quick_generic_jni_trampoline 的地址，用以执行没有 quick code 的 jni 函数。

 

所以 frida 要将 java method 转为 native method，需要将 ARTMethod 结构进行如下修改：

```
patchMethod(methodId, {
  //jnicode入口entry_point_from_jni_改为自定义的代码
  'jniCode': implementation,
  //修改为access_flags_为native
  'accessFlags': (Memory.readU32(methodId.add(artMethodOffset.accessFlags)) | kAccNative | kAccFastNative) >>> 0,
  //art_quick_generic_jni_trampoline函数的地址
  'quickCode': api.artQuickGenericJniTrampoline,
  //artInterpreterToCompiledCodeBridge函数地址
  'interpreterCode': api.artInterpreterToCompiledCodeBridge
});

```

参考文献
====

```
https://evilpan.com/2022/04/05/frida-internal/
https://blog.drov.com.cn/2021/04/hook.html
https://bbs.pediy.com/thread-229215.htm
https://frida.re/docs/home/
https://www.youtube.com/watch?v=uc1mbN9EJKQ

```

[【看雪培训】《Adroid 高级研修班》2022 年夏季班招生中！](https://bbs.pediy.com/thread-271992.htm)

[#基础理论](forum-161-1-117.htm)