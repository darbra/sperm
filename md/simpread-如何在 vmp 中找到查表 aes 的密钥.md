> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vwhTQ5_j4ltk6CCQFo2-Rw)

**前提: 没有魔改 aes**, **没有用白盒 aes**

前不久做了两个大厂 app 的设备注册, trace 的汇编数量都是超过**亿级别**的, 至于哪两个, 肯定是不能透露的.

首先搞清楚普通 aes, 查表 aes, 白盒 aes 的区别

普通 aes
======

指按照 NIST 标准实现的 AES 算法，严格按标准算法进行实现, 每一轮操作是逐步计算的

包括:

```
SubBytes（字节替代）
ShiftRows（行移位）
MixColumns（列混合）
AddRoundKey（轮密钥加）

```

详细的流程可以参考下图, 图片出自龙哥

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXyD5UNTVLmPVY0apqnH7bOXmoUFndpGlfAwxceW0EXkhCk8ATSjmnYfaMaG0gmIDJFxZic1aMTicCQ/640?wx_fmt=png&from=appmsg&watermark=1)

查表 aes
======

查表 AES 是普通 AES 的一种 **性能优化实现**，将多个操作合并为查表操作，以提高速度。

实现方法：

```
合并 SubBytes + ShiftRows + MixColumns 为预先计算好的查找表（T-tables）
用查表+异或的方式来快速执行每轮加密(异或指的是和扩展出的密钥异或,即AddRoundKey)

```

```
合并 SubBytes + ShiftRows + MixColumns 为预先计算好的查找表（T-tables）
用查表+异或的方式来快速执行每轮加密(异或指的是和扩展出的密钥异或,即AddRoundKey)

```

通常来说查表法的表都是 4 字节, 并且表中的元素数量很多, 并且有一些规律, 比如 0x31d7e6e6, 会有两个字节是一样的, e6e6. 但不一定都是这样的, 有的查表自己改了比较多东西的话常量是搜索不到的, 但是却还是标准的.

比如常量 0x6363a5c6,0x31d7e6e6google 即可找到是 aes 中常用的常量

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXyD5UNTVLmPVY0apqnH7bORgDncTvRN9TWstACw3z5n4aicasbY8DM4H62YkLA8K8V0utkM0SGbGQ/640?wx_fmt=png&from=appmsg&watermark=1)

查表 aes 不仅是计算时查表, 密钥扩展时也通常是查表进行.

白盒 aes
======

将密钥运算融合进查表 aes 里面, 密钥不可见, 属于查表 aes 的增强版本.

实现方法

```
将 AES 的逻辑转化为多个查找表（T-box、G-box等）；
将密钥混淆嵌入查找表中；
添加多层编码（Input/output encoding）来混淆每一轮的中间结果；
不使用可分离的密钥变量，密钥和算法结构被深度融合。

```

但仍然可以 dfa 找到密钥.

如何找查表 aes 密钥
============

这里我们不讨论白盒 aes, 聊聊如何在 trace 的亿级别汇编中找到查表 aes 的密钥.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXyD5UNTVLmPVY0apqnH7bOZ0ibLseXKTtde0iaz67gXIjCZMeOIU7kIxghVXiawKgnBrbNtjgpCOV6Q/640?wx_fmt=png&from=appmsg&watermark=1)

在不知道明文只知道密文的情况下, 肯定是从最后往前推, 最后一个运算是第 10 轮的轮密钥加 (异或)

由于查表都是 4 字节运算, 所以假设结果是 0x2066ec39, 来自 0x12ad6549^0x32cb8970

```
0x12ad6549^0x32cb8970=0x2066ec39 (瞎写的)

```

这就意味着 0x12ad6549 和 0x32cb8970 其中一个是上一个运算得到的 state, 另一个是 k10 的 4 字节.

aes 分组 16 字节, 就会有 4 组这样的运算

```
0x12ad6549^0x32cb8970=0x2066ec39
0x789ca192^0xac781930=0xd4e4b8a2
0x19803782^0x138109bc=0xa013e3e
0x13908971^0x3a78998b=0x29e810fa

```

