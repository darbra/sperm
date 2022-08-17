> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gL-83inHhCkqaln4zRe7zA)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

elf 虽说学了很久了但是只有对于理论的了解, 在实战部分还是去使用别人的脚本去修复 elf 结构, 索性心血来潮分析下 elf 的修复的原理及其实现的代码, 前人栽树后人乘凉, 已经有前人根据 ThomasKing 的思路来实现了修复 section 的代码, 本文就是一个读书笔记的形式来总结。

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif)

  

  

  

  

 目录

  

⊙一. 仅处理 so 文件头的

⊙二. 无 section 信息的

⊙三. 读取 elf_header 和 program_header_table 到 elf 结构中

⊙四. 修复 elf 结构

⊙总结

一. 仅处理 so 文件头的

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcp8GJOAvLIswdxUZoP8XSQstBHhBkBw5lEpoDAYMY6GgibNUP2DCFiaTGQ/640?wx_fmt=png)

这里作者所讲的一种情况是, 在 so 文件和 apk 集成的时候直接删除了节的数量或者把节表中能够显示字符串的表的 index 给清除了, 这种情况按照上面的修复方案就可以解决, 这种情况俺在逆向工程中还没遇到过, 不过不失一种为了保护 so 的方法。

二. 无 section 信息的

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcp5dpfibX40CMibalQFCX9icbv8HknrzMdRPgiaHicc9M9hcpYsUL6nxPgBeg/640?wx_fmt=png)

读了上文作者的话, 感到豁然开朗, 上文的情况就是对应在日常逆向分析的过程中, 需要将有一些被加密或者被加壳的 so, 从内存中 dump 出来的情景, 但是我们日常逆向过程中, dump 出来拖入到 ida 中打开节表会发现除了一个 load 之外什么也发现不了, 这个时候就需要修复了, 经过我去查阅 elf 的结构, 就算没有节但是 program_header_table 有一些信息可以帮助我们修复节。

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpfkwFNpkxk3ruxRqrKQjKSJfWt1GGBxaPGk5ic8bsPrDPjFd1wChQ9fA/640?wx_fmt=png)

上面的七步就是整个过程, 为了从理解上和代码层面能更能透透彻彻的理解这个修复过程怎么实现的。接下来源码解析环节解析前辈们的思路

三. 读取 elf_header 和 program_header_table 到 elf 结构中

