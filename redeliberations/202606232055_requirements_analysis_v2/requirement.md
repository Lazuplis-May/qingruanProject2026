# 再审议需求指令

对 `docs/1_requirements_analysis_v1.md`（当前 SRS v1）进行修订，产出 `docs/1_requirements_analysis_v2.md`（SRS v2）。

## 核心修订指令

### 1. 明确前端技术栈为 Vue3 + TypeScript

全文统一前端技术栈描述，消除所有与 Vue3 + TypeScript 矛盾的描述：

- **第 1 节项目定位**：将"平台以 HTML、CSS、JavaScript 为前端技术栈"修改为"平台以 **Vue 3 + TypeScript** 为前端技术栈，使用 **Vite** 作为构建工具"
- **第 8.1 节在范围内**：删除"采用原生 HTML/CSS/JavaScript 开发，不依赖构建工具"等与 Vue3 + Vite 矛盾的技术选型描述；删除"第三方库通过 CDN 或本地 lib 目录引入"的描述，改为 npm 包管理方式
- **第 8.1 节目录结构**：更新为 Vue3 + Vite + TypeScript 项目的标准目录结构（src/views/、src/components/、src/router/、src/stores/、src/api/、src/types/、vite.config.ts、tsconfig.json 等）
- **第 16.14 条修订说明**（如存在）：若其中引用了"原生 HTML/CSS/JavaScript"，需同步修正为 Vue3 + TypeScript

### 2. 统一所有代码示例为 TypeScript

- 所有 JavaScript 代码示例统一为 TypeScript（添加类型注解）
- 前端文件扩展名统一为 `.ts` / `.vue`
- Vue Router 配置使用 TypeScript 语法
- Pinia Store 使用 TypeScript 泛型

### 3. 消除所有内部技术矛盾

- 全文不应再出现与 Vue3 + TypeScript + Vite 技术栈相矛盾的任何描述
- 确保 iframe 架构（如仍保留）与 Vue3 SPA 架构之间的描述一致
- 确保所有对前端构建、模块加载、第三方库引入方式的描述与 Vite + npm 生态一致

## 约束条件

- 除上述技术栈修订外，保持文档其余内容不变（功能需求、API 规格、数据模型、非功能需求等）
- 修订说明章节需新增本轮（v1 → v2）的修订记录
- 最终产出文件路径：docs/1_requirements_analysis_v2.md
