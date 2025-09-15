> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kZPYm_Ir-39cg7_Y7Ise3A)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LkKHLPtcJ06Ird3Cjgd69jp7USxAy64OoPrTRmRpiazoMzgU0EcGms2A/640?wx_fmt=png&from=appmsg#imgIndex=0)

声明
--

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

**本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号【K 哥爬虫】联系作者立即删除！**

逆向目标
----

*   目标：某当劳 APP
    
*   apk 版本：7.0.15.1
    
*   下载地址：`aHR0cHM6Ly93d3cuZG93bmt1YWkuY29tL2FuZHJvaWQvMTU0OTg1Lmh0bWw=`
    

逆向分析
----

直接注入 frida 代码，frida 命令如下：

```
frida -U -f com.mcdonalds.gma.cn -l .\3.js


```

结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kToFYjwHUzMSDlKwnv2chc1KE9eAOYm6g2yUic77aye0iaqtNO55B7iaGPQjceCMbDbDibxbcFuWkKjDag/640?wx_fmt=png&from=appmsg#imgIndex=1)

发现闪退，老样子，按照之前的思路，我们可以先 hook `dlopen` 方法，监控动态库的加载情况。

dlopen 原型函数：

```
void *dlopen(const char *filename, int flag);


```

<table><thead><tr><th><section>参数</section></th><th><section>说明</section></th></tr></thead><tbody><tr><td><code>filename</code></td><td><section>so 文件的路径，例如&nbsp;<code>"libfoo.so"</code>&nbsp;或完整路径&nbsp;<code>/data/app/.../libfoo.so</code>。传&nbsp;<code>NULL</code>&nbsp;表示获取主程序自身句柄。</section></td></tr><tr><td><code>flag</code></td><td><section>加载选项，常见值：•&nbsp;<code>RTLD_LAZY</code>：按需解析符号（调用时绑定）•&nbsp;<code>RTLD_NOW</code>：立即解析所有未定义符号 •&nbsp;<code>RTLD_GLOBAL</code>：符号导出，可被后续库使用 •&nbsp;<code>RTLD_LOCAL</code>：符号仅在本库内可见（默认）</section></td></tr></tbody></table>

hook `android_dlopen_ext` 代码如下：

```
function hook_dlopen() {
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            var path_ptr = args[0];
            var path = ptr(path_ptr).readCString();
            console.log("[android_dlopen_ext -> enter", path);
            if (args[0].readCString() != null && args[0].readCString().indexOf("libmsaoaidsec.so") >= 0) {
                // hook_call_constructors()
                hook_pth()
            }
        },
        onLeave: function (retval) {
            console.log("android_dlopen_ext -> leave")

        }
    });
}
hook_dlopen()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LonEL67nbywFATc3YA8YtFWtgCFQCMWpwzoB1RDfHBcTwvnKxSicKmHw/640?wx_fmt=png&from=appmsg#imgIndex=2)

可以看到，程序虽然在我们的 libmsaoaidsec.so 退出了，但是在这之前加载了一个 libDexHelper.so 文件，通过 MT 管理器查看可知，是某梆加固，而某梆加固都是 libDexHelper.so 进行 frida 检测的，所以我们应该先分析这个文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LKkmChdrmuA49AJqj2OibNq82icllrSoX7tkJbuMndua4r81BZ7xGhkcA/640?wx_fmt=png&from=appmsg#imgIndex=3)

so 文件分析
-------

知道在这个 so 文件加密之后，我们直接 hook `pthread_create`  看看创建了哪些线程，hook 代码如下：

```
function hook_pthread_create(){
    var pthread_create_addr = Module.findExportByName("libc.so", "pthread_create");
    console.log("pthread_create_addr: ", pthread_create_addr);
    Interceptor.attach(pthread_create_addr,{
        onEnter:function(args){
            console.log(args[2], Process.findModuleByAddress(args[2]).name);
        },onLeave:function(retval){
        }
    });
}
hook_pthread_create();

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3Ly0x1444ic2RetPNKNicia8NRZVOlI2UA5qQ5Fvg74KYlRWWFZKIibsia9hg/640?wx_fmt=png&from=appmsg#imgIndex=4)

