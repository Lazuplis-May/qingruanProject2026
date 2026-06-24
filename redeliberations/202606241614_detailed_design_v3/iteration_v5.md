# 再审议判定报告（v5）

## 判定结果

RETRY

## 判定理由

### 一、输入信息确认

- **组件B诊断报告**：`b_v5_diag_v2.md`，识别 10 个质量问题（3 严重 / 5 一般 / 2 轻微）
- **组件B质询报告**：`b_v5_challenge_v2.md`，质询结果为 LOCATED
- **内部循环最大轮次**：12
- **内部循环实际轮次**：2（实际 < 最大，说明提前终止，非循环耗尽）

### 二、质询报告终止原因分析

质询报告结果显示 LOCATED，且实际轮次（2）远小于最大轮次（12），表明组件B在第 2 轮即明确定位到问题并确认诊断结论成立，**不属于"达到最大轮次仍未 LOCATED"的情形**。根据判定标准，"组件B经多轮审议后未能定位到明确问题（达到最大轮次仍未 LOCATED）"方可判 PASS，本情形不满足该条件。

质询报告对诊断报告进行了三维度独立审查：
- **证据充分性**：10 个问题的证据均经独立交叉核实，事实判断准确，附具体行号引用
- **逻辑完整性**：10 个问题之间无逻辑矛盾，改进建议与问题对应可行，严重程度分级合理
- **覆盖完备性**：任务描述要求的三个审查维度（需求响应充分度、事实错误与逻辑矛盾、深度与完整性）均已覆盖，工程实施视角评估、技术风险与缓解措施评估、非功能性需求响应评估均已系统性补充

质询报告完全确认了诊断报告的结论，未对任何问题提出质疑或否定。因此诊断报告所识别的问题成立。

### 三、问题等级判定

诊断报告中识别的问题等级分布如下：

- **严重问题（3 个）**：
  - 问题 1：POST /api/auth/register 响应不返回 JWT Token，与需求 6.1 节直接矛盾，需"用户无需重复登录"但实际要求用户手动登录
  - 问题 2：POST /api/auth/login 响应 role 字段位置与需求 6.1 节不一致，且 authStore.login() 伪代码未设置 role 状态，将导致管理员路由守卫失效（authStore.role 恒为 null，管理员无法访问 /admin）
  - 问题 3：life_plans 表缺少 plan_id（方案组 ID）列，PUT /api/plan/adjust 接收的 plan_id 参数无数据库支撑，成为死参数
- **一般问题（5 个）**：问题 4（pregnancy 转换机制未文档化）、问题 5（punch_type CHECK 含 'other' 与需求 5 节直接矛盾）、问题 6（PunchCreateResponse 类型泛化）、问题 7（User 接口与登录响应 user 对象不一致）、问题 8（authStore 缺 clearMustChangePassword 方法定义但被调用）
- **轻微问题（2 个）**：问题 9（PlanResponse 缺 other_plans 字段）、问题 10（articles tags 字段 JSON 序列化策略未集中文档化）

### 四、判定标准应用

根据判定指令：
- **PASS 条件**：审查报告不含严重或一般等级的问题；或组件B经多轮审议后未能定位到明确问题（达到最大轮次仍未 LOCATED）；或发现的问题均为轻微等级
- **RETRY 条件**：审查报告包含严重或一般等级的问题

本案例中：
1. 诊断报告包含 3 个严重问题和 5 个一般问题，**不满足"不含严重或一般等级问题"的 PASS 条件**
2. 质询报告结果为 LOCATED 且实际轮次（2）小于最大轮次（12），**不满足"达到最大轮次仍未 LOCATED"的 PASS 条件**
3. 诊断报告含严重和一般等级问题，**不满足"问题均为轻微等级"的 PASS 条件**
4. 诊断报告包含严重和一般等级问题，**满足 RETRY 条件**

### 五、综合结论

诊断报告所识别的 3 个严重问题均涉及需求契约的直接违反或核心功能失效：
- 问题 1 直接违反需求 6.1 节"用户无需重复登录"的明确要求
- 问题 2 将导致管理员权限路由守卫失效，管理员无法访问 /admin 页面，属核心功能失效
- 问题 3 导致 PUT /api/plan/adjust 接口的 plan_id 参数成为死参数，API 契约有误导性

5 个一般问题涉及字段类型安全、接口契约一致性、未定义方法被调用等，若不修复将导致前后端联调失败或运行时错误。

质询报告已通过三维度独立审查确认上述问题成立，且实际轮次远小于最大轮次，不构成循环耗尽情形。诊断报告和质询报告共同构成完整的问题证据链。

综上，判定结果为 **RETRY**，组件A需重新运行以修复上述问题。

## 需要解决的问题

