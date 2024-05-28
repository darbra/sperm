> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XAwKmbfNQ57XlnXvWLufeA)

点击上方 “小白逆向之旅”，选择 “加为星标”

第一时间关注逆向技术干货！

前言
--

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者 (wx：xxb00414) 立即删除！

样本
--

aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMXR2eG12elhzQ1UzbXJwMktRdDRpRWc/cHdkPXh4eGI=

抓包
--

app 歌单接口花瓶抓包如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3XPJIVUucwLrYaZuvAuBY6yjPRKLoeicnfSfgzTfJcNkmWMP3K8USNRjZ3kxicxOTKbGMd7jg4ARTg/640?wx_fmt=png&from=appmsg)

可以看到 sign mask body 都是加密的 今天咱们的受害者就一个 sign 其他参数读者可以阅读完本文之后自行研究

Java 层分析
--------

jadx 反编译 apk 搜索 x-sign-data-type 可以定位到 sign 是在如下位置生成的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3XPJIVUucwLrYaZuvAuBY6fpxJ3PY6CcuBI2wvSrk12bnxibK9pTaiaP5YM54Aia7wXy517peQgzCRw/640?wx_fmt=png&from=appmsg)

sign 是由 MERJni.calc 函数返回 跟进 映入眼帘的是一个 native 方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3XPJIVUucwLrYaZuvAuBY67kWksHHWnHEbIa3Dv26KtAm0iaRicXWicTSv1SzoiaLKHUZ8FEvFshZEXQ/640?wx_fmt=png&from=appmsg)

使用 frida hook 验证一下 以下是 hook 脚本

```
var ByteString = Java.use("com.android.okhttp.okio.ByteString");
function toUtf8(tag, data) {
    console.log(tag + " Utf8: ", ByteString.of(data).utf8());
}
Java.perform(function () {
    let MERJni = Java.use("com.tencent.qqmusic.modular.framework.encrypt.logic.MERJni");
    MERJni["calc"].implementation = function (bArr, bArr2) {
        toUtf8("body", bArr)
        toUtf8("param", bArr2)
        let result = this["calc"](bArr, bArr2);
        console.log(`MERJni.calc result=${result}`);
        return result;
    };
})

```

搜索 app 成功被 hook 并打印了以下日志

```
body Utf8:  {"comm":{"tyt_exp_env":"0","uid":"5482662491","sid":"202405251918595482662491","OpenUDID2":"ffffffffc99f7b9c0000018ee03fd787","OpenUDID":"ffffffffc99f7b9c000000000033c587","udid":"ffffffffc99f7b9c000000000033c587","ct":"11","cv":"13050008","v":"13050008","chid":"72280","os_ver":"10","aid":"bb242b3f3eab8716","tmeLoginType":"0","tmeLoginMethod":"0","phonetype":"Pixel","fPersonality":"0","devicelevel":"30","newdevicelevel":"10","deviceScore":"302.99","QIMEI36":"184fdeb5f885f741d3f7558710001661840f","taid":"0101869F3CA41CEFD47D5CF7E7F3189CA2F6A60E8CBB995D3B9634121A80264EE3A4029C112D70E06B2ACCC5","tmeAppID":"qqmusic","tid":"4432201308658353152","modeSwitch":"6","teenMode":"0","M-Value":"pCS3+/6PXCG82WGlXyTbpQ\u003d\u003d","ui_mode":"1","nettype":"1030","wid":"5482662491","rom":"google/google/sailfish/sailfish:10/QP1A.191005.007.A3/5972272:user/release-keys/","v4ip":"120.244.191.20","hotfix":"100000000","traceid":"11_05482662491_1716635772","free_mode":"0","ext":"{\"bluetooth\":\"\"}","br":"1"},"music.diyplaylist.PlDiyInfoRead.Read":{"module":"music.diyplaylist.PlDiyInfoRead","method":"Read","param":{"strTid":"7581901981","from":"1"}},"music.video.TencentVideoQuery.GetVideoListByPlaylistId":{"module":"music.video.TencentVideoQuery","method":"GetVideoListByPlaylistId","param":{"playlistId":7581901981}},"music.srfDissInfo.PlSongExtServer.getPlSongExtInfo":{"module":"music.srfDissInfo.PlSongExtServer","method":"getPlSongExtInfo","param":{"tid":7581901981,"need":[1,2]}},"music.SspAdvert.PhoenixAdProxySvr.GetAdByPos":{"module":"music.SspAdvert.PhoenixAdProxySvr","method":"GetAdByPos","param":{"adPos":122,"pt":"banner-folder","subPt":"","uin":"null","params":{"extItem-1":"1","extItem-2":"{\"open_udid\":\"ffffffffc99f7b9c000000000033c587\",\"android_id\":\"bb242b3f3eab8716\",\"id_type\":2,\"mac_addr\":\"02:00:00:00:00:00\",\"hw_model\":\"Pixel\",\"hw_machine\":\"sailfish\",\"os_version\":\"Android 10\",\"mid\":\"a8ddfcf170a5188c43059c87bd4c45ddefd9769a\",\"sdk_version\":\"QNaPhoneV5.5.5\",\"app_name\":\"QQ音乐 13.5.0.8\",\"brands\":\"google\",\"net_status\":\"wifi\",\"chid\":\"1\",\"resolution\":\"1080x1794\",\"req_extend\":{\"id1\":\"7581901981\",\"type\":2,\"id8\":\"0\"},\"music_ad_id\":\"10501\",\"source\":\"\"}","phoenix_pos":"{\"confs\":[{\"posType\":1,\"value\":\"7581901981\"}]}"}}},"music.srfDissInfo.DissInfo.CgiGetDiss":{"module":"music.srfDissInfo.DissInfo","method":"CgiGetDiss","param":{"new_format":1,"disstid":7581901981,"enc_host_uin":"","dirid":0,"onlysonglist":0,"need_game_ad":1,"optype":2,"orderlist":1,"guid":"ffffffffc99f7b9c0000018ee03fd787","tag":1,"userinfo":1,"is_mobile":1,"censor_status":1,"local_time":1716635755,"update_rtime":1,"rec_flag":1,"expire_couple":0,"ext":{"bluetooth":""}}},"music.srfDissInfo.PlExtServer.getPlExtInfo":{"module":"music.srfDissInfo.PlExtServer","method":"getPlExtInfo","param":{"dirid":0,"tid":7581901981,"need":[2,3,4,6],"bWithPlTag":true}},"music.interactiveplaylist.PlInteractiveInfoRead.GetPlInteractiveInfo":{"module":"music.interactiveplaylist.PlInteractiveInfoRead","method":"GetPlInteractiveInfo","param":{"bExpireCouple":true,"tid":"7581901981","bShare":false,"getDetail":true}}}
param Utf8:  11&13050008&5482662491&null&1716635755643&ffffffffc99f7b9c000000000033c587&android
MERJni.calc result=SEdLUVFLUkFNWkpOtM3Sc5+VLxPz8PVthusUef5kYT0= j6AzMt5fHA0Dt0eBebuPY2iiB6TLdenXyrqDF4B+pflZvH1JSGqVsIG0QUhxsJlmsCuuMKym9uCOp5JInqPsCtMZbt023RmUpI/prbSQYNjGmd1L0iIFFWWyTLx+xjUD

```

