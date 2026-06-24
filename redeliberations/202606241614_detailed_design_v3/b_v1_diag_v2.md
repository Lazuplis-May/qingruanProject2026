# 糖尿病预治智能助手 —— 详细设计文档（a_v1_imported.md）质量审查诊断报告

本审查报告针对待审查文件 [a_v1_imported.md](file:///c:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/a_v1_imported.md) 进行了全面评估，重点审查了其对用户需求文档 [requirement.md](file:///c:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/requirement.md) 的响应度、整体深度、完整性、逻辑自洽性及工程可实施性。现将发现的质量问题总结如下。

---

### 问题一：数据库 CHECK 约束及 API 字段中的枚举值定义存在严重偏差（中文值 vs 英文值）
* **问题描述**：
  用户需求书第 5 节“数据需求”明确规定，数据库中多个表的枚举型字段应存储英文小写字符串（如 `user_risk_info` 的 `gender` 字段约束为 `'male'/'female'`，`family_history` 约束为 `'yes'/'no'`，`diabetes_history` 约束为 `'healthy'/'prediabetes'/'diagnosed'`，`diabetes_type` 约束为 `'type1'/'type2'/'gestational'/'other'`；`life_plans` 的 `plan_type` 约束为 `'diet'/'exercise'/'other'`；`punch_in` 的 `punch_type` 约束为 `'diet'/'exercise'`，`completion_status` 约束为 `'completed'/'uncompleted'`）。
  然而，详细设计文档中所有的 DDL 语句、数据字典、TypeScript 类型定义以及 API 的 JSON Schema，全部直接使用了中文值（如 `'男'/'女'`、`'有'/'无'`、`'健康'/'糖尿病前期'/'已确诊'`、`'饮食'/'运动'/'其他'`、`'已完成'/'未完成'`)。这导致详细设计与需求文档发生根本性偏离。
* **所在位置**：
  * 第 2.2 节 完整 DDL 语句中 `users`、`user_risk_info`、`life_plans`、`punch_in` 表的创建脚本（第 709, 712, 727, 754, 755 行等）
  * 第 2.5 节 数据字典各表说明中的约束定义（第 1031, 1034, 1049, 1076, 1077 行等）
  * 第 3.2 节 各 API 接口请求与响应规格（如 3.2.7、3.2.16 等，第 1365-1375 行、第 1600-1605 行等）
  * 第 3.8 节 TypeScript 类型定义（如 `RiskPredictRequest` 等，第 2290-2300 行、第 2348-2349 行等）
* **严重程度**：致命（Critical）
* **改进建议**：
  * 将 DDL 中的所有中文 CHECK 约束修改为符合需求文档定义的英文小写枚举值。例如，将 `gender TEXT NOT NULL CHECK(gender IN ('男', '女'))` 修改为 `gender TEXT NOT NULL CHECK(gender IN ('male', 'female'))`。
  * 修改数据字典中的“值/约束”描述，使之与英文枚举值对齐。
  * 在 API 接口规格和 TypeScript 类型中，要么将字段值调整为英文，要么在 Express 代理层设计明确的“中英双向映射转换层”（详细设计中必须体现该映射层的机制、职责以及转换函数设计）。

---

### 问题二：`user_risk_info` 表设计中缺失关键字段 `diabetes_history`（糖尿病病史）
* **问题描述**：
  用户需求书第 5 节明确要求，用户风险信息表 `user_risk_info` 必须包含“糖尿病病史（`diabetes_history`，TEXT，CHECK(...)）”字段，用于与 `POST /api/risk/predict` 的请求参数相对应并实现持久化。然而，在详细设计的 DDL 语句（2.2 节）和数据字典（2.5 节）中，`user_risk_info` 表完全遗漏了 `diabetes_history` 字段，且 ER 图（2.1 节）中也没有该字段。这导致前端提交的病史数据无法在数据库中被持久化。
