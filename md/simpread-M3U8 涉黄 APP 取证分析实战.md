> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ca0LQUWQPSv3Wf-ZieCoWQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/4ic1PjqhJrM4GvWLY0rHv4ZPxPQZoaGYJLFofPBr2cGDrKZk8jbAUvxy4qh02G0r7O4GoBptiaM6bDYWoeJUibl2A/640?wx_fmt=gif)

**1. 引言**

随着移动互联网技术和网络经济的迅捷发展，越来越多的网络犯罪活动已经由线下转移到线上实施，移动智能终端的普及, 又开始涌现出大量钓鱼、诈骗、赌博、色情等各种类型的应用 APP，利用这类应用 APP 实施电信诈骗、网络传销、网络赌博、黑灰产、套路贷、暗网交易等，可见移动应用程序作为新型犯罪的主要手段，取证民警在获取到涉案 AP 后，应该如何从中获取到关键侦办线索，本文以笔者办案过程中，遇到的 M3U8 涉黄视频 APP 为例，通过 M3U8 格式文件的概念切入，使用 APP 取证通常的手段，即动态抓包、静态动态逆向分析、Python 代码模拟客户端与涉案服务器通信，从而快速获取相关证据数据信息。

**关键词：**安卓 APP 取证、手机取证、m3u8 视频、m3u8 解密、APP 逆向分析

**2. M3U8 文件是什么**

HTTP 实时流（也称为 HLS）是一种基于 HTTP 的媒体流通信协议，由 Apple Inc. 制定。它的工作原理是将整个流分解为一系列基于 HTTP 的小文件下载，每次下载都加载一个分片。当播放流时，客户端可以从多个不同备选流中选择，这些备选流包含不同编码速率的码流，从而允许流会话适应可用的数据速率。在流会话开始时，HLS 下载一个扩展的 M3U(m3u8) PlayList 文件，其中包含各种可用子流的元数据。M3U8 视频格格式也是一种 M3U，只是它的编码格式是 UTF-8 格式，M3U 用 Latin-1 字符集编码。M3U8 格式特点是带有一个目录信息或文件。M3U8 文件实质是一个播放列表（playlist）。

HLS 是如何对视频文件进行加密的，一般步骤为用户将本地视频 (未加密) 上传到服务器，服务器使用 ffmpeg(ffmpeg 是一个免费的开源程序库，一个命令行工具软件，专门用来编辑处理各种音视频或图像) 对视频进行切片处理，将生成的 m3u8 播放列表文件存储在数据库，后端编写接口将 m3u8 文件以自定义加密的形式传递给客户端，客户端使用 key 文件解密后放入播放器进行播放。加解密处理流程图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSKBlfxJg1qI7sAIClZyVfnRuDFEKgoQK4SekAyRSnCQ1OcAUJviaocJA/640?wx_fmt=png)

图 1 m3u8 视频加解密过程

**2.1 m3u8 加密过程**

HLS 允许视频流加密和不加密，针对不加密的 HLS，那么生成的 m3u8 索引文件中的视频片段，可以直接播放，仅仅是对原始视频切片成多个小视频片段而已，常见的有腾讯视频、爱奇艺视频等均是采用未加密切片成 m3u8 索引播放列表，如果要下载这类视频，只需要抓包找到 m3u8 文件即可下载所有视频片段，在本地合成后及可与原视频一样播放。而加密视频的方式通常有三种 aes-128, sample-aes, drm 等。不同的加密方式原理不一样，生成的方式也不一样，目前，市场上几乎清一色的采用 aes-128 加密方式生成视频片段，比如腾讯课堂、慕课网等等均采用 aes-128 加密方式。而该加密方式的原理为加密是对分片进行整个加密，加密得到的分片流是检查不出媒体信息的，即相当于将整个分片文件进行加密，不区分媒体头之类的定义。对视频采用 aes-128 加密方法实操过程如下：

1、准备加密密钥：随机生成一个 16 字节（128 位）的数据作为密钥，保存到 enc.key 中，加密的 key 的名字可以随意命名，key 的关键是 16 个字节的密文。笔者使用 openssl 命令生成的 key 及以 16 进制查看 key 的内容如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSdYOmgvC1nZKzCUxLpxOibR4Z7CjHFpfvySC2mlQmT1KNeEck6TFtxGw/640?wx_fmt=png)