```
FILE* fr = NULL;
  FILE* fw = NULL;
  long flen = 0,result = 0;
  char* buffer = NULL;
  Elf32_Ehdr* pehdr = NULL;
  Elf32_Phdr* pphdr = NULL;
  if (argc < 2) {
    printf("less args\n");
    return;
  }
  fr = fopen(argv[1],"rb");
  if(fr == NULL) {
    printf("Open failed: \n");
    goto error;
  }
  flen = get_file_len(fr);
  buffer = (char*)malloc(sizeof(char)*flen);
  memset(buffer, 0, flen);
  if (buffer == NULL) {
    printf("Malloc error\n");
    goto error;
  }
  result = fread (buffer,1,flen,fr);
  if (result != flen) {
    printf("Reading error\n");
    goto error;
  }
  fw = fopen("fix.so","wb");
  if(fw == NULL) {
    printf("Open failed: fix.so\n");
    goto error;
  }
  printf("----------[get_elf_header]----------\n");
  pehdr = (Elf32_Ehdr*)malloc(sizeof(Elf32_Ehdr));
  get_elf_header(buffer, &pehdr);
  printf("ehdr->e_type=\t\t%x\n", pehdr->e_type);
  printf("ehdr->e_machine=\t%x\n", pehdr->e_machine);
  printf("ehdr->e_version=\t%x\n", pehdr->e_version);
  printf("ehdr->e_entry=\t\t%x\n", pehdr->e_entry);
  printf("ehdr->e_phoff=\t\t%x\n", pehdr->e_phoff);
  printf("ehdr->e_shoff=\t\t%x\n", pehdr->e_shoff);
  printf("ehdr->e_flags=\t\t%x\n", pehdr->e_flags);
  printf("ehdr->e_ehsize=\t\t%x\n", pehdr->e_ehsize);
  printf("ehdr->e_phentsize=\t%x\n", pehdr->e_phentsize);
  printf("ehdr->e_phnum=\t\t%x\n", pehdr->e_phnum);
  printf("ehdr->e_shentsize=\t%x\n", pehdr->e_shentsize);
  printf("ehdr->e_shnum=\t\t%x\n", pehdr->e_shnum);
  printf("ehdr->e_shstrndx=\t%x\n", pehdr->e_shstrndx);
  printf("\n");
  printf("----------[get_program_table]----------\n");
  pphdr = (Elf32_Phdr*)malloc(pehdr->e_phentsize * pehdr->e_phnum);
  get_program_table(*pehdr, buffer, &pphdr);
  printf("phdr->e_type=\t%x\n", pphdr->p_type);
  printf("phdr->p_offset=\t%x\n", pphdr->p_offset);
  printf("phdr->p_vaddr=\t%x\n", pphdr->p_vaddr);
  printf("phdr->p_paddr=\t%x\n", pphdr->p_paddr);
  printf("phdr->p_filesz=\t%x\n", pphdr->p_filesz);
  printf("phdr->p_memsz=\t%x\n", pphdr->p_memsz);
  printf("phdr->p_flags=\t%x\n", pphdr->p_flags);
  printf("\n");
  printf("----------[fix_section_table]----------\n");

```

我们看下把待修复的 so 送入到 Elf32_Ehdr 类型的结构体中 (从二进制解析出 elf_header)

```
void get_elf_header(char* buffer, Elf32_Ehdr** pehdr) {
  int header_len = sizeof(Elf32_Ehdr);
  memset(*pehdr, 0, header_len);
  memcpy(*pehdr, (void*)buffer, header_len);
}

```

其中 Elf32_Ehdr 对应的是我们从 010editor 中看到的

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpklkTdmhKHkUibzQkKCSkqoYRAPvQCfH9h1uFYUEVWE7T1zMdvaHkWnw/640?wx_fmt=png)

对应 elf.h 源码中

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpHibqpmSJCpibkzIXbJHdGtCiauz7spjROu9vGjQn6ELHhvMFa6yHDGf3g/640?wx_fmt=png)

我们看下把待修复的 so 送入到 Elf32_Ehdr 类型的结构体中 (从二进制解析出 program_table)

```
void get_program_table(Elf32_Ehdr ehdr, char* buffer, Elf32_Phdr** pphdr) {
    //这个地方
  int ph_size = ehdr.e_phentsize;
  int ph_num = ehdr.e_phnum;
  memset(*pphdr, 0, ph_size * ph_num);
  memcpy(*pphdr, buffer + ehdr.e_phoff, ph_size * ph_num);
}

```

其中 Elf32_Ehdr 对应的是我们从 010editor 中看到的

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpu0fSxztA3kcfzCB3OFUicEsDiauI4KJTrZmcIUbk3RTqN5WYcP4a7nHA/640?wx_fmt=png)

对应 elf.h 源码中

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpH3eLdAC07GDfje5FjPmkVq36ic1kRKQtmH9zUXyQHqROXIwF2MqvlyQ/640?wx_fmt=png)

有了 elf 头和 program_table 我们就可以修复 elf 结构了

四. 修复 elf 结构

4.1 修复节中的. bss,.dynamic .ARM.exidx

4.1.1 修复. bss

修复这个结构之前我们首先知道这个节在 010editor 是怎么显示的

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpYgwHuFcJlG2AOCt3LzzSpv3d6Y8iaullKibonBibJtZ3F2RRfR7l0maWA/640?wx_fmt=png)

