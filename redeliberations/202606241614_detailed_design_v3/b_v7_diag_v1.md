# 质量审查报告 — b_v7_diag_v1

**审查对象**：`a_v7_copy_from_v6.md`（详细设计文档 v7，经 v1~v15 共 15 轮修订）
**审查轮次**：第 7 次
**审查重点**：需求响应充分度、事实错误/逻辑矛盾、深度与完整性（侧重内部审议未充分覆盖的维度）

---

## 审查发现

### 问题 1（严重）：7.3.3 节 `dispatchParameterizedQuery` 函数与 5.2.5/5.2.6 节工具定义严重不一致，导致专用工具回调全部失败

**问题描述**：
v15 修订在 7.3.3 节新增了 `dispatchParameterizedQuery` 函数用于处理专用工具回调，但该函数实现的工具清单与 5.2.5 节（diabetes-assistant-agent）和 5.2.6 节（admin-manager-agent）定义的工具清单存在三类不一致：

1. **工具名不匹配**：5.2.5 节定义工具名为 `query_risk_history`，但 7.3.3 节 dispatch 函数实现的 case 为 `query_risk_info`（第 5813 行）。Agent 按 5.2.5 节配置的 tool_name 回调时，dispatch 函数无法匹配，返回 400 `BAD_REQUEST`。
2. **缺失工具实现**：
   - 5.2.5 节定义的 `query_health_advice`、`write_health_advice`、`update_user_profile` 三个工具在 dispatch 函数中完全没有实现。
   - 5.2.6 节定义的全部 5 个管理员工具（`query_table`、`insert_record`、`update_record`、`delete_record`、`get_table_schema`）在 dispatch 函数中完全没有实现。这意味着 admin-manager-agent 的所有专用工具调用都会落入 `default` 分支返回"未知的 tool_name"错误，**管理员智能管理功能完全失效**。
3. **多余的工具实现**：dispatch 函数实现了 `query_article_collections`、`query_all_users`、`query_doctor_list` 三个工具，但这些工具在 5.2.5 和 5.2.6 节均未定义，属于"幽灵工具"——Agent 永远不会调用它们。

**所在位置**：
- 7.3.3 节 `dispatchParameterizedQuery` 函数（第 5802-5861 行）
- 5.2.5 节 diabetes-assistant-agent 工具定义表（第 4750-4760 行）
- 5.2.6 节 admin-manager-agent 工具定义表（第 4807-4814 行）

**严重程度**：严重

**改进建议**：
1. 将 dispatch 函数的 case 列表与 5.2.5/5.2.6 节工具定义表逐一对齐：
   - 将 `case 'query_risk_info'` 重命名为 `case 'query_risk_history'`，对齐 5.2.5 节工具名
   - 补充 `query_health_advice`、`write_health_advice`、`update_user_profile` 的参数化查询处理器实现
   - 补充 5.2.6 节全部 5 个管理员工具（`query_table`、`insert_record`、`update_record`、`delete_record`、`get_table_schema`）的处理器实现，注意 `insert_record`/`update_record`/`delete_record` 涉及写操作，需补充表名白名单校验和 admin_logs 审计日志写入逻辑
2. 删除或保留 `query_article_collections`/`query_all_users`/`query_doctor_list` 的实现——若保留需在 5.2.5/5.2.6 节补充对应工具定义；若删除需从 dispatch 函数中移除
3. 同步修正 3.2.29 节（第 2256 行）工具名列举，将 `query_risk_info` 改为 `query_risk_history`，将 `query_article_collections` 改为实际定义的工具名

---

### 问题 2（一般）：8.1 节注册验收标准与 3.2.1 节注册响应契约不一致

**问题描述**：
v14 修订已将 3.2.1 节注册成功响应从 `{user_id, username}` 修改为与登录响应一致的结构（含 `token`/`role`/`user` 对象），对齐需求 6.1 节"注册成功后直接返回 JWT Token 和用户信息，用户无需重复登录"。但 8.1 节 API 验收标准清单中注册成功的验收项仍为旧契约描述——"POST /api/auth/register 返回 201，含 user_id 和 username"。

验收标准未同步更新会导致：验收人员按旧契约验证注册响应时，期望 `user_id` 和 `username` 字段，但实际响应中这两个字段不存在（响应为 `user.id` 和 `user.username` 嵌套结构 + 顶层 `token`/`role`），验收判定错误。

**所在位置**：8.1 节 API 端点验收标准表（第 6018 行）

**严重程度**：一般

**改进建议**：将 8.1 节注册成功验收项的量化标准更新为"POST /api/auth/register 返回 201，data 对象含 token（JWT）、role（'user'|'admin'）、user（含 id/username/avatar），对齐 3.2.1 节 v14 修订后的响应结构"。

---

### 问题 3（一般）：1.2 节登录态同步描述仍使用 `setToken()`，与 v14 修订后的 `login()` 逻辑不一致

**问题描述**：
1.2 节第 105 行描述"登录成功 -> authStore.setToken(token) -> Pinia响应式自动通知所有订阅组件更新登录态"。但 v14 修订已将 `authStore.login()` 修改为直接设置 token/role/user 三字段并写入 localStorage，不再调用 `setToken()`。`setToken()` 方法仅设置 token 不设置 role 和 user，与 v14 修订"login() 必须同步设置 token/role/user 三字段"的目标矛盾。

