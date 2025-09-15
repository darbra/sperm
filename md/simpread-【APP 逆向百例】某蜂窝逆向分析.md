> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dv65d7DXZ717NtBehJeKFw)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImwmmViad8KqQbMXCypIQodgBMolMFSlWZrSGf1SYFXpL5lTTNTSZcb0A/640?wx_fmt=png&from=appmsg#imgIndex=0)

声明
--

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！**

**本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号【K 哥爬虫】联系作者立即删除！**

逆向目标
----

*   目标：某蜂窝 APP
    
*   apk 版本：11.2.0
    
*   逆向参数：zzzghostsign
    

抓包分析
----

打开 app，在首页进行刷新，charles 配合 SocksDroid 进行抓包，结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImsNy4LVSp9U9cD6LaztJExmwYZlLtUf2q8yotoBtAmMeO1Q1RiaBF1wg/640?wx_fmt=png&from=appmsg#imgIndex=1)

其中要逆向的参数有很多，这里只对 zzzghostsign 参数展开分析。

逆向分析
----

对 Java 常见的 api 进行 hook 操作：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImolD3LeibR3tG62xW6JBgRsou2vxIeCsxI2yzRVkqOJiac6Ficdg16Nn8w/640?wx_fmt=png&from=appmsg#imgIndex=2)

定位这个参数的方法有很多，这里我们使用 frida 对 java 的 StringBuilder 类进行 hook，`StringBuilder` 是 Java 中用于创建和操作可变字符串的类，一般操作字符串都会使用这个类，hook 代码如下：

```
function showStacks() {
    Java.perform(function () {
      console.log(Java.use("android.util.Log").getStackTraceString(
        Java.use("java.lang.Throwable").$new()
    ));
 })
}

function hook_StringBuilder(){
    var stringBuilderClass = Java.use("java.lang.StringBuilder");
    stringBuilderClass.toString.implementation = function (){
    var res = this.toString.apply(this,arguments);
    showStacks();
    console.log("StringBuilder-->" + res.toString())
    return res
    }
}
Java.perform(function() {
    hook_StringBuilder()
});


```

通过使用 `frida -UF -l demo.js -o 1.txt` 进行 frida 注入，命令解释如下：

*   -U 代表 **USB**，表示 Frida 会与连接的 USB 设备（比如 Android 手机）通信；
    
*   -F 启动并附加到目标应用的进程；
    
*   -l demo.js 要加载并执行的 JavaScript 文件；
    
*   -o 1.txt 将 Frida 执行过程中的输出保存到一个文件中。
    

注意，捕获的东西有点多，可能导致卡死、应用闪退，可以在输出一段时间后手动关掉，或者根据相关字段进行过滤。

输出结果如下，直接输入 zzzghostsign 的值，会发现存在，并打印出了相关堆栈：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im2Irhn2lVzvn2QvFwJDBD2d44UoQy8HxHEJJwkEdqTODQEOpXNmV48Q/640?wx_fmt=png&from=appmsg#imgIndex=3)

java 层分析
--------

把 apk 文件拖到 jadx，根据堆栈信息进行搜索，最终定位到下图这个地方：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImmszlJjh6Uu8NJicMesViblgfHPZEBFTTzvOoXrZCfWGLa8yyJicicpicYBw/640?wx_fmt=png&from=appmsg#imgIndex=4)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im39dwD3Q8dySPtib089HB1gSah4YaSCshZEAZufuzKqguRIhpPm94BcA/640?wx_fmt=png&from=appmsg#imgIndex=5)

其中 ghostSign 函数就是参数生成处，可以使用 frida 代码验证位置是否正确，鼠标放到 ghostSign 右键复制 frida  代码：

