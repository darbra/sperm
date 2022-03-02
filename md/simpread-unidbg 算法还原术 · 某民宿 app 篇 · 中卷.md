> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjA5NDE1OQ==&mid=2247485004&idx=1&sn=069bc22bc51804156ae4195d85cc83fc&chksm=9f8bb76ca8fc3e7ac35ead494f6365f5d34d7b4775e627b5b3aa339b275f62d7e690873f7e96&mpshare=1&scene=1&srcid=10131nsND0h4vAvW6oFb8k7g&sharer_sharetime=1634055434006&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

上回说到 EncodeHTTP 函数

伪代码又臭又长，看不懂没关系，从下往上看；找关键的函数；

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3HYOh1bIyDDFvNq4dWRWpVPD0xvE5geBD8rtzcW6T6zK4XTJTAQJDGg/640?wx_fmt=png)

而且之前猜测是 sha1 算法，那我们找找 sha1 的特征；

伪代码虽然很长，但是函数还是没几个的，都点进去看看

tjtxtutf8 看上去很像 base64，后边验证

tjcreate 函数里面有点东西

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy33wDQAenUUrtM4s3U8pGkamNQCXPHKHoqu9oKeeawbgbfJk4Qo75t0Q/640?wx_fmt=png)

有个 sha1 的特征，为什么？  

哈希算法最明显的特征就是初始化链接常量 (幻数) 和固定常数 K 值

**初始化链接常量 (幻数)**

md5 算法有 4 个初始化链接常量

```
A=0x01234567，B=0x89abcdef，C=0xfedcba98，D=0x76543210

```

但考虑到内存数据存储大小端的问题我们将其赋值为：

```
A=0x67452301，B=0xefcdab89，C=0x98badcfe，D=0x10325476

```

sha1 是 md5 的亲兄弟，比 md5 多了一个初始化链接常量;

```
A=0x67452301，B=0xefcdab89，C=0x98badcfe，D=0x10325476，E=0xCA62C1D6

```

**k 值**  

md5 k 值取值 （64 个，对应 64 步运算）  

```
constTable=[
0xd76aa478, 0xe8c7b756, 0x242070db, 0xc1bdceee, 0xf57c0faf,
0x4787c62a, 0xa8304613, 0xfd469501, 0x698098d8, 0x8b44f7af,
0xffff5bb1, 0x895cd7be, 0x6b901122, 0xfd987193, 0xa679438e,
0x49b40821, 0xf61e2562, 0xc040b340, 0x265e5a51, 0xe9b6c7aa,
0xd62f105d, 0x2441453, 0xd8a1e681, 0xe7d3fbc8, 0x21e1cde6,
0xc33707d6, 0xf4d50d87, 0x455a14ed, 0xa9e3e905, 0xfcefa3f8,
0x676f02d9, 0x8d2a4c8a, 0xfffa3942, 0x8771f681, 0x6d9d6122,
0xfde5380c, 0xa4beea44, 0x4bdecfa9, 0xf6bb4b60, 0xbebfbc70,
0x289b7ec6, 0xeaa127fa, 0xd4ef3085, 0x4881d05, 0xd9d4d039,
0xe6db99e5, 0x1fa27cf8, 0xc4ac5665, 0xf4292244, 0x432aff97,
0xab9423a7, 0xfc93a039, 0x655b59c3, 0x8f0ccc92, 0xffeff47d,
0x85845dd1, 0x6fa87e4f, 0xfe2ce6e0, 0xa3014314, 0x4e0811a1,
0xf7537e82, 0xbd3af235, 0x2ad7d2bb, 0xeb86d391]

```

sha1 k 值取值（4 个 k 值，每个 k 值对应 20 步运算）

```
第1轮 0≤t≤19步  Kt=0x5A827999 
第2轮 20≤t≤39步 Kt=0x6ED9EBA1
第3轮 40≤t≤59步 Kt=0x8F188CDC  
第4轮 60≤t≤7步  Kt=0xCA62C1D6 

```