图 2 生成 16 进制 key 文件

2、生成 IV（Initialization Vector 可选）：在实际解密过程中，该值一般固定在 m3u8 文件中，解密过程无需关注该值。

3、创建 enc.info 文件：创建一个文本文件来记录 key 的信息，文件名可以起随意名字，内容由两部分组成如下：Key URI 以及 IV (optional)。其中，Key URI: 指的 key 的位置，即放置 enc.key 的路径，一般为服务器的路径，这个路径会在 m3u8 里。这个 url 在测试时，需要填写正确的路径，本地测试时，可以不用服务器的路径。IV (optional)：可选，将之前生成的值填到这里，一个完整的 enc.info 内容如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeS1BlCOh2twzIkV1PSWMmoPcSVRV1CTgiagbRZHfSR8mELzSYxgqXpngw/640?wx_fmt=png)

图 3 enc.info 文件

4、使用 ffmpeg 命令对视频进行加密切片，具体指令含义如下。

ffmpeg -y  -i test.mp4  -hls_time 9  -hls_key_info_file enc.info  -hls_playlist_type vod  -hls_segment_filename "index%d.ts" playlist.m3u8

•  -i：输入视频

•  -hls_time: 分成几个分片，这里是 9

•  -hls_key_info_file：加密信息文件

•  -hls_playlist_type：这个是可选的，有 VOD, EVENT, LIVE, 点播一般是 vod

•  -hls_segment_filename：分片名字

命令执行后，输出结果为 playlist.m3u8，可以是其他名字，操作如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeStAyCA5w2ZAeXib9xjsCGNjlgGZf8ZSxp5LVibl9wjmQM4R2NIjbI6azg/640?wx_fmt=png)

图 4 加密后生成 mu38 文件及 ts 加密流文件

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSfVW52jWh8EV3ftRSxBZjtibdBva2LZD5FWnxnODESahwddXibhmhQImg/640?wx_fmt=png)

图 5 m3u8 文件内幕

对于 m3u8 文件， 需要注意，文件中的 key 前面如果没有网址，那么就相对地址，一般是和 m3u8 同样的地址，除了文件名不一样，前面的网址是一样的，m3u8 文件中的视频流 ts 文件，没有前面没有网址，同样是相对地址，在解密处理相对地址时，记得添加基础网址即可。

**2.2 m3u8 解密过程**

通过实操一遍 m3u8 文件加密过程，得知一个未加密视频，使用 ffmpeg 工具传入 16 进制大小的 key 文件进行加密以后，生成了 m3u8 文件播放列表，以及若干个 ts 流视频文件，这些 ts 视频片段是整体加密的，本地根本无法播放，而且 m3u8 播放列表中，包括了 key 的路径以及 ts 视频片段的路径，只要下载到 m3u8 文件播放列表文件，即可在 m3u8 文件的内容中看到 key 文件的路径以及 ts 流视频片段的路径。

那么对于 m3u8 的解密，只需要拿个两个文件，即 m3u8 文件播放列表和 16 进制的 ke 文件，然后通过第三方开源工具进行下载并解密合并 ts 视频流片段即可。常见的第三方 m3u8 解密下载工具有开源的 N_m3u8DL-CLI 或者吾爱破解论坛中用户逍遥一仙的 M3U8 批量下载器。使用编程语 python 的话，也有第三方的开源包 m3u8download_hecoter。

所以，解密的关键为一通过抓包获取到 m3u8 文件，二通过抓包获取 key 文件，或者在 APP 终端通过逆向调试，找到 key 的解密算法，从而得到 16 进制的 key 字符串。比如腾讯课堂的 key 即是在用户的浏览器端通过 JavaScript 计算得出的一个 16 进制大小的字符串，m3u8 解密的流程图如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSQ63d1U9iaAeSrficBtHWt0KzfjQBRKlgN4OvQY7SctN32YAQibuia3yyZA/640?wx_fmt=png)

图 6 m3u8 解密过程

**3.  APP 取证流程**

