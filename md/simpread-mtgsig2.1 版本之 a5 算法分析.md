> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bfGvUyIN8aS4Y1KkOWwRpQ)

**新年快乐~**

**![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxsSicZX0U5A0kYxlgR1aAP24lT9ibv1OyHIkUpcUyDJ1nvaenSUFhnp9g/640?wx_fmt=png)**

搞过 mtgsig 的应该都知道，a0 代表算法版本，a1 可以理解为 mt 系每个 app 的标识码，此文章所用 app 和版本是 dp10.52.15

简单说一下怎么定位 a5 的加密入口以及算法还原

首先这里用的指令执行模拟框架还是 unidbg  

补环境不再说了，直接放出完整调用代码

```
package com.xxx;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.hook.hookzz.*;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.api.ApplicationInfo;
import com.github.unidbg.linux.android.dvm.array.ArrayObject;
import com.github.unidbg.linux.android.dvm.array.ByteArray;
import com.github.unidbg.linux.android.dvm.wrapper.DvmInteger;
import com.github.unidbg.linux.android.dvm.wrapper.DvmLong;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.utils.Inspector;
import com.sun.jna.Pointer;
import java.io.File;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
public class Mtgsig extends AbstractJni implements IOResolver {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private static final String APP_PACKAGE_NAME = "com.xxx.v1";
    Mtgsig() {
        emulator = AndroidEmulatorBuilder.for32Bit().setProcessName(APP_PACKAGE_NAME).build(); // 创建模拟器实例，要模拟32位或者64位，在这里区分
        final Memory memory = emulator.getMemory(); // 模拟器的内存操作接口
        memory.setLibraryResolver(new AndroidResolver(23)); // 设置系统类库解析
        vm = emulator.createDalvikVM(new File("unidbg-android\\src\\test\\java\\com\\xxx\\xxx\xxx.apk")); // 创建Android虚拟机
        vm.setVerbose(true); // 设置是否打印Jni调用细节
        vm.setJni(this);
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android\\src\\test\\java\\com\\xxx\\xxx\\xxx.so"), true);
        emulator.getSyscallHandler().addIOResolver(this);
        module = dm.getModule();
        dm.callJNI_OnLoad(emulator);
    }
    public static void main(String[] args) {
        Mtgsig test = new Mtgsig();
        test.HookByConsoleDebugger();
        test.hookrc4();
        test.hookrc4_key();
        test.hookbase64();
        test.hookbase64_ret();
        test.callInit();
        test.callMain();
    }
    public void callInit(){
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv()); // 第一个参数是env
        list.add(0); // 第二个参数，实例方法是jobject，静态方法是jclazz，直接填0，一般用不到。
        list.add(1);
        ArrayObject myobject = new ArrayObject(null);
        vm.addLocalObject(myobject);
        // 完整的参数2
        list.add(vm.addLocalObject(myobject));
        module.callFunction(emulator, 0x4305, list.toArray());
    };
    public void callMain(){
        List<Object> list = new ArrayList<>(10);
        list.add(vm.getJNIEnv()); // 第一个参数是env
        list.add(0); // 第二个参数，实例方法是jobject，静态方法是jclazz，直接填0，一般用不到。
        list.add(2);
        StringObject input2_1 = new StringObject(vm, "4069cb78-e02b-45f6-9f0a-b34ddccf389c");
        String input="GET /localpush.bin cityid=4";
        ByteArray input2_2 = new ByteArray(vm, input.getBytes(StandardCharsets.UTF_8));
        DvmInteger input2_3 = DvmInteger.valueOf(vm, 2);
        vm.addLocalObject(input2_1);
        vm.addLocalObject(input2_2);
        vm.addLocalObject(input2_3);
        // 完整的参数2
        list.add(vm.addLocalObject(new ArrayObject(input2_1, input2_2, input2_3)));
        Number number = module.callFunction(emulator, 0x4305, list.toArray())[0];
        StringObject result = (StringObject) ((DvmObject[])((ArrayObject)vm.getObject(number.intValue())).getValue())[0];
        System.out.println(result.getValue());
    };
    public void hookrc4(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x88F80 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length = ctx.getIntArg(3);
                Pointer out = ctx.getPointerArg(2);
                ctx.push(out);
                Inspector.inspect(ctx.getPointerArg(1).getByteArray(0, length),"rc4 input");
         };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer output = ctx.pop();
                byte[] outputhex = output.getByteArray(0, 193);
                Inspector.inspect(outputhex, "hook rc4 结果");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
    public void hookrc4_key(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x88EEA + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length = ctx.getIntArg(2);
                Pointer arg1 = ctx.getPointerArg(1);
                // ctx.push(ctx.getR2Pointer());
                Pointer out1 = ctx.getPointerArg(0);
                ctx.push(out1);
                Inspector.inspect(arg1.getByteArray(0, length),"rc4 key input");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer output = ctx.pop();
                System.out.println(output);
                byte[] outputhex = output.getByteArray(0, 256);
                Inspector.inspect(outputhex, "hook rc4 key 结果");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
    public void hookbase64(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0xB468 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length = ctx.getIntArg(2);
                Pointer out = ctx.getPointerArg(0);
                ctx.push(out);
                Inspector.inspect(ctx.getPointerArg(1).getByteArray(0, length),"base64 input");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
    public void hookbase64_ret(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x8C8E0 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length = ctx.getIntArg(2);
                Pointer out = ctx.getPointerArg(1);
                ctx.push(out);
                Inspector.inspect(ctx.getPointerArg(1).getByteArray(0, length),"base64 结果");
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }
    public void HookByConsoleDebugger(){
        Debugger debugger = emulator.attach();
        debugger.addBreakPoint(module.base+0x88EEA+1);//rc4 key
        debugger.addBreakPoint(module.base+0x88F80+1);//rc4
        //   debugger.addBreakPoint(module.base+0xB468);//base64
        //trace a5 baseb4的地址
        //emulator.traceWrite(0x40335300L,0x40335300L+0xc0L);
    }
    @Override
    public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature){
            case "com/meituan/android/common/mtguard/NBridge->getClassLoader()Ljava/lang/ClassLoader;":{
                return vm.resolveClass("java/lang/ClassLoader").newObject(signature);
            }
            case "java/lang/ClassLoader->main2(I[Ljava/lang/Object;)Ljava/lang/Object;":{
                int num = vaList.getIntArg(0);
                System.out.println("fuck Num:"+num);
                switch (num){
                    case 1:{
                        return new StringObject(vm, "com.dianping.v1");
                    }
                    case 4:{
                        return new StringObject(vm, "ms_com.dianping.v1.png");
                    }
                    case 5:{
                        return new StringObject(vm, "ppd_com.dianping.v1.xbt");
                    }
                    case 2:{
                        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);// context
                        return context;
                    }
                    case 6:{
                        return new StringObject(vm, "5.4.11");
                    }
                    case 51:{
                        return new StringObject(vm, "2");
                    }
                    case 3:{
                        DvmObject<?> context = vm.resolveClass("android/content/Context").newObject(null);// context
                        return context;
                    }
                    case 8:{
                        return new StringObject(vm, "");
                    }
                    case 40:{
                        return new StringObject(vm, "");
                    }
                }
                break;
            }
            case "java/lang/ClassLoader->main1(I[Ljava/lang/Object;)Ljava/lang/Object;":{
                int num = vaList.getIntArg(0);
                System.out.println("fuck Num:"+num);
                switch (num){
                    case 49:{
                        return new StringObject(vm, "5.1.12");
                    }
                }
                break;
            }
            case "java/lang/System->getProperty(Ljava/lang/String;)Ljava/lang/String;":{
                String propertyKey = vm.getObject(vaList.getObjectArg(0).hashCode()).getValue().toString();
                System.out.println(propertyKey);
                switch (propertyKey){
                    case "http.proxyHost":{
                        return new StringObject(vm, "");
                    }
                    case "https.proxyHost":{
                        return new StringObject(vm, "");
                    }
                }
                break;
            }
            case "android/os/SystemProperties->get(Ljava/lang/String;)Ljava/lang/String;":{
                String propertyKey = vm.getObject(vaList.getObjectArg(0).hashCode()).getValue().toString();
                System.out.println(propertyKey);
                switch (propertyKey){
                    case "ro.build.id":{
                        return new StringObject(vm, "OPR6.170623.010");
                    }
                    case "persist.sys.usb.config":{
                        return new StringObject(vm, "diag,serial_cdev,rmnet,adb");
                    }
                    case "sys.usb.config":{
                        return new StringObject(vm, "ptp,adb");
                    }
                    case "sys.usb.state":{
                        return new StringObject(vm, "ptp,adb");
                    }
                }
                break;
            }
            case "java/util/UUID->randomUUID()Ljava/util/UUID;":{
                return vm.resolveClass("java/util/UUID").newObject(UUID.randomUUID());
            }
            case "java/lang/Long->valueOf(J)Ljava/lang/Long;":{
                return DvmLong.valueOf(vm, vaList.getLongArg(0));
            }
            case "java/lang/String->format(Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/String;":{
                return new StringObject(vm, String.format(vaList.getObjectArg(0).getValue().toString(), vaList.getIntArg(1)));
            }
        }
        return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);
    }
    @Override
    public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature){
            case "java/lang/ClassLoader->loadClass(Ljava/lang/String;)Ljava/lang/Class;":{
                return dvmObject.getObjectType();
            }
            case "android/content/pm/PackageManager->getApplicationInfo(Ljava/lang/String;I)Landroid/content/pm/ApplicationInfo;":{
                return new ApplicationInfo(vm);
            }
            case "java/util/UUID->toString()Ljava/lang/String;":{
                return new StringObject(vm, dvmObject.getValue().toString());
            }
        }
        return super.callObjectMethodV(vm, dvmObject, signature, vaList);
    }
    @Override
    public DvmObject<?> newObjectV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature){
            case "java/io/File-><init>(Ljava/lang/String;)V":{
                return vm.resolveClass("java/io/File").newObject(new File(vaList.getObjectArg(0).toString()));
            }
            case "java/lang/Integer-><init>(I)V":
                int input = vaList.getIntArg(0);
                return DvmInteger.valueOf(vm, input);
        }
        return super.newObjectV(vm, dvmClass, signature, vaList);
    }
    @Override
    public boolean callBooleanMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
        switch (signature){
            case "java/io/File->canRead()Z":{
                return false;
            }
        }
        return super.callBooleanMethodV(vm, dvmObject, signature, vaList);
    }
    @Override
    public DvmObject<?> getObjectField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
        switch (signature){
            case "android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;":{
                return new StringObject(vm, "/data/app/com.dianping.v1-BVuX2sFQfuNmRz85qE2NuA==/base.apk");
            }
        }
        return super.getObjectField(vm, dvmObject, signature);
    }
    @Override
    public int getStaticIntField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature){
            case "android/content/pm/PackageManager->GET_SIGNATURES:I":{
                return 64;
            }
        }
        return super.getStaticIntField(vm, dvmClass, signature);
    }
    @Override
    public int getIntField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
        switch (signature){
            case "android/content/pm/PackageInfo->versionCode:I":{
                return 1100110203;
            }
        }
        return super.getIntField(vm, dvmObject, signature);
    }
    @Override
    public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {
        switch (signature){
            case "android/os/Build->BRAND:Ljava/lang/String;":{
                return new StringObject(vm, "OPPO");//品牌
            }
            case "android/os/Build->TYPE:Ljava/lang/String;":{
                return new StringObject(vm, "user");
            }
            case "android/os/Build->HARDWARE:Ljava/lang/String;":{
                return new StringObject(vm, "OPM1.171019.011");//硬件
            }
            case "android/os/Build->MODEL:Ljava/lang/String;":{
                return new StringObject(vm, "OPPO R11st");//型号
            }
            case "android/os/Build->TAGS:Ljava/lang/String;":{
                return new StringObject(vm, "release-keys");
            }
            case "android/os/Build$VERSION->RELEASE:Ljava/lang/String;":{
                return new StringObject(vm, "9");
            }
        }
        return super.getStaticObjectField(vm, dvmClass, signature);
    }
    @Override
    public long callStaticLongMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
        switch (signature){
            case "java/lang/System->currentTimeMillis()J":{
                return System.currentTimeMillis();
            }
        }
        return super.callStaticLongMethodV(vm, dvmClass, signature, vaList);
    }
    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
                           这里成包名
        if (("/data/app/com.xxx.v1-BVuX2sFQfuNmRz85qE2NuA==/base.apk").equals(pathname)) {
            return FileResult.success(new SimpleFileIO(oflags, new File("unidbg-android\\src\\test\\java\\com\\xxx\\xxx\\base.apk"), pathname));
        }
        return null;
    }
}

```

