任务详情

任务6-3：使用DeepSeek实现生活方案展示及方案定制功能

建议工时：

1

任务描述

**1. 任务描述**

本任务实现在生活方案页面中展示用户对应的生活方案信息，用户可以基于此方案进行生活打卡，并且实现调用生活方案定制工作流实现生活方案个性化定制功能。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、LocalStorage；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fa81c168eee2645a099c2ba40c7ca6b73.png)

图1 生活方案展示

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250421%2F0fe7d26bbcbf41bd9a9434314f7d7f03.png)

图2 生活方案生成

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250421%2F397ce3872cd9471cadbde3d0830d4c7d.png)

图3 生活打卡



任务指导

#### 1.生活方案展示

在生活方案展示界面展示着饮食方案与运动方案，通过数据管理工作流可获取当前用户的生活方案，并进行分类，在界面中展示出来，除此之外还需要考虑用户如果没有生成生活方案的情况。

使用DeepSeek&Cline实现生活方案展示，参考以下提示词：

```
@/scheme/ 在此页面加载时调用@/js/api.js 中的fetchSQLWorkflow获取用户的生活方案，如果生活方案列表为空则跳转至noScheme.html页面，如果不为空，则根据type类型分为“饮食”和“运动”分别渲染到界面中。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Ffd0cbdbdf973435986ac6c910be41677.png)

图1 提示词

代码编写并修改完毕后，效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2Fa81c168eee2645a099c2ba40c7ca6b73.png)

图2 生活方案展示

除此之外，还需要考虑右上角调整方案的跳转功能，当点击时弹出弹窗提示，如果进行方案跳转，则会删除当前的方案，参考提示词如下：

```
@/scheme/ 为右上角的“调整方案”添加点击事件，当点击时，弹出提示如果调整方案，原先的方案会被删除，点击确定则跳转至getScheme.html页面
```

生成并调整代码，最终效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250418%2F2d0e341a6d85479abf0034f9c91c6bae.png)

图3 修改生活方案提示

在有时进入方案定制页面时，加载时间较长，可考虑添加加载动画或设置缓存机制，此功能可自行编写提示词，并通过DeepSeek&Cline来实现，在此不做赘述。

 

#### 2.方案定制功能实现

在方案定制页面加载时，会获取用户的信息并填充到页面中，用户可以自行填写生活习惯及方案建议，在用户填写生活习惯及建议后，点击“生成生活方案”即调用生活方案生成工作流。

首先在api.js中定义方案定制工作流调用函数，可参考以下代码。在Dify中对应的工作流中生成秘钥，比替换下方代码的xxx。

```
const LP_AUTH_TOKEN = "Bearer xxx";
// 生活方案定制
async function fetchLifePlansWorkflow(inputs, userId) {
    return await fetchWorkflowData(inputs, userId, LP_AUTH_TOKEN);
}
```

接下来在糖尿病方案生成界面中实现用户数据加载与工作流调用，参考以下提示词。

```
 @/scheme\getScheme.html 在此页面加载时，调用获取用户信息方法，并将用户信息填充至界面的个人信息界面中，当用户点击“生成生活方案”时，调用@/js/api.js 中的fetchLifePlansWorkflow生成生活方案。
