# 质量审查诊断报告 — a_v3_req_v2.md

**审查对象**: `a_v3_req_v2.md`（详细设计文档 v7）  
**审查视角**: 实现者视角——需求响应充分度、事实错误/逻辑矛盾、深度与完整性  
**审查轮次**: 第3轮诊断（本轮侧重内部审议未充分覆盖的维度，避免重复验证已确认的技术可行性）

---

## 一、需求响应充分度

### 问题 1：life-plan-generator 输出格式声明存在逻辑矛盾

- **所在位置**: 第5.2.2节"输出格式"段落（第3880-3887行）
- **严重程度**: 一般
- **问题描述**: 文档先声明"Dify 工作流 `life-plan-generator` 以 blocking 模式调用，输出为 JSON 对象"，随后又列出三级解析降级策略（JSON 优先 → 正则提取降级 → LLM 二次调用降级）。这两段描述存在逻辑矛盾——若工作流输出已保证为结构化 JSON，则正则提取和 LLM 二次调用降级策略永远不会被触发；若输出可能为非结构化文本，则开头的"输出为 JSON 对象"声明不成立。实现者无法确定 Dify 工作流输出的真实格式保证级别，也无法判断降级策略是必须实现的安全网还是冗余代码。
- **改进建议**: 明确区分两层语义：(1) 工作流设计目标——LLM 节点应返回 JSON 格式；(2) Express 端的防御性解析策略——因 LLM 输出存在不确定性，需降级策略兜底。将声明改为"以 blocking 模式调用，期望输出为 JSON 对象；因 LLM 输出存在不确定性，Express 端需实现以下三级降级解析策略"。

### 问题 2：SSE 事件类型的 TypeScript 定义与协议定义不一致

- **所在位置**: 第3.3节（第1971-1981行）vs 第3.8.7节（第2397-2442行）vs 第4.4.2节 useSSE.ts（第3457-3469行）
- **严重程度**: 严重
- **问题描述**: 第3.3节"SSE流事件完整格式定义"定义了7种事件类型：`message`, `message_end`, `error`, `workflow_started`, `workflow_finished`, `agent_message`, `agent_thought`。但第3.8.7节 TypeScript 类型定义仅覆盖前3种（`SSEMessageEvent`, `SSEMessageEndEvent`, `SSEErrorEvent`），联合类型 `SSEEvent` 缺失后4种事件的类型定义。第4.4.2节 `useSSE.ts` 的 `streamRequest` 方法中也仅解析 `message`/`message_end`/`error` 三种事件。这意味着前端无法利用 `workflow_started`/`workflow_finished` 事件展示工作流进度，也无法处理 `agent_message`/`agent_thought` 事件展示 Agent 中间推理过程——协议设计中的这些事件实际被丢弃。
- **改进建议**: (1) 在 `types/sse.ts` 中补全 `SSEWorkflowStartedEvent`、`SSEWorkflowFinishedEvent`、`SSEAgentMessageEvent`、`SSEAgentThoughtEvent` 四种类型定义；(2) 更新 `SSEEvent` 联合类型包含全部7种事件；(3) 在 `useSSE.ts` 的 `streamRequest` 事件解析 switch 中增加对4种事件的回调分支；(4) 在 `SSECallback` 类型中新增对应的可选回调函数。

### 问题 3：预查询数据注入失败时的错误处理行为缺失

- **所在位置**: 第3.6节"预查询数据注入说明"段落（第2045-2047行）vs 第5.2.3节/第5.2.4节 vs 第6.3.5节 difyService.js 行为规格
- **严重程度**: 一般
- **问题描述**: `punch-analysis` 和 `health-article-generator` 两个 Dify 工作流依赖 Express 代理层在调用前预查询 SQLite 数据并注入 inputs。但文档未定义预查询失败时的处理行为——若 `punch_in` 表查询时数据库连接断开、或 `user_risk_info` 查询返回空集时，Express 应返回什么错误响应？是中断请求并返回 500，还是传递空数组/空对象给 Dify 工作流让其在 LLM 层面感知"无数据"？
- **改进建议**: (1) 在 difyService.js 行为规格中新增"预查询失败处理"子节，定义各预查询异常的响应行为——数据库错误 → 502 DIFY_ERROR；查询结果为空 → 传递空数组/空对象给 Dify（让 AI 基于"无历史数据"生成通用分析）；(2) 在两个工作流的输入变量定义表中增加"空值行为"列。