```
mtgsig = {
      "a0":"2.1",
      "a1":"4069cb78-e02b-45f6-9f0a-b34ddccf389c",
      "a3":4,
      "a4":1641007666,
      "a5":"SWeM8hCznbxfdTxJtc0KHSzzKXqYFjfTl0UX+7wJk1DDJpB6VVJk/z/nOY4AgFnNPQA9BrwNNfDMYuaiAtIF65ZTLlIiSrFeSKTl1OZnpXzP1FvxE7IOT8lt9vqAkVjldJteCLcNt0hCVGHgpLtSixrX4nnLBa0xx6VDq8D7l+ZuDj6LwbwhfQrTywSliZrBFMNSw7QCBwufpLeZGln+I4Gg4xO4hB78x2QpnuGoGzCQ59Bvpfsb2DYwraeRb7Il",
      "a6":0,
      "a7":"5R12uldnp3hSXPr7QbCyuHdMCrky7li1vGm/mSt+YyAm7r5b5MqI/SlCC8BSeXZq7tAbvMxK3aXI7ZxNMosia7BaRbVILxHDVxFHrN2tRTI=",
      "a8":"DAD724885614A2FEFE4680911D321F89911FC9D29FA7F306DC6AEE1F",
      "a9":"90aa5e3cGg12eitjwSfAQq77re9bzPGk5gYYnXaseK0nggfQbPVktUMdgequOmVsVNUpcP1b0ceoAHDyorGl5cekKABAErse+EP0QwwIVhofmRHst0S59pj/B5h8htH+zosxunRrK0hNZJmvoh1xRBkKHq6jquk3GIKtPf7RXCXwXxN0yS3qqcdFZO2Wep4V8va5C/TkjB0YC6u4mJz/brXx3F/KYX3Z9PaGtyRGZpaRpgKBLxVUKJWA8dwT9MgNB466bfm3XrFyYnHw0sbrgDFaMM5rile4hvzHef+08TjJ3ycIOJT0kFLZOC4c3QVftWm31epg",
      "a10":"{}",
      "x0":1,
      "a2":"52516f82cfdd639bc05dbed46add0c59"
              }

```

