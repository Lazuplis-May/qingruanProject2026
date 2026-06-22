任务详情

任务2-3：使用Dify开发数据管理工作流

建议工时：

1

任务描述

**1. 任务描述**

本任务通过Dify搭建并发布后台服务接口，实现项目的数据管理。

**2. 任务知识**

**知识点 ：**知识库搭建、工作流搭建、工作流发布、工作流接口测试；

**重点 ：**工作流搭建、工作流发布；

**难点 ：**工作流搭建、工作流发布；

**3. 任务成果**

本任务成果为工作流设计，工作流程如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F18abf85340f54d0a83893fbeb6f3821b.png)

图1 工作流设计



任务指导

#### 1.Dify服务端开发流程讲解

传统的服务端开发一般采用SpringBoot、Express、Fast API等后端框架进行开发，在开发项目时，通过数据库保存数据，并开放对外的API接口形式来供前端访问。一般的开发流程为：定义接口形式，然后再根据参数处理数据，构建数据库操作语句（SQL），然后执行SQL语句，得到结果并返回给前端。传统开发的优势是代码执行效率快，对复杂度较高的业务支持较好，但是通常需要编写大量代码，开发过程较为复杂。开发人员需熟悉多种技术，如编程语言、框架、数据库操作等，对技术能力要求较高。

而Dify融合了后端即服务的特点，其搭建的智能体、工作流等均可作为服务，Dify提供直观可视化界面，降低开发门槛。通过拖拽节点即可编排工作流，如构建 “用户提问→检索知识库→生成回答” 流程，无需编写大量代码，使得非技术人员也能参与 AI 应用定义和数据运营。通过Dify工作流与Text2SQL技术，可以实现仅通过一个工作流即可提供绝大部分数据的增删改查。

随着大模型的发展，Text2SQL应用也应运而生，简单来说，Text2SQL就是将自然语言转换为SQL语言，通过自然语言描述，就能对数据库进行增删改查等各种操作。例如查询文章标题：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F2908202cdd5040c3a45a44d55f368d5a.png)

图1 Text2SQL

通过Text2SQL技术即可实现通过一个API，完成多数据表的复杂增删改查操作。但在使用Text2SQL时，还需要让模型了解数据库的表构造，通过表构造来生成SQL语句，可以通过知识库来实现让模型了解数据库表构造。在本项目中的Dify系统中已经预置了Express+SQLite服务作为数据库服务，接下来只需要构建工作流实现通过自然语言来对数据库进行增删改查处理即可。

在本任务中将通过Dify与Text2SQL技术实现数据库管理工作流的开发。主要流程包括

1）搭建数据库表相关的知识库

2）创建数据库对接工作流和工具

3）创建数据管理工作流

#### 2.知识库搭建

知识库是指面向应用领域问题求解的需要，将知识用某种（或某些）知识表示方法表达、组织、存储在计算机中，便于使用和维护，既相互关联又相对独立的知识片集合。

例如通过Dify构建一个糖尿病相关的知识库，知识库里面的内容均为糖尿病相关的知识，那么在使用大模型时通过引用知识库里面的内容，就可以让大模型拥有相关的知识，如果构建知识量非常多的知识库，由于大模型有上下文的限制，无法处理大量的知识，可以采用知识分段的方式，将知识划分为一个个的片段，在每次引用时仅引用片段即可。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F89f7499e433f47f5bdc8ea7e3d3637ae.png)

图2 知识分段

在用户提问时，通过向量相似度算法计算每个片段与提问问题的相似度。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F38776da1e0c04ea9bb4ad959a87424fd.png)

图3 向量相似度计算

在本任务中，可以构建数据库表结构知识库，当通过自然语言进行操作时，计算出相似度最高的表结构，然后通过大模型生成SQL脚本，最后执行得到结果。

进入Dify平台，点击上方“知识库”，然后点击“创建知识库”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F1f959609a08648468f68a81b02975882.png)

图4 创建知识库

选择“创建一个空知识库”，知识库名称为“数据库表结构”，然后点击“确定”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F1113c7c393124523a960a24909243fe1.png)

图5 创建知识库

在知识库中选择“添加文件”，然后将“db.txt”上传到知识库中（db.txt可在资料中下载），点击下一步。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F07b4c6c0f275441b8d049e5cbe0646f1.png)

图6 创建知识库

分段标识符为“--”，索引类型为“经济”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fd0cc4776ded496c8fd0f7ddef8ca6e5%2Frichtext%2Fimage%2F20260506%2F5c0bd44cd62646a882939732b5bac1bd.png)

图7 设置模型

在上述操作中，“--”是区分每段的标识符，在db.txt中每个表结构前都是由--开头，所以通过此标识符可以划分出较为完整的的知识片段。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fd66afd863ab04c0ead117bdba0eddc5f.png)

