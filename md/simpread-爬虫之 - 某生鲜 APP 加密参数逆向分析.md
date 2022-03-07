> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuanrenxue.com](https://www.yuanrenxue.com/app-crawl/app-crawl-1.html)

> 本文是跟我学习爬虫的小伙伴：彭良怀的投稿，稿费是 500。

 PS：他在北京，有看上的老板可以私信我，为人也不错。

学了一段时间 APP 逆向，刚刚入门，我以某生鲜 APP 为例，记录一下逆向过程和一些知识点。为了不影响对方的利益，我文中特意隐去了该 APP 的名字信息，本文仅供学习交流，请勿用作其他用途。

使用到的工具如下：

*   一部 root 后的安卓手机，模拟器也可以
    
*   抓包工具：Charles
    
*   查壳工具：APK Messenger
    
*   APK 反编译工具：jadx-gui 1.1
    
*   SO 文件分析工具：IDA_Pro_v7.0
    
*   Hook 框架：frida
    

### **二、抓包分析**

首先手机配置好代理，打开 APP，用 Charles 抓一下包，还好直接就抓到了，如下图所示：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress10-1582206573.jpg)

可以看到很多的请求参数，翻页再抓包一次，把两次抓到的参数进行比对，看看哪些参数固定，哪些是变化的。这么多参数，要是自己用肉眼看，那就太费劲了，而且还容易看漏，所以直接用在线文本对比工具吧，我用的是这个网站：https://qqe2.com/word/diff，把两次抓包的参数复制上去，如下图所示：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress1-1582206574.jpg)

不同的参数都高亮出来了，一目了然。我简单分析如下：

*   signKey：密文，长度 32 位，可能为 MD5、HmacMD5 加密或随机 UUID
    
*   signKeyV1：密文，长度 64 位，可能为 SHA256、HmacSHA256 加密
    
*   t ：13 位时间戳
    
*   traceId：等于 deviceId （固定的设备 ID）加两个 13 位时间戳
    
*   currentPage：页码
    
*   lastStoreId：上一页最后一家店铺 ID
    

其他固定参数很好理解，我就不阐述了。通过模拟请求验证，修改任意参数的值都无法获取数据，所以推测 signKey 和 signKeyV1 是由其他请求参数加密生成的。那接下来就去看看 java 代码吧。

### **三、Java 层分析**

#### **1. 查壳**

在反编译 apk 之前，首先查下壳，因为加壳（加固）后的 apk 直接反编译是看不到有用信息的。查壳工具很多，这里我使用的是 APK Messenger，打开后，直接将 apk 包拖入界面，即可看到有没有加壳，如下图所示：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress8-1582206574.png)

结果疑似无壳，接下来就可以使用 jadx 反编译 apk 了。需要注意的是，查壳功能的实现往往只是遍历 APK 内文件和目录，以加固厂商（腾讯、360、阿里、百度、梆梆等）的常用文件名作为判断特征，比如百度的加固一般在 lib 目录下有一个 libbaiduprotest.so 文件，但有可能人家使用了新的名字，所以查壳有一定的误判率。你还可以在 Apk Messenger 中查看、增加或编辑加固的判别特征。

#### **2. 分析关键 Java 代码**

用 jadx 打开 apk ，反编译为 java 代码，然后按 Ctrl + Shift + F 全局搜索 signKeyV1，直接可以定位到如下代码：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress0-1582206574.jpg)

这段代码很好理解，明显是在组装参数，可以从中得出以下信息：

*   t 为当前时间戳；
    
*   subVersion 为当前 APP 版本号；
    
*   signKey 是由 k 方法生成的；
    
*   signKeyV1 等于 KEY_NEW_SIGN， KEY_NEW_SIGN 又是由 k2 方法生成的；
    
*   传入方法 k2 的参数为 formatQueryParaMap 方法的返回值；
    
*   方法 k 和 k2 都在 native 层，加载的是 libjdpdj.so 文件；
    

#### **3. 分析 formatQueryParaMap 方法**

