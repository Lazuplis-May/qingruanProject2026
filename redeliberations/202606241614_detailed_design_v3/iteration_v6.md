# 再审议判定报告（v6）

## 判定结果

RETRY

## 判定理由

组件B内部循环最大轮次为12，实际轮次为1（实际<最大），质询报告结果为 LOCATED，表明组件B提前终止循环并确认了诊断报告中识别的问题，无需依赖循环耗尽的兜底判定路径。

诊断报告共识别 11 个问题：2 个严重、7 个一般、2 个轻微。质询报告对 11 个问题逐一进行了证据充分性、逻辑完整性、覆盖完备性三个维度的审查：

- **证据充分性**：11 个问题中 10 个判定为[通过]，仅问题 8（chat_token 加密策略）的证据引用存在轻微偏差（需求 7.5 节条款适用对象为"源代码/版本控制系统"而非数据库存储），但质询报告同时指出"从安全工程实践视角识别数据库敏感字段加密策略缺失本身是合理的技术风险识别"，偏差不构成严重证据缺陷，问题本身仍然成立。
- **逻辑完整性**：11 个问题之间不存在逻辑矛盾，改进建议与问题描述一致且可行，严重程度分级合理。质询报告特别确认问题 1 与问题 6 虽都涉及重复生成风险但分析角度互补不构成重复，问题 3 与问题 6 虽都涉及 plan/generate 端点但分别关注处理流程完整性和并发控制。
- **覆盖完备性**：任务要求的三个审查维度（需求响应充分度、事实错误/逻辑矛盾、深度和完整性）均已覆盖，工程实施视角的三个子维度（可指导实现性、技术风险识别、关键技术决策遗漏）均已覆盖，且本轮发现的问题未与前 5 轮已修复的 47 个问题重复。

根据判定标准，审查报告包含严重等级问题（问题 1、问题 2）和一般等级问题（问题 3-9），不满足 PASS 条件。质询报告 LOCATED 结论确认了问题的真实性，且仅有的轻微证据偏差（问题 8）不改变问题成立性。判定为 RETRY，需重新运行组件A以修复以下问题。

## 需要解决的问题

- **问题描述**：difyService.js blocking 模式自动重试直接违反需求 7.3 节"不自动重试"原则。第 6.3.5 节 blocking 模式（用于 POST /api/risk/predict、POST /api/plan/generate、POST /api/articles/generate）的重试次数标注为 1，重试间隔 2s，与需求 7.3 节"不自动重试，避免重复提交请求加重服务端压力"直接矛盾。更严重的是这三个端点均有数据库写入副作用，自动重试可能导致 user_risk_info、life_plans、articles 表写入重复记录，且 life_plans 的 plan_id 生成机制未明确，is_active 过期机制无法正确清理重复方案。
- **所在位置**：第 6.3.5 节"超时与重试策略"表格（约第 5341-5344 行）
- **严重程度**：严重
- **改进建议**：将 blocking 模式重试次数从 1 改为 0，与需求 7.3 节对齐；在表格下方补充设计依据说明，明确 blocking 模式不自动重试是因为三个端点均有数据库写入副作用；若项目组评估后确认需要重试，需明确说明此为对需求 7.3 节的偏离决策，并补充幂等性保护机制（如基于 user_id + 请求时间窗口的去重）。

- **问题描述**：`/api/admin/execute` 路由处理器未实现 `tool_name` 分发逻辑，与 5.2.5/5.2.6 节工具定义直接矛盾。5.2.5 节明确指出"由 Express 端点根据 tool_name 参数分发至对应的参数化查询处理器"，但 3.2.29 节请求体定义只有 sql/user_id/api_key 三个字段没有 tool_name，7.3.3 节路由处理器伪代码仅解构 sql 字段没有 tool_name 分发逻辑。按当前实现，专用工具回调携带的 tool_name 会被忽略，请求体中可能没有 sql 字段，导致 `db.prepare(undefined)` 抛出运行时错误。
- **所在位置**：3.2.29 节 POST /api/admin/execute 请求体（约第 2200-2216 行）、7.3.3 节路由处理器伪代码（约第 5606-5666 行），与 5.2.5 节（约第 4664-4684 行）、5.2.6 节（约第 4721-4737 行）工具定义矛盾
- **严重程度**：严重
- **改进建议**：在 3.2.29 节请求体定义中补充 tool_name 字段（可选，专用工具回调时必填），说明 tool_name 与 sql 字段的互斥关系；在 7.3.3 节路由处理器伪代码中补充 tool_name 分发分支，若 tool_name 存在则路由至对应参数化查询处理器，若不存在但有 sql 字段走 execute_SQL 兜底路径；补充专用工具参数化查询处理器的伪代码示例（至少 1-2 个，如 query_user_profile）。

