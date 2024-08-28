> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gYb7tq-lv4xyP4sYSJGZ_g)

前言
--

安卓干不动了，试试换换赛道![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/newemoji/Lol.png)

其实这方面的东西早就很多了，我只是把相关的东西再整理加工一下，一来方便我以后自己看，二来方便大家，不用到处去找资料了

环境
--

苹果手机去哪里买，这个就随意了，你买之前问清楚，系统版本，然后最好是能刷机能升级能抹除还原的，如果你还原完有个没见过的账号让你输入密码的，或者还原过程失败的，**恭喜你，中奖了，有隐藏 ID 锁![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/newemoji/Lol.png)**，然后这个设备大概率就废了，具体就不表了，所以最好问清楚，然后这种手机大概率

我这里有三个二手苹果手机，都是苹果 x（能越狱的手机，最经典的就是苹果 8 和 x，当然其他也行）

```
ios 14.0
ios 16.7.8
ios 16.3

```

然后准备一台电脑，mac 或者 win 都行，以下我以 win 为例

准备一根一边是 USB 头，一边是苹果 light 充电头的线，这种各大电商平台都有  

怎么越狱，下面一个一个说  

常识
--

首先先了解下基本的常识

### 越狱和半越狱

越狱就是，可以侧载 app（侧栽就是不通过 appstore 安装 app），并安装一些调试工具，比如 frida 等之类的工具，可以 hook 调试，可以进手机终端，可以有 root 权限，类似安卓端的 root

半越狱就是只能侧载 app，有些工具没法安装，只能一定程度的操作一些，比如巨魔商店，bootstrap 工具安装完了，就算半越狱，极少部分也能安装 frida，但是调试的时候功能受限，有的没法进终端，即使进入了终端也没有 root 权限

> 巨魔之类的东西，后面有空再说  

### 有根越狱和无根越狱

有根越狱即 rootfull，就是完全版的越狱，这个在低版本的手机上才有，常见在苹果 8 及以下，苹果 x 上已经很少有，除非 ios 版本很低，通俗理解重启手机后越狱状态还在  

无根越狱即 rootless，通俗理解就是重启手机后，越狱状态丢失，安装过的插件还在，只是没法使用，app 列表也没有，需要重新越狱，重新越狱后，之前安装过的越狱插件则会继续出现  

### 签名工具

其实签名工具有很多，目的就是为了给 ipa 签名，才能安装在苹果设备上，下面的安装模式有说

```
全能签
轻松签
牛蛙助手
.....

```

### 苹果 app 安装机制

我们都知道，苹果手机，正常想安装一个 app，有三种方式

#### 1.appstore 里下载安装  

这是大多数人都知道的方法  

#### 2. 用描述文件安装

相信在一个躁动的夜晚，有的朋友肯定去找过那种正能量的 app，然后打开释放情绪，释放完了还不知道咋卸载，还百度了下怎么卸载，没错就是那种![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/newemoji/Yellowdog.png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicUpJMHGYMkmwhtZoNBG4Zr65LfUg0WtDCPuZNNfI2dW0xaFEny52JQw/640?wx_fmt=png&from=appmsg)

#### 3. 证书签名安装

**根据苹果的设定，所有的 app 源文件，打包成 ipa 后缀的文件，都是需要证书签名，然后才能安装在苹果设备上的，上面的描述文件模式，也有做签名操作，只是在签名的时候，苹果允许以描述文件的形式导出（我自己的理解，不知道对不对了）**

这个证书可以是 apple ID 账号，也可以是 udid，也可以是企业证书  

apple ID 这个就不难理解，就是你自己的苹果账号

udid 就是每个苹果设备都有一个专属设备 id，udid 证书来源于 688 元 / 年苹果开发者，然后授权设备 id，这样就可以安装 app 了

企业证书就是早期苹果官方开发给有钱的企业，做 app 开发使用，现在已经停止企业申请，所以企业证书很贵，现在只能租用  

