# 再审议判定报告（v3）

## 判定结果

RETRY

## 判定理由

组件B诊断报告（b_v3_diag_v1.md）识别出13个质量问题：2个严重、8个一般、3个轻微。组件B质询报告（b_v3_challenge_v1.md）经抽样验证和逐维度审查后确认所有问题定位准确（LOCATED），三个审查维度（证据充分性、逻辑完整性、覆盖完备性）均通过，仅提出增强性建议。质询报告未对任何问题的存在性或严重程度提出质疑。

组件B内部循环在第1轮即提前终止（实际轮次=1，最大轮次=12），终止原因为LOCATED——审查结论被质询确认，无需继续迭代。

由于诊断报告中存在严重等级问题（2个）和一般等级问题（8个），根据判定标准，不满足PASS的任何条件（不含严重或一般问题、达到最大轮次仍未LOCATED、仅有轻微问题）。判定结果为RETRY，组件A需根据诊断报告的问题列表进行修订。

## 需要解决的问题

### 严重

- **问题描述**：SSE事件类型的TypeScript定义与协议定义不一致。第3.3节协议定义7种事件类型（message/message_end/error/workflow_started/workflow_finished/agent_message/agent_thought），但第3.8.7节TypeScript类型定义仅覆盖前3种，第4.4.2节useSSE.ts也仅解析3种事件，导致workflow_started/agent_message等4种事件被静默丢弃。
- **所在位置**：第3.3节（第1971-1981行）vs 第3.8.7节（第2397-2442行）vs 第4.4.2节useSSE.ts（第3457-3469行）
- **严重程度**：严重
- **改进建议**：(1) 在types/sse.ts中补全SSEWorkflowStartedEvent、SSEWorkflowFinishedEvent、SSEAgentMessageEvent、SSEAgentThoughtEvent四种类型定义；(2) 更新SSEEvent联合类型包含全部7种事件；(3) 在useSSE.ts的streamRequest事件解析switch中增加对4种事件的回调分支；(4) 在SSECallback类型中新增对应的可选回调函数。

- **问题描述**：第3.7节riskFormStore接口定义中`RiskFormData`类型引用不一致。第3.7节formData字段类型声明为`Partial<RiskFormData>`，但第3.8.4节定义的是`RiskPredictRequest`接口，`RiskFormData`从未被定义为interface/type。第3.8.8节中formData又被声明为`Partial<RiskPredictRequest>`，命名不一致导致IDE跳转失败。
- **所在位置**：第3.7节（第2102行）vs 第3.8.8节（第2454行）vs 第3.8.4节（第2289-2300行）
- **严重程度**：严重
- **改进建议**：统一使用`RiskPredictRequest`作为表单数据类型，将第3.7节（第2102行）的`RiskFormData`改为`RiskPredictRequest`，或新增`type RiskFormData = RiskPredictRequest`类型别名。

### 一般

- **问题描述**：life-plan-generator输出格式声明存在逻辑矛盾。文档先声明输出为JSON对象，随后又列出三级解析降级策略（JSON优先→正则提取降级→LLM二次调用降级）。声明与降级策略之间的语义关系未清晰区分，实现者无法判断降级策略是冗余代码还是必要兜底。
- **所在位置**：第5.2.2节"输出格式"段落（第3880-3887行）
- **严重程度**：一般
- **改进建议**：明确区分两层语义：(1) 工作流设计目标——LLM节点应返回JSON格式；(2) Express端的防御性解析策略——因LLM输出存在不确定性，需降级策略兜底。将声明改为"期望输出为JSON对象；因LLM输出存在不确定性，Express端需实现以下三级降级解析策略"。

- **问题描述**：预查询数据注入失败时的错误处理行为缺失。punch-analysis和health-article-generator两个Dify工作流依赖Express预查询SQLite数据并注入inputs，但文档未定义预查询失败时的处理行为（数据库连接断开、查询返回空集时Express应返回什么错误响应）。
- **所在位置**：第3.6节"预查询数据注入说明"段落（第2045-2047行）vs 第5.2.3节/第5.2.4节 vs 第6.3.5节difyService.js行为规格
- **严重程度**：一般
- **改进建议**：(1) 在difyService.js行为规格中新增"预查询失败处理"子节，定义各预查询异常的响应行为——数据库错误→502 DIFY_ERROR；查询结果为空→传递空数组/空对象给Dify；(2) 在工作流输入变量定义表中增加"空值行为"列。

- **问题描述**：Consultation.vue共享组件双模式的行为切换逻辑未完整定义。组件通过`route.params.id`区分医生列表视图和对话视图，但未处理用户先访问对话视图后通过Tab切换离开再返回时是否需要重新验证登录态的边界情况。
- **所在位置**：第1.6.1节路由映射表（第408-409行）vs 第4.1.3节Consultation.vue组件树 vs 第4.3节Consultation.vue流程图
- **严重程度**：一般
- **改进建议**：补充Consultation.vue的`onMounted`伪代码，定义视图判定顺序：(1) 检查route.params.id是否存在；(2) 若存在且需对话视图，检查JWT是否有效，若无效则跳转登录并附带redirect；(3) 从chatStore.conversationMap恢复conversation_id；(4) 渲染对应视图。

