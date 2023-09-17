> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7mz6JkjvM_kClceFMa7JCA)

ps：纯纯装逼文，分享记录 ios 学习的逆向思路，不会透露任何密钥

看完您可能会获得以下技能：

1.ios 应用砸壳

2.frida-stalker trace 算法执行流程

3.trace 日志分析思路

4. 白盒 aes 解决思路

![](https://mmbiz.qpic.cn/mmbiz_jpg/icBZkwGO7uH4w4FlSHlccs26JmIHBeW7IFQbMBxuDGuiczturiaus3PPTR4rc2yu7gl5YqUo6v2UXDk8YYpKTjHAg/640?wx_fmt=jpeg)

正在学习 ios 逆向，所以搞了一个轻车熟路的 app 来分析。此次样本为

ZGlhbnBpbmcgMTAuNTIuMQ==

跟安卓一模一样，只是密钥不同

a2 时间戳

a5 还是 zlib+rc4，hook base64 函数看调用栈即可 sub_10379F2DC

```
{"b1":"{\"7\":\"\",\"3\":\"1\",\"4\":\"1\",\"5\":\"\",\"1\":\"1\",\"33\":\"{\\\"10\\\":\\\"-\\\",\\\"2\\\":\\\"\\\",\\\"3\\\":\\\"\\\",\\\"4\\\":\\\"\\\",\\\"5\\\":\\\"-\\\",\\\"6\\\":\\\"-\\\",\\\"7\\\":\\\"-\\\",\\\"0\\\":3,\\\"8\\\":\\\"-\\\",\\\"1\\\":\\\"57\\\",\\\"9\\\":\\\"-\\\"}\",\"6\":\"1\",\"2\":\"\"}","b2":30,"b3":1,"b4":"com.****.*","b5":"10.52.1","b6":10521,"b7":1693707988,"b8":1693707988,"b9":1693707988,"b10":"5.2.15","b11":"5.2.15","b12":2}

```

a7 设备指纹

a8 设备指纹

a9 还是 zlib+aes，hook aes 函数即可 sub_10379EE50（密钥有点小处理）

```
{"0":2,"1":["0","WiFi","-1.020000","[2,2]","Darwin","Apple","unknown","iOS12.5.7","N61AP","zh-Hans-CN","iphone","xianping的 iPhone","Asia/Shanghai (GMT+8) offset 28800","750*1334","[{\"bssid\":\"\",\"ssid\":\"\"}]","18.7.0"],"2":["0","15989469184","{\"7\":\"\",\"3\":\"1\",\"4\":\"1\",\"5\":\"\",\"1\":\"1\",\"33\":\"{\\\"10\\\":\\\"-\\\",\\\"2\\\":\\\"\\\",\\\"3\\\":\\\"\\\",\\\"4\\\":\\\"\\\",\\\"5\\\":\\\"-\\\",\\\"6\\\":\\\"-\\\",\\\"7\\\":\\\"-\\\",\\\"0\\\":3,\\\"8\\\":\\\"-\\\",\\\"1\\\":\\\"57\\\",\\\"9\\\":\\\"-\\\"}\",\"6\":\"1\",\"2\":\"\"}","0.358021","50","com.dianping.dpscope","10.52.1","7228911616","1693707988541","0000000000000AB5739545DF447*******2093F7F07B7A169354057040620159","1037041664","1693707881960.258","unknown","[]","0","0"],"3":"{\"9\":5,\"127\":5,\"253\":5}"}

```

a2 是 hmacsha1 + 白盒 aes + 异或 key

这版本 a5 和 a9 算法还是比较简单（风控难![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/newemoji/Sigh.png)），不说了

重点分析 a2

**一、砸壳**

砸壳使用 frida-ios-dump-master

```
 https://github.com/AloneMonkey/frida-ios-dump

```

安装依赖，已经有 frida 环境的话，记得把 frdia 注释掉，不然给你换了版本

```
pip3 install -r /opt/dump/frida-ios-dump/requirements.txt

```

使用很简单，先去 dump.py 配置一下你的 ssh ip 和端口、账号密码，他默认 2222 端口，需要端口转发，不转发就就直接使用 22 端口  

mac 转发命令如下

```
打开Mac终端，输入iproxy 2222 22把当前连接设备的22端口(SSH端口)，映射到电脑的2222端口。

```

运行命令

```
python3 dump.py app名字

```

然后就是等，win 上脱可能会卡死，如果不幸怎么都脱不下来就找个 mac 的朋友帮忙

然后在当前目录就有个 app 名字的文件夹

进去，显示包内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6WydXoiaQHibdNb95nnGZvQzbWBsfdUY2BYnl9CLC6NWr60qyYib5cCtUw/640?wx_fmt=png)

