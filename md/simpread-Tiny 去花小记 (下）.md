> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4Gn_0yXEj-W3FQ2qH2ibog)

这篇讲如何在花指令混淆的 so 中追一个设备指纹来源

首先你得知道他上传的设备指纹原文是啥吧？

不管你是 hook 还是抓包解密都能拿到他的设备原文, tiny 应该人均水平有手就行 (手动狗头)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4GFO9aibk1Ot3kjLWakicry9jkyARWG0fu2ROYgfTEzIOF7JuKCQArgdw/640?wx_fmt=png&watermark=1#imgIndex=0)

解密设备指纹请求后就能得到上报的明文数据 有些看值一眼就能看出是什么，但是还有些值没法看出来就得逆他生成逻辑了

就拿这个 x46 来举例

定位数据的话我们得先找到他数据存放的代码，他把数据放进一个 json 里肯定都会用同一个 set 代码放入 只需要找到这个点 然后 hook 打 lr 或者调用栈就能快速回溯这个值生成

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxU1238XpxlQAsf41XFnTQTNU6x6bFiavBmNL7YCQGEmInictJIKWYHNSng/640?wx_fmt=png#imgIndex=1)

这个脚本是不能公开的 比较敏感 但是去花之后看看逻辑也明白了

到 lr 这个地址 F5 报错

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4riaJuytHiaLibM42eCQ20iaPPp0xPCSMEMvbaryXQD8DDibSuNAqliaLFmLg/640?wx_fmt=png&watermark=1#imgIndex=2)

1ABE7C 这个地址调用有问题 是调用一个函数 点进去看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxUfUX3Et1e7KkoomWlOWaNMpH72pN5WRUpXcdMP33D9KP5HKiaqVI2OUQ/640?wx_fmt=png#imgIndex=3)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4Uh4l7icC6HuViaGuUWBPax0qcyGGrzR7rsBtzS6LypXLDyKl8l7hUXNg/640?wx_fmt=png&watermark=1#imgIndex=4)

这个就是 cxa_guard_release 函数 和上一篇的线程锁作用相反 释放线程锁

给他重命名上 那么为什么会报错呢 盲猜就是 ida 给他参数搞太多了 其实入参就是一个按 y 给他其他入参都去了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4ZIFy0TRtcVCjmTKPcJNQMS7VZiausfxmj9k3Y904BlYeHeSn2x5JWcQ/640?wx_fmt=png&watermark=1#imgIndex=5)

然后回到他刚刚 lr 的地址 F5 就正常了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4c3ScADWDFHAPx67e5AVvYvNn6So7nMSniaQnoPnWf3JeG5JNzIeIsMw/640?wx_fmt=png&watermark=1#imgIndex=6)

ok 很恶心的代码 啥也看不出来

我们还是先确定函数范围 然后用上一篇的脚本 hook 实际跳转地址，再用 ida 脚本 patch br blr 指令

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxUpo2BMOPBDSZoZDp9oPLBJNP5xE5eiad9ziaMFbBNQNduO91Kmm8kNDHA/640?wx_fmt=png#imgIndex=7)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxUscK7ZV9fgFL05R14xrSaycZQuw3sck1kF0rWr3SERecst7KDsIGcWg/640?wx_fmt=png#imgIndex=8)

可以看到 ret 和 BL .__stack_chk_fail 在 0x1abea8 之后的地址我们选大的用 0x1b7610（可以 hook 多一点 信息更多）正好下面也是一串 STP 函数的开栈指令

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H44WvBBqMsNiapicf7gp9ARNYMfknWY1dPdL0HN9cxqkib1OVAw2qmUCrBQ/640?wx_fmt=png&watermark=1#imgIndex=9)

往上的话可以选择 1aae24 这个函数的 ret 和 BL .__stack_chk_fail 看看

正好 BL .__stack_chk_fail 往下翻翻也有开栈指令 那就拿他当起始地址

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxU43JyZUxWdDT25lZVN4SC3xIteFd0YEWb0oXF1tnnqGvd5u87gnVicSw/640?wx_fmt=png&watermark=1#imgIndex=10)

0x1ABB40-0x1b7610 这个地址范围的 br 和 blr 指令 直接用上一篇文章拿到的指令复制出来 hook

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxUu0cZjaBia7C5I1pA2ltDv0jWbgWxVXgkKeiaxzmfIyA4RhB0sic3QaU9Q/640?wx_fmt=png&watermark=1#imgIndex=11)

