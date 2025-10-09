> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zS7LOH-F0lswZW3MUiTgCA)

声明
--

**本文仅供学习交流，如有侵权请联系删除！**

分析
--

首先要知道携程走的并不是`https`协议，而是小众的`SOTP`协议 (**因为类名里有并且堆栈也有打印**)。

`SOTP`是一种安全的覆盖层传输协议，旨在提供增强的安全性和功能，尽管`SOTP`不是一个广泛标准化的协议。

携程与小红书使用的都是`libmsao`进行`frida`检测，这次开源的魔改`server`可以直接过`libmsao`，不再需要脚本辅助，链接会放到评论区里。

`00000003`是日志上报，链接追踪方面，不过搞非`https`协议的办法是通用的，通过`hook`系统`Socket`的`socketWrite0`可以得到发包前的堆栈，根据堆栈进行逆向分析。

```
0x00000000 0000000300000A 50315C 43 FC 5373 C3 AB .......P1\C.Ss..
0x00000010 D0 29 AC 767F A1 3151 D4 7673 FB 81 D0 4C C9
java.lang.Throwable
        at java.net.SocketOutputStream.socketWrite0(Native Method)
        at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:117)
        at java.net.SocketOutputStream.write(SocketOutputStream.java:149)
        at e4.i.n(SourceFile:42)
        at e4.i.l(SourceFile:47)
        at e4.i.j(SourceFile:40)
        at e4.d.r(SourceFile:91)
        at e4.d.q(SourceFile:120)
        at e4.h.run(SourceFile:142)
        at java.lang.Thread.run(Thread.java:919)


```

JAVA 层
------

进入`e4.i.n`方法，传入的`p1`就是加密后发送的数据，继续向前追溯，在`e4.d.q`方法内可以得知该数据是如何生成的，它自己封装了一个`mobData`类，对这个对象进行了加密。

```
private byte[] n(Socket p0,byte[] p1){
    outputStream.write(p1);
}
//e4.d.q
byte[] uobyteArray = ((mobData = obj.k()) != null)? uod.d(mobData): null;
if (uobyteArray != null) {
    uobyteArray = uod.m(uobyteArray);
}


```

这里直接查看`mobData`与`uod.d,uod.m`系列方法，首先进行了`encode`编码，然后进行了`Gzip`压缩，最后进行了`AES/ECB/PKCS7`加密，既然知道了如何加密那就去解密。

```
//uod.d c
uobyteArray = a.a(h.a(p0.encode()));
//h.a
GZIPOutputStream gZIPOutputSt = new GZIPOutputStream(uByteArrayOu);
gZIPOutputSt.write(p0);
//a.a
a.b().doFinal(p0);
//a.b
Cipher instance = Cipher.getInstance("AES/ECB/PKCS7Padding");
instance.init(1, new SecretKeySpec(a.c(), "AES"));


```

`AES`的密钥的生成逻辑就在`a.c()`中，直接复制代码到`idea`中跑一下就行，`GZIP`与`AES`均没有进行魔改，直接套库调用就行。

```
private staticbyte[] c(){
    int b;
    // 第一个硬编码的8字节数组
    byte[] uobyteArray = newbyte[8]{'f', 0xef, 'p', '%', 0xbf, 0x90, 0xd5, 0x09};
    // 第二个硬编码的8字节数组  
    byte[] uobyteArray1 = newbyte[8]{0xac, 0xcd, '*', 0x0e, 0x0c, 0x95, 0xb7, 'H'};
    // 最终生成的16字节密钥数组
    byte[] uobyteArray2 = newbyte[16];
    // 组合两个8字节数组为一个16字节数组
    for (int i = 0; i < 16; i = i + 1) {
        if (i < 8) {
            // 前8字节取自第一个数组
            b = uobyteArray[i];
        } else {
            // 后8字节取自第二个数组
            b = i - 8;  // 计算第二个数组的索引
            b = uobyteArray1[b];
        }
        uobyteArray2[i] = (byte) b;
    }
    return uobyteArray2;
}


```

重点在于`p0.encode`方法，本身的`mobData`就是一个`protobuf`，追踪一直跟入后会发现是一个`ProtoAdapter_MobData`的类，这实际上就是`mobData`的`protobuf`数据结构。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqr7qDhBpWjJqkk0P1HG3ib6XRzkLv8hCQJiaZXzexeNetYFM1XZApZG0lg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

```
//mobData
message MobData {
    Header header = 1;           // 消息头
    bytes data = 2;             // 数据内容
    repeated Payload payloads = 3; // 负载列表
}


```

然后就是对`header`与`payloads`的`protobuf`结构进行追踪，进入`MobData`内就可以看到对应的数据结构，`header`的结构是非常简单的。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqruicbBY6l3ibRbNw8RXvhG4qrK1dv8tMA8kwlibgIR9Ge4jia3L6L8PLnIw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)

`payload`的结构有点复杂，他是在`e4.d`的`n`方法下进行处理的，跟入这个`b`方法，会发现一系列字段，这些就是序列化结构字段，其中每一个`case`都是一个`message`类型，比如里面的`UserTrace`，它内部也是有`encde`方法，跟`header`一样，基本都是这么去找，然后需要具体扣出来写成`.proto`文件。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqreKxUSyFTYqN1Pw2hGGXIkcyyqClnV6K7fKvLUHWYibynZewZX80mKZQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqrkyric55yqk5I8gicnLNd0Xe5WdYOkFBsHWmucZiccK9BGpacPjYtPVCVg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3)

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqrIIQj0nGZLsgO14iaAoqdQSSnCRSPpgxSSwEgnvLXlJgYvtb7F4RHYlA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)

最后的这个`uod.m`方法就是简单的添加了一个标识头，然后把数据的长度添加进去，这样做方便服务器去根据开始的前 4 字节判断请求类型，选择对应的方法解密。

```
 uobyteArray = uod.m(uobyteArray);
 //uod.m
byte[] uobyteArray = new byte[4];
byte[] uobyteArray1 = new byte[4];
q.c(3, uobyteArray, 0);//[00,00,00,03]
q.c(p0.length, uobyteArray1, 0);//写入[00,00,00,数据长度]
return q.a(q.a(uobyteArray, uobyteArray1), p0);//将其拼接为一个完整数组 [00,00,00,03,00,00,00,长度,数据]
//q.c是大端序写入
p1[p2] = (byte)((p0 >> 24) & 0x00ff);
p1[(p2 + 1)] = (byte)((p0 >> 16) & 0x00ff);
p1[(p2 + 2)] = (byte)((p0 >> 8) & 0x00ff);
p1[(p2 + 3)] = (byte)(p0 & 0x00ff);


```

总结
--

请求体解密就去`hook socketRead0`，具体跟踪流程跟这个类似，跟着堆栈找，其中走`ctrip.business.sotp.SOTPConnection.j0`这个类的才是真正发送请求的，具体也不细说了，如下图，该位置`bArr`就是明文字节，在内部进行加密，直接把`bArr`转换后就... 懂得都懂。

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqrpmlQozAa3fxMvic3jhF6dKonynCRngFIJKexbvHiahTWPABd5DA5ib48g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5)

![](https://mmbiz.qpic.cn/mmbiz_png/iaiafKP2ECpXpQG6FEoonIf4yIFyicqiaNqrO1ry3aZZGa6yTqMvQhQASp3hgrrapaYNZcSJqHCos74J2dUrDGVW7g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)