找到 Unix 可执行文件  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR655hh7IX6Bn2X2luv7OwrVf0TmLGIV5AlbMibs4Zq8liaLrE7Lp48LO6Q/640?wx_fmt=png)

或者终端 file * | grep -i mach

这个扔到 ida，不出所料，慢的一批

在 ida 就跟分析安卓 so 差不多了，出了 oc 语言，但是核心算法还是 c/c++ 的  

oc 不细说，我也不会

二、算法分析

hmacsha1 很容易找到，用 findcrypt3 插件，具体请看上篇文章

```
function myhexdump(name, hexdump_obj, len_obj){
    console.log("-------------------"+ name.toString()+"-------------------\\n");
    console.log(hexdump(hexdump_obj,{
        length:len_obj
    }) );
    console.log("-------------------ENDEND-------------------\\n")
}
function print_c_stack(context, str_tag) {
    console.log('');
    console.log("=============================" + str_tag.toString() + " Stack strat=======================");
    console.log(Thread.backtrace(context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n'));
    console.log("=============================" + str_tag.toString() + " Stack end  =======================");
}
function hook_a2() {
    var func_addr = get_func_addr('DPScope', 0x37CBAB8)
    Interceptor.attach(ptr(func_addr), {
        onEnter: function (args) {
            console.log("onEnter");
            res = args[0];
            console.log("param:" + args[0],args[1],args[2],args[3]);
          myhexdump("hook_a2() arg0->",args[0],parseInt(args[1]))
          myhexdump("hook_a2() arg2->",args[2],parseInt(args[3]))
            this.buffer = args[4];
            print_c_stack(this.context, func_addr);
        },
        onLeave: function (retval) {
            console.log("onLeave");
           myhexdump('hook_a2 sha1hmac()结果:',  this.buffer, 20)
        }
    });
}

```

‍

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6Y7YWo4ibLO3lonlgF5vTg1eeHgKTkT6fSdQbWCJ8QOZ6cDc12lOPGgA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6tKxwEPYd3fwokGdzcjzhLuoR1DVF8b72I47Bqs7gpFCMqpFwzZibpAw/640?wx_fmt=png)

这个函数就是把 hmacsha1 计算结果加密成 a2

hook 一下

```
function a2_two() {
    //a2 hmacsha1后的操作
    var addr=0x383763C;
    var func_addr = get_func_addr('DPScope', addr)
    console.log("func_addr", func_addr);
    Interceptor.attach(ptr(func_addr), {
        onEnter: function (args) {
               myhexdump('test arg0:', args[0], parseInt(args[1]))
            myhexdump('test arg2:', args[2], 32)
            this.buffer = args[2];
        },
        onLeave: function (retval) {
            myhexdump('test ret:', retval, 32)
        }
    });
}

```

没错，那就主动调用

```
function call_aes(){
    var base = Module.getBaseAddress("DPScope");
    var func = new NativeFunction(base.add(0x0383763C),'pointer',['pointer','int','pointer']);
// 00000000: 08 fb 21 11 cc 84 81 9D  79 CC 71 22 8A AA AA DC  ..!.....y.q"....
// 00000010: D0 D7 57 63                                       ..Wc
    var data_len = 20;
    const data = Memory.alloc(data_len);
    Memory.writeByteArray(data,[8, 251, 33, 17, 204, 132, 129, 157, 121, 204, 113, 34, 138, 170, 170, 220, 208, 215, 87, 99]);
    // Memory.writeByteArray(data,[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]);
        //主动调用
    var res = func(data,data_len,data);
    myhexdump("res",res,16)
}

```

不要算法的这里写个 rpc 已经完事了

ida 静态分析还是不好分析，很多花指令，为了分析顺畅还是的 trace 一下  

，之前都是用 unidbg trace 比较多，现在直接用真机 + frida-stalker，

这里用到了 Virenz 大佬写好的 stalker 脚本（https://github.com/Virenz/frida-js）

106 行修改，改成 hexdump 打印，这样就没有乱码了  

