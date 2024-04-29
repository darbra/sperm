> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nuLiYFMZclAcg9qbJCmFhg)

```
一





Basic protection

```

使用模拟器打开游戏，发现进入不了猜测是检测，使用 syscall hook 获取全部系统调用，发现使用 prctl 和 seccomp 后程序卡死。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM0TdHel6E1C2AzDkLjP5CzD0wAEFD94ic9QDXdSe7ibNPwPYNoSVkXrsQ/640?wx_fmt=jpeg&from=appmsg)  
  

[Handler_Syscall__NR_seccomp] __NR_seccomp(0x1, 0x1, 0xbc9bac30) libUE4.so+0x46D293B

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMVyjpRs3nahwIqO3XafniciaYKicedOIJ2oZY2EpQyKURBricBUZnu2Hib4g/640?wx_fmt=jpeg&from=appmsg)  
  

Hook 调用 seccomp 将 prog 数据获取下来，分析发现进行系统调用号拦截 0x129 __NR_recvmsg 杀死线程，将 prog->len 设置为 0 游戏正常运行。  

```
sock_filter[0] { code: 0x20, jt: 0x0, jf: 0x0, k: 0x0 } LD W ASB
sock_filter[1] { code: 0x15, jt: 0x0, jf: 0x1, k: 0x129 } JEQ(0x129)
sock_filter[2] { code: 0x6, jt: 0x0, jf: 0x0, k: 0x0 } RET_KILL_THREAD
sock_filter[3] { code: 0x6, jt: 0x0, jf: 0x0, k: 0x7fff0000 } RET_ALLOW


```

26 秒后游戏调用 access 进行环境检测，环境异常游戏闪退。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM7lcctziaTo7icRrIglNq0KkIlDGwQbTzB3nZRcXClymKA4Lg4ludXcTQ/640?wx_fmt=jpeg&from=appmsg)  
  

这里捕获到 access 两个返回地址：  
1.libUE4.so+0x46D1FE5  
2.libUE4.so+0x46D2093  
可能是不同的检测。

  
分析 libUE4.so+0x46D1FE5，我给检测函数命名为 access_check_risk_46d1f8c，调用来源于 libUE4.so+0x46d1be4。  

使用 Syscall(0x21) access 检测是否存在异常文件。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMpON8NC9funncT4jLtaPyztuiarlQb0XEs8U6R3PmO4U5JO47g1uYtxA/640?wx_fmt=jpeg&from=appmsg)  
  

分析函数 sub_46d1be4

一、get_main_base_46d1d38
-----------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMtn5IzKdT8jznlom9lClOIWriaGTpQuWUeUEVhTz8nVsfF0fUCSW3Mpw/640?wx_fmt=jpeg&from=appmsg)  

还原代码

```
auto f = fopen("/proc/self/maps");
auto base;
while(buf = fgets(f)) {
    if (strstr(buf, "libUE4.so")) 
        base = strtoull(buf);
    }
close(f)
return base;


```

二、init_crc_buf_46d1e74。
-----------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMQm0eTY8V5mSWicGiaLzRkayzjwatKahwicgxu0bdibF3kSbpQia6UkyCc0A/640?wx_fmt=jpeg&from=appmsg)  
  

分析地址 0x046d1ef4 向内存中 libUE4.so + 4FA9950 地址写入 0, 0x77073096，根据这个判断是 crc_buf。

三、get_buf_crc_46d1f28(key, (_FINI_1 + libUE4.so), 0x36cc964);
-------------------------------------------------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM5JQWq9icc7ZZxJlVMKMFQP31qsK1qpXcjhibrYSM1GsSnnQImymCribIg/640?wx_fmt=jpeg&from=appmsg)  
  

这段代码从_FINI_1 到_FINI_1+0x36cc964 正好是对应着. text 段开始和结束，返回的 crc 值存入 data_4fa9d50 = crc，结果判断 if (crc == 0xd18b51ab) 。