### 问题 4：Consultation.vue 共享组件双模式的行为切换逻辑未完整定义

- **所在位置**: 第1.6.1节路由映射表（第408-409行）vs 第4.1.3节 Consultation.vue 组件树 vs 第4.3节 Consultation.vue 流程图
- **严重程度**: 一般
- **问题描述**: `/consultation`（公开）和 `/consultation/doctor/:id`（需认证）共享同一个懒加载组件 `Consultation.vue`，但路由守卫对两条路径的权限要求不同（前者 `requiresAuth=false`，后者 `requiresAuth=true`）。组件内部通过 `route.params.id` 区分"医生列表视图"和"对话视图"。然而流程图（第3037-3038行）将"有id→切换至对话视图"分支放在首位，未处理一个关键边界情况：用户先访问 `/consultation/doctor/1`（进入对话视图），然后通过 Tab 切换离开再返回时，若 `chatStore.conversationMap` 已持久化该医生的 conversation_id，组件是否需要重新验证登录态？实现者缺乏明确的行为契约。
- **改进建议**: 补充 Consultation.vue 的 `onMounted` 伪代码：定义视图判定顺序——(1) 检查 `route.params.id` 是否存在；(2) 若存在且需对话视图，检查 JWT 是否有效（调用 `useAuth().isTokenExpired()`），若无效则跳转登录并附带 redirect；(3) 从 `chatStore.conversationMap` 恢复 conversation_id；(4) 渲染对应视图。

---

## 二、事实错误与逻辑矛盾

### 问题 5：第3.7节 riskFormStore 接口定义中 `RiskFormData` 类型引用不一致

- **所在位置**: 第3.7节（第2102行）vs 第3.8.8节（第2454行）vs 第3.8.4节（第2289-2300行）
- **严重程度**: 一般
- **问题描述**: 第3.7节 riskFormStore 的 `formData` 字段类型声明为 `Partial<RiskFormData>`，但第3.8.4节定义的是 `RiskPredictRequest` 接口（第2289-2300行），并非 `RiskFormData`。在第3.8.8节中（第2454行），`formData` 的类型又被声明为 `Partial<RiskPredictRequest>`。这导致 `RiskFormData` 在文档中未被定义——实现者需要在两种不同命名（`RiskFormData` vs `RiskPredictRequest`）之间做选择。虽然两者指向相同的数据结构，但类型别名的命名不一致会在 IDE 中造成跳转失败。
- **改进建议**: 统一使用 `RiskPredictRequest` 作为表单数据类型，将第3.7节（第2102行）的 `RiskFormData` 改为 `RiskPredictRequest`，或新增 `type RiskFormData = RiskPredictRequest` 类型别名。

### 问题 6：第4.2节状态管理表中 LifePlan.vue 的 sessionStorage 过期时长与数据流不匹配

- **所在位置**: 第4.2节状态管理表（第2996行）vs 第4.3节 LifePlan.vue 流程图（第3092-3116行）
- **严重程度**: 轻微
- **问题描述**: 状态管理表标注 LifePlan.vue 的 sessionStorage 方案缓存过期时间为"30分钟过期"。但流程图显示 LifePlan.vue 在 `onMounted` 中无条件调用 `GET /api/plan/current`（第3096行步骤B），并未先检查 sessionStorage 缓存。这意味着 sessionStorage 缓存过期时间设计实际不会被使用——每次进入页面都会重新请求 API，sessionStorage 缓存写入后从未被读取。这与 Home.vue 明确实现的"先检查缓存→缓存命中直接渲染→缓存未命中才请求 API"的流程不一致。
- **改进建议**: 在 LifePlan.vue 流程图中增加缓存检查步骤——先检查 sessionStorage 中是否存在未过期的方案缓存，命中则直接渲染，未命中才调用 API。或者，如果方案数据需要实时性（每次进入都应获取最新方案），则删除 sessionStorage 该行的"方案缓存"条目，避免误导实现者实现不会被使用的缓存逻辑。