也是挺多跳转的 hook 一把梭

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4N4LmyvueaQkaeiaW1fhjNhKpmITtU3uLPMWHLC1RMPEORCrGaleKuaA/640?wx_fmt=png&watermark=1#imgIndex=12)

等他不动了再退出保存日志 也可以抓包等上报了这个信息之后再停止

然后 patch 回去 去看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4wuN3afTvP9JSVicyU9icMZpPFAEQOSUWEIDZsMHG3q7ps0ic8lROYRdbw/640?wx_fmt=png&watermark=1#imgIndex=13)

可以看出一些函数调用出来了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4ZlFbcK7B1pjFerEaySZb33CDDmAfuIcsm50cNjoYdPJvflWdpdMDXA/640?wx_fmt=png&watermark=1#imgIndex=14)

lr 对应的是这条 v5 = *(_BYTE *)v4;

我找的 hook 点就是 25C01C 他就是 json 相关存储的函数里面一堆红黑树 链表 节点插入的逻辑 自实现的 json 逻辑 给他改个名字 json_set_func

26DC50 这个函数呢

实际上他是一个字符串解密函数 直接主动调用 (unidbg 或者 frida 都可以 反正没有入参很简单）就能得到返回值

这里返回当然就是我们的 x46

这里只出现了 key 那么 value 呢

当然出现 set 的时候肯定是 value 也生成了 往下找其实就行，当然这里提示一下这个 STACK[0x3F8] 其实就是 x46 的值 可以自行验证

往上找一下谁调用了 1ABE4C

明明上面就是调用的指令 为什么找不到交叉引用呢

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4ibiaBsGSU7WsIYUrb4QNatNhQ5p3agbuJET099sFl76ns0nf0gEG6icRQ/640?wx_fmt=png&watermark=1#imgIndex=15)

patch 完之后 最好 Reanalyze program 重新分析一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4I1E06CPhsW44kFa6MRtb01OyDQYYiblictCIBIZeXWguvnru8Hepy9pA/640?wx_fmt=png&watermark=1#imgIndex=16)

分析完就关联上了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4UbGKeib6mqeVKvhea5GYsOFE36Aic7n6jFB522ibbEEiaqcAGUGSaXzaDw/640?wx_fmt=png&watermark=1#imgIndex=17)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4PMWK8rMGeibohUERwl7LiaBAdxicoV1ZLicdBLPcRVjR8PEgAngBcv3ic8w/640?wx_fmt=png&watermark=1#imgIndex=18)

这里只是一个加锁逻辑 然后继续交叉引用发现又找不到了 为啥？

因为出现了多个分支的情况 在汇编里就看到了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4fvA9ZC5JScVXtkIMibWIZU6gTSGTuW7ySmia0D1Yy9P6r8VcBlw6ZdRg/640?wx_fmt=png&watermark=1#imgIndex=19)

这时候如果找不到哪里调用的 可以去 hook 日志里搜一下 也能知道

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4MOtomiaRqWVRskdxqiahcu5lZYsDVfDboykqogYPCcgKh38glTw20pYg/640?wx_fmt=png&watermark=1#imgIndex=20)

我们看下这个分支调用的第二个地址是干嘛的

他直接跳到了

.text:00000000001ABE80                 LDR             X8, [X23,[#0x3B0](javascript:;)]

跳过了签名逻辑 对应 F5 代码中的

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H45Pwl428CuH6JVSA3yWN8kqian01SlfCg6bgJeSdAZJ6HAhkAkkRFPrQ/640?wx_fmt=png&watermark=1#imgIndex=21)

说明他跳过了线程锁的逻辑 已经解密了这个字符串

为了方便看代码 1ABE20 直接 patch 为 b 0x1abe24 

可以看更多的代码 更有利于分析

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4mqzqeD2WrUlbicGTYGCIrTw2tpIHGRGUgHpQzS6f5zVxhkvWup4abzA/640?wx_fmt=png&watermark=1#imgIndex=22)

然后看这个函数

发现刚刚我说的 STACK[0x3F8] 就在这里初始化了 而且 1796BC 这个函数入参也是他 那么看看这个函数是什么

一些比较简单的逻辑 都是他自实现 libc 里面的系统函数 反汇编代码丢给 ai 看看他就告诉你了 我这边就直接标出来了 就是复制入参 1 的值到入参 0 上

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4QM57W4bVKibE3aOQzDUX2RU8qVpFv2BQpny3IJg2b5PO2uv4H7HssibA/640?wx_fmt=png&watermark=1#imgIndex=23)

那么就是说 unk_6755A0 是关键

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4FAqF01r7EGJheicP84YVASmicmGeO1E6j9lGw6UsKVmFZ8e2K6z9oicFw/640?wx_fmt=png&watermark=1#imgIndex=24)

交叉引用却没有他赋值的地方。。这咋办 那只能继续往上找

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4bYH0TQu579SQkvJnic3Yxr41G3sxzyfQzcB70eapty2vkOZoOe3znAg/640?wx_fmt=png&watermark=1#imgIndex=25)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H422aicjWER1CjYcvdlngM5yb8bf0oDWwjMTxhcsV7tn9uFUKZqAuANZg/640?wx_fmt=png&watermark=1#imgIndex=26)

这里也有个函数调用 但是先别急 我们可以先看看前面这个

(v1 + 0x3A0) + 0x58A66434LL)

(v1 + 0x3A0) + 0x58A6643CLL)

这两个是什么，首先得先确定 v1 是啥 ida 这里标黄色了 说明不在这个函数里赋值的 在汇编里找找 x23 寄存器哪里赋值的（v1 变量注释了是 x23 寄存器）

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4stjsbbFKUbq9WyVv8fsCCiaN1GI28mW4JsN0qTH7bKiaa2moPo3zicNlA/640?wx_fmt=png&watermark=1#imgIndex=27)

往上翻翻就找到了 这里有个不同跳转的逻辑 可以发现和刚刚的差不多

也是加锁逻辑 所以 patch 成 b 0x1abd18

那么这样我们就可以把 1ABCD8 函数和 1ABD40 连起来了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H49drmHOLvQUv8jp5qqacDWGxv1mbpUia89uicT6yicbkVpjIOVmMD3UUCw/640?wx_fmt=png&watermark=1#imgIndex=28)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H42hDt6mcnlwOO76GQrbe2JoG4t1B8VSNFvwh6nxLYDVd7j33s1ibNxDA/640?wx_fmt=png&watermark=1#imgIndex=29)

就成功计算出这个 v1 + 0x3A0 = off_6323A0

那剩下来的是什么意思呢

看汇编可以知道是读 off_6323A0 的值 再加 0x58A66434 算出一个地址 把 sub_2823D8 这个函数的返回值放进去

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpRhypZ5eWHjLyMgfUv8dnxUUPxibib61PAkicvHZdemmQQGBtjfovYGR0IricicS1W01gAxdmchFxJ9XFQ/640?wx_fmt=png#imgIndex=30)

是个 data 段的 也有个值 计算一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4NUcU2qazoBH0MfCF5ficNZNqpUZUcC0iaqfWXbuhxg2Ild98lGRV7ExA/640?wx_fmt=png&watermark=1#imgIndex=31)

这不就是刚刚说的 unk_6755A0 吗

ida 这样显示还是不好看 要如何让他自动帮我们计算这个地址呢

这里的 data 段因为是可读可写的 ida 觉得他不是固定的数据所以不会自动计算 这里直接给 data 段的权限改为只读

Edit->Segments->Edit segment 去掉 Write 的√

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4oU906RCibfDDaibzw5icxwnFBvBTGskyu461tdMlybwziaq1r1RhZcBrYg/640?wx_fmt=png&watermark=1#imgIndex=32)

    然后重新 F5 ida 就自动计算出来了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4tDWEHV0pC0SZJgibSJuzGYymAInibhyAWjX1kvd9eaQRTvxtlhUcRGCQ/640?wx_fmt=png&watermark=1#imgIndex=33)

但是这里交叉引用还是没关联上这里赋值的代码 这是为啥呢

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4HkwsgzVwAfjIUhnibM8X77GlbWf77gqW2job1UQ0jWPHAXs4GhJicQxg/640?wx_fmt=png&watermark=1#imgIndex=34)

因为这里我觉得还是 IDA 比较保守 觉得这是间接跳转不是很直接 所以没有给我们关联上

所以我们直接一点 按如下的指令去 patch

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4NbQWaXvUiaxFJZnt6zsNlPslGMkByuPUKBFGhEgZKjpHApuYtNq7XAQ/640?wx_fmt=png&watermark=1#imgIndex=35)

patch 完重新 ida 分析下 虽然 f5 没变 但是交叉引用出来了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4HUdYs9Z8FYBHjPAyKW63jLTl2BDvGH9qd2qNQanHVgyeJ3pghp3KQw/640?wx_fmt=png&watermark=1#imgIndex=36)

这样手动不太方便 因为我这是结果导向出发，万一我找不到赋值的点呢，所以这里我觉得可以写一个 ida 脚本去识别类似的指令然后 patch 成我写的这种指令 就能让 ida 强行交叉引用上了

那么接下来只需要看 2823D8 的逻辑了

静态看了这么久 其实应该 hook 再确认一下这里的数据是否出现

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4y82c5HnXcLb3WYiczJ35OBEgmTaC3eyPeuicU51Ub6iaxd9OLLWkmMibqw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=37)