发现并没有 libDexHelper.so 相关的线程，这是什么原因呢？遇事不决问 ai，下面是 GPT 给出的部分答案：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LMlLKeBSyZFVIIU9yDD5lOmXNpvmdu2LLcj9JCBIlHSPvwXCSRCnP5Q/640?wx_fmt=png&from=appmsg#imgIndex=5)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3Lhb9vdRs1LiaG0MBM5EEcQ0BuSInWLtXxJYMDCb8juibDFAd0Dm6t4CUA/640?wx_fmt=png&from=appmsg#imgIndex=6)

GPT 给出了重要结论，`pthread_create` 最终会调用 `clone` 方法。

`clone` 是 Linux/Android 系统的一个 **底层系统调用**，用于创建 **线程或进程**，它比 `fork` 更灵活，是 `pthread_create` 的底层实现基础。

clone 函数原型如下：

```
int clone(int (*fn)(void *), void *child_stack, int flags, void *arg, 
          ... /* pid_t *ptid, void *tls, pid_t *ctid */);


```

<table><thead><tr><th><section>参数</section></th><th><section>说明</section></th></tr></thead><tbody><tr><td><code>fn</code></td><td><section>子线程 / 进程起始函数</section></td></tr><tr><td><code>child_stack</code></td><td><section>子线程栈顶地址</section></td></tr><tr><td><code>flags</code></td><td><section>控制资源共享与行为</section></td></tr><tr><td><code>arg</code></td><td><section>传给&nbsp;<code>fn</code>&nbsp;的参数</section></td></tr><tr><td><code>ptid</code></td><td><section>父线程写入子线程 PID 的地址</section></td></tr><tr><td><code>tls</code></td><td><section>子线程 TLS 基址</section></td></tr><tr><td><code>ctid</code></td><td><section>子线程写入自己 PID 的地址</section></td></tr></tbody></table>

那我们直接调用 clone 函数试看看，hook  clone 代码如下：

```
var clone = Module.findExportByName(null, 'clone');

Interceptor.attach(clone, {
    onEnter: function(args) {
        // 获取线程函数地址
        var func = args[0];
        // 获取线程函数所在模块
        varmodule = Process.findModuleByAddress(func);
        if (module) {
            console.log("Thread function is located in module: " + module.name);
        }

        // 打印调用栈
        console.log("Backtrace:");
        console.log(Thread.backtrace(this.context, Backtracer.ACCURATE)
            .map(DebugSymbol.fromAddress)
            .join('\n'));
    },
    onLeave: function(retval) {
        // 可在这里打印返回值或做后续处理
    }
});

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LakCCGtSO7V8JiaLsqRvT55aOLWLdYp69icK2tHDQA7icDT21zNRWKEckA/640?wx_fmt=png&from=appmsg#imgIndex=7)

发现多次调用同一个线程创建，通过下面命令，把 lib.so 文件拷贝到电脑：

```
adb pull /system/lib64/libc.so ./libc64.so 


```

ida 分析
------

我们用 ida 打开 libc.so 文件，搜索 `pthread_create` 查看它的地址：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LtN6GD0pl61qThD4m25Mfxfib3KzvydqFsLWDjRSeicQLzz7bsxBwmR5Q/640?wx_fmt=png&from=appmsg#imgIndex=8)

我们主要关注上面 a3 的值，按住 tab 键找到 `pthread_create` 的地址：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3Loxjuna577ZrjVC3ogg6SBLDnRqatBgZDXkJ6h61yjN5iaPs41r7zDiaA/640?wx_fmt=png&from=appmsg#imgIndex=9)

```
0x7278e88aa8 libc.so!pthread_create+0x290


```

根据上面的输出加上偏移得到 clone 函数的最终地址为 0xAFAA8：

```
0x00000000000AF818 + 0x290 = 0x00000000000AFAA8


```

按 g 跳转到该地址，如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LDA87Dniaia179RibtP3Ho1z5Gq1DVdxOlT4BZqWqfD0dXoHCWXAMIqvfA/640?wx_fmt=png&from=appmsg#imgIndex=10)

```
clone(__pthread_start, v19, 4001536, v31, v31 + 16, v23 + 8, v31 + 16);


