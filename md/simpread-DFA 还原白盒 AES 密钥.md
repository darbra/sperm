> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7L19AIVeH9hJLHnh1xQoCw)

本期内容是关于某 app 模拟登录的, 涉及的知识点比较多, 有 unidbg 补环境及辅助还原算法, ida 中的 md5 以及白盒 aes,fart 脱壳, frida 反调试

本章所有样本及资料均上传到了 123 云盘

首先抓包
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk66pEQ5FkHIOyzJRTBIHKCvJL4S7icTgPiagtPLZdFZ2aXRbxmDbzlG4g/640?wx_fmt=png&from=appmsg)

看 login 请求, 表单和响应都是大长串, 猜测是对称加密算法或者是非对称, 对称常见的有 des 和 aes, 非对称常见的有 rsa.

fart 脱壳
-------

正常流程下应该是拖到 jadx 中反编译一下, 但是目标 app 使用了梆梆企业加固

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkQWP3GpNhcN3aUicTfOOVicvnI62updbDicOx1fGWicKDPO1Ux0jro2qmCA/640?wx_fmt=png&from=appmsg)

我换了一部由 fart 脱壳机定制的 pixel 4 后成功脱壳, 后续我会把脱壳的 dex 放到网盘里, 所以对脱壳不了解的可以略过脱壳这个步骤

寒冰的 fart 脱壳机 github 地址:

把脱下来的 dex 文件 pull 到电脑上

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkMqGHEiaInRicB6Da5mANojxo4uyrNn3S7pglniaiawuAxGxQiaib59gdJWog/640?wx_fmt=png&from=appmsg)

对比下脱壳前后的反编译结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkib1lyDf7amJJTaTYAJicbwiaMibzGBxbljuVyiaDSFEu2R37IMt2cnwPhEA/640?wx_fmt=png&from=appmsg)

加密位置定位
------

接下来是定位加密位置了

尝试搜索 "sd"

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkwxVpT9jCwqEVxd3Zt2JZXgMicw8tvic4kMPtiaIQH85FDxctqTaraXq2Q/640?wx_fmt=png&from=appmsg)

框中的可能性比较大, 其他几个类名都是 android aliyun google tencent 这种系统文件或者第三方厂商的, 框中的包含类名以及 retrofit 框架

这个是目标字段的可能性很大, 点进去看看, 然后查找用例

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkbq7LeZEqEkjwOo5XxGrRbzVg78No5WQkYdLmUx27GXYfkzacRPAXQA/640?wx_fmt=png&from=appmsg)

右下角框中的有一个 decrypt 函数, 应该是响应的解密逻辑, 那上面的应该是加密函数了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkmgXt40QAtrbcuxAb7jicSLicoamvMpmxTfbCjMaJVNIsP0SszGMb0icmA/640?wx_fmt=png&from=appmsg)

点进去然后复制 frida 片段

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p></td><td width="678"><p><code>function</code>&nbsp;<code>call(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>Java.perform(</code><code>function</code>&nbsp;<code>(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>let CheckCodeUtils = Java.use(</code><code>"com.cloudy.linglingbang.model.request.retrofit2.CheckCodeUtils"</code><code>);</code></p><p><code>CheckCodeUtils[</code><code>"encrypt"</code><code>].implementation =&nbsp;</code><code>function</code>&nbsp;<code>(str, i) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>console.log(`CheckCodeUtils.encrypt is called: str=${str}, i=${i}`);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>let result =&nbsp;</code><code>this</code><code>[</code><code>"encrypt"</code><code>](str, i);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>console.log(`CheckCodeUtils.encrypt result=${result}`);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>result;</code></p><p><code>};</code></p><p><code>})</code></p><p><code>}</code></p></td></tr></tbody></table>

frida 反调试
---------

frida 注入 frida -UF -l hook.js

以 attach 方式启动 frida 后报错无法附加进程, 这里我们使用 spwan 方式启动即可

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkwt2n6OBng7Cj2ebKqt3wPnwbsFBUibLwqwgJUiaChxHo2BqnPllTmgSA/640?wx_fmt=png&from=appmsg)

换成 spwan 方式后还是报错了, 应该还有检测 frida-server

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkeGh6p6Eaou5xnzou1Lrj5EUbXzvtbSjz6YXAQ5CMYwU0WVJoWf0n6A/640?wx_fmt=png&from=appmsg)

换成葫芦娃形式的试试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkB6kHdVkia8iawSAL3YxiblMhtCNTQJSrG4UKK3giaqAUAH4MHnNI9FCJ3A/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk5nMWNds1sxuJQFaGBeY0r1FoxqtzrsmtUQyQfiaVTpCqDZDmtDlof1g/640?wx_fmt=png&from=appmsg)

成功了, 接下来就是发个包看看有没有结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkCGK5cKfDurt7spyj0dgoEqhWOFFwjibN2l6veSsOSwiajRRl37bKoic3A/640?wx_fmt=png&from=appmsg)

对比下发现结果差不多就是 hook 的结果把 + 改成空格就是 sd 的值了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkNuFEQRjUoGKtaSibeJbP7x6AXzYWgdV8aDLAPviafRfQbD0PVC37WXbA/640?wx_fmt=png&from=appmsg)

接着分析 jadx 中的函数, checkcode 点进去

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk4HiczcUw96TPedjhZQ6LQZ785lGubbxia9AXDYRviaFRP5iaMM9oFSOU9A/640?wx_fmt=png&from=appmsg)

可以看到目标函数返回 null, 和 hook 的结果不一样, 并且 jadx 给出了警告, 不知道是脱壳脱的不全还是 jadx 的问题, 后续可以用 jeb 试试, jeb 的反编译能力比 jadx 强

同时可以看到下面有两个 native 函数, checkcode, 和 decheckcode, 尝试 hook checkcode 函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk0WvzYF5KQleRSl38hcibg8Sc2e2hWAREm0oX9FwmwE4PPjnMaPXliaFQ/640?wx_fmt=png&from=appmsg)

同样有结果

这两个 native 函数加载自 libencrypt.so

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk50c5ibEMTPvUVBiajxA0OUJDyT6OXK6WZBib7sqzgdsstpr26v3wiaDZqw/640?wx_fmt=png&from=appmsg)

这里我选择 32 位的 so, 拖到 ida32 中搜索 java, 发现是静态注册 (如果是动态注册还可以 hook libart.so 来找导出函数)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkZvUlhL2ccvQfq7CnHmazWibiclNvoUmwIicga8JM8IIY2Wq1ULweXQTqQ/640?wx_fmt=png&from=appmsg)

unidbg 搭架子
----------

接下来是 unidbg 模拟执行