拿到一个涉黄视频类 APP 以后，首先计算它的 hash 值，然后以副本进行分析，一般是对 APP 程序进行抓包，APP 的运行一般使用雷电等模拟器、有条件的使用真机环境，抓包可以使用 fiddler 或者 httpcanary(黄鸟)，fiddler 是电脑端进行代理抓包，而 httpcanary(黄鸟) 本身就是安卓 apk 程序，直接在模拟器或手机端进行安装抓包查看。

在实际工作中，建议两个抓包工具都使用一下，然后对抓包数据进行对比分析，防止仅仅使用一个工具出现漏包的情况。对抓到的域名数据包进行 IP 地址核实，视频类 APP 一般关注登录数据包，确实登录服务器 IP，支付数据包，确定支付调用接口，视频源数据包，确定视频服务器 IP，如果 IP 位于国内的及时进行采取调证措施。

抓包分析以后，大致明白了该 APP 有哪些功能以及出现了哪些数据包的关键字，然后使用逆向工具 JEB、GDA 对关键字进行搜索逆向，获取关键数据的加密算法，从而使用 Python 等编程语言与服务器通信，并得到服务器的拦截认可，从而实现批量下载固定 M3U8 视频，并解密到本地。M3U8 涉黄视频类 APP 取证分析流程如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSrU2pKdPTQ3GgmpibdQHUnYrwNSDzMV2m5tPj4iaLyZESP8PiblH3fesjQ/640?wx_fmt=png)

图 7 APP 取证一般流程

**4. APP 抓包分析**

**4.1 抓包数据分析**

将涉案 APP 安装到雷电模拟器或者真机 (最好是 ROOT 版) 中，然后配置好代理，笔者一般先使用 Fiddler 进行抓包，后再使用 httpcanary(黄鸟)进行抓包，一般关注涉案 APP 的首页数据包、登录数据包、视频流数据包、支付数据包，对这几类数据包进行分析，查看每种的域名是否一样，IP 服务器是否为同一个，然后再多点击几个页面，分析域名及 IP 服务器是否有变化，确认后，对涉案的 IP 服务器进行调证操作。下面以 httpcanary(黄鸟)抓包数据为例，部分关键数据包截图如下。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeS8oXs5DjwKfD20cbYVGWmic0LribXibcCF5jJeHDpiaslBycRK7Fdl5TjYg/640?wx_fmt=png)

图 8 登录及首页域名 IP

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSvqjy2XbP9rFxoebG8xAbEDle5KHLzyicxPM5xttib0yQFwdZpicHCu4Rg/640?wx_fmt=png)

图 9 视频流服务器

因为服务器信息作为 APP 破案的核心资源，一直是案件办理的核心要点。通过域名，我们可以获取到服务器的 IP，进而获取服务器物理位置。同时，域名涉及的注册人信息、DSN 信息、CDN 信息、甚至后端管理平台都可通过相应的服务商调取业务委托人的相关信息，从而形成有价值的线索。下面是笔者在实际工作中，常用的网络查询网站，对于抓到的所有域名及 IP，都需要经过表 1 中的六个网址进行进一步查询核实，从而更好的掌握破案关键线索。

表 1 常用查询网址

<table><tbody><tr><td><p>网址名称</p></td><td><p>网址链接地址</p></td></tr><tr><td><p>站长之家</p></td><td><p>https://tool.chinaz.com</p></td></tr><tr><td><p>whois 查询</p></td><td><p>https://whois.chinaz.com</p></td></tr><tr><td><p>IP 地址查询</p></td><td><p>https://ip138.com</p></td></tr><tr><td><p>SOJSON</p></td><td><p>https://tool.chinaz.com</p></td></tr><tr><td><p>端口查询</p></td><td><p>http://coolaf.com/tool/port</p></td></tr><tr><td><p>工信部备案信息查询</p></td><td><p>ttps://beian.miit.gov.cn/#/Integrated/recordQuery</p></td></tr></tbody></table>

通过上述六个网址查询核实后，及时将相关线索交由案侦民警开展调证工作。取证民警仍需进一步深入分析，上面为了截图更为直观，笔者使用了 httpcanary(黄鸟) 来展示，在实际工作中，fiddler 也是非常强大的代理抓包工具，而且该工具本身就在电脑端，方便编写 Python 脚本与服务器进行通信，因为该涉案 APP 为 M3U8 视频类的 APP，那么最好能将该 APP 的涉黄视频固定到本地，固定的方法一般由两种，第一为传统方式，即录屏，然后点击视频播放，这种方法处理速度较慢，费时费力又费电，不推荐使用。那么第二种方式就是用 Python 脚本模拟 APP 的身份与服务器通信，从而批量固定涉案视频文件到本地。

