> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ashenone66.cn](https://ashenone66.cn/2022/05/13/mou-feng-kong-sdk-ni-xiang-fen-xi/#toc-heading-59)

> 大 A 的个人博客，用于记录学习过程

[](#总体介绍 "总体介绍")总体介绍
--------------------

  国内某知名风控 SDK，本地从 Java 层和 Native 层收集大量设备信息，上传至服务器后生成设备指纹。SDK 混淆非常严重，Java 层的字符串加密可以被 JEB 自动解密，影响不大，但是类名和变量名全是 0 和 O 只能硬着头皮看

![](https://hexo-1259492165.cos.ap-shanghai.myqcloud.com/images/ishumei/java_obfuscation.png)

**java_obfuscation**

  native 层则是使用了 ollvm 和字符串加密

![](https://hexo-1259492165.cos.ap-shanghai.myqcloud.com/images/ishumei/native_ollvm.png)

**native_ollvm**

![](https://hexo-1259492165.cos.ap-shanghai.myqcloud.com/images/ishumei/native_str.png)

**native_str**

  对付 ollvm 这里我使用了 hluwa 大佬了 [obpo-plugin](https://github.com/obpo-project/obpo-plugin)，对我的帮助很大。然后根据静态分析找到字符串解密的函数，使用 frida 主动调用或者用 unidbg 的主动调用来进行字符串解密即可。

### [](#整体流程 "整体流程")整体流程

  SDK 的工作流程大概是先从服务器端请求一个配置文件，然后根据这个配置文件分别从 Java 层和 Native 层收集设备信息的 dict，最后在上传之前将 dict 的每个 key 替换成 a1、a2 之类的无意义的 key，防止被人从 key 中理解到信息的含义。最后加密 dict 并上传至服务器。替换 key 名称使用了注解的方式，如下：

![](https://hexo-1259492165.cos.ap-shanghai.myqcloud.com/images/ishumei/java_replace_key.png)

**java_replace_key**

代码大致如下：

Java

```
if(!field.getName().equals("serialVersionUID")) {
  field.setAccessible(true);
  Object value = field.get(arg8);
  CostumSerializable mCostumSerializable = (CostumSerializable)field.getAnnotation(CostumSerializable.class);
  if(mCostumSerializable == null) {
      v1.put(field.getName(), value);
  }
  else if(value != null || (mCostumSerializable.enableReplaceName())) {
      boolean v7 = mCostumSerializable.isMap();
      String v0_2 = mCostumSerializable.keyname();
      if(v7) {
          v1.put(v0_2, Util.costumSerialize(value));
      }
      else {
          v1.put(v0_2, Util.normalize(field, value));
      }
  }
}

```

### [](#Config "Config")Config

  Config 的内容如下

json

```
{"all_atamper":true,"core_atamper":true,"hook_java_switch":true,"hook_switch":false,
"risk_apps":[
{"xposed":{"pn":"de.robv.android.xposed.installer","uri":""}},
{"controllers":{"pn":"com.soft.controllers","uri":""}},
{"apk008v":{"pn":"com.soft.apk008v","uri":""}},
{"apk008Tool":{"pn":"com.soft.apk008Tool","uri":""}},
{"ig":{"pn":"com.doubee.ig","uri":""}},
{"anjian":{"pn":"com.cyjh.mobileanjian","uri":""}},
{"rktech":{"pn":"com.ruokuai.rktech","uri":""}},
{"magisk":{"pn":"com.topjohnwu.magisk","uri":""}},
{"kinguser":{"pn":"com.kingroot.kinguser","uri":""}},
{"substrate":{"pn":"com.saurik.substrate","uri":""}},
{"touchsprite":{"pn":"com.touchsprite.android","uri":""}},
{"scriptdroid":{"pn":"com.stardust.scriptdroid","uri":""}},
{"toolhero":{"pn":"com.mobileuncle.toolhero","uri":""}},
{"huluxia":{"pn":"com.huluxia.gametools","uri":""}},
{"apkeditor":{"pn":"com.gmail.heagoo.apkeditor.pro","uri":""}},
{"xposeddev":{"pn":"com.sollyu.xposed.hook.model.dev","uri":""}},
{"anywhere":{"pn":"com.txy.anywhere","uri":""}},
{"burgerzwsm":{"pn":"pro.burgerz.wsm.manager","uri":""}},
{"vdloc":{"pn":"com.virtualdroid.loc","uri":""}},
{"vdtxl":{"pn":"com.virtualdroid.txl","uri":""}},
{"vdwzs":{"pn":"com.virtualdroid.wzs","uri":""}},
{"vdkit":{"pn":"com.virtualdroid.kit","uri":""}},
{"vdwxg":{"pn":"com.virtualdroid.wxg","uri":""}},
{"vdgps":{"pn":"com.virtualdroid.gps","uri":""}},
{"a1024mloc":{"pn":"top.a1024bytes.mockloc.ca.pro","uri":""}},
{"drhgz":{"pn":"com.deruhai.guangzi.noroot2","uri":""}},
{"yggb":{"pn":"com.mcmonjmb.yggb","uri":""}},
{"xsrv":{"pn":"xiake.xserver","uri":""}},
{"fakeloc":{"pn":"com.dracrays.fakeloc","uri":""}},
{"ultra":{"pn":"net.anylocation.ultra","uri":""}},
{"locationcheater":{"pn":"com.wifi99.android.locationcheater","uri":""}},
{"dwzs":{"pn":"com.dingweizshou","uri":""}},
{"mockloc":{"pn":"top.a1024bytes.mockloc.ca.pro","uri":""}},
{"anywhereclone":{"pn":"com.txy.anywhere.clone","uri":""}},
{"fakelocc":{"pn":"com.dracrays.fakelocc","uri":""}},
{"mockwxlocation":{"pn":"com.tandy.android.mockwxlocation","uri":""}},
{"anylocation":{"pn":"net.anylocation","uri":""}},
{"totalcontrol":{"pn":"com.sigma_rt.totalcontrol","uri":""}},
{"ipjl2":{"pn":"com.chuangdian.ipjl2","uri":""}}],"risk_dirs":[{"008Mode":{"dir":".system/008Mode","type":"sdcard"}},
{"008OK":{"dir":".system/008OK","type":"sdcard"}},
{"008system":{"dir":".system/008system","type":"sdcard"}},
{"iGrimace":{"dir":"iGrimace","type":"sdcard"}},
{"touchelper":{"dir":"/data/data/net.aisence.Touchelper","type":"absolute"}},
{"elfscript":{"dir":"/mnt/sdcard/touchelf/scripts/","type":"absolute"}},
{"spritelua":{"dir":"/mnt/sdcard/TouchSprite/lua","type":"absolute"}},
{"spritelog":{"dir":"/mnt/sdcard/TouchSprite/log","type":"absolute"}},
{"assistant":{"dir":"/data/data/com.xxAssistant","type":"absolute"}},
{"assistantscript":{"dir":"/mnt/sdcard/com.xxAssistant/script","type":"absolute"}},
{"mobileanjian":{"dir":"/data/data/com.cyjh.mobileanjian","type":"absolute"}}
],
"risk_file_switch":true,"risk_files":"UUNTKuZ2FkqjAACv32cnhokTpMQKyKsPcAdizg0dWcnjz2VsaZWaXVutOsepqx0tF4UzvwTYMEs/MidvfKDHFeEXd9cFXU9Nn6LfHRWAd/vTQHaR0Wpl6+zPW6GoF+3F/sEt+ISOU9r+AjFqqK6OtJDgJJdaT3yWrE945sev3b/zuYKXhv0LTQnLuRnUDvR4+DSfgYsRElEMVXFyaFU9I7Od20pxyB1p2qN+U6bD0LNEFiU4ZzqvDrCj48XoErqUOkYIz5ZsOf95ak9PQpaEoVFNJBd841tkQlREvc84u4Ck0i74Sq8kVXnhOQAYNQgmtBLFNImXb5e/LLg+/F94z2tixp/m265In2CS9966dgiNkSSYB7DzY4x56oqYbDVr3VRJD/r4eyNzxxU/XuYk5+l27n1UJoD8lZVJ1WFkAIye0zDk/BDwx0wagiTQh492MWnps/kmSXQAKXzJKJqF8xIT5acT9amMnIQVcChxtj6roeZlFFLH74+3rjbK5+2t39xvh1IeoUhdNhaEHTkDfZHeU62pMANSRLVmwb9rUQPQ6Rs2ShYnTc+DiKNwu/8hNCBpWulhv82Bqb45oMXbpu+fcISm/SNbBi3s+8BSEqK/iNKTCd+T+dCYwnNjMYc8IM+hmSnZLnZ4ZOhdnF7kPd+iobTpieOflMiO6/wcp8CHYsTWkKrCBRfxNsP+K196jAZ8c4MzmV04G5wH5fHZ9zTBVVpq3StLtqwYXeDFEMcv4id8TXgjrvi9XRHEG/0lHtkAkxp8yoM53GnfeUK8yx2oyzhIZ5NHBwgSw1t0NGcHFkawcRPp0Wu2GG9W0aGaFAG48+Ue7lR40shdnOgSxfwJ79o6Wtqwij2LzQPx2ETYn5Xb0sTzURn0IBxh1G9xs+QfpGdIJHQ1WZUmOe3hH17DUn9jFsxm0x5YS2K8rjO4O+xgW0CdALd6uQk7s9vSQr5fVlo24XKcBUM68Z97M8O9KKNDtiJFU/1/7EoA74LEEyVPjdRFCPx5el/qlQhEEZJJ65trohfor3mPrTkO0vYyhVyEUfu8d4sMFFThVZnovqbnGCfWvrdiVV1MZLy1iCbKnG0FXI166/wSF8/oIBuYF0Q5usOZi45LpXx+gLjeOAtxR/VHOfXfs1wUFvgrmO0q2mOhtIzsZhrPtjNjk2sNp9oM7r2FV0lJXD5pfXtwwlNS4W3FyzgY/rzAA+kBvvYlsQXLfWdhJVqd6/ArAf8FyYvw7mYMYifhOjNPJLGAfbsij2YAYPXk0m648SPkhBNLxWq5e6Ww/g2yCwRSzB07xrxuKq7wptjyjwU1lbk47qiR+XesuiG3p4/IvtLtLjuXzNFkMNfFs1SrAuAYUc0nmu7th0oWRfz+pMUtsLDAQ3shioIaNmy7S9wmqtozKfaHVCUMN+IvSWkrQaCtzRaQB73o0SMkgn5P3RpUfJiHjcWV5egBkHQoh3LX4pHivf1Bf5zJRH/evLXHYf8egmgxKAH4K6SnOFLGbhJyNdJfSS1qyOcvd8aHuai9Bgu16ZD7/yVvyqDbX6KPL90umNj8T6/dkM/4Puly5UbVA+7WdtAMoqEaZzhHXTg70XQQLIsoOH1vyhTlV5jBEtYI/5AOoglt43fHCom869gcLi4avImDYYdPwRLmdXcNBHI/yD0SwGI7gVxWOHCvzd3cYZ2fqkfquzm3OTk+W/YUTfh3q/pnffmuZossqROtjo5rUvyAQh86Y5sQdOUKzS6mWxnA3bJjkWc1su6EjHPjEjhwWPQI97XCqq+AM9zwe2536Vw7CqwdDvTjYaaoMf3wtM8wthIUyyIjN8OblN37H9G+Q/IbIViufH3D5MJWtEfUMEUMMsXNCj+HWP72tlFXwWVDFc2m+uvSXJmzgSGERdUi2LxRq1YA6PZKsBkVI6M+IWmYepBLMdP/JlMlqJBtqtqkt2JpF+CrW/6Pe+DnB71qltw3s8Bj4kNuPrehHk62zPFbK5qbgM9uwsui+fiqwd/gnAlXfbMVLB+pm7BPNt66TZ4UXWzB9s+3xzjZ5NW7bZjn1b8H6d44ibW3hL47jxW+jx5cvOxi2EeLIc+dp/gIowoSUTWymSYHZgTXcoJWA5/xpmR7BKsKnjlTcVnqDnjT3F4YUa28LYtYYRKfFK7BmwZmmR+NwRr4u7mGA0sqGRZM2XwYj2ir/Pk4agps9a6KZyw0zTC8LLibx9WlSbaZcD7vzyyD4JYSYo85O86rVZTe03kKzlE0xZ+8j2MaPNPDRtlJiXzVggf0LzIGahdI2ez6/ZL6Ftab5Gm2f9CRcK0MCKeZeaABeolfGqA7g3sJGQ8KI8ndcR6qALUtmZrvpW6bL8wO9HINg8iQspn3bTRZoGpMfPiFPFAzdfxh9HCgiQD0vsN/T76swvNrd2iAAoAwtGFMLN6T7t29NyFLxaUJh5+etpKi7CKSc+jiiYBUR3SF+PrzhcnUk+HQRLCQJAUfGAUKdnw9KXlkcBYCWB1qUMissDvSI4sfARgMe607LyQlknImNPHAs3ZTnOFN5DzHZxvO8NrvXvHqp8sfzOzUbwc9+5jQPsz4n1nOAQzWUWWITxBGtUaf4i++PDGi6hF8NprhRBmUB+tjlp1xPrzTZD/DQfKtvMSpldgAOBqHHODh0tHR/pp4+Y4Lh+nGDqn/+nnCNseLx2nfmx8P+7Y7ztHLFzHUtyvb+P8ryTxgzgaX8HeMKEmBImwn9otDLNpiUtCAr3Wxc5HT13STAFOFqX615nTV59ob+ohQnrRUsDO+60hRI2G56NMhZo54aMFkawR7pWi2u9Y0ExthPX75wR01pFMOFq3JaeV8PAoMWLa11YQOidnL1g7eG1W83tgQysgfgBokymfTgArQl0muYJdfblBVfGiuktyiSyMgr4rI2ghYFyF6bmeiT2VKxOgXldyaKu29iFxm/pP/XFIWw3qSTZrn/Nbzx84VRagbp5lI3gXPi3kJ1gtfwZPd5GUGAMw7tqh4EwR3qi+9LBAgE72UXrEWV5kLb5gkQ9FCU5YaZUTip6wSVte+TSbG+MUIATub91irzCFb3qcQjoe25QUmi2+tJAqLWRFdWqUmy/38CoyTqV36leqWYv+XCYwi3O25q7/LUM+vMXd1KggSVqNuysYsdEdV37/9iwea1Z6WBCFcKreKRhyGZJ9ML4yGgnvrTcQC16S6uhjZYwXwdDPvKS+71XGKqNXCk9QnBD0aHFOxjrhJDM59wYHwkufFHpqefGtvJ6DxtpCGac6JyEbahWkLieNsc9jMwt3yRJNiXkIljPPB3jNOXPEbg2qAy2BJmXjAxc9itBqc5EVdmhw5ScT/nmD1QDC4u3WtDFadGiYyYgbO6Q76N/Zx7/DlUV8pp3xUMJM5mzacakefeGfkHc0JWmTYsaTjojIRxtgVZDDoKGVIrQWYz5wwSqsLyyLy2R8e7RKRQSPzXN1doU4PdwETjOSNbqdAfWZHQcxmW51dKdulW597eOA6fgsV1O1az4IQ9aFX8E9GLPFBFPaaZGa8DA2PlfMIFyH/hB/vqrwmPp9GlY8XuxpjAxPudyDtjEBDrsCLYQbxYUpe8kberM3GUhJwFdkhsevNkLopzKstVjGb2VIUmsE1WCmrPvNH6MdZ/2eIvWpU61wE15ZqmfPs0fY3l/i3O42vv5ba177PAIWRQFJ6n6yK0VH3CG4bkHsaA+/xISSnMtyjlNHey0rUVDVA8hNKA9Cw12yrHftWoOa+QnNUCX6fGCST1K89aXgSVJQJce9s0MBKmEqLlqXTlghpQkQv5p7GKgq8wpnPnbuK4rccYgQpLM6AyMeyXRvCEvPwPBL3n/XDSKqgHZhJdR5Aaq0DvvTny/ZnDDc8hibu8g8mBX1I+QSiTLfTlBWIKNMg2ojceukUnLuR9aE8kB/F5UvuT/09ClmK5XhYfZKqfNeXdT5iEkIcsNp4kBUnSd2uAzwY+1evKyOcBVK3Q+dORa0IuTORodj8S8eCdu1HST8C3UvAq8OnmVPqJaZYBuQg+6D3JKkoShP7cxULAmhLZXdB0ouNZUJYnUp8Rf+yozHe+ssbpgi46fS4qEuZ3XjpRQiixMztQLwXbw9leWgXJzJ8Qho60BtsOX92qqfb/fEeek5J6SpJx3AtEQO4YJe/h4tc3JsVv2vrqmeJvTID2vDhm6p0DAgxnRYbmRWO76x+SWBRFY6pIGWXh3cATPDEiCRmleAq6oMKb66HxKfoWUh/Xj49RC9vTI5GWZdLedWkrLw=",
"sensitive.ainfo":true,
"sensitive.apps":true,
"sensitive.aps":true,
"sensitive.bssid":true,
"sensitive.camera":true,
"sensitive.cell":true,
"sensitive.gps":false,
"sensitive.iccid":true,
"sensitive.imsi":true,
"sensitive.mac":true,
"sensitive.ssid":true,
"sensitive.tel":false,
"white_apps":[]}

```

其中 risk_file 被加密了，在配置文件传入 Native 层后由 Native 进行解密，解密后内容如下:

json

```
[{"key":"cputemp","type":"file","path":"file:///sys/class/thermal/thermal_zone0/temp","option":"upload"},
{"key":"voltage1","type":"file","path":"file:///sys/class/power_supply/battery/batt_vol_now","option":"upload"},
{"key":"voltage2","type":"file","path":"file:///sys/class/power_supply/battery/voltage_now","option":"upload"},
{"key":"maps","type":"file","path":"file:///proc/self/maps","option":"match_ic","words":["com.bly.dkplat","com.excelliance.dualaid","com.bfire.da.nui","com.svm.proteinbox_multi","com.boly.wxnewcopy","com.juying.Jixiaomi.fenshen","com.qihoo.magic","com.godinsec.godinsec_private_space","com.sellapk.goapp","com.yizhi.ftd","com.qihoo.magic.xposed","com.excean.dualaid","com.shiyue.avatarlauncher","com.excean.masaid","com.rinzz.avatar","info.red.virtual","com.depu.wxfs","com.sheep2.xyfs","cn.nineox.pupfish","com.shaker.wxxh.moli.fs","com.fssq.weichat","com.smallyin.Avaassis","com.meta.app.fenshen","com.yxd.shpk_multi","com.xiandong.fst","com.xunrui.duokai_box","com.felix.shuangkai","com.dbhydbhy.duokai","com.xuanmutech.fenkai","com.felix.duokai","com.magic.app.reader01","com.cxhcxh.duokai","com.dongguaququ.duokai","com.felix.fenshen","com.nox.mopen.app","com.boly.wxmultopen","com.tyzhzxl.dkwxzs","com.chufa.skzs","com.lbe.parallel","dkmodel","io.virtualapp","com.coloros.oppomultiapp","com.lbe.parallel.intl","com.jumobile.multiapp","com.jumobile.smartapp","info.cloneapp.mochat.in.goast","com.excelliance.multiaccounts","com.ludashi.dualspace","cn.lapstudio.weiduokai","com.parallel.space.lite","com.jiubang.commerce.gomultiple","cn.lapstudio.aid","com.arc.multi","com.nox.mopen.app","io.virtualapp.luohe","com.ludashi.superboost","com.zhushou.weichat","zc.wormhole","com.lanrun.yxjl","com.ivymobi.multiaccount.free","cloner.parallel.space.multiple.accounts.twoface","com.lylm.dkzs","com.rinzz.avatar","com.ludashi.multspace","com.trigtech.privateme","com.jun.virtual","com.pldasoft.dualapp","com.youxi.shuangkai.help","com.jumobile.multiapp.pro","com.applisto.appcloner","multiple.multiple.parallel.accounts.cloner.mochat","com.bba.vma","com.rinzz.wdf"]},
{"key":"virtio","type":"dir","path":"file:///sys/bus/virtio","option":"exists"},
{"key":"wlan0","type":"dir","path":"file:///sys/class/net/wlan0","option":"exists"},
{"key":"eth0","type":"dir","path":"file:///sys/class/net/eth0","option":"exists"},
{"key":"interrupts","type":"file","path":"file:///proc/interrupts","option":"match","words":["hypervisor","goldfish"]},
{"key":"iomem","type":"file","path":"file:///proc/iomem","option":"match","words":["qemu-pipe","goldfish","vbox"]},
{"key":"ioports","type":"file","path":"file:///proc/ioports","option":"match","words":["virtio","goldfish"]},
{"key":"misc","type":"file","path":"file:///proc/misc","option":"match","words":["vbox","qemu"]},
{"key":"kallsyms","type":"file","path":"file:///proc/kallsyms","option":"match","words":["vbox","qemu","goldfish"]},
{"key":"arp","type":"file","path":"file:///proc/net/arp","option":"match","words":["eth"]},
{"key":"route","type":"file","path":"file:///proc/net/route","option":"match","words":["eth"]}]

```

另外 config 中有个 hook_java_switch，会对 Java 层的函数进行 hook，其中需要 hook 的类和函数硬编码在 native 层，就是文章开头的那一串魔改 base64，主动调用解密后内容如下：

json

```
[{"key":"element","clazz":"dalvik/system/DexPathList$Element","method":"toString","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"insPkg","clazz":"android/app/ApplicationPackageManager","method":"getInstalledPackages","sig":"(I)Ljava/util/List;","param":["int"],"type":1},
{"key":"spget1","clazz":"android/os/SystemProperties","method":"get","sig":"(Ljava/lang/String;)Ljava/lang/String;","param":["java.lang.String"],"type":2},
{"key":"spget2","clazz":"android/os/SystemProperties","method":"get","sig":"(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;","param":["java.lang.String","java.lang.String"],"type":2},
{"key":"secget","clazz":"android/provider/Settings$Secure","method":"getString","sig":"(Landroid/content/ContentResolver;Ljava/lang/String;)Ljava/lang/String;","param":["android.content.ContentResolver","java.lang.String"],"type":2},
{"key":"dev1","clazz":"android/telephony/TelephonyManager","method":"getDeviceId","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"dev2","clazz":"android/telephony/TelephonyManager","method":"getDeviceId","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"native","clazz":"java/lang/reflect/Modifier","method":"isNative","sig":"()Z","param":[],"type":2},
{"key":"debug","clazz":"android/os/Debug","method":"isDebuggerConnected","sig":"()Z","param":[],"type":2},
{"key":"globalget","clazz":"android/provider/Settings$Global","method":"getInt","sig":"(Landroid/content/ContentResolver;Ljava/lang/String;)I","param":["android.content.ContentResolver","java.lang.String"],"type":2},
{"key":"runpro","clazz":"android/app/ActivityManager","method":"getRunningAppProcesses","sig":"()Ljava/util/List;","param":[],"type":1},
{"key":"runtask","clazz":"android/app/ActivityManager","method":"getRunningTasks","sig":"(I)Ljava/util/List;","param":["int"],"type":1},
{"key":"runservice","clazz":"android/app/ActivityManager","method":"getRunningServices","sig":"(I)Ljava/util/List;","param":["int"],"type":1},
{"key":"appinfo","clazz":"android/app/ApplicationPackageManager","method":"getApplicationInfo","sig":"(Ljava/lang/String;I)Landroid/content/pm/ApplicationInfo;","param":["java.lang.String","int"],"type":1},
{"key":"pkginfo","clazz":"android/app/ApplicationPackageManager","method":"getPackageInfo","sig":"(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;","param":["java.lang.String","int"],"type":1},
{"key":"insapp","clazz":"android/app/ApplicationPackageManager","method":"getInstalledApplications","sig":"(I)Ljava/util/List;","param":["int"],"type":1},
{"key":"exec","clazz":"java/lang/Runtime","method":"exec","sig":"(Ljava/lang/String;)Ljava/lang/Process;","param":["java.lang.String"],"type":1},
{"key":"ppdeviceid","clazz":"com/android/internal/telephony/PhoneProxy","method":"getDeviceId","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"gsmdeviceid","clazz":"com/android/internal/telephony/gsm/GSMPhone","method":"getDeviceId","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"psubdeviceid","clazz":"com/android/internal/telephony/PhoneSubInfo","method":"getDeviceId","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"imei1","clazz":"android/telephony/TelephonyManager","method":"getImei","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"imei2","clazz":"android/telephony/TelephonyManager","method":"getImei","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"btmac","clazz":"android/bluetooth/BluetoothAdapter","method":"getAddress","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"cellloc","clazz":"android/telephony/TelephonyManager","method":"getCellLocation","sig":"()Landroid/telephony/CellLocation;","param":[],"type":1},
{"key":"cellchange","clazz":"android/telephony/PhoneStateListener","method":"onCellLocationChanged","sig":"(Landroid/telephony/CellLocation;)V","param":["android.telephony.CellLocation"],"type":1},
{"key":"tel1","clazz":"android/telephony/TelephonyManager","method":"getLine1Number","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"tel2","clazz":"android/telephony/TelephonyManager","method":"getLine1Number","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"iccid1","clazz":"android/telephony/TelephonyManager","method":"getSimSerialNumber","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"iccid2","clazz":"android/telephony/TelephonyManager","method":"getSimSerialNumber","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"netop1","clazz":"android/telephony/TelephonyManager","method":"getNetworkOperator","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"netop2","clazz":"android/telephony/TelephonyManager","method":"getNetworkOperator","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"netopname1","clazz":"android/telephony/TelephonyManager","method":"getNetworkOperatorName","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"netopname2","clazz":"android/telephony/TelephonyManager","method":"getNetworkOperatorName","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"simop1","clazz":"android/telephony/TelephonyManager","method":"getSimOperator","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"simop2","clazz":"android/telephony/TelephonyManager","method":"getSimOperator","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"simopname1","clazz":"android/telephony/TelephonyManager","method":"getSimOperatorName","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"simopname2","clazz":"android/telephony/TelephonyManager","method":"getSimOperatorName","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"imsi1","clazz":"android/telephony/TelephonyManager","method":"getSubscriberId","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"imsi2","clazz":"android/telephony/TelephonyManager","method":"getSubscriberId","sig":"(I)Ljava/lang/String;","param":["int"],"type":1},
{"key":"phcount","clazz":"android/telephony/TelephonyManager","method":"getPhoneCount","sig":"()I","param":[],"type":1},
{"key":"wmac","clazz":"android/net/wifi/WifiInfo","method":"getMacAddress","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"ssid","clazz":"android/net/wifi/WifiInfo","method":"getSSID","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"rssi","clazz":"android/net/wifi/WifiInfo","method":"getRssi","sig":"()I","param":[],"type":1},
{"key":"netid","clazz":"android/net/wifi/WifiInfo","method":"getNetworkId","sig":"()I","param":[],"type":1},
{"key":"bssid","clazz":"android/net/wifi/WifiInfo","method":"getBSSID","sig":"()Ljava/lang/String;","param":[],"type":1},
{"key":"nettype","clazz":"android/net/NetworkInfo","method":"getType","sig":"()I","param":[],"type":1},
{"key":"netsubtype","clazz":"android/net/NetworkInfo","method":"getSubtype","sig":"()I","param":[],"type":1},
{"key":"neicell","clazz":"android/telephony/TelephonyManager","method":"getNeighboringCellInfo","sig":"()Ljava/util/List;","param":[],"type":1},
{"key":"allcell","clazz":"android/telephony/TelephonyManager","method":"getAllCellInfo","sig":"()Ljava/util/List;","param":[],"type":1},
{"key":"scanre","clazz":"android/net/wifi/WifiManager","method":"getScanResults","sig":"()Ljava/util/List;","param":[],"type":1},
{"key":"wifistate","clazz":"android/net/wifi/WifiManager","method":"getWifiState","sig":"()I","param":[],"type":1},
{"key":"wifienable","clazz":"android/net/wifi/WifiManager","method":"isWifiEnabled","sig":"()Z","param":[],"type":1},
{"key":"getlat","clazz":"android/location/Location","method":"getLatitude","sig":"()D","param":[],"type":1},
{"key":"getlon","clazz":"android/location/Location","method":"getLongitude","sig":"()D","param":[],"type":1},
{"key":"lastknownloc","clazz":"android/location/LocationManager","method":"getLastKnownLocation","sig":"(Ljava/lang/String;)Landroid/location/Location;","param":["java.lang.String"],"type":1},
{"key":"providers","clazz":"android/location/LocationManager","method":"getProviders","sig":"(Z)Ljava/util/List;","param":["boolean"],"type":1},
{"key":"bestprov","clazz":"android/location/LocationManager","method":"getBestProvider","sig":"(Landroid/location/Criteria;Z)Ljava/lang/String;","param":["android.location.Criteria","java.lang.String"],"type":1},
{"key":"addlis","clazz":"android/location/LocationManager","method":"addGpsStatusListener","sig":"(Landroid/location/GpsStatus$Listener;)Z","param":["android.location.GpsStatus$Listener"],"type":1},
{"key":"gpsstat","clazz":"android/location/LocationManager","method":"getGpsStatus","sig":"(Landroid/location/GpsStatus;)Landroid/location/GpsStatus;","param":["android.location.GpsStatus"],"type":1},
{"key":"addnmea","clazz":"android/location/LocationManager","method":"addNmeaListener","sig":"(Landroid/location/OnNmeaMessageListener;)Z","param":["android.location.OnNmeaMessageListener"],"type":1},
{"key":"addnmea","clazz":"android/location/LocationManager","method":"requestLocationUpdates","sig":"(Landroid/location/LocationRequest;Landroid/location/LocationListener;Landroid/os/Looper;Landroid/app/PendingIntent;)V","param":["android.location.LocationRequest","android.location.LocationListener","android.os.Looper","android.app.PendingIntent"],"type":1},
{"key":"txloc","clazz":"com/tencent/mapapi/service/LocationManager","method":"getLocationInfo","sig":"()Landroid/location/Location;","param":[],"type":1},
{"key":"file1","clazz":"java/io/File","method":"<init>","sig":"(Ljava/lang/String;)V","param":["java.lang.String"],"type":3},
{"key":"file2","clazz":"java/io/File","method":"<init>","sig":"(Ljava/lang/String;Ljava/lang/String;)V","param":["java.lang.String","java.lang.String"],"type":3},
{"key":"probuild1","clazz":"java/lang/ProcessBuilder","method":"<init>","sig":"([Ljava/lang/String;)V","param":["java.lang.String"],"type":3},
{"key":"probuild2","clazz":"java/lang/ProcessBuilder","method":"<init>","sig":"(Ljava/util/List;)V","param":["java.util.List"],"type":3}]  

```

[](#设备信息汇总 "设备信息汇总")设备信息汇总
--------------------------

总结一下大概收集了哪些信息，具体收集的方式见下文。

### [](#Java层 "Java层")Java 层

<table><thead><tr><th>key</th><th>value</th></tr></thead><tbody><tr><td>电池</td><td>status、level、scale、temperature、voltage</td></tr><tr><td>android.os.Build</td><td>board、model、serial、brand、manufacturer、fingerprint、cpu_abi、cpu_abi2</td></tr><tr><td>CPU</td><td>vendor_id、model_name、count、cpuMHz</td></tr><tr><td>memory</td><td>totalmem</td></tr><tr><td>MediaDrm</td><td>通过 4 个 uuid 生成 uniqueid</td></tr><tr><td>xposed</td><td>是否加载 xposedBridge.jar、hook 的函数名、fieldCache、methodCache、constructorCache</td></tr><tr><td>Accessibility</td><td>isEnabled、EnabledAccessibilityServiceList</td></tr><tr><td>输入法</td><td>已安装输入法列表</td></tr><tr><td>服务商</td><td>locateServiceName、phoneServiceName</td></tr><tr><td>模拟器检测</td><td>特征文件、DeviceID、服务商名、prop 中的字段</td></tr><tr><td>TelephoneService</td><td>DeviceID、服务商、subscribeID、SimSerialNumber</td></tr><tr><td>权限</td><td>READ_PHONE_STATE、WRITE_EXTERNAL_STORAGE、WRITE_SETTINGS、ACCESS_WIFI_STATE、ACCESS_NETWORK_STATE、ACCESS_FINE_LOCATION、ACCESS_COARSE_LOCATION</td></tr><tr><td>蓝牙</td><td>getAddress、IPC 操作 readString</td></tr><tr><td>network</td><td>SSID、BSSID、MAC、IP、type、NetworkInterface、Proxy、isConnected</td></tr><tr><td>位置信息</td><td>BSSID、celllocation</td></tr><tr><td>签名</td><td>反射 packageManager 获取 SubjectDN</td></tr><tr><td>已安装应用</td><td>白名单 app、系统 app、用户 app、自身是否为系统 app、自身包名、版本</td></tr><tr><td>屏幕</td><td>分辨率、亮度</td></tr><tr><td>sd 卡</td><td>AvailableBytes、FreeBytes、TotalBytes</td></tr><tr><td>传感器</td><td>动态特征：gravity、gryo、light; 其他传感器类型和厂商信息</td></tr><tr><td>ContentResolver</td><td>android_id、mock_location</td></tr><tr><td>系统启动时间</td><td>System.currentTimeMillis() - SystemClock.elapsedRealtime()</td></tr><tr><td>进程</td><td>自身进程名</td></tr><tr><td>systemProp</td><td>“ro.debuggable”、”ro.boot.serialno”、”gsm.network.type”、”gsm.sim.state”、”persist.sys.country”、”persist.sys.language”、”sys.usb.state”、”ro.serialno”；”net.dns1”、”net.dns2”、”net.dns3”、”net.dns4”</td></tr><tr><td>virtualApp 检测</td><td>貌似是通过收集与自身的 UID 相同的进程列表来检测的</td></tr></tbody></table>

### [](#Native层 "Native层")Native 层

<table><thead><tr><th>key</th><th>value</th></tr></thead><tbody><tr><td>_system_property_get</td><td>ro.kernel.qemu、ro.debuggable、ro.secure、ro.build.version.release、ro.build.version.sdk、ro.build.display.id、ro.product.model、ro.product.board、ro.product.brand、ro.product.name、ro.product.manufacturer、ro.boot.baseband、ro.boot.bootloader、ro.serialno、ro.build.fingerprint</td></tr><tr><td>UUID</td><td>/proc/sys/kernel/random/uuid</td></tr><tr><td>libc plt hook</td><td>open、stat、access、fopen、printf、read、mmap、socket、rename</td></tr><tr><td>root 检查</td><td>/sbin/su、/system/xbin/su、/system/bin/su</td></tr><tr><td>iotcl</td><td>0x8912、0x8913、0x8927</td></tr><tr><td>cpu ABI</td><td>从 / system/bin/ls 中解析获取</td></tr><tr><td>HW address</td><td>/proc/net/arp</td></tr><tr><td>boot_id</td><td>/proc/sys/kernel/random/boot_id</td></tr><tr><td>cpu 温度</td><td>/sys/class/thermal/thermal_zone0/temp</td></tr><tr><td>电池电压</td><td>/sys/class/power_supply/battery/batt_vol_now;/sys/class/power_supply/battery/voltage_now</td></tr><tr><td>双开应用检测</td><td>读取 / proc/self/maps 检测包名</td></tr><tr><td>可疑文件是否存在</td><td>/sys/bus/virtio;/sys/class/net/wlan0;/sys/class/net/eth0</td></tr><tr><td>虚拟机检测</td><td>字符串匹配:/proc/iomem、/proc/ioports、/proc/misc、/proc/kallsyms、/proc/net/arp、/proc/net/route</td></tr><tr><td>特定目录文件</td><td>遍历文件夹下所有文件，并计算指纹：/vendor/lib、/system/framework、/system/fonts、/vendor/firmware、/system/bin、/data/system</td></tr></tbody></table>

[](#Java层-1 "Java层")Java 层
--------------------------

### [](#电池信息 "电池信息")电池信息

Java

```
public Map getBatteryInfo() {
    HashMap v0 = new HashMap();
    Context v1 = com.ishumei.common.Context.ctx;
    if(v1 != null) {
        try {
            Intent v1_2 = v1.registerReceiver(null, new IntentFilter("android.intent.action.BATTERY_CHANGED"));
            if(v1_2 != null) {
                int v2 = v1_2.getIntExtra("status", 0);
                int v3 = v1_2.getIntExtra("level", 0);
                int v4 = v1_2.getIntExtra("scale", 100);
                int v5 = v1_2.getIntExtra("temperature", 0);
                int v1_3 = v1_2.getIntExtra("voltage", 0);
                v0.put("status", Integer.valueOf(v2));
                v0.put("level", Integer.valueOf(v3));
                v0.put("scale", Integer.valueOf(v4));
                v0.put("temp", Integer.valueOf(v5));
                v0.put("vol", Integer.valueOf(v1_3));
                return v0;
            }
        }
        catch(Exception v1_1) {
            return v0;
        }
    }

    return v0;
}

```

### [](#android-os-Build "android.os.Build")android.os.Build

Java

```
fields = ReflactUtil.getDeclaredFieldsByClassName("android.os.Build");

//Serial 特判
Object Serial = ReflactUtil.invoke("android.os.Build", "getSerial");

//其他内容，通过反射获取
board,model,serial,brand,manufacturer,fingerprint,cpu_abi,cpu_abi2

```

### [](#CPU "CPU")CPU

#### [](#CPU基本信息 "CPU基本信息")CPU 基本信息

读取 / proc/cpuinfo

Java

```
if((TextUtils.equals("hardware", key)) || (TextUtils.equals("vendor_id", key)))
if(!TextUtils.equals("Processor", key) && !TextUtils.equals("model name", key))

```

#### [](#CPU数量 "CPU数量")CPU 数量

通过 fileInputStream 读取

Java

```
/sys/devices/system/cpu/possible
/sys/devices/system/cpu/present

```

或者列出所有以 cpu 开头的文件数量

Java

```
new File("/sys/devices/system/cpu/possible").listFiles(this.fileFilter).length;

```

CPU 频率

Java

```
File cpuinfo_max_freq = new File("/sys/devices/system/cpu/cpu" + cpuIndex + "/cpufreq/cpuinfo_max_freq");

```

读取 / proc/cpuinfo 中记录的 “cpu MHz” 值

Java

```
fis1 = new FileInputStream("/proc/cpuinfo");
cpuMHz = this.readInt("cpu MHz", fis1);
int v3_1 = cpuMHz * 1000;

```

### [](#内存 "内存")内存

#### [](#TotalMem "TotalMem")TotalMem

Build.VERSION.SDK_INT >= 16

Java

```
ActivityManager.MemoryInfo memoryInfo = new ActivityManager.MemoryInfo();
((ActivityManager)this.ctx.getSystemService("activity")).getMemoryInfo(memoryInfo);
return memoryInfo == null ? -1L : memoryInfo.totalMem;

```

读取 / proc/meminfo

Java

```
fis = new FileInputStream("/proc/meminfo");
v0_2 = this.readInt("MemTotal", fis);
long result = ((long)v0_2) * 0x400L;

```

### [](#MediaDrm "MediaDrm")MediaDrm

Java

```
public String getDeviceUniqueIdFromDrm() {
    if(Build.VERSION.SDK_INT < 23) {
        return "";
    }

    StringBuilder data = new StringBuilder();
    try {
        Class MediaDrm = Class.forName("android.media.MediaDrm");
        Constructor UUIDConstructor = UUID.class.getConstructor(Long.TYPE, Long.TYPE);
        Constructor MediaDrmConstructor = MediaDrm.getConstructor(UUID.class);
        Method getPropertyByteArray = MediaDrm.getMethod("getPropertyByteArray", String.class);
        data.append(this.deviceUniqueId(MediaDrmConstructor, getPropertyByteArray, UUIDConstructor.newInstance(((long)0xEDEF8BA979D64ACEL), ((long)-6645017420763422227L)))).append("_");
        data.append(this.deviceUniqueId(MediaDrmConstructor, getPropertyByteArray, UUIDConstructor.newInstance(((long)0x1077EFECC0B24D02L), ((long)0xACE33C1E52E2FB4BL)))).append("_");
        data.append(this.deviceUniqueId(MediaDrmConstructor, getPropertyByteArray, UUIDConstructor.newInstance(((long)0xE2719D58A985B3C9L), ((long)0x781AB030AF78D30EL)))).append("_");
        data.append(this.deviceUniqueId(MediaDrmConstructor, getPropertyByteArray, UUIDConstructor.newInstance(((long)0x9A04F07998404286L), ((long)0xAB92E65BE0885F95L))));
    }
    catch(Throwable v1) {
    }

    return data.toString();
}

private String deviceUniqueId(Constructor MediaDrm, Method getPropertyByteArray, Object UUID) {
    try {
        return Base64.encodeToString(((byte[])getPropertyByteArray.invoke(MediaDrm.newInstance(UUID), "deviceUniqueId")), 2);
    }
    catch(Throwable v0) {
        return "";
    }
}

```

### [](#Xposed "Xposed")Xposed

#### [](#XposedBridge-jar "XposedBridge.jar")XposedBridge.jar

Java

```
private boolean checkXposed(ClassLoader classloader, String keyword) {
    if(classloader == null) {
        return false;
    }

    if(!(classloader instanceof BaseDexClassLoader)) {
        return false;
    }

    try {
        Class DexPathList = Class.forName("dalvik.system.DexPathList");
        Method toString = Class.forName("dalvik.system.DexPathList$Element").getMethod("toString", null);
        Field dexElements = DexPathList.getDeclaredField("dexElements");
        dexElements.setAccessible(true);
        Field pathList = BaseDexClassLoader.class.getDeclaredField("pathList");
        pathList.setAccessible(true);
        Object[] v0_2 = (Object[])dexElements.get(pathList.get(classloader));
        int v4;
        for(v4 = 0; true; ++v4) {
        label_37:
            if(v4 >= v0_2.length) {
                return false;
            }

            String v1_2 = (String)toString.invoke(v0_2[v4], null);
            if(v1_2 != null) {
                boolean v1_3 = v1_2.contains(keyword);
                break;
            }
        }
    }
    catch(Throwable v0) {
        return false;
    }

    if(v1_3) {
        return true;
    }

    ++v4;
    goto label_37;
}

public boolean hasXposed(String arg4) {
    try {
        ClassLoader v1 = ClassLoader.getSystemClassLoader();
        if(!this.checkXposed(v1, arg4) && !this.checkXposed(v1.getParent(), arg4)) {
            ClassLoader v1_1 = this.getClass().getClassLoader();
            if(this.checkXposed(v1_1, arg4)) {
                return true;
            }

            boolean v1_2 = this.checkXposed(v1_1.getParent(), arg4);
            return v1_2;
        }

        return true;
    }
    catch(Exception v0) {
        return false;
    }
}

```

#### [](#XposedCache "XposedCache")XposedCache

Java

```
public Set getXposedCache() {
    HashSet result = new HashSet();
    try {
        Class xposed = ClassLoader.getSystemClassLoader().loadClass("de.robv.android.xposed.XposedHelpers");
        this.getFieldNames(xposed, "fieldCache", result);
        this.getFieldNames(xposed, "methodCache", result);
        this.getFieldNames(xposed, "constructorCache", result);
    }
    catch(Throwable v1) {
    }

    return result;
}

```

#### [](#hook的函数名-大概 "hook的函数名(大概)")hook 的函数名 (大概)

Java

```
public Map xposed() {
    Object[] v2_4;
    HashSet v6_3;
    Class v8;
    Method v7_1;
    int v2_1;
    Field field;
    HashMap v3 = new HashMap();
    try {
        Field[] fields = ClassLoader.getSystemClassLoader().loadClass("de.robv.android.xposed.XposedBridge").getDeclaredFields();
        int index = 0;
        while(index < fields.length) {
            field = fields[index];
            if("sHookedMethodCallbacks".equals(field.getName())) {
                v2_1 = 0;
                goto label_33;
            }

            if("hookedMethodCallbacks".equals(field.getName())) {
                v2_1 = 1;
                goto label_33;
            }

            ++index;
        }

        v2_1 = 0;
        field = null;
    label_33:
        if(field == null) {
            return v3;
        }

        field.setAccessible(true);
        Map v1_1 = (Map)field.get(null);
        if(v2_1 == 0) {
            Class clz = ClassLoader.getSystemClassLoader().loadClass("de.robv.android.xposed.XposedBridge$CopyOnWriteSortedSet");
            Method getSnapshot = clz.getDeclaredMethod("getSnapshot");
            getSnapshot.setAccessible(true);
            v7_1 = getSnapshot;
            v8 = clz;
        }
        else {
            v7_1 = null;
            v8 = null;
        }

        Iterator v9 = v1_1.entrySet().iterator();
    label_61:
        while(v9.hasNext()) {
            Object v2_3 = v9.next();
            String v6_2 = ((Map.Entry)v2_3).getKey().toString();
            HashSet v1_2 = (Set)v3.get(v6_2);
            if(v1_2 == null) {
                HashSet v1_3 = new HashSet();
                v3.put(v6_2, v1_3);
                v6_3 = v1_3;
            }
            else {
                v6_3 = v1_2;
            }

            Object v1_4 = ((Map.Entry)v2_3).getValue();
            if(v8 != null && (v8.isInstance(v1_4))) {
                v2_4 = (Object[])v7_1.invoke(v1_4);
            }
            else if(TreeSet.class.isInstance(v1_4)) {
                Object[] v1_5 = ((TreeSet)v1_4).toArray();
                v2_4 = v1_5;
            }
            else {
                v2_4 = null;
            }

            if(v2_4 == null) {
                continue;
            }

            int v10 = v2_4.length;
            int v1_6 = 0;
            while(true) {
                if(v1_6 >= v10) {
                    continue label_61;
                }

                v6_3.add(v2_4[v1_6].getClass().getName());
                ++v1_6;
            }
        }
    }
    catch(Exception v1) {
    }

    return v3;
}

```

### [](#Accessibility "Accessibility")Accessibility

Java

```
public Map getAccessibilityInfo() {
    HashMap v3 = new HashMap();
    Object v1 = com.ishumei.common.Context.ctx.getSystemService("accessibility");
    Method v2 = v1.getClass().getDeclaredMethod("isEnabled");
    Method v4 = v1.getClass().getDeclaredMethod("getEnabledAccessibilityServiceList", Integer.TYPE);
    Object v2_1 = v2.invoke(v1);
    ArrayList v4_1 = new ArrayList();
    for(Object v6: ((List)v4.invoke(v1, ((int)-1)))) {
        Object v1_1 = v6.getClass().getDeclaredMethod("getId").invoke(v6);
        if(v1_1 == null) {
            Object v1_2 = v6.getClass().getDeclaredMethod("getResolveInfo").invoke(v6);
            v4_1.add((v1_2 == null ? v6.toString() : v1_2.toString()));
            continue;
        }

        v4_1.add(((String)v1_1));
    }

    v3.put("enable", (((Boolean)v2_1).booleanValue() ? "1" : "0"));
    v3.put("service", v4_1);
    v3.put("suc", "1");
    return v3;
}

```

### [](#服务商 "服务商")服务商

Java

```
public String LocationAndPhoneServiceName() {
    StringBuilder data = new StringBuilder();
    try {
        Method getService = Class.forName("android.os.ServiceManager").getMethod("getService", String.class);
        getService.setAccessible(true);
        Object LocationService = getService.invoke(null, "location");
        Object PhoneService = getService.invoke(null, "phone");
        data.append("locateServiceName:").append(LocationService.getClass().getName()).append("|");
        data.append("phoneServiceName:").append(PhoneService.getClass().getName());
    }
    catch(Throwable v1) {
    }

    return data.toString();
}

```

### [](#模拟器检测 "模拟器检测")模拟器检测

#### [](#特征文件是否存在 "特征文件是否存在")特征文件是否存在

none

```
//firmware
/dev/socket/qemud
/dev/qemu_pipe
//firmware2
/sys/qemu_trace
/system/bin/qemu-props

```

读取 / proc/tty/drivers，判断内容中是否有 “goldfish”

none

```
v3 = new File("/proc/tty/drivers");

```

#### [](#比较DeviceID "比较DeviceID")比较 DeviceID

比较 DeviceID 是否为”000000000000000”  
DeviceID 获取方式见下文

#### [](#比较SimOperator "比较SimOperator")比较 SimOperator

检查 SimOperator 是否为 “android”  
SimOperator 获取方式见下文

#### [](#比较BuildProp "比较BuildProp")比较 BuildProp

Java

```
"unknown".equals(Build.BOARD)
"unknown".equals(Build.BOOTLOADER)
"generic".equals(Build.BRAND)
"generic".equals(Build.DEVICE)
"sdk".equals(Build.MODEL)
"sdk".equals(Build.PRODUCT)
"goldfish".equals(Build.HARDWARE)

```

### [](#DeviceID "DeviceID")DeviceID

Java

```
this.phoneService = ReflactUtil.invoke(com.ishumei.common.Context.ctx, "getSystemService", new Class[]{String.class}, new Object[]{"phone"});
deviceId = (String)ReflactUtil.invoke(this.phoneService, "getDeviceId");
(String)ReflactUtil.invoke(this.phoneService, "getDeviceId", new Class[]{Integer.TYPE}, new Object[]{((int)arg8)});

```

### [](#SimOperator "SimOperator")SimOperator

从上到下依次尝试

Java

```
this.phoneService = ReflactUtil.invoke(com.ishumei.common.Context.ctx, "getSystemService", new Class[]{String.class}, new Object[]{"phone"});
result = (String)ReflactUtil.invoke(this.phoneService, "getSimOperator");
result = (String)ReflactUtil.invoke(this.phoneService, "getNetworkOperatorName");

```

### [](#SubscriberId "SubscriberId")SubscriberId

none

```
v0_2 = (String)ReflactUtil.invoke(this.phoneService, "getSubscriberId");

```

### [](#SimSerialNumber "SimSerialNumber")SimSerialNumber

none

```
v0_2 = (String)ReflactUtil.invoke(this.phoneService, "getSimSerialNumber");

```

### [](#权限 "权限")权限

Java

```
Object[] v6 = {((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.READ_PHONE_STATE") == 0 ? 1 : 0)))), ((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.WRITE_EXTERNAL_STORAGE") == 0 ? 1 : 0)))), ((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.WRITE_SETTINGS") == 0 ? 1 : 0)))), ((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.ACCESS_WIFI_STATE") == 0 ? 1 : 0)))), ((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.ACCESS_NETWORK_STATE") == 0 ? 1 : 0)))), ((int)this.isTrue(((boolean)(ctx.checkSelfPermission("android.permission.ACCESS_FINE_LOCATION") == 0 ? 1 : 0)))), null};
if(ctx.checkSelfPermission("android.permission.ACCESS_COARSE_LOCATION") != 0) {
    v1 = 0;
}

v6[6] = (int)this.isTrue(((boolean)v1));
return String.format(CHINA, "%d%d%d%d%d%d%d", v6);

```

### [](#蓝牙 "蓝牙")蓝牙

Java

```
public String getBluetoothInfo() {
        String v0_7;
        Parcel v2_1;
        Parcel v1_3;
        IBinder v0_5;
        try {
            BluetoothAdapter v0_1 = BluetoothAdapter.getDefaultAdapter();
            Field v1 = BluetoothAdapter.class.getDeclaredField("mService");
            v1.setAccessible(true);
            Object v1_1 = v1.get(v0_1);
            if(v1_1 == null) {
                throw new Exception();
            }

            Object v0_2 = Class.forName("android.bluetooth.IBluetooth$Stub$Proxy").getMethod("getAddress", null).invoke(v1_1, null);
            if(v0_2 != null && ((v0_2 instanceof String))) {
                return (String)v0_2;
            }

            throw new Exception();
        }
        catch(Exception v0) {
            try {
                Class v0_4 = Class.forName("android.os.ServiceManager");
                Class.forName("android.bluetooth.IBluetoothManager");
                Class v1_2 = Class.forName("android.bluetooth.IBluetoothManager$Stub");
                Field v2 = v1_2.getField("FIRST_CALL_TRANSACTION");
                v0_5 = (IBinder)v0_4.getMethod("getService", String.class).invoke(null, "bluetooth_manager");
                v2.getInt(v1_2);
                v1_3 = Parcel.obtain();
                v2_1 = Parcel.obtain();
            }
            catch(Exception v0_3) {
                return "";
            }

            try {
                v1_3.writeInterfaceToken("android.bluetooth.IBluetoothManager");
                if(Build.VERSION.SDK_INT >= 21) {
                    v0_5.transact(5, v1_3, v2_1, 0);
                }
                else {
                    v0_5.transact(10, v1_3, v2_1, 0);
                }

                v2_1.readException();
                v0_7 = v2_1.readString();
                goto label_87;
            }
            catch(Throwable v0_6) {
            }

            try {
                v2_1.recycle();
                v1_3.recycle();
                throw v0_6;
            label_87:
                v2_1.recycle();
                v1_3.recycle();
                return v0_7 == null ? "" : v0_7;
            }
            catch(Exception v0_3) {
            }

            return "";
        }

```

### [](#网络信息 "网络信息")网络信息

Java

```
this.wifiService = ReflactUtil.invoke(this.ctx, "getSystemService", new Class[]{String.class}, new Object[]{"wifi"});
this.mConnectionInfo = ReflactUtil.invoke(this.wifiService, "getConnectionInfo"); SSID
String v0_1 = (String)ReflactUtil.invoke(this.mConnectionInfo, "getSSID");bBSSID
String v0_1 = (String)ReflactUtil.invoke(this.mConnectionInfo, "getBSSID");

```

#### [](#MAC "MAC")MAC

none

```
String v0_1 = (String)ReflactUtil.invoke(this.mConnectionInfo, "getMacAddress");

```

#### [](#IP "IP")IP

none

```
String v0_1 = Formatter.formatIpAddress(((Integer)ReflactUtil.invoke(this.mConnectionInfo, "getIpAddress")).intValue());

```

#### [](#BSSID位置信息 "BSSID位置信息")BSSID 位置信息

Java

```
//先检查权限
v0_1.checkPermission("android.permission.ACCESS_FINE_LOCATION", this.ctx.getPackageName()) != 0 && v0_1.checkPermission("android.permission.ACCESS_COARSE_LOCATION", this.ctx.getPackageName()) != 0

for(Object v0_2: ((List)ReflactUtil.invoke(this.wifiService, "getScanResults"))) {
    ScanResult v0_3 = (ScanResult)v0_2;
    v2.add(Util.replaceMaoHao(((String)ReflactUtil.getValue(v0_3, "BSSID"))) + "," + ((int)(((Integer)ReflactUtil.getValue(v0_3, "level")))));
}

```

#### [](#网络类型 "网络类型")网络类型

Java

```
private String _getNetworkType() {
    try {
        NetworkInfo networkInfo = ((ConnectivityManager)this.ctx.getSystemService("connectivity")).getActiveNetworkInfo();
        if(networkInfo != null && (networkInfo.isAvailable()) && (networkInfo.isConnected())) {
            int type = networkInfo.getType();
            if(type == 1) {
                return "wifi";
            }

            return type != 0 ? "unknown" : NetworkInfoCollector.getNetworkType(((TelephonyManager)this.ctx.getSystemService("phone")).getNetworkType());
        }

        return "nil";
    }
    catch(Exception v0) {
        LogUtil.printStackTrace(v0);
        return "unknown";
    }

    return "unknown";
}
//NetworkInfoCollector.getNetworkType
public static String getNetworkType(int arg4) {
    switch(arg4) {
        case -101: {
            return "wifi";
        }
        case -1: {
            return "nil";
        }
        case 0: {
            return "unknown";
        }
        case 1: {
            return "2g.gprs";
        }
        case 2: {
            return "2g.edge";
        }
        case 3: {
            return "3g.umts";
        }
        case 4: {
            return "2g.cdma";
        }
        case 5: {
            return "3g.evdo_0";
        }
        case 6: {
            return "3g.evdo_a";
        }
        case 7: {
            return "2g.1xrtt";
        }
        case 8: {
            return "3g.hsdpa";
        }
        case 9: {
            return "3g.hsupa";
        }
        case 10: {
            return "3g.hspa";
        }
        case 11: {
            return "2g.iden";
        }
        case 12: {
            return "3g.evdo_b";
        }
        case 13: {
            return "4g.lte";
        }
        case 14: {
            return "3g.ehrpd";
        }
        case 15: {
            return "3g.hspap";
        }
        default: {
            return String.format("%d", ((int)arg4));
        }
    }
}

```

#### [](#NetworkInterface "NetworkInterface")NetworkInterface

Java

```
public List getNetwordIFaceAddrs() {
    String v1_1;
    ArrayList v6 = new ArrayList();
    try {
        Object mNetworkInterfaces = ReflactUtil.invoke("java.net.NetworkInterface", "getNetworkInterfaces");
        Method hasMoreElements = Enumeration.class.getDeclaredMethod("hasMoreElements");
        hasMoreElements.setAccessible(true);
        Method nextElement = Enumeration.class.getDeclaredMethod("nextElement");
        nextElement.setAccessible(true);
        while(((Boolean)hasMoreElements.invoke(mNetworkInterfaces, new Object[0])).booleanValue()) {
            NetworkInterface networkInterface = (NetworkInterface)nextElement.invoke(mNetworkInterfaces);
            if(networkInterface.isLoopback()) {
                continue;
            }

            String hostAddress = "";
            String otherAddr = "";
            byte[] hardwareAddress = networkInterface.getHardwareAddress();
            String hardwareAddressString = hardwareAddress != null && hardwareAddress.length > 0 ? Util.replaceMaoHao(Util.getHexString(hardwareAddress)) : "";
            if((hardwareAddressString.isEmpty()) || (hardwareAddressString.equals("000000000000"))) {
                continue;
            }

            Enumeration inetAddresses = networkInterface.getInetAddresses();
            while(inetAddresses.hasMoreElements()) {
                InetAddress inetAddress = (InetAddress)inetAddresses.nextElement();
                if(inetAddress.isLoopbackAddress()) {
                    v1_1 = otherAddr;
                }
                else {
                    String tmp = inetAddress.getHostAddress();
                    if(tmp.trim().length() < 17) {
                        v1_1 = otherAddr;
                        hostAddress = tmp;
                    }
                    else {
                        v1_1 = tmp;
                    }
                }

                otherAddr = v1_1;
            }

            v6.add(networkInterface.getDisplayName() + "," + hostAddress + "," + hardwareAddressString + "," + otherAddr);
        }
    }
    catch(Exception v0) {
    }

    return v6;
}

```

#### [](#Proxy "Proxy")Proxy

Java

```
String proxyHost = System.getProperty("http.proxyHost");
String proxyPort = System.getProperty("http.proxyPort");

proxyHost = Proxy.getHost(ctx);
proxyPort = String.valueOf(Proxy.getPort(ctx));

```

#### [](#isConnected "isConnected")isConnected

Java

```
NetworkInfo v0_1 = ((ConnectivityManager)com.ishumei.common.Context.ctx.getSystemService("connectivity")).getActiveNetworkInfo();
return v0_1.isConnected();

```

### [](#签名 "签名")签名

Java

```
this.packageManager = ReflactUtil.invoke(this.ctx, "getPackageManager");
Object v0_1 = ReflactUtil.invoke(this.packageManager, "getPackageInfo", new Class[]{String.class, Integer.TYPE}, new Object[]{this.ctx.getPackageName(), ((int)0x40)});
Object[] signatures = (Object[])ReflactUtil.getValue(v0_1, "signatures");
byte[] v0_1 = (byte[])ReflactUtil.invoke(signatures[0], "toByteArray");
return ((X509Certificate)CertificateFactory.getInstance("X.509").generateCertificate(new ByteArrayInputStream(v0_1))).getSubjectDN().toString();

```

### [](#InstalledApps "InstalledApps")InstalledApps

有一个 whiteapp 列表  
根据 applicationInfo.flags 来区分是否为系统 app，以及自身是否 debuggable

Java

```
List installedPackages = (List)ReflactUtil.invoke(packageManager, "getInstalledPackages", new Class[]{Integer.TYPE}, new Object[]{((int)0)});
PackageInfo pkgInfo = (PackageInfo)installedPackages.get(index);
String packageName = pkgInfo.packageName;
int appFlag = applicationInfo.flags;
String pkgName = Build.VERSION.SDK_INT < 29 ? applicationInfo.loadLabel(((PackageManager)packageManager)).toString() : "";
appInfoList.add("" + pkgInfo.firstInstallTime + "," + packageName + "," + pkgName + "," + isUsrApp + "," + pkgInfo.versionCode + "," + pkgInfo.versionName + "," + pkgInfo.lastUpdateTime);

```

### [](#自身package信息 "自身package信息")自身 package 信息

Java

```
String v0 = this.ctx.getPackageManager().getPackageInfo("", 0).versionName;
String PackageName = (String)ReflactUtil.invoke(this.ctx, "getPackageName");
String v0_1 = (String)ReflactUtil.invoke(ReflactUtil.getValue(ReflactUtil.invoke(this.packageManager, "getPackageInfo", new Class[]{String.class, Integer.TYPE}, new Object[]{com.ishumei.common.Context.ctx.getPackageName(), ((int)0)}), "applicationInfo"), "loadLabel", new Class[]{PackageManager.class}, new Object[]{this.packageManager});

public int isDebuggable() {
    return (com.ishumei.common.Context.ctx.getApplicationInfo().flags & 2) <= 0 ? 0 : 1;
}

```

### [](#CellLocation "CellLocation")CellLocation

Java

```
private CellLocation getCellLocationData(List cellInfoList) {
    int v3_4;
    int v0_7;
    GsmCellLocation v2_3;
    int v3_3;
    int v0_5;
    CdmaCellLocation v0_4;
    Object mCellIdentity;
    boolean v0_1;
    int type;
    Class CellInfoCdma;
    Class CellInfoLte;
    Class CellInfoWcdma;
    Class CellInfoGsm;
    if(cellInfoList == null || (cellInfoList.isEmpty())) {
        return null;
    }

    ClassLoader clsLoader = ClassLoader.getSystemClassLoader();
    GsmCellLocation mGsmCellLocation = null;
    CdmaCellLocation mCdmaCellLocation = null;
    int v0 = 0;
    int index = 0;
    while(index < cellInfoList.size()) {
        Object v2 = cellInfoList.get(index);
        if(v2 != null) {
            try {
                CellInfoGsm = clsLoader.loadClass("android.telephony.CellInfoGsm");
                CellInfoWcdma = clsLoader.loadClass("android.telephony.CellInfoWcdma");
                CellInfoLte = clsLoader.loadClass("android.telephony.CellInfoLte");
                CellInfoCdma = clsLoader.loadClass("android.telephony.CellInfoCdma");
                boolean v7 = CellInfoGsm.isInstance(v2);
                if(v7) {
                    type = 1;
                }
                else if(CellInfoWcdma.isInstance(v2)) {
                    type = 2;
                }
                else if(CellInfoLte.isInstance(v2)) {
                    type = 3;
                }
                else {
                    v0_1 = CellInfoCdma.isInstance(v2);
                    goto label_44;
                }

                goto label_49;
            }
            catch(Exception v2_1) {
            }

            int v3_1 = v0;
            goto label_160;
        label_44:
            type = v0_1 ? 4 : 0;
        label_49:
            if(type > 0) {
                Object mCellInfoGsm = null;
                if(type == 1) {
                    try {
                        mCellInfoGsm = CellInfoGsm.cast(v2);
                        goto label_67;
                    label_55:
                        if(type == 2) {
                            mCellInfoGsm = CellInfoWcdma.cast(v2);
                        }
                        else if(type == 3) {
                            mCellInfoGsm = CellInfoLte.cast(v2);
                        }
                        else if(type == 4) {
                            mCellInfoGsm = CellInfoCdma.cast(v2);
                        }

                    label_67:
                        mCellIdentity = this.invoke(mCellInfoGsm, "getCellIdentity", new Object[0]);
                        if(mCellIdentity == null) {
                            v0 = type;
                            goto label_168;
                        }

                        if(type != 4) {
                            goto label_116;
                        }

                        v0_4 = new CdmaCellLocation();
                        goto label_79;
                    }
                    catch(Exception v0_3) {
                        v3_1 = type;
                        goto label_160;
                    }
                }
                else {
                    goto label_55;
                }

                goto label_67;
                try {
                label_79:
                    int SystemId = this.invoke2(mCellIdentity, "getSystemId", new Object[0]);
                    int NetworkId = this.invoke2(mCellIdentity, "getNetworkId", new Object[0]);
                    int BasestationId = this.invoke2(mCellIdentity, "getBasestationId", new Object[0]);
                    int Longitude = this.invoke2(mCellIdentity, "getLongitude", new Object[0]);
                    v0_4.setCellLocationData(BasestationId, this.invoke2(mCellIdentity, "getLatitude", new Object[0]), Longitude, SystemId, NetworkId);
                    return type == 4 ? v0_4 : mGsmCellLocation;
                }
                catch(Exception v1_1) {
                }

                v3_1 = type;
                mCdmaCellLocation = v0_4;
                goto label_160;
            label_116:
                if(type == 3) {
                    try {
                        v0_5 = this.invoke2(mCellIdentity, "getTac", new Object[0]);
                        v3_3 = this.invoke2(mCellIdentity, "getCi", new Object[0]);
                        v2_3 = new GsmCellLocation();
                    }
                    catch(Exception v0_3) {
                        v3_1 = type;
                        goto label_160;
                    }

                    try {
                        v2_3.setLacAndCid(v0_5, v3_3);
                        return type == 4 ? mCdmaCellLocation : v2_3;
                    }
                    catch(Exception v0_6) {
                    }
                }
                else {
                    try {
                        v0_7 = this.invoke2(mCellIdentity, "getLac", new Object[0]);
                        v3_4 = this.invoke2(mCellIdentity, "getCid", new Object[0]);
                        v2_3 = new GsmCellLocation();
                        goto label_154;
                    }
                    catch(Exception v0_3) {
                    }

                    v3_1 = type;
                    goto label_160;
                    try {
                    label_154:
                        v2_3.setLacAndCid(v0_7, v3_4);
                        return type == 4 ? mCdmaCellLocation : v2_3;
                    }
                    catch(Exception v0_8) {
                    }
                }

                v3_1 = type;
                mGsmCellLocation = v2_3;
            label_160:
                v0 = v3_1;
            }
            else {
                v0 = type;
            }
        }

    label_168:
        ++index;
    }

    return v0 == 4 ? mCdmaCellLocation : mGsmCellLocation;
}

```

### [](#屏幕分辨率 "屏幕分辨率")屏幕分辨率

Java

```
DisplayMetrics v0_2 = ctx.getResources().getDisplayMetrics();
return String.format(Locale.CHINA, "%d,%d,%d", ((int)v0_2.widthPixels), ((int)v0_2.heightPixels), ((int)v0_2.densityDpi));

```

### [](#SDCARD "SDCARD")SDCARD

Java

```
StatFs mStatFs = new StatFs(Environment.getExternalStorageDirectory().getPath());
mStatFs.getAvailableBytes()
mStatFs.getFreeBytes()
mStatFs.getTotalBytes()

```

### [](#传感器 "传感器")传感器

#### [](#厂商信息 "厂商信息")厂商信息

Java

```
public List getSensorsTypeAndVendor() {
    ArrayList array = new ArrayList();
    try {
        for(Object sensor: this.mSensorManager.getSensorList(-1)) {
            Sensor sensor = (Sensor)sensor;
            array.add(sensor.getType() + "," + sensor.getVendor());
        }
    }
    catch(Exception v0) {
    }

    return array;
}

```

#### [](#动态特征 "动态特征")动态特征

对重力传感器、光照传感器和陀螺仪进行若干次数据采集。

### [](#android-id "android_id")android_id

Java

```
Object[] v4 = {this.ctx.getContentResolver(), "android_id"};
String android_id = (String)ReflactUtil.invoke("android.provider.Settings$Secure", "getString", new Class[]{ContentResolver.class, String.class}, v4);

```

### [](#系统启动时间 "系统启动时间")系统启动时间

Java

```
return System.currentTimeMillis() - SystemClock.elapsedRealtime();

```

### [](#进程名 "进程名")进程名

Java

```
public String getSelfProcName() {
    String procName;
    String result = "";
    try {
        int selfPid = Process.myPid();
        if(this.ctx != null) {
            Iterator RunningAppProcesses = ((ActivityManager)this.ctx.getSystemService("activity")).getRunningAppProcesses().iterator();
            while(true) {
            label_10:
                if(!RunningAppProcesses.hasNext()) {
                    return result;
                }

                Object running = RunningAppProcesses.next();
                ActivityManager.RunningAppProcessInfo procInfo = (ActivityManager.RunningAppProcessInfo)running;
                if(procInfo.pid == selfPid) {
                    procName = procInfo.processName;
                    goto label_20;
                }
                else {
                    procName = result;
                    result = procName;
                    goto label_10;
                }

                result = procName;
            }
        }
    }
    catch(Exception v0) {
    }

    return result;
label_20:
    if(procName == null) {
        procName = "";
        result = procName;
        goto label_10;
    }

    result = procName;
    goto label_10;
}

```

### [](#屏幕亮度 "屏幕亮度")屏幕亮度

Java

```
Object[] screen_brightness = {this.ctx.getContentResolver(), "screen_brightness"};
return (int)(((Integer)ReflactUtil.invoke("android.provider.Settings$System", "getInt", new Class[]{ContentResolver.class, String.class}, screen_brightness)));

```

### [](#mockLocation "mockLocation")mockLocation

Java

```
return Settings.Secure.getInt(v2.getContentResolver(), "mock_location", 0) == 0 ? 0 : 1;

```

### [](#SystemProp "SystemProp")SystemProp

Java

```
Class v0_1 = Context.class.getClassLoader().loadClass("android.os.SystemProperties");
Method v2 = v0_1.getMethod("get", String.class);
"ro.debuggable"
"ro.boot.serialno"
"gsm.network.type"
"gsm.sim.state"
"persist.sys.country"
"persist.sys.language"
"sys.usb.state"

"ro.serialno"单独做md5处理

```

### [](#DNS "DNS")DNS

Java

```
Class v3 = Context.class.getClassLoader().loadClass("android.os.SystemProperties");
Method v4 = v3.getMethod("get", String.class);
v4.setAccessible(true);
String[] v5 = {"net.dns1", "net.dns2", "net.dns3", "net.dns4"};

```

### [](#virtualApp检测 "virtualApp检测")virtualApp 检测

Java

```
private String get_pw_name(int uid) {
    if(Build.VERSION.SDK_INT > 27) {
        return String.format(Locale.CHINA, "u0_a%d", ((int)(uid - 10000)));
    }

    try {
        Field v1 = Class.forName("libcore.io.Libcore").getDeclaredField("os");
        if(!v1.isAccessible()) {
            v1.setAccessible(true);
        }

        Object v1_1 = v1.get(null);
        if(v1_1 != null) {
            Method getpwuid = v1_1.getClass().getMethod("getpwuid", Integer.TYPE);
            if(getpwuid != null) {
                if(!getpwuid.isAccessible()) {
                    getpwuid.setAccessible(true);
                }

                Object v1_2 = getpwuid.invoke(v1_1, ((int)uid));
                if(v1_2 != null) {
                    Field v0_1 = v1_2.getClass().getDeclaredField("pw_name");
                    if(!v0_1.isAccessible()) {
                        v0_1.setAccessible(true);
                    }

                    return (String)v0_1.get(v1_2);
                }
            }
        }
    }
    catch(Exception v0) {
        return String.format(Locale.CHINA, "u0_a%d", ((int)(uid - 10000)));
    }

    return null;
}

private String get_pw_name() {
    int uid = 0;
    try {
        String cgroup = this.execShell("cat /proc/self/cgroup");
        if(!TextUtils.isEmpty(cgroup)) {
            int v3 = cgroup.lastIndexOf("uid");
            int v1_1 = cgroup.lastIndexOf("/pid");
            if(v3 >= 0) {
                if(v1_1 <= 0) {
                    v1_1 = cgroup.length();
                }

                String uid = cgroup.substring(v3 + 4, v1_1).replaceAll("\n", "");
                if(this.checkValidDigit(uid)) {
                    uid = (int)Integer.valueOf(uid);
                }
            }
        }
    }
    catch(Exception v1) {
    }

    if(uid == 0) {
        Context ctx = com.ishumei.common.Context.ctx;
        if(ctx != null) {
            uid = ctx.getApplicationInfo().uid;
        }
    }

    return uid == 0 ? null : this.get_pw_name(uid);
}


private void _setVirtualAppInfo(DeviceInfoData arg13) {
    int isVirtualApp = 1;
    try {
        String pw_name = this.get_pw_name();
        if(!TextUtils.isEmpty(pw_name)) {
            String psResult = this.execShell("ps");
            if(TextUtils.isEmpty(psResult)) {
                goto label_80;
            }

            String[] lines = psResult.split("\n");
            if(lines.length <= 0) {
                goto label_80;
            }

            ArrayList array = new ArrayList();
            int index = 0;
            int count = 0;
            while(index < lines.length) {
                if(lines[index].contains(pw_name)) {
                    int v3 = lines[index].lastIndexOf(" ");
                    String v8 = lines[index];
                    String pkgName = v8.substring((v3 > 0 ? v3 + 1 : 0), lines[index].length());
                    if(!TextUtils.isEmpty(pkgName) && (new File(String.format("/data/data/%s", new Object[]{pkgName})).exists())) {
                        array.add(pkgName);
                        ++count;
                    }
                }

                ++index;
            }

            Locale CHINA = Locale.CHINA;
            Object[] v6_1 = new Object[1];
            if(count <= 1) {
                isVirtualApp = 0;
            }

            v6_1[0] = (int)this.isTrue(((boolean)isVirtualApp));
            arg13.setIsVirtualApp(String.format(CHINA, "%d", v6_1));
            arg13.setVirtualAppProcCount(String.format(Locale.CHINA, "%d", ((int)count)));
            arg13.setVirtualAppPwName(pw_name);
            arg13.setVirtualAppPkgNames(array);
            return;
        }

    label_80:
    }
    catch(Exception v0) {
    }
}


```

[](#Native层-1 "Native层")Native 层
--------------------------------

### [](#UUID "UUID")UUID

fopen 读取 / proc/sys/kernel/random/uuid  
读取失败则设置为 default_id，最终结果中会有一个根据 UUID 生成的 Key，作用不明，应该是用来内容加密。

### [](#prop "prop")prop

通过_system_property_get 函数获取

none

```
ro.kernel.qemu
ro.debuggable
ro.secure
ro.build.version.release
ro.build.version.sdk
ro.build.display.id
ro.product.model
ro.product.board
ro.product.brand
ro.product.name
ro.product.manufacturer
ro.boot.baseband
ro.boot.bootloader
ro.serialno
ro.build.fingerprint

```

### [](#Libc-PLT-HOOK "Libc PLT HOOK")Libc PLT HOOK

从 / proc/self/map 中获取 libc 地址范围并检查以下关键函数地址是否在 libc 范围内（用于检测 frida？），使用 unidbg 跑出来的结果时 false，用 frida 跑出来的结果是 true

none

```
open
stat
access
fopen
printf
read
mmap
socket
rename

```

### [](#ioctl获取网卡信息 "ioctl获取网卡信息")ioctl 获取网卡信息

获取网卡信息，首先调用 ioctl SIOCGIFCONF 找到所有网卡，另外调用 ioctl SIOCGIFFLAGS 查看网卡是否开启，然后调用 ioctl SIOCGIFHWADDR 查找网卡 MAC 地址（高系统版本非 root 环境下不会获取到），最后还会通过是否有 tun 网卡判断 VPN，是否有 ppp 判断拨号网卡

C++

```
#include <cstdio>
#include <cstring>
#include <net/if.h>
#include <sys/ioctl.h>
#include <sys/socket.h>
#include <zconf.h>
#define MAX_IFS 64
int main(int argc, char **argv) {
    struct ifreq *ifr, *ifend;
    struct ifreq ifreq{};
    struct ifconf ifc{};
    struct ifreq ifs[MAX_IFS];
    int sockfd;
    int on;
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    ifc.ifc_len = sizeof(ifs);
    ifc.ifc_req = ifs;
    ioctl(sockfd, SIOCGIFCONF, &ifc);
    ifend = ifs + (ifc.ifc_len / sizeof(struct ifreq));
    for (ifr = ifc.ifc_req; ifr < ifend; ifr++) {
        if (ifr->ifr_addr.sa_family == AF_INET) {
            strncpy(ifreq.ifr_name, ifr->ifr_name, sizeof(ifreq.ifr_name));
            ioctl(sockfd, SIOCGIFHWADDR, &ifreq);
            ioctl(sockfd, SIOCGIFFLAGS, &ifreq);
            on = (ifreq.ifr_flags & IFF_UP) != 0;
            if (strncmp("wlan", ifreq.ifr_name, 4u) == 0) {
                printf("wlan %d\n", on);
            } else if (strncmp("tun", ifreq.ifr_name, 3u) == 0) {
                printf("tun %d\n", on);
            } else {
                continue;
            }
            printf("%02x:%02x:%02x:%02x:%02x:%02x\n",
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[0],
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[1],
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[2],
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[3],
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[4],
                   (int) ((unsigned char *) &ifreq.ifr_hwaddr.sa_data)[5]
                   );
        }
    }
    close(sockfd);
    return 0;
}

```

### [](#Root "Root")Root

通过 access 函数检查文件是否存在

none

```
/sbin/su
/system/xbin/su
/system/bin/su

```

### [](#CPUABI "CPUABI")CPUABI

通过 open 打开 / system/bin/ls，然后用 pread 读取文件头特定偏移值来判断文件 abi

### [](#HW-address "HW address")HW address

通过读取 / proc/net/arp 来读取 HW address，貌似先对 ip 的最后一位进行判断，若为 1 则读取硬件地址并全局保存

### [](#bootid "bootid")bootid

cat /proc/sys/kernel/random/boot_id

### [](#双开应用检测 "双开应用检测")双开应用检测

none

```
["com.bly.dkplat","com.excelliance.dualaid","com.bfire.da.nui","com.svm.proteinbox_multi","com.boly.wxnewcopy","com.juying.Jixiaomi.fenshen","com.qihoo.magic","com.godinsec.godinsec_private_space","com.sellapk.goapp","com.yizhi.ftd","com.qihoo.magic.xposed","com.excean.dualaid","com.shiyue.avatarlauncher","com.excean.masaid","com.rinzz.avatar","info.red.virtual","com.depu.wxfs","com.sheep2.xyfs","cn.nineox.pupfish","com.shaker.wxxh.moli.fs","com.fssq.weichat","com.smallyin.Avaassis","com.meta.app.fenshen","com.yxd.shpk_multi","com.xiandong.fst","com.xunrui.duokai_box","com.felix.shuangkai","com.dbhydbhy.duokai","com.xuanmutech.fenkai","com.felix.duokai","com.magic.app.reader01","com.cxhcxh.duokai","com.dongguaququ.duokai","com.felix.fenshen","com.nox.mopen.app","com.boly.wxmultopen","com.tyzhzxl.dkwxzs","com.chufa.skzs","com.lbe.parallel","dkmodel","io.virtualapp","com.coloros.oppomultiapp","com.lbe.parallel.intl","com.jumobile.multiapp","com.jumobile.smartapp","info.cloneapp.mochat.in.goast","com.excelliance.multiaccounts","com.ludashi.dualspace","cn.lapstudio.weiduokai","com.parallel.space.lite","com.jiubang.commerce.gomultiple","cn.lapstudio.aid","com.arc.multi","com.nox.mopen.app","io.virtualapp.luohe","com.ludashi.superboost","com.zhushou.weichat","zc.wormhole","com.lanrun.yxjl","com.ivymobi.multiaccount.free","cloner.parallel.space.multiple.accounts.twoface","com.lylm.dkzs","com.rinzz.avatar","com.ludashi.multspace","com.trigtech.privateme","com.jun.virtual","com.pldasoft.dualapp","com.youxi.shuangkai.help","com.jumobile.multiapp.pro","com.applisto.appcloner","multiple.multiple.parallel.accounts.cloner.mochat","com.bba.vma","com.rinzz.wdf"]

```

### [](#虚拟机检测 "虚拟机检测")虚拟机检测

匹配文件中是否存在字符串

#### [](#proc-interrupts "/proc/interrupts")/proc/interrupts

#### [](#proc-iomem "/proc/iomem")/proc/iomem

none

```
qemu-pipe
goldfish
vbox

```

#### [](#proc-ioports "/proc/ioports")/proc/ioports

#### [](#proc-misc "/proc/misc")/proc/misc

#### [](#proc-kallsyms "/proc/kallsyms")/proc/kallsyms

#### [](#proc-net-arp "/proc/net/arp")/proc/net/arp

#### [](#proc-net-route "/proc/net/route")/proc/net/route

### [](#特定目录文件 "特定目录文件")特定目录文件

遍历文件夹下所有文件，并计算指纹

none

```
/vendor/lib
/system/framework
/system/fonts
/vendor/firmware
/system/bin
/data/system

```