- **问题描述**：POST /api/auth/register 响应仅返回 `{user_id, username}`，不含 JWT Token、role、user 对象；`RegisterResponse` 类型定义相应缺失 token/user/role 字段；4.3 节 Login.vue 流程图要求注册成功后切换至登录视图让用户手动登录。与需求 6.1 节"注册成功后直接返回 JWT Token 和用户信息，用户无需重复登录"直接矛盾。
- **所在位置**：3.2.1 节注册响应（第 1378~1388 行）、3.8.2 节 `RegisterResponse` 类型定义（第 2492~2495 行）、4.3 节 Login.vue 流程图（第 3788 行）
- **严重程度**：严重
- **改进建议**：
  1. 将 3.2.1 节注册成功响应修改为与登录响应结构一致（含 token、user 对象、可选 must_change_password）
  2. 将 `RegisterResponse` 类型定义更新为与 `LoginResponse` 一致（或直接复用 `LoginResponse` 类型）
  3. 更新 4.3 节 Login.vue 流程图，注册成功后调用 `authStore.login()` 逻辑（或等效的 setAuth），自动登录并跳转至首页

- **问题描述**：POST /api/auth/login 响应将 `role` 嵌套在 `user` 对象内，与需求 6.1 节定义的 `role` 为顶层字段（与 token、user 平级）不一致；`LoginResponse` TypeScript 类型完全缺少 `role` 字段；1.5.2 节 `authStore.login()` 伪代码只设置了 token 和 user，未设置 role 状态变量，未将 role 写入 localStorage；authStore 状态声明也未声明 role ref 变量。将导致路由守卫 `authStore.role === 'admin'` 判断失效，管理员登录后被重定向至 /home，无法访问 /admin。
- **所在位置**：3.2.2 节登录响应（第 1420~1434 行）、3.8.2 节 `LoginResponse` 类型（第 2486~2490 行）、1.5.2 节 `authStore.login()` 伪代码（第 349~365 行）、3.7 节 AuthActions 接口说明（第 2337 行）、1.5.2 节 authStore 状态声明（第 345~348 行）
- **严重程度**：严重
- **改进建议**：
  1. 明确 role 字段在登录响应中的位置——建议遵循需求 6.1 节将 role 放在顶层（与 token 平级），或在文档中显式说明偏离理由并确保前后端一致
  2. 在 `LoginResponse` 类型中补充 `role: 'user' | 'admin'` 字段（位置与 API 响应一致）
  3. 修正 1.5.2 节 `authStore.login()` 伪代码，从响应中提取 role 并设置到 `role.value` 和 `localStorage.setItem('role', role)`
  4. 在 1.5.2 节 authStore 状态声明中补充 `const role = ref<'user' | 'admin' | null>(localStorage.getItem('role') as 'user' | 'admin' | null)`

- **问题描述**：life_plans 表 DDL 无 plan_id 或 group_id 列，但 API 层返回和接收 plan_id（3.2.13 节响应包含 plan_id，3.2.14 节 PUT /api/plan/adjust 请求体接收 plan_id 参数）。需求 6.5 节明确定义 plan_id 为方案组 ID，"同一批生成的所有方案项共享此 plan_id，用于后续方案调整的整体替换"。当前 PUT /api/plan/adjust 实际通过 is_active 逻辑过期机制实现，plan_id 参数完全未被使用，成为死参数，API 契约有误导性。
- **所在位置**：2.2 节 life_plans DDL（第 884~896 行）、3.2.13 节 POST /api/plan/generate 响应（第 1745 行）、3.2.14 节 PUT /api/plan/adjust 请求体（第 1781 行）、3.8.5 节 PlanResponse/PlanAdjustRequest 类型（第 2643~2652 行）
- **严重程度**：严重
- **改进建议**：
  - 方案 A（推荐）：在 life_plans 表新增 `plan_id INTEGER NOT NULL` 列（同一批生成的方案项共享相同 plan_id 值），并在 DDL、数据字典、ER 图中同步补充。PUT /api/plan/adjust 通过 `UPDATE life_plans SET is_active=0 WHERE user_id=? AND plan_id=?` 精确定位待调整的方案组
  - 方案 B（简化）：若确认"每用户仅保留一套活跃方案"是最终设计决策，则在文档中明确说明 plan_id 的语义（如"plan_id 为首条方案项 id，仅用于前端引用，后端调整时不依赖此参数"），并从 PUT /api/plan/adjust 请求体中移除 plan_id 参数（改为仅接收 feedback），消除死参数

