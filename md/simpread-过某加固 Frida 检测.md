> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RzdyDWRZ4un2_mC4Om8XZg)

过某加固 Frida 检测
=============

现象
--

通过 jadx 分析可知使用了某厂商加固

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3wvq5RiczWR3I0JO4hI7459e5omKhz3LkJ09L5978kRnNJaeRXHhZOlA/640?wx_fmt=png)

使用 frida 启动的时候，会退出应用

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3nM5uYOrmumPoD7vjMBw3gZkemMQqt5bxns2HowWdiaMGaJ6aGb5zTvw/640?wx_fmt=png)

后多次尝试发现，只启动 server 时，也会使程序退出

推测该应用检测了 Frida

定位
--

直接在 lib 目录里搜索 frida

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3LQ4rs7apINDGn3tNcGO8iajAqNLB0mYEVTA7fKB6TUsYuPARH6cSbWw/640?wx_fmt=png)

通过 010 逐一排查，只有红框中的 so 是 “LIBFRIDA”，其他都是 Friday。红框中正是壳 so，我们只需分析它即可

Dump So
-------

首先把 libxxx.so 拖入到 IDA 中，发现报错

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk37iabNlwY1EOzBOQQRzeia2A3s82YH0mB7XPOosl8KcxEKicW9qHRYa2uQ/640?wx_fmt=png)

继续后发现

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3boIiarv8H7tUufpApvTBMHoCg98NSTcbgLqqUMmicMlOVF9XFMDPDdXA/640?wx_fmt=png)

为了防止我们的静态分析，它已经把 section 段的结构打乱了

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3lic7lTyicjhiaDkNlwlibz8Bked60QQNfQ2iaS1dqmpOA8FQzUL6YvVdpzQ/640?wx_fmt=png)

ELF 两种视图：链接视图、程序视图，执行的时候只用知道 Program 中 Segement 即可。

正常情况下 so 加密分两种，一种是 so 自解密可以把解密方法放在. init_array 段，另一种是先加载一个解密 so A，再加载加密后的 so B，让 A 去解密 B。不管哪种为了正常运行，内存中的 so 都应该是解密后的。

使用 GG 模拟器 Dump 内存中的 so，首先先让程序运行起来。

打开 GG 模拟器，选中我们要 Dump 的应用，选择最右侧栏，打开选项选择导出内存

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk35xJFet944iayAKDbP62p4MfHXAML5lkfnow7mwMGXBBhHZfjYXsJw3Q/640?wx_fmt=png)

选择起始地址，点击保存，导出 dump 下来的 so

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3IQZkrIZQyBgzOxJHLFm6kjpWGbUXpmvibx6zcGFrHXQ90eF4ysTD5cQ/640?wx_fmt=png)

因为程序运行的时候是不需要知道 section 的，只用知道 segement 就行，但是静态分析又需要 section，所以我们要通过从内存中导出的 so，去修复 section 段，让 ida 能够正常反编译。

SoFixer[1]

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3FXXGOgHZFoqXgIc58R1AUpwRntn3dvskw3JtkIp7349Tibu43Uxfl9w/640?wx_fmt=png)

-s 添加需要修复的 so

-o 要保存的路径

-m 我们 dump 的起始地址，gg 模拟器已经自动将开始和结束地址保存下来了，只用加开始地址就行（16 进制形式）

-d 输出信息

分析
--

这时候再拖到 IDA 中分析，看到代码已经还原出来了

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3McfKib3vNeHFWy8P4p85IeOyWBHE0nnK2ABSdBvjrQh2mibYdSIoaJyg/640?wx_fmt=png)

首先我们得先了解一下有哪些检测 Frida 的手段

1.  1. 遍历运行的进程列表从而检查 fridaserver 是否在运行
    
2.  2. 遍历 data/local/tmp 目录查看有无 frida 相关文件
    
3.  3. 遍历映射库，检测 frida 相关 so
    
4.  4. 检查 27042 这个默认端口
    
5.  5. 内存中扫描 frida 特征字符串 “LIBFRIDA”
    
6.  6. frida 使用 D-Bus 协议通信，我们为每个开放的端口发送 D-Bus 的认证消息
    
7.  7. 检测有无 frida 特征的线程名，如：gum-js-loop
    

其实通过刚才的现象结合着这些检测点我们已经大致能推测出是哪种方式检测的了，因为还没有使用 frida 附加，仅启动 server 就会使应用退出，那么检测点 1、2、3、5、7 都不符合。

