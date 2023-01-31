> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.betamao.me](https://blog.betamao.me/posts/2021/some-ebook-decrypt/)

> 很久没分析了 android 应用了，再没有大学时那种一天不吃饭干一个 APP 的激情了... 最后，能用就行吧！

换了 MBA 后习惯装上阅读器，但是第一次用就被惊到了，不知道这是有意为之还是出了 BUG，反正等了几个月官方也不修，没办法就打算把书提出来用自带的 BOOK 阅读了：  

![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632371846910-cb0cbf7c-c690-4b2d-a2e5-0a63ae8bf4c7.png)

> ⚠️⚠️⚠️注：本文只是为了技术交流，有兴趣可以自己分析自己用，本文不提供脚本或电子书下载！另外，为了防止被直接拿去下载，本文忽略了一些小细节，解密需要分析整个过程！

安卓逆向基础
------

按惯例，补点基础，然鹅忘得差不多了，以后想到什么再补吧

### 工具

工欲那啥必须那啥，Android 分析的工具太多了，有些小工具会打包成工具箱，比如 AndroidKiller，APKIDE 等，还有很多是单独的工具或脚本，下面就简单记录一些常用的，还有很多很多很多会在某次分析中用到，就不写了...

#### 运行

1.  手机，你需要一台小米 10 嗷欻~~ 才怪~~，太新的手机也不好，我用的小米 2 特方便。不然的话用模拟器也可以，比如[蓝叠](https://www.bluestacks.com/)，可使用修改器 [root](https://bstweaker.tk/)，需要注意的是这次分析的爱屁屁没有 x86 的动态库，而 frida 对模拟器只能使用 x86 的 server，并且若要 hook arm 的库需使用 [realm](https://frida.re/news/2021/02/10/frida-14-2-released/)。
2.  [ADB](https://developer.android.com/studio/command-line/adb) 等工具，简单说明下 ADB 包括三部分，本电脑使用的 adb 和 adb-server 及被调试手机运行的 adbd，执行 adb 的大部分命令会启动并连接 adb-server，由它通过 [usb / 网络](https://developer.android.com/studio/run/device)与 adbd 通信，adbd 执行实际命令，默认 adbd 运行权限也不高，可用 adb root 使它以 root 身份启动，其他常用命令可见 [adbsehll](https://adbshell.com/)。那一套常用的还有 [Logcat](https://developer.android.com/studio/command-line/logcat?hl=zh-cn)，[DDMS](https://developer.android.com/studio/profile/monitor) 等。

#### 分析

[JEB](https://www.pnfsoftware.com/) 和 [GDA](http://www.gda.wiki:9090/index.php) 差不多主要用于分析 Java 层代码，[IDA](https://hex-rays.com/ida-pro/) 用于分析和调试 so 文件

#### 网络

这里分两类，一类是抓包的，比如 [r0capture](https://github.com/r0ysue/r0capture)，[proxydroid](https://github.com/madeye/proxydroid)，另一类是分析的，比如 [charles](https://www.charlesproxy.com/)，[fiddler](https://proxyman.io/posts/2019-10-26-Alternatives-for-charles-proxy-and-wireshark)，[wireshark](https://www.wireshark.org/)，[burpsuite](https://portswigger.net/burp) 等

#### 自动脱壳

[FRIDA-DEXDump](https://github.com/hluwa/FRIDA-DEXDump) 和 [Frida-Apk-Unpack](https://github.com/GuoQiang1993/Frida-Apk-Unpack) 都是使用 Frida 实现的自动化脱壳脚本，而 [FART](https://github.com/hanbinglengyue/FART) 是基于 Android 6.0 实现的 ART 环境下通过主动调用的自动化脱壳方案。

#### 调试插桩

[xposed](https://repo.xposed.info/) 和 [cydia](http://www.cydiasubstrate.com/) 是比较经典的 hook 框架，前者主要用于 Java 层而后者用于 Native 层，现在更多的会使用 [Frida](https://frida.re/)，它动态二进制插桩工具，可以做 objc，java，native 的 HOOK，可以看到上面很多工具是基于它的，之后有时间会单独出一篇分析文章。

### 鸡识

安卓就 Linux 上加个手机框架，框架主要用 Java 虚拟机 (dalvik/art) 运行程序，和 hotspot 的栈机不同，它是基于寄存器的，也不使用 Java 标准的 class 结构，但是很类似，比如它的汇编 smali 其实很像普通的字节码，只是说里面有寄存器了，其实这里面的东西都是可以相互转化的。另外程序也可以运行 Native 的程序或调用 Native 的库，一半的爱屁屁就是 Java＋支持 JNI 语言 (比如 C/C++) 写结合(可参考 [cxm](https://www.zybuluo.com/cxm-2016/note/563686))，显然 Java 的字节码文件很容易反编译为 Java 代码，这丝毫不影响阅读，而其他编译为 Native 代码的部分就可以把客户端上那套保护机制直接套上，况且去符号的二进制文件本身分析就比较困难，所以现在的保护思路也是 Java 转 Native，而之前的那几代在 Java 层面的保护也就是代码混淆，代码抽取，前者我们木的办法咯但是代码抽取的写个脚本也能弄出来，更多可以看 OWASP 的[文档](https://mobile-security.gitbook.io/mobile-security-testing-guide/)。  

说回破解本身，大体和客户端一致，先熟悉下程序，搞清楚目标是啥，再静态分析看看整体长啥样，找找关键点，关键点也是那几拳，什么抓包，字符串，配置，元信息，定位到关键点再扩展一顿操作就🆗啦！  
​  

### 调试

分为 Java 的和 Native 的，前者一般用 idea/as 调它的 smali 代码，应用在启动时若全局设置了 ro.debuggable=1 或本地的 AndroidManifest.xml 设置了 android:debuggable="true" 则会创建调试线程，用户则可以附加调试；

```
java -jar apktool.jar d t.apk -o out # 解包
java -jar apktool.jar b out -o t.apk # 打包
sign t.apk  # 每个apk都需要签名，可以想想自签名也行的那签名的意义
shell adb shell am start -D -n <packagename>/<MainActivity> # 以调试的方式启动，此时会挂起进程
adb forward tcp:8700 jdwp:<pid> # 创建端口转发


```

后者一般用 ida 的 dbg_server 调，这和普通的 Linux 程序调试没太大区别，只是在用 USB 连接时，需要用 adb 进行端口转发

```
adb push android_server /data # 这个目录好呀，可写可执行
# 也可以放根目录，需要改权限 mount -o rw,remount /
chmod +x android_server 
adb forward tcp:23946 tcp:23946  # 通过adb端口转发


```

### 反调试

其实反调就那两种套路：检测调试与对抗调试，检测就是若有调试器就进入非正常逻辑，包括检测自身状态 (如自身 status 文件，使用 ptrace) 与检测外部环境(如监听了 23946 等端口，运行着名为 gdb 的进程等)；而对抗就是使用与调试器有冲突的功能实现正常逻辑（如隐藏自身进程，双子运行，异常运行），这样有调试器时程序就无法正常运行。

反调试是一种初级的对抗手段，因为它是可逆的，所以找到关键点 pass 掉就好了，这里记录一些点：

1.  查看 / proc/\<pid>/ 下的文件，如 status 的 TracePid，stat 的标志，wchan 等
2.  使用 ptrace 进行双子执行或者使用 PT_DENY_ATTACH 等
3.  检查进程名，监听的端口
4.  通过一些系统提供的接口，如 isDebuggerConnected

阅读 APP 分析
---------

### 分析要解密的文件

首先，找个远古时代的版本，因为越新的保护可能做的越好，我们的目的不是看它怎么保护的，而是弄出算法就行。可以去[豌豆荚](https://www.wandoujia.com/)找找，一般都有，比如秋秋：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632566080580-31626003-cc8d-42c6-9c7e-f0febf516ece.png)  
一般升级都会兼容旧版的，所以算法不会变，即使变也可以了解下初代的样子，下载安装好之后就可以运行它，登录账号下几本书后退出，先看看书本身的样子，像书籍这种比较大的数据它们都喜欢放 sdcard 里，最终在`/storage/emulated/0/Android/data/com.jingdong.app.reader.campus/files/`里找到了下载的书，它们以 JEB 后缀命名，但是都无法打开，先使用 binwalk 对文件进行分析，epub 如下：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632208064514-a865a6de-1bb4-4e14-b298-1268c8feb194.png)  
解压后发现图片正常，而文字被加密无法正常显示：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632208212398-3ea90174-2d5f-4aa3-8dff-581fc3d950f0.png)  
而对于 pdf 文件，可见它也使用自定义的过滤器进行了加密：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632304287169-6da7b2de-b055-432a-88a9-fbe19f494759.png)  
​  

### 分析数据库

在移动破解中，数据库里面会有很多有用的线索，所以可以先看看，数据库一般会保存在`/data/data/com.jingdong.app.reader.campus/databases`，取出数据库，直接打开被告知文件非 SQLite 文件或已被加密，没办法只能先看 apk 文件了，用 GDA 打开，搜索 SQL/SELECT 等关键字，会发现数据库使用了 [greendao](http://greenrobot.org/)，打开官网发现它支持使用 [sqlcipher](https://www.zetetic.net/sqlcipher/) 加密数据库，从 so 文件可以看出是版本 3：

```
.rodata:00119EFB aCipherVersion  DCB "cipher_version",0  ; DATA XREF: sqlcipher_codec_pragma_0+178↑o
.rodata:00119EFB                                         ; sqlcipher_codec_pragma_0:off_6F6F8↑o
.rodata:00119F0A a340            DCB "3.4.0",0           ; DATA XREF: sqlcipher_codec_pragma_0+18E↑o


```

  
sqlcipher v3 使用 AES256 CBC 模式加密：  

1.  把一个数据库分成多个 chunk(也可以把它叫做 page, 默认大小为 4kB)，每个 chunk 单独加密，显然 password 是相同的。
2.  不同的是，每个 chunk 有自己的 iv，iv 被存在每个 chunk 的末尾，另外每次写数据都会生成新的 iv。
3.  对每个 chunk，对 iv 与密文生成 MAC(默认 HMAC-SHA512)，读时做校验
4.  对每个数据库，会生成一个 16B 的盐，存储在文件开始处。加密密钥使用 PBKDF2-HMAC-SHA512 方式，默认迭代 256000 次。
5.  HMAC 的 key 与加密的 key 不同，前者由后者使用 PBKDF2 进行二次迭代及一次交换生成。
6.  若为了显示魔数等信息而放弃对数据库头部加密，那么盐只能被存在外部，每次打开数据库时显式指出。
7.  其他。。。

不过不重要，网上一通搜索知道了怎么用工具解密：

```
sqlcipher-shell32.exe sk.db
sqlite> PRAGMA KEY = 'exm';
sqlite> ATTACH DATABASE 'sk_plaintext.db' AS plaintext KEY '';
sqlite> SELECT sqlcipher_export('plaintext');


```

通过全文搜索`getEncryptedWritableDb`或者`getEncryptedreadableDb`可以定位到获得加密数据库实力的代码处，回溯可以分析其密钥生成算法，然而。。。。它的密钥是硬编码的：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632567764223-f3519cb1-8829-45ff-9480-96df1b9225ea.png)

<table><thead><tr><th>库名</th><th>密码</th></tr></thead><tbody><tr><td>books.db</td><td>SessionBookDataUtil</td></tr><tr><td>plugin.db</td><td>SessionPluginDataUtil</td></tr><tr><td>sync.db</td><td>SessionSyncDataUtil</td></tr><tr><td>team.db</td><td>SessionTeamDataUtil</td></tr><tr><td>chapter_divisions_book_store.db</td><td>password</td></tr><tr><td>download_failed_record.db</td><td>password</td></tr><tr><td>file_store.db</td><td>password</td></tr><tr><td>the_whole_book_store.db</td><td>password</td></tr></tbody></table>

把数据库解密，可以看到文件的解密 key 来自 books.db 的 key 列：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632568105067-009347cf-143b-41d7-81d8-171c725554a6.png)  

### 分析 DEX

看到这里我下载的版本是没有做其他加固的，只有部分地方有混淆，通过搜索 openbook，encrypt，decrypt 等可以发现 com.jingdong.app.reader.main.action.OpenBookAction 里有如下内容：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632568439903-cb7711e9-082b-4390-92e1-7643b1fffc07.png)  
经分析 arg11 是 com.jingdong.app.reader.data.database.dao.books.JDBookDao 表的映射，所以通过数据库操作与关键词可定位到 com.jd.read.engine.jni.DocView：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632568781640-3d095023-bda7-4832-a8bd-d6bda67db26a.png)  
可见把这些数据传入到 so 里了，另外它的另一个参数表示书籍是否完整还是分章：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632207679223-18160771-e9e9-4ca0-b7f0-9ff04ad767b3.png)

### 分析 SO

上面已经知道怎么调用 so 了，现在就直接分析它，它是 libDecryptorJni.so 文件，定位函数时，如果时标准的函数名就是一一对应的，否则从 JNI_OnLoad(registerNativeMethods) 里面找，不过这里它用的标准名称！  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632569289125-c990c231-ebf3-4441-8c06-39c2a979ca68.png)  
这里用 jeb 能很好识别 ENV 与 [arm 的特定指令](https://www.kernel.org/doc/Documentation/arm/kernel_user_helpers.txt)，如下在 IDA 中无法识别，遇到这种直接跳过即可：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632569468475-b1d2d6a1-eaf5-4af0-b1e6-5dec34e2acf1.png)  
因为习惯用 IDA，所以还是用它接着搞，继续分析可以看到它根据类型使用三种不同的方式解密数据：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632569725923-4c7e2541-531d-40c6-a864-9a890b0f4479.png)  
网络类型的会生成用现有的 key 生成一个新的 key，分章节的只是使用了传入的 KEY，而完本的把传入的 Key，DeviceID 和 Random 全部传入了，所以先看网络类型生成新 Key 的算法吧：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632569975667-8a326f57-37eb-4fae-9139-e4fac9b83be2.png)  
没有去符号，所以一眼就能看出是个双重 MD5：

```
def generate_net_book_key(data: bytes):
    data = md5(data).digest()
    data = md5(data).hexdigest().upper()
    return data.encode()


```

然后是完本的，它会调用 GetContentKeyBuf，而 GetContentKeyBuf 又会调用 j_j_AnalyticRightFileBuf，后者的结果中将会包含 ID 和 CK，返回的是 CK：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632570204336-08e0c641-a400-4a9f-912a-02d2ca0600ac.png)  
继续跟入 j_j_AnalyticRightFileBuf，后续代码太多就不截图了，直接贴代码：

