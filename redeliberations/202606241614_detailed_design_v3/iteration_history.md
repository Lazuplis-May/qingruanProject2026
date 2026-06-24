## 迭代第 1 轮

1. **问题描述**：数据库 CHECK 约束及 API 字段中的枚举值定义存在严重偏差（中文值 vs 英文值），详细设计与需求文档中“存储英文小写字符串”的要求不符。
   - 所在位置：第 2.2 节（`users`、`user_risk_info`、`life_plans`、`punch_in` 表的 DDL）、第 2.5 节（数据字典约束）、第 3.2 节（API 接口规格）、第 3.8 节（TypeScript 类型定义）
   - 严重程度：严重
   - 改进建议：将 DDL 中的中文 CHECK 约束修改为需求定义的英文小写枚举值，更新数据字典描述；在 API 接口规格和 TypeScript 类型中调整为英文，或在后端设计明确的中英双向映射转换层。

2. **问题描述**：`user_risk_info` 表设计中缺失关键字段 `diabetes_history`（糖尿病病史），导致前端提交的病史数据无法在数据库中持久化。
   - 所在位置：第 2.1 节（ER图）、第 2.2 节（DDL 语句）、第 2.5 节（数据字典）
   - 严重程度：严重
   - 改进建议：在 DDL、数据字典以及 ER 图中补充 `diabetes_history` 字段定义：`diabetes_history TEXT NOT NULL CHECK(diabetes_history IN ('healthy', 'prediabetes', 'diagnosed'))`，并在落库的逻辑流程中增加对应处理。

3. **问题描述**：`life_plans` 表设计中缺失关键字段 `is_active`（是否活跃），未遵循“逻辑过期”机制，可能破坏用户历史打卡数据的可追溯性。
   - 所在位置：第 2.1 节（ER 图）、第 2.2 节（DDL 语句）、第 2.5 节（数据字典）
   - 严重程度：严重
   - 改进建议：在 `life_plans` 表的 DDL、数据字典和 ER 图中补充 `is_active` 字段定义：`is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1))`；在查询当前方案的接口逻辑中，加入 `is_active = 1` 的过滤条件。

4. **问题描述**：前端路由结构与组件划分不符合需求规范，未将医师咨询（医生列表）与医师咨询（对话）页面组件进行拆分。
   - 所在位置：第 1.4 节（目录树）、第 1.6.1 节（Vue Router 路由映射表）、第 4.3 节（相关流程图）
   - 严重程度：一般
   - 改进建议：将 `Consultation.vue` 拆分为 `Consultation.vue`（医生列表）和 `DoctorChatView.vue`（医生对话）两个组件，更新路由映射表并将对应的路由分别指向这两个页面组件。

5. **问题描述**：个人中心子页面的嵌套路由设计不明确，糖尿病风险预测、打卡记录与分析、健康建议被扁平地定义为独立的一级路由。
   - 所在位置：第 1.6.1 节（路由映射表）、第 4.3 节（组件逻辑流程图）
   - 严重程度：轻微
   - 改进建议：调整路由映射表，明确以树形/嵌套结构表示这三个页面是 `/profile` 的子路由（使用 `children`）；在 `Profile.vue` 模板结构中说明 `<router-view />` 的放置和渲染逻辑。

6. **问题描述**：Dify 智能助手会话管理机制中对 `conversation_id` 的设计与需求不一致，使用通用 Map 替代了独立变量。
   - 所在位置：第 3.7 节（`chatStore` 接口定义）、第 4.3 节（相关 Store 读写流程图）
   - 严重程度：轻微
   - 改进建议：将 `chatStore` 定义恢复为需求规范的格式（即 `doctorConversations` Map + `assistantConversationId` 独立变量），以确保代码实现与业务命名约定一致；或者在设计中显式添加重构说明消除歧义。

