# 技术方案审查报告（v1）

## 审查结果

APPROVED

## 逐维度审查

### 1. 技术准确性

**[通过]** 技术选型验证：Vue 3 + TypeScript + Vite、Express 4.x、better-sqlite3 9.x、Pinia 2.x、Vue Router 4.x、Axios 1.x、Vant 4.x 等技术栈版本均为当前主流稳定版本，适用于移动端优先的 SPA 健康管理平台场景。Dify + DeepSeek 的 AI 能力架构设计与需求 1 节定义一致。

**[通过]** 库能力描述准确性：
- Dify 工作流 SSE 流式响应机制（`data: {...}\n\n` 事件格式）与 Dify 官方对话 API 格式一致
- `better-sqlite3` 同步 SQLite 驱动描述准确，适用于 Express 单实例部署
- `bcryptjs` 纯 JS 实现无需编译，与 Node.js 18 LTS 兼容
- `pinia-plugin-persistedstate` 持久化插件对 Map 类型的序列化限制已识别并采用手动 localStorage 读写策略规避

**[通过]** 语言特性用法在 TypeScript 能力范围内：
- 联合类型（如 `'diet' | 'exercise' | 'other'`、`'user' | 'admin' | null`）使用正确
- 泛型（`ApiResponse<T>`、`PaginatedResponse<T>`）定义规范
- `declare module 'vue-router'` 扩展路由元信息类型符合 Vue Router 4 的类型扩展机制
- `ref<LoginUser | null>(null)` 响应式变量声明符合 Vue 3 组合式 API

**[通过]** DDL 与 SQLite 3 能力匹配：
- `CHECK` 约束、`FOREIGN KEY ... ON DELETE CASCADE/SET NULL`、`datetime('now', 'localtime')` 均为 SQLite 支持的语法
- `json_extract()` 函数从 `result` JSON 列提取字段的查询设计符合 SQLite JSON1 扩展能力
- `INTEGER` 存储布尔值（0/1）配合 `CHECK(is_active IN (0, 1))` 是 SQLite 的标准实践

### 2. 完备性

**[通过]** 用户任务中的每个功能要求均有对应的技术方案说明：
- 4.1 系统首页 → Home.vue 组件树 + 路由配置 + 公开访问
- 4.2 医师咨询 → DoctorChatView.vue + SSE 流式对话 + 会话管理（chatStore Map 映射）+ AbortController 并发控制
- 4.3 个人中心与登录注册 → Login.vue + authStore + JWT 认证 + 路由守卫 + 管理员强制改密
- 4.4 糖尿病风险预测 → Risk.vue + riskFormStore 多步骤表单 + sessionStorage 持久化 + Dify 工作流
- 4.5 生活方案 → LifePlan.vue + plan_id 方案组机制 + is_active 逻辑过期 + Dify 工作流
- 4.6 健康资讯 → NewsView.vue + ArticleDetailView.vue + Markdown 渲染 + 收藏管理
- 4.7 打卡记录与分析 → Punch.vue + punch_in 表 + plan_id 外键依从性分析 + Dify 打卡分析工作流
- 4.8 AI 智能助手 → AiChatDialog.vue + Dify Agent + Text2SQL + FAB 全局悬浮按钮
- 4.9 智能管理 → Admin.vue + admin-manager-agent + /api/admin/execute 双认证
- 4.10 用户认证与会话管理 → JWT + bcrypt + authStore + 跨标签页同步
- 4.11 医学免责声明 → 路由守卫 requiresDisclaimer + SweetAlert2 确认弹窗 + 固定免责提示条

**[通过]** 数据流形成完整闭环：
- 常规 CRUD 路径：Vue 组件 → useApi → Express REST → better-sqlite3 → 返回 JSON
- AI 驱动 Text2SQL 路径：Vue 组件 → useSSE → Express 代理 → Dify Agent → Text2SQL 工具回调 /api/admin/execute → 执行 SQL → SSE 流返回
- AI 内容生成持久化路径：Vue 组件 → Express 端点 → Dify 工作流 → 解析 AI 输出 → INSERT/UPDATE 数据库 → 返回前端
- 跨模块数据传递：riskFormStore.saveResult() → LifePlan.vue onMounted 读取，形成风险预测→方案生成的数据流闭环

**[通过]** 迭代需求中的 10 个问题全部已修复：

1. **注册响应缺 JWT Token（严重）**：3.2.1 节注册响应已修改为与登录响应结构一致（含 `token`/`role`/`user`），3.8.2 节 `RegisterResponse` 类型复用 `LoginResponse` 结构，4.3 节 Login.vue 流程图注册成功后调用 `setAuth(token, role, user)` 自动登录并跳转首页。
2. **登录响应 role 字段位置 + authStore.login() 未设置 role（严重）**：3.2.2 节登录响应 `role` 已提升为顶层字段，3.8.2 节 `LoginResponse` 类型新增 `role: 'user' | 'admin'`，1.5.2 节 `authStore.login()` 伪代码设置 `role.value = res.data.role` 并写入 `localStorage('role')`，`role` 作为独立 ref 变量声明。
3. **life_plans 表缺 plan_id 列（严重）**：2.2 节 DDL 新增 `plan_id INTEGER NOT NULL` 列，2.3 节新增 `idx_plans_user_plan` 复合索引，2.5 节数据字典补充 plan_id 说明，3.2.14 节 PUT /api/plan/adjust 处理流程明确通过 `(user_id, plan_id)` 整体定位方案组，消除死参数问题。
4. **pregnancy 字段转换未文档化（一般）**：1.8.2 节各层枚举值规范表补充 pregnancy 行（DDL 层 `INTEGER(0/1)`、API/TS 层 `boolean`），1.8.4 节新增 `pregnancy` 字段端到端转换说明（Express `risk.js` 写入前 `pregnancy ? 1 : 0`，读取后 `row.pregnancy === 1`）。
5. **punch_type CHECK 含 'other' 违反需求 5 节（一般）**：2.2 节 DDL 恢复 `CHECK(punch_type IN ('diet', 'exercise'))`，2.5 节数据字典更新，1.8.1 节 `ENUM_LABELS.punch_type` 移除 'other'，文档说明 'other' 类型方案项仅供展示不支持打卡，与 PlanResponse.other_plans 形成一致性设计。
6. **PunchCreateResponse 类型泛化 string（一般）**：3.8.6 节 `PunchCreateResponse.punch_type` 改为 `'diet' | 'exercise'`，`completion_status` 改为 `'completed' | 'uncompleted'`，与 `PunchCreateRequest` 类型安全一致。
7. **User 接口与登录响应 user 对象不一致（一般）**：3.8.2 节新增 `LoginUser` 接口（仅含 `id`/`username`/`role`/`avatar`），`LoginResponse.user` 与 `RegisterResponse.user` 改为 `LoginUser` 类型，3.8.3 节 `User` 接口保留完整字段（含 `created_at`）用于 GET /api/user/profile，方案 A 落地完成。
8. **authStore 缺 clearMustChangePassword 方法（一般）**：3.7 节 `AuthActions` 接口新增 `clearMustChangePassword(): void` 方法声明，3.8.8 节 TypeScript 类型同步新增，4.3 节 ChangePassword.vue 流程图调用此方法。
9. **PlanResponse 缺 other_plans 字段（轻微）**：3.8.5 节 `PlanResponse` 和 `PlanCurrentResponse` 新增 `other_plans: LifePlan[]` 字段，3.2.13 节和 3.2.15 节 JSON 示例同步新增 `other_plans: []`，与 `LifePlan.plan_type` 联合类型含 'other' 的设计一致。
10. **articles tags 转换策略未文档化（轻微）**：1.8.4 节新增 TEXT 字段 JSON 序列化/反序列化规范，明确 `articles.tags` 和 `life_advice.tags` 的 Express 路由处理器转换职责（写入 `JSON.stringify`，读取 `JSON.parse`，异常降级为空数组）。

**[通过]** 无需实现者自行探索的技术方向性问题：
- 三条数据操作路径（常规 CRUD / AI Text2SQL / AI 内容生成持久化）均有详细流程图和端点规格
- 双认证模式（JWT + API Key）的鉴权逻辑和行级权限约束已明确
- Dify `{{user}}` 变量透传能力的门禁验证任务和替代方案已定义
- SSE 连接并发控制策略（同时活跃连接数上限 1）通过 `registerAbortController`/`abortActiveConnection` 实现

### 3. 可操作性

**[通过]** 技术方案中的每项说明均有明确结论：
- 技术选型表（1.3 节）为每个技术给出了版本、选型理由和引入方式
- 路由配置（1.6 节）给出了完整的路由表和 `beforeEach` 守卫伪代码
- DDL（2.2 节）给出了完整的 `CREATE TABLE` 语句和索引策略
- API 端点（3.2 节）为每个端点给出了请求体/响应体 JSON Schema 和 SQLite 查询设计
- TypeScript 类型（3.8 节）覆盖所有 API 请求/响应、Pinia Store、SSE 事件和业务实体
- 前端组件树（4.1 节）和流程图（4.3 节）给出了每个页面的 DOM 结构和交互逻辑

**[通过]** 实现者可从方案中明确知道"做什么"和"怎么做的大方向"：
- 后端开发者：从 DDL 直接创建数据库，从 API Schema 实现 Express 路由，从 SQLite 查询设计实现数据访问层
- 前端开发者：从组件树创建 .vue 文件，从 TypeScript 类型定义实现接口契约，从流程图实现交互逻辑
- Dify 配置者：从 5.2 节工作流定义和 5.3 节 Agent 配置实现 AI 能力

**[通过]** 方案中的技术引用足够具体：
- 库版本精确到主版本号（如 Vue 3.x、Express 4.x、better-sqlite3 9.x）
- 文件路径明确（如 `src/stores/authStore.ts`、`server/routes/risk.js`、`server/db/init.sql`）
- 代码示例完整（如 `authStore.login()` 伪代码、`router.beforeEach` 守卫、`useApi.ts` Axios 拦截器）
- 环境变量命名规范统一（如 `DIFY_API_BASE_URL`、`JWT_SECRET`、`DIFY_SERVICE_API_KEY`）

**[轻微]** `AuthState.user` 类型标注不一致：3.7 节和 3.8.8 节的 `AuthState` 接口中 `user` 字段类型为 `User | null`（含 `created_at`），但 1.5.2 节 `authStore` 实现伪代码中 `user` ref 声明为 `LoginUser | null`（不含 `created_at`）。登录响应返回的是 `LoginUser` 对象（无 `created_at`），赋值到 `User` 类型变量后 `created_at` 为 `undefined`。建议将 `AuthState.user` 类型统一为 `LoginUser | null`，或在注释中说明 `fetchProfile()` 调用后才会填充 `created_at`。此为类型标注精度的改进建议，不影响实现启动。

**[轻微]** `syncFromStorage()` 方法描述与 `login()` 持久化策略不完全匹配：1.5.2 节跨标签页同步描述中 `syncFromStorage()` 声称从 localStorage 恢复 `token/role/userInfo` 三个字段，但 `login()` 伪代码仅将 `token` 和 `role` 写入 localStorage，未持久化 `user` 对象。页面刷新后 `user` 为 null，需通过 `fetchProfile()` 从 API 恢复。建议在 `syncFromStorage()` 说明中明确 `user` 字段的恢复策略（仅恢复 token/role，user 需调用 `fetchProfile()` 获取），或在 `login()` 中补充 `user` 的 localStorage 持久化。此为跨标签页同步逻辑的完善建议，核心登录功能不受影响。

**[轻微]** 8.3 节验收标准表中注册成功的验收项描述仍为旧版：第 5805 行"注册成功 | POST /api/auth/register 返回 201，含 user_id 和 username"，但 3.2.1 节注册响应已更新为含 `token`/`role`/`user` 结构。建议更新验收标准为"返回 201，含 token、role 和 user 对象"。此为文档一致性改进建议，不影响 API 实现规格。

## 修改要求（不适用，APPROVED）

无阻塞性修改要求。上述轻微问题为改进建议，可在编码阶段同步修正，不影响实现启动。
