# 技术方案审查报告（v1）

## 审查结果

APPROVED

## 逐维度审查

### 1. 技术准确性

**[通过]** 技术选型表（1.3 节）中所列全部库/框架均真实存在且版本适用：Vue 3 + TypeScript 5 + Vite 5 + Vue Router 4 + Pinia 2 + Axios + Vant 4 + DOMPurify 3 + marked.js 12 + SweetAlert2 11 + better-sqlite3 9 + Express 4 + jsonwebtoken 9 + bcryptjs 2 + multer 1 + node-sql-parser。Vant 4 按需引入方案（unplugin-vue-components + @vant/auto-import-resolver）正确。

**[通过]** Vant 4 组件能力描述准确：Tabbar、ActionSheet、DatetimePicker、PullRefresh、Toast、Dialog 均为 Vant 4 提供的移动端特有交互组件；CSS 变量映射表（4.5.1 节）将 `--van-primary-color` 等主题变量映射到自定义 CSS 变量，符合 Vant 4 主题定制机制。

**[通过]** Dify SSE 事件格式（3.3 节）符合 Dify 对话 API 规范：message/message_end/error/workflow_started/workflow_finished/agent_thought 事件类型定义合理；`{{user}}` 变量透传能力明确标注为"未经验证假设"，5.5.1 节定义了门禁验证任务方法、标准和时机，5.5.2 节提供了备选方案（session_id 映射表）。

**[通过]** Pinia + pinia-plugin-persistedstate 持久化机制描述正确；chatStore 的 Map 类型在持久化插件中可能不被正确序列化的问题已被规避——3.7 节明确采用手动 localStorage 读写策略管理 doctorConversations。

**[通过]** 语言特性用法在能力范围内：TypeScript 类型定义完整（3.8 节覆盖 30+ interface/type），Vue 3 Composition API + `<script setup>` 语法正确，AbortController 用于 SSE 连接管理是浏览器原生 API，crypto.timingSafeEqual + SHA-256 哈希防时序攻击方案正确（7.3.2 节）。

**[通过]** node-sql-parser 选型合理：纯 JS 实现，支持 SQLite 方言，将 SQL 解析为 AST 进行结构化校验，规避了正则匹配无法处理子查询、别名、嵌套条件的局限性。

### 2. 完备性

**[通过]** 迭代需求提出的 21 个问题（7 严重 / 10 一般 / 4 轻微）均已逐项修复，验证如下：

