> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzUxMjE3Nzc3Mw==&mid=2247484051&idx=1&sn=967c66798db322e049a1548eb4af3131&chksm=f9692191ce1ea887861845a74f73b46ad24c32b91d5e337e46c14de0b8ac0b7a5bce5155d9fe&mpshare=1&scene=1&srcid=0302JrksiN3jOHoRNvUMAIWI&sharer_sharetime=1646204024692&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

    最近学了点 ollvm 相关的分析方法, 正好之前朋友发我一个小 demo 拿来练练手.

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5ibCiaXqPAKcsrib2lgxrAEqRdia2uVMFF8WY2obaW9cG0YdibnPam7gY1gA/640?wx_fmt=png)

    看上去很简单 就是找 flag 用 jadx 打开发现加壳了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5ickQcQc8PWLakzy75BW2hYjqTibBHZPqLRibWkXb2LbYHfH4bwBYIibIXQ/640?wx_fmt=png)

    然后想试试直接用 fridadexdump 脱壳的时候发现 frida 上就崩了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5DehcibeZ27XEcibnc7dd6D6MS6O2wrUapMl2MDPRnwGgSshWiaOl1kFoQ/640?wx_fmt=png)

    上葫芦娃的 strongfrida 直接重启了!....  

    这只能去过反调试了, 打开 so 找了下. init 和. initarray(反调试常见位置, so 比较早的加载时机)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V58lVHF7KIBcAFJVWBP3k9vpsvf0GtKjR3YaDujibnmLrD8JhOf2xeHWg/640?wx_fmt=png)

    ctrl+s 打开 initarray 里面有两个奇怪的函数 decode, 这就是 ollvm 默认的字符串加密

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5KPTdaibwIw0tKtvrMibowyBBXVyD3YQHib7AhJsW5Tlic11icqF8wOplMIA/640?wx_fmt=png)

    导出函数中搜索 init 找到里面也有函数定义, 创建了一个线程执行反调试函数, 进入这个函数看到字符串都被加密了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5gCWsexicez4OjpgCBMXJ2CVWlvbzy25OYD3ictaZ0ic6JKxS5huqCy6HQ/640?wx_fmt=png)

    看了下代码 有个 kill 函数, 一开始就 hook kill 函数不让他自杀, 发现没用...

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5GkT8IVsjdv49BQGaZCjEM0n0jOtdHJ1ibToiao3LG5DGcz64mibNcjAOQ/640?wx_fmt=png)

    然后我继续找了下发现有个 strstr 函数 这有点可疑, 拿来比较字符串的, 直接 hook 一波

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5vpjD5GNPuiacZIdcicwmicQYCiakCgjicV4cricNzJkLPvnwTAcOJrn5KsjQ/640?wx_fmt=png)

    从代码中得知 str2 是我们要关心的对象, 是个 char * 类型直接 readCstring 打印  

    然后发现出现很多 frida 然后进程就崩了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5iadHgKkzm4fNrLYXiavhZ7T8OUaTRHhGu5pFDORMPicrkrCG4jvsHic3Vw/640?wx_fmt=png)

    所以接下来就有两种方法 直接 hook pthread 不让这个线程起来 和 hook strstr, 我这就直接 hook str 了.

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V508Sj9MPCkZNrRIsTgTyw0hwjVbheSU9qQy0ngia2B3VUhYiaaMficnhoQ/640?wx_fmt=png)

    然后就不崩了, 可以愉快的 frida 了~  

    先拖个壳, Fridadexdump 一波 (可以直接 fart 脱的全, 但是懒的刷机, 直接用这个了..),hook events 定位

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5V6spHicpuDWpAk92tpJACHNuVpm0dsRhaTUFjicQ2riaicNAgYw8R1s1sw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5YCWmCGiclOcJic13XsJqEN5rNkylqOqBtIrEJAHxrttZWpQCen0edCXA/640?wx_fmt=png)

    oncreate 直接是 native 化了 应该是 360 的 vmp, 这里肯定不是让你逆 360 的 vmp, 盲猜一波就是 check 方法, 先 hook 一下.  

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V53IZQr0nYInQrJyFXq0HajQtia64HM9sw5YRmX0b6iaLBPxmjcY04NuFA/640?wx_fmt=png)

    直接返回 false, 应该是 true 就会拿到 flag, 为了不每次都手动输入 直接写个主动调用.

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5icvqImdg12BtYSbSeHmtXjLx7AWzMCbqBujUibggyasV1DQC2ktFZn0Q/640?wx_fmt=png)

    接下来去看 so 了, 导出函数就这几个.. 肯定是动态注册, 先直接 hook

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5L2pwojyHxz4OicTW7n68ANsdtqBm2E1SpZuZC9H9UfChicnmqEW4mbiaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V53xk5eYAHJdlrtraMb1qNz1kpuHDQQADpIqNDTskMjkqRdjxJ0B06pg/640?wx_fmt=png)

    hook 到了偏移, 去 so 看一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5GPASEJotibrHGFmm6oHc66eNKpXc8FUliaecuiadwPCwwEQTuM9rbbJxw/640?wx_fmt=png)

    改了些名字后 操作流程就是将输入的 jstring 转为 char* 然后判断长度是否为 20 看到很多都是 unk_开头的, 这些就是被 ollvm 加密后的字符串, 要怎么看他解密后的内容呢? 因为他加载到内存的时候肯定是解密状态, 直接读这个地址打印 cstring 就可以得到解密后的字符串了.

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5QmF37PaDev6IvFwF01npBKuibI9a4TL6hAr8Im8gft6yEVEdT8vJbYw/640?wx_fmt=png)

    然后继续改名, 改完名字就是这样

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5ve5BvsHOtyvdcYKLxPcFNXUOGH4k88XD0VkZ1ANJMiciawQibns1wJsOQ/640?wx_fmt=png)

    发现他重新对 check 的方法进行了动态注册, 然后 sub_1234 里面也进行了动态注册里面嵌套了好几次动态注册, 然后用 strcmp 比较 输入的值和 &unk_5100(解密后为 kanxue) 如果相等那就返回 true. 但是输入要 20 个字符 kanxue 只有 6 个字符, 我们 hook 一下这个比较的地方看看,

    直接 hook strcmp 蹦的比较厉害, 我选择 inlinehook, 按 tab 切换到汇编, 打开 opcode,s1 的值给到了 R0 我选择在 0x1564 进行 hook, 因为是 2 个 opcode 所以为 thumb 指令, 需要地址 + 1

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5kAsQ41iaghSeT1weTBaJ9PyeExM2icgIn45Qyv9yNmbsoWckPwp8xVlg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5x18joVCW7ZhUNbyEzje1oIygzAPxSYF162wNyoUxRFk53cNbwGYBrw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5Vjz5RaRQticxKETWhMc6DZmDYZPJI24ZqCINgwN9UtAxHOByjuedOlw/640?wx_fmt=png)

    我们输入了 kanxue00000000000000,hexdump 打印 s1, 发现 s1 就是 kanxue, 但是程序没提示通过, 然后 hook registernative hook 到他又进行了 2 次动态注册, 根据这个地址 我们往前找

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5UfWl8n0Eian5s9LEGSibVWtibEBJaQT2pBkqQ9fG1hau7Ie8C6zUlU71Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5fPsvHdDkvw0DnFIbibZ0BKgp7FYZtuwz1SMaicxOsvrkAFaU3hTBoQsQ/640?wx_fmt=png)

    发现第二次动态注册的和最开始的代码很像, 第三次就短了很多, 然后仔细一看第二次动态注册中又注册了 sub_1148, 原来就是嵌套了 3 次动态注册, 注册了 3 个函数, 根据一开始的分析 对 strcmp 这里的字符串解密  

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5icqp3Uucia5qQ9qI5icdIwVmJjluiaznjbWqxickDj9zO9viaf73AFx4L6tg/640?wx_fmt=png)

