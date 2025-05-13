> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WpqLIkehYGtEJU1ZBj42QA)

前言
--

本教程旨在提供一份详尽的、手把手级别的指南，帮助您配置和使用 IDA Pro MCP 进行逆向分析。

本文章作者微信公众号 前沿逆向，严禁私自转载！

当然我认为 ai 一定对带来对攻防对等的收益，所以今天带来优秀作品 IDA Pro MCP 的 macOS 和 Windows 配置手把手教程，以及带来常见的 api 不错的获取渠道，由于这里比较敏感，不打算放到本帖里，打算放到群里，以 markdown 文档形式传播，

此外，文中会提及获取常见大模型 API Key 的合规渠道。工具本身并无好坏之分，关键在于如何运用。无论是革新传统的逆向分析方法论，还是仅仅解决某个具体的技术疑点，IDA Pro MCP 都能在你逆向的时候帮你一把。

如果教程有不懂的地方欢迎留言，我会随时补充！

步骤一：准备大模型 API Key
-----------------

要使用 IDA Pro MCP，首先需要一个大模型服务的 API Key。以下介绍几种国内较为合规的获取渠道，主要以 DeepSeek 为例。您可以根据自己的需求选择其中一种。

### 渠道一：DeepSeek

*   **官方网站:** https://www.deepseek.com/
    
*   **API 管理平台:** https://platform.deepseek.com/usage
    

新用户通常会有免费额度。老用户可以按需充值，最低充值金额一般为 10 元。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibF1AE6KGpY0937003PSPSqAo0PLnkFAawtluNsHz1IShvibRqMk9ibGLA/640?wx_fmt=png&from=appmsg)DeepSeek 充值界面

请按照 DeepSeek 平台的指引完成实名认证和充值。之后，在用户中心或 API 管理页面创建一个新的 API Key。**请务必妥善保管您的 API Key，防止泄露。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibbHpgiciaXtbicOiaPO8tJGVYVtxWnFWonyIfib3o5HwibV2FBeaYzxOO4Iww/640?wx_fmt=png&from=appmsg)DeepSeek API Key 创建界面

获取 API Key 后，我们可以使用 `curl` 命令测试 API 是否可用。请将 `<DeepSeek API Key>` 替换为您自己的 Key。

**示例 Key (已被修改，仅作演示):** `sk-0a78dd928fef46899bf1a45d31666666`

```
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <DeepSeek API Key>" \
  -d '{
        "model": "deepseek-chat",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'

```

**替换后的命令:**

```
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-0a78dd928fef46899bf1a45d31666666" \
  -d '{
        "model": "deepseek-chat",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibxw6jc3wibS4EbCWbsJdyxxWiayPHLVBC2OqQiasgic7HIJdS5y3vu7AHxQ/640?wx_fmt=png&from=appmsg)curl 测试 DeepSeek API 成功截图

如果终端成功输出了 AI 的回复消息，则表示您的 DeepSeek API Key 配置成功。

### 渠道二：硅基流动 (SiliconFlow)

*   **官方网站:** https://siliconflow.cn/
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib4gbtqu6NnftX9JAp1EvIraAnhMgnddZkGibIf6a6XkQLqQnVKibrNZxA/640?wx_fmt=png&from=appmsg) 硅基流动官网截图

登录后，请先完成实名认证，然后在 API 密钥管理页面创建新的 Key。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibwgYC494ia9Yk59icqGgH2h2PAGzsNfL1FwBCboiboI50fr87SUiaFPh2Og/640?wx_fmt=png&from=appmsg)硅基流动 API Key 创建界面

**示例 Key (仅作演示):** `sk-bvygzuetqdtrslnybmzqmcnufvjtmdhgubqiepentgnkncccc`

请确保您的账户有足够的余额。新用户通常有赠送额度，也可以留意官方活动或特定渠道获取优惠。

使用以下 `curl` 命令测试 API，将 `你的key` 替换为您的 SiliconFlow API Key。这里以 `Qwen/Qwen2.5-72B-Instruct` 模型为例，您可以在其模型广场选择其他可用模型。

```
curl https://api.siliconflow.cn/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 你的key" \
  -d '{
        "model": "Qwen/Qwen2.5-72B-Instruct",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'

```

如果返回类似以下的 JSON 响应，则表示请求成功：

```
{"id":"0196c292f544b02f3e5a5365310e8390","object":"chat.completion","created":1747021133,"model":"Qwen/Qwen2.5-72B-Instruct","choices":[{"index":0,"message":{"role":"assistant","content":"Hello! How can I assist you today?"},"finish_reason":"stop"}],"usage":{"prompt_tokens":21,"completion_tokens":9,"total_tokens":30},"system_fingerprint":""}