上面是 undbg 调用出来的 mtgsig，会发现每次调用 a2, a4, a5, a7 都会不一样, 这样很难分析，所以我们第一件事先修改 undbg 中的时间获取函数，让函数返回固定值，看看有什么效果（并不是所有案例都可以）此图放错应修改 gettimeofday，可以在 src/main/java/com/github/unidbg/unix/UnixSyscallHandler.java 中修改

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxLqibsOcFwdbyMfR34SYdH6yWybgDUL311PC0WwzNmEPUbdbJPuCXueQ/640?wx_fmt=png)

这样修改后每次调用 a2, a4, a5, a7 都是固定的  

然后我们看 a5 这种密文，一看就是 base64 输出方式，那就 hook 住 base64 的函数验证下

**怎么找 base64 编码函数？  
**

老方法：ida 打开 shift + f12 搜索 ABCDE

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxju9pN1M2sXKPm1xic5ib2O1AWVibbib4KRgvYp5t0Umib5aicQkdPEP7W6ew/640?wx_fmt=png)

双击第一个结果  

快捷键 x 查看交叉引用

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxBKBphRTsyyThDGQvmRjXMyfjHkW7P3jrFMC6NgYKIf4xxt94zxOzkQ/640?wx_fmt=png)

双击第一个结果  