k 和 k2 都在 native 层，我们还是先看看 formatQueryParaMap 方法吧，按住 Ctrl 键同时鼠标左键点击 formatQueryParaMap 即可跳转到该方法，如下图所示：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress5-1582206575.jpg)

这段代码也好理解，传入该方法的第一个参数为 Map 类型，类似 Python 中的字典，它先根据 key 进行排序，然后再把 value 用 & 字符进行拼接（functionId 的值除外 ），用 Python 代码实现如下：

```
def formatQueryParaMap(param: dict) -> str: 
return '&'.join(param[k] for k in sorted(param.keys()) if k != 'functionId')

```

#### **4. Hook formatQueryParaMap 方法**

如果看不懂或不想分析 formatQueryParaMap() 也没关系，我们直接用 frida hook 一下这个方法，看看它的输入和输出，也能反向推测出这个方法是做什么的，hook 代码如下：

```
Java.perform(function () { 
var util = Java.use('jd.net.ASCIISortUtil'); 
util.formatQueryParaMap.implementation = function (arg1, arg2) { 
console.log('param1: ', arg1); 
console.log('param2: ', arg2); 
var result = this.formatQueryParaMap(arg1, arg2); 
console.log('return: ', result); 
return result; 
};
})

```

打印结果如下：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress6-1582206575.jpg)

很明显，param1 就是最开始抓包到的那些请求参数，那么我们就知道了方法 k2 的输入参数要怎么构造了，接下来分析方法 k 、 k2 是怎么加密的，就不得不分析 .so 文件了。

#### **5. 关于反调试**

这里提一下，该 APP 有反调试，开启 frida-server 后 ，启动 APP 就立即闪退，可别急着去破解它的反调试，即找到反调试的地方干掉后重新打包签名，可这样做就很麻烦了，不知道得掉多少头发。还好，先启动 APP 等进入主界面后再启动 frida-server，就能正常进行 hook 了，虽然偶尔还是会被强制闪退，但频率不高，影响不大。

### **四、Native 层分析**

通过 Java 层的分析知道 ，signKey 和 signKeyV1 分别是方法 k 和 k2 生成的，而这两个方法又是定义在 native 层的，那么就得先找到 k、k2 在 native 层中对应的函数，然后再分析具体的加密过程。为了便于理解，我先讲知识点，再讲操作。

#### **1. 静态注册和动态注册**

因为 java 层和 native 层的代码往往相互调用，使用的是一种叫 JNI (Java Native Interface) 的技术，在 java 层中调用 native 函数之前, 要对 java 中 native 关键字定义的方法进行注册，注册方式有两种：静态注册和动态注册。下面简单介绍一下：

*   静态注册：
    
    静态注册是通过固定格式方法名进行关联，命名规则如下：
    
    > native 函数名 = Java + 包名 + 类名 + 方法名
    
    例如，包名: com.example.test，类名：jd.net.z，方法名：k
    
    如果是静态注册的话，那么 native 中的函数名就该为：Java_com_example_test_jd_net_z_k
    
*   动态注册：
    
    动态注册是通过 RegisterNative() 这个 JNI 函数动态添加映射关系来进行关联的，这种方式可以随便命名函数名，比较灵活。其申明示例如下：
    
*   ```
    jint RegisterNatives(JNIEnv *env, jclass clazz, const JNINativeMethod* methods, jint nMethods)
    
    ```
    
    第 1 个参数是 JNIEnv 指针，所有 JNI 函数第一个参数都是它；
    
    第 2 个参数 clazz 是注册方法对应 Java 层中的类，由 FindClass 函数获取；
    
    第 3 个参数 methods 是一个数组，其中包含了注册方法结构体信息，我们可以从中找到注册前后的方法名，所以我们注意这个参数就行了；
    
    第 4 个参数 nMethods 是动态注册方法的数量。
    

#### **2. 找到 k、k2 对应的 native 函数**

知道了 native 函数的两种注册方式，那就开始具体的操作吧。用 IDA 打开 libjdpdj.so 文件，切换到 Exports 窗口，我们先按照静态注册的命名规则搜索：Java，并没有搜到，那么便是动态注册了。