```
let moduleBase;
let pre_regs = [];
let infoMap = new Map();
let detailInsMap = new Map();
let regs_map = new Map();
function formatArm64Regs(context) {
    let regs = []
    regs.push(context.x0);
    regs.push(context.x1);
    regs.push(context.x2);
    regs.push(context.x3);
    regs.push(context.x4);
    regs.push(context.x5);
    regs.push(context.x6);
    regs.push(context.x7);
    regs.push(context.x8);
    regs.push(context.x9);
    regs.push(context.x10);
    regs.push(context.x11);
    regs.push(context.x12);
    regs.push(context.x13);
    regs.push(context.x14);
    regs.push(context.x15);
    regs.push(context.x16);
    regs.push(context.x17);
    regs.push(context.x18);
    regs.push(context.x19);
    regs.push(context.x20);
    regs.push(context.x21);
    regs.push(context.x22);
    regs.push(context.x23);
    regs.push(context.x24);
    regs.push(context.x25);
    regs.push(context.x26);
    regs.push(context.x27);
    regs.push(context.x28);
    regs.push(context.fp);
    regs.push(context.lr);
    regs.push(context.sp);
    regs.push(context.pc);
    regs_map.set('x0', context.x0);
    regs_map.set('x1', context.x1);
    regs_map.set('x2', context.x2);
    regs_map.set('x3', context.x3);
    regs_map.set('x4', context.x4);
    regs_map.set('x5', context.x5);
    regs_map.set('x6', context.x6);
    regs_map.set('x7', context.x7);
    regs_map.set('x8', context.x8);
    regs_map.set('x9', context.x9);
    regs_map.set('x10', context.x10);
    regs_map.set('x11', context.x11);
    regs_map.set('x12', context.x12);
    regs_map.set('x13', context.x13);
    regs_map.set('x14', context.x14);
    regs_map.set('x15', context.x15);
    regs_map.set('x16', context.x16);
    regs_map.set('x17', context.x17);
    regs_map.set('x18', context.x18);
    regs_map.set('x19', context.x19);
    regs_map.set('x20', context.x20);
    regs_map.set('x21', context.x21);
    regs_map.set('x22', context.x22);
    regs_map.set('x23', context.x23);
    regs_map.set('x24', context.x24);
    regs_map.set('x25', context.x25);
    regs_map.set('x26', context.x26);
    regs_map.set('x27', context.x27);
    regs_map.set('x28', context.x28);
    regs_map.set('fp', context.fp);
    regs_map.set('lr', context.lr);
    regs_map.set('sp', context.sp);
    regs_map.set('pc', context.pc);
    return regs;
}
function getRegsString(index) {
    let reg;
    if (index === 31) {
        reg = "sp"
    } else {
        reg = "x" + index;
    }
    return reg;
}
function isRegsChange(context, ins) {
    let currentRegs = formatArm64Regs(context);
    let entity = {};
    let logInfo = "";
    // 打印寄存器信息
    for (let i = 0; i < 32; i++) {
        if (i === 30) {
            continue
        }
        let preReg = pre_regs[i] ? pre_regs[i] : 0x0;
        let currentReg = currentRegs[i];
        if (Number(preReg) !== Number(currentReg)) {
            if (logInfo === "") {
                //尝试读取string
                let changeString = "";
                try {
                    let nativePointer = new NativePointer(currentReg);
                    changeString = nativePointer.readCString();
                    changeString = Memory.readByteArray(nativePointer, 64)
                    // myhexdump("changeString",nativePointer,64)
                    console.log(changeString)
                } catch (e) {
                    changeString = "";
                }
                if (changeString !== "") {
                    currentReg = currentReg + " (" + changeString + ")";
                }
                logInfo = " " + getRegsString(i) + ": " + preReg + " --> " + currentReg + ", ";
            } else {
                logInfo = logInfo + " " + getRegsString(i) + ": " + preReg + " --> " + currentReg + ", ";
            }
        }
    }
    entity.info = logInfo;
    pre_regs = currentRegs;
    return entity;
}
function myhexdump(name, hexdump_obj, len_obj){
    console.log("-------------------"+ name.toString()+"-------------------\\n");
    console.log(hexdump(hexdump_obj,{
        length:len_obj
    }) );
    console.log("-------------------ENDEND-------------------\\n")
}
function stalkerTraceRange(tid, base, size, offsetAddr) {
    Stalker.follow(tid, {
        transform: (iterator) => {
            const instruction = iterator.next();
            const startAddress = instruction.address;
            const isModuleCode = startAddress.compare(base) >= 0 &&
                startAddress.compare(base.add(size)) < 0;
            do {
                iterator.keep();
                if (isModuleCode) {
                    let lastInfo = '[' + ptr(instruction["address"] - base) + ']' + '\t' + ptr(instruction["address"]) + '\t' + (instruction+';').padEnd(30,' ');
                    let address = instruction.address - base;
                    detailInsMap.set(String(address), JSON.stringify(instruction));
                    infoMap.set(String(address), lastInfo);
                    iterator.putCallout((context) => {
                        let offset = Number(context.pc) - base;
                        let detailIns = detailInsMap.get(String(offset));
                        let insinfo = infoMap.get(String(offset));
                        let entity = isRegsChange(context, detailIns);
                        let info = insinfo + '\t#' + entity.info;
                        let next_pc = context.pc.add(4);
                        let insn_next = Instruction.parse(next_pc);
                        insinfo = '[' + ptr(insn_next["address"] - base) + ']' + '\t' + ptr(insn_next["address"]) + '\t' + (insn_next + ';').padEnd(30,' ');
                        let mnemonic = insn_next.mnemonic;
                        if (mnemonic.startsWith("b.") || mnemonic === "b" || mnemonic === "bl" || mnemonic === "br" ||  mnemonic === "bx" || mnemonic.startsWith("bl") || mnemonic.startsWith("bx")) {
                            info = info + '\n' + insinfo + '\t#';
                        }
                        console.log(info);
                    });
                }
            } while (iterator.next() !== null);
        }
    })
}
function traceAddr(addr,base_addr) {
    let moduleMap = new ModuleMap();
    let targetModule = moduleMap.find(addr);
    console.log('-----start trace：', addr, '------');
    moduleBase = base_addr;
    Interceptor.attach(addr, {
        onEnter: function(args) {
            this.tid = Process.getCurrentThreadId()
            stalkerTraceRange(this.tid,targetModule.base,targetModule.size,addr);
        },
        onLeave: function(ret) {
            Stalker.unfollow(this.tid);
            Stalker.garbageCollect();
            console.log('ret: ' + ret);
            console.log('-----end trace------');
        }
    });
}
// ---------------------------------------------------------------------------------------
// traceNativeFunction
// ---------------------------------------------------------------------------------------
// 打印调用堆栈
function traceFunction(addr, base_addr){
    let moduleMap = new ModuleMap();
    let base_size = moduleMap.find(addr).size;
console.log(base_size)
    Interceptor.attach(addr, {
        onEnter: function(args) {
            this.tid = Process.getCurrentThreadId();
            Stalker.follow(this.tid, {
                events: {
                    call: true
                },
                onReceive: function(events) {
                    let allEvents = Stalker.parse(events);
                    let first_depth = 0;
                    let is_first = true;
                    for (let i = 0; i < allEvents.length; i++) {
                        // 调用的流程, location是哪里发生的调用, target是调用到了哪里
                        if (allEvents[i][0] === "call") {
                            let location = allEvents[i][1]; // 调用地址
                            let target = allEvents[i][2];   // 目标地址
                            let depth = allEvents[i][3];    // depth
                            let description = '';
                            let space_num = '';
                            if (target.compare(base_addr) >= 0 && target.compare(base_addr.add(base_size)) < 0) {
                                if (is_first) {
                                    is_first = false;
                                    first_depth = depth;
                                }
                                let location_description = ' [' + ptr(location).sub(base_addr) + '] ';
                                let target_description = ' [' + ptr(target).sub(base_addr) + ']';
                                let length = (depth - first_depth);
                                for (let j = 0; j < length; j++) {
                                    space_num = space_num + ' ';
                                }
                                description = space_num + target_description + '(' + location_description + ')' + ' -- ' + length;
                                console.log(description); 
                            } 
                        }
                    }
                }
            })
        }, onLeave: function(retval) {
            Stalker.unfollow(this.tid);
        }
    })
}
function tracecode_a2() {
    moduleBase = Module.getBaseAddress("DPScope");
    var functionAddress = moduleBase.add(0x0383763C);
// traceFunction(functionAddress, functionAddress);
    traceAddr(functionAddress, moduleBase)
}
}

```

