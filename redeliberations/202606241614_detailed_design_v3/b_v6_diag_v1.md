# 质量审查报告 — 详细设计 v6 (a_v6_copy_from_v5)

> 本报告基于第 6 轮迭代审查。前 5 轮迭代已识别并修复 47 个问题（含枚举值语言、字段缺失、SSE 连接控制、双认证模式、TypeScript 类型契约等）。本轮审查侧重于**需求响应充分度、整体深度和完整性、工程实施可执行性**，聚焦内部审议（设计-验证/执行-审查循环）未充分覆盖的维度。

---

## 严重问题（Critical）

### 问题 1：difyService.js blocking 模式自动重试直接违反需求 7.3 节"不自动重试"原则

**问题描述**：第 6.3.5 节 difyService.js 行为规格表中，blocking 模式（用于 POST /api/risk/predict、POST /api/plan/generate、POST /api/articles/generate）的"重试次数"明确标注为 `1`，"重试间隔"为 `2s`。但需求 7.3 节"响应超时处理"段明确要求："**不自动重试，避免重复提交请求加重服务端压力**"。这是与需求的直接矛盾。

更严重的是，这三个端点均属于"AI 内容生成持久化路径"——Express 调用 Dify 工作流后将结果写入数据库。若 Dify 实际已成功执行工作流但响应因网络抖动未在 15s 内到达 Express，Express 自动重试将触发第二次工作流调用，可能导致：
- `user_risk_info` 表写入重复的风险预测记录
- `life_plans` 表写入重复方案项（且由于 plan_id 生成机制未明确，新方案的 plan_id 可能与旧方案不同，导致 is_active 逻辑过期机制无法正确清理）
- `articles` 表写入重复文章

**所在位置**：第 6.3.5 节"超时与重试策略"表格（约第 5341-5344 行）

**严重程度**：严重

**改进建议**：
1. 将 blocking 模式的重试次数从 `1` 改为 `0`，与需求 7.3 节"不自动重试"对齐。
2. 在表格下方补充说明："blocking 模式不自动重试的设计依据需求 7.3 节'避免重复提交请求加重服务端压力'原则。由于 risk/predict、plan/generate、articles/generate 三个端点均有数据库写入副作用，自动重试可能导致重复记录。超时后由前端展示超时提示并允许用户手动重试。"
3. 若项目组评估后确认需要重试，需在文档中明确说明此为对需求 7.3 节的偏离决策，并补充幂等性保护机制（如基于 user_id + 请求时间窗口的去重）。

---

### 问题 2：`/api/admin/execute` 路由处理器未实现 `tool_name` 分发逻辑，与 5.2.5/5.2.6 节工具定义直接矛盾

**问题描述**：第 5.2.5 节 diabetes-assistant-agent 工具设计说明明确指出："所有 HTTP 回调工具的回调 URL 统一为 `POST {EXPRESS_PUBLIC_URL}/api/admin/execute`...由 Express 端点根据 `tool_name` 参数分发至对应的参数化查询处理器"；5.2.6 节 admin-manager-agent 亦有同样描述。各专用工具的回调请求体模板均包含 `tool_name` 字段（如 `{"tool_name":"query_user_profile","user_id":"{{user}}","api_key":"..."}`）。

然而：
1. **3.2.29 节 POST /api/admin/execute 请求体定义**（约第 2200-2216 行）只定义了 `sql`、`user_id`、`api_key` 三个字段，**完全没有 `tool_name` 字段**。
2. **7.3.3 节路由处理器核心逻辑伪代码**（约第 5606-5666 行）中 `const { sql } = req.body;` 只解构 `sql` 字段，**没有任何 `tool_name` 分发逻辑**。路由处理器直接对 `sql` 字段执行白名单校验和 `db.prepare(sql).all()/run()`，与"专用工具走参数化查询处理器"的设计完全脱节。

这意味着按当前路由处理器实现，专用工具（如 `query_user_profile`、`write_health_advice`）回调时携带的 `tool_name` 字段会被完全忽略，而请求体中可能没有 `sql` 字段（专用工具的回调请求体模板不含 `sql`），导致 `const { sql } = req.body` 解构得到 `undefined`，后续 `validateRowLevelPermission(undefined, operatorId)` 和 `db.prepare(undefined)` 将抛出运行时错误。

