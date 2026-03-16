---
type: decision
status: accepted
owner: user
date: 2026-03-16
tags: [obsidian, pkg, decision-graph, architecture]
---

# 003-use-obsidian-for-decision-and-architecture

## 决策
在 Project Knowledge Graph 体系中，采用 Obsidian 作为 `Decision Graph` 和主要 `Architecture Graph` 的人类维护界面，而不是让它承担符号层分析职责。

## 背景
当前希望把项目知识拆分为多个层次：符号层、模块层、架构层、决策层。讨论中明确指出，Obsidian 擅长记录决策、架构说明、原则和链接关系，但不适合承担 AST 级别的函数调用、类型引用和依赖分析。

## 备选方案
- 方案 A：让 Obsidian 同时承担决策层、架构层和符号层
- 方案 B：只用符号分析工具，不维护文档型架构与决策层
- 方案 C：Obsidian 负责决策层和架构层，符号层交给专门工具，再通过统一图层聚合

## 选择原因
Obsidian 具备 Markdown、双向链接、图视图和低编辑成本，适合人类持续记录“为什么这样设计”。但符号层需要 AST、调用图、引用图等机器分析能力，因此更合理的边界是：Obsidian 负责人类知识层，符号分析工具负责代码结构层，再通过统一图层连接起来。

## 收益
- 保留人类决策与架构知识
- 避免把 Obsidian 用在不擅长的精确代码分析上
- 为 Agent 提供“代码结构 + 设计原因”两类上下文
- 更适合长期维护项目认知系统

## 代价与风险
- 需要维护文档层与符号层之间的同步关系
- 如果缺少统一图层，知识仍会分裂
- 如果文档不持续更新，决策层会过时

## 后续动作
- 设计统一图层的节点和边模型
- 增加用于记录架构、原则、决策的文档模板
- 明确符号层工具与文档层的同步策略

## 相关文档
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/obsidian-as-decision-graph.md`
- `knowledge/patterns/unified-pkg-layer.md`
- `knowledge/inbox/2026-03-16-ai-software-factory-notes.md`