#### 4.testflight 测试版安装

这个有的正能量的 app 也会选择的，不过有设备数量限制，只能多少设备安装

以上相关的东西，苹果官方都有介绍，这里就不细说了  

#### 5. 巨魔安装

这个是利用了苹果的漏洞，实现直接用 ipa 文件安装，且不需要用证书签名，这个操作又叫做侧载，巨魔的安装条件也很苛刻，只有部分设备，部分 ios 版本才可以的  

IOS14.0
-------

其实 ios 版本越低，越好越狱，当然还得看你手机啥型号，反正苹果 x 及以下最好，以上的可能就很难，因为出厂系统版本就很高，可能很难操作

那就以我这个 14.0 的苹果 x 为例

首先你电脑下一个爱思助手，然后 usb 线插上，就会自动读取设备

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicJvo85C2MtSx9L41fVEibmj4wN6qNkLhRZUUmYeVQY7OUwlc1ln74k5A/640?wx_fmt=png&from=appmsg)

能读取出来就行了，然后我之前越狱过，所以显示已越狱，问题不大，重来一遍就行

**如果爱思助手提示让你装什么驱动，你就跟着操作就行了。如果有信任设备弹窗的，点信任**

**另外这里有个坑，安装这个驱动的时候，会导致你的安卓设备，读取有问题，adb devices 无法读取设备，所以就很骚，我之前就遇到了，搞完苹果，调试安卓就傻眼了，所以你最好安装完，用了如果还要调试安卓的话，就把这个驱动卸载了**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicdMW7Jb1EUkABn0TS81RiceEWBg7AWQR1zFL31ibKokCg4F1XthY2T8pQ/640?wx_fmt=png&from=appmsg)

当然也有可能连接失败，换一个 usb 口，换一根线，继续试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicmJ8KgSgudzf7dNaCFdo8WdFcq8icd9wDrbIuKBFAibt4cjialoxpjibwpQ/640?wx_fmt=png&from=appmsg)

有的时候还会出现一直在读取的状态，这个步骤没办法，多次尝试了  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicPr9OxiaA4lAQqEdW5kjmr6rC1onfiaO90GMxMOL9DDYwhzDtJWaD2nSA/640?wx_fmt=png&from=appmsg)

点工具箱，搜索

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicTdP8QP1KIy8VyzxibD34fqiaAyopBL52HxRRsFBHvTDReOMNmd5oX7gw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicw8RWG0agAXdvibK6wWzk5l18vnGK6YeS9xKpv2ibNfwdeWYzAmbsPlxw/640?wx_fmt=png&from=appmsg)

点开，第一次打开的时候要下载插件，然后打开如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicl7HHPj4ZmiaBaVxvohfmJodFJE5tic1N35VVMk4wDsFsbGoCqlOowKDA/640?wx_fmt=png&from=appmsg)

看看，低版本就是这么爽，就很多种越狱的方式，随便选择。

另外这里也展示，不同的系统版本的话，需要选择不同的越狱工具

通常的话：

```
ios15.0以下的版本用unc0ver
ios15.0-16.6用多巴胺 Dopamine
ios16.6以上到ios17（限定苹果x）用palera1n

```

不过低版本我习惯用 unc0ver，这个也是低版本用的最多的越狱工具

我这个手机是 ios14

点击 unc0ver 右边的勾选按钮，然后点下面的开始越狱

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicOpTJplsKbL5umt9GicRjP5aoWY6ZmaeyTzfNKgoeoPtaq1wNTeTiaP8w/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicfoibzkWF19do9qlTHJ2DyokWfgcmhh31fZwfw9UQGiaLTqGI5KGo9IKA/640?wx_fmt=png&from=appmsg)

然后就会让你输入你这个手机上登录的 apple ID 的账号密码

如果你没有登录的话，就在你手机里登录一下账号密码，如果没有 appleID 账号就去注册一个再登录到手机，然后再走这一步

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicJTPibU81msyevXWKapEpTicjETOY5jvWXHf2rkeof6sasr60xm1gm26g/640?wx_fmt=png&from=appmsg)

