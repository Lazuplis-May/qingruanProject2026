
任务详情

任务3-4：使用DeepSeek实现医师在线咨询功能

建议工时：

1

任务描述

**1. 任务描述**

本任务通过DeepSeek&Cline实现医师在线咨询功能。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、LocalStorage；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2F1c142a28576a4d64b14af2512728afb4.png)

图1 在线咨询



任务指导

#### 1.医师在线咨询功能

首先分析医师在线咨询功能实现的思路，首先用户发送消息，消息出现在聊天列表中，并调用聊天助手API发送消息，并接收消息显示在界面中，需要考虑消息显示时是流式显示还是非流式显示，在一般在线咨询功能中为非流式显示，即医师发送一条消息时，即把所有的消息发送完，除此之外还需要考虑如何保持聊天上下文，在上一任务中讲到通过在请求时携带“conversation_id”参数即可保证聊天的连贯性。

分析完成后即可得出，在在线咨询界面基础上如果想要实现在线聊天功能，则需要以下功能点：

1）实现聊天助手调用函数；

2）在chat.html页面实现发送和接收消息的函数，在发送消息时调用聊天助手接口获取回答，并调用接收消息的函数展示消息，并定义conversation_id参数用于管理聊天上下文；

参考提示词如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Fd7a6da74bd024dee97dfcbaf7690ac88.png)

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F855a070da358493eadbe8631c19ad29b.png)

图1 Cline提示词

通过Cline生成代码，并修改代码，注意在输入参数时，由于还没有实现登录注册等相关内容，所有在输入用户信息时可以使用模拟数据。需要注意的是，Dify输出的内容是带空格和换行效果的，但是如果通过HTML直接展示则不会展示换行和空格，需要为聊天框设置“white-space:pre-line”样式使其正常显示。效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Ff9c904cd7876483c9b2c56ecd8842410.png)

图2 在线咨询

通过测试发现医生回答问题较慢，可以在医生回答问题时添加“正在输入中”提示来优化用户体验。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F351c1db34a0041e68975202489cfe4de.png)

图3 Cline提示词

通过Cline生成代码后，检查并修改代码，最终效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F4908eff9612a45028313a89ff6c975ca.png)

图4 聊天测试

除此之外，当没有聊天记录时，医生会添加打招呼内容，提示词参考如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250425%2Feaff45c74d264cd3a43fe629c291f1ec.png)

图5 提示词

最终效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fd8ef7abcb6b74d85ae23dd70d44a0781.png)

图6 医师介绍

#### 2.实现聊天记录保存与加载功能

当刷新在线咨询页面时，聊天记录则会消失，可以实现将聊天信息存储到本地中，当界面加载时，读取聊天信息并显示到页面中。编写以下提示词实现此功能。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F9635d4f644f242c4ac9aec86d4f3854a.png)

图6 Cline提示词

通过Cline生成代码后，检查并修改代码。当刷新页面后，聊天记录还存在。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fa9577db53cd543c392715e0e90242fc9.png)

图8 聊天记录展示

除此之外，还需要实现，点击右上角的清理按钮，清空聊天记录与聊天数据，参考提示词如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F3c4789a584f947d4b9e792d76eb96bbd.png)

图9 Cline提示词

修改并优化代码后，效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F507b9db265cb467a8cd59ed178490082.png)

图10 清空聊天记录

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，所以本任务代码实现仅供参考。

效果图如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fdc7586e98cfa4116ae3bf4ef12b7e1f7.png)

图1 最终效果

1.修改“/js/api.js”中的代码，实现聊天接口调用功能，代码如下。

