> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PrMqy9Z51JXxDzveEY2Ejg)

**观前提示：**  

**本文章仅供学习交流, 切勿用于非法通途, 如有侵犯贵司请及时联系删除**

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxj224O5cMn1AQuuBwoy2lpcK2Mia219XQoAZEdWHjx1Bj0x9DqoSEPLA/640?wx_fmt=png&from=appmsg)

0x1 前言

在很久以前 笔者发过一篇文章 [App 逆向百例 | 12 | 某电商 App Sign 分析](http://mp.weixin.qq.com/s?__biz=MzUxMjU3ODc1MA==&mid=2247485239&idx=1&sn=a059748eba368c2d69f429f111d283a0&chksm=f96302e6ce148bf07f848cad76f1e04217c31a88f47dedda8e507bee9beeea774ffad3584c70&scene=21#wechat_redirect) 里面介绍了其中最简单的一套`case2`算法 时隔多日 突然刷到吾爱的一篇文章 看完顿悟了 索性就顺便把里面的`case0`和`case1`出一次分析过程吧

0x2 sub_10EA4 简单分析
==================

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxbiabVLIa7U9nMmgFPyXtsmmgdKkXTBNF9baIFA1ZyFue1Ossej9GKag/640?wx_fmt=png&from=appmsg)本次文章针对`case0`和`case1`入手

首先进入`case0`对应的`sub_10E4C`-`sub_12580`

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxQ8XZKmWjH9nxwqRpxCA66PvIMC7Mqu9ugoUNslRFEic6HVOHvCDuphw/640?wx_fmt=png&from=appmsg)

从伪代码就能看出这大概是传入明文 然后遍历取段落去`sub_10EA4`or`sub_119D4`处理

但测试只会走`sub_10EA4`方法 所以只需要关注`sub_10EA4`即可

进到里面 似乎有点规律![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxHYIIocC4P8JlZuXhNnndaEd8qPNU349PJfEicKtI12apF6WNe54pDHg/640?wx_fmt=png&from=appmsg)

转成伪代码看看![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxQ5v59zJboJ9bghBRB94jJickiaYiarqhPyzoxTos07JY8IQzk2tZEX4wg/640?wx_fmt=png&from=appmsg)

OMG 俩百多个变量 又臭又长

关闭 - 回收站 - 关机

虽然是这么说 但是我可是看了攻略的 我可是有备而来的 得啃下来

从图形视图对应伪代码 可以大致分为三块![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx6Vb797DRYpn2BqvFWWpUI7DXHdwbcVozNaPYdmXPSFiboyCaGaX2r6w/640?wx_fmt=png&from=appmsg)

看看第一块 而且很有规律地重复了 8 次

```
v195 = v5 & 0x80;
v196 = v5 & 0x40;
v197 = v5 & 0x20;
v198 = v5 & 0x10;                   
v199 = v5 & 8;                       
v200 = v5 & 4;                       
v201 = v5 & 2;                       
v202 = v5 & 1;                       


```

其实这个和 0x80 0x40 .. 0x1 计算的作用就是取比特位

例如 v5 假如是 0x6b 那么

*   0x6b & 0x80 = 0x00 -> 0
    
*   0x6b & 0x40 = 0x40 -> 1
    
*   0x6b & 0x20 = 0x20 -> 1
    
*   0x6b & 0x10 = 0x00 -> 0
    
*   0x6b & 0x08 = 0x08 -> 1
    
*   0x6b & 0x04 = 0x00 -> 0
    
*   0x6b & 0x02 = 0x02 -> 1
    
*   0x6b & 0x01 = 0x01 -> 1
    

手动计算的结果是`01101011`和下图一致 验证了我刚刚的猜想![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx38Ys0OkH73jpXMDsCQYtUo2SXkDWbZMYGfQsicC9nicwWJZFdaS82uuQ/640?wx_fmt=png&from=appmsg)

所以 传入 8 个字节经过计算会有 64 个比特位

接着看第二块 这一段很长![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxC2rv7tZTsrRw1ctpJn38FibgoJzicYxehPFLYgqTia4BkxhicDlbHaWrgA/640?wx_fmt=png&from=appmsg)第二块是一个大循环 里面同样出现了取比特位的操作 但是不同的是遍历了 v169

通过引用可知 v169 为传入的 a3 而 a3 对应的是 key![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx3YaIHu7cob0QpBYfjFdYwvhZEib33xkVApqN5R0WYWsiaoGNrmBgFukw/640?wx_fmt=png&from=appmsg)