回到 ida  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLfOW5HtibXLPLiaxWx5Ls4Qr8zIqbpEwq3ibXQed08ZG2PjNDdZ51RgsMA/640?wx_fmt=png)

这个 0xCA62C1D6 就是 sha1 的 k 值之一，所以说这个函数有 sha1 的特征

而在工程标准化中，哈希加密一般分为 Init、Update、Final 三步  

简单来说：

1.  Init 是一个初始化函数，初始化核心变量
    
2.  Update 是主计算过程
    
3.  Final 整理和填写输出结果
    

tjcreate 函数可以看作是第一步的 Init; 但是其他四个常量哪去了呢？

看汇编就知道了 (这些汇编划到最下面有解释)  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLIny2iaicBhWdI77HNkia94awdgFmLUqb2J1gYSC00tRXQgyRDTl03hwBQ/640?wx_fmt=png)

为啥还是少了一个幻数  

因为 ida 把 2C70 识别成函数了，我们手动修改它的数据类型

鼠标移到 sub_2C70 处，按 d

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLEolSjsvklEPujaiaqAxWfpgibQeSgf7BTfIicfzicHJaEhNIUg8NXwqatQ/640?wx_fmt=png)

再按 d 转成 word  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLSLKcibRY7RwDS1nb8tVEd34HE7V5yibzrnD0vqXMOYZOaDJ7yEdTRteA/640?wx_fmt=png)

再按 d 转成 dword

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLMsoBqjS1iavTX851aptIlyywfSLDBJJG5EovyjPkOqJoUJOicNSHlbjg/640?wx_fmt=png)

然后找下面的 Update 和 Final

tjreset 函数可以看作是第二步的 Update

```
int __fastcall tjreset(signed __int64 a1, int a2)
{
  int v2; // r4
  _BYTE *v3; // r5
  int v4; // r6
  v2 = a2;
  v3 = (_BYTE *)HIDWORD(a1);
  v4 = a1;
  while ( v2 )
  {
    *(_BYTE *)(v4 + *(_DWORD *)(v4 + 64)) = *v3;
    *(_BYTE *)(v4 + *(_DWORD *)(v4 + 64)) ^= 0x21u;
    LODWORD(a1) = *(_DWORD *)(v4 + 64) + 1;
    *(_DWORD *)(v4 + 64) = a1;
    if ( (_DWORD)a1 == 64 )
    {
      j_tjupdate(v4, v4);
      *(_DWORD *)(v4 + 64) = 0;
      a1 = *(_QWORD *)(v4 + 72) + 512LL;
      *(_QWORD *)(v4 + 72) = a1;
    }
    ++v3;
    --v2;
  }
  return a1;
}

```

一般来说只要 hook 这个函数，拿到入参，就能拿到明文  

我们先用 frida 来试试，

```
function myhexdump(name, hexdump_obj, len_obj){
    console.log("-------------------"+ name.toString()+"-------------------\\n");
    console.log(hexdump(hexdump_obj,{
        length:len_obj
    }) );
    console.log("-------------------ENDEND-------------------\\n")
}
function tjreset() {
      var pointer = Module.findBaseAddress("libtujia_encrypt.so").add(0x2C94 + 1);
      Interceptor.attach(pointer,
          {
              onEnter: function (args) {
                  myhexdump("tjreset arg0:", args[0], 128)
                  this.buffer = args[0];
                  myhexdump('tjreset arg1:', args[1], parseInt(args[2]))
                  console.log('tjreset arg2:' + parseInt(args[2]));
              },
              onLeave: function (retval) {
                  myhexdump("tjreset ret:", this.buffer, 16)
              }
          }
      );
  }

```

注意：这里 ida 识别函数参数个数错了，实际应该是三个参数，看汇编就知道了  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IVdrFxIibnRdm38WZ8U6ib6YQbwYkCfDoBhLxKibVC5OGmGRmSGs7GkrUA/640?wx_fmt=png)

