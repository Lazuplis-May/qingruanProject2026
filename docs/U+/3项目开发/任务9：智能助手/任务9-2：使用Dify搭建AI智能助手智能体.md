任务详情

任务9-2：使用Dify搭建AI智能助手智能体

建议工时：

1

任务描述

**1. 任务描述**

本任务通过Dify搭建智能助手智能体。

**2. 任务知识**

**知识点 ：**Dify聊天助手搭建、提示词编写、工具调用、Function Calling；

**重点 ：**提示词编写、工具调用；

**难点 ：**提示词编写、工具调用；

**3. 任务成果**

本任务成果为AI智能助手智能体搭建：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F2383b651abc44bc8b71f2e782fc7a1f5.png)

图1 智能体搭建页面



任务指导

#### 1.Dify智能体

在 Dify 中，智能体相较于聊天助手，突破性地新增了工具调用能力，这一核心功能如同赋予智能体 “数字工具箱”，是其实现能力跃迁的关键。正如人类会使用工具，遇到复杂的算数问题，可以使用计算器辅助计算，遇到一个未知的问题可以通过搜索引擎来获取知识。智能体通过调用 API 接口、数据库查询、代码执行等多样化工具，突破了单纯语言交互的局限，将知识转化为实际行动。

在Dify中实现工具调用的方式由两种：

ReAct：ReAct方式是绝大部分模型普遍支持的一种能力，在模型执行时，通过“思考-行动-观察”的步骤循环迭代，最终实现效果。其原理是在模型前，为其添加工具列表相关的提示词，让其思考是否需要调用工具，如果是调用工具则执行工具的功能，并返回给大模型结果，直至大模型确认输出。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F2ed5d25dfaab496da1278818caf68e5e.png)

图1 ReAct工作流程

ReAct模型不需要模型进行适配，支持大部分模型，但是由于在执行时需要多轮问答才能最终输出结果，其执行速度和消耗token量也非常多。

Function Calling：Function Calling方式是通过配置大模型，使其原生支持在思考中调用工具的过程，在大模型思考过程中发现自身能力无法支持回答内容，则会查看自己所拥有的工具箱，中断思考并调用工具，调用结束后继续思考，例如使用豆包，询问“今天青岛的天气”。豆包无法获取今天的天气，则通过搜索引擎工具来搜索具体信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2Feaec5ecc02cb4d5cb66305acfc0f5951.png)

图2 Function Calling

Function Calling方式相比于ReAct，执行效率高，但是需要模型支持此能力。DeepSeek支持Function Calling能力。

在本任务中就可以将之前任务开发的工作流发布为工具，智能体再通过工具调用的方式来实现多种额外功能。

#### 2.“智能助手”智能体搭建

1）工具发布：在开始智能体搭建前，首先将之前任务中配置的工作流发布为工具以供智能体调用。包括“数据管理工作流”、“信息管理与预测”、“方案定制”、“打卡分析”。

在工作流中的发布按钮下点击“发布为工具”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F4513ea67a0df4e858c525d0089415d2e.png)

图3 发布工具

以数据管理工作流为例，配置工具名为“execute_SQL”，然后点击保存即可。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F4f9f714d90994c609ae702b703d198a2.png)

图4 配置工具名

其他工作流也同理，调用名称可自定义。

2）创建智能体

在Dify中创建空白应用，选择类型为“Agent”，名称为“智能助手”，然后点击“创建”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F9910847b1fe0455d9d20ca29af6cf6c3.png)

图5 创建智能体

3）添加知识库和工具

首先在智能体中添加知识库，将之前创建的知识库都添加进去，这样智能体就掌握了数据库结构与糖尿病专业信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F1b32f04cba59407ba8f487204e7f5f78.png)

图6 添加知识

接着添加工具，将步骤1中发布的工具全部添加到工具列表中。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F37c21c59e0e04200a6a7c3067ff45a7a.png)

图7 添加技能

继续编写提示词，提示词可以按照“角色”、“技能”、“限制”的格式来编写。首先编写角色部分的内容，智能体的角色为“糖尿病生活助手”，除此之外还需要配置用户的相关信息，让该智能体了解用户实际情况，角色配置提示词如下：

```
# 角色
你是一个专业的糖尿病生活助手，能够凭借丰富的知识和经验，为糖尿病患者提供全面且精准的服务，助力他们更好地管理病情、改善生活质量。
用户信息包括:
用户ID:{{userId}}
性别：{{sex}}
年龄：{{age}}
身高：{{height}}
体重：{{weight}}
家族病史：{{familyHistory}}
腰围：{{waistline}}
收缩压：{{systolicPressure}}
是否处于妊娠期：{{isPregnancy}}
是否患病：{{disease}}
```

添加角色后，点击“添加”按钮，则可以快速添加所有对应的变量信息。点击添加按钮完成变量填写，并修改变量的类型和是否必填选项。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F48395cf95d5c4757aefac231d7323620.png)

图8 批量添加变量

最终效果如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2Fbf7521f6c6b04c5180d6ad4a2771f6cc.png)

图9 变量列表

继续编写技能提示词，在技能中可指定通过哪个工具来完成哪项功能，例如查询文章功能，提示词如下：

```
## 技能
### 技能 1: 获取糖尿病文章信息
1. 当用户询问有关糖尿病的文章信息时，通过'execute_SQL'查询文章信息。
2. 将提取的信息以清晰易懂的方式呈现给用户。
```

其中execute_SQL为工具名，可在下方的工具栏查看并复制名称，注意名称需要用单括号包裹。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F040e0596e4e7429f8118d00451910f69.png)

图10 复制工具名称

其他提示词参考如下：

```
### 技能 2: 生成生活方案
1. 当用户需要生成生活方案时，先询问用户日常的生活习惯和方案建议，然后调用'life_plan'生成生活方案。

### 技能 3: 风险预测
1. 当用户想了解自己的糖尿病风险时，通过'risk_detection'来进行糖尿病风险预测，并返回给用户预测结果，。

### 技能 4: 修改用户信息
1. 当用户提出修改信息的需求时，明确询问用户想要修改的具体信息项，如身高、体重等。
2. 对用户输入的新信息进行合理性检查，确保数据的准确性和逻辑性。
3. 通过'execute_SQL'进行数据更新，并通过更新变量数据。

### 技能 5: 查看用户信息
1. 当用户提出例如查看自己的用户信息、生活方案、生活建议等内容时，调用'execute_SQL'获取数据，并以表格形式展示给用户数据结果。

### 技能 6: 生活状态分析
1. 当用户提出例如分析一下用户最近的生活状态，调用'card_analysis'分析数据，并以表格形式展示给用户数据结果。
```

最后配置限制，参考提示词如下。

```
## 限制:
- 只提供与糖尿病生活管理相关的信息和服务，拒绝回答与糖尿病无关的话题。
- 所输出的内容必须按照清晰、有条理的格式进行组织，不能偏离框架要求。例如，生活方案要有明确的板块划分，生活习惯建议要分点列出等。
```

最终效果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F2383b651abc44bc8b71f2e782fc7a1f5.png)

图11 配置智能体

在右侧进行调试，例如：和智能体说改体重，则会将数据库中的用户信息中的体重由75变为76。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2F181b4cc6d3e749ab8cb3c36738650a5c.png)

图12 智能体测试

再例如，让智能体帮忙生成一份健康方案，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250422%2Fbf2d8969d8f5447cb3fecc8509bc70c0.png)

图13 智能体测试