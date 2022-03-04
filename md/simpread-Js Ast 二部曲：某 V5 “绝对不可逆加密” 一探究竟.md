> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.nightteam.cn](https://bbs.nightteam.cn/forum.php?mod=viewthread&tid=1498&highlight=ast) ![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1616&size=middle)Nanda 声明：本文内容仅供学习交流，严禁用于商业用途，否则由此产生的一切后果均与作者无关，请于 24 小时内删除。  
本文是前几天发的《Js Ast 一部曲：高完整度还原某 V5 的加密》的进阶，上一篇文章提到过的内容本文就不重新讲解了，所以还没看过上一篇文章的小伙伴记得先去看一下哦。  
我就不说闲话了，直入主题。 本文既然是进阶篇，当然要加大难度，所以，选项配置如下图，全部点亮 & 系数选最高。  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzAzfDRjZWU4Yzk1fDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**1.png** _(137.57 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzAzfDRjZWU4Yzk1fDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

初：我们依旧先来看一下前后的对比。  
还原前（格式化后 900 + 行，所以我就不展示格式化后的内容了，可以找  在线 js 格式化网站  自行格式化哦。)  
/*  
 * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。;  
 * 已经打算把这个工具基础功能一直免费下去。还希望支持我。  
 * 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。  
 * 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。  
 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则  
 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。 */  
;var encode_version = 'sojson.v5', gdear = '',  _0x1491=['\x77\x35\x58\x43\x6a\x33\x54\x43\x6b\x77\x77\x3d','\x63\x63\x4f\x6e\x4a\x56\x30\x6d','\x77\x36\x59\x53\x57\x38\x4f\x4f\x77\x6f\x6f\x3d','\x63\x73\x4f\x61\x52\x38\x4f\x6a\x4e\x77\x3d\x3d','\x4a\x6b\x5a\x45\x41\x63\x4b\x41','\x77\x35\x67\x78\x62\x73\x4b\x56\x77\x72\x4d\x3d','\x77\x71\x56\x48\x43\x68\x44\x43\x68\x51\x3d\x3d','\x77\x36\x76\x44\x6b\x79\x6e\x44\x6d\x67\x3d\x3d','\x49\x43\x2f\x44\x6d\x6d\x44\x44\x6c\x51\x3d\x3d','\x42\x73\x4b\x30\x4f\x46\x6e\x44\x76\x77\x3d\x3d','\x77\x6f\x44\x44\x75\x33\x34\x38','\x77\x37\x66\x43\x71\x69\x74\x43\x66\x77\x3d\x3d','\x77\x34\x7a\x44\x67\x63\x4b\x75\x77\x34\x74\x57','\x77\x6f\x30\x70\x56\x4d\x4f\x59\x77\x6f\x55\x3d','\x77\x37\x50\x43\x72\x38\x4b\x57\x77\x70\x78\x4d','\x4f\x58\x37\x44\x73\x6c\x50\x44\x74\x41\x3d\x3d','\x77\x37\x73\x32\x77\x71\x44\x43\x72\x57\x45\x3d','\x64\x4d\x4f\x77\x42\x33\x6f\x50','\x4c\x55\x2f\x44\x6c\x51\x6a\x43\x69\x77\x3d\x3d','\x77\x34\x66\x44\x73\x6a\x6e\x44\x67\x73\x4f\x31','\x62\x38\x4f\x2f\x56\x4d\x4b\x76\x77\x37\x77\x3d','\x41\x63\x4f\x72\x77\x72\x42\x4c\x47\x51\x3d\x3d','\x51\x6a\x7a\x44\x76\x42\x39\x78','\x77\x6f\x54\x44\x6b\x4d\x4f\x48\x52\x32\x55\x3d','\x44\x38\x4b\x2b\x4c\x73\x4f\x56\x57\x67\x3d\x3d','\x77\x71\x6f\x79\x77\x37\x2f\x44\x76\x54\x38\x3d','\x44\x38\x4f\x71\x77\x6f\x5a\x4d\x4d\x51\x3d\x3d','\x77\x36\x49\x32\x65\x4d\x4b\x42\x77\x70\x34\x3d','\x57\x73\x4f\x6e\x4c\x51\x62\x43\x6c\x51\x3d\x3d','\x43\x44\x72\x43\x70\x32\x2f\x44\x67\x67\x3d\x3d','\x64\x63\x4f\x46\x77\x36\x4c\x43\x76\x6d\x38\x3d','\x77\x71\x58\x44\x6b\x4d\x4b\x2b\x77\x35\x4a\x52','\x64\x38\x4f\x42\x56\x38\x4f\x51\x45\x41\x3d\x3d','\x77\x36\x4c\x43\x6c\x67\x78\x2b\x65\x77\x3d\x3d','\x45\x73\x4f\x6f\x77\x70\x2f\x43\x70\x73\x4b\x37','\x46\x63\x4f\x64\x77\x35\x6f\x78\x51\x77\x3d\x3d','\x77\x71\x2f\x44\x72\x63\x4b\x46\x44\x4d\x4f\x6d','\x66\x38\x4f\x48\x77\x37\x4c\x43\x76\x55\x4d\x3d','\x62\x4d\x4b\x62\x77\x35\x68\x52\x49\x41\x3d\x3d','\x41\x63\x4b\x6a\x77\x70\x4c\x44\x6d\x38\x4f\x6d','\x49\x51\x4c\x44\x73\x6b\x37\x44\x6a\x67\x3d\x3d','\x46\x38\x4b\x6b\x77\x71\x2f\x44\x6c\x4d\x4f\x6d','\x77\x70\x38\x4c\x65\x6b\x67\x59','\x77\x37\x49\x5a\x77\x35\x59\x3d','\x77\x71\x50\x44\x72\x6e\x30\x3d','\x41\x4d\x4b\x6d\x77\x70\x73\x3d','\x64\x6a\x6e\x44\x71\x77\x3d\x3d','\x65\x47\x66\x43\x6f\x53\x7a\x44\x76\x48\x55\x2b\x77\x6f\x33\x43\x72\x63\x4b\x64\x77\x35\x42\x43\x4a\x7a\x66\x44\x67\x6d\x50\x43\x6a\x41\x3d\x3d','\x50\x38\x4f\x42\x77\x36\x63\x36\x5a\x4d\x4f\x6a','\x49\x6d\x31\x79','\x77\x70\x48\x44\x71\x63\x4f\x4e','\x42\x4d\x4b\x47\x77\x6f\x77\x3d','\x35\x59\x69\x44\x36\x5a\x69\x6f\x35\x34\x75\x45\x35\x70\x32\x66\x35\x59\x2b\x2f\x37\x37\x32\x70\x41\x38\x4f\x51\x35\x4c\x79\x49\x35\x61\x79\x37\x35\x70\x32\x51\x35\x62\x2b\x38\x35\x36\x69\x6d','\x77\x72\x6a\x43\x6d\x53\x78\x37\x77\x36\x5a\x37','\x5a\x63\x4f\x63\x77\x70\x6f\x3d','\x43\x77\x50\x43\x76\x58\x6e\x44\x74\x63\x4b\x32\x77\x36\x48\x44\x6d\x73\x4b\x49\x58\x55\x73\x33\x77\x6f\x4e\x63\x77\x6f\x66\x44\x75\x41\x3d\x3d','\x77\x70\x66\x43\x72\x63\x4f\x33\x66\x38\x4b\x6a\x77\x6f\x73\x65\x77\x70\x52\x34\x4e\x4d\x4f\x61\x43\x73\x4f\x64\x77\x35\x58\x44\x67\x63\x4b\x43\x66\x45\x5a\x65\x52\x31\x49\x39\x61\x69\x4a\x79\x4a\x73\x4b\x77\x77\x35\x72\x43\x6d\x73\x4f\x68\x77\x71\x48\x44\x6d\x4d\x4f\x42\x77\x72\x73\x2f\x77\x35\x72\x43\x67\x73\x4b\x6c\x77\x35\x42\x59\x77\x35\x4c\x43\x70\x46\x70\x35\x48\x31\x62\x43\x68\x6c\x35\x66\x77\x37\x50\x44\x67\x38\x4b\x55\x77\x35\x72\x43\x6d\x4d\x4f\x74\x54\x73\x4b\x31\x4f\x45\x2f\x43\x70\x30\x6f\x46','\x77\x37\x62\x44\x6d\x44\x50\x44\x6d\x67\x3d\x3d','\x54\x63\x4f\x4c\x41\x55\x55\x51','\x50\x55\x46\x43\x46\x45\x49\x3d','\x46\x41\x44\x44\x72\x31\x54\x44\x75\x77\x3d\x3d','\x77\x70\x62\x43\x69\x52\x4a\x6c\x77\x35\x34\x3d','\x58\x73\x4f\x78\x77\x36\x44\x43\x68\x6e\x77\x3d','\x77\x34\x41\x54\x59\x63\x4f\x46\x77\x6f\x77\x3d','\x55\x73\x4f\x74\x77\x36\x50\x43\x75\x6d\x49\x3d','\x77\x37\x38\x4f\x77\x71\x6b\x68','\x77\x71\x44\x44\x76\x38\x4b\x58\x42\x4d\x4f\x69\x77\x35\x4c\x43\x73\x63\x4f\x4a\x77\x72\x63\x3d','\x4e\x63\x4f\x4c\x77\x70\x4c\x43\x73\x63\x4b\x4c','\x43\x38\x4b\x6d\x45\x63\x4f\x33','\x54\x38\x4f\x72\x58\x4d\x4b\x4a\x77\x37\x59\x3d','\x77\x71\x6e\x44\x71\x4d\x4b\x54','\x77\x34\x55\x6b\x77\x70\x2f\x43\x75\x45\x41\x3d','\x48\x6d\x52\x31\x52\x43\x41\x3d','\x77\x34\x67\x58\x77\x72\x6a\x43\x72\x56\x77\x3d','\x77\x37\x50\x43\x6d\x73\x4b\x54\x77\x71\x31\x77','\x50\x63\x4b\x31\x45\x30\x62\x44\x74\x51\x3d\x3d','\x77\x34\x6a\x43\x6b\x47\x77\x3d','\x50\x63\x4f\x6b\x77\x37\x66\x43\x6f\x4d\x4f\x67','\x43\x73\x4f\x4d\x77\x6f\x6e\x43\x73\x63\x4b\x57','\x77\x35\x30\x69\x51\x63\x4b\x76\x77\x70\x34\x3d','\x77\x72\x74\x54\x77\x34\x52\x4d\x58\x4d\x4b\x61\x77\x71\x37\x43\x74\x7a\x62\x43\x75\x42\x44\x44\x73\x53\x2f\x44\x76\x47\x73\x77\x77\x70\x77\x3d','\x4d\x58\x78\x51\x4d\x6d\x59\x3d','\x77\x6f\x66\x44\x72\x6d\x45\x68\x77\x71\x51\x3d','\x77\x35\x59\x62\x62\x73\x4f\x4a\x77\x6f\x67\x51\x77\x70\x74\x56\x77\x35\x63\x3d','\x4f\x63\x4b\x4e\x43\x47\x7a\x44\x76\x41\x3d\x3d','\x77\x36\x42\x41\x77\x70\x41\x3d','\x77\x37\x6a\x43\x6b\x55\x72\x43\x74\x7a\x6b\x3d','\x77\x71\x77\x72\x66\x73\x4f\x38','\x44\x31\x70\x4a\x4d\x73\x4b\x4f','\x46\x6d\x4c\x44\x75\x47\x2f\x44\x6b\x51\x3d\x3d','\x77\x71\x44\x44\x74\x38\x4f\x2b\x56\x6e\x67\x3d','\x77\x70\x66\x44\x69\x30\x34\x51\x77\x72\x34\x3d','\x53\x63\x4f\x4d\x62\x63\x4b\x2b\x77\x36\x6f\x3d','\x49\x73\x4f\x64\x77\x72\x44\x43\x68\x4d\x4b\x58','\x77\x36\x66\x43\x74\x6b\x6c\x63\x59\x77\x3d\x3d','\x77\x71\x4c\x44\x68\x38\x4b\x6f\x77\x37\x6c\x70','\x77\x34\x62\x44\x67\x6a\x33\x43\x72\x77\x51\x3d','\x77\x6f\x62\x44\x76\x32\x45\x46\x77\x71\x6f\x3d','\x63\x63\x4f\x38\x42\x30\x59\x47','\x53\x63\x4f\x4e\x4f\x58\x55\x4c','\x47\x38\x4f\x4d\x77\x36\x44\x43\x76\x38\x4f\x49','\x77\x36\x6a\x44\x71\x38\x4b\x42\x77\x34\x46\x38','\x77\x36\x49\x6e\x65\x4d\x4f\x4e\x77\x72\x6f\x3d','\x77\x71\x72\x44\x68\x73\x4b\x46\x77\x36\x78\x36','\x77\x6f\x72\x44\x6c\x4d\x4b\x68\x77\x36\x5a\x71','\x45\x73\x4f\x7a\x77\x71\x50\x43\x69\x73\x4b\x6d','\x77\x35\x31\x46\x77\x70\x52\x64\x4c\x77\x3d\x3d','\x77\x72\x33\x44\x73\x73\x4b\x43\x77\x36\x73\x3d','\x52\x73\x4f\x34\x77\x70\x66\x44\x68\x77\x6b\x3d','\x77\x72\x6a\x44\x6b\x38\x4b\x58\x77\x36\x64\x4e','\x45\x30\x68\x6d\x46\x4d\x4b\x65','\x77\x34\x54\x44\x6b\x78\x66\x43\x70\x63\x4f\x66','\x77\x72\x55\x4e\x61\x57\x55\x59','\x5a\x63\x4f\x34\x56\x73\x4b\x50\x77\x35\x45\x3d','\x77\x6f\x62\x44\x6f\x38\x4f\x2b\x63\x6e\x38\x3d','\x77\x36\x6a\x44\x73\x52\x2f\x44\x67\x4d\x4f\x59','\x77\x37\x63\x34\x77\x36\x50\x43\x73\x4d\x4b\x5a','\x42\x63\x4b\x4f\x48\x73\x4f\x2f\x5a\x67\x3d\x3d','\x53\x4d\x4f\x37\x42\x54\x7a\x43\x74\x51\x3d\x3d','\x77\x6f\x54\x44\x69\x63\x4f\x6c\x65\x6e\x45\x3d','\x77\x35\x73\x43\x77\x37\x6a\x43\x69\x63\x4b\x35','\x41\x32\x4c\x44\x73\x52\x50\x43\x6d\x41\x3d\x3d','\x77\x71\x45\x32\x77\x34\x76\x44\x72\x7a\x34\x3d','\x45\x7a\x7a\x44\x6e\x6b\x6a\x44\x75\x41\x3d\x3d','\x77\x37\x37\x44\x75\x73\x4b\x61\x4b\x4d\x4b\x77\x77\x35\x30\x47\x77\x35\x64\x32\x46\x38\x4f\x63\x44\x73\x4f\x48','\x35\x61\x65\x54\x35\x70\x36\x79\x35\x6f\x43\x56\x35\x35\x71\x52\x77\x34\x39\x5a\x36\x59\x53\x4c\x35\x62\x53\x41\x35\x61\x61\x42\x35\x4c\x75\x65\x77\x35\x33\x44\x72\x73\x4b\x39\x37\x37\x32\x4b\x77\x71\x6e\x43\x76\x4d\x4b\x4e\x35\x71\x47\x37\x35\x36\x36\x47\x37\x37\x2b\x69\x35\x36\x32\x46\x35\x36\x79\x6f\x35\x59\x65\x4a\x35\x4c\x6d\x2f\x36\x5a\x2b\x42\x52\x58\x55\x50\x77\x71\x46\x78\x77\x71\x77\x37\x51\x63\x4b\x4e\x77\x6f\x6e\x6e\x6d\x70\x72\x6b\x75\x49\x76\x6e\x6f\x37\x37\x76\x76\x34\x66\x6f\x72\x4b\x44\x6d\x6a\x49\x66\x6c\x6a\x37\x62\x6c\x68\x71\x48\x6d\x6e\x34\x6a\x6c\x68\x34\x66\x6c\x69\x36\x4c\x6c\x72\x5a\x7a\x6a\x67\x49\x6e\x6f\x76\x35\x50\x6b\x75\x61\x66\x6c\x74\x70\x72\x6c\x68\x70\x54\x6b\x75\x61\x62\x6f\x67\x62\x62\x6c\x69\x49\x37\x6c\x72\x36\x33\x43\x74\x4d\x4b\x73\x62\x4f\x4f\x44\x72\x45\x34\x43\x61\x4f\x65\x73\x6e\x75\x61\x72\x74\x4f\x65\x49\x67\x2b\x57\x48\x72\x4f\x57\x75\x73\x41\x3d\x3d','\x36\x4c\x79\x6e\x35\x70\x6d\x55\x35\x4c\x69\x4b\x35\x4c\x69\x4f\x35\x4c\x6d\x68\x35\x37\x4f\x34\x35\x59\x6d\x39\x57\x63\x4b\x6b\x35\x70\x4f\x38\x35\x4c\x36\x6a\x34\x34\x4f\x51','\x77\x37\x6f\x52\x53\x51\x3d\x3d','\x35\x36\x6d\x45\x36\x5a\x61\x63\x35\x6f\x36\x64\x36\x61\x69\x41\x35\x37\x75\x73\x5a\x2b\x4b\x43\x72\x6d\x72\x43\x73\x4f\x57\x4b\x6e\x2b\x57\x74\x72\x2b\x4b\x43\x74\x45\x6e\x6c\x6b\x5a\x39\x4c\x34\x6f\x43\x39\x4a\x57\x33\x6f\x70\x5a\x2f\x6c\x72\x36\x33\x69\x67\x35\x74\x74\x37\x37\x36\x6e\x35\x4c\x36\x69\x35\x59\x32\x4a\x35\x4c\x32\x76\x35\x35\x6d\x5a\x4c\x52\x50\x44\x6e\x75\x4f\x43\x6e\x41\x3d\x3d','\x77\x37\x4c\x43\x69\x38\x4b\x56\x77\x70\x64\x64','\x49\x6e\x31\x53\x64\x51\x45\x3d','\x63\x4d\x4f\x44\x77\x70\x66\x44\x6e\x7a\x55\x3d','\x77\x6f\x30\x74\x4c\x63\x4b\x2b\x77\x36\x67\x3d','\x77\x34\x55\x6e\x65\x63\x4b\x45\x77\x71\x45\x3d','\x77\x70\x59\x76\x59\x4d\x4f\x70\x77\x71\x73\x3d','\x4a\x6a\x44\x43\x6d\x31\x37\x44\x72\x67\x3d\x3d','\x45\x6b\x52\x37\x41\x33\x38\x3d','\x51\x4d\x4f\x73\x62\x63\x4b\x79\x77\x35\x63\x3d','\x77\x35\x58\x44\x73\x6a\x33\x43\x76\x38\x4f\x69','\x4e\x77\x58\x44\x6a\x30\x48\x44\x6a\x67\x3d\x3d','\x63\x4d\x4f\x73\x77\x6f\x33\x44\x67\x77\x6f\x59\x49\x41\x3d\x3d','\x56\x4d\x4b\x46\x77\x6f\x6e\x44\x75\x4d\x4b\x58','\x64\x38\x4f\x61\x77\x34\x6e\x43\x68\x30\x6f\x3d','\x51\x4d\x4b\x45\x77\x6f\x77\x3d','\x77\x35\x58\x43\x70\x46\x49\x3d','\x48\x6c\x46\x50\x49\x73\x4b\x50\x59\x6d\x4c\x43\x6b\x73\x4b\x2b','\x50\x6d\x39\x55\x65\x52\x5a\x4f','\x4e\x31\x76\x44\x6b\x33\x62\x44\x73\x57\x50\x43\x71\x43\x49\x3d','\x54\x45\x30\x4f\x4b\x78\x37\x43\x67\x38\x4b\x53\x77\x34\x39\x79\x77\x70\x67\x63\x42\x68\x38\x3d','\x77\x36\x33\x44\x73\x51\x58\x43\x6d\x44\x67\x3d','\x50\x6c\x31\x75\x47\x48\x38\x3d','\x77\x71\x37\x44\x73\x4d\x4b\x6a\x4b\x63\x4f\x59','\x64\x4d\x4f\x4e\x77\x34\x4c\x43\x6e\x32\x45\x3d','\x53\x73\x4f\x36\x61\x73\x4f\x55\x48\x51\x3d\x3d','\x4f\x38\x4b\x75\x77\x72\x48\x44\x76\x73\x4f\x55','\x77\x36\x37\x43\x69\x6c\x50\x43\x6d\x69\x77\x3d','\x77\x36\x2f\x44\x73\x52\x2f\x44\x6a\x4d\x4f\x58','\x41\x55\x6a\x44\x6b\x30\x62\x44\x6a\x77\x3d\x3d','\x43\x48\x5a\x39\x4e\x57\x6b\x3d','\x55\x63\x4f\x66\x65\x63\x4f\x48\x47\x77\x3d\x3d','\x4e\x4d\x4f\x42\x77\x6f\x74\x54\x4f\x54\x70\x78','\x77\x34\x50\x44\x70\x41\x58\x43\x72\x52\x7a\x44\x75\x4d\x4b\x78','\x77\x36\x30\x79\x61\x63\x4b\x76\x77\x71\x76\x44\x6a\x67\x72\x44\x76\x73\x4f\x59\x77\x35\x46\x35\x77\x72\x55\x43\x77\x6f\x66\x44\x72\x30\x38\x4d','\x77\x34\x62\x44\x70\x43\x66\x43\x6e\x52\x45\x3d','\x4a\x4d\x4f\x65\x77\x6f\x6c\x4a\x49\x67\x3d\x3d','\x46\x73\x4f\x4f\x77\x37\x6a\x43\x70\x38\x4f\x73','\x4b\x63\x4f\x4e\x77\x37\x59\x32\x65\x73\x4f\x77\x53\x73\x4f\x73\x62\x51\x3d\x3d','\x54\x79\x50\x44\x71\x77\x3d\x3d','\x77\x72\x33\x44\x70\x63\x4b\x51\x77\x37\x78\x2b','\x77\x72\x51\x6b\x77\x35\x58\x44\x67\x67\x3d\x3d','\x77\x72\x56\x77\x4e\x53\x6f\x3d','\x77\x71\x45\x4f\x77\x37\x72\x43\x73\x73\x4b\x48','\x77\x71\x6e\x44\x71\x38\x4f\x76\x52\x6d\x55\x3d','\x57\x63\x4f\x70\x51\x73\x4b\x50\x77\x37\x41\x3d','\x77\x36\x37\x43\x71\x6c\x46\x56\x57\x73\x4b\x49\x4a\x51\x3d\x3d','\x77\x36\x6a\x43\x69\x73\x4b\x2b\x77\x70\x74\x5a','\x77\x71\x72\x44\x75\x4d\x4b\x66\x77\x36\x78\x30\x48\x4d\x4b\x43','\x4d\x73\x4f\x63\x77\x70\x64\x50\x4a\x41\x3d\x3d','\x77\x72\x77\x68\x4e\x4d\x4b\x67\x77\x37\x4c\x44\x6e\x6c\x38\x3d','\x77\x70\x44\x44\x75\x32\x38\x39\x77\x72\x63\x3d','\x77\x37\x62\x44\x6d\x44\x7a\x44\x67\x51\x3d\x3d','\x77\x37\x41\x55\x77\x37\x37\x43\x75\x63\x4b\x41\x4d\x57\x63\x3d','\x77\x36\x48\x43\x71\x6c\x67\x3d','\x77\x6f\x48\x44\x75\x43\x68\x59\x59\x6b\x4c\x43\x75\x41\x3d\x3d','\x77\x6f\x66\x44\x72\x79\x56\x4f\x66\x56\x72\x43\x74\x41\x50\x43\x71\x41\x3d\x3d','\x77\x71\x63\x54\x77\x36\x62\x43\x72\x73\x4b\x61\x66\x58\x77\x3d','\x46\x63\x4b\x70\x42\x63\x4f\x32','\x77\x72\x33\x43\x6b\x69\x52\x7a','\x54\x4d\x4f\x59\x77\x34\x58\x43\x6c\x31\x49\x3d','\x58\x38\x4f\x39\x51\x63\x4b\x48\x77\x34\x55\x3d','\x64\x51\x62\x44\x74\x51\x74\x53','\x77\x36\x66\x44\x6d\x54\x4c\x43\x6c\x63\x4f\x5a','\x4c\x4d\x4b\x62\x41\x47\x48\x44\x76\x79\x77\x3d','\x51\x63\x4b\x64\x77\x70\x50\x44\x75\x73\x4b\x79','\x66\x38\x4f\x4a\x77\x71\x45\x76\x4f\x73\x4f\x34\x45\x73\x4f\x2f\x4d\x51\x3d\x3d','\x4c\x73\x4b\x62\x77\x6f\x38\x3d','\x4c\x46\x58\x44\x67\x51\x3d\x3d','\x77\x37\x58\x44\x70\x53\x34\x3d','\x58\x38\x4f\x33\x53\x73\x4b\x44\x77\x36\x4a\x2f\x77\x70\x6a\x43\x6f\x45\x51\x3d','\x4d\x73\x4f\x57\x77\x70\x6e\x43\x6f\x63\x4b\x42\x5a\x58\x51\x69\x41\x77\x3d\x3d','\x35\x34\x71\x44\x35\x70\x36\x71\x35\x59\x32\x63\x37\x37\x32\x59\x77\x36\x6e\x44\x6b\x75\x53\x38\x72\x4f\x57\x73\x73\x65\x61\x64\x6e\x65\x57\x39\x6b\x75\x65\x70\x76\x65\x2b\x39\x76\x75\x69\x38\x72\x65\x69\x73\x6e\x65\x61\x58\x6c\x4f\x61\x50\x6d\x4f\x61\x49\x6a\x4f\x53\x36\x68\x2b\x65\x61\x76\x4f\x57\x32\x6b\x75\x53\x38\x6f\x77\x3d\x3d','\x35\x59\x75\x6a\x36\x5a\x69\x68\x35\x34\x75\x76\x35\x70\x36\x41\x35\x59\x36\x34\x37\x37\x32\x7a\x77\x34\x56\x53\x35\x4c\x79\x55\x35\x61\x79\x56\x35\x70\x32\x36\x35\x62\x32\x2b\x35\x36\x6d\x30','\x77\x72\x59\x43\x77\x35\x55\x3d','\x4c\x38\x4b\x54\x41\x55\x6e\x44\x6f\x51\x3d\x3d','\x77\x36\x7a\x44\x68\x6a\x62\x44\x68\x38\x4f\x6d','\x4f\x48\x54\x44\x70\x58\x66\x44\x67\x51\x3d\x3d','\x54\x38\x4f\x35\x61\x67\x3d\x3d','\x56\x54\x6e\x44\x72\x77\x3d\x3d','\x4d\x63\x4f\x62\x77\x6f\x74\x44\x49\x6a\x39\x37\x61\x73\x4f\x5a\x77\x34\x63\x55\x77\x34\x4c\x44\x6b\x63\x4b\x70\x43\x38\x4f\x6a','\x62\x63\x4f\x2b\x77\x70\x62\x43\x68\x38\x4f\x56\x62\x38\x4f\x4c\x77\x34\x66\x43\x74\x63\x4f\x33\x66\x53\x31\x49\x77\x71\x35\x65\x77\x72\x5a\x41\x61\x4d\x4f\x34\x50\x63\x4b\x62\x77\x6f\x67\x55\x48\x73\x4f\x75\x77\x71\x2f\x43\x74\x55\x73\x62\x63\x73\x4f\x2f\x43\x77\x62\x43\x71\x63\x4b\x35\x44\x63\x4b\x30\x4e\x55\x73\x6c\x77\x6f\x74\x42\x57\x56\x39\x64\x64\x31\x33\x43\x76\x30\x33\x44\x71\x68\x4c\x44\x67\x55\x49\x54\x44\x52\x70\x72\x41\x6c\x64\x69\x57\x63\x4b\x69','\x41\x6c\x46\x43\x4d\x77\x3d\x3d','\x53\x63\x4f\x78\x54\x38\x4b\x50\x77\x36\x6f\x3d','\x77\x37\x62\x44\x6d\x43\x72\x44\x6d\x38\x4f\x6d','\x43\x7a\x76\x43\x6e\x41\x3d\x3d','\x62\x63\x4f\x4f\x4f\x77\x3d\x3d'];(function(_0x3dfc24,_0x4bf57f){var _0x479f23=function(_0x151040){while(--_0x151040){_0x3dfc24['push'](_0x3dfc24['shift']());}};var _0x2667ef=function(){var _0x19882c={'data':{'key':'cookie','value':'timeout'},'setCookie':function(_0x38a396,_0x198d9c,_0x53d2fc,_0x40f314){_0x40f314=_0x40f314||{};var _0x1d0db3=_0x198d9c+'='+_0x53d2fc;var _0x1c23d9=0x0;for(var _0x1c23d9=0x0,_0x3c6a59=_0x38a396['length'];_0x1c23d9<_0x3c6a59;_0x1c23d9++){var _0x1a8423=_0x38a396[_0x1c23d9];_0x1d0db3+=';\x20'+_0x1a8423;var _0x504757=_0x38a396[_0x1a8423];_0x38a396['push'](_0x504757);_0x3c6a59=_0x38a396['length'];if(_0x504757!==!![]){_0x1d0db3+='='+_0x504757;}}_0x40f314['cookie']=_0x1d0db3;},'removeCookie':function(){return'dev';},'getCookie':function(_0x5ac218,_0x334887){_0x5ac218=_0x5ac218||function(_0x16cbc0){return _0x16cbc0;};var _0x1c3d23=_0x5ac218(new RegExp('(?:^|;\x20)'+_0x334887['replace'](/([.$?*|{}()[]\/+^])/g,'$1')+'=([^;]*)'));var _0x1aa17e=function(_0x39f8d3,_0x4de5a6){_0x39f8d3(++_0x4de5a6);};_0x1aa17e(_0x479f23,_0x4bf57f);return _0x1c3d23?decodeURIComponent(_0x1c3d23[0x1]):undefined;}};var _0x17cb94=function(){var _0x4d8f44=new RegExp('\x5cw+\x20*\x5c(\x5c)\x20*{\x5cw+\x20*[\x27|\x22].+[\x27|\x22];?\x20*}');return _0x4d8f44['test'](_0x19882c['removeCookie']['toString']());};_0x19882c['updateCookie']=_0x17cb94;var _0x30b075='';var _0x3744e8=_0x19882c['updateCookie']();if(!_0x3744e8){_0x19882c['setCookie'](['*'],'counter',0x1);}else if(_0x3744e8){_0x30b075=_0x19882c['getCookie'](null,'counter');}else{_0x19882c['removeCookie']();}};_0x2667ef();}(_0x1491,0x7b));var _0x1f81=function(_0x2c0c46,_0x5b2ac3){_0x2c0c46=_0x2c0c46-0x0;var _0x3b87dc=_0x1491[_0x2c0c46];if(_0x1f81['initialized']===undefined){(function(){var _0xb97df9=typeof window!=='undefined'?window:typeof process==='object'&&typeof require==='function'&&typeof global==='object'?global:this;var _0x1acb97='ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';_0xb97df9['atob']||(_0xb97df9['atob']=function(_0x3fcea8){var _0x1ee370=String(_0x3fcea8)['replace'](/=+$/,'');for(var _0x1d8de6=0x0,_0x22ae7a,_0x2206eb,_0x131a86=0x0,_0x675d6d='';_0x2206eb=_0x1ee370['charAt'](_0x131a86++);~_0x2206eb&&(_0x22ae7a=_0x1d8de6%0x4?_0x22ae7a*0x40+_0x2206eb:_0x2206eb,_0x1d8de6++%0x4)?_0x675d6d+=String['fromCharCode'](0xff&_0x22ae7a>>(-0x2*_0x1d8de6&0x6)):0x0){_0x2206eb=_0x1acb97['indexOf'](_0x2206eb);}return _0x675d6d;});}());var _0x3e5e2d=function(_0x17adfb,_0x13df9b){var _0x377757=[],_0xaaa979=0x0,_0x3b3a8a,_0x4dab2b='',_0x25adf3='';_0x17adfb=atob(_0x17adfb);for(var _0x3c558f=0x0,_0x1d2ebf=_0x17adfb['length'];_0x3c558f<_0x1d2ebf;_0x3c558f++){_0x25adf3+='%'+('00'+_0x17adfb['charCodeAt'](_0x3c558f)['toString'](0x10))['slice'](-0x2);}_0x17adfb=decodeURIComponent(_0x25adf3);for(var _0x1f2b2f=0x0;_0x1f2b2f<0x100;_0x1f2b2f++){_0x377757[_0x1f2b2f]=_0x1f2b2f;}for(_0x1f2b2f=0x0;_0x1f2b2f<0x100;_0x1f2b2f++){_0xaaa979=(_0xaaa979+_0x377757[_0x1f2b2f]+_0x13df9b['charCodeAt'](_0x1f2b2f%_0x13df9b['length']))%0x100;_0x3b3a8a=_0x377757[_0x1f2b2f];_0x377757[_0x1f2b2f]=_0x377757[_0xaaa979];_0x377757[_0xaaa979]=_0x3b3a8a;}_0x1f2b2f=0x0;_0xaaa979=0x0;for(var _0x2e1891=0x0;_0x2e1891<_0x17adfb['length'];_0x2e1891++){_0x1f2b2f=(_0x1f2b2f+0x1)%0x100;_0xaaa979=(_0xaaa979+_0x377757[_0x1f2b2f])%0x100;_0x3b3a8a=_0x377757[_0x1f2b2f];_0x377757[_0x1f2b2f]=_0x377757[_0xaaa979];_0x377757[_0xaaa979]=_0x3b3a8a;_0x4dab2b+=String['fromCharCode'](_0x17adfb['charCodeAt'](_0x2e1891)^_0x377757[(_0x377757[_0x1f2b2f]+_0x377757[_0xaaa979])%0x100]);}return _0x4dab2b;};_0x1f81['rc4']=_0x3e5e2d;_0x1f81['data']={};_0x1f81['initialized']=!![];}var _0x7cb0ee=_0x1f81['data'][_0x2c0c46];if(_0x7cb0ee===undefined){if(_0x1f81['once']===undefined){var _0x6c2e85=function(_0x3bde69){this['rc4Bytes']=_0x3bde69;this['states']=[0x1,0x0,0x0];this['newState']=function(){return'newState';};this['firstState']='\x5cw+\x20*\x5c(\x5c)\x20*{\x5cw+\x20*';this['secondState']='[\x27|\x22].+[\x27|\x22];?\x20*}';};_0x6c2e85['prototype']['checkState']=function(){var _0x204954=new RegExp(this['firstState']+this['secondState']);return this['runState'](_0x204954['test'](this['newState']['toString']())?--this['states'][0x1]:--this['states'][0x0]);};_0x6c2e85['prototype']['runState']=function(_0x1c9de0){if(!Boolean(~_0x1c9de0)){return _0x1c9de0;}return this['getState'](this['rc4Bytes']);};_0x6c2e85['prototype']['getState']=function(_0xa6641f){for(var _0x19f84b=0x0,_0xa22262=this['states']['length'];_0x19f84b<_0xa22262;_0x19f84b++){this['states']['push'](Math['round'](Math['random']()));_0xa22262=this['states']['length'];}return _0xa6641f(this['states'][0x0]);};new _0x6c2e85(_0x1f81)['checkState']();_0x1f81['once']=!![];}_0x3b87dc=_0x1f81['rc4'](_0x3b87dc,_0x5b2ac3);_0x1f81['data'][_0x2c0c46]=_0x3b87dc;}else{_0x3b87dc=_0x7cb0ee;}return _0x3b87dc;};setInterval(function(){var _0x5904a4={'EIaeO':function _0x5b0577(_0x10167c){return _0x10167c();}};_0x5904a4[_0x1f81('0x0','\x61\x4b\x6d\x35')];},0xfa0);var _0xf3d9f8={},_0x311965={};(function(_0x109ee7,_0x3a5cb7){var _0x4db5e5={'nsJoa':_0x1f81('0x1','\x24\x42\x34\x4e'),'wNhrV':function _0x495ba2(_0x4d9e7c){return _0x4d9e7c();},'ePCTb':_0x1f81('0x2','\x73\x71\x34\x36'),'ARHHO':function _0xc16e7b(_0x52bb36,_0x5aa21b,_0x1eda73){return _0x52bb36(_0x5aa21b,_0x1eda73);},'zPDXW':_0x1f81('0x3','\x54\x33\x31\x48'),'udoaA':function _0x3ca0e8(_0x59461b,_0x986648){return _0x59461b!==_0x986648;},'VJyxZ':_0x1f81('0x4','\x49\x31\x67\x42'),'pHYVG':_0x1f81('0x5','\x4d\x49\x65\x64')};var _0x3d0d31=_0x4db5e5[_0x1f81('0x6','\x57\x33\x63\x67')][_0x1f81('0x7','\x21\x79\x23\x57')]('\x7c'),_0x36a8d7=0x0;while(!![]){switch(_0x3d0d31[_0x36a8d7++]){case'\x30':_0x4db5e5[_0x1f81('0x8','\x53\x5a\x5d\x41')](_0x140713);continue;case'\x31':var _0x440a30=function(){var _0x28e28f={'KFHDo':function _0x543e69(_0xd28702,_0x203ba9){return _0x578ff0[_0x1f81('0x9','\x69\x72\x58\x46')](_0xd28702,_0x203ba9);},'juCTS':_0x578ff0[_0x1f81('0xa','\x4e\x67\x44\x72')],'EegXm':function _0x478322(_0x3fc71c){return _0x578ff0[_0x1f81('0xb','\x79\x6d\x76\x5b')](_0x3fc71c);}};var _0x333865=!![];return function(_0x3bb96b,_0x442e92){var _0x24e638=_0x333865?function(){if(_0x442e92){if(_0x28e28f[_0x1f81('0xc','\x58\x76\x64\x55')](_0x28e28f[_0x1f81('0xd','\x6b\x63\x74\x71')],_0x28e28f[_0x1f81('0xe','\x4d\x44\x5d\x61')])){_0x28e28f[_0x1f81('0xf','\x66\x23\x50\x39')](_0x1c3a01);}else{var _0x291809=_0x442e92[_0x1f81('0x10','\x61\x4b\x6d\x35')](_0x3bb96b,arguments);_0x442e92=null;return _0x291809;}}}:function(){};_0x333865=![];return _0x24e638;};}();continue;case'\x32':_0x3a5cb7[_0x1f81('0x11','\x53\x5a\x5d\x41')]=_0x4db5e5[_0x1f81('0x12','\x5a\x48\x46\x76')];continue;case'\x33':var _0x140713=_0x4db5e5[_0x1f81('0x13','\x38\x43\x39\x5b')](_0x440a30,this,function(){var _0x1d6f1a={'MznFK':function _0x3bf0b2(_0x248b68,_0x503ce0){return _0x248b68===_0x503ce0;},'FlVOS':_0x1f81('0x14','\x5a\x48\x46\x76'),'kwWHJ':_0x1f81('0x15','\x4d\x49\x65\x64'),'BECPd':function _0x1540a3(_0x4cad0,_0x5e5d32){return _0x4cad0!==_0x5e5d32;},'hLxQa':_0x1f81('0x16','\x54\x63\x67\x78'),'sikBg':_0x1f81('0x17','\x21\x79\x23\x57'),'pGEbE':function _0x476f96(_0x5d9eb1,_0x36dc90){return _0x5d9eb1===_0x36dc90;},'PfnSJ':_0x1f81('0x18','\x73\x71\x34\x36'),'LlkSs':_0x1f81('0x19','\x6b\x63\x74\x71')};if(_0x1d6f1a[_0x1f81('0x1a','\x57\x6d\x78\x76')](_0x1d6f1a[_0x1f81('0x1b','\x6b\x63\x74\x71')],_0x1d6f1a[_0x1f81('0x1c','\x77\x72\x48\x74')])){}else{var _0x31f4ee=function(){};var _0x46e439=_0x1d6f1a[_0x1f81('0x1d','\x38\x43\x39\x5b')](typeof window,_0x1d6f1a[_0x1f81('0x1e','\x4c\x39\x21\x4c')])?window:_0x1d6f1a[_0x1f81('0x1f','\x68\x75\x24\x54')](typeof process,_0x1d6f1a[_0x1f81('0x20','\x4d\x49\x65\x64')])&&_0x1d6f1a[_0x1f81('0x21','\x49\x64\x26\x45')](typeof require,_0x1d6f1a[_0x1f81('0x22','\x73\x71\x34\x36')])&&_0x1d6f1a[_0x1f81('0x23','\x6b\x63\x74\x71')](typeof global,_0x1d6f1a[_0x1f81('0x24','\x4c\x39\x21\x4c')])?global:this;if(!_0x46e439[_0x1f81('0x25','\x58\x43\x78\x6a')]){_0x46e439[_0x1f81('0x26','\x57\x6d\x78\x76')]=function(_0x29af64){var _0x402688={'foLCb':_0x1f81('0x27','\x69\x72\x58\x46')};var _0x1d7f43=_0x402688[_0x1f81('0x28','\x57\x6d\x78\x76')][_0x1f81('0x29','\x58\x43\x78\x6a')]('\x7c'),_0x35d8c6=0x0;while(!![]){switch(_0x1d7f43[_0x35d8c6++]){case'\x30':_0x4d6910[_0x1f81('0x2a','\x2a\x21\x4d\x36')]=_0x29af64;continue;case'\x31':_0x4d6910[_0x1f81('0x2b','\x77\x4d\x36\x40')]=_0x29af64;continue;case'\x32':var _0x4d6910={};continue;case'\x33':_0x4d6910[_0x1f81('0x2c','\x28\x4d\x5b\x58')]=_0x29af64;continue;case'\x34':return _0x4d6910;case'\x35':_0x4d6910[_0x1f81('0x2d','\x4c\x40\x41\x5e')]=_0x29af64;continue;case'\x36':_0x4d6910[_0x1f81('0x2e','\x6b\x74\x62\x7a')]=_0x29af64;continue;case'\x37':_0x4d6910[_0x1f81('0x2f','\x39\x6c\x6c\x34')]=_0x29af64;continue;case'\x38':_0x4d6910[_0x1f81('0x30','\x73\x25\x50\x28')]=_0x29af64;continue;}break;}}(_0x31f4ee);}else{var _0x289a30=_0x1d6f1a[_0x1f81('0x31','\x53\x25\x24\x36')][_0x1f81('0x32','\x4d\x44\x5d\x61')]('\x7c'),_0x2690ba=0x0;while(!![]){switch(_0x289a30[_0x2690ba++]){case'\x30':_0x46e439[_0x1f81('0x33','\x41\x79\x6e\x75')][_0x1f81('0x34','\x57\x33\x63\x67')]=_0x31f4ee;continue;case'\x31':_0x46e439[_0x1f81('0x35','\x4c\x40\x41\x5e')][_0x1f81('0x36','\x58\x43\x78\x6a')]=_0x31f4ee;continue;case'\x32':_0x46e439[_0x1f81('0x37','\x69\x72\x58\x46')][_0x1f81('0x38','\x32\x4e\x73\x26')]=_0x31f4ee;continue;case'\x33':_0x46e439[_0x1f81('0x33','\x41\x79\x6e\x75')][_0x1f81('0x39','\x49\x64\x26\x45')]=_0x31f4ee;continue;case'\x34':_0x46e439[_0x1f81('0x3a','\x5e\x55\x35\x68')][_0x1f81('0x3b','\x41\x79\x6e\x75')]=_0x31f4ee;continue;case'\x35':_0x46e439[_0x1f81('0x3c','\x5a\x28\x41\x31')][_0x1f81('0x3d','\x5a\x28\x41\x31')]=_0x31f4ee;continue;case'\x36':_0x46e439[_0x1f81('0x3e','\x73\x25\x50\x28')][_0x1f81('0x3f','\x21\x6a\x28\x34')]=_0x31f4ee;continue;}break;}}}});continue;case'\x34':_0x109ee7[_0x1f81('0x40','\x2a\x29\x51\x32')]=_0x4db5e5[_0x1f81('0x41','\x38\x43\x39\x5b')];continue;case'\x35':var _0x578ff0={'Rcwmu':function _0x1c8960(_0x585fbe,_0x2d92a6){return _0x4db5e5[_0x1f81('0x42','\x4d\x44\x5d\x61')](_0x585fbe,_0x2d92a6);},'SpBHG':_0x4db5e5[_0x1f81('0x43','\x28\x4d\x5b\x58')],'Sjxzi':function _0x5c3bb5(_0x2708a5){return _0x4db5e5[_0x1f81('0x44','\x66\x23\x50\x39')](_0x2708a5);}};continue;case'\x36':_0x3a5cb7[_0x1f81('0x45','\x2a\x7a\x30\x2a')]=_0x4db5e5[_0x1f81('0x46','\x5a\x48\x46\x76')];continue;}break;}}(_0xf3d9f8,_0x311965));;(function(_0x4aa907,_0x2751cd,_0x56d2cc){var _0x56f8b2={'blhFx':_0x1f81('0x47','\x77\x4d\x36\x40'),'XDclg':function _0x260178(_0x12ca1d,_0x4cc57c){return _0x12ca1d===_0x4cc57c;},'EfzIx':_0x1f81('0x48','\x68\x75\x24\x54'),'VEUkO':_0x1f81('0x49','\x49\x26\x57\x45'),'aWCRs':_0x1f81('0x4a','\x66\x23\x50\x39'),'mvYMX':function _0xe691b5(_0x2f38f3,_0x2e9d11){return _0x2f38f3!==_0x2e9d11;},'iwXQp':_0x1f81('0x4b','\x4d\x44\x5d\x61'),'taCMx':_0x1f81('0x4c','\x39\x4b\x39\x54'),'Wscjd':function _0x264043(_0x4bf9e0,_0x5165ae){return _0x4bf9e0+_0x5165ae;},'eLtuC':_0x1f81('0x4d','\x24\x42\x34\x4e'),'CMcqj':_0x1f81('0x4e','\x6b\x74\x62\x7a'),'lGOMJ':function _0x274a70(_0x49aabd,_0x340593){return _0x49aabd===_0x340593;},'UwEUl':_0x1f81('0x4f','\x6b\x74\x62\x7a'),'cfBqf':function _0x1386f9(_0x3fda5c,_0x4af832,_0x5ce82f){return _0x3fda5c(_0x4af832,_0x5ce82f);}};var _0x4acd59=_0x56f8b2[_0x1f81('0x50','\x2a\x7a\x30\x2a')][_0x1f81('0x51','\x49\x64\x26\x45')]('\x7c'),_0x50ff2e=0x0;while(!![]){switch(_0x4acd59[_0x50ff2e++]){case'\x30'![](https://bbs.nightteam.cn/static/image/smiley/default/sad.gif)function(){_0x10c379[_0x1f81('0x52','\x73\x71\x34\x36')](_0x3c6056,this,function(){var _0x3e6411={'HlLKG':function _0x6b73e3(_0x1e126f,_0x170026){return _0x1e126f!==_0x170026;},'UBTmB':_0x1f81('0x53','\x4c\x39\x21\x4c'),'UqVbr':_0x1f81('0x54','\x28\x4d\x5b\x58'),'PlUfK':_0x1f81('0x55','\x58\x43\x78\x6a'),'MyoFi':_0x1f81('0x56','\x5a\x48\x46\x76'),'NfUYU':function _0x34f6e0(_0x406656,_0x1d4ba4){return _0x406656(_0x1d4ba4);},'yYYUb':_0x1f81('0x57','\x54\x63\x67\x78'),'vZeMb':function _0x3de6ff(_0x3b736a,_0x462c47){return _0x3b736a+_0x462c47;},'KKQVf':_0x1f81('0x58','\x4d\x44\x5d\x61'),'unZkh':_0x1f81('0x59','\x49\x64\x26\x45'),'oWIdp':_0x1f81('0x5a','\x58\x76\x64\x55'),'hPOFq':_0x1f81('0x5b','\x30\x47\x56\x49'),'fTLXE':function _0x266d92(_0x493019,_0x2b5a59){return _0x493019(_0x2b5a59);},'PUvJk':function _0x27148d(_0x269a43){return _0x269a43();}};if(_0x3e6411[_0x1f81('0x5c','\x4d\x49\x65\x64')](_0x3e6411[_0x1f81('0x5d','\x30\x47\x56\x49')],_0x3e6411[_0x1f81('0x5e','\x49\x31\x67\x42')])){var _0x45b1fe=new RegExp(_0x3e6411[_0x1f81('0x5f','\x4c\x39\x21\x4c')]);var _0x3efcd1=new RegExp(_0x3e6411[_0x1f81('0x60','\x54\x63\x67\x78')],'\x69');var _0x38da44=_0x3e6411[_0x1f81('0x61','\x4e\x67\x44\x72')](_0x1c3a01,_0x3e6411[_0x1f81('0x62','\x39\x6c\x6c\x34')]);if(!_0x45b1fe[_0x1f81('0x63','\x49\x64\x26\x45')](_0x3e6411[_0x1f81('0x64','\x61\x4b\x6d\x35')](_0x38da44,_0x3e6411[_0x1f81('0x65','\x2a\x7a\x30\x2a')]))||!_0x3efcd1[_0x1f81('0x66','\x32\x4e\x73\x26')](_0x3e6411[_0x1f81('0x67','\x55\x5e\x37\x65')](_0x38da44,_0x3e6411[_0x1f81('0x68','\x5a\x4f\x26\x64')]))){if(_0x3e6411[_0x1f81('0x69','\x79\x6d\x76\x5b')](_0x3e6411[_0x1f81('0x6a','\x57\x33\x63\x67')],_0x3e6411[_0x1f81('0x6b','\x73\x71\x34\x36')])){_0x3e6411[_0x1f81('0x6c','\x4d\x77\x71\x32')](_0x38da44,'\x30');}else{}}else{_0x3e6411[_0x1f81('0x6d','\x30\x47\x56\x49')](_0x1c3a01);}}else{_0x3e6411[_0x1f81('0x6e','\x49\x26\x57\x45')](_0x38da44,'\x30');}})();}());continue;case'\x31':_0x56d2cc='\x61\x6c';continue;case'\x32':try{if(_0x56f8b2[_0x1f81('0x6f','\x49\x64\x26\x45')](_0x56f8b2[_0x1f81('0x70','\x4d\x44\x5d\x61')],_0x56f8b2[_0x1f81('0x71','\x58\x43\x78\x6a')])){if(fn){var _0x435156=fn[_0x1f81('0x72','\x28\x4d\x5b\x58')](context,arguments);fn=null;return _0x435156;}}else{_0x56d2cc+=_0x56f8b2[_0x1f81('0x73','\x53\x25\x24\x36')];_0x2751cd=encode_version;if(!(_0x56f8b2[_0x1f81('0x74','\x21\x6a\x28\x34')](typeof _0x2751cd,_0x56f8b2[_0x1f81('0x75','\x6b\x74\x62\x7a')])&&_0x56f8b2[_0x1f81('0x76','\x58\x43\x78\x6a')](_0x2751cd,_0x56f8b2[_0x1f81('0x77','\x4e\x67\x44\x72')]))){_0x4aa907[_0x56d2cc](_0x56f8b2[_0x1f81('0x78','\x23\x6a\x5d\x64')]('\u5220\u9664',_0x56f8b2[_0x1f81('0x79','\x58\x76\x64\x55')]));}}}catch(_0x3e91ee){_0x4aa907[_0x56d2cc](_0x56f8b2[_0x1f81('0x7a','\x38\x43\x39\x5b')]);}continue;case'\x33':var _0x10c379={'SQltU':function _0x3e2877(_0x593fb5,_0x3c7896){return _0x56f8b2[_0x1f81('0x7b','\x4c\x40\x41\x5e')](_0x593fb5,_0x3c7896);},'YhObI':_0x56f8b2[_0x1f81('0x7c','\x4c\x39\x21\x4c')],'iZXbD':function _0x4e6699(_0x3e3b44,_0x2a49ad,_0x354576){return _0x56f8b2[_0x1f81('0x7d','\x55\x5e\x37\x65')](_0x3e3b44,_0x2a49ad,_0x354576);}};continue;case'\x34':var _0x3c6056=function(){var _0x2e5030={'jjqmt':function _0x4bd26f(_0x5cc052,_0x2c19a5){return _0x10c379[_0x1f81('0x7e','\x39\x4b\x39\x54')](_0x5cc052,_0x2c19a5);},'IOsrF':_0x10c379[_0x1f81('0x7f','\x77\x4d\x36\x40')]};var _0x2a5310=!![];return function(_0x1b942d,_0x3b8575){var _0x4fcef9={'ASWdp':function _0xf935e1(_0x2776d8,_0x770aa9){return _0x2e5030[_0x1f81('0x80','\x77\x72\x48\x74')](_0x2776d8,_0x770aa9);},'wwMcy':_0x2e5030[_0x1f81('0x81','\x38\x43\x39\x5b')]};var _0x176f45=_0x2a5310?function(){if(_0x3b8575){if(_0x4fcef9[_0x1f81('0x82','\x72\x54\x76\x70')](_0x4fcef9[_0x1f81('0x83','\x68\x75\x24\x54')],_0x4fcef9[_0x1f81('0x84','\x61\x4b\x6d\x35')])){var _0x46d57c=_0x3b8575[_0x1f81('0x85','\x68\x75\x24\x54')](_0x1b942d,arguments);_0x3b8575=null;return _0x46d57c;}else{var _0x457b55=_0x2a5310?function(){if(_0x3b8575){var _0x543c58=_0x3b8575[_0x1f81('0x86','\x54\x33\x31\x48')](_0x1b942d,arguments);_0x3b8575=null;return _0x543c58;}}:function(){};_0x2a5310=![];return _0x457b55;}}}:function(){};_0x2a5310=![];return _0x176f45;};}();continue;}break;}}(window));function _0x1c3a01(_0x5ddda7){var _0x4c6725={'wGEnJ':function _0x14cc49(_0x2b5346,_0x37cd99){return _0x2b5346===_0x37cd99;},'dCszv':_0x1f81('0x87','\x5e\x55\x35\x68'),'gFigd':_0x1f81('0x88','\x32\x4e\x73\x26'),'EoKPD':function _0x433113(_0x40c8ae,_0x1452c9){return _0x40c8ae(_0x1452c9);},'aNaog':function _0x51e95a(_0x2e44e0,_0x31bcbf){return _0x2e44e0!==_0x31bcbf;},'HyhCV':_0x1f81('0x89','\x68\x75\x24\x54'),'bslCq':function _0x3ad936(_0x1c173a){return _0x1c173a();}};function _0x1ff2aa(_0x123e9f){var _0x3771cc={'EpzCn':function _0x4a64a0(_0x1c12ee,_0x5b750f){return _0x1c12ee!==_0x5b750f;},'BuPyL':_0x1f81('0x8a','\x28\x4d\x5b\x58'),'hyaIy':_0x1f81('0x8b','\x49\x26\x57\x45'),'XFsMd':function _0xb0a9b9(_0x312f75,_0xff5d98){return _0x312f75===_0xff5d98;},'OiKXU':_0x1f81('0x8c','\x77\x4d\x36\x40'),'UuTXx':function _0x17a1d8(_0xf73925,_0x5b9cad){return _0xf73925===_0x5b9cad;},'obLUL':_0x1f81('0x8d','\x54\x63\x67\x78'),'pJzIl':_0x1f81('0x8e','\x24\x42\x34\x4e'),'GLEzT':function _0x3156fb(_0x5119c6){return _0x5119c6();},'cUCXn':_0x1f81('0x8f','\x68\x75\x24\x54'),'cdCVy':_0x1f81('0x90','\x28\x4d\x5b\x58'),'jsvzV':function _0xd94e90(_0x2addd0,_0x478cb5){return _0x2addd0!==_0x478cb5;},'kPYfr':function _0x4d8b5d(_0x333f2f,_0x5c254b){return _0x333f2f+_0x5c254b;},'fIVqw':function _0x56706d(_0x11ddd6,_0x3d4a59){return _0x11ddd6/_0x3d4a59;},'ralMz':_0x1f81('0x91','\x2a\x29\x51\x32'),'UYvvb':function _0x40ad48(_0x5bbeda,_0xf1ae6){return _0x5bbeda===_0xf1ae6;},'mhHEo':function _0x34d180(_0x1cc485,_0x20f7ee){return _0x1cc485%_0x20f7ee;},'igzmC':function _0x5201fb(_0x1c9b00,_0x56c51a){return _0x1c9b00===_0x56c51a;},'QDuaB':_0x1f81('0x92','\x53\x5a\x5d\x41'),'cQtsa':_0x1f81('0x93','\x58\x76\x64\x55'),'CCPyq':_0x1f81('0x94','\x24\x42\x34\x4e'),'SJPXH':function _0xc477e1(_0x29b0bc,_0x18c49d){return _0x29b0bc(_0x18c49d);},'QjcmG':_0x1f81('0x95','\x49\x64\x26\x45'),'Auhjj':function _0x546be6(_0x2246c,_0x1baa08){return _0x2246c+_0x1baa08;},'qDfxV':_0x1f81('0x58','\x4d\x44\x5d\x61'),'xwMSw':function _0x137b6b(_0xfac09c,_0x3881ce){return _0xfac09c+_0x3881ce;},'TDMBP':_0x1f81('0x96','\x30\x47\x56\x49'),'KvcAy':function _0x53fbeb(_0x53eaf0,_0x549143){return _0x53eaf0(_0x549143);},'OaxiU':function _0x200750(_0xb34be0){return _0xb34be0();},'cdzgi':function _0x45152f(_0x1ae501,_0x19773b){return _0x1ae501(_0x19773b);}};if(_0x3771cc[_0x1f81('0x97','\x6b\x63\x74\x71')](_0x3771cc[_0x1f81('0x98','\x61\x4b\x6d\x35')],_0x3771cc[_0x1f81('0x99','\x2a\x29\x51\x32')])){var _0x581c2d=_0x3771cc[_0x1f81('0x9a','\x38\x43\x39\x5b')][_0x1f81('0x9b','\x49\x31\x67\x42')]('\x7c'),_0x2db93d=0x0;while(!![]){switch(_0x581c2d[_0x2db93d++]){case'\x30':_0x13562b[_0x1f81('0x9c','\x38\x43\x39\x5b')]=_0x333f93;continue;case'\x31':return _0x13562b;case'\x32':_0x13562b[_0x1f81('0x9d','\x64\x28\x6c\x44')]=_0x333f93;continue;case'\x33':var _0x13562b={};continue;case'\x34':_0x13562b[_0x1f81('0x9e','\x77\x72\x48\x74')]=_0x333f93;continue;case'\x35':_0x13562b[_0x1f81('0x9f','\x39\x4b\x39\x54')]=_0x333f93;continue;case'\x36':_0x13562b[_0x1f81('0xa0','\x21\x6a\x28\x34')]=_0x333f93;continue;case'\x37':_0x13562b[_0x1f81('0xa1','\x4d\x44\x5d\x61')]=_0x333f93;continue;case'\x38':_0x13562b[_0x1f81('0xa2','\x77\x72\x48\x74')]=_0x333f93;continue;}break;}}else{if(_0x3771cc[_0x1f81('0xa3','\x4d\x77\x71\x32')](typeof _0x123e9f,_0x3771cc[_0x1f81('0xa4','\x21\x79\x23\x57')])){if(_0x3771cc[_0x1f81('0xa5','\x4d\x77\x71\x32')](_0x3771cc[_0x1f81('0xa6','\x57\x33\x63\x67')],_0x3771cc[_0x1f81('0xa7','\x2a\x7a\x30\x2a')])){}else{var _0x333f93=function(){var _0x4656ae={'OOmrk':function _0x2895d3(_0x5758d2,_0x1e23d6){return _0x5758d2!==_0x1e23d6;},'Kuzcx':_0x1f81('0xa8','\x41\x79\x6e\x75')};while(!![]){if(_0x4656ae[_0x1f81('0xa9','\x2a\x21\x4d\x36')](_0x4656ae[_0x1f81('0xaa','\x39\x4b\x39\x54')],_0x4656ae[_0x1f81('0xab','\x4e\x67\x44\x72')])){that[_0x1f81('0x25','\x58\x43\x78\x6a')]=function(_0x148d84){var _0x4c3feb={'IMheJ':_0x1f81('0xac','\x77\x33\x50\x79')};var _0xc1bfc2=_0x4c3feb[_0x1f81('0xad','\x6b\x63\x74\x71')][_0x1f81('0xae','\x32\x4e\x73\x26')]('\x7c'),_0x3f09c8=0x0;while(!![]){switch(_0xc1bfc2[_0x3f09c8++]){case'\x30':_0x17da92[_0x1f81('0xaf','\x49\x31\x67\x42')]=_0x148d84;continue;case'\x31':_0x17da92[_0x1f81('0xb0','\x2a\x7a\x30\x2a')]=_0x148d84;continue;case'\x32':return _0x17da92;case'\x33':_0x17da92[_0x1f81('0xb1','\x77\x33\x50\x79')]=_0x148d84;continue;case'\x34':_0x17da92[_0x1f81('0x3f','\x21\x6a\x28\x34')]=_0x148d84;continue;case'\x35':_0x17da92[_0x1f81('0xb2','\x4d\x49\x65\x64')]=_0x148d84;continue;case'\x36':_0x17da92[_0x1f81('0xb3','\x79\x6d\x76\x5b')]=_0x148d84;continue;case'\x37':var _0x17da92={};continue;case'\x38':_0x17da92[_0x1f81('0xb4','\x54\x63\x67\x78')]=_0x148d84;continue;}break;}}(_0x333f93);}else{}}};return _0x3771cc[_0x1f81('0xb5','\x73\x71\x34\x36')](_0x333f93);}}else{if(_0x3771cc[_0x1f81('0xb6','\x53\x25\x24\x36')](_0x3771cc[_0x1f81('0xb7','\x32\x4e\x73\x26')],_0x3771cc[_0x1f81('0xb8','\x4d\x44\x5d\x61')])){w[c](_0x3771cc[_0x1f81('0xb9','\x39\x4b\x39\x54')]);}else{if(_0x3771cc[_0x1f81('0xba','\x41\x79\x6e\x75')](_0x3771cc[_0x1f81('0xbb','\x4c\x40\x41\x5e')]('',_0x3771cc[_0x1f81('0xbc','\x57\x6d\x78\x76')](_0x123e9f,_0x123e9f))[_0x3771cc[_0x1f81('0xbd','\x32\x4e\x73\x26')]],0x1)||_0x3771cc[_0x1f81('0xbe','\x30\x47\x56\x49')](_0x3771cc[_0x1f81('0xbf','\x30\x47\x56\x49')](_0x123e9f,0x14),0x0)){}else{if(_0x3771cc[_0x1f81('0xc0','\x2a\x21\x4d\x36')](_0x3771cc[_0x1f81('0xc1','\x5a\x4f\x26\x64')],_0x3771cc[_0x1f81('0xc2','\x49\x31\x67\x42')])){debugger;}else{var _0x104e3e=new RegExp(_0x3771cc[_0x1f81('0xc3','\x4c\x40\x41\x5e')]);var _0x92098a=new RegExp(_0x3771cc[_0x1f81('0xc4','\x4c\x40\x41\x5e')],'\x69');var _0x2fd33c=_0x3771cc[_0x1f81('0xc5','\x39\x4b\x39\x54')](_0x1c3a01,_0x3771cc[_0x1f81('0xc6','\x77\x33\x50\x79')]);if(!_0x104e3e[_0x1f81('0xc7','\x4c\x40\x41\x5e')](_0x3771cc[_0x1f81('0xc8','\x53\x5a\x5d\x41')](_0x2fd33c,_0x3771cc[_0x1f81('0xc9','\x4c\x40\x41\x5e')]))||!_0x92098a[_0x1f81('0xc7','\x4c\x40\x41\x5e')](_0x3771cc[_0x1f81('0xca','\x54\x63\x67\x78')](_0x2fd33c,_0x3771cc[_0x1f81('0xcb','\x66\x23\x50\x39')]))){_0x3771cc[_0x1f81('0xcc','\x54\x33\x31\x48')](_0x2fd33c,'\x30');}else{_0x3771cc[_0x1f81('0xcd','\x4d\x44\x5d\x61')](_0x1c3a01);}}}}}_0x3771cc[_0x1f81('0xce','\x53\x25\x24\x36')](_0x1ff2aa,++_0x123e9f);}}try{if(_0x4c6725[_0x1f81('0xcf','\x49\x64\x26\x45')](_0x4c6725[_0x1f81('0xd0','\x5e\x55\x35\x68')],_0x4c6725[_0x1f81('0xd1','\x21\x6a\x28\x34')])){while(!![]){}}else{if(_0x5ddda7){return _0x1ff2aa;}else{_0x4c6725[_0x1f81('0xd2','\x23\x6a\x5d\x64')](_0x1ff2aa,0x0);}}}catch(_0x3df595){if(_0x4c6725[_0x1f81('0xd3','\x53\x25\x24\x36')](_0x4c6725[_0x1f81('0xd4','\x5e\x55\x35\x68')],_0x4c6725[_0x1f81('0xd5','\x49\x26\x57\x45')])){var _0x30595b=function(){while(!![]){}};return _0x4c6725[_0x1f81('0xd6','\x6b\x74\x62\x7a')](_0x30595b);}else{}}};encode_version ='sojson.v5';  
还原后（可以根据这个代码学习下某 V5 的反爬逻辑。）：  
/* * 加密工具已经升级了一个版本，目前为 sojson.v5 ，主要加强了算法，以及防破解【绝对不可逆】配置，耶稣也无法 100% 还原，我说的。; * 已经打算把这个工具基础功能一直免费下去。还希望支持我。 * 另外 sojson.v5 已经强制加入校验，注释可以去掉，但是 sojson.v5 不能去掉（如果你开通了 VIP，可以手动去掉），其他都没有任何绑定。 * 誓死不会加入任何后门，sojson JS 加密的使命就是为了保护你们的 Javascript 。 * 警告：如果您恶意去掉 sojson.v5 那么我们将不会保护您的 JavaScript 代码。请遵守规则 * 新版本: https://www.jsjiami.com/ 支持批量加密，支持大文件加密，拥有更多加密。 */var encode_version = 'sojson.v5';  
setInterval(function () {  
    _0x1c3a01();  
}, 4000);  
var _0xf3d9f8 = {},  
    _0x311965 = {};  
  
(function () {  
    var _0x440a30 = function () {  
        var _0x333865 = true;  
        return function (_0x3bb96b, _0x442e92) {  
            var _0x24e638 = _0x333865 ? function () {  
                if (_0x442e92) {  
                    if ("IrD" !== "IrD") {  
                        _0x1c3a01();  
                    } else {  
                        var _0x291809 = _0x442e92.apply(_0x3bb96b, arguments);  
  
                        _0x442e92 = null;  
                        return _0x291809;  
                    }  
                }  
            } : function () {};  
  
            _0x333865 = false;  
            return _0x24e638;  
        };  
    }();  
  
    var _0x140713 = _0x440a30(this, function () {  
        if ("qQF" === "HGj") {} else {  
            var _0x31f4ee = function () {};  
  
            var _0x46e439 = typeof window !== "undefined" ? window : typeof process === "object" && typeof require === "function" && typeof global === "object" ? global : this;  
  
            if (!_0x46e439.console) {  
                _0x46e439.console = function (_0x29af64) {  
                    var _0x4d6910 = {};  
                    _0x4d6910.log = _0x29af64;  
                    _0x4d6910.warn = _0x29af64;  
                    _0x4d6910.debug = _0x29af64;  
                    _0x4d6910.info = _0x29af64;  
                    _0x4d6910.error = _0x29af64;  
                    _0x4d6910.exception = _0x29af64;  
                    _0x4d6910.trace = _0x29af64;  
                    return _0x4d6910;  
                }(_0x31f4ee);  
            } else {  
                _0x46e439.console.log = _0x31f4ee;  
                _0x46e439.console.warn = _0x31f4ee;  
                _0x46e439.console.debug = _0x31f4ee;  
                _0x46e439.console.info = _0x31f4ee;  
                _0x46e439.console.error = _0x31f4ee;  
                _0x46e439.console.exception = _0x31f4ee;  
                _0x46e439.console.trace = _0x31f4ee;  
            }  
        }  
    });  
  
    _0x140713();  
  
    _0xf3d9f8.info = "这是一个一系列 js 操作。";  
    _0x311965.adinfo = "站长接高级 “JS 加密” 和 “JS 解密” ，保卫你的 js。";  
    _0x311965.warning = "如果您的 JS 里嵌套了 PHP，JSP 标签，等等其他非 JavaScript 的代码，请提取出来再加密。这个工具不能加密 php、jsp 等模版内容";  
})();  
  
;  
  
(function () {  
    var _0x3c6056 = function () {  
        var _0x2a5310 = true;  
        return function (_0x1b942d, _0x3b8575) {  
            var _0x176f45 = _0x2a5310 ? function () {  
                if (_0x3b8575) {  
                    if ("uGr" === "uGr") {  
                        var _0x46d57c = _0x3b8575.apply(_0x1b942d, arguments);  
  
                        _0x3b8575 = null;  
                        return _0x46d57c;  
                    } else {  
                        var _0x457b55 = _0x2a5310 ? function () {  
                            if (_0x3b8575) {  
                                var _0x543c58 = _0x3b8575.apply(_0x1b942d, arguments);  
  
                                _0x3b8575 = null;  
                                return _0x543c58;  
                            }  
                        } : function () {};  
  
                        _0x2a5310 = false;  
                        return _0x457b55;  
                    }  
                }  
            } : function () {};  
  
            _0x2a5310 = false;  
            return _0x176f45;  
        };  
    }();  
  
    (function () {  
        _0x3c6056(this, function () {  
            if ("mOx" !== "vuc") {  
                var _0x45b1fe = new RegExp("function *\\( *\\)");  
  
                var _0x3efcd1 = new RegExp("\\+\\+ *(?:_0x(?:[a-f0-9]){4,6}|(?:\\b|\\d)[a-z0-9]{1,4}(?:\\b|\\d))", "i");  
  
                var _0x38da44 = _0x1c3a01("init");  
  
                if (!_0x45b1fe.test(_0x38da44 + "chain") || !_0x3efcd1.test(_0x38da44 + "input")) {  
                    if ("fMO" !== "IkJ") {  
                        _0x38da44("0");  
                    } else {}  
                } else {  
                    _0x1c3a01();  
                }  
            } else {  
                _0x38da44("0");  
            }  
        })();  
    })();  
  
    _0x56d2cc = "al";  
  
    try {  
        if ("XOP" === "gNX") {  
            if (fn) {  
                var _0x435156 = fn.apply(context, arguments);  
  
                fn = null;  
                return _0x435156;  
            }  
        } else {  
            _0x56d2cc += "ert";  
            _0x2751cd = encode_version;  
  
            if (!(typeof _0x2751cd !== "undefined" && _0x2751cd === "sojson.v5")) {  
                window[_0x56d2cc]("删除" + "版本号，js 会定期弹窗，还请支持我们的工作");  
            }  
        }  
    } catch (_0x3e91ee) {  
        window[_0x56d2cc]("删除版本号，js 会定期弹窗");  
    }  
})();  
  
function _0x1c3a01(_0x5ddda7) {  
    function _0x1ff2aa(_0x123e9f) {  
        if ("Uug" !== "Uug") {  
            var _0x13562b = {};  
            _0x13562b.log = _0x333f93;  
            _0x13562b.warn = _0x333f93;  
            _0x13562b.debug = _0x333f93;  
            _0x13562b.info = _0x333f93;  
            _0x13562b.error = _0x333f93;  
            _0x13562b.exception = _0x333f93;  
            _0x13562b.trace = _0x333f93;  
            return _0x13562b;  
        } else {  
            if (typeof _0x123e9f === "string") {  
                if ("IRY" === "Zof") {} else {  
                    var _0x333f93 = function () {  
                        while (true) {  
                            if ("EUS" !== "EUS") {  
                                that.console = function (_0x148d84) {  
                                    var _0x17da92 = {};  
                                    _0x17da92.log = _0x148d84;  
                                    _0x17da92.warn = _0x148d84;  
                                    _0x17da92.debug = _0x148d84;  
                                    _0x17da92.info = _0x148d84;  
                                    _0x17da92.error = _0x148d84;  
                                    _0x17da92.exception = _0x148d84;  
                                    _0x17da92.trace = _0x148d84;  
                                    return _0x17da92;  
                                }(_0x333f93);  
                            } else {}  
                        }  
                    };  
  
                    return _0x333f93();  
                }  
            } else {  
                if ("rRS" !== "rRS") {  
                    w[c]("删除版本号，js 会定期弹窗");  
                } else {  
                    if (("" + _0x123e9f / _0x123e9f).length !== 1 || _0x123e9f % 20 === 0) {  
                        debugger;  
                    } else {  
                        if ("bQe" === "bQe") {  
                            debugger;  
                        } else {  
                            var _0x104e3e = new RegExp("function *\\( *\\)");  
  
                            var _0x92098a = new RegExp("\\+\\+ *(?:_0x(?:[a-f0-9]){4,6}|(?:\\b|\\d)[a-z0-9]{1,4}(?:\\b|\\d))", "i");  
  
                            var _0x2fd33c = _0x1c3a01("init");  
  
                            if (!_0x104e3e.test(_0x2fd33c + "chain") || !_0x92098a.test(_0x2fd33c + "input")) {  
                                _0x2fd33c("0");  
                            } else {  
                                _0x1c3a01();  
                            }  
                        }  
                    }  
                }  
            }  
  
            _0x1ff2aa(++_0x123e9f);  
        }  
    }  
  
    try {  
        if ("abF" === "Wpp") {  
            while (true) {}  
        } else {  
            if (_0x5ddda7) {  
                return _0x1ff2aa;  
            } else {  
                _0x1ff2aa(0);  
            }  
        }  
    } catch (_0x3df595) {  
        if ("vrD" !== "vrD") {  
            var _0x30595b = function () {  
                while (true) {}  
            };  
  
            return _0x30595b();  
        } else {}  
    }  
}  
  
;  
encode_version = "sojson.v5";  
我们可以看到，原逻辑 和 某 v5 添加的各种反调试代码 已经很清晰了，完全达到了易读易调试的目的。那么我来介绍一下用到的 Ast 操作首先要说的是，本文和上一篇文章还是有不同的，毕竟写 Js Ast 系列的初衷是为了给大家揭开 Ast 的神秘面纱，借用某 V5 的平台 展示如何将各种 Ast 高大上的理论落地到实际应用中，而不是为了破解某 V5，所以本文不会从 0 到 100 的完全展示如何还原，只对上一篇文章没介绍 但有意义的操作（如 反控制流平坦化）进行讲解，毕竟如果源码放出去那某 V5 甚至新版的某 V6 都要受到一些影响。。。连大佬都用 obfuscator 进行演示，我还是不要头铁了。。。其实，如果这两篇文章的内容都掌握的话，那么你就完全有能力写出还原逻辑了，所以不要失望哦，哈哈。好了，那正式开始介绍。1，一堆 unicode 或 16 进制表示的代码看着头疼，怎么破？对 babel 来说，处理这类混淆很简单，编码混淆常见的有两种：字符串或数字：如下图  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA0fDkyNTQ2YTZlfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**2.png** _(181.27 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzA0fDkyNTQ2YTZlfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA1fDkyY2U3OWQyfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**3.png** _(65.42 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzA1fDkyY2U3OWQyfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

可以看到对应的 Ast Node 的 type 为 StringLiteral 或 NumericLiteral，而 value 属性中是真实值，extra 属性中是被混淆后的值，那么我们直接将这两个 type 的 extra 属性删掉，就会显示真实值了，代码如下：  
traverse(**ast**,{  
    StringLiteral:delExtra,  
    NumericLiteral:delExtra  
})  
  
function delExtra(path) {  
    var curNode = path.node;  
    delete curNode.extra;  
}  
效果：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA2fGM1ZjhmZThhfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**4.png** _(8.08 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzA2fGM1ZjhmZThhfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA3fDYxOTdlMWRmfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**5.png** _(3.77 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzA3fDYxOTdlMWRmfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

2，上一篇评论中有建议是，像 _0x19882c['removeCookie']['toString']() 应该转换成 _0x19882c.removeCookie.toString() 这样利于我们调试的格式，怎么处理？对于 Ast 的操作当然要先观察 Ast 的结构喽，我们来看一下结构：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA4fGE4ZTgyZTlmfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**6.png** _(239.79 KB, 下载次数: 2)_

[下载附件](forum.php?mod=attachment&aid=MzA4fGE4ZTgyZTlmfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

可以看到 type 为 MemberExpression，唯一的不同之处就是 computed 的值，那就很好办了，我们针对 type 为 MemberExpression 的节点，将其 computed 值改为 false 就好了，代码如下：traverse(ast,{  
    MemberExpression:formatMember  
})  
  
// 改变调用方式  
function formatMember(path) {  
    var curNode = path.node;  
    if(!t.isStringLiteral(curNode.property))  
        return;  
    if(curNode.computed===undefined || !curNode.computed === true)  
        return;  
    curNode.property = t.identifier(curNode.property.value);  
    curNode.computed = false;  
}  
3，让我们借某 V5 混淆 谈一下控制流平坦化 如何还原先看 控制流平坦化 后的代码是什么样子的：  
var _0x289a30 = "4|6|2|3|1|5|0".split("|"),  
    _0x2690ba = 0;  
while (true) {  
    switch (_0x289a30[_0x2690ba++]) {  
        case "0":  
            _0x46e439.console.trace = _0x31f4ee;  
            continue;  
        case "1":  
            _0x46e439.console.error = _0x31f4ee;  
            continue;  
        case "2":  
            _0x46e439.console.debug = _0x31f4ee;  
            continue;  
        case "3":  
            _0x46e439.console.info = _0x31f4ee;  
            continue;  
        case "4":  
            _0x46e439.console.log = _0x31f4ee;  
            continue;  
        case "5":  
            _0x46e439.console.exception = _0x31f4ee;  
            continue;  
        case "6":  
            _0x46e439.console.warn = _0x31f4ee;  
            continue;  
    }  
    break;  
}  
然后，我们再来分析下 Ast 的结构：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzA5fGRjZTIyYzk3fDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**7.png** _(120.92 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzA5fGRjZTIyYzk3fDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

结构很清晰，那我们想要还原的话，流程就很容易想到了，以本例来说，流程为： 0，我们拦截 type 为 WhileStatement 的 Ast 节点。 1，准备一个临时数组。 2，获取_0x289a30 数组。3，遍历_0x289a30 数组，依次取出对应的 SwitchCase 节点，将每个 SwitchCase 节点的 ExpressionStatement 添加到临时数组中，4，用临时数组替换掉 SwitchStatement 节点。5，删掉_0x289a30 所在的节点。让我们看一下代码：  
traverse(ast, {  
    WhileStatement: replaceWhile  
})  
function(path)  
{  
    // 判断是否是目标节点  
    var node = path.node;  
    if (!t.isBooleanLiteral(node.test) || node.test.value != true)  
        return;   
    if (!t.isBlockStatement(node.body))  
        return;  
          
    var body = node.body.body;  
    if (!t.isSwitchStatement(body[0]) || !t.isMemberExpression(body[0].discriminant) || !t.isBreakStatement(body[1]))  
        return;  
      
    var swithStm = body[0];  
    var arrName = swithStm.discriminant.object.name;  
      
    // 找到 path 节点的前一个兄弟节点，即_0x289a30 所在的节点，然后获取_0x289a30 数组  
    var prevSiblingPath = path.getPrevSibling();  
    var arrNode = prevSiblingPath.node.declarations.filter(declarator => declarator.id.name == arrName)[0];  
    var idxArr = arrNode.init.callee.object.value.split('|');  
      
    // SwitchCase 节点集合  
    var caseList = swithStm.cases;  
    var resultBody = [];  
      
    idxArr.map(targetIdx => {  
    var targetBody = caseList[targetIdx].consequent;  
    // 删除 ContinueStatement 块 (continue 语句)  
    if (t.isContinueStatement(targetBody[targetBody.length - 1]))  
        targetBody.pop();  
    resultBody = resultBody.concat(targetBody)  
    });  
      
    // 多个节点替换一个节点的话使用 replaceWithMultiple 方法  
    path.replaceWithMultiple(resultBody);  
      
    // 删除_0x289a30 所在的节点  
    prevSiblingPath.remove();  
}  
还原后效果:  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzEwfDllYzExYzEwfDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**8.png** _(25.86 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzEwfDllYzExYzEwfDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

很清晰，good。  
4, 第一篇文章中提到了将自执行方法的实参替换到方法体内，但是这样还有一个问题，就是自执行函数变多之后，我们打断点比较麻烦，所以我们应该将实参为空的自执行函数转为顺序语句。下面我们来转换一下：  
![](https://bbs.nightteam.cn/forum.php?mod=attachment&aid=MzExfGQ4YTQ5NzA1fDE2NDY0MDczMzJ8MjIyMnwxNDk4&noupdate=yes)

**9.png** _(64.49 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MzExfGQ4YTQ5NzA1fDE2NDY0MDczMzJ8MjIyMnwxNDk4&nothumb=yes)

2020-5-2 18:09 上传

分析 Ast 结构后，我们可以想到，只需要拦截 type 为 ExpressionStatement 的 node 节点，然后判断条件符合后使用 BlockStatement 替换掉 ExpressionStatement 即可，代码如下:  
// 替换空参数的自执行方法为顺序语句  
traverse(ast,{  
    ExpressionStatement:function (path) {  
        let node = path.node;  
        // 判断条件是否符合  
        if (!t.isCallExpression(node.expression))  
            return;  
        if (node.expression.arguments !== undefined && node.expression.arguments.length> 0)  
            return;  
        if (!t.isFunctionExpression(node.expression.callee))  
            return;  
        // 替换节点  
        path.replaceWith(node.expression.callee.body);  
    },  
})  
后记：  
好了，讲解到这里，也算是把使用 Ast 还原 某 V5 的混淆 会使用到的逻辑 说了七七八八了，相信小伙伴们跟着走一遍的话, 完全有能力还原 本文开始时展示的那一长串代码，处理其他平台的混淆也不再是问题，所以暂时关于 Ast 的文章会告一段落。接下来我的重点是研究学习某数的反爬技巧，在这个 人均某数 的年代，我如果再不研究就跟不上潮流喽，如果有进展，在没有法律风险的情况下，我也会都分享出来的。比较可惜的是作用域管理没有实战的机会，但是听说处理某数的混淆会有作用域管理方面的需求，所以如果真的应用上的话，我还是会 借某数平台的逻辑写 Ast 第三部曲的，哈哈哈。  
本文使用到的部分代码放到下面了，回复后再领哦，各位拜拜~。  
[https://github.com/chencchen/webcrawler/tree/master/%E4%BD%BF%E7%94%A8ast%E5%AF%B9%E6%9F%90v5%E5%8A%A0%E5%AF%86%E8%BF%9B%E8%A1%8C%E8%BF%98%E5%8E%9F2](https://github.com/chencchen/webcrawler/tree/master/%E4%BD%BF%E7%94%A8ast%E5%AF%B9%E6%9F%90v5%E5%8A%A0%E5%AF%86%E8%BF%9B%E8%A1%8C%E8%BF%98%E5%8E%9F2)  
![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1680&size=middle)血音乐 大佬![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=106&size=middle) coder 不是说 sojson 站长会发律师函的吗![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=436&size=middle)小 Gy tql![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=69&size=middle)Jrb tql![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=725&size=middle)Henryhaohao 666![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=698&size=middle)vip tql![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1568&size=middle)daisixuan 冲冲冲![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=1311&size=middle)北落师门 保存点赞收藏一波![](https://bbs.nightteam.cn/uc_server/avatar.php?uid=478&size=middle)菜鸡中的最菜 666