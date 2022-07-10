> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NTEwNjcxNg==&mid=2247483992&idx=1&sn=45ad7614454bdc7b5a259323bb1a3a9d&chksm=eaa3262bddd4af3d949f3e27b71c2eefac4b4323327b2caa3b30a70cee16a0dd8ca4b568e243&mpshare=1&scene=1&srcid=0710lucIUjwFO3qSq72NK5EH&sharer_sharetime=1657467763336&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.6.99102&platform=mac#rd)

IO 重定向是一个文件重定位到另外一个文件的操作。当一个文件被 IO 重定向后，它发生打开，读写这些行为，就会变成另外一个文件的打开，读写。

![](https://mmbiz.qpic.cn/mmbiz_png/FteuUCDniaicxL3FVKnx1gHDyqGWN3ibJOhhqsOdibtAEaDJkr1FxIpU9ECibDS2thlDpAZA2LENsibJQpmV3WXl1f6A/640?wx_fmt=png)

IO 重定向多用于过风控，改机，多开等场景中。

实现
--

作为实现重定向的一方，有许多实现方式，下面的例子是用 frida 脚本 hook libc.so 下的 open 函数，在 open 函数被调用的时候，根据文件名来判断是否对打开的重定向，最后将文件描述符返回。

```
function strcmp(str1,str2){
   let ret = -1;
   let i = 0;
for(i = 0; ;i++){
let ch1 = str1.add(i).readU8();
let ch2 = str2.add(i).readU8();

if(ch1 == 0 && ch2 == 0){
    ret = 0;
break;
}

if(ch1 != ch2){
    ret = -1;
    break;
}

}

return ret;
}


function redirect(src,dest){
   const openPtr = Module.getExportByName('libc.so', 'open');
   const open = new NativeFunction(openPtr, 'int', ['pointer', 'int']);
   Interceptor.replace(openPtr, new NativeCallback((pathPtr, flags) => {
       const path = pathPtr.readUtf8String();
       
       const originPath = Memory.allocUtf8String(src)
       let currentPath = Memory.allocUtf8String(path)
       let fd = -1
       if(strcmp(currentPath,originPath) == 0){
          console.warn('redirect file "' + path + '" to "' + dest + '"');
          let targetPath = Memory.allocUtf8String(dest)
          fd = open(targetPath, flags);
      }
       else{
           console.log('open file "' + path + '"');
           fd = open(pathPtr, flags);
      }
       return fd;
  }, 'int', ['pointer', 'int']));

}

redirect("/proc/cpuinfo","/data/data/com.luoye.fileredirectdetect/cpuinfo")

```

经过 hook 之后，当 app 打开 / proc/cpuinfo，文件路径就被重定向到了自定义的文件路径。open 函数返回的是新的文件的文件描述符。

对抗
--

既然 hook 之后返回的是新的文件的文件描述符，那么，对抗方就可以打开需要检测的文件，然后调用 readlink 函数反推文件路径，如果 readlink 返回的文件路径和打开的文件不一致，那么说明打开的文件被重定位到其他文件去了。

```
/**
* 检查文件是否被重定位
* @src_path 要检查的文件
* @new_path 被重定位到的路径
* @return 0 没有重定位；1 被重定位
*/
int checkFileRedirect(const char *src_path,char *new_path,size_t max_len) {
   int src_fd = open(src_path,O_RDONLY);
   if(src_fd < 0){
       return 0;
  }
   int ret = 0;
   char link_path[128] = {0};
   snprintf(link_path, sizeof(link_path), "/proc/%d/fd/%d", getpid(), src_fd);
   char fd_path[256] = {0};
   int link = readlink(link_path, fd_path, sizeof(fd_path));

   if(link < 0){
       return 0;
  }

   size_t len = strnlen(fd_path,256);

   if(len && strncmp(fd_path,src_path,256) != 0) {
       strncpy(new_path, fd_path,max_len);

       ret = 1;
  }

   if(src_fd > 0) {
       close(src_fd);
  }

   return ret;
}

```

下图是在多开软件中检测到的路径重定向结果：

![](https://mmbiz.qpic.cn/mmbiz_png/FteuUCDniaicxL3FVKnx1gHDyqGWN3ibJOhQmJ3jhMYRxgOBn0k0dCfdZCPD7ia3jQYNLsnA4ZjE06pbicsbS1oVWbQ/640?wx_fmt=png)

当然这只是提供的一个思路，文件重定位可以有其他实现，对抗也可以有其他实现，想要做得更好，需要更多的想象力。