> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1814095&extra=page%3D1%26filter%3Dtypeid%26typeid%3D378) ![](https://avatar.52pojie.cn/data/avatar/000/95/98/33_avatar_middle.jpg)psych1

一、脚本说明
------

### 为什么会有这个东西？

数据无价的时代，爬虫与反爬的对抗已经进入白热化状态，其中 Cookie 反爬是`最常见之一`的反爬类型， 网站方通过混淆得亲妈都不认识的 JS 代码设置 Cookie（通常是浏览器指纹、请求时必须带上的 Cookie 之类的），  
面对请求时必须要带上但是又不知道在哪里生成的 Cookie， 你在几万行混淆的亲妈都不认识的 JS 屎海中苦苦挣扎希望能找到生成 Cookie 的地方（要是逆向思路不科学兴许还会呛上几口...），  
甚至几度想找个借口骗自己放弃，或者要不干脆用 Selenium 之类的浏览器模拟方式算了？ 怂个球，此脚本就是来助你一臂之力的！ （你我都知道，这段只是撑场面的废话，你可以略过，如果你没有不幸读完的话...）

### 脚本功能

本脚本的功能大致分为两个部分：

*   monitor： 监控所有 JS 操作 cookie 变化的动作并打印在控制台上
*   debugger: 在 cookie 符合给定条件并且发生变化时打 debugger 断点

### Hook 生效的条件

*   需要本脚本被成功注入到页面头部最先执行，脚本都未注入成功自然无法 Hook
*   需要是 JavaScript 操作 document.cookie 赋值来操作 Cookie 才能够 Hook 到 （目前还没碰到不是这么赋值的...）

### 使用须知

本脚本是通过将自己的 JS 代码注入到页面，Hook 住`document.cookie`来完成各种功能， 因此在使用本脚本之前要先确定要搞的 Cookie 确实是通过 JS 生成的  
（后面介绍了一种非常简单的确定 Cookie 是 JS 生成还是服务器返回的方式）。

二、有何优势？
-------

2.1 不影响浏览器自带的 Cookie 管理
-----------------------

目前很多 Hook 脚本 Hook 姿势并不对，本脚本采用的是一次性、反复 Hook，对浏览器自带的 Cookie 管理无影响：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img.png)

2.2 功能更强：监控 Cookie 变化
---------------------

除了 cookie 断点功能之外，增加了 Cookie 修改监控功能，能够在更宏观的角度分析页面上的 Cookie：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img_1.png)

（算了，放弃打码了...）

颜色是用于区分操作类型：

*   绿色是添加 Cookie
*   红色是删除 Cookie
*   黄色是修改已经存在的 Cookie 的值

每个操作都会跟着一个 code location，单击可以定位到做了此操作的 JS 代码的位置。

2.3 功能更强：打断点时细分 Cookie 变化类型
---------------------------

从 v0.6 开始引入了功能更强大并且配置更灵活的断点规则，引入事件机制， 将 Cookie 修改细分为增加、删除、更新三个事件，支持更细粒度的打断点， 关于 Cookie 事件，详情请参阅本文第五部分。

关于为什么要这样设计？ 一种比较常见的情况，目标网站有反爬的 Cookie 是 JS 设置的， 但是 JS 代码的逻辑是先疯狂的删除，然后删除好多次之后才添加真正的值， 这种方式设置 Cookie 正好能反制一般的 Cookie Hook 调试。

这里是其中一个例子，比如 F5 的 Cookie 保护，有一个 Cookie `TS51c47c46075`，它就是先被删除好多次，然后再被添加一次：  
![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/20f986d7.png)  
这种情况下可以针对**添加**名为`TS51c47c46075`的 Cookie 事件打一个断点， 就可以避免那些红色的删除事件混淆视听。

三、 安装
-----

### 3.1 安装油猴插件

理论上只要本脚本的 JS 代码能够注入到页面上即可，这里采用的是油猴插件来将 JS 代码注入到页面上。

油猴插件可从 Chrome 商店安装：