**所在位置**：
- 3.2.29 节 POST /api/admin/execute 请求体（约第 2200-2216 行）
- 7.3.3 节路由处理器伪代码（约第 5606-5666 行）
- 与 5.2.5 节（约第 4664-4684 行）、5.2.6 节（约第 4721-4737 行）工具定义矛盾

**严重程度**：严重

**改进建议**：
1. 在 3.2.29 节请求体定义中补充 `tool_name` 字段（可选，专用工具回调时必填），并说明 `tool_name` 与 `sql` 字段的互斥关系：专用工具回调携带 `tool_name` + 业务参数，execute_SQL 兜底工具回调携带 `sql`。
2. 在 7.3.3 节路由处理器伪代码中补充 `tool_name` 分发分支：若 `req.body.tool_name` 存在，按 `tool_name` 路由至对应的参数化查询处理器（每个处理器有固定的 SQL 模板，仅参数化填充 user_id 等参数）；若不存在但有 `sql` 字段，走 execute_SQL 兜底路径。
3. 补充专用工具参数化查询处理器的伪代码示例（至少 1-2 个，如 `query_user_profile` 对应 `SELECT id, username, role, avatar, created_at FROM users WHERE id = ?`，参数为 operatorId）。

---

## 一般问题（Major）

### 问题 3：`POST /api/plan/generate` 服务端处理流程未说明旧方案逻辑过期和 plan_id 生成机制，存在多套活跃方案风险

**问题描述**：需求 6.5 节明确说明 POST /api/plan/generate 的服务端处理流程应"将方案项逐条解析并写入 life_plans 表"。v14 修订在 3.2.14 节 PUT /api/plan/adjust 中详细说明了 plan_id 的语义和处理流程（先逻辑过期旧方案组 `UPDATE life_plans SET is_active=0 WHERE user_id=? AND plan_id=?`，再生成新方案组），但 **3.2.13 节 POST /api/plan/generate 没有对应的服务端处理流程说明**，关键问题：

1. **旧方案逻辑过期**：用户已有活跃方案时再次调用 POST /api/plan/generate，是否需要先将旧方案标记为 is_active=0？文档未说明。若不处理，用户将存在多套活跃方案。
2. **plan_id 生成时机**：3.2.14 节说"推荐取本批首条方案项的 id 作为 plan_id，或使用应用层自增序列"。但首条方案项的 id 是 AUTOINCREMENT 生成的，需先 INSERT 才能获取 id，再 UPDATE 该条的 plan_id，或先 INSERT 所有条目再 UPDATE 所有 plan_id。这一实施细节未说明，实现者可能采取不同方案导致行为不一致。
3. **多套活跃方案的防护**：DDL 中 is_active 默认为 1，没有约束防止同一 user_id 存在多个 is_active=1 的方案组。POST /api/plan/generate 如果不主动清理旧活跃方案，将导致数据不一致。

**所在位置**：
- 3.2.13 节 POST /api/plan/generate（约第 1771-1826 行），缺少服务端处理流程段落
- 3.2.15 节 GET /api/plan/current 的 SQLite 查询 `SELECT * FROM life_plans WHERE user_id = ? AND is_active = 1 ORDER BY plan_type, order_num`（约第 1866 行）未按 plan_id 分组，若存在多套活跃方案将返回混乱数据
- 2.2 节 life_plans DDL（约第 918-931 行）缺少防止多套活跃方案的约束

**严重程度**：一般

**改进建议**：
1. 在 3.2.13 节补充"服务端处理流程"段落，明确说明：(a) 调用 Dify 工作流前先将用户当前活跃方案逻辑过期 `UPDATE life_plans SET is_active=0, updated_at=datetime('now','localtime') WHERE user_id=? AND is_active=1`；(b) plan_id 生成策略（推荐使用应用层自增序列或 `SELECT COALESCE(MAX(plan_id), 0) + 1 FROM life_plans WHERE user_id=?`，避免 INSERT-UPDATE 两步操作的事务复杂度）；(c) 所有新方案项写入时 plan_id 统一填入新生成的值。
2. 评估 3.2.15 节 SQLite 查询是否需要限制为最新一套活跃方案（如 `AND plan_id = (SELECT MAX(plan_id) FROM life_plans WHERE user_id=? AND is_active=1)`）。
3. 在 2.5 节 life_plans 数据字典中补充业务约束说明："同一 user_id 同时仅允许存在一套 is_active=1 的方案组，由 POST /api/plan/generate 和 PUT /api/plan/adjust 的服务端处理流程保证"。

