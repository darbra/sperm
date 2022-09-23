> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1691013&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)red1y _ 本帖最后由 red1y 于 2022-9-21 22:26 编辑_  

xx 度灰 app 加密算法分析还原
==================

### 本文包括

*   java 层分析
*   so 层加密算法分析
*   java 层签名分析
*   发包验证
*   b 站视频连接 [https://www.bilibili.com/video/BV1id4y1g7G1/?spm_id_from=333.999.0.0&vd_source=23b9de401c27e819abddbd5551eddbda](https://www.bilibili.com/video/BV1id4y1g7G1/?spm_id_from=333.999.0.0&vd_source=23b9de401c27e819abddbd5551eddbda)

### 一. 修改测试

1.  重新打包签名生成可调试版本后运行闪退
2.  使用 mt 管理修改 dex 文件重新编译后正常运行，访问相关资源会提示正版维护信息
3.  使用 mt 管理器重新签名后运行闪退

### 二、Java 层静态分析

1.  定位启动`Activity`
    
    ```
    <activity android:@style/AppTransparentTheme">
         <intent-filter>
           <action android:/>
           <category android:/>
         </intent-filter>
       </activity>
    
    ```
    
2.  分析`onCreate`函数
    
    `LauncheActivity`继承自`BaseActivity`，`BaseActivity`中调用了几个函数，经分析确定了`w5()`为关键函数：
    
    ```
    @Override  // android.support.v7.app.AppCompatActivity
    protected void onCreate(@nullable Bundle arg2) {
       this.Z5();
       super.onCreate(arg2);
       try {
           this.a = android.databinding.f.l(this.O5(), this.Q5());
           this.V5();
           com.tencent.mm.base.f.c().a(this);
           this.W5(); // 关键函数
           this.U5();
           this.b6();
       }
       catch(Exception v2) {
           v2.printStackTrace();
       }
    }
    
    ```
    
3.  跟踪`w5()`
    
    在前面测试过程中有一个信息是：修改了签名后的 app 并不会直接闪退，而是在申请完相关使用权限后才会退出，如果没有通过权限的申请，会自动正常退出，而不是闪退；在`w5()`中找到了相关权限申请的函数`T6()`
    
    ```
    @Override  // com.tencent.mm.base.BaseActivity
    public void W5() {
       /* other code */
       int v0_1 = this.checkSha1(this) ? 1 : 2;
       com.tencent.mm.network.d.h = this.D6(this) + ":" + v0_1;
       org.greenrobot.eventbus.c.f().t(this);
       this.I = new LaunchModel(this);
       this.z6();
       this.T6(); // 在T6中进行权限申请
       String v0_2 = i1.k().E();
       if(!TextUtils.isEmpty(v0_2)) {
           com.tencent.mm.l.j.d().v(((UserInfoBean)JSON.parseObject(v0_2, UserInfoBean.class)));
       }
    }
    
    ```
    
    在`T6()`中如果没有赋予应用相关权限，则会结束应用，否则进入`B6()`
    
    ```
    public void T6() {
       /* other code */
       // if no permission, return and eixt
       LaunchActivity.this.B6();
    
    ```
    
    跟踪`B6()`后续的一系列函数调用，最终定位到一个向服务器发送请求的函数
    
    ```
    public void q(String arg6) {
       d.D1().N4(arg6);
       d.D1().d4("http://xxxx/.../xxx", d.D1().x1(), new b("/api/xxx/xxx") {
       }
    }
    
    ```
    
    开启`Fiddler`抓报后，发现应用自启动到闪退没有发送任何请求，`d4()`函数中在发送请求前进行了一系列的数据操作
    
    ```
    public void d4(String arg2, HttpParams arg3, com.tencent.mm.network.b arg4) {
       ((PostRequest)((PostRequest)((PostRequest)((PostRequest)((PostRequest)OkGo.post(arg2).tag(arg4.b())).upJson(this.s2(arg3).toJSONString())).headers("token", i1.k().w())).cacheKey(this.C0(arg4.a()))).cacheMode(CacheMode.FIRST_CACHE_THEN_REQUEST)).execute(arg4);
    }
    
    ```
    
    `upJson()`参数即为上传的数据，经过了`s2()`的处理，跟进`s2()`，最终数据的加密封装在`com.szcx.lib.encrypt.c.k()`中进行
    
    ```
    public String k(String arg5) throws JSONException {
       String v5 = this.e(arg5);
       JSONObject v2 = new JSONObject();
       v2.put("timestamp", "1663503240");
       v2.put("_ver", "v1");
       v2.put("data", v5);
       v2.put("sign", this.j(a.e("_ver=v1&data=" + v5 + "×tamp=" + "1663503240" + this.e)));
       return v2.toString();
    }
    
    ```
    
    经过对正常 app 运行时的抓包比较，此处的参数与实际一致，在`e()`中队数据进行了加密，最终调用`native`函数进行加密
    
    ```
    public String f(String arg1, String arg2) {
       return EncryptUtil.encrypt(arg1, arg2);
    }
    
    ```
    
    ```
    public static native String encrypt(String arg0, String arg1) {
    }
    
    ```
    
    同时在代码中发现了多个密钥，包括但不限于，第一个`base64`编码的密钥在跟踪流程中传递给了`native`函数
    
    *   `BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==`
    *   `81d7beac44a86f4337f534ec9332837`

### 三、Java 层动态跟踪、Hook 分析

1.  将前面重新打包签名生成的可调试的`apk`安装到手机上，为防止应用直接闪退，拒绝其相关权限的申请，同时在程序判断权限申请结果处下断，动态修改权限申请的结果，使后续流程继续下去
    
2.  调试跟踪函数，最终定位发现程序在加载上述`native`加密`so`库时闪退
    
    ```
    .method static constructor <clinit>()V
             .registers 1
    00000000  const-string        v0, "sojm"
    00000004  invoke-static       System->loadLibrary(String)V, v0
    0000000A  return-void
    .end method
    
    ```
    
3.  同时在上面的跟踪过程中还可以得到程序生成的一系列请求参数，包含了大量系统、设备信息，但没有`hash`相关的参数
    
4.  使用 `frida` `hook` `encrypt`函数，主动调用其多次加密相同数据，可以发现每次得到的结果都不同，应该使用了某种随机量
    

### 四、so 层静态分析

1.  使用`ida pro`分析`sojm` 库，通过观察函数名可以得到其是通过静态注册的，这里的四个参数也符合常规的`jni`函数
    
    ```
    // JNIEnv* env
    // jclass _clazz
    // jstring a3
    // jstring a4
    int __fastcall Java_com_qq_lib_EncryptUtil_encrypt(int a1, int a2, int a3, int a4)
    {
     int v8; // r4
     int v10[4]; // [sp+4h] [bp-2Ch] BYREF
     int v11; // [sp+14h] [bp-1Ch]
    
     v8 = cgo_wait_runtime_init_done();
     v10[3] = a4; // jstring
     v10[2] = a3; // jstring
     v10[1] = a2; // _clazz
     v10[0] = a1; // env
     v11 = 0;
     crosscall2(cgoexp_17c794619cba_Java_com_qq_lib_EncryptUtil_encrypt, v10, 20, v8);
     cgo_release_context(v8);
     return v11; // jstring
    }
    
    ```
    
2.  但是后续的操作就不太常规了，可以看出它把参数依次赋给了一个数组；同时调用了`crosscall2`，其参数为:
    
    1.  一个函数地址
    2.  参数数组
    3.  应该是参数数组的长度, size
    4.  `init`函数的返回值
    
    值得注意的是，`v10`明明只有四个元素，但是传入的参数 size 却是`20 = 5 * 4`，同时`v11`被置`0`后又没有显式的赋值，最终却被返回，猜测应该是在`cgoexp_17c794619cba_Java_com_qq_lib_EncryptUtil_encrypt`中被赋值了
    
3.  进入`cgoexp_17c794619cba_Java_com_qq_lib_EncryptUtil_encrypt`后发现参数个数很奇怪，而且`sub_BC3C4658`传入了很多重复的参数；
    
    ```
    int __fastcall cgoexp_17c794619cba_Java_com_qq_lib_EncryptUtil_encrypt(int a1, int a2, int a3, int a4, int a5, int a6)
    {
     int v6; // r10
     int v7; // lr
     int v9; // [sp+14h] [bp-4h]
     int v10; // [sp+14h] [bp-4h]
    
     while ( (unsigned int)&a5 <= *(_DWORD *)(v6 + 8) )
       sub_BC360D10();
             sub_BC3C4658( // 通过这个函数可以推测a6为前面传入的数组地址
           a6,
           *(_DWORD *)a6,
           *(_DWORD *)(a6 + 4),
           *(_DWORD *)(a6 + 8),
           v7,
           *(_DWORD *)a6,
           *(_DWORD *)(a6 + 4),
           *(_DWORD *)(a6 + 8),
           *(_DWORD *)(a6 + 12),
           v9);
       // sub_BC3C4658函数没有返回值,局部变量v10也没有被显式地赋值
     *(_DWORD *)(a6 + 16) = v10; // 在这里对a6[4],也就是上面v11的地址处进行了赋值
     sub_BC301DF8();
     return sub_BC2FBDAC();
    }
    
    ```
    
4.  通过上面的观察分析，可以察觉到这不是常规的函数调用约定，而且肯定不是`fastcall`；注意到函数中出现了`cgo`字样，且在该`so`库的函数表中也有大量的`cgo`函数
    
5.  分析：
    
    1.  这个`so`库的调用约定与常规的不同，很可能是全部通过栈进行的，包括参数的传递以及返回值的传递
    2.  `golang`是可以和`c`进行交叉调用的，而且可以编译成`so`库
    3.  这个 so 库的核心加密部分应该是由`golang`编写的，`C`接口函数就起到个连接、转发的作用
6.  定位关键加密函数
    
    虽然看起来有点奇怪，但是这并不妨碍定位到关键函数，跟进上面的`sub_BC3C4658()`函数：
    
    ```
    int __fastcall sub_BC3C4658(int a1, int a2, int a3, int a4, int a5, int a6, int a7, int a8, int a9, int a10)
    {
     int v10; // r10
     int v11; // lr
     int result; // r0
     int v13; // [sp+Ch] [bp-2Ch]
     _DWORD *v14; // [sp+10h] [bp-28h]
     int v15; // [sp+10h] [bp-28h]
     int v16; // [sp+14h] [bp-24h]
     int v17; // [sp+20h] [bp-18h]
     int v18; // [sp+24h] [bp-14h]
     int v19[2]; // [sp+30h] [bp-8h] BYREF
    
     while ( (unsigned int)&a5 <= *(_DWORD *)(v10 + 8) )
       sub_BC360D10();
     v19[0] = a6;
     sub_BC3BED84();
     v19[1] = v13;
     sub_BC3BED84();
     sub_BC3C0D0C(v13, (int)v14, v16, v16, v11, (int)v19, v13, (int)v14, v16, v13, v14, v16, v17, v18);
     sub_BC3BEC54();
     result = v15;
     a10 = v15;
     return result;
    }
    
    ```
    
    跟进`sub_BC3C0D0C()`，发现了`package_name`，`pakcage_hash`字样，且进行了大量函数调用，将动态调试的目光先锁定在它身上
    
    ```
    int __fastcall sub_BC3C0D0C(int a1, int a2, int a3, int a4, int a5, int a6, int a7, int a8, int a9, int a10, _DWORD *a11, int a12, int a13, int a14)
    {
     while ( (unsigned int)&a5 <= *(_DWORD *)(v14 + 8) )
       sub_BC360D10();
     v34 = v15;
     a13 = 0;
     a14 = 0;
     v72 = &off_BC3DFD34;
     sub_BC3C42D8();
     v38 = a11;
     sub_BC3C21F8();
     v16 = v46;
     if ( v61 )
     {
       a13 = 0;
       a14 = 0;
       result = sub_BC3C0CDC();
     }
     else
     {
       v68 = v49;
       v69 = v57;
       sub_BC304C38();
       v71 = v38;
       sub_BC305928();
       if ( dword_BC47A460 )
       {
         sub_BC36294C(dword_BC47A460, v17, v71, &unk_BC3D2F68);
         v18 = v19;
       }
       else
       {
         v18 = v71;
         *v71 = &unk_BC3D2F68;
       }
       v50 = (int)v18;
       sub_BC3A70F8();
       if ( v55 )
       {
         v20 = a9;
         v21 = a8;
         v22 = a7;
       }
       else
       {
         v23 = v65;
         do
           *v23++ = 0;
         while ( (int)v23 <= (int)&v65[15] );
         qmemcpy(v65, "__package_name__", sizeof(v65));
         sub_BC34E438((int)&v65[15], (int)v23, 0, 95, v15, 0, (unsigned __int8 *)v65, 16, (int)&unk_BC3CF818, v50);
         v67 = v47;
         v64 = v51;
         v24 = v65;
         do
           *v24++ = 0;
         while ( (int)v24 <= (int)&v65[15] );
         qmemcpy(v65, "__package_hash__", sizeof(v65));
         sub_BC34E438(v47, (int)v24, v51, 0, v35, 0, (unsigned __int8 *)v65, 16, v47, v51);
         v66 = v48;
         v63 = v52;
         sub_BC3C4318();
         sub_BC34E438(0, v25, v26, v27, v36, 0, v39, v41, v48, v52);
         sub_BC301F40();
         v28 = (unsigned __int8 *)*v71;
         v70 = v42;
         v40 = v28;
         v43 = v67;
         sub_BC309480();
         *v53 = &unk_BC3D2A70;
         if ( dword_BC47A460 )
           sub_BC36294C(v53 + 1, v29, v53 + 1, v70);
         else
           v53[1] = v70;
         sub_BC3C440C();
         sub_BC34E438(0, v30, v31, v32, v37, 0, v40, v43, v64, (int)v53);
         sub_BC3C094C();
         sub_BC301F40();
         v70 = v44;
         v45 = v66;
         sub_BC309480();
         *v54 = &unk_BC3D2A70;
         if ( dword_BC47A460 )
           sub_BC36294C(v54, dword_BC47A460, v54 + 1, v70);
         else
           v54[1] = v70;
         sub_BC3AEB0C();
         v22 = v45;
         v21 = v63;
         v20 = (int)v54;
       }
       if ( v16 )
       {
         a13 = 0;
         a14 = 0;
       }
       else
       {
         sub_BC3C05EC(v56, v22, v21, v20, v34, v22, v21, v20, v69, v58, v59, v68, v55, v56, v59, 0);
         a13 = v60;
         a14 = v62;
       }
       result = sub_BC3C0CDC();
     }
     return result;
    }
    
    ```
    

### 五、so 层动态调试、分析调用约定

1.  在`jni`接口处下断，符合`fastcall`调用约定
    
    ![](https://attach.52pojie.cn/forum/202209/21/215001x0008jxllzpjz8ki.png)
    
    **image-20220921175805969.png** _(102.07 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2M3xjM2MzYzg0MnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:50 上传
    
2.  传递给`cgo`的参数
    
    ![](https://attach.52pojie.cn/forum/202209/21/215124r8onk75fa7k8apsa.png)
    
    **image-20220921180034269.png** _(75.33 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2NHw3NmE0YzM2OXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:51 上传
    
    ![](https://attach.52pojie.cn/forum/202209/21/215141rfo4lx28loczxryr.png)
    
    **image-20220921180136723.png** _(40.66 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2NXxkMmZhYjYyMnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:51 上传
    
3.  进入`cgo`函数，首先观察栈平衡循环
    
    ![](https://attach.52pojie.cn/forum/202209/21/215257nyi7x91u9tl87t17.png)
    
    **image-20220921180451141.png** _(83.75 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2NnwyYWJlNDdlYnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:52 上传
    
    结束后
    
    ![](https://attach.52pojie.cn/forum/202209/21/215317unl6h6me5ep6vvs8.png)
    
    **image-20220921180715293.png** _(109.45 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2N3xlMjdjZGNhZXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:53 上传
    
4.  观察从哪里取得参数
    
    ![](https://attach.52pojie.cn/forum/202209/21/215336ia5a5rkscl1k4s44.png)
    
    **image-20220921181020664.png** _(104.08 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2OHw4MzcwMmMxZHwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:53 上传
    
5.  在下一个函数调用前下断，观察参数传递
    
    ![](https://attach.52pojie.cn/forum/202209/21/215352aw9eezotft11moeb.png)
    
    **image-20220921181238897.png** _(132.09 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc2OXwyMDZjN2Y3N3wxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:53 上传
    
6.  f8 步过，观察栈变化以及从哪里取的返回值
    
    ![](https://attach.52pojie.cn/forum/202209/21/215406wx5c90icvnxyi4tc.png)
    
    **image-20220921181412980.png** _(103.19 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3MHxlN2Q2MGQ0ZHwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:54 上传
    
7.  总结得出函数调用：参数完全通过栈传递，返回值存储在参数往下的地址中
    

### 六、so 层加密算法还原

1.  前置工作分析：可以跟踪调试`sub_BC3C0D0C`函数，发现这里只是进行了一些参数以及其他操作，真正的加密处理函数在这个函数的结尾处调用，即：`sub_C2DC05EC`
    
2.  需要说明的是，这个函数中对`java`层传入的`key`进行了`base64`解码，并得到两个密钥：
    
    1.  `key1`: `4c7e?2fab4f4>1>2>5d0ee4bb61d7b03`
    2.  `key2`: `mIZUjjghGd`
3.  在`sub_C2DC05EC`处下断，分析参数
    
    ![](https://attach.52pojie.cn/forum/202209/21/215421pa1bl616bw9u8lhv.png)
    
    **image-20220921182713971.png** _(33.09 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3MXxhMThlZDg4ZnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:54 上传
    
4.  首先对传入的两个`key`进行了异或得到一个新`key`
    
    ![](https://attach.52pojie.cn/forum/202209/21/215442x83gh33bwjrg84e7.png)
    
    **image-20220921182838936.png** _(70.15 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3MnwzZjVhY2Q0OXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:54 上传
    
5.  再对`key`进行了两次转换，得到
    
    第一次得到
    
    ![](https://attach.52pojie.cn/forum/202209/21/215455wxlzxv4lqtqxxlz9.png)
    
    **image-20220921182913411.png** _(20.14 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3M3w0YzRkN2RmY3wxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:54 上传
    
    第二次得到，此时密钥已经成为一个不可读的字节序列，这也是最终加密算法使用的密钥
    
    ![](https://attach.52pojie.cn/forum/202209/21/215511b5z2kbbsyihhj9k5.png)
    
    **image-20220921183107882.png** _(20.25 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3NHxhODlmY2VmNHwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:55 上传
    
6.  之后生成了一个长度为`0x10`的随机串，这是最终加密算法使用的初始向量
    
    ![](https://attach.52pojie.cn/forum/202209/21/215525dggoidamvtbewhet.png)
    
    **image-20220921183027646.png** _(66.45 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3N3xmM2NkOWU5Y3wxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:55 上传
    
7.  之后传入密钥，调用一个函数后返回了一个全局地址和一个指针
    
    ![](https://attach.52pojie.cn/forum/202209/21/215540tkdddgvgok11ddyz.png)
    
    **image-20220921183450895.png** _(76.12 KB, 下载次数: 1)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3OHwzZDg1ODE5MnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:55 上传
    
    其中指针的内容是
    
    ![](https://attach.52pojie.cn/forum/202209/21/215556zmg02om1dymtc05s.png)
    
    **image-20220921183648512.png** _(103.95 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc3OXw4MjNlYTM0ZnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:55 上传
    
    到这里的话，因为前面已经猜测这是一个`golang`编写的`so`库，此时基本可以确定这是使用的`go`的`crypto/cipher`加密库了；
    
    通过查看`go`加密的源码，能发现其`newCipher`最终会生成两个长度为`0x3C`即`60`的密钥，分别用来加密和解密；
    
    又由于是对称加密，因此使用的是用一个密钥，这里生成的两个密钥刚好是逆序的关系，可能是因为方便实现的原因
    
    ![](https://attach.52pojie.cn/forum/202209/21/215609w3yf3xfrfxlz3x5e.png)
    
    **image-20220921195430035.png** _(33.27 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4MHwzYWU1NTc5NHwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:56 上传
    
    ![](https://attach.52pojie.cn/forum/202209/21/215621lkkp4pmsmp2k2ci3.png)
    
    **image-20220921195522728.png** _(17.1 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4MXw1YmI4YWMwNXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:56 上传
    
    ![](https://attach.52pojie.cn/forum/202209/21/215635tugt3g91f1bglgya.png)
    
    **image-20220921195620953.png** _(28.55 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4MnxmOGYwNmRjYXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:56 上传
    
8.  说明：这里 usb 断了一次连接，因此下面一些参数的地址可能和上面不同
    
9.  接着，调用了一个保存在寄存器的地址，并将明文和前面密钥生成的结构当作参数传了进去
    
    ![](https://attach.52pojie.cn/forum/202209/21/215915dislvv05h0z5slgf.png)
    
    **image-20220921190408627.png** _(136.95 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc5MXxjNjMzMDRkMHwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:59 上传
    
10.  函数返回后，那片内存空间里已经由全 0 填充为了字节序列，可以确定其为加密函数
    
    ![](https://attach.52pojie.cn/forum/202209/21/215709zd9sdqmjshs9lr5r.png)
    
    **image-20220921190708165.png** _(64.11 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4NHwyZDJlN2U0ZnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:57 上传
    
    ![](https://attach.52pojie.cn/forum/202209/21/215725x5ryg339t378j8yb.png)
    
    **image-20220921190906871.png** _(90.51 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4NXwwMTA0NzE5MnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:57 上传
    
11.  最后，又对加密生成的序列进行了一次字母表映射，字母表为 16 进制的 16 个字符
    
    ![](https://attach.52pojie.cn/forum/202209/21/215738jtcyqslcqz9u83ci.png)
    
    **image-20220921191342058.png** _(48.45 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4NnwyZmU3MjQ1Y3wxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:57 上传
    
12.  最终得到的密文为
    
    ![](https://attach.52pojie.cn/forum/202209/21/215750p42s630ukv89sm2s.png)
    
    **image-20220921191405844.png** _(53.37 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4N3xlYWFhOGQ2YnwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:57 上传
    
13.  算法还原
    
14.  找到最后一步密钥转换后生成的字节序列，这就是真正的加密密钥
    
15.  确定加密算法，根据几个特征，推测应该是一种有初始向量的流加密，最终确定为 CFB 模式的 aes 加密
    
    1.  go 的加密库
    2.  有初始向量的加密算法
    3.  没有 padding 操作
16.  之后，首先用 go 还原一下，验证加密算法无误
    
17.  之后用 python 重现，这里要注意的是默认的 python 和 go 的 CFB 加密结果是不同的，需要在 python 加密中设置以下属性：
    
    ![](https://attach.52pojie.cn/forum/202209/21/215801p8lvpwppkovw6ppo.png)
    
    **image-20220921191855157.png** _(13.15 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4OHw4YzdjZDU3N3wxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:58 上传
    
18.  最后验证 python 和 go 的加密结果一致即可
    

### 七、Java 层签名算法分析还原

1.  接下来就是 java 层 sign 字段的生成了，这个比较简单，它依次调用了两个哈希算法
    
    1.  sha256：根据 post 参数的格式及各个参数生成输入，经过 sha256 得到一个十六进制字符串
        
        ```
          public static String e(String arg2) {
              try {
                  MessageDigest v0 = MessageDigest.getInstance("SHA-256");
                  v0.update(arg2.getBytes("UTF-8"));
                  return a.b(v0.digest()); // 将字节数组转换为16进制字符串
              }
              catch(NoSuchAlgorithmException v2_1) {
                  v2_1.printStackTrace();
                  return "";
              }
              catch(UnsupportedEncodingException v2) {
                  v2.printStackTrace();
                  return "";
              }
          }
        
        
        ```
        
    2.  md5：将上面的到的 sha256 字符串经过 md5 变换得到最终的 sign 值
        
        ```
          public static String b(String arg2) {
              try { // a() 将字节数组转换为16进制字符串
                  return c.a(MessageDigest.getInstance("MD5").digest(arg2.getBytes("UTF-8")));
              }
              catch(Exception v2) {
                  v2.printStackTrace();
                  return "";
              }
          }
        
        ```
        

### 八、发包验证算法正确性

1.  用 python 实现它的数据加解密以及签名、封装过程，生成请求数据，并向其服务器的一个接口发起请求验证：
    
    ![](https://attach.52pojie.cn/forum/202209/21/215816kjo5xjnxoox1v4vo.png)
    
    **image-20220921194210696.png** _(385.57 KB, 下载次数: 0)_
    
    [下载附件](forum.php?mod=attachment&aid=MjU1NTc4OXxiOWZiN2EzYXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)
    
    2022-9-21 21:58 上传
    
2.  服务器正常返回数据，解密得到数据：
    

![](https://attach.52pojie.cn/forum/202209/21/215828sa8aft76guy1fy66.png)

**image-20220921194419585.png** _(133.61 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1NTc5MHxlZWI3ZDNiZXwxNjYzOTI1NjM5fDE0MTAxOTh8MTY5MTAxMw%3D%3D&nothumb=yes)

2022-9-21 21:58 上传

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)TY-1314 厉害 &#128077;&#127995;&#128077;&#127995;&#128077;&#127995;&#128077;&#127995;![](https://avatar.52pojie.cn/images/noavatar_middle.gif)hjw01 很不错哦，还有一种方法是直接脱壳解析所有类，重新生成 dex，修正 androidmanifest 应该是可行的。有空再试试![](https://avatar.52pojie.cn/data/avatar/001/10/94/58_avatar_middle.jpg)正己 加个精，期待大佬后续佳作![](https://static.52pojie.cn/static/image/smiley/laohu/laohu33.gif)![](https://avatar.52pojie.cn/data/avatar/000/15/32/25_avatar_middle.jpg)孤灯独饮 啥 APP  是好看的 APP 么?![](https://avatar.52pojie.cn/data/avatar/000/24/69/19_avatar_middle.jpg) 博爵 支持，可惜没有成品![](https://avatar.52pojie.cn/images/noavatar_middle.gif)一介书生 老哥，apk 可以放个链接么，不是成品![](https://avatar.52pojie.cn/data/avatar/000/39/14/88_avatar_middle.jpg)泥河湾メ~ 晓亮﹀ 感谢楼主分享![](https://avatar.52pojie.cn/images/noavatar_middle.gif) mozhongzhou 高 太高了![](https://avatar.52pojie.cn/images/noavatar_middle.gif) CA99588 感谢分享！![](https://avatar.52pojie.cn/images/noavatar_middle.gif)娃娃菜啊 你这是纯技术型的，虽然知道软件无奈看不懂过程，祝早日突破技术更上一层