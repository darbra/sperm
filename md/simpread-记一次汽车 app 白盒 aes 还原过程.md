> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iaS_mfEtK2e-yakE14IvxQ)

本文仅作学习移动安全交流，请勿用于非法用途。

目标 app：5LqU6I+x5rG96L2m  
包名：Y29tLmNsb3VkeS5saW5nbGluZ2Jhbmc=  
版本：8.2.1  
加固：梆梆企业版

  

---

```
一





抓包&加密字段的定位

```

（1）抓取应用点击登录的接口，可以看到请求体和返回体被加密了，加密的字段名都为 sd。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rFIWqSiaCeibiareQNxaYL3rWYvUM9QTWVrk2SCxhPBUgiamb8gea6fFRFA/640?wx_fmt=png&from=appmsg)

（2）使用 xposed 尝试注入自吐脚本：发现没有需要的结果，猜测这是一个 native 层函数。

  
不管静态注册还是动态注册，最终都要走 RegisterNative 这个函数，直接使用 frida hook RegisterNative，看看有哪些 native 层函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r9e5zmCUkYm6oKpkb24ma1SkjFc8SXLdOXCELoot2ZIDHHx0r0CFL9w/640?wx_fmt=png&from=appmsg)

发现应用注册了非常多的 native 层函数，搜索一下 encrypt，其中注意到有一个名为 encrypt 的 so 文件，注册了几次一个名为 checkcode 的方法，我们 hook 一下这个方法看看是不是我们想要的。

  
（3）编写 frida 脚本 hook 一下 com.bangcle.comapiprotect.CheckCodeUtil.checkcode 这个方法。

```
function hook_checkcode(){
    Java.perform(function(){
        let CheckCodeUtil = Java.use("com.bangcle.comapiprotect.CheckCodeUtil");
CheckCodeUtil["checkcode"].overload('java.lang.String', 'int', 'java.lang.String').implementation = function (str1, int1, str2) {
    console.log('checkcode is called' + ', ' + 'str: ' + str1 + ', ' + 'i: ' + int1 + ', ' + 'str2: ' + str2);
    let ret = this.checkcode(str1, int1, str2);
    console.log('checkcode ret value is ' + ret);
    return ret;
};
    })
}


```

注入后，在手机上点一下登录，发现控制台输出了内容，我们与重新抓包的内容比对一下，看看是不是我们需要的结果。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rWlMFqmMvy2CbIWVRuuJ4iaYP1ia74LpnN9VmytSM4J6OCMb1Advibb0ibQ/640?wx_fmt=png&from=appmsg)

可以看到，这个 checkcode 函数，传入的参数中有我们在应用中输入的手机号，且函数返回的内容与我们抓包抓到的结果一致。至此，定位加密的结果有了。

  
ps: 这个加密参数的定位，十分投机取巧。能找到纯属运气，正常情况下应该对应用进行脱壳再一层层通过调用栈进行定位。

  

---

```
二





libencrypt.so分析

```

（1）在应用包 lib/arm64-v8a 下拿到 libencrypt.so 放到 ida 中，在导出函数中搜索 checkcode。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21ricEWgzzNqialHVicHMmuob61knF1pSgicVkjXicRiav2yNvdFWDIDHFFHg1g/640?wx_fmt=png&from=appmsg)

有两个有关 checkcode 的函数，根据函数名，另外一个应该就是解密函数了。  

（2）跟入 checkcode 函数 按下 f5 看伪代码

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rsQFzym56JSq6KkuY1LWKXVHDkOiccBQ9bjCMfjeWCjTUIT7daXBwickA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rIsx1EM1pibZHva5Srr6ARh0FOXK9dyFhHbuHibwQp5b81MQuL115wr6g/640?wx_fmt=png&from=appmsg)  

