> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjA5NDE1OQ==&mid=2247484931&idx=1&sn=74582e0037883ae013790dc533abb64a&chksm=9f8bb723a8fc3e3514cd422bf5c47230742e58a223d9cd3b0f97e6470255ed4d4f7315dac2cd&mpshare=1&scene=1&srcid=0302Y5N5n4xB46iTyRX63oRx&sharer_sharetime=1646203557665&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

本来打算一篇写完的，发现太长，加上我拖延症严重，等到写好不知猴年马月，遂拆成 3 篇;

第一篇模拟执行，第二篇签名加密算法还原，第三篇 body 加密算法还原；

**以前的我**：

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3xBMR1a091Vb8Zxc7LcpA3Q6GDeLSF6NYbVPaAygKTuwSZh0Dk1IqPw/640?wx_fmt=jpeg)

**现在的我：**

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3wwia1uOgibv9l3txQaMYIUic4HpoefGKIy9d9xpGDgnBDcXmJEONBuNtQ/640?wx_fmt=jpeg)

**样本：**

```
aHR0cHM6Ly93d3cud2FuZG91amlhLmNvbS9hcHBzLzEyMzU5NjMvaGlzdG9yeV92MjQy

```

**抓包信息：**

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6rWzGly01TibGEvCI3EPqwl4VeVcCialdU5m6nwonvmdISic0j1uba4o2rG5FKkaSNv2iaQ6D9slsibpQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6rWzGly01TibGEvCI3EPqwlg7h0DMIz64LK2wBCPZmDVdCRtbO6KkuMzWvtibibLBvqicwfdRhPAFgyg/640?wx_fmt=png)

需要搞定的是签名和 body 加密;  

X-TJH：签名;  

Kf64g3......：请求 body;  

**jadx 定位参数；**

具体就不说了，很容易定位到在这里;

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6rWzGly01TibGEvCI3EPqwlCTTIK9URlgBkdPZh5xLbSTsqjJdSXFw8atDdrhQ2JiaQKexn8fvNpyQ/640?wx_fmt=png)

签名是 encrypt 函数生成的，body 是 bodyEncrypt 生成的;

**模拟调用**

先用 fida 主动调用一波（参数当然是 hook 这个函数得到的，但是被我截断了，方便后边算法分析）;

没毛病我们再来用 unidbg；

**frida-rpc encrypt:**

```
function call_encrypt(){
    Java.perform(function () {
        var Gundam = Java.use("com.tujia.gundam.Gundam");
        var StrCls = Java.use('java.lang.String');
        var arg0_str ="";
        var arg0 = StrCls.$new(arg0_str);
        var arg1_str = "Mozilla/5.0 (Linux; Android 8.1.0; OPPO R11st Build/OPM1.171019.011; wv)"
        var arg1 = StrCls.$new(arg1_str);
        var arg2_str = "LON=null;LAT=null;CID=226809753;LAC=42272;"
        var arg2 = StrCls.$new(arg2_str);
        var arg3_str = "{\"code\":null,\"parameter\":{\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":true,\"v\":\"C\"}}},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"\"}"
        var arg3 = StrCls.$new(arg3_str);
        var arg4 = arg3.getBytes().length;
        var arg5 = 1630845461;
        var ret = Gundam.encrypt(arg0,arg1,arg2,arg3,arg4,arg5);
        console.log("#encrypt ret:" + ret);
    } );
}

```

****frida-rpc** bodyEncrypt:**