就是 x46 的数据 我们静态分析的没错

进入这个函数看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4JGUwPZdm5WIIdmUN5u0I1I86SZW33TpR2UW5ITa8TkXCibZuuDeuH2g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=38)

点进 sub_26E32C 看了下 这和我一开始说的 x46 的字符串解密函数格式差不多 主动调用下就可以拿到解密后的字符串了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4IXHovF1n9icAicGdEwRBY4hVGxHpC1ia2e8baTaEjjjqzaRhdm4muickwQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=39)

那这里其实已经知道这个指纹是 android_id 了 但是为了严谨 我们继续跟一下代码看看他是如何获取的

继续进入 sub_28424C 函数

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4Q14klc27h9dt6fXyxfNykrN2EMWWla4ibhnjTl9ZrhJwib75HsicH9BSA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=40)

这里我们只知道第 2 个参数是什么 第 0 和第 1 个还得确认一下

可以看到参数 0 很多加比较大偏移方法调用 这种直接转 jnienv 试试运气

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4mzLD5kibibcvA4EibfgfTpRCeCplib86IeBWLXib0KKgG528ibDumnoFhBQA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=41)

直接解析出来了 那么第 0 个就确定了 第 1 个参数先不着急 就一个地方用到

先看看 4B2E44 这个函数干了什么 入参也是 env

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H41FRpBYibicT62VKicthXUuxqbCrxnMgBeVKfjN5CcfXXmTzPNJdUFF77w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=42)