发现有大量的控制流混淆，好在混淆的不算特别严重，认真分析下还是能看出一个大概的。

  
（3）把函数的几个入参改一下类型和名字，方便 ida 识别出 JNI 结构体，也方便我们后续分析。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r5ActgeQuvFicZuRbdrJgoabjsOhwHiaLsyOPlffrkkmumDYbQYrMMZicQ/640?wx_fmt=png&from=appmsg)  

往下看，开始先判断了传入的第一个参数的 ascii 码判断采用哪个加密函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r8fE1pnurPImnPwhibH8UkdEtQ6dia8RTTibXFwGGMzARSkq5DO7ngicPsA/640?wx_fmt=png&from=appmsg)

接着就是取了一些指纹信息。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rJjjtux6bQAEZSSibC2jrpBbykKbA9kOIKpriaJ6VWEnIicYV2be4sqhrQ/640?wx_fmt=png&from=appmsg)

取完指纹信息后，把字符串进行拼接，最后加密，并把结果 base64 编码。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rERHUAib6zXicPUgKyIJ4FeWzhQKfvEmXFGmsD9US00v49XFpnw0m1dGQ/640?wx_fmt=png&from=appmsg)

（4）拿到加密的函数了，写个代码 hook 下看看入参和返回。

```
function hook_aesencode(){
    let baseaddr = Module.findBaseAddress("libencrypt.so")
    Interceptor.attach(baseaddr.add(0xA5BC),{
        onEnter:function(args){
            console.log("args0:",args[0])
            console.log("args1:",args[1])
            console.log("args2:",args[2])
        },onLeave:function(retval){
            console.log("ret:",retval)
        }
    })
    
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rbodQvsdgJicwnBEP8nJw9JHLTTNDt5J1Z3xFCnDdzq90gdTOl7kcVKA/640?wx_fmt=png&from=appmsg)  

可以看到打印出来的是几个地址，再 hexdump 看看，是什么内容。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r4j0CHuxAvcoj1LW2kibKVZp6ozDHVAwIone5YlHvBatUxBYOibDObYmw/640?wx_fmt=png&from=appmsg)

可以看到，第一个是我们要加密的明文，后面两个看不出是什么先不管。

  
（5）跟入这个 aes_encrypt1 函数

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rAarT3FKfcQrs02ia3ZGfIT4vUkudOaTfOO7DgdY2m6AGJUHeqpX9KzQ/640?wx_fmt=png&from=appmsg)  

可以看到，这个函数貌似是一个 write box(白盒 aes 加密) 先是初始化了一个 CWAESCipher 对象，然后进行表转换，最后加密并返回，跟入 encryptCBC。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21ruwGQhLD7GZl2XmwGoOzEM6yUweC59luE7TCXNjkJpkDClg5o9GDS6Q/640?wx_fmt=png&from=appmsg)

可以看到这个函数先进行了填充，然后进行了一些不知道东西的异或，再往下看。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rjzY6wUvuibTBHMKk3YNSIrIMA59Sd0GNU493P5NVry8oHJTCFtMoCMQ/640?wx_fmt=png&from=appmsg)

这里作了一个循环，根据函数名字判断是进行一个块加密，并判断是否全部加密完成 就跳出循环。

  
（6）再跟入 EncryptOneBlock 函数

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rswcU0icKLibfChic88vicKgN8vib1XdSWibtiaPaictVzUG0EKfp84SuqRoQ0w/640?wx_fmt=png&from=appmsg)

这里应该就是白盒加密的核心部分了，有一些有关 AES 加密流程的相关符号。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r12CUn3132fRLcyhzZicpdfQXyTVA3R1cBPkOpp8icdUGGg8c4beyUqTQ/640?wx_fmt=png&from=appmsg)

最后把当前块加密的结果放入 a3 数组中，并返回结果。

  

---

```
三





Unidbg模拟执行

```

因为是白盒 aes，密钥被隐藏在一个大表中，没办法直接获得加密的 key，又有控制流混淆，所以先考虑模拟执行，再进行分析。

  
（1）先搭个架子：

```
public class CheckCodeUtil extends AbstractJni{
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;

