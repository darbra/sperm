> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9FnAk4fvnYonjO0p2vneKg)

### 抓包数据分析

我们开启抓包进入 APP，发现第一个请求便是指纹数据。数据包如下：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcAfqucxq3nvez2S2lsYWiaY9rNM63TaBEyPKL4UNPab6RN9TQSSUibB9w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)p1

查看 Content-Type 发现使用的二进制格式发送的数据包，因此我们排除这是 Base64 编码的数据包。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcSQVc3OJVHnyDEKazmrh8L3doICKgH1hUUfCbUsbFYUtZYibwRCruyVQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1)p2

现在我们需要知道，数据包是从哪里发出的。我们 HOOK 一下 RegisterNatives，看看发包前加载了哪些 SO 以及注册了哪些函数。

```
[RegisterNatives] java_class: com.xxx.HelperJNI name: call0 sig: (ILjava/lang/Object;)Ljava/lang/Object; fnPtr: 0x6dda03ef14 libxxx.so!0x82f14

```

观察得知在发包前 SO 注册了上图这个函数，我们 HOOK 一下这个函数看看调用情况。

```
let targetModule = Process.findModuleByName(so_name)
Interceptor.attach(targetModule.base.add(0x82f14), {
    onEnter(args) {
        //查看函数签名得知第一个参数是 int 类型。JNI函数前两个函数分别是 JNIenv 与 Jobject，因此我们打印 args[2]
                console.log('[*] Argument 1 (int):', args[2].toInt32())
    }
})

```

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcD6ibEjic16WTI6mcAvUVqwolEkyEEict3969icxOWUeuThpoAYpuDNyEDw/640?wx_fmt=png&from=appmsg#imgIndex=2)p3

这里看到一共调用了三百多次，这是典型的传入不同编号走不同处理分支的分发函数，各大厂的算法与设备信息收集都是这样设计的。因此，我们可以大胆下判断，发包的请求就藏在这三百多个调用中。那么我们怎么确定是哪个编号发出的请求呢？我们可以 HOOK 一下 memcpy 函数，因为在 Native 层要进行数据传递通过 memcpy 来复制数据十分便捷高效，而数据包就极有可能在 memcpy 中被复制过。这里插播一个知识点：为什么各大厂商都知道 memcpy 可能是是暴露数据的库函数，为什么还要调用它，而不自行实现？因为 libc 中的 memcpy 使用汇编实现，同时针对不同的 cpu 架构会使用特定的高级指令来加速数据的拷贝。我们可以查看 AOSP 中的 common/arch/arm64/lib/memcpy.S 文件源码来验证：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc1eHePKq2MNvBhyk7vCTFv6sicJXvXfpJiczgLibMsckurwAbr5y62VnnQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3)p4

可以看到这里判断了是否支持 MOPS ，如果支持则使用特定指令完成拷贝。而且部分编译器在编译期间会将 memcpy 替换为 memcpy_chk 函数，memcpy_chk 是增加了 src 缓冲区长度校验的版本，防止缓冲区溢出从而导致程序崩溃。这也相当于为程序的健壮性做了兜底保证。当然，如果厂商想硬实现 memcpy 也是可以的。但是这仍不能避免被我们分析，因为最终 APP 要运行在我们的设备上，而设备是我们能完全控制的。因此，在性能与实现复杂度的双重考量下，大部分厂商仍选择使用库函数的 memcpy。下面，我们回到正题。我们 HOOK memcpy 的同时将函数的调用编号也打印出来，观察目标数据包前后的函数编号，以此来确定是在哪个编号中发出的数据包。

```
Interceptor.attach(Module.findExportByName("libc.so", "memcpy"), {
    onEnter(args) {
          //判断下是否来自目标SO的调用，并且长度大于200才打印。因为 memcpy 是超高频调用函数，不做条件筛选打印程序大概率会崩溃。
        if (targetModule.base.compare(this.context.lr) === -1 && targetModule.base.add(targetModule.size).compare(this.context.lr) === 1 && args[2].toInt32() > 200) {
              console.log(hexdump(args[1], {
                  length: args[2].toInt32()
              }))
        }
    }
})

```

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcG0ZZlk3IKwgXjW2p3WEhx8RzxNKqWe2BBk2TTmZxA71o6Oju3fkMvw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)p5

从上图中我们可以看出在目标数据包前后，有两个编号：83、12。我们怎么判定是在哪个编号中发出的请求？可以分别搜索一下这两个编号，看看哪个编号是第一次被传入的，如果有编号是第一次被传入的那么大概率便是这个编号发出的数据包。经过搜索发现编号 83 出现了两次，而 12 只出现了一次，那么我们可以初步判定编号 12 为发包编号。那么我们怎么判定这就是最终的发包编号？我们可以在编号 12 被传入时，将进程暂停，如果暂停了仍然发包出去了，那么证明 12 不是最终的发包编号，反之则实锤这是发包编号。下面我们验证：

```
let targetModule = Process.findModuleByName(so_name)
Interceptor.attach(targetModule.base.add(0x82f14), {
    onEnter(args) {
        //查看函数签名得知第一个参数是 int 类型。JNI函数前两个函数分别是 JNIenv 与 Jobject，因此我们打印 args[2]
                console.log('[*] Argument 1 (int):', args[2].toInt32())
        if (args[2].toInt32() == 12) {
            let sleep = new NativeFunction(Module.getExportByName('libc.so', 'sleep'), 'uint', ['uint'])
            sleep(5)
        }
    }
})

```

在编号 12 被传入时我们暂停进程 5 秒，观察抓包发现并没有请求发出，那么至此我们可以确定数据包就是在编号为 12 的函数调用中发出。知道了发包的位置，接下来我们看看包数据是怎么组装加密的。

### 识别加密算法

在我们 HOOK RegisterNatives 时得到了分发函数的偏移 0x82f14。现在我们进入 SO 中看看这个函数。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcgic0g1Zb9HJ9W0bxuff2gYZQ9ENnO40PqJ1l3NAicxDCl0Y8TibkxficLA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5)p6

这个函数只是做了些初始化引导工作，我稍微跟了下初始化函数发现是有 OLLVM 混淆与 VMP 防护的。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcjkH8Cjd6nzTyjWia64Eo8J33L1kHVarzzL2NDCltibvZgQhfOSYkJZDw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)p7

这种情况下如果我们不深究其虚拟机实现的话，可以使用 trace 方案拿到执行指令流展开算法的分析。这里我用自己写的 trace 跟踪了编号 12 函数调用时的指令流，最终得到一个 8.3G，8000W 行的指令流文件。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcK57GQdI72icpjL2xPHboJjicFl3KaaJDn0nB57loCcVxEibbns9QFc7fg/640?wx_fmt=png&from=appmsg#imgIndex=7)p8

现在我们已知加密密文如下：

```
59 07 5E 07 0D 59 01 59 4C 59 0A 01 09 4C 5B 0E 06 56 50 5A 54 56 5A 53 42 0D 55 5F 56 42 58 01 07 51 AC 3F 32 85 19 6C 5A 31 4A 2F 66 58 EB D2 .............

```

既然我们得到了完整的指令流文件，那么密文数据一定会在某条指令中出现。因此，我们可以在文件中搜索对应的字节来找到密文生成的位置。

我以四个字节为一组尝试在文件中搜索，直至搜索到 33-36 字节时，也就是 0751AC3F，才第一次在文件中搜索到对应数据。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcskzllKuZP5PhCVchicxia5icjibibLeGFRJrm5o7Lx88PKkibTJJGVgyfdiaQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=8)p9

我们可以看到，这组数据第一次生成的位置在 0xa59c8 ，因此我们在 IDA 中跳转到目标地址查看伪代码看其是如何生成的。