R0、R1、R2 分别对应参数 1、2、3

开始 hook

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IuVm1cciaibUpqbVqop3URGmqVpFicnibkwrHE9B6ic89PK3o15PuI657kNg/640?wx_fmt=png)

unidbg hook:

```
public void hook2c94(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x2C94 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer arg1 = ctx.getPointerArg(1);
                int length = ctx.getIntArg(2);
                Pointer out = ctx.getPointerArg(0);
                ctx.push(out);
                ctx.push(length);
                Inspector.inspect(arg1.getByteArray(0, length),"tjreset arg1");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length =ctx.pop();
                Pointer output = ctx.pop();
                byte[] outputhex = output.getByteArray(0, length);
                Inspector.inspect(outputhex, "tjreset ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
public static void main(String[] args) throws FileNotFoundException {
    Test test = new Test();
    test.hook2c94();//要在函数调用之前hook，不然hook个p
    test.call_encrypt();
  // test.get_bodyencrypt();
    }

```

unidbg 的输出跟 frida 是一样的：  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IOC3yZyyDLWemsoNPiaktB6bQHtB1qo9xJ6ib967ZjNXLWdVGF7UgWApA/640?wx_fmt=png)

一般情况来说 tjreset arg1 的值就是明文

打开逆向之友，去碰碰运气

```
https://gchq.github.io/CyberChef/

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Ie9CdD076acjSwMEhpL1Se8mBVgTLl9p5KsUaBwHR10nyr0uESHich2A/640?wx_fmt=png)

结果对不上，难道 sha1 魔改了？？

莫急，冷静分析，再看下伪代码，和标准的有没有啥区别

发现 13 行有个异或不对劲，标准算法没这一步

```
*(_BYTE *)(v4 + *(_DWORD *)(v4 + 64)) ^= 0x21u;

