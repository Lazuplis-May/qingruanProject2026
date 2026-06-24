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

## 迭代第 4 轮

1. **问题描述**：`useSSE.ts` 使用原生 fetch API 且完全没有处理 401 响应的逻辑，导致所有 SSE 流式端点（医师对话、AI 助手、管理员对话）在 Token 过期时前端无法感知 401 状态，违反需求 4.10 节的 Token 过期处理契约。
   - 所在位置：第 4.4.2 节 `useSSE.ts`（第 3747-3784 行）
   - 严重程度：严重
   - 改进建议：在 `useSSE.ts` 的 `streamRequest` 函数中，在 `fetch` 调用后立即检查 `response.status === 401`，若为 401 则调用 `authStore.clearAuth()` + 展示 Toast + 保持对话窗口打开 + 标记需要重新登录；在 4.4.2 节伪代码中补充 401 处理分支；在 4.3 节相关流程图中增加"fetch 返回 401 → 触发登录引导"分支。

2. **问题描述**：chatStore 方法命名与需求 4.10 节定义不一致（多了 "Id" 后缀，返回类型 `undefined` vs 需求的 `null`），且完全缺失 `clearDoctorConversation` 和 `clearAssistantConversation` 方法，导致用户无法在 UI 中删除历史会话。
   - 所在位置：第 3.7 节 chatStore 接口定义（第 2321-2334 行）
   - 严重程度：严重
   - 改进建议：将方法名统一为需求规范 `setDoctorConversation` / `getDoctorConversation` / `setAssistantConversation`；新增 `clearDoctorConversation(doctorId: number): void` 和 `clearAssistantConversation(): void` 方法并同步持久化清理 localStorage；同步更新 4.3 节 DoctorChatView.vue 流程图中的"清空对话"按钮逻辑。

3. **问题描述**：`POST /api/risk/predict` 0 值校验错误使用 HTTP 400，但错误码为 `VALIDATION_ERROR`，需求 6.13 节明确定义 `VALIDATION_ERROR` 对应 HTTP 422。错误码与 HTTP 状态码不匹配，前端 Axios 拦截器若按 422 处理参数校验错误将无法正确识别 400 响应。
   - 所在位置：第 3.2.7 节（第 1542-1550 行）
   - 严重程度：严重
   - 改进建议：将 HTTP 状态码从 400 改为 422，与需求 6.13 节和 3.4 节错误码枚举表保持一致。

4. **问题描述**：`GET /api/articles/:id` 响应缺少 `is_collected` 字段，与需求 6.7 节直接矛盾。ArticleDetailView.vue 流程图改为"并行请求 GET /api/articles/collections 判断收藏状态"，与需求"避免额外的收藏状态查询请求"的设计意图直接矛盾。
   - 所在位置：第 3.2.20 节（第 1928-1945 行）
   - 严重程度：严重
   - 改进建议：在 3.2.20 节响应 JSON 中新增 `is_collected: boolean` 字段，由 Express 端点同步联查 `article_collections` 表判断收藏状态；同步更新 3.2.21 节响应；更新 ArticleDetailView.vue 流程图，删除"并行请求 GET /api/articles/collections"分支，改为直接读取响应中的 `is_collected` 字段。

5. **问题描述**：`GET /api/articles` 列表响应缺少 `tags`/`summary` 字段，字段名与需求 6.7 节不一致（需求用 `publish_time`/`read_count`，详细设计用 `created_at`/`views`），导致前端文章列表卡片无法展示标签和摘要，TypeScript 类型定义与需求规范不一致。
   - 所在位置：第 3.2.19 节（第 1907-1925 行）
   - 严重程度：严重
   - 改进建议：在 3.2.19 节响应 JSON 中补充 `tags: string[]` 和 `summary: string` 字段；评估是否统一字段名，若保留 `created_at`/`views` 命名需在文档中明确说明映射关系并确认需求方接受；同步更新 3.8.3 节 `Article` TypeScript 接口；同步更新 articles 表 DDL 补充 `tags` 和 `summary` 列。

6. **问题描述**：Dify Agent 工具定义与需求 6.11 节工具清单严重不符——需求定义了 8+5 个语义化专用工具（含细粒度权限约束），详细设计简化为单一 `execute_SQL` 工具，丢失了细粒度权限约束和 `knowledge_search` 知识库检索能力（Dify 知识库检索不能通过 SQL 实现）。
   - 所在位置：第 5.2.5 节（第 4352-4364 行）和第 5.2.6 节（第 4400 行）
   - 严重程度：严重
   - 改进建议：按需求 6.11 节定义补充 8+5 个专用工具的完整定义（工具名、回调 URL、请求体模板、参数说明、权限约束）；评估是否保留 `execute_SQL` 作为兜底工具；若保留单一 `execute_SQL` 设计作为架构决策偏离，需明确说明理由并补充 `knowledge_search` 工具的独立定义；在 7.3.4 节 `validateRowLevelPermission` 补充 `life_advice` 表写入约束规则。

7. **问题描述**：未采纳需求 8.1 节推荐的 Vant 4 移动端 UI 组件库，导致移动端特有交互组件（Tabbar、ActionSheet、DatetimePicker、PullRefresh 等）需从零开发，CSS 变量体系未与 Vant 4 主题变量建立映射，违反需求 7.1 节要求，前端 package.json 未包含 Vant 4 依赖。
   - 所在位置：第 1.3 节技术选型表（第 110-138 行）
   - 严重程度：严重
   - 改进建议：在 1.3 节技术选型表新增 Vant 4 条目；在 6.3.4 节前端 package.json dependencies 中新增 `"vant": "^4.9.0"`；在 4.5.1 节 CSS 变量定义后补充 Vant 4 主题变量映射表；评估 TabBar.vue、日期筛选器、Toast、Dialog 等组件是否改用 Vant 4 组件替代自研实现。