```
const BASE_API = "http://192.168.59.246";
const WORKFLOWS_API_PATH = "/v1/workflows/run";
const CHAT_API_PATH = "/v1/chat-messages"; // Added missing constant

const SQL_AUTH_TOKEN = "Bearer app-ywDAQwqYjDjAIzEY56DMbK9F";
async function fetchWorkflowData(inputs, userId, AUTH_TOKEN) {
    try {
        const headers = new Headers({
            "Authorization": AUTH_TOKEN,
            "Content-Type": "application/json"
        });

        const response = await fetch(`${BASE_API}${WORKFLOWS_API_PATH}`, {
            method: 'POST',
            headers: headers,
            body: JSON.stringify({
                inputs: inputs,
                response_mode: "blocking",
                user: userId
            }),
            redirect: 'follow'
        });

        // 检查响应状态
        if (!response.ok) {
            throw new Error(`请求失败，状态码：${response.status}`);
        }

        const result = await response.text();
        const responseData = JSON.parse(result);
        try {
            // 添加是否是string的判断
            if (typeof responseData.data.outputs.body === 'string') {
                return JSON.parse(responseData.data.outputs.body);
            }
            return responseData.data.outputs.body;
        } catch (e) {
            throw new Error('body字段解析失败，内容不是有效的JSON');
        }

    } catch (error) {
        console.error('请求处理失败:', error);
        throw new Error(`数据处理失败: ${error.message}`);
    }
}


// Text2sql工作流函数
async function fetchSQLWorkflow(intention, userId) {
    return await fetchWorkflowData({ intention: intention }, userId, SQL_AUTH_TOKEN);
}

// 智能体的调用，包括输入参数，输入信息，用户id、聊天智能体的token值
async function fetchChatflow(inputs, message, userId, CHAT_TOKEN, conversationId = "") {
    try {
        const headers = new Headers({
            "Authorization": CHAT_TOKEN,
            "Content-Type": "application/json"
        });

        const response = await fetch(`${BASE_API}${CHAT_API_PATH}`, {
            method: 'POST',
            headers: headers,
            body: JSON.stringify({
                inputs: inputs,
                query: message,
                conversation_id: conversationId,
                response_mode: "streaming",
                user: userId
            })
        });
        // 检查响应状态
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        // 获取响应体的可读流
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let conversation_id = null
        let resultStr = ''
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            
            const chunk = decoder.decode(value, { stream: true }).replace("event: ping","");
            //以data:作为分割，分割出多个字符串
            const chunks = chunk.split('data: ');
            chunks.forEach((ck) => {
                   //判断是否可以转为JSON对象
                try {
                    const data = JSON.parse(ck);
                    if(data.answer){
                        resultStr += data.answer;
                    }
                    if (conversation_id==null&&data.conversation_id) {
                        conversation_id = data.conversation_id;
                    }

                }catch (error) {
                }
            })
        }
        // 返回处理后的结果
        return {
            id: conversation_id,
            answer: resultStr
        };

    } catch (error) {
        console.error('请求处理失败:', error);
        throw new Error(`数据处理失败: ${error.message}`);
    }
}
```

2.修改/chat/chat.html代码，最终代码如下。