至此 可以判断 sign 是由该 native 方法生成 

Unidbg 调用
---------

这个时候就可以祭出 unidbg 了 去调用一下这个 so 代码如下

```
package com.qqmusic;
import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.backend.Unicorn2Factory;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.AbstractJni;
import com.github.unidbg.linux.android.dvm.DalvikModule;
import com.github.unidbg.linux.android.dvm.DvmClass;
import com.github.unidbg.linux.android.dvm.VM;
import com.github.unidbg.memory.Memory;
import java.io.File;
import java.nio.charset.StandardCharsets;
public class Music extends AbstractJni {
    private final AndroidEmulator emulator;
    private final DvmClass MERJni;
    private final VM vm;
    private final Module module;
    public Music() {
        emulator = AndroidEmulatorBuilder
                .for32Bit()
                .addBackendFactory(new Unicorn2Factory(true))
                .setProcessName("com.tencent.qqmusic")
                .build();
        Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));
        vm = emulator.createDalvikVM(new File("unidbg-android/src/test/resources/qqmusic/1.apk"));
        vm.setJni(this);
        vm.setVerbose(true);
        DalvikModule dm = vm.loadLibrary("mer", true);
        module = dm.getModule();
        MERJni = vm.resolveClass("com.tencent.qqmusic.modular.framework.encrypt.logic.MERJni");
        dm.callJNI_OnLoad(emulator);
    }
    public void callCalc() {
        byte[] arg1 = "{\"comm\":{\"tyt_exp_env\":\"0\",\"uid\":\"5482662491\",\"sid\":\"202405251918595482662491\",\"OpenUDID2\":\"ffffffffc99f7b9c0000018ee03fd787\",\"OpenUDID\":\"ffffffffc99f7b9c000000000033c587\",\"udid\":\"ffffffffc99f7b9c000000000033c587\",\"ct\":\"11\",\"cv\":\"13050008\",\"v\":\"13050008\",\"chid\":\"72280\",\"os_ver\":\"10\",\"aid\":\"bb242b3f3eab8716\",\"tmeLoginType\":\"0\",\"tmeLoginMethod\":\"0\",\"phonetype\":\"Pixel\",\"fPersonality\":\"0\",\"devicelevel\":\"30\",\"newdevicelevel\":\"10\",\"deviceScore\":\"302.99\",\"QIMEI36\":\"184fdeb5f885f741d3f7558710001661840f\",\"taid\":\"0101869F3CA41CEFD47D5CF7E7F3189CA2F6A60E8CBB995D3B9634121A80264EE3A4029C112D70E06B2ACCC5\",\"tmeAppID\":\"qqmusic\",\"tid\":\"4432201308658353152\",\"modeSwitch\":\"6\",\"teenMode\":\"0\",\"M-Value\":\"pCS3+/6PXCG82WGlXyTbpQ\\u003d\\u003d\",\"ui_mode\":\"1\",\"nettype\":\"1030\",\"wid\":\"5482662491\",\"rom\":\"google/google/sailfish/sailfish:10/QP1A.191005.007.A3/5972272:user/release-keys/\",\"v4ip\":\"120.244.191.20\",\"hotfix\":\"100000000\",\"traceid\":\"11_05482662491_1716635772\",\"free_mode\":\"0\",\"ext\":\"{\\\"bluetooth\\\":\\\"\\\"}\",\"br\":\"1\"},\"music.diyplaylist.PlDiyInfoRead.Read\":{\"module\":\"music.diyplaylist.PlDiyInfoRead\",\"method\":\"Read\",\"param\":{\"strTid\":\"7581901981\",\"from\":\"1\"}},\"music.video.TencentVideoQuery.GetVideoListByPlaylistId\":{\"module\":\"music.video.TencentVideoQuery\",\"method\":\"GetVideoListByPlaylistId\",\"param\":{\"playlistId\":7581901981}},\"music.srfDissInfo.PlSongExtServer.getPlSongExtInfo\":{\"module\":\"music.srfDissInfo.PlSongExtServer\",\"method\":\"getPlSongExtInfo\",\"param\":{\"tid\":7581901981,\"need\":[1,2]}},\"music.SspAdvert.PhoenixAdProxySvr.GetAdByPos\":{\"module\":\"music.SspAdvert.PhoenixAdProxySvr\",\"method\":\"GetAdByPos\",\"param\":{\"adPos\":122,\"pt\":\"banner-folder\",\"subPt\":\"\",\"uin\":\"null\",\"params\":{\"extItem-1\":\"1\",\"extItem-2\":\"{\\\"open_udid\\\":\\\"ffffffffc99f7b9c000000000033c587\\\",\\\"android_id\\\":\\\"bb242b3f3eab8716\\\",\\\"id_type\\\":2,\\\"mac_addr\\\":\\\"02:00:00:00:00:00\\\",\\\"hw_model\\\":\\\"Pixel\\\",\\\"hw_machine\\\":\\\"sailfish\\\",\\\"os_version\\\":\\\"Android 10\\\",\\\"mid\\\":\\\"a8ddfcf170a5188c43059c87bd4c45ddefd9769a\\\",\\\"sdk_version\\\":\\\"QNaPhoneV5.5.5\\\",\\\"app_name\\\":\\\"QQ音乐 13.5.0.8\\\",\\\"brands\\\":\\\"google\\\",\\\"net_status\\\":\\\"wifi\\\",\\\"chid\\\":\\\"1\\\",\\\"resolution\\\":\\\"1080x1794\\\",\\\"req_extend\\\":{\\\"id1\\\":\\\"7581901981\\\",\\\"type\\\":2,\\\"id8\\\":\\\"0\\\"},\\\"music_ad_id\\\":\\\"10501\\\",\\\"source\\\":\\\"\\\"}\",\"phoenix_pos\":\"{\\\"confs\\\":[{\\\"posType\\\":1,\\\"value\\\":\\\"7581901981\\\"}]}\"}}},\"music.srfDissInfo.DissInfo.CgiGetDiss\":{\"module\":\"music.srfDissInfo.DissInfo\",\"method\":\"CgiGetDiss\",\"param\":{\"new_format\":1,\"disstid\":7581901981,\"enc_host_uin\":\"\",\"dirid\":0,\"onlysonglist\":0,\"need_game_ad\":1,\"optype\":2,\"orderlist\":1,\"guid\":\"ffffffffc99f7b9c0000018ee03fd787\",\"tag\":1,\"userinfo\":1,\"is_mobile\":1,\"censor_status\":1,\"local_time\":1716635755,\"update_rtime\":1,\"rec_flag\":1,\"expire_couple\":0,\"ext\":{\"bluetooth\":\"\"}}},\"music.srfDissInfo.PlExtServer.getPlExtInfo\":{\"module\":\"music.srfDissInfo.PlExtServer\",\"method\":\"getPlExtInfo\",\"param\":{\"dirid\":0,\"tid\":7581901981,\"need\":[2,3,4,6],\"bWithPlTag\":true}},\"music.interactiveplaylist.PlInteractiveInfoRead.GetPlInteractiveInfo\":{\"module\":\"music.interactiveplaylist.PlInteractiveInfoRead\",\"method\":\"GetPlInteractiveInfo\",\"param\":{\"bExpireCouple\":true,\"tid\":\"7581901981\",\"bShare\":false,\"getDetail\":true}}}".getBytes(StandardCharsets.UTF_8);
        byte[] arg2 = "11&13050008&5482662491&null&1716635755643&ffffffffc99f7b9c000000000033c587&android".getBytes(StandardCharsets.UTF_8);
        String res = MERJni.callStaticJniMethodObject(emulator, "calc([B[B)Ljava/lang/String;", arg1, arg2).getValue().toString();
        System.out.println(res);
    }
    public static void main(String[] args) {
        Music music = new Music();
        music.callCalc();
    }
}

```