```
// key = 0001xxxxxxx<HS>0044yyyyyyy
v8 = (unsigned __int8 *)j_memstr((char *)a1, a2, "<HS>", 4);   // 截取<HS>之后的部分，前四字节是长度，从而获取后面的数据
j_base64Decode(v12, v31, (char *)v11)       // 解码数据，看后面可知它是32字节的hash值
    v28 = (char *)v11;
*(_DWORD *)v45 = (((a1[3] << 8) | a1[2]) << 16) | (a1[1] << 8) | *a1;   // 获取key前四字节，做版本号
v13 = a2 - 12 - v31;
v14 = j_DecryptByVersion((char *)a1 + 4, v13, &v42, &v41, v45);  // 继续解密
v40 = 0;
v39 = 0;
j_j_Hash_256((int)v42, v41, (int)&v40, (int)&v39);  
j_memcmp(v40, v28, 32)  // 判断解密后的数据的hash是否与hash一致
 ... 
j_DecryptCK(v42, v41, v38, v37, v36, v35, a5, a6); // 所有处理完的数据一起再进行解密


```

#### DecryptByVersion

继续入 j_DecryptByVersion，需要版本号为 0001，之后会调用 StringDecryptQomolangma:

```
v9 = j_strcmp(a5, "0001");
v10 = j_j_StringDecryptQomolangma(a1, a2, a3, a4);


```