```

在调用工作流后，还需要进行数据处理与展示，在此可以直接跳回之前的方案定制展示页面，效果如下图所示，可自行编写提示词并调整提示词，完成以上功能。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250421%2F0fe7d26bbcbf41bd9a9434314f7d7f03.png)

图4 方案生成

#### 3.生活打卡功能实现

用户每天根据方案进行饮食或运动后，可以进行打卡，点击打卡后可以填写今天的打卡说明，包括已达成、超额完成、未达成。点击后将今天的打卡记录写入到数据库表中，并且打卡按钮设置为“已打卡”。在页面加载时也同步加载今日是否打卡的信息。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250421%2F397ce3872cd9471cadbde3d0830d4c7d.png)

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250421%2F42a706db9de345488652e114a6a95940.png)

打卡弹窗可通过sweetalert2弹窗库来实现，可自行编写提示词，通过DeepSeek&Dify来实现此功能。

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，固任务实现仅供参考。

1.在js/api.js中添加生活方案定制工作流的调用，代码如下。

```
const LP_AUTH_TOKEN = "Bearer xxx";
// 生活方案定制
async function fetchLifePlansWorkflow(inputs, userId) {
    return await fetchWorkflowData(inputs, userId, LP_AUTH_TOKEN);
}
```

2.修改/scheme/scheme.html代码，实现生活方案展示及打卡功能，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>糖尿病方案定制</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <script src="/js/showAlert.js"></script>
    <script src="/js/loading.js"></script>
    <script src="js/scheme.js"></script>
    <link rel="stylesheet" href="/css/loading.css">

    <link rel="stylesheet" href="css/scheme.css">
    <link rel="stylesheet" href="/css/all.min.css">
</head>

<body class="w-full max-w-screen-sm mx-auto bg-white min-h-screen">
    <nav class="sticky top-0 w-full bg-white shadow-sm z-50 px-4 h-14 flex items-center justify-between safe-top">
        <button class="flex items-center !rounded-button">
            <i class="fas fa-arrow-left text-gray-600"></i>
        </button>
        <h1 class="text-lg font-bold">生活方案</h1>
        <button class="flex items-center !rounded-button text-primary" onclick="adjustmentScheme()"
            style="color:#3261FF ;">
            调整方案
        </button>
    </nav>
    <main class="pt-4 pb-24 px-4 safe-top safe-bottom">
        <div class="space-y-4">

            <div class="flex items-center justify-between main_item">
                <div class="flex items-center">
                    <div>
                        <h3 class="text-base sm:text-lg font-medium">饮食管理</h3>
                        <p class="text-sm sm:text-base">定制专属饮食计划</p>
                    </div>
                </div>
                <button id="eatBtn" class="px-4 py-1 text-xs bg-primary text-white rounded-button btn"
                    onclick="checkIn('饮食')">
                    打卡
                </button>
            </div>
            <div class="space-y-2" id="eat">
            </div>
            <div class="flex items-center justify-between main_item">
                <div class="flex items-center">
                    <div>
                        <h3 class="text-base sm:text-lg font-medium">运动管理</h3>
                        <p class="text-sm sm:text-base">科学运动指导</p>
                    </div>
                </div>
                <button id="sportBtn" class="px-4 py-1 text-xs bg-primary text-white rounded-button btn"
                    onclick="checkIn('运动')">
                    打卡
                </button>
            </div>
            <div class="space-y-2" id="sport">
            </div>
    </main>
</body>

</html>
```

对应的scheme.css代码如下：

```
body {
    background-color: #F8FAFC;
}

@supports(padding: max(0px)) {
    .safe-top {
        padding-top: max(12px, env(safe-area-inset-top));
    }

    .safe-bottom {
        padding-bottom: max(16px, env(safe-area-inset-bottom));
    }

    .safe-left {
        padding-left: max(16px, env(safe-area-inset-left));
    }

    .safe-right {
        padding-right: max(16px, env(safe-area-inset-right));
    }
}

.btn {
    background-color: #3261FF;
    border-radius: 10px;
}

.chart-container {
    width: 100%;
    height: 120px;
}

input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    background: #4A90E2;
    border-radius: 50%;
    cursor: pointer;
}

.main_item {
    background-image: url("/img/plan_bg.png");
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    color: white;
    padding: 12px 16px;
}
```

对应的scheme.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#3261FF',
                secondary: '#81C784'
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
// Function to add new meal item
function addMeal(type, time, foods) {
    const mealsContainer = document.getElementById('eat');
    const newMeal = document.createElement('div');
    newMeal.className = 'bg-blue-50 p-3 rounded-lg';
    newMeal.innerHTML = `
        <div class="flex items-center justify-between mb-2">
            <span class="text-sm font-medium">${type}</span>
            <span class="text-xs text-gray-500">${time}</span>
        </div>
        <p class="text-xs text-gray-600">${foods}</p>
    `;
    mealsContainer.appendChild(newMeal);
}

