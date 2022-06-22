> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5ODgyOTUzNw==&mid=2247484156&idx=1&sn=8493384dc6489fc0237a6f9366d0e811&chksm=febf7272c9c8fb64b81caa7a2c524277a909cfa3e171a133dd2c0bd68cb8af1b4f4d2406957b&mpshare=1&scene=1&srcid=0622k2pUFLoJBXLoRwVTyMSa&sharer_sharetime=1655876814783&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

![](https://mmbiz.qpic.cn/mmbiz_jpg/PicMqQs6T6MPPjqgFChxkZVTyOa8cGqcQnxqvu7ozmhNElGibaibXZggx4CZ4XKZvXkjjLESqjSKFuawgff4hlWicQ/640?wx_fmt=jpeg)

最近有大佬在 android 上实践 ebpf 成功  
前有 evilpan 大佬：https://bbs.pediy.com/thread-271043.htm  
后有 weishu 大佬：https://mp.weixin.qq.com/s/mul4n5D3nXThjxuHV7GpMA  
当然还有其他隐藏的大佬啦，就不一一列举啦  
遂 android-ebpf 大火  
两位大佬的方案也很有代表性，一个是 androdeb + 自编内核 + 内核移植 + 内核 4.19（文章中看的），一个是 androdeb + 内核 5.10(pixel 6)  
目前来看，androdeb + 高版本内核 方案可以更快上手，花钱投资个新设备就好了，而且 weishu 大佬也已经手把手把工具都准备好了  
故本次就是对 weishu 大佬视频号直播的 "搭建 Android eBPF 环境" 的文字实践 + 反调试样本测试  

eBPF 是啥
-------

> 来自大佬的总结：https://mp.weixin.qq.com/s/eI61wqcWh-_x5FYkeN3BOw

失败尝试
----

> 魅族 18 内核版本 5.4  
> 虽说环境编译成功了，但体验脑壳疼  
> opensnoop 没有 path  
> execsnoop pwd 命令监控不到，长命令被截断  

环境准备
----

> PC 环境：macOS  
> 小米 12 内核版本 5.10.43  
> magisk 提供 root  
> androdeb 连接方式选取的也是 ssh 方式，故安装 SSH for Magisk 模块提供 ssh 功能  
> 手机最好也科学上网一下吧，要 git 拉一些东西  

环境准备 over，开干
------------

> #### 确保手机 ssh 已开启，先去 adb shell 中 ps 一下
> 
> ps -ef|grep sshd
> 
> #### 没问题的话，就查看下 PC 上的 ssh key
> 
> cat ~/.ssh/id_rsa.pub
> 
> #### 然后把 key 粘贴到手机 authorized_keys 文件中，再改下权限
> 
> su  
> cd /data/ssh/root/.ssh/  
> /data/adb/magisk/busybox vi authorized_keys  
> chmod 600 authorized_keys  
> 
> #### 再看下手机 ip（因为是 ssh 连接，故手机和 PC 在同一局域网下）
> 
> ifconfig |grep addr
> 
> #### 在 PC 上测试下 ssh 是否可以成功连接
> 
> ssh root@手机 ip
> 
> #### 没问题的话，直接开搞准备好的 androdeb 环境了（weishu 大佬用 rust 重写了叫 eadb）
> 
> sudo chmod 777 ./eadb-darwin  
> ./eadb-darwin --ssh root@手机 ip prepare -a androdeb-fs.tgz
> 
> #### 等待完成后，进 androdeb shell, 开始编译 bcc
> 
> ./eadb-darwin --ssh root@手机 ip shell  
> git clone https://github.com/tiann/bcc.git --depth=1  
> cd bcc && mkdir build && cd build  
> cmake .. make -j8 && make install
> 
> #### 等待成功后，就有各种工具可以用了

> #### 👆👆👆得益于 weishu 大佬的手把手环境工具包，androdeb + 内核 5.10 的 eBPF 环境搭建起来就是这么简单

反调试样本实操
-------

> DetectFrida.apk 核心逻辑: https://github.com/kumar-rahul/detectfridalib/blob/HEAD/app/src/main/c/native-lib.c
> 
> #### 哎😆，这里我直接就拿山佬的实践来说，至于为啥后面再说
> 
> ![](https://mmbiz.qpic.cn/mmbiz_jpg/PicMqQs6T6MPPjqgFChxkZVTyOa8cGqcQtnYibEB7jhbjzNic3UINAtbicvic3tY97VAShcrfGku83OXEicvvZrS28Zg/640?wx_fmt=jpeg)
> 
> ![](https://mmbiz.qpic.cn/mmbiz_jpg/PicMqQs6T6MPPjqgFChxkZVTyOa8cGqcQv7bXDDgKks9DUjKMTwQJPic7dOV3DlHGsSk4OfiaialiczOdMmGrkvDjJQ/640?wx_fmt=jpeg)
> 
> #### 还少了一个关键的
> 
> ![](https://mmbiz.qpic.cn/mmbiz_jpg/PicMqQs6T6MPPjqgFChxkZVTyOa8cGqcQNuw5FuAfzqysVDib81sN1k43s8rBqnUl2bcjK2MUMcFHLbV0HEq0ib1w/640?wx_fmt=jpeg)
> 
> #### 手写 trace 干它
> 
> trace 'do_readlinkat"%s", arg2@user' --uid 10229  
> 
> #### 再来一次
> 
> ![](https://mmbiz.qpic.cn/mmbiz_jpg/PicMqQs6T6MPPjqgFChxkZVTyOa8cGqcQqxNZmt6lVdbBctmNMiarJD1sWiaJEEwpf4oRL3nm7Zsbcq4YJR7UlUWQ/640?wx_fmt=jpeg)
> 
> #### 👆👆👆可以了，差不多了，这样分析已经为后续对抗 bypass 提供了很大的帮助
> 
> #### 当然了，上述只是最基础的操作，后续还得继续深入探索学习，解锁更多顶级玩法
> 
> #### 还有就是，其实我的 Xiaomi 12 还没搞好，在等解 BL 锁，至于秒解，我不想花钱，所以就拿山佬的实践来借花献佛了，真是个好主意啊，哈哈😄

总结
--

基于内核级别的监控，让应用中所有的加固 / 隐藏 / 内联汇编等防御措施形同虚设，而且可以在应用启动的初期进行观察，让应用的一切行为在我们眼中无所遁形  
这是真真正正的降维打击，内核级的探测能力提供了无限可能，堪称：屠龙技  

最后
--

文中用的工具和软件，我已经打包整理好了  
公众号聊天界面回复 "ebpf" 即可  
再次感谢先行者大佬们的无私奉献，和为技术发展所做的贡献🎉🎉🎉