    private final DvmClass CheckCodeUtil;

    private final Memory memory;

    private final DalvikModule dvm;
    public CheckCodeUtil(){
        emulator = AndroidEmulatorBuilder.for64Bit()
        .setProcessName("com.cloudy.linglingbang")
        .build(); // 创建模拟器实例，要模拟32位或者64位，在这里区分
        memory = emulator.getMemory(); // 模拟器的内存操作接口
        memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
        vm = emulator.createDalvikVM(new File("H:\\JavaProject\\unidbg-0.9.7\\unidbg-android\\src\\test\\java\\com\\cloudy\\linglingbang\\wbaes.apk")); // 创建Android虚拟机
        vm.setVerbose(true); // 设置是否打印Jni调用细节
        vm.setJni(this);
        dvm = vm.loadLibrary(new File("H:\\JavaProject\\unidbg-0.9.7\\unidbg-android\\src\\test\\java\\com\\cloudy\\linglingbang\\libencrypt.so"), true); // 加载libttEncrypt.so到unicorn虚拟内存，加载成功以后会默认调用init_array等函数
        module = dvm.getModule(); // 加载好的libttEncrypt.so对应为一个模块
        vm.callJNI_OnLoad(emulator,module);
        CheckCodeUtil = vm.resolveClass("com/bangcle/comapiprotect/CheckCodeUtil");
    }
    public static void main(String[] args) {
        CheckCodeUtil checkCodeUtil = new CheckCodeUtil();
    }
}


```

跑一下看看加载 so 调用 JNIOnload 有没有什么问题：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rbSILz1peSDicKWPMzzU1dUQZlljIIibauDraAMNyhaia7nF9GsCHyRuEw/640?wx_fmt=png&from=appmsg)

报了个环境异常，补上：

```
@Override
public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
    switch (signature) {
        case "android/app/ActivityThread->currentActivityThread()Landroid/app/ActivityThread;":{
            return vm.resolveClass("android/app/ActivityThread").newObject(null);
        }
    }
    return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rEW0zZiaicuia8U7sH6upOibzoVibicTGus0DTakQ7HQNwYY2Gr47TUAamZibQ/640?wx_fmt=png&from=appmsg)

接着补：

```
@Override
public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "android/app/ActivityThread->getSystemContext()Landroid/app/ContextImpl;": {
            return vm.resolveClass("android/app/ContextImpl").newObject(null);
        }       
    }
    return super.callObjectMethod(vm, dvmObject, signature, varArg);
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rs114oZRhbol58oaQmiaIibxniaD0icnbRXNzzbmYcJdQiaoQuQ2cURAgd6Q/640?wx_fmt=png&from=appmsg)

补上补上：

```
@Override
public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "android/app/ActivityThread->getSystemContext()Landroid/app/ContextImpl;": {
            return vm.resolveClass("android/app/ContextImpl").newObject(null);
        }        
        case "android/app/ContextImpl->getPackageManager()Landroid/content/pm/PackageManager;": {
            return vm.resolveClass("android/content/pm/PackageManager").newObject(null);
        }      
    }
    return super.callObjectMethod(vm, dvmObject, signature, varArg);
}


```

补完不报错了，这样 so 加载就没问题了，unidbg 还帮我们把 jni 的调用细节打印出来了。

  
（2）跑目标函数 --checkcode  

写一个 call_checkcode()：

```
public void checkcode(){
    //这里的参数是前面hook java层得到的
    String str1 = "mobile=13288888888&password=123456&client_id=2019041810299999999&client_secret=a72a27b1e11b63d8161f0dfd3cab8cef&state=V6g2Lm8888&response_type=token&ostype=ios&imei=00&mac=00:00:00:00:00:00&model=Pixel 4&sdk=29&serviceTime=1706188888888&mod=Google&checkcode=dd9766a6e55044b08d6880c2430fa6eb";
    String str3 = "1706172888888";
    DvmObject ret = CheckCodeUtil.callStaticJniMethodObject(emulator, "checkcode(Ljava/lang/String;ILjava/lang/String;)Ljava/lang/String;",str1,1,str3);
    String strOut = (String)ret.getValue();
    System.out.println("\ncall checkcode: " + strOut);

}


