---
type: pattern
status: draft
owner: user
created: 2026-03-16
updated: 2026-03-16
tags: [runtime, sqlite, state, schema, supervisor]
---

# Runtime State Schema

## 问题
如果共享状态只有原则和概念，没有明确的数据模型，Supervisor、`cursor-agent` 和上层编排器之间就难以稳定协作，任务状态也难以恢复、查询和审计。

## 上下文
该规范用于定义 `SQLite + artifacts/` 方案中的运行时状态模型，服务于以下目标：
- 记录 spec 与 task 生命周期
- 保存执行事件、反馈、工件索引
- 支持 Supervisor 状态推进与审查
- 支持 `cursor-agent` 重启后的任务恢复

## 目标
为第一版共享状态提供最小但可扩展的数据结构，使执行系统至少能够：
- 跟踪 spec
- 跟踪 task
- 记录事件流
- 保存工件索引
- 保存反馈与修正建议

## 设计原则
- 状态字段应服务于状态机推进，而不是纯归档
- JSON 字段只用于承载可变扩展信息，核心检索字段应结构化
- 工件内容尽量落文件系统，数据库只存索引和元数据
- 每个 task 和 spec 都应有稳定 ID
- 事件流应支持审计与恢复

## 存储形态
推荐运行时目录：

```text
runtime/
  state.db
  artifacts/
    task-001/
      cursor.log
      patch.diff
      summary.md
```

其中：
- `runtime/state.db`：SQLite 数据库
- `runtime/artifacts/`：任务日志、patch、报告等大对象

## 表结构

### 1. `specs`
保存总体 spec 或执行计划。

建议字段：
- `id TEXT PRIMARY KEY`
- `title TEXT NOT NULL`
- `status TEXT NOT NULL`
- `source_type TEXT`
- `content_json TEXT NOT NULL`
- `created_at TEXT NOT NULL`
- `updated_at TEXT NOT NULL`

推荐状态：
- `draft`
- `active`
- `paused`
- `completed`
- `cancelled`

### 2. `tasks`
保存可执行任务。

建议字段：
- `id TEXT PRIMARY KEY`
- `spec_id TEXT NOT NULL`
- `title TEXT NOT NULL`
- `status TEXT NOT NULL`
- `priority INTEGER DEFAULT 0`
- `assignee TEXT`
- `goal TEXT`
- `scope_json TEXT`
- `constraints_json TEXT`
- `acceptance_json TEXT`
- `payload_json TEXT`
- `last_result_json TEXT`
- `created_at TEXT NOT NULL`
- `updated_at TEXT NOT NULL`

外键关系：
- `spec_id -> specs.id`

推荐状态：
- `planned`
- `ready`
- `dispatched`
- `in_progress`
- `review`
- `needs_fix`
- `blocked`
- `completed`
- `failed`

### 3. `task_events`
保存任务执行过程中的事件流。

建议字段：
- `id INTEGER PRIMARY KEY AUTOINCREMENT`
- `task_id TEXT NOT NULL`
- `event_type TEXT NOT NULL`
- `payload_json TEXT`
- `created_at TEXT NOT NULL`

常见事件：
- `task_created`
- `task_dispatched`
- `execution_started`
- `execution_result_received`
- `review_passed`
- `review_failed`
- `task_blocked`
- `task_completed`

### 4. `artifacts`
保存工件索引。

建议字段：
- `id INTEGER PRIMARY KEY AUTOINCREMENT`
- `task_id TEXT NOT NULL`
- `kind TEXT NOT NULL`
- `path TEXT NOT NULL`
- `meta_json TEXT`
- `created_at TEXT NOT NULL`

常见 `kind`：
- `log`
- `diff`
- `summary`
- `report`

### 5. `feedback`
保存 Supervisor 或上层系统给出的修正意见。

建议字段：
- `id INTEGER PRIMARY KEY AUTOINCREMENT`
- `task_id TEXT NOT NULL`
- `source TEXT NOT NULL`
- `status TEXT`
- `message TEXT NOT NULL`
- `payload_json TEXT`
- `created_at TEXT NOT NULL`

常见 `source`：
- `supervisor`
- `openclaw`
- `reviewer`

## 最小索引建议
第一版建议增加以下索引：
- `INDEX idx_tasks_spec_id ON tasks(spec_id)`
- `INDEX idx_tasks_status ON tasks(status)`
- `INDEX idx_task_events_task_id ON task_events(task_id)`
- `INDEX idx_artifacts_task_id ON artifacts(task_id)`
- `INDEX idx_feedback_task_id ON feedback(task_id)`

## 推荐查询场景

### 查询当前可派发任务
```sql
SELECT id, title
FROM tasks
WHERE status = 'ready'
ORDER BY priority DESC, updated_at ASC;
```

### 查询某个 task 的完整执行轨迹
```sql
SELECT event_type, payload_json, created_at
FROM task_events
WHERE task_id = ?
ORDER BY created_at ASC;
```

### 查询某个 task 的工件索引
```sql
SELECT kind, path
FROM artifacts
WHERE task_id = ?
ORDER BY created_at ASC;
```

## 状态推进约束
建议遵循以下约束：
- 只有 `ready` task 才能被派发
- 只有 `dispatched` task 才能进入 `in_progress`
- `review` 后必须进入 `completed`、`needs_fix` 或 `blocked`
- `blocked` task 需要附带 blocker 信息
- `completed` task 不应再次被派发，除非生成新的 follow-up task

## 与协议层的关系
- `Supervisor Task Payload Spec` 对应 `tasks.payload_json` 的生成来源
- `Cursor Result Payload Spec` 对应 `tasks.last_result_json` 与部分 `task_events`、`artifacts` 的写入来源
- `feedback` 用于保存 Supervisor 的修正意见与复审结论

## 优点
- 可以直接支撑 SQLite MVP
- 便于后续扩展到更多角色或更多状态
- 便于恢复、审计、回放和统计

## 代价
- 需要定义状态迁移规则和写入规范
- 如果 JSON 字段滥用，后续查询会变差

## 不适用场景
- 极简的一次性单步执行
- 完全不需要恢复与审计的实验任务

## 相关文档
- `knowledge/methods/spec-driven-cursor-supervision.md`
- `knowledge/patterns/supervisor-task-payload-spec.md`
- `knowledge/patterns/cursor-result-payload-spec.md`
- `knowledge/principles/shared-state-over-session-memory.md`