然后在 program 中那个结构才能去填充这个节 : PT_LOAD, 下方就是前辈实现的代码

我们逐行分析下: 当然需要前置知识  
这个是程序表头的一些结构, 我们最终用这个程序表头的一些数据来填充. bss 这个节

```
typedef struct elf32_phdr{ 
    Elf32_Word p_type; //段的类型 
    Elf32_Off p_offset; //段的位置相对于文件开始处的偏移 
    Elf32_Addr p_vaddr; //段在内存中的首字节地址 
    Elf32_Addr p_paddr;//段的物理地址 
    Elf32_Word p_filesz;//段在文件映像中的字节数 
    Elf32_Word p_memsz;//段在内存映像中的字节数 
    Elf32_Word p_flags;//段的标记 
    Elf32_Word p_align;，/段在内存中的对齐标记 
)Elf32_Phdr;

```

```
typedef struct
{
  Elf32_Word    sh_name;        /* Section name (string tbl index) */
  Elf32_Word    sh_type;        /* Section type */
  Elf32_Word    sh_flags;       /* Section flags */
  Elf32_Addr    sh_addr;        /* Section virtual addr at execution */
  Elf32_Off     sh_offset;      /* Section file offset */
  Elf32_Word    sh_size;        /* Section size in bytes */
  Elf32_Word    sh_link;        /* Link to another section */
  Elf32_Word    sh_info;        /* Additional section information */
  Elf32_Word    sh_addralign;       /* Section alignment */
  Elf32_Word    sh_entsize;     /* Entry size if section holds table */
} Elf32_Shdr;

```

```
for(; i < ph_num; i++) {
            //判断程序表头是PT_LOAD类型
        if (phdr[i].p_type == : PT_LOAD) {
      if (phdr[i].p_vaddr > 0x0) {
        printf("get PT_LOAD\n");
        load = phdr[i];
        //算出节名的偏移
                shdr[BSS].sh_name = (Elf64_Word)(strstr(str, ".bss") - str);
                //节的类型:正常用010edito可以看到,在elf的文件中
        shdr[BSS].sh_type = SHT_NOBITS;
        //010中显示的为SF32_Alloc_Exec,但是作者写的这个(IDA解析无错误,暂时存疑)
        shdr[BSS].sh_flags = SHF_WRITE | SHF_ALLOC;
        //得出节的映射到虚拟空间的地址
        shdr[BSS].sh_addr =  phdr[i].p_vaddr + phdr[i].p_filesz;
        //节在文件中的偏移就是在虚拟中的地址-0x1000;
        shdr[BSS].sh_offset = shdr[BSS].sh_addr - 0x1000;
        shdr[BSS].sh_addralign = 1;
        continue;
      }
    }
}

```

修复 dynamic .ARM.exidx 和上面的思路一样不再赘述

```
  if(phdr[i].p_type == PT_DYNAMIC) {
      printf("get PT_DYNAMIC\n");
      shdr[DYNAMIC].sh_name = strstr(str, ".dynamic") - str;
      shdr[DYNAMIC].sh_type = SHT_DYNAMIC;
      shdr[DYNAMIC].sh_flags = SHF_WRITE | SHF_ALLOC;
      shdr[DYNAMIC].sh_addr = phdr[i].p_vaddr;
      shdr[DYNAMIC].sh_offset = phdr[i].p_offset + 0x1000;
      shdr[DYNAMIC].sh_size = phdr[i].p_filesz;
      shdr[DYNAMIC].sh_link = 2;
      shdr[DYNAMIC].sh_info = 0;
      shdr[DYNAMIC].sh_addralign = 4;
      shdr[DYNAMIC].sh_entsize = 8;
      dyn_size = phdr[i].p_filesz;
      // 之前使用  phdr[i].p_offset 字段获取 dyn_off ，是有问题的
      // 从内存中dump下来的数据 应使用vaddr字段
        dyn_off = phdr[i].p_vaddr;
        continue;
    }
    if(phdr[i].p_type == PT_LOPROC || phdr[i].p_type == PT_LOPROC + 1) {
      printf("get PT_LOPROC\n");
      shdr[ARMEXIDX].sh_name = strstr(str, ".ARM.exidx") - str;
      shdr[ARMEXIDX].sh_type = SHT_LOPROC;
      shdr[ARMEXIDX].sh_flags = SHF_ALLOC;
      shdr[ARMEXIDX].sh_addr = phdr[i].p_vaddr;
      shdr[ARMEXIDX].sh_offset = phdr[i].p_offset;
      shdr[ARMEXIDX].sh_size = phdr[i].p_filesz;
      shdr[ARMEXIDX].sh_link = 7;
      shdr[ARMEXIDX].sh_info = 0;
      shdr[ARMEXIDX].sh_addralign = 4;
      shdr[ARMEXIDX].sh_entsize = 8;
      continue;
    }

```

