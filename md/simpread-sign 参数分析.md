> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Kq8VRVGL_PMPjVvr7zVzzA)

接口参数定位
======

目标接口：搜索 https://api.m.ooxx.com/client.action?functionId=search

版本：13.1.0

目标参数：body 中的 x-api-eid-token  和 sign

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fum6rPywWLQem3PDKyViaIA2r9Nh7Sq0OBTz8eIuibIjrIjxcXr92MEng/640?wx_fmt=png&from=appmsg)

定位接口参数的方法

◆搜索大发

◆hook Hashmap

◆Hook StringBuilder

这里我们直接使用最朴素的搜索大法，一次就中。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fMPxzx7PpicTDS2bziaKGb7oGVK5YotMRf6I2rY9H8jXuaxorQxqJquAA/640?wx_fmt=png&from=appmsg)

继续跟踪，进最下面这个看看。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fcKLaX4xhsE3yTedBIbl28JaNb1yOuMbKPQI9oxByqW0RXcxQMn9uNQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fAxIfCN9hP41AYX2FBbCicLJkZs68IGma8KcEfiav8KoobickaSRGrjAwQ/640?wx_fmt=png&from=appmsg)

下面拼接了 urlEncodeUTF82  ，看英文就是一个 utf 编码之后的数据。继续向上跟踪

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fXaVaV79pVmIrjJVHs40nyfC5vWdSwZKlqkl9oXoUDu1w9EKeicywUCw/640?wx_fmt=png&from=appmsg)

进入第一个，函数返回的是一个字符串。还不是加密的地方。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fcx1QkN5HYic9649TZVdkrHh1ibV4IG68SesdTnq1NwOibE8FQ8oblpL6w/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f8rz1ZVzZqspln52rUGfMhLK0fyxtxWSByLicforbATjulAXMW4RNSAw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fWr6LH72KPs1icsP2RmCxicVIbUZmR3L2GkPSZfaGotRpWn9VQqPRSE9w/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fGArL15RjrCAeU3xnBzOic3l1sukAN4DCI4ZiaQIdAyeHUwpric4V5rm6A/640?wx_fmt=png&from=appmsg)

JDHttpTookit.getEngine().getSignatureHandlerImpl().signature 这就是签名函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fdoLP9DibU7CiamLJs1x6BTr26StNHnvpkc9ciaxFwAduHU4ZHbkGwfotQ/640?wx_fmt=png&from=appmsg)

接口函数一般是 implement 或者直接 new 出来。这里跟进 new 出来的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fm3BxIaqBomHibbhxbPsTsD4IH4d2I8ttex1q8esbg7gB5ZViaH4PVryQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fiaQKWkZpXB5DO0T31frx4YMNiahJZpfLeyvibjW1W33Vq6gjLD0upps5Q/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f9ZW0BdueYsT9dicK5fgoOovEFjELeSibeTCRcytdI07zzzfxtVZxbQibg/640?wx_fmt=png&from=appmsg)

Hook 获取参数
---------

通过 frida 获取传入的参数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fzR9NAlz941n9uPlict6c6INfOqJ4xQ2Wovu0B7lULrjbiaZV2B2ZAkpw/640?wx_fmt=png&from=appmsg)

主动调用，方便后续做测试

```
function call_by_java() {
    Java.perform(function(){
        let BitmapkitUtils = Java.use("com.ooxx.common.utils.BitmapkitUtils");
        let context = Java.use("android.app.ActivityThread").currentApplication().getApplicationContext(); 
        let str = 'search'
        let str2 = '{"addrFilter":"1","addressId":"0","articleEssay":"1","attrRet":"0","buriedExpLabel":"","deviceidTail":"38","exposedCount":"0","filterServiceIds":"1468131091","first_search":"1","frontExpids":"F_001","gcAreaId":"1,72,55674,0","gcLat":"39.944093","gcLng":"116.482276","imagesize":{"gridImg":"531x531","listImg":"358x358","longImg":"531x708"},"insertArticle":"1","insertScene":"1","insertedCount":"0","isCorrect":"1","jdv":"0|kong|t_2018512525_cpv_nopay|tuiguang|17303608941925019140008|1730360893","keyword":"空气加湿器","localNum":"2","newMiddleTag":"1","newVersion":"3","oneBoxMod":"1","orignalSearch":"1","orignalSelect":"1","page":"1","pageEntrance":"1","pagesize":"10","populationType":"232","pvid":"","searchVersionCode":"10110","secondInsedCount":"0","showShopTab":"yes","showStoreTab":"1","show_posnum":"0","sourceRef":[{"action":"","eventId":"MyJD_WordSizeResult","isDirectSearch":"0","logid":"","pageId":"Home_Main","pvId":""},{"action":"","eventId":"Search_History","isDirectSearch":"0","logid":"","pageId":"Search_Activity","pvId":"632ba208e4854bb1839e6e32a5e6b841"}],"stock":"1","ver":"142"}'
        let str3 = "bd132c578e85c7cd";
        let str4 = "android";
        let str5 = "13.1.0";

        let result = BitmapkitUtils.getSignFromJni(context, str, str2, str3, str4, str5);
        console.log("BitmapkitUtils.getSignFromJni result = " + result);
    })
}


```