```

这是把 tjreset arg1 先异或了 0x21 再进行下面的计算

那我们再用 CyberChef 试试

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8ImaC0IdKnKH2ceVoAHptfZ9ltROoUVSSw4upQeicPRHqd8iaInK8MmibtA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH7AOnSA7XXltmdUNPcicEzTQXhtLXzpMia52PAPiaP3UicE9Vr7yNY3YG8mAaZHaYALibQRXIvCdBIroxA/640?wx_fmt=jpeg)

看看明文是啥呢

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IGiacbRnfXHAaWJ8tT8kmLSzD56W9HjqenKGTsfI1AWCaD4XXnGF0hHA/640?wx_fmt=png)

看着是 base64 编码，结合之前猜测 tjtxtutf8 就是 base64，那就解码看看吧

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8I0VSMTTuA0LefqB6RAEOfC9rZzfNFbUHKa4OqWPj9HsCicwicLmcL75gw/640?wx_fmt=png)

哦？明文出来了，茅塞顿开

所以这个明文组成就是：

1.#arg1_str#arg5# 固定字符串 #arg2_str#arg3_str 的字符串排序

2. 字符串反转

3.base64 编码

不过

有人说了，我就是看不出这有个异或 0x21，你就说咋办吧。。而且这样逆向出来明文好像是靠蒙的，没有成就感啊

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IGWMByiaXezfYkCuoB1aDwU9rrnrU54NcKCNSdiaHHHygKGp6wbwib6KPg/640?wx_fmt=jpeg)

行，再露一手

来看看 18 行的 j_tjupdate 函数

```
int __fastcall tjupdate(int a1, int a2)
{
  int i; // r3
  int v3; // r1
  char *v4; // r5
  int v5; // r2
  int v6; // r3
  int v7; // r5
  int v8; // r10
  int v9; // r1
  int v10; // r6
  int v11; // r11
  int v12; // r3
  int v13; // r0
  int v14; // r2
  int v15; // r4
  int v16; // r3
  int v17; // r5
  int v18; // r0
  int v19; // r6
  int v20; // r1
  int v21; // r8
  int v22; // r0
  int v23; // r2
  int v24; // r6
  int v25; // r2
  int v26; // r12
  int v27; // r3
  int v28; // r0
  int v29; // r4
  int v30; // r5
  int v31; // r3
  int v32; // r0
  int v33; // r1
  __int64 v34; // r3
  int v36; // [sp+4h] [bp-17Ch]
  int v37; // [sp+8h] [bp-178h]
  int v38; // [sp+Ch] [bp-174h]
  int v39; // [sp+10h] [bp-170h]
  int v40; // [sp+14h] [bp-16Ch]
  int v41; // [sp+1Ch] [bp-164h]
  char v42[320]; // [sp+20h] [bp-160h]
  char _70[320]; // [sp+70h] [bp-110h]
  char _C0[320]; // [sp+C0h] [bp-C0h]
  char _110[320]; // [sp+110h] [bp-70h]
  int v46; // [sp+160h] [bp-20h]
  for ( i = 0; i != 64; i += 4 )
    *(_DWORD *)&v42[i] = bswap32(*(_DWORD *)(a2 + i));
  v3 = 0;
  while ( v3 != 256 )
  {
    v4 = &v42[v3];
    v5 = *(_DWORD *)&v42[v3 + 8] ^ *(_DWORD *)&v42[v3 + 32] ^ *(_DWORD *)&v42[v3 + 52];
    v6 = *(_DWORD *)&v42[v3];
    v3 += 4;
    *((_DWORD *)v4 + 16) = __ROR4__(v5 ^ v6, 31);
  }
  v8 = *(_QWORD *)(a1 + 80) >> 32;
  v7 = *(_QWORD *)(a1 + 80);
  v9 = *(_DWORD *)(a1 + 100);
  v10 = 0;
  v12 = *(_DWORD *)(a1 + 92);
  v11 = *(_DWORD *)(a1 + 96);
  v41 = a1;
  v38 = *(_DWORD *)(a1 + 88);
  v13 = *(_DWORD *)(a1 + 88);
  v14 = v12;
  v37 = v12;
  v40 = v7;
  v39 = v8;
  v36 = v11;
  while ( 1 )
  {
    v15 = v14;
    v14 = v13;
    v16 = v7;
    if ( v10 == 20 )
      break;
    v17 = *(_DWORD *)&v42[4 * v10];
    v18 = (v13 & v8 | v15 & ~v8) + v11 + __ROR4__(v16, 27) + v9;
    ++v10;
    v11 = v15;
    v7 = v17 + v18;
    v13 = __ROR4__(v8, 2);
    v8 = v16;
  }
  v19 = 0;
  while ( 1 )
  {
    v20 = v15;
    v15 = v14;
    v21 = v16;
    if ( v19 == 20 )
      break;
    v22 = *(_DWORD *)&_70[4 * v19++];
    v23 = (v14 ^ v8 ^ v20) + __ROR4__(v16, 27) + v11;
    v11 = v20;
    v16 = v23 + *(_DWORD *)(v41 + 104) + v22;
    v14 = __ROR4__(v8, 2);
    v8 = v21;
  }
  v24 = 0;
  while ( 1 )
  {
    v25 = v20;
    v20 = v15;
    v26 = v21;
    if ( v24 == 20 )
      break;
    v15 = __ROR4__(v8, 2);
    v27 = *(_DWORD *)&_C0[4 * v24];
    v28 = (v25 & v20 ^ (v25 ^ v20) & v8) + v11 + __ROR4__(v21, 27) + *(_DWORD *)(v41 + 108);
    ++v24;
    v8 = v21;
    v21 = v28 + v27;
    v11 = v25;
  }
  v29 = 0;
  while ( 1 )
  {
    v30 = v25;
    v25 = v20;
    v31 = v26;
    if ( v29 == 20 )
      break;
    v32 = *(_DWORD *)&_110[4 * v29++];
    v33 = (v20 ^ v8 ^ v30) + __ROR4__(v26, 27) + v11;
    v11 = v30;
    v26 = v33 + *(_DWORD *)(v41 + 112) + v32;
    v20 = __ROR4__(v8, 2);
    v8 = v31;
  }
  LODWORD(v34) = v26 + v40;
  HIDWORD(v34) = v39 + v8;
  *(_QWORD *)(v41 + 80) = v34;
  *(_DWORD *)(v41 + 88) = v20 + v38;
  *(_DWORD *)(v41 + 92) = v37 + v30;
  *(_DWORD *)(v41 + 96) = v36 + v11;
  return _stack_chk_guard - v46;
}

