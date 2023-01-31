> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/P0ESUUXBmq2aQnrqDHsDaw)

本文介绍一种在 Andorid 中实现单应用、全局、优雅的抓包方法。

> 本文于去年端午节编写，由于种种原因，当时藏拙并未发布。现删除一些敏感信息后分享出来，希望对各位有所帮助。

背景
==

昨天在测试一个 Android APK 的时候发现使用 WiFi 的 HTTP 代理无法抓到包，在代理的日志中没有发现任何 SSL Alert，因此可以判断不是证书问题；另外 APP 本身仍可以正常收发数据，这说明代理设置被应用绕过了。

根据我们前一篇文章 ([终端应用安全之网络流量分析](https://mp.weixin.qq.com/s?__biz=MzA3MzU1MDQwOA==&mid=2247484053&idx=1&sn=8b9dd1ad310f0c602bfde12be360a688&scene=21#wechat_redirect "终端应用安全之网络流量分析")) 中所介绍的，遇到这种情况时就可以使用路由抓包方法，确保接管所有流量。但是因为端午放假被封印在家，且用于抓包的树莓派放在了公司，因此只有另谋他路。

本来接着考虑装个 DroidProxy 去试一下，但突然间灵光一闪，为什么不直接用 iptables 去修改流量呢？于是，就有了这篇小记。

iptables 101
============

`iptables` 应该大家都不会陌生，说起来这也是我入门 “黑客” 时就接触的命令，因为我的网络安全入门第一战就是使用 `aircrack` 去破解邻居的 WiFi 密码。多年以前还写过一篇 [Linux 内核转发技术](https://mp.weixin.qq.com/s?__biz=MzA3MzU1MDQwOA==&mid=2247484053&idx=1&sn=8b9dd1ad310f0c602bfde12be360a688&scene=21#wechat_redirect "Linux内核转发技术")，介绍 iptables 的常用操作，但当时年幼无知，很多概念自己并没有完全理解。其实介绍 iptables 最好的资料就是官方的 man-pages，因此这里也就不做一个无情的翻译机器人了，只简单介绍一些关键的概念。

basic
-----

首先是我们作为系统管理员最为关心的命令行参数，在坊间流传的各类防火墙、WiFi 热点、流控 shell 脚本中，充斥着各种混乱而难以理解的 iptables 命令，但实际上其命令行参数非常优雅，可以概况为以下表述:

```
iptables [-t table] {-A|-C|-D} chain rule-specification

rule-specification = [matches...] [target]
match = -m matchname [per-match-options]
target = -j targetname [per-target-options]

```

一个 table 中有多个 chain，除了内置的 chain，用户也可以自己新建 (比如 DOCKER 链)。常用的 table 及其包含的 chain 有以下这些:

*   • filter
    

*   • INPUT
    
*   • FORWARD
    
*   • OUTPUT
    

*   • nat
    

*   • PREROUTING
    
*   • INPUT
    
*   • OUTPUT
    
*   • POSTROUTING
    

*   • mangle
    

*   • PREROUTING
    
*   • OUTPUT
    
*   • INPUT
    
*   • FORWARD
    
*   • POSTROUTING
    

*   • raw
    

*   • PREROUTING
    
*   • OUTPUT
    

其中有的表比其他表包含更多的 chain，这是其定位决定的。正如其名字而言，**filter** 主要用于流量过滤，**nat** 表主要用于网络地址转换，**mangle** 表用于数据包修改，而 **raw** 表则用于网络包更早期的配置。除此之外还有 **security** 表用于权限控制，不过用得不多。

虽然看起来各个表各司其职，但实际中也没有强制的差异。比如 mangle 表虽然用来修改流量，但也可以用来做网络地址转换，filter 表也是同理。在日常中设置 iptables 规则的时候主要考虑的是数据包的时序，而这和 chain 的关系更大一些。

上面提到的这些常见 chain，不管在哪个表中，其含义都是类似的:

*   • INPUT: 表示数据包从远端发送到本地；
    
*   • OUTPUT: 表示数据包在本地生成，并准备发送到远端；
    
*   • PREROUTING: 接收到数据包的第一时间，在内核进行路由之前；
    
*   • POSTROUTNG: 表示数据包准备离开的前一刻；
    
*   • FOWARD: 本机作为路由时正要准备转发的时刻；
    

table 结合对应的 chain，网络数据包在 iptables 中的移动路径如下图所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3eicVGzibzClAQn9CxBWn1GMYN5UJjhS0tv6yOYqEPRKDib2VPkzxFLECHFublf0cHzmFTEXJzjepiaOuQibymbrhHA/640?wx_fmt=png)flow

extensions
----------

对于 `iptables` 而言重点无疑是其中的规则定义，上文提到的参数无非就是将自定义的规则加入到对应 CHAIN 之中，比如 `-A` 是将规则插入到链的末尾 (append)，`-I` 是插入到链的头部 (insert)，`-D` 是删除对应规则 (delete)，等等。

而规则又分为两个部分，即数据包匹配以及匹配之后的操作，分别通过 `-m` 和 `-j` 来指定。这其中就引入了成百的命令行参数，以至于社区还就此产生了不少段子:

```
Overheard: “In any team you need a tank, a healer, a damage dealer, someone with crowd control abilities and another one who knows iptables”

— Jérome Petazzoni (@jpetazzo) June 27, 2015

```

不过实际上社区对 iptables 的抱怨更多是在多用户系统中规则配置冲突以及由此引发的艰难调试之旅，在没有冲突的情况下，配置规则也是比较简单的。定义 iptables 规则的参考主要是 iptables-extensions(8)，其中定义了一系列 **匹配拓展 (MATCH EXTENSIONS)** 以及 ** 目标拓展 (TARGET EXTENSIONS)**。

### match

先看匹配拓展，一般我们使用 iptables 都是根据 ip 或者端口进行匹配，比如 `-m tcp --dport 22`。但其中也有一些比较有趣的匹配规则，比如上一篇文章中介绍过的 Android 单应用抓包方法:

```
$ iptables -A OUTPUT -m owner --uid-owner 1000 -j CONNMARK --set-mark 1
$ iptables -A INPUT -m connmark --mark 1 -j NFLOG --nflog-group 30 
$ iptables -A OUTPUT -m connmark --mark 1 -j NFLOG --nflog-group 30 
$ dumpcap -i nflog:30 -w uid-1000.pcap

```

用到了两个匹配拓展，一个是 `owner` 拓展，使用 `--uid-owner` 参数表示创建当前数据包的应用 UID。但是这样只能抓到外发的包，而服务器返回的包由于并不是本地进程创建的，因此没有对应的 UID 信息，因此 owner 拓展只能应用于 `OUTPUT` 或者 `POSTROUTING` 链上。为了解决这个问题，上面使用了另一个拓展 `connmark`，用来匹配 tcp 连接的标志，这个标志是在第一条命令中的外发数据中进行设置的。

还有个值得一提的匹配拓展是 `bpf`，支持两个参数，可以使用 `--object-pinned` 直接加载编译后的 eBPF 代码，也可以通过 `--bytecode` 直接指定字节码。直接指定的字节码格式类似于 `tcpdump -ddd` 的输出结果，第一条是总指令数目。

例如以下 bpf 指令 (ip proto 6):

```
4               # number of instructions
48 0 0 9        # load byte  ip->proto
21 0 1 6        # jump equal IPPROTO_TCP
6 0 0 1         # return     pass (non-zero)
6 0 0 0         # return     fail (zero)

```

实际调用时候需用用逗号分隔每条指令，且不支持注释等其他符号:

```
iptables -A OUTPUT -m bpf --bytecode '4,48 0 0 9,21 0 1 6,6 0 0 1,6 0 0 0' -j ACCEPT

```

对于其他遇到的匹配拓展，可以在官方文档中查看其详细用法。

### target

target 表示数据包匹配之后要执行的操作，一般使用大写表示。标准操作有 ACCEPT/DROP/RETURN 这三个，其他都定义在 target extensions 即目标拓展中。

比如我们前面提到的 `CONNMARK` 就是其中一个拓展，其作用是对当前**链接**进行打标，这样 TCP 请求的返回数据也会带上我们的标记。类似的还有 `MARK` 拓展，表示对当前**数据包**设置标志，主要用于后续 table/chain 的识别。

前面用到的另一个拓展是 `NFLOG`，表示 netfilter logging，规则匹配后内核会将其使用对应的日志后端进行保存，通常与 `nfnetlink_log` 一起使用，通过多播的方式将获取到的数据包发送到 `netlink` 套接字中，从而可以让用户态的抓包程序获取并进行进一步分析。

其他常用的拓展还有 `SNAT/DNAT` 用于修改数据包的源地址和目的地址，`LOG` 可以使内核 dmesg 打印匹配的数据包信息，`TRACE` 可以使内核打印规则信息用于调试分析等。

Android Proxy
=============

复习完 iptables 的基础后，我们继续回到文章开头的问题，有什么办法可以在不设置代理的基础上代理所有流量呢？

这个问题可以从两方面去考虑，即:

1.  1. 如何匹配目标数据包；
    
2.  2. 匹配之后如何转发到代理地址；
    

第一个问题比较简单，我们需要匹配从本地发出的，目的端口是 80/443 的 tcp 流量，因此匹配规则可以写为:

```
-p tcp -m tcp --dport 443

```

在不确定目标 web 服务器端口的情况下，可以将 dport 指定为 `0:65535`，对所有端口都进行劫持转发；当然也可以直接不写 match，默认就是匹配所有 tcp 包。不过可以稍微过滤一下目的地址，比如 `! -d 127.0.0.1`，以免本地的 RPC 请求也被误拦截。

或者，更优雅的方案是使用 `multiport` 来一次性指定多个端口:

```
-m multiport --dports 80,443

```

第二个问题，既然我们需要将流量转发到代理工具，那么可以选择透明代理模式，上篇文章也有提到过。因此一个最简单的方法是使用 DNAT 修改目的地址。查阅文档可知，DNAT 只能用在 `nat` 表中的 `PREROUTING` 和 `OUTPUT` 链。再根据上文中的流程图，如果代理地址在本地，那只能使用 OUTPUT、如果是远程地址，那么两个链任选一个即可。

综上所述，假设 HTTP 透明代理监听在 `127.0.0.1:8080`，那么可以直接用以下方法设置代理并进行抓包:

```
iptables -t nat -A OUTPUT -p tcp ! -d 127.0.0.1 -m multiport --dports 80,443 -j DNAT --to-destination 127.0.0.1:8080

```

更进一步
====

通过这么一条 iptables 命令，配合上透明代理就可以实现全局的 HTTPS 抓包了。所以就这样了吗？回忆一下之前我们其实是可以通过 `owner` target 去进行 UID 匹配的，只不过之前是使用 NFLOG 配合 tcpdump 进行抓包。因此我们其实也可以通过类似的方式实现基于 UID 的透明代理。

转发规则并没有太大变化，只需要在匹配规则上新增一个约束。

```
iptables -t nat -A OUTPUT -p tcp ! -d 127.0.0.1 -m owner --uid-owner 2000 -m multiport --dports 80,443 -j DNAT --to-destination 127.0.0.1:8080

```

这样，不需要额外的路由抓包设备，甚至不需要引入 VPN Service 等其他应用，只需要一行命令即可实现针对单个 Android 应用的全局 HTTP/HTTPS 抓包。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3eicVGzibzClAQn9CxBWn1GMYN5UJjhS0teMf7COv94xzk3JRdwCY0ZD6CKBZ2Fd1SagXsUlZ7O2S6ls3Z8uXibrQ/640?wx_fmt=png)elegant

总结
==

本文主要介绍了 iptables 规则的配置方法，并且实现了一种在 Android 中全局 HTTP(S) 抓包的方案，同时借助 `owner` 拓展实现应用维度的进一步过滤，从而避免手机中其他应用的干扰。

相比于传统的 HTTP 代理抓包方案，该方法的优势是可以实现全局抓包，应用无法通过禁用代理等方法绕过；而相比于 Wireshark 等抓包方案，该方法基于透明代理，因此可以使用 BurpSuite、MITMProxy 等成熟的 HTTP/HTTPS 网络分析工具来对流量进行快速的可视化、拦截 / 重放，以及脚本分析等操作，这些优势是传统抓包方案所无法比拟的。

参考链接
====

*   • iptables(8) — Linux manual page
    
*   • iptables-extensions(8) — Linux manual page
    
*   • Iptables Tutorial 1.2.2
    
*   • Iptables packet flow (and various others bits and bobs)
    
*   • [终端应用安全之网络流量分析](https://mp.weixin.qq.com/s?__biz=MzA3MzU1MDQwOA==&mid=2247484053&idx=1&sn=8b9dd1ad310f0c602bfde12be360a688&scene=21#wechat_redirect "终端应用安全之网络流量分析")