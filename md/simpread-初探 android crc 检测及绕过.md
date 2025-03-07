> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5Bvku-K-UBDrkAQgrimXfQ)

```
1





前言




```

在开始之前先说下什么是 crc 检测, 通俗点讲就是, 把本地文件中的数据和内存中的数据进行 crc 计算得到的结果进行比较, 来校验结果是否一致, 不一致则判定数据被篡改。

举个例子: 以 libc.so 为目标 so, 当我们第一次用 frida 以 spwan 的方式注入 hook 时, 未对 libc.so 的函数进行 hook 的话, app 未退出, 一旦我们对 libc.so 中的函数或指令进行了修改注入 (不考虑 inline hoook 的因素影响),app 便直接崩溃退出, 这种情况基本就是检测到了数据被篡改, 也就是 crc 检测。

基本的介绍到这里, 下面开始对 crc 相关途径的检测进行分析, 以及如何去绕过。

本次分析以 libc.so 为目标, 以下的绕过都是用 frida 去处理。

```
2





分析环境




```

app 是我总结的一部分 crc 检测, 会把链接放置在结尾。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lckJn6YDMXABpGnNiaGksegicWWsSfTCufkDz45XIVZsztt569yG0o6VDg/640?wx_fmt=other&from=appmsg)  

frida 版本: 16.5.9  
目标 app: LinkerDemo  
分析 so: libc.so  
ELF 工具: 010Editor  
arm 平台: arm64

```
3





分析及绕过




```

(一) 检测种类
--------

目前 crc 的检测大的方向分两种:  
1. 本地文件与所属 app 的 / proc/{id}/maps 文件中 so 的内存范围作比较  
2. 本地文件与 linker 中获取到的 so 的内存范围作比较

(二) 本地文件与 maps 内存的校验
--------------------

先说一下此校验方法的相关逻辑。

描述：提取本地文件 / apex/com.android.runtime/lib64/bionic/libc.so 的可执行段数据和 app 在 / proc/{id}/maps 下映射的 libc.so 可执行段内存进行 crc 校验。

### 1. 用 010Editor 打开 libc.so

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcF8WrcXJQ0k20Rq6HNGSypqSPecqhCaOwJKqFp4iaqXlgNbILZ94JIJw/640?wx_fmt=other&from=appmsg)  

获取可执行段表中的 p_offset(在文件中的偏移) 和 p_filesz(在文件中的大小)。后续都是以这个为参照物与内存进行 crc 校验。

### 2. 获取 maps 内存中 libc.so 的可执行段

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcibGAe3EX0J7gZcick3ZEEtxSuCy7cxMXCBsF4KSakcrCjDVEqV5TypAA/640?wx_fmt=other&from=appmsg)  

(正常来说只有一行 r-xp 段，因为我使用了 frida, 所以会出现这种内存布局)

提取出里面带有 x 的内存段数据。

### 3. 校验

最后通过相关算法计算出两种途径获取到的内存结果进行比较。

  
算法一般都是 crc32，当然个例可能会使用其他的算法，比如 md5,aes 等等, 很少见的。

### 4.hook 现象

打开 app, 点击 libc maps crc，可以在控制台看到如下输出：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lc3EvJQZqYRoMDOAEd0cDZo7U3QFLqG9dFI1x5GH6p8xAt7qRVOSOG2w/640?wx_fmt=other&from=appmsg)  

此时我们的环境是正常的。接着我们用 frida 注入, 对 libc.so 中的 pthread_create 方法进行 hook, 得到以下输出：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lco0v0N53PvJA3mkH1VlFibtZppV017AxpUaW6m2qpBhcIbC9Px4R1Wng/640?wx_fmt=other&from=appmsg)  

可以很明显的看出内存中可执行段的 crc 值与文件中可执行的不一致，并且检测出了环境是 hook 的。

### 5. 绕过

针对上述的检测，我们可以在 maps 中模拟一段可执行段数据, 并把 libc.so 原本的可执行段名称给抹去, 变为匿名内存。最后 app 获取到的 maps 内存范围就是我们模拟的一段数据。