我们就推测它通过检测点 6 去检测 Frida，向每个开放端口发送认证消息，认证消息里会包含 “AUTH“字符串，我们直接在 Strings Window 里搜索 “AUTH“

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk3ggt4mGmagPmA1u2elwbKS9YG7pAa2z1GiblMBZab7373hr69CvnzuvA/640?wx_fmt=png)

查找引用果然发现检测代码

```
void __noreturn sub_2CA40()
{
  FILE *v0; // r11
  int v1; // r4
  __pid_t v2; // r0
  unsigned int v3; // r0
  char v4; // r0
  char *v5; // r1
  unsigned int v6; // r2
  int v7; // r5
  _BYTE *v8; // r6
  int v9; // r3
  int v10; // r4
  int v11; // r6
  bool v12; // zf
  unsigned int v13; // r0
  unsigned int v14; // r2
  unsigned int v15; // r6
  int v16; // r4
  __pid_t v17; // r0
  int v18; // [sp+0h] [bp-458h]
  int v19; // [sp+4h] [bp-454h]
  void *v20; // [sp+8h] [bp-450h]
  int v21; // [sp+10h] [bp-448h]
  unsigned int v22; // [sp+14h] [bp-444h]
  void *v23; // [sp+18h] [bp-440h]
  int v24; // [sp+20h] [bp-438h]
  int v25; // [sp+24h] [bp-434h]
  void *v26; // [sp+28h] [bp-430h]
  char v27; // [sp+30h] [bp-428h]
  int buf; // [sp+430h] [bp-28h]
  __int16 v29; // [sp+434h] [bp-24h]
  char v30; // [sp+436h] [bp-22h]
  struct sockaddr addr; // [sp+438h] [bp-20h]

  *(_DWORD *)&addr.sa_data[6] = 0;
  *(_DWORD *)&addr.sa_data[10] = 0;
  *(_DWORD *)&addr.sa_family = 0;
  *(_DWORD *)&addr.sa_data[2] = 0;
  addr.sa_family = 2;
  inet_aton("127.0.0.1", (struct in_addr *)&addr.sa_data[2]);
  while ( 1 )
  {
    v26 = 0;
    v24 = 0;
    v25 = 0;
    v0 = fopen("/proc/net/tcp", "r");
    if ( v0 )
    {
      while ( fgets(&v27, 1024, v0) )
      {
        v23 = 0;
        v21 = 0;
        v22 = 0;
        v3 = strlen(&v27);
        sub_138AC((unsigned int *)&v21, (int)&v27, v3);
        v4 = v21;
        v5 = (char *)v23;
        v6 = v22;
        if ( !(v21 & 1) )
        {
          v6 = (unsigned int)(unsigned __int8)v21 >> 1;
          v5 = (char *)&v21 + 1;
        }
        if ( v6 >= 0xA )
        {
          v7 = (int)&v5[v6];
          v8 = v5 + 9;
          v9 = (int)&v5[v6];
          if ( (signed int)(v6 - 9) >= 1 )
          {
            v10 = v6 - 9;
            do
            {
              v9 = (int)v8;
              if ( *v8 == 58 )
                break;
              --v10;
              ++v8;
              v9 = (int)&v5[v6];
            }
            while ( v10 );
          }
          v11 = v9 - (_DWORD)v5;
          v12 = v9 == v7;
          if ( v9 != v7 )
            v12 = v11 == -1;
          if ( !v12 )
          {
            v20 = 0;
            v13 = v6 - (v11 + 1);
            v14 = v11 + 5;
            v18 = 0;
            v19 = 0;
            if ( v13 < v11 + 5 )
              v14 = v13;
            sub_138AC((unsigned int *)&v18, v9 + 1, v14);
            if ( v24 & 1 )
            {
              *(_BYTE *)v26 = 0;
              v25 = 0;
            }
            else
            {
              LOWORD(v24) = 0;
            }
            sub_1E4F8(&v24, 0);
            v24 = v18;
            v25 = v19;
            v26 = v20;
            v15 = std::__ndk1::stoi(&v24, 0, 16);
            v16 = socket(2, 1, 0);
            *(_WORD *)addr.sa_data = bswap32(v15) >> 16;
            if ( connect(v16, &addr, 0x10u) != -1 )
            {
              v30 = 0;
              v29 = 0;
              buf = 0;
              send(v16, byte_72C60, 1u, 0);
              send(v16, "AUTH\r\n", 6u, 0);
              usleep(0xBB8u);
              if ( recv(v16, &buf, 6u, 64) != -1 && !strcmp((const char *)&buf, "REJECT") )
              {
                close(v16);
                v17 = getpid();
                kill(v17, 9);
              }
            }
            close(v16);
            v4 = v21;
          }
        }
        if ( v4 & 1 )
          operator delete(v23);
      }
      fclose(v0);
    }
    else
    {
      v1 = socket(2, 1, 0);
      *(_WORD *)addr.sa_data = -23959;
      if ( connect(v1, &addr, 0x10u) != -1 )
      {
        v30 = 0;
        v29 = 0;
        buf = 0;
        send(v1, byte_72C60, 1u, 0);
        send(v1, "AUTH\r\n", 6u, 0);
        usleep(0xBB8u);
        if ( recv(v1, &buf, 6u, 64) != -1 && !strcmp((const char *)&buf, "REJECT") )
        {
          close(v1);
          v2 = getpid();
          kill(v2, 9);
        }
      }
      close(v1);
    }
    sleep(4u);
    if ( v24 & 1 )
      operator delete(v26);
  }
}

```