[https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)

如果无法翻墙，可以在百度搜索 “Tampermonkey” 字样寻找第三方网站下载，但请注意不要安装了虚假的恶意插件，推荐从官方商店安装。

其它工具亦可，只要能够将本脚本的 JS 代码注入到页面最头部执行即可。

### 3.2 安装本脚本

安装油猴脚本可以从官方商店，也可以拷贝代码自己在本地创建。

#### 3.2.1 从油猴商店安装本脚本

推荐此方式，从油猴商店安装的油猴脚本有后续版本更新时能够自动更新，本脚本已经在油猴商店上架：

[https://greasyfork.org/zh-CN/scripts/419781-js-cookie-monitor-debugger-hook](https://greasyfork.org/zh-CN/scripts/419781-js-cookie-monitor-debugger-hook)

#### 3.2.2 手动创建插件

如果您觉得自动更新太烦，或者有其它的顾虑，可以在这里复制本脚本的代码：

[https://github.com/CC11001100/js-cookie-monitor-debugger-hook/blob/main/js-cookie-monitor-debugger-hook.js](https://github.com/CC11001100/js-cookie-monitor-debugger-hook/blob/main/js-cookie-monitor-debugger-hook.js)

review 确认没问题之后在油猴的管理面板添加即可。

四、监控 Cookie 的变化（monitor）
------------------------

### 4.1 基本用法

注意，监控是为了在宏观上有一个全局的认识，并不是为了定位细节 （通常情况下正确的使用工具才能提高效率哇，当然一个人的认知是有限的，欢迎大家反馈更有意思的玩法）， 比如打开一个页面时：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img_1.png)

根据这张图，我们就能够对这个网站上哪些 cookie 是 JS 操作的，什么时间如何操作的有个大致的了解。

### 4.2 基本用法进阶

再比如借助 monitor 观察 cookie 的变化规律，比如这个页面，根据时间能够看出这个 cookie 每隔半分钟会被改变一次：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img_2.png)

### 4.3 过滤打印信息，只查看某个 Cookie 的变化

（2021-1-7 18:27:49 更新 v0.4 添加此功能）： 如果控制台打印的信息过多， 可以借助 Chrome 浏览器自带的过滤来筛选，打印的日志的格式已经统一，只需要`cookieName = Cookie名字`即可， 比如：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img_9.png)

请注意，搜索时要保证你的搜索信息是 URL 解码了的，否则可能会不匹配， 因为控制台的打印信息都是先 URL 解码再打印的。

### 4.4 过滤打印信息，快速确定 Cookie 是否是 JS 本地生成的

如果你不确定要搞的 Cookie 是本地生成的还是某个请求服务器`set-cookie`返回的， 则可以把本脚本打开，然后刷新目标网站的页面，然后在控制台搜索 Cookie 名字即可，  
方法与上一节类似，当 Cookie 的名字比较短没有标识性的时候可以加`cookieName`辅助定位，比如：

```
cookieName = v

```

### 4.5 减少冗余信息（不推荐）

有时候目标网站可能会反复设置一个 cookie，还都是同样的值，这个变量用于忽略此类事件：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/img_8.png)

一般保持默认即可。

五、 定位 Cookie 的变化（debugger）
--------------------------

`home.php?mod=space&uid=441028 v0.6`  
此部分的文档适用于 v0.6 + 版本，如果您本地的版本小于 0.6，请升级版本后再来阅读文档。

从 v0.6 开始，在 Cookie 的值发生改变时打断点变得很复杂，也变得很简单， 复杂是因为引入了事件机制，简单是因为简化了断点规则配置更灵活。

断点规则可以分为`标准规则`和`简化规则`，标准规则是程序底层方便实现处理的， 简化规则是为了用户更方便地配置，通常情况下您只需要了解简化规则就可以了， 当简化规则配置无法满足需求时再来查阅标准规则如何配置。

#### 5.1 debuggerRules