8. **问题描述**：DoctorChatView.vue 未处理"切换至其他医生对话"的 SSE 连接关闭场景——Vue Router 路由参数变化时同一组件会被复用，onUnmounted 不会触发，用户从医生 A 对话直接跳转到医生 B 对话时原 SSE 连接不会被关闭，违反需求 4.2 节"同时活跃的 SSE 连接数上限为 1"约束。
   - 所在位置：第 4.3 节 DoctorChatView.vue 流程图（第 3312-3336 行）
   - 严重程度：一般
   - 改进建议：在 DoctorChatView.vue 中使用 `watch(() => route.params.id, ...)` 监听路由参数变化并调用 `chatStore.abortActiveConnection()`；或使用 `beforeRouteUpdate` 守卫；在 4.3 节流程图中新增"路由参数变化 → abortActiveConnection → 重新初始化"分支。

9. **问题描述**：Admin.vue 和 AiChatDialog.vue 的 SSE 流程未调用 `abortActiveConnection`/`registerAbortController`，AiChatDialog.vue 缺少独立流程图，三个 SSE 端点的并发控制策略未说明。
   - 所在位置：第 4.3 节 Admin.vue 流程图（第 3571-3595 行）和 AiChatDialog.vue（无独立流程图）
   - 严重程度：一般
   - 改进建议：在 Admin.vue 流程图中补充 `registerAbortController` 和 `abortActiveConnection` 调用；为 AiChatDialog.vue 补充独立的 Mermaid 流程图，明确弹窗关闭时调用 `abortActiveConnection`；在 3.7 节 chatStore 中明确并发控制策略。

10. **问题描述**：`POST /api/plan/generate` 响应结构与需求 6.5 节不一致——需求返回单一 `items` 数组，详细设计拆分为 `diet_plans` + `exercise_plans`；字段名不一致（需求用 `type`/`order`/`time`，详细设计用 `plan_type`/`order_num`/`time_desc`），导致前端 TypeScript 类型定义与需求契约不一致。
    - 所在位置：第 3.2.13 节（第 1727-1755 行）
    - 严重程度：一般
    - 改进建议：评估分组结构的优劣（分组结构对前端渲染更友好可作为合理偏离保留）；统一字段命名（建议保留详细设计命名以避免 SQL 保留字冲突），但在文档中明确说明与需求字段名的映射关系；life_plans 表字段名也应统一并说明偏离。

11. **问题描述**：跨浏览器标签页登录态同步使用 `setToken()` 而非 `setAuth()`，未同步更新 role 和 userInfo，导致用户在标签页 A 切换账号时标签页 B 的 role 和 userInfo 仍是原用户数据，造成 UI 显示与实际 token 状态错乱。
    - 所在位置：第 1.2 节（第 108 行）
    - 严重程度：一般
    - 改进建议：修改 1.2 节描述为 token 变化时调用 `authStore.fetchProfile()` 重新获取用户信息，或新增 `syncFromStorage()` 方法从 localStorage 同步恢复三个字段；在 3.7 节 authStore 接口中补充需求 4.10 节定义的 `setAuth(newToken, newRole, user)` 方法。

12. **问题描述**：NewsView.vue 流程图在"点击生成健康资讯"按钮时直接发起 AI 生成请求，没有调用 `hasAcceptedDisclaimer()` 判定，违反需求 4.11 节"资讯生成"是 AI 功能入口的要求。
    - 所在位置：第 4.3 节 NewsView.vue 流程图（第 3415-3424 行）
    - 严重程度：一般
    - 改进建议：在 NewsView.vue 流程图的"点击生成健康资讯"分支前增加免责声明判定节点（hasAcceptedDisclaimer? → 否则 showDisclaimer → 用户同意后写入 localStorage → 发起请求）；在 4.4.4 节 useUI.ts 免责声明函数调用点说明中补充 NewsView.vue 的调用点。

13. **问题描述**：SSE 事件 `message`/`message_end` 缺少 `created_at` 字段，与需求 6.9 节定义不一致，3.8.7 节 `SSEMessageEvent` 和 `SSEMessageEndEvent` TypeScript 接口也缺少该字段，导致前端无法获取消息创建时间戳。
    - 所在位置：第 3.3 节 SSE 流事件完整格式定义（第 2203-2215 行）
    - 严重程度：一般
    - 改进建议：在 3.3 节 `message` 和 `message_end` 事件 data 字段结构中补充 `created_at: number` 字段；在 3.8.7 节相关 TypeScript 接口中新增 `created_at: number` 成员；在 4.3 节对话流程图中明确前端应将 `created_at` 渲染为消息时间戳。

14. **问题描述**：`difyService.js` blocking 模式读取超时 60s 与需求 7.3 节"所有 AI 接口超时阈值统一为 15 秒"不一致，导致风险预测、方案生成、文章生成等 AI 操作可能等待 60 秒才超时。
    - 所在位置：第 6.3.5 节 difyService.js 行为规格（第 4987-4990 行）
    - 严重程度：一般
    - 改进建议：将 blocking 模式读取超时从 60s 调整为 15s 与需求 7.3 节统一；或在文档中明确说明偏离理由（如 Dify 工作流执行时间可能超过 15 秒）并补充前端加载状态提示覆盖整个超时窗口。

