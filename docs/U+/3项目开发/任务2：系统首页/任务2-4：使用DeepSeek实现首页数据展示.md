任务详情

任务2-4：使用DeepSeek实现首页数据展示

建议工时：

2

任务描述

**1. 任务描述**

本任务通过DeepSeek&Cline实现在首页中调用数据管理工作流，实现首页面数据展示。并实现在首页点击健康科普和糖尿病类型时，跳转到对应的页面并展示详情数据。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染、SessionStorage；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c6493c93996a4084b6395e130741e277%2Frichtext%2Fimage%2F20250430%2Fd18ede8781e44668ad6de5eecaa8b57f.png)

图1 首页数据展示



任务指导

#### 1.通过数据管理工作流实现首页面数据加载功能

在页面初始化时，通过调用数据库工作流获取首页的医师数据、文章数据和糖尿病类型数据，首先创建数据库工作流函数，然后再在main.html页面中调用该函数，并实现界面数据渲染功能。

1）实现工作流调用函数

通过DeepSeek&Cline实现接口调用函数，编写以下提示词完成此功能。

```
在js目录中创建api.js，实现工作流接口的调用，接口参考curl -X POST 'http://192.168.59.246/v1/workflows/run' \
--header 'Authorization: Bearer {api_key}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {},
    "response_mode": "blocking",
    "user": "abc-123"
}'，还需要考虑到响应JSON字符串转码问题。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F3b37f4cb42cd40cb8845669e63c9610a.png)

图1 Cline提示词

手动修改IP和key，并完善此代码，需要注意的是，如果是通过云沙箱环境运行dify，则需要将IP修改为云沙箱80端口映射地址。

2）在主页中调用工作流渲染

通过DeepSeek&Cline实现接口调用函数，编写以下提示词完成此功能。

```
@/main\main.html 在此页面中调用数据管理工作流，实现医生列表、文章和糖尿病类型的渲染，接口参考@/js/api.js 
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2F4b9714b72dbb420ba7dc3397b0a7877e.png)

图2 Cline提示词

运行项目，并根据错误修改代码，最终效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c6493c93996a4084b6395e130741e277%2Frichtext%2Fimage%2F20250430%2F05c13cfca25b4168a5bd14f93515feaa.png)

图3 首页数据展示

运行发现，每次首页数据加载时间都较长，所以可以通过缓存设计来实现加快多次调用的数据加载速度。

#### 2.首页数据缓存功能实现

通过DeepSeek&Cline实现接口调用函数，编写以下提示词完成缓存设计功能。

```
@/main\main.html 为此页面设计数据缓存，优先从SessionStorage中读取缓存，如果没有则通过API接口获取数据
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250415%2Fbcb55dae13f649eaa69aed4b1e76cb19.png)

图4 Cline代码生成

通过Cline完成代码编写、修改与调试，实现效果如下图所示。该页面仅在第一次加载时加载速度较慢，后续通过缓存技术，加载速度提高。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c6493c93996a4084b6395e130741e277%2Frichtext%2Fimage%2F20250430%2F62bf7f4e0da345f4b3038e225f01cb21.png)

图5 首页数据展示

#### 3.实现文章详情加载功能

当点击健康科普中的文章信息时，跳转至健康科普页面，为了区分用户点击的是哪一个文章，在点击跳转时在URL中还可以传入id信息。然后在文章详情界面加载时，根据id获取文章信息，并渲染到界面中。

首先实现点击文章跳转功能，在main.js中实现文章跳转功能，参考提示词如下：

```
@/main\js\main.js 在此文件中实现点击文章项跳转功能，跳转时携带文章的id信息。
```

当实现跳转功能后，接下来再实现数据加载功能，可参考以下提示词。

```
@/main/js/article.js 获取URL中传入的文章id，并通过工作流接口调用根据id获取文章信息，可参考@/main\js\main.js 
```

在以上代码中，通过让大模型参考之前编写完成的代码，可以提高大模型的准确率，实现完成以上功能后，还需考虑一些细节问题，例如在页面加载时添加加载特效以优化用户体验等，可自行编写提示词并实现该功能，最终效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c6493c93996a4084b6395e130741e277%2Frichtext%2Fimage%2F20250430%2F414ca84792814c2dbddc94617162517d.png)

图6 文章信息

 

#### 4.实现糖尿病类型加载功能

此功能的实现思路和文章加载功能基本一致，当在首页点击糖尿病类型时，携带类型id跳转至糖尿病类型界面中，在界面加载时获取类型id，并根据id加载类型信息，最后将数据渲染到界面中。参考提示词如下：

1）界面跳转提示词

```
@/main\js\main.js 在此文件中实现点击糖尿病类型跳转功能，跳转时携带糖尿病类型的id信息。
```

2）数据加载提示词

```
@/main/js/diabetes.js 获取URL中传入的类型id，并通过工作流接口调用根据id获取糖尿病类型，可参考@/main\js\main.js 
```

最终实现效果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/c6493c93996a4084b6395e130741e277%2Frichtext%2Fimage%2F20250430%2F49f4b318975743c8a894f65cf4c21403.png)

图7 糖尿病类型

 

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，所以本任务代码实现仅供参考。

1.在js目录中新建api.js文件，代码如下，需要注意的是每个人服务器的IP地址和Key均不同，请修改为自己服务对应的IP地址和Key，如果使用云沙箱环境，则API地址修改为80端口的映射地址。

```
const BASE_API = "http://192.168.59.246/";
const WORKFLOWS_API_PATH = "/v1/workflows/run";

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
```

2.main/main.html代码如下：

```
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>主内容页面</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="js/main.js"></script>
    <link rel="stylesheet" href="/css/swiper-bundle.min.css" />
    <script src="/js/swiper-bundle.min.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="css/main.css">
