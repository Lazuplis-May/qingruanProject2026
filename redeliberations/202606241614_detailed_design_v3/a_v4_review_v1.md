# 技术方案审查报告（v1）

## 审查结果

REJECTED

## 逐维度审查

### 1. 技术准确性

**[通过]** 问题1（严重）修复技术准确：第 7.3.3 节路由处理器伪代码（第 5255-5265 行）在 Dify Agent 回调路径中通过 `db.prepare('SELECT role FROM users WHERE id = ?').get(operatorId)` 动态查询操作者角色，并将 `getDatabase()` 调用上提至函数入口供身份解析与 SQL 执行共用。用户不存在时返回 403，逻辑正确。

**[通过]** 问题2（严重）修复技术准确：第 7.3.4 节新增 `validateRowLevelPermission` 技术规范，选用 `node-sql-parser`（npm 包，纯 JS 实现，支持 SQLite 方言）进行 AST 解析，不采用正则匹配。表分类校验规则（用户私有表/公共只读表/禁止访问表）清晰，fail-closed 原则（AST 解析失败一律返回 false）正确。伪代码含 `extractTableNames`、`containsUserIdConstraint`、`insertContainsUserId` 辅助函数说明，技术路径可行。

**[通过]** 问题5（一般）修复技术准确：第 5.2.1 节 Dify 工作流输出结构新增 `risk_level_label`（中文）和 `matched_diabetes_type`（中文）字段；新增第 5.2.1.1 节端到端字段映射契约表，覆盖"Dify 输出 → 后端映射 → 数据库存储 → SQLite 查询提取 → API 响应 → 前端渲染"整条链路。后端 `risk.js` 将 `risk_level_detail` 与 `suggestions` 拼接为 `advice` 字符串后连同其他字段序列化为 JSON 写入 `user_risk_info.result`，SQLite `json_extract` 提取键名与存储键名完全一致，契约闭环。

**[通过]** `difyAuth.js` 中间件（第 7.3.2 节）使用 SHA-256 哈希后再 `timingSafeEqual` 的方案正确，确保两个 Buffer 长度恒为 32 字节，防范长度差异导致的 DoS。

**[一般]** 问题3（一般）修复存在技术性矛盾：DDL（第 910 行）已将 `punch_in.punch_type` 的 CHECK 约束扩充为 `('diet', 'exercise', 'other')`，但 `mapper.js` 的 `punch_type` 映射字典（第 646-649 行）仍只包含 `'饮食' ↔ 'diet'`、`'运动' ↔ 'exercise'` 两个条目，缺少 `'其他' ↔ 'other'` 映射。当系统对 `'other'` 类型的方案项进行打卡时，`toEnglish('punch_type', '其他')` 将返回 `undefined`，导致 SQLite INSERT 因 CHECK 约束失败。修复在 DDL 层放开约束但在转换层未同步放开，形成技术性断裂。

**[轻微]** 问题6（轻微）修复存在类型契约不一致：第 3.2.16 节 JSON 响应示例（第 1819 行）已补齐 `remarks` 字段，但第 3.8.6 节 `PunchCreateResponse` TypeScript 接口定义（第 2613-2619 行）仍缺少 `remarks: string` 成员。前后端 TypeScript 契约与 JSON 示例不匹配。

### 2. 完备性

**[通过]** 问题1（严重）修复完备：动态查询用户角色的方案覆盖了 Dify Agent 回调路径下的所有场景，管理员经 `admin-manager-agent` 操作和普通用户经 `diabetes-assistant-agent` 操作均能正确获得对应角色权限。

**[通过]** 问题2（严重）修复完备：`validateRowLevelPermission` 技术规范覆盖了 SELECT/INSERT/UPDATE/DELETE 四种 SQL 类型的校验规则，表分类涵盖所有 10 张数据表（用户私有表 6 张、公共只读表 3 张、禁止访问表 1 张）。AST 解析方案能处理子查询、别名、嵌套条件等复杂形态。

