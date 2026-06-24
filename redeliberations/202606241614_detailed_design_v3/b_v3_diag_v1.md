# 糖尿病预治智能助手 —— 详细设计文档（v3_tech_v2）质量审查诊断报告（v3_diag_v1）

本报告针对待审查文件 `a_v3_tech_v2.md` 进行质量审查，重点诊断需求响应充分度、整体深度、逻辑一致性以及工程实施可行性。本轮诊断共识别出 7 个质量问题，详细说明如下：

## 1. 质量问题清单

### 问题 1：Dify 回调接口 `/api/admin/execute` 中硬编码 operatorRole 为 'user'，导致管理员的 Text2SQL 功能失效
*   **问题描述**：在第 7.3.3 节的 SQL 执行处理器路由伪代码（`executeHandler`）中，处理 Dify Agent 回调认证路径时，系统硬编码了 `operatorRole = 'user'`：
    ```javascript
    if (req.difyAuth && req.difyAuth.mode === 'callback') {
      operatorId = req.difyAuth.userId;
      operatorRole = 'user';  // Dify回调路径的操作者始终视为普通用户
      authMode = 'dify_callback';
    }
    ```
    然而，系统设计了 `admin-manager-agent` 专门协助系统管理员通过自然语言管理和操作 SQLite 数据库（如查询所有用户、修改/删除任意数据等）。由于在此回调路径中 `operatorRole` 始终被硬编码为 `'user'`，当管理员在聊天弹窗或页面中向 `admin-manager-agent` 发送管理指令并触发 Text2SQL 接口时，该请求在后端将被视为普通用户请求，从而进入行级权限校验（`validateRowLevelPermission(sql, operatorId)`）。这会导致管理员的跨用户查询与管理操作被安全策略拦截，返回 403 Forbidden（"仅允许操作本人数据"），彻底破坏了管理员的核心管理功能。
*   **所在位置**：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5196 行）。
*   **严重程度**：严重 (Major)
*   **改进建议**：后端在通过 Dify API Key 校验后，应根据 `operatorId` 查询数据库中的用户角色（例如执行 `SELECT role FROM users WHERE id = ?`），动态确定 `operatorRole` 的实际值（'admin' 或 'user'），而不能一律硬编码为 'user'。

### 问题 2：`/api/admin/execute` 的安全校验函数 `validateRowLevelPermission` 缺失具体实现规范
*   **问题描述**：在第 7.3.3 节中，对普通用户的 SQL 执行请求使用 `validateRowLevelPermission(sql, operatorId)` 进行行级安全校验，以防止用户通过 Text2SQL 越权访问或修改他人数据。但在整个设计文档中，完全没有定义或说明该函数的具体校验规则、实现逻辑或算法（例如：是使用 AST 解析 SQL 树，还是通过正则表达式匹配）。由于利用大语言模型（LLM）生成的任意 SQL 语句进行行级安全控制是一个极具技术挑战和高风险的实现，完全缺失该函数的设计规格将导致后端开发人员无法着手实现，存在巨大的行级权限漏洞或返工风险。
*   **所在位置**：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5209 行）。
*   **严重程度**：严重 (Major)
*   **改进建议**：增设专门的技术规范段落，定义 `validateRowLevelPermission` 的校验策略。建议方案：在后端使用 SQL 解析库（如 `sql-parser`）解析 SQL 生成抽象语法树（AST），提取所有涉及的表名和过滤条件，并强制检查其 `WHERE` 子句中是否包含 `user_id = operatorId`；或提供详细的校验伪代码以指导开发。

### 问题 3：`life_plans`（生活方案）和 `punch_in`（打卡记录）表的 plan_type 约束存在逻辑矛盾
*   **问题描述**：在 SQLite DDL 物理建模中，`life_plans` 表的方案类型约束为：
    `plan_type TEXT NOT NULL CHECK(plan_type IN ('diet', 'exercise', 'other'))`
    而 `punch_in` 表的打卡类型约束为：
    `punch_type TEXT NOT NULL CHECK(punch_type IN ('diet', 'exercise'))`
    由于打卡记录通过 `plan_id` 外键与具体方案关联。如果系统生成了 `'other'`（其他）类型的方案项（如作息管理或心理调节），当用户在前端对该项进行打卡时，要么无法传递正确的打卡类型，要么落库时由于 `'other'` 不符合 `punch_in` 的约束而触发 SQLite 写入异常。这导致 `life_plans` 中的 `'other'` 方案项在打卡业务中成为摆设。
*   **所在位置**：第 2.2 节完整 DDL 语句中相关表定义（第 857 行与第 885 行）。
*   **严重程度**：一般 (Medium)
*   **改进建议**：统一 `punch_in` 表和 `life_plans` 表的分类维度，将 `punch_in.punch_type` 的 CHECK 约束扩充为 `'diet', 'exercise', 'other'`；或者在文档中明确说明 `'other'` 类型的方案仅供展示，不提供打卡按钮及打卡记录生成。