接着看下面都做了些什么![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxgIb2TYyH4cuNNhj49KFib0fg1hTZPWaicibNfV24ajSN1iarfR628z7J6A/640?wx_fmt=png&from=appmsg)这里进行取比特位后进行判断跟着的是一堆变量赋值的操作 随便去回缩其中的`v141`和`v143`

`v141`->`v207`->`v6 & 0x8`  
`v143`->`v239`->`v10 & 0x8`

可以看出 这段操作就是循环遍历 key 取比特位走不同逻辑 将 64 个比特打散的一个操作

第三块出现了循环 4 而后又出现了取`i`的值和`2*i`的值![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxFVdaUYiaDFBZTNZic0KAIgFRKz8LKLhchzskWhAs1rpL1lo1NwDQ4aWA/640?wx_fmt=png&from=appmsg)

取完值 根据变量是否判断成立 返回 0x80 or 0 接着使用 (|) 或运算 可以知道这是一个二进制转十六进制的操作

还是以上面的 0x6b 为例 bits`01101011`

*   0 -> 0
    
*   1 -> 0x40
    
*   1 -> 0x20
    
*   0 -> 0
    
*   1 -> 0x8
    
*   0 -> 0
    
*   1 -> 0x2
    
*   1 -> 0x1
    

计算结果![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxtNdPlW0FKfNJOicoy9NdaQcj1Hz2Bic4uoOExXIcFmDVol2CNibMEtZHA/640?wx_fmt=png&from=appmsg)所以猜想验证成功

那么总结一下这个方法就是传入 8 个明文 转二进制 将比特位根据`key`去打散 再最终还原为十六进制

0x3 unidbg 辅助分析 sub_10EA4
=========================

前面我们说过根据`key`去打散 而 key 是固定的 那么我能不能由此推断因为 key 固定打散的规律也固定呢？答案是可以的

我们只需要使用 unidbg 控制传入值 记录下打散后的结果对比 就能知道打散的前后位置变化了

例如我传入的值为`0x80`

转化为二进制为`10000000`

假如返回结果为`0x40`

那么转为二进制为`01000000`

根据我们之前的分析情况来看 可以认为他将第一个比特位打散到了第二个比特位

那我们只需要遍历传入

80 00 00 00 00 00 00 00  
40 00 00 00 00 00 00 00  
..  
01 00 00 00 00 00 00 00  
00 80 00 00 00 00 00 00  
..  
00 01 00 00 00 00 00 00  
..  
00 00 00 00 00 00 00 01

就可以遍历所有比特位的打散结果了

unidbg 就可以很好的帮我们实现这个操作

```
    public void callSub_10EA4() {
        hookTable = new byte[64][];
        for (int i = 0; i < 8; i++) {
            for (int k = 0; k < 8; k++) {
                byte[] byteArray = new byte[8];
                for (int j = 0; j < byteArray.length; j++) {
                    byteArray[j] = (byte) 0x0;
                }
                byteArray[i] = (byte) (0x80 >> k);
                hookIndex = i * 8 + k;
                ArrayList<Object> args = new ArrayList<>(10);
                args.add(0);
                args.add("44e715a6e322ccb7d028f7a42fa55040".getBytes());
                args.add(32);
                args.add(byteArray);
                module.callFunction(emulator, 0x10EA4 + 1, args.toArray());
            }
        }
        System.out.printf(Arrays.deepToString(hookTable));
    }


```

再加上我们的 hook 代码 将打散后的比特位去对比

看回`sub_10EA4`

此处为打散完的结果 再往下看是二进制转十六进制的循环操作了![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxeXfRRRDMJNzBAbIwnZsYZ0nW5vEbSohe8QQiawJL5SlzjCJicPicia3wLA/640?wx_fmt=png&from=appmsg)

所以 `0x118CE`是一个很好的 hook 点

我们只需要 hook 拿到`r3`寄存器的值就能拿到打散的二进制结果了

```
public void hook10EA4() {
        Debugger attach = emulator.attach();
        attach.addBreakPoint(module.base + 0x118CE, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                RegisterContext context = emulator.getContext();
                byte[] r3 = context.getPointerArg(3).getByteArray(0, 64);
                Inspector.inspect(r3, "r3");
                int cnt = 0;
                for (int i = 0; i < r3.length; i++) {
                    if (r3[i] != 0) {
                        break;
                    }
                    cnt++;
                }
                hookTable[hookIndex] = new byte[]{(byte) hookIndex, (byte) cnt};
                return true;
            }
        });
    }


```

