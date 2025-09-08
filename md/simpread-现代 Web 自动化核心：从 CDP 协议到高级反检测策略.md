> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/C7iHwfUgJpTlPynzSNelrA)

本文旨在简明扼要地梳理 Web 自动化中的三大核心问题：

> 底层通信协议的选择
> 
> 自动化框架的选型
> 
> 高级反检测技术的原理与实践

### 一、 浅谈 Chrome DevTools Protocol (CDP)

#### 1. CDP 是什么？

可以把 CDP 想象成是**浏览器内核的专属遥控器或 API**。无论是开发者手动打开 DevTools F12 进行调试，还是自动化框架对浏览器进行操作，其背后与浏览器内核的通信都依赖于 CDP。

它是通过一个 WebSocket 连接，由客户端（如 Playwright、Pyppeteer）向浏览器发送 JSON-RPC 格式的命令，并接收浏览器返回的事件和结果。

#### 2. CDP 能做什么？

答案是**几乎一切**。因为它工作在最低层，所以权限极高。常见应用包括：

*   **DOM 操作**：精准地查找、修改元素，执行 JavaScript 脚本。
    
*   **网络控制**：拦截、修改网络请求和响应，轻松实现 Mock 数据或修改接口返回。
    
*   **深度模拟**：修改设备参数（如 User-Agent、时区、地理位置）、伪造浏览器环境（如 `navigator` 属性、插件、Canvas 指纹）。这是实现高级反检测的关键。
    
*   **输入模拟**：以接近原生的方式派发鼠标、键盘事件，绕过对常规自动化点击的检测。
    

#### 3. 总结

CDP 是一套功能强大、工作在底层的浏览器通信协议。相比传统的 WebDriver，它提供了更精细、更全面的控制能力，尤其在网络拦截和深度环境模拟方面优势明显。

### 二、 如何选择合适的自动化方案？

选择一个合适的框架，直接决定了开发效率和项目的稳定性。

<table><thead><tr><th><section>特性</section></th><th><section>Selenium</section></th><th><section>Playwright</section></th><th><section>Pyppeteer</section></th><th><section>DrissionPage</section></th></tr></thead><tbody><tr><td><strong>核心原理</strong></td><td><section>WebDriver 协议</section></td><td><section>CDP 协议 (为主) &amp; 自有协议</section></td><td><section>CDP 协议</section></td><td><section>WebDriver &amp; CDP 协议混合</section></td></tr><tr><td><strong>开发者</strong></td><td><section>开源社区</section></td><td><section>Microsoft</section></td><td><section>原为 Google, 现社区维护</section></td><td><section>aaron-clark (个人)</section></td></tr><tr><td><strong>浏览器支持</strong></td><td><section>主流浏览器</section></td><td><section>Chromium, Firefox, WebKit</section></td><td><section>仅 Chromium/Chrome</section></td><td><section>Chromium 内核浏览器</section></td></tr><tr><td><strong>主要优势</strong></td><td><section>生态成熟，跨浏览器</section></td><td><section>速度快，API 现代，自动等待</section></td><td><section>轻量，与 CDP 深度集成</section></td><td><section>融合两大协议优点，语法简洁</section></td></tr><tr><td><strong>主要劣势</strong></td><td><section>速度慢，API 相对笨重</section></td><td><section>生态相对年轻</section></td><td><section>仅支持 Chromium，活跃度降低</section></td><td><section>个人项目，生态较小</section></td></tr><tr><td><strong>自动等待</strong></td><td><section>需显式等待</section></td><td><strong>内置</strong></td><td><strong>内置</strong></td><td><strong>智能等待</strong></td></tr><tr><td><strong>推荐场景</strong></td><td><section>传统大规模跨浏览器测试</section></td><td><section>现代 Web 应用，高性能爬虫</section></td><td><section>轻量级爬虫，PDF 生成</section></td><td><section>快速开发，爬虫项目</section></td></tr></tbody></table>

#### **选择建议**

*   **历史方案**：`Selenium` + `undetected_chromedriver` + `stealth.min.js`。通过给驱动打补丁和注入 JS 来伪装浏览器环境。
    
*   **当前主流推荐**：
    

*   `Playwright` + `playwright-stealth`：功能全面，性能卓越，社区活跃，是现代 Web 自动化的首选。
    
