> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/5952c3701e3c)

前言：
---

现在市面上改机的软件很多，大部分都是修改 Java 层的一些参数和变量，去修改或者直接反射的方式去 Set 成自己修改过的数据。  
如果通过正常的 API 去获取设备信息的时候，就很容易被欺骗，导致获取的设备信息是被欺骗过的。

这时候我们可以通过读取文件的方式去获取设备信息，还需要加 CRC 对底层函数进行判断，很是麻烦  
因为底层的 IO 函数一旦被 Hook，比如 openat 函数，就算读取文件的方式去可能获取的设备也可能是假的。

### 这时候有没有一种相对稳定的方式去最真实的设备信息呢？

通过 syscall 直接调用 svc 指令的方式让 Linux 切换到内核态，执行完毕以后去直接拿返回结果即可  
（systcall 是 Linux 内核的入口，切换到内核态以后，无法被 Hook）

### 方案：

实现也很简单，提供两种方式：

*   1，直接调用 libc.so 里面的 syscall
*   2，内联汇编，将 Libc.so 里面的代码抠出来  
    本质区别不大

### 实现：

*   方式 1：  
    直接调用 syscall

```
std::string FileUtils::getFileText(char *path,int BuffSize) {

    char buffer[BuffSize];
    memset(buffer, 0, BuffSize);
    std::string str;
    //int fd = open(path, O_RDONLY);
    long fd = syscall(__NR_open, path, O_RDONLY);

    //失败 -1；成功：>0 读出的字节数  =0文件读完了
    while (syscall(__NR_read,fd, buffer, 1) != 0) {
        //LOG(ERROR) << "读取文件内容  " <<buffer;
        str.append(buffer);
    }
    syscall(__NR_close,fd);
    return str;
}


```

*   方式 2  
    通过内联汇编的方式调用 Svc

```
std::string FileUtils::getRawFileText(char *path,int BuffSize) {

    char buffer[BuffSize];
    memset(buffer, 0, BuffSize);
    std::string str;
    //int fd = open(path, O_RDONLY);
    long fd = raw_syscall(__NR_open, path, O_RDONLY);

    //失败 -1；成功：>0 读出的字节数  =0文件读完了
    while (read(fd, buffer, 1) != 0) {
        //LOG(ERROR) << "读取文件内容  " <<buffer;
        str.append(buffer);
    }
    syscall(__NR_close,fd);
    return str;
}


```

重点看一下 raw_syscall

内联汇编代码主要分 32 和 64

*   32 位实现

```
    .text
    .global raw_syscall
    .type raw_syscall,%function

raw_syscall:
        MOV             R12, SP
        STMFD           SP!, {R4-R7}
        MOV             R7, R0
        MOV             R0, R1
        MOV             R1, R2
        MOV             R2, R3
        LDMIA           R12, {R3-R6}
        SVC             0
        LDMFD           SP!, {R4-R7}
        mov             pc, lr


```

*   64 位实现

```
    .text
    .global raw_syscall
    .type raw_syscall,@function

raw_syscall:
        MOV             X8, X0
        MOV             X0, X1
        MOV             X1, X2
        MOV             X2, X3
        MOV             X3, X4
        MOV             X4, X5
        MOV             X5, X6
        SVC             0
        RET



```

cmake 里添加

```
enable_language(C ASM)


```

编译即可

比如获取网卡设备信息

```
LOG(ERROR) << "读取文件内容  " <<
            FileUtils::getFileText("/sys/class/net/p2p0/address",20);


```

### 这种方式一定是安全的么？

答案是否定的

目前主流的两种方法

*   可以通过 Ptrace 进行 svc 拦截，在使用前需要将 Ptrace 方法堵住  
    （因为 Ptrace 作为 Linux 的调试函数，是可以调试 svc 指令的，很多游戏辅助也都是这么搞得）  
    方法也很多，可以像一般壳子的方式提前占坑，或者读取调试状态，去判断是否被调试 都是不错的办法。
*   使用一些特殊的虚拟机，比如 GS 虚拟机之类的。  
    底层原理相当于在安卓上面实现一个新的安卓，这个很复杂，用到了 Google 开源的一些东西  
    比如 gVisor（Google 的 gVisor 则是 linux 上实现 linux），需要重新实现一遍安卓内核 ，所以就算 svc 指令也可以去拦截。

帖子根据个人经验梳理，如有不足，及时告知。