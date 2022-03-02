> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjA5NDE1OQ==&mid=2247485139&idx=1&sn=6875f205e642ef27880c8935e7756217&chksm=9f8bb7f3a8fc3ee592dc4320a8996179b975214319ad8ba1434dff2c3c90f6bbbcc26137dd6a&mpshare=1&scene=1&srcid=1030u3BXGR0GFJ5hM59qxMBf&sharer_sharetime=1635588940320&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3xBMR1a091Vb8Zxc7LcpA3Q6GDeLSF6NYbVPaAygKTuwSZh0Dk1IqPw/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH7Gwg06nY3t8zv0Pz9icqpy3wwia1uOgibv9l3txQaMYIUic4HpoefGKIy9d9xpGDgnBDcXmJEONBuNtQ/640?wx_fmt=jpeg)

```
int __fastcall bodyEncrypt(JNIEnv *a1, int a2, int a3, int a4, int a5, int a6, int a7, int a8, int a9, int a10)
{
  JNIEnv *v10; // r4
  int v11; // r6
  int v12; // r5
  int v13; // r11
  char *v14; // r9
  int v15; // ST14_4
  int v16; // r6
  int v17; // r10
  size_t v18; // r0
  int v19; // r8
  void *v20; // r5
  int v21; // r4
  int result; // r0
  int v23; // [sp+1Ch] [bp-24h]
  int v24; // [sp+20h] [bp-20h]
  v10 = a1;
  v11 = a9;
  v12 = a3;
  v13 = a7;
  if ( !a9 )
    v11 = ((*a1)->NewStringUTF)(a1, &unk_13ECB);
  if ( !a7 )
    v13 = ((*v10)->NewStringUTF)(v10, &unk_13ECB);
  if ( !v12 )
    v12 = ((*v10)->NewStringUTF)(v10, &unk_13ECB);
  v14 = ((*v10)->GetStringUTFChars)(v10, v12, 0);
  v15 = v11;
  v16 = ((*v10)->GetStringUTFChars)(v10, v11, 0);
  v17 = ((*v10)->GetStringUTFChars)(v10, v13, 0);
  v23 = 0;
  v18 = strlen(v14);
  v19 = j_tj_crypt(v14, v18, a5, a6, v17, a8, v16, a10, &v23);
  ((*v10)->ReleaseStringUTFChars)(v10, v12, v14);
  ((*v10)->ReleaseStringUTFChars)(v10, v13, v17);
  ((*v10)->ReleaseStringUTFChars)(v10, v15, v16);
  if ( v19 )
  {
    v20 = j_tjtxtutf8(v19, v23, 10);
    v21 = ((*v10)->NewStringUTF)(v10, v20);
    free(v20);
  }
  else
  {
    v21 = ((*v10)->NewStringUTF)(v10, &unk_13ECB);
  }
   result = _stack_chk_guard - v24;
  if ( _stack_chk_guard == v24 )
    result = v21;
  return result;
}

```

流程如下

```
v19 = j_tj_crypt(v14, v18, a5, a6, v17, a8, v16, a10, &v23);
v20 = j_tjtxtutf8(v19, v23, 10);
v21 = ((*v10)->NewStringUTF)(v10, v20);
result = v21;
return result;

```

v19 是经过加密函数 j_tj_crypt  加密后生成的 bytes，再经过 j_tjtxtutf8（上篇知道第三个参数为 10 的时候是标准的 base64）base64 编码  

可以打个断点验证下

j_tjtxtutf8：

0x38FC 下个断点

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4kBzMcllIeOMlWyRbvalfyvBhkSRRbxNX9aG2JVzbGNpvPAicTqbWcRcFv4riaoiawcDiaxr477qqSSQ/640?wx_fmt=png)

打印 r0 的值，长度是 0x31

mr0 0x31  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4kBzMcllIeOMlWyRbvalfyz6blLggE2nic2lUDZ4DSe4xhTHZ0xgj1JNHhBj76eQ1ru5nyWgEfQFQ/640?wx_fmt=png)

blr 添加函数结束后断点  

c 跳到下个断点

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4kBzMcllIeOMlWyRbvalfyME1ZsG6CLcS9Rwib9emVuoqL5mOia855LibWJFdULLYbONmJ0O4rpM9ww/640?wx_fmt=png)

逆向之友验证

```
https://gchq.github.io/CyberChef

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4kBzMcllIeOMlWyRbvalfyK73yjet1iafSVibtlwvGUoHlxiaribZskWia2IoODgHRzzY4YlSQvRkQQIQ/640?wx_fmt=png)

没毛病，接下来看 j_tj_crypt 是怎么生成 v19 的

进到 tj_crypt 函数

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4QXibVXWOQF6KOgCffrcxgKsAzVOA46uNDEI857Cg9ibn2F9Mfjtlvib2eYErqhQdG1kdUwiczXMAkww/640?wx_fmt=png)

看到熟悉的 key 了

然后这些 "1","2","3" 是代表加密模式

stmcmp 是 c 函数

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4QXibVXWOQF6KOgCffrcxgKGNjtslMXkPyEDYCNrQtRss9XtQ6kXmz0QuU3DEyCzEXmjetmvRJhcA/640?wx_fmt=png)

也就是根据第一个参数的值才决定用哪种加密

我们可以在 unidbg 上修改第一个参数的值，可以看到三种加密的结果是不一样的

```
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

```

当 String arg0_str = "1"  

结果为

```
2uEIVLrtvmHQ72DYG1+fFcCSo5obEd62UMKP+O7zFRx+K6u3FhLY0f3gYdZds/BMRA==

```

当 String arg0_str = "2"

结果为

```
NO3dVW0Fr4yk5Xxoc64Nn903e6sSZK3Uif1TB8dHmVVlXXqzMuteseCb4hfd1/WUQFyHPmxIIRM86pCkymjlgQ==