f5  

```
int __fastcall sub_B468(int a1, int a2, signed int a3)
{
  int v3; // r4
  int v4; // r0
  _BYTE *v5; // r6
  int v6; // r5
  int v7; // r2
  int v8; // r0
  int v9; // r4
  int v10; // r2
  char v11; // r1
  int v12; // r1
  int v13; // r0
  _DWORD *v14; // r0
  int v16; // [sp+4h] [bp-24h]
  signed int v17; // [sp+8h] [bp-20h]
  int v18; // [sp+Ch] [bp-1Ch]
  int v19; // [sp+10h] [bp-18h]
  char v20; // [sp+14h] [bp-14h]
  int v21; // [sp+18h] [bp-10h]
  v3 = a1;
  if ( !a2 || !a3 )
  {
    v12 = sub_370C(_stack_chk_guard) + 132;
    v13 = v3;
LABEL_13:
    sub_8C608(v13, v12);
    return _stack_chk_guard - v21;
  }
  v19 = a2;
  v18 = a1;
  v17 = a3;
  v4 = sub_98230(a3 + 2, 3u);
  v5 = malloc((4 * v4 | 1) + 1);
  if ( !v5 )
  {
    v12 = sub_370C(0) + 132;
    v13 = v3;
    goto LABEL_13;
  }
  v6 = 0;
  sub_2A514();
  v7 = v17;
  v16 = v5;
  v8 = v19;
  if ( v17 >= 3 )
  {
    v6 = 0;
    do
    {
      *v5 = aAbcdefghijklmn[*(v8 + v6) >> 2];
      v5[1] = aAbcdefghijklmn[(*(v8 + v6 + 1) >> 4) | 16 * *(v8 + v6) & 0x30];
      v5[2] = aAbcdefghijklmn[(*(v8 + v6 + 2) >> 6) | 4 * *(v8 + v6 + 1) & 0x3C];
      v5[3] = aAbcdefghijklmn[*(v8 + v6 + 2) & 0x3F];
      v8 = v19;
      v5 += 4;
      v6 += 3;
    }
    while ( v6 < v17 - 2 );
    v7 = v17;
  }
  if ( v6 >= v7 )
  {
    v14 = v3;
  }
  else
  {
    v9 = v7;
    *v5 = aAbcdefghijklmn[*(v8 + v6) >> 2];
    v10 = 16 * *(v8 + v6) & 0x30;
    if ( v6 == v9 - 1 )
    {
      v5[1] = aAbcdefghijklmn[v10];
      v11 = 61;
    }
    else
    {
      v5[1] = aAbcdefghijklmn[(*(v8 + v6 + 1) >> 4) | v10];
      v11 = aAbcdefghijklmn[4 * *(v8 + v6 + 1) & 0x3C];
    }
    v14 = v18;
    v5[3] = 61;
    v5[2] = v11;
    v5 += 4;
  }
  *v5 = 0;
  sub_8C8E0(v14, v16, &v5[~v16 + 1], &v20);
  free(v16);
  return _stack_chk_guard - v21;
}

```