运行代码 报错如下

```
JNIEnv->FindClass(com/tencent/qqmusic/MusicApplication) was called from RX@0x400460f1[libmer.so]0x460f1
JNIEnv->GetStaticMethodID(com/tencent/qqmusic/MusicApplication.getContext()Landroid/content/Context;) => 0x7cd42176 was called from RX@0x4004622d[libmer.so]0x4622d
[19:29:43 601]  WARN [com.github.unidbg.linux.ARM32SyscallHandler] (ARM32SyscallHandler:532) - handleInterrupt intno=2, NR=-1073746824, svcNumber=0x171, PC=unidbg@0xfffe07a4, LR=RX@0x400455eb[libmer.so]0x455eb, syscall=null
java.lang.UnsupportedOperationException: com/tencent/qqmusic/MusicApplication->getContext()Landroid/content/Context;
  at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethodV(AbstractJni.java:503)
  at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethodV(AbstractJni.java:437)
  at com.github.unidbg.linux.android.dvm.DvmMethod.callStaticObjectMethodA(DvmMethod.java:69)
  at com.github.unidbg.linux.android.dvm.DalvikVM$114.handle(DalvikVM.java:1835)
  at com.github.unidbg.linux.ARM32SyscallHandler.hook(ARM32SyscallHandler.java:131)
  at com.github.unidbg.arm.backend.Unicorn2Backend$11.hook(Unicorn2Backend.java:347)
  at com.github.unidbg.arm.backend.unicorn.Unicorn$NewHook.onInterrupt(Unicorn.java:109)

```

常规报错补环境就行 unidbg 怎么补环境可以在龙哥星球学 这里就不过多介绍了 `环境补完` 运行程序 加密就出来了

