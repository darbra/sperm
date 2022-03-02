> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-266377.htm)

> [原创] 记一次 unicorn 半自动化逆向——还原某东 sign 算法

前言
--

关于题目。

 

题目是帖子快写完时候想到的，unicorn 节省了极大的劳动力，当然也可以使用其他的虚拟执行引擎，但是使用 unicorn 你只需要`pip install unicorn`，然后建一个`py`文件就可以快乐地写代码跑了。

 

为什么是半自动化逆向？

 

因为需要自己写代码去控制`unicorn`怎么执行。

 

为什么会想起来还原某东 sign 算法？

 

因为已经有很多文章来分析怎么直接调用某东的 sign 算法了，比如使用 frida rpc 调的：某东直播弹幕实时抓取 https://www.52pojie.cn/thread-1332545-1-1.html。

 

让我尤其佩服的是一个老哥把它当作了练手项目玩出花了...

 

虽然老哥的文章有点散，但是入门极其友好，大家感兴趣可以去学习下，给老哥捧下场。

 

然后刚好最近想提升自己的汇编分析水平，于是就上手分析了。

 

http://91fans.com.cn/

 

![](https://bbs.pediy.com/upload/attach/202103/802108_TS5XXDGHN6UXKVP.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_4M55B83SFSBWHKQ.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_BBZBCWBKRGD5SP8.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_NU6TGMCD6NGMCU6.png)

分析准备
----

### 集成 so 文件

为了方便分析和调试，我选择了主动集成生成 sign 的 so 文件到自己写的 apk 中，然后主动调用。

 

可能是 gradle 版本的问题，搜索到的文章大都无效，踩坑十分多。

 

最后配置成功主要是根据两篇文章：http://www.sorgs.cn/post/7510/，https://blog.csdn.net/u011106915/article/details/106543464。

 

其实配置很简单，在`src/main/`目录下建立`jniLibs/armeabi-v7a/`目录，把 so 文件放在里面。

 

然后配置好`build.gradle`和`CMakeLists.txt`就行了，当然，不同的`gradle`版本会有不同的问题，自己多搜索折腾下就好了。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_WVRP2V7X99YHRN4.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_NA35CRWW7BNKZFX.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_YZ93PGHJFXBAQWJ.png)

 

以及 so 文件中会有两处简单的校验，直接 nop 两条跳转指令就解决了，十分简单，就不多说了。

### 总加密流程

总的加密函数执行流程：

 

sub_127E4——>sub_126AC——>sub_18B8——>sub_227C

 

核心的加密算法都在 sub_126AC 中，传入待加密字符串，待加密字符串长度和两个随机值，

 

![](https://bbs.pediy.com/upload/attach/202103/802108_YQK9PD9PD74BF93.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_Z9X7DWZWJEZYRU3.png)

 

sub_126AC 会返回加密后的字节，然后 sub_18B8 进行 base64 加密，最后 sub_227C 会进行标准的 md5 加密。

### [](#sub_126ac——加密选择)sub_126AC——加密选择

sub_126AC 只是加密的入口，会根据传入的两个随机数选择加密方式和相应的 key。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_KCEQUAT3YRNTQXB.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_DGH65P7KN78N5TT.png)

 

我们定义三种加密方式为 version0，version1 和 version2，其中 version0 和 version1 加密流程全部一样，只不过里面函数传入的常量 version0 是 32，version1 是 16，我们在分析过程中以 Version0 为例。

Version0 加密
-----------

### 执行流程

加密函数执行流程：

 

sub_10E4C——>sub_125F0——>sub_12580——>sub_10EA4——>sub_10D70

 

待加密字符串会被每 8 个字节分为一组传进 sub_10EA4 进行加密，如果最后还有字节剩余，会单独进入 sub_10D70 加密。

### sub_10EA4 比特位初始化

首先看下 sub_10EA4 函数的流程图，先进行初始化，接着有一个循环，循环次数是传入的 key0 的长度。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_VZQ8TBJPMHW8XQC.png)

 

最上面的两个长框框就是初始化过程，做了什么呢，IDA 进行 f5 后比较简洁，我们截取一部分看下。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_67CUG224YCBE68U.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_H9X5AWGJYCRHATC.png)

 

我们注意到分别传入的 8 个字节分别和 0x80（b'1000 0000'），0x40（b'0100 0000'），0x20（b'0010 0000'），0x10（b'0001 0000'），0x8（b’0000 1000‘），0x4（b'0000 0100'），0x2（b'0000 0010'），0x1（b'0000 0001'）进行与操作，然后赋值给一些变量，我们观察这些与操作的对象，会发现其实很有规律，这些变量正是输入的 8 个字节的 64 个比特位，后面会进行打乱比特位然后还原。

### sub_10EA4 打乱比特位

然后我们来看流程图中很有规律的十六个小框框，

 

![](https://bbs.pediy.com/upload/attach/202103/802108_RK9SW9TC24VX7H4.png)

 

截取一些 IDA 进行 f5 后的片段进行分析。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_3GNZCGXZR98V3HW.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_C4BYJX2NP7VA4WP.png)

 

其中 key0_i 是 key0 的第 i 个字符，在循环中 key0_i 也会和 0x80（b'1000 0000'），0x40（b'0100 0000'），0x20（b'0010 0000'），0x10（b'0001 0000'），0x8（b’0000 1000‘），0x4（b'0000 0100'），0x2（b'0000 0010'），0x1（b'0000 0001'）进行与操作，结果作为条件判断，也就是判断 key0_i 二进制的对应比特为是 0 还是 1。然后会进入对应的分支交换初始化过程中得到的变量，也就是打乱比特位。

