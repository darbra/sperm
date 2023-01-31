> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.betamao.me](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/)

> 之前想看一本书只有 XXX 上有，于是开了个会员准备把它解出来，但是后面沉迷其他事耽误了，今天突然想起一看会员要白开了赶紧分析，最后在调试这里被坑了很久最终没弄完，先记下来以后有时间再看看....

之前想看一本书只有 XXX 上有，于是开了个会员准备把它解出来，但是后面沉迷其他事耽误了，今天突然想起一看会员要白开了赶紧分析，最后在调试这里被坑了很久最终没弄完，先记下来以后有时间再看看....

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636252353558-dc805cbf-4546-469f-8cd5-52cf626fe0e5.png)

> 注：分析只是为了方便把书籍放 Pad 上阅读，为了保护知识产权本文只分享思路，并且会使用 XXX 表示 APP 名称，不会提供任何代码或解密后书籍，请勿联系索取...

分析
--

### 分析数据库和被保护数据

当然还是从数据库和文件入手，经分析它的数据库和缓存文件如下：

```
dream2lte:/data/data/com.XXX.player # ls
app_bugly        app_ebook_base_font  app_packages  app_process_lock  app_tbs       app_webview  code_cache  files  lib-main
app_crashrecord  app_flutter          app_patrons   app_sslcache      app_textures  cache        databases   lib    shared_prefs

dream2lte:/sdcard/.XXX # ls
__MACOSX  apk  audio  cache  chatVoice  ddjk.ttf  ebook  frame.html  igetgetBook  pic


```

从数据库看它的信息比较简单，token 时 jwt 应该有点东西，chapters 是 json 格式，从里边看它是分章节存储的 epub 文件： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636368736100-eb6b8924-c86a-40a4-b477-e8a552c7002c.png) 打开 epub 文件，看到它是被加密了： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636368955946-7b6de0e3-23d9-478d-9117-71a988d10474.png) 而且看着像是 epub 规范规定的加密，[搜索可知](https://www.w3.org/TR/2002/REC-xmlenc-core-20021210/Overview.html#kw-aes128)确实像： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636369199887-c4ec9463-0dee-482b-b9dc-c42d09097edf.png) 但是如果真像这样就好咯，可以直接解密，然鹅这里的 encryption.xml 格式不对劲：

```
  <enc:EncryptionKey>
    <enc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#rsa-1_5" Version="1.0"/>
    <ds:KeyInfo>
      <ds:KeyName>XXX.Inc</ds:KeyName>
    </ds:KeyInfo>
    <enc:CipherData>
      <enc:CipherValue>DRM</enc:CipherValue>
    </enc:CipherData>
  </enc:EncryptionKey>

  <enc:EncryptedData>
    <enc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#kw-aes128"/>
    <ds:KeyInfo>
      <ds:RetrievalMethod Type="http://www.w3.org/2001/04/xmlenc#EncryptionKey" URI="#EK"/>
    </ds:KeyInfo>
    <enc:CipherData>
      <enc:CipherReference URI="OEBPS/Images/image-150.jpg"/>
    </enc:CipherData>
    <enc:EncryptionProperties>
      <enc:EncryptionProperty xmlns:ns="http://www.idpf.org/2016/encryption#compression">
        <ns:Compression Method="8" OriginalLength="21075"/>
      </enc:EncryptionProperty>
    </enc:EncryptionProperties>
  </enc:EncryptedData>


```

可以看到它的 Key 格式根本不对，应该是经过修改的，于是只能先看看 apk 了，通过搜索 openbook，token 之类的关键词找到这个点很可疑： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636538261746-588a2d07-61a8-4519-80f7-6d6597f8c1b3.png)

### 定位关键点

