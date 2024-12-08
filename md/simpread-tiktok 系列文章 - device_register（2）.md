> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZRpGNuJgm_XvBmLUlP3REw?poc_token=HKilVWejx3Z2vCSLlGtsEjwEEUNSgy5ZOKQBCOT1)

**声明：文章内容仅供学习参考，严禁用于商业用途，否则由此产生一切后果与作者无关，如有侵权请联系作者进行删除**

app 版本：30.8.4  

firda 版本：12.4.8  

ida 版本: 7.7

jadx 版本 1.5.0  

平台：andriod arm64-v8a

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3f1bq5L74aK6BUVOQPiaS2E9ibexDQo80ia62CVZVic5Kf5fFGzwzqst0N8w/640?wx_fmt=png&from=appmsg)

在首次打开该 app 时，会发现设备发送一条请求，请求回来的数据代表着当前设备的唯一标识，其中的 device_id 和 install_id 更是后面每一步请求必不可少的参数，故首先分析该条请求。

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fj8AwnwkoJr6dO4xxZJbvOgm1DxhHXsFph8782S15qzLPFq4UT0Kwmg/640?wx_fmt=png&from=appmsg)

通过解决抓包问题后，我们可以得到，由图可知，该条请求的请求体为我们所不熟知的 hex 格式。  

**So 定位**

请求体 body 加密，开 jadx 分析，全局搜索 service/2/device_register/；到达类 X.A7o，通过 hook 该类，进行堆栈跟踪。

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fCbg4UXGRzZgpPwCic4Ae0SmwGZMEibCGVjMqGdeiazqwC1lsDxslywLcQ/640?wx_fmt=png&from=appmsg)

跟踪到类：

com.bytedance.frameworks.encryptor.EncryptorUtil

对该类进行 hook，直接进行 hook frida 报错，时机问题，故设置全局变量。

```
let is_hook_EncryptorUtil = false;
function hook_EncryptorUtil() {
    Java.perform(function () {
        if (!is_hook_EncryptorUtil){
            is_hook_EncryptorUtil = true;
            let EncryptorUtil = Java.use("com.bytedance.frameworks.encryptor.EncryptorUtil");
            EncryptorUtil["ttEncrypt"].implementation = function (bArr, i2) {
                console.log(`EncryptorUtil.ttEncrypt is called: bArr=${bArr}, i2=${i2}`);
                let result = this["ttEncrypt"](bArr, i2);
                // console.log(`EncryptorUtil.ttEncrypt result=${result}`);
                console.log(`EncryptorUtil.ttEncrypt bArr=${bytesToHex(bArr)}, result=${bytesToHex(result)}`);
                printStacks("ttEncrypt")
                return result;
            };
        };
    });
}
function hook_java() {
    Java.perform(function () {
        let C24609A7o = Java.use("X.A7o");
        C24609A7o["$init"].overload('[Ljava.lang.String;', '[Ljava.lang.String;', '[Ljava.lang.String;', 'java.lang.String', 'java.lang.String', '[Ljava.lang.String;', 'java.lang.String', 'java.lang.String').implementation = function (strArr, strArr2, strArr3, str, str2, strArr4, str3, str4) {
            hook_EncryptorUtil();
            console.log(`C24609A7o.$init is called: strArr=${strArr}, strArr2=${strArr2}, strArr3=${strArr3}, str=${str}, str2=${str2}, strArr4=${strArr4}, str3=${str3}, str4=${str4}`);
            this["$init"](strArr, strArr2, strArr3, str, str2, strArr4, str3, str4);
            printStacks("C24609A7o")
        };
    });
}

```

在加载引用 service/2/device_register / 时，开启对 EncryptorUtil 的 hook；通过 hook, 可以看到 EncryptorUtil.ttEncrypt 的返回值就是 device_register 的 body。

写一个 EncryptorUtil.ttEncrypt 的主动调用，成功调用，同时在该类中，上下文发现 Librarian.LIZ("Encryptor") 处，确定并对 libEncryptor.so 进行分析 。

