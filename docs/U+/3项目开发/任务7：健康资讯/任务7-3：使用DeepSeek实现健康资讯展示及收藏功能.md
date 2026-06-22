任务详情

任务7-3：使用DeepSeek实现健康资讯展示及收藏功能

建议工时：

1

任务描述

**1. 任务描述**

本任务实现在健康资讯展示及收藏功能，主要包括健康资讯标签展示、健康资讯详情展示、健康资讯收藏、收藏查看等功能。

**2. 任务知识**

**知识点 ：**DeepSeek&Cline辅助编码、Fetch API、HTML界面渲染；

**重点 ：**DeepSeek&Cline辅助编码、Fetch API；

**难点 ：**DeepSeek&Cline辅助编码、Fetch API；

**3. 任务成果**

本任务成果如下：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2F8d6065659c744d9893eb46f8a997f084.png)

图1 健康资讯页面

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Ff1da049816034477adacbd2930383ece.png)

图2 健康资讯详情

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2F6fabb3225d694fe7bcfa0aa3bfadaad9.png)

图3 健康资讯列表



任务指导

##### 1. 健康资讯标签展示

在页面加载时，根据用户的信息调用健康资讯工作流，并将数据展示到界面中，需要注意以下几点：

1）工作流执行较慢，需要做缓存设计和加载特效；

2）为各个资讯项设置点击事件，当点击时携带参数跳转到健康资讯详情中；

3）用户进入此界面前可能没有填写个人信息，如果没有个人信息则提示用户填写。

实现以上功能不需要一次性完成，可以通过多次问答来实现，也可以通过先用Plan模式做好规划，然后再进行代码生成。

最终效果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2F8d6065659c744d9893eb46f8a997f084.png)

图1 健康资讯页面

##### 2. 健康资讯详情展示

当页面加载时，首先获取用户信息和健康资讯标签信息，调用健康资讯工作流生成资讯文章并渲染到界面中。当点击收藏时，调用数据管理工作流将文章保存到文章收藏表中，如下图所示。需要注意的点包括：

1）健康资讯生成加载速度较慢，需要添加加载特效；

2）如果用户未填写相关信息，则需要使用通用信息来生成文章；

3）点击“收藏”后，需要修改收藏按钮，变为“已收藏”的提示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Ff1da049816034477adacbd2930383ece.png)

图2 健康资讯详情

##### 3. 查看收藏健康资讯

当用户点击个人中心中的“我的资讯”时，跳转至健康资讯收藏列表。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fe865185dfd2744a1a366a1ef55e54e46.png)

图3 个人中心

在健康资讯列表加载时，调用数据访问工作流获取用户的健康资讯信息，然后加载到界面中。当点击资讯列则弹出资讯详情信息。当点击删除按钮时弹出删除提示，点击“确定”则调用数据访问工作流删除该资讯。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250428%2Fa5192d42ca98418981f521f243c11727.png)

图4 健康资讯列表

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的界面内容完全相同，固任务实现仅供参考。

1.在js/api.js中添加工作流调用函数，代码如下：

```
const LA_AUTH_TOKEN = "Bearer xxx";//填写自己生成的秘钥
// 健康资讯工作流函数
async function fetchLifeAdviceWorkflow(inputs, userId) {
    return await fetchWorkflowData(inputs, userId, LA_AUTH_TOKEN);
}
```

\2. 修改lifeadvice/lifeAdvice.html文件，实现健康资讯标签展示功能，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>糖尿病健康资讯</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/loading.js"></script>
    <script src="js/lifeAdvice.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="css/lifeAdvice.css">
</head>