这函数里有 base64 编码流程，看上去没魔改，编码的结果是放在 v5，v5=v16,v16 传给 sub_bc8e0, 但还是要验证下 base64 魔改没，a5 走不走这个函数也要验证

所以 hook 一把梭

hook sub_B468 和 sub_8C8E0

```
    public static void main(String[] args) {
        Mtgsig test = new Mtgsig();
        test.hookbase64();
        test.hookbase64_ret();
        test.callInit();      
        test.callMain();
    }

```

结果在运行日志里找到了 a5 的密文  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxBupsHkW6Tuvs9OOAPGiaYGIGib2bAdXrU8lrPvGxU1y6JkrgdMiaPjxAw/640?wx_fmt=png)

可以看到 base64 的入参是乱码，所以肯定是某种算法加密后的字节流，

把这个入参经过标准 base64 编码之后发现和 unidbg 算出来的结果一致，说明没有魔改  

所以下一步我们要顺藤摸瓜，找到这个某算法  

**定位某算法**

方法很多，我用 trace，因为 unidbg 的地址随机化是关闭的，也就是说，你模拟执行一千次，加密结果存放的内存地址是不会变的，所以可以监控某个地址块的读写操作，从而找到操作地址的地方

console debugger 在 0xB468 处打个断点

```
    public void HookByConsoleDebugger(){
        Debugger debugger = emulator.attach();
        debugger.addBreakPoint(module.base+0xB468);
    }

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxNIH1eSnJXYvDKAgGVeQgicFGXkFZ3lLicTlYk27wibEupyhwgvLQmcLRQ/640?wx_fmt=png)

可以看到 r1 寄存器存放的是某算法加密后的字节流，r2 是长度，那么我们 trace 这个地址的写入，看是哪个地址操作了这个地址

```
    public void HookByConsoleDebugger(){
        Debugger debugger = emulator.attach();
        debugger.addBreakPoint(module.base+0xB468);
        //trace a5 baseb4 输入的地址
        emulator.traceWrite(0x40335300L,0x40335300L+0xc0L);
    }

```

注意地址是 long 类型，不加 L 是 trace 不了的  

然后运行

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxRaPCvN4iaCDGKrCdBx4UO4yRfaxmWzCiarffNQeUK7IKfhica7SibWoujQ/640?wx_fmt=png)

有结果了，后边的地址就是我们要的东西

ida g 跳到 0x88fdc

```
int __fastcall sub_88F80(int result, _BYTE *a2, _BYTE *a3, signed int a4)
{
  int v4; // r4
  int v5; // r5
  char v6; // ST0C_1
  _BYTE *v7; // [sp+0h] [bp-28h]
  _BYTE *v8; // [sp+4h] [bp-24h]
  if ( a4 >= 1 )
  {
    v8 = (result + 257);
    v7 = (result + 256);
    do
    {
      LOBYTE(v4) = *v7 + 1;
      *v7 = v4;
      v4 = v4;
      LOBYTE(v5) = *v8 + *(result + v4);
      *v8 = v5;
      v6 = *(result + v4);
      v5 = v5;
      *(result + v4) = *(result + v5);
      *(result + v5) = v6;
      *a3 = *a2 ^ *(result + ((*(result + *v8) + *(result + *v7)) & 0xFF));
      --a4;
      ++a3;
      ++a2;
    }
    while ( a4 );
  }
  return result;
}