// Function to add new exercise item
function addExercise(name, time, description) {
    const exercisesContainer = document.getElementById('sport');
    const newExercise = document.createElement('div');
    newExercise.className = 'bg-blue-50 p-3 rounded-lg';
    newExercise.innerHTML = `
        <div class="flex items-center justify-between mb-2">
            <span class="text-sm font-medium">${name}</span>
            <span class="text-xs text-gray-500">${time}</span>
        </div>
        <p class="text-xs text-gray-600">${description}</p>
    `;
    exercisesContainer.appendChild(newExercise);
}
//获取当前日期，格式为YYYY-mm-DD
function getDate() {
    const date = new Date();
    const year = date.getFullYear();
    const month = (date.getMonth() + 1).toString().padStart(2, '0');
    const day = date.getDate().toString().padStart(2, '0');
    return `${year}-${month}-${day}`;
}
function adjustmentScheme() {
    Swal.fire({
        title: '是否要调整方案',
        text: '原先的方案会被删除',
        icon: 'info',
        showCancelButton: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消'
    }).then((result) => {
        if (result.isConfirmed) {
            top.location.href = 'getScheme.html'
        }
    });
}
function loadData(data) {
    if (data.length == 0) {
        window.location.href = "noscheme.html"
    } else {
        let eatList = []
        let sportList = []

        data.forEach(item => {
            if (item.type == "饮食") {
                eatList.push(item)
            } else {
                sportList.push(item)
            }
        })
        eatList.sort((a, b) => a.order - b.order)
        sportList.sort((a, b) => a.order - b.order)
        eatList.forEach(item => {
            addMeal(item.title, item.time, item.content)
        })
        sportList.forEach(item => {
            addExercise(item.title, item.time, item.content)
        })
        const userinfo = getUserInfo()
    }
}

