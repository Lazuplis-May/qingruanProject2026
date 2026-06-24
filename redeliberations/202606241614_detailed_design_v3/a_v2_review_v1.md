# 技术方案审查报告（v2）

## 审查结果

REJECTED

## 逐维度审查

### 1. 技术准确性
**[一般]** `chatStore` 与 `authStore` 接口定义与描述不一致：
- 在 1.5.1 节概要表及 5856 行的总结中，声称 `chatStore` 的属性已更新为使用 `doctorConversations`、`assistantConversationId` 和 `adminConversationId`。而在 3.7 节的核心接口定义中，实际声明的仍是原先的 `conversationMap`。
- 在 1.5.1 节概要表及 5831 行中，声称 `chatStore` 已引入 `activeAbortController`、`registerAbortController`、`abortActiveConnection` 等用于管理 SSE 并发连接，但在 3.7 节的接口定义中完全遗漏了这些属性和 actions 的声明。
- 在 1.2 节中提到调用 `authStore.clearAuth()` 方法，但 3.7 节的 `authStore` 接口定义中并没有 `clearAuth()` 字段，只定义了 `clearToken()`。

### 2. 完备性
**[通过]** 设计方案已基本覆盖上轮迭代提出的所有核心要求：
- 数据库 CHECK 约束已由中文改为英文小写（`male`, `female`, `yes`, `no`, `healthy`, `prediabetes`, `diagnosed` 等）。
- `user_risk_info` 表补充了 `diabetes_history` 字段，`life_plans` 表补充了 `is_active` 字段。
- 增加了 `server/utils/mapper.js` 数据字段映射与中英转换机制设计，并明确了控制器的拦截逻辑。
- 前端路由与组件结构中，成功将医师咨询拆分为 `Consultation.vue` 和 `DoctorChatView.vue`，将健康资讯拆分为 `NewsView.vue` 和 `ArticleDetailView.vue`。
- 补充了跨标签页登录态同步机制设计。
- 补充了特定输入项的 0 值有效性验证（`waist` 和 `systolic_bp`）及错误处理逻辑。

### 3. 可操作性
**[一般]** 接口定义层面的矛盾将直接阻碍 TS 类型的声明及前端开发。若直接按照 3.7 节的接口定义声明 `useChatStore` 和 `useAuthStore`，前端在组件中调用 `chatStore.abortActiveConnection()`、`authStore.clearAuth()` 等方法时将会遇到编译错误，难以进行下一步开发。

## 修改要求

- **问题**：`stores/chatStore.ts` 和 `stores/authStore.ts` 接口定义的代码细节与正文/概要表描述不一致。
- **原因**：这会导致实现者在编写 TypeScript 类型定义时遇到接口签名缺失、命名不一致等问题，从而阻碍编码实现。
- **建议方向**：
  1. 彻底修改 3.7 节中 `stores/chatStore.ts` 的接口声明，将 `conversationMap: Map<string, string>` 替换为 `doctorConversations: Map<number, string>`、`assistantConversationId: string | null` 和 `adminConversationId: string | null`；
  2. 在 3.7 节 `chatStore` 接口中显式补充 `activeAbortController: AbortController | null`、`registerAbortController(controller: AbortController): void` 和 `abortActiveConnection(): void` 等 SSE 并发控制的属性与 Action 的代码声明；
  3. 在 3.7 节 `authStore` 接口中将 `clearToken` 修正或补充为 `clearAuth(): void`，以与正文描述同步。
