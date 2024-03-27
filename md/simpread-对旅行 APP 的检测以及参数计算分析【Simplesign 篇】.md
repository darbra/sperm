> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542926&idx=1&sn=47ff5f90a90cf436b8797d1f9bd0563e&chksm=b18d53c486fadad239ad156749f1a3b6ede82a24537d5570f2602bfcb757ec81ec11c824bd55&scene=58&subscene=0#rd)

本来想把 SimpleSign 拆解进行发帖的，但是又觉得是挤牙膏，本贴我可能会持续更新，关于一些细节以及使用的技术手段，将 ss 函数完全解析出来以学习其实现混淆的方式方法。本篇为前置篇，后续会对此出一个补充的帖子，细数 scmain 中一些影响计算的点具体的影响因子等因素。

  

上一篇导航：对某旅行 APP 的检测以及参数计算分析【新手向 - 准备篇】（一）

（https://bbs.kanxue.com/thread-278621.htm）

  

---

本贴只讨论其实现原理，若有侵权请联系我删除
---------------------

  

**简要概述：**

##### 目标 so：scmain.so

##### 讨论的生成过程：SimpleSign

##### 使用工具：IDA pro 7.7、 Binary Ninja、Frida、Frida Stalker

##### 本篇文章实现：SimpleSign 的计算过程，包括前、中、后、变换四个主体阶段，文章中会详细介绍。

```
一





起手准备

```

  

上篇文章中，我们定位到了 SimpleSign 函数所在的地址偏移，所以我们根据 offset 去 IDA 定位其反汇编的代码，先观察其展示出来的东西是否满足我们的推倒过程。

  
SimpleSign 的 native 函数偏移为 0x7D4B4。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicbj6ibOWRkb8mh5ykGEsOLkssppcl8EG4BTFia3YrfzY7lSw2pR1k7nww/640?wx_fmt=jpeg&from=appmsg)  
  

结果很明显，代码做了混淆，但是其中我们可以发现一些反射调用的特征，`GetByteArrayElements`,`GetArrayLength`,`GetStringUTFChars`等，因为我们在 JNI Native 中知道 SS 函数传入的参数是一个字节数组和一个字符串，所以我们推断出此处跟我们要找的函数入口有关联。我们看一下 sub_7d4b4 的网状结构。  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicZNm5uc7EhNTDICXT94icutpKLSm6iaRHBlPkHvCpQ7fj6Yqt2xpxbu2w/640?wx_fmt=jpeg&from=appmsg)  
  

因为本文是新手向，我们就介绍一些简单点、通俗易懂的方法来分析 (难的我也不会)。

  

  

```
二





Trace - Frida Stalker

```

  

关于`Stalker`我在上一篇中已经介绍过了，包括对 msaoaidsec.so 的 anti 操作。我们直接跳到使用。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicUicJ3r2TmoQH6e1ngbwP3pibZVNpib7bJ6LQXUcCkp4F1eHW1uRCIc7tg/640?wx_fmt=jpeg&from=appmsg)  
  

关于 Stalker 的起始位置，以及长度，这一块需要我们自己去试，调整长度，因为很多时候有一些汇编指令的地址并不在我们 trace 的范围之内，会造成指令流 trace 的 log 记录不到的情况存在。

  
另外，记得要对 Java native 函数也 hook 上，方便我们对传入的参数有更直观的展示以及返回值的分析。

  

```
Java.perform(function () {
    let SecurityUtil = Java.use("ctrip.android.security.SecurityUtil");
    SecurityUtil["simpleSign"].implementation = function (bArr, str) {
        console.log('simpleSign is called' + ', ' + 'bArr: ' + bArr + ', ' + 'str: ' + str);
        let ret = this.simpleSign(bArr, str);
        console.log('simpleSign ret value is ' + ret);
        return ret;
    };


    SecurityUtil["init"].overload('android.content.Context', 'int').implementation = function (context, i2) {
        console.log(`SecurityUtil.init is called: context=${context}, i2=${i2}`);
        this["init"](context, i2);
    };

});


```

  

关于 trace，有几点要讲：

1. 我们对 msaoaidsec 已经进行了 anti 操作，但是并不影响其有一些其他的检测手段，会造成进程被 kill。

2. 我测试了几个版本的 frida，貌似 16.1.0 可以完整 trace 下来，我有点记不清了。

3. 魔改 frida，这个是另一个范畴了，暂时不表，后续会对其检测能力做更深的剖析。

  