### 问题 7：第1.1节系统架构图中服务器1的 Express :3000 端口与第6.1.2节 Nginx 配置的 upstream 地址不匹配

- **所在位置**: 第1.1节架构图（第50-55行）vs 第6.1.2节服务器1 Nginx 配置（第4383-4384行）
- **严重程度**: 轻微
- **问题描述**: 架构图显示服务器1上的 Nginx :80 通过 `proxy_pass http://127.0.0.1:3000` 代理到 Express。但第6.1.1节服务器2/3的 Nginx 中 upstream `backend_api` 指向的是 `10.0.1.10:3000`（服务器1的内网 IP）。这里存在潜在的不一致：当服务器2/3通过 upstream 代理请求到服务器1的 :3000 端口时，Express 必须监听在 `0.0.0.0:3000`（而非 `127.0.0.1:3000`）才能接受来自服务器2/3的请求。第6.3.1节 server.js 中 `app.listen(PORT, '0.0.0.0', ...)` 已正确设置为 `0.0.0.0`，但第1.1节架构图的文字描述"Express :3000"和连接线未明确标注监听地址，容易让部署者误以为 Express 仅监听 localhost。
- **改进建议**: 在第1.1节架构图文字说明中补充"Express 监听 0.0.0.0:3000，接受来自本机 Nginx 和服务器2/3 Nginx upstream 的请求"。

---

## 三、深度与完整性（实现者视角）

### 问题 8：Consultation.vue 的"清除对话"按钮行为未定义后端接口

- **所在位置**: 第4.1.3节 Consultation.vue 组件树（第2578行）vs 第3.1.4节 API 端点清单
- **严重程度**: 一般
- **问题描述**: Consultation.vue 对话视图头部包含 `btn-delete`（清空对话）按钮（第2578行），但第3.1.4节医师对话相关端点清单中无任何删除/清除对话的 API 端点（仅有 `POST /api/chat/doctor/:id` 发送消息和 `GET /api/chat/doctor/:id/conversations` 获取会话列表）。清空对话操作是清空前端消息列表（纯本地操作），还是需要调用 Dify API 删除会话（需新增 Express 代理端点）？实现者无法确定按钮行为。类似的，Admin.vue 对话视图缺少清除对话按钮（与 Consultation.vue 不对称）但 AI 助手弹窗 AiChatDialog.vue 也没有——三种对话场景的"清空对话"功能设计不统一。
- **改进建议**: (1) 明确"清空对话"按钮的行为——是纯前端操作（清空 chatStore.messages 并调用 `removeConversationId`），还是需要后端删除 Dify 会话；(2) 若需后端支持，新增 `DELETE /api/chat/doctor/:id/conversations/:conversation_id` 端点；(3) 为 Admin.vue 和 AiChatDialog.vue 补充清空对话功能入口。

### 问题 9：Express Dify 代理端点的请求体组装逻辑不够详尽

- **所在位置**: 第3.6节 Express代理层请求参数映射表（第2032-2047行）vs 第3.2.32节 Dify 代理端点请求体（第1956-1965行）
- **严重程度**: 一般
- **问题描述**: 第3.6节映射表定义了 Express 端点字段到 Dify API 参数名的逐一映射（如 `message → query`、`age → inputs.age`），但未定义完整请求体的组装逻辑。具体缺失：
  1. `user` 参数（Dify API 要求必填的 `user` 字段）如何从 JWT payload 中提取并注入请求体
  2. 当 Express 端点请求体中存在 Dify 不需要的字段（如 risk/predict 请求体中的 `diabetes_history` 的枚举值验证由 Express 完成）时，是否全部透传还是需要过滤
  3. 嵌套对象（如 `health_info`）是整体透传到 `inputs.health_info` 还是展开为 `inputs.health_info.age`、`inputs.health_info.gender` 等独立字段
