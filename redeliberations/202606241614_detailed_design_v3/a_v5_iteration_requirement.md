根据以下审查结果，迭代上一轮的产出，形成新版的文件，从而更好地满足用户需求。

## 当前审查结果

本轮诊断报告共识别 21 个质量问题（7 严重 / 10 一般 / 4 轻微），质询报告结论为 **LOCATED**（已确认），证据充分性、逻辑完整性、覆盖完备性三个维度均通过——所有问题项均标注了具体行号位置并引用了需求文档与设计文档的对应内容进行对照，未发现仅凭推测的判定。请组件A按以下问题清单逐项修复。

### 严重问题（优先修复）

1. **`useSSE.ts` 未处理 401 响应，对话中 Token 过期处理链路断裂**（第 4.4.2 节，第 3747-3784 行）
   - 问题：`useSSE.ts` 使用原生 `fetch` API 而非 axios，代码中完全没有处理 401 响应的逻辑——既不检查 `response.status`，也不调用 `authStore.clearAuth()`，也不触发跳转登录。Axios 响应拦截器无法拦截 `useSSE.ts` 中的原生 fetch 调用。这导致所有 SSE 流式端点（医师对话、AI 助手、管理员对话）在 Token 过期时前端无法感知 401 状态，违反需求 4.10 节的 Token 过期处理契约。
   - 改进建议：在 `useSSE.ts` 的 `streamRequest` 函数中，在 `fetch` 调用后立即检查 `response.status === 401`，若为 401 则调用 `authStore.clearAuth()` + 展示 Toast + 保持对话窗口打开 + 标记需要重新登录；在 4.4.2 节伪代码中补充 401 处理分支，与 4.4.1 节 axios 拦截器的 401 逻辑等价；在 4.3 节 DoctorChatView.vue / Admin.vue / AiChatDialog.vue 流程图中增加"fetch 返回 401 → 触发登录引导"分支。

2. **chatStore 方法命名与需求 4.10 节定义不一致，且缺失清除方法**（第 3.7 节，第 2321-2334 行）
   - 问题：详细设计定义的方法名为 `setDoctorConversationId`/`getDoctorConversationId`/`setAssistantConversationId`（多了 "Id" 后缀，返回类型 `undefined` vs 需求的 `null`），且完全没有定义 `clearDoctorConversation` 和 `clearAssistantConversation` 方法。需求 4.2 节和 4.8 节均明确要求"会话不设置自动过期，由用户在 UI 中手动删除"——缺失 clear 方法将导致用户无法在 UI 中删除历史会话。
   - 改进建议：将方法名统一为需求规范 `setDoctorConversation`/`getDoctorConversation`/`setAssistantConversation`；新增 `clearDoctorConversation(doctorId: number): void` 和 `clearAssistantConversation(): void` 方法，并同步持久化清理 localStorage 中的对应键；同步更新 4.3 节 DoctorChatView.vue 流程图中的"清空对话"按钮逻辑调用 `clearDoctorConversation`。

3. **`POST /api/risk/predict` 0 值校验错误使用 HTTP 400，与需求 6.13 节错误码定义不一致**（第 3.2.7 节，第 1542-1550 行）
   - 问题：需求 6.13 节明确定义 `VALIDATION_ERROR` 对应 HTTP 422。详细设计 3.2.7 节的 0 值校验错误响应使用 HTTP 400，错误码 `VALIDATION_ERROR` 与 HTTP 400 不匹配。且设计文档第 4996 行自身将 Dify 400 映射为 Express 422 VALIDATION_ERROR，存在内部不一致。前端 Axios 拦截器若按 422 处理参数校验错误，将无法正确识别 400 响应。
   - 改进建议：将 HTTP 状态码从 400 改为 422，与需求 6.13 节和 3.4 节错误码枚举表保持一致。

4. **`GET /api/articles/:id` 响应缺少 `is_collected` 字段，与需求 6.7 节直接矛盾**（第 3.2.20 节，第 1928-1945 行）
   - 问题：需求 6.7 节明确要求 `GET /api/articles/:id` 响应包含 `is_collected`（boolean）字段——"当前登录用户是否已收藏此文（未登录时为 false）。此字段为推断补充，用于前端在文章详情页直接渲染收藏按钮的已收藏/未收藏状态，避免额外的收藏状态查询请求。"详细设计 3.2.20 节响应 JSON 中完全没有 `is_collected` 字段。同时 ArticleDetailView.vue 流程图（第 3439-3448 行）改为"并行请求 GET /api/articles/collections 判断收藏状态"——这与需求"避免额外的收藏状态查询请求"的设计意图直接矛盾，每次打开文章详情都会发起额外的收藏列表请求，浪费网络资源。
   - 改进建议：在 3.2.20 节响应 JSON 中新增 `is_collected: boolean` 字段，由 Express 端点在查询文章详情时同步联查 `article_collections` 表判断当前用户收藏状态（未登录时返回 false）；同步更新 3.2.21 节 `POST /api/articles/generate` 响应（需求 6.7 节要求结构与详情一致）；更新 ArticleDetailView.vue 流程图，删除"并行请求 GET /api/articles/collections"分支，改为直接读取响应中的 `is_collected` 字段渲染收藏按钮状态。