```

调用一下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rFCu6klvBTO1cJgR6598SlQq2oY6zIevqCF7cq1tDdQaS864d3iaMeTw/640?wx_fmt=png&from=appmsg)

叕叕叕叕报错了，这里是我们前面在 ida 中分析到的，一些环境指纹，补上补上：

```
    @Override
    public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature) {
            case "android/os/Build->MODEL:Ljava/lang/String;": {
                return new StringObject(vm, "Pixel");
            }
            case "android/os/Build->MANUFACTURER:Ljava/lang/String;": {
                return new StringObject(vm, "Google");
            }
            case "android/os/Build$VERSION->SDK:Ljava/lang/String;": {
                return new StringObject(vm, "23");
            }
        }
        return super.getStaticObjectField(vm, dvmClass, signature);
    }


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21reUCh034zyZk5qcDIOR3gFUibErx6ibiaJZRsKUA5HxVXaarHx0WgmaNRg/640?wx_fmt=png&from=appmsg)

结果出来了也不知道对不对，前面提到有 decheckcode 方法，把我们模拟执行的结果调用一下解密看看有没有问题：

```
    public void decheckcode(){
        String str = "MIAYqXNjLK86IzEnTVphowxg1pOlR2iRGolkgHCMocOPKOIlxMZw1yyH4qYEpz2Wc91ZoI0gIb89LZMUFBv5n+oNepjY3fm4DuFwdRiFUNPvBnqKe23fL98oH9ZLa/Ib4ovZcndvhmMykqQ+c+1kSo8h4aPTUlSBHlhyrNRNVyjtMp/UJEkB7DBF1WtC6iJUKNiL+4uQuTNcofh80ZHnPiUro9Igbq4Do68jJ+uJsMW3W/02KiFP2eCTmJ2l5O14I+iQ4LZzszOibuoqGa8SQ+NeYMpXjf7f981NFLrj/zv0EmLo2DNACH/BQREORMVqxAe3lVNvHRg3TM2ypi4+EUQGzOD+hYVZ+Dv1aGBD4zaDwa2o438nAUDzTKDDLKV1jCa0yTFCAEbqm06h3g1f1kQ==";
        DvmObject ret = CheckCodeUtil.callStaticJniMethodObject(emulator,"decheckcode(Ljava/lang/String;)Ljava/lang/String;",str);
        System.out.println("\ncall decheckcode: " + ret.getValue().toString());
    }


```

调用一下：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rHx9cF7ibPcrvFLR8kLKcoKxribkTE3WGLGForSVBpoV42Yo2sEfIU6Gw/640?wx_fmt=png&from=appmsg)

发现结果并不正确，应该是环境补的有问题，这时候得向上排错了。

  
（3）补环境排错

  
好在 unidbg 很贴心的在控制台中打印了 JNI 的调用细节，可以看到最后一行 0x2c604 这个地址调用了一个 jni 函数之后，程序就结束了。ida 跳过去看看：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r77FVVk8lxfboYPEZqu5fLibl8CJPDZVM7aou5GZDwtiaGiaBDR8dyDb3w/640?wx_fmt=png&from=appmsg)

发现程序在走到 LABEL_71 这个代码块这里就直接退出了，按 x 看看这个代码块是从哪里被调用了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r1ytMI2vcmmXnJormw66SNIBwZia0U7Ta6CiaQ4JE7Np89ia6tdibKJ7V0w/640?wx_fmt=png&from=appmsg)

这里貌似是做了一个有关签名校验或者包名校验的东西，一旦有一个不匹配的就会跳转到 LABEL_71 代码块。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rM6ibPCrMuh5EVg99K4tALocHOg4l4cXeLXzejulfXkkcknarr4TsvicA/640?wx_fmt=png&from=appmsg)  