- **改进建议**: 新增"请求体组装流程"伪代码子节，包含：(1) 从 `req.body` 提取字段 → (2) 按映射表转换为 Dify 参数 → (3) 注入 JWT 提取的 `user` 字段 → (4) 注入 `response_mode` → (5) 注入预查询数据（如适用）→ (6) 调用 difyService。

### 问题 10：punch-analysis 工作流的 date_range 参数传递路径未贯通

- **所在位置**: 第3.1.6节（第1144行 GET /api/punch/analysis）vs 第3.6节映射表 vs 第5.2.4节（第3955行）
- **严重程度**: 一般
- **问题描述**: 第3.1.6节定义 `GET /api/punch/analysis` 端点但不接受任何查询参数（无 date_range）。第5.2.4节 punch-analysis 工作流输入变量中定义了可选的 `date_range` 参数（包含 start/end）。但整个数据通路存在断裂——前端如何将日期筛选条件传递给 punch-analysis 工作流？如果前端 Punch.vue 页面有日期筛选器（第4.1.8节中确实有 start-date/end-date input），那么 `GET /api/punch/analysis` 端点需要支持 `?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD` 查询参数，Express 端需将其转换为 `inputs.date_range` 并注入 Dify 请求。当前端点定义缺少这两个查询参数。
- **改进建议**: (1) 更新 `GET /api/punch/analysis` 端点定义为 `GET /api/punch/analysis?startDate=&endDate=`；(2) 在第3.6节映射表中新增 `startDate, endDate → inputs.date_range.{start, end}` 映射规则；(3) 在第4.3节 Punch.vue 流程图中明确分析区域数据请求时携带当前日期筛选器的 startDate/endDate 参数。

### 问题 11：AI 助手健康建议生成的触发机制不明确

- **所在位置**: 第5.2.5节 Skill 6 描述（第3989行）vs 第3.1.9节端点清单 vs 第4.1.10节 AiChatDialog.vue
- **严重程度**: 一般
- **问题描述**: diabetes-assistant-agent 的 Skill 6 描述了健康建议生成功能——"当用户在对话中表达健康管理意图时，分析用户数据后生成结构化建议，通过 execute_SQL 写入 life_advice 表"。此设计依赖 Dify Agent 在自然语言对话中自主判断用户意图并触发建议生成。但文档未定义：(1) 建议生成是否需要用户确认（Agent 是直接写入还是先展示建议预览等待确认）；(2) 生成的建议如何与 `GET /api/assistant/advice` 端点关联——Agent 通过 `execute_SQL` 写入 `life_advice` 表后，`user_id` 从何处获取（依赖 `{{user}}` 变量透传）；(3) 若 Agent 多次为同一用户生成建议，life_advice 表以追加模式存储（无去重逻辑），用户可能看到重复建议。
- **改进建议**: (1) 补充 Skill 6 的交互流程：Agent 先展示建议预览 → 用户确认 → Agent 通过 execute_SQL 写入；(2) 在 life_advice 表中新增 `UNIQUE(user_id, title)` 约束或在前端 GET /api/assistant/advice 中增加去重逻辑（按 title+content 哈希）；(3) 明确健康建议写入时的 user_id 来源（需要 Dify `{{user}}` 变量透传验证通过后才能正确执行）。

### 问题 12：管理员强制密码修改完成后 authStore 状态清除与路由跳转的竞态条件未处理

