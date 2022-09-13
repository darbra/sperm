> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VhyG4diX1wNUR4xdtCK8uQ)

  

     大家好，我是 TheWeiJun。前期发布的两篇某大厂的文章因为收到律师函，作者已经对文章进行删除。今天我们分享一个 protobuf 协议逆向分析，教你如何在无结构的数据中抽取并定义 proto 文件。各位读者在阅读的同时不要忘记点赞 + 关注哦⛽️

特别声明：本公众号文章只作为学术研究，不用于其它用途；如有侵权联系删除。

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A6N3l9obtATicYg0vSreUhmEsiciaibJfHjG8MEHndnoJycRKpgeeK3LkxKu6qlz7oyC2Gelexa4W4Bfw/640?wx_fmt=gif)

  

立即加星标

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6N3l9obtATicYg0vSreUhmEG7mwRfEUOichNPSVJkJl4IDcjeqvJyO5GGxL9CaYcRQqg13wjyXVzicQ/640?wx_fmt=png)

每天看好文

 目录

  

一、什么是 protobuf？

二、protobuf 堆栈结构

三、proto 定义及编译

四、完整代码实现

五、心得分享及总结

趣味模块

      小明是一名爬虫开发工程师，自从上次小明解决了字体反爬、websocket 协议、B 站 protobuf 协议后，小明一直所向披靡，过五关斩六将，在一个多月的时间里一直没有遇到过有难度的问题。但是今天，小红遇到了某某 App 端的 protobuf 协议加密，不再是 js 端一样去找定义的类型 id 了。那么今天我们去分析下小红同学遇到的新问题吧，如何通过无结构定义 proto 文件并还原某某 App 的 protobuf 协议！

一、什么是 protobuf 协议？

**前言：**Protobuf (Protocol Buffers) 是谷歌开发的一款无关平台，无关语言，可扩展，轻量级高效的序列化结构的数据格式，用于将自定义数据结构序列化成字节流，和将字节流反序列化为数据结构。所以很适合做数据存储和为不同语言，不同应用之间互相通信的数据交换格式，只要实现相同的协议格式，即后缀为 proto 文件被编译成不同的语言版本，加入各自的项目中，这样不同的语言可以解析其它语言通过 Protobuf 序列化的数据。目前官方提供 c++，java，go 等语言支持。

二、protobuf 堆栈输出  

1、首先从 charles 应用中保存我们抓到的某某 App 数据包，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6OSdhmXKrTicAthCdf9HeITbibFJUicYUibrSeLDso0EJ0gAu6w0uqYEaSw/640?wx_fmt=png)

2、使用 Postman 访问协议接口，将 response 响应内容保存为. bin 格式文件，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6yGBUI9jrwtibVwpXgRMXaPv8qB4PtQQVIqXlfPNx61OsrDraFXg63sA/640?wx_fmt=png)

3、使用 **protobuf_inspector** python 第三方包打印 protobuf 文件堆栈信息，截图如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6Fe8YibMxlZz0S8HWKDFD4GF13BfyuW1wYyxbrFyjicSukRdLA9gyLtdQ/640?wx_fmt=png)

**总结：**打印 protobuf 数据堆栈信息后，接下来我们只需要根据打印的堆栈信息定义 proto id 类型文件并编译为 python 可执行文件即可完成对某某 App protobuf 协议还原。

三、proto 文件定义及编译

1、通过分析打印的堆栈信息，确定我们本次要提取的文字内容，截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6jNojqJsBIKNDwuwJy6ln4XYvbuwP2M9jERILHhgR9iaysLAMiaYYlF7g/640?wx_fmt=png)

2、确定我们要提取的文字内容后，定义 proto 文件格式，定义后的结构体如下所示：

```
syntax = 'proto3';
message DanMo{
  message Message{
    int32 id = 1;
    int32 time = 8;
    string content = 7;
  }
  repeated Message message = 1;
}

```

3、执行如下命令，编译为 python protobuf 可执行文件：

```
protoc  --python_out=. *.prot

```

4、运行命令后，生成 python protobuf py 文件，截图如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6wx5qIYSIbEmE3Q4quMtMcs2lZBJgchSlAZlONO4sT0xl2ibwykqiaKMw/640?wx_fmt=png)

**总结：**走到这里 protobuf 协议就完全还原了，接下来让我们进入完整代码实现环节吧。

四、完整代码实现

1、整个项目 Python 完整代码实现如下：

```
from google.protobuf.json_format import MessageToDict
from danmo_pb2 import DanMo
with open('test.bin', 'rb') as f:
    _info = DanMo()
    _info.ParseFromString(f.read())
    _data = MessageToDict(_info, preserving_proto_field_name=True)
    items = _data.get("message", [])
    for item in items:
        _id = item.get("id")
        _time = item.get("time")
        content = item.get("content")
        print(_time, content, _id)

```

2、代码编写完成后，我们运行代码，代码输出截图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7tj3p18Qv2gHB2jkQHnTT6NzuJzaaUuQvwmIdXvAaUQHiceBg7Sextib8nGZibPRgKa0d1ibwib3DG19Q/640?wx_fmt=png)

五、心得总结分享

      通过本次案例分享，我发现有些技术不是难，而是没有找到好的方案去解决；有的问题需要思考 3 个小时甚至更久，但是解决只需要 10 分钟。今天分享到这里就结束了，欢迎大家关注下期文章，我们不见不散⛽️

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A43BLL5j8ShhkUSRLT9ayEsmde5yxQmlKq8qZgsltoYaFpliaXKlmJUtNr6uZHibrn2xgIsQJUXu2nA/640?wx_fmt=png)

关注我们获得更多精彩内容

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7k7QiaRDzFktiaSbkClw0pXa6NV1Q9ge9a6D5nxGOojicqVQUQqQK0NOHg/640?wx_fmt=png)

_**END**_

![](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7zpB5TQPCHMeJVyX5BicWRibtHzfCIJvrJRAiaLC9akyJxXrfKVMnUS6rw/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691Wt6FFglTFeeibsPZT5F1vAiafn06J37WwvPkkGVX2B14Qh3gpPmic5Dpw/640?wx_fmt=gif)

    我是 **TheWeiJun**，有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友。

    分享日常学习中关于爬虫、逆向和分析的一些思路，文中若有错误的地方，欢迎大家多多交流指正☀️

![](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691PicricStHwRzqmIO1cPGTPsCk5SmfU2AZQLL2B6KSpxaHGguqZjXnjiaw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLwXGFXXOX978IvJSokfVzoN9L2EUB8bUAC0QqT1WJmckZfBfZJl3FFA/640?wx_fmt=gif)

**点分享**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLAWgUzHv0vx1STQhKykWTpicN12F4UdUeNXS1WsXYicqeBzmEUQbB3dAg/640?wx_fmt=gif)

**点收藏**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLJzbzyLgP8z7NIwqluryicesaw2PBoEPNvQ8K0jpyqn7AlA3vHGq1n3Q/640?wx_fmt=gif)

**点点赞**

![](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A7lKEcIibT9GVduic8eSDCiaFLXdRibvzbwpGJb8wcTtFaUYTx1WXpUiaaD9TYy6Rk7jYhSwAL7c7BsSbA/640?wx_fmt=gif)

**点在看**