任务详情

任务5-3：使用DeepSeek实现个人信息展示与糖尿病风险评估功能

建议工时：

1

任务描述

**1. 任务描述**

本任务通过DeepSeek&Cline实现个人信息展示与糖尿病风险评估功能。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、LocalStorage；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Ff2804a80cceb4d0f93d5e5df014fcc41.png)

图1 个人信息页面

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250425%2F2e23e10a752149a298e539a359e1e6a1.png)

图2 糖尿病风险信息页面



任务指导

##### 1. 实现用户信息展示功能

由于在后续任务中需要多次用到用户信息，所以可以先在user.js中创建用户信息获取函数，然后在个人信息页面中调用该函数，加载个人信息。

通过DeepSeek&Cline实现用户信息获取功能，可参考以下提示词。

```
@/js/user.js 在此页面中实现用户风险信息展示函数，需要考虑到通过缓存来提升执行效率，通过用户id获取用户风险信息，请求接口可参考@/main/main.html 
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Ffc9dd23b2f1f4af3a8c907ef13ced586.png)

图1 提示词

然后在个人信息界面中获取并展示，参考提示词如下：

```
@/userinfo\userinfo.html 在此页面加载时调用获取用户风险信息的方法，并且将数据展示在界面中
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fb19ec568b9d94845b62b83c41db89b6b.png)

图2 提示词

除此之外，还需要考虑如果用户没有填写风险信息，则需要提醒用户去填写，并且首次请求加载时间较长，可以添加加载动画来优化用户体验。

参考提示词如下。

```
@/userinfo\userinfo.html 添加对用户数据是否为空的判断，如果为空，则在页面中展示编辑信息的HTML代码，并且在获取用户风险信息请求前添加加载效果，请求后取消加载效果，加载效果可参考@/js/loading.js 
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fe6234de7bd714c5f80e5ff66cef81fa8.png)

图3 提示词

最终效果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fa8ccd8f653ac45cdaff0719d00ac6539.png)

图4 数据加载

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Ff2804a80cceb4d0f93d5e5df014fcc41.png)

图5 信息展示

最后添加mine.html到本页面的点击跳转功能与从修改信息页面到是否患病选择页面的跳转。

除此之外，还需要注意在之前任务中的医师咨询页面中聊天工作流调用时，由于当时没有实现用户信息获取方法，是先通过模拟数据的方式来实现聊天助手中用户数据填充功能，在实现获取用户数据方法后，则可以在聊天前加载用户数据，让医师了解用户的基本信息。

参考提示词如下。

```
@/chat\js\chat.js  在此文件中实现用户风险数据加载功能，调用@/js\user.js 中的getUserRiskInfo加载用户数据。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250427%2Fa10f0a3a456c49ebabe114785410486d.png)

图6  聊天页面数据加载提示词

##### 2. 实现用户信息填写功能

用户信息填写分为是否患病填写和个人信息填写，首先填写是否患病，如果选择“是”则展示是患的哪种糖尿病，参考提示词如下。

```
@/userinfo/diabetesinfo.html 如果选择是，则显示患糖尿病的类型，当点击下一步按钮时，判断选择是或者否，如果选择是则判断是否选择了糖尿病类型，没有选择则添加提示，选择后则携带选择结果（否或糖尿病类型）跳转至informationGathering.html页面
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F6578019d35ca49e8958226c666550511.png)

图7 用户信息填写提示词

在个人信息填写页面完成后，点击提交按钮则调用上一任务中实现的工作流来进行个人信息保存和糖尿病风险预测，首先实现工作流调用api，在Dify中，工作流是通过秘钥来进行区分，为上一任务的工作流创建秘钥。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F83370059d99e4cf9ab37b02e3f1cc6c6.png)

图8 生成秘钥

然后在api.js中添加信息管理与风险预测工作流，代码如下。注意{秘钥}要替换为上一步骤中创建的秘钥。

```
const DD_AUTH_TOKEN = "Bearer {秘钥}"
// 糖尿病检测工作流函数
async function fetchDiabetesDetectionWorkflow(inputs, userId) {
    return await fetchWorkflowData(inputs, userId, DD_AUTH_TOKEN);
}
```

然后在个人信息填写界面加载时，获取是否患病页面中传来的参数，为提交按钮设置点击事件，当点击时，获取用户id，并判断是否必填项都填写完毕，如果填写完毕后则调用糖尿病检测工作流来保存数据并检测糖尿病风险，需要注意的是，如果用户选择性别为女，则还需要展示是否处于妊娠期内。

参考提示词如下：

```
@/userinfo/informationGathering.html 在此页面加载时获取url中diabetes的值，当用户选择性别为女时，显示是否处于妊娠期的选项，当用户点击按钮提交时，调用@/js/api.js 中fetchDiabetesDetectionWorkflow函数来提交，inputs中包含用户id，diabetes及表单中的所有数据，需要注意在调用时需要添加加载的特效，可使用@/js/loading.js  的代码。
```

工作流返回数据包括患病情况与分析内容，如果未患病则进行风险预测，携带分析内容跳转至风险预测结果界面，如果已患病则返回个人信息界面，由于该功能比较简单，直接修改发送请求后的处理，参考代码如下。

```
//将结果保存到本地
localStorage.setItem("riskInfo",JSON.stringify(res))

