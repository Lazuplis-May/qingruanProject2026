根据以下审查结果，迭代上一轮的产出，形成新版的文件，从而更好地满足用户需求。

## 当前审查结果

本轮审查（第 6 轮诊断）识别 11 个问题（2 严重 / 7 一般 / 2 轻微），均集中于前 5 轮迭代未充分覆盖的工程实施维度。质询报告确认诊断结论为 LOCATED（所有问题证据充分、逻辑完整、覆盖完备），可作为可信依据直接采纳。

### 严重问题（Critical）

**问题 1：difyService.js blocking 模式自动重试直接违反需求 7.3 节"不自动重试"原则**

- 所在位置：第 6.3.5 节"超时与重试策略"表格（约第 5341-5344 行）
- 问题描述：blocking 模式（用于 POST /api/risk/predict、POST /api/plan/generate、POST /api/articles/generate）的重试次数标注为 1，重试间隔 2s，与需求 7.3 节"不自动重试，避免重复提交请求加重服务端压力"直接矛盾。三个端点均有数据库写入副作用，自动重试可能导致 user_risk_info、life_plans、articles 表写入重复记录。
- 改进建议：将 blocking 模式重试次数从 1 改为 0，与需求 7.3 节对齐；在表格下方补充设计依据说明（blocking 模式不自动重试是因为三个端点均有数据库写入副作用，超时后由前端展示超时提示并允许用户手动重试）；若项目组评估后确认需要重试，需明确说明此为对需求 7.3 节的偏离决策，并补充幂等性保护机制（如基于 user_id + 请求时间窗口的去重）。

**问题 2：/api/admin/execute 路由处理器未实现 tool_name 分发逻辑，与 5.2.5/5.2.6 节工具定义直接矛盾**

- 所在位置：3.2.29 节 POST /api/admin/execute 请求体（约第 2200-2216 行）、7.3.3 节路由处理器伪代码（约第 5606-5666 行），与 5.2.5 节（约第 4664-4684 行）、5.2.6 节（约第 4721-4737 行）工具定义矛盾
- 问题描述：5.2.5 节明确指出"由 Express 端点根据 tool_name 参数分发至对应的参数化查询处理器"，但 3.2.29 节请求体定义只有 sql/user_id/api_key 三个字段没有 tool_name，7.3.3 节路由处理器伪代码仅解构 sql 字段没有 tool_name 分发逻辑。按当前实现，专用工具回调携带的 tool_name 会被忽略，请求体中可能没有 sql 字段，导致 `db.prepare(undefined)` 抛出运行时错误。
- 改进建议：在 3.2.29 节请求体定义中补充 tool_name 字段（可选，专用工具回调时必填），说明 tool_name 与 sql 字段的互斥关系（专用工具回调携带 tool_name + 业务参数，execute_SQL 兜底工具回调携带 sql）；在 7.3.3 节路由处理器伪代码中补充 tool_name 分发分支（若 tool_name 存在则路由至对应参数化查询处理器，若不存在但有 sql 字段走 execute_SQL 兜底路径）；补充专用工具参数化查询处理器的伪代码示例（至少 1-2 个，如 query_user_profile 对应 `SELECT id, username, role, avatar, created_at FROM users WHERE id = ?`）。

### 一般问题（Major）

**问题 3：POST /api/plan/generate 服务端处理流程未说明旧方案逻辑过期和 plan_id 生成机制，存在多套活跃方案风险**