```

### 渠道三：火山引擎方舟平台 (VolcEngine Ark)

您可以通过火山引擎方舟平台调用包括 DeepSeek 在内的多种大模型。

1.  **访问模型页面:** 前往火山引擎方舟平台的 DeepSeek 模型详情页：https://console.volcengine.com/ark/region:ark+cn-beijing/model/detail?Id=deepseek-r1![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib5FdFd9E1m7zu1B8ibCyUvkdJvWEBUibbpMrx8HSGNzShCUmSfdzKnNtQ/640?wx_fmt=png&from=appmsg)
    
2.  **点击 “推理”:** 在页面右下角找到并点击【推理】按钮。
    
3.  **创建 API Key:** 在弹出的 “快捷 API 接入” 页面，点击【创建 API Key】按钮。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibKiasOxbW3tGqanyq29gfk4JulMBoRW3ypn4rR0iaDe1nDgdrXPa01ceQ/640?wx_fmt=png&from=appmsg)
    
4.  **获取 API Key:** 创建成功后，在页面右下角点击【选择 API Key 并复制】。同时，您还需要记录下 **API 地址 (Endpoint)** 和 **模型 ID (Model ID)**。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibDH2xdO193vIic94RrITx6ms0KfYntALl9UBHavUvdasS9s4cuJ743ZA/640?wx_fmt=png&from=appmsg)
    

集齐 API Key、API 地址和模型 ID 这三个要素后，您就可以在支持火山引擎接口的客户端中配置并使用了。

### 渠道四：阿里云通义千问 (Alibaba Tongyi)

*   **官方网站:** https://www.aliyun.com/product/tongyi
    

阿里云提供了通义千问系列模型，包括性能强大的 Qwen2 等。您可以访问其官网了解详情并申请 API Key。

### 更多渠道

除了上述渠道，百度文心、智谱 AI 等国内厂商也提供了各自的大模型服务。您可以根据自己的需求和偏好，自行探索和选择合适的模型及 API 服务。

步骤二：选择并配置 MCP 客户端
-----------------

### 理解 MCP (Model Context Protocol)

在深入配置之前，让我们先简单理解一下什么是 MCP。

MCP (Model Context Protocol) 是一种协议，旨在让 AI 模型能够更智能地理解和利用当前工作环境中的上下文信息。当您与 AI 对话时，支持 MCP 的客户端（或称为 “载体”）会自动收集相关的上下文信息（例如当前打开的文件、选中的代码片段、项目结构等），并将其提供给 AI 模型。

早期使用大模型时，一个常见的挑战是如何有效地提供上下文。例如，如果您想让 AI 修改代码，仅仅告诉它 “请修改我的代码” 是远远不够的。一个更有效的方法是，将相关的头文件 (`.h`)、源文件 (`.c`, `.cpp` 等) 以及描述业务逻辑的代码片段都提供给 AI。这样，AI 才能更准确地理解您的意图并生成高质量的代码。

MCP 的目标就是自动化这个过程。它通过标准化上下文信息的收集和传输方式，让 AI 应用（如 VS Code 插件、独立客户端等）能够无缝地与 IDA Pro 这类专业工具集成，从而在逆向分析等复杂场景下发挥 AI 的最大潜力。

优秀的 AI 编程助手，如 Cursor，正是通过高效的上下文管理（包括向量化索引、上下文压缩等技术）实现了出色的代码理解和生成能力，从而获得了广泛应用。IDA Pro MCP 遵循了类似的理念，将 MCP 的能力带入了逆向工程领域。

### 客户端一：VS Code 插件 (Cline / RooCode)

Cline 和 RooCode 是两款支持 MCP 的 VS Code 插件，可以将 VS Code 作为 IDA Pro MCP 的客户端。

#### 安装

1.  打开 VS Code。
    
2.  点击左侧边栏的 “扩展” 图标 (Extensions)。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibk5K4TCRdh9J6HQ4EskNLiczQGS1tq9tv2LSMvPKXj4103x6HepFhk3Q/640?wx_fmt=png&from=appmsg)
    
3.  在搜索框中输入 `Cline` 或 `RooCode`。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibmCCXC7ia6IhZhU1ibS83ItgD8PdwGqCHwVIM7ksMELL0icYONl8RAwFmg/640?wx_fmt=png&from=appmsg)
    
4.  找到对应的插件并点击 “安装”(Install)。
    

#### 初始化配置

安装完成后，通常需要进行初始化配置，主要是设置您的大模型 API Key。

1.  打开 Cline 或 RooCode 的设置界面（通常可以通过点击插件图标或使用命令面板 `Ctrl+Shift+P` / `Cmd+Shift+P` 搜索插件名称找到）。
    
2.  选择 “使用自己的 API Key”(Use your own API key)。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibIjib4HzMCMdDpV5eaN11knxQjVlI5RG4B07mVydHerwChUIhicCYl6lQ/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib7FEiaJsv9IItZ3IqTO3tJbic0XL7RBPico3s9DI1eWNNle6cNMKwqmsibw/640?wx_fmt=png&from=appmsg)
    

#### 配置 API Key

根据您选择的 API 服务商进行配置：

*   **DeepSeek:**
    

*   选择 `AI Provider` 为 `DeepSeek`。
    
*   在 `API Key` 字段中填入您的 DeepSeek API Key。
    
*   （可选）根据需要配置 `Model` 等其他参数。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibDjKEPAfLkxMBCsiaib0icOvsj8P27GCL5QUlJ098Zj00vjRFSVibAezYjg/640?wx_fmt=png&from=appmsg)
    
*   配置完成后，尝试发送一条消息进行测试。如果成功收到回复，则配置成功。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibEaSzuImEt5Y0KwkARC0HYhnQ112rslgzpmJ5WO7GSdjHHb8pKqZ6Vw/640?wx_fmt=png&from=appmsg)
    

*   **硅基流动 (SiliconFlow) 或其他 OpenAI 兼容接口:**
    

*   选择 `AI Provider` 为 `OpenAI Compatible`。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibBtibQBwAK6hiafibSRqfB5f3UmUeMW7yHejXYcEEicyH6HEkYAwKyz3YrQ/640?wx_fmt=png&from=appmsg)
    
*   在 `Base URL` 字段中填入 API 服务商提供的 Base URL。对于硅基流动，通常是 `https://api.siliconflow.cn/v1`。
    