```
JNIEnv->NewStringUTF("VFNVTUtVUEtHVFhETLVa8ntaqQc9I8DbUdYURO1dE+w= tZTYrid2garuBRE/iLzV80uR3lZPmwWQhZYDrp3FynNsXwaleE+jp0oppPfMHI5v+uU17MA0hhvU4avoHgPeZFxtEfBuRDqdx63G0ajZU9ooH+HsYJ4Rw5Ktw/bi+b92") was called from RX@0x4000ef87[libmer.so]0xef87
VFNVTUtVUEtHVFhETLVa8ntaqQc9I8DbUdYURO1dE+w= tZTYrid2garuBRE/iLzV80uR3lZPmwWQhZYDrp3FynNsXwaleE+jp0oppPfMHI5v+uU17MA0hhvU4avoHgPeZFxtEfBuRDqdx63G0ajZU9ooH+HsYJ4Rw5Ktw/bi+b92

```

Unidbg 算法分析
-----------

在 unidbg 输出的日志来看 密文在`0xef87`这个地址打印的 ida 里边看下这个地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3WVwbZVqeucRuMWgYibuOMZaRLAt3Eicr5p4Kh15K8OvfcPicJiaGwFvPs7Yiazk4FcdicVN0vvQiaKkkGw/640?wx_fmt=png&from=appmsg)

a2 就是密文 unidbg hook 这个函数 打印 a2 地址跟值

```
HookZz hook 0xEEC8
a2 address: 0x4031b00c
>-----------------------------------------------------------------------------<
[13:54:57 577]arg2, md5=5d9cb44a60222045366adcc3d5094eb4, hex=52314243546c6847533052595630565a544c5661386e7461715163394938446255645955524f3164452b773d2054377345774f563943356b65554e4542784730717473435359716c6b34766d6a306b39674962722b774b534e7152766b59666571584954343573325447557a5178587458425648454c4d324e7a744550387070342b375446427a35616968636f57394d366856424241427878456e56636d6b65484268446c2f676232314677320000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 256
0000: 52 31 42 43 54 6C 68 47 53 30 52 59 56 30 56 5A    R1BCTlhGS0RYV0VZ
0010: 54 4C 56 61 38 6E 74 61 71 51 63 39 49 38 44 62    TLVa8ntaqQc9I8Db
0020: 55 64 59 55 52 4F 31 64 45 2B 77 3D 20 54 37 73    UdYURO1dE+w= T7s
0030: 45 77 4F 56 39 43 35 6B 65 55 4E 45 42 78 47 30    EwOV9C5keUNEBxG0
0040: 71 74 73 43 53 59 71 6C 6B 34 76 6D 6A 30 6B 39    qtsCSYqlk4vmj0k9
0050: 67 49 62 72 2B 77 4B 53 4E 71 52 76 6B 59 66 65    gIbr+wKSNqRvkYfe
0060: 71 58 49 54 34 35 73 32 54 47 55 7A 51 78 58 74    qXIT45s2TGUzQxXt
0070: 58 42 56 48 45 4C 4D 32 4E 7A 74 45 50 38 70 70    XBVHELM2NztEP8pp
0080: 34 2B 37 54 46 42 7A 35 61 69 68 63 6F 57 39 4D    4+7TFBz5aihcoW9M
0090: 36 68 56 42 42 41 42 78 78 45 6E 56 63 6D 6B 65    6hVBBABxxEnVcmke
00A0: 48 42 68 44 6C 2F 67 62 32 31 46 77 32 00 00 00    HBhDl/gb21Fw2...
00B0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00C0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00D0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00E0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

```

监控 0x4031b00c -> 0x4031b00c+44 这个地址段的写入

```
emulator.traceWrite(0x4031b00cL, 0x4031b00cL + 44);

```

```
[14:02:33 879] Memory WRITE at 0x4031b00c, data size = 1, data value = 0x53, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b00d, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b00e, data size = 1, data value = 0x68, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b00f, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b010, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b011, data size = 1, data value = 0x6b, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b012, data size = 1, data value = 0x5a, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b013, data size = 1, data value = 0x49, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b014, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b015, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b016, data size = 1, data value = 0x5a, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b017, data size = 1, data value = 0x43, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b018, data size = 1, data value = 0x57, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b019, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 879] Memory WRITE at 0x4031b01a, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b01b, data size = 1, data value = 0x52, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b01c, data size = 1, data value = 0x54, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b01d, data size = 1, data value = 0x4c, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b01e, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b01f, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b020, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b021, data size = 1, data value = 0x6e, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b022, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b023, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b024, data size = 1, data value = 0x71, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b025, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b026, data size = 1, data value = 0x63, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b027, data size = 1, data value = 0x39, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b028, data size = 1, data value = 0x49, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b029, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02a, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02b, data size = 1, data value = 0x62, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02c, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02d, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02e, data size = 1, data value = 0x59, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b02f, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b030, data size = 1, data value = 0x52, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b031, data size = 1, data value = 0x4f, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b032, data size = 1, data value = 0x31, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b033, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b034, data size = 1, data value = 0x45, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b035, data size = 1, data value = 0x2b, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b036, data size = 1, data value = 0x77, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b037, data size = 1, data value = 0x3d, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
[14:02:33 880] Memory WRITE at 0x4031b038, data size = 1, data value = 0x20, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x40077c17[libmer.so]0x77c17
HookZz hook 0xEEC8
a2 address: 0x4031b00c
>-----------------------------------------------------------------------------<
[14:02:33 881]arg2, md5=4214e2938ff6eb13e2a6094e60d0433f, hex=53556851556b5a4955565a4357557452544c5661386e7461715163394938446255645955524f3164452b773d206548502b464473774830413238563174364b796e4354367552783153677a4963586c2f79626b52486c4c3567677543415745373443714a2b5479444d574b63316d314d387a475846525a78475a2f396f694b6c674b307773474c753343596932422f6f515a5257476863493754364b726733437054726447415a5a4b77494d430000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 256
0000: 53 55 68 51 55 6B 5A 49 55 56 5A 43 57 55 74 52    SUhQUkZIUVZCWUtR
0010: 54 4C 56 61 38 6E 74 61 71 51 63 39 49 38 44 62    TLVa8ntaqQc9I8Db
0020: 55 64 59 55 52 4F 31 64 45 2B 77 3D 20 65 48 50    UdYURO1dE+w= eHP
0030: 2B 46 44 73 77 48 30 41 32 38 56 31 74 36 4B 79    +FDswH0A28V1t6Ky
0040: 6E 43 54 36 75 52 78 31 53 67 7A 49 63 58 6C 2F    nCT6uRx1SgzIcXl/
0050: 79 62 6B 52 48 6C 4C 35 67 67 75 43 41 57 45 37    ybkRHlL5gguCAWE7
0060: 34 43 71 4A 2B 54 79 44 4D 57 4B 63 31 6D 31 4D    4CqJ+TyDMWKc1m1M
0070: 38 7A 47 58 46 52 5A 78 47 5A 2F 39 6F 69 4B 6C    8zGXFRZxGZ/9oiKl
0080: 67 4B 30 77 73 47 4C 75 33 43 59 69 32 42 2F 6F    gK0wsGLu3CYi2B/o
0090: 51 5A 52 57 47 68 63 49 37 54 36 4B 72 67 33 43    QZRWGhcI7T6Krg3C
00A0: 70 54 72 64 47 41 5A 5A 4B 77 49 4D 43 00 00 00    pTrdGAZZKwIMC...
00B0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00C0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00D0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00E0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

```