运行拿到结果 例如 [1,4] 为将第一位的值放到第四位上

```
[[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2], [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21], [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55], [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41], [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28], [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50], [63, 48]]


```

所以复现一下就是

```
def sub_10EA4(input_bytes):
table = [[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2], [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21], [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55], [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41], [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28], [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50], [63, 48]]
bits = bytes2bit(input_bytes)
resbit = bytearray(len(bits))
for i in range(len(bits)):
  resbit[table[i][1]] = bits[table[i][0]]
return bytearray([bit2bytes(resbit[_:_ + 8]) for _ in range(0, len(resbit), 8)])


```

具体验证等细节就省略了 读者们自行验证吧

0x4 sub_10D70 简单分析
==================

接着看后面

在`sub_12510`循环结束后会进入`sub_10D70`方法

![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx02ntl4rdOY9nsNRaVqicYx0Xiac6qOicWQEPicoXOtm2jrYndiaTzXqKaVA/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxJUVBSbxqzibpN72tYFBhldIdtprfGQNgAoMqMUtP1eaibcVgxewGGXqw/640?wx_fmt=png&from=appmsg)

根据 hook`sub_10D70`的入参可知 每次 8 个字节 8 个字节操作后 剩余字节会根据其长度 - 1 走不同的 case

这里以`case 0`为例 根据 hook 可知其传入内容长度为 1 个字节 进到`sub_4B7C`里面细看![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxEIjjjebmsDKXFVMuAsxAV4o9OINUydj4fKuA2EAUOOlq1ZzltC9sicA/640?wx_fmt=png&from=appmsg)这是 OLLVM 吗？但是里面并没有出现什么 OLLVM 的特征。

看看伪代码 进来一看 好像也是取比特位？但是又有其他的计算操作![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx6b6sQds55yZ35H7TVCLtCwUPboDCBTjxpLEYm71VnRrQvLhSicGVXUg/640?wx_fmt=png&from=appmsg)假设 方法开头也是转为二进制的操作 那么 方法的结尾按理也是还原十六进制 翻到最后看到 好像确实如此![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxUSibn6pWfTITHg9WkpQkia26KSJR6VKicztH7iccrxI8hImHFs8xqEddEg/640?wx_fmt=png&from=appmsg)那么他中间做了啥 回到中间那一部分的开头看到

这又是一个遍历的操作![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxyZzHs3qnS81szHIB9zCdlZQibbiaJTAEIL0mBib0S8F2khX1kUUWQkTDQ/640?wx_fmt=png&from=appmsg) v354 为传入的 a1 而传入的 a1 为 key 这熟悉的感觉![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxKQOb35ibFfD7jjXw9icFqOvYvEapIPK4b3RDWnNrZiaajDW4dcQPAnHZQ/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxpVibT6Ke2bKvrRDia8Zsto59A7wLh8RmCwqjUEXf1uib1t6k6V3Ljric7w/640?wx_fmt=png&from=appmsg)

这里我大胆假设：开头类似转二进制 中间遍历 key(key 是固定) 结尾类似转十六进制 那么和我们上面分析的`sub_10EA4`是不是也是如出一辙的打散操作呢

带着疑惑 上 unidbg 调试看看

0x5 unidbg 辅助分析 sub_4B7C
========================

先码入一个主动调用传入一个`0x0`

```
public void callSub_4B7C() {
    byte[] byteArray = new byte[1];
    for (int j = 0; j < byteArray.length; j++) {
        byteArray[j] = (byte) 0x0;
    }
    Inspector.inspect(byteArray,"input");
    ArrayList<Object> args = new ArrayList<>(10);
    args.add("44e715a6e322ccb7d028f7a42fa55040".getBytes());
    args.add(32);
    args.add(1);
    args.add(byteArray);
    module.callFunction(emulator, 0x4B7C + 1, args.toArray());
}


```

再下一个 hook 断点

```
emulator.attach().addBreakPoint(module.base + 0x5282);


```

断下来后看到返回的结果是`0x48`

接着再去拿到传入 0x80 0x40 0x20 的结果 做一个对比

