# 质量审查报告 — 详细设计 v2（第2轮）

## 审查元信息

| 维度 | 内容 |
|------|------|
| 审查对象 | `a_v2_req_v2.md`（详细设计 v2，5073 行） |
| 审查视角 | 实现者视角：需求是否可直接作为开发依据、验收标准是否明确可量化、是否存在隐含的模糊地带 |
| 审查侧重 | 需求响应充分度、整体深度和完整性、事实错误与逻辑矛盾（避免重复验证内部审议已覆盖的技术可行性维度） |
| 迭代轮次 | 第 2 轮 |
| 参考文档 | SRS v2（`docs/1_requirements_analysis_v2.md`）、迭代反馈（`iteration_history.md`） |

## 审查概览

第 1 轮迭代反馈的 5 个问题（2 严重 + 3 一般）已在当前产出中修复：iframe 架构已全面替换为 Vue3 SPA、TypeScript 类型定义已新增（3.8 节）、章节号引用已修正、SQL 安全校验已改为白名单模式、跨模块数据传递路径已重新设计。修复质量良好，无遗漏。

本轮审查发现 **12 个新问题**（3 严重、5 一般、4 轻微），主要集中于：与 SRS v2 的 Store 结构不一致、技术选型表与代码示例矛盾、Vue3 SPA 开发基础设施（main.ts、Vite 代理、.vue 类型声明）缺失、以及若干代码级技术错误。

---

## 严重问题（Critical）

### 问题1：chatStore 结构与 SRS v2 根本性不一致

- **问题描述**：SRS v2 第 4.10 节（line 419-479）明确定义 chatStore 使用按医生 ID 维护的 `Map<number, string>` 映射（`doctorConversations`），配合 `setDoctorConversation`/`getDoctorConversation` 方法，支持同时维护多个医生的独立对话会话。但详细设计 3.7/3.8 节定义的 chatStore 仅使用单一 `conversationId: string | null` 字段，无法区分不同医生的会话。这意味着用户从医生 A 切换到医生 B 对话时，医生 A 的 conversation_id 会丢失，Dify 无法恢复医生 A 的对话上下文。
- **所在位置**：第 3.7 节（line 1998-2014）chatStore 接口定义；第 3.8.8 节（line 2367）ChatState 类型定义
- **严重程度**：严重 — 导致多医生对话功能的核心数据模型不成立
- **改进建议**：将 chatStore 的 conversation_id 从单一字段改为 `Map<number, string>`（键为 doctorId），新增 `setDoctorConversation(doctorId, id)` 和 `getDoctorConversation(doctorId)` 方法；保留 `assistantConversationId` 用于 AI 助手场景；保留 `fabOpen`、`isStreaming`、`messages` 等 UI 状态字段（这些是详细设计合理的增量细化）

### 问题2：技术选型表与代码示例矛盾 — SweetAlert2/marked.js 引入方式不可共存

- **问题描述**：第 1.3 节技术选型表将 SweetAlert2 和 marked.js 的引入方式列为 `/static/lib/` 静态 script 文件。但第 4.4.4 节 useUI.ts 使用 `import Swal from 'sweetalert2'`（ES 模块导入）。在 Vite + TypeScript 构建环境中，静态 `<script>` 标签加载的库不会暴露为可 import 的 ES 模块，而 `npm install` 的包不需要放入 `/static/lib/` 目录。这两种引入方式互斥，实现者无法确定应遵循哪一个。
- **所在位置**：
  - 第 1.3 节技术选型表（line 121-122）：SweetAlert2 行标注引入方式为 `/static/lib/sweetalert2.min.js`；marked.js 行标注引入方式为 `/static/lib/marked.min.js`（line 124）
  - 第 4.4.4 节 useUI.ts（line 3423）：`import Swal from 'sweetalert2'`
- **严重程度**：严重 — 实现者按任一方式操作后另一处代码无法通过编译/运行
- **改进建议**：统一选择一种引入策略：(A) 推荐方案——将 SweetAlert2 和 marked.js 改为 npm 依赖（加入 package.json），从技术选型表中移除对应的 `/static/lib/` 条目，4.4.4 节代码维持 ES import；(B) 备选方案——保持 static 文件引入，4.4.4 节改为声明全局类型 `declare const Swal: typeof import('sweetalert2').default` 并使用全局变量

