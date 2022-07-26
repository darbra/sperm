> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/S_3tM_TbOfbBRprmQGDwrA)

****一、a2 流程分析：****

```
package com.candy;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.HookStatus;
import com.github.unidbg.arm.backend.Backend;
import com.github.unidbg.arm.backend.CodeHook;
import com.github.unidbg.arm.context.Arm32RegisterContext;
import com.github.unidbg.arm.context.RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.hook.ReplaceCallback;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.hook.xhook.IxHook;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.XHookImpl;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ArrayObject;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.linux.android.dvm.wrapper.DvmInteger;
import com.github.unidbg.linux.file.ByteArrayFileIO;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.memory.MemoryBlock;
import com.github.unidbg.pointer.UnidbgPointer;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import unicorn.ArmConst;
import unicorn.Unicorn;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.PrintStream;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
public class Meituan15 extends AbstractJni implements IOResolver {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
                                                      改一下啊
    private static final String APP_PACKAGE_NAME = "com.脱敏.com";
    Meituan15(){
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(APP_PACKAGE_NAME).build();
        final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
        memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/apk/11.9.405.apk")); // 创建Android虚拟机
        vm.setVerbose(true); // 设置是否打印Jni调用细节
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\candy\\libmtguard1.5.so"), true);
        emulator.getSyscallHandler().addIOResolver(this);
        module = dm.getModule(); //
        vm.setJni(this);
        dm.callJNI_OnLoad(emulator);
    }
    public static void main(String[] args) throws FileNotFoundException {
        Meituan15 test = new Meituan15();
        test.init();
        test.call_main();
    }
    public String call_main() throws FileNotFoundException {
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
        list.add(203);
        StringObject input2_1 = new StringObject(vm, "9b69f861-e054-4bc4-9daf-d36ae205ed3e");
        //之前的抓包参数里有个人隐私- -，所以就改成1111111了， 影响分析的话，call me
        ByteArray input2_2 = new ByteArray(vm,"POST /mtapi/v8/poi/food __reqTra11111111111111111111111111111".getBytes(StandardCharsets.UTF_8));
        DvmInteger input2_3 = DvmInteger.valueOf(vm, 2);
        vm.addLocalObject(input2_1);
        vm.addLocalObject(input2_2);
        vm.addLocalObject(input2_3);
        list.add(vm.addLocalObject(new ArrayObject(input2_1, input2_2, input2_3)));
        Number number = module.callFunction(emulator, 0x5a38d, list.toArray())[0];
        StringObject result = (StringObject) ((DvmObject[])((ArrayObject)vm.getObject(number.intValue())).getValue())[0];
        System.out.println(result.getValue());
        return result.getValue();
    };
    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
        改一下啊
        if (("/data/app/com.脱敏.meituan-TEfTAIBttUmUzuVbwRK1DQ==/base.apk").equals(pathname)) {
            return FileResult.success(new SimpleFileIO(oflags, new File("unidbg-android\\src\\test\\java\\com\\candy\\base.apk"), pathname));
        }
        if ("/proc/self/cmdline".equals(pathname)) {
            return FileResult.success(new ByteArrayFileIO(oflags, pathname, "com.sankuai.meituan".getBytes()));
        }
        return null;
    }
    public void init(){
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv());
        list.add(0);
        list.add(111);
        DvmObject<?> obj = vm.resolveClass("java/lang/object").newObject(null);
        vm.addLocalObject(obj);
        ArrayObject myobject = new ArrayObject(obj);
        vm.addLocalObject(myobject);
        list.add(vm.addLocalObject(myobject));
        module.callFunction(emulator, 0x5a38d, list.toArray());
    };
    @Override
    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "com/meituan/android/common/mtguard/NBridge->getPicName()Ljava/lang/String;":{
                return new StringObject(vm, "ms_com.sankuai.meituan");
            }
            case "com/meituan/android/common/mtguard/NBridge->getSecName()Ljava/lang/String;":{
                return new StringObject(vm, "ppd_com.sankuai.meituan.xbt");
            }
            case "com/meituan/android/common/mtguard/NBridge->getAppContext()Landroid/content/Context;":{
                return vm.resolveClass("android/content/Context").newObject(null);
            }
            case "com/meituan/android/common/mtguard/NBridge->getMtgVN()Ljava/lang/String;": {
                return new StringObject(vm, "4.4.7.3");
            }
            case "com/meituan/android/common/mtguard/NBridge->getDfpId()Ljava/lang/String;": {
                return new StringObject(vm, "189d738852b0190bbdb8c13f1add56da56eff5f83b11e8be1aff77c1");
            }
        }
        return super.callStaticObjectMethodV(vm, dvmClass, signature,vaList);
    }
    @Override
    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature) {
            case "android/content/Context->getPackageCodePath()Ljava/lang/String;":{
                return new StringObject(vm, "/data/app/com.sankuai.meituan-TEfTAIBttUmUzuVbwRK1DQ==/base.apk");
            }
        }
        return super.callObjectMethodV(vm, dvmObject, signature, vaList);
    }
    @Override
    public int getIntField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
        switch (signature){
            case "android/content/pm/PackageInfo->versionCode:I":{
                return 1100090405;
            }
            case "android/content/pm/PackageManager->GET_SIGNATURES:I":{
                return 1100090405;
            }
        };
        return super.getIntField(vm, dvmObject, signature);
    }
    @Override
    public int getStaticIntField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature){
            case "android/content/pm/PackageInfo->versionCode:I":{
                return 1100090405;
            }
            case "android/content/pm/PackageManager->GET_SIGNATURES:I":{
                return 1100090405;
            }
        };
            return super.getStaticIntField(vm, dvmClass, signature);
    }
    public static byte[] hexStr2bytes(String hexStr) {
        if(hexStr.length()%2 != 0) {//长度为单数
            hexStr = "0" + hexStr;//前面补0
        }
        char[] chars = hexStr.toCharArray();
        int len = chars.length/2;
        byte[] bytes = new byte[len];
        for (int i = 0; i < len; i++) {
            int x = i*2;
            bytes[i] = (byte)Integer.parseInt(String.valueOf(new char[]{chars[x], chars[x+1]}), 16);
        }
        return bytes;
    }
    @Override
    public DvmObject<?> newObjectV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature) {
            case "java/lang/Integer-><init>(I)V":
                int input = vaList.getIntArg(0);
                return new DvmInteger(vm, input);
        }
        return super.newObjectV(vm, dvmClass, signature, vaList);
    }
}

```

先从标准算法入手

ida 算法识别插件：Findcrypt3

```
https://github.com/polymorf/findcrypt-yara
将findcrypt3.py和findcrypt3.rules放到IDA/plugins目录下

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahCdNhq1HOpFIkqyppIs9wUW7an0ibFTfzbdhY3Y4Tcia3jmWxx4BXtT3w/640?wx_fmt=png)

a4 是长度 40 的 hex 字符串，猜想到 sha1 哈希编码，hook sha1 的地址

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahSiagH8PNK2nNwAP85UxzibUmPgHt4A6IL1KhDAPqqAFb0mrMyg1Nlbibw/640?wx_fmt=png)

用 unidbg 的 ConsoleDebugger 来验证走了哪个函数，

```
Debugger debugger = emulator.attach();
debugger.addBreakPoint(module.base+0x95E5C);

```

发现走了 sub_95E5C 函数

查看寄存器 r0 的值  

发现了 hmca 的特征

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahZqUUzphHWQgv2icFNQhQiaD2TYmlutTwhACSxLHnGzdqgovh3HvMK4icw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahRXzjXhnjhQu8PRZZEqIveh0ISkm4kRKeMJUZ588EzLVH2myokzIKYA/640?wx_fmt=png)

第二次

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahrxG9P6JRQRicKCyFm5nAicy97ziclJHzD9MFbatGsuwH7aMkN8qEweCXA/640?wx_fmt=png)

查看上层函数

```
int __fastcall sub_95D48(int a1, int a2, int a3, int a4, int a5)
{
  int result; // r0
  int i; // r0
  char v8; // r1
  int v9; // r4
  char *v10; // r5
  char v14[88]; // [sp+14h] [bp-14Ch] BYREF
  char v15[24]; // [sp+6Ch] [bp-F4h] BYREF
  char v16[68]; // [sp+84h] [bp-DCh] BYREF
  char v17[68]; // [sp+C8h] [bp-98h] BYREF
  char v18[68]; // [sp+10Ch] [bp-54h] BYREF
  result = 0;
  if ( a1 && a3 && a4 )
  {
    memset(v18, 0, 0x41u);
    sub_9C5CA(v18, a1, a2);
    memset(v17, 0, 0x41u);
    memset(v16, 0, 0x41u);
    for ( i = 0; i != 64; ++i )
    {
      v8 = v18[i];
      v17[i] = v8 ^ 0x5C;
      v16[i] = v8 ^ 0x36;
    }
    v9 = a4 + 64;
    v10 = (char *)malloc(a4 + 65);
    result = 0;
    if ( v10 )
    {
      sub_9C580(v10, 0, v9);
      sub_9C5CA(v10, v16, 64);
      sub_9C5CA(v10 + 64, a3, a4);
      memset(v15, 0, 0x15u);
      sub_95E5C((int)v10, v9, (int)v15);
      memset(v14, 0, 0x55u);
      sub_9C5CA(v14, v17, 64);
      sub_9C5CA(&v14[64], v15, 20);
      sub_95E5C((int)v14, 84, a5);
      free(v10);
      return 1;
    }
  }
  return result;
}

```

典型的 hamcsha1

hook

```
debugger.addBreakPoint(module.base+0x95D48);

