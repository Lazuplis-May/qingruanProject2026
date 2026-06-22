任务详情

任务5-2：使用Dify开发个人信息管理及风险预测工作流

建议工时：

1

任务描述

**1. 任务描述**

本任务通过Dify搭建个人信息管理及风险预测工作流，当用户输入个人信息后，如果非必填数据为空，则补全非必填数据，然后判断用户是否患病，如果未患病则进行糖尿病风险预测，预测结果为高风险时则判断是风险来自于哪类糖尿病，最后将数据保存到数据库，并返回风险信息。

**2. 任务知识**

**知识点 ：**Dify工作流搭建；

**重点 ：**Dify工作流搭建；

**难点 ：**Dify工作流搭建；

**3. 任务成果**

本任务成果为Dify工作流搭建：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F42553ceb54674aab88e1129553719b65.png)

图1 工作流



任务指导

#### 1.工作流功能分析

本任务的工作流功能包括个人信息管理与风险预测。个人信息管理即为将用户发送的信息存入用户信息数据库表中，而风险预测针对于未患病的用户，根据用户信息参数判断是否有高风险患病，如果为高风险，则判断是否患的是那种类型的糖尿病，并且根据用户信息给出个性化的风险建议。在判断患的是哪一类糖尿病时，还需要注意糖尿病类型中包括“妊娠型糖尿病”，所以还需要将用户是否怀孕考虑进去。

 

#### 2.工作流搭建

进入Dify，并创建名为“信息管理与风险预测”的工作流，在开始界面中定义用户输入参数。

根据上一任务糖尿病风险标准评判规则，需要输入的参数包括：年龄、性别、身高、体重、家族病史、腰围、收缩压。除此之外，还需要知道用户的id信息，用户是否患病和用户是否处于妊娠期，其中腰围、收缩压和是否处于妊娠期均为选填，在开始节点中添加以下变量。

| 变量名           | 显示名         | 类型        | 是否必填 |
| ---------------- | -------------- | ----------- | -------- |
| userId           | 用户id         | 数字        | 是       |
| age              | 年龄           | 数字        | 是       |
| sex              | 性别           | 文本        | 是       |
| height           | 身高           | 数字        | 是       |
| weight           | 体重           | 数字        | 是       |
| familyHistory    | 家族病史       | 文本（255） | 是       |
| waistline        | 腰围           | 数字        | 否       |
| systolicPressure | 收缩压         | 数字        | 否       |
| isPregnancy      | 是否处于妊娠期 | 文本        | 否       |
| disease          | 患病情况       | 文本        | 是       |

效果如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fe271c15d131c430b9cf88da857f03770.png)

图1 开始参数

接下来处理非必填的数据，是否处于妊娠期不影响风险预测判断，仅影响风险预测是哪类糖尿病，所以可以先不做考虑，腰围与收缩压如果不填写的话会影响风险判断，如果用户未填写，可以通过其他数据与公式来计算出这两项的预测值。

**腰围计算公式**

- **男性基础腰围**：当性别为男性时，基础腰围 baseWaist 是身高 height 的 0.47 倍，即![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fee7e42d7c8444bc9b6b4c49938af05dc.png)。
- **女性基础腰围**：当性别为女性时，基础腰围 baseWaist 是身高 height 的 0.45 倍，即![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F709ab5dff108452da242ea97c5f4b324.png)。
- **BMI 大于 24 时的调整**：当 BMI 大于 24 时，需要对基础腰围进行调整。调整后的腰围计算公式为![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fb9611627753a47d48fb670480512f928.png)。

**收缩压计算公式**

- 男性收缩压

  ：

  - 当 BMI 小于 24 时，收缩压范围 pressureRange 为 115，即 ![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fe530f14a33384ee1a15050e3dbfe84eb.png)。
  - 当 BMI 大于等于 24 且小于 28 时，收缩压范围 pressureRange 为 125，即![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fbf9545284fa04c3680248661e1fdf319.png)。
  - 当 BMI 大于等于 28 时，收缩压范围 pressureRange 为 135，即 ![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F0bf384aba3624c23a7c24ef1c3eed4f9.png)。

- 女性收缩压

  ：

  - 当 BMI 小于 24 时，收缩压范围 pressureRange 为 110，即![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fdbd41aee2d444de9afd49281d874bbf5.png)。
  - 当 BMI 大于等于 24 且小于 28 时，收缩压范围 pressureRange 为 120，即![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F02cc369c6c654fc28bfc02ae85c6d394.png) 。
  - 当 BMI 大于等于 28 时，收缩压范围 pressureRange 为 130，即 ![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F0abb5d69fb2745ea8e92c695a5166122.png)。

 

