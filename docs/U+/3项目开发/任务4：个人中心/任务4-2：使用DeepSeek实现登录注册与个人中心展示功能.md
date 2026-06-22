任务详情

任务4-2：使用DeepSeek实现登录注册与个人中心展示功能

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

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fb38e52dd8aa24509991daf31c3d9d933.png)

图1 个人信息展示页面



任务指导

##### 1. 登录注册功能实现

在之前的任务2-4中已经实现了数据管理工作流，在本任务中可以借助数据管理工作流来实现登录和注册功能。

登录操作的本质是通过用户名和密码查找用户信息，而注册的本质是新增一条用户信息。

进入到Dify中的数据管理工作流，首先测试登录功能，提示词可参考：查询用户名为“test_user”且密码为"password123"的用户信息。如下图所示：

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2F924041239cf545efae8a409c0abfc5cc.png)

图1 测试数据管理工作流

接下来测试注册功能，提示词可参考：创建用户名为“admin”密码为“123456”的用户。结果如下图所示。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250416%2Ff564b80bd6aa4b7a8077791b64758421.png)

图2 测试结果

登录和注册都验证成功后，那么登录功能与注册功能则可以通过直接调用数据管理工作流来实现，注意：在本项目中仅为演示工作流接入操作，在实际的开发中，登录和注册流程都有比较严格的校验逻辑。

在Cline中编写以下提示词，让大模型通过main.html中接口的调用方式来实现注册和登录功能。

```
@/login.html 实现注册和登录功能，接口调用方式可参考@/main/main.html 
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F3b6624fb57374803998180ac8b349c25.png)

图3 登录和注册提示词

生成代码后，优化代码逻辑。

##### 2. 个人中心用户显示功能设计与功能实现

在登录后，用户信息一般是存放在本地LocalStorage或SessionStorage中，如果想要实现获取用户信息并展示功能，首先编写获取用户信息方法，然后再根据数据渲染展示。

首先实现获取个人信息方法，判断如果没有获取到个人信息，则跳转至登录页，参考提示词如下。

```
在js目录中创建user.js文件，并实现通过LocalStorage获取用户信息方法，如果没有获取到用户信息，则跳转至登录页面。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fbdf18cfc88104652a3aaa42ff4be3568.png)

图4 个人信息展示提示词

生成代码后，再在mine.html文件中实现用户信息显示功能，除此之外还需要考虑如果是admin用户（管理员）的话，则显示“AI智能管理平台”操作栏。

```
@/mine\mine.html 在此文件中渲染用户头像和用户名，如果用户名为admin则显示“AI智能管理平台”操作栏
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fb570ad908aa7463da01d28c71935c09c.png)

图5 管理栏展示提示词

除此之外，还需要实现退出登录功能，当在主页面中点击“退出登录”按钮时，清空用户数据，并跳转至登录页面。参考以下提示词。

```
@/mine\mine.html 实现退出登录功能，当点击退出按钮时清空缓存，并跳转至登录页面。
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2F70cbb74d44ad4a7d869c1957cd6cab74.png)

图6 退出登录提示词

通过DeepSeek&Cline完成代码后，测试其项目功能并修改代码，最终效果如下。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250417%2Fb38e52dd8aa24509991daf31c3d9d933.png)

图7 个人中心

任务实现

由于每个人的提示词可能存在差异，AI工具再生成代码时也可能存在差异，本次项目不要求每人的代码内容完全相同，本任务实现代码仅供参考。