```
function hiddenSoExecSegmentInMaps(so_path) {
    //      /apex/com.android.runtime/lib64/bionic/libc.so
    let mmap_addr = Module.findExportByName("libc.so", "mmap");
    let mmapFunc = new NativeFunction(mmap_addr, 'pointer', ['pointer', 'int', 'int', 'int', 'int', 'int'])

    let munmap_addr = Module.findExportByName("libc.so", "munmap");
    let munmapFunc = new NativeFunction(munmap_addr, 'int', ['pointer', 'int'])

    let mremap_addr = Module.findExportByName("libc.so", "mremap");
    let mremapFunc = new NativeFunction(mremap_addr, 'pointer', ['pointer', 'int64', 'int64', 'int64', 'pointer'])

    let open_addr = Module.findExportByName("libc.so", "open");
    var openFunc = new NativeFunction(open_addr, 'int', ['pointer', 'int']);

    let memset_addr = Module.findExportByName("libc.so", "memset");
    var memsetFunc = new NativeFunction(memset_addr, 'pointer', ['pointer', 'int', 'int'])

    let close_addr = Module.findExportByName("libc.so", "close");
    var closeFunc = new NativeFunction(close_addr, 'int', ['int'])

    const parts = so_path.split('/');
    const so_name = parts.pop();
    let soExecSegmentRangeFromMaps = findSoExecSegmentRangeFromMaps(so_name);
    let startAddress = soExecSegmentRangeFromMaps.base;
    let size = soExecSegmentRangeFromMaps.size;

    if (startAddress === 0 || size === 0) {
        console.log("可执行段未找到:", startAddress, size)
        return;
    }

    let soExecSegmentFromFile = findSoExecSegmentFromFile(so_path);

    //创建匿名内存,临时存储so可执行段内存
    let new_addr = mmapFunc(ptr(-1), size, 7, 0x20 | 2, -1, 0);//0x20:匿名内存标识符(MAP_ANONYMOUS), 2:私有(MAP_PRIVATE)
    console.log("创建的可执行段匿名内存起始地址:" + new_addr);

    //把so可执行段内存复制到创建的匿名内存中去
    Memory.copy(new_addr, startAddress, size);
    console.log("复制完毕")

    //调整so,使传入的so可执行段内存变成匿名内存
    let ret = mremapFunc(new_addr, size, size, 1 | 2, startAddress);
    if (ret === -1) {
        console.log("mremap  调整失败")
        return;
    }
    console.log("匿名目标so可执行段完成 ret:" + ret)

    // 打开需要模拟的文件路径，用于后续在maps中生成指定名称的内存区域
    let moniter_path = so_path;
    let moniter_path_addr = Memory.allocUtf8String(moniter_path);
    var fd = openFunc(moniter_path_addr, 0);
    if (fd === -1) {
        console.log("open " + moniter_path + " is error")
        return -1;
    }

    //在maps中创建传入so路径名称的内存区域
    let target_addr = mmapFunc(ptr(-1), size, 7, 2, fd, 0);
    console.log("模拟的可执行段内存起始地址:" + target_addr);

    closeFunc(fd)

    //给创建的so内存区域全部置0
    memsetFunc(target_addr, 0, size);

    //把so文件中获取的可执行段内存复制到创建的so名称的内存区域中
    Memory.copy(target_addr, soExecSegmentFromFile.start, soExecSegmentFromFile.size)
    Memory.protect(target_addr, size, "r-x");

    //卸载映射的匿名内存
    // munmapFunc(new_addr, size);
    console.log("maps中隐藏可执行段完成")

}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcLYrlaFiawibZzUpmgDfV3ErFNvk3kzNvDibicV7IA8OSib5dy14IGdK4HmA/640?wx_fmt=other&from=appmsg)  

注入上述代码后再次点击按钮看控制台输出：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcXx4RPJj9ycfVbxXunof3cES7XcdCBZfic0hOibKn65Qg9bQZfKAdnEIw/640?wx_fmt=other&from=appmsg)  

可以看到两种方式获取到的值一致了，环境也是安全的了。

这里面主要使用到了 mmap 在 maps 中映射一段名称为 libc.so 的数据, 用 mremap 把原本的可执行段数据给设置为匿名内存。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcsx1TMJkoIzCgaYbQJopIXaX9iczPCYOP3wiac7Gr53ueSsvfnWYo9dGg/640?wx_fmt=other&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcccOTHZSdtxDquAJJ3fgYrPQiaTY77rySTIaDGwc2FRMk5ib6gxXtaLwQ/640?wx_fmt=other&from=appmsg)  

这里就可以看到 maps 中 libc.so 的可执行段已设置为匿名内存, 并且 map 中也有我们模拟的可执行段内存。

(三) 本地文件与 Linker 获取的内存校验
------------------------

本地文件获取的方式不再赘述了，直接看如何从 Linker 中获取内存。

### 1. 获取 libc.so soinfo 结构体

linker 作为 so 加载器, 里面存放了所有已经加载的 so, 并把这些已经加载的 so 会依次存放进 solist 变量中, solist 存储了所有 so 的 soinfo 结构体, 它是一个 soinfo 结构体数组, 我们可以从 solist 中获取到自己想要的 so。

那么如何获取到 libc.so 的结构体呢?

带着这个疑问我们先了解下 soinfo 的相关结构组成。

```
 struct soinfo {
  #if defined(__work_around_b_24465209__)
   private:
    char old_name_[SOINFO_NAME_LEN];
  #endif
   public:
    const ElfW(Phdr)* phdr; //0
    size_t phnum;	//1
  #if defined(__work_around_b_24465209__)
    ElfW(Addr) unused0; // DO NOT USE, maintained for compatibility.
  #endif
    ElfW(Addr) base;	//2
    size_t size;	//3
  
  #if defined(__work_around_b_24465209__)
    uint32_t unused1;  // DO NOT USE, maintained for compatibility.
  #endif
  
    ElfW(Dyn)* dynamic;	//4
  
  #if defined(__work_around_b_24465209__)
    uint32_t unused2; // DO NOT USE, maintained for compatibility
    uint32_t unused3; // DO NOT USE, maintained for compatibility
  #endif
  
    soinfo* next;	//5
   private:
    uint32_t flags_;	//6
  
    const char* strtab_;	//7 .dynstr
    ElfW(Sym)* symtab_;	//8		.dynsym
  
    size_t nbucket_;
    size_t nchain_;
    uint32_t* bucket_; //11
    uint32_t* chain_; //12
  
  #if defined(__mips__) || !defined(__LP64__)
    // This is only used by mips and mips64, but needs to be here for
    // all 32-bit architectures to preserve binary compatibility.
    ElfW(Addr)** plt_got_;	//arm64未被使用
  #endif
  
  #if defined(USE_RELA)
    ElfW(Rela)* plt_rela_; //.real.plt	13
    size_t plt_rela_count_;
  
    ElfW(Rela)* rela_;	//15 //real.dyn
    size_t rela_count_; 			//16
  #else
    ElfW(Rel)* plt_rel_;	
    size_t plt_rel_count_;
  
    ElfW(Rel)* rel_;		
    size_t rel_count_;		
  #endif
  
    linker_ctor_function_t* preinit_array_;//空 17
    size_t preinit_array_count_;//空	18
  
    linker_ctor_function_t* init_array_;
    size_t init_array_count_;
    linker_dtor_function_t* fini_array_;
    size_t fini_array_count_;
  
    linker_ctor_function_t init_func_; //空 23
    linker_dtor_function_t fini_func_;//	空 24
  
  #if defined(__arm__)	//arm64不进入
   public:
    // ARM EABI section used for stack unwinding.
    uint32_t* ARM_exidx;
    size_t ARM_exidx_count;
   private:
  #elif defined(__mips__)//arm64不进入
    uint32_t mips_symtabno_;
    uint32_t mips_local_gotno_;
    uint32_t mips_gotsym_;
    bool mips_relocate_got(const VersionTracker& version_tracker,
                           const soinfo_list_t& global_group,
                           const soinfo_list_t& local_group);
  #if !defined(__LP64__)//arm64不进入
    bool mips_check_and_adjust_fp_modes();
  #endif
  #endif
    size_t ref_count_;	//25
   public:
    link_map link_map_head;	//26 27 28 29 30
      // struct link_map {
            //     ElfW(Addr) l_addr;
            //     char* l_name;
            //     ElfW(Dyn)* l_ld;
            //     struct link_map* l_next;
            //     struct link_map* l_prev;
            // };
    bool constructors_called;
  
    // When you read a virtual address from the ELF file, add this
    // value to get the corresponding address in the process' address space.
    ElfW(Addr) load_bias; //32
  
  #if !defined(__LP64__)
    bool has_text_relocations;
  #endif
    bool has_DT_SYMBOLIC;
  
   public:
    soinfo(android_namespace_t* ns, const char* name, const struct stat* file_stat,
           off64_t file_offset, int rtld_flags);
    ~soinfo();
  
    void call_constructors();
    void call_destructors();
    void call_pre_init_constructors();
    bool prelink_image();
    bool link_image(const soinfo_list_t& global_group, const soinfo_list_t& local_group,
                    const android_dlextinfo* extinfo, size_t* relro_fd_offset);
    bool protect_relro();
  
    void add_child(soinfo* child);
    void remove_all_links();
  
    ino_t get_st_ino() const;
    dev_t get_st_dev() const;
    off64_t get_file_offset() const;
  
    uint32_t get_rtld_flags() const;
    uint32_t get_dt_flags_1() const;
    void set_dt_flags_1(uint32_t dt_flags_1);
  
    soinfo_list_t& get_children();
    const soinfo_list_t& get_children() const;
  
    soinfo_list_t& get_parents();
  
    bool find_symbol_by_name(SymbolName& symbol_name,
                             const version_info* vi,
                             const ElfW(Sym)** symbol) const;
  
    ElfW(Sym)* find_symbol_by_address(const void* addr);
    ElfW(Addr) resolve_symbol_address(const ElfW(Sym)* s) const;
  
    const char* get_string(ElfW(Word) index) const;
    bool can_unload() const;
    bool is_gnu_hash() const;
  
    bool inline has_min_version(uint32_t min_version __unused) const {
  #if defined(__work_around_b_24465209__)
      return (flags_ & FLAG_NEW_SOINFO) != 0 && version_ >= min_version;
  #else
      return true;
  #endif
    }
  
    bool is_linked() const;
    bool is_linker() const;
    bool is_main_executable() const;
  
    void set_linked();
    void set_linker_flag();
    void set_main_executable();
    void set_nodelete();
  
    size_t increment_ref_count();
    size_t decrement_ref_count();
    size_t get_ref_count() const;
  
    soinfo* get_local_group_root() const;
  
    void set_soname(const char* soname);
    const char* get_soname() const;
    const char* get_realpath() const;
    const ElfW(Versym)* get_versym(size_t n) const;
    ElfW(Addr) get_verneed_ptr() const;
    size_t get_verneed_cnt() const;
    ElfW(Addr) get_verdef_ptr() const;
    size_t get_verdef_cnt() const;
  
    int get_target_sdk_version() const;
  
    void set_dt_runpath(const char *);
    const std::vector<std::string>& get_dt_runpath() const;
    android_namespace_t* get_primary_namespace();
    void add_secondary_namespace(android_namespace_t* secondary_ns);
    android_namespace_list_t& get_secondary_namespaces();
  
    soinfo_tls* get_tls() const;
  
    void set_mapped_by_caller(bool reserved_map);
    bool is_mapped_by_caller() const;
  
    uintptr_t get_handle() const;
    void generate_handle();
    void* to_handle();
  
   private:
    bool is_image_linked() const;
    void set_image_linked();
  
    bool elf_lookup(SymbolName& symbol_name, const version_info* vi, uint32_t* symbol_index) const;
    ElfW(Sym)* elf_addr_lookup(const void* addr);
    bool gnu_lookup(SymbolName& symbol_name, const version_info* vi, uint32_t* symbol_index) const;
    ElfW(Sym)* gnu_addr_lookup(const void* addr);
  
    bool lookup_version_info(const VersionTracker& version_tracker, ElfW(Word) sym,
                             const char* sym_name, const version_info** vi);
  
    template<typename ElfRelIteratorT>
    bool relocate(const VersionTracker& version_tracker, ElfRelIteratorT&& rel_iterator,
                  const soinfo_list_t& global_group, const soinfo_list_t& local_group);
    bool relocate_relr();
    void apply_relr_reloc(ElfW(Addr) offset);
  
   private:
    // This part of the structure is only available
    // when FLAG_NEW_SOINFO is set in this->flags.
    uint32_t version_;
  
    // version >= 0
    dev_t st_dev_;
    ino_t st_ino_;
  
    // dependency graph
    soinfo_list_t children_;
    soinfo_list_t parents_;
  
    // version >= 1
    off64_t file_offset_;
    uint32_t rtld_flags_;
    uint32_t dt_flags_1_;
    size_t strtab_size_;
  
    // version >= 2
  
    size_t gnu_nbucket_;
    uint32_t* gnu_bucket_;
    uint32_t* gnu_chain_;
    uint32_t gnu_maskwords_;
    uint32_t gnu_shift2_;
    ElfW(Addr)* gnu_bloom_filter_;
  
    soinfo* local_group_root_;
  
    uint8_t* android_relocs_;
    size_t android_relocs_size_;
  
    const char* soname_;
    std::string realpath_;
  
    const ElfW(Versym)* versym_;
  
    ElfW(Addr) verdef_ptr_;
    size_t verdef_cnt_;
  
    ElfW(Addr) verneed_ptr_;
    size_t verneed_cnt_;
  
    int target_sdk_version_;
  
    // version >= 3
    std::vector<std::string> dt_runpath_;
    android_namespace_t* primary_namespace_;
    android_namespace_list_t secondary_namespaces_;
    uintptr_t handle_;
  
    friend soinfo* get_libdl_info(const char* linker_path, const soinfo& linker_si);
  
    // version >= 4
    ElfW(Relr)* relr_;
    size_t relr_count_;
  
    // version >= 5
    std::unique_ptr<soinfo_tls> tls_;
    std::vector<TlsDynamicResolverArg> tlsdesc_args_;
  }


