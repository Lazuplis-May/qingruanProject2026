# 质量审查报告 — v5 详细设计文档（第 5 轮）

## 审查范围

- **待审查产出**：`a_v5_copy_from_v4.md`（6572 行，含 v2~v13 共 12 轮内部修订说明）
- **用户需求**：`requirement.md`
- **历史迭代反馈**：`iteration_history.md`（迭代 1~4 轮共 37 个问题）
- **审查视角**：需求响应充分度、事实错误与逻辑矛盾、工程实施可指导性、技术决策完整性

## 总体评价

v5 产出经过 13 轮内部修订，已系统解决前 4 轮迭代反馈中的绝大多数问题（枚举值语言规范、SSE 401 处理、chatStore 方法命名、articles 字段补全、Dify Agent 工具定义、Vant 4 引入、admin_logs 字段重命名等）。文档在技术可行性维度已较为完善。

但本次审查从工程实施视角逐项核对需求契约与产出一致性后，仍发现若干关键问题——主要集中在认证接口契约偏离需求、数据库 schema 缺列导致 API 参数无支撑、类型定义与 API 响应不一致等方面。这些问题若不在实施前修复，将直接导致前后端联调失败或运行时错误。

---

## 严重问题（Critical）

### 问题 1：POST /api/auth/register 响应不返回 JWT Token，与需求 6.1 节直接矛盾

**所在位置**：3.2.1 节（第 1378~1388 行）、3.8.2 节 `RegisterResponse` 类型定义（第 2492~2495 行）、4.3 节 Login.vue 流程图（第 3788 行）

**问题描述**：

需求 6.1 节明确要求："端点 `/api/auth/register` 正常响应（HTTP 201）：**与登录响应结构一致**（注册成功后直接返回 JWT Token 和用户信息，**用户无需重复登录**）。"

但 v5 产出的注册响应仅返回 `{user_id, username}`，不含 token、role、user 对象：

```json
// v5 3.2.1 节响应
{
  "success": true,
  "message": "注册成功",
  "data": {
    "user_id": 1,
    "username": "newuser"
  }
}
```

对应的 TypeScript 类型也只定义了 `{user_id, username}`：
```typescript
interface RegisterResponse {
  user_id: number;
  username: string;
}
```

此外，4.3 节 Login.vue 流程图显示注册成功后的流程为"SweetAlert2 提示'注册成功' → 自动切换至登录视图 → 预填用户名"，即要求用户手动登录，与需求"用户无需重复登录"直接矛盾。

**严重程度**：严重

**改进建议**：
1. 将 3.2.1 节注册成功响应修改为与登录响应结构一致（含 `token`、`user` 对象、可选 `must_change_password`）。
2. 将 `RegisterResponse` 类型定义更新为与 `LoginResponse` 一致（或直接复用 `LoginResponse` 类型）。
3. 更新 4.3 节 Login.vue 流程图，注册成功后调用 `authStore.login()` 逻辑（或等效的 setAuth），自动登录并跳转至首页，而非切换至登录视图。

---

### 问题 2：POST /api/auth/login 响应中 role 字段位置与需求 6.1 节不一致，authStore.login() 伪代码未设置 role 状态

**所在位置**：3.2.2 节登录响应（第 1420~1434 行）、3.8.2 节 `LoginResponse` 类型（第 2486~2490 行）、1.5.2 节 `authStore.login()` 伪代码（第 349~365 行）、3.7 节 AuthActions 接口说明（第 2337 行）

**问题描述**：

需求 6.1 节定义的登录响应结构中，`role` 为顶层字段（与 `token`、`user` 平级）：

```json
// 需求 6.1 节
{
  "token": "eyJ...",
  "role": "user",
  "user": { "id": 1, "username": "zhangsan", "avatar": "..." }
}
```

但 v5 产出将 `role` 放在 `user` 对象内部，且 `user` 对象多了 `role` 字段：

```json
// v5 3.2.2 节
{
  "success": true,
  "data": {
    "token": "eyJ...",
    "user": { "id": 1, "username": "user1", "role": "user", "avatar": "..." }
  }
}
```

同时，`LoginResponse` TypeScript 类型完全缺少 `role` 字段定义：

```typescript
interface LoginResponse {
  token: string;
  user: User;
  must_change_password?: boolean;
  // 缺少 role 字段
}
```