四、access_check_risk_46d1f8c，上面分析过
---------------------------------

五、anti_frida_46d2038，反汇编和内存都看得见
-------------------------------

```
access("/data/local/tmp/frida-server") access("/data/local/tmp/re.frida.server")


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrML3FCuWh9qrAByhvrNbFCP97RcrsmWutfib5yTovIogyOnBOr1RkbzGA/640?wx_fmt=jpeg&from=appmsg)

六、anti_frida_046d20e4
---------------------

```
auto f = fopen("/proc/self/maps", "r");
while (buf = fgets(f)) {
    str = get_str_frida(); // 46D21BB "frida"
    if (strstr(buf, str)) retrun 1;
    str = get_str_gadget(); // "gadget"
    if (strstr(buf, str)) retrun 1;
}
fclose(f);
return 0;


```

七、anti_frida_46D2261
--------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMYVuE6uVewgzHgXb2jrCicicGxqJnJh7mZ1gLb19BnYgllibOguD82wuOA/640?wx_fmt=jpeg&from=appmsg)  
  

这里可以看得出来是去遍历 / proc/self/task / 全部线程。

```
while(ptr = readdir(dir))  {
// 过滤目录
if(strcmp(ptr->d_name,".")==0||strcmp(ptr->d_name,"..")==0) continue;

// 获取文件内容 /proc/self/task/{taskId}/status
    sprinf(path, "/%s/%s/status", "proc/self/task",  ptr->d_name);
    auto f = fopen(path);
    buf = fgets(f);

    // 判断是否包含{"gmain", "gum-js-loop", "frida", "ggdbus"}
    for (auto str : {"gmain", "gum-js-loop", "frida", "ggdbus"}) {
        if (strstr(buf, str.c_str())) return 1;
    }
}
return 0;


```

八、kill_46D26D5
--------------

以上检测函数如果有返回 1 则会调用 memclr 让程序闪退。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMs8yPQ9sST9GaD4JZp6cdmicj9ZmVthgcTvK6znRZmicR6ziaic3emukSIA/640?wx_fmt=jpeg&from=appmsg)

```
二






绕过方案

```

1. 直接将 start1 和 start2 写入 return 0
----------------------------------

```
// start1_anti_hack 过检测1
DobbyHook((void*)(UE4 + 0x46d1be4 + 1), (void*)sub_46d1be4, (void**)&sub_46d1be4_orig);
// start2_anti_hack 过检测2
DobbyHook((void*)(UE4 + 0x46d1afc + 1), (void*)sub_46d1afc, 
(void**)&sub_46d1afc_orig);


```

2. 绕过 CRC
---------

将正常游戏. text 拷贝到自己申请的内存中，创建一个 maps 文件，内容写入刚刚创建的地址十六进制加上 libUE4.so 让检测识别我们拷贝正常游戏. text 的地址为 crc 目标地址，在通过 hook 系统 fopen 函数对打开 / proc/self/maps 文件重定向到自己的 maps。

3. 环境检测
-------

游戏检测调用的并 svc 而是 libc 中的 syscall 函数，因此可以在这个地方进行 syscal 拦截，判断__number == 0x21 这直接将 access 的 path 路径改成空这样就绕过检测了。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMytW8zXY1wo2L6icXaGSOO2qHE1zrFoOvaUzY8TB88XaNFoPCYjfq4oA/640?wx_fmt=jpeg&from=appmsg)

4.frida 检测
----------

hook fopen 函数判断 path 是否包含 / proc/self/task，将 path 重定向  
hook syscall 函数判断__number == 0x21，将 path 重定向。

5.seccomp
---------

可以直接修改过滤器的字节码将 00000006 00000000 (RET_KILL_THREAD) 修改为 00000006 7FFF0000 (RET_ALLOW)，或者使用 hook syscall 拦截__number == 0x17F。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMOxs8GKxO6DT4lnVrWf74975fTwDodj0lgu78wlez6JoY6w47gTsz9Q/640?wx_fmt=jpeg&from=appmsg)

6. 无敌字符串大法
----------

可以发现很多字符串会被解密到. bss 段这也就意味着没有 crc，观察被加密的字符串结尾都是以 00 01 结尾，没错 01 就代表这个字符串被解密了，这里就可以修改字符串实现绕过检测，当然在真正的 tersafe 中字符串会被二次检测。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMocQSK8fRomCcQcIwRCEliciccEe0uXmDH8zLicWILvjctTohaL2ibs8qicg/640?wx_fmt=jpeg&from=appmsg)

```
三