15. **问题描述**：`POST /api/punch` 请求体使用中文枚举值（"饮食"/"已完成"），与需求 6.6 节英文枚举值（"diet"/"completed"）不一致；同样问题存在于 `POST /api/risk/predict`、`RiskPredictRequest` 类型、`LifePlan.plan_type` 类型等，是全局性设计决策偏离。
    - 所在位置：第 3.2.16 节（第 1800-1808 行）
    - 严重程度：一般
    - 改进建议：评估两种方案——方案 A（遵循需求使用英文枚举值，前端 UI 层自行映射展示，无需 mapper.js）；方案 B（保留当前设计，在 1.8 节明确标注为设计决策偏离需求 6.3/6.6 节规范并说明理由）；选择后统一执行并同步更新相关 TypeScript 类型定义。

16. **问题描述**：`AiChatDialog.vue` DOM 结构缺少明确的"登录引导提示"和"跳转登录页按钮"元素，`welcome-tips` 区域用途不明确（混淆欢迎语和登录引导），缺少独立的"跳转至登录页"按钮元素，违反需求 4.8 节要求。
    - 所在位置：第 4.1.1 节 AiChatDialog.vue DOM 树（第 2736-2746 行）
    - 严重程度：一般
    - 改进建议：将 `welcome-tips` 区域拆分为两个独立子区域（已登录用户的欢迎语 + 未登录用户的登录引导含跳转登录按钮）；通过 `v-if="!authStore.token"` 和 `v-else` 控制显隐；在流程图中补充未登录用户点击 FAB 的处理分支。

17. **问题描述**：`admin_logs` 表字段名 `admin_id` 用于记录普通用户 user_id 产生语义混淆——普通用户通过 AI 助手触发 Text2SQL 操作时其 user_id 也会写入 `admin_id` 字段，字段名暗示是管理员 ID 但实际可能存储普通用户 ID。
    - 所在位置：第 2.2 节 admin_logs DDL（第 918-926 行）和第 2.5 节数据字典（第 1243-1252 行）
    - 严重程度：一般
    - 改进建议：将 `admin_id` 字段重命名为 `operator_id`（操作者 ID）消除语义混淆；同步更新 DDL、数据字典、ER 图、7.3.3 节伪代码、3.2.30 节响应字段名；在数据字典中明确说明通过 `operation_type` 字段区分管理员操作和普通用户 Text2SQL 操作。

## 迭代第 5 轮

1. **问题描述**：POST /api/auth/register 响应仅返回 `{user_id, username}`，不含 JWT Token、role、user 对象；`RegisterResponse` 类型定义相应缺失 token/user/role 字段；4.3 节 Login.vue 流程图要求注册成功后切换至登录视图让用户手动登录。与需求 6.1 节"注册成功后直接返回 JWT Token 和用户信息，用户无需重复登录"直接矛盾。
   - 所在位置：3.2.1 节注册响应（第 1378~1388 行）、3.8.2 节 `RegisterResponse` 类型定义（第 2492~2495 行）、4.3 节 Login.vue 流程图（第 3788 行）
   - 严重程度：严重
   - 改进建议：
     1. 将 3.2.1 节注册成功响应修改为与登录响应结构一致（含 token、user 对象、可选 must_change_password）
     2. 将 `RegisterResponse` 类型定义更新为与 `LoginResponse` 一致（或直接复用 `LoginResponse` 类型）
     3. 更新 4.3 节 Login.vue 流程图，注册成功后调用 `authStore.login()` 逻辑（或等效的 setAuth），自动登录并跳转至首页

2. **问题描述**：POST /api/auth/login 响应将 `role` 嵌套在 `user` 对象内，与需求 6.1 节定义的 `role` 为顶层字段（与 token、user 平级）不一致；`LoginResponse` TypeScript 类型完全缺少 `role` 字段；1.5.2 节 `authStore.login()` 伪代码只设置了 token 和 user，未设置 role 状态变量，未将 role 写入 localStorage；authStore 状态声明也未声明 role ref 变量。将导致路由守卫 `authStore.role === 'admin'` 判断失效，管理员登录后被重定向至 /home，无法访问 /admin。
   - 所在位置：3.2.2 节登录响应（第 1420~1434 行）、3.8.2 节 `LoginResponse` 类型（第 2486~2490 行）、1.5.2 节 `authStore.login()` 伪代码（第 349~365 行）、3.7 节 AuthActions 接口说明（第 2337 行）、1.5.2 节 authStore 状态声明（第 345~348 行）
   - 严重程度：严重
   - 改进建议：
     1. 明确 role 字段在登录响应中的位置——建议遵循需求 6.1 节将 role 放在顶层（与 token 平级），或在文档中显式说明偏离理由并确保前后端一致
     2. 在 `LoginResponse` 类型中补充 `role: 'user' | 'admin'` 字段（位置与 API 响应一致）
     3. 修正 1.5.2 节 `authStore.login()` 伪代码，从响应中提取 role 并设置到 `role.value` 和 `localStorage.setItem('role', role)`
     4. 在 1.5.2 节 authStore 状态声明中补充 `const role = ref<'user' | 'admin' | null>(localStorage.getItem('role') as 'user' | 'admin' | null)`