```
function call_ttEncrypt(){
    Java.perform(function () {
        console.log(`call_ttEncrypt start =======================================================================`);
        let EncryptorUtil = Java.use("com.bytedance.frameworks.encryptor.EncryptorUtil");
        let StrimgClass = Java.use("java.lang.String");
        let str_str = "hello, jqiu";
        let result = EncryptorUtil.ttEncrypt(StrimgClass.$new(str_str).getBytes(), str_str.length);
        console.log(`call_ttEncrypt result=${bytesToHex(result)}`);
    });
}

```

**So 分析**

在 libEncryptor.so 中 shift+F2 按字符串搜索 ttEncrypt，找到对应函数 sub_7D8C, 对其中主逻辑运算函数 sub_2BD8 进行 hook；

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3ficKRwpmTCEFL35LAvomfugtHXagu9L9cZOfRCSLEbDFA8KLkiaGNW4Rw/640?wx_fmt=png&from=appmsg)

sub_2BD8 的输入与输出， 与 ttEncrypt 的输入输出相等：

```
sub_2BD8 onEnter: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF
6fab23e050 68 65 6c 6c 6f 2c 20 6a 71 69 75 hello, jqiu
sub_2BD8 onLeave: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF
6ec6a539e0 74 63 05 10 00 00 0b 54 d7 8e 46 6f 65 b5 49 91 tc.....T..Foe.I.
6ec6a539f0 4f fe 23 82 ec c6 a5 3a a7 6f b1 41 cb 4d 69 95 O.#....:.o.A.Mi.
6ec6a53a00 ec a1 eb e3 18 f6 6e 67 bb b0 6d 65 2a 39 b8 44 ......ng..me*9.D
6ec6a53a10 76 64 33 b6 a0 75 61 9a 71 0c 94 a7 d0 2c 22 33 vd3..ua.q....,"3
6ec6a53a20 6f dd 03 98 58 7b d3 ee ee 27 e3 15 75 51 84 7f o...X{...'..uQ..
6ec6a53a30 2a 9e 80 66 c0 7f cc 6b ad cd ad cd b3 c2 25 0c *..f...k......%.
6ec6a53a40 99 ac f9 18 c8 27 9c 42 78 5b 46 0a de 1e ee 25 .....'.Bx[F....%
6ec6a53a50 b8 ad 9b d1 2f c0 ..../.
EncryptorUtil.ttEncrypt bArr=68656c6c6f2c206a716975, result=7463051000000b54d78e466f65b549916MoRnbUfHBdCarU9683KT3AsB67n92UtFca1ebe318f66e67bbb06d652a39b844766433b6a075619a710c94a7d02c22336fdd0398587bd3eeee27e3157551847f2a9e8066c07fcc6badcdadcdb3c2250c99acf918c8279c42785b460ade1eee25b8ad9bd12fc0

```

接下来 sub_2BD8 函数中调用 sub_2D2C, 函数 sub_2D2C 较为复杂，可使用多种 trace：

stalker：

https://www.cnblogs.com/c-x-a/p/17076881.html；

frida-qbdi-tracer：

https://github.com/lasting-yang/frida-qbdi-tracer；  

这里选择 frida-qbdi-tracer。  

分析结果组成，其中通过多次重放发现