```
function call_bodyEncrypt(){
    Java.perform(function () {
        var Gundam = Java.use("com.tujia.gundam.Gundam");
        var StrCls = Java.use('java.lang.String');
        var arg0 = '1';
        var arg1 = 1630808396;
        var arg2_str = "YM0A2TMAIEWA3xMAMM1xTjQEUYGD0wZAYAh32AdgQATDzAZNZA33WAEIZAzzhAMMNAzyTDAkZMzTizYOYN11TTgMRcDThyNMZO44GTIAVQGDi5OONN2wWGMMUVWj1iMNOYxxGmUU";
        var arg2 = StrCls.$new(arg2_str)
        var arg3 = arg2.length();
        var arg4_str = "{\"code\":null,\"parameter\":{\"activityTask\":{},\"defa"//'{"code":null,"parameter":{"activityTask":{},"defaultKeyword":"","abTest":{"AppSearchHouseList":"B","listabtest8":"B"},"returnNavigations":true,"returnFilterConditions":true,"specialKeyType":0,"returnGeoConditions":true,"abTests":{"T_login_831":{"s":true,"v":"A"},"searchhuojiatujia":{"s":true,"v":"D"},"listfilter_227":{"s":true,"v":"D"},"T_renshu_292":{"s":true,"v":"D"},"Tlisttest_45664":{"s":true,"v":"C"},"Tlist_168":{"s":true,"v":"C"},"T_LIST27620":{"s":false,"v":"A"}},"pageSize":10,"excludeUnitIdSet":null,"historyConditions":[],"searchKeyword":"","url":"","isDirectSearch":false,"sceneCondition":null,"returnAllConditions":true,"searchId":null,"pageIndex":0,"onlyReturnTotalCount":false,"conditions":[{"type":2,"value":"2021-09-05"},{"type":3,"value":"2021-09-06"},{"label":"广州","type":1,"value":"45"}]},"client":{"abTest":{},"abTests":{},"adTest":{"m1":"Color OS V5.2.1","m2":"ade40419318f085ff21b4776f2eef21f","m3":"armeabi-v7a","m4":"armeabi","m5":"100","m6":"2","m7":"2"},"api_level":260,"appFP":"qA/Ch2zqjORBz90YV34sUZpcFXFV6vzhmAISdTjYAFeqMBTtMUukzQFXqkDokr+sMau0bWClwjtk36nbrVBWVtKF9gYuplycneojVw1pIsBYHtUa5n0jFF50c6v1MBvzWJZFk/SirrTIzpJd4mMJ/J4F23BfgLbda7zzJhI8bpXMmglTI9M0hz5X3MMi5uS+/dt6ErjrDv44vY4Y8/1r5dahiujqI90q8/BWuNDxVqp2vGN5XILei7rc/JicoO6S+upWlrY/oAqkMhdmAbh1Anzjd/AZ7Q4HQubQJhsQNSL/T86A6uW0oC+mJHwOLnP8HKN0q2Fu3rTcKZ+Prbs/dcBHaWJi1C1tHZFza2O+1gUQTgvg+Kq57BvE6IjEhveT","appId":"com.tujia.hotel","appVersion":"260_260","appVersionUpdate":"rtag-20210803-183436-zhengyuan","batteryStatus":"full","buildTag":"rtag-20210803-183436-zhengyuan","buildVersion":"8.38.0","ccid":"51742042410923060391","channelCode":"qq","crnVersion":"254","devModel":"OPPO R11st","devToken":"18071adc03d2b69cdfa","devType":2,"dtt":"","electricity":"100","flutterPkgId":"277","gps":null,"kaTest":{"k1":"2_1_2","k2":"sdm660","k3":"ubuntu-16","k4":"R11st_11_A.43_200402","k5":"OPPO/R11st/R11s:8.1.0/OPM1.171019.011/1577198226:user/release-keys","k6":"R11st","k7":"OPM1.171019.011"},"latitude":"23.105868","locale":"zh-CN","longitude":"113.470269","networkType":"1","osVersion":"8.1.0","platform":"1","salt":"YM0A2TMAIEWA3xMAMM1xTjQEUYGD0wZAYAh32AdgQATDzAZNZA33WAEIZAzzhAMMNAzyTDAkZMzTizYOYN11TTgMRcDThyNMZO44GTIAVQGDi5OONN2wWGMMUVWj1iMNOYxxGmUU","screenInfo":"","sessionId":"a3e57f1c-03a8-3bc4-8709-fb36e7698ae4_1630806934108","tId":"21090509553913313246","tbTest":{"j1":"11d890e2","j2":"R11s","j3":"OPPO R11st","j4":"OPPO","j5":"OPPO","j6":"unknown","j7":"qcom","j8":"2.1.0  (ART)"},"traceid":"1630808396402_1630808396176_1630807074415","uID":"a3e57f1c-03a8-3bc4-8709-fb36e7698ae4","version":"260","wifi":null,"wifimac":"i5ZQ9aI14FDr9VMJ/ECIg7KAOE+Xev1/CrFoa53WLbE="},"psid":"d5a4fc67-5e67-47af-81e2-bac409860287","type":null,"user":null,"usid":null}'
        var arg4 = StrCls.$new(arg4_str)
        var arg5 =arg4.getBytes().length;
        var myRc = Gundam.bodyEncrypt(arg0,arg1,arg2,arg3,arg4,arg5);
        console.log("#bodyEncrypt ret:" + ret);
    } );
}

```