3. **问题描述**：life_plans 表 DDL 无 plan_id 或 group_id 列，但 API 层返回和接收 plan_id（3.2.13 节响应包含 plan_id，3.2.14 节 PUT /api/plan/adjust 请求体接收 plan_id 参数）。需求 6.5 节明确定义 plan_id 为方案组 ID，"同一批生成的所有方案项共享此 plan_id，用于后续方案调整的整体替换"。当前 PUT /api/plan/adjust 实际通过 is_active 逻辑过期机制实现，plan_id 参数完全未被使用，成为死参数，API 契约有误导性。
   - 所在位置：2.2 节 life_plans DDL（第 884~896 行）、3.2.13 节 POST /api/plan/generate 响应（第 1745 行）、3.2.14 节 PUT /api/plan/adjust 请求体（第 1781 行）、3.8.5 节 PlanResponse/PlanAdjustRequest 类型（第 2643~2652 行）
   - 严重程度：严重
   - 改进建议：
     - 方案 A（推荐）：在 life_plans 表新增 `plan_id INTEGER NOT NULL` 列（同一批生成的方案项共享相同 plan_id 值），并在 DDL、数据字典、ER 图中同步补充。PUT /api/plan/adjust 通过 `UPDATE life_plans SET is_active=0 WHERE user_id=? AND plan_id=?` 精确定位待调整的方案组
     - 方案 B（简化）：若确认"每用户仅保留一套活跃方案"是最终设计决策，则在文档中明确说明 plan_id 的语义（如"plan_id 为首条方案项 id，仅用于前端引用，后端调整时不依赖此参数"），并从 PUT /api/plan/adjust 请求体中移除 plan_id 参数（改为仅接收 feedback），消除死参数

4. **问题描述**：v13 修订将 pregnancy 的 DDL 类型改为 INTEGER（存储 0/1），但 API 请求/响应和 TypeScript 类型使用 boolean，v13 同时移除了后端 mapper.js 转换层。1.8.2 节各层枚举值规范表未包含 pregnancy 字段，5.2.1.1 节端到端字段映射契约表也未列出 pregnancy。后端开发者不知道需要将 boolean 转换为 INTEGER(0/1) 写入 SQLite，Dify 工作流输入变量类型为 boolean 但无法直接写入 SQLite INTEGER 列。
   - 所在位置：2.2 节 user_risk_info DDL（第 874 行）、3.2.7 节请求体（第 1547 行）、3.8.4 节 RiskPredictRequest（第 2600 行）、5.2.1 节 Dify 输入变量（第 4349 行）、1.8 节各层枚举值规范表（第 649~655 行）
   - 严重程度：一般
   - 改进建议：
     1. 在 1.8.2 节各层枚举值规范表中补充 pregnancy 行，明确 DDL 层为 INTEGER(0/1)、API/TS 层为 boolean
     2. 在 5.2.1.1 节端到端字段映射契约中补充 pregnancy 的转换说明（Express risk.js 在写入前 `pregnancy ? 1 : 0`，读取后 `row.pregnancy === 1`）
     3. 或在 3.2.7 节请求体注释中补充"后端将 boolean 转换为 INTEGER(0/1) 存储"的说明

5. **问题描述**：需求 5 节对 punch_in 表 punch_type 字段的定义明确为 `CHECK(punch_type IN ('diet', 'exercise'))`，但 v5 DDL 扩展为包含 'other'：`CHECK(punch_type IN ('diet', 'exercise', 'other'))`，直接违反需求 5 节的显式定义，且文档中未说明此偏离需求的理由。扩展 'other' 后打卡分析维度的 diet_completion_rate 和 exercise_completion_rate 无法覆盖 'other' 类型打卡数据。
   - 所在位置：2.2 节 punch_in DDL（第 915 行）、2.5 节 punch_in 数据字典（第 1248 行）
   - 严重程度：一般
   - 改进建议：
     - 方案 A（遵循需求）：将 punch_type CHECK 约束恢复为 `IN ('diet', 'exercise')`，在文档中说明 'other' 类型的方案项仅供展示，不支持打卡
     - 方案 B（保留扩展但说明偏离）：若确认扩展 'other' 是合理设计决策，在 2.5 节数据字典中显式标注"此为偏离需求 5 节的设计决策，理由：与 life_plans.plan_type 枚举维度对齐"，并在 3.2.18 节打卡分析响应中补充 `other_completion_rate` 字段

6. **问题描述**：`PunchCreateRequest` 中 punch_type 和 completion_status 使用了精确的联合类型，但 `PunchCreateResponse` 中同名字段退化为 string 类型，降低了类型安全性，前端无法利用 TypeScript 类型系统对枚举值进行编译期校验。
   - 所在位置：3.8.6 节 PunchCreateResponse（第 2672~2679 行）
   - 严重程度：一般
   - 改进建议：将 `PunchCreateResponse` 中的 punch_type 改为 `'diet' | 'exercise' | 'other'`，completion_status 改为 `'completed' | 'uncompleted'`，与 `PunchCreateRequest` 保持一致