### 问题3：authStore 结构与 SRS v2 不一致 — role 存储策略和 API 签名矛盾

- **问题描述**：SRS v2 第 4.10 节（line 378-416）定义 authStore 具有独立于 `userInfo` 的 `role` ref，且 role 值单独持久化到 `localStorage.getItem('role')`，使用 `setAuth(token, role, user)` / `clearAuth()` 方法对。详细设计 3.7 节定义的 authStore 将 role 作为 `user` 对象的嵌套字段（`user: User | null`，User 接口含 role），仅持久化 token 到 localStorage，使用 `login()` / `logout()` / `setToken()` / `clearToken()` 方法。SRS v2 的独立 role localStorage 存储策略有其安全考量（路由守卫在 Pinia 未完全初始化时即可读取 role 判断管理员权限），详细设计未实现此机制。
- **所在位置**：
  - 第 3.7 节（line 1979-1996）authStore 接口定义
  - 第 3.8.8 节（line 2363-2364）AuthState 类型定义
  - 第 1.6.2 节（line 360-387）路由守卫代码中访问 `authStore.role`
- **严重程度**：严重 — 路由守卫在应用冷启动时可能因 role 未从 localStorage 恢复而错误放行/拦截
- **改进建议**：(1) 在 authStore 中新增独立的 `role` ref，启动时从 `localStorage.getItem('role')` 恢复；(2) 登录成功时将 role 同时写入 localStorage；(3) 路由守卫先检查独立 role ref 再降级检查 user.role；(4) logout 时同时清除 localStorage 中的 role

---

## 一般问题（Major）

### 问题4：DOMPurify 在技术选型表中重复出现

- **问题描述**：第 1.3 节技术选型表第 8 行（line 119）和第 10 行（line 125）均列出 DOMPurify 3.x，两行的"用途"描述不同但"引入方式"不同（前者写 `/static/lib/purify.min.js`，后者写 `CDN或 /static/lib/purify.min.js`）。重复条目造成混淆，实现者不确定应参考哪一行。
- **所在位置**：第 1.3 节技术选型表（line 119、line 125）
- **严重程度**：一般
- **改进建议**：合并为一行，注明"用途"涵盖两行的描述，引入方式统一为 `/static/lib/purify.min.js`（与全本地引入策略一致）

### 问题5：useApi.ts 中 upload 函数的 Content-Type 设置有误

- **问题描述**：第 4.4.1 节 useApi.ts 的 `upload` 函数手动设置 `'Content-Type': 'multipart/form-data'`（line 3330）。浏览器发送 FormData 时需自动附加 `boundary` 参数生成 `multipart/form-data; boundary=----WebKitFormBoundary...` 格式的 Content-Type 头。手动覆盖为不含 boundary 的纯字符串会导致 multer 无法解析上传文件，返回错误。这是已知的 axios + FormData 常见坑点。
- **所在位置**：第 4.4.1 节（line 3328-3333）upload 函数
- **严重程度**：一般 — 导致头像上传功能不可用
- **改进建议**：删除 `headers: { 'Content-Type': 'multipart/form-data' }` 配置项，让 axios 自动检测 FormData 并设置正确的带 boundary 的 Content-Type 头

### 问题6：Vue3 SPA 入口文件 main.ts 结构缺失

