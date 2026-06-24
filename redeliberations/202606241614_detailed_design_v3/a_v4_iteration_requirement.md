根据以下审查结果，迭代上一轮的产出，形成新版的文件，从而更好地满足用户需求。

## 当前审查结果

本轮诊断（b_v3_diag_v1.md）已通过质询（LOCATED），结论可信，共识别 7 个质量问题。请逐项修复：

### 问题 1（严重）：`/api/admin/execute` 中硬编码 `operatorRole` 为 `'user'`，导致管理员 Text2SQL 功能失效
- **位置**：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5196 行）。
- **问题**：在 Dify Agent 回调认证路径中，`operatorRole` 被一律硬编码为 `'user'`。但系统设计了 `admin-manager-agent` 协助管理员通过自然语言管理 SQLite 数据库，此硬编码会使管理员的管理指令在后端被当作普通用户请求，进入行级权限校验后被拦截返回 403，彻底破坏管理员核心管理功能。
- **改进建议**：后端在通过 Dify API Key 校验后，应根据 `operatorId` 查询数据库中的用户角色（如执行 `SELECT role FROM users WHERE id = ?`），动态确定 `operatorRole` 的实际值（'admin' 或 'user'），不能一律硬编码为 'user'。

### 问题 2（严重）：`/api/admin/execute` 的安全校验函数 `validateRowLevelPermission` 缺失具体实现规范
- **位置**：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5209 行）。
- **问题**：对普通用户的 SQL 执行请求使用 `validateRowLevelPermission(sql, operatorId)` 进行行级安全校验，但整个文档未定义该函数的校验规则、实现逻辑或算法（是 AST 解析还是正则匹配）。利用 LLM 生成的任意 SQL 进行行级安全控制是高风险实现，缺失设计规格将导致后端开发人员无法着手实现，存在行级权限漏洞或返工风险。
- **改进建议**：增设专门的技术规范段落，定义 `validateRowLevelPermission` 的校验策略。建议方案：在后端使用 SQL 解析库（如 `sql-parser`）解析 SQL 生成抽象语法树（AST），提取所有涉及的表名和过滤条件，并强制检查其 `WHERE` 子句中是否包含 `user_id = operatorId`；或提供详细的校验伪代码以指导开发。

### 问题 3（一般）：`life_plans` 和 `punch_in` 表的 `plan_type`/`punch_type` 约束存在逻辑矛盾
- **位置**：第 2.2 节完整 DDL 语句中相关表定义（第 857 行与第 885 行）。
- **问题**：`life_plans.plan_type` 约束为 `('diet', 'exercise', 'other')`，而 `punch_in.punch_type` 约束仅为 `('diet', 'exercise')`。打卡记录通过 `plan_id` 外键关联方案，若系统生成 `'other'` 类型方案项，用户对该项打卡时要么无法传递正确类型，要么落库时触发 SQLite 写入异常，使 `'other'` 方案项在打卡业务中成为摆设。
- **改进建议**：统一两表分类维度，将 `punch_in.punch_type` 的 CHECK 约束扩充为 `('diet', 'exercise', 'other')`；或者在文档中明确说明 `'other'` 类型方案仅供展示，不提供打卡按钮及打卡记录生成。

### 问题 4（一般）：医学免责声明确认弹窗（`showDisclaimer`）仅定义了方法，在用户交互流程中完全未被调用
- **位置**：第 4.4.4 节 `useUI.ts` 声明（第 3792 行）与第 1.6.2 节路由守卫代码块。
- **问题**：需求文档第 4.11 节要求用户首次访问任何 AI 功能入口前必须弹出医学免责声明确认弹窗。虽然 `useUI.ts` 中定义了 `showDisclaimer()` 和 `hasAcceptedDisclaimer()`，但该函数在前端路由守卫（`router/index.ts`）以及各相关页面组件（`DoctorChatView.vue`, `Risk.vue`, `LifePlan.vue` 等）的初始化流程中均未被实际调用，导致合规性控制流悬空，用户可绕过免责声明使用 AI 功能。
- **改进建议**：
  1. 在 Vue Router 全局前置守卫（`router/index.ts`）中增加免责拦截逻辑，或在涉及 AI 功能的页面组件挂载时（`onMounted`）调用 `hasAcceptedDisclaimer()` 进行判定。
  2. 若未同意，调用 `showDisclaimer()` 弹窗。若用户拒绝，重定向返回上一页；若同意，将 `'disclaimer_accepted' = 'true'` 写入 `localStorage` 以跨标签页持久化。