Section 1

```

分析游戏登录
------

```
sub_19f4304(utf16 input){
    Buf = malloc(0x100);
        sub_223c524(to_utf8(input), Buf );
        // 检测返回Buf是否等于;
        return Buf == { 0x3D, 0xF2, 0x2C, 0xF8, 0x8F, 0xFB, 0x47, 0x5B, 0x49, 0x04, 0x78, 0xD9, 0x4E, 0x31, 0xEF, 0x3E, 0xA1, 0xA7, 0xAA, 0x7B, 0xCF, 0x72, 0xA8, 0xBC, 0x53, 0x2B, 0x67, 0x00, 0xB2, 0xB0, 0x32, 0xFA }; // 这个后面回用到
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMibxk5DQtdPe3YXE9N8Q877ocbIFOicFhg207kZyPqPUAibmLhibaGjSILQ/640?wx_fmt=jpeg&from=appmsg)

虚拟机入口分析
-------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMyhysyEibEhoFE384nwTpRSqdWoowRPQF1xianfL4SPSLqOJJibrjwqwDw/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMBoxiakVf97doRbVzw5z972oLlogIiaFNHgiax1ujPVicNZB9TUoIKbr3Dw/640?wx_fmt=jpeg&from=appmsg)  

初始化寄存器 0~17 赋值 = 0。

```
R0 = 0x4FA9D54; // key字符串地址tencentgamesecfs
R1 = 0x10 ; // key字符串长度
R2 = Input; // 输入字符串
R3 = output; // 输出数据
(*SP) = outputSize; // 0x20


```

初始化虚拟机字节码
---------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMSDeohQ8bwLWETcskfUSTTAJe3dZbukTZxSOVgpPbCgflk6kZtNZMhw/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMYIQhPHwGFsYZo5FIOtLSuPa93hr7L5iaicGkIWxG3BrfibNBNPuzzgewg/640?wx_fmt=jpeg&from=appmsg)

vm 分析与还原
--------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM8VxMYQbwKB7Oe5GQO9XHyx52kUx8rhG5ialmEVkMtxOxHRkibX7md2DA/640?wx_fmt=jpeg&from=appmsg)  
  

ctx + 3C 获取 pc 寄存器地址，取指令后 bswap，opcode >> 27 执行对应 handler。（因为取指令的肯定是 PC 寄存器啊）

虚拟机介绍
-----

其实一开始我挺蒙的，直到我看到这个 get_bit 函数，一切都合情合理了在去年比赛的时候用到的是 Arm64 虚拟机，Arm64 解析里面使用各种 asm_ubfx 无符号位域提取，解析指令需要的位域，今年则是 Arm 虚拟机。

  
具体查看：[原创]2023 腾讯游戏安全大赛 - 安卓赛道决赛 "TVM" 分析与还原。（https://bbs.kanxue.com/thread-279011.htm）

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMbKVmTJyqyE07icVE2icicuib8zEYHzGhxpfD5OibZujPxVxpQicDbjIib06Eg/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMJ7FgXjWnfEUIaThS0iaAWNve77ZFNg078anCKrDDibGwbufNToo0iaSibw/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMddXWm8z9qf97UfukOLd7oqzIQLm3ok1c7GL9P7CgxOBmMm8JSaAabA/640?wx_fmt=jpeg&from=appmsg)

