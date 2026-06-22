任务详情

任务1-1：Dify服务搭建（沙箱版）

建议工时：

2

任务描述

**1. 任务知识**

知识点：Dify服务搭建与启动、模型配置流程、模型测试

重点：Dify服务搭建与启动

难点：模型配置流程

**2. 任务成果**

Dify服务部署图：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F22e8d8361c274baf942ede45906f591a.png)

图1 Dify工作台

 



任务指导

**1.启动Dify服务**

本项目通过Dify服务结合Express+SQLite数据库服务来作为后端服务，首先打开云沙箱环境，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F9d81e14752174fb68097378bee8af6f1.png)

图1 沙箱桌面

为方便启动服务，在沙箱中已经制作好了服务启动脚本，双击桌面上的“start.sh”,然后选择“Execute in Terminal”，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F4e6b99785e7442628455ff46d3ebb47b.png)

图2 启动Dify

弹出如下图所示弹窗则为启动成功。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fe55756d1842f42b99b61bfe363ec2fa4.png)

图3 启动Dify

沙箱通过公网端口映射方式来与外界进行通信，鼠标移至右侧菜单栏最下方，则可以看到端口映射地址，点击“复制”按钮，则复制映射的IP和端口。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Ff3e10c67c924495ea0cc27e1f3c55735.png)

图4 复制端口映射地址

在本地浏览器中地址栏中粘贴以上复制的地址地址，则进入到Dify服务的登录页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F5891b893127b4e78b92749b6e3605c2d.png)

图5 登录Dify

登录邮箱为“qst@qst.com”，密码为“qst123456”。点击登录，进入到Dify开发页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F22e8d8361c274baf942ede45906f591a.png)

图6 Dify工作台

**2.模型配置**

在进行Dify服务开发前，还需要进行模型配置，在本项目中使用DeepSeek模型。

点击头像，然后选择“设置”，打开设置界面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F0ba2a177b3254572a531cc11b95c0fff.png)

图7 模型配置

在设置界面中选择“模型供应商”，然后点击在“深度求索”中点击“设置”，输入DeepSeek的API Key，然后点击“保存”（此Key具体信息可询问负责实训的教师）。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fde1e152cef5845328c5e0cc36072aff5.png)

图8 配置Deepseek模型

**3.模型测试**

在以上步骤中，所有模型均配置完成，接下来验证模型是否生效，点击“ESC”返回到主界面，点击“创建空白应用”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F0919bc9e5de94bea87736b6f8db84061.png)

图9 创建空白应用

选择应用类型为“聊天助手”，应用名称为“糖尿病专家”，然后点击“创建”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fe36b7f36fb1b447bbf92f9cbb60093d2.png)

图10 创建聊天助手

在编排界面添加以下提示词，右上角的模型选择为“deepseek-reasoner”。

```
一、角色设定
你是一位拥有 15 年临床经验的糖尿病专科医生，持有内分泌学专业医师资格证，精通糖尿病分型诊断、血糖管理、并发症防治等核心领域。你的知识体系覆盖：

糖尿病医学指南（ADA/AACE/CDS 最新标准）
胰岛素 / 口服药作用机制及个体化用药方案
医学营养治疗（MNT）与运动疗法循证依据
糖尿病肾病 / 视网膜病变 / 神经病变等并发症筛查标准
妊娠糖尿病、青少年糖尿病等特殊人群管理规范
二、对话原则
专业严谨：所有建议基于最新临床指南，标注参考文献来源（如《中国 2 型糖尿病防治指南 2020 版》）
循证优先：优先推荐有 RCT 研究支持的干预措施
个体化导向：需主动询问患者基本信息（年龄 / 分型 / 病程 / 并发症 / 用药史 / 过敏史 / 肝肾功能 / 血糖监测数据 / 生活方式等）
风险提示：对胰岛素使用、低血糖处理等关键环节进行明确风险告知
人文关怀：使用通俗易懂语言，避免医学术语堆砌，关注患者心理状态（如焦虑情绪疏导）
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F8e973a6627764f0583660b31962bed4e.png)

图11 配置提示词

在右侧对话栏中进行聊天，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F954b77c0410a42a4aac2d5808e3b4488.png)

图12 聊天测试