if(res.disease=="否"){
   window.location.replace("riskOutcome.html")
}else{
   window.location.replace("userinfo.html")
}
```

##### 3. 风险展示界面设计

风险展示界面包括风险等级和健康提示，在上一步中将工作流的相应结果存储到LocalStorage中，在本步骤中将在风险展示页面读取并展示风险提示信息，可参考以下提示词。

```
@/userinfo\riskOutcome.html 当此页面加载时，提取出LocalStorage中riskInfo，并获取其result作为风险信息，风险信息包含风险状态与健康建议，参考“【高风险】你的健康建议是xxx”，将此字符串进行拆分，并且渲染到界面中
```

最终效果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250425%2F2e23e10a752149a298e539a359e1e6a1.png)

图9 风险展示

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，固任务实现仅供参考。

1.js/api.js中添加信息管理与风险预测工作流，代码如下：

```
const DD_AUTH_TOKEN = "Bearer {秘钥}"
// 糖尿病检测工作流函数
async function fetchDiabetesDetectionWorkflow(inputs, userId) {
    return await fetchWorkflowData(inputs, userId, DD_AUTH_TOKEN);
}
```

并在js/user.js中添加获取用户信息函数，代码如下：

```
function getUserInfo() {
    const userJson = localStorage.getItem("user")
    if (userJson == null) {
         top.location.href = "/login.html"
        return false 
    } 
    // @ts-ignore
    return JSON.parse(userJson)
}
/**
 * Get user risk information with caching
 * @returns {Promise<Object>} Risk information data
 */
async function getUserRiskInfo() {
    const user = getUserInfo();
    if (!user) {
        throw new Error('User not logged in');
    }
    const userId = user.user_id;
    const cacheKey = `risk_info_${userId}`;
    const cached = sessionStorage.getItem(cacheKey);
    
    if (cached) {
        const { data, timestamp } = JSON.parse(cached);
        // Cache expires after 1 hour (3600000 ms)
        if (Date.now() - timestamp <= 3600000) {
            return data;
        }
    }

    try {
        const data = await fetchSQLWorkflow(`查询id为${userId}的用户的用户风险信息`, "tourist");
        res = data.result
        if (res.length > 0) {
            sessionStorage.setItem(cacheKey, JSON.stringify({
                data: res[0],
                timestamp: Date.now()
            }));
        }
        return res[0];
    } catch (error) {
        console.error('获取用户风险信息失败:', error);
        return null;
    }
}
```

\2. userinfo/userinfo.html代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>个人信息收集</title>
    <script src="/js/tailwindcss.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <script>

    </script>
    <script src="/js/user.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/loading.js"></script>
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="css/userinfo.css">
</head>

<body class="min-h-[1024px] bg-[#F3F8FC]">
    <div class="flex flex-col h-screen">
        <!-- 顶部导航栏 -->
        <header class="shadow-sm bg-white">
            <div class="flex items-center justify-between px-4 py-3">
                <button onclick="window.history.back()">
                    <i class="fas fa-chevron-left text-lg"></i>
                </button>
                <h1 class="text-lg font-bold ">个人信息</h1>
                <button onclick="window.location.replace('diabetesinfo.html')">
                    修改信息
                </button>
            </div>

        </header>
        <div id="userInfo" class="space-y-6">
            <div class="bg-gray-50 p-6 rounded-lg shadow-md mt-6 mx-6 mb-4">
                <h3 class="text-lg font-bold text-gray-900 mb-4 pb-2 border-b border-dashed border-gray-300">基本信息</h3>
                <div class="space-y-4">
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">年龄:</span>
                        <span id="userAge" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">性别:</span>
                        <span id="userGender" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">家族病史:</span>
                        <span id="familyHistory" class="text-gray-900 text-right"></span>
                    </div>
                </div>
            </div>
            <div class="bg-gray-50 p-6 rounded-lg shadow-md mt-6 mx-6 mb-4">
                <h3 class="text-lg font-bold text-gray-900 mb-4 pb-2 border-b border-dashed border-gray-300">身体数据</h3>
                <div class="space-y-4">
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">身高(cm):</span>
                        <span id="height" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">体重(kg):</span>
                        <span id="weight" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-20">腰围(cm):</span>
                        <span id="waistline" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-500 w-32">收缩压(mmHg):</span>
                        <span id="systolicPressure" class="text-gray-900 text-right"></span>
                    </div>
                </div>
            </div>
            <div class="bg-gray-50 p-6 rounded-lg shadow-md mt-6 mx-6 mb-4">
                <h3 class="text-lg font-bold text-gray-900 mb-4 pb-2 border-b border-dashed border-gray-300">糖尿病病情</h3>
                <div class="space-y-4">
                    <div class="flex items-center">
                        <span class="text-gray-500 w-20">患病情况</span>
                        <span id="disease" class="text-gray-900 text-right"></span>
                    </div>
                    <div class="mb-2">
                        <div class="text-gray-500">风险及建议</div>
                        <div id="message" class="text-gray-900 mt-1"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>


</body>

</html>
```

