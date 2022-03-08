> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.seeflower.dev](https://blog.seeflower.dev/archives/67/)

> Be patient.

本文章转载自龙哥，已获许可，持续更新中。

**本文总结了 Unidbg Hook and Call 的知识，部分 Hook 代码采用 Frida 与 Unidbg 对照的方式，帮助熟悉 Frida 但不熟悉 Unidbg 的读者快速入门。**

样例前往百度云下载：

链接：[https://pan.baidu.com/s/1ZRPtQrx4QAPEQhrpq6gbgg](https://pan.baidu.com/s/1ZRPtQrx4QAPEQhrpq6gbgg) 提取码：6666

**更多 Unidbg 使用和算法还原的教程可见星球。**

[![](https://blog.seeflower.dev/images/1.png)](https://blog.seeflower.dev/images/1.png)

[TOC]

一、基础知识
------

### 1. 获取 SO 基地址

#### Ⅰfrida 获取基地址

```
var baseAddr = Module.findBaseAddress("libnative-lib.so");

```

#### Ⅱ Unidbg 获取基地址

```
DalvikModule dm = vm.loadLibrary("libnative-lib.so", true);

module = dm.getModule();

System.out.println("baseAddr:"+module.base);

```

加载了多个 SO 的情况

```
Module yourModule = emulator.getMemory().findModule("yourModuleName");

System.out.println("baseAddr:"+yourModule.base);

```

如果只主动加载一个 SO，其基址恒为 0x40000000 , 这是一个检测 Unidbg 的点，可以在 com/github/unidbg/memory/Memory.java 中做修改

```
public interface Memory extends IO, Loader, StackMemory {

    long STACK_BASE = 0xc0000000L;
    int STACK_SIZE_OF_PAGE = 256; 

    
    long MMAP_BASE = 0x40000000L;

    UnidbgPointer allocateStack(int size);
    UnidbgPointer pointer(long address);
    void setStackPoint(long sp);

```

### 2. 获取函数地址

#### Ⅰ Frida 获取导出函数地址

```
Module.findExportByName("libc.so", "strcmp")

```

#### Ⅱ Unidbg 获取导出函数地址

```
DalvikModule dm = vm.loadLibrary("libnative-lib.so", true);

module = dm.getModule();
int address = (int) module.findSymbolByName("funcNmae").getAddress();

```

#### Ⅲ Frida 获取非导出函数地址

```
var soAddr = Module.findBaseAddress("libnative-lib.so");
var FuncAddr = soAddr.add(0x1768 + 1);

```

#### Ⅳ Unidbg 获取非导出函数地址

```
DalvikModule dm = vm.loadLibrary("libnative-lib.so", true);

module = dm.getModule();

int offset = 0x1768;

int address = (int) (module.base + offset);

```

### 3.Unidbg Hook 大盘点

Unidbg 在 Android 上支持的 Hook，可以分为两大类

*   Unidbg 内置的第三方 Hook 框架，包括 xHook/Whale/HookZz
*   Unicorn Hook 以及 Unidbg 基于它封装的 Console Debugger

**第一类是 Unidbg 支持并内置的第三方 Hook 框架，有 Dobby(前身 HookZz)/Whale 这样的 Inline Hook 框架，也有 xHook 这样的 PLT Hook 框架。有小伙伴可能困惑 Unidbg 是否能支持 Frida？我个人观点是目前阶段不现实，Frida 比 Dobby 或者 xHook 都复杂的多，Unidbg 目前还跑不通，除此之外，Dobby + Whale + xHook 也绝对够用了，没有非 Frida 不可的需求。**

**第二类是当 Unidbg 的底层引擎选择为 Unicorn 时（默认引擎），Unicorn 自带的 Hook 功能。Unicorn 提供了各种级别和粒度的 Hook，内存 Hook / 指令 / 基本块 Hook / 异常 Hook 等等，十分强大和好用，而且 Unidbg 基于它封装了更便于使用的 Console Debugger。**

该怎么选择 Hook 方案？这得看使用 Unidbg 的目的。如果项目用于**模拟执行**，那么建议使用 Console Debugger 做快速分析，排查错误，跑通代码后用第三方 Hook 框架做持久化，为什么？这得从 Unidbg 支持的汇编执行引擎说起。Unidbg 支持多种底层引擎，最早也是默认的引擎是 Unicorn，从名字也能看出，Unidbg 和 Unicorn 有很大关系。但后续 Unidbg 又支持了数个引擎，任何提高程序复杂度的行为，肯定都为了解决某个痛点。

hypervisor 引擎可以在搭载了 _Apple Silicon_ 芯片的设备上模拟执行；

KVM 引擎可以在树莓派上模拟执行；

Dynarmic 引擎是为了更快的模拟执行；

Unicorn 是最强大最完善的模拟执行引擎，但它相比 Dynarmic 太慢了，同场景下，Dynarmic 比 Unicorn 模拟执行快数倍甚至十数倍。如果使用 Unidbg 是为了实现生产环境下的模拟执行，速度最重要，那么 Dynarmic + **[unidbg-boot-server](https://github.com/anjia0532/unidbg-boot-server)** 这个高并发 server 服务器，是完美之选。一般实操中，先使用 Unicorn 引擎跑通模拟执行代码，切换成 Dynarmic 无误后，直接上生产环境。

_Dynarmic 引擎使用_

```
private static AndroidEmulator createARMEmulator() {
    return AndroidEmulatorBuilder.for32Bit()
            
            .addBackendFactory(new DynarmicFactory(true))
            .build();
}

```

_Unicorn 默认引擎_

```
private static AndroidEmulator createARMEmulator() {
    return AndroidEmulatorBuilder.for32Bit()
            .build();
}

```

使用 Unidbg 的第二个场景是**辅助算法还原**，即模拟执行只是算法还原的前奏，在模拟执行无误后，使用 Unidbg 辅助算法还原。这种情况下自然使用 Unicorn 引擎，两大类 Hook 方案都可以使用，选择哪类？我倾向于自始至终使用第二类方案，即基于 Unicorn Hook 的方案。

我个人认为有三点优势

*   HookZz 或者 xHook 等方案，都可以基于其 Hook 实现原理进行检测，但 Unicorn 原生 Hook 不容易被检测。
*   Unicorn Hook 没有局限，其他方案局限性较大。比如 Inline Hook 方案不能 Hook 短函数，或者两个相邻的地址；PLT Hook 不能 Hook Sub_xxx 子函数。
*   第三方 inline Hook 框架和原生 Hook 方案同时使用时会摩擦出 BUG 的火花，事实上，单使用 Unicorn 的某些 Hook 功能都有 BUG。所以说，统一用原生 Hook 会少一些 BUG，少一些麻烦。

总结如下

#### Ⅰ 以模拟执行为目的

使用第三方 Hook 方案，arm32 下 HookZz 的支持较好，arm64 下 Dobby 的支持较好，HookZz/Dobby Hook 不成功时，如果函数是导出函数，使用 xHook，否则使用 Whale。

#### Ⅱ 以算法还原为目的

使用 Console Debugger 和 Unicorn Hook，并建议不优先使用第三方 Hook 方案。

### 4. 本篇的基础代码

即模拟执行 demo 的代码

```
package com.tutorial;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.HookStatus;
import com.github.unidbg.arm.backend.Backend;
import com.github.unidbg.arm.backend.CodeHook;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.hook.HookContext;
import com.github.unidbg.hook.ReplaceCallback;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.hook.whale.IWhale;
import com.github.unidbg.hook.whale.Whale;
import com.github.unidbg.hook.xhook.IxHook;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.XHookImpl;
import com.github.unidbg.linux.android.dvm.DalvikModule;
import com.github.unidbg.linux.android.dvm.DvmClass;
import com.github.unidbg.linux.android.dvm.DvmObject;
import com.github.unidbg.linux.android.dvm.VM;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import unicorn.ArmConst;
import unicorn.Unicorn;

import java.io.File;

public class hookInUnidbg {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    hookInUnidbg() {

        
        emulator = AndroidEmulatorBuilder.for32Bit().build();

        
        final Memory memory = emulator.getMemory();
        
        memory.setLibraryResolver(new AndroidResolver(23));
        
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/tutorial/hookinunidbg.apk"));




        
        DalvikModule dm = vm.loadLibrary("hookinunidbg", true);
        
        module = dm.getModule();

        
        dm.callJNI_OnLoad(emulator);
    }

    public void call(){
        DvmClass dvmClass = vm.resolveClass("com/example/hookinunidbg/MainActivity");
        String methodSign = "call()V";
        DvmObject<?> dvmObject = dvmClass.newObject(null);

        dvmObject.callJniMethodObject(emulator, methodSign);

    }


    public static void main(String[] args) {
        hookInUnidbg mydemo = new hookInUnidbg();
        mydemo.call();
    }


}

```

运行时有一些日志输出，为正常逻辑。

二、Hook 函数
---------

demo hookInunidbg 中运行了数个函数，在本节中关注其中运行的 base64_encode 函数。

```
unsigned int
base64_encode(const unsigned char *in, unsigned int inlen, char *out);

```

参数解释如下

> char *out：一块 buffer 的首地址，用来存放转码后的内容。
> 
> char *in：原字符串的首地址，指向原字符串内容。
> 
> int inlen：原字符串长度。
> 
> 返回值：正常情况下返回转换后字符串的实际长度。

本节的任务就是打印 base64 前的内容，以及编码后的内容。

### 1.Frida

```
function main(){
    
    var base_addr = Module.findBaseAddress("libhookinunidbg.so");

    if (base_addr){
        var func_addr = Module.findExportByName("libhookinunidbg.so", "base64_encode");
        console.log("hook base64_encode function")
        Interceptor.attach(func_addr,{
            
            onEnter: function (args) {
                console.log("\n input:")
                this.buffer = args[2];
                var length = args[1];
                console.log(hexdump(args[0],{length: length.toUInt32()}))
                console.log("\n")
            },
            
            onLeave: function () {
                console.log(" output:")
                console.log(this.buffer.readCString());
            }
        })
    }


}

setImmediate(main);

```

### 2.Console Debugger

Console Debugger 快速打击、快速验证 的交互调试器

```
emulator.attach().addBreakPoint(module.findSymbolByName("base64_encode").getAddress());

```

需要重申和强调几个概念

*   运行到对应地址时触发断点，类似于 GDB 调试或者 IDA 调试，时机为**目标指令执行前**。
*   断点不具有函数的种种概念，需要从 ARM 汇编指令的角度去理解函数。
*   Console Debugger 用于辅助算法分析，快速分析、确认某个函数的功能。在 Unicorn 引擎下才可以用。

针对第二条做补充

> 根据 ARM ATPCS 调用约定，当参数个数小于等于 4 个的时候，子程序间通过 R0~R3 来传递参数（即 R0-R3 代表参数 1 - 参数 4），如果参数个数大于 4 个，余下的参数通过 sp 所指向的数据栈进行参数传递。而函数的返回值总是通过 R0 传递回来。

以目标函数为例，函数调用前，调用方把三个参数依次放在 R0-R2 中。

[![](https://blog.seeflower.dev/images/2.png)](https://blog.seeflower.dev/images/2.png)

立即数可以直接查看，比如此处的参数 2 是 5。如果怀疑是指针，比如参数 1 和参数 3，交互调试中输入 mxx 查看。mrx 等价于 Frida 中的 hexdump(xxx)。以这里的 r0 为例，既可以 mr0 也可以 m0x400022e0 查看其指向的内存。

Unidbg 在数据展示上，相较于 Frida Hexdump，有一些不同，体现在两方面

*   Frida hexdump 时，左侧基地址从当前地址开始，而 Unidbg 从 0 开始。
*   Unidbg 给出了所打印数据块的 md5 值，方便对比两块数据块内容是否一致，而且 Unidbg 展示数据的 Hex String，方便在大量日志中搜索。

Console Debugger 支持许多调试、分析的命令，全部展示如下

```
c: continue
n: step over
bt: back trace

st hex: search stack
shw hex: search writable heap
shr hex: search readable heap
shx hex: search executable heap

nb: break at next block
s|si: step into
s[decimal]: execute specified amount instruction
s(blx): execute util BLX mnemonic, low performance

m(op) [size]: show memory, default size is 0x70, size may hex or decimal
mr0-mr7, mfp, mip, msp [size]: show memory of specified register
m(address) [size]: show memory of specified address, address must start with 0x

wr0-wr7, wfp, wip, wsp <value>: write specified register
wb(address), ws(address), wi(address) <value>: write (byte, short, integer) memory of specified address, address must start with 0x
wx(address) <hex>: write bytes to memory at specified address, address must start with 0x

b(address): add temporarily breakpoint, address must start with 0x, can be module offset
b: add breakpoint of register PC
r: remove breakpoint of register PC
blr: add temporarily breakpoint of register LR

p (assembly): patch assembly at PC address
where: show java stack trace

trace [begin end]: Set trace instructions
traceRead [begin end]: Set trace memory read
traceWrite [begin end]: Set trace memory write
vm: view loaded modules
vbs: view breakpoints
d|dis: show disassemble
d(0x): show disassemble at specify address
stop: stop emulation
run [arg]: run test
cc size: convert asm from 0x400008a0 - 0x400008a0 + size bytes to c function

```

在 Frida 代码中，用 `console.log(hexdump(args[0],{length: args[1].toUInt32()}))`来表示 **打印参数 1 指向的内存块，以参数 2 为长度** 这样的效果，Unidbg 中同样可以处理长度。

```
mr0 5

>-----------------------------------------------------------------------------<
[23:41:37 891]r0=RX@0x400022e0[libhookinunidbg.so]0x22e0, md5=f5704182e75d12316f5b729e89a499df, hex=6c696c6163
size: 5
0000: 6C 69 6C 61 63                                     lilac
^-----------------------------------------------------------------------------^

```

目前 Console Debugger 还不支持 _mr0 r1_ 这样的语法。

至此实现了 Frida OnEnter 的功能，接下来要获取 OnLeave 的时机点，即函数执行完的时机。在 ARM 汇编中，LR 寄存器存放了程序的返回地址，当函数跑到 LR 所指向的地址时，函数已经结束了。又因为断点是在目标地址执行前触发，所以在 LR 处的断点断下时，目标函数执行完且刚执行完，这就是 Frida OnLeave 时机点的原理。在 Console Debugger 交互调试中，使用 blr 命令可以在 lr 处下一个临时断点，它只会触发一次。

整体逻辑如下

*   在目标函数的地址处下断点
*   运行到断点处，进入 Console Debugger 交互调试
*   mxx 系列查看参数
*   blr 在函数返回处下断点
*   c 使程序继续运行，到返回值处断下
*   查看此时的 buffer

需要注意的是，在 onLeave 中 mr2 是胡闹。R2 只在程序入口处表示参数 3，在函数运算的过程中，R2 作为通用寄存器被用于存储、运算，它已经不是指向 buffer 的地址了。在 Frida 中，我们在 OnEnter 里将 args[2] 即 R2 的值保存在 this.buffer 中，OnLeave 中再取出来打印。而在 Console Debugger 交互调试中，办法更简单粗暴——鼠标往上拉一下，看看原来 r2 的值是什么，发现是 0x401d2000，然后 m0x401d2000。

这样我们就实现了 Frida 的等效功能。听起来有一些麻烦，但熟练后你会认同我的观点——Console Debugger 是最好最快最稳定的调试工具。除此之外，Console Debugger 也可以做持久化的 Hook，代码如下。

```
public void HookByConsoleDebugger(){
    emulator.attach().addBreakPoint(module.findSymbolByName("base64_encode").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext context = emulator.getContext();
            Pointer input = context.getPointerArg(0);
            int length = context.getIntArg(1);
            Pointer buffer = context.getPointerArg(2);

            Inspector.inspect(input.getByteArray(0, length), "base64 input");
            
            emulator.attach().addBreakPoint(context.getLRPointer().peer, new BreakPointCallback() {
                @Override
                public boolean onHit(Emulator<?> emulator, long address) {
                    String result = buffer.getString(0);
                    System.out.println("base64 result:"+result);
                    return true;
                }
            });
            return true;
        }
    });
}

```

onHit 返回 ture 时，断点触发时不会进入交互界面；为 false 时会。当函数被调用了三五百次时，我们不希望它反复停下来，然后不停 “c” 来继续运行。

### 3. 第三方 Hook 框架

如下目标函数均在 JNIOnLoad 前调用

#### ⅠxHook

```
public void HookByXhook(){
    IxHook xHook = XHookImpl.getInstance(emulator);
    xHook.register("libhookinunidbg.so", "base64_encode", new ReplaceCallback() {
        @Override
        public HookStatus onCall(Emulator<?> emulator, HookContext context, long originFunction) {
            Pointer input = context.getPointerArg(0);
            int length = context.getIntArg(1);
            Pointer buffer = context.getPointerArg(2);
            Inspector.inspect(input.getByteArray(0, length), "base64 input");
            context.push(buffer);
            return HookStatus.RET(emulator, originFunction);
        }
        @Override
        public void postCall(Emulator<?> emulator, HookContext context) {
            Pointer buffer = context.pop();
            System.out.println("base64 result:"+buffer.getString(0));
        }
    }, true);
    
    xHook.refresh();
}

```

xHook 是爱奇艺开源的 Android PLT hook 框架，优点是挺稳定好用，缺点是不能 Hook Sub_xxx 子函数。

#### Ⅱ HookZz

```
public void HookByHookZz(){
    IHookZz hookZz = HookZz.getInstance(emulator); 
    hookZz.enable_arm_arm64_b_branch(); 
    hookZz.wrap(module.findSymbolByName("base64_encode"), new WrapCallback<HookZzArm32RegisterContext>() {
        @Override
        public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext context, HookEntryInfo info) {
            Pointer input = context.getPointerArg(0);
            int length = context.getIntArg(1);
            Pointer buffer = context.getPointerArg(2);
            Inspector.inspect(input.getByteArray(0, length), "base64 input");
            context.push(buffer);
        }
        @Override
        public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext context, HookEntryInfo info) {
            Pointer buffer = context.pop();
            System.out.println("base64 result:"+buffer.getString(0));
        }
    });
    hookZz.disable_arm_arm64_b_branch();
}

```

HookZz 也可以实现类似于单行断点的 Hook，但在 Unidbg 的 Hook 大环境下感觉用处不大，不建议使用。

```
IHookZz hookZz = HookZz.getInstance(emulator);
hookZz.instrument(module.base + 0x978 + 1, new InstrumentCallback<RegisterContext>() {
    @Override
    public void dbiCall(Emulator<?> emulator, RegisterContext ctx, HookEntryInfo info) {
        System.out.println(ctx.getIntArg(0));
    }
});

```

HookZz 现在叫 Dobby，Unidbg 中是 HookZz 和 Dobby 是两个独立的 Hook 库，因为作者认为 HookZz 在 arm32 上支持较好，Dobby 在 arm64 上支持较好。HookZz 是 inline hook 方案，因此可以 Hook Sub_xxx，缺点是短函数可能出 bug，受限于 inline Hook 原理。

#### Ⅲ Whale

```
public void HookByWhale(){
    IWhale whale = Whale.getInstance(emulator);
    whale.inlineHookFunction(module.findSymbolByName("base64_encode"), new ReplaceCallback() {
        Pointer buffer;
        @Override
        public HookStatus onCall(Emulator<?> emulator, long originFunction) {
            RegisterContext context = emulator.getContext();
            Pointer input = context.getPointerArg(0);
            int length = context.getIntArg(1);
            buffer = context.getPointerArg(2);
            Inspector.inspect(input.getByteArray(0, length), "base64 input");
            return HookStatus.RET(emulator, originFunction);
        }

        @Override
        public void postCall(Emulator<?> emulator, HookContext context) {
            System.out.println("base64 result:"+buffer.getString(0));
        }
    }, true);
}

```

Whale 是一个跨平台的 Hook 框架，在 Andorid Native Hook 上也是 inline Hook 方案，具体情况我了解的不多。

### 4.Unicorn Hook

如果想对某个函数进行集中的、高强度的、同时又灵活的调试，Unicorn CodeHook 是一个好选择。比如我想查看目标函数第一条指令的 r1，第二条指令的 r2，第三条指令的 r3，类似于这种需求。

hook_add_new 第一个参数是 Hook 回调，我们这里选择 CodeHook，它是逐条指令 Hook，参数 2 是起始地址，参数 3 是结束地址，参数 4 一般填 null。这意味着从起始地址到终止地址这个执行范围内的每条指令，我们都可以在其执行前处理它。

找到目标函数的代码范围

[![](https://blog.seeflower.dev/images/3.png)](https://blog.seeflower.dev/images/3.png)

```
public void HookByUnicorn(){
    long start = module.base+0x97C;
    long end = module.base+0x97C+0x17A;
    emulator.getBackend().hook_add_new(new CodeHook() {
        @Override
        public void hook(Backend backend, long address, int size, Object user) {
            RegisterContext registerContext = emulator.getContext();
            if(address == module.base + 0x97C){
                int r0 = registerContext.getIntByReg(ArmConst.UC_ARM_REG_R0);
                System.out.println("0x97C 处 r0:"+Integer.toHexString(r0));
            }
            if(address == module.base + 0x97C + 2){
                int r2 = registerContext.getIntByReg(ArmConst.UC_ARM_REG_R2);
                System.out.println("0x97C +2 处 r2:"+Integer.toHexString(r2));
            }
            if(address == module.base + 0x97C + 4){

                int r4 = registerContext.getIntByReg(ArmConst.UC_ARM_REG_R4);
                System.out.println("0x97C +4 处 r4:"+Integer.toHexString(r4));
            }
        }

        @Override
        public void onAttach(Unicorn.UnHook unHook) {

        }

        @Override
        public void detach() {

        }
    }, start, end, null);
}

```

三、Replace 参数和返回值
----------------

### 1. 替换参数

需求：如果入参为 lilac，改为 hello world，对应的入参长度也要改。正确结果是 **aGVsbG8gd29ybGQ=**。

#### ⅠFrida

```
function main(){
    
    var base_addr = Module.findBaseAddress("libhookinunidbg.so");

    if (base_addr){
        var func_addr = Module.findExportByName("libhookinunidbg.so", "base64_encode");
        console.log("hook base64_encode function")
        var fakeinput = "hello world"
        var fakeinputPtr = Memory.allocUtf8String(fakeinput);
        Interceptor.attach(func_addr,{
            onEnter: function (args) {
                args[0] = fakeinputPtr;
                args[1] = ptr(fakeinput.length);
                this.buffer = args[2];
            },
            
            onLeave: function () {
                console.log(" output:")
                console.log(this.buffer.readCString());
            }
        })
    }


}

setImmediate(main);

```

#### Ⅱ Console Debugger

快速打击、快速验证的 Console Debugger 如何实现这一目标？

①下断点，运行代码后进入 debugger

```
emulator.attach().addBreakPoint(module.findSymbolByName("base64_encode").getAddress());

```

②通过命令修改参数 1 和 2

```
wx0x40002403 68656c6c6f20776f726c64

>-----------------------------------------------------------------------------<
[14:06:46 165]RX@0x40002403[libhookinunidbg.so]0x2403, md5=5eb63bbbe01eeed093cb22bb8f5acdc3, hex=68656c6c6f20776f726c64
size: 11
0000: 68 65 6C 6C 6F 20 77 6F 72 6C 64                   hello world
^-----------------------------------------------------------------------------^
wr1 11
>>> r1=0xb


```

Console Debugger 支持下列写操作

```
wr0-wr7, wfp, wip, wsp <value>: write specified register
wb(address), ws(address), wi(address) <value>: write (byte, short, integer) memory of specified address, address must start with 0x
wx(address) <hex>: write bytes to memory at specified address, address must start with 0x

```

但这其实并不方便，还是做持久化比较舒服。

```
public void ReplaceArgByConsoleDebugger(){
    emulator.attach().addBreakPoint(module.findSymbolByName("base64_encode").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext context = emulator.getContext();
            String fakeInput = "hello world";
            int length = fakeInput.length();
            
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R1, length);
            MemoryBlock fakeInputBlock = emulator.getMemory().malloc(length, true);
            fakeInputBlock.getPointer().write(fakeInput.getBytes(StandardCharsets.UTF_8));
            
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, fakeInputBlock.getPointer().peer);

            Pointer buffer = context.getPointerArg(2);
            
            emulator.attach().addBreakPoint(context.getLRPointer().peer, new BreakPointCallback() {
                @Override
                public boolean onHit(Emulator<?> emulator, long address) {
                    String result = buffer.getString(0);
                    System.out.println("base64 result:"+result);
                    return true;
                }
            });
            return true;
        }
    });
}

```

#### Ⅲ 第三方 Hook 框架

变来变去的只有外壳。

① xHook

```
public void ReplaceArgByXhook(){
    IxHook xHook = XHookImpl.getInstance(emulator);
    xHook.register("libhookinunidbg.so", "base64_encode", new ReplaceCallback() {
        @Override
        public HookStatus onCall(Emulator<?> emulator, HookContext context, long originFunction) {
            String fakeInput = "hello world";
            int length = fakeInput.length();
            
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R1, length);
            MemoryBlock fakeInputBlock = emulator.getMemory().malloc(length, true);
            fakeInputBlock.getPointer().write(fakeInput.getBytes(StandardCharsets.UTF_8));
            
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, fakeInputBlock.getPointer().peer);

            Pointer buffer = context.getPointerArg(2);
            context.push(buffer);
            return HookStatus.RET(emulator, originFunction);
        }
        @Override
        public void postCall(Emulator<?> emulator, HookContext context) {
            Pointer buffer = context.pop();
            System.out.println("base64 result:"+buffer.getString(0));
        }
    }, true);
    
    xHook.refresh();
}

```

② HookZz

```
public void ReplaceArgByHookZz(){
    IHookZz hookZz = HookZz.getInstance(emulator); 
    hookZz.enable_arm_arm64_b_branch(); 
    hookZz.wrap(module.findSymbolByName("base64_encode"), new WrapCallback<HookZzArm32RegisterContext>() {
        @Override
        public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext context, HookEntryInfo info) {
            Pointer input = context.getPointerArg(0);
            String fakeInput = "hello world";
            input.setString(0, fakeInput);
            context.setR1(fakeInput.length());

            Pointer buffer = context.getPointerArg(2);
            context.push(buffer);
        }
        @Override
        public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext context, HookEntryInfo info) {
            Pointer buffer = context.pop();
            System.out.println("base64 result:"+buffer.getString(0));
        }
    });
    hookZz.disable_arm_arm64_b_branch();
}

```

因为可以用 HookZzArm32RegisterContext，相对来说代码简单一些。

### 2. 修改返回值

修改返回值的逻辑和替换参数并没什么区别，但它可以引出第四节，所以还是仔细讲一下。

在 demo 中，有一个 verifyApkSign 函数，它总是返回 1，并导致 APK 校验失败，因此目标就是让它返回 0。

```
extern "C"
JNIEXPORT void JNICALL
Java_com_example_hookinunidbg_MainActivity_call(JNIEnv *env, jobject thiz) {
    int verifyret = verifyApkSign();
    if(verifyret == 1){
        LOGE("APK sign verify failed!");
    } else{
        LOGE("APK sign verify success!");
    }
    testBase64();
}

extern "C" int verifyApkSign(){
    LOGE("verify apk sign");
    return 1;
};

```

#### ⅠFrida

```
function main(){
    
    var base_addr = Module.findBaseAddress("libhookinunidbg.so");

    if (base_addr){
        var func_addr = Module.findExportByName("libhookinunidbg.so", "verifyApkSign");
        console.log("hook verifyApkSign function")
        Interceptor.attach(func_addr,{
            onEnter: function (args) {

            },
            
            onLeave: function (retval) {
                retval.replace(0);
            }
        })
    }


}

setImmediate(main);

```

#### Ⅱ Console Debugger

```
public void ReplaceRetByConsoleDebugger(){
    emulator.attach().addBreakPoint(module.findSymbolByName("verifyApkSign").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext context = emulator.getContext();
            
            emulator.attach().addBreakPoint(context.getLRPointer().peer, new BreakPointCallback() {
                @Override
                public boolean onHit(Emulator<?> emulator, long address) {
                    emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, 0);
                    return true;
                }
            });
            return true;
        }
    });
}

```

我们的 Hook 生效了，但 verifyApkSign 函数里的 log 还是打印出来了。在一些情况中，我们想改掉函数原本的执行行为，而不是仅仅打印一些信息或者替换入参和返回值。即需要彻底的函数替换——替换原有函数，使用自己的函数。

四、替换函数
------

### 1.Frida

```
const verifyApkSignPtr = Module.findExportByName("libhookinunidbg.so", "verifyApkSign");
Interceptor.replace(verifyApkSignPtr, new NativeCallback(() => {
    console.log("replace verifyApkSign Function")
    return 0;
}, 'void', []));

```

### 2. 第三方 Hook 框架

这里只演示 xHook

```
public void ReplaceFuncByHookZz(){
    HookZz hook = HookZz.getInstance(emulator);
    hook.replace(module.findSymbolByName("verifyApkSign").getAddress(), new ReplaceCallback() {
        @Override
        public HookStatus onCall(Emulator<?> emulator, HookContext context, long originFunction) {
            emulator.getBackend().reg_write(Unicorn.UC_ARM_REG_R0,0);
            return HookStatus.RET(emulator,context.getLR());
        }
    });
}

```

xHook 的版本很清晰易懂，我们做了两件事

*   R0 赋值为 0
*   LR 赋值给 PC，这意味着函数一行不执行就返回了，又因为 R0 赋值 0 所以返回值为 0。

### 3.Console Debugger

```
public void ReplaceFuncByConsoleDebugger(){
    emulator.attach().addBreakPoint(module.findSymbolByName("verifyApkSign").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            System.out.println("替换函数 verifyApkSign");
            RegisterContext registerContext = emulator.getContext();
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_PC, registerContext.getLRPointer().peer);
            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, 0);
            return true;
        }
    });
}

```

非常清晰易懂。

五、Call 函数
---------

分析具体算法时，常需要对其进行主动调用，进行更灵活和细致的分析

// TODO

### 1.Frida

### 2.Unidbg

六、Patch 与内存检索
-------------

### 1.Patch

Patch 就是直接对二进制文件进行修改，Patch 本质上只有两种形式

*   patch 二进制文件
*   在内存里 patch

Patch 的应用场景很多，在一些场景比 Hook 更好用，这就是需要介绍它的原因。Patch 二进制文件的形式是大多数人所熟悉的，在 IDA 中使用 KeyPatch 打补丁的体验很友好。但这里我们关注的是内存 Patch。

[![](https://blog.seeflower.dev/images/6.png)](https://blog.seeflower.dev/images/6.png)

0x8CA 处调用了签名校验函数，第三四节中通过 Replace 返回值或函数的方式来处理它，但实际上，修改 0x8CA 处这条四字节指令也是好办法。

[![](https://blog.seeflower.dev/images/7.png)](https://blog.seeflower.dev/images/7.png)

需要注意的是，本文只讨论了 arm32，指令集只考虑最常见的 thumb2，arm 以及 arm64 可以自行测试。

#### ⅠFrida

①方法一

```
var str_name_so = "libhookinunidbg.so";    
var n_addr_func_offset = 0x8CA;         

var n_addr_so = Module.findBaseAddress(str_name_so);
var n_addr_assemble = n_addr_so.add(n_addr_func_offset);

Memory.protect(n_addr_assemble, 4, 'rwx'); 
n_addr_assemble.writeByteArray([0x00, 0x20, 0x00, 0xBF]);

```

但这并不是最佳实践，因为相较于 Unidbg，Frida 操作在真实 Android 系统上，存在两个问题

*   是否存在多线程操纵目标地址处的内存？是否有冲突
*   arm 的缓存刷新机制

所以 Frida 提供了更安全可靠的系列 API 来修改内存中的字节

②方法二

```
var str_name_so = "libhookinunidbg.so";    
var n_addr_func_offset = 0x8CA;         

var n_addr_so = Module.findBaseAddress(str_name_so);
var n_addr_assemble = n_addr_so.add(n_addr_func_offset);


Memory.patchCode(n_addr_assemble, 4, function () {
    
    var cw = new ThumbWriter(n_addr_assemble);
    
    
    cw.putInstruction(0x2000)
    
    cw.putInstruction(0xBF00);
    cw.flush(); 
    console.log(hexdump(n_addr_assemble))
});

```

#### Ⅱ Unidbg

Unidbg 在修改内存上，既可以传机器码，也可以传汇编指令

①方法一

```
public void Patch1(){
    
    int patchCode = 0xBF002000; 
    emulator.getMemory().pointer(module.base + 0x8CA).setInt(0,patchCode);
}

```

②方法二

```
public void Patch2(){
    byte[] patchCode = {0x00, 0x20, 0x00, (byte) 0xBF};
    emulator.getBackend().mem_write(module.base + 0x8CA, patchCode);
}

```

③方法三

```
public void Patch3(){
    try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {
        KeystoneEncoded encoded = keystone.assemble("movs r0,0;nop");
        byte[] patchCode = encoded.getMachineCode();
        emulator.getMemory().pointer(module.base + 0x8CA).write(0, patchCode, 0, patchCode.length);
    }
}

```

### 2. 内存检索

假设 SO 存在碎片化，比如要分析某个 SO 的多个版本，需要 Patch 签名校验或者某处汇编，地址在不同版本不固定，但函数特征固定，内存检索 + 动态 Patch 就是一个好办法，可以很好适应不同版本、碎片化。

搜索特征片段依据需求，可能是搜索函数开头十字节，也可能是搜索目标地址上下字节或者其他。

[![](https://blog.seeflower.dev/images/8.png)](https://blog.seeflower.dev/images/8.png)

#### ⅠFrida

```
function searchAndPatch() {
    var module = Process.findModuleByName("libhookinunidbg.so");
    var pattern = "80 b5 6f 46 84 b0 03 90 02 91"
    var matches = Memory.scanSync(module.base, module.size, pattern);
    console.log(matches.length)
    if (matches.length !== 0)
    {
        var n_addr_assemble = matches[0].address.add(10);
        
        Memory.patchCode(n_addr_assemble, 4, function () {
            
            var cw = new ThumbWriter(n_addr_assemble);
            
            
            cw.putInstruction(0x2000)
            
            cw.putInstruction(0xBF00);
            cw.flush(); 
            console.log(hexdump(n_addr_assemble))
        });
    }
}

setImmediate(searchAndPatch);

```

#### Ⅱ Unidbg

```
public void SearchAndPatch(){
    byte[] patterns = {(byte) 0x80, (byte) 0xb5,0x6f,0x46, (byte) 0x84, (byte) 0xb0,0x03, (byte) 0x90,0x02, (byte) 0x91};
    Collection<Pointer> pointers = searchMemory(module.base, module.base+module.size, patterns);
    if(pointers.size() > 0){
        try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb)) {
            KeystoneEncoded encoded = keystone.assemble("movs r0,0;nop");
            byte[] patchCode = encoded.getMachineCode();
            ((ArrayList<Pointer>) pointers).get(0).write(10, patchCode, 0, patchCode.length);
        }
    }

}

private Collection<Pointer> searchMemory(long start, long end, byte[] data) {
    List<Pointer> pointers = new ArrayList<>();
    for (long i = start, m = end - data.length; i < m; i++) {
        byte[] oneByte = emulator.getBackend().mem_read(i, 1);
        if (data[0] != oneByte[0]) {
            continue;
        }

        if (Arrays.equals(data, emulator.getBackend().mem_read(i, data.length))) {
            pointers.add(UnidbgPointer.pointer(emulator, i));
            i += (data.length - 1);
        }
    }
    return pointers;
}

```

_值得一提的是，本节的内容也可用 [LIEF](https://github.com/lief-project/LIEF) Patch 二进制文件实现。_

七、Hook 时机过晚问题
-------------

上文中，Hook 代码都位于 **SO 加载后， 执行 JNI_OnLoad 之前**，和如下 Frida 代码同等时机。

```
var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
if (android_dlopen_ext != null) {
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            this.hook = false;
            var soName = args[0].readCString();
            if (soName.indexOf("libhookinunidbg.so") !== -1) {
                this.hook = true;
            }
        },
        onLeave: function (retval) {
            if (this.hook) {
                this.hook = false;
                
            }
        }
    });
}

```

但如果**.init 和. init_array 段**存在代码逻辑（init→init_array→JNIOnLoad），Hook 时机就太晚了，这种情况下就需要将 Hook 时机点提前到 init 执行前。

在 Frida 中，为了实现这一点，需要在 linker 中做文章，通常做法是 Hook Linker 中的 call_function 或 call_constructor 函数。而在 Unidbg 中，有以下一些办法。

以我们的 demo hookInUnidbg 为例，其中 init 段里就有如下逻辑，比较两个字符串的大小。

```
extern "C" void _init(void) {
    char str1[15];
    char str2[15];
    int ret;


    strcpy(str1, "abcdef");
    strcpy(str2, "ABCDEF");

    ret = strcmp(str1, str2);

    if(ret < 0)
    {
        LOGI("str1 小于 str2");
    }
    else if(ret > 0)
    {
        LOGI("str1 大于 str2");
    }
    else
    {
        LOGI("str1 等于 str2");
    }

}


```

当前显示 **str1 大于 str2**，我们的 Hook 目标是让其显示 **str1 小于 str2**。

### 1. 提前加载 libc

提前加载 libc，然后 hook strcmp 函数，修改其返回值为 - 1 是一个办法。如下是完整代码，提供了 Console Debugger 以及 HookZz 两个版本。

```
package com.tutorial;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;
import unicorn.ArmConst;
import java.io.File;

public class hookInUnidbg {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private final Module moduleLibc;

    hookInUnidbg() {

        
        emulator = AndroidEmulatorBuilder.for32Bit().build();

        
        final Memory memory = emulator.getMemory();
        
        memory.setLibraryResolver(new AndroidResolver(23));
        
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/tutorial/hookinunidbg.apk"));

        
        DalvikModule dmLibc = vm.loadLibrary(new File("unidbg-android/src/main/resources/android/sdk23/lib/libc.so"), true);
        moduleLibc = dmLibc.getModule();

        
        hookStrcmpByUnicorn();
        
        

        
        DalvikModule dm = vm.loadLibrary("hookinunidbg", true);
        
        module = dm.getModule();

        
        dm.callJNI_OnLoad(emulator);
    }

    public void call(){
        DvmClass dvmClass = vm.resolveClass("com/example/hookinunidbg/MainActivity");
        String methodSign = "call()V";
        DvmObject<?> dvmObject = dvmClass.newObject(null);
        dvmObject.callJniMethodObject(emulator, methodSign);

    }


    public static void main(String[] args) {
        hookInUnidbg mydemo = new hookInUnidbg();
        mydemo.call();
    }


    public void hookStrcmpByUnicorn(){
        emulator.attach().addBreakPoint(moduleLibc.findSymbolByName("strcmp").getAddress(), new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                RegisterContext registerContext = emulator.getContext();
                String arg1 = registerContext.getPointerArg(0).getString(0);

                emulator.attach().addBreakPoint(registerContext.getLRPointer().peer, new BreakPointCallback() {
                    @Override
                    public boolean onHit(Emulator<?> emulator, long address) {
                        if(arg1.equals("abcdef")){
                            emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, -1);
                        }
                        return true;
                    }
                });
                return true;
            }
        });
    }

    public void hookStrcmpByHookZz(){
        IHookZz hookZz = HookZz.getInstance(emulator); 
        hookZz.enable_arm_arm64_b_branch(); 
        hookZz.wrap(moduleLibc.findSymbolByName("strcmp"), new WrapCallback<HookZzArm32RegisterContext>() {
            String arg1;
            @Override
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                arg1 = ctx.getPointerArg(0).getString(0);
            }
            @Override
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                if(arg1.equals("abcdef")){
                    ctx.setR0(-1);
                }
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
}


```

但如果想 hook 的目标函数不是 libc 里的函数，就没效果了。比如想在 0x978 下个断点。

[![](https://blog.seeflower.dev/images/9.png)](https://blog.seeflower.dev/images/9.png)

### 2. 固定地址下断点

这是最常用也最方便的方式，但只有 Unicorn 引擎下可以使用。

通过 `vm.loadLibrary` 加载的第一个用户 SO，其基地址是 0x40000000，因此可以在 IDA 中看函数偏移，通过绝对地址 Console Debugger Hook。

```
package com.tutorial;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;
import unicorn.ArmConst;
import java.io.File;

public class hookInUnidbg {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private Module moduleLibc;

    hookInUnidbg() {

        
        emulator = AndroidEmulatorBuilder.for32Bit().build();

        
        final Memory memory = emulator.getMemory();
        
        memory.setLibraryResolver(new AndroidResolver(23));
        
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/tutorial/hookinunidbg.apk"));

        emulator.attach().addBreakPoint(0x40000000 + 0x978);

        
        DalvikModule dm = vm.loadLibrary("hookinunidbg", true);
        
        module = dm.getModule();

        
        dm.callJNI_OnLoad(emulator);
    }

    public void call(){
        DvmClass dvmClass = vm.resolveClass("com/example/hookinunidbg/MainActivity");
        String methodSign = "call()V";
        DvmObject<?> dvmObject = dvmClass.newObject(null);
        dvmObject.callJniMethodObject(emulator, methodSign);

    }


    public static void main(String[] args) {
        hookInUnidbg mydemo = new hookInUnidbg();
        mydemo.call();
    }
    
}

```

[![](https://blog.seeflower.dev/images/10.png)](https://blog.seeflower.dev/images/10.png)

如果加载了多个用户 SO，可以先运行一遍代码，确认目标 SO 的基地址（Unidbg 中不存在地址随机化，目标函数每次地址都固定。）然后在 loadLibrary 前 Hook 该地址，即可保证 Hook 不遗漏。

### 3. 使用 Unidbg 提供的模块监听器

实现自己的模块监听器

```
package com.tutorial;

import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.ModuleListener;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.hook.hookzz.HookEntryInfo;
import com.github.unidbg.hook.hookzz.HookZz;
import com.github.unidbg.hook.hookzz.InstrumentCallback;

public class MyModuleListener implements ModuleListener {
    private HookZz hook;

    @Override
    public void onLoaded(Emulator<?> emulator, Module module) {
        
        if(module.name.equals("libc.so")){
             hook = HookZz.getInstance(emulator);
        }

        
        if(module.name.equals("libhookinunidbg.so")){
            hook.instrument(module.base + 0x978 + 1, new InstrumentCallback<RegisterContext>() {
                @Override
                public void dbiCall(Emulator<?> emulator, RegisterContext ctx, HookEntryInfo info) {
                    System.out.println(ctx.getIntArg(0));
                }
            });
        }
    }
}

```

通过`memory.addModuleListener`绑定。

```
package com.tutorial;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;
import java.io.File;

public class hookInUnidbg{
    private final AndroidEmulator emulator;
    private final VM vm;

    hookInUnidbg() {

        
        emulator = AndroidEmulatorBuilder.for32Bit().build();

        
        final Memory memory = emulator.getMemory();

        
        memory.addModuleListener(new MyModuleListener());

        
        memory.setLibraryResolver(new AndroidResolver(23));
        
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/tutorial/hookinunidbg.apk"));

        
        DalvikModule dm = vm.loadLibrary("hookinunidbg", true);
        
        Module module = dm.getModule();

        
        dm.callJNI_OnLoad(emulator);
    }

    public void call(){
        DvmClass dvmClass = vm.resolveClass("com/example/hookinunidbg/MainActivity");
        String methodSign = "call()V";
        DvmObject<?> dvmObject = dvmClass.newObject(null);
        dvmObject.callJniMethodObject(emulator, methodSign);

    }


    public static void main(String[] args) {
        hookInUnidbg mydemo = new hookInUnidbg();
        mydemo.call();
    }

}

```

_每种方法都有对应使用场景，按需使用。除此之外也可以修改 Unidbg 源码，在 callInitFunction 函数前添加自己的逻辑。_

八、条件断点
------

在算法分析时，条件断点可以减少干扰信息。以 strcmp 为例，整个进程的所有模块都可能调用 strcmp 函数。

### 1. 限定于某 SO

#### ⅠFrida

```
Interceptor.attach(
    Module.findExportByName("libc.so", "strcmp"), {
        onEnter: function(args) {
            var moduleName = Process.getModuleByAddress(this.returnAddress).name;
            console.log("strcmp arg1:"+args[0].readCString())
            
            console.log("call from :"+moduleName)
        },
        onLeave: function(ret) {
        }
    }
);

```

#### Ⅱ Unidbg

```
public void hookstrcmp(){
    long address = module.findSymbolByName("strcmp").getAddress();
    emulator.attach().addBreakPoint(address, new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext registerContext = emulator.getContext();
            String arg1 = registerContext.getPointerArg(0).getString(0);
            String moduleName = emulator.getMemory().findModuleByAddress(registerContext.getLRPointer().peer).name;
            if(moduleName.equals("libhookinunidbg.so")){
                System.out.println("strcmp arg1:"+arg1);
            }
            return true;
        }
    });
}

```

### 2. 限定于某函数

比如某个函数在 SO 中被大量使用，现在只想分析这个函数在函数 a 中的使用。

#### ⅠFrida

```
var show = false;
Interceptor.attach(
    Module.findExportByName("libc.so", "strcmp"), {
        onEnter: function(args) {
            if(show){
                console.log("strcmp arg1:"+args[0].readCString())
            }
        },
        onLeave: function(ret) {

        }
    }
);

Interceptor.attach(
    Module.findExportByName("libhookinunidbg.so", "targetfunction"),{
        onEnter: function(args) {
            show = this;
        },
        onLeave: function(ret) {
            show = false;
        }
    }
)

```

#### Ⅱ Unidbg

```
public void hookstrcmp(){
    emulator.attach().addBreakPoint(module.findSymbolByName("targetfunction").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext registerContext = emulator.getContext();

            show = true;
            emulator.attach().addBreakPoint(registerContext.getLRPointer().peer, new BreakPointCallback() {
                @Override
                public boolean onHit(Emulator<?> emulator, long address) {
                    show = false;
                    return true;
                }
            });
            return true;
        }
    });

    emulator.attach().addBreakPoint(module.findSymbolByName("strcmp").getAddress(), new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator<?> emulator, long address) {
            RegisterContext registerContext = emulator.getContext();
            String arg1 = registerContext.getPointerArg(0).getString(0);
            if(show){
                System.out.println("strcmp arg1:"+arg1);
            }
            return true;
        }
    });
}

```

### 3. 限定于某处

[![](https://blog.seeflower.dev/images/11.png)](https://blog.seeflower.dev/images/11.png)

比如上图，只关注 0xA00 处发生的 strcmp。一个办法是 hook strcmp 函数，只在 lr 寄存器 = module.base + 0xA00 + 4 + 1 时打印输出。

另一个办法是 Console Debugger , 也很方便。

```
emulator.attach().addBreakPoint(module, 0xA00);
emulator.attach().addBreakPoint(module, 0xA04);

```

一定要掌握这些知识，并做到灵活变通。在实战中，诸如 “A hook 生效后再打印 B 函数的输出 “是很常见的，否则每个函数都打印几百行看的人眼都迷糊。

九、系统调用拦截——以时间为例
---------------

这里说的系统调用拦截，并不是要对系统调用进行 Hook，比如 [frida - syscall - intercceptor](https://github.com/AeonLucid/frida-syscall-interceptor) 这样，系统调用全部是 Unidbg 自己实现的，日志一开就能看，显然也没有 Hook 的必要。Unidbg 的**系统调用拦截**是为了替换系统调用，修改 Unidbg 中系统调用的实现。

有两个问题需要解释

*   为什么要修改系统调用？
    
    Unidbg 中部分系统调用没实现或者没实现好，以及有时候想要固定其输出，比如获取时间的系统调用，这些需求需要我们修复或修改 Unidbg 中系统调用的实现。
    
*   为什么不直接修改 Unidbg 源码
    
    1 是灵活性较差，2 是我们的实现或修改并不是完美的，直接改 Unidbg 源码是对运行环境的污染，影响其他项目。
    

在分析算法时，输入不变的前提下，如果输出在不停变化，会干扰算法分析，这种情况的一大来源是时间戳参与了运算。在 Frida 中，为了控制这种干扰因素，常常会 Hook libc 的 gettimeodfay 这个时间获取函数。

### 1.Frida

_hook time_

```
var time = Module.findExportByName(null, "time");
if (time != null) {
    Interceptor.attach(time, {
        onEnter: function (args) {

        },
        onLeave: function (retval) {
            
            retval.replace(100);
        }
    })
}

```

_hook gettimeofday_

```
function hook_gettimeofday() {
    var addr_gettimeofday = Module.findExportByName(null, "gettimeofday");
    var gettimeofday = new NativeFunction(addr_gettimeofday, "int", ["pointer", "pointer"]);
    
    Interceptor.replace(addr_gettimeofday, new NativeCallback(function (ptr_tz, ptr_tzp) {

        var result = gettimeofday(ptr_tz, ptr_tzp);
        if (result == 0) {
            console.log("hook gettimeofday:", ptr_tz, ptr_tzp, result);
            var t = new Int32Array(ArrayBuffer.wrap(ptr_tz, 8));
            t[0] = 0xAAAA;
            t[1] = 0xBBBB;
            console.log(hexdump(ptr_tz));
        }
        return result;
    }, "int", ["pointer", "pointer"]));
}

```

但 Frida 做这件事并不容易做圆满，单是 libc.so，就有 time、gettimeodfay、clock_gettime、clock 这四个库函数可以获取时间戳，而且样本可以通过内联汇编使用系统调用，获取时间戳。

### 2.Unidbg

Unidbg 中可以更方便、更大范围的固定时间，不必像 Frida 那般。time 和 gettimeodfay 库函数基于 gettimeodfay 这个系统调用，clock_gettime 和 clock 基于 clock_gettime 系统调用。所以只要在 Unidbg 中固定 gettimeodfay 和 clock_gettime 这两个系统调用获取的时间戳，就可以一劳永逸。

首先实现时间相关的系统调用处理器，其中的 _System.currentTimeMillis()_ 和 _System.nanoTime()_ 改成定数。

```
package com.tutorial;

import com.github.unidbg.Emulator;
import com.github.unidbg.linux.ARM32SyscallHandler;
import com.github.unidbg.memory.SvcMemory;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.unix.struct.TimeVal32;
import com.github.unidbg.unix.struct.TimeZone;
import com.sun.jna.Pointer;
import unicorn.ArmConst;

import java.util.Calendar;

public class TimeSyscallHandler extends ARM32SyscallHandler {
    public TimeSyscallHandler(SvcMemory svcMemory) {
        super(svcMemory);
    }

    @Override
    protected boolean handleUnknownSyscall(Emulator emulator, int NR) {
        switch (NR) {
            case 78:
                
                mygettimeofday(emulator);
                return true;
            case 263:
                
                myclock_gettime(emulator);
                return true;

        }

        return super.handleUnknownSyscall(emulator, NR);
    }


    private void mygettimeofday(Emulator<?> emulator) {
        Pointer tv = UnidbgPointer.register(emulator, ArmConst.UC_ARM_REG_R0);
        Pointer tz = UnidbgPointer.register(emulator, ArmConst.UC_ARM_REG_R1);
        emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, mygettimeofday(tv, tz));
    };

    private int mygettimeofday(Pointer tv, Pointer tz) {
        long currentTimeMillis = System.currentTimeMillis();

        long tv_sec = currentTimeMillis / 1000;
        long tv_usec = (currentTimeMillis % 1000) * 1000;

        TimeVal32 timeVal = new TimeVal32(tv);
        timeVal.tv_sec = (int) tv_sec;
        timeVal.tv_usec = (int) tv_usec;
        timeVal.pack();

        if (tz != null) {
            Calendar calendar = Calendar.getInstance();
            int tz_minuteswest = -(calendar.get(Calendar.ZONE_OFFSET) + calendar.get(Calendar.DST_OFFSET)) / (60 * 1000);
            TimeZone timeZone = new TimeZone(tz);
            timeZone.tz_minuteswest = tz_minuteswest;
            timeZone.tz_dsttime = 0;
            timeZone.pack();
        }
        return 0;
    }

    private static final int CLOCK_REALTIME = 0;
    private static final int CLOCK_MONOTONIC = 1;
    private static final int CLOCK_THREAD_CPUTIME_ID = 3;
    private static final int CLOCK_MONOTONIC_RAW = 4;
    private static final int CLOCK_MONOTONIC_COARSE = 6;
    private static final int CLOCK_BOOTTIME = 7;
    private final long nanoTime = System.nanoTime();

    private int myclock_gettime(Emulator<?> emulator) {
        int clk_id = emulator.getBackend().reg_read(ArmConst.UC_ARM_REG_R0).intValue();
        Pointer tp = UnidbgPointer.register(emulator, ArmConst.UC_ARM_REG_R1);
        long offset = clk_id == CLOCK_REALTIME ? System.currentTimeMillis() * 1000000L : System.nanoTime() - nanoTime;
        long tv_sec = offset / 1000000000L;
        long tv_nsec = offset % 1000000000L;

        switch (clk_id) {
            case CLOCK_REALTIME:
            case CLOCK_MONOTONIC:
            case CLOCK_MONOTONIC_RAW:
            case CLOCK_MONOTONIC_COARSE:
            case CLOCK_BOOTTIME:
                tp.setInt(0, (int) tv_sec);
                tp.setInt(4, (int) tv_nsec);
                return 0;
            case CLOCK_THREAD_CPUTIME_ID:
                tp.setInt(0, 0);
                tp.setInt(4, 1);
                return 0;
        }
        throw new UnsupportedOperationException("clk_id=" + clk_id);
    }
}

```

在自己的模拟器上使用它，原来模拟器创建是这么一句

```
emulator = AndroidEmulatorBuilder.for32Bit().build();

```

修改如下

```
AndroidEmulatorBuilder builder = new AndroidEmulatorBuilder(false) {
    public AndroidEmulator build() {
        return new AndroidARMEmulator(processName, rootDir,
                backendFactories) {
            @Override
            protected UnixSyscallHandler<AndroidFileIO>
            createSyscallHandler(SvcMemory svcMemory) {
                return new TimeSyscallHandler(svcMemory);
            }
        };
    }
};

emulator = builder.build();

```

十、Hook 检测
---------

Anti Unidbg 的方法浩如烟海，但事实上几乎没有主动 Anti Unidbg 的样本，有两方面原因

*   Unidbg 自身的多个重大弱点没有解决，比如多线程和信号机制尚未实现。
*   Unidbg 普及率和推广度还不高。

所以本节专注于 Hook 检测。

### 1. 检测第三方 Hook 框架

基于其 Hook 实现原理，可以对应检测。

#### ⅠInline Hook

以我熟悉的 inline Hook 检测为例，inline Hook 需要修改 Hook 处的前几个字节，跳转到自己的地方实现逻辑，最后再跳转回来。那么就有两类思路实现检测，首先开辟一个检测线程，对关键函数做如下二选一循环操作

*   函数开头前几个字节是否被篡改
*   函数体是否完整未被修改，常使用 crc32 校验，为什么不用 md5 或其他哈希函数？因为 crc32 极快，性能影响小，碰撞率又在可接受的范围内

相关项目：[check_fish_inline_hook](https://github.com/liaogang/check_fish_inline_hook)

#### Ⅱ Got Hook

相关项目：[SliverBullet5563/CheckGotHook: 检测 got hook（使用 xhook 测试）](https://github.com/SliverBullet5563/CheckGotHook)

### 2. 检测 Unicorn Based Hook

Unicorn Hook 似乎不可检测，但 Unicorn 也是可检测的。在星球的 Anti-Unidbg 系列，就提到过一种检测方式。在 Android 系统中，只支持对四字节对齐的内存地址做读写操作，所以通过内联汇编尝试向 SP+1 的位置做读写，在真机上会导致 App 崩溃，而 Unidbg 模拟执行不会出任何问题。当然，我们并不希望 App 崩溃，所以需要在代码中实现自己的信号处理函数，当此处发生异常时，信号处理函数接收信号并做出某种处理，因为 Unidbg 中程序不会异常，所以也不会走到信号处理函数，这里面可以设计形成差异。

除此之外，Unicorn 下断点调试或者做指令追踪时，必然会导致函数运行时间超出常理，基于运行时间的反调试策略也可行。

十一、Unidbg Trace 四件套
-------------------

基于 Frida 存在许多 trace 方案，比如用于 trace JNI 函数的 [JNItrace](https://github.com/chame1eon/jnitrace)，用于 trace Java 调用的 [ZenTrace](https://github.com/hluwa/ZenTracer)、[r0tracer](https://github.com/r0ysue/r0tracer)，又或者是官方的多功能 trace 工具 [frida-trace](https://frida.re/docs/frida-trace/)，用于指令级 trace 的 [Frida Stalker](https://frida.re/docs/stalker/)，又或者是 trace SO 中所有函数的 [trace_natives](https://github.com/Pr0214/trace_natives) ，以及 Linux 上著名的 [strace](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/strace.html) 或者 基于 Frida 的 [frida-syscall-interceptor](https://github.com/AeonLucid/frida-syscall-interceptor)，用于 trace 系统调用。

在 Unidbg 上，上述的大部分 trace，只需要调整日志等级就能实现。我们这里所讲的 trace，聚焦于如何让使用者对代码执行流有更强的掌控。

### 1.Instruction tracing

令追踪包括两部分

*   记录每条指令的执行，打印地址、机器码、汇编等信息
*   打印每条指令相关的寄存器值

Unidbg 基于 Unicorn CodeHook 封装了指令追踪，方法和效果如下

```
TraceHook traceCode();
TraceHook traceCode(long begin, long end);
TraceHook traceCode(long begin, long end, TraceCodeListener listener);

```

[![](https://blog.seeflower.dev/images/12.png)](https://blog.seeflower.dev/images/12.png)

Unidbg 的指令追踪，在第一部分的工作做得很好，采用 模块名 + 相对偏移 + 机器码 + 绝对地址 + 汇编 的展示形式，但美中不足的是，它并没有做第二部分的工作，可以使用如下的脚本在 Unidbg 中实现完善的指令追踪，其原理也是实现了一个自己的 codehook。

[增加 trace 的部分 by dqzg12300 · Pull Request #214](https://github.com/zhkl0228/unidbg/pull/214/commits/64f9835ebc0d529b2b92cf2bb4846210dff20e3c)

### 2.Function Tracing

指令 Trace 是最细粒度的 Trace，优点是细，缺点是动辄数百上千万行，让人迷失其中。函数粒度的 trace 则不然，粗糙但容易理解全貌，在算法还原的一些场景中会起到帮助。在 IDA Debug 中，即可选择函数追踪来记录函数调用，包括了有符号函数以及 IDA 识别并命名为 Sub_addr 的子函数。

[![](https://blog.seeflower.dev/images/13.png)](https://blog.seeflower.dev/images/13.png)

在 Frida 上，可以使用 [trace_natives](https://github.com/Pr0214/trace_natives) 对 一个 SO 中的全部函数进行 Trace，并形成如下调用图。

[![](https://blog.seeflower.dev/images/14.png)](https://blog.seeflower.dev/images/14.png)

Unidbg 中可以做到这一点吗？不妨看一下 Frida trace_natives 脚本，其中有三个关注点。

*   如何获得一个 SO 的全部函数列表，就像 IDA 一样
*   如何 Hook 函数
*   如何获得调用层级关系，形成树结构

关于问题 1，trace_natives 怎么解决的？直接编写 IDA 脚本获取 IDA 的函数列表

```
def getFunctionList():
    functionlist = ""
    minLength = 10
    maxAddress = ida_ida.inf_get_max_ea()
    for func in idautils.Functions(0, maxAddress):
        if len(list(idautils.FuncItems(func))) > minLength:
            functionName = str(idaapi.ida_funcs.get_func_name(func))
            oneFunction = hex(func) + "!" + functionName + "\t\n"
            functionlist += oneFunction
    return functionlist

```

脚本获取了函数以及对应函数名列表，同时通过 minLength 过滤较短的函数，至少包含 10 条汇编指令的函数才会被计入。这么做有两个原因

*   过短的函数可能导致 Frida Hook 失败（inline hook 原理所致）
*   过短的函数可能是工具函数，调用次数多，但价值不大，让调用图变得臃肿不堪

完整的 IDA 插件 getFunctions 代码如下

```
import os
import time

import ida_ida
import ida_nalt
import idaapi
import idautils
from idaapi import plugin_t
from idaapi import PLUGIN_PROC
from idaapi import PLUGIN_OK


def getFunctionList():
    functionlist = ""
    minLength = 10
    maxAddress = ida_ida.inf_get_max_ea()
    for func in idautils.Functions(0, maxAddress):
        if len(list(idautils.FuncItems(func))) > minLength:
            functionName = str(idaapi.ida_funcs.get_func_name(func))
            oneFunction = hex(func) + "!" + functionName + "\t\n"
            functionlist += oneFunction
    return functionlist



def getSoPathAndName():
    fullpath = ida_nalt.get_input_file_path()
    filepath, filename = os.path.split(fullpath)
    return filepath, filename


class getFunctions(plugin_t):
    flags = PLUGIN_PROC
    comment = "getFunctions"
    help = ""
    wanted_name = "getFunctions"
    wanted_hotkey = ""

    def init(self):
        print("getFunctions(v0.1) plugin has been loaded.")
        return PLUGIN_OK

    def run(self, arg):

        so_path, so_name = getSoPathAndName()
        functionlist = getFunctionList()
        script_name = so_name.split(".")[0] + "_functionlist_" + str(int(time.time())) + ".txt"
        save_path = os.path.join(so_path, script_name)
        with open(save_path, "w", encoding="utf-8") as F:
            F.write(functionlist)
        F.close()
        print(f"location: {save_path}")

    def term(self):
        pass


def PLUGIN_ENTRY():
    return getFunctions()

```

关于问题 2：使用 Frida Native Hook

关于问题 3：Frida 的 frida-trace 自带调用层级关系，所以 trace_Natives 脚本依赖 Frida-trace，展示出了树结构的调用图。分析源码发现，frida-trace 使用了 Frida 在 Interceptor.attach 环境中的 depth 如下代码中的 this.depth），depth 表示了调用深度。那深度的值哪来的呢？其最终依赖于 Frida 的栈回溯。

```
Interceptor.attach(Module.getExportByName(null, 'read'), {
  onEnter(args) {
    console.log('Context information:');
    console.log('Context  : ' + JSON.stringify(this.context));
    console.log('Return   : ' + this.returnAddress);
    console.log('ThreadId : ' + this.threadId);
    console.log('Depth    : ' + this.depth);
    console.log('Errornr  : ' + this.err);

    
    this.fd = args[0].toInt32();
    this.buf = args[1];
    this.count = args[2].toInt32();
  },
  onLeave(result) {
    console.log('----------')
    
    const numBytes = result.toInt32();
    if (numBytes > 0) {
      console.log(hexdump(this.buf, { length: numBytes, ansi: true }));
    }
    console.log('Result   : ' + numBytes);
  }
})

```

这三个问题能在 Unidbg 中解决吗？如果能解决，那就有了 Unidbg 版的 Function Tracing。

首先问题一，只是一个获取函数列表的插件，与使用 Frida 还是 Unidbg 无关，构不成问题。我们还可以更进一步思考，trace_Natives 依赖 IDA 实现对 SO 函数的识别，但与此同时也增加了使用的复杂度，而且加壳的 SO 无法直接识别函数，必须得先 dump+fix SO，其实还挺折腾人，不如不依赖 IDA，换个办法识别函数。ARM 中，函数序言常常以 push 指令开始，这可以代表绝大多数函数。配合 Unidbg 的 BlockHook 或者 CodeHook，就可以解析并 Hook 这些函数，问题二也顺带解决了。少部分函数会遗漏，但也无关痛痒。BlockHook 还会提供当前基本块的大小，我们设置对较小的块不予理睬。

接下来就是问题三，栈回溯这块，Unidbg 也实现了 arm unwind 栈回溯，一些情况下有 Bug，但总体应该能用。但 Unidbg 没有提供打印深度的函数，在 Unwinder 类中添加一个它。

_src/main/java/com/github/unidbg/unwind/Unwinder.java_

```
public final int depth(){
    int count = 0;
    Frame frame = null;
    while((frame = unw_step(emulator, frame)) != null) {
        if(frame.isFinish()){
            return count;
        }
        count++;
    }
    return count;
}

```

接下来三步骤合一，组装代码

```
PrintStream traceStream = null;
try {
    
    String traceFile = "unidbg-android/src/test/resources/app/traceFunctions.txt";
    traceStream = new PrintStream(new FileOutputStream(traceFile), true);
} catch (FileNotFoundException e) {
    e.printStackTrace();
}

final PrintStream finalTraceStream = traceStream;
emulator.getBackend().hook_add_new(new BlockHook() {
    @Override
    public void hookBlock(Backend backend, long address, int size, Object user) {
        if(size>8){
            Capstone.CsInsn[] insns = emulator.disassemble(address, 4, 0);
            if(insns[0].mnemonic.equals("push")){
                int level = emulator.getUnwinder().depth();
                assert finalTraceStream != null;
                for(int i = 0 ; i < level ; i++){
                    finalTraceStream.print("    |    ");
                }
                finalTraceStream.println("  "+"sub_"+Integer.toHexString((int) (address-module.base))+"  ");
            }
        }

    }

    @Override
    public void onAttach(Unicorn.UnHook unHook) {

    }

    @Override
    public void detach() {

    }
}, module.base, module.base+module.size, 0);

```

可以发现代码非常的简洁优雅，效果也不错

[![](https://blog.seeflower.dev/images/15.png)](https://blog.seeflower.dev/images/15.png)

### 3.Unidbg-FindKey

[Unidbg_FindKey](https://github.com/Pr0214/Unidbg_FindKey)

// TODO 原理和扩充

### 4.Unidbg-Findcrypt

Findcrypt 是老牌经典工具，Unidbg 版的 Findcrypt 是要做啥？解决什么痛点？有三个主要原因

*   Findcrypt 处理不了加壳 SO
*   Findcrypt 中说存在某种加密，但 SO 中并不一定用，我们的目标函数更不一定用。
*   从 Findcrypt 提示的常数不一定能找到对应函数，静态交叉分析有局限

// TODO

十二、固定随机数
--------

// TODO

十三、杂项
-----

无需 Hook，Unidbg 中通过其他方式实现

// TODO