//获取当前日期，格式为YYYY-mm-DD
function getDate() {
    const date = new Date();
    const year = date.getFullYear();
    const month = (date.getMonth() + 1).toString().padStart(2, '0');
    const day = date.getDate().toString().padStart(2, '0');
    return `${year}-${month}-${day}`;
}
function checkIn(type) {
    Swal.fire({
        title: `今天达成了${type}计划了吗`,
        icon: 'question',
        showDenyButton: true,
        showCancelButton: true,
        confirmButtonText: '已达成',
        denyButtonText: '超额完成',
        cancelButtonText: '未达成',
        confirmButtonColor: '#3b82f6', // blue-500
        denyButtonColor: '#22c55e', // green-500
        cancelButtonColor: '#ef4444', // red-500
        input: 'textarea',
        inputPlaceholder: '请填写完成情况说明',
        inputValidator: (value) => {
            // Optional field - no validation needed
            return false;
        }
    }).then((result) => {
        const userinfo = getUserInfo()
        // value为textarea的value
        let value = ''
        let areas = document.getElementsByClassName('swal2-textarea')
        if (areas.length > 0) {
            value = areas[0].value
        }
        let status = ""
        if (result.isConfirmed) {
            status = "已达成"
        } else if (result.isDenied) {
            status = "超额完成"
        } else {
            status = "未达成"
        }
        const text = `为用户id为${userinfo.user_id}的用户添加一条punch_in，message为："${value}"，status为："${status}",type为"${type}"，打卡时间为"${getDate()}"`
        if (type == "饮食") {
            changeCardStatus(document.getElementById("eatBtn"))
        } else {
            changeCardStatus(document.getElementById("sportBtn"))
        }
        fetchSQLWorkflow(text, "system")
            .then(res => {
                if (res.result[0].result > 0) {
                    showFloatingAlert("打卡成功", "success")
                } else {
                    showFloatingAlert("打卡失败", "error")
                }
            })
    });
}
//传入一个button，将文字设置为“已打卡”，并将按钮颜色设置为绿色，点击事件设置为null
function changeCardStatus(element) {
    element.innerHTML = "已打卡"
    element.style.backgroundColor = "#22c55e"
    element.onclick = null
}
window.onload = function () {
    let loading = startLoading()
    let loadDataTotal = 0
    const userinfo = getUserInfo()
    fetchSQLWorkflow(`查询id为${userinfo.user_id}的用户的生活方案数据`, "tourist")
        .then(res => {
            data = res.result
            if (data.length != 0) {
                sessionStorage.setItem("scheme", JSON.stringify(data))
            }
            loadData(data)
            loadDataTotal++
            if (loadDataTotal == 2) {
                stopLoading(loading)
            }
        })
    fetchSQLWorkflow(`查询用户id为${userinfo.user_id}，且日期为${getDate()}打卡数据`, "system")
        .then(res => {
            data = res.result
            console.log(data)
            if (data.length != 0) {
                data.forEach(item => {
                    if (item.punch_type == "饮食") {
                        changeCardStatus(document.getElementById("eatBtn"))
                    } else {
                        changeCardStatus(document.getElementById("sportBtn"))
                    }
                })
            }
           loadDataTotal++
            if (loadDataTotal == 2) {
                stopLoading(loading)
            }
        })



}
```

2.修改scheme/getScheme.html代码，实现生活方案定制功能，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title class="font-pacifico bg-gradient-to-r from-primary to-secondary bg-clip-text text-transparent">生活方案制定
    </title>
    <script src="/js/tailwindcss.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <script src="js/getScheme.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/loading.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="css/getScheme.css">

</head>

<body class="bg-white w-full min-h-screen relative">
    <!-- 顶部导航栏 -->
    <header class="fixed top-0 left-0 right-0 z-10 shadow-sm bg-white">
        <div class="flex items-center justify-between px-4 py-3">
            <button>
            </button>
            <h1 class="text-lg font-bold">方案定制</h1>
            <button>
            </button>
        </div>
    </header>
    <main class="pt-16 px-4 pb-8 flex flex-col max-w-md mx-auto w-full">
        <div class="w-full flex flex-col gap-4">
            <img src="/img/getScheme_title.jpg" alt="糖尿病生活管理插画" >

            <div class="w-full">
                <h2 class="text-lg font-bold mb-3">个人信息</h2>
                <div class="space-y-3 card">
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">年龄</span>
                        <span id="age" class="text-gray-900">18</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">性别</span>
                        <span id="sex" class="text-gray-900">男</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">身高</span>
                        <span id="height" class="text-gray-900">170</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">体重</span>
                        <span id="weight" class="text-gray-900">75</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">是否患病</span>
                        <span id="disease" class="text-gray-900">否</span>
                    </div>
                </div>
            </div>
            <div class="w-full">
                <h2 class="text-lg font-bold mb-3">生活习惯</h2>
                <div class="space-y-3 card">
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">作息时间</span>
                        <select
                            class="w-32 h-9 px-3 bg-gray-50 !rounded-button appearance-none text-right pr-8 relative">
                            <option>请选择</option>
                            <option>早睡早起</option>
                            <option>晚睡晚起</option>
                            <option>早睡晚起</option>
                            <option>晚睡早起</option>
                        </select>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">是否经常做饭</span>
                        <select
                            class="w-32 h-9 px-3 bg-gray-50 !rounded-button appearance-none text-right pr-8 relative">
                            <option>请选择</option>
                            <option>经常</option>
                            <option>偶尔</option>
                            <option>很少</option>
                        </select>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">饮食口味</span>
                        <select
                            class="w-32 h-9 px-3 bg-gray-50 !rounded-button appearance-none text-right pr-8 relative">
                            <option>请选择</option>
                            <option>清淡</option>
                            <option>适中</option>
                            <option>重口味</option>
                        </select>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">运动习惯</span>
                        <select
                            class="w-32 h-9 px-3 bg-gray-50 !rounded-button appearance-none text-right pr-8 relative">
                            <option>请选择</option>
                            <option>经常运动</option>
                            <option>偶尔运动</option>
                            <option>很少运动</option>
                        </select>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-gray-600">饮酒习惯</span>
                        <select
                            class="w-32 h-9 px-3 bg-gray-50 !rounded-button appearance-none text-right pr-8 relative">
                            <option>请选择</option>
                            <option>从不</option>
                            <option>偶尔</option>
                            <option>经常</option>
                        </select>
                    </div>

                </div>
            </div>
            <div class="mt-4">
                <h2 class="text-lg font-bold mb-3">方案建议</h2>
                <div class="flex items-start gap-2 card">
                    <textarea placeholder="请输入您对生成方案的建议..."
                        class="w-full h-24 px-3 py-2 bg-gray-50 !rounded-button border-none resize-none"></textarea>
                </div>
            </div>
            <button
                class="w-full h-12 bg-primary text-white font-medium !rounded-button flex items-center justify-center active:opacity-90 transition-opacity mt-6"
                onclick="getLifeScheme()">
                生成生活方案
            </button>
        </div>
    </main>

</body>

</html>
```

