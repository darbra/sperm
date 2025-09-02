> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zeeklog.com](https://zeeklog.com/android-an-quan-qi-dong-yan-zheng-lian/)

> 前言 1、签名验签 启动验证链核心技术就是签名验签，这里复习下，流程如上图： android 基础算法使用的是 RSA 红色部分是编译时对需要验签的镜像与文件进行签名操作；需要用到私钥，这个私钥一般放在编译代码或者签名服务器。

前言

1、签名验签
------

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-22f2b95dc4c8b.png)

启动验证链核心技术就是签名验签，这里复习下，流程如上图：

android 基础算法使用的是 RSA

红色部分是编译时对需要验签的镜像与文件进行签名操作；需要用到私钥，这个私钥一般放在编译代码或者签名服务器。

黄色部分是开机启动时对启动镜像与文件的验签；需要用到公钥，这个公钥集成到镜像中，一般对开发与使用者都是可见的。

2、安全启动验证链
---------

极简的开机启动流程如下

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-03541fd8b9d46.png)

红色部分是 secure boot 部分，黄色部分是 AVB 部分（AVB 包括 verify-boot 与 dm-verity）。

以 MTK 为例：

Secure Boot 验证阶段，Boot Rom 程序段上电直接运行，这段程序不能修改或者跳过，它将获取芯片内部非易失只读性存储中的公钥 or 公钥 HASH；用于验证引导加载程序（preloader）镜像末尾的 root 公钥，验证成功后启动 preloader；然后 preloader 代码段的公钥头文件（oemkey.h）验证 ATF、TEE、lk；在启动到 lk 后，利用 TEE 提供的加密库使用公钥头文件（oemkey.h）验证 md1、scp、logo 等。

AVB（android verify boot）校验阶段，启动到 lk 会用公钥头文件 (avbkey.h) 验证 vbmeta 分区，再用验证通过的 vbmeta 分区里面存的公钥对一般较小的分区（例如 boot、dtbo）进行 hash 校验；验证完毕后进入 kernel，kernel 使用 dm-verity 对动态分区（一般较大的分区，例如 system，vendor）进行 hash tree 的校验，所有分区校验成功后正常挂载，设备继续进入 android 启动流程。

### 2.1、secure boot

发现有价值文章：

现在网上内容很多，不再赘述，解答标红的三个问题

QCOM 启动流程图参考

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-36bf05dde4987.png)

MTK 启动流程图参考

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-ea72160fa8be4.jpeg)

MTK 的 secure boot 配置参考:

QCOM 的 secure boot 配置参考：

**1、只是刷入 efuse.img 镜像设备就熔丝了？**

fastboot flash efuse 只是刷写 emmc/ufs 上的分区数据，熔丝并不是 fastboot 做的事。如果还在调试，efuse 熔丝公钥与 preloader 签名不匹配那不就完蛋了吗？

我们看不到 Boot Rom 代码，在 secure boot 安全验证上，以及芯片手册上能猜测其流程。其实从 QCOM 网站看到，文档名记不住了，大概意思如下流程图。

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-d4a5e609cf2fc.png)

*   熔丝真正操作在 TEE，efuse 公钥 hash 判断有效后，重启才真正生效，从此每次开机进行 secure boot 校验。
*   硬件关闭公钥存储写入能力，也就是熔断操作，会耗电；如同读写标志位强制拉低，从此此段存储再也无法篡改。
*   熔丝寄存器位也置为有效，再也无法关闭。

这就创建了一个值得信任的根。

**2、为什么 secure boot 有多级验证？**

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-7a43e929acef2.png)

MTK 对其使用的镜像 md1、scp、logo 等等进行二次签名，那么开机需要进行二次验证增加启动耗时，安全性只是倍增，而不是指数级增加，有必要吗？

先了解多级认证的作用

我理解这是供应商自己玩的隔离，即 md1、scp、logo 部分使用 MTK 自己签名授权的一级签名，二级签名交给 ODM。这样 MTK 就控制了 md1、scp 等部分的芯片功能。（貌似还是跟 QCOM 学的。）

**3、如何联网签名？**

软件架构方案已经有了，但开发人员随意签名漏洞版本、可恶意窃取数据版本怎么办？如何降低开发人员的攻击呢？

ODM 只需要管理好发布版本，管理好私钥即可，这就是非对称密钥的好处。将私钥存储到服务器，开发人员编译镜像需要授予签名权限，控制版本输出。MTK 与 QCOM 的 secboot 签名都做好了接口，修改非常方便，有钱也可直接请人搭建👌。