*   在 `API Key` 字段中填入您的 API Key。
    
*   在 `Model` 字段中填入您想使用的模型名称（例如 `Qwen/Qwen2.5-72B-Instruct`）。请确保该模型在服务商处可用。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibjT8m1G1AVLlKYVCSwEq7bM8TWuUJdRp9uCn9F86RlrfGoD2amP8ZmQ/640?wx_fmt=png&from=appmsg)
    
*   如果配置遇到问题，可以查阅插件的官方文档或相关社区寻求帮助，例如参考文章：https://deepseek.csdn.net/67b87cf03c9cd21f4cb9c548.html
    

**注意:**

*   Cline 和 RooCode 的配置流程类似。如果在一个插件中配置失败，可以尝试另一个。
    
*   使用这些插件调用大模型会消耗 Token。建议关注 API 服务商的定价和用量，或寻找优惠渠道。
    
*   自行部署本地模型通常效果不如云端 API，且配置复杂，不推荐初学者尝试。
    

### 客户端二：Claude Desktop (需付费订阅)

Claude Desktop 是 Anthropic 官方提供的桌面客户端，也支持 MCP。

*   **官方文档:** https://modelcontextprotocol.io/quickstart/user
    

**安装前提:**

*   需要有效的 Claude 订阅（通常需要付费）。
    
*   需要安装 Node.js 和 uv (一个 Rust 编写的 Python 包管理器)。
    

**安装教程参考:**

*   Node.js: https://blog.csdn.net/Small_Yogurt/article/details/104968169 (或其他可靠教程)
    
*   uv: https://www.cnblogs.com/wang_yb/p/18635441 (或其他可靠教程)
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibJZfU7FQ2ursaID8u2LjibtUZ9mYRJSEqiat8U8ibTd2nVBUmJKadJdXibA/640?wx_fmt=png&from=appmsg)Claude Desktop 截图

由于可能涉及地区限制和付费订阅，这里不做详细展开。如果您已有 Claude 订阅并完成了 Node.js 和 uv 的安装，可以参考官方文档进行配置。

### 客户端三：Cursor (推荐)

Cursor 是一款强大的、以 AI 为核心的代码编辑器，内置了对多种大模型的支持，并且可以直接接入 MCP。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib9LERssdzD7EicOAlichs6Bb4kymL5jrfFUK75asXvhY6JIHMtT1Hib5iaQ/640?wx_fmt=png&from=appmsg)Cursor 编辑器截图

**优点:**

*   通常可以免费使用性能较好的模型（有一定额度限制）。
    
*   用户体验流畅，与 AI 交互自然。
    
*   社区活跃，有多种获取使用次数的方式（请注意合规性）。
    

如果您已经在使用 Cursor，那么将其作为 IDA Pro MCP 的客户端是一个非常方便的选择。

步骤三：部署 IDA Pro MCP (macOS)
--------------------------

本节将指导您如何在 macOS 环境下安装和配置 `ida-pro-mcp` 插件。

### 前置条件

*   **IDA Pro 版本:** `ida-pro-mcp` 插件要求 IDA Pro 8.3 或更高版本。在 macOS 上，常见的 IDA 版本可能较旧。请确保您拥有兼容的 IDA Pro 版本（例如 9.x）。
    
*   **Homebrew:** macOS 的包管理器。如果尚未安装，请先安装 Homebrew。
    

### 1. 安装 Homebrew

