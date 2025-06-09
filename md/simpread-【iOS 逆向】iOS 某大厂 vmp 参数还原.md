> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eZyUp_rjLy5l7RnyWJ088A)

### 前言

文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理。严正声明禁止用于商业和非法用途，否则由此产生的一切后果与作者本人无关。若有侵权，请在 vx【amuncocoL】联系我

### 说明

工具说明

```
trace:
    这里用的棕熊的iOS版本qbdi,链接:https://bbs.kanxue.com/thread-287137.htm#msg_header_h1_1
frida
charles
IDA


```

样本: 脱敏！保密！保命！![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnfnIAn2ld63b3Nfaan9yYeyIumIvPJricW64gJTVNXL3oDEAI8OHwKWw/640?wx_fmt=png&from=appmsg&watermark=1)

### 抓包

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnI5dkkozc3dwSJGsia5kGQUuhcZYRyF2YV7Y4xX1CeZK1qy8eS2dPZQA/640?wx_fmt=png&from=appmsg&watermark=1)抓包可以看到 header 里 xxx 中包含 x1,x2,x3,x4,x5,x6,x7。 x6 就是本次分析的目标。

分析
--

首先就是定位参数, 然后找到参数算法生成的位置, 都是 OC 层代码, 很简单直接跳过。![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGne2icF6yqAGhMdggjkMXIDVYKG3uOztovN20zSaTKhNv8ViatLOmZo6YA/640?wx_fmt=jpeg&from=appmsg&watermark=1)

进入核心算法触发的调用点, 固定入参发现结果一直在变, 找到内部的随机数逻辑 hook 掉。 跑一个 trace, 这个样本倒不是很大。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnx1bPmsZ4suQjFFYj64MwyYM2ZjwOC2Ma14MP3LiaiaPH3sdTy3ms5xcg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

开始进入分析 x6

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGne1wnfcRqSahJB8rmep9scNTksF4P3eHvP52HvFAS2ETCFODUhFKgibQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)

### x6:

```
"x6":"97c8cc421b9ada2d94103c174b5814b617cda51cbf148a8a1eb499609bbc9483"


```

找到 x6 生成的地方

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnhEB1Sc371muPqNCXHjIaodhcN3IQuQ9ogVXCY0Cr24yscfmaGK6djg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

可以看到 x6 的计算为: w8 ^ w9 = target

```
  w8^w9   = target
0xf1^0x66 = 0x97   
0x69^0xa1 = 0xc8
0x74^0xb8 = 0xcc
0x09^0x4b = 0x42
0x22^0x39 = 0x1b
0x1a^0x80 = 0x9a
0x78^0x55 = 0x2d
0x57^0xc3 = 0x94
0xe8^0xf8 = 0x10
0x4a^0x3c = 0x76
...
0xf8^0x7b = 0x83


```

#### W8 分析

查找 w8 这列数值计算的地方![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn7R7w0QWgJk38jN1FU5sy0nTichZ0DPeHoKPWiaQCDNVKtw8v94FIP3ibA/640?wx_fmt=jpeg&from=appmsg&watermark=1)结果计算的地方

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGntVXXiaicFghLsDVejZhBa3KicSryq7ibwuO4wK5em8KN1SUlia0iaMt8fzYw/640?wx_fmt=jpeg&from=appmsg&watermark=1)

同样是由 w8,w9 两个值 add 得到

```
add w8, w9, w8 ;W8=0x42 -> 0xf1, W9=0xaf // 0xf1
add w8, w9, w8 ;W8=0x0 -> 0x69, W9=0x69 //0x69
add w8, w9, w8 ;W8=0x88 -> 0x174, W9=0xec  //0x174 strB
add w8, w9, w8 ;W8=0x73 -> 0x109, W9=0x96
add w8, w9, w8 ;W8=0x40 -> 0x122, W9=0xe2
add w8, w9, w8 ;W8=0x73 -> 0x109, W9=0x96
add w8, w9, w8 ;W8=0x40 -> 0x122, W9=0xe2
add w8, w9, w8 ;W8=0xe -> 0x1a, W9=0xc
省略N次...


```

##### w8 即 0x42 这列分析

搜一下![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnotUeibiaBibw02icue4NJ3liatWOj8nBibzrvFO1rUsuPV7v6m8Uxykt47zQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)

```
Call addr: 0x20a19b8d0 [libsystem_platform.dylib!_platform_memmove] (dst=0x13bd375b0, src=0x13bd36ed8, n=32)
源内存内容：
 (dst=0x13bd375b0, src=0x13bd36ed8, n=32)0000: 42 00 88 73 40 0e 03 08 30 6f c1 b1 07 54 11 6d   B..s@...0o...T.m
 (dst=0x13bd375b0, src=0x13bd36ed8, n=32)0010: 8b dc f9 04 c7 7d 26 75 76 84 2e fc 3f a4 ce 94   .....}&uv...?...


```

继续找这块数据生成的部分

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnT3FvsBiaMFY1EdfIZfNbnTVeS6oPyvWIQ2QTFTMJ4t5Sl7mpLpzibeeQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)

看下 0x42008873 生成的地方

```
0x1003e8b40 0xc0b40  add w14, w15, w14 ;W14=0xf485e8dc -> 0x42008873, W15=0x4d7a9f97


```

继续看 W15=0x4d7a9f97 的生成

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnytibKXITBzW28icB55roxExeH5ib3AMk5seAtTMD7ialGt9oSMaYgLsVjg/640?wx_fmt=jpeg&from=appmsg)

```
0x1003e8b40 0xc0b40  add w14, w15, w14 ;W14=0xe370b930 -> 0x4d7a9f97, W15=0x6a09e667 


```

熟悉的朋友想必已经发现 0x6a09e667 是 sha256 的 h0![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGntUyDFfCgxYc5ibHibFNdQdxYhnSe0unM97mP4olZsQvcYyvCnYLVIxYA/640?wx_fmt=jpeg&from=appmsg&watermark=1) 回到上头 0x42008873 的生成中, 看 W14=0xf485e8dc

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnias9O3UuBANzJBICgrWicoSRzenibRTPpOjEpPZJKZXJhMwhry155YsLA/640?wx_fmt=jpeg&from=appmsg)

```
W14=0xf485e8dc = 0xaedaced5+ 0x45ab1a07
0x1003e8aec 0xc0aec  add w13, w3, w15 ;W13=0x23b32947 -> 0xaedaced5, W3=0xa9237f7f, W15=0x5b74f56 
0x1003e8af0 0xc0af0  add w13, w13, w14 ;W13=0xaedaced5 -> 0xf485e8dc, W14=0x45ab1a07    //  W13+W14 = (w3+w15)+w14


```

继续 W13=0xaedaced5 的生成中 w3=0xa9237f7f 的生成

```
0x1003e8aec 0xc0aec  add w13, w3, w15 ;W13=0x23b32947 -> 0xaedaced5, W3=0xa9237f7f, W15=0x5b74f56 


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnZiapgJbBQMalWus0YdQAKv1xlLegu1JCrrweRllMJbAiafN3j2IUdiaaw/640?wx_fmt=jpeg&from=appmsg&watermark=1)那么 w3 分支对应 sha256 中 S0,s0 组成的 temp1 会和 temp2 有 add,temp2 中有 W[i] 参与, 目的通过 w[i] 找明文。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn18Dll6dKp3sT51g3hVHVicia6g1ibWY4f9fN7YyHQSS2defLLl4qrCucw/640?wx_fmt=jpeg&from=appmsg&watermark=1)找 w[i], 发现 0x5b74f56 中有明文操作经过分析:

```
0x1003e8aa4 0xc0aa4  add w16, w16, w17 ;W16=0x3102044 -> 0x9a4d5032, W17=0x973d2fee
0x1003e8aa8 0xc0aa8  add w15, w16, w15 ;W15=0xedfe1951 -> 0x884b6983, W16=0x9a4d5032
0x1003e8aac 0xc0aac  add w15, w15, w3 ;W15=0x884b6983 -> 0x504b6e0b, W3=0xc8000488
0x1003e8ab0 0xc0ab0  add w15, w15, w5 ;W15=0x504b6e0b -> 0x16bce6fd, W5=0xc67178f2 // w5 = K[i] = 0xc67178f2为K表的常量最后一轮
0x1003e8ab4 0xc0ab4  add w15, w15, w4 ;W15=0x16bce6fd -> 0x5b74f56, W4=0xeefa6859  //  w16+w17+w15+w3+w5(即K[i])+w4[即W[i]]


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnKk267OG4WJw8lnKAqks0bmFXK3IUWoSSV3oXZtyrGUOau0L3a0oDfw/640?wx_fmt=jpeg&from=appmsg&watermark=1)w[64] = 0xeefa6859，找到 0xeefa6859 的生成地方, 搜下:![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnM4rJjanxWTscJxAl4VHANIicqZve1fSsJIJ0siaiaxck25TP38G7LuEoA/640?wx_fmt=jpeg&from=appmsg&watermark=1) 找到 W[0]

```
0x1003e8aa0 0xc0aa0  ldr w4, [x6, x4] ;W4=0x0 -> 0x5f15b545, X6=0x13bd36d58     // W[0]


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn3lexnBeBHqka12T3XJGmjfwTlDnDYVwLo3iaV4d3djrWjCOLyFnOjibw/640?wx_fmt=jpeg&from=appmsg&watermark=1)

找 0x5f15b545 即 w[0] 的生成, 中途看到 W[0] 参与的计算![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnAKvAGAUtmkMTL4FRT4Y1mjibzaPhvEwOst01nPnficnuCqNubJvJj4Ow/640?wx_fmt=jpeg&from=appmsg&watermark=1)对应

```
    for i from 16 to 63
        s0 := (w[i-15] rightrotate  7) xor (w[i-15] rightrotate 18) xor (w[i-15] rightshift  3)
        s1 := (w[i-2] rightrotate 17) xor (w[i-2] rightrotate 19) xor (w[i-2] rightshift 10)
        w[i] := w[i-16] + s0 + w[i-7] + s1


```

继续找 w[0] 的生成, 找到![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnedicKkDt5p76yrbaevjxytg84Aq37uicU8wGwtEjjKdqibZ6uDzPtzEqg/640?wx_fmt=jpeg&from=appmsg&watermark=1)搜下 0xc097c, 明文填充 0x80, 拼接 0 拼接长度, 然后 bswap![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnEq25aicu6yroiacwchyPpyia638w4I3w1xm5GRwPNy4DibcXDx2iavUccXw/640?wx_fmt=jpeg&from=appmsg&watermark=1) 继续定位 0x5f15b545 的来源

```
0x1003e8b40 0xc0b40  add w14, w15, w14 ;W14=0xaf9a9a47 -> 0x5f15b545, W15=0xaf7b1afe // 得到 0x5f15b545 新一轮sha256


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn1y7QRkFNWfuKGcVaibEG596Jjbic2h4WicdJBC5lpPvKeDYnibr4G7N5ew/640?wx_fmt=jpeg&from=appmsg)发现又是新一次轮 sha256, 这里标记下, 并且汇编地址 0xc0b40 上面有看到。 w14-> 0xaf9a9a47 的生成

```
0x1003e8aec 0xc0aec  add w13, w3, w15 ;W13=0x65fc8355 -> 0x2fd71d2c, W3=0x20561d14, W15=0xf810018
0x1003e8af0 0xc0af0  add w13, w13, w14 ;W13=0x2fd71d2c -> 0xaf9a9a47, W14=0x7fc37d1b
0x1003e8af4 0xc0af4  stp w13, w16, [sp, #8] ;W13=0xaf9a9a47, W16=0x37c43fab, SP=0x13bd36d30


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnRBJbZKcnMibbJVWselWsJJv4QciaS8JnCOL19u8tc1It3STUCnF5o1sw/640?wx_fmt=jpeg&from=appmsg&watermark=1)

```
0x1003e8aec 0xc0aec  add w13, w3, w15 ;W13=0x65fc8355 -> 0x2fd71d2c, W3=0x20561d14, W15=0xf810018


```

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnUEjicfQOFTH7h8ZGa3ZqicZClmnCckiafNPjs8qQFdiaVt7oVNCicDadfsQ/640?wx_fmt=jpeg&from=appmsg)找 0xf810018, 之前图里提过 w4=w[i]![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnyIb9NafJQfbz89OQCyYouibTeIgHmiacklUzzX8QKa44jfvrkT9bCMnQ/640?wx_fmt=jpeg&from=appmsg&watermark=1) 找 0x633c8d9f 即 w[i]

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn3MqJS4zaXbr6XNLO1ombWNyM2w6icuVZMzySr5ZqaDI4pZDn7ywlgYQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)找到 w[0]![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnN2GP35niaeA0pcpnMeHoCOrPBytJLMIa1EtCTIscnr7JYucBXD0sXMw/640?wx_fmt=jpeg&from=appmsg&watermark=1) 找到填充 bwap 的地方![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnQguYk4nAEtxMjCFpKzH5VIE6YzID3ddr7Dr1iaxh7142ickXpyUadfKA/640?wx_fmt=jpeg&from=appmsg&watermark=1)看到这里明文为转了大端序:

```
38 d7 48 09 f8 1e 1f 55 73 24 fa 38 cf 68 05 b7


```

长度为 0x280 对不上，结合上面的 0x5c,2 次 sha56, 诸多特征当然还有我后续的分析, 确定这是 HMAC-256。 验证下没问题：![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnEamo9gAc1BiczFuLStbsM1mSKGCQiaIfkxia8GNfTsNk7uuSyAxibrBz1w/640?wx_fmt=jpeg&from=appmsg&watermark=1)

Key 的还原略过, 找到后异或就行了。 继续分析数据明文:

```
38 d7 48 09 f8 1e 1f 55 73 24 fa 38 cf 68 05 b7


```

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn8bfweOgZAyE1VvNOzCyroQ5zicD3WMUibiczT4ElktLZMqZXZRtliaIPdg/640?wx_fmt=png&from=appmsg&watermark=1)

```
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x1c -> 0x38, W9=0x24
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xfe -> 0xd7, W9=0x29
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xe8 -> 0x48, W9=0xa0
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x20 -> 0x9, W9=0x29
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xfc -> 0xf8, W9=0x4
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x5 -> 0x1e, W9=0x1b
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xa7 -> 0x1f, W9=0xb8
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xa9 -> 0x55, W9=0xfc
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x8c -> 0x73, W9=0xff
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xdf -> 0x24, W9=0xfb
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x75 -> 0xfa, W9=0x8f
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xba -> 0x38, W9=0x82
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x97 -> 0xcf, W9=0x58
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x77 -> 0x68, W9=0x1f
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0x91 -> 0x5, W9=0x94
0x1003fd29c 0xd529c  eor w8, w9, w8 ;W8=0xd0 -> 0xb7, W9=0x67



```

###### w9=0x24 这列数据

找一下和文章最后 w9 分析一致, v1 通过 a45678 解码, v1 是固定的，a45678 也是固定的, 结果解出来也是固定的, 直接拿到结果保存使用就行。

###### W8=0x1c 这列数据:

![](https://mmbiz.qpic.cn/mmbiz_png/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnZuicahyDShicWTGPkkkmUnrmUYgeAFcA6gupicVE6nqyPbMDzPTVmQKXw/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGndhXosEOxtOWLOeRDXUFUSJdcPuiaRD8zllOLVQW9CX1Q7eXxGuZXpzA/640?wx_fmt=jpeg&from=appmsg&watermark=1)

```
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x24 -> 0x11c, W9=0xf8 // w8 = w8+w9(ldr)
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x68 -> 0xfe, W9=0x96
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x96 -> 0xe8, W9=0x52
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x64 -> 0x120, W9=0xbc
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x39 -> 0xfc, W9=0xc3
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x3c -> 0x105, W9=0xc9
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x29 -> 0xa7, W9=0x7e
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x83 -> 0xa9, W9=0x26
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0xf7 -> 0x18c, W9=0x95
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x3f -> 0xdf, W9=0xa0
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0x70 -> 0x75, W9=0x5
0x1003fcea4 0xd4ea4  add w8, w9, w8 ;W8=0xd9 -> 0x1ba, W9=0xe1
省略...


