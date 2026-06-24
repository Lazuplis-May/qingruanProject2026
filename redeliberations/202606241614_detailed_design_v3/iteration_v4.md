# 再审议判定报告（v4）

## 判定结果

RETRY

## 判定理由

本次判定基于组件B诊断报告（b_v4_diag_v1.md）和质询报告（b_v4_challenge_v1.md）的综合分析。

**质询状态分析**：
- 组件B内部循环最大轮次：12，实际轮次：1
- 质询结果为 LOCATED，实际轮次（1）< 最大轮次（12），说明质询在首轮即提前终止并确认了诊断报告中的问题，审查结论已被独立验证确认。

**诊断报告问题统计**：
- 严重问题：7 项（问题 1-7）
- 一般问题：10 项（问题 8-17）
- 轻微问题：4 项（问题 18-21）

**质询报告确认情况**：
质询报告在证据充分性、逻辑完整性、覆盖完备性三个维度均判定"通过"，逐项抽样验证了问题 1、2、3、4、6、13、14 的证据准确性，确认所有问题项均标注了具体行号位置并引用了需求文档与设计文档的对应内容进行对照，未发现仅凭推测的判定。问题分级合理，改进建议与问题描述一一对应，问题之间无矛盾。

**判定依据**：
根据判定标准，审查报告包含严重或一般等级的问题时应判定为 RETRY。本次诊断报告包含 7 项严重问题和 10 项一般问题，且质询报告已 LOCATED 确认这些问题成立，不满足 PASS 的任何条件。具体而言：

1. 严重问题涉及需求契约直接矛盾（HTTP 状态码 400 vs 422、API 响应字段缺失 is_collected/tags/summary、chatStore 方法命名不一致且缺失 clear 方法、Dify Agent 工具设计偏离需求 6.11 节、Vant 4 UI 组件库未采纳）和关键功能链路断裂（useSSE.ts 未处理 401 响应导致 Token 过期处理失效）。

2. 一般问题涉及 SSE 连接管理覆盖不完整（Admin.vue/AiChatDialog.vue 缺失 abortActiveConnection 调用、路由参数变化时未关闭旧连接）、跨标签页登录态同步未更新 role/userInfo、NewsView.vue 资讯生成未调用免责声明判定、SSE 事件缺少 created_at 字段、blocking 模式超时 60s 与需求 15s 不一致、API 枚举值中英文不一致、AiChatDialog.vue 登录引导 UI 元素缺失、admin_logs 字段名语义混淆等。

这些问题均已被质询报告确认成立，需要重新运行组件A进行修订。

## 需要解决的问题

### 问题 1
- **问题描述**：`useSSE.ts` 使用原生 fetch API 且完全没有处理 401 响应的逻辑，导致所有 SSE 流式端点（医师对话、AI 助手、管理员对话）在 Token 过期时前端无法感知 401 状态，违反需求 4.10 节的 Token 过期处理契约。
- **所在位置**：第 4.4.2 节 `useSSE.ts`（第 3747-3784 行）
- **严重程度**：严重
- **改进建议**：在 `useSSE.ts` 的 `streamRequest` 函数中，在 `fetch` 调用后立即检查 `response.status === 401`，若为 401 则调用 `authStore.clearAuth()` + 展示 Toast + 保持对话窗口打开 + 标记需要重新登录；在 4.4.2 节伪代码中补充 401 处理分支；在 4.3 节相关流程图中增加"fetch 返回 401 → 触发登录引导"分支。

### 问题 2
- **问题描述**：chatStore 方法命名与需求 4.10 节定义不一致（多了 "Id" 后缀，返回类型 `undefined` vs 需求的 `null`），且完全缺失 `clearDoctorConversation` 和 `clearAssistantConversation` 方法，导致用户无法在 UI 中删除历史会话。
- **所在位置**：第 3.7 节 chatStore 接口定义（第 2321-2334 行）
- **严重程度**：严重
- **改进建议**：将方法名统一为需求规范 `setDoctorConversation` / `getDoctorConversation` / `setAssistantConversation`；新增 `clearDoctorConversation(doctorId: number): void` 和 `clearAssistantConversation(): void` 方法并同步持久化清理 localStorage；同步更新 4.3 节 DoctorChatView.vue 流程图中的"清空对话"按钮逻辑。