### 问题 5（一般）：Dify 风险预测工作流输出结构与后端数据库提取查询存在数据契约不一致
- **位置**：第 5.2.1 节输出结构定义（第 4106 行）与第 3.2.8 节 SQLite 查询设计（第 1579 行与第 1580 行）。
- **问题**：第 5.2.1 节定义 `diabetes-risk-prediction` 工作流输出结构包含 `risk_score`、`risk_level`（英文值）、`risk_level_detail`（中文描述）、`diabetes_type`（英文值）、`suggestions`（数组）。但第 3.2.8 节的 SQLite 查询直接从 `result` 字段中提取 `risk_level_label` 和 `matched_diabetes_type`：
    ```sql
    json_extract(result, '$.risk_level_label') AS risk_level_label,
    json_extract(result, '$.matched_diabetes_type') AS matched_diabetes_type,
    ```
    若 Dify 返回的 JSON 中不存在这两个字段，SQL 查询结果将全部为 NULL，前端历史页面无法正常渲染，出现严重的契约断裂。
- **改进建议**：修改 Dify 工作流输出结构，使其直接包含 `risk_level_label`（中文高/中/低风险）和 `matched_diabetes_type`（中文糖尿病类型）；或者在后端 Express 的预测接口路由中，对 Dify 返回的原始 JSON 键值进行映射转换（如将 `diabetes_type` 转换为 `matched_diabetes_type`），转换后再存入数据库 `result` 字段。

### 问题 6（轻微）：`POST /api/punch`（记录打卡）的响应体缺失 `remarks` 字段
- **位置**：第 3.2.16 节 `POST /api/punch` 响应体定义（第 1785-1791 行）。
- **问题**：打卡请求体支持传入 `remarks`（打卡备注）并落库，但成功响应体（201 Created）仅返回 `id`、`plan_id`、`punch_type`、`completion_status` 和 `punch_time`，缺失 `remarks` 字段。这与 `GET /api/punch/list` 返回结构不一致，且会导致前端在接收打卡响应后无法更新该条打卡的备注展示。
- **改进建议**：在 `POST /api/punch` 成功响应的 `data` 对象中补齐 `remarks` 字段返回。

### 问题 7（轻微）：`AiChatDialog.vue`（AI 智能助手）组件的 DOM 结构缺失医学免责提示元素
- **位置**：第 4.1.1 节 `AiChatDialog.vue` 组件 DOM 树结构（第 2705-2714 行）。
- **问题**：需求文档第 4.11 节规定"所有 AI 生成内容底部固定展示免责提示文案"。虽然 `DoctorChatView.vue`、`LifePlan.vue` 和 `HealthAdvice.vue` 的 DOM 树设计中都包含 `<p class="disclaimer-text">`，但 AI 智能助手核心悬浮窗页面 `AiChatDialog.vue` 的 DOM 树中并未设计任何免责提示文案的存放节点，遗漏了合规交互节点。
- **改进建议**：在 `AiChatDialog.vue` 的对话输入区域上方或消息列表底部，增加渲染免责提示文本的 `<p class="disclaimer-text">` 元素节点。

## 历史迭代回顾

经与 iteration_history.md 中第 1、2 轮反馈比对，本轮（第 3 轮）7 个问题与历史反馈的关系如下：

### 已解决的问题（出现在历史反馈但当前反馈中不再提及）

第 1 轮共 11 个问题、第 2 轮共 8 个问题，绝大多数已在 v3 产出中修复，当前反馈不再提及，确认已解决，无需重复处理。主要包括：
- 第 1 轮 #1 枚举值中英文偏差：已在 v2 引入 `mapper.js` 双向转换层解决。
- 第 1 轮 #2 缺失 `diabetes_history` 字段：已补充。
- 第 1 轮 #3 缺失 `is_active` 字段：已补充。
- 第 1 轮 #4/#8 路由组件未拆分（医师列表/对话、资讯列表/详情）：已拆分。
- 第 1 轮 #5 个人中心子页面嵌套路由不明确：已明确。
- 第 1 轮 #6 `chatStore` 中 `conversation_id` 设计：已对齐需求。
- 第 1 轮 #7 缺少数据类型映射机制：已通过 `mapper.js` 落地。
- 第 1 轮 #9 跨标签页登录态同步：已补充 `storage` 事件监听。
- 第 1 轮 #10 SSE 流式连接控制与并发限制：已引入 `AbortController`。
- 第 1 轮 #11 `waist`/`systolic_bp` 0 值校验：已补充。
- 第 2 轮 #1 `timingSafeEqual` 长度校验 DoS 隐患：已修复。
- 第 2 轮 #2 `router/index.ts` 语法嵌套错误：已修复。
- 第 2 轮 #3 `mapper.js` 漏配 `punch_type` 映射：已补全。
- 第 2 轮 #5 `pregnancy` 字段类型不一致：已统一为 INTEGER + CHECK。
- 第 2 轮 #6 `seed.sql` 占位符风险：已补充提示。
- 第 2 轮 #7 布尔字段类型不统一：已统一为 INTEGER + CHECK。