```
function hook1(){
    let AuthorizeHelper = Java.use("com.mfw.tnative.AuthorizeHelper");
    AuthorizeHelper["xPreAuthencode"].implementation = function (context, str, str2) {
    console.log('xPreAuthencode is called' + ', ' + 'context: ' + context + ', ' + 'str: ' + str + ', ' + 'str2: ' + str2);
    let ret = this.xPreAuthencode(context, str, str2);
    console.log('xPreAuthencode ret value is ' + ret);
    return ret;
};


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im73NbxonAibQEFm1URiaeYBFf8w94VQpiaFJuIlvngiceDz2YQQpKCmTPOw/640?wx_fmt=png&from=appmsg#imgIndex=6)

发现结果一致，我们继续向下分析，点进去这个函数，发现跳到了 java iterface 接口处：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImFDtibFYt8yQ0udLtvvxianOHtxSpMibPuMxiajibqSRsd5K9IIKmtHEeiauw/640?wx_fmt=png&from=appmsg#imgIndex=7)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImxtSAAEshwesJGZYWzIrBVB9uV5NO6wNQFOLXzgqwJAaibibsf5oH6DXQ/640?wx_fmt=png&from=appmsg#imgIndex=8)

java 的接口本身不能实例化。因此我们需要找到他具体的实现方式，可以搜索这个 java 类 Authorizer：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImqJ4yPMz3tt9PL4OYLIGPibUqb75BNd5gmjTHO7f28c97P4TWeWBas6Q/640?wx_fmt=png&from=appmsg#imgIndex=9)

下面就是接口实现的具体方法，我们进入到 m22665c 方法中去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Imo21xvqQuF8Se8AK450phz2aGwjBSdCfZV2ZnJhEuNZdibVbM4NtRvTQ/640?wx_fmt=png&from=appmsg#imgIndex=10)

最后定位到 native 层加密，并加载 mfw.so 文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImpT15Mdk50Pmv02icv22fXqDAtel5zTFicv5Juz10sTwE92hfA9vuozXg/640?wx_fmt=png&from=appmsg#imgIndex=11)

把这个加密写成主动调用的方式，方便我们后续进行分析，代码如下：

```
// xPreAuthencode
function call_mfw(){
    let AuthorizeHelper = Java.use("com.mfw.tnative.AuthorizeHelper");
    var current_application = Java.use('android.app.ActivityThread').currentApplication();
    var context = current_application.getApplicationContext();
    let str = "GET&https%3A%2F%2Fmapi.mafengwo.cn%2Fdiscovery%2Fget_index%2Fv7&app_code%3Dcom.mfw.roadbook%26app_ver%3D11.0.2%26app_version_code%3D1052%26brand%3Dgoogle%26channel_id%3DMFW-WDJPPZS-1%26dev_ver%3DD2313.0%26device_id%3D82d917db80c8eae2%26device_mid%3D860000000000001%26device_type%3Dandroid%26hardware_model%3DPixel%25203%26has_notch%3D0%26jsondata%3D%257B%2522top_tab_id%2522%253A%252255%2522%252C%2522filter_id%2522%253A%2522all%2522%252C%2522top_refresh%2522%253A%25221%2522%252C%2522by_user%2522%253A%25221%2522%257D%26mfwsdk_ver%3D20140507%26o_coord%3Dwgs%26o_lat%3DMzAuNDg5NjIy%26o_lng%3DMTE0LjQyMDQ5MQ%253D%253D%26oauth_consumer_key%3D5%26oauth_nonce%3D536f6e42-bc4c-463f-acb5-ee145e63f10e%26oauth_signature_method%3DHMAC-SHA1%26oauth_timestamp%3D1732784973%26oauth_token%3D0_0969044fd4edf59957f4a39bce9200c6%26oauth_version%3D1.0%26open_udid%3D82d917db80c8eae2%26screen_height%3D2028%26screen_scale%3D2.88%26screen_width%3D1080%26shumeng_id%3DDUx0wb9S3BzdWvrdsQ8G8T6MnVVJL6kbZPb2RFV4MHdiOVMzQnpkV3ZyZHNROEc4VDZNblZWSkw2a2JaUGIyc2h1%26sys_ver%3D11%26time_offset%3D480%26x_auth_mode%3Dclient_auth"
    let str2 = "com.mfw.roadbook"
    let ret = AuthorizeHelper.$new("com.mfw.roadbook").xPreAuthencode(context, str, str2);
    console.log('主动调用 ret value is ' + ret);
}