4.2 修复. dynsym .hash .rel.dyn .rel.plt .fini_array

这些都是利用 dynamic segment 来修复的 代码如下

```
for (; i < dyn_size / sizeof(Elf32_Dyn); i++) {
    switch(dyn[i].d_tag) {
      case DT_SYMTAB:
        printf("get DT_SYMTAB\n");
        shdr[DYNSYM].sh_name = strstr(str, ".dynsym") - str;
        shdr[DYNSYM].sh_type = SHT_DYNSYM;
        shdr[DYNSYM].sh_flags = SHF_ALLOC;
        shdr[DYNSYM].sh_addr = dyn[i].d_un.d_ptr;
        shdr[DYNSYM].sh_offset = dyn[i].d_un.d_ptr;
        shdr[DYNSYM].sh_link = 2;
        shdr[DYNSYM].sh_info = 1;
        shdr[DYNSYM].sh_addralign = 4;
        shdr[DYNSYM].sh_entsize = 16;
        break;
      case DT_STRTAB:
        printf("get DT_STRTAB\n");
        shdr[DYNSTR].sh_name = strstr(str, ".dynstr") - str;
        shdr[DYNSTR].sh_type = SHT_STRTAB;
        shdr[DYNSTR].sh_flags = SHF_ALLOC;
        shdr[DYNSTR].sh_offset = dyn[i].d_un.d_ptr;
        shdr[DYNSTR].sh_addr = dyn[i].d_un.d_ptr;
        shdr[DYNSTR].sh_addralign = 1;
        shdr[DYNSTR].sh_entsize = 0;
        break;
      case DT_HASH:
        printf("get DT_HASH\n");
        shdr[HASH].sh_name = strstr(str, ".hash") - str;
        shdr[HASH].sh_type = SHT_HASH;
        shdr[HASH].sh_flags = SHF_ALLOC;
        shdr[HASH].sh_addr = dyn[i].d_un.d_ptr;
        shdr[HASH].sh_offset = dyn[i].d_un.d_ptr;
        memcpy(&nbucket, buffer + shdr[HASH].sh_offset, 4);
        memcpy(&nchain, buffer + shdr[HASH].sh_offset + 4, 4);
        shdr[HASH].sh_size = (nbucket + nchain + 2) * sizeof(int);
        shdr[HASH].sh_link = 4;
        shdr[HASH].sh_info = 1;
        shdr[HASH].sh_addralign = 4;
        shdr[HASH].sh_entsize = 4;
        break;
      case DT_REL:
        printf("get DT_REL\n");
        shdr[RELDYN].sh_name = strstr(str, ".rel.dyn") - str;
        shdr[RELDYN].sh_type = SHT_REL;
        shdr[RELDYN].sh_flags = SHF_ALLOC;
        shdr[RELDYN].sh_addr = dyn[i].d_un.d_ptr;
        shdr[RELDYN].sh_offset = dyn[i].d_un.d_ptr;
        shdr[RELDYN].sh_link = 4;
        shdr[RELDYN].sh_info = 0;
        shdr[RELDYN].sh_addralign = 4;
        shdr[RELDYN].sh_entsize = 8;
        break;
      case DT_JMPREL:
        printf("get DT_JMPREL\n");
        shdr[RELPLT].sh_name = strstr(str, ".rel.plt") - str;
        shdr[RELPLT].sh_type = SHT_REL;
        shdr[RELPLT].sh_flags = SHF_ALLOC;
        shdr[RELPLT].sh_addr = dyn[i].d_un.d_ptr;
        shdr[RELPLT].sh_offset = dyn[i].d_un.d_ptr;
        shdr[RELPLT].sh_link = 1;
        shdr[RELPLT].sh_info = 6;
        shdr[RELPLT].sh_addralign = 4;
        shdr[RELPLT].sh_entsize = 8;
        break;
      case DT_PLTRELSZ:
        printf("get DT_PLTRELSZ\n");
        shdr[RELPLT].sh_size = dyn[i].d_un.d_val;
        break;
      case DT_FINI:
        printf("get DT_FINI\n");
        shdr[FINIARRAY].sh_name = strstr(str, ".fini_array") - str;
        shdr[FINIARRAY].sh_type = 15;
        shdr[FINIARRAY].sh_flags = SHF_WRITE | SHF_ALLOC;
        shdr[FINIARRAY].sh_offset = dyn[i].d_un.d_ptr - 0x1000;
        shdr[FINIARRAY].sh_addr = dyn[i].d_un.d_ptr;
        shdr[FINIARRAY].sh_addralign = 4;
        shdr[FINIARRAY].sh_entsize = 0;
        break;
      case DT_INIT:
        printf("get DT_INIT\n");
        shdr[INITARRAY].sh_name = strstr(str, ".init_array") - str;
        shdr[INITARRAY].sh_type = 14;
        shdr[INITARRAY].sh_flags = SHF_WRITE | SHF_ALLOC;
        shdr[INITARRAY].sh_offset = dyn[i].d_un.d_ptr - 0x1000;
        shdr[INITARRAY].sh_addr = dyn[i].d_un.d_ptr;
        shdr[INITARRAY].sh_addralign = 4;
        shdr[INITARRAY].sh_entsize = 0;
        break;
      case DT_RELSZ:
        printf("get DT_RELSZ\n");
        shdr[RELDYN].sh_size = dyn[i].d_un.d_val;
        break;
      case DT_STRSZ:
        shdr[DYNSTR].sh_size = dyn[i].d_un.d_val;
        break;
      case DT_PLTGOT:
        printf("get DT_PLTGOT\n");
        shdr[GOT].sh_name = strstr(str, ".got") - str;
        shdr[GOT].sh_type = SHT_PROGBITS;
        shdr[GOT].sh_flags = SHF_WRITE | SHF_ALLOC; 
        shdr[GOT].sh_addr = shdr[DYNAMIC].sh_addr + shdr[DYNAMIC].sh_size;
        shdr[GOT].sh_offset = shdr[GOT].sh_addr - 0x1000;
        shdr[GOT].sh_size = dyn[i].d_un.d_ptr;
        shdr[GOT].sh_addralign = 4;
        break;
    }
  }
  shdr[GOT].sh_size = shdr[GOT].sh_size + 4 * (shdr[RELPLT].sh_size) / sizeof(Elf32_Rel) + 3 * sizeof(int) - shdr[GOT].sh_addr;
  //STRTAB地址 - SYMTAB地址 = SYMTAB大小
  shdr[DYNSYM].sh_size = shdr[DYNSTR].sh_addr - shdr[DYNSYM].sh_addr;
  //FINIARRAY大小 = INITARRAY的虚拟地址 - FINIARRAY的虚拟地址
  shdr[FINIARRAY].sh_size = shdr[INITARRAY].sh_addr - shdr[FINIARRAY].sh_addr;
  //INITARRAY大小 = DYNAMIC的虚拟地址 - INITARRAY的虚拟地址
  shdr[INITARRAY].sh_size = shdr[DYNAMIC].sh_addr - shdr[INITARRAY].sh_addr;

```