**[通过]** 问题4（一般）修复完备：第 1.6.2 节路由守卫步骤5 对 `meta.requiresDisclaimer: true` 的路由（医师对话/生活方案/风险预测/健康建议）调用 `hasAcceptedDisclaimer()` 判定；第 4.4.4 节补充 `chatStore.toggleFab()` 打开 AI 助手弹窗前也需调用免责判定。路由守卫路径和非路由路径（FAB 弹窗）均已覆盖，合规控制流不再悬空。

**[通过]** 持续性问题1（风险预测数据契约）已彻底解决：第 5.2.1.1 节以表格形式显式列出端到端字段映射契约，每一跳键名严格对应，并在第 3.2.7 节和第 3.2.8 节补充字段来源标注，杜绝键名漂移。

**[通过]** 持续性问题2（SQLite 查询设计深度）已解决：第 3.2.8 节 SQLite 查询的 `json_extract` 提取键名与存储键名完全一致，并标注字段来源（JSON 提取 vs 表独立列 vs 实时计算）；第 3.2.17 节打卡列表查询含动态条件过滤和字段来源说明。

**[一般]** 问题3（一般）修复不完备：DDL 已扩充 `punch_in.punch_type` 约束为含 `'other'`，但以下层未同步更新，形成多处设计级矛盾：
1. **数据字典（第 1240 行）**：`punch_type` 字段约束仍标注为 `CHECK(punch_type IN ('diet', 'exercise'))`，业务含义仅说明 `diet=饮食, exercise=运动`，与 DDL 不一致。
2. **mapper.js 映射字典（第 646-649 行）**：缺少 `'其他' ↔ 'other'` 映射条目，导致转换函数对 `'其他'` 值返回 `undefined`。
3. **TypeScript 类型定义**：`PunchRecord.punch_type`（第 2509 行）、`PunchCreateRequest.punch_type`（第 2607 行）、`PunchListParams.punch_type`（第 2624 行）均标注为 `'饮食' | '运动'`，缺少 `'其他'` 联合类型成员。而 `LifePlan.plan_type`（第 2490 行）已包含 `'其他'`，两表类型定义不一致。
4. **控制器拦截策略描述（第 664 行）**：仍描述为"将 `punch_type`（'diet'/'exercise'）及 `completion_status`（'completed'/'uncompleted'）转换为英文"，未提及 `'other'`。
5. **life_plans 数据字典（第 1211 行）**：仍标注"当前版本仅使用'diet'和'exercise'两种类型，'other'为预留扩展值，当前版本前端不渲染此类型的方案项UI"。此描述与扩充 `punch_in.punch_type` 的目标矛盾——若 `'other'` 类型方案项在前端不渲染，则用户无法对其打卡，扩充 `punch_type` 约束便失去意义。

上述矛盾使实现者无法确定 `'other'` 类型方案项是否可打卡、mapper.js 是否需要处理 `'其他'` 转换、TypeScript 类型是否需要包含 `'其他'`，阻碍实现启动。

**[轻微]** 问题6（轻微）修复不完备：JSON 响应示例已补齐 `remarks`，但 TypeScript 接口 `PunchCreateResponse`（第 2613-2619 行）未同步更新，前端按 TypeScript 契约编码时无法访问 `remarks` 属性。

**[轻微]** 第 1238 行 punch_in 表数据字典 `plan_id` 字段业务含义存在笔误："关联 Rar 的方案项ID"应为"关联的方案项ID"或"关联 life_plans 的方案项ID"。

### 3. 可操作性

**[通过]** 问题1修复路径清晰：实现者可直接按伪代码实现动态角色查询，逻辑无歧义。

**[通过]** 问题2修复路径清晰：实现者可按技术规范选择 `node-sql-parser` 包，按表分类规则和伪代码实现 AST 校验。选型理由（正则无法处理子查询/别名/嵌套条件）充分。

**[通过]** 问题4修复路径清晰：路由守卫步骤5 和 `chatStore.toggleFab()` 调用点均有明确伪代码和文字说明，`useUI.ts` 的 `showDisclaimer()`/`hasAcceptedDisclaimer()` 函数签名完整。

**[通过]** 问题5修复路径清晰：端到端字段映射契约表使实现者能明确知道每一层的字段命名，后端 `risk.js` 映射逻辑有 5 步明确说明。