Frida Stalker trace 的过程时间其实是比较长的，日志大概是 60MB 左右，90 万行左右，其中有一些在 MD5 算法的部分漏掉了，我没有重新跑，范围大概锁定在这个区间内，给大家一个参考。

  
展示下 trace 的结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic9UxBWGSUW7ickSUho5XQ4mFicpW4AXHW9BHc42om2QAtdXEkWbHS31eg/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic4R2OX1gDspVPlnZ2km3u0cFna3E2jlm5YAvzjYE8Km52ibcm1aDvkibw/640?wx_fmt=jpeg&from=appmsg)  
  

以上就是一个几乎完整的 SimpleSign 的计算过程。下面我们开始着手分析。

  

  

```
三





分析前32位

```

###   

### 3.1 设想

  

起初，我认为结果这 75 位的字符串应该是 MD5 + 某些特征 + MD5 组成的，可是通过 Frida Hook Native 函数发现，前 32 位几乎是不变的。第 41-43 位也是几乎不变的。那么我假设，此部分的构成是由一个特征 (32 位) + 每次都会变化的特征(8 位) + 不变的(3 位) + 疑似 MD5(32 位) 组成的。  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicyL1BrNSicLaoemjEiaaVHdaibQp2YHI2p7pOdiaoEqpYqgujO5y1xsAVBA/640?wx_fmt=jpeg&from=appmsg)  
  

我们先对前 32 位进行分析。取前 4 位，在 trace 日志中进行搜索。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicdJSplZLtOKRwibNCqF9ml7XoPgXjsCJUsUib9hGFl5iammdqI07UsrWrQ/640?wx_fmt=jpeg&from=appmsg)  
  

可以看到图中我做了标识，E0AA 是由 d2538 处的汇编代码执行了异或运算，我们试着在 IDA 中去 d2538 处观察其计算逻辑。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicIISuX9bBqw7iaXVeg7Y8JZfgNw4tP0NfPwvBGoqJjTzDEoeH2yvRCpA/640?wx_fmt=jpeg&from=appmsg)  

  

猜测 v17 的值应该就是我们的 0x45304141 了，推测`sub_D1DB4`的参数 a2 是用来存放前 32 位的地址，我们验证一下 v5 的记过是否是 5-8 位，鼠标放在 v5 处，Tab 切换到汇编代码，根据其地址在 trace 日志中搜索。

  

```
libscmain.so d23dc		ldr x16, [sp, #0x50] ; 	 x16 = 0x7190e94e3d --> 0x7189db0948   (1e0f3c2d5a4b7869) 
libscmain.so d23e0		ldr w18, [x16, #4] ; 	 x18 = 0x7 --> 0x64326333 
libscmain.so d23e4		eor w15, w15, w18 ; 	 x15 = 0x57765407 --> 0x33443734


```

  

0x33443734 恰恰就是第 5-8 位，那么我们几乎就确定了这个函数就是我们要找的前 32 位生成的位置，但是此方法中只有 4 个变量来存放结果，但是我们在 trace 日志中所搜该地址发现结果是 2 个，那么我们可以假设此方法执行了两次，两次的执行结果相加正好是 32 位。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicvlZO1lu9pLM9SBKstl2x63C2CrXBoSklIZHUEmKxKzmxY58Z7iaNLyw/640?wx_fmt=jpeg&from=appmsg)  

  

至此，我们确定了此函数的作用，以及参数 a2 的功能，那么下一步我们要确认如下几点：

1.a1、a3 参数。

2. 此函数的调用过程是怎样的。

  

对于调用过程，可以参考 IDA 的`X`键，查看交叉引用，但是如果存在过多的调用情况排查起来其实略麻烦，配合 trace 日志能更方便的节省一点时间，但是也有可能存在跳转指令是处在花指令的范围内，如果这样的话那根据日志排查起来就略微有一点点麻烦。还有就是可以用 frida 打印调用栈，这个方法略微有一些看脸。

  
碰碰运气

  

```
libscmain.so d5f2c		ldp x0, x30, [sp], #0x10 ; 	 x0 = 0xc4 --> 0x71edd1f2f0   (���*�g�Iԥxt.��)	 sp = 0x7189db0860 --> 0x7189db0870 
libscmain.so d5f30		ldur x0, [x29, #-8] ;  
libscmain.so d5f34		ldr x1, [sp, #0x10] ;  
libscmain.so d5f38		ldr x2, [sp, #8] ;  
libscmain.so d5f3c		bl #0x7190d1adb4 ;  
libscmain.so d1db4		stp x0, x30, [sp, #-0x10]! ; 	 sp = 0x7189db0870 --> 0x7189db0860   (����q) 
libscmain.so d1db8		ldr w0, #0x7190d1adc0 ; 	 x0 = 0x71edd1f2f0 --> 0xd1 
libscmain.so d1dbc		bl #0x7190d1ae60 ;  
libscmain.so d1e60		sub x0, x0, #0x11 ; 	 x0 = 0xd1 --> 0xc0 
libscmain.so d1e64		eor x0, x0, #0xc0 ; 	 x0 = 0xc0 --> 0x0   (null) 
libscmain.so d1e68		add x0, x0, #1 ; 	 x0 = 0x0 --> 0x1 
libscmain.so d1e6c		ldr w0, [x30, x0, sxtx #2] ; 	 x0 = 0x1 --> 0xb8


```

  