7. **问题描述**：`User` 接口定义包含 created_at 字段，但 3.2.2 节登录响应的 user 对象缺少 created_at 字段（仅含 id/username/role/avatar），3.2.4 节 GET /api/user/profile 响应字段完整。前端若用 User 类型接收登录响应，created_at 字段实际为 undefined，若组件中使用 user.created_at（如 Profile.vue 展示注册时间）将出错。
   - 所在位置：3.8.3 节 User 接口（第 2503~2509 行）、3.2.2 节登录响应 user 对象（第 1426~1431 行）、3.2.4 节 GET /api/user/profile 响应（第 1483~1488 行）
   - 严重程度：一般
   - 改进建议：
     - 方案 A：为登录响应定义单独的 `LoginUser` 类型（仅含 id/username/role/avatar），`User` 类型保留完整字段用于 GET /api/user/profile
     - 方案 B：统一登录响应的 user 对象包含 created_at 字段（后端在登录时联查 users 表的 created_at 列）

8. **问题描述**：4.3 节 ChangePassword.vue 流程图中调用了 `authStore.clearMustChangePassword()`，但 3.7 节 AuthActions 接口定义中未声明此方法。前端开发者按流程图实现时，调用未定义的 Store 方法将导致 TypeScript 编译错误或运行时 undefined is not a function 错误。
   - 所在位置：3.7 节 AuthActions 接口（第 2336~2344 行）、4.3 节 ChangePassword.vue 流程图（第 3814 行）
   - 严重程度：一般
   - 改进建议：在 3.7 节 AuthActions 接口中补充 `clearMustChangePassword(): void` 方法定义，说明其职责为将 mustChangePassword 状态置为 false 并同步至 localStorage（或 pinia-plugin-persistedstate 持久化）

## 迭代第 6 轮

1. **问题描述**：difyService.js blocking 模式自动重试直接违反需求 7.3 节"不自动重试"原则。第 6.3.5 节 blocking 模式（用于 POST /api/risk/predict、POST /api/plan/generate、POST /api/articles/generate）的重试次数标注为 1，重试间隔 2s，与需求 7.3 节"不自动重试，避免重复提交请求加重服务端压力"直接矛盾。更严重的是这三个端点均有数据库写入副作用，自动重试可能导致 user_risk_info、life_plans、articles 表写入重复记录，且 life_plans 的 plan_id 生成机制未明确，is_active 过期机制无法正确清理重复方案。
   - 所在位置：第 6.3.5 节"超时与重试策略"表格（约第 5341-5344 行）
   - 严重程度：严重
   - 改进建议：将 blocking 模式重试次数从 1 改为 0，与需求 7.3 节对齐；在表格下方补充设计依据说明，明确 blocking 模式不自动重试是因为三个端点均有数据库写入副作用；若项目组评估后确认需要重试，需明确说明此为对需求 7.3 节的偏离决策，并补充幂等性保护机制（如基于 user_id + 请求时间窗口的去重）。

2. **问题描述**：`/api/admin/execute` 路由处理器未实现 `tool_name` 分发逻辑，与 5.2.5/5.2.6 节工具定义直接矛盾。5.2.5 节明确指出"由 Express 端点根据 tool_name 参数分发至对应的参数化查询处理器"，但 3.2.29 节请求体定义只有 sql/user_id/api_key 三个字段没有 tool_name，7.3.3 节路由处理器伪代码仅解构 sql 字段没有 tool_name 分发逻辑。按当前实现，专用工具回调携带的 tool_name 会被忽略，请求体中可能没有 sql 字段，导致 `db.prepare(undefined)` 抛出运行时错误。
   - 所在位置：3.2.29 节 POST /api/admin/execute 请求体（约第 2200-2216 行）、7.3.3 节路由处理器伪代码（约第 5606-5666 行），与 5.2.5 节（约第 4664-4684 行）、5.2.6 节（约第 4721-4737 行）工具定义矛盾
   - 严重程度：严重
   - 改进建议：在 3.2.29 节请求体定义中补充 tool_name 字段（可选，专用工具回调时必填），说明 tool_name 与 sql 字段的互斥关系；在 7.3.3 节路由处理器伪代码中补充 tool_name 分发分支，若 tool_name 存在则路由至对应参数化查询处理器，若不存在但有 sql 字段走 execute_SQL 兜底路径；补充专用工具参数化查询处理器的伪代码示例（至少 1-2 个，如 query_user_profile）。

3. **问题描述**：POST /api/plan/generate 服务端处理流程未说明旧方案逻辑过期和 plan_id 生成机制，存在多套活跃方案风险。3.2.14 节 PUT /api/plan/adjust 详细说明了 plan_id 处理流程，但 3.2.13 节 POST /api/plan/generate 缺少对应说明，关键缺口包括：旧方案逻辑过期未说明、plan_id 生成时机未明确（需先 INSERT 才能获取 AUTOINCREMENT id 再 UPDATE）、DDL 中 is_active 默认为 1 没有约束防止同一 user_id 存在多个 is_active=1 的方案组。
   - 所在位置：3.2.13 节 POST /api/plan/generate（约第 1771-1826 行）缺少服务端处理流程段落；3.2.15 节 GET /api/plan/current 的 SQLite 查询（约第 1866 行）未按 plan_id 分组；2.2 节 life_plans DDL（约第 918-931 行）缺少防止多套活跃方案的约束
   - 严重程度：一般
   - 改进建议：在 3.2.13 节补充"服务端处理流程"段落，明确说明调用 Dify 工作流前先将当前活跃方案逻辑过期、plan_id 生成策略（推荐使用应用层自增序列或 `SELECT COALESCE(MAX(plan_id), 0) + 1 FROM life_plans WHERE user_id=?`）、新方案项写入时 plan_id 统一填入新生成的值；评估 3.2.15 节查询是否需限制为最新一套活跃方案；在 2.5 节 life_plans 数据字典中补充业务约束说明。