5. **`GET /api/articles` 列表响应缺少 `tags`/`summary` 字段，字段名与需求 6.7 节不一致**（第 3.2.19 节，第 1907-1925 行）
   - 问题：需求 6.7 节列表响应元素结构包含字段：`id, title, cover, author, publish_time, category, tags, read_count, summary`。详细设计响应元素结构为：`id, title, cover, author, category, views, created_at`。差异：（1）缺失字段 `tags`（标签数组）、`summary`（摘要文本）——这两个字段是前端文章列表卡片渲染的关键信息，缺失将导致卡片无法展示标签和摘要；（2）字段名不一致：需求用 `publish_time` 和 `read_count`，详细设计用 `created_at` 和 `views`——前端 TypeScript 类型定义（3.8.3 节 `Article` 接口）也使用了 `created_at` 和 `views`，与需求规范不一致。
   - 改进建议：在 3.2.19 节响应 JSON 中补充 `tags: string[]` 和 `summary: string` 字段；评估是否将 `publish_time`/`read_count` 与 `created_at`/`views` 统一——若保留 `created_at`/`views` 命名，需在文档中明确说明与需求字段名的映射关系，并确认需求方接受此偏离；同步更新 3.8.3 节 `Article` TypeScript 接口，补充 `tags` 和 `summary` 字段；同步更新 articles 表 DDL——当前 DDL（第 826-835 行）没有 `tags` 和 `summary` 列，需要补充（`tags TEXT DEFAULT '[]'`、`summary TEXT DEFAULT ''`）。

6. **Dify Agent 工具定义与需求 6.11 节工具清单严重不符**（第 5.2.5 节，第 4352-4364 行；第 5.2.6 节，第 4400 行）
   - 问题：需求 6.11 节明确定义了两个 Agent 的预期工具清单：diabetes-assistant-agent 应有 8 个专用工具（`query_user_profile`、`query_risk_history`、`query_punch_records`、`query_life_plans`、`query_health_advice`、`write_health_advice`、`update_user_profile`、`knowledge_search`）；admin-manager-agent 应有 5 个专用工具（`query_table`、`insert_record`、`update_record`、`delete_record`、`get_table_schema`）。需求 6.11 节每个工具都明确定义了"权限约束"列——这些细粒度权限约束是行级权限的安全保障。详细设计将所有工具简化为单一的 `execute_SQL` 工具，由 Agent 自行生成 SQL 通过 `/api/admin/execute` 端点执行。问题：（1）完全偏离需求设计意图——需求希望每个工具有明确的语义边界和权限约束，而单一 `execute_SQL` 是通用工具，安全控制完全依赖 `validateRowLevelPermission` 的 AST 解析；（2）Agent 系统提示词仍引用"通过 execute_SQL 工具查询"，但需求工具清单中的 `knowledge_search`（知识库检索）能力无法通过 execute_SQL 实现——知识库检索是 Dify 平台原生能力，不是 SQL 操作；（3）`write_health_advice` 工具在需求中是独立的 INSERT 工具，详细设计将其简化为 execute_SQL 的 INSERT 操作，与需求工具清单的语义化设计不符。
   - 改进建议：按 5.2.5 节模式补充需求 6.11 节定义的 8+5 个专用工具的完整定义（工具名、回调 URL、请求体模板、参数说明、权限约束）；评估是否保留 `execute_SQL` 作为兜底工具，或完全替换为专用工具集；若决定保留单一 `execute_SQL` 设计作为架构决策偏离，需在文档中明确说明偏离需求规范的理由和影响，并补充 `knowledge_search` 工具的独立定义（Dify 知识库检索不能通过 SQL 实现）；在 7.3.4 节 `validateRowLevelPermission` 的表分类中补充 `life_advice` 表的写入约束规则。