frida 加载脚本调用 tracecode_a2(), 还有个小细节，app 本身会一只发送网络请求，那么有些业务请求就会调用 a2 算法，这样会干扰我们的 trace，这时可以选择去一个干净的页面，等待几秒，把网络给关了，

然后 call_aes()

tarce 的速度看手机，这个函数几分钟就完事  

需要这份 trace 日志的后台发送 "trace 日志"，先到先得![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/Expression/Expression_30@2x.png)

结果全局搜索，发现存放到 0x282c85a50 这个地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLKY2LSVbs81lanK4NyzukcXaicjl78wHgGH0uM8vx6vPic3O69RrS8mqQ/640?wx_fmt=png)

全局搜索 0x282c85a50

一个个看

发现是这里出现了第一个字节 0x48，ldr 加载 x20+x8 上的数据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLQs44g0mL3p8qN3HoeJFz0fLB5haoKgE3gfmDyf7zVOe0ctHFTjzNqQ/640?wx_fmt=png)

往上看看 0x48 是在哪里 str(写入)0x282c85a50 这个地址的  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLFcSbAWKctFQMiccxKOtpyJjc3YBDpmIp4Pa611O2Sohoc3XojtY6BbQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLPKCUiccDYHdLNNURQ50DLFhRlxs4j0nrKdIoWTekTDichJC0sGo7OvNQ/640?wx_fmt=png)

