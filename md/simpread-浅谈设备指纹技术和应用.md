> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzUxODkyODE0Mg==&mid=2247489364&idx=1&sn=105b191c86eef427356768fbe192a9df&chksm=f9803535cef7bc23050f1e8bced3a0621f8ec158be662b0886bee2940a98203b98f9561dd7c4&mpshare=1&scene=1&srcid=0613UR9PRXZD3TqbB85OcLZr&sharer_sharetime=1655091039124&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

 ![](http://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7asExq0Evsib5pHoD24M70jEAcyibfbqhMc6X4ZXy9ezVsmbY7Fictyz3GM6C3V7mu0KrenKlv0wwmw/0?wx_fmt=png) ** 小道安全 ** 以安全开发、逆向破解、黑客技术、病毒技术、灰黑产攻防为基础，兼论程序研发相关的技术点滴分享。 41 篇原创内容  公众号

**背景**

当你手机 APP 上刷着某些视频并多停留几秒，后续再刷视频的时候，是否有感觉到更多是推送同类型的视频；

当你在某 APP 搜索某产品的时候，后面在启动 APP 时候，是否能感觉到更多给你推送相关的产品信息；

当你更换手机的时候，在登录同一个 APP 的时候，是否会有提示你是新环境登录需要特别验证才能登录；

当你手机上安装一些作弊软件，在进行支付的时候，是否会有提示你当前环境不安全不允许支付或支付不成功；

这些的背后都是依靠哪些技术进行支撑实现呢？这些场景下都离不开一个重要的**设备指纹技术**，下面就梳理设备指纹技术的细节。

**理论基础**

**设备指纹是指可以用于唯一标识出该设备的设备特征或者独特的设备标识。**

设备指纹围绕着设备的唯一 ID、设备环境风险特征、设备历史风险标签等维度对设备进行全方位刻画，识别设备风险、并且透传设备风险特征。

设备指纹在同一设备中的不同应用，必须具备设备 ID 不变，同一设备卸载重装 APP 应用，设备 ID 同样要保持不变，在 IOS 设备中重置 IDFA 后，设备 ID 不变改机软件修改属性后，设备保持 ID 不变。

设备指纹需要考虑设备指纹唯一标识的稳定性、唯一标识的唯一性、设备风险标签的精准度、设备风险标签的准召率、设备指纹所需的隐私权限、微行为无感识别能力、设备终端覆盖识别。

**设备指纹的关键用途：设备信息唯一性、渠道流量检测、风险设备识别、通用风控策略。**

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpZKuuDlw3ib82HqK2crdRojm3uZUEENtSpwAQib8SAkdTEkqYkpbwmRnw/640?wx_fmt=png)

（上图来源网络）

**技术分析**

采集设备信息需要关注的问题：用户设备是真实设备？哪个设备信息是稳定？Android 系统大版本升级是否导致权限变动？采集的数据是否符合隐私合规政策？采用什么算法来计算出唯一 ID？新 APP 上线所有设备 ID 是全新？

**技术实现流程：通过采集客户端的特征属性信息并将其加密上传到云端，然后通过特定的算法分析并为每台设备生成唯一的 ID 来标识这台设备。**

**设备指纹必须具备**：稳定性、唯一性、安全性、易用性、高性能。

**设备环境风险特征**：识别模拟器环境、多开、ROOT、篡改设备参数、脚本，等异常环境特征。具有稳定性高，性能高。

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpWCdnnKErVSu3cJf0CEQm7fy91sGB2nprDWP4JurPz2icpeurggZfSDQ/640?wx_fmt=png)

通常情况下，设备指纹采集到用户的设备数据后，数据会通过**异步方式**先上传到业务的服务器上，然后再通过代理服务端进行转发到对应设备指纹的服务端。这样也是为了保证数据的安全性，客户端采集数据功能**防止被剥离**，从而采集不到设备数据。**采集的数据一般通过 json 格式加密数据进行上传。**

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpibZlONlqYXicUGYhbtwmStFtApQSa93F4Tf29JYlpzibekcUF7VdHHMzA/640?wx_fmt=png)

