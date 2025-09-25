> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.kanxue.com](https://bbs.kanxue.com/thread-288596.htm)

> [原创] 基于 trace 内存爆破标准算法

[原创] 基于 trace 内存爆破标准算法

发表于: 2 小时前 332

举报

* * *

#### **起因**

前几年看 Android 逆向的好文章，即觉得美妙，又有些烦躁，烦躁中还带着酸味儿。真的有这么丝滑吗？问了就是靠直觉和天赋。直到最近有师傅问我怎么逆出来的问我流程，才恍然大悟。故记录出来，己所不欲勿施于人。 文章思路来源于龙哥。

一、概括

几乎每种算法，都存在某些固定依赖的常数和表，我们称之为魔数和常量表。第一步肯定是识别，而他们中的明文，密文又会在内存中出现并且使用，我们可以基于内存读写监控，做内存追踪和数据溯源。

二、算法分析 例一

请出我们的老演员 某咖啡 演员是发帖本天下的最新版 ![](https://bbs.kanxue.com/upload/attach/202509/813639_ZS3N8E25N49HAQT.webp)  
开始 trace

众所周知，每个 native 函数执行需要经过 artJniMethodStart 这个函数 ，我直接在这里开始跟踪，第一可以绕过本身函数的 inline hook 的检测，第二不需要 hook 注册函数符合我们快速逆向的标准。  
![](https://bbs.kanxue.com/upload/attach/202509/813639_TBEBEUGYBE5M5G4.webp)  
trace 的话目前外面已经非常多的方案拿到了，我在这里讲讲自己使用的，不局限。

目前是通过 stalker 和 qbdi 结合来使用 你问我为啥这样用 因为 frida 老相好不能丢，qbdi 功能强大得用。

通过 stalker 的 GUM_BLOCK 和 qbdi 的 vm.run 来结合

<table><tbody><tr><td><p>1</p><p>2</p></td><td><p><code>GumEventSink </code><code>*</code><code>sink </code><code>=</code> <code>gum_event_sink_make_from_callback(GUM_BLOCK, sink_callback, nullptr,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>nullptr);</code></p></td></tr></tbody></table>

![](https://bbs.kanxue.com/upload/attach/202509/813639_J7VNN92F33BG65V.webp)  
qbdi 直接就有内存读写监控的 api 我们直接拿来使用

<table><tbody><tr><td><p>1</p></td><td><p><code>vm.addMemAccessCB(QBDI::MEMORY_READ, memory_r_dump_call, nullptr);</code></p></td></tr></tbody></table>

通过读写的地址的后 100 位 hex 写入到文本中

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></td><td><p><code>oss &lt;&lt; </code><code>"&nbsp;&nbsp; accessaddhex:"</code><code>;</code></p><p><code>for</code> <code>(size_t i </code><code>=</code> <code>0</code><code>; i &lt; </code><code>100</code><code>; i</code><code>+</code><code>+</code><code>) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>oss &lt;&lt; std::</code><code>hex</code> <code>&lt;&lt; std::setw(</code><code>2</code><code>) &lt;&lt; std::setfill(</code><code>'0'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;&lt; static_cast&lt;</code><code>int</code><code>&gt;(accessadd1vdata[i]);</code></p><p><code>}</code></p><p><code>fputs(oss.</code><code>str</code><code>().c_str(), fp2);</code></p></td></tr></tbody></table>

注入 so 拿到 trace 文本  
![](https://bbs.kanxue.com/upload/attach/202509/813639_PKEK2NWNU8G3NM8.webp)  
把文本拉到电脑上进行下一步

首先得识别算法用 010 打开 通过脚本来识别读取的内存地址 hex 文本  
![](https://bbs.kanxue.com/upload/attach/202509/813639_5AEYXKR83DHVGZE.webp)  
很快识别到 aes  
![](https://bbs.kanxue.com/upload/attach/202509/813639_HZFWTF8KBZXWRZY.webp)  
把 trace 读取的文本拉到 golang 中进行内存爆破 ![](https://bbs.kanxue.com/upload/attach/202509/813639_QUS5C3ZA5R2BFKY.webp)  
很快就爆破完了 拿到 key 结束了

整个过程甚至没有打开 ida, 所以有的师傅跟我唠流程，ida，白盒，dfa，hook，我自己都不知道还咋教你，我就 hook 了一个 art 函数，so 名字和地址都不清楚

原理就是拿着结果去内存搜索爆破，代码会上传到附件  
![](https://bbs.kanxue.com/upload/attach/202509/813639_4RYZJYMTETY62VS.webp)  
代码优化 有师傅看到了 这一个算法一个模版不得累死 那就用 c++ 写个复合模版吧

ai 花了一会就写好了  
![](https://bbs.kanxue.com/upload/attach/202509/813639_UDHXPC6N8JU3JWE.webp)  
完结撒花 所有脚本和代码会放到附件 下篇文章给师傅们唠唠 vm 逆向流程

如果你对 签名校验 环境检测 以及移动安全 算法还原 有兴趣 可以加我一起交流学习 我打算建一个学习交流群  
![](https://bbs.kanxue.com/upload/attach/202509/813639_U44STE9SARWF74B.webp)  
![](https://bbs.kanxue.com/upload/attach/202509/813639_7K5J5GA7DY9483B.webp)

[[培训] 传播安全知识、拓宽行业人脉——看雪讲师团队等你加入！](https://bbs.kanxue.com/thread-275828.htm)

最后于 17 分钟前 被蜕无痕编辑 ，原因：

上传的附件：

*   附件. zip （767.48kb，14 次下载）