```

key  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahBt1Wgmxra881kPDguzwYibk7W52aLkQQega2nzFH9NkNEVonuJRpOuA/640?wx_fmt=png)

明文：  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiah0ia58IZH8sbWFr3heHJk8V6mTfbyDICCIMQdfZ8vrKFhndK7Tb7d4hw/640?wx_fmt=png)

这里的明文就是 arg1 的第二个元素去掉了前边的 32 长度字符串 (POST /mtapi/v8/poi/food __reqTra)

blr（函数结束后下个断点）

验证结果：

```
https://gchq.github.io/CyberChef

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiah7bedJIXLv6hZl0HNTJaHdVoQq1XT2YOZibk84wd7taAe89RWickPAvUQ/640?wx_fmt=png)

所以是标准的 hmacsha1

但是发现明文一变，key 也会随之改变

改一下输入

```
//之前的抓包参数里有个人隐私- -，所以就改成1111111了
ByteArray input2_2 = new ByteArray(vm,"POST /mtapi/v8/poi/food __reqTra22221111111111111111111111111".getBytes(StandardCharsets.UTF_8));

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4avV0NsU6Z2zbpa0feNfgK2WZZrI8HrwXpXyVKlKOREzBvBS835njgh5DzWZGZUsSsXNxr3JMPAw/640?wx_fmt=png)

所以 a2 是由标准的 hmacsha1 算法计算而来，但是 key 不是固定的，jave 层不同的入参会计算出不同的 key  

按 c 会发现会在 hmac 处再次断住，刚好 d1 也是 40 长度的 hex 字符串

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4l5qU0zWDVzf8dPBvFwcGoPBuSjhJvWJKudb1MwluxX0MAfzfyefiaDQ/640?wx_fmt=png)

key

```
mr0 0x24
>-----------------------------------------------------------------------------<
[19:20:14 394]r0=RW@0x4031a2d0, md5=ca286f49561ff88253a2a4ba04c987f3, hex=7450575028557b0144554077055451312c02150030385f7e267105121f00686f3c57421f
size: 36
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.

```

明文  

```
mr2 0x1ca
>-----------------------------------------------------------------------------<
[19:20:31 290]r2=RW@0x403ba20c, md5=0c0fa436dec520286a4e484ece6c53d3, hex=7b226130223a22312e35222c226131223a2239623639663836312d653035342d346263342d396461662d643336616532303565643365222c226132223a2233636364646233343031633464346135313863323161346232343032356330343766346463323162222c226133223a322c226134223a313635363733333138372c226135223a22736b49523142504e5a5037784e7346617164686868336174776b3271706f756d384b316246473058594b734164434e36574356326d76452b384f4b684651642b58444a334f30336a2f2b75644b385470754658466f4f7772576e712f516c334751746136536e584e574e584f4f446b66506c43536a504c62524d584f5656516f4d4d765453464952736f4f3935464b5970366b634d674478345954716e6345574f5a373246335463546e64356d4a59694c546a6d67366567564841527956786e583843314862416646534f2b4b31713256396a33687376733576754d38702b566542427a687331416f36307a365a59584f37672b764365684873586641593375344f7772755a67466954676a4831386a4a504238784e756761527a3334383550755937567433695a4257497a5a416e7a5533436a6c384a2b5034453d222c226136223a307d
size: 458
0000: 7B 22 61 30 22 3A 22 31 2E 35 22 2C 22 61 31 22    {"a0":"1.5","a1"
0010: 3A 22 39 62 36 39 66 38 36 31 2D 65 30 35 34 2D    :"9b69f861-e054-
0020: 34 62 63 34 2D 39 64 61 66 2D 64 33 36 61 65 32    4bc4-9daf-d36ae2
0030: 30 35 65 64 33 65 22 2C 22 61 32 22 3A 22 33 63    05ed3e","a2":"3c
0040: 63 64 64 62 33 34 30 31 63 34 64 34 61 35 31 38    cddb3401c4d4a518
0050: 63 32 31 61 34 62 32 34 30 32 35 63 30 34 37 66    c21a4b24025c047f
0060: 34 64 63 32 31 62 22 2C 22 61 33 22 3A 32 2C 22    4dc21b","a3":2,"
0070: 61 34 22 3A 31 36 35 36 37 33 33 31 38 37 2C 22    a4":1656733187,"
0080: 61 35 22 3A 22 73 6B 49 52 31 42 50 4E 5A 50 37    a5":"skIR1BPNZP7
0090: 78 4E 73 46 61 71 64 68 68 68 33 61 74 77 6B 32    xNsFaqdhhh3atwk2
00A0: 71 70 6F 75 6D 38 4B 31 62 46 47 30 58 59 4B 73    qpoum8K1bFG0XYKs
00B0: 41 64 43 4E 36 57 43 56 32 6D 76 45 2B 38 4F 4B    AdCN6WCV2mvE+8OK
00C0: 68 46 51 64 2B 58 44 4A 33 4F 30 33 6A 2F 2B 75    hFQd+XDJ3O03j/+u
00D0: 64 4B 38 54 70 75 46 58 46 6F 4F 77 72 57 6E 71    dK8TpuFXFoOwrWnq
00E0: 2F 51 6C 33 47 51 74 61 36 53 6E 58 4E 57 4E 58    /Ql3GQta6SnXNWNX
00F0: 4F 4F 44 6B 66 50 6C 43 53 6A 50 4C 62 52 4D 58    OODkfPlCSjPLbRMX
0100: 4F 56 56 51 6F 4D 4D 76 54 53 46 49 52 73 6F 4F    OVVQoMMvTSFIRsoO
0110: 39 35 46 4B 59 70 36 6B 63 4D 67 44 78 34 59 54    95FKYp6kcMgDx4YT
0120: 71 6E 63 45 57 4F 5A 37 32 46 33 54 63 54 6E 64    qncEWOZ72F3TcTnd
0130: 35 6D 4A 59 69 4C 54 6A 6D 67 36 65 67 56 48 41    5mJYiLTjmg6egVHA
0140: 52 79 56 78 6E 58 38 43 31 48 62 41 66 46 53 4F    RyVxnX8C1HbAfFSO
0150: 2B 4B 31 71 32 56 39 6A 33 68 73 76 73 35 76 75    +K1q2V9j3hsvs5vu
0160: 4D 38 70 2B 56 65 42 42 7A 68 73 31 41 6F 36 30    M8p+VeBBzhs1Ao60
0170: 7A 36 5A 59 58 4F 37 67 2B 76 43 65 68 48 73 58    z6ZYXO7g+vCehHsX
0180: 66 41 59 33 75 34 4F 77 72 75 5A 67 46 69 54 67    fAY3u4OwruZgFiTg
0190: 6A 48 31 38 6A 4A 50 42 38 78 4E 75 67 61 52 7A    jH18jJPB8xNugaRz
01A0: 33 34 38 35 50 75 59 37 56 74 33 69 5A 42 57 49    3485PuY7Vt3iZBWI
01B0: 7A 5A 41 6E 7A 55 33 43 6A 6C 38 4A 2B 50 34 45    zZAnzU3Cjl8J+P4E
01C0: 3D 22 2C 22 61 36 22 3A 30 7D                      =","a6":0}

```

结果  

```
m0xbffff670 0x14
>-----------------------------------------------------------------------------<
[19:22:37 448]unidbg@0xbffff670, md5=c5027a7e769cb70f4c2849e4f2c9018f, hex=b9cb092e87c050d78fbff5630fcb92b910fba01f
size: 20
0000: B9 CB 09 2E 87 C0 50 D7 8F BF F5 63 0F CB 92 B9    ......P....c....
0010: 10 FB A0 1F

```

key 是不变的

明文是 a1-a6 组装 json 字符串

```
{"a0":"1.5","a1":"9b69f861-e054-4bc4-9daf-d36ae205ed3e","a2":"3ccddb3401c4d4a518c21a4b24025c047f4dc21b","a3":2,"a4":1656733187,"a5":"skIR1BPNZP7xNsFaqdhhh3atwk2qpoum8K1bFG0XYKsAdCN6WCV2mvE+8OKhFQd+XDJ3O03j/+udK8TpuFXFoOwrWnq/Ql3GQta6SnXNWNXOODkfPlCSjPLbRMXOVVQoMMvTSFIRsoO95FKYp6kcMgDx4YTqncEWOZ72F3TcTnd5mJYiLTjmg6egVHARyVxnX8C1HbAfFSO+K1q2V9j3hsvs5vuM8p+VeBBzhs1Ao60z6ZYXO7g+vCehHsXfAY3u4OwruZgFiTgjH18jJPB8xNugaRz3485PuY7Vt3iZBWIzZAnzU3Cjl8J+P4E=","a6":0}

```

结果就是 d1  

```
"d1":"b9cb092e87c050d78fbff5630fcb92b910fba01f"

```

看起来 d1 里面只有 a2 和 a5 需要分析  

接下来先分析 a2 动态 key

        1.a2 动态 key 的流程

动态 key 长这样

```
debugger.addBreakPoint(module.base+0x95D48);

```

```
00000000: 31 FA 79 C0 69 86 37 AF  F9 B3 06 12 04 79 F0 A1  1.y.i.7......y..
00000010: F6 5A 50 78 42 36 79 1F  10 1B 2F B4 49 AE AC 93  .ZPxB6y.../.I...

```

traceWrite 监控 r0 寄存器的写入

```
emulator.traceWrite(0x40312040L,0x40312040L+0x20L);

```

```
### Memory WRITE at 0x40312040, data size = 1, data value = 0x31 pc=RX@0x4005ced8[libmtguard.so]0x5ced8 lr=RX@0x400c2db7[libmtguard.so]0xc2db7
### Memory WRITE at 0x40312041, data size = 1, data value = 0xfa pc=RX@0x4005ced8[libmtguard.so]0x5ced8 lr=RX@0x400c2db7[libmtguard.so]0xc2db7
### Memory WRITE at 0x40312042, data size = 1, data value = 0x79 pc=RX@0x4005ced8[libmtguard.so]0x5ced8 lr=RX@0x400c2db7[libmtguard.so]0xc2db7
### Memory WRITE at 0x40312043, data size = 1, data value = 0xc0 pc=RX@0x4005ced8[libmtguard.so]0x5ced8 lr=RX@0x400c2db7[libmtguard.so]0xc2db7

```

  
ida 跳到 0x5ced8

```
_BYTE *__fastcall sub_5CE60(int a1, size_t byte_count, int a3, size_t a4)
{
  size_t v5; // r6
  int v6; // r4
  _BYTE *v7; // r0
  int v8; // r4
  size_t v9; // r5
  char v10; // r6
  int v11; // r1
  char v12; // r0
  int v13; // r4
  int v14; // r1
  _BYTE *v16; // [sp+4h] [bp-1Ch]
  size_t v18; // [sp+Ch] [bp-14h]
  v5 = byte_count;
  v6 = 0;
  if ( a4 )
  {
    if ( byte_count )
    {
      if ( a1 )
      {
        if ( a3 )
        {
          v7 = malloc(byte_count);
          if ( v7 )
          {
            v16 = v7;
            sub_9C580(v7, 0, v5);
            v18 = a4;
            if ( v5 <= a4 )
            {
              v13 = 0;
              do
              {
                sub_C2DAC(v13, v5);
                v16[v14] = *(_BYTE *)(a1 + v14) ^ *(_BYTE *)(a3 + v13++);
              }
              while ( a4 != v13 );
              return v16;
            }
            else
            {
              v8 = 0;
              do
              {
                v9 = v5;
                v10 = *(_BYTE *)(a1 + v8);
                sub_C2DAC(v8, v18);
                v12 = *(_BYTE *)(a3 + v11) ^ v10;
                v5 = v9;
                v16[v8++] = v12;
              }
              while ( v9 != v8 );
              return v16;
            }
          }
        }
      }
    }
  }
  return (_BYTE *)v6;
}

```

只是做一些异或操作  

x 查看这个函数的引用  

发现有三个  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahOwMUTSiavXozKX8vBSgIuwqIrPdPWciaRyicuibn4OVO8xv94DE4bbhlSQ/640?wx_fmt=png)

这个地方有两个调用  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahJTHibRb5WTQeykicAeicPxrIgkonNQPLwWDOBKPsd35KUVcfV8ibIuahzQ/640?wx_fmt=png)

全部 hook 一下，看这个 key 是第几次出现的  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahBt1Wgmxra881kPDguzwYibk7W52aLkQQega2nzFH9NkNEVonuJRpOuA/640?wx_fmt=png)

```
 debugger.addBreakPoint(module.base+0x5CE60);

```

第一次

r0  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahXflMZxicwkWsM2q10kZlQdTkhibHBrlInj7pyrrWGNRyAAjUfM2YsxgA/640?wx_fmt=png)

r2

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahicgVJlG3ZM5vF0wc3JIP8PsVc9dKoEFuA7BsEg7WyhrZfiba4sMU8C9Q/640?wx_fmt=png)

结果存放到 r0  

blr

c  

第一次调用结果

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jwHc2ZeBicTk7LmyMzAoduKhZd24rGicwRbZwkJtO96LWRTeh9YaoMVbQ/640?wx_fmt=png)

第二次调用

r0 是 a1

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahNSleZ3mfQHnK1DianWkNicKJSQGtT8T635zz5UyTSEw1BMDz31qAsezA/640?wx_fmt=png)

r2 不变的

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahZ3d9R2mUiaRIM6FlTy7CqwYo8jNrBkMyTLQs57PBm0ibtpvHENCcXawQ/640?wx_fmt=png)

第二次调用结果  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jpBMF3TPt0F07Ngl2aGlofbYXsOqWPKcxpwX9t7YUicTZ5W4hnlCaEBw/640?wx_fmt=png)

第三次调用

r0 不知道怎么来的  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahA3iaj8SwutxgIoFqStfich938JMBRJFOd2lByz3iaicnVJ32vkxhHibsgsw/640?wx_fmt=png)

r2 第二次调用的结果  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6UIZicoeBusHsgZaXoAHPiahruoqREvpbnG3Bza0GNQlJQwpc2kprmbyZJAeA8zSgCZmjAw7X6pIPg/640?wx_fmt=png)

第三次调用结果  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jrmeovIccHz8ZSAoNcwX3W5ibMzvUhAIYuQdmuIOpLajplxtKHMupE0A/640?wx_fmt=png)

第四次调用

r0 不知道怎么来的

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jRrOQzMOG95ruvqrN7VJPM5RCWy6iaE2ApHyVgpOBEVFLmN8nMVmHwRA/640?wx_fmt=png)

r2 a1

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jkKt8GtAomNJIHOvVSVABqibUrH7ic1pdrRSC4HVgak60RbA1SpGXotBQ/640?wx_fmt=png)

第四次调用结果

hamcsha1 的 key

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jOibkgOiakmx4CJ7EzsQRBBPKyO5apwZsy8p9ZcoSO65WibfARHASUBIWA/640?wx_fmt=png)

再次 c，就会到 hmacsha1 函数

key 也就是第四次调用的结果

```
debugger break at: 0x40095d48

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j2qFUBhApo4Vtciaq2ibJYY2dmJDDuTsiaiaYbcasp50nKyy3IQ1ydZzhgw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jhZnGyO0lxKRKJ4Xbe0Oic8b5bKZgbXCicQiaGwGI4EjF89AvNKTeAj4qw/640?wx_fmt=png)

