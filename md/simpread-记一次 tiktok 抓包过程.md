> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/s1doxCFy9riCF9t7zOeIuQ)

叠甲：**文章中所有内容仅供学习交流使用，不用于其他任何目的，仅供学习参考**

**![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9Iib1XiafBSAAWM7ICVKVkLCcb8Do6cSu906mFYPOXlGkvGwcDuRPA3qA/640?wx_fmt=jpeg)**

**00 主要内容  
**

先放效果图 （搜索）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9qaA6s1CPzdOFYWPX1iclkN2YHibrS77qtSKdAGwibb6b1nPMz17Eh18Bw/640?wx_fmt=png&from=appmsg)

**01 安装**

Apk 来源：

```
谷歌商店直接下载
搜索下载：https://www.apkmirror.com/

```

主要问题：tiktok 对于国内用户有锁区的情况，且需要代理，并且除了网络外还会识别设备信息比如说 sim 归属之类。最直接的方式是通过某些手段将其修改，下面有两种方法。（下面介绍第二种方式的插件使用）

```
- 重打包（非root方案）：更改对于设备识别这部分，将开发的插件重新打包进apk，安装后即可使用。（网上有打包好的）
** 注意 **
! 对应的插件推荐：https://github.com/Xposed-Modules-Repo/com.nnnen.plusne
! 网上有很多打包好的，如果想要自己打包则需要一个root设备借助lspatch来进行重打包(https://github.com/LSPosed/LSPatch)
- hook：前提需要xposed环境，通过xposed插件来进行对指定软件进行修改
** 注意 **
! 对应插件推荐：https://github.com/appenv/appenv.github.io

```

打开应用变量选择目标应用

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9jQR9o9j6P7fSw8ibmvtnjzdhv39SBAoLMerSKzrrkeBsy86FHiaaUUyw/640?wx_fmt=png&from=appmsg)                  ![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9YK6AJ38TicCZbHRJ2yRbdLM3GkJ465icTLdFDkuqTI7Zic8XWDaib8s30g/640?wx_fmt=png&from=appmsg)

配置对应 apk 的国家为其他允许使用的海外国家 比如说 us，保存后观察是否生效（如果没生效重启一下设备）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9LvYf8nSbZOvzickCsO3ER5kshXYLY6icyun0ibNE6SJyQPMz00gZ56ykA/640?wx_fmt=png&from=appmsg)

上述配置完成，打开代理，就不会再出现网络的问题

**02 配置抓包环境**

charles 配置抓包：由于目标为海外 app 需要代理，因此需要配置 `External Proxy Settings`为 pc 端海外代理的接口  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9cuh0pguWTUNtWicEHKGYF9kYHpCuBc67yStZR7BeliaTTXe2jicXSZmMA/640?wx_fmt=png&from=appmsg)

配置的地址和 ip 要和你自己的代理一致  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZYeIDv4JOubGQtdBrmoiaib5AYb7mWxwJ9fdspqTEHndpAT1gCxKTE1VhnyavoqZTDqicsafjXJ9IibyaHFeWySQUQ/640?wx_fmt=png&from=appmsg)

抓包插件配置  

*   流量全局转发：使用 **postern** 进行对于流量的转发
    
*   【插件一】升降级脚本:  quic 切换为 http
    

```
// 参考 https://blog.csdn.net/jmm18363027827/article/details/132217390
// 通杀脚本
// xposed 脚本
String packageName = "com.zhiliaoapp.musically";
......
if (lpparam.packageName.equals(packageName)){
    Log.d(TAG, "handleLoadPackage: =======dyCapture=====================");
            Class<?> CronetClient = XposedHelpers.findClass("org.chromium.CronetClient", lpparam.classLoader);
            XposedBridge.hookAllMethods(CronetClient, "tryCreateCronetEngine",
                    new XC_MethodReplacement() {
                        @Override
                        protected Object replaceHookedMethod(XC_MethodHook.MethodHookParam methodHookParam) throws Throwable {
                            Log.d(TAG, "replaceHookedMethod: hook 成功");
                            return null;
                        }
                    });
}

```

*   【插件二】sslpinning：发现连上 charles 就出现断网，以往经验来看是 sslpinning 问题， 使用经典`JustTrueMe`插件即可解决