搭架子

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p><p>48</p><p>49</p><p>50</p><p>51</p><p>52</p><p>53</p><p>54</p><p>55</p><p>56</p><p>57</p><p>58</p><p>59</p><p>60</p><p>61</p><p>62</p><p>63</p><p>64</p><p>65</p></td><td width="678"><p><code>package</code>&nbsp;<code>com;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.AndroidEmulator;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.Module;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidEmulatorBuilder;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidResolver;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.AbstractJni;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.DalvikModule;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.StringObject;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.VM;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.memory.Memory;</code></p><p><code>import</code>&nbsp;<code>java.io.File;</code></p><p><code>import</code>&nbsp;<code>java.util.ArrayList;</code></p><p><code>import</code>&nbsp;<code>java.util.List;</code></p><p><code>public</code>&nbsp;<code>class</code>&nbsp;<code>demo2&nbsp;</code><code>extends</code>&nbsp;<code>AbstractJni {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>AndroidEmulator emulator;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>VM vm;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Module module;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Memory memory;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(</code><code>"com.cloudy.linglingbang"</code><code>).build();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 获取模拟器的内存操作接口</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory = emulator.getMemory();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置系统类库解析</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory.setLibraryResolver(</code><code>new</code>&nbsp;<code>AndroidResolver(</code><code>23</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm = emulator.createDalvikVM(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/llb.apk"</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置JNI</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setJni(</code><code>this</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 打印日志</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setVerbose(</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 加载目标SO</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>DalvikModule dm = vm.loadLibrary(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/libencrypt.so"</code><code>),&nbsp;</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>//获取本SO模块的句柄,后续需要用它</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>module = dm.getModule();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 调用JNI OnLoad</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>dm.callJNI_OnLoad(emulator);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>String callByAddress(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// args list</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>List&lt;Object&gt; list =&nbsp;</code><code>new</code>&nbsp;<code>ArrayList&lt;&gt;(</code><code>5</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jnienv</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.getJNIEnv());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jclazz</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>0</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str1</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"mobile=13535535353&amp;password=fjfjfjffk&amp;client_id=2019041810222516127&amp;client_secret=c5ad2a4290faa3df39683865c2e10310&amp;state=eu4acofTmb&amp;response_type=token"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// int</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>2</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str2</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"1709100421650"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>Number number = module.callFunction(emulator,&nbsp;</code><code>0x13A19</code><code>, list.toArray());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String result = vm.getObject(number.intValue()).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"======encrypt:"</code><code>+result);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>result;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>static</code>&nbsp;<code>void</code>&nbsp;<code>main(String[] args) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2 llb =&nbsp;</code><code>new</code>&nbsp;<code>demo2();</code></p><p><code>//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; llb.callByAddress();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>}</code></p></td></tr></tbody></table>

补环境
---

运行报错,`currentActivityThread` 通常用于一些需要获取全局上下文或执行一些与应用程序状态相关的操作的场景

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkttPicNuBDI3jR9JyI8OXXoAr4pEKqO5sXVmTafSl5cL7lic04tLUyAibA/640?wx_fmt=png&from=appmsg)

补上, 这里没什么好说的, 孰能生巧

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></td><td width="685"><p><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ActivityThread-&gt;currentActivityThread()Landroid/app/ActivityThread;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/app/ActivityThread"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.callStaticObjectMethod(vm, dvmClass, signature, varArg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p></td></tr></tbody></table>

接着运行, SystemProperties 中的 get 像是在获取系统的某个属性

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkYQUBDWbuSnOkjUDRXZxjzpiaicVuD5mFxcGV9LfHyvNjSTbSKaez6Bmw/640?wx_fmt=png&from=appmsg)

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p></td><td width="685"><p><code>case</code>&nbsp;<code>"android/os/SystemProperties-&gt;get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"SystemProperties get arg:"</code><code>+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk0LTmFpU4d0rNFdAYXXUvPaULrFOX8xJxaWibCP3AFW7ybD7RfYsTQGg/640?wx_fmt=png&from=appmsg)

获取手机序列号的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkTg9veowJIfnHejLrAyic74tN4ibmae7I9iaIxPX2aIibWh0g2qBkqq6Cew/640?wx_fmt=png&from=appmsg)

adb shell getprop ro.serialno

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkfKLD9FwIPMsk5M7uMS7UdFtTycpaDJRdMIPyEZQ6SicCWJnfglNRHYg/640?wx_fmt=png&from=appmsg)

完整的补上

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p></td><td width="685"><p><code>case</code>&nbsp;<code>"android/os/SystemProperties-&gt;get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"SystemProperties get arg:"</code><code>+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code><code>(arg.equals(</code><code>"ro.serialno"</code><code>)){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"9B131FFBA001Y5"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p></td></tr></tbody></table>