也可以写个 HookZz 持久化 就不用每次打断点

```
    public void hook_5CE60(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x5CE60+1 , new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer r0 = ctx.getPointerArg(0);
                System.out.println("r0:"+r0);
                int r1 = ctx.getIntArg(1);
//                ctx.push(r0);
                ctx.push(r1);
                Pointer r2 = ctx.getPointerArg(2);
                int r3 = ctx.getIntArg(3);
                Inspector.inspect(r0.getByteArray(0, r1),"sub_5CE60 r1");
                Inspector.inspect(r2.getByteArray(0, r3),"sub_5CE60 r2");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int leng = ctx.pop();
                Pointer output = ctx.getPointerArg(0);
                 System.out.println("output:"+output);
                byte[] outputhex = output.getByteArray(0, leng);
                Inspector.inspect(outputhex, "sub_5CE60 ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

接下来还是逆推  

在第四次调用的地方断住 查看调用栈

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j9ZgqPGRckHYic6Pp6B1AcjgpHjy9zMPS0AzgHEg3ZO2BmQOlZ8hw3pQ/640?wx_fmt=png)

g 0x5c4cf

ida 中打开发现就在 sub_5C384 函数里 47 行

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jA5BDprg8nHA07gRCtUnMco0Rx98DXicr4xjYzaEbOQenTjZcpo4libyw/640?wx_fmt=png)

r0 的来源未知

r2 的来源就是 a1

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jYUxh9MAE3sKDu3qnmygKILut2UKX2elJ7Sc38AYViauKc0WWmeuGU2Q/640?wx_fmt=png)

所以分析 r0 的来源

```
00000000: 54 9E 4A A5 0F BE 01 9E  D4 D6 36 27 30 54 C4 C3  T.J.......6'0T..
00000010: 95 6E 7D 41 26 57 1F 32  74 28 19 D5 2C 9C 9C A6  .n}A&W.2t(..,...

```

这里可以看伪代码分析

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jxjsbxk05iamKPaIVI1xHnb6iad4ZcWJK2CqtfD33ZrsuibsroBHo9YTKA/640?wx_fmt=png)

p 来自 v12

v12 来自 v23

hook 这一行

```
v12 = (void *)sub_67308(v23, &byte_count);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4P5LMAHsVDUE0bnAIqTgn7NDbomlqC0zcXxV2icWptR8DIKkoOkfSa4w/640?wx_fmt=png)

```
debugger.addBreakPoint(module.base+0x5C490);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jgK1CqXChVWqGd94tLcX0BFA8jx5EuEtZNc1mYiblN3jJicicflNZAgjYQ/640?wx_fmt=png)

到这里已经被转成了 16 进制字符串

所以还要往上追到这里 sub_5C5F8((int)v23, a2, (int)v11, 32);

进去  

进去发现很多花指令干扰了 ida 导致无法静态分析

还是用 trace 的方式追踪

```
emulator.traceWrite(0x4032223cL,0x4032223c+0x24);

```

```
### Memory WRITE at 0x4032223c, data size = 1, data value = 0x35 pc=RX@0x401c1650[libc.so]0x17650 lr=RX@0x400a7b9b[libmtguard.so]0xa7b9b
### Memory WRITE at 0x4032223d, data size = 1, data value = 0x34 pc=RX@0x401c1650[libc.so]0x17650 lr=RX@0x400a7b9b[libmtguard.so]0xa7b9b
### Memory WRITE at 0x4032223e, data size = 1, data value = 0x39 pc=RX@0x401c1650[libc.so]0x17650 lr=RX@0x400a7b9b[libmtguard.so]0xa7b9b
### Memory WRITE at 0x4032223f, data size = 1, data value = 0x65 pc=RX@0x401c1650[libc.so]0x17650 lr=RX@0x400a7b9b[libmtguard.so]0xa7b9b
### Memory WRITE at 0x40322240, data size = 1, data value = 0x34 pc=RX@0x401c1694[libc.so]0x17694 lr=RX@0x400a7b9b[libmtguard.so]0xa7b9b

