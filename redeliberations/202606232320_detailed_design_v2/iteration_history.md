## 迭代第 1 轮

1. **问题描述**：整份文档的前端设计部分（约占全文档60%）基于错误的 iframe + Hash路由 + postMessage 架构，与需求明确要求的 Vue3 + TypeScript + Vite SPA 架构完全矛盾。全文12处系统性地使用了 iframe 架构特有的机制（postMessage 消息总线、Hash路由、.html 页面、sessionStorage 中转），无任何 Vue3 SPA 相关内容
   - 所在位置：全文（第1-7章中涉及前端的所有部分）
   - 严重程度：严重
   - 改进建议：按 requirement.md 的五条修订指令逐条执行：(1) 重绘系统架构图为 Vue3 SPA 架构；(2) 替换前端架构图为 Vue Router + 组件层级 + Pinia Store；(3) 所有 Hash 路由替换为 Vue Router 4 history 路由模式（参照 SRS v2 第36-138行）；(4) 所有 postMessage 消息总线替换为 Pinia Store 跨组件通信（参照 SRS v2 第378-546行）；(5) 所有 .html 页面引用替换为 .vue 组件引用
2. **问题描述**：全文缺失 TypeScript 类型定义。所有代码示例均为纯 JavaScript（api.js、auth.js、message.js、ui.js 的函数签名无类型标注），API 端点的请求/响应 JSON Schema 未转化为 TypeScript interface/type 定义，实现者无法获得 IDE 类型提示和编译期检查
   - 所在位置：全文
   - 严重程度：严重
   - 改进建议：(1) 新增 TypeScript 类型定义章节，将所有 JSON Schema 转换为 interface/type 定义；(2) 将公共 JS 模块改写为 TypeScript 模块并标注完整函数签名类型；(3) 定义统一的泛型分页响应类型 PaginatedResponse\<T\>；(4) 定义 SSE 事件类型的 discriminated union；(5) 继承 SRS v2 已有的 authStore、chatStore、riskFormStore 类型定义
3. **问题描述**：第5.1节 Dify 应用总览表的"所属功能"列引用了一组不存在的章节号（4.4 风险预测、4.5 生活方案等），这些是 SRS 的章节号而非详细设计文档的实际章节号，实现者无法根据这些引用定位到对应的设计内容
   - 所在位置：第5.1节 Dify 应用总览表格
   - 严重程度：一般
   - 改进建议：将"所属功能"列改为引用详细设计文档内的实际章节号（如 3.1.4 医师咨询、3.1.3 风险预测），或改为直接的功能描述，删除对 SRS 章节号的交叉引用
4. **问题描述**：第5.2.6节 Agent 系统提示词禁止所有 DDL 操作，但第7.3.3节 SQL 安全校验正则仅匹配 DROP TABLE/DATABASE、ALTER TABLE、CREATE TABLE 四种 DDL，未覆盖 CREATE INDEX、DROP INDEX、CREATE VIEW、DROP VIEW、CREATE TRIGGER、DROP TRIGGER 等 SQLite 支持的 DDL 语句，存在安全校验盲区
   - 所在位置：第5.2.6节与第7.3.3节
   - 严重程度：一般
   - 改进建议：(1) 将正则改为白名单反向匹配（仅允许 SELECT/INSERT/UPDATE/DELETE）；(2) 或在 better-sqlite3 层面禁止 db.exec()；(3) 在 Agent 系统提示词中明确列出所有禁止的 DDL 类型，使策略声明与代码校验对齐
5. **问题描述**：第1.5.3节描述的"风险预测->生活方案"跨模块数据传递路径（risk.html -> postMessage -> sessionStorage -> postMessage转发 -> life-plan.html）依赖 iframe 架构特有的三个机制，在 Vue3 SPA 中全部不存在，且文档未提供替代路径设计
   - 所在位置：第1.5.3节及第1.7节
   - 严重程度：一般
   - 改进建议：在 Vue3 SPA 架构下重新设计跨模块数据传递策略，使用 Pinia shared store 或 Vue Router query params 传递，删除所有 sessionStorage('transfer_data') 和 postMessage(DATA_TRANSFER) 相关描述

## 迭代第 2 轮

1. **问题描述**：chatStore 仅使用单一 `conversationId` 字段（`string | null`），与 SRS v2 要求的 `Map<number, string>` 类型不匹配
   - 所在位置：第3.7节 chatStore 接口定义（第1998-2014行）
   - 严重程度：严重
   - 改进建议：将 chatStore 的 `conversationId` 从 `string | null` 改为 `Map<number, string>` 或 `Record<number, string>`，并按用户 ID 管理多个对话会话