因为 JNI_OnLoad() 是加载 so 文件的初始函数，可以从中找到 RegisterNative()。那么搜索 JNI_OnLoad ，双击进入，按 F5 把汇编转成伪 C 代码，你会发现并没有找到 RegisterNative，别急，这是因为 IDA 不能准确的识别函数声明或变量类型，反编译不完全正确造成的，但我们可手动将其还原。

凡是看到类似 (*(_DWORD *)v2 + 860))(v2, …) 这种代码的其实都是 JNI 函数，我们选中参数 v2 后按 Y 键会弹出窗口，输入 JNIEnv * ，点击 OK 即可还原函数名，还原后如下所示：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress5-1582206575.png)

根据前面的介绍，我们只需要看第 3 个参数即可，双击 &off_117004 跳转到如下汇编代码：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress4-1582206575.jpg)

从 117004 偏移量那一行开始，每 3 行为一个结构体，一共 8 个。我们看第一个，其第一行右边的注释 “k” 就是 java 层的方法名，第二行为 JNI 字段描述符，描述了该方法的参数类型和返回值类型，第三行就是我们要找的动态注册后的函数名，可以看到为：gk；同样的，”k2″ 对应的就是：gk2。

搜搜看，这就很容易找到了：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress9-1582206576.png)

不过有些 APP 为了防止被静态分析，对注册函数做了混淆，通过这种方式并不能直接找到，这里我就不讨论了，遇到的童鞋可以参考赵四这篇博客：http://www.520monkey.com/archives/1289

#### **3. JNI 静态调试的一些技巧**

在分析 gk 函数之前我先谈谈静态分析 native 函数的一些技巧和个人经验。

##### (1) 批量还原 JNI 函数名

native 函数中经常会用到很多的 JNI 函数，而 IDA 并不能很好的识别，每次我们都要一个个手动修改未免太麻烦了点，所以我介绍一个可以批量转换的方式：

*   按 Ctrl + F9 ，选择 jni.h 头文件导入
    
*   导入成功后，鼠标左键点击其中一个 JNI 函数的参数，然后右键选择 Convert to Struct *
    
*   在弹出的 Select a structure 窗口中 选择 _JNIEnv，点击 OK
    

这样就可以把当前打开的 native 函数里面所有 JNI 函数名一次性还原了。注意 jni.h 头文件第一导入会报错，需要根据报错信息修改 jni.h 对应的代码。

##### (2) 强制调出函数参数

有时会遇到 IDA 反编译出来的函数连参数都没有，如下面的 GetArrayLength 函数后面的参数为空：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress3-1582206576.png)

这时需要鼠标左键点击该函数，然后鼠标右键选择 Force call type ，就能强制把参数调出来。

##### (3) 常用快捷键

*   shift + F12：查看 so 文件中所有常量字符串的值；
    
*   tab 键：汇编和伪 C 代码之间相互切换；
    
*   / 键：添加注释；
    
*   N 键：变量重命名；
    
*   X 键：查看某变量的所有引用；
    
*   = 键：消除冗余的中间变量；
    
    由于 IDA 反编译出来总是会有很多冗余的中间变量，如：
    
    v2 = v1;  
    result = encrypt(v2);
    
    选中 v2，按键盘上的 = 键，再点击 OK，即可消除中间变量 v2：
    
    result = encrypt(v1);
    

##### (4) 静态调试思路

*   根据函数入参，至上而下分析
    
*   根据函数返回值，至下而上分析
    
*   寻找关键的函数进行分析，一般可以把函数分为以下几种：
    
    ① 标准库函数：如 strlen()，计算字符串的长度，见名知意；
    
    ② JNI 函数：如 FindClass()，调用 Java 中的类，JNI 函数一般也是见名知意；
    
    ③ 用户自定义的函数：如 MD5::MD5()，一看就知道是 MD5 加密，这类需特别注意；
    
    ④ IDA 命名的函数：如 sub_567C()，IDA 会对没有名字的函数自动命名，命名规则就是 sub_ + 函数地址，这类函数也是重点。
    
    从追求效率的角度来说，最好先找关键函数，看看有没有常见的加密函数名，找到后直接用 frida hook，一些简单的往往能够一击中的，快速破解。从学技术的角度来说，可以多尝试一行一行代码地分析，锻炼看代码的能力。当然复杂点的还不得不分析 arm 指令，要是被混淆后就更加难了，难的我也不会，以后多练多学吧。
    