4.3 修复 plt text data shstrtab

```
  shdr[PLT].sh_name = strstr(str, ".plt") - str;
  shdr[PLT].sh_type = SHT_PROGBITS;
  shdr[PLT].sh_flags = SHF_ALLOC | SHF_EXECINSTR;
  shdr[PLT].sh_addr = shdr[RELPLT].sh_addr + shdr[RELPLT].sh_size;
  shdr[PLT].sh_offset = shdr[PLT].sh_addr;
  shdr[PLT].sh_size = (20 + 12 * (shdr[RELPLT].sh_size) / sizeof(Elf32_Rel));
  shdr[PLT].sh_addralign = 4;

```

上文的实现思路通过

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcpyFvFBNbdicoBO2hz0QdKdr4xZb7RNS58rqib6krW9j85tHI3UhgkmODA/640?wx_fmt=png)

```
//text是由plt推出来的
  shdr[TEXT].sh_name = strstr(str, ".text") - str;
  shdr[TEXT].sh_type = SHT_PROGBITS;
  shdr[TEXT].sh_flags = SHF_ALLOC | SHF_EXECINSTR;
  shdr[TEXT].sh_addr = shdr[PLT].sh_addr + shdr[PLT].sh_size;
  shdr[TEXT].sh_offset = shdr[TEXT].sh_addr;
  shdr[TEXT].sh_size = shdr[ARMEXIDX].sh_addr - shdr[TEXT].sh_addr;

```

