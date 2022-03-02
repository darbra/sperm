> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5ODQ2NTU1OA==&mid=2247483690&idx=1&sn=27bf7e794923eddb33db649e8cff3c78&chksm=c0636b53f714e2455ee92ea38d24c29d75a3a60054c3c328739eea2fad641748b9e72ea0c6c6&mpshare=1&scene=1&srcid=0302WsjmRKB1KVsoGm0GKQrv&sharer_sharetime=1646203591311&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

转发说明：

本文章对 app 的逆向分析全程使用平头哥工具链、包括 hook、插件、抓包、脱壳等。除此之外平头哥的多开分身、自动化框架、rdp、调度任务框架、sekiro 继承的 RPC 框架等。可以覆盖移动安全分析的多个业务需求

对平头哥感兴趣的同学，可以添加微信：virjar1，备注平头哥，拉入微信交流群

以下为黑同学对平头哥使用的文章全文：  

使用渣总的平头哥对这个 app 进行脱壳，抓包，破解，分析

```
平头哥项目地址：https://github.com/virjarRatel
克隆如下图的项目模板地址：https://github.com/virjarRatel/ratel-module-template

```

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0FibY4HLfpc79mE5yuPPwkAricVK0kNUDpopvI8vL7gib3sEHWTuExiaKvw/640?wx_fmt=png)

克隆下来之后用 Android Studio 打开这个项目

我们通过以下命令来快速创建平头哥项目，因为我是 win 所以使用 bat 来创建

linux 要使用 sh 创建 

```
template.bat -a C:\Users\LEGION\Desktop\ptg2.0\com.nst.android_7.5.5_408308_ratel.apk -m create_nst

```

命令：template.bat -a 【指定你的 apk 所在的路径】-m 【给你的插件项目起一个名字】如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0v2BKhZ6htpQDaWk4rAH3hsQnIl9pQdicFyVGhp4h39a7RwDKEQSwsWw/640?wx_fmt=png)

就会生成一个后缀名字为 create_nst 的插件 要在 settings.gradle 设置里面添加你生成的 app 的名字，看到有一个小绿点说明我们配置的没有问题了，接下来就要开始搞 app 了

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH06XGELnCh5nRsU6wsITSicSoN0pIhKEHqeFiceTBrgW5XmgfWqWA2P6Jw/640?wx_fmt=png)

打开创建好的项目会发现有一些代码逻辑，我们可以给他删除掉编写自己的代码逻辑  

样本 app 手机商城直接搜索 农商通  下载即可 jadx 打开发现这个 app 是加固的所以我们要用平头哥进行脱壳，

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0KxyhFNGAFhEzXasAJU0cTTNwfu26dPicJjZ2mNz1d8iaJrAfRAm09ibWA/640?wx_fmt=png)

平头哥脱壳代码 如图所示 因为脱下来的壳会 copy 到 / sdcard/xxxx.apk 这个路径下

```
public class Unpack {
    public static final String tag = "TR_HOOK";
    public static void entry(RC_LoadPackage.LoadPackageParam lpparam) {
          // TODO 抽取壳要把这个选项打开
//        UnPackerToolKit.autoEnable(true);
        UnPackerToolKit.unpack(new UnPackerToolKit.UnpackEvent() {
            @Override
            public void onFinish(File file) {
                try {
                    FileUtils.copyFile(file, new File("/sdcard/nst-unpack.apk"));
                } catch (IOException e) {
                    Log.e(tag, "unpacked error", e);
                    e.printStackTrace();
                }
            }
        });
    }
}

```

注意：这个 app 必须是平头哥感染后的应用才可以 

下面安装插件进行脱壳 点击这个按钮等待安装  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0fgQ0zSHvffgTqWib4Y4VKxZrRWlIfEFbxruTFq3UWDibZAE7s9KwpUww/640?wx_fmt=png)

安装成功之后在会在 RatelManager.apk 中看见如下 app 就是感染成功了

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0jXsxYVNm5NvAibLDEXHICKuIL1DSAUibspor9iaTRV1K2IHhqSqN6GFjw/640?wx_fmt=png)

下面运行我们写好的插件开始进行脱壳  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0wfKvjjejvxZeribvkKxdsXCgHeViaOpujU8NLl1K3BzF3Kr7b55S45uw/640?wx_fmt=png)

进入 cmd 输入 命令  adb logcat -s unpack 然后点击 app 倒数 三二一

就会看到脱壳日志 平头哥会重新组装成一个 apk，为了 jadx 更好的分析看到平头哥最后打印的日志我们去这个路径下看看