<body class="w-full min-h-screen bg-gray-50">
    <!-- 顶部导航栏 -->
    <nav class="sticky top-0 w-full bg-white shadow-sm z-50 px-4 h-14 flex items-center justify-between safe-top">
        <button class="flex items-center !rounded-button">
        </button>
        <h1 class="text-lg font-bold">健康资讯</h1>
        <button class="flex items-center !rounded-button text-primary" onclick="adjustmentScheme()"
            style="color:#3261FF ;">
        </button>
    </nav>
    <main class="pt-4 pb-20 px-4">
        <div class="bg-white rounded-lg shadow-sm p-4 mb-4 card">
            <div class="flex items-center mb-3">
                <img src="/img/la1.png" class="w-[41px] h-[31px]  mr-3 ml-3" />
                <div class="flex-1">
                    <h3 class="text-lg font-medium mb-1">饮食指导</h3>
                    <p class="text-sm text-gray-600">合理搭配饮食，控制血糖水平</p>
                </div>
                <i class="fas fa-chevron-right text-gray-400"></i>
            </div>
            <div class="border-b border-dashed border-gray-300 mb-3"></div>
            <div id="eatTag" class="flex flex-wrap gap-2">
            </div>
        </div>
        <div class="bg-white rounded-lg shadow-sm p-4 mb-4 card">
            <div class="flex items-center mb-3">
                <img src="/img/la2.png" class="w-[41px] h-[31px]  mr-3 ml-3" />
                <div class="flex-1">
                    <h3 class="text-lg font-medium mb-1">运动指南</h3>
                    <p class="text-sm text-gray-600">适度运动，增强体质</p>
                </div>
                <i class="fas fa-chevron-right text-gray-400"></i>
            </div>
            <div class="border-b border-dashed border-gray-300 mb-3"></div>
            <div id="sportTag" class="flex flex-wrap gap-2">
            </div>
        </div>
        <div class="bg-white rounded-lg shadow-sm p-4 mb-4 card">
            <div class="flex items-center mb-3">
                <img src="/img/la3.png" class="w-[41px] h-[31px]  mr-3 ml-3" />
                <div class="flex-1">
                    <h3 class="text-lg font-medium mb-1">日常习惯</h3>
                    <p class="text-sm text-gray-600">养成健康生活方式</p>
                </div>
                <i class="fas fa-chevron-right text-gray-400"></i>
            </div>
            <div id="dailyCard" class="grid grid-cols-2 gap-3">
            </div>
        </div>
        <div class="bg-white rounded-lg shadow-sm p-4">
            <div class="flex items-center mb-3">
                <img src="/img/la4.png" class="w-[41px] h-[31px]  mr-3 ml-3" />
                <div class="flex-1">
                    <h3 class="text-lg font-medium mb-1">糖尿病科普</h3>
                    <p class="text-sm text-gray-600">了解糖尿病相关知识</p>
                </div>
                <i class="fas fa-chevron-right text-gray-400"></i>
            </div>
            <div id="popularizationCard" class="grid grid-cols-2 gap-3">
            </div>
        </div>
    </main>
    <script>

    </script>
</body>

</html>
```

对应的lifeAdvice.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4F46E5',
                secondary: '#304FFF'
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
        // 生成标签
        function createTags(container, tags) {
            container.innerHTML = '';
            tags.forEach(tag => {
                const span = document.createElement('span');
                span.className = 'px-3 py-1 bg-blue-50 text-primary text-sm rounded-full';
                span.textContent = tag;
                span.addEventListener('click', () => {
                    jumpToDetail(tag)
                });
                container.appendChild(span);
            });
        }

        // 生成卡片
        function createCards(container, cards) {
            container.innerHTML = '';
            cards.forEach(card => {
                const div = document.createElement('div');
                div.className = 'bg-blue-50 p-4 rounded-lg';
                div.innerHTML = `
                    <div class="flex items-center mb-2">
                        <span class="text-sm text-gray-600">${card.title}</span>
                    </div>
                    <p class="text-sm text-gray-600">${card.content}</p>
                `;
                div.addEventListener('click', () => {
                    jumpToDetail(card.title)
                })
                container.appendChild(div);
            });
        }
        function loadTags(message) {
            // 使用用户信息作为缓存键的一部分
            const cacheKey = `lifeAdvice_${message}`;
            
            // 检查是否有缓存数据
            const cachedData = sessionStorage.getItem(cacheKey);
            if (cachedData) {
                const data = JSON.parse(cachedData);
                const tags = data.tags;
                if (tags) {
                    createTags(eatTag, tags.eat);
                    createTags(sportTag, tags.sport);
                    createCards(dailyCard, tags.daily);
                    createCards(popularizationCard, tags.popularization);
                }
                return;
            }

            let data = {
                type: "标签",
                userInfo: message
            }
            const loading = startLoading("正在生成个性化健康资讯")
            fetchLifeAdviceWorkflow(data, "system")
                .then(data => {
                    stopLoading(loading)
                    const tags = data.tags
                    if (tags) {
                        // 缓存数据，有效期到会话结束
                        sessionStorage.setItem(cacheKey, JSON.stringify(data));
                        
                        const eat = tags.eat
                        const sport = tags.sport
                        const daily = tags.daily
                        const popularization = tags.popularization
                        createTags(eatTag, eat);
                        createTags(sportTag, sport);
                        createCards(dailyCard, daily);
                        createCards(popularizationCard, popularization);
                    }
                })
        }
        function jumpToDetail(tag) {
            top.location.href = `lifeAdviceInfo.html?tag=${tag}`
        }
        window.onload = async function () {
            const eatTag = document.getElementById('eatTag');
            const sportTag = document.getElementById('sportTag');
            const dailyCard = document.getElementById('dailyCard');
            const popularizationCard = document.getElementById('popularizationCard');
            let riskMessage = '无信息'
            // const riskInfo = await getRiskInfo()
            getUserRiskInfo()
                .then(riskInfo => {
                    if (!riskInfo) {
                        Swal.fire({
                            title: '无健康信息',
                            text: '是否需要完善信息，信息未完善则无法提供个性化的健康资讯',
                            icon: 'info',
                            showCancelButton: true,
                            confirmButtonText: '去完善',
                            cancelButtonText: '取消'
                        }).then((result) => {
                            if (result.isConfirmed) {
                                top.location.href = '/scheme/getSchene.html'
                            } else {
                                loadTags("无信息")
                            }
                        });
                    } else {
                        riskMessage = `年龄:${riskInfo.age} 性别:${riskInfo.sex} 身高:${riskInfo.height} 体重:${riskInfo.weight} 是否患糖尿病:${riskInfo.disease} 医生健康资讯：${riskInfo.message}`
                        loadTags(riskMessage)
                    }
                })

        }
```

