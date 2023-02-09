> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.seeflower.dev](https://blog.seeflower.dev/archives/207/)

> 前言本文结合多篇已有文章，基于 iptables + redsocks2 + Charles，最终实现对安卓上特定 APP 进行抓包，且 APP 无感知即 APP 不能通过检查系统代理或者 VPN 来判断是不是有...

本文结合多篇已有文章，基于`iptables + redsocks2 + Charles`，最终实现对安卓上特定 APP 进行抓包，且 APP 无感知

即 APP 不能通过检查系统代理或者 VPN 来判断是不是有抓包行为

首先先保存开机后的 iptables，如果已经修改过，请重启手机

```
iptables-save > /data/local/tmp/iptables.rules

```

要恢复 iptables 为之前的规则，则使用如下命令，或者重启手机

```
iptables-restore /data/local/tmp/iptables.rules

```

使用如下命令，作用是：将`uid`为`10428`所请求的在`0-65535`端口上的`tcp`流量，转发到`127.0.0.1:16666`，但是排除了来自`127.0.0.1`的请求

参考：[iptables 在 Android 抓包中的妙用](https://mp.weixin.qq.com/s/P0ESUUXBmq2aQnrqDHsDaw)

```
iptables -t nat -A OUTPUT -p tcp ! -d 127.0.0.1 -m owner --uid-owner 10428 --dport 0:65535 -j DNAT --to-destination 127.0.0.1:16666

```

要实现抓包，其中`127.0.0.1:16666`是一个透明代理的地址

然而根据已有文章可知，如果直接设置为 Charles 的透明代理地址，对于 https 将会出现`invalid first line in request`错误，只有 http 的请求数据会被正常解析

参考：[利用 Redsocks 解决透明代理的远程抓包问题](https://blog.mythsman.com/post/62791fb4b5467000017d5c6e/)

根据文章可知，借助`redsocks`进行转发即可

我这里使用的是 [redsocks2](https://github.com/semigodking/redsocks)，编译参考：[静态交叉编译 Android 的 redsocks2](https://fh0.github.io/%E7%BC%96%E8%AF%91/2019/12/07/android-redsocks2.html)

这里直接使用编译好的即可，也可以考虑自己编译下，下载后解压压缩包

*   [https://fh0.github.io/assets/android-redsocks2.tgz](https://fh0.github.io/assets/android-redsocks2.tgz)

创建配置文件，名为`redsocks.conf`，内容如下：

```
base {
    log_debug = off;
    log_info = on;
    log = stderr;
    daemon = off;
    redirector = iptables;
}

redsocks {
    bind = "127.0.0.1:16666";
    relay = "192.168.1.14:8889";
    type = socks5;
    autoproxy = 0;
    timeout = 13;
}


```

其中`bind`就是透明代理地址，`relay`就是 Charles 的代理地址，更多详细用法请查阅`redsocks2`的 readme

注意配置文件的每一对`{}`后面都应该有一个空行，否则会提示`unclosed section`

现在把`redsocks.conf`和`redsocks2_arm64`推送到`/data/local/tmp`

然后在 root 用户下运行`redsocks2_arm64`即可

```
adb push redsocks2_arm64 /data/local/tmp/redsocks
adb shell chmod +x /data/local/tmp/redsocks
adb shell
su
cd /data/local/tmp
./redsocks

```

现在不出意外的话，Charles 应该就能看到`uid`为`10428`所对应 APP 的全部`tcp`数据包了（除去来自 127.0.0.1 的请求）

如果是只想对特定端口抓包，那么应该使用`-m multiport --dports 80,443`这样来限定一个或者多个端口

```
iptables -t nat -A OUTPUT -p tcp ! -d 127.0.0.1 -m owner --uid-owner 10428 -m multiport --dports 80,443 -j DNAT --to-destination 127.0.0.1:16666

```

1.  使用`iptables`将来自特定 uid 的全部 tcp 流量转到指定的透明代理上

```
iptables -t nat -A OUTPUT -p tcp ! -d 127.0.0.1 -m owner --uid-owner 10428 --dport 0:65535 -j DNAT --to-destination 127.0.0.1:16666

```

1.  使用`redsocks`将流量转发到正向代理，如 Charles 的 socks5 代理

**redsocks2_arm64 下载地址如下**

> [https://fh0.github.io/assets/android-redsocks2.tgz](https://fh0.github.io/assets/android-redsocks2.tgz)

**redsocks.conf 内容如下**

```
base {
    log_debug = off;
    log_info = on;
    log = stderr;
    daemon = off;
    redirector = iptables;
}

redsocks {
    bind = "127.0.0.1:16666";
    relay = "192.168.1.14:8889";
    type = socks5;
    autoproxy = 0;
    timeout = 13;
}


```

其他补充：

*   安装 Charles 证书到系统分区，Charles 才能解密 https

如果你发现了文章中的错误，请指出

![](https://blog.seeflower.dev/usr/uploads/2023/02/2884810608.png)

![](https://blog.seeflower.dev/usr/uploads/2023/02/2283399819.png)

*   [iptables 在 Android 抓包中的妙用](https://mp.weixin.qq.com/s/P0ESUUXBmq2aQnrqDHsDaw)
*   [利用 Redsocks 解决透明代理的远程抓包问题](https://blog.mythsman.com/post/62791fb4b5467000017d5c6e/)
*   [静态交叉编译 Android 的 redsocks2](https://fh0.github.io/%E7%BC%96%E8%AF%91/2019/12/07/android-redsocks2.html)
*   [redsocks2](https://github.com/semigodking/redsocks)
*   [默认配置文件报错](https://github.com/semigodking/redsocks/issues/131)
*   [redsocks 搭配 iptables 实现真全局代理](https://anuoua.github.io/2020/05/28/redsocks%E6%90%AD%E9%85%8Diptables%E5%AE%9E%E7%8E%B0%E7%9C%9F%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86/)

  
**广告：星球 & 加群交流**  
![](https://blog.seeflower.dev/images/zsxq.png)