除此之外，这个地方的判断也会让程序的控制流走向 LABEL_71 代码块。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rRm8MnzicfaorzawSaQdofsTQHh23cyYCJibAVdd2SwLWosRuRYE7G9nw/640?wx_fmt=png&from=appmsg)

v6，v7 这两个参数在上面 sub_1B2F0 中有引用，进去看看。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rc9Yw2FezzvAP1yQZQPHBYBwJrNJMjk0PRdbXBeQpUoaGBzeNpdzLbA/640?wx_fmt=png&from=appmsg)  

又是长长的恶心人的控制流混淆：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rcBNO9KhV960YficpL6gVF736ra8VC6ibFF6pqXmFzyrY9ibJ98h6oo2Xg/640?wx_fmt=png&from=appmsg)

sub_1B2F0 这个函数大概就是取到当前应用的一个包名和签名，再看看 sub_1AB74 函数。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r7L9pw9YIXYvP4Jic0bQKYicZlQvm5tiaISnwz2b34QQwIDUYlw4UKuOBA/640?wx_fmt=png&from=appmsg)  

sub_1AB74 函数应该是做了一个文件的读取，进 shell cat 一下这个文件看看里面什么内容。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rJcmazxuXjwL9msoC0KKJYgqglgcoCu6eSDsHBUAXSXvr4BmuJddaIg/640?wx_fmt=png&from=appmsg)  

可以看到这个文件是储存了当前进程的包名，到这里我们就能理解为什么：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rUFU3qW0yvHB2BxVJmwQKgDpehmkiaiapnKd1EbcDcT2byqmXDibHMmQ2w/640?wx_fmt=png&from=appmsg)

unidbg 会在控制台抛出一条提示，序有进行文件读取的操作，写代码把这个文件访问补上：

```
    FileResult<AndroidFileIO> f1;
    //补文件访问
    public FileResult<AndroidFileIO> getF1(String pathname, int oflags) {
        if (f1 == null) {
            f1 = FileResult.<AndroidFileIO>success(new ByteArrayFileIO(oflags, pathname, "com.cloudy.linglingbang".getBytes()));
        }
        return f1;
    }

    @Override
    public FileResult<AndroidFileIO> resolve(Emulator<AndroidFileIO> emulator, String pathname, int oflags) {
        if ("/proc/self/cmdline".equals(pathname) || ("/proc/" + emulator.getPid() + "/cmdline").equals(pathname)) {
            return getF1(pathname, oflags);
        }

        return null;
    }


```

（4）再跑一下 decheckcode 函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rn6sXnQn6xeuswIgdFCibtHzibqkBBqK0OJlbpogydUYV2DKRmofhG5og/640?wx_fmt=png&from=appmsg)

发现没问题了，解密能解出来了。至此 unidbg 调用 checkcode 函数完成。

  

---

```
四





寻找DFA（差分故障攻击）攻击点

```

(1) 根据前面在 ida 中对 libencrypt.so 进行的静态分析，判断函数

WBACRAES_EncryptOneBlock 应该是整个白盒加密的关键部位，在 ida 中找到这个函数的地址 0x86F8 unidbg 下个断点看看入参。