### 问题 4：医学免责声明确认弹窗（`showDisclaimer`）仅定义了方法，在用户交互流程中完全未被调用
*   **问题描述**：需求文档（第 4.11 节）明确要求，用户在首次访问任何 AI 功能入口（如医师对话、风险预测提交、生活方案生成等）前，前端必须弹出医学免责声明确认弹窗，经用户同意后方可继续。在详细设计中，虽然在 `useUI.ts` 工具库中定义了弹窗函数 `showDisclaimer()`（第 3792 行）和状态检查函数 `hasAcceptedDisclaimer()`（第 3802 行），但该函数在前端路由守卫（`router/index.ts`）以及各相关页面组件（`DoctorChatView.vue`, `Risk.vue`, `LifePlan.vue` 等）的初始化流程中均**未被实际调用**。这导致合规性控制流悬空，用户可以直接绕过免责声明使用 AI 功能。
*   **所在位置**：第 4.4.4 节 `useUI.ts` 声明（第 3792 行）与第 1.6.2 节路由守卫代码块。
*   **严重程度**：一般 (Medium)
*   **改进建议**：
    1. 在 Vue Router 全局前置守卫（`router/index.ts`）中增加免责拦截逻辑，或在涉及 AI 功能的页面组件挂载时（`onMounted`）调用 `hasAcceptedDisclaimer()` 进行判定。
    2. 若未同意，调用 `showDisclaimer()` 弹窗。若用户拒绝，重定向返回上一页；若同意，将 `'disclaimer_accepted' = 'true'` 写入 `localStorage` 以跨标签页持久化。

### 问题 5：Dify 风险预测工作流输出结构与后端数据库提取查询存在数据契约不一致
*   **问题描述**：在第 5.2.1 节中，定义了 `diabetes-risk-prediction` 工作流的输出结构包含：`risk_score`、`risk_level` (英文值)、`risk_level_detail` (中文描述)、`diabetes_type` (英文值)、`suggestions` (数组)。
    但是，第 3.2.8 节中，用户历史记录的 SQLite 查询语句中，却是直接从 `result` 字段（直接存储了 Dify 返回的 JSON 对象）中提取 `risk_level_label` 和 `matched_diabetes_type`：
    ```sql
    json_extract(result, '$.risk_level_label') AS risk_level_label,
    json_extract(result, '$.matched_diabetes_type') AS matched_diabetes_type,
    ```
    如果 Dify 返回的 JSON 中根本不存在 `risk_level_label` 和 `matched_diabetes_type` 字段，直接通过 SQL `json_extract` 查询出的数据结果将全部为 `NULL`，导致前端历史页面无法正常渲染，出现严重的契约断裂。
*   **所在位置**：第 5.2.1 节输出结构定义（第 4106 行）与第 3.2.8 节 SQLite 查询设计（第 1579 行与第 1580 行）。
*   **严重程度**：一般 (Medium)
*   **改进建议**：修改 Dify 工作流输出结构，使其直接包含 `risk_level_label`（中文高/中/低风险）和 `matched_diabetes_type`（中文糖尿病类型）；或者在后端 Express 的预测接口路由中，对 Dify 返回的原始 JSON 键值进行映射转换（如将 `diabetes_type` 转换为 `matched_diabetes_type`），转换后再存入数据库 `result` 字段。

### 问题 6：`POST /api/punch`（记录打卡）的响应体缺失 `remarks` 字段
*   **问题描述**：打卡请求体中支持传入 `remarks`（打卡备注，如"早餐按方案执行，感觉不错"）并落库。但在第 3.2.16 节定义的成功响应体（201 Created）数据结构中，仅返回了 `id`、`plan_id`、`punch_type`、`completion_status` 和 `punch_time`，缺失了 `remarks` 字段。这与 `GET /api/punch/list` 返回的结构不一致，且会导致前端在接收打卡响应后无法更新该条打卡的备注展示。
*   **所在位置**：第 3.2.16 节 `POST /api/punch` 响应体定义（第 1785-1791 行）。
*   **严重程度**：轻微 (Minor)
*   **改进建议**：在 `POST /api/punch` 成功响应的 `data` 对象中补齐 `remarks` 字段返回。

### 问题 7：`AiChatDialog.vue`（AI 智能助手）组件的 DOM 结构缺失医学免责提示元素
*   **问题描述**：需求文档第 4.11 节规定："所有 AI 生成内容（...、AI 助手对话消息、...）底部固定展示免责提示文案"。虽然在 `DoctorChatView.vue`、`LifePlan.vue` 和 `HealthAdvice.vue` 的 DOM 树设计中都包含 `<p class="disclaimer-text">`，但 AI 智能助手的核心悬浮窗页面 `AiChatDialog.vue` 的 DOM 树中并未设计任何免责提示文案的存放节点（无论是在对话消息气泡底部还是弹窗底部）。这遗漏了合规交互节点。
*   **所在位置**：第 4.1.1 节 `AiChatDialog.vue` 组件 DOM 树结构（第 2705-2714 行）。
*   **严重程度**：轻微 (Minor)
*   **改进建议**：在 `AiChatDialog.vue` 的对话输入区域上方或消息列表底部，增加渲染免责提示文本的 `<p class="disclaimer-text">` 元素节点。