```

特征很明显，有经验的一看就知道是 rc4（boss 直聘那篇也是这个

hook

```
    public static void main(String[] args) {
        Mtgsig test = new Mtgsig();
        test.HookByConsoleDebugger();
        test.hookrc4();
        test.callInit();
        test.callMain();
    }

```

结果  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxKEAYpmibrE4EGZnnEbKyx33BQZbG9LtKGYGEwibzSLLlulicwUIZeibBBQ/640?wx_fmt=png)

入参开头是 789c， 是 zlib 压缩算法特征

```
import zlib
print(zlib.decompress(bytes.fromhex("789c9591bb0e83300c45ffc5338e72f382f22d594a2b550ca59dba20febd7696ba4895e88038d738768e5869028db45642a5b11257ea2a05c3d1706adc303744e3625a7ac343e3d4f8f4390a6f7a60f72298ae6838d9a66c3ed8d5514f88898ef7dd3f466c94f880121b253ee6c4568a7f5af14e6bdba8a329d0087945f5a229c90fbb3ceeee3a9f97e7bcdcdc0bda94a50cef7270c89a8b66782f4ff0512bbd8c29494a7d2945f2b0cba75d869711d92587b600f88e722b0ab4bd0143e08309")).decode())

```

现在明文知道了 还要知道 rc4 的 key，key 一般在另一个函数里，通过 key 生成 S 盒

sub_88F80 按 x 交叉引用 可以到这里

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSx3ctvACmD9HAo5wXiaT72QFPDbcGZrL8hRAbZLXr45WK7Lr26IsPPmQg/640?wx_fmt=png)

```
int __fastcall sub_88EEA(int a1, int key, int key_len, int a4)
{
  int ret; // r6
  int v5; // r0
  int v6; // r5
  int v7; // r4
  int v8; // r1
  int v9; // r0
  int v10; // r1
  int result; // r0
  int v12; // r4
  int v13; // r5
  int v14; // ST00_4
  int v15; // r1
  int v16; // r1
  int key1; // [sp+8h] [bp-18h]
  int v18; // [sp+Ch] [bp-14h]
  ret = a1;
  v5 = 0;
  do
  {
    *(ret + v5) = v5;
    ++v5;
  }
  while ( 256 != v5 );
  key1 = key;
  *(ret + 257) = 0;
  *(ret + 256) = 0;
  v18 = key_len;
  if ( a4 )
  {
    v6 = 0;
    v7 = 0;
    do
    {
      sub_982CC(v6, key_len);
      v9 = *(key1 + v8);
      v10 = *(ret + v6);
      key_len = v18;
      v7 = (v6 + v7 + v10 + v9) & 0xFF;
      *(ret + v6) = *(ret + v7);
      *(ret + v7) = v10;
      ++v6;
      result = 256;
    }
    while ( 256 != v6 );
  }
  else
  {
    v12 = 0;
    LOBYTE(result) = 0;
    do
    {
      v13 = *(ret + v12);
      v14 = result + v13;
      sub_982CC(v12, key_len);
      result = v14 + *(key1 + v15);
      v16 = (v14 + *(key1 + v15)) & 0xFF;
      *(ret + v12) = *(ret + v16);
      key_len = v18;
      *(ret + v16) = v13;
      ++v12;
    }
    while ( 256 != v12 );
  }
  return result;
}

```

hook sub_88EEA  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxUVvDV0MzFzVdn6NibxhX2pwtAkGxgUohbM9PaPgKNcQCGKAM4mTRFmw/640?wx_fmt=png)

key 是 mtgsig['a1'] + mtgsig['a3'] + mtgsig['a4']

打开逆向之友，看看是否魔改了

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxibHhibhqzsDjzDsmxtymwGlq1xfOqQe0OeicQgpUF1ySIYgu6Pqc5nebQ/640?wx_fmt=png)

oh shit 标准加密出来结果不对，经过验证，确实是魔改了

魔改的是密钥生成 S 的部分，huok 这个函数的返回值就是 S 盒，可以发现和标准算法生成的 S 盒不一样

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxKPiaHmO9lwITicsRRAADoKFDZMPx7CnWib919jReibqicvTRicWt68Zl57Aw/640?wx_fmt=png)

两个分支，因为我之前搞过 1.5 和 2.0 的， 我发现之前版本是只有下面的分支，下面的是标准 rc4   

现在因为 a4=1 所以走的上面的分支  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxiaCNnibmVtYXnLvgcTEOicIPSnXYoSblWNlianpLiaahnKYks5spIGowE8A/640?wx_fmt=png)

然后经过我野兽般的眼神和嗅觉，呵，不就这里多 + 了个变量吗

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxBfow7teWPibGWm5Vp6UJ3h0DoOodca4JEqOlyOuRqJrOzebobiby9OMQ/640?wx_fmt=png)

好了 知道了  

先搞一份 rc4 py 源码，rc4 原理这个作者也讲的很好

```
https://www.biaodianfu.com/rc4.html