```
emulator.attach().addBreakPoint(module.base + 0x86F8);


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rOicfX1a2S88pQGX3ISmgcKjYUZQsNbdViaxCufUeKKCIxU4AibYxevOpg/640?wx_fmt=png&from=appmsg)

unidbg 在 0x86F8 处断下，注意到 x0 和 x1 都是指针，在控制台输 mx0 和 mx1，看看这两个地址存的什么内容。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rbbgWiarF17VDkDHVXO1ibfv6elTOENnnJCrJMv7y5ofiaq4gg6rO3qeFw/640?wx_fmt=png&from=appmsg)

看起来 x0 处存的还是一个指针，根据 ida 中的分析来看，应该是 CWAESCipher 结构体的指针。

  
而 x1 存的是我们输入的明文 (为了方便分析 我把加密的明文改成了 aaaaa)

  
我们记住 x1 存的地址 0x40559020 这个地址在我们每次重新调用程序进行分析都是不变的 这也是 unidbg 在算法还原方面的一个优势所在。

  
（2）利用 unidbg 中的 emulator.traceRead api 追踪一下 0x40559020-0x40559030 这段存放了明文的地址，看看哪里对明文进行了读取。

```
emulator.traceRead(0x40559020,0x40559030);


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rJtFsiatLwPq2SldR6CxkXDrtUGKayRQZ75jadkmVicxpDvblnjtsjP9w/640?wx_fmt=png&from=appmsg)

这里对这段地址进行了十六次的读取，刚好对应了我们前面下断点读取到的明文，ida 跳到 0x7888 看看怎么个事。

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rSN1CHw5QgFClmqV4qfgVxqFBxK47IpXvbn9Q7Lyznd6tP35bjEiabIQ/640?wx_fmt=png&from=appmsg)  

这看起来好像是对明文进行了某种排序，我们 hook 下看看，在函数入口 0x7874 下个断点。看看 a3 的地址，在函数结束后读一下看看是什么内容。

```
emulator.attach().addBreakPoint(module.base + 0x7874);//PrepareAESMatrix start
emulator.attach().addBreakPoint(module.base + 0x7910);//PrepareAESMatrix over


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21ricH0YXn7BKuhvZv4HqPtf6BAzTfWtaqNylnpsPwYCOFSJUNIPo08oyw/640?wx_fmt=png&from=appmsg)

在 0x7874 处拿到 x2 寄存器的地址 0xbfffeb10 再让程序运行到函数尾部，看看这个地址存的内容变成什么样。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rSR7oN3Y1graRmY25kzpg1avGVTf6ibnBLZVbnhKeokp4FDazcLAfjTA/640?wx_fmt=png&from=appmsg)  

根据对这个地址内存的查看，我们知道了 PrepareAESMatrix 这个函数就是对明文进行排序，应该是我们 aes 加密中的 plaintext->state 阶段，我们再对这段地址进行 trace read。

```
emulator.traceRead(0xbfffeb10L,0xbfffeb30L);


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rjgVPADb4gqEzwia2HicxvysngdRVem4VblrPuC0I7r5armicg1w0rpEBQ/640?wx_fmt=png&from=appmsg)

ida 跳到 0x8c00 看看：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rxXmscxA0WicibPJvDy8kwJ7vyDKpmN85chwy7vcNY7RsjZphiaibqFuuOg/640?wx_fmt=png&from=appmsg)

根据数组符号和前面 state 转换传入的参数，确定了这一部分就是进行 aes 加密中的轮运算的地方，有几个 do..while 循环嵌套。

  
（3）走到这里，发现几个 do..while 循环的嵌套，单单静态分析还是很难看出哪个循环是单独走完了一轮加密，所以对几个 do..while 循环进行 hook，看看哪里是只走了 9 次（对应 aes 中的前九轮运算）。