```
input output
0x00  0x48 | [0, 0, 0, 0, 0, 0, 0, 0] [0, 1, 0, 0, 1, 0, 0, 0]
0x80  0x4a | [1, 0, 0, 0, 0, 0, 0, 0] [0, 1, 0, 0, 1, 0, 1, 0]
0x40  0x40 | [0, 1, 0, 0, 0, 0, 0, 0] [0, 1, 0, 0, 0, 0, 0, 0]
0x20  0x4c | [0, 0, 1, 0, 0, 0, 0, 0] [0, 1, 0, 0, 1, 1, 0, 0]


```

从返回的结果看 当我们传入 0x0 的时候 返回 0x48 可见 他在里面还是做了除了打散之外的计算操作的

那我们就从传入前后的二进制入手

以第一个返回值会基准 和每个结果单独拉出来对比

0, 1, 0, 0, 1, 0, `0`, 0  
0, 1, 0, 0, 1, 0, `1`, 0

0, 1, 0, 0, `1`, 0, 0, 0  
0, 1, 0, 0, `0`, 0, 0, 0

0, 1, 0, 0, 1, `0`, 0, 0  
0, 1, 0, 0, 1, `1`, 0, 0

从中可以看出 每次结果都只会变化一个比特位 而且 如果变化前的 bit 为 0 返回则为 1 反之 1 变为 0

总结一下就是 [[6,0],[4,1],[5,0]...]

这里的数组意思就是第 0 为会被打散到第 6 位 而且记录了初始结果第 6 位的值 例如：

当第一个比特位为 1 结果会打散到第 6 个比特位 如果初始结果的第六个比特位为 0 那么结果为 1 反之为 0

```
# 以第一位比特位为例
index = 0
table = [[6,0]...]
inputBits = [1, 0, 0, 0, 0, 0, 0, 0]
result = [0, 0, 0, 0, 0, 0, 0, 0]
if inputBits[index]:
  if table[index][0]:
    result[table[index][0]] = 0
  else:
    result[table[index][0]] = 1
else:
  result[table[index][0]] = table[index][1]


```

验证了这个结果可行 那就手动获取这 8 个字节的打散顺序吧

根据刚刚的 hook 断点 再加上我们手动对比出比特位的交换位置 得到结果`[[6, 0], [4, 1], [5, 0], [0, 0], [2, 0], [3, 0], [1, 1], [7, 0]]`

翻译成 python 就是

```
def sub_4B7C(input_bytes):
    table = [[6, 0], [4, 1], [5, 0], [0, 0], [2, 0], [3, 0], [1, 1], [7, 0]]
    bits = bytes2bit(input_bytes)
    resbit = bytearray(len(bits))
    for i in range(len(bits)):
        resbit[table[i][0]] = (0 if table[i][1] else 1) if bits[i] else table[i][1]
    return bytearray([bit2bytes(resbit[_ * 8:_ * 8 + 8]) for _ in range(len(input_bytes))])


```

验证结果和 unidbg 跑出来的结果的一致![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxT0hqYSOySfHlkz47eVbPg5310d5wEOJ0tuYOnH6XFDpCaOyz6cIkcA/640?wx_fmt=png&from=appmsg)

0x6 其余 case 分析
==============

看回 case 1 `sub_61A0` 也就是传入参数的长度为 2 字节

2 个字节处理成比特位连带计算![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx3s7HRibkAD8tkLAcGsLwicJUl2jHYZOqIR5YWFw7YDojUOib91JrV2UKw/640?wx_fmt=png&from=appmsg)中间计算方式没变![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxrdnVB0vdTrjSS8kxHXz2oKc194QLpS6Vq8AdheBxgdibznclj3PT2qg/640?wx_fmt=png&from=appmsg)结尾也是还原十六进制 不过就变成 2 个字节![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hx2rknw5bnJKtCGX6zSic0Eibl9dshMfZWw0L08jL9FU8n3GAa57ibL2Riag/640?wx_fmt=png&from=appmsg)所以大致内容没变 不过就是传入和传出都变成 2 个字节 计算的代码量变多了 操作基本没变

多看几个 case 发现也是一样 发现 case 2 `sub_7994`就是传入 3 个字节传出 3 个字节

根据这个 我们就可以写出 unidbg 的 hook 脚本 把他全部位置一次性倒出来

