> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-2052520-1-1.html)

> [md][md]## unidbg 模拟执行的去除 ollvm 混淆 ### 1. 函数简单分析 *** 项目地址：https://github.com/Aar0n3906/Anti-BR-Obf*** 对 libtpr......

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)A4r0n _ 本帖最后由 A4r0n 于 2025-8-19 18:05 编辑_  

[md]## unidbg 模拟执行的去除 ollvm 混淆

#### 1. 函数简单分析

**_项目地址：[https://github.com/Aar0n3906/Anti-BR-Obf](https://github.com/Aar0n3906/Anti-BR-Obf)_**

对 libtprt.so 中的 JNI_Onload 函数进行去混淆

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250324222238242.png)

可以发现在函数后方使用了 BR X9 作为间接跳转，IDA 无法分析控制流了，因为在此处 X9 为寄存器，在未执行时不知道寄存器的值为多少，所以静态看我们无法了解程序往哪走

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250324222522858.png)

F5 反编译后可以看到 jni->GetEnv 函数后，执行 BR X9 后就无法看到其余逻辑了

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250324222737563.png)

在 JNI_Onload 下方还能看到许多对寄存器操作的汇编代码，猜测下方的汇编也为 JNI_Onload 执行的一部分

#### 2.Unidbg 环境的搭建

在这段混淆中我们使用模拟执行对函数进行去混淆

##### 2.1 创建项目

直接在项目的 unidbg-android/src/test/java 目录下建立我们的模拟执行类：AntiOllvm

1.  创建 64 位模拟器实例,  
    `emulator = AndroidEmulatorBuilder.for64Bit().build();`
2.  创建模拟器内存接口  
    `Memory memory = emulator.getMemory();`
3.  设置系统类库解析  
    `memory.setLibraryResolver(new AndroidResolver(23));`
4.  创建 Android 虚拟机  
    `vm = emulator.createDalvikVM();`
5.  加载 so 到虚拟内存, 第二个参数的意思表示是否执行动态库的初始化代码  
    `DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/java/com/xxx/xxx.so"),true);`
6.  获取 so 模块的句柄  
    `module = dm.getModule();`
7.  设置 JNI  需要继承`AbstractJni`  
    `vm.setJni(this);`
8.  打印日志  
    `vm.setVerbose(true);`
9.  调用 JNI_Onload  
    `dm.callJNI_OnLoad(emulator);`
10.  创建 jobject， 如果没用到的话可以不写 ，要用需要调用函数所在的 Java 类完整路径，比如 a/b/c/d 等等，注意. 需要用 / 代替  
    `cNative = vm.resolveClass("com/xxx/xxx")`

加载动态库 ==>

```
    public AntiOllvm() {
//        创建模拟器
        emulator = AndroidEmulatorBuilder
                .for64Bit()
                .addBackendFactory(new Unicorn2Factory(true))
                .setProcessName("com.example.antiollvm")
                .build();
        Memory memory = emulator.getMemory();
//        安卓SDK版本
        memory.setLibraryResolver(new AndroidResolver(23));
//        创建虚拟机
        vm = emulator.createDalvikVM();
        vm.setVerbose(true);

//        libtprt.so的依赖库
        vm.loadLibrary(new File("D:/unidbg/unidbg-android/src/main/resources/android/sdk23/lib64/libc.so"),false);
        vm.loadLibrary(new File("D:/unidbg/unidbg-android/src/main/resources/android/sdk23/lib64/libm.so"),false);
        vm.loadLibrary(new File("D:/unidbg/unidbg-android/src/main/resources/android/sdk23/lib64/libdl.so"),false);
        vm.loadLibrary(new File("D:/unidbg/unidbg-android/src/main/resources/android/sdk23/lib64/libstdcpp.so"),false);

        dm = vm.loadLibrary(new File("D:/unidbg/unidbg-android/src/test/resources/AntiOllvm/libtprt.so"), false);
        module = dm.getModule();
    }

```

加载后需要先执行 jni_onload，而 DalvikModule(dm) 这个类已经实现了 callJNI_OnLoad 方法，我们直接调用即可

```
    public void callJniOnload(){
        dm.callJNI_OnLoad(emulator);
    }
    public static void main(String[] args) {
        AntiOllvm AO = new AntiOllvm();
        AO.callJniOnload();
    }

```

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250324224556059.png)