```

这个函数就是 sha1 的一个明文分组计算过程，啥是明文分组呢，简单来说就是你要加密的明文如果很长很长，就要把明文分组，每一组是 512bit，也就是 64 字节长度，不足的要填充，填充规则如下

```
1 在信息的后面填充一个1和无数个0，直到满足上面的条件时才停止用0对信息的填充。
2 在这个结果后面附加一个以64位二进制表示的填充前信息长度（单位为Bit），如果二
进制表示的填充前信息长度超过64位，则取低64位。
经过这两步的处理，信息的位长=N*512+448+64=(N+1）*512，即长度恰好是512的整数倍。
这样做的原因是为满足后面处理中对信息长度的要求。

```

要讲明白还是有点难（为了不误人子弟）大家感兴趣的自己搜资料研究；

回到 tjupdate 函数

每一个分组进行 4 轮变换，每一轮计算 20 步，伪代码也很明显（78 行、94 行、109 行、125 行）  

我们 hook 这个函数，就能知道每一轮的明文；

```
function tjupdate() {
      var pointer = Module.findBaseAddress("libtujia_encrypt.so").add(0x2AB4 + 1);
      Interceptor.attach(pointer,
          {
              onEnter: function (args) {
                  myhexdump("update arg0:", args[1], 64)
                  console.log("update arg0:"+Memory.readUtf8String(args[1]))
            },
              onLeave: function (retval) {
              }
          }
      );

```

unidbg hook：

```
public void hook_2AB4(){
        IHookZz hookZz = HookZz.getInstance(emulator); 
        hookZz.enable_arm_arm64_b_branch(); 
        hookZz.wrap(module.base + 0x2AB4 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer text = ctx.getPointerArg(1);
                byte[] texthex = text.getByteArray(0, 64);
                Inspector.inspect(texthex, "block");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    };

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IoKVOVJzuQpxbjHOGicCZeopT3w8GQ7wSujKBibT9J0ibyjkJP4kg14rCA/640?wx_fmt=png)

然后把每次的明文拼接起来，就是整个的明文

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IVsov6JbDlCkNGHiazE7Ce12SOh7yibgaJ6zGOEYqR4QVCBwokhsQ4l8A/640?wx_fmt=png)

然后再 sha1，是不是一样？  

话说回来，Final 还没看呢，Final 是最后整理输出的，

有些情况不参与计算，有些情况参与计算，也就是 tjget 函数了, 在这个案例里是参与了计算的

hook 一个

```
function tjget() {
      var pointer = Module.findExportByName("libtujia_encrypt.so",'tjget');
      console.log('case tjget:' + pointer);
      Interceptor.attach(pointer,
          {
              onEnter: function (args) {
                  this.buffer = args[1];
              },
              onLeave: function (retval) {
                  myhexdump("tjget 结果：", this.buffer, 20)
              }
          }
      );
  }

```

unidbg hook：  

```
    public void hook_2CEA(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x2CEA + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer arg1 = ctx.getPointerArg(1);
                ctx.push(arg1);
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer output = ctx.pop();
                byte[] outputhex = output.getByteArray(0, 20);
                Inspector.inspect(outputhex, "tjget ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

hook 结果：

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IAia8hIC7rLZ6GLmZorrtVicxdCRerEkvkUB482WY9QmevFYkmJecOyGw/640?wx_fmt=png)

**明文的生成**

分析完最后一步的标准算法，回过头来分析明文的生成  

看看具体实现的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IQibsia3NlKY42jUwbKNSc9QUW5eC268YxrOwiceohLcicUeGo4ypU8G8Rw/640?wx_fmt=png)

首先这里生成了固定的 key，

这里用 unidbg 的另一种 hook 方式

在这个地方下个断点

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IficjSEPrSf94gVbJWIZmHuC9pJTwh0wdng8DZnMx0evtGmMdTicUic1XQ/640?wx_fmt=png)

代码如下：

```
    public void HookByConsoleDebugger() {
        Debugger debugger = emulator.attach();
        //在module.base+0x3108地址处添加一个断点
        debugger.addBreakPoint(module.base+0x3108);
    }
    public static void main(String[] args) throws FileNotFoundException {
        Test test = new Test();
        //test.hook2c94();
       // test.hook_2AB4();
        test.HookByConsoleDebugger();
        test.call_encrypt();
       // test.get_bodyencrypt();
    }

```

运行，程序会断下  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IiconIbURy6ozHUKhqxgfMxMkKjgVyhyfw0SsEqic92jYMelACFPvre6w/640?wx_fmt=png)

这里的 r0 寄存器存放的是参数 1 的值，r1 是参数 2，r3 是参数 3

怎么查看 r0 的值呢

输入：mr0

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8I8z5t3iboojjJN3unuX8Wo1mKxu1xemIvXWro7JGoJEjrOUX6sE8pr5w/640?wx_fmt=png)

也可以输入地址：m0xbffff65c

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IIDQLZhHam42GM83GSHqt8JiafLhmypW1Gwk1wGichiacESAFGiclPHULJA/640?wx_fmt=png)

默认 size 是 112，可以指定长度  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IIglibS7jgiboV8iajOd9qOuV9jnWq34hKwKWbhucqyEdCrJxOe9y3axlw/640?wx_fmt=png)

所以这里的参数 1 是 tjhchk

返回值怎么看呢，根据 arm 汇编的约定，返回值会放到 r0

这下面几个地址都可以

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Ij1uC6kicPeFXkuvogFS5fLibNfqiaX9nVDrZwTPPczaElHWEoRc3zy7Mg/640?wx_fmt=png)

在 0x310C 加个断点，命令：b0x310C

然后按 c（跳到下一个断点）

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8ItibhAmS6zMvXq3xrwl7icjAfF2LPDhahxOV4wxcLicHX7rsv7WMkCRuiaw/640?wx_fmt=png)

然后查看 r0 的值

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IZ4NxdiaD9qwNX4quZIPlicgsESfEGexiag493VquichpQRCK5rMFcsMqPQ/640?wx_fmt=png)

验证后发现是标准的 base64

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8ILE41sQahfzt48xT4K18NmgBJUqcXl23DhwVK7qicSqrLUzCL4jVwr2A/640?wx_fmt=png)

还有一种方法能获取返回值  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8I3Uzbn1MtEFl0KYNFXdjib9eiaBNk4RPTseuKNsP6b3WXnOJRwmnasMqg/640?wx_fmt=png)

0x291C 下个断点 (函数具体实现的地址)

```
debugger.addBreakPoint(module.base+0x291C);

```

重新运行程序  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IgPfiaAsRBqBmskyFOXUwyjVuFzv1wQUpWFOibevSN1PblS1mwuDV4Ffg/640?wx_fmt=png)

这时 r0 一样是 tjhchk

然后输入命令 blr，blr 会在函数返回的地方下一个断点

c（跳到一下个断点）

查看 r0 的值

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IoWwXy7O7Qdj6Wuew5PKjUQLibG9ia7c8o9AzesoMmygICPPd3Cuoz89A/640?wx_fmt=png)

生成了 key；然后往下看

可以看到很多地方都在异或 0x21

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Iys1HVKfWPASYPx9iblmsVTnvNkskScQHbhTEricZFnBzIBhNhcq29ZDQ/640?wx_fmt=png)

这些都是把各个参数先异或 0x21，再拼接，最后是在 j_tjsplittxt 函数做字符串翻转  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IHoZKq1JnpzyPXGIZdAJgh0AX5qDBCZgV7ibYXUwlGUdaicUQberPASdQ/640?wx_fmt=png)

来打个断点瞧瞧  

```
debugger.addBreakPoint(module.base+0x2DEC);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IDl6QpE4nOH0tHreDkpjafNfq6icxtKwKibjnBrpgFEd1K55yyLAEsqJw/640?wx_fmt=png)