### 问题 3
- **问题描述**：`POST /api/risk/predict` 0 值校验错误使用 HTTP 400，但错误码为 `VALIDATION_ERROR`，需求 6.13 节明确定义 `VALIDATION_ERROR` 对应 HTTP 422。错误码与 HTTP 状态码不匹配，前端 Axios 拦截器若按 422 处理参数校验错误将无法正确识别 400 响应。
- **所在位置**：第 3.2.7 节（第 1542-1550 行）
- **严重程度**：严重
- **改进建议**：将 HTTP 状态码从 400 改为 422，与需求 6.13 节和 3.4 节错误码枚举表保持一致。

### 问题 4
- **问题描述**：`GET /api/articles/:id` 响应缺少 `is_collected` 字段，与需求 6.7 节直接矛盾。ArticleDetailView.vue 流程图改为"并行请求 GET /api/articles/collections 判断收藏状态"，与需求"避免额外的收藏状态查询请求"的设计意图直接矛盾。
- **所在位置**：第 3.2.20 节（第 1928-1945 行）
- **严重程度**：严重
- **改进建议**：在 3.2.20 节响应 JSON 中新增 `is_collected: boolean` 字段，由 Express 端点同步联查 `article_collections` 表判断收藏状态；同步更新 3.2.21 节响应；更新 ArticleDetailView.vue 流程图，删除"并行请求 GET /api/articles/collections"分支，改为直接读取响应中的 `is_collected` 字段。

### 问题 5
- **问题描述**：`GET /api/articles` 列表响应缺少 `tags`/`summary` 字段，字段名与需求 6.7 节不一致（需求用 `publish_time`/`read_count`，详细设计用 `created_at`/`views`），导致前端文章列表卡片无法展示标签和摘要，TypeScript 类型定义与需求规范不一致。
- **所在位置**：第 3.2.19 节（第 1907-1925 行）
- **严重程度**：严重
- **改进建议**：在 3.2.19 节响应 JSON 中补充 `tags: string[]` 和 `summary: string` 字段；评估是否统一字段名，若保留 `created_at`/`views` 命名需在文档中明确说明映射关系并确认需求方接受；同步更新 3.8.3 节 `Article` TypeScript 接口；同步更新 articles 表 DDL 补充 `tags` 和 `summary` 列。

### 问题 6
- **问题描述**：Dify Agent 工具定义与需求 6.11 节工具清单严重不符——需求定义了 8+5 个语义化专用工具（含细粒度权限约束），详细设计简化为单一 `execute_SQL` 工具，丢失了细粒度权限约束和 `knowledge_search` 知识库检索能力（Dify 知识库检索不能通过 SQL 实现）。
- **所在位置**：第 5.2.5 节（第 4352-4364 行）和第 5.2.6 节（第 4400 行）
- **严重程度**：严重
- **改进建议**：按需求 6.11 节定义补充 8+5 个专用工具的完整定义（工具名、回调 URL、请求体模板、参数说明、权限约束）；评估是否保留 `execute_SQL` 作为兜底工具；若保留单一 `execute_SQL` 设计作为架构决策偏离，需明确说明理由并补充 `knowledge_search` 工具的独立定义；在 7.3.4 节 `validateRowLevelPermission` 补充 `life_advice` 表写入约束规则。

### 问题 7
- **问题描述**：未采纳需求 8.1 节推荐的 Vant 4 移动端 UI 组件库，导致移动端特有交互组件（Tabbar、ActionSheet、DatetimePicker、PullRefresh 等）需从零开发，CSS 变量体系未与 Vant 4 主题变量建立映射，违反需求 7.1 节要求，前端 package.json 未包含 Vant 4 依赖。
- **所在位置**：第 1.3 节技术选型表（第 110-138 行）
- **严重程度**：严重
- **改进建议**：在 1.3 节技术选型表新增 Vant 4 条目；在 6.3.4 节前端 package.json dependencies 中新增 `"vant": "^4.9.0"`；在 4.5.1 节 CSS 变量定义后补充 Vant 4 主题变量映射表；评估 TabBar.vue、日期筛选器、Toast、Dialog 等组件是否改用 Vant 4 组件替代自研实现。