对应的/userinfo/js/userinfo.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#3B82F6',
                secondary: '#64748B'
            },
            borderRadius: {
                'none': '0px',
                'sm': '2px',
                DEFAULT: '4px',
                'md': '8px',
                'lg': '12px',
                'xl': '16px',
                '2xl': '20px',
                '3xl': '24px',
                'full': '9999px',
                'button': '4px'
            }
        }
    }
}

window.onload = async function () {
    //sessionStorage删除以risk_info_开头的键值对
    for (let i = 0; i < sessionStorage.length; i++) {
        let key = sessionStorage.key(i);
        if (key.startsWith("risk_info_")) {
            sessionStorage.removeItem(key);
        } 
    }
    try {
        // Check if we have existing user data
        let userData = {};
        const loadingOverlay = startLoading('正在获取用户信息...');

        try {
            userData = await getUserRiskInfo();

            // If all fields are empty, show edit form
            if (!userData) {
                document.getElementById('userInfo').innerHTML = `
                <div class="p-6 rounded-lg">
                    <h2 class="text-xl font-medium mb-4">尚未填写健康信息</h2>
                    <p class="mb-6">请填写您的健康信息以获取糖尿病风险评估</p>
                    <div class="flex justify-end">
                        <button onclick="window.location.replace('diabetesinfo.html')" 
                            class="bg-black text-white px-6 py-3 rounded-button hover:bg-blue-700 transition-all">
                            <i class="fas fa-edit mr-2"></i>
                            立即填写
                        </button>
                    </div>
                </div>
            `;
            } else {
                // 安全更新元素内容
                const updateElement = (id, content) => {
                    const el = document.getElementById(id);
                    if (el) el.textContent = content;
                };

                // 更新基本信息
                updateElement('userAge', `${userData.age} 岁` || '未填写');
                updateElement('userGender', userData.sex || '未填写');
                updateElement('familyHistory', userData.family_history || '无');

                // 更新身体数据
                updateElement('height', `${userData.height}` || '未测量');
                updateElement('weight', `${userData.weight}` || '未测量');
                updateElement('waistline', `${userData.waistline}` || '未测量');
                updateElement('systolicPressure', `${userData.systolicPressure}` || '未测量');

                // 更新糖尿病病情数据
                updateElement('disease', userData.disease || '未评估');
                updateElement('message', userData.message || '暂无风险评估建议');
            }
        } catch (error) {
            console.error('获取用户信息失败:', error);
            document.getElementById('userInfo').style.display = 'none';
            document.getElementById('editForm').style.display = 'block';
        } finally {
            stopLoading(loadingOverlay);
        }

    } catch (error) {
        console.error('请求失败:', error);
    }
}
```

\3. userinfo/js/diabetesinfo.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4A90E2',
                secondary: '#6BB6FF',
                third: '#F9FAFB'
            },
            borderRadius: {
                'none': '0px',
                'sm': '2px',
                DEFAULT: '4px',
                'md': '8px',
                'lg': '12px',
                'xl': '16px',
                '2xl': '20px',
                '3xl': '24px',
                '6xl': '60px',
                'full': '9999px',
                'button': '4px'
            }
        }
    }
}
window.onload = function () {
    const diabetesStatus = document.getElementById('diabetesStatus');
    const typeButtons = document.querySelectorAll('.bg-white.border');
    const nextBtn = document.getElementById('nextBtn');
    let selectedType = null;
    // Toggle diabetes type visibility based on status
    diabetesStatus.addEventListener('change', function() {
        const typeSection = document.querySelector('.bg-white.rounded-lg.shadow-md.p-4.mt-4:nth-child(2)');

        if (this.value === 'yes') {
            typeSection.style.display = 'block';
        } else {
            typeSection.style.display = 'none';
            selectedType = null;
        }
    });

    // Initialize - hide type selection if default is "no"
    if (diabetesStatus.value === 'no') {
        document.querySelector('.bg-white.rounded-lg.shadow-md.p-4.mt-4:nth-child(2)').style.display = 'none';
    }

    // Handle type selection
    typeButtons.forEach(button => {
        button.addEventListener('click', function() {
            // Remove selection from all buttons
            typeButtons.forEach(btn => {
                btn.classList.remove('border-primary', 'bg-third');
            });
            
            // Add selection to clicked button
            this.classList.add('border-primary', 'bg-third');
            selectedType = this.querySelector('p:first-child').textContent;
        });
    });

    // Handle next button click
    nextBtn.addEventListener('click', function() {
        if (diabetesStatus.value === 'yes' && !selectedType) {
            alert('请选择糖尿病类型');
            return;
        }

        const result = diabetesStatus.value === 'no' ? '否' : selectedType;
        window.location.replace(`informationGathering.html?disease=${encodeURIComponent(result)}`);
    }); 
}
```