通过 fiddler 抓包发现，涉案 APP 打开以后，会首选发送一个 checkAPi 的数据包到服务器，而服务器返回了一 Json 消息数据，并且有个字段 msg=ok，该数据包详情如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSjzhv6b9fl3UzwTGa09fYLwdUgqPZiaGAB9movCJ68LrIGTpO6daYgcg/640?wx_fmt=png)

图 10 服务器验证身份数据包

通过数据包的名字，大概猜测是验证客户端身份证的一条数据包，在本地模拟该数据包发送给服务器，得到的消息为 "非法请求" ，服务器的成功的识别出该数据包不是涉案 APP 发出的。Python 发送数据包的语句及结果如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSYJ2v64ib67LnaIZjglG3yyS3OmH82pR9xWkia8811zicaOhYkB0WunGiaw/640?wx_fmt=png)

图 11 Python 发送数据包请求

服务器只关心接受的数据包信息，通过对接受的数据包信息进行计算判断是否为真正的涉案 APP 发送的请求，笔者通过多次抓包，多次收集该 checkAPi 的数据包字段信息，从而找出不同点如表 2 所示。

表 2 数据包字段解读

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSE5E1EoML6C12FWuicSaQOClSRGfKYe2Tb9xoiaajGFpe944nrIib15zng/640?wx_fmt=png)

**5. APP 逆向分析**

对于涉案 APP 的逆向，可以使用 JEB、GDA 等逆向工具打开，首先查看该 APP 的 AndroidManifest.xml 文件，该文件记录了 APP 的包名，使用的 android 系统版本信息，描述 APP 本身的版本信息，描述应用对外暴露的组件、以及使用了哪家的消息推送 API 等等。对于消息推送的 API，也是调证的一个重点关注信息，用于查询该 APP 的开发者在消息推送厂商的注册信息。

**5.1 加壳检测及脱壳**

通过查壳工具查看，发现并未识别到加固行为。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSH4OJibZUOJlMvFN83bhTWRXhmOjNia3Ul5Wl5SXVZ1cK2AguljXyF0EQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeShLNC1Ohdur4ebqFRoocp0YJicjr3gAfb5S8JMEsc5G29xl29SNms8uA/640?wx_fmt=png)

常见的加固特征都是去识别 lib 目录下的 so 文件，主要是识别文件特征。但是查壳能力取决于加固文件特征的完善程度，有的加固比较小众，市面的工具可能都无法识别。

实际上分析代码时，无法定位到程序入口其实程序就应该是使用了未知的加固方式，其实判断一个程序是够加固，有一个很好的判断方法。就是看能否定位程序入口。这个最准确。接下来以反编译 JEB 工具为例，先反编译一个不加固的程序。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSyrzaibaKQwf452QFXxicvfMibeNES0w1NiaECicyHjNFlpkSc3FT3vN5ImA/640?wx_fmt=png)

#!MainActivity:com.ljhhr.mobile.ui.login.splash.SplashActivity (SplashActivity)

这个 Main Activity 其实就是程序入口。或者叫做程序启动页, 主界面等等。没有加固的我们直接点击图中蓝色的 (SplashActivity)，就会跳转过去。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeS9qOF9BBu813cDedAHF7alfJyQf3kPl2LssRdb2qUSrAmeicZdWhoaaA/640?wx_fmt=png)

只要能正常调转的说明没有加固。再试试目标程序。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeS1IE59KQCicgobZWQufLeBm23ItksGtmAlVFvf4ykJeyxf9ic0SNjicQaA/640?wx_fmt=png)

首先左下部分可以看出来代码很少，其实中间那块区域也没有发现 Main Activity，实际上程序入口是去查看 Manifest, 当反编译工具无法直接识别到这个主界面的时候，其实就可以判断程序一定加固了。发现了这个加固行为，就去尝试脱壳。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSiavPYdcUaIQobguPPyib4DmIMYRmSzsfymKKFYOtQQlVcJShLV4Von1A/640?wx_fmt=png)

