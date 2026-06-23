# 需求文档审查报告（v1）

## 审查结果

REJECTED

## 逐维度审查

### 1. 忠实性

**[通过]** 需求文档（a_v3_copy_from_v2.md）作为详细设计 v2 的澄清迭代产出（v6修订版），整体忠实传达了迭代需求（a_v3_iteration_requirement.md）中要求的 22 个问题的修复意图。22 个问题中，22 个在详细设计章节（第3-8章）的代码接口定义、流程图、配置文件中已得到实质性修正。

**[问题-一般]** 第1.5.1节 Pinia Store 概要表（第333行）chatStore 的 state 字段仍标注为 `conversationId: string`，与第3.7节（第2078行）详细接口定义中的 `conversationMap: Map<string, string>` 直接矛盾。概要表是设计者的第一入口，错误的字段名会误导实现者对 Store 数据结构的理解，可能导致按单字段而非 Map 结构实现。

**[问题-轻微]** 第1.5.1节 Pinia Store 概要表（第333行）chatStore 的 actions 列仅列出 `sendMessage(), toggleFab(), navigate()`，遗漏了第3.7节（第2089-2091行）新增的三个 conversation_id 管理方法：`getConversationId()`, `setConversationId()`, `removeConversationId()`。概要表使用者会遗漏这三个关键 action。

**[问题-轻微]** 第1.5.1节 Pinia Store 概要表（第334行）riskFormStore 的 actions 列遗漏了第3.7节（第2110行）新增的 `loadFromStorage()` action，该 action 是页面刷新恢复表单数据的入口，概要表中缺失可能使实现者忽略 sessionStorage 恢复逻辑。

### 2. 清晰性

**[通过]** 以下关键澄清点表述清晰、无歧义：
- conversation_id 统一管理策略（第2095行）：明确三种对话场景的 key 命名规范（doctor_{id}、assistant、admin），解释了如何解决旧方案的不一致问题。
- riskFormStore 的 sessionStorage 持久化策略（第2114行）：清楚说明 saveStep/loadFromStorage/reset 与 sessionStorage 的读写关系，Store 作为唯一读写入口的设计意图明确。
- 第三方库引入方式统一策略（第4447行）：清楚划分 npm 管理库与 /static/lib/ 路径库的边界。
- 多环境 API 代理方案（第4437-4447行）：以表格形式清晰呈现三种环境的代理方案。

**[问题-轻微]** 第4638行 difyService.js 行为规格中，`callAgentStreaming` 函数签名使用 `conversationId`（单数形式）作为参数名，虽不影响下游理解（参数名语义正确），但在全文统一使用 `conversationMap` 管理多会话的语境下，可能引起关于单会话 vs 多会话模型的短暂困惑。建议加注说明该参数为用户当前对话场景的 conversation_id 值（从 chatStore.conversationMap.get(key) 获取）。

### 3. 完备性

**[通过]** 22 个迭代需求问题在以下章节中均已覆盖：
- 新增第8章验收标准清单（90项）：覆盖 API 端点（38项）、前端交互（36项）、SSE 对话（6项）、安全功能（10项）
- 新增 6.3.4 节前端 package.json：完整的前端依赖和 scripts 配置
- 新增 6.3.5 节 difyService.js 行为规格：blocking/streaming 模式流程、超时重试策略、错误码映射
- 新增 6.2 节多环境 API 代理方案：三种环境的路由方案
- 数据库初始化 bcrypt 哈希自动生成（第4610-4617行）
- 生活方案输出格式的三层降级解析策略（第3880-3887行）
- Keepalived 密码占位符化及 VRRP 安全说明（第4710/4721行）

## 修改要求

- **问题**：第1.5.1节 Pinia Store 概要表（第330-334行）的 chatStore 行与第3.7节详细接口定义存在两处不一致——(1) state 字段 `conversationId: string` 应为 `conversationMap: Map<string, string>`；(2) actions 缺失 `getConversationId()`, `setConversationId()`, `removeConversationId()` 三个方法。同时 riskFormStore 的 actions 缺失 `loadFromStorage()`。
- **原因**：概要表是设计者浏览文档的第一入口，错误的 state 字段名（单字段 vs Map）会直接导致实现者按错误的数据结构设计 Store，造成与后续详细定义章节的实现矛盾。
- **建议方向**：(1) 将 chatStore 的"关键 state"列从 `conversationId: string, messages: Message[], fabOpen: boolean` 改为 `conversationMap: Map<string, string>, messages: Message[], fabOpen: boolean, isStreaming: boolean`；(2) 将 chatStore 的"关键 actions"列从 `sendMessage(), toggleFab(), navigate()` 改为 `sendMessage(), toggleFab(), navigate(), getConversationId(), setConversationId(), removeConversationId()`；(3) 将 riskFormStore 的"关键 actions"列从 `saveStep(), saveResult(), reset()` 改为 `saveStep(), saveResult(), reset(), loadFromStorage()`。