```
              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
713b184600  74 63 05 10 00 00 57 28 49 37 6c e5 98 a6 50 c3  tc....W(I7l...P.
713b184610  b2 35 1e b7 22 ba 6b 58 dc 73 50 1f f0 67 d7 1f  .5..".kX.sP..g..
713b184620  e2 13 e6 b7 5c 3e 8b 4e cf 26 56 cb 9c 10 b7 08  ....\>.N.&V.....
713b184630  05 7e 43 93 63 f0 fe 3c f4 7e 57 07 59 52 81 e5  .~C.c..<.~W.YR..
713b184640  90 a6 34 82 50 d2 ce 3a e1 10 2b 11 f3 e7 63 5e  ..4.P..:..+...c^
713b184650  59 98 e8 3e 3b ea 2d 1d 6c f3 1d cc 93 63 4a fb  Y..>;.-.l....cJ.
713b184660  32 40 a1 04 9f e5 33 62 bd fd 5d c7 2a 95 4c a3  2@....3b..].*.L.
713b184670  fe 24 4a 43 6d fe 00 00 00 00 00 00 00 00 00 00  .$JCm...........
713b184680  00  
74 63 05 10 00 00  为固定值                                             .
其中 trace 日志
call libc.so!memcpy 7094e17b2c <- 6fb5ff35e0 size:20
0000: 57 28 49 37 6c e5 98 a6 50 c3 b2 35 1e b7 22 ba  W(I7l...P..5..".
0010: 6b 58 dc 73 50 1f f0 67 d7 1f e2 13 e6 b7 5c 3e  kX.sP..g......\>
往上搜索赋值处
70BE458888 [libEncryptor.so!6888]   add    x12, x10, x9              ;  r[x10=7094e17b2f x9=0] w[x12=7094e17b2f]
70BE45888C [libEncryptor.so!688C]  ldur   x12, [x12, #-3]          ;  r[x12=7094e17b2f] w[x12=a698e56c37492857]
memory read at 0x7094e17b2c, data size = 8, data value = a698e56c37492857
70BE458890 [libEncryptor.so!6890]  rev    x12, x12                  ;  r[x12=a698e56c37492857] w[x12=572849376ce598a6]
70BE458894 [libEncryptor.so!6894]  str    x12, [x11, x9]            ;  r[x12=572849376ce598a6 x11=7094e177a0 x9=0]
memory write at 0x7094e177a0, data size = 8, data value = 572849376ce598a6
* add x12, x10, x9：将 x10 和 x9 的值相加，结果存储在 x12 中。此时，x12 = 7094e17b2f。
* ldur x12, [x12, #-3]：从 x12 - 3 的内存地址 (0x7094e17b2c) 处读取 8 字节数据，并将其加载到 x12 中。读取的值是 a698e56c37492857，往上找0x7094e17b2c 赋值处，无，继续 data value = 57。
call libc.so!rand
70BE454C90 [libEncryptor.so!2C90]  subs   x20, x20, #1             ;  r[x20=20] w[x20=1f]
70BE454C94 [libEncryptor.so!2C94]  strb   w0, [x19], #1            ;  r[w0=223b7c57 x19=6fb5ff35e0] w[x19=6fb5ff35e1]
memory write at 0x6fb5ff35e0, data size = 1, data value = 57
70BE454C98 [libEncryptor.so!2C98]  b.ne   #-12                     ;
70BE454C8C [libEncryptor.so!2C8C]  bl #-8700                     ;  r[sp=7094e18130] w[lr=70be454c90]
70BE452A90 [libEncryptor.so!A90]   adrp   x16, #135168             ;  w[x16=70be473000]
70BE452A94 [libEncryptor.so!A94]   ldr    x17, [x16, #3984]         ;  r[x16=70be473000] w[x17=721a8557b0]
memory read at 0x70be473f90, data size = 8, data value = 721a8557b0
70BE452A98 [libEncryptor.so!A98]   add    x16, x16, #3984           ;  r[x16=70be473000] w[x16=70be473f90]
70BE452A9C [libEncryptor.so!A9C]   br x17                        ;  r[x17=721a8557b0]
call libc.so!rand
70BE454C90 [libEncryptor.so!2C90]  subs   x20, x20, #1             ;  r[x20=1f] w[x20=1e]
70BE454C94 [libEncryptor.so!2C94]  strb   w0, [x19], #1            ;  r[w0=47946028 x19=6fb5ff35e1] w[x19=6fb5ff35e2]
memory write at 0x6fb5ff35e1, data size = 1, data value = 28
往下32次调用，刚好对应第二段buff；
对rand函数进行hook：
let rand = Module.findExportByName(null, "rand");
Interceptor.attach(rand, {
    onEnter: function (args) {
    }, onLeave: function (retval) {
        console.log("rand onLeave:", retval)
    }
});
结论为调用32次rand函数生成的后两位；

```