```
_BYTE *__fastcall sub_A5114(unsigned __int8 *a1, _BYTE *a2, _DWORD *a3)
{
  int v3; // w8
  _BYTE *result; // x0
  int i; // [xsp+28h] [xbp-48h]
  int v6; // [xsp+2Ch] [xbp-44h]
  unsigned int v7; // [xsp+30h] [xbp-40h]
  unsigned int v8; // [xsp+34h] [xbp-3Ch]
  unsigned int v9; // [xsp+38h] [xbp-38h]
  unsigned int v10; // [xsp+3Ch] [xbp-34h]
  unsigned int v11; // [xsp+40h] [xbp-30h]
  unsigned int v12; // [xsp+40h] [xbp-30h]
  unsigned int v13; // [xsp+44h] [xbp-2Ch]
  unsigned int v14; // [xsp+44h] [xbp-2Ch]
  unsigned int v15; // [xsp+48h] [xbp-28h]
  unsigned int v16; // [xsp+48h] [xbp-28h]
  unsigned int v17; // [xsp+4Ch] [xbp-24h]
  unsigned int v18; // [xsp+4Ch] [xbp-24h]
  _DWORD *v19; // [xsp+50h] [xbp-20h]

  v19 = a3;
  v17 = (*a1 << 24) ^ (a1[1] << 16) ^ (a1[2] << 8) ^ a1[3] ^ *a3;
  v15 = (a1[4] << 24) ^ (a1[5] << 16) ^ (a1[6] << 8) ^ a1[7] ^ a3[1];
  v13 = (a1[8] << 24) ^ (a1[9] << 16) ^ (a1[10] << 8) ^ a1[11] ^ a3[2];
  v11 = (a1[12] << 24) ^ (a1[13] << 16) ^ (a1[14] << 8) ^ a1[15] ^ a3[3];
  v6 = a3[60] >> 1;
  for ( i = 1442911471; ; i = 1442911471 )
  {
    while ( 1 )
    {
      while ( i == 1442911471 )
      {
        v10 = table_1[HIBYTE(v17)] ^ table_2[BYTE2(v15)] ^ table_3[BYTE1(v13)] ^ table_4[v11] ^ v19[4];
        v9 = table_1[HIBYTE(v15)] ^ table_2[BYTE2(v13)] ^ table_3[BYTE1(v11)] ^ table_4[v17] ^ v19[5];
        v8 = table_1[HIBYTE(v13)] ^ table_2[BYTE2(v11)] ^ table_3[BYTE1(v17)] ^ table_4[v15] ^ v19[6];
        v7 = table_1[HIBYTE(v11)] ^ table_2[BYTE2(v17)] ^ table_3[BYTE1(v15)] ^ table_4[v13] ^ v19[7];
        v19 += 8;
        if ( --v6 )
          v3 = 1306952226;
        else
          v3 = -1331144664;
        i = v3;
      }
      if ( i != -1331144664 )
        break;
      i = -268623357;
    }
    if ( i != 1306952226 )
      break;
    v17 = table_1[HIBYTE(v10)] ^ table_2[BYTE2(v9)] ^ table_3[BYTE1(v8)] ^ table_4[v7] ^ *v19;
    v15 = table_1[HIBYTE(v9)] ^ table_2[BYTE2(v8)] ^ table_3[BYTE1(v7)] ^ table_4[v10] ^ v19[1];
    v13 = table_1[HIBYTE(v8)] ^ table_2[BYTE2(v7)] ^ table_3[BYTE1(v10)] ^ table_4[v9] ^ v19[2];
    v11 = table_1[HIBYTE(v7)] ^ table_2[BYTE2(v10)] ^ table_3[BYTE1(v9)] ^ table_4[v8] ^ v19[3];
  }
  v18 = table_3[HIBYTE(v10)] & 0xFF000000 ^ table_4[BYTE2(v9)] & 0xFF0000 ^ table_1[BYTE1(v8)] & 0xFF00 ^ table_2[v7] ^ *v19;
  *a2 = HIBYTE(v18);
  a2[1] = BYTE2(v18);
  a2[2] = BYTE1(v18);
  a2[3] = v18;
  v16 = table_3[HIBYTE(v9)] & 0xFF000000 ^ table_4[BYTE2(v8)] & 0xFF0000 ^ table_1[BYTE1(v7)] & 0xFF00 ^ table_2[v10] ^ v19[1];
  a2[4] = HIBYTE(v16);
  a2[5] = BYTE2(v16);
  a2[6] = BYTE1(v16);
  a2[7] = v16;
  v14 = table_3[HIBYTE(v8)] & 0xFF000000 ^ table_4[BYTE2(v7)] & 0xFF0000 ^ table_1[BYTE1(v10)] & 0xFF00 ^ table_2[v9] ^ v19[2];
  a2[8] = HIBYTE(v14);
  a2[9] = BYTE2(v14);
  a2[10] = BYTE1(v14);
  result = a2;
  a2[11] = v14;
  v12 = table_3[HIBYTE(v7)] & 0xFF000000 ^ table_4[BYTE2(v10)] & 0xFF0000 ^ table_1[BYTE1(v9)] & 0xFF00 ^ table_2[v8] ^ v19[3];
  a2[12] = HIBYTE(v12);
  a2[13] = BYTE2(v12);
  a2[14] = BYTE1(v12);
  a2[15] = v12;
  return result;
}

```

跳转到 0xa59c8 后我们看到这个函数，函数中有大量的查表与异或操作，因此我们可以判断这是一个加密函数。既然是加密函数，那么我们就要识别这是个什么加密算法。我们可以看看这四个表的数据分别是什么，以此来判断这是什么加密算法。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcay24DPAgHWyEQeBKicMAXicrFaLvFSIJ9n9KDWXibuGpkYMWus29jCjpA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=9)p10![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc7XOZpe7pSZW1iaBFbnbCpj9fvfBuqTDS1Rna5EMSq5Fe26SEYmZsTQA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=10)p11

我们可以看到表 1 与表 2 分别有大量的 63 7C 77 99 等字节出现，这些字节一般会出现在 AES 加密算法中的 S 盒中，因此我们初步判断这是个类 AES 算法。我们再观察下加密函数发现，a1 一进来便与 a3 进行了异或。而我们通过伪代码可知 a1 是个长度为 16 的字节数组，a3 是个长度为 4 的字节数组。将 a1 中的数据以四个字节为一组，通过 << 位操作组成一个 32 位的数据以对应 a3 的数据格式，随后与 a3 进行异或。我们再观察下函数的结尾部分，对 a2 中的每个元素进行赋值，赋值长度为 16，并且每个赋值语句的最后一步都是为 v19 进行异或，而 v19 是由 a3 赋值而来。因此我们得出如下结论：

> 将输入数据 a1 经过加密得到输出数据 a2，每一轮循环中都有与 a3 异或的步骤。每轮异或的字节长度为 16，并且输入输出数据长度也均为 16 。

我们回忆下 AES 128 算法实现：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcjLnM948NJKxhw4uMV6XmyBeRyvHxydiaAcIPuicgmIOZckJ4281zZ52Q/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11)

> 1byte = 8bit  16byte * 8 = 128bit

输入明文 128 bit，输出密文 128 bit。加密开始时输入明文与扩展密钥进行异或，随后进行 10 轮加密，最后一轮的最后一步是与扩展密钥进行异或得到最终密文。现在我们将 AES 128 与我们得到的结论进行对比发现三个相似点：

*   • 输入输出长度吻合
    
*   • 每轮进行轮密钥加吻合
    
*   • 开始与结束都是与密钥进行异或。
    

并且结合表数据判断，我们可以确定这是个魔改 AES 128 查表实现。查表实现是标准实现的性能优化版，节省列混淆的计算过程达到提升性能的目的。并且在逆向中不易被还原。

### 寻找密钥

现在我们确定了加密算法为 AES 128，那么我们就可以根据 AES 的实现步骤来找到密钥。现在我们已知的信息有：

*   • 输入明文为 a1
    
*   • 输出密文为 a2 = 07 51 AC 3F 32 85 19 6C 5A 31 4A 2F 66 58 EB D2
    
*   • 密钥集为 a3
    

我们知道 aes 最后一步加密为 state ^ key = ciphertext。因此，我们可以在指令流中找到最后一步密文生成的位置，看看是哪些数据与 state 异或后得到了密文，与 state 异或的便是密钥。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcskzllKuZP5PhCVchicxia5icjibibLeGFRJrm5o7Lx88PKkibTJJGVgyfdiaQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=12)p12

我们可以看到

```
eor w18, w18, w2 ; w18=0xbd9c788b w18=0xbd9c788b w2=0xbacdd4b4 -> w18=0x751ac3f

```

这行指令便是密文最终生成的位置。我们将其还原成伪 C 代码：state = state ^ key 也就是说左边的 w18 是 state，右边的 w2 为密钥。至此我们就得到了最后一轮的第一组密钥：0xbacdd4b4。

那么我们怎么得到最后一轮的全部密钥呢？我们可以以四字节为一组依次向后搜索密文生成的位置，将所有的被异或寄存器拼接起来便是最后一轮的密钥。搜索后得到如下四条指令：

```
eor w18, w18, w2 ; w18=0xbd9c788b w18=0xbd9c788b w2=0xbacdd4b4 -> w18=0x751ac3f 
eor w18, w18, w2 ; w18=0x9fe04871 w18=0x9fe04871 w2=0xad65511d -> w18=0x3285196c 
eor w18, w18, w2 ; w18=0x66e5fad9 w18=0x66e5fad9 w2=0x3cd4b0f6 -> w18=0x5a314a2f
eor w8, w8, w9 ; w8=0x257e5876 w8=0x257e5876 w9=0x4326b3a4 -> w8=0x6658ebd2

```

将被异或寄存器的值拼接起来，我们得到了最后一轮的密钥：

ba cd d4 b4 ad 65 51 1d 3c d4 b0 f6 43 26 b3 a4

现在我们验证一下我们的密钥是不是正确的。根据 AES 的密钥扩展算法可知，我们只需要得到任意一轮中的密钥，就能逆推出初始密钥。这里我们使用 Stack 提供的实现来逆推出初始密钥。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODczQB5aS1qlLNb0v28GohrYuJBaPn3MINvibjCChWmcibibJtzOIic5Xed9w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=13)p13

```
K00: C0789428EBD15058106777E942A95EBF

K01: 12209C04F9F1CC5CE996BBB5AB3FE50A

K02: 65F9FB669C08373A759E8C8FDEA16985

K03: 53006C7BCF085B41BA96D7CE6437BE4B

K04: C1AEDF380EA68479B43053B7D007EDFC

K05: 14FB6F481A5DEB31AE6DB8867E6A557A

K06: 3607B5BB2C5A5E8A8237E60CFC5DB376

K07: 3A6A8D0B1630D3819407358D685A86FB

K08: 042E824E121E51CF86196442EE43E2B9

K09: 05B6D46617A885A991B1E1EB7FF20352

K10: BACDD4B4AD65511D3CD4B0F64326B3A4

```

