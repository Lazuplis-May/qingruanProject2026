# 技术方案审查报告（v1）

## 审查结果

REJECTED

## 逐维度审查

### 1. 技术准确性
- **[通过]** 本方案对 `difyAuth.js` 中使用 Node.js 原生的 `crypto.timingSafeEqual` 进行校验的安全隐患做出了正确的修复。更新后的伪代码中，通过在比较前使用 SHA-256 算法对输入的 `api_key` 和预期的 `DIFY_SERVICE_API_KEY` 分别进行哈希计算，生成固定长度（32 字节）的 Buffer 后再进行 `crypto.timingSafeEqual` 校验，保证了输入 Buffer 长度的始终一致，消除了拒绝服务（DoS）的崩溃隐患。
- **[通过]** 规范了前端路由文件 `router/index.ts` 的定义结构，去除了上一轮设计中存在的嵌套语法和标点符号混淆错误，确保了前端编译与 Vite 构建路径的可行性。
- **[通过]** 数据库建模具有一致性。将 `users` 表的 `password_changed` 物理类型和 `user_risk_info` 表的 `pregnancy` 物理类型统一调整为了 `INTEGER`，并增加了相应的 `CHECK(pregnancy IN (0, 1) OR pregnancy IS NULL)` 和 `CHECK(password_changed IN (0, 1))` 约束，避免了隐式类型转换问题。
- **[严重]** 数据双向转换层 `mapper.js` 的映射字典代码存在遗漏。虽然在第 5939 行更新说明中声称“在双向转换层 `mapper.js` 的 `MAPPINGS` 字典中，补全了 `punch_type` 字段的映射定义（'饮食' ↔ 'diet'，'运动' ↔ 'exercise'）”，但第 1.8.1 节实际代码的 `MAPPINGS` 字典中，依然未包含 `punch_type` 字段的定义。由于打卡接口会在落库和展示前经过双向转换逻辑，若该字典缺失此规则，将直接导致转换逻辑返回 `undefined` 并使打卡记录落库失败。

### 2. 完备性
- **[通过]** 方案补充了针对管理员首次登录时在不启动后端服务、通过命令行或第三方工具直接/离线导入 `seed.sql` 导致管理员默认密码占位符未被替换的风险警告提示。
- **[一般]** `POST /api/risk/predict` 及相关接口响应字段存在严重的前后不一致与契约偏差：
  1. 在第 3.2.7 节（接口设计）中，API 的正常响应数据（200）依然为旧格式，返回的字段如：`risk_level`（中文，例如"高风险"，而非英文枚举 `'low'|'medium'|'high'`）、`risk_level_detail`、`diabetes_type` 和 `suggestions` 等，且缺失了关键的 `risk_level_label`（中文标签）。
  2. 而在第 3.8.4 节（TypeScript 类型定义）中，TypeScript 类型 `RiskPredictResponse` 则正确定义了 `record_id`、`risk_score`、`risk_level: 'low' | 'medium' | 'high'`、`risk_level_label: '低风险' | '中风险' | '高风险'`、`matched_diabetes_type` 和 `advice`。
  上述两处定义严重冲突，且接口设计处（3.2.7节和3.2.8节历史接口）直接违背了需求文档中对预测响应字段的契约规定，会导致前端渲染和状态解析失败。
- **[严重]** 核心数据查询 SQL 语句缺失。虽然在第 5959 行更新说明中声称“针对数据驱动的端点 `GET /api/risk/history` 和 `GET /api/punch/list` 补充了详细的 SQLite 查询设计伪代码”，但全文检索除此更新历史行外，完全没有这两个接口对应的 SQLite SQL 查询语句，无法直接指导后端开发。

### 3. 可操作性
- **[严重]** 由于 `MAPPINGS` 转换字典的实际代码遗漏了打卡字段，`POST /api/risk/predict` 返回体在接口声明与 TypeScript 契约中存在字段命名及类型矛盾，且 `GET /api/risk/history` 和 `GET /api/punch/list` 等核心数据读取接口缺失了具体的 SQLite 查询语句，导致后续实现者在按照文档编码时将遇到严重的阻碍，无法实现功能逻辑的正常流转与数据的准确查询。

## 修改要求

- **问题**：数据转换字典 `MAPPINGS` 遗漏 `punch_type` 代码。
  - **原因**：落库前无法正确将打卡类型（'饮食'/'运动'）转换为英文，导致数据库 CHECK 约束失败；读取数据时亦无法正常还原为中文。
  - **建议方向**：在第 1.8.1 节 `MAPPINGS` 对象定义中，补充 `'punch_type': { '饮食': 'diet', '运动': 'exercise', 'diet': '饮食', 'exercise': '运动' }` 映射对。

- **问题**：`POST /api/risk/predict` 及 `GET /api/risk/history` 响应体结构与 TypeScript 类型不一致且不符合原始需求。
  - **原因**：违反前后端契约，阻碍前端渲染，并导致前端 TypeScript 编译报错。
  - **建议方向**：重构第 3.2.7 节及第 3.2.8 节中响应 JSON 示例，将 `risk_level`（中文值）、`risk_level_detail`、`suggestions` 和 `diabetes_type` 字段全部纠正为 `risk_level`（英文枚举 `'low'|'medium'|'high'`）、`risk_level_label`（中文标签）、`matched_diabetes_type` 和 `advice`（Markdown 建议文本），使其与需求及 3.8.4 节 TypeScript interface 保持完全一致。

- **问题**：缺乏对核心数据端点（`GET /api/risk/history` 和 `GET /api/punch/list`）的 SQLite 查询设计。
  - **原因**：使实现者无法从方案中获取如何计算 BMI、如何从 `result` 的 JSON 列中提取数据、以及如何处理打卡日期范围和类型的过滤与分页。
  - **建议方向**：在第 3.2.8 节和第 3.2.17 节中，补充对应的 SQLite 查询设计伪代码。对于 `GET /api/risk/history` 接口，提供实时计算 BMI（`ROUND(...)`）及利用 SQLite 的 `json_extract()` 提取 `result` JSON 属性的分页 SQL 示例；对于 `GET /api/punch/list` 接口，提供包含 `startDate`、`endDate`、`punch_type` 过滤条件及分页的 SQL 示例。