发现数据在 0x77c17 写入 ida 查看这个地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3WVwbZVqeucRuMWgYibuOMZPbicrJ5IfclPWHvwxmclQfuEQdvt6zYGIwibiamaXiczN0FvDdUiaEh2fwg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k3WVwbZVqeucRuMWgYibuOMZ2WCKtMe4u6cEon87ymyQCqzW755aibtvmd8OouCesTMGzziavneuvQKA/640?wx_fmt=png&from=appmsg)

a1 就是密文 hook 下 sub_77BF4 这个函数 打印 a1 地址跟值

```
HookZz hook 0x77BF4
a1 address: 0x4023400c
>-----------------------------------------------------------------------------<
[14:10:31 676]arg2, md5=4428e16c825f28f249514c7e9442078a, hex=526c4e4a5530564a566c4652536c5a47544c5661386e7461715163394938446255645955524f3164452b773d2030672f4f42563544346b4e5673667258686b33776649575930714e61304c6338583349594d72646a72705350472b376c594b5354654e384b652b71617849487363517978392b68366d2f5478784e495472726a6a764868326d3843326d727246375a4c6246446f32794f435a596a6645704b39545156314954736a6d643436500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 256
0000: 52 6C 4E 4A 55 30 56 4A 56 6C 46 52 53 6C 5A 47    RlNJU0VJVlFRSlZG
0010: 54 4C 56 61 38 6E 74 61 71 51 63 39 49 38 44 62    TLVa8ntaqQc9I8Db
0020: 55 64 59 55 52 4F 31 64 45 2B 77 3D 20 30 67 2F    UdYURO1dE+w= 0g/
0030: 4F 42 56 35 44 34 6B 4E 56 73 66 72 58 68 6B 33    OBV5D4kNVsfrXhk3
0040: 77 66 49 57 59 30 71 4E 61 30 4C 63 38 58 33 49    wfIWY0qNa0Lc8X3I
0050: 59 4D 72 64 6A 72 70 53 50 47 2B 37 6C 59 4B 53    YMrdjrpSPG+7lYKS
0060: 54 65 4E 38 4B 65 2B 71 61 78 49 48 73 63 51 79    TeN8Ke+qaxIHscQy
0070: 78 39 2B 68 36 6D 2F 54 78 78 4E 49 54 72 72 6A    x9+h6m/TxxNITrrj
0080: 6A 76 48 68 32 6D 38 43 32 6D 72 72 46 37 5A 4C    jvHh2m8C2mrrF7ZL
0090: 62 46 44 6F 32 79 4F 43 5A 59 6A 66 45 70 4B 39    bFDo2yOCZYjfEpK9
00A0: 54 51 56 31 49 54 73 6A 6D 64 34 36 50 00 00 00    TQV1ITsjmd46P...
00B0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00C0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00D0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00E0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

```

继续跟进 监控 0x4023400c->0x4023400c+44 这个地址段的写入

```
emulator.traceWrite(0x4023400cL, 0x4023400cL + 44);

```

