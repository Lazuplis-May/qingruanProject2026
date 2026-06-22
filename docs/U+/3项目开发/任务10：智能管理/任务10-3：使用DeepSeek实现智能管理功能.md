任务详情

任务10-3：使用DeepSeek实现智能管理功能

建议工时：

1

任务描述

**1. 任务描述**

本任务实现智能智能管理功能，通过在智能管理平台中调用管理助手智能体，实现通过自然语言对网站的数据进行管理与维护。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、LocalStorage；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Ff76e31c79ddf4672ae1426d587291c82.png)

图1 智能管理页面



任务指导

##### 1. 智能管理功能

在智能管理平台中输入想要执行的操作内容，点击右侧按钮则调用智能体执行内容，在执行结果中展示执行状态与操作，在数据概览中展示查询的数据信息（如果无数据则不展示），实现效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F1d2fda99b4bc45479d52ad1c544edad8.png)

图1 数据查询

自行编写提示词，并通过DeepSeek&Cline实现此功能，在此不做赘述。

##### 2. 历史操作列表展示

每次执行完操作后，将操作存入本地数据中，当页面加载时则自动加载该操作历史信息，当点击操作历史列表中的列表项时，将执行结果与数据概览展示到界面中，如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Ff76e31c79ddf4672ae1426d587291c82.png)

图2 历史操作日志

自行编写提示词，并通过DeepSeek&Cline实现此功能，在此不做赘述。

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，固任务实现仅供参考。

修改ai/adminAI.html页面代码，实现智能管理功能机操作日志展示功能，代码如下。

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 智能管理</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <script src="/js/loading.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="/css/loading.css">
    <script src="js/adminAI.js"></script>
    <link rel="stylesheet" href="css/adminAI.css">
</head>

<body class="w-full min-h-screen mx-auto relative">
    <!-- 顶部导航栏 -->
    <header class="fixed top-0 left-0 right-0 z-10 shadow-sm bg-white">
        <div class="flex items-center justify-between px-4 py-3">
            <button onclick="window.history.back()">
                <i class="fas fa-chevron-left text-lg"></i>
            </button>
            <h1 class="text-lg font-medium">AI智能管理平台</h1>
            <button id="clearChatBtn">
                <i class="fa-regular fa-circle-question text-lg"></i>


            </button>
        </div>
    </header>
    <main class="pt-[72px] px-4 w-full max-w-md mx-auto">
        <div class="mb-2">
            <div class="relative">
                <input type="text" placeholder="请输入操作指令"
                    class="w-full h-[48px] bg-white rounded-lg p-4 text-sm border-none input-shadow">
                <button id="btn" class="absolute right-4 top-1/2 transform -translate-y-1/2">
                    <i class="fas fa-search text-gray-400"></i>
                </button>
            </div>
        </div>
        <div class="space-y-4">
            <div class="bg-white p-4 rounded-lg card-shadow">
                <h3 class="text-base font-medium mb-3">执行结果</h3>
                <div class="space-y-3">
                    <div class="flex items-center space-x-2">
                        <span class="text-sm text-gray-600">状态：</span>
                        <span class="text-sm text-gray-800" id="status"></span>
                    </div>
                    <div class="flex items-center space-x-2">
                        <span class="text-sm text-gray-600">操作：</span>
                        <span class="text-sm text-gray-800" id="message"></span>
                    </div>
                </div>
            </div>
            <div class="bg-white p-4 rounded-lg card-shadow">
                <h3 class="text-base font-medium mb-3">数据概览</h3>
                <div class="overflow-x-auto w-full" id="dataTableContainer">
                </div>
            </div>
            <div class="bg-white p-4 rounded-lg card-shadow">
                <h3 class="text-base font-medium mb-3">操作日志</h3>
                <div class="space-y-3">
                </div>
            </div>
        </div>
    </main>
</body>