```
/data/user/0/com.nst.android/files/ratel_unpack/unpacked.apk

```

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0BG85vqS0xsxbqmicXT74bWrBZHBbMkk3vNiaq3jk497GCrZ2b9kjOPIQ/640?wx_fmt=png)

可以看到这个有很多 dex 文件，还有一个 apk 证明平头哥脱壳成功了  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0Gias4B8JuyA2WFaGzia6DaTjMuzDibpbSsFd9slCXHgiaZCibYiaNMvpcWLw/640?wx_fmt=png)

因为我们脱壳代码写的是把脱壳 apk copy 到这个路径下所以我们直接看这个路径下有没有

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0TcOwh2tdlej6G7eMs9axlJnbYVL4ratvj7ZUc5NQ7TSyVdDzC1qyCQ/640?wx_fmt=png)

发现是有这个 apk 的，我们只需要拿这个 apk 分析就可以了，还有一种情况是拿不到这个 apk 的，原因是你在这个插件里面设置了一机多号的功能

如果设置一机多号功能是有一个默认用户（default_0）需要在下面路径下寻找的  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0CgrhbhbM4TB0oEeYibnbNQOFaqwbBXVI2B4stotNQCmePvNu0XLHLDg/640?wx_fmt=png)

也可以直接导出 dex 分析也是可以的

现在我们开始抓包，为了方便我就先用 fiddler 抓包 发现有一个 x 签名和 okhttp3

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60owDKrsHvaJRxJVINdVsgH0NCxYhwLtscLygES07BC36vJuJDcd9GqyTLEibpMg1mr6rI0afkI9krg/640?wx_fmt=png)

下面用平头哥进行抓包和堆栈打印 非常简单建议去看渣总的平头哥的文档 重新安装插件进行抓包

```
SocketMonitor.setPacketEventObserver(new FileLogEventObserver(new File(RatelToolKit.whiteSdcardDirPath, "socketMonitor")));

```

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaBfGBepx6l4EqpibRypMiaCltu2m3McSkHgIcbC7ggOgF2PeAWzPLWxQA/640?wx_fmt=png)

默认抓包文件是在 / sdcard/ratel_white_dir/com.nst.android 路径下的 socketMonitor 文件里面，我们直接导出分析

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaLkkmhOhPDsQKiceE5qkTiaXNTTv3cWkpN4b1Qlib1jSuiaVKgzVKQrYHng/640?wx_fmt=png)

我们对比 fiddler 抓包发现基本上报文响应是一样的  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGacZHYDK2gkFgv5lY2kpPJQIWN1TNCOwCghbvIzb6pibo5Jj27dYibwuZA/640?wx_fmt=png)

因为平头哥下面打印出调用栈了，又是 okhttp3 的请求库所以我们直接看调用栈就会发现下面的调用栈可能就是这个 app 的收发包函数，因为 okhttp3 是系统库所以我们不用关心，下面去找这个发包位置  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaq6FUtnYY0284tWh4OQe69UFUTldZpIrvYAf4Wp7WCdGMvfHEJjBqNg/640?wx_fmt=png)

经过查找果不其然是这个发包函数在下面我们正好看到了有关签名的信息 me.androidlibrary.network.okhttp.OkhttpFactory$1.intercept

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaYsjeXWnxtibcHWtrT3fOQ4wiaULQzGfOEuACNibwylrV9S5oJToe8w0EQ/640?wx_fmt=png)

之前抓包的时候对比过有一个 “x” 的签名我们根据平头哥的调用栈追溯到 app 发包函数的具体位置当然这个是一个拦截器，签名逻辑就是在拦截器里  

具体的签名逻辑就是这个 我们只关心 x 的签名就可以了因为这个是没有登录的情况下

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGa7CH1bdsB3RHUibXWzvLiacZsJ0p7icr4LGpwkawibvMNo6GcRBEAVf7PHA/640?wx_fmt=png)

x：的签名逻辑发现是调用的这个类下的这个方法

再往下追溯就到到了加密实现的类标准的 aes 算法  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGattABzsTIBrWricrKHj328UxshsOJCmUTThp4hibSf1uw1KEGwTh830cg/640?wx_fmt=png)

我们开始 hook 这个类中的方法

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaPHrK4NncTHmzAbLngKwnSEB4DOp0mSicmdcx3Kh1goYIqg7QLqOQNfw/640?wx_fmt=png)

其实到这里我们已经分析的差不多了，现在要做的就是知道她的明文是什么就可以了，我们开始 hook 这个 开始写平头哥插件其实和 xposed 的写法是一摸一样的

