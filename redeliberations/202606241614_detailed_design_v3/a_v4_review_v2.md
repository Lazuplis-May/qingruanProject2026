# 技术方案审查报告（v2）

## 审查结果

APPROVED

## 逐维度审查

### 1. 技术准确性

**[通过]** 第 7.3.4 节 `validateRowLevelPermission` 选用 `node-sql-parser`（npm 包，纯 JS 实现，支持 SQLite 方言）进行 AST 解析校验。该包真实存在（npm registry 可查），支持 `parser.astify(sql, { database: 'sqlite' })` API，适用于 Node.js 后端 SQL 安全校验场景。明确拒绝正则匹配方案的理由（无法处理子查询、别名、嵌套条件）技术合理。

**[通过]** 第 7.3.3 节 `operatorRole` 动态查询逻辑使用 `db.prepare('SELECT role FROM users WHERE id = ?').get(operatorId)` 从 `users` 表读取角色，SQL 语法与 `better-sqlite3` 同步 API 一致，`.get()` 返回单行对象，符合 better-sqlite3 9.x 文档用法。

**[通过]** 第 5.2.1.1 节端到端字段映射契约中，`json_extract(result, '$.risk_level_label')` 与 `json_extract(result, '$.matched_diabetes_type')` 的 SQLite JSON 函数语法正确，与 SQLite 3.38+ 内置 JSON1 扩展一致。`user_risk_info.result` 列定义为 TEXT 存储 JSON 字符串，`json_extract` 提取键名与后端 `risk.js` 序列化写入键名完全对应。

**[通过]** 第 2.2 节 DDL 中 `punch_in.punch_type` CHECK 约束扩充为 `('diet', 'exercise', 'other')`，与 `life_plans.plan_type` 枚举维度统一，SQLite CHECK 约束语法正确。

**[通过]** 第 1.6.2 节 Vue Router 4 `beforeEach` 守卫中 `meta.requiresDisclaimer` 路由元信息字段声明与 `RouteMeta` 接口扩展一致，`useUI()` 组合式函数在守卫内调用符合 Vue3 Composition API 规范。`SweetAlert2` 的 `Swal.fire()` 返回 `Promise<{isConfirmed: boolean}>`，与 `showDisclaimer(): Promise<boolean>` 的类型签名一致。

**[轻微]** 第 7.3.4 节校验伪代码中 `containsUserIdConstraint`、`insertContainsUserId`、`extractTableNames` 三个辅助函数仅给出文字说明而非完整实现，但已明确其遍历 AST 节点的逻辑目标（`binary_expr` 节点、`where` 节点、`from/join/into/update` 节点）和判定规则。此为编码阶段的实现细节，不影响技术决策明确性。

### 2. 完备性

**[通过]** 问题 1（严重）已修复：第 7.3.3 节 `POST /api/admin/execute` 路由处理器在 Dify Agent 回调认证路径中，通过 `SELECT role FROM users WHERE id = ?` 动态查询操作者角色，将实际值（`'admin'` 或 `'user'`）赋给 `operatorRole`，不再硬编码为 `'user'`。用户不存在时返回 403，数据流闭环完整。

**[通过]** 问题 2（严重）已修复：新增第 7.3.4 节《行级权限校验函数 validateRowLevelPermission 技术规范》，完整定义了：选型（node-sql-parser AST 解析）、表分类与校验规则表（用户私有表/公共只读表/禁止访问表三类）、校验伪代码、fail-closed 原则。开发人员可据此直接实现，不存在需要自行探索的技术方向性问题。

**[通过]** 问题 3（一般）已修复：`punch_in.punch_type` CHECK 约束已扩充为 `('diet', 'exercise', 'other')`，与 `life_plans.plan_type` 枚举维度统一。`mapper.js` 的 `punch_type` 映射字典同步新增 `'其他' ↔ 'other'`，TypeScript 类型定义 `PunchRecord`/`PunchCreateRequest`/`PunchListParams` 的联合类型同步更新。数据流闭环完整。

**[通过]** 问题 4（一般）已修复：第 1.6.2 节 Vue Router 全局前置守卫新增步骤 5，对标记 `meta.requiresDisclaimer: true` 的路由（医师对话/生活方案/风险预测/健康建议）调用 `hasAcceptedDisclaimer()` 判定，未同意时调用 `showDisclaimer()` 弹窗，同意后写入 `localStorage['disclaimer_accepted'] = 'true'` 跨标签页持久化，拒绝则 `next(false)` 回退导航。同时第 4.4.4 节明确 `chatStore.toggleFab()` 打开 AI 助手悬浮窗前需补充调用 `hasAcceptedDisclaimer()`，覆盖了非路由入口的合规控制流。合规控制流不再悬空。

