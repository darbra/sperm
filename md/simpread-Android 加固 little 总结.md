> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI3Mzk2OTkxNg==&mid=2247483928&idx=1&sn=f7a18db4947437e78272ad8cd14f8edd&chksm=eb1a658bdc6dec9de4787930105b3ca1a05593011dc3fd1de37de628d54bd4a0a2817662ad23&mpshare=1&scene=1&srcid=05117inmlAXN0CyxSuTjzQBP&sharer_sharetime=1652257242268&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.2.90460&platform=mac#rd)

Android 软件加固
============

Android 软件加固概述
--------------

从 2012 年开始，移动互联网进入快速发展阶段，Android App 开发热潮的兴起，也推动了 Android 平台软件保护技术的发展。

*   • 为何做加固
    

1.  1. 保护核心代码
    
2.  2. 防止营销作弊的手段
    
3.  3. 防止代码被篡改 ...
    

加固代际
----

根据不同的理解，现在加固代际基本上可以按照五代或者三代去区分。

### 第一代：动态加载类

Apk 中没有完整原始的 Dex，需要运行时动态的加载到内存中

#### 原理

*   • 落地加载
    

我们拿到需要加密的 Apk 和自己的壳程序 Apk，然后用加密算法对源 Apk 进行加密再将壳 Apk 进行合并得到新的 Dex 文件，最后替换壳程序中的 dex 文件即可，得到新的 Apk, 那么这个新的 Apk 我们也叫作脱壳程序 Apk. 他已经不是一个完整意义上的 Apk 程序了，他的主要工作是：负责解密源 Apk. 然后加载 Apk, 让其正常运行起来。运行时首先将我们的 Dex 文件或者 Apk 文件解密，然后利用 DexClassLoader 加载器将其加载进内存中，然后利用反射加载待加固的 Apk 的 Appkication，然后运行待加固程序即可。

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawUSsXCE5Yq2eCQ6jRq0DHBTlVuWTLlhTReK431cqsIfTT22tUsHriaNA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iaw7MOGibmdzviajW0twnNJ0FLQwdaXJP94XGWmL4EGcFJ2vDNlAAjdT8PA/640?wx_fmt=png)

*   • 不落地加载
    

落地加载将 Dex 文件解密出来会保存到文件中，再通过 DexClassLoader 加载进内存中，而不落地加载直接重写 DexClassLoader 使其可以直接加载字节数组，避免写入文件中。我们要做的是重写 DexClassLoader，而这涉及到三个函数 defineClass、findClass、loadClass，在一个类被加载的时候，会先后调用这三个函数加载一个类，所以我们需要重写这三个函数。系统的 DexClassLoader 加载 Dex 进入内存的也必然是通过字节加载的，而在系统 so 中的 libdvm.so 中的 openDexFile 可以直接加载 Dex 文件，那么现在清楚了，我们可以通过编写 so 文件调用 openDexFile 函数加载 Dex 字节数组，值得注意的是，openDexFile 函数返回值为一个 int 类型的 cookie，可以简单理解成一个 dex 文件的'身份码'，通过该'身份码'即可操控这个 dex 文件, 至于怎么调用该函数，可以通过 dlopen 和 dlsym 函数调用。

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawqG5NVaF6g0uDRCeFBhIxJvC3DV8EU3fJDHerj1Nced6ib69GBH8czicw/640?wx_fmt=png)

#### 优劣

*   • 优点
    

比较容易实现，无明显的兼容性问题 能有效对抗静态分析和二次打包

*   • 缺点
    

启动时需要进行大量的解密运算，容易造成卡死的情况 在内存中的数据为完整的 Dex，通过动态调试 Dump 内存即可获取完整的 Dex

#### 特点

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawWKIHYkLHcNE9FxicrnPCry1n6hHhcguOmKxlvf2iatFYt3BWtPRb8iaKg/640?wx_fmt=png)

Dex 字符串加密 资源加密 对抗反编译 对抗调试 Dex 动态加载 so 加密

### 第二代：函数抽取类

#### 原理

主要分为两个步骤指令抽取和指令还原

*   • 指令抽取
    

解析原始 Dex 文件格式，保存所有方法的代码结构体信息，通过传入需要置空指令的方法和类名，检索到其代码结构体信息。通过方法的代码结构体信息获取指令个数和偏移地址，构造空指令集，然后覆盖原始指令，重新计算 dex 文件的 checksum 和 signature 信息，回写到头部信息中。

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawVDMiaJwmJgic7Sd4GTxiargzOgxIzD6mxTZokiaXhmuPiaYAsFxz39MJOBg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawaHhZwl7cYVibRwnnUtPCr7oHKCqoXEBib6ctdficqQ8IFq5OzpgT8sxyw/640?wx_fmt=png)

*   • 指令还原
    

native 层 hook 系统函数 dexFindClass，获取类结构体信息（dexFindClass 函数用于查找类的 DexClassDef 结构），获取类中所有的方法信息，通过指定方法名进行过滤，获取该方法的代码结构体信息，获取该方法被抽取的指令集，修改方法对应的内存地址为可读属性，直接进行指令还原。

