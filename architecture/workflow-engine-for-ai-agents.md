---
type: architecture
status: draft
owner: user
created: 2026-03-17
updated: 2026-03-17
tags: [workflow, agents, orchestration, temporal, ai-saas]
---

# Workflow Engine For AI Agents

## 目标
定义 `Workflow Engine` 在 AI Agent 系统中的位置和作用，解释为什么当系统从单步推理走向多步骤、长时任务时，必须把编排与恢复能力提升为独立架构层。

## 适用场景
- 长任务 AI Agent
- 多步骤 Skill 执行
- 需要重试、等待、恢复、人工介入的流程
- AI SaaS 平台与 Agent 平台

## 核心判断
当 AI 系统只是一次问答时，`Prompt + LLM + Tool` 可能够用。

但当系统开始处理：
- 多步骤任务
- 长时间运行任务
- 异步任务
- 可恢复任务
- 需要人工介入的任务

就必须引入 `Workflow Engine`。

原因很简单：

```text
LLM 适合推理
Workflow Engine 适合推进流程
```

## 为什么不能只靠 Planner
Planner 擅长：
- 理解目标
- 拆解步骤
- 生成计划

但 Planner 不擅长：
- 稳定保存任务状态
- 长时间等待外部结果
- 处理重试与超时
- 支持任务恢复
- 记录可审计的流程执行轨迹

因此：
- Planner 负责“想怎么做”
- Workflow Engine 负责“让它持续做完”

## Workflow Engine 的核心职责

### 1. 任务编排
把多步骤任务按顺序、条件或并行关系执行。

### 2. 状态持久化
在每一步之间保存状态，确保任务可恢复。

### 3. 重试与超时
统一处理外部 API 失败、超时和重试策略。

### 4. 人工介入
支持在关键节点暂停，等待人工确认或补充信息。

### 5. 审计与回放
记录执行轨迹，使任务过程可回放、可排查。

## 在 AI 系统中的推荐位置
推荐结构：

```text
User
-> Planner
-> DSL / Plan
-> Policy Engine
-> Workflow Engine
-> Skill Executor / cursor-agent / services
```

关键点：
- Workflow Engine 不替代 Planner
- Workflow Engine 不直接决定业务意图
- Workflow Engine 负责把计划变成稳定执行过程

## 典型任务示例

### 示例 1：文献调研
```text
收集论文
-> 去重
-> 摘要生成
-> 结构化报告
-> 人工复核
-> 发送结果
```

### 示例 2：代码任务
```text
解析 spec
-> 派发 task
-> cursor-agent 执行
-> review
-> needs_fix 重试
-> 知识回写
```

### 示例 3：医疗流程
```text
读取患者资料
-> 检查禁忌症
-> 检查交互作用
-> 生成建议
-> 医生确认
-> 输出结论
```

这些都不是单步调用，而是 workflow。

## Workflow Engine 负责什么，不负责什么

### 负责
- 任务状态推进
- 重试、超时、等待、恢复
- 条件分支与并行执行
- 执行轨迹记录

### 不负责
- 自己创造业务目标
- 替代 Planner 进行复杂语义推理
- 绕过 Policy Engine 直接放行高风险行为

## 为什么这层对 AI SaaS 尤其重要
SaaS 平台往往意味着：
- 多租户
- 多用户并发
- 长任务持续存在
- 失败后需要恢复
- 平台要可审计和可计费

没有 Workflow Engine 时，常见后果是：
- 任务中途丢状态
- 异步任务难管理
- 人工介入节点无处落地
- 重试逻辑散落在各个 Skill 中
- 系统越来越不可维护

## 与 Event Bus 的关系
两者可以配合，但不是同一层：

- `Event Bus`：消息传递与异步事件流
- `Workflow Engine`：任务状态机与流程推进

简单理解：
- Event Bus 解决“消息怎么流”
- Workflow Engine 解决“任务怎么走完”

## 与 Supervisor 模式的关系
你当前设计的：
- `Supervisor Subagent`
- `cursor-agent`
- `SQLite + artifacts/`

本质上已经是一个轻量级 workflow 模式。

可以理解为：
- 当前方案是 Workflow Engine 的领域化 MVP
- 后续若任务复杂度增加，可向 `Temporal` 这类引擎升级

## 典型能力需求
第一版 Workflow Engine 至少应支持：
- `start`
- `pause`
- `resume`
- `retry`
- `timeout`
- `await_human`
- `complete`
- `fail`

## 推荐技术选择
在当前技术偏好下：

### MVP
- 自建轻量状态机
- `SQLite + artifacts/`
- Supervisor 驱动任务推进

### 升级版
- `Temporal`
- 或带持久状态与重试能力的 workflow 系统

## 什么时候必须升级到正式 Workflow Engine
如果出现以下信号，通常说明必须升级：
- 任务执行时间显著变长
- 单任务步骤数越来越多
- 需要并行与条件分支
- 人工审批节点越来越常见
- 重试、恢复、超时逻辑越来越分散

## 常见错误信号
以下现象通常说明缺少 Workflow Engine：
- 长任务状态靠聊天记录维护
- 失败恢复靠人工重新触发
- 多步骤任务每次都要重新规划
- Skill 本身承担了过多流程控制逻辑
- 人工介入点只能靠临时消息提醒

## 与现有知识库的关系
这篇文档属于平台级蓝图，适合向下连接：
- `methods/`：如何逐步实现 Supervisor / Workflow MVP
- `patterns/`：运行时状态模型、任务协议、review 规则
- `decisions/`：是否引入 Temporal 或正式 Workflow 层

## 后续可继续拆分的主题
- `temporal-adoption-plan.md`
- `workflow-state-machine-pattern.md`
- `human-in-the-loop-gates.md`
- `event-bus-vs-workflow-engine.md`

## 相关文档
- `knowledge/architecture/ai-application-os.md`
- `knowledge/architecture/policy-engine-for-ai-saas.md`
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/methods/mvp-supervisor-implementation-plan.md`
- `knowledge/patterns/runtime-state-schema.md`
