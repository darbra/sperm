> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4qsseRqcOALSpCMYbh6vtw)

纯 ARM64 平台逆向 VMProtect(VMP) 的专项指南

也不知道咋讲这件事，trace 的话就是要熟悉算法，反推算法的明文。

这块只大概讲讲参考下搞搞。

广东是一个 sha256 的 vmp，前边参数很好追，sha1<-sha256（vmp）<-***

sha1 中的 zhaoboy 就是输入明文 = 7a68616f626f79

zhaoboy+9ddcbd818e1eb3a1c9792a70b406eb622f92cccad1454ac612751d7451360006

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHbonvoa8ibWjyCVxbxWvric87Ee5OnDp0z6ogk8BNI3xPfCiczhrWIDG4ZYbnicJRzYnHszQtKnQat2icg/640?wx_fmt=png&from=appmsg)

定位 9ddcbd818e1eb3a1c9792a70b406eb622f92cccad1454ac612751d7451360006 怎么来

[memcpy] [LR]:RX@0x40022c88[libgdwg.so]0x22c88 [arg1]: RW@0x404145c7 [arg2]: RW@0x40420b80 [len]: 32 [memcpy] [sourceBar]  [-99, -36, -67, -127, -114, 30, -77, -95, -55, 121, 42, 112, -76, 6, -21, 98, 47, -110, -52, -54, -47, 69, 74, -58, 18, 117, 29, 116, 81, 54, 0, 6] [memcpy] [sourceHex] 9ddcbd818e1eb3a1c9792a70b406eb622f92cccad1454ac612751d7451360006

tracewrite(0x40420b80)

[20:06:17 931] Memory WRITE at 0x40420b80, data size = 1, data value = 0x9d, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 931] Memory WRITE at 0x40420b81, data size = 1, data value = 0x4c, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 932] Memory WRITE at 0x40420b81, data size = 1, data value = 0xdc, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 932] Memory WRITE at 0x40420b82, data size = 1, data value = 0xc7, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 932] Memory WRITE at 0x40420b82, data size = 1, data value = 0xbd, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 932] Memory WRITE at 0x40420b83, data size = 1, data value = 0x97, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 932] Memory WRITE at 0x40420b83, data size = 1, data value = 0x81, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608 [20:06:17 933] Memory WRITE at 0x40420b84, data size = 1, data value = 0x95, PC=RX@0x40024674[libgdwg.so]0x24674, LR=RX@0x40024608[libgdwg.so]0x24608

隔一行 取一个值，这里看过去是 vmp

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHbonvoa8ibWjyCVxbxWvric87zZZWpCibkBsU7VbsAk63IqGCEeEjexronLFEnTicUuybLfA3qTj6dbHw/640?wx_fmt=png&from=appmsg)

```
0x249f4] [4925c91a] 0x400249f4: "lsr w9, w10, w9" w10=0x71200879 w9=0x7 => w9=0xe24010 
0x249f4] [4925c91a] 0x400249f4: "lsr w9, w10, w9" w10=0x71200879 w9=0x12 => w9=0x1c48
0x249f4] [4925c91a] 0x400249f4: "lsr w9, w10, w9" w10=0x71200879 w9=0x3 => w9=0xe24010f
根据这个来，sha256的明文编排，明文编排的第二个分组，分别右移7、18、3
  rot7 = right_rotate(W[i - 15], 7) 
  rot18 = right_rotate(W[i - 15], 18)
  shr3 = W[i - 15] >> 3 
0xe25c58fed81d57这个是s1的结果 0x6959d44c这个就是第一个明文
[libgdwg.so 0x24704] [4901098b] 0x40024704: "add x9, x10, x9" x10=0xe25c58fed81d57 x9=0x6959d44c => x9=0xe25c596831f1a3


```

0x71200879+0x6831f1a3 这个有点难推，得仔细找，

看看其他明文的编排 这里很快就找到了

```
         # "eor w9, w10, w9" w10=0x1104400c w9=0xe818980 => w9=0x1f85c98c
            # x10=0x14e1883b2ae1883 x9=0x2887293fa8 => x9=0x14e18ab3587272b
            # x10=0x5be0cd19 x9=0x14e18ab3587272b => x9=0x14e18ab9167f444 H+S1+** 
            # "add x9, x10, x9" x10=0x14e18ab9167f444 x9=0x1f85c98c => x9=0x14e18abb0edbdd0
            temp1 = (h + S1 + ch + K[i] + W[i]) & 0xffffffff 
   # 0x24704] [4901098b] 0x40024704: "add x9, x10, x9" x10=0x14e18abb0edbdd0 x9=0x428a2f98 => x9=0x14e18abf377ed68
跟k表相加的地方就是明文 直接找，其实就是数据大小k表第15个
# W[ 0]: 0x6959d44c | W[ 1]: 0x71200879 | W[ 2]: 0x7fc8738a | W[ 3]: 0x557ec398
        # W[ 4]: 0x80000000 | W[ 5]: 0x00000000 | W[ 6]: 0x000000 | W[ 7]: 0x00000000
        # W[ 8]: 0x00000000 | W[ 9]: 0x00000000 | W[10]: 0x00000000 | W[11]: 0x00000000
        # W[12]: 0x00000000 | W[13]: 0x00000000 | W[14]: 0x00000000 | W[15]: 0x00000080



```