这里使用 frida 配合 dexdump.js 来进行脱壳，首先运行 frida-server。

这里我已经重命名了 frida-server 为 fs，并且赋予了 777 的权限，然后运行

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeShiaFaxdybtFxlQ1fXebicNcvbCgngkTQ6bics5COvtWuEbACjYSOIKEAw/640?wx_fmt=png)

输入 frida 指令来完成脱壳 frida -Uf sk8bo.rlr1q9.e1na5g --no-pause -l dexDump.js

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSLwyhjoSD7TTo6cl8k0LG4cxKvIXB2HQHyeDPibCqgHmgiaChhO2tYBvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSmbW5ZNoAyxSnQicKCcEMj1IvMRN9V7Xm7YsrnrDAPLLsN8Gu7k8NLdQ/640?wx_fmt=png)

然后用 adb pull /data/data/sk8bo.rlr1q9.e1na5g/8664560.dex \ 本地路径

这里因为这个 8664560.dex 出现多次，我们就优先分析它就好，实际上我们也可以把所有 dump 下来的 dex 一起分析，但是因为测试过目标代码在 8664560.dex 中，这里就不一一 pull 出来了。脱壳使用的方法蛮多的，只需要掌握任意一种能 dump 出内存中的 dex 就行，不限于 frida。

**5.2 核心算法解密**

通过前面抓包及数据包分析得知，要想使用 Python 脚本与服务器通信，需要获得服务器的信任认可，认可数据包的关键字段为 noise 和 sign，这两个字段是如何计算出来的。使用 JEB 3.19 对脱壳的 dex 文件进行逆向分析，既然知道了要分析的字段名称，并且为 http 请求，可以搜索 “sign=”、"noise"、"http" 等关键字，或者根据每次抓包发现 Referer 的值都为 "www.hahaha.hehe" ，也可以作为逆向的关键词搜索，关键操作步骤如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSHH2h9vrJ11hgNTUfx8NPf2nZ5s4nqE5r9zxwV2ba4vdVd8Sjqdlzug/640?wx_fmt=png)

图 14 字符串关键词搜索

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSgJEKHgSLSsiaorD6tv4eJ1KiaEsHCIaHw9ITV2CNxwUND7YkcBp2tLOg/640?wx_fmt=png)

图 15 sign 算法

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSbam9QwibTMbaF9kB773JrnDsVHTDnaxmHNAzWicOm7HTNHbkywjWB7Aw/640?wx_fmt=png)

图 16 sign 算法

通过上面的分析，得知 sign 为加盐后的 md5 值，noise 为 16 位的随机数，既然是随机数那么写死也属于随机数，而 sign 传递的参数为 token、timestamp、secret、uid、noise 的字符串值拼接后再拼接固定值 "JauE^Xl@6upwx1Ov4SCPLOW=8w4J0naI" 后再计算 MD5 即可，使用 Python 实现 sign 的算法后，传入图 10 中的值，计算出 sign 与 图 10 的 sign 值一样，从而确定了 sign 字段的计算逻辑。如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSXGF8exP11uiaXsHkbCqgUH65OwqLiclUUA9gJdo46Wicia0jIzCq3n4aXg/640?wx_fmt=png)

图 17 字段 sign 的算法

实际上 sign 的还原还有一个小困难，这个困难要看反编译工具。上图的代码来自 jeb 3.19.1。如果使用的是别的版本会得到下图的代码。这里会发现字符串被混淆了。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSYTGN4UQPtO9wVQAKIYeLYJZibzzSj9eqxLL08ktyonYEJ4ars2vSfOw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSPbnO7duicFZcsvdjSKMa75xdfia6PMSgp8R80nXK3pScOCoD1ia8XTDAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSAp8DPvHz4scVib1Unu4JepNRibpicic6rUtK4gA3IuEmF5Pibpy9b2zgIYw/640?wx_fmt=png)

fuu6zl.vs420[] 这个方法我们扣出来调用一下。我们处理一下 bHZ4cQ。方法名我改成了 a，直接调用运行，发现解密后是 sign。看着很像是 base64 编码，但是我尝试了用 base64 解码，发现不是标准的。所以这个方法应该是混淆字符串的。所以工具其实也很影响分析代码，建议分析的时候多装几个版本的工具。目前发现 JEB 3.19.1 还原能力应该在 JEB 中最强大。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSD1Kqve9tPGQuVKMYMYt9ONibLUS0ic9Y3icDHVBbKniaUWXibiavafQpPUuA/640?wx_fmt=png)