```

so 层分析
------

我们从 apk 拿到 mfw.so 文件放到 ida 进行分析：

先在 Exports 表里搜索 java，看是否是静态注册的，发现没有任何内容，那么这个函数一般就是动态注册，那我们在搜索 `JNI_Onload`，点进去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im89FiaQdAqLIxsDqEkGV3sGE3X47jUjSK4eyjc3WnctjJBx4Mk3RpfnA/640?wx_fmt=png&from=appmsg#imgIndex=12)

动态函数一般是通过 RegisterNatives 函数注册，一般有四个参数：

*   `env`：JNI 环境指针，用于与 JVM 交互；
    
*   `clazz`：Java 类的引用，表示要将本地方法注册到哪个 Java 类；
    
*   `methods`：指向 `JNINativeMethod` 结构体数组的指针，每个结构体包含本地方法的名称、签名和实现函数；
    
*   `nMethods`：`methods` 数组的大小，表示注册方法的数量。
    

下图函数具体实现就是 `off_FF020`，注册方法数量是 4 个：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImRiaaibzSPpoCM8BtugLmdqR98Au4xs3M2MDBS95VqqNzZhU8RELROvEA/640?wx_fmt=png&from=appmsg#imgIndex=13)

点到函数中去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImCFZJpnRHetqGxaqgkHqoGpoTzjiaEN10FB06Xbicjv7hfV1Nypr2uSicQ/640?wx_fmt=png&from=appmsg#imgIndex=14)

可以看到我们要的方法 xPreAuthencode 的偏移是 396C8，按下 G 搜索该地址，定位到 `sub_396C8` 函数，这里的 `sub_函数偏移` 是 ida 给我们取的函数名：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im9dYQUNGiaadCvfOGtaoiaWb8FpAnoBtk8EVjQh8Q4I2W8AwGY9ic368kQ/640?wx_fmt=png&from=appmsg#imgIndex=15)

因为静态注册和动态注册函数，第一个参数值类型都是 JNIEnv，按 y 修改一下 a1 指针的类型为 `JNIEnv * a1`，这样 ida 就可以识别 JNI 相关的方法了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImCgF808x0L5ayhicVGRDcr8SAuicUxSv6TXzlbs2qfiaJj00AYRoM2eQHQ/640?wx_fmt=png&from=appmsg#imgIndex=16)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImjMIDaCZA1zZhaKZIK6fIPV6Uic6Q0eNKP8jX8icoeJYziawHFBuqGQE5w/640?wx_fmt=png&from=appmsg#imgIndex=17)

这里为了方便分析，直接使用 traceNative 工具帮助我们分析，下载地址如下：

> Pr0214/trace_natives：https://github.com/Pr0214/trace_natives

我们把项目里面的 `traceNatives.py` 文件复制到我们本地 ida 目录下的 plugins 文件夹里面，再次打开 ida，我们就有该插件了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImXMfcNPKobnBBoOo5MCIjmEPpgA9ClP87Ykswrys7I6RsFQ3kIC49qQ/640?wx_fmt=png&from=appmsg#imgIndex=18)

运行插件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im6De3pVaADlZOsTsAuwOuEH6mawr3h90BXouhl7mx0hhLZOh9dpHuDw/640?wx_fmt=png&from=appmsg#imgIndex=19)

结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Imd15gtwRGxsRIUZMiaMYxRiaZcqsQiapRpLncvfH1t2yChcxCMd0fDS3Ig/640?wx_fmt=png&from=appmsg#imgIndex=20)

traceNative 插件会把 so 文件里面相关的函数调给 hook，放入到 `__handlers__` 文件夹中，等待函数调用，我们刷新一下 app，会得到相关函数的调用堆栈：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImibMylgrv0eqsaUMHNu0tIBIlnvxBe5MFTnZdPDvn6QejHVFW3XQm7Jw/640?wx_fmt=png&from=appmsg#imgIndex=21)

我们把需要分析的函数的调用堆栈单独拿出来：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImEGiaic41qxnzA3BJDGBia3tLJWhKW3OhaVmAhIPQibuvGM6EicxUQ1R3bHw/640?wx_fmt=png&from=appmsg#imgIndex=22)

会发现加密函数首先调用了 `sub_3c9c4`，然后一直在调用 3e1d0 函数。我们先按 g 搜索 3c9c4 进入这个函数，按住 y 修改函数第一个、第二个参数类型：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Imsy8ic3CCeHYo9jaLiazVbblHMahsURgEC9Sc9DrTacAK4toGEnwda7GA/640?wx_fmt=png&from=appmsg#imgIndex=23)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImNAHB9PfU0CoV44XHAH3E9jkSBraefzsDTpdKppESEIb8KvvDB7s71w/640?wx_fmt=png&from=appmsg#imgIndex=24)

这里获取到了 app 相关的签名信息，并调用了几个函数，最后成功执行返回布尔值 1。

接着再往下分析 `sub_3e1d0` 函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImDgG8A7Uos5bqp7wQ5scOQbuciaPTXDaHw6gctfTeahL8lNJib8Z1MTBw/640?wx_fmt=png&from=appmsg#imgIndex=25)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImoLKyxibtQnhhTiaicMv6bprTs455CGQNeXkDpLlmqWzONLyT7yvprMNkg/640?wx_fmt=png&from=appmsg#imgIndex=26)

hook 一下这个函数：

```
function hook_sha1(){
    var addr = Module.findBaseAddress("libmfw.so");
    // console.log(addr)
    var func_addr = addr.add(0x3E1D0);
    // console.log(func_addr);
    Interceptor.attach(func_addr,{
        onEnter:function (args){
            console.log(args[1].readCString())
            console.log(hexdump(args[1]))
            // console.log(args[0].readCString())
        },
        onLeave:function (retval){
            // console.log(retval)
        }
    })
}
hook_sha1()


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImDSoOdKRRUFr1D5yrPZVqBepWYy60CZmsMwo2ZdXj1nztibVMeH9GX4A/640?wx_fmt=png&from=appmsg#imgIndex=27)