查看 trace 日志发现，D1DB4 方法调用的上方代码块有可能是正常的代码，根据地址 d5f38 去 IDA 中查看。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicgTFWOPEIW44dkjY3EnCNZE5iaKPleicMqFa5wUtjkSXkc1lIPmdR7SBg/640?wx_fmt=jpeg&from=appmsg)  
  

继续向上找

  

```
libscmain.so ed09c		ldr w0, #0x7190d360a4 ; 	 x0 = 0x71edd1f2f0 --> 0xec 
libscmain.so ed0a0		bl #0x7190d35dec ;  
libscmain.so ecdec		bl #0x7190d35ecc ;  
libscmain.so ececc		eor x0, x0, #0xc0 ; 	 x0 = 0xec --> 0x2c 
libscmain.so eced0		lsr x0, x0, #0 ;  
libscmain.so eced4		add x0, x0, #1 ; 	 x0 = 0x2c --> 0x2d 
libscmain.so eced8		ldr w0, [x30, x0, sxtx #2] ; 	 x0 = 0x2d --> 0x748 
libscmain.so ecedc		add x30, x30, x0 ;  
libscmain.so ecee0		ret ;  
libscmain.so ed538		ldp x0, x30, [sp], #0x10 ; 	 x0 = 0x748 --> 0x71edd1f2f0   (���*�g�Iԥxt.��)	 sp = 0x7189db0890 --> 0x7189db08a0 
libscmain.so ed53c		mov w3, wzr ; 	 x3 = 0x7190e94d78 --> 0x0   (null) 
libscmain.so ed540		bl #0x7190d1ee5c ;  
libscmain.so d5e5c		stp x0, x30, [sp, #-0x10]! ; 	 sp = 0x7189db08a0 --> 0x7189db0890   (����q) 
libscmain.so d5e60		ldr w0, #0x7190d1ee68 ; 	 x0 = 0x71edd1f2f0 --> 0x536 
libscmain.so d5e64		bl #0x7190d1ee80 ;  
libscmain.so d5e80		sub x0, x0, #0x3a ; 	 x0 = 0x536 --> 0x4fc 
libscmain.so d5e84		eor x0, x0, #0xfc ; 	 x0 = 0x4fc --> 0x400 
libscmain.so d5e88		lsr x0, x0, #0xa ; 	 x0 = 0x400 --> 0x1


```

  

试试 ecedc

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicoXichvLAegIN5TPctCCOa7fOB3eLibYjN4r3M1Av2PDGplUibEqRTvZdw/640?wx_fmt=jpeg&from=appmsg)  
  

定位到了 sub_ECDE4, 我们继续向上走，根据 ecde4 在日志中查找上层。

  

```
libscmain.so efcf8		eor x0, x0, #0xe0 ; 	 x0 = 0xf6 --> 0x16 
libscmain.so efcfc		sub x0, x0, #0x10 ; 	 x0 = 0x16 --> 0x6 
libscmain.so efd00		add x0, x0, #1 ; 	 x0 = 0x6 --> 0x7 
libscmain.so efd04		ldr w0, [x30, x0, sxtx #2] ; 	 x0 = 0x7 --> 0x1e0 
libscmain.so efd08		add x30, x30, x0 ;  
libscmain.so efd0c		ret ;  
libscmain.so efe78		ldp x0, x30, [sp], #0x10 ; 	 x0 = 0x1e0 --> 0x71edc30020   (TracerPid)	 sp = 0x7189db0a00 --> 0x7189db0a10 
libscmain.so efe7c		adrp x0, #0x7190eb7000 ; 	 x0 = 0x71edc30020 --> 0x7190eb7000 
libscmain.so efe80		add x0, x0, #0x1bc ; 	 x0 = 0x7190eb7000 --> 0x7190eb71bc   (ed756e23400710596bbd71988670248va4c8db2ae867a149d4a578742e90ec) 
libscmain.so efe84		sub x1, x29, #0x10 ; 	 x1 = 0x20 --> 0x7189db0a80   ( ) 
libscmain.so efe88		bl #0x7190d35de4 ;  
libscmain.so ecde4		stp x0, x30, [sp, #-0x10]! ; 	 sp = 0x7189db0a10 --> 0x7189db0a00   (�q��q) 
libscmain.so ecde8		ldr w0, #0x7190d35df0 ; 	 x0 = 0x7190eb71bc --> 0xd4 
libscmain.so ecdec		bl #0x7190d35ecc ;  
libscmain.so ececc		eor x0, x0, #0xc0 ; 	 x0 = 0xd4 --> 0x14 
libscmain.so eced0		lsr x0, x0, #0 ;


```

  

