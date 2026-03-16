---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [cursor, result, payload, supervisor, protocol]
---

# Cursor Result Payload Spec

## 问题
如果 cursor-agent 回传给 Supervisor 的结果没有固定结构，监督层就很难自动判断任务是否完成、是否越界、是否需要修正，也很难把执行结果稳定写入共享状态与工件索引。

## 上下文
该规范定义 `cursor-agent -> Supervisor Subagent` 的单任务结果回传格式。它与 `Supervisor Task Payload Spec` 配对使用，负责表达一次任务执行后的状态、变更、阻塞、工件和建议动作。

## 目标
让每次执行结果都能稳定回答：
- 当前 task 执行到了什么状态
- 改了哪些文件
- 产生了哪些工件
- 是否遇到阻塞
- 是否满足验收
- 下一步需要什么动作

## 设计原则
- 结果只对应一个 `taskId`
- 状态必须显式，而不是让 Supervisor 从自然语言猜测
- 文件变更、命令结果、阻塞原因应结构化表达
- 可落盘、可审计、可供状态机消费
- 支持 `completed`、`needs_fix`、`blocked` 等常见结果分支

## 顶层结构
推荐使用 JSON：

```json
{
  "taskId": "task-007",
  "specId": "spec-001",
  "status": "review",
  "summary": "已为生成器增加 list/repeat 结构支持",
  "changes": {},
  "checks": [],
  "artifacts": {},
  "blockers": [],
  "writeback": {},
  "nextAction": "await_review"
}
```

## 字段规范

### 1. `taskId`
当前结果对应的任务 ID。

要求：
- 必须与派发载荷中的 `taskId` 一致
- 作为关联状态与工件的主键之一

### 2. `specId`
所属 spec 的 ID。

用途：
- 便于 Supervisor 在总任务树中定位本次结果

### 3. `status`
本次执行后的任务状态建议。

推荐值：
- `review`
- `completed`
- `needs_fix`
- `blocked`
- `failed`

说明：
- `cursor-agent` 可以提出状态建议
- 最终状态由 `Supervisor` 决定并写入共享状态

### 4. `summary`
对本次执行结果的一句话摘要。

要求：
- 说明完成了什么
- 避免只有“已处理”“已修改”这种空泛表述

### 5. `changes`
描述本次代码变更情况。

建议字段：

```json
{
  "changedFiles": [
    "packages/generator/src/page.ts",
    "packages/ui-schema/src/types.ts"
  ],
  "addedFiles": [],
  "deletedFiles": [],
  "outOfScopeChanges": []
}
```

说明：
- `outOfScopeChanges` 用于显式标记可能超范围的改动

### 6. `checks`
记录执行过程中的验证动作及结果。

示例：

```json
[
  {
    "type": "test",
    "target": "packages/generator",
    "status": "passed",
    "summary": "generator tests passed"
  },
  {
    "type": "build",
    "target": "packages/ui-schema",
    "status": "skipped",
    "summary": "not required for this task"
  }
]
```

推荐字段：
- `type`：如 `test`、`build`、`lint`、`manual-check`
- `target`
- `status`：`passed`、`failed`、`skipped`
- `summary`

### 7. `artifacts`
记录本次执行产物的位置。

示例：

```json
{
  "logPath": "runtime/artifacts/task-007/cursor.log",
  "diffPath": "runtime/artifacts/task-007/patch.diff",
  "summaryPath": "runtime/artifacts/task-007/summary.md"
}
```

### 8. `blockers`
记录当前阻塞信息；无阻塞时可为空数组。

示例：

```json
[
  {
    "type": "missing-context",
    "message": "未找到 list 组件的既有映射逻辑",
    "needs": [
      "需要确认 list 组件应映射到哪类 Taro 结构"
    ]
  }
]
```

推荐字段：
- `type`
- `message`
- `needs`

### 9. `writeback`
说明是否建议回写知识库、规范或状态文档。

示例：

```json
{
  "knowledgeUpdateSuggested": true,
  "targets": [
    "knowledge/patterns/ui-schema-spec.md"
  ],
  "reason": "新增了 list.repeat.source 字段支持"
}
```

### 10. `nextAction`
对下一步的建议。

推荐值：
- `await_review`
- `ready_for_next_task`
- `needs_clarification`
- `retry_after_fix`

## 最小可用结果示例

```json
{
  "taskId": "task-007",
  "specId": "spec-001",
  "status": "review",
  "summary": "已为生成器增加 list/repeat 结构支持，并补充类型定义",
  "changes": {
    "changedFiles": [
      "packages/generator/src/page.ts",
      "packages/ui-schema/src/types.ts"
    ],
    "addedFiles": [],
    "deletedFiles": [],
    "outOfScopeChanges": []
  },
  "checks": [
    {
      "type": "test",
      "target": "packages/generator",
      "status": "passed",
      "summary": "generator tests passed"
    }
  ],
  "artifacts": {
    "logPath": "runtime/artifacts/task-007/cursor.log",
    "diffPath": "runtime/artifacts/task-007/patch.diff",
    "summaryPath": "runtime/artifacts/task-007/summary.md"
  },
  "blockers": [],
  "writeback": {
    "knowledgeUpdateSuggested": true,
    "targets": [
      "knowledge/patterns/ui-schema-spec.md"
    ],
    "reason": "新增了 list.repeat.source 的支持"
  },
  "nextAction": "await_review"
}
```

## 审查要点
Supervisor 在消费结果时应检查：
- `taskId` 与当前 in-progress task 是否一致
- `status` 是否与 `blockers`、`checks` 内容一致
- `changes.changedFiles` 是否超出 task scope
- `checks` 是否满足最小验收要求
- `writeback` 是否提示需要更新知识库
- `nextAction` 是否与当前状态机兼容

## 优点
- 便于自动审查执行结果
- 便于写入 SQLite 状态与事件表
- 便于生成 review、修正意见和审计记录
- 让 Cursor 执行结果从“自然语言回答”升级为“结构化回传”

## 代价
- 需要执行器严格遵守回传格式
- 若字段设计过多，会增加第一版集成复杂度

## 不适用场景
- 非结构化、纯人工盯执行的临时任务
- 完全不需要自动审查与恢复的短流程

## 相关文档
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/supervisor-subagent-with-shared-store.md`
- `knowledge/methods/spec-driven-cursor-supervision.md`