后面的环境不说了, 大概也是这样的流程, 遇到不会的就 google 一下或者问问 ai, 我这里就直接贴一下代码了

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p><p>48</p><p>49</p><p>50</p><p>51</p><p>52</p><p>53</p><p>54</p><p>55</p><p>56</p><p>57</p><p>58</p><p>59</p><p>60</p><p>61</p><p>62</p><p>63</p><p>64</p><p>65</p><p>66</p><p>67</p><p>68</p><p>69</p><p>70</p><p>71</p><p>72</p><p>73</p><p>74</p><p>75</p><p>76</p><p>77</p><p>78</p><p>79</p><p>80</p><p>81</p><p>82</p><p>83</p><p>84</p><p>85</p><p>86</p><p>87</p><p>88</p><p>89</p><p>90</p><p>91</p><p>92</p><p>93</p><p>94</p><p>95</p><p>96</p><p>97</p><p>98</p><p>99</p><p>100</p><p>101</p><p>102</p><p>103</p><p>104</p><p>105</p><p>106</p><p>107</p><p>108</p><p>109</p><p>110</p><p>111</p><p>112</p><p>113</p><p>114</p><p>115</p><p>116</p><p>117</p><p>118</p></td><td width="671"><p><code>package</code>&nbsp;<code>com;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.AndroidEmulator;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.Module;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidEmulatorBuilder;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidResolver;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.*;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.memory.Memory;</code></p><p><code>import</code>&nbsp;<code>java.io.File;</code></p><p><code>import</code>&nbsp;<code>java.util.ArrayList;</code></p><p><code>import</code>&nbsp;<code>java.util.List;</code></p><p><code>public</code>&nbsp;<code>class</code>&nbsp;<code>demo2&nbsp;</code><code>extends</code>&nbsp;<code>AbstractJni {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>AndroidEmulator emulator;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>VM vm;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Module module;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Memory memory;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(</code><code>"com.cloudy.linglingbang"</code><code>).build();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 获取模拟器的内存操作接口</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory = emulator.getMemory();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置系统类库解析</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory.setLibraryResolver(</code><code>new</code>&nbsp;<code>AndroidResolver(</code><code>23</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm = emulator.createDalvikVM(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/llb.apk"</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置JNI</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setJni(</code><code>this</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 打印日志</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setVerbose(</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 加载目标SO</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>DalvikModule dm = vm.loadLibrary(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/libencrypt.so"</code><code>),&nbsp;</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>//获取本SO模块的句柄,后续需要用它</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>module = dm.getModule();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 调用JNI OnLoad</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>dm.callJNI_OnLoad(emulator);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>String callByAddress(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// args list</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>List&lt;Object&gt; list =&nbsp;</code><code>new</code>&nbsp;<code>ArrayList&lt;&gt;(</code><code>5</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jnienv</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.getJNIEnv());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jclazz</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>0</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str1</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"mobile=13535535353&amp;password=fjfjfjffk&amp;client_id=2019041810222516127&amp;client_secret=c5ad2a4290faa3df39683865c2e10310&amp;state=eu4acofTmb&amp;response_type=token"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// int</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>2</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str2</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"1709100421650"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>Number number = module.callFunction(emulator,&nbsp;</code><code>0x13A19</code><code>, list.toArray());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String result = vm.getObject(number.intValue()).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"======encrypt:"</code><code>+result);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>result;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>static</code>&nbsp;<code>void</code>&nbsp;<code>main(String[] args) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2 llb =&nbsp;</code><code>new</code>&nbsp;<code>demo2();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>llb.callByAddress();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ActivityThread-&gt;currentActivityThread()Landroid/app/ActivityThread;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/app/ActivityThread"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/SystemProperties-&gt;get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"SystemProperties get arg:"</code><code>+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code><code>(arg.equals(</code><code>"ro.serialno"</code><code>)){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"9B131FFBA001Y5"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.callStaticObjectMethod(vm, dvmClass, signature, varArg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; callObjectMethod(BaseVM vm, DvmObject&lt;?&gt; dvmObject, String signature, VarArg varArg) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ActivityThread-&gt;getSystemContext()Landroid/app/ContextImpl;"</code><code>:{</code></p><p><code>//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println("22222");</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/app/ContextImpl"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ContextImpl-&gt;getPackageManager()Landroid/content/pm/PackageManager;"</code><code>: {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/content/pm/PackageManager"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ContextImpl-&gt;getSystemService(Ljava/lang/String;)Ljava/lang/Object;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println("getSystemService arg:"+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android.net.wifi"</code><code>).newObject(signature);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/net/wifi-&gt;getConnectionInfo()Landroid/net/wifi/WifiInfo;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/net/wifi/WifiInfo"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/net/wifi/WifiInfo-&gt;getMacAddress()Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"02:00:00:00:00:00"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.callObjectMethod(vm, dvmObject, signature, varArg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build-&gt;MODEL:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"Pixel 4 XL"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build-&gt;MANUFACTURER:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"Google"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build$VERSION-&gt;SDK:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"29"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.getStaticObjectField(vm, dvmClass, signature);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>}</code></p></td></tr></tbody></table>

再次运行下, 出结果了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkmicEWeyTm81rUBhAnxeJpJntx3D7kF2jdVMZGThRFeWos89UOOVBRHQ/640?wx_fmt=png&from=appmsg)

但是怎么验证结果是否正确呢, 我这里想着是把结果拿去解密看看, 代码如下

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p><p>48</p><p>49</p><p>50</p><p>51</p><p>52</p><p>53</p><p>54</p><p>55</p><p>56</p><p>57</p><p>58</p><p>59</p><p>60</p><p>61</p><p>62</p><p>63</p><p>64</p><p>65</p><p>66</p><p>67</p><p>68</p><p>69</p><p>70</p><p>71</p><p>72</p><p>73</p><p>74</p><p>75</p><p>76</p><p>77</p><p>78</p><p>79</p><p>80</p><p>81</p><p>82</p><p>83</p><p>84</p><p>85</p><p>86</p><p>87</p><p>88</p><p>89</p><p>90</p><p>91</p><p>92</p><p>93</p><p>94</p><p>95</p><p>96</p><p>97</p><p>98</p><p>99</p><p>100</p><p>101</p><p>102</p><p>103</p><p>104</p><p>105</p><p>106</p><p>107</p><p>108</p><p>109</p><p>110</p><p>111</p><p>112</p><p>113</p><p>114</p><p>115</p><p>116</p><p>117</p><p>118</p><p>119</p><p>120</p><p>121</p><p>122</p><p>123</p><p>124</p><p>125</p><p>126</p><p>127</p><p>128</p><p>129</p><p>130</p><p>131</p><p>132</p><p>133</p></td><td width="671"><p><code>package</code>&nbsp;<code>com;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.AndroidEmulator;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.Module;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidEmulatorBuilder;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.AndroidResolver;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.linux.android.dvm.*;</code></p><p><code>import</code>&nbsp;<code>com.github.unidbg.memory.Memory;</code></p><p><code>import</code>&nbsp;<code>java.io.File;</code></p><p><code>import</code>&nbsp;<code>java.util.ArrayList;</code></p><p><code>import</code>&nbsp;<code>java.util.List;</code></p><p><code>public</code>&nbsp;<code>class</code>&nbsp;<code>demo2&nbsp;</code><code>extends</code>&nbsp;<code>AbstractJni {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>AndroidEmulator emulator;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>VM vm;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Module module;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>private</code>&nbsp;<code>final</code>&nbsp;<code>Memory memory;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建模拟器实例,进程名建议依照实际进程名填写，可以规避针对进程名的校验</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(</code><code>"com.cloudy.linglingbang"</code><code>).build();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 获取模拟器的内存操作接口</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory = emulator.getMemory();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置系统类库解析</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>memory.setLibraryResolver(</code><code>new</code>&nbsp;<code>AndroidResolver(</code><code>23</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm = emulator.createDalvikVM(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/llb.apk"</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 设置JNI</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setJni(</code><code>this</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 打印日志</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>vm.setVerbose(</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 加载目标SO</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>DalvikModule dm = vm.loadLibrary(</code><code>new</code>&nbsp;<code>File(</code><code>"unidbg-android/apks/llb/libencrypt.so"</code><code>),&nbsp;</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>//获取本SO模块的句柄,后续需要用它</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>module = dm.getModule();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 调用JNI OnLoad</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>dm.callJNI_OnLoad(emulator);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>String callByAddress(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// args list</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>List&lt;Object&gt; list =&nbsp;</code><code>new</code>&nbsp;<code>ArrayList&lt;&gt;(</code><code>5</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jnienv</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.getJNIEnv());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jclazz</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>0</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str1</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"mobile=13535535353&amp;password=fjfjfjffk&amp;client_id=2019041810222516127&amp;client_secret=c5ad2a4290faa3df39683865c2e10310&amp;state=eu4acofTmb&amp;response_type=token"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// int</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>2</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str2</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"1709100421650"</code><code>)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>Number number = module.callFunction(emulator,&nbsp;</code><code>0x13A19</code><code>, list.toArray());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String result = vm.getObject(number.intValue()).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"======encrypt:"</code><code>+result);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>result;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>};</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>static</code>&nbsp;<code>void</code>&nbsp;<code>main(String[] args) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>demo2 llb =&nbsp;</code><code>new</code>&nbsp;<code>demo2();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>llb.callByAddress();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>llb.decrtpy(</code><code>"Mhub8kSp2n38SHF4COj57zjesFrzCIB2JiH76iCwZZffL3Y4+1/fq1uEDKKWe4yAwiacSVxXNSq1sWN5TwtfHaVgxpOREVGT2+qZEZFkvjP1GaxPCPP2jwuy4x3GvPgHl2NhG2kpsfcXHHQK9HJ5iBdtO44QdDO0vtgqU9MGGb+3q+HJwKlgfWJZj24t8HOSypJNigdCXbUEC6HGEhZhAhMX+Za1lffLlxUouhVh8rzKyESEF97li1h1vTbEf6TJyMbbdEpxh355FbxV9wZgorCa93rDfu+bsVLDbQaAF1TcacxnokoS/yv92hYaqzwzSX3UdH5oQutjW6A4gH1Zk/1Yb3k+IHofvc6Lfm+cxrLHLDtsus9SM/4+2oqsE7tsbgUny37/PQXtUJEOwebDtpz5oYxPgEIbLKIHvptVKwh4="</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ActivityThread-&gt;currentActivityThread()Landroid/app/ActivityThread;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/app/ActivityThread"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/SystemProperties-&gt;get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"SystemProperties get arg:"</code><code>+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code><code>(arg.equals(</code><code>"ro.serialno"</code><code>)){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"9B131FFBA001Y5"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.callStaticObjectMethod(vm, dvmClass, signature, varArg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; callObjectMethod(BaseVM vm, DvmObject&lt;?&gt; dvmObject, String signature, VarArg varArg) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ActivityThread-&gt;getSystemContext()Landroid/app/ContextImpl;"</code><code>:{</code></p><p><code>//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println("22222");</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/app/ContextImpl"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ContextImpl-&gt;getPackageManager()Landroid/content/pm/PackageManager;"</code><code>: {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/content/pm/PackageManager"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/app/ContextImpl-&gt;getSystemService(Ljava/lang/String;)Ljava/lang/Object;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String arg = varArg.getObjectArg(</code><code>0</code><code>).getValue().toString();</code></p><p><code>//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println("getSystemService arg:"+arg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android.net.wifi"</code><code>).newObject(signature);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/net/wifi-&gt;getConnectionInfo()Landroid/net/wifi/WifiInfo;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>vm.resolveClass(</code><code>"android/net/wifi/WifiInfo"</code><code>).newObject(</code><code>null</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/net/wifi/WifiInfo-&gt;getMacAddress()Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"02:00:00:00:00:00"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.callObjectMethod(vm, dvmObject, signature, varArg);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>DvmObject&lt;?&gt; getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>switch</code>&nbsp;<code>(signature){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build-&gt;MODEL:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"Pixel 4 XL"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build-&gt;MANUFACTURER:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"Google"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>case</code>&nbsp;<code>"android/os/Build$VERSION-&gt;SDK:Ljava/lang/String;"</code><code>:{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>new</code>&nbsp;<code>StringObject(vm,&nbsp;</code><code>"29"</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>super</code><code>.getStaticObjectField(vm, dvmClass, signature);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>void</code>&nbsp;<code>decrtpy(String str){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// args list</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>List&lt;Object&gt; list =&nbsp;</code><code>new</code>&nbsp;<code>ArrayList&lt;&gt;(</code><code>5</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jnienv</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.getJNIEnv());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// jclazz</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(</code><code>0</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// str</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>list.add(vm.addLocalObject(</code><code>new</code>&nbsp;<code>StringObject(vm, str)));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// int</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>Number number = module.callFunction(emulator,&nbsp;</code><code>0x165E1</code><code>, list.toArray());</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String result = vm.getObject(number.intValue()).getValue().toString();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"======decrypt:"</code><code>+result);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>}</code></p></td></tr></tbody></table>

运行结果如下, 好像不太正常

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkf95tuIKa5iaUySvyvmRjFZwNDUZq08j9AicdNz7YjQ2rrS1ia13UXIsiaA/640?wx_fmt=png&from=appmsg)

从 ida 中的 decheckcode 点进去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkLCLGBjG3E1ic2LnsaOCDLK6Yic3OcZjSHa5OtzqC25BBiaolaPcsj5ZIw/640?wx_fmt=png&from=appmsg)

放回的结果异常, 说明走了异常的分支, 看样子像是返回的是 26 行的值, 判断! v6 的值是否为真, v6 来自上面的 sub_138AC 函数, 点进去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk10KIED8buzKNNH5cgAWphvY9RN1UcRxdibxavpNc2tt9O6oSzic7jCkQ/640?wx_fmt=png&from=appmsg)

中间的 sub_ED04 是一个很大的函数, 看这像是检测某种环境

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkbctx5krqJoMuWhOvfAQ08QDXDNl6bdysWkCaX1NEdibuvOdHuL4I6YQ/640?wx_fmt=png&from=appmsg)

往下滑可以看到像是 md5 的 64 轮运算, 和结尾解密得到的 32 位数据对应上了, 所以说程序大概率是走了这个分支后直接返回了数据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkQJdPYI3Xaj5KxvA4rgybGS1U9BM65QGaTQWg8uFibVroRh1UiasRwdPQ/640?wx_fmt=png&from=appmsg)

如果是这样的话就好办了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkZuMqS8rZnKhEIzSuzvMsQ6MWxHSvXHcsktjGAjeiaZgTxmicBtMYyicZw/640?wx_fmt=png&from=appmsg)

直接在 v6 的地方取反就好了, 看下此次的汇编代码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkL98ajm1kqB31iaXpjHfNcCHty1EFJ9Sm9IQmLXmH64GEj8uliaO794Dw/640?wx_fmt=png&from=appmsg)

是一个条件跳转, CBNZ 意思是如果 r0 寄存器的值不为 0 就跳到 loc_16610 处, 取反的指令就是 CBZ(少了个 N not), 为 0 就跳

拿到 hex 转 arm 网站上看看指令, 20 B9 对应的是 cbnz r0, #0xc

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkef1NtpZibibHY1CAGh95IAmQRBZjwqqy1LN6iaF7MhssTkqrpyXj2E6HQ/640?wx_fmt=png&from=appmsg)