此处我们注意到一个字符串，`add x0, x0, #0x1bc`此处需要注意的一点是，他的汇编代码与 IDA 的反汇编并不一致，道理是相同的，粗俗一点理解其实就是根据某个偏移取到了内存空间中的某个值，这个值从哪里来其实我们目前暂时没办法确定，在 ida 的反汇编中，他的呈现是这样的`ADRL X0, unk_26E1BC`，在一个未处理字，暂时推测是某个代码块中应该向其赋了值。

  
IDA 根据此地址跳转，发现找到了上一层调用。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicchMBH77kxicQLX7MPXlKN3ojxBib1VuFM9HqK93Awn4azv2clT1dOInQ/640?wx_fmt=jpeg&from=appmsg)  
  

sub_EFC8C 函数我们查找交叉引用，发现只有一个函数调用了它 -> sub_F10C0, 0xF10C0 的交叉引用我们发现他的上一层其实就是我们的 sub_7D4B4。至此，整条 simplesign 的大体执行流程我们已经基本了解了，现在开始详细的解析 simplesign 是如何生成的。

  

### 3.2 详细解剖前 32 位是如何组成的

  

上面的快速预览中，我们知道了前 32 位的前置在 sub_EFC8C 中调用了 sub_ECDE4 函数，其中有两个参数，第一个就是我们 trace 中那一个 64 位的字符串，第二个呢？

  
`v7 = qword_270030(10L)`如果我们点击进去发现并没有什么，因为他是一个数据段，我们点击`qword_270030`再点击`X`会发现他其实是指向的是某个函数，这里我们发现他是在 so init 时候进行了定义。

  

```
qword_270030 = (__int64 (__fastcall *)(_QWORD))dlsym(handle, "malloc");
      qword_270038 = (__int64)dlsym(handle, "calloc");
      qword_270028 = (__int64 (__fastcall *)(_QWORD))dlsym(handle, "free");
      qword_270040 = (__int64 (__fastcall *)(_QWORD, _QWORD))dlsym(handle, "realloc");


```

  

那么这里的 270030 就代表了 malloc，申请了一块长度为 10 的内存空间。

  

#### 3.2.1 分析 sub_ECDE4

  

大致整理了一下，我们看图说。

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicpczibVtpRdvfD4qJC5AZJMX9tZFogtuxdNBemSIyNJdWlhWjwa9dFAw/640?wx_fmt=jpeg&from=appmsg)  

  

大致如图所示，需要关注的是`v17 = sub_D0404("f0e1d2c3b4a59687", 128LL, v22)。`  
  

这里不详细分析，因为我们看到传入了 3 个参数，第一个是`f0e1d2c3b4a59687`, 第二个是`128`，第三个是`v22。`  

第一个参数其实就是个固定值，推测跟版本有关，第二个长度，第三个传入的 v22，是决定前 32 位计算的重要参数，但是我们可以偷个懒，发现前两个参数是固定的，v22 用作存储计算后的一个地址指针。所以他的值是固定的，他的计算是通过第一个参数来变换的。这一块还原计算流程也不难，就不占用篇幅了。

  

#### 3.2.2 sub_D1DB4

  

直接看图

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicIWRYKjAicYuuWCFBbVAOAJnpiaJLM5tqF1GPn54qjGb2SJuDXTTk9p7A/640?wx_fmt=jpeg&from=appmsg)  

  

试试用 python 还原一下。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic95ibEhnrttuGHlXYt6ibnsmQ4cg8iaCeTljMM9RD5TPGPKwcKyme0JpibA/640?wx_fmt=jpeg&from=appmsg)  
  

与 trace 的结果一致。

  

  

```
四





分析第33-40位

```

  