</html>
```

对应的“ai\js\adminAI.js”代码如下。需要注意在调用智能体代码中将秘钥换成自己生成的秘钥。

```
// 表格设置函数
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#007AFF',
                secondary: '#5856D6'
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
// 表格设置函数
function renderObjectTable(data) {
    const container = document.getElementById('dataTableContainer');
    if (!container || !Array.isArray(data) || data.length === 0) return;

    // Create table element
    const table = document.createElement('table');
    table.className = 'w-full min-w-max whitespace-nowrap';

    // Create table header
    const thead = document.createElement('thead');
    const headerRow = document.createElement('tr');
    headerRow.className = 'border-b border-gray-200';

    // Get headers from first object keys
    Object.keys(data[0]).forEach(key => {
        const th = document.createElement('th');
        th.className = 'py-2 px-3 text-left text-sm font-medium text-gray-600';
        th.textContent = key;
        headerRow.appendChild(th);
    });
    thead.appendChild(headerRow);
    table.appendChild(thead);

    // Create table body
    const tbody = document.createElement('tbody');
    data.forEach(item => {
        const row = document.createElement('tr');
        row.className = 'border-b border-gray-100';

        Object.values(item).forEach(value => {
            const td = document.createElement('td');
            td.className = 'py-2 px-3 text-sm text-gray-600';
            td.textContent = typeof value === 'object' ? JSON.stringify(value) : value;
            row.appendChild(td);
        });

        tbody.appendChild(row);
    });
    table.appendChild(tbody);

    // Clear container and append table
    container.innerHTML = '';
    container.appendChild(table);
}
function getOperationLogs() {
    return JSON.parse(localStorage.getItem('operationLogs') || '[]');
}
// Operation log functions
function generateUUID() {
    return crypto.randomUUID();
}

function saveOperationLog(requirement, result) {
    const logs = getOperationLogs();
    logs.push({
        id: generateUUID(),
        requirement,
        result:JSON.stringify(result),
        time: new Date().toLocaleString()
    });
    localStorage.setItem('operationLogs', JSON.stringify(logs));
    displayOperationLogs();
}
function displayOperationLogs() {
    const logs = getOperationLogs();
    const logContainer = document.querySelector('.bg-white.p-4.rounded-lg.card-shadow:last-child .space-y-3');
    logContainer.innerHTML = '';

    logs.slice().reverse().forEach(log => {
        const logItem = document.createElement('div');
        logItem.className = 'flex flex-col space-y-1 p-2 bg-gray-50 rounded';
        logItem.innerHTML = `
                        <div class="flex justify-between">
                            <span class="text-sm font-medium">${log.requirement}</span>
                            <span class="text-xs text-gray-500">${log.time}</span>
                        </div>
                        <div class="text-sm text-gray-600">${JSON.parse(log.result).message}</div>
                    `;
        logItem.addEventListener('click', () => loadLog(log.id));                
        logContainer.appendChild(logItem);
    });
}
function json2Object(str) {
    // 首先去除首位的json'''和'''
    str = str.replace(/^```json|```$/g, '');
    // 然后使用JSON.parse解析字符串为对象
    return JSON.parse(str);
}
//加载请求结果
function loadResult(result) {
    if (result.data) {
        renderObjectTable(result.data)
    }
    if (result.status) {
        document.getElementById('status').innerHTML = result.status;
    }
    if (result.message) {
        document.getElementById('message').innerHTML = result.message;
    }
}
function loadLog(id) {
    const logs = getOperationLogs()
    const log = logs.find(log => log.id === id)
    if (log) {
        loadResult(JSON.parse(log.result))
        document.querySelector('input').value = log.requirement
    }

}
let containerId = ""
window.onload = function () {
    displayOperationLogs()
    //    为按钮添加点击事件
    const button = document.getElementById('btn');
    button.addEventListener('click', function () {
        //    获取输入框内容
        const input = document.querySelector('input');
        const inputValue = input.value;
        //清空输入框
        input.value = '';
        let loading = startLoading("AI智能分析ing")
        fetchChatflow({}, inputValue, "admin", "Bearer app-GjovNjKKvnPAOdzufvRGPEPr").then(res => {
            stopLoading(loading)
            conversationId = res.id;
            let result = json2Object(res.answer)
            saveOperationLog(inputValue, result);
            loadResult(result)


        })
    })
}
```