那么前 38 位确定为固定值 + 随机数；

```
待解析加密剩余：
713b184620  8b 4e cf 26 56 cb 9c 10 b7 08  ....\>.N.&V.....
713b184630  05 7e 43 93 63 f0 fe 3c f4 7e 57 07 59 52 81 e5  .~C.c..<.~W.YR..
713b184640  90 a6 34 82 50 d2 ce 3a e1 10 2b 11 f3 e7 63 5e  ..4.P..:..+...c^
713b184650  59 98 e8 3e 3b ea 2d 1d 6c f3 1d cc 93 63 4a fb  Y..>;.-.l....cJ.
713b184660  32 40 a1 04 9f e5 33 62 bd fd 5d c7 2a 95 4c a3  2@....3b..].*.L.
713b184670  fe 24 4a 43 6d fe 
搜索 data value = 8b，得以下逻辑， 跟踪地址0x2A1C,可到达函数sub_26E0,以及其上层函数sub_2A5C
70BE454A1C [libEncryptor.so!2A1C]   strb   w10, [x1]                ;  r[w10=8b x1=7094e180c8]
memory write at 0x7094e180c8, data size = 1, data value = 8b
70BE454A20 [libEncryptor.so!2A20]  strb   w13, [x1, #1]            ;  r[w13=8b4e x1=7094e180c8]
memory write at 0x7094e180c9, data size = 1, data value = 4e
70BE454A24 [libEncryptor.so!2A24]  strb   w14, [x1, #2]            ;  r[w14=8b4ecf x1=7094e180c8]
memory write at 0x7094e180ca, data size = 1, data value = cf
70BE454A28 [libEncryptor.so!2A28]  strb   w8, [x1, #4]             ;  r[w8=56 x1=7094e180c8]

```

通过对 sub_26E0 进行 hook, 可得出结果为多次调用该函数加密形成 onLeave arg1，对上层函数 sub_2A5C 进行 hook 和静态分析，

sub_2A5C 函数中存在函数 sub_4678，为完全数据计算，且其中数组常量为 AES 常量；sub_2A5C 的参数传入 sub_4678，

参数 1 为空内存，参数 2 为地址，猜测为 key 和 iv；参数 3 为数值，猜测为长度；

拿取 hexdump 结果使用 aes 解密，得明显 call_native 字符串；得到 key， iv， arg；

重新进行 hook 结果：

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3frcqrDxPO81uiaia4zkb48ydkgBsRrdozxPmBBUjKcb5OEiccHpbfRqZJg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3faQk4P1rdAsrLuicaWFL0S1LK7HlUF0SZckNfeEg8Exp7AWmJhicZ84icQ/640?wx_fmt=png&from=appmsg)

trace 日志分析：

key 为定位 sub_4678 参数 2 位置，拿到对应地址：

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fJtEYo4G6lFeLWANIl8Bibuem5u5e0GGMxAPicsRn3aWibFgO1FSxeCsibA/640?wx_fmt=png&from=appmsg)

iv 为定位 sub_66D4 参数 3 位置，拿到对应地址：

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fTe3czbfY75U8TMRnEArtbUgndXP0z3hsGQL7rrE6wrA9Er14SkbVUg/640?wx_fmt=png&from=appmsg)

得结果：

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fKfcCjSFY2qGpnicXD36Yl6IWSdQBWpOokTUpnPRQ68VgiceCaMv7Q96w/640?wx_fmt=png&from=appmsg)

那么接下来分析 key， iv， arg 来源：