```
BitmapkitUtils.getSignFromJni result = st=1731485532078&sign=3c9820dce84fc15ceaf29bf0e0630306&sv=111


```

结果中就包含了我们今天的主角 ==sign==  。长度为 32 脑海中就冒出 MD5 了。

确定 so
-----

调用到了 native 层了。java 层的分析就到这里了。这里类中并没有看到加载 so 。需要通过 frida hook 导出的符号表，反查 so 的名字。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fEia1FGGkpmEAspV8XPTbngIP1ZsAGRb0PWKMMLJ5utlUSs95iarLTQkA/640?wx_fmt=png&from=appmsg)

得到模板 so 的名字为  libjdbitmapkit.so

so 分析
=====

IDA 打开可以看到 getSignFromJni 的导出函数 。通过 IDA frida-trace 先 trace 一份日志方便后面分析

工具：https://github.com/Pr0214/trace_natives

把下载到的 traceNatives.py 放到 ida 根目录的 plugin 目录下重启 IDA。点击 edit -> plugins -> traceNatives 会生成一个文件夹给 frida-trace 使用。

```
 frida-trace -H iP:port -F -O D:\ooxx\libjdbitmapkit_1731482430.txt


```

frida 就会开始 trace 此时不要关闭命令行，直接调用刚才的主动触发的函数 call_by_java() 就会生成追踪数据，复制保存成一个 log 文件方便后面分析。

再用 findHash 插件看看能不能找到什么有用的信息

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fFw4g6ZqxUlzFrvj1yjLJLmuiag7Zgk35RRy5LOL276czJd6BVj8VTnw/640?wx_fmt=png&from=appmsg)

把两个数据做对比发现了 sub_27A4 这个函数在两边都有出现。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fv0fuC6V7uCkibOBm9sibYuaO8iao3ic36DCAqfNgG9e1IAKRWyIwFuruWQ/640?wx_fmt=png&from=appmsg)

IDA 中按 g 跳转过去看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5flmjYfWw5C5QZ8IuKygAbmsEeFibtQUooM8gfFZvM2Ff4icXYtJzR0Cjg/640?wx_fmt=png&from=appmsg)

看上去很像 MD5 哦，点击数字按 h 转换成十六进制然后去搜索一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fzkwicUPm26L3cT61vN0Pk7Ngmn427NMwnjNxdBK6L3hb2bzbAvKrYoA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fqVQkglSUfVCKn1hvicK9gkpV5AocDQjxkibRiarhdxYiaRkdicrPicYmC18g/640?wx_fmt=png&from=appmsg)

有点意思！应该是 MD5。查看 trace  日志，sub_8134() 是最开始的地方，IDA 中看看是什么内容。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5faib0nrwOaWOzvjn9ASJA0COX1icVibQ29Txs3eA2tXDaKfhic0VWhvu9SA/640?wx_fmt=png&from=appmsg)

经过对比发现，我们的入口函数也就是 8134 。那还有什么说的，盘他咯！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fxsM7MkfXaTSm0eNfSV4jIvZDdrib0EVhqlEPicC61g2Bs73nnzXw1LZg/640?wx_fmt=png&from=appmsg)

在 trace 日志中 8134  这层调用的最后一个函数是 33b4，进入 33b4 之后就是 MD5 算法了。我们先 hook 7e08 查看入参。