- 所在位置：3.2.13 节 POST /api/plan/generate（约第 1771-1826 行）缺少服务端处理流程段落；3.2.15 节 GET /api/plan/current 的 SQLite 查询（约第 1866 行）未按 plan_id 分组；2.2 节 life_plans DDL（约第 918-931 行）缺少防止多套活跃方案的约束
- 问题描述：3.2.14 节 PUT /api/plan/adjust 详细说明了 plan_id 处理流程，但 3.2.13 节 POST /api/plan/generate 缺少对应说明。关键缺口：旧方案逻辑过期未说明、plan_id 生成时机未明确（需先 INSERT 才能获取 AUTOINCREMENT id 再 UPDATE）、DDL 中 is_active 默认为 1 没有约束防止同一 user_id 存在多个 is_active=1 的方案组。
- 改进建议：在 3.2.13 节补充"服务端处理流程"段落，明确说明：(a) 调用 Dify 工作流前先将用户当前活跃方案逻辑过期 `UPDATE life_plans SET is_active=0, updated_at=datetime('now','localtime') WHERE user_id=? AND is_active=1`；(b) plan_id 生成策略（推荐使用应用层自增序列或 `SELECT COALESCE(MAX(plan_id), 0) + 1 FROM life_plans WHERE user_id=?`，避免 INSERT-UPDATE 两步操作的事务复杂度）；(c) 所有新方案项写入时 plan_id 统一填入新生成的值；评估 3.2.15 节查询是否需限制为最新一套活跃方案（如 `AND plan_id = (SELECT MAX(plan_id) FROM life_plans WHERE user_id=? AND is_active=1)`）；在 2.5 节 life_plans 数据字典中补充业务约束说明（同一 user_id 同时仅允许存在一套 is_active=1 的方案组）。

**问题 4：punch_in.plan_id 在 DDL 中为 DEFAULT NULL，但需求 6.6 节 API 契约要求必填，存在契约矛盾**

- 所在位置：2.2 节 punch_in DDL（约第 945-957 行）；与需求 6.6 节、3.2.16 节 POST /api/punch 请求体（约第 1883 行）必填标注矛盾
- 问题描述：需求 6.6 节明确标注 plan_id 为"必填"，但详细设计 2.2 节 punch_in DDL 中为 `plan_id INTEGER DEFAULT NULL` 允许为空，文档未说明此偏离理由，也未说明何种场景下 plan_id 可以为 NULL。
- 改进建议：方案 A（遵循需求，建议）将 DDL 改为 `plan_id INTEGER NOT NULL`，保留 `ON DELETE SET NULL` 外键约束，并在文档中说明 NOT NULL 约束与 ON DELETE SET NULL 的协作机制（INSERT 时强制要求 plan_id，但 life_plans 记录删除时 punch_in.plan_id 自动设为 NULL 保留打卡历史）；方案 B（说明偏离理由）保留 DEFAULT NULL，但在 2.5 节 punch_in 数据字典中明确说明"plan_id 在 POST /api/punch 接口层为必填，DDL 层允许 NULL 是为了支持 ON DELETE SET NULL 外键约束和管理员 Text2SQL 场景"。

**问题 5：Dify 工作流输出解析失败的处理策略仅 life-plan-generator 有定义，diabetes-risk-prediction 和 health-article-generator 缺失**

- 所在位置：5.2.1 节 diabetes-risk-prediction（约第 4392-4455 行）、5.2.3 节 health-article-generator（约第 4541-4577 行），缺少输出格式和解析策略；与 5.2.2 节 life-plan-generator（约第 4530-4537 行）的完整定义对比
- 问题描述：5.2.2 节 life-plan-generator 有完整的三层降级解析策略（JSON 优先 → 正则提取降级 → LLM 二次调用降级）和解析失败错误响应 PLAN_PARSE_ERROR，但 5.2.1 节 diabetes-risk-prediction 和 5.2.3 节 health-article-generator 均未说明输出格式和解析失败处理策略。三个工作流都属于"AI 内容生成持久化路径"，应有一致的解析失败处理框架。
- 改进建议：在 5.2.1 节补充输出格式说明（JSON 对象）和解析失败处理策略（至少定义 JSON.parse 失败时的错误响应，如 RISK_PARSE_ERROR）；在 5.2.3 节补充输出格式说明（JSON 对象含 title/cover/content/category/tags/summary 字段）和解析失败处理策略（如 ARTICLE_PARSE_ERROR）；考虑在 6.3.5 节 difyService.js 行为规格中补充统一的"工作流输出解析框架"段落，定义通用的三层降级策略模板，各工作流在此基础上定制字段映射。