```
[10:04:48 834] Memory WRITE at 0x4023400c, data size = 1, data value = 0x0, PC=RX@0x40089666[libmer.so]0x89666, LR=unidbg@0x18
[10:04:48 834] Memory WRITE at 0x4023400c, data size = 1, data value = 0x57, PC=RX@0x400792e0[libmer.so]0x792e0, LR=unidbg@0x18
[10:04:48 834] Memory WRITE at 0x4023400d, data size = 1, data value = 0x0, PC=RX@0x400792e0[libmer.so]0x792e0, LR=unidbg@0x18
[10:04:48 834] Memory WRITE at 0x4023400d, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x56
[10:04:48 834] Memory WRITE at 0x4023400e, data size = 1, data value = 0x4e, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x4023400f, data size = 1, data value = 0x50, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234010, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234011, data size = 1, data value = 0x6b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234012, data size = 1, data value = 0x46, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234013, data size = 1, data value = 0x4e, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234014, data size = 1, data value = 0x54, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234015, data size = 1, data value = 0x6b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234016, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234017, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234018, data size = 1, data value = 0x57, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x40234019, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x4023401a, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x4023401b, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 834] Memory WRITE at 0x4023401c, data size = 1, data value = 0x54, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023401d, data size = 1, data value = 0x4c, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023401e, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023401f, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234020, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234021, data size = 1, data value = 0x6e, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234022, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234023, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234024, data size = 1, data value = 0x71, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234025, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234026, data size = 1, data value = 0x63, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234027, data size = 1, data value = 0x39, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234028, data size = 1, data value = 0x49, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234029, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402a, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402b, data size = 1, data value = 0x62, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402c, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402d, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402e, data size = 1, data value = 0x59, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x4023402f, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234030, data size = 1, data value = 0x52, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234031, data size = 1, data value = 0x4f, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234032, data size = 1, data value = 0x31, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234033, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234034, data size = 1, data value = 0x45, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234035, data size = 1, data value = 0x2b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 835] Memory WRITE at 0x40234036, data size = 1, data value = 0x77, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 836] Memory WRITE at 0x40234037, data size = 1, data value = 0x3d, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x50
[10:04:48 836] Memory WRITE at 0x40234038, data size = 1, data value = 0x20, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x20
HookZz hook 0x77BF4
a1 address: 0x4023400c
>-----------------------------------------------------------------------------<
[10:04:48 836]arg1, md5=5dd25e406643ab841b3b5e78f1bfb88e, hex=57564e50566b464e546b744457566461544c5661386e7461715163394938446255645955524f3164452b773d207938647157654c304e334364666b4c6c74526d5346384e51676e523859774f57596f416b467361414b686656675a6c706b4b4655503769546b333870552f756e6e76645875594e537032572b71715639395034455546464c763248334d74734b4d32394f7537312b4638554848676556714b746f6f5a43334c39786d6b4f4c510000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 256
0000: 57 56 4E 50 56 6B 46 4E 54 6B 74 44 57 56 64 61    WVNPVkFNTktDWVda
0010: 54 4C 56 61 38 6E 74 61 71 51 63 39 49 38 44 62    TLVa8ntaqQc9I8Db
0020: 55 64 59 55 52 4F 31 64 45 2B 77 3D 20 79 38 64    UdYURO1dE+w= y8d
0030: 71 57 65 4C 30 4E 33 43 64 66 6B 4C 6C 74 52 6D    qWeL0N3CdfkLltRm
0040: 53 46 38 4E 51 67 6E 52 38 59 77 4F 57 59 6F 41    SF8NQgnR8YwOWYoA
0050: 6B 46 73 61 41 4B 68 66 56 67 5A 6C 70 6B 4B 46    kFsaAKhfVgZlpkKF
0060: 55 50 37 69 54 6B 33 38 70 55 2F 75 6E 6E 76 64    UP7iTk38pU/unnvd
0070: 58 75 59 4E 53 70 32 57 2B 71 71 56 39 39 50 34    XuYNSp2W+qqV99P4
0080: 45 55 46 46 4C 76 32 48 33 4D 74 73 4B 4D 32 39    EUFFLv2H3MtsKM29
0090: 4F 75 37 31 2B 46 38 55 48 48 67 65 56 71 4B 74    Ou71+F8UHHgeVqKt
00A0: 6F 6F 5A 43 33 4C 39 78 6D 6B 4F 4C 51 00 00 00    ooZC3L9xmkOLQ...
00B0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00C0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00D0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00E0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

```

看输出 除了前三个字节在 libmer.so 写入 其余字节在 libc.so 里面写入的 修改一下代码 打印调用栈 查看上层调用地址

```
emulator.traceWrite(0x4023400cL, 0x4023400cL + 44, (emulator, address, size, value) -> {
    emulator.getUnwinder().unwind();
    return true;
});

```