我们看到 Stack 逆推出了每一轮的密钥，现在我们以四个字节为一组在指令流中搜索密钥。如果能搜到则证明我们的密钥没有问题。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc4H8N5wKRPmbMVaO2XHISdjOIg2ibC7WH4icjDpzOWK0m3T2W3ibKcZicog/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=14)p14![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc7ia7Jwu9rUib6eVbUI3DH2fCTXQFOMIMiaSw07ylfT9Z6rkD4cHJ89EHw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=15)p15

我们可以看到 K09 的最后四个字节能搜索到，而初始密钥 K00 的最后四个字节却搜索不到。因此，我们可以断定厂商魔改了密钥扩展算法。接下来，我们还原密钥扩展算法。

### 密钥扩展算法还原

在搜索 7FF20352 时发现一共有 954 个结果，而第一个结果便是 7FF20352 最初生成的位置。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODclmNnsNAfLtus5fsplXIwITtKTTIkCeR7Z0HaCPMahEq2aQ0vbJohsw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=16)p16

因此我搜索了 0x2f83c 地址执行的所有指令，发现最后一轮与 K09 的密钥均在这条指令中出现：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcRRWoMT0ocibztGdicicRIZYz9j9NM899Ho4rwRGl8vFXV702LEXMAZVfA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=17)p17

那么我们可以判断，这条指令是密钥扩展算法中的最后一条指令。那是不是意味着我们在 IDA 中跳转到这个地址，就能看到魔改后的密钥扩展算法？我们跳转到目标地址看看：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcFperGCBbEwrmfBzdcKD3Sz8ulp372QJDzdDw8v3icXLFkD0VAIUWTng/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=18)p18

跳转到 0x2f83c 后我稍微看了下这个有 3000 多行的函数，这貌似是个 VM 的分发器。并且通过左下角的调用链可以看到这是个经过了超高强度的混淆函数，而且没有任何与密钥扩展算法相关的信息。因此，我们只能回到汇编指令流中来分析密钥扩展算法。

在 0x2f83c 这条指令搜索结果中的最上面，我们有了意外发现：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcpfYwLzOw76nd2uHmlibhFRkIibpzjd72HJhgKmAwvfSCE6rRBFcFjib9w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=19)p19

我们看到标红区域的 X8 结果便是我们发送的数据包开头 32 个字节。并且这些结果都是 X8 与 X2 异或得来的结果，异或之前 X8 的值是一串固定的常量，因为异或之前的 X8 与厂商信息有关，所以我将其码掉了。现在，我们思考为什么要将这 32 个字节与固定常量异或后发送给服务端。是不是因为这是密钥？如果我们顺着这个逻辑思考下去便可以推导出：与固定常量异或是为了隐藏初始密钥，防止密钥暴露在请求数据包中。而为什么要将它发送给服务端呢？因为服务端拿不到初始密钥便不能对数据包进行解密。

事实上，动态密钥上报是非常常见的设计。如果厂商使用一个固定的密钥来对数据进行加密，那么攻击者只用破解成功一次便可以拿这串密钥去解密这个厂商旗下所有的 APP。而使用动态密钥迫使攻击者每碰见一个 APP 就必须得去找到固定常量才能解密密钥，因此这相当于增加了攻击成本。这是一个非常好的对抗策略。

> 如果我们不能保证应用 100% 不被破解，那么至少我们可以增加攻击者的攻击成本以达到让攻击者停止攻击的目的。

好了，我们言归正传。现在我们思考，为什么是 32 个字节呢？在 AES 128 加密中密钥只占 16 个字节，如果密钥只占 16 个字节那么剩下的 16 个字节是什么呢？我们不妨回顾下 AES 的加密模式有哪些？常见的有两种：ECB、CBC。ECB 与 CBC 模式的唯一区别便是：ECB 没有 IV（初始向量）参与运算，而 CBC 则必须有 IV 参与运算。而 IV 在 AES 的加密中也是 16 个字节，那么我们就可以初步判断这 32 个字节是 KEY + IV。那么我们怎么验证？在 CBC 加密模式中，CBC 将待加密的明文切割成一组组 16 字节大小的数据集，在将第一组明文输入进 AES 加密之前，先将明文与 IV 逐字节进行异或，再输入 AES 中加密。而后每一组的明文数据都与上一组的密文数据进行异或后再输入进 AES 加密。也就是从第二组开始 IV 便是上一组 AES 加密后的密文。好了，现在我们搞清楚什么是 CBC 加密模式后，我们验证一下，看不看是不是真的如我们所说使用 CBC 模式进行加密。

首先我们先整理出来密钥与 IV：

密钥：35 62 31 66 62 35 64 36 2d 36 66 64 66 2d 34 62

IV：    63 39 31 35 38 33 35 32 2d 61 30 30 37 2d 34 64

我们还是看 0x2f83c 这条指令：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcuCd9gcaSP6ibNm50c5m4bkGHAqm5NbVELIgKe25KUMroRlpEZvZjenA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=20)p20

我们可以看到红色方框内 X8 异或之前的值便是我们的 IV，而蓝色方框内 X8 异或之前的值便是我们的第一组加密后的密文。符合我们所说的 CBC 从第二组开始将上一组的加密密文作为 IV 来与明文异或。至此我们确定了加密模式使用的是 CBC 并且我们找到的 IV 是正确的。现在我们需要知道我们的密钥是不是正确的。

我们回到这张图：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcRRWoMT0ocibztGdicicRIZYz9j9NM899Ho4rwRGl8vFXV702LEXMAZVfA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=21)p21

往上滚动到第一次出现四字节异或的位置：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcL4KiapmThuZCMJzM32ggARYxQ4568Bnmr3dOjWZSV7cgpepnbJibWFFg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=22)p22

我们看到开头三条指令将我们的密钥前四个字节加载了出来，而后将密钥前四个字节与 0xf1892131 异或后得到 0xc4eb1057 。现在我们假设 0xc4eb1057 便是我们第一轮的第一组密钥，如果我们能在加密函数开头部分找到 0xc4eb1057 这个四字节与 a1 异或了，那么便证明这是第一轮的密钥。现在我们回到加密函数的开头指令流部分：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODckrnmQMnTFUwicmUfIGbDl7401u97Bsa4tpJY2N6iagnmhbBQWyJfoZTQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=23)p23

我们可以看到从函数地址 0xa5114 开始部分不到 50 行便找到了我们假设的密钥与明文进行了异或，并且我们可以看到 0x16b6a170 正是我们的初始 IV 与明文异或后的中间明文。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcIT2qtBMR0jPXqvXD1ujZOYXicn3vCwdO9NgZZTsSISyrNYxsQvvibictA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=24)p24

至此我们可以确定我们的密钥是正确的。现在我们拿到了正确的初始密钥就可以开始还原厂商魔改的密钥扩展算法。

再回到这张图：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcL4KiapmThuZCMJzM32ggARYxQ4568Bnmr3dOjWZSV7cgpepnbJibWFFg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=25)p25

我们发现第一轮第一组密钥是通过初始密钥 0x35623166  与 0xf1892131 异或得到的，因此我需要知道 0xf1892131 是如何而来的。我们还是在指令流中搜索这串数据：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc7XiasWWZydGrOPajJH5CRem6JWujR8tQZE9HJ8zD2PbjQRCt2fXHmFg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=26)p26

我们可以看到这串数据通过一条位或指令得到。如果按照正常的还原流程我们还需要弄清除 X9 与 X8 的值是怎么来的，但是由于经过 VMP 加持的指令流变得非常之长，如果我们要在文章中展开怎么追溯 0xf1892131 生成位置的话，那么整个文章的结构与篇幅都将变得冗长、混乱不可读。因此，我在这里直接将与初始密钥异或的常量表给贴出来了。并且因为它是固定不变的，所以也不影响我们后续的阅读体验。

```
key_table = [0xf1892131, 0xff001123, 0xf1001356, 0xf1234890]

```

现在我们可以整理出与初始密钥异或后的 K00 轮密钥：

c4 eb 10 57 9d 35 75 15 dc 36 75 32 97 0e 7c f2

得到 K00 轮密钥之后我们需要知道 K01 轮密钥是怎么生成的，我们只要搞清楚 K01 轮密钥的生成规则，就能按照相同的规则扩展出其余轮的所有密钥。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcRentHjrEHZLFCYujnOswsGscGNNRezHHibF013varVGDocLumvGwkfw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=27)p27

通过这张图可以看到红色方框内标志着 K00 轮密钥生成的结束，蓝色方框内标志着 K01 轮密钥生成的开始。通过观察我们发现，K01 轮密钥的生成已不再是通过三条指令加载出初始密钥将其与一个常量表异或了。而是通过四条指令加载出 K00 轮的第一组密钥，而其余三组的密钥则由上一组密钥与上一轮中的组密钥异或得到。下面，我们可以回顾下 AES 的标准密钥扩展算法得知到底是哪部分被魔改了。我们看看下面这两张图：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcyBBLic6PnqIhLyXC3V1yb86OAj64vxjbGlILVk4KUlgPsKJkhL88I7A/640?wx_fmt=png&from=appmsg#imgIndex=28)p28![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODc3qVU8aiaf6vUX3nRTJ7TXgEMg4hLFkTRa0HbmJ5koJzvLenBAjpedqQ/640?wx_fmt=png&from=appmsg#imgIndex=29)p29

