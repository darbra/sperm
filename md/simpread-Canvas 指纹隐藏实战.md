> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.mythsman.com](https://blog.mythsman.com/post/60d1a161e8493b525875166c/#%E5%AE%9E%E6%88%98-%E6%9F%90%E9%9F%B3)

> 前言 前两天和隔壁做风控的同学聊天，据说他们经常使用浏览器指纹来识别和标记爬虫（当然具体的细节是不能透露的），联想到我们最近也经常遇到被风控的情况，于是就花了点时间研究下浏览器指纹相关的知识。

前两天和隔壁做风控的同学聊天，据说他们经常使用浏览器指纹来识别和标记爬虫（当然具体的细节是不能透露的），联想到我们最近也经常遇到被风控的情况，于是就花了点时间研究下浏览器指纹相关的知识。

需要明确的是，“**隐藏浏览器指纹**” 和 “**隐藏 Webdriver/Selenium/Puppeteer**”、“**匿名浏览**” 都不是同一个问题。“隐藏 Webdriver/Selenium/Puppeteer” 的目的是告诉服务端自己不是自动化爬虫（这个似乎可以尝试用 [stealth.min.js](https://github.com/berstend/puppeteer-extra/blob/master/packages/extract-stealth-evasions/readme.md) 来做）；“匿名浏览” 的目的是不让浏览记录在本地留下痕迹（这个可以用浏览器的匿名模式来实现）；“隐藏浏览器指纹” 是为了不让服务端追踪 “现在的我和之前的我是同一个我”（这个是我们这次想解决的问题）。

浏览器指纹一般是通过 Javascript 能提取或计算得到的一些具有一定**稳定性**和**独特性**的参数。风控的同学将这些参数做一定取舍和组合，就能得到一台浏览器的大致标识。在没有强制登陆的情况下，有了这个标识，风控的同学可以做反爬、广告的同学可以做推荐、数据的同学可以算 uv 等等，用处还是不小的。

常见的浏览器指纹会提取如下东西：

1.  UserAgent 和平台信息
2.  浏览器加载的字体信息
3.  声卡指纹
4.  Canvas 指纹
5.  显示器分辨率
6.  语言、地区时区信息
7.  CPU 核数和可用内存信息
8.  本地存储、Cookie 等信息
9.  浏览器安装的插件信息
10.  科学计算的指纹信息
11.  IP 和代理信息

可以看出来，这里有的指标稳定性比较好，比如 UserAgent、分辨率、地区、语言、字体信息等，但是这些指标通常区分度不是很好。因此我们通常更喜欢用一些硬件指纹信息来进行区分。当然，为了衡量指标的这种 “**独特性**”，[Peter Eckersley](https://pde.is/) 提出了用一个指标对设备唯一指纹引入的熵来衡量指标的这种特性，下面是我的设备在 [coveryyourtracks](https://coveryourtracks.eff.org/) 中跑出来的一些常见指标的效果：

```
USER AGENT
Bits of identifying information: 9.04
One in x browsers have this value: 524.71
 
HTTP_ACCEPT HEADERS
Bits of identifying information: 8.96
One in x browsers have this value: 496.75
 
BROWSER PLUGIN DETAILS
Bits of identifying information: 3.25
One in x browsers have this value: 9.54
 
TIME ZONE OFFSET
Bits of identifying information: 5.26
One in x browsers have this value: 38.38
 
TIME ZONE
Bits of identifying information: 6.83
One in x browsers have this value: 114.13
 
SCREEN SIZE AND COLOR DEPTH
Bits of identifying information: 6.28
One in x browsers have this value: 77.57
 
SYSTEM FONTS
Bits of identifying information: 3.92
One in x browsers have this value: 15.12
 
ARE COOKIES ENABLED?
Bits of identifying information: 0.13
One in x browsers have this value: 1.09
 
LIMITED SUPERCOOKIE TEST
Bits of identifying information: 1.41
One in x browsers have this value: 2.67
 
HASH OF CANVAS FINGERPRINT
Bits of identifying information: 9.73
One in x browsers have this value: 847.61
 
HASH OF WEBGL FINGERPRINT
Bits of identifying information: 9.75
One in x browsers have this value: 862.69
 
WEBGL VENDOR & RENDERER
Bits of identifying information: 8.5
One in x browsers have this value: 362.9
 
DNT HEADER ENABLED?
Bits of identifying information: 0.95
One in x browsers have this value: 1.93
 
LANGUAGE
Bits of identifying information: 7.47
One in x browsers have this value: 176.69
 
PLATFORM
Bits of identifying information: 3.11
One in x browsers have this value: 8.62
 
TOUCH SUPPORT
Bits of identifying information: 0.76
One in x browsers have this value: 1.7
 
AD BLOCKER USED
Bits of identifying information: 0.15
One in x browsers have this value: 1.11
 
AUDIOCONTEXT FINGERPRINT
Bits of identifying information: 5.92
One in x browsers have this value: 60.59
 
CPU CLASS
Bits of identifying information: 0.12
One in x browsers have this value: 1.09
 
HARDWARE CONCURRENCY
Bits of identifying information: 1.93
One in x browsers have this value: 3.82
 
DEVICE MEMORY (GB)
Bits of identifying information: 2.31
One in x browsers have this value: 4.96

```

其中，“Bits of identifying information” 表示这个指标对我的浏览器引入的唯一性的位数，这个位数越大，越表明这个指标更能区分我的浏览器和其他的浏览器；“One in x browsers have this value” 表示平均多少个浏览器的这个指标和我的浏览器的这个指标一样，这个数越大，越表明这个指标的区分度好。

显然，从这里看，区分度最高的指标就是 "UserAgent" , "WebGL 指纹" 和 "Canvas 指纹" 。那这些指纹是怎么 work 的呢？例如 Canvas 指纹，其实就是由于 Web 浏览器使用的图像处理引擎、图像导出选项、压缩级别、操作系统的字体，抗锯齿和亚像素渲染算法等的不同，导致画出来的图片在像素级别存在的微小但稳定的差距。这样我们就可以拿这个生成图片的校验和或算一个 Hash 作为他的指纹。

目前业内较为知名的浏览器指纹检测网站大概有下面这三个：

1.  [https://fingerprintjs.github.io/fingerprintjs/](https://fingerprintjs.github.io/fingerprintjs/)
2.  [http://f.vision/](http://f.vision/)
3.  [https://coveryourtracks.eff.org/](https://coveryourtracks.eff.org/)

第一个是最常见的浏览器指纹生成项目，之前某手用过一段时间，后来好像不用了，可以作为入门项目来看。

第二个是一个进阶的在线检测指纹的网站，检测点更全，也有一定的反伪造能力。

第三个是 [Peter Eckersley](https://pde.is/) 实验用的检测指纹的网站，教育意义更明显，也有一定的反伪造能力。

有矛就有盾，作为（爬虫）普通用户，我们显然不想让服务端跟踪我们的操作路径，但又不想用 [Tor](https://www.torproject.org/) 浏览器。因此我们就要想一些对付这些指纹检测的办法。下面就是我做的一些尝试。

指纹生成算法
------

首先我们先看下 fingerprintjs 检测 Canvas 指纹的核心代码，作为我们首先需要绕过的目标：

```
function makeTextImage(canvas, context) {
    
    canvas.width = 240;
    canvas.height = 60;
    context.textBaseline = 'alphabetic';
    context.fillStyle = '#f60';
    context.fillRect(100, 1, 62, 20);
    context.fillStyle = '#069';
    
    
    context.font = '11pt "Times New Roman"';
    
    
    
    
    
    
    var printedText = "Cwm fjordbank gly " + String.fromCharCode(55357, 56835) ;
    context.fillText(printedText, 2, 15);
    context.fillStyle = 'rgba(102, 204, 0, 0.2)';
    context.font = '18pt Arial';
    context.fillText(printedText, 4, 45);
    return save(canvas);
}
function makeGeometryImage(canvas, context) {
    
    canvas.width = 122;
    canvas.height = 110;
    
    
    
    context.globalCompositeOperation = 'multiply';
    for (var _i = 0, _a = [
        ['#f2f', 40, 40],
        ['#2ff', 80, 40],
        ['#ff2', 60, 80],
    ]; _i < _a.length; _i++) {
        var _b = _a[_i], color = _b[0], x = _b[1], y = _b[2];
        context.fillStyle = color;
        context.beginPath();
        context.arc(x, y, 40, 0, Math.PI * 2, true);
        context.closePath();
        context.fill();
    }
    
    
    
    context.fillStyle = '#f9c';
    context.arc(60, 60, 60, 0, Math.PI * 2, true);
    context.arc(60, 60, 20, 0, Math.PI * 2, true);
    context.fill('evenodd');
    return save(canvas);
}

```

这里主要生成了下面两个东西：

指定形状、字体、文字、颜色等，画了一个图。

![](https://blog.mythsman.com/content/images/2021/06/image-9.png)

指定了颜色、位置，画了一些奇奇怪怪的圆弧。

![](https://blog.mythsman.com/content/images/2021/06/image-10.png)

然后 fingerprintjs 会用这两个图，根据 murmurHash3 算一个指纹。

Canvas Fingerprint Defender
---------------------------

Chrome 有一个声称能防止 canvas 指纹泄露的插件 [Canvas Fingerprint Defender](https://chrome.google.com/webstore/detail/canvas-fingerprint-defend/lanfdkkpgfjfdikkncbnojekcppdebfp) ，我们姑且拿他来检测绕过 fingerprintjs 看看。

安装后，找到 crx 安装目录，发现他的逻辑主要是在 `data/content_script/inject.js` 中，核心逻辑如下：

```
var inject = function () {
  const toBlob = HTMLCanvasElement.prototype.toBlob;
  const toDataURL = HTMLCanvasElement.prototype.toDataURL;
  const getImageData = CanvasRenderingContext2D.prototype.getImageData;
  
  var noisify = function (canvas, context) {
    if (context) {
      const shift = {
        'r': Math.floor(Math.random() * 10) - 5,
        'g': Math.floor(Math.random() * 10) - 5,
        'b': Math.floor(Math.random() * 10) - 5,
        'a': Math.floor(Math.random() * 10) - 5
      };
      
      const width = canvas.width;
      const height = canvas.height;
      if (width && height) {
        const imageData = getImageData.apply(context, [0, 0, width, height]);
        for (let i = 0; i < height; i++) {
          for (let j = 0; j < width; j++) {
            const n = ((i * (width * 4)) + (j * 4));
            imageData.data[n + 0] = imageData.data[n + 0] + shift.r;
            imageData.data[n + 1] = imageData.data[n + 1] + shift.g;
            imageData.data[n + 2] = imageData.data[n + 2] + shift.b;
            imageData.data[n + 3] = imageData.data[n + 3] + shift.a;
          }
        }
        
        window.top.postMessage("canvas-fingerprint-defender-alert", '*');
        context.putImageData(imageData, 0, 0);
      }
    }
  };
  
  Object.defineProperty(HTMLCanvasElement.prototype, "toBlob", {
    "value": function () {
      noisify(this, this.getContext("2d"));
      return toBlob.apply(this, arguments);
    }
  });
  
  Object.defineProperty(HTMLCanvasElement.prototype, "toDataURL", {
    "value": function () {
      noisify(this, this.getContext("2d"));
      return toDataURL.apply(this, arguments);
    }
  });
  
  Object.defineProperty(CanvasRenderingContext2D.prototype, "getImageData", {
    "value": function () {
      noisify(this.canvas, this);
      return getImageData.apply(this, arguments);
    }
  });
  
  document.documentElement.dataset.cbscriptallow = true;
};
inject();

```

代码基本不用解释，主要就做了一件事情：重写 Canvas 的 toBlob，toDataURL 方法和 Context 的 getImageData 方法，使他们在生成图片时加一些噪点。

通过下面的 selenium 代码注入上述脚本：

```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
 
options = Options()
 
driver = webdriver.Chrome(options=options, executable_path="/path/to/chromedriver")
 
f = open('./inject.js')
js = f.read()
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": js})
 
driver.get('https://fingerprintjs.github.io/fingerprintjs/')

```

连续跑两次，发现指纹的确并不相同：

![](https://blog.mythsman.com/content/images/2021/06/image-19.png)![](https://blog.mythsman.com/content/images/2021/06/image-20.png)

但是上面的脚本如果应用在 f.vision 上（或者  [canvas-fingerprint](https://webbrowsertools.com/canvas-fingerprint/) ），虽然也会生成不同的指纹，但是会发现有下面的结果：

![](https://blog.mythsman.com/content/images/2021/06/image-27.png)

有意思，果然是做了一些检测，让我们摘下面具看看他做了啥。一番搜索后发现了如下检测点。

检测点一
----

```
CanvasRenderingContext2D.prototype.getImageData.length !== 4 
|| !CanvasRenderingContext2D.prototype.getImageData.toString().match(/^\s*function getImageData\s*\(\)\s*\{\s*\[native code\]\s*\}\s*$/) 
|| (CanvasRenderingContext2D.prototype.getImageData.name !== "getImageData" && !ie)

```

原来他 check 了一下关键对象的 prototype 的属性，我们来看下当前实际的结果：

```
CanvasRenderingContext2D.prototype.getImageData.length
0
 
CanvasRenderingContext2D.prototype.getImageData.toString()
"function () {\n      noisify(this.canvas, this);\n      return getImageData.apply(this, arguments);\n    }"
 
CanvasRenderingContext2D.prototype.getImageData.name
"value"

```

这里果然和没有改过的不一样。

检测点二
----

```
function cKnownPixels(size) {
    "use strict";
 
    var canvas = document.createElement("canvas");
    canvas.height = size;
    canvas.width = size;
    var context = canvas.getContext("2d");
    if (!context)
        return false;
 
    context.fillStyle = "rgba(0, 127, 255, 1)";
    var pixelValues = [0, 127, 255, 255];
    context.fillRect(0, 0, canvas.width, canvas.height);
    var p = context.getImageData(0, 0, canvas.width, canvas.height).data;
    for (var i = 0; i < p.length; i += 1) {
        if (p[i] !== pixelValues[i % 4]) {
            return false;
        }
    }
    return true;
}

```

这个是个很容易想到的检测方法，输入一个已知的稳定像素图（由于不存在字体、复杂计算等，因此不同机器的渲染几乎不会有差异），如果输出的结果和已知输入不一样，那肯定是人为加了噪点。

检测点三
----

```
function cReadOut() {
 
    "use strict";
 
    var canvas = document.createElement("canvas");
    var context = canvas.getContext("2d");
 
    if (!context)
        return false;
 
    var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < imageData.data.length; i += 1) {
        if (i % 4 !== 3) {
            imageData.data[i] = Math.floor(256 * Math.random());
        } else {
            imageData.data[i] = 255;
        }
    }
    context.putImageData(imageData, 0, 0);
 
    var imageData1 = context.getImageData(0, 0, canvas.width, canvas.height);
    var imageData2 = context.getImageData(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < imageData2.data.length; i += 1) {
        if (imageData1.data[i] !== imageData2.data[i]) {
            return false;
        }
    }
    return true;
}

```

这也是个比较容易想到的检测方法，将同一份像素点连续输出两次，如果这两次的结果不一样，那肯定是加了某些随机的噪点。

检测点四
----

```
function cDoubleReadOut() {
 
    "use strict";
 
    var canvas = document.createElement("canvas");
    var context = canvas.getContext("2d");
 
    if (!context)
        return false;
 
    var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < imageData.data.length; i += 1) {
        if (i % 4 !== 3) {
            imageData.data[i] = Math.floor(256 * Math.random());
        } else {
            imageData.data[i] = 255;
        }
    }
 
    var imageData1 = context.getImageData(0, 0, canvas.width, canvas.height);
 
    var canvas2 = document.createElement("canvas");
    var context2 = canvas2.getContext("2d");
    context2.putImageData(imageData1, 0, 0);
    var imageData2 = context2.getImageData(0, 0, canvas.width, canvas.height);
 
    for (let i = 0; i < imageData2.data.length; i += 1) {
        if (imageData1.data[i] !== imageData2.data[i]) {
            return false;
        }
    }
    return true;
}

```

和上面的检测挺像，但是检测出来的结果比较 trick，没太搞懂这是检测的啥，不过实践中发现如果 a 通道随机到的噪点偏移值不太好，很容易检测不通过。

各个击破
----

道高一尺，魔高一丈，知道他是怎么检测的，我们就可以有针对性的处理了。解决思路如下：

1.  再次 mock 出对应 prototype 的正确 length、toString、name 属性。
2.  对于没有添加文字、没有复杂位图变换的图，不进行随机填充。
3.  随机参数在启动时就生成且只生成一次，保证相同的图多次处理结果也一样。

具体操作可以参考下面的代码：

```
function random(list) {
    let min = 0;
    let max = list.length
    return list[Math.floor(Math.random() * (max - min)) + min];
}
 
let rsalt = random([...Array(7).keys()].map(a => a - 3))
let gsalt = random([...Array(7).keys()].map(a => a - 3))
let bsalt = random([...Array(7).keys()].map(a => a - 3))
let asalt = random([...Array(7).keys()].map(a => a - 3))
 
const rawGetImageData = CanvasRenderingContext2D.prototype.getImageData;
 
let noisify = function (canvas, context) {
    let ctxIdx = ctxArr.indexOf(context);
    let info = ctxInf[ctxIdx];
    const width = canvas.width, height = canvas.height;
    const imageData = rawGetImageData.apply(context, [0, 0, width, height]);
    if (info.useArc || info.useFillText) {
        for (let i = 0; i < height; i++) {
            for (let j = 0; j < width; j++) {
                const n = ((i * (width * 4)) + (j * 4));
                imageData.data[n + 0] = imageData.data[n + 0] + rsalt;
                imageData.data[n + 1] = imageData.data[n + 1] + gsalt;
                imageData.data[n + 2] = imageData.data[n + 2] + bsalt;
                imageData.data[n + 3] = imageData.data[n + 3] + asalt;
            }
        }
    }
    context.putImageData(imageData, 0, 0);
};
 
let ctxArr = [];
let ctxInf = [];
 
(function mockGetContext() {
    let rawGetContext = HTMLCanvasElement.prototype.getContext
 
    Object.defineProperty(HTMLCanvasElement.prototype, "getContext", {
        "value": function () {
            let result = rawGetContext.apply(this, arguments);
            if (arguments[0] === '2d') {
                ctxArr.push(result)
                ctxInf.push({})
            }
            return result;
        }
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.constructor, "length", {
        "value": 1
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.constructor, "toString", {
        "value": () => "function getContext() { [native code] }"
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.constructor, "name", {
        "value": "getContext"
    });
})();
 
(function mockArc() {
    let rawArc = CanvasRenderingContext2D.prototype.arc
    Object.defineProperty(CanvasRenderingContext2D.prototype, "arc", {
        "value": function () {
            let ctxIdx = ctxArr.indexOf(this);
            ctxInf[ctxIdx].useArc = true;
            return rawArc.apply(this, arguments);
        }
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.arc, "length", {
        "value": 5
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.arc, "toString", {
        "value": () => "function arc() { [native code] }"
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.arc, "name", {
        "value": "arc"
    });
})();
 
(function mockFillText() {
    const rawFillText = CanvasRenderingContext2D.prototype.fillText;
    Object.defineProperty(CanvasRenderingContext2D.prototype, "fillText", {
        "value": function () {
            let ctxIdx = ctxArr.indexOf(this);
            ctxInf[ctxIdx].useFillText = true;
            return rawFillText.apply(this, arguments);
        }
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.fillText, "length", {
        "value": 4
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.fillText, "toString", {
        "value": () => "function fillText() { [native code] }"
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.fillText, "name", {
        "value": "fillText"
    });
})();
 
 
(function mockToBlob() {
    const toBlob = HTMLCanvasElement.prototype.toBlob;
 
    Object.defineProperty(HTMLCanvasElement.prototype, "toBlob", {
        "value": function () {
            noisify(this, this.getContext("2d"));
            return toBlob.apply(this, arguments);
        }
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toBlob, "length", {
        "value": 1
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toBlob, "toString", {
        "value": () => "function toBlob() { [native code] }"
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toBlob, "name", {
        "value": "toBlob"
    });
})();
 
(function mockToDataURL() {
    const toDataURL = HTMLCanvasElement.prototype.toDataURL;
    Object.defineProperty(HTMLCanvasElement.prototype, "toDataURL", {
        "value": function () {
            noisify(this, this.getContext("2d"));
            return toDataURL.apply(this, arguments);
        }
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toDataURL, "length", {
        "value": 0
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toDataURL, "toString", {
        "value": () => "function toDataURL() { [native code] }"
    });
 
    Object.defineProperty(HTMLCanvasElement.prototype.toDataURL, "name", {
        "value": "toDataURL"
    });
})();
 
 
(function mockGetImageData() {
    Object.defineProperty(CanvasRenderingContext2D.prototype, "getImageData", {
        "value": function () {
            noisify(this.canvas, this);
            return rawGetImageData.apply(this, arguments);
        }
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.getImageData, "length", {
        "value": 4
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.getImageData, "toString", {
        "value": () => "function getImageData() { [native code] }"
    });
 
    Object.defineProperty(CanvasRenderingContext2D.prototype.getImageData, "name", {
        "value": "getImageData"
    });
})();
 

```

再用 selenium 跑两下 f.vision，结果如下：

![](https://blog.mythsman.com/content/images/2021/06/image-21.png)![](https://blog.mythsman.com/content/images/2021/06/image-22.png)

这下老实了。

有了上面的能力，理论上我们就可以找个小白鼠来实战试验一下。

下图是某音网页版的推荐页 https://douyin.com/recommend , 指纹计算的逻辑在他的 **webmssdk.js** 中，具体逻辑没仔细看，姑且当成黑盒测试一下。可以看到他给我计算了一个 Canvas 指纹 (即 canvasHash 变量)。多次打开、清理 cookie、使用不同显示器后指纹均能够保持稳定。

![](https://blog.mythsman.com/content/images/2021/09/image-2.png)

然后上一下我们的代码，可以发现指纹的确发生了变化，且每次打开均不同：

![](https://blog.mythsman.com/content/images/2021/09/image-5.png)

本来以为上面的代码足够应对绝大多数场景了，但是回头跑了一下 [https://coveryourtracks.eff.org/](https://coveryourtracks.eff.org/) 这个（检测时可能要翻墙），忽然发现他竟然还是能检测出来我做了伪造:

![](https://blog.mythsman.com/content/images/2021/06/image-24.png)

回想到他在检测时似乎有刷新页面的操作，因此猜测他也是通过刷新页面的方式，比较两次生成的指纹是否相同来检测伪造。于是我尝试将随机性从 js 脚本中提取到 python 代码里，保证相同会话无论刷新多少次都是用的同一套随机数。结果果然印证了我的猜想。

![](https://blog.mythsman.com/content/images/2021/06/image-25.png)![](https://blog.mythsman.com/content/images/2021/06/image-26.png)

虽然看起来上述操作能解决已知的 Canvas 检测问题，但像所有的攻守对抗一样，只要检测方愿意，他们可以十分轻松的想出很多种办法检测出我们检测出了他们的检测代码😅，问题只在于他们是否有闲情做这件事、以及这件事情是否必要。据我所知目前被搞的比较多的网站大都也都没有做这些检测，毕竟除了 Canvas 指纹之外，他们也有太多的工具可以使用。

[BrowserLeaks](https://browserleaks.com/)

[Coveryourtracks.eff.org](http://coveryourtracks.eff.org/)

[How Unique Is Your Web Browser?](https://coveryourtracks.eff.org/static/browser-uniqueness.pdf)

[Device_fingerprint（Wikipeida）](https://en.wikipedia.org/wiki/Device_fingerprint)

[What is Fingerprinting?](https://trac.webkit.org/wiki/Fingerprinting)

[IT IS *NOT* POSSIBLE TO DETECT AND BLOCK CHROME HEADLESS](https://intoli.com/blog/not-possible-to-block-chrome-headless/)

[Do I have a canvas fingerprint spoofer installed in my browser](https://webbrowsertools.com/canvas-fingerprint/)