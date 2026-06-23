# 需求文档审查报告（v1）

## 审查结果

**REJECTED**

## 逐维度审查

### 1. 忠实性

**[通过]** 第1.2节已替换为 Vue3 SPA 前端架构图（路由树 + 组件层级 + Pinia Store 体系），与用户需求一致。

**[通过]** 第1.3节技术选型表已新增 Vue 3、TypeScript 5、Vite 5、Vue Router 4、Pinia 2、Axios、DOMPurify 条目。

**[通过]** 第1.4节模块结构已完整重写为 Vue3 SPA 目录结构（views/、composables/、stores/、types/、components/、router/）。

**[通过]** 第1.5节跨模块通信已从 postMessage 整体替换为 Pinia Store 跨组件通信。

**[通过]** 第1.6节前端路由已从 Hash 路由整体替换为 Vue Router 4 history 模式路由。

**[通过]** 第3.8节新增完整的 TypeScript 类型定义（API 通用类型、认证类型、业务实体类型、风险预测类型、方案类型、打卡类型、对话/SSE 类型、Pinia Store 类型），覆盖全部 32 个端点。

**[通过]** 第4.4节公共模块已从纯 JS（api.js、auth.js、message.js、ui.js）整体替换为 TypeScript Composables（useApi.ts、useSSE.ts、useAuth.ts、useUI.ts），函数签名带完整类型标注。

**[通过]** 第5.1节 Dify 应用总览表"所属功能"列已改为引用详细设计文档内部章节号（API接口 3.1.x），不再引用不存在的 SRS 章节号。

**[通过]** 第7.3.3节 SQL 安全校验已从黑名单正则（仅覆盖4种DDL）改为白名单模式（仅允许 SELECT/INSERT/UPDATE/DELETE），完全对齐 Agent 系统提示词（5.2.6节）中的 DDL 禁止声明。

**[通过]** 第1.5.3节跨模块数据流图已重写为 Vue3 SPA 两套替代方案（Pinia Store 共享方案A + 路由参数传递方案B）。

**[通过]** 第7.5节 Markdown 安全行已从 "marked.js sanitize选项" 改为 "marked.js + DOMPurify 净化"，第1.3节技术选型表已新增 DOMPurify 条目。

**[通过]** 第5.2.5节工具回调 URL 已从硬编码内网 IP 改为 `{EXPRESS_PUBLIC_URL}` 环境变量；第6.2.2节 .env.example 已新增该变量；新增6.4节网络可达性要求。

**[通过]** 第3.7节 chatStore 已定义 `fabOpen` 和 `toggleFab()` 替代 FAB_OPEN 消息；第4.6.3节健康建议为空模板已改用 `@click="chatStore.toggleFab()"`。

**[通过]** 第4.5.2节 rounded-full 表述已从"需要全圆角使用"修正为"需要正圆或胶囊形使用"，补充了适用场景说明。

**[问题-严重]** 第1.1节系统整体架构图（第7-64行 ASCII 框图）仍展示 iframe 架构：图中前端层为 "index.html (主框架 SPA)" 内含 "iframe 容器" 挂载 `/pages/home.html`、`/pages/consultation.html` 等 .html 子页面。用户需求（requirement.md 第1条修订指令）明确要求"将 iframe 容器 + .html 子页面架构替换为 Vue3 SPA 架构图（Vue Router + 组件树）"。当前第1.1节的架构图与已更新的第1.2节 Vue3 SPA 架构图直接矛盾——同一份文档的第1.1节和第1.2节描述了两种互斥的前端架构，读者无法判断哪一个是正确的。

**[问题-严重]** 第4.1节（页面组件树）全部 12 个子节（4.1.1-4.1.12）的 DOM 结构模板仍描述旧 iframe 架构的 HTML 页面层级。虽然每个子节标题标注了过渡映射（如 "index.html → App.vue"），但实际 DOM 树内容未更新为 Vue 3 单文件组件 Template 结构。典型矛盾：4.1.1 节第 2382 行仍包含 `<iframe id="app-iframe" src="/pages/home.html">` 元素，这与第1.2节描述的 `<router-view />` 路由出口机制完全矛盾。

**[问题-严重]** 第4.2节状态管理方案表（第2884-2910行）仍以 .html 页面为粒度描述状态存储（home.html、consultation.html、risk.html 等共12行），其中多个条目引用了仅存在于旧 iframe 架构中的存储模式：如 "sessionStorage transfer_data"（第2893行 risk.html 行）、"localStorage conversation_doctor_{id}"（第2890行 consultation.html 行）。这些存储模式依赖 iframe 架构特有的"跨页面 sessionStorage 中转"机制，在 Vue Router SPA 路由切换中不存在意义——Vue SPA 页面切换不销毁全局 Pinia Store，不需要通过 localStorage/sessionStorage 中介传递数据。

**[问题-严重]** 第4.3节 JS 逻辑流程图中有 5 个页面的流程图仍引用已废弃的 iframe 架构机制：
- index.html 流程图（第3043-3079行）完整描述了 Hash路由管理器（"注册hashchange监听"、第3048行）、postMessage消息总线（"注册message事件监听器"、第3050行）、postMessage 消息路由按 type 分发（第3062-3071行）、"postMessage广播AUTH_SYNC"（第3077行），这些机制在 Vue3 SPA 中全部已废弃。
- home.html 流程图（第2915-2931行）使用 "postMessage NAVIGATE" 进行跨页面导航（第2928-2929行）。
- profile.html 流程图（第3081-3114行）使用 "postMessage AUTH_SYNC" 进行登录态同步（第3111行）。
- login.html 流程图（第3193-3231行）使用 "postMessage AUTH_SYNC" 进行登录态广播（第3218行）。
这些旧架构流程与第1.5-1.6节描述的 Pinia Store 响应式通信 + Vue Router 导航机制直接矛盾。

