> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CuP48cQvuLRDKKR3t9WUQA)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/DuibU3GqmxVmRsdItbBVRKegNHicHQvAHDdZsGpLVU7touSU1AU1twHTfRjG3Vu5aUh0RnPPllfVUhs4qdWF5QYQ/640?wx_fmt=png&wxfrom=13#imgIndex=0)

声明：文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用，任何人不得将其用于非法用途给予盈利等目的，否则后果自行承担！如有侵权烦请告知，我会立即删除并致歉。谢谢！

文章有疑问的，可以公众号发消息问我，或者留言。我每天都会看的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9zYJrD2VibHmqgf4y9Bqh9nDynW5fHvgbgkSGAfRboFPuCGjVoC3qMl6wlFucsx3Y3jt4gibQgZ6LxpoozE0Tdow/640?wx_fmt=png&wxfrom=13#imgIndex=1)

> 字数 452，阅读大约需 3 分钟

前言
--

项目地址：https://github.com/hackcatml/zygisk-gadget

为什么使用 gadget 可以绕过检测？  
这里要谈到 frida-server hook APP 的原理。  
当我们执行`./frida-server`时，其会主动注入`zygote`进程。

zygote，翻译为孵化器，安卓系统中的每个 APP 都是由其 “孵化” 来的，即每个 APP 启动都需要经过它。

随后，每个启动的 APP，Frida 会将需要的 so 注入 zygote 中。  
高版本的 Frida 会在 frida -U -f 包名 后，才会将 frida-agent.so 注入进去。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3ej53BOHiaXC1A0cujSf9xSeXfClCWN16WSC12Q3u4mPefJM1k2BLH5Q/640?from=appmsg#imgIndex=2)e3e43d2cac9cecfd7595e7de125942ff.png

因为启动 frida-server 会修改 zygote，有的厂商就通过检测 zygote 是否被修改来判断是否被 hook。

对于这种情况，我们就可以选择通过注入 **frida-gadget.so** 的形式，来绕过这个限制。  
我们不依赖 frida-server 注入 so，而是通过其他方法将该文件注入进去。即可过某些企业壳的检测。

基本使用
----

下载 zygisk-gadget-v1.2.1-release.zip  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3WiaIkRruoKm2oDz08iajnusKwB9wW2J3Arb3iaoGloBl45qcG59qu6B6Q/640?from=appmsg#imgIndex=3)8635d8b3d338af38a1594f81b007ae04.png

zygisk 依赖 https://github.com/Dr-TSNG/ZygiskNext

Magisk/kernelsu/apatch 导入 ZygiskNext 后，安装 zygisk-gadget 即可。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3nAJT7AGFAm6K0sfR4JFT1b3RuibZGBwXkGAWSL6aP7kU8xyibUuCSE2w/640?from=appmsg#imgIndex=4)11c260ba09e374462d9ba65bd44d1f65.png

通过命令启动

```
adb shell
su
cd /data/adb/modules/zygisk_gadget/tool/arm64-v8a
chmod +x zygisk-gadget
./zygisk-gadget -p 包名 -d 300000

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3ibVEIONh5SEmaf2fHx5oprp8ibtrGSSZRdYeqeoVPSNrh50dfO615iaHQ/640?from=appmsg#imgIndex=5)9de41a9ba64f4d6b6eff7bfc56c03c26.png

```
frida -U -F -l demo.js

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3dricdccQkmJuFdtyQsrcyWgUaSX4ZZn3OgPVqxGVPJyVpIHlD8Qt5AQ/640?from=appmsg#imgIndex=6)ff90c108b27c95e2cbb467095ec5b528.png

命令选项

```
/data/local/tmp/zygisk-gadget -h                                                                                       
Usage: ./zygisk-gadget -p <packageName> <option(s)>
 Options:
  -d, --delay <microseconds>             Delay in microseconds before loading frida-gadget
  -c, --config                           Activate config mode (default: false)
  -h, --help                             Show help

```

其中内置的魔改的 frida-gadget 来自另一个项目：  
https://github.com/hackcatml/ajeossida

该项目在前人的基础上 Patch 了其他特征  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbrhPw9Vxv8zLYqPepC3sw7b3ILkONdDQjKOf26phX1odF3QgK8R0HsbjBQWlXYKqJab5e0ic8NSdjrg/640?from=appmsg#imgIndex=7) d6d8a6f09b9895631fed82f158866584.png

> 我试过直接用这个项目里的 frida-server，有一定效果，但不如用 gadget 的效果好。

如果想自定义 frida-gadget.config，就在`/data/adb/modules/zygisk_gadget/`路径下，创建 frida-gadget.config

按照官方文档配置即可  
https://frida.re/docs/gadget/

参考资料
----

*   • 神奇日游保护分析——从 Frida 的启动说起 https://github.com/hackcatml/zygisk-gadget
    
*   • https://github.com/hackcatml/zygisk-gadget/issues/1
    

欢迎加入知识星球，领取优惠券，仅需 25~

![](https://mmbiz.qpic.cn/sz_mmbiz_png/a1BOUvqnbria6KN80dMHzudmBQuUfsjkrQ5MkoIbSR9USqojTtHZcicKIZTwU2Tbuib8iafcZNQg0VbyM6ozWTTTSA/640?wx_fmt=png&from=appmsg#imgIndex=8)