### sub_10EA4 比特位复原

在函数最后进行的是比特位复原，四轮循环，每轮还原两个字节。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_TXMP3RZXRKDGHUK.png)

 

![](https://bbs.pediy.com/upload/attach/202103/802108_KRM66A9CEQM8UH8.png)

 

如有不清楚的地方，可以自行调试观察。

### 使用 unicorn 还原 sub_10EA4 算法

我们怎么还原 sub_10EA4 中的算法呢，过程不难就是量多，难道真的要一点点对比着 IDA 的 f5 反编译手动写出一样的逻辑吗？当然不是，这里有更简单的方法，我们可以借助 unicorn 来快速还原。

 

我们已经分析出来 sub_10EA4 算法做的不过是打乱传入八个字节的比特位，生成新的八个字节，也就说总的比特位只是顺便变了，并没有被改变值，那么我们可不可以找到每个比特位在打乱前和被打乱后的映射关系呢？当然可以。

 

我们只要借助 unicorn，控制 sub_10EA4 函数输入的八个字节的 64 个二进制比特位，如果 64 个比特中只有一个是 1，那么结果的比特位中会有几个 1 呢？也是只有一个，然后计算前和计算后比特位的 1 的索引位置就是要找的映射关系，我们进行 64 次计算，然后每次计算 1 的索引位置不同，最后就能得到全部比特位的计算前和计算后的映射关系了。

 

篇幅所限，unicorn 的使用就不多说了。

```
from unicorn import *
from unicorn.arm_const import *
 
table = []
 
def bytes2bin(bytes):
    arr = []
    for v in [m for m in bytes]:
        arr.append(
            [(v & 128) >> 7, (v & 64) >> 6, (v & 32) >> 5, (v & 16) >> 4, (v & 8) >> 3, (v & 4) >> 2, (v & 2) >> 1,
             v & 1])
    return [i for j in arr for i in j]
 
 
def bin2bytes(arr):
    length = len(arr) // 8
    arr1 = [0 for _ in range(length)]
    for j in range(length):
        arr1[j] = arr[j * 8] << 7 | arr[j * 8 + 1] << 6 | arr[j * 8 + 2] << 5 | arr[j * 8 + 3] << 4 | arr[
            j * 8 + 4] << 3 | arr[j * 8 + 5] << 2 | arr[j * 8 + 6] << 1 | arr[j * 8 + 7]
    return bytes(arr1)
 
 
def read(name):
    with open(name, 'rb') as f:
        return f.read()
 
 
def hook_code(mu, address, size, user_data):
    if address == BASE + 0x119cc:
        arr2 = []
        for byte in mu.mem_read(PLAINTEXT_ADDR, 8):
            arr2.append(byte)
        table.append([user_data.index(1), bytes2bin(arr2).index(1)])
 
 
if __name__ == "__main__":
    key0 = b'44e715a6e322ccb7d028f7a42fa55040'
    mu = Uc(UC_ARCH_ARM, UC_MODE_THUMB)
    BASE = 0x400000
    STACK_ADDR = 0x0
    STACK_SIZE = 1024
    PLAINTEXT_ADDR = 1024 * 2
    PLAINTEXT_SIZE = 1024
    KEY_ADDR = 1024 * 3
    KEY_SIZE = 1024
    mu.mem_map(BASE, 1024 * 1024)
    mu.mem_map(STACK_ADDR, STACK_SIZE)
    mu.mem_map(PLAINTEXT_ADDR, PLAINTEXT_SIZE)
    mu.mem_map(KEY_ADDR, KEY_SIZE)
    mu.reg_write(UC_ARM_REG_SP, STACK_ADDR + STACK_SIZE - 1)
    # mu.mem_write(BASE, read("F:\\Code\\Pycharm\\JDSign\\libjdbitmapkit.so"))
    mu.mem_write(BASE, read("./libjdbitmapkit.so"))
    mu.mem_write(KEY_ADDR, key0)
 
    for i in range(64):
        arr1 = [0 for j in range(64)]
        arr1[i] = 1
        h = mu.hook_add(UC_HOOK_CODE, hook_code, arr1)
        mu.mem_write(PLAINTEXT_ADDR, bin2bytes(arr1))
        mu.reg_write(UC_ARM_REG_R1, KEY_ADDR)
        mu.reg_write(UC_ARM_REG_R2, 32)
        mu.reg_write(UC_ARM_REG_R3, PLAINTEXT_ADDR)
        mu.emu_start(BASE + 0x00010EA4 + 1, BASE + 0x000119D0)
        mu.hook_del(h)
    print(table)

```

可以得到映射表，例如 [1,4] 表示 64 个比特位在打乱后第 1 个位置的比特位会到第 4 个位置。

```
[[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2], [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21], [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55], [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41], [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28], [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50], [63, 48]]

```

然后可轻松还原 sub_10EA4 算法。

```
def sub_10EA4(input):
    table = [[0, 0], [1, 4], [2, 61], [3, 15], [4, 56], [5, 40], [6, 6], [7, 59], [8, 62], [9, 58], [10, 17], [11, 2],
             [12, 12], [13, 8], [14, 32], [15, 60], [16, 13], [17, 45], [18, 34], [19, 14], [20, 36], [21, 21],
             [22, 22], [23, 39], [24, 23], [25, 25], [26, 26], [27, 20], [28, 1], [29, 33], [30, 46], [31, 55],
             [32, 35], [33, 24], [34, 57], [35, 19], [36, 53], [37, 37], [38, 38], [39, 5], [40, 30], [41, 41],
             [42, 42], [43, 18], [44, 47], [45, 27], [46, 9], [47, 44], [48, 51], [49, 7], [50, 49], [51, 63], [52, 28],
             [53, 43], [54, 54], [55, 52], [56, 31], [57, 10], [58, 29], [59, 11], [60, 3], [61, 16], [62, 50],
             [63, 48]]
    arr = bytes2bin(input)
    arr1 = [0 for i in range(len(arr))]
    for i in range(len(table)):
        arr1[table[i][1]] = arr[table[i][0]]
    return bin2bytes(arr1)

```

### sub_10D70 函数分析

sub_10D70 函数在 IDA 反编译也比较清晰，我们来看下。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_5QXABCYN6RJE5UP.png)

 