67295 行第一个值还是 0x35，还不是 0x48，说明就这 300 多行汇编代码，务必会把 0x48 写入 0x282c85a50

所以这 300 多行代码复制下来 新建一个文本搜索 0x48，  

只有一个地方，取的是 x9 指针上的值，0x14a22fef8，

这个地址全局搜一下，发现 67295 行之后只有一个

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmL65dQcw21nO9wOO0ibR4wT2sw6jFwlXnib6SAibGSoEGAviaXZt3m0ZHhQg/640?wx_fmt=png)

不对劲，改成搜 ldr w9, [x9];  试试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLl3ma1cUh6ImYlKRuXPuSwoA6Sa1tnnHgiaTtOxIHGJTFgicuiahNOA99w/640?wx_fmt=png)

舒服了，结果全部出来了， 

搜下这个地址：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLI2oD1xfAiboDUM90wQhiaxmHubcQ3Ur73nAibFK9P7kRuSKFl52wZaFAw/640?wx_fmt=png)

从 6w 多行开始看，

x11 写到 [x29, #-0x78], 按照尿性后面都会从这里取出来，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLHQzN8A4MLAXKJiaqQqGnw3maDvgTPdB0dXqjtYhI14hXgyI07NdTe3A/640?wx_fmt=png)

67159 行 ldr 给 x9 了  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLBKicyAoo5OkHVBsWvRW6uBmBt99ib5zGqkoR9FaTRhAL9LG6BH5vpUsQ/640?wx_fmt=png)

看图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmLF5pPNT3cSf5aAEibjxVZibPQiaAwg63W4a6QhXy4HUtN9UiaqfpzNMZGaw/640?wx_fmt=png)

搜一下 eor x8, x8, x22;  
看是不是都走这个逻辑

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH7eT0skLmcuudAOptf7UFmL2sqPZdY8CaEqaSQaibhcwfEkFcf7rvSic3VxfHHDLkenS81SdhKbnv8A/640?wx_fmt=png)

有些值没打印出来，应该 trace 有点 bug ，不过问题不大，应该都是一样的，先还原再说

接下来找 eor 的两个值, 根据异或可逆特性，一般是把一个动态计算出来的参数异或一个写死的 key，用来加强安全性，那么这里的 x8 和 x22，哪个是固定的，哪个是动态的？  

修改主动调用参数值，再 trace 一遍即可, 第一个 8 改成 88  

```
function call_a2(){
    var base = Module.getBaseAddress("DPScope");
    // console.log("base.add(0x383763C): ",base.add(0x0383763C));
    var func = new NativeFunction(base.add(0x0383763C),'pointer',['pointer','int','pointer']);
// 00000000: 08 FB 21 11 CC 84 81 9D  79 CC 71 22 8A AA AA DC  ..!.....y.q"....
// 00000010: D0 D7 57 63                                       ..Wc
    var data_len = 20;
    const data = Memory.alloc(data_len);
    Memory.writeByteArray(data,[88, 251, 33, 17, 204, 132, 129, 157, 121, 204, 113, 34, 138, 170, 170, 220, 208, 215, 87, 99]);
    // Memory.writeByteArray(data,[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]);
        //主动调用
    var res = func(data,data_len,data);
    myhexdump("res",res,16)
    // var aes_result = Memory.readByteArray(aes_r, data_len);
    //正确结果48 2f 5f b2 3d f8 56 b9 da d5 6c 51 f1 2d c0 f4
    console.log(res);(aes_r);
}
haaa

```