3.修改lifeadvice/lifeAdviceInfo.html文件，实现展示详情信息，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>健康资讯详情</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/loading.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/showAlert.js"></script>
    <script src="js/lifeAdviceInfo.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="/css/loading.css">
    <link href="/css/loading.css" rel="stylesheet">
</head>

<body class="bg-gray-50">
    <div class="w-full max-w-md min-h-screen mx-auto bg-white relative">
        <nav class="fixed top-0 w-full max-w-md h-14 bg-white border-b flex items-center justify-between px-4 z-50">
            <button class="w-[40px] h-[40px] flex items-center justify-center">
                <i class="fas fa-arrow-left text-gray-700"></i>
            </button>
            <h1 class="text-lg font-medium">健康资讯</h1>
            <button class="w-[40px] h-[40px] flex items-center justify-center">
            </button>
        </nav>
        <main class="pt-16 px-4 pb-6">
            <div class="mb-6 px-2">
                <h2 id="title" class="text-lg md:text-xl font-medium mb-2"></h2>
                <div class="flex items-center text-sm text-gray-500 mb-3">
                </div>
                <div id="tagList" class="flex gap-2 mb-4 flex-wrap">
                </div>
            </div>

            <article class="text-gray-700 text-sm md:text-base leading-relaxed mb-8 px-2">
            </article>
            <div id="collect" style="display: none;">
                如果感觉本健康资讯对您有帮助，请点击收藏按钮<br />
            <div class="flex items-center justify-between mb-8 px-4">
                <div class="flex items-center gap-4 md:gap-6" onclick="collect()">
                    <button class="flex items-center gap-2">
                        <i class="far fa-bookmark text-xl text-gray-600"></i>
                        <span class="text-gray-600">收藏</span>
                    </button>
                </div>
            </div>
            </div>
        </main>
    </div>
    <div
        class="fixed bottom-0 left-0 right-0 bg-yellow-100 p-3 md:p-4 text-center text-sm md:text-base text-yellow-800 font-medium">
        本资讯由AI生成，请谨慎鉴别
    </div>
</body>

</html>
```

对应的lifeAdviceInfo.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4F46E5',
                secondary: '#6366F1'
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
let lifeAdviceInfo = null;
function collect() {
    let loading = startLoading("收藏ing")
    const bookmarkIcon = document.querySelector('.fa-bookmark')
    const bookmarkText = document.querySelector('.fa-bookmark').nextElementSibling
    bookmarkIcon.classList.remove('far')
    bookmarkIcon.classList.add('fas')
    bookmarkIcon.classList.remove('text-gray-600')
    bookmarkIcon.classList.add('text-primary')
    bookmarkText.textContent = '已收藏'
    bookmarkText.classList.remove('text-gray-600')
    bookmarkText.classList.add('text-primary')
    const userInfo = getUserInfo()
    const collectMessage = `为用户ID为${userInfo.user_id}的用户在life_advice中添加一条数据，标题为${lifeAdviceInfo.title}，内容为${lifeAdviceInfo.content}，标签为${lifeAdviceInfo.tags}`
    fetchSQLWorkflow(collectMessage, "system").then(data => {
        showFloatingAlert("收藏成功", 'success')
        stopLoading(loading)

    })
}
window.onload = async function () {
    const title = document.getElementById("title")
    const tagList = document.getElementById("tagList")
    const article = document.querySelector("article")
    const urlParams = new URLSearchParams(window.location.search);
    const tag = urlParams.get('tag');
    let message = ""
    const riskInfo = await getUserRiskInfo()
    if (riskInfo) {
        message = `年龄:${riskInfo.age} 性别:${riskInfo.sex} 身高:${riskInfo.height} 体重:${riskInfo.weight} 是否患糖尿病:${riskInfo.disease} 医生建议：${riskInfo.message}`
    }
    const data = {
        type: "详情",
        title: tag,
        userInfo: message
    }
    const loading = startLoading(message == "" ? "正在生成健康资讯" : "正在生成个性化健康资讯")
    fetchLifeAdviceWorkflow(data, "system")
        .then(data => {
            stopLoading(loading)
            const content = data.content
            if (content) {
                lifeAdviceInfo = content
                title.textContent = content.title
                article.innerHTML = content.content
                const tags = content.tags
                tags.forEach(tag => {
                    const span = document.createElement("span")
                    span.className = "px-2 py-1 bg-blue-50 text-primary text-sm rounded-full whitespace-nowrap"
                    span.textContent = tag
                    tagList.appendChild(span)

                })
                document.getElementById("collect").style.display = "block"
            }
        })
}
```