更严重的是，1.5.2 节 `authStore.login()` 伪代码只设置了 `token` 和 `user`，**未设置 `role` 状态变量**，也未将 `role` 写入 localStorage：

```typescript
async function login(username: string, password: string) {
  const res = await api.post<LoginResponse>('/api/auth/login', { username, password });
  token.value = res.data.token;
  localStorage.setItem('token', res.data.token);
  user.value = res.data.user;
  // 未设置 role.value，未写入 localStorage('role')
}
```

而 3.7 节 AuthActions 接口注释明确说"同时将 role 写入 localStorage('role')"，与伪代码实现矛盾。

**影响分析**：
- `AuthState` 中 `role` 是独立状态变量，路由守卫（1.6.2 节第 528~537 行）依赖 `authStore.role === 'admin'` 判断管理员权限
- 若 `authStore.login()` 不设置 `role`，登录后 `authStore.role` 恒为 `null`，导致：
  - 管理员登录后路由守卫第 4 步 `requiresAdmin && authStore.role !== 'admin'` 恒为 true，管理员被重定向至 `/home`，无法访问 `/admin`
  - Profile.vue 中管理员菜单项不渲染（依赖 `authStore.isAdmin` 计算属性）

**严重程度**：严重

**改进建议**：
1. 明确 `role` 字段在登录响应中的位置——建议遵循需求 6.1 节将 `role` 放在顶层（与 `token` 平级），或在文档中显式说明偏离理由并确保前后端一致。
2. 在 `LoginResponse` 类型中补充 `role: 'user' | 'admin'` 字段（位置与 API 响应一致）。
3. 修正 1.5.2 节 `authStore.login()` 伪代码，从响应中提取 `role` 并设置到 `role.value` 和 `localStorage.setItem('role', role)`。

---

### 问题 3：life_plans 表缺少 plan_id（方案组 ID）列，PUT /api/plan/adjust 接收的 plan_id 参数无数据库支撑

**所在位置**：2.2 节 life_plans DDL（第 884~896 行）、3.2.13 节 POST /api/plan/generate 响应（第 1745 行）、3.2.14 节 PUT /api/plan/adjust 请求体（第 1781 行）、3.8.5 节 PlanResponse/PlanAdjustRequest 类型（第 2643~2652 行）

**问题描述**：

需求 6.5 节明确定义 `plan_id` 为方案组 ID：

> `plan_id`（number）：方案组 ID（**同一批生成的所有方案项共享此 plan_id**，用于后续方案调整的整体替换和 GET /api/plan/current 的活跃方案查询）

但 v5 的 `life_plans` DDL 中**无 `plan_id` 或 `group_id` 列**：

```sql
CREATE TABLE IF NOT EXISTS life_plans (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    plan_type TEXT NOT NULL CHECK(plan_type IN ('diet', 'exercise', 'other')),
    order_num INTEGER NOT NULL DEFAULT 0,
    time_desc TEXT DEFAULT '',
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1)),
    created_at TEXT NOT NULL DEFAULT (datetime('now', 'localtime')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now', 'localtime')),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
-- 无 plan_id 列
```

然而 API 层却返回和接收 `plan_id`：
- 3.2.13 节响应包含 `"plan_id": 1`（此值的来源未说明，可能是第一条方案项的 id）
- 3.2.14 节 PUT /api/plan/adjust 请求体接收 `plan_id` 参数，但后端无法通过 plan_id 定位"同一批生成的方案项"

当前 PUT /api/plan/adjust 的实现逻辑（依据需求 4.5 节"通过 is_active 逻辑过期机制"）实际上是将当前用户的所有 `is_active=1` 方案标记为 `is_active=0`，再生成新方案——`plan_id` 参数在此流程中完全未被使用，成为死参数。

**影响分析**：
- 若未来需要支持"多套方案共存"或"按方案组删除/调整"，缺少 plan_id 列将无法实现
- API 契约返回的 plan_id 值语义不明（是首条方案项 id？还是临时生成的？），前端无法可靠引用
- PUT /api/plan/adjust 接收 plan_id 但不使用，接口契约有误导性

**严重程度**：严重

**改进建议**：
方案 A（推荐）：在 `life_plans` 表新增 `plan_id INTEGER NOT NULL` 列（同一批生成的方案项共享相同 plan_id 值），并在 DDL、数据字典、ER 图中同步补充。PUT /api/plan/adjust 通过 `UPDATE life_plans SET is_active=0 WHERE user_id=? AND plan_id=?` 精确定位待调整的方案组。

方案 B（简化）：若确认"每用户仅保留一套活跃方案"是最终设计决策，则在文档中明确说明 plan_id 的语义（如"plan_id 为首条方案项 id，仅用于前端引用，后端调整时不依赖此参数"），并从 PUT /api/plan/adjust 请求体中移除 plan_id 参数（改为仅接收 feedback），消除死参数。

---

## 一般问题（Major）

### 问题 4：pregnancy 字段 boolean ↔ integer 转换机制未文档化

**所在位置**：2.2 节 user_risk_info DDL（第 874 行 `pregnancy INTEGER`）、3.2.7 节请求体（第 1547 行 `"pregnancy": false`）、3.8.4 节 RiskPredictRequest（第 2600 行 `pregnancy?: boolean`）、5.2.1 节 Dify 输入变量（第 4349 行 `pregnancy | boolean`）、1.8 节各层枚举值规范表（第 649~655 行）

**问题描述**：

v13 修订将 `pregnancy` 的 DDL 类型从 TEXT 改为 `INTEGER`（存储 0/1），但 API 请求/响应和 TypeScript 类型使用 `boolean`。v13 同时移除了后端 `mapper.js` 转换层（1.8 节声明"原 server/utils/mapper.js 后端转换层移除"）。

然而：
- 1.8.2 节"各层枚举值规范"对照表只覆盖了 gender/family_history/diabetes_history/plan_type/punch_type/completion_status 六个字段，**未包含 pregnancy 字段**
- pregnancy 的 boolean ↔ integer 转换由谁负责（Express 路由处理器？Dify 工作流？）未在任何章节说明
- 5.2.1.1 节"端到端字段映射契约"表格中也未列出 pregnancy 的各层映射

**影响**：后端开发者在实现 `risk.js` 路由处理器时，不知道需要将 API 接收的 `boolean` 转换为 `INTEGER(0/1)` 写入 SQLite，也不知道从 SQLite 读取 `INTEGER` 后需转换回 `boolean` 返回前端。Dify 工作流输入变量的 pregnancy 类型为 boolean，但 Dify 无法直接写入 SQLite 的 INTEGER 列。

**严重程度**：一般

**改进建议**：
1. 在 1.8.2 节各层枚举值规范表中补充 `pregnancy` 行，明确 DDL 层为 `INTEGER(0/1)`、API/TS 层为 `boolean`。
2. 在 5.2.1.1 节端到端字段映射契约中补充 pregnancy 的转换说明（Express risk.js 在写入前 `pregnancy ? 1 : 0`，读取后 `row.pregnancy === 1`）。
3. 或在 3.2.7 节请求体注释中补充"后端将 boolean 转换为 INTEGER(0/1) 存储"的说明。

---

### 问题 5：punch_in.punch_type CHECK 约束包含 'other' 与需求 5 节直接矛盾

**所在位置**：2.2 节 punch_in DDL（第 915 行）、2.5 节 punch_in 数据字典（第 1248 行）

**问题描述**：

需求 5 节对 punch_in 表 punch_type 字段的定义明确为：

> 打卡类型（punch_type，TEXT，CHECK(punch_type IN ('diet', 'exercise'))，对应中文展示为饮食/运动）

但 v5 DDL 扩展为包含 'other'：

```sql
punch_type TEXT NOT NULL CHECK(punch_type IN ('diet', 'exercise', 'other')),
```

数据字典中也说明"该三种类型方案项均可正常生成展示与打卡"。

v3 迭代第 3 个问题曾指出 `life_plans.plan_type` 与 `punch_in.punch_type` 约束维度不一致，v5 的解决方案是将 punch_type 也扩展为包含 'other' 以与 plan_type 对齐。但此扩展**直接违反需求 5 节的显式定义**，且文档中未说明此偏离需求的理由。

**影响**：需求 5 节定义的打卡类型仅有饮食/运动两类（对应 4.7 节"按饮食/运动类型的完成率"分析维度），扩展 'other' 后打卡分析维度的 `diet_completion_rate` 和 `exercise_completion_rate` 无法覆盖 'other' 类型打卡数据。

**严重程度**：一般

**改进建议**：
方案 A（遵循需求）：将 punch_type CHECK 约束恢复为 `IN ('diet', 'exercise')`，在文档中说明 'other' 类型的方案项仅供展示，不支持打卡。
方案 B（保留扩展但说明偏离）：若确认扩展 'other' 是合理设计决策，在 2.5 节数据字典中显式标注"此为偏离需求 5 节的设计决策，理由：与 life_plans.plan_type 枚举维度对齐"，并在 3.2.18 节打卡分析响应中补充 `other_completion_rate` 字段。

---

### 问题 6：PunchCreateResponse 类型使用泛化 string 而非联合类型，类型安全不足

**所在位置**：3.8.6 节 PunchCreateResponse（第 2672~2679 行）

**问题描述**：

`PunchCreateRequest` 中 `punch_type` 和 `completion_status` 使用了精确的联合类型：

```typescript
interface PunchCreateRequest {
  punch_type: 'diet' | 'exercise' | 'other';
  completion_status: 'completed' | 'uncompleted';
  ...
}
```

但 `PunchCreateResponse` 中同名字段却退化为 `string`：

```typescript
interface PunchCreateResponse {
  id: number;
  plan_id: number;
  punch_type: string;           // 应为 'diet' | 'exercise' | 'other'
  completion_status: string;    // 应为 'completed' | 'uncompleted'
  remarks: string;
  punch_time: string;
}
```

**影响**：前端在处理打卡响应时无法利用 TypeScript 类型系统对枚举值进行编译期校验，降低了类型安全性，可能导致拼写错误在运行时才暴露。

**严重程度**：一般

**改进建议**：将 `PunchCreateResponse` 中的 `punch_type` 和 `completion_status` 改为与 `PunchCreateRequest` 一致的联合类型。

---

### 问题 7：User 接口字段与登录响应中 user 对象不一致

**所在位置**：3.8.3 节 User 接口（第 2503~2509 行）、3.2.2 节登录响应 user 对象（第 1426~1431 行）、3.2.4 节 GET /api/user/profile 响应（第 1483~1488 行）

**问题描述**：

`User` 接口定义包含 5 个字段：

```typescript
interface User {
  id: number;
  username: string;
  role: 'user' | 'admin';
  avatar: string | null;
  created_at: string;  // ← 此字段
}
```

但不同接口返回的 user 对象字段不一致：
- **3.2.2 登录响应**的 user 对象：`{id, username, role, avatar}` — **缺少 `created_at`**
- **3.2.4 GET /api/user/profile** 响应：`{id, username, role, avatar, created_at}` — 字段完整

前端若用 `User` 类型接收登录响应，`created_at` 字段实际为 `undefined`。若组件中使用了 `user.created_at`（如 Profile.vue 展示注册时间），在从登录响应获取 user 后直接渲染将出错。

**严重程度**：一般

**改进建议**：
方案 A：为登录响应定义单独的 `LoginUser` 类型（仅含 id/username/role/avatar），`User` 类型保留完整字段用于 GET /api/user/profile。
方案 B：统一登录响应的 user 对象包含 created_at 字段（后端在登录时联查 users 表的 created_at 列）。

---

### 问题 8：authStore 缺少 clearMustChangePassword 方法定义，但 ChangePassword.vue 流程图中调用了此方法

**所在位置**：3.7 节 AuthActions 接口（第 2336~2344 行）、4.3 节 ChangePassword.vue 流程图（第 3814 行）

**问题描述**：

4.3 节 ChangePassword.vue 流程图中包含调用：

```
P[authStore.clearMustChangePassword<br/>Pinia响应式自动通知]
```

但 3.7 节 `AuthActions` 接口定义中**未声明 `clearMustChangePassword` 方法**，仅包含：login、logout、setToken、setAuth、syncFromStorage、clearAuth、fetchProfile。

**影响**：前端开发者按流程图实现时，调用未定义的 Store 方法将导致 TypeScript 编译错误或运行时 `undefined is not a function` 错误。

**严重程度**：一般

**改进建议**：在 3.7 节 `AuthActions` 接口中补充 `clearMustChangePassword(): void` 方法定义，说明其职责为将 `mustChangePassword` 状态置为 `false` 并同步至 localStorage（或 pinia-plugin-persistedstate 持久化）。

---

## 轻微问题（Minor）

### 问题 9：PlanResponse 类型缺少 other_plans 字段，无法承载 'other' 类型的方案项

**所在位置**：3.8.5 节 PlanResponse/PlanCurrentResponse（第 2648~2659 行）

**问题描述**：

`LifePlan.plan_type` 联合类型包含 `'other'`，但 `PlanResponse` 只有 `diet_plans` 和 `exercise_plans` 两个分组数组，无 `other_plans`。若 Dify 工作流未来输出 'other' 类型方案项，API 响应无处安放。

当前 Dify 工作流（5.2.2 节）仅生成 diet（4 项）和 exercise（3 项），此问题暂不会触发，但类型系统与 API 响应结构的设计不一致。

**严重程度**：轻微

**改进建议**：若确认 'other' 类型方案不会生成，可在 LifePlan.plan_type 联合类型中移除 'other'（仅保留 'diet' | 'exercise'）；若保留 'other'，则在 PlanResponse 中补充 `other_plans: LifePlan[]` 字段。

---

### 问题 10：articles 表 tags 字段 TEXT 存储 JSON 数组，转换策略未在 1.8 节规范表中列出

**所在位置**：2.2 节 articles DDL（第 836 行 `tags TEXT NOT NULL DEFAULT '[]'`）、3.8.3 节 Article 接口（第 2527 行 `tags: string[]`）、1.8 节各层枚举值规范表

**问题描述**：

articles 表的 `tags` 列在 DDL 中为 `TEXT` 类型存储 JSON 数组字符串（如 `'["饮食", "血糖管理"]'`），但 TypeScript `Article` 接口中为 `tags: string[]`。这需要 Express 路由处理器在写入时 `JSON.stringify()`、读取时 `JSON.parse()`。

1.8 节"数据字段映射与中英转换机制设计"仅覆盖枚举值转换，未将 tags 的 JSON 序列化/反序列化纳入转换规范表。`life_advice` 表的 `tags` 字段（第 903 行）存在同样问题。

**严重程度**：轻微

**改进建议**：在 1.8 节或 3.6 节补充说明：articles 和 life_advice 表的 tags 字段在 DDL 中为 TEXT 存储 JSON 数组字符串，Express 路由处理器负责 JSON.parse/JSON.stringify 转换。或在 3.2.19/3.2.20 节响应字段说明中标注 tags 字段的序列化方式。

---

## 已确认解决的历史问题

经逐项核对，迭代 1~4 轮反馈中的以下关键问题已在 v5 中正确修复：

- 迭代 1：DDL 枚举值中英混杂问题（v13 统一为英文枚举值 + 前端 enumLabel 映射）
- 迭代 1：user_risk_info 缺少 diabetes_history 字段（已补充）
- 迭代 1：life_plans 缺少 is_active 字段（已补充）
- 迭代 1：前端路由组件未拆分问题（已拆分 ConsultationView/DoctorChatView、NewsView/ArticleDetailView）
- 迭代 2：difyAuth.js timingSafeEqual 长度不一致 DoS 隐患（改用 SHA-256 哈希后比较）
- 迭代 2：router/index.ts 语法错误（已修正）
- 迭代 2：mapper.js 漏配 punch_type 映射（v13 移除 mapper.js，统一英文枚举值无需转换）
- 迭代 3：/api/admin/execute 硬编码 operatorRole='user'（改为动态查询 users 表）
- 迭代 3：validateRowLevelPermission 缺失实现规范（7.3.4 节补充 AST 解析方案）
- 迭代 4：useSSE.ts 未处理 401 响应（已补充 401 检查分支）
- 迭代 4：chatStore 方法命名不一致（已对齐需求 4.10 节）
- 迭代 4：POST /api/risk/predict 0 值校验 HTTP 状态码（改为 422）
- 迭代 4：GET /api/articles/:id 缺少 is_collected（已补充）
- 迭代 4：GET /api/articles 缺少 tags/summary（已补充）
- 迭代 4：Dify Agent 工具定义（5.2.5/5.2.6 节补充 8+1 和 5+1 工具）
- 迭代 4：未引入 Vant 4（1.3 节和 package.json 已补充）
- 迭代 4：DoctorChatView.vue 路由参数变化 SSE 连接（补充 watch route.params.id）
- 迭代 4：AiChatDialog.vue 缺少独立流程图（已补充）
- 迭代 4：difyService.js blocking 超时 60s（改为 15s）
- 迭代 4：admin_logs.admin_id 语义混淆（重命名为 operator_id）
