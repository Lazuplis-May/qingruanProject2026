任务详情

任务3-3：使用Dify搭建医师聊天助手

建议工时：

1

任务描述

**1. 任务描述**

本任务通过Dify搭建智能聊天助手。

**2. 任务知识**

**知识点 ：**Dify聊天助手搭建、提示词编写、知识库搭建；

**重点 ：**提示词编写、知识库搭建；

**难点 ：**提示词编写、知识库搭建；

**3. 任务成果**

本任务成果为Dify聊天助手搭建：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fc85652abce424357a1411fe3080d94ad.png)

图1 聊天助手

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F06589633433e4abdba134b937138152d.png)

图2 聊天测试



任务指导

#### 1.Dify聊天助手

在Dify平台构建聊天助手，采用一问一答的交互模式，能与用户展开多轮持续对话，功能与大模型类似。并且，基于大模型，还可灵活配置知识库、提示词、变量等，实现更强大且个性化的应用。 以配置医生聊天助手为例，该助手一方面要依托通用大模型，掌握通用知识信息；另一方面，还需接入医疗领域的专业知识库，获取专业医学资讯，以此构建扎实的知识储备体系。在与用户交流过程中，助手能够通过设置变量，精准收集用户个人信息，如既往病史、过敏史等。同时，借助精心编写的提示词，规范医生助手的言行风格，使其完全契合专业医生的口吻与行事逻辑，严格按照标准流程，逐步深入剖析并解决用户提出的问题。 经过上述一系列配置，Dify即可打造出高度专业化、定制化的行业聊天助手，精准服务于特定领域的用户需求。 

#### 2.糖尿病知识库搭建

在打造Dify聊天助手前，先需要创建糖尿病的专业知识库，进入Dify，并创建一个空知识库，知识库名称为糖尿病专业知识，并将糖尿病相关知识文档上传至知识库中（该文档可在资料中下载）。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fc860a6efd8094cc4887ccf1a1aaf45a7.png)

图1 导入知识库

分段设置默认即可，索引设置选择“经济”，然后点击“保存并处理”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fd0cc4776ded496c8fd0f7ddef8ca6e5%2Frichtext%2Fimage%2F20260506%2F2ac279cdec7643d8b079326c12fc2024.png)

图2 保存并处理知识

等待所有文档处理完成，点击“前往文档”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F270d02ecfc7347959fd7f037d46b50f6.png)

图3 上传文档

点击对应文档，即可查看分段信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fc8e693ad95144249862f4126ec4afbab.png)

图4 查看分段信息

在召回测试中模拟用户询问问题进行测试，例如Omega，可以看到召回了两个相关的知识片段（需注意经济模式下的召回对中文分词效果不好，尽量使用英文进行分词）。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fd0cc4776ded496c8fd0f7ddef8ca6e5%2Frichtext%2Fimage%2F20260506%2F0020f7a5c0084f91bba82b47487b43be.png)

图5 召回测试

至此，糖尿病专业知识的知识库配置完成。

#### 3.聊天助手创建

在Dify中选择“工作台”，然后点击“创建空白应用”，然后选择“聊天助手”，应用名称就填写医生的对应名，例如“赵晓峰医生”。然后点击“创建”按钮创建聊天助手。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F4e2ad83104774e69b21b9bdcce5faf14.png)

图6 创建聊天助手

创建完成后，进入到聊天助手编排界面，编排界面左侧可以进行提示词，知识库，变量的配置，右侧可以进行模型配置与测试。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fb198fa7d06b84e0999d543b2dcc8a5c6.png)

图7 聊天助手编排页面

首先进行知识库配置，在知识库模块中点击“添加”，选择刚刚创建的“糖尿病专业知识”的知识库。如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F7aebfa3ca6c546c0b0a9166fcbeb613b.png)

图8 配置知识库

接下来进行提示词和变量配置，首先讲解提示词与变量的配置规则：

提示词有多种模板，通用模板格式为角色、技能、限制，例如：

```
#角色
你是一位糖尿病方面的专家
#技能
技能1：xxx
技能2:xxx
#限制
- 只提供与糖尿病预防和治疗相关的医疗建议和指导，拒绝回答与该领域无关的话题。
```

注意，以上提示词中的#仅用于分标题的作用，并无其他特殊含义，在编写提示词时只要让人看懂即可，并无特殊要求。

变量是可以在用户使用聊天助手前初始化的数据，一般包含用户的个人数据，例如身高、体重、用户名等。例如添加“身高”变量，点击“变量”模块中的“添加”按钮，然后选择类型为“数字”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F579155cf57414e249f2fb4a8b882ebfe.png)

图9 添加变量

点击添加后，可以看到变量栏中有一个空白变量，点击编辑按钮修改变量。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F6ab98bddb4744c6a81e66a9d3432c8dd.png)