**这里这个操作，原理就是上面说的 ID 签名的方式，用你的 apple ID 取签名并安装 unc0ver 到你手机。这个详细的我放下一篇说吧，展示如果在未越狱的手机上做一些操作****![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/newemoji/Yellowdog.png)**

好的，输入账号密码，点确定，接下来就是等这个进度条走完了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicE3UGHicTDJibYY0MKjccZg3pMiaZJ3e177w4H4SGayPAYdQXibVibrGETmA/640?wx_fmt=png&from=appmsg)

如果中途出现问题，就多次重试，还有问题可以找个签名工具，自签名安装 unc0ver 到手机上即可，这个步骤主要就是把 app 安装上，你用什么工具其实都无所谓的，所以这里也是为啥要让你登录 apple id

等待片刻，然后手机上就会出现一个 unc0ver 的 app，找到它，点开  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicUOIkXnFV64xibvmTJd4iasXAvMdNPM1dIMgkPhUwguqvRWwSoib3emoAg/640?wx_fmt=png&from=appmsg)

在点开之前，保证你的网络畅通，然后后续操作会需要科学操作（你懂的），这个步骤就省略了，自己想办法了

然后这里，有个坑，有的手机的时间，是不对的，你得先把手机校准到真实的时间，要不然你的网路用不了，这里就很像安卓的那个问题，时间不对的话，浏览器打开任何网站，都提示有错误。而苹果是如果时间有问题，直接整个手机，所有 app 打开都没网络

比如我这里这个手机的时间就是有问题的，设置里配置下即可

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicticTbpfsCRJ0zibNkOZnAWUiaCarJVBoTWBQNF0ykvdjtxkXibqDCQzxibQ/640?wx_fmt=png&from=appmsg)

卧槽，1970，肯定有问题啊，手动改好

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicSJp7G0kZYSpj6gMiaD5OGkB2G4OUmKwbespXVaxpLd35otwaOq47k2w/640?wx_fmt=png&from=appmsg)

改完之后，safari 打开一个网站，确认能访问不  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIichORNylr8ib33zICHfGxODHOSWQHBIYLwTqOGfQcX2ibRmUbjCFYZdQjA/640?wx_fmt=png&from=appmsg)

ok，然后带上科学

打开 unc0ver:  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicqTyGIEibR6LLqkHoOOIPcCJn0EwtszJ0lNe5TxXrUYUAb7dPa9KDJfA/640?wx_fmt=png&from=appmsg)

有弹窗，问题不大，去设置里信任下  

设置 - 通用 - 描述文件与设备管理，找到你自己的那个：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicIMjd61v7DDKKGMs632xTsJ6FRic0XKpRqgMAUicWAWYfLib8DuEdxHfEw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIickgg1h70pt94osUVzupsoanP2fh7FLjmQDNztY5njHibhptIxSS6wPqA/640?wx_fmt=png&from=appmsg)

点一下，变成已验证就可以了  

再次打开 unc0ver

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicyd805M0kqZ4Kw4icyHL4EIb0QLMSicQhQ4RZYViac6Nzib41LsIAm7wiatQ/640?wx_fmt=png&from=appmsg)

点这个【jailbreak】按钮，这个单词意思就是越狱  

等待片刻，这里也是，如果第一次不行，就多点几次  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic7DqQiat3mOFbibSmkVJvkJMbKf1kTdcZ4TUq8ZP8h7w07NQFZCI7zMOg/640?wx_fmt=png&from=appmsg)

出现这个的时候就是成功了，这是他的广告，点叉掉  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicoCMTw6Irc8PxoW1yZ735libT3etKwFx31xJ9vGrKaXPuujS5G5Z6BIg/640?wx_fmt=png&from=appmsg)