在开始节点后添加代码节点，节点名修改为“处理腰围和收缩压”，输入性别、身高、体重、腰围和收缩压，计算出新的腰围和收缩压，节点配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F003e397cd09a403c81e2dd4e015a9399.png)

图2 处理腰围和收缩压

节点代码如下：

```
function main(param) {
    let sex=param.sex
    let height=param.height
    let weight=param.weight
    let waistline=param.waistline
    let systolicPressure=param.systolicPressure

    function estimateWaist(sex, height, bmi) {
        let baseWaist;
        if (sex === '男') {
            baseWaist = height * 0.47;
        } else {
            baseWaist = height * 0.45;
        }
        if (bmi > 24) {
            baseWaist = baseWaist * (1 + (bmi - 22) / 10);
        }
        return baseWaist;
    }

    function estimateSystolicPressure(sex, bmi) {
        let pressureRange;
        if (sex === '男') {
            if (bmi < 24) {
                pressureRange = 115;
            } else if (bmi < 28) {
                pressureRange = 125
            } else {
                pressureRange = 135;
            }
        } else {
            if (bmi < 24) {
                pressureRange = 110;
            } else if (bmi < 28) {
                pressureRange = 120;
            } else {
                pressureRange = 130;
            }
        }
        return pressureRange;
    }
    
    const bmi = weight / ((height / 100) * (height / 100));
    if(waistline==null||waistline==0){
        waistline = estimateWaist(sex, height, bmi);
    }
    if(systolicPressure==null||systolicPressure==0){
        systolicPressure = estimateSystolicPressure(sex, bmi);
    }
    return {
        waistline:Number(waistline.toFixed(2)),
        systolicPressure:Number(systolicPressure.toFixed(2))
    }
}
```

接着判断是否得了糖尿病，在代码节点后添加一个条件分支节点，判断“disease”值是否为“否”，如果为“否”则未得糖尿病，如果为“是”则得糖尿病。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fe5c8da8a1913474988553ebef9eec1b5.png)

图3 条件分支

未得糖尿病，即if分支节点中，需要进行糖尿病风险判断，在此分支下添加一个LLM节点，节点名为“判断糖尿病风险”，模型可以选择“DeepSeek-reasoner”，在System提示词中输入以下内容进行风险判断。

```
你是一个糖尿病专家，可以通过以下条件判断用户患糖尿病的几率：
评估指标范围分值
年龄（岁）
20～24
0
25～34
4
35～39
8
40～44
11
45～49
12
50～54
13
55～59
15
60～64
16
65～74
18
BMI（kg/m²）
＜22.0
0
22.0～23.9
1
24.0～29.9
3
≥30.0
5
腰围（cm）
男性＜75.0，女性＜70.0
0
男性 75.0～79.9，女性 70.0～74.9
3
男性 80.0～84.9，女性 75.0～79.9
5
男性 85.0～89.9，女性 80.0～84.9
7
男性 90.0～94.9，女性 85.0～89.9
8
男性≥95.0，女性≥90.0
10
收缩压（mmHg）
＜110
0
110～119
1
120～129
3
130～139
6
140～149
7
150～159
8
≥160
10
糖尿病家族史
无
0
有（父母、同胞、子女）
6
性别
女性
0
男性
2
总分≥25 分：提示糖尿病高危风险，需进一步检测空腹血糖、餐后 2 小时血糖或糖化血红蛋白（HbA1c）以明确诊断。
总分＜25 分：风险较低，但建议定期体检并保持健康生活方式。
输出格式为：
[【低风险/高风险】“建议内容”]
```

点击下方添加消息，添加User提示词，并在User提示词中添加以下内容。注意{{xxx}}表示上一节点或开始节点中xxx的值，可先输入"{"字符，然后进行选择。

```
年龄:{{age}}
性别:{{sex}}
身高:{{height}}
体重:{{weight}}
家族病史:{{familyHistory}}
腰围：{{waistline}}
收缩压:{{systolicPressure}}
```

节点最终效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fba840faf9a0a4c9097bdd7613c258ae8.png)

图4 LLM节点

在上述中System（系统）提示词和User（用户）提示词都是提示词的一种，系统提示词是用于定义大模型的身份与技能，例如：“你是一个xxx专家”，而用户提示词则为用户对大模型说的话，例如：“xx专家，我想咨询一些关于xxx的问题”。

填写完毕后，点击运行按钮运行该节点进行测试。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F66ea3affd5674d7b95978815afa5c323.png)

