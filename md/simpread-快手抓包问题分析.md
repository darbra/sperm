> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.mythsman.com](https://blog.mythsman.com/post/6213072906087a00011b7e4d/)

> 背景 不知从什么版本后，对快手进行简单抓包似乎 “不可行” 了。

背景
--

不知从什么版本后，对快手进行简单抓包似乎 “不可行” 了。表现就是使用常规的 HTTP 正向代理抓包工具（[charles](https://www.charlesproxy.com/)、[mitmproxy](https://mitmproxy.org/)、[fiddler](https://www.telerik.com/fiddler) 之类）并且把自签名证书种到系统证书里后，数据依然能刷出来，也能抓到一些零星的报文，但是关键出数据的那些接口报文都没有。

一般来说，常规方法无法抓安卓应用的 https 包，通常有以下几种可能：

1.  证书信任问题。在 Android 7 以上，应用会默认不信任用户证书，只信任系统证书，如果配置不得当则是抓不到包的。
2.  应用配置了 SSL pinning，强制只信任自己的证书。这样一来由于客户端不信任我们种的证书，因此也无法抓包。
3.  应用使用了纯 TCP 传输私有协议（通常也会套上一层 TLS）。由于甚至都不是 HTTP 协议，当然就抓不到包了。
4.  应用使用 WebSocket 长链接，将不同的接口封装在这个长链接里。在 WebSocket 里承载的协议一般是用某种自定义方式来模拟 http 请求，因此也难以抓包。
5.  应用使用了基于 UDP 的 http3.0 协议等。大部分代理工具目前还不支持 quic 等协议，所以这样一般是抓不到包的。
6.  应用配置了 Proxy.NO_PROXY ，强制不走系统代理。这样 http 流量就绕过我们配置的代理，自然抓不到包。

当前的现象是数据能刷出来，那就说明并不是证书信任相关的问题。接下来就需要验证它究竟是使用了什么样的传输方式，对症下药。

最稳妥的验证方式当然是白盒测试看源码，不过快手反编译的代码看起来也费劲，于是考虑直接当成黑盒来测试看看。以下验证方式均以 **快手 8.2.31.17191** 版本为例。

分析
--

### 环境部署

#### 准备自签名 CA 证书

需要在 Linux 主机上使用 openssl 工具生成一波证书。当然，这一步可以忽略，直接使用 mitmproxy 生成的证书。只是手动配置一下能够加深一下对 openssl 的理解。

```
openssl genrsa -out cert.key 2048


openssl req -new -x509 -key cert.key -out -days 5480 cert.crt


cat cert.key cert.crt > cert.pem


cp cert.pem ~/.mitmproxy/mitmproxy-ca.pem


mitmproxy -p 8000 


curl -x localhost:8000 --cacert ~/.mitmproxy/mitmproxy-ca.pem https://www.baidu.com


openssl x509 -subject_hash_old -in cert.crt -noout


cp cert.crt a5176621.0


```

值得注意的是，不要尝试使用 `mitmproxy --certs` 来配置证书，这种方式只能配置 leaf 证书，而不能配置根 CA 证书。因此还是老老实实的把根证书放在默认路径下。

#### 准备设备

为了方便测试，我在 arm 服务器上使用 [redroid](https://github.com/remote-android/redroid-doc) 准备了一台安卓虚拟机。

```
docker run -itd --rm --memory-swappiness=0 \
--privileged --name redroid \
--mount type=bind,source=/home/myths/exp/a5176621.0,target=/system/etc/security/cacerts/a5176621.0 \
-p 5555:5555 \
redroid/redroid:11.0.0-arm64 \
redroid.width=720 \
redroid.height=1280 \
redroid.fps=15 \
redroid.gpu.mode=guest

```

其中 /home/myths/exp/a5176621.0 替换成实际文件所在路径。

然后在 arm 主机上用 adb 连接安卓的 tcpip 端口，下载并安装快手 8.2.31.17191 版本。

```
adb connect localhost:5555


adb install kuaishou.apk

```

为了方便远程操作，需要在本地机器上连接 arm 服务器上的安卓虚拟机，并用 scrcpy 操作。

```
adb connect <ip for arm server>:5555


scrcpy

```

到这一步骤时，可以检测安卓中的网络应该都已经是通的了。

### 复现抓包问题

先尝试使用传统正向代理的方式进行抓包。

```
mitmproxy -p 8000


adb shell settings put global http_proxy 172.17.0.1:8000

```

设置完成后，观察手机的网络状况，现象如下：

1.  使用浏览器访问普通网站，返回均正常，mitmproxy 也能抓到包。
2.  刷快手推荐页，返回也正常，但是 mitmproxy 只能抓到一些静态资源，无法抓到接口。

![](https://blog.mythsman.com/content/images/2022/02/Xnip2022-02-20_15-30-21.png)

### Ban 掉不走代理的所有流量

有数据但是抓不到包，怀疑应当是有些流量漏掉了，于是尝试把这些流量 Ban 掉看看效果。

```
echo 1 > /proc/sys/net/ipv4/ip_forward


mitmproxy -p 8000


adb shell settings put global http_proxy 172.17.0.1:8000



sudo iptables -t nat -I PREROUTING -p tcp -s 172.17.0.12 ! -d 172.17.0.1 -j REDIRECT --to-ports 1234

sudo modprobe ipt_owner


iptables -t nat -D PREROUTING 1

```

设置完成后，观察手机的网络状况，现象如下：

1.  使用浏览器访问普通网站，返回均正常，mitmproxy 也能抓到包。
2.  刷快手推荐页，显示 “无网络连接 “。

说明这里的确是有流量漏了，没有走正向代理，但是依然出了外网。

### Ban 掉不走代理的 443/80 流量

那么这些流量到底是私有的四层 TCP 流量、还是没走正向代理的 80/443 流量呢？因此尝试把非 80/443 的流量放开试试。

```
mitmproxy -p 8000


adb shell settings put global http_proxy 172.17.0.1:8000



sudo iptables -t nat -I PREROUTING -p tcp -s 172.17.0.12  --dport 80 -j REDIRECT --to-ports 1234
sudo iptables -t nat -I PREROUTING -p tcp -s 172.17.0.12  --dport 443 -j REDIRECT --to-ports 1234


iptables -t nat -D PREROUTING 1
iptables -t nat -D PREROUTING 1

```

设置完成后，观察手机的网络状况，现象如下：

1.  使用浏览器访问普通网站，返回均正常，mitmproxy 也能抓到包。
2.  刷快手推荐页，依然显示 “无网络连接 “。

这就说明，控制快手推荐页的流量并不是所谓私有流量，而就是走的 80/443 端口，只是没有走正向代理。

### 改用透明代理模式

既然七层的代理配置会被忽略，那就尝试使用四层的透明代理，将流量强制转到透明代理服务器上即可。

```
mitmproxy -p 8000 -m transparent


adb shell settings put global http_proxy :0



sudo iptables -t nat -I PREROUTING -p tcp  -s 172.17.0.12 --dport 80 -j REDIRECT --to-ports 8000
sudo iptables -t nat -I PREROUTING -p tcp  -s 172.17.0.12 --dport 443 -j REDIRECT --to-ports 8000


iptables -t nat -D PREROUTING 1
iptables -t nat -D PREROUTING 1

```

设置完成后，观察手机的网络状况，现象如下：

1.  使用浏览器访问普通网站，返回均正常，mitmproxy 也能抓到包。
2.  刷快手推荐页，能成功刷出，并且 mitmproxy 也抓到了包。

![](https://blog.mythsman.com/content/images/2022/02/K20220220-174206.png)

总结
--

目前看来，快手并非像很多网上传的那样，大多数接口都走了 kquic 协议导致无法抓包。其实很多接口只是做了一个禁止走系统代理的设置，简单使用透明代理模式进行抓包即可轻松绕过。当然，不排除有些关键接口也做了 SSL pinning、走私有协议之类的。。。

参考
--

[https://docs.mitmproxy.org/stable/concepts-certificates/](https://docs.mitmproxy.org/stable/concepts-certificates/)

[https://stackoverflow.com/questions/56830858](https://stackoverflow.com/questions/56830858)

[https://square.github.io/okhttp/4.x/okhttp/okhttp3/-ok-http-client/-builder/proxy/](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-ok-http-client/-builder/proxy/)