所以我们需要的就是 cbz r0, #0xc

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkkFWmCQibbh8oEZVYU7ZjXz9p8yfbFYAUCxmKlB88AWEnhtMKBibZK1fQ/640?wx_fmt=png&from=appmsg)

把 20 B9 改成 20 B1 就可以了, 比较原始的方式就是用 ida 或者 010editor 改, unidbg 也提供了 patct 的方式直接在程序执行前改机器码

ida 和 010editor 改的方式就不说了, 网上有教程, unidbg 中这样改

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td width="685"><p><code>public</code>&nbsp;<code>void</code>&nbsp;<code>patch(){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>UnidbgPointer pointer = UnidbgPointer.pointer(emulator,module.base +&nbsp;</code><code>0x16604</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>byte</code><code>[] code =&nbsp;</code><code>new</code>&nbsp;<code>byte</code><code>[]{(</code><code>byte</code><code>)&nbsp;</code><code>0x20</code><code>, (</code><code>byte</code><code>)&nbsp;</code><code>0xB1</code><code>};</code><code>//直接用硬编码改原so的代码：&nbsp; 4FF00109</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>pointer.write(code);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p></td></tr></tbody></table>

在调用 callByAddress 函数之前调用 patch 就可以了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkMvFFiaTbR8lKeeuJEK1q0dJBAl6iayXvicMM1pibYhAcGpP8tLLUv63xIA/640?wx_fmt=png&from=appmsg)

解密结果也是出来了, 可以看到有手机号, 密码还有一些设备信息

还原算法
----

接下来就是 unidbg 辅助还原算法了

前面在加密函数的位置看到了 aes 字眼, 所有猜测使用了 aes 加密

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkLEFUEFe7GBoCtV7q9AeKAxtlFy1YrG2Nr0crlnYaiaucZq5tSEVVqMw/640?wx_fmt=png&from=appmsg)

还原 aes 加密需要确认密钥 加密模式 (ecb cbc 等等) 是否有 iv, 填充方式, 接下来就是漫长的猜测验证再猜测的过程了, 利用 unidbg 可以 console debugger 的优点, 可以非常方便的还原算法

由于加密函数快 3000 多行, 我这里就说大概得关键位置了, 如果写的太细内容就太多了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkz8ofLU2cxTqmEF2Q3ibnWWvTKrFQaZLmerKNP1R00m0oqV4HbmSicibHA/640?wx_fmt=png&from=appmsg)

结合着 ida 静态分析和 unidbg 动态调试可以猜测 2884 行应该是进行 aes 加密的, 并且后续进行了 base64 编码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkk1xcS2NQncUhdMQHHIxDd97mdlNe15GVPNL5SeReYaTh6rhlkUVTbg/640?wx_fmt=png&from=appmsg)

点进去发现来到了. bss 段, .bss 段是用来存放程序中未初始化的全局变量的一块内存区域

看下此次的汇编代码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk26JnNI8ZyTVDUiavfoS9rEILYKBicwCRynv6uy4aSfyiaPWS0w9Z6UAnA/640?wx_fmt=png&from=appmsg)

BLX R3 意思是跳转到寄存器 `R3` 中存储的地址处执行, 所以在 unidbg 中 0x163FE 下断, 看看 R3 寄存器的地址

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p></td><td width="685"><p><code>debugger.addBreakPoint(module.base+</code><code>0x163FE</code><code>);</code></p></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkPxzzM0IOAR7LzoibPFjnSqdp3e2UCslG3TTzBticIOQfdvfQXt9STRSQ/640?wx_fmt=png&from=appmsg)

断在 0x163FE 处了, 前面的 0x400 是加上了 unidbg 的基地址, 可以看到 R3 的地址减去基地址也就是后面的地址是 0x5a35, 再减去 thumb 的地址加 1 也就是 0x5a34

ida 中按 G 跳转到 0x5a34

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkDyCia0ciaWekYhFpbticQfibJSibYs0ggO0m6eJompb69VyBcy0OfBz3ibvg/640?wx_fmt=png&from=appmsg)

可以看到 aes 的具体逻辑就在这里面的几个函数中, 最后的 WBACRAES128_EncryptCBC 貌似是在说 white box aes128 cbc 模式

