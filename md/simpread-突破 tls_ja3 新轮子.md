> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dti6j1OFH6VW3m_vw-pCFA)

我之前的文章介绍了 SSL 指纹识别

[https://mp.weixin.qq.com/s/BvotXrFXwYvGWpqHKoj3uQ](https://mp.weixin.qq.com/s?__biz=MzI5Mjg3MzM3OA==&mid=2247483806&idx=1&sn=92d0e3c000613ac56edc2aef163de026&scene=21#wechat_redirect)

很多人来问我 ByPass 的方法

#### 主流的 BYPASS 方法有两大类：

1.  使用定制 ja3 的网络库 go 在这块的库比较流行（比如 go 的库 requests 还有 cycletls） 缺点在于，就是得用 go 语言开发（cycletls 有 nodejs 的但是也是开了一个 go 语言的一个 websocket）
    
2.  魔改 openssl, curl，最有名的就是 curl-impersonate 对应 win 版本的 https://github.com/depler/curl-impersonate-win 缺点就是编译复杂，使用方式上，得用包装 curl 的第三方库，用的多就是 python 的 pycurl 其他语言的比较少
    

正好五一有时间，站在巨人们的肩膀上我用 go 语言开发了一个代理服务.

只需要设置这个代理服务，就可以自定义 ja3 参数

**这样任何语言都可以直接用了，而且只需要加一个 webproxy 即可**

**具体效果可以往下看**

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTVpsrgbdHLTEOmGmtNv1oklrCKiaQbIzlIEU5eYLJFyVnUT2WsesEedw/640?wx_fmt=png)image

解压后如上图，包含 2 个文件 (文件包加我要)

*   ja3proxy.exe  **go 开发的一个控制台程序**
    
*   localhost_root.pfx 本地证书
    

为了方便本机测试，先安装 localhost_root.pfx 证书， （如果不安装证书，也可以运行，只不过你需要将请求忽略 ssl verify）![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTG3DjhAz7fFm912HT9a5UamXNprmQ5SzPUIeC6W2icSHGq8cVyrMiaRMg/640?wx_fmt=png)

证书密码为 123456

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTsh5A8ITagib1usObDEriczHtYgUjvpIygpS7GF3h5PEluabaicfR7Ep3Q/640?wx_fmt=png)image

选择位置为：受信任的根证书颁发机构

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTNQyjniaMb5xMe6swXv0U4Co11mFTbEMlSibqs29RmseY8DVP8QJOqBLg/640?wx_fmt=png)image

安装成功后，使用如下命令 运行 ja3proxy.exe （为了测试方便，用 win 版本）

```
ja3proxy.exe -pfxFile=localhost_root.pfx -pfxPwd=123456



```

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTbiaYdS5jVTRlrtR0LsTR4n2iaKqnKAdAFo0UswSakX8iaOQwxTqzxqicxQ/640?wx_fmt=png)image

支持的参数共有如下：

*   pfxFile （pfx 类型证书）
    
*   pfxPwd (pfx 证书的密码)
    
*   authName 如果你要开启 ja3proxy 代理服务的 basicauth 认证, 可以设置
    
*   authPwd (同上)
    
*   httpPort （ja3proxy 代理的 http 端口，默认为 8080）
    
*   httpsPort （ja3proxy 代理的 https 端口，默认为 8443）
    
*   certFile 非 pfx 类型证书可以设置
    
*   keyFile 同上
    

ja3proxy 运行成功后，测试 csharp 代码如下：

```
var proxy = new WebProxy
{
    // 这就是我们的ja3proxy
 Address = new Uri($"http://localhost:8080")
};

var httpClientHandler = new HttpClientHandler
{
 Proxy = proxy,
};

// 因为我们再上面把证书添加到本机受信任了 所以这行代码不需要，如果你不操作受信任证书的话，就需要
//httpClientHandler.ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;


var client2 = new HttpClient(handler: httpClientHandler, disposeHandler: true);

// 设置ja3指纹
client2.DefaultRequestHeaders.Add("tls-ja3","771,4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,17513-10-18-11-51-13-27-0-35-65281-43-16-45-5-23-21,29-23-24,0");
// 设置ja3proxy执行请求的超时
client2.DefaultRequestHeaders.Add("tls-timeout","10");
// 设置ja3proxy执行请求用代理，设置后请求目标服务器拿到的就是代理ip
// client2.DefaultRequestHeaders.Add("tls-proxy","http://252.45.26.333:5543");

// 设置当前请求的useragent
client2.DefaultRequestHeaders.UserAgent.ParseAdd("Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36");


var result = await client2.GetStringAsync("https://kawayiyi.com/tls");
Console.WriteLine(result);



```