```
key & iv:
70BE458D3C [libEncryptor.so!6D3C]   ldrb   w12, [x11, #7]           ;  r[x11=7094e17ae8] w[w12=af]
memory read at 0x7094e17aef, data size = 1, data value = af
70BE458D40 [libEncryptor.so!6D40]  add    x13, x10, x8              ;  r[x10=7094e17e83 x8=0] w[x13=7094e17e83]
70BE458D44 [libEncryptor.so!6D44]  add    x8, x8, #8                ;  r[x8=0] w[x8=8]
70BE458D48 [libEncryptor.so!6D48]  cmp    x8, #64                   ;  r[x8=8] w[xzr=]
70BE458D4C [libEncryptor.so!6D4C]  sturb  w12, [x13, #-3]         ;  r[w12=af x13=7094e17e83]
memory write at 0x7094e17e80, data size = 1, data value = af
70BE458D50 [libEncryptor.so!6D50]  ldrh   w12, [x11, #6]           ;  r[x11=7094e17ae8] w[w12=afd3]
memory read at 0x7094e17aee, data size = 2, data value = afd3
70BE458D54 [libEncryptor.so!6D54]  sturb  w12, [x13, #-2]         ;  r[w12=afd3 x13=7094e17e83]
memory write at 0x7094e17e81, data size = 1, data value = d3
70BE458D58 [libEncryptor.so!6D58]  ldr    x12, [x11]                ;  r[x11=7094e17ae8] w[x12=afd33089f2cb914b]
memory read at 0x7094e17ae8, data size = 8, data value = afd33089f2cb914b

```

搜索 data value =afd33089f2cb914b，得以上逻辑， 跟踪地址 0x6D3C, 可到达函数 sub_6C40, 以及其上层函数 sub_6DAC

对函数函数 sub_6C40 ，其上层函数为 sub_6DAC，对两个函数进行 hook 和静态分析；

可得 sub_6C40 返回参数一跟 sub_6DAC 返回参数二, 得结果：

前 16 个字节 即 0x00~0xF 为 key；17~32 字节即 0x10~0x1F 字节为 iv。

函数 sub_6DAC 中含有 v11=0x5BE0CD19137E2179LL，为 sha512 常量，验证确定为 sha512。

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fiaqIbqjQOVcHBX4krvncbRdiaZcAXPjKKpR0ON38oia3L7wvMpadleicWw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fp83uBofDzdc8ic0DUEmRUG39F6aaK2oV34icJehtNmiclaxmFmUeGPvjQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fy52ftCUicFu2e769k9qkTgISwCZUVpSAAKOLw9pNXasmndYkjHgk0uA/640?wx_fmt=png&from=appmsg)

key 与 iv 向量得解析来到，确定 sha512 得 arg；