```

在 lib 里面计算的，看 lr 的地址 0xa7b9b

跳转到这里

```
_BYTE *__fastcall sub_A7B58(_BYTE *a1, _BYTE *a2)
{
  _BYTE *v3; // r4
  size_t v4; // r5
  int v5; // r0
  int v6; // r1
  int v8; // [sp+4h] [bp-14h]
  v3 = &unk_F4964;
  if ( a1 != a2 )
  {
    if ( !a1 )
    {
      sub_A7DD4();
      sub_B75EC(&unk_F1CE2);
    }
    v4 = a2 - a1;
    v5 = sub_B7E48(a2 - a1, 0);
    v6 = v5;
    v3 = (_BYTE *)(v5 + 12);
    if ( v4 == 1 )
    {
      *v3 = *a1;
    }
    else
    {
      v8 = v5;
      qmemcpy((void *)(v5 + 12), a1, v4);
      v6 = v8;
    }
    if ( (_UNKNOWN *)v6 != &unk_F4958 )
    {
      *(_DWORD *)v6 = v4;
      *(_DWORD *)(v6 + 8) = 0;
      v3[v4] = 0;
    }
  }
  return v3;
}

```

这个函数太多调用，所以要有个合适的 hook 点  

先在这个地方打个断点

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4J4Jia1S1K42bRmibZeBqoonaQuDgQ0xodMKfLaHy1TZ2Jib7GUT1k6XyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4GpUiavbUib93lx6VGhViacSDf0rb7OadyZR1x7tcvxzD6Qich4l3ymIEHQ/640?wx_fmt=png)

```
debugger.addBreakPoint(module.base+0x5C47A);

```

再下 sub_A7B58 函数的断点，就能过滤掉前边的调用

b0xA7B58

c 第三次就出现了

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j8hvA9Yqia4A36MDLncfycROv5DMSQ8JvGE3ibRZ3BMZoERuOiahBPvdicg/640?wx_fmt=png)

然后 bt 打印调用栈

```
bt
[0x40000000][libmtguard.so][0xb8379]
[0x40000000][libmtguard.so][0x66edf]
[0x40000000][libmtguard.so][0x66f83]
[0x40000000][libmtguard.so][0x5c8bf]
[0x40000000][libmtguard.so][0x5c47b]
[0x40000000][libmtguard.so][0x5ab47]

```

```
int __fastcall sub_66F44(_DWORD *a1, int a2, int a3)
{
  void *v7; // [sp+Ch] [bp-3Ch] BYREF
  char v8[4]; // [sp+10h] [bp-38h] BYREF
  char v9[36]; // [sp+14h] [bp-34h] BYREF
  int v10; // [sp+38h] [bp-10h]
  if ( a2 && a3 )
  {
    memset(v9, 0, 0x21u);
    sub_97DFC(a2, a3, (int)v9);
    sub_66E50(&v7, (unsigned __int8 *)v9, 32);
    *a1 = v7;
    v7 = &unk_F4964;
    sub_B8118(&unk_F4958, v8);
  }
  else
  {
    sub_B8204(a1, &unk_F2DF0);
  }
  return _stack_chk_guard - v10;
}

```

0x66f83 下断点

b0x66f83

发现 r1 就是目标值，也就是 v9

```
sub_97DFC(a2, a3, (int)v9);
sub_66E50(&v7, (unsigned __int8 *)v9, 32);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jIKGcAMZicRrYdmiaeUadKqVzc82iaKaxeI2PpY7CPiasHQI1icDB02NOSibg/640?wx_fmt=png)

v9 是 sub_97DFC 的第三个参数，

hook sub_97DFC()  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j26cnlQEInPxvlPA3AF21iaDXGrQiaky5RzRzKRxc0XgPUgnjoXRax8RA/640?wx_fmt=png)

sub_97DFC 的结果放到二级指针 v9

blr

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jZq0ZZGhiciaJ6BwdLz5jSibjY12PMKh35MQCElymAdphvGx4tMs429h0Q/640?wx_fmt=png)

目标结果出现

所以调用 sub_97DFC() 函数，即可得到 54 9E 4A A5。。。

参数是  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4wgTy3STMpmJtg1ricL3QgQ6cSaC3nIcFkoN3tOKVicD0luJPlv4hVYsw/640?wx_fmt=png)

97 cd 开头的参数有点眼熟

往上翻翻，发现是 sub_56e60 计算得到的

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jN7KeSMam2p7ibmTaMJN5F625bCyNz9lopvPMVibJeiaRq5QSBbRdRsibAA/640?wx_fmt=png)

这里涉及到的两个参数，

E3 9D 67 47 开头的未知

74 50 57 50 开头的也是 sub_56e60() 计算得到

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4vzYvt9OgI1YiaiafXcHhBUzuRUGvR1hErGVepugiaDYRH4Mxm6rUeLIEA/640?wx_fmt=png)

**接下来分析 E3 9D 67 47 开头的来源**

sub_5ce60 第三次调用的时候，查看 r0 的地址

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4NOyLUI0nDiaJbftnYpaicSwKZjnX34CgE1BPyOW9eE6HNLzItY0OrCrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jtZnSPR6HsApnmVx7wBiaYfVLib2ias48PRwaL6GHxtCXdRhAuNruIQibiag/640?wx_fmt=png)

trace 大法：

监控 r0 地址的写入

```
emulator.traceWrite(0x4031f060L,0x4031f060L+0x30);

```

```
### Memory WRITE at 0x4031f060, data size = 1, data value = 0xe3 pc=RX@0x4009799a[libmtguard.so]0x9799a lr=RX@0x40097aeb[libmtguard.so]0x97aeb
### Memory WRITE at 0x4031f061, data size = 1, data value = 0x9d pc=RX@0x4009799a[libmtguard.so]0x9799a lr=RX@0x40097aeb[libmtguard.so]0x97aeb
### Memory WRITE at 0x4031f062, data size = 1, data value = 0x67 pc=RX@0x4009799a[libmtguard.so]0x9799a lr=RX@0x40097aeb[libmtguard.so]0x97aeb
### Memory WRITE at 0x4031f063, data size = 1, data value = 0x47 pc=RX@0x4009799a[libmtguard.so]0x9799a lr=RX@0x40097aeb[libmtguard.so]0x97aeb
### Memory WRITE at 0x4031f064, data size = 1, data value = 0xe1 pc=RX@0x4009799a[libmtguard.so]0x9799a lr=RX@0x40097aeb[libmtguard.so]0x97aeb

```

ida 中查看 0x9799a  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4t6DF8mk8WMiaI7fianibhdJiarPhgiaEiaX5kfRlG9icCq73N5CsdbeJxF5AQ/640?wx_fmt=png)

r1 是从 r5 加载的

下个断点查看

b0x97998

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4OBVFud9usFm3NsicMzeQWk0zYjpeFOGxTKWK49soIjW9pOzBbDt5Bjw/640?wx_fmt=png)

0xe3 来自 r5：0x4031f030

查看调用栈

```
bt
[0x40000000][libmtguard.so][0x97ceb] b sub_97528
[0x40000000][libmtguard.so][0x5ca9f] b sub_97BB4
[0x40000000][libmtguard.so][0x5c47b] b sub_5C5F8
[0x40000000][libmtguard.so][0x5ab47]

```

0x97ceb 函数是 sub_97528

hook sub_97528

b0x97528

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jI6lTk7RLyfAsotQiaWtajQNSZ5Ow0ntREpYayiaGFUhPjJK2Y2Be5oMA/640?wx_fmt=png)

r0 参数, r2 参数长度 0x10

blr  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jYvk5vpz2RMSKkfM1qiblFcPcaBYsSSoPQtVD630yNYjwibdTcO5qZfiaA/640?wx_fmt=png)

出现目标结果的前 16 个字节

c  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jgFJJzmxZCa9XxdiaOicsziaGdd400y7k9hdquQImtqyfMr4UiajF89JcmQ/640?wx_fmt=png)

一组 16 个字节

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j8DgDMyYlkluZPUA6sdzbW99s2dPtQicmZkVddFdk5iagxmoS99APZC0Q/640?wx_fmt=png)

第二组结果出现

c

第三次是填充数  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jr3u9yWmWFFIQdskM84KlkhdUcUJKdUg9dpp5p3kl1fibySQEHvS85Pw/640?wx_fmt=png)

blr  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jZ5ED7BjfqTm2913amKstSLZUOicD8hWNghFpRU1s9I6iaRYfhBflv4Pg/640?wx_fmt=png)

三组连起来就是

```
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51   ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04   ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a'.bI.e.

```

再 hook 上一层的 sub_97BB4

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4qxe77EM0yrMKPvkqiaNoDe4Ot7erEnd535DBpahFLfaBap3VziannZtA/640?wx_fmt=png)

hook sub_97BB4

 参数

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4Vbfo9rTyzYlVMdwZKAIFicwknkFROQR2wO6BpXbEhicaHe61n6vpPnDA/640?wx_fmt=png)

blr

结果

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4z0QmUXYvicZUKZQgVrmBFMOaZdDQKzsM2ib8w7GMd4WrexyKzfBgWib5A/640?wx_fmt=png)

所以调用 sub_97BB4() 函数，即可得到 E3 9D 67 47。。。