- **问题描述**：文档在多处引用 `main.ts`（第 1.4 节目录结构 line 153、4.2 节状态管理表 line 2900），但未给出 main.ts 的初始化内容——包括：`createApp(App)` 调用、`.use(router)` / `.use(pinia)` 注册顺序、全局组件注册（TabBar、FabButton）、Axios 拦截器的安装时机。作为 Vue3 SPA 的核心启动文件（相当于 v1 架构中 index.html 的 `<script>` 初始化代码），其缺失意味着实现者需要自行推断初始化顺序和依赖关系，存在引入启动时序 bug 的风险。
- **所在位置**：全文缺失；最接近的内容是第 4.3 节 App.vue 流程图（line 3060-3089）中的"初始化模块"步骤，但该步骤描述的是运行时行为而非 main.ts 代码结构
- **严重程度**：一般 — 实现者需自行设计入口文件的初始化逻辑
- **改进建议**：新增 "main.ts 初始化流程" 子节，明确：(1) createApp(App) → use(pinia) → use(router) 调用顺序；(2) Axios 拦截器注册时机（应在 createApp 之后、app.mount 之前）；(3) 全局组件注册方式；(4) pinia-plugin-persistedstate 的安装和配置

### 问题7：Vite 开发代理配置缺失 — 前后端开发环境联调断点

- **问题描述**：生产环境通过 Nginx 反向代理将 `/api/*` 转发至 Express :3000，但文档未涉及开发环境配置。Vite 开发服务器默认运行在 5173 端口，Express 运行在 3000 端口，前端 `useApi().get('/api/user/profile')` 请求会发往 `http://localhost:5173/api/user/profile`，无法到达 Express。需要 Vite 的 `server.proxy` 配置将 `/api` 代理到 `http://localhost:3000`。缺失此配置，开发阶段前后端无法联调。
- **所在位置**：全文缺失；第 1.4 节目录结构中列出 `vite.config.ts`（line 145）但未给出其内容
- **严重程度**：一般 — 后续开发者需自行发现并解决此断点
- **改进建议**：在 vite.config.ts 的规格说明中补充开发代理配置：`server: { proxy: { '/api': { target: 'http://localhost:3000', changeOrigin: true } } }`

### 问题8：riskFormStore 缺少 sessionStorage 持久化 — 表单刷新丢失

- **问题描述**：SRS v2 第 4.4 节（line 260-262）明确要求风险预测多步骤表单数据在页面刷新后通过 sessionStorage 自动恢复——"每完成一步，表单数据写入 Pinia riskFormStore 并同步序列化到 sessionStorage"。详细设计 3.7/3.8 节的 riskFormStore 定义中仅有内存级 `formData` 和 `result`，无 sessionStorage 读写逻辑（saveStep/saveResult 仅更新 ref，reset 仅重置 ref）。用户在网络不稳定环境下刷新页面将丢失已填写的表单步骤数据。
- **所在位置**：第 3.7 节（line 2016-2029）riskFormStore 接口定义；第 3.8.8 节（line 2370）RiskFormState 类型定义
- **严重程度**：一般 — 表单数据丢失导致用户体验显著下降
- **改进建议**：在 riskFormStore 的 `saveStep` 方法中增加 `sessionStorage.setItem('risk_form_data', JSON.stringify(formData))`；在 store 初始化时增加 `loadFromStorage()` 从 sessionStorage 恢复数据；在 `reset()` 中增加 `sessionStorage.removeItem('risk_form_data')`

---

## 轻微问题（Minor）

### 问题9：缺少 `.vue` 模块 TypeScript 类型声明文件

- **问题描述**：第 1.4 节目录结构（line 151-193）未包含 `src/env.d.ts` 或 `src/shims-vue.d.ts` 文件。TypeScript 编译器默认不识别 `.vue` 文件的模块类型，在所有 `.ts` 文件中 `import Xxx from '@/views/Home.vue'` 会报 `Cannot find module` 错误。Vite + Vue 官方脚手架（create-vue）自动生成此声明文件，手动搭建项目时必须显式添加。
- **所在位置**：第 1.4 节目录结构
- **严重程度**：轻微 — 缺少此文件会导致 TypeScript 编译报错但 Vite 仍可构建（Vite 的 vue 插件处理 .vue 文件不依赖 TS 类型声明）
- **改进建议**：在 `src/` 下增加 `env.d.ts`，内容为 `/// <reference types="vite/client" />` + `declare module '*.vue' { import type { DefineComponent } from 'vue'; const component: DefineComponent<{}, {}, any>; export default component; }`

### 问题10：useSSE.ts reader 循环条件逻辑缺陷