老样子，根据我们得到的 simplesign 的第 33-40 位去 trace 日志中搜索，得到了`ldr x0, [sp, #0xf8] ; x0 = 0x34 --> 0x7189db0d6c (3469dc64E0AA3D74F268AE*****************)`，但是我们无法找到计算或者生成的地方，但是我们之前说过有怀疑这里是时间戳，那么我们对其进行转换，转换成 10 进制然后再时间戳转换试试，具体过程不细说了，直接说结果，转换的 10 进制并不符合时间戳，因为在这里要处理端续，转换成 0x64dc6934 再去匹配发现转换成时间戳就对得上我们 trace 的时间了。

  
这块其实是 syscall 了 gettimeofday 出来的，可以自行看一下，不多赘述。

  

### 4.1 41-43 位

  

至于 12C 的生成，后续会详细说明。

  

  

```
五





后32位的逻辑

```

  

我们继续假设后 32 位跟前 32 位一样的逻辑，进行拆分查找，日志搜索前 8 位，0x825B340C。

  

```
搜索到日志的第一条
libscmain.so eef28		ldur x11, [x29, #-0x10] ; 	 x11 = 0x7189db0d40 --> 0x716d81ec00   (825b340c)


```

  

汇编指令 ldur，证明是从内存中读取出来了，那么说明我们这个思路可能不正确，可以试试搜索`825b`发现也是一样，都是出现在了`0xeef28`位置上，那么我们就需要去分析一下，此位置是一个什么样的结构或者功能。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicdu0eLMOAglqPHXrABPbj33tQ7SlNOZzeLFVQicUD0MDAEKbA5PgtlEA/640?wx_fmt=jpeg&from=appmsg)  
  

很清晰明了，我们去逆推`v3 -> v4 -> v8 -> result = param1(第一个参数)。`

  
`X`键查看`sub_EEE38`的交叉引用，发现其恰好都在我们上一级`sub_F10C0`中，花一分钟去 trace 日志，我们基本可以定位到具体哪里调用了 eee38。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic5p2b7xNwv0Vic1xgJzx3JpFwyN3e2Lzvxcb0Kj4B3L1icM3yWrIoiaxicQ/640?wx_fmt=jpeg&from=appmsg)  

  

定位最后一个`sub_EEE38`的 param1 ->`v32`，观察其规律，发现`sub_F0E04`中有关联，我们试着用 frida 看一下 v32 的变化。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic7dfzdMwDAsB5EJibBLwZ4wp7NQVFsY84cjgqBmRgVvOPWQ1le7Hc6PQ/640?wx_fmt=jpeg&from=appmsg)  
  

因为我这图是后补的，所以后 32 位生成的值与上面不一样，我们只需要看不同的地方，由图可知，`sub_F0E04`在调用的时候，a1 的值应该就是后 32 位的值了，但是函数执行结束时，a1 的值是有变化的，而变化后的值恰恰就是最终生成的 simplesign 的后 32 位。那么我们假设，后 32 位计算后，会经过 f0e04 这个函数对后 10 位进行了某些变化。

  
我们先去分析后 32 位是怎么生成的再去研究这后 10 位的变化逻辑。

  

### 5.1 后 32 位的生成过程

  

我们接着看`sub_F10C0`，观察 v32 的轨迹，在 IDA 中我们观察 v32 并没有操作什么，那么问题很有可能出在了上一篇中，scmain 存在的花指令混淆的原因，试着去修复会很费时间，有没有其他方式能展现出各函数的执行流程呢？我们试一下`Binary Ninja`去反汇编，看看能不能比 IDA 展示的更好。

  
Binary Ninja 打开 scmain 会消耗一段时间，这期间不要管。我们看下结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicrlBqyrrExkf4Qib6Mb6iceNA6EbgMGeL8eHbDHZjb9jxEZZWmkjjyOkg/640?wx_fmt=jpeg&from=appmsg)  
  

Binary Ninja 的`sub_f0f0c`-> IDA 的`sub_F0E04`

Binary Ninja 的`sub_F77A4`-> IDA 的`sub_F2794`(Binary Ninja 识别出了跳转)  

为了防止阅读出现混乱，我依旧以 IDA 的反汇编来分析流程。

  

### 5.2 sub_F77A4 (Binary Ninja)

  

IDA 无法对`sub_F2794`进行有效的反编译，所以我们使用 Binary Ninja 来分析。  

通过 Binary Ninja (后续简称 BN) 分析，`sub_F0E04`的参数来自`sub_F77A4`的第三个参数 (param3)，而且 param3 在 BN 中也没有发现有其他函数参与修改、计算，那么我们推测，F77A4 是计算后 32 位的函数，点击进入。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicS62QxLwIB3arYicCv0nzrB57zUA0HqzxB8PplrQicpC7PxiaQSN4TibCMg/640?wx_fmt=jpeg&from=appmsg)  
  

