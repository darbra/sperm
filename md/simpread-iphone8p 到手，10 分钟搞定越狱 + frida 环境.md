> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NKktK21jE2KYx6E5RQCphg)

1. 电脑一部（Win 系统的和 Mac 设备都行）、越狱 iphone 一部

2. 爱思助手

3. 数据线一条

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkyg3v06rzwtVuUuHKNCF6jtXXVv5whIg5nNxHS6iaSrfibOUUOVRxWtDA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkS3yWicWK1ktqibFcMiaDoX1XRlZMbqVz6b9xmKyGDIORxOTCZzXQGMcsQ/640?wx_fmt=png)

开始越狱，完成之后打开 unc0verapp，如果出现 unsupport，说明不支持的  

信任证书

打开设置 通用 描述文件与设备管理

无脑点 Jailbroken

unc0ver 如果出现 unsupport，说明不支持当前版本

那就换 checkra1n，同样如果支持的版本可以直接 start

有些版本像这样需要改一下配置，点击 options

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkGJOSXzDmWq6G3buBl1ezLnRelckMiarrb3xhTkl84dFhbs1ePxZXUzg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkgPibmTPeUP3kWLrlkMNcHsfRg7I0UoqAxwY3xAricHIvFFwSic2ayu0Ng/640?wx_fmt=png)

钩上这几个，back 之后点 start

然后按照他的图来按电源键 音量下键

完事之后等待一会，桌面会出现 checkra1n app，进去，安装 cydia，没有 vpn 会很慢

安装 openssh  

wifi 详情获取 ip 地址  

ssh root@ip

默认密码 alpine

passwd 更改密码  

安卓 frida，去 github 下载 deb 格式的

推到手机上  

```
scp frida_15.1.1_iphoneos-arm.deb root@ip:/var/mobile/Media

```

安装：

```
dpkg -i /var/mobile/Media/frida_15.1.1_iphoneos-arm.deb

```

查看 frida 进程

```
ps aux |grep frida

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkoE1y9PKeDxVia4h4ot3OtR4ibDRxx9xNIwDlxqhgDic4icLWGUdRmCpTTQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkxRtp0UIndVXhMhTBMPyAeEuykrV3bj7RPywtzXMfTPxrj89jIibpMnw/640?wx_fmt=png)

这样就是成功的