> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LS5QKLSYRNFGJm4-YPPTig)

免责声明  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6eccyrNSOic72rU60MknlI9pEoWgY2WcDNygkNG5NtzxaWSibj18LFoScqQ/640?wx_fmt=png)

文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【爬虫逆向小林哥】联系作者

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecnicjTbwdy5ze4PutY5ADhJD1xKy7WpcxMqLhyw93ccIQO4QCP1T30Xg/640?wx_fmt=png)

01

—

前言  

_**好久没发文了，之前有发过通过 hook 方式降级 douyin-quic 协议进行抓包。今天分享从 okhttp3-retrofit2 通信框架拦截 douyin 通信的请求和响应。**_

02

效果  

评论  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAa05BCtGFbxjJyibjW8JMojvrqpPZXTCialQNHhW37JyQbqUHeiafQqVrBQ/640?wx_fmt=png&from=appmsg) 

        橱窗           

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaOGxBC1sHyTp1CD3h0ooMYria1UHNAVS4mo8zFxv34ohPu2KrDGOzSAw/640?wx_fmt=png&from=appmsg)

03

—  

知识点  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaLpR6icybTpDGiaTS2bU5bjnGkkHT9l6jSQADWCuGvouf63xuupI1GgKg/640?wx_fmt=png&from=appmsg)

**okhttp**

> OkHttp 是一个高效的 HTTP 客户端，是目前 Android 使用最广泛的网络框架

特点：支持 http1、http2、quic、websocket；无缝支持 gzip 数据压缩，减少数据流量（抖音的有些 post-data 就是 gzip 发包的）；连接池复用底层 TCP；

```
private void testOkHttp() throws IOException {
//Step1
final OkHttpClient client = new OkHttpClient();
//Step2
final Request request = new Request.Builder()
.url("https://www.google.com.hk").build();
//Step3
Call call = client.newCall(request);
//step4 发送网络请求，获取数据，进行后续处理
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
    }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        Log.i(TAG,response.toString());
        Log.i(TAG,response.body().string());
    }
});
}

```

如果需要加代理的话，通过 builder 添加：  

```
// 设置代理服务器，这里以HTTP代理为例
final String proxyHost = "your_proxy_host";
final int proxyPort = your_proxy_port;
Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxyHost, proxyPort));
// 设置代理
builder.proxy(proxy);

```

_**说到代理，这里给大家推荐款 IP 代理商：**__**青果网络**_ _**！！！**_

最近笔者也在使用这家代理，主要是给的优惠足够大  哈哈哈

> 青果的国内短效代理，提供动态和短效的 IP 资源，其基于拨号 VPS 构建的高品质代理服务器，部署全国 200 + 城市与地区，IP 日流水超 600 万，更有 6 小时不限量测试时长；使用后的感觉整体 ip 稳定性、业务成功率和性价比都挺不错的。

评估下来个人感受有一下几大优势：

第一：极速响应

第二：支持短时间大量提取 IP（应对短期需要切换大量 IP 的爬虫任务）、

第三：提供代理请求统计分析（方便用于监控用量）。  

第四：支持负载均衡（平均分配请求负载，防止单个服务器过载）

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwdw9VuNibYVGvjwic2iaQFLiaricrmCX1JpzRIYAMSMEvUquN6kyR7FtOmEhzgXzyD6HEIq17fGA6tTlQ/640?wx_fmt=png&from=appmsg)

_**感兴趣老板可以点击文尾**__**阅读原文**__**查看！！！**_

_**切回正片！！！**_

**retrofit**

> Retrofit 将您的 HTTP API 转换为 Java 接口；Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装，网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责网络请求接口的封装；

准确来说，Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装。原因：网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责网络请求接口的封装。App 应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、Url 等信息，之后由 OkHttp 完成后续的请求操作。

在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit 根据用户的需求对结果进行解析。所以，网络请求的本质仍旧是 OkHttp 完成的，retrofit 只是帮使用者来进行工作简化的，比如配置网络，处理数据等工作，提高这一系列操作的复用性。

```
  Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") //设置网络请求的Url地址
                .addConverterFactory(GsonConverterFactory.create()) //设置数据解析器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
// 创建 网络请求接口 的实例
GetRequest_Interface request = retrofit.create(GetRequest_Interface.class);
//对 发送请求 进行封装
Call<Reception> call = request.getCall("");
// 异步请求
call.enqueue(new Callback<Reception>() {
    //请求成功时回调
    @Override
    public void onResponse(Call<Reception> call, Response<Reception> response) {
        //请求处理,输出结果
        response.body().show();
    }
    //请求失败时候的回调
    @Override
    public void onFailure(Call<Reception> call, Throwable throwable) {
        System.out.println("连接失败");
    }
});
  //同步请求
try {
    Response<Reception> response = call.execute();
    response.body().show();
} catch (IOException e) {
    e.printStackTrace();
}

```

