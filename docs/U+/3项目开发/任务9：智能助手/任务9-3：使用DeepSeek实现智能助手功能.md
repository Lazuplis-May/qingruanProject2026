任务详情

任务9-3：使用DeepSeek实现智能助手功能

建议工时：

1

任务描述

**1. 任务描述**

本任务实现智能助手聊天功能，由于智能助手聊天与医师咨询功能比较相似，所以可以借鉴前面部分医生咨询功能的内容，但是现在大模型普遍都是通过流式输出的形式，当发送消息后，模型会立即返回回答，但是返回回答内容一次性返回为一个字或者几个字，这样可以做到及时响应和持续反馈，提高用户使用体验。所以在本任务中除需实现聊天功能外，还需实现流式显示功能。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、HTTP流式传输处理；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2F3dc1121665dc497ba3d15b5d221046ba.png)

图1 AI助手聊天



任务指导

##### 1. 智能助手聊天功能解析

在Dify中，智能体支持流式传输，即发送一个HTTP请求后，会返回多个响应数据流，每一个数据流都包含一个或多个字。在之前实现的医师咨询任务中，将流式聊天信息转为一个完整答案并显示，而在本任务中，就将返回的数据进行立即显示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F3fe6b7e08af940dc9fed4bd67662ebc1.png)

图1 AI助手流式输出内容

其具体的实现思路可询问DeepSeek。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F5b29e9162e3840c5af91d459daf922a8.png)

图2 通过Deepseek获取思路

#### 2. 智能助手聊天功能实现

询问DeepSeek，并整理出实现思路后，自行编写提示词，并通过DeepSeek&Cline来实现此功能，在此不做过多赘述。最终实现效果为当询问智能助手时，助手会立即回答，并以流式的方式进行数据展示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fce90e703331c496282cd2b182c49fcd1.png)

图3 AI助手聊天

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，固任务实现仅供参考。

1.在js/api.js文件内添加智能体调用代码，参考如下。

```
const AI_CHAT_TOKEN = "Bearer xxxxxx"

async function fetchAIChatflow(inputs, message, userId, conversationId = "",  callFunc) {
    try {
        const headers = new Headers({
            "Authorization": AI_CHAT_TOKEN,
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
                    if (conversation_id==null&&data.conversation_id) {
                        conversation_id = data.conversation_id;
                    }
                    if(data.answer){
                        callFunc(data.answer, conversation_id);
                    }


                }catch (error) {
                }
            })
        }
        callFunc('done', conversation_id);
        // 返回处理后的结果

    } catch (error) {
        console.error('请求处理失败:', error);
        throw new Error(`数据处理失败: ${error.message}`);
    }
}
```

2.修改ai/js/userAI.js代码，实现聊天功能，最终代码如下：

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

let userinfo = null
let userId = 0
let conversationId = null;
let  messageInput = null;
let sendBtn = null;
let chatHistory = [];

