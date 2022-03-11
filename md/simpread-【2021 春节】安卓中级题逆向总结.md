> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1378761-1-1.html)

**作者****论****坛账号：Light 紫星**

【2021 春节】解题领红包之三  
逆向总结本题所用到的工具：  
Jadx1.2.0  
FrIDA14.2.13  
ida7.5.0  
Python3.7.2  
首先安装 apk，打开后是这个界面：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZJTibibw28ic2DrbYUibtwtCfCMdGibliaouQ0q3db7Vl8xADubEq89ZpTlvbCSGLOsRZhg17te2y3yauicA/640?wx_fmt=png)  
随便输入 flag，提示 flag 格式错误，请重试。拖入 jadx，找到关键函数，如下：

```
package p004cn.pojie52.cm01;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
/* renamed from: cn.pojie52.cm01.MainActivity */
public class MainActivity extends AppCompatActivity {
    public native boolean check(String str);
    static {
        System.loadLibrary("native-lib");
    }
    /* access modifiers changed from: protected */
    @Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, androidx.appcompat.app.AppCompatActivity, androidx.fragment.app.FragmentActivity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(C0266R.layout.activity_main);
        final EditText editText = (EditText) findViewById(C0266R.C0268id.flag);
        findViewById(C0266R.C0268id.check).setOnClickListener(new View.OnClickListener() {
            /* class p004cn.pojie52.cm01.MainActivity.View$OnClickListenerC02651 */
            public void onClick(View view) {
                String trim = editText.getText().toString().trim();
                if (trim.length() != 30) {
                    Toast.makeText(MainActivity.this, "flag格式错误，请重试", 0).show();
                } else if (MainActivity.this.check(trim)) {
                    Toast.makeText(MainActivity.this, "恭喜你，验证正确！", 0).show();
                } else {
                    Toast.makeText(MainActivity.this, "flag错误，再接再厉", 0).show();
                }
            }
        });
    }
}

```

  
这里判断了输入的长度是否为 30 位，然后进入了 so 验证。  
下一步，把 so 拖入 ida，直接定位到关键函数：Java_cn_pojie52_cm01_MainActivity_check  
该函数内容如下：  