参数是

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4Vbfo9rTyzYlVMdwZKAIFicwknkFROQR2wO6BpXbEhicaHe61n6vpPnDA/640?wx_fmt=png)

似曾相识  

是 sub_5ce60 计算得到，第一次调用的结果

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4zibdPtpKOo9WISxKWgwG5vqkSSbLaN61CfmgHBib04icAiaXsKLS5b3ZGg/640?wx_fmt=png)

总结 a2 的流程：  

**1.** 获取 java 层传入的参数：

```
input_str="POST /mtapi/v8/poi/food __reqTra11111111111111111111111111111"//之前的抓包参数里有个人隐私- -，所以就改成1111111了
input_str[:32]="POST /mtapi/v8/poi/food __reqTra"
input_str[32:]="ceID=fd0bba3f-b1da-431a-a827-acf98096a93b&ad_activity_flag=&allowance_alliance_scenes=&app=0&brand_page_type=0&ci=20&content_info=&f=android&group_chat_share=&msid=DeviceId01656731651891&page_index=0&partner=4&platform=4&poilist_mt_cityid=20&poilist_wm_cityid=440100&push_token=dpsh6aa39a8185255a368853e6022cd12db0atpu&recall_type=0&recommend_product=&req_time=1656733188476&request_mark=%7B%22seckill%22%3A0%7D&search_log_id=&search_word=&source_page_type=0&style_template_ids=%5B%22746%22%2C%227.36_decision_info%22%5D&userid=-1&utm_campaign=AgroupBgroupC0E0Ghomepage_category1_394__a1__c-1024&utm_content=d95e3aeb06eb8788&utm_medium=android&utm_source=wandoujia&utm_term=1100090405&uuid=00000000000008154001C3A79471BB102404E7E994738A163513645095681841&version=11.9.405&version_

```

切割成两份：input_str[:32]；input_str[32:]，作为参数调用 sub_5CE60() 函数

得到

```
[16:50:10 558]sub_5CE60 ret, md5=cbc0248e7e394b358de23bed178e1581, hex=692962610d190e4453135d4b440c1f145a5918075c4254166d6d5f51436c104c
size: 32
0000: 69 29 62 61 0D 19 0E 44 53 13 5D 4B 44 0C 1F 14    i)ba...DS.]KD...
0010: 5A 59 18 07 5C 42 54 16 6D 6D 5F 51 43 6C 10 4C    ZY..\BT.mm_QCl.L

```

sub_5CE60_one = sub_5CE60(input_str[:32],input_str[32:])

**2.**sub_5CE60_one 作为参数调用 sub_97BB4() 函数:

得到  

```
[17:02:21 459]r0=RW@0x4031f060, md5=b8e5e6a67b617ea1fcfd0668be5b3970, hex=e39d6747e129d0f5fe73c71252948051c2c65ce308f5b8901b48e021d20599043ab3249f6849e1612720a86249de65b1
size: 48
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51    ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04    ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a' .bI.e.4031f060

```

sub_97BB4_ret = sub_97BB4(sub_5CE60_one)

**3.a1,salt 作为参数调用 sub_5CE60() 函数**

```
a1="9b69f861-e054-4bc4-9daf-d36ae205ed3e"
salt="/HntC9XIfdUyII/UiVfx020EQPpHz2XZY3qzM2aiNmM0i0pB1yeSO689TY9SBB3s"

```

得到

```
[17:05:05 878]sub_5CE60 ret, md5=ca286f49561ff88253a2a4ba04c987f3, hex=7450575028557b0144554077055451312c02150030385f7e267105121f00686f3c57421f
size: 36
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.

```

sub_5CE60_two = sub_5CE60(a1,salt)

**4.sub_97BB4_ret 和 sub_5CE60_two 作为参数调用 sub_5CE60() 函数**

```
_0x4031f060：
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51    ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04    ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a' .bI.e.4
sub_5CE60_two：
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.

```

得到  

```
[17:05:05 879]sub_5CE60 ret, md5=5c0b152dfa72d0193e78fa6b84258ff8, hex=97cd3017c97cabf4ba26876557c0d160eec449e338cde7ee3d39e533cd05f16b06e466801c19b6310f75d3630d8b25c6
size: 48
0000: 97 CD 30 17 C9 7C AB F4 BA 26 87 65 57 C0 D1 60    ..0..|...&.eW..`
0010: EE C4 49 E3 38 CD E7 EE 3D 39 E5 33 CD 05 F1 6B    ..I.8...=9.3...k
0020: 06 E4 66 80 1C 19 B6 31 0F 75 D3 63 0D 8B 25 C6    ..f....1.u.c..%.

```

sub_5CE60_three = sub_5CE60(sub_97BB4_ret,sub_5CE60_two)

**5.sub_5CE60_three 作为参数**调用 sub_97DFC() 函数

```
sub_5CE60_three：
0000: 97 CD 30 17 C9 7C AB F4 BA 26 87 65 57 C0 D1 60    ..0..|...&.eW..`
0010: EE C4 49 E3 38 CD E7 EE 3D 39 E5 33 CD 05 F1 6B    ..I.8...=9.3...k
0020: 06 E4 66 80 1C 19 B6 31 0F 75 D3 63 0D 8B 25 C6    ..f....1.u.c..%.

```

得到

```
[17:19:23 631]r4=unidbg@0xbffff644, md5=7a213bab4a3226df05f3444dbd6237d7, hex=549e4aa50fbe019ed4d636273054c4c3956e7d4126571f32742819d52c9c9ca6
size: 32
0000: 54 9E 4A A5 0F BE 01 9E D4 D6 36 27 30 54 C4 C3    T.J.......6'0T..
0010: 95 6E 7D 41 26 57 1F 32 74 28 19 D5 2C 9C 9C A6    .n}A&W.2t(..,...

```

sub_97DFC_ret = sub_97DFC(**sub_5CE60_three**)

**6.**sub_97DFC_ret 和 a1 作为参数调用 sub_5CE60() 函数****

```
sub_97DFC_ret：
0000: 54 9E 4A A5 0F BE 01 9E D4 D6 36 27 30 54 C4 C3    T.J.......6'0T..
0010: 95 6E 7D 41 26 57 1F 32 74 28 19 D5 2C 9C 9C A6    .n}A&W.2t(..,...
a1="9b69f861-e054-4bc4-9daf-d36ae205ed3e"

```

得到 hmacsha1 的动态 key

```
0000: 31 FA 79 C0 69 86 37 AF F9 B3 06 12 04 79 F0 A1    1.y.i.7......y..
0010: F6 5A 50 78 42 36 79 1F 10 1B 2F B4 49 AE AC 93    .ZPxB6y.../.I...

```

sub_5CE60_four = hmacsha1key = sub_5CE60(****sub_97DFC_ret****,****a1****)

7.input_str[32:] 和 sub_5CE60_four 作为参数调用 sub_0x95D48()（hmacsha1）

```
input_str[32:]="ceID=fd0bba3f-b1da-431a-a827-acf98096a93b&ad_activity_flag=&allowance_alliance_scenes=&app=0&brand_page_type=0&ci=20&content_info=&f=android&group_chat_share=&msid=DeviceId01656731651891&page_index=0&partner=4&platform=4&poilist_mt_cityid=20&poilist_wm_cityid=440100&push_token=dpsh6aa39a8185255a368853e6022cd12db0atpu&recall_type=0&recommend_product=&req_time=1656733188476&request_mark=%7B%22seckill%22%3A0%7D&search_log_id=&search_word=&source_page_type=0&style_template_ids=%5B%22746%22%2C%227.36_decision_info%22%5D&userid=-1&utm_campaign=AgroupBgroupC0E0Ghomepage_category1_394__a1__c-1024&utm_content=d95e3aeb06eb8788&utm_medium=android&utm_source=wandoujia&utm_term=1100090405&uuid=00000000000008154001C3A79471BB102404E7E994738A163513645095681841&version=11.9.405&version_
sub_5CE60_four:
0000: 31 FA 79 C0 69 86 37 AF F9 B3 06 12 04 79 F0 A1    1.y.i.7......y..
0010: F6 5A 50 78 42 36 79 1F 10 1B 2F B4 49 AE AC 93    .ZPxB6y.../.I...

```

得到 a2

```
[17:28:42 562]r6=unidbg@0xbffff6b0, md5=fa9c0f8c960ff5efb29dd1c626d28c36, hex=3ccddb3401c4d4a518c21a4b24025c047f4dc21b
size: 20
0000: 3C CD DB 34 01 C4 D4 A5 18 C2 1A 4B 24 02 5C 04    <..4.......K$.\.
0010: 7F 4D C2 1B                                        .M..

```

a2 = sub_0x95D48(input_str[32:],sub_5CE60_four)

算法还原

接下来分析 sub_97DFC

**sub_97DFC 函数还原**

逆向过程总是充满试错和灵感  

比如 64 长度 hex 字符传，有没有可能是 sha256 呢？

试一下

明文

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j26cnlQEInPxvlPA3AF21iaDXGrQiaky5RzRzKRxc0XgPUgnjoXRax8RA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710j01pV0s8NZAia4pg2GcjqjJod0mrXOAebCNFTd9UnibXjpISyJRib3Oib4A/640?wx_fmt=png)

```
a0842caa6f3fb8be0efa9294cbc2d8e653a83baad520270f88a2af3e9efa30c1

```

不对。那就算了

trace 存放地址的写入操作

```
emulator.traceWrite(0xbffff644L,0xbffff644L+0x20);

```