**6. 批量固证解密 M3U8**

解决了字段 sign 以后，Python 脚本已经可以得到服务器的信任，进行通信。继续分析 m3u8 的数据包，发现当点击一个具体视频时，会向服务器的 "/index/Get/videoInfo" 这个 API 发送请求，在服务器返回的响应数据包中，包含了该视频的 m3u8 链接，涉案 APP 接受到该 m3u8 链接后，会继续发送 Get 请求该链接，服务器会继续返回 m3u8 文件的内容，里面包含了相对地址的 key 文件以及相对地址的 ts 视频流文件，涉案 APP 接受到 key 文件以后，使用 key 文件和 m3u8 文件对后续下载的 ts 视频流文件进行解密，从而可以在涉案 APP 的视频窗口观看解密后的 ts 视频流文件。m3u8 视频响应数据包如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeS4NTyB9u6w4wTzWam5aFKFDia1dEOIFQ7DXNQdUQyXkN8YUiaRMrLjQDw/640?wx_fmt=png)

图 18  m3u8 数据包来源

通过观察 m3u8 数据包的来源，得知每个视频有唯一的 id 字段，比如上图中的视频的 id 为 23310，而通过首页的相应数据包可知，每个专题有专题的唯一 id，点击专题，那么下面的所有视频的 id 都会展示出来，那么只要会下载一个视频到本地，外层循环加上专题的 id 就是批量下载该专题的所有视频文件到本地。如果在首页的相应数据中加上循环即可下载该涉案 APP 的全部视频文件到本地。

所以，批量下载的关键是能成功下载一个视频，下面以上图的 id 为 23310 为例下载该并解密合并该视频流文件集到本地。单个视频下载的全部 Python 脚本代码如下，视频下载过程和下载后查看如下图所示。如果需要批量固证，在下面的代码基础上，加上两层遍历循环即可。

