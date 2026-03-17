---
type: principle
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [tdd, testing, quality, workflow, engineering]
---

# Test Driven Development

## 原则
在关键能力开发中，优先用测试定义行为边界和验收标准，再实现代码，以提高系统可验证性、可回归性和长期演化稳定性。

## 背景
对 AI 平台、Agent 平台和自动化编程系统来说，很多能力不仅要“能跑”，还要：
- 可重复验证
- 可防回归
- 可解释失败原因
- 可支撑多轮重构

如果没有测试先行，常见后果是：
- 行为边界不清晰
- 需求被实现细节替代
- 回归保护不足
- Agent / Workflow / Policy 这类关键能力越改越脆弱

## 适用场景
- Skill / Workflow / Policy / Billing / Audit 等核心能力开发
- 需要长期维护的 AI SaaS 平台
- 自动化编程系统与 Agent 系统

## 核心要求
1. 先定义预期行为，再写实现
2. 测试应覆盖关键路径，而不只是覆盖代码行
3. 回归测试应保护已确认的系统行为
4. 测试不仅验证结果，也应验证边界和失败模式
5. 对协议、状态机、策略判断等高风险能力，优先建立契约测试

## 在当前系统中的含义
在当前知识体系中，TDD 尤其适用于：
- `Supervisor Task Payload` / `Cursor Result Payload` 协议
- `Runtime State Schema` 与状态机流转
- `Supervisor Review Rules`
- `Reconcile` 规则和 warning 分类
- `Policy Engine` 的 allow / deny 行为

## 价值
- 让方法论更容易落到真实实现
- 降低长链路系统的回归风险
- 让 Agent 平台的行为更可预测
- 让“重构”变得更安全

## 代价与边界
- 初期开发速度可能变慢
- 如果测试只覆盖表面，不覆盖行为，收益有限
- 对探索性原型阶段，可适当降低 TDD 强度

## 推荐做法
- 协议类能力优先写契约测试
- 状态机优先写状态迁移测试
- Policy 优先写 allow / deny 用例
- Workflow 优先写关键路径与失败恢复测试
- Reconcile 优先写组件级输入输出测试

## 相关文档
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/patterns/runtime-state-schema.md`
- `knowledge/patterns/supervisor-review-rules.md`