这里的 W 便代表我们所说的组，n 则等于（轮 - 1）* 4 + 4 = n。第一张图代表了每轮的第一组密钥生成规则，第二张图代表了每轮的其余三组生成规则。通过对比这两张图我们发现，第一组与其余组生成的规则多了个 g 函数。我们再观察上面的指令流得知，K01 轮的第二组的密钥是通过：

0x7dca99df ^ 0x9d357515 = 0xe0ffecca 得到，我们将这组数据代入我们的图 2 公式：

n = 00 * 4 + 4 = 4。而因为我们计算的是 K01 轮的第二组密钥，因此我们需要加上 1 。所以 n = 5。

W(5) = W(4) xor W(1)

W(4) = 0x7dca99df

W(1) = 0x9d357515

W(5) = 0x7dca99df xor 0x9d357515

W(5) = 0xe0ffecca

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcalNbW8cBPdCy6ib0B8rpibHAYyia77hibp4RibuXh3almKTHcY1AmmGEB9w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=30)p30

可以看到我们最后计算出来的值与在指令流中的值是一样的。因此我们可以断定，除了第一组密钥生成规则被魔改之外，其余组均未被魔改。所以接下来我们主要关注第一组密钥的生成。

我们通过对比知道第一组密钥的生成多了个 g 函数，因此我们要从汇编中将 g 函数的实现扣下来。现在我们回到 K01 轮第一组密钥的指令流中，我们将其生成的过程指令拿下来：

```
eor x8, x8, x2 ; x8=0xab000000 x8=0xab000000 x2=0xc4eb1057 -> x8=0x6feb1057
eor x8, x8, x2 ; x8=0x100000 x8=0x100000 x2=0x6feb1057 -> x8=0x6ffb1057
eor x8, x8, x2 ; x8=0x8900 x8=0x8900 x2=0x6ffb1057 -> x8=0x6ffb9957
eor x8, x8, x2 ; x8=0x88 x8=0x88 x2=0x6ffb9957 -> x8=0x6ffb99df
eor x8, x8, x2 ; x8=0x12310000 x8=0x12310000 x2=0x6ffb99df -> x8=0x7dca99df

```

我们知道其中 0xc4eb1057 是 K00 轮中的第一组密钥，代入我们的图一公式可知：

W(4) = g(W(3)) xor W(0)

W3 = 0x970e7cf2

W0 = 0xc4eb1057

在这段指令流中我们看到 W0 的出现却没有发现 W3 的出现，并且多出了一串我们未曾见过的数据：

0xab000000 ^ 0x100000 ^ 0x8900 ^ 0x88 = 0xab108988

那么我们是不是可以假设 W3 被替换成了这串数据？那么这串数据怎么生成的就成了我们分析的关键。下面这部分指令是我从上万条指令中肉眼筛查出来的关键指令，这 6 条指令最终生成了 0xab000000 数据。

```
asr x8, x2, x20 ; x8=0x6dd9fecfb0 x2=0x970e7cf2 x20=0x10 -> x8=0x970e
and x8, x8, x2 ; x8=0xff x8=0xff x2=0x970e -> x8=0xe 
mul x23, x10, x9 ; x23=0xb400006dd01ca740 x10=0xe x9=0x4 -> x23=0x38
add x2, x27, x20 ; x2=0x6dd970a348 x27=0x38 x20=0x6dda0a3598 -> x2=0x6dda0a35d0
ldr w2, [x23] ; w2=0xe x23=0x6dda0a35d0 mem_r=0x6dda0a35d0 -> w2=0xabe64dab
and x8, x8, x2 ; x8=0xff000000 x8=0xff000000 x2=0xabe64dab -> x8=0xab000000

```

我们可以将这段汇编指令还原成伪 C 代码：

```
W3 = 0x970e7cf2;
bword = W3 >> 16 & 0xff
offset = bword * 4
table_index = 0x6dda0a3598 + offset
data = table_index & 0xff000000
// data = 0xab000000

```

0x6dda0a3598 是我们的表基址，表基址加上偏移得到 0xabe64dab，再取 0xabe64dab 的高 8 位得到 0xab000000。偏移来自于 W3 的中高 8 位 * 4。因此，拿到表数据是我们还原的关键。那么怎么拿到表数据呢？我们可以通过指令的绝对地址减去相对地址得到 SO 的加载基址，再用表基址减去 SO 基址得到表在 SO 中的相对偏移。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcHNO8uCcV4daiblESgzmYzribcmEY4sibysb1sZtmHTc59HPWG1MPjxPmw/640?wx_fmt=png&from=appmsg#imgIndex=31)p31

0x6dda0a3598 - (0x6dd9feb83c - 0x2f83c) = 0xE7598。

我们在 IDA 中跳转到目标地址看看：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcibxWMh4brceM3t4U0Eh7AkfM4vSPW2RqwsmN1pgBhhKndt3DtxJzB2g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=32)p32

我们发现这个表正是我们在加密函数中看到的 table_3，并且可以看到表数据分别是 4 字节一组，对应我们的 offset = bword * 4 这行代码。至此我们可以将 W3 中高 8 位替换函数定义为：

```
def replace_key_middle_high_8(group_key):
  return table_3[(group_key >> 16 & 0xff) * 4] && 0xff000000

```

现在我们知道了 W3 的中高 8 位替换公式，需要验证其他三个字节是不是也是相同的公式。这里我就不将指令流片段贴出来了，免得冗长。经过分析得到其余三个字节的替换公式：

```
def replace_key_middle_low_8(group_key):
  return table_4[(group_key >> 8 & 0xff) * 4] && 0xff0000

```

```
def replace_key_low_8(group_key):
  return table_1[(group_key >> 0 & 0xff) * 4] && 0xff00

```

```
def replace_key_high_8(group_key):
  return table_2[(group_key >> -8 & 0xff) * 4] && 0xff

```

我们通过观察这四个函数可以看到其具有一定的规律性，因此我们可以将其合并成一个函数：

```
def replace_key(group_key):
    after_group_key = 0x0
    shift_index = 16
    offset_index = 0
    tables = [table_3, table_4, table_1, table_2]
    for i in range(4):
        find_table = tables[i]
        if shift_index < 0:
            shift_index = 24
        table_index = group_key >> shift_index & 0xff
        after_group_key ^= find_table[table_index] & (0xff000000 >> offset_index)
        shift_index -= 8
        offset_index += 8
    return after_group_key

```

现在我们拿到了 W3 的替换函数，就相当于完成了 0xab108988 ^ 0xc4eb1057 这一步，但是我们看到第一组密钥还与 0x12310000 进行异或才是我们最终的第一组密钥值。因此我们需要知道 0x12310000 是怎么生成的。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcHNO8uCcV4daiblESgzmYzribcmEY4sibysb1sZtmHTc59HPWG1MPjxPmw/640?wx_fmt=png&from=appmsg#imgIndex=33)p33

这里又经过了一番追溯，最终定位到这也是一个常量表中的值。因为这是固定的常量表，不影响我们后续的分析，所以就不写追溯过程了。

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcic7RxQAwKb3YnSMvuyFziaw68icAuOXIvhGk3yhdHys3onAlRj3OrEcibA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=34)

我们将这个常量表定义为 g_table：

```
g_table = [0x12310000, 0x02000100, 0x04020000, 0x08020200, 0x10102000, 0x30020400, 0x40002000, 0x80002000, 0x1B002000, 0x36200200]

```

接着我们观察这个常量表，发现长度是 10，恰好密钥扩展的轮数也是 10，那么是不是每一个元素对应每一轮扩展密钥的异或值？我们可以先这么假设将这个表融入到密钥扩展中，看看最终的生成出来的密钥是不是与指令流中的一致。

现在我们整理一下我们已知的所有信息：

> 初始密钥：35 62 31 66 62 35 64 36 2d 36 66 64 66 2d 34 62

> IV：63 39 31 35 38 33 35 32 2d 61 30 30 37 2d 34 64

> key_table、g_table、table_1、table_2、table_3、table_4

还有我们的替换函数 replace_key 与 密钥生成流程，所以我们现在可以根据这些已知的信息编写魔改后的密钥扩展算法：

```
table_3 = []

table_4 = []

table_1 = []

table_2 = []

key_table = [0xf1892131, 0xff001123, 0xf1001356, 0xf1234890]

g_table = [0x12310000, 0x02000100, 0x04020000, 0x08020200, 0x10102000, 0x30020400, 0x40002000, 0x80002000, 0x1B002000,
           0x36200200]

def group_print(input):
    for i in range(len(input) // 4):
        print(hex(input[i * 4]), hex(input[i * 4 + 1]), hex(input[i * 4 + 2]), hex(input[i * 4 + 3]))

def replace_key(group_key):
    after_group_key = 0x0
    shift_index = 16
    offset_index = 0
    tables = [table_3, table_4, table_1, table_2]
    for i in range(4):
        find_table = tables[i]
        if shift_index < 0:
            shift_index = 24
        table_index = group_key >> shift_index & 0xff
        after_group_key ^= find_table[table_index] & (0xff000000 >> offset_index)
        shift_index -= 8
        offset_index += 8
    return after_group_key


def g(group_key, round):
    return replace_key(group_key) ^ g_table[round]


def key_expansion(key):
    keys = []
    # 转为 4*4 矩阵
    for i in range(4):
        group_key = key[i * 4: i * 4 + 4]
        keys.append((group_key[0] << 24 ^ group_key[1] << 16 ^ group_key[2] << 8 ^ group_key[3]) ^ key_table[i])

    # 扩展 10 轮密钥
    for i in range(10):
        for j in range(4):
            if j == 0:
                # 第一组密钥生成
                keys.append(g(keys[-1], i) ^ keys[-4])
            else:
                # 其余组密钥生成
                keys.append(keys[-1] ^ keys[-4])
    return keys

key = [0x35, 0x62, 0x31, 0x66, 0x62, 0x35, 0x64, 0x36, 0x2d, 0x36, 0x66, 0x64, 0x66, 0x2d, 0x34, 0x62]
keys = key_expansion(key)
group_print(keys)


```

