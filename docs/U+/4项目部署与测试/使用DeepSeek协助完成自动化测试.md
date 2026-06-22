任务详情

使用DeepSeek协助完成自动化测试

建议工时：

2

任务描述

**1. 任务描述**

上一步任务中我们学习了Playwright如何进行自动化测试，本任务将借助DeepSeek完成测试脚本，在Playwright运行测试脚本，完成项目功能测试。

**2. 任务知识**

**知识点：**完成测试用例、使用DeepSeek生成测试脚本，自动化测试

**重点：**使用DeepSeek生成测试脚本

**3. 任务成果**

生成测试脚本，完成自动化测试

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F291cda3d86c44f15a0ce4fce04104b34.png)

图1 生成测试脚本

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F2384fa62c0e548cca96361fe0240995b.png)

图2 查看测试报告信息



任务指导

\1. 使用DeepSeek协助生成首页的测试用例

打开DeepSeek首页，编写提示词进行生成，由于本项目的功能非常多，所以仅通过登录页面进行说明距离。

```
假如你是一个专业的测试工程师，请根据以下任务描述，生成UI界面的测试用例：
展示登录页面：登录页面包括登录、注册功能和表单切换功能；
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2Fa7b2aa0a5af74674aba57e538d21e179.png)

图1 编写测试用例

 

得到结果如下：

```
一、表单切换功能测试
用例编号	测试场景	测试步骤	预期结果
TS-001	默认显示登录表单	1. 访问登录页面	登录表单显示（包含邮箱/手机号、密码字段）
TS-002	切换至注册表单	1. 点击"注册"按钮	显示注册表单（含用户名、邮箱、密码、确认密码字段）
TS-003	从注册切换回登录表单	1. 在注册表单中点击"登录"按钮	返回登录表单，注册表单隐藏
TS-004	切换时表单数据清空	1. 在登录表单输入数据
2. 切换至注册表单
3. 切换回登录表单	登录表单输入框内容被清空
二、登录功能测试
用例编号	测试场景	测试步骤	预期结果
TL-001	空字段提交	1. 不输入任何内容点击"登录"	显示"请输入邮箱/手机号"提示
TL-002	无效邮箱格式	1. 输入"user@domain"
2. 点击登录	显示"邮箱格式不正确"提示
TL-003	正确格式未注册账户	1. 输入未注册邮箱
2. 输入任意密码
3. 点击登录	显示"账户不存在，请先注册"提示
TL-004	密码错误	1. 输入已注册邮箱
2. 输入错误密码
3. 点击登录	显示"密码不正确"提示
TL-005	正常登录	1. 输入正确凭证
2. 点击登录	跳转至用户主页/仪表盘
三、注册功能测试
用例编号	测试场景	测试步骤	预期结果
TR-001	必填字段缺失	1. 留空任一必填字段点击注册	对应字段显示"该字段为必填项"提示
TR-002	密码复杂度不足	1. 输入简单密码（如123456）	显示"密码需包含大小写字母和数字，至少8位"提示
TR-003	密码不一致	1. 输入不同密码和确认密码	显示"两次输入的密码不一致"提示
TR-004	重复邮箱注册	1. 输入已注册邮箱进行注册	显示"该邮箱已被注册"提示
TR-005	正常注册	1. 输入有效新用户信息
2. 点击注册	显示"注册成功"提示并自动跳转至登录页
四、界面与交互验证
用例编号	测试点	验证内容
UI-001	视觉一致性	页面元素符合设计规范（颜色/字体/间距）
UI-002	响应式布局	在移动端（320px-768px）显示正常，表单无横向滚动
UI-003	焦点状态	Tab键切换焦点顺序符合逻辑，输入框获得焦点时有明显视觉反馈
UI-004	错误提示样式	错误消息使用红色文字+图标，显示在对应字段下方
UI-005	加载状态反馈	提交表单时显示加载动画，按钮变为不可点击状态
五、安全性测试
用例编号	测试场景	验证内容
SEC-001	密码字段屏蔽	密码输入显示为星号或圆点
SEC-002	XSS防护	输入<script>alert(1)</script>时进行转义处理
SEC-003	频繁登录限制	连续5次失败登录后显示验证码或临时锁定
六、辅助功能测试
用例编号	测试点	验证内容
AC-001	屏幕阅读器支持	表单字段均有正确的aria-label描述
AC-002	键盘操作	可通过Enter键提交表单
AC-003	高对比度模式	在Windows高对比度主题下文字可清晰辨识
说明：建议配合自动化工具（如Selenium）实现核心功能用例的自动化回归测试，同时对关键路径（TL-005/TR-005）进行端到端测试。
```

DeepSeek生成内容比较多，可以根据实际情况删除部分测试内容，并根据实际情况修改测试内容。

\2. 使用DeepSeek协助生成Playwright自动化测试脚本

打开VSCode，根据刚才生成的测试用例，在Cline中编写提示词，协助生成Playwright自动化测试脚本。并且可让其参考测试的文件。

```
假如你是一位测试工程师，请根据以下测试用例，生成playwright可以运行的测试脚本。放入新创建的homepage.spec.js文件中，并对脚本代码添加注释。
 UI界面测试用例