* **所在位置**：
  * 第 2.1 节 完整 ER 图
  * 第 2.2 节 完整 DDL 语句中的 `user_risk_info` 创建脚本（第 705-721 行）
  * 第 2.5 节 数据字典中的 `user_risk_info` 字段表（第 1024-1042 行）
* **严重程度**：高（Major）
* **改进建议**：
  * 在 `user_risk_info` 表的 DDL、数据字典以及 ER 图中，补充 `diabetes_history` 字段的定义：`diabetes_history TEXT NOT NULL CHECK(diabetes_history IN ('healthy', 'prediabetes', 'diagnosed'))`。
  * 并在落库的逻辑流程与映射中增加该字段的处理。

---

### 问题三：`life_plans` 表设计中缺失关键字段 `is_active`（是否活跃）
* **问题描述**：
  用户需求书第 4.5 节“方案调整”和第 5 节“数据需求”明确规定，为保证打卡记录的外键关联完整性，避免因物理删除旧方案导致打卡数据悬空，`life_plans` 表必须引入“是否活跃（`is_active`，BOOLEAN，默认 TRUE）”字段，并在方案调整时采用“逻辑过期”机制。然而，在详细设计的 DDL（2.2 节）和数据字典（2.5 节）中，`life_plans` 表完全遗漏了该字段。这会导致后端在执行方案调整逻辑时，如果直接物理删除旧记录，会由于外键关联导致 `punch_in` 表出现垃圾数据或级联删除，从而破坏用户历史打卡数据的可追溯性，使得打卡依从性分析彻底失效。
* **所在位置**：
  * 第 2.1 节 完整 ER 图中的 `life_plans` 属性
  * 第 2.2 节 完整 DDL 语句中的 `life_plans` 表定义（第 724-735 行）
  * 第 2.5 节 数据字典中的 `life_plans` 字段表（第 1043-1056 行）
* **严重程度**：高（Major）
* **改进建议**：
  * 在 `life_plans` 表的 DDL、数据字典和 ER 图中，补充 `is_active` 字段的定义：`is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1))`（在 SQLite 中使用 0 和 1 代表布尔状态）。
  * 在 `GET /api/plan/current` 的 DDL/查询设计中，明确加入 `is_active = 1` 的过滤条件。

---

### 问题四：前端路由结构与组件划分不符合需求规范（医师咨询与对话组件未拆分）
* **问题描述**：
  用户需求书第 1.1 节明确规定，医师咨询（医生列表）路由 `/consultation` 应映射至页面组件 `views/ConsultationView.vue`；而医师咨询（对话）路由 `/consultation/doctor/:id` 应映射至页面组件 `views/DoctorChatView.vue`。然而，详细设计文档在 1.4 节（目录树）中仅规划了单独的 `Consultation.vue` 组件，并且在 1.6.1 节的路由映射表中，将 `/consultation` 和 `/consultation/doctor/:id` 两个路由均指向了同一个 `Consultation.vue` 页面组件。
* **所在位置**：
  * 第 1.4 节 模块划分与依赖关系目录树（第 165 行）
  * 第 1.6.1 节 Vue Router 4 路由映射表（第 408-409 行）
  * 第 4.3 节 中的 `Consultation.vue` 流程图（第 3033-3057 行）
* **严重程度**：中（Medium）
* **改进建议**：
  * 在目录结构中，将 `Consultation.vue` 拆分为 `Consultation.vue` (医生列表) 和 `DoctorChatView.vue` (医生对话)。
  * 更新 1.6.1 节路由表，将 `/consultation` 映射到 `Consultation.vue`，将 `/consultation/doctor/:id` 映射到 `DoctorChatView.vue`。
  * 相应拆分或调整 4.3 节的 JS 逻辑流程图。

---

