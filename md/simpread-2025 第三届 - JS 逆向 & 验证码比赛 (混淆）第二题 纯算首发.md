> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pd08MpIqw42yv3HMfFUVZw)

```
(())([.....])(248, '233', '1749317316')

```

首先看一下入参 ，参数分别是三个 

248, '233', '1749317316'

248 为滑动距离   '233', '1749317316' 这两就是 通过 "洋芋粑" 解出来的 自己看一眼前面的算法 mcc 比较简单就不说了

```
yrx_ss = {    "data1": "...",    "data2": "...",    "洋芋粑": "7e02ce990e7977bc2802f00f96"}returenRusult = draw_image2(yrx_ss)console.log("参数1:", returenRusult.arg1);console.log("参数2:", returenRusult.arg2);

```

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRcZDIibtF7Rb3Rhk3aLIETZGWibhQlFsFqCecIL2QicKgl0RYicRicCDIf0g/640?wx_fmt=png&from=appmsg)

运行这里可以看到 最终的结果生成

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRwF0RUCE8zQMia8ibgnhq5zsu9aszkibV7ORPqIV7Dncq7Gn6EiaoIAbWWA/640?wx_fmt=png&from=appmsg&watermark=1)

分析代码
----

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRnoaiblehnxen9GHajht855WJjibmcq8C9MIxY3woVrV71Uv3PovmYqPg/640?wx_fmt=png&from=appmsg&watermark=1)

他为什么要这么写呢 这是在干啥

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR1SN1ljco0DRhWK3ia59yvug5chTAAdF53mG2eriaQHuxLiaSn0RQNhzqQ/640?wx_fmt=png&from=appmsg&watermark=1)

debug 发现 每次走都会进到这里，

并且每个都在走这个逻辑

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRZrRJhYM7fCNlKFJxKeX3FmQUQFZ33VJ9P4JyejeJ9iaom6C0p05EaMA/640?wx_fmt=png&from=appmsg&watermark=1)

看看这个是啥

可以发现 这里就是对应的栈的操作

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR3phd9iau11aQbDP8mkB3dVv36oicV6p58cr4D6RUOZqQBgl8cFnJnxYA/640?wx_fmt=png&from=appmsg&watermark=1)

而第 2，第三个参数

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRQ8UKPeOs5EGr7pOBc2C4IdWVyzTzR46sPUNvC3GQYwDSCNu2FGKyrA/640?wx_fmt=png&from=appmsg&watermark=1)

一个是栈，一个是 结果

比如如果是 push 那可以在这里直接看到结果

那就直接插装

'方法 ->',yrx_ss"$","栈 ->",yrx_ssss,"结果 ->",yrx_sssss

然后保存日志来分析

分析日志
----

日志大概 3.3w 行

先倒着看  

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR3qke18HvPL0JAdrVTJdhTkicXnH19ibxibmiaLYlNNrYY2fJgzogC4AUOQ/640?wx_fmt=png&from=appmsg&watermark=1)

可以注意到 他是从 ['2a', '85', '78', '73', '2f', 'b7', '20', 'bd', '76', '80', 'ce', '2d', '37', '5b', '6e', '57'] 在拼接得来的

接着搜

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRICWwxPYAMBoSl8anL9lSSicc9q0iaqAABe8OMloicX5d2ttCJhCYuqmSg/640?wx_fmt=png&from=appmsg&watermark=1)

可以看到他是用了 toString 的方法转成的也就是转成了 16 进制

num = 42 

42

num.toString(16,1)  

'2a'

继续搜可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRUByNcnv67exxwNJicrgVzAaBvHgtFjxpR5mqqRJEuKMib1CZicHkDicLkQ/640?wx_fmt=png&from=appmsg&watermark=1)

现在就是要看这个数组是怎么生成的

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRY7upNMDr6BsD3kWVb8hj2B2GuMHook6ThlfmST4hzJPHpqNYDr2gbQ/640?wx_fmt=png&from=appmsg&watermark=1)

这么插发现日志不是很清晰但是 928738903 255 有关 可以猜到  87 -> 928738903&255

那说明 还需要在指令集插一下，可以让日志更容易看清， 怕丢的全插上就好了

现在懂了 

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRbSSVt3YX5euUJYibJxRZW8Yn8oArNnlNzxstJAuwclV05oqPJ4CmISA/640?wx_fmt=png&from=appmsg&watermark=1)

根据这个规律往上看几个

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR2JaIl2CHzQqSsrCdng3R66rfSfx1FbdqJqf9lZRDhRUuNUia64y2ojA/640?wx_fmt=png&from=appmsg&watermark=1)

你会发现规律 其实就是 第一个红标是 index 第二个红边 有一个四位数组 第三个就是结果

