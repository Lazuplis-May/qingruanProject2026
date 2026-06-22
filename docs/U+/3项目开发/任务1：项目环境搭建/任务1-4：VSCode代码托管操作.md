任务详情

任务1-4：VSCode代码托管操作

建议工时：

1

任务描述

**1. 任务描述**

本任务讲解在VSCode中的代码拉取，推送等操作。

**2. 任务知识**

**知识点 ：**VSCode中Git图形化界面使用；

**重点：** VSCode中Git图形化界面使用；

**难点：**VSCode中Git图形化界面使用；



任务指导

**1. VSCode代码托管基本操作**

1）推送代码到远程仓库

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F1FE821AC337D45C584E4A421468FD8BA.png)

图1 推送代码到远程仓库

2）从远程仓库中拉取代码

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F9235F9CF203547B9AA15B630B6FAA53C.png)

图2 从远程仓库中拉取代码

任务实现

**1. VSCode代码托管基本操作**

**1）推送代码到远程仓库**

在上一节任务中已经完成本地代码仓库创建以及本地与远程仓库关联，可以省去创建仓库以及关联远程仓库步骤。

首先在项目中创建README.MD文件，并修改README.MD中的内容，例如修改为test01：

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F1564AD98CE184AA0BD76D73682519FAC.png)

图1 修改README.MD文件

点击右侧源代码管理工具，可以看到修改的内容。

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2FDEE9F53424FD4CDAB636840F9886FBC4.png)

图2 源代码管理工具查看修改的内容

点击加号键暂存修改，如下图所示：

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2FE4EC9BD8B3274506ABC22B82AB43C819.png)

图3 提交代码-1

输入提交信息，并点击对号键提交到本地仓库中，如下图所示：

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F20D6FF3E70BB4FBFBDB89AA0E45849B3.png)

图4 提交代码-2

提交完毕后选择推送，如下图所示，点击后等待推送完毕：

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F1FE821AC337D45C584E4A421468FD8BA.png)

图5 提交代码-3

推送完毕后查看远程代码仓库中的README.MD文件是否同步修改，如下图所示，说明推送成功。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b2449468f2744ec5b93394dc95e05db0%2Frichtext%2Fimage%2F20220818%2F191c2a91da534a40bff259a256852cbb.png)

图6 查看README.MD文件

**2）从远程仓库中拉取代码**

在远程仓库中修改README.MD文件，例如将内容修改为test01-pull，如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b2449468f2744ec5b93394dc95e05db0%2Frichtext%2Fimage%2F20220818%2F99c60e8fdf354009b473ee42486f17e7.png)

图7 修改README.MD文件

打开VSCode，点击源代码管理，点击拉取，如下图所示：

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2F9235F9CF203547B9AA15B630B6FAA53C.png)

图8 拉取代码

 

拉取后查看README.MD文件，如下图所示，则说明拉取成功。

![img](https://staticfile.eec-cn.com/4028807c7c2c4a16017c63d065c854cb%2Frichtext%2Fimage%2F20211217%2FCE759B3EAB234ACFB883E5419BCFCD09.png)

图9 查看README.MD文件