#### **4. 静态分析 gk 函数**

接下来开始具体操作吧，双击 gk 函数后看到汇编 arm 指令，按 F5 键反汇编为伪 C 代码，并把 JNI 函数名还原。我这里就不一一分析每行代码了，直接先找关键函数，很容易就找到如下代码：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress6-1582206576.png)

很明显是 MD5 加密，MD5Init() 是一个初始化函数，MD5Update() 才是 MD5 的主计算过程，所以直接 hook MD5Update() ，用 frida hook native 层函数得需要找到目标函数的绝对地址，而目标函数可能是导出函数，也可能是未导出函数，我先分别介绍一下怎么获取他们的地址吧：

获取导出函数的绝对地址:

// JNI_OnLoad 肯定是导出函数, 可直接根据名字获取

```
var onload_addr = Module.getExportByName('libjdpdj.so', 'JNI_OnLoad');

```

获取未导出函数的绝对地址，我列举以下 3 种方式：

*   方式一：
    
*   ```
    var onload_addr = Module.getExportByName('libjdpdj.so', 'JNI_OnLoad'); 
    var base_addr = parseInt(onload_addr ) - parseInt('0x34D6C'); 
    var md5_update_addr = ptr(base_addr + parseInt('0x34E18'));
    
    ```
    
*   方式二：
    
*   ```
    var onload_addr = Module.getExportByName('libjdpdj.so', 'JNI_OnLoad'); 
    var md5_update_addr = onload_addr.sub(0x34D6C).add(0x34E18);
    
    ```
    
*   方式三：
    
*   ```
    var md5_update_addr = Module.findBaseAddress("libjdpdj.so").add(0x34E18 + 1);
    
    ```
    
    方式一看注释很好理解，方式二其实就是方式一的简化，用 frida 提供的的 add() 和 sub() 函数进行地址的加减。方式三是进一步简化，但是用这种方式一定要记得对地址 +1，为什么要 +1 呢？我引用赵四的原话解释吧：
    
    > 因为 thumb 和 arm 指令的区分，地址最后一位的奇偶性来进行标志
    
    获取未导出函数地址的方式也完全适用于导出函数，所以不管导出还是未导出，我都用方式三获取，代码简单优雅。
    

那么我们 hook MD5Update() 的代码如下：

```
var pointer = Module.findBaseAddress("libjdpdj.so").add(0x34E18 + 1); 
console.log('MD5Update pointer:', pointer);
Interceptor.attach(pointer, { 
onEnter: function(args) { 
console.log('参数1:', args[0]); 
console.log('参数2:', Memory.readCString(args[1])); 
console.log('参数3:', parseInt(args[2])); 
console.log('----------------');
 }, 
onLeave: function(retval) {
 } 
})

```

hook 的时候我们同时对其抓包，以便验证，hook 打印的结果如下：

```
MD5Update pointer: 0xaed5ae19 
参数1: 0xbef0eb8c 
参数2: {"city":"重庆市","latitude":29.57252,"longitude":106.53355,"address":"观音桥", "coordType":"2","channelId":"4037","appVersion":"7.4.0","platform":"2","currentPage":1, "pageSize":10,"areaCode":4,"ref":"home","ctp":"channel"}923047ae3f8d11d8b19aeb9f3d1bc002 
参数3: 259

```

—————-

可以看到参数 2 为部分请求参数再上加尾部的盐值，这便是加密前的原文。我们把它拿去用 MD5 在线加密一下，其结果和抓包到的 signKey 进行对比，经验证完全相同，那么 signKey 被一击中的，具体的代码都不用去分析了。其实服务器并没有对该参数进行校验，我们直接生成一个随机的 32 位字符就行，我这里主要是讲一下方法。