发现和 hook 的结果一致，那就来用 unidbg 模拟执行；

**unidbg** **encrypt:**

unidbg 补环境简单说一下（想要补的又快又好，还是得看龙哥得 csdn 和星球![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopscqPSEb8nUyhupqiagFTbwe1Rvzialiagk8fVXbjcYgtmbLfMVtHsicIJyg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopscqPSEb8nUyhupqiagFTbwe1Rvzialiagk8fVXbjcYgtmbLfMVtHsicIJyg/640?wx_fmt=png)）;

```
csdn:https://blog.csdn.net/qq_38851536
星球：https://t.zsxq.com/2F2J6au

```

首先先运行 JNI_OnLoad （如下代码第 40 行）  

```
package com.test;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.memory.MemoryBlock;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import java.io.File;
import java.io.FileNotFoundException;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;
public class Test extends AbstractJni {
        private final AndroidEmulator emulator;
        private final VM vm;
        private final Module module;
        private static final String APP_PACKAGE_NAME = "com.xxx.hotel";
    Test() {
            emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(APP_PACKAGE_NAME).build();
            final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
            memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
            vm = emulator.createDalvikVM(new File("xxx.apk")); // 创建Android虚拟机 报错用绝对路径
            vm.setVerbose(true); // 设置是否打印Jni调用细节
            vm.setJni(this);
            DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\tujia\\libxxx_encrypt.so"), true);//报错用绝对路径
            module = dm.getModule();
            dm.callJNI_OnLoad(emulator);
        }
    public static void main(String[] args) throws FileNotFoundException {
        Test test = new Test();
    }
}

```

运行发现报错;  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsObF0ZmKXxYsjPEKesqVYrMezBCibib4J0zx6vbZDBGemZnhUcFGEQe3w/640?wx_fmt=png)

为啥呢，因为 unidbg 没有实现这个类，我们得喂给它;

补充的代码逻辑可以写到 AbstractJni.java 类 367 行那里;

（截图别骂了后面有完整代码）

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopskAlH4tNbxhyROdU8IhFzwlYiajIv7Ugf1p3iatQAlalfO0CJEG8MyEmQ/640?wx_fmt=png)

也可以在我们的 Test 类中补，因为 Test 类继承了 AbstractJNI（上面的 26 行）;

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsov3w9z5QgkYFnYjdUD4zqYCedJLyYBooiaxyFxowwQjPDYDO2AYgXPA/640?wx_fmt=png)

这里为了方便演示，我都补到 Test 类中;

补完一个就运行一次，看报错来补;

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsibx8QUBY3oEAN8E6FVbeV7sK2v28NhBOJYAB1xXlE91VDh03RHysYUQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEops1GglC8XFgLEniaGy7hH4Dn9ZkYSzNmwbgHob8k8P7IFguEL4Im2Tplw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsb3KBr5Zv5hPl5rANLusg81usNDlsCkMsHbywm297MKsUicpjQs7uyaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsu0oJeDtaicYv5Q00MSFUKRON9LUHPXV0TKqEFHdrRBqbgOxGnyAiaJXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsGH5KILefJa6VibVjoicKQy28C59pHEQswYdzTpLCdscib3do5W9zGqxCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsSrns70YcaL7mNAECWGvOibr4ly12m6IX1YkwHdiaFc4Qibg0eUvklQWrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsUAHbSLbYWo4GaurdPEf16LjXHKSsgdVoTdib3FI69V2PBZabsPTicIMg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopscq5Yr2CO7hAgHicXLNmLS79pnZpcYyf19boUlz40ccJb1iccB6lZrJbA/640?wx_fmt=png)

补完这个 JNI_OnLoad 就能运行成功了;

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsZlCEhfwiaUht2XW98lQSfTWGnKxSmqzFMdt6CPXyMibqeeVTBNtG7SDw/640?wx_fmt=png)

看日志都知道它是在做签名校验咯；

最后附上 JNI_OnLoad 能成功运行的完整代码；