所有的规则都是配置在`debuggerRules`数组中的，在脚本的头部有一个变量：  
![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/45ecea34.png)  
如果找不到的话，可以按 Ctrl+F 按变量的名字搜索：

```
debuggerRules

```

这个变量是一个数组类型，里面存放着一些规则条件，来决定什么情况下会进入断点。

注意，这是一个数组，数组中的规则是或的关系，触发 Cookie 修改事件时， 会顺序匹配每条规则， 只要有一条规则匹配成功就会进入一次断点。

### 5.2 常用配置方式（简化的配置规则）

#### 5.2.1 Cookie 名字过滤

当名为`foo`的 Cookie 发生变化时进入断点：

```
const debuggerRules = ["foo"];

```

上面这种方式指定一个字符串，会按照 Cookie 名字等于给定的字符串去匹配。

注意，此处的完全匹配如果有被 URL 编码的部分也需要先 URL 解码再粘贴到这里， 其它涉及到字符串的地方都一样后面不再赘述。

如果 Cookie 的名字中包含一直变化的部分，比如时间戳、UUID 之类的， 通过名字已经无法定位，则通过正则匹配：

```
const debuggerRules = [/foo.+/];

```

绝大多数情况只需要这两种配置就够了。

下面来实践一下，当打开这个页面

[https://www.ishumei.com/trial/captcha.html](https://www.ishumei.com/trial/captcha.html)

能看到脚本检测到了一些 Cookie 操作：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/36eb394d.png)

其中有个`smidV2`很可疑，于是我们为它添加一个断点：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/5415caa1.png)

修改完`debuggerRules`数组要注意按 Ctrl+S 保存脚本，然后因为油猴是在页面加载的时候注入 JS 代码的， 所以要刷新页面重新注入，当刷新页面的时候就自动进入了断点：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/47c3b465.png)

上图的红色框 A 中是专门传进来的一些变量，通过将鼠标移动到这些变量上查看值， 我们能够大概知道当前断点的一些情况：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/0cf06995.png)

然后就是红色框 B，我们打 Cookie 断点就是为了追踪调用栈定位生成 Cookie 的地方， 红色方框内是本脚本的调用栈，有很明显的`userscript.html`标识， 忽略此部分的调用栈即可。

然后追溯调用栈，能够看到设置 Cookie 的地方：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/33dc63f1.png)

当然看这个栈对我们没用，我们要做的就是逐步往前定位， 直到定位到真正生成 Cookie 的地方，但是呢，本脚本只能帮你打个断点， 后面星辰大海的征程就要靠你自己啦！

#### 5.2.2 Cookie 名字和事件结合

在名为`foo`的 Cookie 被`添加`时进入断点：

```
const debuggerRules = [{"add": "foo"}];

```

在名为`foo`的 Cookie 被`删除`时进入断点：

```
const debuggerRules = [{"delete": "foo"}];

```

在名为`foo`的 Cookie 已经存在但是值被`更新`时进入断点：

```
const debuggerRules = [{"update": "foo"}];

```

条件可以同时指定多个，在`添加和更新`时进入断点，相当于是把删除排除在外：

```
const debuggerRules = [{"add|update": "foo"}];

```

涉及到 Cookie 名字匹配的地方都可以使用字符串或者正则：

```
const debuggerRules = [{"add": /foo_\d+/}];

```

### 5.3 标准的配置规则

上面的简化规则会被转化为标准规则，您也可以直接在`debuggerRules`数组中配置标准规则， 一条标准的规则的格式：

```
{
    "events": "{add|delete|update}",
    "name": {"cookie-name" | /cookie-name-regex/},
    "value": {"cookie-value" | /cookie-value-regex/}
}

```

#### events:

字符串类型，表示此条规则匹配的事件类型，可以是单个事件，比如`add`， 也可以是多个事件，多个事件之间使用`|`来分隔，比如`add|update`， 如果觉得挤的话还可以在`|`两侧加空格，比如`add | update`  
当配置了事件类型时只会匹配给定的事件类型，当不配置此选项时，默认匹配所有事件类型。