#### 优劣

*   • 优点
    

加密粒度变小，加密技术从 Dex 文件级变为方法级 按需解密，解密操作延迟到某类方法被执行前，如果方法不被执行，则不被解密 解密后的代码在内存不连续，克服了内存被 Dump 的缺点，有效保护了移动客户端的 Java 代码

*   • 缺点
    

使用大量的虚拟机内部结构，会出现兼容性问题，无法保护所有方法 无法对抗自定义虚拟机 它跟虚拟机的 JIT 优化出现冲突，达不到最佳的性能表现

#### 特点

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawhhuz4E0gjBI9uric4kwn3thf1CcIymIchMhNvIr7ibFlyMcacibIh4QbQ/640?wx_fmt=png)

除一代有的特点外 内存中无完整、连续的 Dex so 代码混淆、膨胀

### 第三代：VMP、Dex2C 类

#### 原理

*   • VMP
    

执行到关键代码时进入壳 so 执行，关于这一点，不同的厂商有着不同的做法，比如把关键函数变成 native 函数，在壳 so 中动态或者静态注册

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iaw2HgrvLGII917rkzAW5zxz8BQuZaNicdl4H0CApNk1xGdRxib8IH09kCw/640?wx_fmt=png)

再比如更改关键方法的方法体

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawgmlqxcJldiaUu7GyeKOEArV46Zn188D10gicGmpjpGbdzVnicZDOQibw6w/640?wx_fmt=png)

无论哪种方法其实都是为了能让壳函数代替原函数去执行 当执行该函数的指令时，解析出指令的 OpCode, 通过一个巨大的 switch case 找到处理对应 OpCode 的函数，然后执行

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawVMpKp5zmz9GPOj1stAhDrK1wLiahrS854o0JCuCeo2gfpHsaaywzX8w/640?wx_fmt=png)

简单讲就是壳将原本的指令进行一次封装，将原本的指令转换为另一种表现形式

*   • Dex2C
    

首先也是将关键代码注册为 native 函数，主要借助于 JNI 反射技术，将 Java 层的方法全部反射为 native 层，增大分析难度。之后再通过混淆、字符串加密等操作生成 so，最后将 so 进行加固保护。

#### 优劣

*   • 优点
    

加固强度高，目前没有公开的脱壳工具 经过混淆加密后很难还原原函数

*   • 缺点
    

效率较低，启动、运行时都比较耗时稳定性、可控性差，容易产生崩溃

#### 特点

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iaw0j5iaUy6Ev5rl2YH1RPUv8htYD88tWMHIQaicp5f1IDjEJdsIRKG6Aww/640?wx_fmt=png)

除一、二代全部特点外 so 代码虚拟化 对抗之前所有的脱壳方法

### so 加密

#### section 加密

*   • 原理
    

将关键方法，存放在自定义的 section 中，通过解析每个 section，将我们自定义的 section 进行加密。因为 so 在加载时会优先加载. init_array，所以将解密方法放在. init_array 中，获取内存中各个 section 的起始地址和大小，将需要解密的 section 还原。

#### 函数加密

*   • 原理
    

解析 so，根据方法名找到指定的方法，将方法进行加密。加载 so 时，获取指定方法的地址，通过解密方法将指定方法解密。

#### 特点

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawRwTe3baicqT915EPpFrojJra4ic1daOVawvhmIVmysYcREIKq9IEb8xQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawuYSKxkoFgn6W5BKld2LS9VicYKrGV6iaqng7PAUwCaeftEe0bUKxs0hw/640?wx_fmt=png)

由于原本的指令已经被加密成其他的字节，IDA 等静态分析工具中会出现大段无法识别的代码

各厂商特征
-----

除此外还有很多大佬们可以自行总结

### 某梆

lib/libDexHelper.so、lib/libDexHelper-x86.so、

### 某加密

assets/ijiami.ajm、assets/ijiami.dat、assets/ijm_lib/libexec.so、assets/ijm_lib/libexecmain.so

### 某企鹅

lib/libshell-super.2019.so、lib/libshella-4.1.0.29.so

### 某数字

assets/libjiagu.so、assets/libjiagu_x86.so

### 某迦

lib/libxloader.so

assets/libvdog、assets/libvdog64、assets/libvdog-x86

### 某付盾

lib/libegis.so、lib/libegis-x86.so

脱壳工具
----

### FRIDA-DEXDump

#### 原理

通过 Frida 在内存中搜索 dex\n035，因为 Dex 的头部都会存在一个 dex\n035 的模数，所以通过在内存中搜索 dex\n035 可以搜索到 Dex 文件。对于一些 Dex 它们被抹去了头部信息，对于这样的情况，FRIDA-DEXDump 也提供了对应的方法，通过遍历当前进程中所有可以读的内存段，通过判断这个段的大小和 Dex 文件中一些关键区域的关系可以判断是否为一个 Dex。

#### 使用