逆向还原 Handler
------------

比如我现在要还原 handler：6，opcode：30020000  
[SP, 4] = get(opcode, 0x16, 0x1B); // = 00000000

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMALrKlucFTH71h30CsnobIlNPUvAc8rX20WLgSLyLqrOGRWd40hF1Sw/640?wx_fmt=jpeg&from=appmsg)  
  

[SP, 8] = get(opcode, 0x11, 0x16); // = 00000001

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMcKPqmPu1cRGEJPnDlINP9esHwV8pXdkyErI25K75VT04FEgQpaQb7w/640?wx_fmt=jpeg&from=appmsg)  
  

R0 = get(opcode, 0x0C, 0x11); // = 00000000

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM4B0yMa3F08m7y7DJF9opBMS1ficI4kQTYtQ9kibz7jEwMHkUYojPgq6w/640?wx_fmt=jpeg&from=appmsg)  
  

将输入寄存器 1 值和输入寄存器 2 值进行 ROR 然后写入输出寄存器，这就是指令解析过程。

  
Handler6 实现为 ROR，当然这里只有一种情况，有些指令内部又有多种情况。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMnUZlzWonT4SBUzb8BAaPLF8DZT5WByuic3JiaRPNtZia0QiaCOlINoQFfQ/640?wx_fmt=jpeg&from=appmsg)  
  

接着解析剩余的 Handler 也是成功扣了一天一夜。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM5rCo6xwBIDvYAYQ95A9eiaIQxEdcBqplEnMiaQdEk5g35m5GkNFphAfQ/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMuDtIBNjib3NznaDVPuUnsLjOmQSaBUtWLWDK4ia8gXYLt35ibcLHH8bFA/640?wx_fmt=jpeg&from=appmsg)

vm_to_asm
---------

将虚拟机的字节码 Dump 下来这里我用的 Dowrd Hex，我称它为 vm_shellcode。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMnKYbqM0SqQjNBVodKLZdd6iaPQj1cNc77GLHlob1dRHYo4VBTROv4TQ/640?wx_fmt=jpeg&from=appmsg)  
  

开始调用解析函数，从 vm_shellcode 的 0x7FC 地方开始解析（当然你想解析哪里都行）。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM7cXzNP0t2RtSvYfUC0N8Ybe0jlBiaukZzgYNia2pQvgLW6icrl8WcXVNw/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMJOKGicUQnBnV9c7labuhQuWLWxnW94VAbG70AZIvvbFe5JYQJtdfymw/640?wx_fmt=jpeg&from=appmsg)  
  

解析出来 Arm32 汇编，在通过 ARM to HEX 网站把汇编转换成十六进制字节码。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMsdcXzc3Sk0ORof6IgexXKe4v7rLw5MKEobyCUCSbALZfBpbVoyJJsg/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMuh3dHGvGwLMpMGUvgTBSBSauTVg4icR51LUogTh6AoxmvK78O7RL3UA/640?wx_fmt=jpeg&from=appmsg)  
  

然后将 ARM to HEX 生成的字节码替换 vm_shellcode。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMnvtEBklVj4NqH5u1vvgAw3ibesLeBoeskfvKMLRxQj7nGkMIokQXxuw/640?wx_fmt=jpeg&from=appmsg)  
  

将文件写出 vm_to_asm.img。

asm_to_c
--------

vm_to_asm.img 拖入 ida pro。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMnib3B5ehbqvb0ia0MWQ8XSm2cvV1PGEqnHsficquWkzPjNDy7xpZTX6Nw/640?wx_fmt=jpeg&from=appmsg)

算法还原
----