6959d44c712008797fc8738a557ec398800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080

明文 =`6959d44c712008797fc8738a557ec398`   

```
59c2c38ef09f068f13638e434ff007870d1cca21480e7ca3e64fb39f9190a421 为sha256的密文


```

这里就是 sha256 的结果了，后边有其他 add 和 eor 算法。最后结果是一致的，不追踪了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/RQicOzqf0IHbonvoa8ibWjyCVxbxWvric87Oju4yhdZ6BI404uFyU49zUOkwRbUibTmLCfM0shvtfTk6zDAcBvH62Q/640?wx_fmt=png&from=appmsg)

如下为 AI 生成，仅供参考。

本指南专注于 ARM64 架构下的 VMP 逆向技术，完全排除 x86 相关内容，提供针对 ARM64 架构的深度逆向方法。

1. ARM64 架构下 VMP 的核心特点
----------------------

### 1.1 ARM64 特有保护机制

• **AArch64 指令集虚拟化**：将 ARM64 指令转换为自定义字节码 • **PAC(指针认证) 滥用**：利用 ARMv8.3 + 的指针认证特性增强保护 • **SVC 调用混淆**：通过系统调用实现控制流转移 • **NEON 指令虚拟化**：对 SIMD 指令的特殊处理

### 1.2 ARM64 VMP 二进制特征

• `.vmpa64`专用段标识 • 使用`x16/x17`寄存器作为虚拟 IP(指令指针) • 频繁的`brk #0x1`指令用于反调试 • 基于`msr/mrs`的系统寄存器操作

2. ARM64 专用工具链
--------------

### 2.1 静态分析工具

• **IDA Pro with ARM64 处理器模块** • **Ghidra 10.3+**：改进的 ARM64 反编译器 • **Binary Ninja**：支持 ARM64 控制流图分析 • **VMArm64Analyzer**：专用插件

### 2.2 动态分析工具

• **Frida-gum**：ARM64 内联 hook 框架 • **Qiling Framework**：ARM64 模拟器 • **ARM64 GDB with GEF**：增强调试功能 • **ADBI**：安卓 ARM64 动态二进制插桩工具

3. ARM64 静态分析技术
---------------

### 3.1 入口点识别

```
# 使用readelf定位.init_array
readelf -d libtarget.so | grep INIT_ARRAY

# 检查JNI_OnLoad变形
objdump -D libtarget.so | grep -A 20 "JNI_OnLoad"


```

### 3.2 IDA Pro 专项分析

```
# IDAPython定位ARM64虚拟入口
def find_vm_entry():
    for seg in Segments():
        if SegName(seg) == ".vmpa64":
            print(f"Found VMP segment at 0x{seg:x}")
            for func in Functions(seg, GetSegmentAttr(seg, SEGATTR_END)):
                if "vm_start" in GetFunctionName(func):
                    return func
    return BADADDR


```

4. ARM64 动态调试方案
---------------

### 4.1 Frida ARM64 专用脚本

```
// 拦截ARM64寄存器访问
Interceptor.attach(Module.findExportByName(null, "mprotect"), {
    onEnter: function(args) {
        console.log(`mprotect(${args[0]}, ${args[1]}, ${args[2]})`);
        console.log("X0-X3:", 
            this.context.x0, this.context.x1, 
            this.context.x2, this.context.x3);
    }
});

// 绕过ARM64反调试
const svc_handler = new NativeFunction(
    Module.findExportByName(null, "svc#0"),
    'void', ['uint32']);
Interceptor.replace(svc_handler, new NativeCallback(function(num) {
    if(num === 0x1) return; // 忽略brk #0x1
    return svc_handler(num);
}, 'void', ['uint32']));


```

### 4.2 Qiling 模拟框架配置

