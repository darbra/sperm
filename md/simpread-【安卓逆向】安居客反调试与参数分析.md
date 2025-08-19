> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/goqtsr3vaBocHz59NctmKg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpXCsMKeGcRgL6FAZ6W54HFddJleBGcibI0EvPlpvendXUNrg9ZDWVRZw/640?wx_fmt=jpeg&from=appmsg&watermark=1)

  

集美语录
----

> ❝
> 
> “婆婆住院，老公让我拿彩礼钱应急，我没同意后老公记恨我怎么办”

  

介绍
--

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpUEcNytTF53TT3pNaIicwDUk8KtpibWNRtArYUB6ZUdelHt7bAfYuBsWg/640?wx_fmt=png&from=appmsg&watermark=1)

ssl
---

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpsDJxSxO9Vy6wkEQf2tfMOLM9asVXX5gyu1J3KgibvEjRgUvyquSbjyg/640?wx_fmt=png&from=appmsg&watermark=1)

本来想用 frida 脚本过下 ssl 的，发现有 frida 检测，直接用 truastme 吧

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpD6ptWnV4sxpyvf0xaAlMPkYtwMBPuciaAicO9Fww3yIGldzVvkHharvA/640?wx_fmt=png&from=appmsg&watermark=1)

好了

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpWGbDibsKnxB9bXXk56ibZQAHOsicdgSuAYDvJcI8MxicDdDwBV1kqicNTNg/640?wx_fmt=png&from=appmsg&watermark=1)

frida 检测
--------

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpvTUibic84Y11vx6Zz6qRh37ukcjQ13FkUKYQYowibGMvS5xVdWhhGnUVQ/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUp7vw50icRBsTiafduupjOsXvUE12EeqKwZGUBpEguibnBT49rL1BdmkYOw/640?wx_fmt=png&from=appmsg&watermark=1)

ida 看下这个线程

```
void __noreturn sub_11E1C()
{
  unsigned int v0; // r0
  __useconds_t v1; // r4
  int v2; // r0

  v0 = sub_81FC();
if ( v0 < 0x64 )
    v1 = 2000000;
else
    v1 = v0;
while ( 1 )
  {
    v2 = sub_117F0();
    if ( v2 == -1 || v2 && !sub_11630(v2, v2 + 1) || sub_11CF8() == 777 )
      sub_BA20();
    usleep(v1);
  }
}


```

很像一个监测点啊

1 死循环 + usleep  
2 检测函数三连

```
sub_117F0()                 // 读取 /proc/self/maps 或 /proc/self/task 等
sub_11630(...)             // 字符串匹配（搜 “frida”、“gum-js-loop” 等）
sub_11CF8() == 777         // 第二个检测函数返回特定魔数 → 触发退出


```

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpsweVFSaWiaIJLlx0FZ8kIw8D6wxpd4IGTfWvkcf9CWtYRL8d8sGTq8Q/640?wx_fmt=png&from=appmsg&watermark=1)

解决方案可以看之前的文章，这里也可以直接把 libmsaoaidsec  android_dlopen_ext 进入时候把 path 改空阻止加载

```
function hook_dlopen(soName) {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                if (path.indexOf(soName) >= 0) {
                    ptr(pathptr).writeUtf8String("");
                }
            }
        }
    });
}
hook_dlopen('libmsaoaidsec.so')


```

当然找到了这个循环检测的函数后，也可以把它 Nop 了，同时把闪退的函数 BA20 也杀了，也可以过检测

```
const TARGET_SO = 'libmsaoaidsec.so';

Interceptor.attach(Module.findExportByName(null, 'android_dlopen_ext'), {
    onEnter: function (args) {
        this.path = Memory.readUtf8String(args[0]);
    },
    onLeave: function () {
        if (this.path && this.path.indexOf(TARGET_SO) >= 0) {
            const base = Module.findBaseAddress(TARGET_SO);
            console.log(`[+] ${TARGET_SO} loaded @ ${base}`);

            const loopOffset = 0x11E1C; 
            const loopAddr   = base.add(loopOffset);
            const exitOffset = 0xBA20;
            const exitAddr   = base.add(exitOffset);
            
            Interceptor.replace(loopAddr, new NativeCallback(() => {
            }, 'void', []));

            Interceptor.replace(exitAddr, new NativeCallback(() => {
            }, 'void', []));
        }
    }
});


```

done![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpXKWkfFArNvO504yvT1ZYGfPpAk80KvbPCOAEIJERnTneVgeOo3deGw/640?wx_fmt=png&from=appmsg&watermark=1)

