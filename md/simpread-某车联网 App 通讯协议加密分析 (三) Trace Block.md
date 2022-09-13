> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/f7KbPmTDnOJLh0Sm0GlN1Q)

一、目标

之前我们已经用 unidbg 跑通了 libencrypt.so，那么如何判断跑出来的结果是对是错？再如何纠正 unidbg 跑错误的流程，是我们今天的目标。

v6.1.0

二、步骤
----

### 找到明显的接口来判断

checkcode 是加密，加密的结果确实不好判断是否正确。不过我们可以试试解密，能解密就是对的，简单粗暴。这里解密函数是 decheckcode 。

跑一下，这个结果明显不对，死心了

```
call decheckcode: fac34ffa581987c7a1ffa5b876ca96ce

```

### 分析问题

结果不对，肯定是过程不对。

那么解决方案就是 分析对比 unidbg 运行的流程和 app 运行的流程 有哪里有不同?

对比运行流程有三个粒度， 函数、代码块和代码。 (在 ida 里按空格，出现的流程图中的每个块就是代码块)

我们今天主要要对比 decheckcode 函数，所以先从代码块的粒度来做 Trace。

### Trace Block

unidbg 提供一个 BlockHook，每运行到一个代码块就触发这 Hook，我们就利用他来做 Trace Block

Trace Block 结束之后，把命中的代码块地址都打印出来，用于在 Frida 中去 hook

有了这两个函数就可以干活了

在执行 decheckcode 之前去做 Trace， 执行之后去打印所有命中的 Block 地址。

###### Tip: 

这个样本没那么复杂，所以就直接 Trace 所有代码范围，讲究人是需要缩小范围，只 Trace 自己感兴趣的部分。

```
Find native function Java_com_bangcle_comapiprotect_CheckCodeUtil_decheckcode => RX@0x4002b1bc[libencrypt.so]0x2b1bc
  sub_2b1bc
  sub_2b20c
  sub_2b238
  sub_2b254
  sub_2b270
  sub_2b28c
  sub_2b2a8
  sub_2b2c4
  sub_2b2e0
  sub_2b2fc
  sub_2b318
  sub_2b334
    ...

subTrace len = 127
  ,['sub_21400','0x21400']  ,['sub_21c00','0x21c00']  ,['sub_22200','0x22200']  ,['sub_2b604','0x2b604']  ...

```

Trace 的结果出来了，命中的地址列表也打印出来了，一共命中了 127 个地址块。

### frida Hook 对比

把命中的地址列表导入到 frida 里面去 hook，然后就可以对比出来 unidbg 跑的流程和 App 跑的流程的差别了。

### 对比结果

对比的方法比较 low，先把 unidbg Trace Block 的结果复制到文本文件 1，然后把 frida hook 打印的结果复制到文本文件 2。 最后开启 Beyond Compare 来对比

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2LDicFEPdEMBgOzlic0CTnjOTZjd0nrFeZUiaQZK9PickIib8ILzAARXma3pFrMJo1YGRCoicUVKyPJt7w/640?wx_fmt=png)1:cmp

过程虽然很 low，但是结果可一点都不 low，从对比的结果看，大家之前都是好朋友，不过 sub_18650 之后就开始分道扬镳了。

这时候就需要问问 ida 了。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2LDicFEPdEMBgOzlic0CTnjOicGwBIYaBLLyicVETibpTcCx0f7S1Y4nKhY3RLKP0XmncqoIDu8DU7cKQ/640?wx_fmt=png)1:fopen

这里 fopen 了一个文件，文件名是做了 base64。

base64 谁不会呢，随便写两行代码就可以解出来了 /proc/%d/cmdline

这又在考我们的 android 编程知识了，问了下谷哥，哥说了，这是在读进程名，对于 apk 来说，进程名就是他的包名。

回想在 unidbg 中 有个不起眼的报错

```
[11:26:48 927]  INFO [com.github.unidbg.linux.ARM64SyscallHandler] (ARM64SyscallHandler:1309) - openat dirfd=-100, pathname=/proc/2256/cmdline, oflags=0x0, mode=0

```

这也是在提醒我们，读取进程名失败了。

### 重定向 io

unidbg 是支持这种情况的，先让 CaranywhereDemo 多继承一个 IOResolver 来做 io 重定向

ok 了，这几步又和 App 跑的一样了，大家又是好朋友了。

不过问题还是没有解决，跑出来的结果还是不对，木有解密成功。而且貌似这个 app 还有坑，hook 点一多就摆烂，直接崩溃。

得找新武器对付它了，期待下一章的大结局吧。

三、总结
----

何以解忧，唯有 Trace。

能下断点 Debug 的 App，一定就逃不出手心了。所以现在 App 的关注点都是抵抗 Debug，抵抗下断点。

结果不对，就和正确的结果去对比流程，跑的和你一模一样，总没毛病吧？

![](https://mmbiz.qpic.cn/mmbiz_jpg/llIox45YGib2LDicFEPdEMBgOzlic0CTnjOhy2kB68bIZ3kPTkt1tVIQ1y2QmW3sHevBicNm8c0ANntN6REm5gBZsg/640?wx_fmt=jpeg)1:ffshow

凡说之难 在知所说之心 可以吾说当之

###### Tip: 

: 本文的目的只有一个就是学习更多的逆向技巧和思路，如果有人利用本文技术去进行非法商业获取利益带来的法律责任都是操作者自己承担，和本文以及作者没关系，本文涉及到的代码项目可以去 奋飞的朋友们 知识星球自取，欢迎加入知识星球一起学习探讨技术。有问题可以加我 wx: fenfei331 讨论下。

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2LDicFEPdEMBgOzlic0CTnjOmoJrsUBF9exib74V3ack5OZianQKbAYfNmo8MlMbiaP88IcqDZQHf5Orw/640?wx_fmt=png)

关注微信公众号，最新技术干货实时推送

![](https://mmbiz.qpic.cn/mmbiz_png/llIox45YGib2LDicFEPdEMBgOzlic0CTnjOFklWic5CrgDmsUB30FKL0J4Y4KWhusPFNxoaU9SyQQnXu2fpgX5GibicQ/640?wx_fmt=png)

 手机查看不方便，可以网页看

    http://91fans.com.cn