```
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>医师咨询</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <script src="js/chat.js"></script>
    <script src="/js/user.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="css/chat.css">

</head>

<body class="bg-gray-50 font-sans">
    <div class="flex flex-col h-screen">
        <!-- 顶部导航栏 -->
        <header class="fixed top-0 left-0 right-0 z-10 shadow-sm bg-white">
            <div class="flex items-center justify-between px-4 py-3">
                <button onclick="window.history.back()">
                    <i class="fas fa-chevron-left text-lg"></i>
                </button>
                <h1 class="text-lg font-medium">医师咨询</h1>
                <button id="clearChatBtn">
                    <i class="fa-solid fa-trash text-gray-600 text-lg"></i>
                </button>
            </div>
        </header>

        <!-- 医师信息区域 -->
        <div class="pt-16 pb-4 bg-white shadow-md rounded-b-lg mx-4">
            <div class="flex items-center px-4">
                <div class="relative">
                    <img src="/img/doc1.jpg" class="doctor-avatar object-cover" alt="医师头像">
                    <div class="absolute bottom-0 right-0 bg-white p-0.5 rounded-full">
                        <div class="status-dot bg-green-500"></div>
                    </div>
                </div>
                <div class="ml-3">
                    <div class="flex items-center">
                        <h2 class="font-bold text-gray-900">张明远 医师</h2>
                        <span class="ml-2 text-xs text-gray-500">在线</span>
                    </div>
                    <p class="text-sm text-gray-500">心血管内科主任医师 · 15年经验</p>
                </div>
            </div>
        </div>

        <!-- 聊天内容区域 -->
        <div id="messages-container" class="flex-1 overflow-y-auto px-4 py-2 space-y-4">

        </div>

        <!-- 底部输入区域 -->
        <div class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 px-4 py-2" style="height: 65px;">
            <div class="flex items-center input-container">
                <div class="flex-1 bg-gray-100 rounded-full px-4 py-2">
                    <input type="text" placeholder="输入消息..." class="w-full bg-transparent outline-none text-sm"
                        id="messageInput">
                </div>
                <button class="ml-2 bg-primary text-white rounded-full send-btn hidden" id="sendBtn">
                    <i class="fas fa-paper-plane text-white text-sm"></i>
                </button>
            </div>
        </div>
    </div>
</body>

</html>
```

修改chat/js/chat.js代码，最终效果如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#304FFF',
                secondary: '#10B981'
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
let doc = null
let userId = 1
let conversationId = null;
let inputs = {
    "userId": "123",
    "sex": "male",
    "age": 30,
    "height": 175,
    "weight": 70,
    "familyHistory": "none",
    "waistline": 80,
    "systolicPressure": 120,
    "isPregnancy": "no",
    "disease": "no"
}
let historyMessage = [];
// 从localStorage加载医生信息
function getDoctorFromStorage() {
    const stored = localStorage.getItem('currentDoctor');
    return stored ? JSON.parse(stored) : null;
}

// 更新医生信息显示
function updateDoctorInfo(doctor) {
    if (!doctor) return;
    console.log(doctor)
    document.querySelector('.doctor-avatar').alt = `${doctor.doctor_name} 医师头像`;
    document.querySelector('.doctor-avatar').src = doctor.image_url;
    document.querySelector('h2').textContent = `${doctor.doctor_name} 医师`;
    document.querySelector('p').textContent = `${doctor.department} ${doctor.title}`;

    // 更新所有消息中的医生名称
    document.querySelectorAll('.text-gray-900').forEach(el => {
        if (el.textContent.includes('张明远')) {
            el.textContent = el.textContent.replace('张明远', `${doctor.doctor_name}`);
        }
    });
    // 遍历所有的img标签，检查alt属性是否包含医师头像
    document.querySelectorAll('img').forEach(img => {
        if (img.alt.includes('医师头像')) {
            // 如果alt包含医师头像，则更新src属性为医生的头像URL
            img.src = doctor.image_url;
        }
    });


}

// 保存聊天记录
function saveChatHistory() {
    if (!doc) return;
    const key = `chat_history_${doc.info_id}`;
    localStorage.setItem(key, JSON.stringify(historyMessage));
    localStorage.setItem("conversationId",conversationId)
}
function timestampToDate(timestamp) {
    const date = new Date(timestamp);
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    return `${hours}:${minutes}`;
}
// 加载并显示历史消息
function loadChatHistory() {
    const messagesContainer = document.getElementById('messages-container');
    historyMessage.forEach(msg => {
        const timeString = timestampToDate(msg.timestamp);
        const message = msg.text;
        if (msg.sender === 'user') {
            // 用户消息
            const messageHTML = `
            <div class="flex items-start justify-end">
                <div class="mr-2 text-right">
                    <div class="flex items-center justify-end">
                        <span class="text-xs text-gray-400">${timeString}</span>
                        <span class="ml-2 text-sm font-medium text-gray-900">我</span>
                    </div>
                    <div class="mt-1 bg-primary rounded-lg shadow-sm px-3 py-2 chat-bubble">
                        <p class="text-sm text-white">${message}</p>
                    </div>
                </div>
                <img src="/img/user.jpg" class="message-avatar object-cover" alt="我的头像">
            </div>`;
            messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
        } else {
            // 医生消息
            const messageHTML = `
            <div class="flex items-start">
                <img src="/img/doc1.jpg" class="message-avatar object-cover" alt="医师头像">
            <div class="ml-2" style="width:82%">
                    <div class="flex items-center">
                        <span class="text-sm font-medium text-gray-900">${doc ? doc.doctor_name : '张明远'} 医师</span>
                        <span class="ml-2 text-xs text-gray-400">${timeString}</span>
                    </div>
                <div class="mt-1 bg-white rounded-lg shadow-sm px-3 py-2 msg text-sm ">${message}</div>
                </div>
            </div>`;
            messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
        }
    });
}