图8 查看分隔符

点击最下方的“保存并处理”，跳转至如下图所示页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F3e72cb3d4e474ec48a288d33363892b2.png)

图9 上传文档

点击“前往文档”，即可看到划分的知识片段。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F0aff579019574625b7d42f92ded5cc4a.png)

图10 知识片段

点击“召回测试”，输入“users”，点击“测试”，即可召回最相关的知识片段信息，至此知识库已搭建完毕（注意由于文档索引格式选择为经济，中文召回效果不好，尽量使用英文进行召回）。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fd0cc4776ded496c8fd0f7ddef8ca6e5%2Frichtext%2Fimage%2F20260506%2Fd2363ac5bb8e40268a72aab5f8d0a8a6.png)

图11 召回测试

#### 3.工作流搭建

由于后续任务需要多次通过Express服务与数据库进行交互，所以先创建数据库接口调用工作流，然后再创建数据库操作工作流，在数据库操作工作流中调用接口调用工作流，实现数据库操作。

点击“工作室”，然后点击“创建空白应用”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F76bb929da0044cb5b3d10c4d7e66a6e3.png)

图12 创建空白应用

点击“工作流”，工作流名称填写“database_workflow”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F65764d1389254a3484e7087a77397ba2.png)

图13 创建工作流

工作流包括多种节点，以起始节点开始，以结束节点结束，中间可以进行各种节点操作。

该工作流接收一个SQL代码，调用数据库服务执行代码并返回结果，所以在开始节点中添加名为“sql”的变量，点击开始节点，然后点击“+”添加字段，安装如下图所示添加变量。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Ff3c9f72379b24fbfbd4ba4a8ec72f391.png)

图14 添加变量

在开始节点后添加一个HTTP请求节点，配置如下。

API：POST http://{IP}:3000/execute/

HEADERS :Content-Type application/json

BODY: JSON {"sql": "{sql}"}

其中{IP}为本地内网IP，本地虚拟机可通过命令：ip addr show enp0s3查询，云沙箱环境可在最右侧查看到沙箱IP配置。{sql}为开始节点的SQL，输入方式为先输入“/”字符，然后再选择“开始”节点的sql。最终效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F704277a7829249eabd4d48359b56b299.png)

图15 添加节点

请求的响应数据为字符串，接下来再加一个“代码执行”节点，将字符串转换为数组变量，输入变量为“body”，值为HTTP请求节点的body，代码类型为“JavaScript”，代码为：

```
function main({body}) {
    let obj = JSON.parse(body)
    return {
        result: obj.result
    }
}
```

输出变量为“result”，格式为“Array[object]”，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fb096141934ff43eeaa5c2a0415519693.png)

图16 配置节点

最后添加“结束”节点，输出变量为“result”，值为“代码执行的result”。

点击“运行”，输入“select * from articles limit 4;”此段语句为查询4条文章信息，然后点击“开始运行”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fec9f91bba7b64af49c4deabea7489837.png)

图17 开始运行

稍等片刻，在输出中即可看到JSON格式的文章数据。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F23ccd9cac2b74d1abb67829d611b57f3.png)

图18 测试结果

此工作流开发完成，接下来就将此工作流发布为工具，以供其他工作流或智能体调用，点击上方“发布”，然后先点击“发布更新”，再点击“发布为工具”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F0b98652efea5482182159d07eb1621b1.png)

图19 发布工作流

工具调用名称与名称一致，然后点击保存。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F616edc7401454b04956e2adeaf417b00.png)

图20 配置工具名

接下来创建一个名为“数据管理工作流”的工作流，在开始节点添加“intention”变量,设置字段类型为“段落”，最大长度可设置为60000，显示名为意图，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fff8efa572b6344eaa9a349d83ec8a975.png)

图21 添加变量

为防止输入内容太多，影响后续的知识库搜索，在开始节点后添加一个“代码执行”节点，将其名称修改为“简化意图”，保留前50个字符来进行知识库搜索，配如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F090a08ddcf9d456c949f4b4c8fb48b9e.png)

图22 配置工作流

代码如下：

```
function main({text}) {
    if(text.length>50){
        text = text.substring(0, 50);
    }
    return {
        result: text
    }
}
```

接下来创建一个“知识检索”节点，查询变量为简化意图的result，知识库为“数据库表结构”，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F7d780b084a3a40a186a9f7836c08cad4.png)

图23 配置知识检索

知识检索得到的数据为Object对象，需要进行进一步处理，在知识检索后添加一个代码执行节点，节点名为“处理检索知识”，配置如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F9ae68e84332442d79339b9f60fec45ae.png)

图24 配置代码节点

代码如下图所示：

```
function main({kwList}) {
    let result = ''
    kwList.forEach(item=>{
        result+=item.content
        result+="\n"
    })
    return {
        result: result
    }
}
```