```
__int64 __fastcall Java_cn_pojie52_cm01_MainActivity_check(_JNIEnv *a1, __int64 a2, __int64 a3)
{
  const char *v5; // x21
  size_t v6; // w0
  int v7; // w0
  __int64 v8; // x0
  _BYTE *v9; // x0
  int8x16_t v10; // q0
  int8x16_t v11; // q4
  int8x16_t v12; // q2
  int8x16_t v13; // q5
  int8x16_t v14; // q1
  int8x16_t v15; // q0
  __int64 v16; // x8
  unsigned int v17; // w19
  _BYTE v19[33]; // [xsp+0h] [xbp-A0h]
  int v20; // [xsp+21h] [xbp-7Fh]
  char v21; // [xsp+25h] [xbp-7Bh]
  char v22; // [xsp+26h] [xbp-7Ah]
  char v23; // [xsp+27h] [xbp-79h]
  char v24; // [xsp+28h] [xbp-78h]
  char dest[16]; // [xsp+38h] [xbp-68h] BYREF
  __int128 v26; // [xsp+48h] [xbp-58h]
  __int128 v27; // [xsp+58h] [xbp-48h]
  __int128 v28; // [xsp+68h] [xbp-38h]
  __int64 v29; // [xsp+78h] [xbp-28h]
  v29 = *(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  if ( a1->functions->GetStringUTFLength(a1, a3) != 30 )
    return 0;
  v5 = a1->functions->GetStringUTFChars(a1, a3, 0LL);
  v28 = 0u;
  v27 = 0u;
  v26 = 0u;
  *dest = 0u;
  v6 = strlen(v5);
  strncpy(dest, v5, v6);
  a1->functions->ReleaseStringUTFChars(a1, a3, v5);
  v7 = strlen(dest);
  sub_B90(dest, v7, "areyousure??????");
  v8 = strlen(dest);
  v9 = sub_D90(dest, v8);
  *v19 = unk_11A1;
  *&v19[16] = unk_11B1;
  *&v19[25] = unk_11BA;
  v10.n128_u64[0] = 0xB2B2B2B2B2B2B2B2LL;
  v10.n128_u64[1] = 0xB2B2B2B2B2B2B2B2LL;
  v11.n128_u64[0] = 0xFEFEFEFEFEFEFEFELL;
  v11.n128_u64[1] = 0xFEFEFEFEFEFEFEFELL;
  v19[0] = 53;
  v12 = veorq_s8(vaddq_s8(veorq_s8(vaddq_s8(*&v19[1], v10), xmmword_1130), xmmword_1140), v11);
  v13.n128_u64[0] = 0x101010101010101LL;
  v13.n128_u64[1] = 0x101010101010101LL;
  v14.n128_u64[0] = 0x3E3E3E3E3E3E3E3ELL;
  v14.n128_u64[1] = 0x3E3E3E3E3E3E3E3ELL;
  *&v19[1] = vaddq_s8(
               veorq_s8(vsubq_s8(v13, vorrq_s8(vshrq_n_u8(v12, 7uLL), vshlq_n_s8(v12, 1uLL))), xmmword_1150),
               v14);
  v20 = 1782990162;
  v15 = veorq_s8(vaddq_s8(veorq_s8(vaddq_s8(*&v19[17], v10), xmmword_1160), xmmword_1170), v11);
  v21 = ((1
        - ((2 * ((((unk_11C6 - 78) ^ 0xB2) - 117) ^ 0xFE)) | ((((((unk_11C6 - 78) ^ 0xB2) - 117) ^ 0xFE) & 0x80) != 0))) ^ 0x25)
      + 62;
  v16 = 0LL;
  v22 = ((1
        - ((2 * ((((unk_11C7 - 78) ^ 0xB1) - 118) ^ 0xFE)) | ((((((unk_11C7 - 78) ^ 0xB1) - 118) ^ 0xFE) & 0x80) != 0))) ^ 0x26)
      + 62;
  *&v19[17] = vaddq_s8(
                veorq_s8(vsubq_s8(v13, vorrq_s8(vshrq_n_u8(v15, 7uLL), vshlq_n_s8(v15, 1uLL))), xmmword_1180),
                v14);
  v23 = ((1
        - ((2 * ((((unk_11C8 - 78) ^ 0xB0) - 119) ^ 0xFE)) | ((((((unk_11C8 - 78) ^ 0xB0) - 119) ^ 0xFE) & 0x80) != 0))) ^ 0x27)
      + 62;
  v24 = ((1
        - ((2 * ((((unk_11C9 - 78) ^ 0xBF) - 120) ^ 0xFE)) | ((((((unk_11C9 - 78) ^ 0xBF) - 120) ^ 0xFE) & 0x80) != 0))) ^ 0x28)
      + 62;
  while ( v9[v16] == v19[v16] )
  {
    if ( v9[v16] )
    {
      if ( ++v16 != 41 )
        continue;
    }
    v17 = 1;
    goto LABEL_9;
  }
  v17 = 0;
LABEL_9:
  free(v9);
  return v17;
}

```

  
大概看一下流程，先判断输入是否 30 位，然后把输入的数据传入 sub_B90 进行处理，再传入 sub_D90 处理一次，处理后的结果为 v9，最后 v9 和 v19 进行比较。所以这里要先看一下 sub_b90 和 sub_d90 是干什么的，直接使用 frida 进行 hook 调用这两个函数，  
frida 代码如下：  

```
# -*- coding: UTF-8 -*-
import frida, sys
jscode = ''' 
function inline_hook() {
    var so_addr = Module.findBaseAddress("libnative-lib.so");
    if (so_addr) {
        console.log("so_addr:", so_addr);
        var addr_b90 = so_addr.add(0xb90);
        var sub_b90 = new NativeFunction(addr_b90 , 'int', ['pointer', 'int','pointer']);
        var arg1 = Memory.allocUtf8String('111111111111111111111111111111');
        var arg2 = 30;
        var arg3 = Memory.allocUtf8String('areyousure??????');
        var ret_b90 = sub_b90(arg1,arg2,arg3);
        console.log(Memory.readByteArray(arg1,64));
        var addr_d90 = so_addr.add(0xd90);
        var sub_d90 = new NativeFunction(addr_d90 , 'pointer', ['pointer', 'int' ]);
        var arg1 = Memory.allocUtf8String('111111111111111111111111111111');
        var arg2 = 30; 
        var ret_d90 = sub_d90(arg1,arg2);
        console.log(Memory.readByteArray(ret_d90,64));
    }
}
setImmediate(inline_hook)
'''
def on_message(message, data):
    if message['type'] == 'send':
        print(" {0}".format(message['payload']))
    else:
        print(message)
pass
#print(frida.enumerate_devices())
# 查找USB设备并附加到目标进程
device =  frida.get_remote_device()
#pid = device.spawn(["com.live.xctv"])
#session = device.attach(pid)
session =device.attach('cn.pojie52.cm01') #这里是要注入的apk包名
# 在目标进程里创建脚本
script = session.create_script(jscode)
# 注册消息回调
script.on('message', on_message)
print(' Start attach')
# 加载创建好的javascript脚本
script.load()
# 读取系统输入
sys.stdin.read()

```

  
结果如下：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZJTibibw28ic2DrbYUibtwtCfCMWOPR3xJqIq6QSaF3ow8nOTAA6qSIUQKu058IwXqe5Xy5ZRF4KS6voQ/640?wx_fmt=png)  
然后我们先进入 sub_b90 看一下，  
sub_b90 函数内容如下：  

