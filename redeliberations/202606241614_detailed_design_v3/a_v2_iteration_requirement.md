根据以下审查结果，迭代上一轮的产出，形成新版的文件，从而更好地满足用户需求。

## 当前审查结果
### 问题一：数据库 CHECK 约束及 API 字段中的枚举值定义存在严重偏差（中文值 vs 英文值）
- **严重程度**：致命 (Critical)
- **改进建议**：
  1. 将 DDL 中的所有中文 CHECK 约束修改为符合需求文档定义的英文小写枚举值。例如，将 `gender TEXT NOT NULL CHECK(gender IN ('男', '女'))` 修改为 `gender TEXT NOT NULL CHECK(gender IN ('male', 'female'))`。
  2. 修改数据字典中的“值/约束”描述，使之与英文枚举值对齐。
  3. 在 API 接口规格和 TypeScript 类型中，要么将字段值调整为英文，要么在 Express 代理层设计明确的“中英双向映射转换层”（详细设计中必须体现该映射层的机制、职责以及转换函数设计）。

### 问题二：`user_risk_info` 表设计中缺失关键字段 `diabetes_history`（糖尿病病史）
- **严重程度**：高 (Major)
- **改进建议**：
  1. 在 `user_risk_info` 表的 DDL、数据字典以及 ER 图中，补充 `diabetes_history` 字段的定义：`diabetes_history TEXT NOT NULL CHECK(diabetes_history IN ('healthy', 'prediabetes', 'diagnosed'))`。
  2. 并在落库的逻辑流程与映射中增加该字段的处理。

### 问题三：`life_plans` 表设计中缺失关键字段 `is_active`（是否活跃）
- **严重程度**：高 (Major)
- **改进建议**：
  1. 在 `life_plans` 表的 DDL、数据字典和 ER 图中，补充 `is_active` 字段的定义：`is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1))`（在 SQLite 中使用 0 和 1 代表布尔状态）。
  2. 在 `GET /api/plan/current` 的 DDL/查询设计中，明确加入 `is_active = 1` 的过滤条件。

### 问题四：前端路由结构与组件划分不符合需求规范（医师咨询与对话组件未拆分）
- **严重程度**：中 (Medium)
- **改进建议**：
  1. 在目录结构中，将 `Consultation.vue` 拆分为 `Consultation.vue` (医生列表) 和 `DoctorChatView.vue` (医生对话)。
  2. 更新 1.6.1 节路由表，将 `/consultation` 映射到 `Consultation.vue`，将 `/consultation/doctor/:id` 映射到 `DoctorChatView.vue`。
  3. 相应拆分或调整 4.3 节的 JS 逻辑流程图。

### 问题五：个人中心子页面的嵌套路由设计不明确
- **严重程度**：低 (Minor)
- **改进建议**：
  1. 调整 1.6.1 节的路由映射表，明确以树形/嵌套结构表示这三个页面是 `/profile` 的子路由（使用 `children` 字段）。
  2. 在 `Profile.vue` 的模板结构中明确说明如何放置子路由出口 `<router-view />`，以及在子路由活跃时如何控制主菜单的显示与隐藏。

### 问题六：Dify 智能助手会话管理机制中对 `conversation_id` 的设计与需求不一致
- **严重程度**：低 (Minor)
- **改进建议**：
  1. 建议将 `chatStore` 的定义恢复为需求文档第 4.10 节规范的格式（即 `doctorConversations` Map + `assistantConversationId` 独立变量），以确保代码实现与业务需求的命名约定严格一致；或者在设计中显式地添加重构说明，指出为了统一会话管理而使用了 `conversationMap`，以消除开发人员的歧义。

### 问题七：缺少具体的数据类型与映射转换机制说明（技术决策遗漏）
- **严重程度**：高 (Major)
- **改进建议**：
  1. 明确增加“数据字段映射与国际化/映射层”的技术决策。
  2. 建议在 Express 后端设计一个专用的 `utils/mapper.ts` 转换模块，定义详细的中英枚举转换函数（如 `'男' ↔ 'male'`, `'糖尿病前期' ↔ 'prediabetes'`)，并在所有 controller 的参数校验与落库前/出库后进行显式拦截转换；或者将 API 的 JSON Schema 和 TypeScript 接口直接规范为英文，由前端在 UI 渲染层映射为中文展示。

