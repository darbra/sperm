> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bilibili.com](https://www.bilibili.com/read/cv15350941)

> Zygisk 版本面具 Magisk 虽然移除了 MagiskHide 但也可以实现隐藏 Root 具体该如何实现呢且听我细细道来各位看官老爷们大家好呀我是沉迷搞机无法自拔的阿蒲之前我们聊过最新版本的面具 Magisk......

Zygisk 版本面具 Magisk 虽然移除了 MagiskHide 但也可以实现隐藏 Root

具体该如何实现呢

且听我细细道来

各位看官老爷们

大家好呀

我是沉迷搞机无法自拔的阿蒲

之前我们聊过最新版本的面具 Magisk 移除在线模块和 MagiskHide 功能

最新 Zygisk 版面具 Magisk 将隐藏的相关工作交了出来

![](http://i0.hdslb.com/bfs/article/7a98793546ab3ab90584f73ff0f0cc37000c2ee0.png@894w_600h_progressive.webp)

反而促进了隐藏领域的快速发展

其中 LSPosed 开发者出品的 Shamiko 广受好评

我们先刷入面具 Magisk

![](http://i0.hdslb.com/bfs/article/1444be6c3cbdd669222e393afda3985a74eaf37b.png@942w_707h_progressive.webp)

  
在其设置中找到并点击隐藏 Magisk 应用

![](http://i0.hdslb.com/bfs/article/737b379fe7f00b3fda2c757e5433994a6c4af0c2.jpg@942w_531h_progressive.webp)

设置一个新的 Magisk 应用名称并点击确认

![](http://i0.hdslb.com/bfs/article/341b9c9145293401c7582662bebc2eecc2452ce8.jpg@942w_531h_progressive.webp)

稍等片刻后会生产一个随机包名的面具包

安装后我们添加一个快捷方式到桌面

我们通过该快捷方式打开面具

在其设置中我们找到 Zygisk 并点击开启

![](http://i0.hdslb.com/bfs/article/76c671b53481da844a68d4e71bddecc36ba993b6.jpg@942w_1058h_progressive.webp)

然后重启手机

再次打开面具

在设置中找到遵守排除列表并开启

![](http://i0.hdslb.com/bfs/article/f275eecf12fe53b17e5120388bbd22908dcf7886.jpg@942w_782h_progressive.webp)

在设置中点击配置排除列表

在排除列表中找到需要隐藏 ROOT 的对应应用并一一开启

![](http://i0.hdslb.com/bfs/article/7e1e692ebfe9134a347c3bbe6b5c96378cb7ab4b.jpg@942w_1029h_progressive.webp)

开启时注意将应用下的选项全部开启

否则可能会出现隐藏 ROOT 失败的问题

![](http://i0.hdslb.com/bfs/article/71c007cd61f4f22f3360d3592408e8feb866c8c9.jpg@573w_1080h_progressive.webp)

我们打开排除列表中的银行类应用

基本上不会出现因 ROOT 而产生的异常提醒

但是此时排除列表中的应用是无法使用虚拟框架和模块的

如果我想对某个排除列表中的应用使用虚拟框架和模块

就需要使用到 Shamiko 模块

![](http://i0.hdslb.com/bfs/article/f0dfd488b74a46dd2d404bc92df627fb3b26e369.png@942w_483h_progressive.webp)

我们去浏览器中下载 Shamiko 模块并保存到方便的位置

在面具 Magisk 的模块功能中使用本地安装刷入该模块

![](http://i0.hdslb.com/bfs/article/dd960796932f0c9ee294c5d0ca36321e886e8db1.jpg@942w_531h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/0962964954ede1947f46839f7845c841b4b2813c.jpg@942w_531h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/a45eff57b84c57a36f219124a44e5f37a5837b9f.jpg@585w_1080h_progressive.webp)

重启手机

再次打开面具 Magsik

在其模块功能中可以看到 Shamiko 模块已经成功刷入并加载

此时 Shamiko 中有显示 Shamiko doesn't work since enforce denylist is enabled

![](http://i0.hdslb.com/bfs/article/d24196ef7189e01035eb10d676de8824801321c8.jpg@942w_531h_progressive.webp)

我们在排除列表中开启支付宝

在浏览器或 LSPosed 仓库中下载秋风模块并安装

![](http://i0.hdslb.com/bfs/article/da0b8a6d9ef69a47a05ee73623ae7ebf38632b0d.jpg@942w_531h_progressive.webp)

去 LSPosed 的模块功能开启该模块并设置其作用域

![](http://i0.hdslb.com/bfs/article/26d42facbc7a094b0dc1ae36309d1a8491b11d7f.jpg@942w_531h_progressive.webp)

打开秋风设置功能进行一定的个性化设置

![](http://i0.hdslb.com/bfs/article/0499f3f09bee35848deabf54c93f75911585e933.jpg@942w_911h_progressive.webp)

我们在面具 Magisk 的设置中关闭遵守排除列表

![](http://i0.hdslb.com/bfs/article/786d215d631f76d3335c799bcbde706e82d50932.jpg@942w_531h_progressive.webp)

再次回到模块功能

此时 Shamiko 中显示 Shamiko is working as blacklist mode

我们打开排除列表中的银行类应用

仍然未出现因 ROOT 而产生的异常提醒

我们重启手机

打开支付宝即可看到秋风模块已生效

大家有什么好玩有趣有料的资源

欢迎留言推荐

各位看官老爷们

有钱的捧个钱场

没钱的捧个人场

跪求一键三连

咱们回见

本期关键词：【Shamiko】

![](http://i0.hdslb.com/bfs/article/33b44d59f06d2d2de108f1c6222b3e695803c228.jpg@942w_530h_progressive.webp)