```
sub_6DAC onLeave arg0:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189401780  53 d6 00 62 a4 a0 2d 44 bf d5 26 63 17 d3 4d 25  S..b..-D..&c..M%
7189401790  07 c0 d6 5d 01 9b 1c 8b b1 fe e4 c0 aa b3 e5 35  ...]...........5
71894017a0  1d aa e9 57 c9 3f e8 93 b0 e3 84 62 b2 0e 50 8e  ...W.?.....b..P.
71894017b0  8e 53 8e f3 40 b4 66 04 37 30 04 45 c3 56 30 60  .S..@.f.70.E.V0`
71894017c0  4d d4 c2 e6 b8 31 62 09 0e 52 b3 c7 a6 73 3b a4  M....1b..R...s;.
71894017d0  1c b2 46 2b 82 9a b5 8a 19 6b 39 db 57 17 75 24  ..F+.....k9.W.u$
71894017e0  f4 9b af 7f 08 e8 d6 8d 26 a7 2e 37 c1 a9 5a 2f  ........&..7..Z/
71894017f0  1f 05 a5 18 92 ae f2 94 97 32 b6 2a 38 aa dd 58  .........2.*8..X
确定前64位 random随机数为sha512结果
sub_6DAC onLeave arg0:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189400400  4a aa 33 36 9b b6 51 13 41 92 3e 31 29 0f 63 1b  J.36..Q.A.>1).c.
7189400410  5f 9d 33 28 54 b7 b7 a3 12 d7 c2 c3 81 91 d9 cb  _.3(T...........
7189400420  31 00 00 00 00 00 00 00 0e 00 00 00 00 00 00 00  1...............
7189400430  00 29 40 89 71 00 00 00 68 72 65 61 64 20 30 00  .)@.q...hread 0.
7189400440  a0 ba 53 1c 72 00 00 00 a0 e5 53 1c 72 00 00 00  ..S.r.....S.r...
7189400450  00 90 46 89 71 00 00 00 68 72 65 61 64 20 30 00  ..F.q...hread 0.
7189400460  01 00 00 00 01 00 00 00 a8 a3 42 89 71 00 00 00  ..........B.q...
7189400470  01 00 00 00 74 00 69 67 75 72 61 74 69 6f 6e 00  ....t.iguration.
7189400480  01 00 00 00 01 00 00 00 40 51 42 89 71 00 00 00  ........@QB.q...
7189400490  00 00 00 00 6e 66 69 67 75 72 61 74 69 6f 6e 00  ....nfiguration.
71894004a0  00 00 00 00 00 00 00 00 a0 04 40 89 71 00 00 00  ..........@.q...
71894004b0  a0 04 40 89 71 00 00 00 00 00 00 00 00 00 00 00  ..@.q...........
71894004c0  01 00 00 00 01 00 00 00 48 a4 42 89 71 00 00 00  ........H.B.q...
71894004d0  01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
71894004e0  01 00 00 00 01 00 00 00 90 51 42 89 71 00 00 00  .........QB.q...
71894004f0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
sub_6DAC onLeave arg1: 0x20
sub_6DAC onEnter arg2:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189a7c570  53 d6 00 62 a4 a0 2d 44 bf d5 26 63 17 d3 4d 25  S..b..-D..&c..M%
7189a7c580  07 c0 d6 5d 01 9b 1c 8b b1 fe e4 c0 aa b3 e5 35  ...]...........5
7189a7c590  1d aa e9 57 c9 3f e8 93 b0 e3 84 62 b2 0e 50 8e  ...W.?.....b..P.
7189a7c5a0  8e 53 8e f3 40 b4 66 04 37 30 04 45 c3 56 30 60  .S..@.f.70.E.V0`
7189a7c5b0  00 3e 20 8e 71 00 00 00 00 32 5b 85 71 00 00 00  .> .q....2[.q...
7189a7c5c0  10 c6 a7 89 71 00 00 00 f4 43 0d 39 71 00 00 00  ....q....C.9q...
7189a7c5d0  f0 c9 a7 89 71 00 00 00 c4 cb 0d 39 71 00 00 00  ....q......9q...
7189a7c5e0  f3 71 10 8e 71 00 00 00 30 c9 a7 89 71 00 00 00  .q..q...0...q...
7189a7c5f0  f5 71 10 8e 71 00 00 00 80 c6 a7 89 71 00 00 00  .q..q.......q...
7189a7c600  f0 c9 a7 89 71 00 00 00 14 cc 0d 39 71 00 00 00  ....q......9q...
7189a7c610  02 00 00 00 00 00 00 00 a0 c6 a7 89 71 00 00 00  ............q...
7189a7c620  01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7189a7c630  03 00 00 00 00 00 00 00 e0 c6 a7 89 71 00 00 00  ............q...
7189a7c640  a0 5b ad 8d 71 00 00 00 00 04 40 89 71 00 00 00  .[..q.....@.q...
7189a7c650  20 00 00 00 00 00 00 00 70 c5 a7 89 71 00 00 00   .......p...q...
7189a7c660  80 00 00 00 00 00 00 00 80 17 40 89 71 00 00 00  ..........@.q...

```

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fEF5KI2P9uQVt8XUeUdelGTCW16pmyHuyzT9JYCoDcX4zTb4VsmKwlg/640?wx_fmt=png&from=appmsg)

```
7084be1d40  4d d4 c2 e6 b8 31 62 09 0e 52 b3 c7 a6 73 3b a4  M....1b..R...s;.
7084be1d50  1c b2 46 2b 82 9a b5 8a 19 6b 39 db 57 17 75 24  ..F+.....k9.W.u$
7084be1d60  f4 9b af 7f 08 e8 d6 8d 26 a7 2e 37 c1 a9 5a 2f  ........&..7..Z/
7084be1d70  1f 05 a5 18 92 ae f2 94 97 32 b6 2a 38 aa dd 58  .........2.*8..X
后半段在so文件中确定为固定字符串。

```