一眼就看出逻辑了 初始化一些方法 id 有写过获取 android_id 代码的同学看到这一眼就知道他是如何获取的了

重命名一下 后续用到这些全局变量的时候更好看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4sD7UqoVSbTMLJBL0AUAHRVaBV8NRNvfeJIP1OIDmv3RHuibaeg8xdBw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=43)

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4AiajS2yVXtjsm4KV9D4Nm6iaoweLxcTGEvkN8gmGU7hD9T4ib63sgwxfQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=44)

可以看到这个函数的入参就出来了

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4I1M5iccSGr7gHCtvbE8dQSHecdWyJwKvUIMhhE4VZAcq5JGxeZ3n8iaA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=45)

点击去一看就知道他是 jni 的 CallStaticObjectMethodV 方法

所以逻辑大概清楚了 就是使用

android/provider/Settings$Secure 的 getString 方法传入 android_id 获取的安卓 id

那么让 ai 给我写个 demo 代码

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4Oskv7knUCbUJ61ricJaegctYLIKibNPG2diaiad2Pgn5r5eBggt9qiaYmMw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=46)

所以刚刚的没解析的参数 1 就是 ContentResolver 实例

跑一下看看

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H4kn8ysMqqeIqSXFfEbXNzQcBpia9sM9Yk73CU2KMs3OZaic0mF0UYEItQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=47)

为啥和刚刚 app 获取的 android 不一样 难道我们找错了吗？

直接问 ai

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H43aa6Z54MicNHPrXYicp20JLd2VLu7Qe5sDsC51PiclwNTrj6Ef1utfyQw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=48)

那么让 ai 写一个 frida 获取 android_id 的代码在原 app 里验证一下

![](https://mmbiz.qpic.cn/mmbiz_png/J7WeDiaX8bpQblEXKJys7YVibxZcOrb9H43IiaUPHkbooUVq2Qqvb7XrmexnNK8wN4oHQPZTfnCTrXUwVJhVOCfWA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=49)

对应上了 至此一个设备指纹获取逻辑就还原完毕。

主要还是得对 ida 操作和汇编指令熟悉 就能得心应手的还原 然后就全是体力活了