---

### 问题 4：punch_in.plan_id 在 DDL 中为 DEFAULT NULL，但需求 6.6 节 API 契约要求必填，存在契约矛盾

**问题描述**：需求 6.6 节 POST /api/punch 请求体参数表中 `plan_id` 标注为"必填"，需求 5 节 punch_in 表定义中 `plan_id INTEGER, FOREIGN KEY REFERENCES life_plans(id)`。但详细设计 2.2 节 punch_in DDL 中为 `plan_id INTEGER DEFAULT NULL`，允许为空。

DDL 中 `ON DELETE SET NULL` 外键约束说明 plan_id 在关联的 life_plans 记录被删除时会被设为 NULL，这是合理的。但 DEFAULT NULL 意味着 INSERT 时不传 plan_id 也能成功，与 API 必填要求矛盾。

文档未说明此偏离的理由，也未说明何种场景下 plan_id 可以为 NULL（如管理员通过 Text2SQL 修正异常打卡记录时是否允许不传 plan_id）。

**所在位置**：
- 2.2 节 punch_in DDL（约第 945-957 行）：`plan_id INTEGER DEFAULT NULL`
- 与需求 6.6 节"plan_id | number | 是 | 关联的方案项ID"矛盾
- 与 3.2.16 节 POST /api/punch 请求体（约第 1883 行）`"plan_id": 1` 必填标注矛盾

**严重程度**：一般

**改进建议**：
- 方案 A（遵循需求，建议）：将 DDL 改为 `plan_id INTEGER NOT NULL`，但保留 `ON DELETE SET NULL` 外键约束。这样 INSERT 时强制要求 plan_id，但 life_plans 记录删除时 punch_in.plan_id 自动设为 NULL 保留打卡历史。需在文档中说明 NOT NULL 约束与 ON DELETE SET NULL 的协作机制。
- 方案 B（说明偏离理由）：若确认存在 plan_id 可为空的场景（如管理员 Text2SQL 创建无关联方案项的打卡记录），保留 DEFAULT NULL，但在 2.5 节 punch_in 数据字典中明确说明"plan_id 在 POST /api/punch 接口层为必填，DDL 层允许 NULL 是为了支持 ON DELETE SET NULL 外键约束和管理员 Text2SQL 场景"。

---

### 问题 5：Dify 工作流输出解析失败的处理策略仅 life-plan-generator 有定义，diabetes-risk-prediction 和 health-article-generator 缺失

**问题描述**：5.2.2 节 life-plan-generator 工作流有完整的三层降级解析策略（JSON 优先 → 正则提取降级 → LLM 二次调用降级）和解析失败错误响应 `PLAN_PARSE_ERROR`。但：

1. **5.2.1 节 diabetes-risk-prediction 工作流**：输出结构定义了 JSON 字段（risk_score、risk_level 等），但未说明 Dify 工作流输出为 JSON 还是自然语言文本，未定义 Express 端解析失败时的处理策略。若 Dify 返回自然语言文本而非 JSON，`JSON.parse` 失败后如何处理？
2. **5.2.3 节 health-article-generator 工作流**：同样未说明输出格式和解析失败处理。文章正文为 Markdown 文本，但 title/cover/tags/summary 等结构化字段的解析策略未定义。

这三个工作流都属于"AI 内容生成持久化路径"，应有一致的解析失败处理框架。当前只有 life-plan-generator 有完整定义，另两个存在关键技术决策遗漏。

**所在位置**：
- 5.2.1 节 diabetes-risk-prediction（约第 4392-4455 行），缺少输出格式和解析策略
- 5.2.3 节 health-article-generator（约第 4541-4577 行），缺少输出格式和解析策略
- 与 5.2.2 节 life-plan-generator（约第 4530-4537 行）的完整定义对比

**严重程度**：一般