FindCrypt3 插件查找到 AES_Rijndael_rcon 算法。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM9lCe0bZnoQmZs0x81Y3zbR9njJbfFB1eI1icmfCWIZe7RErQapzRyoQ/640?wx_fmt=jpeg&from=appmsg)  
  

将函数对应名称补齐，发现没有 uint8_t* iv 且多次使用 inv，函数为 aes ECB Decrypt 算法。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMbQZfNO1CVNX3K0dBKQJpw5lBpgejXF9LsMInl8IhZP16lMIUeBTHrg/640?wx_fmt=jpeg&from=appmsg)  
  

完整还原后发现有几处变化，aesKey.dK 变成了 aesKey.eK、invShiftRows 和 invSubBytes 顺序颠倒、invMixColumns 被替换成了 mixColumns 与 addRoundKey 顺序颠倒。

验证
--

将解密算法用游戏同样的方式修改，在游戏内输入 32 个 1，验证一下解密数据相同。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMvbzicl5j2pbNVWVlTqRx6CJfqN5GooY8XDH1VvichgE0bRTWAA6qm3Bw/640?wx_fmt=jpeg&from=appmsg)  
  

将加密算法用游戏同样的方式修改。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMBPvH3ialvXyNFyDmd2vPWwTrZfFlkZCicvibIjDWcI7dO8CgRWEbiaIA0g/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMh6l8cloQmR1nvSgTj2xMng5edguVgcMH7ibmDlXzIwoHg39nv6KicXOQ/640?wx_fmt=jpeg&from=appmsg)  
  

将 login 验证结果 (不懂请返回分析游戏登录部分) 进行加密，游戏内输入加密结果登录成功。

  
登录密码：dde8cdf098e8434b93f04f86085a88f9

```
四





Section 0

```

一、方框透视
------

### 世界坐标转换_读写路径不同

#### 1. 使用 ViewMatrix

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMfTy0GUpsdgIA6ACC4qM2HsSFIeUMkUBpujOPuX27w4PpYyMenBlJCQ/640?wx_fmt=jpeg&from=appmsg)

#### 2. 使用 CameraCacheEntry

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMP7s2Rr0lcTjqwxiabBluNsAzHom7COFm3CD7G7QfXrxKdT8eFPWN67w/640?wx_fmt=jpeg&from=appmsg)

#### 3. 使用 ProjectWorldLocationToScreen

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM0XctquP1r37ILN1wCurlAa2NNsbjJOyvANOHMnasH929YnJjo9YlRg/640?wx_fmt=jpeg&from=appmsg)

### 获取世界坐标_读写路径不同

#### 1.Transform

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMYVbcwtR6P4Nr4WTKfPcUcCNanxUJL8LZ2UrGRWC7FFqfWUuLFIYA0w/640?wx_fmt=jpeg&from=appmsg)

#### 2.RelativeLocation

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMFB8TWFh9EVHUHee50iamhavFzEaqaTmUnVlDxUKtftx48VLDItiburBg/640?wx_fmt=jpeg&from=appmsg)

#### 3.GetCenterOfMass

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM2jj65nCE1q22H816noYS7O0JcbGl8SQDfz2QCUd1ckc8gk4obShrUA/640?wx_fmt=jpeg&from=appmsg)

#### 4.GetActorBounds

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM4YEZA5WhTOOWuHS9VG0IWTmyiatRYNNibXNpbmvGWFpvjUCAnzwK6a3g/640?wx_fmt=jpeg&from=appmsg)

### 使用不同排列组合绘制出来

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMT0Nlcq01EPZgunGYicibquFZdg4u2S6blvSznDRUrZmiaf39aRwys7nVg/640?wx_fmt=jpeg&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrM5hHqdjkDlfPqw7juibGxMOibUqDF1aeea9zIA3xYSj08jB7MJ86YZOZA/640?wx_fmt=jpeg&from=appmsg)

二、自动瞄准：按下屏幕时自瞄，当 ControlRotation 触发写入时停止自瞄
------------------------------------------