是不是有眼前一亮的感脚，明文的 16 进制是不是很像`MD5`中的`K表`，还有`位移数`数量也不多不少，正好 64 个。

  
看一眼 Graph。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicHjTFtonGRkklQ6U3JkESaMqWoFibqK2biaHc2N4wG75KEJM0CvtPCu1A/640?wx_fmt=jpeg&from=appmsg)  

  

硬肝控制流对于我们来说没有任何好处与意义。因为我们是新手教程，所以就使用最简单有效的方式。

  
前 8 位，`825B340C`因为我们知道了后面是由 MD5 生成的，所以端续我们可以确定，去 trace 日志搜索`0xC345B82`, 第一条结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicK6lf2U10PT18lmxxe5Dr4juS3n0nQ87VeY7etp7NPOFfGKUv62zQfg/640?wx_fmt=jpeg&from=appmsg)  
  

定位汇编指令位置`0x1041b4`, 在`BN`按`G`输入跳转，发现会跳转到函数头部，因为 BN 的逻辑跟 IDA 不太一样，定位不到具体变量或者参数的位置，以结果所在的寄存器为地址，那么我们试试将我们的指令地址 + 4 或者 - 4。

  

```
001041b0                      int32_t x18_75 = ror.d(x8_1091 + 0x70363da3 + x13_196 + ((x13_367 | not.d(x9_673)) ^ x8_1070), 0x1a) + x13_367


```

  

至此，我们定位到了 MD5 结果的`A`所在的位置。

  
我们知道算法的代码了，SV 也知道了，但是我们还没有得到`入参`以及初始化 ABCD(魔数), 试着在 MD5 第一行计算中找规律，因为 A、B、C、D 一定会参与到前 4 行的运算中。

  

```
int32_t x8_1104 = ror.d(x19_10 + 0x500fe759 + (((var_158.d ^ var_168.d) & var_170.d) ^ var_268.d) + var_178.d, 0x19) + var_270.d
int32_t x9_810 = ror.d(x8_330 + 0x6fa2f477 + var_158.d + (((var_168.d ^ var_170.d) & x8_1104) ^ var_168.d), 0x14) + x8_1104
int32_t x0_127 = ror.d(x0_34 - 0x5cbacc06 + var_168.d + (((var_170.d ^ x8_1104) & x9_810) ^ var_170.d), 0xf) + x9_810
int32_t x8_1220 = ror.d(x18_156 + 0x46d88dcf + var_170.d + ((x9_810 & x0_127) | (x8_1104 & not.d(x0_127))), 0xa) + x0_127


```

  

我们可以看到，每一行的结果都会放到下一条计算逻辑的最后去相加。

  
我们梳理一下前两行相加计算的逻辑。  
`line1 = ror( x_19 + 0x500fe759 + var_178 + ((var_158 ^ var_168) & var_170) ^ var_268 , 0x19) + var_170(var_270=var_170)`  

  

通过 trace 日志，或者直接看反汇编，我们知道`x_19`就是传入的参数`M[0]`, 继续简化公式。  
`line1=ror(M[0]+k[0]+A+(异或与运算),移位数)+魔数之一`  

后面的第二行——第四行，我们就可以知道四个魔数对应的变量，通过 trace 日志可知道其值。四个魔数变量分别是`var_178`,`var_268 = var_158`,`var270 = var_170`,`var_168。`  
  

可是我们在当前的 if 分支中，并没有找到`var_178`与`var_168。`  
根据汇编指令流，分析当前 if 分支的第一行运行结果的计算过程，可以得到`var_178`的值，在 trace 日志中搜索得到如下内容，我们跳转到了另一个分支中。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicssLGeyHxsGB62zwyF8DUYHZib7PhVtWVz4G0YzOGWsGp3IEDXl1E8ng/640?wx_fmt=jpeg&from=appmsg)  
(请无视我的备注，那是还原算法时，做对比用的)  

  

我们发现刚才的 MD5 的魔数是另一个 MD5(我们简称 MD5_A) 的倒数第 4 行的计算结果。

  
那我们是否可以假设，MD5_A 的最后四行的结果就是我们之前 MD5(简称 MD5_B) 的魔数呢？

  
用 trace 日志做一下验证。

  
我们发现，MD5_B 的魔数是由 MD5_A 的结果与 MD5_A 的初始魔数相加而成得，而且 MD5_A 的计算逻辑与 K 表以及移位数都是一样的，推断两个 MD5 的算法是相同的。那么我们先去找 MD5_A 的魔数来源，根据 MD5_A 分支，我们推倒出 A、B、C、D 四个魔数对应的变量值，再去找赋值的来源，发现了一个变量`var_e0`, 继续逆推。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicsdSx0pnV4nuRiaCSBmfdekQAbADr0lMQFpc7ibdw839Zvn69rXvIblzA/640?wx_fmt=jpeg&from=appmsg)  