设备指纹上传一般采用 URL 的 POST 请求，并集成 json 格式，并且所采集的字段信息中会有一些字段是无用的，有一些字段适用于对 json 信息采用强校验的混淆信息。

**设备指纹中设备风险识别的微行为常用的属性**：电池状态、重力传感器、加速度传感器状态、联网状态、USB 状态、触摸轨迹、压感、按压时长、剪切板。

**设备指纹的 SDK 主要以 java 代码和 C、C++ 代码为主**，java 代码部分是以 aar 包或 jar 包方式存在，C\C++ 代码主要以 SO 方式存储的。例如某易的设备指纹就是以 aar 包 (NEDevice-SdkRelease_v1.7.0_2022xxxxxx.aar) 单独方式存在，某盾的设备指纹以 aar 文件（fraudmetrix-xxx.aar）和 so 文件（libtongdun.so）两者相结合存在，某美的设备指纹以 aar 包 (smsdk-x.x.x-release.aar) 和 so 文件（libsmsdk.so）相结合存在。

设备指纹主要是通过集成到 APP 中的 SDK，还有小程序的 SDK，通常情况下是采用 aar 包方式进行提供的 SDK，还有就是强度较高的是通过 aar 包和 SO 文件进行结合集成的设备指纹 SDK。将关键的采集信息集成到 SO 中代码中实现，并且 SO 文件采用虚拟机保护。

下面是某设备指纹以 aar 形式的，它关键代码都是 java 实现的。**并且 java 代码利用 proguard 混淆规则**进行对 aar 的 class 类进行混淆类名而已，并没有混淆到函数名称和变量名称，字符串信息。

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpKurUo7Dx5qs6nYs8vYxNL4D0ho1nVoPX9w3G8NJpbzsBmOPcDensmw/640?wx_fmt=png)

下面是某设备指纹的 java 代码和 C++ 代码部分，java 代码和 C++ 代码都采用了**虚拟化保护技术进行保护。**

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEplEvDV6DE6JPwDWKFBp6WRFEBeW3ZAuibqtg2fGdFOnFGaL2lTLTDjvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpkN653wGjqwMh5nM13nibI9eOJX1tXdhqks3BtE3eoZpMy0ZibhiaW1HLg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpv5s3QDIMibe6s8NOaukXibUXLataE47iaUMBJWcSZrl35RfomSCMiciaFpA/640?wx_fmt=png)

设备指纹读取用户信息，通常需要涉及到向用户申请权限的情况，所以**在 android 的 AndroidManifest.xml 配置文件**中通常有一系列的权限申请。

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEplwxGPUnUF6otQF1WFss5jedJ7Aic1OdoSiaTZkXePL4ianRsDJJZzR1Zg/640?wx_fmt=png)

（上图只是申请权限的一小部分）

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpYEzZ030qubOrib5F0SK3dOvEQ0PMRGZXxFt2TNlmp4zZEb2K79pdXNg/640?wx_fmt=png)

**设备指纹合规**

设备指纹应用中，在采集用户设备指纹信息的过程，首先必须确保用户 APP 中有**《用户隐私政策》**，并且在首次启动 APP 时就弹出《用户隐私政策》获得用户的同意，不得默认用户已勾选。并且确保只有用户同意的时候才可以进行对用户信息的采集。

根据**《网络安全法》**等相关法律法规要求，APP 应当在隐私政策中向最终用户告知收集、使用、于第三方共享最终用户个人信息的目的、方式和范围，并征得最终用户明示同意才可以采集用户信息。

设备信息采集需要遵循必要最小化低频采集非敏感信息原则，在采集用户属性时候，不应采集用户行为和应用列表、传感器状态、通讯录、相册等敏感信息，采集这些属性容易出现不符合隐私合规政策。支持按需采集和合规上架指导，采集信息 合规和安全加固，不触碰用户隐私，不会被黑产破解，兼容性好。

设备指纹 SDK 的初始化时机，在安装后首次启动时，并且只有在用户同意隐私协议后，才进行设备指纹 SDK 初始化。**如果用户没有同意隐私协议不可进行采集数据。**

**风控场景分析**