2. **问题描述**：技术选型表中 SweetAlert2/marked.js 引入方式为 `/static/lib/` 路径，但 useUI.ts 中使用 `import Swal from 'sweetalert2'` 的 npm 引入方式，两种方式互斥
   - 所在位置：第1.3节技术选型表（第121-122行）vs 第4.4节 useUI.ts（第3423行）
   - 严重程度：严重
   - 改进建议：统一引入方式——若采用 npm 依赖（推荐），更新技术选型表并移除 `/static/lib/` 中的冗余文件；若保留 CDN 引入，替换 useUI.ts 中 import 语句为全局变量引用
3. **问题描述**：authStore 将 role 作为 user 嵌套字段存储，未独立持久化到 localStorage，页面刷新后角色信息依赖 JWT 解码恢复，增加了不必要的 decode 开销
   - 所在位置：第3.7节 authStore 接口定义（第1979-1996行）
   - 严重程度：严重
   - 改进建议：将 role 独立写入 localStorage（如 `localStorage.setItem('user_role', role)`），authStore 初始化时优先从 localStorage 恢复 role，避免每次页面刷新后重新解码 JWT
4. **问题描述**：Dify diabetes-assistant-agent 系统提示词 Skill 7 引用 `card_analysis` 工具，但该 Agent 仅定义了 `execute_SQL` 一个工具，`card_analysis` 工具在全文未定义
   - 所在位置：第5.2.5节（第3898行）vs 第3918-3930行工具定义
   - 严重程度：严重
   - 改进建议：若 Skill 7 应通过 `execute_SQL` 实现，修改系统提示词为通过 execute_SQL 查询 punch_in 表；若需独立工具，新增 card_analysis 工具定义（名称、类型、回调 URL、请求体模板、对应 Express 端点）
5. **问题描述**：riskFormStore 接口定义仅含内存级状态（`Partial<RiskFormData>`、`RiskResult | null`），无 sessionStorage 读写逻辑；但 Risk.vue 流程图显式依赖 sessionStorage 的读写和清除操作，两者互相矛盾
   - 所在位置：第3.7节（第2016-2029行）riskFormStore 接口定义 vs 第4.3节（第2979-3005行）Risk.vue 流程图
   - 严重程度：严重
   - 改进建议：推荐方案——在 riskFormStore 的 `saveStep` 中增加 sessionStorage 写入，`reset` 中增加 sessionStorage 清除，初始化时增加 `loadFromStorage` 恢复逻辑，保持流程图不变
6. **问题描述**：DOMPurify 在技术选型表中重复出现两行
   - 所在位置：第1.3节技术选型表（第119行和第125行）
   - 严重程度：轻微
   - 改进建议：删除重复行，保留一条 DOMPurify 条目
7. **问题描述**：useApi.ts 中 upload 函数手动设置 `'Content-Type': 'multipart/form-data'`，浏览器使用 FormData 时应自动设置带 boundary 的 Content-Type，手动设置会导致缺少 boundary 参数
   - 所在位置：第4.4节 useApi.ts upload 函数（第3330行）
   - 严重程度：一般
   - 改进建议：删除手动设置的 `'Content-Type': 'multipart/form-data'` 头，让浏览器自动生成带 boundary 的 Content-Type
8. **问题描述**：main.ts 仅在目录结构中列出文件名，全文无具体初始化内容
   - 所在位置：第1.4节目录结构（第153行）
   - 严重程度：一般
   - 改进建议：补充 main.ts 初始化代码规格——createApp、use(router)、use(pinia)、mount 的调用顺序和参数
9. **问题描述**：Vite 开发代理配置缺失——前端 `useApi.ts` 的 `baseURL: '/api'` 依赖代理将请求转发至 Express :3000，但 vite.config.ts 仅列出文件名无实际配置
   - 所在位置：第1.4节目录结构（第145行）
   - 严重程度：一般
   - 改进建议：补充 vite.config.ts 的 server.proxy 配置——将 `/api` 路径代理至 `http://localhost:3000`
10. **问题描述**：`.vue` 模块 TypeScript 类型声明文件缺失，TypeScript 编译 `.vue` 文件时无法识别模块类型
   - 所在位置：全文缺失 env.d.ts 或 shims-vue.d.ts
   - 严重程度：轻微
   - 改进建议：新增 `src/env.d.ts`，包含 `declare module '*.vue' { import type { DefineComponent } from 'vue'; const component: DefineComponent<{}, {}, any>; export default component; }`
11. **问题描述**：useSSE.ts 中 SSE reader 循环条件 `while (reader)` 恒为真，readableStream 的 reader 对象始终为 truthy，应使用 `while (!done)` 结合 read() 返回的 done 标志
   - 所在位置：第4.4节 useSSE.ts（第3372行）
   - 严重程度：轻微
   - 改进建议：将 `while (reader)` 改为 `while (true)` 配合 `const { done, value } = await reader.read(); if (done) break;`
12. **问题描述**：第4.2节文本描述"极少使用 sessionStorage"，但其后的状态管理表格中多个页面使用 sessionStorage，文字描述与表格数据矛盾
   - 所在位置：第4.2节（第2896行）
   - 严重程度：轻微
   - 改进建议：修改文字描述为"部分页面使用 sessionStorage 存储临时表单数据"，与表格实际使用情况一致
