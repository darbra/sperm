> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-271815.htm)

> [原创] 分享一个 Android 通用 svc 跟踪以及 hook 方案——Frida-Seccomp

[](#一个android通用svc跟踪以及hook方案——frida-seccomp)一个 Android 通用 svc 跟踪以及 hook 方案——Frida-Seccomp
=========================================================================================

[](#效果：)效果：
===========

对 openat 进行跟踪
-------------

![](https://bbs.pediy.com/upload/attach/202203/903162_55EERQXHHTXD4FW.jpg)

对 recvfrom 进行跟踪
---------------

![](https://bbs.pediy.com/upload/attach/202203/903162_AE84XGKCDB8RBY6.jpg)  
**在这里感谢珍惜大佬介绍的 seccomp 机制，推荐一波珍惜大佬的课程能学到很多有趣的骚操作。**

什么是 seccomp
===========

[seccomp 沙箱机制介绍文章](http://pollux.cc/2019/09/22/seccomp%E6%B2%99%E7%AE%B1%E6%9C%BA%E5%88%B6%20&%202019ByteCTF%20VIP/#%E9%80%9A%E8%BF%87%E4%BD%BF%E7%94%A8%E8%AF%A5%E5%BA%93%E7%9A%84%E5%87%BD%E6%95%B0%E5%AE%9E%E7%8E%B0%E7%A6%81%E7%94%A8execve%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8)  
seccomp 是 Linux 内核提供的一种应用程序沙箱机制，主要通过限制进程的系统调用来完成部分沙箱隔离功能。seccomp-bpf 是 seccomp 的一个扩展，它可以通过配置来允许应用程序调用其他的系统调用。

如何和 frida 结合
============

基本原理
----

```
这是一个bpf规则：
struct sock_filter filter[] = {
        BPF_STMT(BPF_LD + BPF_W + BPF_ABS,
                 (offsetof(struct seccomp_data, nr))),
        BPF_JUMP(BPF_JMP + BPF_JEQ + BPF_K, nr, 0, 1),
        BPF_STMT(BPF_RET + BPF_K, SECCOMP_RET_TRAP),
        BPF_STMT(BPF_RET + BPF_K, SECCOMP_RET_ALLOW),
};

```

seccomp 的具体用法可以参考「**什么是 seccomp**」中的 seccomp 介绍文章。当返回规则设置为「SECCOMP_RET_TRAP」，目标系统调用时 seccomp 会产生一个 SIGSYS 系统信号并软中断，这时就可以通过捕获这个 SIGSYS 信号获得 svc 调用和打印具体参数。

如何脚本化安装 seccomp 规则呢
-------------------

这里使用 Frida 的 API「CModule」，CModule 提供强大的动态编译功能可以让你在 JS 中写 C，  
**frida 文档中的示例**

```
const cm = new CModule(`
#include void hello(void) {
  printf("Hello World from CModule\\n");
}
`);
const hello = new NativeFunction(cm.hello, 'void', []);
hello(); 
```

如何捕获异常
------

使用 Frida 的 API「**Process.setExceptionHandler**」即可捕获异常并在自己写的回调中进行数据处理。  
数据处理的逻辑解释写在注释里啦。

```
// 异常处理
    Process.setExceptionHandler(function (details) {
        const current_off = details.context.pc - 4;
        // 判断是否是seccomp导致的异常 读取opcode 010000d4 == svc 0
        if (details.message == "system error" && details.type == "system" && hex(ptr(current_off).readByteArray(4)) == "010000d4") {
            // 上锁避免多线程问题
            lock(syscall_thread_ptr)
            // 获取x8寄存器中的调用号
            const nr = details.context.x8.toString(10);
            let loginfo = "\n=================="
            loginfo += `\nSVC[${syscalls[nr][1]}|${nr}] ==> PC:${addrToString(current_off)} P${Process.id}-T${Process.getCurrentThreadId()}`
            // 构造线程syscall调用参数
            const args = Memory.alloc(7 * 8)
            args.writePointer(details.context.x8)
            let args_reg_arr = {}
            for (let index = 0; index < 6; index++) {
                eval(`args.add(8 * (index + 1)).writePointer(details.context.x${index})`)
                eval(`args_reg_arr["arg${index}"] = details.context.x${index}`)
            }
            // 获取手动堆栈信息
            loginfo += "\n" + stacktrace(ptr(current_off), details.context.fp, details.context.sp).map(addrToString).join('\n')
            // 打印传参
            loginfo += "\nargs = " + JSON.stringify(args_reg_arr)
            // 调用线程syscall 赋值x0寄存器
            details.context.x0 = call_task(syscall_thread_ptr, args, 0)
            loginfo += "\nret = " + details.context.x0.toString()
            // 打印信息
            call_thread_log(loginfo)
            // 解锁
            unlock(syscall_thread_ptr)
            return true;
        }
        return false;
    })

```

还有什么坑
-----

### 1.syscall 调用 resume

#### 问题描述

根据 Frida 文档介绍「**setExceptionHandler**」捕获异常后只需要让回调返回 true 就会 resume 原本的线程，但是其只是跳过了 svc 指令继续执行，实际上并不会执行 svc，这时候如果不执行 syscall 轻则导致 APP 数据异常，重则导致 APP 直接崩溃。所以在异常的回调中需要手动调用了 syscall 并赋值给 x0。  
但这时候会发生个新的问题，因为在主线程开启 seccomp 后，主线程和其后创建出来的线程都会被 seccomp 规则约束，在异常处理函数直接调用 syscall 同样会被 seccomp 约束再次抛出异常，就形成了” 死锁 “了。

#### 如何解决

可以注意到上面 “死锁” 部分描述，那我们在主线程被约束前，提前创建一个线程，这个线程就是不被约束的，同时受到线程池启发，我们让这个 syscall 线程循环接受任务，就能完成在一个没有约束的线程里进行 syscall 调用。

### 2. 堆栈回溯

#### 问题描述

直接使用 Frida 的 API「**Thread.backtrace**」很容易导致崩溃，原因可能是 seccomp 规则或者「prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0)」导致的权限收紧和 Frida 实现堆栈回溯功能冲突。

#### 如何解决

手动实现堆栈回溯，原理是 Arm64 中每个函数都会在函数头部位置对 x29、x30 寄存器存入栈中，所以可以对 x29 不断读取往上回溯，最后得到完整的堆栈信息。  
**实现**

```
function stacktrace(pc, fp, sp) {
    let n = 0, stack_arr = [], fp_c = fp;
    stack_arr[n++] = pc;
    const mem_region = call_thread_read_maps(sp);
    while (n < MAX_STACK_TRACE_DEPTH) {
        if (parseInt(fp_c.toString()) < parseInt(sp.toString()) || fp_c < mem_region.start || fp_c > mem_region.end) {
            break
        }
        let next_fp = fp_c.readPointer()
        let lr = fp_c.add(8).readPointer()
        fp_c = next_fp
        stack_arr[n++] = lr
    }
    return stack_arr;
}

```

### 3.「**Process.findModuleByAddress**」「**Process.enumerateModules**」类的 API 导致崩溃或找不到 Module 信息

#### 问题描述

疑似同坑 2 的原因

#### 如何解决

在 CModule 中手动实现了通过地址查 soinfo 信息「base, size, soname」等

### 4.write 调用约束下调用 Frida 的 API「send」崩溃

#### 问题描述

同坑 2 的原因

#### 如何解决

直接改用 syscall 线程使用「**__android_print_log**」打印信息

还可以实现什么
=======

在调用线程 syscall 前后可以更改传参、返回值、地址等更改，达到 HOOK 的效果

GITHUB
======

**求 star**  
[https://github.com/Abbbbbi/Frida-Seccomp](https://github.com/Abbbbbi/Frida-Seccomp)

如何使用
====

```
pip3 install frida
python3 multi_frida_seccomp.py

```

log 信息可以在 logcat 过滤 “seccomp” 查看  
同时也自动保存到了「包名_pid_时间戳」文件夹内（支持多进程）  
![](https://bbs.pediy.com/upload/attach/202203/903162_DSYWMSBGQMMVBJ8.png)

[【公告】看雪团队招聘安全工程师，将兴趣和工作融合在一起！看雪 20 年安全圈的口碑，助你快速成长！](https://job.kanxue.com/position-read-1104.htm)

[#基础理论](forum-161-1-117.htm) [#NDK 分析](forum-161-1-119.htm) [#HOOK 注入](forum-161-1-125.htm) [#系统相关](forum-161-1-126.htm) [#工具脚本](forum-161-1-128.htm)