\4. 修改lifeadvice/lifeAdviceList.html文件，代码如下：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>健康资讯列表</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/api.js"></script>
    <script src="/js/sweetalert2.all.min.js"></script>
    <script src="/js/user.js"></script>
    <script src="/js/loading.js"></script>
    <script src="/js/showAlert.js"></script>
    <script src="js/lifeAdviceList.js"></script>
    <link rel="stylesheet" href="/css/loading.css">
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="css/lifeAdviceList.css">
</head>

<body class="relative min-h-screen">
    <!-- 顶部导航栏 -->
    <header class="nav-bar bg-white shadow-sm">
        <div class="flex flex-col">
            <div class="flex items-center justify-between px-4 py-3">
                <button class="fa-icon text-gray-700" onclick="window.history.back()">
                    <i class="fas fa-chevron-left"></i>
                </button>
                <h1 class="text-lg font-semibold text-gray-900">健康资讯列表</h1>
                <button class="fa-icon text-gray-700">
                    <i class="fas fa-search"></i>
                </button>
            </div>
        </div>
    </header>
    <!-- 主要内容区域 -->
    <main class="content-area">

    </main>
</body>

</html>
```

对应的lifeAdviceList.js代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#3B82F6',
                secondary: '#10B981',
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
function showArticleDetail(title, content) {
    Swal.fire({
        title: `<div class="text-lg">${title}</div>`,
        html: `<div class="text-left p-4">${content}</div>`,
        showConfirmButton: true,
        confirmButtonText: '关闭',
        width: '90%',
        customClass: {
            popup: 'rounded-lg',
            title: 'text-lg',
            confirmButton: 'bg-primary text-white px-4 py-2 rounded-button hover:bg-blue-600'
        }
    });
}

function loadLA(arr) {
    const contentArea = document.querySelector(".content-area");
    arr.forEach(item => {
        let template = `
        <div class="divide-y divide-gray-100">
            <article class="article-item px-4 py-4" onclick="showArticleDetail('${item.title}', '${item.content.replace(/'/g, "\\'")}')">
                <div class="flex gap-4">
                    <div class="flex-1">
                        <h2 class="text-lg font-bold text-gray-900 mb-2">${item.title}</h2>
                        <p class="text-sm text-gray-500 mb-3 line-clamp-2">
                            ${item.content.substring(0, 40) + "......"}
                        </p>
                    </div>
                    <!-- 删除按钮 -->
                    <button class="fa-icon text-gray-400" onclick="event.stopPropagation(); removeLA(${item.id})">
                        <i class="fas fa-trash"></i>
                    </button>
                </div>
            </article>
        </div>
            `;
        contentArea.innerHTML += template;
    })

}
function loadPage(userId) {
    let loading = startLoading("正在加载")
    fetchSQLWorkflow(`查询用户id为${userId}的所有life_advice数据`, "system")
        .then(data => {
            stopLoading(loading)
            const arr = data.result;

            if (arr.length == 0) {
                showFloatingAlert("暂无数据", "error")
            } else {
                loadLA(arr)
            }
        })
}
function removeLA(id) {
    Swal.fire({
        title: '确认删除',
        text: '您确定要删除这条健康资讯吗？',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: '确认删除',
        cancelButtonText: '取消',
        customClass: {
            confirmButton: 'bg-red-500 text-white px-4 py-2 rounded-button hover:bg-red-600',
            cancelButton: 'bg-gray-200 text-gray-700 px-4 py-2 rounded-button hover:bg-gray-300'
        }
    }).then((result) => {
        if (result.isConfirmed) {
            fetchSQLWorkflow(`删除id为${id}的life_advice数据`, "system")
                .then(data => {
                    showFloatingAlert('删除成功', 'success');
                    // Reload the page to reflect changes
                    document.querySelector(".content-area").innerHTML = "";
                    const userinfo = getUserInfo();
                    loadPage(userinfo.user_id);
                })
                .catch(error => {
                    showFloatingAlert('删除失败', 'error');
                    console.error(error);
                });
        }
    });
}
window.onload = function () {
    const userinfo = getUserInfo();
    loadPage(userinfo.user_id);
}
```