通过搜索上面的加密关键字也可以定位到`libebk-engine.so`库，先看看它注册的 Native 函数，搜索发现它使用的`RegsiterNative`方式注册的，指定类型后如下： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636460777671-aa5d2645-3ca8-4bbc-b10f-95950cf9ad01.png) 这里直接跟着 getToken.../openBook... 等也能分析出解密过程，但是想找捷径，先直接用 [findcrypt](https://github.com/polymorf/findcrypt-yara) 去找找 AES 表，这里只有个逆盒还没交叉引用： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636373017560-9314686d-7625-462d-8341-c1116db95ab7.png) 那就先上土办法直接搜 0xA56363C6: ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636373414236-efa30276-2bd1-4882-ad99-d8488cdee498.png) 定位到 AES 操作的位置：

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636548078757-45c5d6c0-7338-410d-9c12-75ded4bf1881.png)

这里是静态编译进去的，不像上一个应用那么明显，一般也不用分析算法，再向上翻，通过交叉引用到如下函数，图里我分析时已经加了符号，本来是无符号的： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636548555120-d09c0139-9134-4cd5-a2a3-92b3f34b003a.png) 可以看到这里面已经有很多运算了，而且一些点基本能猜出来，继续分析其他参数的作用，如通过 memcpy 或赋值，通过类型等可猜测如下：

```
int __fastcall dec_70A9384(char *ibuf, _DWORD *obuf, char *key, int key_size, int total_size, int *out_size, int curpos, int typ)


```

现在可以开始验证猜测，以 MD5 为例，此时可用 Frida Dump 出数据，启动方式见下一节，此处直接上代码：

```
let ebookModName = 'libebk-engine.so';
let ebookkBase = Module.getBaseAddress(ebookModName);

function decInerTrace() {
    let dec_70A9384 = ebookkBase.add(0x10A9384);
    Interceptor.attach(dec_70A9384, {
        onEnter: function (args) {
            this.ibuf = dumpmem(args[0], min(64, args[4].toUInt32()));  // 函数进入时，备份密文，否则在解密时可能会被破坏
            this.l = min(64, args[4].toUInt32());  // 密文长度
            this.obufAddr = args[1];  // 备份明文的地址，在输出时才能读到明文
            this.key = dumpmem(args[2], args[3].toUInt32());  // key的长度
            this.curpos = args[6].toUInt32();  // 这个是后来分析的，先忽略
        },
        onLeave: function (retval) {
          // 如下，在函数结束时，输出整个函数调用的结果
            console.log(`dec(
 ibuf: ${this.ibuf}
 key: ${this.key}
 curpos: ${this.curpos}
 obuf:${dumpmem(this.obufAddr, this.l)}
)`)
        }
    });
}


```

通过这种方式能获取到此函数的输入与输出：

```
dec(
 ibuf: 9710de00  f6 01 7f fa 6d 3f fe 7b b7 30 fb fc 11 b8 76 19  ....m?.{.0....v.
             9710de10  d9 9d 9d 69 af 61 e3 42 be f4 fb 26 96 c2 60 36  ...i.a.B...&..`6
             9710de20  ea c7 3e d5 86 6b 35 f0 60 e4 83 f7 e4 80 76 ea  ..>..k5.`.....v.
             9710de30  82 84 a6 e9 5d 0c a4 a9 1a 52 8b 1a 0f 72 c7 6a  ....]....R...r.j
 key:  7cfac0a0  5a 80 25 78 b3 68 88 98 3f 0b 2d 60 2f ea 8c 32  Z.%x.h..?.-`/..2
 curpos: 278528
 obuf: 9710de00  8d 91 49 5f fe 39 a4 47 eb 93 b6 dd 1d 2c 0e 10  ..I_.9.G.....,..
           9710de10  d0 88 9a a3 1e 35 89 8b 18 ff 00 49 93 1f 66 8c  .....5.....I..f.
             9710de20  b9 ef ab 4f 45 8d 17 70 aa 86 2b 02 87 2c 24 80  ...OE..p..+..,$.
             9710de30  18 00 16 96 1e 6f 8b ab c4 4b ef a6 47 2d df 2e  .....o...K..G-..
)


```

首先验证下 MD5 的猜测是否正确，此时再对 md5final 做插桩：

```
function md5Trace() {
    let md5Final = ebookkBase.add(0x10AD3F8);
    Interceptor.attach(md5Final, {
        onEnter: function (args) {
            this.outBuf = args[0];
        },
        onLeave: function (retval) {
            console.log(`md5Final(${dumpmem(this.outBuf, 16)})`)
        }
    })
}


