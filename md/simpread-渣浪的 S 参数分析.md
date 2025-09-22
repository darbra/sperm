> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/R_tay_9MUavrBZVE8Z0L6Q)

**初始化操作**

```
public void nativeInit(){

String M = "10EA095010";
String N = "10EA095010";
String O = "10EA095060";

String nativeInit = "nativeInit(Ljava/lang/String;)V";
SLib.callStaticJniMethod(emulator, nativeInit, M);
}

```

**S 参数的计算**

```
public void nativeNewCalculateS(){
Objectcustom =null;
DvmObjectcontext = vm.resolveClass("android/content/ContextWrapper").newObject(custom);
    System.out.println(context);

Stringstr ="2015876998797";
intmode =3;
StringnativeNewCalculateS ="nativeNewCalculateS(Landroid/content/Context;Ljava/lang/String;I)Ljava/lang/String;";
Objectobj = SLib.callStaticJniMethodObject(emulator, nativeNewCalculateS, context, str, mode);
    System.out.println(obj);
}

```

10EA095010 为初始化参数，2015876998797 为需要进行 HASH 的字符串。

**Unidbg 的运行日志如下:**

```
1.	JNIEnv->FindClass(com/sina/weibo/security/SLib) was called from RWX@0x120026d4[libslib.so]0x26d4
2.	JNIEnv->RegisterNatives(com/sina/weibo/security/SLib, RWX@0x12050008[libslib.so]0x50008, 8) was called from RWX@0x1200259c[libslib.so]0x259c
3.	RegisterNative(com/sina/weibo/security/SLib, nativeInit(Ljava/lang/String;)V, RWX@0x120023b8[libslib.so]0x23b8)
4.	RegisterNative(com/sina/weibo/security/SLib, nativeNewCalculateS(Landroid/content/Context;Ljava/lang/String;I)Ljava/lang/String;, RWX@0x1200241c[libslib.so]0x241c)
5.	RegisterNative(com/sina/weibo/security/SLib, nativeGetV1TimeCost()J, RWX@0x12002080[libslib.so]0x2080)
6.	RegisterNative(com/sina/weibo/security/SLib, nativeGetV2TimeCost()J, RWX@0x12002150[libslib.so]0x2150)
7.	RegisterNative(com/sina/weibo/security/SLib, nativeCheck()Z, RWX@0x120022e4[libslib.so]0x22e4)
8.	RegisterNative(com/sina/weibo/security/SLib, nativeGetCheckErr()Ljava/lang/String;, RWX@0x12001e08[libslib.so]0x1e08)
9.	RegisterNative(com/sina/weibo/security/SLib, nativeGetCheckErrType()I, RWX@0x12001fb8[libslib.so]0x1fb8)
10.	RegisterNative(com/sina/weibo/security/SLib, nativeGetPkgTimeCost()J, RWX@0x12002218[libslib.so]0x2218)
11.	Find native function Java_com_sina_weibo_security_SLib_nativeInit => RWX@0x120023b8[libslib.so]0x23b8
12.	JNIEnv->NewGlobalRef("10EA095010") was called from RWX@0x120023f0[libslib.so]0x23f0
13.	android.content.ContextWrapper@50675690
14.	Find native function Java_com_sina_weibo_security_SLib_nativeNewCalculateS => RWX@0x1200241c[libslib.so]0x241c
15.	JNIEnv->FindClass(android/content/ContextWrapper) was called from RWX@0x120046a0[libslib.so]0x46a0
16.	JNIEnv->GetMethodID(android/content/ContextWrapper.getPackageManager()Landroid/content/pm/PackageManager;) =>0x53f2c391 was called from RWX@0x12004440[libslib.so]0x4440
17.	JNIEnv->FindClass(android/content/pm/PackageManager) was called from RWX@0x120047f8[libslib.so]0x47f8
18.	JNIEnv->CallObjectMethodV(android.content.ContextWrapper@50675690, getPackageManager() => android.content.pm.PackageManager@47d384ee) was called from RWX@0x12004af8[libslib.so]0x4af8
19.	JNIEnv->FindClass(android/content/pm/PackageInfo) was called from RWX@0x120049ec[libslib.so]0x49ec
20.	JNIEnv->GetMethodID(android/content/pm/PackageManager.getPackageInfo(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;) =>0x3bca8377 was called from RWX@0x12004700[libslib.so]0x4700
21.	JNIEnv->GetMethodID(android/content/ContextWrapper.getPackageName()Ljava/lang/String;) =>0x8bcc2d71 was called from RWX@0x12004964[libslib.so]0x4964
22.	JNIEnv->CallObjectMethodV(android.content.ContextWrapper@50675690, getPackageName() => "com.sina.weibo") was called from RWX@0x12004af8[libslib.so]0x4af8
23.	JNIEnv->CallObjectMethodV(android.content.pm.PackageManager@47d384ee, getPackageInfo("com.sina.weibo", 0x40) => android.content.pm.PackageInfo@2acf57e3) was called from RWX@0x12004af8[libslib.so]0x4af8
24.	JNIEnv->GetFieldID(android/content/pm/PackageInfo.signatures [Landroid/content/pm/Signature;) =>0x25f17218 was called from RWX@0x12004794[libslib.so]0x4794
25.	JNIEnv->GetObjectField(android.content.pm.PackageInfo@2acf57e3, signatures [Landroid/content/pm/Signature; => [android.content.pm.Signature@36f6e879]) was called from RWX@0x12004618[libslib.so]0x4618
26.	JNIEnv->FindClass(android/content/pm/Signature) was called from RWX@0x1200473c[libslib.so]0x473c
27.	JNIEnv->GetMethodID(android/content/pm/Signature.toByteArray()[B) =>0x6a3e2031 was called from RWX@0x120045ac[libslib.so]0x45ac
28.	JNIEnv->GetObjectArrayElement([android.content.pm.Signature@36f6e879], 0) => android.content.pm.Signature@36f6e879 was called from RWX@0x12004844[libslib.so]0x4844
29.	JNIEnv->CallObjectMethodV(android.content.pm.Signature@36f6e879, toByteArray() => [B@3551a94) was called from RWX@0x12004af8[libslib.so]0x4af8
30.	JNIEnv->GetByteArrayElements(false) => [B@3551a94 was called from RWX@0x12003f74[libslib.so]0x3f74
31.	JNIEnv->NewStringUTF("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1200e97c[libslib.so]0xe97c
32.	JNIEnv->NewGlobalRef("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1200e9a4[libslib.so]0xe9a4
33.	JNIEnv->GetStringUtfChars("2015876998797") was called from RWX@0x1203b48c[libslib.so]0x3b48c
34.	JNIEnv->ReleaseStringUTFChars("2015876998797") was called from RWX@0x1203b390[libslib.so]0x3b390
35.	JNIEnv->GetStringUtfChars("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1203b48c[libslib.so]0x3b48c
36.	JNIEnv->ReleaseStringUTFChars("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1203b390[libslib.so]0x3b390
37.	JNIEnv->GetStringUtfChars("10EA095010") was called from RWX@0x1203b48c[libslib.so]0x3b48c
38.	JNIEnv->ReleaseStringUTFChars("10EA095010") was called from RWX@0x1203b390[libslib.so]0x3b390
39.	JNIEnv->NewStringUTF("a17486b9") was called from RWX@0x1200ec2c[libslib.so]0xec2c
40.	"a17486b9"

```

