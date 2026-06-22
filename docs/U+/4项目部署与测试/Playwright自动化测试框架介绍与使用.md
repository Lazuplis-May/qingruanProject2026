任务详情

Playwright自动化测试框架介绍与使用

建议工时：

1

任务描述

**1. 任务描述**

自动化测试可以帮助开发者快速发现和修复软件中的问题，提高软件质量，并确保软件的稳定性。Playwright是一个由微软开发的现代浏览器自动化库，它旨在提供一个简洁、强大和灵活的方式来编写跨浏览器（包括Chrome、Firefox和WebKit）的自动化测试。本任务将学习Playwright如何进行自动化测试

**2. 任务知识**

**知识点 ：**Playwright框架的搭建与配置，使用Playwright进行自动化测试

**难点 ：**Playwright框架的搭建与配置，使用Playwright进行自动化测试

**3. 任务成果**

本任务完成Playwright框架的搭建与配置，并进行验证。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250402%2F915d7b960bb54ae8a4b5db5f63e08354.png)

图1 使用Playwright进行自动化测试



任务指导

**1. 什么是****自动化测试**

自动化测试是一种通过使用自动化测试工具和脚本来执行测试用例，以验证软件系统是否满足预期功能和性能要求的测试方法。通过自动化测试，可以提高测试效率，减少人工测试的工作量和时间成本，尤其是在重复执行相同测试用例时；可以提高测试的准确性和可靠性，避免人工测试可能出现的疏忽和错误；能够更快速地发现软件中的缺陷，有助于及时修复问题，提高软件质量；支持持续集成和持续交付流程，确保软件在每次代码更新后都能及时进行测试，保证系统的稳定性和可靠性。

 

**2. 什么是Playwright框架**

Playwright 是一个由微软开发的自动化测试框架，专为现代网络应用的端到端（E2E）测试而设计。它能支持多种编程语言，像 JavaScript、TypeScript、Python、.NET 和 Java 等。主要特点如下：

- 多浏览器支持：Playwright 可以在 Chromium（涵盖 Chrome 和 Edge）、Firefox 以及 WebKit（用于 Safari）等主流浏览器上执行测试，确保应用在不同浏览器环境下的兼容性。
- 跨平台兼容：它能在 Windows、macOS 和 Linux 等多个操作系统上运行，无论是本地开发环境还是 CI/CD（持续集成 / 持续部署）系统，都能很好地支持。
- 自动等待机制：该框架具备自动等待机制，在执行操作前会自动等待元素加载完成、可交互等，减少手动添加等待时间的需求，避免因页面加载缓慢而导致的测试失败。
- 高速稳定：Playwright 采用了底层浏览器驱动技术，测试执行速度快且稳定性高，能够高效地处理复杂的页面交互和动态内容。
- 丰富的 API：提供了一系列简洁易用的 API，可用于模拟各种用户操作，如点击、输入、滚动等，还能对页面元素进行检查和断言。
- 可视化调试：支持有头模式运行测试，即显示浏览器窗口，方便调试。同时，还提供了 Trace Viewer 工具，能记录测试执行过程中的详细信息，包括页面截图、网络请求等，便于分析测试失败的原因。
- 并行测试：支持并行运行测试用例，充分利用多核处理器的性能，显著缩短测试时间。

 

**3. 搭建****Playwright自动化测试框架**

**1）初始化项目**

Playwright框架依赖于nodejs环境，在初始化项目前，先下载并安装nodejs，可在项目资料中进行下载。

创建一个新的项目，当作自动化测试项目，如创建一个Playwright_test项目目录，并用VSCode打开该项目。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/b6ede7dc667344c9b6cf219f83a9cd69%2Frichtext%2Fimage%2F20250427%2Fdf466a5a5e864166a07d8212b958a9cc.png)

图1 创建项目

 

点击VSCode上方，选择“终端”->“新建终端”，在新建的终端中输入并执行以下命令，在该目录下初始化 package.json 文件。

```
npm init -y
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250402%2F50535e4684e94bf2b018d9bd3fd0b6df.png)

图2 创建package.json 文件

 

**2）安装 Playwright 及测试包**

在项目中安装 Playwright 及其测试包，同时安装必要的浏览器。执行以下命令：

```
# 安装 Playwright 测试包作为开发依赖
npm install --save-dev @playwright/test
# 安装 Playwright 支持的浏览器
npx playwright install
```

 

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250402%2Fad44ef9167124fd19f6c2dd4c970ffa4.png)

图3 安装 Playwright 及测试包

 

**3）配置测试脚本**

修改 package.json 文件如下：

```
{
  "devDependencies": {
    "@playwright/test": "^1.51.1"
  }
}
```

**4）配置**

如果需要对 Playwright 进行更多的配置，比如指定浏览器、设置测试超时时间等，可以在项目根目录下创建一个 playwright.config.js 文件，并进行相应的配置：

```
// @ts-check
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,
  expect: {
    timeout: 5000
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    actionTimeout: 0,
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
});
```

**5）创建测试文件**

在项目根目录下创建一个 tests 文件夹，用于存放测试文件。在 tests 文件夹中创建一个测试文件，例如 example.spec.js，并编写简单的测试代码：

```
// tests/example.spec.js
const { test, expect } = require('@playwright/test');

test('basic test', async ({ page }) => {
    // 导航到目标页面
    await page.goto('https://example.com');
    // 获取页面标题
    const title = await page.title();
    // 断言页面标题是否符合预期
    expect(title).toBe('Example Domain');
});
```

**6）运行测试**

完成上述步骤后，就可以运行测试了。在终端中执行以下命令：

```
npx playwright test tests/example.spec.js
```

Playwright 会自动查找 tests 文件夹下的example.spec.js文件，并执行其中的测试用例。执行完成后，会在终端输出测试结果。

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250402%2F915d7b960bb54ae8a4b5db5f63e08354.png)

图4 运行测试

 

**7）查看测试报告**

根据提示，可以使用以下语句访问自动生成的测试报告，系统会自动打开浏览器中的测试报告。

```
npx playwright show-report
```

![img](https://tecs-prod-static.obs.cn-north-1.myhuaweicloud.com/fa7280d63263430a935ef1c9945a639b%2Frichtext%2Fimage%2F20250402%2F8cffc36b097940de8a0907a498ee84e1.png)

图5 查看测试报告