接下来添加一个“LLM”节点，即大模型节点，配置模型为“deepseek-chat”,该模型的特点为执行速度快，但是模型性能低，对于部分时效性高，但是较为简单的功能，可以通过“deepseek-chat”模型来处理。参考提示词如下：

```
你是一个SQL专家，可以根据用户说的自然语言转为SQL语句.输出格式仅限输出SQL语句。输出格式为:<sql>Select*from title</sql>，关键数据库可参考:{result},
完整数据库表创建命令可参考：
-- 创建用户表
CREATE TABLE IF NOT EXISTS users (
 user_id INTEGER PRIMARY KEY AUTOINCREMENT,
 username TEXT NOT NULL UNIQUE,
 password TEXT NOT NULL,
 avatar_url TEXT
);
-- 创建医生资讯表
CREATE TABLE IF NOT EXISTS doctor_information (
 info_id INTEGER PRIMARY KEY AUTOINCREMENT,
 doctor_name TEXT NOT NULL,
 department TEXT,
 title TEXT,
 introduction TEXT,
 image_url TEXT,
chat_token TEXT
);
-- 创建文章科普表
CREATE TABLE IF NOT EXISTS articles (
 article_id INTEGER PRIMARY KEY AUTOINCREMENT,
 title TEXT NOT NULL,
 cover_url TEXT,
 author TEXT NOT NULL,
 publish_time TEXT NOT NULL,
 content TEXT NOT NULL,
 category TEXT,
 views INTEGER DEFAULT 0
);
-- 创建糖尿病种类表
CREATE TABLE IF NOT EXISTS diabetes_types (
 type_id INTEGER PRIMARY KEY AUTOINCREMENT,
 type_name TEXT NOT NULL,
 img TEXT,
 pathogenesis TEXT,
 manifestation TEXT,
 treatment
);
-- 创建文章收藏表
CREATE TABLE IF NOT EXISTS article_collections (
 collection_id INTEGER PRIMARY KEY AUTOINCREMENT,
 user_id INTEGER,
 article_id INTEGER,
 FOREIGN KEY (user_id) REFERENCES users(user_id),
 FOREIGN KEY (article_id) REFERENCES articles(article_id)
);
-- 创建用户风险信息表
CREATE TABLE IF NOT EXISTS user_risk_info (
    userId INTEGER PRIMARY KEY AUTOINCREMENT,
    age INTEGER,
    sex TEXT,
    height REAL,
    weight REAL,
    familyHistory TEXT,
    waistline REAL,
    systolicPressure REAL,
    isPregnancy TEXT,
    message TEXT,
    disease TEXT
);  
-- 创建生活方案表
CREATE TABLE IF NOT EXISTS life_plans (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    type TEXT CHECK (type IN ('饮食', '运动', '其他')) NOT NULL,
    `order` INTEGER NOT NULL,
    time TEXT NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL
);
-- 创建生活建议表
CREATE TABLE life_advice (
	id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
	user_id INTEGER NOT NULL,
	title TEXT(255),
	tags TEXT(255),
	content TEXT(65535)
);
-- 打卡记录表
CREATE TABLE punch_in (
    user_id INTEGER NOT NULL,
    punch_time DATETIME NOT NULL,
    punch_type TEXT NOT NULL,
    completion_status TEXT NOT NULL,
    message TEXT
);
```

点击下方“添加消息”，在新输入框中选择开始节点中的“intention”变量。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/441bd959ad5a491e8b4de77faaf01bb4%2Frichtext%2Fimage%2F20260604%2F70966a174a6b4d7b865fae0a28bb817f.png)

通过该节点实现Text2SQL功能，告诉大模型意图以及数据库结构，让其返回SQL语句，参考如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fd0cc4776ded496c8fd0f7ddef8ca6e5%2Frichtext%2Fimage%2F20260506%2F78fcd1e7e41e406dab250f8dc8f41826.png)

图25 配置LLM节点

由于大模型输出内容不固定，所以为了规范输出，输出格式被规定为<sql>xxx</sql>，接下来添加一个代码节点，通过将sql标签内的代码提取出来，参考如下图所示配置。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fec21f28d66194e07ad39de73d75de7f6.png)

图26 配置代码节点

代码如下：

```
function main({str}) {
    // 使用正则表达式删除 <details> 标签及其内容
    const removeDetailsRegex = /<details[^>]*>[\s\S]*?<\/details>/gi;
    const strWithoutDetails = str.replace(removeDetailsRegex, '');

    // 使用正则表达式提取 <sql> 标签内的内容
    const extractSQLRegex = /<sql>(.*?)<\/sql>/is;
    const match = strWithoutDetails.match(extractSQLRegex);
    let result = match ? match[1] : null
    if(result!=null){
        result =result.replaceAll("\\n","")
    }
    return {
        result:result
    }
}
```

