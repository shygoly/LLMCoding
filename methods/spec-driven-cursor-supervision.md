---
type: method
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [cursor, supervisor, sqlite, spec, agent]
---

# Spec Driven Cursor Supervision

## 目标
建立一套“由规范驱动、由监督子代理推进、由 cursor-agent 执行”的自动化编程流程，使长任务不依赖会话记忆，而依赖可追踪、可恢复的共享状态。

## 适用场景
- 已有较完整的 spec、任务清单或产品需求说明
- 希望让 cursor-agent 持续执行多步任务，而不是一次性补丁式修改
- 需要在 OpenClaw、Spec 工具、cursor-agent 之间建立稳定交接
- 需要可恢复、可审计、可重试的执行流程

## 前提条件
- 已有任务来源，如 `spec-kit`、GitHub issue、手工拆解任务或结构化 PRD
- 有一个监督执行的子代理，负责推进任务状态
- 有共享状态存储，用于保存 spec、task、结果、反馈
- cursor-agent 以 daemon 或可持续调用的形式运行

## 输入
- 总体 spec 或目标说明
- 任务列表与依赖关系
- 架构约束、知识文档、决策文档
- 目标仓库与文件范围

## 核心步骤
1. 从 spec 中抽取可执行任务，形成任务树和验收标准
2. 把任务与约束写入共享状态，而不是只留在会话 prompt 中
3. 监督子代理按任务状态推进流程，只把当前任务相关上下文交给 cursor-agent
4. cursor-agent 执行当前任务，返回变更摘要、日志、工件路径和阻塞信息
5. 监督子代理对结果进行审查，决定 `completed`、`needs_fix`、`blocked` 或 `escalated`
6. 对重要变更回写知识库、规范或 review 文档，避免上下文再次丢失

## 产物
- 结构化 spec 记录
- 可追踪的任务状态机
- 执行日志与变更工件
- 监督反馈记录
- 回写后的知识文档或决策更新

## 推荐角色分工
- `OpenClaw`：规划者与编排者，负责目标拆解、知识组织、流程决策
- `Supervisor Subagent`：执行监督者，负责任务推进、结果审查、偏航纠正
- `cursor-agent`：代码执行者，负责修改仓库、运行命令、产出结果
- `Shared Store`：共享事实源，保存状态、日志索引、反馈和工件引用

## 推荐执行流程

```text
Spec / PRD / Issue
    -> OpenClaw 提炼执行范围
    -> Supervisor 写入共享状态
    -> cursor-agent 执行单个 task
    -> Supervisor 审查结果
    -> 通过则推进下一任务
    -> 如有必要，回写 knowledge/
```

## 共享存储方案
MVP 阶段推荐使用：

```text
SQLite + artifacts/
```

其中：
- `SQLite`：保存结构化状态、任务、事件、反馈、工件索引
- `artifacts/`：保存日志、patch、报告、大文本输出

推荐目录：

```text
runtime/
  state.db
  artifacts/
    task-001/
      cursor.log
      patch.diff
      review.md
```

## 为什么用 SQLite
- 单文件部署，适合本地和单机流程
- 支持事务，比 JSON 文件更适合维护状态一致性
- 方便查询任务状态、执行历史和失败记录
- 足以支持低并发、多轮次的监督执行流程
- 未来若要升级到 Postgres，模型也容易迁移

## 最小数据模型
第一版建议至少包含以下实体：

### `specs`
- `id`
- `title`
- `status`
- `content_json`
- `created_at`
- `updated_at`

### `tasks`
- `id`
- `spec_id`
- `title`
- `status`
- `priority`
- `assignee`
- `input_json`
- `acceptance_json`
- `updated_at`

### `task_events`
- `id`
- `task_id`
- `event_type`
- `payload_json`
- `created_at`

### `artifacts`
- `id`
- `task_id`
- `kind`
- `path`
- `meta_json`
- `created_at`

### `feedback`
- `id`
- `task_id`
- `source`
- `message`
- `status`
- `created_at`

## 推荐状态机
任务状态建议从以下集合起步：
- `planned`
- `ready`
- `dispatched`
- `in_progress`
- `review`
- `needs_fix`
- `blocked`
- `completed`
- `failed`

建议流转：

```text
planned -> ready -> dispatched -> in_progress -> review -> completed
```

异常流转：

```text
in_progress -> blocked
review -> needs_fix
needs_fix -> dispatched
blocked -> failed
blocked -> escalated
```

## Supervisor 的职责边界
监督子代理应重点负责：
- 任务裁剪：把大 spec 切成 Cursor 稳定可执行的小步
- 上下文筛选：只传当前任务相关知识与文件范围
- 结果审查：检查是否越界、是否满足验收、是否需要补知识回写
- 状态推进：更新任务状态并决定下一步

监督子代理不应承担：
- 完整长期架构设计的最终裁决
- 无边界地替代 Cursor 自己写代码
- 把所有知识都重新总结一遍再发送给执行器

## 风险与边界
- 如果 spec 本身质量差，监督流程也只能更稳定地执行错误目标
- 如果共享状态不是真正权威来源，系统仍会退化为会话驱动
- 如果 Supervisor 太“聪明”，会演变成另一个不可控的自由 Agent
- 如果 cursor-agent 直接跳过共享状态自主扩展任务，流程会失控

## 验证方式
- daemon 重启后，任务是否可从共享状态恢复
- 长任务是否能追踪每一步的输入、结果与反馈
- 新执行轮次是否能复用上一次状态，而不是重新理解项目
- 监督子代理是否能识别偏航并给出 `needs_fix` 或 `blocked`

## 相关文档
- `knowledge/principles/agent-query-before-code.md`
- `knowledge/patterns/unified-pkg-layer.md`
- `knowledge/inbox/2026-03-16-ai-software-factory-notes.md`