如果您的系统尚未安装 Homebrew，请打开终端并执行以下命令：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

```

如果遇到网络问题，可以参考国内镜像源的帮助文档进行安装：

*   清华大学开源软件镜像站: https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/
    
*   其他教程参考:
    

*   https://zhuanlan.zhihu.com/p/547898033
    
*   https://cloud.tencent.com/developer/article/2486985
    

### 2. 安装 Python 3.11

`ida-pro-mcp` 依赖 Python 3.11 或更高版本。IDA Pro 自带的 Python 版本可能不满足要求，因此需要使用 Homebrew 单独安装。

```
brew install python@3.11

```

安装完成后，查看 Python 3.11 的安装信息，特别是安装路径：

```
brew info python@3.11

```

您会看到类似以下的输出：

```
==> python@3.11: stable 3.11.12 (bottled)
Interpreted, interactive, object-oriented programming language
https://www.python.org/
Installed
/opt/homebrew/Cellar/python@3.11/3.11.12 (3,306 files, 62.1MB) *  # <--- 记下这个路径
  Poured from bottle using the formulae.brew.sh API on 2025-05-12 at 17:15:19
From: https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git/Formula/p/python@3.11.rb
License: Python-2.0
==> Dependencies
Build: pkgconf ✔
Required: mpdecimal ✔, openssl@3 ✔, sqlite ✔, xz ✔
==> Caveats
Python is installed as
  /opt/homebrew/bin/python3.11
Unversioned and major-versioned symlinks `python`, `python3`, `python-config`, `python3-config`, `pip`, `pip3`, etc. pointing to
`python3.11`, `python3.11-config`, `pip3.11` etc., respectively, are installed into
  /opt/homebrew/opt/python@3.11/libexec/bin
You can install Python packages with
  pip3.11 install <package>
They will install into the site-package directory
  /opt/homebrew/lib/python3.11/site-packages
... (省略其他信息) ...

```

请记下 `Installed` 下方的路径，例如 `/opt/homebrew/Cellar/python@3.11/3.11.12`。这个路径在后续步骤中会用到。

### 3. (可选) 安装 Node.js 和 uv

如果您计划使用需要 Node.js 或 uv 的 MCP 客户端（如 Claude Desktop），可以通过 Homebrew 安装：

```
brew install node
brew install uv
# 验证安装
node --version
uv --version

