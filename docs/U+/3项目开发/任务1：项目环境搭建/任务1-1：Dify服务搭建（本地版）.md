任务详情

任务1-1：Dify服务搭建（本地版）

建议工时：

2

任务描述

**1. 任务知识**

知识点：虚拟机软件安装、虚拟机创建、Dify服务搭建与启动、模型配置与模型测试

重点：虚拟机创建流程、Dify服务搭建与启动

难点：Dify服务搭建与启动

**2. 任务成果**

Dify服务工作台：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F22e8d8361c274baf942ede45906f591a.png)

图1 Dify工作台



任务指导

**1.安装Virtual Box**

VirtualBox 是由 Oracle（甲骨文）公司推出的一款开源虚拟机软件。支持多种操作系统，可在 Windows、Mac OS、Linux 等主机系统上创建并运行包括 Windows、Mac OS、Linux、Solaris、Android 等在内的不同客户操作系统。支持硬件虚拟化技术，能让虚拟机直接访问计算机硬件资源，提升性能。提供多种网络连接模式，如 NAT、桥接模式、内部网络等，满足不同网络需求。支持多种虚拟磁盘格式，可方便地进行虚拟磁盘的创建、挂载、卸载、克隆等操作，还能实现主机与虚拟机之间的数据共享。

在资料中可下载Virtual Box的压缩包，解压安装包，并执行安装，按照提示一步步完成软件安装，安装完成后如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F5eb96563f4904769b8b7c3831dc18783.png)

图1 Virtual Box

**2.通过Dify镜像安装Dify服务虚拟机**

Dify 是一款开源的大语言模型（LLM）应用开发平台。它融合了后端即服务（Backend as Service）和 LLMOps 的理念，旨在帮助开发者与非技术人员快速构建、部署和管理基于大型语言模型的 AI 应用。

Dify一般通过Docker来进行安装，但是Docker安装与配置比较复杂，所以在本项目中通过预置的Dify镜像文件创建Dify服务虚拟机，并通过Dify服务虚拟机来启动Dify服务。

首先下载Dify服务镜像文件，可在资料中下载。

下载完毕后解压该文件，得到名为“dify.ova”的文件，此文件则为Dify服务的镜像文件。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F1f48e4f792344ceb85260ab483f5ef1b.png)

图2 镜像文件

 

在Virtual Box软件中选择“导入”。文件选择刚刚下载的镜像文件。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F1348bd4fcb654ecca0c4c69b55349e10.png)

图3 导入镜像

在“设置”中修改“默认虚拟电脑位置”为自定义位置，需要保证所选位置至少有30GB及以上的存储控件。然后点击“完成”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fc1c036b27f86414da3d761a263fb0c0c.png)

图4 配置虚拟机位置

等待片刻后，导入虚拟电脑完成，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F9aa3af1dbe454e9288c8b9d0c48f4576.png)

图5 虚拟机

双击“已关闭”按钮即可开启虚拟机，在虚拟机中鼠标会不再显示，点击右“ctrl”键可以显示鼠标。

在虚拟机开启后，按下回车键即可进行登录，输入登录用户名为“root”，登录密码为“Qst122333@”即可完成登录。

通过回车键确认登录名和密码,不要使用小键盘进行密码输入，在输入密码时不会显示任何内容，正常输入即可。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fbe2c08a1c46c4e5f90ce13c2876e8876.png)

图6 登录虚拟机

登录成功后，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fab382ac84a40497ab6500cb17bb2046e.png)

图7 虚拟机主页

**3.启动Dify服务**

本项目通过Dify服务结合Express+SQLite数据库服务来作为后端服务，为方便启动服务，在虚拟机中已经制作好了服务启动脚本，在服务器命令行中输入以下命令启动服务。

```
./start.sh
```

如下图所示则为启动成功。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F41a6797808a94894a931ca072e538750.png)

图8 启动Dify服务

此命令除启动Dify服务外，还启动了Express+SQLite服务端，访问端口为3000，可通过HTTP请求执行SQL语句，后续任务中会使用此服务端来存储数据。

虚拟机和本地电脑通过桥接方式进行通信，在访问Dify服务前，首先要获取虚拟机的IP地址，输入以下命令查看虚拟机的IP地址。

```
ip addr show enp0s3
```

如下图所示，“192.168.59.246”则为虚拟机的IP地址。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F093376d6ad1e48019dccbc56ea970825.png)

图9 查看IP

在本地浏览器中输入以下IP地址，则进入到Dify服务的登录页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F72ec4ef28d724000b8764e0af28f9c14.png)

图10 登录Dify

登录邮箱为“qst@qst.com”，密码为“qst123456”。点击登录，进入到Dify开发页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F22e8d8361c274baf942ede45906f591a.png)

图11 Dify主页

**4.模型配置与模型测试**

在进行Dify服务开发前，还需要进行模型配置，在本项目中使用DeepSeek模型。

点击头像，然后选择“设置”，打开设置界面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F0ba2a177b3254572a531cc11b95c0fff.png)

图12 Dify设置

在设置界面中选择“模型供应商”，然后点击在“深度求索”中点击“设置”，输入DeepSeek的API Key，然后点击“保存”（此Key具体信息可询问负责实训的教师）。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fde1e152cef5845328c5e0cc36072aff5.png)

图13 配置API Key

此时所有模型均配置完成，接下来验证模型是否生效，点击“ESC”返回到主界面，点击“创建空白应用”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F0919bc9e5de94bea87736b6f8db84061.png)

图14 创建空白应用

选择应用类型为“聊天助手”，应用名称为“糖尿病专家”，然后点击“创建”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250425%2Fcefce74164f64d4e902001879dd1c878.png)

图15 聊天助手

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

图16 编写提示词

在右侧对话栏中进行聊天，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F954b77c0410a42a4aac2d5808e3b4488.png)

图17 聊天助手对话