这个参数 1 是乱码，我们把他异或 0x21 试试呢  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IdVb87jC2ruYWWxlZXndgL9cZibGxe7H9bDwoYrKJicFyMIl7WgZrzx1g/640?wx_fmt=png)

这里就已经拼接好了嘛

看看返回值，blr，c

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IicibZYub2z5MxA3tuwsFNYWEDllOqZrwbtKHx6YDldrgFr2X0aOwe4KQ/640?wx_fmt=png)

这里的返回值在 r8，我猜是因为在 Thumb 程序中，只能使用 r4~r7 来保存局部变量

直接 mr8 是不行的，

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Izhw5aIH48OO6laLbXpAQIdrLpprmAKEfxOqZhxAAExSwfw4KhtAXCw/640?wx_fmt=png)

所以我们 m0x40223180

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IOUzMZnBd7jqCiceMXhhUKzicfw2icnjdax1b8aEkB0ZjWh4cZWMeybaFA/640?wx_fmt=png)

或者输入 s，单步调试，让它把 r8 赋值到 r0  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IibXVFvtKrm6VfBqZ1TQsI6xKShI9YlFe47L3uR4Z8gw0uXDS6u1tp7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Ic26v1gaDF9KQ4xEIEibpURtBQnfLg2xYTFagHUjneUNj6zPhoYQLJZA/640?wx_fmt=png)