然后分别 inlinehook 剩下几个 strcmp 验证下!

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5EXXagDTuNNX2R25qw4lQEevAHQB9augyp6XlscIbQI0zSvKz4nQH5g/640?wx_fmt=png)

    发现第二次比较是输入的 8 个 0 这样区分不了是第几个输入, 我们用 abcd 来实验, 输入 kanxueabcdefghieklfn 20 个字符  

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5rT32Kzc0KrusvQCXLhu80IWhhBoEUSenmyDK4DDoX1g0EQ2uiakm8xg/640?wx_fmt=png)

    是 abcdefgh 这 8 个, 所以是加到 kanxue 后面的, 然后真正的字符串之前我们已经解密了, 是 unk_50F7(即为 training), 然后后面猜也猜得到剩下 6 个字符是之前解密的 unk_5096(即为 course) 为了保证严谨性, 我就在 inlinehook 一次  

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5dricfDbxuF8qt2mYfL2LcsILHsRjE76QQZJesPVAZsNibDJvJpRvhAzQ/640?wx_fmt=png)

    果然就是我们最后六位进行比较 所以答案前面的解密的字符合起来, 就是 kanxuetrainingcourse  

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5sx24KmxUY18Z4icG6OHQricRUYZiaqoBow3EroiblAkXLKdq1amPqnFe6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQs6E5OK6eLtbgvvYxp95V5KQEm4z50x8KLbg8bywfMOicgphTLlicqX8E9fK2nF8eRAybUUiaaiaqxpA/640?wx_fmt=png)

    这里有个 bug 每次输错都要重启一次, 不然到最后答案就变成 course 或者 trainingcourse 了~