### 问题 8
- **问题描述**：DoctorChatView.vue 未处理"切换至其他医生对话"的 SSE 连接关闭场景——Vue Router 路由参数变化时同一组件会被复用，onUnmounted 不会触发，用户从医生 A 对话直接跳转到医生 B 对话时原 SSE 连接不会被关闭，违反需求 4.2 节"同时活跃的 SSE 连接数上限为 1"约束。
- **所在位置**：第 4.3 节 DoctorChatView.vue 流程图（第 3312-3336 行）
- **严重程度**：一般
- **改进建议**：在 DoctorChatView.vue 中使用 `watch(() => route.params.id, ...)` 监听路由参数变化并调用 `chatStore.abortActiveConnection()`；或使用 `beforeRouteUpdate` 守卫；在 4.3 节流程图中新增"路由参数变化 → abortActiveConnection → 重新初始化"分支。

### 问题 9
- **问题描述**：Admin.vue 和 AiChatDialog.vue 的 SSE 流程未调用 `abortActiveConnection`/`registerAbortController`，AiChatDialog.vue 缺少独立流程图，三个 SSE 端点的并发控制策略未说明。
- **所在位置**：第 4.3 节 Admin.vue 流程图（第 3571-3595 行）和 AiChatDialog.vue（无独立流程图）
- **严重程度**：一般
- **改进建议**：在 Admin.vue 流程图中补充 `registerAbortController` 和 `abortActiveConnection` 调用；为 AiChatDialog.vue 补充独立的 Mermaid 流程图，明确弹窗关闭时调用 `abortActiveConnection`；在 3.7 节 chatStore 中明确并发控制策略。

### 问题 10
- **问题描述**：`POST /api/plan/generate` 响应结构与需求 6.5 节不一致——需求返回单一 `items` 数组，详细设计拆分为 `diet_plans` + `exercise_plans`；字段名不一致（需求用 `type`/`order`/`time`，详细设计用 `plan_type`/`order_num`/`time_desc`），导致前端 TypeScript 类型定义与需求契约不一致。
- **所在位置**：第 3.2.13 节（第 1727-1755 行）
- **严重程度**：一般
- **改进建议**：评估分组结构的优劣（分组结构对前端渲染更友好可作为合理偏离保留）；统一字段命名（建议保留详细设计命名以避免 SQL 保留字冲突），但在文档中明确说明与需求字段名的映射关系；life_plans 表字段名也应统一并说明偏离。

### 问题 11
- **问题描述**：跨浏览器标签页登录态同步使用 `setToken()` 而非 `setAuth()`，未同步更新 role 和 userInfo，导致用户在标签页 A 切换账号时标签页 B 的 role 和 userInfo 仍是原用户数据，造成 UI 显示与实际 token 状态错乱。
- **所在位置**：第 1.2 节（第 108 行）
- **严重程度**：一般
- **改进建议**：修改 1.2 节描述为 token 变化时调用 `authStore.fetchProfile()` 重新获取用户信息，或新增 `syncFromStorage()` 方法从 localStorage 同步恢复三个字段；在 3.7 节 authStore 接口中补充需求 4.10 节定义的 `setAuth(newToken, newRole, user)` 方法。

### 问题 12
- **问题描述**：NewsView.vue 流程图在"点击生成健康资讯"按钮时直接发起 AI 生成请求，没有调用 `hasAcceptedDisclaimer()` 判定，违反需求 4.11 节"资讯生成"是 AI 功能入口的要求。
- **所在位置**：第 4.3 节 NewsView.vue 流程图（第 3415-3424 行）
- **严重程度**：一般
- **改进建议**：在 NewsView.vue 流程图的"点击生成健康资讯"分支前增加免责声明判定节点（hasAcceptedDisclaimer? → 否则 showDisclaimer → 用户同意后写入 localStorage → 发起请求）；在 4.4.4 节 useUI.ts 免责声明函数调用点说明中补充 NewsView.vue 的调用点。

