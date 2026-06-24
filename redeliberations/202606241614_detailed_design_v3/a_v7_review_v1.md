# 技术方案审查报告（v1）

## 审查结果

APPROVED

## 逐维度审查

### 1. 技术准确性

**[通过]** 三个工作流（`diabetes-risk-prediction`、`life-plan-generator`、`health-article-generator`）的解析失败策略采用统一的三层降级框架（JSON 优先 → 正则提取降级 → LLM 二次调用降级），错误码分别为 `RISK_PARSE_ERROR`/`PLAN_PARSE_ERROR`/`ARTICLE_PARSE_ERROR`，技术路径合理且与 5.2.2 节已有设计一致（6.3.5 节"工作流输出解析框架"表格）。

**[通过]** `chat_token` 加密策略选用 AES-256-GCM（Node.js 内置 `crypto` 模块），密钥通过 `crypto.scryptSync(JWT_SECRET, salt, 32)` 派生，密文格式 `base64(iv):base64(authTag):base64(ciphertext)`。AES-256-GCM 同时提供机密性和完整性保护，scrypt 是 NIST 推荐的密钥派生函数，技术选型准确（7.8 节）。

**[通过]** `admin_logs` 事务一致性采用 `better-sqlite3` 的 `db.transaction(() => { ... })()` 同步事务包裹 SQL 执行与日志写入，与 better-sqlite3 的同步驱动特性匹配。事务失败时整体回滚，返回 500 INTERNAL_ERROR，符合需求 4.9 节"操作日志不可删除"的硬性要求（7.3.3 节伪代码）。

**[通过]** `tool_name` 分发逻辑通过 `dispatchParameterizedQuery` 函数实现，使用 `db.prepare(sql).all()` 占位符绑定，杜绝 SQL 注入。各专用工具（`query_user_profile`、`query_risk_info`、`query_life_plans`、`query_punch_records`、`query_article_collections`、`query_all_users`、`query_doctor_list`）的 SQL 模板均为预定义参数化查询，技术路径正确（7.3.3 节）。

**[通过]** blocking 模式重试次数改为 0，与需求 7.3 节"所有 AI 接口不自动重试，避免重复提交请求加重服务端压力"完全对齐。设计依据说明清晰：三个端点（risk/predict、plan/generate、articles/generate）均有数据库写入副作用，自动重试可能导致重复记录（6.3.5 节超时与重试策略表 + 设计依据说明）。

**[通过]** 幂等性保护采用"内存级最近请求时间戳 `Map<userId, lastRequestAt>` 或基于 `created_at` 列查询"方案，30 秒窗口返回 409 CONFLICT。该方案对单实例 Express 部署场景适用，与项目"Vue3 SPA + Express + SQLite"的单实例架构匹配（3.2.13/3.2.21 节）。

### 2. 完备性

**[通过]** 用户任务中的 11 个问题（2 严重 / 7 一般 / 2 轻微）全部有对应的技术方案说明：
- 问题 1（blocking 重试）→ 6.3.5 节超时与重试策略表 + 设计依据说明
- 问题 2（tool_name 分发）→ 3.2.29 节请求体定义 + 7.3.3 节路由处理器伪代码 + `dispatchParameterizedQuery` 函数
- 问题 3（plan/generate 服务端处理流程）→ 3.2.13 节"服务端处理流程"段落（7 步骤）+ 2.5 节 life_plans 业务约束说明
- 问题 4（punch_in.plan_id DEFAULT NULL）→ 2.5 节 punch_in 数据字典方案 B 说明（DDL 层 NULL + API 层必填的协作机制）
- 问题 5（Dify 工作流输出解析策略）→ 5.2.1/5.2.3 节补充输出格式与解析失败策略 + 6.3.5 节统一解析框架
- 问题 6（幂等性保护）→ 3.2.13/3.2.21 节幂等性检查 + 3.4 节错误码表 + 4.3 节前端流程图
- 问题 7（admin_logs 事务一致性）→ 7.3.3 节事务包裹伪代码 + 失败回滚处理
- 问题 8（chat_token 加密）→ 2.2 节 DDL 注释 + 2.5 节数据字典 + 7.8 节加密策略 + 6.7 节备份策略
- 问题 9（logout 前端状态清理）→ 3.2.3 节"前端登出完整流程"5 步骤 + 3.7 节 chatStore.clearAllConversations() + 4.3 节 Profile.vue 流程图
- 问题 10（created_at 字段标注）→ 3.2.7 节响应字段说明 + 3.8.4 节 RiskPredictResponse 类型统一为必填
- 问题 11（tags 字段约束）→ 2.2 节 life_advice DDL + 2.5 节数据字典统一为 NOT NULL DEFAULT '[]'

**[通过]** 数据流闭环完整：plan/generate 的 plan_id 生成（`SELECT COALESCE(MAX(plan_id), 0) + 1`）→ 旧方案逻辑过期（`UPDATE ... SET is_active=0`）→ 新方案写入（INSERT 含 plan_id）→ 响应返回新 plan_id，形成完整闭环。

**[通过]** `tool_name` 与 `sql` 字段的互斥关系定义清晰：专用工具回调携带 `tool_name` + 业务参数（不含 `sql`），`execute_SQL` 兜底工具回调携带 `sql`（不含 `tool_name`），两种场景均携带 `user_id` 和 `api_key`。路由处理器先检查 `tool_name` 分发，再走 `sql` 兜底路径，逻辑分支完备（3.2.29 节 + 7.3.3 节）。

### 3. 可操作性

**[通过]** 每项修改均有明确结论：blocking 重试次数为 0、chat_token 加密算法为 AES-256-GCM、幂等性窗口为 30 秒、plan_id 生成策略为 `COALESCE(MAX(plan_id), 0) + 1`，实现者无需自行探索技术方向。

**[通过]** 实现者可从方案中明确知道"做什么"和"怎么做的大方向"：
- `dispatchParameterizedQuery` 函数提供了 7 个专用工具的完整 SQL 模板示例（含占位符绑定、行级权限校验、管理员角色判断）
- 前端登出流程按顺序列出 5 个步骤（中止 SSE → 清理对话 → 清除表单 → 清除认证 → 可选清理缓存），并说明清理顺序的重要性
- AES-256-GCM 加密策略表格列出 7 个实施维度（算法、密钥派生、密文格式、加密时机、解密时机、密钥管理、密钥轮换）

**[通过]** 技术引用足够具体：所有修改均标注"v15 修订新增"并说明与原章节的联动关系（如 7.8 节加密策略联动 2.2 节 DDL 注释、2.5 节数据字典、6.7 节备份策略），实现者可按图索骥定位所有相关修改点。

**[轻微]** 文件中存在大量"v15 修订新增"内联标注，但文末未新增对应的"修订说明（v15）"汇总段落（最新汇总仍为 v14）。这属于文档规范性问题，不影响技术决策的明确性和实现路径的清晰性，实现者仍可通过内联标注定位所有修改点。建议后续迭代补充 v15 修订汇总说明以保持文档规范性。
