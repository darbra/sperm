> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/11398)

> 先知社区，先知安全技术社区

1. 前言
-----

本文将从网络通信原理浅析在 android 中出现的一些代理转发检测，这些功能会使我们测试 app 时出现**抓不到包**或者**应用闪退**等情况，针对这种场景，我搭建了测试环境，并对其场景展开分析与实施应对方案。

2.OSI 7 层网络模型
-------------

网络通信嘛，首先得知道什么是 OSI 7 层模型。下面是百度的解释：

> 为了使不同计算机厂家生产的计算机能够相互通信，以便在更大的范围内建立计算机网络，国际标准化组织（ISO）在 1978 年提出了 “开放系统互联参考模型”，即著名的 OSI/RM 模型（Open System Interconnection/Reference Model）。
> 
> 它将计算机网络体系结构的通信协议划分为七层，自下而上依次为：
> 
> [物理层](https://baike.baidu.com/item/%E7%89%A9%E7%90%86%E5%B1%82)（Physics Layer）、[数据链路层](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E9%93%BE%E8%B7%AF%E5%B1%82)（Data Link Layer）、[网络层](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E5%B1%82/4329439)（Network Layer）、[传输层](https://baike.baidu.com/item/%E4%BC%A0%E8%BE%93%E5%B1%82)（Transport Layer）、[会话层](https://baike.baidu.com/item/%E4%BC%9A%E8%AF%9D%E5%B1%82)（Session Layer）、[表示层](https://baike.baidu.com/item/%E8%A1%A8%E7%A4%BA%E5%B1%82)（Presentation Layer）、[应用层](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E5%B1%82/16412033)（Application Layer）

使用网络数据的传输离不开网络协议七层模型，通过理解每一层协议的分工，也就能对网络故障逐一排查，这样的思维逻辑在安卓应用中也同样适用。

**OSI 7 层模型**各层功能及对应的协议、设备如下表所示：

<table><thead><tr><th>OSI 对应的层</th><th>功能</th><th>TCP/IP 对应的协议</th><th>设备</th></tr></thead><tbody><tr><td>应用层</td><td>文件传输，电子邮件，文件服务，虚拟终端</td><td>TFTP，HTTP，SNMP，FTP，SMTP，DNS，Telnet</td><td>/</td></tr><tr><td>表示层</td><td>数据格式化，代码转换，数据加密</td><td>/</td><td>/</td></tr><tr><td>会话层</td><td>解除或建立与别的接点的联系</td><td>/</td><td>/</td></tr><tr><td>传输层</td><td>提供端对端的接口</td><td>TCP，UDP</td><td>四层交换机和四层路由</td></tr><tr><td>网络层</td><td>为数据包选择路由</td><td>IP，ICMP，RIP，OSPF，BGP，IGMP</td><td>三层交换机和路由</td></tr><tr><td>数据链路层</td><td>传输有地址的帧以及错误检测功能</td><td>ARP，RARP，MTU，SLIP，CSLIP，PPP</td><td>网桥、交换机、网卡</td></tr><tr><td>物理层</td><td>以二进制数据形式在物理媒体上传输数据</td><td>ISO2110，IEEE802，IEEE802.2</td><td>中继器、集线器、双绞线</td></tr></tbody></table>

知识点：HTTPS 协议是`HTTP+SSL`

根据上表可知，SSL 做数据加密是在表示层，也就是说，HTTPS 实际上是建立在 SSL 之上的 HTTP 协议，而普通的 HTTP 协议是建立在 TCP 协议之上的。所以，当 HTTPS 访问 URL 时，由于 URL 在网络传送过程中最后是处于 HTTP 协议数据报头中，而 HTTP 协议位于 SSL 的上层，所以凡是 HTTP 协议所负责传输的数据就全部被加密了；但是 IP 地址并没加密，因为处理 IP 地址的协议（网络层）位于处理 SSL 协议（表示层）的下方。

额，说了这么多，就是要告诉你一个重要的关键点：数据的封装是`自下而上`的 ！在网络数据处理方面，如果是上层做了检测处理，则需要在同层或下层进行逻辑绕过，这就是攻与防的关键了，偷家（底层）才是硬道理。

接下来，我们再理解一下代理与 VPN。

3. 代理与 VPN
----------

### 3.1、代理

**代理（proxy）** 也称网络代理，是一种特殊的网络服务，允许一个终端（一般为客户端）通过这个服务与另外一个终端（一般为服务器）进行非直接的连接。

一个`完整的代理请求过程`为：客户端首先根据代理服务器所使用的**代理协议**，与**代理服务器**创建连接，接着按照协议请求对目标服务器创建连接、或者获得目标服务器的指定资源。

### 3.2、VPN

**VPN**（virtual private network）（**虚拟专用网络** ）是常用于连接中、大型企业或团体间私人网络的通讯方法。它利用**隧道协议（Tunneling Protocol）**来达到发送端认证、消息保密与准确性等功能。

### 3.3、代理和 VPN 的区别

从各自的定义，我们就能看出 VPN 的特点是采取**隧道协议**进行数据传输和保护；而代理使用的则是对应的**代理协议**。

下面是 VPN 和代理的常用协议：

<table><thead><tr><th></th><th>协议名称</th></tr></thead><tbody><tr><td>VPN</td><td>OpvenVPN、IPsec、IKEv2、PPTP、L2TP、WireGuard 等</td></tr><tr><td>代理</td><td>HTTP、HTTPS、SOCKS、FTP、RTSP 等</td></tr></tbody></table>

VPN 协议大多是作用在 OSI 的第二层和第三层之间，所以使用 VPN 时，几乎能转发所有的流量。

而代理协议多作用在应用层，最高层。

4. 安卓代理检测
---------

知道了代理与 VPN 的作用后，在 APP 中，如果开发人员在代码中添加了一些网络层的检测机制，而这些机制恰恰又是针对工作层协议进行的检测，那么只要分析出工作在 IOS 的哪一层，抢先一步在下层做出应对，那 APP 在上层无论怎么检测，都没有用。下面将对测试场景进行详细分析。

抓包的步骤：

1. 在客户端（手机）中设置代理服务器的地址

2. 开启代理服务器（burp）的代理功能

如果在客户端对代理服务进行过滤，禁止客户端通过代理服务器进行访问 Internet，添加如下代码：

```
connection = (HttpURLConnection) url.openConnection(Proxy.NO_PROXY);


```

官方对于 **Proxy.NO_PROXY** 的描述如下：

```
/**
 * A proxy setting that represents a {@code DIRECT} connection,
 * basically telling the protocol handler not to use any proxying.
 * Used, for instance, to create sockets bypassing any other global
 * proxy settings (like SOCKS):
 * <P>
 * {@code Socket s = new Socket(Proxy.NO_PROXY);}
 *
 */public final static Proxy NO_PROXY = new Proxy();

// Creates the proxy that represents a {@code DIRECT} connection.private Proxy() {
    type = Type.DIRECT;
    sa = null;
}


```

**NO_PROXY** 实际上就是 type 属性为 **DIRECT** 的一个 Proxy 对象，这个 type 有三种：

*   DIRECT
*   HTTP
*   SOCKS

所以，Proxy.NO_PROXY 的意思是 connection 的请求是**直连**。

此时若通过系统进行代理，app 对外请求会失效，也就是视觉上看到的卡死状态，就是不让走系统代理。

安卓手机上设置**系统代理**即是在【设置】-【WLAN】-【修改网络】手动设置代理。

**针对不走系统代理的情况有如下两种应对：**

1、使用基于`VPN`模式的`Postern`

2、使用基于`iptables`的`ProxyDroid`

对此，我做出了如下一些测试：

### 4.1、使用系统代理

APP 关键代码如下：

```
private void sendRequestWithHttpURLConnection(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try{
                    URL url = new URL("http://www.baidu.com");
                    connection = (HttpURLConnection) url.openConnection(Proxy.NO_PROXY);
                    connection.setRequestMethod("GET");
                    InputStream in = connection.getInputStream();

                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null){
                        response.append(line);
                    }

                    showResponse(response.toString());


                } catch (Exception e){
                    e.printStackTrace();
                } finally {
                    if (reader != null) {
                        try {
                            reader.close();
                        } catch (IOException e){
                            e.printStackTrace();
                        }
                    }
                    if (connection != null){
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }


```

针对 **Proxy.NO_PROXY**，先测试一下，系统代理是否真的不能抓包。

如下图先设置系统代理，burp 监听 8888，此时打开 APP，点击发送请求无任何反应，burp 中也抓不到包，说明系统代理被禁了。

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512144301.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512144301.png)

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512162425.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512162425.png)