- **问题描述**：v13 修订将 pregnancy 的 DDL 类型改为 INTEGER（存储 0/1），但 API 请求/响应和 TypeScript 类型使用 boolean，v13 同时移除了后端 mapper.js 转换层。1.8.2 节各层枚举值规范表未包含 pregnancy 字段，5.2.1.1 节端到端字段映射契约表也未列出 pregnancy。后端开发者不知道需要将 boolean 转换为 INTEGER(0/1) 写入 SQLite，Dify 工作流输入变量类型为 boolean 但无法直接写入 SQLite INTEGER 列。
- **所在位置**：2.2 节 user_risk_info DDL（第 874 行）、3.2.7 节请求体（第 1547 行）、3.8.4 节 RiskPredictRequest（第 2600 行）、5.2.1 节 Dify 输入变量（第 4349 行）、1.8 节各层枚举值规范表（第 649~655 行）
- **严重程度**：一般
- **改进建议**：
  1. 在 1.8.2 节各层枚举值规范表中补充 pregnancy 行，明确 DDL 层为 INTEGER(0/1)、API/TS 层为 boolean
  2. 在 5.2.1.1 节端到端字段映射契约中补充 pregnancy 的转换说明（Express risk.js 在写入前 `pregnancy ? 1 : 0`，读取后 `row.pregnancy === 1`）
  3. 或在 3.2.7 节请求体注释中补充"后端将 boolean 转换为 INTEGER(0/1) 存储"的说明

- **问题描述**：需求 5 节对 punch_in 表 punch_type 字段的定义明确为 `CHECK(punch_type IN ('diet', 'exercise'))`，但 v5 DDL 扩展为包含 'other'：`CHECK(punch_type IN ('diet', 'exercise', 'other'))`，直接违反需求 5 节的显式定义，且文档中未说明此偏离需求的理由。扩展 'other' 后打卡分析维度的 diet_completion_rate 和 exercise_completion_rate 无法覆盖 'other' 类型打卡数据。
- **所在位置**：2.2 节 punch_in DDL（第 915 行）、2.5 节 punch_in 数据字典（第 1248 行）
- **严重程度**：一般
- **改进建议**：
  - 方案 A（遵循需求）：将 punch_type CHECK 约束恢复为 `IN ('diet', 'exercise')`，在文档中说明 'other' 类型的方案项仅供展示，不支持打卡
  - 方案 B（保留扩展但说明偏离）：若确认扩展 'other' 是合理设计决策，在 2.5 节数据字典中显式标注"此为偏离需求 5 节的设计决策，理由：与 life_plans.plan_type 枚举维度对齐"，并在 3.2.18 节打卡分析响应中补充 `other_completion_rate` 字段

- **问题描述**：`PunchCreateRequest` 中 punch_type 和 completion_status 使用了精确的联合类型，但 `PunchCreateResponse` 中同名字段退化为 string 类型，降低了类型安全性，前端无法利用 TypeScript 类型系统对枚举值进行编译期校验。
- **所在位置**：3.8.6 节 PunchCreateResponse（第 2672~2679 行）
- **严重程度**：一般
- **改进建议**：将 `PunchCreateResponse` 中的 punch_type 改为 `'diet' | 'exercise' | 'other'`，completion_status 改为 `'completed' | 'uncompleted'`，与 `PunchCreateRequest` 保持一致

- **问题描述**：`User` 接口定义包含 created_at 字段，但 3.2.2 节登录响应的 user 对象缺少 created_at 字段（仅含 id/username/role/avatar），3.2.4 节 GET /api/user/profile 响应字段完整。前端若用 User 类型接收登录响应，created_at 字段实际为 undefined，若组件中使用 user.created_at（如 Profile.vue 展示注册时间）将出错。
- **所在位置**：3.8.3 节 User 接口（第 2503~2509 行）、3.2.2 节登录响应 user 对象（第 1426~1431 行）、3.2.4 节 GET /api/user/profile 响应（第 1483~1488 行）
- **严重程度**：一般
- **改进建议**：
  - 方案 A：为登录响应定义单独的 `LoginUser` 类型（仅含 id/username/role/avatar），`User` 类型保留完整字段用于 GET /api/user/profile
  - 方案 B：统一登录响应的 user 对象包含 created_at 字段（后端在登录时联查 users 表的 created_at 列）

- **问题描述**：4.3 节 ChangePassword.vue 流程图中调用了 `authStore.clearMustChangePassword()`，但 3.7 节 AuthActions 接口定义中未声明此方法。前端开发者按流程图实现时，调用未定义的 Store 方法将导致 TypeScript 编译错误或运行时 undefined is not a function 错误。
- **所在位置**：3.7 节 AuthActions 接口（第 2336~2344 行）、4.3 节 ChangePassword.vue 流程图（第 3814 行）
- **严重程度**：一般
- **改进建议**：在 3.7 节 AuthActions 接口中补充 `clearMustChangePassword(): void` 方法定义，说明其职责为将 mustChangePassword 状态置为 false 并同步至 localStorage（或 pinia-plugin-persistedstate 持久化）