### 问题五：个人中心子页面的嵌套路由设计不明确
* **问题描述**：
  用户需求书第 1.1 节的路由配置结构（代码块）明确展示，糖尿病风险预测（`/profile/risk`）、打卡记录与分析（`/profile/punch`）、健康建议（`/profile/advice`）应当是个人中心（`/profile`）的**嵌套子路由（`children` 数组）**，共享同一个父路由组件的上下文。然而，在详细设计的 1.6.1 节路由映射表中，这三者被扁平地定义为独立的一级路由（指向各自独立的 lazy import 页面组件，例如 `() => import('@/views/Risk.vue')` ）。在 4.3 节的 `Profile.vue` 和各子组件逻辑流程图中，也没有明确阐明父子路由的嵌套布局结构和 `<router-view />` 的渲染逻辑。
* **所在位置**：
  * 第 1.6.1 节 Vue Router 4 路由映射表（第 413-416 行）
  * 第 4.3 节 中 `Profile.vue` 相关流程图与文字（第 3175-3208 行）
* **严重程度**：低（Minor）
* **改进建议**：
  * 调整 1.6.1 节的路由映射表，明确以树形/嵌套结构表示这三个页面是 `/profile` 的子路由（使用 `children` 字段）。
  * 在 `Profile.vue` 的模板结构中明确说明如何放置子路由出口 `<router-view />`，以及在子路由活跃时如何控制主菜单的显示与隐藏。

---

### 问题六：Dify 智能助手会话管理机制中对 `conversation_id` 的设计与需求不一致
* **问题描述**：
  用户需求书第 4.10 节中明确定义了 `chatStore`，其中定义：对于医师对话，按医生 ID 维护会话：`doctorConversations = ref<Map<number, string>>()`；对于 AI 智能助手，使用独立变量 `assistantConversationId = ref<string | null>()`。但在详细设计的 3.7 节 `chatStore` 定义中，将其重构为了一个通用的 `conversationMap: Map<string, string>`。虽然在功能上类似，但这种接口定义的差异会导致前端状态定义与用户需求规范不一致，容易造成开发者的困惑。
* **所在位置**：
  * 第 3.7 节 `chatStore` 接口定义中（第 2078-2080 行）
  * 第 4.3 节 中的相关 Store 读写流程图（如 Consultation.vue、Admin.vue 流程图）
* **严重程度**：低（Minor）
* **改进建议**：
  * 建议将 `chatStore` 的定义恢复为需求文档第 4.10 节规范的格式（即 `doctorConversations` Map + `assistantConversationId` 独立变量），以确保代码实现与业务需求的命名约定严格一致；或者在设计中显式地添加重构说明，指出为了统一会话管理而使用了 `conversationMap`，以消除开发人员的歧义。

---

### 问题七：缺少具体的数据类型与映射转换机制说明（技术决策遗漏）
* **问题描述**：
  详细设计中提到 Express 代理层负责前端字段名与 Dify API 参数名的映射，并且提到数据库将存储英文的枚举值。但由于 API 接口规格和 JSON Schema 仍大量使用中文枚举，详细设计文档遗漏了一个关键的技术决策——“数据转换与映射机制”。目前未指定是“前端负责将中文转化为英文后提交给 API，且 API 响应返回英文，由前端负责中文本地化渲染”，还是“API 接口接受中文，由后端在 controller 层将中文映射为英文以符合 SQLite DDL 的 CHECK 约束”。由于缺乏这一映射层的具体定义，开发人员在编写前后台代码时无法确定如何进行数据传输与校验，极易导致接口调用及落库出错。
* **所在位置**：
  * 第 3.6 节 Express 代理层请求参数映射表（第 2030-2043 行）
  * 第 5 节 数据需求与落库路径部分
* **严重程度**：高（Major）
* **改进建议**：
  * 明确增加“数据字段映射与国际化/映射层”的技术决策。
  * 建议在 Express 后端设计一个专用的 `utils/mapper.ts` 转换模块，定义详细的中英枚举转换函数（如 `'男' ↔ 'male'`, `'糖尿病前期' ↔ 'prediabetes'`)，并在所有 controller 的参数校验与落库前/出库后进行显式拦截转换；或者将 API 的 JSON Schema 和 TypeScript 接口直接规范为英文，由前端在 UI 渲染层映射为中文展示。

---