### 4.2、使用 Postern 代理

用过这款软件的都知道，当开启代理服务后状态栏会有个`钥匙`的标志，这可能也是基于 VPN 模式工作的特征

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512162635.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512162635.png)

同样的 APP，点击请求，此时成功绕过了 Proxy.NO_PROXY 检测！也说明了 VPN 协议在 HTTP 协议的下层。

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512163122.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512163122.png)

5. 安卓 VPN 检测
------------

VPN 也是代理的一种，但是由于通讯协议的差异，所以检测代码也不一样。

当客户端运行`VPN虚拟隧道协议`时，会在当前节点`创建`基于 eth 之上的`tun0`接口或`ppp0`接口，所以一旦出现带有明显特征的网络接口名称，就可以认定是使用了 VPN 协议进行通信。

下面这段代码的检测方式：出现特征 **tun0** 或者 **ppp0** 则退出应用，也就是我们看到的闪退效果。

```
private void isDeviceInVPN() {
        try {
            Enumeration<NetworkInterface> networkInterfaces = NetworkInterface.getNetworkInterfaces();
            while (networkInterfaces.hasMoreElements()) {
                String name = networkInterfaces.nextElement().getName();
                if (name.equals("tun0") || name.equals("ppp0")) {
                    stop();
                }
            }
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }


```