### 问题 13
- **问题描述**：SSE 事件 `message`/`message_end` 缺少 `created_at` 字段，与需求 6.9 节定义不一致，3.8.7 节 `SSEMessageEvent` 和 `SSEMessageEndEvent` TypeScript 接口也缺少该字段，导致前端无法获取消息创建时间戳。
- **所在位置**：第 3.3 节 SSE 流事件完整格式定义（第 2203-2215 行）
- **严重程度**：一般
- **改进建议**：在 3.3 节 `message` 和 `message_end` 事件 data 字段结构中补充 `created_at: number` 字段；在 3.8.7 节相关 TypeScript 接口中新增 `created_at: number` 成员；在 4.3 节对话流程图中明确前端应将 `created_at` 渲染为消息时间戳。

### 问题 14
- **问题描述**：`difyService.js` blocking 模式读取超时 60s 与需求 7.3 节"所有 AI 接口超时阈值统一为 15 秒"不一致，导致风险预测、方案生成、文章生成等 AI 操作可能等待 60 秒才超时。
- **所在位置**：第 6.3.5 节 difyService.js 行为规格（第 4987-4990 行）
- **严重程度**：一般
- **改进建议**：将 blocking 模式读取超时从 60s 调整为 15s 与需求 7.3 节统一；或在文档中明确说明偏离理由（如 Dify 工作流执行时间可能超过 15 秒）并补充前端加载状态提示覆盖整个超时窗口。

### 问题 15
- **问题描述**：`POST /api/punch` 请求体使用中文枚举值（"饮食"/"已完成"），与需求 6.6 节英文枚举值（"diet"/"completed"）不一致；同样问题存在于 `POST /api/risk/predict`、`RiskPredictRequest` 类型、`LifePlan.plan_type` 类型等，是全局性设计决策偏离。
- **所在位置**：第 3.2.16 节（第 1800-1808 行）
- **严重程度**：一般
- **改进建议**：评估两种方案——方案 A（遵循需求使用英文枚举值，前端 UI 层自行映射展示，无需 mapper.js）；方案 B（保留当前设计，在 1.8 节明确标注为设计决策偏离需求 6.3/6.6 节规范并说明理由）；选择后统一执行并同步更新相关 TypeScript 类型定义。

### 问题 16
- **问题描述**：`AiChatDialog.vue` DOM 结构缺少明确的"登录引导提示"和"跳转登录页按钮"元素，`welcome-tips` 区域用途不明确（混淆欢迎语和登录引导），缺少独立的"跳转至登录页"按钮元素，违反需求 4.8 节要求。
- **所在位置**：第 4.1.1 节 AiChatDialog.vue DOM 树（第 2736-2746 行）
- **严重程度**：一般
- **改进建议**：将 `welcome-tips` 区域拆分为两个独立子区域（已登录用户的欢迎语 + 未登录用户的登录引导含跳转登录按钮）；通过 `v-if="!authStore.token"` 和 `v-else` 控制显隐；在流程图中补充未登录用户点击 FAB 的处理分支。

### 问题 17
- **问题描述**：`admin_logs` 表字段名 `admin_id` 用于记录普通用户 user_id 产生语义混淆——普通用户通过 AI 助手触发 Text2SQL 操作时其 user_id 也会写入 `admin_id` 字段，字段名暗示是管理员 ID 但实际可能存储普通用户 ID。
- **所在位置**：第 2.2 节 admin_logs DDL（第 918-926 行）和第 2.5 节数据字典（第 1243-1252 行）
- **严重程度**：一般
- **改进建议**：将 `admin_id` 字段重命名为 `operator_id`（操作者 ID）消除语义混淆；同步更新 DDL、数据字典、ER 图、7.3.3 节伪代码、3.2.30 节响应字段名；在数据字典中明确说明通过 `operation_type` 字段区分管理员操作和普通用户 Text2SQL 操作。