可以看到在 0x87670 处进行了 RegisterNative，注册的函数名为：initialize，地址在 0x86e34

到这一步我们成功完成了使用 Unidbg 对安卓动态库的运行，并且正常运行了动态库的 Jni_Onload 函数

##### 2.2 基本的指令 hook

我们使用 hook 将每一步运行过的指令都打印出来

```
public void logIns()
    {
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user)  {
                Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM64,Capstone.CS_MODE_ARM);
                byte[] bytes = emulator.getBackend().mem_read(address, 4);
                Instruction[] disasm = capstone.disasm(bytes, 0);
                System.out.printf("%x:%s %s\n",address-module.base ,disasm[0].getMnemonic(),disasm[0].getOpStr());
            }

            @Override
            public void onAttach(UnHook unHook) {

            }

            @Override
            public void detach() {

            }
        }, module.base+start, module.base+end, null); 
    }

```

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250325010000849.png)

我们可以看到 br x9 往后执行的指令就是汇编代码中 BR 之后的指令

这段代码在 unidbg 中的作用是为指定模块的代码段添加**指令级动态跟踪钩子**，其效果是实时反汇编并打印该模块每条执行指令的详细信息。

* * *

**核心功能解析**

1.  **钩子注册**
    
    ```
    emulator.getBackend().hook_add_new(new CodeHook() { ... }, module.base, module.base+module.size, null);
    
    ```
    
    *   在模块的内存范围 `[module.base, module.base+module.size)` 内注册一个代码执行钩子。
    *   当模拟器执行到该范围内的任意指令时，会触发 `hook()` 方法。
2.  **指令反汇编**
    
    ```
    Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM64, Capstone.CS_MODE_ARM);
    byte[] bytes = emulator.getBackend().mem_read(address, 4);
    Instruction[] disasm = capstone.disasm(bytes, 0);
    
    ```
    
    *   使用 Capstone 反汇编引擎，将当前指令地址（`address`）处的 4 字节机器码转换为可读的汇编指令。
    *   **ARM64 指令特性**：固定长度为 4 字节（Thumb 模式为 2/4 字节混合，但此处明确指定`CS_MODE_ARM`，表明处理的是 ARM 模式指令）。
3.  **输出格式**
    
    ```
    System.out.printf("%x:%s %s\n", address - module.base, disasm[0].getMnemonic(), disasm[0].getOpStr());
    
    ```
    
    *   打印内容：
        *   **相对偏移**：`address - module.base` 显示指令相对于模块基址的位置，方便定位代码段中的具体位置。
        *   **助记符**：如 `BL`、`MOV` 等汇编指令名称。
        *   **操作数**：指令的具体参数（如寄存器、立即数等）。

#### 3. 去除间接跳转

```
CMP W8,W25
CSEL X9,X21,X25,CC
LDR X9,[X24,X9]
ADD X9,X9,X27
BR X9

```

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250325114221585.png)

1.  **比较寄存器值**  
    `CMP W8, W25`  
    比较 32 位寄存器 W8 和 W25 的值，设置条件标志位。若 W8 < W25，则进位标志位（C）被清除（CC 条件成立）。
2.  **条件选择偏移量**  
    `CSEL X9, X21, X26, CC`  
    根据 CC 条件（即 W8 < W25），选择 X21 或 X26 的值赋给 X9：
    *   若 W8 < W25，选择 X26 的值。
    *   否则，选择 X21 的值。
3.  **加载跳转地址**  
    `LDR X9, [X24, X9]`  
    以 X24 为基址，加上 X9 中的偏移量，从内存中加载一个 64 位地址到 X9。这通常用于访问跳转表（如函数指针表）。
4.  **调整地址**  
    `ADD X9, X9, X27`  
    将 X27 的值加到 X9 中，进一步调整目标地址。X27 可能存储固定偏移或基址，用于定位最终跳转位置。
5.  **跳转执行**  
    `BR X9`  
    无条件跳转到 X9 指向的地址，执行对应代码。

X27 的值由 MOV 和 MOVK 分别赋值 8 位和 16 位的值，固定为 ==> 0x84FA7910

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326002335785.png)

X24 的值是一个数组

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326002507046.png)

数组里面分别存了很多指令的地址，用于后续跳转使用