```
from qiling import Qiling
from qiling.const import QL_ARCH, QL_OS

def arm64_vmp_sandbox():
    ql = Qiling(["libtarget.so"], 
                rootfs="/android_rootfs",
                arch=QL_ARCH.ARM64, 
                ostype=QL_OS.LINUX)
    
    # 设置ARM64特定hook
    def syscall_hook(ql, *args):
        print(f"SVC #{ql.reg.x8} called")
    
    ql.set_syscall(0x1337, syscall_hook)
    ql.run()


```

5. ARM64 虚拟机逆向
--------------

### 5.1 虚拟寄存器映射

```
struct ARM64_VRegs {
    uint64_t vX[31];    // 对应X0-X30
    uint64_t vIP;       // 虚拟指令指针(X16)
    uint64_t vSP;       // 虚拟堆栈指针(X17)
    uint32_t vNZCV;     // 虚拟条件标志
    uint64_t vFP[32];   // 虚拟浮点寄存器
};


```

### 5.2 关键 handler 识别表

<table><thead><tr><th><section>字节码</section></th><th><section>ARM64 等效指令</section></th><th><section>Handler 特征</section></th></tr></thead><tbody><tr><td><section>0x1A</section></td><td><section>ADD Xd, Xn, Xm</section></td><td><section>使用 x0-x3 作为临时寄存器</section></td></tr><tr><td><section>0x2B</section></td><td><section>LDR Xd, [Xn]</section></td><td><section>包含内存访问检查</section></td></tr><tr><td><section>0x3C</section></td><td><section>B.cond label</section></td><td><section>修改 vNZCV 标志</section></td></tr><tr><td><section>0x4D</section></td><td><section>SVC #imm</section></td><td><section>调用系统服务</section></td></tr></tbody></table>

6. ARM64 控制流分析
--------------

### 6.1 虚拟跳转解析

```
def analyze_vm_jump(uc, address):
    # 读取ARM64虚拟跳转指令
    opcode = uc.mem_read(address, 4)
    if (opcode[0] & 0xF0) == 0x80:
        # 解析相对跳转偏移
        offset = (opcode[1] << 16) | (opcode[2] << 8) | opcode[3]
        return address + (offset * 4)
    return address + 4


```

### 6.2 基本块划分算法

```
def split_basic_blocks(start, end):
    blocks = []
    current = start
    while current < end:
        size = get_instr_size(current)
        if is_branch_instr(current):
            blocks.append((current, current+size))
            current += size
            if not is_terminator(current-size):
                next_block = get_branch_target(current-size)
                blocks += split_basic_blocks(next_block, end)
            break
        current += size
    return blocks


```

7. ARM64 反反调试技术
---------------

### 7.1 绕过 ptrace 检测

```
# 使用ADBI注入绕过
adb push adbi /data/local/tmp
adb shell chmod +x /data/local/tmp/adbi
adb shell /data/local/tmp/adbi -p `pidof target` -l libbypass.so


```

### 7.2 处理内存校验

```
// Frida脚本绕过内存校验
const mprotect = Module.findExportByName(null, "mprotect");
Interceptor.attach(mprotect, {
    onLeave: function(retval) {
        if(this.returnAddress.toString().includes("libvmp.so")) {
            Memory.protect(ptr(this.context.x0), 
                         this.context.x1.toInt32(),
                         'rwx');
        }
    }
});


```

8. ARM64 性能优化
-------------

### 8.1 并行模拟技术

```
from multiprocessing import Pool

def parallel_emulate(blocks):
    with Pool(4) as p:
        results = p.map(emulate_block, blocks)
    return merge_results(results)


```

### 8.2 缓存机制实现

```
class VMCache:
    def __init__(self):
        self.block_cache = {}
    
    def get(self, address):
        if address in self.block_cache:
            return self.block_cache[address]
        block = self._analyze_block(address)
        self.block_cache[address] = block
        return block


```

9. ARM64 实战案例
-------------

### 9.1 算法还原流程

1.  定位加密函数在 so 中的位置
    
2.  识别 ARM64 虚拟化指令模式
    
3.  重建虚拟寄存器流转
    
4.  转换为标准 ARM64 汇编
    

### 9.2 协议逆向方法

1.  拦截 SSL_write/read 调用
    
2.  追踪 ARM64 寄存器中的缓冲区
    
3.  分析加密前 / 后的数据变化
    
4.  重建协议加解密流程
    

10. 法律合规建议
----------

1.  仅针对自己拥有版权的应用进行分析
    
2.  避免绕过 DRM 或付费验证机制
    
3.  研究结果仅用于安全防御目的
    
4.  遵守《计算机软件保护条例》相关规定
    

本指南提供的技术需要专业的 ARM64 架构知识，建议在合法授权的环境下使用。ARM64 逆向通常需要 200-500 小时的专业训练才能熟练掌握。