*   `DrissionPage`：将 WebDriver 的易用性与 CDP 的强大功能巧妙结合，代码极其简洁，非常适合快速开发和爬虫项目。
    

### 三、 自动化反检测：原理与终极策略

网站检测自动化的核心在于**识别人类行为与机器行为的差异**。

#### 1. 检测原理

主要分为两大类：

*   **环境检测 (Environmental Fingerprinting)**  
    网站通过执行 JS 来检查浏览器的 “指纹” 信息是否“干净”。常见的检测点包括：
    
    **常规对策**：使用 `playwright-stealth` 或 `stealth.min.js` 等 “隐身” 库。它们的原理就是提前注入 JS，将上述这些暴露自动化特征的属性伪装成正常浏览器的值。
    

*   `navigator.webdriver`：在自动化工具控制下，此值通常为 `true`。
    
*   `navigator.plugins`、`navigator.languages` 等浏览器固有属性。
    
*   浏览器函数的 `toString()` 结果，自动化环境下可能被修改。
    
*   Canvas / WebGL 指纹，用于生成设备唯一标识。
    

*   **行为检测 (Behavioral Analysis)**  
    主要针对用户的交互行为，如点击和滑动验证码。
    

*   **滑动轨迹**：机器生成的轨迹往往过于平滑、呈线性，或速度恒定，而人类的轨迹则充满随机抖动和变速。
    
*   **鼠标事件**：人类一次完整的点击 / 滑动，会触发一连串的事件（`mousedown` -> `mousemove` -> `mouseup`）。自动化工具的原生 `click()` 可能事件链不完整，或者事件对象的属性（如 `isTrusted`）暴露了其机器来源。
    

#### 2. 绕过策略与终极方案

当你已经模拟了完美的滑动轨迹，并且使用了 stealth 库来伪装环境，但点击或滑动依然失败，这大概率是触发了更深层次的**事件上下文检测**。

网站会检查触发事件的 “纯净度”，比如检查事件的原型链，或者判断它是否由浏览器内核真实派发。此时，即便是 `page.evaluate()` (等同于 `run_js`) 注入的 JS 来模拟点击也可能失效，因为这种方式生成的事件 `isTrusted` 标志位为 `false`。

**终极策略：使用 CDP 原生事件注入**

这是绕过行为检测的 “杀手锏”。与其用上层 API 或 JS 模拟，不如直接调用 CDP 的底层命令，请求浏览器内核为你“真实地” 执行一次输入。

*   **实现方式**：调用 CDP 的 `Input.dispatchMouseEvent` 命令。
    
*   **核心优势**：通过此方法派发的鼠标事件，是由浏览器内核直接生成和分发的，其事件对象的所有属性、原型链都与真人操作产生的事件**完全一致**，从而无法被检测脚本分辨。
    

**总结**：一套成功的反检测方案是**多层防御**的结合：  
**现代化框架 (Playwright/DrissionPage) + Stealth 隐身插件 (处理环境指纹) + CDP 原生事件 (处理关键行为)**

* * *

<table><thead><tr><th><strong>检测方法</strong></th><th><strong>run_js (page.evaluate)</strong></th><th><strong>Playwright 高层 API</strong></th><th><strong>CDP Input.dispatchMouseEvent</strong></th></tr></thead><tbody><tr><td><strong>event.isTrusted</strong></td><td><section>❌false (致命)</section></td><td><section>✅true</section></td><td><section>✅true (原生)</section></td></tr><tr><td><strong>调用栈</strong></td><td><section>❌可疑 (来自 JS)</section></td><td><section>✅正常</section></td><td><section>✅完美 (来自底层)</section></td></tr><tr><td><strong>navigator.webdriver</strong></td><td><strong>依赖脚本伪装</strong></td><td><strong>依赖脚本伪装</strong></td><td><strong>依赖脚本伪装</strong></td></tr><tr><td><strong>环境指纹一致性</strong></td><td><strong>依赖全面伪装</strong></td><td><strong>依赖全面伪装</strong></td><td><strong>依赖全面伪装</strong></td></tr><tr><td><strong>执行上下文</strong></td><td><section>❌ 页面 JS 环境</section></td><td><section>⚠️Playwright 封装环境</section></td><td><section>✅浏览器内核输入系统</section></td></tr></tbody></table>