**问题 6：POST /api/articles/generate 和 POST /api/plan/generate 缺少幂等性保护，用户重复提交可能导致重复生成**

- 所在位置：3.2.13 节 POST /api/plan/generate（约第 1771 行）、3.2.21 节 POST /api/articles/generate（约第 2038 行）、4.3 节 LifePlan.vue 流程图（约第 3560 行）和 NewsView.vue 流程图（约第 3575 行）
- 问题描述：两个端点均为有数据库写入副作用的 AI 生成端点，用户快速双击、网络重试、前端未正确禁用提交按钮时可能触发多次请求，导致 articles 表写入重复文章、life_plans 表写入多套方案。文档仅依赖前端"按钮 loading 态"和组件级 isSubmitting 状态防重复，缺少服务端幂等性保护。
- 改进建议：在 3.2.13 节和 3.2.21 节服务端处理流程中补充幂等性检查（如同一 user_id 在 N 秒内（如 30s）已有生成请求在进行中或刚完成则拒绝重复请求并返回 409 CONFLICT）；或采用客户端幂等键方案（前端生成唯一 requestId，服务端基于 requestId + user_id 去重）；在 3.4 节错误码枚举表中补充 CONFLICT 错误码的触发场景说明。

**问题 7：admin_logs 日志写入与 SQL 执行的事务一致性未说明，存在审计日志丢失风险**

- 所在位置：7.3.3 节路由处理器伪代码（约第 5654-5665 行）
- 问题描述：7.3.3 节路由处理器伪代码中 SQL 执行和 insertAdminLog 是两个独立的数据库操作，若 SQL 执行成功但 insertAdminLog 失败（如 operation_content 字段过长触发 SQLite 错误、或进程在两次操作间崩溃），将导致 SQL 操作已生效但审计日志丢失，违反需求 4.9 节"管理员的所有数据库修改操作均可追溯...操作日志不可删除"的硬性要求。better-sqlite3 是同步驱动支持事务，但文档未说明是否将两者包裹在同一事务中。
- 改进建议：在 7.3.3 节伪代码中将 SQL 执行和 insertAdminLog 包裹在 better-sqlite3 事务中（`const result = db.transaction(() => { const r = sqlType === 'SELECT' ? db.prepare(sql).all() : db.prepare(sql).run(); if (sqlType !== 'SELECT') { insertAdminLog(...); } return r; })();`）；补充事务失败时的错误处理（事务整体回滚，返回 500 INTERNAL_ERROR，SQL 操作不生效，日志也不写入）；说明 SELECT 操作是否需要记录日志。

**问题 8：doctor_information 表 chat_token 字段明文存储 Dify API Key，缺少加密策略说明**

- 所在位置：2.2 节 doctor_information DDL（约第 858 行）、2.5 节 doctor_information 数据字典（约第 1192 行）、7.5 节 XSS 防御和 7.6 节 SQL 注入防护均未涉及 chat_token 加密
- 问题描述：chat_token 字段存储 Dify 聊天助手的 API Secret（格式 `app-XXX`），属于敏感凭证，2.2 节 DDL 中为 `chat_token TEXT NOT NULL` 明文存储。若数据库文件泄露（如备份文件未加密传输、服务器被入侵），所有医生的 Dify API Key 将直接暴露。
- 改进建议：方案 A（推荐，加密存储）chat_token 列改为存储 AES-256-GCM 加密后的密文，Express 读取后用 JWT_SECRET 派生密钥解密使用，在 2.5 节数据字典说明加密策略，在 7 节安全设计中补充"敏感字段加密"段落；方案 B（说明风险并接受）在 2.5 节 chat_token 字段说明中补充风险提示"chat_token 为明文存储，依赖 SQLite 文件级访问控制保障安全，生产环境应考虑字段级加密"；无论哪种方案，应在 6.7 节备份策略中补充说明备份文件包含 chat_token 敏感数据，备份存储需访问控制。

