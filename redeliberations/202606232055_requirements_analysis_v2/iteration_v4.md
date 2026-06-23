# 再审议判定报告（v4）

## 判定结果

RETRY

## 判定理由

组件B诊断报告（b_v4_diag_v1.md）识别出 9 个质量问题：严重 2 项、一般 5 项、轻微 2 项。具体为：

- **严重问题**：POST /api/articles/generate 分类阶段响应结构缺失（阻塞前后端并行开发）、plan_id 类型不一致（string vs number，可能导致运行时类型校验错误）。
- **一般问题**：三个 GET 端点响应结构缺失、Markdown 渲染 XSS 安全性未提及、risk/history 响应结构模糊、life_plans 表 'other' 枚举值无业务场景、Dify Agent api_key 配置循环依赖。
- **轻微问题**：Vant 4 按钮行为与规范矛盾、sessionStorage 清除时序空白。

组件B质询报告（b_v4_challenge_v1.md）对诊断报告进行了逐维度审查（证据充分性、逻辑完整性、覆盖完备性），结论为 LOCATED——全部 9 个问题均被确认证据充分、逻辑自洽、定位准确，无被质疑项。组件B内部循环实际轮次为 1（最大轮次 12），提前终止且确认审查结论有效。

根据判定标准，审查报告包含严重和一般等级的问题，满足 RETRY 条件。需将问题列表反馈给组件A（生产者）进行修订。

## 需要解决的问题

- **问题描述**：POST /api/articles/generate 分类生成阶段的正常响应 JSON 结构未定义，阻塞前后端并行开发——实现者无法确定分类标签列表的数据格式（简单字符串数组 vs 结构化对象数组）
- **所在位置**：第 6.7 节，第 919-925 行（请求参数表）及第 963 行
- **严重程度**：严重
- **改进建议**：（1）补充分类阶段的正常响应 JSON 结构示例，明确定义分类标签列表的数据格式（字段名、类型、每个分类对象包含的属性）；（2）说明标签列表是否包含描述文本、是否按优先级排序、标签数量上限；（3）标注该响应结构为基于 Dify 工作流输出字段的推断

- **问题描述**：PUT /api/plan/adjust 请求体的 plan_id 参数类型标注为 string，但 POST /api/plan/generate 响应中 plan_id 类型为 number——类型不一致可能导致运行时校验错误，且 plan_id 语义（方案组 ID vs 方案项 ID）不明确
- **所在位置**：第 6.5 节，第 853 行（参数表 plan_id=string）与第 864 行（响应 JSON plan_id=number）
- **严重程度**：严重
- **改进建议**：（1）统一 plan_id 为 number 类型（与 life_plans 表 INTEGER 主键一致）；（2）在 PUT /api/plan/adjust 参数说明中明确 plan_id 为"方案组 ID"；（3）补充说明调整行为是"整个方案组替换"

- **问题描述**：GET /api/assistant/advice、GET /api/chat/doctor/:id/conversations、GET /api/assistant/conversations 三个 GET 端点的正常响应 JSON 结构未定义，与同类列表端点（GET /api/doctors、GET /api/articles）形成对比
- **所在位置**：第 6.9 节第 979-980 行，第 6.4 节第 808 行
- **严重程度**：一般
- **改进建议**：（1）为 GET /api/assistant/advice 补充响应 JSON 示例（至少含 id、title、tags、content、created_at）；（2）为两个会话列表端点补充响应结构——若透传 Dify 原始响应则明确声明，若裁剪重构则给出字段列表

- **问题描述**：全文未提及 AI 生成的 Markdown 文章在前端渲染前的 HTML 净化（XSS 防护）——marked 库默认不执行净化，AI 生成内容是最主要的 XSS 攻击面
- **所在位置**：第 4.6 节第 295 行、第 8.1 节第 1310 行、第 7.3 节安全性条款
- **严重程度**：一般
- **改进建议**：（1）在 8.1 节 marked 引入处同步提及 DOMPurify 等 HTML 净化库；"marked 仅负责 Markdown→HTML 转换，HTML 净化由 DOMPurify 在渲染前处理"；（2）在 7.3 节安全性条款中增加前端渲染安全条目

- **问题描述**：GET /api/risk/history 响应描述过于模糊——使用"子集"+"等"字的笼统表述，实现者无法确定哪些字段包含在内（advice 长文本？matched_diabetes_type？）
- **所在位置**：第 6.3 节第 799 行
- **严重程度**：一般
- **改进建议**：（1）补充明确的 data 数组元素 JSON 结构（推荐字段：id、risk_score、risk_level、risk_level_label、matched_diabetes_type、created_at，不包含 advice 长文本）；（2）若需展示建议摘要，补充 advice_summary 字段

- **问题描述**：life_plans 表 plan_type 的 CHECK 约束含 'other' 枚举值，但全文无对应业务场景描述——实现者无法确定该值何时产生、前端如何渲染、打卡分析如何归类
- **所在位置**：第 5 节第 611 行
- **严重程度**：一般
- **改进建议**：（1）若为预留值则标注"预留值，当前版本不使用"；（2）若 Dify 工作流可能产生则补充业务说明；（3）若为设计错误则从 CHECK 约束中移除

- **问题描述**：Dify Agent Text2SQL 工具回调的 api_key 配置与 Express 环境变量 DIFY_SERVICE_API_KEY 形成跨平台密钥同步循环依赖——密钥轮换时需两侧同步更新，文档未说明同步机制和故障排查指引
- **所在位置**：第 5 节第 622-626 行，第 6.10 节第 1034-1036 行
- **严重程度**：一般
- **改进建议**：（1）增加部署注意事项——明确两端密钥需一致，轮换时需同步更新；（2）在 .env.example 模板中为 DIFY_SERVICE_API_KEY 添加注释提示；（3）补充密钥不同步时的故障排查指引

- **问题描述**：检验按钮"保持可点击"行为规定与 Vant 4 Form 组件默认行为（校验失败时按钮 disabled）矛盾，实现者面临两种实现路径的选择
- **所在位置**：第 7.1 节第 1211 行与第 8.1 节第 1310 行
- **严重程度**：轻微
- **改进建议**：（1）若保持可点击是设计意图，增加"以本规范为准"的覆盖声明；（2）若可接受 Vant 默认行为，修订规范为 disabled 态

- **问题描述**："重新填写"按钮清除 sessionStorage 的时序与页面刷新恢复策略之间存在空白——错误的清除时序可能导致旧数据残留回填
- **所在位置**：第 4.4 节第 259-265 行
- **严重程度**：轻微
- **改进建议**：补充清除时序约定——"先清除 sessionStorage，再重置 Pinia store"，并在 riskFormStore 示例中明确引用