```

### 4. 切换 IDA Pro 的 Python 环境

IDA Pro 需要知道使用哪个 Python 解释器。我们需要将其指向刚刚安装的 Python 3.11。

1.  在 “访达”(Finder) 中找到您的 IDA Pro 应用程序。
    
2.  右键点击 IDA Pro 图标，选择 “显示包内容”(Show Package Contents)。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibn5wv5qH92wDofdXlgkBqicCuXIFTQm7grJ9YQCOwcKUxJe6bvLwMc5w/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib4xYGPz9XHwGRGpHlA79g9OSSQPAbPtsKzT3rOUgVedsfico9kLHmfxw/640?wx_fmt=png&from=appmsg)
    
3.  导航到 `Contents/MacOS` 目录。
    
4.  在此目录下打开终端（可以在访达窗口顶部右键点击路径栏，选择 “新建位于文件夹位置的终端窗口”）。
    
5.  执行以下命令，将 `<python_install_path>` 替换为您在步骤 2 中记下的 Python 3.11 安装路径：
    
    ```
    ./idapyswitch --force-path <python_install_path>/Frameworks/Python.framework/Versions/3.11/Python
    
    ```
    
    **示例:** 如果您的 Python 3.11 安装路径是 `/opt/homebrew/Cellar/python@3.11/3.11.12`，则命令为：
    
    ```
    ./idapyswitch --force-path /opt/homebrew/Cellar/python@3.11/3.11.12/Frameworks/Python.framework/Versions/3.11/Python
    
    ```
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibSpxpgdBLWZzibEGicMqhqHABx7YiaOaCp7LOnRAaibZianUJ6uo3GqicBEBw/640?wx_fmt=png&from=appmsg)执行 idapyswitch 命令
    
    成功执行后，通常不会有任何输出。
    

*     
    

*     
    

### 5. 安装 `ida-pro-mcp` Python 包

现在，我们需要在 Python 3.11 环境中安装 `ida-pro-mcp` 包。

1.  打开终端，进入 Python 3.11 的 `bin` 目录。该目录通常位于您记下的安装路径下。 **示例:**
    
    ```
    cd /opt/homebrew/Cellar/python@3.11/3.11.12/bin
    
    ```
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibkhrm2sDrEO9tkDB2sicEib2xVoDxboBqNVBfqnumZDRvTgicjJib48ddDQ/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibx5diazx5Xds2alib8owgIbGw9qc2EN1MLPAtaX3TSGQAPicgxusmSBlFQ/640?wx_fmt=png&from=appmsg)
    

*     
    

3.  使用该目录下的 `pip3.11` 安装 `ida-pro-mcp`。**请确保您的网络可以访问 GitHub。** 需要干啥我不多说了
    
    ```
    ./pip3.11 install --upgrade git+https://github.com/mrexodia/ida-pro-mcp
    
    ```
    

*     
    

### 6. 安装 MCP 插件到 IDA Pro

`ida-pro-mcp` 包安装完成后，还需要将其作为插件安装到 IDA Pro 中。

1.  首先，找到 `ida-pro-mcp` 的安装位置和可执行文件路径。在 Python 3.11 的 `bin` 目录下执行：
    
    ```
    ./pip3.11 show -f ida-pro-mcp
    
    ```
    
    输出会包含类似以下信息：
    
    ```
    Name: ida-pro-mcp
    Version: 1.3.0
    Summary: Vibe reversing with IDA Pro
    Home-page:
    Author: mrexodia
    Author-email:
    License:
    Location: /opt/homebrew/lib/python3.11/site-packages  # <--- 包安装位置
    Requires: mcp
    Required-by:
    Files:
      ../../../bin/ida-pro-mcp                     # <--- 可执行文件相对路径
      ../../../bin/idalib-mcp
      ... (省略其他文件) ...
    
    ```
    

*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    

*     
    

4.  根据 `Location` 和 `Files` 中的可执行文件相对路径，拼接出 `ida-pro-mcp` 可执行文件的绝对路径。 **示例:** `Location` 是 `/opt/homebrew/lib/python3.11/site-packages` 可执行文件相对路径是 `../../../bin/ida-pro-mcp` 拼接后的绝对路径是 `/opt/homebrew/lib/python3.11/site-packages/../../../bin/ida-pro-mcp`，这通常会解析为 `/opt/homebrew/bin/ida-pro-mcp`。
    
    _更可靠的方法是直接使用 `which` 命令（如果 `pip` 将可执行文件放到了 PATH 中）或者直接在 Python 3.11 的 `bin` 目录下查找 `ida-pro-mcp` 可执行文件。_ 假设可执行文件路径为 `/opt/homebrew/bin/ida-pro-mcp`。
    
5.  执行安装命令：
    
    ```
    /opt/homebrew/bin/ida-pro-mcp --install
    
    ```
    
    _(请将 `/opt/homebrew/bin/ida-pro-mcp` 替换为您找到的实际可执行文件路径)_
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibiasum2vDMHZic9ep99PVibZxiacO1yic0iaicmqK5AktTRluF2ibAnbrUIJNrQ/640?wx_fmt=png&from=appmsg)执行 ida-pro-mcp --install 命令
    
    这个命令会将 `ida-pro-mcp` 插件复制到 IDA Pro 的插件目录中。
    

*     
    

### 7. 检查 MCP 客户端连接

安装完成后，启动 IDA Pro。如果一切顺利，`ida-pro-mcp` 插件会自动加载。

然后，启动您选择的 MCP 客户端（Cline, RooCode, Claude Desktop, Cursor 等）。在客户端的 MCP Server 设置或连接部分，应该能看到名为 `github.com/mrexodia/ida-pro-mcp` 的服务器。

*   **Cline:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTiboOibIZMcv17NdXj3vWB1zzAgRlFLhjSVL4WPNR1GT44TQSvKJibmISeA/640?wx_fmt=png&from=appmsg)
    
*   **RooCode:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibmibPsubMcvxwef6IiaTRQfGTwwiag0ib76TlG7y6SQUaZWSjQicZITXrib8w/640?wx_fmt=png&from=appmsg)
    
*   **Claude Desktop:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibnlz2EvM2JK0NTv7IBoaVPEqwCJxKaDBOian0sHACjKFa0RnwbTtaYjg/640?wx_fmt=png&from=appmsg)
    
*   **Cursor:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibwK8djO14uHNpBpxn3MHkpze6ndbBjTq7QoP30LJ2qKBcA0obYicLMVQ/640?wx_fmt=png&from=appmsg)
    

### 理解 MCP 连接机制

所有支持 MCP 的客户端都遵循相同的协议。当 IDA Pro MCP 插件运行时，它会启动一个本地 HTTP 服务器（默认为 `http://localhost:13337`）。MCP 客户端通过连接这个服务器来与 IDA Pro 交互。

在客户端的设置中，通常会有一个管理 MCP Server 连接的地方。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibsSANQgEUYpg4obicVFhMh9C72p47VmotDC3QK0m4ZoWIXnk6IgkibT2Q/640?wx_fmt=png&from=appmsg)MCP Server 配置入口

点击配置，您可以看到类似 JSON 的配置信息，定义了可用的 MCP Server。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib38DYZndPK6Pc2ZufU9XmYDJXSEd3y9a9T9G6C7xicLq5kNFHDGOo1IQ/640?wx_fmt=png&from=appmsg)MCP Server JSON 配置示例