```

这里我已经标注了相关变量在内存中的指针索引。里面描述了 so 的基址和一些节表和大量的方法。当然这些不是我们这里的关注重点, 我们只需要多关注以下的变量：

```
phdr:程序头表(段表)
base:	so基址
size:	so大小
dynamic: .dynamic节
next：下一个soinfo结构体
strtab_：.dynstr节
symtab_: .dynsym节
plt_rela_: .real.plt节
rela_：	.real.dyn节
link_map_head: 存储有so的基址和名字(dl_iterate_phdr方法可以获取)
load_bias: so基址


```

### 2.hook 现象

检测逻辑：  
lib base mem crc：获取本地文件的可执行段的偏移地址, 计算 soinfo 结构体中的 base 与可执行段偏移地址的和, 得到内存中的可执行段地址, 再取内存中可执行段数据和本地文件可执行段数据作比较。

  
lib func mem crc: 通过 dl_iterate_phdr 方法获取到 linker map, 取 linker map 中的 dlpi_addr 获取到 so 的基址, 获取本地文件的可执行段的偏移地址, 计算基址和偏移的和, 通过再取内存中可执行段数据和本地文件可执行段数据作比较。

分别点击 libc base mem crc 按钮和 libc func mem crc 按钮，控制台输出如下。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcuLSMQ2ibvXpHg8ibLDLBQq0GHt8Dop5AicOEH9ph1BhoaXzCeyBN2nILw/640?wx_fmt=other&from=appmsg)

### 3. 绕过

针对上述的检测，我们可以把 soinfo 结构体中的 base, 和 link_map_head 指针指向我们在 maps 中映射的地址, 达到绕过检测的目的。

```
function hiddenSobaseInMem(so_path) {
    //  /apex/com.android.runtime/lib64/bionic/libc.so
    let mmap_addr = Module.findExportByName("libc.so", "mmap");
    let mmapFunc = new NativeFunction(mmap_addr, 'pointer', ['pointer', 'int', 'int', 'int', 'int', 'int'])

    const parts = so_path.split('/');
    const so_name = parts.pop();

    let soExecSegmentFromFile = findSoExecSegmentFromFile(so_path);

    //获取maps中so的内存
    let soRangeFromMaps = findSoRangeFromMaps(so_name);
    let startAddress = soRangeFromMaps.base;
    let size = soRangeFromMaps.size;
    console.log(startAddress,size)
    //创建匿名内存,存储so内存区域的内存
    let new_addr = mmapFunc(ptr(-1), size, 7, 0x20 | 2, -1, 0);
    console.log("创建的匿名内存起始地址:" + new_addr);

    //把maps中so内存区域的内存复制到创建的匿名内存中去
    Memory.copy(new_addr, startAddress, size);
    console.log("复制完毕")

    //把文件中的可执行段复制到匿名内存中去
    Memory.copy(ptr(new_addr).add(soExecSegmentFromFile.p_offset), ptr(soExecSegmentFromFile.start), soExecSegmentFromFile.size);
    console.log("真实节区复制成功")

    //从linker中获取到soinfo结构链表
    let solist = getSolist();

    let num = 0;
    let soinfo_next;
    let realpath;

    console.log("开始遍历")
    do {
        realpath = getRealpath(solist);//获取soinfo所属的名字
        console.log(num + "-->" + realpath);

        if (realpath.indexOf(so_name) !== -1) {

            Memory.protect(ptr(solist).add(Process.pointerSize * 2), 4, "rw");
            ptr(solist).add(Process.pointerSize * 2).writePointer(new_addr);//so base

            Memory.protect(ptr(solist).add(Process.pointerSize * 26), 4, "rw");
            ptr(solist).add(Process.pointerSize * 26).writePointer(new_addr);//linker map

            //load_bias 没法直接修改 否则会因为找不到符号崩溃
            break;
        }
        soinfo_next = ptr(solist).add(Process.pointerSize * 5).readPointer();//soinfo结构体
        num++;
        solist = soinfo_next;
    } while (soinfo_next.toUInt32() !== 0);
    console.log("遍历完成:")
    console.log("内存中隐藏so首地址完成");
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcWRERFnkuaAHV5UM7vFbkHyYtvc3J7RKoyvbs99HgQle9NVeRzj5JmA/640?wx_fmt=other&from=appmsg)  

