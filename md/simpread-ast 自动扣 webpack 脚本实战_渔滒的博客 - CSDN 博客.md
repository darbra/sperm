> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zjq592767809/article/details/122355530?spm=1001.2014.3001.5501)

### ast 自动扣 [webpack](https://so.csdn.net/so/search?q=webpack&spm=1001.2101.3001.7020) 脚本实战

*   *   [webpack 是什么](#webpack_2)
    *   [前置阅读](#_8)
    *   [webpack 自动扣代码脚本](#webpack_14)
    *   [实战内容](#_44)

webpack 是什么
-----------

webpack 是一个现代 JavaScript 应用程序的静态模块打包器 (module bundler)，通俗来讲，我的理解就是把你在加解密过程中要调用的模块打包成为一个 JS 文件，然后引入它，自动展现出你要引入的资源。可能我这里说的不是很清楚，可以推荐一篇文章：https://www.cnblogs.com/Eeyhan/p/10584014.html

![](https://img-blog.csdnimg.cn/271912f4f9a34d8a851ba0d08e037276.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

前置阅读
----

本文章主要是讲述脚本的使用方式，而 js 的分析过程在本文中会省略。文章中实战的来源于下方网址，请先阅读下方网站中的分析过程，再尝试阅读本文，会更加清晰。

阅读地址：[用 3 个实战案例带你理解 webpack](https://mp.weixin.qq.com/s/c4nU1oSGvH1pJHPC8bsZMQ)

webpack 自动扣代码脚本
---------------

项目地址: [渔滒 / webpack_ast](https://gitcode.net/zjq592767809/webpack_ast)

使用命令行的方式

node webpack_mixer.txt -l runtime.62249a5.js -m app.597640f.js -o webout.js

参数说明：

-l 加载器的 js 路径

加载器的 js 特征：

1. 以自执行函数开头

2. 定义导出函数，类似 return e[n].call(r.exports, r, r.exports, d), r.l = !0, r.exports

3. 为导出函数添加多个方法，类似 d.e，d.m，d.n 等等

-m 函数模块的 js 路径

函数模块的 js 特征：

1. 一般以 (window.webpackJsonp 开头

-o 输入结果的 js 路径

备注：如果 js 本身有检测等，需要自行补头或者其他处理

实战内容
----

文章中的第一个网站是猿人学的 webpack，因为这个不是一个常规的，所以这里不适用，直接从第二个开始

从文章中可以知道，加密的方法来自于 home.min.js 这个 js 文件，那么直接将这个文件下载下来，这个文件就是一个加载器

![](https://img-blog.csdnimg.cn/d4f3e8b425a84adfbd79a4aa39afd16d.png#pic_center)

下载完成后，把脚本文件 webpack_mixer.js 放到同一目录，并执行

```
node webpack_mixer.js -l home.min.js -o webpack_out.js

```

可以得到 webpack_out.js 这个已经扣取完成的文件，自己在同目录创建一个 test.js 的测试文件，直接导入文件

![](https://img-blog.csdnimg.cn/9a5a63fe970947c2b54438cc86aaf91f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

可以正常获取到所有的导出函数

![](https://img-blog.csdnimg.cn/b50d3648c7004f8d82b5143ef50c9b73.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

当调用加密的时候，缺少环境。那么直接导入 jsdom

![](https://img-blog.csdnimg.cn/0bb84c645f974e7abbbf2230de3c15b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
继续运行发现 ASN1 未定义，主要是因为浏览器和 node 环境不同导致的，在下面三个位置前面加上 window 来调用就可以

![](https://img-blog.csdnimg.cn/1340435cd7e94d32809920e3d057d609.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

再次运行得到加密结果

![](https://img-blog.csdnimg.cn/954278ff05564ed8af6bf5d6ce476570.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
继续来到第三个实战

根据文章中的分析，密码的加密调用到了 login.5170f665.js 这个文件，但是这个文件只是一个模块，还要找到它的加载器 app.f24d08e9.js 一起下载下来

```
node webpack_mixer.js -l app.f24d08e9.js -m login.5170f665.js -o webpack_out.js

```

尝试对密码进行加密

![](https://img-blog.csdnimg.cn/bd468bfa66c8405e99e1f8e1b4e28ffb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

出现这个错误，那么一定就是还是缺少需要的模块，经过查找是缺少了 chunk-vendors.bb13f90f.js 这个模块，那么继续下载这个模块放到一起

```
node webpack_mixer.js -l app.f24d08e9.js -m login.5170f665.js -m chunk-vendors.bb13f90f.js -o webpack_out.js

```

![](https://img-blog.csdnimg.cn/8d1f2ddaa9eb4b8bbf31dc9f4d591cd6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
此时密码加密已经完成，还差一个请求体的加密，发现是调用另外一个文件 DCSAPPClientAPI-0.0.0.7.js 进行加密的，也下载下来

![](https://img-blog.csdnimg.cn/61280c0da4df4df1b7b100da01a6476e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5riU5ruS,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

通过导入文件，就可以实现全部的加密了，至此实战完成