MTK 的 secure boot 最终签名接口调用 hsm.py，路径应该在 vendor/mediatek/proprietary/scripts/secure_chip_tools，如注释部分描述，替换 lib.cert.sig_gen 的实现即可。

hsm_rsa_sign(data, key, padding, sig) 函数中，data 是镜像 hash、key 私钥名、padding 加密方式，sig 返回签名后内容。

```
def hsm_rsa_sign(data, key, padding, sig):  
    """ 
    sign data with HSM 
    """  
    # note that key is pubk actually, use it as index for  
    # HSM parameters such as key selection  
    hsm_param_obj = HsmParam()  
    key_table = create_key_table()  
    hsm_param_obj.m_prvk = query_key_table(key_table, key)  
     if hsm_param_obj.m_prvk is None:  
         print 'not valid HSM parameter'  
         return -1  
   
     print "========================"  
     print "HSM parameter:"  
     print "    m_prvk  = " + hsm_param_obj.m_prvk  
     print "========================"  
   
     # place hsm request here -- start  
     # we re-direct it to signing with private key to mimic HSM  
     # data is not hashed here, you can hash data here to reduce  
     # network usage  
     lib.cert.sig_gen(data, hsm_param_obj.m_prvk, padding, sig)  
     # place hsm request here -- end  
     return 0   

```

联网签名步骤参考：

*   连接服务器
*   透传前面三个参数
*   服务器解析
*   验证账号与密钥对应权限
*   使用指定的密钥签名，
*   返回签名数据

联网签名使用的是镜像 hash，联网签名时间不受被签名镜像的大小影响。

### 2.2、AVB

现在网上内容很多，不再赘述，解答标红的问题

android 启动流程

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-a40756883e33c.png)

android 的 AVB 配置：

开启只需要打开 AVB 开关即可

BOARD_AVB_ENABLE:=True

BOARD_AVB_ALGORITHM := SHA256_RSA2048

BOARD_AVB_KEY_PATH := device/mediatek/common/oem_prvk.pem

如果是 MTK 需要开启 kernel 的 config

MTK_AVB20_SUPPORT default y

换密钥参考：

其中 2、3 步生成公钥头文件，MTK 有现成转换工具 der_extractor

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-6696aa24de768.png)

AVB 工具

vbmeta 结构

**1、AVB 如何联网签名？**

google 当然帮你想好了。avbtool 有两个参数（--signing_helper 与 --signing_helper_with_files）均能达到联网签名的目的。

![](https://qiniu.meowparty.cn/coder.2023/2025-01-04/Lesson-dd8d1da64995a.png)

**2、新增镜像如何纳入验证链?**

如果放入 secureboot 流程，在 env_cfg 文件配置；

如果放入 verify-boot 流程，用 vbmeta 结构进行 hash 验证，可额外定制密钥，avbtool add_hash_footer 进行镜像签名，avbtool make_vbmeta_image 制作 vbmeta 镜像时增加相关参数。

如果放入 dm-verity 流程，用 vbmeta 结构进行 hashtree 验证，可额外定制密钥，avbtool add_hashtree_footer 进行镜像签名，avbtool make_vbmeta_image 制作 vbmeta 镜像时增加相关参数。

3、安全启动验证链总结
-----------

安全启动验证链说明：

1.  整个启动验证链的信任根是一次性写入的只读性存储中的公钥 or 公钥 HASH；
2.  这个信任根验证芯片厂商（vendor）所用到的所有镜像，关键是会验证 lk 和 TEE；
3.  lk 与 TEE 证明可信之后，才能进一步验证 kernel、dtbo、vbmeta，并确认结果；如果 vbmeta 有 hash 校验的分区，在此步直接校验；
4.  kernel 与 vbmeta 证明可信后，才能进一步验证 system、vendor，odm 等 hashtree 校验的分区，并确认结果。

所有不会改动的分区都应该纳入到这个验证链中，芯片厂商相关镜像一般放到 secure boot，小镜像一般用 verify-boot，大镜像一般放到动态分区进行 dm-verity。

android 安全启动验证链说明有类似 Blog，参考，我们要学的是这个链式验证的思想，其中 vbmeta 结构还能扩展很多密钥，android 的 APEX 功能就利用 vbmeta 结构也挂在这个验证链上。