- **问题描述**：Consultation.vue的"清除对话"按钮行为未定义后端接口。对话视图头部包含btn-delete清空对话按钮，但第3.1.4节医师对话相关端点清单中无任何删除/清除对话的API端点。清空对话是纯前端操作还是需要调用Dify API删除会话，未明确。三种对话场景（Consultation/Admin/AiChatDialog）的清空对话功能设计不统一。
- **所在位置**：第4.1.3节Consultation.vue组件树（第2578行）vs 第3.1.4节API端点清单
- **严重程度**：一般
- **改进建议**：(1) 明确清空对话按钮的行为——纯前端操作（清空chatStore.messages并调用removeConversationId），还是需要后端删除Dify会话；(2) 若需后端支持，新增DELETE /api/chat/doctor/:id/conversations/:conversation_id端点；(3) 为Admin.vue和AiChatDialog.vue补充清空对话功能入口。

- **问题描述**：Express Dify代理端点的请求体组装逻辑不够详尽。第3.6节映射表定义了字段逐一映射，但未定义：(1) user参数如何从JWT payload提取并注入；(2) Dify不需要的字段是否全部透传还是需要过滤；(3) 嵌套对象是整体透传还是展开为独立字段。
- **所在位置**：第3.6节Express代理层请求参数映射表（第2032-2047行）vs 第3.2.32节Dify代理端点请求体（第1956-1965行）
- **严重程度**：一般
- **改进建议**：新增"请求体组装流程"伪代码子节，包含：(1) 从req.body提取字段→(2) 按映射表转换为Dify参数→(3) 注入JWT提取的user字段→(4) 注入response_mode→(5) 注入预查询数据→(6) 调用difyService。

- **问题描述**：punch-analysis工作流的date_range参数传递路径未贯通。第3.1.6节定义`GET /api/punch/analysis`端点但不接受任何查询参数，但第5.2.4节punch-analysis工作流输入变量中定义了可选的date_range参数，前端Punch.vue页面有日期筛选器但无法将日期条件传递给工作流。
- **所在位置**：第3.1.6节（第1144行GET /api/punch/analysis）vs 第3.6节映射表 vs 第5.2.4节（第3955行）
- **严重程度**：一般
- **改进建议**：(1) 更新GET /api/punch/analysis端点定义为`GET /api/punch/analysis?startDate=&endDate=`；(2) 在第3.6节映射表中新增`startDate, endDate→inputs.date_range.{start, end}`映射规则；(3) 在第4.3节Punch.vue流程图中明确分析区域数据请求时携带日期参数。

- **问题描述**：AI助手健康建议生成的触发机制不明确。diabetes-assistant-agent的Skill 6描述Agent在自然语言对话中自主判断用户意图并触发建议生成，但未定义：(1) 建议生成是否需要用户确认；(2) 生成的建议如何与GET /api/assistant/advice端点关联；(3) 多次生成同一用户建议时的去重逻辑。
- **所在位置**：第5.2.5节Skill 6描述（第3989行）vs 第3.1.9节端点清单 vs 第4.1.10节AiChatDialog.vue
- **严重程度**：一般
- **改进建议**：(1) 补充Skill 6交互流程：Agent先展示建议预览→用户确认→Agent通过execute_SQL写入；(2) 在life_advice表中新增UNIQUE(user_id, title)约束或在前端增加去重逻辑；(3) 明确健康建议写入时的user_id来源。

### 轻微（不阻塞RETRY判定，但建议一并修复）

- **问题描述**：第4.2节状态管理表中LifePlan.vue的sessionStorage过期时长与数据流不匹配。状态管理表标注缓存过期时间为30分钟，但流程图显示onMounted中无条件调用API，未先检查sessionStorage缓存，缓存写入后从未被读取。
- **所在位置**：第4.2节状态管理表（第2996行）vs 第4.3节LifePlan.vue流程图（第3092-3116行）
- **严重程度**：轻微
- **改进建议**：在LifePlan.vue流程图中增加缓存检查步骤，或删除sessionStorage该行的方案缓存条目。

- **问题描述**：第1.1节系统架构图中服务器1的Express :3000端口与第6.1.2节Nginx配置的upstream地址不匹配。架构图未明确标注Express监听地址，可能让部署者误以为仅监听localhost。
- **所在位置**：第1.1节架构图（第50-55行）vs 第6.1.2节服务器1 Nginx配置（第4383-4384行）
- **严重程度**：轻微
- **改进建议**：在第1.1节架构图文字说明中补充"Express监听0.0.0.0:3000，接受来自本机Nginx和服务器2/3 Nginx upstream的请求"。

- **问题描述**：管理员强制密码修改完成后authStore状态清除与路由跳转的竞态条件未处理。改密成功后设置mustChangePassword=false然后router.push/admin，Pinia状态更新和路由跳转之间可能存在竞态窗口导致死循环。
- **所在位置**：第1.6.2节路由守卫（第449行）vs 第4.3节ChangePassword.vue流程图（第3344-3350行）
- **严重程度**：轻微
- **改进建议**：路由守卫中增加`if (to.path === '/change-password') return next()`优先通过，避免死循环。

- **问题描述**：前端与后端的JWT过期时间缺乏明确协调机制。JWT有效期24小时，前端仅在收到401时清除Token，未主动检查Token过期，每次过期请求都产生一次失败HTTP往返。
- **所在位置**：第7.1节JWT鉴权流程（第4773行）vs 第4.4.1节useApi.ts 401拦截（第3386行）vs 第4.3节App.vue流程图
- **严重程度**：轻微
- **改进建议**：在App.vue流程图中增加Token过期主动检查逻辑，使用setInterval每5分钟检查一次isTokenExpired(authStore.token)，若即将过期则提前提示。

---

*判定完成时间: 2026-06-24*