```

<table><thead><tr><th><section>参数</section></th><th><section>说明</section></th></tr></thead><tbody><tr><td><code>__pthread_start</code></td><td><section>线程函数入口（pthread 内部包装&nbsp;<code>start_routine</code>）</section></td></tr><tr><td><code>v19</code></td><td><section>新线程栈顶地址</section></td></tr><tr><td><code>4001536</code></td><td><section>clone flags（如 CLONE_VM)</section></td></tr><tr><td><code>v31</code></td><td><section>入口函数参数（通常封装了 start_routine + arg）</section></td></tr><tr><td><code>v31 + 16</code></td><td><section>父线程写入子线程 PID 的地址（ptid）</section></td></tr><tr><td><code>v23 + 8</code></td><td><section>新线程 TLS 基址</section></td></tr><tr><td><code>v31 + 16</code></td><td><section>子线程写自己 PID 的地址（ctid）</section></td></tr></tbody></table>

在 pthread 内部，线程函数会存储在线程控制块中：

```
*(_QWORD *)(v31 + 96) = a3;  // 将用户线程函数写入线程控制块


```

通过读取 `v31 + 96` 的地址，我们可以获取实际执行的线程函数，hook 代码如下：

```
var clone = Module.findExportByName('libc.so', 'clone');
Interceptor.attach(clone, {
    onEnter: function(args) {
        // 只有当 args[3] 不为 NULL 时，才说明上层确实把 “线程控制块指针” 传进来了
        if(args[3] != 0){
            // 真正的用户线程函数地址
            var addr = args[3].add(96).readPointer()
            // 根据线程函数地址 addr，找它属于哪个模块
            var so_name = Process.findModuleByAddress(addr).name;
            // 获取该 so 在进程里的基址
            var so_base = Module.getBaseAddress(so_name);
            // 获取相对于 so_base 的偏移
            var offset = (addr - so_base);
            console.log("===============>", so_name, addr,offset, offset.toString(16));
        }
    },
    onLeave: function(retval) {

    }
});


```

结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LOW3wxjYPY42hcADVMOZ2FKGehojBXl0UiaeBU9nZ0hgvyNrgq5xS4qQ/640?wx_fmt=png&from=appmsg#imgIndex=11)

可以看到成功输出了该 so 文件的线程函数，接下来，我们尝试着先 nop 掉这几个函数，看能否过检测，hook 代码如下：

```
function nopFunc(parg2) {
    Memory.protect(parg2, 4, 'rwx');  // 修改该地址的权限为可读可写
    var writer = new Arm64Writer(parg2);
    writer.putRet();   // 直接跳到 ret 返回地方 ，不反回值
    writer.flush();   // 写入操作刷新到目标内存，使得写入的指令生效。  从缓存中写道内存
    writer.dispose();  // 释放 Arm64Writer 使用的资源。
    console.log("nop " + parg2 + " success");
}

function hook_dlopen(so_name) {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                console.log("[android_dlopen_ext -> enter", path);
                if (path.indexOf(so_name) !== -1) {
                    this.match = true
                }
            }
        },
        onLeave: function (retval) {
            if (this.match) {
                console.log(so_name, "加载成功");
                var base = Module.findBaseAddress("libDexHelper.so")
                nopFunc(base.add(308204));
                nopFunc(base.add(362896));
                nopFunc(base.add(332536));
                nopFunc(base.add(366304));
                nopFunc(base.add(385348));
            }
            console.log("android_dlopen_ext -> leave")
        }
    });
}
hook_dlopen("libDexHelper.so")