```

```
import base64
import zlib
MOD = 256
def KSA(key):
    key_length = len(key)
    S = list(range(MOD)) 
    j=0
    for i in range(MOD):
        j = (j + S[i] + key[i % key_length]) % MOD
        S[i], S[j] = S[j], S[i] 
    return S
def PRGA(S):
    i = 0
    j = 0
    while True:
        i = (i + 1) % MOD
        j = (j + S[i]) % MOD
        S[i], S[j] = S[j], S[i]  # swap values
        K = S[(S[i] + S[j]) % MOD]
        yield K
def get_keystream(key):
    S = KSA(key)
    return PRGA(S)
def encrypt_logic(key, text)->bytes:
    if isinstance(key, bytes):
        key = [c for c in key]
    else:
        key = [ord(c) for c in key]
    keystream = get_keystream(key)
    res = []
    for c in text:
        val = c ^ next(keystream)
        res.append(val)
    return bytes(res)
def encrypt(key, plaintext)->bytes:
    if isinstance(plaintext,bytes):
        plaintext = [c for c in plaintext]
    else:
        plaintext = [ord(c) for c in plaintext]
    return encrypt_logic(key, plaintext)
def decrypt(key, ciphertext)->bytes:
    ciphertext = base64.b64decode(ciphertext.encode())
    res = encrypt_logic(key, ciphertext)
    return res

```

改秘钥 key 生成 S 盒的部分  

```
def KSA(key):
    key_length = len(key)
    S = list(range(MOD)) 
    j=0
    for i in range(MOD):
        #多加了个 i 看到没？？？
        j = (i(我是混进来的) + j + S[i] + key[i % key_length]) % MOD
        S[i], S[j] = S[j], S[i] 
    return S

```

完整 py 代码  

```
# -*- coding: utf-8 -*-
import base64
import zlib
MOD = 256
def KSA(key):
    key_length = len(key)
    # create the array "S"
    S = list(range(MOD))  # [0,1,2, ... , 255]
    j=0
    for i in range(MOD):
      #  print(hex(key[i % key_length]))
        j = (i + j + S[i] + key[i % key_length]) % MOD
        S[i], S[j] = S[j], S[i]  # swap values
    return S
def PRGA(S):
    i = 0
    j = 0
    while True:
        i = (i + 1) % MOD
        j = (j + S[i]) % MOD
        S[i], S[j] = S[j], S[i]  # swap values
        K = S[(S[i] + S[j]) % MOD]
        yield K
def get_keystream(key):
    S = KSA(key)
    return PRGA(S)
def encrypt_logic(key, text)->bytes:
    if isinstance(key, bytes):
        key = [c for c in key]
    else:
        key = [ord(c) for c in key]
    keystream = get_keystream(key)
    res = []
    for c in text:
        val = c ^ next(keystream)
        res.append(val)
    return bytes(res)
def encrypt(key, plaintext)->bytes:
    if isinstance(plaintext,bytes):
        plaintext = [c for c in plaintext]
    else:
        plaintext = [ord(c) for c in plaintext]
    return encrypt_logic(key, plaintext)
def decrypt(key, ciphertext)->bytes:
    ciphertext = base64.b64decode(ciphertext.encode())
    res = encrypt_logic(key, ciphertext)
    return res
def a5_decrypt(mtgsig):
    xina5key = f"{mtgsig['a1']}{mtgsig['a3']}{mtgsig['a4']}"
    a5_str = decrypt(xina5key, mtgsig['a5'])
    a5_str = zlib.decompress(a5_str).decode()
    print(a5_str)
    return a5_str