#### name:

可以是字符串，也可以是正则，当 Cookie 的名字匹配给定的字符串或者正则时为 true， 此条不可忽略必须配置。

#### value:

可以是字符串，也可以是正则，当 Cookie 的值匹配给定的字符串或者正则时此规则为 true， 可以不配置，不配置则会忽略此选项。

### 5.4 事件类型详解

前面介绍断点规则的配置，多次提到了事件类型， 我们只知道每个事件对应的名字的字符串是啥了， 但是还不知道每种事件意味着底层发生了啥， 本部分就是解释每种事件的实现机制。

Cookie 发生了变化细分为增加 Cookie、删除 Cookie、更新已有的 Cookie 的值，其中每个事件对应着一个事件名字：

*   增加 Cookie（add）
*   删除 Cookie（delete）
*   更新 Cookie（update）

#### 增加 Cookie 事件

Cookie 之前在本地不存在，这是第一次添加。 有可能是第一次访问这个网站 ，也有可能是清除了 Cookie 重新访问，或者是每次访问网站都会生成新的 Cookie，  
甚至可能是网站自己的代码把 Cookie 删了重新添加，这都会触发增加 Cookie 事件。

比如执行下面的代码，这里为了保证 Cookie 之前不存在，在 cookie 的名字中加了时间戳：

```
document.cookie = "foo_" + new Date().getTime() + "=bar; expires=Fri, 31 Dec 9999 23:59:59 GMT; path=/";

```

当我们在控制台运行这行代码的时候，就会触发 Cookie 添加事件：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/10ea2db6.png)

#### 更新 Cookie 事件

当一个 Cookie 在本地已经存在，然后又尝试为它设置值，就会触发更新 Cookie 事件。

比如下面的代码：

```
document.cookie = "foo_10086=blabla; expires=Fri, 31 Dec 9999 23:59:59 GMT; path=/";
document.cookie = "foo_10086=wuawua; expires=Fri, 31 Dec 9999 23:59:59 GMT; path=/";

```

第一条设置 Cookie 的语句会触发 Cookie 新增事件， 第二条设置 Cookie 的语句因为要设置的 Cookie 已经存在了， 所以触发了 Cookie 更新事件。

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/fa06f80c.png)

#### 删除 Cookie 事件

如果前端开发者在设置 Cookie 的时候，给了一个早于当前时间的 expires， 则意味着要删除这个 Cookie，比如一种常见的删除 Cookie 的方式：

```
const expires = new Date(new Date().getTime() - 1000 * 30).toGMTString();
document.cookie = "foo=; expires=" + expires + "; path=/"

```

当我们在控制台运行这段代码时，就会触发 Cookie 删除事件：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/35720fae.png)

由上面也可以看出来，触发 Cookie 删除事件纯粹是检测 expires， 并不会真的去检查这个 Cookie 之前是否存在。

### 5.5 控制事件类型断点是否开启的标志位

前面介绍了在配置 Cookie 断点规则的时候有个事件类型， 事实上每个事件类型都对应着一个此事件类型的断点是否开启的标志位， 这个标志位的优先级是最高的，比如没有开启删除 Cookie 断点的情况下，触发了 Cookie 删除事件，  
会先检查 Cookie 删除断点是否开启标志位，如果是关闭的， 则直接忽略本次事件不再尝试匹配断点规则 （开发者工具控制台上是仍然会打印本次删除事件的日志的）。

所以现在的情况就变得非常复杂了，让我们再捋一下这一个小小的 Cookie 断点要走的流程：

1.  触发 Cookie 增加、删除、修改事件，然后检查对应的事件类型断点是否开启
2.  如果没有开启，则忽略，如果已经开启，则顺序检查是否匹配给定的规则
3.  每匹配成功一条规则，则进入断点一次.

默认情况下只开启了 Cookie 增加事件和 Cookie 修改事件的断点：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/2a5b0f6c.png)