**因为是带壳的 app，渣总对平头哥做了封装 所以我们可以使用下面的 api 进行获取 apk 运行的类加载器，这个样我们就不用手动拦截 attch 的方式获取 context**

**可以看到使用平头哥的插件 hook 的就是加密后的返回的结果**

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGabBic7aVrFRRibOTYkyTbpoZTAfgOs3gOEnFlwxic0EmUnoiaD7FaFb7JBQ/640?wx_fmt=png)

最后的明文加密数据就是就是 13 位的时间戳 + 一个固定的 key + AES 算法的中的 CBC 模式的 vi 偏移量  

vi: 固定值 0312032293271340

str：就是时间戳 13 位的  

str2：也是固定值 key 45ryu230a@n2x302

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGag3h33Jt8s5XSbVibzvjKyVODbFjPA4rlgl5rBkBatm29iaXdxia5F6MuA/640?wx_fmt=png)

最后献上平头哥 hook 代码

```
package ratel.com.nst.android;
import android.util.Log;
import com.virjar.ratel.api.RatelToolKit;
import com.virjar.ratel.api.extension.FileLogger;
import com.virjar.ratel.api.inspect.ClassLoadMonitor;
import com.virjar.ratel.api.rposed.IRposedHookLoadPackage;
import com.virjar.ratel.api.rposed.RC_MethodHook;
import com.virjar.ratel.api.rposed.RposedHelpers;
import com.virjar.ratel.api.rposed.callbacks.RC_LoadPackage;
public class HookNst implements IRposedHookLoadPackage {
    private static final String tag = "NST_HOOK";
    @Override
    public void handleLoadPackage(RC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        Log.i(tag, "start");
        ClassLoadMonitor.addClassLoadMonitor("com.nst.android.utils.aes.AESUtils", new ClassLoadMonitor.OnClassLoader() {
            @Override
            public void onClassLoad(Class<?> clazz) {
                Log.i(tag, "find class: " + clazz.getName());
                RposedHelpers.findAndHookMethod(clazz, "encrypt",
                        String.class,
                        String.class,
                        new RC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        String arg1 = (String) param.args[0];
                        String arg2 = (String) param.args[1];
                        Log.i(tag, "加密前的参数1：" + arg1);
                        Log.i(tag, "加密前的参数2：" + arg2);
                        Log.i(tag, "-------------------------");
                    }
                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        super.afterHookedMethod(param);
                        String agr3 = (String) param.getResult();
                        Log.i(tag, "加密后的结果：" + agr3);
                    }
                });
            }
        });
        Log.i(tag, "end");
    }
}

```

最后线上 python 算法还原这里是搜了一位大佬的博客，我是直接拿来用的  

```
import base64
from Crypto.Cipher import AES
def AES_Encrypt(key, data):
    vi = '0312032293271340'
    pad = lambda s: s + (16 - len(s) % 16) * chr(16 - len(s) % 16)
    data = pad(data)
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, vi.encode('utf8'))
    encryptedbytes = cipher.encrypt(data.encode('utf8'))
    # 加密后得到的是bytes类型的数据
    encodestrs = base64.b64encode(encryptedbytes)
    # 使用Base64进行编码,返回byte字符串
    enctext = encodestrs.decode('utf8')
    # 对byte字符串按utf-8进行解码
    return enctext
def AES_Decrypt(key, data):
    vi = '0102030405060708'
    data = data.encode('utf8')
    encodebytes = base64.decodebytes(data)
    # 将加密数据转换位bytes类型数据
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, vi.encode('utf8'))
    text_decrypted = cipher.decrypt(encodebytes)
    unpad = lambda s: s[0:-s[-1]]
    text_decrypted = unpad(text_decrypted)
    # 去补位
    text_decrypted = text_decrypted.decode('utf8')
    return text_decrypted
key = '45ryu230a@n2x302'
data = '1633919358891'
AES_Encrypt(key, data)
enctext = AES_Encrypt(key, data)
print(enctext)
text_decrypted = AES_Decrypt(key, enctext)
print(text_decrypted)
# python实现的算法，和hook出来的是一摸一样的

```

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGa0qlACP8ficXfV58oVNYHZmHrkiaODvvcQ94kV9BqK7mticVH8FQGDpllQ/640?wx_fmt=png)

最后试试请求可以成功不 成功了，说明我们的算法是没有问题的现在就可以愉快的玩耍了  

![](https://mmbiz.qpic.cn/mmbiz_png/icp3x3Pxg60ppUaVFHPm4ib3dkic21TqfGaiaXg843Mkwf7V6gB7xeEmmxmXP0J6vGJltCrLceQ7LOKCHdhGfiaj7HQ/640?wx_fmt=png)