```
public void hookSub_10D70_case(int caseIndex, long addr, int resultRegister) {
        Debugger attach = emulator.attach();
        attach.addBreakPoint(module.base + addr, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                byte[] result;
                BackendArm32RegisterContext context = emulator.getContext();
                if (resultRegister == 3) {
                    result = context.getR3Pointer().getByteArray(0, caseIndex);
                } else if (resultRegister == 5) {
                    result = context.getR5Pointer().getByteArray(0, caseIndex);
                } else if (resultRegister == 6) {
                    result = context.getR6Pointer().getByteArray(0, caseIndex);
                } else if (resultRegister == 7) {
                    result = context.getR7Pointer().getByteArray(0, caseIndex);
                } else if (resultRegister == 9) {
                    result = context.getR9Pointer().getByteArray(0, caseIndex);
                } else if (resultRegister == 12) {
                    result = context.getR12Pointer().getByteArray(0, caseIndex);
                } else {
                    result = new byte[]{};
                }
                byte[] byteArray = new byte[8 * result.length];
                for (int i = 0; i < result.length; i++) {
                    for (int j = 0; j < 8; j++) {
                        if (((0x80 >> j) & result[i]) != 0) {
                            byteArray[i * 8 + j] = (byte) 0x1;
                        } else {
                            byteArray[i * 8 + j] = (byte) 0x0;
                        }
                    }
                }
                hookTable[hookIndex] = byteArray;
                return true;
            }
        });
    }
public void callSub_10D70_case(int caseIndex, long addr) {
        hookIndex = 0;
        hookTable = new byte[1 + (8 * caseIndex)][];
        for (int i = 0; i < caseIndex; i++) {
            for (int j = 0; j < 9; j++) {
                byte[] byteArray = new byte[caseIndex];
                for (int k = 0; k < byteArray.length; k++) {
                    byteArray[k] = (byte) 0x0;
                }
                byteArray[i] = (byte) (0x80 >> (j - 1));
                if (!(i != 0 && j == 0)) {
//                    Inspector.inspect(byteArray, "input");
                    ArrayList<Object> args = new ArrayList<>(10);
                    args.add("44e715a6e322ccb7d028f7a42fa55040".getBytes());
                    args.add(32);
                    args.add(1);
                    args.add(byteArray);
                    module.callFunction(emulator, addr + 1, args.toArray());
                    hookIndex++;
                }
            }
        }
        byte[][] resTable = new byte[8 * caseIndex][];
        byte[] initBits = hookTable[0];
        for (int i = 1; i < (1 + (8 * caseIndex)); i++) {
            for (int j = 0; j < initBits.length; j++) {
                if (hookTable[0][j] != hookTable[i][j]) {
                    resTable[i - 1] = new byte[]{(byte) j, hookTable[0][j]};
                }
            }
        }
        System.out.printf("case %d:\n%s\n", caseIndex - 1, Arrays.deepToString(resTable));
    }


```

运行脚本

```
xx.hookSub_10D70_case(1, 0x5282, 6);
xx.hookSub_10D70_case(2, 0x6AE2, 7);
xx.hookSub_10D70_case(3, 0x849E, 5);
xx.hookSub_10D70_case(4, 0x9DEC, 9);
xx.hookSub_10D70_case(5, 0xBA84, 12);
xx.hookSub_10D70_case(6, 0xD998, 7);
xx.hookSub_10D70_case(7, 0xFC14, 5);

xx.callSub_10D70_case(1, 0x4B7C); // case 0
xx.callSub_10D70_case(2, 0x61A0); // case 1
xx.callSub_10D70_case(3, 0x7994); // case 2
xx.callSub_10D70_case(4, 0x91AC); // case 3
xx.callSub_10D70_case(5, 0xABF8); // case 4
xx.callSub_10D70_case(6, 0xC8C0); // case 5
xx.callSub_10D70_case(7, 0xE7FC); // case 6


```

运行拿到结果