// 页面加载时初始化
window.onload = async function () {
    const messageInput = document.getElementById('messageInput');
    const sendBtn = document.getElementById('sendBtn');
    // 优先从URL获取医生信息，没有则从localStorage加载
    doc = getDoctorFromStorage();

    // 加载用户风险数据
    try {
        const userRiskInfo = await getUserRiskInfo();
        if (userRiskInfo) {
            console.log(userRiskInfo)
            inputs = {
                "userId": userRiskInfo.userId.toString() || "123",
                "sex": userRiskInfo.sex || "male",
                "age": userRiskInfo.age || 30,
                "height": userRiskInfo.height || 175,
                "weight": userRiskInfo.weight || 70,
                "familyHistory": userRiskInfo.familyHistory || "none",
                "waistline": userRiskInfo.waistline || 80,
                "systolicPressure": userRiskInfo.systolicPressure || 120,
                "isPregnancy": userRiskInfo.isPregnancy || "no",
                "disease": userRiskInfo.disease || "no"
            };
        }
    } catch (error) {
        console.error('加载用户风险数据失败:', error);
        // 使用默认输入值继续
    }
    updateDoctorInfo(doc);
    if (doc) {
        const key = `chat_history_${doc.info_id}`;
        const history = localStorage.getItem(key);
        historyMessage = history ? JSON.parse(history) : [];
        if (historyMessage.length === 0) {
            loadStartMessage()
        } else {
            loadChatHistory()
        }
        conid = localStorage.getItem("conversationId")
        if(conid)conversationId=conid
    }
    messageInput.addEventListener('input', function () {
        if (this.value.trim() !== '') {
            sendBtn.classList.remove('hidden');
        } else {
            sendBtn.classList.add('hidden');
        }
    });
    // 清空聊天记录
    document.getElementById('clearChatBtn').addEventListener('click', function () {
        if (doc) {
            Swal.fire({
                title: '确定要清空聊天记录吗？',
                text: "此操作无法撤销！",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#304FFF',
                cancelButtonColor: '#d33',
                confirmButtonText: '确定',
                cancelButtonText: '取消'
            }).then((result) => {
                if (result.isConfirmed) {
                    // 清空聊天界面
                    document.getElementById('messages-container').innerHTML = '';
                    // 清空本地存储的聊天记录
                    const key = `chat_history_${doc.info_id}`;
                    historyMessage = []
                    conversationId = null
                    localStorage.setItem("conversationId",null)
                    localStorage.setItem(key, JSON.stringify(historyMessage));

                    Swal.fire(
                        '已清空！',
                        '聊天记录已被清空。',
                        'success'
                    );
                    loadStartMessage()
                }
            });
        }
    });

    sendBtn.addEventListener('click', function () {
        const message = messageInput.value.trim();
        if (message !== '') {
            sendMessage(message);
            messageInput.value = '';
            sendBtn.classList.add('hidden');
        }
    });
    messageInput.addEventListener('keypress', function (e) {
        if (e.key === 'Enter' && this.value.trim() !== '') {
            const message = messageInput.value.trim();
            sendMessage(message);
            messageInput.value = '';
            sendBtn.classList.add('hidden');
        }
    });
}