建立 socket 链接，然后通过端口去发送认证消息，recv 函数去发送这个消息，如果不等与 - 1 表示能发送成功，比较返回结果中是否包含 “REJECT”，如果包含则说明存在 fridaserver

过检测
---

我们可以直接 Hook recv 函数，这个是 libc 的函数，让它返回 - 1

```
Interceptor.attach(Module.getExportByName("libc.so", "recv"), {
    onEnter: function(args) {
        // console.log(Memory.readCString(args[1]))

    },
    onLeave: function(retval) {
        retval.replace(-1)
        console.log("bypass")
    }
});

```

这时我们再次启动应用，发现已经可以正常使用 frida 了

![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBuljAdKNFDh8siaPSMgibTXNVk33eYQ2h1dvNDfsClias2NpLhG8odvbcx3oxQ6h8kiaxxiahgVvsG29Fibeg/640?wx_fmt=png)

#### 引用链接

`[1]` SoFixer: _https://github.com/F8LEFT/SoFixer_

[AOSP 编译保姆级教程（二）](http://mp.weixin.qq.com/s?__biz=MzI3Mzk2OTkxNg==&mid=2247485231&idx=1&sn=06bb88069289cf1b686a26dbba50b6ae&chksm=eb1a60bcdc6de9aa80a913c659f8c20df87de4e698c4f0e836b63e9f54e6538c3e7826e77c5d&scene=21#wechat_redirect)  

[Android 病毒分析基础（二）](http://mp.weixin.qq.com/s?__biz=MzI3Mzk2OTkxNg==&mid=2247485190&idx=1&sn=2ae591514733e03d72b79bd428bea06b&chksm=eb1a6095dc6de98368d5be4bdb8009965f9ac20f6058e6138d4e1c4b1af30292cbe75e57d8fd&scene=21#wechat_redirect)  

[ROOT 检测通杀 - 改造优化篇](http://mp.weixin.qq.com/s?__biz=MzI3Mzk2OTkxNg==&mid=2247485162&idx=1&sn=ea692f2c5cbed633a649ef7d27ff1722&chksm=eb1a6179dc6de86f057971834b196a28ea17bb56ec975610e7d960923e1cc0b31da9da2514f3&scene=21#wechat_redirect)  

[编译 Frida16.0.1 并魔改相关特征](http://mp.weixin.qq.com/s?__biz=MzI3Mzk2OTkxNg==&mid=2247485149&idx=1&sn=922d7be04644dcfad45f4ea775b5e4de&chksm=eb1a614edc6de8589c11a220a9a96b566f71b61bd46bdff823b3222e4de790ecf293a9e32b27&scene=21#wechat_redirect)  

![](https://mmbiz.qpic.cn/mmbiz_jpg/uVibiccmzBulhVCW50eBA98rxXekpIU3VXE5E2cxSjzYe4coRpZhsfxF0KbAm7dx17kSABJzJ2cDKbIHPiaRTdU3Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)![](https://mmbiz.qpic.cn/mmbiz_png/uVibiccmzBulhjUxulpjWt1s3EwkAX6nBD0pMhG7AjB4OLAFZ4qDHrfUjpUPrDTTSSzSSomvlibwGlNFG7lqQE83w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

随手分享、点赞、在看是对我们最大的支持![](https://mmbiz.qpic.cn/mmbiz_gif/uVibiccmzBulhVCW50eBA98rxXekpIU3VXnygTsE8U2uib95Ub2AicqZnyYbNycVt7YuJOSiao8IqIPCnyibY6JUsEnQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)