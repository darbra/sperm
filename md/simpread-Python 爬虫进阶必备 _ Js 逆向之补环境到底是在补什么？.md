> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651266047&idx=1&sn=932ec5044b283d470b385c9a685dc940&chksm=8d315f90ba46d686a561f7deb39e4b5fa4b871fba992acf60b1db39587411960adf78c1b0d21&mpshare=1&scene=1&srcid=0310h4xceCx0N4owVhhkjB1Z&sharer_sharetime=1646881346407&sharer_shareid=56da189f782ce62249ab4f6494feca50&version=3.1.20.90367&platform=mac#rd)

Official Account

点击上方 “咸鱼学 Python”，选择 “加为星标”

第一时间关注 Python 技术干货！

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGd3RW99kP9ia4yiauaWyNoHLx7XNqa4RIOmicSq7CBD7rg0fdXqwcyzblVA/640?wx_fmt=png)

序言
--

之前我就发过一篇文章，提了一嘴关于我理解的爬虫的本质

[**面试官问我会不会 APP 抓包, 我..**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651261597&idx=1&sn=67d62f684c19f5a5fa3c1fe2ed396f3c&chksm=8d314ef2ba46c7e42d036544b88afeb78dc06b91b11264ebe8aa060b05f5402f230f3159425a&scene=21#wechat_redirect)  

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdkusbkWWqBDRk3hWRJiaeDlsqIbHtjb15uMXJNClib6L0YjsqVPJ8lH2A/640?wx_fmt=png)

虽然当时的主题写的是 App 爬虫，不过并不妨碍的我们理解爬虫。

今天写的 Js 逆向之补环境，就可以理解是在 Js 环境下精进我们的 "骗术"

正文
--

大家在看文章之前应该都清楚，Node 环境和浏览器环境是完全不同的，平台有很多的检测点可以发现我们是在浏览器运行 Js 还是在 Node 环境下运行 Js

补环境做的就是尽可能根据网页上的 Js 完善本地的 Node 环境，让 Js 运行在 Node 中像浏览器一样正常运行，最高境界当然是 Js 拿来套上环境就跑，不过可以说是道阻且长![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdrXV9kU0obXL9xiakXgl28RzVpriajgUPUPz9a2squVnDu0gmiaCHKR3hA/640?wx_fmt=png)。

### window

最基本的补环境是公众号早期的文章，例如下面的几篇文章

