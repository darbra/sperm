> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzMjE4NDgyMg==&mid=2247485635&idx=1&sn=cc496104f7ac8fd584f2d34800a63f3c&chksm=c25edf5af529564c2037b31b9b080b3e5e72b98770e3822cef6b27006db6f29a55b0b5232169&mpshare=1&scene=1&srcid=0303BEkGyo0ipMnH418bEqua&sharer_sharetime=1646295077139&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/j6JcMCXCIIgjR4j04DUarll32p0Y8SYoeo8jNsFMARNkY2BrIic2VdSwK6o3k3BDshb8KJic9UTKhACvbibib1hiaicg/640?wx_fmt=png)

        因为相信， 所以看见           

  

  

  

  

答疑，就是在每周四，把问的比较多的，统一回答下。

写文章也会比私信回复更详细些。

本来应该是昨天发的，昨天睡觉去了，拖到了今天。

  

0

序言

可能有的大佬并不知道 frida gadget 是个啥。  

这里先看看官方对于 gadget 的解释

官方链接

https://frida.re/docs/gadget/

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TBQqP8Yibozsk3m5RccHHsxib1Br2MR1q2oYQaNcfpFR3EOWB51oxs1Mg/640?wx_fmt=png)

这句英语意思大概意思是:

![](https://mmbiz.qpic.cn/mmbiz_png/4gCJbFBBaxgr0WD3mMgto4yFaYwwjQMbuxDDBKibrhNlW5YFLV3K1XvkGj1sP1BiaYtibMLdQVrvth08BVUWP7oGw/640?wx_fmt=png)

frida Gadget 是一个动态库，如果注入不适用于当前场景 (一般是被检测 或者没 root)，就用程序加载这个库来达到 hook 的效果。

frida gadget 使用场景

1.  免 root 使用 frida
    
2.  反调试 反 root 反 frida 很强，绕不过去的时候
    
3.  frida 持久化，frida gadget 直接嵌入 app 不用每次打开 frida_server 了
    

这里用一个案例演示 frida 免 root 注入  

app 一般分为有 so 库和无 so 库，**这篇文章演示有 so 库的情况**。

连接方式一般分为等待连接  和 直接执行脚本，**这篇文章演示****直接执行脚本**。

主要分为以下几个步骤

1  

    下载 frida gadget.so 

**2**  

    app so 添加依赖 加入 frida gadget.so

3

    编写 frida gadget config 配置文件

4

    编写注入 js

5

    打包新 apk 执行

  

1

 下载 frida gadget.so

去哪里下载呢？  

当当当当

当然是全球最大同性交友网站 ** github**

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TCZXviaurMaByIKzibfmXcDLiaiaPtPrmdhhTvv0tKZ8c6944YyibzQgQLOw/640?wx_fmt=png)

直接去 frida 的 github 仓，点击 release **找想要的版本** 去下载就可

链接 https://github.com/frida/frida/releases  

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TewpRHcTdoicnSvAZrHBn3K9cibdzIntcvGXtianHTxXqLxibSZXicreBtQQ/640?wx_fmt=png)

  

2

app so 添加依赖 加入 frida gadget.so

这一步的原理是，让 app 在执行 so 的文件的时候，加载 frida-gadget.so  

大部分 so 文件，在运行的时候，都有一些依赖库。

这一步就是把 frida gadget.so **加入到 apk 本身 so 的依赖库中**。

这一步很多大佬是用 lief 实现的。

实际上实现这一步的办法挺多的。

喜欢用 ubuntu 的，可以安装 patchelf 然后一行命令就搞定了

patchelf 地址：https://github.com/NixOS/patchelf

```
patchelf --add-needed frida-gadget.so apk.so

```

so easy , 再也不用担心你不会添加依赖了。

**实例演示：**

看看这里的示例 apk

apk 只有 armv7 的 so 文件 其他都没有

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TCBmHpdU1icAzgMmfiaQTV1qeDvwOz8aLF3rh7ATaC9Zu5GMoVrmyT4qg/640?wx_fmt=png)

wtt.apk 里面有一个 so  **libnative-lib.so**

这里，直接给 libnative-lib.so 添加 frida-gadget.so 依赖库

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TnYynA4KdnvFpMW0G1ic7z2nVlibSCzibCWFibk4SnQkvZVtJZ6yC3O96ag/640?wx_fmt=png)

为了防止检测，frida 这特征也太明显了，很容易被检测 被针对

最好把这个 so 改个名字，比如 libcaiji.so  改名后如下图

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TjsDvSh7W7jibxzFVmP50FXBs3ojw85AUmt3BOaewNLdF5M6fFtQtX1Q/640?wx_fmt=png)

运行 patchelf 添加依赖库

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89T8kic5gPbw5PcdicxN6UibHyNnxZOqTRibGRePOtibHv5FgcF9gIesbvOeAw/640?wx_fmt=png)

没有添加 frida-gadget.so 依赖库之前， **libnative-lib.so** 的依赖库如下

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TXzwsdiaXIcwwr8JOKRApOZeXuOMCwqasd9hYTrP3aj3zrbHHWpsL7ww/640?wx_fmt=png)

搞完之后 拖进 ida 看一下  **libnative-lib.so 的**依赖库

这里可以看到多了一个依赖 so

libcaiji.so 就是 frida-gadget.so 改名后的 so

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89To1G1baQmhyhxvVNzia0ia4ibudNAFXI1QtjqhntxgLJUYUpibTqUccBibwQ/640?wx_fmt=png)

  

3

编写 frida gadget config 配置文件

关于用 gadget.so 直接执行脚本  