**主要的内容如下:**

```
1.	JNIEnv->NewStringUTF("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1200e97c[libslib.so]0xe97c
2.	JNIEnv->NewGlobalRef("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1200e9a4[libslib.so]0xe9a4
3.	JNIEnv->GetStringUtfChars("2015876998797") was called from RWX@0x1203b48c[libslib.so]0x3b48c
4.	JNIEnv->ReleaseStringUTFChars("2015876998797") was called from RWX@0x1203b390[libslib.so]0x3b390
5.	JNIEnv->GetStringUtfChars("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1203b48c[libslib.so]0x3b48c
6.	JNIEnv->ReleaseStringUTFChars("5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo") was called from RWX@0x1203b390[libslib.so]0x3b390
7.	JNIEnv->GetStringUtfChars("10EA095010") was called from RWX@0x1203b48c[libslib.so]0x3b48c
8.	JNIEnv->ReleaseStringUTFChars("10EA095010") was called from RWX@0x1203b390[libslib.so]0x3b390
9.	JNIEnv->NewStringUTF("a17486b9") was called from RWX@0x1200ec2c[libslib.so]0xec2c
10.	"a17486b9"

```

在 0xe97c 处计算得出了 5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo 字符串

5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo 是由内置的字符串 OWA8W1RiZGVVOHxGOzU4R0VGO157OUo4OVpUazV 经过 sub_34F40 解密得出的

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscAmibHyHEib6ibsk2UqG5Q1mvFKJby8rWB1AfXN2ibVDapxEEdcxV5icwoaQ/640?wx_fmt=other&from=appmsg#imgIndex=0)

在 0x3b48c 处引用了需要进行 HASH 的值 2015876998797  
  

在 0x3b48c 处引用了解密的值 5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo  
  

在 0x3b48c 处引用了初始化的值 10EA095010

计算 S 参数的结果是 a17486b9 在 0xec2c 进行字符串生成的

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscSaz36GrfDHvibaRJDcDYDkJxHzEKDMibyCw1fwtGUh2VmuoD7icwZ0GRA/640?wx_fmt=other&from=appmsg#imgIndex=1)

对 loc_EC1C 进行分析，在 0xEC1C 下断点

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscv6sUYiabsfia6bMXcUOiclLsj0p4YCoA97yicLv5xxRAu7ahZvEfKmODcg/640?wx_fmt=other&from=appmsg#imgIndex=2)

0xe4fff631 存储的就是 HASH 后的结果

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscckxmm4XDGqmSyibNicc2yhoHhKbHE3wCHdZHAiaFoibCgzFFuBR7bU6aYw/640?wx_fmt=other&from=appmsg#imgIndex=3)

对 0xe4fff631 - 0xe4fff638 下读写断点

```
public void write_trace(){
        emulator.traceRead(0xe4fff631L, 0xe4fff638L);
        emulator.traceWrite(0xe4fff631L, 0xe4fff638L);
    }

```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXschTicG96ba5Jk27CHMYV7gdMq7lpibPibxSHjQUWCHETrRaCp6Uos5PJnw/640?wx_fmt=other&from=appmsg#imgIndex=4)

在 0xe780 的地址进行了结果的赋值

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscIShR4AuWic8Iwe8p3sj7GFA6sc1icNFGBzYCvEdsfRLBk5MmTREicrUuw/640?wx_fmt=other&from=appmsg#imgIndex=5)

```
LDRX9, [SP,#0x180+var_D8]
LDURQ0, [SP,#0x180+var_E8]
MOVW8, WZR
STRX9, [X28,#0x10]
ADDX9, SP, #0x180+var_E8
STRQ0, [X28] ; 0x396236383437316110


```