```
### Memory WRITE at 0xbffff661, data size = 1, data value = 0xfa pc=RX@0x4009c074[libmtguard.so]0x9c074 lr=RX@0x4009c07b[libmtguard.so]0x9c07b
### Memory WRITE at 0xbffff660, data size = 1, data value = 0x9e pc=RX@0x4009c074[libmtguard.so]0x9c074 lr=RX@0x4009c07b[libmtguard.so]0x9c07b
### Memory WRITE at 0xbffff644, data size = 1, data value = 0x18 pc=RX@0x4009c074[libmtguard.so]0x9c074 lr=RX@0x4009c07b[libmtguard.so]0x9c07b
### Memory WRITE at 0xbffff644, data size = 1, data value = 0x54 pc=RX@0x4009c074[libmtguard.so]0x9c074 lr=RX@0x4009c07b[libmtguard.so]0x9c07b

```

因为前面还很多调用，所以 0x9c074 下 inline_hook

```
    public void inline_hook(){
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                if (address == (module.base + 0x9c074)) {
                    Arm32RegisterContext ctx = emulator.getContext();
                    int r0 = ctx.getIntArg(0);
                    System.out.println(
                            "r0:" + Long.toHexString(r0) + ", "
                   );
                }
            }
            @Override
            public void onAttach(Unicorn.UnHook unHook) {
            }
            @Override
            public void detach() {
            }
        }, module.base , module.base + module.size, null);
    }

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jcS6SdyickGdB69qZe8kmzxjA8sKshFjxRKbSINMK0BD4BicDE1u6PbibQ/640?wx_fmt=png)

在合适的地方打印调用者，下断点， 并打印汇编代码

```
    public void inline_hook(){
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                if (address == (module.base + 0x9c074)) {
                    Arm32RegisterContext ctx = emulator.getContext();
                    int r0 = ctx.getIntArg(0);
                    if (r0 == 0x118){
                        System.out.println("r0 == 0x118");
                        emulator.getUnwinder().unwind();
                        Debugger debugger = emulator.attach();
                        debugger.addBreakPoint(module.base+0x9c074);//hmacsh1
                        emulator.traceCode();
                    }
                        System.out.println(
                            "r0:" + Long.toHexString(r0) + ", "
                   );
                }
            }
            @Override
            public void onAttach(Unicorn.UnHook unHook) {
            }
            @Override
            public void detach() {
            }
        }, module.base , module.base + module.size, null);
    }

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jsbT6wcfHianooQsywEZXxicNdAxDH3XCZvW155Gu7O68ZhabgsK01yIg/640?wx_fmt=png)

找 r0 的来源，往上追

```
### Trace Instruction [libmtguard.so] [0x9b897] [       4a 40 ] 0x4009b896: eors r2, r1>>> r1=0x4c r2=0x18

```

发现来自 r2

打印具体值  

```
    public void inline_hook(){
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                if (address == (module.base + 0x9b896)) {
                    Arm32RegisterContext ctx = emulator.getContext();
                    int r1 = ctx.getIntArg(1);
                    int r2 = ctx.getIntArg(2);
                    int res = r1 ^ r2;
                    System.out.println(
                            "eors r2, r1 --- " +
                                    "r2:" + Long.toHexString(r2) + ", " +
                                    "r1:" + Long.toHexString(r1) + ", " +
                                    "res:" + Long.toHexString(res)
                    );
                }
            }
            @Override
            public void onAttach(Unicorn.UnHook unHook) {
            }
            @Override
            public void detach() {
            }
        }, module.base , module.base + module.size, null);
    }

```

```
eors r2, r1 --- r2:18, r1:4c, res:54
eors r2, r1 --- r2:e, r1:90, res:9e
eors r2, r1 --- r2:30, r1:7a, res:4a
eors r2, r1 --- r2:b3, r1:16, res:a5
eors r2, r1 --- r2:14, r1:1b, res:f
eors r2, r1 --- r2:77, r1:c9, res:be
eors r2, r1 --- r2:b7, r1:b6, res:1
eors r2, r1 --- r2:f7, r1:69, res:9e
eors r2, r1 --- r2:ae, r1:7a, res:d4
eors r2, r1 --- r2:b3, r1:65, res:d6
eors r2, r1 --- r2:b2, r1:84, res:36
eors r2, r1 --- r2:54, r1:73, res:27
eors r2, r1 --- r2:b2, r1:82, res:30
eors r2, r1 --- r2:97, r1:c3, res:54
eors r2, r1 --- r2:c0, r1:4, res:c4
eors r2, r1 --- r2:d2, r1:11, res:c3
eors r2, r1 --- r2:1, r1:94, res:95
eors r2, r1 --- r2:c5, r1:ab, res:6e
eors r2, r1 --- r2:5f, r1:22, res:7d
eors r2, r1 --- r2:9a, r1:db, res:41
eors r2, r1 --- r2:4a, r1:6c, res:26
eors r2, r1 --- r2:ba, r1:ed, res:57
eors r2, r1 --- r2:82, r1:9d, res:1f
eors r2, r1 --- r2:b2, r1:80, res:32
eors r2, r1 --- r2:85, r1:f1, res:74
eors r2, r1 --- r2:5a, r1:72, res:28
eors r2, r1 --- r2:d9, r1:c0, res:19
eors r2, r1 --- r2:a6, r1:73, res:d5
eors r2, r1 --- r2:c7, r1:eb, res:2c
eors r2, r1 --- r2:db, r1:47, res:9c
eors r2, r1 --- r2:d6, r1:4a, res:9c
eors r2, r1 --- r2:f9, r1:5f, res:a6

```

一般是密文和一个固定 (key) 异或，这里两个哪个才是 key 呢，改一下入参看看哪个不变的就是了

修改后运行

```
eors r2, r1 --- r2:71, r1:4c, res:3d
eors r2, r1 --- r2:1b, r1:90, res:8b
eors r2, r1 --- r2:de, r1:7a, res:a4
eors r2, r1 --- r2:c0, r1:16, res:d6
eors r2, r1 --- r2:af, r1:1b, res:b4
eors r2, r1 --- r2:ee, r1:c9, res:27
eors r2, r1 --- r2:af, r1:b6, res:19
eors r2, r1 --- r2:b6, r1:69, res:df
eors r2, r1 --- r2:fe, r1:7a, res:84
eors r2, r1 --- r2:34, r1:65, res:51
eors r2, r1 --- r2:6a, r1:84, res:ee
eors r2, r1 --- r2:4c, r1:73, res:3f
eors r2, r1 --- r2:90, r1:82, res:12
eors r2, r1 --- r2:55, r1:c3, res:96
eors r2, r1 --- r2:b4, r1:4, res:b0
eors r2, r1 --- r2:ce, r1:11, res:df
eors r2, r1 --- r2:9d, r1:94, res:9
eors r2, r1 --- r2:6b, r1:ab, res:c0
eors r2, r1 --- r2:0, r1:22, res:22
eors r2, r1 --- r2:17, r1:db, res:cc
eors r2, r1 --- r2:a2, r1:6c, res:ce
eors r2, r1 --- r2:61, r1:ed, res:8c
eors r2, r1 --- r2:4f, r1:9d, res:d2
eors r2, r1 --- r2:8f, r1:80, res:f
eors r2, r1 --- r2:e7, r1:f1, res:16
eors r2, r1 --- r2:93, r1:72, res:e1
eors r2, r1 --- r2:33, r1:c0, res:f3
eors r2, r1 --- r2:69, r1:73, res:1a
eors r2, r1 --- r2:85, r1:eb, res:6e
eors r2, r1 --- r2:b3, r1:47, res:f4
eors r2, r1 --- r2:89, r1:4a, res:c3
eors r2, r1 --- r2:e5, r1:5f, res:ba

```

发现 r1 是不变的  

so

```
r1_arr
00000000: 4C 90 7A 16 1B C9 B6 69  7A 65 84 73 82 C3 04 11  L.z....ize.s....
00000010: 94 AB 22 DB 6C ED 9D 80  F1 72 C0 73 EB 47 4A 5F  ..".l....r.s.GJ_
r1_hexstr:4c907a161bc9b6697a65847382c3041194ab22db6ced9d80f172c073eb474a5f

```

```
key=[0x4c, 0x90, 0x7a, 0x16, 0x1b, 0xc9, 0xb6, 0x69, 0x7a, 0x65, 0x84, 0x73, 0x82, 0xc3, 0x4, 0x11, 0x94, 0xab,0x22, 0xdb, 0x6c, 0xed, 0x9d, 0x80, 0xf1, 0x72, 0xc0, 0x73, 0xeb, 0x47, 0x4a, 0x5f]

```

接下来就是找 r2 的来源

```
Trace Instruction [libmtguard.so] [0x9b895] [       02 68 ] 0x4009b894: ldr r2, [r0]>>> r0=0x4057fde0 r2=0x4057fe08
eors r2, r1 --- r2:18, r1:4c, res:54
Trace Instruction [libmtguard.so] [0x9b897] [       4a 40 ] 0x4009b896: eors r2, r1>>> r1=0x4c r2=0x18

```

r2 是从 0x4057fde0 读取的

监控写入  

```
emulator.traceWrite(0x4057fde0L,0x4057fde0L+0x20)

```

发现很多结果，一顿分析发现  

```
Memory WRITE at 0x4057fde0, data size = 4, data value = 0x118 pc=RX@0x4009c24c[libmtguard.so]0x9c24c lr=RX@0x4009c07b[libmtguard.so]0x9c07b

```

在这个地址 inline_hook 0x9c24c