设备指纹的在游戏应用场景中主要风控维度：手机设备、游戏账号、账号行为、账号动机。

设备指纹识别设备的风控特征：模拟器、协议刷数据、脚本外挂、设备改机、多开工具、云手机等。

设备指纹识别游戏账号的风控特征：批量注册游戏账号、猫池、接码手机、通讯小号等等。

设备指纹识别游戏行为的风控特征：批量养号、黄牛养号、鱼塘账号。

设备指纹识别账号动机的风控特征：账号的买卖、游戏的代练等

设备指纹识别风险识别重点在于：注册、登录、营销、交易、充值、渠道推广等业务场景中，识别出虚假注册、盗号、养号、薅羊毛、虚假推广、作弊行为，然后对应采取一定的对抗策略方案。

游戏中黑灰产破解移动端的技术及工具不断在更新变化发展，设备指纹中核心的技术攻防点主要围绕，root（非法读取文件，反安全检测）、自动化工具（批量注册、活动作弊）、模拟器（自动注册小号、秒杀）、多开（虚假作弊、养号、）、改机、群控（薅羊毛、虚假流量），app 重打包（植入广告、破解功能限制）等问题。

**通过基于设备指纹技术，可以实现 IP 风险画像、风险情报、邮箱风险画像去识别游戏的黑灰产行为，然后对游戏黑灰产进行重点打击**。

![](https://mmbiz.qpic.cn/mmbiz_png/jVCRndy8Lr7jracFzJVIKFlR5eXB7TEpcWRbzCkB3PcDKX2V1gcf17ltdxuRrnfZFu37oPYpHZG3IRAVT8NbjA/640?wx_fmt=png)

（上图来源网络）

**设备指纹思考**

一个人常用设备的总是有限，一般正常情况下一段时间内**不会超过 5 个**，因此可以通过这些信息进行作为风控的策略，而设备指纹中关键的一个采集点是**网络相关信息的采集**，通过采集网络相关信息，可以判断出同一网络下的用户的设备数量。

一个好的设备指纹必须具备：**1. 可以灵活定制化**，可以根据不同的业务需求提供灵活的接口；**2. 高准确性**，对设备唯一码必须准确率足够高；**3. 性能卓越**，设备指纹的 sdk 不能影响到 APP 的性能；**4. 系统兼容性好**，需要兼容到 android4.0 到 android 13 的所有系统。

如果作为开发者，开发一个设备指纹 sdk 中需要综合考虑的：LaunchTime、闪退率、体积大小、网络消耗情况。

设备指纹技术存在一定的被动性。黑灰产研发者处在暗处，公司的业务在明处，在什么时间段，采用什么方式的攻击方式，都是黑灰产攻击者决定的，这一点业务安全人员也是非常头疼。更进一步，**黑灰产可以不断试错通过分析设备指纹技术采用的算法**，尝试采用新的攻击方式绕过已有加解密算法，就可以达到新的攻击目的。**设备指纹技术对抗过程中是存在一定的滞后性。**

  

  

结束

  

  

**【推荐阅读】**

[**APP 应用安全检测**](http://mp.weixin.qq.com/s?__biz=MzUxODkyODE0Mg==&mid=2247489000&idx=1&sn=a9ff1413307a7d20691ea0ca1696f20b&chksm=f9803789cef7be9f8b7617c063eefaa0ecd1bffb43de85a66fce7c453eb17b8c7b37edf1fa51&scene=21#wechat_redirect)

[**App 安全测试**](http://mp.weixin.qq.com/s?__biz=MzUxODkyODE0Mg==&mid=2247486713&idx=1&sn=967a0a8435add4949a829eeaf2c4c647&chksm=f9802e98cef7a78e8a5439c9d64c0f8d59d800050a517eb0388877b13187d7577ef0106f1298&scene=21#wechat_redirect)

[**APP 安全合规**](http://mp.weixin.qq.com/s?__biz=MzUxODkyODE0Mg==&mid=2247485420&idx=1&sn=20f1069713342043488db05568afec9e&chksm=f980258dcef7ac9b2e142ef68076b89878da5026834854a5baf517cdecd3be961947137c5c47&scene=21#wechat_redirect)