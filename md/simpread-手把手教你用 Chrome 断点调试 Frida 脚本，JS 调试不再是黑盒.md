> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4sxqv5dp3_7bexZL-e4m8Q)

> 版权归作者所有，如有转发，请注明文章出处：https://cyrus-studio.github.io/blog/

使用 Chrome 断点调试 JS
=================

只需要在执行 frida 命令时，加上下面参数即可：

```
--runtime=v8 --debug

```

参数说明：

*   • --runtime=v8，指定 Frida 使用 V8 引擎（Google Chrome 使用的 JS 引擎）来运行 JS 脚本。默认 Frida 使用 QuickJS 引擎（轻量但功能有限，调试能力较差）。
    
*   • --debug，启用调试模式，输出更多细节信息，包括：脚本加载日志、错误栈追踪、JS 异常信息等
    

关于 Frida 的详细使用参考：一文搞懂如何使用 Frida Hook Android App[1]

比如：

*   • 启动应用并附加到当前启动进程
    

```
frida -H 127.0.0.1:1234 -l classloader_utils.js -f com.shizhuang.duapp --runtime=v8 --debug

```

*   • 附加到当前设备的前台应用
    

```
frida -H 127.0.0.1:1234 -F -l classloader_utils.js --runtime=v8 --debug

```

当你看到：Chrome Inspector server listening on port 9229，这说明你已经成功开启了一个 Frida（V8 模式）调试服务

```
(anti-app) PS D:\Python\anti-app\frida_java> frida -H 127.0.0.1:1234 -F -l classloader_utils.js --runtime=v8 --debug
     ____
    / _  |   Frida 14.2.18 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
Chrome Inspector server listening on port 9229

[Remote::**]->

```

在 Chrome 浏览器地址栏输入：

```
chrome://inspect

```

点击 Open dedicated DevTools for Node

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jnciavUXbfl0wG0IFAcaDGM9dibvoED1B2wOQc9oFbmTtia36XibfTn10hwFScoIdRaXbFegymyT6xdtg/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image1.png

切换到 Sources Tab，Ctrl + Shift + P 加载调试脚本，在 Node 中打开要调试的脚本，可以看到源码，下断点调试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jnciavUXbfl0wG0IFAcaDGM9cSRibXLwjvId7ozmeDgwoafxp67Y5CPphaP8wx23NSeUykWldQbomVQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image2.png

**动态调试相关快捷键：**

<table><thead><tr><td><section>快捷键（Windows）</section></td><td><section>快捷键（Mac）</section></td><td><section>功能说明</section></td></tr></thead><tbody><tr><td><section>F12 / Ctrl + Shift + I</section></td><td><section>Cmd + Option + I</section></td><td><section>打开 / 关闭开发者工具</section></td></tr><tr><td><section>Ctrl + P</section></td><td><section>Cmd + P</section></td><td><section>快速打开文件（搜索源码）</section></td></tr><tr><td><section>Ctrl + Shift + F</section></td><td><section>Cmd + Option + F</section></td><td><section>全局搜索源码内容</section></td></tr><tr><td><section>F8</section></td><td><section>F8</section></td><td><section>继续执行（Resume）</section></td></tr><tr><td><section>F10</section></td><td><section>F10</section></td><td><section>单步执行（Step Over）</section></td></tr><tr><td><section>F11</section></td><td><section>F11</section></td><td><section>进入函数（Step Into）</section></td></tr><tr><td><section>F9</section></td><td><section>F9</section></td><td><section>单步调试（Step）</section></td></tr><tr><td><section>Shift + F11</section></td><td><section>Shift + F11</section></td><td><section>跳出函数（Step Out）</section></td></tr><tr><td><section>Ctrl + F8</section></td><td><section>Cmd + F8</section></td><td><section>切换断点的启用 / 禁用状态</section></td></tr><tr><td><section>Esc</section></td><td><section>Esc</section></td><td><section>切换 Console 面板</section></td></tr><tr><td><section>Ctrl + /</section></td><td><section>Cmd + /</section></td><td><section>注释 / 取消注释选中代码</section></td></tr><tr><td><section>Alt + ← / →</section></td><td><section>Ctrl + - / +</section></td><td><section>源码导航前进 / 后退</section></td></tr></tbody></table>

在 Console 中可以直接调用 js 中的函数

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hibUrj5WA1jnciavUXbfl0wG0IFAcaDGM9K6Iicw2M1ibsicSppSvFQF7Aic7Qn05BTiabYtyJAcAQouk241aAlzosevQ/640?wx_fmt=png&from=appmsg&watermark=1)word/media/image3.png

原理拆解（结合 Frida 源码）
=================

用 Chrome 断点调试 Frida 脚本，本质上是利用 Frida 提供的 V8 引擎（--runtime=v8）和 Chrome DevTools 协议（Inspector Protocol）建立调试通道。

增加 --debug 参数会开启 Chrome DevTools 协议服务（WebSocket 监听 9229 端口），Chrome DevTools 就可以像调试 Node.js 一样对 Frida 脚本进行断点、单步、变量查看等操作。

```
def _on_script_created(self, script: frida.core.Script) -> None:
    if self._enable_debugger:
        script.enable_debugger()
        self._print("Chrome Inspector server listening on port 9229\n")

```

https://github.com/frida/frida-tools/blob/8ab0cf7b0c3c93952fc64052b17d9cdc80a580e9/frida_tools/application.py#L648

Frida 支持 qjs（默认） 和 v8 JS 引擎，在 frida-core 源码中可以看到：

```
const OptionEntry[] options = {
{ "device", 'D', 0, OptionArg.STRING, ref device_id, "connect to device with the given ID", "ID" },
{ "file", 'f', 0, OptionArg.STRING, ref spawn_file, "spawn FILE", "FILE" },
{ "pid", 'p', 0, OptionArg.INT, ref target_pid, "attach to PID", "PID" },
{ "name", 'n', 0, OptionArg.STRING, ref target_name, "attach to NAME", "NAME" },
{ "realm", 'r', 0, OptionArg.STRING, ref realm_str, "attach in REALM", "REALM" },
{ "script", 's', 0, OptionArg.FILENAME, ref script_path, null, "JAVASCRIPT_FILENAME" },
{ "runtime", 'R', 0, OptionArg.STRING, ref script_runtime_str, "Script runtime to use", "qjs|v8" },
{ "parameters", 'P', 0, OptionArg.STRING, ref parameters_str, "Parameters as JSON, same as Gadget", "PARAMETERS_JSON" },
{ "eternalize", 'e', 0, OptionArg.NONE, ref eternalize, "Eternalize script and exit", null },
{ "interactive", 'i', 0, OptionArg.NONE, ref interactive, "Interact with script through stdin", null },
{ "development", 0, 0, OptionArg.NONE, ref enable_development, "Enable development mode", null },
{ "version", 0, 0, OptionArg.NONE, ref output_version, "Output version information and exit", null },
{ null }
};

```

https://github.com/frida/frida-core/blob/57d2a83548187d6a93fe2981bcd710fd54d2de11/inject/inject.vala#L24

Frida 提供 V8 和 QuickJS 两个 runtime，是为了在追求高性能（V8）和低资源消耗、快速启动、移植性强（QuickJS）之间做出权衡，并支持更强的调试能力（V8 支持 Chrome DevTools）。

#### 引用链接

`[1]` 一文搞懂如何使用 Frida Hook Android App: _https://cyrus-studio.github.io/blog/posts/%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-frida-hook-android-app/_