7. **未采纳需求 8.1 节推荐的 Vant 4 移动端 UI 组件库**（第 1.3 节技术选型表，第 110-138 行）
   - 问题：需求 8.1 节明确要求："移动端 UI 组件库选型：推荐采用 Vant 4（移动端优先的 Vue 3 组件库），提供 Tabbar、ActionSheet、DatetimePicker、PullRefresh、Toast、Dialog 等移动端特有交互组件，避免从零开发。"需求 7.1 节也明确："若采用 Vant 4 作为移动端 UI 组件库（见 8.1 节），设计系统的 CSS 变量应与 Vant 4 的主题变量体系（如 `--van-primary-color`、`--van-font-size-md` 等）建立映射关系。"详细设计 1.3 节技术选型表完全未包含 Vant 4，仅使用 Tailwind CSS + SweetAlert2 + Swiper + Font Awesome。问题：（1）移动端特有交互组件（Tabbar、ActionSheet、DatetimePicker、PullRefresh 等）需要从零开发，增加工作量；（2）Punch.vue 的日期范围筛选器（第 3069-3072 行）使用原生 `<input type="date">`，移动端体验差；（3）CSS 变量体系（4.5.1 节）未与 Vant 4 主题变量建立映射，违反需求 7.1 节要求；（4）前端 package.json（6.3.4 节）未包含 Vant 4 依赖。
   - 改进建议：在 1.3 节技术选型表新增 Vant 4 条目；在 6.3.4 节前端 package.json dependencies 中新增 `"vant": "^4.9.0"`；在 4.5.1 节 CSS 变量定义后补充 Vant 4 主题变量映射表（如 `--van-primary-color: var(--color-primary)`）；评估 TabBar.vue、日期筛选器、Toast、Dialog 等组件是否改用 Vant 4 组件替代自研实现。

### 一般问题（开发启动前澄清）

8. **DoctorChatView.vue 未处理"切换至其他医生对话"的 SSE 连接关闭场景**（第 4.3 节，第 3312-3336 行）
   - 问题：需求 4.2 节明确要求："用户与医生 A 建立 SSE 连接进行流式对话期间，若切换至医生 B 的对话界面，前端应立即关闭医生 A 的 SSE 连接（调用 AbortController.abort()）。"详细设计 DoctorChatView.vue 流程图仅在两个场景调用 `abortActiveConnection`：用户点击返回按钮、组件销毁 onUnmounted。但 Vue Router 在路由参数变化（如 `/consultation/doctor/1` → `/consultation/doctor/2`）时，同一组件会被复用，onUnmounted 不会触发。用户从医生 A 对话直接跳转到医生 B 对话，原 SSE 连接不会被关闭，违反需求 4.2 节"同时活跃的 SSE 连接数上限为 1"的约束。
   - 改进建议：在 DoctorChatView.vue 的 `<script setup>` 中使用 `watch(() => route.params.id, ...)` 监听路由参数变化，参数变化时调用 `chatStore.abortActiveConnection()` 关闭旧连接；或使用 Vue Router 的 `beforeRouteUpdate` 守卫处理同组件路由参数变化；在 4.3 节流程图中新增"路由参数变化 → abortActiveConnection → 重新初始化"分支。

9. **Admin.vue 和 AiChatDialog.vue 的 SSE 流程未调用 abortActiveConnection**（第 4.3 节 Admin.vue 流程图第 3571-3595 行；AiChatDialog.vue 无独立流程图）
   - 问题：需求 4.2 节明确要求"同时活跃的 SSE 连接数上限为 1"。3.7 节 chatStore 定义了 `activeAbortController` 状态和 `registerAbortController`/`abortActiveConnection` actions 用于 SSE 连接管理。但：（1）Admin.vue 流程图没有调用 `registerAbortController` 注册新的 AbortController，也没有在视图切换或组件卸载时调用 `abortActiveConnection`；（2）AiChatDialog.vue 作为全局 FAB 弹窗组件，没有独立的流程图，且其 SSE 流式对话的生命周期管理（弹窗关闭时是否关闭 SSE 连接）未明确；（3）三个 SSE 端点（医师对话/AI 助手/管理员对话）的并发控制策略未说明——用户同时打开医师对话和 AI 助手弹窗时，哪个连接应该被关闭？
   - 改进建议：在 Admin.vue 流程图中补充 `registerAbortController` 调用（发送消息前）和 `abortActiveConnection` 调用（视图切换/组件卸载时）；为 AiChatDialog.vue 补充独立的 Mermaid 流程图，明确弹窗关闭时调用 `abortActiveConnection` 中止 AI 助手 SSE 连接；在 3.7 节 chatStore 的"连接控制与并发限制机制"中明确：当用户在医师对话页面打开 AI 助手 FAB 弹窗时，是否需要关闭医师对话的 SSE 连接？还是允许两个连接并存（违反"上限为 1"约束）？