故 key 与 iv 的解析完成 剩下 aes 加密的 arg；

```
arg：
分为两段
b433616c57de5f43c9a658e14af43a00bbbbddf29a94b7ff1e688e5eb0f5bac9c2a52c03b73ce8fab87a616MoRnbUfHBdCarU9683KT3AsB67n92UtF5eab90e5f
68656c6c6f2c206a716975
其中 68656c6c6f2c206a716975 为输入内容 
sub_6DAC onEnter arg0: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF
7189417020 68 65 6c 6c 6f 2c 20 6a 71 69 75 1c 72 00 00 00 hello, jqiu.r...
7189417030 35 9b 7a 2e 3d cc 2b 31 05 7d a7 7e 92 75 8c 2e 5.z.=.+1.}.~.u..
7189417040 8c 75 be f6 6c ae e2 68 54 2c 4c 83 19 3f f6 5f .u..l..hT,L..?._
由上述可知 sub_6DAC 为 sha512, 
sub_6DAC onLeave arg2:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189a7caa0  b4 33 61 6c 57 de 5f 43 c9 a6 58 e1 4a f4 3a 00  .3alW._C..X.J.:.
7189a7cab0  bb bb dd f2 9a 94 b7 ff 1e 68 8e 5e b0 f5 ba c9  .........h.^....
7189a7cac0  c2 a5 2c 03 b7 3c e8 fa b8 7a 61 2e 1d 67 ce 54  ..,..<...za..g.T
7189a7cad0  72 69 63 bb af 87 e9 d5 8c 7b 13 85 ea b9 0e 5f  ric......{....._

```

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fNlI41iauHicj3tL9tKwHwic9QpwOy0MQib3ORUAhKNaZpVqlEoYOLwNyog/640?wx_fmt=png&from=appmsg)

