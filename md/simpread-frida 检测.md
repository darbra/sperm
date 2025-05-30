> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oz9Xo-ByH0P5za-S9IXWcg)

1、创建虚假 pthread_create

2、memfdfrida:rpc  FridaScriptEngine   GLib-GIO   GDBusProxy   GumScript（内存动态扫描 maps）

3、pretty method

4、svc  自定义的 动态生成的  汇编

5、ELF magic  抹除文件头

6、tracepid  

7、inlinehook 函数 hook 之前和之后的机器码 一般是前 16 位

检测 inlinehook 正常是 mov 常数到寄存器 然后是 br 寄存器指令 检查第二条指令高十六位是不是 0xd61f

8、断点指令  00 00 20 d4    brk #0  检测断点前后发生变化的字节

9、ptrace 禁止 frida attach

ptrace 占坑  ptrace(0, 0 ,0 ,0);

开启一个子进程附加父进程

守护进程 子进程附加父进程 目的是不让别人附加

10、进程名检测，遍历运行的进程列表，检测 frida-server 是否运行

11、端口检测，检测 frida-server 默认端口 27042 是否开放

12、D-Bus 协议通信

Frida 使用 D-Bus 协议通信，可以遍历 / proc/net/tcp 文件，或者直接从 0-65535

向每个开放的端口发送 D-Bus 认证消息，哪个端口回复了 REJECT，就是 frida-server

13、扫描 maps 文件

maps 文件用于显示当前 app 中加载的依赖库 Frida 在运行时会先确定路径下是否有 re.frida.server 文件夹

若没有则创建该文件夹并存放 frida-agent.so 等文件，该 so 会出现在 maps 文件中

14、通过 readlink 查看 / proc/self/fd、/proc/self/task/pid/fd 下所有打开的文件，检测是否有 Frida 相关文件

15、常见用于检测的系统函数

strstr、strcmp、open、read、fread、readlink、fopen、fgets

扫描内存中是否有 Frida 库特征出现，例如字符串 LIBFRIDA

16、扫描 task 目录

扫描目录下所有 / task/pid/status 中的 Name 字段

寻找是否存在 frida 注入的特征

具体线程名为 gmain、gdbus、gum-js-loop、pool-frida 等

17、task/xxx/cmdline 检测进程名，防重打包

18、status 检测进程是否被附加

19、task/xxx/stat 检测进程是否被附加

20、fd/xxx 检测 app 是否打开的 Frida 相关文件

21、RWXP 权限  根据执行权限不同 在 hook 时

22、crc 检测 对 text 段进行 hash

23、somainv  没有判断是否为空

24、通过 dlsym 查找线程创建的符号 实现调用 pthread

25、libart.so 的检测 frida hook 之后发生改变较多

26、libc.so 的检测  一些函数入口处发生改变