### 2. 清晰性

**[问题-严重]** 文档存在系统性自相矛盾。第1.2-1.7节描述 Vue3 SPA（Pinia Store、Vue Router 4、组件树），第4.1-4.3节仍大量描述 iframe SPA（postMessage、hashchange、.html 页面）。一个架构师或开发者阅读此文档时，将无法确定应该按第1章还是第4章的描述来实现前端——两者的技术方案互斥且不可共存。具体矛盾点：
- 前端的跨模块通信是用 Pinia Store（第1.5节）还是 postMessage（第4.3节 index.html 流程图）？
- 前端路由是用 Vue Router 4 history 模式（第1.6.1节）还是 Hash 路由 + hashchange（第4.3节 index.html 流程图）？
- 页面是 .vue 单文件组件（第1.4节目录结构）还是 .html 页面（第4.1节全部 DOM 模板）？

**[通过]** 第3.2节各端点请求/响应 JSON Schema 格式统一，边界清晰。

**[通过]** 第5章 Dify 配置描述完整，工作流编排逻辑清晰。

**[通过]** 第6-7章部署和安全设计无歧义。

### 3. 完备性

**[通过]** TypeScript 类型定义（3.8节）覆盖所有 API 请求/响应类型、业务实体类型、SSE 事件类型、Pinia Store 类型。

**[通过]** 第4.4节 Composables 设计提供了完整的函数签名和模块职责说明。

**[通过]** 修订说明（v2-v5）详细记录了各轮迭代的修改内容和跨节结构性变更，可追溯性好。

**[问题-一般]** 第4.2节状态管理方案表缺少对 Vue3 SPA 中"组件内 ref/reactive 状态"与"Pinia Store 全局状态"的明确划分标准。当前表中多数页面数据仍标记为 sessionStorage/localStorage 存储，而在 Vue SPA 中这些数据通常应由 Pinia Store 或组件内响应式变量管理，仅在需要跨会话持久化的场景（JWT Token、用户偏好）才使用 Web Storage。

## 修改要求

- **问题**：第1.1节系统整体架构图仍展示 iframe 架构，与已更新的第1.2节 Vue3 SPA 架构图矛盾
- **原因**：下游架构师/开发者阅读第1章时会看到两种互斥的前端架构图，无法确定应以哪个为准做技术决策。第1.1节作为系统整体架构图是全文首个技术描述，其错误导向影响范围最大
- **建议方向**：重绘第1.1节整体架构图的"用户浏览器"层，将 iframe 容器 + .html 子页面替换为 Vite 构建产物 dist/ 目录 + Vue Router `<router-view />` 路由出口。可将第1.2节的组件树摘要嵌入第1.1节前端层位置

- **问题**：第4.1节全部 12 个子节的 DOM 模板仍描述旧 iframe 架构的 .html 页面层级，第4.2节状态管理表仍以 .html 页面为粒度，第4.3节 5 个页面流程图仍引用 postMessage/Hash路由/hashchange
- **原因**：这些章节合计约占全文 30% 的篇幅，描述的是已废弃的旧架构。实现者若按这些章节编写代码，将产出无法集成的废代码；若忽略这些章节，则缺少 Vue3 SPA 下的页面组件设计参考。无论哪种情况都导致设计文档在实践层面的可用性严重受损
- **建议方向**：
  1. 第4.1节：将全部 12 个子节的 DOM 模板从旧 iframe .html 层级重写为 Vue 3 .vue 单文件组件 Template 结构（`<template>` 块），删除 `<iframe>` 元素，TabBar/FAB/AiChatDialog 移入 App.vue 层级
  2. 第4.2节：将状态管理方案表按 Vue 3 组件粒度重列，明确区分 Pinia Store 全局状态（authStore、chatStore、riskFormStore）与组件内 `ref`/`reactive` 局部状态，仅对跨会话持久化场景保留 localStorage 引用
  3. 第4.3节：将所有包含 postMessage/Hash路由/hashchange 引用的流程图（index.html、home.html、profile.html、login.html）更新为基于 Pinia Store + Vue Router 的 Vue3 SPA 流程：
     - 将 "postMessage AUTH_SYNC" → "authStore.login() 自动触发响应式通知"
     - 将 "postMessage NAVIGATE" → "router.push()"
     - 将 "Hash路由管理器/hashchange" → "Vue Router beforeEach 全局前置守卫"
     - 将 "postMessage广播AUTH_SYNC" → "authStore.clearToken() 响应式自动通知"
  4. 将第4.1-4.3节中所有 .html 文件引用统一替换为对应的 .vue 组件名（如 home.html → Home.vue）

- **问题**：第4.2节状态管理方案表中多处引用仅存在于 iframe 架构中的存储模式（sessionStorage transfer_data、localStorage conversation_doctor_{id}）
- **原因**：在 Vue SPA 中页面切换通过路由而非 iframe src 更新实现，不存在跨 iframe 通信需求，sessionStorage 中转缓存模式完全无意义。若实现者按此模式编写代码（如在 LifePlan.vue onMounted 中从 sessionStorage 而非 Pinia riskFormStore 读取数据），将导致数据传递断裂
- **建议方向**：将 risk.html 的 "sessionStorage transfer_data" 行替换为 "Pinia riskFormStore.result"；将 consultation.html 的 "localStorage conversation_doctor_{id}" 评估是否应升入 Pinia chatStore 或保留（Vue SPA 中 localStorage 对跨会话持久化仍有效）