注入上述代码后, 再次点击这两个按钮查看控制台输出：

  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lc4qwiasFcayKWeiafHQX4xq0UJ3eJ60uPl3cGXWo11GpN1UkSWPoLkBQw/640?wx_fmt=other&from=appmsg)  

此时环境也正常了

注: 对 base 和 load_bias 进行修改, 会有 app 崩溃的风险

4.libc section mem crc 的绕过

  
这个我就简单说下检测及绕过思路。

#### 检测:

获取到 soinfo 结构体中的节表地址 (这里以 strtab_变量作检测), 再与本地文件或取到的节表偏移相见得到 so 的基地址, 计算基地址与可执行段的偏移得到可执行段的地址, 最后提取内存中可执行段数据和本地文件可执行段数据作比较。

#### 绕过：

把 soinfo 结构体中的节表指针指向我们在 maps 中映射的地址, 达到绕过检测的目的。

#### hook 现象:

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcHrUyPDjgnUfqsLb4MFFLCW6v8f4J3kRvoicsbAPiaycIsshVmUwoWX2A/640?wx_fmt=other&from=appmsg)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lc925gaRwMjH0xIAOsIpDcOBAHPWsJOKCTSlB8ZruUXOibVn8t0c8OBwQ/640?wx_fmt=other&from=appmsg)  