```
import requests
import time, os
import hashlib
import base64
from m3u8download_hecoter import m3u8download
token = ''
def now_to_timestamp(digits=13):
    time_stamp = time.time()
    digits = 10**(digits - 10)
    time_stamp = int(round(time_stamp * digits))
    return str(time_stamp)
def get_sign_md5(token, timestamp, secret, uid, noise):  #secret：1非vip
    md5str = token + timestamp + secret + uid + noise[8:] + "JauE^Xl@6upwx1Ov4SCPLOW=8w4J0naI"
    md5res = hashlib.md5(md5str.encode("utf-8"))
    return md5res.hexdigest()
def http_Login(token):
    headers = {
        'User-Agent': 'android-aoteman',
        'and_ver': '31',
        'Referer': 'https://www.hahaha.hehe/',
        'Content-Type': 'application/x-www-form-urlencoded; charset=utf-8',
        'Connection': 'Keep-Alive',
        'Accept-Encoding': 'gzip',
    }
    timestamp = now_to_timestamp()
    secret = '1'
    uid = '0'
    noise = "Z9603VS2P14ETLAA"
    sign = get_sign_md5(token, timestamp, secret, uid, noise)
    data = "sign=" + sign + "&client_type=android&secret=" + secret + "&time=" + timestamp + "&token=" + token + "&other_info=%7B%22agent_code%22%3A2369%7D&device_code=ffffffff-da92-eed5-0033-c5870033c587&and_ver=31&uid=" + uid + "&noise=" + noise + "&"
    url = 'http://api.tangnvshi.com/index/Login/doLogin111'
    token = requests.post(url, data=data, headers=headers).json()['token']
    return token
#根据视频ID，获取视频的M3U8链接
def get_videoInfo(token, video_id):
    headers = {
        'User-Agent': 'android-aoteman',
        'and_ver': '31',
        'Referer': 'https://www.hahaha.hehe/',
        'Content-Type': 'application/x-www-form-urlencoded; charset=utf-8',
        'Connection': 'Keep-Alive',
        'Accept-Encoding': 'gzip',
    }
    timestamp = now_to_timestamp()
    secret = '1'
    uid = '4405531'
    noise = "UA9J789K872Q5C9K"
    sign = get_sign_md5(token, timestamp, secret, uid, noise)
    data = "sign=" + sign + "&secret=" + secret + "&time=" + now_to_timestamp() + "&token=" + token + "&id=" + video_id + "&and_ver=31&uid=" + uid + "&noise=" + noise + "&"
    #print("data:", data)
    url = 'http://api.tangnvshi.com/index/Get/videoInfo'
    m3u8url = requests.post(url, data=data, headers=headers).json()['data']['video']['link']
    return m3u8url
#下载 m3u8文件到本地
def down_m3u8(token, m3u8_url):
    timestamp = now_to_timestamp()
    isvip = '0'
    uid = '4405531'
    noise = "RNQ21F69F1698337"
    sign = get_sign_md5(token, timestamp, isvip, uid, noise)
    url_start = m3u8_url.rindex('/')
    base_url = m3u8_url[:url_start + 1]
    #print(base_url)
    headers = {
        'User-Agent': 'android-aoteman',
        'Accept': '*/*',
        'Range': 'bytes=0-',
        'Icy-MetaData': '1',
        'Referer': 'https://www.hahaha.hehe/',
        'sign': sign,
        'time': timestamp,
        'vip': '0',
        'token': token,
        'and_ver': '31',
        'uid': uid,
        'noise': noise
    }
    work_dir = "123"
    title = "test"
    m3u8_key = requests.get(url=base_url + 'v-crypt.key', headers=headers).content
    if not os.path.exists(work_dir):
        os.makedirs(work_dir)
    temp_dir = work_dir + '/' + title
    if not os.path.exists(temp_dir):
        os.makedirs(temp_dir)
    with open(f'{work_dir}/{title}/v-crypt.key', mode='ab') as f:
        f.write(m3u8_key)
    m3u8download(m3u8url=m3u8_url,
                 title=title,
                 base_uri_parse=base_url,
                 work_dir=work_dir,
                 threads=1,
                 key=base64.b64encode(m3u8_key).decode(),
                 headers=headers,
                 enable_del=False,
                 merge_mode=1)
# 模拟登录，获取到登录凭证 token数据
token = http_Login(token)
#获取视频的m3u8视频链接地址
m3u8_url = get_videoInfo(token, '23310')
#下载 M3U8 视频到本地
down_m3u8(token, m3u8_url)

```

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM76DO7JTW8Dw7WlnQ0CgAeSFmSfMTt6icuyribviaLbnwhv09vEb5Mw9wDmBa7Rd8CV1iclAI4FEqWpAQ/640?wx_fmt=png)

图 19 视频下载解密后

**7. 总结**

通过对 M3U8 涉黄 APP 案件的取证分析，笔者认为，在取证过程中，对于陌生的软件、程序不要有畏难心理。只要能了解安卓 APP 取证的一般流程，即抓包、逆向、Python 脚本模拟请求，就能一步一步按部就班分析逆向，最后编写脚本与服务器模拟通信，批量固定涉案视频数据。希望能通过该涉黄 APP 的固证实战，以此开阔取证思路，达到抛砖引玉的目的。

安全为先，洞鉴未来，奇安信盘古石取证团队竭诚为您提供电子数据取证专业的解决方案与服务。如需试用，请联系奇安信各区域销售代表，或致电 95015，期待您的来电！

  

  

  

  

  

“盘古石”团队是奇安信科技集团股份有限公司旗下专注于电子数据取证技术研发的团队，由来自国内最早从事电子数据取证的成员组成。盘古石团队以 “安全为先，洞鉴未来” 为使命，以 “漏洞思维” 解决电子数据取证难题，以 “数据驱动安全” 为技术思想，以安全赋能取证，研发新一代电子数据取证产品，产品涵盖计算机取证、移动终端取证、网络空间取证、IoT 取证、取证数据分析平台等电子数据取证全领域产品和解决方案，为包括公安执法、党政机关、司法机关以及行政执法部门等提供全面专业的支持与服务。

  

  

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/4ic1PjqhJrM6emb8EMkeRc3ejAY0AcpI0bVeCx8USkRy2oIv4Cy4usbG6q40GmWb38wkLlVKB19w95A7pJhvx0w/640?wx_fmt=png)