4. **问题描述**：punch_in.plan_id 在 DDL 中为 DEFAULT NULL，但需求 6.6 节 API 契约要求必填，存在契约矛盾。需求 6.6 节明确标注 plan_id 为"必填"，但详细设计 2.2 节 punch_in DDL 中为 `plan_id INTEGER DEFAULT NULL` 允许为空，文档未说明此偏离理由，也未说明何种场景下 plan_id 可以为 NULL。
   - 所在位置：2.2 节 punch_in DDL（约第 945-957 行）；与需求 6.6 节、3.2.16 节 POST /api/punch 请求体（约第 1883 行）必填标注矛盾
   - 严重程度：一般
   - 改进建议：方案 A（遵循需求，建议）将 DDL 改为 `plan_id INTEGER NOT NULL`，保留 `ON DELETE SET NULL` 外键约束，并在文档中说明 NOT NULL 约束与 ON DELETE SET NULL 的协作机制；方案 B（说明偏离理由）保留 DEFAULT NULL，但在 2.5 节 punch_in 数据字典中明确说明 plan_id 在 API 层必填、DDL 层允许 NULL 是为了支持 ON DELETE SET NULL 外键约束和管理员 Text2SQL 场景。

5. **问题描述**：Dify 工作流输出解析失败的处理策略仅 life-plan-generator 有定义，diabetes-risk-prediction 和 health-article-generator 缺失。5.2.2 节 life-plan-generator 有完整的三层降级解析策略（JSON 优先 → 正则提取降级 → LLM 二次调用降级）和解析失败错误响应 PLAN_PARSE_ERROR，但 5.2.1 节 diabetes-risk-prediction 和 5.2.3 节 health-article-generator 均未说明输出格式和解析失败处理策略。三个工作流都属于"AI 内容生成持久化路径"，应有一致的解析失败处理框架。
   - 所在位置：5.2.1 节 diabetes-risk-prediction（约第 4392-4455 行）、5.2.3 节 health-article-generator（约第 4541-4577 行），缺少输出格式和解析策略；与 5.2.2 节 life-plan-generator（约第 4530-4537 行）的完整定义对比
   - 严重程度：一般
   - 改进建议：在 5.2.1 节补充输出格式说明（JSON 对象）和解析失败处理策略（如 RISK_PARSE_ERROR 错误响应）；在 5.2.3 节补充输出格式说明（JSON 对象含 title/cover/content/category/tags/summary 字段）和解析失败处理策略（如 ARTICLE_PARSE_ERROR）；考虑在 6.3.5 节 difyService.js 行为规格中补充统一的"工作流输出解析框架"段落，定义通用三层降级策略模板。

6. **问题描述**：POST /api/articles/generate 和 POST /api/plan/generate 缺少幂等性保护，用户重复提交可能导致重复生成。两个端点均为有数据库写入副作用的 AI 生成端点，用户快速双击、网络重试、前端未正确禁用提交按钮时可能触发多次请求，导致 articles 表写入重复文章、life_plans 表写入多套方案。文档仅依赖前端"按钮 loading 态"和组件级 isSubmitting 状态防重复，缺少服务端幂等性保护。
   - 所在位置：3.2.13 节 POST /api/plan/generate（约第 1771 行）、3.2.21 节 POST /api/articles/generate（约第 2038 行）、4.3 节 LifePlan.vue 流程图（约第 3560 行）和 NewsView.vue 流程图（约第 3575 行）
   - 严重程度：一般
   - 改进建议：在 3.2.13 节和 3.2.21 节服务端处理流程中补充幂等性检查，如同一 user_id 在 N 秒内（如 30s）已有生成请求在进行中或刚完成则拒绝重复请求并返回 409 CONFLICT；或采用客户端幂等键方案，前端生成唯一 requestId，服务端基于 requestId + user_id 去重；在 3.4 节错误码枚举表中补充 CONFLICT 错误码的触发场景说明。

7. **问题描述**：admin_logs 日志写入与 SQL 执行的事务一致性未说明，存在审计日志丢失风险。7.3.3 节路由处理器伪代码中 SQL 执行和 insertAdminLog 是两个独立的数据库操作，若 SQL 执行成功但 insertAdminLog 失败（如 operation_content 字段过长触发 SQLite 错误、或进程在两次操作间崩溃），将导致 SQL 操作已生效但审计日志丢失，违反需求 4.9 节"管理员的所有数据库修改操作均可追溯...操作日志不可删除"的硬性要求。better-sqlite3 是同步驱动支持事务，但文档未说明是否将两者包裹在同一事务中。
   - 所在位置：7.3.3 节路由处理器伪代码（约第 5654-5665 行）
   - 严重程度：一般
   - 改进建议：在 7.3.3 节伪代码中将 SQL 执行和 insertAdminLog 包裹在 better-sqlite3 事务中（`db.transaction(() => { ... })()`），事务整体失败时回滚并返回 500 INTERNAL_ERROR；补充事务失败时的错误处理说明；明确 SELECT 操作是否需要记录日志。