```
function print_arg(addr) {
    try {
        var module = Process.findRangeByAddress(addr);
        if (module != null) return "\n"+hexdump(addr) + "\n";
        return ptr(addr) + "\n";
    } catch (e) {
        return addr + "\n";
    }
}

function hook_native(funptr,paramsNum) {
    var md = Process.findModuleByAddress(funptr);
    console.log("hook func ");
    
    try {
        //hook 指定函数
        Interceptor.attach(funptr,{
            onEnter: function(args){
                this.logs =""
                this.params = [];
                this.logs = this.logs.concat("So: "+md.name  +"  Method: " + ptr(funptr).sub(md.base) + "\n")
                for (var i = 0; i < paramsNum; i++) {
                    //参数 
                    this.params.push(args[i]);
                    this.logs = this.logs.concat("this.args "+i+" onEnter: " +print_arg(args[i])+"\n")
                }
            },
            onLeave: function(retval){
                for (let i = 0; i < paramsNum; i++) {
                    this.logs=this.logs.concat("this.args" + i + " onLeave: " + print_arg(this.params[i]));
                }
                this.logs=this.logs.concat("retval onLeave: " + print_arg(retval) + "\n");
                console.log(this.logs);
            }
        });
    } catch (error) {
        console.log(error);
        
    }
}

 hook_native(Module.findBaseAddress("libjdbitmapkit.so").add(0x33B4), 0x3);


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fUe9cyAuysnmyjAbQ33flS1xgNqteibicFSLwmjZ5Mthv0GtU6QicpQ9qg/640?wx_fmt=png&from=appmsg)

传入的参数很像 base64 但是解密之后还是乱码。传入之前做了处理。向上追踪，在 trace 日志，上一个被调用的函数是 2698

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fpCYPicia5ccqBWVE1ShRPib5wp5RPHTbdsnvicRKlqMqcic4bUHYSwrVsEw/640?wx_fmt=png&from=appmsg)

hook  2698 看看返回值，刚好就是 33B4 的入参

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fQybvBl11Foiac5CKH0LkOL7xlukGMicRtbVpt5thgrEuMecEJ8NQIOyg/640?wx_fmt=png&from=appmsg)

跟踪 2698  第二个参数，它来自 v37 , v37 又是来自 7E08 这个函数 。而且 在 trace 日志中也有它的身影。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f0Cp2ugx87zD2varZpmch0rDzGM6gxdE7vZJ0ozDnokqvL7R4YalE5g/640?wx_fmt=png&from=appmsg)

经过分析可以发现 7E08 最后两个参数是两个随机数，就是这两个随机数决定了之后使用不同的分支算法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fdFm2iaiaIpuo0kOMibzSC1cbsW4kJmvmfJQ20uKqArJoDu1pHkTfhvicRQ/640?wx_fmt=png&from=appmsg)

进入 7E08 。先判断最后一个参数的随机数，用不同方式在生成一个随机数，然后与倒数第二参数的随机数相加产生新的随机数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fl0dkpoPCK0KiayCFXDmJUXQicbAxl58ias7oPptX3YV258lB51zs1d5ww/640?wx_fmt=png&from=appmsg)

使用新的随机数来决定最终进入那个分支的算法。特别注意一下 gen_str_44e 这是一个固定的字符串。在进入不同分支的时候，根据新的随机数取了这个字符串中不同的数据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f8ic5ZtJ3jNFhIkic69ITGMAqz0Kw75lYb9jpr4gyprfdFf3Ge3jvFENQ/640?wx_fmt=png&from=appmsg)

hook 7E08 得到的参数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f1RTbia15hzGCJwAM7JG2AghOlwbMxrZtuSNyB000Bz31w1dC51vGkUA/640?wx_fmt=png&from=appmsg)

在我们的 trace log 中 进入了 case2 分支，就先分析这个分支。

> 这里要特别注意一下，因为这里是根据随机数来进入不同的分支的。所以在测试的时候不是每次都进入到这个 8cc8 分支。多调用几次，总会有一个进入的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5f83NnppCbmlk0YSibfbGQKskMmIuicjfYKQyKPCLjicID8NroXicxmuH4tA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fOWtNSZVk4eVFQmYwA349b8ib7jZDvibpqicEKMV34Ko86ue6z5AhJJjWw/640?wx_fmt=png&from=appmsg)

通过代码看到各参数都进入到 1ADCC 这个函数，优先分析这个函数。hook 查看参数

```
hook_native(Module.findBaseAddress("libjdbitmapkit.so").add(0x1ADCC), 0x5);


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fEsLfEAopNnyjQvlk98jkt4QHxibLuMGEyUqnl6sKVnRS9eVQHteiaBWw/640?wx_fmt=png&from=appmsg)

