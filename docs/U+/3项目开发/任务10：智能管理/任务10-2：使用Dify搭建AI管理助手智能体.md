任务详情

任务10-2：使用Dify搭建AI管理助手智能体

建议工时：

1

任务描述

**1. 任务描述**

在上一任务中完成了智能管理页面的界面设计，在本任务中将搭建一个智能体，通过该智能体来与管理平台进行数据对接，处理用户发送过来的指令信息，执行指令信息并返回结果。

**2. 任务知识**

**知识点 ：**Dify聊天助手搭建、提示词编写、工具调用、Function Calling；

**重点 ：**提示词编写、工具调用；

**难点 ：**提示词编写、工具调用；

**3. 任务成果**

本任务成果为AI管理助手智能体搭建：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250427%2F9d1e19f309f749f3a1078c671d4484b6.png)

图1 智能体搭建



任务指导

#### 1.AI管理助手智能体搭建

在Dify中创建AI智能管理助手智能体。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F681c03b561fd4e81b49b570bbcb4df07.png)

图1 创建智能体

该智能体需要拥有数据库操作工具，在工具栏中添加“database_workflow”工具。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F772afccdbe244284a53fbd3fd77d611b.png)

图2 添加工具

作为一个网站管理智能体，还需要了解网站中的数据结构信息，在知识库中添加“数据库表结构”知识库。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Ff1f0189d426c4bdfab53410855e67310.png)

图3 设置知识库

接下来就开始编写智能体的提示词信息，该智能体需要实现增删改查能力，其中增删改只显示操作状态和结果即可，例如：

输入：“删除所有没有填写用户信息的用户”。

输出：状态：执行成功；操作结果：删除了5条数据。

而查询在增删改的基础上还需要添加查询数据信息。同时需要注意智能体输出格式需要为JSON格式，方便前端页面进行解析。

可以按照：角色、技能、限制的模板进行提示词编写，示例提示词如下：

```
# 角色
你是一个专业的AI数据助理，能够准确理解用户需求，调用数据库工作流对数据进行增删改查操作，并清晰说明操作结果。

## 技能
### 技能 1: 处理数据查询请求
当用户提出查询需求时，查询数据库表结构中的数据库表，并生成对应的SQL语句。调用'database_workflow'流执行查询操作，查询结果返回为JSON类型，不要返回除JSON类型的其他类型，将查询的JSON列表内对象中的属性名改为中文，例如“title”改为“标题”。
===回复示例===
​```json
{ 
message:"已查询到xx条数据",
status:"查询成功",
data:[xxx]
}
​```
===示例结束===
### 技能 2: 处理数据增删改请求
当用户提出增删改数据时，查询数据库表结构中的数据库表，并生成对应的SQL语句。调用'database_workflow'流执行增删改操作，并返回操作结果，结果返回为JSON类型，不要返回除JSON类型的其他类型。
===回复示例===
​```json
{ 
    message:"已删除xxx数据",
    status:"成功"
}
​```
===示例结束===

## 限制:
- 只处理与数据查询和修改相关的操作，拒绝回答无关话题。
- 所输出的格式必须为JSON格式，禁止输出任何其他格式内容。
```

#### 2.AI管理助手智能体功能测试

完成以上功能后，即可进行功能测试，以简单查询为例：“查询所有用户数量”。结果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Fda2f8c65aaa34e8eaff063625e51c467.png)

图4 测试智能体

以复杂查询为例：“查询打卡次数最多的用户名”

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Fd9d5c062efa049dd95df16c0bc9c4af3.png)

图5 测试智能体

以修改功能为例：“将所有用户的用户名后添加@1”

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F4814a91200e04e8e9ea3737f37697d13.png)

图6 测试智能体