注意：上述问题虽已解决，但在采用 COPY_AND_EDIT 模式定向修改时，请勿破坏这些已修复点，避免回退。

### 持续存在的问题（在多轮反馈中反复出现，需重点解决）

以下问题主题在多轮中以不同形态反复出现，属于顽疾，本轮须彻底根除：

1. **风险预测数据契约不一致（持续 2 轮）**
   - 第 2 轮 #4 指出 `POST /api/risk/predict` 响应结构与需求契约偏差（缺 `risk_level_label`、`matched_diabetes_type` 命名错误、`advice` 被错误拆分）。
   - 第 3 轮（本轮）#5 进一步发现：即便响应结构已调整，Dify 工作流输出 JSON 的键名（`risk_level`/`diabetes_type`）与后端 SQLite `json_extract` 读取的键名（`risk_level_label`/`matched_diabetes_type`）仍不匹配，导致历史记录查询全为 NULL。
   - **根因**：Dify 输出层、后端存储层、后端查询层三者之间的字段命名契约未端到端拉通。
   - **要求**：本轮须一次性打通"Dify 输出 → 后端映射/转换 → 数据库存储 → SQLite 查询提取 → 前端渲染"整条链路的字段命名，确保每一跳的键名完全一致，并在文档中以表格形式显式列出该端到端字段映射契约，避免再次以新形态复发。

2. **数据驱动接口的 SQLite 查询设计深度不足（持续 2 轮）**
   - 第 2 轮 #8 指出 `GET /api/risk/history` 和 `GET /api/punch/list` 等接口缺乏具体 SQLite 查询伪代码。
   - 第 3 轮（本轮）#5 表明：虽然 `risk/history` 的查询语句已补出（第 1579-1580 行），但补出的查询本身存在契约 bug（提取了不存在的 JSON 字段）。说明上轮仅"补了形"而未"补对质"。
   - **要求**：本轮对所有数据驱动接口的 SQLite 查询，除给出伪代码外，须确保查询字段与存储字段严格对应，并标注字段来源（来自哪个 Dify 输出键或哪个落库键），杜绝"有查询但不一致"的浅层补全。

### 新发现的问题（本轮新识别）

以下 5 个问题为本轮首次识别，历史反馈中未出现，属新增设计缺陷：

1. **`/api/admin/execute` 硬编码 `operatorRole='user'`（本轮 #1，严重）**：管理员 Text2SQL 功能被行级权限拦截而失效。需改为按 `operatorId` 动态查询用户角色。
2. **`validateRowLevelPermission` 缺失实现规范（本轮 #2，严重）**：行级权限校验函数无任何算法/伪代码定义，开发无法落地。须增设技术规范段落，定义 AST 解析校验策略。
3. **`life_plans` 与 `punch_in` 表约束逻辑矛盾（本轮 #3，一般）**：`'other'` 类型方案无法打卡。须统一 CHECK 约束或明确说明该类型不提供打卡。
4. **医学免责声明弹窗未实际调用（本轮 #4，一般）**：`showDisclaimer`/`hasAcceptedDisclaimer` 仅定义未调用，合规控制流悬空。须在路由守卫或页面 `onMounted` 中接入。
5. **`AiChatDialog.vue` DOM 缺失免责提示元素（本轮 #7，轻微）**：AI 智能助手悬浮窗无免责文案节点。须补充 `<p class="disclaimer-text">`。

此外，本轮 #6（`POST /api/punch` 响应体缺 `remarks` 字段，轻微）亦为新增问题，请一并修复。

## 上一轮产出路径

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/a_v3_tech_v2.md

## 用户需求

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/requirement.md
