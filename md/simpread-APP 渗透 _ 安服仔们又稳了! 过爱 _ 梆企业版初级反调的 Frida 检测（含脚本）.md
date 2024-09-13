> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/34c5JVJzSCEfqlPanV1FtA)

  

此文章只为学习而生，请勿干违法违禁之事，本公众号只在技术的学习上做以分享，通过现实的几个案例来实战锻炼水平，有个大佬公众号私聊我讨论了一下看了看，遇到所以试过，所有行为与本公众号无关。

  

  

sdfd

01

日常截图  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOUfsGPk57aDcQmmYsC4QLNpleBSbDRgsUfNh3fGsFlevaqa1y8tnlKw/640?wx_fmt=png&from=appmsg&random=0.8687113455639703&random=0.3533966643309001&random=0.4829709958050883&random=0.5619768792020099&random=0.6023584121035848&random=0.49424287449791704&random=0.8395803087308455&random=0.9101265662013172&random=0.9671217704920141&random=0.3457769439550411)

漏洞盒子众测的某金融 APP 用了爱马氏加密啊，检测 Frida 难受啊。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOKcornhVOpJ4tI5noNES10tJTziclWwlVQybknVtvTGsoy8SyRtsYlIA/640?wx_fmt=png&from=appmsg&random=0.5701184250571996&random=0.4771847360021966&random=0.8974004854520867&random=0.10119547000615214&random=0.4746543107931396&random=0.10208274560181407&random=0.2608987225152777&random=0.5601461251930107&random=0.9444352220239787&random=0.7941258388574042)

然后又测试了一下梆某人的加密啊，又检测 Frida 难受啊，还不能通用，烦死了。

02

反调思路来源  

  

1. 绕过 bilibili frida 反调试：https://bbs.kanxue.com/thread-277034.htm

2. frida 常用检测点及其原理 - 一把梭方案：https://bbs.kanxue.com/thread-278145.htm

3. 绕过最新版 bilibili app 反 frida 机制：https://bbs.kanxue.com/thread-281584.htm

4. 某车联网 APPx 梆反调试分析：https://bbs.kanxue.com/thread-277692.htm

5. 记一次爱加密反调试分析及绕过思路：https://bbs.kanxue.com/thread-259619.htm 

03

起手式  

  

 第一个肯定先看的金融 APP 的爱马仕加密。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOtIZqX4cXf6ib5gWiaB8zbE70YlB0wpcDFSHrqEDMdl4uvCMHApT4ov3A/640?wx_fmt=png&from=appmsg&random=0.524704229226234&random=0.4964949978171229&random=0.6589812389653922&random=0.06796353578573311&random=0.10224801955400453&random=0.4063259435924387&random=0.19288876242876607&random=0.8440596218479293)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOjyVaUvXCS3o38c0mq82uXkVFNldlXXJXVO7qweDu9FtoLJKzANsefA/640?wx_fmt=png&from=appmsg&random=0.7155358589996546&random=0.0755839895663406&random=0.9890761580400038&random=0.2305407748802637&random=0.3973604726487079&random=0.5036266332255788&random=0.8834211986980858&random=0.7121806913327118)

在启动后就 Process terminated。

01

Hook android_dlopen_ext

大佬们说检测 Frida 一般都在 Native 层实现，会用线程轮询的方式来检测，那 hook 线程创建函数 android_dlopen_ext，来看看由哪个 so 实现或者在加载到哪个 so 时候触发反调试进程终止。

```
// 检测SO代码然后将hook到的so文件进行过检测
function hook_dlopen() {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
        {
            onEnter: function (args) {
                var pathptr = args[0];
                if (pathptr !== undefined && pathptr != null) {
                    var path = ptr(pathptr).readCString();
                    console.log("load " + path);
                }
            }
        }
    );
}
// 调用查看检测的
hook_dlopen()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfO1iaOxyibUV4bupR7tudt371PicBJU2BdgjlJ9icicaqWicOicXG2ExVqarHug/640?wx_fmt=png&from=appmsg&random=0.06773537261346885&random=0.514918069514178&random=0.5884421750789015&random=0.24994710106909812&random=0.2994589378367165&random=0.7062335496541257&random=0.48307324477301683)

发现了 libexecmain.so 执行后进程退出。  

我们可以猜测 libexecmain.so 中创建了一个线程检测到了 Frida 使其退出。

02

HOOK pthread_create

 现在这步是确认是否由 libexecmain 创建的检测线程，用以下脚本：

```
function hook_pthread_create(){
    Interceptor.attach(Module.findExportByName(null, "pthread_create"),
        {
            onEnter: function (args) {
                var module = Process.findModuleByAddress(ptr(this.returnAddress))
                if (module != null) {
                    console.log("[pthread_create] called from", module.name)
                }
                else {
                    console.log("[pthread_create] called from", ptr(this.returnAddress))
                }
            },
        }
)
}
hook_pthread_create()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfO5xiaHUELZRmOcIjjxqQhjQiae3MewegyGZLpqJsuJ8ZGFlQxHAvjBRLw/640?wx_fmt=png&from=appmsg&random=0.2745442654359409&random=0.2754837177468674&random=0.3453818569819149&random=0.6370844167311944&random=0.3071596735652795&random=0.32091923051561877)