上面是我将 4 个表数据抽空后的魔改密钥扩展算法。运行后得到如下输出：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcC4K8FT0mXe5VzvTg9hU6kdlf6ClDsf43SedoywlAGhAiaBgJFfYVzicA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=35)p34

我们随机挑选两组密钥在指令流中搜索：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcOWqkdcPhTwWibl93BM4GZ46iasTibKvhaloSAx7fxtb6Sf47UBA2Tj1Jg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=36)![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhUOs1XFmWHvglk4QBiaQODcmZRXaqLjicWk8p22qiaQOcxeyhABAupRLibYZU8C78EicrrpUE9NeAxwuw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=37)

我们看到两组密钥都能搜到，证明我们的魔改密钥扩展算法还原成功了。

### 轮加密还原

现在我们拿到了每一轮的密钥，并且我们已知这是个查表实现的 AES 128。那么我们只需要跟踪加密函数的汇编指令，将其中第一轮的轮加密过程给还原出来，就能通过相同的规则跑完剩下的 9 轮。我们将函数开头部分汇编指令拿下来

```
# input： 
16 b6 a1 70 28 74 43 ce aa 18 fb ba 3e 70 22 5b

# output:
07 51 ac 3f 32 85 19 6c 5a 31 4a 2f 66 58 eb d2

# keys:
0xc4eb1057 0x9d357515 0xdc367532 0x970e7cf2
0x7dca99df 0xe0ffecca 0x3cc999f8 0xabc7e50a
0xb913ffbd 0x59ec1377 0x65258a8f 0xcee26f85
0x25b96836 0x7c557b41 0x1970f1ce 0xd7929e4b
0x62b0d938 0x1ee5a279 0x79553b7 0xd007cdfc
0xb71d4948 0xa9f8eb31 0xae6db886 0x7e6a757a
0x858297bb 0x2c7a7c8a 0x8217c40c 0xfc7db176    
0x3a4a8f0b 0x1630f381 0x9427378d 0x685a86fb
0x40ea04e 0x123e53cf 0x86196442 0xee43e2b9
0x596d666 0x17a885a9 0x91b1e1eb 0x7ff20352
0xbacdd4b4 0xad65511d 0x3cd4b0f6 0x4326b3a4

add_round_key 1 start

ldrb w4, [x0] ; w4=0xda0a3198 x0=0xb400006dd014708c mem_r=0xb400006dd014708c -> w4=0x16 
uxtb w4, w4 ; w4=0x16 w4=0x16 -> w4=0x16 
lsl w4, w4, w8 ; w4=0x16 w4=0x16 w8=0x18 -> w4=0x16000000 
ldr x0, [sp, #0x68] ; x0=0xb400006dd014708c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
ldrb w5, [x0, #1] ; w5=0x76 x0=0xb400006dd014708c mem_r=0xb400006dd014708d -> w5=0xb6 
uxtb w5, w5 ; w5=0xb6 w5=0xb6 -> w5=0xb6 
lsl w5, w5, w9 ; w5=0xb6 w5=0xb6 w9=0x10 -> w5=0xb60000 
eor w4, w4, w5 ; w4=0x16000000 w4=0x16000000 w5=0xb60000 -> w4=0x16b60000 
ldr x0, [sp, #0x68] ; x0=0xb400006dd014708c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
ldrb w5, [x0, #2] ; w5=0xb60000 x0=0xb400006dd014708c mem_r=0xb400006dd014708e -> w5=0xa1 
uxtb w5, w5 ; w5=0xa1 w5=0xa1 -> w5=0xa1 
lsl w5, w5, w10 ; w5=0xa1 w5=0xa1 w10=0x8 -> w5=0xa100 
eor w4, w4, w5 ; w4=0x16b60000 w4=0x16b60000 w5=0xa100 -> w4=0x16b6a100 
ldr x0, [sp, #0x68] ; x0=0xb400006dd014708c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
ldrb w5, [x0, #3] ; w5=0xa100 x0=0xb400006dd014708c mem_r=0xb400006dd014708f -> w5=0x70 
uxtb w5, w5 ; w5=0x70 w5=0x70 -> w5=0x70 
eor w4, w4, w5 ; w4=0x16b6a100 w4=0x16b6a100 w5=0x70 -> w4=0x16b6a170 
ldr x0, [sp, #0x50] ; x0=0xb400006dd014708c sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x0=0xb400006dd014709c 
ldr w5, [x0] ; w5=0x70 x0=0xb400006dd014709c mem_r=0xb400006dd014709c -> w5=0xc4eb1057 
eor w4, w4, w5 ; w4=0x16b6a170 w4=0x16b6a170 w5=0xc4eb1057 -> w4=0xd25db127 

state[0] ^ keys[0] = state[0] = 0xd25db127

str w4, [sp, #0x4c] ; w4=0xd25db127 sp=0x6dd970a650 mem_w=0x6dd970a69c 
ldr x0, [sp, #0x68] ; x0=0xb400006dd014709c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
add x0, x0, x11 ; x0=0xb400006dd014708c x0=0xb400006dd014708c x11=0x4 -> x0=0xb400006dd0147090 
ldrb w4, [x0] ; w4=0xd25db127 x0=0xb400006dd0147090 mem_r=0xb400006dd0147090 -> w4=0x28 
uxtb w4, w4 ; w4=0x28 w4=0x28 -> w4=0x28 
lsl w4, w4, w8 ; w4=0x28 w4=0x28 w8=0x18 -> w4=0x28000000 
ldr x0, [sp, #0x68] ; x0=0xb400006dd0147090 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
add x0, x0, x11 ; x0=0xb400006dd014708c x0=0xb400006dd014708c x11=0x4 -> x0=0xb400006dd0147090 
ldrb w5, [x0, #1] ; w5=0xc4eb1057 x0=0xb400006dd0147090 mem_r=0xb400006dd0147091 -> w5=0x74 
uxtb w5, w5 ; w5=0x74 w5=0x74 -> w5=0x74 
lsl w5, w5, w9 ; w5=0x74 w5=0x74 w9=0x10 -> w5=0x740000 
eor w4, w4, w5 ; w4=0x28000000 w4=0x28000000 w5=0x740000 -> w4=0x28740000 
ldr x0, [sp, #0x68] ; x0=0xb400006dd0147090 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
add x0, x0, x11 ; x0=0xb400006dd014708c x0=0xb400006dd014708c x11=0x4 -> x0=0xb400006dd0147090 
ldrb w5, [x0, #2] ; w5=0x740000 x0=0xb400006dd0147090 mem_r=0xb400006dd0147092 -> w5=0x43 
uxtb w5, w5 ; w5=0x43 w5=0x43 -> w5=0x43 
lsl w5, w5, w10 ; w5=0x43 w5=0x43 w10=0x8 -> w5=0x4300 
eor w4, w4, w5 ; w4=0x28740000 w4=0x28740000 w5=0x4300 -> w4=0x28744300 
ldr x0, [sp, #0x68] ; x0=0xb400006dd0147090 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x0=0xb400006dd014708c 
add x11, x0, x11 ; x11=0x4 x0=0xb400006dd014708c x11=0x4 -> x11=0xb400006dd0147090 
ldrb w5, [x11, #3] ; w5=0x4300 x11=0xb400006dd0147090 mem_r=0xb400006dd0147093 -> w5=0xce 
uxtb w5, w5 ; w5=0xce w5=0xce -> w5=0xce 
eor w4, w4, w5 ; w4=0x28744300 w4=0x28744300 w5=0xce -> w4=0x287443ce 
ldr x11, [sp, #0x50] ; x11=0xb400006dd0147090 sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x11=0xb400006dd014709c 
ldr w5, [x11, #4] ; w5=0xce x11=0xb400006dd014709c mem_r=0xb400006dd01470a0 -> w5=0x9d357515 
eor w4, w4, w5 ; w4=0x287443ce w4=0x287443ce w5=0x9d357515 -> w4=0xb54136db 

state[1] ^ keys[1] = state[1] = 0xb54136db

str w4, [sp, #0x48] ; w4=0xb54136db sp=0x6dd970a650 mem_w=0x6dd970a698 
ldr x11, [sp, #0x68] ; x11=0xb400006dd014709c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x12 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x12=0x8 -> x11=0xb400006dd0147094 
ldrb w4, [x11] ; w4=0xb54136db x11=0xb400006dd0147094 mem_r=0xb400006dd0147094 -> w4=0xaa 
uxtb w4, w4 ; w4=0xaa w4=0xaa -> w4=0xaa 
lsl w4, w4, w8 ; w4=0xaa w4=0xaa w8=0x18 -> w4=0xaa000000 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147094 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x12 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x12=0x8 -> x11=0xb400006dd0147094 
ldrb w5, [x11, #1] ; w5=0x9d357515 x11=0xb400006dd0147094 mem_r=0xb400006dd0147095 -> w5=0x18 
uxtb w5, w5 ; w5=0x18 w5=0x18 -> w5=0x18 
lsl w5, w5, w9 ; w5=0x18 w5=0x18 w9=0x10 -> w5=0x180000 
eor w4, w4, w5 ; w4=0xaa000000 w4=0xaa000000 w5=0x180000 -> w4=0xaa180000 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147094 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x12 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x12=0x8 -> x11=0xb400006dd0147094 
ldrb w5, [x11, #2] ; w5=0x180000 x11=0xb400006dd0147094 mem_r=0xb400006dd0147096 -> w5=0xfb 
uxtb w5, w5 ; w5=0xfb w5=0xfb -> w5=0xfb 
lsl w5, w5, w10 ; w5=0xfb w5=0xfb w10=0x8 -> w5=0xfb00 
eor w4, w4, w5 ; w4=0xaa180000 w4=0xaa180000 w5=0xfb00 -> w4=0xaa18fb00 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147094 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x12 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x12=0x8 -> x11=0xb400006dd0147094 
ldrb w5, [x11, #3] ; w5=0xfb00 x11=0xb400006dd0147094 mem_r=0xb400006dd0147097 -> w5=0xba 
uxtb w5, w5 ; w5=0xba w5=0xba -> w5=0xba 
eor w4, w4, w5 ; w4=0xaa18fb00 w4=0xaa18fb00 w5=0xba -> w4=0xaa18fbba 
ldr x11, [sp, #0x50] ; x11=0xb400006dd0147094 sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x11=0xb400006dd014709c 
ldr w5, [x11, #8] ; w5=0xba x11=0xb400006dd014709c mem_r=0xb400006dd01470a4 -> w5=0xdc367532 
eor w4, w4, w5 ; w4=0xaa18fbba w4=0xaa18fbba w5=0xdc367532 -> w4=0x762e8e88 

state[2] ^ keys[2] = state[2] = 0x762e8e88

str w4, [sp, #0x44] ; w4=0x762e8e88 sp=0x6dd970a650 mem_w=0x6dd970a694 
ldr x11, [sp, #0x68] ; x11=0xb400006dd014709c sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x13 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x13=0xc -> x11=0xb400006dd0147098 
ldrb w4, [x11] ; w4=0x762e8e88 x11=0xb400006dd0147098 mem_r=0xb400006dd0147098 -> w4=0x3e 
uxtb w4, w4 ; w4=0x3e w4=0x3e -> w4=0x3e 
lsl w8, w4, w8 ; w8=0x18 w4=0x3e w8=0x18 -> w8=0x3e000000 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147098 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x13 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x13=0xc -> x11=0xb400006dd0147098 
ldrb w4, [x11, #1] ; w4=0x3e x11=0xb400006dd0147098 mem_r=0xb400006dd0147099 -> w4=0x70 
uxtb w4, w4 ; w4=0x70 w4=0x70 -> w4=0x70 
lsl w9, w4, w9 ; w9=0x10 w4=0x70 w9=0x10 -> w9=0x700000 
eor w8, w8, w9 ; w8=0x3e000000 w8=0x3e000000 w9=0x700000 -> w8=0x3e700000 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147098 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x13 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x13=0xc -> x11=0xb400006dd0147098 
ldrb w9, [x11, #2] ; w9=0x700000 x11=0xb400006dd0147098 mem_r=0xb400006dd014709a -> w9=0x22 
uxtb w9, w9 ; w9=0x22 w9=0x22 -> w9=0x22 
lsl w9, w9, w10 ; w9=0x22 w9=0x22 w10=0x8 -> w9=0x2200 
eor w8, w8, w9 ; w8=0x3e700000 w8=0x3e700000 w9=0x2200 -> w8=0x3e702200 
ldr x11, [sp, #0x68] ; x11=0xb400006dd0147098 sp=0x6dd970a650 mem_r=0x6dd970a6b8 -> x11=0xb400006dd014708c 
add x11, x11, x13 ; x11=0xb400006dd014708c x11=0xb400006dd014708c x13=0xc -> x11=0xb400006dd0147098 
ldrb w9, [x11, #3] ; w9=0x2200 x11=0xb400006dd0147098 mem_r=0xb400006dd014709b -> w9=0x5b 
uxtb w9, w9 ; w9=0x5b w9=0x5b -> w9=0x5b 
eor w8, w8, w9 ; w8=0x3e702200 w8=0x3e702200 w9=0x5b -> w8=0x3e70225b 
ldr x11, [sp, #0x50] ; x11=0xb400006dd0147098 sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x11=0xb400006dd014709c 
ldr w9, [x11, #0xc] ; w9=0x5b x11=0xb400006dd014709c mem_r=0xb400006dd01470a8 -> w9=0x970e7cf2 
eor w8, w8, w9 ; w8=0x3e70225b w8=0x3e70225b w9=0x970e7cf2 -> w8=0xa97e5ea9 

state[3] ^ keys[3] = state[3] = 0xa97e5ea9
add_round_key 1 end

str w8, [sp, #0x40] ; w8=0xa97e5ea9 sp=0x6dd970a650 mem_w=0x6dd970a690 
ldr x11, [sp, #0x58] ; x11=0xb400006dd014709c sp=0x6dd970a650 mem_r=0x6dd970a6a8 -> x11=0xb400006dd014709c 
ldr w8, [x11, #0xf0] ; w8=0xa97e5ea9 x11=0xb400006dd014709c mem_r=0xb400006dd014718c -> w8=0xa 
asr w8, w8, w14 ; w8=0xa w8=0xa w14=0x1 -> w8=0x5 
str w8, [sp, #0x2c] ; w8=0x5 sp=0x6dd970a650 mem_w=0x6dd970a67c 
str w15, [sp, #0x28] ; w15=0x560114ef sp=0x6dd970a650 mem_w=0x6dd970a678 
str x3, [sp, #0x20] ; x3=0x6dda0a3198 sp=0x6dd970a650 mem_w=0x6dd970a670 
str x18, [sp, #0x18] ; x18=0x6dda0a2d98 sp=0x6dd970a650 mem_w=0x6dd970a668 
str x17, [sp, #0x10] ; x17=0x6dda0a3998 sp=0x6dd970a650 mem_w=0x6dd970a660 
str x16, [sp, #8] ; x16=0x6dda0a3598 sp=0x6dd970a650 mem_w=0x6dd970a658 
mov w8, #0x14ef ; w8=0x5 -> w8=0x14ef 
movk w8, #0x5601, lsl #16 ; w8=0x14ef -> w8=0x560114ef 
ldr w9, [sp, #0x28] ; w9=0x970e7cf2 sp=0x6dd970a650 mem_r=0x6dd970a678 -> w9=0x560114ef 
cmp w8, w9 ; w8=0x560114ef w9=0x560114ef -> w8=0x560114ef 
cset w8, eq ; w8=0x560114ef -> w8=0x1 
str w9, [sp, #4] ; w9=0x560114ef sp=0x6dd970a650 mem_w=0x6dd970a654 
tbnz w8, #0, #0x6dda0613a0 ; w8=0x1 
mov w8, #0 ; w8=0x1 -> w8=0x0 
mov w9, #-1 ; w9=0x560114ef -> w9=0xffffffff 
mov x10, #0x20 ; x10=0x8 -> x10=0x20 
mov w11, #0xff ; w11=0xd014709c -> w11=0xff 
mov w12, #0x8222 ; w12=0x8 -> w12=0x8222 
movk w12, #0x4de6, lsl #16 ; w12=0x8222 -> w12=0x4de68222 
mov w13, #0x5828 ; w13=0xc -> w13=0x5828 
movk w13, #0xb0a8, lsl #16 ; w13=0x5828 -> w13=0xb0a85828 
mov x14, #4 ; x14=0x1 -> x14=0x4 
mov w15, #8 ; w15=0x560114ef -> w15=0x8 
mov w16, #0x10 ; w16=0xda0a3598 -> w16=0x10 
mov w17, #0x18 ; w17=0xda0a3998 -> w17=0x18 

round 1 start
find table 1 start

ldr w18, [sp, #0x4c] ; w18=0xda0a2d98 sp=0x6dd970a650 mem_r=0x6dd970a69c -> w18=0xd25db127 
lsr w18, w18, w17 ; w18=0xd25db127 w18=0xd25db127 w17=0x18 -> w18=0xd2 
mov w0, w18 ; w0=0xd014708c w18=0xd2 -> w0=0xd2 
ubfx x0, x0, #0, #0x20 ; x0=0xd2 x0=0xd2 -> x0=0xd2 
mul x0, x14, x0 ; x0=0xd2 x14=0x4 x0=0xd2 -> x0=0x348 
ldr x1, [sp, #0x18] ; x1=0xb400006dd01471f0 sp=0x6dd970a650 mem_r=0x6dd970a668 -> x1=0x6dda0a2d98 
add x0, x1, x0 ; x0=0x348 x1=0x6dda0a2d98 x0=0x348 -> x0=0x6dda0a30e0 
ldr w18, [x0] ; w18=0xd2 x0=0x6dda0a30e0 mem_r=0x6dda0a30e0 -> w18=0x71b5b5c4 

offset = state[0] >> 24 & 0xff
index = table_offset * 4
table_1[index] = table_value1 = 0x71b5b5c4

ldr w2, [sp, #0x48] ; w2=0xd014709c sp=0x6dd970a650 mem_r=0x6dd970a698 -> w2=0xb54136db 
lsr w2, w2, w16 ; w2=0xb54136db w2=0xb54136db w16=0x10 -> w2=0xb541 
and w2, w2, w11 ; w2=0xb541 w2=0xb541 w11=0xff -> w2=0x41 
mov w0, w2 ; w0=0xda0a30e0 w2=0x41 -> w0=0x41 
ubfx x0, x0, #0, #0x20 ; x0=0x41 x0=0x41 -> x0=0x41 
mul x0, x14, x0 ; x0=0x41 x14=0x4 x0=0x41 -> x0=0x104 
ldr x3, [sp, #0x20] ; x3=0x6dda0a3198 sp=0x6dd970a650 mem_r=0x6dd970a670 -> x3=0x6dda0a3198 
add x0, x3, x0 ; x0=0x104 x3=0x6dda0a3198 x0=0x104 -> x0=0x6dda0a329c 
ldr w2, [x0] ; w2=0x41 x0=0x6dda0a329c mem_r=0x6dda0a329c -> w2=0x9e1d8383 

offset = state[1] >> 16 & 0xff
index = table_offset * 4
table_2[index] = table_value2 = 0x9e1d8383

eor w18, w18, w2 ; w18=0x71b5b5c4 w18=0x71b5b5c4 w2=0x9e1d8383 -> w18=0xefa83647

table_value1 ^= table_value2

ldr w2, [sp, #0x44] ; w2=0x9e1d8383 sp=0x6dd970a650 mem_r=0x6dd970a694 -> w2=0x762e8e88 
lsr w2, w2, w15 ; w2=0x762e8e88 w2=0x762e8e88 w15=0x8 -> w2=0x762e8e 
and w2, w2, w11 ; w2=0x762e8e w2=0x762e8e w11=0xff -> w2=0x8e 
mov w0, w2 ; w0=0xda0a329c w2=0x8e -> w0=0x8e 
ubfx x0, x0, #0, #0x20 ; x0=0x8e x0=0x8e -> x0=0x8e 
mul x0, x14, x0 ; x0=0x8e x14=0x4 x0=0x8e -> x0=0x238 
ldr x4, [sp, #8] ; x4=0x70 sp=0x6dd970a650 mem_r=0x6dd970a658 -> x4=0x6dda0a3598 
add x0, x4, x0 ; x0=0x238 x4=0x6dda0a3598 x0=0x238 -> x0=0x6dda0a37d0 
ldr w2, [x0] ; w2=0x8e x0=0x6dda0a37d0 mem_r=0x6dda0a37d0 -> w2=0x192b3219 

offset = state[2] >> 8 & 0xff
index = table_offset * 4
table_3[index] = table_value3 = 0x192b3219

eor w18, w18, w2 ; w18=0xefa83647 w18=0xefa83647 w2=0x192b3219 -> w18=0xf683045e 

table_value1 ^= table_value3

ldr w2, [sp, #0x40] ; w2=0x192b3219 sp=0x6dd970a650 mem_r=0x6dd970a690 -> w2=0xa97e5ea9 
and w2, w2, w11 ; w2=0xa97e5ea9 w2=0xa97e5ea9 w11=0xff -> w2=0xa9 
mov w0, w2 ; w0=0xda0a37d0 w2=0xa9 -> w0=0xa9 
ubfx x0, x0, #0, #0x20 ; x0=0xa9 x0=0xa9 -> x0=0xa9 
mul x0, x14, x0 ; x0=0xa9 x14=0x4 x0=0xa9 -> x0=0x2a4 
ldr x5, [sp, #0x10] ; x5=0xdc367532 sp=0x6dd970a650 mem_r=0x6dd970a660 -> x5=0x6dda0a3998 
add x0, x5, x0 ; x0=0x2a4 x5=0x6dda0a3998 x0=0x2a4 -> x0=0x6dda0a3c3c 
ldr w2, [x0] ; w2=0xa9 x0=0x6dda0a3c3c mem_r=0x6dda0a3c3c -> w2=0xd3d36ebd 

offset = state[3] >> 0 & 0xff
index = table_offset * 4
table_4[index] = table_value4 = 0xd3d36ebd

eor w18, w18, w2 ; w18=0xf683045e w18=0xf683045e w2=0xd3d36ebd -> w18=0x25506ae3 

table_value1 ^= table_value4

ldr x0, [sp, #0x50] ; x0=0x6dda0a3c3c sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x0=0xb400006dd014709c 
ldr w2, [x0, #0x10] ; w2=0xd3d36ebd x0=0xb400006dd014709c mem_r=0xb400006dd01470ac -> w2=0x7dca99df 
eor w18, w18, w2 ; w18=0x25506ae3 w18=0x25506ae3 w2=0x7dca99df -> w18=0x589af33c 
str w18, [sp, #0x3c] ; w18=0x589af33c sp=0x6dd970a650 mem_w=0x6dd970a68c 

keys[4] ^  table_value1 = state[0] = 0x6dd970a68c
find table 1 end

```