修改一下 hook 代码把第二参数输出一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fricYg9cuGzhxFeSYS8CltofQq8WibFB43rcBAibuKJHZhlHRUoqYwC7Lg/640?wx_fmt=png&from=appmsg)

第二参数是字符串，第四个参数是传入的数据，第五个参数是传入数据的长度。所以可以让 hexdump 根据这个参数 dump 出完整的数据。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fCpeAiac2Sziaf4nzesMHm38cXtNoQrIV4UhiblkvIWcib69xUvMiakJicuCw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5foKSgPBvpcepzXyQRZ6HeYUCBdPcTAZp3No7sCohnMdMNlSArIJSBEw/640?wx_fmt=png&from=appmsg)

分析之后 1ADCC 就是最终的算法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fY2jQ3SkiaiaBorXQMg55KzHo85jwjtYVWgInqtzuVGJX1egRy860otkQ/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fKsvWCnuNn9z9kgVHQaicHqxf7icdr1PEvplnzJ587dQiasDVAW7ubT6yQ/640?wx_fmt=png&from=appmsg)

下面是还原之后的算法

```
#include <stdio.h>
void alg2(char* rawdata, int input_len){
    int v4,v10;
    char v12 ;
// 分支 2 的算法
    char* key = "80306f4370b39fd5630ad0529f77adb6";
    unsigned char table[0x10] = {0x37, 0x92, 0x44, 0x68, 0xA5, 0x3D, 0xCC, 0x7F, 0xBB,0xF, 0xD9, 0x88, 0xEE, 0x9A, 0xE9, 0x5A};
    for (int i = 0; i != input_len; ++i) {
        v4 = i&7;
        // v10 = byte_FA0[v9 & 0xF];
        v10 = table[i& 0xF];
        // v12 = ((v10 ^ *(_BYTE *)(rawdata + v9) ^ *(_BYTE *)(randondata + (v9 & 7))) + v11) ^ v10;
        v12 = ((v10 ^ *(unsigned char *)(rawdata + i) ^ *(unsigned char *)(key + (i & 7))) + v10) ^ v10;

        // *(_BYTE *)(rawdata + v9) = v12;
        *(unsigned char  *)(rawdata + i) = v12;
    //   *(_BYTE *)(rawdata + v9) = v12 ^ *(_BYTE *)(randondata + (v9 & 7));
      *(unsigned char *)(rawdata + i) = v12 ^ *(unsigned char *)(key + (i & 7));
    }
}

int main(int argc, char const *argv[])
{
    char input [] ="functionId=search&body={\"addrFilter\":\"1\",\"addressId\":\"0\",\"articleEssay\":\"1\",\"attrRet\":\"0\",\"buriedExpLabel\":\"\",\"deviceidTail\":\"38\",\"exposedCount\":\"0\",\"filterServiceIds\":\"1468131091\",\"first_search\":\"1\",\"frontExpids\":\"F_001\",\"gcAreaId\":\"1,72,55674,0\",\"gcLat\":\"39.944093\",\"gcLng\":\"116.482276\",\"imagesize\":{\"gridImg\":\"531x531\",\"listImg\":\"358x358\",\"longImg\":\"531x708\"},\"insertArticle\":\"1\",\"insertScene\":\"1\",\"insertedCount\":\"0\",\"isCorrect\":\"1\",\"jdv\":\"0|kong|t_2018512525_cpv_nopay|tuiguang|17303608941925019140008|1730360893\",\"keyword\":\"ç©ºæ°å æ¹¿å¨\",\"localNum\":\"2\",\"newMiddleTag\":\"1\",\"newVersion\":\"3\",\"oneBoxMod\":\"1\",\"orignalSearch\":\"1\",\"orignalSelect\":\"1\",\"page\":\"1\",\"pageEntrance\":\"1\",\"pagesize\":\"10\",\"populationType\":\"232\",\"pvid\":\"\",\"searchVersionCode\":\"10110\",\"secondInsedCount\":\"0\",\"showShopTab\":\"yes\",\"showStoreTab\":\"1\",\"show_posnum\":\"0\",\"sourceRef\":[{\"action\":\"\",\"eventId\":\"MyJD_WordSizeResult\",\"isDirectSearch\":\"0\",\"logid\":\"\",\"pageId\":\"Home_Main\",\"pvId\":\"\"},{\"action\":\"\",\"eventId\":\"Search_History\",\"isDirectSearch\":\"0\",\"logid\":\"\",\"pageId\":\"Search_Activity\",\"pvId\":\"632ba208e4854bb1839e6e32a5e6b841\"}],\"stock\":\"1\",\"ver\":\"142\"}&uuid=bd132c578e85c7cd&client=android&clientVersion=13.1.0&st=1731550738362&sv=102\"";
    alg2(input,sizeof(input)-1);
    for (int i = 0; i < sizeof(input)-1; i++){
        printf("%02x",(unsigned char)input[i]);
    }
    
    return 0;
}


```

