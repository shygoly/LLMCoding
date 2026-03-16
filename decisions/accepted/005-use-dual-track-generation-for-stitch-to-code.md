---
type: decision
status: accepted
owner: user
date: 2026-03-16
tags: [stitch, generation, fidelity, schema, frontend]
---

# 005-use-dual-track-generation-for-stitch-to-code

## 决策
在 `Stitch -> 前端代码` 的生成链路中，采用“双轨生成”策略：
- 一条轨道负责高保真视觉复原
- 一条轨道负责工程结构建模
- 最终通过对齐 / 重写阶段合并为可维护代码

## 背景
实践表明：
- 直接从 Stitch 让 LLM 生成前端代码，视觉保真度高
- 直接从 Schema 生成前端代码，工程结构更稳定

但单独使用任一路径都会牺牲另一类目标。因此需要把“页面像不像”和“代码稳不稳”分开处理，而不是让单一路径独自承担全部责任。

## 备选方案
- 方案 A：只用 `UI Schema -> Code`
- 方案 B：只用 `Stitch -> LLM -> Final Code`
- 方案 C：采用双轨生成，后期对齐融合

## 选择原因
双轨生成可以同时保留：
- Stitch 输入中的视觉细节
- Schema 输入中的工程边界、数据结构、动作语义和长期维护能力

相比单一路径，这种方式更适合建设长期可演化的自动化生成系统。

## 收益
- 提高页面对原始设计的还原度
- 保留 Schema 驱动的工程可维护性
- 更适合后续接 Mock API、真实 API 和状态管理
- 为 LLM 与生成器各自发挥优势提供明确边界

## 代价与风险
- 需要额外维护 `Visual Spec` 或视觉层中间表示
- 需要实现对齐 / 重写逻辑
- 流程比单一路径更复杂，需要控制中间产物一致性

## 后续动作
- 增加 `Visual Spec` 的最小字段规范
- 设计 `Reconcile` 阶段的输入输出协议
- 为高保真草稿与工程骨架建立对齐策略

## 相关文档
- `knowledge/principles/fidelity-and-structure-dual-track.md`
- `knowledge/patterns/visual-spec-plus-ui-schema.md`
- `knowledge/decisions/accepted/002-use-ui-schema-as-generation-boundary.md`
- `knowledge/patterns/ui-schema-spec.md`