到这里，点 ok

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicQwbaZ9HGhZJ5WRhj89rMxIBd0KrqRyqicb6NgYfCkCkUzib5nXibCWX8Q/640?wx_fmt=png&from=appmsg)

这时候手机自动重启，几秒过后，重启成功，然后就会自动安装一个叫 cydia 的 app

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicb1MKK51DSeCNajvju8ICk90Y3Vxq5tOJy3ibvqwGvWGlqgvmrRxicTLw/640?wx_fmt=png&from=appmsg)

点开，这就是越狱成功了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicNRrDTxnOSgrvOJcSMZrl54qf9zu3VSc8HDzF2g8rL5eKLxhAoibz8AQ/640?wx_fmt=png&from=appmsg)

然后装个 frida，添加下 cydia 源

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicqwqXubanVdqX5g7RAsuOY1nfLMktU4ufwNQ0uHfjptlOV45fXr3KPw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicZNRfuvsiatvRjqMx3iaBM50EX9HdsnDBcpfkhwrDbVpFnLOcoFuRWekA/640?wx_fmt=png&from=appmsg)

如下操作：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicrjC3RbFg36jNUsJv8UZGMHhhzlKYUStvuhETLJ2JgBhG95KufX1EWw/640?wx_fmt=png&from=appmsg)

这就有了这个源  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicrvcicY2NAL3MKNYM8MJ7n8CrpON0rhwgRS63cRsz9RMagNqVpVd8TsA/640?wx_fmt=png&from=appmsg)

然后点搜索  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIickwicUewI88gCqEUicyjl2U89pxhVC81ibgy53gnw5YXd0hvT5jIn7xibBg/640?wx_fmt=png&from=appmsg)

输入 frida，点进去安装即可，这里也要科学操作，你懂的，要不然速度很慢

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic6xibMKlibiaiclVMCaLcZhOu15E7n2yia50lPTXjYWDicfZ4j1smEJBxYibrQ/640?wx_fmt=png&from=appmsg)

相关的源，常规的插件，就自行查找资料安装了，网上有很多这方面的  

然后 ios 端的这些插件，只要有越狱环境，有 cydia，他就会自动加载，所以也就是说，frida 现在就是启动状态，不需要像安卓一样去终端手动执行，当然也可以手动执行，比如你要指定魔改 ios 端 frida 启动的时候（这个具体就自行查阅资料了）  

然后 pc 端配置好 frida 环境，最好是相同的 frida 版本，这个相信搞安卓的朋友早就踩过坑了  

我这里用的 pycharm 的虚拟环境，右下角点一下就切换了，相对方便，习惯了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicQzNv4DNV52VUsUIpKAvy720LV6bUVEVzngS2PMNGonQTBbYsyicNqAA/640?wx_fmt=png&from=appmsg)

我手机安装的是 frida16.3.3，切换到指定版本，开一个终端，执行：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicibLxjl603ZbgCVDzP9pgvDsEQOxNo0IXKuib4QjjZcG2ytlsayicRCYkg/640?wx_fmt=png&from=appmsg)

这个什么 wait 设备的，经常遇到，这种大概率是线不行，我 pdd 买的几块钱两根，没法，将就用，拔插下重来：  

好了，现在有了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicE5g8IKGXRBkiccrnMTSP4jicopdkJOqz32SrMgRLHukPOr8c97fLlxkw/640?wx_fmt=png&from=appmsg)

接下来就是相关的 hook 调试操作了，这后续的步骤就跟安卓没啥区别了  

当然也是可以指定 ip 启动的，反正安卓端的操作，ios 端的 frida 上基本都可以

ios14.0 就这么操作，其实很简单，熟悉的朋友可能就两三分钟搞定

IOS16.3
-------

先看下我设备相关参数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic8QAXNXTsfOBdF9cM23QgW2E0O2AMAgps42TuGEvM5tF3ich21YNgZzQ/640?wx_fmt=png&from=appmsg)