```

当 String arg0_str = "3"

结果为

```
HAT5BvN81B8PjW69r1sqCvf0cwN/Pey9XoAWBnkuGYEeVpqWTDBuyei8kaZ00XTJig/SKP4qThHwuGgWmlr4fg==

```

开始逐一击破  

**加密模式 1**

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH4QXibVXWOQF6KOgCffrcxgK5jqNJqBMPAYwU7VHXCVkqEbxjlibJqNib3MBWj2QGnozQYSAS3d2ILEQ/640?wx_fmt=png)

strncmp "1" 和 v10 比较，v10=1 所以相等返回 0，!0 则是 true

所以跳到 LABEL_17

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI2icu8ccmwjNcPWEUo2ZLn2qqxtD5KUYjTGvrN4pbQ2VVQVJkicXoLYLA/640?wx_fmt=png)

有三个函数，分别是 sub_3880、sub_302C、j_CCCrypt

显而易见，加密逻辑在 j_CCCrypt 函数里，而且因为最后 return 的 v11=v22，v22 就是放解密结果的地方

hook CCCrypt:

```
    public void hook_CCCrypt(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x4480 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int arg0 = ctx.getIntArg(0);
                int arg1 = ctx.getIntArg(1);
                int arg2 = ctx.getIntArg(2);
                Pointer arg3 = ctx.getPointerArg(3);
                byte[] arg3_ = arg3.getByteArray(0, ctx.getIntArg(4));
                int arg4 = ctx.getIntArg(4);
                int arg5 = ctx.getIntArg(5);
                String arg6 = ctx.getPointerArg(6).getString(0);
                int arg7 = ctx.getIntArg(7);
                Pointer arg8 = ctx.getPointerArg(8);
                int arg9 = ctx.getIntArg(9);
                System.out.println("arg0:"+arg0);
                System.out.println("arg1:"+arg1);
                System.out.println("arg2:"+arg2);
                Inspector.inspect(arg3_, "arg3");
                System.out.println("arg4:"+arg4);
                System.out.println("arg5:"+arg5);
                System.out.println("arg6:"+arg6);
                System.out.println("arg7:"+arg7);
                ctx.push(arg8);
                ctx.push(arg9);
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length=ctx.pop();
                Pointer output = ctx.pop();
                byte[] outputhex = output.getByteArray(0, length);
                Inspector.inspect(outputhex, "CCCrypt ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

hook 结果  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIcMAtV76Nico7rLvuial1xlcvvvrQXOBicUluYW9qDkYj0tT8oPsWSOK8g/640?wx_fmt=png)

大胆一点猜，arg3(v4) 就是密钥，是 sub_302C 函数生成的

arg6(a7) 就是明文，是什么加密还不知道，反正 99% 是对称加密

这时候是先看密钥怎么生成的还是先找出是什么加密算法呢？

都可以，我喜欢倒着推，所以先看 CCCrypt，双击进入函数

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHtr5I62m4IGmyKLF5auKcldIiaiaw6uet42WmvN665y62K6eC63j5dT1g/640?wx_fmt=png)

显而易见  j_CCCryptorCreate 是加密的初始化函数

双击进去 

```
int __fastcall CCCryptorCreate(int a1, unsigned int a2, void *a3, int a4, int a5, int a6, _DWORD *a7)
{
  unsigned int v7; // r5
  void **v8; // r8
  int v9; // r9
  void *v10; // r11
  int v11; // r10
  int v12; // r4
  size_t v14; // r4
  _BYTE *v15; // r0
  int v16; // r0
  void *v17; // [sp+10h] [bp-28h]
  size_t v18; // [sp+14h] [bp-24h]
  int v19; // [sp+18h] [bp-20h]
  v11 = a1;
  v12 = 0xFFFFEF34;
  if ( a7 )
  {
    v7 = a2;
    if ( a2 <= 5 )
    {
      v9 = a4;
      v10 = a3;
      v8 = off_1EB14[a2];
      v12 = (*v8)(a1);
      if ( !v12 )
        goto LABEL_6;
    }
  }
  while ( _stack_chk_guard != v19 )
  {
LABEL_6:
    v17 = v10;
    v14 = v18 + 20;
    v18 = v14;
    v15 = malloc(v14);
    if ( v15 )
    {
      v10 = v15;
      *(v15 + 1) = v14;
      *(v15 + 2) = v11;
      *(v15 + 3) = v7;
      *(v15 + 4) = v8;
      *v15 = 1;
      v16 = (v8[1])(v15 + 20, v11, v7, v17, v9, a5, a6);
      if ( v16 )
      {
        v12 = v16;
        free(v10);
      }
      else
      {
        v12 = 0;
        *a7 = v10;
      }
    }
    else
    {
      v12 = -4302;
    }
  }
  return v12;
}

```

这里伪代码看着有点懵？看上去好像没函数? 但是这里不对劲？  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHVdqQTCaGsI2Mlb8elWicib3dBjVufVurFMKPbnBLRCbmpDeNHibuyk5yg/640?wx_fmt=png)

这里 v8[1] 应该是个函数地址？后边是参数？

那 v8[1] 的地址是多少呢？

按 tab，转成汇编再看看

