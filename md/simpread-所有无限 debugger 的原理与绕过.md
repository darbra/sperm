> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hpJ3qSaYI4yFJU0LWsm23A)

一、无限 debugger 的原理与绕过
====================

> debugger 是 JavaScript 中定义的一个专门用于断点调试的关键字，只要遇到它，JavaScript 的执行便会在此处中断，进入调试模式。有了 debugger 这个关键字，我们就可以非常方便地对 JavaScript 代码进行调试，比如使用 JavaScript Hook 时，我们可以加入 debugger 关键字，使其在关键的位置停下来，以便查找逆向突破口。但有时候 debugger 会被网站开发者利用，使其成为阻挠我们正常调试的拦路虎

1. 无限 debugger 的产生方法：
---------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQZObA5zzcRdHMGKKbcd4EJRqfOY2Mxibqlqkx5allG1szVvv958icev0w/640?wx_fmt=png&from=appmsg#imgIndex=0)这几个是最基础的 debugger，其中 **Function** 和 **eval** 创造的 debugger 会进入虚拟环境中

> 在 Chrome 的开发者工具中，你可能会看到一些以 VM 开头的 JavaScript 文件（如 VM1057）。出现的原因：动态执行的 JavaScript 代码。比如通过 eval 函数或者 new Function 方法，Chrome 浏览器会创建一个 VM 文件来展示这段临时执行的代码。比如某个网页因为反爬虫，动态生成了 debugger，这些断点并没有直接写在服务器上的原始 JavaScript 文件中，而是在某些 JavaScript 代码的执行过程中被生成，并因此触发 debugger。

2. 下面来讲述形成的原理
-------------

### 2.1Function:

> JavaScript 中的 Function 是一个对象，表示可执行的代码块，允许传入参数并返回值，用于实现代码的重用和模块化。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQK7bMxCbhgqPN8pOO40wibOyQicibwsSE8f0bckp3WmQCQmIqX6pALxHIA/640?wx_fmt=png&from=appmsg#imgIndex=1)

### 2.2eval:

> JavaScript 中的 eval 是一个函数，用于将字符串作为代码执行

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQkHp6yQP9kIUEibiavJROVTABZicESmJvdKsrGTblfOA46djYG5s3oefAg/640?wx_fmt=png&from=appmsg#imgIndex=2)

### 2.3Function.constructor().call()

> Function.constructor('debugger').call('action')
> 
> 这种方式的工作原理：
> 
> *   Function.constructor 实际上就是 Function 本身
>     
> *   相当于 new Function('debugger')，创建一个包含 debugger 语句的新函数
>     
> *   .call('action') 调用这个新创建的函数，'action' 作为 this 值传入
>     

### 2.4 (function(){return !![];...}

> (function(){return !![];}'constructor''call')
> 
> 这是一种混淆版本：
> 
> *   function(){return !![]} 创建一个返回 true 的匿名函数
>     
> *   ['constructor'] 使用方括号语法访问 constructor 属性
>     
> *   本质上还是利用 Function.constructor 创建新函数
>     
> *   使用方括号语法 ['call'] 来调用
>     

### 2.5 eval('(function (){}...)

> [图片]
> 
> 这是最复杂的版本：
> 
> *   使用 eval 执行字符串形式的代码
>     
> *   字符串中的代码逻辑与第三种方式类似
>     
> *   额外的 eval 包装增加了代码的混淆程度
>     
> 
> 那我们基本就只要过掉这几个生成方式就行

我们基于此可以写一个**万能脚本**，首先先看几个案例热热身：

案例网站一：
======

_https://www.sojson.com/jsobfuscator.html_