```

输出如下：

```
md5Final(a8921518  0f b4 08 49 4d af fe e5 b8 72 04 db ad 26 69 d1  ...IM....r...&i.)


```

于是写 Python 代码验证下：

```
In [1]: import hashlib

In [2]: hashlib.md5(bytes.fromhex('5a 80 25 78 b3 68 88 98 3f 0b 2d 60 2f ea 8c 32')).hexdigest()
Out[2]: '0fb408494daffee5b87204dbad2669d1'


```

可见猜测一致，继续用这种方法 trace aes 解密函数：

```
function aesDecTrace() {
    let aesDec = ebookkBase.add(0x10A97D8);
    Interceptor.attach(aesDec, {
        onEnter: function (args) {
            this.arg1 = dumpmem(args[0], args[2].toUInt32());
            this.arg2 = args[1];
            this.arg3 = args[2].toUInt32();
            this.iv = dumpmem(args[3], 16);
            this.key = dumpmem(args[5], 16);
        },
        onLeave: function (retval) {
            console.log(`aes dec (ret=${retval})=> (
        key:${this.key}
        iv:${this.iv}
        in:${this.arg1}
        in_size:${this.arg3}
        out:${dumpmem(this.arg2, this.arg3)})`);
        }
    });
}


```

由于它每次解密 16 字节，因此会有大量的输出，但是这里面有些并不是 16 字节，如：

```
aes dec (ret=0x0)=> (
        key:a892144c  5a 80 25 78 b3 68 88 98 3f 0b 2d 60 2f ea 8c 32  Z.%x.h..?.-`/..2
        iv:a8921500  aa 11 ad ec da 00 00 00 e8 0a 5b 40 1d d7 a1 7e  ..........[@...~
        in:9af4d3a0  3c b1 ad 77 10 3a 5f 1f f1 8a 61 0e d9 89        <..w.:_...a...
        in_size: 14
        out:a8921520  c9 69 b7 fa 3f a0 69 37 fd 7f d5 bd ff 00        .i..?.i7......)
aes dec (ret=0x0)=> (
        key:a8920a8c  5a 80 25 78 b3 68 88 98 3f 0b 2d 60 2f ea 8c 32  Z.%x.h..?.-`/..2
        iv:a8920b40  aa 11 ad ec 80 5d 00 00 e8 0a 5b 40 1d d7 a1 7e  .....]....[@...~
        in:a5b97200  d2 2a a5                                         .*.
        in_size: 3
        out:a8920b60  7f ff d9                                         ...)


```

可见它的输入并不一定是 AES 加密的填充长度，此时要么用 XTS，要么自己实现，要么就是用 CFB 或 OFB 模式： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636551070635-733d02bb-5217-4bba-8db8-6b007b3e7889.png) 这两种模式区别还是很明显，看看输入的 IV 和 Key 是怎么和密文交互就知道了。不过这里还是直接猜测，由于密钥和 IV 是正常长度前者可以排除，因此尝试 CFB 和 OFB 发现 OFB 模式满足要求：

```
In [3]: from Crypto.Cipher import AES

In [4]: iv = bytes.fromhex('aa 11 ad ec da 00 00 00 e8 0a 5b 40 1d d7 a1 7e')

In [5]: key = bytes.fromhex('5a 80 25 78 b3 68 88 98 3f 0b 2d 60 2f ea 8c 32')

In [6]: cipher_text = bytes.fromhex('3c b1 ad 77 10 3a 5f 1f f1 8a 61 0e d9 89')

In [7]: cipher = AES.new(key, mode=AES.MODE_OFB, IV=iv)

In [8]: cipher.decrypt(cipher_text).hex()
Out[8]: 'c969b7fa3fa06937fd7fd5bdff00'


```

其他猜测全可使用这种方式验证，之后不再演示。仔细观察上文的输出会发现 IV 一直在变化：

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636550582729-c828a2fa-0256-479b-9126-3cc23b95e9b3.png)

于是可写出如下代码：

```
import struct
from hashlib import md5

from Crypto.Cipher import AES