- **问题描述**：POST /api/plan/generate 服务端处理流程未说明旧方案逻辑过期和 plan_id 生成机制，存在多套活跃方案风险。3.2.14 节 PUT /api/plan/adjust 详细说明了 plan_id 处理流程，但 3.2.13 节 POST /api/plan/generate 缺少对应说明，关键缺口包括：旧方案逻辑过期未说明、plan_id 生成时机未明确（需先 INSERT 才能获取 AUTOINCREMENT id 再 UPDATE）、DDL 中 is_active 默认为 1 没有约束防止同一 user_id 存在多个 is_active=1 的方案组。
- **所在位置**：3.2.13 节 POST /api/plan/generate（约第 1771-1826 行）缺少服务端处理流程段落；3.2.15 节 GET /api/plan/current 的 SQLite 查询（约第 1866 行）未按 plan_id 分组；2.2 节 life_plans DDL（约第 918-931 行）缺少防止多套活跃方案的约束
- **严重程度**：一般
- **改进建议**：在 3.2.13 节补充"服务端处理流程"段落，明确说明调用 Dify 工作流前先将当前活跃方案逻辑过期、plan_id 生成策略（推荐使用应用层自增序列或 `SELECT COALESCE(MAX(plan_id), 0) + 1 FROM life_plans WHERE user_id=?`）、新方案项写入时 plan_id 统一填入新生成的值；评估 3.2.15 节查询是否需限制为最新一套活跃方案；在 2.5 节 life_plans 数据字典中补充业务约束说明。

- **问题描述**：punch_in.plan_id 在 DDL 中为 DEFAULT NULL，但需求 6.6 节 API 契约要求必填，存在契约矛盾。需求 6.6 节明确标注 plan_id 为"必填"，但详细设计 2.2 节 punch_in DDL 中为 `plan_id INTEGER DEFAULT NULL` 允许为空，文档未说明此偏离理由，也未说明何种场景下 plan_id 可以为 NULL。
- **所在位置**：2.2 节 punch_in DDL（约第 945-957 行）；与需求 6.6 节、3.2.16 节 POST /api/punch 请求体（约第 1883 行）必填标注矛盾
- **严重程度**：一般
- **改进建议**：方案 A（遵循需求，建议）将 DDL 改为 `plan_id INTEGER NOT NULL`，保留 `ON DELETE SET NULL` 外键约束，并在文档中说明 NOT NULL 约束与 ON DELETE SET NULL 的协作机制；方案 B（说明偏离理由）保留 DEFAULT NULL，但在 2.5 节 punch_in 数据字典中明确说明 plan_id 在 API 层必填、DDL 层允许 NULL 是为了支持 ON DELETE SET NULL 外键约束和管理员 Text2SQL 场景。

- **问题描述**：Dify 工作流输出解析失败的处理策略仅 life-plan-generator 有定义，diabetes-risk-prediction 和 health-article-generator 缺失。5.2.2 节 life-plan-generator 有完整的三层降级解析策略（JSON 优先 → 正则提取降级 → LLM 二次调用降级）和解析失败错误响应 PLAN_PARSE_ERROR，但 5.2.1 节 diabetes-risk-prediction 和 5.2.3 节 health-article-generator 均未说明输出格式和解析失败处理策略。三个工作流都属于"AI 内容生成持久化路径"，应有一致的解析失败处理框架。
- **所在位置**：5.2.1 节 diabetes-risk-prediction（约第 4392-4455 行）、5.2.3 节 health-article-generator（约第 4541-4577 行），缺少输出格式和解析策略；与 5.2.2 节 life-plan-generator（约第 4530-4537 行）的完整定义对比
- **严重程度**：一般
- **改进建议**：在 5.2.1 节补充输出格式说明（JSON 对象）和解析失败处理策略（如 RISK_PARSE_ERROR 错误响应）；在 5.2.3 节补充输出格式说明（JSON 对象含 title/cover/content/category/tags/summary 字段）和解析失败处理策略（如 ARTICLE_PARSE_ERROR）；考虑在 6.3.5 节 difyService.js 行为规格中补充统一的"工作流输出解析框架"段落，定义通用三层降级策略模板。