</head>

<body>
    <!-- 顶部导航栏 -->
    <div class="nav-bar bg-white shadow-sm">
        <div class="flex items-center justify-between px-4 py-3">
            <img src="/img/logo_main.png" width="120">
            <div class="relative w-8">
                <i class="fa-icon absolute right-0 top-1/2 transform -translate-y-1/2 text-gray-400">
                    <i class="fas fa-search text-xl"></i>
                </i>
            </div>
        </div>
    </div>

    <!-- 完整内容区域 -->
    <div class="content-area">
        <!-- 轮播图 -->
        <div class="swiper swiper-initialized swiper-horizontal swiper-ios swiper-backface-hidden">
            <div class="swiper-wrapper" id="bannerDiv" aria-live="off"
                style="transition-duration: 0ms; transform: translate3d(-780px, 0px, 0px); transition-delay: 0ms;">
                <div class="swiper-slide swiper-slide-next" role="group" aria-label="NaN / 3" style="width: 390px;">
                    <img src="/img/lb1.png" class="w-full h-full object-cover" alt="医疗科技">
                </div>
                <div class="swiper-slide swiper-slide-prev" role="group" aria-label="NaN / 3" style="width: 390px;">
                    <img src="/img/lb2.png" class="w-full h-full object-cover" alt="医疗科技">
                </div>
                <div class="swiper-slide swiper-slide-active" role="group" aria-label="NaN / 3" style="width: 390px;">
                    <img src="/img/lb3.jpeg" class="w-full h-full object-cover" alt="医疗科技">
                </div>
            </div>
            <!-- 分页器 -->
            <div
                class="swiper-pagination swiper-pagination-clickable swiper-pagination-bullets swiper-pagination-horizontal">
                <span class="swiper-pagination-bullet" tabindex="0" role="button"
                    aria-label="Go to slide 1"></span><span class="swiper-pagination-bullet" tabindex="0" role="button"
                    aria-label="Go to slide 2"></span><span
                    class="swiper-pagination-bullet swiper-pagination-bullet-active" tabindex="0" role="button"
                    aria-label="Go to slide 3" aria-current="true"></span></div>
            <span class="swiper-notification" aria-live="assertive" aria-atomic="true"></span>
        </div>
        <!-- 专业医师团队 -->
        <div class="px-4 py-3">
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-lg font-semibold text-gray-800">专业医师团队</h2>
                <a href="#" class="text-sm text-primary">查看全部 <i class="fas fa-chevron-right ml-1"></i></a>
            </div>
            <div id="doctors-container" class="flex space-x-3 overflow-x-auto pb-2 -mx-4 px-4">
                <!-- Doctors will be loaded here dynamically -->
                <div class="text-center py-10 w-full text-gray-500">加载医生数据中...</div>
            </div>
        </div>

        <!-- 健康科普栏 -->
        <div class="px-4 py-3">
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-lg font-semibold text-gray-800">健康科普</h2>
                <a href="#" class="text-sm text-primary">更多 <i class="fas fa-chevron-right ml-1"></i></a>
            </div>
            <div id="articles-container" class="bg-white rounded-lg overflow-hidden">
                <!-- Articles will be loaded here dynamically -->
                <div class="text-center py-10 text-gray-500">加载文章数据中...</div>
            </div>
        </div>

        <!-- 糖尿病类型栏 -->
        <div class="px-4 py-3">
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-lg font-semibold text-gray-800">糖尿病类型</h2>
                <a href="#" class="text-sm text-primary">全部 <i class="fas fa-chevron-right ml-1"></i></a>
            </div>
            <div id="types-container" class="grid grid-cols-2 gap-3">
                <!-- Diabetes types will be loaded here dynamically -->
                <div class="text-center py-10 col-span-2 text-gray-500">加载糖尿病类型数据中...</div>
            </div>
        </div>
    </div>