def a5_encrypt(mtgsig):
    xina5key = f"{mtgsig['a1']}{mtgsig['a3']}{mtgsig['a4']}"
    a5_str =r'{"b1":"{\"1\":\"-\",\"2\":\"-\",\"3\":\"-\",\"4\":\"\",\"5\":\"1\",\"6\":\"-\",\"7\":\"-\",\"8\":\"4\",\"9\":\"\",\"10\":\"-\",\"11\":\"-\",\"12\":\"\",\"13\":\"\",\"14\":\"-\",\"15\":\"\",\"16\":\"-\",\"33\":{\"0\":0,\"1\":\"-\",\"2\":\"-\",\"3\":\"-\",\"4\":\"-\",\"5\":\"-\",\"6\":\"-\",\"7\":\"-\",\"8\":\"-\",\"9\":\"-\",\"10\":\"-\",\"11\":\"-\",\"12\":\"-\",\"13\":\"-\",\"14\":\"-\",\"15\":\"-\",\"16\":\"-\"}}","b2":1,"b3":0,"b4":"com.dianping.v1","b5":"10.52.15","b6":"1100110203","b7":1641007666,"b8":1641007666,"b9":1641007666,"b10":"5.4.11","b11":"5.4.11","b12":"2"}'
    a5_str = zlib.compress(a5_str.encode())
    a5 = encrypt(xina5key,a5_str)
  #  hexdump(a5)
    print(base64.b64encode(a5).decode())
    return base64.b64encode(a5).decode()
def main():
    a5_decrypt(mtgsig)
    a5_encrypt(mtgsig)
if __name__ == '__main__':
    mtgsig = {"a0":"2.1",
              "a1":"4069cb78-e02b-45f6-9f0a-b34ddccf389c",
              "a3":4,
              "a4":1641007666,
              "a5":"SWeM8hCznbxfdTxJtc0KHSzzKXqYFjfTl0UX+7wJk1DDJpB6VVJk/z/nOY4AgFnNPQA9BrwNNfDMYuaiAtIF65ZTLlIiSrFeSKTl1OZnpXzP1FvxE7IOT8lt9vqAkVjldJteCLcNt0hCVGHgpLtSixrX4nnLBa0xx6VDq8D7l+ZuDj6LwbwhfQrTywSliZrBFMNSw7QCBwufpLeZGln+I4Gg4xO4hB78x2QpnuGoGzCQ59Bvpfsb2DYwraeRb7Il",
              "a6":0,
              "a7":"5R12uldnp3hSXPr7QbCyuHdMCrky7li1vGm/mSt+YyAm7r5b5MqI/SlCC8BSeXZq7tAbvMxK3aXI7ZxNMosia7BaRbVILxHDVxFHrN2tRTI=",
              "a8":"DAD724885614A2FEFE4680911D321F89911FC9D29FA7F306DC6AEE1F",
              "a9":"90aa5e3cGg12eitjwSfAQq77re9bzPGk5gYYnXaseK0nggfQbPVktUMdgequOmVsVNUpcP1b0ceoAHDyorGl5cekKABAErse+EP0QwwIVhofmRHst0S59pj/B5h8htH+zosxunRrK0hNZJmvoh1xRBkKHq6jquk3GIKtPf7RXCXwXxN0yS3qqcdFZO2Wep4V8va5C/TkjB0YC6u4mJz/brXx3F/KYX3Z9PaGtyRGZpaRpgKBLxVUKJWA8dwT9MgNB466bfm3XrFyYnHw0sbrgDFaMM5rile4hvzHef+08TjJ3ycIOJT0kFLZOC4c3QVftWm31epg",
              "a10":"{}",
              "x0":1,
              "a2":"52516f82cfdd639bc05dbed46add0c59"
              }
    main()

```

a5 完结，下篇讲讲 a9  a9 也是魔改了标准算法，应该是 Blowfish 或者 Twofishs 算法，不过因为对这算法不熟悉，愣是不知道魔改了哪里，一气之下 (走投无路![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxqP1jBmibofdMdWea65gblzUwVgYTe3n5jsjC0deJkx0Rl3wbuaHoJmg/640?wx_fmt=png)) 撸了一遍汇编，后面有空再研究研究算法

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxGMW7icptL9MV5FAC3OXNv8Dxh7x1KLOMuAic6nHtMf9roIWsicWzVLhicg/640?wx_fmt=png)

不过汇编更 6 了 ![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxQt7dBdXRpp0sWpEN0swibBH5jAtbby9gr2Kl4GuXq5OBcTibMpb4w2zA/640?wx_fmt=png)

翻译到三分之一我就觉得这样好傻逼 一部分简单指令可以自动化转成 py 代码，然后就加快了速度~

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSx3CPIwgPiaI394vE6NPCKrczl2F6Wem2iaLeicFT6wy7tOOSGvVMia2uFGg/640?wx_fmt=png)

哦还有 a2，a2 还是跟 mtgsig2.0 的一样

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4Bmp6OCwibuCVX7LWFIibicSxXNYrw3ukRg0NDiblf9stuBzvroLUjHrCibAibROWCnGRZQX6ibx4NOvXaQ/640?wx_fmt=png)