这个函数调用了很多次，对明文进行不断的处理，另外出现了很多 1518500249、1859775393、1894007588 等常数，这里为了方便观看，我们转化为 16 进制数据：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImhynWRic79vOtGC6Ar2r71o0RQpIJbpVRCBaCSwnggCG74heBaQrGVbA/640?wx_fmt=png&from=appmsg#imgIndex=28)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImelIlFjKxibCH2JzBeUoKT6OerGkY999dFwfbXB1G57Wmn3ibd5VqibQrw/640?wx_fmt=png&from=appmsg#imgIndex=29)

根据加密生成的位数以及这些常数，推测出为 sha1 算法，可以使用 k 哥工具站：https://www.kgtools.cn/secret/sha，来验证是否为标准算法：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImK9I1MXbNicGPR1RLDEnu0MhGmJQVTAn8libtpbpbshPFpUz3XWTd8cDQ/640?wx_fmt=png&from=appmsg#imgIndex=30)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImdMibNko9miaLMHrbf9ian8mX1mtGlw6CCibC3QrWev2dLz5NXuf0JxiahJw/640?wx_fmt=png&from=appmsg#imgIndex=31)

可以看出不是标准算法，那么就有两种基本方案，第一是扣魔改的 sha1 算法，一步步分析是哪里魔改了。第二种就是使用 unidbg，补 unidbg 相关环境，让 unidbg 去执行 so 文件代码，得到最终的加密结果。

这里简要分析一下 sha1 算法的初始化常数是否魔改。回到 xPreAuthencode 函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Ima4k22uSXw4Al11VfXZnlbhUkbJuALnDyE1TEocv4D3wDQWPvQLNxIw/640?wx_fmt=png&from=appmsg#imgIndex=32)

hook 该函数：

```
function hook_sha1(){
    var addr = Module.findBaseAddress("libmfw.so");
    // console.log(addr)
    var func_addr = addr.add(0x3DEC4);
    // console.log(func_addr);
    Interceptor.attach(func_addr,{
        onEnter:function (args){
            console.log(args[0].readCString())
            console.log(hexdump(args[1]))
            console.log(args[2].toInt32())
        },
        onLeave:function (retval){
            // console.log(retval)
        }
    })
}
hook_sha1()


```

hook 结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImEp1qjfzEic0IruhAWD6vovnGId2fcuQpOWkcZNdYHibRM6VrzreQm6SQ/640?wx_fmt=png&from=appmsg#imgIndex=33)

其中包含了 sha1 算法加密的明文参数和明文长度，`&v38` 包含了 sha1 算法的初始化模数，跳转到该函数位置：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im5Mb71QJeLeYJODVNB5bmDcfxOiaHstqtPalW4DgIPXSpjFsjtCk59HQ/640?wx_fmt=png&from=appmsg#imgIndex=34)