然后把这个值异或 0x21

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IX58JCuWibnLGwDIETCzOibMONPfQLtZ1yhVk1pFbr7lxlrX7UoHgs7pA/640?wx_fmt=png)

然后又是 base64

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8IbY8icH8VWwcQtGian8AGiaqzW6pgdGQGQSQITlz5OnaylwhGNOVJicCQXA/640?wx_fmt=png)

上面那个调用第三个参数是 10，这里是 30，有何大咪咪呢？

进去看看

编码之前参数 3 不等于 10 的时候编码之前把明文异或 0x21

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8I2fQLeTBOaZg5xJasP3w6jvZGCvibP3DBkCONhoqnWK0orxLCxvnTq7A/640?wx_fmt=png)

编码之后参数 3 不等于 10 的时候把明文异或 0x21

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8Iw4yIzEO71Xx88siaicv4r4aH8lna5zpkxC0qxXssiaX6ia20FhdbIAeT9A/640?wx_fmt=png)

所以总结下来就是，

1.  每个参数和固定的 key 分别异或 0x21（有个参数要排序）
    
2.  拼接
    
3.  翻转（j_tjsplittxt）
    
4.  异或 0x21（tjtxtutf8）
    
5.  base64 编码（tjtxtutf8）
    
6.  异或 0x21（tjtxtutf8）
    
7.  异或 0x21（j_tjreset）
    
8.  sha1（j_tjreset）
    

发现没有，4 次异或 0x21，等于就是没有异或。。所以可以简化成 3 步

1.  参数拼接（有个参数要排序）
    
2.  翻转
    
3.  sha1
    

剩下就是写代码了。。