8. **问题描述**：doctor_information 表 chat_token 字段明文存储 Dify API Key，缺少加密策略说明。chat_token 字段存储 Dify 聊天助手的 API Secret（格式 `app-XXX`），属于敏感凭证，2.2 节 DDL 中为 `chat_token TEXT NOT NULL` 明文存储，7.6 节 SQL 注入防护表提到"SQLite 仅 Express 进程访问"但未说明 chat_token 加密策略。若数据库文件泄露（如备份文件未加密传输、服务器被入侵），所有医生的 Dify API Key 将直接暴露。
   - 所在位置：2.2 节 doctor_information DDL（约第 858 行）、2.5 节 doctor_information 数据字典（约第 1192 行）、7.5 节 XSS 防御和 7.6 节 SQL 注入防护均未涉及 chat_token 加密
   - 严重程度：一般
   - 改进建议：方案 A（推荐，加密存储）chat_token 列改为存储 AES-256-GCM 加密后的密文，Express 读取后用 JWT_SECRET 派生密钥解密使用，在 2.5 节数据字典说明加密策略，在 7 节安全设计中补充"敏感字段加密"段落；方案 B（说明风险并接受）在 2.5 节 chat_token 字段说明中补充风险提示"chat_token 为明文存储，依赖 SQLite 文件级访问控制保障安全，生产环境应考虑字段级加密"；无论哪种方案，应在 6.7 节备份策略中补充说明备份文件包含 chat_token 敏感数据，备份存储需访问控制。

9. **问题描述**：POST /api/auth/logout 端点设计不完整，未说明登出时需清理的前端状态。3.2.3 节仅有简单的请求体和响应体，未说明登出时需清理的前端状态，4.3 节 Profile.vue 流程图仅调用 authStore.logout() 清除 Token + user info，但以下状态未说明如何处理：chatStore 中的所有 conversation_id、活跃的 SSE 连接、riskFormStore 中的表单数据、sessionStorage 中的页面级缓存。
   - 所在位置：3.2.3 节 POST /api/auth/logout（约第 1513-1523 行）、4.3 节 Profile.vue 流程图登出分支（约第 3698-3702 行）、1.5.2 节 authStore.logout() 伪代码（约第 367-375 行）仅清理 token/role/user
   - 严重程度：一般
   - 改进建议：在 3.2.3 节补充"前端登出完整流程"段落，明确登出时按顺序清理（中止活跃 SSE 连接、清理所有会话、清除表单数据、清除 token/role/user 并跳转首页、可选清理 sessionStorage 页面级缓存）；在 3.7 节 chatStore 接口中评估是否需要新增 clearAllConversations() 方法；在 4.3 节 Profile.vue 流程图的登出分支中补充上述清理步骤。


## 迭代第 7 轮

1. **问题描述**：7.3.3节 `dispatchParameterizedQuery` 函数与5.2.5/5.2.6节工具定义严重不一致，导致专用工具回调全部失败（存在工具名不匹配、缺失工具实现、多余工具实现的问题）。
   - 所在位置：7.3.3 节 `dispatchParameterizedQuery` 函数、5.2.5 节、5.2.6 节、3.2.29 节
   - 严重程度：严重
   - 改进建议：将 dispatch 函数的 case 列表与 5.2.5/5.2.6 节工具定义表逐一对齐，补充缺失的实现，清理或定义多余的工具，同步修正 3.2.29 节。
2. **问题描述**：8.1节注册验收标准与3.2.1节注册响应契约不一致。
   - 所在位置：8.1 节 API 端点验收标准表
   - 严重程度：一般
   - 改进建议：将 8.1 节注册成功验收项的量化标准更新为包含 token、role、user 等的实际响应结构。
3. **问题描述**：1.2节登录态同步描述仍使用 `setToken()`，与 v14 修订后的 `login()` 逻辑不一致。
   - 所在位置：1.2 节 Vue3 SPA 跨组件通信机制第 1 项
   - 严重程度：一般
   - 改进建议：将描述修改为调用 `authStore.login()`，与 1.5.2 节保持一致。
4. **问题描述**：3.7节 `AuthState.user` 类型为 `User`，但 `login()` 实际赋值为 `LoginUser`，导致类型不匹配。
   - 所在位置：3.7 节 AuthState 接口和 AuthActions.setAuth 签名、3.8.8 节 AuthState、1.5.2 节 login() 伪代码
   - 严重程度：一般
   - 改进建议：将 `AuthState.user` 类型及相关参数修改为 `LoginUser | null`。
5. **问题描述**：3.7节 ChatActions 中 `clearChat()` 与 `clearAllConversations()` 职责重叠未区分。
   - 所在位置：3.7 节 ChatActions 接口
   - 严重程度：轻微
   - 改进建议：明确区分两者职责并在注释中说明，若重叠则删除其中之一。

## 迭代第 8 轮

1. **问题描述**：打卡分析工作流缺少解析失败兜底策略。详细设计的第5.2.4节定义了punch-analysis工作流的输入变量和输出结构定义，但遗漏了三层降级解析兜底策略，可能引发前端解析异常。
   - 所在位置：第 5.2.4 节（punch-analysis 工作流）
   - 严重程度：一般
   - 改进建议：在5.2.4节补充“输出格式与解析失败处理策略”段落，对齐其他工作流的三层降级解析框架，定义当JSON解析失败或输出非结构化文本时的默认返回结构及错误响应。
2. **问题描述**：admin_logs审计日志表存在被删除的风险与列名校验错误。详细设计7.3.4节中，将dmin_logs错误归类为了“用户私有表（含 user_id 列）”，且允许执行带有user_id = operatorId的UPDATE/DELETE操作。这导致列名校验错位抛错，且允许修改或删除操作记录，严重违背了不可删除的安全底线。
   - 所在位置：第 7.3.4 节（SQL权限控制伪代码 - validateRowLevelPermission）
   - 严重程度：严重
   - 改进建议：在7.3.4节的表分类中为dmin_logs单独建立规则，明确规定对dmin_logs严禁执行任何INSERT、UPDATE和DELETE操作；若允许Agent查询日志，则SELECT语句中需强制校验operator_id = operatorId。