```
[0x40000000][0x400792dd][   libmer.so][0x792dd]
[0x40000000][0x4007c073][   libmer.so][0x7c073]
[0x40000000][0x400750bd][   libmer.so][0x750bd]
[0x40000000][0x4000e46d][   libmer.so][0x0e46d] Java_com_tencent_qqmusic_modular_framework_encrypt_logic_MERJni_calc + 0x9a5
[10:08:44 922] Memory WRITE at 0x4023400c, data size = 1, data value = 0x0, PC=RX@0x40089666[libmer.so]0x89666, LR=unidbg@0x18
[0x40000000][0x4007c073][   libmer.so][0x7c073]
[0x40000000][0x400750bd][   libmer.so][0x750bd]
[0x40000000][0x4000e46d][   libmer.so][0x0e46d] Java_com_tencent_qqmusic_modular_framework_encrypt_logic_MERJni_calc + 0x9a5
[10:08:44 922] Memory WRITE at 0x4023400c, data size = 1, data value = 0x56, PC=RX@0x400792e0[libmer.so]0x792e0, LR=unidbg@0x18
[0x40000000][0x4007c073][   libmer.so][0x7c073]
[0x40000000][0x400750bd][   libmer.so][0x750bd]
[0x40000000][0x4000e46d][   libmer.so][0x0e46d] Java_com_tencent_qqmusic_modular_framework_encrypt_logic_MERJni_calc + 0x9a5
[10:08:44 923] Memory WRITE at 0x4023400d, data size = 1, data value = 0x0, PC=RX@0x400792e0[libmer.so]0x792e0, LR=unidbg@0x18
[0x00000000][0x0000002c][        0x2c]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 923] Memory WRITE at 0x4023400d, data size = 1, data value = 0x30, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x30
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 923] Memory WRITE at 0x4023400e, data size = 1, data value = 0x31, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 923] Memory WRITE at 0x4023400f, data size = 1, data value = 0x48, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 924] Memory WRITE at 0x40234010, data size = 1, data value = 0x57, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 924] Memory WRITE at 0x40234011, data size = 1, data value = 0x6b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 924] Memory WRITE at 0x40234012, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234013, data size = 1, data value = 0x4a, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234014, data size = 1, data value = 0x52, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234015, data size = 1, data value = 0x6b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234016, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234017, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 925] Memory WRITE at 0x40234018, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 926] Memory WRITE at 0x40234019, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 926] Memory WRITE at 0x4023401a, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 926] Memory WRITE at 0x4023401b, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 926] Memory WRITE at 0x4023401c, data size = 1, data value = 0x54, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 927] Memory WRITE at 0x4023401d, data size = 1, data value = 0x4c, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 927] Memory WRITE at 0x4023401e, data size = 1, data value = 0x56, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 927] Memory WRITE at 0x4023401f, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 927] Memory WRITE at 0x40234020, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 928] Memory WRITE at 0x40234021, data size = 1, data value = 0x6e, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 928] Memory WRITE at 0x40234022, data size = 1, data value = 0x74, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 928] Memory WRITE at 0x40234023, data size = 1, data value = 0x61, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 928] Memory WRITE at 0x40234024, data size = 1, data value = 0x71, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 929] Memory WRITE at 0x40234025, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 929] Memory WRITE at 0x40234026, data size = 1, data value = 0x63, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 929] Memory WRITE at 0x40234027, data size = 1, data value = 0x39, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 929] Memory WRITE at 0x40234028, data size = 1, data value = 0x49, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 930] Memory WRITE at 0x40234029, data size = 1, data value = 0x38, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 930] Memory WRITE at 0x4023402a, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 930] Memory WRITE at 0x4023402b, data size = 1, data value = 0x62, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 930] Memory WRITE at 0x4023402c, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 930] Memory WRITE at 0x4023402d, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x4023402e, data size = 1, data value = 0x59, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x4023402f, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x40234030, data size = 1, data value = 0x52, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x40234031, data size = 1, data value = 0x4f, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x40234032, data size = 1, data value = 0x31, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 931] Memory WRITE at 0x40234033, data size = 1, data value = 0x64, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 932] Memory WRITE at 0x40234034, data size = 1, data value = 0x45, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 932] Memory WRITE at 0x40234035, data size = 1, data value = 0x2b, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 932] Memory WRITE at 0x40234036, data size = 1, data value = 0x77, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x00000044][        0x44]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 933] Memory WRITE at 0x40234037, data size = 1, data value = 0x3d, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x48
[0x00000000][0x0000001c][        0x1c]
[0x40000000][0x4007c08b][   libmer.so][0x7c08b]
[10:08:44 933] Memory WRITE at 0x40234038, data size = 1, data value = 0x20, PC=RX@0x400f4608[libc.so]0x17608, LR=unidbg@0x20
HookZz hook 0x77BF4
a1 address: 0x4023400c
>-----------------------------------------------------------------------------<
[10:08:44 933]arg1, md5=383b4c844f77b28b674ef309459b56c1, hex=56303148576b564a526b745656556461544c5661386e7461715163394938446255645955524f3164452b773d204a693535496446363139477446585368354e4e344c55654b4e7a536f4c503354512f63715a71457567496e6551576d61446e2b49495a5a485a4c756d4a6a3639766c772f474e43344f38466f3039762f39385173757766393939303963444b62594c727069614439654138377a31454b776366796c376447506d4132656b38790000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
size: 256
0000: 56 30 31 48 57 6B 56 4A 52 6B 74 56 56 55 64 61    V01HWkVJRktVVUda
0010: 54 4C 56 61 38 6E 74 61 71 51 63 39 49 38 44 62    TLVa8ntaqQc9I8Db
0020: 55 64 59 55 52 4F 31 64 45 2B 77 3D 20 4A 69 35    UdYURO1dE+w= Ji5
0030: 35 49 64 46 36 31 39 47 74 46 58 53 68 35 4E 4E    5IdF619GtFXSh5NN
0040: 34 4C 55 65 4B 4E 7A 53 6F 4C 50 33 54 51 2F 63    4LUeKNzSoLP3TQ/c
0050: 71 5A 71 45 75 67 49 6E 65 51 57 6D 61 44 6E 2B    qZqEugIneQWmaDn+
0060: 49 49 5A 5A 48 5A 4C 75 6D 4A 6A 36 39 76 6C 77    IIZZHZLumJj69vlw
0070: 2F 47 4E 43 34 4F 38 46 6F 30 39 76 2F 39 38 51    /GNC4O8Fo09v/98Q
0080: 73 75 77 66 39 39 39 30 39 63 44 4B 62 59 4C 72    suwf99909cDKbYLr
0090: 70 69 61 44 39 65 41 38 37 7A 31 45 4B 77 63 66    piaD9eA87z1EKwcf
00A0: 79 6C 37 64 47 50 6D 41 32 65 6B 38 79 00 00 00    yl7dGPmA2ek8y...
00B0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00C0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00D0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00E0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
00F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
^-----------------------------------------------------------------------------^

```

ida 里面查看 0x7c08b 地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32crghWp59OicuQjyfLibTExwic8vkeZ382l9GeppkQ0eVnOqF1fNrUKkicw/640?wx_fmt=png&from=appmsg)

下面的操作就跟上面一样了 hook 拿到存储密文的地址 + traceWrite 监控地址的写入

最后定位到下面这个函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32iblCs3Kp9Po0evOxXwvLuR8Gyic23U56JvPvs9msy0wzrMRSjibibBG6Sg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32vYZJBUQoy2IWkl7C9piberytHcfsFetNmmhuUla6sLBRWibR1OwHNB1w/640?wx_fmt=png&from=appmsg)

这里就比较明显了 base64 编码函数 hook 函数 拿到密文编码前的地址 再溯源