下篇 body 分析再见

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH4CSjdGEGiabI6rw0gJY5e8I3qibNB54S5D95gqNibiaqWgibgFQ8GFkKrktyLzrBica2X00YMSFJJ0294g/640?wx_fmt=jpeg)

**彩蛋**

初始化链接常数加载汇编指令解析

0x2C48 下个断点  blr  c  查看 r0

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLsOEvIgmm516UKMJywsBDea11jxic8MJUueVZVA4UwicjJcq24h11ianIg/640?wx_fmt=png)

上面说了转成小端就是 sha1 五个幻数

```
0x67452301 0xEFCDAB89 0x98BADCFE  0x10325476 0xC3DEE1F0

```

看看汇编是怎么加载的

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH42GylcEhBjdBbp7pLyu7kLlwFTTYUODtGibt5YbAM6qKUfv8OWB2Q9LlShRC4ibaaEiaeFrHXF4kGzw/640?wx_fmt=png)

```
ADR  R1, dword_2C80

```

adr 用来加载地址，而且是相对地址寻址

dword 修饰一个操作数为 Double Word，即 4 字节

即 0x2C80 地址放到 R1

看看内存中的值

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58o5bofiaQqtdXdlUsT6tVMNXO4TyxJyJLBM3JsZAsMIHhicCx5M6ETmcbg/640?wx_fmt=png)

然后就是下面的两行

```
VLD1.64   {D16-D17}, [R1]
VST1.64   {D16-D17}, [R0]

```

V 开头表示是 NEON 指令

什么是 NEON（抄的）

```
Arm NEON 技术是针对 Arm Cortex-A 系列和 Cortex-R52 处理器的高级 SIMD（单指令多数据）架构扩展。
具有NEON技术的处理器都会配备了32个64位的寄存器和16个128位的寄存器,它们分别被标识为(D0-D31),(Q0-Q15)

```

```
vld1.64 {d16, d17}, [r1]
将r1内存中的数据映射到d16, d17寄存器上面。这样就可以直接通过d16, d17寄存器来操作数据,64表示一个寄存器有64位(8个16进制)

```

在这里就是把 r1 中的 01 23 45 67 89 AB CD EF 放到 d16 寄存器，FE DC BA 98 76 54 32 10 放到 d17 寄存器

```
vst1.64 {d16, d17}, [r0]
将d16, d17寄存器中的数据映射回r0内存中，这样就可以通过打印r0来看到结果

```

在这里就是把 d16(01 23 45 67 89 AB CD EF) 放到 r0 寄存器，d17(FE DC BA 98 76 54 32 10) 放到 r0 寄存器

之后呢在 tjupdate 函数里，加载出来使用

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58ojm7RI8TdiaOCsXcNZGexHKmicoSWnGYJL5iaHQaF1sQvducaib33CEx6icA/640?wx_fmt=png)

汇编

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58obiaNM13OH2EqZiafxIbo0oLjibvSjibcGqgmlVW1wuIbRicxvIicibQBCCZrA/640?wx_fmt=png)

重点说下这个地方

```
LDMIA.W R11, {R2,R3,R11}
LDMIA指令，IA表示每次传送后地址加4,W修饰一个操作数为Double Word，即4字节

```

R11 在 unidbg 中是 sp

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58oxLPWQGEIDwib5yf3sZGqGf1Pib01ZbXOl3GbYq8ibAV7jNuQRJl5uTSOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58ogNvrO98yoPLfTeTtto9Md3yw0B8JbicvMqWm2F2OHm65qdVVareKlRQ/640?wx_fmt=png)

依次把 FE DC BA 98 放到 R2; 76 54 32 10 放到 R3

按 s

可以看到 R2、R3 已经被重新赋值 (小端)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5EUJXagGDvgbWxZEhwib58o1GlwIzUmhGpahGHsFky7qBdTInDrg3v8W16AmuIIZj9Zg2ukYuaJ4w/640?wx_fmt=png)

其他的地方可以自己动态调试看看