在点击监听中放置 isDeviceInVPN() 功能，点击即触发，如果检测到了使用了 VPN 则直接退出。

```
@Override
    public void onClick(View view){
        if (view.getId() == R.id.send_request){
            isDeviceInVPN();
            sendRequestWithHttpURLConnection();

        }
    }


```

### 5.1、使用 ProxyDroid 代理

当前场景：APP 同时开启了代理检测以及 VPN 检测

这时使用 **iptables** 进行数据转发的软件 **ProxyDroid** 进行测试，配置如下图所示：

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512164343.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512164343.png)

开启之后，系统状态栏不会出现钥匙的形状，这时再次进行抓包测试。

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512164629.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512164629.png)

burp 成功获取到了请求，至此代理与 VPN 的应对方法均已实现。所以，**iptables** 竟然能从 OSI 的 2、3 层下面走吗，下面我们继续分析。

6.iptables 原理
-------------

我们都知道安卓使用的是 linux 内核，而 linux 内核提供的防火墙工具是 **Netfilter/Iptables**。

**Netfilter** 是由 linux 内核集成的 IP 数据包过滤系统，其工作在内核内部，而 **Iptables** 则是让用户定义规则集的表结构。

也就是，**iptables** 是一个命令行工具，位于用户空间，它真正操作的框架实现在**内核**当中。

> Netfilter 是一个数据包处理模块，它具有`网络地址转换`、`数据包内容修改`、`数据包过滤`等功能。 要使 netfilter 能够工作，就需要将所有的规则读入内存中。netfilter 自己维护一个内存块，在此内存块中有 4 个表：filter 表、NAT 表、mangle 表和 raw 表。在每个表中有相应的链，链中存放的是一条条的规则，规则就是过滤防火的语句或者其他功能的语句。也就是说表是链的容器，链是规则的容器。实际上，每个链都只是一个 hook 函数（钩子函数）而已。

**Iptables** 主要工作在 OSI 七层的 2.3.4 层，好像也没比 VPN 的工作协议低，反而还有高的，但是测试结果证明，是我想错了，iptables 不是由于协议低，而是没有出现 **tun0** 或者 **ppp0** 这两个关键的网卡特征，所以成功绕过了 VPN 的检测。

基于 iptables 这个流量转发，我还发现了一个新的名词，叫做 “`透明代理`”，iptables 的转发模式就是这种。