前面有说，待加密字符串会被每 8 个字节分为一组传进 sub_10EA4 进行加密，sub_10D70 会加密最后余出来的几个字节，这 6 个 case 就是加密余出来的 1 到 7 个字节。case 0 到 case 6 对应的六个函数都是一个模板，我们以 case 0 为例来分析，即 sub_4B7C。

### sub_4B7C 初始化

sub_4B87 只会加密处理一个字节，首先我们来看下 sub_4B7C 的流程图。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_6B3342JYE57VS4U.png)

 

可以看到密密麻麻，其他五个 case 也都长这样，看到后首先第一个反应就是，这是 ollvm 吗？在经过仔细的分析以及动态调试之后，我判断这个并不是 ollvm，没有看到控制流平坦化会带有的标志性大量的常量，也没有找到不可到达的分支，虚假控制流以及指令替换的痕迹，当然也可能是我水平太低了，没有认出来这种 ollvm，总之，我还是铁着头把整个流程从头到尾看了一遍。

 

来看下初始化，

 

![](https://bbs.pediy.com/upload/attach/202103/802108_M9RXY7VFPH6RYB4.png)

 

看样子有点像 sub_10EA4，但仔细一看又很多不一样，上半部分取了传入的一个字节的比特位，还做了些其他的计算操作，赋值给变量。

 

而下半部分呢，则是取一些变量的地址，然后放到另一些变量地址加偏移处，这些说起来可能很模糊，但是如果看一下 sub_4B7C 的栈空间就会清晰很多。

 

为了方便理解，我们把这样连着五个变量在一起的当作一个数组，这样的数组往下拉可以看到是有八个，每个数组的前四个位置都存放着其他数组首的地址，而第五个位置则存放着前面说的输入字节的比特位经过计算后的值，其实分析后面的 case 就会发现，有多少输入的比特位，就会有多少个这样的数组。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_AF7CZG9VBJPBU2C.png)

 

初始化就这么多，而后面就开始根据这些东西进行大量的循环计算。

### sub_4B7C 计算分析

![](https://bbs.pediy.com/upload/attach/202103/802108_7WDQ7DW75GME39J.png)

 

整个循环是根据 key0 的每个字节的每个比特位作为判断条件来选择分支，然后进行循环的，所有的计算类型只有两种，就是下面的两个大方框，是两个小循环。

 

它们的汇编如下。

```
.text:00005BE2 loc_5BE2                                ; CODE XREF: sub_4B7C+104↑j
.text:00005BE2                 CMP             R1, R2
.text:00005BE4                 MOV             R12, R1
.text:00005BE6                 BEQ             loc_5C00
.text:00005BE8                 MOV             R6, R1
.text:00005BEA                 MOV             R8, R2
.text:00005BEC                 B               loc_5BF0
.text:00005BEE ; ---------------------------------------------------------------------------
.text:00005BEE
.text:00005BEE loc_5BEE                                ; CODE XREF: sub_4B7C+1082↓j
.text:00005BEE                 MOV             R12, R6
.text:00005BF0
.text:00005BF0 loc_5BF0                                ; CODE XREF: sub_4B7C+1070↑j
.text:00005BF0                 LDRB            R6, [R6,#0x10]
.text:00005BF2                 STRB.W          R6, [R8,#0x10]
.text:00005BF6                 MOV             R8, R12
.text:00005BF8                 LDR.W           R6, [R12,#0xC]
.text:00005BFC                 CMP             R6, R2
.text:00005BFE                 BNE             loc_5BEE
.text:00005C00

```

```
text:00005C00                 CMP             R11, R2
.text:00005C02                 STRB.W          R4, [R12,#0x10]
.text:00005C06                 MOV             R4, R11
.text:00005C08                 LDRB.W          R8, [SP,#0xE0+var_98]
.text:00005C0C                 IT NE
.text:00005C0E                 MOVNE           R12, R2
.text:00005C10                 BNE             loc_5C16
.text:00005C12                 B               loc_5C2C
.text:00005C14 ; ---------------------------------------------------------------------------
.text:00005C14
.text:00005C14 loc_5C14                                ; CODE XREF: sub_4B7C+10AE↓j
.text:00005C14                 MOV             R4, R6
.text:00005C16
.text:00005C16 loc_5C16                                ; CODE XREF: sub_4B7C+1094↑j
.text:00005C16                 LDRB            R6, [R4,#0x10]
.text:00005C18                 RSBS.W          R6, R6, #1
.text:00005C1C                 IT CC
.text:00005C1E                 MOVCC           R6, #0
.text:00005C20                 STRB.W          R6, [R12,#0x10]
.text:00005C24                 LDR             R6, [R4,#4]
.text:00005C26                 MOV             R12, R4
.text:00005C28                 CMP             R6, R2
.text:00005C2A                 BNE             loc_5C14

```

这两个小循环做了什么呢？其实十分简单，就是不断地查刚才我们定义的数组的前四个位置存放的变量地址值，判断是否等于另一个变量的地址值，只不过判断的过程中会不断地移动这些数组的第五个位置的值。

### [](#使用unicorn还原case0——sub_4b7c算法)使用 unicorn 还原 case0——sub_4B7C 算法

这个算法似乎很难搞，确实很难搞，不同于 sub_10EA4 的对输入字节的比特位进行简单的交换，在获取了比特位后又进行了很多计算。

 

该怎么办呢，我们来看下 sub_4B7C 算法的最后部分。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_V7AWBUNK72Q5VYS.png)

 