### 问题八：前端路由结构与组件划分不符合需求规范（健康资讯与文章详情组件未拆分）
- **严重程度**：中 (Medium)
- **改进建议**：
  1. 在目录结构中，将 `News.vue` 拆分为 `NewsView.vue` (资讯列表) 和 `ArticleDetailView.vue` (资讯详情)。
  2. 更新 1.6.1 节路由映射表，将 `/news` 指向 `NewsView.vue`，`/news/article/:id` 指向 `ArticleDetailView.vue`。
  3. 相应地在第 4 节中拆分其相关的交互逻辑流程和状态说明。

### 问题九：缺少跨浏览器标签页登录态同步机制的设计
- **严重程度**：中 (Medium)
- **改进建议**：
  1. 在详细设计文档第 1.2 节和 3.7 节中，补充“跨浏览器标签页的登录态同步”设计。
  2. 明确说明在 `App.vue` 挂载时注册 `storage` 事件监听，并定义当 localStorage 中 `token` 或 `role` 发生变化时，如何调用 `authStore.clearAuth()` 或进行状态同步的协作逻辑。

### 问题十：缺少前端 SSE 流式对话连接控制与并发限制机制
- **严重程度**：高 (Major)
- **改进建议**：
  1. 在 `chatStore` 中引入 SSE 连接管理器，设计保存和中止 SSE 连接 Action。
  2. 明确使用 `AbortController` 管理每个活跃的 SSE 连接，在组件销毁或路由切换时调用其 `abort()` 方法进行连接注销。
  3. 在详细设计中补充具体的并发连接限制策略，确保同时活跃 of SSE 对话长连接数最大为 1。

### 问题十一：特定输入项缺少 0 值的有效性验证及特定的错误处理逻辑
- **严重程度**：中 (Medium)
- **改进建议**：
  1. 在 `POST /api/risk/predict` 接口规范的请求体参数描述中，明确对 `waist` 和 `systolic_bp` 补充 `0` 值校验规则。
  2. 在接口异常响应设计中，定义当上述字段为 `0` 时，返回 `400 Bad Request`，错误码为 `VALIDATION_ERROR`，提示信息为“腰围/收缩压不能为 0，请填写有效值或留空”。
  3. 在前端 `Risk.vue` 的表单校验设计中补充对应的 0 值校验拦截，防止用户输入非法的 0 值。

## 历史迭代回顾
- 已解决的问题：
  无
- 持续存在的问题：
  - 问题一：数据库 CHECK 约束及 API 字段中的枚举值定义存在严重偏差（中文值 vs 英文值）
  - 问题二：`user_risk_info` 表设计中缺失关键字段 `diabetes_history`（糖尿病病史）
  - 问题三：`life_plans` 表设计中缺失关键字段 `is_active`（是否活跃）
  - 问题四：前端路由结构与组件划分不符合需求规范（医师咨询与对话组件未拆分）
  - 问题五：个人中心子页面的嵌套路由设计不明确
  - 问题六：Dify 智能助手会话管理机制中对 `conversation_id` 的设计与需求不一致
  - 问题七：缺少具体的数据类型与映射转换机制说明（技术决策遗漏）
  - 问题八：前端路由结构与组件划分不符合需求规范（健康资讯与文章详情组件未拆分）
  - 问题九：缺少跨浏览器标签页登录态同步机制的设计
  - 问题十：缺少前端 SSE 流式对话连接控制与并发限制机制
  - 问题十一：特定输入项缺少 0 值的有效性验证及特定的错误处理逻辑
- 新发现的问题：
  无

## 上一轮产出路径
c:\Users\DELL\Desktop\qingruanProject2026\redeliberations\202606241614_detailed_design_v3\a_v1_imported.md

## 用户需求
c:\Users\DELL\Desktop\qingruanProject2026\redeliberations\202606241614_detailed_design_v3/requirement.md