```
.text:00004274                 EXPORT CCCryptorCreate
.text:00004274 CCCryptorCreate                         ; CODE XREF: j_CCCryptorCreate+8↑j
.text:00004274                                         ; DATA XREF: LOAD:00000450↑o ...
.text:00004274
.text:00004274 anonymous_0     = -0x34
.text:00004274 var_28          = -0x28
.text:00004274 var_24          = -0x24
.text:00004274 var_20          = -0x20
.text:00004274 anonymous_1     =  8
.text:00004274 arg_8           =  0x10
.text:00004274
.text:00004274 ; __unwind {
.text:00004274                 PUSH            {R4-R7,LR}
.text:00004276                 ADD             R7, SP, #0xC
.text:00004278                 PUSH.W          {R1-R11}
.text:0000427C                 MOV             R10, R0
.text:0000427E                 LDR             R0, =(__stack_chk_guard_ptr - 0x4284)
.text:00004280                 ADD             R0, PC  ; __stack_chk_guard_ptr
.text:00004282                 LDR             R6, [R0] ; __stack_chk_guard
.text:00004284                 LDR             R0, [R6]
.text:00004286                 STR             R0, [SP,#0x38+var_20]
.text:00004288                 LDR             R0, =0xFFFFEF32
.text:0000428A                 ADDS            R4, R0, #2
.text:0000428C                 LDR             R0, [R7,#arg_8]
.text:0000428E                 CBZ             R0, loc_42B2
.text:00004290                 MOV             R5, R1
.text:00004292                 CMP             R1, #5
.text:00004294                 BHI             loc_42B2
.text:00004296                 LDR             R0, =(off_1EB14 - 0x42A2) ; r0=0x1EB14 - 0x42A2 = 0x1A872
.text:00004298                 MOV             R9, R3
.text:0000429A                 MOV             R11, R2
.text:0000429C                 ADD             R2, SP, #0x38+var_24
.text:0000429E                 ADD             R0, PC  ; off_1EB14
.text:000042A0                 MOV             R1, R5
.text:000042A2                 LDR.W           R8, [R0,R5,LSL#2]
.text:000042A6                 MOV             R0, R10
.text:000042A8                 LDR.W           R3, [R8]
.text:000042AC                 BLX             R3
.text:000042AE                 MOV             R4, R0
.text:000042B0                 CBZ             R0, loc_42C8
.text:000042B2
.text:000042B2 loc_42B2                                ; CODE XREF: CCCryptorCreate+1A↑j
.text:000042B2                                         ; CCCryptorCreate+20↑j ...
.text:000042B2                 LDR             R0, [R6]
.text:000042B4                 LDR             R1, [SP,#0x38+var_20]
.text:000042B6                 SUBS            R0, R0, R1
.text:000042B8                 ITTTT EQ
.text:000042BA                 MOVEQ           R0, R4
.text:000042BC                 ADDEQ           SP, SP, #0x1C
.text:000042BE                 POPEQ.W         {R8-R11}
.text:000042C2                 POPEQ           {R4-R7,PC}
.text:000042C4                 BLX             __stack_chk_fail
.text:000042C8 ; ---------------------------------------------------------------------------
.text:000042C8
.text:000042C8 loc_42C8                                ; CODE XREF: CCCryptorCreate+3C↑j
.text:000042C8                 LDR             R0, [SP,#0x38+var_24]
.text:000042CA                 STR.W           R11, [SP,#0x38+var_28]
.text:000042CE                 ADD.W           R4, R0, #0x14
.text:000042D2                 STR             R4, [SP,#0x38+var_24]
.text:000042D4                 MOV             R0, R4  ; size
.text:000042D6                 BLX             malloc
.text:000042DA                 CBZ             R0, loc_4312
.text:000042DC                 MOV             R11, R0
.text:000042DE                 LDRD.W          R1, R0, [R7,#8]
.text:000042E2                 MOVS            R2, #1
.text:000042E4                 STRD.W          R4, R10, [R11,#4]
.text:000042E8                 STRD.W          R5, R8, [R11,#0xC]
.text:000042EC                 STRB.W          R2, [R11]
.text:000042F0                 MOV             R2, R5
.text:000042F2                 LDR.W           R4, [R8,#4]
.text:000042F6                 LDR             R3, [SP,#0x38+var_28]
.text:000042F8                 STRD.W          R9, R1, [SP]
.text:000042FC                 MOV             R1, R10
.text:000042FE                 STR             R0, [SP,#0x38+anonymous_0+4]
.text:00004300                 ADD.W           R0, R11, #0x14
.text:00004304                 BLX             R4
.text:00004306                 CBZ             R0, loc_4316
.text:00004308                 MOV             R4, R0
.text:0000430A                 MOV             R0, R11 ; ptr
.text:0000430C                 BLX             free
.text:00004310                 B               loc_42B2
.text:00004312 ; ---------------------------------------------------------------------------

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHLibOUuywicYraD9dRD8laMIwXA5tTxATMSJnwymibNc4GicPtXdERIr1Ag/640?wx_fmt=png)

按 tab 键，可以看到对应的汇编是这行，

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHTRmUOtGk4tqESIdrNcuicCr3wbfLFGqyGT0ibiaKrkRsbHb06Kiawb16jg/640?wx_fmt=png)

汇编: BLX  R4

```
指令BLX 
指令的格式为：BLX 目标地址BLX 
指令从ARM 指令集跳转到指令中所指定的目标地址，
并将处理器的工作状态有ARM 状态切换到Thumb 状态。