注入代码后再次点击按钮：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcZdwk1ZibQkITmHXT11TbKWibiclD6hzCXgObRFzzhDrV2fFD5icLcxUkMg/640?wx_fmt=other&from=appmsg)

此次分享主要是提供个思路仅供参考，包括 libart.so 也可以这样去弄，总之 crc 的大方向就是上述的两个，其次小方向的检测手段就是有很多细节去相互嵌套了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lciaLhyGcBziaVSALkyl66TbJo49iamzIDSUbBaicKdKibicVJQiavWTqMdxZ9Q/640?wx_fmt=png&from=appmsg)

  

看雪 ID：九天 666

https://bbs.kanxue.com/user-home-947335.htm

* 本文为看雪论坛优秀文章，由 九天 666 原创，转载请注明来自看雪社区

# 往期推荐

1、[2024 春秋杯网络安全联赛冬季赛 - RE 所有题目 WP](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458590381&idx=2&sn=5afb07b56b06347c4af821ed37826f12&scene=21#wechat_redirect)  

2、[安卓签名校验 - 探讨](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458590330&idx=1&sn=2cebc5b0ae34a171f418d56fa1086982&scene=21#wechat_redirect)

3、[pyd 文件逆向](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458590213&idx=1&sn=8ac33f2b66257296cc0e4c41ae141301&scene=21#wechat_redirect)

4、[AMSI 简介及绕过方法总结](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458590191&idx=2&sn=5303a8192f311d09f00a5e8db0ef8b38&scene=21#wechat_redirect)

5、[2025 HGAME WEEK1 RE WP](https://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458590117&idx=2&sn=3b4b874dabd5b0cd85af1196f318834a&scene=21#wechat_redirect)

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uia4617poZXP96fGaMPXib13V1bJ52yHq9ycD9Zv3WhiaRb2rKV6wghrNa4VyFR2wibBVNfZt3M5IuUiauQGHvxhQrA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lceIkxaY71mzW5gOxdWdic7QibhaLjckpeKza7MqEmCoMJXy5BTFKNFnNg/640?wx_fmt=gif&from=appmsg)

**球分享**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lceIkxaY71mzW5gOxdWdic7QibhaLjckpeKza7MqEmCoMJXy5BTFKNFnNg/640?wx_fmt=gif&from=appmsg)

**球点赞**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lceIkxaY71mzW5gOxdWdic7QibhaLjckpeKza7MqEmCoMJXy5BTFKNFnNg/640?wx_fmt=gif&from=appmsg)

**球在看**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/1UG7KPNHN8FwPmAUPj9VFC18uibWbG9lcsCl0NKVuiczeaZgcXQSicwNPVib5raMJMHc1Xqicn9mUN4FpzVRicibtKp9Q/640?wx_fmt=gif&from=appmsg)

点击阅读原文查看更多