> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-2012413-1-1.html)

> [md]### 某瑞数 5 代 cookie 和 url 后缀补环境代码。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)pxfsim

#### 某瑞数 5 代 cookie 和 url 后缀补环境代码。

##### url: aHR0cHM6Ly93d3cub3V5ZWVsLmNvbS9zZWFyY2gtbmcvcXVlcnlSZXNvdXJjZS9pbmRleA==

##### 本人新手希望各位大佬看到此贴有错误时多多指点

##### 相关代码

```
delete __dirname;
delete __filename;

_null = function () {
    console.log(arguments)
};

window = global;

window.top = window;
window.self = window;
window.HTMLAnchorElement = function () {
};
window.setInterval = _null;
window.setTimeout = _null;

// content="_all_content";
content = 'y0lyZZ.UMTsawNDd9WWhtfutBd5UjK6aRtk3rV1I7QSVORHOijFveta8b9xTFs._';

window.name = '';

window.addEventListener = _null;

window.attachEvent = undefined;

window.Request = function (args) {
    console.log('window.Request ------>', args);
    console.log(arguments);
    return {};
};

window.fetch = function (args) {
    console.log('window.fetch ------>', args);
    console.log(arguments);
    return {};
};

window.DOMParser = function (args) {
    console.log('window.DOMParser ------>', args);
    console.log(arguments);
    return {};
};

window.open = function (args) {
    console.log('window.open ------>', args);
    console.log(arguments);
    return {};
};

window.TEMPORARY = 0;

webkitRequestFileSystem = {};

window.webkitRequestFileSystem = webkitRequestFileSystem;

_mutationObserver = {
    observe: function (args) {
        console.log('_mutationObserver->observe', args)
        console.log(arguments);
        return {};
    }
};

window.MutationObserver = function () {
    console.log('window.MutationObserver ------>', arguments);
    return _mutationObserver;
};

localStorage = {
    removeItem: function (args) {
        console.log('window.localStorage ------>', args);
        console.log(arguments);
        return {};
    },
    getItem: function (args) {
        console.log('window.localStorage getItem------>', args);
        console.log(arguments);
        return this[args];
    },
};

sessionStorage = {
    removeItem: function (args) {
        console.log('window.sessionStorage removeItem------>', args);
        console.log(arguments);
        return {};
    },
    getItem: function (args) {
        console.log('window.sessionStorage getItem------>', args);
        console.log(arguments);
        return this[args];
    },
};

indexedDB = {
    open: function () {
        console.log('indexedDB->open: ', arguments)
        return {};
    },
};

// 自己去浏览器去取
location = {
    "ancestorOrigins": {},
    "href": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "origin": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "protocol": "https:",
    "host": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "hostname": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "port": "",
    "pathname": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "search": "",
    "hash": ""
};

div_i = {length: 0};

div = {
    getElementsByTagName: function (arg) {
        console.log('div->getElementsByTagName', arguments)
        if (arg === "i") {
            return div_i;
        }
    }
}

meta = {
    getAttribute: function (arg) {
        if (arg === "r") {
            return "m"
        }
    },
    parentNode: {
        removeChild: function (arg) {
            console.log(arg)
        }
    },
    content: content
}

navigator = {
    appVersion: "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",
    languages: ['en-GB', 'zh-CN', 'zh'],
    userAgent: "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",
    connection: {
        downlink: 2.4,
        effectiveType: "4g",
        onchange: null,
        rtt: 50,
        saveData: false
    },
    platform: "Win32",
    languages: ['zh-CN', 'en', 'zh'],
    getBattery: function () {
        return {};
    },
    webdriver: false
};

window.clientInformation = navigator;

script = {
    type: "text/javascript",
    r: 'm',
    parentElement: {
        getAttribute: function (args) {
            console.log('head1->parentElement->getAttribute: ', args)
            console.log(arguments)
            //debugger;
            if (args == 'r') {
                return 'm';
            }
        },
        getElementsByTagName: function (args) {
            console.log('head1->getElementsByTagName: ', args)
            console.log(arguments)
            //debugger
        },
        removeChild: function (args) {
            console.log('head1->parentElement->removeChild', args);
            console.log(arguments);
            //debugger;
        },
    },
    getAttribute: function (args) {
        console.log('script1->getAttribute: ', args)
        console.log(arguments)
        //debugger;
        if (args == 'r') {
            return 'm';
        }
    },
    // 自己去浏览器取
    src: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
};
frist_get = 1;

form = {};

l_input = {};

l2_input = {};

l3_input = {};

form_id = '';

Object.defineProperty(form, 'id', {
    get() {
        return l3_input;
    },
    set(ctx) {
        form_id = ctx;
    }
});

form_action = '';
Object.defineProperty(form, 'action', {
    get() {
        return l_input;
    },
    set(ctx) {
        form_action = ctx;
    }
});

form_textContent = '';

Object.defineProperty(form, 'textContent', {
    get: function () {
        return l2_input;
    },
    set: function (ctx) {
        form_textContent = ctx;
    }
});

form_innerText = '';

Object.defineProperty(form, 'innerText', {
    get: function () {
        return l3_input;
    },
    set: function (ctx) {
        form_innerText = ctx;
    }
});

/*
   这个标签一定要补对 根据自己请求的url去补
   pathname,href,hostname,protocol
   在浏览器环境中 创建一个a标签，然后对a.href赋值会同时对host, pathname,port,hostname,protocol,search赋值
   可使用用以下代码在node环境和chrome环境中测试
   ***在chrome环境中使用控制台测试时不要用console.log(a_node.href),直接用a_node.href查看相关数据
   var a_node = document.createElement('a');
   console.log(a_node.href);
   console.log(a_node.pathname);
   console.log(a_node.port);
   console.log(a_node.hostname);
   console.log(a_node.protocol);
   console.log(a_node.search);
   a_node.href = '/hello?page=1';
   console.log(a_node.href);
   console.log(a_node.pathname);
   console.log(a_node.port);
   console.log(a_node.hostname);
   console.log(a_node.protocol);
   console.log(a_node.search);
   a_node.href = 'https://www.baidu.com:8080/hello?page=1';
   console.log(a_node.href);
   console.log(a_node.pathname);
   console.log(a_node.port);
   console.log(a_node.hostname);
   console.log(a_node.protocol);
   console.log(a_node.search);
*/
a_label = {
    pathname: 'xxxxxxxxxxxxxxxxxxxxxxxxxx',
    port: '',
    host: 'xxxxxxxxxxxx',
    hostname: 'xxxxxxxxxxxxxxxxxxx',
    protocol: 'https:',
    href: 'xxxxxxxxxxxxxxxxxxxxx',
    search: '',
    hash: '',
};

l_obj = {};

input_count = 0;

document = {
    characterSet: 'UTF-8',
    charset: 'UTF-8',
    createExpression: function () {
        console.log('document->createExpression ', arguments);
        return l_obj;
    },
    cookie: '',
    visibilityState: 'hidden',
    body: null,
    createElement: function (a) {
        console.log('document->createElement: ', a);
        // console.log(this);
        console.log(arguments);
        if (a === "div") {
            return div;
        } else if (a === "a") {
            debugger;
            return a_label;
        } else if (a === "form") {
            return form;
        } else if (a == 'input') {
            input_count++;
            console.log(input_count);
            if (input_count == 1) {
                return l_input;
            } else if (input_count == 2) {
                return l2_input;
            } else if (input_count == 3) {
                input_count = 0;
                return l3_input;
            }
        }
        return l_obj;
    },
    getElementsByTagName: function (arg) {
        console.log("getElementsByTagName-->", arguments)
        if (arg === "script") {
            if (frist_get == 1) {
                frist_get = 0;
                return [script, script];
            }
            return []
        }
        if (arg === "meta") {
            return [meta, meta]
        }
        if (arg === "base") {
            return []
        }
    },
    getElementById: function () {
        console.log(arguments)
    },
    addEventListener: function () {
    },
    appendChild: function (args) {
        console.log('appendChild: ', args);
        console.log(arguments);
        return {}
    },
    removeChild: _null,
    documentElement: {}
};

const v8 = require('v8');
const vm = require('vm');
v8.setFlagsFromString('--allow-natives-syntax');
let undetectable = vm.runInThisContext("%GetUndetectable()");

v8.setFlagsFromString('--no-allow-natives-syntax');
Object.defineProperty(document, 'all', {
    configurable: true,
    enumerable: true,
    value: undetectable,
    writable: true,
})

Object.defineProperty(document.all, 'length', {
    get: function () {
        console.log('document.all.length ------------------------------------->')
        return Object.keys(document.all).length
    }
});

document.all[0] = null;
document.all[1] = null;
document.all[2] = null;
document.all[3] = null;
document.all[4] = null;
document.all[5] = null;

XMLHttpRequest = function () {
};

var req_param;

XMLHttpRequest.prototype.open = function (method, url, args) {
    debugger;
    console.log('XMLHttpRequest.prototype.open: ', args)
    console.info(arguments)
    req_param = url;
    return {};
};

require('./ts')

require('./link')

function get_cookie() {
    // console.log(document.cookie)
    return document.cookie;
}

function get_curr(_url) {
    const urls = new URL(_url);
    var host = urls.host;
    var pathname = urls.pathname;
    var search = urls.search;
    var path = pathname + search;
    var g = new XMLHttpRequest();
    debugger;
    console.log('get_curr: ', _url);
    g.open("POST", path, true);
    console.log(req_param);
    return req_param;
}


```

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)lhzy77 想请问下楼主，运行失败了，报错：TypeError: Cannot read properties of undefined (reading 'toString')，好像检测 tostring![](https://avatar.52pojie.cn/data/avatar/000/35/89/70_avatar_middle.jpg)Light 紫星 学习了，楼主厉害![](https://avatar.52pojie.cn/images/noavatar_middle.gif) mirs 6 代后缀有吗？  
![](https://avatar.52pojie.cn/images/noavatar_middle.gif)缘木求鱼  
学习了，谢谢楼主![](https://avatar.52pojie.cn/images/noavatar_middle.gif) fly0079 学习了，谢谢楼主![](https://avatar.52pojie.cn/images/noavatar_middle.gif) lislee zhege hao ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)anxiangyipiao 6 代后缀咋弄![](https://avatar.52pojie.cn/images/noavatar_middle.gif) laoshenshila 感谢楼主的分享！![](https://avatar.52pojie.cn/images/noavatar_middle.gif)Taylor20200522 感谢分享，学习一下![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 614694258 感谢楼主的分享