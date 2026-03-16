---
type: principle
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [schema, codegen, ui, architecture]
---

# Schema First Generation

## 原则
在自动化页面生成与多端代码生成场景中，优先以 Schema 作为生成边界，而不是直接在设计产物与目标代码之间做硬转译。

## 背景
当系统需要从设计稿、HTML、设计节点数据或 AI 生成结果继续演化为 Taro、React、Web 或其他前端页面时，直接转译虽然起步快，但会在维护、重生成、数据接入和跨端复用上迅速失控。

## 适用场景
- Design to Code
- HTML to Taro / React
- AI 自动生成页面
- 需要先接 Mock API、后接真实 API 的前端系统
- 需要跨端生成或重复生成的场景

## 实践要求
1. 在设计输入与目标代码之间建立明确的 `UI Schema` 或 `Page Schema`
2. Schema 至少表达组件树、数据位、动作、资源引用与页面元信息
3. 页面层依赖 Schema 生成结果，而不是依赖某一种设计工具的私有结构
4. 数据需求应尽可能从 Schema 中显式提取，以支持 Mock API 与真实 API 对接
5. 生成器、校验器、Mock 层应围绕 Schema 组织，而不是围绕 HTML 片段组织

## 价值
- 让生成过程可维护、可重生成、可测试
- 让 UI、数据、交互具备统一边界
- 为多端生成和后续 Agent 自动化留出稳定接口

## 代价与边界
- 需要额外维护 Schema 规范与版本
- 对语义缺失的设计输入，仍需要人工补充
- 静态一次性页面不一定值得引入完整 Schema 层

## 相关文档
- `knowledge/decisions/accepted/002-use-ui-schema-as-generation-boundary.md`
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/patterns/html-to-ui-schema.md`