10. **`POST /api/plan/generate` 响应结构与需求 6.5 节不一致**（第 3.2.13 节，第 1727-1755 行）
    - 问题：需求 6.5 节响应返回单一 `items` 数组（含所有方案项），详细设计拆分为 `diet_plans` + `exercise_plans` 两个分组数组；字段名不一致：需求用 `type`/`order`/`time`，详细设计用 `plan_type`/`order_num`/`time_desc`；详细设计多了 `success` 包装字段（这是详细设计的统一响应格式，可接受）。这种结构性偏差导致前端 TypeScript 类型定义（3.8.5 节 `PlanResponse`）与需求契约不一致，前端开发者按需求契约编码将无法正确解析详细设计响应。
    - 改进建议：评估两种结构的优劣——`diet_plans`/`exercise_plans` 分组结构对前端渲染更友好（无需自行分组），可作为合理设计偏离保留；但字段名 `type`/`order`/`time` vs `plan_type`/`order_num`/`time_desc` 的不一致需要统一——建议与数据库字段名（`plan_type`/`order_num`/`time_desc`）保持一致，但在文档中明确说明与需求字段名的映射关系；life_plans 表字段名也应统一——需求 5 节用 `order` 和 `time`，详细设计 DDL 用 `order_num` 和 `time_desc`。建议保留详细设计的命名（`order` 是 SQL 保留字，使用 `order_num` 更安全），但在文档中明确说明偏离。

11. **跨浏览器标签页登录态同步使用 `setToken()` 而非 `setAuth()`，未同步更新 role 和 userInfo**（第 1.2 节，第 108 行）
    - 问题：需求 4.10 节明确要求："若 `token` 被移除则调用 `authStore.clearAuth()` 触发登出；若 `token` 被修改为其他值则同步更新 `authStore.setAuth()`。"需求 4.10 节定义的 `setAuth` 方法签名：`function setAuth(newToken: string, newRole: string, user: UserInfo): void`——同时更新 token、role、userInfo 三个字段。详细设计 1.2 节描述："如果新值不一致，则更新当前标签页的 authStore.setToken() 以保持多标签页之间的登录态同步。"——只调用了 `setToken()`，没有同步更新 role 和 userInfo。这导致：若用户在标签页 A 切换账号（登录态从用户 1 变为用户 2），标签页 B 的 storage 事件监听只更新 token，role 和 userInfo 仍是用户 1 的数据，造成 UI 显示用户 1 的头像和用户名但实际 token 是用户 2 的状态错乱。
    - 改进建议：修改 1.2 节描述为：token 变化时调用 `authStore.fetchProfile()` 重新获取用户信息，或直接从 localStorage 恢复 role 和 userInfo（若持久化了）；或在 authStore 中新增 `syncFromStorage()` 方法，从 localStorage 同步恢复 token/role/userInfo 三个字段，由 storage 事件监听器调用；需求 4.10 节定义的 `setAuth(newToken, newRole, user)` 方法在详细设计的 authStore 接口（3.7 节）中也没有定义——应补充此方法。

12. **NewsView.vue 流程图未在"点击生成健康资讯"按钮时调用免责声明判定**（第 4.3 节，第 3415-3424 行）
    - 问题：需求 4.11 节明确要求："用户首次访问以下任一 AI 功能入口时（医师对话、风险预测提交、方案生成、资讯生成、AI 助手对话），前端弹出免责声明确认弹窗。"详细设计 1.6.2 节路由守卫为 `/consultation/doctor/:id`、`/life-plan`、`/profile/risk`、`/profile/advice` 设置了 `requiresDisclaimer: true`，但 `/news` 和 `/news/article/:id` 未设置（因为浏览资讯本身不是 AI 功能）。这是合理的——浏览资讯不需要免责声明。但点击"生成健康资讯"按钮触发 AI 生成时，NewsView.vue 流程图（第 3415 行）的流程是：`L[点击生成健康资讯按钮] --> M[发起请求 POST /api/articles/generate 不传 category]`。直接发起了 AI 生成请求，没有调用 `hasAcceptedDisclaimer()` 判定！违反需求 4.11 节"资讯生成"是 AI 功能入口的要求。
    - 改进建议：在 NewsView.vue 流程图的"点击生成健康资讯按钮"分支前增加免责声明判定节点：`L[点击生成健康资讯] --> L1{hasAcceptedDisclaimer?} -->|否| L2[showDisclaimer] --> L3{用户同意?} -->|否| 终止 -->|是| 写入 localStorage --> M`；在 4.4.4 节 useUI.ts 免责声明函数调用点说明中补充 NewsView.vue 的调用点。

13. **SSE 事件 `message`/`message_end` 缺少 `created_at` 字段，与需求 6.9 节定义不一致**（第 3.3 节，第 2203-2215 行）
    - 问题：需求 6.9 节明确定义 SSE 事件 data 字段核心字段包含 `created_at`（number）：Unix 时间戳（秒），消息创建时间。详细设计 3.3 节 `message` 事件 data 字段结构：`{"event": "message", "answer": "...", "conversation_id": "xxx", "message_id": "xxx"}` —— 缺少 `created_at`；`message_end` 事件 data 字段结构也缺少 `created_at`；3.8.7 节 `SSEMessageEvent` 和 `SSEMessageEndEvent` TypeScript 接口也缺少 `created_at` 字段。这导致前端无法获取消息创建时间戳，对话消息的时间显示将缺失。
    - 改进建议：在 3.3 节 `message` 和 `message_end` 事件 data 字段结构中补充 `created_at: number` 字段；在 3.8.7 节 `SSEMessageEvent` 和 `SSEMessageEndEvent` 接口中新增 `created_at: number` 成员；在 4.3 节对话流程图中明确前端应将 `created_at` 渲染为消息时间戳（`<span class="msg-time">` 节点的内容来源）。