如果是这样的话, 由于白盒 aes11 个轮秘钥嵌在程序里, 很难直接提取出, 需要用 dfa(差分故障攻击) 获取到第 10 轮的秘钥, 再利用 aes_keyschedule 这个模块还原出主密钥

WBACRAES128_EncryptCBC 点进去

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkGiaZHpicxER6L92kHAfNMicoMB5cHOsQnqXWbFibuU4w1z203977KNkOnw/640?wx_fmt=png&from=appmsg)

可以看到首先对明文进行了填充, 往下滑

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkhP0HsTiadTCaaCrwdzExE9naVlVUYxPXmiavJYBjESr5VvLiajv6dAAcQ/640?wx_fmt=png&from=appmsg)

WBACRAES_EncryptOneBlock 视乎是运算的主体, 点进去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkicflr6pO8iapjnyewXsD2ick73fEMJPa9D44RgcNpVCHicibwwA5r9hTs4Q/640?wx_fmt=png&from=appmsg)

这里因为我每个地址都下断看了下参数值, 实际操作过程需要一步步验证才能走到这

再点进去

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkMNCibwoPCExH0RianyGc4DFXQDXzAI3SGngTMicGXUZPibzhbUGjqy6W5A/640?wx_fmt=png&from=appmsg)

这里 ida f5 出来的看不太懂, 看看汇编视图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkayJBZTMMpSeSwuADiaYmhyUzxckR2plvnOIziaVIAVmiaktbMgx1qbGiaA/640?wx_fmt=png&from=appmsg)

可以看到结尾跳转到 R4 寄存器指向的地址, unidbg 中下断看下

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p></td><td width="685"><p><code>debugger.addBreakPoint(module.base+</code><code>0x5836</code><code>);</code></p></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkaEiaxXk0AeeB2Xw8ic3sOtEGCXyGNYZeL6qEcQTq2nCgiayXaDXoOIL5g/640?wx_fmt=png&from=appmsg)

所以最终会跳到 0x4dcc 位置处, 为什么要 - 1 上面也说过了, 跳到 0x4dcc 去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk1lwiaGU4RGFVCiasO5iaJcic6OPD5l0xwk2qPPibSTLjH1wxyhYWP5o1Q2w/640?wx_fmt=png&from=appmsg)

这里会判断 i=9 的时候跳出循环, PrepareAESMatrix 中 Matrix 是矩阵的意思, 所有这个函数应该是对 state 数据进行矩阵运算

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkl7DCvtjqKKlgWACeDeDxGvlD505okJMGRqKdbpFD9Pj4AryCkntrsQ/640?wx_fmt=png&from=appmsg)

aes 的 1-9 轮和第 10 轮不一样, 第十轮少了一个列混淆运算

为了方便分析秘钥, 我让 unidbg 在 aes 输入明文的地方修改寄存器的值, 这样加密的结果就是 16 字节的, 如果直接修改 unidbg 的入参的话, 由于后续会拼上环境参数二导致参数太长

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p></td><td width="678"><p><code>debugger.addBreakPoint(module.base+</code><code>0x5A34</code><code>,&nbsp;</code><code>new</code>&nbsp;<code>BreakPointCallback() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>boolean</code>&nbsp;<code>onHit(Emulator&lt;?&gt; emulator,&nbsp;</code><code>long</code>&nbsp;<code>address) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String fakeInput =&nbsp;</code><code>"hello"</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>int</code>&nbsp;<code>length = fakeInput.length();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>MemoryBlock fakeInputBlock = emulator.getMemory().malloc(length,&nbsp;</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>fakeInputBlock.getPointer().write(fakeInput.getBytes(StandardCharsets.UTF_8));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 修改r0为指向新字符串的新指针</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, fakeInputBlock.getPointer().peer);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>true</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>});</code></p></td></tr></tbody></table>

接下来在 aes 加密结束后的结果是多少

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p></td><td width="678"><p><code>debugger.addBreakPoint(module.base+</code><code>0x4DCC</code><code>,&nbsp;</code><code>new</code>&nbsp;<code>BreakPointCallback() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>RegisterContext context = emulator.getContext();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>boolean</code>&nbsp;<code>onHit(Emulator&lt;?&gt; emulator,&nbsp;</code><code>long</code>&nbsp;<code>address) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator.attach().addBreakPoint(context.getLRPointer().peer,&nbsp;</code><code>new</code>&nbsp;<code>BreakPointCallback() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>//onleave</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>boolean</code>&nbsp;<code>onHit(Emulator&lt;?&gt; emulator,&nbsp;</code><code>long</code>&nbsp;<code>address) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>false</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>});</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>true</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>});</code></p></td></tr></tbody></table>

由于这个 WBACRAES_EncryptOneBlock 函数结束的时候寄存器中的地址已经不是原先的用来存返回值的地址了, 所有需要提前 hook 看一下入参时目标参数的地址, 代函数执行完直接打印这个地址就是结果了, 这里是 0xbffff50c m0xbffff50c 可以直接看内存中的值

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkWdCj0dnR94vpPT0bLKBNgIS4mgxnxPRAPiaNXZxlXEKtCfaPmeIMeDQ/640?wx_fmt=png&from=appmsg)

所以正确的密文是 57b0d60b1873ad7de3aa2f5c1e4b3ff6

接下来进行 dfa 攻击 (差分故障攻击), 这里需要熟悉 aes 算法的细节, 我这里就不介绍了, 感兴趣的去龙哥的知识星球学习一下

故障注入的时机是倒数两次列混淆之间, 也就是第八轮以及第九轮运算中两次列混淆之间的时机

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkaju9DZXkjyzNXDibftz0CEolM9Ziag2Fc8hND97ic5LN5NN6F39yF3WYQ/640?wx_fmt=png&from=appmsg)