#### **5. 静态分析 gk2 函数**

然后再来看 gk2 函数，同样首先找有没有常见的加密，很快在最后几行看到如下代码：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress0-1582206576.png)

很明显是 hmac_sha256 加密，看到它有 6 个参数，往上追溯可知，第 1 个参数 s 为加密前的字符串，第 2 个参数 v23 为 s 的长度，这里 v23 – 32 说明加密前需要去掉最后 32 个字符，第 3 个参数为密钥，第 4 个参数是密钥的长度，最后两个参数没有什么操作，不用管。那么我们就直接用 frida hook hmac_sha256 函数，打印一下参数看看，代码如下 ：

```
var pointer = Module.findBaseAddress("libjdpdj.so").add(0x361B8 + 1); 
console.log("hmac_sha256 pointer: ", pointer);
Interceptor.attach(pointer, { 
onEnter: function(args) { 
console.log("参数1:", Memory.readUtf8String(args[0])); 
console.log("参数2:", parseInt(args[1])); 
console.log("参数3:", Memory.readCString(args[2])); 
console.log("参数4:", parseInt(args[3])); 
console.log('---------------'); 
}, 
onLeave:function(retval){
 } 
});

```

hook 的时候我们同时对其抓包，以便验证，hook 打印的结果如下：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress3-1582206576-1.png)

参数 1 去掉末尾的 32 位字符就是入参，参数 3 是密钥，于是把入参拿去用 HmacSHA256 加密一下，其结果再和抓包到 signKeyV1 进行对比，经验证完全相同，由此 signKeyV1 也被一击中的。

抱着学习的心态再去分析一下伪 C 代码，具体分析过程我就不介绍了，就说一下大致的逻辑：

*   先调用 java 层的 getsign 方法获取基础 key，
    
*   对基础 key 每个字符的 ASCII 码进行修改，同时拼接到输入参数的尾部，
    
*   取出入参尾部的 32 位作为密钥，
    
*   最后对输入参数进行 hmac_sha256 加密，通过指针返回加密结果。
    

逆向到这儿就结束了，后面用 python 实现不难，我就不贴代码了，关键过程讲清楚了就行。

#### **6. native 函数的参数**

我再啰嗦一下，native 函数要比 java 层对应方法多 2 个参数，它们前两个参数是固定的，第 1 个参数为 JNIEnv 指针；第 2 个为 jobject 或 jclass；从第 3 个参数开始才是 java 层传递过来的。比如：gk() 函数的申明如下：

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress9-1582206576-1.png)

其中 a3 才是 java 层 k() 方法的参数。前两个参数之所以是 int 类型，前面也说过，是因为 IDA 经常不能正确识别参数类型，这里按 Y 键手动转换一下，或者直接忽略，没什么影响。

### **五、总结**

本篇文章的案例 APP 也是大厂开发的，而我们对其 java 层和 native 层的加密函数分析都不难，没有复杂难懂的逻辑，也没有混淆，只有个鸡肋的反调试，直接静态分析加 frida hook 就搞定了。其实目前市面上大多数 APP 的加密参数都能通过这种方式搞定，当然很难的也不少，学习逆向是个无底洞，但我们做爬虫的不要怕逆向，我们只是逆向它的那个加密参数而已，先要有信心，多学习多实操多总结，一点点深入，会学有所成。共勉！

再次跨一下这篇文章，非常不错，继续接受投稿，稿费还不错 300-500 / 篇，快来投稿吧。

PS，给自己广告一下：我继续在教爬虫，真正的爬虫技术。教 APP 逆向抓取 / JS 逆向抓取 / 大规模爬虫框架设计 / 利用爬虫技术做被动收入。

感兴趣的加我微信私聊，备注：爬虫。

最近打算建一个爬虫技术交流群，感兴趣的也可以加我。

![](https://www.yuanrenxue.com/wp-content/uploads/2020/02/beepress7-1582206577.jpeg)