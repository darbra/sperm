> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-272870.htm)

> Android Hook 技术学习——常见的 Hook 技术方案总结

Android Hook 技术学习——常见的 hook 技术方案
================================

目录

*   Android Hook 技术学习——常见的 hook 技术方案
*            [一、前言](#一、前言)
*            [二、编译原理](#二、编译原理)
*                    [1. 编译过程](#1.编译过程)
*                            [（1）链接方式](#（1）链接方式)
*                            [（2）链接库](#（2）链接库)
*                    [2. 可执行文件（ELF）](#2.可执行文件（elf）)
*                            [（1）ELF 文件结构](#（1）elf文件结构)
*                            [（2）GOT 和 PLT](#（2）got和plt)
*            [三、NDK 基础知识](#三、ndk基础知识)
*                    1.Android so 文件的类型
*                    [2.so 文件加载](#2.so文件加载)
*            [四、各类 hook 技术原理分析](#四、各类hook技术原理分析)
*                    1.Xposed hook 技术
*                    2.Frida hook 技术
*                    3.inlinehook 技术
*                            [（1）基本原理](#（1）基本原理)
*                            [（2）inlineHook 组成](#（2）inlinehook组成)
*                            [（3）inlineHook 实现](#（3）inlinehook实现)
*                            （4）Android-Inline-Hook 和 SandHook 技术
*                    4.PLT/GOT hook 技术
*                    5.Unicorn hook 技术
*            [五、各类 hook 技术实操](#五、各类hook技术实操)
*                    1.Xposed hook 实操
*                            [（1）环境安装](#（1）环境安装)
*                            [（2）Xposed 插件编写](#（2）xposed插件编写)
*                    2.frida hook 实操
*                            [（1）环境安装](#（1）环境安装)
*                            [（2）frida 使用](#（2）frida使用)
*                    [3.inlinehook 实操](#3.inlinehook实操)
*                            [（1）Android-lnine-Hook](#（1）android-lnine-hook)
*                                    <1> 编写目标函数 so 文件
*                                    <2> 导入文件
*                                    <3> 修改配置文件
*                                    <4> 编写 hook 代码
*                            [（2）SandHook 实操](#（2）sandhook实操)
*                                    <1> 导入文件
*                                    <2> 配置环境
*                                    <3> 编写 hook 代码
*                    4.PLT/GOT hook 实操
*                                    <1> 获得 so 模块的加载地址
*                                    <2> 找到 got 表的位置
*                                    <3> 定位到节表的地址
*                                    <4> 定位到 got 表的位置和函数位置
*                    5.Unicorn hook 使用
*            [六、实验总结](#六、实验总结)
*            [参考文献](#参考文献)

[](#一、前言)一、前言
-------------

最近一段时间在研究 Android 加壳和脱壳技术，其中涉及到了一些 hook 技术，于是将自己学习的一些 hook 技术进行了一下梳理，以便后面回顾和大家学习。

 

本文第二节主要讲述编译原理，了解编译原理可以帮助进一步理解 hook 技术

 

本文第三节主要讲述 NDK 开发的一些基础知识

 

本文第四节主要讲述各类 hook 技术的实现原理

 

本文第五节主要讲述各 hook 技术的实现步骤和案例演示

[](#二、编译原理)二、编译原理
-----------------

### 1. 编译过程

![](https://bbs.pediy.com/upload/attach/202205/905443_JC5TBVTX9ZNWVFV.png)

 

我们可以借助 gcc 来实现上面的过程：

```
预处理阶段：预处理器（cpp）根据以字符#开头的命令修给原始的C程序，结果得到另一个C程序，通常以.i作为文件扩展名。主要是进行文本替换、宏展开、删除注释这类简单工作。
    命令行：gcc -E hello.c hello.i
编译阶段：将文本文件hello.i翻译成hello.s，包含相应的汇编语言程序
汇编阶段：将.S文件翻译成机器指令，然后把这些指令打包成一种可重定位目标程序的格式，并把结果保存在目标文件.o中（汇编——>机器）
    命令行：gcc -c hello.c hello.o
链接阶段：hello程序调用了printf函数，链接器（Id）就把printf.o文件并入hello.o文件中，得到hello可执行文件，然后加载到存储器中由系统执行。
    函数库包括静态库和动态库
    静态库：编译链接时，把库文件代码全部加入可执行文件中，运行时不需要库文件，后缀为.a。
    动态库：编译链接时，不加入，在程序执行时，由运行时链接文件加载库，这样节省开销，后缀为.so。（gcc编译时默认使用动态库）
再经过汇编器和连接器的作用后输出一个目标文件，这个目标文件为可执行文件

```

这里我们对编译过程做了一个初步的讲解，详细大家可以去看《程序员的自我修养——链接、装载与库》一书，下面我们主要介绍链接方式、链接库、可执行目标文件几个基本概念。

#### [](#（1）链接方式)（1）链接方式

**静态链接：**

```
对于静态库，程序在编译链接时，将库的代码链接到可执行文件中，程序运行时不再需要静态库。在使用过程中只需要将库和我们的程序编译后的文件链接在一起就可形成一个可执行文件。

```

**缺点：**

```
1、内存和磁盘空间浪费：静态链接方式对于计算机内存和磁盘的空间浪费十分严重。假如一个c语言的静态库大小为1MB，系统中有100个需要使用到该库文件，采用静态链接的话，就要浪费进100M的内存，若数量再大，那浪费的也就更多。
2.更新麻烦：比如一个程序20个模块，每个模块只有1MB，那么每次更新任何一个模块，用户都得重新下载20M的程序

```

**动态链接：**

```
由于静态链接具有浪费内存和模块更新困难等问题，提出了动态链接。基本实现思想是把程序按照模块拆分成各个相对独立部分，在程序运行时才将他们链接在一起形成一个完整的程序，而不是像静态链接那样把所有的程序模块都链接成一个单独的可执行文件。所以动态链接是将链接过程推迟到了运行时才进行。

```

例子：

```
同样，假如有程序1，程序2，和Lib.o三个文件，程序1和程序2在执行时都需要用到Lib.o文件，当运行程序1时，系统首先加载程序1，当发现需要Lib.o文件时，也同样加载到内存，再去加载程序2当发现也同样需要用到Lib.o文件时，则不需要重新加载Lib.o，只需要将程序2和Lib.o文件链接起来即可，内存中始终只存在一份Lib.o文件。

```

![](https://bbs.pediy.com/upload/attach/202205/905443_PBRES2N9MDUUJG2.png)

 

优点：

```
（1）毋庸置疑的就是节省内存；
（2）减少物理页面的换入换出；
（3）在升级某个模块时，理论上只需要将对应旧的目标文件覆盖掉即可。新版本的目标文件会被自动装载到内存中并且链接起来；
（4）程序在运行时可以动态的选择加载各种程序模块，实现程序的扩展。

```

#### [](#（2）链接库)（2）链接库

我们在链接的过程中，一般会链接一些库文件，主要分为静态链接库和动态链接库。静态链接库一般为`Windows下的.lib和Linux下的.a`, 动态链接库一般为`Windows下的.dll和Linux下的.so`，这里考虑到我们主要是对 so 文件 hook 讲解，下面我们主要介绍 linux 系统下的情况。

 

**静态库：**

```
命名规范为libXXX.a
库函数会被连接进可执行程序，可执行文件体积较大
可执行文件运行时，不需要从磁盘载入库函数，执行效率较高
库函数更新后，需要重新编译可执行程序

```

**动态库：**

```
命名规范为libXXX.so
库函数不被连接进可执行程序，可执行文件体积较小
可执行文件运行时，库函数动态载入
使用灵活，库函数更新后，不需要重新编译可执行程序

```

### 2. 可执行文件（ELF）

目前 PC 平台比较流行的可执行文件格式主要是 Windows 下的 PE 和 Linux 下的 ELF，它们都是 COFF 格式的变种。在 Windows 平台下就是我们比较熟悉的. exe 文件，而 Linux 平台下现在便是统称的 ELF 文件。这里我们主要介绍一下 Linux 下的 ELF 文件。

 

**ELF 文件的类型：**

```
可重定位目标文件：包含二进制代码和数据，其形式可以和其他目标文件进行合并，创建一个可执行目标文件。比如linux下的.o文件
可执行目标文件：包含二进制代码和数据，可直接被加载器加载执行。 比如/bin/sh文件
共享目标文件：可被动态的加载和链接。比如.so文件

```

**ELF 文件的结构：**

 

elf 文件在不同的平台上有不同的格式，在 Unix 和 x86-64 Linux 上称 ELF：

#### [](#（1）elf文件结构)（1）ELF 文件结构

目标文件既要参与程序链接，又要参与程序执行：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_7TFTERM5FTMWAVF.png)

```
(1)文件开始处：是一个ELF头部（ELF Header），用来描述整个文件的组织。节区部分包含链接视图的大量信息：指令、数据、符号表、重定位信息等。
(2)程序头部表(Program Header Table)：如果存在的话，会告诉系统如何创建进程映像。用来构造进程映像的目标文件必须具有程序头部表，可重定位文件不需要这个表。
(3)节区头部表(Section Header Table)：包含了描述文件节区的信息，每个节区在表中都有一项，每一项给出诸如节区名称、节区大小这类信息。用于链接的目标文件必须包含节区头部表，其他目标文件可以有，也可以没有这个表。

```

下面我们来从分别从连接视角和程序执行的视角来看 ELF 文件：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_5N6DT5YSG5SHXYY.png)

```
ELF Header:描述了描述了体系结构和操作系统等基本信息并指出Section Header Table和Program Header Table在文件中的什么位置
Program Header Table: 保存了所有Segment的描述信息；在汇编和链接过程中没有用到，所以是可有可无的
Section Header Table:保存了所有Section的描述信息；Section Header Table在加载过程中没有用到，所以是可有可无的

```

下面我们来看一张更加详细的 ELF 结构图

 

![](https://bbs.pediy.com/upload/attach/202205/905443_34AAE6M4KDSPFPX.png)

 

从中我们可以详细的知道 ELF 文件各个字段的含义，其他字段的含义如下图

 

![](https://bbs.pediy.com/upload/attach/202205/905443_TFE3WYCC7Q4QHHR.png)

#### [](#（2）got和plt)（2）GOT 和 PLT

上面我们简单的分析了 ELF 的文件结构，而这里我们介绍一下其中两个重要的节表`GOT(全局偏移表)`和`PLT(程序链接表)`

 

首先，我们需要理解为什么需要 GOT 表和 PLT 表

 

经过上面的分析，我们知道程序在经历了编译流程后，就来到了链接过程，链接过程就是将一个或者多个中间文件（.o 文件）通过链接器将它们链接成一个可执行文件，主要要完成以下事情：

```
1.各个中间文之间的同名section合并
2.对代码段，数据段以及各符号进行地址分配
3.链接时重定位修正

```

但是当我们程序运行起来，`glibc`动态库也装载了，函数地址也确定了，那我们程序如何去调用动态库中的函数呢，这个时候就需要理解一下重定位的概念：

 

**重定位：**

```
1.链接重定位：将一个或多个中间文件(.o文件)通过链接器将它们链接成一个可执行文件，一般分为两种情况：
    （1）如果是在其他中间文件中已经定义了的函数，链接阶段可以直接重定位到函数地址，比如我们从头文件访问另一个函数
    （2）如果是在动态库中定义了的函数，链接阶段无法直接重定位到函数地址，只能生成额外的小片段代码，也就是PLT表，然后重定位到该代码片段
2.运行重定位：运行后加载动态库，把动态库中的相应函数地址填入GOT表，由于PLT表是跳转到GOT表的，这就构成了运行时重定位
3.延迟重定位：只有动态库函数在被调用时，才会进行地址解析和重定位工作，这时候动态库函数的地址才会被写入到GOT表项中

```

这里我们就可以明白流程，程序在加载动态库中函数时，需要两部分：

```
需要存放外部函数的代码段表（PLT表）
存放函数地址的数据表（GOT表）

```

这里我用一个实例加深大家的理解，例如程序在链接时发现 scanf 定义在动态库时，链接器生成一小段代码 scanf_stub, 这就是我们的 PLT 表，然后 scanf_stub 地址取代原来的 scanf, 因此程序此时就转换为链接 scanf_stub，这个过程叫链接重定位，然后在运行时动态库 glibc 中的 scanf_libc 地址填入 GOT 表，然后程序通过 scanf_stub 访问到 scanf_libc，这个过程叫运行时重定位。

 

讲到这里，其实我们对 PLT 和 GOT 表的作用已经了解了，`PLT（程序链接表）`就是链接时需要存放外部函数的数据段，`GOT（全局偏移表）`是存放函数地址的代码

 

**PLT 和 GOT 的结构：**

```
PLT表中的第一项为公共表项，剩下的是每个动态库函数为一项,每项PLT都从对应的GOT表项中读取目标函数地址
 
GOT表中前3个为特殊项，分别用于保存 .dynamic段地址、本镜像的link_map数据结构地址和_dl_runtime_resolve函数地址
dynamic段：提供动态链接的信息，例如动态链接中各个表的位置
link_map：已加载库的链表，由动态库函数的地址构成的链表
_dl_runtime_resolve：在第一次运行时进行地址解析和重定位工作

```

根据操作系统规定不允许修改代码段，只能修改数据段，所以 PLT 表是不变的，GOT 表是可以改变的

<table><thead><tr><th>.plt</th><th>代码段</th><th>RE（可读，可执行）</th><th>.plt section 实际就是通常所说的过程链接表（Procedure Linkage Table, PLT）</th></tr></thead><tbody><tr><td>.plt.got</td><td>代码段</td><td>RE</td><td>.plt.got section 用于存放 __cxa_finalize 函数对应的 PLT 条目</td></tr><tr><td>.got</td><td>数据段</td><td>RW（可读，可写）</td><td>.got section 中可以用于存放全局变量的地址；.got section 中也可以用于存放不需要延迟绑定的函数的地址。</td></tr><tr><td>.got.plt</td><td>数据段</td><td>RW</td><td>.got.plt section 用于存放需要延迟绑定的函数的地址</td></tr></tbody></table>

 

因此我们可以看一下程序调用 PLT 表和 GOT 表的逻辑

 

![](https://bbs.pediy.com/upload/attach/202205/905443_E2EUR5PYRBX7ZB2.png)

 

最后我们来详细看一下程序调用函数的变化流程：

 

**程序第一次调用函数时：**

 

![](https://bbs.pediy.com/upload/attach/202205/905443_DJSS6R4KNDG87XM.png)

 

此时第一步由函数调用跳入到 PLT 表中，然后第二步 PLT 表跳到 GOT 表中，可以看到第三步由 GOT 表回跳到 PLT 表中，这时候进行压栈，把代表函数的 ID 压栈，接着第四步跳转到公共的 PLT 表项中，第 5 步进入到 GOT 表中，然后_dl_runtime_resolve 对动态函数进行地址解析和重定位，第七步把动态函数真实的地址写入到 GOT 表项中，然后执行函数并返回，此时 GOT 表中就存放了函数的真实地址

 

**之后函数被调用时：**

 

![](https://bbs.pediy.com/upload/attach/202205/905443_AVAS94K9B65W2CH.png)

 

第一步还是由函数调用跳入到 PLT 表，但是第二步跳入到 GOT 表中时，由于这个时候该表项已经是动态函数的真实地址了，所以可以直接执行然后返回

[](#三、ndk基础知识)三、NDK 基础知识
------------------------

这里我们主要介绍 Android 中的 so 文件加载的原理，为后面 hook 技术讲解做铺垫：

### 1.Android so 文件的类型

NDK 开发的 so 不再具备跨平台特性，需要编译提供不同平台支持

 

![](https://bbs.pediy.com/upload/attach/202205/905443_JD8BB3AV53PKUH3.png)

 

我们从官网可以得知 so 文件在不同架构下也不同，这里依次对应`arm32位和64位，x86_32位和64位`

 

我们可以使用指令查看我们手机的架构：

```
adb shell
cat /proc/cpuinfo

```

![](https://bbs.pediy.com/upload/attach/202205/905443_8B8V7Q8RVKM4QH6.png)

### 2.so 文件加载

Android 中我们通常使用系统提供的两种 API：System.loadLibrary 或者 System.load 来加载 so 文件：

```
//加载的是libnative-lib.so，注意的是这边只需要传入"native-lib"
System.loadLibrary("native-lib");
//传入的是so文件完整的绝对路径
System.load("/data/data/应用包名/lib/libnative-lib.so")

```

System.loadLibrary() 和 System.load() 的区别：

```
（1）loadLibray传入的是编译脚本指定生成的so文件名称，一般不需要包含开头的lib和结尾的.so，而load传入的是so文件所在的绝对路径
（2）loadLibrary传入的不能是路径，查找so时会优先从应用本地路径下(/data/data/${package-name}/lib/arm/)进行查找，不存在的话才会从系统lib路径下(/system/lib、/vendor/lib等)进行查找；而load则没有路径查找的过程
（3）load传入的不能是sdcard路径，会导致加载失败，一般只支持应用本地存储路径/data/data/${package-name}/，或者是系统lib路径system/lib等这2类路径
（4）loadLibrary加载的都是一开始就已经打包进apk或系统的so文件了，而load可以是一开始就打包进来的so文件，也可以是后续从网络下载，外部导入的so文件
（5）重复调用loadLibrar,load并不会重复加载so，会优先从已加载的缓存中读取，所以只会加载一次
（6）加载成功后会去搜索so是否有"JNI_OnLoad"，有的话则进行调用，所以"JNI_OnLoad"只会在加载成功后被主动回调一次，一般可以用来做一些初始化的操作，比如动态注册jni相关方法等

```

**源码分析：**

 

**Android 6.0：**

 

[System.java] java.lang.System:

```
  public static void load(String pathName) {
        Runtime.getRuntime().load(pathName, VMStack.getCallingClassLoader());
   }
 
   public static void loadLibrary(String libName) {
       Runtime.getRuntime().loadLibrary(libName, VMStack.getCallingClassLoader());
}

```

[Runtime.java] java.lang.Runtime:

```
void load(String absolutePath, ClassLoader loader) {
        if (absolutePath == null) {
            throw new NullPointerException("absolutePath == null");
        }
        String error = doLoad(absolutePath, loader);
        if (error != null) {
            throw new UnsatisfiedLinkError(error);
        }
    }
public void loadLibrary(String nickname) {
        loadLibrary(nickname, VMStack.getCallingClassLoader());
    }
 
void loadLibrary(String libraryName, ClassLoader loader) {
        if (loader != null) {
            String filename = loader.findLibrary(libraryName);
            if (filename == null) {...

```

我们对比了 Android6.0 下的 System.load 和 System.loadLibrary:

```
我们可以发现System.loadLibrary()中会修改类加载器，这个在我们后面hook过程可能会报错，而Runtime.loadLibray()中有重写的方法，则可以正确实现

```

**Android 7.0：**

 

[System.java] java.lang.System:

```
public static void load(String filename) {
      Runtime.getRuntime().load0(VMStack.getStackClass1(), filename);
  }
 
public static void loadLibrary(String libname) {
      Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), libname);
  }

```

[Runtime.java] java.lang.Runtime:

```
synchronized void load0(Class fromClass, String filename) {
       if (!(new File(filename).isAbsolute())) {
           throw new UnsatisfiedLinkError(
               "Expecting an absolute path of the library: " + filename);        }
       if (filename == null) {
           throw new NullPointerException("filename == null");
       }
       String error = doLoad(filename, fromClass.getClassLoader());
       if (error != null) {
           throw new UnsatisfiedLinkError(error);
       }
   }
 
    public void loadLibrary(String libname, ClassLoader classLoader) {
       java.lang.System.logE("java.lang.Runtime#loadLibrary(String, ClassLoader)" +
                             " is private and will be removed in a future Android release");
      loadLibrary0(classLoader, libname);
   }

```

我们可以发现不同版本的区别：

```
Android 6.0采用的是loadLibrary,6.0之后都采用的是loadLibrary0; 同理 load函数也一样,6.0之后采用的是load0

```

同时我们分析了 loadLibrary0：

```
1. classLoader存在时，通过classLoader.findLibrary(libraryName)来获取存放指定so文件的路径；
2. classLoader不存在时，则通过getLibPaths()接口来获取
3. 最终调用nativeLoad加载指定路径的so文件

```

[](#四、各类hook技术原理分析)四、各类 hook 技术原理分析
-----------------------------------

hook 技术就是指截获进程对某个 API 函数的调用，使得 API 的执行流程转向我们实现的代码片段，从而实现我们要的功能，在 Android 中使用 hook 的方法有很多，常用的 Xposed 和 frida hook 技术、inlinehook 技术、基于 inlinehook 的开源框架 Sandhook、PLT/Got hook 技术、以及当下模拟 cpu 的 Unicorn 的 hook 技术，下面我们将逐一介绍其原理。

### 1.Xposed hook 技术

Xposed 的基本原理，我在[源码编译（3）——Xposed 框架定制](https://bbs.pediy.com/thread-269627.htm)中已经给大家做了详细的讲解，其主要就是 Android 应用进程都是由 zygote 进程孵化而来，zygote 对应的可执行程序就是 app_process,posed 框架通过替换系统的 app_process 可执行文件以及虚拟机动态链接库，让 zygote 在启动应用程序进程时注入框架代码，进而实现对应用程序进程的劫持。

 

具体怎么实现 hook 技术，Xposed 就是通过修改了 Art 虚拟机，将需要 hook 的函数注册为 Native 函数，当执行这一函数时，虚拟机会优先执行 Native 函数，然后执行 java 函数，这样就成功完成了函数的 hook。

 

![](https://bbs.pediy.com/upload/attach/202205/905443_A6M5RGRCQ2A3HGM.png)

 

具体实现流程:

 

在 Android 系统启动的时候， zygote 进程加载 XposedBridge 将所有需要替换的 Method 通过 JNI 方法 hookMethodNative 指向 Native 方法 xposedCallHandler ， xposedCallHandler 在转入 handleHookedMethod 这个 Java 方法执行用户规定的 Hook Func

 

![](https://bbs.pediy.com/upload/attach/202205/905443_TTUBVG5KA7YWR9W.png)

 

dvmCallMethodV 会根据 accessFlags 决定调用 native 还是 java 函数，因此修改 accessFlags 后，Dalvik 会认为这个函数是一个 native 函数，便走向了 native 分支也就是说 Xposed 在对 java 方法进行 hook 时，先将虚拟机里面这个方法的 Method 的 accessFlag 改为 native 对应的值，然后将该方法的 nativeFunc 指向自己实现的一个 native 方法，这样方法在调用时，就会调用到这个 native 方法，接管了控制权

 

其他的就详细参考上篇文章了

### 2.Frida hook 技术

frida 也是一种动态插桩工具，原理和 Xposed hook 一样，也是把 java method 转为 native method，但是 Art 下的实现与 Dalivk 有所不同，这里就需要了解 ART 的运行机制，这里主要参考博客：[Frida 源码分析](https://mabin004.github.io/2018/07/31/Mac%E4%B8%8A%E7%BC%96%E8%AF%91Frida/)

 

ART 是一种代替 Dalivk 的新的运行时, 它具有更高的执行效率。ART 虚拟机执行 Java 方法主要有两种模式：quick code 模式和 Interpreter 模式

```
quick code 模式：执行 arm 汇编指令
Interpreter 模式：由解释器解释执行 Dalvik 字节码

```

即使是在 quick code 模式中，也有类方法可能需要以 Interpreter 模式执行。反之亦然。解释执行的类方法通过函数 artInterpreterToCompiledCodeBridge 的返回值调用本地机器指令执行的类方法；本地机器指令执行的类方法通过函数 GetQuickToInterpreterBridge 的返回值调用解释执行的类方法

 

这里引用博客中的一张图

 

![](https://bbs.pediy.com/upload/attach/202205/905443_VJRV8CFKFQS475P.png)

 

如图，对于一个 native 方法，ART 虚拟机会先尝试使用 quickcode 的模式去执行，并检查 ARTMethod 结构中的 entry_point_from_quick_compiled_code_ 成员，这里分 3 种情况：

```
1.如果函数已经存在quick code, 则指向这个函数对应的 quick code的起始地址，而当quick code不存在时，它的值则会代表其他的意义；
2.当一个 java 函数不存在 quick code时，它的值是函数 artQuickToInterpreterBridge 的地址，用以从 quick 模式切换到 Interpreter 模式来解释执行 java 函数代码；
3.当一个 java native（JNI）函数不存在 quick code时，它的值是函数 art_quick_generic_jni_trampoline 的地址，用以执行没有quick code的 jni 函数

```

因此，frida 将一个 java method 修改 jni mthod 显然是不存在 quick code，这时需要将 entry_point_from_quick_compiled_code_ 值修改为 art_quick_generic_jni_trampoline 的地址

 

总结，frida 把 java method 改为 jni method，需要修改 ARTMethod 结构体中的这几个值：

```
accessflags = native
entry_point_fromjni = 自定义代码的入口
entry_point_from_quick_compiledcode = art_quick_generic_jni_trampoline函数的地址
entry_point_frominterpreter = artInterpreterToCompiledCodeBridge函数地址

```

### 3.inlinehook 技术

#### [](#（1）基本原理)（1）基本原理

首先，我们先介绍一下什么是 inline Hook:

```
inline Hook是一种拦截目标函数调用的方法，主要用于杀毒软件、沙箱和恶意软件。一般的想法是将一个函数重定向到我们自己的函数，以便我们可以在函数执行它之前和/或之后执行处理；这可能包括：检查参数、填充、记录、欺骗返回的数据和过滤调用。
hook是通过直接修改目标函数内的代码来放置，通常是用跳转覆盖的前几个字节，允许在函数进行任何处理之前重定向执行。

```

#### [](#（2）inlinehook组成)（2）inlineHook 组成

```
hook:一个5字节的相对跳转，在被写入目标函数以钩住它，跳转将从被钩住的函数跳转到我们的代码
proxy:这是我们指定的函数（或代码），放置在目标函数上的钩子将跳转到该函数（或代码）
Trampoline:用于绕过钩子，以便我们可以正常调用钩子函数

```

#### [](#（3）inlinehook实现)（3）inlineHook 实现

![](https://bbs.pediy.com/upload/attach/202205/905443_CQY5DDQWTKHP3WS.png)

 

从示意图上，我们可以这样理解：

```
我们将目标函数MessgeBoxA()中的地址拿出来，然后我们用重写的hook函数替换，然后我们执行完成之后，再回调到函数的执行地址出，保证程序的正常运行

```

![](https://bbs.pediy.com/upload/attach/202205/905443_JXZH5VUZP7ETYGZ.png)

 

我们也可以通过上述示意图去理解 inlinehook 的基本原理

#### （4）Android-Inline-Hook 和 SandHook 技术

Android-lnline-Hook 和 SandHook 都是基于 inlinehook 的两种开源框架，在 Android 中对 native 层 hook，使用的比较常见，前者主要针对 32 位进行 hook，后者即可以用于 32 位也可以用于 64 位，但是官方表示 32 位并未进行测试，所以应用在 64 位上仍然更多

### 4.PLT/GOT hook 技术

前面我们已经很详细的讲述了全局偏移表（GOT）和动态链接表（PLT）,Inline Hook 能 Hook 几乎所有函数，但是兼容性较差，不能达到上线标准, 相比于 inlineHook，GOT Hook 兼容性比较好，可以达到上线标准，但是只能 Hook 基于 GOT 表的一些函数

 

GOT/PLT Hook 主要是通过解析 SO 文件，将待 hook 函数在 got 表的地址替换为自己函数的入口地址，这样目标进程每次调用待 hook 函数时，实际上是执行了我们自己的函数

 

这里我们还要理解 GOT 表中含包含了导入表和导出表

```
导出表指将当前动态库的一些函数符号保留，供外部调用
导入表中的函数实际是在该动态库中调用外部的导出函数

```

例如导入表存放的是一些其他 so 的函数，例如 libc 的 open，而导出表存放的是一些共其他 so 调用的函数，比如自己 so 中编写的函数，而无论导入表还是导出表基本都是针对导出函数，针对非导出函用 inlinehook 更常用一些

### 5.Unicorn hook 技术

Unicore 是一款非常优秀的跨平台模拟执行框架，该框架可以跨平台执行 Arm, Arm64 (Armv8), M68K, Mips, Sparc, & X86 (include X86_64) 等指令集的原生程序，通过模拟 CPU，可以实现很多强大的功能，也可以实现函数级别的 Hook

 

参考资料：无名大佬文章 [Unicorn 在 Android 的应用](https://bbs.pediy.com/thread-253868.htm#msg_header_h1_7)

 

nicorn 内部并没有函数的概念，它只是一个单纯的 CPU， 没有 HOOK_FUNCTION 的 callback，AndroidNativeEmu 中的函数级 Hook 并不是真正意义上的 Hook，它不仅能 Hook 存在的函数，还能 Hook 不存在的函数。AndroidNativeEmu 使用这种技术实现了 JNI 函数 Hook、库函数 Hook。 Jni 函数是不存的，Hook 它只是为了能够用 Python 实现 Jni Functions。有一些库函数是存在的，Hook 只是为了重新实现它

[](#五、各类hook技术实操)五、各类 hook 技术实操
-------------------------------

### 1.Xposed hook 实操

#### [](#（1）环境安装)（1）环境安装

Xposed 环境安装详细可以参考我写的 Xposed 系列文章，这里只是简单的总结一下：

```
(1) 4.4以下Android版本安装比较简单，只需要两步即可
    1.对需要安装Xposed的手机进行root
    2.下载并安装xposedInstaller,之后授权其root权限，进入app点击安装即可
    但是由于官网不在维护，导致无法直接通过xposedinstaller下载补丁包
（2）Android 5.0-8.0 由于5.0后出现ART，所以安装步骤分成两个部分：xposed.zip 和
    XposedInstaller.apk,zip文件是框架主体，需要进入Recovery后刷入，apk文件用于Xposed管理
    1.完成对手机的root，并刷入reconvery(比如twrp),使用Superroot
    2.下载你对应的zip补丁包，并进入recovery刷入
    3.重启手机，安装xposedInstaller并授予root权限即可
    官网地址：https://dl-xda.xposed.info/framework/
（3）由于Android 8.0后，Xposed官方作者没有再对其更新，我们一般就使用国内大佬riyu的Edxposed框架
    Magisk + riyu + Edxposed

```

这里我们用的是 nexus5 进行操作，简单演示一下 android6.0 的 Xposed 安装

 

资源准备：

```
asop镜像：https://developers.google.com/android/ota#hammerhead
twrp:     https://twrp.me/
xposed:   https://dl-xda.xposed.info/framework/
xposed installer https://repo.xposed.info/module/de.robv.android.xposed.installer

```

首先我们先下载 n5 镜像，然后刷机，这里我们已经安装就不再安装了

 

然后我们刷入 twrp-3.4.0-0-hammerhead.img

```
fastboot flash recovery twrp-3.4.0-0-hammerhead.img

```

![](https://bbs.pediy.com/upload/attach/202205/905443_PTP5YTG7G8Q65Q4.png)

 

然后我们就可以进入 recovery 模式了

 

然后我们将 Supersu 拷贝进去，然后将 Xposed-v89-sdk.zip 拷贝进去

 

![](https://bbs.pediy.com/upload/attach/202205/905443_USKZ6Q73UJ67BPY.png)

 

![](https://bbs.pediy.com/upload/attach/202205/905443_BP8GCDGXX6RTWRZ.png)

 

然后我们进入 recovery 模式，将两个文件依次刷入即可

 

接下来我们安装 XposedInstall.apk, 来管理 Xposed

 

![](https://bbs.pediy.com/upload/attach/202205/905443_5Y7BXVDNRSNAVR6.png)

 

如果我们开机后发现 xposed 框架没有激活，尝试再重启一下，我们可以看见

 

![](https://bbs.pediy.com/upload/attach/202205/905443_XNTNEVPKZK7HMWR.png)

 

这样我们的 Xposed 框架就成功安装了

#### [](#（2）xposed插件编写)（2）Xposed 插件编写

Xposed 插件编写的流程网上已经有很多了，这里我就简单的讲解一下

```
基本流程：
    (1)拷贝XposedBridgeApi.jar到新建工程的libs目录
    (2)修改app目录下的build.gradle文件，在AndroidManifest.xml中增加Xposed相关内容
    (3)新建hook类，编写hook代码
    (4)新建assets文件夹，然后在assets目录下新建文件xposed_init,在里面写上hook类的完整路径

```

首先，我们查找 XposedBridgeApi.jar 到新建工程的 libs 目录：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_XSP2C2T7ZFRNP8G.png)

 

然后，修改 AndroidManifest.xml 文件，在 Application 标签下增加内容如下：

修改 app 目录下的 build.gradle 文件：

```
进入app目录下的build.gradle文件，   
    compile fileTree(includes:['*.jar'],dir:'libs')
    替换成
    provided fileTree(includes:['*.jar'],dir:'libs')
现在provided变为 compileOnly
如果使用compile,可以正常编译生成插件apk,但是当安装到手机上后，xposed会报错，无法正常工作

```

编写 hook 类：

 

我们新建一个 hook 类 xposed01，并实现接口 IXposedHookLoadPackage, 并实现里面关键方法 handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam)，该方法会在每个软件被启动的时候回调，所以一般需要通过目标包名过滤

```
public class Xposed01 implements IXposedHookLoadPackage {
 
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(loadPackageParam.packageName.equals("com.example.xposedlesson2")){  //判断目标包名
            XposedBridge.log("XLZH"+loadPackageParam.packageName);  //打出包名的信息
            Log.i("Xposed01",loadPackageParam.packageName);
        }
    }
}

```

新建 assets 文件夹，然后在 assets 目录下新建文件 xposed_init, 在里面写上 hook 类的完整路径

 

![](https://bbs.pediy.com/upload/attach/202205/905443_M8K7JKCWGJZM2C7.png)

 

这里面可以写多个 hook 类，每个类写一个，我们就完成了基本的 Xposed 框架的编写

 

最后勾选模块，并重启即可生效

 

![](https://bbs.pediy.com/upload/attach/202205/905443_K8KRQA95GC2T55T.png)

 

![](https://bbs.pediy.com/upload/attach/202205/905443_WYH6S62TQ4H4W69.png)

 

我们可以发现我们的 xposed 插件生效了，将我们系统中进程名打印出来了, 说明 hook 成功了

### 2.frida hook 实操

#### [](#（1）环境安装)（1）环境安装

frida 安装，使用 frida 过程中我们可以安装 objection 来进一步助力我们的 hook 工作，这个参考肉丝大佬的知识星球

 

工具安装（也可以选用其他版本）：

```
pip install frida==12.8.0
pip install frida-tools==5.3.0
pip install objection==1.8.4

```

安装成功后，查看 frida 和 objection，确定版本正确

```
frida --version
objection --help

```

然后将 frida_server 推送到`/data/local/tmp`下，并启动：（下载地址：https://github.com/frida/frida/releases）

 

![](https://bbs.pediy.com/upload/attach/202205/905443_DK5Z49RCDWTK5V7.png)

#### [](#（2）frida使用)（2）frida 使用

然后我们就可以使用自动化工具 objection 和编写 js 脚本进行 hook 了

 

objection 使用（详细参考肉丝大佬 github 的教程）：

```
常见的hook命令：
objection -g com.android.settings explore  //注入设置应用
android hooking list activities  //查看Activity，service相同
android intent launch_activity com.android.settings.DisplaySettings  //实现Activity跳转
android heap search instances com.android.settings.DisplaySettings   //搜索类的实例
android heap execute 0x2526 getPreferenceScreenResId    //主动调用实例
android hooking list classes  //列出内存中所有类
android hooking search methods display  //列出内存中所有的方法
android hooking watch class android.bluetooth.BluetoothDevice  //hook相关类的所有方法
android hooking watch class_method android.bluetooth.BluetoothDevice.getName --dump-args --dump-return --dump-backtrace  //打印具体方法的参数、返回值、堆栈信息

```

编写脚本：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_R2F8JJFJF5PN2XU.png)

 

启动方式：

```
attach方式 frida -U com.example.test -l hook.js
spwan启动 frida -U -f com.example.test -l demo1.js --no-pause

```

这样我们就可以成功注入了，更加复杂的脚本编写可以参考 [frida 博客](https://github.com/hookmaster/frida-all-in-one)

 

详细案例实操，这里可以参考之前我的文章:[Android 恶意样本分析——frida 破解三层锁机样本](https://bbs.pediy.com/thread-269128.htm)

### 3.inlinehook 实操

这里我们分别实现基于 inlinehook 的两个开源框架的具体使用方法

#### [](#（1）android-lnine-hook)（1）Android-lnine-Hook

开源地址：https://github.com/ele7enxxh/Android-Inline-Hook

 

该框架只能针对 32 位的 so 文件进行 hook

 

我们对 so 文件进行 hook 时，可以按照如下步骤进行：

```
（1）查看so文件中的目标函数
（2）编写Xposed hook代码，hook目标程序
（3）编写so层hook代码，hook so中的函数地址
（4）链接Java层和so层

```

##### <1> 编写目标函数 so 文件

![](https://bbs.pediy.com/upload/attach/202205/905443_5M34E5RGVMHTXVQ.png)

 

我们编写案例，很明显这里会打印失败，然后我们使用 inline-hook 框架进行 hook

##### <2> 导入文件

我们将该框架中如下文件导入我们的项目中

 

我们需要使用 [inlineHook](https://github.com/ele7enxxh/Android-Inline-Hook) 文件夹，并把这些文件直接拷贝到我们的工作目录：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_J35447VZ396W69Y.png)

 

![](https://bbs.pediy.com/upload/attach/202205/905443_PM8D54VPMM4V2DN.png)

##### <3> 修改配置文件

![](https://bbs.pediy.com/upload/attach/202205/905443_RWZAX8NBEZYBSWU.png)

##### <4> 编写 hook 代码

我们导入 inlinehook 头文件就可以开始编写 hook 代码了

 

![](https://bbs.pediy.com/upload/attach/202205/905443_TWS32X2EXSB6JKX.png)

 

编译，报错：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_D67M5NGM8V5PBSZ.png)

 

这是因为框架仅仅针对 32 位，所以我们需要在配置文件里面指定一下

 

![](https://bbs.pediy.com/upload/attach/202205/905443_NXT48W65NN9ZDDC.png)

 

然后编译，发现能正常通过

 

首先声明 hook 的就函数，然后编写对应的新函数，这里我们 hook 的是`strstr函数`

 

![](https://bbs.pediy.com/upload/attach/202205/905443_9JHGK9T47FU4C88.png)

 

然后调用 inlinehook 进行 hook

 

![](https://bbs.pediy.com/upload/attach/202205/905443_X56HZNKEDMKMDRW.png)

 

最后我们发现就可以成功的 hook

 

![](https://bbs.pediy.com/upload/attach/202205/905443_6RH3X3BJK6HVQR6.png)

 

代码分析：

```
源码解析：
    （1）dlopen：该函数将打开一个新库，并把它装入内存
        void *dlopen(const char *filename, int flag);
        参数1：文件名就是一个动态库so文件，标志位：RTLD_NOW 的话，则立刻计算；设置的是 RTLD_LAZY，则在需要的时候才计算
        libc.so是一个共享库
        ======================
        参数中的 libname 一般是库的全路径，这样 dlopen 会直接装载该文件；如果只是指定了库名称，在 dlopen 会按照下面的机制去搜寻：
        根据环境变量 LD_LIBRARY_PATH 查找
        根据 /etc/ld.so.cache 查找
        查找依次在 /lib 和 /usr/lib 目录查找。
        flag 参数表示处理未定义函数的方式，可以使用 RTLD_LAZY 或 RTLD_NOW 。 RTLD_LAZY 表示暂时不去处理未定义函数，先把库装载到内存，等用到没定义的函数再说； RTLD_NOW 表示马上检查是否存在未定义的函数，若存在，则 dlopen 以失败告终。
        参考链接：https://blog.nowcoder.net/n/5b2c04bbcccf431e9f1ab34aa02717fe
        =======================
    （2）dlsym:在 dlopen 之后，库被装载到内存。 dlsym 可以获得指定函数( symbol )在内存中的位置(指针)。
         void *dlsym(void *handle,const char *symbol);
         参数1：文件句柄  参数2：函数名

```

inlinehook 框架使用正确姿势：

```
我们对一个目标so文件hook步骤如下：
    （1）我们获取so的handler，使用dlopen函数
        void* libhandler = dlopen("libc.so",RTLD_NOW);
    （2）我们获取hook目标函数的地址,使用dlsym函数
        void* strstr_addr = dlsym(libhandler,函数名);
    （3）声明原来的函数
        void* (*oldmethod)(char*,char*); //这个格式需要参考hook的函数
        声明现在的函数
        void* newmethod(char* a,char* b){
            return (void *)oldmethod(a,b);
        }
    （3）使用registerInlinehook进行重定向，将hook函数地址重定向我们编写的新函数上
        (registerInlineHook((uint32_t) strstr_addr, (uint32_t) new_strstr, (uint32_t **) &old_strstr) != ELE7EN_OK
        //参数一：hook函数的地址 参数二：替换函数的地址  参数3：用来保存原来函数的地址
    （5）我们判断我们的hook操作是否成功,并且再次调用实现hook
        (inlineHook((uint32_t) strstr_addr) == ELE7EN_OK)

```

#### [](#（2）sandhook实操)（2）SandHook 实操

因为上面使用 inline 框架只支持 32 位，所以这里我们用 SandHook 实现对 64 位 native 函数的 hook，[sandHook](https://github.com/asLody/SandHook) 既支持 32 位、又支持 64 位

 

开源地址：https://github.com/asLody/SandHook

 

同样是上面的案例，这里我们使用 SandHook 进行实操

##### <1> 导入文件

我们此路径下`SandHook/nativehook/src/main/cpp/`文件全部导入

 

![](https://bbs.pediy.com/upload/attach/202205/905443_UENG24U66ADC7WG.png)

##### <2> 配置环境

首先我们在 CMakeList 中加入 c 文件

 

![](https://bbs.pediy.com/upload/attach/202205/905443_K28NFSCXPDMCXZA.png)

 

然后在 java 代码中修改导入的 so 库

 

![](https://bbs.pediy.com/upload/attach/202205/905443_BKTVYYEPN7R47Z6.png)

 

直接编译，报错：

 

![](https://bbs.pediy.com/upload/attach/202205/905443_3UGDSRDEFN48UBY.png)

 

然后我们同理将配置信息加入：

```
cmake {
    arguments '-DBUILD_TESTING=OFF'
    cppFlags "-frtti -fexceptions -Wpointer-arith"
    abiFilters 'armeabi-v7a', 'arm64-v8a'
}

```

再次编译成功

##### <3> 编写 hook 代码

SandHook 使用和上面 inlinehook 框架基本一样

 

首先声明旧的函数，编写新的函数（目标函数 strstr）

 

![](https://bbs.pediy.com/upload/attach/202205/905443_PTP33M4B8NPEF7Y.png)

 

然后进行 hook

 

![](https://bbs.pediy.com/upload/attach/202205/905443_VWARWTJFRXBE7BG.png)

 

最后发现可以成功 hook

 

![](https://bbs.pediy.com/upload/attach/202205/905443_RDJ2MVC9GYJQ8M3.png)

 

SandHook 使用姿势：

```
（1）导包，将SandHook中cpp文件夹下的包全部导入到项目中，并修改CMakeLists.txt中添加native.cpp, 修改java层导入so库为sandHook-native
（2）配置相关的环境
    在配置文件build.gradle中配置
    externalNativeBuild {
          cmake {
        arguments '-DBUILD_TESTING=OFF'
        cppFlags "-frtti -fexceptions -Wpointer-arith"
        abiFilters 'armeabi-v7a', 'arm64-v8a'
          }
        }
（3）编译可以成功通过
（4）使用
    const char * libc = "/system/lib64/libc.so";
    old_fopen = reinterpret_cast(SandInlineHookSym(libc, "fopen",
                                                                         reinterpret_cast(new_fopen)));
参数2：hook的函数 参数3：新的函数
 
添加原理hook旧函数的声明
void* (*old_fopen)(char*,char*);
实现新的函数功能
void* new_fopen(char* a,char* b){
    __android_log_print(6,"windaa","I am from new open %s",a);
    return old_fopen(a,b);
}
（5）运行测试是否成功启动 
```

### 4.PLT/GOT hook 实操

前面我们已经介绍了 Got 表 hook 的原理，下面我们实例操作一下导入表函数的 hook

 

参考博客：https://www.cnblogs.com/goodhacker/p/9306997.html

 

原理：

```
通过解析elf格式，分析Section header table找出静态的.got表的位置，并在内存中找到相应的.got表位置，这个时候内存中.got表保存着导入函数的地址，读取目标函数地址，与.got表每一项函数入口地址进行匹配，找到的话就直接替换新的函数地址，这样就完成了一次导入表的Hook操作了

```

![](https://bbs.pediy.com/upload/attach/202205/905443_ZQSAKZF2U7B5GNE.png)

 

首先，我们编写 demo

 

![](https://bbs.pediy.com/upload/attach/202205/905443_QM94VN789TPJJTK.png)

 

我们编译后使用 010Editor 打开`libnative-lib.so`

 

![](https://bbs.pediy.com/upload/attach/202205/905443_EKYS5SE78TQUKNV.png)

 

![](https://bbs.pediy.com/upload/attach/202205/905443_GVKDZZSTB8VBTEA.png)

 

然后我们用 ida 打开，并直接跳转到该地址

 

![](https://bbs.pediy.com/upload/attach/202205/905443_V3PY8VJY2TYXEEX.png)

 

在 got 表中我们找到对应的 mywin0 函数

 

![](https://bbs.pediy.com/upload/attach/202205/905443_6WBC3YU8VG6KQMG.png)

##### <1> 获得 so 模块的加载地址

我们可以使用`/proc/self/maps`去获得 so 模块的加载地址

```
char line[1024];
    int *start;
    int *end;
    int n=1;
    //1.拿到so的起始地址
//    749e5d7000-749e5db000 r--p 000f4000 103:09 441                           /system/bin/linker64
//    749e5db000-749e5dc000 rw-p 000f8000 103:09 441                           /system/bin/linker64
    FILE *fd = fopen("/proc/self/maps","r");
    while (fgets(line,sizeof(line),fd)){
        if(strstr(line,"libnative-lib.so")){
            __android_log_print(6,"windaa","%s",line);
            if(n==1){
                start = reinterpret_cast(strtoul(strtok(line, "-"),NULL,16));
                end = reinterpret_cast(strtoul(strtok(NULL, " "),NULL,16));
            }
            else{
                strtok(line,"-");
                end = reinterpret_cast(strtoul(strtok(NULL, " "),NULL,16));
            }
            n++;
        }
    } 
```

##### <2> 找到 got 表的位置

我们首先根据段头找到 section_header 的首地址

 

![](https://bbs.pediy.com/upload/attach/202205/905443_Q7NVUZ5B2X57YS4.png)

 

![](https://bbs.pediy.com/upload/attach/202205/905443_TVKKU6G357P3HWZ.png)

 

然后我们遍历这个表就可以找到. got, 然后根据 got 表地址再轮训找到函数地址

 

因为这种方法不能在内存中直接找到段头，内存中会抹去段头，所以我们可以通过加载 so 文件来定位

 

![](https://bbs.pediy.com/upload/attach/202205/905443_K8TVAMSKQ6KQC6U.png)

##### <3> 定位到节表的地址

然后我们来获得节表的地址：

```
//读取elf文件
    Elf64_Ehdr ehd;
    int fp =open("/data/local/tmp/libnative-lib.so", O_RDONLY);
    if(fp == -1){
        __android_log_print(4,"windaa","%s","error");
    }
    //读取elf文件的文件头
    read(fp,&ehd,sizeof(Elf64_Ehdr));
    //读取节表的地址
    unsigned long shof = ehd.e_shoff;
    //读取节表的数量
    int shnum = ehd.e_shnum;
    //读取每个节表的大小
    int shsize = ehd.e_ehsize;
    //记录一下str表的偏移，主要是获取后面got的字符串值
    int shstr = ehd.e_shstrndx;

```

我们打印一下此事 shof 的值，验证一下节表的地址

 

![](https://bbs.pediy.com/upload/attach/202205/905443_VJRHUBSV8PK6TYC.png)

 

这里可以发现成功读取

##### <4> 定位到 got 表的位置和函数位置

然后我们拿到字符串的偏移值进行定位到 got 表，再进一步定位到函数

```
//2.拿到字符串表
    Elf64_Shdr shdr;
    //定位字符串，节表地址加字符串表偏移×节表个数
    lseek(fp,shof+shstr*shsize,SEEK_SET);
    //此时节表就定位到字符串表开头
    read(fp,&shdr,shsize);
    //分配一个字符串表大小
    char* strtable = (char *)malloc(shdr.sh_size);
    __android_log_print(6,"windaa","shdrsize %p",shdr.sh_offset);
    //将字符串片指针移动到0x34104上
    lseek(fp,shdr.sh_offset,SEEK_SET);
    read(fp,strtable,shdr.sh_size);
    //将指针移动到节表开头
    lseek(fp,shof,SEEK_SET);
 
    //遍历查找到got
    for(int i=0;i(mywin0) == value)
                {
                    __android_log_print(6,"windaa","value %p",value);
                    //替换mywind地址
                    // 获取当前内存分页的大小
                    uint64_t page_size = getpagesize();
                    // 获取内存分页的起始地址（需要内存对齐）
                    //page要保护的是函数的绝对地址，而不是相对地址
                    uint64_t entry_page_start = (uint64_t)(saddr+j/4) & (~(page_size - 1));
                    // 修改内存属性为可读可写可执行
                    if(mprotect((uint64_t*)entry_page_start, page_size, PROT_READ | PROT_WRITE | PROT_EXEC) == -1){
                        __android_log_print(6,"windaa","%s","mprotect failed");
                    }
                    value = (uint64_t)mywin1;
                    //将mywind0函数的地址换成mywind1函数的地址
                    memcpy((saddr+j/4),&value,16);
                }
 
 
 
            }
        }
    } 
```

![](https://bbs.pediy.com/upload/attach/202205/905443_TX7ZCATWPAERB55.png)

 

这里我们就可以发现成功的 hook

 

got hook 使用姿势：

```
（1）使用/proc/self/maps去获得so模块的加载地址
（2）使用ElfHeader找到Section的首地址，并计算offset和size来获取StringTable
（3）找到got表位置，计算其内存位置，并指针指向got表首地址
（4）遍历got表中的函数，找到要hook的函数，使用mprotect进行hook
（5）将hook的函数地址替换为我们定义的函数地址

```

### 5.Unicorn hook 使用

这里我们简单了解一下基于 unicorn 的框架 Unidbg 的 hook 使用

 

开源地址：https://github.com/zhkl0228/unidbg

 

这里我们直接 idea 将项目拉取下来，然后等下项目环境配置完成

 

![](https://bbs.pediy.com/upload/attach/202205/905443_KR77QZB8XUD3JQQ.png)

 

配置完成后，我们直接启动里面的示例代码查看 hook 效果

 

![](https://bbs.pediy.com/upload/attach/202205/905443_JJDDRDSMTYJCX7D.png)

 

这里 unidbg 使用了 xHook，xHook 是一种 PLT hook 的方式，当然这只是 unidbg 强大功能其中的一种，也是 hook 技术中一种，这里就简单介绍到这，后续再详细讲如何使用

 

unidbg 使用参考博客：https://www.qinless.com/670

[](#六、实验总结)六、实验总结
-----------------

本文从程序加载的原理出发，讲解了当下常用的一些基本的 hook 方式和手段，后续对其中一些 hook 方式再次深入讲解，实验的一些样本和代码会上传到知识星球和 github，文章参考学了了很多大佬的文章和大佬星球的内容，参考文献放在末尾，有什么问题，就请各位大佬一一指出了。

 

github 的地址：[github](https://github.com/WindXaa)

参考文献
----

参考书籍：

```
参考书目：《程序员的自我修养——链接、装载与库》

```

GOT 和 PLT:

```
https://www.geek-share.com/detail/2774116640.html
https://www.jianshu.com/p/0ac63c3744dd
https://www.zhihu.com/question/21249496
https://www.codeleading.com/article/37234101170/

```

hook 技术

```
https://zhuanlan.zhihu.com/p/389889716
https://mabin004.github.io/2018/07/31/Mac%E4%B8%8A%E7%BC%96%E8%AF%91Frida/
https://zhuanlan.zhihu.com/p/269441842
https://blog.csdn.net/sdoyuxuan/article/details/78481239
https://www.cnblogs.com/codingmengmeng/p/6046481.html
https://blog.csdn.net/sssssuuuuu666/article/details/78788369
https://www.malwaretech.com/2015/01/inline-hooking-for-programmers-part-1.html
https://juejin.cn/post/6844903993668272141

```

got/plt hook:

```
https://www.likecs.com/show-203321775.html
https://www.lmlphp.com/user/65342/article/item/709806/

```

[【看雪培训】《Adroid 高级研修班》2022 年春季班招生中！](https://bbs.pediy.com/thread-271992.htm)

最后于 4 天前 被随风而行 aa 编辑 ，原因：

[#基础理论](forum-161-1-117.htm) [#逆向分析](forum-161-1-118.htm) [#NDK 分析](forum-161-1-119.htm) [#程序开发](forum-161-1-124.htm) [#HOOK 注入](forum-161-1-125.htm) [#源码框架](forum-161-1-127.htm)