理论上，您可以将这个 JSON 配置复制到任何兼容 MCP 的客户端中使用。

### 实战演示：使用 Cursor 分析示例程序

让我们以一个简单的示例程序（假设名为 `xiaoyuan`）来演示如何使用 IDA Pro MCP 和 Cursor 进行分析。

**准备工作:**

1.  **解决 Python 环境依赖问题:** 切换到 Python 3.11 后，您可能会发现之前安装的一些 IDA 插件（如 Keypatch，依赖 unicorn）无法使用，因为它们安装在旧的 Python 环境中。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibLEwlCSOxq6wKMHhI8xse4bk7mfLA9V8zNJBBMQ01X79KWJsBIGUX1Q/640?wx_fmt=png&from=appmsg)您需要使用 Python 3.11 的 `pip` (即 `pip3.11`) 重新安装这些依赖包。进入 Python 3.11 的 `bin` 目录执行：
    
    ```
    ./pip3.11 install six unicorn yara-python # 根据实际需要的包进行安装
    
    ```
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib83JQR7lFC0HabC1eCDldgvLlMib2gsqgDszUMH2BqO7B7fKhUCRL11g/640?wx_fmt=png&from=appmsg)安装完成后，重启 IDA Pro，插件应该就能正常加载了。
    

*     
    

3.  **启动 IDA Pro 和 MCP Server:** 打开 `xiaoyuan` 程序，并确保 IDA Pro 的 Output 窗口显示 MCP Server 已启动。
    
    ```
    [MCP] Server started at http://localhost:13337
    
    ```
    

*     
    

5.  **连接 Cursor:** 打开 Cursor，确保它已连接到 `github.com/mrexodia/ida-pro-mcp` 服务器。
    

**开始分析:**

1.  在 IDA Pro 中，导航到您感兴趣的函数或代码段。
    
2.  在 Cursor 中，向 AI 提问。由于 Cursor 通过 MCP 连接了 IDA Pro，它可以自动获取 IDA 当前的上下文信息（如当前函数、地址等）。
    
    **示例提问:** "@ida 请分析当前函数的功能"![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibN7IOUV3AAQUr3bKkFbmsBPVia8Kul1We5orGpiamABzTU2KxDZU2XMiaw/640?wx_fmt=png&from=appmsg)
    
    Cursor 会将问题和从 IDA 获取的上下文一起发送给大模型。
    
3.  **查看分析结果:** AI 模型会返回分析结果。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib5Ef4DTybIzBNxibflbDDcDP4ia35ZseBibaicOMZ95HMmPichpr1kiaN2Flw/640?wx_fmt=png&from=appmsg)
    
4.  **让 AI 添加注释:** 您可以进一步要求 AI 将分析结果作为注释添加到 IDA Pro 中。
    
    **示例提问:** "@ida 将上述分析结果作为注释添加到函数起始位置"![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibOrX3wCzOJcvSsibJMnlDTbKuHx6XXHqOdf9IffDyjKsQybj8yibibP0FA/640?wx_fmt=png&from=appmsg)
    
5.  **在 IDA Pro 中查看效果:** 切换回 IDA Pro，您会看到 AI 添加的注释。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibzFAEVa4JtKt3soKceImdkrltV3RNprxKd0Bh2Lrk5KtY90EQRWe1Cw/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibKWf5KzWH2YjyxxewJct272CZTYyh5AvJqMI2EiaaicfQBuWia3PAsicf2Q/640?wx_fmt=png&from=appmsg)
    

**案例分析 (xiaoyuan):**

假设 `xiaoyuan` 实现了一个哈希算法。通过上述步骤，Cursor (背后的大模型) 分析了从 IDA Pro 获取的函数代码，并给出了它的功能判断。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibVrHXtTv6icURQhYbev1Cd8nnJKzUwbFBRLHEfqNj5loV51UHZZz8sSQ/640?wx_fmt=png&from=appmsg)Cursor 分析 xiaoyuan 为哈希函数

这个例子展示了 AI 如何借助 MCP 提供的上下文信息，辅助逆向工程师理解代码逻辑，甚至自动添加注释，极大地提高了分析效率。

步骤四：部署 IDA Pro MCP (Windows)
----------------------------

本节介绍在 Windows 环境下配置 IDA Pro MCP 的步骤。我们将以 IDA Pro 8.3 版本为例，因为许多用户可能仍在使用此版本以兼容其他插件。

### 1. 安装 Python 3.11

与 macOS 类似，Windows 环境也需要安装 Python 3.11 或更高版本。

1.  **下载安装程序:** 访问 Python 官方网站下载 Python 3.11 的 Windows 安装程序。建议选择 64 位版本。
    