可以看到和 sub_10EA4 函数最后的比特位还原部分几乎是一样的，这时候我们进行下大胆的猜测，sub_4B7C 函数其实也是打乱输入字节的比特位进行了还原，只不过比特位在打乱后还进行了计算，而且每个比特位进行计算的规则都是一样的。即输入的字节第 x 个比特位是 0 的话，打乱计算还原后，第 y 个比特位会是 m；如果输入的字节第 x 个比特位是 1 的话，第 y 个比特位则会是 n，（x, 0）——>（y,m）（x,1）——>（y,n）。后面经过实践，证明了这样的猜想是正确的。

 

然后我们还是可以借助 unicorn 来找到所有的映射关系，还原 sub_4B7C 算法。思路我们进行下改变，我们可以 unicorn 控制输入字节的比特位，用八个比特位只有一个 1 的计算结果和八个比特位全是 0 的计算结果进行对比从而得到所有的映射关系。

```
from unicorn import *
from unicorn.arm_const import *
 
table = []
table1 = []
 
def bytes2bin(bytes):
    arr = []
    for v in [m for m in bytes]:
        arr.append(
            [(v & 128) >> 7, (v & 64) >> 6, (v & 32) >> 5, (v & 16) >> 4, (v & 8) >> 3, (v & 4) >> 2, (v & 2) >> 1,
             v & 1])
    return [i for j in arr for i in j]
 
 
def bin2bytes(arr):
    length = len(arr) // 8
    arr1 = [0 for _ in range(length)]
    for j in range(length):
        arr1[j] = arr[j * 8] << 7 | arr[j * 8 + 1] << 6 | arr[j * 8 + 2] << 5 | arr[j * 8 + 3] << 4 | arr[
            j * 8 + 4] << 3 | arr[j * 8 + 5] << 2 | arr[j * 8 + 6] << 1 | arr[j * 8 + 7]
    return bytes(arr1)
 
 
def read(name):
    with open(name, 'rb') as f:
        return f.read()
 
 
def hook_code(mu, address, size, user_data):
    if address == BASE + 0x5284:
        table.append(bytes2bin(mu.mem_read(PLAINTEXT_ADDR, 1)))
 
 
if __name__ == "__main__":
    key0 = b'44e715a6e322ccb7d028f7a42fa55040'
    mu = Uc(UC_ARCH_ARM, UC_MODE_THUMB)
    BASE = 0x400000
    STACK_ADDR = 0x0
    STACK_SIZE = 1024
    PLAINTEXT_ADDR = 1024 * 2
    PLAINTEXT_SIZE = 1024
    KEY_ADDR = 1024 * 3
    KEY_SIZE = 1024
    mu.mem_map(BASE, 1024 * 1024)
    mu.mem_map(STACK_ADDR, STACK_SIZE)
    mu.mem_map(PLAINTEXT_ADDR, PLAINTEXT_SIZE)
    mu.mem_map(KEY_ADDR, KEY_SIZE)
    mu.reg_write(UC_ARM_REG_SP, STACK_ADDR + STACK_SIZE - 1)
    # mu.mem_write(BASE, read("F:\\Code\\Pycharm\\JDSign\\libjdbitmapkit.so"))
    mu.mem_write(BASE, read("./libjdbitmapkit.so"))
    mu.mem_write(KEY_ADDR, key0)
 
    for i in range(9):
        arr1 = [0 for j in range(8)]
        if i != 0:
            arr1[i-1] = 1
        h = mu.hook_add(UC_HOOK_CODE, hook_code, arr1)
        mu.mem_write(PLAINTEXT_ADDR, bin2bytes(arr1))
        mu.reg_write(UC_ARM_REG_R0, KEY_ADDR)
        mu.reg_write(UC_ARM_REG_R1, 32)
        mu.reg_write(UC_ARM_REG_R2, 1)
        mu.reg_write(UC_ARM_REG_R3, PLAINTEXT_ADDR)
        mu.emu_start(BASE + 0x0004B7C + 1, BASE + 0x0005288)
        mu.hook_del(h)
 
    for i in range(8):
        for j in range(8):
            arr3 = []
            if table[0][j] != table[i+1][j]:
                table1.append([i, j, table[0][j], table[i+1][j]])
    print(table1)

```

就结果为

```
[[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0], [7, 7, 0, 1]]

```

我们可以得到为 [[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0], [7, 7, 0, 1]]，里面的列表每个即为[x,y,m,n]，比如[6, 1, 0, 1] 意味，第 6 个比特位在打乱计算后会放到第 1 个比特位上，如果第 6 个比特位是 0，则在打乱计算后会放到第 6 个比特位上是 0，反之是 1。

 