14. **`difyService.js` blocking 模式读取超时 60s 与需求 7.3 节"15 秒"不一致**（第 6.3.5 节，第 4987-4990 行）
    - 问题：需求 7.3 节明确要求："响应超时处理：所有 AI 接口超时阈值统一为 15 秒，超时后展示超时提示（如'响应超时，请点击重试'）并允许用户手动重试。"需求 7.2 节也要求："单次 AI 请求应在 15 秒内开始返回内容"。详细设计 6.3.5 节 blocking 模式（用于 POST /api/risk/predict、POST /api/plan/generate、POST /api/articles/generate）的读取超时是 60 秒，远超需求统一的 15 秒阈值。这意味着风险预测、方案生成、文章生成等 AI 操作可能等待 60 秒才超时，违反需求 7.3 节的"统一 15 秒"要求。
    - 改进建议：将 blocking 模式读取超时从 60s 调整为 15s，与需求 7.3 节统一；或在文档中明确说明偏离理由——如 Dify 工作流执行时间可能超过 15 秒，需要更长的读取超时——并补充前端加载状态提示覆盖整个超时窗口；评估 streaming 模式"无限制（流式）"是否合理——需求 7.3 节的 15 秒是"开始返回内容"的超时，streaming 模式的连接超时 10s + 流式无限制符合需求"15 秒内开始返回内容"的精神，可保留。

15. **`POST /api/punch` 请求体使用中文枚举值，与需求 6.6 节英文枚举值不一致**（第 3.2.16 节，第 1800-1808 行）
    - 问题：需求 6.6 节 `POST /api/punch` 请求体参数明确定义：`punch_type` 枚举值 `"diet"`/`"exercise"`；`completion_status` 枚举值 `"completed"`/`"uncompleted"`。详细设计 3.2.16 节请求体使用中文值：`{"punch_type": "饮食", "completion_status": "已完成"}`。这种 API 接口层使用中文枚举值的设计依赖于 `mapper.js` 转换层（1.8 节）将其转为英文落库。但：（1）需求 6.6 节明确使用英文枚举值，详细设计的 API 契约与需求规范直接矛盾；（2）同样的问题存在于 `POST /api/risk/predict` 请求体（3.2.7 节使用 `diabetes_history: "健康"`，需求 6.3 节使用 `"healthy"`）；（3）`RiskPredictRequest` TypeScript 类型（3.8.4 节）也使用中文枚举值，与需求 6.3 节英文枚举值不一致；（4）`LifePlan.plan_type`（3.8.3 节）使用 `'饮食' | '运动' | '其他'`，与需求 5 节 `'diet' | 'exercise' | 'other'` 不一致。这是详细设计做出的全局性设计决策偏离——所有 API 接口层使用中文枚举值，依赖 mapper.js 转换。但这种偏离与需求规范直接矛盾，且增加了维护复杂度（需要维护双向映射字典）。
    - 改进建议：评估两种方案的优劣：方案 A（遵循需求）：API 接口层使用英文枚举值，前端 TypeScript 类型使用英文枚举值，前端 UI 渲染时通过查表将英文映射为中文展示。无需 mapper.js 转换层。方案 B（保留当前设计）：API 接口层使用中文枚举值，依赖 mapper.js 转换。需要在文档中明确说明这是对需求规范的偏离，并获得需求方确认。若选择方案 B，需在 1.8 节明确标注此为设计决策偏离需求 6.3/6.6 节规范，并说明理由（如前端 UI 直接展示中文更友好）；若选择方案 A，需要删除 mapper.js 转换层，统一 API 契约为英文枚举值，前端 UI 层自行映射展示。