**[通过]** 问题 5（一般，顽疾）已修复：新增第 5.2.1.1 节《风险预测端到端字段映射契约》表格，显式列出"Dify 工作流输出键 → 后端 risk.js 映射处理 → user_risk_info.result 存储键 → SQLite json_extract 提取键 → 3.2.7 响应键 → 3.2.8 响应键 → 前端渲染键"七列端到端字段映射。Dify 工作流输出层新增 `risk_level_label`（中文）与 `matched_diabetes_type`（中文）字段，后端 `risk.js` 将 `risk_level_detail` 与 `suggestions` 拼接为 `advice` 后连同四个核心字段序列化为 JSON 写入 `result` 列，SQLite `json_extract` 提取键名与存储键名完全一致。第 3.2.8 节 SQLite 查询补充字段来源标注。顽疾已端到端打通。

**[通过]** 问题 6（轻微）已修复：第 3.2.16 节 `POST /api/punch` 成功响应体（201 Created）的 `data` 对象中已补齐 `remarks` 字段，与请求体及 `GET /api/punch/list` 返回结构一致。

**[通过]** 问题 7（轻微）已修复：第 4.1.1 节 `AiChatDialog.vue` 组件 DOM 树中，在消息列表区域与对话输入区域之间新增 `<p class="disclaimer-text">` 元素节点，与 `DoctorChatView.vue`、`LifePlan.vue`、`HealthAdvice.vue`、`NewsView.vue`、`ArticleDetailView.vue`、`Risk.vue` 的免责节点设计保持一致。

**[通过]** 历史轮次已解决问题（第 1 轮 11 项、第 2 轮 8 项）经抽样核验未发生回退：`mapper.js` 双向转换层、`diabetes_history`/`is_active` 字段、路由组件拆分、`chatStore` 的 `conversation_id` 设计、`AbortController` SSE 控制、`timingSafeEqual` 长度校验、`pregnancy` 字段 INTEGER+CHECK、布尔字段统一等均已保留。

### 3. 可操作性

**[通过]** 第 7.3.3 节 `POST /api/admin/execute` 路由处理器伪代码完整覆盖：认证路径分支（Dify 回调 vs 浏览器直连）、身份解析、权限校验（admin 全权限 / user 行级约束）、SQL 白名单校验（仅 SELECT/INSERT/UPDATE/DELETE）、多语句拦截、执行与日志记录。每一步均有明确结论，实现者可据此直接编码。

**[通过]** 第 7.3.4 节 `validateRowLevelPermission` 技术规范给出表分类校验规则表（三类表 × 四种 SQL 操作类型），明确每种组合的允许条件。校验伪代码涵盖 AST 解析、表名收集、禁止表拦截、公共只读表 SELECT 限制、用户私有表 user_id 约束检查（SELECT/UPDATE/DELETE 的 WHERE 子句、INSERT 的 VALUES 列表）全部分支。辅助函数职责以注释形式明确，开发路径清晰。

**[通过]** 第 5.2.1.1 节端到端字段映射契约表格以七列结构显式列出每一跳的字段命名，后端映射逻辑以五步流程说明（调用 Dify → 构造 result 对象 → JSON.stringify 写入 → 响应 3.2.7 解析返回 → 响应 3.2.8 SQLite json_extract 提取），实现者可逐跳对照实现，杜绝键名漂移。

**[通过]** 第 1.6.2 节路由守卫代码块完整给出 `beforeEach` 五步逻辑（公开路由放行 → Token 检查 → 管理员强制改密 → 管理员角色校验 → 免责声明拦截），每步均有明确的 `next()` 调用与重定向路径。免责声明拦截流程另以六步文字说明补充，包含 AI 助手悬浮窗的非路由入口处理。

**[通过]** 第 1.8.1 节 `mapper.js` 转换映射字典完整覆盖 `gender`、`family_history`、`diabetes_history`、`diabetes_type`、`plan_type`、`punch_type`、`completion_status` 七个枚举字段的中英双向映射，与 DDL CHECK 约束的英文值集合严格对应。第 1.8.3 节控制器拦截策略明确各路由的转换时机（落库前 toEnglish / 出库前 toChinese）。

**[通过]** 第 2.2 节 DDL 与第 2.5 节数据字典字段定义一致，第 3.2.x 节 API 接口请求/响应体与第 3.8.x 节 TypeScript 类型定义对应，数据流形成完整闭环。

## 修改要求

无（本审查结果为 APPROVED）。