1.在js/login.js中实现登录功能

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#2563eb',
                secondary: '#60a5fa'
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
window.onload = function() {
    const loginTab = document.getElementById('loginTab');
    const registerTab = document.getElementById('registerTab');
    const loginContent = document.getElementById('loginContent');
    const registerContent = document.getElementById('registerContent');

    // 添加空值检查
    if (!loginTab || !registerTab || !loginContent || !registerContent) {
        console.error('One or more elements not found');
        return;
    }

    loginTab.addEventListener('click', () => {
        loginTab.classList.add('text-primary', 'border-primary');
        loginTab.classList.remove('text-gray-400', 'border-gray-200');
        registerTab.classList.add('text-gray-400', 'border-gray-200');
        registerTab.classList.remove('text-primary', 'border-primary');
        loginContent.classList.add('active');
        registerContent.classList.remove('active');
    });

    registerTab.addEventListener('click', () => {
        registerTab.classList.add('text-primary', 'border-primary');
        registerTab.classList.remove('text-gray-400', 'border-gray-200');
        loginTab.classList.add('text-gray-400', 'border-gray-200');
        loginTab.classList.remove('text-primary', 'border-primary');
        registerContent.classList.add('active');
        loginContent.classList.remove('active');
    }); 
}
function login() {
    const username = document.getElementById("username").value;
    const password = document.getElementById("password").value;
    if (username && password) {
        fetchSQLWorkflow(`查询用户名为${username}且密码为${password}的用户信息`, username)
            .then(data => {
                res = data.result
                console.log(res)
                // 没有查到相关的用户
                if (res.length == 0) {
                    showFloatingAlert("用户名或密码错误")
                } else {
                    localStorage.setItem("user", JSON.stringify(res[0]))
                    location.href = "index.html"
                }
            })
    } else {
        showFloatingAlert("请确保表单填写完整", 'warning')
    }
}
function register() {
    const regUsername = document.getElementById("regUsername").value;
    const regPassword = document.getElementById("regPassword").value;
    const secPassword = document.getElementById("secPassword").value;
    if (regUsername && regPassword & secPassword) {
        if (regPassword == secPassword) {
            fetchSQLWorkflow(`创建用户名为'${regUsername}' 密码为'${regPassword}'的用户`, regUsername)
                .then(data => {
                    res = data.result[0].result
                    console.log(res)
                    if (res == 1) {
                        showFloatingAlert("注册成功", 'success')
                    } else if (res.includes("UNIQUE constraint failed")) {
                        showFloatingAlert("注册失败,用户名已存在", 'error')
                    } else {
                        showFloatingAlert("服务器异常" + res, 'error')
                    }
                })
        } else {
            showFloatingAlert("两次输入的密码不一致", 'warning')
        }
    } else {
        showFloatingAlert("请确保表单填写完整", 'warning')
    }
}
```

对应的login.html代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=375px, initial-scale=1.0">
    <title>医疗平台登录</title>
    <script src="/js/tailwindcss.js"></script>
    <link rel="stylesheet" href="/css/all.min.css">
    <link rel="stylesheet" href="css/login.css">
    <script src="js/api.js"></script>
    <script src="js/login.js"></script>
    <script src="js/showAlert.js"></script>
</head>

<body class="bg-gray-50">
    <div class="min-h-screen flex flex-col items-center justify-center px-4">
        <div class="w-full max-w-[375px] bg-white rounded-xl shadow-lg p-8">
            <div class="flex justify-center mb-8">
                <img src="/img/logo.png" width="100" height="100">
            </div>
            <div class="flex mb-6">
                <button class="flex-1 py-2 text-center text-primary border-b-2 border-primary" id="loginTab">登录</button>
                <button class="flex-1 py-2 text-center text-gray-400 border-b-2 border-gray-200"
                    id="registerTab">注册</button>
            </div>
            <div id="loginContent" class="tab-content active">
                <div class="mb-4">
                    <div class="relative">
                        <i class="fas fa-user absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                        <input id="username" type="text" placeholder="请输入用户名"
                            class="w-full pl-10 pr-4 py-3 border border-gray-200 rounded-button focus:outline-none focus:border-primary">
                    </div>
                </div>
                <div class="mb-4">
                    <div class="relative">
                        <i class="fas fa-lock absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                        <input id="password" type="password" placeholder="请输入密码"
                            class="w-full pl-10 pr-4 py-3 border border-gray-200 rounded-button focus:outline-none focus:border-primary">
                        <i
                            class="fas fa-eye absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 cursor-pointer"></i>
                    </div>
                </div>
                <div class="flex items-center mb-6">
                    <label class="flex items-center cursor-pointer">
                        <div class="relative">
                            <input type="checkbox" class="sr-only">
                            <div class="w-4 h-4 border border-gray-300 rounded-sm"></div>
                            <div
                                class="hidden absolute inset-0 flex items-center justify-center text-white bg-primary rounded-sm">
                                <i class="fas fa-check text-xs"></i>
                            </div>
                        </div>
                        <span class="ml-2 text-sm text-gray-600">记住密码</span>
                    </label>
                    <a href="#" class="ml-auto text-sm text-primary">忘记密码？</a>
                </div>
                <button class="w-full py-3 bg-primary text-white rounded-button mb-6" onclick="login()">登录</button>
            </div>
            <div id="registerContent" class="tab-content">
                <div class="mb-4">
                    <div class="relative">
                        <i class="fas fa-user absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                        <input id="regUsername" type="text" placeholder="请输入用户名"
                            class="w-full pl-10 pr-4 py-3 border border-gray-200 rounded-button focus:outline-none focus:border-primary">
                    </div>
                </div>
                <div class="mb-4">
                    <div class="relative">
                        <i class="fas fa-lock absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                        <input id="regPassword" type="password" placeholder="请设置密码"
                            class="w-full pl-10 pr-4 py-3 border border-gray-200 rounded-button focus:outline-none focus:border-primary">
                        <i
                            class="fas fa-eye absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 cursor-pointer"></i>
                    </div>
                </div>
                <div class="mb-4">
                    <div class="relative">
                        <i class="fas fa-lock absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                        <input id="secPassword" type="password" placeholder="请确认密码"
                            class="w-full pl-10 pr-4 py-3 border border-gray-200 rounded-button focus:outline-none focus:border-primary">
                        <i
                            class="fas fa-eye absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 cursor-pointer"></i>
                    </div>
                </div>
                <button class="w-full py-3 bg-primary text-white rounded-button mb-6" onclick="register()">注册</button>
            </div>
            <div class="text-center text-xs text-gray-400">
                登录即表示同意 <a href="#" class="text-primary">用户协议</a> 和 <a href="#" class="text-primary">隐私政策</a>
            </div>
        </div>
    </div>

</body>

</html>
```