此描述残留会误导开发者按旧逻辑实现登录态同步（仅同步 token），导致 role 和 user 状态未更新——与迭代 5 指出并已在 v14 修复的"login() 未设置 role"问题本质相同，但描述层面的残留使问题有可能在实现时重新引入。

**所在位置**：1.2 节 Vue3 SPA 跨组件通信机制第 1 项（第 105 行）

**严重程度**：一般

**改进建议**：将 1.2 节描述修改为"登录成功 -> authStore.login()（设置 token/role/user 三字段并写入 localStorage）-> Pinia响应式自动通知所有订阅组件更新登录态"，与 1.5.2 节 v14 修订后的 `login()` 伪代码保持一致。

---

### 问题 4（一般）：3.7 节 `AuthState.user` 类型为 `User` 但 `login()` 实际赋值为 `LoginUser`，类型不匹配

**问题描述**：
v14 修订引入了 `LoginUser` 类型（仅含 id/username/role/avatar，不含 created_at）用于登录/注册响应，`LoginResponse.user` 和 `RegisterResponse.user` 均改为 `LoginUser` 类型。但 3.7 节 `AuthState` 接口中 `user: User | null`（第 2441 行）仍使用 `User` 类型（含 created_at），注释标注"来自 GET /api/user/profile"。

实际上 `authStore.login()`（1.5.2 节第 360 行）直接将 `res.data.user`（类型为 `LoginUser`）赋给 `user.value`，并未调用 `GET /api/user/profile` 获取完整 `User` 对象。这导致：
1. TypeScript 类型层面：`LoginUser` 赋值给 `User` 类型变量时，`created_at` 字段缺失，严格模式下编译报错或运行时为 `undefined`
2. 运行时层面：若前端组件读取 `authStore.user.created_at`（如 Profile.vue 展示注册时间），值为 `undefined`，与 `User` 类型契约不符
3. `setAuth(newToken, newRole, user: User)` 方法签名同样使用 `User` 类型，但跨标签页同步恢复时从 localStorage 读取的数据也不含 `created_at`

**所在位置**：
- 3.7 节 AuthState 接口（第 2441 行）：`user: User | null`
- 3.7 节 AuthActions.setAuth 签名（第 2450 行）：`user: User`
- 3.8.8 节 AuthState（第 2891 行）：`user: User | null`
- 1.5.2 节 login() 伪代码（第 360 行）：`user.value = res.data.user`（res.data.user 为 LoginUser 类型）

**严重程度**：一般

**改进建议**：
- 方案 A（推荐）：将 `AuthState.user` 类型从 `User | null` 改为 `LoginUser | null`，`setAuth` 方法签名中 `user` 参数改为 `LoginUser` 类型。authStore 仅持有登录响应返回的核心字段，需要 `created_at` 等完整字段时由组件单独调用 `GET /api/user/profile`
- 方案 B：在 `login()` 成功后追加调用 `fetchProfile()` 获取完整 `User` 对象写入 authStore.user，使实际数据与 `User` 类型一致。但此方案增加了一次 API 请求，且需求 6.1 节登录响应未要求返回 created_at

---

### 问题 5（轻微）：3.7 节 ChatActions 中 `clearChat()` 与 `clearAllConversations()` 职责重叠未区分

**问题描述**：
3.7 节 ChatActions 接口同时定义了 `clearChat(): void`（第 2477 行）和 `clearAllConversations(): void`（第 2490 行）。v15 修订新增 `clearAllConversations()` 用于登出时统一清理所有对话会话和消息历史，但 `clearChat()` 方法仍保留且未说明其用途、与 `clearAllConversations()` 的区别。

两个方法名都暗示"清空对话状态"，实现者无法判断应使用哪一个：`clearChat()` 是否仅清空 messages 数组而保留 conversation_id？`clearAllConversations()` 是否包含 `clearChat()` 的功能？接口语义模糊可能导致实现时重复实现或遗漏。

**所在位置**：3.7 节 ChatActions 接口（第 2477 行、第 2490 行）

**严重程度**：轻微

**改进建议**：
- 若 `clearChat()` 与 `clearAllConversations()` 功能重叠，删除 `clearChat()` 避免混淆
- 若两者有不同职责（如 `clearChat()` 仅清空当前消息列表，`clearAllConversations()` 清理会话 ID + 消息 + localStorage 持久化），在接口注释中明确两者的调用场景和清理范围差异

---

## 整体评价

文档经 15 轮修订后，前序迭代中识别的绝大多数问题（枚举值规范、plan_id 列、pregnancy 转换、SSE 连接控制、敏感字段加密、幂等性保护、事务一致性、免责声明调用等）均已得到有效修复，文档整体深度和完整性较好，能够指导大部分模块的实现。

本轮审查发现的主要问题集中在 **v15 修订新增的 `dispatchParameterizedQuery` 函数与工具定义章节的契约一致性**上（问题 1），这是阻断性问题——admin-manager-agent 的 5 个专用工具全部无法工作，admin-manager-agent 的系统提示词明确要求"优先使用专用工具"，但 dispatch 函数不识别这些工具名，会导致管理员自然语言数据库操作功能完全失效。此问题需优先修复。

其余问题均为文档描述与已修订契约的残留不一致（问题 2-4）或接口语义模糊（问题 5），修复成本低但影响实现准确性，建议一并处理。