执行后，校验 ja3 一致

```
{
  "sni": "kawayiyi.com",
  "tlsVersion": "Tls13",
  "tcpConnectionId": "0HMQ8N2PQCRQE",
  "random": "AwN2jHvxe/TKafrfmZ1KG2JWrD7u6M1N4dpeIGdYQwA=",
  "sessionId": "FvNiwCLizsA2JZt0/8865tX2A5VsfbgjlCu4Qg4jPjg=",
  "tlsHashOrigin": "771,4865-4866-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,17513-10-18-11-51-13-27-0-35-65281-43-16-45-5-23-21,29-23-24,0",
  "tlsHashMd5": "05556c7568c3d3a65c4e35d42f102d78",
  "cipherList": [
    "TLS_AES_128_GCM_SHA256",
    "TLS_AES_256_GCM_SHA384",
    "TLS_CHACHA20_POLY1305_SHA256",
    "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
    "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
    "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
    "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
    "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
    "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
    "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA",
    "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA",
    "TLS_RSA_WITH_AES_128_GCM_SHA256",
    "TLS_RSA_WITH_AES_256_GCM_SHA384",
    "TLS_RSA_WITH_AES_128_CBC_SHA",
    "TLS_RSA_WITH_AES_256_CBC_SHA"
  ],
  "extentions": [
    "extensionApplicationSettings",
    "supported_groups",
    "signed_certificate_timestamp",
    "ec_point_formats",
    "key_share",
    "signature_algorithms",
    "compress_certificate",
    "server_name",
    "session_ticket",
    "renegotiation_info",
    "supported_versions",
    "application_layer_protocol_negotiation",
    "psk_key_exchange_modes",
    "status_request",
    "extended_master_secret",
    "padding"
  ],
  "supportedgroups": [
    "X25519",
    "CurveP256",
    "CurveP384"
  ],
  "ecPointFormats": [
    "uncompressed"
  ],
  "proto": "HTTP/2",
  "h2": {
    "SETTINGS": {
      "1": "65536",
      "3": "1000",
      "4": "6291456",
      "5": "16384",
      "6": "262144"
    },
    "WINDOW_UPDATE": "15663105",
    "HEADERS": [
      ":method",
      ":authority",
      ":scheme",
      ":path"
    ]
  },
  "user_agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36",
  "clientIp": "103.219.192.197"
}



```

h2 的 header 顺序：m,a,s,p 和 chrome 保持一致

nodejs 测试也没问题

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTuYC1QyYKgI2MuQabSSbcCGIHIePex3uAb2rHicmVpOR5CzSO0HAU6nQ/640?wx_fmt=png)

### 原理

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5RZsvvbInbxFNib9ZCibl9MkTNWx89ay9Kpp0rm477UC7OlJOPSE7QePQjomGsLPYATzYT8Ir1fNaDQ/640?wx_fmt=png)

ja3proxy（是一个中间人）接管你的请求，然后自己去目标建立 tls，clienthello 就用你指定的 ja3 参数

### 关于我

![](https://mmbiz.qpic.cn/mmbiz_png/ticEicibZjRw5SO4BTsscauC9SWo8eyZrg6U1eDTKAqIwJuPGscic7zM8KDFy5dsVYYMG6sibUic6ibYtELDkXRvypDNQ/640?wx_fmt=png)image

微软最有价值专家是微软公司授予第三方技术专业人士的一个全球奖项。27 年来，世界各地的技术社区领导者，因其在线上和线下的技术社区中分享专业知识和经验而获得此奖项。

MVP 是经过严格挑选的专家团队，他们代表着技术最精湛且最具智慧的人，是对社区投入极大的热情并乐于助人的专家。MVP 致力于通过演讲、论坛问答、创建网站、撰写博客、分享视频、开源项目、组织会议等方式来帮助他人，并最大程度地帮助微软技术社区用户使用 Microsoft 技术。

更多详情请登录官方网站 [https://mvp.microsoft.com/zh-cn](https://mvp.microsoft.com/zh-cn)