3. **问题描述**：表外键字段命名存在严重语义混淆。life_plans表引入了plan_id作为方案组ID。但在punch_in表中，外键字段plan_id关联目标为life_plans.id（主键）。同名字段语义错位，易引发认知混淆和潜在关联 BUG。
   - 所在位置：第 2.2 节（life_plans 与 punch_in DDL 定义）、第 2.5 节（数据字典）
   - 严重程度：一般
   - 改进建议：为消除歧义，建议将punch_in表中的外键字段明确命名为plan_item_id（指向具体方案项life_plans.id）；或者将life_plans表中的方案组标识重命名为group_id / plan_group_id。

## 迭代第 9 轮

1. **问题描述**：缺失“AI服务完全不可用”时的历史数据展示降级规范及UI定义。当前详细设计遗漏了基于历史数据的UI降级展示策略，缺少状态管理逻辑及UI组件库中“上次生成时间”或“繁忙提示”的模板。
   - 所在位置：第 4.6 节（交互状态组件设计）、涉及 AI 生成对应页面（如 LifePlan.vue、News.vue、HealthAdvice.vue）的前端逻辑描述
   - 严重程度：严重
   - 改进建议：1. 在第 4.6 节新增“历史数据降级提示条”UI组件模板（包含“上次生成时间”展示格式）。2. 在涉及 AI 生成的对应页面的前端逻辑描述中，补充请求异常捕获后的缓存数据读取及降级UI渲染逻辑。
2. **问题描述**：缺失统一的“AI 服务暂不可用”错误状态UI组件。在错误重试组件设计中，仅定义了“网络连接失败”和“无权限访问(403)”等错误模板，遗漏了专门针对 AI 降级场景的统一错误组件。
   - 所在位置：第 4.6.4 节（错误重试组件）
   - 严重程度：一般
   - 改进建议：在第 4.6.4 节中，增加一个“AI 服务不可用”的 HTML 模板，包含“机器人宕机”等相关图标、指定的错误文案（如“AI 服务暂不可用，请稍后重试”）及手动重试按钮。
3. **问题描述**：缺失全局性能指标（如 RTO、响应时间）的架构级说明和验收映射。关于 1s/3s 的前端及接口性能响应边界，以及 30 分钟 RTO 的运维规范未体现为性能测试指标。
   - 所在位置：第 6 节（部署详细设计）、第 8 章（验收标准清单）
   - 严重程度：轻微
   - 改进建议：1. 建议在第 6 节（部署详细设计）补充关于 RTO 及高可用局限性的描述。2. 建议在第 8 章（验收标准）中单列出“非功能及性能验收标准”（如页面 1s/3s 加载阈值验证）。


## 迭代第 10 轮

1. **问题描述**：审计日志（`admin_logs`）防篡改机制被系统后门破坏。在工具分发逻辑的 `validTables` 中错误包含了 `admin_logs`，且原生 `execute_SQL` 兜底路径允许管理员绕过行级校验执行写操作，导致管理员可以通过Text2SQL任意修改或删除审计日志。
   - 所在位置：`server/routes/dify.js`，工具分发逻辑（约5954/5974/5994行）及原生 `execute_SQL` 兜底路径。
   - 严重程度：严重
   - 改进建议：在管理员特权路由及所有相关的写入/删除路径下严格剔除 `admin_logs` 表，并在底层彻底阻断任何对审计日志的篡改或删除途径。
2. **问题描述**：健康资讯分类生成的“降级数据源”存在字段虚无。试图降级使用仅包含账号级别属性的 `users` 表获取健康数据，而该表无任何生理指标，导致向Dify工作流传递空载或无关数据甚至可能引发工作流报错。
   - 所在位置：5.2.3 节 `health-article-generator` 工作流输入定义及降级逻辑。
   - 严重程度：严重
   - 改进建议：重新设计降级策略，例如在用户无相关健康指标时传递合理的默认值，或阻断生成请求并提示用户先完成风险评估，而不是查询无对应字段的 `users` 表。
3. **问题描述**：Express 后端 SSE 流式代理存在“僵尸请求”内存泄露风险。后端遗漏了在客户端主动断开连接时终止上游流式连接的闭环处理。
   - 所在位置：5.2.6 节等 Express 后端 SSE 代理层设计。
   - 严重程度：严重
   - 改进建议：在后端 Express 的相关路由中配置连接断开的监听事件（例如 `req.on('close', () => abortController.abort())`），确保在客户端断开时同步终止上游 Dify 的模型生成以释放服务器资源。
4. **问题描述**：自动生成的资讯直接污染全局 `articles` 库导致数据严重膨胀。自动生成的文章被直接写入公共表 `articles` 且无差别返回给所有用户，会使得文章列表充斥同质化生成的冗余内容。
   - 所在位置：3.2.21 节 `POST /api/articles/generate` API 的写入和读取逻辑。
   - 严重程度：严重
   - 改进建议：引入用户私有化隔离（例如记录文章归属者仅对自己可见）、内容排重机制或添加管理员二次审核机制，避免自动生成内容无差别污染公共文章列表。
