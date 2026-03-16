---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [pkg, graph, obsidian, symbol-graph]
---

# Unified PKG Layer

## 问题
文件图、符号图、架构文档和决策文档分散在不同系统里时，Agent 很难做跨层查询与稳定推理。

## 上下文
适用于希望同时利用代码解析结果与文档图谱的项目，例如用符号分析工具处理代码、用 Obsidian 维护架构与决策说明。

## 结构
- 代码输入：文件关系、符号关系、调用关系
- 文档输入：架构说明、原则、决策、模块笔记
- 统一图层：节点与边的标准化模型
- Agent 查询层：对外暴露统一检索接口

## 实施方式
1. 定义统一节点类型，如 `File`、`Function`、`Module`、`Service`、`DBTable`、`Document`
2. 定义统一关系，如 `IMPORTS`、`CALLS`、`USES`、`READS`、`DESCRIBES`
3. 把符号层与文档层分别导入，再建立跨层链接
4. 要求 Agent 先查图再编码，尤其在多模块任务中

## 优点
- 支持跨层推理
- 降低“只懂代码”或“只懂文档”的单侧偏差
- 更适合长期项目维护

## 代价
- 需要额外维护图谱同步逻辑
- 初期建模成本高于单纯索引代码

## 不适用场景
- 小型仓库或一次性脚本项目

## 相关文档
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/obsidian-as-decision-graph.md`