对应的getScheme.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4F46E5',
                secondary: '#818CF8'
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
                'button': '4px',
            }
        }
    }
}
function getLifeScheme() {
    // 获取个人信息
    const age = document.getElementById('age').textContent;
    const sex = document.getElementById('sex').textContent;
    const height = document.getElementById('height').textContent;
    const weight = document.getElementById('weight').textContent;
    const disease = document.getElementById('disease').textContent;

    // 获取生活习惯
    const selects = document.querySelectorAll('select');
    const sleepSchedule = selects[0].value;
    const cookingFreq = selects[1].value;
    const dietPreference = selects[2].value;
    const exerciseHabits = selects[3].value;
    const drinkingHabits = selects[4].value;

    // 验证生活习惯数据
    if (sleepSchedule === '请选择' || cookingFreq === '请选择' ||
        dietPreference === '请选择' || exerciseHabits === '请选择' ||
        drinkingHabits === '请选择') {
        Swal.fire({
            title: '请完善生活习惯信息',
            text: '所有生活习惯选项都需要选择',
            icon: 'warning'
        });
        return null;
    }
    const userInfo = `个人信息:
                        年龄: ${age}
                        性别: ${sex}
                        身高: ${height}
                        体重: ${weight}
                        是否患病: ${disease}
                        `
    const habit = `生活习惯:
                        作息时间: ${sleepSchedule}
                        做饭频率: ${cookingFreq}
                        饮食口味: ${dietPreference}
                        `
    // 获取要保持的习惯
    const suggestion = document.querySelector('textarea').value;
    const userinfo = getUserInfo()
    const data = {userInfo,habit,suggestion,userId:userinfo.user_id}
    const loading = startLoading("正在生成方案")
    fetchLifePlansWorkflow(data,userinfo.user_id)
        .then(res => {
            stopLoading(loading)
            console.log(res)
            if (res.length > 0) {
                //通过swal显示方案生成成功提示，点击则跳转至方案详情页
                Swal.fire({
                    title: '方案生成成功',
                    text: '点击确认跳转至方案详情页',
                    icon: 'success',
                    confirmButtonText: '确定',
                }).then((result) => {
                if (result.isConfirmed) {
                    top.location.replace('scheme.html')
                }
            });
            }else{
                Swal.fire({
                    title: '方案生成失败',
                    text: '请稍后再试',
                    icon:'error',
                    confirmButtonText: '确定',
                })
            }

        })

}
window.onload = function () {
    let loading = startLoading("正在加载用户信息")
    sessionStorage.removeItem('scheme')
    let riskInfo = null
    getUserRiskInfo().then(ri => {
        stopLoading(loading)
        riskInfo = ri
        console.log(riskInfo)
        if (riskInfo) {
            console.log(riskInfo)
            document.getElementById('age').textContent = riskInfo.age
            document.getElementById('sex').textContent = riskInfo.sex
            document.getElementById('height').textContent = riskInfo.height + "cm"
            document.getElementById('weight').textContent = riskInfo.weight + "kg"
            document.getElementById('disease').textContent = riskInfo.disease

        } else {
            Swal.fire({
                title: '请先添加用户信息',
                text: '点击确认跳转至添加用户信息页面',
                icon: 'info',
                showCancelButton: true,
                confirmButtonText: '确定',
                cancelButtonText: '取消'
            }).then((result) => {
                if (result.isConfirmed) {
                    top.location.href = '/userinfo/userinfo.html'
                }
            });
        }
    })

}
```