**改进建议**：
1. 在 5.2.1 节补充输出格式说明（JSON 对象）和解析失败处理策略（至少定义 JSON.parse 失败时的错误响应，如 `RISK_PARSE_ERROR`）。
2. 在 5.2.3 节补充输出格式说明（JSON 对象含 title/cover/content/category/tags/summary 字段）和解析失败处理策略（如 `ARTICLE_PARSE_ERROR`）。
3. 考虑在 6.3.5 节 difyService.js 行为规格中补充统一的"工作流输出解析框架"段落，定义通用的三层降级策略模板，各工作流在此基础上定制字段映射。

---

### 问题 6：POST /api/articles/generate 和 POST /api/plan/generate 缺少幂等性保护，用户重复提交可能导致重复生成

**问题描述**：POST /api/articles/generate 和 POST /api/plan/generate 均为有数据库写入副作用的 AI 生成端点。用户快速双击"生成文章"或"生成方案"按钮、网络重试、或前端未正确禁用提交按钮时，可能触发多次生成请求，导致：
- articles 表写入多篇重复文章
- life_plans 表写入多套方案（即使有问题 3 中的旧方案逻辑过期，仍可能在极短时间内并发写入两套活跃方案）

文档中 4.3 节 LifePlan.vue 流程图和 NewsView.vue 流程图仅依赖前端"按钮 loading 态"防重复，但前端防重复无法防止网络层重试。7.3.3 节 Admin.vue 流程图提到 `isSubmitting` 防重复标志，但那是组件级状态，不是服务端幂等性。

**所在位置**：
- 3.2.13 节 POST /api/plan/generate（约第 1771 行）
- 3.2.21 节 POST /api/articles/generate（约第 2038 行）
- 4.3 节 LifePlan.vue 流程图（约第 3560 行）和 NewsView.vue 流程图（约第 3575 行）

**严重程度**：一般

**改进建议**：
1. 在 3.2.13 节和 3.2.21 节服务端处理流程中补充幂等性检查：如同一 user_id 在 N 秒内（如 30s）已有生成请求在进行中或刚完成，拒绝重复请求并返回 `409 CONFLICT` 错误码。
2. 或采用客户端幂等键方案：前端生成唯一 requestId，服务端基于 requestId + user_id 去重。
3. 在 3.4 节错误码枚举表中补充 `CONFLICT` 错误码的触发场景说明（已存在但未覆盖此场景）。

---

### 问题 7：admin_logs 日志写入与 SQL 执行的事务一致性未说明，存在审计日志丢失风险

**问题描述**：7.3.3 节路由处理器伪代码中，SQL 执行（`db.prepare(sql).all()/run()`）和日志写入（`insertAdminLog(...)`）是两个独立的数据库操作。若 SQL 执行成功但 insertAdminLog 失败（如 operation_content 字段过长触发 SQLite 错误、或进程在两次操作间崩溃），将导致 SQL 操作已生效但审计日志丢失，违反需求 4.9 节"管理员的所有数据库修改操作均可追溯...操作日志不可删除"的硬性要求。

better-sqlite3 是同步驱动，支持事务（`db.transaction(() => { ... })()`），但文档未说明是否将 SQL 执行和日志写入包裹在同一事务中。

**所在位置**：7.3.3 节路由处理器伪代码（约第 5654-5665 行）

**严重程度**：一般

**改进建议**：
1. 在 7.3.3 节伪代码中将 SQL 执行和 insertAdminLog 包裹在 better-sqlite3 事务中：
   ```javascript
   const result = db.transaction(() => {
     const r = sqlType === 'SELECT' ? db.prepare(sql).all() : db.prepare(sql).run();
     if (sqlType !== 'SELECT') {
       insertAdminLog(operatorId, authMode==='dify_callback'?'user_text2sql':getOpType(sql), sql, '成功');
     }
     return r;
   })();
   ```
2. 补充事务失败时的错误处理：事务整体回滚，返回 500 INTERNAL_ERROR，SQL 操作不生效，日志也不写入（但可在 Express 日志中记录失败信息便于排查）。
3. 说明 SELECT 操作是否需要记录日志（当前伪代码对所有操作都调用 insertAdminLog，但 SELECT 无副作用，是否需要审计可由项目组决定）。

---

### 问题 8：doctor_information 表 chat_token 字段明文存储 Dify API Key，缺少加密策略说明