16. **`AiChatDialog.vue` DOM 结构缺少明确的"登录引导提示"和"跳转登录页按钮"元素**（第 4.1.1 节，第 2736-2746 行）
    - 问题：需求 4.8 节明确要求："未登录用户点击 FAB 按钮时，AI 助手对话窗口仍然弹出，但窗口内不展示对话输入区域，而是展示登录引导提示（如'请先登录后使用 AI 智能助手'）和跳转至登录页的按钮。"详细设计 4.1.1 节 AiChatDialog.vue DOM 树中：（1）`welcome-tips` 区域用途不明确——是欢迎语还是登录引导？注释说"未登录/首次打开展示"，混淆了两种不同场景；（2）缺少明确的"跳转至登录页"按钮元素——需求要求有独立的按钮跳转到 `/login`；（3）输入框和发送按钮在未登录时隐藏，但隐藏后该区域是空白还是有替代内容？未说明。
    - 改进建议：将 `welcome-tips` 区域拆分为两个独立子区域：`<div id="fab-welcome-logged-in">`（已登录用户首次打开的欢迎语 + 推荐提问）和 `<div id="fab-login-prompt">`（未登录用户的登录引导）含 `<p>请先登录后使用 AI 智能助手</p>` 和 `<button @click="router.push('/login')">前往登录</button>`；通过 `v-if="!authStore.token"` 和 `v-else` 控制两个区域的显隐；在流程图中补充未登录用户点击 FAB 的处理分支。

17. **`admin_logs` 表字段名 `admin_id` 用于记录普通用户 user_id 产生语义混淆**（第 2.2 节 admin_logs DDL 第 918-926 行；第 2.5 节数据字典第 1243-1252 行）
    - 问题：需求 4.10 节明确要求："管理员操作日志表（admin_logs）的写入不受行级限制——user 角色的操作同样记录日志（操作类型标注为'user_text2sql'以区分管理员操作）。"详细设计 admin_logs 表使用 `admin_id` 字段名记录操作者 ID。但当普通用户通过 AI 助手触发 Text2SQL 操作时，其 user_id 也会写入 `admin_id` 字段——字段名 `admin_id` 暗示是管理员 ID，但实际可能存储普通用户 ID，产生语义混淆。7.3.3 节路由处理器伪代码（第 5303 行）：`insertAdminLog(operatorId, authMode==='dify_callback'?'user_text2sql':getOpType(sql), sql, '成功');` 无论是 admin 还是 user，都调用 `insertAdminLog(operatorId, ...)`，operatorId 写入 `admin_id` 字段。
    - 改进建议：将 `admin_id` 字段重命名为 `operator_id`（操作者 ID），消除语义混淆；同步更新 DDL、数据字典、ER 图、7.3.3 节伪代码、3.2.30 节响应字段名；在数据字典中明确说明：`operator_id` 记录操作者用户 ID，可能是 admin（管理员操作）或 user（普通用户 Text2SQL 操作），通过 `operation_type` 字段区分。

### 轻微问题（开发过程中逐步完善）

18. **`POST /api/risk/predict` 响应多了 `created_at` 字段，需求 6.3 节未定义**（第 3.2.7 节，第 1554-1565 行）
    - 问题：需求 6.3 节响应字段：`risk_score, risk_level, risk_level_label, matched_diabetes_type, advice, record_id`。详细设计响应多了 `created_at: "2026-06-23T14:30:00"` 字段。这是合理扩展（前端可能需要展示预测时间），但与需求契约不完全一致。
    - 改进建议：保留 `created_at` 字段，但在文档中标注为"详细设计扩展字段，需求 6.3 节未定义"。

19. **chatStore 多了 `adminConversationId` 字段，需求 4.10 节未定义**（第 3.7 节，第 2314 行）
    - 问题：需求 4.10 节 chatStore 类型定义只包含 `doctorConversations` 和 `assistantConversationId` 两个字段。详细设计新增了 `adminConversationId: string | null` 字段用于管理员对话会话管理。这是合理扩展（需求 6.10 节新增了 `POST /api/admin/chat` 端点，需要会话管理），但与需求 4.10 节定义不完全一致。
    - 改进建议：保留此扩展字段，在文档中说明"基于需求 6.10 节 `POST /api/admin/chat` 端点扩展"。

20. **SSE 事件定义多了 `agent_message` 事件类型，需求 6.9 节未定义**（第 3.3 节，第 2212 行）
    - 问题：需求 6.9 节定义的 SSE 事件类型枚举：`message, message_end, error, workflow_started, workflow_finished, agent_thought`。详细设计 3.3 节新增了 `agent_message` 事件类型：`{"event": "agent_message", "answer": "...", "conversation_id": "xxx"}`。需求未定义此事件类型。若 Dify 平台实际推送此事件，详细设计的定义是合理的预扩展；若 Dify 不推送，则此定义是冗余的。
    - 改进建议：在 3.3 节标注 `agent_message` 为"基于 Dify 平台可能的事件类型预扩展，实际支持情况以 Dify 部署版本为准"。