13. **问题描述**：路由元信息字段名 `authRequired`/`adminRequired` 与 SRS v2 规定的 `requiresAuth`/`requiresAdmin` 不一致
   - 所在位置：第3.1节路由配置（第331/365/382行）
   - 严重程度：轻微
   - 改进建议：全局替换 `authRequired` 为 `requiresAuth`，`adminRequired` 为 `requiresAdmin`，与 SRS v2 命名约定保持一致
14. **问题描述**：验收标准完全缺失——5073 行文档中零命中"验收标准""验收清单""可量化"等关键词，实现者无法判断各功能的完成标准
   - 所在位置：全文缺失
   - 严重程度：一般
   - 改进建议：在文档末尾新增"验收标准清单"章节，按功能模块组织为表格（功能模块 | 验收项 | 量化标准 | 验证方式），覆盖 API 端点响应、前端交互行为、SSE 流式对话、安全功能四个维度
15. **问题描述**：前端 package.json 完全缺失——第6.2.3节仅含 Express 后端依赖，技术选型表中的 Vue 3、Vite 5、Pinia 2 等前端依赖及 sweetalert2、marked.js、DOMPurify 等库均未在任何 package.json 中出现
   - 所在位置：第6.2.3节（第4391-4410行）vs 第1.3节技术选型表（第113-137行）
   - 严重程度：一般
   - 改进建议：新增前端 package.json，包含 dependencies（vue、vue-router、pinia、axios、sweetalert2、marked、dompurify）和 devDependencies（typescript、vite、@vitejs/plugin-vue、vue-tsc、tailwindcss、postcss、autoprefixer、pinia-plugin-persistedstate）
16. **问题描述**：Express difyService.js 工作流代理行为规格缺失——未定义 blocking/streaming 模式区分、超时重试策略、Dify 响应错误码映射、工作流响应解析和持久化逻辑
   - 所在位置：全文缺失；最接近内容为第1.7节（第432-438行）和第3.6节（第1958-1973行）
   - 严重程度：一般
   - 改进建议：新增 difyService.js 行为规格子节，覆盖 blocking/streaming 调用流程、错误处理矩阵、超时配置与降级策略、Dify API Key 选取逻辑
17. **问题描述**：Vue3 SPA 架构迁移的交叉影响未处理——服务器1 Nginx 配置不包含 `/api/` 反向代理、第三方库引入方式（static/lib vs npm）与 Nginx 配置未同步
   - 所在位置：第1.3节（第121-124行）、第6.1.1节（第4199行）、第6.1.2节（第4288-4325行）
   - 严重程度：一般
   - 改进建议：新增"多环境 API 代理方案"表格，明确开发/生产/实训环境的 API 请求路由方案；统一第三方库引入方式后同步更新 Nginx location 配置和技术选型表
18. **问题描述**：数据库初始化种子 SQL 中管理员密码使用 bcrypt 占位符，需手动执行 node 命令生成哈希替换，`initDatabase()` 函数不含自动哈希生成逻辑，自动化部署易遗漏
   - 所在位置：第2.4节（第752-756行）vs 第6.3节（第4436-4453行）
   - 严重程度：一般
   - 改进建议：在 initDatabase() 中增加 seed 后处理——对 password_changed='0' 的管理员账号自动调用 bcrypt 生成哈希替换占位符；或在 .env 中增加 ADMIN_DEFAULT_PASSWORD 变量供 seed 脚本使用
19. **问题描述**：life-plan-generator Dify 工作流输出解析规格缺失——未说明输出是 JSON 还是自然语言文本，未定义 Express 端解析策略和解析失败时的降级策略
   - 所在位置：第5.2.2节（第3793行）
   - 严重程度：轻微
   - 改进建议：若 Dify 工作流 LLM 节点输出 JSON，标注格式并定义 Schema；若输出为文本，补充正则提取或二次 LLM 调用的解析策略；补充解析失败时的错误响应格式
20. **问题描述**：Consultation.vue、Admin.vue、AI 助手三种对话场景使用三种不同的 conversation_id 管理策略（localStorage 动态键、localStorage 固定键、Pinia Store），缺乏统一设计模式
   - 所在位置：第4.2节（第2906行、第2923行）vs 第3.7节（第2002行）
   - 严重程度：轻微
   - 改进建议：统一为 Pinia Store 管理所有对话的 conversation_id，使用 Map 或 Record 按对话场景键管理
21. **问题描述**：Keepalived 配置中 VRRP 认证密码 `auth_pass diabetes2026` 为明文，文档未就此安全局限性做出说明
   - 所在位置：第6.6节（第4502行、第4533行）
   - 严重程度：轻微
   - 改进建议：在配置旁增加安全说明注释，将示例密码改为占位符