**问题 9：POST /api/auth/logout 端点设计不完整，未说明登出时需清理的前端状态**

- 所在位置：3.2.3 节 POST /api/auth/logout（约第 1513-1523 行）、4.3 节 Profile.vue 流程图登出分支（约第 3698-3702 行）、1.5.2 节 authStore.logout() 伪代码（约第 367-375 行）仅清理 token/role/user
- 问题描述：3.2.3 节仅有简单的请求体和响应体，未说明登出时需清理的前端状态。以下状态未说明如何处理：chatStore 中的所有 conversation_id、活跃的 SSE 连接、riskFormStore 中的表单数据、sessionStorage 中的页面级缓存。
- 改进建议：在 3.2.3 节补充"前端登出完整流程"段落，明确登出时按顺序清理（调用 `chatStore.abortActiveConnection()` 中止活跃 SSE 连接 → 调用 `chatStore.clearDoctorConversation` / `clearAssistantConversation` 清理所有会话，或新增 `clearAllConversations()` 方法 → 调用 `riskFormStore.reset()` 清除表单数据 → 调用 `authStore.logout()` 清除 token/role/user 并跳转首页 → 可选清理 sessionStorage 页面级缓存）；在 3.7 节 chatStore 接口中评估是否需要新增 `clearAllConversations(): void` 方法；在 4.3 节 Profile.vue 流程图的登出分支中补充上述清理步骤。

### 轻微问题（Minor）

**问题 10：POST /api/risk/predict 响应的 created_at 字段标注为"详细设计扩展字段"，但 TypeScript 类型中为必填字段，存在标注与契约不一致**

- 所在位置：3.2.7 节响应 JSON（约第 1625 行）`"created_at": "2026-06-23T14:30:00"  // 详细设计扩展字段`；3.8.4 节 RiskPredictResponse（约第 2688 行）`created_at: string;`（必填）
- 问题描述："扩展字段"通常意味着可选，但类型契约要求必填，前端按 RiskPredictResponse 类型编码时若后端未返回 created_at 将导致运行时为 undefined 但 TypeScript 编译期不报错。
- 改进建议：方案 A（统一为必填）删除 3.2.7 节的"扩展字段"注释，在 3.8.4 节保留 `created_at: string`，并在 3.2.7 节说明"created_at 由 Express 端点从 user_risk_info.created_at 列返回，用于前端展示预测时间"；方案 B（统一为可选）将 3.8.4 节改为 `created_at?: string`，3.2.7 节保留"扩展字段"标注。

**问题 11：articles 表与 life_advice 表的 tags 字段约束不一致（NOT NULL vs 允许 NULL）**

- 所在位置：2.2 节 articles DDL（约第 870 行）`tags TEXT NOT NULL DEFAULT '[]'`；2.2 节 life_advice DDL（约第 938 行）`tags TEXT DEFAULT '[]'`（无 NOT NULL）
- 问题描述：两表都有 tags 字段存储 JSON 数组字符串，用途相同，但 DDL 约束不一致。life_advice 表允许 NULL，若通过 Text2SQL 的 INSERT 语句显式传入 NULL，SQLite 不会拦截，可能导致 `JSON.parse(null)` 抛出错误。
- 改进建议：将 life_advice 表的 tags 字段改为 `tags TEXT NOT NULL DEFAULT '[]'`，与 articles 表保持一致，统一两个表的约束规范。

## 历史迭代回顾

前 5 轮迭代已识别并修复 47 个基础维度问题，第 6 轮诊断聚焦工程实施维度。由于上一轮组件A产出（a_v6）直接复制自 v5 未做实际修改，第 6 轮识别的 9 个问题全部持续存在。