- **问题描述**：POST /api/articles/generate 和 POST /api/plan/generate 缺少幂等性保护，用户重复提交可能导致重复生成。两个端点均为有数据库写入副作用的 AI 生成端点，用户快速双击、网络重试、前端未正确禁用提交按钮时可能触发多次请求，导致 articles 表写入重复文章、life_plans 表写入多套方案。文档仅依赖前端"按钮 loading 态"和组件级 isSubmitting 状态防重复，缺少服务端幂等性保护。
- **所在位置**：3.2.13 节 POST /api/plan/generate（约第 1771 行）、3.2.21 节 POST /api/articles/generate（约第 2038 行）、4.3 节 LifePlan.vue 流程图（约第 3560 行）和 NewsView.vue 流程图（约第 3575 行）
- **严重程度**：一般
- **改进建议**：在 3.2.13 节和 3.2.21 节服务端处理流程中补充幂等性检查，如同一 user_id 在 N 秒内（如 30s）已有生成请求在进行中或刚完成则拒绝重复请求并返回 409 CONFLICT；或采用客户端幂等键方案，前端生成唯一 requestId，服务端基于 requestId + user_id 去重；在 3.4 节错误码枚举表中补充 CONFLICT 错误码的触发场景说明。

- **问题描述**：admin_logs 日志写入与 SQL 执行的事务一致性未说明，存在审计日志丢失风险。7.3.3 节路由处理器伪代码中 SQL 执行和 insertAdminLog 是两个独立的数据库操作，若 SQL 执行成功但 insertAdminLog 失败（如 operation_content 字段过长触发 SQLite 错误、或进程在两次操作间崩溃），将导致 SQL 操作已生效但审计日志丢失，违反需求 4.9 节"管理员的所有数据库修改操作均可追溯...操作日志不可删除"的硬性要求。better-sqlite3 是同步驱动支持事务，但文档未说明是否将两者包裹在同一事务中。
- **所在位置**：7.3.3 节路由处理器伪代码（约第 5654-5665 行）
- **严重程度**：一般
- **改进建议**：在 7.3.3 节伪代码中将 SQL 执行和 insertAdminLog 包裹在 better-sqlite3 事务中（`db.transaction(() => { ... })()`），事务整体失败时回滚并返回 500 INTERNAL_ERROR；补充事务失败时的错误处理说明；明确 SELECT 操作是否需要记录日志。

- **问题描述**：doctor_information 表 chat_token 字段明文存储 Dify API Key，缺少加密策略说明。chat_token 字段存储 Dify 聊天助手的 API Secret（格式 `app-XXX`），属于敏感凭证，2.2 节 DDL 中为 `chat_token TEXT NOT NULL` 明文存储，7.6 节 SQL 注入防护表提到"SQLite 仅 Express 进程访问"但未说明 chat_token 加密策略。若数据库文件泄露（如备份文件未加密传输、服务器被入侵），所有医生的 Dify API Key 将直接暴露。
- **所在位置**：2.2 节 doctor_information DDL（约第 858 行）、2.5 节 doctor_information 数据字典（约第 1192 行）、7.5 节 XSS 防御和 7.6 节 SQL 注入防护均未涉及 chat_token 加密
- **严重程度**：一般
- **改进建议**：方案 A（推荐，加密存储）chat_token 列改为存储 AES-256-GCM 加密后的密文，Express 读取后用 JWT_SECRET 派生密钥解密使用，在 2.5 节数据字典说明加密策略，在 7 节安全设计中补充"敏感字段加密"段落；方案 B（说明风险并接受）在 2.5 节 chat_token 字段说明中补充风险提示"chat_token 为明文存储，依赖 SQLite 文件级访问控制保障安全，生产环境应考虑字段级加密"；无论哪种方案，应在 6.7 节备份策略中补充说明备份文件包含 chat_token 敏感数据，备份存储需访问控制。

- **问题描述**：POST /api/auth/logout 端点设计不完整，未说明登出时需清理的前端状态。3.2.3 节仅有简单的请求体和响应体，未说明登出时需清理的前端状态，4.3 节 Profile.vue 流程图仅调用 authStore.logout() 清除 Token + user info，但以下状态未说明如何处理：chatStore 中的所有 conversation_id、活跃的 SSE 连接、riskFormStore 中的表单数据、sessionStorage 中的页面级缓存。
- **所在位置**：3.2.3 节 POST /api/auth/logout（约第 1513-1523 行）、4.3 节 Profile.vue 流程图登出分支（约第 3698-3702 行）、1.5.2 节 authStore.logout() 伪代码（约第 367-375 行）仅清理 token/role/user
- **严重程度**：一般
- **改进建议**：在 3.2.3 节补充"前端登出完整流程"段落，明确登出时按顺序清理（中止活跃 SSE 连接、清理所有会话、清除表单数据、清除 token/role/user 并跳转首页、可选清理 sessionStorage 页面级缓存）；在 3.7 节 chatStore 接口中评估是否需要新增 clearAllConversations() 方法；在 4.3 节 Profile.vue 流程图的登出分支中补充上述清理步骤。