```
    public void inline_hook(){
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                if (address == (module.base + 0x9C24C)) {
                    Arm32RegisterContext ctx = emulator.getContext();
                    int r1 = ctx.getIntArg(1);
                    if(r1==0x4057fde8){
                        System.out.println("aaaaaaaaaaa");
                        emulator.getUnwinder().unwind();
                        Debugger debugger = emulator.attach();
                        debugger.addBreakPoint(module.base+0x9c24c);
                        emulator.traceCode();
                    }
                    System.out.println(
                            "r1:" + Long.toHexString(r1) + ", "
                    );
                }
            }
            @Override
            public void onAttach(Unicorn.UnHook unHook) {
            }
            @Override
            public void detach() {
            }
        }, module.base , module.base + module.size, null);
    }

```

往上追发现 0x118 的来源

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH7yibRZm854RWRtaZ3sL710jgUkHISwWGvDAeo94laooOVl7KunIyppaRWkiauF22dlwiaxG9D42b2mg/640?wx_fmt=png)

r1=r2+r1

```
### Trace Instruction [libmtguard.so] [0x9b6e7] [       51 18 ] 0x4009b6e6: adds r1, r2, r1>>> r1=0x78 r2=0xa0
### Trace Instruction [libmtguard.so] [0x9b6e9] [       03 b5 ] 0x4009b6e8: push {r0, r1, lr}>>> r0=0x4057fde0 r1=0x118 LR=0x4009c07b

```

inline_hook 并过滤一下  

```
    public void inline_hook(){
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                if (address == (module.base + 0x9B6E6)) {
                    Arm32RegisterContext ctx = emulator.getContext();
                    int r1 = ctx.getIntArg(1);
                    int r2 = ctx.getIntArg(2);
                    int res = r2 + r1;
                    if((r2>0)&&(r2<0xffff)&&(r1!=1)){
                        System.out.println(
                                "adds r2, r1 --- " +
                                        "r2:" + Long.toHexString(r2) + ", " +
                                        "r1:" + Long.toHexString(r1) + ", " +
                                        "res:" + Long.toHexString(res)
                        );
                    }
                }
            }
            @Override
            public void onAttach(Unicorn.UnHook unHook) {
            }
            @Override
            public void detach() {
            }
        }, module.base , module.base + module.size, null);
    }

```

```
adds r2, r1 --- r2:a0, r1:78, res:118
adds r2, r1 --- r2:84, r1:8a, res:10e
adds r2, r1 --- r2:2c, r1:4, res:30
adds r2, r1 --- r2:aa, r1:9, res:b3
adds r2, r1 --- r2:6f, r1:a5, res:114
adds r2, r1 --- r2:3f, r1:38, res:77
adds r2, r1 --- r2:b8, r1:ff, res:1b7
adds r2, r1 --- r2:be, r1:39, res:f7
adds r2, r1 --- r2:e, r1:a0, res:ae
adds r2, r1 --- r2:fa, r1:b9, res:1b3
adds r2, r1 --- r2:92, r1:20, res:b2
adds r2, r1 --- r2:94, r1:c0, res:154
adds r2, r1 --- r2:cb, r1:e7, res:1b2
adds r2, r1 --- r2:c2, r1:d5, res:197
adds r2, r1 --- r2:d8, r1:e8, res:1c0
adds r2, r1 --- r2:e6, r1:ec, res:1d2
adds r2, r1 --- r2:53, r1:ae, res:101
adds r2, r1 --- r2:a8, r1:1d, res:c5
adds r2, r1 --- r2:3b, r1:24, res:5f
adds r2, r1 --- r2:aa, r1:f0, res:19a
adds r2, r1 --- r2:d5, r1:75, res:14a
adds r2, r1 --- r2:20, r1:9a, res:ba
adds r2, r1 --- r2:27, r1:5b, res:82
adds r2, r1 --- r2:f, r1:a3, res:b2
adds r2, r1 --- r2:88, r1:fd, res:185
adds r2, r1 --- r2:a2, r1:b8, res:15a
adds r2, r1 --- r2:af, r1:2a, res:d9
adds r2, r1 --- r2:3e, r1:68, res:a6
adds r2, r1 --- r2:9e, r1:29, res:c7
adds r2, r1 --- r2:fa, r1:e1, res:1db
adds r2, r1 --- r2:30, r1:a6, res:d6
adds r2, r1 --- r2:c1, r1:38, res:f9

```

看起来是由 res=r2+(r1&0xff)

和上面一样，确定一下两组都是动态还是有一组固定值

改一下输入

```
adds r2, r1 --- r2:58, r1:78, res:d0
adds r2, r1 --- r2:b2, r1:8a, res:13c
adds r2, r1 --- r2:86, r1:4, res:8a
adds r2, r1 --- r2:98, r1:9, res:a1
adds r2, r1 --- r2:56, r1:a5, res:fb
adds r2, r1 --- r2:7c, r1:38, res:b4
adds r2, r1 --- r2:cc, r1:ff, res:1cb
adds r2, r1 --- r2:e3, r1:39, res:11c
adds r2, r1 --- r2:d4, r1:a0, res:174
adds r2, r1 --- r2:e6, r1:b9, res:19f
adds r2, r1 --- r2:2a, r1:20, res:4a
adds r2, r1 --- r2:32, r1:c0, res:f2
adds r2, r1 --- r2:e3, r1:e7, res:1ca
adds r2, r1 --- r2:5, r1:d5, res:da
adds r2, r1 --- r2:95, r1:e8, res:17d
adds r2, r1 --- r2:e0, r1:ec, res:1cc
adds r2, r1 --- r2:53, r1:ae, res:101
adds r2, r1 --- r2:2f, r1:1d, res:4c
adds r2, r1 --- r2:43, r1:24, res:67
adds r2, r1 --- r2:c8, r1:f0, res:1b8
adds r2, r1 --- r2:36, r1:75, res:ab
adds r2, r1 --- r2:20, r1:9a, res:ba
adds r2, r1 --- r2:65, r1:5b, res:c0
adds r2, r1 --- r2:62, r1:a3, res:105
adds r2, r1 --- r2:21, r1:fd, res:11e
adds r2, r1 --- r2:b4, r1:b8, res:16c
adds r2, r1 --- r2:81, r1:68, res:e9
adds r2, r1 --- r2:40, r1:29, res:69
adds r2, r1 --- r2:f4, r1:e1, res:1d5
adds r2, r1 --- r2:51, r1:a6, res:f7
adds r2, r1 --- r2:d9, r1:38, res:111

```

发现 r1 又是不变的

```
r1_arr
00000000: 78 8A 04 09 A5 38 FF 39  A0 B9 20 C0 E7 D5 E8 EC  x....8.9.. .....
00000010: AE 1D 24 F0 75 9A 5B A3  FD B8 2A 68 29 E1 A6 38  ..$.u.[...*h)..8
r1_hexstr:788a0409a538ff39a0b920c0e7d5e8ecae1d24f0759a5ba3fdb82a6829e1a638

```

```
salt_key = [0x78, 0x8a, 0x4, 0x9, 0xa5, 0x38, 0xff, 0x39, 0xa0, 0xb9, 0x20, 0xc0, 0xe7, 0xd5, 0xe8, 0xec, 0xae, 0x1d, 0x24, 0xf0, 0x75, 0x9a, 0x5b, 0xa3, 0xfd, 0xb8, 0x2a, 0x68, 0x29, 0xe1, 0xa6, 0x38]

```

接下来又要追 r2 的来源

```
r2_arr
00000000: A0 84 2C AA 6F 3F B8 BE  0E FA 92 94 CB C2 D8 E6  ..,.o?..........
00000010: 53 A8 3B AA D5 20 27 0F  88 A2 AF 3E 9E FA 30 C1  S.;.. '....>..0.
r2_hexstr:a0842caa6f3fb8be0efa9294cbc2d8e653a83baad520270f88a2af3e9efa30c1

```

逆向过程总是充满惊喜  

这不就是一开始 sha256 的结果？

那么就破案了

sub_97DFC 计算流程为：把 0x30 长度的参数先标准 sha256 编码，然后把编码结果和一个固定 32 位盐相加，相加的结果再和固定的 key 异或

```
input_hexstr="97cd3017c97cabf4ba26876557c0d160eec449e338cde7ee3d39e533cd05f16b06e466801c19b6310f75d3630d8b25c6"
sha256_res = sha256(input_hexstr)
sha256_res = "a0842caa6f3fb8be0efa9294cbc2d8e653a83baad520270f88a2af3e9efa30c1"
byte两两相加
#固定盐值
salt_hexstr = "788a0409a538ff39a0b920c0e7d5e8ecae1d24f0759a5ba3fdb82a6829e1a638"
#相加结果
add_res = "180e30b31477b7f7aeb3b254b297c0d201c55f9a4aba82b2855ad9a6c7dbd6f9"
byte两两异或
##固定key值
key_hexstr = "4c907a161bc9b6697a65847382c3041194ab22db6ced9d80f172c073eb474a5f"
#异或结果
eor_res = "549e4aa50fbe019ed4d636273054c4c3956e7d4126571f32742819d52c9c9ca6"

```

输入

```
input_hexstr="97cd3017c97cabf4ba26876557c0d160eec449e338cde7ee3d39e533cd05f16b06e466801c19b6310f75d3630d8b25c6"

```

还记得怎么来的？

回到主流程，由 sub_5CE60 计算得到

