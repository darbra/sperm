> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ashenone66.cn](https://ashenone66.cn/2021/09/20/duo-chong-zi-shi-hua-yang-shi-yong-frida-zhu-ru/)

> 大 A 的个人博客，用于记录学习过程

[](#前言 "前言")前言
--------------

  多种姿势花样使用 Frida 注入是怎么回事呢？Frida 相信大家都很熟悉，但是多种姿势花样使用 Frida 注入是怎么回事呢，下面就让小编带大家一起了解吧。  
  多种姿势花样使用 Frida 注入，其实就是用不止一种方式注入 Frida，大家可能会很惊讶 Frida 为什么要用多种方式注入？但事实就是这样，小编也感到非常惊讶。  
  这就是关于多种姿势花样使用 Frida 注入的事情了，大家有什么想法呢，欢迎在评论区告诉小编一起讨论哦！

  在实战中使用 Frida 会遇到各种各样的问题来对你进行限制，因此在这里总结和对比一下自己在实战中使用过的一些 frida 的注入方式。关于注入方式的归纳在 [sandhook 的文档](https://github.com/asLody/SandHook/blob/master/doc/doc.md#%E8%BF%9B%E7%A8%8B%E6%B3%A8%E5%85%A5)中有一小段提到过，本文主要讲下如何将这些方式真正的应用到 Frida 上。

[](#方式一：直接使用 "方式一：直接使用")方式一：直接使用
--------------------------------

思路：  
  开启 root 后将 frida-server push 到手机并启动，然后使用`frida -U`命令连接。这是最简单而普遍的方式，网上的 Frida 教程基本上都是这么教的没啥好说的，能够满足大部分 app 的 hook 需求，容易被检测。

具体操作：

> 1.  手机 root（部分可参考附录）
> 2.  下载手机对应架构的 frida-server，一般选择 android-arm64
> 3.  将下载好的 frida-server 解压后改名（以防检测）为 fs 或其他名字，push 到手机 data/local/tmp 目录
> 4.  修改 frida-server 权限，chmod 777 fs
> 5.  直接运行或者修改端口运行（以防检测），改端口使用命令./fs -l 0.0.0.0:12345
> 6.  若未改端口，使用 frida -U 命令连接；若修改端口，先进行端口转发 adb forward tcp:12345 tcp:12345，然后使用命令 frida -H 127.0.0.1:12345 连接
> 7.  遇到无法 attach 目标进程时尝试使用 - f 参数 spawn 一个进程

优点：

> *   简单
> *   可应对大部分 app

缺点：

> *   需要 root
> *   容易被检测

适用场景:  
  适用大部分场景，在手机已 root 的情况下可以优先使用这种方式尝试 hook，如果失败则根据具体情况选择下面提到的其他方式。

[](#方式二：重打包 "方式二：重打包")方式二：重打包
-----------------------------

思路：  
  将 apk 解包后通过修改 smali 或者 patch so 的方式植入 frida-gadget，然后重打包安装，能够达到无 root 的情况下对单个进程 hook 效果。

具体操作：

> 1.  修改 smali：在 app 入口处（一般是 MainActivity 或者 Application 的静态代码区）添加 System.loadLibrary(“frida-gadget”)，然后在 apk 的 lib 目录添加 libfrida-gadget.so（最好改下名字）
> 2.  patch so：取出 lib 目录中确定会被 app 加载的一个 so 文件，最好是没加密和混淆过的，使用 lief 进行 patch，然后替换原来的 so 文件，并加入 frida-gadget，具体 patch 方式参考附录。
> 3.  签名绕过，自行分析，可能难度较大
> 4.  使用工具：可以使用 objection patchapk 一键完成重打包，原理是修改 smali，但无法绕过签名校验。也可以使用 ratel 重打包工具手动完成。ratel 平头哥是一个重打包沙箱，自带 sandhook 和签名校验绕过，可以配合使用让 ratel 将 frida-gadget 注入进去，当然也可以直接使用自带的 sandhook。

优点：

> *   能在未 root 的手机中使用，所以能够绕过 root 检测
> *   无需 ptrace，能绕过一些 ptrace 自身的保护

缺点：

> *   要绕过签名校验，这很难，特别是大厂 app

适用场景：

> *   如果能绕过签名校验的话，重打包的方式还是比较推荐的，如果 app 的签名校验很弱或者根本没有签名校验，使用重打包的方式是不错的选择

[](#方式三：Patch-SO "方式三：Patch SO")方式三：Patch SO
--------------------------------------------

思路：  
  Patch SO 可以用在重打包的方式中，也可以单独拿出来用。重打包的方式不介绍了，这里讲另一种方式。由于无法绕过签名校验，所以可以 patch /data/app/pkgname/lib/arm64(or arm) 目录下的 so 文件，apk 安装后会将 so 文件解压到该目录并在运行时加载，修改该目录下的文件不会触发签名校验。  
  Patch SO 的原理可以参考 [Android 平台感染 ELF 文件实现模块注入](https://gslab.qq.com/portal.php?mod=view&aid=163)

具体操作：

> 1.  根据手机架构，从 apk 中提取待 patch 的 so 文件
> 2.  安装并使用 lief，能很方便的 patch so 文件，为其添加 frida-gadget 的依赖
> 3.  替换 lib 目录下的 so 文件并加入 frida-gadget，这里可以使用临时 root 或 twrp 的方式完成，具体请参考附录

优点：

> *   可以绕过签名校验
> *   可以绕过 root 检测
> *   可以绕过部分 ptrace 自身的保护

缺点：

> *   需要解 bl 锁，部分极端 app 可能会检测 bl 锁
> *   高版本系统下，当 manifest 中的 android:extractNativeLibs 为 false 时，lib 目录文件可能不会被加载，而是直接映射 apk 中的 so 文件

适用场景：

> *   当遇到一个 app 有 root 检测和签名校验而无法轻松绕过时，可以尝试这种方式，前提是 app 没有检测 bl 锁，如果检测了 bl 锁且无法绕过，还是直接放弃这个方案吧。部分 app 可能存在 so 文件被 patch，或者加载了 frida-gadget 后被检测的情况，具体原因还需单独分析。

[](#方式四：编译系统 "方式四：编译系统")方式四：编译系统
--------------------------------

  自己编译系统，修改系统源码，将 frida-gadget 集成到系统中，使得 app 在启动时首先动态加载 frida-gadget。我们其实能直接修改系统源码来影响 app 的动态执行过程，且不引入 frida 特征，但是集成 frida 的优势在于能够 hook app 自身的代码，且修改 hook 点较为方便。

优点：

> *   无需 root
> *   可回锁 bl 锁
> *   无需绕过签名校验

缺点：

> *   编译系统比较麻烦
> *   自己编译系统极易被风控检测，有时即使不嵌入 frida 也无法正常运行 app，或触发其敏感功能

适用场景：

> *   以上三种方式均失效，而 app 能在 aosp 或 lineage 系统中正常运行时，可以考虑这种方式（也是孤注一掷了）。

[](#总结 "总结")总结
--------------

  在实战中可以根据 app 自身的保护情况来选择合适的方式来注入，前提是 app 没有针对 frida。以上情况并不一定都能百分之百成功，不同 app 可能会有自己独特的保护方式。

[](#附录 "附录")附录
--------------

### [](#lief代码示例 "lief代码示例")lief 代码示例

lief 的 API 具体参考 [lief 官方文档](https://lief-project.github.io//doc/latest/tutorials/09_frida_lief.html)

python

```
import lief

libnative = lief.parse("path of ELF source")
libnative.add_library("libfg.so") 
libnative.write("path of patched ELF output")

```

### [](#frida-gadget的使用 "frida-gadget的使用")frida-gadget 的使用

  frida-gadget 一般要配合一个 config 文件来使用，frida-gadget 在被加载后会去读取自身所在目录下的配置文件，配置文件必须以一定的格式命名，例如：frida-gadget 的名称为 libfg.so，那么配置文件的名称则必须时 libfg.config.so，否则该配置文件不会被读取。当没有读取到配置文件时，frida-gadget 会使用默认配置。  
  配置文件的编写，我个人常用以下配置

json

```
{
  "interaction": {
    "type": "listen",
    "address": "127.0.0.1",
    "port": 27042,
    "on_load": "resume"
  }
}

```

默认配置是

json

```
{
  "interaction": {
    "type": "listen",
    "address": "127.0.0.1",
    "port": 27042,
    "on_port_conflict": "fail",
    "on_load": "wait"
  }
}

```

  连接时要使用 frida -U gadget 而不是 frida -U pkgname。具体参考 [frida-gadget 官方文档](https://frida.re/docs/gadget/)

### [](#临时root和twrp的方式修改系统目录 "临时root和twrp的方式修改系统目录")临时 root 和 twrp 的方式修改系统目录

  为了在避免被 root 检测的前提下修改系统目录，我们可以采取临时 root 或使用 twrp 的方式。

#### [](#临时root的方式 "临时root的方式")临时 root 的方式

> 1.  提取系统 boot.img，可以从刷机包中提取，也有其他方式自行百度
> 2.  安装 magisk manage app
> 3.  使用 magisk manage app 对 boot.img 进行 patch
> 4.  将 patch 后的 img 拷贝到 PC
> 5.  手机连接 PC，使用 adb reboot bootloader，进入 bootloader
> 6.  使用 fastboot boot boot_patch.img 启动手机
> 7.  adb shell 后输入 su 检查是否已获取 root
> 8.  完成系统修改后重启则 root 权限清除

#### [](#使用twrp的方式 "使用twrp的方式")使用 twrp 的方式

> 1.  下载 twrp.img
> 2.  手机连接 PC，使用 adb reboot bootloader，进入 bootloader
> 3.  使用 fastboot boot twrp.img 启动手机，进入 twrp
> 4.  若有必要，挂载 system 分区和其他用得上的分区
> 5.  此时 adb shell 即为 root 权限
> 6.  修改完成后重启

  建议使用 twrp 这种方式，相对简单，而且能够修改 system 分区，顺便一提这个技巧的还有一个应用是能够在无 root 的情况下将证书安装到系统目录下。