---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [obsidian, decision-graph, architecture, pkg]
---

# Obsidian As Decision Graph

## 问题
AI 容易重复讨论已经做过的架构选择，也容易忽略“为什么这样设计”的历史上下文。

## 上下文
适用于希望把项目的决策、原则、架构说明沉淀为人类可写、AI 可读的知识层。

## 结构
- 决策文档：记录采纳与否决方案
- 架构文档：记录系统拓扑、模块边界、数据流
- 双向链接：连接决策、模块、代码符号与运维知识
- 查询层：供 AI 与人检索

## 实施方式
1. 用 Markdown 文档记录关键决策与架构说明
2. 用统一 frontmatter 描述类型、状态、标签、日期
3. 通过链接把决策与模块、代码、原则关联起来
4. 把这些文档作为 PKG 的决策层和架构层输入

## 优点
- 人类编辑成本低
- 可视化链接关系清晰
- 很适合作为 Decision Graph 与部分 Architecture Graph 的载体

## 代价
- 不能替代 AST 级符号分析
- 若缺乏更新纪律，会与真实代码逐步偏离

## 不适用场景
- 需要精确函数调用图或自动代码依赖分析时

## 相关文档
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/unified-pkg-layer.md`