```

显而易见，这里是动态跳转的，也就是说 R4 的地址是计算出来的，不是写死的，所以我们得知道 R4 是多少；

这里有两种方法获得 R4 的值：

第一种方法很简单，用 unidbg 或 frida inline hook 0x4304，看 r4 寄存器的值不就行了嘛？

第二种方法就是静态分析，手动计算出来不就行了嘛？

先挑战一下第二种方法看看能出来不？

R4 哪来的？  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI87A4xEjgiaU8GGejzWEWVQ3MR0ZFYgWd6ibPGQDpL3uSS5qzCgpBicZaQ/640?wx_fmt=png)

```
.text:000042F2                 LDR.W           R4, [R8,#4]
即：R4 = [R8 + 0x4]  []代表取R8+R4的指针

```

R8 哪来的？

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI6m5BT2GpRSibWFBkVUkvBiawSfn89Ro86vm2iabOtj1RAM8OAABQL3qIQ/640?wx_fmt=png)

```
.text:000042A2                 LDR.W           R8, [R0,R5,LSL#2]
即：R8 = [R0 + (R5<<2)] ，[]代表取R0+(R5<<2)的指针

```

R0 哪来的？

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWISIqsvQH2AKMqibKyOiaiaotlOAU68uibbpCibzjz5Rz866X6HQZWJXibfXJQ/640?wx_fmt=png)

```
.text:0000429E                 ADD             R0, PC  ; off_1EB14
即：R0 = R0+PC   这里ida已经算出R0=0x1EB14

```

R5 哪来的？

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIkrhQeHJMhxPdNCiaFiaAtV4TryI8fcLguInQs5pKGlDfqjssCJanA9aw/640?wx_fmt=png)

```
.text:00004290                 MOV             R5, R1
R5=R1 也就是函数的第二个参数,最后追到tj_crypt函数里的v25=v17 = 4
    if ( !strncmp("1", v10, n) )
    {
      v16 = v9;
      v17 = 4;
LABEL_15:
      v25 = v17;
      v9 = 0;
      goto LABEL_17;
    }

```

流程：

```
.text:00004290                 MOV             R5, R1
.text:0000429E                 ADD             R0, PC  ; off_1EB14
.text:000042A2                 LDR.W           R8, [R0,R5,LSL#2]
.text:000042F2                 LDR.W           R4, [R8,#4]

```

R5 = 4

R0 = 0x1EB14

R8 = [R0+(4<<2)] = [0x1EB24]

R4 = [R8+4]

[0x1EB24] 表示取 0x1EB24 的指针，也就是指向的地址，怎么看

Ida 中按 g 输入 0x1EB24，回车

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAH3WLgG9zbRtxfZwbDkhTgglrp6GicRvXiajwZz6nSIXRV3Hh8ObXYBiaog/640?wx_fmt=png)

双击 ccRC4Callouts

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHiaiaXwyVxYfyPkianZKAMLiapjv1rVbF6rm6xP1KTJlb00iaJAYmO4xMCdA/640?wx_fmt=png)

0x1EC3C 就是 0x1EB24 的指针

所以 R8 = 0x1EC3C

最后 R4 = [R8+0x4] = [0x1EC3C+4] = [0x1EC40]

[0x1EC40] 表示取 0x1EC40 的指针，就是 0x4E89

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHzsyynEYWxdP5ibQ7eJIwhz6PgP3PR73GjkxIpuUapT2QgHiaE1OcSSXQ/640?wx_fmt=png)

第二种获取 R4 地址得方法很简单

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIj4nZUQ8htC2icoKPzysfWHUnqzMVMSiavMWueHAZ9LO1MCJ3kwlxdvrA/640?wx_fmt=png)

hook 0x4304

```
debugger.addBreakPoint(module.base+0x4304);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWINvsgACiclz64F7KfgaEsRhz4J9XGJJzwibOAcXTMic3TW5mH70OxLcFibQ/640?wx_fmt=png)

笑死，直接就出来了，还静态分析个毛

所以 BLX R4 最终等价于 BLX 0x4E89

ida 中按 g 输入 0x4E89 跳转，也可以双击 sub_4E88

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6j2r8xlLLamev2aSYplQAHdicYKANK8auPia4lxBG80u95hRR6VykM329Z9icL2sgAaooSTsRkiauu3Q/640?wx_fmt=png)

咦，显而易见这是 rc4 算法初始化密钥的函数

Hook 这个函数就能拿到 rc4 的密钥

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIJc2I6FjBcWHdW4DYv0L6jVricXwWCG7BibxUzqX6nsA2pkUibkFHTS6icw/640?wx_fmt=png)

打个断点  

```
debugger.addBreakPoint(module.base+0xC244);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI8rIqXibJZ4oDJ1Ctd40NGySPjaenU86VUziaS1PUORpNVCblbOEuq4vA/640?wx_fmt=png)

查看 r2 的值

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIiadPh4NUmKC5CKGf7QCic6Az2jDXnOQxDlKPuGlGzShY1jOibwMeoTyrQ/640?wx_fmt=png)

密钥是：f4c7fc5d61bdff1914c383b340958ef1

没毛病吧！就是上面说的 V9

然后最终加密的函数呢？

回到 CCCrypt 函数  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIibnoBdVtAFZhziaxBiaaaVKPzvQLV8X0yHy6gT65dbAec26ibS5ttxCEqw/640?wx_fmt=png)

```
v12 = (*(*(v21 + 16) + 12))(v21 + 20);

```

有经验了，这一行一看跟上面差不多嘛，看看汇编  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIZU7YQaibDCBNWF1HFeO7C0uqZTAuFoCQNkTr8P3fI4o0d2zmpuB1Stg/640?wx_fmt=png)

BLX R4

R4 地址又是算出来的，直接用 unidbg hook 拿到地址

```
debugger.addBreakPoint(module.base+0x453A);

```

运行  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWInU9QhOmP3e7g3JoDB8WB2OF1Wv33a8tp99VzgrW2oL5Um2tCYCxh3Q/640?wx_fmt=png)

BLX R4 等价于 BLX 0x4ea5 

ida 中按 g 输入 0x4ea5  跳转

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIaaJ3iaT1a3vVIKPnLhVOibMnbmCMmlocibc05h4JKble1bZfl1OVGeYNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFWdgIeHCd7j1zZ6fotroN4VECNrEdW4bo0Nicca7kZlMmHvyYib3ZCZTw/640?wx_fmt=png)

二话不说 hook 这个函数  

```
    public void hook_CC_RC4(){
        //rc4密钥
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0xBEC8 + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer arg0 = ctx.getPointerArg(3);
                int arg1 = ctx.getIntArg(1);
                Pointer arg2 = ctx.getPointerArg(2);
                System.out.println("arg1:"+arg1);
                System.out.println("arg2:"+arg2.getString(0));
                ctx.push(arg0);
                ctx.push(arg1);
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                int length=ctx.pop();
              //  System.out.println(length);
                Pointer output = ctx.pop();
                byte[] outputhex = output.getByteArray(0, length);
                Inspector.inspect(outputhex, "CC_RC4 ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

hook 结果：  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIBNVtvI6I0NEOicMqbhAEFiaic0P4NJ9qep1OatsGGceXuSCoxnB56c0ng/640?wx_fmt=png)

看来前面猜的都没错嘛,

现在就用逆向之友看看这个 rc4 是不是标准的 rc4 了  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWICAOibwcFYaB36C8tyCOp5vPBDTqkw2Zx2A43AZrRX2De6jV4r2jOVNA/640?wx_fmt=png)

没毛病  

现在明文知道了，也就是 bodyencrypt 的 第五个参数，

加密方式也知道了，对称算法 rc4

最后就要解决密钥是怎么生成的

**rc4 key**

回到 tj_crypt，LABEL_17

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWINIxPfMmcPs9uBZgIjC2WVa80u5E2NsLqK5DEG2ZQ81QwosYyp5GJNw/640?wx_fmt=png)

