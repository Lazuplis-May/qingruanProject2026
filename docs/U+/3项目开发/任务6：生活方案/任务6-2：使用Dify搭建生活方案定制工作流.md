任务详情

任务6-2：使用Dify搭建生活方案定制工作流

建议工时：

1

任务描述

**任务描述**

本任务通过Dify搭建生活方案定制工作流工作流，当获取用户个人信息，用户生活状态及用户的生活方案建议后，通过工作流生成多条生活方案，包含饮食方案和运动方案，并保存到数据库中。

**2. 任务知识**

**知识点 ：**Dify工作流搭建；

**重点 ：**Dify工作流搭建；

**难点 ：**Dify工作流搭建。

**3. 任务成果**

本任务成果为Dify工作流搭建：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250427%2F86d696dac38a4c16ab4d623b9b7ee6c1.png)

图1 工作流流程配置



任务指导

#### 1.工作流功能分析

在定制生活方案时需要考虑三方面的内容：

- 用户当前身体信息
- 用户当前的生活习惯
- 用户对方案的生活建议

首先要针对用户当前的身体状态制定饮食和运动方案，并且制定的方案尽量不与用户生活方案发生重瞳，并且要考虑用户的生活建议，例如用户说对什么食物过敏，就在制定方案时不使用与该食物有关的饮食放哪。制定完方案后，删除用户的原有方案，并且插入新的方案，最后返回方案内容。

#### 2.生活方案制定工作流开发

在Dify中创建名为“方案定制”的工作流。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fc51fd6813316473c90686d058c74ddfe.png)

图1 创建工作流

在开始节点中定义参数，包括用户id，用户信息，习惯与建议，需要注意用户信息、习惯与建议内容可能会比较长，需要使用段落类型，并将长度设置的足够长。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F6253d188b22e404199c5323ff9fb0d6e.png)

图2 开始节点配置

在开始节点后添加LLM节点，节点名为“饮食方案”，在此节点中实现饮食方案的生成。系统提示词参考如下：

```
你是一个糖尿病生活指导师，可以根据用户信息，用户当前的生活习惯给出饮食方面的生活方案，包括早餐、午餐、晚餐、加餐，输出四条方案。需要注意用户必须的生活习惯，尽量不与用户建议产生冲突。
输出内容为列表形式包括执行时间，标题，内容，输出格式严格遵守JSON格式，除JSON数据外不要输出任何其他内容，例如：
JSON:
[
{"time":"7:30-8:00","title":"早餐建议":"content":"一杯牛奶、两片全麦面包和一个苹果。"}
]
```

用户提示词如下(注意将{{xxx}}内的变量修改为开始节点对应的变量)：

```
用户信息：{{userInfo}}
用户生活习惯：{{habit}}
生活习惯方案建议{{suggestion}}
```

效果如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fd7ae242850d44d0c8cab7ff3f84f14fb.png)

图3 LLM节点配置

运动方案与饮食方案格式大致相同，复制“饮食方案”节点，并粘贴到编排面板中，将节点名改为“运动方案”，并修改系统提示词，提示词参考如下：

```
你是一个糖尿病生活指导师，可以根据用户信息，用户当前的生活习惯给出运动方面的生活方案，包括早晚运动和周末运动，在制定运动时需要考虑用户的身体情况及限制。尽量不要与用户给出的方案建议进行冲突
输出内容为列表形式包括执行时间，标题，内容，输出格式为JSON格式，除JSON数据外不要输出任何其他内容，例如：
JSON:
[
{"time":"7:00-7:30","title":"晨练运动":"content":"三十分钟的慢跑，一周三次"}
]
```

将“开始”节点与“运动方案”节点相连，使得“饮食方案”和“运动方案”节点可以并行执行，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F484f02510f3740b0945914a684a1f1ac.png)

图4 工作流结构

在以上提示词中设置输出格式为：“JSON：xxx”，这样描述可以用于将输出内容设置为固定格式，方便后续处理和向量化。接下来在饮食方案后添加一个“代码执行”节点，将上一节点中输出的字符串转为对象数组，配置如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c344d9d7908b473b87ea08696e36cf81%2Frichtext%2Fimage%2F20250904%2Faf7440e92b07494e87552fc41b20971e.png)

图5 代码节点

代码如下：

```
 function main({text,userId}) {
    //取text中“JSON:”后面的文本
    const jsonStr = text.slice(text.indexOf("JSON:") + 5);
    // 解析 JSON 字符串为对象数组
    const jsonArray = JSON.parse(jsonStr);
    // 加上order字段和type字段
    for (let i = 0; i < jsonArray.length; i++) {
        jsonArray[i].order = i + 1;
        jsonArray[i].type = "饮食";
        jsonArray[i].userId = userId;
    }
    return {
        result: jsonArray
    }
}
```

复制以上的代码节点，并添加到运动方案后，此节点功能也是将文本转为对象数组，不过需要注意的是配置type时要将“饮食”改为“运动”。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c344d9d7908b473b87ea08696e36cf81%2Frichtext%2Fimage%2F20250904%2F2c31fd4930f2406396490e08884b53cd.png)

图6 代码节点

在以上两个代码执行节点后新增一个“代码执行”节点，用于将之前的两个节点中的数组对象合并为一个数组对象，配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fd9ef3e1436c14374b6a3922233e9b0fa.png)

图7 代码节点

代码如下：

```
function main({arr1,arr2}) {
    let arr = [];

    for (let i = 0; i < arr1.length; i++) {
        arr.push(arr1[i]); 
    }
    for (let i = 0; i < arr2.length; i++) {
        arr.push(arr2[i]); 
    }
    return {
        arr: arr
    }
}
```

合并成功后，就可以将生活方案写入到数据库中，不过在写入数据库前，需要删除原先的生活方案，在上一节点后续添加“database_workflow”节点，并构造删除语句，语句配置如下：

```
DELETE from life_plans where user_id={{userId}}
```

参考下图：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fee75bb24395d483cb644da8e0381f843.png)

图8 删除方案

接下来新增一个代码执行节点，用于构造SQL语句，配置如下所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F97f7210efbff40d1a5676ea569e177d5.png)

图9 构建SQL

代码如下：

```
function main({arr}) {
    let begin = "INSERT INTO life_plans (user_id, type, `order`, time, title, content) VALUES "
    let data=""
    if(arr.length == 0){
        data = "()"
    }
    for (let i = 0; i < arr.length; i++) {
        let element = arr[i]
        data+=`(${element.userId},'${element.type}',${element.order},'${element.time}','${element.title}','${element.content}')`
        if(i<arr.length-1){
            data+=","
        } 
    }
    return {
        sql: (begin+data)
    }
}
```

在此节点后新增一个“database_workflow”节点，节点功能为执行sql语句添加生活方案，配置参考如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fffb2f669342e435990e3394755efb536.png)

图10 调用工具新增方案

最后添加结束节点，输出合并后的方案列表，配置如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fc52a8395290d4971951b90dd9b50d840.png)

图11 配置结束节点

接下来进行运行测试，运行配置如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F511c7538bbda4a0b939b29670d0700e5.png)

图12 运行测试

 

运行结果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F136fdcdcd3d74c95a27d93ea41ec525f.png)

图13 运行测试