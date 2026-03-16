---
type: review
status: draft
period: weekly
created: 2026-03-16
updated: 2026-03-16
tags: [example, weekly-review]
---

# 2026-03-16 Weekly Review Example

## 回顾范围
本次回顾覆盖 `knowledge/` 初始搭建阶段，重点关注自动化页面生成、PKG、Agent 工作方式与知识组织方法。

## 本期新增
- `knowledge/methods/design-to-schema-to-code.md`
- `knowledge/methods/project-knowledge-graph.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/principles/schema-first-generation.md`
- `knowledge/principles/agent-query-before-code.md`
- `knowledge/prompts/pre-code-query-checklist.md`

## 本期修订
- `knowledge/README.md`：补充 `principles/`、推荐起步文档与 prompt 索引
- `knowledge/templates/review-template.md`：从极简模板升级为可执行回顾模板

## 已验证结论
- 使用 `knowledge/` 分层目录沉淀内容，比直接堆聊天记录更容易持续维护
- `Schema First` 与 `Query Before Code` 可以分别作为 UI 生成与 Agent 工作的一级原则

## 待验证假设
- `UI Schema Spec` 是否足以支持第一版 Taro 页面生成器
- `Obsidian + Symbol Graph + Unified Layer` 是否值得投入完整实现

## 冲突与重复
- 目前 `patterns/html-to-ui-schema.md` 与 `patterns/ui-schema-spec.md` 有一定重叠，后续需明确前者偏“架构模式”，后者偏“规范说明”

## 失效内容
- 暂无明确失效内容

## 决策建议
- 可考虑把 `Agent Query Before Code` 升级为正式 decision 或执行规范
- 可考虑新增关于 `Mock API -> Real API` 切换边界的 decision

## 缺口清单
- 缺少 `UI Schema` 实例文档
- 缺少 `inbox -> knowledge` 自动拆分 prompt
- 缺少月度 review 样例

## 下一步
- 增加一个完整 `UI Schema` 页面示例
- 增加 inbox 拆分 prompt
- 增加一次 monthly review 模板样例

## 相关文档
- `knowledge/README.md`
- `knowledge/patterns/ui-schema-spec.md`
- `knowledge/principles/agent-query-before-code.md`
