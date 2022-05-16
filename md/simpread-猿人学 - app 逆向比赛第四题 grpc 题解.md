> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NDMzMzAwNQ==&mid=2247483889&idx=1&sn=67c589b25ad9267991fa46a466b0b60e&chksm=a68829d391ffa0c51f0c9759244c73dbde660128bdbef26c486dfc8a6f09e65b1244b35c6ab5&mpshare=1&scene=1&srcid=0517MzN1HFdcNztEIdMRoIQm&sharer_sharetime=1652717632886&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.2.90460&platform=mac#rd)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

周五晚 8 点参加了猿人学的 app 逆向比赛, 和我的队友 "duoduo" 兄一起把这道题做出来了, 于是简单分享一下思路和过程。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

  

  

  

  

 目录

  

⊙一 . 什么是 grpc

⊙二. 什么是 protobuf

⊙三. 数据包协议分析

⊙四. sign 加密与 python 代码实现

⊙五. 总结

**一 . 什么是 grpc**

gRPC 是一个现代开源的高性能远程过程调用 (RPC) 框架，可以在任何环境中运行。它可以通过对负载平衡、跟踪、健康检查和身份验证的可插拔支持有效地连接数据中心内和跨数据中心的服务。它也适用于分布式计算的最后一英里，将设备、移动应用程序和浏览器连接到后端服务。

`gRPC`基于 `HTTP/2`协议传输。

客户端传递个函数进去, 然后服务端执行成功后给客户端结果。  

**二. 什么是 protobuf**

Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据序列化，很适合做数据存储或 RPC 数据交换格式。它可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

可以简单理解为，是一种跨语言、跨平台的数据传输格式。与 json 的功能类似，但是无论是性能，还是数据大小都比 json 要好很多。

protobuf 之所以可以跨语言，就是因为数据定义的格式为`.proto`格式，需要基于 protoc 编译为对应的语言。

**三. 数据包协议分析**

**初看 java 层**

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDfsmMmVHiaIHsKibo5ZsSzbiaKSqbJJlf4eBQYbn9soYjH2KHzyEzHyGkQ/640?wx_fmt=png)

主要的逻辑是: 把时间戳和每次下拉之后页数自动加 1 后的数据放入到函数中去执行, java 层被混淆的面目全非, 中间掉用了一个 sign 为 native 的方法, 于是换种思路从数据包上面开始分析。

 3.1 抓包  

拿到这道题的时候受前面几道题目的影响以为直接可以抓到包, 使用 Drony 转发到 charle 上后发现没有这道题的数据包, 初步怀疑使用了其他的协议, 然后在 pc 打开 wireshark 进行数据包的拦截, 同时开启 frida hook 第四题中的 sign。

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDYRSsz7XqYpURJyf9Ra0ttbuBW3Do6LQw04RplSicc84fLWhTPAa8zGg/640?wx_fmt=png)

  
发现 frida hook 的值和 wireshark 抓到的数据包相同, 这也就坐实了这个数据包为第四题的发包函数。  

然后我就想着能不能再找到收到的响应呢？

在 wireshark 输入过滤条件  

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDeIjYD1D6QJB1Z5ViasvcncibdzKnoYJuXIZUjxxuRAMeNHmibBcSDVicQA/640?wx_fmt=png)

然后一个一个包来看

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDfDjF7UUkvuCaZQFBOBcf5wATYvtabUVIcLYYutiaIV6QLouI0cRCEmQ/640?wx_fmt=png)

确实找到了返回数据。  

3.2 数据包分析

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDy3Sx2NwLKIZFDDJXeTQUqJRGqOB88K6icxYm14mwbE02qqYhO3me5pg/640?wx_fmt=png)

于是找来两个不同时间发送的数据包来对比下差异  

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoD5xLVIlClvYmvGjvRsWFt0VaPO4yObYml70TvFONEtOianceoLh5mljQ/640?wx_fmt=png)

也就是说结尾位置的为一些重要的参数, 我们能够伪造到然后加上前面的数据段不久可以请求服务器产生数据了？

那么如何伪造这些参数呢？  

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDjvZLLqJ8A3tN0iaQPGTPYXZibll1GgN8W7s78HibmXMCLpYp09qyutUZg/640?wx_fmt=png)

在数据的 hexdump 中只能看到 sign 的身影, 那么 page 和时间戳的身影跑哪去了, 经过了查阅资料后发现 grpc 数据的传送经过了 protobuf 的编码, 字符串在这里可以显现, 但是时间戳在 hexdump 里面并没有被显现出来。  

那么开始编写 protobuf 文件吧

```
syntax = "proto3";//指定版本为proto3,默认为proto2
message SearchRequest {
  uint32 page =1;
 uint64  time = 2;
 string sign =3;
}

```

 然后执行: protoc --proto_path=./ --python_out=./ test.proto     

