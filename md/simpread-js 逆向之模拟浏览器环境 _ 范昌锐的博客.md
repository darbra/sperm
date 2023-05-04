> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [hexo-fanchangrui.vercel.app](https://hexo-fanchangrui.vercel.app/2022/08/05/js%E9%80%86%E5%90%91%E4%B9%8B%E6%A8%A1%E6%8B%9F%E6%B5%8F%E8%A7%88%E5%99%A8%E7%8E%AF%E5%A2%83/)

> 前段时间在 js 逆向学习过程中，除了一些简单网站的逆向没有使用或检测 dom 和 bom，一般的网站都需要补全浏览器环境，为了更方便进行学习，我找了一些网上的教程结合自己实现了一个简单的浏览器环境框架。

[](#前言 "前言")前言
--------------

前段时间在 js 逆向学习过程中，除了一些简单网站的逆向没有使用或检测 dom 和 bom，一般的网站都需要补全浏览器环境，为了更方便进行学习，我找了一些网上的教程结合自己实现了一个简单的浏览器环境框架。今天我就来讲一下开发的思路和具体怎么使用。

[](#1-介绍 "1 介绍")1 介绍
--------------------

这个框架叫做 catvm，参考了志远大佬的思路实现，完整的代码已经在 github 上开源，如果感兴趣可以访问我的 github 或者点击下面链接查看 [https://github.com/fanchangrui/catvm](https://github.com/fanchangrui/catvm)  
此项目已经被开源安全社区 OSCS 收录并检测安全，请大家放心食用。  
[](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_small)

[![](https://www.oscs1024.com/platform/badge/fanchangrui/catvm.svg?size=small)](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_small)

[**OSCS Status**](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_small)

[![](https://www.oscs1024.com/platform/badge/fanchangrui/catvm.svg?size=large)](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_large)

[**OSCS Status**](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_large)

[](#2-使用方法 "2 使用方法")2 使用方法
--------------------------

首先我们先讲一下使用方法，如何实现和二次开发在第三节中讲到

*   把扣下来的代码放到 code.js 中
*   调试运行 index.js
*   如果报错看错误补充检测的环境
    
    ### [](#没错就是这么简单，如果你逆的网站监测不是那么严格的话，直接调试就可以出结果了，如果还检查了其他地方就要自己补全剩下的环境。 "没错就是这么简单，如果你逆的网站监测不是那么严格的话，直接调试就可以出结果了，如果还检查了其他地方就要自己补全剩下的环境。")没错就是这么简单，如果你逆的网站监测不是那么严格的话，直接调试就可以出结果了，如果还检查了其他地方就要自己补全剩下的环境。
    

[](#3-开发思路 "3 开发思路")3 开发思路
--------------------------

这节我简单讲一下开发的思路和怎么进行补充环境来打造属于你自己的模拟环境，随着你逆向的网站越来越多，补充的环境也会越来越完善，相信你经过一段时间可以应对 80% 的环境检测。

### [](#3-1-目录结构 "3.1 目录结构")3.1 目录结构

src 文件夹下分为 catvm2、tools、code.js、index.js 四个文件夹，其中 catvm2 是 catvm 的源码，tools 是一些工具，code.js 是你扣下来的代码，index.js 是你的调试入口。tools 只是一段方便自动获取浏览器环境的脚本，自己手补的可以不用管这个。  
核心的 catvm2 分为 browser 和 tools，browser 是具体的各个检测对象的实现代码，tools 设计了一些框架缓存，封装的一些工具方法比如保护函数、代理、打印错误等。  

![](https://hexo-fanchangrui.vercel.app/img/805/1.png)

**这是图片**

### [](#3-2-核心思想 "3.2 核心思想")3.2 核心思想

因为 node 和浏览器环境等相似性，我们借用 node 环境等基础来实现，但是现在大多数网站都检测了 node，所以我们引入 vm2 这个包来实现纯净的 v8 环境。  
dom 和 bom 实际上就是在全局挂载了几个对象，我们以前在补充这些时可能直接声明了一个普通对象在加点属性就可以绕过检测，但是实际上这是不严谨的，有些网站会检测在原型上有没有，这也是 js 这个语言的原理。  
现在我们来简单实现如何在原型上添加对象：  
已 document 这个对象为例，不熟悉 js 原型的建议先去学习一下再看

<table><tbody><tr><td></td><td><pre>const Document =function Document()
{

}
Object.defineProperties(Document.prototype,{
    [Symbol.toStringTag]:{
        value:'Document',
        configurable:true,
    }
})
document = {}
document.__proto__ = Document.prototype

</pre></td></tr></tbody></table>

这样就实现了最简单的 document 对象，然后可以在这个对象下面加一下用到了属性和方法比如：

<table><tbody><tr><td></td><td><pre>document.cookie = ''
document.referrer = location.href || ''
document.getElementById = function getElementById(id){
    return null
}
document.addEventListener = function addEventListener(type,listener,useCapture){   
}
document.createElement = function createElement(tagName){
        let tagname = tagName.toLowerCase() + ''
        if(catvm.memory.htmlElements[tagname] == undefined){
            debugger
        }
        return catvm.proxy(catvm.memory.htmlElements[tagname]())

}

</pre></td></tr></tbody></table>

但是实际上 v8 对这个对象做了很多限制，我们可以封装保护函数、代理等方法来加强, 这个具体再 tools 讲。如果一个对象在浏览器中无法看具体方法，我们要在定义的时候抛出 getter 错误。具体每个对象的实现参考 js 官方文档 mdn [https://developer.mozilla.org/zh-CN/](https://developer.mozilla.org/zh-CN/)

<table><tbody><tr><td></td><td><pre>throw new TypeError('Illegal constructor')

</pre></td></tr></tbody></table>

### [](#3-3-tools工具实现 "3.3 tools工具实现")3.3 tools 工具实现

*   memory.js: 内存缓存，用来缓存一我们自己注入的一些对象，不污染全局环境，即使日后被检测了改一下名字也能逃过检测。  
    
    ![](https://hexo-fanchangrui.vercel.app/img/805/2.png)
    
    **这是图片**
    
*   proxy.js: 代理函数，给我们自己模拟的对象加上后就可以查看到这个对象下哪个属性或方法被检测到了，方便我们补环境。  
    
    ![](https://hexo-fanchangrui.vercel.app/img/805/3.png)
    
    **这是图片**
    
*   safefunction.js: 保护函数，因为 js 每个对象的原型上都有 toString 方法，所以我们自己重写一个 toString 方法的来规避检测。尽量每一个函数都用 safefunction 保护一下。  
    
    ![](https://hexo-fanchangrui.vercel.app/img/805/4.png)
    
    **这是图片**
    
*   node.js: 整合工具文件，把上面几个文件合并成一个文件，方便我们使用。

### [](#3-4-注意点 "3.4 注意点")3.4 注意点

*   catcm.node.js 和 tools 下的 node.js 都是整合的功能，每加入一个浏览器属性对象或者工具都要放入. node.js 结尾的文件中，当然你也可以写个遍历函数循环读取加入。
*   本框架用的是 es6 模块语法，如果要使用 node 老版本的 commonjs 语法的请在 pack.json 中删除 “type”，这样自动使用 commonjs 语法进行编译。

[](#后记 "后记")后记
--------------

本框架只补充了浏览器的几个基础对象和开放接口，很有可能无法直接使用，建议有一定基础的开发者研究并不断完善，最后说一句：逆向虽好，但切记不要利用这个去做违法的事。