- **所在位置**: 第1.6.2节路由守卫（第449行）vs 第4.3节 ChangePassword.vue 流程图（第3344-3350行）
- **严重程度**: 轻微
- **问题描述**: ChangePassword.vue 流程图中，改密成功后调用 `authStore.mustChangePassword = false`（第3348行），然后 `router.push /admin`（第3351行）。问题在于 Pinia 状态更新和路由跳转之间存在短暂的竞态窗口——如果 Vue Router 的 beforeEach 守卫在 Pinia 响应式传播完成之前被触发，守卫仍可能读到 `mustChangePassword=true`，导致跳转到 `/admin` 被拦截并重定向回 `/change-password`，形成死循环。
- **改进建议**: (1) 将路由跳转改为 `router.push('/admin')` 延迟执行（使用 `await nextTick()` 确保 Pinia 状态已传播）；或 (2) 在路由守卫中增加例外——若当前已在 `/change-password` 路径且 `mustChangePassword=true`，允许停留在当前页面（不强制跳转，因为用户正在处理改密）；(3) 推荐方案：路由守卫中增加 `if (to.path === '/change-password') return next()` 优先通过，避免死循环。

### 问题 13：前端与后端的 JWT 过期时间缺乏明确协调机制

- **所在位置**: 第7.1节 JWT鉴权流程（第4773行 `expiresIn:'24h'`）vs 第4.4.1节 useApi.ts 401 拦截（第3386行）vs 第4.3节 App.vue 流程图
- **严重程度**: 轻微
- **问题描述**: JWT Token 有效期为 24 小时，前端 Axios 响应拦截器在收到 401 时清除 Token。但前端未主动检查 Token 过期（useAuth.ts 中的 `isTokenExpired` 工具函数存在但未被调用）。这意味着：(1) 用户在 Token 过期前打开页面，过期后继续操作——前端仍使用过期 Token 发送请求，每次请求都产生一次失败的 HTTP 往返后才提示重新登录；(2) SSE 流式连接在 Token 过期后仍保持（流式请求建立后不再验证 Token），存在安全窗口。实现者缺乏关于"是否需要在 App.vue 中增加定时 Token 过期检查"的指导。
- **改进建议**: 在 App.vue 流程图中增加"Token 过期主动检查"逻辑——使用 `setInterval` 每5分钟检查一次 `isTokenExpired(authStore.token)`，若即将过期（剩余时间<5分钟）则展示非阻断续期提示或提前清除 Token。或在路由守卫 beforeEach 中增加 Token 过期检查，在导航前拦截而非等 API 返回 401。

---

## 四、整体质量评价

经过 v1 至 v7 共 7 轮修订，文档已修复了前序迭代中识别的大部分问题（包括 iframe→Vue3 SPA 架构替换、TypeScript 类型定义补全、验收标准新增等重大改进）。当前版本（v7）已达到基本可用的详细设计深度——数据库 DDL、API Schema、Dify 工作流提示词、前端组件树和流程图、部署 Nginx 配置均给出了可执行的规格。

本轮审查从**实现者视角**识别的 13 个问题（2个严重、8个一般、3个轻微）主要集中在以下几个方面：
- **SSE 事件类型的协议-类型-代码三层不一致**（问题2），会导致 workflow_started/agent_message 等事件被静默丢弃；
- **数据通路的不贯通**（问题3、10、11），punch-analysis 的 date_range 传递、预查询失败处理、健康建议生成触发机制存在黑盒；
- **共享组件的双模式行为边界模糊**（问题4、8），Consultation.vue 的列表/对话模式切换需更明确的生命周期逻辑；
- **逻辑矛盾的残余**（问题1、5、6），输出格式声明与降级策略矛盾、类型命名不一致、未使用的缓存设计。

这些问题在**内部设计-验证/实现-审查循环中不易暴露**——因为内部审议侧重技术可行性（API 是否存在、架构是否合理），而上述问题涉及跨章节的连贯性、类型系统的完整性、以及实现者"拿着文档就能写代码"的可用性。建议在 v8 修订中优先处理 2 个严重问题和 8 个一般问题。

---

*审查完成时间: 2026-06-24*
