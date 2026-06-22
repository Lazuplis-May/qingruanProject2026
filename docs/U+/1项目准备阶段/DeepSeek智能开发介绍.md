任务详情

DeepSeek智能开发介绍

建议工时：

1

任务描述

**1. 任务描述**

近年来随着人工智能爆发式的发展，各行各业的都慢慢尝试运用AI来辅助工作提高效率。AI辅助编程的优势不仅体现在提高开发效率和降低开发门槛上，还体现在提升代码质量、增强团队协作、应对复杂项目以及快速适应技术变化等多个方面。随着技术的不断进步，AI将成为开发者不可或缺的助手，帮助他们更好地应对未来的挑战。本任务将介绍使用Cline结合DeepSeek大模型来进行辅助开发的方法。

**2. 任务知识**

**知识点：**DeepSeek模型介绍、Cline介绍**、**Cline结合DeepSeek大模型、提示词的使用

**重点：**Cline结合DeepSeek大模型、提示词的使用

**难点：**提示词的使用

**3. 任务成果**

Cline结合DeepSeek大模型，完成一个简单的智能编程案例。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F10f32c47781147ea90bbfe0366893d2c.png)

图1 DeepSeek大模型辅助编程



任务指导

##### 1. DeepSeek的概念

**DeepSeek 人工智能模型体系概述**
​
DeepSeek 是中国杭州深度求索人工智能基础技术研究有限公司自主研发的先进人工智能大模型技术体系，包含多版本产品矩阵。其中，"DeepSeek R1" 推理优化模型与 "DeepSeek V3" 第三代通用模型构成核心产品序列，分别面向专业化推理场景与通用智能应用领域。​
**核心模型技术特性对比​**
**1）DeepSeek R1 推理优化模型​**

该模型以卓越的逻辑推理能力为核心优势，针对复杂任务的回答准确率显著高于 V3 版本。其独特之处在于具备可解释性推理功能，在问题解答过程中会完整呈现思维链路，便于用户追踪模型的推理逻辑与问题解决路径。这种深度推理能力使其在软件项目功能设计、代码架构规划及编程实现等技术场景中表现突出，成为开发者的高效工具。
**2）DeepSeek V3 第三代通用模型​**

2025 年 3 月 24 日发布的 DeepSeek-V3-0324 版本，创新性融合了 DeepSeek-R1 模型训练中采用的强化学习技术，实现推理性能的跨越式提升。在数学逻辑推理、代码生成等专业评测集上，该版本得分超越 GPT-4.5，标志着其在复杂问题处理能力上达到国际领先水平。
 

**交互模式与开发痛点**

当前 DeepSeek 主要通过问答式交互提供服务，用户可通过自然语言指令获取解决方案。例如，在 HTML 页面设计场景中，用户只需输入 "帮我设计一个 xxx 的 HTML 页面，并给出代码"，即可快速获得生成代码并嵌入项目文件。然而，面对结构复杂、文件众多的大型开发项目时，这种交互模式暴露出显著局限性，局限如下：

上下文管理低效：需多次手动复制粘贴代码片段，且需频繁传递历史代码作为输入上下文以维持项目逻辑连续性

开发过程追溯困难：分散的聊天记录导致代码生成历史难以系统还原，不利于项目版本管理与协作开发
**Cline 工具赋能开发流程**

****针对上述痛点，本项目引入 Cline 作为中间工具平台，构建智能化开发桥梁。通过 Cline 的桥梁作用，DeepSeek 模型的智能输出与实际开发流程实现无缝对接，有效提升复杂项目的开发效率与管理水平，构建从需求分析到代码实现的全链路智能化开发体系。

##### 2. **Cline 插件简介**

VSCode 通过 Cline 插件与 DeepSeek 大模型整合。

Cline 是一款开源的AI编程助手，能够与 VSCode 无缝集成，支持多种编程语言（如 Python、JavaScript、Java、C++ 等），并借助大语言模型（如 DeepSeek、Claude 3.5 Sonnet、Google Gemini 等）为开发者提供智能代码补全、错误检测与修复、代码生成和项目管理等功能。

**主要功能：**

智能代码补全与生成：
    Cline 能根据上下文提供精准的代码建议，快速生成代码片段或完整的函数，支持多种编程语言。

自动错误检测与修复：
    实时检测代码中的语法错误、运行时问题，并提供修复建议，显著减少调试时间。