### 4 位数组生成

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR6RkgPwLNiaB8d2RsFVARRNeicSkicuXzTLUkQ26xqw7t96Y2k6vfibO7wA/640?wx_fmt=png&from=appmsg&watermark=1)

可以看到 这里有个 uint32array 数组 其中包含了

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRu9Qobicf0AGPfVvG1ibWznkiaBFAKCOaMASLuibQhJZrNQFm0MT5pbibEBg/640?wx_fmt=png&from=appmsg&watermark=1)

这么一对比可以发现 他就是 36 位数组 最后四位做了一个反转 (猜测) 可以多打点日志 观察一下 发现是这个逻辑

### 36 位数组生成

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRmZNNzf8VRwkIuA6ibXXULy4EVIfbTZQLjUnv0YaiaDyn86XOeZqHicy8A/640?wx_fmt=png&from=appmsg&watermark=1)

顺着可以找出 他跟另一个 16 位数组有关系 并且 取了索引 然后通过 ^ 或者 + 得到了最后的结果

如果倒着往上找你会发现越来越乱 那现在就从前往后看

先把已知的变量拿出来

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRsMM4KkqshvhNpTX0pz500wbV1j4tkTiafUeoPiamErEEET7lCsR53rIg/640?wx_fmt=png&from=appmsg&watermark=1)

这几个变量先记号 后续会有用

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRMJDVOCe1lIZcBdbgEtFhI6qTaricRvGPoCgDzwM0LubgagXgawAMhibA/640?wx_fmt=png&from=appmsg&watermark=1)

字符串拼接 '248|233174931732'  - > [50, 52, 56, 124, 50, 51, 51, 49, 55, 52, 57, 51, 49, 55, 51, 50

初见 36 位数组

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeReE29CAgbL3vTibMgq1odibFWkcicnIqJQ6m4Fc9vSwNXABnVibnNOrpqoQ/640?wx_fmt=png&from=appmsg&watermark=1)

第一位怎么来的搜索往上看

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRVEIA2ZY4Gx6jhEsBqx3DribGOwsUiatB1zZBiakHpTfhtia8h25RUduMEA/640?wx_fmt=png&from=appmsg&watermark=1)

然后就直接拿到了 2202575649

猜测 - 2092391647 >>> 0 -> 2202575649

然后发现 [880441166, 810043492, 1483438191, 1868057465] 不知道

#### [880441166, 810043492, 1483438191, 1868057465] 生成

继续往上看

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRMYwrSYnfQ4lia5AsaSpXtibnU5L6wfpgFp3bsxDjMYpDJg3gxZIaNOpg/640?wx_fmt=png&from=appmsg&watermark=1)

可以看到 是一个数组 然后异或得来的

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRg7AmOeojCpBpico8GVMuCK0Icx3YXTgx9giadJ1pbdJROGf73XNDhqOg/640?wx_fmt=png&from=appmsg&watermark=1)

再去找最开始的四位数组

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRy54oHBdJReTIOicibFGNGsrZrlOIWgNT41o6X6NEgia0SC442Syno3cEA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRu6NxbqHUfs64oH7QXEWXMFy1VuILibeBkJMjUx1pch1fED3Rf4vzOng/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRDCP88IPVUExOwqT9CzsoeW0WTlem9uIKs31t2pLpTqKkAE0miaaMAQA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRVjTTEQczMAoNQE8icy20oKLxxShzRzXicGEDjZ4HhibaxPgsXpHQ0ppng/640?wx_fmt=png&from=appmsg&watermark=1)

[52, 122, 119, 78, 48, 72, 72, 100, 88, 107, 120, 111, 111, 88, 75, 121]

可以发现他是每四位进行一轮运算 

```
for (leti=0; i<byteArray.length; i+=4) {        // 每 4 个字节作为一组进行处理        constvalue= ((byteArray[i] ||0) <<24|        // 第一个字节移到最高位                       (byteArray[i+1] ||0) <<16|    // 第二个字节移到次高位                       (byteArray[i+2] ||0) <<8|     // 第三个字节移到次低位                       (byteArray[i+3] ||0) <<0) >>>0; // 第四个字节保持在最低位        result.push(value); // 将结果存入数组   }

```

可得到 [880441166, 810043492, 1483438191, 1868057465]

现在就可以继续看刚刚的 36 位数组了

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRdkibQAaTLmcPNxsbuZImMT4WUL4DxAPv7P6RUVVu4zsZ9UJToRuqZxA/640?wx_fmt=png&from=appmsg&watermark=1)

是由异或得到

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR3vGGQmCl74l4z0u6M1Eibx1xH9RbSXFKJ5dW2Ccx3NfVtiaVWC7ibz2pQ/640?wx_fmt=png&from=appmsg&watermark=1)