再搜一下 eor x8, x8, x22;  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkJVhGytibKRiaNfwQAtCkD3ictBkcyqaaFmCHIAoq065tMEx8hFaOVjjqw/640?wx_fmt=png)

显而易见，左边的 x8 两次的值都是一样的，有些没打印出来的不要慌，x8，往上追几行就能看到具体值，得到一个密钥

```
salt_key = [0x7d, 0xb9, 0x3c, 0x36, 0x0b, 0x34, 0x74, 0x25, 0x2a, 0x8c, 0xbf, 0x36, 0xc9, 0x9c, 0xa8, 0x84]

```

那么再继续看 x22 怎么来的

x22 是 x19 的值  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkm3ExwvAyYwf1AzRqFiaIMGLRmcALuDXx92BvoRic9fU1T8RlBnEicFbnA/640?wx_fmt=png)

再往上  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkXlSeresAzq6GULdKVkhFg8t0KichInfulbo23gnobiaYTcDFYq5Nw7lA/640?wx_fmt=png)

学个汇编先

madd x19, x9, x15, x20;    x19=x9*15+x20

0x14a22fec8 这时候可以看到 0x35 已经被写进去了，跟上边的思路一样，去找哪个地方把 0x35 写进去的  

往上，注意看，在这地方，0x14a22fec8 第一个字节还是 ox00，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkCl3fJLITRVOvEck3lXqn3GaAIc0a5icgNcnjBHJeJZ17tYCfgibl8Sqg/640?wx_fmt=png)

这时候就不要往上追了，应该往下找 str 了，

因为 x9 已经等于 0x14a22fec8，所以要找 x9 的 str 操作，往下几行就看到了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqk1NHwm6CcqF16Jb1xVgatfROA6QkgUQryM5NiaY8dd3UHkKPt82HRXeQ/640?wx_fmt=png)

很明显了，

ldr x8, [x10]; ：从把 x10 指针指向的值加载到 x8

ldrsb w8, [x8]; ：w8 是 0x282c85a50 指针的值的第一个字节

str w8, [x9]; ：把 w8 写到 x9(0x14a22fec8),

其实就是依次把 35 96 63 84 36 cc 22 9c f0 59 d3 67 38 b1 68 71 和固定的 key 异或

不信你搜 ldrsb w8, [x8]; 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkgXiarp6kjBfrLTiby9r7WHkoBWU0T8p5cjj1TkOPYGkyYAttSQKKSgfw/640?wx_fmt=png)

然后搜 ldrsb x22, [x19];

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH54nj57icX5wics4piaIXZYVqkRzN1pYCMldGO0Mqac7chGZWEtvKn6mISNKQj12n5e0GM8z2pSNiaQqg/640?wx_fmt=png)

能看出来是依次取 35 96 63 84 36 cc 22 9c f0 59 d3 67 38 b1 68 71  

接下来就找 35 96 63 84 36 cc 22 9c f0 59 d3 67 38 b1 68 71 是怎么来的

看 0x282c85a50 地址是谁写进去的

搜索

看 66400 行，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH76ZrQ8iaQPgyw0M5upApyUw9lmqiajQfwibpaylFJtqicAvkUvzNhCQ73TL0O9GpybkiabiajgbpBjrQlg/640?wx_fmt=png)

开始写进去了  

ldrb w9, [x20, x8];  做一看 x20  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH76ZrQ8iaQPgyw0M5upApyUw213AsVLVbI45zibl8TnP2ZJcjxx0PCSbziawVFEiaSj7D6H2YzpN2fvfw/640?wx_fmt=png)

0xf7^0x86=0x71, 就应该是 35 96 63 84 36 cc 22 9c f0 59 d3 67 38 b1 68 71 最后一个字节吧？

搜 eor w14, w14, w15;  看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH76ZrQ8iaQPgyw0M5upApyUw5JvaQ7ydPQKkicHibGFgg76wdJ7ZkwE8TyyUHML95WCjokuI4se7ZZJw/640?wx_fmt=png)