这是函数开头的初始轮密钥加与 state[0] 的计算过程指令流，用上下两个换行隔开是我添加的伪代码。从这部分指令流我们可以得出两个结论：

*   • add_round_key 没有被魔改
    
*   • state[0] 的计算过程与 state 中的四个元素强相关
    

现在我们可以根据指令流将 state[0] 的计算过程整理为一个函数：

```
def state_0_calc(state, group_key):
    tables = [table_1, table_2, table_3, table_4]
    shift_index = 24
    group_value = 0x00
    for i in range(4):
        table_index = state[i] >> shift_index & 0xff
        find_table = tables[i]
        table_value = find_table[table_index]
        if i == 0:
            group_value = table_value
        else:
            group_value ^= table_value
        shift_index -= 8

    return state[0] ^ group_key

```

接着我又顺着指令流往下分析，得到了 state 中其余三个元素的函数：

```
def state_1_calc(state, group_key):
    tables = [table_1, table_2, table_3, table_4]
    state = state[1:] + state[:1]
    shift_index = 24
    group_value = 0x00
    for i in range(4):
        table_index = state[i] >> shift_index & 0xff
        find_table = tables[i]
        table_value = find_table[table_index]
        if i == 0:
            group_value = table_value
        else:
            group_value ^= table_value
        shift_index -= 8

    return state[1] ^ group_key

def state_2_calc(state, group_key):
    tables = [table_1, table_2, table_3, table_4]
    state = state[2:] + state[:2]
    shift_index = 24
    group_value = 0x00
    for i in range(4):
        table_index = state[i] >> shift_index & 0xff
        find_table = tables[i]
        table_value = find_table[table_index]
        if i == 0:
            group_value = table_value
        else:
            group_value ^= table_value
        shift_index -= 8

    return state[2] ^ group_key

def state_3_calc(state, group_key):
    tables = [table_1, table_2, table_3, table_4]
    state = state[3:] + state[:3]
    shift_index = 24
    group_value = 0x00
    for i in range(4):
        table_index = state[i] >> shift_index & 0xff
        find_table = tables[i]
        table_value = find_table[table_index]
        if i == 0:
            group_value = table_value
        else:
            group_value ^= table_value
        shift_index -= 8

    return state[3] ^ group_key

```