这里的 s 应该就是 state 块

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p></td><td width="678"><p><code>debugger.addBreakPoint(module.base+</code><code>0x4E2A</code><code>,&nbsp;</code><code>new</code>&nbsp;<code>BreakPointCallback() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>int</code>&nbsp;<code>round =&nbsp;</code><code>0</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>UnidbgPointer statePointer = memory.pointer(</code><code>0xbffff458</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>boolean</code>&nbsp;<code>onHit(Emulator&lt;?&gt; emulator,&nbsp;</code><code>long</code>&nbsp;<code>address) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>round +=&nbsp;</code><code>1</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>System.out.println(</code><code>"round:"</code><code>+round);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code>&nbsp;<code>(round %&nbsp;</code><code>9</code>&nbsp;<code>==&nbsp;</code><code>0</code><code>){</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>statePointer.setByte(randInt(</code><code>0</code><code>,&nbsp;</code><code>15</code><code>), (</code><code>byte</code><code>) randInt(</code><code>0</code><code>,&nbsp;</code><code>0xff</code><code>));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>true</code><code>;</code><code>//返回true 就不会在控制台断住</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>});</code></p></td></tr></tbody></table>

DFA 还原白盒 AES 密钥
---------------

接下来就是取多次故障密文了

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p></td><td width="678"><p><code>import</code>&nbsp;<code>phoenixAES</code></p><p><code>with&nbsp;</code><code>open</code><code>(</code><code>'tracefile'</code><code>,&nbsp;</code><code>'wb'</code><code>) as t:&nbsp;&nbsp;</code><code># 第一行是正确密文 后面是故障密文</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>t.write(</code><code>"""57b0d60b1873ad7de3aa2f5c1e4b3ff6</code></p><p><code>57b0d6a41873737de3892f5c2a4b3ff6</code></p><p><code>5720d60baf73ad7de3aa2f9b1e4b02f6</code></p><p><code>57b0f20b18f3ad7daeaa2f5c1e4b3f86</code></p><p><code>8db0d60b1873ad2fe3aa365c1eab3ff6</code></p><p><code>e2b0d60b1873ad5be3aafa5c1e1b3ff6</code></p><p><code>57b04e0b1812ad7d89aa2f5c1e4b3fa7</code></p><p><code>57d1d60b3773ad7de3aa2f8b1e4b2ff6</code></p><p><code>bcb0d60b1873ad21e3aa155c1e3d3ff6</code></p><p><code>57b0bb0b1885ad7d4aaa2f5c1e4b3f29</code></p><p><code>3ab0d60b1873ad67e3aac65c1e193ff6</code></p><p><code>57b0d6531873af7de3302f5c964b3ff6</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"""</code><code>.encode(</code><code>'utf8'</code><code>))</code></p><p><code>phoenixAES.crack_file(</code><code>'tracefile'</code><code>, [],&nbsp;</code><code>True</code><code>,&nbsp;</code><code>False</code><code>,&nbsp;</code><code>3</code><code>)</code></p></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkPDbvX9OZ7ydIqHIsYVKyb4BwkHKldUQruLFLiasZ7tYHxae6Y2edHUA/640?wx_fmt=png&from=appmsg)

拿到结果了

最后用 aes_keyschedule 把主密钥也就是初始秘钥还原出来了, F6F472F595B511EA9237685B35A8F866

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkiaH82umg9icQTySFbchJPom5jQTLicKBgPoO5gQHbzEicXIGTnzp6fYyrQ/640?wx_fmt=png&from=appmsg)

把刚开始的密文拿到 CyberChef 尝试解一下, 因为 cbc 模式需要 iv, 所以先用 ecb 模式, cbc 模式比 ecb 模式多的就是 cbc 模式需要每个明文分组先和上个分组的密文块进行异或, 由于第一组没有上个分组的密文块, 所以需要一个初始化向量 IV

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkicTdthLMlmGEwcpuhy3TxqMuwdlfz0KQibzAZuqXYgHAq3ZEw1LtxTZw/640?wx_fmt=png&from=appmsg)

上面符号 WBACRAES128_EncryptCBC 说的是 cbc 加密模式, 但这个符号不一定可信, 如果使用的是 cbc 模式, 解出来的结果就是明文块和 iv 异或的值 (矩阵异或)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkacy2LhFhrpzN44UBIlvIfDuqPzK6R83ldL3XLcPGlMGhQULKGST27Q/640?wx_fmt=png&from=appmsg)

后面全是 0, 如果是 cbc 模式下, 明文块和 iv 异或了, 由于是矩阵异或, 如果填充方式是 pkcs7, 就意味着 iv 的后面几位是 68656c6c6f 填充后的

68656c6c6f0b0b0b0b0b0b0b0b0b0b0b 后面几位, 也就是 0b0b0b0b0b0b0b0b0b0b0b, 如果这样的话明文一变填充的数据也变了, 可能是 01-0f 中的任何一个, 这样 iv 的值也不固定, 显然在这种情况下就太复杂了.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkoEjJpSXWrdIgApBfhFR1Xudw6wG6ej9YicRmcNZQKvUGcQL8ibAianAGw/640?wx_fmt=png&from=appmsg)

所以我认为应该是 ecb 模式下使用了 Zero Padding 模式, 全部填充 0 直到一个分组长度

为了验证猜想, 在 InsertCBCPadding 函数结束时打印处理过的 state 块, unidbg 中下断

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p></td><td width="685"><p><code>debugger.addBreakPoint(module.base+</code><code>0x58A0</code><code>);&nbsp;</code><code>//m0x40321000</code></p></td></tr></tbody></table>

改变输入后发现后面也还是 0, 也就验证了采用的是 Zero Padding 模式, 并不是常见的 pkcs7 模式

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk4TA1wpKYRmDlXyaMsyibKg7VLnhE5r6KYQxTEYKwTTRrhGuvediaxQeA/640?wx_fmt=png&from=appmsg)

由于 CyberChef 中默认是 pkcs7 填充, 所以把模式调成 nopadding, 这样解密出来的结果就是未填充的一个分组长度了, 也验证了上面的 Zero Padding 模式

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kk2MVKfib8OiajBD2KGdicEQhPyXJAGhvZibhnIiajXjw9AYCLJeiaicJGOITog/640?wx_fmt=png&from=appmsg)

这也就是说上面的 cbc 模式也是错误的, 而是 ecb 模式

尝试加密一下明文和密文对比下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkw2yAZTjum2j0JloAY1nhoVEStfS2rLUA1b0nQvCSoOxQ8nmNFpXoibQ/640?wx_fmt=png&from=appmsg)

正常的密文是 Mhub8kSp2n38SHF4COj57zjesFrzCIB2JiH76iCwZZffL3Y4+1/fq1uEDKKWe4yAwiacSVxXNSq1sWN5TwtfHaVgxpOREVGT2+qZEZFkvjP1GaxPCPP2jwuy4x3GvPgHl2NhG2kpsfcXHHQK9HJ5iBdtO44QdDO0vtgqU9MGGb+3q+HJwKlgfWJZj24t8HOSypJNigdCXbUEC6HGEhZhAhH9QOWkbD6iDkO4mpB0xjvRurFugh+t9P3AeXJeHdhF+MnCXXj3BGlfUgi2qCvoWxYajx2sUcZkXpNbAFbj7VaAlG2ytQnO/L0aZr+SlzTxb90PoLU2VBp98GXNt0ozObSaCwO41UlmZPcKZrr9sxf32nwmoEmUwoTXe14aks2nj72zo5kz8GXyfzh2f6mddZQ==

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkick3aSASjR4dCHNibXccZIl3zqT6icNb4wvHvyau70fUAqhLhrbloibhSA/640?wx_fmt=png&from=appmsg)

对比一下, 除了正常密文前面多了个 M, 以及开头有一段相同的, 后面都不一样, 于是我尝试能不能解密一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkick3aSASjR4dCHNibXccZIl3zqT6icNb4wvHvyau70fUAqhLhrbloibhSA/640?wx_fmt=png&from=appmsg)

可以看到只解密出了前 16 个字节, 到这里我就感觉有点不太对了, 一般来说开发人员不会乱写, 如果后续他维护起来也比较麻烦, 除非是那种故意写出来迷惑逆向人员的, 但前面的 aes 算法他又暴露了出来, 所以我感觉上面的推论可能有点问题, 也就是说可能真的是 cbc 模式. 如果是 ecb 模式下由于分组加密, 每个分组单独加密, 互不关联, 能解第一组的话按理后面的也能解. 但如果是 cbc 模式下每个明文分组先和上个分组的密文块进行异或, 直接放到 ecb 模式下肯定解不出来, 那为什么可以解出来第一组呢? 我们先看加密模式下, 第一个分组下明文和 iv 异或后进行后续加密, 如果只解密第一组则不需要在 cbc 模式下, ecb 就可以, 并且解密出来的结果是明文和 iv 异或的结果, 也就是说明文和 iv 异或后还是明文, a 异或 b 得到 a, 只有一种情况, b 全为 0, 也就是说 iv 是 00000000000000000000000000000000

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KktaEazptCoDibYrGDAoXyXwaQicGdf46iaKiau8XyMsG6OyIKLqFAZLRMMw/640?wx_fmt=png&from=appmsg)

看看结果完全正常, 也就是说上面的推论有问题, 我们再来仔细看看上面的推论

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kks5QQwjr2c9AxeZfYcpgee0QSCpxoFvcR4ib2WJt4tiaSeXJchBdXV3DA/640?wx_fmt=png&from=appmsg)

我们否定了 pkcs7 填充方式, 上面用了两个如果, 并不能否定 cbc 模式, 如果是 cbc 模式下的 zero padding 模式再来看看, 解密结果是 68656c6c6f0000000000000000000000, 这种情况下 68656c6c6f(hello 的 hex 形式)zero padding 后是 68656c6c6f0000000000000000000000, 再和 iv 00000000000000000000000000000000 异或后还是它本身 68656c6c6f0000000000000000000000, 这样的话就说的通了. 所以正确的加密模式应该是 aes128-cbc 模式 - zero _padding 填充