// 添加消息到聊天界面并保存
function addMessageToChat(message, sender) {
    // 保存到历史记录
    if (doc) {
        historyMessage.push({
            text: message,
            sender: sender,
            timestamp: new Date().toISOString()
        });
        saveChatHistory();
    }
    const messagesContainer = document.getElementById('messages-container');
    const now = new Date();
    const timeString = `${now.getHours()}:${now.getMinutes().toString().padStart(2, '0')}`;

    if (sender === 'user') {
        // 用户消息
        const messageHTML = `
        <div class="flex items-start justify-end">
            <div class="mr-2 text-right">
                <div class="flex items-center justify-end">
                    <span class="text-xs text-gray-400">${timeString}</span>
                    <span class="ml-2 text-sm font-medium text-gray-900">我</span>
                </div>
                <div class="mt-1 bg-primary rounded-lg shadow-sm px-3 py-2 chat-bubble">
                    <p class="text-sm text-white">${message}</p>
                </div>
            </div>
            <img src="/img/user.jpg" class="message-avatar object-cover" alt="我的头像">
        </div>`;
        messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
    } else {
        // 医生消息
        const messageHTML = `
        <div class="flex items-start">
            <img src="${doc ? doc.image_url :'/img/doc1.jpg'}" class="message-avatar object-cover" alt="医师头像">
            <div class="ml-2" style="width:82%">
                <div class="flex items-center">
                    <span class="text-sm font-medium text-gray-900">${doc ? doc.doctor_name : '张明远'} 医师</span>
                    <span class="ml-2 mr-2 text-xs text-gray-400">${timeString}</span>
                </div>
                <div class="mt-1 bg-white rounded-lg shadow-sm px-3 py-2 msg text-sm text-grey-800 text-sm">${message}</div>
            </div>
        </div>`;
        messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
    }

    // 滚动到底部
    messagesContainer.scrollTop = messagesContainer.scrollHeight;
}
function loadStartMessage() {
    if (doc) {
        let startMessage = `你好，我是${doc.doctor_name},${doc.introduction}`
        addMessageToChat(startMessage, 'doctor')
    }
}
// 发送消息函数
async function sendMessage(message) {
    if (!message.trim()) return;

    // 添加用户消息到聊天界面
    addMessageToChat(message, 'user');
    showLoading()
    try {
        // 调用API获取医生回复
        const response = await fetchChatflow(
            inputs, // 空输入参数
            message,
            userId,
            doc.chat_token,
            conversationId
        );
        hideLoading()
        // 更新会话ID
        if (response.id) {
            conversationId = response.id;
        }

        // 添加医生回复到聊天界面
        if (response.answer) {
            console.log(response.answer)
            let answer = response.answer.trim();
            addMessageToChat(response.answer, 'doctor');
        }
    } catch (error) {
        console.error(error)
        addMessageToChat('抱歉，获取医生回复时出错，请稍后再试', 'doctor');
    }
}
/**
 * 在聊天列表中显示医生正在输入的提示信息。
 */
function showLoading() {
    // 构建加载提示的 HTML 字符串
    const loadingMessage = `
                <div class="text-center text-xs text-gray-400 my-4">
                    医生正在输入...
                </div>
            `;
    // 将加载提示添加到聊天列表的末尾
    document.getElementById('messages-container').insertAdjacentHTML('beforeend', loadingMessage);
}

/**
 * 隐藏聊天列表中的医生正在输入的提示信息。
 */
function hideLoading() {
    // 获取加载提示的 DOM 元素
    const loadingElement = document.querySelector('.text-center')
    // 如果存在加载提示元素，则移除它
    if (loadingElement) {
        loadingElement.remove();
    }
}
```