点进 C0C30，并按字母 D 把伪指令 DCB 改成 DCD。DCB 占两个字节，DCD 占四个字节：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImdUuYkw8C47PJbs9Cia6l7UHbHNTEyroFMSLCtnTNHsyw11ibf2lzSsEQ/640?wx_fmt=png&from=appmsg#imgIndex=35)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImHF7xUX7kRsxibpotQ1YUrWrzmyEffRHUxeKia2Z1mSud6VwpaMkXV35w/640?wx_fmt=png&from=appmsg#imgIndex=36)

还有一个初始化模数通过指令的方式进行加载：

```
# 指令将值 0x5476 加载到寄存器 W8 的低 16 位中
.text:000000000003DEEC                 MOV             W8, #0x5476
# 将值 0x1032 加载到 W8 的第 16 位（高 16 位）中,不影响其他位的值
.text:000000000003DEF8                 MOVK            W8, #0x1032,LSL#16

# 最终得到的结果为：0x10325476


```

因此，sha1 的初始化模数为：

```
A = 0x67452301
B = 0xEFCDAB89
C = 0x98BADCFE
D = 0x5E4A1F7C
E = 0x10325476


```

可以看到初始化值已经魔改，那么后面就需要分析 sha1 算法是否在填充、明文、80 轮循环的什么地方进行了魔改，网上已经有教程，这里就不多介绍。

unidbg 还原
---------

unidbg 是一款基于 unicorn 和 dynarmic 的逆向工具，一个标准的 java 项目，它通过模拟 Android 运行时环境，让用户能够在没有实际设备的情况下，分析和调试 Android 应用的行为。

> zhkl0228/unidbg：https://github.com/zhkl0228/unidbg/releases

在使用之前，你需要提前配置好 java 和 maven 环境，安装配置网上有很多教程，这里不多做介绍。

关于 unidbg 的更多使用方法，可以看吾爱破解正己大佬写的文章：

> 《安卓逆向这档事》第二十三课、黑盒魔法之 Unidbg：https://www.52pojie.cn/thread-1995107-1-1.html

另外作者也给了我们相关代码示例，我们只需要稍微修改一下就能用，主要代码如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImfW6wwdXzAgUaSiaR6AAVpjr6ViaOibzwReDA7nykDF1VQ3kyz80JHDIDA/640?wx_fmt=png&from=appmsg#imgIndex=37)

执行 TTEncrypt 文件，看看我们的环境是否有问题：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImEr2GkEfFtlA5DuhibYViaW2V0zrtk0xydenhoUEInW98YwHRMH6zurMQ/640?wx_fmt=png&from=appmsg#imgIndex=38)

能正常执行，接着我们在 com 目录下，新建自己的目录，这里命名为 mafengwo，再在该目录下新建 java 文件为 Mfw：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im2KFP3Q09EFXTh3SoiakgpAjibGrfKrC7TPR6ZpjQG1qUwN73jZQglS6Q/640?wx_fmt=png&from=appmsg#imgIndex=39)

把作者给的样例，例如 TTEncrypt 代码 copy 一份，改一下，这个 app 检测环境较少，可以算入门 unidbg 的案例，基本上简单修改完作者给的代码样例就能使用。

最终代码如下：