7. **问题描述**：缺少具体的数据类型与映射转换机制说明（技术决策遗漏），未定义前后台之间中英枚举值数据的转换归属与实现方案。
   - 所在位置：第 3.6 节（Express 代理层请求参数映射表）、第 5 节（数据需求与落库路径）
   - 严重程度：严重
   - 改进建议：增加“数据字段映射与映射层”的技术决策，在 Express 后端设计专用的转换模块（如 `utils/mapper.ts`），或规范 API 和 TS 接口直接使用英文，由前端在 UI 渲染层映射展示。

8. **问题描述**：前端路由结构与组件划分不符合需求规范，未将健康资讯（列表）与健康资讯（详情）页面组件进行拆分。
   - 所在位置：第 1.4 节（目录树）、第 1.6.1 节（Vue Router 路由表）、第 4.1.5 节（组件描述与流程图）
   - 严重程度：一般
   - 改进建议：在目录结构中，将 `News.vue` 拆分为 `NewsView.vue` (资讯列表) 和 `ArticleDetailView.vue` (资讯详情)，更新路由映射表并将对应的路由分别指向这两个页面组件。

9. **问题描述**：缺少跨浏览器标签页登录态同步机制的设计，在同一浏览器打开多标签页时无法同步登录/登出状态。
   - 所在位置：第 1.2 节（前端架构说明）、第 3.7 节（`authStore` 设计与接口定义说明）
   - 严重程度：一般
   - 改进建议：补充“跨浏览器标签页的登录态同步”设计，在 `App.vue` 挂载时监听 `storage` 事件，当检测到 localStorage 中的 `token` 或 `role` 发生变化时进行状态同步或调用清除方法。

10. **问题描述**：缺少前端 SSE 流式对话连接控制与并发限制机制，路由切换时未注销旧连接，且未限制最大并发长连接数。
    - 所在位置：第 1.2 节（前端架构设计）、第 3.7 节（`chatStore` 接口定义）、第 4.3 节（组件流程图）
    - 严重程度：严重
    - 改进建议：在 `chatStore` 中引入 SSE 连接管理器，使用 `AbortController` 并在组件销毁或路由切换时调用其 `abort()` 方法注销连接，且补充并发连接限制策略限制最大连接数为 1。

11. **问题描述**：特定输入项（`waist` 和 `systolic_bp`）缺少 0 值的有效性验证及特定的错误处理逻辑，无法拦截非法的 0 值传入后台。
    - 所在位置：第 3.2.7 节（接口规格）、第 4.3 节（`Risk.vue` 表单验证逻辑设计）
    - 严重程度：一般
    - 改进建议：在后端接口请求体参数描述中对上述两个字段增加 0 值校验规则，并定义返回特定的 `VALIDATION_ERROR` 错误响应；在前端表单验证中补充 0 值拦截规则。
\n## 迭代第 2 轮

1. **问题描述**：Dify API Key 校验中间件（`difyAuth.js`）在使用 `crypto.timingSafeEqual` 进行 API Key 的时序安全比对时，未提前校验两个 Buffer 长度，若客户端请求携带的 API Key 长度与环境变量中的不一致，将抛出 `TypeError` 异常导致 Node.js 服务端崩溃或返回 500 错误，存在拒绝服务（DoS）隐患。
   - 所在位置：第 7.3.2 节 `difyAuth.js` 中间件行为伪代码（第 5087-5090 行）
   - 严重程度：严重
   - 改进建议：在比对前先检查长度，或者通过 SHA-256 哈希将任意长度的密钥输入转换为固定长度 Buffer 后再使用 `crypto.timingSafeEqual` 进行比对。
2. **问题描述**：前端路由配置文件（`router/index.ts`）存在语法嵌套错误和标点符号混淆，如将路由定义错误写为 `routes: const routes: RouteRecordRaw[] = [`，且在路由数组结束时写成 `];,` 导致 Vite 构建与编译失败。
   - 所在位置：第 1.6.2 节 `router/index.ts` 伪代码（第 434 行、第 506-507 行）
   - 严重程度：严重
   - 改进建议：规范前端路由的定义结构，单独声明 `routes` 数组，然后再传入 `createRouter` 实例化。