```
public void StatisticalRound(){

        emulator.attach().addBreakPoint(module.base + 0x877C, new BreakPointCallback() {
            int add_0x877C = 0;
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                add_0x877C += 1;
                System.out.println("add_0x877C onHit:"+add_0x877C);
                return true;
            }
        });

        emulator.attach().addBreakPoint(module.base + 0x8BBC, new BreakPointCallback() {
            int add_0x8BBC = 0;
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                add_0x8BBC += 1;
                System.out.println("add_0x8BBC onHit:"+add_0x8BBC);
                return true;
            }
        });
        emulator.attach().addBreakPoint(module.base + 0x8BBC, new BreakPointCallback() {
            int add_0x8ABC = 0;
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                add_0x8ABC += 1;
                System.out.println("add_0x8ABC onHit:"+add_0x8ABC);
                return true;
            }
        });
        emulator.attach().addBreakPoint(module.base + 0x87A4, new BreakPointCallback() {
            int add_0x87A4 = 0;
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                add_0x87A4 += 1;
                System.out.println("add_0x87A4 onHit:"+add_0x87A4);
                return true;
            }
        });
    }


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r6ApUR5ib3DCWAOsRZOWo0icicY22jic1f3hk1Jg9LhkXCTTFUYXzNvnOqQ/640?wx_fmt=png&from=appmsg)

结果很明显 0x877C 就是一轮计算开始的位置，共循环了九次。

  
（4）前九轮计算循环的位置找到了。接下来就是要找第十轮计算的位置（因为 aes 加密中第十轮计算少了一个列混淆的步骤，所以程序应该有一个单独的代码块来进行第十轮计算）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r93n6ibVWQicoE3xe2AiaG3tvu9bIv3OXFNx0md7AKP5jQYW8t35QjjMVA/640?wx_fmt=png&from=appmsg)

运气很好，因为符号没有抹去，根据符号判断，这里进行了最后一轮计算，有三个控制流，都进行 hook 一下，最终确定最下面的控制流是最终轮计算。

  

---

```
五





开始攻击还原密钥

```

（1）上面我们找到了前九轮计算，每一轮计算的开始点 0x877C 且有了排序好的 state 的地址 0xbfffeb10，接下来就是要开始故障攻击了。

```
    public void dfaAttack(){
        emulator.attach().addBreakPoint(module.base + 0x877C, new BreakPointCallback() {
            int round = 0;
            UnidbgPointer statePointer = memory.pointer(0xbfffeb10L);
            @Override
            public boolean onHit(Emulator<?> emulator, long address) {
                round += 1;
                if (round % 9 == 0){
                    statePointer.setByte(0,(byte)randInt(0,15));//随机注入
                }
                return true;//返回true 就不会在控制台断住
            }
        });

    }
    public static int randInt(int min, int max) {
        Random rand = new Random();
        return rand.nextInt((max - min) + 1) + min;
    }


```

（2）调用一次看看结果

```
encode Results:    09 df ee c3 04 eb 14 ce 3c e2 94 68 9d 7d d4 1c
dfaAttack Results: 5a df ee c3 04 eb 14 b6 3c e2 c9 68 9d cf d4 1c


```

很明显，最终结果的第 1，8，11，14 个字节与原始加密的内容不同，符合 dfa 攻击成功的特征。

  
（3）多次攻击，取不同的故障密文

```
public static void main(String[] args) {
    CheckCodeUtil checkCodeUtil = new CheckCodeUtil();
    checkCodeUtil.dfaAttack();
    for (int i = 0; i < 30; i++) {
        checkCodeUtil.checkcode();
    }
}


```

（4）利用 python 的 phoenixAES 模块，对这些故障密文进行分析。

```
import phoenixAES

with open('tracefile', 'wb') as t:
    t.write("""09dfeec304eb14ce3ce294689d7dd41c #第一行放正确的密文
6ddfeec304eb146c3ce296689df8d41c
...
09dfeec304eb14ce3ce294689d7dd41c
""".encode('utf8'))

phoenixAES.crack_file('tracefile',[],True,False,3)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r1h6JDT63f6fo8j2XVRWuZePTcm2O1BoJpbZzibXR1eFQpfxnpRMJ2KQ/640?wx_fmt=png&from=appmsg)  

最终拿到了第十轮的密钥：8A6E30D74045AE83634D6ECDE1516CA1。

  
（5）计算原始密钥

  
GitHub - SideChannelMarvels/Stark: Repository of small utilities related to key recovery

  
用这个开源项目，根据轮密钥计算出原始密钥。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rEfPkgT0BhKxzzzEfyUp9EnBTiamDn0ccRUfcnpHwfHiapdBsTcT3UJEg/640?wx_fmt=png&from=appmsg)