```
case 0:
[[6, 0], [4, 1], [5, 0], [0, 0], [2, 0], [3, 0], [1, 1], [7, 0]]
case 1:
[[5, 0], [9, 0], [0, 1], [7, 1], [10, 0], [6, 0], [13, 1], [1, 0], [4, 0], [11, 0], [14, 1], [3, 1], [12, 0], [15, 1], [8, 0], [2, 0]]
case 2:
[[17, 0], [7, 0], [5, 0], [19, 1], [18, 0], [15, 1], [22, 0], [21, 0], [16, 0], [4, 0], [12, 0], [2, 1], [10, 1], [13, 1], [20, 1], [8, 1], [9, 0], [23, 0], [11, 1], [6, 0], [1, 0], [3, 1], [0, 1], [14, 0]]
case 3:
[[25, 1], [4, 0], [29, 0], [1, 0], [27, 1], [18, 1], [23, 1], [14, 1], [28, 1], [11, 0], [9, 1], [13, 0], [24, 1], [0, 1], [5, 0], [2, 1], [26, 0], [12, 0], [31, 1], [16, 1], [30, 0], [15, 0], [10, 0], [22, 1], [7, 1], [21, 0], [6, 1], [3, 1], [8, 1], [20, 0], [19, 1], [17, 0]]
case 4:
[[11, 0], [12, 0], [28, 1], [30, 0], [13, 1], [24, 0], [22, 1], [25, 1], [23, 1], [3, 0], [16, 0], [8, 1], [34, 0], [2, 0], [5, 0], [7, 1], [4, 0], [14, 0], [39, 1], [33, 0], [15, 0], [0, 0], [31, 0], [9, 1], [29, 0], [26, 1], [19, 0], [6, 1], [27, 1], [10, 1], [37, 0], [38, 1], [20, 0], [21, 1], [1, 0], [36, 0], [32, 0], [17, 0], [18, 0], [35, 1]]
case 5:
[[11, 0], [45, 0], [15, 1], [22, 0], [10, 0], [7, 0], [3, 0], [42, 0], [17, 1], [21, 0], [4, 0], [8, 1], [19, 1], [32, 0], [28, 1], [31, 1], [29, 0], [14, 1], [39, 1], [27, 1], [2, 1], [24, 0], [26, 1], [9, 1], [41, 0], [1, 1], [47, 0], [44, 0], [23, 1], [0, 1], [12, 1], [18, 0], [33, 0], [36, 0], [40, 1], [34, 0], [25, 0], [16, 1], [5, 1], [35, 0], [38, 0], [37, 1], [13, 0], [20, 1], [6, 0], [43, 0], [30, 0], [46, 1]]
case 6:
[[7, 1], [9, 0], [53, 1], [19, 1], [15, 1], [8, 0], [3, 0], [24, 1], [18, 0], [51, 0], [42, 1], [39, 0], [20, 0], [12, 0], [28, 1], [27, 1], [23, 0], [49, 0], [10, 1], [55, 1], [52, 1], [17, 0], [48, 0], [14, 1], [33, 0], [25, 1], [4, 1], [11, 0], [47, 1], [0, 0], [21, 1], [44, 0], [16, 0], [41, 0], [29, 0], [1, 0], [46, 0], [5, 0], [30, 0], [45, 0], [31, 1], [43, 1], [36, 1], [26, 0], [34, 0], [2, 0], [6, 0], [50, 1], [13, 1], [37, 1], [32, 0], [40, 0], [35, 0], [38, 0], [54, 0], [22, 0]]


```

完美导出 这个结果还得读者自行验证

0x7 sub_10E4C 小结
================

case0 对应的`sub_10E4C`

key 固定为`44e715a6e322ccb7d028f7a42fa55040`

明文传入`sub_10EA4`循环处理后 剩余字节进到`sub_10D70`继续处理

最终得到`sub_10E4C`的结果

0x8 sub_10E18 简单分析
==================

在`sub_126AC`中的 case 1 对应`sub_10E18`-`sub_12510`

key 固定为`7d0069660c9b5d32074facf37c3738a1`

进到`sub_12510` 发现和`case 0`走到逻辑一模一样

也是进到`sub_10EA4`循环 剩余字节进到`sub_10D70`处理![](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMQvg4pFqxPb0qOPBc534hxlWH9GVr21AFJuiaBnzsxZO3KpVTaUbuHMHib3XRQ55gv56ZMqSomKiaow/640?wx_fmt=png&from=appmsg)

和之前不同的是 key 变了

根据前面的分析可知 key 和打散位置有关 所以只需要将前面给出的代码中的 key 变换成`case 1`的 key 即可拿到`case 1`对应的打散位置

代码都有 所以这一部分交给大家自行操作吧

0x9 后记
======

相较于以前写的 case2 的文章 case0 和 case1 总体代码量冗余 流程复杂 好在这俩个 case 的代码有共性

文章很长 内容很干 得多次反复理解 也许我也没写得那么清晰 甚至有错误的地方 请大伙见谅 感谢大佬的文章给我提供的思路 使我可以在巨人的肩膀上实践 膜拜

0x10 参考文章
=========

https://bbs.kanxue.com/thread-266377.htm

**感谢各位大佬观看**

**感谢大佬们的文章分享**

 **如有错误 还请海涵**

**共同进步 带带弟弟**

点赞 在看 分享是你对我最大的支持  

逆向 lin 狗