3. **问题描述**：数据双向转换层（`mapper.js`）的 `MAPPINGS` 字典中漏配了打卡类型 `punch_type` 的映射（'饮食' ↔ 'diet', '运动' ↔ 'exercise'），导致打卡记录落库时触发 SQLite 的 `CHECK` 约束校验失败，打卡功能失效。
   - 所在位置：第 1.8.1 节转换映射字典定义（第 597-624 行）与第 1.8.3 节控制器拦截策略说明（第 633 行）
   - 严重程度：严重
   - 改进建议：在 `server/utils/mapper.js` 的 `MAPPINGS` 对象中补全 `punch_type` 的双向转换映射。
4. **问题描述**：`POST /api/risk/predict` 接口响应结构与用户需求契约存在偏差，字段定义混淆且缺少 `risk_level_label`，同时将 `matched_diabetes_type` 命名为了 `diabetes_type`，并将 `advice` 拆分为了 `risk_level_detail` 和 `suggestions`。
   - 所在位置：第 3.2.7 节 `POST /api/risk/predict` 响应规范（第 1520-1534 行）与第 3.8.4 节 TypeScript 预测响应接口定义（第 2452-2460 行）
   - 严重程度：一般
   - 改进建议：修改响应结构和 TypeScript 类型定义 `RiskPredictResponse`，完全对齐需求文档（包含 `record_id`、`risk_score`、`risk_level`、`risk_level_label`、`matched_diabetes_type`、`advice` 且 advice 为 Markdown 文本）。
5. **问题描述**：`user_risk_info` 表中的 `pregnancy`（妊娠状态）字段在 SQLite 物理 DDL 和数据字典中定义为 `TEXT` 类型，但在 TypeScript 接口和 Dify 配置中定义为 `boolean` 类型，且漏配转换处理，导致数据一致性与写入风险。
   - 所在位置：第 2.2 节 DDL 语句中 `user_risk_info` 表定义（第 838 行）与第 2.5 节数据字典说明（第 1162 行）
   - 严重程度：一般
   - 改进建议：将 `pregnancy` 的物理类型修改为 `INTEGER` 并增加 `CHECK(pregnancy IN (0, 1) OR pregnancy IS NULL)` 约束，且在后端数据库操作中自动进行 Boolean ↔ Integer (0/1) 的映射转换。
6. **问题描述**：初始数据脚本 `seed.sql` 包含管理员密码哈希的占位符，如果开发者直接通过外部命令行或数据库图形化管理工具导入 `seed.sql`（不经过 `initDatabase()` 的替换逻辑），将导致数据库直接存入占位符，从而造成管理员无法登录。
   - 所在位置：第 2.4 节管理员插入部分（第 952 行）与第 6.4 节数据库初始化逻辑（第 4825-4832 行）
   - 严重程度：轻微
   - 改进建议：在 `seed.sql` 中或设计文档相应章节补充风险提示，提醒开发者避免直接脱离后端环境导入执行 `seed.sql`。
7. **问题描述**：数据库中布尔逻辑字段的表现形式不统一，`is_active` 使用 `INTEGER` 存储，`password_changed` 使用 `TEXT` 存储，`pregnancy` 也使用 `TEXT` 且无约束，这增加了开发和维护时的认知负担，且易引发映射错误。
   - 所在位置：第 2.2 节完整 DDL 语句中相关表定义（第 777, 838, 856 行）
   - 严重程度：轻微
   - 改进建议：在 SQLite 数据库物理建模中，统一将所有布尔属性字段声明为 `INTEGER` 类型，并增加 `CHECK(field IN (0, 1))` 约束。
8. **问题描述**：除 `GET /api/plan/current` 以外，对于其他核心数据驱动接口如 `GET /api/risk/history` 和 `GET /api/punch/list`，文档中均缺乏具体能指导后端研发人员编写的 SQLite 查询设计。
   - 所在位置：第 3.2.8 节与第 3.2.17 节详细设计
   - 严重程度：轻微
   - 改进建议：为这些数据驱动的接口补充对应的 SQLite 查询伪代码（如对分页、条件过滤等进行示范）。