官方文章是有介绍的，机器翻译还是有点问题的，凑合看吧

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TmeRbgXG6y3UiaJ9mNCVnTptiaa0XWQ40akBS5RbRCa7icmZibcic9XYAsXA/640?wx_fmt=png)

这里我按照官方的示例，写了一个配置文件

配置**文件名** libcaiji.config.so

这里, 注意配置文件命名的格式，一定是你改名后的名字 + .config.so

```
libxxx.so
libxxx.config.so

```

关于命名规则，官方文档是这么描述的

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TUIwYRFv83CPlLqZ8bL8icXYu4ZWW668iaqX8fC5uphEYgibb01l0D0TNw/640?wx_fmt=png)

这里，按照上面的格式

修改后的 so 名为 ：libcaiji.so

配置文件的名字为：libcaiji.config.so

配置文件代码如下

```
{
"interaction": {
  "type": "script",
  "path": "/data/local/tmp/hook.js"
}
}

```

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89Tmrlibj8ICSiaco3KJkZicTrId6VlWKuHYa7tCwFSLa04k4gdqW3x0rASQ/640?wx_fmt=png)

这里虽然后缀名是 .so  内容其实是 json 配置文件，这点要注意

4

编写注入 js

原 apk 的代码如下

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TIHiakndZr8HXMLupicxNyy13IkpCKnAXPjCnugkIwTa2MiaX3Se1mxDLA/640?wx_fmt=png)

apk 正常运行是这样的

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89Tq7IeDsibZfxJkFcSv5LjicwapEas8iblWSkL9wIGx0bI8cMUTSenjtTMg/640?wx_fmt=png)

这里  编写一个注入的 js

把 弹窗显示的  aaa 改成 bbb  

这里 直接 hook **com.wangtietou.no_root.MainActivity** 的 **aaa** 方法

改下返回值就可以了

hook.js  代码如下

```
var str_name_class = "com.wangtietou.no_root.MainActivity";

Java.perform(function()
{
   var obj = Java.use(str_name_class);

   obj.aaa.implementation = function ()
  {
       return "bbb";
  }
});


```

写完 js 把注入脚本放到之前配置的路径下

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TgBTmgQtpwgdUvhAAeVrqnCgS7GAu2Jk5AsL6qIty7RWKiaGMyjNENibg/640?wx_fmt=png)

这里 /data/local/tmp/hook.js

是之前配置文件配置过的路径

一定要写对，不然，找不到执行的 js, 怎么 hook ，凉凉

5

打包签名新 apk 执行

这里把 改名后的 frida-gadget.so 和 配置文件， 放到 lib 下面 **libnative-lib.so** 同级目录

再压缩

然后签名

就可以了

这里没有修改 dex 文件 也没有修改 AndroidManifest.xml 所以并不用使用 apktool 重新打包

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TG1haUMsx3hyeSC63aVMPduwYE0gTrRkic2HtKMKuq1miaS4dLhTW4FvQ/640?wx_fmt=png)

把修改后的 apk 目录   压缩一下就可以了

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TaOE1nIJaUTTGhPdVAbZHKpOyWum0tQSmiaPoSFRuq8uxK7FqSw3OMtQ/640?wx_fmt=png)

搞完后 用 apktoolbox 重新签个名就 ojbk 了

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TCQgWmRFrzFv98NgVGAG1ySVyic9MPFF7QdpvLWFG6SIa6pKtiaCgA8qA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89TpmWBp6DFrzcB8AO5aYH1V0CZJtYw7WLnYlzh9SpicxUGAPcDxomlJFQ/640?wx_fmt=png)

安装 app 执行 看一下效果

![](https://mmbiz.qpic.cn/mmbiz_png/iaZmLGkOncrIOytnicwdymn8nict7UicT89Tys1icBmyMlQIiaOG86gmhq3EGticvPAcnIZYicRaBFjgrZvMeEPXyGeCjQ/640?wx_fmt=png)

bingo, 成功在没有 root 的手机上用 frida 进行了 hook  

6

结束语

上面搞了这么多，实际上还有更简单的办法

有大佬早就写好了一键脚本

文章地址：https://bbs.pediy.com/thread-268175.htm

脚本地址：https://github.com/nszdhd1/UtilScript/

只不过，脚本用起来不太灵活。

**我准备把脚本改改，让脚本用起来更方便。**

搞完会分享给大佬们的。

这个文章用到的所有文件，**周末会录个视频，然后一起发上去。**

以上。

王某某   2021.0903 于十平米出租屋。  

关于作者：

一个乙方安全公司搬砖的菜鸡，移动安全从业者。

最近忙着找女票，忙着在 b 站当扑街 up 主。

b 站 / 公众号 :  移动安全王铁头  

希望和大佬们一起学习，一起成长  

  

![](https://mmbiz.qpic.cn/mmbiz_png/D1XGIISrRhKQksamIGXRxFbSQuNUWamJYEUwU8KjhNprqa8STuc02vIUak808dBS7Fiao4hg6FS876bicD3uJPgQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/hib5mZ2t0cdTmGAgkySPqAr20oBpFicUtQaleTGbgBdnvlIjH1SRficZ0YseibXwjTN5qd6npxn5QvTVN35MV9v86w/640?wx_fmt=png)

点个

![](https://mmbiz.qpic.cn/mmbiz_png/t3ZJ9oCd1YcJavCa7NoYkIDPhAtHQg5pOlFhHYic59ia9ic2gQGkZFHurrA63iaQeCbCHjial8ZW4XEd1HhOLZj3btQ/640?wx_fmt=png)

在看

你最好看