03

—  

分析过程

打开 douyin-apk，找到 retrofit2  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaz7sEVVibDW3ia4ZJQMDGm5qZBX7ibTcJPbEyY376hYIpiaVWvNM3iah734g/640?wx_fmt=png&from=appmsg)

目录下封装了 client、request、response，以及一个 SsCall 接口  

SsCall 接口有个 execute 方法，然后返回值 Response 类型  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaibJyc1nb3doXI1WdzyqhWNBicJ0WntiaicicEMiciaSwRcMoyuN0xYDPyiaOOg/640?wx_fmt=png&from=appmsg)

整个流程就是一个实现 Client 接口的类调用 newSsCall 会返回一个实现 SsCall 的类，类中通过 execute 调用实现 Call 的类返回 Httpresponse

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaD36yVjU5zg6K7cxMTbscyib4WXfhEoSjHaqWxwnbmicldFn2lXAUaBBw/640?wx_fmt=png&from=appmsg)

查下这个 SsCall 的实现，进第一个查找用例  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAasiaFnfFNap5ia5iczEgvQDtMHJbQYbeUWzp6wOetRbtKd2nNjSIs9l83g/640?wx_fmt=png&from=appmsg)

这里拿到 retrofit2 封装的 Response 的 body

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaaFwYy5ZQYpghVrEeDVKzcMtkShaLdqU1WqFfoITSZ9bSK1akDCy2Bw/640?wx_fmt=png&from=appmsg)

我们 hook 下 Response

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAacaXpLg3lU9mJ3ibvPZNVw2UnFekow6ic8sYpRqxFWTvN76CzHGOdeG3w/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAanFKpjepxJAhaVYJmW9eBaJmDRP30Jmrh0s8tea5hgac0RYxzsm1xsA/640?wx_fmt=png&from=appmsg)

hook 到了  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaVruQLr3TQNv74egfLwmCkle7Km1Ys8H1hAGQqC5MA3bbWrEPpqR7Cg/640?wx_fmt=png&from=appmsg)

参数：  

str 是请求的 url  i=200 应该是状态码  list 是个 object  typedInput 也是 object

从名字上可以看出 list 是 headers 的切片，最重要的是这里的 typedInput

往上回溯找到实参位置  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAagPicAzSnLHqFviblWDPtn7CgWXlJgvuniaQJJw00Rwh6aQLbgyZtVXwoA/640?wx_fmt=png&from=appmsg)

进去

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAanx8ghgly3AZ9W6icRp9uGWe1wPPeCic7eNmW87u0rqTDbYJicV0mY5cXg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaUUacYdW6jVpkqic1PvHVrzGPahGUdKdUhgXaA9CakpEFSRdzDHQL4Ew/640?wx_fmt=png&from=appmsg)

直接 hook 这个 getBytes 即可  

因此我们上面 hook 函数改为：

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaPyxyyhbFWN8g9dicXtwU7tzMPtDBFodym6bkJ9HsjW2Sg4JwRt5lVlg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaE34k96ChjAWGskFPP8JYv4z7HzyCGaKJzKhpG0MuDU0tU35rrcVPeA/640?wx_fmt=jpeg)

我们日志文件里面以及有了字节，我们转为 str 看下  

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaaUdqawpBib2uL6KXbcpX8YpfyqdoyJiaOFdcSMTlRvtxNvHm6YwADdUw/640?wx_fmt=png&from=appmsg)

然后就拿到了文本

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAaicUbwO3bUsF4Cic6Z6dTgF60WkgXfjtXSMOFZXLibeaxzEMrTJFt00spQ/640?wx_fmt=png&from=appmsg)

在 frida 利用 JavaString 将字节转为 string  

```
let JavaString = Java.use("java.lang.String")
let _TypedByteArray = Java.cast(typedInput, TypedByteArray)
let results = JavaString.$new(_TypedByteArray.getBytes());

```

好了分析完毕！  

其实在分析过程中找到了一个 Cronet 的开关，这个函数 hook->return false，同样可以降级抓包  

hook 代码星球见，星球好久没更新要被骂了

04

—  

算法还原

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuym7c6yPA6icicqLRZSzNaWAapib8VjJUn193TDTVL12VjbpGFsRYEhVRJCNo0fxrENKBq7cN1ZnN73Q/640?wx_fmt=jpeg&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwtV3Ot2Y5VCuGU0DibxkkurkYJ2QzbN96L6ibFbBgOEM8TYpH4P8A8eQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwyVZbsGqS6q1xRoreyqHokuq1KdtUq6A4dkuPFpqVjR0loz0QElpNQ/640?wx_fmt=png)

添加好友回复：交流群

戳  “阅读原文 "  了解青果网络~