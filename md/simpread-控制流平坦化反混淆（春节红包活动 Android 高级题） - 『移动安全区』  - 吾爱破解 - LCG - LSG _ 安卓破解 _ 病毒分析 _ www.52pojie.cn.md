> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1937641&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)circle2  

前言
==

春节期间就已经看到红包活动了，但是感觉自己完成时间比较长，当时就没做。后面断断续续做了 Android 这边的题目，还是学习到很多东西的。论坛上有许多红包活动解题的文章，Android 的高级题只有一个，所以解的过程还是很有记录和分享的价值的。

这篇文章主要是记录 Android 高级题控制流平坦化反混淆的思路和一些实现，现在实现还有缺陷，另外目标并不是高级题 flag 的获取，因为还未分析完成，所以后续可能会重新编辑本文，进行更新

使用的是 unidbg，目前可以已经验证可以反混淆的（2024-06-23）：

*   checkSn 函数 -- 0x1befc
*   base64 的函数 -- 0x24290
*   0x1e924（这个函数还有问题，只有前半部分是正常的）

简单看一下效果（这里是最外层函数 0x1befc），左边是进行一些简单 patch 之后的效果，可以看出是控制流平坦化的特征，右边是进行反混淆之后的结果

![](https://imgsrc.baidu.com/forum/pic/item/4610b912c8fcc3ce440a010dd445d688d53f20c2.png)

样本混淆特征 & 为什么反混淆
===============

样本中控制流平坦化控制块是这样的，前面部分和以前了解过的控制流平坦化差不多，但是跳转地址是动态计算的，所以我们无法看清楚方法的全貌。在 [爱破解 2024 春节红包活动 WP(全，含 Android 高级题)](https://www.52pojie.cn/thread-1897975-1-1.html) 文章中对 Android 高级题的处理主要是 1c110 处修改 nzcv 寄存器，1c11c 获取 x10 中的地址，这样就能获取符合 lt 以及不符合 lt 的跳转地址，再进行 patch 即可。

我也这样处理试了一下，我认为有一个缺点，由于还是几乎按照正常执行逻辑进行处理的，所以修改完成后会出现一些未被 patch 的逻辑，经常需要再补

另外这样是进行了简单的 patch，所以修复完成后，它只是回归到了正常的控制流平坦化的样子了，反编译得到的代码还需要进一步处理，为此我写了一个小工具获得当前块的下一个块，但是块之间的连接还是需要再进行处理，还是挺耗时间的

这里我用的是 ghidra, 它的 image base 是 0x100000，所以这里实际偏移是 0x1c0fc, 0x1c100, ...

```
  0011c0fc 6a 52 82 52     mov        w10, #0x1293
  0011c100 ea 0b b7 72     movk       w10, #0xb85f, LSL #16
  0011c104 1f 01 0a 6b     cmp        w8, w10
  0011c108 0a 0a 80 52     mov        w10, #0x50
  0011c10c 0b 23 80 52     mov        w11, #0x118
  0011c110 6a b1 8a 9a     csel       x10, x11, x10, lt
  0011c114 2a 69 6a f8     ldr        x10, [x9, x10, LSL #0x0]
  0011c118 4a 01 0e 8b     add        x10, x10, x14
  0011c11c 40 01 1f d6     br         x10

```

反混淆的思路
======

下面介绍反混淆的思路，反混淆的方法是在 [爱破解 2024 春节红包活动 WP(全，含 Andro 高级题)](https://www.52pojie.cn/thread-1897975-1-1.html) 和 [利用 angr 符号执行去除控制流平坦化](https://bluesadi.me/0x401RevTrain-Tools/angr/10_%E5%88%A9%E7%94%A8angr%E7%AC%A6%E5%8F%B7%E6%89%A7%E8%A1%8C%E5%8E%BB%E9%99%A4%E6%8E%A7%E5%88%B6%E6%B5%81%E5%B9%B3%E5%9D%A6%E5%8C%96/) 基础上改进的

探索
--

这里需要首先介绍一个思路：探索。

将一个方法简化之后，其实就两部分，一是顺序执行，二是分支。如果将分支看作节点，以及节点的 true 和 false 后的执行部分看作两个分支，那么整个方法就是一个二叉树的结构，那么通过强制修改节点位置的 nzcv 寄存器，遍历这个二叉树其实是可能的。正常程序是不需要这样做的，但是这个样本中，由于控制流平坦化的跳转动态计算，无法直接看清全貌，那么就可以通过这样的遍历，建立起方法的全貌，而这样的遍历，我称为探索。

方法看作二叉树其实是简化的，实际可能是个图，因为它是可能存在循环的。另外强制修改 nzcv 寄存器来改变它的分支，可能存在执行错误，比如 read memory failed。

控制流平坦化的反混淆思路
------------

另外这里简单介绍一下控制流平坦化的反混淆思路

1.  区分出入口块，返回块，预分发器，主分发器，子分发器，真实块
2.  获取真实块之间的前后关系
3.  进行 patch

反混淆的过程
======

下面是反混淆的具体步骤，相对重要的是 1 和 7，代码本身只有两个文件：

1.  Test.java：主要是 unidbg 模拟执行的逻辑，1 和 5 才会涉及到，hook 时会调用 InstructionGraph.java 记录指令
2.  InstructionGraph.java: 包含指令合并，控制流平坦化块的区分，以及最终的 patch 输出

1. 初步探索
-------

探索其实就是遍历二叉树的一个过程，每探索一个分支就重新执行一遍程序。之前实现过一次执行，以及仅重复执行方法，均有问题。所以每次探索一个分支就跑一遍 unidbg 的 emulator build，load library，这样处理是最方便的。此外就是探索过程要实际做的事情：获取指令之间的前后顺序

### 1.1 遍历的整体逻辑

path 是一个 Node List，node 中包含的信息是 address 和 cond，表示涉及判断，即获取 NZCV 寄存器指令的地址，以及当时应选择的指令条件满足的状态，true 还是 false

1.  状态重置。这里是探索的代码以及 unidbg 的一些代码混在一块了，所以每次探索前需要将状态重置。这是因为之前尝试仅重复执行方法遗留的代码框架，就没做大的改动，理论上分开是更好的
2.  进行当前 path 的探索。path 最初是空的，我们也没有提前知道相应指令的地址，走指定的 path，那么这个 path 应该有多长，这些问题都是未知的。所以我这里采取的方法是，执行方法时，尽可能的往前探索未知的节点，未知的节点条件都走 true
3.  其他分支的探索。第二步探索之后，会新增一些为 true 的节点，将末尾转化为 false，就可以探索令一条分支了，下面演示以下

入参 path：node1-true, node2-false, node3-false

单次探索结果，也相当与这个新增全 true 的分支走过去了：node1-true, node2-false, node3-false, node4-true, node5-true, node6-true

那么需要探索的其他分支是：

explorer(node1-true, node2-false, node3-false, node4-true, node5-true, node6-false)

explorer(node1-true, node2-false, node3-false, node4-true, node5-false)

explorer(node1-true, node2-false, node3-false, node4-false)

再往前就归属于不属于传入 path 的范畴了，所以当前 path 的探索就结束了

```
private void explorer(List<Node> path) {
    // 每次探索前的准备
    String origin = path.toString();
    int originSize = path.size();
    this.currentPath = path;
    this.curNodeIndex = 0;
    this.accessedNodeInCommonBlock = new HashSet<>();
    this.explorerStatus = Status.RUNNING;
    this.accessedInstructions = new ArrayList<>();
    this.accessedRealInstructions = new ArrayList<>();
    this.accessedStartInstructions = new ArrayList<>();

    // 进行单次探索
    try {
        this.init();
        this.checkSn();
    } catch (Exception ignored) {}

    try {
        this.destory();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }

    System.out.println("origin path: " + origin);
    System.out.println("result path: " + this.currentPath);
    System.out.println("---------------------");

    // 进行其他分支的探索
    List<Node> tmpPath = this.currentPath;
    for (int i = tmpPath.size() - 1; i >= originSize; i--) {
        if (tmpPath.get(i).cond) {
            List<Node> newPath = new ArrayList<>();
            for (int j = 0; j < i; j++) {
                newPath.add(new Node(tmpPath.get(j).address, tmpPath.get(j).cond));
            }
            newPath.add(new Node(tmpPath.get(i).address, false));
            explorer(newPath);
        }
    }
}

class Node {
    long address;
    boolean cond;

    public Node(long address, boolean cond) {
        this.address = address;
        this.cond = cond;
    }

    @Override
    public String toString() {
        return String.format("%x %s", address, cond);
    }
}

```

### 1.2 新分支的探索

这一块是 `emulator.getBackend().hook_add_new` 的 `CodeHook()` 中的内容，每次碰到 nzcv 节点进行如下处理：

*   获取 cpsr，指令中的 cond string，然后得到当前是符合条件还是不符合条件
*   如果 index 还在 path 内，就按照 path 中的 cond 修改
*   如果不再 path 内，则修改为 true，同时创建新的节点添加到 path 中

```
String condStr = getCond(ins);
Cpsr cpsr = Cpsr.getArm64(backend);
boolean cond = isCondSuc(condStr, cpsr);
if (curNodeIndex < currentPath.size() && currentPath.get(curNodeIndex).address != offset) {
    System.err.println(String.format("Unexpected address: %X %s %s", address, ins, currentPath));
}
if (curNodeIndex < currentPath.size() && currentPath.get(curNodeIndex).cond != cond) {
    toggleCpsrStatus(condStr, cpsr);
}
if (curNodeIndex >= currentPath.size()) {
    if (!cond) {
        toggleCpsrStatus(condStr, cpsr);
    }
    // 新节点处理true的情况
    currentPath.add(new Node(offset, true));
}

```

### 1.3 记录指令之间的连接

在 hook 方法中：

```
if (!accessedInstructions.isEmpty()) {
    Instruction last = accessedInstructions.get(accessedInstructions.size() - 1);
    instructionGraph.addLink(last, ins);
}
accessedInstructions.add(ins);

```

在 InstructionGraph 中，这是创建用于处理指令图的类，同时在里面做了后续的控制流平坦化的相关处理

这部分应该比较简单，就是将 node 和 link 储存起来。这里通过 address 来判断是否重复，因为同一 address 的 Instruction 判断好像是不等的，所以 Set 处理不了

```
private Set<Instruction> nodes = new HashSet<>();
private Map<Long, List<Instruction>> toMap = new HashMap<>();
private Map<Long, List<Instruction>> fromMap = new HashMap<>();

public void addLink(Instruction ins, Instruction to) {
    addNode(ins);
    addNode(to);
    putLinkToMap(toMap, ins.getAddress(), to);
    putLinkToMap(fromMap, to.getAddress(), ins);
}

private void addNode(Instruction ins) {
    boolean include = false;
    for (Instruction i : nodes)
        if (i.getAddress() == ins.getAddress()) {
            include = true;
            break;
        }
    if (!include)
        nodes.add(ins);
}

private void putLinkToMap(Map<Long, List<Instruction>> map, long address, Instruction ins) {
    if (!map.containsKey(address)) {
        map.put(address, new ArrayList<>());
    }

    boolean include = false;
    for (Instruction i : map.get(address))
        if (i.getAddress() == ins.getAddress()) {
            include = true;
            break;
        }
    if (!include)
        map.get(address).add(ins);
}

```

### 1.4 避免死循环

需要在两个层面避免

1.  避免 path 中已经出现过的节点再次出现，重复出现直接退出
2.  避免节点之间的 instruction 出现死循环，重复出现直接退出

将 PC 改成一个不正确的数字，就会退出，我这里是改成了 - 1

下面是 `CodeHook` 中的代码

```
// 将PC随便设置个数字，就会异常退出
if (explorerStatus != Status.RUNNING) {
    emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_PC, -1);
    return;
}

// 判断之前的指令是否出现过
for (Instruction i : accessedInstructions)
    if (i.getAddress() == address) {
        // 死循环的需要正确串起来，所以这里需要加ins
        accessedInstructions.add(ins);
        explorerStatus = Status.INFINITE_LOOP;
        return;
    }
// 添加新遍历过的指令
accessedInstructions.add(ins);

// 判断节点是否重复
short[] read = ins.regsAccess().getRegsRead();
if (hasNZCV(ins, read)) {
    for (int i = 0; i < currentPath.size(); i++)
        if (i != curNodeIndex && currentPath.get(i).address == offset) {
            explorerStatus = Status.INFINITE_LOOP;
            return;
        }
}

```

### 1.5 避免重复探索

如果一个节点 true 和 false 均探索完成，那么就没有必要再继续探索了，下次碰到直接退出即可

这个可以极大的加快探索的速度，但是可能会引起探索不够充分的问题，代码比较乱，暂时不贴

2. 合并块
------

合并指令为块，其实相对简单，通过遍历之前存下的指令先后顺序即可创建出来

1.  首先找到 entry，即没有 from 的指令
2.  设置一个栈，储存待处理的指令（这些指令会是每个块的首个指令）
3.  从栈中取指令，判断是否可以 merge，可以 merge 的标准是，指令之间地址是相连的，而且指令 1 的 to 只有指令 2，指令 2 的 from 只有指令 1
4.  直到找到不可 merge 的，将刚刚遍历过去的合在一起，放置到 mergeNodes 中
5.  同时将最后的节点的所有 to 放入栈中

此外，需要一个 accessed 来保证不重复，确保不会陷入死循环中

```
// Set还是List其实没有关系，最终都是通过遍历判断地址是否相同来保证唯一的
private Set<List<Instruction>> mergedNodes = new HashSet<>();
public void findEntry() {
    for (Instruction i : nodes) {
        if (i.getAddress() == 0x1befc + moduleBase) {
            System.out.println(fromMap.containsKey(i.getAddress()));
        }
        if (!fromMap.containsKey(i.getAddress())) {
            entry = i;
        }
    }
    if (entry == null) {
        throw new RuntimeException("entry is null");
    }
}

public void mergeNodes(boolean isFirst) {
    Set<Long> accessed = new HashSet<>();
    Stack<Instruction> pending = new Stack<>();
    // 前一个区块的最后一个地址对应的node，相当于包含了link的信息
    List<Link> graphLinkMap = new ArrayList<>();
    pending.push(entry);
    accessed.add(entry.getAddress());
    while (!pending.empty()) {
        Instruction ins = pending.pop();
        List<Instruction> from = fromMap.get(ins.getAddress());

        List<Instruction> mergedNode = new ArrayList<>();
        mergedNode.add(ins);
        while (canMerge(ins)) {
            ins = toMap.get(ins.getAddress()).get(0);
            mergedNode.add(ins);
        }

        mergedNodes.add(mergedNode);

        if (toMap.get(ins.getAddress()) != null) {
            for (Instruction i : toMap.get(ins.getAddress())) {
                if (!accessed.contains(i.getAddress())) {
                    pending.push(i);
                    accessed.add(i.getAddress());
                }
            }
        }
    }
}

private boolean canMerge(Instruction ins) {
    long address = ins.getAddress();
    List<Instruction> to = toMap.get(address);
    if (to == null || to.size() != 1) {
        return false;
    }
    // 如果下一节点的from只有当前这个元素，认为可合并
    return fromMap.get(to.get(0).getAddress()).size() == 1 && to.get(0).getAddress() <= ins.getAddress() + 4;
}

```

3. 区分控制流平坦化中的各种块
----------------

getPreDispatcher 是获取预分发块，它的特征是 from 的节点最多的

然后从 preDispather 的后继块中获取符合如下特征的的几个指令，那么这里的 w8 就是 flag reg 了

```
mov   w10, #0x1293
movk  w10, #0xb85f, LSL #16
cmp   w8, w10

```

遍历所有 mergedNodes，找出所有符合上述特征的几个指令，这里需要指定 cmp 中存在 flag reg（这里是 w8），符合特征的块就是分发器

不过后面我注意到以 br 结尾的块其实都是控制块，不过不能完全确认，所以之前的特征方式也保留了，如有不符合的但以 br 结尾的打 log

预分发器、主分发器、子分发器都认为是控制块，那么剩余的块就都是真实块了

```
private Set<List<Instruction>> controlNodes = new HashSet<>();
private Set<List<Instruction>> realNodes = new HashSet<>();

public void splitBlockToControlAndReal(boolean isFirst) {
    // 由于补充了新的分块，需要进行重置了
    controlNodes = new HashSet<>();
    realNodes = new HashSet<>();

    this.preDispather = getPreDispather();
    controlNodes.add(preDispather);

    for (Instruction ins : toMap.get(preDispather.get(preDispather.size() - 1).getAddress())) {
        List<Instruction> toBlock = getMergeNodeByStart(ins.getAddress());
        controlNodes.add(toBlock);
        assert toBlock != null;
        List<Instruction> matched = fuzzyMatchDispatch(toBlock);
        if (matched != null) {
            String otherReg = matched.get(0).getOpStr().split(",")[0];
            String[] regs = matched.get(2).getOpStr().split(",");
            reg = otherReg.equals(regs[0].trim())
                ? regs[1].trim()
                : regs[0].trim();
            System.out.printf("Find flag reg, %x reg: %s, ins: %s\n", ins.getAddress(), reg, matched);
        }
    }

    if (reg == null)
        throw new RuntimeException("flag reg is null");
    System.out.println("flag reg: " + reg);
    System.out.println(mergedNodes.size());

    for (List<Instruction> block: mergedNodes) {
        List<Instruction> matched = exactMatchDispatch(block, reg);
        if (matched != null) {
            controlNodes.add(block);
        } else if (block.get(block.size() - 1).getMnemonic().equals("br")) {
            controlNodes.add(block);
            System.err.println("this block is end with br, may be a control block " + Long.toHexString(block.get(0).getAddress() - moduleBase));
        }
    }

    for (List<Instruction> block: mergedNodes) {
        if (!controlNodes.contains(block)) {
            realNodes.add(block);
        }
    }
    System.out.println("control block " + controlNodes.size());
    System.out.println("real block " + realNodes.size());
}

```

4. 获取真实块的合集
-----------

我所看到的控制流平坦化反混淆的文章中，真实块是单个的，处理起来会比较简单，但实际真实块的情况会更复杂，这个样本中就是如此，会存在多个真实块，那么其实要获取在控制块和预分发块之间的真实块的合集，一个控制块就应有这样一个合集（后面发现有例外，但暂不考虑）

另外真实块合集之间并不是完全切割开来的，有一部分共用的块，但这些块都是在预分发器前面

这一步的目标是：

1.  获取真实块的合集，其实主要是获取这个合集中最初始的指令
2.  获取真实块合集的共用块
3.  判断合集的后继合集是一个还是两个，即判断是顺序执行还是有分支

下面是 `InstrutionGraph` 中的代码，这里主要是调用 mergeOneRealArea 去获取真实块合集，在 mergeOneRealArea 中会记录 block 的次数

1.  获取 entry 开头的真实块合集，从方法入口一直到分发器前面都是合集范围
2.  所有真实块中，如果 from 是控制块，那么它也是一个合集的开头
3.  如果 commonBlock 中值超过 1 的，那么就是共用块。这里顺便做了一个验证，确认之前的判断合理，共用块要不是预分发器的前驱，要不就是其他共用块的前驱

```
private Map<Long, Integer> commonBlock = new HashMap<>();

// 代码中都写的是 real area, block area这种，也就是上述的合集
public void mergeRealArea() {
    mergeOneRealArea(entry, to -> {
            for (List<Instruction> block : controlNodes)
                if (block.get(0).getAddress() == to.getAddress())
                    return true;
            return false;
        });

    for (List<Instruction> block : realNodes) {
        List<Instruction> from = fromMap.get(block.get(0).getAddress());
        if (from == null) {
            continue;
        }
        for (Instruction ins : from) {
            // 如果from block是控制块，那么就算是area start，才进行后续的处理
            List<Instruction> lastBlock = getMergeNodeByEnd(ins.getAddress());
            if (!controlNodes.contains(lastBlock)) {
                continue;
            }
            mergeOneRealArea(block.get(0), to -> preDispather.get(0).getAddress() == to.getAddress());
        }
    }

    for (long key : commonBlock.keySet())
        if (commonBlock.get(key) != 1) {
            // commonBlock
            // commonBlock目前认为会是preDispatcher前驱，或者是另一个commonBlock前驱
            List<Instruction> block = getMergeNodeByStart(key);
            List<Instruction> to = toMap.get(block.get(block.size() - 1).getAddress());
            if (to == null || to.size() != 1)
                throw new RuntimeException("unexpected common block form");
            long address = to.get(0).getAddress();
            if (!commonBlock.containsKey(address) && preDispather.get(0).getAddress() != address)
                throw new RuntimeException("unexpected common block form");
        }
}

```

下面看看 `mergeOneRealArea`，这里面主要是去遍历块直到末尾，类似之前第二步 -- 合并块的处理逻辑，只是这次直接处理块

*   遍历块时，如果块中存在指令满足 `"csel {flagReg}" ...` 的模式，则记其为 key ins，这个合集是会产生分支的合集。存下来它的 start 和 key ins 的地址，后续要用到
*   遍历块时，将信息储存到 commonBlock 中，用于后续的处理（大于 1 即为共用块）
*   将块的信息放在 blockAreaStart 中  `this.blockAreaStart.put(ins.getAddress(), area);`

```
private Map<Long, Integer> commonBlock = new HashMap<>();
private Map<Long, Set<Long>> blockAreaStart = new HashMap<>();
// 因为keyIns可能在common block中，所以start to keyins，而不是反过来
private Map<Long, Long> startMapTokeyIns = new HashMap<>();

interface OutBlockJudge {
    boolean isOutBlock(Instruction to);
}

// ins 是合集的开头
// outBlockJudge 是用于判断一个 block 是否是末尾
private void mergeOneRealArea(Instruction ins, OutBlockJudge outBlockJudge) {
    Set<Long> area = new HashSet<>();

    Set<Long> accessed = new HashSet<>();
    Stack<Instruction> pending = new Stack<>();  // 简化解构，只存block的首个ins
    accessed.add(ins.getAddress());
    pending.add(ins);
    int outBlockNum = 0;
    Instruction assign = null;

    // 遍历整个block area
    while (!pending.empty()) {
        Instruction top = pending.pop();
        List<Instruction> b = getMergeNodeByStart(top.getAddress());

        for (Instruction i : b) {
            area.add(i.getAddress());
            if (i.toString().startsWith("csel " + reg))
                assign = i;
        }

        // 计算block的命中次数，获取commonBlock
        int times = commonBlock.getOrDefault(top.getAddress(), 0);
        commonBlock.put(top.getAddress(), times + 1);

        List<Instruction> toList = toMap.get(b.get(b.size() - 1).getAddress());
        if (toList != null)
            for (Instruction to : toList)
                if (!accessed.contains(to.getAddress())) {
                    if (!outBlockJudge.isOutBlock(to)) {
                        pending.push(to);
                        accessed.add(to.getAddress());
                    } else {
                        // 这是顺便借个地方统计以下出口的数量
                        outBlockNum ++;
                    }
                }
    }

    this.blockAreaStart.put(ins.getAddress(), area);

    if (assign != null) {
        System.out.println("assign instruction: " + Long.toHexString(assign.getAddress()) + " " + assign);
        startMapTokeyIns.put(ins.getAddress(), assign.getAddress());
    }
    if (outBlockNum > 1) {
        System.out.println("the multi out block is:");
        for (Instruction i : getMergeNodeByStart(ins.getAddress())) {
            System.out.println(Long.toHexString(i.getAddress() - moduleBase) + " " + i);
        }
        throw new RuntimeException("outBlockNum is not one: " + Long.toHexString(ins.getAddress() - moduleBase));
    }
}

```

5. 二次探索
-------

在理清楚块之间的信息后，需要再进行二次探索，这次探索的主要目标是获取真实块合集之间的联系，所以仅处理在真实块中的指令

另外还有一个目标，是补充块信息。在初步探索完成后，我发现有些块有缺失，原因是缺少前置信息。因为第一次探索时，所有的读取 nzcv 的指令都被当作节点，那么控制块中也有指令，而我配置了节点不重复探索，这样一次探索的范围大概就是一条分支，只有一个真实区域，如果这个真实区域依赖前置数据，就会出错，有些时候 unidbg 会强制退出，导致后续指令都获取不全。

为了实现这两个目标，需要在之前 accessedInstructions 的基础上再加一个 accessedRealInstructions 和 accessedStartInstructions

*   accessedInstructions 依旧保持，这样才能补全后续指令，需要是补全的块和预分发块的联系根据它得到
*   accessedRealInstructions 是为了遍历真实块的，是第二次探索过程中的主力
*   accessedStartInstructions 是第二次探索的结果，用于联系真实块合集，它只有走到合集开头的指令时才有作用

这一块是 `emulator.getBackend().hook_add_new` 的 `CodeHook()` 中的内容，第二次探索前先设置一个 `this.onlyRealBlock = true;`，表明是第二次探索

```
// 连接上一条指令
if (onlyRealBlock && instructionGraph.isInRealBlock(address) && !accessedRealInstructions.isEmpty()) {
    Instruction last = accessedRealInstructions.get(accessedRealInstructions.size() - 1);
    instructionGraph.addRealLink(last, ins);
}
if (onlyRealBlock && instructionGraph.isBlockAreaStartOrEntry(address) && !accessedStartInstructions.isEmpty()) {
    Instruction last = accessedStartInstructions.get(accessedStartInstructions.size() - 1);
    // 这个稍微有点特殊，要加 keyInsCondSuc，如果合集后续是分支，就会用到
    instructionGraph.addBlockAreaLink(last, ins, keyInsCondSuc);
}

// 如果是第二次，判断节点的指令是否出现重复访问，就需要用到 accessedRealInstructions
for (Instruction i : onlyRealBlock ? accessedRealInstructions : accessedInstructions)
    if (i.getAddress() == address) {
        // 死循环的需要正确串起来，所以这里需要加ins
        accessedInstructions.add(ins);
        if (onlyRealBlock && instructionGraph.isInRealBlock(address)) {
            accessedRealInstructions.add(ins);
        }
        if (onlyRealBlock && instructionGraph.isBlockAreaStartOrEntry(address)) {
            accessedStartInstructions.add(ins);
        }
        // commonBlock可以访问多次，所以这里需要把它从正常判断里面剔除
        if (!instructionGraph.isInCommonBlock(address)) {
            explorerStatus = Status.INFINITE_LOOP;
            return;
        }
        break;
    }

// 添加指令
accessedInstructions.add(ins);
if (onlyRealBlock && instructionGraph.isInRealBlock(address)) {
    accessedRealInstructions.add(ins);
}
if (onlyRealBlock && instructionGraph.isBlockAreaStartOrEntry(address)) {
    accessedStartInstructions.add(ins);
}

```

这里也是 `CodeHook` 中的内容，处理会读取 nzcv 的指令，如果是第二次，就只处理在真实块中的指令

另外第二次时，对于在共用块中的指令，需要以 `{startAddr}-{nzcvAddr}` 做 key，保证可以共用块被多次遍历，但是一个合集下只遍历一次

如果命中 key ins 的话，需要知道 cond 是 true 还是 false，用于后续的连接，以及 patch

```
if (hasNZCV(ins, read) && (!onlyRealBlock || instructionGraph.isInRealBlock(address))) {
    // 由于第二遍探索，有一些commonBlock中的应该被遍历多遍，结果只被遍历一遍，特别排除一下
    if (!instructionGraph.isInCommonBlock(address)) {
        for (int i = 0; i < currentPath.size(); i++)
            if (i != curNodeIndex && currentPath.get(i).address == offset) {
                explorerStatus = Status.INFINITE_LOOP;
                return;
            }
    } else {
        Instruction areaStart = accessedStartInstructions.get(accessedStartInstructions.size() - 1);
        String key = areaStart.getAddress() + "-" + address;
        if (accessedNodeInCommonBlock.contains(key)) {
            explorerStatus = Status.INFINITE_LOOP;
            return;
        }
        accessedNodeInCommonBlock.add(key);
    }

    if (instructionGraph.isKeyIns(address)) {
        System.out.println("is key ins: " + Long.toHexString(offset));
        keyInsCondSuc = isCondSuc(condStr, cpsr);
    }
}

```

6. 合并块，对 2 进行补充
---------------

用二次探索的结果扩充之前合并块的结果，大体上可以复用之前的代码，主要注意 `!isFirst` 的逻辑，在产生 mergeNode 后，如果发现之前有记录，就直接删掉，用新的即可

另外的一些工作就是绘图，这样就会有一个相对直观的结果，其他地方也可以做这件事，比如之前区分各种块的时候，在图上标出了控制块，以及这里合并完成之后，可以将真实块之间连接起来

```
public void mergeNodes(boolean isFirst) {
    Set<Long> accessed = new HashSet<>();
    Stack<Instruction> pending = new Stack<>();
    // 前一个区块的最后一个地址对应的node，相当于包含了link的信息
    List<Link> graphLinkMap = new ArrayList<>();
    pending.push(entry);
    accessed.add(entry.getAddress());
    while (!pending.empty()) {
        Instruction ins = pending.pop();
        List<Instruction> from = fromMap.get(ins.getAddress());

        List<Instruction> mergedNode = new ArrayList<>();
        mergedNode.add(ins);
        while (canMerge(ins)) {
            ins = toMap.get(ins.getAddress()).get(0);
            mergedNode.add(ins);
        }

        if (!isFirst) {
            MutableNode node = Factory.mutNode(Label.html("<table border='0'>" + sb.toString() + "</table>"))
                .add(Shape.RECT, Style.ROUNDED);
            if (from != null) {
                for (Instruction i : from) {
                    graphLinkMap.add(new Link(i.getAddress(), node));
                }
            }
            graph.add(node);
            graphNodeMap.put(ins.getAddress(), node); // 记录，用于后续 graph 的链接
        }

        if (!isFirst)
            for (List<Instruction> block : mergedNodes)
                if (block.get(0).getAddress() == mergedNode.get(0).getAddress()) {
                    mergedNodes.remove(block);
                    break;
                }
        mergedNodes.add(mergedNode);

        if (toMap.get(ins.getAddress()) != null) {
            for (Instruction i : toMap.get(ins.getAddress())) {
                if (!accessed.contains(i.getAddress())) {
                    pending.push(i);
                    accessed.add(i.getAddress());
                }
            }
        }
    }

    if (!isFirst) {
        for (Link link : graphLinkMap) {
            MutableNode node = graphNodeMap.get(link.address);
            node.addLink(link.node);
        }
    }
}

```

7. 输出 patch 信息
--------------

到了最重要的 patch 环节，在反混淆控制流平坦化的文章中，真实块只有一个，patch 的过程其实相对简单，这个样本中会比较复杂，主要原因是存在共用块，导致不能直接进行 patch

所以除了获取真实块合集之间的联系之外，最重要的是处理 commonBlock，它每多触发一次，就需要多复制一份出来。可以复制到哪呢？控制流平坦化 patch 的时候，会将所有的控制块 patch 为 nop，所以我们能将这些控制块利用起来，复制到控制块即可。

其实 commonBlock 中的内容不一定重要，但是就怕万一，所以尽可能做到还原

### 7.1 patchAll

这个是整体 patch 的方法，和前面获取真实块合集的外层逻辑是差不多的，从合集的 start 开始遍历，调用 patchInstructionArea 进行真正的 patch，传入了一个 HashMap (patchInfo)，patchInstructionArea 会将 patch 的地址以及 patch 的指令放在这个 HashMap 中

最后将控制块中未被使用的地址都 patch 成 nop，然后输出信息

```
public void patchInstructionAll() {
    // 地址和要修改为的指令
    // 1. 旧跳转指令的patch
    // 2. commonBlock的复制
    // 所以用过的 controlNode 地址是会在这里面保存的
    Map<Long, String> patchInfo = new HashMap<>();

    patchInstructionArea(patchInfo, entry, to -> {
            for (List<Instruction> block : controlNodes)
                if (block.get(0).getAddress() == to.getAddress())
                    return true;
            return false;
        });

    for (List<Instruction> block : realNodes) {
        List<Instruction> from = fromMap.get(block.get(0).getAddress());
        if (from == null) {
            continue;
        }
        for (Instruction ins : from) {
            // 如果from block是控制块，那么就算是area start，才进行后续的处理
            List<Instruction> lastBlock = getMergeNodeByEnd(ins.getAddress());
            if (!controlNodes.contains(lastBlock)) {
                continue;
            }
            patchInstructionArea(patchInfo, block.get(0), to -> preDispather.get(0).getAddress() == to.getAddress());
        }
    }

    for (List<Instruction> block : controlNodes)
        for (Instruction ins : block)
            if (!patchInfo.containsKey(ins.getAddress() - moduleBase))
                patchInfo.put(ins.getAddress() - moduleBase, "nop");

    for (long key : patchInfo.keySet())
        System.out.println("patch: [" + Long.toHexString(key) + "] " + patchInfo.get(key));
}

```

### 7.2 patch 一个真实块合集

这个是 patch 的逻辑，后面有三部分空出来了，下面再进行介绍，这里先看以下对真实块合集的遍历，和之前获取真实块合集类似，要遍历合集中的每个真实块

```
private void patchInstructionArea(Map<Long, String> patchInfo, Instruction start, OutBlockJudge outBlockJudge) {
    int addition = condBlockAreaMap.containsKey(start.getAddress()) ? 1 : 0;
    Set<Long> area = new HashSet<>();

    // block 原有的到新的映射，在一次area中一个block只会被复制一次
    Map<Long, Long> copyBlock = new HashMap<>();
    Set<Long> accessed = new HashSet<>();
    Stack<Instruction> pending = new Stack<>();  // 只存block的首个ins
    accessed.add(start.getAddress());
    pending.add(start);
    // 遍历整个block area
    while (!pending.empty()) {
        Instruction top = pending.pop();
        List<Instruction> b = getMergeNodeByStart(top.getAddress());
        Instruction lastIns = b.get(b.size() - 1);
        long lastInsAddress = lastIns.getAddress();
        List<Instruction> toList = toMap.get(lastInsAddress);

        for (Instruction i : b)
            area.add(i.getAddress());

        if (toList != null) {

            // [1] 复制 commonBlock

            for (Instruction to : toList) {

                // [2] 处理 to 是 commonBlock 的块，会在这里面分配新区域用来复制commonBlock

                if (!accessed.contains(to.getAddress()))
                    if (outBlockJudge.isOutBlock(to)) {

                        // [3] 结尾块，这里要 patch 跳转指令，要跳转到下一个真实区块的合集

                    } else {
                        pending.push(to);
                        accessed.add(to.getAddress());
                    }
            }
        }
    }
}

```

#### 7.2.1 复制 commonBlock

这部分相对简单，因为关键逻辑在 7.2.2 中，这里只是将 commonBlock 复制到新的区块中。7.2.2 中会进行区块的分配，然后将原区块的首地址和新区块的首地址放在 copyBlock 中，所以只要 copyBlock 中有，开始复制就对了，然后将新分配的地址和相应的指令存放在 patchInfo 中

这里需要注意的是，之前遍历得到的指令，如果有跳转，是加了 module base 的，所以这里将 #0x400 给替掉，这个其实有点风险，可能正常逻辑里面也有这个，暂时做的粗糙一点

```
if (commonBlock.getOrDefault(top.getAddress(), 0) > 1 && copyBlock.containsKey(top.getAddress())) {
    // 遍历到commonBlock，此时copyBlock中已经包含了它的信息
    List<Instruction> common = getMergeNodeByStart(top.getAddress());
    long copyTo = copyBlock.get(top.getAddress());
    long base = 0;
    for (int i = 0; i < common.size(); i++, base ++) {
        if (i != 0 && copyBlock.containsKey(common.get(i).getAddress())) {
            copyTo = copyBlock.get(common.get(i).getAddress());
            base = 0;
        }
        copyBlock.put(common.get(i).getAddress(), copyTo + base * 4L);
        patchInfo.put(copyTo + i * 4L - moduleBase, common.get(i).toString().replaceAll("#0x400([0-9a-fA-F]+)", "#0x$1"));
    }
}

```

#### 7.2.2 分配区块用于后续 commonBlock 的复制

其实可以走到 commonBlock 时再进行分配，不过写在这里相对方便一些，但是相对麻烦，还得判断 from 是否是当前区块

当碰到下一个区块是 commonBlock 时，就需要分配了。但是有一个例外，就是 commonBlock 是在当前块后面，那么当前块尾部就不是跳转指令，这个就不进行复制，让它保持原状即可。也就是说有一个合集会直接使用 commonBlock，其他合集就要进行复制。

allocBlock 方法分配新的区块，新区块的大小是 commonBlock 的大小加一点额外空间

1.  如果当前真实区块有分支，那么再留一条指令，用于进行条件的跳转
2.  如果 commonBlock 是结尾不是分支语句，那么它和下一个块就是直接连接的，那么要留一条分支指令，用于跳转后续块

在分配了新的区块后，将映射信息存放在 copyBlock 中，同时将当前块的最后一个跳转指令进行 patch，使它跳转到新分配的区块

```
int addition = condBlockAreaMap.containsKey(start.getAddress()) ? 1 : 0;

// ...

// commonBlock的格式在前面mergeRealArea判断过了
// 如果下个block是commonBlock，并且还不是紧接着的，那么需要复制
if (commonBlock.getOrDefault(to.getAddress(), 0) > 1 && copyBlock.getOrDefault(lastInsAddress, lastInsAddress) + 4 != to.getAddress()) {
    List<Instruction> common = getMergeNodeByStart(to.getAddress());
    // 由于至少要包含一个指令以及一个跳转指令，所以必须
    // + 1 是为了避免一些问题（比如common下一个刚好是preDispatcher）
    // 或者是连续的两个common块，复制出来之后前一个要加跳转，等等
    // 总结是，如果不以b结尾，应该要改，所以要提前预备一个
    int brAddition = common.get(common.size() - 1).getMnemonic().equals("b") ? 0 : 1;
    Instruction alloc = allocBlock(common, common.size() + addition + brAddition, patchInfo, copyBlock);
    if (alloc == null)
        throw new RuntimeException("No enough controlNode " + Long.toHexString(common.get(0).getAddress()));
    else {
        // 连续common块，复制出来，需要patch，会走到这里面，但实际它是不以 b 指令结尾的，这里根据原指令筛除
        if (!lastIns.getMnemonic().equals("b") && lastInsAddress + 4 != to.getAddress())
            throw new RuntimeException("Unexpected last ins" + Long.toHexString(lastInsAddress) + " " + lastIns);
        // 如果是copy的block，那么patch地址是 copyBlock.get(lastInsAddres)，否则就是lastInsAddress
        // 所以用 copyBlock.getOrDefault(lastInsAddress, lastInsAddress) 即可
        boolean needNewIns = !lastIns.getMnemonic().equals("b");
        patchInfo.put(copyBlock.getOrDefault(lastInsAddress, lastInsAddress) + (needNewIns ? 4 : 0) - moduleBase,
                      String.format("b %d", alloc.getAddress() - moduleBase));
        copyBlock.put(to.getAddress(), alloc.getAddress());
    }
}

```

#### 7.2.3 尾块的处理

这里先简单说一下之前二次探索存的真实块之间的 link，如果不是 keyIns 中，那么就代表没有分支，那么直接存在 flowBlockAreaMap 中，如果在 keyIns 中，有分支，那么就存在 condBlockAreaMap 中，如果 cond 是 true，就存在第一个，false 就存在第零个

```
public void addBlockAreaLink(Instruction ins, Instruction to, boolean cond) {
    // TODO: 这里是为了筛除内部循环，但也可能是在控制流平坦化控制下的循环（如果发现 link 的有问题，有缺失，可以处理）
    if (ins.getAddress() == to.getAddress()) {
        System.out.printf("flow block area link error, link self %x -> %x\n", ins.getAddress(), to.getAddress());
        return;
    }
    if (startMapTokeyIns.containsKey(ins.getAddress())) {
        // 从 putLinkToMap 复制上来的，cond的这个有点不一样
        long fromAddress = ins.getAddress();
        if (!condBlockAreaMap.containsKey(fromAddress)) {
            List<Instruction> tmp = new ArrayList<>(2);
            tmp.add(null);
            tmp.add(null);
            condBlockAreaMap.put(fromAddress, tmp);
        }
        condBlockAreaMap.get(fromAddress).set(cond ? 1 : 0, to);
    } else {
        putLinkToMap(flowBlockAreaMap, ins.getAddress(), to);
    }
}

```

下面开始看尾块的处理逻辑，首先要找到跳转地址，这个无论是不是分支，都是会有一个跳转地址的，condBlockAreaMap 中的就取 false 的那个跳转地址

然后是针对不同情况需要找不同地址进行 patch，可能是当前块后续的指令，也可能是在复制块中，具体的情况可以看下面的注释

处理完成一个跳转地址，需要处理有分支情况的另一个跳转地址

*   首先找到当前区里的 keyIns，获取 keyIns 的条件，即 `csel w10, w11, w10, eq` 中的 `eq`，这样就可以得到 patch 的指令了，即 `b.{cond} {trueAddr}`。这里的 trueAddr 是 condBlockAreaMap 中 true 的跳转地址
*   从尾部的指令往前遍历，直到找到 keyIns，它们之间的 ins 假如是 `ins-1, keyIns, ins1, ins2, ..., ins-end-1, ins-end`，那么需要将它移动位置，修改为 `ins-1, ins1, ins2, ..., ins-end-1, keyIns, ins-end`，新的序列中 keyIns 的提防就被 patch 为刚刚的 `b.{cond} {trueAddr}`

```
if (outBlockJudge.isOutBlock(to)) {
    // 对于最后一个block，即out block（之前已经判断过一个area只有一个out block）
    // 首先对于所有的要进行patch的，要先找到一个用于patch跳转的地址
    // 1. 尾部直接存在b指令的，patch这个b即可
    // 2. 没有b指令，entry块，而且to是下一条指令，且to是controlBlock，那么以下一条指令作为patch
    // 3. 没有b指令，to是下一条指令，且to是preDispatcher，且当前是复制块，那么在尾部的下一条指令patch
    // 4. 没有b指令，to是下一条指令，且to是preDispatcher，那么在尾部的下一条指令patch
    // 剩下的是对于cond情况的真实块集合来说，需要一直往上找，直到在 startMapTokeyIns，过程中的指令暂存以下，然后开始 patch 这些指令（调整顺序）
    System.out.println("last ins: " + lastIns);

    long nextAddress = -1;
    if (flowBlockAreaMap.containsKey(start.getAddress()))
        nextAddress = flowBlockAreaMap.get(start.getAddress()).get(0).getAddress();
    else if (condBlockAreaMap.containsKey(start.getAddress()))
        // b.{cond} {trueAddr}
        // b        {falseAddr}
        nextAddress = condBlockAreaMap.get(start.getAddress()).get(0) == null ? -1 : condBlockAreaMap.get(start.getAddress()).get(0).getAddress();
    if (nextAddress == -1)
        // throw new RuntimeException("next address is null " + Long.toHexString(start.getAddress()));
        continue;

    String patchFlowInstruction = String.format("b %d", nextAddress - moduleBase);
    Instruction willPatchCondInstruction = null;
    if (lastIns.getMnemonic().equals("b")) {
        // 这里是假定block大小是大于1的，碰到等于1的再想办法处理
        willPatchCondInstruction = b.get(b.size() - 2);
        patchInfo.put(copyBlock.getOrDefault(lastInsAddress, lastInsAddress) - moduleBase, patchFlowInstruction);
    } else if (start.getAddress() == entry.getAddress()) {
        if (toList.size() == 1 && toList.get(0).getAddress() == lastInsAddress + 4) {
            willPatchCondInstruction = lastIns;
            patchInfo.put(copyBlock.getOrDefault(toList.get(0).getAddress(), toList.get(0).getAddress()) - moduleBase, patchFlowInstruction);
        } else
            throw new RuntimeException("Unknown entry block area format");
    } else if (toList.size() == 1 && toList.get(0).getAddress() == preDispather.get(0).getAddress()) {
        // 如果有复制的话，等后续再处理，这里只记原本的
        willPatchCondInstruction = lastIns;
        if (copyBlock.containsKey(top.getAddress())) {
            long targetPatchAddress = copyBlock.get(top.getAddress()) + b.size() * 4L;
            patchInfo.put(targetPatchAddress - moduleBase, patchFlowInstruction);
        } else {
            patchInfo.put(preDispather.get(0).getAddress() - moduleBase, patchFlowInstruction);
        }
    } else
        throw new RuntimeException("Unknown block area format " + Long.toHexString(start.getAddress()));

    if (condBlockAreaMap.containsKey(start.getAddress()) && condBlockAreaMap.get(start.getAddress()).get(1) != null) {
        List<Instruction> insBetweenKeyInsAndEnd = new ArrayList<>();
        insBetweenKeyInsAndEnd.add(willPatchCondInstruction);
        while (!startMapTokeyIns.containsValue(insBetweenKeyInsAndEnd.get(insBetweenKeyInsAndEnd.size() - 1).getAddress())) {
            Instruction cur = insBetweenKeyInsAndEnd.get(insBetweenKeyInsAndEnd.size() - 1);

            if (fromMap.containsKey(cur.getAddress())) {
                //                                        List<Instruction> from = fromMap.get(cur.getAddress());
                // 如果是commonBlock，可能存在多个from，所以这里需要筛选一下
                Instruction fromInArea = null;
                for (Instruction f : fromMap.get(cur.getAddress()))
                    if (area.contains(f.getAddress())) {
                        if (fromInArea == null)
                            fromInArea = f;
                        else
                            throw new RuntimeException("Find block area key ins error (multi from) " + Long.toHexString(cur.getAddress()) + " " + cur);
                    }
                insBetweenKeyInsAndEnd.add(fromInArea);
            } else
                throw new RuntimeException("Find block area key ins error (no from) " + Long.toHexString(cur.getAddress()) + " " + cur);
        }
        // end .... ins
        // end: b...
        long trueAddress = condBlockAreaMap.get(start.getAddress()).get(1).getAddress();
        Instruction keyIns = insBetweenKeyInsAndEnd.get(insBetweenKeyInsAndEnd.size() - 1);
        String[] insInfoList = keyIns.toString().split(",");
        String cond = insInfoList[insInfoList.length - 1].trim();
        patchInfo.put(copyBlock.getOrDefault(willPatchCondInstruction.getAddress(), willPatchCondInstruction.getAddress()) - moduleBase,
                      String.format("b.%s %d", cond, trueAddress - moduleBase));

        // 创建一个临时的list，因为跳转指令不参与重排
        List<Instruction> tmp = new ArrayList<>();
        for (int i = 0; i < insBetweenKeyInsAndEnd.size(); i++)
            if (!insBetweenKeyInsAndEnd.get(i).getMnemonic().equals("b"))
                tmp.add(insBetweenKeyInsAndEnd.get(i));
        // 从1开始，1 patch 0的，2 patch 1的
        for (int i = 1; i < tmp.size(); i++) {
            patchInfo.put(copyBlock.getOrDefault(tmp.get(i).getAddress(), tmp.get(i).getAddress()) - moduleBase, tmp.get(i - 1).toString());
        }
    }
}

```

### 7.3 allocBlock

这里是最简单的一个分配：如果说某一个控制块有足够的 size，而且 patchInfo, copyBlock 中都没出现过这个 block（代表它没有被分配过），那么就可以使用这块区域，将当前区域的首条指令返回即可，由外部进行 patch

```
private Instruction allocBlock(List<Instruction> common, int size, Map<Long, String> patchInfo, Map<Long, Long> copyBlock) {
    for (List<Instruction> controlNode : controlNodes) {
        // 这里需要避免entry之后的控制块被分配了，因为它可能是接着的，是没有跳转指令来覆盖的
        // 这个控制块一般是preDispatcher的后继
        List<Instruction> from = fromMap.get(controlNode.get(0).getAddress());
        boolean isLinkEntry = false;
        for (Instruction f : from)
            if (f.getAddress() == preDispather.get(preDispather.size() - 1).getAddress())
                isLinkEntry = true;
        if (isLinkEntry)
            continue;

        if (controlNode.size() >= size && !patchInfo.containsKey(controlNode.get(0).getAddress() - moduleBase) && !copyBlock.containsValue(controlNode.get(0).getAddress())) {
            return controlNode.get(0);
        }
    }
}

```

如果 patch 只是这样的话，可以略过，但是真正处理时，出现 size 比较大的情况，所以后面还需继续处理，需要多个控制块才能复制这个 common

倒着分配比较好处理，比如 commonBlock 需要复制为 block1, block2, block3，那么倒着分配的话，可以分配 block2 时，将它的末尾 patch 为 `b {block3Addr}`

一直分配，直到 rest 为 0

*   每次分配实际能用的是 controlNode.size - 1，因为需要 patch 跳转到下个分配块的指令
*   分配完成之后，需要将 controlNode 的首个指令映射到 commonBlock 中，将信息存放在 copyBlock 里面

```
private Instruction allocBlock(List<Instruction> common, int size, Map<Long, String> patchInfo, Map<Long, Long> copyBlock) {
    // ...

    // 倒着分配
    Instruction lastAlloc = null;
    int rest = size;
    // 对于没有一个区块能满足的
    for (List<Instruction> controlNode : controlNodes) {
        // 这里需要避免entry之后的控制块被分配了，因为它可能是接着的，是没有跳转指令来覆盖的
        // 这个控制块一般是preDispatcher的后继
        List<Instruction> from = fromMap.get(controlNode.get(0).getAddress());
        boolean isLinkEntry = false;
        for (Instruction f : from)
            if (f.getAddress() == preDispather.get(preDispather.size() - 1).getAddress())
                isLinkEntry = true;
        if (isLinkEntry)
            continue;

        // size - common.size() 是common块之外的，会有点对不起来，所以得手动处理
        // 理论上只有第一次分配，即分配尾部需注意，这里直接全做判断，简单一些
        if (controlNode.size() > 1 + (size - common.size()) && !patchInfo.containsKey(controlNode.get(0).getAddress() - moduleBase) && !copyBlock.containsValue(controlNode.get(0).getAddress())) {
            //                return controlNode.get(0);
            if (lastAlloc != null) {
                patchInfo.put(controlNode.get(Math.min(controlNode.size() - 1, rest + 1)).getAddress() - moduleBase, String.format("b %d", lastAlloc.getAddress() - moduleBase));
            }
            lastAlloc = controlNode.get(0);
            // 不同区块之间需要一个br指令去跳转下一个alloc区域
            rest = rest - (controlNode.size() - 1);
            if (rest <= 0) {
                return lastAlloc;
            } else
                copyBlock.put(common.get(rest + 1).getAddress(), controlNode.get(0).getAddress());
        }
    }
    if (rest > 0)
        throw new RunTimeExecption("No enough control block");
    return null;
}

```

结语
==

到此为止，反混淆的实现已经完成，它还不是很完美，但是反混淆的结果还是很令我满意的

本文完成之后并未进行细致的检查，而且也不是十分细致，因为写一遍已经耗费我数个小时了，暂时先这样，后续可能再做修正。如果任何问题，欢迎评论