图10 修改变量

按照以下配置设置变量名，并点击保存。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F702ed865b63b46ceb09b4d7d2e0127d0.png)

图11 修改变量

在编写提示词时，通过{{变量名}}的方式即可引用变量，例如引用用户的身高信息:“用户的身高是：{{height}}”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F8ccaa356ac7e4e6c84acc355ec5ec16a.png)

图12 变量引用

接下来继续进行变量与提示词的配置：用户的信息包括用户id、性别、年龄、身高、体重、家庭病史、腰围、收缩压、是否患糖尿病、是否处于妊娠期等。医生的技能包括糖尿病咨询和提供糖尿病治疗建议。所以提示词如下。

```
# 角色
你是内科主任赵晓峰，在糖尿病的预防与治疗领域有着深厚的专业造诣和丰富的临床经验，能够为患者提供专业、精准且贴心的医疗建议和指导。
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

## 技能
### 技能 1: 糖尿病预防咨询
1. 当用户咨询糖尿病预防相关问题时，详细询问用户的生活习惯，如饮食结构、运动频率、作息规律等，以及家族病史、身体基础状况等信息。
2. 根据用户提供的信息，结合专业知识，为用户制定个性化的糖尿病预防方案，包括饮食调整建议、运动计划、生活方式改善等方面。
3. 以通俗易懂的语言向用户解释预防措施的原理和重要性，确保用户理解并能够执行。

### 技能 2: 糖尿病治疗建议
1. 当用户咨询糖尿病治疗问题时，了解患者目前的病情状况，如血糖水平、患病时长、是否有并发症、正在使用的治疗方法等详细信息。
2. 依据专业医疗知识和临床经验，为患者提供针对性的治疗建议，包括药物治疗、饮食控制、运动疗法、血糖监测等方面的具体指导。
3. 告知患者治疗过程中的注意事项，以及可能出现的问题和应对方法，帮助患者更好地管理病情。

## 限制:
- 只提供与糖尿病预防和治疗相关的医疗建议和指导，拒绝回答与该领域无关的话题。
- 所输出的内容必须条理清晰、逻辑连贯，以患者易于理解的方式呈现。
- 输出时不要输出用户的用户信息，输出内容尽量简洁。
- 医疗建议和指导应基于医学专业知识和临床实践经验，确保科学性和可靠性。
- 信息来源主要依靠医学权威文献、临床研究成果以及自身丰富的临床经验。
```

接着添加变量，由于提示词中引用了未定义的变量，则当添加变量时，会提示是否自动添加变量，点击添加即可完成添加。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F3df489b856954bde8ae03c733683d4c0.png)

图13 批量添加变量

接下来修改一下变量的类型，userId、age、height、weight、waistline（腰围）、systolicPressure（收缩压）设置为数字类型，并且将waistline和systolicressure设置为选填，即可完成变量配置。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fa913e5c9b0954131a41ccfd476818ec0.png)

图14 配置变量信息

接下来进行模型配置，模型配置在右上角，点击即可选择模型，并且配置模型的参数。鼠标移动到参数中的“？”区域，即可查看参数的具体含义。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F09833f0e93fc489c9a6c8e61fa95b25a.png)

图15 选择模型

医生回复一般是比较严谨的，所以可以对温度参数进行微调，将文档参数打开，并且值设置为0.7，使回答不那么发散。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F7060912e961b4128a36a305676beacf3.png)

图16 修改模型参数

接下来进行调试，在调试前先配置好用户变量信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fdf0be9943f6646309f41867e26cec6af.png)

图17 配置变量

点击右上角可关闭填写弹窗，然后发送消息给聊天助手，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F06589633433e4abdba134b937138152d.png)

图18 聊天测试

按照以上步骤，可以创建另外两个医生的聊天助手，并根据情况对提示词、模型配置进行调整。

#### 4.聊天助手测试

点击“访问API”即可查看到聊天助手API接口，接口通过API秘钥方式进行鉴权。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fd43dff59e7e5488cad2036894b27f5f7.png)

图19 访问API

先创建API秘钥，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F5422b9abec2544b89a92fe651a41b9bc.png)

图20 配置秘钥

在后续任务中，需要通过此秘钥来和医师对话，首先打开数据管理工作流修改秘钥配置，参考提示词如下（Bearer 为固定格式，后续则为对应的API秘钥，请替换为自己生成的秘钥）：

```
把info_id为1的医生的chat_token值设置为“Bearer app-XmWNqY4BG0AzqUDRnlwepJbT”
```

如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/53e35ed29ed74bb09864339d5c24ea11%2Frichtext%2Fimage%2F20250627%2Ff6221ececf874ab8a6cb288be48ba6e4.png)

图21 修改医生聊天token

同样的将另外两个医生的聊天token替换为对应聊天助手的token。