没错，看 65806

因为搞过安卓的，知道这一部分应该就是白盒 aes 计算出来的结果了，这时候就配合 ida，有 bl 指令的地方，都去看看

这里有 9 轮

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH76ZrQ8iaQPgyw0M5upApyUwky9KtIfAian3dbOVgV00qUKIeXpCznLaVFFAWJ0D68vAdOfxyqsicfbQ/640?wx_fmt=png)

0x3836f4c 地址  p 一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH76ZrQ8iaQPgyw0M5upApyUwTg4SDeibua19AwWa1v3KssVECsB8FY5IPS6Tdegv76bmD7lT2eWzPxA/640?wx_fmt=png)

搞过安卓的就会发现很眼熟啊，这是 aes 的列混淆，不过是白盒 aes，

三、白盒 aes

遇到白盒 aes，常见有三种解决方法：

1。黑盒调用，比如 frida rcp，xposed rpc，unidbg、Unicorn（ios 笔者目前只会 frida![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)）

2.dfa 差分攻击，简单说就是在倒数第一和第二轮列混合之间修改此时中间密文的一个字节，会导致最终密文和正确密文有 4 个字节的不同。通过多次的修改，得到多组错误的密文，然后通过正确密文和这些错误密文能够推算出第 10 轮的密钥，继而能推算出原始密钥。

3. 扣代码，大力出奇迹，伪代码能看就直接扣，不能看就 trace 一遍，拿到汇编流程，比较掉头发

笔者 21 年分析 mt 的时候，不知道是 aes 白盒方案，所以是用 ida 静态和 unidbg 动态调试辅助把伪代码翻译成了 python 代码，22 年分析一个采用白盒方案的 app，扣代码时发现算法逻辑非常像，所以猜测到也是白盒 aes，然后利用 python 代码 + dfa 攻击把密钥弄出来了  

然后我通过对比 mt 和 dp 发现两者只有一块地址的值不同，可以理解为白盒 aes 的 table，然后这一块地址其实是读取 app 静态文件经过计算得来的  

安卓 mt 是 ms_com.dianping.v1.png

安卓 dp 是 ppd_com.dianping.v1.xbt

ios dp 是 ppd_com.dianping.dpscope.xbt