通常来说 state 或者 k 会有一定的规律, 比如左边列全是 state, 右边列全是 k10 中的. 很少出现交叉出现的情况, 如果出现交叉的情况, 可以继续跟踪这组数据的生成规律来判断哪些是一组的.(说的有点抽象, 实际搞过的就懂)

也就是说 0x12ad6549 0x789ca192 0x19803782 0x13908971 和 0x32cb8970 0xac781930 0x138109bc 0x3a78998b 其中一组是密钥 k10.

假设是 k10 是由后面那组拼接的, 即是 32cb8970ac781930138109bc3a78998b

利用 aes_keyschedule 即可还原出主密钥还有各轮密钥, 这个时候如果在 trace 流中 4 字节搜索能搜到其他的密钥就说明猜想没有问题, 主密钥就是 FA4EC704730BC2A7745A1E640FA439AC, 注意搜索的时候大小端都尝试一下.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXyD5UNTVLmPVY0apqnH7bOGMp33JLS2KLuez4jsADhyLCticIeADGHWGxvJRNFmVuk0JkL39jcLpA/640?wx_fmt=png&from=appmsg&watermark=1)

还有一种情况可能是换了一个端序拼接的比如 7089cb32 301978ac bc098113 8b99783a

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdXyD5UNTVLmPVY0apqnH7bO5ofvkRCMAoxQtD3vvKcwOuhwrjqsbgIX75fwuWibKlzF4GNtiadticyiaQ/640?wx_fmt=png&from=appmsg&watermark=1)

如果能搜到其他密钥说明密钥就是 C821F9756D4CB112E13651B928FA9143.

如果都没有也可能密钥是另外一组数据, 可以尝试一下, 说不定有奇迹发生.

以上是一种比较无脑暴力的方法应对标准的白盒 aes, 更为准确的方法是根据标准 aes 的运算规律来猜测是在做相应的什么功能, 从而更加准确的做出判断.

如何你怀疑某一组数据可能是密钥, 也可以通过密钥编排算法生成轮密钥, 如果轮密钥能搜索到也可以证明密钥找对了.