nsign
-----

请求头的 nsign 是我们分析的目标

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUppGYrybbI0icIsys5yfs9T1W1XYgUFjBVW1ZZhBhpErbRzFVmsotCdLQ/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpicPTvpeIUAh7ibtict5YR0uictqyZL5TGpF8BVRuuxj4TeRkickyaOQdFNw/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpPMVZwxWqW1ctUWyhKHscnGfD1YQqFTvOFztwCgAy9L5klaQIzrialsA/640?wx_fmt=png&from=appmsg&watermark=1)

```
Get：
c = SignUtil.c(path, null, b2, uuid);

参数
path=/mobile/v5/sale/property/list
uuid = UUID.randomUUID().toString();
b2 = params

Post:
c = SignUtil.c(path, c2, b2, uuid);
newBuilder2.add("body_md5", SignUtil.a(c2));
c2 = byte[](payload)


```

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpwEHHCwdfnoM4ibbJpR4gWrlukU7Y1NseIibEJ5LJbNx7B54Nu704iclcQ/640?wx_fmt=png&from=appmsg&watermark=1)

参数

```
str, a(bArr), hashMap, str2, bArr.length

str = path
a(bArr) = md5(byte[]) // get 为md5('')
hashMap = params  key字符串  值是value的bytes
str2 = uuid
bArr.length  byte[]长度


```

hook 下静态注册，发现实在 libsignutil.so 里面

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpzud9BZhXVTXIsj8oic2R0Gzg5aicUunEdSFyumR7yl3wicItDia2dGIdzg/640?wx_fmt=png&from=appmsg&watermark=1)

看下参数对不上，前两个字符串好像是从 a2 jclass 中读取的

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpItTr5NFKacGV0U9VbEV77SvibfZHEp5GMKSfoem4JLQqTYmGtdibMSuA/640?wx_fmt=png&from=appmsg&watermark=1)

```
v42 = a2;
v23 = (void *)v42;
v24 = (*a1)->GetStringUTFChars(a1, v42, 0);
v25 = (*a1)->GetStringUTFLength(a1, v23);
v26 = (void *)HIDWORD(v42);
v27 = (*a1)->GetStringUTFChars(a1, HIDWORD(v42), 0);
v28 = (*a1)->GetStringUTFLength(a1, v26);


```

o  函数是__fastcall，前 2 个 32 位参数通过寄存器传递

```
// 常规参数传递
R0(JNIEnv*), R1(jclass), [栈:str, str2, map, str3, i]
// __fastcall参数打包
R0(JNIEnv*), R1(高32位=str2, 低32位=str1), [栈:str, str2, map, str3, i]


```

从后往前看

```
v33 = a0123456789abcd[(unsigned __int8)a5 >> 4];
v34 = a0123456789abcd[a5 & 0xF];
v35 = a0123456789abcd[(a5 >> 8) & 0xF];
v36 = a0123456789abcd[(unsigned __int16)a5 >> 12];
v37 = a0123456789abcd[(unsigned __int8)v48 >> 4];
v49[36] = a0123456789abcd[(unsigned __int16)v48 >> 12];
v49[37] = a0123456789abcd[(v48 >> 8) & 0xF];
v49[38] = v37;
v49[39] = a0123456789abcd[v48 & 0xF];
v38 = a0123456789abcd[v30 & 0xF];
v39 = a0123456789abcd[(unsigned __int8)v30 >> 4];
v40 = a0123456789abcd[((unsigned int)v30 >> 8) & 0xF];
v49[40] = a0123456789abcd[(unsigned __int16)v30 >> 12];
v49[41] = v40;
v49[42] = v39;
v49[43] = v38;
v49[44] = v36;
v49[45] = v35;
v49[46] = v33;
v49[47] = v34;

*(_QWORD *)&v49[48] = *(_QWORD *)v47;


```

下面主要看看这个里的 j_get_sign((int)v24, v29, (int)v27, v28, base, v43, (int)v49);

```
// 低32位读取java参数str
v24 = (*a1)->GetStringUTFChars(a1, v42, 0);
// str 的长度
v25 = (*a1)->GetStringUTFLength(a1, v23);
// 高32位读取java参数str2
v27 = (*a1)->GetStringUTFChars(a1, HIDWORD(v42), 0);
// str2 的长度
v28 = (*a1)->GetStringUTFLength(a1, v26);
// base 包含Map键值对数据的结构体指针
v7 = (*a1)->FindClass(a1, "java/util/HashMap");
v8 = (*a1)->GetMethodID(a1, v7, "size", "()I");
v43 = (*a1)->CallIntMethod(a1, (jobject)a3, v8);
base = &v42 - 2 * v43;
// 键值对的size
v43
// 
v49 是一个 57位字节数组


```

