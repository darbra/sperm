> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.fangg3.com](https://blog.fangg3.com/article/BYDReverse)

> 比亚迪汽车 App, Android 端逆向分析

[](#fa7059b865194007b164f2ac0d42e6a0 "BYD分析
反调试")BYD 分析 反调试
-----------------------------------------------------------

论坛关于反反调试的帖子很多，但是总觉得少了些什么。这里从我自己的角度，一步一步的抽丝剥茧，懂得原理才能走的更远，愿自己不要再做脚本 boy。文中如有暇疵遗漏，欢迎各位斧正。

*   样本：BYD

*   加固：某梆企业版

*   _**本文只是分享自己的思路，对该样本进行分析，请勿对此 App 进行攻击。**_

#### [](#1c627bb1269d40109349f88d20365169 "ptrace防附加")ptrace 防附加

1.  ptrace 占坑

原理：a 进程同一时间只能被 b 进程 ptrace 调试，如果 c 进程对 a 进程 ptrace 调试，则会失败。

运行 App 后，通过`ps -A |grep byd` 可以看到有两个进程：

![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb61401d9-0ef6-41e4-a650-6a50fa0896d4%2FUntitled.png?table=block&id=3018c143-2f69-4021-80ab-d2fdcdc68467&t=3018c143-2f69-4021-80ab-d2fdcdc68467&width=1500&cache=v2)

pid 分别为 29976、30008。其中 29976 为主进程，其父进程为 15576（zygote）。30008 为子进程，父进程为 29976。

继续查看 / proc/29976/status，如下

尝试附加进程：

> a. 得出结论，主进程被子进程 Ptrace 占坑调试，导致 frida 附加不上。 b. 附加子进程时，过段时间闪退，猜测存在 frida 检测。

2.  如何实现 ptrace 占坑

首先，从 ptrace 函数出发，其主要用法如下：

**enum __ptrace_request**

*   PTRACE_TRACEME ：**表示本进程将被其父进程跟踪**，此时剩下的 pid、addr、data 参数都没有实际意义可以全部为 0

这个选项只能用在被调试的进程中，**也是被调试的进程唯一能用的 request 选项**，其他的都只能用父进程调试器使用

*   PTRACE_ATTACH：attach 到一个指定的进程，使其成为当前进程跟踪的子进程，而子进程的行为等同于它进行了一次 PTRACE_TRACEME 操作，可想而知，gdb 的 attach 命令使用这个参数选项实现的

*   PTRACE_CONT：继续运行之前停止的子进程，也可以向子进程发送指定的信号，这个其实就相当于 gdb 中的 continue 命令

由此可拓展出反调试思路：

*   PTRACE_TRACEME 父进程占坑

*   检测当前 tracerPid

循环读取 / proc/self/status 中 tracerPid 的值

*   子线程附加父线程（也就是本 App 的反 ptrace 调试方法）

7.  反 ptrace 占坑

假设，app 的占坑逻辑在某个 So 的_init 中，app 加载该 So 时，已完成了占坑查找。使用 frida attach 注入 app 时，必然会失败。在这个前提下，我们有 3 种方法反占坑：

2.  patch 这个 so，让 ptrace 逻辑不执行。
3.  hook ptrace(PTRACE_TRACEME,0,0,0) 使其返回 0, 让它认为已经占坑成功了。
4.  在尽可能早的时机完成注入操作，例如 frida spawn 注入的时机是 root 启动 app 线程后，app 的 So 加载前。

可以发现，方法 3 并没有更改 So 的逻辑。如果调试进程 Tracer 修改了被调试进程 Tracee 的内存等，方法 1 和方法 2 可能会导致 App 运行异常。

> 总结：使用 frida spawn 即可

#### [](#d75180df4fe34bb38d916398b2c9f19c "frida检测")frida 检测

1.  了解 pthread_create 函数

pthread_create，作用是创建一个线程，可以指定新线程的运行函数地址，以及传递参数。

2.  从开发的角度看 frida 检测

**在我的理解中**，作为开发而言，大概率不会在主线程进行检测，否则有可能会使程序阻塞卡顿。所以，创建一个线程，进行检测会比较好。由于 frida attach 后，经过了几秒后 App 才被 kill，可以推断出使用了定时器，循环检测 frida。

接下来验证我的猜测：

直接启动目标 app，通过`ps -T -p $(主进程pid)`，查看进程中的线程，我们得到以下结果：

这样看起来似乎很复杂，我们知道 linux 中在没有指定线程名称时，线程的名称与进程的名称一致。

我们过滤一下包名，得到下面结果：

这样就可以区分出, 哪些是 App 通过 pthrea_create 自己创建的线程了。

继续往下分析，通过 WCHAN 观察线程状态，其中`hrtimer_nanosleep` 尤其显眼，说明该线程正在调用 msleep 之类的函数, 也就验证了我之前的假设。

3.  绕过 frida 检测

方法 1: patch pthread_create 函数

方法 2: patch 所有调用 pthread_create 函数的 caller

例如：获取 caller 偏移地址为 0xa765c

000a7658 为函数 pthread_create 调用，成功时返回值 0 放在寄存器 X0 中，000a765c 中 W0 则是取寄存器 X0 的低 32 位。

patch `000a7658 80003fd6 blr x4` → `000a7658 80003fd6 ldr x0 #0`

frida 实现如下：

**验证结果：**

**frida 可以正常 spawn，不闪退。**

**查看线程状态，frida 检测线程未创建，不影响执行流程。**

### [](#0ffd2b2b00ad4695b559b82ca301b5eb "todo")todo

虚拟框架加载 So 脱壳

So 分析

checkCode 算法分析

* * *

参考：

![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb61401d9-0ef6-41e4-a650-6a50fa0896d4%2FUntitled.png?table=block&id=2b6d27c9-ddeb-4076-a5dc-bebc90d4ea8f&t=2b6d27c9-ddeb-4076-a5dc-bebc90d4ea8f&width=1500&cache=v2)