### 问题八：前端路由结构与组件划分不符合需求规范（健康资讯与文章详情组件未拆分）
* **问题描述**：
  用户需求书第 1.1 节明确规定，健康资讯（列表）路由 `/news` 应映射至页面组件 `views/NewsView.vue`；健康资讯（详情）路由 `/news/article/:id` 应映射至页面组件 `views/ArticleDetailView.vue`。
  然而，详细设计文档在 1.4 节（目录树）中仅规划了 `News.vue` 组件，并且在 1.6.1 节的前端路由映射表中，将 `/news` 和 `/news/article/:id` 两个不同的路由均指向了同一个组件 `News.vue`。这与需求不符，且将列表和详情混合在同一个组件中会增加组件的复杂度和耦合度。
* **所在位置**：
  * 第 1.4 节 模块划分与依赖关系目录树（第 167 行）
  * 第 1.6.1 节 Vue Router 4 路由映射表（第 411-412 行）
  * 第 4.1.5 节 `News.vue` (健康资讯) 相关描述与流程图（第 2630-2633 行、第 3118 行等）
* **严重程度**：中（Medium）
* **改进建议**：
  * 在目录结构中，将 `News.vue` 拆分为 `NewsView.vue` (资讯列表) 和 `ArticleDetailView.vue` (资讯详情)。
  * 更新 1.6.1 节路由映射表，将 `/news` 指向 `NewsView.vue`，`/news/article/:id` 指向 `ArticleDetailView.vue`。
  * 相应地在第 4 节中拆分其相关的交互逻辑流程和状态说明。

---

### 问题九：缺少跨浏览器标签页登录态同步机制的设计
* **问题描述**：
  用户需求书第 4.10 节中明确要求，当用户在同一浏览器中打开多个标签页时，系统应在应用初始化时监听 `window.addEventListener('storage', ...)` 事件，以实现跨浏览器标签页登录态同步（例如：检测到 `token` 被移除或修改时，同步更新 Pinia authStore 的状态并触发登出或更新）。
  然而，详细设计文档中完全未提及此监听机制，也没有规划在应用根组件（如 `App.vue`）或 `authStore` 中注册和处理 `storage` 事件的逻辑。这将导致多标签页下的登录状态不一致，属于非功能性交互设计的关键缺失。
* **所在位置**：
  * 第 1.2 节 Vue3 SPA 前端架构说明
  * 第 3.7 节 `authStore` 设计与接口定义说明（第 2054-2076 行）
* **严重程度**：中（Medium）
* **改进建议**：
  * 在详细设计文档第 1.2 节和 3.7 节中，补充“跨浏览器标签页的登录态同步”设计。
  * 明确说明在 `App.vue` 挂载时注册 `storage` 事件监听，并定义当 localStorage 中 `token` 或 `role` 发生变化时，如何调用 `authStore.clearAuth()` 或进行状态同步的协作逻辑。

---

### 问题十：缺少前端 SSE 流式对话连接控制与并发限制机制
* **问题描述**：
  用户需求书第 4.2 节和第 4.8 节明确要求，用户与医生建立 SSE 流式对话期间，若切换到其他对话界面，前端应立即关闭前一个连接（调用 `AbortController.abort()`），且应限制并发 SSE 连接上限为 1，以节约网络资源。
  然而，详细设计文档中仅提及了使用 `chatStore` 管理会话 ID 和使用 API 进行 SSE 代理，完全没有包含在页面切换或组件卸载时进行 SSE 连接清理和使用 `AbortController` 进行取消操作的技术设计。这可能导致切换医生后后台仍维持无效的 SSE 长连接，占用浏览器及服务端的连接资源。
* **所在位置**：
  * 第 1.2 节 Vue3 SPA 前端架构设计
  * 第 3.7 节 `chatStore` 接口及方法定义（第 2078-2101 行）
  * 第 4.3 节 医师咨询与对话组件流程图