**问题描述**：doctor_information 表的 chat_token 字段存储 Dify 聊天助手的 API Secret（格式 `app-XXX`），属于敏感凭证。2.2 节 DDL 中为 `chat_token TEXT NOT NULL` 明文存储，7.6 节 SQL 注入防护表中提到"SQLite 仅 Express 进程访问"，但未说明 chat_token 的加密策略。

需求 7.5 节环境变量管理段落明确："所有敏感信息（密钥、API Secret）不得硬编码在源代码中或提交至版本控制系统"。chat_token 虽然存在数据库中而非源代码，但仍属于敏感信息，若数据库文件泄露（如备份文件未加密传输、服务器被入侵），所有医生的 Dify API Key 将直接暴露。

**所在位置**：
- 2.2 节 doctor_information DDL（约第 858 行）：`chat_token TEXT NOT NULL`
- 2.5 节 doctor_information 数据字典（约第 1192 行）
- 7.5 节 XSS 防御、7.6 节 SQL 注入防护均未涉及 chat_token 加密

**严重程度**：一般

**改进建议**：
- 方案 A（推荐，加密存储）：chat_token 列改为存储 AES-256-GCM 加密后的密文，Express 读取后用 JWT_SECRET 派生密钥解密使用。在 2.5 节数据字典中说明加密策略，在 7 节安全设计中补充"敏感字段加密"段落。
- 方案 B（说明风险并接受）：若项目组评估后认为实训项目风险可接受，在 2.5 节 chat_token 字段说明中补充风险提示："chat_token 为明文存储，依赖 SQLite 文件级访问控制（仅 Express 进程可读）保障安全。生产环境应考虑字段级加密。"
- 无论哪种方案，应在数据库备份策略（6.7 节）中补充说明备份文件包含 chat_token 敏感数据，备份存储需访问控制。

---

### 问题 9：POST /api/auth/logout 端点设计不完整，未说明登出时需清理的前端状态

**问题描述**：3.2.3 节 POST /api/auth/logout 仅有简单的请求体（无）和响应体（`{"success": true, "message": "已登出"}`），未说明登出时需清理的前端状态。4.3 节 Profile.vue 流程图中登出逻辑仅调用 `authStore.logout()` 清除 Token + user info，但以下状态未说明如何处理：

1. **chatStore 中的所有 conversation_id**：doctorConversations Map、assistantConversationId、adminConversationId 是否需要清理？需求 4.10 节 chatStore 定义中 clearDoctorConversation/clearAssistantConversation 是用户手动删除单个会话的方法，登出时是否批量清理未说明。
2. **活跃的 SSE 连接**：登出时是否需要调用 `chatStore.abortActiveConnection()` 中止当前活跃的 SSE 连接？若不中止，SSE 连接可能因 Token 失效而由服务端断开，但前端可能仍显示"对话进行中"状态。
3. **riskFormStore 中的表单数据**：登出时是否需要调用 `riskFormStore.reset()` 清除 sessionStorage 中的风险预测表单数据？
4. **sessionStorage 中的页面级缓存**：Home.vue（1小时）、LifePlan.vue（30分钟）、News.vue（5分钟）的缓存是否需要清理？

**所在位置**：
- 3.2.3 节 POST /api/auth/logout（约第 1513-1523 行）
- 4.3 节 Profile.vue 流程图登出分支（约第 3698-3702 行）
- 1.5.2 节 authStore.logout() 伪代码（约第 367-375 行）仅清理 token/role/user

**严重程度**：一般

**改进建议**：
1. 在 3.2.3 节补充"前端登出完整流程"段落，明确登出时需按以下顺序清理：
   - 调用 `chatStore.abortActiveConnection()` 中止活跃 SSE 连接
   - 调用 `chatStore.clearDoctorConversation` / `clearAssistantConversation` 清理所有会话（或新增 `clearAllConversations()` 方法）
   - 调用 `riskFormStore.reset()` 清除表单数据
   - 调用 `authStore.logout()` 清除 token/role/user 并跳转首页
   - 可选：清理 sessionStorage 中的页面级缓存
2. 在 3.7 节 chatStore 接口中评估是否需要新增 `clearAllConversations(): void` 方法用于登出场景批量清理。
3. 在 4.3 节 Profile.vue 流程图的登出分支中补充上述清理步骤。