*   官方下载页面: https://www.python.org/downloads/release/python-3110/ (请选择合适的 Windows installer)
    

3.  **运行安装程序:**
    

*   **重要:** 在安装界面，**不要**勾选 "Add Python 3.11 to PATH" 选项，以避免与系统中可能存在的其他 Python 版本冲突。
    
*   选择 "Customize installation" (自定义安装) 或记下 "Install Now" 下方显示的默认安装路径。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibPdPsVHuEAa4LpELoQp1CD7GhJBNjBHZic574lEKvLOETyt1AkpXa9BQ/640?wx_fmt=png&from=appmsg)
    
*   完成安装。
    

5.  **查找 Python 安装路径:** 如果您不确定安装路径，可以通过以下方法查找：
    
    **替代方案 (特定 IDA 版本):** 如果您使用的是集成了 Python 环境的 IDA Pro 版本，它可能自带了合适的 Python 3.11 环境。您可以直接使用其内置的 Python 路径。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibwg9Da4YXaMLKqPTxwI13wTNk1ccLjY091qA0zpGZsliaFHcjKxm6y3Q/640?wx_fmt=png&from=appmsg)
    

*   在 Windows 开始菜单中搜索 "Python 3.11"。
    
*   右键点击 "Python 3.11 (64-bit)" 或类似条目，选择 "打开文件所在的位置"(Open file location)。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibGx4brXj4VXLJbPAxyvURRrqF1wqafdK38gZzmfjf3ToicY2EG2XkNeg/640?wx_fmt=png&from=appmsg)
    
*   这通常会打开一个包含快捷方式的文件夹。再次右键点击 Python 快捷方式，选择 "打开文件所在的位置"。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibxwPzAibb7DP4OtpAZyJt2aNZdFC7SLs0mUfYVdsl2BDNVHK7ZkVseVw/640?wx_fmt=png&from=appmsg)
    