```
sub_6DAC onLeave arg0:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189417020  68 65 6c 6c 6f 2c 20 6a 71 69 75 1c 72 00 00 00  hello, jqiu.r...
7189417030  35 9b 7a 2e 3d cc 2b 31 05 7d a7 7e 92 75 8c 2e  5.z.=.+1.}.~.u..
7189417040  8c 75 be f6 6c ae e2 68 54 2c 4c 83 19 3f f6 5f  .u..l..hT,L..?._
7189417050  c0 a3 42 89 71 00 00 00 90 51 42 89 71 00 00 00  ..B.q....QB.q...
7189417060  60 a4 42 89 71 00 00 00 e0 51 42 89 71 00 00 00  `.B.q....QB.q...
7189417070  c0 19 5b 1c 72 00 00 00 30 52 42 89 71 00 00 00  ..[.r...0RB.q...
7189417080  00 a5 42 89 71 00 00 00 80 52 42 89 71 00 00 00  ..B.q....RB.q...
7189417090  a0 a5 42 89 71 00 00 00 d0 52 42 89 71 00 00 00  ..B.q....RB.q...
71894170a0  40 a6 42 89 71 00 00 00 20 53 42 89 71 00 00 00  @.B.q... SB.q...
71894170b0  e0 a6 42 89 71 00 00 00 70 53 42 89 71 00 00 00  ..B.q...pSB.q...
71894170c0  80 a7 42 89 71 00 00 00 c0 53 42 89 71 00 00 00  ..B.q....SB.q...
71894170d0  a0 aa 42 89 71 00 00 00 10 54 42 89 71 00 00 00  ..B.q....TB.q...
71894170e0  40 ab 42 89 71 00 00 00 60 54 42 89 71 00 00 00  @.B.q...`TB.q...
71894170f0  e0 ab 42 89 71 00 00 00 b0 54 42 89 71 00 00 00  ..B.q....TB.q...
7189417100  20 ad 42 89 71 00 00 00 00 55 42 89 71 00 00 00   .B.q....UB.q...
7189417110  c0 ad 42 89 71 00 00 00 50 55 42 89 71 00 00 00  ..B.q...PUB.q...
sub_6DAC onLeave arg1: 0xb
sub_6DAC onEnter arg2:              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7189a7caa0  b4 33 61 6c 57 de 5f 43 c9 a6 58 e1 4a f4 3a 00  .3alW._C..X.J.:.
7189a7cab0  bb bb dd f2 9a 94 b7 ff 1e 68 8e 5e b0 f5 ba c9  .........h.^....
7189a7cac0  c2 a5 2c 03 b7 3c e8 fa b8 7a 61 2e 1d 67 ce 54  ..,..<...za..g.T
7189a7cad0  72 69 63 bb af 87 e9 d5 8c 7b 13 85 ea b9 0e 5f  ric......{....._

```

得后半段为输入内容 sha512, 故 aes arg 确定为 输入内容 sha512 拼接输入内容。

```
则最终确定加密模式为，
如 ：
718942d340  74 63 05 10 00 00 4a aa 33 36 9b b6 51 13 41 92  tc....J.36..Q.A.
718942d350  3e 31 29 0f 63 1b 5f 9d 33 28 54 b7 b7 a3 12 d7  >1).c._.3(T.....
718942d360  c2 c3 81 91 d9 cb 65 eb 18 b1 02 13 c1 c5 b5 6a  ......e........j
718942d370  f9 02 a6 d5 74 83 ee a2 21 f1 d6 c3 45 ad 4f f6  ....t...!...E.O.
718942d380  56 c7 e1 d8 20 cf d1 c4 f0 b9 f9 59 62 11 ed 40  V... ......Yb..@
718942d390  79 19 56 59 59 be 9b cc cd 89 f2 0a 1f 1a 2d e4  y.VYY.........-.
718942d3a0  7d 88 4f 26 89 89 7a dc 61 37 08 9a 3d 08 70 56  }.O&..z.a7..=.pV
718942d3b0  4a f1 97 3d 7c a5  
74 63 05 10 00 00  为固定值  
随机32位字符串 4a aa 33 36 9b b6 51 13 41 92 3e 31 29 0f 63 1b 5f 9d 33 28 54 b7 b7 a3 12 d7 c2 c3 81 91 d9 cb
4aaa33369bb6511341923e31290f63JDaxNfPy83okP3kLScwCkGiuMcyC4Pcynb
随机字符串进行 sha512， 拼接固定64位字节 
4d d4 c2 e6 b8 31 62 09 0e 52 b3 c7 a6 73 3b a4 1c b2 46 2b 82 9a b5 8a 19 6b 39 db 57 17 75 24
f4 9b af 7f 08 e8 d6 8d 26 a7 2e 37 c1 a9 5a 2f 1f 05 a5 18 92 ae f2 94 97 32 b6 2a 38 aa dd 58  
4dd4c2e6b83162090e52b3c7a673JDaxNfPy83okP3kLScwCkGiuMcyC4Pcyn524f49baf7f08e8d68d26a72e37c1a95a2f1f05a51892aef2949732b62a38aadd58
进行 sha512，其中前16位字节为 aes key， 16到32位字节为 aes iv；
后内容
65 eb 18 b1 02 13 c1 c5 b5 6a f9 02 a6 d5 74 83 ee a2 21 f1 d6 c3 45 ad 4f f6 56 c7 e1 d8 20 cf d1 c4 f0 b9 f9 59 62 11 ed 40
79 19 56 59 59 be 9b cc cd 89 f2 0a 1f 1a 2d e4 7d 88 4f 26 89 89 7a dc 61 37 08 9a 3d 08 70 56 4a f1 97 3d 7c a5
为 输入内容 经过sha512加密后字节凭借原始字节， 进行aes加密；

```

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fLqWiakXRbwPNyJK31eRQBSJL2N7QpVehuhhGt55lDJr8kXicpL409kAA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RcM7tgiaVAVvVTCTwHsA31QXj6PvicoX3fu0BgMDHCLSpBGz0ZJbZicQSb4DjA1icVcFam2xzWvpADj6eHMOQA6qDQ/640?wx_fmt=png&from=appmsg)