我们观察这四个函数得知其具有一定规律性，计算的是哪个元素就用元素对应的下标做为查表的第一个元素。熟悉 AES 的朋友可能已经知道，这是在循环左移 (shift_rows) 。所以我们可以将其合并为一个函数：

```
def state_calc(state, keys):
    tables = [table_1, table_2, table_3, table_4]
    # 深拷贝
    copy_state = state[:4]
    for i in range(4):
        group_value = 0x00
        shift_index = 24
        for j in range(4):
            table_index = copy_state[j] >> shift_index & 0xff # 因为我们整理出来的表每个元素是四字节，因此我们不需要 * 4
            find_table = tables[j]
            table_value = find_table[table_index]
            if j == 0:
                group_value = table_value
            else:
                group_value ^= table_value
            shift_index -= 8
        state[i] = group_value ^ keys[i]
        #循环左移
        copy_state = copy_state[1:] + copy_state[:1]
    return state

```

现在我们将明文与密钥结合起来跑一遍，得到如下输出：  

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8s97HUiamtZRWI0iaecl2tUVvel7zR8X3HQGUV884L5phfR43WlGU88Lg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=38)p35

我们发现除了第十轮的密文不能在指令流中搜到，其余轮的密文都能搜到：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8B1g8c9csich2oO1pZGIXrnbAypKbFDPiaB59GD6TH3ibMoCx12xJTibryA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=39)p36![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8dJLCKbISJJDIian5qRFlWhofSZUocp6ICFX6DApr5Hp5G3zN1GCAatA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=40)p37

那也就是说第十轮的加密与我们整理出来的函数有出入。我们不妨回忆下标准 AES 的最后一轮前面的每轮有什么区别？是的，最后一轮没有列混淆。因此我们需要再次回到指令流中，将最后一轮的轮加密过程抠出来：