```
[21:21:09 256]sub_5CE60 r1, md5=b8e5e6a67b617ea1fcfd0668be5b3970, hex=e39d6747e129d0f5fe73c71252948051c2c65ce308f5b8901b48e021d20599043ab3249f6849e1612720a86249de65b1
size: 48
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51    ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04    ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a' .bI.e.
^-----------------------------------------------------------------------------^
>-----------------------------------------------------------------------------<
[21:21:09 256]sub_5CE60 r2, md5=ca286f49561ff88253a2a4ba04c987f3, hex=7450575028557b0144554077055451312c02150030385f7e267105121f00686f3c57421f
size: 36
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.
^-----------------------------------------------------------------------------^
output:RW@0x4031f030
>-----------------------------------------------------------------------------<
[21:21:09 257]sub_5CE60 ret, md5=5c0b152dfa72d0193e78fa6b84258ff8, hex=97cd3017c97cabf4ba26876557c0d160eec449e338cde7ee3d39e533cd05f16b06e466801c19b6310f75d3630d8b25c6
size: 48
0000: 97 CD 30 17 C9 7C AB F4 BA 26 87 65 57 C0 D1 60    ..0..|...&.eW..`
0010: EE C4 49 E3 38 CD E7 EE 3D 39 E5 33 CD 05 F1 6B    ..I.8...=9.3...k
0020: 06 E4 66 80 1C 19 B6 31 0F 75 D3 63 0D 8B 25 C6    ..f....1.u.c..%.

```

参数分别是

参数 1

```
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51    ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04    ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a' .bI.e.

```

参数 2

```
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.

```

参数 2 又眼熟  

还是 sub_5CE60 函数计算的

```
[21:21:09 255]sub_5CE60 r1, md5=7f5c9fdbdb01a3e3a5d76c93ebbbba40, hex=39623639663836312d653035342d346263342d396461662d643336616532303565643365
size: 36
0000: 39 62 36 39 66 38 36 31 2D 65 30 35 34 2D 34 62    9b69f861-e054-4b
0010: 63 34 2D 39 64 61 66 2D 64 33 36 61 65 32 30 35    c4-9daf-d36ae205
0020: 65 64 33 65                                        ed3e
^-----------------------------------------------------------------------------^
>-----------------------------------------------------------------------------<
[21:21:09 255]sub_5CE60 r2, md5=07c474d41456fc31cf90aa39a38b679d, hex=2f486e74433958496664557949492f556956667830323045515070487a32585a5933717a4d3261694e6d4d3069307042317965534f3638395459395342423373
size: 64
0000: 2F 48 6E 74 43 39 58 49 66 64 55 79 49 49 2F 55    /HntC9XIfdUyII/U
0010: 69 56 66 78 30 32 30 45 51 50 70 48 7A 32 58 5A    iVfx020EQPpHz2XZ
0020: 59 33 71 7A 4D 32 61 69 4E 6D 4D 30 69 30 70 42    Y3qzM2aiNmM0i0pB
0030: 31 79 65 53 4F 36 38 39 54 59 39 53 42 42 33 73    1yeSO689TY9SBB3s
^-----------------------------------------------------------------------------^
output:RW@0x4031a050
>-----------------------------------------------------------------------------<
[21:21:09 255]sub_5CE60 ret, md5=ca286f49561ff88253a2a4ba04c987f3, hex=7450575028557b0144554077055451312c02150030385f7e267105121f00686f3c57421f
size: 36
0000: 74 50 57 50 28 55 7B 01 44 55 40 77 05 54 51 31    tPWP(U{.DU@w.TQ1
0010: 2C 02 15 00 30 38 5F 7E 26 71 05 12 1F 00 68 6F    ,...08_~&q....ho
0020: 3C 57 42 1F                                        <WB.

```

现在只剩下参数 1 不知道未知  

```
0000: E3 9D 67 47 E1 29 D0 F5 FE 73 C7 12 52 94 80 51    ..gG.)...s..R..Q
0010: C2 C6 5C E3 08 F5 B8 90 1B 48 E0 21 D2 05 99 04    ..\......H.!....
0020: 3A B3 24 9F 68 49 E1 61 27 20 A8 62 49 DE 65 B1    :.$.hI.a' .bI.e.

```

这个参数是在流程 2 计算的  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4DqpJoZ2iavphqWojMKH44byDibM7XntaEXibAnXL9icLDCKpoZGYghKYwg/640?wx_fmt=png)

**sub_97BB4 函数还原**

主动调用

```
    public void call_97BB4(){
        List<Object> list = new ArrayList<>(10);
        String input ="692962610d190e4453135d4b440c1f145a5918075c4254166d6d5f51436c104c";
        byte[] arg0_ = hexStr2bytes(input);
        MemoryBlock memoryBlock1 =emulator.getMemory().malloc(0x30, true);
        UnidbgPointer input_ptr=memoryBlock1.getPointer();
        input_ptr.write(arg0_);
        int input_length = arg0_.length;
        MemoryBlock memoryBlock2 =emulator.getMemory().malloc(0x30, true);
        UnidbgPointer output_buffer=memoryBlock2.getPointer();
        list.add(input_ptr);
        list.add(input_length);
        list.add(output_buffer);
        module.callFunction(emulator, 0x97bb4 + 1,list.toArray());
        Number number=emulator.getBackend().reg_read(ArmConst.UC_ARM_REG_R0);
        int reg = number.intValue();
        Pointer pointer = UnidbgPointer.pointer(emulator, reg);
        byte[] data = pointer.getByteArray(0, 0x30);
        Inspector.inspect(data, "sub_97528 output");
    }

```

```
bt
[0x40000000][libmtguard.so][0x97ceb] b sub_97528
[0x40000000][libmtguard.so][0x5ca9f] b sub_97BB4
[0x40000000][libmtguard.so][0x5c47b] b sub_5C5F8
[0x40000000][libmtguard.so][0x5ab47]

```

之前的调用栈

hook sub_97528

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4UAokCXibRYY0bicBk6kL76icabbppLMPIUOnwHAE4ogGgehm4K2NE5L2Q/640?wx_fmt=png)

发现最后会多出一组 16 字节的数据

这个有没有可能是 PKCS7 填充算法呢

很有可能

改一下输入试试

```
public void call_97BB4() throws FileNotFoundException {
        List<Object> list = new ArrayList<>(10);
        String input = "69";
        byte[] arg0_ = hexStr2bytes(input);
        MemoryBlock memoryBlock1 = emulator.getMemory().malloc(0x30, true);
        UnidbgPointer input_ptr=memoryBlock1.getPointer();
        input_ptr.write(arg0_);
        int input_length = arg0_.length;
        MemoryBlock memoryBlock2 = emulator.getMemory().malloc(0x30, true);
        UnidbgPointer output_buffer=memoryBlock2.getPointer();
        list.add(input_ptr);
        list.add(input_length);
        list.add(output_buffer);
//        String traceFile = "D:\\qxp\\unidbg-xinban\\unidbg-xinban\\unidbg-android\\src\\test\\java\\com\\candy\\trace_a2.txt";
//        PrintStream traceStream = new PrintStream(new FileOutputStream(traceFile), true);
//        emulator.traceCode(module.base,module.base+module.size).setRedirect(traceStream);
//        emulator.traceWrite(module.base,module.base+module.size).setRedirect(traceStream);
//        emulator.traceRead(module.base,module.base+module.size).setRedirect(traceStream);
        module.callFunction(emulator, 0x97bb4 + 1, list.toArray());
        Number number =emulator.getBackend().reg_read(ArmConst.UC_ARM_REG_R0);
        int reg = number.intValue();
        Pointer pointer = UnidbgPointer.pointer(emulator, reg);
        byte[] data = pointer.getByteArray(0, 0x30);
        Inspector.inspect(data, "sub_97528 output");
    }

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5Yw0z5g0EkJicKDxBBbZsia4Kt4j2bTZdmibAZNfX0BGg1Sfd4nKwTBXQ18wMkhxVgPJxRMvtby22Tg/640?wx_fmt=png)

果然是 PKCS7

介绍一下 PKCS7

```
PKCS7是当下各大加密算法都遵循的填充算法
PKCS7Padding的填充方式为当数据长度不足数据块长度时,缺几位补几个几对于
AES128算法其数据块为16Byte（数据长度需要为16Byte的倍数）,如果数据为
"00112233445566778899AA"一共11个Byte,缺了5位,采用PKCS7Padding方式
填充之后的数据为"00112233445566778899AA0505050505"。
特别注意的一点是如果是数据刚好满足数据块长度也要在原数据后在按PKCS7规则
填充一个数据块数据，这样做的目的是为了区分有效数据和补齐数据。仍以AES128
为例：如果数据为"00112233445566778899AABBCCDDEEFF"一共16个符合数据块
规则采用PKCS7Padding方式填充之后的数据为
"00112233445566778899AABBCCDDEEFF10101010101010101010101010101010"

```

```
int __fastcall sub_B26BC(int a1, int a2, _DWORD *a3)
{
  int result; // r0
  int v6; // r5
  void *v7; // r1
  _DWORD *v8; // [sp+0h] [bp-18h]
  int v9; // [sp+4h] [bp-14h]
  int v10; // [sp+8h] [bp-10h]
  result = 0;
  v6 = 0;
  if ( a1 && a2 )
  {
    v8 = a3;
    v9 = 16 - (a2 & 0xF);
    v6 = v9 + a2;
    v7 = malloc(v9 + a2);
    result = 0;
    v10 = (int)v7;
    if ( !v7 )
      return result;
    sub_9C580(v7, 0);
    sub_9C5CA(v10, a1);
    sub_9C580(v10 + a2, v9);
    result = v10;
    a3 = v8;
  }
  *a3 = v6;
  return result;
}

```

什么算法呢，后面跟进去分析之后很像 aes，但是找不到 key？

懂的人已经懂了，但是去年七月份的时候我不懂 -。-