- 严重问题 1（useSSE.ts 401 处理）：4.4.2 节 streamRequest 函数新增 `response.status === 401` 检查分支，调用 `authStore.clearAuth()` + Toast + `callbacks.onError`，与 4.4.1 节 Axios 拦截器 401 逻辑等价；4.3 节 DoctorChatView.vue / Admin.vue / AiChatDialog.vue 流程图均新增 401 分支。
- 严重问题 2（chatStore 方法命名）：3.7 节方法名对齐需求 4.10 节规范（`setDoctorConversation`/`getDoctorConversation`/`setAssistantConversation`），返回类型 `null`（非 `undefined`），新增 `clearDoctorConversation(doctorId)` 和 `clearAssistantConversation()` 方法。
- 严重问题 3（0 值校验 HTTP 状态码）：3.2.7 节错误响应状态码从 400 改为 422，与需求 6.13 节 `VALIDATION_ERROR → 422` 一致。
- 严重问题 4（is_collected 字段）：3.2.20 节响应新增 `is_collected: boolean`；3.2.21 节 generate 响应同步新增；3.8.3 节 ArticleDetail 接口新增；4.3 节 ArticleDetailView.vue 流程图删除并行请求分支，改为直接读取响应字段。
- 严重问题 5（tags/summary 字段）：2.2 节 DDL 新增 `tags TEXT DEFAULT '[]'` 和 `summary TEXT DEFAULT ''` 列；3.2.19 节响应新增字段；3.8.3 节 Article 接口同步；字段名 `created_at`/`views` 与需求 `publish_time`/`read_count` 的映射关系已在 2.5 节明确说明。
- 严重问题 6（Dify Agent 工具定义）：5.2.5 节 diabetes-assistant-agent 定义 8 专用工具（query_user_profile/query_risk_history/query_punch_records/query_life_plans/query_health_advice/write_health_advice/update_user_profile/knowledge_search）+ execute_SQL 兜底；5.2.6 节 admin-manager-agent 定义 5 专用工具（query_table/insert_record/update_record/delete_record/get_table_schema）+ execute_SQL 兜底；7.3.4 节补充 life_advice 表写入约束规则。
- 严重问题 7（Vant 4）：1.3 节技术选型表新增 Vant 4 条目；6.3.4 节 package.json 新增 `"vant": "^4.9.0"` 及按需引入 devDependencies；4.5.1 节补充 Vant 4 主题变量映射表。
- 一般问题 8（DoctorChatView.vue 路由参数变化）：4.3 节流程图新增 `watch route.params.id` 分支，调用 `abortActiveConnection` 关闭旧 SSE 连接。
- 一般问题 9（Admin.vue/AiChatDialog.vue SSE 流程）：4.3 节 Admin.vue 流程图补充 `registerAbortController`/`abortActiveConnection` 调用；新增 AiChatDialog.vue 独立 Mermaid 流程图，明确弹窗关闭时调用 `abortActiveConnection`；3.7 节 chatStore 补充三端并发场景处理策略。
- 一般问题 10（POST /api/plan/generate 响应）：3.2.13 节保留 `diet_plans`/`exercise_plans` 分组结构并标注为合理设计偏离，字段名映射关系（`type→plan_type`、`order→order_num`、`time→time_desc`）明确说明，`plan_type` 改为英文枚举值。
- 一般问题 11（跨标签页同步）：1.2 节描述改为调用 `authStore.syncFromStorage()`；3.7 节新增 `setAuth(newToken, newRole, user)` 和 `syncFromStorage()` 方法，同时更新 token/role/userInfo 三个字段。
- 一般问题 12（NewsView.vue 免责声明）：4.3 节 NewsView.vue 流程图"点击生成健康资讯"分支前新增 `hasAcceptedDisclaimer` 判定节点；4.4.4 节 useUI.ts 调用点说明补充 NewsView.vue。
- 一般问题 13（SSE created_at 字段）：3.3 节 message 和 message_end 事件 data 字段新增 `created_at: number`；3.8.7 节 SSEMessageEvent/SSEMessageEndEvent 接口同步；4.3 节流程图明确 created_at 渲染为 msg-time 时间戳。
- 一般问题 14（blocking 模式超时）：6.3.5 节读取超时从 60s 调整为 15s，与需求 7.3 节"统一 15 秒"一致。
- 一般问题 15（POST /api/punch 枚举值）：1.8 节重写为方案 A（API/DB/TS 三层统一英文），移除 mapper.js 转换层，前端 UI 层通过 `enumLabel()` 函数查表映射为中文展示；3.2.7/3.2.16/3.2.17 节请求/响应均改为英文枚举值；3.8 节 TypeScript 类型同步。
- 一般问题 16（AiChatDialog.vue DOM 登录引导）：4.1.1 节将 welcome-tips 拆分为 `#fab-welcome-logged-in`（v-if="authStore.token"）和 `#fab-login-prompt`（v-else，含"前往登录"按钮）；输入框区域增加 `v-if="authStore.token"` 控制仅已登录用户渲染。
- 一般问题 17（admin_logs 字段名）：2.1 ER 图、2.2 DDL、2.3 索引、2.5 数据字典、3.2.30 API 响应、3.8.3 AdminLog 类型、7.3.3 伪代码统一将 `admin_id` 重命名为 `operator_id`，数据字典明确说明记录 admin 或 user 操作者 ID。
- 轻微问题 18-21：均在对应章节标注为"详细设计扩展字段"或"Dify 平台可能事件类型预扩展"或"门禁验证任务未执行"的说明。

**[通过]** 需求 4.1-4.11 所有功能模块均有对应技术方案覆盖：系统首页（Home.vue）、医师咨询（Consultation.vue + DoctorChatView.vue）、个人中心（Profile.vue + 嵌套子路由）、风险预测（Risk.vue 三步向导 + sessionStorage 持久化）、生活方案（LifePlan.vue + 方案生成/调整/打卡）、健康资讯（NewsView.vue + ArticleDetailView.vue）、打卡记录与分析（Punch.vue）、AI 智能助手（AiChatDialog.vue + diabetes-assistant-agent）、智能管理（Admin.vue + admin-manager-agent）、用户认证（authStore + JWT + 跨标签页同步 + 401 处理）、医学免责声明（路由守卫 + FAB 弹窗 + NewsView.vue 生成按钮）。

