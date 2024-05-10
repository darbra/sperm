> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NJjU7dqA1g0-_xFmr1nrIg)

前言
--

上一篇文章测试了多个补环境的框架 ([多个开源的 js 补环境框架测试](https://mp.weixin.qq.com/s?__biz=MzU0OTkwODU2MA==&mid=2247484913&idx=1&sn=100490dc9cea4d6c3d2aa38dfe12545d&scene=21#wechat_redirect))，这篇来说一下具体怎么补。用`__ac_signature`做测试，这个相对简单一点。

逆向
--

先定位 cookie 生成的位置，清空 cookie 然后刷新网页，可以看到有两次请求。第一次返回了一个`__ac_nonce`的 cookie，未携带`__ac_signature`。第二次就已经带上了这个 cookie，那肯定是之前请求里的 js 加上的。

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWDgl8iawf24BIIia1814mmbwvWricVCibVIrNDJfFibqS7yyoxibRYQmOupQhQ/640?wx_fmt=png&from=appmsg)

看了下中间的几个请求都不是 js，只有可能是在第一次请求返回的内容，不过谷歌浏览器看不到响应内容，只能用抓包工具或者火狐看。

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWD0EptYYyf8AxuZUahJYViauvuw4Wu55r37Kqr9R6V7nWeibXh1C7kIKxw/640?wx_fmt=png&from=appmsg)

html 里面有两个 script 标签，第一个里面就是核心的 js，第二个 script 里是调用和复制 cookie。里面有用的就两行

```
window.byted_acrawler.init({
    aid: 99999999,
    dfp: 0
});
var __ac_signature = window.byted_acrawler.sign("", __ac_nonce);


```

而第一段 js 代码里就是在定义`window.byted_acrawler`，不需要去分析里面做了，直接拿出来整个补环境。

补环境框架
-----

#### node-sandbox

先测试下 node-sandbox。这里再复述一下使用方法，先把源码下载到本地。解压 node.zip 到当前目录，在 main.js 里增加下面的代码：

```
function test_vm() {
    const sandbox = {
        wanfeng: wanfeng,
        globalMy: globalMy,
        console: console,
    }
    let workCode = fs.readFileSync("./work/ac_sign.js");
    a = +new Date;
    var callCode = `window.byted_acrawler.init({
        aid: 99999999,
        dfp: 0
    });
    var __ac_nonce = "06639eaa4009ab37b9a75";
    var __ac_signature = window.byted_acrawler.sign("",__ac_nonce);
    console.log("__ac_signature: ", __ac_signature)`;
    var code = "debugger;\r\n" + globalMy_js + init_env + envCode + "\r\n" + workCode + "\r\n" + endCode + callCode;
    vm.runInNewContext(code, sandbox);
    console.log("运行环境Js + 工作Js 耗时:", +new Date - a, "毫秒");
}
test_vm();


```

然后把 js 放到`./work/ac_sign.js`里，在终端输入`.\node main.js`就会吐出所有的环境，结果也输出出来了

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWDQa7mTHzuDjJ2EEAicv5f6WQaPCliazt8kMlrKVZRpDGODEMG56mmibuVQ/640?wx_fmt=png&from=appmsg)

#### qxVm

qxVm 用起来也是类似，改一下 js 文件名和 userAgent、href 等，完整代码如下：

```
const fs = require('fs');
const QXVM_GENERATE = require('../qxVm_sanbox/qxVm.sanbox');

function ReadCode(name, dir) {
    let file_path = dir === undefined ? `${__dirname}/${name}` : `${__dirname}/${dir}/${name}`;
    return fs.readFileSync(file_path) + "\r\n"
}

const js_code = ReadCode(`./ac_sign.js`);

const user_config = {
    isTest: false,
    runConfig: { proxy: true, logOpen: true},
    window_attribute: {},
    env: {
        navigator: {
            userAgent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
        },
        location: {
            href: "打码处理"
        },
        document: {
            referrer: "打码处理",
            cookie: ''
        }
    }
}
// 帮助信息打印
// QXVM_GENERATE.help()

let result = QXVM_GENERATE.sanbox(js_code, "get_ac_sign", user_config, false);
let ac_sign = result.get_ac_sign("0663c24340045271d4894")
console.log("__ac_signature:", ac_sign)


```

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWDTHP2d11We7bWE3BnPWjfsagD2enl9UAwhVU2ibt1WTUcND3bZxDDVcg/640?wx_fmt=png&from=appmsg)

#### 验证有效性

能生成结果肯定是不够，还得能出数据才行。看了一下，发现 cookie 中的`__ac_signature`其实是可有可无的。如果你直接修改 cookie 来验证的话，也是能返回内容的。

`__ac_signature`的实际作用是为了得到 ttwid，如果`__ac_signature`有效，第二次请求就会在响应头加一个 set-cookie 设置 ttwid 的值，这个值才是后面需要用到的 cookie 值。

所以在 charles 修改第二次请求的`__ac_signature`，看看服务器返回的有没有 set-cookie。如果没有的话，说明触发风控了 (验证码)，可能是`__ac_signature`无效，也可能是请求太多次了。

失败的请求：

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWDwVYcwP3asK1AcpWSwBfJyibYCnLOHrkzXbFickUIRccH9bSm7BqdnA5A/640?wx_fmt=png&from=appmsg)

成功的情况：

![](https://mmbiz.qpic.cn/mmbiz_png/Y56THyb6khO0jgavA7wckVxvMSlhGmWDniamEB7X8Ftm9bydicicJUfQjrKvYR6lH7m0BY4Rmb9KjXLmrBRP8lnew/640?wx_fmt=png&from=appmsg)

测试了下上面两个框架生成的`__ac_signature`都可以得到 ttwid，也就是说环境都是通过了网站的检测。

#### 补环境

既然都把环境吐出来，我还想手动补一下环境，补完发现压根通不过验证，检测的东西太多了。即使环境都补对了而且能在 vm2 里运行，在 node 和 chakracore 里的结果也通不过检测。

还是直接站在巨人的肩膀上开个服务调用吧，这样可以不用折腾那么多事。