同样的用爱思里的工具，不过这里它自动匹配的是 dopamine 了，常规的操作就行了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicH3m9wKXbdUh8kzT9WRJ9ibNajOdvQIHwh1LcOiccNwPD94dpWraNetDQ/640?wx_fmt=png&from=appmsg)

不过它这里给的 dopamine 其实不是最新版，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicppGbUSvELgEEqESIOyezxdMyd3utSmZcPEPQCIhP4OOyV9ChzPrZUg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicpD1txkqC7Cn9KiaNdnAHopMicK0Zs6GR4XEX4aiaEf529XSCFoYdakYmA/640?wx_fmt=png&from=appmsg)

然后就搞定了，里面的设置你可以进去调试下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIickiaYbBHUS2OpLZd7fLKa886WjcMGxAJ9nzN9rO6llGuGMuSPmM3efuA/640?wx_fmt=png&from=appmsg)

然后点越狱就行了，在点之前，可能还得是科学操作

但是我照着这个操作，点完手机就自动重启了，重启了，越狱状态也没有，

反正就是失败了，怎么办？  

那说明爱思在这里失效了，得自己去找签名工具 (上面说过常识了) 安装，签名工具上面说了，你随便选一个就行了

这里我已经安装好签名工具（怎么安装下一篇讲吧，不然篇幅太长了），然后打开签名工具，登录好你的 apple ID

**其实 dopamine 官方推荐的安装方式是用巨魔安装**

所以如果你能安装上巨魔（怎么安装也放下一篇讲吧）的话，直接在巨魔安装就行了。

我这里既然已经安装好签名工具了，那我拿到 dopamine 直接在签名工具里安装了