pack32 = lambda num: struct.pack('<I', num)
unpack32 = lambda data_bytes: struct.unpack('<I', data_bytes)[0]

AES_BLOCK_SIZE = AES.block_size


def aes_dec(aes_key: bytes, cur_pos: int, cipher_text: bytes):
    buf_arr = []
    tmp_iv = md5(aes_key).digest()
    tmp_iv = bytes(map(lambda ch: ch ^ 0xa5, tmp_iv))
    iv = tmp_iv[0:4] + pack32(cur_pos // AES_BLOCK_SIZE) + tmp_iv[4:0xc]
    for i in range(0, len(cipher_text), AES_BLOCK_SIZE):
        aes = AES.new(aes_key, mode=AES.MODE_OFB, IV=iv)
        buf_arr.append(aes.decrypt(cipher_text[i:i + AES_BLOCK_SIZE]))
        iv = iv[0:4] + pack32(unpack32(iv[4:8]) + 1) + iv[8:]
    return b''.join(buf_arr)


```

继续向上分析，发现它被如下函数调用： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636551840345-8f0be2eb-57e0-4b23-8d13-9b20f9d84c8c.png) 通过交叉引用可知是类`future_core::EncryptedInputStream`虚函数，它的父类是`future_core::InputStream`，看名字一般都是抽象类，我们需要找到一个简易的实现类去分析它的虚表，经查找有 future_core::FileInputStream 类，它是对 libc 调用的简单封装，因此很容易通过它分析出虚函数的作用： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636552131931-aa564e84-bb7e-42a2-8a40-8cd8b68de41b.png) 因为虚函数的特性，对照过来就是到了`EncryptedInputStream`的虚函数作用了，接下来就是继续分析 key 的来源。 由于是 CPP 的虚函数调用没有交叉引用了，因此最好使用动态分析，本来想通过 Stalker 绘制个调用图，然鹅目前 Frida 对 arm32 的 Stalker 支持还有 BUG，[QDBI 也不支持 ARM](https://github.com/QBDI/QBDI/releases/tag/v0.8.0) 了：

```
function traceCall() {
    // 只关心ebook，先排除掉其他范围
    let ebookModuleMap = new ModuleMap(function (mod) {
        return mod.path.indexOf(ebookModName) != -1;
    });
    Process.enumerateRanges('--x').forEach(function (range) {
        if (!ebookModuleMap.has(range.base)) {
            Stalker.exclude(range);
        }
    });
    // 追踪所有线程的调用
    Process.enumerateThreads().forEach(function (thread) {
            Stalker.follow(thread.id, {
                events: {
                    call: true, // CALL instructions: yes please
                },
                onReceive(events) {
                    console.log(Stalker.parse(events, {
                        annotate: true,
                        stringify: true
                    }));
                },
            })
        }
    )
}


```

因此就打算用调试，结果遇到了很多问题，最终没弄完，现在有两条路，硬着分析或者看看为什么调试会失败，是不是有反调手段，简单跟了几个点没发现问题：

```
function fopenTrace() {
    Interceptor.attach(Module.getExportByName(null, 'fopen'), {
        onEnter: function (args) {
            this.path = args[0].readCString();
        },
        onLeave: function (retval) {
            console.log(`fopen(${this.path})=>fd=${retval}`)
        }
    });
    Interceptor.attach(Module.getExportByName(null, 'open'), {
        onEnter: function (args) {
            this.path = args[0].readCString();
        },
        onLeave: function (retval) {
            console.log(`open(${this.path})=>fd=${retval}`)
        }
    });
    Interceptor.attach(Module.getExportByName(null, 'openat'), {
        onEnter: function (args) {
            this.dirFd = args[0].toUInt32();
            this.path = args[1].readCString();
        },
        onLeave: function (retval) {
            console.log(`openat(${this.dirFd}, ${this.path})=>fd=${retval}`)
        }
    })
}

function signalTrace() {
    Interceptor.attach(Module.getExportByName(null, 'gsignal'), {
        onEnter: function (args) {
            console.log(`gsignal(${args[0].toUInt32()})`);
        }
    });
    Interceptor.attach(Module.getExportByName(null, 'raise'), {
        onEnter: function (args) {
            console.log(`raise(${args[0].toUInt32()})`);
        }
    });
  //....
}


```

还能用`console.log('Return : ' + this.returnAddress);`去获取函数的返回地址，通过笨办法也能绘制出调用图，硬着分析就是正着来，工作量也不大，暂时不想搞了。。。待续

### 续

又有需求想看本书发现就 DD 上有，重新拿起分析，由于期间也没有再看 frida，这次直接静态分析，根据继承情况可对类进行定义，首先看`EncryptedInputStream`的 open，分析如下：

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/image-20220817200512504.png)

再看其他类，可知加密输入流对象是被嵌入压缩流对象的：

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/image-20220817201008002.png)

所以其实就是通常的 DRM 保护，用 frida 验证一下确实能得到明文，回想之前之所以失败，猜测可能没有设对 zlib 的解压参数，最终通过这种方法可正确解密数据：

![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/image-20220817203622952.png)

调试
--

### Frida 插桩

使用 adb 启动 frida-server，我使用的是 BlueStack 的混合模式，因此 server 需要为 x86 架构的，否则它会显示找不到 libc(其实也可以解决但是没必要)：

```
betamao@DESKTOP # adb shell
dream2lte:/ $ su
dream2lte:/ # /data/frida-server-15.1.2-android-x86 -l 0.0.0.0:55555
/data/frida-server-15.1.2-android-x86 -l 0.0.0.0:55555 ...


```

此处可以直接使用 USB 连接，不过我喜欢端口转发，这里加一步：

```
betamao@DESKTOP # adb forward tcp:55555 tcp:55555


```

这样就可以插桩了，由于要插桩的应用是 arm 架构的，因此需要加上`--realm=emulated`参数，如下：

```
betamao@DESKTOP # frida -l .\xxx.js -n XXX -H 127.0.0.1:55555 --realm=emulated


```

### 使用 ida

frida-gadget 注入成功后不再占用调试接口，因此可使用 ida 进行调试：

```
betamao@DESKTOP # adb shell
dream2lte:/ $ su
dream2lte:/ # cd /data
dream2lte:/data # ./ads -p55556 -v
IDA Android 32-bit remote debug server(ST) v7.5.26. Hex-Rays (c) 2004-2020
Listening on 0.0.0.0:55556..

betamao@DESKTOP # adb forward tcp:55556 tcp:55556


```

但是即使未打任何断点，只要 attach 就会崩溃： ![](https://blog.betamao.me/posts/2021/frida-example-some-app-analyze/pic/1636196856925-10909f4f-6605-4865-bc4f-d28b21f92ae8.png) 看崩溃位置，查看文件类型发现是 x86 架构的：

```
dream2lte:/ # file /system/bin/app_process32_original
/system/bin/app_process32_original: ELF shared object, 32-bit LSB 386, dynamic (/system/bin/linker), for Android 25, BuildID=f79ee9d7df3ceb7c81692f303514779a, stripped


```

### 使用 gdb

怀疑是模拟器原因，因此直接上真机，由于真机 USB+IDA 速度异常的慢，此处使用 gdb 调试，在安装 ndk 时会自动下载 gdbserver，也可以直接去网上下，运行服务：

```
dream2lte:/ # ./gdbserver --attach :55557 ${ps -ef | grep luoji | grep -v 'grep'|cut -d' ' -f7}


```

之后使用 gdb 远程调试，注意需要使用 arm 版或 multiarch 版：

```
betamao@DESKTOP:~/$ apt install gdb-multiarch
betamao@DESKTOP:~/$ gdb-multiarch
pwndbg> set arch arm
pwndbg> target remote 127.0.0.1:55557


```

目标存在大量共享库，为了提高加载速度可先将这些库下载到本地，再使用`set sysroot /some/sysroot`命令指定为本地路径。

Powered by [Pelican](https://getpelican.com/), Theme by [B3taMa0](#), Based on [Smashing Magazine](https://www.smashingmagazine.com/2009/08/designing-a-html-5-layout-from-scratch/)