```

结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3LsUEyqMWwwPK30lFGNcwcyzqsZrG7qw6g2ZrpuPnmakIXNTzSgThldA/640?wx_fmt=png&from=appmsg#imgIndex=12)

发现我们 nop 掉线程后，并没有报错，那证明我们 nop 掉没有问题，但是卡在了最开始的 libmsaoaidsec.so 文件里，对于这个文件，我们同样直接 nop 掉里面的线程即可：

```
function hook_pth() {
    var pth_create = Module.findExportByName("libc.so", "pthread_create");
    console.log("[pth_create]", pth_create);
    Interceptor.attach(pth_create, {
        onEnter: function (args) {
            varmodule = Process.findModuleByAddress(args[2]);
            if (module != null) {
                console.log("开启线程-->", module.name, args[2].sub(module.base));
                if (module.name.indexOf("libmsaoaidsec.so") != -1) {
                    Interceptor.replace(module.base.add(0x175f8), new NativeCallback(function () {
                        console.log("替换成功")
                    }, "void", ["void"]))
                    Interceptor.replace(module.base.add(0x16d30), new NativeCallback(function () {
                        console.log("替换成功")
                    }, "void", ["void"]))
                }
            }

        },
        onLeave: function (retval) {
        }
    });
}


```

最终也是成功绕过了检测，结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqxTsJw3JskTNciaHFhQ3b3Ll25ia968n1oWqIslhwTScRxxpAg4khOWhpmTqy32S4msvJVKYhicSBqg/640?wx_fmt=png&from=appmsg#imgIndex=13)

至此，该 app 的 frida 检测分析流程就结束了。

相关 hook 脚本，会分享到知识星球当中，需要的小伙伴自取，仅供学习交流。

 ![](https://mmbiz.qpic.cn/sz_mmbiz_gif/iabtD4jabia4KDdF6jxLibSq5ssaiaicicKHf2VWdrkFqrsRuDF7CiaKMxAeua0WeLPFmOIQkgcCt66o7L2uOl1wRVuVw/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=14)

**（ 先别划走 点个关注 Orz ）**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqk3p9iaszC2ibDWOXPQ3e0aCy3zsOLCDOV6ZbGbedyRNqfsqWUODEFC5B4nnbhAiaKmslJL07ruia4og/640?wx_fmt=png&from=appmsg#imgIndex=15)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqRIkUKs0D0Yt8cHdIumAAg9cLcxz0cztZiaWDyxEDjTdbKjruhZNxHJG4IyORoBmMZsUeYqHjlVibA/640?wx_fmt=png&from=appmsg#imgIndex=16)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTrTeJ8a4m9ic7JiarhInKgeBuqibVich0SOoiaDWcHfHCI6ol1ach9cP1vZaJyXdMTzGtoeAgIMEQib9zoA/640?wx_fmt=png&from=appmsg#imgIndex=17)

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTrTeJ8a4m9ic7JiarhInKgeBuTxSMfctCrwjIwRy5LyAcB0MI70LbUMytydrKre4gibW7icN76X0IZicCQ/640?wx_fmt=png&from=appmsg#imgIndex=18)](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485137&idx=1&sn=7f5a391d269f96be1eb8cf72d11d3fe7&scene=21#wechat_redirect)

**深耕海内外代理 IP 精研企业级 SaaS 产品**

**安全、稳定、快速、合规的代理服务器**

**海外代理 IP 推荐**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTrTeJ8a4m9ic7JiarhInKgeBu3yX37eYktKGPhGEZNRoy18ms6qut9XAU7US9Nu0ZQ8MWkNRMP88fMA/640?wx_fmt=png&from=appmsg#imgIndex=19)

**- 专注不限量海外代理 IP 服务！**

**- 服务超过半数人工智能、大数据企业！**

**- 123proxy.cn -- 海外大数据采集、爬虫应用的代理 IP 选择！**

  

![](https://mmbiz.qpic.cn/mmbiz_png/7VAgNKQgCMFM8ia5BA9MLZhlCnRr8Er4gR4Rjr7WBmby6jKvlqpH7jZITFBYBIYbibfOgHRCF5obiaJn6yzC321qw/640?wx_fmt=png#imgIndex=20)

**点个****推荐♡****求求你啦**

![](https://mmbiz.qpic.cn/mmbiz_png/NtgFk2rGpiaOPxvr7Ls916UDdGAibFN8ObxF6VKc8qCT18luCwKTUgHicBiaMYJE9SIdicQHL7ouCt8xk7tMtsxKayA/640?wx_fmt=png#imgIndex=21)