一、表单切换功能测试
用例编号	测试场景	测试步骤	预期结果
TS-001	默认显示登录表单	1. 访问登录页面	登录表单显示（包含用户名、密码字段）
TS-002	切换至注册表单	1. 点击"注册"按钮	显示注册表单（含用户名、密码、确认密码字段）
TS-003	从注册切换回登录表单	1. 在注册表单中点击"登录"按钮	返回登录表单，注册表单隐藏
TS-004	切换时表单数据清空	1. 在登录表单输入数据
2. 切换至注册表单
3. 切换回登录表单	登录表单输入框内容被清空
二、登录功能测试
用例编号	测试场景	测试步骤	预期结果
TL-001	空字段提交	1. 不输入任何内容点击"登录"	显示"请输入邮箱/手机号"提示
TL-002	无效邮箱格式	1. 输入"user@domain"
2. 点击登录	显示"邮箱格式不正确"提示
TL-003	正确格式未注册账户	1. 输入未注册邮箱
2. 输入任意密码
3. 点击登录	显示"账户不存在，请先注册"提示
TL-004	密码错误	1. 输入已注册邮箱
2. 输入错误密码
3. 点击登录	显示"密码不正确"提示
TL-005	正常登录	1. 输入正确凭证
 2. 点击登录	跳转至用户主页/仪表盘
三、注册功能测试
用例编号	测试场景	测试步骤	预期结果
TR-001	必填字段缺失	1. 留空任一必填字段点击注册	对应字段显示"该字段为必填项"提示
TR-002	密码复杂度不足	1. 输入简单密码（如123456）	显示"密码需包含大小写字母和数字，至少8位"提示
TR-003	密码不一致	1. 输入不同密码和确认密码	显示"两次输入的密码不一致"提示
TR-004	重复邮箱注册	1. 输入已注册邮箱进行注册	显示"该邮箱已被注册"提示
TR-005	正常注册	1. 输入有效新用户信息
2. 点击注册	显示"注册成功"提示并自动跳转至登录页
登录界面代码参考@/login.html 
```

Cline生成测试脚本。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F291cda3d86c44f15a0ce4fce04104b34.png)

图2 生成测试脚本

 

结果如下。

```
 const { test, expect } = require('@playwright/test');

