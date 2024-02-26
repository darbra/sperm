> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.kanxue.com](https://bbs.kanxue.com/thread-280302.htm)

> [原创] 某 app 加固逆向分析

[原创] 某 app 加固逆向分析

2024-1-23 14:30 9993

### [原创] 某 app 加固逆向分析

![](https://passport.kanxue.com/pc/view/img/moon.gif)![](https://passport.kanxue.com/pc/view/img/moon.gif)

* * *

小伙伴公司需要代码静态分析安全性，上了壳，就逆向分析了下。

Apk 打开没有太多的代码，在自定义的 application 中加载了一个名为 **Helper 的 so。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_C889DC64K9H82WJ.webp)  
在 doAttach 函数中能够找到反射调用原 application 的初始化。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_RC4YPF5745S6T66.webp)

![](https://bbs.kanxue.com/upload/attach/202401/753755_B9SPB3CTZSEXYWC.webp)  
打开 so 查看 dynamic 表，存在. init.proc，initarray 只有一个函数。  
再查看 jni_onload 函数，发现密文，猜测是 init_array 或者 init_proc 中解密 so 中代码段，修复代码段必定涉及 mprotect，要么调用 libc 要么 svc 自己实现，能够快速找到可疑位置。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_85P73NUYS5JW3WG.webp)  
Init.proc，调用 sub_10150C，再调用三次 sub_1011A0 函数修改内存权限，通过 svc 调用 mprotect 修改该 so 在内存中的权限，修复该 so 原本的代码。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_AF5C2DCWAXXJXYB.webp)  
![](https://bbs.kanxue.com/upload/attach/202401/753755_PKEMESVWJRNAKB7.webp)  
Dump 代码段，修复 elf，再次查看 jni_onload，ida 正常解析。

大概看了下，有些功能没有开启的。  
**0x3D2B0** Fork 后 Ptrace，开启各种检测线程。  
**0x32D08** anti_thread_body 函数检查 tracerPid 和 status  
**0x4EEF8** waitpid 子进程是否终止。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_BH4PQ3CXWZZD5XD.webp)  
**0x2F488 **do_hook_log  
![](https://bbs.kanxue.com/upload/attach/202401/753755_N5QC43WQA2VA8D6.webp)  
0x2E624 run_addition 函数，获取 ro.debuggable 和 signatures 等等  
sub_50B44 文件 hash 检测。

**0x3D494** 查找 dex 的指针，找到标志位。通过 java 的 DexCache 类 dexFile 找到 dex 指针，这我也第一次知道这个结构，可以这么定位 dexFile。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_UXKC7UKXSQMMD74.webp)  
![](https://bbs.kanxue.com/upload/attach/202401/753755_UB9MJGSNQ5MBAXE.webp)  
sub_4F8BC 计算 dex 大小，申请一大块内存，存储解密后的完整的所有的 dex。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_FZW6FGG6VRFJNHN.webp)  
0x3E10C 申请内存，解密 dex 的密文部分，并拷贝回上一步那块内存，加密算法，晕了。  
![](https://bbs.kanxue.com/upload/attach/202401/753755_A49YTUVR9FM6UCZ.webp)  
dump 后，得到 11 个 dex, 收工！

附件是修复后的 so，很多字符串明文，很容易理解它的逻辑，仅供学习。

[[CTF 入门培训] 顶尖高校博士及硕士团队亲授《30 小时教你玩转 CTF》，视频 + 靶场 + 题目！助力进入 CTF 世界](http://www.kanxue.com/book-brief-170.htm#h3a6WRhDT9Q_3D)

返回