接下来通过代码来测试聊天功能，在项目中创建chatTest.html，并添加以下代码。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat API Test</title>
    <link rel="stylesheet" href="css/all.min.css">
    <link rel="stylesheet" href="css/index.css">
    <style>
        .chat-container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
        }
        input, textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        #response {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 4px;
            min-height: 100px;
            background-color: #f9f9f9;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <h1>Chat API Test</h1>
        <div class="form-group">
            <label for="apiKey">API Key:</label>
            <input type="text" id="apiKey" placeholder="Enter your API key" value="app-XmWNqY4BG0AzqUDRnlwepJbT">
        </div>
        <div class="form-group">
            <label for="apiUrl">API Server URL:</label>
            <input type="text" id="apiUrl" placeholder="Enter API server URL" value="http://192.168.59.246">
        </div>
        <div class="form-group">
            <label for="query">Query:</label>
            <textarea id="query" rows="3" placeholder="Enter your question" >我想咨询一下2型糖尿病相关的问题</textarea>
        </div>
        <div class="form-group">
            <label for="parameters">Parameters (JSON):</label>
            <textarea id="parameters" rows="3" placeholder='{"key": "value"}' >{
                "userId": "123",
                "sex": "male",
                "age": 30,
                "height": 175,
                "weight": 70,
                "familyHistory": "none",
                "waistline": 80,
                "systolicPressure": 120,
                "isPregnancy": "no",
                "disease": "no"
            }</textarea>
        </div>
        <button onclick="sendChatMessage()">Send Message</button>
        <div id="response"></div>
    </div>

    <script>
        async function sendChatMessage() {
            const apiKey = document.getElementById('apiKey').value;
            const query = document.getElementById('query').value;
            const parametersInput = document.getElementById('parameters').value;
            const responseDiv = document.getElementById('response');
            
            if (!apiKey || !query) {
                alert('Please enter both API key and query');
                return;
            }

            let inputs = {};
            if (parametersInput) {
                try {
                    inputs = JSON.parse(parametersInput);
                } catch (e) {
                    alert('Invalid JSON parameters');
                    return;
                }
            }

            responseDiv.innerHTML = 'Loading...';

            try {
                const apiUrl = document.getElementById('apiUrl').value;
                const response = await fetch(`${apiUrl}/v1/chat-messages`, {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${apiKey}`,
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        inputs: inputs,
                        query: query,
                        response_mode: "streaming",
                        conversation_id: "",
                        user: "abc-123",
                        files: [{
                            type: "image",
                            transfer_method: "remote_url",
                            url: "https://cloud.dify.ai/logo/logo-site.png"
                        }]
                    })
                });

                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                responseDiv.innerHTML = '';

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    
                    const chunk = decoder.decode(value, { stream: true });
                    let chunk1=""
                    //以data:作为分割，分割出多个字符串
                    const chunks = chunk.split('data: ');
                    chunks.forEach((ck) => {
                           //判断是否可以转为JSON对象
                        try {
                            const data = JSON.parse(ck);
                            responseDiv.innerHTML += data.answer;

                        }catch (error) {
                            console.log("error:",ck)
                        }
                    })
                }
            } catch (error) {
                responseDiv.innerHTML = `Error: ${error.message}`;
                console.error('Error:', error);
            }
        }
    </script>
</body>
</html>
```

修改对应的IP地址、Key，点击“Send Message”按钮发送消息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F4e20771beb3347a79ab891566ece7e5e.png)

图22 聊天测试

可以看到，输出内容是通过流式输出。通过持续接收响应中的响应流来输出结果。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Ff2c8fea073594e9b8078bd7eb88272ca.png)

图23 聊天测试

响应流包括以下两类：

1：ping响应，用于维持请求连接，内容如下。

```
event: ping
```

2：message数据，其中answer数据为输出内容，将所有内容进行拼接后即可得到完整的答案；conversation_id为聊天的id，如果请求不加此id则新建一个新的聊天，如果填写id，则问答基于上一轮对话基础上继续。示例数据如下所示。

```
{
    "event": "message",
    "conversation_id": "72d2ec79-0d5c-449d-89f5-bcb113e5cf99",
    "message_id": "c68636cd-ddb1-459f-bb71-c4162af606d3",
    "created_at": 1744780711,
    "task_id": "5072a20f-250e-44bc-8ddc-02b9e558be0f",
    "id": "c68636cd-ddb1-459f-bb71-c4162af606d3",
    "answer": "<details style=\"color:gray;background-color: #f8f8f8;padding: 8px;border-radius: 4px;\" open> <summary> Thinking... </summary>好的",
    "from_variable_selector": null
}
```

当询问问题后，能正常回答问题，则测试成功。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F3f3a2d5c74264853b583bb8fb3751465.png)

图24 聊天结果