</body>

</html>
```

在main/js/main.js文件，其代码如下：

```
let docList = [];
// Helper functions for SessionStorage
function getCachedData(key) {
    const cached = sessionStorage.getItem(key);
    if (!cached) return null;

    const { data, timestamp } = JSON.parse(cached);
    // Cache expires after 1 hour (3600000 ms)
    if (Date.now() - timestamp > 3600000) {
        sessionStorage.removeItem(key);
        return null;
    }
    return data;
}

function setCachedData(key, data) {
    const cache = {
        data,
        timestamp: Date.now()
    };
    sessionStorage.setItem(key, JSON.stringify(cache));
}

// Load doctors data
async function loadDoctors() {
    try {
        const cacheKey = 'doctors_data';
        let data = getCachedData(cacheKey);
        
        if (!data) {
            data = await fetchSQLWorkflow('获取医生列表', 'current_user');
            setCachedData(cacheKey, data);
        }
        const container = document.getElementById('doctors-container');
        container.innerHTML = data.result.map(doctor => `
                    <div class="doctor-card flex-shrink-0 w-32 h-56 bg-white rounded-lg p-3 flex flex-col">
                        <div class="flex flex-col items-center space-y-3">
                            <div class="relative mb-3">
                                <img src="${doctor.image_url || '/img/doc1.png'}" 
                                     alt="${doctor.doctor_name}" class="w-20 h-20 rounded-full object-cover border-2 border-blue-500">
                                <div class="absolute bottom-0 left-0 right-0 bg-white text-primary text-xs text-center py-[1px] font-normal">
                                    ${doctor.title || '专科医师'}
                                </div>
                            </div>
                            <h3 class="text-sm font-semibold text-gray-800 text-center mb-2">${doctor.doctor_name}</h3>
                            <p class="text-xs text-gray-500 mb-3 text-center">${doctor.department || '内分泌科'}</p>
                            <button class="py-1 bg-white text-primary text-xs rounded-button btn consult-btn" 
                                    data-token="${doctor.chat_token}"
                                    data-doctor="${doctor.info_id}">
                                立即咨询
                            </button>
                        </div>
                    </div>
                `).join('');
        docList = data.result
    } catch (error) {
        console.error('加载医生数据失败:', error);
        document.getElementById('doctors-container').innerHTML =
            '<div class="text-center py-10 w-full text-red-500">加载医生数据失败，请稍后重试</div>';
    }
}