最终拿到了我们的 key：F6F472F595B511EA9237685B35A8F866。

  
（6）拿到逆向之友里验证一下：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rhhT85ZrD9m8MRicjeppQt7MQmgMibgB72JQxSaq6IDMOypsQg36PAaPw/640?wx_fmt=png&from=appmsg)没毛病，是标准的 aes。

学逆向一年了，今天第一次写一篇完整的文章。样本难度不高，混淆不算太严重，部分符号没有抹去，有了攻击点。希望这篇文章能对正在学习移动安全的朋友有所帮助  
最后感谢 @白龙的公开文章，令我受益匪浅学到了不少东西。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rYMmkQfH50MP1UpY9Ciba7OLDFciaQOz4O7DHAxibiaYulzVqQKwKNzyF0w/640?wx_fmt=png&from=appmsg)

  

**看雪 ID：劫__**

https://bbs.kanxue.com/user-home-949812.htm

* 本文为看雪论坛优秀文章，由 劫__ 原创，转载请注明来自看雪社区

[![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8FBkdDbwInwy15Ke0oZbDj4IKY5oqgSDM8BR2aB3iaeql3ycjI6FZB7WYH4VpUUQPqB6lMFvapkCYQ/640?wx_fmt=png&from=appmsg)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542213&idx=3&sn=f1310c136b4a1f6193d6ce6463bcec0f&chksm=b18d500f86fad919ce1dde5d7f16c875767beaa98bf334c5806f313237b2808da23fef12f3f6&scene=21#wechat_redirect)

**#** **往期推荐**

1、[使用 Unidbg 在 CTF-Android 题目的快速解题](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542348&idx=1&sn=9aa386ce99973dad49852cdc5a545eb9&chksm=b18d518686fad890f0d5170fc110adafdd13e647ce700fc49de7ac1e9722166783765170fbc1&scene=21#wechat_redirect)

2、[安卓逆向 MagicImageViewer 技巧分享](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542235&idx=1&sn=874ea77470091dd3b83fe44845b23e01&chksm=b18d501186fad9072c62e49036fa4671738e1a54f3fd46264e534c1165ca4b826c2425a65a3a&scene=21#wechat_redirect)

3、[国赛 babytree 赛题解析](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542213&idx=1&sn=1e877cc93653a8e0f7711a3db0e3bea7&chksm=b18d500f86fad919d6a9fd9a603c9d604a97866a446aab68d16369bc640effb5666f82f92206&scene=21#wechat_redirect)

4、[堆利用详解：the house of storm](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542141&idx=1&sn=d487d648669ce3a5f8008eafb532bf99&chksm=b18d50b786fad9a1e6d4d6cabf9f43895206ecf62d5020cb64ee5bfa3600cbc6e6ab6ec08866&scene=21#wechat_redirect)

5、[Discord Stealer 样本分析](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542106&idx=1&sn=37fc28904c3ac3536f9f1baba60d67a2&chksm=b18d509086fad98615768d494d0cfc10a6561f193e337a8116893cfead73c3f0bbe02b51ff2a&scene=21#wechat_redirect)

6、[安卓逆向基础知识之安卓开发与逆向基础](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458542086&idx=1&sn=2af7d2e536458ddd7ae31d82cc59e9a0&chksm=b18d508c86fad99acc26cba9b623e1d00b698fdeb27be3137b68ce729a14f798b65db65ebed7&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r3nmRR5cHhfBCLIduNUibdGVcfq7zLianrfDuU5gf1LsfI40SpWem5OnA/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r3nmRR5cHhfBCLIduNUibdGVcfq7zLianrfDuU5gf1LsfI40SpWem5OnA/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21r3nmRR5cHhfBCLIduNUibdGVcfq7zLianrfDuU5gf1LsfI40SpWem5OnA/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8GeAmUDNQicicabOmpibqzq21rbAwZaWAvCrLIwBLzXJAibUZ3o1lQdXzhfBoChX5eXZK9OUO2rETazHA/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多