整体逻辑就是每次根据比较结果在数组中选择一个 offset，然后用`offset + base`，得到真实的跳转地址

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326002701682.png)

`CMP W8, W25`中的`W8`和`W25`的数值也是写死的

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326003109215.png)

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326003124020.png)

W8:0x3202B1A5

![](https://cdn.jsdelivr.net/gh/Aar0n3906/blog-img/image-20250326003207910.png)

W25:0x58F48322

```
CMP W8,W25
CSEL X9,X21,X25,CC
LDR X9,[X24,X9]
ADD X9,X9,X27
BR X9

```

以上方代码为例

当 CC 条件满足时，X21 的值赋给 X9 作为一个 offset，在`LDR X9,[X24,X9]`中使用 X24 的数组 + 偏移  
根据 CSEL 的 CC 条件有两个分支如下：

`True  Addr:  (*(X24+X21) + X27)`

`False Addr: (*(X24+X25) + X27)`

那么我们可以根据 CMP 的结果使用`BCC / BLO`和`B`对 True Addr 和 False Addr 进行跳转

替换后的汇编如下

```
CMP W8,W25
B.cond True Addr
LDR X9,[X24,X9]
ADD X9,X9,X27
B False addr

```

这样的间接跳转都变为了直接跳转，ida 内就可以继续分析了，并且地址也没有变化，因为寄存器的值已知，我们只是其他将他计算出来再跳转而已。

##### **3.1 目标**

代码的核心目标是自动化修复一种特定的代码混淆技术。这种混淆使用 ARM64 的 `csel` (条件选择) 指令和 `br` (间接跳转) 指令来隐藏真实的跳转目标。

*   **原始混淆代码:**
    
    ```
    cmp w0, w1         ; 比较，设置条件标志 (e.g., EQ, NE)
    ; ... 可能有其他指令 ...
    csel x9, x20, x21, cond ; 如果条件eq为真, x9 = x20, 否则 x9 = x21 (x20/x21存有地址或地址的基址)
    ; ... 可能有其他指令, 可能会修改 x9 (e.g., ldr x9, [x24, x9]) ...
    br x9              ; 跳转到 x9 中的地址
    
    ```
    
*   **修复后代码:**
    
    ```
    cmp w0, w1         ; 保留比较
    ; ... 保留其他指令 ...
    b.cond <目标地址1>   ; Patch 1: 在原 csel 位置替换为条件跳转 (如果cond为真，跳到b1)
    ; ... 保留其他指令 ...
    b <目标地址2>      ; Patch 2: 在原 br 位置替换为无条件跳转 (对应cond为假，跳到b2)
    
    ```
    

为了安全准确地找到 `<目标地址1>` (T) 和 `<目标地址2>` (F)，代码采用了**双模拟器**的方法。

##### **3.2 整体逻辑**

1.  **阶段 1: 发现与收集混淆特征 (使用主模拟器 `emulator`)**
    
    *   通过指令 Hook 监控每一条执行的指令。
    *   识别 `csel` 指令，记录其**操作数、条件、地址**，以及**执行并保存它当前的寄存器状态**。
    *   识别 `br` 指令，并回溯查找与之关联的 `csel`（通过目标寄存器匹配）。
    *   当找到匹配的 `csel` 和 `br` 时，**不立即模拟**，而是创建一个 `SimulationTask`，包含 `csel` 的信息、`br` 的地址以及**关键的 `csel` 执行前的寄存器状态**。将任务添加到 `simulationTasks` 列表。
2.  **阶段 2: 分支模拟与 Patch 生成 (使用临时模拟器 `tmpEmulator`)**
    
    *   主模拟器执行完毕后，遍历 `simulationTasks` 列表。
    *   对于每个任务：
        *   启动**临时模拟器 `tmpEmulator`**。
        *   **模拟真分支**: 恢复 `tmpEmulator` 到 `csel` 执行前的状态，强制 `csel` 目标寄存器为真分支，模拟执行直到原 `br` 位置，读取 `br` 寄存器的最终值得到 `b1 <True Addr>`。
        *   **模拟假分支**: **再次**恢复 `tmpEmulator` 到 `csel` 执行前的状态，强制 `csel` 目标寄存器为假分支，模拟执行直到原 `br` 位置，读取 `br` 寄存器的最终值得到 `b2 <False Addr>`。
        *   如果 `b1` 和 `b2` 有效且不同，则生成两条 Patch 指令（`b.cond b1` 和 `b b2`）并添加到 `patches` 列表。
3.  **阶段 3: 应用 Patch**
    
    *   将 `patches` 列表中的`code`写入文件缓冲区的对应位置。
        
    *   将修改后的数据写入新的 .so-patch 文件。
        

##### **3.3 变量解释**

*   `tmpEmulator`, `MainEmulator`: **临时模拟器**及其相关组件。用于安全地执行分支模拟。**为什么需要两个？** 避免在主模拟器运行时进行分支模拟可能导致的状态污染（寄存器、内存、Hook 状态被意外修改）。在写这段代码的时候尝试使用一个 emulator，但很容易在 patch 后往下走的分支造成非法内存访问，所以我选择使用两个 emu 分别进行特征收集和 patch 执行，这样代码的健壮性会高很多。
    
*   `insStack`: `Deque<InstructionContext>`。存储最近执行的指令及其执行前的寄存器状态。**为什么需要？** 当遇到 `br` 时，需要回溯查找之前的 `csel`，并且需要知道 `csel` 执行前的状态才能正确模拟。
    
    ```
    private final Deque<InstructionContext> insStack = new ArrayDeque<>(128);
    
    ```
    
*   `cselInfoMap`: `Map<Long, CselInfo>`。存储遇到的 `csel` 指令的详细信息，以其相对地址作为 Key，方便快速查找。
    
    ```
    private final Map<Long, CselInfo> cselInfoMap = new HashMap<>();
    
    ```
    

1.  **`DeOllvmBr_TwoEmus()`**:
    
    *   **初始化主模拟器 (`emulator`)**: 使用 `AndroidEmulatorBuilder` 配置并构建主模拟器。
    *   **初始化临时模拟器 (`tmpEmulator`)**: **重复**构建过程，创建第二个独立的模拟器实例。**关键在于**确保两者环境一致
    *   **基地址检查**: 检查 `module.base` 和 `tmpModule.base` 是否相同。这是一个重要的健全性检查。如果不同，所有传递给 `tmpEmulator` 的地址计算都需要做偏移调整。代码假设它们相同以简化。
    *   **设置 Hook**: 调用 `setupMainEmulatorHooks()` **只为主模拟器**设置代码 Hook。临时模拟器不需要全局 Hook。
2.  **`setupMainEmulatorHooks()`**:
    
    *   为**主模拟器**添加代码 Hook (`CodeHook`)。
    *   Hook 的范围是配置的 `START_ADDR` 到 `END_ADDR`。
    *   `hook()` 方法: 当主模拟器执行到范围内的指令时被调用。
        *   检查地址是否已被 `patchedAddresses` 记录。
        *   如果未被 Patch，调用 `processInstruction` 处理该指令。
    *   `onAttach()` 方法: Hook 成功附加后回调，用于保存 `UnHook` 引用。
    
    ```
       private void setupMainEmulatorHooks() {
           if (this.mainHook != null) {
               this.mainHook.unhook();
               this.mainHook = null;
           }
           System.out.println("  [Hook管理] 正在添加主模拟器 Hook...");
           emulator.getBackend().hook_add_new(new CodeHook() {
               @Override
               public void hook(Backend backend, long address, int size, Object user) {
                   // 主模拟器的 Hook 逻辑
                   long relativeAddr = address - module.base;
                   if (relativeAddr >= START_ADDR && relativeAddr <= END_ADDR) {
                       // 检查是否是已 Patch 地址 (基于最终 Patch 目标)
                       if (!patchedAddresses.contains(relativeAddr)) {
                           processInstruction(address, size, backend);
                       }
                   }
               }
    
               @Override
               public void onAttach(UnHook unHook) {
                   System.out.println("  [Hook管理] 主模拟器 Hook 已附加。");
                   DeOllvmBr_TwoEmus.this.mainHook = unHook;
               }
               @Override
               public void detach() {
                   System.out.println("  [Hook管理] 主模拟器 Hook 已分离。");
               }
           }, module.base + START_ADDR, module.base + END_ADDR, null);
       }
    
    ```
    
3.  **`processInstruction()`**:
    
    *   由主模拟器的 Hook 调用。
    *   再次检查 `patchedAddresses`。
    *   `saveRegisters(backend)`: **保存当前指令执行前的寄存器状态**（重中之重
    *   反汇编当前地址的指令。
    *   创建 `InstructionContext` (指令 + 执行前状态)。
    *   将 `context` 压入 `insStack`。
    *   如果是 `csel`，调用 `handleConditionalSelect`。
    *   如果是 `br`，调用 `handleBranchInstruction`。
    
    ```
       private void processInstruction(long absAddress, int size, Backend backend) {
           try {
               long relativeAddr = absAddress - module.base;
               if (patchedAddresses.contains(relativeAddr)) {
                   return;
               }
    
               List<Number> currentRegisters = saveRegisters(backend); // 保存主模拟器当前状态
               byte[] code = backend.mem_read(absAddress, size);
               Instruction[] insns = capstone.disasm(code, absAddress, 1);
               if (insns == null || insns.length == 0) return;
               Instruction ins = insns[0];
    
               InstructionContext context = new InstructionContext(relativeAddr, ins, currentRegisters);
               insStack.push(context);
               if (insStack.size() > 100) insStack.pollLast();
    
               System.out.printf("[MainEmu 执行] 0x%x (Rel: 0x%x): %s %s%n",
                       ins.getAddress(), relativeAddr, ins.getMnemonic(), ins.getOpStr());
    
               if ("csel".equalsIgnoreCase(ins.getMnemonic())) {
                   handleConditionalSelect(context);
               } else if ("br".equalsIgnoreCase(ins.getMnemonic())) {
                   // --- 不再调用模拟，而是检查并创建任务 ---
                   handleBranchInstruction(context);
               }
    
           } catch (Exception e) {
               System.err.printf("处理主模拟器指令错误 @ 0x%x: %s%n", absAddress, e.getMessage());
               e.printStackTrace();
           }
       }
    
    ```
    
4.  **`handleConditionalSelect()`**:
    
    *   从传入的 `InstructionContext` 获取**执行前的寄存器状态**。
    *   读取条件为真 / 假时源寄存器的**值** (`trueSourceValue`, `falseSourceValue`)。
    *   创建 `CselInfo` 对象存储这些信息。
    *   将 `CselInfo` 存入 `cselInfoMap`。
    
    ```
       private void handleConditionalSelect(InstructionContext currentContext) {
           Instruction ins = currentContext.instruction;
           long relativeAddr = currentContext.relativeAddr;
           String opStr = ins.getOpStr();
           String[] ops = opStr.split(",\\s*");
           if (ops.length < 4) return;
    
           String destReg = ops[0].trim();
           String trueReg = ops[1].trim();
           String falseReg = ops[2].trim();
           String condition = ops[3].trim().toLowerCase();
           List<Number> registersBeforeCsel = currentContext.registers; // CSEL 执行前的状态
    
           try {
               long trueSourceValue = getRegisterValue(trueReg, registersBeforeCsel);
               long falseSourceValue = getRegisterValue(falseReg, registersBeforeCsel);
               CselInfo info = new CselInfo(relativeAddr, destReg, condition, trueReg, falseReg, trueSourceValue, falseSourceValue);
               cselInfoMap.put(relativeAddr, info);
               System.out.printf("[MainEmu CSEL 发现] @0x%x: %s = %s ? %s(0x%x) : %s(0x%x). Cond: %s%n",
                       relativeAddr, destReg, condition, trueReg, trueSourceValue, falseReg, falseSourceValue, condition);
           } catch (IllegalArgumentException e) {
               System.err.printf("[MainEmu CSEL 错误] @0x%x: %s%n", relativeAddr, e.getMessage());
           }
       }
    
    ```
    
5.  **`handleBranchInstruction()`**:
    
    *   解析 `br` 指令，获取目标寄存器名。
        
    *   **回溯 `insStack`**: 查找最近执行的指令。
        
    *   检查历史指令是否是 `cselInfoMap` 中记录的 `csel`。
        
    *   如果找到 `csel`，并且其目标寄存器与 `br` 使用的寄存器匹配：
        
        *   **关键**: 调用 `findInstructionContext(prevRelativeAddr)` 从 `insStack` 中获取该 `csel` 对应的、包含**执行前状态**的 `InstructionContext`。
        *   创建 `SimulationTask` 对象，封装 `cselInfo`、`br` 的相对地址、以及最重要的 `registersBeforeCsel`。
        *   将 `task` 添加到 `simulationTasks` 列表。
        *   找到匹配后即返回，不再为同一个 `br` 查找更早的 `csel`。
        
        ```
        private void handleBranchInstruction(InstructionContext brContext) {
         Instruction brIns = brContext.instruction;
         long brRelativeAddr = brContext.relativeAddr;
         String brReg = brIns.getOpStr().trim();
        
         System.out.printf("[MainEmu BR 发现] @0x%x: br %s. 查找匹配 CSEL...%n", brRelativeAddr, brReg);
        
         int searchDepth = 0;
         int maxSearchDepth = 30;
         Iterator<InstructionContext> it = insStack.iterator();
         if (it.hasNext()) it.next(); // Skip self
        
         while (it.hasNext() && searchDepth < maxSearchDepth) {
             InstructionContext prevContext = it.next();
             long prevRelativeAddr = prevContext.relativeAddr;
        
             if (cselInfoMap.containsKey(prevRelativeAddr)) {
                 CselInfo cselInfo = cselInfoMap.get(prevRelativeAddr);
                 if (cselInfo.destinationRegister.equalsIgnoreCase(brReg)) {
                     System.out.printf("  [MainEmu BR 匹配] CSEL @0x%x. 创建模拟任务...%n", prevRelativeAddr);
        
                     // --- 关键：获取 CSEL 执行前的状态 ---
                     InstructionContext cselContext = findInstructionContext(prevRelativeAddr);
                     if (cselContext == null) {
                         System.err.printf("  [MainEmu 错误] 无法找到 CSEL @0x%x 的上下文! 跳过任务创建.%n", prevRelativeAddr);
                         return; // 无法获取必要的状态
                     }
                     List<Number> registersBeforeCsel = cselContext.registers;
        
                     // 创建模拟任务
                     SimulationTask task = new SimulationTask(
                             cselInfo,
                             brRelativeAddr,
                             registersBeforeCsel,
                             module.base + cselInfo.cselAddress, // cselAbsAddr
                             module.base + brRelativeAddr      // brAbsAddr
                     );
                     simulationTasks.add(task);
                     System.out.printf("  [MainEmu 任务已添加] CSEL 0x%x -> BR 0x%x%n", cselInfo.cselAddress, brRelativeAddr);
        
                     // 可选：从 Map 中移除，防止一个 CSEL 被多个 BR 错误匹配
                     // cselInfoMap.remove(prevRelativeAddr);
                     return; 
                 }
             }
             searchDepth++;
         }
         // System.err.printf("[MainEmu BR 警告] @0x%x: 未找到 %s 的匹配 CSEL%n", brRelativeAddr, brReg);
        }
        
        ```
        
6.  **`performSimulationsOnTmpEmu()`**:
    
    *   **协调临时模拟**: 接收一个 `SimulationTask`。
    *   获取 `tmpEmulator` 的后端接口 `tmpBackend`。
    *   调用 `performSingleSimulation(tmpBackend, task, true)` 模拟真分支，得到 `b1`。
    *   调用 `performSingleSimulation(tmpBackend, task, false)` 模拟假分支，得到 `b2`。
    *   比较 `b1` 和 `b2`。如果有效且不同，调用 `generatePatch` 生成 Patch。
    
    ```
       private void performSimulationsOnTmpEmu(SimulationTask task) {
           System.out.printf("%n[TmpEmu] ===> 开始模拟任务: CSEL 0x%x -> BR 0x%x ===>%n",
                   task.cselInfo.cselAddress, task.brRelativeAddr);
    
           Backend tmpBackend = tmpEmulator.getBackend();
    
           // --- 模拟真分支 ---
           System.out.println("  [TmpEmu] --- 模拟真分支 (True) ---");
           long b1 = performSingleSimulation(tmpBackend, task, true);
           System.out.printf("  [TmpEmu] --- 真分支结果 b1 = 0x%x ---%n", b1);
    
           // --- 模拟假分支 ---
           System.out.println("  [TmpEmu] --- 模拟假分支 (False) ---");
           long b2 = performSingleSimulation(tmpBackend, task, false);
           System.out.printf("  [TmpEmu] --- 假分支结果 b2 = 0x%x ---%n", b2);
    
           // --- 处理结果 ---
           if (b1 != -1 && b2 != -1) { // 检查模拟是否成功
               if (b1 != b2) {
                   System.out.printf("  [TmpEmu 成功] 发现不同跳转目标: 真=0x%x, 假=0x%x. 生成 Patch.%n", b1, b2);
                   // 注意：generatePatch 需要绝对地址 b1, b2
                   generatePatch(task.cselInfo, task.brRelativeAddr, b1, b2);
               } else {
                   System.out.printf("  [TmpEmu 注意] 真假分支目标相同 (0x%x). 无需 Patch 或为其他模式.%n", b1);
               }
           } else {
               System.err.printf("  [TmpEmu 失败] 模拟未能确定跳转目标 (b1=0x%x, b2=0x%x).%n", b1, b2);
           }
           System.out.printf("[TmpEmu] <=== 模拟任务结束: CSEL 0x%x -> BR 0x%x <===%n",
                   task.cselInfo.cselAddress, task.brRelativeAddr);
       }
    
    ```
    
7.  **`performSingleSimulation()`**:
    
    *   **核心模拟逻辑**: 在 `tmpEmulator` 上执行。
    *   `restoreRegisters(tmpBackend, task.registersBeforeCsel)`: **重置 `tmpEmulator` 状态**到 `csel` 执行前的样子。
    *   根据 `simulateTrueBranch` 标志，强制向 `csel` 的目标寄存器写入 `trueSourceValue` 或 `falseSourceValue`。
    *   设置 `tmpEmulator` 的 PC 到 `csel` 指令之后的位置 (`startPc`)。
    *   **添加临时 Hook**: 为 `tmpBackend` 添加一个临时的 `CodeHook`。这个 Hook 只关心执行是否到达了原始 `br` 的绝对地址 (`brAbsAddr`)。
        *   如果到达 `brAbsAddr`，Hook 读取 `br` 使用的寄存器的当前值（这就是模拟得到的跳转目标），存入 `resultHolder`，然后调用 `tmpBackend.emu_stop()` **停止当前这次模拟**，并设置 `stopped` 标志。
        *   使用 `UnHook[] tempHookHolder` 模式来在 `onAttach` 中获取 `UnHook` 引用。
    *   `tmpBackend.emu_start(...)`: **启动模拟执行**。从 `startPc` 开始，最多执行到 `brAbsAddr + 8`（留一点余量），并设置指令数超时限制。
    *   获取 `resultHolder` 中的结果（模拟得到的绝对跳转地址）。
    *   **`finally` 块**: 确保移除临时添加的 Hook (`tempHookHolder[0].unhook()`)，清理现场。
    *   返回模拟得到的跳转目标地址 `targetAbsAddress` (或 -1 表示失败)。
8.  **`generatePatch()`**:
    
    *   接收 `cselInfo`、`brRelativeAddr` 和模拟得到的两个**绝对**目标地址 `b1`, `b2`。
    *   计算两个新跳转指令的**相对偏移量**:
        *   `b.cond b1`: 替换 `csel`。PC 是 `csel` 地址，目标是 `b1`。偏移 = `b1 - cselAbsoluteAddr`。
        *   `b b2`: 替换 `br`。PC 是 `br` 地址，目标是 `b2`。偏移 = `b2 - brAbsoluteAddr`。
    *   创建两个 `Patch` 对象，分别对应 `csel` 和 `br` 的位置。
    *   将 `cselRelativeAddr` 和 `brRelativeAddr` 添加到 `patchedAddresses`。
    
    ```
    private void generatePatch(CselInfo cselInfo, long brRelativeAddr, long trueTargetAbsAddress, long falseTargetAbsAddress) {
       long cselRelativeAddr = cselInfo.cselAddress;
    
       // 检查地址是否已被 Patch
       if (patchedAddresses.contains(cselRelativeAddr) || patchedAddresses.contains(brRelativeAddr)) {
           System.out.printf("  [Patch 跳过] 地址 0x%x 或 0x%x 已标记 Patch.%n", cselRelativeAddr, brRelativeAddr);
           return;
       }
       if (cselRelativeAddr == brRelativeAddr || Math.abs(cselRelativeAddr - brRelativeAddr) < 4) {
           System.err.printf("  [Patch 错误/警告] CSEL (0x%x) 和 BR (0x%x) 地址相同或重叠.%n", cselRelativeAddr, brRelativeAddr);
           return; // 避免覆盖
       }
    
       try {
           // 获取绝对地址 (基于主模块)
           long cselAbsoluteAddr = module.base + cselRelativeAddr;
           long brAbsoluteAddr = module.base + brRelativeAddr;
    
           // Patch 1: 条件跳转 @ CSEL 位置 (b.cond b1)
           long offset1 = trueTargetAbsAddress - cselAbsoluteAddr;
           String condJumpAsm = String.format("b.%s #0x%x", cselInfo.condition.toLowerCase(), offset1);
    
           // Patch 2: 无条件跳转 @ BR 位置 (b b2)
           long offset2 = falseTargetAbsAddress - brAbsoluteAddr;
           String uncondJumpAsm = String.format("b #0x%x", offset2);
    
           // 范围检查 (可选)
           // ... (可以保留之前的范围检查代码，使用 offset1 和 offset2) ...
    
           // 添加 Patch (使用相对地址)
           patches.add(new Patch(cselRelativeAddr, condJumpAsm, trueTargetAbsAddress));
           patches.add(new Patch(brRelativeAddr, uncondJumpAsm, falseTargetAbsAddress));
    
           // 标记地址已 Patch
           patchedAddresses.add(cselRelativeAddr);
           patchedAddresses.add(brRelativeAddr);
    
           System.out.printf("    [Patch 已生成] @CSEL 0x%x: %s (目标: 0x%x)%n", cselRelativeAddr, condJumpAsm, trueTargetAbsAddress);
           System.out.printf("                   @BR   0x%x: %s (目标: 0x%x)%n", brRelativeAddr, uncondJumpAsm, falseTargetAbsAddress);
    
       } catch (Exception e) {
           System.err.printf("  [Patch 生成错误] @CSEL 0x%x -> BR 0x%x: %s%n", cselRelativeAddr, brRelativeAddr, e.getMessage());
           e.printStackTrace();
       }
    }
    
    ```
    
9.  **辅助方法**:
    
    *   `saveRegisters`, `restoreRegisters`: 保存 / 恢复 ARM64 通用寄存器状态。
    *   `getRegisterValue`: 从保存的状态列表中读取寄存器值。
    *   `getRegisterId`: 将寄存器名称字符串转为 Unicorn 的常量 ID。
    *   `findInstructionContext`: 在 `insStack` 中根据地址查找对应的上下文。
    *   `bytesToHex`: 格式化输出。

##### 执行前后对比

![](https://bbs.kanxue.com/upload/attach/202507/985355_BX4EQBRZMPH3M2M.png)  
**最后的最后，求各位大佬 Star 本项目**

[/md]![](https://avatar.52pojie.cn/data/avatar/000/11/16/22_avatar_middle.jpg)Poner _ 本帖最后由 Poner 于 2025-8-19 17:30 编辑_  
是我卡了还是咋滴   这帖子我成沙发了？    @ A4r0n 老兄   你的图都加载不出来   建议上传论坛附件勒![](https://avatar.52pojie.cn/images/noavatar_middle.gif) A4r0n [Poner 发表于 2025-8-19 17:29](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=53701490&ptid=2052520)  
是我卡了还是咋滴   这帖子我成沙发了？    @ A4r0n 老兄   你的图都加载不出来   建议上传论坛附件勒 之前应该是图片格式有问题，感谢提醒！已修复![](https://avatar.52pojie.cn/data/avatar/000/11/16/22_avatar_middle.jpg) Poner [A4r0n 发表于 2025-8-20 00:37](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=53702923&ptid=2052520)  
之前应该是图片格式有问题，感谢提醒！已修复 目测是你图床有问题，还是传论坛附件靠谱些![](https://avatar.52pojie.cn/images/noavatar_middle.gif) luojj314 感谢分享![](https://avatar.52pojie.cn/images/noavatar_middle.gif) aaaa25852 学习思路了![](https://avatar.52pojie.cn/data/avatar/001/53/22/67_avatar_middle.jpg) aonima 感谢分享