调试移动屏幕时触发写入 ControlRotation  
调试移动屏幕时触发写入 ControlRotation  
DobbyInstrument((void*)(UE4 + 0x3622218), ControlRotationSet);

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMRICmEqviaqnAb1j7CHybOXnrYfIyOaz7XSEz4lAOk3z6Sdsic4IAUibXw/640?wx_fmt=jpeg&from=appmsg)  

实现自瞄的逻辑

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMmpicRcaOicpdMP8sEESvQXWxGAdotaZMMAtcib3eRC7sicsoQKdAv31Bgw/640?wx_fmt=jpeg&from=appmsg)

```
// 方案一 修改 ControlRotation
ControlRotation->x = rotation.x;
ControlRotation->y = rotation.y;

// 方案二 Call SetControlRotation
((call)(UE4 + 0x3622054))(PlayerController, &rotation);

// 方案三 输入 ControlRotation 增量
// ((call)(UE4 + 0x39C0D24))(PlayerController, 10.f);


```

End
===

易语言 vm_to_asm.e 见原文附件。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HoHYzico0cmy1FK40LE8QrMQGmJ0zJPXqwGiciapaByZFp0kCo3C4vuRfs5pDleomLiboMAD8iahyyVDA/640?wx_fmt=png&from=appmsg)

  

**看雪 ID：a'ゞ Cicada**

https://bbs.kanxue.com/user-home-927003.htm

* 本文为看雪论坛精华文章，由 a'ゞ Cicada 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FQicadOCiboLpxXIRw7OaqYTKNZdiawSaMcFqIgPffYkEQDTf3Vs9OhQ1JCkm7ibC6RLicrKUXSAy0Q2w/640?wx_fmt=jpeg&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551171&idx=1&sn=431e80c01dcbbcf125888660d147922f&chksm=b18db30986fa3a1f15c06232425f27c882e304b7be1fb3a7e564c03e76330c093cf2fe6d1625&scene=21#wechat_redirect)

**#** **往期推荐**

1、[Windows 主机入侵检测与防御内核技术深入解析](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458552007&idx=2&sn=65017982b17cb3930d3276412ecc381a&chksm=b18db64d86fa3f5b47e5fc5724be0a749a1018959bc52e3f332ec1e3a7a428a663f1fc22d425&scene=21#wechat_redirect)

2、[系统总结 ARM 基础](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551831&idx=1&sn=88052805c8c43e2f05a4c86e85aeedea&chksm=b18db69d86fa3f8b34c32693f09b0cbc9bef7ff6b253ef9e60aab30464fb98f9de51416350ff&scene=21#wechat_redirect)

3、[AliyunCTF 2024 - BadApple](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551651&idx=1&sn=91b96cb3efd841999f8381d2f86e2247&chksm=b18db5e986fa3cff8edf467098f6e0dd1bdc8ec3ebb6365ee2ca759e93cd9c509f32c0eb0877&scene=21#wechat_redirect)

4、[VM 逆向，一篇就够了](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551587&idx=1&sn=cf16b3a90a87799576eec5c885ea791b&chksm=b18db5a986fa3cbf4dcf479f4c28de6d1c3ecd101ba5f7d91c1021d51c836a7339971a184239&scene=21#wechat_redirect)

5、[一次恶意程序样本全分析之旅](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458551276&idx=1&sn=1e7a75c39d91578861cc3425bedf9f3e&chksm=b18db36686fa3a70aec2f49454771fbbdef4cca493121437000fec3855dbb056817b48c15b10&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78321RiaLpp3FAylJv0nbibloCFmXdVe4wvW4ibgnCc6srNI8sGBkX14MpQ/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GJubmq65v9uBFmEJuoJD78txPhfvI9WpuGSCawCN8NJCgzD16Y0IwdUkaI33Qr3DpwRRuvibgRQOg/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多