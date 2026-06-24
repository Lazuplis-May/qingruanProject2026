# 再审议判定报告（v3）

## 判定结果

RETRY

## 判定理由

根据对组件B诊断报告与质询报告的评估：
1. 组件B在内部循环的第3轮（最大轮次为12轮）达成了明确的审查结论“LOCATED”，对审查报告予以确认，未出现循环耗尽未定位的情况。
2. 诊断报告共指出了8项质量问题，其中包含：
   - 3项严重（Major）问题：`difyAuth.js` 中间件时序安全比对导致的拒绝服务漏洞、前端路由配置文件语法错误导致构建失败、`mapper.js` 数据双向转换层漏配 `punch_type` 字段映射。
   - 2项一般（Medium）问题：`POST /api/risk/predict` 接口响应结构与用户需求契约不一致、`pregnancy` 字段在数据库物理设计和代码规范中数据类型不一致。
   - 3项轻微或优化级别（Minor / Optimization）问题：管理员密码占位符的命令行导入风险提示、数据库中布尔逻辑字段物理表示不一致、部分数据驱动接口缺失具体 SQL 查询。

由于诊断报告中包含多项严重和一般等级的问题，根据判定标准，判定结果为 **RETRY**（重新运行），组件A需要对这些问题进行修复和优化。

## 需要解决的问题（仅 RETRY 时存在）

- **问题描述**：Dify API Key 校验中间件（`difyAuth.js`）在使用 `crypto.timingSafeEqual` 进行 API Key 的时序安全比对时，未提前校验两个 Buffer 长度，若客户端请求携带的 API Key 长度与环境变量中的不一致，将抛出 `TypeError` 异常导致 Node.js 服务端崩溃或返回 500 错误，存在拒绝服务（DoS）隐患。
- **所在位置**：第 7.3.2 节 `difyAuth.js` 中间件行为伪代码（第 5087-5090 行）
- **严重程度**：严重
- **改进建议**：在比对前先检查长度，或者通过 SHA-256 哈希将任意长度的密钥输入转换为固定长度 Buffer 后再使用 `crypto.timingSafeEqual` 进行比对。

- **问题描述**：前端路由配置文件（`router/index.ts`）存在语法嵌套错误和标点符号混淆，如将路由定义错误写为 `routes: const routes: RouteRecordRaw[] = [`，且在路由数组结束时写成 `];,` 导致 Vite 构建与编译失败。
- **所在位置**：第 1.6.2 节 `router/index.ts` 伪代码（第 434 行、第 506-507 行）
- **严重程度**：严重
- **改进建议**：规范前端路由的定义结构，单独声明 `routes` 数组，然后再传入 `createRouter` 实例化。

- **问题描述**：数据双向转换层（`mapper.js`）的 `MAPPINGS` 字典中漏配了打卡类型 `punch_type` 的映射（'饮食' ↔ 'diet', '运动' ↔ 'exercise'），导致打卡记录落库时触发 SQLite 的 `CHECK` 约束校验失败，打卡功能失效。
- **所在位置**：第 1.8.1 节转换映射字典定义（第 597-624 行）与第 1.8.3 节控制器拦截策略说明（第 633 行）
- **严重程度**：严重
- **改进建议**：在 `server/utils/mapper.js` 的 `MAPPINGS` 对象中补全 `punch_type` 的双向转换映射。

- **问题描述**：`POST /api/risk/predict` 接口响应结构与用户需求契约存在偏差，字段定义混淆且缺少 `risk_level_label`，同时将 `matched_diabetes_type` 命名为了 `diabetes_type`，并将 `advice` 拆分为了 `risk_level_detail` 和 `suggestions`。
- **所在位置**：第 3.2.7 节 `POST /api/risk/predict` 响应规范（第 1520-1534 行）与第 3.8.4 节 TypeScript 预测响应接口定义（第 2452-2460 行）
- **严重程度**：一般
- **改进建议**：修改响应结构和 TypeScript 类型定义 `RiskPredictResponse`，完全对齐需求文档（包含 `record_id`、`risk_score`、`risk_level`、`risk_level_label`、`matched_diabetes_type`、`advice` 且 advice 为 Markdown 文本）。

- **问题描述**：`user_risk_info` 表中的 `pregnancy`（妊娠状态）字段在 SQLite 物理 DDL 和数据字典中定义为 `TEXT` 类型，但在 TypeScript 接口和 Dify 配置中定义为 `boolean` 类型，且漏配转换处理，导致数据一致性与写入风险。
- **所在位置**：第 2.2 节 DDL 语句中 `user_risk_info` 表定义（第 838 行）与第 2.5 节数据字典说明（第 1162 行）
- **严重程度**：一般
- **改进建议**：将 `pregnancy` 的物理类型修改为 `INTEGER` 并增加 `CHECK(pregnancy IN (0, 1) OR pregnancy IS NULL)` 约束，且在后端数据库操作中自动进行 Boolean ↔ Integer (0/1) 的映射转换。

- **问题描述**：初始数据脚本 `seed.sql` 包含管理员密码哈希的占位符，如果开发者直接通过外部命令行或数据库图形化管理工具导入 `seed.sql`（不经过 `initDatabase()` 的替换逻辑），将导致数据库直接存入占位符，从而造成管理员无法登录。
- **所在位置**：第 2.4 节管理员插入部分（第 952 行）与第 6.4 节数据库初始化逻辑（第 4825-4832 行）
- **严重程度**：轻微
- **改进建议**：在 `seed.sql` 中或设计文档相应章节补充风险提示，提醒开发者避免直接脱离后端环境导入执行 `seed.sql`。

- **问题描述**：数据库中布尔逻辑字段的表现形式不统一，`is_active` 使用 `INTEGER` 存储，`password_changed` 使用 `TEXT` 存储，`pregnancy` 也使用 `TEXT` 且无约束，这增加了开发和维护时的认知负担，且易引发映射错误。
- **所在位置**：第 2.2 节完整 DDL 语句中相关表定义（第 777, 838, 856 行）
- **严重程度**：轻微
- **改进建议**：在 SQLite 数据库物理建模中，统一将所有布尔属性字段声明为 `INTEGER` 类型，并增加 `CHECK(field IN (0, 1))` 约束。

- **问题描述**：除 `GET /api/plan/current` 以外，对于其他核心数据驱动接口如 `GET /api/risk/history` 和 `GET /api/punch/list`，文档中均缺乏具体能指导后端研发人员编写的 SQLite 查询设计。
- **所在位置**：第 3.2.8 节与第 3.2.17 节详细设计
- **严重程度**：轻微
- **改进建议**：为这些数据驱动的接口补充对应的 SQLite 查询伪代码（如对分页、条件过滤等进行示范）。