```
def bytes2hex(byte_arr: bytes) -> str:
    return byte_arr.hex()
import test_pb2
# 实例化协议对象
ser = test_pb2.SearchRequest()
ser.page = 1
ser.time = 1652580021332
ser.sign ="b94a543f66e23656"
# 对数据进行序列化
data = ser.SerializeToString()
print(type(data))
print(bytes2hex(data))

```

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDQ9vEVia3DvCrKprp6dyQgcl1Nm2smNiazgxItTiaicQgwBuddzQvnicCwnQ/640?wx_fmt=png)

这个时候当事人 1 和当事人 2 都非常开心, 是不是就可以解决了？解决了就可以睡大觉了

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDnstavm4HmZlv4p2aRlzjBeq8NRrYiaTfW9zicZ9oic8W4icXickBPxQaXxQ/640?wx_fmt=png)

然后十分钟: 连接服务器 把 tcp 头部和数据拼接起来发给服务器。。。。。经过实验 失败，服务器会莫名断开连接。  

四. sign 加密与 python 代码实现

于是想了下 python 如何 grpc 的请求和服务器交互, 说干就干. 经过了一番查阅资：需要以下条件

服务器的地址和端口 (废话), 前文中我们提到 grpc 那么就有要被执行的函数, 有被执行的函数就得有接受函数执行完的结构。

于是乎 下面的代码就出来了  

```
syntax = "proto3";
package challenge;
message app4 {
 uint32 page =1;
 uint64  time = 2;
 string sign =3;
}
message HelloReply {
  repeated string message = 1;
}
service Challenge{
 rpc SayHello (app4) returns (HelloReply) {}
}

```

```
# -*- coding: utf-8 -*-
import grpc
import data_pb2,data_pb2_grpc
_HOST = '180.76.60.***'
_PORT = '9901'
def run():
     conn = grpc.insecure_channel(_HOST + ':' + _PORT)
     client = data_pb2_grpc.ChallengeStub(channel=conn)
     page_time = "1:1652586112285"
     sign = "92979f42250a942b"
     page_time = "4:1652593296974"
     sign = "f9f5c6fe5b3da911"
     page = page_time.split(":")[0]
     time = page_time.split(":")[1]
     response = client.SayHello(data_pb2.app4(page = int(page),sign = sign,time =int(time)))
     print( str(response))
if __name__ == '__main__':
    run()

```

在终端下执行：  

```
#python3 -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. data.proto

```

即可生成 grpc 和 probuf 协议的文件。  

run 一下  

![](https://mmbiz.qpic.cn/mmbiz_jpg/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDeE0u57OvfVeyrtaXtYFRvkCpmbY0Kpdlu73KELE1HMVibUQWUzFDv2w/640?wx_fmt=jpeg)

发现返回的数据结构怎么这么多位, 服务器抽风了还是我的程序抽风了, 后来 "duoduo" 兄说前面的 004 说明后面有四个值是对的。

于是协议模拟的问题告一段落。

要让整个算法跑出来算 100 页的数据, 还需要 sign 参数  

frida 代码如下  

```
        var signclass1 = Java.use("com.yuanrenxue.match2022.fragment.challenge.ChallengeFourFragment");
        signclass1.sign.implementation = function (arg1,arg2) {
            console.log("");
            console.log("page_time= \""+arg1+"\"");
              console.log("sign= \""+this.sign(arg1,arg2)+"\"");
            return this.sign(arg1,arg2);
        }

```

发现 sign ：

第一个参数为 page: 时间戳。第二个参数就是时间就是时间戳

由于比赛原因 谁先搞完就能排名高, 掏出 unidbg(也想还原来着, 那么短时间不太现实)

写 unidbg 调用  

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib2ZORWVdSL5GibQd9DAf7SoDz4E9JLlwvza3xGp6BraV9zHl1I7CF0tFYfWZf4YSMnibVfWhsViaiciaNw/640?wx_fmt=png)

主要代码如下

```
    public String callsign(String input,long j){
        List<Object> list=new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
        list.add(vm.addLocalObject(new StringObject(vm,input)));
        list.add(j);
        //Number number=module.callFunction(emulator,0xdf0,list.toArray())[0];
        Number number = module.callFunction(emulator, "Java_com_yuanrenxue_match2022_fragment_challenge_ChallengeFourFragment_sign", list.toArray());
        return vm.getObject(number.intValue()).getValue().toString();
    }

```

然后排名上升两名  end。

五. 总结

要去看更多的东西才能扩宽自己的眼界, 有些技术不是难, 而是没接触过 (hhh 接触过也不一定会), 还有 "duoduo" 兄 yyds。

**我是 BestToYou, 分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误的地方, 恳请大家联系我批评指正。**

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif)