> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fZAKrRczaoIDribmx-boSw)

![](https://mmbiz.qpic.cn/mmbiz_png/gY5PqCV2kYZJ4sKUniaZPr7YRoHTLnbM8ZCNohcGM2hjy5tMJyRd9RicbR9HrMkoJiaGrBrV7JjW09zqHtCHI8q8g/640?&wx_fmt=png)

xa，x 什么 a，就 xa

![](https://mmbiz.qpic.cn/mmbiz_png/ib93efMPP0actGbYAbS23TrIXMKA4Hfb3wAjuNLrpnMP1iarC8oTugBZFgEibxVv0uVSvt4F6wwMxoDdCWddEL6sw/640?&wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zJkrRpjYPhNfP5zichlATibarrJQVsf9t5nNRBIicAoVVXVx4sZ6RgF24KTG4ibkcFX4vyLITzJosTaCZYPD0iaDicrQ/640?&wx_fmt=png)

  

从后往前

1、xa 结果像是 base64 可以先 base64 解出来 去 trace 中搜索 a1 d8 40 6a, 不过最好的办法是，从 base64 末尾搜索

像 xa 算法 他会在头部拼接

```
00000000: A1 D8 40 6A 26 BC 8D 6D  6F 6B EF 6F DA 50 36 AD  ..@j&..mok.o.P6.
00000010: 23 E3 41 5D 51 B7 31 04  32 8C 76 84 C9 4F 27 CE  #.A]Q.1.2.v..O'.

```

```
可以搜到
0000: 40 6a 26 bc 8d 6d 6f 6b ef 6f da 50 36 ad 23 e3  @j&..mok.o.P6.#.
0010: 41 5d 51 b7 31 04 32 8c 76 84 c9 4f 27 ce e7 13  A]Q.1.2.v..O'...
0020: 23 ec b7 8f 68 26 01 8e bf 2c 98 71 45 79 3b bb  #...h&...,.qEy;.
0030: df 59 97 3c 00 61 3b 7c 50 02 da c3 b6 52 b3 be  .Y.<.a;|P....R..
0040: 1e 80 0e 57 43 58 57 3f 8a ff a5 a8 d8 c5 f2 fe  ...WCXW?........
0050: 1b 79 96 da 70 e6 87 a4 73 cc a8 25 93 9b 34 24  .y..p...s..%..4$
0060: 2d 1e ab 7f 73 1d 83 ab c1 05 a6 7f 13 66 bc 82  -...s........f..
0070: 72 68 a5 dd fb 06 ce ea 28 0d b3 9c 17 61 53 5c  rh......(....aS\
0080: 1e 79 a0 b1 fc 8e fa db f3 44 69 bf 63 b6 f7 ed  .y.......Di.c...
0090: 61 c8 55 92 c0 7a e9 4c b2 ac 71 dd 58 e4 d5 d7  a.U..z.L..q.X...
00A0: a2 d3 00 c8 d1 c9 4f 0f dd db b1 79 ac 37 05 e4  ......O....y.7..
00B0: 5c 4a 61 04 c4 09 f3 58 be 92 1e 2f 0d df fb c0  \Ja....X.../....
00C0: bd 42 84 2d 5a 5f 65 7a ec 0b 1d d9 85 3b 54 21  .B.-Z_ez.....;T!
00D0: 71 a1 81 14 cd ca 9d 90 ff 9c d1 40 98 c2 f2 1b  q..........@....
00E0: 8d ee de 84 df 2d fb d1 b5 aa 3f 2b 76 0c 02 df  .....-....?+v...
00F0: 6b ca 4b dc b5 e8 96 af 90 a8 ef 1f 0b f9 35 36  k.K...........56
0100: 3f 58 36 66 39 e3 62 db a3 c5 14 6e c2 38 7a 76  ?X6f9.b....n.8zv
0110: b5 5a 5a d9 f9 ce 59 eb ae 0b 4d 3f 84 32 7a ed  .ZZ...Y...M?.2z.
0120: 55 b4 b8 4f 63 02 97 e4 4c 76 37 a1 a6 93 6d 3e  U..Oc...Lv7...m>
0130: d4 ea 66 e8 30 62 e2 11 5a 05 b2 9f 9d 6c 55 da  ..f.0b..Z....lU.
0140: 07 5c 08 c1 2f 6f 4e 9f f5 c6 32 50 5b 8e 7f cc  .\../oN...2P[...
0150: ea 6e ff ba b5 27 3e 2e 1d c0 1f e3 b5 30 eb 8f  .n...'>......0..
0160: 20 39 59 c2 4b 20 5b 22 94 44 0d 6e 4e e0 e5 d3   9Y.K [".D.nN...
0170: 17 15 19 17 52 75 de f1 bb 54 d1 a2 69 a5 37 bf  ....Ru...T..i.7.
0180: a5 5d dc 1d f0 3b 5c 65 65 46 a3 87 43 18 33 4c  .]...;\eeF..C.3L

```

根据 trace 写入地址 call libc.so!memmove 7ab3546e90 <- 7ab35576d0 size:400 搜索 7ab35576d0

会发现写入位置，分析伪代码可知是 base64 base64 buffer 头部添加 rand，很好分析，可以尝试寻找

79170F8638 [libmetasec_ov.so!D0638] str q0, [x19], #16            ;  r[q0=6b6f6d8dbc266a40e323ad3650da6fef x19=7ab35576d0] w[x19=7ab35576e0]

寻找下方 hexdump

0000: 40 6a 26 bc 8d 6d 6f 6b ef 6f da 50 36 ad 23 e3  @j&..mok.o.P6.#.

```
Line 4541528  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=69 x20=7ab35576d0 x8=176]"
Line 4541536  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=f4 x20=7ab35576d0 x8=177]"
Line 4541544  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=f4 x20=7ab35576d0 x8=178]"
Line 4541552  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=ff x20=7ab35576d0 x8=179]"
Line 4541560  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=f8 x20=7ab35576d0 x8=17a]"
Line 4541568  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=47 x20=7ab35576d0 x8=17b]"
Line 4541576  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=53 x20=7ab35576d0 x8=17c]"
Line 4541584  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=ff x20=7ab35576d0 x8=17d]"
Line 4541592  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=f8 x20=7ab35576d0 x8=17e]"
Line 4541600  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=47 x20=7ab35576d0 x8=17f]"
Line 4541608  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=53 x20=7ab35576d0 x8=180]"
Line 4541616  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=f7 x20=7ab35576d0 x8=181]"
Line 4541624  Ah  "7917100CEC [libmetasec_ov.so!D8CEC]   strb  w9, [x20, x8]            ;  r[w9=5c x20=7ab35576d0 x8=182]"
Line 4541652  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=183]"
Line 4541658  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=184]"
Line 4541664  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=185]"
Line 4541670  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=186]"
Line 4541676  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=187]"
Line 4541682  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=188]"
Line 4541688  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=189]"
Line 4541694  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18a]"
Line 4541700  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18b]"
Line 4541706  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18c]"
Line 4541712  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18d]"
Line 4541718  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18e]"
Line 4541724  Ah  "79170CF42C [libmetasec_ov.so!A742C]   strb  w9, [x23, x8]            ;  r[w9=d x23=7ab35576d0 x8=18f]"

```

填充 0xd? 根据 trace 伪代码 可以寻找到的

看此处伪代码结合会变 某个东西跟 16 个字节异或 大概率 iv

```
79170F8664 [libmetasec_ov.so!D0664]   ldrb  w9, [x1, x8]             ;  r[x1=7ab35576d0 x8=1] w[w9=e9]
memory read at 0x7ab35576d1, data size = 1, data value = e9
79170F8668 [libmetasec_ov.so!D0668]   ldrb  w10, [x0, x8]            ;  r[x0=78453556c0 x8=1] w[w10=20]
memory read at 0x78453556c1, data size = 1, data value = 20

```

搜索 78453556c0 寻找写入地方 就是 iv key 同理

寻找如参

```
00000000: EC E9 BB C9 5E 01 50 07  18 37 D8 8C F3 A8 43 8D  ....^.P..7....C.
00000010: D9 BB 87 8C 87 73 0E 57  92 0F 4B BB 85 59 73 99  .....s.W..K..Ys.

```

会发现两块

```
call libc.so!memmove 79f38af810 <- 79f34deb30 size:9
0000: ec e9 bb c9 5e 01 50 07 18  ....^.P..

```

```
call libc.so!memmove 7ab35465d0 <- 7ac35abd40 size:385
0000: ec e9 bb c9 5e 01 50 07 18 37 d8 8c f3 a8 43 8d  ....^.P..7....C.
0010: d9 bb 87 8c 87 73 0e 57 92 0f 4b bb 85 59 73 99  .....s.W..K..Ys.
0020: 17 8d 16 56 4e 23 6b c0 58 d8 6c 03 3e 00 32 54  ...VN#k.X.l.>.2T
0030: 8c c7 a1 01 f1 28 c9 a1 74 9b 5d a7 aa b2 d7 c4  .....(..t.].....
0040: 71 98 d3 b8 05 28 53 a8 b1 87 fe d1 09 db e4 3c  q....(S........<

```

这说明 又是有拼接 直接寻找 7ac35abd40 或者 37 d8 8c f3

37 d8 8c f3 a8 43 8d

call libc.so!memmove 7ac35abd49 <- 7ab354aa10 size:376

```
0000: 37 d8 8c f3 a8 43 8d d9 bb 87 8c 87 73 0e 57 92  7....C......s.W.
0010: 0f 4b bb 85 59 73 99 17 8d 16 56 4e 23 6b c0 58  .K..Ys....VN#k.X
0020: d8 6c 03 3e 00 32 54 8c c7 a1 01 f1 28 c9 a1 74  .l.>.2T.....(..t

```

先加到地址存起来然后后面 strb

```
79170BF344 [libmetasec_ov.so!97344]   add  x8, x8, x10               ;  r[x8=177 x10=7ab354aa10] w[x8=7ab354ab87]
79170BEA74 [libmetasec_ov.so!96A74]   ldr  w9, [x12, #8]             ;  r[x12=78453571c0] w[w9=53]
memory read at 0x78453571c8, data size = 4, data value = 53
79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=53 x8=7ab354ab87]
memory write at 0x7ab354ab87, data size = 1, data value = 53

```

寻找 w9

```
Line 3870633  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=37 x8=7ab354aa10]"
Line 3872373  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=d8 x8=7ab354aa11]"
Line 3874113  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=8c x8=7ab354aa12]"
Line 3875853  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=f3 x8=7ab354aa13]"
Line 3877593  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=a8 x8=7ab354aa14]"
Line 3879333  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=43 x8=7ab354aa15]"
Line 3881073  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=8d x8=7ab354aa16]"
Line 3882813  6h  "79170BEA78 [libmetasec_ov.so!96A78]   strb  w9, [x8]                 ;  r[w9=d9 x8=7ab354aa17]"

```

都走一个地方 而且最后两个字节没有 是填充 rand

再寻找 0x37 会发现规律 其实就是一个异或不同的字节 有四个 是会根据 rand 变的

```
79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=c8]
memory read at 0x78453571f8, data size = 8, data value = c8
79170BF2F0 [libmetasec_ov.so!972F0]   ldr  x8, [x11, w8, uxtw #3]    ;  r[x11=78453571c0 w8=1] w[x8=ff]
memory read at 0x78453571c8, data size = 8, data value = ff
79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ff x10=c8] w[x8=37]
79170BF2F8 [libmetasec_ov.so!972F8]   str  x8, [x11, w9, uxtw #3]    ;  r[x8=37 x11=78453571c0 w9=1]

```

一步步找到如参即可  要注意 此处有坑 是先从高顺序向低顺序取

```
Line 3870546  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=c8]"
Line 3872286  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=20]"
Line 3874026  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=cb]"
Line 3875766  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=a0]"
Line 3877506  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=57]"
Line 3879246  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=bb]"
Line 3880986  6h  "79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=78453571c0 w10=7] w[x10=ca]"

```

可以找到如下

call libc.so!memmove 7ac3638e48 <- 7ac362f7f0 size:368

```
0000: a7 b3 91 23 73 df 52 7f 89 8b d1 84 be ef 69 1f  ...#s.R.......i.
0010: ed b8 e1 3f 95 1b d9 dc b0 eb 89 18 57 1f 87 73  ...?........W..s
0020: 35 3d c2 de 95 08 30 d2 63 94 be a9 92 04 6e ab  5=....0.c.....n.
0030: a1 c8 80 ce 56 69 df f6 c5 fe 12 80 93 7b b6 17  ....Vi.......{..
0040: fe 29 c0 02 ef 01 f2 76 7d f7 c2 6c c5 bb ea 43  .).....v}..l...C
0050: 1d cc 55 f4 b0 89 76 79 aa fe a3 2c e4 c1 4e f8  ..U...vy...,..N.
0060: 37 69 2f 59 ae f8 63 ff 40 ec a7 23 bc 14 4d 9f  7i/Y..c.@..#..M.

```

顺着写入一直向上找

```
79170BEB1C [libmetasec_ov.so!96B1C]   ldr  x9, [x12, #8]             ;  r[x12=7845355470] w[x9=7f52df732391b3a7]
memory read at 0x7845355478, data size = 8, data value = 7f52df732391b3a7
79170BEB20 [libmetasec_ov.so!96B20]   str  x9, [x8]                  ;  r[x9=7f52df732391b3a7 x8=7ab34d5ed0]
memory write at 0x7ab34d5ed0, data size = 8, data value = 7f52df732391b3a7

```

# 可以看到是一堆异或 and 等操作 此处还有个 key 扩展 都是比较简单的操作 但是需要

```
79170BF2EC [libmetasec_ov.so!972EC]   ldr  x10, [x11, w10, uxtw #3]  ;  r[x11=7845355450 w10=1] w[x10=a6714616c0779e5d]
memory read at 0x7845355458, data size = 8, data value = a6714616c0779e5d
79170BF2F0 [libmetasec_ov.so!972F0]   ldr  x8, [x11, w8, uxtw #3]    ;  r[x11=7845355450 w8=5] w[x8=d9239965e3e62dfa]
memory read at 0x7845355478, data size = 8, data value = d9239965e3e62dfa
79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d9239965e3e62dfa x10=a6714616c0779e5d] w[x8=7f52df732391b3a7]
79170BF2F8 [libmetasec_ov.so!972F8]   str  x8, [x11, w9, uxtw #3]    ;  r[x8=7f52df732391b3a7 x11=7845355450 w9=7]
memory write at 0x7845355488, data size = 8, data value = 7f52df732391b3a7

```

全局搜索! 972F4 可以看到规律 伪代码看不到 从 trace 还原

```
Line 1606190  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=210048280a4d208 x10=44d93c40830] w[x8=21000cf1360da38]"
Line 1606374  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c41088bb37cf1060 x10=21000cf1360da38] w[x8=c600887424afca58]"
Line 1606542  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=73ff54b8f2fb0aba x10=c600887424afca58] w[x8=b5ffdcccd654c0e2]"
Line 1607527  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3104222ecdf3c418 x10=6bdc889004808085] w[x8=5ad8aabec973449d]"
Line 1607711  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d7ff73335953038a x10=5ad8aabec973449d] w[x8=8d27d98d90204717]"
Line 1607879  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f90b05597c8a428c x10=8d27d98d90204717] w[x8=742cdcd4ecaa059b]"
Line 1608864  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=b5ffdcccd654c0e2 x10=285890a888040b34] w[x8=9da74c645e50cbd6]"
Line 1609048  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d0b37353b2a8166d x10=9da74c645e50cbd6] w[x8=4d143f37ecf8ddbb]"
Line 1609216  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=56828dbca76a33d2 x10=4d143f37ecf8ddbb] w[x8=1b96b28b4b92ee69]"
Line 1610201  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=742cdcd4ecaa059b x10=1620010292244812] w[x8=620cddd67e8e4d89]"
Line 1610385  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=6e5aca2d2e4bb9a4 x10=620cddd67e8e4d89] w[x8=c5617fb50c5f42d]"
Line 1610553  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=dabf024fba7e65f3 x10=c5617fb50c5f42d] w[x8=d6e915b4eabb91de]"
Line 1611538  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=1b96b28b4b92ee69 x10=a910206891110294] w[x8=b28692e3da83ecfd]"
Line 1611722  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5ba456d3aaee477b x10=b28692e3da83ecfd] w[x8=e922c430706dab86]"
Line 1611890  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4f723cdfc323bc6c x10=e922c430706dab86] w[x8=a650f8efb34e17ea]"
Line 1612875  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d6e915b4eabb91de x10=40a0e19346142a84] w[x8=9649f427acafbb5a]"
Line 1613059  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9943e3becd385faa x10=9649f427acafbb5a] w[x8=f0a17996197e4f0]"
Line 1613227  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=362e559273fcdb01 x10=f0a17996197e4f0] w[x8=3924420b126b3ff1]"
Line 1614212  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a650f8efb34e17ea x10=2040001220167120] w[x8=8610f8fd935866ca]"
Line 1614396  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e491082c49acffc4 x10=8610f8fd935866ca] w[x8=6281f0d1daf4990e]"
Line 1614564  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ab3adec05694cdf4 x10=6281f0d1daf4990e] w[x8=c9bb2e118c6054fa]"
Line 1615549  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3924420b126b3ff1 x10=932610000040a8c1] w[x8=aa02520b122b9730]"
Line 1615733  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=26ecb846318153eb x10=aa02520b122b9730] w[x8=8ceeea4d23aac4db]"
Line 1615901  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=925f293468882d35 x10=8ceeea4d23aac4db] w[x8=1eb1c3794b22e9ee]"
Line 1616886  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c9bb2e118c6054fa x10=314300420241c21c] w[x8=f8f82e538e2196e6]"
Line 1617070  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7ac70de52c8ba7b8 x10=f8f82e538e2196e6] w[x8=823f23b6a2aa315e]"
Line 1617238  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f6daabcebd477264 x10=823f23b6a2aa315e] w[x8=74e588781fed433a]"
Line 1618223  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=1eb1c3794b22e9ee x10=e18810102d420274] w[x8=ff39d3696660eb9a]"
Line 1618407  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d39621e07fb50ce9 x10=ff39d3696660eb9a] w[x8=2caff28919d5e773]"
Line 1618575  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e6c0e408cda11699 x10=2caff28919d5e773] w[x8=ca6f1681d474f1ea]"
Line 1619560  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=74e588781fed433a x10=416010020e1e2c0] w[x8=70f389783f0ca1fa]"
Line 1619744  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=29bc5a0751d3c7ab x10=70f389783f0ca1fa] w[x8=594fd37f6edf6651]"
Line 1619912  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ad018e5060c93a1d x10=594fd37f6edf6651] w[x8=f44e5d2f0e165c4c]"
Line 1620897  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ca6f1681d474f1ea x10=481c2a0e140c0890] w[x8=82733c8fc078f97a]"
Line 1621081  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d13974bc38597133 x10=82733c8fc078f97a] w[x8=534a4833f8218849]"
Line 1621249  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8f6701cd7e86ad7c x10=534a4833f8218849] w[x8=dc2d49fe86a72535]"
Line 1622234  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f44e5d2f0e165c4c x10=2848928405040048] w[x8=dc06cfab0b125c04]"
Line 1622418  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=70b527fa1a9c94d7 x10=dc06cfab0b125c04] w[x8=acb3e851118ec8d3]"
Line 1622586  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=459f521891f1af34 x10=acb3e851118ec8d3] w[x8=e92cba49807f67e7]"
Line 1623571  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=dc2d49fe86a72535 x10=1840800066c7c9] w[x8=dc35097e86c1e2fc]"
Line 1623755  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a4b2e92601fd9f9f x10=dc35097e86c1e2fc] w[x8=7887e058873c7d63]"
Line 1623923  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ae544dedf9d26162 x10=7887e058873c7d63] w[x8=d6d3adb57eee1c01]"
Line 1624908  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e92cba49807f67e7 x10=81a5116aec1c0002] w[x8=6889ab236c6367e5]"
Line 1625092  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5b4eb6d5fbb87007 x10=6889ab236c6367e5] w[x8=33c71df697db17e2]"
Line 1625260  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e5053de73e545418 x10=33c71df697db17e2] w[x8=d6c22011a98f43fa]"
Line 1626245  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d6d3adb57eee1c01 x10=80000021030282d4] w[x8=56d3ad947dec9ed5]"
Line 1626429  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5b088046a63d0feb x10=56d3ad947dec9ed5] w[x8=ddb2dd2dbd1913e]"
Line 1626597  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8538f144cc5fe5ec x10=ddb2dd2dbd1913e] w[x8=88e3dc96178e74d2]"
Line 1627582  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d6c22011a98f43fa x10=1c490040e14c080] w[x8=d706b015a79b837a]"
Line 1627766  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=238f72585e39d34a x10=d706b015a79b837a] w[x8=f489c24df9a25030]"
Line 1627934  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=1bea57c0be7b03f8 x10=f489c24df9a25030] w[x8=ef63958d47d953c8]"
Line 1628919  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=88e3dc96178e74d2 x10=4285090289128081] w[x8=ca66d5949e9cf453]"
Line 1629103  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=bd8e56351f654f23 x10=ca66d5949e9cf453] w[x8=77e883a181f9bb70]"
Line 1629271  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=44effe82bb7b30cb x10=77e883a181f9bb70] w[x8=33077d233a828bbb]"
Line 1630256  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ef63958d47d953c8 x10=60c220200011332] w[x8=e96fb78f47d840fa]"
Line 1630440  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=cc1df48cea0a2eec x10=e96fb78f47d840fa] w[x8=25724303add26e16]"
Line 1630608  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=18d4bc6170369ea x10=25724303add26e16] w[x8=24ff08c5bad107fc]"
Line 1631593  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=33077d233a828bbb x10=4908018a51020c20] w[x8=7a0f7ca96b80879b]"
Line 1631777  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=93fc2316eb441ff0 x10=7a0f7ca96b80879b] w[x8=e9f35fbf80c4986b]"
Line 1631945  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8cf10f5f7176c3b7 x10=e9f35fbf80c4986b] w[x8=650250e0f1b25bdc]"
Line 1632930  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=24ff08c5bad107fc x10=200a0c1a2409420] w[x8=26ffa804189193dc]"
Line 1633114  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=94094383c6c96f71 x10=26ffa804189193dc] w[x8=b2f6eb87de58fcad]"
Line 1633282  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8bdc98e2be7be0e6 x10=b2f6eb87de58fcad] w[x8=392a736560231c4b]"
Line 1634267  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=650250e0f1b25bdc x10=2250644000040810] w[x8=475234a0f1b653cc]"
Line 1634451  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e4a9cd95808c712c x10=475234a0f1b653cc] w[x8=a3fbf935713a22e0]"
Line 1634619  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=39db5ca242ed03a x10=a3fbf935713a22e0] w[x8=a0664cff5514f2da]"
Line 1635604  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=392a736560231c4b x10=404c99540020c0a0] w[x8=7966ea316003dceb]"
Line 1635788  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=819933fd5453cb6a x10=7966ea316003dceb] w[x8=f8ffd9cc34501781]"
Line 1635956  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=54b1512cc7f6037f x10=f8ffd9cc34501781] w[x8=ac4e88e0f3a614fe]"
Line 1636941  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a0664cff5514f2da x10=488800c1a60428ac] w[x8=e8ee4c3ef310da76]"
Line 1637125  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=b13a2383ce9853fa x10=e8ee4c3ef310da76] w[x8=59d46fbd3d88898c]"
Line 1637293  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a2e11b043b4e8c87 x10=59d46fbd3d88898c] w[x8=fb3574b906c6050b]"
Line 1638278  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ac4e88e0f3a614fe x10=3460a90204040a13] w[x8=982e21e2f7a21eed]"
Line 1638462  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ecd5d2e41b18142f x10=982e21e2f7a21eed] w[x8=74fbf306ecba0ac2]"
Line 1638630  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f8023ae2b3609e65 x10=74fbf306ecba0ac2] w[x8=8cf9c9e45fda94a7]"
Line 1639615  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=fb3574b906c6050b x10=19c180489a94210c] w[x8=e2f4f4f19c522407]"
Line 1639799  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=33e727917f6a529e x10=e2f4f4f19c522407] w[x8=d113d360e3387699]"
Line 1639967  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e20bd77d0286372d x10=d113d360e3387699] w[x8=3318041de1be41b4]"
Line 1640952  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8cf9c9e45fda94a7 x10=82182408020] w[x8=8cf9c1c5dd9a1487]"
Line 1641136  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=cc60107786f906d0 x10=8cf9c1c5dd9a1487] w[x8=4099d1b25b631257]"
Line 1641304  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=babeffcd6e9893d1 x10=4099d1b25b631257] w[x8=fa272e7f35fb8186]"
Line 1642289  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3318041de1be41b4 x10=240e5c346b810208] w[x8=171658298a3f43bc]"
Line 1642473  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e89cb9fcd7ee061b x10=171658298a3f43bc] w[x8=ff8ae1d55dd145a7]"
Line 1642641  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=75e1f36d555b396b x10=ff8ae1d55dd145a7] w[x8=8a6b12b8088a7ccc]"
Line 1643626  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=fa272e7f35fb8186 x10=1220000014c888] w[x8=fa350e7f35ef490e]"
Line 1643810  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=29ac4ae02229f332 x10=fa350e7f35ef490e] w[x8=d399449f17c6ba3c]"
Line 1643978  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=cd11d885b0a49999 x10=d399449f17c6ba3c] w[x8=1e889c1aa76223a5]"
Line 1644963  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8a6b12b8088a7ccc x10=81018254200050a] w[x8=827b0a9d4a8a79c6]"
Line 1645147  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7a22706a9d888e94 x10=827b0a9d4a8a79c6] w[x8=f8597af7d702f752]"
Line 1645315  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=df668e31c9b3c842 x10=f8597af7d702f752] w[x8=273ff4c61eb13f10]"
Line 1646300  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=1e889c1aa76223a5 x10=e74c00c31221020] w[x8=10fc5c1696403385]"
Line 1646484  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9cffd3187ac4fc40 x10=10fc5c1696403385] w[x8=8c038f0eec84cfc5]"
Line 1646652  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=fc4ab3cc4b3c8c7e x10=8c038f0eec84cfc5] w[x8=70493cc2a7b843bb]"
Line 1647637  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=273ff4c61eb13f10 x10=4010408508408370] w[x8=672fb44316f1bc60]"
Line 1647821  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c124f30a9ee10eed x10=672fb44316f1bc60] w[x8=a60b47498810b28d]"
Line 1647989  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=91cac5108f864a8a x10=a60b47498810b28d] w[x8=37c182590796f807]"
Line 1648974  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=70493cc2a7b843bb x10=4182000206280006] w[x8=31cb3cc0a19043bd]"
Line 1649158  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=df0609641e5be01c x10=31cb3cc0a19043bd] w[x8=eecd35a4bfcba3a1]"
Line 1649326  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=791e1aa07b39e7ff x10=eecd35a4bfcba3a1] w[x8=97d32f04c4f2445e]"
Line 1650311  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=37c182590796f807 x10=326040080440895] w[x8=34e7865987d2f092]"
Line 1650495  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5f4cbc1313c9117a x10=34e7865987d2f092] w[x8=6bab3a4a941be1e8]"
Line 1650663  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ba44b91a503a5f7f x10=6bab3a4a941be1e8] w[x8=d1ef8350c421be97]"
Line 1651648  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=97d32f04c4f2445e x10=a383008000021501] w[x8=34502f84c4f0515f]"
Line 1651832  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=47be0d431086fa5f x10=34502f84c4f0515f] w[x8=73ee22c7d476ab00]"
Line 1652000  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d67637199382f255 x10=73ee22c7d476ab00] w[x8=a59815de47f45955]"
Line 1652985  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d1ef8350c421be97 x10=8100a04844810a1] w[x8=d9ff89544069ae36]"
Line 1653169  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=966057791fd16556 x10=d9ff89544069ae36] w[x8=4f9fde2d5fb8cb60]"
Line 1653337  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4c0d888d9d94d019 x10=4f9fde2d5fb8cb60] w[x8=39256a0c22c1b79]"
Line 1654322  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a59815de47f45955 x10=204a04004183002] w[x8=a79cb59e43ec6957]"
Line 1654506  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e495a8308b06de4 x10=a79cb59e43ec6957] w[x8=a9d5ef1d4b5c04b3]"
Line 1654674  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5cc66951968abfc1 x10=a9d5ef1d4b5c04b3] w[x8=f513864cddd6bb72]"
Line 1655659  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=39256a0c22c1b79 x10=2060c9992a972e5] w[x8=1945a395085699c]"
Line 1655843  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d44e1933775aedcb x10=1945a395085699c] w[x8=d5da430a27df8457]"
Line 1656011  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=46a2c1cfde3fd7f9 x10=d5da430a27df8457] w[x8=937882c5f9e053ae]"
Line 1656996  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f513864cddd6bb72 x10=20800589e040a611] w[x8=d59383c53d961d63]"
Line 1657180  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4de20b17e7814eba x10=d59383c53d961d63] w[x8=987188d2da1753d9]"
Line 1657348  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7f3cf0083907b23c x10=987188d2da1753d9] w[x8=e74d78dae310e1e5]"
Line 1658333  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=937882c5f9e053ae x10=4c18d0a10021c1c3] w[x8=df605264f9c1926d]"
Line 1658517  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9d35e36b8c438797 x10=df605264f9c1926d] w[x8=4255b10f758215fa]"
Line 1658685  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9437b8abb1442a1 x10=4255b10f758215fa] w[x8=4b16ca85ce96575b]"
Line 1659670  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e74d78dae310e1e5 x10=1608850a94040a02] w[x8=f145fdd07714ebe7]"
Line 1659854  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=2c5b2a173a595d6d x10=f145fdd07714ebe7] w[x8=dd1ed7c74d4db68a]"
Line 1660022  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=77766f4fa486b047 x10=dd1ed7c74d4db68a] w[x8=aa68b888e9cb06cd]"
Line 1661007  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4b16ca85ce96575b x10=40900001c3060d8a] w[x8=b86ca840d905ad1]"
Line 1661191  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a9a2e223a72c1b36 x10=b86ca840d905ad1] w[x8=a22428a7aabc41e7]"
Line 1661359  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=60661d210a9cfe2a x10=a22428a7aabc41e7] w[x8=c2423586a020bfcd]"
Line 1662344  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=aa68b888e9cb06cd x10=4020000014d82] w[x8=aa6cba88e9ca4b4f]"
Line 1662528  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=908d61a8082ff37 x10=aa6cba88e9ca4b4f] w[x8=a3646c926948b478]"
Line 1662696  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e72b6bce119c7e16 x10=a3646c926948b478] w[x8=444f075c78d4ca6e]"
Line 1663681  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c2423586a020bfcd x10=8060c38d0880444] w[x8=ca4439be70a8bb89]"
Line 1663865  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=113c1d71e35329b9 x10=ca4439be70a8bb89] w[x8=db7824cf93fb9230]"
Line 1664033  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=88e6075811041d1a x10=db7824cf93fb9230] w[x8=539e239782ff8f2a]"
Line 1665018  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=444f075c78d4ca6e x10=86200702058f0a50] w[x8=c26f005e7d5bc03e]"
Line 1665202  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4e788e5e0bfe3ca9 x10=c26f005e7d5bc03e] w[x8=8c178e0076a5fc97]"
Line 1665370  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=214e239f579b0cf6 x10=8c178e0076a5fc97] w[x8=ad59ad9f213ef061]"
Line 1666355  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=539e239782ff8f2a x10=58a11b2002706081] w[x8=b3f38b7808fefab]"
Line 1666539  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=b566b67c84fbc186 x10=b3f38b7808fefab] w[x8=be598ecb04742e2d]"
Line 1666707  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ad1a1abdf3bb919a x10=be598ecb04742e2d] w[x8=13439476f7cfbfb7]"
Line 1667692  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ad59ad9f213ef061 x10=28420e5cf9f3702] w[x8=afdd8d7aeea1c763]"
Line 1667876  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4d0e51dbdf3efedc x10=afdd8d7aeea1c763] w[x8=e2d3dca1319f39bf]"
Line 1668044  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=2b33723996a95956 x10=e2d3dca1319f39bf] w[x8=c9e0ae98a73660e9]"
Line 1669029  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=13439476f7cfbfb7 x10=808018210660c1c1] w[x8=93c38c57f1af7e76]"
Line 1669213  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=2782ba629cd983a7 x10=93c38c57f1af7e76] w[x8=b44136356d76fdd1]"
Line 1669381  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e1259393b912d6d5 x10=b44136356d76fdd1] w[x8=5564a5a6d4642b04]"
Line 1670366  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c9e0ae98a73660e9 x10=2081024420080400] w[x8=e961acdc873e64e9]"
Line 1670550  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5592969b5190ac11 x10=e961acdc873e64e9] w[x8=bcf33a47d6aec8f8]"
Line 1670718  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c71020c8eeb1ddea x10=bcf33a47d6aec8f8] w[x8=7be31a8f381f1512]"
Line 1671703  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5564a5a6d4642b04 x10=e302051810140220] w[x8=b666a0bec4702924]"
Line 1671887  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ef8c6a3ce07c5449 x10=b666a0bec4702924] w[x8=59eaca82240c7d6d]"
Line 1672055  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=98dc2872c284bdf8 x10=59eaca82240c7d6d] w[x8=c136e2f0e688c095]"
Line 1673040  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7be31a8f381f1512 x10=260c0e088008101] w[x8=7983da6fb01f9413]"
Line 1673224  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=4db8bc39a230257 x10=7983da6fb01f9413] w[x8=7d5851ac2a3c9644]"
Line 1673392  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5fed901578b5c774 x10=7d5851ac2a3c9644] w[x8=22b5c1b952895130]"
Line 1674377  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c136e2f0e688c095 x10=541815281102020] w[x8=c47763a26798e0b5]"
Line 1674561  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8ad706e54a2544c0 x10=c47763a26798e0b5] w[x8=4ea065472dbda475]"
Line 1674729  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=744137c32799feaf x10=4ea065472dbda475] w[x8=3ae152840a245ada]"
Line 1675714  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=22b5c1b952895130 x10=6142840804489030] w[x8=43f745b156c1c100]"
Line 1675898  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=eb854a1028916b68 x10=43f745b156c1c100] w[x8=a8720fa17e50aa68]"
Line 1676066  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f591d6f8e402612c x10=a8720fa17e50aa68] w[x8=5de3d9599a52cb44]"
Line 1677051  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3ae152840a245ada x10=a3c1109210810408] w[x8=992042161aa55ed2]"
Line 1677235  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=778f6566694b2d11 x10=992042161aa55ed2] w[x8=eeaf277073ee73c3]"
Line 1677403  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=46d3a8226b540cfd x10=eeaf277073ee73c3] w[x8=a87c8f5218ba7f3e]"
Line 1678388  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5de3d9599a52cb44 x10=5089120030743e28] w[x8=d6acb59aa26f56c]"
Line 1678572  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a1f23d4862e9fcfa x10=d6acb59aa26f56c] w[x8=ac98f611c8cf0996]"
Line 1678740  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=12c48c4e58c0f820 x10=ac98f611c8cf0996] w[x8=be5c7a5f900ff1b6]"
Line 1679725  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a87c8f5218ba7f3e x10=5c3854900011a22c] w[x8=f444dbc218abdd12]"
Line 1679909  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f971e97e403fc6da x10=f444dbc218abdd12] w[x8=d3532bc58941bc8]"
Line 1680077  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7932cf560f1406e x10=d3532bc58941bc8] w[x8=aa61e4938655ba6]"
Line 1681062  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=be5c7a5f900ff1b6 x10=40c0810604aa608] w[x8=ba50724ff04557be]"
Line 1681246  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=2a987924e1956e98 x10=ba50724ff04557be] w[x8=90c80b6b11d03926]"
Line 1681414  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=cf5f0265bf12a858 x10=90c80b6b11d03926] w[x8=5f97090eaec2917e]"
Line 1682399  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=aa61e4938655ba6 x10=9708020c4081225c] w[x8=9dae1c4578e479fa]"
Line 1682583  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7e5c243abb0a45f9 x10=9dae1c4578e479fa] w[x8=e3f2387fc3ee3c03]"
Line 1682751  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=37948dde0c1808bf x10=e3f2387fc3ee3c03] w[x8=d466b5a1cff634bc]"
Line 1683736  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5f97090eaec2917e x10=2085214396242850] w[x8=7f12284d38e6b92e]"
Line 1683920  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=519ad6873fd8d2f3 x10=7f12284d38e6b92e] w[x8=2e88feca073e6bdd]"
Line 1684088  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=fcea105856f46618 x10=2e88feca073e6bdd] w[x8=d262ee9251ca0dc5]"
Line 1685073  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d466b5a1cff634bc x10=20c4900082040182] w[x8=f4a225a14df2353e]"
Line 1685257  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=498bba4947283717 x10=f4a225a14df2353e] w[x8=bd299fe80ada0229]"
Line 1685425  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c0cfb35370248945 x10=bd299fe80ada0229] w[x8=7de62cbb7afe8b6c]"
Line 1686410  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d262ee9251ca0dc5 x10=e20c1972f4890458] w[x8=306ef7e0a543099d]"
Line 1686594  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f798b2edebfa2db1 x10=306ef7e0a543099d] w[x8=c7f6450d4eb9242c]"
Line 1686762  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=78eb3bf413ff82f9 x10=c7f6450d4eb9242c] w[x8=bf1d7ef95d46a6d5]"
Line 1687747  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7de62cbb7afe8b6c x10=1c3af950028445ab] w[x8=61dcd5eb787acec7]"
Line 1687931  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=fc75fbe5751a9b56 x10=61dcd5eb787acec7] w[x8=9da92e0e0d605591]"
Line 1688099  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f267be944d29aa27 x10=9da92e0e0d605591] w[x8=6fce909a4049ffb6]"
Line 1689084  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=bf1d7ef95d46a6d5 x10=ce9000000093b66c] w[x8=718d7ef95dd510b9]"
Line 1689268  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=bf3a42690127fed9 x10=718d7ef95dd510b9] w[x8=ceb73c905cf2ee60]"
Line 1689436  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=a2d70966ac4afbe4 x10=ceb73c905cf2ee60] w[x8=6c6035f6f0b81584]"
Line 1690421  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=6fce909a4049ffb6 x10=400062e0a0100008] w[x8=2fcef27ae059ffbe]"
Line 1690605  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=b180d7dbc2e05611 x10=2fcef27ae059ffbe] w[x8=9e4e25a122b9a9af]"
Line 1690773  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=25d97b182a96c5ff x10=9e4e25a122b9a9af] w[x8=bb975eb9082f6c50]"
Line 1691758  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=6c6035f6f0b81584 x10=170eb900004c50a1] w[x8=7b6e8cf6f0f44525]"
Line 1691942  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=ee5d7ae420bdb142 x10=7b6e8cf6f0f44525] w[x8=9533f612d049f467]"
Line 1692110  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9aa61d070046b6d0 x10=9533f612d049f467] w[x8=f95eb15d00f42b7]"
Line 1693095  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=bb975eb9082f6c50 x10=152b14000002850e] w[x8=aebc4ab9082de95e]"
Line 1693279  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3e57ac57403d0adc x10=aebc4ab9082de95e] w[x8=90ebe6ee4810e382]"
Line 1693447  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=e4da6eced8b56e7b x10=90ebe6ee4810e382] w[x8=7431882090a58df9]"
Line 1694432  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=f95eb15d00f42b7 x10=2000000021091970] w[x8=2f95eb15f1065bc7]"
Line 1694616  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d0c62082429637e5 x10=2f95eb15f1065bc7] w[x8=ff53cb97b3906c22]"
Line 1694784  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=28cbdb385af55831 x10=ff53cb97b3906c22] w[x8=d79810afe9653413]"
Line 1695769  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=7431882090a58df9 x10=8810214940000007] w[x8=fc21a969d0a58dfe]"
Line 1695953  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5e6042bfa594d04f x10=fc21a969d0a58dfe] w[x8=a241ebd675315db1]"
Line 1696121  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=3a4651b7dbfd293f x10=a241ebd675315db1] w[x8=9807ba61aecc748e]"
Line 1697106  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d79810afe9653413 x10=a60824c108818] w[x8=d792702da575bc0b]"
Line 1697290  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=601ee986bb31d23a x10=d792702da575bc0b] w[x8=b78c99ab1e446e31]"
Line 1697458  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=6702147ccd96e7de x10=b78c99ab1e446e31] w[x8=d08e8dd7d3d289ef]"
Line 1698443  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=9807ba61aecc748e x10=800d1383828103d0] w[x8=180aa9e22c4d775e]"
Line 1698627  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=423a375f4f4a27bf x10=180aa9e22c4d775e] w[x8=5a309ebd630750e1]"
Line 1698795  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=8d1bc45d05aed729 x10=5a309ebd630750e1] w[x8=d72b5ae066a987c8]"
Line 1699780  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d08e8dd7d3d289ef x10=2a52a04089030891] w[x8=fadc2d975ad1817e]"
Line 1699964  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=5cad6b819aa61f23 x10=fadc2d975ad1817e] w[x8=a6714616c0779e5d]"
Line 1700132  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=d9239965e3e62dfa x10=a6714616c0779e5d] w[x8=7f52df732391b3a7]"

```

里面每一组操作用到一个 key 这个 key 扩展到 72 个 每一组会用到这 72 个子 key

每组的操作比较简单 但是 trace 中可能看着不那么清晰

上方是一组操作

```
Line 1606190  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=210048280a4d208 x10=44d93c40830] w[x8=21000cf1360da38]"
Line 1606374  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=c41088bb37cf1060 x10=21000cf1360da38] w[x8=c600887424afca58]"
Line 1606542  6h  "79170BF2F4 [libmetasec_ov.so!972F4]   eor  x8, x8, x10               ;  r[x8=73ff54b8f2fb0aba x10=c600887424afca58] w[x8=b5ffdcccd654c0e2]"

```

73ff54b8f2fb0aba f90b05597c8a428c 56828dbca76a33d2 dabf024fba7e65f3 是原来的 key 用这 4 个扩展 68 个 组成 72 个

前四次运算用到原来 key

然后是 如参 protobuf 用下方 py 库能直接解析出来 然后再寻找 buf 的每个参数含义以及声称方式即可

```
from protobuf_inspector.types import StandardParser
params = bytes.fromhex("08d2a4808204100218c4f3cd2e2204313233332a1337323132343533383938323439323538353031320a3231343238343035353142147630342e30342e30352d6f762d616e64726f696448c094a04052080000000000000000609cb581c30c6a06106e34a2b8c772069d3ed276b3f87a1008b2032008285e30b203388ab581c30c82011941454d65656f4b586644644c745361724f31485f6b4c59313588019cb581c30ca201046e6f6e65a801e205ba011c0a07506978656c203610121a0a676f6f676c65706c61792080d08340c20184014d44476b47707a58715845444c445a317957396e6c64546e6d7246345348386d5744637371713047543065444c453754527a5a704856643146306d784e51436d326567504a676a47776861627551374f554b656b55316558786a446656754d4f6559706e6d685247377a527172635a414675376d3833305659797664767453625566453dc80108d201100802120c04eaf91334fa5ba83a0075de")
parse_params = Argus_pb2.Argus()
parse_params.ParseFromString(params)
hexdump(parse_params.bodyHash)
hexdump(parse_params.queryHash)
hexdump(parse_params.extraInfo.algorithmData)
print(parse_params)
# with open("./test.bin", "wb") as f:
#     f.write(params)
print("\n")
parser = StandardParser()
with open('./test.bin', 'rb') as fh:
    output = parser.parse_message(fh, "message")
    print(output)

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/p5YHVYUwZib3iahCiaUVBBacXK7yLcInfBkDZonnQ3ZBD8brKDLN9XgcOCvLQboNd8PuIftfFwAfx5keVumQUuXnw/640?wx_fmt=png)

需要查看更多好文可以关注我朋友:  

我是 BestToYou, 分享工作或日常学习中关于 Android、iOS 逆向及安全防护的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误或者不足的地方, 恳请大家联系我批评指正。

![](https://mmbiz.qpic.cn/mmbiz_jpg/p5YHVYUwZib1LXXkvk6YVoJQJ90OVr4VIkII5veh8SKAFHibZFGiazq7ic7DWSfeTLl6Usp0GMANicVMvC5U0hKkDDg/640?wx_fmt=jpeg)

扫码加我为好友