（友情提示：标红的类 ALT + 回车可以快速导入（不会有人不知道吧![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopscqPSEb8nUyhupqiagFTbwe1Rvzialiagk8fVXbjcYgtmbLfMVtHsicIJyg/640?wx_fmt=png)））；  

```
package com.tujia;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.memory.MemoryBlock;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import java.io.File;
import java.io.FileNotFoundException;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;
public class Test extends AbstractJni {
        private final AndroidEmulator emulator;
        private final VM vm;
        private final Module module;
        private static final String APP_PACKAGE_NAME = "com.tujia.hotel";
    Test() {
            emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(APP_PACKAGE_NAME).build();
            final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
            memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
            vm = emulator.createDalvikVM(new File("tujia.apk")); // 创建Android虚拟机
            vm.setVerbose(true); // 设置是否打印Jni调用细节
            vm.setJni(this);
            DalvikModule dm = vm.loadLibrary(new File("unidbg-master\\unidbg-android\\src\\test\\java\\com\\tujia\\libtujia_encrypt.so"), true);
            module = dm.getModule();
            dm.callJNI_OnLoad(emulator);
        }
    @Override
    public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
        switch (signature) {
            case "com/tujia/hotel/TuJiaApplication->getInstance()Lcom/tujia/hotel/TuJiaApplication;": {
                return vm.resolveClass("com/tujia/hotel/TuJiaApplication").newObject(signature);
            }
            case "java/security/MessageDigest->getInstance(Ljava/lang/String;)Ljava/security/MessageDigest;":{
                StringObject type = varArg.getObjectArg(0);
                System.out.println(type);
                String name = "";
                if ("\"SHA1\"".equals(type.toString())) {
                    name = "SHA1";
                } else {
                    name = type.toString();
                    System.out.println("else name: " + name);
                }
                try {
                    return vm.resolveClass("java/security/MessageDigest").newObject(MessageDigest.getInstance(name));
                } catch (NoSuchAlgorithmException e) {
                    e.printStackTrace();
                }
            }
        }
        return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);
    }
    public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
        switch (signature){
            case "com/tujia/hotel/TuJiaApplication->getPackageName()Ljava/lang/String;":{
                return new StringObject(vm,"com.tujia.hotel");
            }
            case "com/tujia/hotel/TuJiaApplication->getPackageManager()Landroid/content/pm/PackageManager;":{
                return vm.resolveClass("android/content/pm/PackageManager").newObject(signature);
            }
            case "java/security/MessageDigest->digest([B)[B":{
                MessageDigest messageDigest = (MessageDigest) dvmObject.getValue();
                byte[] input = (byte[]) varArg.getObjectArg(0).getValue();
                byte[] result = messageDigest.digest(input);
                return  new ByteArray(vm,result);
            }
        }
        return super.callObjectMethod(vm, dvmObject, signature, varArg);
    }
    public static void main(String[] args) throws FileNotFoundException {
        Test test = new Test();
    }
}

```

  
然后我们来主动调用 encrypt 函数；

之前呢，我都是这样调用的：

```
public void call_encrypt() {
        DvmClass Native=vm.resolveClass("com/tujia/gundam/Gundam");
        String methodSign = "encrypt(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;IJ)Ljava/lang/String;";        
        Object arg0 = vm.addLocalObject(new StringObject(vm, ""));
        String arg1_str = "Mozilla/5.0 (Linux; Android 8.1.0; OPPO R11st Build/OPM1.171019.011; wv)";
        Object arg1 = vm.addLocalObject(new StringObject(vm, arg1_str));
        String arg2_str = "LON=null;LAT=null;CID=226809753;LAC=42272;";
        Object arg2 = vm.addLocalObject(new StringObject(vm, arg2_str));
        String arg3_str ="{\"code\":null,\"parameter\":{\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":true,\"v\":\"C\"}}},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"\"}";
        Object arg3 = vm.addLocalObject(new StringObject(vm, arg3_str));
        int arg4 = arg3_str.getBytes(StandardCharsets.UTF_8).length;
        long arg5 = 1630845461L;
        Object ret = Native.callStaticJniMethodObject(emulator, methodSign, arg0,arg1,arg2,arg3,arg4,arg5);
        System.out.println("call_encrypt ret:"+((DvmObject) ret).getValue());
    }

```

```
String methodSign = "encrypt(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;IJ)Ljava/lang/String;";

```