// Load articles data
async function loadArticles() {
    try {
        const cacheKey = 'articles_data';
        let data = getCachedData(cacheKey);

        if (!data) {
            data = await fetchSQLWorkflow('查询3篇文章信息', 'current_user');
            setCachedData(cacheKey, data);
        }
        const container = document.getElementById('articles-container');
        console.log('Fetched doctors:', data);

        container.innerHTML = data.result.map(article => `
                    <div class="article-item px-4 py-3" onclick = "top.location.href='article.html?id=${article.article_id}'">
                        <div class="flex">
                            <div class="w-20 h-20 rounded-lg overflow-hidden mr-3">
                                <img src="${article.cover_url || '/img/a1.jpg'}" 
                                     alt="${article.title}" class="w-full h-full object-cover">
                            </div>
                            <div class="flex-1">
                                <h3 class="text-sm font-semibold text-gray-800 mb-1">${article.title}</h3>
                                <p class="text-xs text-gray-500 mb-2 line-clamp-2">${article.content.substring(0, 20) + "..." || '暂无简介'}</p>
                                <div class="flex items-center text-xs text-gray-400">
                                    <i class="fas fa-eye mr-1"></i>
                                    <span>${article.views || '0'} 浏览</span>
                                </div>
                            </div>
                        </div>
                    </div>
                `).join('');
    } catch (error) {
        console.error('加载文章数据失败:', error);
        document.getElementById('articles-container').innerHTML =
            '<div class="text-center py-10 text-red-500">加载文章数据失败，请稍后重试</div>';
    }
}

// Load diabetes types data
async function loadTypes() {
    try {
        const cacheKey = 'diabetes_types_data';
        let data = getCachedData(cacheKey);

        if (!data) {
            data = await fetchSQLWorkflow('获取糖尿病类型', 'current_user');
            setCachedData(cacheKey, data);
        }
        const container = document.getElementById('types-container');

        container.innerHTML = data.result.map(type => `
                    <div class="category-item bg-white rounded-lg overflow-hidden" onclick="top.location.href='diabetes.html?id=${type.type_id}'">
                        <div class="h-24 overflow-hidden">
                            <img src="${type.img || '/img/t1.jpg'}" 
                                 alt="${type.type_name}" class="w-full h-full object-cover">
                        </div>
                        <div class="p-3">
                            <h3 class="text-sm font-semibold text-gray-800 mb-1">${type.type_name}</h3>
                            <p class="text-xs text-gray-500 line-clamp-2">${type.pathogenesis || '暂无描述'}</p>
                        </div>
                    </div>
                `).join('');
    } catch (error) {
        console.error('加载糖尿病类型数据失败:', error);
        document.getElementById('types-container').innerHTML =
            '<div class="text-center py-10 col-span-2 text-red-500">加载糖尿病类型数据失败，请稍后重试</div>';
    }
}

// Handle consultation button clicks
function setupConsultButtons() {
    document.addEventListener('click', (e) => {
        if (e.target.classList.contains('consult-btn')) {
            const id = e.target.getAttribute('data-doctor');
            console.log(id)
            docList.forEach((doc)=>{
                if(doc.info_id == id){
                    localStorage.setItem('currentDoctor', JSON.stringify(doc));
                } 
            })
            // 跳转到聊天页面并传递医生信息
            top.location.href = `../chat/chat.html`;
        }
    });
}
window.onload = function () {
    // 初始化轮播图
    const swiper = new Swiper('.swiper', {
        loop: true,
        autoplay: {
            delay: 3000,
        },
        pagination: {
            el: '.swiper-pagination',
            clickable: true,
        },
        navigation: {
            nextEl: '.swiper-button-next',
            prevEl: '.swiper-button-prev',
        },
    });
    loadDoctors();
    loadArticles();
    loadTypes();
    setupConsultButtons();
}
```

3.修改article.html代码，实现数据加载功能，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>糖尿病科普</title>
    <link href="/css/all.min.css" rel="stylesheet">
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="js/article.js"></script>
    <script src="/js/loading.js"></script>
    <script src="/js/showAlert.js"></script>
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="css/article.css">
</head>

<body>
    <header class="top-0 left-0 right-0 z-10 shadow-sm bg-white">
        <div class="flex items-center justify-between px-4 py-3">
            <button onclick="window.history.back()">
                <i class="fas fa-chevron-left text-lg"></i>
            </button>
            <h2 class="text-lg font-medium">糖尿病科普</h2>
            <button>
            </button>
        </div>
    </header>


    <main >
        <div class="mt-4 rounded-lg overflow-hidden">
            <img src=""
                class="w-full h-auto object-cover px-2">
        </div>

        <div class="mt-4 card shadow-md">
            <h1 class="text-xl font-bold text-gray-900"></h1>
            <div class="mt-3 flex items-center text-sm text-gray-500">
                <span class="ml-2"></span> 
                <span></span>
                <div class="ml-auto flex items-center">
                    <span class="time"></span>
                </div>
            </div>
            <article class="article-content mt-6 bb">
            </article>
            <div class="mt-8 flex flex-wrap items-center justify-end gap-2">
                <button class="">
                    <i class="fa-regular fa-thumbs-up"></i>
                    100
                </button>
                <button>
                    <i class="fa-regular fa-comment"></i>
                    20
                </button>
                <button>
                    <i class="fa-regular fa-eye"></i>
                    322
                </button>
            </div>
        </main>
        </div>
</body>

</html>
```