接下来调用刚刚创建的工作流来执行SQL语句，在工具中选择“database_workflow”节点。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fef146c64fd1d4ba69324f5544e43928e.png)

图27 配置工具节点

sql值选择上一节点的“result”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F559a9b605e3d404b8f19ace14de3e098.png)

图28 配置工具节点

最后添加“结束”节点，在此节点中添加“body”变量，变量值为上一节点的“text”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F0908376909f047bf84c288fc70dd5134.png)

图29 结束节点

测试工作流，输入以下内容。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fb662373aea894ea6b72787f365743329.png)

图30 测试工作流

执行结果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F32476b971628489f8899bca0ec975933.png)

图31 查看执行结果

测试完成后，发布工作流。如果发布时提示Rerank模型为空，则在知识检索中编辑知识库，然后先把知识库设置改为“高质量”，保存后重新改为“经济”，然后再次保存。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/441bd959ad5a491e8b4de77faaf01bb4%2Frichtext%2Fimage%2F20260604%2F7521c95ed3e748f5ba39dad589585773.png)

#### 4.工作流调用

工作流搭建成功后，接下来就需要对工作流进行调用，工作流一般是通过HTTP请求来进行调用，在调用工作流时还需要申请对应的凭证来调用，在“访问API”中即可看到相关的调用文档。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F35665f0ac30b4ab69a7903b958a67b36.png)

图32 工作流调用API

点击右上角的“API秘钥”，即可创建秘钥，复制秘钥信息以供后续调用。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F5722afc4b4c44d4d862a69a062fd8033.png)

图33 创建秘钥

打开VSCode，并在项目根目录中创建“WorkflowTest.html”，编写一个测试页面来测试工作流，代码如下：

```
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Workflow 接口测试</title>
    <script src="js/sweetalert2.all.min.js"></script>
    <script src="js/tailwindcss.js"></script>
</head>

<body class="p-8 bg-gray-100">
    <div class="max-w-4xl mx-auto bg-white rounded-lg shadow-md p-6">
        <h1 class="text-2xl font-bold mb-6 text-gray-800">Workflow 接口测试</h1>

        <div class="space-y-4">
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">API 密钥</label>
                <input type="password" id="apiKey"
                    class="w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Intention</label>
                <input type="text" id="intention"
                    class="w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
            </div>

            <button onclick="submitRequest()"
                class="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 transition-colors">
                发送请求
            </button>
        </div>

        <div class="mt-8">
            <h2 class="text-lg font-semibold mb-4 text-gray-800">响应结果</h2>
            <pre id="responseOutput" class="bg-gray-50 p-4 rounded-md border overflow-x-auto"></pre>
        </div>
    </div>

    <script>
        async function submitRequest() {
            const apiKey = document.getElementById('apiKey').value;
            const intentionInput = document.getElementById('intention');
            const output = document.getElementById('responseOutput');
            const submitBtn = document.querySelector('button');

            // 禁用按钮防止重复提交
            submitBtn.disabled = true;
            submitBtn.textContent = '请求中...';

            if (!apiKey || !intentionInput.value) {
                submitBtn.disabled = false;
                submitBtn.textContent = '发送请求';
                Swal.fire('错误', '请填写所有必填字段', 'error');
                return;
            }

            try {
                output.textContent = '';

                const response = await fetch('http://192.168.59.246/v1/workflows/run', {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${apiKey}`,
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        inputs: {
                            intention: intentionInput.value
                        },
                        response_mode: "blocking",
                        user: "abc-123"
                    })
                });

                if (!response.ok) {
                    throw new Error(`HTTP 错误! 状态码: ${response.status}`);
                }
                const result = await response.text();
                const responseData = JSON.parse(result);
                let text=""
                    // 添加是否是string的判断
                    if (typeof responseData.data.outputs.body === 'string') {
                        text= JSON.parse(responseData.data.outputs.body);
                    }else{
                        text =  responseData.data.outputs.body;

                    }

                output.textContent = JSON.stringify(text, null, 2);


            } catch (error) {
                Swal.fire('请求失败', error.message, 'error');
                console.error('Error:', error);
            } finally {
                // 无论成功失败都恢复按钮状态
                const submitBtn = document.querySelector('button');
                submitBtn.disabled = false;
                submitBtn.textContent = '发送请求';
            }
        }
    </script>
</body>

</html>
```

以上代码中的192.168.59.246为服务器的内网IP地址，请根据实际情况替换，如果使用的是沙箱环境则替换为沙箱环境的80端口的映射地址。

通过Living Server运行此HTML页面，填写API和意图，点击发送请求，稍等片刻后查看响应结果。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F80c4d2c55e634e2bb21d922d50a6ea9c.png)

图34 执行结果