`data_24c0c0`值得我们关注。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicfxEOlsP6eG4tQIgZeVh1ocPMD0GoF0kn2Kc2kl96mmibs836aXNU9Pg/640?wx_fmt=jpeg&from=appmsg)  
  

恰好与我们 MD5_A 的魔数一模一样。

  
完事具备，就差还原了。具体细节闲下来我会在文章内补充，直接看结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicEcw3qffz2JHhJQOOSINZZ8yRC9rGzaPWhAuiaEoic4GjhUxFvKhz4ibNQ/640?wx_fmt=jpeg&from=appmsg)  
  

第一行为 MD5_A 的结果，也就是 MD5_B 的四个魔数。

  
第二行为 MD5_B 的结果，与 trace 的后 32 位的前 22 位完全匹配。

  
**注意！这里有一个问题，最后的结果不一定是后面多少位会变化，这个具体原因后面会详细讲。**

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic0hpLJc2UQswicpwpYN2gxsuYMPR2tD5SicZZAbjzFMzVRvt3FI4b1gPQ/640?wx_fmt=jpeg&from=appmsg)  
  

至于推导的过程，我建议新手朋友自己动手，能再最大程度上加深印象。

  
下一步我们要继续向上推，因为我们目前还不知道参数是什么。

  
通过使用 BN 与 IDA 的观察，F77A4 的上一个函数`sub_F01AC`的 param2 对应`sub_F77A4`的 param1， F01AC 的 param3 对应 F77A4 的 param2。

  
通过 frida hook，我们可以大概得了解到这几个函数参数的对应关系。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicgOBSsib0e6Jib7NezOIzf4iaMPbzzdCu8kkQxF1IT3pfnU46K8fQ2Wicjg/640?wx_fmt=jpeg&from=appmsg)

###   

### 5.3 sub_F10AC

  

此函数的参数 param1+76 处，作为一个计算控制器，给参与计算的 v6 赋值，0xAB 或者 0xCD。还原起来没有什么难度。直接上结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicrEZk3SRY8OSrP3wQVPslthk6jiaPKqaz3fngobeib8slK3vAyN9d8UOg/640?wx_fmt=jpeg&from=appmsg)

###   

### 5.4 sub_167424

  

此函数是一个非常关键的函数，个人觉得花费的时间比 MD5 要长，如图：  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdiciaWgia5rXytFeJ5XmJyCWdY00gBA6Cq1wZeyWbpBJcOmdudGWXsgBYlw/640?wx_fmt=jpeg&from=appmsg)  
  

小白方法，就是按照顺序与 trace 日志做辅助分析，并且根据 trace 的顺序行数在 ida 对应的指令上做记录。

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicS5hzGnOaDJV7hvP5JmA2LKs0gTsjdb3soB9bUayETxocVef6vWU5Sg/640?wx_fmt=jpeg&from=appmsg)  
  

跟着流程做对应的还原。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicmp7wjX6ia34Ba2f11I3ibkja0UASewvYA50TSnutgN9J8uQtibEwfjMCg/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdic9umY93CqAu3wu7sfbgGI3qMJp24Oodj2fvjAw34PwMJ1UdBCoqStSQ/640?wx_fmt=jpeg&from=appmsg)  
  

8 个变量对应了`sub_167424`函数结束时的 param3 对应地址的内容。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdickcbydmiaX0oKYNMmtla1zvZlwfNgcBKVu0ic3ASWvAusc7fRNDZt4ICQ/640?wx_fmt=jpeg&from=appmsg)  
  

param1 与 param2 不变，param3 在函数结束时内容被填充，然后在后续的指令中，param3 经过了序列化后将地址指针赋给了 v21，v21 作为刚才讲的`sub_F01AC`的入参 param1 进行了运算。

  
167424 前面得 F01AC 函数就比较简单了，其实就是 F10C0 传入的参数，也就是 SimpleSign 的那一串字节数组。

  
总结一下流程就是  
`SimpleSign入参 -> F01AC(SimpleSign字节数组作为参数) -> 167424 -> F01AC 再计算一次 -> F77A4 MD5计算生成后32位`