test.describe('登录/注册页面测试', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('login.html');
  });

  test.describe('表单切换功能测试', () => {
    test('TS-001: 默认显示登录表单', async ({ page }) => {
      // 验证默认显示登录表单
      await expect(page.locator('#loginContent')).toBeVisible();
      await expect(page.locator('#registerContent')).not.toBeVisible();
      
      // 验证登录表单包含用户名和密码字段
      await expect(page.locator('#username')).toBeVisible();
      await expect(page.locator('#password')).toBeVisible();
    });

    test('TS-002: 可以切换到注册表单', async ({ page }) => {
      // 点击注册标签
      await page.locator('#registerTab').click();
      
      // 验证注册表单显示且包含所有字段
      await expect(page.locator('#registerContent')).toBeVisible();
      await expect(page.locator('#regUsername')).toBeVisible();
      await expect(page.locator('#regPassword')).toBeVisible();
      await expect(page.locator('#secPassword')).toBeVisible();
    });

    test('TS-003: 可以从注册表单切换回登录表单', async ({ page }) => {
      // 先切换到注册表单
      await page.locator('#registerTab').click();
      
      // 再切换回登录表单
      await page.locator('#loginTab').click();
      
      // 验证登录表单再次显示
      await expect(page.locator('#loginContent')).toBeVisible();
      await expect(page.locator('#registerContent')).not.toBeVisible();
    });

    test('TS-004: 切换时表单数据被清空', async ({ page }) => {
      // 在登录表单输入数据
      await page.locator('#username').fill('testuser');
      await page.locator('#password').fill('password');
      
      // 切换到注册表单再切换回来
      await page.locator('#registerTab').click();
      await page.locator('#loginTab').click();
      
      // 验证输入框内容已被清空
      await expect(page.locator('#username')).toHaveValue('');
      await expect(page.locator('#password')).toHaveValue('');
    });
  });

  test.describe('登录验证测试', () => {
    test('TL-001: 空字段提交显示验证提示', async ({ page }) => {
      await page.locator('button[onclick="login()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('请确保表单填写完整');
    });

    test('TL-002: 无效邮箱格式验证', async ({ page }) => {
      await page.locator('#username').fill('user@domain');
      await page.locator('button[onclick="login()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('邮箱格式不正确');
    });

    test('TL-003: 未注册账户验证', async ({ page }) => {
      await page.locator('#username').fill('nonexistent@user.com');
      await page.locator('#password').fill('anypassword');
      await page.locator('button[onclick="login()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('用户名或密码错误');
    });

    test('TL-005: 成功登录跳转', async ({ page }) => {
      // 此测试需要模拟API响应才能正常工作
      // 跳过实际实现，因为需要后端集成
      test.skip();
    });
  });

  test.describe('注册验证测试', () => {
    test('TR-001: 必填字段验证', async ({ page }) => {
      await page.locator('#registerTab').click();
      await page.locator('button[onclick="register()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('请确保表单填写完整');
    });

    test('TR-002: 密码复杂度验证', async ({ page }) => {
      await page.locator('#registerTab').click();
      await page.locator('#regUsername').fill('newuser');
      await page.locator('#regPassword').fill('123456');
      await page.locator('#secPassword').fill('123456');
      await page.locator('button[onclick="register()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('密码需包含大小写字母和数字');
    });

    test('TR-003: 密码不一致验证', async ({ page }) => {
      await page.locator('#registerTab').click();
      await page.locator('#regUsername').fill('newuser');
      await page.locator('#regPassword').fill('Password1');
      await page.locator('#secPassword').fill('Password2');
      await page.locator('button[onclick="register()"]').click();
      await expect(page.locator('.floating-alert')).toContainText('两次输入的密码不一致');
    });

    test('TR-005: 成功注册', async ({ page }) => {
      // 此测试需要模拟API响应才能正常工作
      // 跳过实际实现，因为需要后端集成
      test.skip();
    });
  });
});
```

\3. 修改测试脚本

根据实际情况，结合需要测试的HTML页面，再查看测试脚本代码，可能需要修改逻辑或脚本元素名称等内容，以正确定位到元素，获得正确的判断。

首先修改第1模块：新闻轮播图测试模块

```
test.beforeEach(async ({ page }) => {
      await page.goto('https://your-website.com');
    });
```

1）每个测试用例前先访问首页，所以需要修改以下语句：

```
await page.goto('https://your-website.com');
```

将其重写为具体的部署好的首页路径：

注意：在上一任务中我们已经部署了本项目，所以本次测试以部署的项目来进行测试。

```
await page.goto('http://192.168.59.246:8080/login.html');
```

检查其他代码是否有问题，可以手动修改或者使用DeepSeek辅助修改。

 

\4. 运行测试

在终端中执行以下命令运行测试：

```
npx playwright test tests/homepage.spec.js
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F48f598ab364645d9987978427049c92b.png)

图3 运行测试

通过上述可知，7个测试用例失败。

\5. 问题修改

当测试执行完毕后，会打开测试报告页面。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F1007d67fcce24996abf7f36a022990c4.png)

图4 查看测试报告

点击其中一个测试用例，可以看到切换表单时，表单内容未被清空，可以选择修改代码新增表单清空功能，也可以忽略此测试用例。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250423%2F2384fa62c0e548cca96361fe0240995b.png)

图5 查看测试详情

根据以上步骤实现其他的功能测试。

 