```
unsigned __int64 __fastcall sub_B90(_BYTE *a1, unsigned int a2, char *s)
{
  unsigned __int64 result; // x0
  unsigned __int64 v7; // x8
  signed int v8; // w9
  int v9; // w11
  int v10; // w9
  int v11; // w12
  int v12; // w9
  signed int v13; // w11
  __int64 v14; // x8
  int v15; // w12
  int v16; // w9
  int v17; // w13
  int v18; // w11
  int v19; // w14
  __int128 v20[2]; // [xsp+0h] [xbp-140h]
  __int128 v21[14]; // [xsp+20h] [xbp-120h] BYREF
  __int64 v22; // [xsp+108h] [xbp-38h]
  v22 = *(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  result = strlen(s);
  v20[0] = xmmword_11D0;
  v20[1] = xmmword_11E0;
  qmemcpy(v21, " !\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmno", 80);
  v21[5] = xmmword_1240;
  v21[6] = xmmword_1250;
  v21[7] = xmmword_1260;
  v21[8] = xmmword_1270;
  v21[9] = xmmword_1280;
  v7 = 0LL;
  v8 = 0;
  v21[10] = xmmword_1290;
  v21[11] = xmmword_12A0;
  v21[12] = xmmword_12B0;
  v21[13] = xmmword_12C0;
  do
  {
    v9 = *(v20 + v7);
    v10 = v8 + v9 + s[v7 - v7 / result * result];
    v11 = v10 + 255;
    if ( v10 >= 0 )
      v11 = v10;
    v8 = v10 - (v11 & 0xFFFFFF00);
    *(v20 + v7++) = *(v20 + v8);
    *(v20 + v8) = v9;
  }
  while ( v7 != 256 );
  if ( a2 )
  {
    v12 = 0;
    v13 = 0;
    v14 = a2;
    do
    {
      v15 = v12 + 1;
      if ( v12 + 1 >= 0 )
        v16 = v12 + 1;
      else
        v16 = v12 + 256;
      v12 = v15 - (v16 & 0xFFFFFF00);
      v17 = *(v20 + v12);
      v18 = v13 + v17;
      v19 = v18 + 255;
      if ( v18 >= 0 )
        v19 = v18;
      v13 = v18 - (v19 & 0xFFFFFF00);
      --v14;
      *(v20 + v12) = *(v20 + v13);
      *(v20 + v13) = v17;
      *a1++ ^= *(v20 + (*(v20 + v12) + v17));
    }
    while ( v14 );
  }
  return result;
}

```

  
大概分析一下 sub_b90，是根据传入的第三个参数 s 把 v20 进行了一个初始化，然后再把参数 a1 和 v20 进行了异或运算，主要看这个异或运算，先设想一下，如果是把 a1 进行了异或，那么得到的结果和 a1 之前的数据再异或就可以计算出异或的 key，这里我们把它叫做 xorkey，那么先看一下我们传入的参数，是 30 个 1，也就是 30 个 0x31 ，然后看结果，第一位是 0xe0，0x31^0xe0 = 209，然后把参数改为 30 个 2，即 0x32，得出首位的结果是 0xe3，0xe3^0x32 结果也是 209，证明我们的思路是正确的，然后依次求出所有的 xorkey，  
最后计算出的结果为：  
xorkey = [209, 90, 6, 144, 68, 230, 199, 229, 222, 40, 247, 242, 102, 145, 200, 133, 66, 223, 249, 224, 130, 1, 43, 59, 56, 99, 55, 189, 46, 77]  
接下来看 sub_d90，咋一看返回值，全是字母，看起来有点像 base64，于是用 base64 编码 30 个 1 进行测试，发现结果吻合，于是可以断定 sub_d90 是 base64 函数。  
接下来，就可以写出通过 v9 求输入参数的函数：  

