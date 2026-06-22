任务详情

任务1-3：CodeArts代码仓库创建

建议工时：

1

任务描述

**1. 任务描述**

Git（读音为/gɪt/）是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。

**2. 任务知识**

**知识点 ：**代码托管及使用流程、本地仓库与云端仓库关联操作

**重点：**本地仓库与云端仓库关联操作

**难点：**代码托管及使用流程。

**3. 任务成果**

本任务实现在CodeArts中创建代码仓库，通过本地Git工具完成本地仓库与云端仓库的关联操作。



任务指导

**1. 参考技术文献**

代码托管介绍：[https://support.huaweicloud.com/productdesc-codeartsrepo/codeartsrepo_01_0002.html ](https://support.huaweicloud.com/productdesc-codeartsrepo/codeartsrepo_01_0002.html)

代码托管使用流程： [https://support.huaweicloud.com/usermanual-codeartsrepo/codeartsrepo_03_0001.html ](https://support.huaweicloud.com/usermanual-codeartsrepo/codeartsrepo_03_0001.html)

**2. 任务实现步骤**

1）创建远程代码仓库。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F9e5d5420a20e47b78943d300932497b1.png)

图1 创建仓库

​    2）安装Git。

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63efdbe95561%2Frichtext%2Fimage%2F20211208%2F3D917859E0F3472DA6FECC5BFDA0CCB1.png)

图2 安装Git

3）创建本地代码仓库。

```
git init
```

4）同步远程代码仓库。

```
git  remote add origin 远程仓库地址
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Ff06041ed46924f3290a9b590a54cca5b.png)

图3 git代码仓库

 

任务实现

### 一、代码托管概述

####    1. 代码托管（CodeHub）

**CodeHub**是面向软件开发者的基于Git的在线代码托管服务，是具备安全管控、成员/权限管理、分支保护/合并、在线编辑、统计服务等功能的云端代码仓库，旨在解决软件开发者在跨地域协同、多分支并发、代码版本管理、安全性等方面的问题。

![img](https://staticfile.eec-cn.com/40288042759811c10175bbb78b0937a1%2Frichtext%2Fimage%2F20210126%2FEFCA8E4381C543FC96911029F0334AD6.png)图1 Git优点

####    2. 代码托管具有以下几个特性：

1）在线代码阅读、修改、提交，随时随地开发，不受地域限制；

2）在线分支管理，包含分支新建、切换、合并，实现多分支并行开发，效率高；

3）分支保护，可防止分支被其他人提交或误删。

####    3. 代码托管服务系统架构，如下图所示；

![img](https://staticfile.eec-cn.com/40288042759811c10175bbb78b0937a1%2Frichtext%2Fimage%2F20210126%2F76468382979C45509C67C725D8FC8B7A.png)图2 系统架构

### 二、代码托管使用流程

通过代码托管服务，将“共享云盘”的代码保存至云端，方便项目成员之间的多人协作开发。完成项目规划后，由项目经理创建代码仓库，开发人员进行代码的编写。代码托管支持三种新建代码仓库方式：普通新建、按模板新建、导入外部仓库。

在本地开发一个maven项目，并使用代码托管服务来管理版本；将本地仓库绑定云端仓库并完成初始化推送，然后使用分布式版本管理方式来继续开发项目，此处使用“普通新建”方式创建仓库，使用流程如下图所示：

![img](https://staticfile.eec-cn.com/40288042759811c10175bbb78b0937a1%2Frichtext%2Fimage%2F20210126%2FA754E6C1C5B24238B9D8F98EA4975F66.png)图3 代码托管流程图

### **三、创建代码托管仓库**

1）通过教学平台进入CodeArts主界面，选择菜单中的“代码 > 代码托管”，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F9c7b41175d9049c0a80995cba928ad10.png)

图4 代码托管

 

2）创建代码仓库，然后在新页面中选择“普通仓库”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F609bf9ae4c2a4100b3726e23d32e119c.png)

图5 新建仓库

3）输入代码仓库名称如 diabetesAssistant，在允许生成README文件中取消勾选，然后点击确定，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fa2cf04f6d9a84c6cbaf5961301c9f7ce.png)

图6 新建仓库

代码仓库创建完成效果如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F7fe75861bb2146c89202f200ac8edc04.png)

图7 创建仓库

 

### 四、项目关联远程仓库

如果在本地没有安装Git，请先安装，安装包在资料中。

在CodeArts代码仓库界面中，可以看到“关联已有代码目录到仓库”，复制第三行到第七行。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fd71b97bcfed14a02ad1d07d4fefd311c.png)

图8 复制关联命令

 

然后在本地打开上一任务创建的项目。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2Fabe69d887f734ca39f232066b5dc2a93.png)

图9 打开项目

在空白处点击右键，然后选择“Git Bash Here”打开命令行窗口，然后将刚刚的命令复制到命令行窗口中，并按回车执行。

执行结果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250319%2Fafffd9ce7ca546d38478e2fb57b2dbb0.png)

图10 打开git终端

在执行时，会弹出认证弹窗。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/6c87bc29b507463cab80bfb09d16d69d%2Frichtext%2Fimage%2F20241115%2Fdef3e8deeb8449eba0caa8a5b086316c.png)

图11 配置用户名和密码

回到CodeArts网页，点击右上角头像，然后选择“个人设置”>“代码托管”>“Https密码”，在此页面中可以查看和设置认证账号和密码。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b27954db9e3b4131ab8205dead193546%2Frichtext%2Fimage%2F20240726%2F3fcb90fde7234bf89aa5b663884e3483.png)

图12 配置密码

可自行设置密码，然后将账号密码输入认证弹窗中。

提交成功后，在CodeArts 代码仓库中即可查看到提交的代码信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250414%2F79a58e04b0e44bf9a464b09776ee2ec0.png)

图13 代码仓库信息