### 5.5 sub_F0E04 及其重要的一个校验点

  

首先，这个函数中有几个内存段需要先行知晓，例如`qword_26FD40``qword_26FE38``byte_26E010`等，因为这些地址的内容中有一些是在 so init 时赋值，有一些是其他环境影响内容变换，所以，要搞清楚这些是做什么的，怎么做的，才能决定最后 16 位的内容是怎样的。

  
先给出我的 so 的备注大概了解一下。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicPakvW95N9dPoOfeIiaY4eI0JIibm9OXPOo0shKJaYd4YGLS6rHd7efwA/640?wx_fmt=jpeg&from=appmsg)

  

simplesign 最后这 16 位的组成其实是前 8 位是当前时间与`JNI_OnLoad`的时间差的十六进制，高位为 0 则为 0，与原 simplesign 计算的后 16 位的前 8 为逐个异或。

  
第 9-10、11-12 则为一个固定数（目前看来）是与 0x00 和 0xff 的异或  
13-14 是取决于`byte_26E010`是否有改动。

  
15-16 则是一个计算公式`v18[7] = (v9 << 7) + 8 * v10 + 4 * v11 + 2 * v12 + v13;`

#### 5.5.1 时间差

  

根据分析`sub_ED574`我们得知，此函数的结果是由获取当前 time 再减去`qword_26FD40`得到的。

  
我们使用 IDA 的查找交叉引用功能，发现其是在`JNI_OnLoad`时被写入了内容。

  

#### 5.5.2 qword_26FE38

  

依旧使用上面的方式查看交叉引用，发现`STR`操作也在`JNI_OnLoad`中，赋值了 255L，那么其内容为`0xff。`

#### 5.5.3 byte_26E010

  

默认值是 0x2C，但是有几处涉及到更改，后续我们再说。

可能下一篇帖子是补充说明，也可能是 bncode。

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HhmzzZl1cjPI7D2FDXMhdicP27yHj2FyzzufE8sHZ1l2cehdwN7RIIkcDKHNg6hVgAkKlcNez5HyQ/640?wx_fmt=jpeg)

  

**看雪 ID：我是红领巾**

https://bbs.kanxue.com/user-home-839701.htm

* 本文为看雪论坛优秀文章，由 我是红领巾 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GAU6FettiaadHxOcltAfCZK5b9Lt5woUYKlIegAaFPP1MYdVRrHvW6kvQ8wplAYR4iafX4xzTIIShg/640?wx_fmt=png&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542141&idx=3&sn=fab46c4929e3e83ab26c616c5180f031&chksm=b18d50b786fad9a1a935d571b2b733503ff937be046b04787dfd7cac4507682aa4db9db55047&scene=21#wechat_redirect)

**#** **往期推荐**

1、[安卓真机无 root 环境下的单机游戏修改 - IL2CPP](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542627&idx=1&sn=b5b540af6000b64a62276b564f810e8b&scene=21#wechat_redirect)

2、[autojs 简介与对抗](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542614&idx=1&sn=9573296d611194968fc1d0abb2d843f7&scene=21#wechat_redirect)

3、[记一次汽车 app 白盒 aes 还原过程](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542568&idx=1&sn=63885b3c10c7f82d05ef175f84f63206&scene=21#wechat_redirect)

4、[使用 Unidbg 在 CTF-Android 题目的快速解题](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542348&idx=1&sn=9aa386ce99973dad49852cdc5a545eb9&scene=21#wechat_redirect)

5、[安卓逆向 MagicImageViewer 技巧分享](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542235&idx=1&sn=874ea77470091dd3b83fe44845b23e01&scene=21#wechat_redirect)

6、[国赛 babytree 赛题解析](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542213&idx=1&sn=1e877cc93653a8e0f7711a3db0e3bea7&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8EFOEfYvaMyfT5ia5LibNwcUgzibZvyt5nRHKuJ8p8JlZXFzH8uQ51GLJP47C3aEUIoDZmQZJR9kVs7g/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8EFOEfYvaMyfT5ia5LibNwcUgzibZvyt5nRHKuJ8p8JlZXFzH8uQ51GLJP47C3aEUIoDZmQZJR9kVs7g/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8EFOEfYvaMyfT5ia5LibNwcUgzibZvyt5nRHKuJ8p8JlZXFzH8uQ51GLJP47C3aEUIoDZmQZJR9kVs7g/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8EFOEfYvaMyfT5ia5LibNwcUgotJtxeMhqVHiaicrL97Lo0cnZxcW7YPkYE9x6s5CLL1NVltwavL2u0Bg/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多