具体分析流程不再细说～ （可以 hook zlib 标准压缩函数![](https://res.wx.qq.com/t/wx_fed/we-emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)）  

```
with open(r"ppd_com.dianping.v1.xbt", "rb") as f:
    xbt = f.read()
    hexdump(xbt)
00000000: 2E 58 42 54 01 00 00 00  B8 A6 02 00 00 E0 02 00  .XBT............
00000010: BD 18 00 A6 62 81 8F 91  00 00 00 00 78 9C 5C 9A  ....b.......x.\.
00000020: 53 73 20 8C B6 44 63 DB  B6 39 B1 9D 89 6D DB B6  Ss ..Dc..9...m..

```

可以看到有 78 9C 这是 zlib 压缩特征

解压一下然后跟一个整数抑或一下就是了，不同的 app 是不同的

```
dt = zlib.decompress(xbt[0x1c:])
dt_list = []
for i in dt:
    i = i ^ int_key
    dt_list.append(i)

```

然后白盒运算过程中用到两个参数是 dt_list 里面的  

```
table_1=dt_list[40960:]
table_2= dt_list[36864:40960]

```

拿到了这俩参数，笔者就可以进行 dfa 攻击了

dfa 之前还得先验证一下，跟 rpc 的结果一不一样

```
def get_a2_(params):
    salt_key = [0x7d, 0xb9, 0x3c, 0x36, 0x0b, 0x34, 0x74, 0x25, 0x2a, 0x8c, 0xbf, 0x36, 0xc9, 0x9c, 0xa8, 0x84]
    sign_two = []
    key = get_onesalt(app_key.encode(), m["a0"].encode())
    hmac_sha1_str = hmacsha1(a2b_hex(key), params.encode())
    //sign_one = a2b_hex(hmac_sha1_str)
    //这个是call_aes的参数 [8, 251, 33, 17, 204, 132, 129, 157, 121, 204, 113, 34, 138, 170, 170, 220, 208, 215, 87, 99]
    sign_one = bytes.fromhex("08 fb 21 11 cc 84 81 9d 79 cc 71 22 8a aa aa dc d0 d7 57 63")
    for i in range(0, len(sign_one), 16):
        sign = list(sign_one[i:i + 16])
        r0 = encrypt(sign)
        sign_two.extend(r0)
        break
    # #dfa攻击
    # print(b2a_hex(bytes(sign_two)).decode())
    a2 = get_sign(salt_key, sign_two)
    return a2

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6s2bXratdp32voKSCGz29RkjhEm1SMHm3YSB54JPlCiak1FLNbHa6pSw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6xCUlzicduXibueIaVHMvkbYjfiaswvUt2Hj7HutazP44mG4a221VicrQlQ/640?wx_fmt=png)

没问题

开始 dfa 攻击，打开注释，先打印一次正确的值

3596638436cc229cf059d36738b16871

然后，在 for 循环执行第 9 轮时，随机修改 state 的一个字节

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6uic8s9Rgz1PiaZT0FfjFo2LcAibcM5coQAmbqQPdibibOC8y9G6ZEEZ62ow/640?wx_fmt=png)

循环执行 50 次

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6lzAiaJv7RicaWK5PRosBFk03JEVrj57pmo0RfSepial03FRs3gFDgLHHA/640?wx_fmt=png)

可以看出每个错误密文和正确密文只有 4 个字节的不同，说明修改时机正确

如果全部不同，说明时机太早，只有一个不同说明时机太晚

接下来就是用 phoenixAES 根据错误密文还原密钥

```
https://github.com/SideChannelMarvels/JeanGrey/tree/master/phoenixAES

```

python3 安装方式：

```
pip3 install phoenixAES

```

第一行为正确密文，其他行为错误密文。

```
import phoenixAES
data = """3596638436cc229cf059d36738b16871
359663dc36cc939cf0f4d36742b16871
bb96638436cc223bf0596e6738c56871
ba96638436cc22d3f059e56738226871
359669843699229c1e59d36738b16832
35da6384f4cc229cf059d36438b11971
7b96638436cc2288f059a36738d86871
3596608436a0229c8159d36738b16875
35965784365c229cdd59d36738b1689a
359668843693229cf659d36738b1681b
4896638436cc224ff059f467382e6871
"""
with open('crackfile', 'w') as fp:
    fp.write(data)
phoenixAES.crack_file('crackfile', [], True, False, verbose=2)

```

    运行，正确输出是这样的

```
Round key bytes recovered:
6F29**********************BF8F45
Last round key #N found:
6F29**********************FBF8F45

```

Last round key 就是第 10 轮的密钥，然后使用 Stark 还原初始密钥

```
https://github.com/SideChannelMarvels/Stark

```

使用方法：编译的二进制文件为 MyStrackNew，10 代表是 aes128

k00 就是 aes 的密钥了

```
D:\>MyStrackNew 6F29**********************FBF8F45 10
K00: 5973**********************E4A43
K01: C7A5**********************D36722
K02: A320**********************A0BA8C
K03: 47D4**********************C94300
K04: 92CE**********************7429E2
K05: 106B**********************76FFEA
K06: 087D**********************B63DF1
K07: 065A**********************EEE239
K08: AEC2**********************EC3BF4
K09: 7B20**********************94403E
K10: 6F29**********************BF8F45

```

当然这上面只用了 10 个错误秘文，是出不来密钥的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR69wYOYR6ZOJCVuFb3WNH0Yk9M3u9pVcl8MYOJ6rHk6YrBmQfLMdM6EA/640?wx_fmt=png)

感兴趣的朋友可以用 frida 动态取 patch 进行 dfa 攻击，合适的位置大概在 0x103837358

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6tRAmrj8mIyUa1QULTBathBJYY3zd9OydRbwXJ77A1Ddxwy4X1NWjSg/640?wx_fmt=png)

拿到密钥来验证一下

```
from Crypto.Cipher import AES
key = bytes.fromhex("59733935434B634E71536D59646E4A43")
cipher = AES.new(key=key, mode=AES.MODE_ECB)
enc_data = bytes.fromhex("08 fb 21 11 cc 84 81 9d 79 cc 71 22 8a aa aa dc")
res = cipher.encrypt(enc_data)
print(res.hex())

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/icBZkwGO7uH5MvYvqc3Fe9DtpDY7BnYR6j39fA6RxOoYoiaXtNop3Ree5ZibfMraEeJQ9UQyAOyVs3xKRX8L5IF7Q/640?wx_fmt=png)