Q0 的值来自 [SP,[#0x180](javascript:;)+var_E8] 地址为 0xE4FFF618

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscJpHV19y1eReic8bh8VAALxU1uRP6paMyLhwOwVFINBxEdMOMogNgJOQ/640?wx_fmt=other&from=appmsg#imgIndex=6)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscJvpVJwtdWMPMRdSHoWZeET3eLGRYu6XEo3FicoC447ZI07aibAPiaA35w/640?wx_fmt=other&from=appmsg#imgIndex=7)

对 0xE4FFF618 - 0xE4FFF620 下读写断点

```
public void write_trace(){
    emulator.traceRead(0xe4fff631L, 0xe4fff638L);
    emulator.traceWrite(0xe4fff631L, 0xe4fff638L);

    emulator.traceRead(0xe4fff618L, 0xe4fff620L);
    emulator.traceWrite(0xe4fff618L, 0xe4fff620L);
}

```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscwRJNQnfNmlicvbv6ba8NTI4ftpibfGjVwZ6IPBqI80Ps4B2ic7Mwj1ChQ/640?wx_fmt=other&from=appmsg#imgIndex=8)

生成结果的地址在 0x8314 就是计算 HASH 的函数 sub_75F0

```
void __usercall sub_75F0(__int64 a1@<X0>, constchar *a2@<X1>, __int64 a3@<X2>, __int64 *a4@<X8>){
constchar *v5; // x9
constchar *v7; // x20
int v8; // w23
constchar *v9; // x8
int v10; // w10
int v11; // w12
int v12; // w13
constchar *v13; // x22
bool v14; // zf
int v15; // w16
constchar *v16; // x0
int v17; // w15
size_t v18; // x0
  _QWORD *v19; // x8
  _QWORD *v20; // x0
constchar *v21; // x1
  _QWORD *v22; // x9
  _QWORD *v23; // x8
int v24; // w10
int v25; // w16
void *v26; // x21
int v27; // w15
int v28; // w16
constchar *v29; // x0
int v30; // w15
size_t v31; // x0
  _QWORD *v32; // x8
  _QWORD *v33; // x0
constchar *v34; // x1
  _QWORD *v35; // x9
  _QWORD *v36; // x8
int v37; // w10
int v38; // w16
  _QWORD *v39; // x20
int v40; // w15
int v41; // w16
constchar *v42; // x0
int v43; // w15
size_t v44; // x0
  _QWORD *v45; // x8
  __int64 *v46; // x0
size_t v47; // x2
int v48; // w16
int v49; // w18
  __int64 *v50; // x1
int v51; // w17
int v52; // w12
int v53; // w15
int v54; // w16
  __int64 v55; // x0
  __int128 v56; // q0
unsignedint v57; // w12
int v58; // w13
int v59; // w8
char *v60; // x0
constchar *v61; // x1
int v62; // w10
int v63; // w16
char *v64; // x20
int v65; // w15
int v66; // w16
constchar *v67; // x0
int v68; // w15
size_t v69; // x0
char *v70; // x0
constchar *v71; // x1
int v72; // w10
int v73; // w16
char *v74; // x20
int v75; // w15
constchar *v76; // x0
int v77; // w15
size_t v78; // x0
int v79; // w15
int v80; // w14
int v81; // w16
size_t v82; // x0
size_t v83; // x20
  __int64 v84; // x10
unsigned __int64 v85; // x26
int v86; // w27
int v87; // w22
int v88; // w8
unsigned __int64 v89; // x10
int v90; // w8
int v91; // w11
int v92; // w9
  __int64 v93; // x8
int v94; // w23
int i; // w9
int v96; // w8
int v97; // w8
int v98; // w8
int v99; // w8
int v100; // w8
int v101; // w8
unsigned __int64 v102; // [xsp+10h] [xbp-2B0h]
char v103; // [xsp+20h] [xbp-2A0h]
int v104; // [xsp+24h] [xbp-29Ch]
  __int64 v105[2]; // [xsp+78h] [xbp-248h] BYREF
void *v106; // [xsp+88h] [xbp-238h]
  __int128 v107; // [xsp+90h] [xbp-230h] BYREF
void *v108; // [xsp+A0h] [xbp-220h]
void *src[3]; // [xsp+B0h] [xbp-210h] BYREF
void *v110[3]; // [xsp+C8h] [xbp-1F8h] BYREF
size_t v111[2]; // [xsp+E0h] [xbp-1E0h] BYREF
void *v112; // [xsp+F0h] [xbp-1D0h]
unsigned __int64 v113; // [xsp+F8h] [xbp-1C8h]
unsignedint v114; // [xsp+100h] [xbp-1C0h]
char s[12]; // [xsp+104h] [xbp-1BCh] BYREF
  _BYTE v116[132]; // [xsp+110h] [xbp-1B0h] BYREF
  _BYTE v117[132]; // [xsp+194h] [xbp-12Ch] BYREF
  __int64 v118; // [xsp+218h] [xbp-A8h] BYREF
  __int64 v119; // [xsp+220h] [xbp-A0h]
  __int64 v120[9]; // [xsp+228h] [xbp-98h] BYREF

  v120[8] = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  v5 = *(constchar **)(a1 + 16);
  v7 = a2;
  v8 = -474758095;
  v9 = (constchar *)(a1 + 1);
if ( ((*(_BYTE *)a1 ^ 0xFE) & *(_BYTE *)a1) != 0 )
    v10 = 1056576168;
else
    v10 = 529704559;
  v11 = -474758095;
do
  {
while ( 1 )
    {
      v12 = v11;
      v13 = a2;
      v11 = 1499283521;
while ( v12 <= 529704558 )
      {
        v14 = v12 == -474758095;
        v12 = v10;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v12 != 529704559 )
break;
      a2 = (constchar *)(a1 + 1);
    }
    a2 = *(constchar **)(a1 + 16);
  }
while ( v12 == 1056576168 );
  v15 = -474758095;
do
  {
while ( 1 )
    {
      v16 = a2;
      v17 = v15;
while ( v17 <= 529704558 )
      {
        v14 = v17 == -474758095;
        v17 = v10;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v17 != 529704559 )
break;
      a2 = v9;
      v15 = 1499283521;
    }
    a2 = v5;
    v15 = 1499283521;
  }
while ( v17 == 1056576168 );
  v18 = strlen(v16);
sub_32C14(v19, v13, v18, &v118);              // MD5
  v20 = sub_8620(v111, &v118, 16LL);
  v118 = 0LL;
  v119 = 0LL;
  v22 = *(_QWORD **)(a3 + 16);
  v23 = (_QWORD *)(a3 + 1);
if ( (~*(_BYTE *)a3 | 0xFFFFFFFE) == 0xFFFFFFFF )
    v24 = 529704559;
else
    v24 = 1056576168;
  v25 = -474758095;
do
  {
while ( 1 )
    {
      v26 = v20;
      v27 = v25;
while ( v27 <= 529704558 )
      {
        v14 = v27 == -474758095;
        v27 = v24;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v27 != 529704559 )
break;
      v20 = v23;
      v25 = 1499283521;
    }
    v20 = v22;
    v25 = 1499283521;
  }
while ( v27 == 1056576168 );
  v28 = -474758095;
do
  {
while ( 1 )
    {
      v29 = v21;
      v30 = v28;
while ( v30 <= 529704558 )
      {
        v14 = v30 == -474758095;
        v30 = v24;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v30 != 529704559 )
break;
      v21 = (constchar *)v23;
      v28 = 1499283521;
    }
    v21 = (constchar *)v22;
    v28 = 1499283521;
  }
while ( v30 == 1056576168 );
  v31 = strlen(v29);
sub_32C14(v32, v26, v31, &v118);              // MD5
  v33 = sub_8620(v110, &v118, 16LL);
  v118 = 0LL;
  v119 = 0LL;
  v35 = (_QWORD *)*((_QWORD *)v7 + 2);
  v36 = v7 + 1;
if ( ((*v7 ^ 0xFE) & *v7) != 0 )
    v37 = 1056576168;
else
    v37 = 529704559;
  v38 = -474758095;
do
  {
while ( 1 )
    {
      v39 = v33;
      v40 = v38;
while ( v40 <= 529704558 )
      {
        v14 = v40 == -474758095;
        v40 = v37;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v40 != 529704559 )
break;
      v33 = v36;
      v38 = 1499283521;
    }
    v33 = v35;
    v38 = 1499283521;
  }
while ( v40 == 1056576168 );
  v41 = -474758095;
do
  {
while ( 1 )
    {
      v42 = v34;
      v43 = v41;
while ( v43 <= 529704558 )
      {
        v14 = v43 == -474758095;
        v43 = v37;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v43 != 529704559 )
break;
      v34 = (constchar *)v36;
      v41 = 1499283521;
    }
    v34 = (constchar *)v35;
    v41 = 1499283521;
  }
while ( v43 == 1056576168 );
  v44 = strlen(v42);
sub_32C14(v45, v39, v44, &v118);              // MD5
sub_8620(src, &v118, 16LL);
  v46 = sub_6F20(v120, src);
if ( (~LOBYTE(v111[0]) | 0xFFFFFFFE) == 0xFFFFFFFF )
    v48 = 529704559;
else
    v48 = 1056576168;
  v49 = -474758095;
do
  {
while ( 1 )
    {
      v50 = v46;
      v51 = v49;
while ( v51 <= 529704558 )
      {
        v14 = v51 == -474758095;
        v51 = v48;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v51 != 529704559 )
break;
      v46 = (__int64 *)((char *)v111 + 1);
      v49 = 1499283521;
    }
    v46 = (__int64 *)v112;
    v49 = 1499283521;
  }
while ( v51 == 1056576168 );
if ( (v111[0] & 1) != 0 )
    v52 = 1552367478;
else
    v52 = 132625284;
  v53 = -1135239934;
do
  {
while ( 1 )
    {
      v54 = v53;
if ( v53 <= 132625283 )
break;
if ( v53 == 132625284 )
      {
        v53 = -1840677486;
        v47 = (unsigned __int64)LOBYTE(v111[0]) >> 1;
      }
else
      {
        v53 = -1840677486;
        v47 = v111[1];
      }
    }
    v53 = v52;
  }
while ( v54 == -1135239934 );
if ( v54 != -1840677486 )
  {
while ( 1 )
      ;
  }
  v55 = sub_9538((int)v120, v50, v47);
  v108 = *(void **)(v55 + 16);
  v56 = *(_OWORD *)v55;
  v57 = 0;
  v113 = v55;
  v107 = v56;
while ( 1 )
  {
    v114 = v57;
    v58 = v57 >= 3 ? -1129254591 : 1805009445;
if ( v58 == -1129254591 )
break;
    *(_QWORD *)(v113 + 8LL * v114) = 0LL;
    v57 = v114 + 1;
  }
if ( ((LOBYTE(v120[0]) ^ 0xFE) & v120[0]) != 0 )
    v59 = 1202870824;
else
    v59 = -946146110;
while ( v59 == 1202870824 )
  {
operator delete((void *)v120[2]);
    v59 = -946146110;
  }
  v60 = (char *)memset(v117, 0, 0x81uLL);
if ( (~(_BYTE)v107 | 0xFFFFFFFE) == 0xFFFFFFFF )
    v62 = 529704559;
else
    v62 = 1056576168;
  v63 = -474758095;
do
  {
while ( 1 )
    {
      v64 = v60;
      v65 = v63;
while ( v65 <= 529704558 )
      {
        v14 = v65 == -474758095;
        v65 = v62;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v65 != 529704559 )
break;
      v60 = (char *)&v107 + 1;
      v63 = 1499283521;
    }
    v60 = (char *)v108;
    v63 = 1499283521;
  }
while ( v65 == 1056576168 );
  v66 = -474758095;
do
  {
while ( 1 )
    {
      v67 = v61;
      v68 = v66;
while ( v68 <= 529704558 )
      {
        v14 = v68 == -474758095;
        v68 = v62;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v68 != 529704559 )
break;
      v61 = (char *)&v107 + 1;
      v66 = 1499283521;
    }
    v61 = (constchar *)v108;
    v66 = 1499283521;
  }
while ( v68 == 1056576168 );
  v69 = strlen(v67);
sub_DA48(v64, v69, v120, 0LL);                // SHA512
sub_A460(v120, v117, 64LL);                   // HEX2STR
sub_6F20(v105, v110);
  v70 = (char *)memset(v116, 0, 0x81uLL);
if ( (~LOBYTE(v105[0]) | 0xFFFFFFFE) == 0xFFFFFFFF )
    v72 = 529704559;
else
    v72 = 1056576168;
  v73 = -474758095;
do
  {
while ( 1 )
    {
      v74 = v70;
      v75 = v73;
while ( v75 <= 529704558 )
      {
        v14 = v75 == -474758095;
        v75 = v72;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v75 != 529704559 )
break;
      v70 = (char *)v105 + 1;
      v73 = 1499283521;
    }
    v70 = (char *)v106;
    v73 = 1499283521;
  }
while ( v75 == 1056576168 );
do
  {
while ( 1 )
    {
      v76 = v71;
      v77 = v8;
while ( v77 <= 529704558 )
      {
        v14 = v77 == -474758095;
        v77 = v72;
if ( !v14 )
        {
while ( 1 )
            ;
        }
      }
if ( v77 != 529704559 )
break;
      v71 = (char *)v105 + 1;
      v8 = 1499283521;
    }
    v71 = (constchar *)v106;
    v8 = 1499283521;
  }
while ( v77 == 1056576168 );
  v104 = -1466222360;
  v78 = strlen(v76);
sub_DA48(v74, v78, v120, 0LL);                // SHA512
sub_A460(v120, v116, 64LL);                   // HEX2STR
  s[8] = 0;
  *(_QWORD *)s = 0LL;
  v79 = 0;
  v80 = 0;
while ( 1 )
  {
    v81 = v79 >= 8 ? 1072340417 : 1158358064;
if ( v81 == 1072340417 )
break;
    v80 = v116[15 * v79 + v80] & 0xF;
    s[v79] = v117[v80 - -15 * v79];
    ++v79;
  }
  a4[1] = 0LL;
  a4[2] = 0LL;
  *a4 = 0LL;
  v82 = strlen(s);                              // s 最终结果 a17486b9
  v83 = v82;
  v103 = 2 * v82;
  v113 = -17LL;
  v84 = 0x75D037FEF5196874LL;
  v85 = ((v82 + 16) & 0xFFFFFFFFFFFFFFF0LL) - 1;
if ( v82 >= 0x17 )
    v86 = -1265047312;
else
    v86 = 532294316;
if ( v82 >= 0x17 )
    v87 = -1136773100;
else
    v87 = 542793441;
  v88 = 1557339140;
while ( 1 )
  {
while ( 1 )
    {
while ( v88 > 532294315 )
      {
if ( v88 == 532294316 )
        {
          v26 = (char *)a4 + 1;
          *(_BYTE *)a4 = v103;
          v88 = 289241049;
        }
else
        {
if ( v88 != 1557339140 )
abort();
if ( v113 >= v83 )
            v88 = 485505648;
else
            v88 = 1132855204;
        }
      }
if ( v88 != -1265047312 )
break;
      v91 = 1244402348;
do
      {
do
        {
          v92 = v91;
          v93 = v84;
          v84 = v85;
          v91 = 542793441;
        }
while ( v92 == -1136773100 );
        v84 = 22LL;
        v91 = v87;
      }
while ( v92 == 1244402348 );
      v120[0] = -1LL;
      v89 = v93 + 1;
      v90 = -1981771486;
while ( v90 == -1981771486 )
      {
if ( v120[0] >= v89 )
          v90 = 1316265428;
else
          v90 = 1353982062;
      }
if ( v90 != 1316265428 )
abort();
      v102 = v89;
      v26 = (void *)operator new(v89);
      a4[2] = (__int64)v26;
      *a4 = v102 & 1 | v102 ^ 1;
      a4[1] = v83;
      v88 = 289241049;
    }
if ( v88 == 289241049 )
break;
    v14 = v88 == 485505648;
    v88 = v86;
if ( !v14 )
    {
while ( 1 )
        ;
    }
  }
if ( v83 )
    v94 = 1317118332;
else
    v94 = -1355603513;
for ( i = -2002632165; ; i = -1355603513 )
  {
do
    {
      v96 = i;
      v14 = i == -2002632165;
      i = v94;
    }
while ( v14 );
if ( v96 != 1317118332 )
break;
memcpy(v26, s, v83);                        // v26 s 最终结果 a17486b9
  }
if ( v96 != -1355603513 )
  {
while ( 1 )
      ;
  }
  *((_BYTE *)v26 + v83) = 0;
if ( ((LOBYTE(v105[0]) ^ 0xFE) & v105[0]) != 0 )
    v97 = 1202870824;
else
    v97 = -946146110;
while ( v97 == 1202870824 )
  {
operator delete(v106);
    v97 = -946146110;
  }
if ( (~(_BYTE)v107 | 0xFFFFFFFE) == 0xFFFFFFFF )
    v98 = -946146110;
else
    v98 = 1202870824;
while ( v98 == 1202870824 )
  {
operator delete(v108);
    v98 = -946146110;
  }
if ( ((LOBYTE(src[0]) ^ 0xFE) & (__int64)src[0]) != 0 )
    v99 = 1202870824;
else
    v99 = -946146110;
while ( v99 == 1202870824 )
  {
operator delete(src[2]);
    v99 = -946146110;
  }
if ( (~LOBYTE(v110[0]) | 0xFFFFFFFE) == 0xFFFFFFFF )
    v100 = -946146110;
else
    v100 = 1202870824;
while ( v100 == 1202870824 )
  {
operator delete(v110[2]);
    v100 = -946146110;
  }
while ( 1 )
  {
while ( v104 == -1466222360 )
    {
if ( (~LOBYTE(v111[0]) | 0xFFFFFFFE) == 0xFFFFFFFF )
        v101 = -946146110;
else
        v101 = 1202870824;
      v104 = v101;
    }
if ( v104 != 1202870824 )
break;
operator delete(v112);
    v104 = -946146110;
  }
  _ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));
}

```

函数比较长 0x8314 就是 memcpy(v26, s, v83);

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscjlaEy3bwWVibsU7Sg5WMMYtwib2SLmbnkwG8nphDC5icXSCVK2ISrQVwA/640?wx_fmt=other&from=appmsg#imgIndex=9)

然后查找 v83 的值生成处

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXsc6LRqFbg8HJld4hAf7NPF5yAaRat7dcXCZCrEptaUbwumrNeqd1GyKw/640?wx_fmt=other&from=appmsg#imgIndex=10)

V116 和 v117 的值

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscahUHicC2MD6pcdIQw82TNJY8fdkPU807bThiadOtIJdoTcNPzLeVRvKQ/640?wx_fmt=other&from=appmsg#imgIndex=11)

以上还包含 3 个 MD5 个计算函数

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FpFETU052TaH2z1UoQIXscME5kia2UfmCFqrQic2hhtqZ6ic6tu2m5njDrnszT7UxhEaORN0iaR6nqYw/640?wx_fmt=other&from=appmsg#imgIndex=12)

计算流程如下：

```
#################################################################################

2015876998797

mx0

>-----------------------------------------------------------------------------<
[15:21:14 599]x0=unidbg@0xe4fff681, md5=85be9552d873d32d8371455c9d141acc, hex=32303135383736393938373937000000000000000000000300000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 112
0000: 32 30 31 35 38 37 36 39 39 38 37 39 37 00 00 00    2015876998797...
0010: 00 00 00 00 00 00 00 03 00 00 00 00 00 00 00 00    ................
0020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0040: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

MD5

m0xe4fff4c8

>-----------------------------------------------------------------------------<
[15:16:33 932]x0=unidbg@0xe4fff4c8, md5=520c1f7c5d658c18b99490bdc90baef8, hex=016855df4e62560477882d1086be88995a013039000000004016feff000000001b6f8e090000000035a1fdc600000000aec7dc3400000000fa77ca060000000050f6ffe4000000004016feff00000000000000000000000030f6ffe4000000005a013039000000004016feff00000000
size: 112
0000: 01 68 55 DF 4E 62 56 04 77 88 2D 10 86 BE 88 99    .hU.NbV.w.-.....
0010: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
0020: 1B 6F 8E 09 00 00 00 00 35 A1 FD C6 00 00 00 00    .o......5.......
0030: AE C7 DC 34 00 00 00 00 FA 77 CA 06 00 00 00 00    ...4.....w......
0040: 50 F6 FF E4 00 00 00 00 40 16 FE FF 00 00 00 00    P.......@.......
0050: 00 00 00 00 00 00 00 00 30 F6 FF E4 00 00 00 00    ........0.......
0060: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
^-----------------------------------------------------------------------------^

#################################################################################


#################################################################################

mx0

>-----------------------------------------------------------------------------<
[18:23:09 307]x0=unidbg@0xe4fff651, md5=288333d7a3552aba0ff6ac3c28f267e4, hex=31304541303935303130000000000000000000000000003100000000000000200000000000000060304d12000000001a32303135383736393938373937000000000000000000000300000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 112
0000: 31 30 45 41 30 39 35 30 31 30 00 00 00 00 00 00    10EA095010......
0010: 00 00 00 00 00 00 00 31 00 00 00 00 00 00 00 20    .......1....... 
0020: 00 00 00 00 00 00 00 60 30 4D 12 00 00 00 00 1A    .......`0M......
0030: 32 30 31 35 38 37 36 39 39 38 37 39 37 00 00 00    2015876998797...
0040: 00 00 00 00 00 00 00 03 00 00 00 00 00 00 00 00    ................
0050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

MD5 

m0xe4fff4c8

>-----------------------------------------------------------------------------<
[18:23:48 615]unidbg@0xe4fff4c8, md5=a6ce121b955278c35335b033384a7877, hex=dc9067da902243b7f8309bdc61012cb85a013039000000004016feff000000001b6f8e090000000035a1fdc600000000aec7dc3400000000fa77ca060000000050f6ffe4000000004016feff00000000000000000000000030f6ffe4000000005a013039000000004016feff00000000
size: 112
0000: DC 90 67 DA 90 22 43 B7 F8 30 9B DC 61 01 2C B8    ..g.."C..0..a.,.
0010: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
0020: 1B 6F 8E 09 00 00 00 00 35 A1 FD C6 00 00 00 00    .o......5.......
0030: AE C7 DC 34 00 00 00 00 FA 77 CA 06 00 00 00 00    ...4.....w......
0040: 50 F6 FF E4 00 00 00 00 40 16 FE FF 00 00 00 00    P.......@.......
0050: 00 00 00 00 00 00 00 00 30 F6 FF E4 00 00 00 00    ........0.......
0060: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
^-----------------------------------------------------------------------------^


#################################################################################

#################################################################################

mx0

>-----------------------------------------------------------------------------<
[18:27:01 725]x0=RW@0x124d3060, md5=459d9fa6760ea582929457f188dd923c, hex=356c3057586e68695934704a3739344b494a3752773546343556586739736a6f00000000000000000000000000000000633264333139633463356135633764353365363262356137373034653663316200000000000000000000000000000000356c3057586e68695934704a3739344b
size: 112
0000: 35 6C 30 57 58 6E 68 69 59 34 70 4A 37 39 34 4B    5l0WXnhiY4pJ794K
0010: 49 4A 37 52 77 35 46 34 35 56 58 67 39 73 6A 6F    IJ7Rw5F45VXg9sjo
0020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030: 63 32 64 33 31 39 63 34 63 35 61 35 63 37 64 35    c2d319c4c5a5c7d5
0040: 33 65 36 32 62 35 61 37 37 30 34 65 36 63 31 62    3e62b5a7704e6c1b
0050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060: 35 6C 30 57 58 6E 68 69 59 34 70 4A 37 39 34 4B    5l0WXnhiY4pJ794K
^-----------------------------------------------------------------------------^

MD5

m0xe4fff4c8

>-----------------------------------------------------------------------------<
[18:27:29 657]unidbg@0xe4fff4c8, md5=5c2d5d0d6d5be6b1534465e63ba810a0, hex=185fa820bc8d329ecee24c3d982cfb415a013039000000004016feff000000001b6f8e090000000035a1fdc600000000aec7dc3400000000fa77ca060000000050f6ffe4000000004016feff00000000000000000000000030f6ffe4000000005a013039000000004016feff00000000
size: 112
0000: 18 5F A8 20 BC 8D 32 9E CE E2 4C 3D 98 2C FB 41    ._. ..2...L=.,.A
0010: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
0020: 1B 6F 8E 09 00 00 00 00 35 A1 FD C6 00 00 00 00    .o......5.......
0030: AE C7 DC 34 00 00 00 00 FA 77 CA 06 00 00 00 00    ...4.....w......
0040: 50 F6 FF E4 00 00 00 00 40 16 FE FF 00 00 00 00    P.......@.......
0050: 00 00 00 00 00 00 00 00 30 F6 FF E4 00 00 00 00    ........0.......
0060: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
^-----------------------------------------------------------------------------^



#################################################################################

#################################################################################

0x007e1c strlen

0x7E30 sha512

mx1 0x80

>-----------------------------------------------------------------------------<
[13:11:53 657]x1=unidbg@0xe4fff1d0, md5=814fa4f8eb2065c4c1b57cd713375949, hex=3138356661383230626338643332396563656532346333643938326366623431646339303637646139303232343362376638333039626463363130313263623830313638353564663465363235363034373738383264313038366265383839398000000000000000000000000000000000000000000000000000000000000300
size: 128
0000: 31 38 35 66 61 38 32 30 62 63 38 64 33 32 39 65    185fa820bc8d329e
0010: 63 65 65 32 34 63 33 64 39 38 32 63 66 62 34 31    cee24c3d982cfb41
0020: 64 63 39 30 36 37 64 61 39 30 32 32 34 33 62 37    dc9067da902243b7
0030: 66 38 33 30 39 62 64 63 36 31 30 31 32 63 62 38    f8309bdc61012cb8
0040: 30 31 36 38 35 35 64 66 34 65 36 32 35 36 30 34    016855df4e625604
0050: 37 37 38 38 32 64 31 30 38 36 62 65 38 38 39 39    77882d1086be8899
0060: 80 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0070: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 03 00    ................
^-----------------------------------------------------------------------------^


185fa820bc8d329ecee24c3d982cfb41dc9067da902243b7f8309bdc61012cb8016855df4e62560477882d1086be8899
sha512: 
738a5d3956370848ccc11d54348b3f5717bb9beaf1274f03ffd643d36d0d3490875918f34bf5b6f3d0465e8992fc7530bd630feb7e94fe495375d4bc9a677953


m0xe4fff4d8

>-----------------------------------------------------------------------------<
[18:32:47 333]unidbg@0xe4fff4d8, md5=74019ca3dab1aab268eabf13deaf47d5, hex=738a5d3956370848ccc11d54348b3f5717bb9beaf1274f03ffd643d36d0d3490875918f34bf5b6f3d0465e8992fc7530bd630feb7e94fe495375d4bc9a677953000000000000000030f6ffe4000000005a013039000000004016feff000000001b6f8e090000000035a1fdc600000000
size: 112
0000: 73 8A 5D 39 56 37 08 48 CC C1 1D 54 34 8B 3F 57    s.]9V7.H...T4.?W
0010: 17 BB 9B EA F1 27 4F 03 FF D6 43 D3 6D 0D 34 90    .....'O...C.m.4.
0020: 87 59 18 F3 4B F5 B6 F3 D0 46 5E 89 92 FC 75 30    .Y..K....F^...u0
0030: BD 63 0F EB 7E 94 FE 49 53 75 D4 BC 9A 67 79 53    .c..~..ISu...gyS
0040: 00 00 00 00 00 00 00 00 30 F6 FF E4 00 00 00 00    ........0.......
0050: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
0060: 1B 6F 8E 09 00 00 00 00 35 A1 FD C6 00 00 00 00    .o......5.......
^-----------------------------------------------------------------------------^

#################################################################################



#################################################################################

0x7FC0 sha512

mx0

>-----------------------------------------------------------------------------<
[18:43:14 422]x0=RW@0x124d9000, md5=ba992c8d6e5e3ead975bc4967f633965, hex=64633930363764613930323234336237663833303962646336313031326362383031363835356466346536323536303437373838326431303836626538383939000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 112
0000: 64 63 39 30 36 37 64 61 39 30 32 32 34 33 62 37    dc9067da902243b7
0010: 66 38 33 30 39 62 64 63 36 31 30 31 32 63 62 38    f8309bdc61012cb8
0020: 30 31 36 38 35 35 64 66 34 65 36 32 35 36 30 34    016855df4e625604
0030: 37 37 38 38 32 64 31 30 38 36 62 65 38 38 39 39    77882d1086be8899
0040: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-------------------

dc9067da902243b7f8309bdc61012cb8016855df4e62560477882d1086be8899
sha512: 
38cd906f671151fa544ef0148c5875fdb93511e8d9c67c007a9fefc77b950d42866d2bdc7b575dd27ff545fff383f19a395a12c03d25ef66e9537145b49ddb85


m0xe4fff4d8

>-----------------------------------------------------------------------------<
[18:48:36 421]unidbg@0xe4fff4d8, md5=2a263a31ba7534eac55cd8d01c93c2fb, hex=38cd906f671151fa544ef0148c5875fdb93511e8d9c67c007a9fefc77b950d42866d2bdc7b575dd27ff545fff383f19a395a12c03d25ef66e9537145b49ddb85000000000000000030f6ffe4000000005a013039000000004016feff000000001b6f8e090000000035a1fdc600000000
size: 112
0000: 38 CD 90 6F 67 11 51 FA 54 4E F0 14 8C 58 75 FD    8..og.Q.TN...Xu.
0010: B9 35 11 E8 D9 C6 7C 00 7A 9F EF C7 7B 95 0D 42    .5....|.z...{..B
0020: 86 6D 2B DC 7B 57 5D D2 7F F5 45 FF F3 83 F1 9A    .m+.{W]...E.....
0030: 39 5A 12 C0 3D 25 EF 66 E9 53 71 45 B4 9D DB 85    9Z..=%.f.SqE....
0040: 00 00 00 00 00 00 00 00 30 F6 FF E4 00 00 00 00    ........0.......
0050: 5A 01 30 39 00 00 00 00 40 16 FE FF 00 00 00 00    Z.09....@.......
0060: 1B 6F 8E 09 00 00 00 00 35 A1 FD C6 00 00 00 00    .o......5.......
^-----------------------------------------------------------------------------^

#################################################################################



```

3 次 MD5 2 次 HASH512 再加一次计算得出结果

```
package com.sina.weibo.security;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.nio.charset.StandardCharsets;
import java.math.BigInteger;

public class WeiBo {

public static void main(String[] args) {
Stringinfo ="2015876998797";
Stringinfo_md5 = md5(info);
        System.out.println(info_md5);

Stringsalt ="10EA095010";
Stringsalt_md5 = md5(salt);
        System.out.println(salt_md5);

Stringkey ="5l0WXnhiY4pJ794KIJ7Rw5F45VXg9sjo";
Stringkey_md5 = md5(key);
        System.out.println(key_md5);

Strings1 = sha_512(key_md5 + salt_md5 + info_md5);
        System.out.println(s1);

Strings2 = sha_512(salt_md5 + info_md5);
        System.out.println(s2);

byte[] s1_ = s1.getBytes();
byte[] s2_ = s2.getBytes();
byte[] s = new byte[8];

intx =0x00;
for(inti =0 ; i < 8 ; i++){
            x = s2_[15 * i + x] & 0xF;
            s[i] = s1_[x - -15 * i];
        }

        System.out.println(new String(s));
    }

public static String md5(String input){
try {
MessageDigestmd = MessageDigest.getInstance("MD5");
byte[] hashBytes = md.digest(input.getBytes(StandardCharsets.UTF_8));
return bytesToHex(hashBytes);
        } catch (NoSuchAlgorithmException e) {
throw new RuntimeException(e);
        }
    }

private static String bytesToHex(byte[] bytes) {
BigIntegerbi =new BigInteger(1, bytes);
return String.format("%032x", bi);
    }

public static String sha_512(String input) {
try {
MessageDigestmd = MessageDigest.getInstance("SHA-512");
byte[] hashBytes = md.digest(input.getBytes(StandardCharsets.UTF_8));
StringBuilderhexString =new StringBuilder();
for (byte b : hashBytes) {
Stringhex = Integer.toHexString(0xff & b);
if(hex.length() == 1) hexString.append('0');
                hexString.append(hex);
            }
return hexString.toString();
        } catch (NoSuchAlgorithmException e) {
throw new RuntimeException(e);
        }
    }

}

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8FpFETU052TaH2z1UoQIXscIF5YpzbtxhIPROpaibl59zh8L1NJic6UlPdsXiaL0kxfYzwYRS7ia1icMVA/640?wx_fmt=png&from=appmsg#imgIndex=13)

  

看雪 ID：易之生生

https://bbs.kanxue.com/user-home-920134.htm

* 本文为看雪论坛优秀文章，由 易之生生 原创，转载请注明来自看雪社区

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HohCk2q4XUPhRRYV5vYHHyWvjbyTeUqyCEZfCcuVLOrXFzSI6qaJQ9zhID6cwS56hOFk1FeTP9gQ/640?wx_fmt=jpeg&from=appmsg#imgIndex=14)

1.25 折门票开售！看雪 · 第九届安全开发者峰会（SDC 2025）

# 往期推荐

[无 "痕" 加载驱动模块之傀儡驱动 (上)](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458598044&idx=1&sn=4378be6ff58a7bdab353764d141eaf0c&scene=21#wechat_redirect)

[为 CobaltStrike 增加 SMTP Beacon](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458598039&idx=2&sn=98892510d4dbce926d7c963df4bcb779&scene=21#wechat_redirect)

[隐蔽通讯常见种类介绍](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458597880&idx=1&sn=91fa8fdbe75cd055aa1afe2807f98b07&scene=21#wechat_redirect)

[buuctf-re 之 CTF 分析](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458597875&idx=2&sn=b08592712fd8724bda5dbf293a097ee1&scene=21#wechat_redirect)

[物理读写 / 无附加读写实验](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458597873&idx=1&sn=07c7905e2d895effb20393537dd63a0c&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=16)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8Hice1nuesdoDZjYQzRMv9tpvJW9icibkZBj9PNBzyQ4d4JFoAKxdnPqHWpMPQfNysVmcL1dtRqU7VyQ/640?wx_fmt=gif&from=appmsg#imgIndex=17)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8Hice1nuesdoDZjYQzRMv9tpvJW9icibkZBj9PNBzyQ4d4JFoAKxdnPqHWpMPQfNysVmcL1dtRqU7VyQ/640?wx_fmt=gif&from=appmsg#imgIndex=18)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8Hice1nuesdoDZjYQzRMv9tpvJW9icibkZBj9PNBzyQ4d4JFoAKxdnPqHWpMPQfNysVmcL1dtRqU7VyQ/640?wx_fmt=gif&from=appmsg#imgIndex=19)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8Hice1nuesdoDZjYQzRMv9tpUHZDmkBpJ4khdIdVhiaSyOkxtAWuxJuTAs8aXISicVVUbxX09b1IWK0g/640?wx_fmt=gif&from=appmsg#imgIndex=20)

点击阅读原文查看更多