**[通过]** 数据流形成完整闭环：常规 CRUD 路径（前端 → Express → SQLite）、AI Text2SQL 路径（前端 → Express → Dify Agent → 回调 /api/admin/execute → SQLite）、AI 内容生成持久化路径（前端 → Express → Dify 工作流 → SQLite）三条路径均完整定义，含端到端字段映射契约（5.2.1.1 节风险预测字段映射表）。

**[通过]** 跨模块数据传递方案明确：方案 A（Pinia Store 共享，如 riskFormStore.result 跨页面传递）、方案 B（Vue Router query params 瞬时数据传递）。

### 3. 可操作性

**[通过]** 技术方案中的每项说明都有明确结论：
- API 端点请求/响应 JSON Schema 完整定义（3.2 节 32 个端点均有请求体、响应体、错误响应示例）
- TypeScript 类型定义覆盖所有实体（3.8 节 8 个子节，30+ interface/type 定义）
- Dify 工作流/Agent 配置规格完整（5.2 节含系统提示词、输入变量表、输出结构定义、工具定义表）
- 数据库 DDL 完整（2.2 节 10 张表）含索引策略（2.3 节）
- 部署配置完整（6.1 Nginx 配置、6.3 Express 启动脚本和 .env 模板、6.3.3/6.3.4 前后端 package.json、6.7 Keepalived 配置）

**[通过]** 实现者能从方案中明确知道"做什么"和"怎么做的大方向"：
- 每个页面组件的 DOM 结构（4.1 节 12 个组件树）、状态管理（4.2 节状态存储位置表）、JS 逻辑流程图（4.3 节 12 个 Mermaid flowchart）完整
- SSE 流式请求封装 useSSE.ts（4.4.2 节）含 401 处理和 ReadableStream 读取循环完整伪代码
- Pinia Store 接口定义完整（3.7 节 authStore/chatStore/riskFormStore 的 state/actions/持久化策略/并发控制机制）
- 行级权限校验 validateRowLevelPermission（7.3.4 节）基于 AST 解析，含表分类校验规则表、校验伪代码、辅助函数说明和 fail-closed 原则
- Express 路由处理器核心逻辑（7.3.3 节）含双认证路径区分、动态角色查询、SQL 白名单校验、多语句检测、日志记录完整伪代码

**[通过]** 技术引用足够具体：
- 所有第三方库版本号明确（1.3 节技术选型表）
- 所有 API 端点路径和参数名明确（3.1/3.2 节端点清单和 Schema）
- 所有数据表字段名、类型、约束明确（2.2 DDL + 2.5 数据字典）
- 所有环境变量命名和示例值明确（6.3.2 .env.example）
- Dify 工作流/Agent 工具回调请求体模板明确（5.2.5/5.2.6 节工具定义表）

**[轻微]** 需求 4.10 节 authStore 字段名 `userInfo` 与详细设计 3.7 节字段名 `user` 存在命名差异，需求 riskFormStore 方法 `updateFormData`/`clearFormData` 与详细设计 `saveStep`/`saveResult`/`reset` 存在方法名差异。这些差异属于详细设计阶段的合理细化（详细设计方法更细化为分步骤保存），不影响实现路径的清晰性。需求 4.10 节已明确"完整字段定义、方法签名和序列化策略在概要设计阶段确定"，详细设计阶段的方法名细化在合理范围内。

**[轻微]** 需求 5 节 punch_in.punch_type CHECK 约束为 `('diet', 'exercise')`，详细设计 2.2 节扩展为 `('diet', 'exercise', 'other')`。详细设计已说明理由（与 life_plans.plan_type 枚举维度一致，使 'other' 类型方案项可正常打卡），2.5 节 life_plans 数据字典已标注"该三种类型方案项均可正常生成展示与打卡"。此扩展合理且已说明。

## 修改要求（不适用）

本审查结论为 APPROVED，无修改要求。