发现只有 libexec.so 一直在创建进程，而 libexecmain.so 从未出现，而 libexec.so 是上一个调用的，可能就是由 libexec.so 创建的检测线程。

03

Patch pthread_create  

 知道 App 通过哪些 SO 进行 pthrea_create 自己创建的线程了，我们可以尝试进行 Patch HOOK 过掉看看能不能过掉检测。

```
// 1. patch  pthread_create函数
function patchPthreadCreate(){
    let pthread_create = Module.findExportByName(null, "pthread_create")
        let org_pthread_create = new NativeFunction(pthread_create, "int", ["pointer", "pointer", "pointer", "pointer"])
        let my_pthread_create = new NativeCallback(function (a, b, c, d) {
            let m = Process.getModuleByName("libexec.so");
            let base = m.base
            console.log(Process.getModuleByAddress(c).name)
            if (Process.getModuleByAddress(c).name == m.name) {
                console.log("pthread_create")
                return 0;
            }
            return org_pthread_create(a, b, c, d)
        }, "int", ["pointer", "pointer", "pointer", "pointer"])
        Interceptor.replace(pthread_create, my_pthread_create)
}
patchPthreadCreate()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOXcPL2V4fo68ChKIVFro6GBzib1qtErxwdwFEnUicCGcWicYk5ITnZSZuQ/640?wx_fmt=png&from=appmsg&random=0.5323243552779509&random=0.18056944463664437&random=0.8189343503244693&random=0.014722463524038831&random=0.0663866072205912)

发现应用跑无响应了，但是经过了很长的时间没有退出说明有点戏，试试另一个方法。

04

Patch 所有调用 pthread_create 函数的 caller

```
function patchPthreadCreateCaller(){
    let pthread_create = Module.findExportByName(null, "pthread_create")
    Interceptor.attach(pthread_create, {
        onEnter: function (args) {
            this.isHook = false
            let m = Process.getModuleByName("libexec.so");
            let base = m.base
            console.log(Process.getModuleByAddress(args[2]).name)
            if (Process.getModuleByAddress(args[2]).name == m.name) {
                              //start_rtn地址  start_rtn偏移       caller地址               
                console.log(args[2], args[2].sub(base), this.context.lr.sub(m.base))
            }
            let addr = args[2].sub(base)
            console.log(addr)
        }
    })
}
patchPthreadCreateCaller()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOzyFeA5mDibB0Od4rRy5OXqiaD8Lp2GdYSTEB0wBvDdlia8XGiaia8Uz38lg/640?wx_fmt=png&from=appmsg&random=0.7840600902767163&random=0.19266441644883003&random=0.5241827719549752&random=0.0418161491054625)

单独列出来的 addr 就是创建线程的偏移量，所以再改进一下。

```
function patchPthreadCreateCaller(){
    let pthread_create = Module.findExportByName(null, "pthread_create")
    Interceptor.attach(pthread_create, {
        onEnter: function (args) {
            this.isHook = false
            let m = Process.getModuleByName("libexec.so");
            let base = m.base
            console.log(Process.getModuleByAddress(args[2]).name)
            if (Process.getModuleByAddress(args[2]).name == m.name) {
                              //start_rtn地址  start_rtn偏移       caller地址               
                console.log(args[2], args[2].sub(base), this.context.lr.sub(m.base))
                if (args[2] >= base && args[2] < base.add(m.size)) {
                    Memory.patchCode(args[2], 4, code => {
                        console.log("Patching thread function at " + args[2]);
                        const cw = new Arm64Writer(code, {pc: args[2]});
                        cw.putLdrRegU64('x0', 0);
                        cw.flush();
                      });
                }
            }
        }
    })  
}
patchPthreadCreateCaller()

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfONY4y5sznDEn4JIw6SZZpgbV8DYSRXqibsibUH1NIGlBVY3aUGtgib5icug/640?wx_fmt=png&from=appmsg&random=0.20293221876416956&random=0.7911974389503544&random=0.056786363707499676&random=0.6861878288342258)

还是不行，直到看到一位大神评论：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOhADka5NAOAvb66tOckkYQU4tvaRY9WXA0oEwndguicpfG3eHChr0swg/640?wx_fmt=png&from=appmsg&random=0.6586510333405886&random=0.7386823030022605&random=0.4924615578073257&random=0.9286821226709903)

04

初级脚本最终版  

  

01

爱马仕加密最终方案：自建 pthread_create

```
function hook_pthread() {
  var pthread_create_addr = Module.findExportByName(null, 'pthread_create');
  console.log("pthread_create_addr,", pthread_create_addr);
  var pthread_create = new NativeFunction(pthread_create_addr, "int", ["pointer", "pointer", "pointer", "pointer"]);
  Interceptor.replace(pthread_create_addr, new NativeCallback(function (parg0, parg1, parg2, parg3) {
    var so_name = Process.findModuleByAddress(parg2).name;
    var so_path = Process.findModuleByAddress(parg2).path;
    var so_base = Module.getBaseAddress(so_name);
    var offset = parg2 - so_base;
    console.log("so_name", so_name, "offset", offset, "path", so_path, "parg2", parg2);
    var PC = 0;
    if ((so_name.indexOf("libexec.so") > -1) || (so_name.indexOf("xxxx") > -1)) {
      console.log("find thread func offset", so_name, offset);
      if ((207076 === offset)) {
        console.log("anti bypass");
      } else if (207308 === offset) {
        console.log("anti bypass");
      } else if (283820 === offset) {
        console.log("anti bypass");
      } else if (286488 === offset) {
        console.log("anti bypass");
      } else if (292416 === offset) {
        console.log("anti bypass");
      } else if (293768 === offset) {
        console.log("anti bypass");
      } else if (107264 === offset) {
        console.log("anti bypass");
      } else {
        PC = pthread_create(parg0, parg1, parg2, parg3);
        console.log("ordinary sequence", PC)
      }
    } else {
      PC = pthread_create(parg0, parg1, parg2, parg3);
      // console.log("ordinary sequence", PC)
    }
    return PC;
  }, "int", ["pointer", "pointer", "pointer", "pointer"]))
}
hook_pthread();