```
import base64
xorkey = [209, 90, 6, 144, 68, 230, 199, 229, 222, 40, 247, 242, 102, 145, 200, 133, 66, 223, 249, 224, 130, 1, 43, 59, 56, 99, 55, 189, 46, 77]
def sub_B90(data,l):
    ret = []
    for i in range(l):
        ret.append(((data[i]))^xorkey[i])
    s=''
    for i in ret:
        s+=chr(i)
    print(s)
    return ret
def resv(data):
    data =base64.b64decode(data)
    t = sub_B90(data,len(data))
    return(t)

```

  
调用 resv 即可计算出输入的参数。  
这个时候我们发现，还有一个 v19 是我们不知道的，如果找到 v19 然后代入 resv 就能求出本题的结果！  
根据 ida 的注释，我们知道 v19 是 xsp+0h，而 dest 是 xsp+38h，而 dest 又作为参数传入了 sub_b90，这里我直接 hooksub_b90，得到 xsp，然后再在 v19 初始化结束之后输出 xsp 的值，即可得到 v19，  
这里的 hook 代码如下：  

```
# -*- coding: UTF-8 -*-
import frida, sys
jscode = ''' 
var destAddr = '';  //定位xsp地址
function inline_hook() {
    var so_addr = Module.findBaseAddress("libnative-lib.so");
    if (so_addr) {
        console.log("so_addr:", so_addr);
        var addr_b90 = so_addr.add(0xB90);
        var sub_b90 = new NativeFunction(addr_b90 , 'int', ['pointer', 'int', 'pointer']);
        Interceptor.attach(sub_b90, { 
            onEnter: function(args) 
            {  
            destAddr = args[0];
            console.log('onEnter B90'); 
            },
            //在hook函数之后执行的语句
            onLeave:function(retval)
            { 
            console.log('onLeave B90');
            } 
        }); 
        var addr_b2c = so_addr.add(0xb2c);
        console.log("The addr_b2c:", addr_b2c);
        Java.perform(function() {
            Interceptor.attach(addr_b2c, {
                onEnter: function(args) { 
                console.log("addr_b2c OnEnter :",  Memory.readByteArray(destAddr.sub(0x38),64) );
                }
            })
        })
    } 
}
setImmediate(inline_hook)
'''
def on_message(message, data):
    if message['type'] == 'send':
        print(" {0}".format(message['payload']))
    else:
        print(message)
pass
#print(frida.enumerate_devices())
# 查找USB设备并附加到目标进程
device =  frida.get_remote_device()
#pid = device.spawn(["com.live.xctv"])
#session = device.attach(pid)
session =device.attach('cn.pojie52.cm01') #这里是要注入的apk包名
# 在目标进程里创建脚本
script = session.create_script(jscode)
# 注册消息回调
script.on('message', on_message)
print(' Start attach')
# 加载创建好的javascript脚本
script.load()
# 读取系统输入
sys.stdin.read()

```

  
随便输入一个 30 位的注册码，得到的结果如下：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZJTibibw28ic2DrbYUibtwtCfCMwmRIGMlSuUNuXz1fzZH38qdwOxrUiaB5r49CGIyW12ahz6GpMQC86HA/640?wx_fmt=png)  
看来这个字符串就是我们要的了。把这个字符串代入函数 resv，即可求出本题的 flag：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZJTibibw28ic2DrbYUibtwtCfCMH6iaB0etSWEBG1UJkZZNrm9IuJwBrvpRSNZASpXjibAEP5L7s3E0lHEg/640?wx_fmt=png)  
输入 52pojieHappyChineseNewYear2021 到输入框，点击验证按钮，提示成功！本题分析结束。  

****-- 官方论坛****

www.52pojie.cn

**-- 推荐给朋友**

公众微信号：吾爱破解论坛

或搜微信号：pojie_52