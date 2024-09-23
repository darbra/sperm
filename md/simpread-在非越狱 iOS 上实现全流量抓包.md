> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dy534TfqE-XyL6DnYL-fcw) ![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/3eicVGzibzClD0a7dtdzY0gD0yGOibA89F0ibfSflicOPUTicGibW26q43fe72hq7T1RraxeSbBFB8xQHiaQsbARJG8jgQ/640?wx_fmt=jpeg&from=appmsg)看到题图应该都懂了吧

背景
==

昨晚在测试一个手机应用抓包的时候发现使用 WiFi 的 HTTP 代理无法抓到包，并且在 Burp 的日志中没有看到任何 SSL Alert，因此可以判断不是证书问题；另外虽然没有抓到包但是应用依然可以正常收发数据，这说明代理的配置被绕过了。

应用开发者要想绕过用户端的代理配置有很多办法，这种情况在笔者之前的博文中已经介绍过不少了，比如：

*   • [终端应用安全之网络流量分析](https://mp.weixin.qq.com/s?__biz=MzA3MzU1MDQwOA==&mid=2247484053&idx=1&sn=8b9dd1ad310f0c602bfde12be360a688&scene=21#wechat_redirect "终端应用安全之网络流量分析")
    
*   • [iptables 在 Android 抓包中的妙用](https://mp.weixin.qq.com/s?__biz=MzA3MzU1MDQwOA==&mid=2247484228&idx=1&sn=1393d6d90b27cc565f5c76197fe05812&scene=21#wechat_redirect "iptables 在 Android 抓包中的妙用")
    

但是这次的情况有点不一样。首先笔者常用的用于近源钓鱼的 WiFi 热点工具没有带回家，其次手头上没有 Andorid 手机只有一个 iPhone，且手机由于日常使用并没有越狱 (属于是强行给自己上强度了)。

初步尝试
====

在现有条件下，还是先尝试了正常的抓包方案，使用 Burpsuite 启动代理并在手机端安装好证书，正常抓包发现没有问题。但是正如前面所说，在使用目标应用时发现在一些关键流量上并没有展现，由此判断是应用代码中特意忽略了系统代理配置。

这种情况下一般需要尝试透明代理或者 VPN 抓包方案，我们先考虑后者。在 iOS 中我们有 shadowsxxksR、Surge 等应用，不过都要收费，而且这些软件主要还是用于非 MITM 场景的代理。

在互联网上检索一番发现有个叫 Stream[1] 的免费软件，基于 iOS 9+ 的 Network Extension Api 进行 HTTP/HTTPS 网络流量的劫持。虽然最近一次更新已经是 4 年前了，但现在基本功能还是可用的。

使用该工具点点点之后发现已经能抓到我们想要的请求包了。按理说，我们的任务已经完成了，但是，用别人现成的工具，**你的价值体现在哪里？这件事为什么是你做不是别人做？当初招你进来的时候……**

…… 开个玩笑。其实这个工具也有一些不是很完美的地方，比如抓包和改包只能在手机上点，不适合自动化；还比如对于一些非 HTTP 流量其实也没有办法抓到。

因此我们接着看有没有其他更优雅的解决方案。

WireGuard
=========

既然手法限制在了 VPN 抓包，那就不得不提 WireGuard 了。

WireGuard 是一种新型的 VPN（虚拟专用网络）协议和免费开源软件，通过 UDP 封装 IP 数据包，提供了一种既安全又高效的 VPN 解决方案。它适用于点对点连接、中心辐射型网络拓扑以及全网状网络拓扑。其设计哲学是简化 VPN 的配置和使用，同时不牺牲安全性和性能。

WireGuard 的安装和使用可以参考官网的文章：

*   • Wireguard - install[2]
    
*   • Wireguard - quickstart[3]
    
*   • Source Code Repositories and Official Projects[4]
    

WireGuard 的配置核心在于秘钥路由特性 (Cryptokey Routing)。作为一个网络接口 (eth0、wlan0、tun0)，会根据路由表进行实际传入和传出网络的分发。WireGuard 同样运行在网络接口层，只不过会在分发前进行网络包的加解密，并且使用 UDP 进行传输。

基本使用
----

在 Linux 中新建网卡和设置路由表都可以通过 `ip` 命令操作，这里新建一个名为 `wg0` 的网络接口：

```
# 网卡配置
ip link add wg0 type wireguard
ip addr add 10.0.0.1/24 dev wg0
ip link set wg0 up

```

然后使用 `wg` 命令对该接口进行配置，包括设置私钥、端口和 peer 等信息。

```
# 生成公私钥
wg genkey > private
# 添加私钥
wg set wg0 private-key ./private
# 设置端口
wg set wg0 listen-port 51820
# 添加端点
wg set wg0 peer <peer-公钥> allowed-ips 192.168.88.0/24 endpoint 209.202.254.14:8172

# 可以组合成一句话
wg set wg0 listen-port 51820 private-key './private' peer 'ABCDEF...' allowed-ips '192.168.88.0/24' endpoint '209.202.254.14:8172'

```

这些配置何况使用配置文件的方式进行管理。

```
wg setconf wg0 ./config.ini

```

也可以不用手动创建网卡而是使用 wg-quick 脚本一键部署：

```
sudo wg-quick up config.ini

```

> `wg` 和 `wg-quick` 都是 wireguard-tools 的一部分

以下面一个 Server 的配置为例：

```
[Interface]
PrivateKey = yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=
ListenPort = 51820

[Peer]
PublicKey = xTIBA5rboUvnH4htodjb6e697QjLERt1NAB4mZqp8Dg=
AllowedIPs = 10.192.122.3/32, 10.192.124.1/24

[Peer]
PublicKey = gN65BkIKy1eCE9pP1wdc8ROUtkHLF2PfAqYdyYBz6EA=
AllowedIPs = 10.10.10.230/32

```

其中 `Interface` 指定接口的私钥和监听端口，`Peer` 则指定了客户端的公钥和 IP 网段。用简化的说法，对于发送到子网 `10.10.10.230/32` 的请求，会使用该 Peer 的公钥进行加密；对于发送到本机接口的请求，则用本地私钥进行解密。

对应 Client 的配置如下：

```
[Interface]
PrivateKey = gI6EdUSYvn8ugXOt8QQD6Yc+JyiZxIhp3GInSWRfWGE=
ListenPort = 21841

[Peer]
PublicKey = HIgo9xNzJMWLKASShiTqIybxZ0U3wGLiUeJ1PKf8ykw=
Endpoint = 192.95.5.69:51820
AllowedIPs = 0.0.0.0/0

```

其中在 Interface 字段指定了本机的私钥，Peer 字段指定服务器的公钥和地址信息。前面用到 `wg genkey` 命令来生成私钥，同时也可以用 `wg pubkey` 查看对应私钥的公钥：

```
$ wg pubkey <<< gI6EdUSYvn8ugXOt8QQD6Yc+JyiZxIhp3GInSWRfWGE=
gN65BkIKy1eCE9pP1wdc8ROUtkHLF2PfAqYdyYBz6EA=
$ wg pubkey <<< yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=
HIgo9xNzJMWLKASShiTqIybxZ0U3wGLiUeJ1PKf8ykw=

```

可以看到客户端和服务端配置的一大区别是客户端配置指定的 `Peer` 包含 Endpoint 字段，指定服务器的地址和端口。

服务端上除了 WireGuard 本身的配置还要注意网卡的配置，一般情况下可能需要调整本机防火墙的策略，对于需要将虚拟网卡流量转发到实际网络接口上的场景还需要配置内核 IP 转发以及路由转发的规则，详情可参考：

*   • 在 Ubuntu22.04 上设置 WireGuard[5]
    
*   • Routing & Network Namespace Integration[6]
    
*   • Share wg0 over eth0[7]
    
*   • Access internal devices through the WireGuard tunnel[8]
    

Wireshark
---------

我们可以在手机上使用 WireGuard 作为 VPN 客户端，同时在 PC 上运行 WireGuard 服务端即可以获取所有手机上的网络流量。不过这里有个问题是网卡收到的数据是加密的 UDP 数据，因此我们需要先将其解密还原成原始数据。

其实在 Wireshark 中已经支持了 WireGuard 流量解密的功能。在 Wireshark Edit -> Preferences -> Protocols -> WireGuard 中可以设置 static key 和 keylog 对加密的 UDP 流量进行解密。

这个解密过程和 SSL/TLS 的解密过程类似，其中需要 wireguard-tools 中的 extract-handshakes.sh[9] 脚本来在获取 WireGuard 握手过程中的临时秘钥，该脚本基于 kprobe 来 hook 内核中 wireguard 驱动的相关代码来提取秘钥，因此只能用于 Linux 系统。

为什么即使有双方的 Peer 私钥也无法对会话进行解密？因为握手过程使用随机生成的临时公私钥对进行 Diffie-Hellman 密钥交换，以生成共享会话密钥。这些临时密钥不会被存储且仅用于当前会话，可以认为 Peer 私钥仅参与验证而不参与加密，因此也就无法解密流量。即便某一个会话的秘钥泄露也不会影响其他会话的保密性，这个就叫前向安全 (Forward Security)。

不过话说回来，对于我们的抓包场景其实还不需要介入到解密过程，因为实际流量是进入到我们自己的机器上，然后才转发到实际的服务器，后者的流量是已经解密的。因此我们可以直接抓取网络网卡 (如 eth0) 其实也是能看到解密后的流量的。甚至我们可以直接抓取 Wireguard 网卡(如 wg0) 也能获得明文流量，从而实现了 “全流量抓包” 的愿景。

*   • WireGuard / Wireshark[10]
    
*   • Decoding WireGuard with WireShark[11]
    
*   • WireGuard：抓包和实时解密 [12]
    

MITMProxy
---------

使用 WireGuard+Wireshark 我们已经实现了在 iOS 中抓取所有流量的目的，包括 UDP 和 TCP 的链接也包括 ICMP 等 IP 层数据。不过还是有一点点小瑕疵，其中获取到的 HTTPS 流量还是 TLS 加密的。

解决这个问题的方法也很多，一个最直接的方法是通过 iptables 将进入本机 (server) 443 端口的 TCP 流量转发到其他透明 HTTP 代理端口，比如 Burpsuite 或者 Yakit，借助他们的 MITM 功能来实现 HTTPS 流量加解密和重放。

不过，既然都引入了三方抓包工具，其实有一个更为便捷的方案。那就是使用 MITMProxy，其在 9.0 版本之后引入了了一种新的透明代理模式，即 wireguard mode，使用方法非常简单，如下所示：

```
mitmproxy -m wireguard

```

其会自动生成一个 WireGuard 客户端配置文件，例如：

```
[Interface]
PrivateKey = mLorvtz8GDGUH+Qm+voV4wFg2r6yJFN/tCRq4fY/Rm8=
Address = 10.0.0.1/32
DNS = 10.0.0.53

[Peer]
PublicKey = lyZoK//28trI/gSaTTaxKEce4RlU2Sxx9VafyWlqR34=
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.31.21:51820

```

在手机上直接使用 WireGuard 导入该配置或者将其转为二维码扫描即可连入基于 MITMProxy 的 VPN，所有流量都会被 MITMProxy 进行劫持和转发，从而实现便捷的抓包。

值得一提的是在该模式下 MITMProxy 并不会真正地使用 WireGuard 内核模块，而是使用了其自身的应用层实现，mitmproxy-wireguard。所以观察网卡也不会看到 wg0 等虚拟接口。这样的好处是避免了系统的依赖性，同时功能也会有所裁剪，比如传统 VPN 的子网互通就无法实现。

mitmproxy-wireguard 用 Rust 编写，目前已经转移到了 `mitmproxy_rs` 仓库中，感兴趣的可以去查看其源码实现。

*   • A more user-friendly transparent mode, based on WireGuard[13]
    
*   • mitmproxy_rs，前生为 mitmproxy-wireguard[14]
    

总结
==

本文探讨了几种在非越狱 iOS 手机上进行抓包的方案，除了使用如 Surge、Stream 等商业工具，还可以使用 WireGuard、Wireshark、MITMProxy 等开源工具实现最初的目标。在基于 WireGuard 的方案中，Wireshark 可以确保抓取所有流量不会有遗漏，在某些游戏应用中可能会遇到类似的自定义 TCP/UDP 数据流；而使用 MITMProxy 则可以方便地对 HTTPS 流量进行解密，同时基于应用层可以实现平台无关性。

与 Android 相比，如果只是日常抓包可能 iOS 会更方便一点。因为 Android 7 及其以后都默认不信任用户添加的根证书，想加证书抓包要么 root 要么重打包，相对门槛会高一些；而 iOS 还是一如既往的可以随意添加证书，对于日常使用偶尔抓包分析的场景倒是比较方便了。

#### 引用链接

`[1]` Stream: _https://blog.csdn.net/weixin_44504146/article/details/121946958_  
`[2]` Wireguard - install: _https://www.wireguard.com/install/_  
`[3]` Wireguard - quickstart: _https://www.wireguard.com/quickstart/_  
`[4]` Source Code Repositories and Official Projects: _https://www.wireguard.com/repositories/_  
`[5]` 在 Ubuntu22.04 上设置 WireGuard: _https://cainiaojiaocheng.com/%E5%A6%82%E4%BD%95%E5%9C%A8Ubuntu22.04%E4%B8%8A%E8%AE%BE%E7%BD%AEWireGuard_  
`[6]` Routing & Network Namespace Integration: _https://www.wireguard.com/netns/_  
`[7]` Share wg0 over eth0: _https://www.reddit.com/r/WireGuard/comments/nkg77b/share_wg0_over_eth0/_  
`[8]` Access internal devices through the WireGuard tunnel: _https://docs.pi-hole.net/guides/vpn/wireguard/internal/_  
`[9]` extract-handshakes.sh: _https://git.zx2c4.com/WireGuard/tree/contrib/examples/extract-handshakes/README_  
`[10]` WireGuard / Wireshark: _https://wiki.wireshark.org/WireGuard_  
`[11]` Decoding WireGuard with WireShark: _https://blog.salrashid.dev/articles/2022/wireguard_wireshark/_  
`[12]` WireGuard：抓包和实时解密: _https://blog.jinmiaoluo.com/posts/wireguard-with-wireshark_  
`[13]` A more user-friendly transparent mode, based on WireGuard: _https://mitmproxy.org/posts/wireguard-mode/_  
`[14]` mitmproxy_rs，前生为 mitmproxy-wireguard: _https://github.com/mitmproxy/mitmproxy_rs_