```

 爱马仕加密的初步反调可以用此脚本无脑过。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOVmPj4JT4LSO3SUxic8BYib83xKzJS9SAKgaGibW1stv1Y3KJzvITVV5LQ/640?wx_fmt=png&from=appmsg&random=0.07314996661521844&random=0.21392432151331864&random=0.12773470704090095)

我们直接看 if elseif 我改为 1 使其错误，得到 offset 偏移量：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfO3SLN90JfCHjHkyDznAD9shb5Skoqdvd1y1TVV5TmQ3fjnH9CwAYicUg/640?wx_fmt=png&from=appmsg&random=0.25737332471337226&random=0.022583758972157808&random=0.6110486171108709)

然后一个个填进 if elseif 里即可无脑过。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOpx8Y8sVdDdibeIxNybJv9UIqBs8W4QPhQwI43tXmXtj1mHqlDZH1fMw/640?wx_fmt=png&from=appmsg&random=0.8660826156510502&random=0.6672756196225622&random=0.5171069171043878)

02

梆某人最终脚本

```
function replace_str() {
    var pt_strstr = Module.findExportByName("libc.so", 'strstr');
    var pt_strcmp = Module.findExportByName("libc.so", 'strcmp');
    Interceptor.attach(pt_strstr, {
        onEnter: function (args) {
            var str1 = args[0].readCString();
            var str2 = args[1].readCString();
            if (str2.indexOf("tmp") !== -1 ||
                str2.indexOf("frida") !== -1 ||
                str2.indexOf("gum-js-loop") !== -1 ||
                str2.indexOf("gmain") !== -1 ||
                str2.indexOf("gdbus") !== -1 ||
                str2.indexOf("pool-frida") !== -1||
                str2.indexOf("linjector") !== -1) {
                // console.log("strstr-->", str1, str2);
                this.hook = true;
            }
        }, onLeave: function (retval) {
            if (this.hook) {
                retval.replace(0);
            }
        }
    });
    Interceptor.attach(pt_strcmp, {
        onEnter: function (args) {
            var str1 = args[0].readCString();
            var str2 = args[1].readCString();
            if (str2.indexOf("tmp") !== -1 ||
                str2.indexOf("frida") !== -1 ||
                str2.indexOf("gum-js-loop") !== -1 ||
                str2.indexOf("gmain") !== -1 ||
                str2.indexOf("gdbus") !== -1 ||
                str2.indexOf("pool-frida") !== -1||
                str2.indexOf("linjector") !== -1) {
                console.log("strcmp-->", str1, str2);
                this.hook = true;
            }
        }, onLeave: function (retval) {
            if (this.hook) {
                retval.replace(0);
            }
        }
    })
}
replace_str()

```

so 的加载流程，那么就会知道 linker 会先对 so 进行加载与链接，然后调用 so 的. init_proc 函数，接着调用. init_array 中的函数，最后才是 JNI_OnLoad 函数, 但是啥也不管一把梭是最方便的，可能会有点卡，多起几次。(https://bbs.kanxue.com/thread-278145.htm)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hFPkDXcMlMvhkdMhE4gHIZ1P5km7OWfOadevonicCRzhojic006RbIRrmcqsx3ksjNzSibR58r3qzMbLFibff6vjHg/640?wx_fmt=png&from=appmsg&random=0.8072778441474695&random=0.03529120284000564) 

05

小叙  

  

最近和前启明师傅聊了一下过抓包检测的问题，佬给我提供了一个很好的思路，过几天再更新通杀 APP 抓包。