显而易见！

我们要进入 sub_302 的身体里去探索

```
char *__fastcall sub_302C(int a1, int a2, int a3, int a4)
{
  int v4; // r8
  char *result; // r0
  int v6; // r4
  int v7; // r9
  char *v8; // r6
  v4 = a1;
  result = 0;
  if ( a2 )
  {
    v6 = a4;
    if ( a4 )
    {
      v7 = a3;
      v8 = malloc(0x15u);
      result = 0;
      *(v8 + 13) = 0;
      *v8 = 0LL;
      *(v8 + 1) = 0LL;
      *(v8 + 17) = 0;
      if ( v8 )
      {
        j_CCHmac(0, v7, v6, v4);
        sub_33E4(v8, 20);
        result = v8;
      }
    }
  }
  return result;
}

```

先 hook 一下，看看参数和结果

```
    public void hook_sub_302C(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x302C + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer arg0 = ctx.getPointerArg(0);
                int arg1 = ctx.getIntArg(1);
                int arg3 = ctx.getIntArg(3);
                Pointer arg2 = ctx.getPointerArg(2);
                System.out.println("arg0:"+arg0.getString(0));
                System.out.println("arg1:"+arg1);
                System.out.println("arg2:"+arg2.getString(0));
                System.out.println("arg3:"+arg3);
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer output = ctx.getR0Pointer();
                byte[] outputhex = output.getByteArray(0, 128);
                Inspector.inspect(outputhex, "sub_302C ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

显而易见是个 hmac 哈希编码

hook 结果：  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAF0c21kFm3IJ5TQ7LM9wpj0U2Kz3uJeHTXWCE6N53SqFNicUABN9Yu8LQ/640?wx_fmt=png)

arg0 是明文，可以发现是由 arg2_str+arg1 拼接而来

哪个函数拼接的？

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFA7CX5jKTeRIGhkcXC6XdUkb07qJBMmkTPjRnDeficPTN4siahAoYLdsA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFVEeRgTzNJVicv8zuq2I3yliby7TIZPic6yIS1JHh14MQvKHicRh4W0d4eg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFwjsmdJpO4DxKthicibAQZDpjGqeHNnxIcWQdnrQmdcuvvyDhTjUEQ4QQ/640?wx_fmt=png)

arg2 是密钥，结果是长度 20 个字节（00 00 00 00 是结束标志），可以盲猜是 hmacsha1，然后去逆向之友看看是不是 hmacsha1

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI6skLQEicEscQJcLfUtsOfiadayLuTBW0YN10oSokFPfGxatQfXm1bl5A/640?wx_fmt=png)

咦，不对呢，但是可以发现 hmacsha1 的结果出现在这里

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI595uBNN3TOW4fYKE0Vm5sCyCl1Vhsa9R9ibdiaKGibTyaDkvb5kd2MdDQ/640?wx_fmt=png)

j_CCHmac 下面还有个 sub_33E4(v8, 20)

应该是 sub_33E4 搞得鬼

hook hook 看

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIXqpRnJT9QvYug0ylfx221mkHa0ibLOXU9Nlf5Gb3GZdFcfNHk31dE5g/640?wx_fmt=png)

这次没错了

如果猜不出是 hmacsha1，那就进入 j_CCHmac、CCHmac

```
int __fastcall CCHmac(int a1, int a2, int a3, int a4, int a5, int a6)
{
  int v6; // r11
  int v7; // r4
  char v9; // [sp+4h] [bp-194h]
  int v10; // [sp+Ch] [bp-18Ch]
  void (__fastcall *v11)(int *, int, int); // [sp+160h] [bp-38h]
  int v12; // [sp+184h] [bp-14h]
  int v13; // [sp+188h] [bp-10h]
  v13 = v6;
  v7 = a4;
  j_CCHmacInit(&v9, a1, a2, a3);
  v11(&v10, v7, a5);
  j_CCHmacFinal(&v9, a6);
  return _stack_chk_guard - v12;
}