```

###### 找 24 68 96 64

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnbdZd47cJqnMveE7GIJGvCV9VOSSEaXW5HicNap3TrGNt8vgddhibwUNQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)看截图里 0x2AD7D2BB,0xeb86d391 还有运算这 MD5 嘛![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnthhDyTw3E4OhAdF1oeArXhufMjkYFXMvrQuEaQgmXB8mTiaPlzhHFfw/640?wx_fmt=jpeg&from=appmsg&watermark=1)找下入参明文, 由 x1, x2, x3, x4, x5, x7 组成。![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGn0sH8fadEQDrCpficUNmH1e55JeSSLLTOZ3Wej0eTVlWTzJEU5uZNGcw/640?wx_fmt=jpeg&from=appmsg&watermark=1)入参拿到后验证下, 标准 md5 没问题![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnmE9ULiaGVfsO9NJm1DMGlkmtEHQgJoSOSoB6VRurBNAr5DfUxI8qORg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

###### W9=0xf8 这列

```
0xf8 0x96 0x52 0xbc 0xc3 0xc9 0x7e 0x26
保密...


```

找一下和文章最后 w9 分析一致, v1 通过 a45678 解码, v1 是固定的，a45678 也是固定的, 结果解出来也是固定的, 直接拿到结果保存使用就行。

##### w9 即 0xaf 这列分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnQsmdibsoOricb2ITGwCAYPdG1M4KibX974m4Fx55cxjek977WQvY0LuEg/640?wx_fmt=jpeg&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnH3ZicI7lJ2StAx4zjuAYN8Fu9D3asKGdbiamC4qVibSDpgRrdfqvicwZ9g/640?wx_fmt=jpeg&from=appmsg&watermark=1)省略 N 次 与下面 W9 分析那里逻辑一致, 解密出来结果的不同部分而已, 直接保存用就行。

#### W9 分析

查找到 w9 的值, 继续找生成![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnUhEZnrrdKkbicw7ibtIUN5XJW43p7Hwkhr7W8ZCnsudp4hz39hrPwOgg/640?wx_fmt=jpeg&from=appmsg&watermark=1)继续查找找到生成的地方

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnxx2sJPapowQUMamxTYiacViaq0ZtibMJmAman01X0sdK31A51g8tUWPVw/640?wx_fmt=jpeg&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnlxYB51qYjbO32ZsEwJc50Y8CbaESKsNlhjWrMUhGAibbnUZnLIrcyrA/640?wx_fmt=jpeg&from=appmsg&watermark=1)省略 N 次...

按这个逻辑对应 ida 伪代码

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnprHZ66nlox1dAHPll7DQltclPticCLkerDo7OGxH8Mic9smickKpturHg/640?wx_fmt=jpeg&from=appmsg&watermark=1)这块和上边一样也是对 v1 通过 a45678 解码, v1 是固定的 a45678 也是固定的。结果解出来也是固定的, 直接拿到结果保存使用就行。

最后
--

本次是 iOS 基于 trace 法还原 vmp 的小尝试, vmp 可借鉴材料的不多，自己也算摸索出来了一点皮毛。 文中如有错误，欢迎大家斧正，欢迎交流。 后面有时间会接着写剩下的参数还原~![](https://mmbiz.qpic.cn/mmbiz_gif/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnCpibichSrAyoHLas7XSGjFniaZibdTfDWOhCdsg7KSmSDY3pw4xJAAzE9g/640?wx_fmt=gif&from=appmsg) 欢迎大家关注，你们的关注是我自律的最大动力~

![](https://mmbiz.qpic.cn/mmbiz_jpg/wm415EXtN8qbCsyKbNK6nBDCHkFhibNGnot5QGsfrSrvc9pnE36JSvKEkeHRZU6f1XiajIqtx5ft5pLLfWWCtPnw/640?wx_fmt=jpeg&from=appmsg&watermark=1)