这个是 smali 的写法 (先用 apktool 反编译 apk, 然后在 smail 文件夹里根据类名找）  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEopsCPlTGY8JQq4ibicasewhGxERtact0gOSkADYtTUx35McLOJC4oKZ8zDg/640?wx_fmt=png)

现在呢，在龙哥的熏陶下，都是这样调用;

```
public void call_encrypt() {
        List<Object> list = new ArrayList<>();
        DvmClass Native=vm.resolveClass("com/tujia/gundam/Gundam");
        DvmObject<?> jclass =Native.newObject(null);
        Object arg0 = vm.addLocalObject(new StringObject(vm, ""));
        String arg1_str = "Mozilla/5.0 (Linux; Android 8.1.0; OPPO R11st Build/OPM1.171019.011; wv)";
        Object arg1 = vm.addLocalObject(new StringObject(vm, arg1_str));
        String arg2_str = "LON=null;LAT=null;CID=226809753;LAC=42272;";
        Object arg2 = vm.addLocalObject(new StringObject(vm, arg2_str));
        String arg3_str ="{\"code\":null,\"parameter\":{\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":true,\"v\":\"C\"}}},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"\"}";
        Object arg3 = vm.addLocalObject(new StringObject(vm, arg3_str));
        int arg4 = arg3_str.getBytes(StandardCharsets.UTF_8).length;
        long arg5 = 1630845461L;
        list.add(vm.getJNIEnv()); //第一个参数都是JNIEnv
        list.add(vm.addLocalObject(jclass));//第一个参数都是jclass
        list.add(arg0);
        list.add(arg1);
        list.add(arg2);
        list.add(arg3);
        list.add(arg4);
        list.add(arg5);
        Number number = module.callFunction(emulator, 0x36a9, list.toArray())[0];
        String result = vm.getObject(number.intValue()).getValue().toString();
        System.out.println("result: " + result);
    }

```

22 行 0x36a9 是啥？

是 encrypt 函数的在 so 文件中的偏移地址;

哪里得到的？  

上面成功运行完 JNI_OnLoad，会显示动态注册的函数地址;  

```
RegisterNative(com/tujia/gundam/Gundam, encrypt(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;IJ)Ljava/lang/String;, RX@0x400036a9[libtujia_encrypt.so]0x36a9)
RegisterNative(com/tujia/gundam/Gundam, bodyEncrypt(Ljava/lang/String;JLjava/lang/String;ILjava/lang/String;I)Ljava/lang/String;, RX@0x4000380d[libtujia_encrypt.so]0x380d)

```

两种方法，结果都是一样的；任君选择；

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH51SmQ6OU118e9dtylhEops4ib0G7Epctgav9meAWKSPdJxjD2J4zFanKeOsP8JicNF7pN0bmNPa6FA/640?wx_fmt=png)

ps：如果还有人用的是老版本的 unidbg（比如在写这篇以前的我）;  

结果可能是下图的结果，应该是老版本第 6 个参数长整型参数有 bug，导致第六个参数是 0;  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5kiaGuPzfFvDLZIutvMYQc9ctWl52uicvKCkxPQUS4Zd6HGRYzjymHzibB6DwrMSDqnMXHQqpVSFuwg/640?wx_fmt=png)

解决办法：拥抱新版本![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5kiaGuPzfFvDLZIutvMYQc9goCicsYe6pF1dibKwVNzsU7Ud5OiaSeymDk4jI9To4Nw4r7HUN7AbFDnQ/640?wx_fmt=png);

```
https://github.com/zhkl0228/unidbg

```

bodyEncrypt 同理不再细说;

**完整代码:**