接着跟入 StringDecryptQomolangma：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632570970077-593a371d-3e4e-489e-90d7-a521a8e07d9f.png)  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632571050342-b2cc8823-0b1e-43c3-b3f6-88d1a602df50.png)  
先说最后一种变换吧，就很简单的:

```
    res = []
    a_ascii, z_ascii, A_ascii, Z_ascii, zero_ascii, nine_ascii = map(ord, ('a', 'z', 'A', 'Z', '0', '9'))
    for c in data:
        if a_ascii <= c <= z_ascii:
            c -= 3
            c = c if c >= a_ascii else c + 26
        elif A_ascii <= c <= Z_ascii:
            c -= 3
            c = c if c >= A_ascii else c + 26
        elif zero_ascii <= c <= nine_ascii:
            c -= 1
            c = c if c >= zero_ascii else nine_ascii
        res.append(c)
    data = bytes(res)


```

现在分析那三个函数：

##### j_JY_Crypt

![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632571271045-3f5efc5d-92cc-4e81-a3bb-8908ff4e178f.png)  
很明显是 RC4 算法，不多说。

##### BillDecode

经查询，它和网上的某种 DRM 解码很像：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632571463682-74b43891-3271-4bc5-a0ae-5ae1244e46d7.png)  
经分析它用了自适应的解码算法，如下：