然后可轻松还原 sub_4B7C 算法。

```
def sub_4B7C(input):
    table = [[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0],
             [7, 7, 0, 1]]
    arr = bytes2bin(input)
    arr1 = [0 for i in range(8)]
    for i in range(8):
        if arr[i] == 0:
            arr1[table[i][1]] = table[i][2]
        else:
            arr1[table[i][1]] = table[i][3]
    return bin2bytes(arr1)

```

### 使用 unicorn 一步求出所有 case 的映射关系

好了，现在我们已经还原出第一个 case，我们再会过头来看下 sub_10D70，可以看到一共有七个 case，分别对应的七个函数不同的只有函数起始结束地址，以及要处理的字节数，开拓下思维，这时候我们完全可以 all in one，一次得到所有 case 的映射关系。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_5QXABCYN6RJE5UP.png)

 

代码如下：

```
from unicorn import *
from unicorn.arm_const import *
 
table = []
table1 = []
 
 
def bytes2bin(bytes):
    arr = []
    for v in [m for m in bytes]:
        arr.append(
            [(v & 128) >> 7, (v & 64) >> 6, (v & 32) >> 5, (v & 16) >> 4, (v & 8) >> 3, (v & 4) >> 2, (v & 2) >> 1,
             v & 1])
    return [i for j in arr for i in j]
 
 
def bin2bytes(arr):
    length = len(arr) // 8
    arr1 = [0 for _ in range(length)]
    for j in range(length):
        arr1[j] = arr[j * 8] << 7 | arr[j * 8 + 1] << 6 | arr[j * 8 + 2] << 5 | arr[j * 8 + 3] << 4 | arr[
            j * 8 + 4] << 3 | arr[j * 8 + 5] << 2 | arr[j * 8 + 6] << 1 | arr[j * 8 + 7]
    return bytes(arr1)
 
 
def read(name):
    with open(name, 'rb') as f:
        return f.read()
 
 
def hook_code(mu, address, size, user_data):
    if address == BASE + user_data[2] -4:
        table.append(bytes2bin(mu.mem_read(PLAINTEXT_ADDR, m[0])))
 
 
if __name__ == "__main__":
    key0 = b'44e715a6e322ccb7d028f7a42fa55040'
    mu = Uc(UC_ARCH_ARM, UC_MODE_THUMB)
    BASE = 0x400000
    STACK_ADDR = 0x0
    STACK_SIZE = 1024 * 10
    PLAINTEXT_ADDR = 1024 * 10
    PLAINTEXT_SIZE = 1024
    KEY_ADDR = 1024 * 11
    KEY_SIZE = 1024
    mu.mem_map(BASE, 1024 * 1024)
    mu.mem_map(STACK_ADDR, STACK_SIZE)
    mu.mem_map(PLAINTEXT_ADDR, PLAINTEXT_SIZE)
    mu.mem_map(KEY_ADDR, KEY_SIZE)
    mu.reg_write(UC_ARM_REG_SP, STACK_ADDR + STACK_SIZE - 1)
    # mu.mem_write(BASE, read("F:\\Code\\Pycharm\\JDSign\\libjdbitmapkit.so"))
    mu.mem_write(BASE, read("./libjdbitmapkit.so"))
    mu.mem_write(KEY_ADDR, key0)
 
    arr = [[1, 0x0004B7C, 0x0005288], [2, 0x00061A0, 0x0006AE8], [3, 0x0007994, 0x000084A6], [4, 0x000091AC, 0x00009DF4],
           [5, 0x0000ABF8, 0x0000BA8C], [6, 0x0000C8C0, 0x0000D9A0], [7, 0x0000E7FC, 0x0000FC1C]]
    for m in arr:
        for i in range(m[0]*8+1):
            arr1 = [0 for j in range(m[0]*8)]
            if i != 0:
                arr1[i - 1] = 1
            h = mu.hook_add(UC_HOOK_CODE, hook_code, m)
            mu.mem_write(PLAINTEXT_ADDR, bin2bytes(arr1))
            mu.reg_write(UC_ARM_REG_R0, KEY_ADDR)
            mu.reg_write(UC_ARM_REG_R1, 32)
            mu.reg_write(UC_ARM_REG_R2, 1)
            mu.reg_write(UC_ARM_REG_R3, PLAINTEXT_ADDR)
            mu.emu_start(BASE + m[1] + 1, BASE + m[2])
            mu.hook_del(h)
 
        for i in range(m[0]*8):
            for j in range(m[0]*8):
                arr3 = []
                if table[0][j] != table[i + 1][j]:
                    table1.append([i, j, table[0][j], table[i + 1][j]])
        print("case %s 映射关系:"%(m[0]-1))
        print(table1)
        table.clear()
        table1.clear()

```

运行结果：