代码重构与优化：
    支持代码重构功能，帮助开发者优化代码结构，提升代码可读性和维护性。

终端命令执行：
    Cline 可以在 VSCode 的终端中执行命令，例如安装依赖、运行脚本、启动开发服务器等，简化开发流程。

Web 开发支持：
   通过浏览器启动网站，进行交互操作（如点击、输入、滚动），捕获截图或控制台日志，帮助开发者调试和优化 Web 应用。

多语言模型支持：
    支持多种语言模型，用户可以根据需求选择免费或付费的高性能模型。

项目理解与任务执行：
    Cline 能分析项目结构，根据任务描述自动完成复杂的编程任务，例如初始化项目、添加功能模块等。

安全交互：
    所有文件更改和终端命令都需要用户授权，确保操作的安全性和透明性。

扩展能力：
    通过 Model Context Protocol (MCP)，Cline 可以连接外部服务（如 GitHub）、控制浏览器、访问数据库等，进一步扩展其功能。

##### 3. VSCode接入DeepSeek大模型

**1）安装Cline插件**

在VSCode插件市场中找到Cline插件并安装。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250312%2Fbfface7d25094730b7028a409bc8d19f.png)

图1 安装Cline插件

**2）在 Cline 中配置 DeepSeek 的 API key**

安装成功后，在左侧会出现Cline图标，点击该图标进入Cline页面，首次进入后会提示配置模型，也可以点右上角的小齿轮键打开配置界面，可分别配置Plan Model和Act Model。“Plan”模式用于做规划，“Act”模式用于执行（编写代码），在实现一个功能时，首先切换至Plan模式，规划好如何实现此功能，然后再切换至Act模式，进行代码编写。对于部分简单的任务也可以直接切换为Act模式来辅助编程。

在配置界面中选择“API Provider”为“DeepSeek”，DeepSeek API Key 为对应的 API Key。建议 Plan Mode 中将 Model 配置为“deepseek-reasoner”（R1模型），Act Mode中则配置“deepseek-chat”（V3模型）。

点击“Save”保存模型。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250424%2F2f2fd658a07b495280c5940b2f5189f1.png)

图2 在Cline插件中配置DeepSeek的API key

接着点击下方箭头设置Cline权限：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F5883358cd3114716af30397cd16a497e.png)

图3 配置Cline插件

按照下图设置读取文件和目录、编辑文件、执行安全命令、使用浏览器等权限。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250424%2Ffe995b6112f2498c86832082019bf0df.png)

图4 配置Cline权限

 

**3）编码辅助**

点击下方的“Plan”按钮将模式切换为“Plan”模式，然后使用@符号指明操作的文件，并编写提示词，以生成购物网站首页为例，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F0742eb192b47411ca68cf59f41fecb2b.png)

图5 使用Cline创建页面

发现回答内容均为英文，并回答说需要更多的偏好才能进行设计。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F9cf16031d72b4cf4bee132c590b0e85e.png)

图6 Cline生成计划

我们可以在聊天框中继续输入以下内容：

```
页面需要适配手机端，页面包括轮播图，商品分类，热门商品，请用中文回答我的问题。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F3ce904379b87458ead5708dec046391c.png)

图7 Cline生成计划

可以看到Cline已经设计出初步的设计方案，可以继续与Cline对话，修改设计方案，或者切换至Act模式来编写代码。

接下来点击右下方的“Act”按钮切换至Act模式。等待片刻后，可以看到Cline已经自动编写了html对应的代码，可以点击“save”保存代码，或通过聊天栏让其修改代码。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F10f32c47781147ea90bbfe0366893d2c.png)

图8 代码生成

在实现一个项目功能时，一般需要多次编写和调整代码，点击“save”按钮后，Cline会自动创建css文件并编写对应的代码，如下图所示。

点击“save”保存代码，或者让其修改代码，经多次迭代逐步完善代码。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fa1b906a67036415993f51631c5c0d55f.png)

图9 代码生成

由于大模型有上下文的限制，且Cline工作时要携带大量代码作为上下文，问答或代码编写不能无限制。点击“Task”可以查看当前的上下文使用量，可以看到当前使用了18K上下文，共有64K的大小，所以在进行问答时，问题一定要明确，尽量避免多次问答导致上下文超过限制。当上下文内容超出大小限制时，可以点击上方“+”按钮新建一个对话来进行辅助编写代码。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Ff09b42d3e9164caa978cccebd180f388.png)

图10 代码生成完毕