```

显而易见，又是个 init、update、Final 流程，在中卷已经说了，忘了的回去看看

进入 j_CCHmacInit、CCHmacInit

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIgnHqqHv9H8C5hb8xWP3LynDicrNkzNn1iabzibibxLXcqs7YsGeYWDWJzQ/640?wx_fmt=png)

看到有个控制流，根据 a2 的值来决定走哪个分支

这里 a2 是 0

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIAU615QtliaVOLECuN8gUX8zpvFVWEUWh7NicEHb5dSQM30F94bCOh6mQ/640?wx_fmt=png)

所以进到这里  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIsfSibOcw7dfexkWNKiae829B5nSlXZe7lztic3Z4v93u3g0NvlE8npJibA/640?wx_fmt=png)

可以看到初始化常量

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIJ56hG6Y0gUgKfFM6zciam7kjc3WTia2hyT2adFzy0OvUicW1P9PK3A8OQ/640?wx_fmt=png)

回到上面  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIAXdlXwGAwMh8IqtMMOkoj1EH6hvvuibywp4KUZsBnibKNhCuaRhZJAaw/640?wx_fmt=png)

这里的 v11 又是动态跳转，要看的自己 hook 下，不在细说

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWI7mZkpYQ5pstYNyuWDw9MgRTrQETfVdb57c4D0JVlviatfprluJS3ib1w/640?wx_fmt=png)

再回到上面，解决这个函数

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6TasXnTSrq21cYUkEIRGWIyGicV25ZgUMPmfrqRerMPBxvrz9gQudLKjykfMNpLFiaR8XicQ4cCGkaQ/640?wx_fmt=png)

进入 sub_33E4 函数

```
const char *__fastcall sub_33E4(const char *result, unsigned int a2)
{
  unsigned int v2; // r6
  const char *v3; // r9
  _BYTE *v4; // r4
  int v5; // r2
  int v6; // r3
  signed int v7; // r2
  const char *v8; // r0
  const char *v9; // r1
  int v10; // r3
  char v11; // t1
  char v12; // t1
  int v13; // r2
  _BYTE *v14; // r0
  int v15; // r1
  _BYTE *v16; // r3
  char v17; // r5
  if ( a2 >= 2 )
  {
    v2 = a2;
    v3 = result;
    result = strlen(result);
    if ( result >= v2 )
    {
      v4 = malloc(v2 + 1);
      _aeabi_memclr(v4, v2 + 1, v5, v6);
      v7 = 1;
      v8 = &v3[v2 - 1];
      v9 = v3;
      while ( 1 )
      {
        v10 = &v4[v7];
        if ( v9 >= v8 )
          break;
        v11 = *v9++;
        *(v10 - 1) = v11;
        v12 = *v8--;
        v4[v7] = v12;
        v7 += 2;
      }
      if ( v8 == v9 )
        *(v10 - 1) = *v9;
      v13 = v2 - 1;
      v14 = v3 + 1;
      v15 = 0;
      while ( 1 )
      {
        v16 = &v4[v15];
        if ( &v4[v15] >= &v4[v13] )
          break;
        v17 = v4[v13--];
        *(v14 - 1) = v17;
        ++v15;
        *v14 = *v16;
        v14 += 2;
      }
      if ( v13 == v15 )
        *(v14 - 1) = v4[v15];
      result = j_free(v4);
    }
  }
  return result;
}

```

就是对 hmacsha1 编码后的 byte 移位

看着 ida 反编译的伪代码不好翻译，可以伪代码为主，汇编为辅助去还原；

也可以汇编为主，伪代码和 unidbg 动态调试为辅助去还原；  

都不会的还有个更简单的，只需要多搞几组数据对比下，找出规律自己实现就行了；

放出伪代码为主，汇编辅助的代码

```
import hashlib
import hmac
from hexdump import hexdump
def CCHmac(key:str,input_str:str)->bytes:
    hmacsha1_ret = hmac.new(key.encode(), input_str.encode(), hashlib.sha1).digest()
    return hmacsha1_ret
def sub_33E4(hmacsha1_ret:bytes,size=20)->bytes:
    v4 = [0 for i in range(size)]
    v5 = 1
    v6 = size
    v7 = 0
    while (1):
        if (v7 >= v6):
            break
        v9 = hmacsha1_ret[v7]
        v7 += 1
        v4[v5 - 1] = v9
        v6 -= 1
        v10 = v6
        v4[v5] = hmacsha1_ret[v10]
        v5 += 2
    if v6 == v7:
        hmacsha1_ret = v4
    v11 = size
    v13 = 0
    v12 = 1
    v4 = [0 for i in range(size)]
    while (1):
        if (v13 >= v11):
            break
        v11 -= 1
        v10 = v11
        v4[v12 - 1] = hmacsha1_ret[v10]
        v9 = hmacsha1_ret[v13]
        v13 += 1
        v4[v12] = v9
        v12 += 2
    return bytes(v4)[:16]
if __name__ == '__main__':
    input_str = "YM0A2TMAIEWA3xMAMM1xTjQEUYGD0wZAYAh32AdgQATDzAZNZA33WAEIZAzzhAMMNAzyTDAkZMzTizYOYN11TTgMRcDThyNMZO44GTIAVQGDi5OONN2wWGMMUVWj1iMNOYxxGmUU1630808396"
    Hmackey = "dGp******dG8K"
    v9 = CCHmac(Hmackey,input_str)
    key = sub_33E4(v9,20)
    hexdump(key)

```

 rc4 自己写  

**加密模式 2**

**其实后面两种加密模式流程都差不多，简单说一下吧。。文章已经太长了**

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAF483jRbCkDwI6Kr6LrCltWjIjdbXP5Q9RRbsDibRrNDo0RFm0Ou33rgg/640?wx_fmt=png)

这里改成 2

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFUviaZp1wVJ1EUPMWJAssrGgbwribKbbqNcDrM821DDJNfwfkHOxkUpeA/640?wx_fmt=png)

然后根据上面模式 1 分析的可以知道这个 v25 就是 R5（就忘了？划上去看）

R5=v25=v17=0

R8, [R0,R5,LSL#2]

即：R8=[R0+(0<<2)]=[0x1EB14]

同理可得 0x1EB14 的指针是 0x1EB2C

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFKRHTicjeGdLAlItMB4EjVA5wtPkeJNPP5rxGBdoOvAWMmtgf1Hnt8hw/640?wx_fmt=png)

R4=[R8+4]

最后 R4=[R8+0x4]=[0x1EB2C+4] = [0x1EB30]

[0x1EB30] 表示取 0x1EB30 的指针，就是 0x4829

不信的话用 unidbg 打个断点  

还是这个地方

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFdObEqtXsf9icow6BW3WicIkVAicLz3c5pae1pSEbe8ic3V9KV4MHBEgZEw/640?wx_fmt=png)

```
debugger.addBreakPoint(module.base+0x4304);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFKzniakxtVKtDd4qSR5JSe6eSw0IQ4cHBibVuNyW01PDdrYfjib9Opicc5Q/640?wx_fmt=png)