打开 f12![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQlx8MXoHNOG2Lsrdu03eTOibesiclYh1FpM0IKKiccRvrUNOGZ6dnLLyBw/640?wx_fmt=png&from=appmsg#imgIndex=3) 往上跟栈![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQRYZchseaJYEZo8na3Jl7RAK5tMWYBrktiaopwBmYs9dW158yaibCryTA/640?wx_fmt=png&from=appmsg#imgIndex=4)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQQD1WUyescRVfkvNnsibZjQIxiaGiaiamodUvIdNo5ygicz8GforH1Pt4djw/640?wx_fmt=png&from=appmsg#imgIndex=5)代码特征：

> 代码经过了高度混淆
> 
> debugger 语句被拆分和重组
> 
> 使用了 arguments[0] 动态构造
> 
> 通过函数调用链和条件判断来触发 debugger

具体实现方式：![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQBia6YzzUtudBV4aglyCGaGwCI3HRuzNcTSibvgatQTicp9v9KEewv91ng/640?wx_fmt=png&from=appmsg#imgIndex=6)绕过方法：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQZ0F3XeqtOAj9woQ7ibGV3ZOOTpTorFn7REaKKqialHWlHIzdLq6tHS1w/640?wx_fmt=png&from=appmsg#imgIndex=7)

方法一，二：
------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQRvFc2nQUMAqx7OgcBvbb8oDrhke5Crre11CeJmjJ8c9Dp5y62m7qwQ/640?wx_fmt=png&from=appmsg#imgIndex=8)

方法三：
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQ90jFVFCH2gRoq0ngbHUCMUT8sWGmdRgHL9iaibR0TJm4ib5IAw07AMZZA/640?wx_fmt=png&from=appmsg#imgIndex=9)

案例二：
====

_https://antispider8.scrape.center/_

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQTyhhssD6C9x8mLibqpUnQw1eVxuicw8v7XJoxTiaiaiaNMPbpG4aVsaOjNA/640?wx_fmt=png&from=appmsg#imgIndex=10)遇到类似 setInterval 类型的![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQMhx97kbE0EGzZQFryMU7ENArF2xswVNiaOdRib1Qjw7iaexG3ibo7IQq5Q/640?wx_fmt=png&from=appmsg#imgIndex=11)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQlYJlnLZ1gb1JRYrL3NsmNpuKaliaANc19mfqthDAePybw3lDgcheS8w/640?wx_fmt=png&from=appmsg#imgIndex=12)~ 就可以直接过

方法四：
----

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQNwia6gp4MjqW3AicrHXYLXOMc18lVqAeYBRAviaz0EQx1G2nbicHgqv7cg/640?wx_fmt=png&from=appmsg#imgIndex=13)

方法 5：
-----

直接一律不在此暂停![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQsRibDc0grfnHlmGic7tfWzia3kLm7icbXL0ovRr6kLDPo610AatDGsRiaVQ/640?wx_fmt=png&from=appmsg#imgIndex=14)

案例 3：
=====

*https://artmeta.cn/pages/login/index![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQMhw6QcjwF9A3DCScg7iaQbjvYyzpYXV7XsgicaD1iahlEibzxw9nE1rG5g/640?wx_fmt=png&from=appmsg#imgIndex=15)*![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQVjicbXCvwFIa0C6Pbwd6DlzP2Au4QBfHudhx8ltHf2VfsicD7JcEQfIg/640?wx_fmt=png&from=appmsg#imgIndex=16)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQPoZ93JmmewaV5hAjwUVsZOPUrag6qwuxR9icmL84pEYXmhQd9FwhjCQ/640?wx_fmt=png&from=appmsg#imgIndex=17)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQFYujtBVezrODzyAQedYlialw5C9XZbIqMwcv1ErS8D4ibzykw7gy1VFA/640?wx_fmt=png&from=appmsg#imgIndex=18)发现他是通过 constructor 构造成的 debugger，然后持续调用 docheck 这个函数，从而一直形成 debugger 看了几个案例我们得到了基本构成方法：

脚本片段一：
======

第一个就是通过 eval，里面加一下混淆就轻松实现：

那我们先把这几个方法制空：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQC8nq6cdcDoIAbhvwujkUBLBeR17GicP5xMWjQKAh87lsSrcYegqc9UA/640?wx_fmt=png&from=appmsg#imgIndex=19)这段代码主要是用来 hook（劫持 / 重写）JavaScript 的全局 eval() 函数，主要目的是移除代码中的 debugger 语句。让我们逐步分析：

1.  首先保存原始的 eval 函数：
    

```
var temp_eval = eval;


```

然后重写全局 eval 函数：

```
window.eval = function () {// ...}


```

核心功能：

*   检查传入 eval 的参数是否为字符串
    
*   如果是字符串，查找其中是否包含 debugger 关键字
    
*   如果存在 debugger 语句，则将其全部删除
    
*   最后使用原始的 eval 函数执行处理后的代码
    

这种技术通常用于：

*   反调试：阻止使用 debugger 语句进行调试
    
*   代码保护：移除可能被用于分析代码的调试点
    
*   绕过某些网站的反调试机制
    

举个例子：![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQmRMTt7RE0SFyWjia31cNQ8tibiczZMibwBjKiaDNqFDMbbBepxqicd5lIUWw/640?wx_fmt=png&from=appmsg#imgIndex=20)

脚本片段二：
======

争对用定时器 setinterval 类型的脚本：![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQYEhnDR4pdZoepmcSBwq2MNTGZI8oVGiacDT5KEy6wKcbmzyicM8ln2nw/640?wx_fmt=png&from=appmsg#imgIndex=21)let _setInterval = setInterval;

*   首先保存原始的 setInterval 函数到变量 _setInterval
    
*   这样做是为了后面还能调用原始的函数
    
    setInterval = function(func, delay) { ... }
    
*   重写（覆盖）全局的 setInterval 函数
    
*   这个新函数接收两个参数：
    
*   func: 要定期执行的函数
    
*   delay: 执行间隔时间（毫秒）
    

```
if (func.toString().indexOf("debugger") !== -1)


```

将传入的函数转换为字符串

*   检查字符串中是否包含 "debugger" 关键字
    
*   如果找到 debugger，则：
    
*   打印警告日志："Hook setInterval debugger!"
    
*   返回一个空函数 function() {}，从而阻止原始代码的执行
    
*   如果没有检测到 debugger，则调用原始的 setInterval 函数正常执行
    

脚本片段三：
======

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQ8UkJicicTZRGlrUh18pY7ob24G3eNWdYzWQBT2I0GhkOFX5IgOgqGZyg/640?wx_fmt=png&from=appmsg#imgIndex=22)var _debugger = Function;

*   保存原始的 Function 构造函数到 _debugger 变量
    
*   这样做是为了后面能够调用原始的构造函数
    
*   Function = function () { ...}
    
*   重写全局的 Function 构造函数
    
*   这个新的构造函数会处理所有传入的参数
    
*   核心处理逻辑：
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQ8gQX9Sf8H5jy7OCexp55nFgGZf7ffQV6T4xiazp9xib1FBHR86HgicGIA/640?wx_fmt=png&from=appmsg#imgIndex=23)
    

遍历所有传入的参数

*   如果参数是字符串类型，就检查是否包含 "debugger" 关键字
    
*   如果找到 debugger，就循环删除所有的 debugger 语句
    
*   使用正则表达式 /debugger/ 来匹配和替换
    

```
return _debugger(...arguments);


```

*   使用处理后的参数调用原始的 Function 构造函数
    
*   使用展开运算符 ... 传递所有参数
    

```
Function.prototype = _debugger.prototype;


```

*   确保新的 Function 构造函数继承原始 Function 的原型链
    
*   这样可以保持所有 Function 的原型方法正常工作
    

脚本片段四：
======

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQKFsr3FkHkCX1ApJq9P4rk67wNgtF1EAiblSUJwCMkWC5SssG32PUVUw/640?wx_fmt=png&from=appmsg#imgIndex=24)这里和片段三逻辑差不多，不做赘述，但是我们产生一个疑问，就是为啥![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQZbb3ibemIlrJ0nJVnT39G84Viauq1n9IF2BSGuBiaZWBLAEj8TTGRku1g/640?wx_fmt=png&from=appmsg#imgIndex=25)而片段一片段二不用捏：

> setInterval 不需要维护原型链是因为：
> 
> *   setInterval 是一个全局函数（window 对象的方法）
>     
> *   它是一个普通函数，不是构造函数
>     
> *   不会用 new setInterval() 的方式来创建实例
>     
> *   没有需要继承的原型方法和属性
>     
> 
> Function 需要维护原型链因为：
> 
> *   Function 是一个构造函数
>     
> *   可以使用 new Function() 创建新的函数实例
>     
> *   所有函数都继承自 Function.prototype
>     

而原型链的知识不做赘述 基于此我们的脚本基本成型，但是真的完美了吗 我们测试一下 rs 网站：_https://www.lzu.edu.cn/_

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQzhKWGZdFR7SdGjRx3n6LE4q6M7rEiagsNVbdY5n1W7hsHe3H1Y8MicjQ/640?wx_fmt=png&from=appmsg#imgIndex=26)我们发现报错了，找到报错的位置和代码，发现他检测了 hook： 那么是如何检测的捏 我们在注入之前是 eval() { [native code] } 注入之后变成：![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQ0iau99e5coaUhAC5MKMaoV3KVf0j6GU8sK2AaAesH9j4ibXJ4Irc83qg/640?wx_fmt=png&from=appmsg#imgIndex=27)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQMjuUj3FUhvcLyuRibdu4ficVYSzg6Sm1ehITNEMdiaXQw4WLk3Uz5pP0w/640?wx_fmt=png&from=appmsg#imgIndex=28)那么基于此我们就可以检测 tostring 来发现是不是提前 hook 了，有没有办法捏，当然：

我们 hook tostring 不就完事：

脚本片段五
=====

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQGtLPm7BJF1oHwcqY89cZ3ZH1GNSQQUpNplu91Rk53AwnouHx8A2uWw/640?wx_fmt=png&from=appmsg#imgIndex=29)看一下效果：![](https://mmbiz.qpic.cn/sz_mmbiz_png/FgxuicwP5dibo32DdB0aoeaXAoDTSn4CSQePxbWxaH0Hn9ialxO9ZacRdoDc9EkUcCTC7JnNaqbRibvnTczvdtqaKg/640?wx_fmt=png&from=appmsg#imgIndex=30)注入前和注入后是一个东西了，我顺带写了其他如何被检测的地方，读者可自行查看

万能脚本:
=====

```
(function() {  // 立即执行函数，创建独立作用域
    'use strict';  // 启用严格模式，防止一些不规范的代码写法
    // 调试配置对象
    //按0开启日志，默认关闭
    const DEBUG = {
        enable: 1,    // 控制是否输出日志信息的开关
        deb: 1       // 控制是否在关键位置设置断点的开关
    };
//作者：Dexter
//公众号：我不是蜘蛛
    // 定义日志输出函数，根据 DEBUG.enable 决定是否输出日志
    const log = function(...args) {
        if (DEBUG.enable==0) {
            console.log(...args);
        }
    };
        // 保存原始的 setInterval 函数
        var originalSetInterval = window.setInterval;
        // 用新的函数替换原始的 setInterval
        window.setInterval = function(callback, delay) {
            // 获取除了 callback 和 delay 的其他额外参数
            var args = Array.prototype.slice.call(arguments, 2);
            // 如果 callback 是字符串，则删除其中的 debugger 语句
            if (typeof callback === 'string') {
                callback = callback.replace(/debugger;/g, '');
            } else if (typeof callback === 'function') {
                // 如果 callback 是函数，替换包含 debugger 的回调
                var originalCallback = callback;
                callback = function() {
                    // 获取原始回调函数的源码
                    var callbackSource = originalCallback.toString();
                    // 如果源码包含 debugger，将其删除并创建新的函数
                    if (callbackSource.indexOf('debugger') !== -1) {
                        callbackSource = callbackSource.replace(/debugger;/g, '');
                        originalCallback = new Function('return ' + callbackSource)();
                    }
                    // 调用修改后的回调函数，并传入参数
                    originalCallback.apply(this, arguments);
                };
            }
            // 调用原始的 setInterval 函数，并传入修改后的 callback、delay 和其他参数
            return originalSetInterval.apply(window, [callback, delay].concat(args));
        }
    //============ toString 相关防护 ============
    var temp_eval = eval;                      // 保存原始的 eval 函数
    var temp_toString = Function.prototype.toString;  // 保存原始的 toString 方法
    // 改进toString方法的处理
    Function.prototype.toString = function () {
        if (this === eval) {
            return 'function eval() { [native code] }';
        } else if (this === Function) {
            return 'function Function() { [native code] }';
        } else if (this === Function.prototype.toString) {
            return 'function toString() { [native code] }';
        } else if(this===window.setInterval){
            return 'function setInterval() { [native code] }';
        }
        return temp_toString.apply(this, arguments);
    }
    //============ eval 相关hook ============
    window.eval = function () {  // 重写全局 eval 函数
        const stackTrace = new Error().stack;  // 获取调用栈
        const callLocation = stackTrace;       // 保存调用位置
        log(callLocation);                     // 输出调用信息
        if (DEBUG.deb==0) {                       // 根据配置决定是否断点
            debugger;
        }
        log("=============== eval end ===============");
        // 处理传入的字符串参数，移除 debugger 语句
        if (typeof arguments[0] == "string") {
            var temp_length = arguments[0].match(/debugger/g);
            if (temp_length != null) {
                temp_length = temp_length.length;
                var reg = /debugger/;
                while (temp_length) {
                    arguments[0] = arguments[0].replace(reg, "");
                    temp_length--;
                }
            }
        }
        return temp_eval(...arguments);  // 使用原始 eval 执行处理后的代码
    }
    //============ Function 相关hook ============
    var _debugger = Function;  // 保存原始 Function 构造函数
    Function = function () {  // 重写 Function 构造函数
        const stackTrace = new Error().stack;  // 获取调用栈
        const callLocation = stackTrace;       // 保存调用位置
        log(callLocation);                     // 输出调用信息
        if (DEBUG.deb==0) {                       // 根据配置决定是否断点
            debugger;
        }
        log("=============== Function end ===============");
        // 处理所有参数中的 debugger 语句
        var reg = /debugger/;
        for (var i = 0; i < arguments.length; i++) {
            if (typeof arguments[i] == "string") {
                var temp_length = arguments[i].match(/debugger/g);
                if (temp_length != null) {
                    temp_length = temp_length.length;
                    while (temp_length) {
                        arguments[i] = arguments[i].replace(reg, "");
                        temp_length--;
                    }
                }
            }
        }
        return _debugger(...arguments);  // 使用原始 Function 构造函数创建函数
    }
    // 保持原型链的完整性
    Function.prototype = _debugger.prototype;
    // 重写 Function 构造函数的 constructor
    Function.prototype.constructor = function () {
        const stackTrace = new Error().stack;  // 获取调用栈
        const callLocation = stackTrace;       // 保存调用位置
        log(callLocation);                     // 输出调用信息
        if (DEBUG.deb==0) {                       // 根据配置决定是否断点
            debugger;
        }
        log("=============== Function constructor end ===============");
        // 处理所有参数中的 debugger 语句
        var reg = /debugger/;
        for (var i = 0; i < arguments.length; i++) {
            if (typeof arguments[i] == "string") {
                var temp_length = arguments[i].match(/debugger/g);
                if (temp_length != null) {
                    temp_length = temp_length.length;
                    while (temp_length) {
                        arguments[i] = arguments[i].replace(reg, "");
                        temp_length--;
                    }
                }
            }
        }
        return _debugger(...arguments);  // 使用原始 Function 构造函数
    }
    // 确保构造函数的原型链正确
    Function.prototype.constructor.prototype = Function.prototype;
})();

```

结语：

写下这篇文章还是花了不少功夫的，一直在想能不能有个万能脚本解决一切无聊的 debugger，后面与 Spade sec 的作者交流后灵感迸发，遂出此篇，在此鸣谢。

因篇幅原因，关注作者或者加入知识星球喵（点击下方原文）

如果有过不了的 debugger，留言在下方，免费解决

本文大部分都是自己的想法，如有不对的地方，欢迎指正