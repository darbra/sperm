> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3JmQCLP5falWZVUqgLDzew)

声明
--

**本文仅供学习交流，如有侵权请联系删除**

检测
--

抖音采用的是谷歌的`Cronet`网络协议栈，这个`Cronet`是从谷歌浏览器剥离出来的。

在`Chromium/net/socket/ssl_client_socket_impl.cc`源码中可以看到与抖音`libsscronet.so`中相似的伪代码，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpY52C8y8GOm3GVLW65icsvyMHXEJLYHCiaVUq4lZcgdIhasz1vKBEHjicuib2OmX21f8vRy3hxU7118g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

  

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpY52C8y8GOm3GVLW65icsvybVx6xZ9DecEiaTXGibRb6WTGiahKI7zgkX3kVGy6giaYOLLmJsVo4JYiatg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)

  

检测方法就是`SSL_CTX_set_custom_verify`，这个方法最终会调用`boringssl`中的`SSL_CTX_set_custom_verify`。其中`mode`参数的取值就是下面的三种情况，`callback`回调函数就是自定义证书校验逻辑。

```
// SSL_VERIFY_NONE, on a client, verifies the server certificate but does not
// make errors fatal. The result may be checked with |SSL_get_verify_result|. On
// a server it does not request a client certificate. This is the default.
#define SSL_VERIFY_NONE 0x00

// SSL_VERIFY_PEER, on a client, makes server certificate errors fatal. On a
// server it requests a client certificate and makes errors fatal. However,
// anonymous clients are still allowed. See
// |SSL_VERIFY_FAIL_IF_NO_PEER_CERT|.
#define SSL_VERIFY_PEER 0x01

// SSL_VERIFY_FAIL_IF_NO_PEER_CERT configures a server to reject connections if
// the client declines to send a certificate. This flag must be used together
// with |SSL_VERIFY_PEER|, otherwise it won't work.
#define SSL_VERIFY_FAIL_IF_NO_PEER_CERT 0x02

void SSL_CTX_set_custom_verify(
    SSL_CTX *ctx, int mode,
    enum ssl_verify_result_t (*callback)(SSL *ssl, uint8_t *out_alert));


```

如何过
---

给出两种解决方案，第一种是使用`frida`直接将`mode`改为`0x0`，第二种就是`patch so`文件，硬编码一下`so`，然后重新放回去。

```
function hook_dlopen1(module_name) {
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
    if (android_dlopen_ext) {
        Interceptor.attach(android_dlopen_ext, {
            onEnter: function (args) {
                var pathptr = args[0];
                if (pathptr) {
                    this.path = (pathptr).readCString();
                    console.log(this.path);
                    if (this.path.indexOf(module_name) >= 0) {
                        this.canhook = true;
                    }
                }
            },
            onLeave: function (retval) {
                if (this.canhook) {
                    let verifyadd = Module.getExportByName("libsscronet.so", "SSL_CTX_set_custom_verify");
                    Interceptor.attach(verifyadd, {
                        onEnter(args) {
                            console.log(args[1])
                            args[1] = ptr(0x0)
                        }
                    })
                }
            }
        });
    }
}


```

第二种使用`ida`自带的`keypatch`，将`[#1](javascript:;)`改为`0`后，保存`so`文件，重新放入，赋予对应权限后既可使用。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpY52C8y8GOm3GVLW65icsvyDbU9bNZibqk4yQCdKPF2ZtiaLQAUxcliabVklJcOdNcstWAj8jVQiajEicg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

效果
--

可以看到，`reply`接口评论可以正常抓取，其他接口也正常，但还有小部分是`connect`失败，该方法对于`tk`抓包同样有效，至于`quic`协议，尝试过`wireshark`抓取，但是连`quic`的影子都没看到，不知道哪个接口用了。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpY52C8y8GOm3GVLW65icsvy5tVQpcFR8aFrPNaiazVCEg7VQzU3XHLGwGm3q0fgP8hFyibvGdgUIpvw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3)

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpY52C8y8GOm3GVLW65icsvyVV4oxae7Z3asH1qsP1MAHhYMs7IraD2iaHWBw5BJK7n1SBcsw4Lnf0w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)