key 为 F6F472F595B511EA9237685B35A8F866,iv 为 00000000000000000000000000000000

小坑
--

这里有个坑, 当我把明文用上面的加密模式加密一遍, 发现结果不对, CyberChef 中默认是 pkcs7 填充, 如果能完全解密就说明就是 pkcs7 填充, 可是我们上面的推论也每错啊!!! 别急, 听我细说.

我把之前的修改 r0 为指向新字符串的新指针注释掉, 采用原始的明文进行填充, 这是填充前, 304 字节刚好 19 轮

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkKS8QMQglxyrqHlLQXuOHcJMmRCnjiaNP1Skib1VUjvhrdAS42hN1Qylw/640?wx_fmt=png&from=appmsg)

InsertCBCPadding 执行后

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkUGeOJyYSYYaYstc3Oia2CGxGMUfDHCcLNdPQYV7JqQSPW07hJvRkCTw/640?wx_fmt=png&from=appmsg)

末尾填充了 3 个 03, 这正是 pkcs7 的填充模式, 那为什么上面用 hello 的明文填充后后面是 0 呢, 这个我也不太清楚这个修改 r0 寄存器指向新指针的操作, 看下面的代码

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p></td><td width="678"><p><code>debugger.addBreakPoint(module.base+</code><code>0x5A34</code><code>,&nbsp;</code><code>new</code>&nbsp;<code>BreakPointCallback() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>@Override</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>public</code>&nbsp;<code>boolean</code>&nbsp;<code>onHit(Emulator&lt;?&gt; emulator,&nbsp;</code><code>long</code>&nbsp;<code>address) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>String fakeInput =&nbsp;</code><code>"hello"</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>int</code>&nbsp;<code>length = fakeInput.length();</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>MemoryBlock fakeInputBlock = emulator.getMemory().malloc(length,&nbsp;</code><code>true</code><code>);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>fakeInputBlock.getPointer().write(fakeInput.getBytes(StandardCharsets.UTF_8));</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>// 修改r0为指向新字符串的新指针</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>emulator.getBackend().reg_write(ArmConst.UC_ARM_REG_R0, fakeInputBlock.getPointer().peer);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>true</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>});</code></p></td></tr></tbody></table>

这里我用 python aes 存算计算了如果使用 zero padding 模式加密得到的结果也正是最开始的密文 57b0d60b1873ad7de3aa2f5c1e4b3ff6

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2Kkbq5Oo2iab8ibbZVicdO2xRwpZ8zkWfyGJPXDyo2CuQRphkRxnv5Lp20iaQ/640?wx_fmt=png&from=appmsg)

说明上面的推论都没有错, 只不过是修改 r0 为指向新字符串的新指针后经过 InsertCBCPadding 并没有完成 pkcs7 填充, 但是正常的明文是经过了 pkcs7 填充的, 这里我也不清楚是为什么, 但肯定和这个修改 r0 为指向新字符串的新指针有很大关系.

所以正确的加密模式应该是 aes128-cbc 模式 - pkcs7 填充

key 为 F6F472F595B511EA9237685B35A8F866,iv 为 00000000000000000000000000000000

写到这里我本来想把上面的错误推论删掉, 但是想了想, 并不是只有得到正确的结果才会让人进步, 所以我保留了, 相信每个读者逆向的时候都会有自己的思路, 我想我把自己的思路比较完整的写出来了.

md5
---

再来看上面的明文块

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p></td><td width="685"><p><code>mobile</code><code>=</code><code>13535535353</code><code>&amp;password</code><code>=</code><code>fjfjfjffk&amp;client_id</code><code>=</code><code>2019041810222516127</code><code>&amp;client_secret</code><code>=</code><code>c5ad2a4290faa3df39683865c2e10310&amp;state</code><code>=</code><code>eu4acofTmb&amp;response_type</code><code>=</code><code>token&amp;ostype</code><code>=</code><code>ios&amp;imei</code><code>=</code><code>unknown&amp;mac</code><code>=</code><code>02</code><code>:</code><code>00</code><code>:</code><code>00</code><code>:</code><code>00</code><code>:</code><code>00</code><code>:</code><code>00</code><code>&amp;model</code><code>=</code><code>Pixel&nbsp;</code><code>4</code>&nbsp;<code>XL&amp;sdk</code><code>=</code><code>29</code><code>&amp;serviceTime</code><code>=</code><code>1709100421650</code><code>&amp;mod</code><code>=</code><code>Google&amp;checkcode</code><code>=</code><code>6be9743e9f528df4cd9465a97cb645a1</code></p></td></tr></tbody></table>

前面几个应该是可以固定的, 后面有个 checkcode, 按单词的意思就是检查代码, 中文翻译过来可以理解为验签, 防止 aes 被人破解的情况下如果明文被篡改需要把这个值一并改掉, 否则不给通过.

接下来重点看看这个 checkcode,32 位首先猜 md5, 上面的图中也看到了疑似 md5 的 64 轮运算

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkcfgR7z3jic5NPSs9Y078VDr4AIDeRCyP5yJ4S2JogRAXhibGVTrrnadQ/640?wx_fmt=png&from=appmsg)

这里我先对明文加密了一下, 但是不确定是否有盐值

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p></td><td width="685"><p><code>6be9743e9f528df4cd9465a97cb645a1</code>&nbsp;<code>明文中的结果</code></p><p><code>7cb645a19f528df4cd9465a96be9743e</code>&nbsp;<code>md5后的结果</code></p></td></tr></tbody></table>

这样一对比好像中间一串是一样的, 拆分看看

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p></td><td width="685"><p><code>6be9743e</code>&nbsp;<code>9f528df4</code>&nbsp;<code>cd9465a9&nbsp;</code><code>7cb645a1</code></p><p><code>7cb645a1</code>&nbsp;<code>9f528df4</code>&nbsp;<code>cd9465a9&nbsp;</code><code>6be9743e</code></p></td></tr></tbody></table>

明眼人都能看出来前 4 个字节和后 4 个字节调换了顺序, 这样的话也不需要去 ida 中看代码了, 直接就得到了结果, 这确实有点运气的成分在, 但是运气也是实力的一部分啊!

完整算法
----

替换你自己的 mobile 和 password 即可, 友情提醒, 本文章中所有内容仅供学习交流使用，不用于其他任何目的, 请勿对目标 app 发生大规模请求, 否则后果自负!!!

#### 本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请联系作者立即删除！