```
package com.mafengwo;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.memory.Memory;

import java.io.File;
import java.io.IOException;

public class Mfw extends AbstractJni {

    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private final DvmClass AuthorizeHelper;
    private final boolean logging;

    Mfw(boolean logging) {
        this.logging = logging;

        emulator = AndroidEmulatorBuilder.for64Bit().setProcessName("com.mfw.roadbook").build(); // 创建模拟器实例，要模拟32位或者64位，在这里区分
        final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
        memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析

        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/java/com/xxx/mafengwo/com.mfw.roadbook.apk")); // 创建Android虚拟机
        vm.setJni(this);
        vm.setVerbose(logging); // 设置是否打印Jni调用细节
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/java/com/xxx/mafengwo/libmfw.so"), false); // 加载libttEncrypt.so到unicorn虚拟内存，加载成功以后会默认调用init_array等函数
        dm.callJNI_OnLoad(emulator); // 手动执行JNI_OnLoad函数
        module = dm.getModule(); // 加载好的libttEncrypt.so对应为一个模块
        AuthorizeHelper = vm.resolveClass("com/mfw/tnative/AuthorizeHelper");
    }

    void destroy() throws IOException {
        emulator.close();
        if (logging) {
            System.out.println("destroy");
        }
    }

    public static void main(String[] args) throws Exception {
        Mfw test = new Mfw(true);
        test.callFunc();
        test.destroy();
    }

    void callFunc() {

        String data = "GET&https%3A%2F%2Fmapi.mafengwo.cn%2Fdiscovery%2Fget_index%2Fv7&app_code%3Dcom.mfw.roadbook%26app_ver%3D11.0.2%26app_version_code%3D1052%26brand%3Dgoogle%26channel_id%3DMFW-WDJPPZS-1%26dev_ver%3DD2313.0%26device_id%3D82d917db80c8eae2%26device_mid%3D860000000000001%26device_type%3Dandroid%26hardware_model%3DPixel%25203%26has_notch%3D0%26jsondata%3D%257B%2522top_tab_id%2522%253A%252255%2522%252C%2522filter_id%2522%253A%2522all%2522%252C%2522top_refresh%2522%253A%25221%2522%252C%2522by_user%2522%253A%25221%2522%257D%26mfwsdk_ver%3D20140507%26o_coord%3Dwgs%26o_lat%3DMzAuNDg5NjIy%26o_lng%3DMTE0LjQyMDQ5MQ%253D%253D%26oauth_consumer_key%3D5%26oauth_nonce%3D536f6e42-bc4c-463f-acb5-ee145e63f10e%26oauth_signature_method%3DHMAC-SHA1%26oauth_timestamp%3D1732784973%26oauth_token%3D0_0969044fd4edf59957f4a39bce9200c6%26oauth_version%3D1.0%26open_udid%3D82d917db80c8eae2%26screen_height%3D2028%26screen_scale%3D2.88%26screen_width%3D1080%26shumeng_id%3DDUx0wb9S3BzdWvrdsQ8G8T6MnVVJL6kbZPb2RFV4MHdiOVMzQnpkV3ZyZHNROEc4VDZNblZWSkw2a2JaUGIyc2h1%26sys_ver%3D11%26time_offset%3D480%26x_auth_mode%3Dclient_auth";
        StringObject strResult = AuthorizeHelper.callStaticJniMethodObject(
                emulator,
                "xPreAuthencode(Landroid/content/Context;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;",
                vm.addLocalObject(vm.resolveClass("android/content/Context").newObject(null)),
                new StringObject(vm, data),
                new StringObject(vm, "com.mfw.roadbook")
        ); // 执行Jni方法
        System.out.println(strResult);
    }

}


```

最后也是成功返回了加密数据：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImJxQGSbg2clzU6XJLzhZ5ukprNQDcFsvdcO8MwJwGKhrliaVJCckpbnw/640?wx_fmt=png&from=appmsg#imgIndex=40)

成功打印了加密结果，那么我们怎么给 python 使用呢？

unidbg-boot-server  搭建接口服务
--------------------------

我们可以使用 unidbg-boot-server 项目，作者已经帮我们封装好了相关代码。下载地址：

> unidbg-server 提供 http api 服务：https://github.com/anjia0532/unidbg-boot-server

和 unidbg 使用一样，需要配置好 java 和 maven 环境，用法和 unidbg 差不多，我们把项目下载下来：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImyNnn4yiafsNUVoBynE4J8vbeHShFA07VMLphEdibKDNKe9pAugu0dXOw/640?wx_fmt=png&from=appmsg#imgIndex=41)

相关配置环境加载好后，直接运行 UnidbgServerApplication 文件，显示以下内容证明环境没有问题，可以正常执行：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImJEX733NMRwFTt865vFWd5UO1ibZeP4eDuDEPSuT0aOFBDTDOz9oLTuQ/640?wx_fmt=png&from=appmsg#imgIndex=42)

作者也很贴心的给我们准备了代码样例，主要逻辑都在 com.anjia.unidbgserver 里面。

其中 `web.*Controller` 目录是暴露给外部 http 调用的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImibPeQaUY6gzcSIWKGocickeRJwbPzdfHibXDibI2ThibfwgKAPWExW0ibIUA/640?wx_fmt=png&from=appmsg#imgIndex=43)