废话不多说，直接 g 0x4829 跳过去

```
signed int __fastcall sub_4828(int a1, int a2, int a3, char a4, int a5, unsigned int a6, int a7)
{
  int v7; // r5
  char v8; // r8
  int v9; // r6
  char *v10; // r0
  signed int v11; // r4
  int *v12; // r6
  char v13; // r3
  int v14; // r12
  v7 = a1;
  v8 = a4;
  v9 = a2;
  v10 = sub_4DE0(a3);
  v11 = -4300;
  if ( a5 && v10 && *((_DWORD *)v10 + 2) <= a6 && *((_DWORD *)v10 + 3) >= a6 )
  {
    *(_DWORD *)v7 = v10;
    if ( v9 == 1 )
    {
      v12 = (int *)(v10 + 40);
      *(_DWORD *)(v7 + 4) = *((_DWORD *)v10 + 8);
      v13 = 0;
    }
    else
    {
      if ( v9 )
        return v11;
      v12 = (int *)(v10 + 36);
      *(_DWORD *)(v7 + 4) = *((_DWORD *)v10 + 7);
      v13 = 1;
    }
    v14 = *v12;
    *(_BYTE *)(v7 + 49) = v8 & 1;
    *(_BYTE *)(v7 + 48) = v13;
    *(_DWORD *)(v7 + 8) = v14;
    if ( v8 & 2 )
    {
      *(_WORD *)(v7 + 50) = 0;
    }
    else
    {
      *(_BYTE *)(v7 + 50) = 1;
      if ( v10[16] )
        *(_BYTE *)(v7 + 51) = 0;
      else
        *(_BYTE *)(v7 + 51) = 1;
    }
    *(_DWORD *)(v7 + 28) = 0;
    if ( !(*((int (__fastcall **)(int))v10 + 5))(v7 + 52) )
    {
      if ( *(_BYTE *)(v7 + 50) )
        sub_4E0C((unsigned __int8 *)v7, (int *)a7);
      v11 = 0;
    }
  }
  return v11;
}

```

这函数里只有 sub_4DE0 跟 sub_4E0C 两个函数，sub_4DE0 很短没什么信息

那肯定就再 sub_4E0C 函数里了

进去

```
int __fastcall sub_4E0C(unsigned __int8 *a1, int *a2)
{
  int *v2; // r3
  JNIEnv *v3; // r2
  int v4; // r1
  int v5; // r2
  int v7; // [sp+0h] [bp-30h]
  __int64 v8; // [sp+8h] [bp-28h]
  __int64 v9; // [sp+10h] [bp-20h]
  __int64 v10; // [sp+18h] [bp-18h]
  int v11; // [sp+24h] [bp-Ch]
  v2 = a2;
  v3 = *a1;
  if ( *(*a1 + 16) )
  {
    if ( !a2 )
    {
      v2 = &v7;
      *&v7 = 0LL;
      v8 = 0LL;
      v9 = 0LL;
      v10 = 0LL;
    }
    (v3[6])(a1 + 52, a1[48], v2);
  }
  else
  {
    v4 = (a1 + 12);
    if ( !a1[48] )
      v4 = (a1 + 32);
    v5 = *(v3 + 4);
    if ( v2 )
      _aeabi_memmove(v4, v2, v5);
    else
      _aeabi_memclr(v4, v5, v5, 0);
  }
  return _stack_chk_guard - v11;
}

```

(v3[6])(a1 + 52, a1[48], v2);

这一行很熟悉了吧，看汇编

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFqa9cUCWia9oCwLdYjvA5tcgebE2cjV4FpVyO2UtEiaZzOW6d4hKu0rtg/640?wx_fmt=png)

直接 hook，静态分析个毛，早 hook 早发表

```
debugger.addBreakPoint(module.base+0x4E40);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFQdE78wCiaIicaduGMzbDo42SRIqaBXTvia8sPWibJiczNN8o6Mib3ey81RIw/640?wx_fmt=png)

g 0x11a51

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFMOupiaLXRRLXRadsMyoWBZhzdiberfgUeqgFic2JNzicwT3uHpVOPHxbOw/640?wx_fmt=png)

显而易见，这是 aes 算法，设置 iv 的地方

hook

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAF8rOl0Gw48PR2XTiaC2hlfrrO7TXDsbhsoYtr8QmTicJSVDPHUd3zU9Ew/640?wx_fmt=png)

```
debugger.addBreakPoint(module.base+0x11A50);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFyFGbgFFfYYLAMDAHgNdqK8ibuIXrHpdMMsNdHPxNiaY96icx2OFRUlKRg/640?wx_fmt=png)

打印 iv

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFRG1RwPKr38Ktq3u4SltaZZ4jUChypAgLaVicfQmZ8D4m7Vzfw88by6Q/640?wx_fmt=png)

那 aes 的 key 是哪里初始化的？一般是先初始化 key 再 iv，前面是不是漏掉了什么关键的地方? 回去看看

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFQ6rLySzo59RmEZibHibLreQn0xZg1MCFaEWjQ8giciakpialuoPPOUXQMEg/640?wx_fmt=png)

噢这里还有这 BLX

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFdRaqiaRV1DPW93MhanCu0VeeeIK6CuYskq6IEBHSyLiclv1kA82Uv69w/640?wx_fmt=png)

