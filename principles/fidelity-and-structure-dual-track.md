---
type: principle
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [fidelity, structure, ui, generation, stitch]
---

# Fidelity And Structure Dual Track

## 原则
在从 Stitch 或类似设计输入生成前端代码时，应把“高保真视觉复原”和“高可维护工程结构”视为两个不同目标，并采用双轨生成策略，而不是强行让单一表示层同时完美承担两者。

## 背景
实践中常出现两种极端：
- 只走 `Schema -> Code`，结构稳定但视觉还原度下降
- 只走 `Stitch -> LLM -> Code`，视觉保真很好但工程可维护性变差

问题不在于哪一条路完全错误，而在于两条路分别优化了不同目标。如果过早抽象设计输入，就会丢失高密度视觉信息；如果只追求视觉生成，又会失去长期工程演化能力。

## 适用场景
- Stitch / Figma / HTML 到前端代码生成
- 需要既保留设计还原度，又支持数据绑定、状态管理和 API 接入
- 需要后续重生成、跨端生成或组件化维护

## 核心要求
1. 高保真视觉信息不能在第一步就被过度抽象掉
2. 工程结构不能完全依赖视觉生成结果临时推断
3. 视觉层与工程层应分别建模，再在后续阶段对齐
4. LLM 的高保真能力应被利用，但不应直接等同于最终工程代码
5. Schema 应负责工程约束，而不是独自承担所有视觉保真责任

## 推荐做法
- 用一条轨道保留视觉真相，如 `Visual Spec`、布局树、样式 token、高保真草稿代码
- 用另一条轨道保留工程真相，如 `UI Schema`、数据源、动作、状态、资源与组件边界
- 在生成后期做一次对齐或重写，把视觉草稿与工程约束融合为最终代码

## 价值
- 兼顾页面“像不像”和代码“稳不稳”
- 降低因过度抽象导致的设计损失
- 降低因纯视觉生成导致的工程失控
- 更适合后续重生成、Mock API 接入与真实 API 替换

## 代价与边界
- 会增加中间产物和流程复杂度
- 需要维护视觉层与工程层之间的映射
- 小型一次性页面未必值得引入完整双轨体系

## 相关文档
- `knowledge/patterns/visual-spec-plus-ui-schema.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/methods/design-to-schema-to-code.md`