window.onload = async function () {
    messageInput = document.getElementById('messageInput');
    sendBtn = document.getElementById('sendBtn');
    // 监听输入框的输入事件
    messageInput.addEventListener('input', function () {
        if (this.value.trim() !== '') {
            sendBtn.classList.remove('hidden');
        } else {
            sendBtn.classList.add('hidden');
        }
    });
    // 清空聊天记录
    document.getElementById('clearChatBtn').addEventListener('click', function () {
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
                const key = `aichat_history_${userId}`;
                localStorage.removeItem(key);
                chatHistory = [];
                Swal.fire(
                    '已清空！',
                    '聊天记录已被清空。',
                    'success'
                );
            }
        });
    });
    // 发送消息
    sendBtn.addEventListener('click', function () {
        const message = messageInput.value.trim();
        if (message !== '') {
            sendMessage(message);
            messageInput.value = '';
            sendBtn.classList.add('hidden');
        }
    });
    // 监听输入框的按键事件
    messageInput.addEventListener('keypress', function (e) {
        if (e.key === 'Enter' && this.value.trim() !== '') {
            const message = messageInput.value.trim();
            sendMessage(message);
            messageInput.value = '';
            sendBtn.classList.add('hidden');
        }
    });
    userinfo = await getUserRiskInfo()
    userId = userinfo.userId
    const key = `aichat_history_${userId}`;
    const localHistory = localStorage.getItem(key);
    chatHistory = localHistory ? JSON.parse(localHistory) : [];
    chatHistory.forEach(msg => {
        loadMessageToChat(msg);
    });
}
// 保存聊天记录
function saveChatHistory() {
    const key = `aichat_history_${userId}`;
    localStorage.setItem(key, JSON.stringify(chatHistory));
}
function timestampToDate(timestamp) {
    const date = new Date(timestamp);
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    return `${hours}:${minutes}`;
}
function loadMessageToChat(message) {
    const text = message.text;
    const sender = message.sender;
    const messagesContainer = document.getElementById('messages-container');
    const now = new Date();
    const timeString = timestampToDate(message.timestamp);

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
                            <p class="text-sm text-white text-left">${text}</p>
                        </div>
                    </div>
                    <img src="/img/user.jpg" class="message-avatar object-cover" alt="我的头像">
                </div>`;
        messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
    } else {
        // AI消息
        const messageHTML = `
                <div class="flex items-start">
                    <img src="/img/aichat_logo.png" class="message-avatar object-cover" >
                    <div class="ml-2" style="width:82%">
                        <div class="flex items-center">
                            <span class="text-sm font-medium text-gray-900">AI助手</span>
                            <span class="ml-2 text-xs text-gray-400">${timeString}</span>
                        </div>
                        <div class="mt-1 bg-white rounded-lg shadow-sm px-3 py-2 text-sm  msg">${text}</div>
                    </div>
                </div>`;
        messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
    }

    // 滚动到底部
    messagesContainer.scrollTop = messagesContainer.scrollHeight;
}
// 添加消息到聊天界面并保存
function addMessageToChat(message) {
    chatHistory.push({
        text: message,
        sender: 'user',
        timestamp: new Date().toISOString()
    });
    saveChatHistory();
    const messagesContainer = document.getElementById('messages-container');
    const now = new Date();
    const timeString = `${now.getHours()}:${now.getMinutes().toString().padStart(2, '0')}`;
    // 用户消息
    const messageHTML = `
                <div class="flex items-start justify-end">
                    <div class="mr-2 text-right">
                        <div class="flex items-center justify-end">
                            <span class="text-xs text-gray-400">${timeString}</span>
                            <span class="ml-2 text-sm font-medium text-gray-900">我</span>
                        </div>
                        <div class="mt-1 bg-primary rounded-lg shadow-sm px-3 py-2 chat-bubble">
                            <p class="text-sm text-white text-left">${message}</p>
                        </div>
                    </div>
                    <img src="${userinfo.avatar_url || "/img/user.jpg"}" class="message-avatar object-cover" alt="我的头像">
                </div>`;
    messagesContainer.insertAdjacentHTML('beforeend', messageHTML);

    // 滚动到底部
    messagesContainer.scrollTop = messagesContainer.scrollHeight;
}
// 发送消息函数
async function sendMessage(message) {
    if (!message.trim()) return;

    // 添加用户消息到聊天界面
    addMessageToChat(message, 'user');
    const now = new Date();
    const timeString = `${now.getHours()}:${now.getMinutes().toString().padStart(2, '0')}`;
    const messagesContainer = document.getElementById('messages-container');
    //加载一个随机数作为id
    const randomId = 'chat_' + Math.floor(Math.random() * 10000);
    // AI消息
    const messageHTML = `
               <div class="flex items-start">
                   <img src="/img/aichat_logo.png" class="message-avatar object-cover">
                    <div class="ml-2" style="width:82%">
                       <div class="flex items-center">
                           <span class="text-sm font-medium text-gray-900">AI助手</span>
                           <span class="ml-2 text-xs text-gray-400">${timeString}</span>
                       </div>
                        <div class="mt-1 bg-white rounded-lg shadow-sm px-3 py-2 msg" text-sm  id="${randomId}"></div>
                   </div>
               </div>`;
    messagesContainer.insertAdjacentHTML('beforeend', messageHTML);
    const messageElement = document.getElementById(randomId)
    fetchAIChatflow(userinfo, message, userId, conversationId, (ans, con_id) => {
        conversationId = con_id
        if (ans && ans != 'done') {
            messageElement.textContent += ans
            // 滚动到底部
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
            // 保存聊天记录
        } else {
            chatHistory.push({
                text: messageElement.textContent,
                sender: 'ai',
                timestamp: new Date().toISOString()
            });
            saveChatHistory();
        }
    })
}
```