```
# input： 
0xcd8a142c 0x6e1c5ee5 0xd3a0c10f 0xc22ad4ce  # 最后一轮的输入为第九轮的密文

# output:
07 51 ac 3f 32 85 19 6c 5a 31 4a 2f 66 58 eb d2

# keys:
0xbacdd4b4 0xad65511d 0x3cd4b0f6 0x4326b3a4

ldr w18, [sp, #0x3c] ; w18=0xe59700da sp=0x6dd970a650 mem_r=0x6dd970a68c -> w18=0xcd8a142c 
lsr w18, w18, w17 ; w18=0xcd8a142c w18=0xcd8a142c w17=0x18 -> w18=0xcd 
mov w0, w18 ; w0=0x96 w18=0xcd -> w0=0xcd 
ubfx x0, x0, #0, #0x20 ; x0=0xcd x0=0xcd -> x0=0xcd 
mul x0, x14, x0 ; x0=0xcd x14=0x4 x0=0xcd -> x0=0x334 
ldr x1, [sp, #8] ; x1=0x6dda0a2d98 sp=0x6dd970a650 mem_r=0x6dd970a658 -> x1=0x6dda0a3598 
add x0, x1, x0 ; x0=0x334 x1=0x6dda0a3598 x0=0x334 -> x0=0x6dda0a38cc 
ldr w18, [x0] ; w18=0xcd x0=0x6dda0a38cc mem_r=0x6dda0a38cc -> w18=0xbddc61bd 

offset = state[0] >> 24 & 0xff
index = table_offset * 4
table_3[index] = table_value1 = 0xbddc61bd

0x6dda06193c!0xa593c and w18, w18, w11 ; w18=0xbddc61bd w18=0xbddc61bd w11=0xff000000 -> w18=0xbd000000 

group_value = table_value1 & 0xff000000

ldr w2, [sp, #0x38] ; w2=0x91b1e1eb sp=0x6dd970a650 mem_r=0x6dd970a688 -> w2=0x6e1c5ee5 
lsr w2, w2, w16 ; w2=0x6e1c5ee5 w2=0x6e1c5ee5 w16=0x10 -> w2=0x6e1c 
and w2, w2, w10 ; w2=0x6e1c w2=0x6e1c w10=0xff -> w2=0x1c 
mov w0, w2 ; w0=0xda0a38cc w2=0x1c -> w0=0x1c 
ubfx x0, x0, #0, #0x20 ; x0=0x1c x0=0x1c -> x0=0x1c 
mul x0, x14, x0 ; x0=0x1c x14=0x4 x0=0x1c -> x0=0x70 
ldr x3, [sp, #0x10] ; x3=0x6dda0a3198 sp=0x6dd970a650 mem_r=0x6dd970a660 -> x3=0x6dda0a3998 
add x0, x3, x0 ; x0=0x70 x3=0x6dda0a3998 x0=0x70 -> x0=0x6dda0a3a08 
ldr w2, [x0] ; w2=0x1c x0=0x6dda0a3a08 mem_r=0x6dda0a3a08 -> w2=0x9c9cbf23 

offset = state[1] >> 16 & 0xff
index = table_offset * 4
table_4[index] = table_value2 = 0x9c9cbf23


and w2, w2, w9 ; w2=0x9c9cbf23 w2=0x9c9cbf23 w9=0xff0000 -> w2=0x9c0000 
eor w18, w18, w2 ; w18=0xbd000000 w18=0xbd000000 w2=0x9c0000 -> w18=0xbd9c0000 

group_value ^= table_value2 & 0xff0000

ldr w2, [sp, #0x34] ; w2=0x9c0000 sp=0x6dd970a650 mem_r=0x6dd970a684 -> w2=0xd3a0c10f 
lsr w2, w2, w15 ; w2=0xd3a0c10f w2=0xd3a0c10f w15=0x8 -> w2=0xd3a0c1 
and w2, w2, w10 ; w2=0xd3a0c1 w2=0xd3a0c1 w10=0xff -> w2=0xc1 
mov w0, w2 ; w0=0xda0a3a08 w2=0xc1 -> w0=0xc1 
ubfx x0, x0, #0, #0x20 ; x0=0xc1 x0=0xc1 -> x0=0xc1 
mul x0, x14, x0 ; x0=0xc1 x14=0x4 x0=0xc1 -> x0=0x304 
ldr x4, [sp, #0x18] ; x4=0x6dda0a3598 sp=0x6dd970a650 mem_r=0x6dd970a668 -> x4=0x6dda0a2d98 
add x0, x4, x0 ; x0=0x304 x4=0x6dda0a2d98 x0=0x304 -> x0=0x6dda0a309c 
ldr w2, [x0] ; w2=0xc1 x0=0x6dda0a309c mem_r=0x6dda0a309c -> w2=0xf0787888 

offset = state[2] >> 8 & 0xff
index = table_offset * 4
table_1[index] = table_value3 = 0xf0787888

and w2, w2, w8 ; w2=0xf0787888 w2=0xf0787888 w8=0xff00 -> w2=0x7800 
eor w18, w18, w2 ; w18=0xbd9c0000 w18=0xbd9c0000 w2=0x7800 -> w18=0xbd9c7800 

group_value ^= table_value3 & 0xff00

ldr w2, [sp, #0x30] ; w2=0x7800 sp=0x6dd970a650 mem_r=0x6dd970a680 -> w2=0xc22ad4ce 
and w2, w2, w10 ; w2=0xc22ad4ce w2=0xc22ad4ce w10=0xff -> w2=0xce 
mov w0, w2 ; w0=0xda0a309c w2=0xce -> w0=0xce 
ubfx x0, x0, #0, #0x20 ; x0=0xce x0=0xce -> x0=0xce 
mul x0, x14, x0 ; x0=0xce x14=0x4 x0=0xce -> x0=0x338 
ldr x5, [sp, #0x20] ; x5=0x6dda0a3998 sp=0x6dd970a650 mem_r=0x6dd970a670 -> x5=0x6dda0a3198 
add x0, x5, x0 ; x0=0x338 x5=0x6dda0a3198 x0=0x338 -> x0=0x6dda0a34d0 
ldr w2, [x0] ; w2=0xce x0=0x6dda0a34d0 mem_r=0x6dda0a34d0 -> w2=0x860d8b8b 

offset = state[3] >> 0 & 0xff
index = table_offset * 4
table_2[index] = table_value4 = 0x860d8b8b

and w2, w2, w10 ; w2=0x860d8b8b w2=0x860d8b8b w10=0xff -> w2=0x8b 
eor w18, w18, w2 ; w18=0xbd9c7800 w18=0xbd9c7800 w2=0x8b -> w18=0xbd9c788b 

group_value ^= table_value4 & 0xff

ldr x0, [sp, #0x50] ; x0=0x6dda0a34d0 sp=0x6dd970a650 mem_r=0x6dd970a6a0 -> x0=0xb400006dd014713c 
ldr w2, [x0] ; w2=0x8b x0=0xb400006dd014713c mem_r=0xb400006dd014713c -> w2=0xbacdd4b4 
eor w18, w18, w2 ; w18=0xbd9c788b w18=0xbd9c788b w2=0xbacdd4b4 -> w18=0x751ac3f 

state[0] = group_value ^ keys[0]

```

可以看到，最后一轮得到的 table_value 没有再与下一个元素得到的 table_value 进行异或。而是将他们从高 8 位取到低 8 位依次组成一个 32 位的 group_value 后直接与密钥异或得到我们的最终密文。接下来，我们根据指令流还原出最后一轮的加密函数：

```
def final_state_calc(state):
    tables = [table_3, table_4, table_1, table_2]
    # 深拷贝
    copy_state = state[:4]
    for i in range(4):
        group_value = 0x00
        shift_index = 24
        group_index = 0
        for j in range(4):
            table_index = copy_state[j] >> shift_index & 0xff # 因为我们整理出来的表每个元素是四字节，因此我们不需要 * 4
            find_table = tables[j]
            table_value = find_table[table_index]
            group_value ^= table_value & (0xff000000 >> group_index)
            shift_index -= 8
            group_index += 8
        state[i] = group_value
        # 循环左移
        copy_state = copy_state[1:] + copy_state[:1]

    return state

```

我们运行后得到如下输出：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8kKq720zia3VADx44TvRuuibabYpw3Ppx2OIP0vdYFKgzxYzdhMs9EYCw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=41)p38

可以看到最后一轮的数据便是我们的最终密文，那么至此我们的算法就还原成功了。

### 指纹数据

事实上，在输入 AES 加密之前厂商还将数据进行了两轮 XOR 加密。但是，我们已经还原了 AES，XOR 就相对简单多了。这里考虑到厂商隐私安全的关系，我就不在文章中展开分析了。最后，贴一下未加密前的指纹数据：

![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8ZNd6D0KNFicjibCOWexMbLwamEngKR4cWPlHK4EVP5hZczXB2OJ7NGmw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=42)p39![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8GMfZ36RjqcddiattkyMuX7ORjtwbjH6rnLr4QZIgTIUbelvibMeONHWw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=43)p40![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8UwPXZTsjGtkANZ9wG6gegfLS9Cbqtsy4wwicykKD1Fnic9xpkjAlLOAQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=44)p41![](https://mmbiz.qpic.cn/mmbiz_png/lYTWqGn01QhibichP8jt171BDR7NSGsng8FaXIlBzJIrEIrvepKiaYm9QQ7iatDPYcEwD0bib35v68GBbWLiboLcgsCg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=45)p42