---
type: inbox
status: raw
source: 2026-03-16 对话整理
created: 2026-03-16
tags: [ai-software-factory, pkg, ui-schema, taro, supabase, obsidian]
---

# AI Software Factory Notes

## 原始内容
本条目用于收集今天关于自动化编程架构的原始讨论，作为后续提炼 `method`、`pattern` 与 `decision` 的来源。

核心主题包括：

### 1. Stitch / HTML / Taro 自动化生成链路
- 推荐链路不是直接 `HTML -> Taro`
- 更稳妥的链路是 `Design -> HTML -> UI Schema / Page Schema -> Code Generator -> Taro`
- Schema 是后续接 Mock API、真实 API、页面重生成的关键中间层
- 页面数据需求可以反向生成 Mock API
- 后期可从 Mock API 平滑切换到 Supabase

### 2. 自动化页面系统的工程结构
- 多端项目可以拆分为 `apps` 与 `packages`
- 生成器、Schema、API SDK 可以放在共享包中
- 图片资源可能需要上传到对象存储，例如 Supabase Storage
- MCP 可承担“读取设计结果、解析结构、生成 Schema、触发代码生成”的角色

### 3. Project Knowledge Graph 的核心价值
- 普通 AI coding 的问题是每次都在重新理解项目
- PKG 的目标是让 Agent 持续理解项目，而不是冷启动读代码文本
- 可分为文件层、符号层、模块层、架构层、运行层、决策层
- 决策层被认为特别重要，因为它回答“为什么这样设计”

### 4. Obsidian 与 Symbol Graph 工具的协作
- Obsidian 适合作为决策图与部分架构图的载体
- 符号分析工具更适合维护函数、类型、调用关系
- 两者直接并列会造成 Graph 分裂
- 更合理的做法是引入 `Unified Graph Layer`
- 统一图层需要标准化节点和边，如 `Function`、`Module`、`DBTable`、`DESCRIBES`、`CALLS`、`READS`

### 5. 知识沉淀方式
- 原始对话不应直接视为最终结论
- 应按照 `idea -> method -> pattern -> decision` 的流程逐步加工
- 应长期维护一个知识目录，而不是把架构判断散落在聊天记录中

## 初步判断
这批材料可拆为以下类型：
- method：`design-to-schema-to-code`、`project-knowledge-graph`
- pattern：`html-to-ui-schema`、`mock-api-to-supabase`、`obsidian-as-decision-graph`、`unified-pkg-layer`
- decision：是否明确采用 `knowledge/` 作为沉淀主目录；是否采用 “Schema 中间层” 作为页面生成标准链路

## 待提炼问题
- 是否把 `HTML -> UI Schema -> Taro` 升级为正式架构决策
- 是否把 `Obsidian 负责决策层 / 架构层` 升级为正式决策
- 是否需要增加 `principles/` 目录，用于记录通用工程原则
- 是否需要给 `prompts/` 增加“从 inbox 自动拆分多篇文档”的提示模板

## 关联文档
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/html-to-ui-schema.md`
- `knowledge/patterns/obsidian-as-decision-graph.md`
- `knowledge/patterns/unified-pkg-layer.md`