同理 这样就拿到了 36 位数组的前四位

```
constxorResult=newArray.map((value, index) => {        return (value^referenceArray[index]) >>>0; // 异或并确保是无符号整数 });

```

看剩下的 32 位怎么来的

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRpNQyPJSg35FYbxvjvLP64QNMMTDeX8YUic21mpfalkVcrxIvKFCWBew/640?wx_fmt=png&from=appmsg&watermark=1)

分析单独一块的日志

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRmDNyFBJU1nhPXOT7HYCnUDGOAyRC2WLeViaFBsJvh77VBMI7XGopVkg/640?wx_fmt=png&from=appmsg&watermark=1)

首先通过刚刚的四位从第一位开始异或

 xorResult = arr[j] ^ arr[j + 1] ^ arr[j + 2];

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeROKNLjqsEAib6alJ70ic299LDIJhQh90Pa5iaWtHqA8OhotPrwrtVe3YZg/640?wx_fmt=png&from=appmsg&watermark=1)

然后与 32 位数组的第一位异或 得到结果 - 20298919

这么一想 是不是就会想到 如果这样每个操作 都会与 32 位 有关那刚刚好生成 就是 36 位

第二阶段

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRP6Jgwa8QLvWQvr5vdFyX3licuMx3LH9BLk2lj2f39HvPuMAQgibJTjeQ/640?wx_fmt=png&from=appmsg&watermark=1)

```
>>> 24位然后 &25 5然后得到的结果从256位数组映射出来再<<这看着真的很像aes的沙盒letorigin= (xorResult^array32[i]);saveresult1= (origin>>>24) &255save1=array256[saveresult1] <<24

```

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR9BOlWnuvmvLMRzQvj6icQqQW9X4pPENd6KzW9iavF23WhOj1ZV2jJK4Q/640?wx_fmt=png&from=appmsg&watermark=1)

```
save2= (origin>>>16) &255save3=array256[save2] <<16save4=save3|save1

```

可以看到是这样的过程

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRQYAu7QWyOQbegoWSF0wK9icCHQGQ04cNkLph1KjX6pIkUNeia31vpKNg/640?wx_fmt=png&from=appmsg&watermark=1)

继续看日志 

```
save5=origin>>>8&255// 16697923  -> 67


```

```
save6=array256[save5] <<8// 17152


```

```
save7=save6|save4//-1031453952


```

```
save8= (origin>>>0) &255// 


```

```
save9=array256[save8]save10= (save9|save7) >>>0

```

就是重复这个过程 24,16,8,0 阶段结果 3263546356

接下来分析日志就是循环左移的过程

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR7y9YSrzf4BLV2L5D0JY1obPwzfYoztCauqTJnibM44ah9vliaDRakKaQ/640?wx_fmt=png&from=appmsg&watermark=1)

红色标记位刚 36 位数组的第一个 与结果异或

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRW6iapvVvxpUJ67DbOficfG7tCxGSWUQyM1icDibWl2CO8YlzdxqPJ8GOicA/640?wx_fmt=png&from=appmsg&watermark=1)

接下里就是

```
save11=save10^arr[i] //1103978709// 4+0 = 4save12=save11<<13//-1407541248


```

```
save13=save11>>> (32-13) //2105


```

```
save14= (save12|save13) >>>0// -1407539143 -2887428153 


```

```
save15=save14^save11

```

然后是 23  和 0  就可以得到结果 71273853

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR0aPiaib8mmpic1lVcepicj5WC9DJBDiaA3IrD8SH5BgXW5ST26Q4hbcD2nQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRVIo5iaicibYpnu01mbBITL7wamTzicmE6uAeqWp9qLSmjCIepBNQkn2w7A/640?wx_fmt=png&from=appmsg&watermark=1)

按照这个方法皆可以拿到 第一个 36 位数组

接下来的操作跟上面差不多 会通过你传入的参数组成 新的四位数组 与你刚生成的 32 位进行一些位运算

然后拿第二次生成 最后四位倒序 得到生成结果

![](https://mmbiz.qpic.cn/mmbiz_png/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeR8mlXF0B5tWvSokx7ef1024U9REKwzAmicTN9OfcNVib0YKX72xKwOVpA/640?wx_fmt=png&from=appmsg&watermark=1)

结果与网页一致

致辞结束

有兴趣可以一起交流

![](https://mmbiz.qpic.cn/mmbiz_jpg/cpaZyYcQ8zpJaL9JezMO1ZVWkVjAhBeRyqdwSI0kQ3J8bzWPrg3Z9kw4uydiaTnnZpwMuWa3PqicRtSt0PUstWkQ/640?wx_fmt=jpeg&from=appmsg&watermark=1)