因为一般情况下，增加 Cookie 和更新 Cookie 可以混为一谈，它们都是为 Cookie 赋了一个值， 而大部分情况下我们不会关注 Cookie 被删除的事件，所以这里就这么设置了，如果无法满足你的需求，  
可以自行修改`enableEventDebugger`对应的值。

六、 问题反馈
-------

在使用的过程过程中遇到任何问题，可以在 GitHub 的`Issues`中反馈， 也可以在油猴脚本的评论区反馈，还可以给我发邮件，我看到之后会尽快处理。

二哥:  
[动画表情]

七、FAQ
-----

### 7.1 如何调整控制台打印的字体大小？

从 v0.6 版本开始增加了一个变量用于调整本脚本在控制台打印的日志的字体大小，单位为 px：

![](https://raw.githubusercontent.com/JSREI/js-cookie-monitor-debugger-hook/main/images/README_images/8b47aea4.png)

随着版本迭代，可能不在这个位置了，如果一下找不到，就在代码搜索：

```
consoleLogFontSize

```

然后修改这个变量的值即可。

或者另一种方案，可以在开发者工具控制台按住 Ctrl + 鼠标滚轮缩放调整整体大小， 这是 Chrome 浏览器自带的功能。

### 7.2 为什么 Cookie 明明是 JS 设置的，但是没有 Hook 到？你个大骗子！  :（

在本文的最开始就交代了，本脚本要能够成功的注入到页面的开头部分并且执行才能够 Hook 成功， 对于类似于加速乐第一层那种整个页面只返回一个 script，里面是这种逻辑：

```
<script>
    document.cookie = 这里是一些奇奇怪怪的JS用于计算出Cookie;
    location.href = "跳转走了";
</script>

```

设置了 Cookie 并且立刻就重定向到了新的页面，对于这种操作，有可能会 Hook 不到，这是油猴脚本的问题，如果坚持要 Hook， 可以采用挂代理将本脚本注入到这个 URL 的响应的头部。

最后附上链接 [https://github.com/JSREI/js-cookie-monitor-debugger-hook](https://github.com/JSREI/js-cookie-monitor-debugger-hook)

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)阿清 function hookcookie(){  
    (function () {  
      'use strict';  
      var cookieTemp = '';  
      Object.defineProperty(document, 'cookie', {  
        set: function (val) {  
          if (val.indexOf('目标 cookie 字符串') != -1) {  
            debugger;  
          }  
          console.log('Hook 捕获到 cookie 设置 ->', val);  
          cookieTemp = val;  
          return val;  
        },  
        get: function () {  
          return cookieTemp;  
        },  
      });  
    })();  
} 直接这样不是更简单  
![](https://avatar.52pojie.cn/images/noavatar_middle.gif)hang119 最近看前端，膜拜大佬![](https://avatar.52pojie.cn/data/avatar/002/00/82/35_avatar_middle.jpg) uuwatch 这个是原作者吗，今天刚看见朋友推的，进来膜拜下大佬![](https://avatar.52pojie.cn/images/noavatar_middle.gif) Luker 支持，正需要，感谢楼主![](https://avatar.52pojie.cn/images/noavatar_middle.gif) BTFKM 而你 是真正的英雄我的朋友, 断点断到死![](https://avatar.52pojie.cn/images/noavatar_middle.gif) kukudexin11 试试这个新东西![](https://avatar.52pojie.cn/images/noavatar_middle.gif) shikongliangze 大佬厉害，感谢![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 54cuihua 插眼将来要用![](https://static.52pojie.cn/static/image/smiley/laohu/laohu36.gif)点赞![](https://static.52pojie.cn/static/image/smiley/mogu/lh.gif)![](https://avatar.52pojie.cn/data/avatar/000/41/53/32_avatar_middle.jpg) windflower57521 真的学到了，谢谢![](https://avatar.52pojie.cn/data/avatar/000/95/98/33_avatar_middle.jpg) psych1 在这边做个说明，朋友的工具和文章，委托我在这边发一下! 大家可以多点点 star 