可以看到是一致的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fewWq03w25AgEtccr2njgoyPkWvCFgRNeGu3Zh1ArtC7gFTqs27jtRw/640?wx_fmt=png&from=appmsg)

最后把这段数据 base64 之后再进行 md5 就是最终的 sign 值。最终拼接上 sv 和 t 值返回到 java 层

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8HdGonls53t1vVzOPbheS5fFicQ0MqDJCp99GC9zojeiaA8ce3iciat9MFnGGkn2o68iaNOghfckGXeHhg/640?wx_fmt=png&from=appmsg)

今天就先分析 case2 的分支，以后有机会再分析其他分支啦！

总结
==

以 x-api-eid-token 作为入口点，分析 java 层的最终点在 BitmapkitUtils.getSignFromJni。结合 frida-trace 和 findHash 找到关键点 sub_27A4 。在这个函数内部使用了随机数的方式进入不同的分支。使用 CASE2 分支作为今天的入口点，最终成功得到 sign 具体的算法逻辑。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8HdGonls53t1vVzOPbheS5frQZKeW4h1Y2sEicYDkp7j4NVE0c3uMboSjN3926ia9dOtjx38e3S7WJw/640?wx_fmt=jpeg)

  

看雪 ID：绿豆粥

_https://bbs.kanxue.com/user-home-791353.htm_

* 本文为看雪论坛优秀文章，由 绿豆粥 原创，转载请注明来自看雪社区

# 往期推荐

1、[PWN 入门 - SROP 拜师](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458579476&idx=2&sn=4f9adc1e7d61c7357bdc85ba654f24cb&chksm=b18dc29e86fa4b88c483a581131de043b076918cd7c7436a82e9bb56bc37c8f1edf6c87d8350&scene=21#wechat_redirect)

2、[一种 apc 注入型的 Gamarue 病毒的变种](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458579387&idx=1&sn=9d6fbf25f11b3d99c92c5ac8de0587d5&chksm=b18dc13186fa4827ae7a7bf909e0d2b9490c6df4417c1d7eebc27127133daa9771c212b4f310&scene=21#wechat_redirect)

3、[野蛮 fuzz：提升性能](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458579145&idx=1&sn=9134327916f678cfe7e2bc3371cedeaf&chksm=b18dc04386fa49557abc8c7e6ce3410dd4042ed88635c48961fda72b7fa4425698e56bb86ff6&scene=21#wechat_redirect)

4、[关于安卓注入几种方式的讨论，开源注入模块实现](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458579138&idx=1&sn=fef09513ae9f594e68a503f69a312f4f&chksm=b18dc04886fa495e440990cd2dbddb24693452562e53bd8cb565063ddee921b7e288477f4eea&scene=21#wechat_redirect)

5、[2024 年 KCTF 水泊梁山 - 反混淆](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458579017&idx=2&sn=a97dacde8a6c913108999da8a96a667f&chksm=b18dc0c386fa49d57ce9f0ce6923690d6eb8efb3ccb8032e8c6b923184af3dd29b1b4471f9a2&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8HdGonls53t1vVzOPbheS5fJLlWxshPcLbnryXM6ctubWP9kWLNaWhmLBzP5IqGdN6uyeEE30ydxQ/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8HdGonls53t1vVzOPbheS5fJLlWxshPcLbnryXM6ctubWP9kWLNaWhmLBzP5IqGdN6uyeEE30ydxQ/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8HdGonls53t1vVzOPbheS5fJLlWxshPcLbnryXM6ctubWP9kWLNaWhmLBzP5IqGdN6uyeEE30ydxQ/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8HdGonls53t1vVzOPbheS5fkomm9raOzrnbrC62tV5C5eefkEiaBeOIfQDRgbFfplvHn0AtIfaf15g/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多