```
BILL_TABLE0 = b"n5Pr6St7Uv8Wx9YzAb0Cd1Ef2Gh3Jk4M"
BILL_TABLE1 = b"AaZzB0bYyCc1XxDdW2wEeVv3FfUuG4g-TtHh5SsIiR6rJjQq7KkPpL8lOoMm9Nn_"


def bill_decode(data: bytes):
    result = b''
    for c in BILL_TABLE0[:8]:
        if data[0] == c:
            table = BILL_TABLE0
            break
    else:
        for c in BILL_TABLE1[:4]:
            if data[0] == c:
                table = BILL_TABLE1
                break
        else:
            raise Exception('????')

    for i in range(0, len(data) - 1, 2):
        high = table.find(data[i])
        low = table.find(data[i + 1])
        if (high == -1) or (low == -1):
            break
        value = (((high * len(table)) ^ 0x80) & 0xFF) + low
        result += byte(value)
    return result


```

##### ExchangeChar

![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632571572113-0aeb1499-6ea5-4968-859c-a9914aac6ab8.png)  
这就是一个简单的交换，以 8 字节为单位：

```
def exchange_char(data: bytes) -> bytes:
    res = bytearray(data)
    for i in range(0, len(data), 8):
        res[i:i + 8] = data[i:i + 8][::-1]
    return bytes(res)


```