<table cellpadding="0" cellspacing="0" width="717"><tbody><tr><td width="20"><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p><p>48</p><p>49</p><p>50</p><p>51</p><p>52</p><p>53</p><p>54</p><p>55</p><p>56</p><p>57</p><p>58</p><p>59</p><p>60</p><p>61</p><p>62</p><p>63</p><p>64</p></td><td width="678"><p><code>import</code>&nbsp;<code>base64</code></p><p><code>from</code>&nbsp;<code>Crypto.Cipher&nbsp;</code><code>import</code>&nbsp;<code>AES</code></p><p><code>import</code>&nbsp;<code>requests</code></p><p><code>import</code>&nbsp;<code>hashlib</code></p><p><code>from</code>&nbsp;<code>Crypto.Util.Padding&nbsp;</code><code>import</code>&nbsp;<code>unpad</code></p><p><code>def</code>&nbsp;<code>__pkcs7padding(plaintext):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>block_size&nbsp;</code><code>=</code>&nbsp;<code>16</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>text_length&nbsp;</code><code>=</code>&nbsp;<code>len</code><code>(plaintext)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>bytes_length&nbsp;</code><code>=</code>&nbsp;<code>len</code><code>(plaintext.encode(</code><code>'utf-8'</code><code>))</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>len_plaintext&nbsp;</code><code>=</code>&nbsp;<code>text_length&nbsp;</code><code>if</code>&nbsp;<code>(bytes_length&nbsp;</code><code>=</code><code>=</code>&nbsp;<code>text_length)&nbsp;</code><code>else</code>&nbsp;<code>bytes_length</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>plaintext&nbsp;</code><code>+</code>&nbsp;<code>chr</code><code>(block_size&nbsp;</code><code>-</code>&nbsp;<code>len_plaintext&nbsp;</code><code>%</code>&nbsp;<code>block_size)&nbsp;</code><code>*</code>&nbsp;<code>(block_size&nbsp;</code><code>-</code>&nbsp;<code>len_plaintext&nbsp;</code><code>%</code>&nbsp;<code>block_size)</code></p><p><code>def</code>&nbsp;<code>aes_encrypt(mobile,password):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>_str&nbsp;</code><code>=</code>&nbsp;<code>f</code><code>'mobile={mobile}&amp;password={password}&amp;client_id=2019041810222516127&amp;client_secret=c5ad2a4290faa3df39683865c2e10310&amp;state=eu4acofTmb&amp;response_type=token&amp;ostype=ios&amp;imei=unknown&amp;mac=02:00:00:00:00:00&amp;model=Pixel 4 XL&amp;sdk=29&amp;serviceTime=1709100421650&amp;mod=Google'</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>checkcode&nbsp;</code><code>=</code>&nbsp;<code>hashlib.md5(_str.encode()).hexdigest()</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>swapped_string&nbsp;</code><code>=</code>&nbsp;<code>checkcode[</code><code>24</code><code>:]&nbsp;</code><code>+</code>&nbsp;<code>checkcode[</code><code>8</code><code>:</code><code>24</code><code>]&nbsp;</code><code>+</code>&nbsp;<code>checkcode[:</code><code>8</code><code>]</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>plaintext&nbsp;</code><code>=</code>&nbsp;<code>_str</code><code>+</code><code>'&amp;checkcode='</code><code>+</code><code>swapped_string</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>key&nbsp;</code><code>=</code>&nbsp;<code>bytes.fromhex(</code><code>'F6F472F595B511EA9237685B35A8F866'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>iv&nbsp;</code><code>=</code>&nbsp;<code>bytes.fromhex(</code><code>'00000000000000000000000000000000'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>aes&nbsp;</code><code>=</code>&nbsp;<code>AES.new(key, AES.MODE_CBC, iv)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>content_padding&nbsp;</code><code>=</code>&nbsp;<code>__pkcs7padding(plaintext)&nbsp;&nbsp;</code><code># 处理明文, 填充方式</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>encrypt_bytes&nbsp;</code><code>=</code>&nbsp;<code>aes.encrypt(content_padding.encode(</code><code>'utf-8'</code><code>))&nbsp;&nbsp;</code><code># 加密</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>'M'</code>&nbsp;<code>+</code>&nbsp;<code>str</code><code>(base64.b64encode(encrypt_bytes), encoding</code><code>=</code><code>'utf-8'</code><code>)&nbsp;&nbsp;</code><code># 重新编码</code></p><p><code>def</code>&nbsp;<code>decrypt(text):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>ciphertext&nbsp;</code><code>=</code>&nbsp;<code>base64.b64decode(text)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>key&nbsp;</code><code>=</code>&nbsp;<code>bytes.fromhex(</code><code>'F6F472F595B511EA9237685B35A8F866'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>iv&nbsp;</code><code>=</code>&nbsp;<code>bytes.fromhex(</code><code>'00000000000000000000000000000000'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>cipher&nbsp;</code><code>=</code>&nbsp;<code>AES.new(key, AES.MODE_CBC, iv)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>plaintext&nbsp;</code><code>=</code>&nbsp;<code>cipher.decrypt(ciphertext)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>decrypted_data&nbsp;</code><code>=</code>&nbsp;<code>unpad(plaintext, AES.block_size, style</code><code>=</code><code>'pkcs7'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp;<code>decrypted_data.decode(</code><code>"utf-8"</code><code>)</code></p><p><code>def</code>&nbsp;<code>login():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>headers&nbsp;</code><code>=</code>&nbsp;<code>{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"channel"</code><code>:&nbsp;</code><code>"yingyongbao"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"platformNo"</code><code>:&nbsp;</code><code>"Android"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"appVersionCode"</code><code>:&nbsp;</code><code>"1481"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"version"</code><code>:&nbsp;</code><code>"V8.0.14"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"imei"</code><code>:&nbsp;</code><code>"a-759f0c27ef7fe3b6"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"imsi"</code><code>:&nbsp;</code><code>"unknown"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"deviceModel"</code><code>:&nbsp;</code><code>"Pixel 4"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"deviceBrand"</code><code>:&nbsp;</code><code>"google"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"deviceType"</code><code>:&nbsp;</code><code>"Android"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"accessChannel"</code><code>:&nbsp;</code><code>"1"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code># "oauthConsumerKey": "2019041810222516127",</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"timestamp"</code><code>:&nbsp;</code><code>"1709100421649"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"nonce"</code><code>:&nbsp;</code><code>"PCpLXbXts7"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"Content-Type"</code><code>:&nbsp;</code><code>"application/x-www-form-urlencoded; charset=utf-8"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"Host"</code><code>:&nbsp;</code><code>"api.00bang.cn"</code><code>,</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"User-Agent"</code><code>:&nbsp;</code><code>"okhttp/4.9.0"</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>url&nbsp;</code><code>=</code>&nbsp;<code>""</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>mobile&nbsp;</code><code>=</code>&nbsp;<code>''&nbsp;&nbsp;</code><code># 换成你自己的</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>password&nbsp;</code><code>=</code>&nbsp;<code>''&nbsp;</code><code># 换成你自己的</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>sd&nbsp;</code><code>=</code>&nbsp;<code>aes_encrypt(mobile,password)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(sd)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>data&nbsp;</code><code>=</code>&nbsp;<code>{</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>"sd"</code><code>: sd</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>response&nbsp;</code><code>=</code>&nbsp;<code>requests.post(url, headers</code><code>=</code><code>headers, data</code><code>=</code><code>data,verify</code><code>=</code><code>False</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'加密结果:'</code><code>,response.text)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(response)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'解密结果'</code><code>,decrypt(response.json()[</code><code>'sd'</code><code>][</code><code>1</code><code>:]))</code></p><p><code>if</code>&nbsp;<code>__name__&nbsp;</code><code>=</code><code>=</code>&nbsp;<code>'__main__'</code><code>:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>login()</code></p></td></tr></tbody></table>

总结
--

1 由于本节涉及知识点重多, 有很多讲解不到位的地方还请在评论区指出!

2 本章所涉及的材料都上传在网盘了,, 刚兴趣的自行还原验证, 相信对你的安卓逆向水平一定会有提升!

3js 逆向转安卓逆向, 如有讲解错误的还请多多包涵!

4 技术交流 + v lyaoyao__i(两个杠)

最后
--

微信公众号: 爬虫爬呀爬

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkXibTiamUIvqPodV9icsJHmLy3JXnp1pUpRxfY9pE3TXhuEOib5KpwkKicIQ/640?wx_fmt=jpeg&from=appmsg)

如果你觉得这篇文章对你有帮助, 不妨请作者喝一杯咖啡吧!

![](https://mmbiz.qpic.cn/sz_mmbiz_png/VVlcG4MNLdURvOtPsIY1CRv5gSObH2KkscGssShMDLhqfWhWstNIzgSchCheJrp5H6UibiaicyJeF7zyzhTZuWicYg/640?wx_fmt=png&from=appmsg)