### 已解决的问题（出现在历史反馈但当前反馈中不再提及的问题）

以下问题在前 5 轮迭代中识别并已修复，当前诊断报告不再提及：

- **第 1 轮**（11 个）：枚举值中英文偏差、user_risk_info 缺 diabetes_history、life_plans 缺 is_active、Consultation 未拆分、个人中心子页面嵌套路由、chatStore conversation_id 设计、数据类型映射转换机制、News 未拆分、跨标签页登录态同步、SSE 连接控制与并发限制、waist/systolic_bp 0 值校验
- **第 2 轮**（8 个）：difyAuth.js timingSafeEqual DoS 隐患、router/index.ts 语法错误、mapper.js 漏配 punch_type、risk/predict 响应结构偏差、pregnancy 字段类型不一致、seed.sql 占位符风险、布尔字段类型不统一、核心接口缺 SQLite 查询
- **第 3 轮**（7 个）：operatorRole 硬编码、validateRowLevelPermission 缺失实现、plan_type/punch_type 约束矛盾、免责声明未调用、Dify 风险预测输出契约不一致、punch 响应缺 remarks、AiChatDialog 缺免责提示
- **第 4 轮**（17 个）：useSSE 未处理 401、chatStore 方法命名不一致、risk/predict 0 值校验 HTTP 状态码、articles/:id 缺 is_collected、articles 列表缺 tags/summary、Dify Agent 工具定义偏离、未采用 Vant 4、DoctorChatView SSE 未关闭、Admin/AiChatDialog SSE 流程缺失、plan/generate 响应结构偏离、跨标签页 setAuth 不完整、NewsView 缺免责判定、SSE 事件缺 created_at、difyService 超时 60s 偏离、punch 中文枚举、AiChatDialog 缺登录引导、admin_logs 字段名混淆
- **第 5 轮**（8 个）：register 响应缺 token、login 响应 role 位置错误、plan_id 死参数、pregnancy 转换层缺失、punch_type 扩展 other 偏离需求、PunchCreateResponse 类型退化、User 接口 created_at 不一致、clearMustChangePassword 未定义

### 持续存在的问题（在多轮反馈中反复出现的问题，需重点解决）

以下 9 个问题在第 6 轮诊断中首次识别，由于 a_v6 直接复制自 v5 未做修改，这些问题在第 7 轮诊断中持续存在，**本轮必须全部修复**：

1. **[严重] difyService.js blocking 模式自动重试违反需求 7.3 节**（当前问题 1）
2. **[严重] /api/admin/execute 路由处理器未实现 tool_name 分发逻辑**（当前问题 2）
3. **[一般] POST /api/plan/generate 缺少服务端处理流程**（当前问题 3）
4. **[一般] punch_in.plan_id DDL DEFAULT NULL 与需求矛盾**（当前问题 4）
5. **[一般] Dify 工作流输出解析失败策略缺失**（当前问题 5）
6. **[一般] POST /api/articles/generate 和 plan/generate 缺幂等性保护**（当前问题 6）
7. **[一般] admin_logs 事务一致性未说明**（当前问题 7）
8. **[一般] chat_token 明文存储缺加密策略**（当前问题 8）
9. **[一般] POST /api/auth/logout 未说明登出时前端状态清理**（当前问题 9）

### 新发现的问题（本轮新识别的问题）

本轮（第 7 轮）新识别 2 个轻微问题，属于第 6 轮审查未覆盖的文档内部一致性维度：

10. **[轻微] POST /api/risk/predict 响应 created_at 字段标注与 TypeScript 类型契约不一致**（当前问题 10）
11. **[轻微] articles 与 life_advice 表 tags 字段约束不一致**（当前问题 11）

## 上一轮产出路径

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/a_v6_copy_from_v5.md

## 用户需求

C:/Users/DELL/Desktop/qingruanProject2026/redeliberations/202606241614_detailed_design_v3/requirement.md