2.在js目录中创建user.js，实现获取用户信息功能。代码如下：

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
```

3.在mine/js/mine.js中实现用户信息显示及退出登录功能。代码如下：

```
tailwind.config = {
    theme: {
        extend: {
            colors: {
                primary: '#4F46E5',
                secondary: '#E5E7EB'
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
window.onload = function() {
    const user = getUserInfo()
    console.log(user)
    if (user) {
        document.getElementById('userAvatar').src = user.avatar_url || '/img/user2.png'
        document.getElementById('userName').textContent = user.username || '用户'
        
        // Hide admin menu if not admin
        const adminMenuItem = document.querySelector('#menuList div:nth-child(6)')
        if (adminMenuItem && user.username !== 'admin') {
            adminMenuItem.style.display = 'none'
        }
    }

    // Add logout functionality
    document.getElementById('logoutBtn').addEventListener('click', function() {
        localStorage.removeItem('user')
        top.location.href = '/login.html'
    }) 
}
```

对应修改mine/mine.html代码，代码如下：

```
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>个人中心</title>
    <script src="/js/tailwindcss.js"></script>
    <script src="/js/user.js"></script>
    <script src="js/mine.js"></script>
    <link rel="stylesheet" href="css/mine.css">
    <link rel="stylesheet" href="/css/all.min.css">
</head>

<body class="bg-gray-50">
    <!-- 顶部导航栏 -->
    <header class="fixed top-0 left-0 right-0 z-10 shadow-sm bg-white">
        <div class="flex items-center justify-between px-4 py-3">
            <button>
            </button>
            <h1 class="text-lg font-medium">个人中心</h1>
            <button >
            </button>
        </div>
    </header>

    <div class="safe-area">
        <div class="m-4 bg-white rounded-lg shadow-md overflow-hidden">
            <div class="px-4 pt-6 pb-6">
                <div class="text-center">
                    <div class="relative w-20 h-20 mx-auto">
                        <img id="userAvatar" src="/img/user2.png" class="w-full h-full rounded-full object-cover border-2 border-blue-500" alt="avatar">
                    </div>
                    <h2 class="text-lg font-medium mt-2" id="userName">赵晓峰</h2>
                </div>
            </div>
            <div class="mt-3">
            <div class="px-4 py-2" id="menuList">
                <div class="py-3 flex items-center justify-between">
                    <div class="flex items-center" onclick="top.location.href = '/userinfo/userinfo.html'">
                        <img src="/img/menu1.png" width="22" height="22">
                        <span class="ml-3">个人信息</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>
                <div class="py-3 flex items-center justify-between border-t border-gray-100">
                    <div class="flex items-center" onclick="top.location.href = '/scheme/scheme.html'">
                        <img src="/img/menu2.png" width="22" height="22">
                        <span class="ml-3">我的方案</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>
                <div class="py-3 flex items-center justify-between border-t border-gray-100">
                    <div class="flex items-center" onclick="top.location.href = '/lifeadvice/lifeAdviceList.html'">
                        <img src="/img/menu3.png" width="22" height="22">
                        <span class="ml-3">我的建议</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>
                <div class="py-3 flex items-center justify-between border-t border-gray-100">
                    <div class="flex items-center" onclick="top.location.href = '/checkcard/checkcardList.html'">
                        <img src="/img/menu4.png" width="22" height="22">
                        <span class="ml-3">打卡记录</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>
                <div class="py-3 flex items-center justify-between border-t border-gray-100">
                    <div class="flex items-center">
                        <img src="/img/menu5.png" width="22" height="22">
                        <span class="ml-3">帮助中心</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>
                <div class="py-3 flex items-center justify-between border-t border-gray-100">
                    <div class="flex items-center">
                        <img src="/img/menu7.png" width="22" height="22">
                        <span class="ml-3">AI智能管理平台</span>
                    </div>
                    <i class="fas fa-chevron-right text-gray-400"></i>
                </div>

            </div>
        </div>

            </div>
            <div class="px-4 mt-6 pb-6">
                <button id="logoutBtn" class="block w-2/3 mx-auto bg-white border border-gray-200 text-gray-500 py-3 !rounded-button">
                    退出登录
                </button>
            </div>
        </div>
    </div>
    
</body>

</html>
```