21. **Dify 平台能力验证任务的验证结果未在文档中说明**（第 5.5.1 节，第 5110-5123 行）
    - 问题：需求 5 节明确要求："此验证任务被设定为门禁任务（gate task），必须在概要设计启动前完成。"详细设计 5.5.1 节完整定义了验证任务的方法、标准和时机，但文档中没有记录验证结果——验证是通过还是失败？若失败，是否启用备选方案？由于详细设计阶段已经完成，门禁任务应该已经执行。若未执行，整个 Text2SQL 授权管道的设计基础（`{{user}}` 变量透传）就是未经验证的假设。
    - 改进建议：在 5.5.1 节末尾补充"验证结果"段落，记录门禁任务的实际执行结果（通过/失败）；若验证失败，说明是否启用 5.5.2 节备选方案，并标注主方案设计需相应调整的章节；若验证未执行，明确标注"本设计基于 Dify `{{user}}` 变量透传能力的假设，未经验证，开发环境搭建阶段必须优先执行验证任务"。

## 历史迭代回顾

通过对前 4 轮迭代反馈的回顾分析，当前问题与历史反馈的关系如下：

### 已解决的问题（出现在历史反馈但当前反馈中不再提及的问题）

- **迭代 1 问题 2**（`user_risk_info` 表缺失 `diabetes_history` 字段）：已在历史迭代中补充字段定义，当前不再提及。
- **迭代 1 问题 3**（`life_plans` 表缺失 `is_active` 字段）：已补充字段定义和逻辑过期机制。
- **迭代 1 问题 4**（医师咨询列表与对话页面组件未拆分）：已拆分为 `Consultation.vue` 和 `DoctorChatView.vue`。
- **迭代 1 问题 5**（个人中心子页面嵌套路由设计不明确）：已调整为树形/嵌套结构。
- **迭代 1 问题 8**（健康资讯列表与详情页面组件未拆分）：已拆分为 `NewsView.vue` 和 `ArticleDetailView.vue`。
- **迭代 2 问题 1**（`difyAuth.js` 使用 `timingSafeEqual` 未校验长度导致 DoS 隐患）：已补充长度校验或哈希处理。
- **迭代 2 问题 2**（`router/index.ts` 语法嵌套错误和标点符号混淆）：已规范路由定义结构。
- **迭代 2 问题 3**（`mapper.js` 漏配 `punch_type` 映射）：已补全双向转换映射。
- **迭代 2 问题 4**（`POST /api/risk/predict` 响应结构字段混淆且缺少 `risk_level_label`）：已对齐需求文档。
- **迭代 2 问题 5**（`pregnancy` 字段类型不一致）：已统一为 `INTEGER` 类型并增加约束。
- **迭代 2 问题 6**（`seed.sql` 包含管理员密码哈希占位符）：已补充风险提示。
- **迭代 2 问题 7**（布尔逻辑字段表现形式不统一）：已统一为 `INTEGER` 类型。
- **迭代 2 问题 8**（核心数据驱动接口缺乏 SQLite 查询设计）：已补充查询伪代码。
- **迭代 3 问题 1**（`/api/admin/execute` 硬编码 `operatorRole` 为 `'user'`）：已改为动态查询用户角色。
- **迭代 3 问题 3**（`life_plans` 和 `punch_in` 表类型约束逻辑矛盾）：已统一分类维度。
- **迭代 3 问题 5**（Dify 风险预测工作流输出结构与后端查询不一致）：已统一数据契约。
- **迭代 3 问题 6**（`POST /api/punch` 响应体缺失 `remarks` 字段）：已补齐字段。
- **迭代 3 问题 7**（`AiChatDialog.vue` DOM 结构缺失医学免责提示元素）：已增加 `<p class="disclaimer-text">` 元素。

### 持续存在的问题（在多轮反馈中反复出现的问题，需重点解决）