## 迭代第 3 轮

1. **问题描述**：Dify 回调接口 `/api/admin/execute` 中硬编码 `operatorRole` 为 `'user'`，导致管理员的 Text2SQL 功能失效。
   - 所在位置：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5196 行）
   - 严重程度：严重
   - 改进建议：后端在通过 Dify API Key 校验后，应根据 `operatorId` 查询数据库中的用户角色，动态确定 `operatorRole` 的实际值（'admin' 或 'user'），而不能一律硬编码为 'user'。
2. **问题描述**：`/api/admin/execute` 的安全校验函数 `validateRowLevelPermission` 缺失具体实现规范。
   - 所在位置：第 7.3.3 节 `POST /api/admin/execute` 路由处理器核心逻辑伪代码（第 5209 行）
   - 严重程度：严重
   - 改进建议：增设专门的技术规范段落，定义 `validateRowLevelPermission` 的校验策略。例如使用 SQL 解析库解析为抽象语法树（AST）后，检查其 WHERE 子句中是否包含 `user_id = operatorId`；或提供详细的校验伪代码以指导开发。
3. **问题描述**：`life_plans`（生活方案）和 `punch_in`（打卡记录）表的 `plan_type`/`punch_type` 约束存在逻辑矛盾。
   - 所在位置：第 2.2 节完整 DDL 语句中相关表定义（第 857 行与第 885 行）
   - 严重程度：一般
   - 改进建议：统一 `punch_in` 表 and `life_plans` 表的分类维度，将 `punch_in.punch_type` 的 CHECK 约束扩充为 `'diet', 'exercise', 'other'`；或者在文档中明确说明 `'other'` 类型的方案仅供展示，不提供打卡记录生成。
4. **问题描述**：医学免责声明确认弹窗（`showDisclaimer`）仅定义了方法，但在用户交互流程中完全未被实际调用。
   - 所在位置：第 4.4.4 节 `useUI.ts` 声明（第 3792 行）与第 1.6.2 节路由守卫代码块
   - 严重程度：一般
   - 改进建议：在 Vue Router 全局前置守卫（`router/index.ts`）中增加免责拦截逻辑，或在涉及 AI 功能的页面组件挂载时调用 `hasAcceptedDisclaimer()` 进行判定。若未同意，调用 `showDisclaimer()` 弹窗；若同意，将 `'disclaimer_accepted' = 'true'` 写入 `localStorage`。
5. **问题描述**：Dify 风险预测工作流输出结构与后端数据库提取查询存在数据契约不一致。
   - 所在位置：第 5.2.1 节输出结构定义（第 4106 行）与第 3.2.8 节 SQLite 查询设计（第 1579 行与第 1580 行）
   - 严重程度：一般
   - 改进建议：修改 Dify 工作流输出结构，使其直接包含 `risk_level_label` 和 `matched_diabetes_type`；或者在后端接口路由中，对 Dify 返回的原始 JSON 键值进行映射转换后再存入数据库。
6. **问题描述**：`POST /api/punch`（记录打卡）的响应体缺失 `remarks` 字段。
   - 所在位置：第 3.2.16 节 `POST /api/punch` 响应体定义（第 1785-1791 行）
   - 严重程度：轻微
   - 改进建议：在 `POST /api/punch` 成功响应的 `data` 对象中补齐 `remarks` 字段返回。
7. **问题描述**：`AiChatDialog.vue`（AI 智能助手）组件的 DOM 结构缺失医学免责提示元素。
   - 所在位置：第 4.1.1 节 `AiChatDialog.vue` 组件 DOM 树结构（第 2705-2714 行）
   - 严重程度：轻微
   - 改进建议：在 `AiChatDialog.vue` 的对话输入区域上方或消息列表底部，增加渲染免责提示文本的 `<p class="disclaimer-text">` 元素节点。