**[一般]** 问题3修复路径不清晰：DDL 与数据字典、mapper.js、TypeScript 类型、控制器策略描述、life_plans 数据字典说明之间存在多处矛盾，实现者无法确定应以哪一层为准。具体而言：
- DDL 允许 `'other'`，但 mapper.js 不处理 `'其他'` 转换——实现者不知是否应在 mapper.js 中补充 `'其他' ↔ 'other'`
- DDL 允许 `'other'`，但 life_plans 数据字典说 `'other'` 类型前端不渲染——实现者不知 `'other'` 类型方案项是否可打卡
- DDL 允许 `'other'`，但 TypeScript 类型不含 `'其他'`——实现者不知前端类型定义是否应包含 `'其他'`

**[轻微]** 问题6修复路径基本清晰：JSON 示例正确，但 TypeScript 接口未同步，实现者需自行判断以哪份为准。

## 修改要求（仅 REJECTED 时存在）

### 修改要求1：同步更新 `punch_type` 扩充至 `'other'` 的全链路一致性

- **问题**：DDL 已将 `punch_in.punch_type` 的 CHECK 约束扩充为 `('diet', 'exercise', 'other')`，但数据字典、mapper.js 映射字典、TypeScript 类型定义、控制器拦截策略描述、life_plans 数据字典说明均未同步更新，形成 5 处设计级矛盾。
- **原因**：实现者无法确定 `'other'` 类型方案项是否可打卡、转换层是否需要处理 `'其他'` 值、前端类型是否需要包含 `'其他'`，多处自相矛盾使实现者无法着手编码。
- **建议方向**：
  1. 第 2.5 节 punch_in 数据字典（第 1240 行）：将 `punch_type` 约束更新为 `CHECK(punch_type IN ('diet', 'exercise', 'other'))`，业务含义补充 `other=其他`。
  2. 第 1.8.1 节 mapper.js 的 `punch_type` 映射字典（第 646-649 行）：新增 `'其他': 'other'` 和 `'other': '其他'` 两个条目。
  3. 第 3.8.6 节 TypeScript 类型定义：`PunchRecord.punch_type`（第 2509 行）、`PunchCreateRequest.punch_type`（第 2607 行）、`PunchListParams.punch_type`（第 2624 行）的联合类型均补充 `'其他'` 成员，与 `LifePlan.plan_type` 保持一致。
  4. 第 1.8.3 节控制器拦截策略描述（第 664 行）：将 `punch_type` 枚举值列表更新为 `'diet'/'exercise'/'other'`。
  5. 第 2.5 节 life_plans 数据字典 `plan_type` 字段说明（第 1211 行）：删除"当前版本仅使用'diet'和'exercise'两种类型，'other'为预留扩展值，当前版本前端不渲染此类型的方案项UI"的描述，改为说明 `'other'` 类型方案项可正常打卡，与 `punch_in.punch_type` 约束扩充保持业务语义一致。

### 修改要求2：同步更新 `PunchCreateResponse` TypeScript 接口的 `remarks` 字段

- **问题**：第 3.2.16 节 JSON 响应示例已补齐 `remarks` 字段，但第 3.8.6 节 `PunchCreateResponse` TypeScript 接口（第 2613-2619 行）仍缺少 `remarks: string` 成员。
- **原因**：前端实现者按 TypeScript 接口编码时无法访问 `remarks` 属性，导致打卡成功后无法更新备注展示，与第 3.2.16 节 JSON 示例和第 3.2.17 节 `GET /api/punch/list` 返回结构不一致。
- **建议方向**：在 `PunchCreateResponse` 接口中新增 `remarks: string` 成员，与 JSON 响应示例和 `PunchRecord` 类型保持一致。

### 修改要求3（轻微）：修正 punch_in 数据字典笔误

- **问题**：第 1238 行 `plan_id` 字段业务含义为"关联 Rar 的方案项ID"，含不明字符串"Rar"。
- **原因**：笔误使实现者对字段含义产生歧义。
- **建议方向**：修改为"关联的方案项ID"或"关联 life_plans 的方案项ID"。