* **严重程度**：高（Major）
* **改进建议**：
  * 在 `chatStore` 中引入 SSE 连接管理器，设计保存和中止 SSE 连接 the action。
  * 明确使用 `AbortController` 管理每个活跃的 SSE 连接，在组件销毁或路由切换时调用其 `abort()` 方法进行连接注销。
  * 在详细设计中补充具体的并发连接限制策略，确保同时活跃的 SSE 对话长连接数最大为 1。

---

### 问题十一：特定输入项缺少 0 值的有效性验证及特定的错误处理逻辑
* **问题描述**：
  用户需求书第 4.4 节关于风险预测的数据采集要求中规定，若用户未填写腰围（`waist`）或收缩压（`systolic_bp`），系统会执行自动估算；但如果字段传入值为 `0` 时，应视为无效输入，由前端进行拦截，且后端接口 `POST /api/risk/predict` 在接收到参数值为 `0` 时必须校验失败并返回特定的 `VALIDATION_ERROR` 错误，提示相应信息。
  然而，详细设计文档 3.2.7 节（`POST /api/risk/predict` 的请求规格）中并未体现对于 `0` 值的校验约束和返回 `VALIDATION_ERROR` 错误的设计，且前端相关表单校验部分也缺少对 0 值的拦截规则。这可能导致非法的 0 值传入后台参与医学逻辑计算，影响系统的健壮性。
* **所在位置**：
  * 第 3.2.7 节 `POST /api/risk/predict` 接口规格（第 1361-1377 行）
  * 第 4.3 节 风险预测组件（Risk.vue）的表单验证逻辑设计
* **严重程度**：中（Medium）
* **改进建议**：
  * 在 `POST /api/risk/predict` 接口规范的请求体参数描述中，明确对 `waist` 和 `systolic_bp` 补充 `0` 值校验规则。
  * 在接口异常响应设计中，定义当上述字段为 `0` 时，返回 `400 Bad Request`，错误码为 `VALIDATION_ERROR`，提示信息为“腰围/收缩压不能为 0，请填写有效值或留空”。
  * 在前端 `Risk.vue` 的表单校验设计中补充对应的 0 值校验拦截，防止用户输入非法的 0 值。

---

## 修订说明（v2）

| 质询意见 | 回应 |
|---------|------|
| 资讯组件拆分遗漏：详细设计中健康资讯列表路由 `/news` 和详情路由 `/news/article/:id` 被合并指向单一组件 `News.vue`，需拆分。 | 已确认该质询意见成立。已在审查报告中新增「问题八」，指出该项路由组件合并偏离了需求书的要求，并给出了将其拆分为 `NewsView.vue` 和 `ArticleDetailView.vue` 的改进建议。 |
| 跨标签页登录态同步遗漏：需求书第 4.10 节要求在详细设计中体现“监听 `window.addEventListener('storage', ...)` 以实现跨浏览器标签页登录态同步”，但详细设计完全未提及。 | 已确认该质询意见成立。已在审查报告中新增「问题九」，指出详细设计缺失了多标签页下通过 `storage` 事件同步登录态的关键逻辑，并给出了在 `App.vue` 和 `authStore` 中补充该机制的建议。 |
| SSE 连接控制与并发限制遗漏：需求书第 4.2 节要求前端切换医生时调用 `AbortController.abort()` 并限制并发 SSE 连接上限为 1，但设计文档中未涉及。 | 已确认该质询意见成立。已在审查报告中新增「问题十」，指出详细设计遗漏了 SSE 连接的注销与并发控制方案，容易导致资源浪费，并给出了利用 `AbortController` 进行连接清理的改进建议。 |
| 特定输入验证规则遗漏：需求书第 4.4 节要求对 `waist/systolic_bp` 值为 0 时校验并抛出 `VALIDATION_ERROR`，但设计文档中对此类 0 值的参数校验规范缺失。 | 已确认该质询意见成立。已在审查报告中新增「问题十一」，指出对 0 值（腰围与收缩压）特定业务验证规则的遗漏可能影响系统健壮性，并给出了在前后端分别补充校验逻辑与 `VALIDATION_ERROR` 响应的建议。 |