在 FRIDA-DEXDumpGitHub 中下载代码到本地或者通过`pip3 install frida-dexdump` 安装

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawZxUyYtNWIFOcDhsIiaria6qQm86MAptmPiaJDtFwNI9HCABfjyUCwJw1w/640?wx_fmt=png)

用法就是首先打开，我们需要脱壳的软件，然后执行`python main.py -d`程序执行开始检索内存中的 Dex

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawia2DrpJcv0nsaIEv9ZiaLL6BNvCkMpgE0VCNrwqA3v6MXx1WBb4fNh5A/640?wx_fmt=png)

检索到后会保存在 SavePath 对应的目录下

也可以通过`python main.py -h`查看它的其他使用方法

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iaw7SKQ2Yib1WPeSMibC3DbNquS6RIPo5UZXktTZCs2Euia98SHdprTksGVA/640?wx_fmt=png)

最终得到它脱出来的 Dex，可以看下它脱壳效果

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawamw2jjNIpDyAQDyMqxlvGLubW6Rrsfcjm6f1xBPKSNxxekhuF3m0ibA/640?wx_fmt=png)

### Youpk

#### 原理

从 ClassLinker 中遍历所有 DexFile 对象，在虚拟机中 Dex 文件都用 DexFile 对象来表示，并 Dump 出所有 Dex 文件。此时只是整体 Dump，Dex 中的方法还未还原。遍历 DexFile 中的 ClassDef 结构，获取到所有 Class，主动调用 Class 中的所有的方法，让程序强制走 switch 解释器执行，在解释器中添加 Hook 代码，当方法执行时自动保存 CodeItem。根据保存的 CodeItem 和 Dump 下来的 Dex 进行合并，还原 Dex 中被抽取的指令。

#### 使用

首先下载刷机包和还原工具，目前仅支持刷 pixel 1 手机

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawMq8jwTlXyUPtzuP78lIGxkrZ5Xw24xTneXfvxOYypFIZDu5qLaarvg/640?wx_fmt=png)

解压相关的镜像，`fastboot flash xxx /Download/xxx.img`依次刷入

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawMvopyKmGeETQhcOibFATHxP28CXBL2pEf5PKZjFGtkjDOEAhNxC8Idg/640?wx_fmt=png)

配置待脱壳的 App 包名`adb shell "echo com.xxx >> /data/local/tmp/unpacker.config"`

启动 Apk 等待脱壳，每隔 10 秒将自动重新脱壳 (已完全 dump 的 dex 将被忽略), 当日志打印 unpack end 时脱壳完成

dump 文件路径为 / data/data / 包名 / unpacker，使用 adb 命令将文件 pull 出

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawDr7qxwBQ9ZwXBoatVibppo1bq1awbYgjv76Q4icbBTnBKK3eBekicIgIQ/640?wx_fmt=png)

使用 dexfixer.jar 修复 Dex，`java -jar dexfixer.jar /path/unpacker /path/output`

最后我们对比一下还原前后的代码

还原前

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawY0gJlYiaJ4LjqLIQP7mJtjTvo8icvB0Ug2vafEIibmZkAzvpzpjfmGWhA/640?wx_fmt=png)

还原后

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawGjdPoTewQcibhLW8tRXwCZBLOsw86h4FsZZZLibicHhE1XgOb6bEsSdkA/640?wx_fmt=png)

总结
--

本文简单地总结了一下 Android 加固的背景和发展历史，也介绍了一些目前常见的脱壳工具。对于 so 加密的情况，目前也有许多方法应对，比如 ida 动态调试 dump 内存中的 so，GG 模拟器 dump 内存，frida dump so，unidbg 等等。对于混淆的 so，也有 jnitrace，unidbg，还有 hluwa 大佬的大作 obpo 等工具辅助我们分析。除此以外还有还有一些优秀的脱壳工具如 Fart 等，我也没有再做介绍了。总之感谢这些大佬们的努力和开源，没有你们就没有白嫖的我们，哈哈...

 ![](http://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljpJn6DJXbqM8FVX0ynHBmZbKhTD3GgNfA1ylfcbiceVb5dIbMP4aT4B5YnUVKo5IdbJib7A13iccicicw/0?wx_fmt=png) ** 移动安全星球 ** 定期分享移动安全攻防小知识（Android 安全、iOS 安全） 3 篇原创内容  Official Account

![](https://mmbiz.qpic.cn/mmbiz_jpg/uVibiccmzBuliavwY5oaiaiaI6dyib7bnaicyBTHrv6jDzQ9mOiaDRjJKAwRKRjvNXMZaajcumok7sjIicFIsyZ8BoDc0gQ/640?wx_fmt=jpeg)

随手分享、点赞、在看是对我们最大的支持![](https://mmbiz.qpic.cn/mmbiz_gif/uVibiccmzBulgdib3nvLDicu47GtKpYEj2iawMYWia9konOnQuXgEwNKib23Inb506UCbCLLzhbLJ9nj9yjEicJxzKypmQ/640?wx_fmt=gif)