- **枚举值语言规范问题（迭代 1 问题 1 → 迭代 4 问题 15）**：迭代 1 反馈 DDL 中使用中文 CHECK 约束与需求英文小写要求不符；当前迭代 4 反馈 API 接口层使用中文枚举值（`punch_type`/`completion_status`/`diabetes_history`/`plan_type`）与需求英文枚举值矛盾，依赖 `mapper.js` 转换层。问题从 DDL 层转移到 API 契约层，需在本轮迭代中彻底决策方案 A（遵循需求英文）或方案 B（保留中文并明确偏离说明），避免问题持续。
- **chatStore 设计与需求 4.10 节不一致（迭代 1 问题 6 → 迭代 4 问题 2）**：迭代 1 反馈 chatStore 使用通用 Map 替代需求规范的独立变量；当前迭代 4 反馈方法命名多了 "Id" 后缀、返回类型 `undefined` vs `null`、且缺失 `clearDoctorConversation`/`clearAssistantConversation` 方法。需在本轮迭代中完全对齐需求 4.10 节规范。
- **跨浏览器标签页登录态同步机制（迭代 1 问题 9 → 迭代 4 问题 11）**：迭代 1 反馈缺少跨标签页同步机制；当前迭代 4 反馈存在同步机制但使用 `setToken()` 而非 `setAuth()`，未同步更新 role 和 userInfo，且未定义 `setAuth` 方法。需在本轮迭代中完善 `setAuth` 方法和完整的字段同步逻辑。
- **SSE 流式对话连接控制与并发限制（迭代 1 问题 10 → 迭代 4 问题 1、8、9）**：迭代 1 反馈缺少 SSE 连接管理和并发限制；当前迭代 4 反馈虽然引入了 `AbortController` 机制，但存在三个具体缺陷：（1）`useSSE.ts` 未处理 401 响应；（2）DoctorChatView.vue 未处理路由参数变化时的连接关闭；（3）Admin.vue 和 AiChatDialog.vue 的 SSE 生命周期管理缺失。需在本轮迭代中全面完善 SSE 连接管理的所有场景。
- **0 值校验与错误码规范（迭代 1 问题 11 → 迭代 4 问题 3）**：迭代 1 反馈 `waist` 和 `systolic_bp` 缺少 0 值校验；当前迭代 4 反馈虽然已增加 0 值校验，但错误响应使用 HTTP 400 与需求 6.13 节 `VALIDATION_ERROR` 对应 HTTP 422 不一致。需在本轮迭代中将 HTTP 状态码改为 422。
- **医学免责声明调用完整性（迭代 3 问题 4 → 迭代 4 问题 12）**：迭代 3 反馈 `showDisclaimer` 方法仅定义未实际调用；当前迭代 4 反馈虽然已在路由守卫和部分组件中接入免责声明，但 NewsView.vue 的"点击生成健康资讯"按钮仍直接发起 AI 生成请求未调用 `hasAcceptedDisclaimer()` 判定。需在本轮迭代中补充所有 AI 功能入口的免责声明判定。
- **`validateRowLevelPermission` 实现完整性（迭代 3 问题 2 → 迭代 4 问题 6）**：迭代 3 反馈 `validateRowLevelPermission` 缺失具体实现规范；当前迭代 4 反馈虽然已补充实现，但在 Dify Agent 工具设计偏离需求的背景下，`life_advice` 表的写入约束规则需补充。需在本轮迭代中结合 Agent 工具设计的修订同步完善。
- **数据映射转换层设计决策（迭代 1 问题 7 → 迭代 4 问题 15）**：迭代 1 反馈缺少数据字段映射与映射层的技术决策；当前迭代 4 反馈虽然已引入 `mapper.js` 转换层，但该设计决策偏离需求 6.3/6.6 节规范且未在文档中明确说明。需在本轮迭代中明确方案选择并补充偏离说明。

### 新发现的问题（本轮新识别的问题）

- **问题 1**（`useSSE.ts` 未处理 401 响应）：本轮新发现，属于 SSE 连接管理在新场景下的具体缺陷。
- **问题 4**（`GET /api/articles/:id` 响应缺少 `is_collected` 字段）：本轮新发现，需求契约对齐问题。
- **问题 5**（`GET /api/articles` 列表响应缺少 `tags`/`summary` 字段，字段名不一致）：本轮新发现，需求契约对齐问题。
- **问题 6**（Dify Agent 工具定义与需求 6.11 节工具清单严重不符）：本轮新发现，架构设计偏离需求意图。
- **问题 7**（未采纳需求 8.1 节推荐的 Vant 4 移动端 UI 组件库）：本轮新发现，技术选型偏离需求推荐。
- **问题 8**（DoctorChatView.vue 未处理路由参数变化时的 SSE 连接关闭）：本轮新发现，Vue Router 同组件复用场景下的连接管理缺陷。
- **问题 9**（Admin.vue 和 AiChatDialog.vue 的 SSE 流程未调用 abortActiveConnection）：本轮新发现，SSE 生命周期管理覆盖不完整。
- **问题 10**（`POST /api/plan/generate` 响应结构与需求 6.5 节不一致）：本轮新发现，需求契约对齐问题。
- **问题 13**（SSE 事件 `message`/`message_end` 缺少 `created_at` 字段）：本轮新发现，需求契约对齐问题。
- **问题 14**（`difyService.js` blocking 模式读取超时 60s 与需求 7.3 节"15 秒"不一致）：本轮新发现，超时配置偏离需求。
- **问题 16**（`AiChatDialog.vue` DOM 结构缺少登录引导提示和跳转登录页按钮）：本轮新发现，UI 元素缺失违反需求 4.8 节。
- **问题 17**（`admin_logs` 表字段名 `admin_id` 产生语义混淆）：本轮新发现，字段命名语义问题。
- **问题 18-21**（轻微问题）：本轮新发现的合理扩展或文档说明缺失问题。

## 上一轮产出路径

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/a_v4_tech_v2.md

## 用户需求

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/requirement.md