对应的main\js\article.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4A90E2',
                secondary: '#81B3F3'
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
    let loading = startLoading();
    try {
        // 获取url上的article_id
        const urlParams = new URLSearchParams(window.location.search);
        const articleId = urlParams.get('id');
        fetchSQLWorkflow(`查询id为${articleId}的文章信息`, "system")
            .then(res => {
                data = res.result
                let article = data[0]
                console.log(article)
                // Update dynamic content
                document.querySelector('img').src = article.cover_url;
                document.querySelector('img').alt = article.title;
                document.querySelector('h1.text-xl').textContent = article.title;
                document.querySelector('span.ml-2').textContent = article.author;
                document.querySelector('span.time').textContent = article.publish_time;
                document.querySelector('.article-content').innerHTML = article.content;
                stopLoading(loading);

            })
    } catch (error) {
        console.error('Error loading article:', error);
        stopLoading(loading);
        showFloatingAlert("加载文章失败，请稍后重试。", "error");
    }
}
```

4.修改diabetes.html中的代码，实现数据加载功能，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>糖尿病知识</title>
    <link href="/css/all.min.css" rel="stylesheet">
    <script src="/js/tailwindcss.js"></script>
    <script src="js/diabetes.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/loading.js"></script>
    <script src="/js/showAlert.js"></script>
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="css/diabetes.css">
</head>

<body>
    <header class="top-0 left-0 right-0 z-10 shadow-sm bg-white">
        <div class="flex items-center justify-between px-4 py-3">
            <button onclick="window.history.back()">
                <i class="fas fa-chevron-left text-lg"></i>
            </button>
            <h1 class="text-lg font-medium" id="type_name"></h1>
            <button>
            </button>
        </div>
    </header>

    <div style="width: 100vw;">
        <div class="img-container">
            <img id="img" src="/img/t1.jpg" class="flex w-full h-full object-cove " alt="糖尿病封面">
        </div>
        <div class="px-4 py-6">
            <section class="mb-8">
                <h2>发病机制</h2>
                <div id="pathogenesis" class="space-y-4">
                </div>
            </section>

            <section class="mb-8">
                <h2>临床表现</h2>
                <div id="manifestation" class="space-y-4">
                </div>
            </section>

            <section class="mb-8">
                <h2>治疗方法</h2>
                <div id="treatment" class="space-y-4">
                </div>
            </section>
        </div>
    </div>
</body>

</html>
```

对应的main\js\diabetes.js代码如下：

```
window.onload = function () {
    let loagding = startLoading();
    // 定义病症类型
    var res = null
    // 获取查询的病症类型id
    const searchParams = new URLSearchParams(window.location.search);
    const tpid = searchParams.get('id');
    fetchSQLWorkflow(`查询id为${tpid}的糖尿病类型信息`, "abc")
        .then(data => {
            stopLoading(loagding);
            const res = data.result[0];
            console.log('解析后的数据:', res);

            // DOM更新移到此处确保数据就绪
            if (res) {
                // 更新类型名称
                document.getElementById('type_name').innerText = res.type_name || '糖尿病知识';
                document.getElementById('img').src = res.img
                // 更新发病机制
                document.getElementById('pathogenesis').innerHTML = `
                <p class="text-gray-600 leading-6">${res.pathogenesis || '暂无发病机制说明'}</p>
            `;
                // 更新临床表现
                document.getElementById('manifestation').innerHTML = `
                <p class="text-gray-600 leading-6">${res.manifestation || '暂无临床表现说明'}</p>
            `;
                // 更新治疗方法
                document.getElementById('treatment').innerHTML = `
                <p class="text-gray-600 leading-6">${res.treatment || '暂无治疗方法说明'}</p>
            `;

            }
        })
        .catch(error => {
            stopLoading(loagding);
            console.error(error)
        });
}
```