```
case 0 映射关系:
[[0, 6, 0, 1], [1, 4, 1, 0], [2, 5, 0, 1], [3, 0, 0, 1], [4, 2, 0, 1], [5, 3, 0, 1], [6, 1, 1, 0], [7, 7, 0, 1]]
case 1 映射关系:
[[0, 5, 0, 1], [1, 9, 0, 1], [2, 0, 1, 0], [3, 7, 1, 0], [4, 10, 0, 1], [5, 6, 0, 1], [6, 13, 1, 0], [7, 1, 0, 1], [8, 4, 0, 1], [9, 11, 0, 1], [10, 14, 1, 0], [11, 3, 1, 0], [12, 12, 0, 1], [13, 15, 1, 0], [14, 8, 0, 1], [15, 2, 0, 1]]
case 2 映射关系:
[[0, 17, 0, 1], [1, 7, 0, 1], [2, 5, 0, 1], [3, 19, 1, 0], [4, 18, 0, 1], [5, 15, 1, 0], [6, 22, 0, 1], [7, 21, 0, 1], [8, 16, 0, 1], [9, 4, 0, 1], [10, 12, 0, 1], [11, 2, 1, 0], [12, 10, 1, 0], [13, 13, 1, 0], [14, 20, 1, 0], [15, 8, 1, 0], [16, 9, 0, 1], [17, 23, 0, 1], [18, 11, 1, 0], [19, 6, 0, 1], [20, 1, 0, 1], [21, 3, 1, 0], [22, 0, 1, 0], [23, 14, 0, 1]]
case 3 映射关系:
[[0, 25, 1, 0], [1, 4, 0, 1], [2, 29, 0, 1], [3, 1, 0, 1], [4, 27, 1, 0], [5, 18, 1, 0], [6, 23, 1, 0], [7, 14, 1, 0], [8, 28, 1, 0], [9, 11, 0, 1], [10, 9, 1, 0], [11, 13, 0, 1], [12, 24, 1, 0], [13, 0, 1, 0], [14, 5, 0, 1], [15, 2, 1, 0], [16, 26, 0, 1], [17, 12, 0, 1], [18, 31, 1, 0], [19, 16, 1, 0], [20, 30, 0, 1], [21, 15, 0, 1], [22, 10, 0, 1], [23, 22, 1, 0], [24, 7, 1, 0], [25, 21, 0, 1], [26, 6, 1, 0], [27, 3, 1, 0], [28, 8, 1, 0], [29, 20, 0, 1], [30, 19, 1, 0], [31, 17, 0, 1]]
case 4 映射关系:
[[0, 11, 0, 1], [1, 12, 0, 1], [2, 28, 1, 0], [3, 30, 0, 1], [4, 13, 1, 0], [5, 24, 0, 1], [6, 22, 1, 0], [7, 25, 1, 0], [8, 23, 1, 0], [9, 3, 0, 1], [10, 16, 0, 1], [11, 8, 1, 0], [12, 34, 0, 1], [13, 2, 0, 1], [14, 5, 0, 1], [15, 7, 1, 0], [16, 4, 0, 1], [17, 14, 0, 1], [18, 39, 1, 0], [19, 33, 0, 1], [20, 15, 0, 1], [21, 0, 0, 1], [22, 31, 0, 1], [23, 9, 1, 0], [24, 29, 0, 1], [25, 26, 1, 0], [26, 19, 0, 1], [27, 6, 1, 0], [28, 27, 1, 0], [29, 10, 1, 0], [30, 37, 0, 1], [31, 38, 1, 0], [32, 20, 0, 1], [33, 21, 1, 0], [34, 1, 0, 1], [35, 36, 0, 1], [36, 32, 0, 1], [37, 17, 0, 1], [38, 18, 0, 1], [39, 35, 1, 0]]
case 5 映射关系:
[[0, 11, 0, 1], [1, 45, 0, 1], [2, 15, 1, 0], [3, 22, 0, 1], [4, 10, 0, 1], [5, 7, 0, 1], [6, 3, 0, 1], [7, 42, 0, 1], [8, 17, 1, 0], [9, 21, 0, 1], [10, 4, 0, 1], [11, 8, 1, 0], [12, 19, 1, 0], [13, 32, 0, 1], [14, 28, 1, 0], [15, 31, 1, 0], [16, 29, 0, 1], [17, 14, 1, 0], [18, 39, 1, 0], [19, 27, 1, 0], [20, 2, 1, 0], [21, 24, 0, 1], [22, 26, 1, 0], [23, 9, 1, 0], [24, 41, 0, 1], [25, 1, 1, 0], [26, 47, 0, 1], [27, 44, 0, 1], [28, 23, 1, 0], [29, 0, 1, 0], [30, 12, 1, 0], [31, 18, 0, 1], [32, 33, 0, 1], [33, 36, 0, 1], [34, 40, 1, 0], [35, 34, 0, 1], [36, 25, 0, 1], [37, 16, 1, 0], [38, 5, 1, 0], [39, 35, 0, 1], [40, 38, 0, 1], [41, 37, 1, 0], [42, 13, 0, 1], [43, 20, 1, 0], [44, 6, 0, 1], [45, 43, 0, 1], [46, 30, 0, 1], [47, 46, 1, 0]]
case 6 映射关系:
[[0, 7, 1, 0], [1, 9, 0, 1], [2, 53, 1, 0], [3, 19, 1, 0], [4, 15, 1, 0], [5, 8, 0, 1], [6, 3, 0, 1], [7, 24, 1, 0], [8, 18, 0, 1], [9, 51, 0, 1], [10, 42, 1, 0], [11, 39, 0, 1], [12, 20, 0, 1], [13, 12, 0, 1], [14, 28, 1, 0], [15, 27, 1, 0], [16, 23, 0, 1], [17, 49, 0, 1], [18, 10, 1, 0], [19, 55, 1, 0], [20, 52, 1, 0], [21, 17, 0, 1], [22, 48, 0, 1], [23, 14, 1, 0], [24, 33, 0, 1], [25, 25, 1, 0], [26, 4, 1, 0], [27, 11, 0, 1], [28, 47, 1, 0], [29, 0, 0, 1], [30, 21, 1, 0], [31, 44, 0, 1], [32, 16, 0, 1], [33, 41, 0, 1], [34, 29, 0, 1], [35, 1, 0, 1], [36, 46, 0, 1], [37, 5, 0, 1], [38, 30, 0, 1], [39, 45, 0, 1], [40, 31, 1, 0], [41, 43, 1, 0], [42, 36, 1, 0], [43, 26, 0, 1], [44, 34, 0, 1], [45, 2, 0, 1], [46, 6, 0, 1], [47, 50, 1, 0], [48, 13, 1, 0], [49, 37, 1, 0], [50, 32, 0, 1], [51, 40, 0, 1], [52, 35, 0, 1], [53, 38, 0, 1], [54, 54, 0, 1], [55, 22, 0, 1]]

```