hook 之

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFAtueKZ16fKAibseL85b2jHzgco1L73EBV2h1xUa88KtEZa3KUDnibNYg/640?wx_fmt=png)

ida g 0x119e9

unidbg b 0x119e9   

c

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFLgCCicUWgx8NAFWzmhc9tAvZyh8L0yKLuzribnM8cRJiaSDz1bqdQiawZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFy0jfHOobB6LNMP689mQZK9wXHNX7Zt8tDBPKq54XQVRW73xSniaIVJw/640?wx_fmt=png)

进去

hook 之

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAF0icOM8bUgNUACodPD8rcRmMgrjToxsAprlrkQ56jriciaC6SvcnYfYykQ/640?wx_fmt=png)

b0xE398 

c  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFx2Wd9qzWbeuHFtBcibG5vQ7ibT945mBiasWVHPibVKicibI2Lz2hWBc38ePQ/640?wx_fmt=png)

打印 r0

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFXnlH0YJXVVuowFgczhb6F71wC3mYWhPdDsQFLZjh0t4Vdq5m0gtq7Q/640?wx_fmt=png)

可以看到这个 key 跟 rc4 的 key 是一样的

```
加密方式:aes_cbc
明文:{"code":null,"parameter":{"activityTask":{},"defa
aes_key:f4c7fc5d61bdff1914c383b340958ef1
aes_iv:00000000000000000000000000000000

```

逆向之友验证

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFPy5VAknia6kDibVyrYRCw8b1NwqBCva0ib2xIyyUD4qF6EFooPkdLcG3A/640?wx_fmt=png)

没毛病

还想知道 aes_encrypt 在哪的

hook 这里

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFCqw8zIzbZYRGRZtVcicbFc9ib7yb7tOZFYuTJqnzLOT23ojtK4fTEcibA/640?wx_fmt=png)

拿到 R4 地址  

再 hook 这里   

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFVXmV3hUvTia9fT50qkiaBLFkHYlxeCyHXzVUhkx4AbnZ2rXC3PupfEPg/640?wx_fmt=png)

拿到 R5 地址

就是了  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFm7LhibqcZhZgjuia37P9jwZFESye2PoDAXrccPb0MAOolxRFW74ZNUzA/640?wx_fmt=png)

**加密模式 3**

**跟模式 2 一样是 aes_cbc, 只是 iv 不一样，所以只说 iv 的生成**

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFWdj4PYsP4jK5yrODlN4Btficia24K3wicMI8gu1rOMKOjkFicLDN4q50pA/640?wx_fmt=png)

改成 3

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFT5SEPYbUuXYf9OvHWiajCaKMaf3Kl2YYHefdlicQ1rptboJIe3fRWl5g/640?wx_fmt=png)

看样子就是比模式二多走了一个 sub_302c，

```
public void hook_sub_302C(){
        IHookZz hookZz = HookZz.getInstance(emulator); // 加载HookZz
        hookZz.wrap(module.base + 0x302C + 1, new WrapCallback<HookZzArm32RegisterContext>() {
            @Override
            // 方法执行前
            public void preCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer arg0 = ctx.getPointerArg(0);
                int arg1 = ctx.getIntArg(1);
                int arg3 = ctx.getIntArg(3);
                Pointer arg2 = ctx.getPointerArg(2);
                System.out.println("arg0:"+arg0.getString(0));
                System.out.println("arg1:"+arg1);
                System.out.println("arg2:"+arg2.getString(0));
                System.out.println("arg3:"+arg3);
            };
            @Override
            // 方法执行后
            public void postCall(Emulator<?> emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
                Pointer output = ctx.getR0Pointer();
                byte[] outputhex = output.getByteArray(0, 128);
                Inspector.inspect(outputhex, "sub_302C ret");
            }
        });
        hookZz.disable_arm_arm64_b_branch();
    }

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFiaTVWJyps8q6gbMmdnk6bTicGVs3doauo8gqg7Eo6IfhCpnCaPSu8wQQ/640?wx_fmt=png)

然后后边的流程就跟模式二就没区别了，就不说了

hook aes_cc_set_iv  

```
debugger.addBreakPoint(module.base+0x11A50);

```

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFUV3efpPAwLZZibDoDnlwraj0o6Kia9a6MZDEVCwicAEaAoAtGJHk1ibXGA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFISIPad3msBcucwHTDU6lQdnhmhUlIicvuyuQlLGs9AFGzY3Hb4o9o0Q/640?wx_fmt=png)

可以看到这个 iv 就是第一次 sub_302c 生成的值

这个 iv 和 key 的生成方式都是一样的，只是明文少了个时间戳而已

```
加密方式:aes_cbc
明文:{"code":null,"parameter":{"activityTask":{},"defa
aes_key:f4c7fc5d61bdff1914c383b340958ef1
aes_iv:7b53e03485ddbbcf07bd9698718abc8a

```

逆向之友验证  

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFJmY27BCqict3TuRMbtiac1EwaKq3wFrrsRU0A9I6qp1dDaVvDtgGXpHQ/640?wx_fmt=png)

这些都是标准的算法，遇到魔改算法的时候，就需要对算法的原理非常熟悉了，这时就需要看这个男人的密码学专栏![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFq4QzFib1kyspVcM3vKMgBeMpibmicPOGMznxribZZvM63aGs71P9g24Qvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFb6P5cAm09YK4FXXmeQsdoK5kFtUQibrEnKJgPBwPtoia6OTlnUYy6s3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAFMbJr6icmbiallGwbX4rjWuLJeBPSyug2vxIpL0YzULgmH3GlLbVnfSNA/640?wx_fmt=jpeg)

****完结！****  

**饮茶去了**

![](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH6Tsm0MiceegNzEQmZCyXVAF3ctIAJQuhYibVRPyjnmTiclzkxEPr8HMgL3kTnM8TgK3tJibF2X6SB6qA/640?wx_fmt=png)