图5 运行测试

测试结果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F8c0782f9b91d4cb1bb961f1dd1b7030d.png)

图6 测试结果

可以看到输出的text中还包含了思考过程，接下来就需要提取文本里面的具体内容，并提取出是否为高风险，在此节点后添加代码节点，名为“提取内容”。删去思考的内容，仅保留回答内容，节点如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fdc2b4023d02a4a1c9f5a9743e1798906.png)

图7 提取结果

代码如下。

```
function main({text}) {
    let result = ""
    const startIndex = text.indexOf('【')||text.indexOf('[');
    if (startIndex!== -1) {
        result = text.slice(startIndex);
    }
    return {
        result:result
    }
}
```

接着添加条件分支节点，判断提取后的内容是否以“【高风险】”，代码如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F9d3f2c29c9c04af99e73629ae39e9a0b.png)

图8 判断是否为高风险

如果是高风险（if分支），则在后续节点中新增一个LLM节点，名为“判断是哪类糖尿病”，节点配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F360c2575c5f041a08070bec7b0722e91.png)

图9 判断糖尿病类型

系统提示词如下：

```
       您是一个糖尿病专家，可以根据用户的身体各项数据判断用户最有得哪类糖尿病的风险。
        输出时，请用清晰简洁的语言描述评估结果，并避免使用任何专业术语，确保用户能够轻松理解。
例如：
用户输入：年龄：35岁，性别：女，身高：160cm，体重：70kg，家族糖尿病史：父亲患有三型糖尿病，腰围：110cm ，收缩压：120mmHg。
输出：【2 型糖尿病】“建议您饮食调整、运动、口服降糖药，必要时注射胰岛素。”
```

以上节点回答的问题同样要删除思考过程，将“提取内容”代码节点复制一份，并粘贴到此节点后，输入变量的值也要同步修改，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F8f791ba3886844e199e4d5e6d1011da9.png)

图10 提取信息

最后将风险变量合并，在提取内容节点后新增一个代码节点，名为“提取风险”，并将判断是否为高风险节点的ELSE也连接进来，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F8b6bb2850cc5404eb4415092b06e1ad4.png)

图11 提取风险

提取风险节点接收两个“提取内容”节点的参数，并判断哪类糖尿病后的参数是否为空，如果不为空则返回此参数，为空则返回另一个，节点配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F0b51ca7908e146a78831b336e73a6bbf.png)

图12 提取风险

代码如下：

```
function main({ arg1, arg2 }) {
    let result = ""
    if (arg1 == null || arg1 == "") result = arg2
    else
    result = arg1
    if(result == null)
    result = ""
    return {
        ret: result
    }
}
```

接下来在此节点后添加一个database_workflow节点，编写SQL语句实现数据保存。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F67bb459def184512b25bca03813ffe0f.png)

图13 保存用户信息

代码如下（请手动修改{{xxx}}的参数值）：

```
 INSERT OR REPLACE INTO user_risk_info (userId, age, sex, height,weight, familyHistory, waistline, systolicPressure, isPregnancy, message,disease)VALUES({{userId}},{{age}},'{{sex}}',{{height}},{{weight}},'{{familyHistory}}','{{waistline}}','{{systolicPressure}}','{{isPregnancy}}','{{ret}}','{{disease}}')
```

也将判断是否得了糖尿病的ELSE节点与此节点相连。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F3bc28ef46e184fc0927e9ca8385226a3.png)

图14 整体流程

在工作流输出时，输出内容包括“是否患病”和“整合后的风险”参数，将这两个参数整合为对象，最后输出到结束节点中。

在“DATABASE_WORKFLOW”工作流后添加代码节点，代码节点配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Ffd7d63aef29f46c78446ba549b60a2e8.png)

图15 合并变量

最后将此节点后添加结束节点，将此节点的输出内容作为结束节点的“body”变量输出。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F9afdf498c99a45298574c67c5ec01600.png)

图16 结束节点

工作流整体架构如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F42553ceb54674aab88e1129553719b65.png)

图17 工作流流程设计

最后进行运行测试。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F8f0f5e31470644fea1b6d60e47b10425.png)

图18 运行工作流

结果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F4ca013e010d14c96a7a4d74619fc69d7.png)

图19 执行结果

接下来测试是否已将健康风险信息同步至数据库，调用数据管理工作流，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fd6153a693391434681161738a2de541c.png)

图20 查询用户风险信息

可以看到用户风险信息已经同步到数据库中。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F84c65f690e5e46ac81fad0ec7abb35dc.png)

图21 用户风险