Version2 加密
-----------

### 执行流程

加密执行函数流程：sub_10DE4——>sub_12FF0——>sub_12ECC——>sub_130D0，其中 sub_12ECC 是重点，我们直接看 sub_12ECC。

### sub_12ECC 初始化

进入函数`sub_12ECC`，先看下开始处的汇编片段。`r1`寄存器存放着`key2`字符串地址，`r2`寄存器存放着常量 1，`r3`寄存器存放着待加密字符串地址，在开辟栈空间后`SP,#0x48+arg_0`地址存放着是待加密字节的长度。

```
.text:00012ECC                 LDR.W           R12, =(__stack_chk_guard_ptr - 0x12ED8)
.text:00012ED0                 PUSH.W          {R4-R11,LR}
.text:00012ED4                 ADD             R12, PC ; __stack_chk_guard_ptr
.text:00012ED6                 LDR.W           R12, [R12] ; __stack_chk_guard
.text:00012EDA                 SUB             SP, SP, #0x24
.text:00012EDC                 MOV             R10, R3
.text:00012EDE                 LDR.W           R3, [R12]
.text:00012EE2                 MOV             R7, R1
.text:00012EE4                 LDR.W           R8, [SP,#0x48+arg_0]
.text:00012EE8                 MOV             R9, R2
.text:00012EEA                 STR             R3, [SP,#0x48+var_2C]
.text:00012EEC                 CMP.W           R8, #0
.text:00012EF0                 BNE             loc_12F02

```

我们注意到下面这四句汇编指令，是把待加密字符串存放的地址放到`r10`寄存器中，`key2`字符串地址放到`r7`寄存器中，常量 1 放到`r9`寄存器，从`SP,#0x48+arg_0`地址取出待加密字符串长度后放到 r8 寄存器中。

```
.text:00012EDC                 MOV             R10, R3
 
.text:00012EE2                 MOV             R7, R1
 
.text:00012EE8                 MOV             R9, R2
 
.text:00012EE4                 LDR.W           R8, [SP,#0x48+arg_0]

```

### sub_12ECC 加密流程

然后我们在函数`sub_12ECC`中往下走，找到`0x12F68`地址处开始的关键汇编片段，即 sign 算法加密部分，这里是一个循环。

```
.text:00012F66                 MOVS            R3, #0
.text:00012F68
.text:00012F68 loc_12F68                              
.text:00012F68                 AND.W           R2, R3, #0xF
.text:00012F6C                 ADD             R0, SP, #0x48+var_28
.text:00012F6E                 ADD             R2, R0
.text:00012F70                 AND.W           R1, R3, #7
.text:00012F74                 LDRB.W          R0, [R10]
.text:00012F78                 ADDS            R3, #1
.text:00012F7A                 LDRB.W          R2, [R2,#-0x14]
.text:00012F7E                 CMP             R3, R8
.text:00012F80                 LDRB            R4, [R7,R1]
.text:00012F82                 EOR.W           R0, R2, R0
.text:00012F86                 EOR.W           R0, R0, R4
.text:00012F8A                 ADD             R0, R2
.text:00012F8C                 EOR.W           R2, R2, R0
.text:00012F90                 UXTB            R2, R2
.text:00012F92                 STRB.W          R2, [R10],#1
.text:00012F96                 LDRB            R1, [R7,R1]
.text:00012F98                 EOR.W           R2, R2, R1
.text:00012F9C                 STRB.W          R2, [R10,#-1]
.text:00012FA0                 BNE             loc_12F68

```

**r10 寄存器**

 

我们抓重点，看待加密字符串是怎么被加密的，待加密字符串地址放在了`r10`寄存器中，涉及`r10`寄存器的汇编指令有三条。

 

`00012F74`处的`LDRB.W R0, [R10]`是取出待加密字符串的一个字节放到`r0`寄存器中随后会进行计算操作；

 

`00012F92`处的`STRB.W R2, [R10],#1`，这条指令首先会把放在`r2`寄存器中的计算结果值放到`r10`寄存器当前存储的地址上，这个并不重要，因为还没有计算完成，后面的指令才是把最后的计算值存放值到这个地址上，重要的是这条指令随后`r10`寄存器中的地址值会加一，因为这是一个基于索引后置修改取址模式；

 

在`00012F92`和`00012F9C`之间，我们可以看到取了`r7`寄存器存放的地址值的一个偏移地址的值（偏移值为`r1`寄存器中的值）放到了`r1`寄存器中，随后把`r1`和`r2`寄存器中的值进行亦或，结果存放在`r2`寄存器中。

 