```
Sbox = (
    0x63, 0x7C, 0x77, 0x7B, 0xF2, 0x6B, 0x6F, 0xC5, 0x30, 0x01, 0x67, 0x2B, 0xFE, 0xD7, 0xAB, 0x76,
    0xCA, 0x82, 0xC9, 0x7D, 0xFA, 0x59, 0x47, 0xF0, 0xAD, 0xD4, 0xA2, 0xAF, 0x9C, 0xA4, 0x72, 0xC0,
    0xB7, 0xFD, 0x93, 0x26, 0x36, 0x3F, 0xF7, 0xCC, 0x34, 0xA5, 0xE5, 0xF1, 0x71, 0xD8, 0x31, 0x15,
    0x04, 0xC7, 0x23, 0xC3, 0x18, 0x96, 0x05, 0x9A, 0x07, 0x12, 0x80, 0xE2, 0xEB, 0x27, 0xB2, 0x75,
    0x09, 0x83, 0x2C, 0x1A, 0x1B, 0x6E, 0x5A, 0xA0, 0x52, 0x3B, 0xD6, 0xB3, 0x29, 0xE3, 0x2F, 0x84,
    0x53, 0xD1, 0x00, 0xED, 0x20, 0xFC, 0xB1, 0x5B, 0x6A, 0xCB, 0xBE, 0x39, 0x4A, 0x4C, 0x58, 0xCF,
    0xD0, 0xEF, 0xAA, 0xFB, 0x43, 0x4D, 0x33, 0x85, 0x45, 0xF9, 0x02, 0x7F, 0x50, 0x3C, 0x9F, 0xA8,
    0x51, 0xA3, 0x40, 0x8F, 0x92, 0x9D, 0x38, 0xF5, 0xBC, 0xB6, 0xDA, 0x21, 0x10, 0xFF, 0xF3, 0xD2,
    0xCD, 0x0C, 0x13, 0xEC, 0x5F, 0x97, 0x44, 0x17, 0xC4, 0xA7, 0x7E, 0x3D, 0x64, 0x5D, 0x19, 0x73,
    0x60, 0x81, 0x4F, 0xDC, 0x22, 0x2A, 0x90, 0x88, 0x46, 0xEE, 0xB8, 0x14, 0xDE, 0x5E, 0x0B, 0xDB,
    0xE0, 0x32, 0x3A, 0x0A, 0x49, 0x06, 0x24, 0x5C, 0xC2, 0xD3, 0xAC, 0x62, 0x91, 0x95, 0xE4, 0x79,
    0xE7, 0xC8, 0x37, 0x6D, 0x8D, 0xD5, 0x4E, 0xA9, 0x6C, 0x56, 0xF4, 0xEA, 0x65, 0x7A, 0xAE, 0x08,
    0xBA, 0x78, 0x25, 0x2E, 0x1C, 0xA6, 0xB4, 0xC6, 0xE8, 0xDD, 0x74, 0x1F, 0x4B, 0xBD, 0x8B, 0x8A,
    0x70, 0x3E, 0xB5, 0x66, 0x48, 0x03, 0xF6, 0x0E, 0x61, 0x35, 0x57, 0xB9, 0x86, 0xC1, 0x1D, 0x9E,
    0xE1, 0xF8, 0x98, 0x11, 0x69, 0xD9, 0x8E, 0x94, 0x9B, 0x1E, 0x87, 0xE9, 0xCE, 0x55, 0x28, 0xDF,
    0x8C, 0xA1, 0x89, 0x0D, 0xBF, 0xE6, 0x42, 0x68, 0x41, 0x99, 0x2D, 0x0F, 0xB0, 0x54, 0xBB, 0x16,
)
Rcon = (0x00, 0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1B, 0x36) # 11个
def shiftRound(array, num):
    return array[num:] + array[:num]
def g(array, index):
    # 首先循环左移1位
    array = shiftRound(array, 1)
    # 字节替换
    array = [Sbox[i] for i in array]
    # 首字节 和 rcon 中对应元素异或
    array = [(Rcon[index] ^ array[0])] + array[1:]
    return array
def xorTwoArray(array1, array2):
    return [array1[i] ^ array2[i] for i in range(len(array1))]
def showRoundKeys(kList):
    for i in range(len(kList)):
        print("K%02d:" %i +"".join("%02x" % k for k in kList[i]))
def keyExpand(key):
    key = key.to_bytes(16, byteorder="big")
    extendKey = []
    for i in range(4):
        extendKey.append([j for j in key[4*i:4*i+4]]) # 4字节切割
    round_keys = [[0] * 4 for i in range(44)]
    # 11个密钥
    # 最开始的 k00
    for i in range(4):
        round_keys[i] = extendKey[i]
    # k01-k10
    for i in range(4, 4 * 11): # 40轮扩展出来10个密钥
        if i % 4 == 0:
            round_keys[i] = xorTwoArray(g(round_keys[i - 1], i // 4), round_keys[i - 4])
        else:
            round_keys[i] = xorTwoArray(round_keys[i - 1], round_keys[i - 4])
    kList = [[] for i in range(11)]
    for i in range(len(round_keys)):
        kList[i//4] += round_keys[i]
    showRoundKeys(kList)
    return kList
key = 0xC821F9756D4CB112E13651B928FA9143
kList = keyExpand(key)

```

至于解密出来模式是 ecb 还是 cbc, 可以尝试先 ecb 解密, 如果不能解密出来或者解密出部分, 可能就是 cbc 模式, 可以跟踪 k0 的异或来找明文还有 iv.

**为什么开发人员不搞更加难破解的白盒 aes?**

白盒 aes 密钥固定, 如果被破解就可以直接解密. 查表 aes 还有密钥扩展部分, 如果把密钥弄成动态加密一起发送到服务器就还需要还原密钥加密算法, 这是我做的某个大厂 app 是这样做的.

还有就是白盒 aes 难度实现大一些.

**以上说辞属于纯理论, 仅供参考.**