进入 md5 函数内部，先 hook 下看看

```
Java.perform(function() {
    var moduleName = "libsignutil.so";
    var targetAddr = Module.findExportByName(moduleName, "SIGNMD5");
    if (targetAddr) {
        console.log("找到SIGNMD5函数: " + targetAddr);
        Interceptor.attach(targetAddr, {
            onEnter: function(args) {
                // 记录参数
                this.a1 = args[0];
                this.a2 = args[1];
                this.a3 = args[2]; // 这是存储MD5结果的缓冲区

                console.log("[SIGNMD5] 调用参数:");
                console.log("  a1: " + hexdump(this.a1, { length: parseInt(this.a2) }));
                console.log("  a2: " + this.a2 + " (数据长度)");
                console.log("  a3: " + this.a3 + " (结果缓冲区)");
            },
            onLeave: function(retval) {
                console.log("  md5: " + hexdump(this.a3, { length: 32}));
            }
        });
    } else {
        console.log("未找到SIGNMD5函数");
    }
});



```

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpx0abI6FhyAwGZ2nvr9V2seianib1yCsUmHoDdO0ia2WcX4s5HCxibJdvmw/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpw7qO4e1AicUbXEqoCxHP9W0z5ybFDo7ibqwErIhibR1tuAVSslvpZpWqg/640?wx_fmt=png&from=appmsg&watermark=1)

参数： "bcb2e93c6b527180099601a2dd8ef8b1" + api + java 传的 str1 + map

标准的 md5

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUp3B7YNX61qTouxqyre0ibqwM21pQHmhdMWnoMsYDdkBtINr2xViavFbYg/640?wx_fmt=png&from=appmsg&watermark=1)

主要看操作 a7 的地方![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUp2rXibRV1RHj9h3WzicYsNXwt3l4f6wlDPdR6Mdeh4B7cwBBT6LPlibQicg/640?wx_fmt=png&from=appmsg&watermark=1)

```
*(_DWORD *)a7 = 808464433 = 0x30303031; // 四字节
j_SIGNMD5(v18, v17, a7 + 4); // 将指针从首地址向后移动 4 字节

// 取刚刚md5生成结果的第一个字节的值
result = *(unsigned __int8 *)(a7 + 4);


*(_BYTE *)(a7 + 4 + (result & 0xF)) = result;


```

下面生成的 md5 首字节进行对应的调整![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpZZt7icn3vX7Z4WVMq32wRib0VJ41eLd0aoHSJlFpB6gzGPZjwfYmQcPQ/640?wx_fmt=png&from=appmsg&watermark=1)

后面就算十六进制编码转换了

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpgjP6bzg0oVtwEcEtWxWSWuXdMvhnP8JEicNJaSicD1OsdhxQ77QNkSUA/640?wx_fmt=png&from=appmsg&watermark=1)

最后的 v49[36] ---> v49[47]

```
String str, String str2, Map<String, byte[]> map, String str3, int i
a5 = i = 0

v33 = a0123456789abcd[(unsigned __int8)a5 >> 4]; = '0'
v34 = a0123456789abcd[a5 & 0xF]; = '0'
v35 = a0123456789abcd[(a5 >> 8) & 0xF]; = '0'
v36 = a0123456789abcd[(unsigned __int16)a5 >> 12]; = '0'


v48
遍历HashMap中的每个键值对：
获取键的UTF-8长度 → v19
获取值的字节数组长度 → v20
累加到 v48
最终 v48= 所有键和值的总数据长度


v37 = a0123456789abcd[(unsigned __int8)v48 >> 4];


```

END
---

> ❝
> 
> 有需要用于爬虫、逆向、抓包的测试机又不想花时间折腾 Root、刷机、环境配置可以来看下下面联系方式，或者 B 站视频评论区见

![](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpggxj55Scd37hAIbyVPnJ1mgCEy9oC3ziaJEveK0qwg5UOwzNBVQGcOg/640?wx_fmt=png&from=appmsg&watermark=1)![](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuzjPcke8SdVlymWx0FAcqUpBbJRechTfBF3V5TFaVllyMljR5eDhjo6LuwLm4YcMA56bErCwUyqvA/640?wx_fmt=jpeg&from=appmsg&watermark=1)