```
package com.tujia;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.memory.MemoryBlock;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import java.io.File;
import java.io.FileNotFoundException;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;
public class Test extends AbstractJni {
        private final AndroidEmulator emulator;
        private final VM vm;
        private final Module module;
        private static final String APP_PACKAGE_NAME = "com.tujia.hotel";
    Test() {
            emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(APP_PACKAGE_NAME).build();
            final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
            memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
            vm = emulator.createDalvikVM(new File("D:\\qxp\\Frida_hook\\Encrypt\\tujia.apk")); // 创建Android虚拟机
            vm.setVerbose(true); // 设置是否打印Jni调用细节
            vm.setJni(this);
            DalvikModule dm = vm.loadLibrary(new File("D:\\qxp\\unidbg-master\\unidbg-android\\src\\test\\java\\com\\tujia\\libtujia_encrypt.so"), true);
            module = dm.getModule();
        Debugger debugger = emulator.attach();
          //  debugger.addBreakPoint(module.base + 0x415C );
            dm.callJNI_OnLoad(emulator);
        }
    @Override
    public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
        switch (signature) {
            case "com/tujia/hotel/TuJiaApplication->getInstance()Lcom/tujia/hotel/TuJiaApplication;": {
                return vm.resolveClass("com/tujia/hotel/TuJiaApplication").newObject(signature);
            }
            case "java/security/MessageDigest->getInstance(Ljava/lang/String;)Ljava/security/MessageDigest;":{
                StringObject type = varArg.getObjectArg(0);
                System.out.println(type);
                String name = "";
                if ("\"SHA1\"".equals(type.toString())) {
                    name = "SHA1";
                } else {
                    name = type.toString();
                    System.out.println("else name: " + name);
                }
                try {
                    return vm.resolveClass("java/security/MessageDigest").newObject(MessageDigest.getInstance(name));
                } catch (NoSuchAlgorithmException e) {
                    e.printStackTrace();
                }
            }
        }
        return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);
    }
    public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
        switch (signature){
            case "com/tujia/hotel/TuJiaApplication->getPackageName()Ljava/lang/String;":{
                return new StringObject(vm,"com.tujia.hotel");
            }
            case "com/tujia/hotel/TuJiaApplication->getPackageManager()Landroid/content/pm/PackageManager;":{
                return vm.resolveClass("android/content/pm/PackageManager").newObject(signature);
            }
            case "java/security/MessageDigest->digest([B)[B":{
                MessageDigest messageDigest = (MessageDigest) dvmObject.getValue();
                byte[] input = (byte[]) varArg.getObjectArg(0).getValue();
                byte[] result = messageDigest.digest(input);
                return  new ByteArray(vm,result);
            }
        }
        return super.callObjectMethod(vm, dvmObject, signature, varArg);
    }
    public void call_encrypt() {
        List<Object> list = new ArrayList<>();
        DvmClass Native=vm.resolveClass("com/tujia/gundam/Gundam");
        DvmObject<?> jclass =Native.newObject(null);
        Object arg0 = vm.addLocalObject(new StringObject(vm, ""));
        String arg1_str = "Mozilla/5.0 (Linux; Android 8.1.0; OPPO R11st Build/OPM1.171019.011; wv)";
        Object arg1 = vm.addLocalObject(new StringObject(vm, arg1_str));
        String arg2_str = "LON=null;LAT=null;CID=226809753;LAC=42272;";
        Object arg2 = vm.addLocalObject(new StringObject(vm, arg2_str));
        String arg3_str ="{\"code\":null,\"parameter\":{\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":true,\"v\":\"C\"}}},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"\"}";
        Object arg3 = vm.addLocalObject(new StringObject(vm, arg3_str));
        int arg4 = arg3_str.getBytes(StandardCharsets.UTF_8).length;
        long arg5 = 1630845461L;
        list.add(vm.getJNIEnv());
        list.add(vm.addLocalObject(jclass));
        list.add(arg0);
        list.add(arg1);
        list.add(arg2);
        list.add(arg3);
        list.add(arg4);
        list.add(arg5);
        Number number = module.callFunction(emulator, 0x36a9, list.toArray())[0];
        String result = vm.getObject(number.intValue()).getValue().toString();
        System.out.println("result: " + result);
//        DvmClass Native=vm.resolveClass("com/tujia/gundam/Gundam");
//        String methodSign = "encrypt(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;IJ)Ljava/lang/String;";
//        Object arg0 = vm.addLocalObject(new StringObject(vm, ""));
//        String arg1_str = "Mozilla/5.0 (Linux; Android 8.1.0; OPPO R11st Build/OPM1.171019.011; wv)";
//        Object arg1 = vm.addLocalObject(new StringObject(vm, arg1_str));
//        String arg2_str = "LON=null;LAT=null;CID=226809753;LAC=42272;";
//        Object arg2 = vm.addLocalObject(new StringObject(vm, arg2_str));
//        String arg3_str ="{\"code\":null,\"parameter\":{\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":true,\"v\":\"C\"}}},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"\"}";
//        Object arg3 = vm.addLocalObject(new StringObject(vm, arg3_str));
//        int arg4 = arg3_str.getBytes(StandardCharsets.UTF_8).length;
//        long arg5 = 1630845461L;
//        Object ret = Native.callStaticJniMethodObject(emulator, methodSign, arg0,arg1,arg2,arg3,arg4,arg5);
//        System.out.println("call_encrypt ret:"+((DvmObject) ret).getValue());
    }
    public void get_bodyencrypt() throws FileNotFoundException {
        List<Object> list = new ArrayList<>();
        DvmClass Native=vm.resolveClass("com/tujia/gundam/Gundam");
        DvmObject<?> jclass =Native.newObject(null);
        String arg0_str = "1";
        Object arg0 = vm.addLocalObject(new StringObject(vm, arg0_str));
        long arg1 = 1630808396L;
        String arg2_str = "YM0A2TMAIEWA3xMAMM1xTjQEUYGD0wZAYAh32AdgQATDzAZNZA33WAEIZAzzhAMMNAzyTDAkZMzTizYOYN11TTgMRcDThyNMZO44GTIAVQGDi5OONN2wWGMMUVWj1iMNOYxxGmUU";
        Object arg2 = vm.addLocalObject(new StringObject(vm, arg2_str));
        int arg3 = arg2_str.getBytes(StandardCharsets.UTF_8).length;
        String arg4_str = "{\"code\":null,\"parameter\":{\"activityTask\":{},\"defa";//"{\"code\":null,\"parameter\":{\"activityTask\":{},\"defaultKeyword\":\"\",\"abTest\":{\"AppSearchHouseList\":\"B\",\"listabtest8\":\"B\"},\"returnNavigations\":true,\"returnFilterConditions\":true,\"specialKeyType\":0,\"returnGeoConditions\":true,\"abTests\":{\"T_login_831\":{\"s\":true,\"v\":\"A\"},\"searchhuojiatujia\":{\"s\":true,\"v\":\"D\"},\"listfilter_227\":{\"s\":true,\"v\":\"D\"},\"T_renshu_292\":{\"s\":true,\"v\":\"D\"},\"Tlisttest_45664\":{\"s\":true,\"v\":\"C\"},\"Tlist_168\":{\"s\":true,\"v\":\"C\"},\"T_LIST27620\":{\"s\":false,\"v\":\"A\"}},\"pageSize\":10,\"excludeUnitIdSet\":null,\"historyConditions\":[],\"searchKeyword\":\"\",\"url\":\"\",\"isDirectSearch\":false,\"sceneCondition\":null,\"returnAllConditions\":true,\"searchId\":null,\"pageIndex\":0,\"onlyReturnTotalCount\":false,\"conditions\":[{\"type\":2,\"value\":\"2021-09-05\"},{\"type\":3,\"value\":\"2021-09-06\"},{\"label\":\"大理州\",\"type\":1,\"value\":\"36\"}]},\"client\":{\"abTest\":{},\"abTests\":{},\"adTest\":{\"m1\":\"Color OS V5.2.1\",\"m2\":\"ade40419318f085ff21b4776f2eef21f\",\"m3\":\"armeabi-v7a\",\"m4\":\"armeabi\",\"m5\":\"100\",\"m6\":\"2\",\"m7\":\"5\"},\"api_level\":260,\"appFP\":\"qA/Ch2zqjORBz90YV34sUZpcFXFV6vzhmAISdTjYAFeqMBTtMUukzQFXqkDokr+sMau0bWClwjtk36nbrVBWVrjmrPTCkXFIraNHdgRVW/QT6g4eLWuM3hhP8qsWgGnrErk2KA+GFxr/OBRMYfV4l0v+TYUDZ5k4bUCUawafdLY5b3aC02SuOrqjW3jjrXiB/dt6ErjrDv44vY4Y8/1r5Z6ut/2BmcErxM37MniKpW6EZc8F4CjJ9S1KRTtEPJ2Kkd2Sd8602jqdgtssJ6QKXyx2+qsKvybydVe+zSTXQGn/T86A6uW0oC+mJHwOLnP8HKN0q2Fu3rTcKZ+Prbs/dcBHaWJi1C1tHZFza2O+1gUQTgvg+Kq57BvE6IjEhveT\",\"appId\":\"com.tujia.hotel\",\"appVersion\":\"260_260\",\"appVersionUpdate\":\"rtag-20210803-183436-zhengyuan\",\"batteryStatus\":\"full\",\"buildTag\":\"rtag-20210803-183436-zhengyuan\",\"buildVersion\":\"8.38.0\",\"ccid\":\"51742042410923060391\",\"channelCode\":\"qq\",\"crnVersion\":\"254\",\"devModel\":\"OPPO R11st\",\"devToken\":\"\",\"devType\":2,\"dtt\":\"\",\"electricity\":\"100\",\"flutterPkgId\":\"277\",\"gps\":null,\"kaTest\":{\"k1\":\"2_1_2\",\"k2\":\"sdm660\",\"k3\":\"ubuntu-16\",\"k4\":\"R11st_11_A.43_200402\",\"k5\":\"OPPO/R11st/R11s:8.1.0/OPM1.171019.011/1577198226:user/release-keys\",\"k6\":\"R11st\",\"k7\":\"OPM1.171019.011\"},\"latitude\":\"23.105714\",\"locale\":\"zh-CN\",\"longitude\":\"113.470271\",\"networkType\":\"1\",\"osVersion\":\"8.1.0\",\"platform\":\"1\",\"salt\":\"ZMmADTBAMIzAzxMAYMkxWjVEFY2TkwNMZAh2WAdAFATzmAZMYA3wTAEckAzT2AMMMAz52DAIZMzThzYMMN1xWTgYMcDD0yNNMO41zTIQUQGDx5OOZN2wDGMMEVWjziMNZYxxDmUA\",\"screenInfo\":\"\",\"sessionId\":\"a3e57f1c-03a8-3bc4-8709-fb36e7698ae4_1630844995950\",\"tId\":\"21090509553913313246\",\"tbTest\":{\"j1\":\"11d890e2\",\"j2\":\"R11s\",\"j3\":\"OPPO R11st\",\"j4\":\"OPPO\",\"j5\":\"OPPO\",\"j6\":\"unknown\",\"j7\":\"qcom\",\"j8\":\"2.1.0  (ART)\"},\"traceid\":\"1630845462612_1630845462444_1630845358749\",\"uID\":\"a3e57f1c-03a8-3bc4-8709-fb36e7698ae4\",\"version\":\"260\",\"wifi\":null,\"wifimac\":\"i5ZQ9aI14FDr9VMJ/ECIg7KAOE+Xev1/CrFoa53WLbE=\"},\"psid\":\"76938405-a463-464a-9c49-752022daf516\",\"type\":null,\"user\":null,\"usid\":null}";
        Object arg4 = vm.addLocalObject(new StringObject(vm, arg4_str));
        int arg5 = arg4_str.getBytes(StandardCharsets.UTF_8).length;
        list.add(vm.getJNIEnv());
        list.add(vm.addLocalObject(jclass));
        list.add(arg0);
        list.add(arg1);
        list.add(arg2);
        list.add(arg3);
        list.add(arg4);
        list.add(arg5);
        Number number = module.callFunction(emulator, 0x380c+1, list.toArray())[0];
        String result = vm.getObject(number.intValue()).getValue().toString();
        System.out.println("result: " + result);
    }
    public static void main(String[] args) throws FileNotFoundException {
        Test test = new Test();
        test.call_encrypt();
        test.get_bodyencrypt();
    }
}

```

**算法还原**

**encrypt:**

签名是 40 位长度的 16 进制字符串，很快想到是 sha1；让我们来 see see 是不是;

把 so 文件扔进 ida；按 g 跳到 0x36a9（为什么搜这个？划上去再看一遍）;

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3F74kVDHBesTjOWlJmbibvND4tdXAsxnvmRQgnPgWjpwYhQE22KJGuug/640?wx_fmt=png)

静态分析一波;  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3u79uLibZOrrqVVDSYfQfdaMBRorAmAEYicHgluA26pL9zxMWQxBYzSDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3uS8r51ibxIxErlYiaIrLlwo8GPWQJUlT21zTrynrRRIFNKhYSQqGKgnA/640?wx_fmt=png)

EncodeHTTP 函数好长, 我们从下往上看;

欲知后事如何  

请看下回分解