\4. userinfo/js/informationGathering.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#3B82F6',
                secondary: '#64748B'
            },
            borderRadius: {
                'none': '0px',
                'sm': '2px',
                DEFAULT: '4px',
                'md': '8px',
                'lg': '12px',
                'xl': '16px',
                '2xl': '20px',
                '3xl': '24px',
                'full': '9999px',
                'button': '4px'
            }
        }
    }
}
window.onload = function () {
    // 获取url中的disease参数
    const urlParams = new URLSearchParams(window.location.search);
    const disease = urlParams.get('disease');
    const userinfo = getUserInfo();
    // 控制当性别选择女时添加妊娠期选项
    const genderRadios = document.querySelectorAll('input[name="sex"]');
    const pregnancyGroup = document.getElementById('pregnancyGroup');
    genderRadios.forEach(radio => {
        radio.addEventListener('change', function () {
            if (this.value === '女') {
                pregnancyGroup.style.display = 'block';
            } else {
                pregnancyGroup.style.display = 'none';
            }
        });
    });
    //重写表单提交方法
    const form = document.getElementById('personalInfoForm');
    form.addEventListener('submit', function (event) {
        event.preventDefault();
        let overlay = startLoading()
        const formData = new FormData(form);
        let data = Object.fromEntries(formData.entries());
        data.userId = userinfo.user_id
        data.disease = disease
        const checkboxes = document.querySelectorAll('input[name="familyHistory"]');
        let selectedOptions = "";
        checkboxes.forEach(checkbox => {
            if (checkbox.checked && checkbox.value != "other") {
                selectedOptions += checkbox.value + ",";
            } else if (checkbox.value == "other") {
                let other = document.getElementById("otherInput").value
                selectedOptions += other + ",";
            }
        });
        if (selectedOptions.length > 0) {
            selectedOptions = "无"
        }
        data.familyHistory = selectedOptions;
        fetchDiabetesDetectionWorkflow(data, "users")
            .then(res => {
                stopLoading(overlay)
                localStorage.setItem("riskInfo", JSON.stringify(res))

                if (res.disease == "否") {
                    window.location.replace("riskOutcome.html")
                } else {
                    window.location.replace("userinfo.html")
                }
            })
            .catch(err => {
                stopLoading(overlay)
            })
        console.log(data);
        // fetchDiabetesDetectionWorkflow()
    });
}
```

\5. userinfo/js/riskOutcome.js

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4A90E2',
                secondary: '#FFA500'
            },
            borderRadius: {
                'none': '0px',
                'sm': '2px',
                DEFAULT: '4px',
                'md': '8px',
                'lg': '12px',
                'xl': '16px',
                '2xl': '20px',
                '3xl': '24px',
                'full': '9999px',
                'button': '4px'
            }
        }
    }
}
function extractContent(str) {
    const regex = /【([^】]+)】/;
    const match = str.match(regex);
    const extracted = match ? match[1] : null;
    const remainingText = match ? str.replace(regex, '') : str;
    return {
        extracted: extracted,
        remainingText: remainingText
    };
}
window.onload = async function () {
   // 通过LocalStorage获取风险信息，并加载到界面中
const riskInfo = localStorage.getItem('riskInfo')
if (riskInfo) {
    const riskInfoObj = JSON.parse(riskInfo)
    const result = riskInfoObj.result
    console.log(result)

    let obj = extractContent(result)
    document.getElementById('resultSpan').textContent = obj.extracted
    document.getElementById('resultText').textContent = `您目前处于${obj.extracted}风险水平`
    document.getElementById('messageDiv').innerHTML = obj.remainingText

} 
}
```

 