- **问题描述**：第 4.4.2 节 useSSE.ts（line 3372）的 while 循环使用 `while (reader)` 作为条件，但 `reader` 是从 `response.body?.getReader()` 获取的 ReadableStreamDefaultReader 对象，一旦获取成功即为真值且永不变化。循环实际上靠 `if (done) { callbacks.onComplete?.(); break; }` 退出，条件 `while (reader)` 永远不会为 falsy。虽然功能上正确（循环被 break 终止），但 `while (reader)` 条件具有误导性——它暗示 reader 可能变为 falsy，但实际上不会。
- **所在位置**：第 4.4.2 节（line 3372）
- **严重程度**：轻微 — 不影响功能但降低代码可读性
- **改进建议**：将 `while (reader)` 改为 `while (true)`，使循环退出逻辑完全由 `done` 控制，意图更清晰；同时如果 `reader` 为 null（response.body 不存在），应在循环前提前处理而非依赖 while 条件

### 问题11：4.2 节导言与表格内容自相矛盾

- **问题描述**：第 4.2 节导言（line 2896）声明 sessionStorage 在 Vue SPA 中"极少使用（页面切换不销毁 Pinia Store，无需 sessionStorage 中转）"。但紧随其后的状态管理方案表（line 2898-2926）显示 sessionStorage 在 4 个页面中被使用（Home.vue：数据缓存 1 小时过期；LifePlan.vue：方案缓存 30 分钟过期；News.vue：列表分页状态和列表数据缓存 5 分钟过期；Consultation.vue：无 sessionStorage...）。"极少使用"的定性描述与表格中 4/12 页面的实际使用频率矛盾，可能误导实现者对 sessionStorage 的适用场景产生困惑。
- **所在位置**：第 4.2 节导言段落（line 2896）vs 状态管理方案表（line 2898-2926）
- **严重程度**：轻微 — 不影响实现但造成架构原则表述不一致
- **改进建议**：将导言中的"极少使用"改为"仅用于页面级数据缓存（非跨页面通信）"，使定性描述与表格中的数据缓存使用场景一致

### 问题12：路由元信息字段命名与 SRS v2 不一致

- **问题描述**：详细设计 1.6.1 节使用 `authRequired` 和 `adminRequired` 作为路由元信息字段名（line 333-345），而 SRS v2 第 1.1 节（line 34, 44-45）使用 `requiresAuth` 和 `requiresAdmin`。1.6.2 节路由守卫代码（line 365-367）也使用 `to.meta.authRequired` 与 SRS v2 的 `to.meta.requiresAuth` 不一致。虽然功能等价，但字段名不一致意味着：如果实现者先阅读 SRS v2 生成了路由配置，然后对照详细设计补充守卫逻辑，会因为字段名不匹配而出现鉴权逻辑空跑（守卫检查的字段始终为 undefined，所有路由被放行）。
- **所在位置**：第 1.6.1 节路由映射表（line 333-345）；第 1.6.2 节路由守卫代码（line 365-367）
- **严重程度**：轻微 — 功能等价，但字段名不同增加 SRS→设计→实现三阶段之间信息传递的出错概率
- **改进建议**：统一使用 SRS v2 的字段命名 `requiresAuth` / `requiresAdmin`（SRS 是权威来源），或至少在 1.6.1 节增加注释说明与 SRS v2 的字段名映射关系

---

## 审查结论

当前产出（a_v2_req_v2.md）已完成第 1 轮迭代反馈的全部修复，前端架构从 iframe 到 Vue3 SPA 的转换质量良好。本轮审查新发现 12 个问题，其中 3 个严重问题（chatStore 结构矛盾、SweetAlert2/marked.js 引入方式互斥、authStore role 存储策略不一致）均涉及详细设计与 SRS v2 权威需求的对齐缺口，需优先修复。5 个一般问题（DOMPurify 重复、upload 函数 Content-Type bug、main.ts 缺失、Vite 代理缺失、riskFormStore 无 sessionStorage 持久化）会直接阻塞或阻碍开发实施。4 个轻微问题影响代码质量和文档一致性。

建议优先修复 3 个严重问题和 5 个一般问题后，再进行下一轮审议。