#### DecryptCK

该函数大致如下：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632571972479-d0a6dcf9-506d-4894-8edb-ec6d7dc4ead4.png)  
它解密了起始的 32 字节，并连接剩余数据再用之前分析的算法解码，这里还需要注意 AES 解密的工作模式，它其实用的是 CBC 模式，IV 被硬编码进去了，全是 0:

```
IV = b'0000000000000000'

def aes_pkcs7pad_decrypt(key: bytes, data: bytes):
    key = sha256(key).digest()
    cipher = AES.new(key, IV=IV, mode=AES.MODE_CBC)
    plain = cipher.decrypt(data)
    return pkcs7_unpad(plain)


```

#### jddecompress

综上，完本的也解密完了，终于轮到解密书籍的部分了，它们使用了同样的逻辑，就是 AES 解密之后再解压：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632572397344-c43b8ffc-febd-4270-b460-c2f083d3ef5d.png)  
最后就是解密了，主要我想说 PDF 解密，现在已知 PDF 使用了 filter 去加解密数据，需要去写一个解密工具，本来我在网上搜了个[库](https://github.com/jsvine/pdfplumber)，看了下它不支持自定义过滤器，又打了个补丁让它去解密：

```
    old_get_data = PDFStream.get_data
    old_get_filters = PDFStream.get_filters

    def get_data(self):
        old_filters = old_get_filters(self)
        for (f, params) in old_filters:
            if f.name == 'JDPDFENCRYPTBY360BUY':
                try:
                    self.rawdata = aes_pkcs7pad_decrypt(key, self.rawdata[:-1])
                except Exception as e:
                    print(e)
        return old_get_data(self)

    def get_filters(self):
        old_filters = old_get_filters(self)
        for (f, params) in old_filters:
            if f.name == 'JDPDFENCRYPTBY360BUY':
                old_filters.remove((f, params))
        return old_filters

    PDFStream.get_data = get_data
    PDFStream.get_filters = get_filters


```

然鹅，解完密翻代码没找到 save 方法，翻文档才发现它不支持修改文件...  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632314409430-59c8589d-3a54-4e11-8ab6-76276e1c2d85.png)  
！！！！最后，根据 [pdf 文件格式](https://resources.infosecinstitute.com/topic/pdf-file-format-basic-structure/)，花了好几个晚上写了个简易的解析类才肝出来，简单说下，PDF 从尾部向前解析，先读 trailer 获取 xref，再根据它解析对象，对象只需要关注 stream 对象，把它解密并去掉字典里的过滤器，再调整几个表的偏移即可：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632572972232-f6ab2c51-2288-4466-8384-f8c12bd7616b.png)  
最后的结果如下：  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632486531303-78fb3d64-7aa3-414b-a061-ce9f8c137af0.png)  

最后最后最后
------

欢迎孟晚舟回家！！！！！  
![](https://blog.betamao.me/posts/2021/some-ebook-decrypt/pic/1632579205049-f611a98a-818b-44ba-921f-bfe1026536eb.png)

Powered by [Pelican](https://getpelican.com/), Theme by [B3taMa0](#), Based on [Smashing Magazine](https://www.smashingmagazine.com/2009/08/designing-a-html-5-layout-from-scratch/)