由此，延伸了一个新的代理模式，通过 burp 进行 “透明代理”，网上的教程错综复杂，亲测使用过程如下。

7. 透明代理
-------

*   原理：透明代理技术可以让客户端`感觉不到代理的存在`，用户不需要在浏览器中设置任何代理，只需设置缺省网关即可。在访问外部网络时，客户端的数据包被发送到缺省网关，通过缺省网关的路由，最终到达代理服务器，最后代理服务器运行代理进程，数据实际被重定向到代理服务器的代理端口，即由本地代理服务器向外请求所需数据然后拷贝给客户端。

接下来我将尝试：结合安卓端的透明代理技术与 burp 存在的 invisible 模式

### 7.1、使用 Burp 透明代理

#### （1）安卓端设置

首先在设备上手动进行设置：将所以请求 80、443 端口的 tcp 流量进行 nat 转发到 192.168.50.177（burp 的监听地址）的对应端口上

```
adb shell
su
iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to  192.168.50.177:80
iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to  192.168.50.177:443


```

查看当前规则是否成功添加

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512174024.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512174024.png)

#### （2）代理服务器端设置

添加 80 和 443 的端口监听

在【Binding】中设置端口，选中 【All interfaces】

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155619.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155619.png)

并对【Request handing】做出如下设置

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155224.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155224.png)

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155347.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155347.png)

> **Redirect to port** - 如果配置了这个选项，Burp 会在每次请求转发到指定的端口，而不必受限于浏览器所请求的目标。
> 
> **Force use of SSL** - 如果配置了这个选项，Burp 会使用 HTTPS 在所有向外的连接，即使传入的请求中使用普通的 HTTP。您可以使用此选项，在与 SSL 相关的响应修改选项结合，开展 sslstrip 般的攻击使用 Burp，其中，强制执行 HTTPS 的应用程序可以降级为普通的 HTTP 的受害用户的流量在不知不觉中通过 BurpProxy 代理。

设置之后，Proxy 状态如下

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155443.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220511155443.png)

此时 burp 就可对转发到这里的 80 和 443 端口的流量进行**透明代理**

> 注意：如果出现 443 端口被占用，查找进程 kill 掉即可。
> 
> [![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154711616.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154711616.png)
> 
> [![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154455104.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154455104.png)
> 
> 以管理员身份运行 cmd 执行如下代码
> 
> [![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154419965.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/image-20220511154419965.png)

经过测试，burp 成功抓取到了请求包。

**这里不禁思考，如果是基于 iptables 进行的数据转发，那么刚才的 ProxyDroid 是否也内置了一些路由规则呢？**

查看一下开启 ProxyDroid 时 iptables 当下的规则

[![](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512175635.png)](https://raw.githubusercontent.com/dummersoul/Picture/main/img/20220512175635.png)

从图中可以看到共有六条策略，其中最后两条就是我们刚才手动添加的，但并没有看到 burp 监听的 8888 端口，8123、8124 一定是软件内置的代理转发端口，想要知道具体原理还需要详细分析 ProxyDroid 的源码。

**血泪避坑**：网上出现了很多教程，最关键的 iptables 规则写法不一，导致多次测试结果并不成功，如果将安卓终端的 80 和 443 端口同时转发到 burp 上监听的唯一一个端口则会出现连接错误。根据 burp 官方文档说明为每个端口号设置监听器会更加稳定，也就是要设置两个代理监听。

8. 总结
-----

根据不同的代码检测，也会有不同的应对方法，所以，遇到 APP 出现抓包闪退等问题，先逆向，查看源码，在通信处仔细进行分析，再针对检测代码进行绕过，才是正解。本文提到的并不是固定的处理方法，如果文章有叙述不当，尽请矫正。

9. 参考链接
-------

[burp invisible 官方文档](https://portswigger.net/burp/documentation/desktop/tools/proxy/options/invisible)

[代理与 VPN](https://mp.weixin.qq.com/s/u4WwEGFADvRIYFudrMDsRQ)

[iptables 的内核原理](https://cloud.tencent.com/developer/article/1619659)