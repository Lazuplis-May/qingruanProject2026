# 再审议需求指令

对 `docs/2_detailed_design_v1.md`（当前详细设计 v1）进行修订，产出 `docs/2_detailed_design_v2.md`（详细设计 v2）。

## 核心修订指令

### 1. 与 SRS v2 对齐前端技术栈

详细设计 v1 基于旧版 SRS v1 编写，前端架构为 iframe + Hash路由 + postMessage。SRS v2 已明确前端技术栈为 **Vue 3 + TypeScript + Vite**，采用 Vue Router 4（history 模式）管理路由，Pinia 管理状态。详细设计 v2 必须与 SRS v2 完全对齐：

- **第 1 节系统架构图**：将 iframe 容器 + .html 子页面架构替换为 Vue3 SPA 架构图（Vue Router + 组件树）
- **第 1.2 节 iframe SPA主框架架构图**：整体替换为 Vue3 SPA 前端架构图（路由树 + 组件层级 + Pinia Store 体系）
- **Hash路由管理器**：全部替换为 Vue Router 4 history 路由模式
- **postMessage 消息总线**：全部替换为 Pinia Store 跨组件通信
- **所有 .html 页面引用**：替换为 .vue 组件引用（views/ 目录）

### 2. 消除所有与 SRS v2 矛盾的技术描述

- 全文不应再出现与 Vue3 + TypeScript + Vite 技术栈相矛盾的任何描述
- 所有 iframe 架构描述、Hash路由描述、postMessage 通信协议描述必须删除或替换
- 所有前端文件扩展名统一为 `.ts` / `.vue`
- 静态资源引用路径从 `/pages/`、`/src/` 调整为 Vite 构建产物路径

### 3. 保留非前端架构的内容

- 后端 Express API 设计、数据库设计、Dify 工作流设计、部署架构等非前端架构内容原则上保留
- 仅当这些内容与 Vue3 SPA 架构存在交叉引用矛盾时才需修订

## 参考文件

- SRS v2：`docs/1_requirements_analysis_v2.md`（前端技术栈的权威来源）
- SRS v1：`docs/1_requirements_analysis_v1.md`（原始 SRS，供对照）

## 约束条件

- 除上述技术栈修订外，尽量保持文档其余内容不变
- 修订说明章节需新增本轮（v1 → v2）的修订记录
- 最终产出文件路径：docs/2_detailed_design_v2.md