`service.*ServiceWorker` 目录是用多线程包装了一层业务逻辑，主要用来调用我们的加密逻辑`service.*Service` 里的加密函数相关的逻辑，也就是 unidbg 的代码。

ServiceWorker：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImtzwklHhwTs0gp9XrBqg70dRxGkgWMghrT4XOU9rJhgia0NbS0RUn8XA/640?wx_fmt=png&from=appmsg#imgIndex=44)

Service：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Imh0RIVy8jOnQSic9MSXaicoY3BT9yR3xpDY0jUzFzZrdvhqrw3tshSl5g/640?wx_fmt=png&from=appmsg#imgIndex=45)

仿造代码样例，写出加密函数的代码，运行 UnidbgServerApplication 结果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImmV1rNOXRklMpcMV2IRSp6WWiaUdBhu3cKyVENAwp7dBwWO8WaxCjO4g/640?wx_fmt=png&from=appmsg#imgIndex=46)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0ImiaWNppAgEM31tO34iatFYyLx8NibfOmsicJ0kuLryE5OviaaauUxwZTBmxQ/640?wx_fmt=png&from=appmsg#imgIndex=47)

最后通过 python 请求这个接口就可以得到加密参数了，不再赘述。

至此，该参数加密分析流程就结束了。

相关代码，会分享到知识星球当中，需要的小伙伴自取，仅供学习交流。

结果验证
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTo1y7tlrypNf47jblj2Q0Im0Vf1xXhxXXfSahO2iaORg7mEibnERr8rRuMBp0zxUiaicwkMyk1DypicGLw/640?wx_fmt=png&from=appmsg#imgIndex=48)

 ![](https://mmbiz.qpic.cn/sz_mmbiz_gif/iabtD4jabia4KDdF6jxLibSq5ssaiaicicKHf2VWdrkFqrsRuDF7CiaKMxAeua0WeLPFmOIQkgcCt66o7L2uOl1wRVuVw/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=49)

**（ 先别划走 点个关注 Orz ）**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/POicI8TV3kTqk3p9iaszC2ibDWOXPQ3e0aCy3zsOLCDOV6ZbGbedyRNqfsqWUODEFC5B4nnbhAiaKmslJL07ruia4og/640?wx_fmt=png&from=appmsg#imgIndex=50)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/iabtD4jabia4Isu9YmfRmf0BLWYicCG4MGM86Enex1Hgia9lEmfXibhwSo1AcGJsfbzXL5S2qCW3FialoEh535pBibKUA/640?wx_fmt=jpeg#imgIndex=51)

**国内外代理 IP 推荐**

  

******隧道代理 Pro（免费测试体验 1 天）******

  

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/iabtD4jabia4Lkias4uJRLk5Eey033Zk5CSuOXnkWEcJdAboIzMqzTARLwFsDZJm4uTxaKhMdCyFcpBcXnSpHBYhw/640?wx_fmt=png#imgIndex=52)](https://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485066&idx=1&sn=5972621f35e8ff81c2beb9b4f49f5ee7&scene=21#wechat_redirect)

********独享代理住宅版（新上线七五折）********

  

[![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/POicI8TV3kToZDMot3WV8jCNDHaq9ictIffsyibcK1icj9ctyXkiaj7ZMHO0yvobiau2nHdFuf4V8me2Ukrm34al0X6A/640?wx_fmt=jpeg&from=appmsg#imgIndex=53)](http://mp.weixin.qq.com/s?__biz=MzkyMDIxNTM1OA==&mid=2247485084&idx=1&sn=e67e88b52b9b4a1d35711ce7fbddb7f5&chksm=c1970ccdf6e085db5e7a2927fcb1359f92bdec97761ae63c67b5f390bbf547bbeba6cf00db99&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/7VAgNKQgCMFM8ia5BA9MLZhlCnRr8Er4gR4Rjr7WBmby6jKvlqpH7jZITFBYBIYbibfOgHRCF5obiaJn6yzC321qw/640?wx_fmt=png#imgIndex=54)

**点个****在看****你最好看**

![](https://mmbiz.qpic.cn/mmbiz_png/NtgFk2rGpiaOPxvr7Ls916UDdGAibFN8ObxF6VKc8qCT18luCwKTUgHicBiaMYJE9SIdicQHL7ouCt8xk7tMtsxKayA/640?wx_fmt=png#imgIndex=55)