---

## 轻微问题（Minor）

### 问题 10：`POST /api/risk/predict` 响应的 `created_at` 字段标注为"详细设计扩展字段"，但 TypeScript 类型中为必填字段，存在标注与契约不一致

**问题描述**：3.2.7 节 POST /api/risk/predict 响应 JSON 中 `created_at` 字段注释为"详细设计扩展字段，需求 6.3 节未定义；保留用于前端展示预测时间"。但 3.8.4 节 RiskPredictResponse TypeScript 接口定义中 `created_at: string` 是必填成员（非可选 `?`）。

"扩展字段"通常意味着可选，但类型契约要求必填，前端按 RiskPredictResponse 类型编码时若后端未返回 created_at 将导致运行时为 undefined 但 TypeScript 编译期不报错。

**所在位置**：
- 3.2.7 节响应 JSON（约第 1625 行）：`"created_at": "2026-06-23T14:30:00"  // 详细设计扩展字段`
- 3.8.4 节 RiskPredictResponse（约第 2688 行）：`created_at: string;`（必填）

**严重程度**：轻微

**改进建议**：
- 方案 A（统一为必填）：删除 3.2.7 节的"扩展字段"注释，在 3.8.4 节 RiskPredictResponse 中保留 `created_at: string`，并在 3.2.7 节说明"created_at 由 Express 端点从 user_risk_info.created_at 列返回，用于前端展示预测时间"。
- 方案 B（统一为可选）：将 3.8.4 节改为 `created_at?: string`，3.2.7 节保留"扩展字段"标注。

---

### 问题 11：articles 表与 life_advice 表的 tags 字段约束不一致（NOT NULL vs 允许 NULL）

**问题描述**：articles 表和 life_advice 表都有 tags 字段存储 JSON 数组字符串，用途相同，但 DDL 约束不一致：
- articles 表（2.2 节约第 870 行）：`tags TEXT NOT NULL DEFAULT '[]'`
- life_advice 表（2.2 节约第 938 行）：`tags TEXT DEFAULT '[]'`（无 NOT NULL）

articles 表强制 NOT NULL，life_advice 表允许 NULL。若 life_advice 记录写入时未提供 tags，字段值为默认值 '[]'（而非 NULL），但若通过 Text2SQL 的 INSERT 语句显式传入 NULL，SQLite 不会拦截。1.8.4 节的 JSON 序列化规范说明了两表的转换逻辑一致，但约束不一致可能导致 life_advice.tags 出现 NULL 值时 `JSON.parse(null)` 抛出错误（1.8.4 节虽有"NULL 安全降级为空数组"说明，但增加了不必要的复杂度）。

**所在位置**：
- 2.2 节 articles DDL（约第 870 行）
- 2.2 节 life_advice DDL（约第 938 行）

**严重程度**：轻微

**改进建议**：将 life_advice 表的 tags 字段改为 `tags TEXT NOT NULL DEFAULT '[]'`，与 articles 表保持一致，统一两个表的约束规范。

---

## 整体评价

本轮审查识别 11 个问题（2 严重 / 7 一般 / 2 轻微），均集中于前 5 轮迭代未充分覆盖的工程实施维度。产出文档经 14 轮内部修订后，在架构一致性、字段契约、类型定义等维度已达到较高质量；但在**服务端处理流程的完整性**（plan/generate 缺少处理流程、logout 缺少状态清理）、**文档内部一致性**（tool_name 分发逻辑断裂、重试策略与需求矛盾）、**工程实施的可执行性**（幂等性保护、事务一致性、敏感数据加密）等维度仍存在关键缺口。

修复优先级建议：
1. **立即修复**（严重）：问题 1（重试策略违反需求）、问题 2（tool_name 分发逻辑断裂）
2. **修复后可进入开发**（一般）：问题 3（plan/generate 处理流程）、问题 7（事务一致性）、问题 9（logout 状态清理）
3. **开发中并行修复**（一般/轻微）：问题 4-6、8、10、11

修复完成后，建议补充一轮针对"服务端处理流程完整性"的专项审查，确保所有有数据库写入副作用的 POST/PUT 端点都有完整的服务端处理流程段落。