最后是涉及到`r10`寄存器的第三条指令`00012F9C`处，`STRB.W R2, [R10,#-1]`，把计算结果值放到`R10,#-1`地址处，注意到`r10`寄存器中的地址值刚才已经加一，所以现在减一后还是刚才存储的地址，所以这里才是计算结果最后存放的指令。

 

**r3 寄存器**

 

好了，我们已经知道计算结果是放到了 r2 寄存器中，我们从`00012F66`开始看，一步步看是怎么计算出来的。

 

可以看到，`r3`作为计数器，每轮循环会在`.text:00012F78 ADDS R3, #1`处加一，并在`.text:00012F7E CMP R3, R8`处和`r8`寄存器值即加密字符串长度进行比较。

 

**r2 寄存器**

 

在开始处，r3 首先和 0xf 进行与操作，结果放在`r2`寄存器中，随后将与操作的结果值和`SP, #0x48+var_28`地址值进行相加，相加结果仍放在 r2 寄存器中，随后会进行`.text:00012F7A LDRB.W R2, [R2,#-0x14]`，也就是说现在`r2`寄存器中存放的是`SP, #0x48+var_28`地址加上一个偏移值（即（i &0xf） - 0x14）这个地址上存放的值。

 

**r4 寄存器**

 

然后在`.text:00012F70 AND.W R1, R3, #7`处把`r3`寄存器中的值和 7 进行与操作，结果放在`r1`寄存器中，然后在`.text:00012F80 LDRB R4, [R7,R1]`处把`r1`寄存器中的地址值作为偏移加上`r7`寄存器中的值，取出这个地址中的值放在 r4 寄存器中，`r7`寄存器前面我们说了放的是`key2`, 所以放在`r4`寄存器中这个的值即是`key2[i&7]`。

 

**计算部分**

 

随后便是计算部分：

```
.text:00012F82                 EOR.W           R0, R2, R0
.text:00012F86                 EOR.W           R0, R0, R4
.text:00012F8A                 ADD             R0, R2
.text:00012F8C                 EOR.W           R2, R2, R0
.text:00012F90                 UXTB            R2, R2
 
.text:00012F96                 LDRB            R1, [R7,R1]
.text:00012F98                 EOR.W           R2, R2, R1

```

此时，涉及计算的`r0 r2 r4`我们都已经知道是什么了，`r0`寄存器中的值我们在提`r10`寄存器时候有知道它是待加密字符串取出的一个字节。

 

**IDA F5**

 

这部分使用 IDA 进行 f5 的效果是这样的。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_79PXHH9H634B478.png)

### sub_12ECC 算法还原

现在我们想还原这个加密过程该怎么办，要看我们缺什么。待加密字符串，key2 和加密计算过程我们都有了，只有一个`SP, #0x48+var_28`地址加上一个偏移 `-0x14`，这个地址存放有一个数组，每轮加密循环会取出一个值（数组偏移（i & 0xf）处）放在`r2`寄存器中然后进行计算，我们通过静态分析无法知道这个数组值是什么，所以进行 ida 动态调试，可以得到这个地方存放的数组值为 [0x37, 0x92, 0x44, 0x68, 0xA5, 0x3D, 0xCC, 0x7F, 0xBB, 0xF, 0xD9, 0x88, 0xEE, 0x9A, 0xE9, 0x5A]。

 

![](https://bbs.pediy.com/upload/attach/202103/802108_MF75YCHM4MJPKC8.png)

 

随后可 python 写出 sub_12ECC 函数中的加密计算过程。

```
def sub_12ECC(input):
    arr = [0x37, 0x92, 0x44, 0x68, 0xA5, 0x3D, 0xCC, 0x7F, 0xBB, 0xF, 0xD9, 0x88, 0xEE, 0x9A, 0xE9, 0x5A]
    key2 = b"80306f4370b39fd5630ad0529f77adb6"
    arr1 = [0 for _ in range(len(input))]
    for i in range(len(input)):
        r0 = int(input[i])
        r2 = arr[i & 0xf]
        r4 = int(key2[i & 7])
        r0 = r2 ^ r0
        r0 = r0 ^ r4
        r0 = r0 + r2
        r2 = r2 ^ r0
        r1 = int(key2[i & 7])
        r2 = r2 ^ r1
        arr1[i] = r2 & 0xff
    return bytes(arr1)

```

sign 算法全流程还原
------------

好了，到现在所有的关键函数已经被我们分析出来了，想还原出来所有的 sign 加密流程已经成了一个时间问题的体力劳动了，本来想放个半成品给大家参考下，经评论区老哥友情提醒，分析流程已经足够了，这部分就先和谐掉吧...

 

以及文章目的是学习交流的，请勿不正确使用，用于违法行为。

后记
--

最近经常会在想自己大部分时候只会用 frida 去 hook 十分像一个脚本小子（逃，迫切需要提升汇编分析水平，分析到这里对我自己来说进步不小，但是要学习的东西还很多，如果有什么希望的话，就是希望能早一点摆脱对 IDA f5 的依赖，可以手撕汇编：）

 

最后，如果你有学到有用的东西，请不要忘记点赞，不枉我写了这么多。

[【公告】欢迎大家踊跃尝试高研班 11 月试题，挑战自己的极限！](https://bbs.pediy.com/thread-270220.htm)

最后于 2021-3-14 11:16 被 0x 指纹编辑 ，原因：

[#逆向分析](forum-161-1-118.htm)

上传的附件：

*   [app-debug.apk](javascript:void(0)) （1.75MB，200 次下载）