*   这样就能找到 Python 的实际安装目录。记下 `python.exe` 所在的完整路径。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibEaSzuImEt5Y0KwkARC0HYhnQ112rslgzpmJ5WO7GSdjHHb8pKqZ6Vw/640?wx_fmt=png&from=appmsg)**示例路径:** `C:\Users\YourUsername\AppData\Local\Programs\Python\Python311\` (请替换 `YourUsername`) **`python.exe` 路径:** `C:\Users\YourUsername\AppData\Local\Programs\Python\Python311\python.exe` **`python3.dll` 路径:** `C:\Users\YourUsername\AppData\Local\Programs\Python\Python311\python3.dll` (或 `python311.dll`)
    

### 2. 切换 IDA Pro 的 Python 环境

与 macOS 类似，需要告知 IDA Pro 使用新安装的 Python 3.11。

1.  打开命令提示符 (cmd) 或 PowerShell。
    
2.  执行 `idapyswitch.exe` 命令。该程序位于 IDA Pro 的安装目录下。您需要提供 IDA Pro 的路径和 Python 3.11 的 `python3.dll` (或 `python311.dll`) 的路径。
    
    ```
    "C:\Path\To\Your\IDA Pro\idapyswitch.exe" --force-path "C:\Path\To\Your\Python311\python3.dll"
    
    ```
    
    **请务必将命令中的路径替换为您自己的实际路径！**
    
    **示例:**
    
    ```
    "C:\Users\jiqiu2021\Desktop\IDA_Pro_v8.3_Portable\idapyswitch.exe" --force-path "C:\Users\jiqiu2021\AppData\Local\Programs\Python\Python311\python3.dll"
    
    ```
    
    成功执行后通常没有输出。
    

*     
    

*     
    

### 3. 安装 `ida-pro-mcp` Python 包

使用 Python 3.11 的 `pip` 来安装 `ida-pro-mcp`。

1.  打开命令提示符或 PowerShell，并导航到 Python 3.11 的 `Scripts` 目录（如果 `pip` 不在系统 PATH 中），或者直接使用 `python.exe -m pip`。 **示例 (使用 `python.exe -m pip`):** 这里一定要能访问 github 一定要做我们心里想的那件科字开头的东西
    
    ```
    "C:\Path\To\Your\Python311\python.exe" -m pip install --upgrade git+https://github.com/mrexodia/ida-pro-mcp
    
    ```
    
    _(请替换 Python 路径)_
    
    **注意:** 确保您的网络可以访问 GitHub。如果遇到网络问题，可能需要配置 pip 的代理或使用国内镜像源。
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibQhoBaBTJuiaKK7LtT92wBribHialYDKAWsBvy2jibiaxmAOxJQTwcExS6vw/640?wx_fmt=png&from=appmsg)Windows 安装 ida-pro-mcp
    

*     
    

### 4. 安装 MCP 插件到 IDA Pro

与 macOS 类似，需要执行安装命令将插件复制到 IDA Pro。

1.  找到 `ida-pro-mcp.exe` 可执行文件的路径。可以使用 `pip show -f ida-pro-mcp` 命令查找。
    
    ```
    "C:\Path\To\Your\Python311\python.exe" -m pip show -f ida-pro-mcp
    
    ```
    
    _(请替换 Python 路径)_
    
    输出会包含类似信息：
    
    ```
    Name: ida-pro-mcp
    Version: 1.3.0
    ...
    Location: C:\Users\jiqiu2021\AppData\Local\Programs\Python\Python311\Lib\site-packages # <--- 包安装位置
    ...
    Files:
      ..\..\Scripts\ida-pro-mcp.exe  # <--- 可执行文件相对路径
      ..\..\Scripts\idalib-mcp.exe
      ...
    
    ```
    

*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    
*     
    

*     
    

4.  根据 `Location` 和 `Files` 中的相对路径，拼接出 `ida-pro-mcp.exe` 的绝对路径。 **示例:** `Location`: `C:\Users\jiqiu2021\AppData\Local\Programs\Python\Python311\Lib\site-packages` 相对路径: `..\..\Scripts\ida-pro-mcp.exe` 绝对路径: `C:\Users\jiqiu2021\AppData\Local\Programs\Python\Python311\Scripts\ida-pro-mcp.exe`
    
5.  执行安装命令：
    
    ```
    "C:\Path\To\Your\Python311\Scripts\ida-pro-mcp.exe" --install
    
    ```
    
    _(请将路径替换为实际的可执行文件路径)_
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibwr2vjutOicEtEWaATIzpNKbw6Rm2Nl2maT0G9zNgibDjwO9uTX775dEg/640?wx_fmt=png&from=appmsg)Windows 执行安装命令
    

*     
    

### 5. 检查安装情况和效果展示

1.  **启动 IDA Pro:** 打开 IDA Pro，加载一个文件。检查 Output 窗口是否显示 MCP Server 已启动。![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibVA9pZRbpoTpxZd63ME3adCJCaHmW32d6hcED08UzVfkebiaqHb6Lia1g/640?wx_fmt=png&from=appmsg)
    
2.  **启动 MCP 客户端:** 打开您选择的 MCP 客户端 (Cline, RooCode, Cursor, Claude Desktop 等)。
    
3.  **检查连接:** 确认客户端已连接到 `github.com/mrexodia/ida-pro-mcp` 服务器。
    

*   **Claude Desktop:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibrr3TyHoAt0HQOXsIhdphKducNlOnwe5Xlbd2nwKVNcHHEb2awnwK1g/640?wx_fmt=png&from=appmsg)
    
*   **Cursor:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib0zfGZA5WUpN3lRibJss8rAoBmN6diamNbSeWVStDib98NHUA61IW4bvyw/640?wx_fmt=png&from=appmsg)
    
*   **Cline:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibqyMOSo5YrdfhUzia4ibicVYicx2FFuJNGlKpibZSllVPB7m2zD8zCV6Z85A/640?wx_fmt=png&from=appmsg)
    

5.  **进行测试:** 尝试使用客户端与 IDA Pro 交互，例如让 AI 分析函数或添加注释。
    

*   **Claude 示例:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibAveKsTa55BNpI8GVicu1HTLduu5nl3FKQqoS6pctic0qGYfibFcCO64pQ/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTib8BhRtNmVu16Gn3POT067ib6ibiaXbTJ4hlzcwF5xaZUqtaKvLicDQyEjHQ/640?wx_fmt=png&from=appmsg)
    
*   **Cline 示例:**![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibYuibf2AMAQd3v579lLHibXhfSjVdumOPJt7qmTWn8yKiacQiaugdia51FWA/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/3c9Ae6oWciaGeZTkrL3gxCEd46EgflsTibMroAM3dZ5JX34j5NTUFMq7rocuibibo8UawQspOe5ghriaNzSmemv5bdg/640?wx_fmt=png&from=appmsg)
    

结语
--

AI 的浪潮已经到来，将其融入逆向工程的工作流是大势所趋。IDA Pro MCP 为我们提供了一个强大的框架，将 AI 的分析能力与 IDA Pro 的专业功能相结合。

希望本教程能帮助您顺利配置并开始使用这一利器。我们鼓励您积极探索，将 AI 技术应用于实际的逆向分析任务中，发掘更多可能性。

未来，我们的公众号将持续关注 AI 在逆向工程领域的应用，并分享更多实用的案例和教程。如果您在配置或使用过程中遇到任何问题，或有任何想法和建议，欢迎通过公众号 “前沿逆向” 与我们联系交流。

我是看雪棕熊🐻，爱你么么么哒～ 上面的文章被 ai 优化过了，如果想要原稿，请私信我获取。

想加交流群，微信公众号前沿逆向私信我就可以啦～