```
https://github.com/opa334/Dopamine/releases/tag/2.2.2
https://ellekit.space/dopamine/

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicFl2sbNaC1iaDJyQRCt0DA28hnlrkdtUtfT3mh0IXvAGNPspo2Rg9OYQ/640?wx_fmt=png&from=appmsg)

下载这个 ipa 就行了，那个 tipa 就是巨魔专用版  

下载好了后，想办法导入到手机上  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicqLE1VH0ibCE8t3WB5ibZ37Qk72LxEIHMCEGDLzzW5A2API1FAUBgc3Qw/640?wx_fmt=png&from=appmsg)

但是苹果手机不像安卓那么自由，能用 adb 就行，用爱思里面的文件也不太好，你放进去，在手机上根本找不到在哪，文件管理我感觉就是一坨 x

所以你要嘛用微信传输，如果你是 macos，可以用隔空投送

或者打开科学，在手机上用 safari 打开下载，这样才能到手机上，然后你还能找到在哪

我选用直接用 safari 下载  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIickoq0QWsDH78peUsenEFLhJicDxj5zLBr9CtAdpWPObBNbSdtRdeffWQ/640?wx_fmt=png&from=appmsg)

完了打开这个

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic4c0dbKiaxZ4MibSMmx6rRj8Zwzc8UzHD1BfyeT0xUPa1t8YeMTBwf7eA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicVEaod1VIHPicUk3BT3K254I7nDsCxNCPKBiclt71aE51FOnnJY6Z8uiaA/640?wx_fmt=png&from=appmsg)

找到它，点下  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicm8H6N2XWicOdXgnqRKch4quh42gyiaT4nR0ICCFtl0icM03TSbHVnuyUg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic2ChI91ZkQLolYNrXGCWNCYLRLhLtfbwHdNJE5J6eziaDNeibTYMMrngQ/640?wx_fmt=png&from=appmsg)

往后翻，找到你的签名工具  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicTlia5fvGz92Mf0oCsImqHUn9Ew7SCNEBiagtUA9W8GjQAvG9MRUd7CGQ/640?wx_fmt=png&from=appmsg)

点下签名工具，就自动导入了  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicibtRLTjqpwf7FHZUyZc6piaDRMuibJysUMhHianaTHyUYCfWqG0azwArAg/640?wx_fmt=png&from=appmsg)

然后就可以签名、安装了，后续都是常规操作了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicCbnpqomWPiao2AfPFKyX6TaS69jV6c8Fb03Ko4mkQBhibzIYKG1McDBg/640?wx_fmt=png&from=appmsg)

这些都是每个签名工具自带的功能  

操作完，静等结束，然后去设置里，点下已验证

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic4ibeMSdUo5NCGtt7EnBibVVGMerqJicaUjq81nhj4D0ibds5MaJf5MZJKA/640?wx_fmt=png&from=appmsg)

现在就有了，上面这个是爱思安装的，下面是签名工具安装的  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic07EEdwOiaZeuP1ud2m0YAHs5y2dcO5TeCFSTVLiaPHiadJSaycWeJ9AWg/640?wx_fmt=png&from=appmsg)

打开 app 继续操作，点越狱

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic1j6zO3KjkwDF4qjicRaIeLXFabDM5a7qEAHgibmiaUj8mkYeLfWj5fMNg/640?wx_fmt=png&from=appmsg)

把这两个都勾上，继续

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicjqPAY2qjfbhJAkZG9bJxWDMo7SnvUZiaPreFLyoeibk94Qf5BvjbjicFg/640?wx_fmt=png&from=appmsg)

如果你们之前没有安装过的，就结束了，如果安装过其他的，就可能会遇到我这个错误，问题不大，把之前的卸载清理干净重来  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicKC4t5c7TUqGbtcjXQ6OEttEEGy0Ktm8ibkxFxZibjAkEBXVOArSbahibw/640?wx_fmt=png&from=appmsg)

**只要设备没事，能正常开机，都是小事，我买的是支持还原刷机的，所以只要能开机的，都是小问题**

卸载完了，再重新操作，如果还不行，签名的时候可能得改下 bundleID(类似安卓里的包名)，因为之前的安装环境可能有同 bundle iD 冲突  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicsuSicmTHOfPx48UV6eLkTxAR8Me4n3m9HNy9f6kK0RfcRKicjBiaibJU0g/640?wx_fmt=png&from=appmsg)

再次操作，设置下 root 密码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic53KuydhWGq0r2jJWQgs7icHMicLuycGP7IX8Zt3SSIPCWBLnjFa6JianQ/640?wx_fmt=png&from=appmsg)

这里随便你咋设置，只要能记住，我习惯还是 root 或者 alpine

点【set】，手机自动重启，重启后就显示已越狱了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic220Faf6BcOQEqC7xKk7zkLGpZArHqgh3z6F6027xHbME7C0uzpLicSg/640?wx_fmt=png&from=appmsg)

再次打开 dopamine 显示如下：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicC4pbHdM137rUQcND6wu6Ct0YxibM35yeGGWcG5s8WxZEWGicmHicud6RA/640?wx_fmt=png&from=appmsg)

然后这里，就不再是 cydia 作为安装插件的 app 了，而是 Sileo 或者 Zebra

我习惯用 Sileo，其实 cydia 和 Sileo，Zebra 本质没啥区别，就是工具的不一样而已，然后有些软件源只有 cydia 可以用而已，这都是小问题

同样的，安装完 frida，安装流程省略，跟上面的一样啊，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIic3FjE0dQtfWic6c0lwyZSL4fwMsxV3MvHEX17mrvSibJAT3JQoibGqjR7g/640?wx_fmt=png&from=appmsg)

电脑操作测试试试，frida-ps -Ua  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/l4m5icTfxSXUZKesiaXibnKbvgr20RictjIicPibK2ibG1Vm7QlCfI4qLMv8BvkcYPRic5Ls0OqyIicgiaiaLpLH0zdiaVibK6w/640?wx_fmt=png&from=appmsg)

ok，能搞，over  

卧槽，一不小心，又有 3 千多个字了，那 16.7.8 的放下一篇吧，要不然这篇文章又好长  

结语
--

其实操作起来很简单的，多尝试，多测试，多研究，就会了