```
shdr[DATA].sh_name = strstr(str, ".data") - str;
  shdr[DATA].sh_type = SHT_PROGBITS;
  shdr[DATA].sh_flags = SHF_WRITE | SHF_ALLOC;
  shdr[DATA].sh_addr = shdr[GOT].sh_addr + shdr[GOT].sh_size;
  shdr[DATA].sh_offset = shdr[DATA].sh_addr - 0x1000;
  shdr[DATA].sh_size = load.p_vaddr + load.p_filesz - shdr[DATA].sh_addr;
  shdr[DATA].sh_addralign = 4;
  shdr[GOT].sh_size = shdr[DATA].sh_offset - shdr[GOT].sh_offset;

```

上文的思路通过

![](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1KSrNyppgtPNlfQ8utIzcp01eSznqibx50UcjgRpWvpWHZlYGk3bZlmSjWumv2frKtR2QgA3BPk2w/640?wx_fmt=png)

然后最后一步修复字符串表

```
  shdr[STRTAB].sh_name = strstr(str, ".shstrtab") - str;
  shdr[STRTAB].sh_type = SHT_STRTAB;
  shdr[STRTAB].sh_flags = SHT_NULL;
  shdr[STRTAB].sh_addr = 0;
  shdr[STRTAB].sh_offset = shdr[BSS].sh_addr - 0x1000;
  shdr[STRTAB].sh_size = strlen(str) + 1;
  shdr[STRTAB].sh_addralign = 1;

```

五. 总结

  
分析整个代码和思路, 收获甚大, 熟悉了 elf 的结构和如何修复怎么修复, 感谢大佬的思路和实现代码才能让笔者如此顺利的走完整个流程, 拒绝当工具侠, 要知其所以然。  

源码来源:https://github.com/WangYinuo/FixElfSection  
思路来源:https://bbs.pediy.com/thread-192874.htm

我是 BestToYou, 分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序, 文中若有错误的地方, 恳请大家联系我批评指正。  

![](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif)