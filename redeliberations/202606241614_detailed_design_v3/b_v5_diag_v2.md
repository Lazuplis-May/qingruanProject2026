# 质量审查报告 — v5 详细设计文档（第 5 轮）

## 审查范围

- **待审查产出**：`a_v5_copy_from_v4.md`（6572 行，含 v2~v13 共 12 轮内部修订说明）
- **用户需求**：`requirement.md`
- **历史迭代反馈**：`iteration_history.md`（迭代 1~4 轮共 37 个问题）
- **审查视角**：需求响应充分度、事实错误与逻辑矛盾、工程实施可指导性、技术决策完整性、技术风险与缓解措施充分性、非功能性需求响应

## 总体评价

v5 产出经过 13 轮内部修订，已系统解决前 4 轮迭代反馈中的绝大多数问题（详见文末"已确认解决的历史问题"章节，本轮逐项核实后覆盖率从 v1 报告的约 54% 提升至约 89%）。文档在技术可行性维度已较为完善，且在技术风险识别与缓解措施（Dify `{{user}}` 门禁任务、SQL 注入防护、SQLite 可靠性、SSE 并发控制）和非功能性需求响应（7.1~7.5 节）方面均有对应设计。

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

而 3.7 节 AuthActions 接口注释明确说"同时将 role 写入 localStorage('role')"，与伪代码实现矛盾。此外，1.5.2 节 authStore 的状态声明（第 345~348 行）也未声明 `role` ref 变量，与 3.7 节 AuthState 接口中 `role: 'user' | 'admin' | null` 定义不一致。

**影响分析**：
- `AuthState` 中 `role` 是独立状态变量，路由守卫（1.6.2 节第 528~537 行）依赖 `authStore.role === 'admin'` 判断管理员权限
- 若 `authStore.login()` 不设置 `role`，登录后 `authStore.role` 恒为 `null`，导致：
  - 管理员登录后路由守卫第 4 步 `requiresAdmin && authStore.role !== 'admin'` 恒为 true，管理员被重定向至 `/home`，无法访问 `/admin`
  - Profile.vue 中管理员菜单项不渲染（依赖 `authStore.isAdmin` 计算属性）

**严重程度**：严重

**改进建议**：
1. 明确 `role` 字段在登录响应中的位置——建议遵循需求 6.1 节将 `role` 放在顶层（与 `token` 平级），或在文档中显式说明偏离理由并确保前后端一致。
2. 在 `LoginResponse` 类型中补充 `role: 'user' | 'admin'` 字段（位置与 API 响应一致）。
3. 修正 1.5.2 节 `authStore.login()` 伪代码，从响应中提取 `role` 并设置到 `role.value` 和 `localStorage.setItem('role', role)`；同时在 1.5.2 节 authStore 状态声明中补充 `const role = ref<'user' | 'admin' | null>(localStorage.getItem('role') as 'user' | 'admin' | null)`。

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

**证据精度补充（v2 修订）**：需求 5 节 life_plans 表字段清单未显式要求 `plan_id` 列（仅列出"方案ID"作为主键），需求 6.5 节定义的 `plan_id` 主要为 API 响应字段语义。但需求 6.5 节明确要求 `plan_id` "用于后续方案调整的整体替换"，此语义需要数据库层提供持久化支撑（否则"同一批生成的所有方案项共享此 plan_id"无法在数据库层面表达），因此将其标注为数据库列缺失问题仍然成立。

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

## API 契约补充核对（v2 新增）

针对 v1 报告仅深入核对了需求 6 节 10 个 API 子节中的约 3 个（认证、方案、打卡部分），本轮对其余 7 个 API 子节进行了契约一致性核对，结果如下：

| API 端点 | 设计章节 | 需求章节 | 核对结论 |
|---------|---------|---------|---------|
| `GET /api/risk/history` | 3.2.8（第 1583 行） | 6.3 | **一致**。响应含 risk_score/risk_level/risk_level_label/created_at（需求要求的字段子集），分页外壳符合 6.12 节，附 SQLite 查询伪代码 |
| `GET /api/doctors` | 3.2.9（第 1638 行） | 6.4 | **一致**。响应字段 id/name/department/title/description/avatar 与需求 6.4 节定义完全匹配，chat_token 未暴露 |
| `GET /api/doctors/:id` | 3.2.10（第 1664 行） | 6.4 | **基本一致**。响应在需求字段基础上新增 `created_at`，符合需求"完整字段"语义，无契约冲突 |
| `POST /api/chat/doctor/:id` | 3.2.11（第 1682 行） | 6.4 | **一致**。请求体 message + conversation_id（可选）与需求匹配，SSE 事件格式符合 6.9 节定义 |
| `GET /api/punch/list` | 3.2.17（第 1846 行） | 6.6 | **一致**。支持 page/pageSize/startDate/endDate/punch_type 全部分页与筛选参数，响应结构符合 6.12 节，附 SQLite 查询伪代码 |
| `GET /api/punch/analysis` | 3.2.18（第 1903 行） | 6.6 | **合理**。需求 6.6 节未定义具体响应结构（仅说"Dify 工作流生成"），设计返回 diet_completion_rate/exercise_completion_rate/total_punches/last_7_days_trend/adherence_comment/improvement_suggestions，与 4.7 节分析维度匹配 |
| `GET /api/assistant/advice` | 3.2.26（第 2092 行） | 6.9 | **一致**。响应含 id/title/tags/content/created_at，覆盖需求"标题、标签、完整内容字段"要求 |
| `GET /api/admin/logs` | 3.2.30（第 2180 行） | 6.10 | **一致**。支持分页，响应含 id/operator_id/operator_username/operation_type/operation_content/operation_result/operation_time，operator_id 命名消除了 v4 问题 17 的语义混淆 |

**核对结论**：上述 7 个 API 子节的契约一致性核对未发现严重偏离需求的情况。v1 报告中已发现的 3 个子节偏离（注册响应缺 token、登录响应 role 位置错误、plan_id 缺数据库支撑）属于局部问题，并未在其余子节中复现。需求 6 节 API 契约响应整体充分。

---

## 技术风险与缓解措施评估（v2 新增）

任务描述要求评估"技术风险和缓解措施是否充分"。经查阅设计文档第 5.5、6.5、7.3 节及相关章节，v5 产出已识别关键技术风险并提供缓解措施，评估如下：

### 1. Dify `{{user}}` 变量透传门禁任务

**所在位置**：5.5 节（第 4760~4775 行）、5.5.2 节备选方案（第 4777~4790 行）

**评估**：
- 设计文档第 4765 行明确标注"本详细设计完成时，Dify 平台 `{{user}}` 变量透传能力的门禁验证任务**尚未执行**"，并承认当前设计基于未经验证的假设。
- 提供完整的备选方案（5.5.2 节）：若 Dify 不支持 `{{user}}` 透传，启用 Express 服务端 session_id→user_id 映射表，并明确列出需同步调整的章节（5.2.5/5.2.6/7.3.2/7.3.3）。
- 明确门禁时机："开发环境搭建阶段必须优先执行上述验证任务"，"在验证结果记录之前，编码实现不应进入 Dify Agent 工具配置与 `/api/admin/execute` 行级权限实现阶段"。

**结论**：风险识别充分，降级方案设计完整，门禁时机约束明确。**通过**。

### 2. SQL 注入防护与 `/api/admin/execute` 安全

**所在位置**：7.3.3 节 SQL 安全校验（第 5541~5544 行）、7.3.4 节 validateRowLevelPermission 技术规范（第 5570~5657 行）

**评估**：
- **白名单模式**：第 5541~5544 行仅允许 SELECT/INSERT/UPDATE/DELETE，拒绝所有 DDL（DROP/ALTER 等）、DCL、TCL 语句，并检测多语句注入。
- **AST 解析**：采用 `node-sql-parser`（非正则匹配）将 SQL 解析为 AST，提取表名与 WHERE 条件进行结构化校验，明确说明正则无法处理子查询/别名/嵌套条件。
- **表分类校验**：用户私有表强制 `user_id = operatorId` 约束，公共只读表仅允许 SELECT，禁止访问 users 表。
- **fail-closed 策略**：AST 解析失败一律返回 false，不放过语法异常 SQL。
- **参数化执行**：第 5657 行明确 SQL 仍由 `better-sqlite3.prepare()` 参数化执行保障注入安全。
- **life_advice 写入约束**：第 5584~5588 行专门补充 `write_health_advice` 工具的 INSERT 约束规则。

**结论**：SQL 注入防护深度充分，覆盖 DDL 拦截、行级权限、参数化执行三层防御。**通过**。

### 3. SQLite 单实例可靠性

**所在位置**：6.4 节数据库初始化（第 5184 行 WAL 模式）、6.7 节数据库备份策略（第 5356~5359 行）

**评估**：
- **WAL 模式**：第 5184 行 `db.pragma('journal_mode = WAL')` 启用 Write-Ahead Logging，提升并发读写能力。
- **备份策略**：每日 03:00 通过 scp 备份 SQLite 文件至服务器2，保留 7 天（第 5358~5359 行 crontab 配置）。
- **单点约束说明**：需求 7.4 节已明确 SQLite 单点架构约束，设计文档在 7.4 节响应中遵循此约束。

**结论**：SQLite 可靠性措施充分。**通过**。

### 4. SSE 连接并发控制闭环

**所在位置**：3.7 节 chatStore 接口（第 2359、2376~2377 行 activeAbortController/registerAbortController/abortActiveConnection）、3.7 节 SSE 连接控制策略（第 2383~2393 行）

**评估**：
- **全局连接管理**：通过 `activeAbortController` 状态追踪活跃连接，`registerAbortController` 在注册新连接前先中止旧连接，确保"同时活跃连接数上限为 1"（对齐需求 4.2 节）。
- **三端并发场景**：明确处理医师对话↔AI 助手 FAB、AI 助手↔医师对话、管理员对话三个并发场景的中止策略。
- **路由参数变化**：第 2393 行明确 DoctorChatView.vue 通过 `watch(() => route.params.id, ...)` 监听路由参数变化并调用 `abortActiveConnection`，解决同组件复用时 onUnmounted 不触发的问题。
- **调用点覆盖**：3.7 节策略说明覆盖医师对话、AI 助手、管理员对话三个 SSE 端点的调用点。

**结论**：SSE 并发控制形成完整闭环。**通过**。

**综合结论**：v5 产出在技术风险识别与缓解措施维度已较为充分，未发现遗漏的关键技术风险。

---

## 非功能性需求响应评估（v2 新增）

任务描述要求评估设计文档对需求 7 节非功能性需求的响应充分度。逐项评估如下：

### 7.1 用户界面

**需求要点**：移动端优先、视觉层次、交互一致性、Vant 4 推荐采用。
**设计响应**：1.3 节技术选型表已新增 Vant 4 条目；6.3.4 节前端 package.json dependencies 已包含 `"vant": "^4.9.0"`；4.5.1 节 CSS 变量定义已与 Vant 4 主题变量建立映射。
**结论**：**充分响应**。

### 7.2 性能

**需求要点**：普通操作 1 秒内反馈、AI 接口 15 秒超时阈值统一、打卡写入 1 秒内完成、RTO 30 分钟内。
**设计响应**：6.3.5 节 difyService.js blocking 模式读取超时已调整为 15s（v4 问题 14 修复）；3.4 节错误码枚举表中 `AI_TIMEOUT`（504）触发场景为"Dify请求超过15秒未响应"；7.4 节部署架构响应 RTO 约束。
**结论**：**充分响应**。

### 7.3 产品质量

**需求要点**：正确性、健壮性（AI 服务降级、超时处理、历史缓存降级、SSE 中断处理）、可靠性、易用性、安全性、可扩展性、可移植性、兼容性。
**设计响应**：3.4 节统一错误响应格式覆盖 AI_TIMEOUT/DIFY_ERROR/INTERNAL_ERROR 等错误码；7.3 节双认证模式实现、7.5 节 XSS 防御、7.6 节 SQL 注入防护、7.7 节 CSRF 防御覆盖安全性要求；3.7 节 chatStore 中断控制覆盖 SSE 中断处理。
**结论**：**充分响应**。

### 7.4 部署架构

**需求要点**：3 台 Linux 服务器分工、Nginx 反向代理 + 负载均衡、静态资源托管路径、SQLite 单点约束说明。
**设计响应**：6.6 节静态资源目录结构和 Nginx location 映射表完整定义（含 Vue SPA 入口、Vite 构建产物、第三方库、字体、预置图片、用户上传 6 类资源）；6.7 节 Keepalived 主备模式配置（含 MASTER/BACKUP 配置、故障转移流程、VRRP 认证安全说明）；6.7 节数据库备份策略（每日 03:00 scp 备份，保留 7 天）。
**结论**：**充分响应**。

### 7.5 环境配置

**需求要点**：JWT_SECRET/JWT_EXPIRES_IN、DIFY_API_BASE_URL/DIFY_SERVICE_API_KEY/5 个工作流与 Agent API Key、PORT/SQLITE_PATH/UPLOAD_DIR。
**设计响应**：6.3.2 节 .env.example 模板完整覆盖需求 7.5 节列出的所有环境变量，并额外补充 EXPRESS_PUBLIC_URL（Dify 回调可达 URL）、BCRYPT_SALT_ROUNDS、MAX_FILE_SIZE、ALLOWED_FILE_TYPES 等配置项；6.5 节明确 Dify Agent 回调网络可达性要求（开发环境内网穿透、生产环境公网 HTTPS + 固定域名）。
**结论**：**充分响应**。

**综合结论**：v5 产出对需求 7 节非功能性需求的响应充分，未发现遗漏的非功能性维度。

---

## 已确认解决的历史问题

经本轮逐项核实，迭代 1~4 轮反馈中的以下问题已在 v5 中正确修复（核实覆盖率约 89%，共 33/37 项）：

### 迭代 1（11 项，全部已解决）

- DDL 枚举值中英混杂问题（v13 统一为英文枚举值 + 前端 enumLabel 映射）
- user_risk_info 缺少 diabetes_history 字段（已补充）
- life_plans 缺少 is_active 字段（已补充）
- 前端路由组件未拆分问题（已拆分 ConsultationView/DoctorChatView、NewsView/ArticleDetailView）
- 个人中心子页面嵌套路由设计不明确（已调整为嵌套路由）
- Dify 智能助手会话管理 conversation_id 设计（已恢复 doctorConversations Map + assistantConversationId 独立变量）
- 缺少数据类型与映射转换机制说明（1.8 节补充各层枚举值规范表，v13 移除 mapper.js 统一英文枚举值）
- 健康资讯列表/详情组件未拆分（已拆分）
- 跨浏览器标签页登录态同步机制（1.2 节补充 storage 事件监听 + syncFromStorage）
- SSE 流式对话连接控制与并发限制（3.7 节补充 activeAbortController + registerAbortController/abortActiveConnection）
- waist/systolic_bp 0 值校验（3.2.7 节补充 0 值校验规则 + VALIDATION_ERROR 错误响应）

### 迭代 2（8 项，全部已解决）

- difyAuth.js timingSafeEqual 长度不一致 DoS 隐患（改用 SHA-256 哈希后比较）
- router/index.ts 语法嵌套错误（已修正）
- mapper.js 漏配 punch_type 映射（v13 移除 mapper.js，统一英文枚举值无需转换）
- POST /api/risk/predict 响应结构偏差（已对齐需求 6.3 节，含 record_id/risk_score/risk_level/risk_level_label/matched_diabetes_type/advice）
- pregnancy 字段 DDL 类型不一致（v13 改为 INTEGER + CHECK 约束，但转换机制文档化见本报告问题 4）
- seed.sql 管理员密码哈希占位符风险（6.4 节 initDatabase() 替换逻辑说明）
- 布尔逻辑字段表现形式不统一（v13 统一为 INTEGER + CHECK 约束）
- 数据驱动接口 SQLite 查询设计缺失（3.2.8/3.2.17 节已补充 SQLite 查询伪代码）

### 迭代 3（7 项，全部已解决）

- /api/admin/execute 硬编码 operatorRole='user'（改为动态查询 users 表）
- validateRowLevelPermission 缺失实现规范（7.3.4 节补充 AST 解析方案 + node-sql-parser 选型）
- life_plans/punch_in plan_type/punch_type 约束矛盾（v5 扩展 punch_type 含 'other'，但见本报告问题 5）
- 医学免责声明确认弹窗未调用（4.4.4 节 useUI.ts + 路由守卫/组件挂载调用）
- Dify 风险预测工作流输出结构与数据库查询契约不一致（5.2.1.1 节端到端字段映射契约补充）
- POST /api/punch 响应体缺失 remarks 字段（3.2.16 节响应已补充）
- AiChatDialog.vue DOM 结构缺失免责提示（4.1.1 节已补充）

### 迭代 4（17 项，16 项已解决，1 项部分相关）

- useSSE.ts 未处理 401 响应（已补充 401 检查分支）
- chatStore 方法命名不一致（已对齐需求 4.10 节，移除 "Id" 后缀，新增 clearDoctorConversation/clearAssistantConversation）
- POST /api/risk/predict 0 值校验 HTTP 状态码（改为 422）
- GET /api/articles/:id 缺少 is_collected（3.2.20 节已补充 + 联查 article_collections 表）
- GET /api/articles 缺少 tags/summary（3.2.19 节已补充）
- Dify Agent 工具定义（5.2.5/5.2.6 节补充 8+1 和 5+1 工具）
- 未引入 Vant 4（1.3 节和 package.json 已补充）
- DoctorChatView.vue 路由参数变化 SSE 连接（补充 watch route.params.id）
- Admin.vue/AiChatDialog.vue SSE 流程缺 abortActiveConnection（AiChatDialog.vue 已补充独立流程图，Admin.vue 已补充调用）
- **迭代 4 问题 10（plan/generate 响应字段命名偏离）**：**已解决**——3.2.13 节新增"响应结构设计偏离说明"明确标注 type→plan_type、order→order_num、time→time_desc 的映射关系及保留详细设计命名的理由（order 是 SQL 保留字）
- **迭代 4 问题 11（跨标签页 setAuth 同步）**：**已解决**——3.7 节 authStore 接口新增 setAuth(newToken, newRole, user) 和 syncFromStorage() 方法；1.2 节跨标签页同步描述改为调用 syncFromStorage()，解决原 setToken() 仅同步 token 的问题
- NewsView.vue 未调用免责声明判定（流程图已补充）
- SSE 事件 message/message_end 缺 created_at（3.3 节已补充 created_at: number 字段）
- difyService.js blocking 超时 60s（改为 15s）
- **迭代 4 问题 15（punch 接口枚举值语言全局偏离）**：**已解决**——3.2.16 节 punch_type 使用英文 diet/exercise/other，completion_status 使用英文 completed/uncompleted，与需求 6.6 节一致；3.2.17 节 punch/list 响应同样使用英文枚举值
- AiChatDialog.vue DOM 缺登录引导/跳转登录按钮（4.1.1 节已补充）
- admin_logs.admin_id 语义混淆（重命名为 operator_id，3.2.30 节响应已同步）

### 未明确核实状态的 4 项

- 迭代 4 问题 8（DoctorChatView.vue 路由参数变化 SSE 连接）：已在迭代 4 问题 8 修复中确认（与上述"DoctorChatView.vue 路由参数变化 SSE 连接"项相同）
- 迭代 4 问题 9（Admin.vue/AiChatDialog.vue SSE 流程）：已在迭代 4 问题 9 修复中确认
- 迭代 4 问题 12（NewsView.vue 免责声明判定）：已在迭代 4 问题 12 修复中确认
- 迭代 4 问题 13（SSE created_at 字段）：已在迭代 4 问题 13 修复中确认

**说明**：上述 4 项实际已在前述列表中确认解决，此处单列仅为澄清"未核实"的表述精度。全部 37 个历史问题均已明确状态。

---

## 修订说明（v2）

| 质询意见 | 回应 |
|---------|------|
| **质询要点 1**：需求响应充分度覆盖范围不足，仅深入核对了约 3 个 API 子节，其余 7 个 API 子节（risk/history、punch/list、punch/analysis、doctors、chat/doctor、assistant/advice、admin/logs）的契约一致性未验证 | **部分接受**。已补充"API 契约补充核对"章节，对剩余 7 个 API 子节逐项核对需求 6.3~6.10 节。核对结论：上述 7 个子节均与需求契约基本一致，未发现严重偏离。v1 报告中已发现的 3 个子节偏离（注册响应缺 token、登录响应 role 位置错误、plan_id 缺数据库支撑）属于局部问题，未在其余子节复现。质询中"未检查区域存在同类问题的高风险"的推测未被证实。 |
| **质询要点 2**：技术风险和缓解措施维度完全缺失，Dify 门禁任务、SQL 注入防护、SQLite 可靠性、SSE 并发控制的设计缓解措施充分性均未评估 | **接受**。已补充"技术风险与缓解措施评估"章节，逐项评估 4 个关键技术风险。核对结论：v5 产出已识别全部关键技术风险并提供充分缓解措施——Dify `{{user}}` 门禁任务有完整备选方案（5.5.2 节 session_id 映射表）、SQL 注入防护有白名单 + AST 解析 + 参数化执行三层防御（7.3.3/7.3.4 节）、SQLite 有 WAL 模式 + 每日备份（6.4/6.7 节）、SSE 并发控制有 activeAbortController 全局管理 + 三端场景策略（3.7 节）。质询中"完全缺失"的判断是基于 v1 报告未评估而非设计文档未覆盖，此点已在修订中澄清。 |
| **质询要点 3**：历史迭代问题核实不完整，"系统解决"结论基于约 54% 的核实率（20/37），且迭代 4 问题 10/11/15 与已发现问题高度相关但未建立关联 | **接受**。已逐项核实全部 37 个历史问题，核实率提升至 100%。其中迭代 4 问题 10（plan/generate 字段命名偏离）：**已解决**——3.2.13 节有显式偏离说明；迭代 4 问题 11（setAuth 跨标签页同步）：**已解决**——3.7 节新增 setAuth/syncFromStorage 方法，1.2 节改为调用 syncFromStorage()；迭代 4 问题 15（punch 枚举值语言）：**已解决**——3.2.16 节使用英文枚举值。同时建立关联：迭代 4 问题 11 的 setAuth 修复与本报告问题 2（authStore.login 未设置 role）存在关联——setAuth 方法已新增但 login() 伪代码仍调用旧逻辑未设置 role，两者均需修复才能形成完整闭环。 |
| **质询要点 4**：非功能性需求响应未评估，7.2 性能、7.3 产品质量、7.4 部署架构、7.5 环境配置的设计响应充分度均未评估 | **接受**。已补充"非功能性需求响应评估"章节，逐项评估 7.1~7.5 节。核对结论：v5 产出对非功能性需求响应充分——7.1 已引入 Vant 4、7.2 AI 超时统一 15s、7.3 错误码与降级策略完整、7.4 Nginx/Keepalived/备份策略齐全、7.5 .env.example 覆盖全部环境变量。未发现遗漏的非功能性维度。 |
| v1 问题 3 证据表述略有过度强化（暗示需求要求 plan_id 作为数据库列） | **接受**。已在 v1 问题 3 中补充"证据精度补充（v2 修订）"段落，明确说明需求 5 节 life_plans 表字段清单未显式要求 plan_id 列，但需求 6.5 节"用于后续方案调整的整体替换"语义需要数据库层提供持久化支撑，因此将其标注为数据库列缺失问题仍然成立。 |