[**实战案例浅析 JS 加密 - DES 与 Base64**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651261622&idx=1&sn=bb1f04ec80c9aeec17ca5b47083b1094&chksm=8d314ed9ba46c7cf36913e31456ad5afcff5b28cc1bdc54e34836a658cb6ac1ccfa88750c856&scene=21#wechat_redirect)  

[**实战案例浅析 JS 加密 - RSA 与 XXTEA**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651261627&idx=1&sn=1f58708b5e2dd7adb5f6f160b842d15e&chksm=8d314ed4ba46c7c255344847f7547f790a5ab8abf487d0810059ef8baa7350d7f9f47469a8ff&scene=21#wechat_redirect)  

这些个文章中大家经常遇到的是

```
'window' is not defined


```

那像这样的报错提示应该如何处理？

Node 环境下一般如下定义

```
window = global;


```

如果只是单单缺少了`window`这一个变量的定义，像上面这样报错自然就消失了。

### document

除了`window`之外，我们经常还遇到类似下面这些文章中的情况

[**JS 逆向 | 助力新手 , 两个 JS 逆向喂饭教程**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651261831&idx=1&sn=ac2024651406cac642c606f7336abffc&chksm=8d314fe8ba46c6fe967ac2d320ca9a29df2b43e12707599b7c0196fab255bcfbd265db4aa727&scene=21#wechat_redirect)  

[**JS 逆向 | 助力新手 , 再来一套喂饭教程！**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651261852&idx=1&sn=addc60b8639d89c0d3aba0077608810a&chksm=8d314ff3ba46c6e5caee83bf5688c75367deb602d826625074c4ad5457942935da036f5f86f8&scene=21#wechat_redirect)  

[**Python 爬虫进阶必备 | 某采购网站 cookie 加密分析（仿加速乐）**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651265241&idx=1&sn=10de82ee0ebe9b511b3ac44b39ed4436&chksm=8d315cb6ba46d5a0b86634c371cb4cf00dcb1ccff1e81e658e476d32f332c729164f334c80e9&scene=21#wechat_redirect)  

[**Python 爬虫进阶必备 | 某常见 cookie 加密算法逻辑分析 （加速乐 - jsl）**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651265644&idx=1&sn=e6d272d5d2d61f0303cb745fc160c825&chksm=8d315e03ba46d715c47bd010324da6cf02537a79f265b7f3253bfb71211bb8997ec736a40637&scene=21#wechat_redirect)  

这些网站我们一般称之为`加速乐`，因为标志性的 cookie 参数是以`jsl`开头，而这种类型的加密一般操纵的是`document.cookie`这个参数

那像这样的`document`应该怎么补？

在没有检测只是为了能让 js 运行不报错的情况下，我是这样写的

```
var document = {
    cookie:"xxxxxx"
}


```

或者像下面这样写

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdrKia41KeqX8NHb42dlHaaicVNhdtda9B948NibzXNNbMWSqX0LYKAwdCw/640?wx_fmt=png)

那么这样写就一定保险吗？

不一定。开头就已经提及，要根据网页上的 Js 完善本地的 Node 环境，所以只要网页上的 Js 不检测我们这么写也没毛病

那么检测的 Js 长什么样？

### Object.getOwnPropertyDescriptor

这里可以参考之前写的关于某乎的分析文章

[**Python 爬虫进阶必备 | 某著名人均百万问答社区 header 参数加密逻辑分析**](http://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651265603&idx=1&sn=2ff71d1d4b6c294812b2d6dba93cfc52&chksm=8d315e2cba46d73a32b5cb492d69c4968d7aeb28dbfed75a4e579273bbf1bf0e285a9f9b0499&scene=21#wechat_redirect)  

这里`Header`中的`x-zse-96`的加密逻辑之前看过文章的，通过插桩调试应该可以看到下面这样的代码

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdoSLaMzkDnD7KFicVibGfiadSteDxftRrEvFcViaoAYePC5foJc3FxaSNiaw/640?wx_fmt=png)

这个东西是个啥？

官方定义是这样的

> “
> 
> **`Object.getOwnPropertyDescriptor()`** 方法返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）

通过描述有点晦涩，你可以这样理解，如果是自己构造的对象，例如

```
var navigator = {
    platform:"win32"
}


```

就没办法通过这样的检测

这里是检测对象的属性是不是自己赋值给他的

这里打印下刚刚的例子看看有啥不一样的地方

先是我们自己写的例子

```
var navigator1 = {
    platform:"win32"
}


```

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdFDwLB8UC0tde2HRSPfhp1OznHfGlverh8Y1w8zkmHgMC585EljRCww/640?wx_fmt=png)

再看看浏览器里面是啥样的

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGd2KDltnNwK46RUTXnxbFoCwJ5YrhrWrzic7CXsPwdSibtcepX88OWbaVg/640?wx_fmt=png)

所以对于这样的代码检测，我们应该如何构造呢？

这里站在前人的肩膀上，写一下

```
var Navigator = function() {};
Navigator.prototype = {"platform": "win32"};
navigator = new Navigator();
# 只针对检测 Navigator 原型链的写法


```

这样就可以通过上面的原型链检测了

这里的`platform`是`Navigator`上的属性，在使用 new 实例化后，navigator.platform 取到的是继承自`Navigator`的属性值，而不是直接赋予该对象的属性，所以得到的结果和浏览器是一样的。

这样看是不是很简单，但是像某乎的校验用到了很多次`getOwnPropertyDescriptor`，就需要一个个插桩调试他检测了什么对象的什么属性，相当恶心。

那么又回到上面的代码，这里用到的`prototype`又是个啥？

### prototype 与 _ _ _proto_ _ _

首先先上一张图

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdBj8hzqLkPMic93rDzSFko0dHgeR0gibgX6ibBnJW7WTwgtuk3EaNMlawA/640?wx_fmt=png)

是不是很懵逼，懵逼就对了，我们是搞爬虫的，知道个大概意思就行了。

我们需要知道的是 prototype 与 _ _ _proto_ _ _ 咋对应起来就行了

按照官方的说法

> “
> 
> 在 JS 里，万物皆对象。方法（Function）是对象，方法的原型 (Function.prototype) 是对象。因此，它们都会具有对象共有的特点。即：对象具有属性_ _ _proto_ _ _，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例能够访问在构造函数原型中定义的属性和方法。

> 方法（Function）这个特殊的对象，除了和其他对象一样有上述_proto_属性之外，还有自己特有的属性——原型属性（prototype），这个属性是一个指针，指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对象）。原型对象也有一个属性，叫做 constructor，这个属性包含了一个指针，指回原构造函数。

理解大概的意思（别被搞晕了）可以得出下面这行代码

```
 xxx.__proto__ == yyy.prototype


```

这里我们需要用到谷歌浏览器验证一下我们的想法

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdpZKticgP4DefLeD8YIibIPicaDoYiaZ9t95yylWDkBDq2B8iaoR3JzLrFfA/640?wx_fmt=png)

有一定基础的同学知道，浏览器里 window 是由 Window 实例而来的，那么 Window 是怎么来的？

我们在浏览器的控制台里看看

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdhROVMLS5nnGpLKFOEtfrhtJ8JYYwmXLGsicS3HwB003ibMYSAtibIqoRQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdib6rB9ElHKBRhk0cSXribdpLibmXI3UMpSu6VY7Igzn22a1ncLy3tgKJg/640?wx_fmt=png)

可以看到原来我们之前以为简简单单就构造出来的 window 和 navigator 这么复杂

这样一看像我们上面直接使用`var`定义的方法去补环境就很容易被识别

例如

```
var window1 = {
    bbb:"xxxx"
};
# 在浏览器里没法定义 window 这里用 window1 做个样子


```

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdsIgTVQWzh8YwNnXiajaZuVbAOUDRRvjeoLH0DzDWgGW9mzfKPiblO5EA/640?wx_fmt=png)

这里`window1`的结果和我们上面浏览器中`window`的输出结果大相径庭。

不仅仅是`window`，包括`document`以及`navigator`等等囊括`DOM`、`BOM`、`worker`、网络请求这些的方方面面都需要我们一个一个分析他的`__proto__`以及属性是定义在自己的`prototype`上还是继承自上一层。

这里就又涉及了关于上述各类的继承关系，这里分享我找到的一张`DOM`中各类的继承关系图，希望对大家补环境有所帮助

![](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl3otEQDnoxAQADQN6DsyiaGdVMg7HOu8oia2AwGHGt8SmiayPmz2SOibxIuoK1eV32ysdhQbcBdOFkt7g/640?wx_fmt=png)

总结
--

这篇文章我的定义并不是关于补环境的总纲或者是总述，只能说是掀起了补环境神秘面纱的一角，希望大家多多交流，不断完善自己的环境框架，做到一键通杀~

还有就是关于文章中如果描述不准确的地方，欢迎在留言区指正，大家共同进步。

以上就是本次的全部内容了，咱们下次再会~

**Peace and Love**