```
[10:48:24 582] Memory WRITE at 0xbffff0c8, data size = 1, data value = 0x55, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0c9, data size = 1, data value = 0x4d, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0ca, data size = 1, data value = 0x41, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0cb, data size = 1, data value = 0x42, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0cc, data size = 1, data value = 0x53, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0cd, data size = 1, data value = 0x49, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0ce, data size = 1, data value = 0x54, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0cf, data size = 1, data value = 0x41, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0d0, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0d1, data size = 1, data value = 0x57, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0d2, data size = 1, data value = 0x4b, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0d3, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005947d[libmer.so]0x5947d
[10:48:24 582] Memory WRITE at 0xbffff0d4, data size = 1, data value = 0x4c, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 582] Memory WRITE at 0xbffff0d5, data size = 1, data value = 0xb5, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 582] Memory WRITE at 0xbffff0d6, data size = 1, data value = 0x5a, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 582] Memory WRITE at 0xbffff0d7, data size = 1, data value = 0xf2, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0d8, data size = 1, data value = 0x7b, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0d9, data size = 1, data value = 0x5a, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0da, data size = 1, data value = 0xa9, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0db, data size = 1, data value = 0x7, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0dc, data size = 1, data value = 0x3d, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0dd, data size = 1, data value = 0x23, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0de, data size = 1, data value = 0xc0, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0df, data size = 1, data value = 0xdb, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e0, data size = 1, data value = 0x51, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e1, data size = 1, data value = 0xd6, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e2, data size = 1, data value = 0x14, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e3, data size = 1, data value = 0x44, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e4, data size = 1, data value = 0xed, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e5, data size = 1, data value = 0x5d, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e6, data size = 1, data value = 0x13, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
[10:48:24 583] Memory WRITE at 0xbffff0e7, data size = 1, data value = 0xec, PC=RX@0x400f4608[libc.so]0x17608, LR=RX@0x4005948d[libmer.so]0x5948d
HookZz hook 0x174CC
a2 address: 0xbffff0c8
>-----------------------------------------------------------------------------<
[10:48:24 583]arg2, md5=b17c9a027e8e223740ec4ab9f30a800e, hex=554d41425349544151574b444cb55af27b5aa9073d23c0db51d61444ed5d13ec00000000000000000000000000000000000000000000000000000000000000004cb55af27b5aa9073d23c0db51d61444ed5d13ec576712401400000040f1ffbf18d02040e4011c405e7952e1673f14d0684016d75396e2666a4f13d219efb087e8011c40e4011c4006000000640820400c00000054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b40dcab0b4088aa0b40dcab0b4088aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40d8aa0b4054ac0b40
size: 256
0000: 55 4D 41 42 53 49 54 41 51 57 4B 44 4C B5 5A F2    UMABSITAQWKDL.Z.
0010: 7B 5A A9 07 3D 23 C0 DB 51 D6 14 44 ED 5D 13 EC    {Z..=#..Q..D.]..
0020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0040: 4C B5 5A F2 7B 5A A9 07 3D 23 C0 DB 51 D6 14 44    L.Z.{Z..=#..Q..D
0050: ED 5D 13 EC 57 67 12 40 14 00 00 00 40 F1 FF BF    .]..Wg.@....@...
0060: 18 D0 20 40 E4 01 1C 40 5E 79 52 E1 67 3F 14 D0    .. @...@^yR.g?..
0070: 68 40 16 D7 53 96 E2 66 6A 4F 13 D2 19 EF B0 87    h@..S..fjO......
0080: E8 01 1C 40 E4 01 1C 40 06 00 00 00 64 08 20 40    ...@...@....d. @
0090: 0C 00 00 00 54 AC 0B 40 D8 AA 0B 40 54 AC 0B 40    ....T..@...@T..@
00A0: D8 AA 0B 40 54 AC 0B 40 D8 AA 0B 40 54 AC 0B 40    ...@T..@...@T..@
00B0: D8 AA 0B 40 54 AC 0B 40 D8 AA 0B 40 54 AC 0B 40    ...@T..@...@T..@
00C0: D8 AA 0B 40 54 AC 0B 40 D8 AA 0B 40 DC AB 0B 40    ...@T..@...@...@
00D0: 88 AA 0B 40 DC AB 0B 40 88 AA 0B 40 54 AC 0B 40    ...@...@...@T..@
00E0: D8 AA 0B 40 54 AC 0B 40 D8 AA 0B 40 54 AC 0B 40    ...@T..@...@T..@
00F0: D8 AA 0B 40 54 AC 0B 40 D8 AA 0B 40 54 AC 0B 40    ...@T..@...@T..@
^-----------------------------------------------------------------------------^

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32UXLC0uDcOdVibLttM8JQO6JwCCensR621x7qMR4cZ7U7Y2aic4fPrngw/640?wx_fmt=png&from=appmsg)

定位到这里 从 v480 获取 20 字节 从 * v451 获取 12 字节 就组成了编码前的 32 字节 多次运行程序可以发现 变化的只有前 12 字节 后面 20 字节是固定的 合理猜测 20 字节就是某种哈希的结果 继续回溯 密文字节地址的写入 定位到最终密文在这里生成

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32ibYkDnyMDH1BO3znIfLTwZw0PdxEcoVjNpbZoJ7uscz3tuXOgCp2o6A/640?wx_fmt=png&from=appmsg)

粗略看 看不出来什么东西 debugger 下这个函数 查看函数入参

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EiaqCy3pb5k0Y7PEUBz0r8PialgV0eAia32C6rr2wKZtTmbMQf6ww5iaHSambpfnskY4h2osep5WbgWqVnn29bWTng/640?wx_fmt=png&from=appmsg)

到这里 熟悉常见加密算法特征的朋友看到内存中的这个值应该已经知道答案了 为了防止侵权这里就不点出来了 最后验证过了 该哈希没有魔改 再好好分析下入参就能很快搞定

最后总结下 sign 加密流程：

    1. 对 so 层第一个入参进行 base64 编码

    2. 将 base64 编码结果翻转

    3. 进行哈希签名

    4. 哈希之后字节拼接前 12 个随机字节

    5. base64 编码拿到最后密文

今天的分享到这里就结束了 咱们下次再会~